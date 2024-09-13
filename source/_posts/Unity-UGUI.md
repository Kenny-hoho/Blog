---
title: Unity-UGUI源码解析和优化方法
date: 2024-6-17
index_img: "/img/bg/xt2.png"
tags: [Unity]
categories: 
   -[Unity]
---

<!-- more -->

[Unity3D UGUI系列之合批](https://blog.csdn.net/sinat_25415095/article/details/112388638)
[UGUI性能优化](https://www.drflower.top/posts/aad79bf1)

# UGUI原理

## UGUI流程

UGUI渲染以Canvas为单位调用DrawCall，Canvas会尽力把其下的所有图元合并成一个mesh，因此 **ReBatch** 和 **ReBuild** 也是以Canvas为单位。Canvas下的每个图元都有一个**CanvasRenderer**组件负责将该图元传递给Canvas，当需要更新图元时（几何和材质发生了变化），**CanvasUpdateRegistry** 会把 **PerformUpdate()** 函数注册到事件 **Canvas.willRenderCanvases** （Canvas每帧都会发送此事件），就会开始对图元的重建，分为两步，首先遍历 **m_LayoutRebuildQueue** 对其中的图元进行 layout重建，之后遍历 **m_GraphicRebuildQueue** 对其中的图元进行 Graphic 重建。所有的图元都继承自**Graphic**类，每个图元类会实现 **Rebuild()** 对几何和材质的重建。

总结几个UGUI基础概念：
- Canvas：负责合批操作，调用DrawCall的单位
- CanvasRenderer：负责将每个图元的数据发给Canvas
- Graphic：所有图元的基类，每个图元子类都需要实现 Rebuild() 完成对自己的更新
- Rebuild：重新计算布局和图形网格的过程
- Rebatch：重新计算合批，发生在一个Canvas下有任何图元发生变化时，Rebuild通常会引起Rebutch

下图展示UGUI的渲染流程：[图片来源](https://km.woa.com/articles/show/454768?kmref=search&from_page=1&no=3)
![](/article_img/2024-06-21-11-31-28.png)

## C#层Rebuild

**图元相关类图**：

![](/article_img/UGUI图元类图.png)

可以从该函数入手，**CanvasUpdateRegistry** 负责将PerformUpdate注册到willRenderCanvases（每帧会执行该函数），因此更新逻辑就在PerformUpdate中；
```csharp
class CanvasUpdateRegistry{
   protected CanvasUpdateRegistry(){
      Canvas.willRenderCanvases += PerformUpdate;
   }
}
```
**CanvasUpdateRegistry** 维护两个队列：
- IndexedSet\<ICanvasElement> m_LayoutRebuildQueue：存放需要进行 **布局重建** 的图元
- IndexedSet\<ICanvasElement> m_GraphicRebuildQueue：存放需要进行 **图元重建** 的图元

当图元属性改变时会更新图元重建队列，当布局变化时会更新布局重建队列；PerformUpdate中会遍历这两个队列进行 **布局重建** 和 **图元重建**；

重建时会根据脏标记分别调用 **UpdateGeometry** 更新几何，**UpdateMaterial** 更新材质，更新几何时会调用 **DoMeshGeneration**，又会调用 **OnPopulateMesh**，最终会**new出新的顶点信息**添加到 **VertexHelper**；
![](/article_img/Rebuild函数调用.png)

可以看出UGUI的脏标记只区分了材质和几何两方面，因此只要任何顶点信息发生变化都会进行几何重建，同时会新建顶点放入vertexHelper中而不是更改对应顶点的数据，这个是UGUI的主要耗时之一；例如颜色变化很常见被大量用于UI渐变动画等操作，color信息也存在顶点信息中，每次color的变化都会带来几何重建；在旧版本的UGUI中使用了布尔值UseRenderColor来控制不适用顶点color，而是使用CanvasRenderer的color修改颜色从而避免修改颜色带来的重建，但是在Unity2021的UGUI中取消了这个变量。但是仍然保留了CanvasRenderer的**color**属性和**SetColor**方法（会设置CanvasRender的颜色，与顶点颜色相乘）
因此：**建议需要对顶点颜色做频繁变化的操作时使用CanvasRenderer.SetColor(Color color)** 避免频繁的几何重建消耗。
![](/article_img/2024-07-26-11-21-28.png)
同时，**当图元Disable时会清空所有顶点数据**，因此频繁的对图元 active/deactive 会造成顶点信息的频繁销毁和创建，有不小的消耗；
```csharp
   // disable中调用CanvasRenderer.Clear
   protected override void OnDisable()
   {
      //...
      if (canvasRenderer != null)
            canvasRenderer.Clear();
      //...
   }
   // cpp中的CanvasRenderer.Clear函数
   void CanvasRenderer::Clear()
   {
      SetMesh(NULL);
      SetColorNoSync(ColorRGBAf(1.0f, 1.0f, 1.0f, 1.0f));
      SetMaterialCount(0);
      SetTexture(NULL);
      SetAlphaTexture(NULL);

      DirtySyncTypeFlag(kForceSync | kSyncColor | kSyncWorldRect | kSyncBounds | kSyncVertexPtr | kSyncMaterial);
   }
```

## Cpp层Rebatch

Canvas是Native层实现的Unity组件，其负责将canvas下的图元合批并调用DrawCall，这个合批过程被称为ReBatch；图元通过调用 **GraphicRegistry.RegisterGraphicForCanvas()** 和 **GraphicRegistry.UnregisterGraphicForCanvas()**，将当前图元绑定到离自己最近的canvas，绑定由字典实现；

**Rebatch过程：**
1. 首先生成一个数组包括所有的待合批图元；
2. 之后对这个数据进行排序，具体规则依次是：深度排序、材质排序、纹理排序得到队列；
3. 再判断相邻队列元素能否合批，最后将能合批的元素网格合并成一个网格，提交渲染；
UGUI合批示意图：[图片来源](https://km.woa.com/articles/show/454768?kmref=search&from_page=1&no=3)
![](/article_img/2024-06-21-14-27-58.png)

# UGUI优化

我们已经知道UGUI渲染中主要的耗时操作为：
1. C#层中的**Rebuild**：Canvas.SendWillRenderCanvases
2. Cpp层中的**Rebatch**：Canvas.BuildBatch

![](/article_img/2024-07-26-14-55-46.png)
下面从两方面总结使用UGUI时的优化手段

## Rebuild优化

### 触发条件

图元需要Rebuild时会调用：CanvasUpdateRegistry.RegisterCanvasElementForGraphicRebuild(this)，将自己加入到需要rebuild队列中，查看该函数的调用即可知道什么时候会触发Rebuild；

1. Layout修改RectTransform部分影响布局的属性
2. Graphic的Mesh或Material发生变化时（顶点属性变化，Enable/Disable）
3. Mask裁剪内容变化

### 优化技巧

1. 少用layout，简单布局用RectTransform
2. 频繁修改颜色时使用CanvasRenderer.SetColor，不要直接修改图元Color
3. 设置可见性时不用Enable/Disable，可以使用CanvasRenderer.SetAlpha，设置透明度为0，该函数内部调用CanvasRenderer.SetColor；或者设置scale为0
4. 设置UI宽高时可以考虑不适用RectTransform.width\height，而是使用RectTransform.Scale，后者不会触发几何重建，会在UIGeometryJob中修改矩阵；
5. 降低更新频率，需要实时更新的血条等可以考虑隔帧更新或者设置更新阈值

## Rebatch优化

UGUI以Canvas为单位进行Rebatch，当Canvas下（不包括子Canvas）的任意图元发生Rebuild时会触发Rebatch，按照上述Rebuild优化技巧减少Rebuild次数也可以减少Rebatch次数。因此这里主要考虑对于Rebatch本身的优化。

### 优化技巧

1. 动静分离，通过上述分析很好理解，将需要频繁更新的图元和不需要频繁更新的图元分开到不同的Canvas下，使得频繁变化的Canvas里只对自己的Canvas下的元素进行Rebatch，而节省掉另一个Canvas中不需要变化的元素的Rebatch计算。注意只有相同的Canvas下的图元才有可能合批，动静分离本质是用更多的合批数换取BuildBatch时间的减少，因此在实际操作时要权衡DrawCall和BuildBatch；在Unity5.2后对BuildBatch做了优化，使用子线程计算，导致目前动静分离产生的优化效果可能没有那么显著；
2. 注意合批规则：UGUI会batch规则对Canvas下的图元进行排序得到队列再判断相邻队列能否合批（相同material和texture），具体规则依次是：深度排序、材质排序、纹理排序得到队列；因此要尽量减少节点的层级（排序计算速度快），使用相同材质和纹理的节点处于同一深度等，总之就是尽量不要打断合批；

## 通用优化

除了对Rebuild和Rebatch过程的具体优化，还有一些通用的优化技巧

### Overdraw

1. 慎用自带的OutLIne和Shadow（会多次绘制）
2. 减少UI重叠层级
3. 对于需要暂时隐藏的UI，不要直接把Color属性的Alpha值改为0，UGUI中这样设置后仍然会渲染，应该用CanvasGroup组件把Alpha值置零

### Raycast

1. UGUI创建的所有可点击的组件都会默认开启RaycastTarget，当进行点击操作时，会遍历所有开启RaycastTarget的组件进行检测和排序，因此在创建UI时将RaycastTarget关闭；
2. 需要响应Raycast事件时，不要使用空Image，可以自定义组件继承自MaskableGraphic，重写OnPopulateMesh把网格清空，这样可以响应Raycast而又不需要绘制Mesh