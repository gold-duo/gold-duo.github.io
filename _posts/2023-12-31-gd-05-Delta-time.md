---
title: 05Delta time
categories: [godot,The ultimate introduction to Godot 4]
tags: [godot,The ultimate introduction to Godot 4]
date: 2023-12-31 13:30
---

游戏在不同性能配置的计算机上运行，得到的帧率是不一致（pc越快帧率越大，那么游戏中角色(图像/物体)的移动越快）。这时delta time派上用场

## delta time 衡量创建当前桢花费的时间
- 比如：当前桢率是每秒60帧，delta time=1/60=0.0167,也即帧速率越高delta time也小
- deta time用来保证游戏中任何对象的恒定速度（运动量和delta相乘即可），而不受帧率影响
![05_delta_time01](/img/godot/05_delta_time01.png)

## 函数中不使用的参数，godot编辑器显示黄色警告，参数前加下划线前缀表示不使用参数

