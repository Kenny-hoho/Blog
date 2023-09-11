---
title: 3DSlicer配置与编译
date: 2022-06-19 
index_img: "/img/bg/3DSlicerLogo.png"
tags: [3DSlicer]
categories: 
   -[3DSlicer]
---

3DSlicer配置与编译
<!-- more -->

# 3DSlicer配置与编译
## 1.安装编译所需工具
参考[官方文档](https://slicer.readthedocs.io/en/latest/developer_guide/build_instructions/windows.html)和[CSDN文档](https://blog.csdn.net/yaoxingdong/article/details/108051384)

![](/article_img/2022-05-30-18-51-12.png)
### 个人版本:  
CMake 3.23.2  
Git 2.26.2  
VS 2022 (最好安装所有与V143和CMake相关的包，防止编译出错)  
Qt 5.15.2 (Qt必须5.15以上版本，但Qt6取消了Qt Script，没有尝试是否有影响)  
NSIS  

Qt 5.15开始官方取消离线安装包，需要用
[在线下载工具](http://download.qt.io/official_releases/online_installers/)

第一次登录时注意选择个人用户，否则会自动下载商业版。

## 2. 按照官方文档开始编译
可能会报错

![](/article_img/2022-05-30-19-01-47.png)

![](/article_img/2022-05-30-19-02-18.png)

![](/article_img/2022-05-30-19-03-35.png)

将箭头所指处换成\git\Git\usr\bin\patch.exe  **git安装位置因人而异**， 之后继续点击**Configure**直到方框中红色项全部变为白色，之后后点击**generate**生成。

之后点击**Open Project**，将**All_BUILD**设为启动项目。再点击生成，之后可能要等待非常长的时间，生成的文件也会很大，大约50G。

![](/article_img/2022-05-30-19-12-19.png)

很幸运没什么问题生成完成了，如果之后再次生成有什么问题可以参考开头的博客（
![](/article_img/2022-05-31-14-18-58.png)

## 3. 打开文件

编译全部成功之后，会在 \S4D\Slicer-build 目录下生成一个 Slicer.exe 的可执行文件，双击即可打开3D-slicer 程序。

![](/article_img/2022-05-31-15-48-06.png)

这只是一个启动程序，用来配置环境变量之类的，看到下图等一会就会启动3DSlicer软件了。

![](/article_img/2022-05-31-15-45-30.png)

3Dslicer启动之后这个窗口也不能关闭

![](/article_img/2022-05-31-15-48-55.png)

**标志着我们编译成功了！！！**

## 4. 进行测试（可能可以跳过）

根据官方文档进行操作

![](/article_img/2022-05-31-15-59-31.png)

打开CMD，切换到 \S4D\Slicer-build 目录，运行**Slicer.exe --VisualStudioProject**命令，之后会启动VS 2022

文档中的Select build configuration指：
![](/article_img/2022-05-31-16-12-50.png)

开始生成之后，会自动执行600多个测试，期间会不断跳出各种测试窗口，不用理会（这个过程也很长，不做好像也没什么。。）

![](/article_img/2022-05-31-16-22-07.png)

可能会报错，先点击重试，会自动跳过（也不一定用到报错的这个功能）

![](/article_img/2022-05-31-16-23-44.png)

**之后就可以正常使用了**











