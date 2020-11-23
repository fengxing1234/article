---
title: 深入理解setState更新机制
date: 2020-08-04 13:54:51
tags:
	- flutter
	- 源码解析
categories: 
	- flutter
	- 源码解析
---

> 
> 基于Flutter 1.5的源码剖析， 分析flutter的StatefulWidget的UI更新机制，相关源码：

```
widgets/framework.dart
widgets/binding.dart
scheduler/binding.dart
lib/ui/window.dart
flutter/runtime/runtime_controller.cc
```

## 一、概述

对于Flutter来说，万物皆为Widget，常见的Widget子类为StatelessWidget(无状态)和StatefulWidget(有状态)；

- StatelessWidget：内部没有保存状态，界面创建后不会发生改变；
- StatefulWidget：内部有保存状态，当状态发生改变，调用setState()方法会触发StatefulWidget的UI发生更新，对于自定义继承自StatefulWidget的子类，必须要重写createState()方法。

接下来看看setState()究竟干了哪些操作。

## 二、Widget更新流程

### 2.1 setState

[-> framework.dart:: State]

```Java
abstract class State<T extends StatefulWidget> extends Diagnosticable {
  StatefulElement _element;

  void setState(VoidCallback fn) {
    ...
    _element.markNeedsBuild();  //[见小节2.2]
  }
}
```

这里需要注意setState()不应该在dispose()之后调用，可通过mounted属性值来判断父widget是否还包含该widget。

### 2.2 markNeedsBuild

[-> framework.dart:: Element]

```Java
abstract class Element extends DiagnosticableTree implements BuildContext {
  void markNeedsBuild() {
    if (!_active)
      return;
    if (dirty)
      return;
    _dirty = true;
    owner.scheduleBuildFor(this);   //[见小节2.3]
  }
}
```

设置_dirty为true，

### 2.3 scheduleBuildFor

[-> framework.dart:: BuildOwner]

```Java
void scheduleBuildFor(Element element) {
  if (element._inDirtyList) {
    _dirtyElementsNeedsResorting = true;
    return;
  }
  if (!_scheduledFlushDirtyElements && onBuildScheduled != null) {
    _scheduledFlushDirtyElements = true;
    onBuildScheduled();  //[见小节2.4]
  }
  _dirtyElements.add(element);  //记录所有的脏元素
  element._inDirtyList = true;
}
```

### 2.4 _handleBuildScheduled

[-> binding.dart:: WidgetsBinding]

```Java
mixin WidgetsBinding on BindingBase, SchedulerBinding, GestureBinding, RendererBinding, SemanticsBinding {
  @override
  void initInstances() {
    super.initInstances();
    _instance = this;
    buildOwner.onBuildScheduled = _handleBuildScheduled;   //赋值onBuildScheduled
    ui.window.onLocaleChanged = handleLocaleChanged;
    ui.window.onAccessibilityFeaturesChanged = handleAccessibilityFeaturesChanged;
    SystemChannels.navigation.setMethodCallHandler(_handleNavigationInvocation);
    SystemChannels.system.setMessageHandler(_handleSystemMessage);
  }

  void _handleBuildScheduled() {
    ensureVisualUpdate();  //[见小节2.5]
  }
}
```

在[Flutter应用启动](http://gityuan.com/2019/06/29/flutter_run_app/)过程初始化WidgetsBinding时，赋值onBuildScheduled等于_handleBuildScheduled()。

### 2.5 ensureVisualUpdate

[-> scheduler/binding.dart:: SchedulerBinding]

```Java
mixin SchedulerBinding on BindingBase, ServicesBinding {
  @override
  void initInstances() {
    super.initInstances();
    _instance = this;
    ui.window.onBeginFrame = _handleBeginFrame;
    ui.window.onDrawFrame = _handleDrawFrame;
    SystemChannels.lifecycle.setMessageHandler(_handleLifecycleMessage);
  }

  void ensureVisualUpdate() {
    switch (schedulerPhase) {
      case SchedulerPhase.idle:
      case SchedulerPhase.postFrameCallbacks:
        scheduleFrame();  //[见小节2.6]
        return;
      case SchedulerPhase.transientCallbacks:
      case SchedulerPhase.midFrameMicrotasks:
      case SchedulerPhase.persistentCallbacks:
        return;
    }
  }
}
```

schedulerPhase的初始值为SchedulerPhase.idle。SchedulerPhase是一个enum枚举类型，有以下5个可取值：

| 状态                    | 含义                                                         |
| :---------------------- | :----------------------------------------------------------- |
| **idle**                | 没有正在处理的帧，可能正在执行的是WidgetsBinding.scheduleTask，scheduleMicrotask，Timer，事件handlers，或者其他回调等 |
| **transientCallbacks**  | SchedulerBinding.handleBeginFrame过程， 处理动画状态更新     |
| **midFrameMicrotasks**  | 处理transientCallbacks阶段触发的微任务（Microtasks）         |
| **persistentCallbacks** | WidgetsBinding.drawFrame和SchedulerBinding.handleDrawFrame过程，build/layout/paint流水线工作 |
| **postFrameCallbacks**  | 主要是清理和计划执行下一帧的工作                             |

### 2.6 scheduleFrame

[-> scheduler/binding.dart:: SchedulerBinding]

```Java
void scheduleFrame() {
  //只有当APP处于用户可见状态才会准备调度下一帧方法
  if (_hasScheduledFrame || !_framesEnabled)
    return;
  ensureFrameCallbacksRegistered();
  ui.window.scheduleFrame();  // native方法 [见小节2.7]
  _hasScheduledFrame = true;
}
```

`WidgetBinding`的`scheduleFrame`会首先调用`ensureFrameCallbacksRegistered`方法确保`window`的回调函数以被注册。再调用`window`的`scheduleFrame`的方法。

```
void ensureFrameCallbacksRegistered() {
    window.onBeginFrame ??= _handleBeginFrame;
    window.onDrawFrame ??= _handleDrawFrame;
}
```

### 2.7 scheduleFrame

[-> lib/ui/window.dart:: Window]

```Java
void scheduleFrame() native 'Window_scheduleFrame';
```

window是Flutter引擎中跟图形相关接口打交道的核心类，这里是一个native方法

#### 2.7.1 ScheduleFrame(C++)

[-> window.cc ]

```C++
void ScheduleFrame(Dart_NativeArguments args) {
  //  [见小节2.7.2]
  UIDartState::Current()->window()->client()->ScheduleFrame();
}
```

通过RegisterNatives()完成native方法的注册，“Window_scheduleFrame”所对应的native方法如上所示。

#### 2.7.2 RuntimeController::ScheduleFrame

[flutter/runtime/runtime_controller.cc]

```C++
void RuntimeController::ScheduleFrame() {
  client_.ScheduleFrame(); //  [见小节2.7.3]
}
```

#### 2.7.3 Engine::ScheduleFrame

flutter/shell/common/engine.cc

```C++
void Engine::ScheduleFrame(bool regenerate_layer_tree) {
  animator_->RequestFrame(regenerate_layer_tree);
}
```

文章[Flutter渲染机制—UI线程](http://gityuan.com/2019/06/15/flutter_ui_draw/)文中小节[2.1]介绍Engine::ScheduleFrame()经过层层调用，最终会注册Vsync回调。 等待下一次vsync信号的到来，然后再经过层层调用最终会调用到Window::BeginFrame()。

### 2.8 Window::BeginFrame

[-> flutter/lib/ui/window/window.cc]

```Java
void Window::BeginFrame(fml::TimePoint frameTime) {
  std::shared_ptr<tonic::DartState> dart_state = library_.dart_state().lock();
  if (!dart_state)
    return;
  tonic::DartState::Scope scope(dart_state);

  int64_t microseconds = (frameTime - fml::TimePoint()).ToMicroseconds();

  // [见小节4.2]
  DartInvokeField(library_.value(), "_beginFrame",
                  {
                      Dart_NewInteger(microseconds),
                  });

  //执行MicroTask
  UIDartState::Current()->FlushMicrotasksNow();

  // [见小节4.4]
  DartInvokeField(library_.value(), "_drawFrame", {});
}
```

Window::BeginFrame()过程主要工作：

- 执行_beginFrame
- 执行FlushMicrotasksNow
- 执行_drawFrame

可见，Microtask位于beginFrame和drawFrame之间，那么Microtask的耗时会影响ui绘制过程。

### 2.9 handleBeginFrame

[-> lib/src/scheduler/binding.dart:: SchedulerBinding]

```Java
void handleBeginFrame(Duration rawTimeStamp) {
  Timeline.startSync('Frame', arguments: timelineWhitelistArguments);
  _firstRawTimeStampInEpoch ??= rawTimeStamp;
  _currentFrameTimeStamp = _adjustForEpoch(rawTimeStamp ?? _lastRawTimeStamp);
  if (rawTimeStamp != null)
    _lastRawTimeStamp = rawTimeStamp;

  profile(() {
    _profileFrameNumber += 1;
    _profileFrameStopwatch.reset();
    _profileFrameStopwatch.start();
  });

  //此时阶段等于SchedulerPhase.idle;
  _hasScheduledFrame = false;
  try {
    Timeline.startSync('Animate', arguments: timelineWhitelistArguments);
    _schedulerPhase = SchedulerPhase.transientCallbacks;
    //执行动画的回调方法
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
```

该方法主要功能是遍历_transientCallbacks，执行相应的Animate操作，可通过scheduleFrameCallback()/cancelFrameCallbackWithId()来完成添加和删除成员，再来简单看看这两个方法。

### 2.10 handleDrawFrame

[-> lib/src/scheduler/binding.dart:: SchedulerBinding]

```Java
void handleDrawFrame() {
  assert(_schedulerPhase == SchedulerPhase.midFrameMicrotasks);
  Timeline.finishSync(); // 标识结束"Animate"阶段
  try {
    _schedulerPhase = SchedulerPhase.persistentCallbacks;
    //执行PERSISTENT FRAME回调
    for (FrameCallback callback in _persistentCallbacks)
      _invokeFrameCallback(callback, _currentFrameTimeStamp);

    _schedulerPhase = SchedulerPhase.postFrameCallbacks;
    // 执行POST-FRAME回调
    final List<FrameCallback> localPostFrameCallbacks = List<FrameCallback>.from(_postFrameCallbacks);
    _postFrameCallbacks.clear();
    for (FrameCallback callback in localPostFrameCallbacks)
      _invokeFrameCallback(callback, _currentFrameTimeStamp);
  } finally {
    _schedulerPhase = SchedulerPhase.idle;
    Timeline.finishSync(); //标识结束”Frame“阶段
    profile(() {
      _profileFrameStopwatch.stop();
      _profileFramePostEvent();
    });
    _currentFrameTimeStamp = null;
  }
}
```

`handleDrawFrame`方法上面的注释已经说了该方法的作用，被引擎调用创建一个新的帧。这个方法流程也比较清晰，首先会循环执行`_persistentCallbacks`中的callback，这里的callback可以通过`WidgetsBinding.instance.addPersistentFrameCallback(fn)`注册；然后，再复制一份`_postFrameCallbacks`的拷贝，并将原`_postFrameCallbacks`列表清空，`_postFrameCallbacks`中保存重绘后执行的回调函数，并且只执行一次，可以通过`WidgetsBinding.instance.addPostFrameCallback(fn)`添加回调。执行完`_persistentCallbacks`和`_postFrameCallbacks`后，便将状态设置为`SchedulerPhase.idle`表示已经刷新过。

该方法主要功能：

- 遍历_persistentCallbacks，执行相应的回调方法，可通过addPersistentFrameCallback()注册，一旦注册后不可移除，后续每一次frame回调都会执行；
- 遍历_postFrameCallbacks，执行相应的回调方法，可通过addPostFrameCallback()注册，handleDrawFrame()执行完成后会清空_postFrameCallbacks内容。

### 2.11 addPersistentFrameCallback

```
 /// Adds a persistent frame callback.
  ///
  /// Persistent callbacks are called after transient
  /// (non-persistent) frame callbacks.
  ///
  /// Does *not* request a new frame. Conceptually, persistent frame
  /// callbacks are observers of "begin frame" events. Since they are
  /// executed after the transient frame callbacks they can drive the
  /// rendering pipeline.
  ///
  /// Persistent frame callbacks cannot be unregistered. Once registered, they
  /// are called for every frame for the lifetime of the application.
  void addPersistentFrameCallback(FrameCallback callback) {
    _persistentCallbacks.add(callback);
  }
```

通过注释可以知道是通过`addPersistentFrameCallback`来驱动渲染的。通过搜索，可以看到在`RendererBinding`的`initInstances`方法中注册了`persistentFrameCallback`回调。

```
mixin RendererBinding on BindingBase, ServicesBinding, SchedulerBinding, GestureBinding, SemanticsBinding, HitTestable {
  @override
  void initInstances() {
    super.initInstances();
    ...
    initRenderView();
    _handleSemanticsEnabledChanged();
    assert(renderView != null);
    addPersistentFrameCallback(_handlePersistentFrameCallback);
    initMouseTracker();
  }
}
```

### 2.12 _handlePersistentFrameCallback

在`_handlePersistentFrameCallback`回调函数中直接调用了`drawFrame`方法。

```
void _handlePersistentFrameCallback(Duration timeStamp) {
    drawFrame();
    _mouseTracker.schedulePostFrameCheck();
  }
```

```
@protected
void drawFrame() {
    assert(renderView != null);
    pipelineOwner.flushLayout();
    pipelineOwner.flushCompositingBits();
    pipelineOwner.flushPaint();
    if (sendFramesToEngine) {
        renderView.compositeFrame(); // this sends the bits to the GPU
        pipelineOwner.flushSemantics(); // this also sends the semantics to the OS.
        _firstFrameSent = true;
    }
}

```

需要注意的是，`WidgetsBinding`也实现了`drawFrame`，并且`WidgetsBinding`在被mixin到`WidgetsFlutterBinding`类时是在最右边，所以它的方法优先级最高。`_handlePersistentFrameCallback`中调用`drawFrame`方法时，会先调用`WidgetsBinding`中的`drawFrame`方法。

### 2.13 WidgetsBinding drawFrame

```
@override
  void drawFrame() {
    ...  
    try {
      if (renderViewElement != null)
        buildOwner.buildScope(renderViewElement);
      super.drawFrame();
      buildOwner.finalizeTree();
    } finally {
      assert(() {
        debugBuildingDirtyElements = false;
        return true;
      }());
    }
    ...
  }
```

在`WidgetsBinding`的`drawFrame`方法中，先调用了`buildOwner`的`buildScope`方法，然后再调用了`super.drawFrame()`，通过`super.drawFrame()`可以调用到`RendererBinding`的`drawFrame`方法。先看`buildOwner`的`buildScope`方法。

### 2.14 BuildOwner buildScope

```
void buildScope(Element context, [ VoidCallback callback ]) {
    if (callback == null && _dirtyElements.isEmpty)
      return;
    ...
    try {
      _dirtyElements.sort(Element._sort);
      _dirtyElementsNeedsResorting = false;
      int dirtyCount = _dirtyElements.length;
      int index = 0;
      while (index < dirtyCount) {
        ...
        try {
          _dirtyElements[index].rebuild();
        } catch (e, stack) {
          ...
        }
        index += 1;
        if (dirtyCount < _dirtyElements.length || _dirtyElementsNeedsResorting) {
          _dirtyElements.sort(Element._sort);
          _dirtyElementsNeedsResorting = false;
          dirtyCount = _dirtyElements.length;
          while (index > 0 && _dirtyElements[index - 1].dirty) {
            // It is possible for previously dirty but inactive widgets to move right in the list.
            // We therefore have to move the index left in the list to account for this.
            // We don't know how many could have moved. However, we do know that the only possible
            // change to the list is that nodes that were previously to the left of the index have
            // now moved to be to the right of the right-most cleaned node, and we do know that
            // all the clean nodes were to the left of the index. So we move the index left
            // until just after the right-most clean node.
            index -= 1;
          }
        }
      }
      ...
    } finally {
      for (final Element element in _dirtyElements) {
        assert(element._inDirtyList);
        element._inDirtyList = false;
      }
      _dirtyElements.clear();
      _scheduledFlushDirtyElements = false;
      _dirtyElementsNeedsResorting = null;
      ...
    }
  }
```

`buildScope`的核心逻辑就是，首先对`_dirtyElements`按照深度进行排序，再遍历`_dirtyElements`列表，调用其中元素的`rebuild`方法。`rebuild`方法定义在Element类中

```
void rebuild() {
    ...
    performRebuild();
    ...
  }

```

```
@protected
  void performRebuild();

```

`performRebuild`是`Element`类中的抽象方法，各个子类会实现该方法。`StateElement`的父类是`ComponentElement`，先看`ComponentElement`的`performRebuild`方法

```
@override
  void performRebuild() {
    ...
    Widget built;
    try {
      ..
      built = build();
      ..
    } catch (e, stack) {
      _debugDoingBuild = false;
      built = ErrorWidget.builder(
        _debugReportException(
          ErrorDescription('building $this'),
          e,
          stack,
          informationCollector: () sync* {
            yield DiagnosticsDebugCreator(DebugCreator(this));
          },
        ),
      );
    } finally {
      ...
    }
    try {
      _child = updateChild(_child, built, slot);
      assert(_child != null);
    } catch (e, stack) {
      ...
      _child = updateChild(null, built, slot);
    }
    ...
  }

```

在这个方法中，直接调用`build`方法创建Widget，如果`build`方法产生异常，就会创建一个`ErrorWidget`,就是经常看到的红色警告界面。调用完`build`方法后，会再调用`updateChild(_child, built, slot)`更新子Widget。 `StatelessElement`和`StatefulElement`重写了`build`方法，分别调用了`Widget`和`State`的`build`方法。

```
///StatelessElement
@override
Widget build() => widget.build(this);

```

```
@override
Widget build() => _state.build(this);

```

前面提到`WidgetsBinding`的`drawFrame`方法会通过`super.drawFrame()`调用到`RendererBinding`的`drawFrame`方法，再回头看`RendererBinding`的`drawFrame`方法。

```
@protected
  void drawFrame() {
    assert(renderView != null);
    pipelineOwner.flushLayout();
    pipelineOwner.flushCompositingBits();
    pipelineOwner.flushPaint();
    if (sendFramesToEngine) {
      renderView.compositeFrame(); // this sends the bits to the GPU
      pipelineOwner.flushSemantics(); // this also sends the semantics to the OS.
      _firstFrameSent = true;
    }
  }
```

`RendererBinding`的`drawFrame`方法，通过`pipelineOwner`对象重新layout和paint，已达到更新UI的效果。

## 三、小结

可见，setState()过程主要工作是记录所有的脏元素，添加到BuildOwner对象的_dirtyElements成员变量，然后调用scheduleFrame来注册Vsync回调。 当下一次vsync信号的到来时会执行handleBeginFrame()和handleDrawFrame()来更新UI。在收到刷新回调后，遍历`dirtyElements`中的元素，执行`rebuild`操作，以更新显示状态。