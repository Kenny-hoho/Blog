---
title: UE5.4官方MotionMatching源码
date: 2024-8-7
index_img: "/article_img/2024-07-14-17-57-49.png"
tags: [计算机角色动画]
categories: 
   -[笔记]
---

分析UE5.4官方MotionMatching源码
<!-- more -->

# 动画蓝图节点（AnimNode_MotionMatching）

## 概述

动画蓝图中的MotionMatching节点的cpp类是 **AnimNode_MotionMatching**，继承自 **FAnimNode_BlendStack_Standalone** （负责过渡），该类又继承自 **FAnimNode_AssetPlayerBase**，SequencePlayer和SequenceEvaluator也继承自该类，因此可以理解MM节点就是一个复杂的SequenceEvaluator（序列求值器），求出当前要播放哪一帧。
所有的AnimNode都继承自 **FAnimNode_Base** 类，其中有几个重要的函数 ：
1. **Initialize_AnyThread** （节点第一次被调用时的初始化操作）
   ![](/article_img/2024-01-24-09-31-13.png)
2. **Update_AnyThread** （图表更新时调用，一般用来计算影响骨骼姿势的权重）
   ![](/article_img/2024-01-24-09-31-01.png)
3. **Evaluate_AnyThread** （根据update中计算出的权重估计本地空间下的骨骼变换）
   ![](/article_img/2024-01-24-09-30-51.png)
我们看任何一个AnimNode的代码都可以从这几个函数入手。

## UpdateAssetPlayer

注意到MM节点没有覆写 **Update_AnyThread** 因为MM节点继承自 **FAnimNode_AssetPlayerBase**，该父类将 **Update_AnyThread** 定义为了final，无法覆写：
![](/article_img/2024-01-24-09-27-28.png)
MM节点的主要逻辑都写在 **UpdateAssetPlayer** 中，其中逻辑简单分为两步：
1. 执行MM核心算法：UPoseSearchLibrary::UpdateMotionMatchingState(...)
2. 过渡到新的姿势：FAnimNode_BlendStack_Standalone::BlendTo(...)

## 重要的成员变量

1. **FMotionMatchingState MotionMatchingState**：用于封装MM算法，核心算法都在里面，其中有结构体 **FSearchResult CurrentSearchResult** 用于存放匹配的结果，匹配的动画帧由 **动画资产** 和 **当前播放时间**，这点与第三方插件Motion Symphony中的将所有数据库中的帧分配id，匹配结果为id不同；
2. **TObjectPtr\<const UPoseSearchDatabase> Database = nullptr;**：搜索用的动画数据库；

## 总结

AnimNode_MotionMatching的主要工作是**控制匹配的动画数据库**，**过渡到匹配结果**，核心的MM算法被封装到了UpdateMotionMatchingState()函数中了。

# 核心算法逻辑

## PoseSearchLibrary.UpdateMotionMatchingState

该函数负责控制MM的状态，即**控制什么情况下进行匹配**：如果当前正在播放的动画资产不能继续向后播放了（例如某一个动画序列播放到了最后一帧）并且距离上次匹配过去了匹配的间隔时间，则开启新的匹配。核心代码如下：
```C++
if(bSearch){
    if (bSearchContinuingPose)
    {
        // 搜索连续姿势
        SearchResult = CurrentResultDatabase->SearchContinuingPose(SearchContext);
        SearchContext.UpdateCurrentBestCost(SearchResult.PoseCost);
    }
    bool bJumpToPose = false;
    // 遍历所有数据库
    for (const TObjectPtr<const UPoseSearchDatabase>& Database : Databases)
    {
        if (ensure(Database))
        {
            // 搜索得到该数据库中的新的候选结果
            FSearchResult NewSearchResult = Database->Search(SearchContext);
            if (NewSearchResult.PoseCost.GetTotalCost() < SearchResult.PoseCost.GetTotalCost())
            {
                // 替换搜索结果
                bJumpToPose = true;
                SearchResult = NewSearchResult;
                SearchContext.UpdateCurrentBestCost(SearchResult.PoseCost);
            }
        }
    }
}
```

## 搜索逻辑

**PoseSearchDatabase.Search()** 函数负责在单个数据库中进行匹配搜索，其中会选择不同的搜索算法，目前官方提供了了：暴力，VPTree和PCAKDTree三种；先分析暴力方法，其他两种的函数调用上和暴力相同。

首先，进行了一些检查判断当前数据库是否还有搜索的意义，如果当前SearchContext的最佳cost已经优于 SearchIndex.MinCostAddend（？可以保证所有的cost都大于这个值，还需要再看是怎么得到这个值的），就不需要再搜索当前数据库了。之后，需要得到**查询数组**（QueryValues，就是动画特征数据库），并且构建 **姿势数组** （当前姿势的特征数组），调用 **EvaluatePoseKernel** 计算cost得到匹配结果。

## 构建查询数组

PoseSearchSchema::BuildQuery()




