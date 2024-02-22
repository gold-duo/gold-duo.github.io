---
title: 三基础组件-3文本及样
date: 2021-8-31
categories: [flutter,Flutter Learning] 
tags: [flutter,Flutter Learning] 
---


介绍[文本及样式](https://book.flutterchina.club/chapter3/text.html)

## 1.Text


属性|作用|其他
---|---|---
textAlign|文本对齐方式|如TextAlign.left
textScaleFactor|对于当前字体大小的缩放因子|相对于style属性的fontSize。该属性的默认值可以通过MediaQueryData.textScaleFactor获得，如果没有MediaQuery，那么会默认值将为1.0。
style|用于指定文本显示的样式如颜色、字体、粗细、背景等|见下表TextStyle


### **TextStyle**
属性|作用|其他
---|---|---
color|字体颜色|
fontFamily|字体|
fontSize|字体大小|
background|背景色|
decoration|装饰器|如extDecoration.underline
decorationStyle|装饰器style|如TextDecorationStyle.dashed

## 2.TextSpan
```dart
const TextSpan({
  TextStyle style, //见上TextStyle
  Sting text,
  List<TextSpan> children,
  GestureRecognizer recognizer,//用于对该文本片段上用于手势进行识别处理
});
```

### example
```dart
Text.rich(TextSpan(//Text其实就是RichText的一个包装，而RichText是可以显示多种样式(富文本)的widget。
    children: [
     TextSpan(
       text: "Home: "
     ),
     TextSpan(
       text: "https://flutterchina.club",
       style: TextStyle(
         color: Colors.blue
       ),  
       recognizer: _tapRecognizer
     ),
    ]
))
```

## 3.DefaultTextStyle(在Widget树中，文本的样式默认是可以被继承的)
```dart
DefaultTextStyle(
  //1.设置文本默认样式  
  style: TextStyle(
    color:Colors.red,
    fontSize: 20.0,
  ),
  textAlign: TextAlign.start,
  child: Column(
    crossAxisAlignment: CrossAxisAlignment.start,
    children: <Widget>[
      Text("hello world"), //继承红色、20号字体
      Text("I am Jack"),   //继承红色、20号字体
      Text("I am Jack",    //不继承
        style: TextStyle(
          inherit: false, //2.不继承默认样式
          color: Colors.grey
        ),
      ),
    ],
  ),
);
```

## 4.字体

使用字体分两步
### 1.在asset中声明。先在pubspec.yaml中声明，然后将字体文件复制到在pubspec.yaml中指定的位置。

```dart
flutter:
  fonts:
    - family: Raleway
      fonts:
        - asset: assets/fonts/Raleway-Regular.ttf
        - asset: assets/fonts/Raleway-Medium.ttf
          weight: 500
```
### 2.通过TextStyle属性使用字体

```dart
// 声明文本样式
const textStyle = const TextStyle(
  fontFamily: 'Raleway',
);

// 使用文本样式
var buttonText = const Text(
  "Use the font for this text",
  style: textStyle,
);
```
**使用Package中的字体**

```dart
const textStyle = const TextStyle(
  fontFamily: 'Raleway',
    package: 'my_package', //指定包名
  );
  ```