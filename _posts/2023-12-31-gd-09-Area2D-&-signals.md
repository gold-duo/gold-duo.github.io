---
title: 09Area2D & signals
categories: [godot,The ultimate introduction to Godot 4]
tags: [godot,The ultimate introduction to Godot 4]
date: 2023-12-31 13:30
---

介绍Area2D & signals

## sinals:就是节点之间互通信。其实是就是一个回调指针、或者一些语言的事件，godot搞个多余的signal概念。比如：
- 1.玩家进入了某个区域要触发通知该消息给其他节点

## Area2D用于检查物体进入区域。
- conect sinal:右侧面板 **Inspector** tab 左边的 **node**, 双击相应的sinal名称（ 或右键菜单相点击**conect**）,在弹出的对话框中选择处理 signal 的节点
- 用到的两个signal：`body_entered(body)、body_exited(body)`

