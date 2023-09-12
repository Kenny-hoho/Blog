---
title: 腾讯游戏客户端公开课-UE入门
date: 2023-09-12
index_img: "/img/bg/ue5.jpg"
tags: [游戏客户端公开课]
categories: 
   -[客户端]
---

<!-- more -->

# 编译虚幻引擎

[官方文档——下载虚幻引擎源代码](https://docs.unrealengine.com/5.2/zh-CN/downloading-unreal-engine-source-code/)

虚幻引擎是开源的，但是要访问github上的虚幻引擎源码需要将自己的github账号和epic账号绑定并授权，具体操作流程参考UE[官方教程](https://www.unrealengine.com/zh-CN/ue-on-github)。之后我们就可以愉快的访问虚幻引擎的github仓库了：

![](/article_img/2023-09-12-09-54-18.png)

**编译源码需要预留大约250G的硬盘空间**，我全部编译完UE5.2的文件夹大小为215G

clone源码：
```git
git clone https://github.com/EpicGames/UnrealEngine.git
```

直接clone源码默认是在release分支下，当我们把源码全部clone到本地之后，记得切换到想要编译的版本再进行后续操作，这里选择编译安装的5.2.1版本。

```git
git checkout 5.2.1-release 
```

之后依次运行根目录下的 **Setup.bat** (下载一些需要的依赖，需要一两个小时) 和 **GenerateProjectFiles.bat**（生成UE5.sln，几分钟）

之后就可以从 **VS2022** 打开 **UE5.sln** 开始编译了，为了避免一些可能的编译问题，最好在Visual Studio Installer中安装如下模块（使用Unity的游戏开发可以不安装）：

![](/article_img/2023-09-12-10-13-51.png)
![](/article_img/2023-09-12-10-14-11.png)

在打开UE5.sln之后，VS会自动检测需要安装那些组件，跟随指引安装即可：
![](/article_img/2023-09-09-19-55-45.png)

安装完成之后，右键UE5，点击生成，开始编译（我电脑编译大约七个小时。。）
![](/article_img/2023-09-12-10-18-00.png)

编译完成之后找到这个地址 **%Engine-Folder%\UnrealEngine\Engine\Binaries\Win64\UnrealEditor.exe** ，启动！
![](/article_img/2023-09-12-10-32-37.png)

# 创建项目

记录从源码版引擎创建第三人称模板，打包到安卓平台的过程。
参考[官方文档——Android开始入门](https://docs.unrealengine.com/5.2/zh-CN/getting-started-and-setup-for-android-projects-in-unreal-engine/)

打开虚幻编辑器后，按照如下选项新建项目，注意目标平台选择 **移动平台**，质量预设选择 **可缩放**，项目名称为全英文：
![](/article_img/2023-09-12-10-36-08.png)

创建完成之后会自动打开VS2022，显示刚刚创建的项目工程，右键项目名称，点击生成：
![](/article_img/2023-09-12-10-47-44.png)
对于第三人称模板来说大约需要几分钟，之后就可以在虚幻编辑器中找到该项目；

打开项目之后，由于我们想要打包到安卓平台，为了保证开发中和最后在手机上运行效果一致，我们需要设置预览渲染级别（第一次设置之后编译shader需要一段时间）：
![](/article_img/2023-09-12-10-58-00.png)

通过运行按钮左边的按钮可以设置当前的预览渲染模式：
![](/article_img/2023-09-12-10-59-18.png) | ![](/article_img/2023-09-12-10-59-41.png)
---|---

# 打包到安卓平台

按照[官方文档——Android开始入门](https://docs.unrealengine.com/5.2/zh-CN/getting-started-and-setup-for-android-projects-in-unreal-engine/)步骤进行即可，这里记录几个需要注意的点和自己遇到的报错；

## Android Studio

安装 Android Studio 时需要严格按照官方文档下载 **Android Studio 4.0**，不要下载最新版，因为最新版不自带jdk，会导致之后打包时报错。安装完成之后，要把系统变量中的 **JAVA_HOME** 值改为 Android Studio文件夹中的jre路径（设置完毕后重启电脑），在我们按照教程启动 **SetupAndroid.bat** 的时候会去找系统变量中的 **JAVA_HOME**：
![](/article_img/2023-09-12-11-13-23.png)

如果使用自己的jdk会报如下错误或者类似的和rungradle.bat有关的错误：
![](/article_img/2023-09-12-12-09-35.png)

## UE项目设置

需要勾选生成apk，否则不能正常安装到手机：
![](/article_img/2023-09-12-13-54-18.png)

这里前三项默认即可（会在 **SetupAndroid.bat** 中自动设置），后两项需要和对应的版本一致，**SDK API Level**这里默认填写的是 **latest**，会使用电脑中最新版本的SDK，这里要注意不要和**SetupAndroid.bat** 中的设置冲突（该文件中会定义用到的SDK的版本，如果下载最新版的Android Studio会默认下载更高版本的SDK，也会造成打包失败）；
![](/article_img/2023-09-12-13-55-48.png)

## 手机设置

在完成上面的操作之后，还需要在手机上打开开发者模式，这样就能通过打包完成后的文件夹中的批处理命令将游戏安装到手机上：
![](/article_img/2023-09-12-14-03-24.png)

手机的具体设置UE官方文档写的很详细：[设置Android设备](https://docs.unrealengine.com/5.2/zh-CN/setting-up-your-android-device-for-developing-applications-in-unreal-engine/)，这里记录一下我遇到问题，我准备使用退役的小米8（没有SD卡）当作测试用机，在安装时会报如下错误：
![](/article_img/2023-09-12-14-07-43.png)

这是由于MIUI禁止了在没有安装SD卡时通过USB安装程序，需要先插上SD卡，打开 **USB安装**，之后再拔掉SD卡，之后不需要SD卡也可以正常安装了：
![](/article_img/2023-09-12-14-11-00.png)

当手机通过USB连接在电脑上并且设置好上面的各种配置后，可以选择直接在Android设备上启动，如果没有打开USB安装选项也会报如下错误：
![](/article_img/2023-09-11-21-55-11.png)

启动成功！
![](/article_img/2023-09-12-14-22-38.png)

# 参考

[UE文档-分享和发布项目-Android-开始入门](https://docs.unrealengine.com/5.2/zh-CN/getting-started-and-setup-for-android-projects-in-unreal-engine/)
[Windows10 UE4.27 新手向打包安卓项目流程整理](https://zhuanlan.zhihu.com/p/562504560)
[UE4学习笔记（1）：UE源码下载编译+安卓打包](https://zhuanlan.zhihu.com/p/655375421)
[《UE4开发笔记》Tip 1 编译完全指南](https://zhuanlan.zhihu.com/p/509308558)