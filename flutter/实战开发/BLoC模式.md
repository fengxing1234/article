---
title: BLoC模式
date: 2020-06-28 13:58:24
tags:
	- Flutter
categories: 
	- Flutter
	- Flutter实战
---

# 一、概述

BLoC是**B**usiness **L**ogic **C**omponents的缩写。BLoC的哲学就是app里的所有东西都应该被认为是事件流：一部分组件订阅事件，另一部分组件则响应事件。BLoC居中管理这些会话。Dart甚至把流（Stream）内置到了语言本身里。

这个模式最好的地方就是你不需要引入任何的插件，也不需要学习其他的语法。所有需要的内容Flutter都有提供。

写app的时候，不管你用的是Flutter或者其他的框架，把类分层都是很关键的。这更像是一个非正式的约定，不是一定要在代码里有怎么样的体现。

每层，或者一组类，都负责一个总体的职责。在初始项目里有一个目录**DataLayer**。这个数据层专门用来负责app的model和与后台通信。它对UI一无所知。

# 二、BLoC

BLoC基本就是基于Dart的流（Stream）的。

![image.png](https://upload-images.jianshu.io/upload_images/4118241-919a03da365990e8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

流，和Future一样，也是在`dart:async`包里。一个流就像一个future，不同的是流不只是异步的返回一个值，流可以随着时间的推移返回很多的值。如果一个future最终是一个值的话，那么一个流就是会随着时间可以返回一个系列的值。

`dart:async`包提供了一个`StreamController`类。流控制器管理的两个对象流和槽（sink）。sink和流相对应，流提供提供数据，sink接受输入值。

总结一下，BLoC用来处理逻辑，sink接受输入，流输出。

