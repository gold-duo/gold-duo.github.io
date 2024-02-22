---
title: 20Connecting the laser to the level
categories: [godot,The ultimate introduction to Godot 4]
tags: [godot,The ultimate introduction to Godot 4]
date: 2023-12-31 13:30
---

给玩家增加手表右键/空格键单击时增加发射激光动作，如果激光击中的[物体带有`hit`函数](#section20_01)那么调用它表明被攻击了。

## 一些细节
- 判断对象是否有某个方法：
    - 1.has_method:`obj.has_method(name)`
    - 2.`in`检查对象是否具有属性、方法 `name in obj`
- 激光自动销毁
    - 1.激光和其他对象碰撞时消费（queue_free）
    - 2.用计时器`timeout`来销毁

