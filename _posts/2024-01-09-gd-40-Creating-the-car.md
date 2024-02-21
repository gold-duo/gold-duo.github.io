---
title: 40Creating the car
categories: [godot,The ultimate introduction to Godot 4]
tags: [godot,The ultimate introduction to Godot 4]
date: 2024-01-09 18:50
---

创建敌人系统里的car

##  car特点
- 1.car按[指定的路线运行](#section_40_01)(这更像是火车)
- 2.player进入边界car上的两杆枪瞄准player，[激光器会变粗](#section_40_02)（两条红外线瞄准，玩家一进入就有个光线从无到粗到无的效果循环）
- 5.[瞄准结束后开火](#section_40_03)

## 新的节点
### <a id="section_40_01">Path2D节点</a>:可以让 PathFollow2D 子节点沿着 Curve2D 移动。(通过Path2D绘制的曲线就给子节点PathFollo2D制造了一条跟随路线)
- PathFollow2D这个节点专为Path2D服务的。
    - 有两个比较重要的属性
        - 1.Progress进度(围绕Path2D绘制的曲线)
        - 2.ProgressRatio进度比率(围绕Path2D绘制的曲线)
        
        ```python
        #实例中汽车的不停绕着路线图运动就是通过修改progress_ratio
        progress_ratio +=delta*0.05
        ```
    - 在PathFollow2D节点添加Sprite2D节点那么该节点也是按照Path2D的路线运动的。
### RayCast2D(光线投射节点):2D空间中的射线，用于查找第一个相交(碰撞)的 CollisionObject2D。（实例中这个东西没有用到什么东西）
- `get_collider()`:返回被投射到的对象
- `target_position`:光线的目标点，相对于RayCast2D的position。

### Line2D:绘制线条
    - `add_point()`：添加相对于多段线自身位置具有指定位置的点。如果未提供索引，则新点将添加到点数组的末尾。如果给出索引，则在索引索引标识的现有点之前插入新点。新点之后的点的指数增加 1。提供的索引不得超过多段线中现有点的数量。

## 细节
- <a id="section_40_02">瞄准效果</a>：这里用AnimationPlayer改变Line2D的width（从0-10-0）

- <a id="section_40_03">开火的实现</a>:
    - 在枪口放火焰纹理，普通状态 modulate:a的值为0
    - 瞄准后 modulate:a=1
    - 做tween动画 将 modulate:a=0

- player离开的收尾动作
    - 暂停瞄准动画
    - tween将lind2d的width设置为0
    - `await tween.finished`再停止瞄准动画