---
title: 基于MotionMatching毕业论文框架
date: 2025-1-7
index_img: "/img/bg/InteractionMM.png"
tags: [计算机角色动画]
categories: 
   -[笔记]
---

毕业论文框架
<!-- more -->

# 研究内容

1. 轨迹跟随算法（解决目前长距离轨迹跟随算法跟随效果较差以及效率偏低的问题）
   1. 轨迹特征提取（额外标记轨迹终点的动画帧编号）
   2. 重建数据库中的长距离轨迹
   3. 实时采集指定轨迹点（考虑动态采集，锐角动态调整）
   4. 使用DTW等方法进行匹配（对比效果的性能，理论上性能得到提升）
2. 交互动MM算法（解决目前角色交互动画效果失真，相同效果实现复杂的问题）
   1. 交互特征提取
   2. 生成toTarget轨迹（要停下来，要考虑结束方向）
   3. 使用轨迹跟随算法完成轨迹跟随
   4. 实时交互特征检测
   5. 进行动作匹配（对比效果，理论上效果更佳自然）

# ToDo

## 实验

### 轨迹

1. 真实动画数据轨迹提取
2. 轨迹匹配对比实验（单纯轨迹匹配，非实机）
3. 数据库中长距离轨迹重建
4. 最终效果
   1. 轨迹跟随效果对比
   2. 效率对比

### 场景交互

1. 蒙太奇先做出来
2. toTarget轨迹生成算法
3. 实时检测
   1. 参考MMInteraction根据动画数据分区检测

## 论文

1. 算法伪代码
2. 论文框架
3. 公式


# 框架参考
![](/article_img/2025-01-07-16-22-31.png) | ![](/article_img/2025-01-07-16-22-48.png)
---|---

