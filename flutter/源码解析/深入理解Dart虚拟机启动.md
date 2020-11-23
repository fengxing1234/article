---
title: 深入理解Dart虚拟机启动
date: 2020-08-04 11:27:38
tags:
	- flutter
	- 源码解析
categories: 
	- flutter
	- 源码解析
---

## 一、概述

### 1.1 Dart虚拟机概述

Dart虚拟机拥有自己的Isolate，完全由虚拟机自己管理的，Flutter引擎也无法直接访问。Dart的UI相关操作，是由Root Isolate通过Dart的C++调用，或者是发送消息通知的方式，将UI渲染相关的任务提交到UIRunner执行，这样就可以跟Flutter引擎相关模块进行交互。

Isolate之间内存并不共享，而是通过Port方式通信，每个Isolate是有自己的内存以及相应的线程载体。从字面上理解，Isolate是“隔离”，isolate之间的内是逻辑隔离的。Isolate中的代码也是按顺序执行，因为Dart没有共享内存的并发，没有竞争的可能性，故不需要加锁，也没有死锁风险。对于Dart程序的并发则需要依赖多个isolate来实现。

### 1.2 Dart虚拟机启动工作

文章在介绍[flutter引擎启动](http://gityuan.com/2019/06/22/flutter_booting/)过程，有两个环节没有展开讲解，那就是DartVM和Isolate的创建过程。Dart与Isolate的启动过程是在FlutterActivity的onCreate()过程触发，在引擎启动的过程有3个环节会跟Dart与Isolate的启动相关，如下所示：

- AndroidShellHolder对象的创建过程，会执行Shell::Create()方法，这里会调用DartVMRef::Create()，[见小节2.1]
- Shell::Create()方法完成，然后创建完Shell完成对象，再接着Engine对象的创建过程，先创建RuntimeController对象，这里会调用DartIsolate::CreateRootIsolate()，[见小节3.1]
- AndroidShellHolder创建完成后，回调FlutterActivityDelegate的runBundle()过程，经过层层调用，会调用到DartIsolate::Run()，[见小节4.1]

#### 1.2.1 DartVM启动工作

AndroidShellHolder对象的创建过程，会调用到DartVMRef::Create()，进行DartVM创建，主要是为Dart虚拟机解析数据DartVMData，注册一系列Native方法，创建名专属vm的Isolate对象，初始化虚拟机内存、堆、线程、StoreBuffer等大量对象，工作内容如下：

1. 同一个进程只有一个Dart虚拟机，所有的Shell共享该进程中的Dart虚拟机，当leak_vm为false则在最后一个Shell对象退出时会回收dart虚拟机， 当leak_vm为true则即便Shell对象全部退出也不会回收dart虚拟机，这是为了优化再次启动的速度；

2. 创建的IsolateNameServer对象，里面有一个重要的成员port_mapping_，以端口名为key，端口号为value的map结构，记录所有注册的port端口； 可通过RegisterIsolatePortWithName()注册Isolate端口，通过LookupIsolatePortByName()根据端口名来查询端口号;

3. 创建DartVMData对象，从settings中解析出vm_snapshot,isolate_snapshot这两个DartSnapshot对象，DartSnapshot里有data_和instructions_两个字段；

4. 创建ListeningSocketRegistry对象，其中有两个重要的成员变量sockets_by_port_（记录以端口号为key的socket集合），sockets_by_fd_（记录以fd为key的socket集合）；

5. 通过pthread_create创建名为”dart:io EventHandler”的线程，然后进入该线程进入poll轮询方法，一旦收到事件则执行HandleEvents()

6. 执行DartUI::InitForGlobal()：执行相关对象的RegisterNatives()注册Native方法，用于Dart代码调用C++代码。

   - 创建DartLibraryNatives类型的g_natives对象，作为静态变量，其成员变量entries_和symbols_分别用于记录NativeFunction和Symbol信息；通过该对象的Register()，注册dart的native方法，用于Dart调用C++代码；通过GetNativeFunction(),GetSymbol()来查询Native方法和符号信息。
   - Canvas、DartRuntimeHooks、Paragraph、Scene、Window等大量对象都会依次执行RegisterNatives(g_natives)来注册各种Native方法，

7. 执行Dart::Init()：传递的参数params记录isolate和文件等相关callback，用于各种对象的初始化

   - 初始化VirtualMemory、OSThread、PortMap、FreeListElement、ForwardingCorpse、Api、NativeSymbolResolver、SemiSpace、StoreBuffer、MarkingStack对象
   - 初始化ThreadPool线程池对象
   - 创建名为”vm-isolate”的Isolate对象，注意此处不允许混淆符号，将新创建的isolate添加到isolates_list_head_链表；

   - 为该isolate创建Heap、ApiState等对象；
   - 创建IsolateMessageHandler，其继承于MessageHandler，MessageHandler中有两个比较重要的成员变量queue_和oob_queue_，用于记录普通消息和oob消息的队列。
   - isolate需要有一个Port端口号，通过PortMap::CreatePort()随机数生成一个整型的端口号，PortMap的成员变量map_是一个记录端口entry的HashMap，每一个entry里面有端口号，handler，以及端口状态。

**[DartVM创建流程](http://gityuan.com/img/dart_vm/DartVM.jpg)**

![DartVM](http://gityuan.com/img/dart_vm/DartVM.jpg)

#### 1.2.2 Isolate启动工作

Dart虚拟机创建完成后，需要创建Engine对象，然后调用DartIsolate::CreateRootIsolate()来创建Isolate。每个Engine对应需要一个Isolate对象。

1. 创建DartIsolate和UIDartState对象，该对象继承于UIDartState对象；
2. 创建Root Isolate，并初始化Isolate相关信息，这个跟前面的Isolate过程一致。
   - 创建名为“Isolate-xxx”的Isolate对象，其中xxx是通过PortMap::CreatePort所创建port端口号
3. 只有root isolates能和window交互，也只有Root Isolate才会执行DartMessageHandler::Initialize()，设置task_dispatcher_等价于UITaskRunner->PostTask()；
4. DartRuntimeHooks安装过程，包括dart:isolate，dart:_internal，dart:core，dart:async，dart:io，

**[RootIsolate创建流程](http://gityuan.com/img/dart_vm/RootIsolate.jpg)**

![RootIsolate](http://gityuan.com/img/dart_vm/RootIsolate.jpg)

### 1.3 说明

#### 1.3.1 如何注册Native方法

DartUI::InitForGlobal()这个过程再细看看

#### 1.3.2 Isolate说明

关于Isolate有以下四种：

- 虚拟机Isolate：”vm-isolate”，运行在ui线程
- 常见Isolate：“Isolate-xxx”，对于RootIsolate则属于这个类别，运行在ui线程；
- 虚拟机服务Isolate：”vm-service”，运行在独立的线程；
- 内核服务Isolate：”kernel-service”；

另外，Runtime Mode运行时模式有3种：debug，profile，release，其中debug模式使用JIT，profile/release使用AOT。在整个源码过程会有不少通过宏来控制针对不同运行模式的代码。 比如当处于非预编译模式(DART_PRECOMPILED_RUNTIME)，则开启Assert断言，否则关闭断言。

#### 1.3.3 相关类图

**[Isolate类图](http://gityuan.com/img/dart_vm/ClassIsolate.jpg)**

![ClassIsolate](http://gityuan.com/img/dart_vm/ClassIsolate.jpg)

#### 1.3.4 核心代码入口

文章在介绍[flutter引擎启动](http://gityuan.com/2019/06/22/flutter_booting/)过程，有两个环节没有展开讲解，那就是DartVM和Isolate的创建过程。Dart与Isolate的启动过程是在FlutterActivity的onCreate()过程触发，在引擎启动的过程有3个环节会跟Dart与Isolate的启动相关，如下所示：

- AndroidShellHolder对象的创建过程，会执行Shell::Create()方法，这里会调用DartVMRef::Create()，[见小节2.1]
- Shell::Create()方法完成，然后创建完Shell完成对象，再接着Engine对象的创建过程，先创建RuntimeController对象，这里会调用DartIsolate::CreateRootIsolate()，[见小节3.1]
- AndroidShellHolder创建完成后，回调FlutterActivityDelegate的runBundle()过程，经过层层调用，会调用到DartIsolate::Run()，[见小节4.1]

```Java
//创建Dart虚拟机
new AndroidShellHolder
  Shell::Create()
    DartVMRef::Create()

//创建Isolate
new Engine
  new RuntimeController
    DartIsolate::CreateRootIsolate

AndroidShellHolder::Launch
    Engine::Run
      Engine::PrepareAndLaunchIsolate
        DartIsolate::Run
```

接下来依次看看上述这3个过程。

##  二、创建Dart虚拟机

### 2.1 DartVMRef::Create

[-> flutter/runtime/dart_vm_lifecycle.cc]

```Java
static std::weak_ptr<DartVM> gVM;
static std::shared_ptr<DartVM>* gVMLeak;

DartVMRef DartVMRef::Create(Settings settings,
                            fml::RefPtr<DartSnapshot> vm_snapshot,
                            fml::RefPtr<DartSnapshot> isolate_snapshot,
                            fml::RefPtr<DartSnapshot> shared_snapshot) {
  std::lock_guard<std::mutex> lifecycle_lock(gVMMutex);

  //当该进程存在一个正在运行虚拟机，则获取其强引用，复用该虚拟机
  if (auto vm = gVM.lock()) {
    return DartVMRef{std::move(vm)};
  }

  std::lock_guard<std::mutex> dependents_lock(gVMDependentsMutex);
  ...
  //创建isolate服务
  auto isolate_name_server = std::make_shared<IsolateNameServer>();
  //创建虚拟机[见小节2.2]
  auto vm = DartVM::Create(std::move(settings),          
                           std::move(vm_snapshot),       
                           std::move(isolate_snapshot),  
                           std::move(shared_snapshot),   
                           isolate_name_server           
  );
  ...
  gVMData = vm->GetVMData();
  gVMServiceProtocol = vm->GetServiceProtocol();
  gVMIsolateNameServer = isolate_name_server;
  gVM = vm;
  //用于优化第二次的启动速度
  if (settings.leak_vm) {
    gVMLeak = new std::shared_ptr<DartVM>(vm);
  }

  return DartVMRef{std::move(vm)};
}
```

同一个进程只有一个Dart虚拟机，所有的Shell共享该进程中的Dart虚拟机，当leak_vm为false则在最后一个Shell对象退出时会回收dart虚拟机，当leak_vm为true则即便Shell对象全部退出也不会回收dart虚拟机，这是为了优化再次启动的速度。

这里创建的IsolateNameServer里面有一个重要的成员port_mapping_，以端口名为key，端口号为value的map结构，记录所有注册的port端口。

### 2.2 DartVM::Create

[-> flutter/runtime/dart_vm.cc]

```Java
std::shared_ptr<DartVM> DartVM::Create(
    Settings settings,
    fml::RefPtr<DartSnapshot> vm_snapshot,
    fml::RefPtr<DartSnapshot> isolate_snapshot,
    fml::RefPtr<DartSnapshot> shared_snapshot,
    std::shared_ptr<IsolateNameServer> isolate_name_server) {
  //只有settings有初值，其余参数为空[见小节2.2.1]
  auto vm_data = DartVMData::Create(settings,                     //
                                    std::move(vm_snapshot),       //
                                    std::move(isolate_snapshot),  //
                                    std::move(shared_snapshot)    //
  );
  ...
  //[见小节2.3]
  return std::shared_ptr<DartVM>(
      new DartVM(std::move(vm_data), std::move(isolate_name_server)));
}
```

#### 2.2.1 DartVMData::Create

[-> flutter/runtime/dart_vm_data.cc]

```Java
std::shared_ptr<const DartVMData> DartVMData::Create(
    Settings settings,
    fml::RefPtr<DartSnapshot> vm_snapshot,
    fml::RefPtr<DartSnapshot> isolate_snapshot,
    fml::RefPtr<DartSnapshot> shared_snapshot) {
  if (!vm_snapshot || !vm_snapshot->IsValid()) {
    vm_snapshot = DartSnapshot::VMSnapshotFromSettings(settings);
    ...
  }

  if (!isolate_snapshot || !isolate_snapshot->IsValid()) {
    isolate_snapshot = DartSnapshot::IsolateSnapshotFromSettings(settings);
    ...
  }

  if (!shared_snapshot || !shared_snapshot->IsValid()) {
    shared_snapshot = DartSnapshot::Empty();
    ...
  }
  //[见小节2.2.2]
  return std::shared_ptr<const DartVMData>(new DartVMData(
      std::move(settings),          //
      std::move(vm_snapshot),       //
      std::move(isolate_snapshot),  //
      std::move(shared_snapshot)    //
      ));
}
```

从settings中解析出vm_snapshot,isolate_snapshot数据。

#### 2.2.2 DartVMData初始化

[-> flutter/runtime/dart_vm_data.cc]

```Java
DartVMData::DartVMData(Settings settings,
                       fml::RefPtr<const DartSnapshot> vm_snapshot,
                       fml::RefPtr<const DartSnapshot> isolate_snapshot,
                       fml::RefPtr<const DartSnapshot> shared_snapshot)
    : settings_(settings),
      vm_snapshot_(vm_snapshot),
      isolate_snapshot_(isolate_snapshot),
      shared_snapshot_(shared_snapshot) {}
```

### 2.3 DartVM初始化

[-> flutter/runtime/dart_vm.cc]

```Java
DartVM::DartVM(std::shared_ptr<const DartVMData> vm_data,
               std::shared_ptr<IsolateNameServer> isolate_name_server)
    : settings_(vm_data->GetSettings()),
      vm_data_(vm_data),
      isolate_name_server_(std::move(isolate_name_server)),
      //创建ServiceProtocol
      service_protocol_(std::make_shared<ServiceProtocol>()) {
  TRACE_EVENT0("flutter", "DartVMInitializer");
  gVMLaunchCount++;
  {
    TRACE_EVENT0("flutter", "dart::bin::BootstrapDartIo");
    //[见小节2.4]
    dart::bin::BootstrapDartIo();
    if (!settings_.temp_directory_path.empty()) {
      dart::bin::SetSystemTempDirectory(settings_.temp_directory_path.c_str());
    }
  }

  std::vector<const char*> args;
  args.push_back("--ignore-unrecognized-flags"); //忽略无法识别的flags参数

  for (auto* const profiler_flag : ProfilingFlags(settings_.enable_dart_profiling)) {
    args.push_back(profiler_flag);
  }
  PushBackAll(&args, kDartLanguageArgs, arraysize(kDartLanguageArgs));

  if (IsRunningPrecompiledCode()) {
    PushBackAll(&args, kDartPrecompilationArgs, arraysize(kDartPrecompilationArgs));
  }

  bool enable_asserts = !settings_.disable_dart_asserts;
  if (IsRunningPrecompiledCode()) {
    enable_asserts = false; //当为预编译模式，则关闭Dart断言
  }

  if (enable_asserts) {
    PushBackAll(&args, kDartAssertArgs, arraysize(kDartAssertArgs));
  }

  if (settings_.start_paused) {
    PushBackAll(&args, kDartStartPausedArgs, arraysize(kDartStartPausedArgs));
  }

  if (settings_.disable_service_auth_codes) {
    PushBackAll(&args, kDartDisableServiceAuthCodesArgs,
                arraysize(kDartDisableServiceAuthCodesArgs));
  }

  if (settings_.endless_trace_buffer || settings_.trace_startup) {
    //开启无限大小的buffer，保证信息不被丢失
    PushBackAll(&args, kDartEndlessTraceBufferArgs,
                arraysize(kDartEndlessTraceBufferArgs));
  }

  if (settings_.trace_systrace) {
    PushBackAll(&args, kDartSystraceTraceBufferArgs,
                arraysize(kDartSystraceTraceBufferArgs));
    PushBackAll(&args, kDartTraceStreamsArgs, arraysize(kDartTraceStreamsArgs));
  }

  if (settings_.trace_startup) {
    PushBackAll(&args, kDartTraceStartupArgs, arraysize(kDartTraceStartupArgs));
  }

  for (size_t i = 0; i < settings_.dart_flags.size(); i++)
    args.push_back(settings_.dart_flags[i].c_str());

  char* flags_error = Dart_SetVMFlags(args.size(), args.data());
  ...

  DartUI::InitForGlobal();  //[见小节2.8]

  {
    TRACE_EVENT0("flutter", "Dart_Initialize");
    Dart_InitializeParams params = {};
    params.version = DART_INITIALIZE_PARAMS_CURRENT_VERSION;
    params.vm_snapshot_data = vm_data_->GetVMSnapshot().GetData()->GetSnapshotPointer();
    params.vm_snapshot_instructions = vm_data_->GetVMSnapshot().GetInstructionsIfPresent();
    params.create = reinterpret_cast<decltype(params.create)>(
                  DartIsolate::DartIsolateCreateCallback);
    params.shutdown = reinterpret_cast<decltype(params.shutdown)>(
                  DartIsolate::DartIsolateShutdownCallback);
    params.cleanup = reinterpret_cast<decltype(params.cleanup)>(
                  DartIsolate::DartIsolateCleanupCallback);
    params.thread_exit = ThreadExitCallback;
    params.get_service_assets = GetVMServiceAssetsArchiveCallback;
    params.entropy_source = dart::bin::GetEntropy;
    //[见小节2.9]
    char* init_error = Dart_Initialize(&params);

    //应用生命周期中最早的可记录时间戳发送到timeline
    if (engine_main_enter_ts != 0) {
      Dart_TimelineEvent("FlutterEngineMainEnter",
                         engine_main_enter_ts, engine_main_enter_ts,     
                         Dart_Timeline_Event_Duration, 0, nullptr, nullptr                        
      );
    }
  }

  Dart_SetFileModifiedCallback(&DartFileModifiedCallback);

  //允许Dart vm输出端stdout和stderr
  Dart_SetServiceStreamCallbacks(&ServiceStreamListenCallback,
                                 &ServiceStreamCancelCallback);

  Dart_SetEmbedderInformationCallback(&EmbedderInformationCallback);

  if (settings_.dart_library_sources_kernel != nullptr) {
    std::unique_ptr<fml::Mapping> dart_library_sources =
        settings_.dart_library_sources_kernel();
    //设置dart：*库的源代码以进行调试。
    Dart_SetDartLibrarySourcesKernel(dart_library_sources->GetMapping(),
                                     dart_library_sources->GetSize());
  }
}
```

该方法主要功能：

- 根据条件，创建Dart所需要的args信息
- 是否开启assert断言，取决于是否运行预编译代码。只有在debug模式才以JIT模式编译dart代码，对于profile/release都是以AOT模式编译成机器码；

### 2.4 BootstrapDartIo

[-> third_party/dart/runtime/bin/dart_io_api_impl.cc]

```Java
void BootstrapDartIo() {
  TimerUtils::InitOnce();
  //[见小节2.5]
  EventHandler::Start();
}
```

### 2.5 EventHandler::Start

[-> third_party/dart/runtime/bin/eventhandler.cc]

```Java
void EventHandler::Start() {
  ListeningSocketRegistry::Initialize(); //[见小节2.5.1]
  shutdown_monitor = new Monitor();   //[见小节2.5.3]
  event_handler = new EventHandler();  //[见小节2.5.4]
  event_handler->delegate_.Start(event_handler); //[见小节2.6]
}
```

此处delegate_是EventHandlerImplementation对象。

#### 2.5.1 ListeningSocketRegistry::Initialize

[-> third_party/dart/runtime/bin/socket.cc]

```Java
ListeningSocketRegistry* globalTcpListeningSocketRegistry = NULL;

void ListeningSocketRegistry::Initialize() {
  //[见小节2.5.2]
  globalTcpListeningSocketRegistry = new ListeningSocketRegistry();
}
```

初始化全局的socket注册对象，并保存在全局变量globalTcpListeningSocketRegistry中。

#### 2.5.2 ListeningSocketRegistry初始化

[-> third_party/dart/runtime/bin/socket.h]

```Java
class ListeningSocketRegistry {
  ListeningSocketRegistry()
      : sockets_by_port_(SameIntptrValue, kInitialSocketsCount),
        sockets_by_fd_(SameIntptrValue, kInitialSocketsCount),
        mutex_(new Mutex()) {}

  SimpleHashMap sockets_by_port_;
  SimpleHashMap sockets_by_fd_;
  Mutex* mutex_;
}
```

ListeningSocketRegistry有两个重要的成员变量：

- sockets_by_port_：类型为SimpleHashMap，记录以端口号为key的socket集合；
- sockets_by_fd_：类型为SimpleHashMap，记录以fd为key的socket集合；

#### 2.5.3 Monitor初始化

[-> third_party/dart/runtime/bin/thread_android.cc]

```Java
Monitor::Monitor() {
  pthread_mutexattr_t mutex_attr;
  pthread_mutexattr_init(&mutex_attr);
  pthread_mutex_init(data_.mutex(), &mutex_attr);
  pthread_mutexattr_destroy(&mutex_attr);

  pthread_condattr_t cond_attr;
  pthread_condattr_init(&cond_attr);
  pthread_cond_init(data_.cond(), &cond_attr);
  pthread_condattr_destroy(&cond_attr);
}

Monitor::~Monitor() {
  pthread_mutex_destroy(data_.mutex());
  pthread_cond_destroy(data_.cond());
}
```

此处的data_是Monitor类中的数据类型为MonitorData的私有成员变量，其中mutex()和cond()对应如下两个成员变量：

- mutex_：类型为pthread_mutex_t（互斥锁），用于在多线程中对共享变量的保护
- cond_ ：类型为pthread_cond_t（条件变量），一般和pthread_mutex_t配合使用，以防止有可能对共享变量读写出现非预期行为

对于Monitor有Enter/Exit，以及Wait/Notify/NotifyAll方法，内部的实现便是基于以下方法：

- pthread_mutex_lock: 上锁
- pthread_mutex_unlock：解锁
- pthread_cond_wait：阻塞在条件变量
- pthread_cond_timedwait：带超时机制的阻塞
- pthread_cond_signal：唤醒在条件变量阻塞的线程
- pthread_cond_broadcast：唤醒所有在该条件变量上的线程

#### 2.5.4 EventHandler初始化

[-> third_party/dart/runtime/bin/eventhandler.h]

```Java
class EventHandler {
  EventHandler() {}
  //[见小节2.5.5]
  EventHandlerImplementation delegate_;
}
```

#### 2.5.5 EventHandlerImplementation初始化

[-> third_party/dart/runtime/bin/eventhandler_android.h]

```Java
class EventHandlerImplementation {
 public:
  EventHandlerImplementation();

 private:
  SimpleHashMap socket_map_;
  TimeoutQueue timeout_queue_;
  bool shutdown_;
  int interrupt_fds_[2];
  int epoll_fd_;
};
```

### 2.6 EventHandlerImplementation::Start

[-> third_party/dart/runtime/bin/eventhandler_android.cc]

```Java
void EventHandlerImplementation::Start(EventHandler* handler) {
  //[见小节2.7]
  int result = Thread::Start("dart:io EventHandler",
          &EventHandlerImplementation::Poll, reinterpret_cast<uword>(handler));
}
```

通过Thread::Start()来创建一个名为“dart:io EventHandler”的线程，过程如下所示。

#### 2.6.1 Thread::Start

[-> third_party/dart/runtime/bin/thread_android.cc]

```Java
int Thread::Start(const char* name,
                  ThreadStartFunction function,
                  uword parameter) {
  pthread_attr_t attr;
  int result = pthread_attr_init(&attr);
  result = pthread_attr_setdetachstate(&attr, PTHREAD_CREATE_DETACHED);
  result = pthread_attr_setstacksize(&attr, Thread::GetMaxStackSize());

  ThreadStartData* data = new ThreadStartData(name, function, parameter);
  pthread_t tid;
  //通过pthread_create创建线程，然后触发执行Poll，见[小节2.7]
  result = pthread_create(&tid, &attr, ThreadStart, data);
  result = pthread_attr_destroy(&attr);
  return 0;
}
```

通过pthread_create创建线程，然后在线程中执行function，此处等于EventHandlerImplementation对象中的Poll(EventHandler)方法。

### 2.7 Poll

[-> third_party/dart/runtime/bin/eventhandler_android.cc]

```Java
void EventHandlerImplementation::Poll(uword args) {
  ThreadSignalBlocker signal_blocker(SIGPROF);
  static const intptr_t kMaxEvents = 16;  //最多处理16个事件
  struct epoll_event events[kMaxEvents];
  EventHandler* handler = reinterpret_cast<EventHandler*>(args);
  EventHandlerImplementation* handler_impl = &handler->delegate_;

  while (!handler_impl->shutdown_) {
    int64_t millis = handler_impl->GetTimeout();
    intptr_t result = TEMP_FAILURE_RETRY_NO_SIGNAL_BLOCKER(
        //进入epoll的状态等待事件
        epoll_wait(handler_impl->epoll_fd_, events, kMaxEvents, millis));
    if (result != -1) {
      handler_impl->HandleTimeout(); //[见小节2.7.1]
      handler_impl->HandleEvents(events, result); //[见小节2.7.2]
    }
  }
  handler->NotifyShutdownDone(); //[见小节2.7.4]
}
```

一旦收到事件，则执行HandleEvents()方法。

#### 2.7.1 HandleTimeout

[-> third_party/dart/runtime/bin/eventhandler_android.cc]

```Java
void EventHandlerImplementation::HandleTimeout() {
  if (timeout_queue_.HasTimeout()) {
    int64_t millis = timeout_queue_.CurrentTimeout() -
                     TimerUtils::GetCurrentMonotonicMillis();
    if (millis <= 0) {
      //当已超时，则向指定端口号发送一个空消息
      DartUtils::PostNull(timeout_queue_.CurrentPort());
      timeout_queue_.RemoveCurrent();
    }
  }
}
```

#### 2.7.2 HandleEvents

[-> third_party/dart/runtime/bin/eventhandler_android.cc]

```Java
void EventHandlerImplementation::HandleEvents(struct epoll_event* events,
                                              int size) {
  bool interrupt_seen = false;
  for (int i = 0; i < size; i++) {
    if (events[i].data.ptr == NULL) {
      interrupt_seen = true;
    } else {
      DescriptorInfo* di = reinterpret_cast<DescriptorInfo*>(events[i].data.ptr);
      const intptr_t old_mask = di->Mask();
      const intptr_t event_mask = GetPollEvents(events[i].events, di);
      if ((event_mask & (1 << kErrorEvent)) != 0) {
        di->NotifyAllDartPorts(event_mask);
        UpdateEpollInstance(old_mask, di);
      } else if (event_mask != 0) {
        Dart_Port port = di->NextNotifyDartPort(event_mask);
        UpdateEpollInstance(old_mask, di);
        DartUtils::PostInt32(port, event_mask);
      }
    }
  }
  if (interrupt_seen) {
    //执行socket事件后，在处理当前事件之前避免关闭套接字。 [见小节2.7.3]
    HandleInterruptFd();
  }
}
```

#### 2.7.3 HandleInterruptFd

[-> third_party/dart/runtime/bin/eventhandler_android.cc]

```Java
void EventHandlerImplementation::HandleInterruptFd() {
  const intptr_t MAX_MESSAGES = kInterruptMessageSize;
  InterruptMessage msg[MAX_MESSAGES];
  //从interrupt_fds_中读取消息
  ssize_t bytes = TEMP_FAILURE_RETRY_NO_SIGNAL_BLOCKER(
      read(interrupt_fds_[0], msg, MAX_MESSAGES * kInterruptMessageSize));
  for (ssize_t i = 0; i < bytes / kInterruptMessageSize; i++) {
    if (msg[i].id == kTimerId) {
      //当属于定时器消息，则将该消息加入超时消息队列
      timeout_queue_.UpdateTimeout(msg[i].dart_port, msg[i].data);
    } else if (msg[i].id == kShutdownId) {
      //当属于关闭消息，则会停止轮询poll
      shutdown_ = true;
    } else {
      Socket* socket = reinterpret_cast<Socket*>(msg[i].id);
      RefCntReleaseScope<Socket> rs(socket);

      DescriptorInfo* di =
          GetDescriptorInfo(socket->fd(), IS_LISTENING_SOCKET(msg[i].data));
      if (IS_COMMAND(msg[i].data, kShutdownReadCommand)) {
        //关闭socket读取
        VOID_NO_RETRY_EXPECTED(shutdown(di->fd(), SHUT_RD));
      } else if (IS_COMMAND(msg[i].data, kShutdownWriteCommand)) {
        //关闭socket写入
        VOID_NO_RETRY_EXPECTED(shutdown(di->fd(), SHUT_WR));
      } else if (IS_COMMAND(msg[i].data, kCloseCommand)) {
        // 关系socket，释放系统资源，并移动到下一个消息
        ...
        //向端口发送一个销毁的消息
        DartUtils::PostInt32(port, 1 << kDestroyedEvent);
      } else if (IS_COMMAND(msg[i].data, kReturnTokenCommand)) {
        int count = TOKEN_COUNT(msg[i].data);
        intptr_t old_mask = di->Mask();
        di->ReturnTokens(msg[i].dart_port, count);
        UpdateEpollInstance(old_mask, di);
      } else if (IS_COMMAND(msg[i].data, kSetEventMaskCommand)) {
        intptr_t events = msg[i].data & EVENT_MASK;
        intptr_t old_mask = di->Mask();
        di->SetPortAndMask(msg[i].dart_port, msg[i].data & EVENT_MASK);
        UpdateEpollInstance(old_mask, di);
      } else {
        UNREACHABLE();
      }
    }
  }
}
```

#### 2.7.4 NotifyShutdownDone

[-> third_party/dart/runtime/bin/eventhandler.cc]

```Java
void EventHandler::NotifyShutdownDone() {
  MonitorLocker ml(shutdown_monitor);
  ml.Notify(); //唤醒处于等待状态的shutdown_monitor
}
```

唤醒处于等待状态的shutdown_monitor，然后会清理全局注册的socket，见小节[2.7.5]

#### 2.7.4 Stop

[-> third_party/dart/runtime/bin/eventhandler.cc]

```Java
void EventHandler::Stop() {
  {
    MonitorLocker ml(shutdown_monitor);
    event_handler->delegate_.Shutdown();
    ml.Wait(Monitor::kNoTimeout);  //无限等待，直到被唤醒
  }

  // 清理
  delete event_handler;
  event_handler = NULL;
  delete shutdown_monitor;
  shutdown_monitor = NULL;

  //销毁全局注册的socket
  ListeningSocketRegistry::Cleanup();
}
```

到此[小节2.4]BootstrapDartIo方法执行完成，再回到[小节2.3]执行InitForGlobal()方法

### 2.8 DartUI::InitForGlobal

[-> flutter/lib/ui/dart_ui.cc]

```Java
void DartUI::InitForGlobal() {
   //只初始化一次
  if (!g_natives) {
    g_natives = new tonic::DartLibraryNatives(); // [见小节2.8.1]
    Canvas::RegisterNatives(g_natives);
    CanvasGradient::RegisterNatives(g_natives);
    CanvasImage::RegisterNatives(g_natives);
    CanvasPath::RegisterNatives(g_natives);
    CanvasPathMeasure::RegisterNatives(g_natives);
    Codec::RegisterNatives(g_natives);
    DartRuntimeHooks::RegisterNatives(g_natives);  // [见小节2.8.2]
    EngineLayer::RegisterNatives(g_natives);
    FontCollection::RegisterNatives(g_natives);
    FrameInfo::RegisterNatives(g_natives);
    ImageFilter::RegisterNatives(g_natives);
    ImageShader::RegisterNatives(g_natives);
    IsolateNameServerNatives::RegisterNatives(g_natives);
    Paragraph::RegisterNatives(g_natives);
    ParagraphBuilder::RegisterNatives(g_natives);
    Picture::RegisterNatives(g_natives);
    PictureRecorder::RegisterNatives(g_natives);
    Scene::RegisterNatives(g_natives);
    SceneBuilder::RegisterNatives(g_natives);
    SceneHost::RegisterNatives(g_natives);
    SemanticsUpdate::RegisterNatives(g_natives);
    SemanticsUpdateBuilder::RegisterNatives(g_natives);
    Vertices::RegisterNatives(g_natives);
    Window::RegisterNatives(g_natives);

    // 第二个isolates不提供UI相关的APIs
    g_natives_secondary = new tonic::DartLibraryNatives();
    DartRuntimeHooks::RegisterNatives(g_natives_secondary);
    IsolateNameServerNatives::RegisterNatives(g_natives_secondary);
  }
}
```

先创建DartLibraryNatives类型的g_natives，然后利用该对象来注册各种Native方法，这里以DartRuntimeHooks为例来说明[小节2.8.2]

#### 2.8.1 DartLibraryNatives初始化

[-> third_party/tonic/dart_library_natives.h]

```Java
class DartLibraryNatives {
 public:
  DartLibraryNatives();
  ~DartLibraryNatives();

  struct Entry {
    const char* symbol;
    Dart_NativeFunction native_function;
    int argument_count;
    bool auto_setup_scope;
  };

  void Register(std::initializer_list<Entry> entries);

  Dart_NativeFunction GetNativeFunction(Dart_Handle name,
                                        int argument_count,
                                        bool* auto_setup_scope);
  const uint8_t* GetSymbol(Dart_NativeFunction native_function);

 private:
  std::unordered_map<std::string, Entry> entries_;
  std::unordered_map<Dart_NativeFunction, const char*> symbols_;
};
```

来DartLibraryNatives的成员变量entries_和symbols_用于记录NativeFunction和Symbol信息，该类有3个方法：

- Register：注册dart的native方法，用于Dart调用C++代码。
- GetNativeFunction：根据Symbol和参数个数来查找NativeFunction
- GetSymbol：根据NativeFunction来查找Symbol

#### 2.8.2 RegisterNatives

[-> flutter/lib/ui/dart_runtime_hooks.cc]

```Java
void DartRuntimeHooks::RegisterNatives(tonic::DartLibraryNatives* natives) {
  //[见小节2.8.3/ 2.8.4]
  natives->Register({BUILTIN_NATIVE_LIST(REGISTER_FUNCTION)});
}
```

#### 2.8.3 Register

[-> third_party/tonic/dart_library_natives.cc]

```Java
void DartLibraryNatives::Register(std::initializer_list<Entry> entries) {
  for (const Entry& entry : entries) {
    symbols_.emplace(entry.native_function, entry.symbol);
    entries_.emplace(entry.symbol, entry);
  }
}
```

将注册的entry都记录DartLibraryNatives对象的成员变量symbols_和entries_里面。

#### 2.8.4 BUILTIN_NATIVE_LIST

[-> flutter/lib/ui/dart_runtime_hooks.cc]

```Java
#define REGISTER_FUNCTION(name, count) {"" #name, name, count, true},
#define BUILTIN_NATIVE_LIST(V) \
  V(Logger_PrintString, 1)     \
  V(SaveCompilationTrace, 0)   \
  V(ScheduleMicrotask, 1)      \
  V(GetCallbackHandle, 1)      \
  V(GetCallbackFromHandle, 1)
```

这是一个宏定义，REGISTER_FUNCTION的4个参数代表的如下所示的DartLibraryNatives中的Entry结构体

```Java
struct Entry {
  const char* symbol;
  Dart_NativeFunction native_function;
  int argument_count;
  bool auto_setup_scope;
};
```

可知，DartRuntimeHooks中注册了ScheduleMicrotask()等共5个Native方法。也就是说当调用natives.dart带有native标识的方法，最终会调用到dart_runtime_hooks.cc中的ScheduleMicrotask()方法，如下所示的两个相对应的方法。

[-> flutter/lib/ui/natives.dart]

```Java
void _scheduleMicrotask(void callback()) native 'ScheduleMicrotask';
```

[-> flutter/lib/ui/dart_runtime_hooks.cc]

```Java
void ScheduleMicrotask(Dart_NativeArguments args) {
  Dart_Handle closure = Dart_GetNativeArgument(args, 0);
  UIDartState::Current()->ScheduleMicrotask(closure);
}
```

### 2.9 Dart_Initialize

[-> third_party/dart/runtime/vm/dart_api_impl.cc]

```Java
DART_EXPORT char* Dart_Initialize(Dart_InitializeParams* params) {
  ...
  //[见小节2.10]
  return Dart::Init(params->vm_snapshot_data, params->vm_snapshot_instructions,
                    params->create, params->shutdown,
                    params->cleanup, params->thread_exit,
                    params->file_open, params->file_read,
                    params->file_write, params->file_close,
                    params->entropy_source, params->get_service_assets,
                    params->start_kernel_isolate);
}
```

### 2.10 Dart::Init

[-> third_party/dart/runtime/vm/dart.cc]

```Java
char* Dart::Init(const uint8_t* vm_isolate_snapshot,
                 const uint8_t* instructions_snapshot,
                 Dart_IsolateCreateCallback create,
                 Dart_IsolateShutdownCallback shutdown,
                 Dart_IsolateCleanupCallback cleanup,
                 Dart_ThreadExitCallback thread_exit,
                 Dart_FileOpenCallback file_open,
                 Dart_FileReadCallback file_read,
                 Dart_FileWriteCallback file_write,
                 Dart_FileCloseCallback file_close,
                 Dart_EntropySource entropy_source,
                 Dart_GetVMServiceAssetsArchive get_service_assets,
                 bool start_kernel_isolate) {
  const Snapshot* snapshot = nullptr;
  if (vm_isolate_snapshot != nullptr) {
    //创建Snapshot
    snapshot = Snapshot::SetupFromBuffer(vm_isolate_snapshot);
  }

  if (snapshot != nullptr) {
    char* error = SnapshotHeaderReader::InitializeGlobalVMFlagsFromSnapshot(snapshot);
  }
  ...
  FrameLayout::Init();
  //设置dart的线程退出函数、文件操作函数
  set_thread_exit_callback(thread_exit);
  SetFileCallbacks(file_open, file_read, file_write, file_close);
  set_entropy_source_callback(entropy_source);

  OS::Init();
  start_time_micros_ = OS::GetCurrentMonotonicMicros();
  VirtualMemory::Init(); //初始化虚拟内存 见[小节2.10.1]
  OSThread::Init(); //初始化系统线程 见[小节2.10.2]
  Isolate::InitVM(); //初始化Isolate 见[小节2.10.3]
  PortMap::Init(); //初始化PortMap 见[小节2.10.4]
  FreeListElement::Init();
  ForwardingCorpse::Init();
  Api::Init();
  NativeSymbolResolver::Init();
  SemiSpace::Init();
  StoreBuffer::Init();
  MarkingStack::Init();

  predefined_handles_ = new ReadOnlyHandles(); //创建只读handles
  //创建VM isolate，完成虚拟机初始化 见[小节2.10.5]
  thread_pool_ = new ThreadPool();
  {
    const bool is_vm_isolate = true;
    Dart_IsolateFlags api_flags;
    Isolate::FlagsInitialize(&api_flags);
    //[见小节2.11]
    vm_isolate_ = Isolate::InitIsolate("vm-isolate", api_flags, is_vm_isolate);

    Thread* T = Thread::Current();
    StackZone zone(T);
    HandleScope handle_scope(T);
    Object::InitNull(vm_isolate_);
    ObjectStore::Init(vm_isolate_);
    TargetCPUFeatures::Init();
    Object::Init(vm_isolate_);
    ArgumentsDescriptor::Init();
    ICData::Init();
    //根据vm_isolate_snapshot是否为空，以及snapshot的kind类型、编译模式来出现相关流程
    if (vm_isolate_snapshot != NULL) {
      ...
    } else {
      ...
    }

    T->InitVMConstants(); //初始化VM isolate常量
    {
      Object::FinalizeVMIsolate(vm_isolate_);
    }
  }
  Api::InitHandles();
  Thread::ExitIsolate();  //取消注册该线程中的VM isolate
  Isolate::SetCreateCallback(create);
  Isolate::SetShutdownCallback(shutdown);
  Isolate::SetCleanupCallback(cleanup);
  ...
  return NULL;
}
```

根据vm_isolate_snapshot是否则需要进行相关处理，以下3种情况则会直接返回，不再往下执行：

- 当vm_isolate_snapshot不为空的情况时：
  - 当snapshot类型为kFullAOT，且并非处于预编译模式(DART_PRECOMPILED_RUNTIME);
  - 当snapshot类型为kFull，且处于预编译模式(DART_PRECOMPILED_RUNTIME);
  - 当snapshot类型不是kFullAOT或kFullJIT或kFull；
- 当vm_isolate_snapshot等于空的情况时：
  - 当处于预编译模式(DART_PRECOMPILED_RUNTIME)，预编译模式需要预编译的snapshot；

#### 2.10.1 VirtualMemory::Init

[-> third_party/dart/runtime/vm/virtual_memory_posix.cc]

```Java
void VirtualMemory::Init() {
  page_size_ = getpagesize(); //获取页大小
}
```

#### 2.10.2 OSThread::Init

[-> third_party/dart/runtime/vm/os_thread.cc]

```Java
void OSThread::Init() {
  //分配全局OSThread锁
  if (thread_list_lock_ == NULL) {
    thread_list_lock_ = new Mutex();
  }

  //创建线程本地Key
  if (thread_key_ == kUnsetThreadLocalKey) {
    thread_key_ = CreateThreadLocal(DeleteThread);
  }

  //使能虚拟机中的OSThread创建
  EnableOSThreadCreation();

  //创建一个新的OSThread结构体，并赋予TLS
  OSThread* os_thread = CreateOSThread();
  OSThread::SetCurrent(os_thread);
  os_thread->set_name("Dart_Initialize");
}
```

#### 2.10.3 Isolate::InitVM

[-> third_party/dart/runtime/vm/isolate.cc]

```Java
void Isolate::InitVM() {
  create_callback_ = NULL;
  if (isolates_list_monitor_ == NULL) {
    isolates_list_monitor_ = new Monitor();
  }
  EnableIsolateCreation(); //使能Isolate创建
}
```

#### 2.10.4 PortMap::Init

[-> third_party/dart/runtime/vm/port.cc]

```Java
void PortMap::Init() {
  if (mutex_ == NULL) {
    mutex_ = new Mutex();
  }
  prng_ = new Random();

  static const intptr_t kInitialCapacity = 8;
  if (map_ == NULL) {
    map_ = new Entry[kInitialCapacity];
    capacity_ = kInitialCapacity;
  }
  memset(map_, 0, capacity_ * sizeof(Entry));
  used_ = 0;
  deleted_ = 0;
}
```

#### 2.10.5 ThreadPool初始化

[-> third_party/dart/runtime/vm/thread_pool.cc]

```Java
ThreadPool::ThreadPool()
    : shutting_down_(false),
      all_workers_(NULL),
      idle_workers_(NULL),
      count_started_(0),
      count_stopped_(0),
      count_running_(0),
      count_idle_(0),
      shutting_down_workers_(NULL),
      join_list_(NULL) {}
```

### 2.11 Isolate::InitIsolate

[-> third_party/dart/runtime/vm/isolate.cc]

```Java
Isolate* Isolate::InitIsolate(const char* name_prefix,
                              const Dart_IsolateFlags& api_flags,
                              bool is_vm_isolate) {
  //创建Isolate [2.11.1]
  Isolate* result = new Isolate(api_flags);

  bool is_service_or_kernel_isolate = false;
  if (ServiceIsolate::NameEquals(name_prefix)) {
    is_service_or_kernel_isolate = true;
  }
#if !defined(DART_PRECOMPILED_RUNTIME)
  if (KernelIsolate::NameEquals(name_prefix)) {
    KernelIsolate::SetKernelIsolate(result);
    is_service_or_kernel_isolate = true;
  }
#endif
  //初始化堆[见小节2.11.2]
  Heap::Init(result,
             is_vm_isolate ? 0 : FLAG_new_gen_semi_max_size * MBInWords,
             (is_service_or_kernel_isolate ? kDefaultMaxOldGenHeapSize
                                           : FLAG_old_gen_heap_size) * MBInWords);
  // [见小节2.11.3]
  if (!Thread::EnterIsolate(result)) {
    ... //当进入isolate失败，则关闭并删除isolate，并直接返回
    return NULL;
  }

  //创建isolate的消息处理对象 [见小节2.11.4]
  MessageHandler* handler = new IsolateMessageHandler(result);
  result->set_message_handler(handler);

  //创建Dart API状态的对象
  ApiState* state = new ApiState();
  result->set_api_state(state);
  // [见小节2.11.5]
  result->set_main_port(PortMap::CreatePort(result->message_handler()));
  result->set_origin_id(result->main_port());
  result->set_pause_capability(result->random()->NextUInt64());
  result->set_terminate_capability(result->random()->NextUInt64());
  //设置isolate的名称
  result->BuildName(name_prefix);

  //将isolate添加到isolates_list_head_列表
  if (!AddIsolateToList(result)) {
    //当添加失败，则关闭并删除isolate，并直接返回
    ...
  }
  return result;
}
```

该方法主要功能：

- DartVM初始化过程创建的是名为”vm-isolate”的isolate对象，对于isolate的命名方式：
  - 当指定名称name_prefix，则按指定名，比如此处为”vm-isolate”；
  - 当没有指定名称，则为“isolate-xxx”，此处的xxx为main_port端口号；
- 创建Heap，IsolateMessageHandler，ApiState对象，都保存到该isolate对象的成员变量；
- 如果整个过程执行成功，则将新创建的isolate添加到isolates_list_head_链表；如果失败，则会关闭并删除isolate对象。

#### 2.11.1 Isolate初始化

[-> third_party/dart/runtime/vm/isolate.cc]

```Java
Isolate::Isolate(const Dart_IsolateFlags& api_flags)
    : BaseIsolate(),
      user_tag_(0),
      current_tag_(UserTag::null()),
      default_tag_(UserTag::null()),
      ic_miss_code_(Code::null()),
      object_store_(NULL),
      class_table_(),
      single_step_(false),
      store_buffer_(new StoreBuffer()),
      marking_stack_(NULL),
      heap_(NULL),
      isolate_flags_(0),
      background_compiler_(NULL),
      optimizing_background_compiler_(NULL),
      start_time_micros_(OS::GetCurrentMonotonicMicros()),
      thread_registry_(new ThreadRegistry()),
      safepoint_handler_(new SafepointHandler(this)),
      message_notify_callback_(NULL),
      name_(NULL),
      main_port_(0),
      origin_id_(0),
      pause_capability_(0),
      terminate_capability_(0),
      init_callback_data_(NULL),
      environment_callback_(NULL),
      library_tag_handler_(NULL),
      api_state_(NULL),
      random_(),
      simulator_(NULL),
      ... //省略若干metex
      message_handler_(NULL),
      spawn_state_(NULL),
      defer_finalization_count_(0),
      pending_deopts_(new MallocGrowableArray<PendingLazyDeopt>()),
      deopt_context_(NULL),
      tag_table_(GrowableObjectArray::null()),
      deoptimized_code_array_(GrowableObjectArray::null()),
      sticky_error_(Error::null()),
      reloaded_kernel_blobs_(GrowableObjectArray::null()),
      next_(NULL),
      loading_invalidation_gen_(kInvalidGen),
      boxed_field_list_(GrowableObjectArray::null()),
      spawn_count_monitor_(new Monitor()),
      spawn_count_(0),
      handler_info_cache_(),
      catch_entry_moves_cache_(),
      embedder_entry_points_(NULL),
      obfuscation_map_(NULL),
      reverse_pc_lookup_cache_(nullptr) {
  FlagsCopyFrom(api_flags);
  SetErrorsFatal(true);
  set_compilation_allowed(true);
  set_user_tag(UserTags::kDefaultUserTag);
  ... //DartVM不允许混淆符号
  if (FLAG_enable_interpreter) {
    NOT_IN_PRECOMPILED(background_compiler_ = new BackgroundCompiler(this));
  }
  NOT_IN_PRECOMPILED(optimizing_background_compiler_ = new BackgroundCompiler(this));
}
```

#### 2.11.2 Heap::Init

[-> third_party/dart/runtime/vm/heap/heap.cc]

```Java
void Heap::Init(Isolate* isolate,
                intptr_t max_new_gen_words,
                intptr_t max_old_gen_words) {
  //创建堆Heap
  Heap* heap = new Heap(isolate, max_new_gen_words, max_old_gen_words);
  isolate->set_heap(heap);
}
```

#### 2.11.3 Thread::EnterIsolate

[-> third_party/dart/runtime/vm/thread.cc]

```Java
bool Thread::EnterIsolate(Isolate* isolate) {
  const bool kIsMutatorThread = true;
  Thread* thread = isolate->ScheduleThread(kIsMutatorThread);
  if (thread != NULL) {
    thread->task_kind_ = kMutatorTask;
    thread->StoreBufferAcquire();
    if (isolate->marking_stack() != NULL) {
      thread->MarkingStackAcquire();
      thread->DeferredMarkingStackAcquire();
    }
    return true;
  }
  return false;
}
```

#### 2.11.4 IsolateMessageHandler初始化

[-> third_party/dart/runtime/vm/isolate.cc]

```Java
class IsolateMessageHandler : public MessageHandler {
 public:
  explicit IsolateMessageHandler(Isolate* isolate);
}
```

IsolateMessageHandler继承于MessageHandler，MessageHandler中有两个比较重要的成员变量queue_和oob_queue_，用于记录普通消息和oob消息的队列。

#### 2.11.5 PortMap::CreatePort

[-> third_party/dart/runtime/vm/port.cc]

```Java
Dart_Port PortMap::CreatePort(MessageHandler* handler) {
  MutexLocker ml(mutex_);

  Entry entry;
  entry.port = AllocatePort();  //采用随机数生成一个整型的端口号
  entry.handler = handler;
  entry.state = kNewPort;

  intptr_t index = entry.port % capacity_;
  Entry cur = map_[index];
  while (cur.port != 0) {
    index = (index + 1) % capacity_;
    cur = map_[index];
  }

  if (map_[index].handler == deleted_entry_) {
    deleted_--;
  }
  map_[index] = entry;

  used_++;
  MaintainInvariants();
  return entry.port;
}
```

map_是一个记录端口entry的HashMap，每一个entry里面有端口号，handler，以及端口状态。

- 端口号port采用的是用随机数生成一个整型的端口号，
- handler是并记录handler指针；
- 端口状态state有3种类型，包括kNewPort(新分配的端口)，kLivePort(普通端口)，kControlPort(特殊控制类的端口)

##  三、创建Isolate

### 3.1 DartIsolate::CreateRootIsolate

[-> flutter/runtime/dart_isolate.cc]

```Java
std::weak_ptr<DartIsolate> DartIsolate::CreateRootIsolate(
    const Settings& settings,
    fml::RefPtr<const DartSnapshot> isolate_snapshot,
    fml::RefPtr<const DartSnapshot> shared_snapshot,
    TaskRunners task_runners,
    std::unique_ptr<Window> window,
    fml::WeakPtr<SnapshotDelegate> snapshot_delegate,
    fml::WeakPtr<IOManager> io_manager,
    std::string advisory_script_uri,
    std::string advisory_script_entrypoint,
    Dart_IsolateFlags* flags) {
  TRACE_EVENT0("flutter", "DartIsolate::CreateRootIsolate");
  Dart_Isolate vm_isolate = nullptr;
  std::weak_ptr<DartIsolate> embedder_isolate;

  //创建DartIsolate[见小节3.2]
  auto root_embedder_data = std::make_unique<std::shared_ptr<DartIsolate>>(
      std::make_shared<DartIsolate>(
          settings, std::move(isolate_snapshot),
          std::move(shared_snapshot), task_runners,                 
          std::move(snapshot_delegate), std::move(io_manager),        
          advisory_script_uri, advisory_script_entrypoint,    
          nullptr));
  //[见小节3.4]
  std::tie(vm_isolate, embedder_isolate) = CreateDartVMAndEmbedderObjectPair(
      advisory_script_uri.c_str(), advisory_script_entrypoint.c_str(),  
      nullptr, nullptr, flags,                               
      root_embedder_data.get(), true, &error);

  std::shared_ptr<DartIsolate> shared_embedder_isolate = embedder_isolate.lock();
  if (shared_embedder_isolate) {
    //只有root isolates能和window交互
    shared_embedder_isolate->SetWindow(std::move(window));
  }
  root_embedder_data.release();
  return embedder_isolate;
}
```

### 3.2 DartIsolate初始化

[-> flutter/runtime/dart_isolate.cc]

```Java
DartIsolate::DartIsolate(const Settings& settings,
                         fml::RefPtr<const DartSnapshot> isolate_snapshot,
                         fml::RefPtr<const DartSnapshot> shared_snapshot,
                         TaskRunners task_runners,
                         fml::WeakPtr<SnapshotDelegate> snapshot_delegate,
                         fml::WeakPtr<IOManager> io_manager,
                         std::string advisory_script_uri,
                         std::string advisory_script_entrypoint,
                         ChildIsolatePreparer child_isolate_preparer)
    : UIDartState(std::move(task_runners),
                  settings.task_observer_add,
                  settings.task_observer_remove,
                  std::move(snapshot_delegate),
                  std::move(io_manager),
                  advisory_script_uri,
                  advisory_script_entrypoint,
                  settings.log_tag,
                  settings.unhandled_exception_callback,
                  DartVMRef::GetIsolateNameServer()),
      settings_(settings),
      isolate_snapshot_(std::move(isolate_snapshot)),
      shared_snapshot_(std::move(shared_snapshot)),
      //当isolate准备运行的时候，会设置子isolate preparer
      child_isolate_preparer_(std::move(child_isolate_preparer)) {
  phase_ = Phase::Uninitialized;
}
```

该方法说明：

- 这是root isolate，故需要伪造一个父embedder对象；
- isolate生命周期完全由VM管理；
- 将settings的task_observer_add和task_observer_remove传递给UIDartState的add_callback和remove_callback；

### 3.3 UIDartState初始化

[-> flutter/lib/ui/ui_dart_state.cc]

```Java
UIDartState::UIDartState(
    TaskRunners task_runners,
    TaskObserverAdd add_callback,
    TaskObserverRemove remove_callback,
    fml::WeakPtr<SnapshotDelegate> snapshot_delegate,
    fml::WeakPtr<IOManager> io_manager,
    std::string advisory_script_uri,
    std::string advisory_script_entrypoint,
    std::string logger_prefix,
    UnhandledExceptionCallback unhandled_exception_callback,
    std::shared_ptr<IsolateNameServer> isolate_name_server)
    : task_runners_(std::move(task_runners)),
      add_callback_(std::move(add_callback)),
      remove_callback_(std::move(remove_callback)),
      snapshot_delegate_(std::move(snapshot_delegate)),
      io_manager_(std::move(io_manager)),
      advisory_script_uri_(std::move(advisory_script_uri)),
      advisory_script_entrypoint_(std::move(advisory_script_entrypoint)),
      logger_prefix_(std::move(logger_prefix)),
      unhandled_exception_callback_(unhandled_exception_callback),
      isolate_name_server_(std::move(isolate_name_server)) {
  //添加task observer [见小节3.3.1]
  AddOrRemoveTaskObserver(true /* add */);
}

UIDartState::~UIDartState() {
  //移除task observer
  AddOrRemoveTaskObserver(false ;
}
```

在UIDartState对象创建时添加task observer，该对象销毁时移除task observer。

#### 3.3.1 AddOrRemoveTaskObserver

[-> flutter/lib/ui/ui_dart_state.cc]

```Java
void UIDartState::AddOrRemoveTaskObserver(bool add) {
  auto task_runner = task_runners_.GetUITaskRunner();
  ...
  if (add) {
    //[见小节3.3.3]
    add_callback_(reinterpret_cast<intptr_t>(this),
                  [this]() { this->FlushMicrotasksNow(); });
  } else {
    remove_callback_(reinterpret_cast<intptr_t>(this));
  }
}
```

根据[小节3.2]传递的参数，可知add_callback_等于settings.task_observer_add，再来看看settings.task_observer_add到底是什么。

#### 3.3.2 FlutterMain::Init

[-> flutter/shell/platform/android/flutter_main.cc]

```Java
void FlutterMain::Init(...) {
  ...
  //初始化observer的增加和删除方法
  settings.task_observer_add = [](intptr_t key, fml::closure callback) {
    fml::MessageLoop::GetCurrent().AddTaskObserver(key, std::move(callback));
  };

  settings.task_observer_remove = [](intptr_t key) {
    fml::MessageLoop::GetCurrent().RemoveTaskObserver(key);
  };
}
```

在Flutter引擎启动时执行FlutterActivity的onCreate()过程，会调用FlutterMain::Init()方法。由此可见，settings的task_observer_add和task_observer_remove分别对应MessageLoopImpl的AddTaskObserver()和RemoveTaskObserver()方法。

#### 3.3.3 AddTaskObserver

[-> flutter/fml/message_loop_impl.cc]

```Java
void MessageLoopImpl::AddTaskObserver(intptr_t key, fml::closure callback) {
  task_observers_[key] = std::move(callback);
}

void MessageLoopImpl::RemoveTaskObserver(intptr_t key) {
  task_observers_.erase(key);
}
```

task_observers_是一个map类型，以UIDartState实例对象为key，以FlushMicrotasksNow()方法为value。该过程task_observers_中新增了一个FlushMicrotasksNow()方法，用于在MessageLoop过程来消费Microtask。即task_observers_的子项observer.second()对应的是FlushMicrotasksNow()方法。

### 3.4 CreateDartVMAndEmbedderObjectPair

[-> flutter/runtime/dart_isolate.cc]

```Java
std::pair<Dart_Isolate, std::weak_ptr<DartIsolate>>  
DartIsolate::CreateDartVMAndEmbedderObjectPair(
    const char* advisory_script_uri,
    const char* advisory_script_entrypoint,
    const char* package_root,
    const char* package_config,
    Dart_IsolateFlags* flags,
    std::shared_ptr<DartIsolate>* p_parent_embedder_isolate,
    bool is_root_isolate,
    char** error) {
  TRACE_EVENT0("flutter", "DartIsolate::CreateDartVMAndEmbedderObjectPair");

  std::unique_ptr<std::shared_ptr<DartIsolate>> embedder_isolate(p_parent_embedder_isolate);
  if (!is_root_isolate) {
    ...
  }

  // [见小节3.5]
  Dart_Isolate isolate = Dart_CreateIsolate(
      advisory_script_uri, advisory_script_entrypoint,  
      (*embedder_isolate)->GetIsolateSnapshot()->GetData()->GetSnapshotPointer(),
      (*embedder_isolate)->GetIsolateSnapshot()->GetInstructionsIfPresent(),
      (*embedder_isolate)->GetSharedSnapshot()->GetDataIfPresent(),
      (*embedder_isolate)->GetSharedSnapshot()->GetInstructionsIfPresent(),
      flags, embedder_isolate.get(), error);
  ...
  // [见小节3.7]
  if (!(*embedder_isolate)->Initialize(isolate, is_root_isolate)) {
    return {nullptr, {}};
  }
  // [见小节3.8]
  if (!(*embedder_isolate)->LoadLibraries(is_root_isolate)) {
    return {nullptr, {}};
  }
  //获取DartIsolate的弱引用
  auto weak_embedder_isolate = (*embedder_isolate)->GetWeakIsolatePtr();
  ...
  embedder_isolate.release();
  //embedder的所有权由Dart VM控制，因此返回给调用者的是弱引用
  return {isolate, weak_embedder_isolate};
}
```

### 3.5 Dart_CreateIsolate

[-> third_party/dart/runtime/vm/dart_api_impl.cc]

```Java
DART_EXPORT Dart_Isolate
Dart_CreateIsolate(const char* script_uri,
                   const char* name,
                   const uint8_t* snapshot_data,
                   const uint8_t* snapshot_instructions,
                   const uint8_t* shared_data,
                   const uint8_t* shared_instructions,
                   Dart_IsolateFlags* flags,
                   void* callback_data,
                   char** error) {
  API_TIMELINE_DURATION(Thread::Current());
  //[见小节3.6]
  return CreateIsolate(script_uri, name, snapshot_data, snapshot_instructions,
                       shared_data, shared_instructions, NULL, 0, flags,
                       callback_data, error);
}
```

### 3.6 CreateIsolate

[-> third_party/dart/runtime/vm/dart_api_impl.cc]

```Java
static Dart_Isolate CreateIsolate(const char* script_uri,
                                  const char* name,
                                  const uint8_t* snapshot_data,
                                  const uint8_t* snapshot_instructions,
                                  const uint8_t* shared_data,
                                  const uint8_t* shared_instructions,
                                  const uint8_t* kernel_buffer,
                                  intptr_t kernel_buffer_size,
                                  Dart_IsolateFlags* flags,
                                  void* callback_data,
                                  char** error) {
  Dart_IsolateFlags api_flags;
  if (flags == NULL) {
    Isolate::FlagsInitialize(&api_flags);  //初始化flags
    flags = &api_flags;
  }
  // [见小节3.6.1]
  Isolate* I = Dart::CreateIsolate((name == NULL) ? "isolate" : name, *flags);
  ...
  {
    Thread* T = Thread::Current();
    StackZone zone(T);
    HANDLESCOPE(T);
    T->EnterApiScope();
    const Error& error_obj = Error::Handle(Z,
        // [见小节3.6.2]
        Dart::InitializeIsolate(snapshot_data, snapshot_instructions,
                                shared_data, shared_instructions, kernel_buffer,
                                kernel_buffer_size, callback_data));
    if (error_obj.IsNull()) {
      T->ExitApiScope();
      T->set_execution_state(Thread::kThreadInNative);
      T->EnterSafepoint();
      ...
      return Api::CastIsolate(I);
    }
    ...
    T->ExitApiScope();
  }
  Dart::ShutdownIsolate(); //创建错误，则会关闭Isolate
  return reinterpret_cast<Dart_Isolate>(NULL);
}
```

#### 3.6.1 Dart::CreateIsolate

[-> third_party/dart/runtime/vm/dart.cc]

```Java
Isolate* Dart::CreateIsolate(const char* name_prefix,
                             const Dart_IsolateFlags& api_flags) {
  //创建Isolate [见小节2.11]
  Isolate* isolate = Isolate::InitIsolate(name_prefix, api_flags);
  return isolate;
}
```

关于Isolate的创建方法见[小节2.11]，每次创建RuntimeController对象，都会创建相应的Root Isolate。

#### 3.6.2 Dart::InitializeIsolate

[-> third_party/dart/runtime/vm/dart.cc]

```Java
RawError* Dart::InitializeIsolate(const uint8_t* snapshot_data,
                                  const uint8_t* snapshot_instructions,
                                  const uint8_t* shared_data,
                                  const uint8_t* shared_instructions,
                                  const uint8_t* kernel_buffer,
                                  intptr_t kernel_buffer_size,
                                  void* data) {
  Thread* T = Thread::Current();
  Isolate* I = T->isolate();
  StackZone zone(T);
  HandleScope handle_scope(T);
  ObjectStore::Init(I);

  Error& error = Error::Handle(T->zone());
  error = Object::Init(I, kernel_buffer, kernel_buffer_size);

  if ((snapshot_data != NULL) && kernel_buffer == NULL) {
    //读取snapshot，并初始化状态
    const Snapshot* snapshot = Snapshot::SetupFromBuffer(snapshot_data);
    ...
    FullSnapshotReader reader(snapshot, snapshot_instructions, shared_data,
                              shared_instructions, T);
    //读取isolate的Snapshot
    const Error& error = Error::Handle(reader.ReadIsolateSnapshot());
    ReversePcLookupCache::BuildAndAttachToIsolate(I);
  } else {
    ...
  }

  Object::VerifyBuiltinVtables();
  ...

  const Code& miss_code = Code::Handle(I->object_store()->megamorphic_miss_code());
  I->set_ic_miss_code(miss_code);
  I->heap()->InitGrowthControl();
  I->set_init_callback_data(data);
  Api::SetupAcquiredError(I);
  //名为"vm-service"的Isolate 则为ServiceIsolate
  ServiceIsolate::MaybeMakeServiceIsolate(I);
  //发送Isolate启动的消息
  ServiceIsolate::SendIsolateStartupMessage();

  //创建tag表
  I->set_tag_table(GrowableObjectArray::Handle(GrowableObjectArray::New()));
  //设置默认的UserTag
  const UserTag& default_tag = UserTag::Handle(UserTag::DefaultTag());
  I->set_current_tag(default_tag);
  return Error::null();
}
```

到这里Isolate的创建过程便真正完成。

### 3.7 DartIsolate::Initialize

[-> flutter/runtime/dart_isolate.cc]

```Java
bool DartIsolate::Initialize(Dart_Isolate dart_isolate, bool is_root_isolate) {
  ...
  auto* isolate_data = static_cast<std::shared_ptr<DartIsolate>*>(
                                  Dart_IsolateData(dart_isolate));
  SetIsolate(dart_isolate);
  //保存当前的isolate，进入新的scope
  Dart_ExitIsolate();

  tonic::DartIsolateScope scope(isolate());
  //[见小节3.7.1]
  SetMessageHandlingTaskRunner(GetTaskRunners().GetUITaskRunner(),
                               is_root_isolate);

  if (tonic::LogIfError(
          Dart_SetLibraryTagHandler(tonic::DartState::HandleLibraryTag))) {
    return false;
  }
  //更新ui/gpu/io/platform的线程名
  if (!UpdateThreadPoolNames()) {
    return false;
  }

  phase_ = Phase::Initialized;
  return true;
}
```

#### 3.7.1 DartIsolate::SetMessageHandlingTaskRunner

[-> flutter/runtime/dart_isolate.cc]

```Java
void DartIsolate::SetMessageHandlingTaskRunner(
    fml::RefPtr<fml::TaskRunner> runner, bool is_root_isolate) {
  //只有root isolate才会执行该过程
  if (!is_root_isolate || !runner) {
    return;
  }
  message_handling_task_runner_ = runner;

  //[见小节3.7.2]
  message_handler().Initialize([runner](std::function<void()> task) { runner->PostTask(task); });
}
```

这里需要重点注意的是，只有root isolate才会执行Initialize()过程。

#### 3.7.2 DartMessageHandler::Initialize

[-> third_party/tonic/dart_message_handler.cc]

```Java
void DartMessageHandler::Initialize(TaskDispatcher dispatcher) {
  //设置task_dispatcher_
  task_dispatcher_ = dispatcher;
  //设置message_notify_callback
  Dart_SetMessageNotifyCallback(MessageNotifyCallback);
}
```

由此可见：

- task_dispatcher_值等价于UITaskRunner->PostTask()，执行的是向UI线程PostTask的操作；
- message_notify_callback值等价于DartMessageHandler::MessageNotifyCallback；

### 3.8 DartIsolate::LoadLibraries

[-> flutter/runtime/dart_isolate.cc]

```Java
bool DartIsolate::LoadLibraries(bool is_root_isolate) {
  TRACE_EVENT0("flutter", "DartIsolate::LoadLibraries");
  if (phase_ != Phase::Initialized) {
    return false;
  }

  tonic::DartState::Scope scope(this);

  DartIO::InitForIsolate(); //[见小节3.8.1]

  DartUI::InitForIsolate(is_root_isolate);  //[见小节3.8.4]

  const bool is_service_isolate = Dart_IsServiceIsolate(isolate());
    //[见小节3.8.5]
  DartRuntimeHooks::Install(is_root_isolate && !is_service_isolate,
                            GetAdvisoryScriptURI());

  if (!is_service_isolate) {
    class_library().add_provider(
        "ui", std::make_unique<tonic::DartClassProvider>(this, "dart:ui"));
  }

  phase_ = Phase::LibrariesSetup;
  return true;
}
```

#### 3.8.1 DartIO::InitForIsolate

[-> flutter/lib/io/dart_io.cc]

```Java
void DartIO::InitForIsolate() {
  //[见小节3.8.1]
  Dart_Handle result = Dart_SetNativeResolver(
      Dart_LookupLibrary(ToDart("dart:io")),
          dart::bin::LookupIONative, dart::bin::LookupIONativeSymbol);
}
```

来看看Dart_SetNativeResolver和Dart_LookupLibrary的实现。

#### 3.8.2 Dart_SetNativeResolver

[-> third_party/dart/runtime/vm/dart_api_impl.cc]

```Java
DART_EXPORT Dart_Handle
Dart_SetNativeResolver(Dart_Handle library,
                       Dart_NativeEntryResolver resolver,
                       Dart_NativeEntrySymbol symbol) {
  DARTSCOPE(Thread::Current());
  const Library& lib = Api::UnwrapLibraryHandle(Z, library);
  lib.set_native_entry_resolver(resolver);
  lib.set_native_entry_symbol_resolver(symbol);
  return Api::Success();
}
```

#### 3.8.3 Dart_LookupLibrary

[-> third_party/dart/runtime/vm/dart_api_impl.cc]

```Java
DART_EXPORT Dart_Handle Dart_LookupLibrary(Dart_Handle url) {
  DARTSCOPE(Thread::Current());
  const String& url_str = Api::UnwrapStringHandle(Z, url);

  const Library& library = Library::Handle(Z, Library::LookupLibrary(T, url_str));
  if (library.IsNull()) {
    ...
  } else {
    return Api::NewHandle(T, library.raw());
  }
}
```

#### 3.8.4 DartUI::InitForIsolate

[-> flutter/lib/ui/dart_ui.cc]

```Java
void DartUI::InitForIsolate(bool is_root_isolate) {
  auto get_native_function =
      is_root_isolate ? GetNativeFunction : GetNativeFunctionSecondary;
  auto get_symbol = is_root_isolate ? GetSymbol : GetSymbolSecondary;
  Dart_Handle result = Dart_SetNativeResolver(
      Dart_LookupLibrary(ToDart("dart:ui")), get_native_function, get_symbol);
}
```

#### 3.8.5 DartRuntimeHooks::Install

[-> flutter/lib/ui/dart_runtime_hooks.cc]

```Java
void DartRuntimeHooks::Install(bool is_ui_isolate,
                               const std::string& script_uri) {
  Dart_Handle builtin = Dart_LookupLibrary(ToDart("dart:ui"));
  InitDartInternal(builtin, is_ui_isolate);
  InitDartCore(builtin, script_uri);
  InitDartAsync(builtin, is_ui_isolate);
  InitDartIO(builtin, script_uri);
}
```

## 四、运行DartIsolate

### 4.1 DartIsolate::Run

[-> flutter/runtime/dart_isolate.cc]

```Java
bool DartIsolate::Run(const std::string& entrypoint_name, fml::closure on_run) {
  TRACE_EVENT0("flutter", "DartIsolate::Run");
  ...

  tonic::DartState::Scope scope(this);
  //获取用户入口函数，也就是主函数main()
  auto user_entrypoint_function =
      Dart_GetField(Dart_RootLibrary(), tonic::ToDart(entrypoint_name.c_str()));
  //[见小节4.2]
  if (!InvokeMainEntrypoint(user_entrypoint_function)) {
    return false;
  }
  phase_ = Phase::Running;

  if (on_run) {
    on_run();
  }
  return true;
}
```

### 4.2 InvokeMainEntrypoint

[-> flutter/runtime/dart_isolate.cc]

```Java
static bool InvokeMainEntrypoint(Dart_Handle user_entrypoint_function) {
  // [见小节4.2.1]
  Dart_Handle start_main_isolate_function =
      tonic::DartInvokeField(Dart_LookupLibrary(tonic::ToDart("dart:isolate")),
                             "_getStartMainIsolateFunction", {});

  //[见小节4.3]
  if (tonic::LogIfError(tonic::DartInvokeField(
          Dart_LookupLibrary(tonic::ToDart("dart:ui")), "_runMainZoned",
          {start_main_isolate_function, user_entrypoint_function}))) {
    return false;
  }
  return true;
}
```

经过Dart虚拟机最终会调用的Dart层的_runMainZoned()方法，其中参数start_main_isolate_function等于_startIsolate。

#### 4.2.1 _getStartMainIsolateFunction

[-> third_party/dart/runtime/lib/isolate_patch.dart]

```Java
@pragma("vm:entry-point", "call")
Function _getStartMainIsolateFunction() {
  return _startMainIsolate;
}

@pragma("vm:entry-point", "call")
void _startMainIsolate(Function entryPoint, List<String> args) {
  _startIsolate(
      null, // no parent port
      entryPoint,
      args,
      null, // no message
      true, // isSpawnUri
      null, // no control port
      null); // no capabilities
}
```

### 4.3 _runMainZoned

[-> flutter/lib/ui/hooks.dart]

```Java
void _runMainZoned(Function startMainIsolateFunction, Function userMainFunction) {
   //[见小节4.4]
  startMainIsolateFunction((){
    runZoned<Future<void>>(() {
      const List<String> empty_args = <String>[];
      if (userMainFunction is _BinaryFunction) {
        (userMainFunction as dynamic)(empty_args, '');
      } else if (userMainFunction is _UnaryFunction) {
        (userMainFunction as dynamic)(empty_args);
      } else {
        userMainFunction();  //[见小节4.5]
      }
    }, onError: (Object error, StackTrace stackTrace) {
      _reportUnhandledException(error.toString(), stackTrace.toString());
    });
  }, null);
}
```

该方法的startMainIsolateFunction等于_startIsolate，userMainFunction等于main.dart文件中的main()方法，也就是整个Dart业务代码。

### 4.4 _startIsolate

[-> third_party/dart/runtime/lib/isolate_patch.dart]

```Java
@pragma("vm:entry-point", "call")
void _startIsolate(
    SendPort parentPort,
    Function entryPoint,
    List<String> args,
    var message,
    bool isSpawnUri,
    RawReceivePort controlPort,
    List capabilities) {
  //控制端口(也称主isolate端口)不处理任何消息
  if (controlPort != null) {
    controlPort.handler = (_) {};
  }
  ...

  // 将所有用户代码处理延迟到下一次消息循环运行。 这允许拦截事件调度中的某些条件，例如在暂停状态下开始。
  RawReceivePort port = new RawReceivePort();
  port.handler = (_) {
    port.close();

    if (isSpawnUri) {
      if (entryPoint is _BinaryFunction) {
        (entryPoint as dynamic)(args, message);
      } else if (entryPoint is _UnaryFunction) {
        (entryPoint as dynamic)(args);
      } else {
        entryPoint(); //[见小节4.5]
      }
    } else {
      entryPoint(message);
    }
  };
  //确保消息handler已触发
  port.sendPort.send(null);
}
```

此处entryPoint便是runZoned，然后几次调用后，回到_runMainZoned()方法的userMainFunction，而userMainFunction

### 4.5 main

[-> lib/main.dart]

```
void main() => runApp(Widget app);
```

![runzoned](http://gityuan.com/img/flutter_boot/runzoned.png)

也就是说FlutterActivity.onCreate()方法，经过层层调用后开始执行dart层的main()方法，执行runApp()的过程，这便开启执行整个Dart业务代码。

## 附录

```Java
third_party/dart/runtime/vm/
  - dart_api_impl.cc
  - dart.cc
  - port.cc
  - isolate.cc
  - heap/heap.cc
  - thread.cc
  - thread_pool.cc
  - virtual_memory_posix.cc
  - os_thread.cc

third_party/dart/runtime/bin/
  - dart_io_api_impl.cc
  - eventhandler.cc
  - socket.cc
  - socket.h
  - thread_android.cc
  - eventhandler.h
  - eventhandler_android.h
  - eventhandler_android.cc
  - thread_android.cc
  - eventhandler_android.cc
  - eventhandler.cc

third_party/tonic/
  - dart_library_natives.h
  - dart_library_natives.cc

flutter/runtime/
  - dart_isolate.cc
  - dart_vm_lifecycle.cc
  - dart_vm.cc
  - dart_vm_data.cc

flutter/lib/ui/
  - ui_dart_state.cc
  - dart_ui.cc
  - dart_runtime_hooks.cc
  - hooks.dart

third_party/dart/runtime/lib/isolate_patch.dart
flutter/shell/platform/android/flutter_main.cc
flutter/fml/message_loop_impl.cc
flutter/lib/io/dart_io.cc
```