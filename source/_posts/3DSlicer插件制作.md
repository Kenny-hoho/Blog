---
title: 3DSlicer配置与编译
date: 2022-08-07
index_img: "/img/bg/3DSlicerLogo.png"
tags: [3DSlicer]
hide: true
categories: 
   -[笔记]
---
# 3DSlicer插件制作
## 参考文档：  
[3DSlicer建立C++插件步骤](https://blog.csdn.net/sang_12345/article/details/121805914)  
[3DSlicer: Module-Create Loadable](https://www.freesion.com/article/1531976481/)  
[官方文档](https://slicer.readthedocs.io/en/latest/developer_guide/extensions.html)

## 1.准备工作
如果选择开发命令行接口（Command Line Interface (CLI)）或者可加载插件需要先编译3DSlicer，如果是**单纯python脚本不需要编译3DSlicer**
![](/article_img/2022-06-06-18-35-54.png)
## 2. 插件模板生成
使用3DSlicer自带的Extension Wizard模块可以方便的创建插件。这里建立一个script类型的插件，具体参考官方文档，会自动生成一个script类型的插件的模板python文件，可以在这个模板文件中进行修改。
## 3. 插件功能开发流程
3DSlicer功能强大完全可以仅使用3DSlicer和代码编辑器就可以进行插件的实现和调试，这里就仅使用pycharm作为编辑器（没有配置python编译器，编译调试均在3DSlicer中进行）

![](/article_img/2022-06-20-14-08-41.png)

**Python Interactor** 可以在左上角的 **View** 中打开，或者使用快捷键 **ctrl+ 3** 打开，利用模板生成的按钮可以进行调试：

### **常用按钮**

**Reload**：可以重新加载插件，编辑python文件之后保存，点击这个按钮就可以重载插件，十分方便用于调试。注：对上方的**Help & Acknowledge** 的修改需要重启之后才能应用。
**Edit UI**：直接从Designer打开 ui 文件，方便直接对插件界面进行修改（对界面的修改仅限于左侧部分界面，右侧窗口不能修改）

## 4. 插件功能实现

对插件功能的实现主要是利用3DSlicer本身集成的python package和其自定义的包进行开发，也可以使用3DSlicer不包含的第三方包，使用方法是利用3DSlicer中的pip

```python
try:
    import library_name
except:
    slicer.util.pip_install('library_name')
    import library_name
```

### **1）参考文档**
[Script Repository](https://slicer.readthedocs.io/en/latest/developer_guide/script_repository.html#change-slice-orientation) 介绍常用功能实现的脚本

[slicer package](https://slicer.readthedocs.io/en/latest/developer_guide/slicer.html#slicer.util.MRMLNodeNotFoundException) 对 **slicer** 库进行介绍,开发插件常用的主要功能的实现都需要使用这个库，来对3DSlicer软件进行定制，[部分slicer库函数代码介绍](mwoehlke-kitware.github.io/Slicer/Base/slicer.html#slicer.util.quit)

### **2）导入图像功能实现**

本插件主要处理的图像是2D图像，格式为jpeg，png等，这些图导入3DSlicer时不能直接将这些格式的图像导入，需要先转换成3DSlicer支持的数据类型，图片格式一般要转换成 **Volume** 类型，具体[数据类型](https://www.slicer.org/wiki/Documentation/4.8/SlicerApplication/SupportedDataFormat)参考如下：

![](/article_img/2022-06-20-14-41-24.png)

导入的图片有几种不同的方法，都是利用 **slicer** 库：

```python
# loadNodeFromFile方法，这个会导入同文件夹中的好几张图片，尽管只选择导入一个文件
slicer.util.loadNodeFromFile('filepath', 'VolumeFile')

# loadVolume方法，可以设定只导入一张，**选择这个方法**
loadedVolumeNode = slicer.util.loadVolume('filepath', {"singleFile": True})

# 直接调用3DSlicer的导入数据模块，但不能得到用户选择的文件路径
slicer.util.openAddDataDialog()
```

这三种方法导入之后，都会自动显示到右侧窗口，覆盖之前显示的内容。

针对前两种方法，要获得用户选择的文件需要使用3DSlicer自带的 **qt** 库，这里的qt库不是一般我们在python中使用的 **pyqt** 库，一开始没有找到这个库，以为需要导入 **pyqt**，但是 **pyqt** 的函数都需要定义在一个继承自qt的库，这里不满足这个条件，之后在[论坛讨论](https://discourse.slicer.org/t/opendialog-does-not-load-node-into-scene/23874)中找到了解决方法：
```python
import qt

d = qt.QFileDialog()
d.exec()
slicer.util.loadVolume(d.selectedFiles()[0])
```

这里其实本来也有3DSlicer封装好的函数，用来打开文件浏览器，

```python
# openDialog()方法，但是这个函数暂时出了bug，在写文章一周前才在源码层面解决，所以就用上面那种方法。
import slicer

 io = slicer.app.ioManager()
 params = {}
 io.openDialog("VolumeFile", slicer.qSlicerFileDialog.Read, params)
```

### **3）显示图像实现**

![](/article_img/2022-06-20-18-05-00.png)

在3DSlicer中，其右侧的窗口不能直接显示jpeg，png等等图片，需要先 **Load** 也就是上述的导入，在导入之后，会自动将图片显示在右侧，就算不是三维图像也会在 **yellow** 层和 **Green** 层（也就是三视图的另外两个方向），所以需要自定义显示的窗口

```python
# 显示到红色窗口
red_logic = slicer.app.layoutManager().sliceWidget("Red").sliceLogic()
red_logic.GetSliceCompositeNode().SetBackgroundVolumeID(VolumeName.GetID())
# 将图像移到窗口中间，相当于点击上图中的Reset field of view
slicer.util.resetSliceViews()
```

![](/article_img/2022-06-20-18-11-02.png)

希望实现的效果如上图，左侧是原始图像，右侧是处理之后的图像。但是 **yellow** 窗口默认方向是 **sagittal** 用如下代码进行更改

```python
# 改变黄色窗口的显示方向
slice_node = slicer.app.layoutManager().sliceWidget("Yellow").mrmlSliceNode()
slice_to_ras = slice_node.GetSliceToRAS()
transform = vtk.vtkTransform()
transform.SetMatrix(slice_to_ras)
transform.RotateX(90)
transform.RotateZ(90)
slice_to_ras.DeepCopy(transform.GetMatrix())
slice_node.UpdateMatrices()
```

### **4）显示图像实现**