---
title: 12RigidBody2D
categories: [godot,The ultimate introduction to Godot 4]
tags: [godot,The ultimate introduction to Godot 4]
date: 2023-12-31 13:30
---

通过grenade介绍RigidBody2D的使用，通过Player扔的方向投出grenade

## RigidBody2D的一些属性
- Gravity Scale(重力缩放)：0静止，默认为1物体向下移动
- Linear Velocity(线性加速度)
    - Damp（阻尼）
    - ：(包含x、y轴)：比如设置x=100，y=0(在未设置PhsicsMaterial Override情况下)，物体将向右移动
- Angular Velocity(角加速度)：包含x、y轴
- PhsicsMaterial Override(物理材质):默认情况下RigidBod2D不会发生碰撞，除非添加PhsicsMaterial（Inspector面板添加）.属性
    - Bounce（弹性）：0则没有
    - Rough：
    - Friction（摩擦力）
- Constan Forces(持续聚焦)

## 其他
- 一个向量的默认方向总是水平向右
- 玩家跟随鼠标方向`look_at(get_global_mouse_position())`

- **获取鼠标当前点击相对玩家的方向**
```python
var player_direction = (get_global_mouse_position() - position).normalized()
# get_global_mouse_position、position分别是两个向量相减
# normalized返回伸缩的向量长度单元，相当于v/v.lenth()
```
- rotation_degrees是角度，rotation是弧度
```python
#下面这两句是等价的
laser.rotation_degrees = rad_to_deg(direction.angle()) + 90
laser.rotation=direction.angle()+1.5708 # deg_to_rad(90)=1.5707963267949
```
- rad_to_deg:将以弧度表示的角度转换为度数。
```python
rad_to_deg(0.523599) # Returns 30
rad_to_deg(PI)       # Returns 180
rad_to_deg(PI * 2)   # Returns 360
```

