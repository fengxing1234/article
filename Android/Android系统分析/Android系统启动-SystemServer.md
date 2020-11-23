---
title: Android系统启动-SystemServer
date: 2020-05-29 11:05:49
tags: 
	- Android系统启动-SystemServer
categories: 
	- Android
	- Android系统分析
---



```
/frameworks/base/core/java/com/android/internal/os/
  - ZygoteInit.java
  - RuntimeInit.java
  - Zygote.java

/frameworks/base/core/services/java/com/android/server/
  - SystemServer.java

/frameworks/base/core/jni/
  - com_android_internal_os_Zygote.cpp
  - AndroidRuntime.cpp

/frameworks/base/cmds/app_process/App_main.cpp
```

# 启动流程

SystemServer的在Android体系中所处的地位，SystemServer由Zygote fork生成的，进程名为`system_server`，该进程承载着framework的核心服务。 [Android系统启动-zygote篇](http://gityuan.com/2016/02/13/android-zygote/)中讲到Zygote启动过程中会调用startSystemServer()，可知`startSystemServer()`函数是system_server启动流程的起点， 启动流程图如下：![system_server_boot_process](http://gityuan.com/images/boot/systemServer/system_server.jpg)

上图前4步骤（即颜色为紫色的流程）运行在是`Zygote`进程，从第5步（即颜色为蓝色的流程）ZygoteInit.handleSystemServerProcess开始是运行在新创建的`system_server`，这是fork机制实现的（fork会返回2次）。下面从startSystemServer()开始讲解详细启动流程。

## `ZygoteInit`

在Android9.0版本是在main方法中直接调用,开启system_server后直接运行。

### `main()`

```
public static void main(String argv[]) {
		...
	if (startSystemServer) {
                Runnable r = forkSystemServer(abiList, socketName, zygoteServer);

                // {@code r == null} in the parent (zygote) process, and {@code r != null} in the
                // child (system_server) process.
                if (r != null) {
                    r.run();
                    return;
                }
            }
   ...
}
```

### `forkSystemServer()`

```
private static Runnable forkSystemServer(String abiList, String socketName,
            ZygoteServer zygoteServer) {
        /* Hardcoded command line to start the system server */
            //参数准备
        String args[] = {
            "--setuid=1000",
            "--setgid=1000",
            "--setgroups=1001,1002,1003,1004,1005,1006,1007,1008,1009,1010,1018,1021,1023,1024,1032,1065,3001,3002,3003,3006,3007,3009,3010",
            "--capabilities=" + capabilities + "," + capabilities,
            "--nice-name=system_server",
            "--runtime-args",
            "--target-sdk-version=" + VMRuntime.SDK_VERSION_CUR_DEVELOPMENT,
            "com.android.server.SystemServer",
        };
        ZygoteConnection.Arguments parsedArgs = null;

        int pid;

        try {
 		        //用于解析参数，生成目标格式
            parsedArgs = new ZygoteConnection.Arguments(args);
            ZygoteConnection.applyDebuggerSystemProperty(parsedArgs);
            ZygoteConnection.applyInvokeWithSystemProperty(parsedArgs);

            boolean profileSystemServer = SystemProperties.getBoolean(
                    "dalvik.vm.profilesystemserver", false);
            if (profileSystemServer) {
                parsedArgs.runtimeFlags |= Zygote.PROFILE_SYSTEM_SERVER;
            }

            /* Request to fork the system server process */
            //// fork子进程，该进程是system_server进程
            pid = Zygote.forkSystemServer(
                    parsedArgs.uid, parsedArgs.gid,
                    parsedArgs.gids,
                    parsedArgs.runtimeFlags,
                    null,
                    parsedArgs.permittedCapabilities,
                    parsedArgs.effectiveCapabilities);
        } catch (IllegalArgumentException ex) {
            throw new RuntimeException(ex);
        }

        /* For child process */
        // 完成system_server进程剩余的工作
        if (pid == 0) {				    
            if (hasSecondZygote(abiList)) {
                waitForSecondaryZygote(socketName);
            }
						//关闭父进程zygote复制而来的Socket
            zygoteServer.closeServerSocket();
            // 完成system_server进程剩余的工作
            return handleSystemServerProcess(parsedArgs);
        }
        return null;
    }
```

调用`Zygote.forkSystemServer()`来创建`system_server`进程，返回进程pid，最后执行`handleSystemServerProcess`方法做后续处理。

# 如何创建`system_server`进程

## `Zygote`

### `forkSystemServer()`

```
public static int forkSystemServer(int uid, int gid, int[] gids, int runtimeFlags,
            int[][] rlimits, long permittedCapabilities, long effectiveCapabilities) {
        VM_HOOKS.preFork();
        // Resets nice priority for zygote process.
        resetNicePriority();
        int pid = nativeForkSystemServer(
                uid, gid, gids, runtimeFlags, rlimits, permittedCapabilities, effectiveCapabilities);
        // Enable tracing as soon as we enter the system_server.
        if (pid == 0) {
            Trace.setTracingEnabled(true, runtimeFlags);
        }
        VM_HOOKS.postForkCommon();
        return pid;
    }
```

nativeForkSystemServer()方法在AndroidRuntime.cpp中注册的，调用com_android_internal_os_Zygote.cpp中的register_com_android_internal_os_Zygote()方法建立native方法的映射关系，所以接下来进入如下方法。

### `nativeForkSystemServer()`

```
native private static int nativeForkSystemServer(int uid, int gid, int[] gids, int runtimeFlags,
            int[][] rlimits, long permittedCapabilities, long effectiveCapabilities);
```

## `com_android_internal_os_Zygote`

### `com_android_internal_os_Zygote_nativeForkSystemServer()`

```
static jint com_android_internal_os_Zygote_nativeForkSystemServer(
        JNIEnv* env, jclass, uid_t uid, gid_t gid, jintArray gids,
        jint debug_flags, jobjectArray rlimits, jlong permittedCapabilities,
        jlong effectiveCapabilities) {
  //fork子进程，见【见小节4】
  pid_t pid = ForkAndSpecializeCommon(env, uid, gid, gids,
                                      debug_flags, rlimits,
                                      permittedCapabilities, effectiveCapabilities,
                                      MOUNT_EXTERNAL_DEFAULT, NULL, NULL, true, NULL,
                                      NULL, NULL);
  if (pid > 0) {
      // zygote进程，检测system_server进程是否创建
      gSystemServerPid = pid;
      int status;
      if (waitpid(pid, &status, WNOHANG) == pid) {
          //当system_server进程死亡后，重启zygote进程
          RuntimeAbort(env);
      }
  }
  return pid;
}
```

当system_server进程创建失败时，将会重启zygote进程。这里需要注意，对于Android 5.0以上系统，有两个zygote进程，分别是zygote、zygote64两个进程，system_server的父进程，一般来说64位系统其父进程是zygote64进程

- 当kill system_server进程后，只重启zygote64和system_server，不重启zygote;
- 当kill zygote64进程后，只重启zygote64和system_server，也不重启zygote；
- 当kill zygote进程，则重启zygote、zygote64以及system_server。

### `ForkAndSpecializeCommon()`

```
static pid_t ForkAndSpecializeCommon(JNIEnv* env, uid_t uid, gid_t gid, jintArray javaGids, jint debug_flags, jobjectArray javaRlimits, jlong permittedCapabilities, jlong effectiveCapabilities, jint mount_external, jstring java_se_info, jstring java_se_name, bool is_system_server, jintArray fdsToClose, jstring instructionSet, jstring dataDir) {
  SetSigChldHandler(); //设置子进程的signal信号处理函数
  pid_t pid = fork(); //fork子进程
  if (pid == 0) {
    //进入子进程
    DetachDescriptors(env, fdsToClose); //关闭并清除文件描述符

    if (!is_system_server) {
        //对于非system_server子进程，则创建进程组
        int rc = createProcessGroup(uid, getpid());
    }
    SetGids(env, javaGids); //设置设置group
    SetRLimits(env, javaRlimits); //设置资源limit

    int rc = setresgid(gid, gid, gid);
    rc = setresuid(uid, uid, uid);

    SetCapabilities(env, permittedCapabilities, effectiveCapabilities);
    SetSchedulerPolicy(env); //设置调度策略

     //selinux上下文
    rc = selinux_android_setcontext(uid, is_system_server, se_info_c_str, se_name_c_str);

    if (se_info_c_str == NULL && is_system_server) {
      se_name_c_str = "system_server";
    }
    if (se_info_c_str != NULL) {
      SetThreadName(se_name_c_str); //设置线程名为system_server，方便调试
    }
    UnsetSigChldHandler(); //设置子进程的signal信号处理函数为默认函数
    //等价于调用zygote.callPostForkChildHooks()
    env->CallStaticVoidMethod(gZygoteClass, gCallPostForkChildHooks, debug_flags,
                              is_system_server ? NULL : instructionSet);
    ...

  } else if (pid > 0) {
    //进入父进程，即zygote进程
  }
  return pid;
}
```

fork()创建新进程，采用copy on write方式，这是linux创建进程的标准方法，会有两次return,对于pid==0为子进程的返回，对于pid>0为父进程的返回。 到此system_server进程已完成了创建的所有工作，接下来开始了system_server进程的真正工作。在前面startSystemServer()方法中，zygote进程执行完forkSystemServer()后，新创建出来的system_server进程便进入handleSystemServerProcess()方法。关于fork()，可查看另一个文章[理解Android进程创建流程](http://gityuan.com/2016/03/26/app-process-create/#nativeforkandspecialize)。

# system_server进程已创建看看如何做后续处理

## `ZygoteInit`

### `handleSystemServerProcess()`

```
private static Runnable handleSystemServerProcess(ZygoteConnection.Arguments parsedArgs) {
        // set umask to 0077 so new files and directories will default to owner-only permissions.
        Os.umask(S_IRWXG | S_IRWXO);
				//设置当前进程名为"system_server"
        if (parsedArgs.niceName != null) {
            Process.setArgV0(parsedArgs.niceName);
        }

        final String systemServerClasspath = Os.getenv("SYSTEMSERVERCLASSPATH");
        if (systemServerClasspath != null) {
        //执行dex优化操作
            performSystemServerDexOpt(systemServerClasspath);
            // Capturing profiles is only supported for debug or eng builds since selinux normally
            // prevents it.
            boolean profileSystemServer = SystemProperties.getBoolean(
                    "dalvik.vm.profilesystemserver", false);
            if (profileSystemServer && (Build.IS_USERDEBUG || Build.IS_ENG)) {
                try {
                    prepareSystemServerProfile(systemServerClasspath);
                } catch (Exception e) {
                    Log.wtf(TAG, "Failed to set up system server profile", e);
                }
            }
        }

        ...
						//启动应用进程
            WrapperInit.execApplication(parsedArgs.invokeWith,
                    parsedArgs.niceName, parsedArgs.targetSdkVersion,
                    VMRuntime.getCurrentInstructionSet(), null, args);

            throw new IllegalStateException("Unexpected return from WrapperInit.execApplication");
        } else {
        // 创建类加载器，并赋予当前线程
            ClassLoader cl = null;
            if (systemServerClasspath != null) {
                cl = createPathClassLoader(systemServerClasspath, parsedArgs.targetSdkVersion);

                Thread.currentThread().setContextClassLoader(cl);
            }
					  //system_server故进入此分支
            return ZygoteInit.zygoteInit(parsedArgs.targetSdkVersion, parsedArgs.remainingArgs, cl);
        }

```

### `zygoteInit()`

```
public static final Runnable zygoteInit(int targetSdkVersion, String[] argv, ClassLoader classLoader) {
        if (RuntimeInit.DEBUG) {
            Slog.d(RuntimeInit.TAG, "RuntimeInit: Starting application from zygote");
        }

        Trace.traceBegin(Trace.TRACE_TAG_ACTIVITY_MANAGER, "ZygoteInit");
        //重定向log输出
        RuntimeInit.redirectLogStreams();
				// 通用的一些初始化
        RuntimeInit.commonInit();
        // zygote初始化
        ZygoteInit.nativeZygoteInit();
        // 应用初始化
        return RuntimeInit.applicationInit(targetSdkVersion, argv, classLoader);
    }
```

### `nativeZygoteInit()`

```
private static final native void nativeZygoteInit();
```

## `AndroidRuntime.cpp`

### `com_android_internal_os_RuntimeInit_nativeZygoteInit()`

```
static void com_android_internal_os_RuntimeInit_nativeZygoteInit(JNIEnv* env, jobject clazz) {
    //此处的gCurRuntime为AppRuntime，是在AndroidRuntime.cpp中定义的
    gCurRuntime->onZygoteInit();
}
```

ProcessState::self()是单例模式，主要工作是调用open()打开/dev/binder驱动设备，再利用mmap()映射内核的地址空间，将Binder驱动的fd赋值ProcessState对象中的变量mDriverFD，用于交互操作。startThreadPool()是创建一个新的binder线程，不断进行talkWithDriver()，在binder系列文章中的[注册服务(addService)](http://gityuan.com/2015/11/14/binder-add-service/)详细这两个方法的执行原理。

## `RuntimeInit`

### `applicationInit()`

```
protected static Runnable applicationInit(int targetSdkVersion, String[] argv,
            ClassLoader classLoader) {
        //true代表应用程序退出时不调用AppRuntime.onExit()，否则会在退出前调用
        nativeSetExitWithoutCleanup(true);

       //设置虚拟机的内存利用率参数值为0.75
        VMRuntime.getRuntime().setTargetHeapUtilization(0.75f);
        VMRuntime.getRuntime().setTargetSdkVersion(targetSdkVersion);
				//解析参数
        final Arguments args = new Arguments(argv);

        // The end of of the RuntimeInit event (see #zygoteInit).
        Trace.traceEnd(Trace.TRACE_TAG_ACTIVITY_MANAGER);

        //调用startClass的static方法 main() 
        return findStaticMain(args.startClass, args.startArgs, classLoader);
    }
```

在startSystemServer()方法中通过硬编码初始化参数，可知此处args.startClass为”com.android.server.SystemServer”。

### `findStaticMain()`

```
 protected static Runnable findStaticMain(String className, String[] argv,
            ClassLoader classLoader) {
        Class<?> cl;

        try {
            cl = Class.forName(className, true, classLoader);
        } catch (ClassNotFoundException ex) {
            throw new RuntimeException(
                    "Missing class when invoking static main " + className,
                    ex);
        }

        Method m;
        try {
            m = cl.getMethod("main", new Class[] { String[].class });
        } catch (NoSuchMethodException ex) {
            throw new RuntimeException(
                    "Missing static main on " + className, ex);
        } catch (SecurityException ex) {
            throw new RuntimeException(
                    "Problem getting static main on " + className, ex);
        }
        
        int modifiers = m.getModifiers();
        if (! (Modifier.isStatic(modifiers) && Modifier.isPublic(modifiers))) {
            throw new RuntimeException(
                    "Main method is not public and static on " + className);
        }
        return new MethodAndArgsCaller(m, argv);
    }
```

## `MethodAndArgsCaller`

```
static class MethodAndArgsCaller implements Runnable {
        /** method to call */
        private final Method mMethod;

        /** argument array */
        private final String[] mArgs;

        public MethodAndArgsCaller(Method method, String[] args) {
            mMethod = method;
            mArgs = args;
        }

        public void run() {
            try {
                mMethod.invoke(null, new Object[] { mArgs });
            } catch (IllegalAccessException ex) {
                throw new RuntimeException(ex);
            } catch (InvocationTargetException ex) {
                Throwable cause = ex.getCause();
                if (cause instanceof RuntimeException) {
                    throw (RuntimeException) cause;
                } else if (cause instanceof Error) {
                    throw (Error) cause;
                }
                throw new RuntimeException(ex);
            }
        }
    }
}
```

实现Runnable接口，通过反射执行SystemServer中manin方法。

# 进入`SystemServer`中

至此已经进入SystemServer中的main方法中，

## `SystemServer`

### `main()`

```
public static void main(String[] args) {
        new SystemServer().run();
    }
```

先初始化，在调用run方法

### `run()`

```
private void run() {
		//当系统时间比1970年更早，就设置当前系统时间为1970年
    if (System.currentTimeMillis() < EARLIEST_SUPPORTED_TIME) {
        SystemClock.setCurrentTimeMillis(EARLIEST_SUPPORTED_TIME);
    }
    
     //变更虚拟机的库文件，对于Android 6.0默认采用的是libart.so
    SystemProperties.set("persist.sys.dalvik.vm.lib.2", VMRuntime.getRuntime().vmLibrary());

    if (SamplingProfilerIntegration.isEnabled()) {
        ...
    }
    
    //清除vm内存增长上限，由于启动过程需要较多的虚拟机内存空间
    VMRuntime.getRuntime().clearGrowthLimit();

    //设置内存的可能有效使用率为0.8
    VMRuntime.getRuntime().setTargetHeapUtilization(0.8f);
    // 针对部分设备依赖于运行时就产生指纹信息，因此需要在开机完成前已经定义
    Build.ensureFingerprintProperty();

    //访问环境变量前，需要明确地指定用户
    Environment.setUserRequired(true);

    //确保当前系统进程的binder调用，总是运行在前台优先级(foreground priority)
    BinderInternal.disableBackgroundScheduling(true);
    android.os.Process.setThreadPriority(android.os.Process.THREAD_PRIORITY_FOREGROUND);
    android.os.Process.setCanSelfBackground(false);

    // 主线程looper就在当前线程运行
    Looper.prepareMainLooper();

    //加载android_servers.so库，该库包含的源码在frameworks/base/services/目录下
    System.loadLibrary("android_servers");

    //检测上次关机过程是否失败，该方法可能不会返回[见小节1.2.1]
    performPendingShutdown();

    //初始化系统上下文 【见小节1.3】
    createSystemContext();

    //创建系统服务管理
    mSystemServiceManager = new SystemServiceManager(mSystemContext);
    //将mSystemServiceManager添加到本地服务的成员sLocalServiceObjects
    LocalServices.addService(SystemServiceManager.class, mSystemServiceManager);


    //启动各种系统服务
    try {
        startBootstrapServices(); // 启动引导服务【见小节1.4】
        startCoreServices();      // 启动核心服务【见小节1.5】
        startOtherServices();     // 启动其他服务【见小节1.6】
    } catch (Throwable ex) {
        Slog.e("System", "************ Failure starting system services", ex);
        throw ex;
    }

    //用于debug版本，将log事件不断循环地输出到dropbox（用于分析）
    if (StrictMode.conditionallyEnableDebugLogging()) {
        Slog.i(TAG, "Enabled StrictMode for system server main thread.");
    }
    //一直循环执行
    Looper.loop();
    throw new RuntimeException("Main thread loop unexpectedly exited");
}
```

LocalServices通过用静态Map变量sLocalServiceObjects，来保存以服务类名为key，以具体服务对象为value的Map结构。

### `performPendingShutdown()`

```
private void performPendingShutdown() {
    final String shutdownAction = SystemProperties.get(
            ShutdownThread.SHUTDOWN_ACTION_PROPERTY, "");
    if (shutdownAction != null && shutdownAction.length() > 0) {
        boolean reboot = (shutdownAction.charAt(0) == '1');

        final String reason;
        if (shutdownAction.length() > 1) {
            reason = shutdownAction.substring(1, shutdownAction.length());
        } else {
            reason = null;
        }
        // 当"sys.shutdown.requested"值不为空,则会重启或者关机
        ShutdownThread.rebootOrShutdown(null, reboot, reason);
    }
}
```

### `createSystemContext()`

```
private void createSystemContext() {
    //创建system_server进程的上下文信息
    ActivityThread activityThread = ActivityThread.systemMain();
    mSystemContext = activityThread.getSystemContext();
    //设置主题
    mSystemContext.setTheme(android.R.style.Theme_DeviceDefault_Light_DarkActionBar);
}
```

[理解Application创建过程](http://gityuan.com/2017/04/02/android-application/)已介绍过createSystemContext()过程， 该过程会创建对象有ActivityThread，Instrumentation, ContextImpl，LoadedApk，Application。

### `startBootstrapServices()`

```
private void startBootstrapServices() {
    //阻塞等待与installd建立socket通道 Installer的onStart()
    Installer installer = mSystemServiceManager.startService(Installer.class);

    //启动服务ActivityManagerService
    mActivityManagerService = mSystemServiceManager.startService(
            ActivityManagerService.Lifecycle.class).getService();
    mActivityManagerService.setSystemServiceManager(mSystemServiceManager);
    mActivityManagerService.setInstaller(installer);

    //启动服务PowerManagerService
    mPowerManagerService = mSystemServiceManager.startService(PowerManagerService.class);

    //初始化power management
    mActivityManagerService.initPowerManagement();

    //启动服务LightsService
    mSystemServiceManager.startService(LightsService.class);

    //启动服务DisplayManagerService
    mDisplayManagerService = mSystemServiceManager.startService(DisplayManagerService.class);

    //Phase100: 在初始化package manager之前，需要默认的显示.
    mSystemServiceManager.startBootPhase(SystemService.PHASE_WAIT_FOR_DEFAULT_DISPLAY);

    //当设备正在加密时，仅运行核心
    String cryptState = SystemProperties.get("vold.decrypt");
    if (ENCRYPTING_STATE.equals(cryptState)) {
        mOnlyCore = true;
    } else if (ENCRYPTED_STATE.equals(cryptState)) {
        mOnlyCore = true;
    }

    //启动服务PackageManagerService
    mPackageManagerService = PackageManagerService.main(mSystemContext, installer,
            mFactoryTestMode != FactoryTest.FACTORY_TEST_OFF, mOnlyCore);
    mFirstBoot = mPackageManagerService.isFirstBoot();
    mPackageManager = mSystemContext.getPackageManager();

    //启动服务UserManagerService，新建目录/data/user/
    ServiceManager.addService(Context.USER_SERVICE, UserManagerService.getInstance());

    AttributeCache.init(mSystemContext);

    //设置AMS
    mActivityManagerService.setSystemProcess();

    //启动传感器服务
    startSensorService();
}
```

该方法所创建的服务：ActivityManagerService, PowerManagerService, LightsService, DisplayManagerService， PackageManagerService， UserManagerService， sensor服务.

### `startCoreServices()`

```
private void startCoreServices() {
		    //启动服务BatteryService，用于统计电池电量，需要LightService.
        mSystemServiceManager.startService(BatteryService.class);
		    //启动服务UsageStatsService，用于统计应用使用情况
        mSystemServiceManager.startService(UsageStatsService.class);
        mActivityManagerService.setUsageStatsManager(
                LocalServices.getService(UsageStatsManagerInternal.class));

        if (mPackageManager.hasSystemFeature(PackageManager.FEATURE_WEBVIEW)) {
            traceBeginAndSlog("StartWebViewUpdateService");
            //启动服务WebViewUpdateService
            mWebViewUpdateService = mSystemServiceManager.startService(WebViewUpdateService.class);
        }
        BinderCallsStatsService.start();
    }
```

启动服务BatteryService，UsageStatsService，WebViewUpdateService。

### `startOtherServices()`

该方法比较长，有近千行代码，逻辑很简单，主要是启动一系列的服务，这里就不具体列举源码了，在第四节直接对其中的服务进行一个简单分类。

```
 private void startOtherServices() {
        ...
        SystemConfig.getInstance();
        mContentResolver = context.getContentResolver(); // resolver
        ...
        mActivityManagerService.installSystemProviders(); //provider
        mSystemServiceManager.startService(AlarmManagerService.class); // alarm
        // watchdog
        watchdog.init(context, mActivityManagerService); 
        inputManager = new InputManagerService(context); // input
        wm = WindowManagerService.main(...); // window
        inputManager.start();  //启动input
        mDisplayManagerService.windowManagerAndInputReady();
        ...
        mSystemServiceManager.startService(MOUNT_SERVICE_CLASS); // mount
        mPackageManagerService.performBootDexOpt();  // dexopt操作
        ActivityManagerNative.getDefault().showBootMessage(...); //显示启动界面
        ...
        statusBar = new StatusBarManagerService(context, wm); //statusBar
        //dropbox
        ServiceManager.addService(Context.DROPBOX_SERVICE,
                    new DropBoxManagerService(context, new File("/data/system/dropbox")));
         mSystemServiceManager.startService(JobSchedulerService.class); //JobScheduler
         lockSettings.systemReady(); //lockSettings

        //phase480 和phase500
        mSystemServiceManager.startBootPhase(SystemService.PHASE_LOCK_SETTINGS_READY);
        mSystemServiceManager.startBootPhase(SystemService.PHASE_SYSTEM_SERVICES_READY);
        ...
        // 准备好window, power, package, display服务
        wm.systemReady();
        mPowerManagerService.systemReady(...);
        mPackageManagerService.systemReady();
        mDisplayManagerService.systemReady(...);
        
        //重头戏[见小节2.1]
        mActivityManagerService.systemReady(new Runnable() {
            public void run() {
              ...
            }
        });
    }
```

SystemServer启动各种服务中最后的一个环节便是AMS.systemReady()，详见[ActivityManagerService启动过程](http://gityuan.com/2016/02/21/activity-manager-service/).

到此, System_server主线程的启动工作总算完成, 进入Looper.loop()状态,等待其他线程通过handler发送消息到主线再处理.

# 至此System_Server启动完成

# 服务启动阶段

SystemServiceManager的startBootPhase()贯穿system_server进程的整个启动过程：

![system_server服务启动流程](http://gityuan.com/images/boot/systemServer/system_server_boot_process.jpg)

**各个启动阶段所在源码的大致位置：**

```
public final class SystemServer {

    private void startBootstrapServices() {
      ...
      //phase100
      mSystemServiceManager.startBootPhase(SystemService.PHASE_WAIT_FOR_DEFAULT_DISPLAY);
      ...
    }

    private void startCoreServices() {
      ...
    }

    private void startOtherServices() {
      ...
      //phase480 && 500
      mSystemServiceManager.startBootPhase(SystemService.PHASE_LOCK_SETTINGS_READY);
      mSystemServiceManager.startBootPhase(SystemService.PHASE_SYSTEM_SERVICES_READY);
      
      ...
      mActivityManagerService.systemReady(new Runnable() {
         public void run() {
             //phase550
             mSystemServiceManager.startBootPhase(
                     SystemService.PHASE_ACTIVITY_MANAGER_READY);
             ...
             //phase600
             mSystemServiceManager.startBootPhase(
                     SystemService.PHASE_THIRD_PARTY_APPS_CAN_START);
          }
      }
    }
}
```

接下来再说说简单每个阶段的大概完成的工作：

## Phase0

创建四大引导服务:

- ActivityManagerService
- PowerManagerService
- LightsService
- DisplayManagerService

## Phase100

进入阶段`PHASE_WAIT_FOR_DEFAULT_DISPLAY`=100回调服务

onBootPhase(100)

- DisplayManagerService

然后创建大量服务下面列举部分:

- PackageManagerService
- WindowManagerService
- InputManagerService
- NetworkManagerService
- DropBoxManagerService
- FingerprintService
- LauncherAppsService
- …

## Phase480

进入阶段`PHASE_LOCK_SETTINGS_READY`=480回调服务

onBootPhase(480)

- DevicePolicyManagerService

阶段480后马上就进入阶段500.

## Phase500

`PHASE_SYSTEM_SERVICES_READY`=500，进入该阶段服务能安全地调用核心系统服务.

onBootPhase(500)

- AlarmManagerService
- JobSchedulerService
- NotificationManagerService
- BackupManagerService
- UsageStatsService
- DeviceIdleController
- TrustManagerService
- UiModeManagerService
- BluetoothService
- BluetoothManagerService
- EthernetService
- WifiP2pService
- WifiScanningService
- WifiService
- RttService

各大服务执行systemReady():

- WindowManagerService.systemReady():
- PowerManagerService.systemReady():
- PackageManagerService.systemReady():
- DisplayManagerService.systemReady():

接下来就绪AMS.systemReady方法.

## Phase550

`PHASE_ACTIVITY_MANAGER_READY`=550， AMS.mSystemReady=true, 已准备就绪,进入该阶段服务能广播Intent;但是system_server主线程并没有就绪.

onBootPhase(550)

- MountService
- TelecomLoaderService
- UsbService
- WebViewUpdateService
- DockObserver
- BatteryService



接下来执行: (AMS启动native crash监控, 加载WebView，启动SystemUi等),如下

- mActivityManagerService.startObservingNativeCrashes();
- WebViewFactory.prepareWebViewInSystemServer();
- startSystemUi(context);
- networkScoreF.systemReady();
- networkManagementF.systemReady();
- networkStatsF.systemReady();
- networkPolicyF.systemReady();
- connectivityF.systemReady();
- audioServiceF.systemReady();
- Watchdog.getInstance().start();



## Phase600

`PHASE_THIRD_PARTY_APPS_CAN_START`=600

onBootPhase(600)

- JobSchedulerService
- NotificationManagerService
- BackupManagerService
- AppWidgetService
- GestureLauncherService
- DreamManagerService
- TrustManagerService
- VoiceInteractionManagerService

接下来,各种服务的systemRunning过程:

WallpaperManagerService、InputMethodManagerService、LocationManagerService、CountryDetectorService、NetworkTimeUpdateService、CommonTimeManagementService、TextServicesManagerService、AssetAtlasService、InputManagerService、TelephonyRegistry、MediaRouterService、MmsServiceBroker这些服务依次执行其`systemRunning()`方法。

## Phase1000

在经过一系列流程，再调用`AMS.finishBooting()`时，则进入阶段`Phase1000`。

到此，系统服务启动阶段完成就绪，system_server进程启动完成则进入`Looper.loop()`状态，随时待命，等待消息队列MessageQueue中的消息到来，则马上进入执行状态。

# 服务类别

system_server进程，从源码角度划分为引导服务、核心服务、其他服务3类。 以下这些系统服务的注册过程, 见[Android系统服务的注册方式](http://gityuan.com/2016/10/01/system_service_common/)

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

Service类别众多，其中表中加粗项是指博主挑选的较重要或者较常见的Service，并且在本博客中已经展开或者计划展开讲解的Service，当然如果有精力会讲解更多service，后续再更新。

# 客户端使用服务

## 一个简单的SystemService

我们从一个简单的系统服务Vibrator服务来看一下一个系统服务是怎样建立的。
Vibrator服务提供的控制手机振动的接口，应用可以调用Vibrator的接口来让手机产生振动，达到提醒用户的目的。
从Android的官方文档中可以看到Vibrator只是一个抽象类，只有4个抽象接口：

- abstract void cancel() 取消振动
- abstract boolean hasVibrator() 是否有振动功能
- abstract void vibrate(long[] pattern, int repeat) 按节奏重复振动
- abstract void vibrate(long milliseconds) 持续振动

应用中使用振动服务的方法也很简单，如让手机持续振动500毫秒：

```
Vibrator mVibrator = (Vibrator) getSystemService(Context.VIBRATOR_SERVICE);
mVibrator.vibrate(500);
```

`Vibrator`使用起来很简单，我们再来看一下实现起来是不是也简单。从文档中可以看到Vibrator只是定义在android.os 包里的一个抽象类，在源码里的位置即`frameworks/base/core/java/android/os/Vibrator.java`，那么应用中实际使用的是哪个实例呢？应用中使用的Vibrator实例是通过Context的一个方法`getSystemService(Context.VIBRATOR_SERVICE)`获得的，而Context的实现一般都在`ContextImpl`中，那我们就看一下ContextImpl是怎么实现getSystemService的：
`frameworks/base/core/java/android/app/ContextImpl.java`

```
@Override
public Object getSystemService(String name) {
    return SystemServiceRegistry.getSystemService(this, name);
}
```

`frameworks/base/core/java/android/app/SystemServiceRegistry.java`

(`SystemServiceRegistry`是 Android 6.0之后才有的，Android 6.0 之前的代码没有该类，下面的代码是直接写在`ContextImpl`里的)

```
public static Object getSystemService(ContextImpl ctx, String name) {
    ServiceFetcher<?> fetcher = SYSTEM_SERVICE_FETCHERS.get(name);
    return fetcher != null ? fetcher.getService(ctx) : null;
}
```

`SYSTEM_SERVICE_MA`P是一个HashMap，通过我们服务的名字name字符串，从这个HashMap里取出一个ServiceFetcher，再return这个ServiceFetcher的getService()。ServiceFetcher是什么？它的getService()又是什么？既然他是从SYSTEM_SERVICE_MAP这个HashMap里get出来的，那就找一找这个HashMap都put了什么。
通过搜索SystemServiceRegistry可以找到如下代码：

```
private static <T> void registerService(String serviceName, Class<T> serviceClass,
        ServiceFetcher<T> serviceFetcher) {
    SYSTEM_SERVICE_NAMES.put(serviceClass, serviceName);
    SYSTEM_SERVICE_FETCHERS.put(serviceName, serviceFetcher);
}
```

这里往`SYSTEM_SERVICE_MAP`里`put`了一对`String`与`ServiceFetcher`组成的key/value对，`registerService()`又是从哪里调用的？继续搜索可以发现很多类似下面的代码：

```
static {
    registerService(Context.ACCESSIBILITY_SERVICE, AccessibilityManager.class,
            new CachedServiceFetcher<AccessibilityManager>() {
        @Override
        public AccessibilityManager createService(ContextImpl ctx) {
            return AccessibilityManager.getInstance(ctx);
        }});

    ...
    registerService(Context.VIBRATOR_SERVICE, Vibrator.class,
            new CachedServiceFetcher<Vibrator>() {
        @Override
        public Vibrator createService(ContextImpl ctx) {
            return new SystemVibrator(ctx);
        }});
    ...
}
```

`SystemServiceRegistry`的static代码块里通过`registerService`注册了很多的系统服务，其中就包括我们正在调查的`VIBRATOR_SERVICE`，通过结合上面的分析代码可以可以知道`getSystemService(Context.VIBRATOR_SERVICE)`得到的是一个`SystemVibrator`的实例，通过查看`SystemVibrator`的代码也可以发现`SystemVibrator`确实是继承自Vibrator：

```
public class SystemVibrator extends Vibrator {
    ...
}
```

我们再从`SystemVibrator`看一下系统的振动控制是怎么实现的。以`hasVibrator()`为例，这个是查询当前系统是否能够振动，在`SystemVibrator`中它的实现如下：

```
public boolean hasVibrator() {
    ...
    try {
        return mService.hasVibrator();
    } catch (RemoteException e) {
    }
    ...
}
```

这里直接调用了一个`mService.hasVibrator()`。`mService`是什么？哪来的？搜索一下可以发现：

```
private final IVibratorService mService;
public SystemVibrator() {
    ...
    mService = IVibratorService.Stub.asInterface(
            ServiceManager.getService("vibrator"));
}
```

`mService` 是一个`IVibratorService`，我们先不去管`IVibratorService.Stub.asInterface`是怎么回事，先看一下`IVibratorService`是什么。搜索一下代码发现这并不是一个java文件，而是一个aidl文件：`frameworks/base/core/java/android/os/IVibratorService.aidl`
AIDL (Android Interface Definition Language) 是Android中的接口定义文件，为系统提供了一种简单跨进程通信方法。
IVibratorService 中定义了几个接口，SystemVibrator中使用的也是这几个接口，包括我们刚才使用的hasVibrator()

```
interface IVibratorService
{
    boolean hasVibrator();
    void vibrate(...);
    void vibratePattern(...);
    void cancelVibrate(IBinder token);
}
```

这里又只是接口定义，接口实现在哪呢？通过在`frameworks/base`目录下进行grep搜索，或者在AndroidXRef搜索，可以发现IVibratorService接口的实现在`frameworks/base/services/java/com/android/server/VibratorService.java`

```
public class VibratorService extends IVibratorService.Stub
```

可以看到 VibratorService实现了IVibratorService定义的所有接口，并通过JNI调用到native层，进行更底层的实现。更底层的实现不是这篇文档讨论的内容，我们需要分析的是VibratorService怎么成为系统服务的。那么VibratorService是怎么注册为系统服务的呢？在SystemServer里面：

```
VibratorService vibrator = null;
...
//实例化VibratorService并添加到ServiceManager
traceBeginAndSlog("StartVibratorService");
vibrator = new VibratorService(context);
ServiceManager.addService("vibrator", vibrator);
Trace.traceEnd(Trace.TRACE_TAG_SYSTEM_SERVER);
...
//通知服务系统启动完成
Trace.traceBegin(Trace.TRACE_TAG_SYSTEM_SERVER, "MakeVibratorServiceReady");
try {
    vibrator.systemReady();
} catch (Throwable e) {
    reportWtf("making Vibrator Service ready", e);
}
Trace.traceEnd(Trace.TRACE_TAG_SYSTEM_SERVER);
```

这样在`SystemVibrator`里就可以通过下面的代码连接到`VibratorService`，与底层的系统服务进行通信了：

```
IVibratorService.Stub.asInterface(ServiceManager.getService("vibrator"));
```

`mService`相当于`IVibratorService`在应用层的一个代理，所有的实现还是在`SystemServer`的`VibratorService`里。

看代码时可以发现`registerService`是在static代码块里静态调用的，所以`getSystemServcr`获得的各个Manager也都是单例的。

**从上面的分析，我们可以总结出Vibrator服务的整个实现流程：**

1. 定义一个抽象类`Vibrator`，定义了应用中可以访问的一些抽象方法(抽象)
   `frameworks/base/core/java/android/os/Vibrator.java`

2. 定义具体的类`SystemVibrator`继承`Vibrator`，实现抽象方法（客户端调用）

   `frameworks/base/core/java/android/os/SystemVibrator.java`

3. 定义一个AIDL接口文件`IVibratorService`，定义系统服务接口（服务端接口）
   frameworks/base/core/java/android/os/IVibratorService.aidl

4. 定义服务VibratorService，实现IVibratorService定义的接口(服务端实现)
   `frameworks/base/services/java/com/android/server/VibratorService.java`

   ```
   public class VibratorService extends IVibratorService.Stub
   ```

5. 将`VibratorServicey`添加到系统服务
   `frameworks/base/services/java/com/android/server/SystemServer.java`

   ```
   VibratorService vibrator = null;
   ...
   //实例化VibratorService并添加到ServiceManager
   Slog.i(TAG, "Vibrator Service");
   vibrator = new VibratorService(context);
   ServiceManager.addService("vibrator", vibrator);
   ...
   //通知服务系统启动完成
   try {
       vibrator.systemReady();
   } catch (Throwable e) {
       reportWtf("making Vibrator Service ready", e);
   }
   ```

6. 在`SystemVibrator`中通过`IVibratorService`的代理连接到`VibratorService`，这样`SystemVibrator`的接口实现里就可以调用`IVibratorService`的接口：

   `frameworks/base/core/java/android/os/SystemVibrator.java`

   ```
   private final IVibratorService mService;
   ...
   public SystemVibrator() {
       ...
       mService = IVibratorService.Stub.asInterface(
               ServiceManager.getService("vibrator"));
       ...
       public boolean hasVibrator() {
           ...
           try {
               return mService.hasVibrator();
           } catch (RemoteException e) {
           }
           ...
       }
   }
   ```

7. 在`Context`里定义一个代表`Vibrator`服务的字符串

   `frameworks/base/core/java/android/content/Context.java`

   ```
   public static final String VIBRATOR_SERVICE = "vibrator";
   ```

8. 在ContextImpl里添加SystemVibrator的实例化过程
   `frameworks/base/core/java/android/app/ContextImpl.java`

   ```
   registerService(VIBRATOR_SERVICE, new ServiceFetcher() {
   public Object createService(ContextImpl ctx) {
       return new SystemVibrator(ctx);
   }});  
   ```

9. 在应用中使用`Vibrator`的接口

   ```
   Vibrator mVibrator = (Vibrator) getSystemService(Context.VIBRATOR_SERVICE);
   mVibrator.vibrate(500);
   ```

10. 为保证编译正常，还需要将AIDL文件添加到编译配置里
    `frameworks/base/Android.mk`

    ```
    LOCAL_SRC_FILES += \
    ...
    core/java/android/os/IVibratorService.aidl \
    ```

    