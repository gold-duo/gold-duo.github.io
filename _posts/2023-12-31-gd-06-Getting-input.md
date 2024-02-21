---
title: 06Getting input
categories: [godot,The ultimate introduction to Godot 4]
tags: [godot,The ultimate introduction to Godot 4]
date: 2023-12-31 13:30
---

## 描述：实现如何在Godot中捕获输入(物理键、鼠标、手柄)
## 获取输入2个步骤：
- 1.创建输入映射(Input Map)
    - 通过菜单：Project -> Project Setting -> Input Map 添加要映射的输入(物理键、鼠标、手柄)：`比如物理键 left、a为左边移动，命名为 "left"`
- 2.访问输入映射.代码中则可以用 `Input.is_action_pressed(“left”)` 判断 left、a按下
## `Input.get_vector`：获取一个移动向量.比如:`direction=Input.get_vector("left","right","up","down")`
    ## Sprite朝输入方向移动:`position +=Input.get_vector("left","right","up","down")*delta*STEEP`，检查四个按键事件，并返回一个方向向量的累加值。

