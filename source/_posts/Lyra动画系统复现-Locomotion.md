---
title: Lyra动画系统-Locomotion
date: 2023-12-23
index_img: "/img/bg/Lyra.jpg"
tags: [计算机角色动画]
categories: 
   -[笔记]
---

<!-- more -->

[UE5 骨骼动画 Lyra 距离匹配 速度匹配](https://zhuanlan.zhihu.com/p/572811605)
[Distance Matching in UE5](https://zhuanlan.zhihu.com/p/574219921)
[官方文档](https://docs.unrealengine.com/5.0/zh-CN/distance-matching-in-unreal-engine/)

# 距离匹配（Distance Matching）

距离匹配是一种避免**胶囊体速度与动画根运动速度不匹配**造成的滑步等现象的技术；我们人类在运动的时候几乎不会是匀速运动或者匀加速运动，因此动捕出来的动画数据的加速度也不是匀速的，而由程序驱动的胶囊体的加速是匀速的，就会造成速度与动画不匹配，产生滑步现象：
![](/article_img/2024-01-03-10-34-26.png)

在UE中距离匹配是一个大概念，其中包括了三个具体的函数：**Advance Time by Distance Matching**，**Distance Match to Target**，**Set Playrate to Match Speed**；
![](/article_img/2024-01-03-10-12-00.png)

接下来我们逐一进行分析学习；

## Advance Time by Distance Matching

Advance Time by Distance Matching（根据距离匹配前进时间），看名字我们就可以看出这个函数的作用是前进时间，也就是直接跳到该播放的位置开始播放。下面是**AdvanceTimeByDistanceMatching**函数源码：

```C++
FSequenceEvaluatorReference UAnimDistanceMatchingLibrary::AdvanceTimeByDistanceMatching(...)
{
// ...省略一些代码
   const float CurrentTime = InSequenceEvaluator.GetExplicitTime();
   const float CurrentAssetLength = InSequenceEvaluator.GetCurrentAssetLength();
   const bool bAllowLooping = InSequenceEvaluator.GetShouldLoop();

   // 得到曲线ID
   const USkeleton::AnimCurveUID CurveUID = UE::Anim::DistanceMatchingUtility::GetCurveUID(AnimSequence, DistanceCurveName);
   // ---------- 核心操作，通过GetTimeAfterDistanceTraveled函数得到应该前进到那一帧 ----------
   float TimeAfterDistanceTraveled = UE::Anim::DistanceMatchingUtility::GetTimeAfterDistanceTraveled(AnimSequence, CurrentTime, DistanceTraveled, CurveUID, bAllowLooping);
   // -------------------------------------------------------------------------------------

   // 如果计算出应该前进到的帧比当前帧小，则说明是循环动画
   if (TimeAfterDistanceTraveled < CurrentTime)
   {
      TimeAfterDistanceTraveled += CurrentAssetLength;
   }
   // 计算播放速率应该是多少，算出后再做一个clamp，限制在合理的范围
   float EffectivePlayRate = (TimeAfterDistanceTraveled - CurrentTime) / DeltaTime;
   if (PlayRateClamp.X >= 0.0f && PlayRateClamp.X < PlayRateClamp.Y)
   {
      EffectivePlayRate = FMath::Clamp(EffectivePlayRate, PlayRateClamp.X, PlayRateClamp.Y);
   }

   // 最后前进时间，这里速率又乘上了时间，表示就是前进到之前GetTimeAfterDistanceTraveled函数算出的那一帧
   float NewTime = CurrentTime;
   FAnimationRuntime::AdvanceTime(bAllowLooping, EffectivePlayRate * DeltaTime, NewTime, CurrentAssetLength);
}
```
通过阅读以上源码可以看出函数思路还是较为简单，要注意的是，函数是**正在计算**当前应该播放那一帧动画，也就是说函数中的CurrentTime其实是已经播放完的动画的上一帧，根据胶囊体在当前帧（实际也是上一帧，胶囊体已经移动过了）移动的距离，在根运动曲线中找到要移动到对应距离应该走到那一帧：
![](/article_img/AdvanceTimebyDistanceMatch.jpg)

其中得到**应该前进到哪一帧**的关键函数**GetTimeAfterDistanceTraveled**的核心代码如下：
```C++
// 省略一些
float NewTime = CurrentTime;
const float StepTime = 1.f / 30.f; // 每次前进的固定步长
// 这个循环每次前进固定时长，寻找应该前进到那一帧
while ((AccumulatedDistance < DistanceTraveled) && (bAllowLooping || (NewTime + StepTime < SequenceLength)))
{
   const float CurrentDistance = AnimSequence->EvaluateCurveData(CurveUID, NewTime);
   const float DistanceAfterStep = AnimSequence->EvaluateCurveData(CurveUID, NewTime + StepTime);
   const float AnimationDistanceThisStep = DistanceAfterStep - CurrentDistance;

   if (!FMath::IsNearlyZero(AnimationDistanceThisStep))
   {
      // 没有达到真实前进的位置就继续以固定步长前进
      if (AccumulatedDistance + AnimationDistanceThisStep < DistanceTraveled)
      {
         // 此函数可以理解为：NewTime=NewTime+StepTime；
         FAnimationRuntime::AdvanceTime(bAllowLooping, StepTime, NewTime, SequenceLength);
         AccumulatedDistance += AnimationDistanceThisStep;
      }
      // 一旦超过了真实前进的距离就按比例计算出应该前进到哪一帧
      else
      {
         const float DistanceAlpha = (DistanceTraveled - AccumulatedDistance) / AnimationDistanceThisStep;
         FAnimationRuntime::AdvanceTime(bAllowLooping, DistanceAlpha * StepTime, NewTime, SequenceLength);
         AccumulatedDistance = DistanceTraveled;
         break;
      }

      StuckLoopCounter = 0;
   }
}
return NewTime;
```

该函数一般用于起步动画：
![](/article_img/2024-01-03-10-18-50.png)

## Distance Match to Target

Distance Match to Target（距离匹配到目标点），一般用于停步动画，避免总是完整播放停步动画（角色已经停止还在播放剩余的停步动画）从而产生的滑步现象，根据真实停止所需的距离从动画中匹配要开始播放的帧；
![](/article_img/2024-01-03-12-33-05.png)

核心代码：
```C++
// 使用GetAnimPositionFromDistance函数找到开始播放的帧，这个函数实现就是一个二分查找
const float NewTime = UE::Anim::DistanceMatchingUtility::GetAnimPositionFromDistance(AnimSequence, -DistanceToTarget, CurveUID);
```

在使用该函数时，提供的 **Stop Location（停止所需的距离）** 可以通过自带的节点根据CharacterMovementComponent中的制动相关参数计算得到：
![](/article_img/2024-01-03-12-48-54.png)

## Set Playrate to Match Speed

Set Playrate to Match Speed（根据匹配速度设置播放速率），一般用在跑步走路等循环动画，用来根据真实移动速度调整动画播放速率，避免滑步；
![](/article_img/2024-01-03-12-52-28.png)
```C++
const float AnimLength = AnimSequence->GetPlayLength();
if (!FMath::IsNearlyZero(AnimLength))
{
   // Calculate the speed as: (distance traveled by the animation) / (length of the animation)
   const FVector RootMotionTranslation = AnimSequence->ExtractRootMotionFromRange(0.0f, AnimLength).GetTranslation();
   const float RootMotionDistance = RootMotionTranslation.Size2D();
   if (!FMath::IsNearlyZero(RootMotionDistance))
   {
      // 计算出动画移动速度
      const float AnimationSpeed = RootMotionDistance / AnimLength;
      // 根据真实移动速度和动画根移动速度的比值计算应该播放的速率
      float DesiredPlayRate = SpeedToMatch / AnimationSpeed;
      if (PlayRateClamp.X >= 0.0f && PlayRateClamp.X < PlayRateClamp.Y) // clamp
      {
         DesiredPlayRate = FMath::Clamp(DesiredPlayRate, PlayRateClamp.X, PlayRateClamp.Y);
      }

      if (!InSequencePlayer.SetPlayRate(DesiredPlayRate)) // 设置播放速率
      {
         //...
      }
   }
}
```
这里要注意，不要使用CharacterMovementComp中的速度值作为真实移动速度，经测试，在加速和减速过程中速度值和真实移动速度不同：
![](/article_img/2024-01-03-13-02-30.png)
因此要重新根据位置计算一个位移速度。
