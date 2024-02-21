---
title: function
date: 2021-09-03 22:12
categories: [python,python编程从入门到实践] 
tags: [python,python编程从入门到实践] 
---

## p130_function

```python
# 定义任意数量的参数(实质是将参数封装到一个元组中)
def make_pizza(*toppings):   # *toppings是一个元组
    print(toppings)


make_pizza('a')
make_pizza('a', 'b', 'c', 'd')


# 定义任意数量的关键字参数
def buildProfile(first, last, **user_info):  # **user_info是一个字典
    profile = {}
    profile['first_name'] = first
    profile['last_name'] = last
    for k, v in user_info.items():
        profile[k] = v
    return profile


profile = buildProfile('fan', 'liao', location='gz', sex='man')
```

