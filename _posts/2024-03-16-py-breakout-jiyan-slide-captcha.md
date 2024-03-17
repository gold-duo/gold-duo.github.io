---
title: 用动画插值算法突破极验滑动验证码的"行为识别"
date: 2024-03-15 21:00
categories: [python,网络爬虫]
tags: [python,网络爬虫,滑动验证码,动画插值] 
---

滑动验证码的缺口坐标识别有一些库或者自己训练模型来处理，而在移动鼠标拼合缺口时极验收集了鼠标轨迹用作"行为识别"。
简单的随机移动鼠标轨迹过于粗糙，极验立马就告诉你“怪物吃了拼图”。

## 市面上突破"行为识别"的方法大体有
### 1.模拟人移动鼠标的动作

市面上模拟人移动鼠标的算法也挺多的，识别率也不一、算法奇特。

### 2.啃js。破解其中的算法或者达到直接提交缺口坐标

算法的破解是永无止境的。

### 3.用它的"行为识别"训练出自己的鼠标轨迹。

这是在github看到的一个项目，他爬下极验的js自己写了一个网页模拟出轨迹。
嗯。极验的算法用在极验上看似挺好的，但是也面临一些问题
- ①.接入算法到项目的难度

    一个网页自己手动操作再怎么套进项目中去？？貌似复杂了。

- ②.维护难度。

    跟2一样“算法的破解是永无止境的”，极验算法改了又得爬它的js。

## 动画插值突破"行为识别"

"行为识别"验证是否人操作，最多只能收集一定的鼠标轨迹和时间变量。而人移动鼠标不可能按照一个固定的轨迹、时间间隔，所以像极验这类验证码厂商不可能将鼠标轨迹规定的太过死板，规则过于死板人类移动的误判就增加了。也就是说它仅是一个模糊的判断规则。

看了一堆模拟人类移动鼠标的算法后，第一想到的是套android里各种动画插值算法。试了几个还真能派上用场。下面是几个用了可以通过识别的插值器

### 1.LinearInterpolator

线性插值器。 匀速运动的插值器，在安卓的activity转场、view移动时经常用到。

- 公式：y=x
- 算法图示

![]({{site.url}}/img/python/breakout-jiyan-slide-captcha01.png)

### 2.DecelerateInterpolator

减速插值器。 其运动从快到慢，在安卓的activity转场、view移动时经常用到。

- 公式：y=(1.0 - (1.0 - x) * (1.0 - x))
- 算法图示

![]({{site.url}}/img/python/breakout-jiyan-slide-captcha02.png)


### 3.AccelerateInterpolator

加速插值器。和DecelerateInterpolator相反其运动先慢后快，在安卓的activity转场、view移动时经常用到。

- 公式： y=x * x
- 算法图示

![]({{site.url}}/img/python/breakout-jiyan-slide-captcha03.png)

### 3.AccelerateDecelerateInterpolator

头尾慢中间快插值器。 其运动慢快慢，在安卓的activity转场、view移动时经常用到。

- 公式： y=(cos((x + 1) * PI) / 2.0) + 0.5
- 算法图示

![]({{site.url}}/img/python/breakout-jiyan-slide-captcha04.png)

## 实现动画插值算法的轨迹序列

移动验证码的“行为识别”主要还是判断移动的X坐标。至于y坐标和时间用随机数基本可以，有影响但不是决定因素。

看了一些实现模拟算法用到了一个大大的数组，果然写pyhon的人挺自由的。极验收集的行为轨迹30个，也就是我们只要模拟大于30个坐标基本就可以应付了。下面我用Generator(简单省空间)产生这些x坐标

```python
class MoveXGenerator:
    def __init__(self,dx:int):
        self.dx=dx
        self.size=44
        self.i=0
        self.interpolator=[
            lambda i:i,                                     #匀速,LinearInterpolator
            lambda i:(math.cos((i + 1) * math.pi)/2.0)+0.5, #头尾慢中间加速，AccelerateDecelerateInterpolator
            lambda i:1.0 - (1.0 - i) * (1.0 - i),           #减速，DecelerateInterpolator
            lambda i:pow(i,2),                              #加速，AccelerateInterpolator
        ][random.randint(0,3)]

    def __iter__(self):
        return self
    def __next__(self)->int:     
        if self.i<self.size:
            self.i+=1
            return round(self.interpolator(self.i/self.size)*self.dx)
        raise StopIteration    
```

## 成果
用网络爬虫实践里的练习网址[https://captcha1.scrape.center/](https://captcha1.scrape.center/),试了下效果还是挺好的虽然不能说是100%，但错了只要随机换个插值算法重头再来😄。

![]({{site.url}}/img/python/breakout-jiyan-slide-captcha05.gif)

## 总结

- 1.动画插值算法用于突破滑动验证码的"行为识别"是可行的

    有很多插值算法当然不一定都适合。

- 2.“行为识别”是一种模糊的匹配。

    仅靠收集有限的轨迹和现场数据不可能精确判断是不是人的神来之手。

- 3.由2就导致市面上各种模拟轨迹算法。

    有些算法复杂通过率并不好，还是换个思路轻松，还有很多动画插值算法可以再试。