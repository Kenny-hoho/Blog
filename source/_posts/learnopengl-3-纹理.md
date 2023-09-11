---
title: learnopengl-3-纹理
date: 2022-11-01
index_img: "/img/bg/opengl.jpg"
tags: [OpenGL]
categories: 
   -[OpenGL笔记]
---

learnopengl-3-纹理
<!-- more -->

[纹理-LearnOpenGL CN](https://learnopengl-cn.github.io/01%20Getting%20started/06%20Textures/)

纹理是一个2D图片（甚至也有1D和3D的纹理），它可以用来添加物体的细节；你可以想象纹理是一张绘有砖块的纸，无缝折叠贴合到你的3D的房子上，这样你的房子看起来就像有砖墙外表了。

# 纹理坐标

用来标明该从纹理图像的哪个部分采样（译注：采集片段颜色）。使用纹理坐标获取纹理颜色叫做采样(Sampling)。

![](/article_img/2022-11-01-16-58-12.png)

纹理坐标格式：

```c++
float texCoords[] = {
    0.0f, 0.0f, // 左下角
    1.0f, 0.0f, // 右下角
    0.5f, 1.0f // 上中
};
```

# 纹理环绕方式

| 环绕方式  | 描述  |
| :---: | :---: |
|   GL_REPEAT   |   对纹理的默认行为。重复纹理图像。   |
|   GL_MIRRORED_REPEAT   |   和GL_REPEAT一样，但每次重复图片是镜像放置的。   |
|   GL_CLAMP_TO_EDGE  | 纹理坐标会被约束在0到1之间，超出的部分会重复纹理坐标的边缘，产生一种边缘被拉伸的效果。 |
|   GL_CLAMP_TO_BORDER | 超出的坐标为用户指定的边缘颜色 |

![](/article_img/2022-11-01-17-15-23.png)

用法：  

```C++
glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_WRAP_S, GL_MIRRORED_REPEAT);
glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_WRAP_T, GL_MIRRORED_REPEAT);
```
可以单独对坐标轴设置纹理环绕方式。

GL_CLAMP_TO_BORDER 方式由于是用户指定的边缘颜色，因此要再指定一个边缘的颜色，同时也不需要再指定坐标轴了。
```C++
float borderColor[] = {1.0f, 1.0f, 0.0f, 1.0f};
glTexParameterfv(GL_TEXTURE_2D, GL_TEXTURE_BORDER_COLOR, borderColor);
```

# 纹理过滤

## 纹理像素（Texture Pixel）

Texture Pixel也叫Texel，你可以想象你打开一张.jpg格式图片，不断放大你会发现它是由无数像素点组成的，这个点就是纹理像素，其实就是**纹理的分辨率**。

如果要将一个分辨率较低的纹理加在一个很大的物体上，看上去物体的分辨率就很低，为了减轻这种现象，需要纹理过滤。

## 纹理过滤

这一部分在[Games101-着色](Games101-4-着色.md)中有过详细介绍

1. 邻近过滤（GL_NEAREST）  
   ![](/article_img/2022-11-01-17-16-23.png)
2. 线性过滤（GL_LINEAR）  
   ![](/article_img/2022-11-01-17-16-55.png)  
   使用的算法就是双线性插值，由周围几个像素的颜色算出最终应该采用的颜色。  
   ![](/article_img/2022-08-29-18-33-20.png)

下图是两种过滤方法的效果对比
![](/article_img/2022-11-01-17-17-08.png)

用法：
```C++
glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MIN_FILTER， GL_NEAREST);
glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MAG_FILTER， GL_LINEAR);
```

# 多级渐远纹理

这一部分也在[Games101-着色](Games101-4-着色.md)中有过详细介绍

当物体很小，而纹理分辨率很大的时候难以决定最终真正显示的像素的颜色，因为最终可见的像素很可能覆盖了纹理贴图上的很大一部分，同时还会产生很多浪费。

OpenGL中使用Mipmap解决这个问题。

![](/article_img/2022-08-29-18-41-41.png)

可以使用glGenerateMipmaps函数来自动生成Mipmap

![](/article_img/2022-11-01-17-30-18.png)

在切换Mipmap时，在两个层之间会产生生硬的边界，因此在切换时也需要过滤。

| 过滤方式  | 描述  |
| :---: | :---: |
|   GL_NEAREST_MIPMAP_NEAREST   |   使用最邻近的多级渐远纹理来匹配像素大小，并使用邻近插值进行纹理采样   |
|   GL_LINEAR_MIPMAP_NEAREST   |   使用最邻近的多级渐远纹理级别，并使用线性插值进行采样   |
|   GL_NEAREST_MIPMAP_LINEAR | 在两个最匹配像素大小的多级渐远纹理之间进行线性插值，使用邻近插值进行采样 |
|   GL_LINEAR_MIPMAP_LINEAR | 在两个邻近的多级渐远纹理之间使用线性插值，并使用线性插值进行采样 |

![](/article_img/2022-08-29-18-52-05.png)

这里看这个三线性插值（GL_LINEAR_MIPMAP_LINEAR）的原理图就很容易理解，GL_**A**\_MIPMAP_**B** 中**A**是指不同层级之间的切换方式，**B**指每个层级内部的选择方式。

使用：
```C++
glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MIN_FILTER, GL_LINEAR_MIPMAP_LINEAR);
glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MAG_FILTER, GL_LINEAR);
```

注意：  
函数的第二个参数，是分别设置纹理缩小时过滤和纹理放大时过滤的参数。**Mipmap只在纹理被缩小时有效。**

# 加载与创建纹理

## stb_image.h

加载纹理使用stb_image.h头文件，这个头文件中就包含了函数的实现，因此只需要将其添加到工程中，再创建一个C++文件输入：
```C++
#define STB_IMAGE_IMPLEMENTATION
#include "stb_image.h"
```
即可调用**stbi_load**函数进行图片的加载。

## 生成纹理

生成一个纹理的过程：
```C++
unsigned int texture;
glGenTextures(1, &texture);
glBindTexture(GL_TEXTURE_2D, texture);
// 为当前绑定的纹理对象设置环绕、过滤方式
glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_WRAP_S, GL_REPEAT);   
glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_WRAP_T, GL_REPEAT);
glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MIN_FILTER, GL_LINEAR);
glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MAG_FILTER, GL_LINEAR);
// 加载并生成纹理
int width, height, nrChannels;
unsigned char *data = stbi_load("container.jpg", &width, &height, &nrChannels, 0);
if (data)
{   
    // 生成纹理
    glTexImage2D(GL_TEXTURE_2D, 0, GL_RGB, width, height, 0, GL_RGB, GL_UNSIGNED_BYTE, data);
    glGenerateMipmap(GL_TEXTURE_2D);
}
else
{
    std::cout << "Failed to load texture" << std::endl;
}
stbi_image_free(data);
```

# 应用纹理

## 基本应用

纹理的应用需要相应的改变着色器代码，在顶点着色器中设定一个输出向量将纹理坐标传给片段着色器，在片段着色器中定义一个类型是sampler2D的uniform用来接收纹理图片，最后调用GLSL中的texture函数来采样纹理的颜色。

sampler2D是一个GLSL的数据类型，叫做采样器，还有sampler1D, sampler3D, 用来把纹理对象传给片段着色器。

顶点着色器：
```C++
#version 330 core
layout (location = 0) in vec3 aPos;
layout (location = 1) in vec3 aColor;
layout (location = 2) in vec2 aTexCoord;

out vec3 ourColor;
out vec2 TexCoord;

void main()
{
    gl_Position = vec4(aPos, 1.0);
    ourColor = aColor;
    TexCoord = aTexCoord;
}
```

片段着色器：
```C++
#version 330 core
out vec4 FragColor;

in vec3 ourColor;
in vec2 TexCoord;

uniform sampler2D ourTexture;

void main()
{
    FragColor = texture(ourTexture, TexCoord);
}
```

渲染循环：
```C++
glBindTexture(GL_TEXTURE_2D, texture);
glBindVertexArray(VAO);
glDrawElements(GL_TRIANGLES, 6, GL_UNSIGNED_INT, 0);
```

## 纹理单元

![](/article_img/2022-11-02-15-10-07.png)

纹理单元的目的是为了让我们可以使用多个纹理，通过把纹理单元**赋值**给采样器，我们可以一次绑定多个纹理，只要我们首先**激活**对应的纹理单元；

OpenGL保证有16个纹理单元供使用，且GL_TEXTRUE0默认被激活，所以只用一个纹理时可以不激活纹理单元0。

使用方法：
```C++


// 激活纹理单元并将纹理绑定到相应的纹理单元
glActiveTexture(GL_TEXTURE0);
glBindTexture(GL_TEXTURE_2D, texture1);
glTexImage2D(..., data1);
glGenerateMipmap(GL_TEXTURE_2D);
glActiveTexture(GL_TEXTURE1);
glBindTexture(GL_TEXTURE_2D, texture2);
glTexImage2D(..., data2);
glGenerateMipmap(GL_TEXTURE_2D)

// 设置uniform的值
ourShader.setInt("texture1", 0);
ourShader.setInt("texture2", 1);
```

# 练习

1. 修改片段着色器，仅让笑脸图案朝另一个方向看
   ```C++
   FragColor = mix(texture(texture1, TexCoord), texture(texture2, vec2(1.0 - TexCoord.x, TexCoord.y)), 0.2);
   ```
2. 尝试用不同的纹理环绕方式，设定一个从0.0f到2.0f范围内的（而不是原来的0.0f到1.0f）纹理坐标。试试看能不能在箱子的角落放置4个笑脸。记得一定要试试其它的环绕方式。

    环绕方式和过滤方式需要对不同的纹理分别设置
   ```C++
    unsigned int texture1, texture2;
    glGenTextures(1, &texture1);
    glActiveTexture(GL_TEXTURE0);
    glBindTexture(GL_TEXTURE_2D, texture1);
    // 设置环绕，过滤方式
    glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_WRAP_S, GL_CLAMP_TO_EDGE);
    glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_WRAP_T, GL_CLAMP_TO_EDGE);
    glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MIN_FILTER, GL_LINEAR);
    glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MAG_FILTER, GL_LINEAR);

    glTexImage2D(GL_TEXTURE_2D, 0, GL_RGB, width, height, 0, GL_RGB, GL_UNSIGNED_BYTE, data1);
    glGenerateMipmap(GL_TEXTURE_2D);
    // 释放图像内存
    stbi_image_free(data1);

    glGenTextures(1, &texture2);
    glActiveTexture(GL_TEXTURE2);
    glBindTexture(GL_TEXTURE_2D, texture2);
    // 设置环绕，过滤方式
    glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_WRAP_S, GL_MIRRORED_REPEAT);
    glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_WRAP_T, GL_MIRRORED_REPEAT);
    glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MIN_FILTER, GL_LINEAR);
    glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MAG_FILTER, GL_LINEAR);

    glTexImage2D(GL_TEXTURE_2D, 0, GL_RGBA, width, height, 0, GL_RGBA, GL_UNSIGNED_BYTE, data2);
    glGenerateMipmap(GL_TEXTURE_2D);
   ```
   ![](/article_img/2022-11-01-19-04-15.png)
3. 尝试在矩形上只显示纹理图像的中间一部分，修改纹理坐标，达到能看见单个的像素的效果。尝试使用GL_NEAREST的纹理过滤方式让像素显示得更清晰
   ```C++
    float offset = 0.03;

    float vertices[] = {
        //     ---- 位置 ----       ---- 颜色 ----     - 纹理坐标 -
         0.5f,  0.5f, 0.0f,   1.0f, 0.0f, 0.0f,   0.5f + offset, 0.5f + offset,   // 右上
         0.5f, -0.5f, 0.0f,   0.0f, 1.0f, 0.0f,   0.5f + offset, 0.5f - offset,   // 右下
        -0.5f, -0.5f, 0.0f,   0.0f, 0.0f, 1.0f,   0.5f - offset, 0.5f - offset,   // 左下
        -0.5f,  0.5f, 0.0f,   1.0f, 1.0f, 0.0f,   0.5f - offset, 0.5f + offset    // 左上
    };
   ```
   将纹理放大后的效果：
   ![](/article_img/2022-11-01-19-10-25.png)
   ```C++
    glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MIN_FILTER, GL_NEAREST);
    glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MAG_FILTER, GL_NEAREST);
   ```
   可以明显看出使用GL_LINEAR过滤方法锯齿感更弱
   ![](/article_img/2022-11-01-19-12-45.png)
4. 使用一个uniform变量作为mix函数的第三个参数来改变两个纹理可见度，使用上和下键来改变箱子或笑脸的可见度
   ![](/article_img/2022-11-01-19-40-30.png)
   ![](/article_img/2022-11-01-19-40-57.png)
   ```C++
   // 输入函数，让所有输入代码保持整洁
    void processInput(GLFWwindow* window, float &mix_ratio) {
        if (glfwGetKey(window, GLFW_KEY_ESCAPE) == GLFW_PRESS) {
            std::cout << "ESCAPE PRESSED!" << std::endl;
            glfwSetWindowShouldClose(window, true);
        }
            
        if (glfwGetKey(window, GLFW_KEY_UP) == GLFW_PRESS) {
            mix_ratio += 0.1f;
            if (mix_ratio > 1.0f)
                mix_ratio = 1.0f;
            std::cout << "UP PRESSED!" << std::endl;
        }
        if (glfwGetKey(window, GLFW_KEY_DOWN) == GLFW_PRESS) {
            mix_ratio -= 0.1f;
            if (mix_ratio < 0.0f)
                mix_ratio = 0.0f;
            std::cout << "DOWN PRESSED!" << std::endl;
        }
    }

    // 渲染循环
    ourShader.setFloat("mix_ratio", mix_ratio);

   ```