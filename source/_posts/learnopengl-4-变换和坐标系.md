---
title: learnopengl-4-变换和坐标系
date: 2022-11-02
index_img: "/img/bg/opengl.jpg"
tags: [OpenGL]
categories: 
   -[OpenGL笔记]
---

learnopengl-4-变换和坐标系
<!-- more -->

[坐标系统-LearnOpenGL CN](https://learnopengl-cn.github.io/01%20Getting%20started/08%20Coordinate%20Systems/)

[变换-LearnOpenGL CN](https://learnopengl-cn.github.io/01%20Getting%20started/07%20Transformations/)

# 变换

这一部分设计线性代数的知识，以及在[games101](Games101-2-变换.md)中有过详细介绍，因此这一部分主要记录在OpenGL中的实践，原理不过多解释（如果之后看了[线性代数的本质](https://www.bilibili.com/video/BV1ys411472E/?spm_id_from=333.999.0.0)再进行补充）

## GLM

GLM是OpenGL Mathematics的缩写，它是一个只有头文件的库,把头文件的根目录复制到你的includes文件夹，然后你就可以使用这个库了。

```C++
#include <glm/glm.hpp>
#include <glm/gtc/matrix_transform.hpp>
#include <glm/gtc/type_ptr.hpp>
```
这个库使用方法有些独特，定义变换矩阵十分方便：
```C++
glm::mat4 trans; // 定义单位矩阵
trans = glm::translate(trans, glm::vec3(1.0f, 1.0f, 0.0f));  // 定义位移矩阵调用tranlate函数，参数是位移向量
trans = glm::rotation(trans, glm::radians(90.0f), glm::vec3(0.0, 0.0, 1.0)); // 定义旋转矩阵调用rotation函数，参数是旋转角度和旋转轴
trans = glm::scale(trans, glm::vec3(0.5, 0.5, 0.5)); // 定义缩放矩阵调用scale函数，参数是x,y,z方向上的缩放值
```

这些矩阵都是在trans的基础上生成的，相当于和trans右乘，按上面的代码就是先做缩放变换，之后旋转变换，最后位移变换。

## 实践

只需要在顶点着色器中定义一个uniform，将变换矩阵传递进shader，和原来的顶点坐标相乘:
```C++
#version 330 core
layout (location = 0) in vec3 aPos;
layout (location = 1) in vec2 aTexCoord;

out vec2 TexCoord;

uniform mat4 transform;

void main()
{
    gl_Position = transform * vec4(aPos, 1.0f);
    TexCoord = vec2(aTexCoord.x, 1.0 - aTexCoord.y);
}
```
在程序中将变换矩阵传递给uniform:
```C++
unsigned int transformLoc = glGetUniformLocation(ourShader.ID, "transform");
glUniformMatrix4fv(transformLoc, 1, GL_FALSE, glm::value_ptr(trans));
```
## 练习

1. 使用应用在箱子上的最后一个变换，尝试将其改变为先旋转，后位移。看看发生了什么，试着想想为什么会发生这样的事情：
   ![](/article_img/2022-11-02-15-31-22.png)
   会绕着原点旋转，这里是因为旋转操作都是默认绕原点的，代码中的先旋转后位移在实际变换时是先位移后旋转；
   如果希望绕任意一点旋转，需要再加一个位移矩阵。
   ```C++
    glm::mat4 trans; 
    trans = glm::translate(trans, glm::vec3(0.5f, -0.5f, 0.0f));
    trans = glm::rotate(trans, (float)glfwGetTime(), glm::vec3(0.0f, 0.0f, 1.0f));
    trans = glm::translate(trans, glm::vec3(0.5f, -0.5f, 0.0f));
   ```
   ![](/article_img/2022-11-02-15-34-54.png)
2. 尝试再次调用glDrawElements画出第二个箱子，只使用变换将其摆放在不同的位置。让这个箱子被摆放在窗口的左上角，并且会不断的缩放（而不是旋转）。（sin函数在这里会很有用，不过注意使用sin函数时应用负值会导致物体被翻转）
   ![](/article_img/2022-11-02-15-36-15.png)
   这里注意调用glDrawElements画出第二个箱子时不需要再绑定别的VAO，但是在画之前要重置变换矩阵，否则就是在同一个位置画了两次，完全看不出。


# 坐标系统

![](/article_img/2022-11-10-16-13-36.png)

"我们之所以将顶点变换到各个不同的空间的原因是有些操作在特定的坐标系统中才有意义且更方便。例如，当需要对物体进行修改的时候，在局部空间中来操作会更说得通；如果要对一个物体做出一个相对于其它物体位置的操作时，在世界坐标系中来做这个才更说得通，等等。如果我们愿意，我们也可以定义一个直接从局部空间变换到裁剪空间的变换矩阵，但那样会失去很多灵活性。"

## 局部空间（Local Space）

就像在Blender中创建了一个立方体，原点有可能位于(0, 0, 0)

## 世界空间（World Space）

世界空间中的坐标正如其名：是指顶点相对于（游戏）世界的坐标。就是将不同的模型摆放在世界的不同位置，该变换由模型矩阵（Model Matrix）实现。

模型矩阵是一种变换矩阵，它能通过对物体进行位移、缩放、旋转来将它置于它本应该在的位置或朝向。

## 观察空间（View Space）

观察空间经常被人们称之OpenGL的摄像机(Camera)（所以有时也称为摄像机空间(Camera Space)或视觉空间(Eye Space)），由视图矩阵（View Matrix）实现。

## 裁剪空间（Clip Space）

在一个顶点着色器运行的最后，OpenGL期望所有的坐标都能落在一个特定的范围内，且任何在这个范围之外的点都应该被裁剪掉(Clipped)。为了将顶点坐标从观察变换到裁剪空间，我们需要定义一个投影矩阵(Projection Matrix)，它指定了一个范围的坐标，比如在每个维度上的-1000到1000。投影矩阵接着会将在这个指定的范围内的坐标变换为标准化设备坐标的范围(-1.0, 1.0)。

投影矩阵分为透视投影（Perspective Projection Matrix）和正交投影（Orthographic Projection Matrix）

正交投影视锥（Frustum）  
![](/article_img/2022-11-10-16-26-15.png)  
透视投影视锥（Frustum）  
![](/article_img/2022-11-10-16-25-36.png)

两种投影方法的对比：
![](/article_img/2022-11-10-16-24-26.png)

在OpenGL中的用法：
```C++
// 正交投影
glm::ortho(0.0f, 800.0f, 0.0f, 600.0f, 0.1f, 100.0f);
// 透视投影
glm::mat4 proj = glm::perspective(glm::radians(45.0f), (float)width/(float)height, 0.1f, 100.0f);
```

## MVP变换

![](/article_img/2022-11-10-16-28-11.png)

经过Model-View-Projection变换，物体坐标就被变换到了裁剪坐标，之后对裁剪坐标执行透视除法从而将它们变换到标准化设备坐标，最后进行视口变换，将坐标映射到屏幕上的每一个像素。

## Z缓冲（Z-Buffer）

OpenGL自动进行深度测试，判断物体的遮挡关系。

```C++
glEnable(GL_DEPTH_TEST);

// 清理深度缓冲
glClear(GL_COLOR_BUFFER_BIT | GL_DEPTH_BUFFER_BIT);
```

## 练习

1. 使用模型矩阵只让是3倍数的箱子旋转（以及第1个箱子），而让剩下的箱子保持静止
   ```C++
   for (unsigned int i = 0; i < 10; i++)
   {
      glm::mat4 model;
      model = glm::translate(model, cubePositions[i]);
      if (i % 3 == 0) {
            model = glm::rotate(model, (float)glfwGetTime() * glm::radians(50.0f), cubePositions[i] + glm::vec3(1.0f, 0.3f, 0.5f));
      }
      unsigned int modelLoc = glGetUniformLocation(ourShader.ID, "model");
      glUniformMatrix4fv(modelLoc, 1, GL_FALSE, glm::value_ptr(model));

      glDrawArrays(GL_TRIANGLES, 0, 36);
   }
   ```
   ![](/article_img/2022-11-10-16-34-14.png)