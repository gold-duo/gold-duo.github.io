---
title: QA-flutter
date: 2021-8-31 22:00
categories: [flutter]
tags: [flutter,Q&A] 
---

## 6.android studio打开flutter项目`android`下面的java文件`import`标红问题。
> flutter插件太烂了，它需要在另外一个studio实例里根目录打开`android`.右键`android`->`flutter`->`Open android module in android studio`.


## 5.Could not resolve all artifacts for configuration ':fluttertoast:classpath'.

打开`\flutter_sdk\.pub-cache\hosted\pub.flutter-io.cn\fluttertoast-x.x.x\android\build.gradle`修改`repositories`为国内镜像或者翻墙

## 4.flutter upgrade报`unable to access 'https://github.com/flutter/flutter.git/'`
`git config --global http.sslVerify false`

## 3.[isolate](https://github.com/dart-lang/sdk/blob/3f23b9b69de2fc3b2f156a781c4236d79d1715ed/runtime/vm/isolate.cc)、[linke2](https://www.yuque.com/xytech/flutter/kwoww1#46xshe)
- 1.由于dart是单线程，而async、await运行在event队列，所以**async、await并不是并发**。
- 2.isolate创建步骤`(引入'dart:isolate'包)`
    - 1.通过`Isolate.spawn`创建isolate

    ```dart
    //1,创建ReceivePort，因为SendPort需要通过ReceivePort来获取
    ReceivePort receivePort = ReceivePort();
    
    //2,创建isolate
    //  dataLoader是入口函数，必须且是static
    //  receivePort.sendPort是传入给dataLoader的参数
    await Isolate.spawn(dataLoader, receivePort.sendPort);
    ```
    - 2.发送数据
    ```dart
    //1,获取sendPort用于发送数据（dataLoader函数中的relyTo）
    SendPort sendPort = await receivePort.first;//这个receivePort就是1中创建的receivePort
    //2,创建用于接收消息的ReceivePort,在返回消息时需要用到(dataloader中的relayTo)
    ReceivePort response = ReceivePort();
    //3,发送数据
    //  [msg, response.sendPort]:发送的消息
    sendPort.send([msg, response.sendPort]);
    //4,获取返回数据
    response.first;
    ```
    - 3.isolate入口函数
    ```dart
    //创建isolate必须要的参数
    static void dataLoader(SendPort sendPort){
      final port = new ReceivePort();
      //1,绑定发送来的port
      sendPort.send(port.sendPort);
      
      //2,监听消息
      port.listen((message){
        //获取数据并解析
        final data = message[0] as String;  //对应发送时的数组元素1
        final replyTo = message[1] as SendPort;
        //3,返回结果
        replyTo.send(xxxx);
      });
    }
    ```
    - 4.完整例子

    ```dart
    loadData2() async {
      ReceivePort receivePort = ReceivePort();
      await Isolate.spawn(dataLoader, receivePort.sendPort);
      print("spanw....");

      // The 'echo' isolate sends its SendPort as the first message
      SendPort sendPort = await receivePort.first;
      print("await receivePort.first....");

      List msg = await sendReceive(sendPort, "https://jsonplaceholder.typicode.com/posts");
      print("sendReceive!!");
      setState(() {
        widgets = msg;
      });
    }

    // the entry point for the isolate
    static dataLoader(SendPort sendPort) async {
      // Open the ReceivePort for incoming messages.
      ReceivePort port = ReceivePort();

      // Notify any other isolates what port this isolate listens to.
      sendPort.send(port.sendPort);

      await for (var msg in port) {
        String data = msg[0];       //对应sendReceive 中port.send([msg, response.sendPort]);
        SendPort replyTo = msg[1];

        String dataURL = data;
        http.Response response = await http.get(dataURL);
        // Lots of JSON to parse
        replyTo.send(json.decode(response.body));
      }
    }

    Future sendReceive(SendPort port, msg) {
      print("sendReceive...");
      ReceivePort response = ReceivePort();
      port.send([msg, response.sendPort]);
      return response.first;
    }
  }
    ```
- 3.isolate的简化调用方法`compute`引入`package:flutter/foundation.dart'`包

```dart
 loadData() async{
    List msg =  await compute(_httpReq, "https://jsonplaceholder.typicode.com/posts");
    setState(() {
      widgets = msg;
    });
  }

  static Future<List> _httpReq(String dataURL) async{
        http.Response response = await http.get(dataURL);
        return json.decode(response.body);
  }
```

## 2.配置国内镜像
- mac
打开`~/.bash_profile`添加

```bash
export PUB_HOSTED_URL=https://pub.flutter-io.cn
export FLUTTER_STORAGE_BASE_URL=https://storage.flutter-io.cn
```

检查`~/.zlogin`是否存在一下配置

```bash
if [ -f ~/.bash_profile ]; then
   source ~/.bash_profile
fi
```

- windows添加到环境变量

## 1.flutter Running Gradle task 'assmebleDebug'
- 1.修改flutter sdk目录的`/packages/flutter_tools/gradle/flutter.gradle`

```gradle
    repositories {
//       google()
//       jcenter()
        maven{url 'https://maven.aliyun.com/repository/google'}
        maven{url 'https://maven.aliyun.com/repository/public' }
        //maven{url 'https://maven.aliyun.com/repository/gradle-plugin'}
        //mavenLocal()
        //mavenCentral()
    }
```

- 2.修改项目目录下`/android/build.gradle` 同上
- 3.**检查`flutter.gradle`中的`classpath 'com.android.tools.build:gradle:xxx'`和项目中的是否一致**（很多疑难杂症都是版本不一致导致的）