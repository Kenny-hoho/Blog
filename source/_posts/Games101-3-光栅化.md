---
title: Games101-3-光栅化
date: 2022-08-01 
index_img: "/img/bg/cg_bg.png"
tags: [Games101]
categories: 
   -[Games101笔记]
---

Games101-3-光栅化
<!-- more -->


# 基础定义

![](/article_img/2022-08-03-20-33-50.png)

# 反走样（抗锯齿）

走样现象

![](/article_img/2022-08-26-15-10-45.png)

锯齿的出现一般是由于采样率不足导致的。

![](/article_img/2022-08-26-15-16-08.png)

## 采样

![](/article_img/2022-08-26-15-12-11.png)

采样瑕疵的出现是由于信号变化过快而采样频率太慢导致。

## 反走样方法——先模糊后采样

![](/article_img/2022-08-26-15-13-22.png)

注意：先采样后模糊是错误的，不能达到反走样的效果。

![](/article_img/2022-08-27-09-09-18.png)

模糊操作是通过卷积操作实现的，即取平均。

采样可以看作是频率的重复，走样现象是频率重叠导致，故可以先滤掉高频，再进行采样，也就是先模糊后采样。

![](/article_img/2022-08-27-09-10-34.png)

![](/article_img/2022-08-27-09-10-47.png)

# 反走样的具体方法

![](/article_img/2022-08-27-09-14-04.png)

## MSAA

MSAA就是加大采样率，从而达到抗锯齿的效果。

![](/article_img/2022-08-27-09-15-15.png)

实际应用中，MSAA的采样点一般是不规则的，对一些采样点进行了复用，从而提高效率。

## 其他方法

![](/article_img/2022-08-27-09-17-28.png)

# 可见性/遮挡（Z-buffering）

![](/article_img/2022-08-27-09-21-07.png)

![](/article_img/2022-08-27-09-21-26.png)

![](/article_img/2022-08-27-09-21-42.png)

Z-buffering 时间复杂度为O(n)

# 参考

[课程视频](https://www.bilibili.com/video/BV1X7411F744?p=6&vd_source=93b215eab72b2548f75d0772e28f8b20)  
[课程网址](https://sites.cs.ucsb.edu/~lingqi/teaching/games101.html)  
[GAMES101_Lecture_05.pdf](https://sites.cs.ucsb.edu/~lingqi/teaching/resources/GAMES101_Lecture_05.pdf)  
[GAMES101_Lecture_06.pdf](https://sites.cs.ucsb.edu/~lingqi/teaching/resources/GAMES101_Lecture_06.pdf)  
[GAMES101_Lecture_07.pdf](https://sites.cs.ucsb.edu/~lingqi/teaching/resources/GAMES101_Lecture_07.pdf)
