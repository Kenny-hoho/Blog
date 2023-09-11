---
title: learnopengl-2-着色器
date: 2022-10-28
index_img: "/img/bg/opengl.jpg"
tags: [OpenGL]
categories: 
   -[OpenGL笔记]
---

learnopengl-2-着色器
<!-- more -->

[着色器-LearnOpenGL CN](https://learnopengl-cn.github.io/01%20Getting%20started/05%20Shaders/)

# GLSL

着色器是使用一种叫GLSL的类C语言写成的。GLSL是为图形计算量身定制的，它包含一些针对向量和矩阵操作的有用特性。

Shader的典型结构

```C++
#version version_number
in type in_variable_name;
in type in_variable_name;

out type out_variable_name;

uniform type uniform_name;

int main()
{
  // 处理输入并进行一些图形操作
  ...
  // 输出处理过的结果到输出变量
  out_variable_name = weird_stuff_we_processed;
}
```

# 数据类型

GLSL中包含C等其它语言大部分的默认基础数据类型：int、float、double、uint和bool。

还有两种容器类型，分别是向量（Vector）和矩阵（Matrix）

## 向量

| 类型  |             含义              |
| :---: | :---------------------------: |
| vecn  |  包含n个float分量的默认向量   |
| bvecn |     包含n个bool分量的向量     |
| ivecn |   包含n个int分量的默认向量    |
| uvecn | 包含n个unsigned int分量的向量 |
| dvecn |  包含n个double分量的默认向量  |

向量的使用十分灵活，如下所示：
```C++
vec2 someVec;
vec4 differentVec = someVec.xyxx;
vec3 anotherVec = differentVec.zyw;
vec4 otherVec = someVec.xxxx + anotherVec.yxzy;

vec2 vect = vec2(0.5, 0.7);
vec4 result = vec4(vect, 0.0, 0.0);
vec4 otherResult = vec4(result.xyz, 1.0);
```

# 输入与输出

GLSL定义了in和out关键字专门来实现这个目的。每个着色器使用这两个关键字设定输入和输出，只要一个输出变量与下一个着色器阶段的输入匹配，它就会传递下去。但有两个特例：顶点着色器和片段着色器。

## 顶点着色器

顶点着色器需要一种特殊的输入，从顶点数据中直接接收输入。使用**location**这一元数据指定输入变量

```C++
#version 330 core
layout (location = 0) in vec3 aPos; // 位置变量的属性位置值为0

out vec4 vertexColor; // 为片段着色器指定一个颜色输出

void main()
{
    gl_Position = vec4(aPos, 1.0); // 注意我们如何把一个vec3作为vec4的构造器的参数
    vertexColor = vec4(0.5, 0.0, 0.0, 1.0); // 把输出变量设置为暗红色
}
```

## 片段着色器

片段着色器需要一个vec4颜色输出变量，因为片段着色器需要生成一个最终输出的颜色。

```C++
#version 330 core
out vec4 FragColor;

in vec4 vertexColor; // 从顶点着色器传来的输入变量（名称相同、类型相同）

void main()
{
    FragColor = vertexColor;
}
```

# Uniform

Uniform是一种从CPU中的应用向GPU中的着色器发送数据的方式, 具体使用时就是在shader（GPU）中定义uniform变量，该变量可以在main函数（CPU）中被赋值和改变。

uniform是全局的(Global)，必须在每个着色器中都是独一无二的。

在程序中改变uniform的值需要先找到着色器中uniform属性的位置，用**glGetUniformLocation**函数查询，使用方法如下：

```C++
int vertexColorLocation = glGetUniformLocation(shader_name, "uniform_name");
```

之后使用**glUniform**函数改变uniform的值：
```C++
glUniform4f(vertexColorLocation, 0.0f, greenValue, 0.0f, 1.0f);
```
![](/article_img/2022-10-28-17-09-17.png)

# 自己的着色器类

定义一个着色器类实现从硬盘读取着色器，然后编译并链接它们，并对它们进行错误检测。

## 从文件读取shader

之前我们都使用硬编码shader代码的方式使用shader，在这里尝试从外部文件中读取shader代码。

![](/article_img/2022-10-29-12-19-29.png)

# 练习

1. 修改顶点着色器让三角形上下颠倒：
    ```C++
    #version 330 core
    layout (location = 0) in vec3 aPos;
    layout (location = 1) in vec3 aColor;

    out vec3 ourColor;

    void main()
    {
        gl_Position = vec4(aPos.x, -aPos.y, aPos.z, 1.0); // 将y坐标取反即可
        ourColor = ourColor;
    }
    ```
    ![](/article_img/2022-10-28-19-52-39.png)
2. 使用uniform定义一个水平偏移量，在顶点着色器中使用这个偏移量把三角形移动到屏幕右侧：
    ```C++
    #version 330 core
    layout (location = 0) in vec3 aPos;
    layout (location = 1) in vec3 aColor;

    out vec3 ourColor;
    uniform float hori_offset; // 在程序中修改即可

    void main()
    {
        gl_Position = vec4(aPos.x-hori_offset, aPos.y, aPos.z, 1.0);
        ourColor = ourColor;
    }
    ```
    ![](/article_img/2022-10-28-19-53-14.png)
3. 使用out关键字把顶点位置输出到片段着色器，并将片段的颜色设置为与顶点位置相等（来看看连顶点位置值都在三角形中被插值的结果）。做完这些后，尝试回答下面的问题：为什么在三角形的左下角是黑的?：
    ```C++
    #version 330 core
    layout (location = 0) in vec3 aPos;
    layout (location = 1) in vec3 aColor;

    void main()
    {
        gl_Position = vec4(aPos.x, aPos.y, aPos.z, 1.0);
        ourColor = aPos;
    }
    ```
    ![](/article_img/2022-10-28-19-11-33.png)
    左下角是黑色是因为左下角的坐标是（-0.5f, -0.5f, 0.0f）是负值，而负值在颜色计算时会转换成0，并且插值后的结果也仍然是负值，故直到三角形中心才开始有颜色的渐变。
