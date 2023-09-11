---
title: Games101-7-辐射度量学
date: 2022-10-12
index_img: "/img/bg/cg_bg.png"
tags: [Games101]
categories: 
   -[Games101笔记]
---

Games101-7-辐射度量学
<!-- more -->

# 基础物理概念

![](/article_img/2022-10-12-12-03-15.png)

## Radiant Energy and Flux(Power)

![](/article_img/2022-10-12-11-59-26.png)

Flux（Power） : energy per unit time（单位时间的能量）

可以理解为某一时刻的能量

Flux的单位Lumen（流明）

## Radiant Intensity

某一个方向上的亮度（能量）

![](/article_img/2022-10-12-12-04-58.png)

### 立体角（Solid Angles）

![](/article_img/2022-10-12-12-08-22.png)

![](/article_img/2022-10-12-12-12-57.png)

## Irradiance

![](/article_img/2022-10-12-12-28-37.png)

这个面积是要做投影，就是要计算垂直角度上的能量，类似于Lambert's Cosine law：

![](/article_img/2022-10-12-12-30-45.png)

正确理解Falloff

![](/article_img/2022-10-12-12-34-50.png)

越远球壳面积越大，能量（Flux/Power）总体不变，单位面积上的能量——Irradiance就越小。

## Radiance

![](/article_img/2022-10-12-12-38-07.png)

某一个面积向某一个方向辐射多少能量。

![](/article_img/2022-10-12-12-40-29.png)

就是把Irradiance（单位面积上总共收到多少能量）分解到各个方向（从某一个方向收到了多少能量）。

![](/article_img/2022-10-12-12-46-03.png)

# BRDF

双向反射分布函数 (Bidirectional ReflectanceDistribution Function）

![](/article_img/2022-10-12-12-51-23.png)

从某一方向发射来的radiance会转化为反射点的irradiance，之后这些能量（irradiance）再转化为从另一个方向射出的radiance.

![](/article_img/2022-10-12-12-57-17.png)

BRDF就是定义**向某个方向辐射的能量**（radiance）与**入射点能量**（irradiance）的比例。

# 反射方程

任何一个方向入射的radiance到着色点，经过BRDF会得到向观测方向射出的radiance；将所有的入射radiance积分起来，就能得到最终看到着色点应该是什么样。

![](/article_img/2022-10-12-13-03-11.png)

递归:

![](/article_img/2022-10-12-13-10-52.png)

# 渲染方程

渲染方程 = 自发光 + 反射方程

![](/article_img/2022-10-12-13-12-07.png)


# 参考

[课程视频](https://www.bilibili.com/video/BV1X7411F744?p=14&vd_source=93b215eab72b2548f75d0772e28f8b20)  
[GAMES101: 现代计算机图形学入门](https://sites.cs.ucsb.edu/~lingqi/teaching/games101.html)  
[GAMES101_Lecture_14.pdf](https://sites.cs.ucsb.edu/~lingqi/teaching/resources/GAMES101_Lecture_14.pdf)  
[GAMES101_Lecture_15.pdf](https://sites.cs.ucsb.edu/~lingqi/teaching/resources/GAMES101_Lecture_15.pdf)  
