---
title: 33Scout enemy
categories: [godot,The ultimate introduction to Godot 4]
tags: [godot,The ultimate introduction to Godot 4]
date: 2023-12-31 13:30
---

创建侦察兵(Scout),侦察兵发射激光的逻辑和玩家一样，不过是多了左右枪交替发射功能

## Scout
- 特征：静态的但可以像玩家一样射击,拥有两把枪来回射击
- 发现玩家进入边界：使用一个Area2D节点(覆盖sout的很大半径范围)，并将其layer设置在 Items&Zone只对玩家可见
- 在一个scene中可能有多个Scout,每个signal都connect事件非常繁琐容易出错的事情，为了connect Scout发射的laser signal用到Groups

```python
#在level.gd
func _onready():
    for it in get_tree().get_nodes_in_group("Scouts"):#处理所有Scout发射激光signal
        it.laser.connect(_on_scout_laser)
```

## GDScript跟Python一样会将bool类型true、false自动转换成1、0（实例中用这个功能来交替左右开枪）。有点狗屎,不是个好逻辑

