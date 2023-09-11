---
title: learnopengl-1-你好三角形
date: 2022-10-27
index_img: "/img/bg/opengl.jpg"
tags: [OpenGL]
categories: 
   -[OpenGL笔记]
---

learnopengl-1-你好三角形
<!-- more -->

[你好，三角形-LearnOpenGL CN](https://learnopengl-cn.github.io/01%20Getting%20started/04%20Hello%20Triangle/#_7)

[傅老師/OpenGL教學 第一章](https://www.bilibili.com/video/BV11W411N7b9?p=10&vd_source=93b215eab72b2548f75d0772e28f8b20)

环境的搭建和窗口的生成有时间之后补上。

本篇笔记先按照网站的顺序梳理一边基本概念，之后结合傅老师的讲解，**串联**所有概念和步骤。

# OpenGL的图形渲染管线（Graphics Pipline）

![](/article_img/2022-10-24-14-28-44.png)

图形渲染管线可以被划分为两个主要部分：第一部分把你的3D坐标转换为2D坐标，第二部分是把2D坐标转变为实际的有颜色的像素。

1. 顶点着色器（Vertex Shader）：顶点着色器主要的目的是把3D坐标转为另一种3D坐标；
2. 图元装配（Primitive Assembly）：将顶点着色器输出的所有顶点装配成指定图元的形状；
3. 几何着色器（Geometry Shader）：几何着色器把图元形式的一系列顶点的集合作为输入，它可以通过产生新顶点构造出新的（或是其它的）图元来生成其他形状；
4. 光栅化（Rasterization Stage）：把图元映射为最终屏幕上相应的像素，生成供片段着色器(Fragment Shader)使用的片段(Fragment)；
5. 片段着色器（Fragment Shader）：计算一个像素的最终颜色；
6. 测试和混合（Test and Blending）：检测深度决定哪些片段被舍弃，以及检查alpha值对物体进行混合；

# 顶点输入

![](/article_img/2022-10-24-14-47-45.png)

定义这样的顶点数据以后，我们会把它作为输入发送给图形渲染管线的第一个处理阶段：顶点着色器。

## VBO（顶点缓冲对象）

VBO用来储存输入的一大堆顶点数据，可以一次性将一大批数据发送到显卡上。

以下代码就是VBO的基本定义和使用方法。
```C++
unsigned int VBO;
glGenBuffers(1, &VBO); // 生成
glBindBudder(GL_ARRAY_BUFFER, VBO);  // 绑定到状态机
glBufferData(GL_ARRAY_BUFFER, sizeof(vertices), vertices, GL_STATIC_DRAW);  // 加数据
```

## EBO（元素缓冲对象）

这个本来是在教程最后讲解的，但是EBO也是对输入的顶点数据进行处理的对象。

EBO简单来说就是用来解决顶点重复定义的问题。

假如现在要通过绘制两个三角形的方法绘制一个矩形，如果没有EBO，需要定义如下顶点集合：
```c++
float vertices[] = {
    // 第一个三角形
    0.5f, 0.5f, 0.0f,   // 右上角
    0.5f, -0.5f, 0.0f,  // 右下角
    -0.5f, 0.5f, 0.0f,  // 左上角
    // 第二个三角形
    0.5f, -0.5f, 0.0f,  // 右下角
    -0.5f, -0.5f, 0.0f, // 左下角
    -0.5f, 0.5f, 0.0f   // 左上角
};
```
可以看到有一些相同的顶点被重复定义了，而EBO是一个缓冲区，就像一个顶点缓冲区对象一样，它存储 OpenGL 用来决定要绘制哪些顶点的索引。
```c++
float vertices[] = {
    0.5f, 0.5f, 0.0f,   // 右上角
    0.5f, -0.5f, 0.0f,  // 右下角
    -0.5f, -0.5f, 0.0f, // 左下角
    -0.5f, 0.5f, 0.0f   // 左上角
};

unsigned int indices[] = {
    // 注意索引从0开始! 
    // 此例的索引(0,1,2,3)就是顶点数组vertices的下标，
    // 这样可以由下标代表顶点组合成矩形

    0, 1, 3, // 第一个三角形
    1, 2, 3  // 第二个三角形
};
```
之后EBO的使用也和VBO类似，之后在渲染循环中，将glDrawArrays()替换成glDrawElements()
```C++
unsigned int EBO;
glGenBuffers(1, &EBO); // 生成
glBindBudder(GL_ELEMENT_ARRAY_BUFFER, EBO);  // 绑定到状态机
glBufferData(GL_ELEMENT_ARRAY_BUFFER, sizeof(indices), indices, GL_STATIC_DRAW);  // 加数据
/*
...
*/
// 渲染循环中
glBindBuffer(GL_ELEMENT_ARRAY_BUFFER, EBO);
glDrawElements(GL_TRIANGLES, 6, GL_UNSIGNED_INT, 0);
```

# 着色器

## 顶点着色器
shader语法和编写在下一节具体展开，这里只简单介绍shader如何配置。

和之前的VBO和EBO类似，也需要先定义一个shader编号，之后生成shader，告诉生成的shader源码是哪些，最后编译shader

```c++
unsigned int vertexShader;
vertexShader = glCreateShader(GL_VERTEX_SHADER);

glShaderSource(vertexShader, 1, &vertexShaderSource, NULL);
glCompileShader(vertexShader);
```

## 片段着色器

和顶点着色器完全一样的用法
```c++
unsigned int fragmentShader;
fragmentShader = glCreateShader(GL_FRAGMENT_SHADER);

glShaderSource(fragmentShader, 1, &fragmentShaderSource, NULL);
glCompileShader(fragmentShader);
```

## 着色器程序对象

之前编译好的两个着色器并不能直接使用，需要将其合并（代码上体现出来就是将之前两个shader附加到着色器程序上），之后链接成一个着色器程序对象，在渲染时激活这个着色器程序。

```c++
unsigned int shaderProgram;
shaderProgram = glCreateProgram(); // 经典创建对象

// 附加（合并）shader到shaderProgram上
glAttachShader(shaderProgram, vertexShader);
glAttachShader(shaderProgram, fragmentShader);
glLinkProgram(shaderProgram); // 链接shader

// 渲染循环中
glUseProgram(shaderProgram);
```

在把着色器对象链接到程序对象之后，就可以删除着色器对象了

# 链接顶点属性

这部分在第一次读的时候较难理解，之后结合[傅老師/OpenGL教學 第一章](https://www.bilibili.com/video/BV11W411N7b9?p=10&vd_source=93b215eab72b2548f75d0772e28f8b20)才理解。

我们之前定义了VBO来将一堆顶点数据输入GPU，但是GPU其实并不知道这些顶点数据的含义，glVertexAttribPointer函数就是为了告知GPU如何正确理解这些“杂乱”的顶点信息的。


```c++
glVertexAttribPointer(0, 3, GL_FLOAT, GL_FALSE, 3 * sizeof(float), (void*)0);
glEnableVertexAttribArray(0);
```

# 顶点数组对象(VAO)

VAO使得当配置顶点属性指针时，你只需要将那些调用执行一次，之后再绘制物体的时候只需要绑定相应的VAO就行了。

![](/article_img/2022-10-27-14-16-43.png)

![](/article_img/2022-10-27-14-21-46.png)

由以上图可以看出VAO包含很多attribute pointer和一个EBO，且可以看出真正储存顶点信息的是一个个的VBO，VAO中存储着如何去解释VBO中数据的顶点属性指针和EBO。

## 深入理解VAO和VBO

[如何正确理解OpenGL的VAO?](https://www.zhihu.com/question/30095978/answer/2362612406)

可以使用一个VAO绑定多个VBO但是要**注意**：  
多个VBO中的顶点信息不能重复，如VBO1中存储所有的顶点位置信息，VBO2中存储所有的颜色信息，不能VBO1和VBO2都存储位置信息。

如果混合存储，如下图，绑定VBO2的时候会覆盖掉之前绑定的VBO1；
```c++
float triangle[] = { // VBO1的data
   // 位置              // 颜色
   0.5f, -0.5f, 0.0f,  1.0f, 0.0f, 0.0f,   // 右下
   -0.5f, -0.5f, 0.0f,  0.0f, 1.0f, 0.0f,   // 左下
   0.0f,  0.5f, 0.0f,  0.0f, 0.0f, 1.0f    // 顶部
};
float triangle2[] = { // VBO2的data
   // 位置              // 颜色
   0.5f, -0.5f, 0.0f,  1.0f, 0.0f, 0.0f, 
   0.0f,  0.5f, 0.0f,  0.0f, 0.0f, 1.0f, 
   0.5f, 0.5f, 0.0f,  0.0f, 1.0f, 0.0f   
};
```
![](/article_img/2022-10-28-19-32-16.png)


将顶点信息和颜色信息分别存入到VBO1和VBO2中，再将VBO1和VBO2绑定在VAO上就可以正常显示两个三角形；

```c++
float triangles[] = {  // VBO1的data
   0.5f, -0.5f, 0.0f,
   -0.5f, -0.5f, 0.0f,
   0.0f,  0.5f, 0.0f,
   0.5f, -0.5f, 0.0f,
   0.0f,  0.5f, 0.0f,
   0.5f, 0.5f, 0.0f
};
float colors[] = {   // VBO2的data
   1.0f, 0.0f, 0.0f,
   0.0f, 1.0f, 0.0f,
   0.0f, 0.0f, 1.0f,
   1.0f, 0.0f, 0.0f,
   0.0f, 0.0f, 1.0f,
   0.0f, 1.0f, 0.0f
};
```

![](/article_img/2022-10-28-19-37-42.png)

要先绑定VAO，再绑定EBO，否则会出错。

```c++
unsigned int VAO;
glGenVertexArrays(1, &VAO);
glBindVertexArray(VAO);
```

# 总结

![](/article_img/2022-10-27-15-12-46.png)

首先要理解OpenGL采用状态机架构，所以绑定就意味着使用。

![](/article_img/2022-10-27-14-53-13.png)

# 练习

1. 添加更多顶点到数据中，使用glDrawArrays，尝试绘制两个彼此相连的三角形
   ![](/article_img/2022-10-27-15-32-33.png)

   关键代码：
   ```c++
   float triangle[] = {
      -1.0f, -0.5f, 0.0f,
      0.0f, -0.5f, 0.0f,
      -0.5f, 0.5f, 0.0f,
      0.0f, -0.5f, 0.0f,
      1.0f, -0.5f, 0.0f,
      0.5f, 0.5f, 0.0f
   };

   // 渲染循环
   glDrawArrays(GL_TRIANGLES, 0, 6);
   ```
2. 创建相同的两个三角形，但对它们的数据使用不同的VAO和VBO
   ![](/article_img/2022-10-27-15-54-08.png)

   关键代码：  
   这里要注意要分别为两个VAO设置顶点属性，两个VAO该怎样处理VBO中的数据都要分别告知。
   ```c++
   unsigned int VAO[2], VBO[2];
   glGenBuffers(2, VBO);
   glGenVertexArrays(2, VAO);

   glBindVertexArray(VAO[0]);
   glBindBuffer(GL_ARRAY_BUFFER, VBO[0]);
   glBufferData(GL_ARRAY_BUFFER, sizeof(triangle1), triangle1, GL_STATIC_DRAW);
   // 为VAO[0]设置顶点属性
   glVertexAttribPointer(0, 3, GL_FLOAT, GL_FALSE, 3 * sizeof(float), (void*)0);
   glEnableVertexAttribArray(0);


   glBindVertexArray(VAO[1]);
   glBindBuffer(GL_ARRAY_BUFFER, VBO[1]);
   glBufferData(GL_ARRAY_BUFFER, sizeof(triangle2), triangle2, GL_STATIC_DRAW);
   // 为VAO[1]设置顶点属性
   glVertexAttribPointer(0, 3, GL_FLOAT, GL_FALSE, 3 * sizeof(float), (void*)0);
   glEnableVertexAttribArray(0);

   // 渲染循环
   glBindVertexArray(VAO[0]);
   glDrawArrays(GL_TRIANGLES, 0, 3);

   glBindVertexArray(VAO[1]);
   glDrawArrays(GL_TRIANGLES, 0, 3);
   ```
3. 创建两个着色器程序，第二个程序使用一个不同的片段着色器，输出黄色；再次绘制这两个三角形，让其中一个输出为黄色
   ![](/article_img/2022-10-27-16-10-17.png)
   关键代码：  
   ```c++
   // 渲染循环
   glUseProgram(shaderProgram1);
   glBindVertexArray(VAO[0]);
   glDrawArrays(GL_TRIANGLES, 0, 3);

   glUseProgram(shaderProgram2);
   glBindVertexArray(VAO[1]);
   glDrawArrays(GL_TRIANGLES, 0, 3);
   ```