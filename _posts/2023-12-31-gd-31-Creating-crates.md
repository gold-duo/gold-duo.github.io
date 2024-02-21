---
title: 31Creating crates
categories: [godot,The ultimate introduction to Godot 4]
tags: [godot,The ultimate introduction to Godot 4]
date: 2023-12-31 13:30
---

在 [29节](29.Creating%20items.md) 的基础上，射击某个物品（比如宝箱[crate]、厕所[toilet]）时创建 items 撒点钞票或者补血

## [Groups](https://docs.godotengine.org/zh-cn/4.x/tutorials/scripting/groups.html)：Godot 与其他软件中的标签类似。可以将节点加入任意数量的你想要的分组之中(用来组织大型场景、对代码解耦)。
### Groups的作用：你可以使用 SceneTree 来
- 获取某个分组中的节点列表。
- 在分组中的所有节点上调用方法。
- 向分组中的所有节点发送通知。
### Groups的创建
- 1.代码创建和移除

```python
    Node.add_to_group(StringName group, bool persistent=false) #创建
    Node.remove_from_group(StringName group )#移除
```
- 2.编辑器创建：选中节点-> 右侧面板 -> `Inspector`右边的`Node` Tab-> 点击`Groups`-> 输入Group 名称-> 点击`Add`即添加到对应的 group
- 3.使用编辑器> 右侧面板 -> `Inspector`右边的`Node` Tab-> 点击`Groups`-> `Manage Groups`创建
### Groups的一些功能

函数|描述|实例
---|---|---
get_nodes_in_group(group_name)|获取分组内所有成员|get_tree().get_nodes_in_group("guards")
call_group(group_name, node_func)|调用分组中的所有成员 node_func 函数。|get_tree().call_group("guards", "enter_alert_mode")

### `Can't change this state while flushing queries. Use call_deferred() or set_deferred() to change monitoring state instead.`
 >  直接调用方法去  修改secne_tree (添加、删除子节点)，处理图形碰撞等都会造成线程不安全。用 call_deferred(fn_name, ...)替代如add_child

