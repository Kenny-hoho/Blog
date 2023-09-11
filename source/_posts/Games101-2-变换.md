---
title: Games101-2-变换
date: 2022-07-21 
index_img: "/img/bg/cg_bg.png"
tags: [Games101]
categories: 
   -[Games101笔记]
---

Games101-2-变换， 对应Games101第三、四节课
<!-- more -->

# Games101现代计算机图形学入门-3

![](/article_img/2022-10-15-15-38-04.png)

# 线性变换

## 缩放

## 切变
![](/article_img/2022-08-01-18-08-53.png)

## 旋转
![](/article_img/2022-08-01-18-15-35.png)

旋转-a角度的矩阵就是旋转a角度的转置矩阵，也是逆矩阵

# 仿射变换（齐次坐标）
为了解决平移变换不能简单写成矩阵相乘 **x'= Mx**

![](/article_img/2022-08-01-18-23-48.png)

向量具有**平移不变性**，因此向量最后是0

![](/article_img/2022-08-01-18-27-54.png)

在齐次坐标下，点+点表示这两点的中点

![](/article_img/2022-08-01-18-34-08.png)

**逆变换对应逆矩阵**

# 变换的组合

![](/article_img/2022-08-01-18-43-06.png)

# 三维变换

![](/article_img/2022-08-01-18-50-39.png)

![](/article_img/2022-08-01-18-53-06.png)

**对这种矩阵来说，是先线性变换后仿射变换**

## 三维旋转

![](/article_img/2022-08-01-20-43-53.png)

基于右手螺旋定则，y由z叉乘x得到，固正好相反

![](/article_img/2022-08-03-10-12-36.png)

### 罗格里德斯旋转公式

![](/article_img/2022-08-03-10-22-45.png)

# 视图变换

确定相机位置

![](/article_img/2022-08-03-10-31-16.png)

![](/article_img/2022-08-03-10-39-27.png)

将任意向量旋转到标准轴困难，故做逆操作，将标准轴转到任意向量，之后对旋转矩阵做逆变换，恰好旋转矩阵是**正交矩阵**，其逆矩阵就是转置矩阵。

# 投影变换

## 正交投影

![](/article_img/2022-08-10-16-12-45.png)

## 透视投影

![](/article_img/2022-08-03-17-35-43.png)

推导思路：先将Frustum挤压成一个长方体，之后再做一次正交投影

推导过程：

![](/article_img/2022-10-27-14-53-37.png)

在unity等游戏引擎中，需要在做完投影矩阵变换之后再进行 **透视除法**，才能将顶点变换到 **归一化设备坐标 NDC（Normalized Device Coordinates）** 。而在数学中点（x, y, z, w）和（x/w, y/w, z/w, 1）是完全一样的，我们也就认为进行完投影变换后，变换就结束了。

在实际计算中点（x, y, z, w）和（x/w, y/w, z/w, 1）虽然只是写法不同，但是会影响后续计算，所以需要将所有的（x, y, z, w）进行归一化变为（x/w, y/w, z/w, 1），这一步操作叫做 **透视除法**，在渲染流水线中在顶点着色器输出之后进行。
![](/article_img/2023-07-07-15-30-15.png)

## 视锥

![](/article_img/2022-08-03-20-29-06.png)

1. aspect ratio: 宽高比（观测角度）
2. Field of View(fovY): 可视角度

![](/article_img/2022-08-11-08-56-46.png)

# 总结

![](/article_img/2023-07-08-17-19-44.png)

# homework0

## 代码
```c++
// 给定一个点 P=(2,1), 将该点绕原点先逆时针旋转 45◦，再平移 (1,2), 计算出变换后点的坐标（要求用齐次坐标进行计算）。
// 定义点P
Eigen::Vector3f p(2.0f, 1.0f, 1.0f);
// 旋转矩阵
Eigen::Matrix3f rotation;
rotation <<
   std::cos(45.0 / 180.0 * acos(-1)), -std::sin(45.0 / 180.0 * acos(-1)), 0,
   std::sin(45.0 / 180.0 * acos(-1)), std::cos(45.0 / 180.0 * acos(-1)), 0,
   0, 0, 1;
// 平移矩阵
Eigen::Matrix3f trans;
trans <<
   1, 0, 1,
   0, 1, 2,
   0, 0, 1;
// 变换
std::cout << trans * rotation * p;
```
重点关注点，向量和矩阵定义的方法

# homework1

## 代码
```c++
// get_model_matrix(float rotation_angle)

   Eigen::Matrix4f model = Eigen::Matrix4f::Identity();

   float angle = rotation_angle / 180.0f * 3.14;
   float c = cosf(angle);
   float s = sinf(angle);
   Eigen::Matrix4f rotation;
   rotation <<
      c, -s, 0, 0,
      s, c, 0, 0,
      0, 0, 1, 0,
      0, 0, 0, 1;
   model = model * rotation;


// get_projection_matrix(float eye_fov, float aspect_ratio, float zNear, float zFar)

   float n, f, t, b, r, l;
   t = abs(zNear) * tanf((eye_fov * 3.14 / 180.0) / 2);
   r = aspect_ratio * t;
   b = -t;
   l = -r;
   n = -zNear;
   f = -zFar;

   Eigen::Matrix4f projection = Eigen::Matrix4f::Identity();

   Eigen::Matrix4f trans, scale, p2o, prep;
   trans <<
      1, 0, 0, -(r + l) / 2,
      0, 1, 0, -(t + b) / 2,
      0, 0, 1, -(n + f) / 2,
      0, 0, 0, 1;
   scale <<
      2 / (r - l), 0, 0, 0,
      0, 2 / (t - b), 0, 0,
      0, 0, 2 / (n - f), 0,
      0, 0, 0, 1;
   p2o <<
      n, 0, 0, 0,
      0, n, 0, 0,
      0, 0, n + f, -(n * f),
      0, 0, 1, 0;

   projection =  trans * scale * p2o * projection;
```
运行结果：
![](/article_img/2022-08-11-11-15-21.png)
注意：
1. 角度要进行转换
2. 一开始绘制的三角形是倒着的，是因为默认zNear和zFar是正值，但实际上其均为负值，故对其取负值。


# 参考

[课程视频](https://www.bilibili.com/video/BV1X7411F744?p=2&vd_source=93b215eab72b2548f75d0772e28f8b20)  
[课程网址](https://sites.cs.ucsb.edu/~lingqi/teaching/games101.html)  
[GAMES101_Lecture_03.pdf](https://sites.cs.ucsb.edu/~lingqi/teaching/resources/GAMES101_Lecture_03.pdf)  
[GAMES101_Lecture_04.pdf](https://sites.cs.ucsb.edu/~lingqi/teaching/resources/GAMES101_Lecture_04.pdf)