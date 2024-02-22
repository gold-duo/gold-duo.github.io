---
title: 11Creating custom scenes
categories: [godot,The ultimate introduction to Godot 4]
tags: [godot,The ultimate introduction to Godot 4]
date: 2023-12-31 13:30
---

标题是Creating custom scenes,更应该是动态加载scene，因为改节讲的是怎么把laser动态加载到level场景中

## 动态加载scene步骤
- 1.加载：`var x_scene:PackedScene = preload('res://path/name.tscn')`
- 2.实例化：`scene=x_scene.instantiate()`
- 3.添加到节点树：`add_child(scene)`

## Marker2D 节点：
用于编辑的通用 2D 位置提示。类似于普通的 Node2D，但它始终在 2D 编辑器中显示十字。该节点处于选中状态时，可以使用 2D 编辑器中的小工具来设置十字的视觉大小

- 实例中用改节点来定位 激光器(laser)和手雷(grenade)

## $Node.get_children()：
获取子节点，返回一个数组.(`LaserStartPositions.get_children()`)

## randi()随机数

## global_position vs position:

这个显示区域的position和相对于在父节点内的position

