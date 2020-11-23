---
title: startActivity源码分析(1)
date: 2020-05-19 15:17:42
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

# 相关源码

```
frameworks/base/services/core/java/com/android/server/am/
  - ActivityManagerService.java
  - ActivityStackSupervisor.java
  - ActivityStack.java
  - ActivityRecord.java
  - ProcessRecord.java

frameworks/base/core/java/android/app/
  - IActivityManager.java
  - ActivityManagerNative.java (内含AMP)
  - ActivityManager.java

  - IApplicationThread.java
  - ApplicationThreadNative.java (内含ATP)
  - ActivityThread.java (内含ApplicationThread)

  - ContextImpl.java
```

## 源码地址

[https://android.googlesource.com/platform/frameworks/base/+/refs/tags/android-vts-9.0_r13/core/java/android/app/IApplicationThread.aidl](https://android.googlesource.com/platform/frameworks/base/+/refs/tags/android-vts-9.0_r13/core/java/android/app/IApplicationThread.aidl)

# 总结

Activity启动发起后，通过Binder最终交由system进程中的AMS来完成，则启动流程如下图：

![start_activity](http://gityuan.com/images/activity/start_activity.jpg)

![start_activity_process](http://gityuan.com/images/activity/start_activity_process.jpg)

启动流程：

1. 点击桌面App图标，Launcher进程采用Binder IPC向system_server进程发起startActivity请求；
2. system_server进程接收到请求后，向zygote进程发送创建进程的请求；
3. Zygote进程fork出新的子进程，即App进程；
4. App进程，通过Binder IPC向sytem_server进程发起attachApplication请求；
5. system_server进程在收到请求后，进行一系列准备工作后，再通过binder IPC向App进程发送scheduleLaunchActivity请求；
6. App进程的binder线程（ApplicationThread）在收到请求后，通过handler向主线程发送LAUNCH_ACTIVITY消息；
7. 主线程在收到Message后，通过发射机制创建目标Activity，并回调Activity.onCreate()等方法。

到此，App便正式启动，开始进入Activity生命周期，执行完onCreate/onStart/onResume方法，UI渲染结束后便可以看到App的主界面。 启动Activity较为复杂，后续计划再进一步讲解生命周期过程与系统是如何交互，以及UI渲染过程，敬请期待。

# 开始分析

## `MainActivity`

发起页面请求

```
@Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        findViewById(R.id.btn_pic).setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                startActivity(new Intent(MainActivity.this, TestSamplePictureActivity.class));
            }
        });
    }
```

## `Activity`

### `startActivity()`

```
@Override
    public void startActivity(Intent intent) {
        this.startActivity(intent, null);
    }
    
    public void startActivity(Intent intent, @Nullable Bundle options) {
        if (options != null) {
            startActivityForResult(intent, -1, options);
        } else {           
            startActivityForResult(intent, -1);
        }
    }
```

### `startActivityForResult()`

```
public void startActivityForResult(@RequiresPermission Intent intent, int requestCode) {
        startActivityForResult(intent, requestCode, null);
    }
    
    public void startActivityForResult(@RequiresPermission Intent intent, int requestCode,
            @Nullable Bundle options) {
            ...
            Instrumentation.ActivityResult ar =
                mInstrumentation.execStartActivity(
                    this, mMainThread.getApplicationThread(), mToken, this,
                    intent, requestCode, options);
          	...
    }
```

`execStartActivity(Context who, IBinder contextThread, IBinder token, Activity target,Intent intent, int requestCode, Bundle options)`

**参数：**

- context who:当前Activity
- IBinder contextThread：`ActivityThread`中的`ApplicationThread`
- IBinder token：attach()中赋值的IBinder
- Activity target:当前Activity
- Intent intent：自己传递的intent
- int requestCode：返回码
-  Bundle options：传null也会有一些附加信息`transferSpringboardActivityOptions`、

在返回`ActivityResult ar`后就过`mMainThread.sendActivityResult`方法返回到`ActivityThread`中。**（后续分析）**

## `Instrumentation`

每一个应用程序只有一个Instrumentation对象，每个Activity内都有一个对该对象的引用。Instrumentation可以理解为应用进程的管家，ActivityThread要创建或暂停某个Activity时，都需要通过Instrumentation来进行具体的操作。

### `execStartActivity()`

```
/*
执行应用程序发出的startActivity调用。 默认实现负责更新任何活动的{@link ActivityMonitor}对象，并将此调用分派给系统活动管理器。 您可以覆盖它以监视应用程序启动活动，并修改执行该操作时发生的情况。

此方法返回一个{@link ActivityResult}对象，可以在拦截应用程序调用时使用该对象以避免执行start activity操作，但仍返回应用程序期望的结果。 为此，请重写此方法以捕获对启动活动的调用，以便它返回一个新的ActivityResult，其中包含您希望应用程序看到的结果，而不调用超级类。 请注意，如果requestCode 0，则应用程序仅期望结果

如果未找到用于运行给定Intent的Activity，则此方法将引发{@link android.content.ActivityNotFoundException}。
@param who从中开始活动的上下文。
@param contextThread从其开始活动的Context的主线程。
@param token 内部令牌，标识正在启动活动的系统； 可以为null。
@param target哪个活动正在执行启动（并因此接收到任何结果）； 如果不是通过活动进行此调用，则可以为null。
@param intent实际启动的Intent。
@param requestCode该请求结果的标识符； 如果呼叫者不期望结果，则小于零。
@param options附加选项。
@return要强制返回特定结果，请返回包含所需数据的ActivityResult对象。 否则返回null。 默认实现始终返回null。
*/
public ActivityResult execStartActivity(
            Context who, IBinder contextThread, IBinder token, Activity target,
            Intent intent, int requestCode, Bundle options) {
            ...
        try {
            intent.migrateExtraStreamToClipData();
            intent.prepareToLeaveProcess(who);
            //通过am去执行startActivity
            int result = ActivityManager.getService()
                .startActivity(whoThread, who.getBasePackageName(), intent,
                        intent.resolveTypeIfNeeded(who.getContentResolver()),
                        token, target != null ? target.mEmbeddedID : null,
                        requestCode, 0, null, options);
            //根据result结果判断抛出异常
            checkStartActivityResult(result, intent);
        } catch (RemoteException e) {
            throw new RuntimeException("Failure from system", e);
        }
        return null;
    }
```

## `ActivityManager`

### `getService()`

```
public static IActivityManager getService() {
        return IActivityManagerSingleton.get();
    }

    private static final Singleton<IActivityManager> IActivityManagerSingleton =
            new Singleton<IActivityManager>() {
                @Override
                protected IActivityManager create() {
                    //1. 获取服务的Binder对象
                    final IBinder b = ServiceManager.getService(Context.ACTIVITY_SERVICE);
                     //2. aidl 获取AMS
                    final IActivityManager am = IActivityManager.Stub.asInterface(b);
                    return am;
                }
            };
```

ServiceManager是安卓中一个重要的类，用于管理所有的系统服务，维护着系统服务和客户端的binder通信。返回的是Binder对象,用来进行应用与系统服务之间的通信的.

## `IActivityManager`

```
//Aidl接口
public interface IActivityManager extends IInterface {
}
```

### `startActivity()`

```
public int startActivity(IApplicationThread caller, String callingPackage, Intent intent,
                             String resolvedType, IBinder resultTo, String resultWho, int requestCode, int flags,
                             ProfilerInfo profilerInfo, Bundle options) throws RemoteException;
```

IActivityManager是Aidl的客户端，通过客户端去调用服务端的`startActivity`的实现方法。我们要找到`IActivityManager`的具体实现类，在找到他的代理就可以查看它的源码了。

## `ActivityManagerService`

**此时进入system_server进程**

```
public class ActivityManagerService extends IActivityManager.Stub
        implements Watchdog.Monitor, BatteryStatsImpl.BatteryCallback {
        }
```

`ActivityManagerService`就是`IActivityManager`的具体实现类（如果不懂的话自己先写aidl完成通讯，在查看系统生成的实现类就理解了）

### `startActivity()`

```
@Override
    public final int startActivity(IApplicationThread caller, String callingPackage,
            Intent intent, String resolvedType, IBinder resultTo, String resultWho, int requestCode,
            int startFlags, ProfilerInfo profilerInfo, Bundle bOptions) {
        return startActivityAsUser(caller, callingPackage, intent, resolvedType, resultTo,
                resultWho, requestCode, startFlags, profilerInfo, bOptions,
                UserHandle.getCallingUserId());
    }
    
```

### `startActivityAsUser()`

```
 public final int startActivityAsUser(IApplicationThread caller, String callingPackage,
            Intent intent, String resolvedType, IBinder resultTo, String resultWho, int requestCode,
            int startFlags, ProfilerInfo profilerInfo, Bundle bOptions, int userId,
            boolean validateIncomingUser) {
        enforceNotIsolatedCaller("startActivity");

        userId = mActivityStartController.checkTargetUser(userId, validateIncomingUser,
                Binder.getCallingPid(), Binder.getCallingUid(), "startActivityAsUser");

        // TODO: Switch to user app stacks here.
        //在此处切换到用户应用堆栈。
        return mActivityStartController.obtainStarter(intent, "startActivityAsUser")
                .setCaller(caller)
                .setCallingPackage(callingPackage)
                .setResolvedType(resolvedType)
                .setResultTo(resultTo)
                .setResultWho(resultWho)
                .setRequestCode(requestCode)
                .setStartFlags(startFlags)
                .setProfilerInfo(profilerInfo)
                .setActivityOptions(bOptions)
                .setMayWait(userId)
                .execute();

    }
```



AMS的startActivity方法会调用AMS的startActivityAsUser方法,然后又调用另一个startActivityAsUser方法.最后来了一串链式调用设置信息,最后会来到ActivityStarter的execute方法.

**obtainStarter**返回`ActivityStarter`对象。

## `ActivityStartController`

### `obtainStarter()`

```
    /**
     * @return A starter to configure and execute starting an activity. It is valid until after
     *         {@link ActivityStarter#execute} is invoked. At that point, the starter should be
     *         considered invalid and no longer modified or used.
     */
    ActivityStarter obtainStarter(Intent intent, String reason) {
        return mFactory.obtain().setIntent(intent).setReason(reason);
    }
```

mFactory.obtain()从对象池中获取一个`ActivityStarter`对象。

## `ActivityStarter`

控制器，用于解释如何启动活动。
此类收集了用于确定将意图和标志如何转换为活动以及相关联的任务和堆栈的所有逻辑。

它是加载Activity的控制类，会收集所有的逻辑来决定如何将Intent和Flags转换为Activity，并将Activity和Task以及Stack相关联。

#### execute

```
/**
     * Starts an activity based on the request parameters provided earlier.
     * @return The starter result.
     */
    int execute() {
        try {
            // TODO(b/64750076): Look into passing request directly to these methods to allow
            // for transactional diffs and preprocessing.
            //TODO（b / 64750076）：研究将请求直接传递给这些方法，以允许事务性差异和预处理。
            if (mRequest.mayWait) {
                return startActivityMayWait(mRequest.caller, mRequest.callingUid,
                        mRequest.callingPackage, mRequest.intent, mRequest.resolvedType,
                        mRequest.voiceSession, mRequest.voiceInteractor, mRequest.resultTo,
                        mRequest.resultWho, mRequest.requestCode, mRequest.startFlags,
                        mRequest.profilerInfo, mRequest.waitResult, mRequest.globalConfig,
                        mRequest.activityOptions, mRequest.ignoreTargetSecurity, mRequest.userId,
                        mRequest.inTask, mRequest.reason,
                        mRequest.allowPendingRemoteAnimationRegistryLookup);
            } else {
                return startActivity(mRequest.caller, mRequest.intent, mRequest.ephemeralIntent,
                        mRequest.resolvedType, mRequest.activityInfo, mRequest.resolveInfo,
                        mRequest.voiceSession, mRequest.voiceInteractor, mRequest.resultTo,
                        mRequest.resultWho, mRequest.requestCode, mRequest.callingPid,
                        mRequest.callingUid, mRequest.callingPackage, mRequest.realCallingPid,
                        mRequest.realCallingUid, mRequest.startFlags, mRequest.activityOptions,
                        mRequest.ignoreTargetSecurity, mRequest.componentSpecified,
                        mRequest.outActivity, mRequest.inTask, mRequest.reason,
                        mRequest.allowPendingRemoteAnimationRegistryLookup);
            }
        } finally {
            onExecutionComplete();
        }
    }
```

根据之前提供的请求参数启动活动。mayWait默认true表示我们应该等待启动请求的结果。

不管如何都不执行startActivity方法。

### startActivityMayWait

这个方法很长很长,还看不懂，最后还是执行`startActivity()`

```
private int startActivityMayWait(IApplicationThread caller, int callingUid,
            String callingPackage, Intent intent, String resolvedType,
            IVoiceInteractionSession voiceSession, IVoiceInteractor voiceInteractor,
            IBinder resultTo, String resultWho, int requestCode, int startFlags,
            ProfilerInfo profilerInfo, WaitResult outResult,
            Configuration globalConfig, SafeActivityOptions options, boolean ignoreTargetSecurity,
            int userId, TaskRecord inTask, String reason,
            boolean allowPendingRemoteAnimationRegistryLookup) {
        // Refuse possible leaked file descriptors
        //拒绝可能的泄漏文件描述符
        if (intent != null && intent.hasFileDescriptors()) {
            throw new IllegalArgumentException("File descriptors passed in Intent");
        }
        //当我们开始启动活动时，尽早通知跟踪器。应该是处理日志的。
        mSupervisor.getActivityMetricsLogger().notifyActivityLaunching();
        //处理意图的应用程序组件的名称。
        boolean componentSpecified = intent.getComponent() != null;
				//返回向您发送当前正在处理的事务的进程的ID。 该pid可以与更高级别的系统服务一起使用，以确定其身份并检查权限。 如果当前线程当前未在执行传入事务，则返回其自己的pid。			
        final int realCallingPid = Binder.getCallingPid();
        //Uid
        final int realCallingUid = Binder.getCallingUid();

       ...

        // Save a copy in case ephemeral needs it
        //保存副本以防临时需要
        final Intent ephemeralIntent = new Intent(intent);
        // Don't modify the client's object!
        intent = new Intent(intent);        
        ...                        
						//历史记录堆栈中的一个实体，代表一项活动。
            final ActivityRecord[] outRecord = new ActivityRecord[1];
            int res = startActivity(caller, intent, ephemeralIntent, resolvedType, aInfo, rInfo,
                    voiceSession, voiceInteractor, resultTo, resultWho, requestCode, callingPid,
                    callingUid, callingPackage, realCallingPid, realCallingUid, startFlags, options,
                    ignoreTargetSecurity, componentSpecified, outRecord, inTask, reason,
                    allowPendingRemoteAnimationRegistryLookup);
						//将当前线程上的传入IPC的身份还原回{@link #clearCallingIdentity}返回的先前身份。
            Binder.restoreCallingIdentity(origId);
           ...
    }
```

还是走下面的方法

### startActivity()

```
private int startActivity(IApplicationThread caller, Intent intent, Intent ephemeralIntent,
            String resolvedType, ActivityInfo aInfo, ResolveInfo rInfo,
            IVoiceInteractionSession voiceSession, IVoiceInteractor voiceInteractor,
            IBinder resultTo, String resultWho, int requestCode, int callingPid, int callingUid,
            String callingPackage, int realCallingPid, int realCallingUid, int startFlags,
            SafeActivityOptions options, boolean ignoreTargetSecurity, boolean componentSpecified,
            ActivityRecord[] outActivity, TaskRecord inTask, String reason,
            boolean allowPendingRemoteAnimationRegistryLookup) {

        if (TextUtils.isEmpty(reason)) {
            throw new IllegalArgumentException("Need to specify a reason.");
        }
        mLastStartReason = reason;
        mLastStartActivityTimeMs = System.currentTimeMillis();
        mLastStartActivityRecord[0] = null;

        mLastStartActivityResult = startActivity(caller, intent, ephemeralIntent, resolvedType,
                aInfo, rInfo, voiceSession, voiceInteractor, resultTo, resultWho, requestCode,
                callingPid, callingUid, callingPackage, realCallingPid, realCallingUid, startFlags,
                options, ignoreTargetSecurity, componentSpecified, mLastStartActivityRecord,
                inTask, allowPendingRemoteAnimationRegistryLookup);

        if (outActivity != null) {
            // mLastStartActivityRecord[0] is set in the call to startActivity above.
            outActivity[0] = mLastStartActivityRecord[0];
        }

        return getExternalResult(mLastStartActivityResult);
    }
```

同样执行的还是很长很长的startActivity()方法；

```
private int startActivity(IApplicationThread caller, Intent intent, Intent ephemeralIntent,
            String resolvedType, ActivityInfo aInfo, ResolveInfo rInfo,
            IVoiceInteractionSession voiceSession, IVoiceInteractor voiceInteractor,
            IBinder resultTo, String resultWho, int requestCode, int callingPid, int callingUid,
            String callingPackage, int realCallingPid, int realCallingUid, int startFlags,
            SafeActivityOptions options,
            boolean ignoreTargetSecurity, boolean componentSpecified, ActivityRecord[] outActivity,
            TaskRecord inTask, boolean allowPendingRemoteAnimationRegistryLookup) {
      	...
        return startActivity(r, sourceRecord, voiceSession, voiceInteractor, startFlags,
                true /* doResume */, checkedOptions, inTask, outActivity);
    }
```

上边的代码就是做一些规则判断，是否符合开启Activity的条件，不符合就直接返回错误码，**还记得`Instrumentation`的`checkStartActivityResult（）`么，就是根据这里返回的结果抛出异常的比如：`ActivityNotFoundException`**最后还是执行startActivity方法

```
private int startActivity(final ActivityRecord r, ActivityRecord sourceRecord,
                IVoiceInteractionSession voiceSession, IVoiceInteractor voiceInteractor,
                int startFlags, boolean doResume, ActivityOptions options, TaskRecord inTask,
                ActivityRecord[] outActivity) {
        int result = START_CANCELED;
        try {
        //开始推迟布局传递。 在进行多项更改时很有用，但为了优化性能，仅应执行一次布局遍历。 可以多次调用此方法，并且在最后一个调用者致电{@link continueSurfaceLayout}后，将恢复布局
            mService.mWindowManager.deferSurfaceLayout();
            //注意：只能从{@link startActivity}调用此方法。
            result = startActivityUnchecked(r, sourceRecord, voiceSession, voiceInteractor,
                    startFlags, doResume, options, inTask, outActivity);
        } finally {
            // If we are not able to proceed, disassociate the activity from the task. Leaving an
            // activity in an incomplete state can lead to issues, such as performing operations
            // without a window container.
            //如果无法继续，请取消将活动与任务关联。 
            //将活动置于不完整状态可能会导致问题，例如在没有窗口容器的情况下执行操作。
            
            //当前任务的堆栈值，如果没有任务，则返回null。
            final ActivityStack stack = mStartActivity.getStack();
            //返回启动是否成功。
            if (!ActivityManager.isStartResultSuccessful(result) && stack != null) {
            //如果此活动已从历史记录列表中删除，则返回true；如果仍在列表中并将稍后删除，则返回false。
                stack.finishActivityLocked(mStartActivity, RESULT_CANCELED,
                        null /* intentResultData */, "startActivity", true /* oomAdj */);
            }
            //延后布局恢复通过。 参见{@link deferSurfaceLayout（）}
            mService.mWindowManager.continueSurfaceLayout();
        }

        postStartActivityProcessing(r, result, mTargetStack);

        return result;
    }
```

主要执行了`startActivityUnchecked`方法又是一个很长很长的方法，脑袋疼。

之后执行了`postStartActivityProcessing`,这个方法内部有一个描述

> //我们正在等待活动启动完成，但是该活动只是将另一个活动带到了前面。 我们还必须处理由于蹦床活动处于同一任务而导致任务已经在最前面的情况（由于蹦床将要完成，因此将被视为重点关注）。 让startActivityMayWait（）知道这一点，因此它等待新活动变为可见。
>
> 该活动已经在运行，因此尚未启动，但是由于它已经在最前面，所以要么被置于最前面，要么被传递给它新的意图。 通知对此信息感兴趣的任何人。

### `startActivityUnchecked()`

```
// Note: This method should only be called from {@link startActivity}.
    private int startActivityUnchecked(final ActivityRecord r, ActivityRecord sourceRecord,
            IVoiceInteractionSession voiceSession, IVoiceInteractor voiceInteractor,
            int startFlags, boolean doResume, ActivityOptions options, TaskRecord inTask,
            ActivityRecord[] outActivity) {
            ...
				mSupervisor.resumeFocusedStackTopActivityLocked(mTargetStack, mStartActivity,
                        mOptions);
				...
        return START_SUCCESS;
    }
```

## `ActivityStackSupervisor`

ActivityStack是一个管理类，用来管理系统所有Activity的各种状态，其内部维护了TaskRecord的列表，因此从Activity任务栈这一角度来说，ActivityStack也可以理解为Activity堆栈。它由ActivityStackSupervisor来进行管理的，而ActivityStackSupervisor在AMS中的构造方法中被创建。ActivityStackSupervisor中有多种ActivityStack实例。

```
public ActivityManagerService(Context systemContext) {
...
 mStackSupervisor = new ActivityStackSupervisor(this);
... 
}
```

```
public final class ActivityStackSupervisor implements DisplayListener {
   ...
    ActivityStack mHomeStack;
    ActivityStack mFocusedStack;
    private ActivityStack mLastFocusedStack;
    ...
}
```

- **ActivityRecord**：存在历史栈的一个实例，代表一个Activity。
- **TaskRecord**：Activity栈，内部维护一个ArrayList<ActivityRecord>
- **ActivityStack**：并不是一个Activity栈，真正意义上的Activity栈是TaskRecord，这个类是负责管理各个Activity栈，内部维护一个ArrayList<TaskRecord>
- **ActivityStackSupervisor**：内部持有一个ActivityStack，而ActivityStack内部也持有ActivityStackSupervisor，相当于ActivityStack的辅助管理类

### `resumeFocusedStackTopActivityLocked()`

```
boolean resumeFocusedStackTopActivityLocked(
            ActivityStack targetStack, ActivityRecord target, ActivityOptions targetOptions) {

        if (!readyToResume()) {
            return false;
        }

        if (targetStack != null && isFocusedStack(targetStack)) {
            return targetStack.resumeTopActivityUncheckedLocked(target, targetOptions);
        }

        final ActivityRecord r = mFocusedStack.topRunningActivityLocked();
        if (r == null || !r.isState(RESUMED)) {
            mFocusedStack.resumeTopActivityUncheckedLocked(null, null);
        } else if (r.isState(RESUMED)) {
            // Kick off any lingering app transitions form the MoveTaskToFront operation.
            mFocusedStack.executeAppTransition(targetOptions);
        }

        return false;
    }
```

## `ActivityStack`

![image.png](https://upload-images.jianshu.io/upload_images/4118241-35ca17cdd514d7cb.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### resumeTopActivityUncheckedLocked()

确保在栈顶部活动恢复。

```
    /**
     * Ensure that the top activity in the stack is resumed.
     *
     * @param prev The previously resumed activity, for when in the process
     * of pausing; can be null to call from elsewhere.
     * @param options Activity options.
     *
     * @return Returns true if something is being resumed, or false if
     * nothing happened.
     *
     * NOTE: It is not safe to call this method directly as it can cause an activity in a
     *       non-focused stack to be resumed.
     *       Use {@link ActivityStackSupervisor#resumeFocusedStackTopActivityLocked} to resume the
     *       right activity for the current system state.
     */
    @GuardedBy("mService")
    boolean resumeTopActivityUncheckedLocked(ActivityRecord prev, ActivityOptions options) {
        if (mStackSupervisor.inResumeTopActivity) {
            // Don't even start recursing.
            return false;
        }

        boolean result = false;
        try {
            // Protect against recursion.
            //防止递归。
            mStackSupervisor.inResumeTopActivity = true;
            result = resumeTopActivityInnerLocked(prev, options);

            // When resuming the top activity, it may be necessary to pause the top activity (for
            // example, returning to the lock screen. We suppress the normal pause logic in
            // {@link #resumeTopActivityUncheckedLocked}, since the top activity is resumed at the
            // end. We call the {@link ActivityStackSupervisor#checkReadyForSleepLocked} again here
            // to ensure any necessary pause logic occurs. In the case where the Activity will be
            // shown regardless of the lock screen, the call to
            // {@link ActivityStackSupervisor#checkReadyForSleepLocked} is skipped.
            ////恢复顶层活动时，可能有必要暂停顶层活动（例如，返回到锁定屏幕。由于顶层活动会在最后恢复，因此我们在{@link #resumeTopActivityUncheckedLocked}中取消了正常的暂停逻辑 为了确保发生任何必要的暂停逻辑，我们在此处再次调用{@link ActivityStackSupervisor＃checkReadyForSleepLocked}。
            final ActivityRecord next = topRunningActivityLocked(true /* focusableOnly */);
            if (next == null || !next.canTurnScreenOn()) {
                checkReadyForSleep();
            }
        } finally {
            mStackSupervisor.inResumeTopActivity = false;
        }

        return result;
    }

```

### `resumeTopActivityInnerLocked()`

    @GuardedBy("mService")
    private boolean resumeTopActivityInnerLocked(ActivityRecord prev, ActivityOptions options) {
        if (!mService.mBooting && !mService.mBooted) {
            // Not ready yet!
            //还没有准备好！
            return false;
        }
    
     		 ...
     		 
    		//堆栈中没有任何活动，让我们看看其他地方。
        if (!hasRunningActivity) {
            // There are no activities left in the stack, let's look somewhere else.
            return resumeTopActivityInNextFocusableStack(prev, options, "noMoreActivities");
        }
    		...       
        mStackSupervisor.startSpecificActivityLocked(next, true, true);
        return true;
    }
## `ActivityStackSupervisor`

### `startSpecificActivityLocked()`

```
void startSpecificActivityLocked(ActivityRecord r,
            boolean andResume, boolean checkConfig) {
        // Is this activity's application already running?
        //此活动的应用程序已经在运行吗？
        //获取应用进程信息
        ProcessRecord app = mService.getProcessRecordLocked(r.processName,
                r.info.applicationInfo.uid, true);

        getLaunchTimeTracker().setLaunchTime(r);
				//进程存在则启动
        if (app != null && app.thread != null) {
            try {
                if ((r.info.flags&ActivityInfo.FLAG_MULTIPROCESS) == 0
                        || !"android".equals(r.info.packageName)) {
                    // Don't add this if it is a platform component that is marked
                    // to run in multiple processes, because this is actually
                    // part of the framework so doesn't make sense to track as a
                    // separate apk in the process.
                    //如果它是标记为可以在多个进程中运行的平台组件，则不要添加它，因为它实际上是框架的一部分，因此在进程中作为单独的apk进行跟踪没有意义。
                    app.addPackage(r.info.packageName, r.info.applicationInfo.longVersionCode,
                            mService.mProcessStats);
                }
                realStartActivityLocked(r, app, andResume, checkConfig);
                return;
            } catch (RemoteException e) {
                Slog.w(TAG, "Exception when starting activity "
                        + r.intent.getComponent().flattenToShortString(), e);
            }

            // If a dead object exception was thrown -- fall through to
            // restart the application.
            ////如果抛出了死对象异常-请重新启动应用程序。
        }
				    //进程不存在则创建
        mService.startProcessLocked(r.processName, r.info.applicationInfo, true, 0,
                "activity", r.intent.getComponent(), false, false, true);
    }
```

如果进程存在调用`realStartActivityLocked`直接开起activity;如果没有进程则AMS创建一个进程`mService.startProcessLocked`

## `ActivityManagerService`

AMS(ActivityManagerService)是贯穿Android系统组件的核心服务，负责了系统中四大组件的启动、切换、调度以及应用进程管理和调度工作。

### `startProcessLocked()`

```
    @GuardedBy("this")
    final ProcessRecord startProcessLocked(String processName,
            ApplicationInfo info, boolean knownToBeDead, int intentFlags,
            String hostingType, ComponentName hostingName, boolean allowWhileBooting,
            boolean isolated, boolean keepIfLarge) {
        return startProcessLocked(processName, info, knownToBeDead, intentFlags, hostingType,
                hostingName, allowWhileBooting, isolated, 0 /* isolatedUid */, keepIfLarge,
                null /* ABI override */, null /* entryPoint */, null /* entryPointArgs */,
                null /* crashHandler */);
    }
```

规则校验的代码太多了此类会经过一系列的startxxx方法

```
final ProcessRecord startProcessLocked(String processName, ApplicationInfo info,
            boolean knownToBeDead, int intentFlags, String hostingType, ComponentName hostingName,
            boolean allowWhileBooting, boolean isolated, int isolatedUid, boolean keepIfLarge,
            String abiOverride, String entryPoint, String[] entryPointArgs, Runnable crashHandler){
            ...
            checkTime(startTime, "startProcess: stepping in to startProcess");
       		 final boolean success = startProcessLocked(app, hostingType, hostingNameStr, abiOverride);
       		 checkTime(startTime, "startProcess: done starting proc!");
      		  ...
            }
```

```
private final boolean startProcessLocked(ProcessRecord app, String hostingType,
            String hostingNameStr, boolean disableHiddenApiChecks, String abiOverride) {
						...
						return startProcessLocked(hostingType, hostingNameStr, entryPoint, app, uid, gids,
                    runtimeFlags, mountExternal, seInfo, requiredAbi, instructionSet, invokeWith,
                    startTime);
            ...
}
```

```
private boolean startProcessLocked(String hostingType, String hostingNameStr, String entryPoint,
            ProcessRecord app, int uid, int[] gids, int runtimeFlags, int mountExternal,
            String seInfo, String requiredAbi, String instructionSet, String invokeWith,
            long startTime) {
            	...
            	final ProcessStartResult startResult = startProcess(hostingType, entryPoint, app,
                        uid, gids, runtimeFlags, mountExternal, seInfo, requiredAbi, instructionSet,
                        invokeWith, startTime);
                handleProcessStartedLocked(app, startResult.pid, startResult.usingWrapper,
                        startSeq, false);
            	...
            }
```

### `startProcess()`

```
private ProcessStartResult startProcess(String hostingType, String entryPoint,
            ProcessRecord app, int uid, int[] gids, int runtimeFlags, int mountExternal,
            String seInfo, String requiredAbi, String instructionSet, String invokeWith,
            long startTime) {
            ...
            if (hostingType.equals("webview_service")) {
                startResult = startWebView(entryPoint,
                        app.processName, uid, uid, gids, runtimeFlags, mountExternal,
                        app.info.targetSdkVersion, seInfo, requiredAbi, instructionSet,
                        app.info.dataDir, null,
                        new String[] {PROC_START_SEQ_IDENT + app.startSeq});
            } else {
                startResult = Process.start(entryPoint,
                        app.processName, uid, uid, gids, runtimeFlags, mountExternal,
                        app.info.targetSdkVersion, seInfo, requiredAbi, instructionSet,
                        app.info.dataDir, invokeWith,
                        new String[] {PROC_START_SEQ_IDENT + app.startSeq});
            }
            ...
            }
```

AMS最后会调用Process类中的start方法

## `Process`

根据类的解释`Process`：用于管理操作系统进程的工具。

### `start()`

```
public static final ProcessStartResult start(final String processClass,
                                  final String niceName,
                                  int uid, int gid, int[] gids,
                                  int runtimeFlags, int mountExternal,
                                  int targetSdkVersion,
                                  String seInfo,
                                  String abi,
                                  String instructionSet,
                                  String appDataDir,
                                  String invokeWith,
                                  String[] zygoteArgs) {
        return zygoteProcess.start(processClass, niceName, uid, gid, gids,
                    runtimeFlags, mountExternal, targetSdkVersion, seInfo,
                    abi, instructionSet, appDataDir, invokeWith, zygoteArgs);
    }
```

```
/ **
 开始一个新的进程。
 
 <p>如果启用了进程，则会创建一个新进程，并在其中执行<var> processClass </ var>的静态main（）函数。该函数返回后，该过程将继续运行。
     *
     * <p>如果进程可用的，则会在调用者的进程中创建一个新进程，并在其中调用<var> processClass </ var>的main（）。
     *
     * <p> niceName参数（如果不是一个空字符串）是一个自定义名称，该名称将赋予进程而不是使用processClass。即使您使用相同的基本<var> processClass </ var>启动它们，这也使您可以轻松识别进程。
     *
     *当invokeWith不为null时，该过程将作为一个新鲜的应用而不是合子叉启动。请注意，仅在uid 0或runtimeFlags包含DEBUG_ENABLE_DEBUGGER时才允许这样做。
     *
     * @param processClass用作流程的主要入口点的类。
     * @param niceName用于该过程的更易读的名称。
     * @param uid进程将在其下运行的用户ID。
     * @param gid将在其下运行进程的组ID。
     * @param gids与该进程关联的其他组ID。
     * @param runtimeFlags运行时的其他标志。
     * @param targetSdkVersion应用程序的目标SDK版本。
     * @param seInfo为新进程提供空的SELinux信息。
     * @param abi非空，此应用程序应以ABI开头。
     * @paramstructionSet为null，确定要使用的指令集。
     * @param appDataDir空-确定应用程序的数据目录。
     * @param invokeWith null-确定要调用的命令。
     * @param zygoteArgs提供给zygote进程的其他参数。
     *
     * @return一个对象，描述尝试启动该过程的结果。
     * @致命启动失败时抛出RuntimeException
     * /
```

最后执行的是`ZygoteProcess`中start方法.

## `ZygoteProcess`

和孵化进程保持通讯状态。 此类负责和孵化进程打开套接字，并代表{@link android.os.Process}类启动进程。

### `start()`

```
public final Process.ProcessStartResult start(final String processClass,
                                                  final String niceName,
                                                  int uid, int gid, int[] gids,
                                                  int runtimeFlags, int mountExternal,
                                                  int targetSdkVersion,
                                                  String seInfo,
                                                  String abi,
                                                  String instructionSet,
                                                  String appDataDir,
                                                  String invokeWith,
                                                  String[] zygoteArgs) {
        try {
            return startViaZygote(processClass, niceName, uid, gid, gids,
                    runtimeFlags, mountExternal, targetSdkVersion, seInfo,
                    abi, instructionSet, appDataDir, invokeWith, false /* startChildZygote */,
                    zygoteArgs);
        } catch (ZygoteStartFailedEx ex) {
            Log.e(LOG_TAG,
                    "Starting VM process through Zygote failed");
            throw new RuntimeException(
                    "Starting VM process through Zygote failed", ex);
        }
    }
```

### `startViaZygote()`

```
    private Process.ProcessStartResult startViaZygote(final String processClass,
                                                      final String niceName,
                                                      final int uid, final int gid,
                                                      final int[] gids,
                                                      int runtimeFlags, int mountExternal,
                                                      int targetSdkVersion,
                                                      String seInfo,
                                                      String abi,
                                                      String instructionSet,
                                                      String appDataDir,
                                                      String invokeWith,
                                                      boolean startChildZygote,
                                                      String[] extraArgs)
                                                      throws ZygoteStartFailedEx {
         //Zygote进程Main方法中的参数                                             
        ArrayList<String> argsForZygote = new ArrayList<String>();

        // --runtime-args, --setuid=, --setgid=,
        // and --setgroups= must go first
        argsForZygote.add("--runtime-args");
        argsForZygote.add("--setuid=" + uid);
        argsForZygote.add("--setgid=" + gid);
        argsForZygote.add("--runtime-flags=" + runtimeFlags);
        if (mountExternal == Zygote.MOUNT_EXTERNAL_DEFAULT) {
            argsForZygote.add("--mount-external-default");
        } else if (mountExternal == Zygote.MOUNT_EXTERNAL_READ) {
            argsForZygote.add("--mount-external-read");
        } else if (mountExternal == Zygote.MOUNT_EXTERNAL_WRITE) {
            argsForZygote.add("--mount-external-write");
        }
        argsForZygote.add("--target-sdk-version=" + targetSdkVersion);

        // --setgroups is a comma-separated list
        if (gids != null && gids.length > 0) {
            StringBuilder sb = new StringBuilder();
            sb.append("--setgroups=");

            int sz = gids.length;
            for (int i = 0; i < sz; i++) {
                if (i != 0) {
                    sb.append(',');
                }
                sb.append(gids[i]);
            }

            argsForZygote.add(sb.toString());
        }

        if (niceName != null) {
            argsForZygote.add("--nice-name=" + niceName);
        }

        ...

        argsForZygote.add(processClass);

        if (extraArgs != null) {
            for (String arg : extraArgs) {
                argsForZygote.add(arg);
            }
        }

        synchronized(mLock) {
            return zygoteSendArgsAndGetResult(openZygoteSocketIfNeeded(abi), argsForZygote);
        }
    }
```

通过孵化机制创建一个新的进程，这个方法还拼接了一些参数，在孵化进程中的main方法取出。

### `openZygoteSocketIfNeeded()`

```
    /**
     * Tries to open socket to Zygote process if not already open. If
     * already open, does nothing.  May block and retry.  Requires that mLock be held.
     */
    @GuardedBy("mLock")
    private ZygoteState openZygoteSocketIfNeeded(String abi) throws ZygoteStartFailedEx {
        Preconditions.checkState(Thread.holdsLock(mLock), "ZygoteProcess lock not held");

        if (primaryZygoteState == null || primaryZygoteState.isClosed()) {
            try {
                primaryZygoteState = ZygoteState.connect(mSocket);
            } catch (IOException ioe) {
                throw new ZygoteStartFailedEx("Error connecting to primary zygote", ioe);
            }
            maybeSetApiBlacklistExemptions(primaryZygoteState, false);
            maybeSetHiddenApiAccessLogSampleRate(primaryZygoteState);
        }
        if (primaryZygoteState.matches(abi)) {
            return primaryZygoteState;
        }

        // The primary zygote didn't match. Try the secondary.
        if (secondaryZygoteState == null || secondaryZygoteState.isClosed()) {
            try {
                secondaryZygoteState = ZygoteState.connect(mSecondarySocket);
            } catch (IOException ioe) {
                throw new ZygoteStartFailedEx("Error connecting to secondary zygote", ioe);
            }
            maybeSetApiBlacklistExemptions(secondaryZygoteState, false);
            maybeSetHiddenApiAccessLogSampleRate(secondaryZygoteState);
        }

        if (secondaryZygoteState.matches(abi)) {
            return secondaryZygoteState;
        }

        throw new ZygoteStartFailedEx("Unsupported zygote ABI: " + abi);
    }

```

如果尚未打开，则尝试打开套接字到Zygote进程。 如果已经打开，则不执行任何操作。 可能会阻止并重试。 要求保留mLock。

这个方法就是和孵化进程使用socker进行连接，如果连接了不执行操作。

现在socker已经连接上了孵化继承，就差通讯了。

### `zygoteSendArgsAndGetResult()`

```
    /**
     * Sends an argument list to the zygote process, which starts a new child
     * and returns the child's pid. Please note: the present implementation
     * replaces newlines in the argument list with spaces.
     *
     * @throws ZygoteStartFailedEx if process start failed for any reason
     */
    @GuardedBy("mLock")
    private static Process.ProcessStartResult zygoteSendArgsAndGetResult(
            ZygoteState zygoteState, ArrayList<String> args)
            throws ZygoteStartFailedEx {
        try {
            // Throw early if any of the arguments are malformed. This means we can
            // avoid writing a partial response to the zygote.
            int sz = args.size();
            for (int i = 0; i < sz; i++) {
                if (args.get(i).indexOf('\n') >= 0) {
                    throw new ZygoteStartFailedEx("embedded newlines not allowed");
                }
            }

            /**
             * See com.android.internal.os.SystemZygoteInit.readArgumentList()
             * Presently the wire format to the zygote process is:
             * a) a count of arguments (argc, in essence)
             * b) a number of newline-separated argument strings equal to count
             *
             * After the zygote process reads these it will write the pid of
             * the child or -1 on failure, followed by boolean to
             * indicate whether a wrapper process was used.
             */
            final BufferedWriter writer = zygoteState.writer;
            final DataInputStream inputStream = zygoteState.inputStream;

            writer.write(Integer.toString(args.size()));
            writer.newLine();

            for (int i = 0; i < sz; i++) {
                String arg = args.get(i);
                writer.write(arg);
                writer.newLine();
            }

            writer.flush();

            // Should there be a timeout on this?
            Process.ProcessStartResult result = new Process.ProcessStartResult();

            // Always read the entire result from the input stream to avoid leaving
            // bytes in the stream for future process starts to accidentally stumble
            // upon.
            result.pid = inputStream.readInt();
            result.usingWrapper = inputStream.readBoolean();

            if (result.pid < 0) {
                throw new ZygoteStartFailedEx("fork() failed");
            }
            return result;
        } catch (IOException ex) {
            zygoteState.close();
            throw new ZygoteStartFailedEx(ex);
        }
    }

```

将参数列表发送到Zygote进程，该进程将启动一个新的子进程并返回该子进程的pid。

此方法完成和Zygote进程socker通讯，阻塞等待socker信息返回，返回来的数据用ProcessStartResult包装。

接下来请看**【Android进程创建流程】**，经过层层代码，最后fork出一个新进程，利用反射进入`ActivityThread.main`

## `ActivityThread`

这可以管理应用程序进程中主线程的执行，调度和执行活动，广播以及根据活动管理器的请求对其执行的其他操作。

### main()

```
public static void main(String[] args) {

    
    Looper.prepareMainLooper();

    ......

    ActivityThread thread = new ActivityThread();
    thread.attach(false, startSeq);

    Looper.loop();

    throw new RuntimeException("Main thread loop unexpectedly exited");
}
```

（未完待续。。。）

请转移至**【ActivityThread开启Activity】**

