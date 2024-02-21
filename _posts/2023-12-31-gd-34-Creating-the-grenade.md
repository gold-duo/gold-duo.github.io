---
title: 34Creating the grenade
categories: [godot,The ultimate introduction to Godot 4]
tags: [godot,The ultimate introduction to Godot 4]
date: 2023-12-31 13:30
---

完善 Grenade场景：grenade爆炸后在距离范围内打开crate、toliet和hit中Player、Scout

## 再次用到了Goups

将Player、Scout归类到Entity。（“Manage Groups”不好用、Godot的编辑器这么弱鸡[没有全局搜索功能]，到处用Groups实在不方便）

## 判断两个节点是否在距离范围

`node1.global_position.distance_to(node2.global_position)> x_distance`

## 两个Goups的节点相加(数组直接相加这个功能和python一样)
`get_tree().get_nodes_in_group("Container") + get_tree().get_nodes_in_group("Entity")`


