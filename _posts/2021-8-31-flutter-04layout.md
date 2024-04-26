---
title: 四布局
date: 2021-8-31 23:24
categories: [flutter,Flutter Learning] 
tags: [flutter,Flutter Learning] 
---

## 一.[基础类和继承结构](https://book.flutterchina.club/chapter4/intro.html)
### 1.基础类

Widget|对应的Element| 用途 
---|---|---
LeafRenderObjectWidget        | LeafRenderObjectElement        | Widget树的叶子节点，用于没有子节点的widget，通常基础组件都属于这一类，如Image。
SingleChildRenderObjectWidget | SingleChildRenderObjectElement | 包含一个子Widget，如：ConstrainedBox、DecoratedBox等
MultiChildRenderObjectWidget  | MultiChildRenderObjectElement  | 包含多个子Widget，一般都有一个children参数，接受一个Widget数组。如Row、Column、Stack等

### 2.继承结构

```mermaid
graph TD
    Widget-->A[RenderObjectWidget]
    A-->|LeafRenderObjectElement|C[LeafRenderObjectWidget]
    A-->|SingleChildRenderObjectElement|D[SingleChildRenderObjectWidget]
    A-->|MultiChildRenderObjectElement|E[MultiChildRenderObjectWidget]
    C-->C1[Image]
    D-->D1[ConstrainedBox/DecoratedBox]
    E-->E1[Row/Column/Stack]
```
![](/assets/flutter/0401layout.png)

## 二.[Row、Column](https://book.flutterchina.club/chapter4/row_and_column.html)(线性布局)

公共属性:

属性 | 作用 | 其他
---|---|---
textDirection| 水平/垂直方向子组件的布局顺序(rtl/ltr)|Locale环境的文本方向(如中文、英语都是从左往右，而阿拉伯语是从右往左)
verticalDirection|垂直/水平方向子组件的布局顺序|默认VerticalDirection.down：表示从上到下(从左到右)<br/>up：与down相反
mainAxisSize|Row/Column在主轴(水平/垂直)方向占用的空间| 默认**MainAxisSize.max**：Row/Column的**宽/高**度始终等于**水平/垂直**方向的**最大宽/高度**<br/>min：尽可能少的占用**水平/垂直**空间（Row的实际宽度=所有子组件占用的的水平空间）
mainAxisAlignment|子组件在Row/Column所占用的**水平**/**垂直**空间内对齐方式|**textDirection是mainAxisAlignment的参考系**<br/>start：沿textDirection的初始方向对齐<br/>end：则与start相反<br/>center:居中对齐<br/>**当mainAxisSize=min时对于Row此属性无意义**
crossAxisAlignment|子组件在Row/Column所占用的**垂直**/**水平**空间内对齐方式|**verticalDirection是crossAxisAlignment的参考系**<br/>start：verticalDirection=down顶部/左边对齐,verticalDirection=up底部/右边对齐<br/>end：和start相反
 

## 三.[Flex](https://book.flutterchina.club/chapter4/flex.html)

- Row(direction: Axis.horizontal)、Column(direction: Axis.vertical)是Flex子类.顾除了direction Flex具有Row、Column的所有属性
- 只有**和Expanded（Space）组件配合**才能实现Flex布局
- Expanded
    - 1.flex属性：子组件所占用的空间（0或null，则child不会被扩伸占用的空间）

## 四.[Wrap、Flow](https://book.flutterchina.club/chapter4/wrap_and_flow.html)(流式布局)

- 为什么需要Wrap、Flow？
> Row、Column的子widget超出屏幕范围flutter默认不能自适应换行，而是报溢出错误。

### 1.Wrap

属性 | 作用 | 其他
---|---|---
spacing|主轴方向子widget的间距|
runSpacing|纵轴方向的间距|
runAlignment|纵轴方向的对齐方式|
direction、crossAxisAlignment、textDirection、verticalDirection|参照Row、Column属性|

### 2.FLow 

Flow提供了dalegate（FlowDelegate）的方式对子Widget的位置进行定位处理。主要用于一些需要自定义布局策略或性能要求较高(如动画中)的场景
- FlowDelegate Flow.delegate必须提供FlowDelegate抽象类子类的实例。

函数 | 作用
---|---
void paintChildren(FlowPaintingContext context) | 此方法中调用context.paintChild 进行重绘，而context.paintChild在重绘时使用了转换矩阵，并没有实际调整组件位置。
Size getSize(BoxConstraints constraints) | 指定Flow大小
bool shouldRepaint(covariant FlowDelegate oldDelegate)| 是否要重绘
BoxConstraints getConstraintsForChild(int i, BoxConstraints constraints)|重写控制每个子widget的constraints

## 五.[Stack、Positioned](https://book.flutterchina.club/chapter4/stack.html)(层叠布局)
- 单独Stack效果和android FrameLayout相同
- 配合Positioned实现绝对布局

### 1.Stack

属性 | 作用 | 其他
---|---|---
alignment|此参数决定如何去对齐**没有定位**(**没有使用Positioned**)或部分定位的子组件|这里特指没有在某一个轴上定位：left、right为横轴，top、bottom为纵轴，<br/>只要包含某个轴上的一个定位属性就算在该轴上有定位。
fit|用于确定**没有定位**的子组件如何去适应Stack的大小|StackFit.loose：使用子组件的大小<br/>expand：扩伸到Stack的大小
overflow|如何显示超出Stack显示空间的子组件|Overflow.clip：超出部分会被剪裁（隐藏）<br/>visible 时则不会
textDirection|参考Row、Column|-

### 2.Positioned(相当于android的AbsoluteLayout)
- left、top 、right、 bottom分别代表离Stack左、上、右、底四边的距离
- width和height用于指定需要定位元素的宽度和高度.left、width、right中width、right只能存在一个否则报错，同理top、height、bottom也是。


## 六.[Align(相对定位)、Center](https://book.flutterchina.club/chapter4/alignment.html)
### 1.Align

A用于调整子组件的位置，并可根据子组件的宽高确定自身的的宽高
- 主要属性

属性 | 作用 | 其他
---|---|---
alignment|子组件在父组件中的起始位置|使用Alignment定义的常量
widthFactor|缩放因子，确定Align 组件本身宽度|eg：子widget宽60，则Align的宽=widthFactor*widget
heightFactor|缩放因子，确定Align 组件本身高度|-

- Alignment
- FractionalOffset

### 2.Center

Align的子类并将alignment设置为Alignment.center

