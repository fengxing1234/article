---
title: Flutter异步编程原理
date: 2020-08-03 14:33:47
tags:
	- flutter
	- dart
	- 异步
categories: 
	- flutter
	- dart
	- 异步
---

# Flutter 异步编程原理

### Dart 中的事件循环模型

下面是 event loop 大致的运行图：

消息队列采用先进先出，下面引用了几张 `Dart` 开发者网站的介绍：

![image.png](https://upload-images.jianshu.io/upload_images/4118241-b99607310736c09f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

将消息换成具体的类型后，类似下图：

![image.png](https://upload-images.jianshu.io/upload_images/4118241-3c4dc2d404429156.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

这个很好理解，事件 events 加到 Event queue 里，Event loop 循环从 Event queue 里取 Event 执行。

Dart 是一种单线程模型运行语言，其运行原理如下图所示：



![image.png](https://upload-images.jianshu.io/upload_images/4118241-fcf4135399f21934.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

Dart 在单线程中是以消息循环机制来运行的，其中包含两个任务队列，一个是“微任务队列” microtask queue，另一个叫做“事件队列” event queue。 从图中可以发现，微任务队列的执行优先级高于事件队列。

现在我们来介绍一下Dart线程运行过程，如上图中所示，入口函数 main() 执行完后，消息循环机制便启动了。 首先会按照先进先出的顺序逐个执行微任务队列中的任务，当所有微任务队列执行完后便开始执行事件队列中的任务，事件任务执行完毕后再去执行微任务， 如此循环往复，生生不息。

在Dart中，所有的外部事件任务都在事件队列中，如IO、计时器、点击、以及绘制事件等，而微任务通常来源于Dart内部，并且微任务非常少， 之所以如此，是因为微任务队列优先级高，如果微任务太多，执行时间总和就越久，事件队列任务的延迟也就越久， 对于GUI应用来说最直观的表现就是比较卡，所以必须得保证微任务队列不会太长。 在事件循环中，当某个任务发生异常并没有被捕获时，程序并不会退出，而直接导致的结果是当前任务的后续代码就不会被执行了， 也就是说一个任务中的异常是不会影响其它任务执行的。

我们可以看出，将任务加入到MicroTask中可以被尽快执行，但也需要注意，当事件循环在处理MicroTask队列时，Event队列会被卡住，应用程序无法处理鼠标单击、I/O消息等等事件。 同时，当事件循坏出现异常时，dart 中也可以通过 try/catch/finally 来捕获异常。

### 任务调度

#### MicroTask 任务队列

添加到 MicroTask 任务队列两种方法：

```
import 'dart:async';

void myTask(){
  print("this is my task");
}

void main() {
  /// 1. 使用 scheduleMicrotask 方法添加
  scheduleMicrotask(myTask);
  /// 2. 使用Future对象添加
  Future.microtask(myTask);
}
复制代码
```

两个 task 都会被执行，控制台输出：

```
this is my task
this is my task
复制代码
```

#### Event 任务队列

```
import 'dart:async';

void myTask(){
  print("this is my event task");
}

void main() {
  Future(myTask);
}
复制代码
```

控制台输出:

```
this is my event task
复制代码
```

示例：

```
import 'dart:async';

void main() {
  print("main start");

  Future((){
    print("this is my task");
  });

  Future.microtask((){
    print("this is microtask");
  });

  print("main stop");
}
复制代码
```

控制台输出：

```
main start
main stop
this is microtask
this is my task
复制代码
```

可以看到，代码的运行顺序并不是按照我们的编写顺序来的，将任务添加到队列并不等于立刻执行，它们是异步执行的，当前main方法中的代码执行完之后，才会去执行队列中的任务，且MicroTask队列运行在Event队列之前。

#### 延时任务

```
Future.delayed(new  Duration(seconds:1),(){
    print('task delayed');
});
复制代码
```

使用 Future.delayed 可以使用延时任务，但是延时任务不一定准

```
import 'dart:async';
import 'dart:io';

void main() {
  var now = DateTime.now();
  print("程序运行开始时间:" + now.toString());
	Future.delayed(Duration(seconds:1),(){
    now = DateTime.now();
    print('延时任务执行时间：' + now.toString());
  });
  Future((){
    // 模拟耗时5秒
    sleep(Duration(seconds:5));
    now = DateTime.now();
    print("5s 延时任务执行时间:" + now.toString());
  });
  now = DateTime.now();
  print("程序结束执行时间:" + now.toString());
}
复制代码
```

输出结果：

```
程序运行开始时间:2019-09-11 20:46:18.321738
程序结束执行时间:2019-09-11 20:46:18.329178
5s 延时任务执行时间:2019-09-11 20:46:23.330951
延时任务执行时间：2019-09-11 20:46:23.345323
复制代码
复制代码
```

可以看到，5s 的延时任务，确实实在当前程序运行之后的 5s 后得到了执行，但是另一个延时任务，是在当前延时的基础上又延时执行的， 也就是，延时任务必须等前面的耗时任务执行完，才得到执行，这将导致延时任务延时并不准。

### Future 与 FutureBuilder

Future 的所有API的返回值仍然是一个Future对象，所以可以很方便的进行链式调用。

#### Future 使用

Future 的几种创建方法：

- Future() 默认异步方法
- Future.microtask() 微任务队列
- Future.sync() 同步方法
- Future.value() 返回指定 value 值
- Future.delayed() 延时
- Future.error() 错误

**then 回调、 catchError 捕获异常和 whenComplete 一定执行**

```
import 'dart:async';

void main() {
  print("main start");

  Future.delayed(new Duration(seconds: 2),(){
    //return "hi world!";
    throw AssertionError("Error");
  }).then((data){
    //执行成功会走到这里  
    print("success");
  }).catchError((e){
    //执行失败会走到这里
    print(e);
  }).whenComplete((){
    //无论成功或失败都会走到这里
    print("this is the end...");
  });

  print("main stop");
}
复制代码
```

输出：

```
main start
main stop
Assertion failed
复制代码
```

在本示例中，我们在异步任务中抛出了一个异常，then 的回调函数将不会被执行，取而代之的是 catchError 回调函数将被调用；但是，并不是只有 catchError 回调才能捕获错误，then 方法还有一个可选参数 onError，我们也可以它来捕获异常：

```
import 'dart:async';

void main() {
  print("main start2");

  Future.delayed(new Duration(seconds: 2), () {
    //return "hi world!";
    throw AssertionError("Error");
  }).then((data) {
    print("success");
  }, onError: (e) {
    print(e);
  });

  print("main stop2");
}
复制代码
```

结果：

```
main start2
main stop2
Assertion failed
复制代码
```

**wait 完成后回调** Future.wait 表示多个任务都完成之后的回调。

```
import 'dart:async';

void main() {
  print("main start4");

  Future.wait([
    // 2秒后返回结果
    Future.delayed(new Duration(seconds: 2), () {
      return "hello";
    }),
    // 4秒后返回结果
    Future.delayed(new Duration(seconds: 4), () {
      return " world";
    })
  ]).then((results){
    print(results[0]+results[1]);
  }).catchError((e){
    print(e);
  });

  print("main stop4");
}
复制代码
```

结果：

```
main start4
main stop4
hello world
复制代码
```

#### FutureBuilder 的使用

很多时候我们会依赖一些异步数据来动态更新UI，比如在打开一个页面时我们需要先从互联网上获取数据，在获取数据的过程中我们显式一个加载框，等获取到数据时我们再渲染页面；又比如我们想展示Stream（比如文件流、互联网数据接收流）的进度。

```
FutureBuilder({
  this.future, // FutureBuilder依赖的Future，通常是一个异步耗时任务
  this.initialData, // 初始数据，用户设置默认数据
  @required this.builder, // Widget构建器:该构建器会在Future执行的不同阶段被多次调用
})
复制代码
```

builder 构造器参数结构

```
Function (BuildContext context, AsyncSnapshot snapshot)
复制代码
```

示例：

```
import 'package:flutter/material.dart';

void main() => runApp(MyApp());

class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Flutter Demo',
      theme: ThemeData(
        primarySwatch: Colors.blue,
      ),
      home: MyHomePage(title: 'Flutter Demo Home Page'),
    );
  }
}

class MyHomePage extends StatefulWidget {
  MyHomePage({Key key, this.title}) : super(key: key);
  final String title;
  @override
  _MyHomePageState createState() => _MyHomePageState();
}

class _MyHomePageState extends State<MyHomePage> {
  Future<String> mockNetworkData() async {
    return Future.delayed(Duration(seconds: 2), () => "这是网络请求的数据。。。");
  }
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text(widget.title),
      ),
      body: Center(
        child: FutureBuilder<String>(
          future: mockNetworkData(),
          builder: (BuildContext context, AsyncSnapshot snapshot) {
            // 请求已结束
            if (snapshot.connectionState == ConnectionState.done) {
              if (snapshot.hasError) {
                // 请求失败，显示错误
                return Text("Error: ${snapshot.error}");
              } else {
                // 请求成功，显示数据
                return Text("Contents: ${snapshot.data}");
              }
            } else {
              // 请求未结束，显示loading
              return CircularProgressIndicator();
            }
          },
        ),
      ),
    // This trailing comma makes auto-formatting nicer for build methods.
    );
  }
}
复制代码
```

演示效果：

![image.png](https://upload-images.jianshu.io/upload_images/4118241-5c76135b31fcc5d1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### async/await

在 Dart1.9 中加入了 async 和 await 关键字，有了这两个关键字，我们可以更简洁的编写异步代码，而不需要调用 Future 相关的 API。

#### 使用特点

- 被修饰的方法会将一个 Future 对象作为返回值；
- 该方法会同步执行其中的方法的代码直到第一个 await 关键字，然后它暂停该方法其他部分的执行；
- 一旦由 await 关键字引用的 Future 任务执行完成，await 的下一行代码将立即执行。

```
// 导入io库，调用sleep函数
import 'dart:io';

// 模拟耗时操作，调用sleep函数睡眠2秒
doTask() async{
  await sleep(const Duration(seconds:2));
  return "Ok";
}

// 定义一个函数用于包装
test() async {
  var r = await doTask();
  print(r);
}

void main(){
  print("main start");
  test();
  print("main end");
}
复制代码
```

结果：

```
main start
main end
Ok
复制代码
```

### Stream 与 StreamBuilder

Stream 也是用于接收异步事件数据，和 Future 不同的是，它可以接收多个异步操作的结果（成功或失败）。 也就是说，在执行异步任务时，可以通过多次触发成功或失败事件来传递结果数据或错误异常。 Stream 常用于会多次读取数据的异步任务场景，如网络内容下载、文件读写等。

#### Stream 的使用

```
void main(){
  print("main start");
  Stream.fromFutures([
    // 1秒后返回结果
    Future.delayed(new Duration(seconds: 1), () {
      return "hello 1";
    }),
    // 抛出一个异常
    Future.delayed(new Duration(seconds: 2),(){
      throw AssertionError("Error");
    }),
    // 3秒后返回结果
    Future.delayed(new Duration(seconds: 3), () {
      return "hello 3";
    })
  ]).listen((data){
    print(data);
  }, onError: (e){
    print(e.message);
  },onDone: (){

  });
  print("main end");
}
复制代码
```

结果：

```
main start
main end
hello 1
Error
hello 3
复制代码
```

#### StreamBuilder 的使用

StreamBuilder正是用于配合Stream来展示流上事件（数据）变化的UI组件。 下面看一下StreamBuilder的默认构造函数：

```
StreamBuilder({
  Key key,
  this.initialData,
  Stream<T> stream,
  @required this.builder,
})
复制代码
```

可以看到和 FutureBuilder 的构造函数只有一点不同：前者需要一个 future，而后者需要一个 stream。 示例：

```
Stream<int> counter() {
  return Stream.periodic(Duration(seconds: 1), (i) {
    return i;
  });
}

Widget buildStream (BuildContext context) {
  return StreamBuilder<int>(
    stream: counter(), //
    //initialData: ,// a Stream<int> or null
    builder: (BuildContext context, AsyncSnapshot<int> snapshot) {
      if (snapshot.hasError)
        return Text('Error: ${snapshot.error}');
      switch (snapshot.connectionState) {
        case ConnectionState.none:
          return Text('没有Stream');
        case ConnectionState.waiting:
          return Text('等待数据...');
        case ConnectionState.active:
          return Text('active: ${snapshot.data}');
        case ConnectionState.done:
          return Text('Stream已关闭');
      }
      return null; // unreachable
    },
  );
}

return Scaffold(
  appBar: AppBar(
    title: Text(widget.title),
  ),
  body:Center(
    child:  buildStream(context),
  )
);
复制代码
```

效果：

![image.png](https://upload-images.jianshu.io/upload_images/4118241-b812d93a5fae72ea.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



### isolate

Dart 是基于单线程模型的语言。但是在开发当中我们经常会进行耗时操作比如网络请求，这种耗时操作会堵塞我们的代码，所以在 Dart 也有并发机制，名叫 isolate。 APP 的启动入口 main 函数就是一个类似 Android 主线程的一个主 isolate。和 Java 的 Thread 不同的是，Dart 中的 isolate 无法共享内存，类似于 Android 中的多进程。

**isolate 与普通线程的区别** isolate 神似 Thread，但实际上两者有本质的区别。操作系统内的线程之间是可以有共享内存的，而 isolate 不能。



![image.png](https://upload-images.jianshu.io/upload_images/4118241-3769a07ac8e2e680.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



#### isolate 工作原理

isolate 创建主要步骤：

- 初始化isolate数据结构
- 初始化堆内存(Heap)
- 进入新创建的isolate，使用跟isolate一对一的线程运行isolate
- 配置Port
- 配置消息处理机制(Message Handler)
- 配置Debugger，如果有必要的话
- 将isolate注册到全局监控器（Monitor）



![image.png](https://upload-images.jianshu.io/upload_images/4118241-785ef3da2dccf846.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



Dart 本身抽象了 isolate 和 thread，实际上底层还是使用操作系统的提供的 OSThread，Dart isolate 跟 Flutter Runner 是相互独立的，他们通过任务调度机制相互协作。

#### 1）UI Task Runner Thread（Dart Runner）

UI Task Runner 用于执行 Dart root isolate 代码（isolate 我们后面会讲到，姑且先简单理解为 Dart VM 里面的线程）。Root isolate 比较特殊，它绑定了不少 Flutter 需要的函数方法，以便进行渲染相关操作。对于每一帧，引擎要做的事情有：

- Root isolate 通知 Flutter Engine 有帧需要渲染。
- Flutter Engine 通知平台，需要在下一个 vsync 的时候得到通知。
- 平台等待下一个 vsync。
- 对创建的对象和 Widgets 进行 Layout 并生成一个 Layer Tree，这个 Tree 马上被提交给 Flutter Engine。当前阶段没有进行任何光栅化，这个步骤仅是生成了对需要绘制内容的描述。
- 创建或者更新 Tree，这个 Tree 包含了用于屏幕上显示 Widgets 的语义信息。这个东西主要用于平台相关的辅助 Accessibility 元素的配置和渲染。

#### 2）GPU Task Runner

GPU Task Runner 主要用于执行设备 GPU 的指令。UI Task Runner 创建的 Layer Tree 是跨平台的，它不关心到底由谁来完成绘制。GPU Task Runner 负责将 Layer Tree 提供的信息转化为平台可执行的 GPU 指令。GPU Task Runner 同时负责绘制所需要的 GPU 资源的管理。资源主要包括平台 Framebuffer，Surface，Texture和Buffers 等。

一般来说 UI Runner 和 GPU Runner 跑在不同的线程。GPU Runner 会根据目前帧执行的进度去向 UI Runner 要求下一帧的数据，在任务繁重的时候可能会告诉 UI Runner 延迟任务。这种调度机制确保 GPU Runner 不至于过载，同时也避免了 UI Runner 不必要的消耗。

建议为每一个 Engine 实例都新建一个专用的 GPU Runner 线程。

#### 3）IO Task Runner

前面讨论的几个 Runner 对于执行流畅度有比较高的要求。Platform Runner 过载可能导致系统 WatchDog 强杀，UI 和 GPU Runner 过载则可能导致 Flutter 应用的卡顿。但是 GPU 线程的一些必要操作，例如 IO，放到哪里执行呢？答案正是 IO Runner。

IO Runner 的主要功能是从图片存储（比如磁盘）中读取压缩的图片格式，将图片数据进行处理为 GPU Runner 的渲染做好准备。IO Runner 首先要读取压缩的图片二进制数据（比如 PNG，JPEG），将其解压转换成 GPU 能够处理的格式然后将数据上传到 GPU。

获取诸如 ui.Image 这样的资源只有通过 async call 去调用，当调用发生的时候 Flutter Framework 告诉 IO Runner 进行加载的异步操作。

IO Runner 直接决定了图片和其它一些资源加载的延迟间接影响性能。所以建议为 IO Runner 创建一个专用的线程。

#### 4）Platform Task Runner

Flutter Engine 的主 Task Runner，类似于 Android Main Thread 或者 iOS 的 Main Thread。但是需要注意他们还是有区别的。

一般来说，一个 Flutter 应用启动的时候会创建一个 Engine 实例，Engine创建的时候会创建一个线程供 Platform Runner 使用。

跟 Flutter Engine 的所有交互（接口调用）必须在 Platform Thread 进行，否则可能导致无法预期的异常。这跟 iOS UI 相关的操作都必须在主线程进行相类似。需要注意的是在 Flutter Engine 中有很多模块都是非线程安全的。

阻塞 Platform Thread 不会直接导致 Flutter 应用的卡顿（跟 iOS Android 主线程不同）。尽管如此，也不建议在这个 Runner 执行繁重的操作，长时间卡住 Platform Thread 应用有可能会被系统 Watchdog 强杀。

#### 各个平台的线程配置

**1.iOS** 为每个引擎实例的UI，GPU和IO任务运行程序创建专用线程。所有引擎实例共享相同的Platform Thread和Platform Task Runner。 **2.Android** 为每个引擎实例的UI，GPU和IO任务运行程序创建专用线程。所有引擎实例共享相同的Platform Thread和Platform Task Runner。 **3.Fuchsia** 每一个Engine实例都为UI，GPU，IO，Platform Runner创建各自新的线程。 **4.Flutter Tester (used by flutter test)** UI，GPU，IO和Platform任务运行器使用相同的主线程。

#### 创建 isolate：使用 spawnUri

spawnUri 方法有三个必须的参数

- 第一个是 Uri，指定一个新 Isolate 代码文件的路径;
- 第二个是参数列表，类型是List;
- 第三个是动态消息。

需要注意，用于运行新 Isolate 的代码文件中，必须包含一个 main 函数，它是新 Isolate 的入口方法，该 main 函数中的 args 参数列表， 正对应 spawnUri 中的第二个参数。如不需要向新 Isolate 中传参数，该参数可传空 List 主 Isolate 中的代码：

```
import 'dart:isolate';

void main() {
  print("main isolate start");
  create_isolate();
  print("main isolate stop");
}

// 创建一个新的 isolate
create_isolate() async {
  ReceivePort rp = ReceivePort();
  ///创建发送端
  SendPort port1 = rp.sendPort;
  ///创建新的 isolate ，并传递了发送端口。
  Isolate newIsolate = await Isolate.spawnUri(Uri(path: "./other_task.dart"), ["hello, isolate", "this is args"], port1);

  SendPort port2;
  rp.listen((message){
    print("main isolate message: $message");
    if (message[0] == 0){
      port2 = message[1];
    }else{
      port2?.send([1,"这条信息是 main isolate 发送的"]);
    }
  });

  // 可以在适当的时候，调用以下方法杀死创建的 isolate
  // newIsolate.kill(priority: Isolate.immediate);
}
复制代码
```

在主 isolate 文件的同级路径下，新建一个 other_task.dart ，内容如下：

```
import 'dart:isolate';
import 'dart:io';

///接收到了发送端口。
void main(args, SendPort port1) {
  print("isolate_1 start");
  print("isolate_1 args: $args");

  ReceivePort receivePort = ReceivePort();
  SendPort port2 = receivePort.sendPort;

  receivePort.listen((message){
    print("isolate_1 message: $message");
  });

  // 将当前 isolate 中创建的SendPort发送到主 isolate中用于通信
  port1.send([0, port2]);
  // 模拟耗时5秒
  sleep(Duration(seconds:5));
  port1.send([1, "isolate_1 任务完成"]);
  print("isolate_1 stop");
}
复制代码
```

运行主 isolate,输出结果：

```
main isolate start
main isolate stop
isolate_1 start
isolate_1 args: [hello, isolate, this is args]
main isolate message: [0, SendPort]
isolate_1 stop
main isolate message: [1, isolate_1 任务完成]
isolate_1 message: [1, 这条信息是 main isolate 发送的]
复制代码
复制代码
```

isolate 之间的通信时通过 ReceivePort 来完成的，ReceivePort 可以理解为一根水管，消息的传递方向时固定的，把这个水管的发送端发送给对方， 对方就能通过这个水管发送消息。

#### 创建 isolate：通过 spawn

除了使用 spawnUri，更常用的是使用 spawn 方法来创建新的 Isolate，我们通常希望将新创建的 Isolate 代码和 main Isolate 代码写在同一个文件， 且不希望出现两个 main 函数，而是将指定的耗时函数运行在新的 Isolate，这样做有利于代码的组织和代码的复用。 spawn 方法有两个必须的参数

- 第一个是需要运行在新 Isolate 的耗时函数
- 第二个是动态消息，该参数通常用于传送主 Isolate 的 SendPort 对象
- （可选）第三个是命名方法，这个值可以做表意使用，但他不是唯一的



```
import 'dart:isolate';
import 'dart:io';

void main() {
  print("主程序开始");
  create_isolate();
  print("主程序结束");
}

// 创建一个新的 isolate
create_isolate() async{
  ReceivePort rp = new ReceivePort();
  SendPort port1 = rp.sendPort;

  Isolate newIsolate = await Isolate.spawn(doWork, port1);

  SendPort port2;
  rp.listen((message){
    print("main isolate message: $message");
    if (message[0] == 0){
      port2 = message[1];
    }else{
      port2?.send([1,"这条信息是 main isolate 发送的"]);
    }
  });
}

// 处理耗时任务
void doWork(SendPort port1){
  print("new isolate start");
  ReceivePort rp2 = new ReceivePort();
  SendPort port2 = rp2.sendPort;

  rp2.listen((message){
    print("doWork message: $message");
  });

  // 将新isolate中创建的SendPort发送到主isolate中用于通信
  port1.send([0, port2]);
  // 模拟耗时5秒
  sleep(Duration(seconds:5));
  port1.send([1, "doWork 任务完成"]);

  print("new isolate end");
}
复制代码
```

结果：

```
主程序开始
主程序结束
new isolate start
main isolate message: [0, SendPort]
new isolate end
main isolate message: [1, doWork 任务完成]
doWork message: [1, 这条信息是 main isolate 发送的]
复制代码
```

#### 创建 isolate：Compute 函数

Compute 函数对 isolate 的创建和底层的消息传递进行了封装，使得我们不必关系底层的实现，只需要关注功能实现。

首先我们需要：

1. 一个函数：必须是顶级函数或静态函数
2. 一个参数：这个参数是上面的函数定义入参（函数没有参数的话就没有）
3. （可选）debugLabel：字典类参数，命名线程

比如计算斐波那契数列：

```
void main() async{
  // 调用compute函数，compute函数的参数就是想要在isolate里运行的函数，和这个函数需要的参数
  print(await compute(syncFibonacci, 20));
  runApp(MyApp());
}

int syncFibonacci(int n){
  return n < 2 ? n : syncFibonacci(n-2) + syncFibonacci(n-1);
}
复制代码
```

运行后的结果如下：

```
flutter: 6765
```