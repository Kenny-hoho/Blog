---
title: Games101-6-光线追踪(Whitted-Style)
date: 2022-10-11
index_img: "/img/bg/cg_bg.png"
tags: [Games101]
categories: 
   -[Games101笔记]
---

Games101-6-光线追踪(Whitted-Style)
<!-- more -->

# 为什么要用光线追踪

光栅化无法生成效果好的环境光，如软阴影。

光栅化效果一般但是渲染速度很快，光线追踪效果很好但是渲染速度慢。

![](/article_img/2022-10-11-17-42-00.png)

# Whitted-Style 的光线追踪

![](/article_img/2022-10-11-17-42-58.png)

# 光线与表面求交

## 光线方程

![](/article_img/2022-10-11-17-45-24.png)

## 光线与球面求交

![](/article_img/2022-10-11-17-46-35.png)

同理可以推广到光线与所有隐式表面的求交，因为隐式表面有定义其的方程，可以将光线方程带入转化为解方程。

## 光线与三角形求交

### 方法一：先与平面求交

三角形处于一个平面中，首先判断光线是否与该平面相交，再判断是否与三角形相交（交点是否在三角形内部：**叉乘的应用**）。

**平面方程**： 由一个点和一个法相向量定义

![](/article_img/2022-10-11-17-50-28.png)

![](/article_img/2022-10-11-17-54-18.png)

### 方法二：直接利用重心坐标求交

![](/article_img/2022-10-11-17-57-15.png)

Recall：重心坐标由三个点定义，可以表示这三个点所在平面的任意点。当系数都为正的时候表示点在三角形内。

![](/article_img/2022-08-29-16-22-31.png)

# 加速结构

## Bounding Volumes(包围盒)

先判断光线是否与包围盒相交，如果不与包围盒相交那就不可能与物体相交。

![](/article_img/2022-10-11-18-08-40.png)

## Axis-Aligned Bounding Box(AABB)

为了简化计算，包围盒都使用轴对齐包围盒。

![](/article_img/2022-10-11-18-10-27.png)

### 与轴对齐包围盒求交

![](/article_img/2022-10-11-18-12-37.png)

将包围盒想象成三对平面围成的，分别用光线与每对平面求交，得到光线进入这对平面的时间 tmin 和射出这对平面的时间 tmax（负值也可以，这里先将光线看作直线）。

**思考**：  
光线进入包围盒**仅当**光线进入所有三对平面  
光线射出包围盒**直到**光线离开所有三对平面

因此，就是要求最大的 tmin 和最小的 tmax：

![](/article_img/2022-10-11-18-18-49.png)

![](/article_img/2022-10-11-18-19-43.png)

## 包围盒划分

### Uniform Grids
![](/article_img/2022-10-11-18-30-32.png)

![](/article_img/2022-10-11-18-30-38.png)

![](/article_img/2022-10-11-18-32-03.png)

仅仅适用于有大量物体且其均匀分布在整个场景中的情况。

e.g. 当一个场景有大量留白只有中间有一个茶壶的情况，这种划分需要光线与大量不包括任何物体的盒子求交，因此效率很低。

### KD-Tree

![](/article_img/2022-10-11-18-35-53.png)

![](/article_img/2022-10-11-18-36-23.png)

但是这种划分有个致命的缺点：难以求得哪些表面与划分出的包围盒相交，因此引出BVH。

## 物体划分（Bounding Volume Hierarchy）

为了避免统计哪些表面与包围盒相交，直接对表面进行划分，再对划分后的表面求新的包围盒。

![](/article_img/2022-10-11-18-40-01.png)

![](/article_img/2022-10-11-18-40-53.png)

![](/article_img/2022-10-11-18-41-58.png)

# 参考

[课程视频](https://www.bilibili.com/video/BV1X7411F744?p=13&vd_source=93b215eab72b2548f75d0772e28f8b20)  
[课程网址](https://sites.cs.ucsb.edu/~lingqi/teaching/games101.html)  
[GAMES101_Lecture_13.pdf](https://sites.cs.ucsb.edu/~lingqi/teaching/resources/GAMES101_Lecture_13.pdf)  
[GAMES101_Lecture_14.pdf](https://sites.cs.ucsb.edu/~lingqi/teaching/resources/GAMES101_Lecture_14.pdf)  
