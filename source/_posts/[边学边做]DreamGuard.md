---
title: 【边学边做】DreamGuard（New Input System的使用方法）
date: 2023-03-22
index_img: "/img/bg/Logo.jpg"
tags: 【边学边做】
categories: 
   -游戏Demo
---

记录学习unity开发中的各种问题和常见解决方法
<!-- more -->

# 第三人称角色控制器

在开发DreamGuard的时候，原本打算使用unity自带的第三人称模板，但是发现其实现是基于New Input System，阅读代码出现困难，就想着自己实现一下。

## New Input System

[官方文档](https://docs.unity3d.com/Packages/com.unity.inputsystem@1.5/manual/index.html)

[视频教程1](https://www.bilibili.com/video/BV1tg411j7ni/?spm_id_from=333.880.top_right_bar_window_custom_collection.content.click&vd_source=93b215eab72b2548f75d0772e28f8b20)

[视频教程2](https://www.bilibili.com/video/BV17h4y1W7Z2/?spm_id_from=333.337.search-card.all.click&vd_source=93b215eab72b2548f75d0772e28f8b20)

我对New Input System的理解是unity希望规范开发者对用户输入的处理。New Input System除了实现老版本输入系统的功能外，主要还方便了用户输入设备的切换（如从键盘切换到手柄）以及方便对不同输入模式的切换（如在游戏中空格代表跳跃，在ui界面中空格代表确定）

### 新输入系统的几种用法

![](/article_img/2023-03-22-12-13-16.png)

按照官网的介绍，新输入系统有四种用法，第一种是和老版本输入系统类似的写法；后面三种都需要借助Actions Asset，可以在Project窗口右键创建，可以将其理解为配置输入的界面，通过Actions Asset就可以可视化的将输入分类，实现输入设备的切换和输入模式的切换。同时在配置Actions Asset时可以选择生成一个C#类来方便在代码中直接使用Actions Asset。

![](/article_img/2023-08-12-12-34-07.png)

Action Maps方便区分输入场景，比如游戏中的输入和UI界面的输入；
Move，Look等等就是接收具体的输入了，input system可以方便的设置不同的输入设备并在其中切换。

### Using an Actions Asset and a PlayerInput component

unity自带的第三人称模板使用的是第四种，这是一种我认为较为直观的使用方法，借助PlayerInput组件，可以选择要使用的Actions Asset以及输入时的行为，下图中的Behavior。

![](/article_img/2023-03-22-12-25-07.png)

第三人称模板使用的Behavior是[Send Messages](https://docs.unity3d.com/ScriptReference/GameObject.SendMessage.html)，这个方法是最简单的物体间的通信方法，他会将信息传递给该物体上附加的所有脚本；[Broadcast Messages](https://docs.unity3d.com/ScriptReference/GameObject.BroadcastMessage.html)会将信息传递给附加到该游戏对象及其所有子对象的每个脚本；[Invoke Unity Events](https://docs.unity3d.com/ScriptReference/Events.UnityEvent.html)对每个单独的消息类型使用单独的 UnityEvent；Invoke C Sharp Events和Invoke Unity Events类似，只是会触发一个C#事件。

由于Send Messages效率较低（？还不太懂为啥），所以我自己实现的第三人称控制器使用Invoke Unity Events方法。

这种方法需要配置Events，只需要为每个输入消息配置一个函数，就可以实现当有输入时，直接调用这个函数，使用起来十分方便。

![](/article_img/2023-03-22-12-49-01.png)

### Using an Actions Asset

这种方法就对代码更加友好，其实和上述的第四种用法一样，需要勾选生成C#类选项：
![](/article_img/2023-08-11-20-23-58.png)
之后在需要使用输入系统的脚本中就可以使用这个类了：
```Csharp
public class PlayerController : MonoBehaviour
{
   public PlayerInputControl inputControl; // 声明生成的Actions Asset类
   private void Awake(){
      inputControl = new PlayerInputControl();
   }
   void Update(){
      // Gameplay是设定的Action Maps，Move是设置的监听移动的Actions
      inputDirection = inputControl.Gameplay.Move.ReadValue<Vector2>();
   }
}
```


## 脚本实现

第三人称控制器需要相机根据玩家鼠标的移动来回旋转，角色的移动是相对与玩家视角的移动（当玩家看向哪，前进方向就是哪），角色的旋转则永远指向角色的移动方向（不会随相机视角的旋转而旋转）。

### 相机旋转

这里的相机使用了Cinemachine系统中的virtual camera，其可以实现跟随角色以及看向角色。

![](/article_img/2023-03-22-12-56-23.png)

由于其会一直看向Look At设置的对象，因此要旋转相机只需要旋转相机看向的这个对象。在角色对象上添加一个空的子物体cameraLookAt，作为相机看向的方向。

![](/article_img/2023-03-22-13-00-16.png)

在Actions Asset中设置输入，获取玩家鼠标的与上一帧的相对移动，由于是相对移动，直接将这个值不断累加并且映射到角度值，就可以得到cameraLookAt这个子物体的旋转值。注意只有在用户有输入时才改变旋转值，否则会导致用户没有输入时旋转值也为0，由旋转回默认方向。

```Csharp
    private void CameraRotate()
    {
        if (lookDirection.sqrMagnitude > 0.1f)
        {
            cameraPitch += lookDirection.y;
            cameraYaw += lookDirection.x;
        }
        cameraPitch = ClampAngle(cameraPitch, -70f, 70f);
        cameraYaw = ClampAngle(cameraYaw, float.MinValue, float.MaxValue);

        cameraLookAt.transform.rotation = Quaternion.Euler(cameraPitch, cameraYaw, 0);
    }
```

### 角色移动和旋转

角色移动依靠Character Controller实现。

根据需要，角色需要是相对相机视角的移动，因此不能直接使用输入系统获取的相对与世界坐标系的方向，要将该方向变换到相机坐标系中。由于角色只在XZ平面移动，因此这种变换可以用欧拉角旋转实现。

先计算输入应该使角色旋转多少°（即相对与相机的旋转角度），再将这个值和相机旋转相加，得到世界坐标中应该旋转的值（因为Character Controller中的Move方法是相对与世界坐标系定义的）。这里的mainCamera就是现在游戏中正在起作用的相机!

```Csharp
   targetRotation = Mathf.Atan2(moveDirection.x, moveDirection.z) * Mathf.Rad2Deg + mainCamera.transform.rotation.eulerAngles.y;
   // mainCamera = GameObject.FindGameObjectWithTag("MainCamera");在awake中得到
```

有了最关键的移动方向，角色旋转和移动就很好实现了。

### Bug

在一开始的实现中，我将角色移动和旋转和相机旋转都放进了Update中，产生了奇怪的bug，发现原因是角色的旋转会带动其子物体cameraLookAt的旋转，导致角色旋转会带动玩家视角旋转，天旋地转。因此相机的旋转应该不受角色旋转的影响，只受玩家鼠标移动的控制。但是unity中子物体必然跟随父物体旋转，一个思路就是在角色旋转后，将相机再旋转回去，由于我们的旋转都是相对与世界坐标系的旋转，因此可以相机旋转放在LateUpdate中，在角色旋转后，再对相机进行旋转（这里容易产生误解，因为旋转是一个变换，其实在实现一个旋转时，每一帧都是从初始位置开始旋转的，只是旋转的角度不断增加产生旋转的效果，**角色旋转后再对相机旋转，不需要向反方向旋转，因为相机还是从初始位置开始旋转，旋转的角度完全由鼠标移动控制**）

### 角色跳跃

角色跳跃基于Character Controller组件的isGrounded，当角色在地面时才能跳跃，给予角色一个向上的速度即可，这个速度通过 **v^2 = 2gh** 计算，这样可以方便控制角色跳跃的高度。

Character Controller并没有实现角色重力，要自己实现，只需要检测当角色不在地面时，给角色一个向下的速度，这个速度由 **v=gt** 累加计算得来。

```Csharp
if(characterController.isGrounded){
   if(verticalSpeed < -1) verticalSpeed = -1;  // 当角色落地后将竖直速度置零
}else{
   verticalSpeed += gravity * Time.deltaTime;
}
```

### 地面检测

地面检测可以使用函数 **Physics2D** 进行检测：
```Csharp
public class physicsCheck ： MonoBehaviour{
   public bool isGround;

   public LayerMask groundLayer; // 指定对哪一些layer检测
   public float checkRaduis;
   public Vector2 bottomOffset;

   void Update(){
      Check();
   }
   public void Check(){
      isGround = Physics2D.OverlapCircle((Vector)transform.position + bottomOffset, checkRadius, groundLayer);
   }
   private void OnDrawGizmoSelected(){
      Gizmos.DrawWireSphere((Vector2)transform.postion + bottomOffset, checkRaduis);
   }
}
```
通过改变Offset可以进行任何检测，比如检测角色是否碰到了墙。