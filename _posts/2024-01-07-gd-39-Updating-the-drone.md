---
title: 39Updating the drone
categories: [godot,The ultimate introduction to Godot 4]
tags: [godot,The ultimate introduction to Godot 4]
date: 2024-01-07 19:30
---


这节完善敌人系统里的drone

## drone特点
- 1.Player进入drone区域，drone朝玩家[加速移动](#section39_01)
- 2.drone与[任何物体碰撞则爆炸](#section39_02)
- 3.drone被击中产生爆炸效果([AnimationPlayer]()
- 4.drone爆炸附近范围的物体(Container、Entity组)也[被殃及(触发爆炸)](#section39_03)

## 一些细节
- <a id="section39_01">向player加速移动</a>

```python
#在player进入边界，用补间动画，在短时间内将 speed 属性从0调整到600
var tween = create_tween()
tween.tween_property(self, "speed",max_speed, 6)
```
- <a id="section39_02">判断是否与其他物体碰撞：`move_and_collide( velocity*delta )`</a>返回一个 KinematicCollision2D，其中包含有关停止时或沿运动触摸另一个实体时碰撞的信息。

- drone爆炸后，drone的爆炸效果仍然跟随玩家问题

```python
#他的方法是通过速度乘数来实现：默认speed_multiplier=1，击中speed_multiplier=0那么就不会移动
velocity = (Globals.player_pos - position).normalized() * speed * speed_multiplier

#实际可通过explosion_active判断，那么下面的代码则不会执行，drone也不会移动
func _process(delta):
    if active and  not explosion_active:
        #.....
```

- <a id="section39_03">爆炸殃及池鱼</a>。这个实现基本和`grenade`差不多

```python
if explosion_active:
    var targets = get_tree().get_nodes_in_group("Container")\
                + get_tree().get_nodes_in_group("Entity")
    for it in targets:
        if "hit" in it\
            and it.global_position.distance_to(global_position)<400:
            it.hit()
```