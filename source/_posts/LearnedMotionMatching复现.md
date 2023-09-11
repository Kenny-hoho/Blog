---
title: LearnedMotionMactching复现
date: 2023-03-25
index_img: "/img/bg/LearnedMM.png"
tags: [计算机角色动画]
categories: 
   -[笔记]
---

LearnedMotionMactching复现
<!-- more -->

在尝试跑LearnedMotionMatching代码的时候出现了很多问题，也是自己平时对编译源码不熟悉导致的，折腾了几天终于把代码跑起来了，记录一下遇到的问题和解决方法。
![](/article_img/2023-03-25-14-47-55.png)

# 相关链接

[github: Learned Motion-Matching](https://github.com/orangeduck/Motion-Matching)
[article: Code vs Data Driven Displacement](https://theorangeduck.com/page/code-vs-data-driven-displacement)
[paper(siggraph2020): Learned Motion Matching](https://theorangeduck.com/page/learned-motion-matching)


# 前置依赖

这个演示Demo是基于raylib和raygui的（在国内很小众，是两个C++库用来开发一些小游戏），需要提前安装这两个库，建议都用github最新源码自行编译，否则会出现一些版本问题。

[raylib](https://github.com/raysan5/raylib)
[raygui](https://github.com/raysan5/raygui)

## raylib安装

下载好源码之后，创建一个raylib_build文件夹存放用CMake构建出的结果，之后在raylib_build中找到raylib.sln（vs项目文件），点击打开，生成解决方案，生成完毕之后会在/raylib_build/raylib/路径下生成一个Debug文件夹，其中存放raylib.lib文件。这时raylib就已经安装完成了，但是我们还需要在vs2022中配置raylib库。

![](/article_img/2023-03-25-15-11-00.png)

可以新建两个文件夹用来管理第三方库，Include文件夹存放.h文件，Libs文件夹存放.lib文件，这样之后就不需要去费力的寻找这些库文件了。

## raygui安装

raygui就简单多了，只需要从github下载源码即可，不需要自己编译，可以在src文件夹中找到raygui.h文件，将其放入上面我们建好的Include文件夹中即可。

## vs2022配置第三方库

![](/article_img/2023-03-25-15-16-38.png)

这里的包含目录直接填我们新建的Include文件夹，库目录填Libs文件夹即可。

感谢ChatGPT！

# 编译源码


## 作者提供的方式

按照github上的[Issue](https://github.com/orangeduck/Motion-Matching/issues/2)，其实这种方法理论上不需要编译raylib源码，直接去raylib官网下载Installer即可一键安装：

![](/article_img/2023-03-25-15-20-53.png)

但是我在实际运行时出现了下图这样的错误，提示说UpdateCamera()缺少一个参数。

![](/article_img/2023-03-25-15-24-46.png)

![](/article_img/2023-03-25-15-26-06.png)

按照ChatGPT的回答，可以确定应该还是依赖库的版本问题，我这里就直接改了源码，将第二个参数设置为了CAMERA_CUSTOM，即可正常编译，生成controller.exe文件。

## VS2022编译方式

考虑到之后可能会对源码进行修改，就尝试在VS里编译源码，我首先在VS中配置好了raylib和raygui，之后将源码中的文件全部添加到VS项目中，这是发现我的编译器似乎并不能识别部分源码的写法，如下：

![](/article_img/2023-03-25-15-32-24.png)

这里将括号去掉就正常了。

![](/article_img/2023-03-25-15-33-19.png)

这里写成下面的__restrict就可以识别了。

![](/article_img/2023-03-25-15-36-39.png)

这里和上述一样随便加了一个参数。

这些写法问题解决之后，还不要忘了把源码中的resouce文件夹放到VS项目文件夹里，这样才能加载模型和数据库等等资源。
