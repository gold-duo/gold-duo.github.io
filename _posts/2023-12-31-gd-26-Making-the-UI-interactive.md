---
title: 26Making the UI interactive
categories: [godot,The ultimate introduction to Godot 4]
tags: [godot,The ultimate introduction to Godot 4]
date: 2023-12-31 13:30
---

## Autoload

GDScript（甚至scene）自动加载到各个场景中(常用来保存全局数据，比如生命值、子弹数)

- 添加Autoload:Project -> Project Setting -> Autoload：选择文件、设置名称(比如Globals)点击Add
- 访问：通过AutoLoad设置的名称访问。比如:`Globals.laster_amount`

## 节点引用

在GDScript的头文件直接引用节点（比如`var laser_label: Label = $LaserCounter/VBoxContainer/Label`）Godot将报错，这是由于节点这个时候还没被添加到树中。解决的方法有两种

- 1.在`_onReady`中赋值
- 2.在变量前添加`@onready`注解。（比如`@onready var laser_label: Label = $LaserCounter/VBoxContainer/Label`）

