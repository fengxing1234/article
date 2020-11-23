---
title: Dart面试题
date: 2020-11-06 10:17:32
tags:
---

# Dart面试题

### 一、Dart单线程模型



#### 问：Dart语言是单线程模型还是多线程模型的编程语言。

答：Dart是单线程模型的编写语言。

理由：**在Java和OC中，如果程序发生异常且没有被捕获，那么程序将会终止，但在Dart或JavaScript中则不会**，究其原因，这和它们的运行机制有关系，Java和OC都是多线程模型的编程语言，任意一个线程触发异常且没被捕获时，整个进程就退出了。但Dart和JavaScript不同，它们都是单线程模型，运行机制很相似(但有区别)。





#### 问：简单说说单线程模型的运行机制

![在这里插入图片描述](https://www.pianshen.com/images/737/b1bff507dd798c9656ae621a27f173f1.png)

答：**Dart 在单线程中是以消息循环机制来运行的**，**其中包含两个任务队列，一个是“微任务队列” microtask queue，另一个叫做“事件队列” event queue。从图中可以发现**，**微任务队列的执行优先级高于事件队列。**

Dart线程运行过程：如上图中所示，**入口函数 main() 执行完后，消息循环机制便启动了**。**首先会按照先进先出的顺序逐个执行微任务队列中的任务，当所有微任务队列执行完后便开始执行事件队列中的任务，事件任务执行完毕后再去执行微任务，如此循环往复，生生不息**

在事件循环中，当某个任务发生异常并没有被捕获时，程序并不会退出，而直接导致的结果是当前任务的后续代码就不会被执行了，也就是说一个任务中的异常是不会影响其它任务执行的。



#### 问：Flutter main future mirotask 的执行顺序? 

答：main-->mirotask-->future



#### 问：**Dart 是如何实现多任务并行的？**

答： Dart 是单线程的，不存在多线程，那如何进行多任务并行的呢？其实，Dart的多线程和前端的多线程有很多的相似之处。Flutter的多线程主要依赖Dart的并发编程、异步和事件驱动机制。

![在这里插入图片描述](https://segmentfault.com/img/remote/1460000021459863)

在Dart中，一个Isolate对象其实就是一个isolate执行环境的引用，一般来说我们都是通过当前的isolate去控制其他的isolate完成彼此之间的交互，而当我们想要创建一个新的Isolate可以使用Isolate.spawn方法获取返回的一个新的isolate对象，两个isolate之间使用SendPort相互发送消息，而isolate中也存在了一个与之对应的ReceivePort接受消息用来处理，但是我们需要注意的是，ReceivePort和SendPort在每个isolate都有一对，只有同一个isolate中的ReceivePort才能接受到当前类的SendPort发送的消息并且处理。





#### 问：如何将任务添加到`MicroTask`队列？

答：将任务添加到`MicroTask`队列有两种方法：

```
import  'dart:async';

void  myTask(){
    print("this is my task");
}

void  main() {
    # 1. 使用 scheduleMicrotask 方法添加
    scheduleMicrotask(myTask);

    # 2. 使用Future对象添加
    new  Future.microtask(myTask);
}
```

1. 使用 scheduleMicrotask 方法添加
2. 使用Future对象添加



#### 问：如何将任务添加到`Event`队列

```
import  'dart:async';

void  myTask(){
    print("this is my task");
}

void  main() {
    new  Future(myTask);
}
```



### 二、异步

#### 问：Dart异步编程中的 Future关键字

答：所有的 Dart 代码均运行在一个 *isolate* 的上下文环境中，该 isolate 中拥有对应 Dart 代码片段运行所需的所有内存。如果Dart代码阻塞，例如，运行一个长时间运算或者等待I/O操作，程序将要冻结。异步操作可以让你在等待一个操作完成时，进行其他工作。Dart使用Future对象（Futures）表示异步执行结果。

- Dart代码运行在一个已执行的线程内。
- 阻塞线程执行的代码能够使程序冻结。
- Future对象（Futures）表示异步执行的结果（将要完成的执行结果或I/O操作）。
- 异步函数使用await()(或者用then())，暂停执行直到future完成。
- 异步函数使用try-cache表达式，捕获错误。
- 构造一个isolate(web 使用 worker)，立刻运行代码





#### 问：如何使用Future

答：使用future，有两种方式：

- 用async 和 await
- 用Future API



#### 问：Future如何创建？如何创建同步任务？延时任务？错误任务？

答：

- Future() 默认异步方法
- Future.microtask() 微任务队列
- Future.sync() 同步方法
- Future.value() 返回指定 value 值
- Future.delayed() 延时
- Future.error() 错误



#### 问：如何获取Future回调、捕获异常？任务完成回调？

答：使用Future中的then函数。

```
import 'dart:async';

void main() {
  print("main start");

  Future.delayed(new Duration(seconds: 2),(){
    //return "hi world!";
    throw AssertionError("Error");
  }).then((data){
    //data就是回调数据  
    print("success");
  }).catchError((e){
    //执行失败会走到这里，捕获异常
    print(e);
  }).whenComplete((){
    //无论成功或失败都会走到这里
    print("this is the end...");
  });

  print("main stop");
}
```



#### 问：Future.wait()使用场景

答：`wait` 等待多个任务全部完成后在回调，如果其中某个任务失败调用失败回调`catchError`。



#### 问：async声明的方法有何意义？

答：async声明的方法有如下意义

- 方法返回一个Future
- await只能在async中出现
- 该方法会同步执行方法中的代码，直到遇到第一个await，会等待await完成再执行下面的代码
- 一旦由await引用的Future任务完成，await的下一行代码立即执行
- async函数中可以有多个await，每遇见一个就返回一个Future,效果类似then串起来的回调
- async函数可以没有await，执行完毕返回一个Future





#### 问：Stream与Future关系？区别？

答：Stream 和 Future 是 Dart 异步处理的核心 API。Future 表示稍后获得的一个数据，所有异步的操作的返回值都用 Future 来表示。**但是 Future 只能表示一次异步获得的数据。而 Stream 表示多次异步获得的数据**。比如界面上的按钮可能会被用户点击多次，所以按钮上的点击事件（onClick）就是一个 Stream 。简单地说，Future将返回一个值，而Stream将返回多次值。Dart 中统一使用 Stream 处理异步事件流。Stream 和一般的集合类似，都是一组数据，只不过一个是异步推送，一个是同步拉取。



#### 问：Stream 两种订阅模式？

Stream有两种订阅模式：**单订阅(single)** 和 **多订阅（broadcast）**。单订阅就是只能有一个订阅者，而广播是可以有多个订阅者。这就有点类似于消息服务（Message Service）的处理模式。单订阅类似于点对点，在订阅者出现之前会持有数据，在订阅者出现之后就才转交给它。而广播类似于发布订阅模式，可以同时有多个订阅者，当有数据时就会传递给所有的订阅者，而不管当前是否已有订阅者存在。



#### 问：什么是await for ？如何使用？

答：await for是不断获取stream流中的数据，然后执行循环体中的操作。它一般用在直到stream什么时候完成，并且必须等待传递完成之后才能使用，不然就会一直阻塞。

```
Stream<String> stream = new Stream<String>.fromIterable(['不开心', '面试', '没', '过']);
main() async{
  print('上午被开水烫了脚');
  await for(String s in stream){
    print(s);
  }
  print('晚上还没吃饭');
}
上午被开水烫了脚
不开心
面试
没
过
晚上还没吃饭
```

### 二、Isolate

 #### 问什么是Isolate？有什么作用？

答：Flutter的dart代码默认运行在root isolate。Root Isolate负责UI渲染以及用户交互操作，需要及时响应。即便是创建的异步future方法也不例外，而对于耗时方法势必会导致UI线程阻塞，从而导致卡顿。

当存在耗时操作，需要创建新的isolate，isolate采用的是同一进程内的多线程间内存不共享的设计方案。

> dart语言之所以这样设计isolate，是为了避免线程竞争出现。正因为isolate之间无法共享内存数据，为此同时设计了套SendPort/ReceivePort的Port端口通信机制，让各isolate间彼此独立，但又可以相互的通信。对于port通信的实现原理其实还是异步消息机制。Dart的Isolate是由Dart VM管理的，对于Flutter引擎也无法直接访问。

阅读整个isolate源码过程，每一次创建新的isolate的同时会通过pthread_create()来创建一个OSThread线程，isolate的工作便是在该线程中运行，其实可以简单理解成isolate是一种内存不共享的线程，isolate只是在系统现在线程基础之上做了一层封装而已。可通过Dart_CreateIsolate和Dart_ShutdownIsolate分别对应isolate的创建与关闭。



#### 问：创建一个新的`Isolate有几种方法?`

答：创建一个新的`Isolate`有三种方法：

```
static Future<Isolate> spawnUri()
```

`spawnUri`方法有三个必须的参数，第一个是Uri，指定一个新`Isolate`代码文件的路径，第二个是参数列表，类型是`List<String>`，第三个是动态消息。需要注意，用于运行新`Isolate`的代码文件中，必须包含一个main函数，它是新`Isolate`的入口方法，该main函数中的args参数列表，正对应`spawnUri`中的第二个参数。如不需要向新`Isolate`中传参数，该参数可传空`List`

```
static Future<Isolate> spawn()
```

除了使用`spawnUri`，更常用的是使用`spawn`方法来创建新的`Isolate`，我们通常希望将新创建的`Isolate`代码和`main Isolate`代码写在同一个文件，且不希望出现两个main函数，而是将指定的耗时函数运行在新的`Isolate`，这样做有利于代码的组织和代码的复用。`spawn`方法有两个必须的参数，第一个是需要运行在新`Isolate`的耗时函数，第二个是动态消息，该参数通常用于传送主`Isolate`的`SendPort`对象。

还有一种创建方法Flutter创建见下文。





#### 问：Flutter如何创建Isolate？

答：使用`compute`函数来创建新的`Isolate`并执行耗时任务

```
import 'package:flutter/foundation.dart';
import  'dart:io';

// 创建一个新的Isolate，在其中运行任务doWork
create_new_task() async{
  var str = "New Task";
  var result = await compute(doWork, str);
  print(result);
}

void doWork(String value){
  print("new isolate doWork start");
  // 模拟耗时5秒
  sleep(Duration(seconds:5));

  print("new isolate doWork end");
  return "complete:$value";
}
```

`compute`函数有两个必须的参数，第一个是待执行的函数，这个函数必须是一个顶级函数，不能是类的实例方法，可以是类的静态方法，第二个参数为动态的消息类型，可以是被运行函数的参数。需要注意，使用`compute`应导入`'package:flutter/foundation.dart'`包。



#### 问：说说Isolate使用场景，何时使用？

答：`Isolate`虽好，但也有合适的使用场景，不建议滥用`Isolate`，应尽可能多的使用Dart中的事件循环机制去处理异步任务，这样才能更好的发挥Dart语言的优势。

根据某些任务的平均时间来选择何时使用：

- 方法执行在几毫秒或十几毫秒左右的，应使用`Future`
- 如果一个任务需要几百毫秒或之上的，则建议创建单独的`Isolate`
- 除此之外，还有一些可以参考的场景，如JSON 解码、加密、图像处理：比如剪裁、长时间的网络请求来加载资源





### 三、Dart语法

#### 问：Dart是强类型语言

答：Dart是强类型语言，但可以用var或 dynamic来声明一个变量，Dart会自动推断其数据类型,dynamic类似c#。



#### 问：Dart是单继承还是多继承，Java呢？

答：Dart中的继承是单继承，子类重写超类的方法要用@Override，子类调用超类的方法要用super。



#### 问：没有赋初值的变量的默认值？

答：null。

#### 问：【**??=**】 和【??】的区别？【?.】如何使用

答：**??=**是赋值运算符，**??**是条件表达式。

- **??=** 赋值运算符，该运算符仅在变量值为空的时候才为其赋值；

- **??**在表达式值不为 null 的时候，它会返回左侧表达式的值，否则返回右侧表达式的值:
- **?.**这是个条件属性访问符，用于访问可能为空的对象或者属性



#### 问： 在 Dart 中 ._() 方法是什么意思？

答：

- 在 Dart 中，以下划线开头的函数表明该函数是私有的。
- 形如 Class._(); 的函数可能是一个构造函数 (在 Flutter 中，也可能是某个对象的副本的构造函数，如：AnotherClass.copy(...);)._
- _形如 Class._(); 不是必须的，除非你想要在构造函数中执行某些操作。





#### 问： dart是值传递还是引用传递

答：基本数据类型是值传递，类、对象是引用传递。



#### 问：Dart中的「..」表示什么意思？

Dart 当中的 「..」意思是 「级联操作符」，为了方便配置而使用。「..」和「.」不同的是 调用「..」后返回的相当于是 this，而「.」返回的则是该方法返回的值 。



#### 问：Dart 的作用域？

答：Dart 没有 「public」「private」等关键字，默认就是公开的，私有变量使用 下划线 _开头。



#### 问：什么是mixin机制

答：在面向对象的语言中,mixins类是一个可以把自己的方法提供给其他类使用，但却不需要成为其他类的父类。 

`mixins`是要通过非继承的方式来复用类中的代码。

例子：有一个类A，A中有一个方法a()，还有一个方法B，也想使用a()方法，而且不能用继承，那么这时候就需要用到mixins,类A就是mixins类(混入类)，类B就是要被mixins的类，对应的Dart代码如下:

```
class A {
  String content = 'A Class';

  void a(){
    print("a");
  }
}

class B with A{

}

B b = new B();
print(b.content);
b.a();

```

#### 问：使用mixins的条件是什么？

答：

1. mixins类只能继承自object
2. mixins类不能有构造函数
3. 一个类可以mixins多个mixins类
4. 可以mixins多个类，不破坏Flutter的单继承

#### 问：使用mixins需要注意什么？

1. mixins的对象是类
2. mixins绝不是继承，也不是接口，而是一种全新的特性
3. 可以mixins多个类
4. mixins的使用需要满足一定条件

#### 问：mixins要用到的关键字是什么？

答：with

- 继承 -> extends

- mixins -> with

#### 问：如果一个类中同时存在extends、mixins、implements则调用的顺序？

答：extends -> mixins -> implements。



#### 问：说说Mixin的线性化？如果在两个 Mixin 模块都有一个同名方法，这两个模块同时混入一个类，那最终调用的会是哪个模块的方法呢？

答：

- with 修饰的会覆盖 extends 中修饰的同名方法.

- with 列表中后一个模块功能会覆盖之前的



#### 问： 说说Mixin的限定？

答：例如mixins X on A，意思是要mixins X的话，得先接口实现或者继承A。这里A可以是类，也可以是接口，但是在mixins的时候用法有区别.

```
class A {
  void a(){
    print("a");
  }
}

mixin X on A{
  void x(){
    print("x");
  }
}

class mixinsX extends A with X{
}
```

#### 问：怎么来监测判定一个异步 viod 方法已经执行完成呢？

答：修改返回类型为 `Future<void>`.

```
Future<void> save(Folder folder) async {  
   .....
}
```

然后你可就可以使用 `await save(...);` 和 `save().then(...);` 来判断了。

#### 问：如何在 Dart 中将异步函数声明为变量？

在普通函数基础之上，dart 提供了一个 Future 语法糖。只要变更该函数返回类型为一个 Future 就可以了：

```
class Example {
  Future<void> Function() asyncFuncVar;
  Future<void> asyncFunc() async => print('Do async stuff...');

  Example() {
    asyncFuncVar = asyncFunc;
    asyncFuncVar().then((_) => print('Hello'));
  }
}

void main() => Example();
```

#### 问：whenCompleted () 与 then () 的异同？

答：`whenComplete` 将在 Future 没有错误的情况下完成的时候触发，而 `then` 会返回一个新的 Future ，根据其结果然后通过 onValue (成功的返回) 或者 onError (失败，带有错误的返回) 来处理。`whenCompleted` 是 “finally“。



#### 问：假定你有 list `[1,2,3,4,5,6,7]` 和 `[3,5,6,7,9,10]`. 那么该如何获取两者之间的不同呢？既 `[1, 2, 4]`？

答：

```
List<double> first = [1,2,3,4,5,6,7];
List<double> second = [3,5,6,7,9,10];
List<double> output = first.where((element) => !second.contains(element));
```

```
List<double> first = [1,2,3,4,5,6,7];
List<double> second = [3,5,6,7,9,10];
List<double> output = [];

first.forEach((element) {
    if(!second.contains(element)){
    output.add(element);
}
});

// output list 应该就是你想要的结果
```

