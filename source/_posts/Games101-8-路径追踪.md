---
title: Games101-8-路径追踪
date: 2022-10-13
index_img: "/img/bg/cg_bg.png"
tags: [Games101]
categories: 
   -[Games101笔记]
---

Games101-8-路径追踪
<!-- more -->

![](/article_img/2022-10-12-19-16-44.png)

# 概率论知识

![](/article_img/2022-10-12-19-17-41.png)

# Monte Carlo Integration

一种数值方法计算定积分的值，不需要算出原函数。

![](/article_img/2022-10-12-19-20-11.png)

![](/article_img/2022-10-12-19-20-41.png)

![](/article_img/2022-10-12-19-18-35.png)

# 仅考虑直接光照

![](/article_img/2022-10-13-14-37-13.png)

这里的wi是随机在半球上采样

# 引入间接光照

![](/article_img/2022-10-13-14-40-09.png)

直接光照就按照光源来计算，间接光照递归计算

但是还没完：

1. 光线会爆炸
   ![](/article_img/2022-10-13-14-41-58.png)
   
   因此随机选择一条路径进行采样

   ![](/article_img/2022-10-13-14-42-30.png)

   在一个像素中增加采样点，最后平均，来降低噪声

   ![](/article_img/2022-10-13-14-42-38.png)

2. 没有递归出口

   **俄罗斯轮盘赌**

   这里就是利用了**期望**的概念：  
   事先假定一个概率p，有p的概率发生反射，并且计算的结果除以p，有（1-p）的概率不发生反射，得到0；这种方法得到的结果的期望仍然是Lo，即能量没有损失。

   ![](/article_img/2022-10-13-14-45-56.png)

   ![](/article_img/2022-10-13-14-50-16.png)

# 提高路径追踪效率

![](/article_img/2022-10-13-14-51-20.png)

SPP（samples per pixel）

![](/article_img/2022-10-13-15-58-06.png)

思考为什么low SPP效果不好会出现噪点？

因为我们的采样时在整个半球上随机采样的，而俄罗斯轮盘赌会终结一些光线，导致路径还没有打到光源就被终止，这条路径返回了空值，相当于这次路径追踪白做了（光线被浪费掉了），本质就是采样不足导致的。

因此我们要么加大采样量（增大路径到达光源的概率），要么尽量不要浪费光线，也就是进行重要性采样，对光源采样，就是先计算对该点颜色贡献最大的直接光照。

在对光源采样时，要把光源平面投影到半球面上，也可以理解为求**有效光源**(dA*costheta)的立体角。

![](/article_img/2022-10-14-12-42-03.png)

之后就可以计算Lo

![](/article_img/2022-10-14-12-45-56.png)

现在我们的策略是先计算直接光照（不使用俄罗斯轮盘赌），再计算间接光照（随机在半球中采样，使用俄罗斯轮盘赌）

![](/article_img/2022-10-14-12-49-41.png)

再加上判断直接光照是否被遮挡

伪代码：
```
shade(p,wo)
   # 来自光源的贡献（直接光照）
   均匀对光源采样 x' (pdf_light = 1/A)
   Shoot a ray from p to x'
   if the ray is not blocked in the middle
      L_dir = L_i * f_r * cos theta' / |x'-p|^2 / pdf_light

   # 来自反射的贡献（间接光照）
   L_indir = 0.0
   判断俄罗斯轮盘赌是否终止
   在半球上均匀采样 wi （pdf_hemi = 1/2pi）
   Trace a ray r(p, wi)
   if ray r hit a non-emitting object at q
      L_indir = shade(q, -wi) * f_r * cos theta / pdf_hemi / P_RR
   
   Return L_dir + L_indir

```

# 参考

[课程视频](https://www.bilibili.com/video/BV1X7411F744?p=16&vd_source=93b215eab72b2548f75d0772e28f8b20)  
[GAMES101: 现代计算机图形学入门](https://sites.cs.ucsb.edu/~lingqi/teaching/games101.html)  
[GAMES101_Lecture_15.pdf](https://sites.cs.ucsb.edu/~lingqi/teaching/resources/GAMES101_Lecture_15.pdf)  
[GAMES101_Lecture_16.pdf](https://sites.cs.ucsb.edu/~lingqi/teaching/resources/GAMES101_Lecture_16.pdf)  
