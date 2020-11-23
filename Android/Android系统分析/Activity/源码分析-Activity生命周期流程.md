---
title: 源码分析-Activity生命周期流程
date: 2020-06-02 20:41:53
tags: 
	- Android
	- Android系统分析
	- Activity生命周期
categories: 
	- Android
	- Android系统分析
	- Activity
---

# 概述

Activity 经典生命周期图：

![image.png](https://upload-images.jianshu.io/upload_images/4118241-9ceca5ca67e6e73f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![image.png](https://upload-images.jianshu.io/upload_images/4118241-cc817b0ebc885e28.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

不要在`onCreate，onStart，onResume，onPause`几个方法进行耗时操作，否则会造成页面切换卡顿。

## Activity的四种状态

Activity的生命周期中存在四种基本的状态：

- 活动状态(Active/Runing)
- 暂停状态(Paused)
- 停止状态(Stopped)
- 销毁状态(Killed)。

### 活动状态

一个新的Activity入栈后，当处于Activity栈顶时，此事它处于可见并且能与用户进行交互。

- 期间触发的生命周期：`onCreate()-> onStart()-> onResume()`

### 暂停状态

当Activity失去焦点，一个新的非全屏Activity或者透明的Activity被放置栈顶，此时处于暂停状态。一个处于暂定状态的Activity只有在系统极度匮乏的时候才会被回收掉。

在Activity之上显示dialog对话框或者`PopupWindow`该Activity并不会变成暂停状态。
当下拉通知栏时，Activity会走什么生命周期?以及弹出Dialog时会走什么生命周期?关于这两知识点，今天特意写了一下，发现啥都没走，也就说任何生命周期都不会走，也简单查了一下原因，下拉通知栏因为是系统窗口，所以不影响activity的生命周期，Dialog或者Toast源码中也能看windowmanager.addView()，也就说都是和系统管理者有关系，不会影响到本应用。

- 运行状态到暂停状态触发的生命周期：`onResume() --> onPause()`。
- 从暂停状态恢复到运行状态触发的生命周期：`onPause() -> onResume()`。

### 停止状态

一个Activity完全被另一个Activity完全覆盖或者，用户不可见或者退到后台，叫做停止状态。它仍保存着所有状态和成员信息，不能和用户交互，当系统内存需要的时候Stopped状态的Activity会被强制回收掉。

- 从暂停状态到停止状态经过的生命周期：`onPause()-->onStop()`
- 从停止状态到运行状态经过的生命周期：`onStop() --> onRestart() --> onStart() --> onResume()`。不会走onCreate()方法。

### 销毁状态

如果一个Activity再Paused或者Stopped状态，系统可以将Activity从内存中删除，删除的方式有两种：一种是要求Activity结束，一种直接结束掉它的进程。当有需要的时候就得重新创建Activity了。

触发的生命周期：onDestroy()，直接结束进程的话是不会走onDestroy()的。



# 源码分析

在开启Activity流程分析时说过，开启Activity会经过`ActivityStackSupervisor#realStartActivityLocked()`方法，然后经过一些列流程最终调用`onCreate`方法，其实其它生命周期也是类似的。

## `ActivityStackSupervisor`

### `realStartActivityLocked()`

```
final boolean realStartActivityLocked(ActivityRecord r, ProcessRecord app,
            boolean andResume, boolean checkConfig) throws RemoteException {
            ...
            // Create activity launch transaction.
                final ClientTransaction clientTransaction = ClientTransaction.obtain(app.thread,
                        r.appToken);
                 // 添加 LaunchActivityItem
                clientTransaction.addCallback(LaunchActivityItem.obtain(new Intent(r.intent),
                        System.identityHashCode(r), r.info,
                        // TODO: Have this take the merged configuration instead of separate global
                        // and override configs.
                        mergedConfiguration.getGlobalConfiguration(),
                        mergedConfiguration.getOverrideConfiguration(), r.compat,
                        r.launchedFromPackage, task.voiceInteractor, app.repProcState, r.icicle,
                        r.persistentState, results, newIntents, mService.isNextTransitionForward(),
                        profilerInfo));

                // Set desired final state.
                final ActivityLifecycleItem lifecycleItem;
                if (andResume) {
                    lifecycleItem = ResumeActivityItem.obtain(mService.isNextTransitionForward());
                } else {
                    lifecycleItem = PauseActivityItem.obtain();
                }
                // 2. 设置生命周期状态 这里是开启resume的关键，类型：ResumeActivityItem
                clientTransaction.setLifecycleStateRequest(lifecycleItem);

                // Schedule transaction.
                //执行生命周期
                mService.getLifecycleManager().scheduleTransaction(clientTransaction);
                ...
}            
```

首先还是看`mService.getLifecycleManager()`

## AMS中

### `getLifecycleManager()`

```
ClientLifecycleManager getLifecycleManager() {
        return mLifecycleManager;
    }
```

返回ClientLifecycleManager对象，然后调用`scheduleTransaction()`

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

调用`mClient`中`scheduleTransaction`方法，`mClient`是`IApplicationThread`,是`ActivityThread`内部类.

## `ActivityThread#IApplicationThread`

`IApplicationThread` 是极其重要的一个 Binder 接口，它维护了应用进程和 AMS 的之间的通讯。这个 `mClient` 就是应用进程在 AMS 进程中的代理对象，AMS 通过 `mClient` 来指挥应用进程。

### `scheduleTransaction()`

```
@Override
        public void scheduleTransaction(ClientTransaction transaction) throws RemoteException {
            ActivityThread.this.scheduleTransaction(transaction);
        }
```

调用`ActivityThread`中`scheduleTransaction`方法

## `ActivityThread`

### `scheduleTransaction()`

```
void scheduleTransaction(ClientTransaction transaction) {
        transaction.preExecute(this);
        sendMessage(ActivityThread.H.EXECUTE_TRANSACTION, transaction);
    }
```

线程间通讯，我们找到EXECUTE_TRANSACTION对应的实现。

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
```

## `TransactionExecutor`

### `execute()`

```
public void execute(ClientTransaction transaction) {
        final IBinder token = transaction.getActivityToken();
        log("Start resolving transaction for client: " + mTransactionHandler + ", token: " + token);
				//在这里执行onCreate()
        executeCallbacks(transaction);
				//在这里执行onResume onStart()
        executeLifecycleState(transaction);
        mPendingActions.clear();
        log("End resolving transaction");
    }
```

其实在之前的文章已经讲过onCreate是如何执行的了，这里就当预习了,`executeCallbacks`就不说了,最后执行`onCreate`,其实流程是差不多的

### `executeLifecycleState()`

```
private void executeLifecycleState(ClientTransaction transaction) {
        final ActivityLifecycleItem lifecycleItem = transaction.getLifecycleStateRequest();
        ...
        //执行的是onStart
        cycleToPath(r, lifecycleItem.getTargetState(), true /* excludeLastState */);

        // Execute the final transition with proper parameters.
        //onResume
        lifecycleItem.execute(mTransactionHandler, token, mPendingActions);
        lifecycleItem.postExecute(mTransactionHandler, token, mPendingActions);
    }
```

`transaction.getLifecycleStateRequest()`是在前头传入的`ResumeActivityItem`，我们直接看`execute`方法，`cycleToPath`稍后在说

## `ResumeActivityItem`

### `execute()`

```
@Override
    public void execute(ClientTransactionHandler client, IBinder token,
            PendingTransactionActions pendingActions) {
        Trace.traceBegin(TRACE_TAG_ACTIVITY_MANAGER, "activityResume");
        client.handleResumeActivity(token, true /* finalStateRequest */, mIsForward,
                "RESUME_ACTIVITY");
        Trace.traceEnd(TRACE_TAG_ACTIVITY_MANAGER);
    }
```

`ClientTransactionHandler`是一个抽象类，具体实现是`ActivityThread`

## `ActivityThread`

`handleResumeActivity`类似于这种方法还有很多，例如：`handlePauseActivity`,`handleStopActivity`等等。对应的Activity的生命周期.

### `handleResumeActivity()`

```
@Override
    public void handleResumeActivity(IBinder token, boolean finalStateRequest, boolean isForward,
            String reason) {
        ...
        final ActivityClientRecord r = performResumeActivity(token, finalStateRequest, reason);
        ...
       Looper.myQueue().addIdleHandler(new Idler());
}
```

### `performResumeActivity()`

```
public ActivityClientRecord performResumeActivity(IBinder token, boolean finalStateRequest,
            String reason) {
        final ActivityClientRecord r = mActivities.get(token);
        
        ...
        r.activity.performResume(r.startsNotResumed, reason);
        ...
}        
```

最后调用的Activity中的`performResume`

## `Activity`

### `performResume()`

```
final void performResume(boolean followedByPause, String reason) {
				//执行onRestart生命周期
        performRestart(true /* start */, reason);
        mFragments.execPendingActions();
				//执行onResume
        mInstrumentation.callActivityOnResume(this);        
        ...
        mFragments.dispatchResume();
        mFragments.execPendingActions();

        onPostResume();
        ...
    }
```

# `onRestart`

### `performRestart()`

```
final void performRestart(boolean start, String reason) {
			...
			mInstrumentation.callActivityOnRestart(this);
			...
			if (start) {
			//最后执行onStart
                performStart(reason);
            }
}
```

这里会通过`Instrumentation`执行`onRestart`

`performStart`执行`onStart`后面说

## `Instrumentation`

### `callActivityOnRestart()`

```
activity.onRestart();
```

就这一行代码但是已经足够了

## `Activity`

### `onRestart`

```
protected void onRestart() {
        mCalled = true;
    }
```

到此onRestart便完成了。

# `onResume`

回到`performResume()`方法执行`mInstrumentation.callActivityOnResume(this); `

道理是一样的使用`Instrumentation`执行`onResume`

```
public void callActivityOnResume(Activity activity) {
        activity.mResumed = true;
        activity.onResume();
        
        if (mActivityMonitors != null) {
            synchronized (mSync) {
                final int N = mActivityMonitors.size();
                for (int i=0; i<N; i++) {
                    final ActivityMonitor am = mActivityMonitors.get(i);
                    am.match(activity, activity, activity.getIntent());
                }
            }
        }
    }
```

```
protected void onResume() {
        if (DEBUG_LIFECYCLE) Slog.v(TAG, "onResume " + this);
        getApplication().dispatchActivityResumed(this);
        mActivityTransitionState.onResume(this, isTopOfTask());
        if (mAutoFillResetNeeded) {
            if (!mAutoFillIgnoreFirstResumePause) {
                View focus = getCurrentFocus();
                if (focus != null && focus.canNotifyAutofillEnterExitEvent()) {
                    // TODO: in Activity killed/recreated case, i.e. SessionLifecycleTest#
                    // testDatasetVisibleWhileAutofilledAppIsLifecycled: the View's initial
                    // window visibility after recreation is INVISIBLE in onResume() and next frame
                    // ViewRootImpl.performTraversals() changes window visibility to VISIBLE.
                    // So we cannot call View.notifyEnterOrExited() which will do nothing
                    // when View.isVisibleToUser() is false.
                    getAutofillManager().notifyViewEntered(focus);
                }
            }
        }
        mCalled = true;
    }
```

最后我们在看看如何执行onStart方法的,我们在前面分析时说到`cycleToPath`最终执行onStart，来看看如何实现

## `TransactionExecutor`

### `cycleToPath()`

这个方法是根据state计算生命周期的路径，简单说就是，当前处于 `onCreate()` 状态，`setLifecycleState()` 设置的 **final state** 是 **onResume()** ，就需要执行 onCreate 到 onResume 之间的状态，即 `onStart()` 。下面简单看下源码。

```
private void cycleToPath(ActivityClientRecord r, int finish,
            boolean excludeLastState) {
        final int start = r.getLifecycleState();
        log("Cycle from: " + start + " to: " + finish + " excludeLastState:" + excludeLastState);
        final IntArray path = mHelper.getLifecyclePath(start, finish, excludeLastState);
        performLifecycleSequence(r, path);
    }
```

### `performLifecycleSequence()`

```
 private void performLifecycleSequence(ActivityClientRecord r, IntArray path) {
        final int size = path.size();
        for (int i = 0, state; i < size; i++) {
            state = path.get(i);
            log("Transitioning to state: " + state);
            switch (state) {
                case ON_CREATE:
                    mTransactionHandler.handleLaunchActivity(r, mPendingActions,
                            null /* customIntent */);
                    break;
                case ON_START:
                    mTransactionHandler.handleStartActivity(r, mPendingActions);
                    break;
                case ON_RESUME:
                    mTransactionHandler.handleResumeActivity(r.token, false /* finalStateRequest */,
                            r.isForward, "LIFECYCLER_RESUME_ACTIVITY");
                    break;
                case ON_PAUSE:
                    mTransactionHandler.handlePauseActivity(r.token, false /* finished */,
                            false /* userLeaving */, 0 /* configChanges */, mPendingActions,
                            "LIFECYCLER_PAUSE_ACTIVITY");
                    break;
                case ON_STOP:
                    mTransactionHandler.handleStopActivity(r.token, false /* show */,
                            0 /* configChanges */, mPendingActions, false /* finalStateRequest */,
                            "LIFECYCLER_STOP_ACTIVITY");
                    break;
                case ON_DESTROY:
                    mTransactionHandler.handleDestroyActivity(r.token, false /* finishing */,
                            0 /* configChanges */, false /* getNonConfigInstance */,
                            "performLifecycleSequence. cycling to:" + path.get(size - 1));
                    break;
                case ON_RESTART:
                    mTransactionHandler.performRestartActivity(r.token, false /* start */);
                    break;
                default:
                    throw new IllegalArgumentException("Unexpected lifecycle state: " + state);
            }
        }
    }
```

`getLifecyclePath()` 方法就是根据 `start` 和 `finish` 计算出中间的生命周期路径。这里计算出来就是 `ON_START` 。

找到 `ON_START` 分支，调用了 `ActivityThread.handleStartActivity()` 方法，又是一样的套路了，这里就不再赘述了。

依次类推，`ActivityThread` 中应该有如下方法：

```
handleLaunchActivity()
handleStartActivity()
handleResumeActivity()
handlePauseActivity()
handleStopActivity()
handleDestroyActivity()
```

# onPause延续startActivity流程

在`startActivity`启动流程中会调用`ActivityStarter#startActivity()`然后调用`startActivityUnchecked`

## `ActivityStarter`

### `startActivityUnchecked()`

```
private int startActivityUnchecked(final ActivityRecord r, ActivityRecord sourceRecord,
            IVoiceInteractionSession voiceSession, IVoiceInteractor voiceInteractor,
            int startFlags, boolean doResume, ActivityOptions options, TaskRecord inTask,
            ActivityRecord[] outActivity) {

        setInitialState(r, options, inTask, doResume, startFlags, sourceRecord, voiceSession,
                voiceInteractor);
				// 处理activity的启动模式.绑定activity的启动模式
        computeLaunchingTaskFlags();
        computeSourceStack();
        
        //给相应activity的intent设置flag
        mIntent.setFlags(mLaunchFlags);

        .......
            //根据activity的启动模式设置相应的栈
            if (mStartActivity.getTask() == null && !clearTopAndResetStandardLaunchMode) {
                mStartActivity.setTask(reusedActivity.getTask());
            }

            if (reusedActivity.getTask().intent == null) {
                // This task was started because of movement of the activity based on affinity...
                // Now that we are actually launching it, we can assign the base intent.
                //设置ActivityRecord,之后会传入到app_service进行activity的周期控制
                reusedActivity.getTask().setIntent(mStartActivity);
            }

       
            //当activity设置为FLAG_ACTIVITY_CLEAR_TOP是的逻辑,即判断栈顶是否有目标activty,没有就重新创建,并且加入栈顶
            if ((mLaunchFlags & FLAG_ACTIVITY_CLEAR_TOP) != 0
                    || isDocumentLaunchesIntoExisting(mLaunchFlags)
                    || isLaunchModeOneOf(LAUNCH_SINGLE_INSTANCE, LAUNCH_SINGLE_TASK)) {
                final TaskRecord task = reusedActivity.getTask();

               
                final ActivityRecord top = task.performClearTaskForReuseLocked(mStartActivity,
                        mLaunchFlags);

                // The above code can remove {@code reusedActivity} from the task, leading to the
             
              ......
            //设置目标栈与现有的activity匹配.根据需要清除栈
            reusedActivity = setTargetStackAndMoveToFrontIfNeeded(reusedActivity);

            .......
        //根据activity的启动模式来判断是直接插入已存在栈顶还是重新开始插入;
        mTargetStack.startActivityLocked(mStartActivity, topFocused, newTask, mKeepCurTransition,
                mOptions);
        if (mDoResume) {   // mDoResume = true
            final ActivityRecord topTaskActivity =
                    mStartActivity.getTask().topRunningActivityLocked();
            if (!mTargetStack.isFocusable()
                    || (topTaskActivity != null && topTaskActivity.mTaskOverlay
                    && mStartActivity != topTaskActivity)) {
                
                mTargetStack.ensureActivitiesVisibleLocked(null, 0, !PRESERVE_WINDOWS);
              
                mService.mWindowManager.executeAppTransition();
            } else {
             
                if (mTargetStack.isFocusable() && !mSupervisor.isFocusedStack(mTargetStack)) {
                    mTargetStack.moveToFront("startActivityUnchecked");
                }
                
                //注释8
                mSupervisor.resumeFocusedStackTopActivityLocked(mTargetStack, mStartActivity,
                        mOptions);
            }
        } else if (mStartActivity != null) {
            mSupervisor.mRecentTasks.add(mStartActivity.getTask());
        }
        .......

        return START_SUCCESS;
    } 
```

## `ActivityStackSupervisor`

### `resumeFocusedStackTopActivityLocked()`

```
boolean resumeFocusedStackTopActivityLocked(
            ActivityStack targetStack, ActivityRecord target, ActivityOptions targetOptions) {

        ...
        if (targetStack != null && isFocusedStack(targetStack)) {
            return targetStack.resumeTopActivityUncheckedLocked(target, targetOptions);
        }
				...

        return false;
    }
```

判断targetStack不为空 且isFocusedStack(targetStack) == true targetStack 就是activity的任务栈 用于管理即将启动的activity的任务栈,进而调用:`targetStack.resumeTopActivityUncheckedLocked(target, targetOptions);`

## `ActivityStack`

### `targetStack.resumeTopActivityUncheckedLocked()`

```
 boolean resumeTopActivityUncheckedLocked(ActivityRecord prev, ActivityOptions options) {
        	...
            result = resumeTopActivityInnerLocked(prev, options);
					...
        return result;
    }
```

### `resumeTopActivityInnerLocked()`

```
private boolean resumeTopActivityInnerLocked(ActivityRecord prev, ActivityOptions options) {
        // 处理下一个 activity?
        final ActivityRecord next = topRunningActivityLocked(true /* focusableOnly */);

       ......
        boolean pausing = mStackSupervisor.pauseBackStacks(userLeaving, next, false);
        // 注释1
        if (mResumedActivity != null) {
            if (DEBUG_STATES) Slog.d(TAG_STATES,
                    "resumeTopActivityLocked: Pausing " + mResumedActivity);
            pausing |= startPausingLocked(userLeaving, false, next, false);
        }
        if (pausing && !resumeWhilePausing) {
            if (DEBUG_SWITCH || DEBUG_STATES) Slog.v(TAG_STATES,
                    "resumeTopActivityLocked: Skip resume: need to start pausing");
            
            if (next.app != null && next.app.thread != null) {
                mService.updateLruProcessLocked(next.app, true, null);
            }
            if (DEBUG_STACK) mStackSupervisor.validateTopActivitiesLocked();
            if (lastResumed != null) {
                lastResumed.setWillCloseOrEnterPip(true);
            }
            return true;
        } else if (mResumedActivity == next && next.isState(RESUMED)
                && mStackSupervisor.allResumedActivitiesComplete()) {
            executeAppTransition(options);
            if (DEBUG_STATES) Slog.d(TAG_STATES,
                    "resumeTopActivityLocked: Top activity resumed (dontWaitForPause) " + next);
            if (DEBUG_STACK) mStackSupervisor.validateTopActivitiesLocked();
            return true;
        }
                    .......
                    //注释2
                    mStackSupervisor.startSpecificActivityLocked(next, true, false);
                    if (DEBUG_STACK) mStackSupervisor.validateTopActivitiesLocked();
                    return true;
                }
            }
       ......
        return true;
    }
```

- 注释1:从这里的源码可以看到当启动新的activity的时候,上一个activity走完onPause,之后才可以走新activity:onCreate --->onStart--->onResume...
- 注释2:回调过程重新交给ActivityStackSupervisor的startSpecificActivityLocked;继续走startActivity的流程，也可以说是onCreate()流程。

# 开始onPause

### `startPausingLocked()`

```
final boolean startPausingLocked(boolean userLeaving, boolean uiSleeping,
            ActivityRecord resuming, boolean pauseImmediately) {
        if (mPausingActivity != null) {
        ...
        mStackSupervisor.resumeFocusedStackTopActivityLocked();
        ...
        if (prev.app != null && prev.app.thread != null) {
            if (DEBUG_PAUSE) Slog.v(TAG_PAUSE, "Enqueueing pending pause: " + prev);
            try {
                EventLogTags.writeAmPauseActivity(prev.userId, System.identityHashCode(prev),
                        prev.shortComponentName, "userLeaving=" + userLeaving);
                mService.updateUsageStats(prev, false);
								//还是原来的配方，还是原来的味道
                mService.getLifecycleManager().scheduleTransaction(prev.app.thread, prev.appToken,
                        PauseActivityItem.obtain(prev.finishing, userLeaving,
                                prev.configChangeFlags, pauseImmediately));
            } catch (Exception e) {
                // Ignore exception, if process died other code will cleanup.
                Slog.w(TAG, "Exception thrown during pause", e);
                mPausingActivity = null;
                mLastPausedActivity = null;
                mLastNoHistoryActivity = null;
            }
        } else {
            mPausingActivity = null;
            mLastPausedActivity = null;
            mLastNoHistoryActivity = null;
        }
        ...
}        
```

` mService.getLifecycleManager().scheduleTransaction(prev.app.thread, prev.appToken,
                        PauseActivityItem.obtain(prev.finishing, userLeaving,
                                prev.configChangeFlags, pauseImmediately));`

多么的相似直接`scheduleTransaction`,就是复制上面的流程,最终执行：

## `ActivityThread`

### `handleStopActivity()`

```
Override
    public void handlePauseActivity(IBinder token, boolean finished, boolean userLeaving,
            int configChanges, PendingTransactionActions pendingActions, String reason) {
        ActivityClientRecord r = mActivities.get(token);
        if (r != null) {
            if (userLeaving) {
                performUserLeavingActivity(r);
            }

            r.activity.mConfigChangeFlags |= configChanges;
            performPauseActivity(r, finished, reason, pendingActions);

            // Make sure any pending writes are now committed.
            if (r.isPreHoneycomb()) {
                QueuedWork.waitToFinish();
            }
            mSomeActivitiesChanged = true;
        }
    }
```

# `onStop`

onStop执行是在`onResume`执行方法中，具体在`ActivityThread`的`handleResumeActivity`

## `ActivityThread`

### `handleResumeActivity()`

```
@Override
    public void handleResumeActivity(IBinder token, boolean finalStateRequest, boolean isForward,
            String reason) {
        ...
        final ActivityClientRecord r = performResumeActivity(token, finalStateRequest, reason);
        ...
       Looper.myQueue().addIdleHandler(new Idler());
}
```

接下来看一下Idler都做了什么。当MessageQueue空闲的时候就会回调Idler.queueIdle方法，经过层层调用跳转到ActivityStack.stopActivityLocked方法。

```java
private class Idler implements MessageQueue.IdleHandler {
        @Override
        public final boolean queueIdle() {
           		...
              am.activityIdle(a.token, a.createdConfig, stopProfiling);
              ...
                        
            return false;
        }
    }
```

am最终执行的是AMS中同名方法

## `AMS`

### `activityIdle()`

```
@Override
    public final void activityIdle(IBinder token, Configuration config, boolean stopProfiling) {
        final long origId = Binder.clearCallingIdentity();
        synchronized (this) {
            ActivityStack stack = ActivityRecord.getStackLocked(token);
            if (stack != null) {
                ActivityRecord r =
                        mStackSupervisor.activityIdleInternalLocked(token, false /* fromTimeout */,
                                false /* processPausingActivities */, config);
                if (stopProfiling) {
                    if ((mProfileProc == r.app) && mProfilerInfo != null) {
                        clearProfilerLocked();
                    }
                }
            }
        }
        Binder.restoreCallingIdentity(origId);
    }
```

## `ActivityStackSupervisor`

### `activityIdleInternalLocked()`

```
final ActivityRecord activityIdleInternalLocked(final IBinder token, boolean fromTimeout,
            boolean processPausingActivities, Configuration config) {
            ...
            for (int i = 0; i < NS; i++) {
            r = stops.get(i);
            final ActivityStack stack = r.getStack();
            if (stack != null) {
                if (r.finishing) {
                    stack.finishCurrentActivityLocked(r, ActivityStack.FINISH_IMMEDIATELY, false,
                            "activityIdleInternalLocked");
                } else {
                    stack.stopActivityLocked(r);
                }
            }
        }
        ...
}            
```

## `ActivityStack`

### `stopActivityLocked()`

```
final void stopActivityLocked(ActivityRecord r) {
				mService.getLifecycleManager().scheduleTransaction(r.app.thread, r.appToken,
                        StopActivityItem.obtain(r.visible, r.configChangeFlags));
}
```

同样的逻辑,最终执行

## `ActivityThread`

### `handleStopActivity()`

```
 @Override
    public void handleStopActivity(IBinder token, boolean show, int configChanges,
            PendingTransactionActions pendingActions, boolean finalStateRequest, String reason) {
        final ActivityClientRecord r = mActivities.get(token);
        r.activity.mConfigChangeFlags |= configChanges;

        final StopInfo stopInfo = new StopInfo();
        performStopActivityInner(r, stopInfo, show, true /* saveState */, finalStateRequest,
                reason);

        if (localLOGV) Slog.v(
            TAG, "Finishing stop of " + r + ": show=" + show
            + " win=" + r.window);

        updateVisibility(r, show);

        // Make sure any pending writes are now committed.
        if (!r.isPreHoneycomb()) {
            QueuedWork.waitToFinish();
        }

        stopInfo.setActivity(r);
        stopInfo.setState(r.state);
        stopInfo.setPersistentState(r.persistentState);
        pendingActions.setStopInfo(stopInfo);
        mSomeActivitiesChanged = true;
    }
```

