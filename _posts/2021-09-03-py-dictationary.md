---
title: dictationary
date: 2021-09-03 22:12
categories: [python,python编程从入门到实践] 
tags: [python,python编程从入门到实践] 
---

## p83_dictationary

```python
dic = {'points': 5}  # 创建一个字典.(一对空大括号则创建空字典)
print(dic)

dic["color"] = "green"  # 如果没有对应的key值则添加，有则修改
print(dic)

# 1.遍历
for k, v in dic.items():
    print(k+"="+str(v))

# 2.遍历所有key
for k in dic.keys():
    print(k)

# 3.遍历所有value
for v in dic.values():
    print(v)

# 4.判断key是否在字典里
f = 'points' in dic
print(f)

del dic['points']  # 删除
print(dic)
```
