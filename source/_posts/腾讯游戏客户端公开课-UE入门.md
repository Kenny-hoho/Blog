---
title: 腾讯游戏客户端公开课-UE入门
date: 2023-09-08
index_img: "/img/bg/West2.jpg"
tags: [游戏客户端公开课]
hide: true
categories: 
   -[客户端]
---

<!-- more -->

# 游戏引擎介绍

**游戏引擎：专门为游戏而设计的工具及科技集合**

![](/article_img/2023-09-08-19-27-29.png)

## 渲染模块

![](/article_img/2023-09-08-19-31-00.png)

1. Immediate Mode Rendering
   ![](/article_img/2023-09-08-19-33-48.png)
   内存和带宽要求很高，不适合移动端
2. Tile Based Rendering （TBR）
   ![](/article_img/2023-09-08-19-34-43.png)
   将屏幕分块渲染，避免了带宽和能耗的问题，适合移动端
3. TileBased Deferred Rendering (TBDR)
   ![](/article_img/2023-09-08-19-36-26.png)

## 物理模块

![](/article_img/2023-09-08-19-36-55.png)

UE使用自研的Chaos引擎。

# UE引擎介绍

Unreal的发展历程：
![](/article_img/2023-09-08-19-42-42.png)

## UE的优势

![](/article_img/2023-09-08-19-14-22.png)

## Laucher

通过EPIC启动；

## 编辑器使用

![](/article_img/2023-09-08-19-53-25.png)

![](/article_img/2023-09-08-20-08-01.png)

![](/article_img/2023-09-08-20-08-46.png)

UE命名的规范：
![](/article_img/2023-09-08-20-10-44.png)

## 项目结构

![](/article_img/2023-09-08-20-11-14.png)

![](/article_img/2023-09-08-20-12-23.png)

![](/article_img/2023-09-08-20-13-05.png)

![](/article_img/2023-09-08-20-13-24.png)

![](/article_img/2023-09-08-20-13-45.png)

## 源码构建

UE源码需要将github账号绑定到epic账号，才能进入UE的github仓库。

![](/article_img/2023-09-08-20-43-15.png)

![](/article_img/2023-09-08-20-18-21.png)

![](/article_img/2023-09-08-20-19-02.png)

![](/article_img/2023-09-08-20-20-20.png)

![](/article_img/2023-09-08-20-21-03.png)

![](/article_img/2023-09-08-20-21-55.png)

![](/article_img/2023-09-08-20-22-43.png)

![](/article_img/2023-09-08-20-22-56.png)

![](/article_img/2023-09-08-20-23-10.png)

![](/article_img/2023-09-08-20-23-28.png)

![](/article_img/2023-09-08-20-24-19.png)

# UE编程技巧

## 游戏框架

![](/article_img/2023-09-08-20-25-18.png)

![](/article_img/2023-09-08-20-25-53.png)

![](/article_img/2023-09-08-20-26-15.png)

![](/article_img/2023-09-08-20-26-45.png)

![](/article_img/2023-09-08-20-27-13.png)

## Lua和蓝图

![](/article_img/2023-09-08-20-27-38.png)

![](/article_img/2023-09-08-20-28-17.png)

![](/article_img/2023-09-08-20-28-28.png)

![](/article_img/2023-09-08-20-29-12.png)

![](/article_img/2023-09-08-20-29-38.png)

蓝图与C++的交互：
![](/article_img/2023-09-08-20-30-00.png)

![](/article_img/2023-09-08-20-30-42.png)

![](/article_img/2023-09-08-20-31-34.png)

![](/article_img/2023-09-08-20-32-07.png)

蓝图与Lua：
![](/article_img/2023-09-08-20-33-11.png)

![](/article_img/2023-09-08-20-33-18.png)

C++ in UE：
![](/article_img/2023-09-08-20-33-45.png)

C++ 编码规范（不能随便写）：
![](/article_img/2023-09-08-20-34-07.png)

![](/article_img/2023-09-08-20-34-42.png)

![](/article_img/2023-09-08-20-35-12.png)

![](/article_img/2023-09-08-20-36-49.png)

![](/article_img/2023-09-08-20-37-40.png)

![](/article_img/2023-09-08-20-38-00.png)

# UE引擎工具

![](/article_img/2023-09-08-20-38-26.png)

![](/article_img/2023-09-08-20-38-44.png)

![](/article_img/2023-09-08-20-39-11.png)

![](/article_img/2023-09-08-20-40-00.png)

![](/article_img/2023-09-08-20-40-46.png)

# 作业

![](/article_img/2023-09-08-20-48-10.png)




# 编译

![](/article_img/2023-09-09-19-55-45.png)