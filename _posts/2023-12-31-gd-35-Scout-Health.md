---
title: 35Scout Health
categories: [godot,The ultimate introduction to Godot 4]
tags: [godot,The ultimate introduction to Godot 4]
date: 2023-12-31 13:30
---

实现hit函数逻辑，注意Scout不能被连续攻击（被激光攻击了换个grenade很快就挂了。再来个计时器控制下）

### Scout总是朝Player射击：
- 1.用`Globals.player_pos`记录Player的`global_position`(player._proccess更新)
- 2.(Globals.player_pos-global_position).normalized()获取到射击方向朝向player（这里跟获取玩家方向发射laser、grenade一样）