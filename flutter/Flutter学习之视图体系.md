---
title: Flutter学习之视图体系
date: 2020-06-14 20:38:51
tags:
	- flutter
categories: 
	- flutter
---

#  Flutter学习之视图体系

## 一、前言

经过之前的学习，可以知道**Flutter**是一种全新的响应式跨平台的移动开发框架，越来越多的开发者参与学习或者研究中，在**iOS**和**Android**平台上能够用一套代码构建出性能比较高的应用程序。我刚开始接触**Flutter**[Flutter中文网](https://flutterchina.club/technical-overview/)看到这么一句话：`Widget`是**Flutter**应用程序用户界面的基本构建块。每个`Widget`都是用户界面一部分的不可变声明。与其他将视图、控制器、布局和其他属性分离的框架不同，**Flutter**具有一致的统一对象模型：Widget。在开发过程中也可以知道`Widget`可以被定义按钮(button)、样式(style)、填充(Padding)、布局(Row)、手势(GestureDetector)等，我刚开始以为这个`Widget`就是眼中所看到的视图，然而并不是这样的，下面慢慢讲述。

## 二、视图基础

### 1.Widget

在[Flutter官方网站](https://flutter.dev/docs/development/ui/widgets-intro)介绍`Widgets`开篇有这么一段话：

> Flutter widgets are built using a modern react-style framework, which takes inspiration from React. The central idea is that you build your UI out of widgets. Widgets describe what their view should look like given their current configuration and state. When a widget’s state changes, the widget rebuilds its description, which the framework diffs against the previous description in order to determine the minimal changes needed in the underlying render tree to transition from one state to the next.

这段话的意思是：`Flutter widgets`是采取React思想使用响应式框架构建的。核心思想就是使用`widgets`构建出UI(界面)。`Widgets`根据其当前配置和状态描述了它们的视图。当某个`widget`的状态发生更改时，`widget`会重新构建所描述的视图，framework会根据前面所描述的视图(状态没改变时)进行区分，以确定底层呈现树从一个状态转换到下一个状态所需的最小更改步骤。

在[Flutter开发者文档](https://docs.flutter.io/flutter/widgets/Widget-class.html)对`Widget`的定义如下：

> Describes the configuration for an Element.
>
> Widgets are the central class hierarchy in the Flutter framework. A widget is an immutable description of part of a user interface. Widgets can be inflated into elements, which manage the underlying render tree.

意思是：`widget`为`element`(下面再描述)提供配置信息，这里可以知道`widget`和`element`存在某种联系。`Widgets`在Flutter framework是中心类层次结构，`widget`是不可变的对象并且是界面的一部分，`widget`会被渲染在`elements`上，并(elelments)管理底层渲染树(render tree)，这里可以得到一个信息：`widget`在渲染的时候会最终转换成`element`。继续往下看：

> Widgets themselves have no mutable state (all their fields must be final). If you wish to associate mutable state with a widget, consider using a StatefulWidget, which creates a State object (via StatefulWidget.createState) whenever it is inflated into an element and incorporated into the tree.

意思是：`Wigets`本身是没有可变的状态(其所有的字段必须是final)。如果你想吧可变状态和一个`widget`关联起来，可以使用`StatefulWidget`，`StatefulWidget`通过使用`StatefulWidget.createState`方法创建`State`对象，并且扩充到`element`和合并到树中。那么这段可以得出的信息是：`widget`并不会直接渲染和管理状态，管理状态是交给`State`对象负责。继续往下一段看：

> A given widget can be included in the tree zero or more times. In particular a given widget can be placed in the tree multiple times. Each time a widget is placed in the tree, it is inflated into an Element, which means a widget that is incorporated into the tree multiple times will be inflated multiple times.

意思是：给定的`widget`可以**零次或者多次**被包含在树中，一个给定的`widget`可以多次放置在树中，每次将一个`widget`放入树中，他都会被扩充到一个`Element`，这就意味着多次并入树中的`widget`将会多次扩充到对应的`Element`。这段可以这么理解：在一个界面中，有多个`Text`被挂载在视图树上，这些`Text`的`widget`会被填充进自己独立的`Element`中，就算`widget`被重复使用，还是会创建多个不同的`element`对象。继续往下看：

> The key property controls how one widget replaces another widget in the tree. If the runtimeType and key properties of the two widgets are operator==, respectively, then the new widget replaces the old widget by updating the underlying element (i.e., by calling Element.update with the new widget). Otherwise, the old element is removed from the tree, the new widget is inflated into an element, and the new element is inserted into the tree.

每一个`widget`都有自己的唯一的**key**，这里也很容易理解，就是借助**key**作为唯一标识符。这段话的意思是：**key**这个属性控制一个`widget`如何替换树中的另一个`widget`。如果两个`widget`的**runtimeType**和**key**属性相等==，则新的`widget`通过更新`Element`(通过新的`widget`来来调用**Element.update**)来替换旧的`widget`。否则，如果两个`widget`的**runtimeType**和**key**属性不相等，则旧的`Element`将从树中被移除，新的`widget`将被扩充到一个新的`Element`中，这个新的`Element`将被插入树中。这里可以得出：如果涉及到`widget`的移动或者删除操作前，会根据`widget`的**runtime**和**key**进行对比。

综上所述：

- widget向Element提供配置信息(数据)，界面构造出来的widget树其实只是一颗配置信息树，为了构造出Element树。
- Element和widget有对应关系，因为，element是通过widget来生成的。
- 同一个widget可以创建多个element，**也就是一个widget对象可以对应多个element对象**。

下面初步看看widget源码：

```
@immutable
abstract class Widget extends DiagnosticableTree {
  /// Initializes [key] for subclasses.
  const Widget({ this.key });

  //省略注释
  final Key key;

  //构建出element
  @protected
  Element createElement();

  //简短文字描述这个widget
  @override
  String toStringShort() {
    return key == null ? '$runtimeType' : '$runtimeType-$key';
  }
  
  //根据字面意思 应该是调试诊断树的信息
  @override
  void debugFillProperties(DiagnosticPropertiesBuilder properties) {
    super.debugFillProperties(properties);
    properties.defaultDiagnosticsTreeStyle = DiagnosticsTreeStyle.dense;
  }


  //静态方法，跟上一段解释一样，就是是否用新的widget对象去更新旧UI渲染树的配置
  //如果oldWidget和newWidget的runtimeType和key同时相等就会用newWidget对象去更新对应element信息
  static bool canUpdate(Widget oldWidget, Widget newWidget) {
    return oldWidget.runtimeType == newWidget.runtimeType
        && oldWidget.key == newWidget.key;
  }
}
复制代码
```

还要注意:`widget`是抽象类，在平时，一般继续`StatelessWidget`和`StatefulWidget`，而这两个类其实也是继承`Widget`，这两个类肯定会实现这个`createElement`方法，简单看一下：

```
StatelessWidget
abstract class StatelessWidget extends Widget {
  /// Initializes [key] for subclasses.
  const StatelessWidget({ Key key }) : super(key: key);
  ....
  @override
  StatelessElement createElement() => StatelessElement(this);
  ....
}

StatefulWidget
abstract class StatefulWidget extends Widget {
  /// Initializes [key] for subclasses.
  const StatefulWidget({ Key key }) : super(key: key);
  .....
  @override
  StatefulElement createElement() => StatefulElement(this);
  .....
    
}
复制代码
```

确实可以看出，都会创建`Element`，只是`StatelessWidget`和`StatefulWidget`所创建的`Element`类型不一样，这里就先不深入了。上面可以知道`widget`和`element`存在对应的关系，那下面看看`element`。

### 2.Element

看看[官方开发者文档](https://api.flutter.dev/flutter/widgets/Element-class.html)中开篇看到：

> An instantiation of a Widget at a particular location in the tree.

意思是：`element`是树中特定位置的`widget`实例。这里描述的很明显，也就是`Widget`是总监，部署技术规划，而`element`就是员工，真正干活。继续往下阅读：

> Widgets describe how to configure a subtree but the same widget can be used to configure multiple subtrees simultaneously because widgets are immutable. An Element represents the use of a widget to configure a specific location in the tree. Over time, the widget associated with a given element can change, for example, if the parent widget rebuilds and creates a new widget for this location.

意思是：`widget`描述如何配置子树，由于`widgets`是不可变的，所以可以用相同的`widget`来同时配置多个子树，`Element`表示`widget`配置树中的特定位置的实例，随着时间的推移，和给定的`Element`关联的`Widget`可能会随时变化，例如，如果父`widget`重建并为此位置创建新的`widget`。

> Elements form a tree. Most elements have a unique child, but some widgets (e.g., subclasses of RenderObjectElement) can have multiple children.

`Elements`构成一棵树，大多数`elements`都会有**唯一**的孩子，但是一些`widgets`(如RenderObjectElement)可以有多个孩子。

> Elements have the following lifecycle:
>
> - The framework creates an element by calling Widget.createElement on the widget that will be used as the element's initial configuration.
> - The framework calls mount to add the newly created element to the tree at a given slot in a given parent. The mount method is responsible for inflating any child widgets and calling attachRenderObject as necessary to attach any associated render objects to the render tree.
> - At this point, the element is considered "active" and might appear on screen.
> - At some point, the parent might decide to change the widget used to configure this element, for example because the parent rebuilt with new state. When this happens, the framework will call update with the new widget. The new widget will always have the same runtimeType and key as old widget. If the parent wishes to change the runtimeType or key of the widget at this location in the tree, can do so by unmounting this element and inflating the new widget at this location.
> - At some point, an ancestor might decide to remove this element (or an intermediate ancestor) from the tree, which the ancestor does by calling deactivateChild on itself. Deactivating the intermediate ancestor will remove that element's render object from the render tree and add this element to the owner's list of inactive elements, causing the framework to call deactivate on this element.
> - At this point, the element is considered "inactive" and will not appear on screen. An element can remain in the inactive state only until the end of the current animation frame. At the end of the animation frame, any elements that are still inactive will be unmounted. *If the element gets reincorporated into the tree (e.g., because it or one of its ancestors has a global key that is reused), the framework will remove the element from the owner's list of inactive elements, call activate on the element, and reattach the element's render object to the render tree. (At this point, the element is again considered "active" and might appear on screen.)
> - If the element does not get reincorporated into the tree by the end of the current animation frame, the framework will call unmount on the element. At this point, the element is considered "defunct" and will not be incorporated into the tree in the future.
> - At this point, the element is considered "defunct" and will not be incorporated into the tree in the future.

意思如下： Element具有以下生命周期：

- framework通过调用即将用来作`element`的初始化配置信息的`Widget`的**Widget.createElement**方法来创建一个`element`。
- framework通过调用mount方法以将新创建的Element添加到给定父级中给定槽点的树上。 mount方法负责将任何子`Widget`扩充到`Widget`并根据需要调用`attachRenderObject`，以将任何关联的渲染对象附加到渲染树上。
- 此时，`element`被视为`激活`，可能出现在屏幕上。
- 在某些情况下，父可能会更改用于配置此`Element`的Widget，例如因为父重新创建了新状态。发生这种情况时，framework将调用新的`Widget`的update方法。新`Widget`将始终具有与旧`Widget`相同的**runtimeType**和**key**属性。如果父希望在树中的此位置更改`Widget`的**runtimeType**或**key**，可以通过unmounting(卸载)此Element并在此位置扩充新`Widget`来实现。
- 在某些时候，祖先(Element)可能会决定从树中移除该`element`（或中间祖先），祖先自己通过调用**deactivateChild**来完成该操作。停用中间祖先将从渲染树中移除该`element`的渲染对象，并将此`element`添加到所有者属性中的非活动元素列表中，从而framework调用**deactivate**方法作用在此`element`上。
- 此时，该`element`被视为“无效”，不会出现在屏幕上。一个`element`直到动画帧结束前都可以保存“非活动”状态。动画帧结束时，将卸载仍处于非活动状态的所有`element`。
- 如果`element`被重写组合到树中(例如，因为它或其祖先之一有一个全局建(global key)被重用)，framework将从所有者的非活动`elements`列表中移除该`element`，并调用该`element`的**activate方法**，并重新附加到`element`的渲染对象到渲染树上。(此时，该元素再次被视为“活动”并可能出现在屏幕上)
- 如果`element`在当前动画帧的末尾(最后一帧)没有被重新组合到树中，那么framework将会调用该元素的**unmount方法**。

这里可以知道`element`的生命周期。并且平时开发没有接触到`Element`，都是直接操控`widget`，也就是说Flutter已经帮我们对`widget`的操作映射到`element`上，我这里想象到的有点事降低开发复杂。下面结合一个例子(绘制Text)，看看`element`是不是最后渲染出来的view：

```
new Text("hello flutter");
复制代码
```

下面看下`new Text`的源码：

```
  ...
  @override
  Widget build(BuildContext context) {
    final DefaultTextStyle defaultTextStyle = DefaultTextStyle.of(context);
    TextStyle effectiveTextStyle = style;
    if (style == null || style.inherit)
      effectiveTextStyle = defaultTextStyle.style.merge(style);
    if (MediaQuery.boldTextOverride(context))
      effectiveTextStyle = effectiveTextStyle.merge(const TextStyle(fontWeight: FontWeight.bold));
    Widget result = RichText( ---->Text原来是通过RichText这个widget来构建树
      textAlign: textAlign ?? defaultTextStyle.textAlign ?? TextAlign.start,
      textDirection: textDirection, // RichText uses Directionality.of to obtain a default if this is null.
      locale: locale, // RichText uses Localizations.localeOf to obtain a default if this is null
      softWrap: softWrap ?? defaultTextStyle.softWrap,
      overflow: overflow ?? defaultTextStyle.overflow,
      textScaleFactor: textScaleFactor ?? MediaQuery.textScaleFactorOf(context),
      maxLines: maxLines ?? defaultTextStyle.maxLines,
      strutStyle: strutStyle,
      text: TextSpan(
        style: effectiveTextStyle,
        text: data,
        children: textSpan != null ? <TextSpan>[textSpan] : null,
      ),
    );
    ....
复制代码
```

继续往`RichText`里面看：

```
  ....
  @override
  RenderParagraph createRenderObject(BuildContext context) {
    assert(textDirection != null || debugCheckHasDirectionality(context));
    //返回RenderParagraph widget 
    return RenderParagraph(text,
      textAlign: textAlign,
      textDirection: textDirection ?? Directionality.of(context),
      softWrap: softWrap,
      overflow: overflow,
      textScaleFactor: textScaleFactor,
      maxLines: maxLines,
      strutStyle: strutStyle,
      locale: locale ?? Localizations.localeOf(context, nullOk: true),
    );
  }
  ....
复制代码
```

发现最终它会返回`RenderParagraph`widget，继续往里看：

```
class RichText extends LeafRenderObjectWidget {
}
复制代码
```

可以发现`RichText`继承`LeafRenderObjectWidget`，继续往下看：

```
//用于配置RenderObject子类的RenderObjectWidgets的超类，没有孩子，也就是没有字节点(child)
abstract class LeafRenderObjectWidget extends RenderObjectWidget {
  const LeafRenderObjectWidget({ Key key }) : super(key: key);
  
  //构建出类型是LeafRenderObjectElement
  @override
  LeafRenderObjectElement createElement() => LeafRenderObjectElement(this);
}
复制代码
```

可以看到`LeafRenderObjectWidget`是抽象类，并且继承了`RenderObjectWidget`，继续往下看：

```
/// RenderObjectWidgets provide the configuration for [RenderObjectElement]s,
/// which wrap [RenderObject]s, which provide the actual rendering of the
/// application.
//上面注释是：RenderObjectWidgets向[RenderObjectElement]提供了配置信息，包装了[RenderObject]，在应用程序了提供了实际的渲染
abstract class RenderObjectWidget extends Widget {

  const RenderObjectWidget({ Key key }) : super(key: key);
  
  //创建element
  @override
  RenderObjectElement createElement();
  //创建RenderObject
  @protected
  RenderObject createRenderObject(BuildContext context);

  //更新RenderObject
  @protected
  void updateRenderObject(BuildContext context, covariant RenderObject renderObject) { }
  
  //卸载RenderObject
  @protected
  void didUnmountRenderObject(covariant RenderObject renderObject) { }
}
复制代码
```

由注释可以知道`RenderObject`才是绘制UI背后的真正对象，那下面继续简单跟踪：

### 3.RenderObjectWidget

先看看`RenderObjectWidget`继承关系：



![RenderObjectWidget继承关系](https://user-gold-cdn.xitu.io/2019/3/28/169c2dc43541e51a?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

看看文档的一一介绍：



#### 3.1.SingleChildRenderObjectWidget



![SingleChildRenderObjectWidget](https://user-gold-cdn.xitu.io/2019/3/28/169c2e6861c9534a?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

官方文档写的很清楚： 是`RenderObjectWidgets`的超类，用于配置有单个孩子的`RenderObject`子类(为子类提供存储，实际不提供更新逻辑)，并列了具体的实际`widget`,如常用的Align、ClipRect、DecoratedBox都是属于这类。



#### 3.2.MultiChildRenderObjectWidget



![MultiChildRenderObjectWidget](https://user-gold-cdn.xitu.io/2019/3/28/169c2eff26313931?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

同样也是`RenderObjectWidgets`的超类，用于配置有单个**children**(也就是多个child)的`RenderObject`子类，如Flex、Flow、Stack都属于这类。



#### 3.3.LeafRenderObjectWidget



![LeafRenderObjectWidget](https://user-gold-cdn.xitu.io/2019/3/28/169c2f8f6f8577bd?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

同样也是`RenderObjectWidgets`的超类，用于配置没有孩子的`RenderObject`子类，如：RichText(平时的Text)、RawImage、Texture等。 这里总结一下：



- `SingleChildRenderObjectWidget`用作只有一个child的`widget`。
- `MultiChildRenderObjectWidget`用作有children(多个孩子)的`widget`。
- `LeafRenderObjectWidget`用作没有child的`widget`。
- `RenderObjectWidget`定义了创建，更新，删除RenderObject的方法，子类必须实现，`RenderObject`是最终布局、UI渲染的实际对象。

那么假如我现在界面上，假如布局如下：



![例子布局](https://user-gold-cdn.xitu.io/2019/3/28/169c3322c6d67e23?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

那么渲染流程如下：





![渲染转换](https://user-gold-cdn.xitu.io/2019/3/28/169c346cd1a92ea9?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

这时候会想：为什么要加中间层`Element`呢，不直接由`Widget`直接转换成`RendObject`不是直接更好么？其实并不是这样的。首先知道Flutter是响应式框架，在某一个时刻，可能会受到不同的输入流影响，中间层`Element`对这一时刻的事件做了汇总，最后将需要修改的部分同步到`RendObject`tree上，也就是：



- 尽可能的降低`RenderObject`tree的更改，提高页面渲染效率。
- 没有直接操作UI，通过数据驱动视图，代码更容易理解和简洁。

上面得出UI树是由一个个`element`节点组成，但是最终的渲染是通过`RenderObject`来完成，一个`widget`从创建到显示到界面的流程是：widget生成element，然后通过**createRenderObject**方法创建对应的`RenderObject`关联到`Element.renderObject`上，最后通过`RenderObject`来完成绘制。

## 三、启动到显示

上面知道，`widget`并不是真正显示的对象，知道了一个`widget`是怎样渲染在屏幕上显示，那下面就从main()方法入口，看看一个app从点击启动到运行的流程：

### 1.WidgetsFlutterBinding.ensureInitialized

```
//app入口
void main() => runApp(MyApp());
复制代码
```

app入口函数就是调用了**runApp**方法，看看**runApp**方法里面做了什么：

```
void runApp(Widget app) {
  WidgetsFlutterBinding.ensureInitialized()
    ..attachRootWidget(app)
    ..scheduleWarmUpFrame();
}
复制代码
```

首先参数(Widget app)是一个`widget`，下面一行一行分析，进入**WidgetsFlutterBinding.ensureInitialized()**方法：

```
/// A concrete binding for applications based on the Widgets framework.
/// This is the glue that binds the framework to the Flutter engine.
//意思：基于widget framework的应用程序的具体绑定
//这是将framework widget和Flutter engine绑定的桥梁
class WidgetsFlutterBinding extends BindingBase with GestureBinding, ServicesBinding, SchedulerBinding, PaintingBinding, SemanticsBinding, RendererBinding, WidgetsBinding {

  /// Returns an instance of the [WidgetsBinding], creating and
  /// initializing it if necessary. If one is created, it will be a
  /// [WidgetsFlutterBinding]. If one was previously initialized, then
  /// it will at least implement [WidgetsBinding].
  ///
  /// You only need to call this method if you need the binding to be
  /// initialized before calling [runApp].
  ///
  /// In the `flutter_test` framework, [testWidgets] initializes the
  /// binding instance to a [TestWidgetsFlutterBinding], not a
  /// [WidgetsFlutterBinding].
  //上面注释意思是：返回[WidgetsBinding]的具体实例，必须创建和初始化，如果已//经创建了，那么它就是一个[WidgetsFlutterBinding]，如果已经初始化了，那么//至少要实现[WidgetsBinding]
  //如果你只想调用这个方法，那么你需要在调用runApp之前绑定并且初始化
  static WidgetsBinding ensureInitialized() {
    if (WidgetsBinding.instance == null)
      WidgetsFlutterBinding();
    return WidgetsBinding.instance;
  }
}
复制代码
```

看到这个`WidgetsFlutterBinding`混入(with)很多的`Binding`，下面先看父类`BindingBase`:

```
abstract class BindingBase {
  BindingBase() {
    developer.Timeline.startSync('Framework initialization');

    assert(!_debugInitialized);
    initInstances();
    assert(_debugInitialized);

    assert(!_debugServiceExtensionsRegistered);
    initServiceExtensions();
    assert(_debugServiceExtensionsRegistered);

    developer.postEvent('Flutter.FrameworkInitialization', <String, String>{});

    developer.Timeline.finishSync();
  }

  static bool _debugInitialized = false;
  static bool _debugServiceExtensionsRegistered = false;
  ui.Window get window => ui.window;//获取window实例
  @protected
  @mustCallSuper
  void initInstances() {
    assert(!_debugInitialized);
    assert(() { _debugInitialized = true; return true; }());
  }
}
复制代码
```

看到有句代码**ui.Window get window => ui.window;**，在Android中所有的视图都是通过`window`来呈现的，那Flutter中也有`window`，那看看`window`在Flutter中的作用看看官方对它的定义：



![window定义](https://user-gold-cdn.xitu.io/2019/3/28/169c38d8bd7b5dce?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

意思是：链接宿主操作系统的接口，也就是Flutter framework 链接宿主操作系统的接口。系统中有一个Window实例，可以从window属性来获取，看看源码：



```
class Window {
  Window._();

  //返回DPI，DPI是每英寸的像素点数，是设备屏幕的固件属性
  //获取可能不准确
  double get devicePixelRatio => _devicePixelRatio;
  double _devicePixelRatio = 1.0;

  //绘制UI的区域大小
  Size get physicalSize => _physicalSize;
  Size _physicalSize = Size.zero;

  //获取矩形的物理像素
  WindowPadding get viewInsets => _viewInsets;
  WindowPadding _viewInsets = WindowPadding.zero;

  //获取内边距
  WindowPadding get padding => _padding;
  WindowPadding _padding = WindowPadding.zero;

  //当绘制区域改变时触发
  VoidCallback get onMetricsChanged => _onMetricsChanged;
  VoidCallback _onMetricsChanged;
  Zone _onMetricsChangedZone;
  set onMetricsChanged(VoidCallback callback) {
    _onMetricsChanged = callback;
    _onMetricsChangedZone = Zone.current;
  }

  //当系统语言发生变化时触发回调
  Locale get locale {
    if (_locales != null && _locales.isNotEmpty) {
      return _locales.first;
    }
    return null;
  }

  //获取系统语言
  List<Locale> get locales => _locales;
  List<Locale> _locales;

  //当Local发生改变时触发回调
  VoidCallback get onLocaleChanged => _onLocaleChanged;
  VoidCallback _onLocaleChanged;
  Zone _onLocaleChangedZone;
  set onLocaleChanged(VoidCallback callback) {
    _onLocaleChanged = callback;
    _onLocaleChangedZone = Zone.current;
  }

  //当系统字体大小改变时触发回调
  VoidCallback get onTextScaleFactorChanged => _onTextScaleFactorChanged;
  VoidCallback _onTextScaleFactorChanged;
  Zone _onTextScaleFactorChangedZone;
  set onTextScaleFactorChanged(VoidCallback callback) {
    _onTextScaleFactorChanged = callback;
    _onTextScaleFactorChangedZone = Zone.current;
  }

  //屏幕亮度改变时触发回调
  VoidCallback get onPlatformBrightnessChanged => _onPlatformBrightnessChanged;
  VoidCallback _onPlatformBrightnessChanged;
  Zone _onPlatformBrightnessChangedZone;
  set onPlatformBrightnessChanged(VoidCallback callback) {
    _onPlatformBrightnessChanged = callback;
    _onPlatformBrightnessChangedZone = Zone.current;
  }

  //屏幕刷新时会回调
  FrameCallback get onBeginFrame => _onBeginFrame;
  FrameCallback _onBeginFrame;
  Zone _onBeginFrameZone;
  set onBeginFrame(FrameCallback callback) {
    _onBeginFrame = callback;
    _onBeginFrameZone = Zone.current;
  }

  //绘制屏幕时回调
  VoidCallback get onDrawFrame => _onDrawFrame;
  VoidCallback _onDrawFrame;
  Zone _onDrawFrameZone;
  set onDrawFrame(VoidCallback callback) {
    _onDrawFrame = callback;
    _onDrawFrameZone = Zone.current;
  }

  //点击或者指针事件触发回调
  PointerDataPacketCallback get onPointerDataPacket => _onPointerDataPacket;
  PointerDataPacketCallback _onPointerDataPacket;
  Zone _onPointerDataPacketZone;
  set onPointerDataPacket(PointerDataPacketCallback callback) {
    _onPointerDataPacket = callback;
    _onPointerDataPacketZone = Zone.current;
  }

  //获取请求的默认路由
  String get defaultRouteName => _defaultRouteName();
  String _defaultRouteName() native 'Window_defaultRouteName';

  //该方法被调用后，onBeginFrame和onDrawFrame将紧连着会在合适时机调用
  void scheduleFrame() native 'Window_scheduleFrame';

  //更新应用在GPU上的渲染，这方法会直接调用Flutter engine的Window_render方法
  void render(Scene scene) native 'Window_render';

  //窗口的语义内容是否改变
  bool get semanticsEnabled => _semanticsEnabled;
  bool _semanticsEnabled = false;

  //当窗口语言发生改变时回调 
  VoidCallback get onSemanticsEnabledChanged => _onSemanticsEnabledChanged;
  VoidCallback _onSemanticsEnabledChanged;
  Zone _onSemanticsEnabledChangedZone;
  set onSemanticsEnabledChanged(VoidCallback callback) {
    _onSemanticsEnabledChanged = callback;
    _onSemanticsEnabledChangedZone = Zone.current;
  }

  //当用户表达写的动作时回调
  SemanticsActionCallback get onSemanticsAction => _onSemanticsAction;
  SemanticsActionCallback _onSemanticsAction;
  Zone _onSemanticsActionZone;
  set onSemanticsAction(SemanticsActionCallback callback) {
    _onSemanticsAction = callback;
    _onSemanticsActionZone = Zone.current;
  }

  //启用其他辅助功能回调
  VoidCallback get onAccessibilityFeaturesChanged => _onAccessibilityFeaturesChanged;
  VoidCallback _onAccessibilityFeaturesChanged;
  Zone _onAccessibilityFlagsChangedZone;
  set onAccessibilityFeaturesChanged(VoidCallback callback) {
    _onAccessibilityFeaturesChanged = callback;
    _onAccessibilityFlagsChangedZone = Zone.current;
  }

  //更新此窗口的语义数据
  void updateSemantics(SemanticsUpdate update) native 'Window_updateSemantics';

  //设置Isolate调试名称
  void setIsolateDebugName(String name) native 'Window_setIsolateDebugName';

  //向特定平台发送消息
  void sendPlatformMessage(String name,
                           ByteData data,
                           PlatformMessageResponseCallback callback) {
    final String error =
        _sendPlatformMessage(name, _zonedPlatformMessageResponseCallback(callback), data);
    if (error != null)
      throw new Exception(error);
  }
  String _sendPlatformMessage(String name,
                              PlatformMessageResponseCallback callback,
                              ByteData data) native 'Window_sendPlatformMessage';

  //获取平台消息的回调
  PlatformMessageCallback get onPlatformMessage => _onPlatformMessage;
  PlatformMessageCallback _onPlatformMessage;
  Zone _onPlatformMessageZone;
  set onPlatformMessage(PlatformMessageCallback callback) {
    _onPlatformMessage = callback;
    _onPlatformMessageZone = Zone.current;
  }

  //由_dispatchPlatformMessage调用
  void _respondToPlatformMessage(int responseId, ByteData data)
      native 'Window_respondToPlatformMessage';

  //平台信息响应回调
  static PlatformMessageResponseCallback _zonedPlatformMessageResponseCallback(PlatformMessageResponseCallback callback) {
    if (callback == null)
      return null;

    // Store the zone in which the callback is being registered.
    final Zone registrationZone = Zone.current;

    return (ByteData data) {
      registrationZone.runUnaryGuarded(callback, data);
    };
  }
}
复制代码
```

可以知道`window`包含当前设备系统的一些信息和回调，那么现在看看混入的各种`Binding`:

- GestureBinding：绑定手势子系统，提供`onPointerDataPacket`回调。
- ServicesBinding：绑定平台消息通道，提供`onPlatformMessage`回调。
- SchedulerBinding：绑定绘制调度子系统，提供`onBeginFrame`和`onDrawFrame`回调。
- PaintingBinding：绑定绘制库，处理图像缓存。
- SemanticsBinding：语义层和flutter engine的桥梁，对辅助功能的底层支持。
- RendererBinding：是渲染树和Flutter engine的桥梁，提供`onMetricsChanged`和`onTextScaleFactorChanged`回调。
- WidgetsBinding：是Flutter widget和engine的桥梁，提供`onLocaleChanged`，`onBuildSchedule`回调。

也就是`WidgetsFlutterBinding.ensureInitialized()`这行代码看名字是将`WidgetsFlutterBinding`实例初始化。其实并非那么简单，另外**获取一些系统基本信息和初始化监听`window`对象的一些事件，然后将这些事件按照上层Framework模型规则进行包装、抽象最后分发**。简而言之`WidgetsFlutterBinding`是`Flutter engine`和`Framework`的桥梁。

### 2.attachRootWidget(app)

第二行是`attachRootWidget(app)`，看名字意思是将根`RootWidget`挂载。点进去看:

```
  //如果这个widget有必要创建，就将它附加到[renderViewElement]
  void attachRootWidget(Widget rootWidget) {
    _renderViewElement = RenderObjectToWidgetAdapter<RenderBox>(
      container: renderView,
      debugShortDescription: '[root]',
      child: rootWidget,
    ).attachToRenderTree(buildOwner, renderViewElement);
  }
复制代码
```

`renderView`是UI渲染树的根节点：

```
  // The render tree that's attached to the output surface.
  //挂载在渲染树 rootNode根节点
  RenderView get renderView => _pipelineOwner.rootNode;
复制代码
```

`rootWidget`是外面所传进来的根`Widget`，这里不用管它，下面看`attachToRenderTree(buildOwner, renderViewElement)`这个方法，根据意思，这个方法应该构建`RenderTree`，这个方法需要两个参数，首先看`buildOwner`这个参数：

```
  /// The [BuildOwner] in charge of executing the build pipeline for the
  /// widget tree rooted at this binding.
  BuildOwner get buildOwner => _buildOwner;
  final BuildOwner _buildOwner = BuildOwner();
复制代码
```

看官网对它的定义：



![BuildOwner](https://user-gold-cdn.xitu.io/2019/3/28/169c4793b37168d4?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

意思是：是widget framework的管理类，用来跟踪哪些widget需要重建，并处理widget树的其他任务，例如管理树的非活动元素列表，**并在调试时在热重载期间在必要时触发“重组”命令**，下面看另外一个参数`renderViewElement`，代码注释如下：



```
  /// The [Element] that is at the root of the hierarchy (and which wraps the
  /// [RenderView] object at the root of the rendering hierarchy).
  ///
  /// This is initialized the first time [runApp] is called.
  Element get renderViewElement => _renderViewElement;
  Element _renderViewElement;

复制代码
```

`renderViewElement`是`renderView`对应的`Element`对象，因为`renderView`是树的根，所以`renderViewElement`位于层次结构的根部，那下面点击`attachToRenderTree`的源码看：

```
  /// Inflate this widget and actually set the resulting [RenderObject] as the
  /// child of [container].
  ///
  /// If `element` is null, this function will create a new element. Otherwise,
  /// the given element will have an update scheduled to switch to this widget.
  ///
  /// Used by [runApp] to bootstrap applications.
  RenderObjectToWidgetElement<T> attachToRenderTree(BuildOwner owner, [ RenderObjectToWidgetElement<T> element ]) {
    if (element == null) {
      owner.lockState(() {
        element = createElement();
        assert(element != null);
        element.assignOwner(owner);
      });
      owner.buildScope(element, () {
        element.mount(null, null);
      });
    } else {
      element._newWidget = this;
      element.markNeedsBuild();
    }
    return element;
  }
复制代码
```

跟着代码往下走如果根`element`没有创建，那么就调用`createElement`创建根`element`，`element`和`widget`进行关联。接着调用`element.assignOwner(owner)`，这个方法其实就是设置这个`element`的跟踪，最后调用`owner.buildScope`这个方法，这个方法是确定更新`widget`的范围。如果`element`已经创建了，将根`element`和关联的`widget`设为新的，并且重新构建这个`element`，为了后面的复用。

### 3.scheduleWarmUpFrame

runApp()方法最后一行执行`scheduleWarmUpFrame`方法：

```
  /// Schedule a frame to run as soon as possible, rather than waiting for
  /// the engine to request a frame in response to a system "Vsync" signal.
  ///
  /// This is used during application startup so that the first frame (which is
  /// likely to be quite expensive) gets a few extra milliseconds to run.
  ///
  /// Locks events dispatching until the scheduled frame has completed.
  ///
  /// If a frame has already been scheduled with [scheduleFrame] or
  /// [scheduleForcedFrame], this call may delay that frame.
  ///
  /// If any scheduled frame has already begun or if another
  /// [scheduleWarmUpFrame] was already called, this call will be ignored.
  ///
  /// Prefer [scheduleFrame] to update the display in normal operation.
  void scheduleWarmUpFrame() {
    if (_warmUpFrame || schedulerPhase != SchedulerPhase.idle)
      return;

    _warmUpFrame = true;
    Timeline.startSync('Warm-up frame');
    final bool hadScheduledFrame = _hasScheduledFrame;
    // We use timers here to ensure that microtasks flush in between.
    Timer.run(() {
      assert(_warmUpFrame);
      handleBeginFrame(null);--->1
    });
    Timer.run(() {
      assert(_warmUpFrame);
      handleDrawFrame(); ---->2
      // We call resetEpoch after this frame so that, in the hot reload case,
      // the very next frame pretends to have occurred immediately after this
      // warm-up frame. The warm-up frame's timestamp will typically be far in
      // the past (the time of the last real frame), so if we didn't reset the
      // epoch we would see a sudden jump from the old time in the warm-up frame
      // to the new time in the "real" frame. The biggest problem with this is
      // that implicit animations end up being triggered at the old time and
      // then skipping every frame and finishing in the new time.
      resetEpoch();
      _warmUpFrame = false;
      if (hadScheduledFrame)
        scheduleFrame();
    });

    // Lock events so touch events etc don't insert themselves until the
    // scheduled frame has finished.
    lockEvents(() async {
      await endOfFrame;
      Timeline.finishSync();
    });
  }
复制代码
```

首先这个方法在`SchedulerBinding`里，这两个方法主要执行了`handleBeginFrame`和`handleDrawFrame`方法：

#### 3.1.handleBeginFrame

```
  void handleBeginFrame(Duration rawTimeStamp) {
    ...
    try {
      // TRANSIENT FRAME CALLBACKS
      Timeline.startSync('Animate', arguments: timelineWhitelistArguments);
      _schedulerPhase = SchedulerPhase.transientCallbacks;
      final Map<int, _FrameCallbackEntry> callbacks = _transientCallbacks;
      _transientCallbacks = <int, _FrameCallbackEntry>{};
      callbacks.forEach((int id, _FrameCallbackEntry callbackEntry) {
        if (!_removedIds.contains(id))
          _invokeFrameCallback(callbackEntry.callback, _currentFrameTimeStamp, callbackEntry.debugStack);
      });
      _removedIds.clear();
    } finally {
      _schedulerPhase = SchedulerPhase.midFrameMicrotasks;
    }
  }
复制代码
```

可以看到主要是对`transientCallbacks`队列操作，这个集合主要是放一些临时回调，存放动画回调。

#### 3.2.handleDrawFrame

```
  void handleDrawFrame() {
    assert(_schedulerPhase == SchedulerPhase.midFrameMicrotasks);
    Timeline.finishSync(); // end the "Animate" phase
    try {
      // PERSISTENT FRAME CALLBACKS ----->
      _schedulerPhase = SchedulerPhase.persistentCallbacks;
      for (FrameCallback callback in _persistentCallbacks)
        _invokeFrameCallback(callback, _currentFrameTimeStamp);

      // POST-FRAME CALLBACKS  ------>
      _schedulerPhase = SchedulerPhase.postFrameCallbacks;
      final List<FrameCallback> localPostFrameCallbacks =
          List<FrameCallback>.from(_postFrameCallbacks);
      _postFrameCallbacks.clear();
      for (FrameCallback callback in localPostFrameCallbacks)
        _invokeFrameCallback(callback, _currentFrameTimeStamp);
    } finally {
      _schedulerPhase = SchedulerPhase.idle;
      Timeline.finishSync(); // end the Frame
      profile(() {
        _profileFrameStopwatch.stop();
        _profileFramePostEvent();
      });
      assert(() {
        if (debugPrintEndFrameBanner)
          debugPrint('▀' * _debugBanner.length);
        _debugBanner = null;
        return true;
      }());
      _currentFrameTimeStamp = null;
    }
  }
复制代码
```

可以看到执行`persistentCallbacks`队列，这个队列用于存放一些持久的回调，不能再此类回调中在请求新的绘制帧，持久回调一经注册则不能移除。接着执行`postFrameCallbacks`这个队列在每一`Frame`(一次绘制)结束时只会调用一次，调用后被系统移除。

也就是`scheduleWarmUpFrame`这个方法安排帧尽快执行，当一次帧绘制结束之前不会响应各种事件，这样保证绘制过程中不触发重绘。上面说过：

> RendererBinding：是渲染树和Flutter engine的桥梁，提供onMetricsChanged和onTextScaleFactorChanged回调

Flutter真正渲染和绘制是在这个绑定里：

### 4.渲染绘制

```
  void initInstances() {
    super.initInstances();
    _instance = this;//初始化
    _pipelineOwner = PipelineOwner(
      onNeedVisualUpdate: ensureVisualUpdate,
      onSemanticsOwnerCreated: _handleSemanticsOwnerCreated,
      onSemanticsOwnerDisposed: _handleSemanticsOwnerDisposed,
    );
    //添加设置监听
    window
      ..onMetricsChanged = handleMetricsChanged
      ..onTextScaleFactorChanged = handleTextScaleFactorChanged
      ..onPlatformBrightnessChanged = handlePlatformBrightnessChanged
      ..onSemanticsEnabledChanged = _handleSemanticsEnabledChanged
      ..onSemanticsAction = _handleSemanticsAction;
    initRenderView();
    _handleSemanticsEnabledChanged();
    assert(renderView != null);
    //添加persistentFrameCallback
    addPersistentFrameCallback(_handlePersistentFrameCallback);
    //创建触摸管理
    _mouseTracker = _createMouseTracker();
  }
复制代码
```

`addPersistentFrameCallback`这个方法主要向**persistentFrameCallback**添加了回调：

```
  void addPersistentFrameCallback(FrameCallback callback) {
    _persistentCallbacks.add(callback);
  }
复制代码
```

再看`_handlePersistentFrameCallback`这个回调做了什么：

```
  void _handlePersistentFrameCallback(Duration timeStamp) {
    drawFrame();
  }
复制代码
  @protected
  void drawFrame() {
    assert(renderView != null);
    pipelineOwner.flushLayout();//更新布局信息
    pipelineOwner.flushCompositingBits();//在flushLayout只后调用，在flushPaint之前调用，更新RenderObject是否需要重绘
    pipelineOwner.flushPaint();//更新绘制RenderObject
    renderView.compositeFrame(); // 发送bit数据给GPU
    pipelineOwner.flushSemantics(); // 发送语义数据给操作系统
  }
复制代码
```

下面一个一个方法走：

#### 4.1.flushLayout

```
  void flushLayout() {
    profile(() {
      Timeline.startSync('Layout', arguments: timelineWhitelistArguments);
    });
    assert(() {
      _debugDoingLayout = true;
      return true;
    }());
    try {
      // TODO(ianh): assert that we're not allowing previously dirty nodes to redirty themselves
      while (_nodesNeedingLayout.isNotEmpty) {
        final List<RenderObject> dirtyNodes = _nodesNeedingLayout;
        _nodesNeedingLayout = <RenderObject>[];
        for (RenderObject node in dirtyNodes..sort((RenderObject a, RenderObject b) => a.depth - b.depth)) {
          if (node._needsLayout && node.owner == this)
            node._layoutWithoutResize();
        }
      }
    } finally {
      assert(() {
        _debugDoingLayout = false;
        return true;
      }());
      profile(() {
        Timeline.finishSync();
      });
    }
  }
复制代码
```

看源码得知首先获取哪些标记为`脏`的`RenderObject`的布局信息，然后通过`ode._layoutWithoutResize();`重新调整这些`RenderObject`。

#### 4.2.flushCompositingBits

```
  void flushCompositingBits() {
    profile(() { Timeline.startSync('Compositing bits'); });
    _nodesNeedingCompositingBitsUpdate.sort((RenderObject a, RenderObject b) => a.depth - b.depth);
    for (RenderObject node in _nodesNeedingCompositingBitsUpdate) {
      if (node._needsCompositingBitsUpdate && node.owner == this)
        node._updateCompositingBits();
    }
    _nodesNeedingCompositingBitsUpdate.clear();
    profile(() { Timeline.finishSync(); });
  }
复制代码
```

检查`RenderObject`是否需要重绘，并且通过`node._updateCompositingBits();`更新`_needsCompositing`这个属性，若为`true`就要重新绘制，否则不需要。

#### 4.3.flushPaint

```
  void flushPaint() {
    profile(() { Timeline.startSync('Paint', arguments: timelineWhitelistArguments); });
    assert(() {
      _debugDoingPaint = true;
      return true;
    }());
    try {
      final List<RenderObject> dirtyNodes = _nodesNeedingPaint;
      _nodesNeedingPaint = <RenderObject>[];
      // Sort the dirty nodes in reverse order (deepest first).
      //方向遍历这些标记过的node
      for (RenderObject node in dirtyNodes..sort((RenderObject a, RenderObject b) => b.depth - a.depth)) {
        assert(node._layer != null);
        if (node._needsPaint && node.owner == this) {
          if (node._layer.attached) {
            //重新绘制
            PaintingContext.repaintCompositedChild(node);
          } else {
            node._skippedPaintingOnLayer();
          }
        }
      }
      assert(_nodesNeedingPaint.isEmpty);
    } finally {
      assert(() {
        _debugDoingPaint = false;
        return true;
      }());
      profile(() { Timeline.finishSync(); });
    }
  }

复制代码
```

这个方法通过反向遍历(`dirty`标记)取得需要重绘的`RenderObject`，最后通过`PaintingContext.repaintCompositedChild(node);`重绘。

#### 4.4.compositeFrame

```
  void compositeFrame() {
    Timeline.startSync('Compositing', arguments: timelineWhitelistArguments);
    try {
      //创建Scene对象
      final ui.SceneBuilder builder = ui.SceneBuilder();
      final ui.Scene scene = layer.buildScene(builder);
      if (automaticSystemUiAdjustment)
        _updateSystemChrome();
      //使用render方法将Scene对象显示在屏幕上    
      _window.render(scene);//调用flutter engine的渲染API
      scene.dispose();
      assert(() {
        if (debugRepaintRainbowEnabled || debugRepaintTextRainbowEnabled)
          debugCurrentRepaintColor = debugCurrentRepaintColor.withHue((debugCurrentRepaintColor.hue + 2.0) % 360.0);
        return true;
      }());
    } finally {
      Timeline.finishSync();
    }
  }
复制代码
```

`Scene`是用来保存渲染后的最终像素信息，这个方法将`Canvas`画好的`Scene`对象传给`window.render()`方法，该方法会直接将`Scene`信息发送给Flutter engine，最终Flutter engine将图像画在设备屏幕上，这样整个绘制流程就算完了。

注意：`RendererBinding`只是混入对象，最终混入到`WidgetsBinding`，回到最开始来看：

```
class WidgetsFlutterBinding extends BindingBase with GestureBinding, ServicesBinding, SchedulerBinding, PaintingBinding, SemanticsBinding, RendererBinding, WidgetsBinding {

  /// Returns an instance of the [WidgetsBinding], creating and
  /// initializing it if necessary. If one is created, it will be a
  /// [WidgetsFlutterBinding]. If one was previously initialized, then
  /// it will at least implement [WidgetsBinding].
  ///
  /// You only need to call this method if you need the binding to be
  /// initialized before calling [runApp].
  ///
  /// In the `flutter_test` framework, [testWidgets] initializes the
  /// binding instance to a [TestWidgetsFlutterBinding], not a
  /// [WidgetsFlutterBinding].
  static WidgetsBinding ensureInitialized() {
    if (WidgetsBinding.instance == null)
      WidgetsFlutterBinding();
    return WidgetsBinding.instance;
  }
}
复制代码
```

所以应该`WidgetsBinding`来重写实现`drawFrame`方法：

```
  @override
  void drawFrame() {
    assert(!debugBuildingDirtyElements);
    assert(() {
      debugBuildingDirtyElements = true;
      return true;
    }());
    try {
      if (renderViewElement != null)
        buildOwner.buildScope(renderViewElement);
      super.drawFrame(); //调用Renderbinding的drawFrame方法
      buildOwner.finalizeTree();
    } finally {
      assert(() {
        debugBuildingDirtyElements = false;
        return true;
      }());
    }
    profile(() {
      if (_needToReportFirstFrame && _reportFirstFrame) {
        developer.Timeline.instantSync('Widgets completed first useful frame');
        developer.postEvent('Flutter.FirstFrame', <String, dynamic>{});
        _needToReportFirstFrame = false;
      }
    });
  }
复制代码
```

## 四、总结

- `widget`的功能是向`element`提供配置信息，每一个`widget`在Flutter里是一份配置数据，而代表屏幕背后的元素是`element`，而真正的布局、渲染是通过`RenderObject`来完成的，从创建到渲染的主要流程是：`widget`信息生成`element`，创建对应的`RenderObject`关联到`Element.renderObject`属性上，最后通过`RenderObject`布局和绘制。
- Flutter从启动到显示图像在屏幕主要经过：首先监听处理`window`对象的事件，将这些事件处理包装为Framework模型进行分发，通过`widget`创建`element`树，接着通过`scheduleWarmUpFrame`进行渲染，接着通过`Rendererbinding`进行布局，绘制，最后通过调用`ui.window.render(scene)`Scene信息发给Flutter engine，Flutter engine最后调用渲染API把图像画在屏幕上。