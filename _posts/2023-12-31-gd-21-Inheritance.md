---
title: 21Inheritance
categories: [godot,The ultimate introduction to Godot 4]
tags: [godot,The ultimate introduction to Godot 4]
date: 2023-12-31 13:30
---

## 继承scene

顶部菜单 scene - New Inheritance scene，在弹出的对话框选择要继承的scene

## scene脚步继承（默认子scene附属的 GDScript还是父scene的GDScript）：
- ①.给父scene添加`class_name`(比如`class_name LevelParent`)
- ②.子scene添加附属自己的场景, 且子scene的GdScript `extends`父scene的`class_name`(比如`extends LevelParent`)