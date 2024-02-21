---
title: 三基础组件-4常用widget
date: 2021-8-31 23:24
categories: [flutter,Flutter Learning] 
tags: [flutter,Flutter Learning] 
---

介绍[按钮](https://book.flutterchina.club/chapter3/buttons.html)

## 公共属性
```dart
@required this.onPressed, //按钮点击回调
this.textColor, //按钮文字颜色
this.disabledTextColor, //按钮禁用时的文字颜色
this.color, //按钮背景颜色
this.disabledColor,//按钮禁用时的背景颜色
this.highlightColor, //按钮按下时的背景颜色
this.splashColor, //点击时，水波动画中水波的颜色
this.colorBrightness,//按钮主题，默认是浅色主题 
this.padding, //按钮的填充
this.shape, //外形。如RoundedRectangleBorder(borderRadius: BorderRadius.circular(20.0))
@required this.child, //按钮的内容
this.elevation = 2.0, //正常状态下的阴影
this.highlightElevation = 8.0,//按下时的阴影
this.disabledElevation = 0.0,// 禁用时的阴影
```

## 常用按钮

按钮|描述|其他
---|---|---
RaisedButton|默认带有阴影和灰色背景。按下后阴影会变大|
FlatButton|默认背景透明并不带阴影。按下后会有背景色|
OutlineButton|默认有一个边框，不带阴影且背景透明。按下后，边框颜色会变亮、同时出现背景和阴影(较弱)|
IconButton|点击的Icon，不包括文字，默认没有背景，点击后会出现背景|

## [图片及ICON](https://book.flutterchina.club/chapter3/img_and_icon.html)
## 1.ImageProvider
  - A.抽象类，主要定义了图片数据获取的接口load()
  - B.AssetImage实现了从Asset中加载图片;NetworkImage实现了从网络加载图片;

## 2.Image

```dart
Image(
this.image,//ImageProvider
this.width, //图片的宽
this.height, //图片高度
this.color, //图片的混合色值
this.colorBlendMode, //混合模式
this.fit,//缩放模式.参见BoxFit枚举
this.alignment = Alignment.center, //对齐方式
this.repeat = ImageRepeat.noRepeat, //重复方式。如：ImageRepeat.repeatY 
)
```
**快捷的构造函数**
  
  - 1.`Image.asset("assert/images/avatar.png",{width: w,height:h})`
  - 2.`Image.network("http://wwww.abc.com/images/avatar.png",{width: w,height:h})`
  - 3.`Image.file`
  - 4.`Image.memory`

**BoxFit**
枚举|作用
---|---
fill|拉伸填满显示空间，图片本身长宽比会发生变化
cover|按图片长宽比放大后居中填满显示空间，图片不会变形，超出显示空间部分会被剪裁
contain|`默认适应规则`保证图片长宽比不变的情况下缩放以适应显示空间，图片不会变形。
fitWidth|宽度会缩放到显示空间的宽度，高度会按比例缩放，然后居中显示，图片不会变形，超出显示空间部分会被剪裁。
fitHeight|跟fitWidth相似
none|图片没有适应策略，会在显示空间内显示图片，如果图片比显示空间大，则显示空间只会显示图片中间部分。

**color和 colorBlendMode**：在图片绘制时可以对每一个像素进行颜色混合处理，color指定混合色，而colorBlendMode指定混合模式。

**Image缓存：**默认最大缓存数量是1000，最大缓存空间为100M。

## 3.Icon


## 6.[Switch和CheckBox](https://book.flutterchina.club/chapter3/radio_and_checkbox.html)

然它们都是继承自StatefulWidget，但它们本身**不保存当前选中状态**，选中状态由父组件来管理。当被**点击时会触发它们的onChanged回调**，可在此处理选中状态改变逻辑。

