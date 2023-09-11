---
title: Games101-9-外观和材质
date: 2022-10-18
index_img: "/img/bg/cg_bg.png"
tags: [Games101]
categories: 
   -[Games101笔记]
---

Games101-9-外观和材质
<!-- more -->

![](/article_img/2022-10-19-14-13-14.png)

# 什么是材质

**材质==BRDF**

## 漫反射材质

![](/article_img/2022-10-19-14-26-47.png)

![](/article_img/2022-10-19-14-21-28.png)

漫反射认为是能量均匀的向四面发射，根据能量守恒：**Lo == Li**，因此漫反射的BRDF为 1/pi，考虑到能量吸收，则乘以ρ（albedo颜色衰减率），就会反射出不同的颜色。

## Glossy材质

![](/article_img/2022-10-19-14-27-00.png)

## 理想反射/折射材质

![](/article_img/2022-10-19-14-27-57.png)

**完美镜面反射**

![](/article_img/2022-10-19-14-31-26.png)

# Snell's Law

![](/article_img/2022-10-19-14-33-18.png)

![](/article_img/2022-10-19-14-33-45.png)

# Fresnel Reflection/Term 菲涅尔项

![](/article_img/2022-10-19-14-36-13.png)

绝缘体的菲涅尔项（玻璃）

![](/article_img/2022-10-19-14-36-58.png)

导体的菲涅尔项（金属）

![](/article_img/2022-10-19-14-37-36.png)

由此可以看出金属会大量反射光线，这也就是为什么古代人用铜镜而不用玻璃照镜子。

![](/article_img/2022-10-19-14-38-56.png)

菲涅尔项的计算十分复杂，所以使用Schlick近似。

# 微表面材质

![](/article_img/2022-10-19-14-48-13.png)

![](/article_img/2022-10-19-14-49-56.png)

![](/article_img/2022-10-19-14-50-38.png)

![](/article_img/2022-10-19-14-51-16.png)

D(h)表示法线分布  
F(i,h)表示菲涅尔项  
G(i,o,h)表示自遮挡

# 各向同性/各向异性材质

![](/article_img/2022-10-20-11-13-23.png)

区分方法：表面是否有方向性

![](/article_img/2022-10-20-11-13-35.png)

## 各向异性材质

![](/article_img/2022-10-20-11-16-05.png)

反射取决于观测的方位角

# BRDF的性质

1. 非负性  
   ![](/article_img/2022-10-20-11-18-34.png)
2. 线性  
   可以线性相加，结果不变
   ![](/article_img/2022-10-20-11-18-53.png)
3. 满足反射定律
   ![](/article_img/2022-10-20-11-19-22.png)
4. 能量守恒  
   该式子表示能量不会增加，只会被反射点吸收或者完全反射
   ![](/article_img/2022-10-20-11-20-06.png)
5. 如果是各向同性材质，BRDF参数可以简化为3个  
   ![](/article_img/2022-10-20-11-23-56.png)

# BRDF的测量

真实材质的BRDF和理论上的有较大差异

![](/article_img/2022-10-20-11-25-41.png)

# 参考

[课程视频](https://www.bilibili.com/video/BV1X7411F744?p=17&vd_source=93b215eab72b2548f75d0772e28f8b20)  
[GAMES101: 现代计算机图形学入门](https://sites.cs.ucsb.edu/~lingqi/teaching/games101.html)  
[GAMES101_Lecture_17.pdf](https://sites.cs.ucsb.edu/~lingqi/teaching/resources/GAMES101_Lecture_17.pdf)  
  

