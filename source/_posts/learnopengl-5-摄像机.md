---
title: learnopengl-5-摄像机
date: 2022-11-12
index_img: "/img/bg/opengl.jpg"
tags: [OpenGL]
categories: 
   -[OpenGL笔记]
---

learnopengl-5-摄像机
<!-- more -->

[摄像机-LearnOpenGL CN](https://learnopengl-cn.github.io/01%20Getting%20started/09%20Camera/)

OpenGL本身没有摄像机(Camera)的概念，但我们可以通过把场景中的所有物体往相反方向移动的方式来模拟出摄像机，产生一种我们在移动的感觉，而不是场景在移动。所以其实是根据view矩阵移动场景中的顶点。

# 摄像机/观察空间

观察矩阵把所有的世界坐标变换为相对于摄像机位置与方向的观察坐标，定义一个摄像机需要在世界空间中的位置，摄像机看向的方向，一个指向右侧的向量和一个指向上方的向量。

![](/article_img/2022-11-12-14-23-16.png)

## 摄像机位置

摄像机位置简单来说就是世界空间中一个指向摄像机位置的向量。

```C++
glm::vec3 cameraPos = glm::vec3(0.0f,0.0f,3.0f);
```

## 摄像机方向

相机位置减看向的点，得到摄像机看向方向的**反方向**，看上面的图，相机z轴的正方向与实际观察方向相反。
```C++
glm::vec3 cameraTarget = glm::vec3(0.0f, 0.0f, 0.0f);
glm::vec3 cameraDirection = glm::normalize(cameraPos - cameraTarget);
```

## 右轴

先定义一个向上向量，之后将这个向量与上一步的摄像机方向向量做叉乘，得到向右向量。
```C++
glm::vec3 up = glm::vec3(0.0f, 1.0f, 0.0f); 
glm::vec3 cameraRight = glm::normalize(glm::cross(up, cameraDirection));
```

## 上轴

将向右向量与相机方向向量做叉乘得到向上方向。
```C++
glm::vec3 cameraUp = glm::cross(cameraDirection, cameraRight);
```

# Look At

![](/article_img/2022-11-12-14-39-20.png)

三个不线性相关的向量，可以构建一个向量空间，LookAt矩阵的含义就是先将场景中的点相对于相机位置移动（右侧矩阵），再将点从世界坐标转换到摄像机空间坐标（左侧矩阵）

使用GLM的lookAt函数可以方便的定义lookAt矩阵：
```C++
glm::mat4 view;
view = glm::lookAt(glm::vec3(0.0f, 0.0f, 3.0f), 
           glm::vec3(0.0f, 0.0f, 0.0f), 
           glm::vec3(0.0f, 1.0f, 0.0f));
```

# 自由移动

之前GLFW的键盘输入已经定义过一个processInput函数了，添加对WASD的检测即可，之后当对应的按键按下时，向对应方向移动（让相机位置加上对应方向的向量）即可。

为了在不同的设备上的相机移动速度相同，需要跟踪一个时间差(Deltatime)变量，它储存了渲染上一帧所用的时间。如果我们的deltaTime很大，就意味着上一帧的渲染花费了更多时间，所以这一帧的速度需要变得更高来平衡渲染所花去的时间。

# 视角移动

## 欧拉角

1. Pitch(俯仰角)
2. Yaw(偏航角)
3. Row(滚转角)

![](/article_img/2022-11-12-14-57-30.png)

在一般的摄像机系统中，不考虑Row的影响。

视角移动就是通过pitch和yaw直接得出摄像机方向。

![](/article_img/2022-11-12-15-18-18.png)

```C++
direction.x = cos(glm::radians(pitch)) * cos(glm::radians(yaw)); // 译注：direction代表摄像机的前轴(Front)，这个前轴是和本文第一幅图片的第二个摄像机的方向向量是相反的
direction.y = sin(glm::radians(pitch));
direction.z = cos(glm::radians(pitch)) * sin(glm::radians(yaw));
```

## 鼠标输入

```C++
void mouse_callback(GLFWwindow* window, double xpos, double ypos);

// 在main函数中启用
glfwSetCursorPosCallback(window, mouse_callback);
```
xpos和ypos记录了鼠标的位置，用GLFW注册了回调函数之后，鼠标一移动就会调用mouse_callback函数，并且得到鼠标位置。

得到鼠标位置之后，需要和上一帧的鼠标位置对比，如果鼠标上下移动就是要调整pitch，左右移动就是调整yaw

```C++
float xoffset = xpos - lastX;
float yoffset = lastY - ypos; // 注意这里是相反的，因为y坐标是从底部往顶部依次增大的
lastX = xpos;
lastY = ypos;

float sensitivity = 0.05f;
xoffset *= sensitivity;
yoffset *= sensitivity;

yaw   += xoffset;
pitch += yoffset;

// 保证用户能看到的最大范围
if(pitch > 89.0f)
  pitch =  89.0f;
if(pitch < -89.0f)
  pitch = -89.0f;
```
这里需要给xoffset和yoffset乘一个灵敏度（应为0到1之间），否者相当于直接将偏移值当作了角度变化值，就会导致视角移动过大。

# 缩放

缩放依靠fov的变化即可，fov越小，看到的图像就越大，就产生了放大的感觉。

```C++
void scroll_callback(GLFWwindow* window, double xoffset, double yoffset)
{
  if(fov >= 1.0f && fov <= 45.0f)
    fov -= yoffset;
  if(fov <= 1.0f)
    fov = 1.0f;
  if(fov >= 45.0f)
    fov = 45.0f;
}

// 注册回调函数
glfwSetScrollCallback(window, scroll_callback);

// 在main函数中将投影矩阵的fov也相应改变
projection = glm::perspective(glm::radians(fov), 800.0f / 600.0f, 0.1f, 100.0f);
```
# 练习

1. 看看你是否能够修改摄像机类，使得其能够变成一个真正的FPS摄像机（也就是说不能够随意飞行）；你只能够呆在xz平面上：  
   只需要设定相机只能朝前或后移动，不能朝相机看向方向移动。
   ```C++
   glm::vec3 WorldFront = glm::vec3(0.0f, 0.0f, -1.0f);
   float velocity = MovementSpeed * deltaTime;
   if (direction == FORWARD)
      Position += WorldFront * velocity;
   if (direction == BACKWARD)
      Position -= WorldFront * velocity;
   ```


