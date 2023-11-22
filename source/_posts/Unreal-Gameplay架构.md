---
title: Unreal-Gameplay架构
date: 2023-11-12
index_img: "/img/bg/ue5.jpg"
tags: [虚幻引擎]
categories: 
   -[客户端]
---

《InsideUE》笔记
<!-- more -->

# 什么是Gameplay架构

UE官方文档中对Gameplay框架的定义是：**游戏规则、玩家输出与控制、相机和用户界面等核心系统。** 也就是 **游戏规则、3C（Camera、Character、Control）和UI**

![](/article_img/2023-11-21-14-44-55.png)

那么在UE中 **游戏规则、3C和UI** 具体是什么呢？

## 游戏规则——GameMode和GameState

**GameMode** 决定游戏规则，比如玩家应该在哪生成，设置当前该显示什么UI等等。每个Level需要设置一个GameMode。GameMode在Unreal里的实现是AGameModeBase类。

**GameState** 指游戏状态，追踪游戏层面的属性，比如当前游戏进行了多长时间，记录剩余敌人数量等等信息。GameState在Unreal中的实现是AGameStateBase类。与GameState类似的还有**PlayerState**用来记录玩家的信息，如血量，得分等信息，由APlayerState实现。

## 3C——Camera Character Control

**Camera**在游戏中十分重要，他决定玩家如何观察这个游戏世界，一般用CameraComponent将其挂载在角色身上。

**Character**就是游戏角色，他的父类是APawn类（**Pawn 是可那些由玩家或 AI 控制的所有 Actor 的基类。Pawn 是玩家或 AI 实体在游戏场景中的具化体现**），Character在Pawn的基础上添加了角色移动组件（CharacterMovementComponent），胶囊体组件（CapsuleComponent）和骨骼网格体组件（SkeletonMeshComponent）。

**Contoller**是一种可以控制Pawn（或Pawn的派生类，例如角色（Character）），从而控制其动作的非实体Actor。默认情况下，控制器与Pawn之间存在一对一的关系；也就是说，每个控制器在任何给定的时间只控制一个Pawn。

![](/article_img/2023-11-21-15-11-29.png)

## UI与HUD

**HUD**指的是游戏期间在屏幕上覆盖的状态和信息，不可互动。

**用户界面（UI）** 指的是菜单和其他互动元素。这些元素通常是在屏幕上覆盖绘制的， 就像HUD一样，但是可以互动。

除了这两种之外，UE还提供了**Slate**，其是一种完全自定义，与平台无关的用户界面框架。

下面我们按照《InsideUE》的顺序详细学习UE的Gameplay框架。

# Actor和Component

一个Gameplay框架最基础的问题是如何看待游戏世界，也就是游戏世界中的种种物体应该如何表示，Unity中使用GameObject，UE中就是Actor。

## Actor

![](/article_img/2023-11-22-15-33-51.png)
AActor继承自UObject（之前我们提到的虚幻中最基础的类，用于实现反射，序列化等等功能，所有的类都继承自UObject），Actor无疑是UE中最重要的角色之一，组织庞大，最常见的有StaticMeshActor, CameraActor和 PlayerStartActor等。

与GameObject不同Actor本身不带有Transform属性，我们如果直接新建一个继承自Actor的蓝图，会发现他会自带一个**SceneComponent**，SceneComponent中实现了Transform属性。
![](/article_img/2023-11-22-15-43-19.png)
这是因为UE认为一些不被显式表示的东西也是Actor，例如AInfo(派生类AWorldSetting,AGameMode,AGameSession,APlayerState,AGameState等)，AHUD,APlayerCameraManager等，这些不需要“放置”在游戏世界中，所以就没有让Actor自身带有Transform属性。UE游戏世界中的各种显式表示，规则，状态等等都是Actor。

## Component

Component和Actor组合使用，基本的Actor甚至连Transform能力都要Component来实现也可以看出，**Actor其实更像是一个容器，只提供了基本的创建销毁，网络复制，事件触发等一些逻辑性的功能**，其他的功能由各种各样的Component来实现，需要用到该功能的时候就添加对应的Component即可。UActorComponent类也继承自UObject。
![](/article_img/2023-11-22-15-56-12.png)
左侧依次提供了物理，材质，网格最终合成了一个StaticMeshComponent，右侧的ChildActorComponent可以让Actor之间互相嵌套。

最基本的Component就是SceneComponent，其提供了**Transform**和**SceneComponent的互相嵌套功能**。
![](/article_img/2023-11-22-16-01-18.png)
可以看出，Actor本身不关心父子嵌套关系，而把父子的关系维护都交给了具体的Component。这也很能体现UE的设计思想：Actor要尽可能简单，连嵌套关系都交给Component实现。

有了Actor和Component我们就有了一个构建游戏世界的抓手，之后我们要把这些虚拟“演员”放到哪里呢？也就是该如何组织这些Actor呢？

# Level和World

最基本的想法就是创建一个虚拟对象“World”来包容所有的Actor们，但是玩家其实不会一次性看到和游玩到整个World，所以一般的游戏引擎会把整个世界拆分为多个部分，Unity中拆成一个个Scene，由一个SceneManager在不同部分之间切换；UE中则是拆成Level，这些Level组成一个World。

## Level

![](/article_img/2023-11-22-16-25-48.png)

# 参考

[InsideUE5专栏](https://www.zhihu.com/column/insideue4)
[Unreal Engine的Gameplay框架和重点](https://zhuanlan.zhihu.com/p/612837045)
[UE甜筒#29.UE4/UE5虚幻引擎的的Gameplay框架—从Actor聊到GameInstance](https://www.bilibili.com/video/BV12Y411a75k/?spm_id_from=333.337.search-card.all.click&vd_source=93b215eab72b2548f75d0772e28f8b20)
