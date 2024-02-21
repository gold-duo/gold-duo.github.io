---
title: 03Coding for interactivity
categories: [godot,The ultimate introduction to Godot 4]
tags: [godot,The ultimate introduction to Godot 4]
date: 2023-12-31 13:30
---

## GDScrip 和 Python的区别
- Datatypes(数据类型)：GDScript没有元组、列表(数组代替)
- 变量：GDScript有两种类型的变量
    - 1.var 普通变量。

    ```python
    # 1.动态类型。
    var speed=200 
    #GDScript的动态据类型可以动态改变。比如 speed="speed" 将字符串赋值给speed

    # 2.固定类型
    var str:String='test'   #指明类型。后面修改只能赋值字符串
    var array:Array[int]=1234
    ```
    - 2.const 常量`const max_sped=500`
- Functions（函数）

```python
# 遵循python的4格缩进语法
func test_fn(p1:int,p2:String)->bool:
    return true
```
- 控制流语句：基本和python一样

```python
#if elif else
if a:
    #...
elif b:
    #...
else 
    #...
#三元操作符
var value= a if test else b #相当于c的 value=test? a:b
```
- Classes（类）
    - **Node就是带默认方法和属性的类**
        - 比如：添加了一个Node2D节点，那么这个对象将具有 position、ratation、scale等属性和方法
    - 内置函数都以下划线开头。
    
    ```python
    # 比如两个重要的内置函数：
    # 1._ready:节点准备好时调用该函数
    # 2._process:游戏运行的每一帧都会调用
    ```
    - 访问其他节点
        - 1.get_node("node path")
        - 2.$node path

