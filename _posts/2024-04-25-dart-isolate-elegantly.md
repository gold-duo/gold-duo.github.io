---
title: dart如何优雅的调用isolate
date: 2024-04-02 21:00
categories: [dart]
tags: [flutter,dart,isolate,async,compute,spawn,sendPort,ReceivePort,isolate_classy] 
---

实现的library [isolate_classy](https://pub.dev/packages/isolate_classy)已上传至 pub.dev.

## dart异步的现状

dart的isolate提供的api并好用。要么使用巨复杂的`Isolate.spawn`用`sendPort`、`ReceivePort`按自己的场景来构造特例代码，要么使用提供的`compute`(这个是flutter foundation的功能)和`Isolate.run`函数。`compute`封装自`Isolate.run`允许传入一个参数，多个参数就得自己构造List或者Map来传递参数了。


## isolate中函数多参数的传递方式

无论官方的`Isolate.spawn`还是`Isolate.run`处理复杂的参数均是通过`List`来间接传递。那么面对 flutter中用的最普遍的命名参数该如何处理呢？

## isolate中函数命名参数的传递

是否就无法在isolate中传递flutter中使用的普遍的命名参数了呢？

答案是：是又不是。是的话我们并不能直接像`fn(namedArg:value1,...)`这么直呼`namedArg`，但要通过符号(`Symbol`)来构造相应的参数。

### 函数是什么东西？

在dart中万物皆对象，函数也是一个对象。我们可以把函数赋值给一个`Function`对象变量（比如`Function f=main;`）然后这个变量就可以调用函数一样了。那么在dart中是怎样处理这个调用过程的呢？

若是在C中函数的赋值不过是函数指针赋值。调用函数指针和函数是一个东西，二进制世界里函数调用就是参数入栈和地址跳转，但是编译器为这一切添加了类型判断(函数的签名统一)。

回到dart中。`Function`是一个被泛化的接口。当我们将一个函数(比如`int fun1(int,String)`)赋值给`Function`对象的变量(`Function fn=fun1;`)后，原来的类型信息就将丢失，这个变量没有了`fun1`的签名，我们可以在代码里随便传递参数:

```dart
fn();//缺少参数
fn('abc',888);//类型不一样
fn(1,'a',22);//多了参数
```

上面的代码这么写是没问题的，不过在运行时dart将会抛出`noSuchMethod`异常。也就是说当函数赋值给`Function`对象后，类型检查就交给了dart vm，而不是在编译阶段。从这点来说dart又具备了一定的动态性了。

### dart函数调用的动态性

虽然dart不是脚本语言不像python那样自由，但是确确实实的这个`Function`对象让我们有了很大的自由空间。或许您会问虽然编译时不校验函数签名，但是在vm运行时还是校验的，何来自由呢？

`Function`除了是一个接口定义，其实还有一个静态方法供我们动态调用函数：

```dart
static apply(Function function, List<dynamic>? positionalArguments,
      [Map<Symbol, dynamic>? namedArguments]);
```

如函数定义，不仅提供了典型的位置参数传递，还有命名参数传递。不过这个命名参数由于是`Symbol`按照dart语法在参数前增加前缀`#`即可。
如果有一个带命名参数的函数:

```dart
String fun2(int a1, int a2,{required String desc, String? other}) =>
 "$desc:${a1 + a2}, other=$other";
```

将`fun2`赋值给`Function`对象的`fn`变量`Function fn=fun2;`,现在用`fn`变量调用函数可以是这样的:

```dart
var ret=Function.apply(fn
  [666,888], 
  { #desc:'call from function.apply',
    #other:'other value'}) as String;
```

## 优雅的函数isolate化

先定义下怎样是优雅的函数isolate:尽可能少的修改代码（比如在函数添加注解），且不影响原来的逻辑就可以让原来的函数跑在isolate。
实现优雅的函数isolate化可能首先想到的方法是用`source_gen`插件定制自动生成对应的代码。我这里用另外一种方式：编译期检测和少量的运行时代码实现isolate化。下面看看我们要实现的目标：

```dart
//假如我们有下面这个 async 函数
Future<int> test2(int a1,int a2) async {
debugPrint("test2:${Service.getIsolateID(Isolate.current)}");
await Future.delayed(const Duration(microseconds: 100));
return Future.value(a1 + a2);
}

//下面跑在主线程
final ret=await test2(66,88);

//下面跑在isolate里
final ret=await test2.isolate(66,88);
//我们只在函数的后面加了个 "isolate" 就将原来跑在主线程的 "test" 函数跑在了 isolate
```

### 典型参数函数的实现

先来看看比较典型的不带命名参数的函数如何实现。如果将上面的`test2`函数封装到一个执行isolate函数调用，照搬`compute`那么它大概可以是这样的：

```dart
typedef IsolateAction2<R,T1,T2>=Future<R> Function(T1,T2);
Future<R> runOnIsolate<R, T1, T2>(IsolateAction2<R,T1,T2> block, T1 arg1, T2 arg2){
    final list=List.filled(3, block)
    ..[1]=arg1
    ..[2]=arg2;
    //Isolate.spawn(_execute,
    //...
}
static _execute(List<dynamic> msg) async {
  //....
  final r=await Function.apply(msg[0], msg.sublist(1));
  //...
}
```

这时候我们还要要手动调用`runOnIsolate`:

```dart
final ret=await runOnIsolate(test2,66,88);
```

虽然原来的代码在逻辑上有变化，似乎也挺好的，这样是不是只要封装成一个Utilities就好了？

我们可以做的更好，隐藏掉后面的的调用，实现更符合语义化、符合逻辑的代码。


#### 用扩展方法实现`isolate`语义化

`test2.isolate`在dart语法里就是：`test2`函数多了一个`isolate` get 属性,它代理`runOnIsolate`调用。

```dart
extension IsolateExtensions2<R,T1,T2> on IsolateAction2<R,T1,T2>{
  IsolateAction2<R,T1,T2>  get isolate=> (arg1,arg2)=>runOnIsolate(this, arg1,arg2);
}
```

Ok,至此我们实现了带两个参数的函数通过在函数后加`.isolate`即将函数跑在isolate中。如果有3、4、5、6或者更多的，那么如法炮制即可。

### 带命名参数函数的实现

带典型参数函数我们可以通过上面的方法来具体代码模版，但用在命名带参数的函数里则无效了。因为命名参数的参数命名是没办法通过这种方式来实现语法匹配的。

前文[dart函数调用的动态性](#dart函数调用的动态性)中描述的`Function`中的`apply`方法中提供了命名参数的入参。

这次我们需要将扩展函数的实现定义到`Function`对象上面。

先修改runOnIsolate添加支持命名参数传递

```dart
Future<R> runOnIsolate<R, T1, T2>(IsolateAction2<R,T1,T2> block, T1 arg1, T2 arg2,[Map<Symbol, dynamic>? named]){
    final list=List.filled(4, block)
    ..[1]=arg1
    ..[2]=arg2
    ..[3]=named;
    //Isolate.spawn(_execute,
    //...
}
static _execute(List<dynamic> msg) async {
  //....
  final Map<Symbol, dynamic>? named;
  final List<dynamic> position;
  final last=msg[msg.length-1];
  if(last is Map<Symbol, dynamic>){
    named=last;
    position=msg.sublist(1,msg.length-1);
  }else{
    position=msg.sublist(1);
  }
  final r=await Function.apply(msg[0],position,named);
  //...
}
```

为`Function`对象添加`isolate`扩展属性

```dart
extension FunctionExtensions on Function {
   Function get isolate=>(arg1,arg2,[Map<Symbol, dynamic>? named]) =>
        runOnIsoLate(this, arg1, arg2, named);
}
```

现在上面[dart函数调用的动态性](#dart函数调用的动态性)的`fun2`函数跑在isolate上又可只在函数名后加`.isolate`了

```dart
final ret=await fun2.isolate(666,
  888, 
  { #desc:'call from function.apply',
    #other:'other value'});
```

#### `Function`泛化的困惑

如上[函数是什么东西?](#函数是什么东西) 所描述，当一个函数赋值给`Function`对象变量后，它的类型信息就被丢失掉(实际还是dart的类型推导不太强，rust的类型推导强得多)。上面的`FunctionExtensions`的扩展实现在`Function`对象上，意味着调用`fun2.isolate`后`fun2`函数的签名返回值类型都被丢弃了。入参参数类型可以忽略，但返回值没有了类型(dynamic)处理转换却带来了不少麻烦。

#### 用可调用对象获取类型信息

遗憾的是现阶段凭借dart语法和提供的`Function`功能，我们并不能在编译时拿到原函数的返回值类型。只能运行期强制转换或者额外添加类型信息，比如`fun2.isolate(..) as T;`、`fun2.isolate<T>(..) as T;`、`T ret=fun2.isolate<T>(..);`。额外添加类型信息总比手动类型转换好点吧？我们就来考虑该如何才能做到。

可调用对象([Callable objects](https://dart.dev/language/callable-objects))并不是什么新鲜的事请,这跟kotlin在class中实现一个invoke函数后类的实例就可以像调用函数一样调用这个invoke函数(rust 的struct也有类似的功能也是`call`函数)，不过dart变成了call函数。

为了拿到返回值类型我们定义一个对象包装下`Function`:

```dart
final class FunctionWrapper {
  final Function _fn;
  FunctionWrapper(this._fn);
  Future<R> call<R>(dynamic arg1,dynamic arg2, [Map<Symbol, dynamic>? named]) =>
      runOnIsoLate( _fn, arg1,arg2,named);
}
```

修改`FunctionExtensions`

```dart
extension FunctionExtensions on Function {
   FunctionWrapper get isolate=>FunctionWrapper(this);
}
```

现在来试试调用上面的`fun2`

```dart
String ret=await fun2.isolate(666,
  888, 
  { #desc:'call from function.apply',
    #other:'other value'});

var ret1=await fun2.isolate<String>(666,
  888, 
  { #desc:'call from function.apply',
    #other:'other value'});
```

#### 如何支持多位置参数

上面实现的是2个位置参数的带命名参数函数跑isolate。如果是0、1、3、4。。。呢？很遗憾由于我们的扩展属性实现在`Function`上，并不能统一用`isolate`getter,所以比较符合的做法是在`isolate`后面按位置参数个数追加数字。比如：isolate0、isolate1、isolate2.。。。

## 总结

至此，讲述了如何实现不带命名参数和带命名的函数实现函数后加个`.isolate`跑在isolate中。不带命名参数的实现还是比较漂亮的，所有的类型信息都完整保留且无论多少个参数都可以用`.isolate`；而带命名参数的实现还略显不足，需要用户指定返回值类型和需要根据位置参数数量在`.isolate`后加上相应的数量。

实现library [isolate_classy](https://pub.dev/packages/isolate_classy)已上传至 pub.dev.