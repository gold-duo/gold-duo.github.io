---
title: 24User Interface
categories: [godot,The ultimate introduction to Godot 4]
tags: [godot,The ultimate introduction to Godot 4]
date: 2023-12-31 13:30
---

## Godot四种节点：2D、3D、Control、Other
## Control Nodes:Labels、images、progress bars、buttons、areas with colors、text input、graphs、menus、videos
## CanvasLayer:本质是游戏上的另一层，该层忽略Camera。(CanvasLayer总是在frams上 )
## Anchor preset(控件锚点设置。中上图标有弹出菜单)。当这个值设置成`Custom`(通过Inspector设置)时则可以通过Anchor Points来控制控件的四个坐标(有意思的是这个数值是游戏可见范围的宽高的百分比)
## 还有一些其他控件

控件名|作用|其他
---|---|---
Control||
HBoxContainer|子控件横向排列容器|
VBoxContainer|子控件竖向排列容器|
GridContainer|网格容器|Columns控制子控件排几列

