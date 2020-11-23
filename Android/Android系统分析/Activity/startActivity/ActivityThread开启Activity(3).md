---
title: ActivityThread开启Activity
date: 2020-05-28 15:30:08
tags: 
	- Android
	- Android系统分析
	- startActivity
categories: 
	- Android
	- Android系统分析
	- Activity
	- startActivity
---

# ActivityThread源码分析(3)

在Zygote进程中会创建新的子进程，然后通过反射调用`ActivityThread`中的Main方法.可以理解为这里是进程的入口。

```
public static void main(String[] args) {
       //初始化UserEnvironment内部类
        Environment.initForCurrentUser();     
				//更改此进程的argv [0]参数。 这对于在诸如“ ps”命令之类的内容中显示更多描述性信息很有用。
        Process.setArgV0("<pre-initialized>");
				//开启主线程中loop
        Looper.prepareMainLooper();
				//创建    ActivityThread
        ActivityThread thread = new ActivityThread();
        thread.attach(false, startSeq);
				//创建主线程Handler final H mH = new H();
        if (sMainThreadHandler == null) {
            sMainThreadHandler = thread.getHandler();
        }
        //主线程开始轮训
        Looper.loop();

        throw new RuntimeException("Main thread loop unexpectedly exited");
    }
```

## 构造函数

```
ActivityThread() {
        mResourcesManager = ResourcesManager.getInstance();
    }
```

看名字应该是一个资源管理类

### `attach()`

```
private void attach(boolean system, long startSeq) {
				sCurrentActivityThread = this;
				//从main中进入是false
        mSystemThread = system;
        if (!system) {
         ViewRootImpl.addFirstDrawHandler();
      			 //获取AMS
        		final IActivityManager mgr = ActivityManager.getService();
            try {
            		//调用AMS中的方法 mAppThread是ApplicationThread aidl实现类
                mgr.attachApplication(mAppThread, startSeq);
            } catch (RemoteException ex) {
                throw ex.rethrowFromSystemServer();
            }  
        }else{
                mInstrumentation = new Instrumentation();
                mInstrumentation.basicInit(this);
                ContextImpl context = ContextImpl.createAppContext(
                this, getSystemContext().mPackageInfo);
                mInitialApplication = context.mPackageInfo.makeApplication(true, null);
                mInitialApplication.onCreate();
        }
 ViewRootImpl.addConfigCallback(configChangedCallback);

}       
```

## `ActivityManagerService`

### `attachApplication()`

```
@Override
    public final void attachApplication(IApplicationThread thread, long startSeq) {
        synchronized (this) {
            int callingPid = Binder.getCallingPid();
            final int callingUid = Binder.getCallingUid();
            final long origId = Binder.clearCallingIdentity();
            attachApplicationLocked(thread, callingPid, callingUid, startSeq);
            //恢复传入IPC的身份对当前线程回，是由{@link #clearCallingIdentity}返回先前的身份。
            Binder.restoreCallingIdentity(origId);
        }
    }
```

### `attachApplicationLocked()`

```
@GuardedBy("this")
    private final boolean attachApplicationLocked(IApplicationThread thread,
            int pid, int callingUid, long startSeq) {
						...
						if (app.isolatedEntryPoint != null) {
                // This is an isolated process which should just call an entry point instead of
                // being bound to an application.
                thread.runIsolatedEntryPoint(app.isolatedEntryPoint, app.isolatedEntryPointArgs);
            } else if (app.instr != null) {
                thread.bindApplication(processName, appInfo, providers,
                        app.instr.mClass,
                        profilerInfo, app.instr.mArguments,
                        app.instr.mWatcher,
                        app.instr.mUiAutomationConnection, testMode,
                        mBinderTransactionTrackingEnabled, enableTrackAllocation,
                        isRestrictedBackupMode || !normalMode, app.persistent,
                        new Configuration(getGlobalConfiguration()), app.compat,
                        getCommonServicesLocked(app.isolated),
                        mCoreSettingsObserver.getCoreSettingsLocked(),
                        buildSerial, isAutofillCompatEnabled);
            } else {
                thread.bindApplication(processName, appInfo, providers, null, profilerInfo,
                        null, null, null, testMode,
                        mBinderTransactionTrackingEnabled, enableTrackAllocation,
                        isRestrictedBackupMode || !normalMode, app.persistent,
                        new Configuration(getGlobalConfiguration()), app.compat,
                        getCommonServicesLocked(app.isolated),
                        mCoreSettingsObserver.getCoreSettingsLocked(),
                        buildSerial, isAutofillCompatEnabled);
            }
            ...
            // See if the top visible activity is waiting to run in this process...
            //是不是有需要运行的Activity
            if (normalMode) {
            try {
                if (mStackSupervisor.attachApplicationLocked(app)) {
                    didSomething = true;
                }
            } catch (Exception e) {
                Slog.wtf(TAG, "Exception thrown launching activities in " + app, e);
                badApp = true;
            }
        }
        ...
}
```

这里主要作两件事：

- 创建Application：`thread.bindApplication`
- 开启Activity：`mStackSupervisor.attachApplicationLocked(app)`

先来看看如何创建Application

# Application创建

## `ActivityThread`

### bindApplication()

```
public final void bindApplication(String processName, ApplicationInfo appInfo,
                List<ProviderInfo> providers, ComponentName instrumentationName,
                ProfilerInfo profilerInfo, Bundle instrumentationArgs,
                IInstrumentationWatcher instrumentationWatcher,
                IUiAutomationConnection instrumentationUiConnection, int debugMode,
                boolean enableBinderTracking, boolean trackAllocation,
                boolean isRestrictedBackupMode, boolean persistent, Configuration config,
                CompatibilityInfo compatInfo, Map services, Bundle coreSettings,
                String buildSerial, boolean autofillCompatibilityEnabled) {
                	...
                	sendMessage(H.BIND_APPLICATION, data);                
                }
```

 sendMessage发送

```
mH.sendMessage(msg);
```

所以找到mH的消息接受回调方法找对应的逻辑：

### `mH.handlerMessage()===BIND_APPLICATION`

```
 switch (msg.what) {
                case BIND_APPLICATION:
                    Trace.traceBegin(Trace.TRACE_TAG_ACTIVITY_MANAGER, "bindApplication");
                    AppBindData data = (AppBindData)msg.obj;
                    handleBindApplication(data);
                    Trace.traceEnd(Trace.TRACE_TAG_ACTIVITY_MANAGER);
                    break;
```

### `handleBindApplication()`

```
private void handleBindApplication(AppBindData data) {
				...
				updateDefaultDensity();
				...
				//设置时区
				...
				//设置代理
				...
				//初始化 ContextImpl
				final ContextImpl appContext = ContextImpl.createAppContext(this, data.info);
				...
				 //获取其classLoader
        final ClassLoader cl = instrContext.getClassLoader();
        //构建Instrumentation 
        mInstrumentation = (Instrumentation)
            cl.loadClass(data.instrumentationName.getClassName()).newInstance();
				...
				//创建Application
				app = data.info.makeApplication(data.restrictedBackupMode, null);
				...
				//最后执行的是Application的onCreate方法
				mInstrumentation.callApplicationOnCreate(app);
}
```

在这里创建了Application并调用onCreate方法

### `makeApplication()`

```
public Application makeApplication(boolean forceDefaultAppClass,
            Instrumentation instrumentation) {
				if (mApplication != null) {
            return mApplication;
        }
        String appClass = mApplicationInfo.className;
        if (forceDefaultAppClass || (appClass == null)) {
            appClass = "android.app.Application";
        }
        try {
            java.lang.ClassLoader cl = getClassLoader();
            if (!mPackageName.equals("android")) {
                Trace.traceBegin(Trace.TRACE_TAG_ACTIVITY_MANAGER,
                        "initializeJavaContextClassLoader");
                initializeJavaContextClassLoader();
                Trace.traceEnd(Trace.TRACE_TAG_ACTIVITY_MANAGER);
            }
            //获取上下文
            ContextImpl appContext = ContextImpl.createAppContext(mActivityThread, this);
            //创建Application
            app = mActivityThread.mInstrumentation.newApplication(
                    cl, appClass, appContext);
            appContext.setOuterContext(app);
        } catch (Exception e) {
            if (!mActivityThread.mInstrumentation.onException(app, e)) {
                Trace.traceEnd(Trace.TRACE_TAG_ACTIVITY_MANAGER);
                throw new RuntimeException(
                    "Unable to instantiate application " + appClass
                    + ": " + e.toString(), e);
            }
        }
        ...
        return app;
}
```

看看application如何创建的

## `Instrumentation`

### `newApplication()`

```
public Application newApplication(ClassLoader cl, String className, Context context)
            throws InstantiationException, IllegalAccessException, 
            ClassNotFoundException {
        Application app = getFactory(context.getPackageName())
                .instantiateApplication(cl, className);
        app.attach(context);
        return app;
    }
```

1. 创建application
2. 调用application的attach()方法

## `AppComponentFactory`

### `instantiateApplication()`

```
public @NonNull Application instantiateApplication(@NonNull ClassLoader cl,
            @NonNull String className)
            throws InstantiationException, IllegalAccessException, ClassNotFoundException {
        return (Application) cl.loadClass(className).newInstance();
    }
```

**利用反射去创建application，然后调用attach()方法，在执行onCreate方法**

# Activity如何开启

回到ActivityManagerService#attachApplicationLocked方法

## `ActivityManagerService`

### `attachApplicationLocked()`

上面说到这个方法会启动Activity

```
// See if the top visible activity is waiting to run in this process...
        if (normalMode) {
            try {
                if (mStackSupervisor.attachApplicationLocked(app)) {
                    didSomething = true;
                }
            } catch (Exception e) {
                Slog.wtf(TAG, "Exception thrown launching activities in " + app, e);
                badApp = true;
            }
        }
```

## `ActivityStackSupervisor`

### `attachApplicationLocked()`

```
boolean attachApplicationLocked(ProcessRecord app) throws RemoteException {
				...
				if (realStartActivityLocked(activity, app,top == activity /* andResume */, true /* checkConfig */)) {
          didSomething = true;
        }
        ...                        
}
```

### `realStartActivityLocked()`

这个方法很熟悉，去前面文章查查，有印象。

一分钟后。。。

在当前类中`startSpecificActivityLocked`方法下：

如果进程存在就走`realStartActivityLocked`如果没有进程则AMS创建一个进程`mService.startProcessLocked`。

上一篇文章说的是创建进程的流程，如果进程存在，直接创建Activity就好了。

开始分析`realStartActivityLocked`

```
final boolean realStartActivityLocked(ActivityRecord r, ProcessRecord app,
            boolean andResume, boolean checkConfig) throws RemoteException {
            //Activity栈
            final TaskRecord task = r.getTask();
       			final ActivityStack stack = task.getStack();
       			...
       			//创建activity启动事务。
       			final ClientTransaction clientTransaction = ClientTransaction.obtain(app.thread,
                        r.appToken);
                //构建LaunchActivityItem对象,并传入clientTransaction中,用作callback
                clientTransaction.addCallback(LaunchActivityItem.obtain(new Intent(r.intent),
                        System.identityHashCode(r), r.info,
                        // TODO: Have this take the merged configuration instead of separate global
                        // and override configs.
                        mergedConfiguration.getGlobalConfiguration(),
                        mergedConfiguration.getOverrideConfiguration(), r.compat,
                        r.launchedFromPackage, task.voiceInteractor, app.repProcState, r.icicle,
                        r.persistentState, results, newIntents, mService.isNextTransitionForward(),
                        profilerInfo));
}
```

注意`clientTransaction.addCallback(LaunchActivityItem.obtain(new Intent(r.intent),`添加回调的类型是`LaunchActivityItem类`。然后调用`mService.getLifecycleManager().scheduleTransaction(clientTransaction);`

方法。mService是AMS

## `ActivityManagerService`

### `getLifecycleManager()`

```
ClientLifecycleManager getLifecycleManager() {
        return mLifecycleManager;
    }
```

## `ClientLifecycleManager`

### `scheduleTransaction()`

```
void scheduleTransaction(ClientTransaction transaction) throws RemoteException {
        final IApplicationThread client = transaction.getClient();
        transaction.schedule();
        if (!(client instanceof Binder)) {
            // If client is not an instance of Binder - it's a remote call and at this point it is
            // safe to recycle the object. All objects used for local calls will be recycled after
            // the transaction is executed on client in ActivityThread.
            transaction.recycle();
        }
    }
```

## `ClientTransaction`

### `schedule()`

```
public void schedule() throws RemoteException {
        mClient.scheduleTransaction(this);
    }
```

mClient的类型是`private IApplicationThread mClient;`IApplicationThread的实现类是`ActivityThread`类

## `ActivityThread`

```
private class ApplicationThread extends IApplicationThread.Stub {
}
```

ApplicationThread 是Aidl服务端实现类，夸进程通信。

### `scheduleTransaction()`

```
@Override
        public void scheduleTransaction(ClientTransaction transaction) throws RemoteException {
            ActivityThread.this.scheduleTransaction(transaction);
        }
```

### `ActivityThread.this.scheduleTransaction()`

```
void scheduleTransaction(ClientTransaction transaction) {
        transaction.preExecute(this);
        sendMessage(ActivityThread.H.EXECUTE_TRANSACTION, transaction);
    }
```

使用Handler发送`EXECUTE_TRANSACTION`类型,找到mH的回调。

### `handleMessage() `

```
case EXECUTE_TRANSACTION:
                    final ClientTransaction transaction = (ClientTransaction) msg.obj;
                    mTransactionExecutor.execute(transaction);
                    if (isSystem()) {
                        // Client transactions inside system process are recycled on the client side
                        // instead of ClientLifecycleManager to avoid being cleared before this
                        // message is handled.
                        transaction.recycle();
                    }
                    // TODO(lifecycler): Recycle locally scheduled transactions.
                    break;
```

## `TransactionExecutor`

### `execute()`

```
public void execute(ClientTransaction transaction) {
        final IBinder token = transaction.getActivityToken();
        log("Start resolving transaction for client: " + mTransactionHandler + ", token: " + token);

        executeCallbacks(transaction);

        executeLifecycleState(transaction);
        mPendingActions.clear();
        log("End resolving transaction");
    }
```

### `executeCallbacks()`

```
    public void executeCallbacks(ClientTransaction transaction) {
        ...
        for (int i = 0; i < size; ++i) {
            final ClientTransactionItem item = callbacks.get(i);
           ...
            item.execute(mTransactionHandler, token, mPendingActions);
            item.postExecute(mTransactionHandler, token, mPendingActions);            
            ...
        	}
        	...
}
```

这里的callback就是前面的`LaunchActivityItem`

```
public class LaunchActivityItem extends ClientTransactionItem {
}
```

实际上调用的是`LaunchActivityItem#execute()`

## `LaunchActivityItem`

### `execute()`

### `preExecute()`

```
@Override
    public void preExecute(ClientTransactionHandler client, IBinder token) {
        client.updateProcessState(mProcState, false);
        client.updatePendingConfiguration(mCurConfig);
    }

    @Override
    public void execute(ClientTransactionHandler client, IBinder token,
            PendingTransactionActions pendingActions) {
        Trace.traceBegin(TRACE_TAG_ACTIVITY_MANAGER, "activityStart");
        ActivityClientRecord r = new ActivityClientRecord(token, mIntent, mIdent, mInfo,
                mOverrideConfig, mCompatInfo, mReferrer, mVoiceInteractor, mState, mPersistentState,
                mPendingResults, mPendingNewIntents, mIsForward,
                mProfilerInfo, client);
        client.handleLaunchActivity(r, pendingActions, null /* customIntent */);
        Trace.traceEnd(TRACE_TAG_ACTIVITY_MANAGER);
    }
```

`client`是一个`ClientTransactionHandler`类型是一个抽象类，

```
public abstract class ClientTransactionHandler {
}
```

他的具体实现类是`ActivityThread`

## `ActivityThread`

```
public final class ActivityThread extends ClientTransactionHandler {
}
```

### `handleLaunchActivity()`

```
public Activity handleLaunchActivity(ActivityClientRecord r,PendingTransactionActions pendingActions, 			Intent customIntent) {
			...
			if (a != null) {
            r.createdConfig = new Configuration(mConfiguration);
            reportSizeConfigurations(r);
            if (!r.activity.mFinished && pendingActions != null) {
                pendingActions.setOldState(r.state);
                pendingActions.setRestoreInstanceState(true);
                pendingActions.setCallOnPostCreate(true);
            }
        } else {
            // If there was an error, for any reason, tell the activity manager to stop us.
            try {
                ActivityManager.getService()
                        .finishActivity(r.token, Activity.RESULT_CANCELED, null,
                                Activity.DONT_FINISH_TASK_WITH_ACTIVITY);
            } catch (RemoteException ex) {
                throw ex.rethrowFromSystemServer();
            }
        }
			...
}
```

### `performLaunchActivity`

```
private Activity performLaunchActivity(ActivityClientRecord r, Intent customIntent) {
			...
			//创建上下文
			ContextImpl appContext = createBaseContextForActivity(r);
        Activity activity = null;
        try {
        //获取类加载器
            java.lang.ClassLoader cl = appContext.getClassLoader();
            //创建Activity
            activity = mInstrumentation.newActivity(
                    cl, component.getClassName(), r.intent);
            StrictMode.incrementExpectedActivityCount(activity.getClass());
            r.intent.setExtrasClassLoader(cl);
            r.intent.prepareToEnterProcess();
            if (r.state != null) {
                r.state.setClassLoader(cl);
            }
        } catch (Exception e) {
            if (!mInstrumentation.onException(activity, e)) {
                throw new RuntimeException(
                    "Unable to instantiate activity " + component
                    + ": " + e.toString(), e);
            }
        }
        ...
        //调用attach方法
        activity.attach(appContext, this, getInstrumentation(), r.token,
                        r.ident, app, r.intent, r.activityInfo, title, r.parent,
                        r.embeddedID, r.lastNonConfigurationInstances, config,
                        r.referrer, r.voiceInteractor, window, r.configCallback);
        ...
        //调用onCreate方法
        if (r.isPersistable()) {
                    mInstrumentation.callActivityOnCreate(activity, r.state, r.persistentState);
                } else {
                    mInstrumentation.callActivityOnCreate(activity, r.state);
                }
        ...      
}
```

- 创建Activity()
- 调用attach()
- 调用onCreate()

## `Instrumentation`

### `newActivity()`

````
public Activity newActivity(ClassLoader cl, String className,
            Intent intent)
            throws InstantiationException, IllegalAccessException,
            ClassNotFoundException {
        String pkg = intent != null && intent.getComponent() != null
                ? intent.getComponent().getPackageName() : null;
        return getFactory(pkg).instantiateActivity(cl, className, intent);
    }
````

### getFactory()

```
private AppComponentFactory getFactory(String pkg) {
        if (pkg == null) {
            Log.e(TAG, "No pkg specified, disabling AppComponentFactory");
            return AppComponentFactory.DEFAULT;
        }
        if (mThread == null) {
            Log.e(TAG, "Uninitialized ActivityThread, likely app-created Instrumentation,"
                    + " disabling AppComponentFactory", new Throwable());
            return AppComponentFactory.DEFAULT;
        }
        LoadedApk apk = mThread.peekPackageInfo(pkg, true);
        // This is in the case of starting up "android".
        if (apk == null) apk = mThread.getSystemContext().mPackageInfo;
        return apk.getAppFactory();
    }
```

返回`AppComponentFactory`类型。

## `AppComponentFactory`

### `getAppFactory()`

```
public AppComponentFactory getAppFactory() {
        return mAppComponentFactory;
    }
```

### `instantiateActivity()`

```
public @NonNull Activity instantiateActivity(@NonNull ClassLoader cl, @NonNull String className,
            @Nullable Intent intent)
            throws InstantiationException, IllegalAccessException, ClassNotFoundException {
        return (Activity) cl.loadClass(className).newInstance();
    }
```

可以看到是通过反射创建出`Activity`

# 如何执行Activity#onCreate()

## `ActivityThread`

### `performLaunchActivity`

```
//调用onCreate方法
        if (r.isPersistable()) {
                    mInstrumentation.callActivityOnCreate(activity, r.state, r.persistentState);
                } else {
                    mInstrumentation.callActivityOnCreate(activity, r.state);
                }
```

## `Instrumentation`

### `callActivityOnCreate()`

```
public void callActivityOnCreate(Activity activity, Bundle icicle) {
        prePerformCreate(activity);
        activity.performCreate(icicle);
        postPerformCreate(activity);
    }
```

## `Activity`

### `performCreate`()

```
final void performCreate(Bundle icicle, PersistableBundle persistentState) {
        mCanEnterPictureInPicture = true;
        restoreHasCurrentPermissionRequest(icicle);
        if (persistentState != null) {
            onCreate(icicle, persistentState);
        } else {
            onCreate(icicle);
        }
        writeEventLog(LOG_AM_ON_CREATE_CALLED, "performCreate");
        mActivityTransitionState.readState(icicle);

        mVisibleFromClient = !mWindow.getWindowStyle().getBoolean(
                com.android.internal.R.styleable.Window_windowNoDisplay, false);
        mFragments.dispatchActivityCreated();
        mActivityTransitionState.setEnterActivityOptions(this, getActivityOptions());
    }
```

### onCreate()

```
public void onCreate(@Nullable Bundle savedInstanceState,
            @Nullable PersistableBundle persistentState) {
        onCreate(savedInstanceState);
    }
```

```
protected void onCreate(@Nullable Bundle savedInstanceState) {
        if (DEBUG_LIFECYCLE) Slog.v(TAG, "onCreate " + this + ": " + savedInstanceState);

        if (mLastNonConfigurationInstances != null) {
            mFragments.restoreLoaderNonConfig(mLastNonConfigurationInstances.loaders);
        }
        if (mActivityInfo.parentActivityName != null) {
            if (mActionBar == null) {
                mEnableDefaultActionBarUp = true;
            } else {
                mActionBar.setDefaultDisplayHomeAsUpEnabled(true);
            }
        }
        if (savedInstanceState != null) {
            mAutoFillResetNeeded = savedInstanceState.getBoolean(AUTOFILL_RESET_NEEDED, false);
            mLastAutofillId = savedInstanceState.getInt(LAST_AUTOFILL_ID,
                    View.LAST_APP_AUTOFILL_ID);

            if (mAutoFillResetNeeded) {
                getAutofillManager().onCreate(savedInstanceState);
            }

            Parcelable p = savedInstanceState.getParcelable(FRAGMENTS_TAG);
            mFragments.restoreAllState(p, mLastNonConfigurationInstances != null
                    ? mLastNonConfigurationInstances.fragments : null);
        }
        mFragments.dispatchCreate();
        getApplication().dispatchActivityCreated(this, savedInstanceState);
        if (mVoiceInteractor != null) {
            mVoiceInteractor.attachActivity(this);
        }
        mRestoredFromBundle = savedInstanceState != null;
        mCalled = true;
    }
```

### TestSamplePictureActivity

```
@Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
      }
```

到此一个Activity的开启已经分析完毕，很麻烦。这篇文章只是记录Activity的启动流程，中间好多类都没有分析到位，比如

- ActivityManagerService 知识流程经过
- Instrumentation类
- 启动模式，任务栈相关
- 等等

Activity的启动流程很少去看c++的代码，在AndroidStudio上就可以完成阅读，前提需要理解Binder机制。

# 总结

![start_activity_process](http://gityuan.com/images/activity/start_activity_process.jpg)