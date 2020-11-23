---
title: Android系统启动-Zygote
date: 2020-05-29 09:38:15
tags: 
	- Android系统启动zygote
categories: 
	- Android
	- Android系统分析
---

```
/frameworks/base/cmds/app_process/App_main.cpp
/frameworks/base/core/jni/AndroidRuntime.cpp

/frameworks/base/core/java/com/android/internal/os/
  - ZygoteInit.java
  - Zygote.java
  - ZygoteConnection.java
  
/frameworks/base/core/java/android/net/LocalServerSocket.java
/system/core/libutils/Threads.cpp
```

[https://android.googlesource.com/platform/frameworks/base/+/refs/tags/android-vts-9.0_r13/cmds/app_process/app_main.cpp](https://android.googlesource.com/platform/frameworks/base/+/refs/tags/android-vts-9.0_r13/cmds/app_process/app_main.cpp)

# 概述

Zygote是由[init进程](http://gityuan.com/2016/02/05/android-init/)通过解析init.zygote.rc文件而创建的，zygote所对应的可执行程序app_process，所对应的源文件是App_main.cpp，进程名为zygote。

Zygote进程能够重启的地方:

- servicemanager进程被杀; (onresart)
- surfaceflinger进程被杀; (onresart)
- Zygote进程自己被杀; (oneshot=false)
- system_server进程被杀; (waitpid)

从App_main()开始，Zygote启动过程的函数调用类大致流程如下：

![zygote_process](http://gityuan.com/images/boot/zygote/zygote_process.jpg)

# Zygote启动过程

## `App_main.cpp`

### `main()`

```
int main(int argc, char* const argv[])
{
    //传到的参数argv为“-Xzygote /system/bin --zygote --start-system-server”
    AppRuntime runtime(argv[0], computeArgBlockSize(argc, argv));
    argc--; argv++; //忽略第一个参数

    int i;
    for (i = 0; i < argc; i++) {
        if (argv[i][0] != '-') {
            break;
        }
        if (argv[i][1] == '-' && argv[i][2] == 0) {
            ++i;
            break;
        }
        runtime.addOption(strdup(argv[i]));
    }
    //参数解析
    bool zygote = false;
    bool startSystemServer = false;
    bool application = false;
    String8 niceName;
    String8 className;
    ++i;
    while (i < argc) {
        const char* arg = argv[i++];
        if (strcmp(arg, "--zygote") == 0) {
            zygote = true;
            //对于64位系统nice_name为zygote64; 32位系统为zygote
            niceName = ZYGOTE_NICE_NAME;
        } else if (strcmp(arg, "--start-system-server") == 0) {
            startSystemServer = true;
        } else if (strcmp(arg, "--application") == 0) {
            application = true;
        } else if (strncmp(arg, "--nice-name=", 12) == 0) {
            niceName.setTo(arg + 12);
        } else if (strncmp(arg, "--", 2) != 0) {
            className.setTo(arg);
            break;
        } else {
            --i;
            break;
        }
    }
    Vector<String8> args;
    if (!className.isEmpty()) {
        // 运行application或tool程序
        args.add(application ? String8("application") : String8("tool"));
        runtime.setClassNameAndArgs(className, argc - i, argv + i);
    } else {
        //进入zygote模式，创建 /data/dalvik-cache路径
        maybeCreateDalvikCache();
        if (startSystemServer) {
            args.add(String8("start-system-server"));
        }
        char prop[PROP_VALUE_MAX];
        if (property_get(ABI_LIST_PROPERTY, prop, NULL) == 0) {
            return 11;
        }
        String8 abiFlag("--abi-list=");
        abiFlag.append(prop);
        args.add(abiFlag);

        for (; i < argc; ++i) {
            args.add(String8(argv[i]));
        }
    }

    //设置进程名
    if (!niceName.isEmpty()) {
        runtime.setArgv0(niceName.string());
        set_process_name(niceName.string());
    }
    if (zygote) {
        // 启动AppRuntime 【见小节2.2】
        runtime.start("com.android.internal.os.ZygoteInit", args, zygote);
    } else if (className) {
        runtime.start("com.android.internal.os.RuntimeInit", args, zygote);
    } else {
        //没有指定类名或zygote，参数错误
        return 10;
    }
}
```

- 启动AppRuntime

## `AndroidRuntime.cpp`

### `start()`

```
void AndroidRuntime::start(const char* className, const Vector<String8>& options, bool zygote)
{
    static const String8 startSystemServer("start-system-server");

    for (size_t i = 0; i < options.size(); ++i) {
        if (options[i] == startSystemServer) {
           const int LOG_BOOT_PROGRESS_START = 3000;
        }
    }
    const char* rootDir = getenv("ANDROID_ROOT");
    if (rootDir == NULL) {
        rootDir = "/system";
        if (!hasDir("/system")) {
            return;
        }
        setenv("ANDROID_ROOT", rootDir, 1);
    }
    JniInvocation jni_invocation;
    jni_invocation.Init(NULL);
    JNIEnv* env;
    // 虚拟机创建【见小节2.3】
    if (startVm(&mJavaVM, &env, zygote) != 0) {
        return;
    }
    onVmCreated(env);
    // JNI方法注册【见小节2.4】
    if (startReg(env) < 0) {
        return;
    }

    jclass stringClass;
    jobjectArray strArray;
    jstring classNameStr;

    //等价 strArray= new String[options.size() + 1];
    stringClass = env->FindClass("java/lang/String");
    strArray = env->NewObjectArray(options.size() + 1, stringClass, NULL);

    //等价 strArray[0] = "com.android.internal.os.ZygoteInit"
    classNameStr = env->NewStringUTF(className);
    env->SetObjectArrayElement(strArray, 0, classNameStr);

    //等价 strArray[1] = "start-system-server"；
    // strArray[2] = "--abi-list=xxx"；
    //其中xxx为系统响应的cpu架构类型，比如arm64-v8a.
    for (size_t i = 0; i < options.size(); ++i) {
        jstring optionsStr = env->NewStringUTF(options.itemAt(i).string());
        env->SetObjectArrayElement(strArray, i + 1, optionsStr);
    }

    //将"com.android.internal.os.ZygoteInit"转换为"com/android/internal/os/ZygoteInit"
    char* slashClassName = toSlashClassName(className);
    jclass startClass = env->FindClass(slashClassName);
    if (startClass == NULL) {
        ...
    } else {
        jmethodID startMeth = env->GetStaticMethodID(startClass, "main",
            "([Ljava/lang/String;)V");
        // 调用ZygoteInit.main()方法【见小节3.1】
        env->CallStaticVoidMethod(startClass, startMeth, strArray);
    }
    //释放相应对象的内存空间
    free(slashClassName);
    mJavaVM->DetachCurrentThread();
    mJavaVM->DestroyJavaVM();
}
```

### `startVm()`

**创建Java虚拟机**方法的主要篇幅是关于虚拟机参数的设置，下面只列举部分在调试优化过程中常用参数。

```
int AndroidRuntime::startVm(JavaVM** pJavaVM, JNIEnv** pEnv, bool zygote)
{
    // JNI检测功能，用于native层调用jni函数时进行常规检测，比较弱字符串格式是否符合要求，资源是否正确释放。该功能一般用于早期系统调试或手机Eng版，对于User版往往不会开启，引用该功能比较消耗系统CPU资源，降低系统性能。
    bool checkJni = false;
    property_get("dalvik.vm.checkjni", propBuf, "");
    if (strcmp(propBuf, "true") == 0) {
        checkJni = true;
    } else if (strcmp(propBuf, "false") != 0) {
        property_get("ro.kernel.android.checkjni", propBuf, "");
        if (propBuf[0] == '1') {
            checkJni = true;
        }
    }
    if (checkJni) {
        addOption("-Xcheck:jni");
    }

    //虚拟机产生的trace文件，主要用于分析系统问题，路径默认为/data/anr/traces.txt
    parseRuntimeOption("dalvik.vm.stack-trace-file", stackTraceFileBuf, "-Xstacktracefile:");

    //对于不同的软硬件环境，这些参数往往需要调整、优化，从而使系统达到最佳性能
    parseRuntimeOption("dalvik.vm.heapstartsize", heapstartsizeOptsBuf, "-Xms", "4m");
    parseRuntimeOption("dalvik.vm.heapsize", heapsizeOptsBuf, "-Xmx", "16m");
    parseRuntimeOption("dalvik.vm.heapgrowthlimit", heapgrowthlimitOptsBuf, "-XX:HeapGrowthLimit=");
    parseRuntimeOption("dalvik.vm.heapminfree", heapminfreeOptsBuf, "-XX:HeapMinFree=");
    parseRuntimeOption("dalvik.vm.heapmaxfree", heapmaxfreeOptsBuf, "-XX:HeapMaxFree=");
    parseRuntimeOption("dalvik.vm.heaptargetutilization",
                       heaptargetutilizationOptsBuf, "-XX:HeapTargetUtilization=");
    ...

    //preloaded-classes文件内容是由WritePreloadedClassFile.java生成的，
    //在ZygoteInit类中会预加载工作将其中的classes提前加载到内存，以提高系统性能
    if (!hasFile("/system/etc/preloaded-classes")) {
        return -1;
    }

    //初始化虚拟机
    if (JNI_CreateJavaVM(pJavaVM, pEnv, &initArgs) < 0) {
        ALOGE("JNI_CreateJavaVM failed\n");
        return -1;
    }
}
```

### ` startReg()`

**JNI方法注册**

```
int AndroidRuntime::startReg(JNIEnv* env)
{
    //设置线程创建方法为javaCreateThreadEtc 【见小节2.4.1】
    androidSetCreateThreadFunc((android_create_thread_fn) javaCreateThreadEtc);

    env->PushLocalFrame(200);
    //进程NI方法的注册【见小节2.4.2】
    if (register_jni_procs(gRegJNI, NELEM(gRegJNI), env) < 0) {
        env->PopLocalFrame(NULL);
        return -1;
    }
    env->PopLocalFrame(NULL);
    return 0;
}
```

### `register_jni_procs()`

```
static int register_jni_procs(const RegJNIRec array[], size_t count, JNIEnv* env) {
    for (size_t i = 0; i < count; i++) {
        //【见小节2.4.3】
        if (array[i].mProc(env) < 0) {
            return -1;
        }
    }
    return 0;
}
```

### `gRegJNI.mProc`

```
static const RegJNIRec gRegJNI[] = {
    REG_JNI(register_com_android_internal_os_RuntimeInit),
    REG_JNI(register_android_os_Binder)，
    ...
}；
```

array[i]是指gRegJNI数组, 该数组有100多个成员。其中每一项成员都是通过**REG_JNI**宏定义的：

```
#define REG_JNI(name) { name }
struct RegJNIRec {
    int (*mProc)(JNIEnv*);
};
```

可见，调用`mProc`，就等价于调用其参数名所指向的函数。 例如`REG_JNI(register_com_android_internal_os_RuntimeInit).mProc`也就是指进入`register_com_android_internal_os_RuntimeInit方法`，接下来就继续以此为例来说明：

```
int register_com_android_internal_os_RuntimeInit(JNIEnv* env) {
    return jniRegisterNativeMethods(env, "com/android/internal/os/RuntimeInit",
        gMethods, NELEM(gMethods));
}

//gMethods：java层方法名与jni层的方法的一一映射关系
static JNINativeMethod gMethods[] = {
    { "nativeFinishInit", "()V",
        (void*) com_android_internal_os_RuntimeInit_nativeFinishInit },
    { "nativeZygoteInit", "()V",
        (void*) com_android_internal_os_RuntimeInit_nativeZygoteInit },
    { "nativeSetExitWithoutCleanup", "(Z)V",
        (void*) com_android_internal_os_RuntimeInit_nativeSetExitWithoutCleanup },
};
```

## `Threads.cpp`

### `androidSetCreateThreadFunc()`

```
void androidSetCreateThreadFunc(android_create_thread_fn func) {
    gCreateThreadFn = func;
}
```

虚拟机启动后startReg()过程，会设置线程创建函数指针`gCreateThreadFn`指向`javaCreateThreadEtc`.

## 小结

c++代码Zygote进程启动流程：

- 启动appRuntime
  - 创建虚拟机
  - jni方法注册
  - 调用ZygoteInit.main()方法

# 进入Java层

前面[小节2.2]AndroidRuntime.start()执行到最后通过反射调用到ZygoteInit.main(),见下文:

## `ZygoteInit`

### `main()`

```
public static void main(String argv[]) {
    try {
        RuntimeInit.enableDdms(); //开启DDMS功能
        SamplingProfilerIntegration.start();
        boolean startSystemServer = false;
        String socketName = "zygote";
        String abiList = null;
        for (int i = 1; i < argv.length; i++) {
            if ("start-system-server".equals(argv[i])) {
                startSystemServer = true;
            } else if (argv[i].startsWith(ABI_LIST_ARG)) {
                abiList = argv[i].substring(ABI_LIST_ARG.length());
            } else if (argv[i].startsWith(SOCKET_NAME_ARG)) {
                socketName = argv[i].substring(SOCKET_NAME_ARG.length());
            } else {
                throw new RuntimeException("Unknown command line argument: " + argv[i]);
            }
        }
        ...

        registerZygoteSocket(socketName); //为Zygote注册socket【见小节3.2】
        preload(); // 预加载类和资源【见小节3.3】
        SamplingProfilerIntegration.writeZygoteSnapshot();
        gcAndFinalize(); //GC操作
        if (startSystemServer) {
            startSystemServer(abiList, socketName);//启动system_server【见小节3.4】
        }
        caller = zygoteServer.runSelectLoop(abiList); //进入循环模式【见小节3.5】
        closeServerSocket();
    }catch (RuntimeException ex) {
        closeServerSocket();
        throw ex;
    }
     // We're in the child process and have exited the select loop. Proceed to execute the
     // command.
        if (caller != null) {
            caller.run();
        }
}
```

最后会调用`caller.run()`。

- 为Zygote注册socket
- 预加载类和资源
- 启动system_server
- 开启循环

### `registerZygoteSocket()`

```
private static void registerZygoteSocket(String socketName) {
    if (sServerSocket == null) {
        int fileDesc;
        final String fullSocketName = ANDROID_SOCKET_PREFIX + socketName;
        try {
            String env = System.getenv(fullSocketName);
            fileDesc = Integer.parseInt(env);
        } catch (RuntimeException ex) {
            ...
        }

        try {
            FileDescriptor fd = new FileDescriptor();
            fd.setInt$(fileDesc); //设置文件描述符
            sServerSocket = new LocalServerSocket(fd); //创建Socket的本地服务端
        } catch (IOException ex) {
            ...
        }
    }
}
```

#### `LocalServerSocket()`

```
impl = new LocalSocketImpl(fd);
        impl.listen(LISTEN_BACKLOG);
        localAddress = impl.getSockAddress();
```

### `preload()`

```
static void preload() {
    //预加载位于/system/etc/preloaded-classes文件中的类
    preloadClasses();

    //预加载资源，包含drawable和color资源
    preloadResources();

    //预加载OpenGL
    preloadOpenGL();

    //通过System.loadLibrary()方法，
    //预加载"android","compiler_rt","jnigraphics"这3个共享库
    preloadSharedLibraries();

    //预加载 文本连接符资源
    preloadTextResources();

    //仅用于zygote进程，用于内存共享的进程
    WebViewFactory.prepareWebViewInZygote();
}
```

执行Zygote进程的初始化,对于类加载，采用反射机制Class.forName()方法来加载。对于资源加载，主要是 com.android.internal.R.array.preloaded_drawables和com.android.internal.R.array.preloaded_color_state_lists，在应用程序中以com.android.internal.R.xxx开头的资源，便是此时由Zygote加载到内存的。

zygote进程内加载了preload()方法中的所有资源，当需要fork新进程时，采用copy on write技术，如下：

![zygote_fork](http://gityuan.com/images/boot/zygote/zygote_fork.jpg)

### `startSystemServer()`

```
private static boolean startSystemServer(String abiList, String socketName) throws MethodAndArgsCaller, RuntimeException {
    long capabilities = posixCapabilitiesAsBits(
        OsConstants.CAP_BLOCK_SUSPEND,
        OsConstants.CAP_KILL,
        OsConstants.CAP_NET_ADMIN,
        OsConstants.CAP_NET_BIND_SERVICE,
        OsConstants.CAP_NET_BROADCAST,
        OsConstants.CAP_NET_RAW,
        OsConstants.CAP_SYS_MODULE,
        OsConstants.CAP_SYS_NICE,
        OsConstants.CAP_SYS_RESOURCE,
        OsConstants.CAP_SYS_TIME,
        OsConstants.CAP_SYS_TTY_CONFIG
    );
    //参数准备
    String args[] = {
        "--setuid=1000",
        "--setgid=1000",
        "--setgroups=1001,1002,1003,1004,1005,1006,1007,1008,1009,1010,1018,1021,1032,3001,3002,3003,3006,3007",
        "--capabilities=" + capabilities + "," + capabilities,
        "--nice-name=system_server",
        "--runtime-args",
        "com.android.server.SystemServer",
    };

    ZygoteConnection.Arguments parsedArgs = null;
    int pid;
    try {
        //用于解析参数，生成目标格式
        parsedArgs = new ZygoteConnection.Arguments(args);
        ZygoteConnection.applyDebuggerSystemProperty(parsedArgs);
        ZygoteConnection.applyInvokeWithSystemProperty(parsedArgs);

        // fork子进程，用于运行system_server
        pid = Zygote.forkSystemServer(
                parsedArgs.uid, parsedArgs.gid,
                parsedArgs.gids,
                parsedArgs.debugFlags,
                null,
                parsedArgs.permittedCapabilities,
                parsedArgs.effectiveCapabilities);
    } catch (IllegalArgumentException ex) {
        throw new RuntimeException(ex);
    }

    //进入子进程system_server
    if (pid == 0) {
        if (hasSecondZygote(abiList)) {
            waitForSecondaryZygote(socketName);
        }
        // 完成system_server进程剩余的工作
        handleSystemServerProcess(parsedArgs);
    }
    return true;
}
```

准备参数并fork新进程，从上面可以看出system server进程参数信息为uid=1000,gid=1000,进程名为sytem_server，从zygote进程fork新进程后，需要关闭zygote原有的socket。另外，对于有两个zygote进程情况，需等待第2个zygote创建完成。更多详情见[Android系统启动-systemServer上篇](http://gityuan.com/2016/02/14/android-system-server/)。

## `Zygote`

### `forkSystemServer`

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

    native private static int nativeForkSystemServer(int uid, int gid, int[] gids, int runtimeFlags,
            int[][] rlimits, long permittedCapabilities, long effectiveCapabilities);

```

调用底层创建system_server进程。

## `ZygoteInit`

### `runSelectLoop()`

```
private static void runSelectLoop(String abiList) throws MethodAndArgsCaller {
    ArrayList<FileDescriptor> fds = new ArrayList<FileDescriptor>();
    ArrayList<ZygoteConnection> peers = new ArrayList<ZygoteConnection>();
    //sServerSocket是socket通信中的服务端，即zygote进程。保存到fds[0]
    fds.add(sServerSocket.getFileDescriptor());
    peers.add(null);

    while (true) {
        StructPollfd[] pollFds = new StructPollfd[fds.size()];
        for (int i = 0; i < pollFds.length; ++i) {
            pollFds[i] = new StructPollfd();
            pollFds[i].fd = fds.get(i);
            pollFds[i].events = (short) POLLIN;
        }
        try {
             //处理轮询状态，当pollFds有事件到来则往下执行，否则阻塞在这里
            Os.poll(pollFds, -1);
        } catch (ErrnoException ex) {
            ...
        }
        
        for (int i = pollFds.length - 1; i >= 0; --i) {
            //采用I/O多路复用机制，当接收到客户端发出连接请求 或者数据处理请求到来，则往下执行；
            // 否则进入continue，跳出本次循环。
            if ((pollFds[i].revents & POLLIN) == 0) {
                continue;
            }
            if (i == 0) {
                //即fds[0]，代表的是sServerSocket，则意味着有客户端连接请求；
                // 则创建ZygoteConnection对象,并添加到fds。
                ZygoteConnection newPeer = acceptCommandPeer(abiList);
                peers.add(newPeer);
                fds.add(newPeer.getFileDesciptor()); //添加到fds.
            } else {
                //i>0，则代表通过socket接收来自对端的数据，并执行相应操作【见小节3.6】
                boolean done = peers.get(i).runOnce();
                if (done) {
                    peers.remove(i);
                    fds.remove(i); //处理完则从fds中移除该文件描述符
                }
            }
        }
    }
}
```

Zygote采用高效的I/O多路复用机制，保证在没有客户端连接请求或数据处理时休眠，否则响应客户端的请求。

### `ZygoteConnection`

### `runOnce()`

```
boolean runOnce() throws ZygoteInit.MethodAndArgsCaller {

    String args[];
    Arguments parsedArgs = null;
    FileDescriptor[] descriptors;

    try {
        //读取socket客户端发送过来的参数列表
        args = readArgumentList();
        descriptors = mSocket.getAncillaryFileDescriptors();
    } catch (IOException ex) {
        ...
        return true;
    }
    ...

    try {
        //将binder客户端传递过来的参数，解析成Arguments对象格式
        parsedArgs = new Arguments(args);
        ...
        //【见小节7】
        pid = Zygote.forkAndSpecialize(parsedArgs.uid, parsedArgs.gid, parsedArgs.gids,
                parsedArgs.debugFlags, rlimits, parsedArgs.mountExternal, parsedArgs.seInfo,
                parsedArgs.niceName, fdsToClose, parsedArgs.instructionSet,
                parsedArgs.appDataDir);
    } catch (Exception e) {
        ...
    }

    try {
        if (pid == 0) {
            //子进程执行
            IoUtils.closeQuietly(serverPipeFd);
            serverPipeFd = null;
            //进入子进程流程
            handleChildProc(parsedArgs, descriptors, childPipeFd, newStderr);
            return true;
        } else {
            //父进程执行
            IoUtils.closeQuietly(childPipeFd);
            childPipeFd = null;
            return handleParentProc(pid, descriptors, serverPipeFd, parsedArgs);
        }
    } finally {
        IoUtils.closeQuietly(childPipeFd);
        IoUtils.closeQuietly(serverPipeFd);
    }
}
```

# 总结

Zygote启动过程的调用流程图：

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



# fork进程

## `Zygote`

### `forkAndSpecialize`

```
public static int forkAndSpecialize(int uid, int gid, int[] gids, int debugFlags, int[][] rlimits, int mountExternal, String seInfo, String niceName, int[] fdsToClose, String instructionSet, String appDataDir) {
    VM_HOOKS.preFork(); //【见小节8】
    int pid = nativeForkAndSpecialize(
              uid, gid, gids, debugFlags, rlimits, mountExternal, seInfo, niceName, fdsToClose,
              instructionSet, appDataDir); //【见小节9】
    ...
    VM_HOOKS.postForkCommon(); //【见小节11】
    return pid;
}
```

VM_HOOKS是Zygote对象的静态成员变量：VM_HOOKS = new ZygoteHooks();

## `ZygoteHooks`

```
public void preFork() {
    Daemons.stop(); //停止4个Daemon子线程【见小节8.1】
    waitUntilAllThreadsStopped(); //等待所有子线程结束【见小节8.2】
    token = nativePreFork(); //完成gc堆的初始化工作【见小节8.3】
}
```

## `Daemons`

### `stop()`

```

public static void stop() {
    HeapTaskDaemon.INSTANCE.stop(); //Java堆整理线程
    ReferenceQueueDaemon.INSTANCE.stop(); //引用队列线程
    FinalizerDaemon.INSTANCE.stop(); //析构线程
    FinalizerWatchdogDaemon.INSTANCE.stop(); //析构监控线程
}
```

此处守护线程Stop方式是先调用目标线程interrrupt()方法，然后再调用目标线程join()方法，等待线程执行完成。

### ` waitUntilAllThreadsStopped()`

```
private static void waitUntilAllThreadsStopped() {
    File tasks = new File("/proc/self/task");
    // 当/proc中线程数大于1，就出让CPU直到只有一个线程，才退出循环
    while (tasks.list().length > 1) {
        Thread.yield();
    }
}
```

### `nativePreFork()`

nativePreFork通过JNI最终调用如下方法：

## dalvik_system_ZygoteHooks

### `ZygoteHooks_nativePreFork`

```
static jlong ZygoteHooks_nativePreFork(JNIEnv* env, jclass) {
    Runtime* runtime = Runtime::Current();
    runtime->PreZygoteFork(); // 见下文
    if (Trace::GetMethodTracingMode() != TracingMode::kTracingInactive) {
      Trace::Pause();
    }
    //将线程转换为long型并保存到token，该过程是非安全的
    return reinterpret_cast<jlong>(ThreadForEnv(env));
}
```

至于runtime->PreZygoteFork的过程：

```
void Runtime::PreZygoteFork() {
    // 堆的初始化工作。这里就不继续再往下追art虚拟机
    heap_->PreZygoteFork();
}
```

VM_HOOKS.preFork()的主要功能便是停止Zygote的4个Daemon子线程的运行，等待并确保Zygote是单线程（用于提升fork效率），并等待这些线程的停止，初始化gc堆的工作, 并将线程转换为long型并保存到token

## `Zygote`

### `nativeForkAndSpecialize()`

## `com_android_internal_os_Zygote`

### `com_android_internal_os_Zygote_nativeForkAndSpecialize()`

```
static jint com_android_internal_os_Zygote_nativeForkAndSpecialize(
    JNIEnv* env, jclass, jint uid, jint gid, jintArray gids,
    jint debug_flags, jobjectArray rlimits,
    jint mount_external, jstring se_info, jstring se_name,
    jintArray fdsToClose, jstring instructionSet, jstring appDataDir) {
    // 将CAP_WAKE_ALARM赋予蓝牙进程
    jlong capabilities = 0;
    if (uid == AID_BLUETOOTH) {
        capabilities |= (1LL << CAP_WAKE_ALARM);
    }
    //【见流程10】
    return ForkAndSpecializeCommon(env, uid, gid, gids, debug_flags,
            rlimits, capabilities, capabilities, mount_external, se_info,
            se_name, false, fdsToClose, instructionSet, appDataDir);
}
```

### `ForkAndSpecializeCommon()`

```
static pid_t ForkAndSpecializeCommon(JNIEnv* env, uid_t uid, gid_t gid, jintArray javaGids, jint debug_flags, jobjectArray javaRlimits, jlong permittedCapabilities, jlong effectiveCapabilities, jint mount_external, jstring java_se_info, jstring java_se_name, bool is_system_server, jintArray fdsToClose, jstring instructionSet, jstring dataDir) {
  //设置子进程的signal信号处理函数
  SetSigChldHandler();
  //fork子进程 【见流程10.1】
  pid_t pid = fork();
  if (pid == 0) { //进入子进程
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
    //在Zygote子进程中，设置信号SIGCHLD的处理器恢复为默认行为
    UnsetSigChldHandler();
    //等价于调用zygote.callPostForkChildHooks() 【见流程10.2】
    env->CallStaticVoidMethod(gZygoteClass, gCallPostForkChildHooks, debug_flags,
                              is_system_server ? NULL : instructionSet);
    ...

  } else if (pid > 0) {
    //进入父进程，即Zygote进程
  }
  return pid;
}
```

### fork()

fork()采用copy on write技术，这是linux创建进程的标准方法，调用一次，返回两次，返回值有3种类型。

- 父进程中，fork返回新创建的子进程的pid;
- 子进程中，fork返回0；
- 当出现错误时，fork返回负数。（当进程数超过上限或者系统内存不足时会出错）

fork()的主要工作是寻找空闲的进程号pid，然后从父进程拷贝进程信息，例如数据段和代码段，fork()后子进程要执行的代码等。 Zygote进程是所有Android进程的母体，包括system_server和各个App进程。zygote利用fork()方法生成新进程，对于新进程A复用Zygote进程本身的资源，再加上新进程A相关的资源，构成新的应用进程A。其中下图中Zygote进程的libc、vm、preloaded classes、preloaded resources是如何生成的，可查看另一个文章[Android系统启动-zygote篇](http://gityuan.com/2016/02/13/android-zygote/#preload)，见下图：

![zygote_fork](http://gityuan.com/images/boot/zygote/zygote_fork.jpg)

copy-on-write过程：当父子进程任一方修改内存数据时（这是on-write时机），才发生缺页中断，从而分配新的物理内存（这是copy操作）。

copy-on-write原理：写时拷贝是指子进程与父进程的页表都所指向同一个块物理内存，fork过程只拷贝父进程的页表，并标记这些页表是只读的。父子进程共用同一份物理内存，如果父子进程任一方想要修改这块物理内存，那么会触发缺页异常(page fault)，Linux收到该中断便会创建新的物理内存，并将两个物理内存标记设置为可写状态，从而父子进程都有各自独立的物理内存。

##### 10.1.1 fork.cpp

[-> bionic/fork.cpp]

```
#define FORK_FLAGS (CLONE_CHILD_SETTID | CLONE_CHILD_CLEARTID | SIGCHLD)
int fork() {
  __bionic_atfork_run_prepare(); //[见小节2.1.1]

  pthread_internal_t* self = __get_thread();

  //fork期间，获取父进程pid，并使其缓存值无效
  pid_t parent_pid = self->invalidate_cached_pid();
  //系统调用【见小节2.2】
  int result = syscall(__NR_clone, FORK_FLAGS, NULL, NULL, NULL, &(self->tid));
  if (result == 0) {
    self->set_cached_pid(gettid());
    __bionic_atfork_run_child(); //fork完成执行子进程回调方法[见小节2.1.1]
  } else {
    self->set_cached_pid(parent_pid);
    __bionic_atfork_run_parent(); //fork完成执行父进程回调方法
  }
  return result;
}
```

功能说明：在执行syscall的前后都有相应的回调方法。

- __bionic_atfork_run_prepare： fork完成前，父进程回调方法
- __bionic_atfork_run_child： fork完成后，子进程回调方法
- __bionic_atfork_run_paren： fork完成后，父进程回调方法

以上3个方法的实现都位于bionic/pthread_atfork.cpp。如果有需要，可以扩展该回调方法，添加相关的业务需求。

#### 10.2 Zygote.callPostForkChildHooks

[-> Zygote.java]

```
private static void callPostForkChildHooks(int debugFlags, boolean isSystemServer, String instructionSet) {
    //调用ZygoteHooks.postForkChild()
    VM_HOOKS.postForkChild(debugFlags, isSystemServer, instructionSet);
}
```

[-> ZygoteHooks.java]

```
public void postForkChild(int debugFlags, String instructionSet) {
    //【见流程10.3】
    nativePostForkChild(token, debugFlags, instructionSet);
    Math.setRandomSeedInternal(System.currentTimeMillis());
}
```

在这里，设置了新进程Random随机数种子为当前系统时间，也就是在进程创建的那一刻就决定了未来随机数的情况，也就是伪随机。

#### 10.3 nativePostForkChild

nativePostForkChild通过JNI最终调用调用如下方法：

[-> dalvik_system_ZygoteHooks.cc]

```
static void ZygoteHooks_nativePostForkChild(JNIEnv* env, jclass, jlong token, jint debug_flags, jstring instruction_set) {
    //此处token是由[小节8.3]创建的，记录着当前线程
    Thread* thread = reinterpret_cast<Thread*>(token);
    //设置新进程的主线程id
    thread->InitAfterFork();
    ..
    if (instruction_set != nullptr) {
      ScopedUtfChars isa_string(env, instruction_set);
      InstructionSet isa = GetInstructionSetFromString(isa_string.c_str());
      Runtime::NativeBridgeAction action = Runtime::NativeBridgeAction::kUnload;
      if (isa != kNone && isa != kRuntimeISA) {
        action = Runtime::NativeBridgeAction::kInitialize;
      }
      //【见流程10.4】
      Runtime::Current()->DidForkFromZygote(env, action, isa_string.c_str());
    } else {
      Runtime::Current()->DidForkFromZygote(env, Runtime::NativeBridgeAction::kUnload, nullptr);
    }
}
```

#### 10.4 DidForkFromZygote

[-> Runtime.cc]

```
void Runtime::DidForkFromZygote(JNIEnv* env, NativeBridgeAction action, const char* isa) {
  is_zygote_ = false;
  if (is_native_bridge_loaded_) {
    switch (action) {
      case NativeBridgeAction::kUnload:
        UnloadNativeBridge(); //卸载用于跨平台的桥连库
        is_native_bridge_loaded_ = false;
        break;
      case NativeBridgeAction::kInitialize:
        InitializeNativeBridge(env, isa);//初始化用于跨平台的桥连库
        break;
    }
  }
  //创建Java堆处理的线程池
  heap_->CreateThreadPool();
  //重置gc性能数据，以保证进程在创建之前的GCs不会计算到当前app上。
  heap_->ResetGcPerformanceInfo();
  if (jit_.get() == nullptr && jit_options_->UseJIT()) {
    //当flag被设置，并且还没有创建JIT时，则创建JIT
    CreateJit();
  }
  //设置信号处理函数
  StartSignalCatcher();
  //启动JDWP线程，当命令debuger的flags指定"suspend=y"时，则暂停runtime
  Dbg::StartJdwp();
}
```

关于信号处理过程，其代码位于signal_catcher.cc文件中，后续会单独讲解。

### 11. postForkCommon

[-> ZygoteHooks.java]

```
public void postForkCommon() {
    Daemons.start();
}

public static void start() {
    ReferenceQueueDaemon.INSTANCE.start();
    FinalizerDaemon.INSTANCE.start();
    FinalizerWatchdogDaemon.INSTANCE.start();
    HeapTaskDaemon.INSTANCE.start();
}
```

VM_HOOKS.postForkCommon的主要功能是在fork新进程后，启动Zygote的4个Daemon线程，java堆整理，引用队列，以及析构线程。

### 12. forkAndSpecialize小结

该方法主要功能：

- preFork： 停止Zygote的4个Daemon子线程的运行，初始化gc堆；
- nativeForkAndSpecialize：调用`fork()`创建新进程，设置新进程的主线程id，重置gc性能数据，设置信号处理函数等功能。
- postForkCommon：启动4个Deamon子线程。

其调用关系链：

```
Zygote.forkAndSpecialize
    ZygoteHooks.preFork
        Daemons.stop
        ZygoteHooks.nativePreFork
            dalvik_system_ZygoteHooks.ZygoteHooks_nativePreFork
                Runtime::PreZygoteFork
                    heap_->PreZygoteFork()
    Zygote.nativeForkAndSpecialize
        com_android_internal_os_Zygote.ForkAndSpecializeCommon
            fork()
            Zygote.callPostForkChildHooks
                ZygoteHooks.postForkChild
                    dalvik_system_ZygoteHooks.nativePostForkChild
                        Runtime::DidForkFromZygote
    ZygoteHooks.postForkCommon
        Daemons.start
```

**时序图：** 点击查看[大图](http://gityuan.com/images/android-process/fork_and_specialize.jpg)

![fork_and_specialize](http://gityuan.com/images/android-process/fork_and_specialize.jpg)

到此App进程已完成了创建的所有工作，接下来开始新创建的App进程的工作。在前面ZygoteConnection.runOnce方法中，zygote进程执行完`forkAndSpecialize()`后，新创建的App进程便进入`handleChildProc()`方法，下面的操作运行在App进程。

