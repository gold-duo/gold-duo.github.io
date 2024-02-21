---
title: 五容器
date: 2021-8-31 23:24
categories: [flutter,Flutter Learning] 
tags: [flutter,Flutter Learning] 
---

flutter的widght本身不具备如android view的padding、margin、width、height属性，所以它需要此类容器来额外描述子组件

## 1.[Padding](https://book.flutterchina.club/chapter5/padding.html)
- 作用：给其子节点添加padding(相当于android的padding)
- 属性：`EdgeInsetsGeometry padding`使用`EdgeInsets`类提供的方法给`padding`属性复制

方法 | 作用
---|---
fromLTRB(left,top,right,botom) | 指定left、top、right、botom
all(double value) | 四个点均用value
only({left, top, right ,bottom })|仅设置某个点
symmetric({ vertical, horizontal })|vertical指定top和bottom，horizontal指定left和right

## 2.[尺寸限制容器](https://book.flutterchina.club/chapter5/constrainedbox_and_sizebox.html)
- 1).ConstrainedBox：用于对子组件的最大/最小的宽高添加约束
    - 属性`BoxConstraints constraints`。BoxConstraints包含minWidth/Height、maxWidth/Height四个属性。一些简化方法
    
    方法 | 作用
    ---|---
    tight(Size size) | 用size的width赋值给min/max width，height赋值给min/max height
    tightFor({double width,double height})|指定的大小创建约束宽高大小
    expand | 尽可能大
    loose(Size size)|最大宽高是size的宽高
- 2）SizedBox：作用固定子组件宽高
    - 属性：width、height
    - 等价ConstrainedBox(ConstrainedBox和SizedBox都是通过RenderConstrainedBox来渲染)

    ```
    ConstrainedBox(
      constraints: BoxConstraints.tightFor(width: w,height: h),
      ...
    )
    ```
    
- 3）UnconstrainedBox:不会对子组件产生任何限制，它允许其子组件按照其本身大小绘制。
- 4）AspectRatio 指定子组件的长宽比
- 5）LimitedBox 用于指定最大宽高
- 6）ractionallySizedBox 根据父容器宽高的百分比来设置子组件宽高


## 3.[DecoratedBox](https://book.flutterchina.club/chapter5/decoratedbox.html)
- 作用：在子组件绘制前(或后)绘制装饰（decoration），如背景、边框、渐变等。
- 属性：`DecorationPosition position`装饰作用域background还是foreground
- 属性：`Decoration decoration` 装饰属性。
    - Decoration是抽象类。函数createBoxPainter()创建一个画笔用于绘制装饰。
    - 常用`BoxDecoration`来实例并赋值给decoration

    ```
    BoxDecoration({
      Color color,                          //颜色
      DecorationImage image,                //图片
      BoxBorder border,                     //边框
      BorderRadiusGeometry borderRadius,    //圆角
      List<BoxShadow> boxShadow,            //阴影,可以指定多个
      Gradient gradient,                    //渐变
      BlendMode backgroundBlendMode,        //背景混合模式
      BoxShape shape = BoxShape.rectangle,  //形状
    })
    ```

## 4.[Transform](https://book.flutterchina.club/chapter5/transform.html)
- 作用：对其子组件应用矩阵变换特效
- 属性

    属性 | 作用
    ---|---
    AlignmentGeometry alignment | 子组件在Transform中的对齐方式
    Matrix4 transform | D转换矩阵.子组件的平移、旋转、缩放、倾斜变换
    Offset origin|指定子组件做平移、旋转、缩放时的中心点
    transformHitTests|点击区域是否也做相应的变换,为true时执行相应的变换,为false不执行
- 静态辅助方法

    属性 | 作用
    ---|---
    translate(offset, child)|接收一个offset参数，在绘制时沿x、y轴对子组件平移指定的距离
    rotate(angle, child)|旋转。类似于RotatedBox（变换是在layout阶段，会影响在子组件的位置和大小）
    scale(scale,child)|缩放
- 注意点：Transform的变换是在绘制阶段，并**不是在布局(layout)阶段**，所以，其**占用空间的大小**和在屏幕上的位置都是**固定不变**的，因为这些是在布局阶段就确定的。
    
## 5.[Container](https://book.flutterchina.club/chapter5/container.html)
- 作用：它是DecoratedBox、ConstrainedBox、Transform、Padding、Align等组件组合的一个多功能容器
- 属性：

```
Container({
  this.alignment,
  this.padding,                 //容器内补白，属于decoration的装饰范围
  Color color,                  //背景色
  Decoration decoration,        //背景装饰（color和decoration是互斥的，如果同时设置它们则会报错！）
  Decoration foregroundDecoration, //前景装饰
  double width,                 //容器的宽度
  double height,                //容器的高度
  BoxConstraints constraints,   //容器大小的限制条件(若和width、height同时存在，优先width、height)
  this.margin,                  //容器外补白，不属于decoration的装饰范围
  this.transform,               //变换
  this.child,
})
```

## 6.[Scaffold](https://book.flutterchina.club/chapter5/material_scaffold.html)

