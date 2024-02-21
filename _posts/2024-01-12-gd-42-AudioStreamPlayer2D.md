---
title: 42AudioStreamPlayer2D
categories: [godot,The ultimate introduction to Godot 4]
tags: [godot,The ultimate introduction to Godot 4]
date: 2024-01-12 09:00
---


## 基本属性/方法
- play:手动播放
- Stream:音频流
- Volume dB:音量
- Pitch Sale：音调
- Playing：播放
- Autoplay:自动播放
- Loop:循环播放。左侧面板`Scene`旁边的`Import`-> Loop点选-> Reimport(右侧AudioStreamPlayer2d-> Stream展开->loop将同步选中状态)

## 代码里动态创建
```python
var player=AduioStreamPlayer2D.new()
player.stream=load('res://soud_path')
add_child(player)
#...
player.play()
```
## 等待声音播发完`await $AudioStreamPalyer.finished`

## AnimationPlayer结合使用:要添加`Audio Playback track` -> Insert key -> 设置 Stream

## 背景音乐(且不受场景切换影响,切换场景也继续播放)
- 添加一个scene
- 将这个场景增加到`AutoLoad`
- 设置 Autoplay=true

## 给鼠标添加图标：Project -> Project Setting -> General ->Display->Mouse Cursor -> Custom Image
- Custom Image Hotspot：设置鼠标图热点(实际触发的点击点)。如果一张图时正方体，默认x&y=0,这时这个点在正方体的左上角，故要设置中心点若图片是32*32,那么x&y=16

