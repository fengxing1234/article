---
title: Fluter消息机制之微任务
date: 2020-06-29 17:13:29
tags:
---

# Fluter消息机制之微任务

在`Flutter`中，异步任务主要是通过`Timer`及微任务来实现。在[Flutter之Timer原理解析](https://juejin.im/post/5ee19d11518825433b134766)一文中，讲述了通过`Timer`来实现异步任务的原理，那么本文就来看异步任务的另一种实现，微任务的使用及其实现原理。

### 1、微任务的使用

先来看微任务的使用，代码很简单，如下。

```
//用法一
Future.microtask(() {
  print("microtask1");
});
//用法二
scheduleMicrotask(() {
  print("microtask2");
});
//用法三
Zone.current.scheduleMicrotask((){
  print("microtask3");
});
//用法四
Zone.root.scheduleMicrotask((){
  print("microtask4");
});
复制代码
```

以上就是微任务的所有用法。基本上都是前两种使用方式比较多，但前面两种用法仅是对后面两种用法的封装而已。下面就来看微任务的实现原理，不过在分析微任务的实现原理之前需要先了解一下UI线程是如何创建的。

### 2、UI线程的创建

在[Flutter之Engine启动流程](https://juejin.im/post/5e535ca2f265da573e6723f3)一文中，提过在`Engine`创建过程中会创建UI线程、IO线程及GPU线程，但未深入。所以这里就以Android平台为例来深入的来了解`Flutter`中UI线程是如何创建的（IO线程、GPU线程与UI线程都是同一类型的对象，但命名不同）。

[-> flutter/shell/platform/android/android_shell_holder.cc]

```
AndroidShellHolder::AndroidShellHolder(
    flutter::Settings settings,
    fml::jni::JavaObjectWeakGlobalRef java_object,
    bool is_background_view)
    : settings_(std::move(settings)), java_object_(java_object) {
  static size_t shell_count = 1;
  auto thread_label = std::to_string(shell_count++);
            
  //创建目标线程
  if (is_background_view) {
    //仅创建UI线程
    thread_host_ = {thread_label, ThreadHost::Type::UI};
  } else {
    //创建UI线程、GPU线程及IO线程
    thread_host_ = {thread_label, ThreadHost::Type::UI | ThreadHost::Type::GPU |
                                      ThreadHost::Type::IO};
  }
  ...
}
复制代码
```

这里的`thread_host_`是一个结构体，所以直接来看该结构体中的具体实现。

[-> flutter/shell/common/thread_host.cc]

```
#include "flutter/shell/common/thread_host.h"

namespace flutter {

...

ThreadHost::ThreadHost(std::string name_prefix, uint64_t mask) {
  if (mask & ThreadHost::Type::Platform) {
    //Platform线程的创建，在Android中，由于Platform线程是Android中的主线程，所以名称为xxxx.platform的platform_thread不会创建
    platform_thread = std::make_unique<fml::Thread>(name_prefix + ".platform");
  }

  if (mask & ThreadHost::Type::UI) {
    //ui线程的创建
    ui_thread = std::make_unique<fml::Thread>(name_prefix + ".ui");
  }

  if (mask & ThreadHost::Type::GPU) {
    //gpu线程的创建
    gpu_thread = std::make_unique<fml::Thread>(name_prefix + ".gpu");
  }

  if (mask & ThreadHost::Type::IO) {
    //io线程的创建
    io_thread = std::make_unique<fml::Thread>(name_prefix + ".io");
  }
}

...

} 
复制代码
```

从上面就可以看出ui线程、io线程及GPU线程是同一类型的对象，但命名不同。那么再来看UI线程的具体实现。

[-> flutter/fml/thread.cc]

```
Thread::Thread(const std::string& name) : joined_(false) {
  fml::AutoResetWaitableEvent latch;
  fml::RefPtr<fml::TaskRunner> runner;
  //创建一个thread对象
  thread_ = std::make_unique<std::thread>([&latch, &runner, name]() -> void {
    //设置当前线程名称，由于这里是UI线程的创建，所以name是xxx.ui
    SetCurrentThreadName(name);
    //创建一个MessageLoop对象
    fml::MessageLoop::EnsureInitializedForCurrentThread();
    //获取MessageLoop对应的loop
    auto& loop = MessageLoop::GetCurrent();
    runner = loop.GetTaskRunner();
    //唤醒
    latch.Signal();
    //运行loop
    loop.Run();
  });
  //等待
  latch.Wait();
  task_runner_ = runner;
}
复制代码
```

在`Thread`的构造函数中，会创建一个新线程。在该线程创建成功后，会给线程设置名称，如xxxxx.ui、xxxxx.gpu、xxxxx.io等。还会给该线程设置一个`MessageLoop`对象，最后再来执行`MessageLoop`的`run`函数，即使`MessageLoop`跑起来。

这里重点来看`MessageLoop`对象的创建，它的实现如下。

[-> flutter/fml/message_loop.cc]

```
//tls_message_loop类似Java中的ThreadLocal，用来保证MessageLoop仅属于某个线程，其他线程不可访问该MessageLoop
FML_THREAD_LOCAL ThreadLocalUniquePtr<MessageLoop> tls_message_loop;

void MessageLoop::EnsureInitializedForCurrentThread() {
  //保证每个进行仅有一个MessageLoop对象
  if (tls_message_loop.get() != nullptr) {
    // Already initialized.
    return;
  }
  tls_message_loop.reset(new MessageLoop());
}

//创建MessageLoop对象
MessageLoop::MessageLoop()
      //MessageLoopImpl对象的创建
    : loop_(MessageLoopImpl::Create()),
      //TaskRunner对象的创建
      task_runner_(fml::MakeRefCounted<fml::TaskRunner>(loop_)) {}
复制代码
```

这里重点在`loop_`。它是一个`MessageLoopImpl`对象，由于各个平台不同，所以`MessageLoopImpl`的具体实现也不一样。这里以Android为例，当调用`create`方法时会创建一个继承自`MessageLoopImpl`的`MessageLoopAndroid`对象。

[-> flutter/fml/platform/android/message_loop_android.cc]

```
static constexpr int kClockType = CLOCK_MONOTONIC;

static ALooper* AcquireLooperForThread() {
  ALooper* looper = ALooper_forThread();

  if (looper == nullptr) {
    //如果当前线程不存在looper，则创建一个新的looper
    looper = ALooper_prepare(0);
  }

  //如果当前线程存在looper，则获取其引用并返回
  ALooper_acquire(looper);
  return looper;
}

//构造函数
MessageLoopAndroid::MessageLoopAndroid()
      //创建一个looper对象
    : looper_(AcquireLooperForThread()),
      timer_fd_(::timerfd_create(kClockType, TFD_NONBLOCK | TFD_CLOEXEC)),
      running_(false) {

  static const int kWakeEvents = ALOOPER_EVENT_INPUT;
  
  //执行回调方法
  ALooper_callbackFunc read_event_fd = [](int, int events, void* data) -> int {
    if (events & kWakeEvents) {
      reinterpret_cast<MessageLoopAndroid*>(data)->OnEventFired();
    }
    return 1;  // continue receiving callbacks
  };

  int add_result = ::ALooper_addFd(looper_.get(),          // looper
                                   timer_fd_.get(),        // fd
                                   ALOOPER_POLL_CALLBACK,  // ident
                                   kWakeEvents,            // events
                                   read_event_fd,          // callback
                                   this                    // baton
  );
}

MessageLoopAndroid::~MessageLoopAndroid() {
  int remove_result = ::ALooper_removeFd(looper_.get(), timer_fd_.get());
  FML_CHECK(remove_result == 1);
}

//looper的运行
void MessageLoopAndroid::Run() {
  FML_DCHECK(looper_.get() == ALooper_forThread());

  running_ = true;

  while (running_) {
    //等待事件执行
    int result = ::ALooper_pollOnce(-1,       // infinite timeout
                                    nullptr,  // out fd,
                                    nullptr,  // out events,
                                    nullptr   // out data
    );
    if (result == ALOOPER_POLL_TIMEOUT || result == ALOOPER_POLL_ERROR) {
      // This handles the case where the loop is terminated using ALooper APIs.
      running_ = false;
    }
  }
}

//终止事件执行
void MessageLoopAndroid::Terminate() {
  running_ = false;
  ALooper_wake(looper_.get());
}

//唤醒事件的执行
void MessageLoopAndroid::WakeUp(fml::TimePoint time_point) {
  bool result = TimerRearm(timer_fd_.get(), time_point);
  FML_DCHECK(result);
}

//监听Loop的回调函数
void MessageLoopAndroid::OnEventFired() {
  if (TimerDrain(timer_fd_.get())) {
    RunExpiredTasksNow();
  }
}
复制代码
```

上面代码其实就是通过`ALooper`来实现一个异步IO。在Android中，`ALooper`可以认为是一个对`Looper`的包装，也就是通过`ALooper`来操作`Looper`。

**注意**：这里所说的`ALooper`来操作`Looper`指的是Native层中的`Looper`，而不是framework层的`Looper`。

当`MessageLoopAndroid`对象创建成功后，再调用该对象的`run`函数使UI线程中的任务处理跑起来。这时候UI线程就成功创建完毕并做了相应的初始化。

以上就是在`Android`平台中UI线程的创建，而在其他平台，UI线程的创建也与Android平台类似，唯一的不同之处就在于异步IO的实现。比如在iOS中，异步IO是采用`CFRunLoop`来实现的。

### 3、微任务实现原理

再回到微任务的实现中。以`scheduleMicrotask`方法为例，来看其代码实现。

```
void scheduleMicrotask(void callback()) {
  _Zone currentZone = Zone.current;
  //当前Zone与_rootZone是否是同一个Zone对象。
  if (identical(_rootZone, currentZone)) {
    // No need to bind the callback. We know that the root's scheduleMicrotask
    // will be invoked in the root zone.
    _rootScheduleMicrotask(null, null, _rootZone, callback);
    return;
  }
  ...
}
复制代码
```

基本上自定义`Zone`都不会来自定义`scheduleMicrotask`方法的实现，所以自定义`Zone`的`scheduleMicrotask`方法最终都是调用`_rootScheduleMicrotask`方法。下面就来看该方法的实现。

```
void _rootScheduleMicrotask(
    Zone self, ZoneDelegate parent, Zone zone, void f()) {
  ...
  _scheduleAsyncCallback(f);
}
复制代码
```

上面代码很简单，就是调用`_scheduleAsyncCallback`方法，再来看该方法的实现。

```
//节点为_AsyncCallbackEntry对象的单链表的头节点
_AsyncCallbackEntry _nextCallback;
//节点为_AsyncCallbackEntry对象的单链表的尾节点
_AsyncCallbackEntry _lastCallback;

 //优先级回调方法放在链表的头部，如果存在多个，则按照添加顺序排列
_AsyncCallbackEntry _lastPriorityCallback;

//当前是否在执行回调方法
bool _isInCallbackLoop = false;

//遍历链表并执行相应的回调方法
void _microtaskLoop() {
  while (_nextCallback != null) {
    _lastPriorityCallback = null;
    _AsyncCallbackEntry entry = _nextCallback;
    _nextCallback = entry.next;
    if (_nextCallback == null) _lastCallback = null;
    (entry.callback)();
  }
}

//开始执行回调方法
void _startMicrotaskLoop() {
  _isInCallbackLoop = true;
  try {
    // Moved to separate function because try-finally prevents
    // good optimization.
    _microtaskLoop();
  } finally {
    _lastPriorityCallback = null;
    _isInCallbackLoop = false;
    if (_nextCallback != null) {
      _AsyncRun._scheduleImmediate(_startMicrotaskLoop);
    }
  }
}

//将回调方法添加到链表中
void _scheduleAsyncCallback(_AsyncCallback callback) {
  _AsyncCallbackEntry newEntry = new _AsyncCallbackEntry(callback);
  if (_nextCallback == null) {
    _nextCallback = _lastCallback = newEntry;
    //如果当前还未处理回调方法
    if (!_isInCallbackLoop) {
      _AsyncRun._scheduleImmediate(_startMicrotaskLoop);
    }
  } else {
    _lastCallback.next = newEntry;
    _lastCallback = newEntry;
  }
}

void _schedulePriorityAsyncCallback(_AsyncCallback callback) {
  if (_nextCallback == null) {
    _scheduleAsyncCallback(callback);
    _lastPriorityCallback = _lastCallback;
    return;
  }
  _AsyncCallbackEntry entry = new _AsyncCallbackEntry(callback);
  if (_lastPriorityCallback == null) {
    entry.next = _nextCallback;
    _nextCallback = _lastPriorityCallback = entry;
  } else {
    entry.next = _lastPriorityCallback.next;
    _lastPriorityCallback.next = entry;
    _lastPriorityCallback = entry;
    if (entry.next == null) {
      _lastCallback = entry;
    }
  }
}
复制代码
```

上面代码很简单，在`_scheduleAsyncCallback`方法中，就是将要执行的微任务包装成一个`_AsyncCallbackEntry`对象，并将该对象添加到链表中，也就是所有微任务都是链表中的一个节点，当遍历该链表并执行节点中的回调方法即是微任务的执行。

这里要注意一下`_schedulePriorityAsyncCallback`方法，它也是将微任务包装成一个`_AsyncCallbackEntry`对象并添加到链表中。但这里的微任务优先级高，是直接将微任务添加到链表的头部。目前仅有当前`Zone`对象的`handleUncaughtError`方法中才会调用`_schedulePriorityAsyncCallback`，也就是捕获错误的优先级比普通微任务的优先级都要高。

链表创建成功后，就需要能够在合适的时机来遍历该链表，这时候就来看`_scheduleImmediate`方法的执行。

```
class _AsyncRun {
  //这里的callback对应的就是_startMicrotaskLoop方法
  external static void _scheduleImmediate(void callback());
}
复制代码
```

#### 3.1、微任务集合

来看`_scheduleImmediate`方法的实现，实现代码很简单，如下。

```
@patch
class _AsyncRun {
  @patch
  static void _scheduleImmediate(void callback()) {
    if (_ScheduleImmediate._closure == null) {
      throw new UnsupportedError("Microtasks are not supported");
    }
    _ScheduleImmediate._closure(callback);
  }
}

typedef void _ScheduleImmediateClosure(void callback());

class _ScheduleImmediate {
  static _ScheduleImmediateClosure _closure;
}

//在Engine初始化时调用
@pragma("vm:entry-point", "call")
void _setScheduleImmediateClosure(_ScheduleImmediateClosure closure) {
  _ScheduleImmediate._closure = closure;
}

@pragma("vm:entry-point", "call")
void _ensureScheduleImmediate() {
  _AsyncRun._scheduleImmediate(_startMicrotaskLoop);
}
复制代码
```

由于`_setScheduleImmediateClosure`是在`RootIsolate`创建成功后的`InitDartAsync`函数中调用的，所以来看`InitDartAsync`函数的实现。

[-> flutter/lib/ui/dart_runtime_hooks.cc]

```
static void InitDartAsync(Dart_Handle builtin_library, bool is_ui_isolate) {
  Dart_Handle schedule_microtask;
  if (is_ui_isolate) {
    schedule_microtask =
        GetFunction(builtin_library, "_getScheduleMicrotaskClosure");
  } else {
    Dart_Handle isolate_lib = Dart_LookupLibrary(ToDart("dart:isolate"));
    Dart_Handle method_name =
        Dart_NewStringFromCString("_getIsolateScheduleImmediateClosure");
    schedule_microtask = Dart_Invoke(isolate_lib, method_name, 0, NULL);
  }
  Dart_Handle async_library = Dart_LookupLibrary(ToDart("dart:async"));
  Dart_Handle set_schedule_microtask = ToDart("_setScheduleImmediateClosure");
  Dart_Handle result = Dart_Invoke(async_library, set_schedule_microtask, 1,
                                   &schedule_microtask);
  PropagateIfError(result);
}
复制代码
```

代码很简单，根据不同`isolate`有不同实现。先来看ui_isolate，也就是`RootIsolate`，在`RootIsolate`中，赋给`_ScheduleImmediate._closure`的值是`_getScheduleMicrotaskClosure`方法，所以来看该方法的实现。

```
void _scheduleMicrotask(void callback()) native 'ScheduleMicrotask';

@pragma('vm:entry-point')
Function _getScheduleMicrotaskClosure() => _scheduleMicrotask; 
复制代码
```

上面代码很简单，就是对应着`ScheduleMicrotask`函数。

[-> flutter/lib/ui/dart_runtime_hooks.cc]

```
void ScheduleMicrotask(Dart_NativeArguments args) {
  Dart_Handle closure = Dart_GetNativeArgument(args, 0);
  UIDartState::Current()->ScheduleMicrotask(closure);
}
复制代码
```

在`ScheduleMicrotask`函数中解析传递的参数，然后在调用当前`UIDartState`对象的`ScheduleMicrotask`函数。

[-> flutter/lib/ui/ui_dart_state.cc]

```
void UIDartState::ScheduleMicrotask(Dart_Handle closure) {
  if (tonic::LogIfError(closure) || !Dart_IsClosure(closure)) {
    return;
  }

  microtask_queue_.ScheduleMicrotask(closure);
}
复制代码
```

在`UIDartState`对象的`ScheduleMicrotask`函数中，又会调用`DartMicrotaskQueue`对象的`ScheduleMicrotask`函数。

[-> tonic/dart_microtask_queue.cc]

```
void DartMicrotaskQueue::ScheduleMicrotask(Dart_Handle callback) {
  queue_.emplace_back(DartState::Current(), callback);
}
复制代码
```

最终来到`DartMicrotaskQueue`对象，在该对象中存在一个集合`queue_`，而`ScheduleMicrotask`函数中就是把`callback`添加到该集合中。也就是在`RootIsolate`中，最终是把`_startMicrotaskLoop`方法作为参数添加到集合`queue_`中。

再来看非`ui_isolate`中的处理情况，在非`ui_isolate`中，赋给`_ScheduleImmediate._closure`的值就变成了`_getIsolateScheduleImmediateClosure`方法。该方法的实现就简单多了，来看下面代码。

```
_ImmediateCallback _pendingImmediateCallback;

void _isolateScheduleImmediate(void callback()) {
  _pendingImmediateCallback = callback;
}

@pragma("vm:entry-point", "call")
Function _getIsolateScheduleImmediateClosure() {
  return _isolateScheduleImmediate;
}
复制代码
```

在上面代码中，仅是把赋给了`_pendingImmediateCallback`，也就是把`_startMicrotaskLoop`方法作为值赋给了`_pendingImmediateCallback`。

#### 3.2、微任务的执行

经过前面的准备，下面就可以在合适的时机来执行`_startMicrotaskLoop`方法，从而来处理所有微任务。

这里也分为`ui_isolate`及非`ui_isolate`两种情况，先来看当前`isolate`是`ui_isolate`的情况。

再来看`UIDartState`对象的实现，除了将`_startMicrotaskLoop`添加到集合中，也会在该对象中通过`FlushMicrotasksNow`函数来执行`_startMicrotaskLoop`方法，代码如下。

[-> flutter/lib/ui/ui_dart_state.cc]

```
void UIDartState::FlushMicrotasksNow() {
  microtask_queue_.RunMicrotasks();
}
复制代码
```

再来看`RunMicrotasks`的实现，代码如下。

[-> tonic/dart_microtask_queue.cc]

```
void DartMicrotaskQueue::RunMicrotasks() {
  while (!queue_.empty()) {
    MicrotaskQueue local;
    std::swap(queue_, local);
    //遍历集合中的所有元素
    for (const auto& callback : local) {
      if (auto dart_state = callback.dart_state().lock()) {
        DartState::Scope dart_scope(dart_state.get());
        //调用_startMicrotaskLoop方法，callback.value()对应是_startMicrotaskLoop
        Dart_Handle result = Dart_InvokeClosure(callback.value(), 0, nullptr);
        ...
      }
    }
  }
}
复制代码
```

至此，知道了在`ui_isolate`中，微任务是如何添加到集合中、如何执行的。那么再来想一个问题，微任务的执行时机是在什么时候尼？这就需要来看调用`FlushMicrotasksNow`函数的时机。

经过查看`Flutter`源码。可以发现，`Flutter`中仅在`Window`对象的`BeginFrame`函数及`UIDartState`对象的`AddOrRemoveTaskObserver`函数中调用了调用了`FlushMicrotasksNow`函数。因此先来看`BeginFrame`的实现。

[-> flutter/lib/ui/window/window.cc]

```
void Window::BeginFrame(fml::TimePoint frameTime) {
  std::shared_ptr<tonic::DartState> dart_state = library_.dart_state().lock();
  if (!dart_state)
    return;
  tonic::DartState::Scope scope(dart_state);

  int64_t microseconds = (frameTime - fml::TimePoint()).ToMicroseconds();
  //调用_beginFrame方法来开始绘制
  tonic::LogIfError(tonic::DartInvokeField(library_.value(), "_beginFrame",
                                           {
                                               Dart_NewInteger(microseconds),
                                           }));
  //执行所有微任务
  UIDartState::Current()->FlushMicrotasksNow();
  //调用_drawFrame来绘制UI
  tonic::LogIfError(tonic::DartInvokeField(library_.value(), "_drawFrame", {}));
}
复制代码
```

代码很简单，但也说明了，在`Flutter`中的`window`调用`_beginFrame`与`_drawFrame`方法之间会把所有微任务处理掉。也就注定了不能在微任务中做耗时操作，否则影响UI的绘制。

再来看`AddOrRemoveTaskObserver`函数。

[-> flutter/lib/ui/ui_dart_state.cc]

```
UIDartState::UIDartState(
    TaskRunners task_runners,
    TaskObserverAdd add_callback,
    TaskObserverRemove remove_callback,
    fml::WeakPtr<SnapshotDelegate> snapshot_delegate,
    fml::WeakPtr<IOManager> io_manager,
    fml::RefPtr<SkiaUnrefQueue> skia_unref_queue,
    fml::WeakPtr<ImageDecoder> image_decoder,
    std::string advisory_script_uri,
    std::string advisory_script_entrypoint,
    std::string logger_prefix,
    UnhandledExceptionCallback unhandled_exception_callback,
    std::shared_ptr<IsolateNameServer> isolate_name_server)
    : task_runners_(std::move(task_runners)),
      //给add_callback_赋值
      add_callback_(std::move(add_callback)),
      ...
      isolate_name_server_(std::move(isolate_name_server)) {
  AddOrRemoveTaskObserver(true /* add */);
}

void UIDartState::AddOrRemoveTaskObserver(bool add) {
  auto task_runner = task_runners_.GetUITaskRunner();
  if (!task_runner) {
    // This may happen in case the isolate has no thread affinity (for example,
    // the service isolate).
    return;
  }
  FML_DCHECK(add_callback_ && remove_callback_);
  if (add) {
    //这里是一个lambda表达式，传递给add_callback_一个函数
    add_callback_(reinterpret_cast<intptr_t>(this),
                  [this]() { this->FlushMicrotasksNow(); });//执行所有微任务
  } else {
    remove_callback_(reinterpret_cast<intptr_t>(this));
  }
}
复制代码
```

这里重点来看`add_callback_`，它是在`UIDartState`对象初始化的时候赋值的。由于`DartIsolate`继承自`UIDartState`，所以来看`DartIsolate`对象的创建。

[-> flutter/runtime/dart_isolate.cc]

```
DartIsolate::DartIsolate(const Settings& settings,
                         TaskRunners task_runners,
                         fml::WeakPtr<SnapshotDelegate> snapshot_delegate,
                         fml::WeakPtr<IOManager> io_manager,
                         fml::RefPtr<SkiaUnrefQueue> unref_queue,
                         fml::WeakPtr<ImageDecoder> image_decoder,
                         std::string advisory_script_uri,
                         std::string advisory_script_entrypoint,
                         bool is_root_isolate)
    : UIDartState(std::move(task_runners),
                  //add_callback_对应的值
                  settings.task_observer_add,
                  settings.task_observer_remove,
                  std::move(snapshot_delegate),
                  std::move(io_manager),
                  std::move(unref_queue),
                  std::move(image_decoder),
                  advisory_script_uri,
                  advisory_script_entrypoint,
                  settings.log_tag,
                  settings.unhandled_exception_callback,
                  DartVMRef::GetIsolateNameServer()),
      is_root_isolate_(is_root_isolate) {
  phase_ = Phase::Uninitialized;
}
复制代码
```

在上面代码中，把`settings`对象的`task_observer_add`赋给了`add_callback_`。而`settings`是在`FlutterMain`的`Init`函数中创建并初始化的，所以在`FlutterMain`初始化时，就会给`settings`对象的`task_observer_add`赋值。

[-> flutter/shell/platform/android/flutter_main.cc]

```
void FlutterMain::Init(JNIEnv* env,
                       jclass clazz,
                       jobject context,
                       jobjectArray jargs,
                       jstring kernelPath,
                       jstring appStoragePath,
                       jstring engineCachesPath) {
  std::vector<std::string> args;
  args.push_back("flutter");
  for (auto& arg : fml::jni::StringArrayToVector(env, jargs)) {
    args.push_back(std::move(arg));
  }
  auto command_line = fml::CommandLineFromIterators(args.begin(), args.end());

  auto settings = SettingsFromCommandLine(command_line);

  ...
  
  //add_callback_对应的函数
  settings.task_observer_add = [](intptr_t key, fml::closure callback) {
    fml::MessageLoop::GetCurrent().AddTaskObserver(key, std::move(callback));
  };

  settings.task_observer_remove = [](intptr_t key) {
    fml::MessageLoop::GetCurrent().RemoveTaskObserver(key);
  };
  ...
}
复制代码
```

再来看线程所对应的`MessageLoopImpl`对象，前面说过`MessageLoopAndroid`继承自`MessageLoopImpl`。所以来看`AddTaskObserver`函数的实现。

[-> flutter/fml/message_loop_impl.cc]

```
void MessageLoopImpl::AddTaskObserver(intptr_t key,
                                      const fml::closure& callback) {
  if (callback != nullptr) {
    //每个MessageLoopImpl对象拥有唯一的queue_id_
    task_queue_->AddTaskObserver(queue_id_, key, callback);
  } else {...}
}
复制代码
```

这里的`task_queue_`是一个`MessageLoopTaskQueues`对象，它的`AddTaskObserver`函数实现如下。

[-> flutter/fml/message_loop_task_queues.cc]

```
void MessageLoopTaskQueues::AddTaskObserver(TaskQueueId queue_id,
                                            intptr_t key,
                                            const fml::closure& callback) {
  std::scoped_lock queue_lock(GetMutex(queue_id));
  //UIDartState为key。包含FlushMicrotasksNow函数调用的callback为value
  queue_entries_[queue_id]->task_observers[key] = std::move(callback);
}
复制代码
```

代码中的`queue_entries_`是一个以`TaskQueueId`为key、`TaskQueueEntry`对象为value的map，而`TaskQueueEntry`中的`task_observers`也是一个map。所以`AddTaskObserver`函数就是把包含`FlushMicrotasksNow`函数调用的`callback`以`UIDartState`对象为key存入map中。

在[Flutter之Timer原理解析](https://juejin.im/post/5ee19d11518825433b134766)一文中，如果在`ui_isolate`中，最终是通过`DartMessageHandler`的`OnMessage`函数来处理`event handler`消息及普通消息，代码如下。

[->third_party/tonic/dart_message_handler.cc]

```
void DartMessageHandler::Initialize(TaskDispatcher dispatcher) {
  // Only can be called once.
  TONIC_CHECK(!task_dispatcher_ && dispatcher);
  task_dispatcher_ = dispatcher;
  Dart_SetMessageNotifyCallback(MessageNotifyCallback);
}

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
复制代码
```

代码中的`task_dispatcher_`是在`DartMessageHandler`对象调用`Initialize`函数时设置的。根据[Flutter之Engine启动流程](https://juejin.im/post/5e535ca2f265da573e6723f3)，知道是在`Initialize`函数是在`RootIsolate`初始化时调用的，那么就来看一下`Initialize`函数的实现。

[-> flutter/runtime/dart_isolate.cc]

```
bool DartIsolate::InitializeIsolate(
    std::shared_ptr<DartIsolate> embedder_isolate,
    Dart_Isolate isolate,
    char** error) {
  ...
  //设置UI线程的消息处理器
  SetMessageHandlingTaskRunner(GetTaskRunners().GetUITaskRunner());
  ...
  return true;
}

void DartIsolate::SetMessageHandlingTaskRunner(
    fml::RefPtr<fml::TaskRunner> runner) {
  if (!IsRootIsolate() || !runner) {
    return;
  }

  message_handling_task_runner_ = runner;
  
  //设置消息处理器
  message_handler().Initialize(
      [runner](std::function<void()> task) { runner->PostTask(task); });
}
复制代码
```

根据上面代码，可以知道`task_dispatcher_`中其实就是将任务task通过`PostTask`函数添加到`looper`中。

[-> flutter/fml/task_runner.cc]

```
TaskRunner::TaskRunner(fml::RefPtr<MessageLoopImpl> loop)
    : loop_(std::move(loop)) {}
void TaskRunner::PostTask(const fml::closure& task) {
  loop_->PostTask(task, fml::TimePoint::Now());
}
复制代码
```

以Android平台为例，这里的`loop_`就是一个`MessageLoopAndroid`对象。所以再来看`MessageLoopAndroid`中`PostTask`的实现。

[-> flutter/fml/message_loop_impl.cc]

```
void MessageLoopImpl::PostTask(const fml::closure& task,
                               fml::TimePoint target_time) {
  ...
  task_queue_->RegisterTask(queue_id_, task, target_time);
}
复制代码
```

在`PostTask`函数中最终还是调用的`task_queue_`的`RegisterTask`函数，再来看该函数的实现。

[-> flutter/fml/message_loop_task_queues.cc]

```
void MessageLoopTaskQueues::RegisterTask(TaskQueueId queue_id,
                                         const fml::closure& task,
                                         fml::TimePoint target_time) {
  std::scoped_lock queue_lock(GetMutex(queue_id));

  size_t order = order_++;
  const auto& queue_entry = queue_entries_[queue_id];
  queue_entry->delayed_tasks.push({order, task, target_time});
  TaskQueueId loop_to_wake = queue_id;
  if (queue_entry->subsumed_by != _kUnmerged) {
    loop_to_wake = queue_entry->subsumed_by;
  }
  WakeUpUnlocked(loop_to_wake,
                 queue_entry->delayed_tasks.top().GetTargetTime());
}

void MessageLoopTaskQueues::WakeUpUnlocked(TaskQueueId queue_id,
                                           fml::TimePoint time) const {
  if (queue_entries_.at(queue_id)->wakeable) {
    queue_entries_.at(queue_id)->wakeable->WakeUp(time);
  }
}
复制代码
```

在`RegisterTask`函数中，把任务task添加到优先级队列`delayed_tasks`中。然后再调用`MessageLoopAndroid`对象的`WakeUp`函数。

[-> flutter/fml/platform/android/message_loop_android.cc]

```
void MessageLoopAndroid::WakeUp(fml::TimePoint time_point) {
  bool result = TimerRearm(timer_fd_.get(), time_point);
  FML_DCHECK(result);
}
复制代码
```

`WakeUp`函数就是通过`TimerRearm`函数来在合适的时机唤醒`looper`。根据前面UI线程的创建过程，可得知在`looper`唤醒后的回调函数`read_event_fd`中是执行`MessageLoopAndroid`对象的`OnEventFired`函数，而在该函数中又直接调用`MessageLoopAndroid`对象的`FlushTasks`函数，下面就来看`FlushTasks`函数的实现。

[-> flutter/fml/message_loop_impl.cc]

```
void MessageLoopImpl::FlushTasks(FlushType type) {
  TRACE_EVENT0("fml", "MessageLoop::FlushTasks");
  std::vector<fml::closure> invocations;

  task_queue_->GetTasksToRunNow(queue_id_, type, invocations);

  for (const auto& invocation : invocations) {
    //执行普通回调方法
    invocation();
    std::vector<fml::closure> observers =
        task_queue_->GetObserversToNotify(queue_id_);
    for (const auto& observer : observers) {
      //observer对应着UIDartState对象的FlushMicrotasksNow函数，这里也就是执行所有的微任务
      observer();
    }
  }
}

void MessageLoopImpl::RunExpiredTasksNow() {
  FlushTasks(FlushType::kAll);
}
复制代码
```

在`FlushTasks`函数中，每一个已到期的`event handler`任务或异步任务执行完毕后，都会执行所有的微任务。

到此，在`ui_isolate`中，最终在以下两种时机来执行微任务。

1. 在调用`window`的`_beginFrame`与`_drawFrame`方法之间会把所有微任务处理掉。也就注定了不能在微任务中做耗时操作，否则影响UI的绘制。
2. 在每一个已到期的`event handler`任务或异步任务执行完毕后，都会执行所有的微任务。

再来看在非`ui_isolate`中微任务的执行时机。也主要分为以下几种情况。

1. 根据[Flutter之Timer原理解析](https://juejin.im/post/5ee19d11518825433b134766)一文可得知，在每一个`event handler`任务或异步任务执行完毕后，都会执行所有的微任务。
2. 如果当前是生产环境，`Isolate`中消息类型是`kDrainServiceExtensionsMsg`且消息优先级是`kImmediateAction`，则也会执行所有微任务

### 4、总结

以上就是微任务的使用及使用原理。还是有一定难度的。结合[Flutter之Timer原理解析](https://juejin.im/post/5ee19d11518825433b134766)一文，基本上就可以了解`Flutter`中的消息机制，这样在使用微任务及其他异步任务时也能做到了然于胸。