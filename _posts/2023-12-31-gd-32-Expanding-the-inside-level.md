---
title: 32Expanding the inside level
categories: [godot,The ultimate introduction to Godot 4]
tags: [godot,The ultimate introduction to Godot 4]
date: 2023-12-31 13:30
---

丰富inside关卡的内容，主要在画面上增加贴花,增添恐怖氛围的物体

## 为什么玩家进入House没有补间动画
- 被他绕晕了 House 节点只在 outside 场景才有，而他在讲继承时将原来 House节点的`player_entered`和`player_exited`仍然放在父场景level中，也就是这两个信号根本就没地方链接

## 与光线交互越多性能损失就越大

inside关卡中的PointLight2D,toilet、crate、streamVent刻意设计成和光线没有交互

## Ording.ZIndex

典型的Z轴概念（默认为零，它还可以是负数）。比如：Node2的ZIndex>Node1，则Node2在Node1的上面。

- 作用：不改变节点的前后逻辑顺序，通过控制ZIndex就可以修改Node显示的顺序
- 实例:
    - Decals贴花节点，用来放些花花草草
    - Ground中的Sprite2D(平铺重复的地板)、TileMap（根据场景需要制作的瓦片地图）
    - 作者为了讲述zindex 拧巴，要将Decals节点放在Ground节点前，但他又希望花花草草显示在地面上
    - 这个时候只要设置 TileMap.zindex=-2、Sprite2D.zindex=-1、Decals.zindex=0,那么从上往下显示就是Decals、Sprite2D、TileMap
- 缺点：打乱原有节点的显示顺序。离奇是实例中inside关卡中的zindex修改居然影响到outside中的Gete和Player节点。哈哈哈，bug?

