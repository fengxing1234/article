---
title: Flutter面试题
date: 2020-11-06 10:01:20
tags:
---

# Flutter面试题

#### 为什么说Flutter性能好？说下和其他跨平台的本质区别！

答：与用于构建移动应用程序的其他大多数框架不同，Flutter是重写了一整套包括底层渲染逻辑和上层开发语言的完整解决方案。这样不仅可以保证视图渲染在Android和iOS上的高度一致性，在代码执行效率和渲染性能上也可以媲美原生App的体验。这，就是Flutter和其他跨平台方案的本质区别。



#### 问： main()和runApp()函数在flutter的作用分别是什么？有什么关系吗？

答：

- main函数是类似于java语言的程序运行入口函数
- runApp函数是渲染根widget树的函数
- 一般情况下runApp函数会在main函数里执行



#### 问：**Widgets、RenderObjects 和 Elements的关系**

答：

- **Widget** ：仅用于存储渲染所需要的信息。
- **RenderObject** ：负责管理布局、绘制等操作。
- **Element** ：才是这颗巨大的控件树上的实体。

Widget会被inflate（填充）到Element，并由Element管理底层渲染树。Widget并不会直接管理状态及渲染,而是通过State这个对象来管理状态。Flutter创建Element的可见树，相对于Widget来说，是可变的，通常界面开发中，我们不用直接操作Element,而是由框架层实现内部逻辑。就如一个UI视图树中，可能包含有多个TextWidget(Widget被使用多次)，但是放在内部视图树的视角，这些TextWidget都是填充到一个个独立的Element中。Element会持有renderObject和widget的实例。记住，Widget 只是一个配置，RenderObject 负责管理布局、绘制等操作。

在第一次创建 Widget 的时候，会对应创建一个 Element， 然后将该元素插入树中。如果之后 Widget 发生了变化，则将其与旧的 Widget 进行比较，并且相应地更新 Element。重要的是，Element 不会被重建，只是更新而已。

#### 问：StatelessWidget和StatefulWidget区别

StatelessWidget用于不需要维护状态的场景.，StatefulWidget用于状态会发生变化的场景，它们都继承自Widget。

StatelessWidget，重写了createElement()方法，StatelessElement 间接继承自Element类,

```
abstract class StatelessWidget extends Widget {
StatelessElement createElement() => StatelessElement(this);
}
```

StatefulWidget也是重写了createElement()方法，不同的是返回的Element 对象并不相同；另StatefulWidget类中添加了一个新的接口createState()。

```
abstract class StatefulWidget extends Widget {
  const StatefulWidget({ Key key }) : super(key: key);

  @override
  StatefulElement createElement() => new StatefulElement(this);

  @protected
  State createState();
}
```

#### 问：State生命周期

答：一个StatefulWidget类会对应一个State类，State表示与其对应的StatefulWidget要维护的状态，

- initState()

  界面初始化状态时调用

- didChangeDependencies()

  当state状态对象发生变化时调用（典型的场景是当系统语言Locale或应用主题改变时，Flutter framework会通知widget调用此回调。）

- build()

  主要是用于构建Widget子树,调用时机如下：
  	在调用initState()之后。
  	在调用didUpdateWidget()之后。
  	在调用setState()之后。
  	在调用didChangeDependencies()之后。
  	在State对象从树中一个位置移除后（会调用deactivate）又重新插入到树的其它位置之后。

- reassemble()

  主要用于调试，热重载时调用，release环境下不会调用

- didUpdateWidget()

  用于更新widget ，Widget.canUpdate返回true则会调用此回调

- deactivate()

  从widget树中移除State对象t时调用（位置交换）

- dispose()

  从widget树中移除State对象，并不再插入此State对象时调用（一般用于释放资源）



#### 问：Flutter中的Widget、State、Context 的核心概念？是为了解决什么问题？

答

- **Widget**: 在Flutter中，几乎所有东西都是Widget。将一个Widget想象为一个可视化的组件（或与应用可视化方面交互的组件），当你需要构建与布局直接或间接相关的任何内容时，你正在使用Widget。
- **Widget树**: Widget以树结构进行组织。包含其他Widget的widget被称为父Widget(或widget容器)。包含在父widget中的widget被称为子Widget。
- **Context**: 仅仅是已创建的所有Widget树结构中的某个Widget的位置引用。简而言之，将context作为widget树的一部分，其中context所对应的widget被添加到此树中。一个context只从属于一个widget，它和widget一样是链接在一起的，并且会形成一个context树。
- **State**: 定义了StatefulWidget实例的行为，它包含了用于”**交互/干预**“Widget信息的行为和布局。应用于State的任何更改都会强制重建Widget。

这些状态的引入，主要是为了解决多个部件之间的交互和部件自身状态的维护。



#### 问：Widget 唯一标识Key有那几种

答：在flutter中，每个widget都是被唯一标识的。这个唯一标识在build或rendering阶段由框架定义。该标识对应于可选的Key参数，如果省略，Flutter将会自动生成一个。

在flutter中，主要有4种类型的Key：

- GlobalKey（确保生成的Key在整个应用中唯一，是很昂贵的，允许element在树周围移动或变更父节点而不会丢失状态）
- LocalKey
- UniqueKey
- ObjectKey。



#### 问：什么是Navigator? MaterialApp做了什么？

答：Navigator是在Flutter中负责管理维护页面堆栈的导航器。

MaterialApp在需要的时候，会自动为我们创建Navigator。

Navigator.of(context)，会使用context来向上遍历Element树，找到MaterialApp提供的_NavigatorState再调用其push/pop方法完成导航操作。



#### 问：**什么是pubspec.yaml文件？**

这是项目的配置文件, 在处理Flutter项目期间会用很多。它允许你如何运行应用程序。它还允许我们为应用设置约束。该文件包含：

- 项目常规设置, 例如项目的名称, 描述和版本。
- 项目依赖项。
- 项目资产(例如图像, 音频等)。



#### 问：**什么是状态管理，你了解哪些状态管理框架？**？

答：Flutter中的状态和前端React中的状态概念是一致的。React框架的核心思想是组件化，应用由组件搭建而成，组件最重要的概念就是状态，状态是一个组件的UI数据模型，是组件渲染时的数据依据。

Flutter的状态可以分为全局状态和局部状态两种。常用的状态管理有ScopedModel、BLoC、Redux / FishRedux和Provider。



#### 问：**Flutter 是如何与原生Android、iOS进行通信的？**

答：Flutter 通过 PlatformChannel 与原生进行交互，其中 PlatformChannel 分为三种：

- **BasicMessageChannel** ：用于传递字符串和半结构化的信息。
- **MethodChannel** ：用于传递方法调用（method invocation）。
- **EventChannel** : 用于数据流（event streams）的通信。



#### 问：**介绍下FFlutter的FrameWork层和Engine层，以及它们的作用**？

答：Flutter的FrameWork层是用Drat编写的框架（SDK），它实现了一套基础库，包含Material（Android风格UI）和Cupertino（iOS风格）的UI界面，下面是通用的Widgets（组件），之后是一些动画、绘制、渲染、手势库等。这个纯 Dart实现的 SDK被封装为了一个叫作 dart:ui的 Dart库。我们在使用 Flutter写 App的时候，直接导入这个库即可使用组件等功能。

Flutter的Engine层是Skia 2D的绘图引擎库，其前身是一个向量绘图软件，Chrome和 Android均采用 Skia作为绘图引擎。Skia提供了非常友好的 API，并且在图形转换、文字渲染、位图渲染方面都提供了友好、高效的表现。Skia是跨平台的，所以可以被嵌入到 Flutter的 iOS SDK中，而不用去研究 iOS闭源的 Core Graphics / Core Animation。Android自带了 Skia，所以 Flutter Android SDK要比 iOS SDK小很多。



#### 问：假如蓝湖设计图给你一张轮播图，宽度是 x 高度是 y（x px * y px），左右间隔是t，如何使用屏幕算法适配全机型屏幕宽和高？

答：答案不唯一。

```
宽度：整宽 - t * 2（左右的）。
高度：(整宽 - t * 2 ) * y / x。
```



#### 问：假如蓝湖设计图给你一个未知数据数量有规则的列表视图，要求一行显示5个，每个间隔为10（含上下），最外边距`margin`左右都为20，高度为50，多出的数据继续往下排并向左对齐，适配任何机型，你会使用什么组件？怎么做适配？

答：

- 使用组件：Wrap
- spacing和runSpacing都设置为10间隔，
- 然后Item的高度设置为50，宽度算法为：( 整宽 - （外边的margin + 内边的space） ) / 5