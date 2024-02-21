---
title: 36Player Health
categories: [godot,The ultimate introduction to Godot 4]
tags: [godot,The ultimate introduction to Godot 4]
date: 2023-12-31 13:30
---

完善玩家的 hit 函数

## 怎么避免玩家连续被 hit
- 在Global.health的set中增加 bool变量和timer控制

```python
if player_vulnerable:
    player_vulnerable=false
    _play_invulnerable()
    health=value
    stats_change.emit()

func _play_invulnerable():
    await get_tree().create_timer(0.5).timeout #创建timer并等待0.5秒
    player_vulnerable=true
```
    
- 上面的处理方式带来新坑：没法处理加血行为(health增加).他的处理也很暴力直接判断设置的value是否大于当前值，是就设置新value

