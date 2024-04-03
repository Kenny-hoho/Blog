---
title: MotionSymphony插件详解
date: 2024-1-23
index_img: "/img/bg/MotionSymphony.jpg"
tags: [计算机角色动画]
categories: 
   -[笔记]
---

MotionSymphony插件报告
<!-- more -->

# Motion Symphony资产和数据结构

## 资产

Motion Symphony是一个实现Motion Matching的虚幻引擎动画工具集，Motion Matching是一个基于数据的动画驱动方案，Motion Symphony中提供了一套完整的工具和工作流用于构建MM需要的动画数据集，在Motion Symphony中这个数据集叫做 **Motion Data**；如下图，可见要构建Motion Data还需要几个其他的Motion Symphony提供的资产；
![](/article_img/2024-01-23-16-26-34.png)

Motion Matching的原理是根据当前姿势和未来轨迹在动画数据库中寻找下一帧要播放的姿势，针对姿势匹配一般选取几个骨骼代表一个姿势，对于轨迹一般分别采样几个过去和未来的点；在Motion Symphony中这些信息被定义在资产 **Motion Matching Config** 中：
![](/article_img/2024-01-23-16-34-54.png)

MM中的寻找，就是一个计算Cost的过程，计算最能匹配当前帧姿势和未来轨迹的动画帧作为下一个要播放的动画帧，计算Cost时需要对不同的特征设置不同的权重（表示我们更关注哪些特征），这些权重信息被定义在资产 **Motion Calibration** 中：
![](/article_img/2024-01-23-16-34-41.png)

这三个资产是Motion Symphony中最重要的三个自定义资产，源码中的位置在**CustomAessets**目录下：
![](/article_img/2024-01-23-16-39-57.png)

## 数据结构

Motion Symphony定义了许多数据结构来方便之后的动画节点计算：
![](/article_img/2024-01-23-16-48-25.png)
其中最重要的是 **AnimChannelState**，这个数据结构中存储了：动画id，动画权重，动画类型（序列，混合空间等）等信息，**用来方便操作（不需要每次都用AnimId去找动画长度，是否循环等等）**，AnimNode_MotionMatching中维护了一个 **BlendChannels** 数组，经过MM后选择了一个要播放的动画帧，就会将选择的动画帧构造成一个 **AnimChannelState** 并加入 **BlendChannels** 数组：
![](/article_img/2024-01-23-16-58-58.png)
之后在 **FAnimNode_MotionMatching::Evaluate_AnyThread(FPoseContext& Output)** 函数中通过BlendChannels数组选择播放的动画帧：
![](/article_img/2024-01-23-17-03-19.png)

# 数据库构建（PreProcess）

![](/article_img/2024-02-21-17-06-19.png)

自定义资产Motion Data就是动画数据库，Motion Symphony的思路是把不同类型的动画单独做成一个Motion Data（比如跑步动画构成一个Motion Data，走路动画构成另一个Motion Data），再用状态机实现各个状态间的过渡，这种方式可以显著提高MM的效率和准确度，因为要搜索的数据库变小了。

## PreProcess框架

如上图，配置好一个Motion Data的源动画和Calibration和Config后，要点击**PreProcess**，进行预处理，**把动画数据库处理成特征数据库**，减少存储大小；Motion Data中最重要的变量是**Poses**，其中并不存储整个Pose而是存储Pose的特征，PoseId，AnimType，NextPoseId等等信息；
![](/article_img/2024-02-21-17-21-04.png)

这一步操作由 **void UMotionDataAsset::PreProcess()** 函数完成（只展示核心代码）：
```C++
void UMotionDataAsset::PreProcess()
{
	MotionMatchConfig->Initialize();

	//Setup mirroring data
	ClearPoses();

	//依次处理选择的Animation Sequences
	for (int32 i = 0; i < SourceMotionAnims.Num(); ++i)
	{
		// 先处理一遍未镜像的
		PreProcessAnim(i, false);
		
		if (MirroringProfile != nullptr && SourceMotionAnims[i].bEnableMirroring)
		{   // 如果镜像再处理一遍镜像的
			PreProcessAnim(i, true);
		}
	}

	/*
	Blend Spaces预处理，和上面一样只是调用的预处理函数针对Blend Space

	Composites预处理，同理
	*/
	

	// 生成NextPoseId和LastPoseId，需要对循环动画进行一些处理
	GeneratePoseSequencing();

	//Standard deviations
	//First Find a list of traits 
	TArray<FMotionTraitField> UsedMotionTraits;
	for (int32 i = 0; i < Poses.Num(); ++i)
	{
		UsedMotionTraits.AddUnique(Poses[i].Traits);
	}

	FeatureStandardDeviations.Empty(UsedMotionTraits.Num());
	// 根据不同的Traits选择不同的权重信息，但是一般也不用Traits进行分类
	for (const FMotionTraitField& MotionTrait : UsedMotionTraits)
	{
		FCalibrationData& NewCalibrationData = FeatureStandardDeviations.Add(MotionTrait, FCalibrationData(this));
		NewCalibrationData.GenerateStandardDeviationWeights(this, MotionTrait);
	}
	PreprocessCalibration->Initialize();

	// 设置优化模块，在搜索时会根据优化模块减少搜索总数
	if(bOptimize && OptimisationModule)
	{
		OptimisationModule->BuildOptimisationStructures(this);
		bIsOptimised = true;
	}
	else
	{
		bIsOptimised = false;
	}
	bIsProcessed = true;
}
```
上面源码概况下来就是先根据不同的源动画类型分别生成**特征（MotionPoseData构成的Poses数组）**，再构建起这些特征的**顺序关系**，最后**设置优化模块**，这里调用的构建优化结构的函数作用是做检查，检查优化模块是否有效；

可以看出进行PreProcess的核心函数是 **void UMotionDataAsset::PreProcessAnim(const int32 SourceAnimIndex, const bool bMirror)**，下面看看这个函数的源码：
```C++
void UMotionDataAsset::PreProcessAnim(const int32 SourceAnimIndex, const bool bMirror /*= false*/)
{
	// 得到MotionAnim，该变量继承自 FMotionAnimAsset，其中包含了动画序列本身以及一些其他的属性如是否循环等，这些属性可以在MotionData资产编辑器中设置；
	FMotionAnimSequence& MotionAnim = SourceMotionAnims[SourceAnimIndex];
	UAnimSequence* Sequence = MotionAnim.Sequence;

	MotionAnim.AnimId = SourceAnimIndex;

	const float AnimLength = Sequence->GetPlayLength();
	float CurrentTime = 0.0f;
	// TimeHorizon用来判断某一个动画帧能否使用，其值为轨迹预测中的最大值
	float TimeHorizon = MotionMatchConfig->TrajectoryTimes.Last();
	
	FMotionTraitField AnimTraitHandle = UMMBlueprintFunctionLibrary::CreateMotionTraitFieldFromArray(MotionAnim.TraitNames);

	// PoseInterval太小没必要且影响性能
	if(PoseInterval < 0.01f)
		PoseInterval = 0.05f;
	
	int32 StartPoseId = Poses.Num();
	int32 EndPoseId = StartPoseId;
	// 进入循环，CurrentTime每循环加PoseInterval
	while (CurrentTime <= AnimLength)
	{
		// 当前帧的PoseId为Poses的个数，可以看出PoseId在每个Motion Data中都是从0开始的（因为Motion Data就是一个数据库，PoseId在同一个数据库中不同即可）
		int32 PoseId = Poses.Num();
		EndPoseId = PoseId;
		// 如果CurrentTime位于动画序列的开头或者结尾的TimeHorizon区域中，意味着其不能支持完整的轨迹预测，就标记为不可使用
		bool bDoNotUse = ((CurrentTime < TimeHorizon) && (MotionAnim.PastTrajectory == ETrajectoryPreProcessMethod::IgnoreEdges))
			|| ((CurrentTime > AnimLength - TimeHorizon) && (MotionAnim.FutureTrajectory == ETrajectoryPreProcessMethod::IgnoreEdges))
			? true : false;

		// 如果是循环则没有上面的顾虑，永远可以支持完整的轨迹预测
		if(MotionAnim.bLoop)
		{
			bDoNotUse = false;
		}

		FVector RootVelocity;
		float RootRotVelocity;
		// 提取根速度
		FMMPreProcessUtils::ExtractRootVelocity(RootVelocity, RootRotVelocity, Sequence, CurrentTime, PoseInterval);

		if (bMirror)
		{
			RootVelocity.X *= -1.0f;
			RootRotVelocity *= -1.0f;
		}

		float PoseCostMultiplier = MotionAnim.CostMultiplier;

		// 根据以上计算构建一个PoseMotionData，也就是Poses中的元素，之后要加入到Poses数组中；
		FPoseMotionData NewPoseData = FPoseMotionData(PoseId, EMotionAnimAssetType::Sequence, 
			SourceAnimIndex, CurrentTime, PoseCostMultiplier, bDoNotUse, bMirror, 
			RootRotVelocity, RootVelocity, AnimTraitHandle);
		
		//Process trajectory for pose  计算轨迹
		for (int32 i = 0; i < MotionMatchConfig->TrajectoryTimes.Num(); ++i)
		{
			FTrajectoryPoint Point;

			if (MotionAnim.bLoop)
			{
				FMMPreProcessUtils::ExtractLoopingTrajectoryPoint(Point, Sequence, CurrentTime, MotionMatchConfig->TrajectoryTimes[i]);
			}
			else
			{
				float PointTime = MotionMatchConfig->TrajectoryTimes[i];

				if (PointTime < 0.0f)
				{
					//past Point
					FMMPreProcessUtils::ExtractPastTrajectoryPoint(Point, Sequence, CurrentTime, PointTime,
						MotionAnim.PastTrajectory, MotionAnim.PrecedingMotion);
				}
				else
				{
					FMMPreProcessUtils::ExtractFutureTrajectoryPoint(Point, Sequence, CurrentTime, PointTime,
						MotionAnim.FutureTrajectory, MotionAnim.FollowingMotion);
				}
			}
		
			if (MotionAnim.bFlattenTrajectory)
			{
				Point.Position.Z = 0.0f;
			}

			if (bMirror)
			{
				Point.Position.X *= -1.0f;
				Point.RotationZ *= -1.0f;
			}

			// 将计算出的轨迹点添加到轨迹中
			NewPoseData.Trajectory.Add(Point);
		}

		const FReferenceSkeleton& RefSkeleton = Sequence->GetSkeleton()->GetReferenceSkeleton();

		//Process joints for pose 提取姿势信息
		for (int32 i = 0; i < MotionMatchConfig->PoseBones.Num(); ++i)
		{
			FJointData JointData;

			if (bMirror)
			{
				FName BoneName = MotionMatchConfig->PoseBones[i].BoneName;
				FName MirrorBoneName = MirroringProfile->FindBoneMirror(BoneName);
				
				const int32 MirrorBoneIndex = RefSkeleton.FindBoneIndex(MirrorBoneName);

				FMMPreProcessUtils::ExtractJointData(JointData, Sequence, MirrorBoneIndex, CurrentTime, PoseInterval);

				JointData.Position.X *= -1.0f;
				JointData.Velocity.X *= -1.0f;
			}
			else
			{
				FMMPreProcessUtils::ExtractJointData(JointData, Sequence, MotionMatchConfig->PoseBones[i], CurrentTime, PoseInterval);
			}
			
			// 加入到NewPoseData中
			NewPoseData.JointData.Add(JointData);
		}
		
		// 添加到Poses数组中，也就是完成了数据库中的一个动画帧的预处理
		Poses.Add(NewPoseData);
		CurrentTime += PoseInterval;
	}

	//PreProcess Tags 处理Tag
	for (FAnimNotifyEvent& NotifyEvent : MotionAnim.Tags)
	{
		UTagSection* TagSection = Cast<UTagSection>(NotifyEvent.NotifyStateClass);
		if (TagSection)
		{
			float TagStartTime = NotifyEvent.GetTriggerTime();

			//Pre-process the tag itself
			TagSection->PreProcessTag(MotionAnim, this, TagStartTime, TagStartTime + NotifyEvent.Duration);

			//Find the range of poses affected by this tag
			int32 TagStartPoseId = StartPoseId + FMath::RoundHalfToEven(NotifyEvent.GetTriggerTime() / PoseInterval);
			int32 TagEndPoseId = StartPoseId + FMath::RoundHalfToEven((NotifyEvent.GetTriggerTime() + NotifyEvent.Duration) / PoseInterval);

			TagStartPoseId = FMath::Clamp(TagStartPoseId, 0, Poses.Num());
			TagEndPoseId = FMath::Clamp(TagEndPoseId, 0, Poses.Num());

			TagStartTime = NotifyEvent.GetTriggerTime();
			float TagEndTime = TagStartTime + NotifyEvent.GetDuration();

			//Apply the tags pre-processing to all poses in this range 这里的PreProcessPose又不同的Tag子类实现，如DoNotUse的Tag就是标记这些Pose为DoNotUse
			for (int32 PoseIndex = TagStartPoseId; PoseIndex < TagEndPoseId; ++PoseIndex)
			{
				TagSection->PreProcessPose(Poses[PoseIndex], MotionAnim, this, TagStartTime, TagEndTime);
			}

			continue; //Don't check for a tag point if we already know its a tag section
		}
		
		// TagPoint就是那种只有一个点的动画通知
		UTagPoint* TagPoint = Cast<UTagPoint>(NotifyEvent.Notify);
		if (TagPoint)
		{
			float TagTime = NotifyEvent.GetTriggerTime();
			int32 TagClosestPoseId = StartPoseId + FMath::RoundHalfToEven(TagTime / PoseInterval);
			TagClosestPoseId = FMath::Clamp(TagClosestPoseId, 0, Poses.Num());

			TagPoint->PreProcessTag(Poses[TagClosestPoseId], MotionAnim, this, TagTime);
		}
	}

#endif
}
```

概括PreProcessAnim函数就是先提取基础信息，之后通过定义在 **FMMPreProcessUtils** 中的一系列工具函数提取**速度信息**，**轨迹信息**和**关节信息**，最后**处理Tag**；那么我们也需要了解一下具体是怎样提取这些信息的，以后如果加入我们自己关注的信息，就可以使用类似的方法提取；

## FMMPreProcessUtils中的工具函数

1. 提取根位移速度和根旋转速度：调用了动画序列自带的提取根运动的函数，计算出根位移速度和根旋转速度；
   ![](/article_img/2024-02-21-19-05-09.png)
2. 提取轨迹点位置和朝向，分为三个不同的函数：循环，提取过去的，提取未来的；
   循环就直接根据时间没有调用提取根运动的函数：
   ![](/article_img/2024-02-21-19-08-43.png)
   提取过去的：
   提取未来的：
3. 提取关节数据：
   ![](/article_img/2024-02-21-20-03-47.png)

# AnimNode（Motion Symphony中的动画节点）

Motion Symphony提供了许多AnimNode来实现各种功能，其中最重要的是 **AnimNode_MotionMatching**，**AnimNode_MotionRecorder** 以及 **AnimNode_PoseMatching**，这三个节点在官方案例中均被使用：
![](/article_img/2024-01-23-16-44-54.png)
![](/article_img/2024-01-23-16-45-14.png)
![](/article_img/2024-01-23-16-45-28.png)

## AnimNode_MotionMatching（MM节点）

MM节点继承自 **FAnimNode_AssetPlayerBase**，SequencePlayer和SequenceEvaluator也继承自该类，因此可以感性理解MM节点就是一个复杂的SequenceEvaluator（序列求值器），求出当前要播放哪一帧；
![](/article_img/2024-01-23-17-07-37.png)
继承关系：
![](/article_img/MotionSymphonyClass.png)

所有的AnimNode都继承自 **FAnimNode_Base** 类，其中有几个重要的函数 ：
1. **Initialize_AnyThread** （节点第一次被调用时的初始化操作）
   ![](/article_img/2024-01-24-09-31-13.png)
2. **Update_AnyThread** （图表更新时调用，一般用来计算影响骨骼姿势的权重）
   ![](/article_img/2024-01-24-09-31-01.png)
3. **Evaluate_AnyThread** （根据update中计算出的权重估计本地空间下的骨骼变换）
   ![](/article_img/2024-01-24-09-30-51.png)
我们看任何一个AnimNode的代码都可以从这几个函数入手。

AnimNode_MotionMatching还覆写了几个其他的 **FAnimNode_Base** 函数，如果用到我们后面再提；
![](/article_img/2024-01-24-09-22-48.png)
注意到MM节点没有覆写 **Update_AnyThread** 因为MM节点继承自 **FAnimNode_AssetPlayerBase**，该父类将 **Update_AnyThread** 定义为了final，无法覆写：
![](/article_img/2024-01-24-09-27-28.png)
MM节点的主要逻辑都写在 **UpdateAssetPlayer** 中，其中关键的函数及调用如下图所示：
![](/article_img/MMFunc.png)

下面我们阅读一下几个重要函数的代码逻辑：

首先Motion Matching算法是基于当前姿势去进行匹配的，因此需要得到当前姿势，这里使用 **ComputeCurrentPose** 函数得到当前姿势，要注意的是，Motion Symphony在实现时使用了两个变量来表示当前姿势：
1. **CurrentInterpolatedPose**：从FAnimNode_MotionRecorder中得到记录的当前姿势；
2. **CurrentChosenPoseId**：通过与上次MM之间的时间间隔计算出的当前已选择的姿势id；之后如果进行MM得到的匹配结果也要赋值给该变量；

下面看看源码（省略了大量代码，只展示最核心的代码）：
```C++
void FAnimNode_MotionMatching::ComputeCurrentPose(const FCachedMotionPose& CachedMotionPose)
{
	const float PoseInterval = FMath::Max(0.01f, MotionData->PoseInterval);

	//====== Determine the next chosen pose ========
	FAnimChannelState& ChosenChannel = BlendChannels.Last();

	float TimePassed = TimeSinceMotionChosen;
	int32 PoseIndex = ChosenChannel.StartPoseId;

	int32 NumPosesPassed = 0;
	if (TimePassed < 0.0f){
		NumPosesPassed = FMath::CeilToInt(TimePassed / PoseInterval);
	}
	else{
		NumPosesPassed = FMath::FloorToInt(TimePassed / PoseInterval);
	}
	// 计算得到当前已选择的姿势id
	CurrentChosenPoseId = PoseIndex + NumPosesPassed;

	//====== Determine the next dominant pose ========
    // 对当前记录的姿势进行轨迹插值，因为正在播放的帧有可能是插值出来的，轨迹信息没有经过预处理得到
	FMotionMatchingUtils::LerpPoseTrajectory(CurrentInterpolatedPose, *BeforePose, *AfterPose, PoseInterpolationValue);
    // 把FAnimNode_MotionRecorder中记录的姿势赋值给CurrentInterpolatedPose
	for (int32 i = 0; i < PoseBoneRemap.Num(); ++i){
		const FCachedMotionBone& CachedMotionBone = CachedMotionPose.CachedBoneData[PoseBoneRemap[i]];
		CurrentInterpolatedPose.JointData[i] = FJointData(CachedMotionBone.Transform.GetLocation(), CachedMotionBone.Velocity);
	}
}
```
清楚了当前姿势是如何得到的我们就可以来看MM算法的“主函数”**UpdateMotionMatching**了，大致流程就是先得到当前姿势，之后进行**SchedulePoseSearch**；
```C++
void FAnimNode_MotionMatching::UpdateMotionMatching(const float DeltaTime, const FAnimationUpdateContext& Context)
{
	bForcePoseSearch = false;
	TimeSinceMotionChosen += DeltaTime;
	TimeSinceMotionUpdate += DeltaTime;

    // 得到负责记录姿势的MotionRecoredNode
	FAnimNode_MotionRecorder* MotionRecorderNode = Context.GetAncestor<FAnimNode_MotionRecorder>();

	if (MotionRecorderNode){   
		// 得到顺序播放情况下当前已选择的姿势CurrentChosenPoseId，从MotionRecoredNode中得到当前姿势CurrentInterpolatedPose
		ComputeCurrentPose(MotionRecorderNode->GetMotionPose());
	}
	else{
		ComputeCurrentPose();
	}

	//If we have ran into a 'DoNotUse' pose. We need to force a new pose search
	if(CurrentInterpolatedPose.bDoNotUse){
		bForcePoseSearch = true;
	}

	UMotionMatchConfig* MMConfig = MotionData->MotionMatchConfig;

	//Past trajectory mode
	if (PastTrajectoryMode == EPastTrajectoryMode::CopyFromCurrentPose){
		for (int32 i = 0; i < MMConfig->TrajectoryTimes.Num(); ++i){
			if (MMConfig->TrajectoryTimes[i] > 0.0f){ 
				break;
			}
			DesiredTrajectory.TrajectoryPoints[i] = CurrentInterpolatedPose.Trajectory[i];
		}
	}

    // 上次MM经过更新时间间隔或者强制进行MM
	if (TimeSinceMotionUpdate >= UpdateInterval || bForcePoseSearch){
        // 重置MM更新时间
		TimeSinceMotionUpdate = 0.0f;
        // 姿势匹配
		SchedulePoseSearch(DeltaTime, Context);
	}
}
```
姿势匹配函数 **SchedulePoseSearch**：首先根据当前已选择的姿势得到下一帧姿势（我们更希望动画连续，也就是尽可能顺着当前动画播放，会在计算Cost更偏向下一帧姿势，使下一帧姿势的Cost更小），开始计算Cost，得到Cost最小的姿势，判断该姿势是否是 **FAnimNode_MotionRecorder** 记录的当前姿势（CurrentInterpolatedPose），以及是否是当前已选择的姿势（CurrentChosenPoseId），都不是就过渡到该Cost最小的姿势；
```C++
void FAnimNode_MotionMatching::SchedulePoseSearch(float DeltaTime, const FAnimationUpdateContext& Context)
{
	FPoseMotionData& NextPose = MotionData->Poses[MotionData->Poses[CurrentChosenPoseId].NextPoseId];

	int32 LowestPoseId = NextPose.PoseId;

	switch (PoseMatchMethod){   // 根据搜索方式，分为优化和线性（优化模式不会搜索全部数据库），搜索数据库得到Cost最小的姿势id
		case EPoseMatchMethod::Optimized: { LowestPoseId = GetLowestCostPoseId(NextPose); } break;
		case EPoseMatchMethod::Linear: { LowestPoseId = GetLowestCostPoseId_Linear(NextPose); } break;
	}

	FPoseMotionData& BestPose = MotionData->Poses[LowestPoseId];
	FPoseMotionData& ChosenPose = MotionData->Poses[CurrentChosenPoseId];

	bool bWinnerAtSameLocation = BestPose.AnimId == CurrentInterpolatedPose.AnimId &&
								 BestPose.bMirrored == CurrentInterpolatedPose.bMirrored &&
								FMath::Abs(BestPose.Time - CurrentInterpolatedPose.Time) < 0.25f
								&& FVector2D::DistSquared(BestPose.BlendSpacePosition, CurrentInterpolatedPose.BlendSpacePosition) < 1.0f;
    // 判断是否时ChosenPose（当前被选择的姿势）
	if (!bWinnerAtSameLocation){
		bWinnerAtSameLocation = BestPose.AnimId == ChosenPose.AnimId &&
								BestPose.bMirrored == ChosenPose.bMirrored &&
								FMath::Abs(BestPose.Time - ChosenPose.Time) < 0.25f
								&& FVector2D::DistSquared(BestPose.BlendSpacePosition, ChosenPose.BlendSpacePosition) < 1.0f;
	}
    // 不是ChosenPose，也不是CurrentInterpolatedPose，过渡到该姿势
	if (!bWinnerAtSameLocation){
		TransitionToPose(BestPose.PoseId, Context);
	}
}
```
接下来，我们就可以看MM算法的核心，匹配算法的实现了，在 **GetLowestCostPoseId** 和 **GetLowestCostPoseId_Linear** 这两个函数中，他们区别不大，唯一的区别就是是否对数据库进行了筛选，因此我们只看 **GetLowestCostPoseId** 即可，同样省略了一些非核心代码；

```C++
int32 FAnimNode_MotionMatching::GetLowestCostPoseId(FPoseMotionData& NextPose)
{
    // 得到计算权重
	FCalibrationData& FinalCalibration = FinalCalibrationSets[RequiredTraits];
    // 得到候选姿势
	TArray<FPoseMotionData>* PoseCandidates = 
	MotionData->OptimisationModule->
	GetFilteredPoseList(CurrentInterpolatedPose, RequiredTraits, FinalCalibration);

	if (!PoseCandidates)
	{   // 没有候选姿势，也就是没有配置优化（后面讲），就线性搜索数据库
		return GetLowestCostPoseId_Linear(NextPose);
	}

	int32 LowestPoseId = 0;
	float LowestCost = 10000000.0f;
    // 遍历候选姿势，开始计算cost
	for (FPoseMotionData& Pose : *PoseCandidates)
	{
		//Body Momentum
		float Cost = FVector::DistSquared(CurrentInterpolatedPose.LocalVelocity, Pose.LocalVelocity) * FinalCalibration.Weight_Momentum;

		//Body Rotational Momentum
		Cost += FMath::Abs(CurrentInterpolatedPose.RotationalVelocity - Pose.RotationalVelocity)
			* FinalCalibration.Weight_AngularMomentum;

		//Trajectory Cost 
		const int32 TrajectoryIterations = FMath::Min(DesiredTrajectory.TrajectoryPoints.Num(), FinalCalibration.TrajectoryWeights.Num());
		for (int32 i = 0; i < TrajectoryIterations; ++i)
		{
			const FTrajectoryWeightSet WeightSet = FinalCalibration.TrajectoryWeights[i];
			const FTrajectoryPoint CurrentPoint = DesiredTrajectory.TrajectoryPoints[i];
			const FTrajectoryPoint CandidatePoint = Pose.Trajectory[i];

			Cost += FVector::DistSquared(CandidatePoint.Position, CurrentPoint.Position) * WeightSet.Weight_Pos;
			Cost += FMath::Abs(FMath::FindDeltaAngleDegrees(CandidatePoint.RotationZ, CurrentPoint.RotationZ)) * WeightSet.Weight_Facing;
		}
        // Pose Cost
		for (int32 i = 0; i < CurrentInterpolatedPose.JointData.Num(); ++i)
		{
			const FJointWeightSet WeightSet = FinalCalibration.PoseJointWeights[i];
			const FJointData CurrentJoint = CurrentInterpolatedPose.JointData[i];
			const FJointData CandidateJoint = Pose.JointData[i];

			Cost += FVector::DistSquared(CurrentJoint.Velocity, CandidateJoint.Velocity) * WeightSet.Weight_Vel;
			Cost += FVector::DistSquared(CurrentJoint.Position, CandidateJoint.Position) * WeightSet.Weight_Pos;
		}

		//Favour Current Pose 如果是顺序播放时当前姿势的下一个姿势，就乘上一个Favour值，让其cost更小（这里默认值是0.95）
		if (bFavourCurrentPose && Pose.PoseId == NextPose.PoseId)
		{
			Cost *= CurrentPoseFavour;
		}

		//Apply Pose Favour
		Cost *= Pose.Favour;

		if (Cost < LowestCost)
		{
			LowestCost = Cost;
			LowestPoseId = Pose.PoseId;
		}
	}

	return LowestPoseId;
}
```

### 匹配算法

看完**GetLowestCostPoseId**函数，我们可以就总结出Motion Symphony使用的匹配算法了！

![](/article_img/2024-01-24-20-32-52.png)

# Debug工具

Motion Symphony提供了一系列Debug工具，大多是通过命令行开启后在视口中打印出相关数据或者绘制出相应轨迹。下面看几个常用的Debug工具如何开启以及如何在代码中如何实现；

## Debugging the Trajectory（轨迹线绘制）

![](/article_img/2024-01-24-16-19-52.png)
开启后效果，红色为匹配姿势的轨迹，绿色为输入轨迹：
![](/article_img/2024-01-24-16-19-20.png)

轨迹线绘制实现在**UpdateAssetPlayer**函数中，如下：
```C++
//Visualize the trajectroy debugging
const int32 TrajDebugLevel = CVarMMTrajectoryDebug.GetValueOnAnyThread();

if (TrajDebugLevel > 0)
{
    if (TrajDebugLevel == 2)
    {
        //Draw chosen trajectory
        DrawChosenTrajectoryDebug(Context.AnimInstanceProxy);
    }

    //Draw Input trajectory
    DrawTrajectoryDebug(Context.AnimInstanceProxy);
}
```
## Debugging the Pose（绘制Pose位置和速度）
![](/article_img/2024-01-24-16-23-49.png)
开启后效果：
![](/article_img/2024-01-24-16-24-48.png)
PoseDebug实现在UpdateAssetPlayer函数中，如下：
```C++
int32 PoseDebugLevel = CVarMMPoseDebug.GetValueOnAnyThread();

if (PoseDebugLevel > 0)
{
    DrawChosenPoseDebug(Context.AnimInstanceProxy, PoseDebugLevel > 1);
}

//Debug the current animation data being played by the motion matching node
int32 AnimDebugLevel = CVarMMAnimDebug.GetValueOnAnyThread();

if(AnimDebugLevel > 0)
{
    DrawAnimDebug(Context.AnimInstanceProxy);
}
```
## Animation Info Debugging（当前姿势的相关信息）
![](/article_img/2024-01-24-16-29-13.png)
开启后效果：
![](/article_img/2024-01-24-16-28-10.png)
也实现在UpdateAssetPlayer函数中：
```C++
//Debug the current animation data being played by the motion matching node
int32 AnimDebugLevel = CVarMMAnimDebug.GetValueOnAnyThread();

if(AnimDebugLevel > 0)
{
    DrawAnimDebug(Context.AnimInstanceProxy);
}
```

## Search / Optimization Debugging

显示候选姿势的个数，并绘制所有候选姿势的轨迹；也可以查看使用了优化手段后与使用线性搜索之间的误差；
![](/article_img/2024-01-24-16-33-52.png)
开启后效果：
![](/article_img/2024-01-24-16-35-45.png)

## Cost Debugging

许多其他的MM方案会提供Cost的debug工具来显示所有姿势的Cost值，Motion Symphony没有提供类似的工具，无法直接查看每个候选姿势的Cost是多少；之后可以仿照上面的其他Debug工具的写法添加一个。

# 优化策略

Motion Symphony的优化策略可以分为两类，一类是处理数据集，为数据集中的每个姿势定义一个候选姿势集，减少要搜索的数据数量；另一类是在逻辑中提前结束搜索；

处理数据集的优化策略都依靠在Motion Data中配置从而在预处理Motion Data时实现候选姿势集的构建，因此定义了几个资产来表示不同的优化策略，分别是**MMOptimisation_MultiClustering**，**MMOptimisation_TraitsBin**和 **MMOptimisation_LayeredAABB**（在编辑器中没找到，应该是功能还没完善），他们均继承自类 **UMMOptimisationModule**；

## K-means Clustering

K-means是一种经典的机器学习分类算法，核心目标是将给定的数据集划分成K个簇（K是超参），并给出每个样本数据对应的中心点。
在Motion Symphony中，使用K-means算法将相同Traits划分后的姿势数据集按照**轨迹**分为K个簇，在匹配过程中只搜索与当前姿势在同一个簇中的姿势。

Motion Symphony定义了一种资产来实现K-means算法：**MMOptimisation_MultiClustering**；其中定义了K-means算法的分簇数，最大迭代次数和期望的查询表的大小，该表中存放一个包含候选姿势集的数组，Desired Lookup Table Size就是这个数组的大小，也是最终姿势数据库会被划分的簇数；
![](/article_img/2024-01-24-19-26-14.png)
核心函数是 **BuildOptimisationStructures**，该函数是父类 **UMMOptimisationModule** 中定义的虚函数；
```C++
void UMMOptimisation_MultiClustering::BuildOptimisationStructures(UMotionDataAsset* InMotionDataAsset)
{
	Super::BuildOptimisationStructures(InMotionDataAsset);

	//First create trait bins with which to cluster on. 相当于先按照Traits划分了一次
	TMap<FMotionTraitField, TArray<FPoseMotionData> > PoseBins;

	for (FPoseMotionData& Pose : InMotionDataAsset->Poses)
	{
		TArray<FPoseMotionData>& PoseBin = PoseBins.FindOrAdd(Pose.Traits);
		PoseBin.Add(FPoseMotionData(Pose));
	}

	//For each trait bin we need to cluster and create a lookup table
	for (auto& TraitPoseSet : PoseBins)
	{
		FCalibrationData FinalPreProcessCalibration = FCalibrationData();
		FinalPreProcessCalibration.GenerateFinalWeights(InMotionDataAsset->PreprocessCalibration, 
			InMotionDataAsset->FeatureStandardDeviations[TraitPoseSet.Key]);

#if WITH_EDITORONLY_DATA
		KMeansClusteringSet.Clear();
#else
		FKMeansClusteringSet KMeansClusteringSet = FKMeansClusteringSet();
#endif
        // K-Means算法划分
		KMeansClusteringSet.BeginClustering(TraitPoseSet.Value, FinalPreProcessCalibration, KMeansClusterCount, KMeansMaxIterations, true);

		FPoseLookupTable& PoseLookupTable = PoseLookupSets.FindOrAdd(TraitPoseSet.Key);
        // 在LookupTable中再做一次K-Means
		PoseLookupTable.Process(TraitPoseSet.Value, KMeansClusteringSet, FinalPreProcessCalibration,
			DesiredLookupTableSize);

		//Set the candidate set Id for each pose that is able to be looked up.
		for (int32 i = 0; i < PoseLookupTable.CandidateSets.Num(); ++i)
		{
			FPoseCandidateSet& CandidateSet = PoseLookupTable.CandidateSets[i];
			CandidateSet.SetId = i;

			for (FPoseMotionData& Pose : CandidateSet.PoseCandidates)
			{   // 为每个姿势分配其候选集
				InMotionDataAsset->Poses[Pose.PoseId].CandidateSetId = i;
			}
		}
	}
}
```
![](/article_img/2024-01-24-20-02-49.png)
其中 **BeginClustering** 函数调用了 bool FKMeansClusteringSet::ProcessClusters(TArray<FPoseMotionData>& Poses)函数，其中可见划分方式是按照轨迹距离划分，**也就是每个姿势的候选匹配姿势集中都是与当前姿势轨迹接近的姿势**；


## TraitsBin

只在需要的Traits里搜索；例如在MM节点里设置需要的Traits为 Walk，那么就只会在Tag被设置为Walk的姿势中搜索；**DoNotUse** Tag原理相同；

```C++
TArray<FPoseMotionData>* UMMOptimisation_TraitBins::GetFilteredPoseList(const FPoseMotionData& CurrentPose, 
	const FMotionTraitField RequiredTraits, const FCalibrationData& FinalCalibration)
{
	if (PoseBins.Contains(RequiredTraits))
	{
		return &PoseBins[RequiredTraits].Poses;
	}

	return nullptr;
}
```


# Motion Symphony 2.0

## Motion Calibration变为optional

在2.0版本中，Motion Calibration变成了可选择的，不是必须的，之前1.0中需要在Motion Calibration中配置权重，而在2.0中权重被定义在Motion Config中；
![](/article_img/2024-04-03-09-55-36.png)

## 输入数据更加通用

在1.0中，mm需要输入一个**trajectory**，而在2.0中这个变量变成了更通用的**input data**，称为“FMotionMatchingInputData”，本质上是一个浮点数组的包装器，其中包含来自所有“输入响应”类型匹配功能的所有数据；
![](/article_img/2024-04-03-10-03-49.png)
使用MMBlueprint Function Library中的函数 **Construct Motion Input Feature Array** 填充Input Data。

函数中会根据MotionConfig中选择的特征数据，调用各特征子类的**SourceInputData**函数：
```C++
int32 FeatureOffset = 0;
for(TObjectPtr<UMatchFeatureBase> MatchFeature : MotionConfig->InputResponseFeatures)
{
	if(MatchFeature && MatchFeature->IsSetupValid())
	{
		MatchFeature->SourceInputData(InputData.DesiredInputArray, FeatureOffset, Actor);
		FeatureOffset += MatchFeature->Size();
	}
	else
	{
		UE_LOG(LogTemp, Error, TEXT("ERROR: 'ConstructMotionInputFeatureArray' node -  Match feature has an invalid setup and cannot be processed."))
	}
}
```
如下图，有这些特征类，均继承自MatchFeatureBase类，都重载SourceInputData函数实现对InputData填充不同的数据：
![](/article_img/2024-04-03-10-42-03.png)
如MatchFeature_Trajectory2D类中的覆写如下，从TrajectoryGenerator中得到轨迹并填充InputData：
```C++
if(UTrajectoryGenerator_Base* TrajectoryGenerator = InActor->GetComponentByClass<UTrajectoryGenerator_Base>())
{
	const FTrajectory& Trajectory = TrajectoryGenerator->GetCurrentTrajectory();

	const int32 Iterations = FMath::Min(TrajectoryTiming.Num(), Trajectory.TrajectoryPoints.Num());
	
	for(int32 i = 0; i < Iterations; ++i)
	{
		const FTrajectoryPoint& TrajectoryPoint = Trajectory.TrajectoryPoints[i];

		FVector RotationVector = FQuat(FVector::UpVector,
			FMath::DegreesToRadians(TrajectoryPoint.RotationZ)) * FVector::ForwardVector;
		RotationVector = RotationVector.GetSafeNormal() * 100.0f;

		const int32 PointOffset = FeatureOffset + (i * 4.0f);

		if(PointOffset + 3 >= OutFeatureArray.Num())
		{
			UE_LOG(LogTemp, Error, TEXT("UMatchFeature_Trajectory2D: SourceInputData(...) - Feature does not fit in FeatureArray"));
			return;
		}
		
		OutFeatureArray[PointOffset] = TrajectoryPoint.Position.X;
		OutFeatureArray[PointOffset + 1] = TrajectoryPoint.Position.Y;
		OutFeatureArray[PointOffset + 2] = RotationVector.X;
		OutFeatureArray[PointOffset + 3] = RotationVector.Y;
	}
}
```

## 不需要配置优化策略

1.0中可以配置优化模块，例如cluster或AABB等，在2.0中废弃了这一功能，默认使用AABB进行搜索优化；

## 不能显式计算各个特征Cost

在2.0中由于特征可选择的更多，因此不能确定选择了那些特征，在计算cost时也就不能显示得到如轨迹cost，姿势cost等信息，而是统一对特征数组进行计算；
