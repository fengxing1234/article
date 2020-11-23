---
title: Android系统总结篇
date: 2020-05-29 13:38:37
tags: 
	- Android系统总结篇
categories: 
	- Android
	- Android系统分析
---

# 总结

## Android 系统架构

Android大致可以分为四层架构：**Linux内核层、系统运行库层、应用框架层和应用层**。

本文作为Android系统架构的开篇，起到提纲挈领的作用，从系统整体架构角度概要讲解Android系统的核心技术点，带领大家初探Android系统全貌以及内部运作机制。虽然Android系统非常庞大且错综复杂，需要具备全面的技术栈，但整体架构设计清晰。Android底层内核空间以Linux Kernel作为基石，上层用户空间由Native系统库、虚拟机运行环境、框架层组成，通过系统调用(Syscall)连通系统的内核空间与用户空间。对于用户空间主要采用C++和Java代码编写，通过JNI技术打通用户空间的Java层和Native层(C++/C)，从而连通整个系统。

![image.png](https://upload-images.jianshu.io/upload_images/4118241-a8c2d9d3a1a3a95b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- Linux内核层

  Android系统是基于Linux内核的，这一层为Android设备的各种硬件提供了底层的驱动，如显示驱动、音频驱动、照相机驱动、蓝牙驱动、Wi-Fi驱动、电源管理等。

- 系统运行库层

  这一层通过一些C/C++库来为Android系统提供了主要的特性支持。如SQLite库提供了数据库的支持，OpenGl|ES库提供了3D绘图的支持，Webkit库提供了浏览器内核的支持等。

  同样这一层还有Android运行时库，它主要提供了一些核心库，能够允许开发者使用Java语言来编写Android应用。另外，Android运行库中还包含了Dalvik虚拟机（5.0系统之后改为ART运行环境），它使得每一个Android应用都能运行在独立的进程中，并且拥有一个自己的Dalvik虚拟机示例。相较于Java虚拟机，Dalvik是专门为移动设备定制的，它针对手机内存、CPU性能有限等情况做了优化处理。

- 应用框架层

  这一层主要提供了构建应用程序时可能用到的各种API,Andorid自带的一些核心应用就是使用这些API完成的，开发者也可以通过使用这些API来构建自己的应用程序。

- 应用层

  所有安装在手机上的应用程序都是属于这一层的，比如系统自带的联系人、短信等程序，或者是你从Google Play上下载的小游戏，当然还包括你自己开发的程序。

  ![脑图](https://user-gold-cdn.xitu.io/2019/12/22/16f2c322e9e0eea1?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

## 系统启动架构

Google提供的5层架构图很经典，但为了更进一步透视Android系统架构，本文更多的是以进程的视角，以分层的架构来诠释Android系统的全貌，阐述Android内部的环环相扣的内在联系。

![image.png](https://upload-images.jianshu.io/upload_images/4118241-68a5968ed562a20e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)





![image.png](https://upload-images.jianshu.io/upload_images/4118241-f03913fd803c3125.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## Init进程

init进程是Linux系统中用户空间的第一个进程，进程号固定为1。Kernel启动后，在用户空间启动init进程，并调用init中的main()方法执行init进程。对于init进程的功能分为4部分：

- 创建一块共享的内存空间，用于属性服务器;

- 解析各个rc文件，并启动相应属性服务进程;
- 初始化epoll，依次设置signal、property、keychord这3个fd可读时相对应的回调函数;
- 进入无限循环状态，执行如下流程：
  - 检查action_queue列表是否为空，若不为空则执行相应的action;
  - 检查是否需要重启的进程，若有则将其重新启动;
  - 进入epoll_wait等待状态，直到系统属性变化事件(property_set改变属性值)，或者收到子进程的信号SIGCHLD，再或者keychord 键盘输入事件，则会退出等待状态，执行相应的回调函数。

可见init进程在开机之后的核心工作就是响应property变化事件和回收僵尸进程。当某个进程调用property_set来改变一个系统属性值时，系统会通过socket向init进程发送一个property变化的事件通知，那么property fd会变成可读，init进程采用epoll机制监听该fd则会 触发回调handle_property_set_fd()方法。回收僵尸进程，在Linux内核中，如父进程不等待子进程的结束直接退出，会导致子进程在结束后变成僵尸进程，占用系统资源。为此，init进程专门安装了SIGCHLD信号接收器，当某些子进程退出时发现其父进程已经退出，则会向init进程发送SIGCHLD信号，init进程调用回调方法handle_signal()来回收僵尸子进程。

### 启动zygone服务

- 创建一个名叫”zygote”的service结构体；
- 创建一个用于socket通信的socketinfo结构体；
- 创建一个包含4个onrestart的action结构体。

![zygote_init](http://gityuan.com/images/boot/init/zygote_init.jpg)

## Zygote进程

Zygote启动过程的函数调用类大致流程如下：

![zygote_process](http://gityuan.com/images/boot/zygote/zygote_process.jpg)

![zygote_start](http://gityuan.com/images/boot/zygote/zygote_start.jpg)

1. 解析init.zygote.rc中的参数，创建AppRuntime并调用AppRuntime.start()方法；
2. 调用AndroidRuntime的startVM()方法创建虚拟机，再调用startReg()注册JNI函数；
3. 通过JNI方式调用ZygoteInit.main()，第一次进入Java世界；
4. registerZygoteSocket()建立socket通道，zygote作为通信的服务端，用于响应客户端请求；
5. preload()预加载通用类、drawable和color资源、openGL以及共享库以及WebView，用于提高app启动效率；
6. zygote完毕大部分工作，接下来再通过startSystemServer()，fork得力帮手system_server进程，也是上层framework的运行载体。
7. zygote功成身退，调用runSelectLoop()，随时待命，当接收到请求创建新进程请求时立即唤醒并执行相应工作。

**Zygote做了哪些事：**

- 创建vm虚拟机
- 注册JNI函数
- 建立socket通道，响应客户端请求
- 预加载通用类、资源、openGL、共享库、WebView
- 创建system_server进程
- 调用runSelectLoop()，当接收到请求创建新进程请求时立即唤醒并执行相应工作，没有请求则休眠。

### Zygote创建进程

进程创建过程的简要图：

![start_app_process](http://gityuan.com/images/android-process/start_app_process.jpg)

Process.start()方法是阻塞操作，等待直到进程创建完成并返回相应的新进程pid，才完成该方法。

当App第一次启动时或者启动远程Service，即AndroidManifest.xml文件中定义了process:remote属性时，都需要创建进程。比如当用户点击桌面的某个App图标，桌面本身是一个app（即Launcher App），那么Launcher所在进程便是这次创建新进程的发起进程，该通过binder发送消息给system_server进程，该进程承载着整个java framework的核心服务。system_server进程从Process.start开始，执行创建进程，流程图（以进程的视角）如下：

![process-create](http://gityuan.com/images/android-process/process-create.jpg)

上图中，`system_server`进程通过socket IPC通道向`zygote`进程通信，`zygote`在fork出新进程后由于fork**调用一次，返回两次**，即在zygote进程中调用一次，在zygote进程和子进程中各返回一次，从而能进入子进程来执行代码。该调用流程图的过程：

1. **system_server进程**（`即流程1~3`）：通过Process.start()方法发起创建新进程请求，会先收集各种新进程uid、gid、nice-name等相关的参数，然后通过socket通道发送给zygote进程；
2. **zygote进程：**接收到system_server进程发送过来的参数后封装成Arguments对象，图中绿色框forkAndSpecialize()方法是进程创建过程中最为核心的一个环节其具体工作是依次执行下面的3个方法：
   - preFork()：先停止Zygote的4个Daemon子线程（java堆内存整理线程、对线下引用队列线程、析构线程以及监控线程）的运行以及初始化gc堆；
   - nativeForkAndSpecialize()：调用linux的fork()出新进程，创建Java堆处理的线程池，重置gc性能数据，设置进程的信号处理函数，启动JDWP线程；
   - postForkCommon()：在启动之前被暂停的4个Daemon子线程。
3. **新进程**：进入handleChildProc()方法，设置进程名，打开binder驱动，启动新的binder线程；然后设置art虚拟机参数，再反射调用目标类的main()方法，即Activity.main()方法。

## system_server进程启动

SystemServer由Zygote fork生成的，进程名为`system_server`，该进程承载着framework的核心服务。 [Android系统启动-zygote篇](http://gityuan.com/2016/02/13/android-zygote/)中讲到Zygote启动过程中会调用startSystemServer()，可知`startSystemServer()`函数是system_server启动流程的起点， 启动流程图如下：![system_server_boot_process](http://gityuan.com/images/boot/systemServer/system_server.jpg)

上图前4步骤（即颜色为紫色的流程）运行在是`Zygote`进程，从第5步（即颜色为蓝色的流程）ZygoteInit.handleSystemServerProcess开始是运行在新创建的`system_server`，这是fork机制实现的（fork会返回2次）。下面从startSystemServer()开始讲解详细启动流程。

在`SystemServer#main`方法中进入`Looper.loop()`状态,等待其他线程通过`handler`发送消息到主线再处理.

system_server进程，从源码角度划分为引导服务、核心服务、其他服务3类。

- 引导服务(7个)：ActivityManagerService、PowerManagerService、LightsService、DisplayManagerService、PackageManagerService、UserManagerService、SensorService；
- 核心服务(3个)：BatteryService、UsageStatsService、WebViewUpdateService；
- 其他服务(70个+)：AlarmManagerService、VibratorService等。

合计总大约80个系统服务：

| `ActivityManagerService`      | `PackageManagerService`    | `WindowManagerService`         |
| ----------------------------- | -------------------------- | ------------------------------ |
| `PowerManagerService`         | `BatteryService`           | `BatteryStatsService`          |
| `DreamManagerService`         | `DropBoxManagerService`    | `SamplingProfilerService`      |
| `UsageStatsService`           | `DiskStatsService`         | `DeviceStorageMonitorService`  |
| SchedulingPolicyService       | `AlarmManagerService`      | DeviceIdleController           |
| ThermalObserver               | JobSchedulerService        | `AccessibilityManagerService`  |
| DisplayManagerService         | LightsService              | `GraphicsStatsService`         |
| StatusBarManagerService       | NotificationManagerService | WallpaperManagerService        |
| UiModeManagerService          | AppWidgetService           | LauncherAppsService            |
| TextServicesManagerService    | ContentService             | LockSettingsService            |
| InputMethodManagerService     | InputManagerService        | `MountService`                 |
| FingerprintService            | TvInputManagerService      | DockObserver                   |
| NetworkManagementService      | NetworkScoreService        | `NetworkStatsService`          |
| NetworkPolicyManagerService   | ConnectivityService        | BluetoothService               |
| WifiP2pService                | WifiService                | WifiScanningService            |
| AudioService                  | MediaRouterService         | VoiceInteractionManagerService |
| MediaProjectionManagerService | MediaSessionService        |                                |
| DevicePolicyManagerService    | PrintManagerService        | `BackupManagerService`         |
| `UserManagerService`          | AccountManagerService      | `TrustManagerService`          |
| `SensorService`               | LocationManagerService     | VibratorService                |
| CountryDetectorService        | GestureLauncherService     | PersistentDataBlockService     |
| EthernetService               | WebViewUpdateService       | ClipboardService               |
| TelephonyRegistry             | TelecomLoaderService       | NsdService                     |
| UpdateLockService             | SerialService              | SearchManagerService           |
| CommonTimeManagementService   | AssetAtlasService          | ConsumerIrService              |
| MidiServiceCameraService      | TwilightService            | RestrictionsManagerService     |
| MmsServiceBroker              | RttService                 | UsbService                     |

Service类别众多，其中表中加粗项是指博主挑选的较重要或者较常见的Service。

### 服务启动阶段

SystemServiceManager的startBootPhase()贯穿system_server进程的整个启动过程：

![system_server服务启动流程](http://gityuan.com/images/boot/systemServer/system_server_boot_process.jpg)

### 注册（添加）服务

**方式1. ServiceManager.addService():**

- 功能：向ServiceManager注册该服务.
- 特点：服务往往直接或间接继承于Binder服务；
- 举例：input, window, package；

**方式2. SystemServiceManager.startService:**

- 功能：
  - 创建服务对象；
  - 执行该服务的onStart()方法；该方法会执行上面的SM.addService()；
  - 根据启动到不同的阶段会回调onBootPhase()方法；
  - 另外，还有多用户模式下用户状态的改变也会有回调方法；例如onStartUser();
- 特点：服务往往自身或内部类继承于SystemService；
- 举例：power, activity；

两种方式真正注册服务的过程都会调用到ServiceManager.addService()方法. 对于方式2多了一个服务对象创建以及 根据不同启动阶段采用不同的动作的过程。可以理解为方式2比方式1的功能更丰富。