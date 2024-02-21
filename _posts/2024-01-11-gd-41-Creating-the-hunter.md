---
title: 41Creating the hunter
categories: [godot,The ultimate introduction to Godot 4]
tags: [godot,The ultimate introduction to Godot 4]
date: 2024-01-11 23:30
---

创建一个hunter场景

## hunter：
- 功能：hunter一只蜘蛛（为什么叫hunter呢）。基本的功能和Bug差不多
    - 1.player进入边界hunter用寻路算法爬(骨骼动画)向player
    - 2.player进入到它的攻击区域开咬，被咬后health-20
    - 3.hunter被攻击 health-10,health<=0销毁,hunter也是被限制不能被连续攻击的(hit)
- 知识点：这节包括两个概念:Pathfinding(路)、Bone animations(骨骼动画)

## Pathfinding:
- NavigationRegion2D:绘制路径区域
    - Navigation Layers设置那个layer的对象在此导航
- NavigationAgent2D:在NavigationRegion2D的区域执行导航
    - 用到的属性
        - Navigation Layers设置那个layer的对象在此执行导航
        - path_desired_distance：：距离阈值，下一个路径位置的距离（相当于步长？）
        - target_desired_distance：距离阈值，用于确定是否已到达最终目标点。 
    - 实现细节
    
    ```python
    #计算到目标对象的导航点
    $NavigationAgent2D.target_position = targetNode.posistion #由于这个调用会导致大量的计算，他有整个Timer每2秒设置一次

    #实质的移动。赋值了 target_position godot实际就已经计算出了一系列的移动坐标点，通过get_next_path_position()还准备好了下一步移动坐标
    # 有了下一步移动坐标根据自身当前坐标就可以获取到移动方向了
    func _physics_process(delta):#由于导航是一个物理框架里的东西，所以下面的代码要在_physics_process里处理
        var next_path_pos: Vector2 = $NavigationAgent2D.get_next_path_position()
        var direction=(next_path_pos - global_position).normalized()
        velocity = speed *  direction
        move_and_slide()
        look_at( direction.angle()+ PI/2) #旋转4/1角度朝向player
    ```

## skeleton animation:核心思想单独围绕身体的某个部位旋转
- 简单总结下：基本原理就是将身体分成N块Bone，那么Bone可以绕着某一个点来运动(要动起来还没那么简单需要用AnimationPlayer来控制Marker2D)。哪么两个相连接的Bone又按照怎样的轨迹来运动呢？这就取决于选择的`SkeletonModification`(比如实例里的SkeletonModification2DTwoBoneIK:两个骨骼按照余弦定理进行旋转)。实例的四腿蜘蛛，他将一条腿分成两Bone（claw链接在leg上），那么要让蜘蛛爬起来还得给四条腿用AnimationPlayer的4条claw的定位Marker2d点移动起来。（非常琐碎，按他的方法要制造一个完整的骨骼动画复杂的模型可能一天时间都消耗）

### 一条腿的制作步骤
- 1.添加一个`Skeleton2D`节点，添加子节点躯干(Torso)Bone2D来链接腿，这个Bone下放上躯干Sprite2D
- 2.在`Torso Bone`下添加子节点前右腿Bone2D(FrontRightLeg),这个节点下放上前右腿Sprite2D
    - ①<a id="section_41_01">设置Bone2D的`Auto Calculate Length and Angle`，调整`Length`和`Angle`链接`leg`两点</a>
- 3.在`FrontRightLeg`下添加子节点钳子Bone2D(FrontRightClaw)，这个节点放上Claw Sprite2D
    - ①参考[2.①](#section_41_01)
- 4.在`FrontRightLeg`的尾端添加Marker2D节点`FrontRight`
- 5.让`FrontRightLeg`和`FrontRightClaw`跟随Marker2D节点`FrontRight`运动：
    - `Skeleton2D`节点 -> Modification Stack -> SkeletonModificationStack2D -> 展开
    - Modifications - Add Element -> 选择[SkeletonModification2DTwoBoneIK](https://docs.godotengine.org/en/stable/classes/class_skeletonmodification2dtwoboneik.html)-> 展开
    - 选择 Target Nodepath=FrontRight(Marker2D)
    - 选择  Joint One Bone 2D Node =FrontRightLeg
    - 选择  Joint Two Bone 2D Node =FrontRightClaw(没有再多了只能链接两个Bone 😄)
    - `Flip bend direction`钩上(该修改中的骨骼在收缩时将向外弯曲，而不是向内弯曲。)
- 6.让前右腿爬起来（说是Skeleton animation还是要用到`AnimationPalyer`来驱动它）
    - 用AnimationPalyer在固定时间序列里通过4个Key调整Marker2D节点`FrontRight`的`Position`

- 小结：实际他讲的是[cutout animation](https://docs.godotengine.org/en/stable/tutorials/animation/cutout_animation.html#doc-cutout-animation),讲的这么复杂是他没用编辑器将Nodes转换成Bone2D节点(🦴 -> Make Bone2D(Nodes) from Nodes)，造成节点创建时层次结构复杂。

- 编辑器的bug:Skeleton2D绑定完Marker2D和Bone2D节点后，所有节点突然出现错乱问题:反选下`Auto Calculate Length and Angle`在恢复