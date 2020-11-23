---
title: Isolate使用场景
date: 2020-08-03 16:18:01
tags:
---

> Root isolate负责UI渲染以及用户交互操作，需要及时响应，当存在耗时操作，则必须创建新的isolate，否则UI渲染会被阻塞。Flutter 实现异步的方式有：`async、awite`、`Futrue`，但是这些本质还是 handle 队列那套，更何况消息队列还是跑在 UI 线程里的，要是你真有什么耗时的操作放在这里妥妥的光剩卡了。所以该开新线程还是得开，这里我们来说说怎么 new Isolate 和 Isolate 之间的通讯，但后再来看看系统给我们提供的简便函数：`compute`



## 一、Isolate

Root isolate负责UI渲染以及用户交互操作，需要及时响应，当存在耗时操作，则必须创建新的isolate，否则UI渲染会被阻塞。 创建isolate的方法便是Isolate.spawn()。Flutter的dart代码默认运行在root isolate，即便是创建的异步future方法也不例外，而对于耗时方法势必会导致UI线程阻塞，从而导致卡顿。如何解决这个问题呢，那就是针对耗时的操作，需要创建新的isolate，isolate采用的是同一进程内的多线程间内存不共享的设计方案。dart语言之所以这样设计isolate，是为了避免线程竞争出现。正因为isolate之间无法共享内存数据，为此同时设计了套SendPort/ReceivePort的Port端口通信机制，让各isolate间彼此独立，但又可以相互的通信。对于port通信的实现原理其实还是异步消息机制。Dart的Isolate是由Dart VM管理的，对于Flutter引擎也无法直接访问。

通过阅读整个isolate源码过程，不难发现，每一次创建新的isolate的同时会通过pthread_create()来创建一个OSThread线程，isolate的工作便是在该线程中运行，其实可以简单理解成isolate是一种内存不共享的线程，isolate只是在系统现在线程基础之上做了一层封装而已。可通过Dart_CreateIsolate和Dart_ShutdownIsolate分别对应isolate的创建与关闭。

## Isolate 使用场景

Isolate 虽好，但也有合适的使用场景，不建议滥用 Isolate，应尽可能多的使用Dart中的事件循环机制去处理异步任务，这样才能更好的发挥Dart语言的优势

那么应该在什么时候使用Future，什么时候使用Isolate呢？一个最简单的判断方法是根据某些任务的平均时间来选择：

- 方法执行在几毫秒或十几毫秒左右的，应使用Future
- 如果一个任务需要几百毫秒或之上的，则建议创建单独的Isolate

除此之外，还有一些可以参考的场景

- JSON 解码
- 加密
- 图像处理：比如剪裁
- 网络请求：加载资源、图片