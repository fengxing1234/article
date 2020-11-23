---
title: Flutter之Timer原理解析
date: 2020-06-29 17:17:22
tags:
---

# Flutter之Timer原理解析

在开发中，`Timer`总是一定无法绕过的。通过它，我们可以来实现任务的轮询、定时执行等。当然，由于一些原因，一些平台中不建议使用`Timer`。在`Android`中，基本上就是不建议使用它，而是通过`Handler`、`ScheduledThreadPoolExecutor`等来替代`Timer`。那如果在`Flutter`中尼？下面就来看`Flutter`中如何使用`Timer`及`Timer`的实现原理。

### 1、Timer的使用

在`Flutter`中，`Timer`是无处不在的，有直接使用其API，也有使用`Timer`的包装类，如`Future`。但在本文，会通过`Timer`及其API来深入了解实现原理。先来看`Timer`的使用。

```
//任务的定时执行。延迟1秒后输出f1
Timer(Duration(milliseconds: 1000), () {
  print("f1");
});
int count = 0;
//任务的周期性执行
Timer.periodic(Duration(milliseconds: 1000), (timer) {
  print("f2");
  count++;
  if (count == 3) {
    //当执行count=3时，取消timer中的任务
    timer.cancel();
  }
});
//异步任务执行，输出f3
Timer.run(() {
  print("f3");
});
复制代码
```

以上就是`Timer`的全部用法，主要是任务的定时执行、任务的周期性执行及任务的异步执行。由于任务周期性执行的实现原理与任务定时执行的实现原理基本相同，所有`Timer`就主要分为定时任务的执行及异步任务的执行两中情况。

下面也就根据这两种情况来分析`Timer`的实现原理。

### 2、Timer原理解析

由于无论那种任务类型都需要创建一个`Timer`对象，所以就先来看`Timer`对象的创建。

```
abstract class Timer {
  //延迟一定时间后执行callback
  factory Timer(Duration duration, void callback()) {
    if (Zone.current == Zone.root) {
      // No need to bind the callback. We know that the root's timer will
      // be invoked in the root zone.
      return Zone.current.createTimer(duration, callback);
    }
    return Zone.current
        .createTimer(duration, Zone.current.bindCallbackGuarded(callback));
  }
  //创建一个timer对象
  external static Timer _createTimer(Duration duration, void callback());
  //创建一个可轮询timer对象
  external static Timer _createPeriodicTimer(
      Duration duration, void callback(Timer timer));
}
复制代码
```

在上面代码中，`Timer`构造函数中最会终调用`_createTimer`来创建一个`_Timer`对象。所以下面就来看`_createTimer`方法的具体实现。

```
@patch
class Timer {
  @patch
  static Timer _createTimer(Duration duration, void callback()) {
    // TODO(iposva): Remove _TimerFactory and use VMLibraryHooks exclusively.
    if (_TimerFactory._factory == null) {
      _TimerFactory._factory = VMLibraryHooks.timerFactory;
    }
    if (_TimerFactory._factory == null) {
      throw new UnsupportedError("Timer interface not supported.");
    }
    int milliseconds = duration.inMilliseconds;
    if (milliseconds < 0) milliseconds = 0;
    return _TimerFactory._factory(milliseconds, (_) {
      callback();
    }, false);
  }
}
复制代码
```

在`_createTimer`中，最终调用的是`_TimerFactory._factory`方法。由于在`Flutter`的第一个`isolate`初始化成功后，会调用`_setupHooks`方法将`_Timer._factory`赋给`_TimerFactory._factory`。所以`_createTimer`中最终调用了`_Timer._factory`方法。

```
@pragma("vm:entry-point", "call")
_setupHooks() {
  VMLibraryHooks.timerFactory = _Timer._factory;
}
复制代码
```

在`_Timer._factory`方法中，直接就是创建一个`_timer`对象，也就是在`Timer`的具体实现类是`_Timer`。下面就来看`_Timer`的具体实现代码。

```
class _Timer implements Timer {
  //消息类型：表示需要取消event handler中已存在某个timer
  static const _NO_TIMER = -1;

  //根据发送的值来区分消息类型
  //消息类型：表示异步执行的timer
  static const _ZERO_EVENT = 1;
  //消息类型：表示已经超时的timer
  static const _TIMEOUT_EVENT = null;

  //创建一个二叉堆，该堆按照唤醒时间进行排序。
  static final _heap = new _TimerHeap();
  //链表中的第一个Timer
  static _Timer _firstZeroTimer;
  //链表中的最后一个Timer
  static _Timer _lastZeroTimer;

  //使用id来对具有相同到期时间的Timer进行排序。
  //ID_MASK入队后或计时器队列为空时，将回收ID。
  static const _ID_MASK = 0x1fffffff;
  static int _idCount = 0;

  static RawReceivePort _receivePort;
  static SendPort _sendPort;
  static int _scheduledWakeupTime;

  static bool _handlingCallbacks = false;

  Function _callback; //timer触发的回调方法，如果timer已被取消，则为null
  int _wakeupTime; //唤醒时间
  final int _milliSeconds; //创建指定的持续时间
  final bool _repeating; //是否周期性
  var _indexOrNext; //如果Timer在_TimerHeap中，该值就是在该堆中的索引。如果是在链表中，则是当前Timer指向的下一个Timer
  int _id; //当前Timer所对应的id，如果到期时间相同，则根据此id来进行排序

  int _tick = 0; // Backing for [tick],

  //获取下一个可用id
  static int _nextId() {
    var result = _idCount;
    _idCount = (_idCount + 1) & _ID_MASK;
    return result;
  }
  //创建一个Timer对象
  _Timer._internal(
      this._callback, this._wakeupTime, this._milliSeconds, this._repeating)
      : _id = _nextId();

  static Timer _createTimer(
      void callback(Timer timer), int milliSeconds, bool repeating) {
    //milliSeconds不能小于0，小于0也就意味着超时，需要立即执行。
    if (milliSeconds < 0) {
      milliSeconds = 0;
    }

    //获取当前时间
    int now = VMLibraryHooks.timerMillisecondClock();
    //得到Timer的唤醒时间
    int wakeupTime = (milliSeconds == 0) ? now : (now + 1 + milliSeconds);
    
    //创建一个Timer对象
    _Timer timer =
        new _Timer._internal(callback, wakeupTime, milliSeconds, repeating);
    //将新创建的Timer放到适当的结构中，并在必要时进行对应的通知。
    //如果Timer中是异步任务，则加入到链表中，否则加入到二叉堆中
    timer._enqueue();
    return timer;
  }
  
  //通过工厂模式来创建一个timer
  factory _Timer(int milliSeconds, void callback(Timer timer)) {
    return _createTimer(callback, milliSeconds, false);
  }
  
  //通过工厂模式来创建一个周期性执行的timer
  factory _Timer.periodic(int milliSeconds, void callback(Timer timer)) {
    return _createTimer(callback, milliSeconds, true);
  }
  
  //timer是否在二叉堆中
  bool get _isInHeap => _indexOrNext is int;

  //首先根据唤醒时间来排序，如果唤醒时间相同则根据timer的_id来排序
  int _compareTo(_Timer other) {
    int c = _wakeupTime - other._wakeupTime;
    if (c != 0) return c;
    return _id - other._id;
  }
  
  //判断timer是否可使用，实际上就是判断回调方法是否为null
  bool get isActive => _callback != null;

  int get tick => _tick;

  //取消已经设置的timer，如果Timer存在于二叉堆中，则将其从堆中删除。否则继续保留在链表中，因为它们需要消耗相应的待处理消息。
  void cancel() {
    _callback = null;
    //实际上只有存在于二叉堆中的Timer被删除。链表中的Timer需要消耗其相应的唤醒消息，以便将它们留在队列中。
    if (!_isInHeap) return;
    bool update = _heap.isFirst(this);
    _heap.remove(this);
    if (update) {
      _notifyEventHandler();
    }
  }
  
  //主要是重新计算下一次的唤醒时间。仅会在周期性执行的Timer中调用，
  void _advanceWakeupTime() {
    //重新计算下一次唤醒时间。 对于已经超时的Timer，当前时间就是下一个唤醒时间。
    _id = _nextId();
    if (_milliSeconds > 0) {
      _wakeupTime += _milliSeconds;
    } else {
      _wakeupTime = VMLibraryHooks.timerMillisecondClock();
    }
  }

  //将Timer添加到二叉堆或者链表中，如果唤醒时间相同则按照先进先出的规则来取出
  void _enqueue() {
    if (_milliSeconds == 0) {
      if (_firstZeroTimer == null) {
        _lastZeroTimer = this;
        _firstZeroTimer = this;
      } else {
        _lastZeroTimer._indexOrNext = this;
        _lastZeroTimer = this;
      }
      // Every zero timer gets its own event.
      _notifyZeroHandler();
    } else {
      _heap.add(this);
      if (_heap.isFirst(this)) {
        _notifyEventHandler();
      }
    }
  }

  //对于包含异步任务的timer，需要发送一个消息类型为_ZERO_EVENT的消息。之所以消息类型是_ZERO_EVENT，主要是为了区分EventHandler消息（_TIMEOUT_EVENT消息）。
  static void _notifyZeroHandler() {
    if (_sendPort == null) {
      _createTimerHandler();
    }
    _sendPort.send(_ZERO_EVENT);
  }

  //从链表中获取即将执行的timer及二叉堆中到期时间小于_firstZeroTimer的timer
  static List _queueFromZeroEvent() {
    var pendingTimers = new List();
    
    //从二叉堆中查询到期时间小于_firstZeroTimer的timer，并加入到一个List中
    var timer;
    while (!_heap.isEmpty && (_heap.first._compareTo(_firstZeroTimer) < 0)) {
      timer = _heap.removeFirst();
      pendingTimers.add(timer);
    }
    //获取链表中的第一个timer
    timer = _firstZeroTimer;
    _firstZeroTimer = timer._indexOrNext;
    timer._indexOrNext = null;
    pendingTimers.add(timer);
    return pendingTimers;
  }

  static void _notifyEventHandler() {
    if (_handlingCallbacks) {
      //如果正在进行timer的回调处理，则不继续向下执行
      return;
    }

    //如果不存在即将执行的timers，则关闭receive port
    if ((_firstZeroTimer == null) && _heap.isEmpty) {
      //没有待处理的计时器，则关闭receive port并通知event handler。
      if (_sendPort != null) {
        _cancelWakeup();
        _shutdownTimerHandler();
      }
      return;
    } else if (_heap.isEmpty) {
      //如果二叉堆中不存在timer，则取消唤醒。
      _cancelWakeup();
      return;
    }

    //仅在请求的唤醒时间与预定的唤醒时间不同时发送消息。
    var wakeupTime = _heap.first._wakeupTime;
    if ((_scheduledWakeupTime == null) ||
        (wakeupTime != _scheduledWakeupTime)) {
      _scheduleWakeup(wakeupTime);
    }
  }

  //获取已经超时的timer
  static List _queueFromTimeoutEvent() {
    var pendingTimers = new List();
    if (_firstZeroTimer != null) {
      //从二叉堆中获取唤醒时间小于链表中第一个timer唤醒时间的timer，并将该timer添加到pendingTimers中
      var timer;
      while (!_heap.isEmpty && (_heap.first._compareTo(_firstZeroTimer) < 0)) {
        timer = _heap.removeFirst();
        pendingTimers.add(timer);
      }
    } else {
      //从二叉堆中获取已经到期的timer并添加到pendingTimers中
      var currentTime = VMLibraryHooks.timerMillisecondClock();
      var timer;
      while (!_heap.isEmpty && (_heap.first._wakeupTime <= currentTime)) {
        timer = _heap.removeFirst();
        pendingTimers.add(timer);
      }
    }
    return pendingTimers;
  }

  static void _runTimers(List pendingTimers) {
    //如果目前没有待处理的timer，那么就有机会在新加入timer之前来重置_idCount
    if (_heap.isEmpty && (_firstZeroTimer == null)) {
      _idCount = 0;
    }

    //如果没有待处理的timer，则结束方法的执行
    if (pendingTimers.length == 0) {
      return;
    }

    // Trigger all of the pending timers. New timers added as part of the
    // callbacks will be enqueued now and notified in the next spin at the
    // earliest.
    _handlingCallbacks = true;
    var i = 0;
    try {
      for (; i < pendingTimers.length; i++) {
        //获取下一个timer
        var timer = pendingTimers[i];
        timer._indexOrNext = null;

        // One of the timers in the pending_timers list can cancel
        // one of the later timers which will set the callback to
        // null. Or the pending zero timer has been canceled earlier.
        if (timer._callback != null) {
          var callback = timer._callback;
          if (!timer._repeating) {
            //将timer标记为无效
            timer._callback = null;
          } else if (timer._milliSeconds > 0) {
            var ms = timer._milliSeconds;
            int overdue =
                VMLibraryHooks.timerMillisecondClock() - timer._wakeupTime;
            if (overdue > ms) {
              int missedTicks = overdue ~/ ms;
              timer._wakeupTime += missedTicks * ms;
              timer._tick += missedTicks;
            }
          }
          timer._tick += 1;
          
          //执行timer中注册的回调方法
          callback(timer);
          // Re-insert repeating timer if not canceled.
          //如果timer未取消，则重新插入链表或者二叉堆中
          if (timer._repeating && (timer._callback != null)) {
            //更新唤醒时间
            timer._advanceWakeupTime();
            timer._enqueue();
          }
          //执行微任务，仅限于非RootIsolate。
          var immediateCallback = _removePendingImmediateCallback();
          if (immediateCallback != null) {
            immediateCallback();
          }
        }
      }
    } finally {
      _handlingCallbacks = false;
      //重新向二叉堆或者链表中插入pendingTimers中还存在的timer
      for (i++; i < pendingTimers.length; i++) {
        var timer = pendingTimers[i];
        timer._enqueue();
      }
      _notifyEventHandler();
    }
  }

  static void _handleMessage(msg) {
    var pendingTimers;
    if (msg == _ZERO_EVENT) {
      //获取包含异步任务的timer
      pendingTimers = _queueFromZeroEvent();
    } else {
      _scheduledWakeupTime = null; // Consumed the last scheduled wakeup now.
      //获取已经超时的timer
      pendingTimers = _queueFromTimeoutEvent();
    }
    //执行Timer的回调方法
    _runTimers(pendingTimers);
    //如果当前没有待执行的Timer，则通知event handler或者关闭port
    _notifyEventHandler();
  }

  //告诉event handler，在特定时间，当前isolated中的timer需要被唤醒
  static void _scheduleWakeup(int wakeupTime) {
    if (_sendPort == null) {
      _createTimerHandler();
    }
    VMLibraryHooks.eventHandlerSendData(null, _sendPort, wakeupTime);
    _scheduledWakeupTime = wakeupTime;
  }

  //取消event handler中等待唤醒的timer
  static void _cancelWakeup() {
    if (_sendPort != null) {
      VMLibraryHooks.eventHandlerSendData(null, _sendPort, _NO_TIMER);
      _scheduledWakeupTime = null;
    }
  }

  //创建一个receive port并注册一个message handler
  static void _createTimerHandler() {
    assert(_sendPort == null);
    _receivePort = new RawReceivePort(_handleMessage);
    _sendPort = _receivePort.sendPort;
    _scheduledWakeupTime = null;
  }

  static void _shutdownTimerHandler() {
    _receivePort.close();
    _receivePort = null;
    _sendPort = null;
    _scheduledWakeupTime = null;
  }
  
  //创建_timer对象
  static Timer _factory(
      int milliSeconds, void callback(Timer timer), bool repeating) {
    if (repeating) {
      return new _Timer.periodic(milliSeconds, callback);
    }
    return new _Timer(milliSeconds, callback);
  }
}
复制代码
```

在`_Timer`中，根据任务类型的不同，将`timer`添加到不同的数据结构中。如果是异步任务，则会将`timer`添加到一个单链表中，根据FIFO的顺序来执行；如果是定时任务，则会将`timer`添加到[二叉堆](https://zh.wikipedia.org/wiki/二叉堆)中并根据唤醒时间来进行排序。

下面就先来看异步任务执行的实现原理。

#### 2.1、异步任务的执行

根据上面代码。可以发现，包含异步任务的`timer`是将`timer`添加到以`Timer`为节点的单链表中，再通过`SendPort`来发送一个类型为`_ZERO_EVENT`的消息。

那么`SendPort`是如何发送消息的尼？这在[Isolate的创建流程](https://juejin.im/post/5e149a7df265da5d3b32e167)一文中做了详细的介绍。其发送消息就是通过`PostMessage`函数来将消息添加到`Isolate`对应的`MessageHandler`中，然后等待`MessageHandler`的处理。下面来看`PostMessage`函数的实现代码。

[->third_party/dart/runtime/vm/message_handler.cc]

```
void MessageHandler::PostMessage(std::unique_ptr<Message> message,
                                 bool before_events) {
  Message::Priority saved_priority;

  {
    MonitorLocker ml(&monitor_);
    ...

    saved_priority = message->priority();
    if (message->IsOOB()) {
      //加入到OOB类型消息的队列中
      oob_queue_->Enqueue(std::move(message), before_events);
    } else {
      //加入到OOB类型消息的队列中
      queue_->Enqueue(std::move(message), before_events);
    }
    if (paused_for_messages_) {
      ml.Notify();
    }
 
    //通过task_running_来防止短时间内多次重复执行
    if (pool_ != nullptr && !task_running_) {
      task_running_ = true;
      //交给线程池来异步执行（非RootIsolate）
      const bool launched_successfully = pool_->Run<MessageHandlerTask>(this);
    }
  }

  //调用自定义的消息通知
  MessageNotify(saved_priority);
}
复制代码
```

由于在`Flutter`的`RootIsolate`中，`pool_`为null。所以在非`RootIsolate`中，消息是通过线程池中的子线程来执行`RawReceivePort`对象创建时设置的回调方法——`_handleMessage`。

再来看`MessageNotify`函数，它的实现是在其子类`IsolateMessageHandler`中。

[->third_party/dart/runtime/vm/isolate.cc]

```
void IsolateMessageHandler::MessageNotify(Message::Priority priority) {
  if (priority >= Message::kOOBPriority) {
    //即使mutator线程繁忙，也要处理优先级为OOB的消息
    I->ScheduleInterrupts(Thread::kMessageInterrupt);
  }
  //获取Isolate的message_notify_callback_的值
  Dart_MessageNotifyCallback callback = I->message_notify_callback();
  if (callback != nullptr) {
    // Allow the embedder to handle message notification.
    (*callback)(Api::CastIsolate(I));
  }
}
复制代码
```

一般情况下，`callback`为null，但`RootIsolate`却例外。是因为在[Flutter的Engine启动](https://juejin.im/post/5e535ca2f265da573e6723f3)过程中，也就是在`RootIsolate`的`MessageHandler`初始化时，会给`callback`赋值。

[->third_party/tonic/dart_message_handler.cc]

```
void DartMessageHandler::Initialize(TaskDispatcher dispatcher) {
  //仅能调用一次
  task_dispatcher_ = dispatcher;
  //给RootIsolate的message_notify_callback_赋值
  Dart_SetMessageNotifyCallback(MessageNotifyCallback);
}
复制代码
```

也就是当调用`callback`时，对应的是`MessageNotifyCallback`函数的执行。

[->third_party/tonic/dart_message_handler.cc]

```
void DartMessageHandler::OnMessage(DartState* dart_state) {
  auto task_dispatcher_ = dart_state->message_handler().task_dispatcher_;

  auto weak_dart_state = dart_state->GetWeakPtr();
  //在Android中，任务交给UI线程中的loop来执行。
  //在iOS中，也是通过类似loop的消息处理器来执行
  task_dispatcher_([weak_dart_state]() {
    if (auto dart_state = weak_dart_state.lock()) {
      dart_state->message_handler().OnHandleMessage(dart_state.get());
    }
  });
}

void DartMessageHandler::MessageNotifyCallback(Dart_Isolate dest_isolate) {
  auto dart_state = DartState::From(dest_isolate);
  //调用OnMessage函数
  dart_state->message_handler().OnMessage(dart_state);
}
复制代码
```

通过上面代码，可以发现在`Android`平台的`RootIsolate`中，消息的处理是通过UI线程中的`loop`来处理。从`Android`角度来看，就是通过`handler`来发送一个消息。

总而言之，`timer`中异步任务的处理主要分为以下两种情况。

1. 在非`RootIsolate`中，是通过线程池获取一个子线程来处理任务。
2. 在`RootIsolate`中，如果是`Android`平台，则通过UI线程中的`loop`来处理任务。如果是iOS平台，则通过UI线程中的类似`loop`的消息处理器来处理任务。

#### 2.2、定时任务的执行

通过对_ZERO_EVENT消息的处理来执行了`timer`中的异步任务。那么再来看定时任务的执行，该任务则是通过`event handler`来处理的。

在[Flutter之Dart虚拟机启动](https://juejin.im/post/5e5478d651882549274a5340)一文中说过，当Dart VM虚拟机启动时会创建一个名为`event handler`的子线程，并在该子线程中通过异步IO来实现任务的执行。根据平台不同，异步IO的实现方式不同。在`Android`中，是通过`Linux`的`epoll`来实现的；在`iOS`中，是通过`kqueue`来实现的。

把`timer`对象添加到[二叉堆](https://zh.wikipedia.org/wiki/二叉堆)之后，会根据唤醒时间来排序，如果当前`timer`对象的唤醒时间最短，则会通知`event handler`。这里的`VMLibraryHooks.eventHandlerSendData`是在`Isolate`初始化时赋值的，它对应着`_EventHandler._sendData`。

```
@patch
class _EventHandler {
  @patch
  static void _sendData(Object sender, SendPort sendPort, int data)
      native "EventHandler_SendData";

  static int _timerMillisecondClock()
      native "EventHandler_TimerMillisecondClock";
}
复制代码
```

[->third_party/dart/runtime/bin/eventhandler.cc]

```
void FUNCTION_NAME(EventHandler_SendData)(Dart_NativeArguments args) {
  // Get the id out of the send port. If the handle is not a send port
  // we will get an error and propagate that out.
  Dart_Handle handle = Dart_GetNativeArgument(args, 1);
  Dart_Port dart_port;
  //拿到SendPort
  handle = Dart_SendPortGetId(handle, &dart_port);
  ...
  
  Dart_Handle sender = Dart_GetNativeArgument(args, 0);
  intptr_t id;
  if (Dart_IsNull(sender)) {
    id = kTimerId;
  } else {...}
  //拿到唤醒时间
  int64_t data = DartUtils::GetIntegerValue(Dart_GetNativeArgument(args, 2));
  //发送消息
  event_handler->SendData(id, dart_port, data);
}
复制代码
```

由于`event_handler`在不同系统有不同实现，所以这里以Android为例。

[->third_party/dart/runtime/bin/eventhandler_android.cc]

```
//发送数据
void EventHandlerImplementation::SendData(intptr_t id,
                                          Dart_Port dart_port,
                                          int64_t data) {
  WakeupHandler(id, dart_port, data);
}

void EventHandlerImplementation::WakeupHandler(intptr_t id,
                                               Dart_Port dart_port,
                                               int64_t data) {
  InterruptMessage msg;
  //消息id
  msg.id = id;
  //传递的dart_port
  msg.dart_port = dart_port;
  //消息需要传递的数据，在当前传递的是任务的唤醒时间
  msg.data = data;
  // WriteToBlocking will write up to 512 bytes atomically, and since our msg
  // is smaller than 512, we don't need a thread lock.
  // See: http://linux.die.net/man/7/pipe, section 'Pipe_buf'.
  ASSERT(kInterruptMessageSize < PIPE_BUF);
  //消息写入
  intptr_t result =
      FDUtils::WriteToBlocking(interrupt_fds_[1], &msg, kInterruptMessageSize);
  if (result != kInterruptMessageSize) {
    if (result == -1) {
      perror("Interrupt message failure:");
    }
    FATAL1("Interrupt message failure. Wrote %" Pd " bytes.", result);
  }
}

//处理拿到的事件
void EventHandlerImplementation::HandleEvents(struct epoll_event* events,
                                              int size) {
  bool interrupt_seen = false;
  for (int i = 0; i < size; i++) {
    if (events[i].data.ptr == NULL) {
      interrupt_seen = true;
    } else {
      DescriptorInfo* di =
          reinterpret_cast<DescriptorInfo*>(events[i].data.ptr);
      const intptr_t old_mask = di->Mask();
      const intptr_t event_mask = GetPollEvents(events[i].events, di);
      if ((event_mask & (1 << kErrorEvent)) != 0) {
        di->NotifyAllDartPorts(event_mask);
        UpdateEpollInstance(old_mask, di);
      } else if (event_mask != 0) {
        Dart_Port port = di->NextNotifyDartPort(event_mask);
        UpdateEpollInstance(old_mask, di);
        //通过消息的dart_port来调用注册的回调方法
        DartUtils::PostInt32(port, event_mask);
      }
    }
  }
  if (interrupt_seen) {
    // Handle after socket events, so we avoid closing a socket before we handle
    // the current events.
    HandleInterruptFd();
  }
}
复制代码
```

通过`epoll`就能在指定的时间来处理事件，然后通过`dart_port`来找到对应的`MessageHandler`并处理。

[->third_party/dart/runtime/bin/dartutils.cc]

```
bool DartUtils::PostInt32(Dart_Port port_id, int32_t value) {
  // Post a message with the integer value.
  int32_t min = 0xc0000000;  // -1073741824
  int32_t max = 0x3fffffff;  // 1073741823
  ASSERT(min <= value && value < max);
  Dart_CObject object;
  object.type = Dart_CObject_kInt32;
  object.value.as_int32 = value;
  return Dart_PostCObject(port_id, &object);
}
复制代码
```

[->third_party/dart/runtime/vm/native_api_impl.cc]

```
DART_EXPORT bool Dart_PostCObject(Dart_Port port_id, Dart_CObject* message) {
  return PostCObjectHelper(port_id, message);
}

static bool PostCObjectHelper(Dart_Port port_id, Dart_CObject* message) {
  ApiMessageWriter writer;
  std::unique_ptr<Message> msg =
      writer.WriteCMessage(message, port_id, Message::kNormalPriority);

  if (msg == nullptr) {
    return false;
  }

  // Post the message at the given port.
  return PortMap::PostMessage(std::move(msg));
}
复制代码
```

最终也就跟_ZERO_EVENT消息的处理流程一样，在当前`isolate`的`MessageHandler`中调用创建`RawReceivePort`对象时设置的回调方法——`_handleMessage`。

#### 2.3、回调方法的执行

无论是异步任务，还是需要延时执行的任务，最终执行的回调方法都是`_handleMessage`。在该回调方法中，会根据消息类型来进行不同的区分，如果消息类型是`_ZERO_EVENT`，则会从`_queueFromZeroEvent`取出对应的`Timer`对象并执行其回调方法；否则就从`_queueFromTimeoutEvent`中取出对应`timer`对象并执行其回调方法。

先来看`_queueFromZeroEvent`方法。

```
  static List _queueFromZeroEvent() {
    var pendingTimers = new List();
    
    //从二叉堆中查询到期时间小于_firstZeroTimer的timer，并加入到一个List中
    var timer;
    while (!_heap.isEmpty && (_heap.first._compareTo(_firstZeroTimer) < 0)) {
      timer = _heap.removeFirst();
      pendingTimers.add(timer);
    }
    //获取链表中的第一个timer
    timer = _firstZeroTimer;
    _firstZeroTimer = timer._indexOrNext;
    timer._indexOrNext = null;
    pendingTimers.add(timer);
    return pendingTimers;
  }
复制代码
```

在该方法中，会将[二叉堆](https://zh.wikipedia.org/wiki/二叉堆)中唤醒时间比链表中的第一个`timer`对象唤醒时间还短的`timer`对象加入到集合`pendingTimers`中，然后再将链表中的第一个`timer`对象加入到集合`pendingTimers`中。

再来看`_queueFromTimeoutEvent`方法。

```
  static List _queueFromTimeoutEvent() {
    var pendingTimers = new List();
    if (_firstZeroTimer != null) {
      //从二叉堆中获取唤醒时间小于链表中第一个timer唤醒时间的timer，并将该timer添加到pendingTimers中
      var timer;
      while (!_heap.isEmpty && (_heap.first._compareTo(_firstZeroTimer) < 0)) {
        timer = _heap.removeFirst();
        pendingTimers.add(timer);
      }
    } else {
      //从二叉堆中获取已经到期的timer并添加到pendingTimers中
      var currentTime = VMLibraryHooks.timerMillisecondClock();
      var timer;
      while (!_heap.isEmpty && (_heap.first._wakeupTime <= currentTime)) {
        timer = _heap.removeFirst();
        pendingTimers.add(timer);
      }
    }
    return pendingTimers;
  }
复制代码
```

在该方法中，也会将[二叉堆](https://zh.wikipedia.org/wiki/二叉堆)中唤醒时间比链表中的第一个`timer`对象唤醒时间还短的`timer`对象加入到集合`pendingTimers`中。如果此时链表的第一个`timer`对象为空，则会将[二叉堆](https://zh.wikipedia.org/wiki/二叉堆)中`Timer`对象的唤醒时间与当前时间进行对比，如果唤醒时间小于当前当前时间，则将`timer`添加到集合`pendingTimers`中。

经过`_queueFromZeroEvent`与`_queueFromTimeoutEvent`两个方法，就获取到了所有待执行的`timer`对象。然后调用`_runTimers`方法来执行所有待执行的`timer`对象。待`_runTimers`方法执行完毕后，还会调用`_notifyEventHandler`来通知`event handler`或者关闭`port`。

再来看`_runTimers`方法的实现。

```
  static void _runTimers(List pendingTimers) {
    //如果目前没有待处理的timer，那么就有机会在新加入timer之前来重置_idCount
    if (_heap.isEmpty && (_firstZeroTimer == null)) {
      _idCount = 0;
    }

    //如果没有待处理的timer，则结束方法的执行
    if (pendingTimers.length == 0) {
      return;
    }

    // Trigger all of the pending timers. New timers added as part of the
    // callbacks will be enqueued now and notified in the next spin at the
    // earliest.
    _handlingCallbacks = true;
    var i = 0;
    try {
      for (; i < pendingTimers.length; i++) {
        //获取下一个timer
        var timer = pendingTimers[i];
        timer._indexOrNext = null;

        // One of the timers in the pending_timers list can cancel
        // one of the later timers which will set the callback to
        // null. Or the pending zero timer has been canceled earlier.
        if (timer._callback != null) {
          var callback = timer._callback;
          if (!timer._repeating) {
            //将timer标记为无效
            timer._callback = null;
          } else if (timer._milliSeconds > 0) {
            var ms = timer._milliSeconds;
            int overdue =
                VMLibraryHooks.timerMillisecondClock() - timer._wakeupTime;
            if (overdue > ms) {
              int missedTicks = overdue ~/ ms;
              timer._wakeupTime += missedTicks * ms;
              timer._tick += missedTicks;
            }
          }
          timer._tick += 1;
          
          //执行timer中注册的回调方法
          callback(timer);
          // Re-insert repeating timer if not canceled.
          //如果timer未取消，则重新插入链表或者二叉堆中
          if (timer._repeating && (timer._callback != null)) {
            //更新唤醒时间
            timer._advanceWakeupTime();
            timer._enqueue();
          }
          //执行微任务，仅限于非RootIsolate。
          var immediateCallback = _removePendingImmediateCallback();
          if (immediateCallback != null) {
            immediateCallback();
          }
        }
      }
    } finally {
      _handlingCallbacks = false;
      //重新向二叉堆或者链表中插入pendingTimers中还存在的timer
      for (i++; i < pendingTimers.length; i++) {
        var timer = pendingTimers[i];
        timer._enqueue();
      }
      _notifyEventHandler();
    }
  }
复制代码
```

在上面代码中，主要就是遍历`pendingTimers`中的`Timer`对象，获取`Timer`中的任务`callback`并执行，在上面示例中，就是输出f1、f2及f3。如果是周期性任务，则会在`callback`执行完毕后更新唤醒时间并重新添加到链表或[二叉堆](https://zh.wikipedia.org/wiki/二叉堆)中。如果在非`RootIsolate`中，还会执行微任务。如果最终`pendingTimers`中还存在未遍历的`Timer`，则将这些`Timer`添加到链表或[二叉堆](https://zh.wikipedia.org/wiki/二叉堆)中并通知`event handler`。

### 3、总结

经过上面的分析，全面了解了`Timer`的使用及其实现原理。它的使用很简单，实现原理也分为以下几点。

1. 如果是异步任务，则通过`isolate`中的`MessageHandler`来处理。使用方式是调用`Timer`的`run`方法。
2. 如果是定时任务或周期性任务，则通过`event handler`来处理并通过`isolate`中的`MessageHandler`来执行任务。使用方式是通过工厂模式创建`Timer`或者调用`Timer`的`periodic`方法。
3. 如果在非`RootIsolate`中，`Timer`的任务执行完毕后都会执行微任务。

由于`Future`的异步机制是通过`Timer`来实现的，所以了解了`Timer`的实现原理，也就知道了`Future`的异步部分的实现原理。

