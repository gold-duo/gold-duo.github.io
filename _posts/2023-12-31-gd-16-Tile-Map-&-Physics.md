---
title: 16Tile Map & Physics
categories: [godot,The ultimate introduction to Godot 4]
tags: [godot,The ultimate introduction to Godot 4]
date: 2023-12-31 13:30
---

介绍Tile Map & Physics

> laser是一个区域(Area2D)不是一个物体,物体可能是玩家(CharacterBody2D)或者手榴弹（RigidBody2D）

## 在Godot中将可碰撞对象放置在不同的layer上，然后告诉它们只与特定的layer上的对象交互。该系统的工作方式是：
- `Layer`决定对象本身位于那一层上（在Inspector->Collision->Layer里设置）
- `Mask`决定它可以和那个Layer交互/碰撞（在Inspector->Collision->Mask里设置）

```
例子
1.Layers:(Porject Setting->General->Layer Names->2D Physics给对应的Layer增加描述，Inspector->Collision->Layer和Mask的tip将显示Layer描述)
    ①.Player
    ②.Enemies(敌人)
    ③.Objects（物体层）
    ④.Projectiles(投射物激光、手榴弹)
    ⑤.Items & Zones(物品和区域)

2.Player(Player scene或plyer的character body2D)将出现在玩家层（layer），但它的Mask将与所有其他层进行物理交互
3.Enmies本身将在敌人层(layer)上，然而它的Mask只能和Player、Objects、Projectiles交互，它无法和 Item & Zones产生交互
4.Gate Area2D出现在 Items & Zones层上，它只能和玩家交互；Gate StaticBody2D出现在Objects层，它可以和Player、Enemies、Projectiles互动（Objects是不会移动的所以不会碰撞）
5.手榴弹(grenade)它出现在Projectiles层而不是Player层;它可和除了Items & Zones外的其他层碰撞（即Mask交互）,如果不设置 Projectiles Mask它们将不会碰撞
6.House它出现在 Items & Zones层，它只和玩家交互
```

## Tile Map 的物理层
- 1.右侧Inspector面板,单击Tile Set下拉 Physics Layers点击添加物理层
- 2.House所在的层在 `Objects` 层,它可以和Player、Enemies、Projectiles碰撞（Mark）
- 3.在中下方面板点击`Tile Set`,点击`Paint`-> Paint Properties选择 物理层
- 4.接下来绘制右侧 Tiles的碰撞区域

