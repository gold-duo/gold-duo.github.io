---
title: 07More on nodes
categories: [godot,The ultimate introduction to Godot 4]
tags: [godot,The ultimate introduction to Godot 4]
date: 2023-12-31 13:30
---

##  描述：访问节点的第三种方式（前两种get_node()、$node、）
##  节点唯一访问id
    - 使用场景:节点多层嵌套，访问子孙节点(`$"n1/n2/n3"`)代码路径太长不易阅读维护
    - 使用方式：右键单击节点 -》"% Access as unique name"。现在代码中直接用 `%节点文件名`(也可以直接拖动该节点到代码)引用
##  访问父节点（`$".."`）、祖节点（`$"../.."`）

