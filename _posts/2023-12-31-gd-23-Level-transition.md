---
title: 23Level transition
categories: [godot,The ultimate introduction to Godot 4]
tags: [godot,The ultimate introduction to Godot 4]
date: 2023-12-31 13:30
---

## 跳转到指定scene（两种方案）:
- 1.change_scene_to_file:`get_tree().change_scene_to_file("res://path/name.tscn")`
- 2.change_scene_to_packed:

```python
var x_scene:PackedScene=preload("res://path/name.tscn")
get_tree().change_scene_to_packed(x_scene)
```