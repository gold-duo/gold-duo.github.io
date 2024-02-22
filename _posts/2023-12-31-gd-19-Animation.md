---
title: 19Animation
categories: [godot,The ultimate introduction to Godot 4]
tags: [godot,The ultimate introduction to Godot 4]
date: 2023-12-31 13:30
---

## AnimatedSprite2D(典型的帧动画2D节点)
- Sprite Frames:需要新建一个 SpriteFrames，中下方面板将出现 `SpriteFrames`
    - 中下面板 `Autoplay on Load`选择是否自动播放
    - 每秒动画帧。默认5 FPS

## Sprite2D
- （为什么要讲这些东西，因为AnimationPlayer用Sprite2D的Frame做属性动画做例子很合适）
- Hframes(列帧)、Vframes(横帧):类似TileSet的思路，通过Hframes和Vframes将图片分成 `N*M` 个格子的棋盘。（比如：8*8，那么意味着Sprite2D节点将Texture分解成64个格子，总共64帧）
- Frame(帧数)。帧数 `X`是在`N*M` 个格子中的某个格子
- Frame Coords(帧坐标)：更具体的定位列和行（也就是格子中的 N 和 M）

## AnimationPlayer(动画播放器)：

任何形式的属性变化动画（move、rotation、动画帧、光能[light energy]、调用方法等等）

### `Add Track`创建步骤：
- ①.添加AnimationPlayer，将在中下面板出现`Animation`
- ②.点击`Animation`下拉框`New`新建一个Animation输入名字（这个名字将出现在中下方下拉框里，AnimationPlayer.play可以用到）
- ③.点击`Add Track`下拉菜单`Property Track`属性轨道，在弹出的对话框选择绑定的节点和节点的属性
- ④.设置好`Property Track`的轨道时长，调整时间轴到0，去到绑定节点的属性点击`钥匙`按钮（或点击`Insert Key`）就会将属性某一个时间点的值出现在轨道上

### `Add Track`的属性
- `Autoplay on Load`：中下面板右上角按钮
- `Animation Looping`:中下面板右上角按钮

### `Add Track`调用节点方法
- 比如点用属性动画播放完后节点消失：（时间轴设置在属性动画的结尾）在 `Add Track`中选择`call method`，选择`Fuctions`轨道右键`Insert Key`搜索选择调用节点方法`queue_free`

### `AnimationPlayer`代码控制播放某个动画`AnimationPlayer.play("animation_name")`

