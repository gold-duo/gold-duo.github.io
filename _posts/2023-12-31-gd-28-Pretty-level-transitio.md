---
title: 28Pretty level transitio
categories: [godot,The ultimate introduction to Godot 4]
tags: [godot,The ultimate introduction to Godot 4]
date: 2023-12-31 13:30
---

## 平滑关卡过渡.用[Autoload](26.Making%20the%20UI%20interactive.md#section26.1)结合[AnimationPlayer](./19.Animation.md#section19.3)制造一个颜色渐变的scene，转关卡时颜色先由透明到黑,加载目标scene再由黑到透明。创建步骤
- 新建一个scene(TransitionLayer),并添加到`Autoload`
- 添加两个节点:`ColorRect`(color=000000[全黑])、`AnimationPlayer`
- 添加轨道(fade_to_black):将`ColorRect.modulate`从透明调到全黑
- 代码控制转scene

```python
func change_scene(target:String)->void:
    $AnimationPlayer.play("fade_to_black")
    await $AnimationPlayer.animation_finished #这句很重要！！一直等到动画播放完
    
    get_tree().change_scene_to_file(target)

    $AnimationPlayer.play_backwards("fade_to_black") #play_backwards将动画翻着播放（相当于全黑到透明）
```

