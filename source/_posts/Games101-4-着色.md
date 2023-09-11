---
title: Games101-4-着色
date: 2022-08-25
index_img: "/img/bg/cg_bg.png"
tags: [Games101]
categories: 
   -[Games101笔记]
---

Games101-4-着色
<!-- more -->
# 着色模型（Blinn-Phong Reflectance Model）

根据感性观察有：  
1. Specular highlights (镜面反射高光)
2. Diffuse reflection (漫反射)
3. Ambient lighting (环境光)

![](/article_img/2022-08-29-15-14-39.png)

## 漫反射

认为漫反射是均匀的向各个方向反射的

![](/article_img/2022-08-29-15-18-27.png)

![](/article_img/2022-08-29-15-18-42.png)

由上图以及地球的季节变化可知，光线直射方向和平面的夹角决定接受光的能量的多少。

![](/article_img/2022-08-29-15-20-23.png)

光的能量也与距离光源的距离有关，与距光源的距离的平方成反比。

![](/article_img/2022-08-29-15-23-14.png)

法线n·入射光l = cos夹角

kd 表示该shading point的颜色（之后texture贴图就是改变kd的值）

**漫反射的着色与观测角度无关** 也就是说无论在哪个方向观测，同一个着色点的颜色总是相同。例如：月球表面

## 镜面反射

![](/article_img/2022-08-29-15-35-09.png)

**镜面反射与观测角度有关** 纯镜面即完全按照镜面反射规律进行反射，若是光滑一些的物体如金属，是在镜面反射出射光线的一个角度范围内可见。

![](/article_img/2022-08-29-15-38-15.png)

h是半程向量，这里的bisector是OpenGL中的函数(平行四边形法则)。指数p就是为了控制出射光线的可见角度范围，一般p为200左右（出射光线的左右5-6度）

![](/article_img/2022-08-29-15-39-32.png)

## 环境光

在Blinn-Phong这个经验模型中，环境光被认为与任何东西都无关，即是一个常量。

将三种光照相加即可得到最终的着色结果。

![](/article_img/2022-08-29-15-42-40.png)

# 着色频率（Shading Frequencies）

![](/article_img/2022-08-29-15-48-55.png)

## flat shading

对每个**三角形**进行着色

每一个三角形有一条法线。

![](/article_img/2022-08-29-15-49-44.png)

## Gouraud shading

对每个**顶点**进行着色

对三角形内部的点，根据三个顶点的颜色进行颜色插值，每一个顶点有一条法线。

![](/article_img/2022-08-29-15-50-45.png)

## Phong shading

对每个**像素**进行着色

对三角形内部的点，根据三个顶点的法线进行法线插值，计算出每一个像素的法线。

![](/article_img/2022-08-29-15-51-54.png)

## 法线定义方法

![](/article_img/2022-08-29-16-17-54.png)

![](/article_img/2022-08-29-16-18-04.png)

利用重心坐标插值

# 重心坐标（Barycentric Coordinates）

![](/article_img/2022-08-29-16-22-31.png)

![](/article_img/2022-08-29-16-23-23.png)

系数均为1/3时，就是三角形的重心。

![](/article_img/2022-08-29-16-24-37.png)

![](/article_img/2022-08-29-16-24-47.png)

插值可以对任何属性进行插值，例如颜色，纹理，法线，深度等等。

重心坐标在投影变换后无法保证依然正确，因此要在投影变换之前进行插值，也就是要用空间中的坐标进行计算。

# 图形（实时渲染）管线

![](/article_img/2022-08-29-16-29-49.png)

# Texture Mapping & 应用纹理

![](/article_img/2022-08-29-16-32-11.png)

![](/article_img/2022-08-29-16-32-22.png)

![](/article_img/2022-08-29-18-30-33.png)

# Texture Magnification (纹理过大过小问题)

## 纹理分辨率过小 

![](/article_img/2022-08-29-18-33-36.png)

Bilinear Interpolation (双线性插值)

![](/article_img/2022-08-29-18-33-20.png)

## 纹理分辨率过大

![](/article_img/2022-08-29-18-35-15.png)

![](/article_img/2022-08-29-18-35-24.png)

问题在于一个像素里包含了很大一片纹理，而仅用一个纹理上的采样点代表这一片纹理。**超采样**（Supersampling）当让可以解决这个问题，但是消耗太大。为了解决这种问题就引入了Mipmap。

## Mipmap

![](/article_img/2022-08-29-18-41-41.png)

![](/article_img/2022-08-29-18-41-51.png)

mipmap的思路是避免采样，直接得到一片区域的平均值，提前生成不同分辨率的图像，之后用这些生成的低分辨率图像代表这一片区域的平均，mipmap多消耗的存储空间是原来的三分之一。

![](/article_img/2022-08-29-18-44-02.png)

![](/article_img/2022-08-29-18-52-05.png)

之后进行trilinear插值，让mipmap的level可以平滑过渡。得到下图的可视化结果。

![](/article_img/2022-08-29-18-53-37.png)

## Mipmap的局限

![](/article_img/2022-08-29-18-54-29.png)

![](/article_img/2022-08-29-18-54-37.png)

由于Mipmap是用一个正方形区域近似真正被像素覆盖的区域，对上图这种长条形的区域就无法取得比较好的近似效果，就会产生过度模糊。

为了避免过度模糊，引入了各向异性过滤。

![](/article_img/2022-08-29-18-56-35.png)

# 环境贴图

![](/article_img/2022-08-29-19-00-41.png)

![](/article_img/2022-08-29-19-00-52.png)

# 凹凸/法线贴图（Bump/normal mapping）

![](/article_img/2022-08-29-19-04-50.png)

![](/article_img/2022-08-29-19-06-16.png)

先求出p点的切线向量，之后旋转九十度就可得到新的法线向量。

![](/article_img/2022-08-29-19-06-26.png)

3d坐标原理类似。

# 参考

[课程视频](https://www.bilibili.com/video/BV1X7411F744?p=9&vd_source=93b215eab72b2548f75d0772e28f8b20)  
[课程网址](https://sites.cs.ucsb.edu/~lingqi/teaching/games101.html)  
[GAMES101_Lecture_07.pdf](https://sites.cs.ucsb.edu/~lingqi/teaching/resources/GAMES101_Lecture_07.pdf)  
[GAMES101_Lecture_08.pdf](https://sites.cs.ucsb.edu/~lingqi/teaching/resources/GAMES101_Lecture_08.pdf)  
[GAMES101_Lecture_09.pdf](https://sites.cs.ucsb.edu/~lingqi/teaching/resources/GAMES101_Lecture_09.pdf)  
[GAMES101_Lecture_10.pdf](https://sites.cs.ucsb.edu/~lingqi/teaching/resources/GAMES101_Lecture_10.pdf)