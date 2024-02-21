---
title: 三基础组件-1Widget简介
date: 2021-8-31
categories: [flutter,Flutter Learning] 
tags: [flutter,Flutter Learning] 
---

Flutter中几乎所有的对象都是一个Widget。它不仅可表示UI元素，也可表示一些功能性的组件如：用于手势检测的 GestureDetector widget、用于APP主题数据传递的Theme等等

## 1.Widget与Element
  - Widget是“描述一个UI元素的配置数据”。**Widget只是UI元素的一个配置数据，并且一个Widget可以对应多个Element**
  - 屏幕上显示元素的类是Element，也就是说Widget只是描述Element的配置数据

## 2.Widget主要接口

```dart
//1.继承自DiagnosticableTree作用是提供调试信息。
@immutable
abstract class Widget extends DiagnosticableTree {
  const Widget({ this.key });
//2.决定是否在下一次build时复用旧的widget，决定的条件在canUpdate()方法中。
  final Key key;

//3.Flutter Framework在构建UI树时，会先调用此方法生成对应节点的Element对象。此方法是Framework隐式调用的，在我们开发过程中基本不会调用到。
  @protected
  Element createElement();

//4. 复写父类的方法，主要是设置诊断树的一些特性
  @override
  void debugFillProperties(DiagnosticPropertiesBuilder properties) {
    super.debugFillProperties(properties);
    properties.defaultDiagnosticsTreeStyle = DiagnosticsTreeStyle.dense;
  }

//5.静态方法。用于在Widget树重新build时是否复用旧的widget（是否用新的Widget对象去更新旧UI树上所对应的Element对象的配置）
//只要newWidget与oldWidget的runtimeType和key同时相等时就会用newWidget去更新Element对象的配置，否则就会创建新的Element。
  static bool canUpdate(Widget oldWidget, Widget newWidget) {
    return oldWidget.runtimeType == newWidget.runtimeType && oldWidget.key == newWidget.key;
  }
}
```

## 3.Flutter中的四棵树
  - Flutter中的四棵树

    树|描述|其他
    ---|---|---
    Widget|Widget描述UI元素的信息|Widget只是描述UI元素的配置信息
    Element|由idget树生成一颗Element树.|Element树中所有节点都继承自 Element
    Render|由Element树生成Render树|Render树所有节点都继承自RenderObject
    Layer|由Render树生成Layer树，绘制显示在屏幕上|Layer树所有节点都继承自Layer

  - Widget、Element、Render对应关系

    树与树|是否一一对应|描述
    ---|---|---
    Widget & Element |是|
    Element & Render|否|⽐如StatelessWidget 和 StatefulWidget 都没有对应的 RenderObject
## 4.StatelessWidget（无状态Widget）
  - 1.StatelessWidget用于不需要维护状态的场景
  - 2.StatelessElement 间接继承自Element类，与StatelessWidget相对应（作为其配置数据）。
  - Widget build(BuildContext context) 中的`context`.context是当前widget在widget树中位置中执行”相关操作“的一个句柄，比如它提供了从当前widget开始向上遍历widget树以及按照widget类型查找父级widget的方法。

  ```dart
      // 在Widget树中向上查找最近的父级`Scaffold` widget
      Scaffold scaffold = context.findAncestorWidgetOfExactType<Scaffold>();
  ```

## 5.StatefulWidget（有状态Widget）

```dart
abstract class StatefulWidget extends Widget {
  const StatefulWidget({ Key key }) : super(key: key);

//1.StatefulElement 间接继承自Element类，与StatefulWidget相对应（作为其配置数据）。。
  @override
  StatefulElement createElement() => new StatefulElement(this);

//2.用于创建和Stateful widget相关的状态，它在Stateful widget的生命周期中可能会被多次调用。
  @protected
  State createState();
}
```

## 6.State
  - 1.一个StatefulWidget类对应一个State类，State表示与其对应的StatefulWidget要维护的状态，State中的保存的状态信息可以：
    - 1.在widget 构建时可以被同步读取。
    - 2.在widget生命周期中可以被改变。当State被改变时，可以手动调用其setState()方法通知framework状态发生改变，framework在收到消息后，会重新调用其build方法重新构建widget树，从而达到更新UI的目的。

  - 2.State中有两个常用属性：
    - 1.widget：与该State实例关联的widget实例。由Flutter framework动态设置。
    - 2.context：StatefulWidget对应的BuildContext

  - 3.State生命周期：

  ```dart
  class _XXWidgetState extends State<CounterWidget> {  
  //1.当Widget第一次插入到Widget树时会被调用.(只会调用一次)
    void initState() {}

  //2.当State对象的依赖发生变化时会被调用
  //如：在之前build() 中包含了一个InheritedWidget，然后在之后的build() 中InheritedWidget发生了变化，那么此时InheritedWidget的子widget的didChangeDependencies()回调都会被调用。
  //典型的场景是当系统语言Locale或应用主题改变时，Flutter framework会通知widget调用此回调。
    void didChangeDependencies() {}

  //3.用于构建Widget子树.会在如下场景被调用：
  //  ①.在调用initState()之后。
  //  ②.在调用didUpdateWidget()之后。
  //  ③.在调用setState()之后。
  //  ④.在调用didChangeDependencies()之后。
  //  ⑤.在State对象从树中一个位置移除后（会调用deactivate）又重新插入到树的其它位置之后。
    Widget build(BuildContext context) {}

  //4.在widget重新构建时。
  // framework会调用Widget.canUpdate来检测Widget树中同一位置的新旧节点，然后决定是否需要更新，如果Widget.canUpdate返回true则会调用此回调。
    void didUpdateWidget(CounterWidget oldWidget) {}

  //5.当State对象从树中被移除时。
    void deactivate() {}

  //6.当State对象从树中被永久移除时调用。通常在此回调中释放资源。
    void dispose() {}

  //7.专为开发调试而提供。在热重载(hot reload)时会被调用，在Release模式下永远不会被调用。
    void reassemble() {}
  }
  ```

  - 4.为什么要将build方法放在State中，而不是放在StatefulWidget中？
    - ①.状态访问不便。
    > 假设build放在StatefulWidget中。为了在每次状态改变时StatefulWidget中的build方法需要访问状态，即需暴露State给StatefulWidget,破坏状态的私密性
    - ②.继承StatefulWidget不便。

## 7.在Widget树中获取State对象
  - (1).findAncestorStateOfType：从当前节点沿着widget树向上查找指定类型的StatefulWidget对应的State对象。

  ```dart
  // 查找父级最近的Scaffold对应的ScaffoldState对象
  ScaffoldState _state = context.findAncestorStateOfType<ScaffoldState>();
  ```

  - (2).通过GlobalKey。步骤分两步：
    - ①.给目标StatefulWidget添加GlobalKey。

    ```dart
    //定义一个globalKey, 由于GlobalKey要保持全局唯一性，我们使用静态变量存储
    static GlobalKey<ScaffoldState> _globalKey= GlobalKey();
    Scaffold(
        key: _globalKey , //设置key
    )
    ```

    - ②.通过GlobalKey来获取State对象
    
    ```dart
    _globalKey.currentState.openDrawer()
    ```

    如果一个widget设置了GlobalKey，那么我们便可以通过`globalKey.currentWidget`获得该`widget`对象、`globalKey.currentElement`来获得widget对应的`element`对象，如果当前widget是StatefulWidget，则可以通过`globalKey.currentState`来获得该widget对应的`state`对象。

    > 使用GlobalKey开销较大，如果有其他可选方案，应尽量避免使用它。另外同一个GlobalKey在整个widget树中必须是唯一的，不能重复。

