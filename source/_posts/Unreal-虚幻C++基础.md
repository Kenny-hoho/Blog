---
title: Unreal-虚幻C++基础
date: 2023-10-12
index_img: "/img/bg/ue5.jpg"
tags: [虚幻引擎]
categories: 
   -[客户端]
---


<!-- more -->

# Unreal项目规范：
## [Allar/ue5-style-guide](https://github.com/Allar/ue5-style-guide)
## [中文翻译版](https://github.com/skylens-inc/ue4-style-guide/blob/master/README.md)

常用的命名规范有：
1. 派生自UObject类的都以U为前缀（派生自U开头类的类也都已U为前缀）
2. 派生自AActor的以A为前缀
3. bool变量以b前缀，如bPendingDestruction
4. 抽象接口以I前缀 
   ![](/article_img/2023-11-11-14-59-41.png)
   虚幻中派生自UInterface的类都有两个定义，一个以U开头实现作为一个基本类的功能，一个以I开头实现接口功能
![](/article_img/2023-10-13-14-17-23.png)

# 虚幻C++基础

虚幻的C++开发不是使用纯粹的C++进行编程，虚幻引擎为了实现各种复杂机制，在C++基础上搭建了一套反射机制。下面是我们新建一个C++类，虚幻自动帮我们生成的头文件的结构，其中有很多宏定义和必须引用的头文件：

```C++
#pragma once

#include "CoreMinimal.h"
#include "GameFramework/Actor.h"
#include "SBaseProjectile.generated.h"

UCLASS()
class CPPRPG_API ASBaseProjectile : public AActor
{
	GENERATED_BODY()
	
public:	
	// Sets default values for this actor's properties
	ASBaseProjectile();

	UPROPERTY(EditAnywhere, Category="Attribute of Base Projectile")
	float ProjectileMoveSpeed;

protected:
	// Called when the game starts or when spawned
	virtual void BeginPlay() override;

public:	
	// Called every frame
	virtual void Tick(float DeltaTime) override;
};

```

## 反射

反射在Java和C#等语言中比较常见，概况的说，反射数据描述了类在运行时的内容。这些数据所存储的信息包括类的名称、类中的数据成员、每个数据成员的类型、每个成员位于对象内存映像的偏移（offset），此外，它也包含类的所有成员函数信息。

简单来说，反射可以实现在程序运行时**动态地对类，对象，方法等进行操作**。例如在虚幻中，反射用来实现序列化（就是将对象转换为字节流，存储和网络传输使用），编辑器中动态调整各种属性，垃圾回收，网络复制，蓝图/C++通信和相互调用的功能。

虚幻引擎的反射机制的核心是**UObject**类，所有的类都继承自UObject：
![](/article_img/2023-11-11-14-54-02.png)

**#include "xxx.generated.h"** 以及 **UPROPERTY()**，**UFUNCTION()**，**UCLASS()** 等宏定义都用来让UE实现反射。

## UBT（Unreal Build Tool）

虚幻项目的编译全权由**UBT**负责，其本质就和其名字一样就是一个**编译工具**，和CMake之类的是一个东西，只不过其是专门为UE量身定做的编译工具。有了UBT就可以方便配置各个平台的参数和编译选项，不用为每个平台单独配置编译了。

我们可以直接使用Engine\Binaries\DotNET\UnrealBuildTool目录下的UnrealBuildTool.exe再命令行中编译我们的项目。在创建新项目时，自动生成的Target.cs，Build.cs都是为这个工具服务的。

UBT工作流程分为两步：
1. 调用UHT（Unreal Header Tool）
2. 调用平台特定的编译工具（VS，LLVM）编译各个模块

## UHT（Unreal Header Tool）

UHT（Unreal Header Tool）是用来解析虚幻C++的，上面我们提到了要实现UE反射需要些各种宏，**UHT就是将这些代码解析成标准的C++代码**，之后再由UBT组织这些标准C++代码进行正常的C++编译。

可以这样直观理解，我们写的虚幻C++代码其实和标准C++不同，有很多宏以及很多省略或者固定的写法（接口函数实现xxx_Implementation），可以将我们写的代码称为U++。**编译过程就是调用UBT，UBT先调用UHT将U++翻译为C++，再调用C++编译工具对C++编译。**

# 参考

[大象无形UE4笔记六：UBT](https://zhuanlan.zhihu.com/p/562841250)
[UE4反射机制](https://zhuanlan.zhihu.com/p/60622181)
[《InsideUE4》基础概念](https://zhuanlan.zhihu.com/p/22814098)

