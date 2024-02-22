---
title: 10create custom signals
categories: [godot,The ultimate introduction to Godot 4]
tags: [godot,The ultimate introduction to Godot 4]
date: 2023-12-31 13:30
---

godot scene内的节点的signal不能跨越scene，这个时候自定义siganl就派上用场

## 自定义 siganl步骤：
- 定义：`signal my_signal([p1,p2...])`
- 发射：`signal my_signal.emit([p1,p2...])`
- 使用：现在跟使用普通的signal一样

## Timer：跟其他的语言的计时器一样的概念。用到的属性和siganl
- wait time:几秒后触发
- one shot:是否只触发一次
- Autostart：自动计时
- start方法：启动计时
- timeout 信号：计时器出发时信号

