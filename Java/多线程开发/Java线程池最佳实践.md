---
title: Java线程池最佳实践
date: 2020-06-09 17:50:48
tags:
---

## 引言

在日常的开发工作当中，线程池往往承载着一个应用中最重要的业务逻辑，因此我们有必要更多地去关注线程池的执行情况，包括异常的处理和分析等等。本文主要聚焦在如何正确使用线程池上，以及提供一些实用的建议。文中会稍微涉及到一些线程池实现原理方面的知识，但是不会做过多展开。网络上关于线程池的原理以及源码解析的文章有很多，感兴趣的同学可以自行查阅。

## 线程池的异常处理

### UncaughtExceptionHandler

我们都知道Runnable接口中的run方法是不允许抛出异常的，因此派生出这个线程的主线程可能无法直接获得该线程在执行过程中的异常信息。如下例：

```
public static void main(String[] args) throws Exception {
    Thread thread = new Thread(() -> {
        Uninterruptibles.sleepUninterruptibly(2, TimeUnit.SECONDS);
        System.out.println(1 / 0); // 这行会导致报错！
    });
    thread.setUncaughtExceptionHandler((t, e) -> {
        e.printStackTrace(); //如果你把这一行注释掉，这个程序将不会抛出任何异常.
    });
    thread.start();
}
复制代码
```

为什么会这样呢？其实我们看一下Thread中的源码就会发现，Thread在执行过程中如果遇到了异常，会先判断当前线程是否有设置UncaughtExceptionHandler，如果没有，则会从线程所在的ThreadGroup中获取。（**注意：**每个线程都有自己的ThreadGroup，即使你没有指定，并且它实现了UncaughtExceptionHandler接口）我们看下ThreadGroup中默认的对UncaughtExceptionHandler接口的实现：

```
public void uncaughtException(Thread t, Throwable e) {
    if (parent != null) {
        parent.uncaughtException(t, e);
    } else {
        Thread.UncaughtExceptionHandler ueh =
            Thread.getDefaultUncaughtExceptionHandler();
        if (ueh != null) {
            ueh.uncaughtException(t, e);
        } else if (!(e instanceof ThreadDeath)) {
            System.err.print("Exception in thread \""
                             + t.getName() + "\" ");
            e.printStackTrace(System.err);
        }
    }
}
复制代码
```

这个ThreadGroup如果有父ThreadGroup，则调用父ThreadGroup的uncaughtException，否则调用全局默认的`Thread.DefaultUncaughtExceptionHandler`，如果全局的handler也没有设置，则只是简单地将异常信息定位到System.err中，这就是为什么我们应当在创建线程的时候，去实现它的UncaughtExceptionHandler接口的原因，这么做可以让你更方便地去排查问题。

### 通过execute提交任务给线程池

回到线程池这个话题，如果我们向线程池提交的任务中，没有对异常进行try...catch处理，并且运行的时候出现了异常，那会对线程池造成什么影响呢？答案是没有影响，线程池依旧可以正常工作，但是异常却被吞掉了。这通常来说不是一个好事情，因为我们需要拿到原始的异常对象去分析问题。

那么怎样才能拿到原始的异常对象呢？我们从线程池的源码着手开始研究这个问题。当然网上关于线程池的源码解析文章有很多，这里限于篇幅，直接给出最相关的部分代码：

```
final void runWorker(Worker w) {
    Thread wt = Thread.currentThread();
    Runnable task = w.firstTask;
    w.firstTask = null;
    w.unlock(); // allow interrupts
    boolean completedAbruptly = true;
    try {
        while (task != null || (task = getTask()) != null) {
            w.lock();
            // If pool is stopping, ensure thread is interrupted;
            // if not, ensure thread is not interrupted.  This
            // requires a recheck in second case to deal with
            // shutdownNow race while clearing interrupt
            if ((runStateAtLeast(ctl.get(), STOP) ||
                 (Thread.interrupted() &&
                  runStateAtLeast(ctl.get(), STOP))) &&
                !wt.isInterrupted())
                wt.interrupt();
            try {
                beforeExecute(wt, task);
                Throwable thrown = null;
                try {
                    task.run();
                } catch (RuntimeException x) {
                    thrown = x; throw x;
                } catch (Error x) {
                    thrown = x; throw x;
                } catch (Throwable x) {
                    thrown = x; throw new Error(x);
                } finally {
                    afterExecute(task, thrown);
                }
            } finally {
                task = null;
                w.completedTasks++;
                w.unlock();
            }
        }
        completedAbruptly = false;
    } finally {
        processWorkerExit(w, completedAbruptly);
    }
}
复制代码
```

这个方法就是真正去执行提交给线程池的任务的代码。这里我们略去其中不相关的逻辑，重点关注第19行到第32行的逻辑，其中第23行是真正开始执行提交给线程池的任务，那么第20行是干什么的呢？其实就是在执行提交给线程池的任务之前可以做一些前置工作，同样的，我们看到第31行，这个是在执行完提交的任务之后，可以做一些后置工作。beforeExecute这个我们暂且不管，重点关注下afterExecute这个方法。我们可以看到，在执行任务过程中，一旦抛出任何类型的异常，都会提交给afterExecute这个方法，然而查看线程池的源代码我们可以发现，默认的afterExecute是个空实现，因此，我们有必要继承ThreadPoolExecutor去实现这个afterExecute方法。（看源码我们可以发现这个afterExecute方法是protected类型的，从官方注释上也可以看到，这个方法就是推荐子类去实现的）当然，这个方法不能随意去实现，需要遵循一定的步骤，具体的官方注释也有讲，这里摘抄如下：

```
 *  <pre> {@code
 * class ExtendedExecutor extends ThreadPoolExecutor {
 *   // ...
 *   protected void afterExecute(Runnable r, Throwable t) {
 *     super.afterExecute(r, t);
 *     if (t == null && r instanceof Future<?>) {
 *       try {
 *         Object result = ((Future<?>) r).get();
 *       } catch (CancellationException ce) {
 *           t = ce;
 *       } catch (ExecutionException ee) {
 *           t = ee.getCause();
 *       } catch (InterruptedException ie) {
 *           Thread.currentThread().interrupt(); // ignore/reset
 *       }
 *     }
 *     if (t != null)
 *       System.out.println(t);
 *   }
 * }}</pre>
复制代码
```

那么通过这种方式，就可以将原先可能被线程池吞掉的异常成功捕获到，从而便于排查问题。

但是这里还有个小问题，我们注意到在runWorker方法中，执行`task.run();`语句之后，各种类型的异常都被抛出了，那这些被抛出的异常去了哪里？事实上这里的异常对象最终会被传入到Thread的dispatchUncaughtException方法中，源码如下：

```
private void dispatchUncaughtException(Throwable e) {
    getUncaughtExceptionHandler().uncaughtException(this, e);
}
复制代码
```

可以看到它会去获取UncaughtExceptionHandler的实现类，然后调用其中的uncaughtException方法，这也就回到了我们上一小节所分析的UncaughtExceptionHandler实现的具体逻辑。那么为了拿到最原始的异常对象，除了实现UncaughtExceptionHandler接口之外，也可以考虑实现afterExecute方法。

### 通过submit提交任务到线程池

这个同样很简单，我们还是先回到submit方法的源码：

```
public <T> Future<T> submit(Callable<T> task) {
    if (task == null) throw new NullPointerException();
    RunnableFuture<T> ftask = newTaskFor(task);
    execute(ftask);
    return ftask;
}
复制代码
```

这里的execute方法调用的是ThreadPoolExecutor中的execute方法，执行逻辑跟通过execute提交任务到线程池是一样的。我们先重点关注这里的newTaskFor方法，其源码如下：

```
protected <T> RunnableFuture<T> newTaskFor(Callable<T> callable) {
    return new FutureTask<T>(callable);
}
复制代码
```

可以看到提交的Callable对象用FutureTask封装起来了。那么我们知道最终会执行到上述runWorker这个方法中，并且最核心的执行逻辑就是`task.run();`这行代码。我们知道这里的task其实是FutureTask类型，因此我们有必要看一下FutureTask中的run方法的实现：

```
public void run() {
    if (state != NEW ||
        !UNSAFE.compareAndSwapObject(this, runnerOffset,
                                     null, Thread.currentThread()))
        return;
    try {
        Callable<V> c = callable;
        if (c != null && state == NEW) {
            V result;
            boolean ran;
            try {
                result = c.call();
                ran = true;
            } catch (Throwable ex) {
                result = null;
                ran = false;
                setException(ex);
            }
            if (ran)
                set(result);
        }
    } finally {
        // runner must be non-null until state is settled to
        // prevent concurrent calls to run()
        runner = null;
        // state must be re-read after nulling runner to prevent
        // leaked interrupts
        int s = state;
        if (s >= INTERRUPTING)
            handlePossibleCancellationInterrupt(s);
    }
}
复制代码
```

可以看到这其中跟异常相关的最关键的代码就在第17行，也就是`setException(ex);`这个地方。我们看一下这个地方的实现：

```
protected void setException(Throwable t) {
    if (UNSAFE.compareAndSwapInt(this, stateOffset, NEW, COMPLETING)) {
        outcome = t;
        UNSAFE.putOrderedInt(this, stateOffset, EXCEPTIONAL); // final state
        finishCompletion();
    }
}
复制代码
```

这里最关键的地方就是将异常对象赋值给了outcome，outcome是FutureTask中的成员变量，我们通过调用submit方法，拿到一个Future对象之后，再调用它的get方法，其中最核心的方法就是report方法，下面给出每个方法的源码：

首先是get方法：

```
public V get() throws InterruptedException, ExecutionException {
    int s = state;
    if (s <= COMPLETING)
        s = awaitDone(false, 0L);
    return report(s);
}
复制代码
```

可以看到最终调用了report方法，其源码如下：

```
private V report(int s) throws ExecutionException {
    Object x = outcome;
    if (s == NORMAL)
        return (V)x;
    if (s >= CANCELLED)
        throw new CancellationException();
    throw new ExecutionException((Throwable)x);
}
复制代码
```

上面是一些状态判断，如果当前任务不是正常执行完毕，或者被取消的话，那么这里的x其实就是原始的异常对象，可以看到会被ExecutionException包装。因此在你调用get方法时，可能会抛出ExecutionException异常，那么调用它的getCause方法就可以拿到最原始的异常对象了。

综上所述，针对提交给线程池的任务可能会抛出异常这一问题，主要有以下两种处理思路：

1. 在提交的任务当中自行try...catch，但这里有个不好的地方就是如果你会提交多种类型的任务到线程池中，每种类型的任务都需要自行将异常try...catch住，比较繁琐。而且如果你只是`catch(Exception e)`，可能依然会漏掉一些包括Error类型的异常，那为了保险起见，你可以考虑`catch(Throwable t)`.
2. 自行实现线程池的afterExecute方法，或者实现Thread的UncaughtExceptionHandler接口。

下面给出我个人创建线程池的一个示例，供大家参考：

```
BlockingQueue<Runnable> queue = new ArrayBlockingQueue<>(DEFAULT_QUEUE_SIZE);
statisticsThreadPool = new ThreadPoolExecutor(DEFAULT_CORE_POOL_SIZE, DEFAULT_MAX_POOL_SIZE,
        60, TimeUnit.SECONDS, queue, new ThreadFactoryBuilder()
        .setThreadFactory(new ThreadFactory() {
            private int count = 0;
            private String prefix = "StatisticsTask";

            @Override
            public Thread newThread(Runnable r) {
                return new Thread(r, prefix + "-" + count++);
            }
        }).setUncaughtExceptionHandler((t, e) -> {
            String threadName = t.getName();
            logger.error("statisticsThreadPool error occurred! threadName: {}, error msg: {}", threadName, e.getMessage(), e);
        }).build(), (r, executor) -> {
    if (!executor.isShutdown()) {
        logger.warn("statisticsThreadPool is too busy! waiting to insert task to queue! ");
        Uninterruptibles.putUninterruptibly(executor.getQueue(), r);
    }
}) {
    @Override
    protected void afterExecute(Runnable r, Throwable t) {
        super.afterExecute(r, t);
        if (t == null && r instanceof Future<?>) {
            try {
                Future<?> future = (Future<?>) r;
                future.get();
            } catch (CancellationException ce) {
                t = ce;
            } catch (ExecutionException ee) {
                t = ee.getCause();
            } catch (InterruptedException ie) {
                Thread.currentThread().interrupt(); // ignore/reset
            }
        }
        if (t != null) {
            logger.error("statisticsThreadPool error msg: {}", t.getMessage(), t);
        }
    }
};
statisticsThreadPool.prestartAllCoreThreads();
复制代码
```

## 线程数的设置

我们知道任务一般有两种：CPU密集型和IO密集型。那么面对CPU密集型的任务，线程数不宜过多，一般选择CPU核心数+1或者核心数的2倍是比较合理的一个值。因此我们可以考虑将corePoolSize设置为CPU核心数+1，maxPoolSize设置为核心数的2倍。那么同样的，面对IO密集型任务时，我们可以考虑以核心数乘以4倍作为核心线程数，然后核心数乘以5倍作为最大线程数的方式去设置线程数，这样的设置会比直接拍脑袋设置一个值会更合理一些。

当然总的线程数不宜过多，控制在100个线程以内比较合理，否则线程数过多可能会导致频繁地上下文切换，导致系统性能反不如前。

## 如何正确关闭一个线程池

说到如何正确去关闭一个线程池，这里面也有点讲究。为了实现优雅停机的目标，我们应当先调用shutdown方法，调用这个方法也就意味着，这个线程池不会再接收任何新的任务，但是已经提交的任务还会继续执行，包括队列中的。所以，之后你还应当调用awaitTermination方法，这个方法可以设定线程池在关闭之前的最大超时时间，如果在超时时间结束之前线程池能够正常关闭，这个方法会返回true，否则，一旦超时，就会返回false。通常来说我们不可能无限制地等待下去，因此需要我们事先预估一个合理的超时时间，然后去使用这个方法。

如果awaitTermination方法返回false，你又希望尽可能在线程池关闭之后再做其他资源回收工作，你可以考虑再调用一下shutdownNow方法，此时队列中所有尚未被处理的任务都会被丢弃，同时会设置线程池中每个线程的中断标志位。shutdownNow并不保证一定可以让正在运行的线程停止工作，除非提交给线程的任务能够正确响应中断。到了这一步，你可以考虑继续调用awaitTermination方法，或者你直接放弃，去做接下来要做的事情。

## 线程池中的其他有用方法

大家可能有留意到，我在创建线程池的时候，还调用了这个方法：**prestartAllCoreThreads**。这个方法有什么作用呢？我们知道一个线程池创建出来之后，在没有给它提交任何任务之前，这个线程池中的线程数为0。有时候我们事先知道会有很多任务会提交给这个线程池，但是等它一个个去创建新线程开销太大，影响系统性能，因此可以考虑在创建线程池的时候就将所有的核心线程全部一次性创建完毕，这样系统起来之后就可以直接使用了。

其实线程池中还提供了其他一些比较有意思的方法。比如我们现在设想一个场景，当一个线程池负载很高，快要撑爆导致触发拒绝策略时，有没有什么办法可以缓解这一问题？其实是有的，因为线程池提供了设置核心线程数和最大线程数的方法，它们分别是**setCorePoolSize方法**和**setMaximumPoolSize方法**。是的，**线程池创建完毕之后也是可以更改其线程数的！**因此，面对线程池高负荷运行的情况，我们可以这么处理：

1. 起一个定时轮询线程（守护类型），定时检测线程池中的线程数，具体来说就是调用getActiveCount方法
2. 当发现线程数超过了核心线程数大小时，可以考虑将CorePoolSize和MaximumPoolSize的数值同时乘以2，当然这里不建议设置很大的线程数，因为并不是线程越多越好的，可以考虑设置一个上限值，比如50, 100之类的。
3. 同时，去获取队列中的任务数，具体来说是调用getQueue方法再调用size方法。当队列中的任务数少于队列大小的二分之一时，我们可以认为现在线程池的负载没有那么高了，因此可以考虑在线程池先前有扩容过的情况下，将CorePoolSize和MaximumPoolSize还原回去，也就是除以2

具体来说如下图：

![image.png](https://upload-images.jianshu.io/upload_images/4118241-d67af9d310145de7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



以上是我个人建议的一种使用线程池的方式。

## 线程池一定是最佳方案吗？

线程池并非在任何情况下都是性能最优的方案。如果是一个追求极致性能的场景，可以考虑使用[Disruptor](https://lmax-exchange.github.io/disruptor/)，这是一个高性能队列。排除Disruptor不谈，单纯基于JDK的话会不会有更好地方案？答案是有的。

我们知道在一个线程池中，多个线程是共用一个队列的，因此在任务很多的情况下，需要对这个队列进行频繁读写，为了防止冲突因此需要加锁。事实上在阅读线程池源代码的时候就可以发现，里面充斥着各种加锁的代码，那有没有更好的实现方式呢？其实我们可以考虑创建一个由单线程线程池构成的列表，每个线程池都使用有界队列这种方式去实现多线程。这么做的好处是，每个线程池中的队列都只会被一个线程去操作，这样就没有竞争的问题。

其实这种用空间换时间的思路借鉴了Netty中的EventLoop的实现机制。试想，如果线程池的性能真的有那么好，为什么Netty不用呢？

## 其他需要注意的地方

1. 任何情况下都不应该使用可伸缩线程池（线程的创建和销毁开销是很大的）
2. 任何情况下都不应该使用无界队列，单测除外（有界队列常用的有ArrayBlockingQueue和LinkedBlockingQueue，前者基于数组实现，后者基于链表。从性能表现上来看，LinkedBlockingQueue的吞吐量更高但是性能并不稳定，实际情况下应当使用哪一种建议自行测试之后决定。顺便说一句，Executors的newFixedThreadPool采用的是LinkedBlockingQueue）
3. 推荐自行实现RejectedExecutionHandler，JDK自带的都不是很好用，你可以在里面实现自己的逻辑。如果需要一些特定的上下文信息，你可以在Runnable实现类中添加一些自己的东西，这样在RejectedExecutionHandler中就可以直接使用了。

## 怎样做到不丢任务

这里其实指的是一种特殊情况，就是比如突然遇到了一股流量尖峰，导致线程池负载已经非常高了，即快要触发拒绝策略的时候，我们可以怎么做来尽量防止提交的任务不会丢失。一般来说当遇到这种情况的时候，应当尽快触发报警通知研发人员来处理。之后不管是限流也好，还是增加机器也好，甚至是上kafka，redis甚至是数据库用来暂存任务数据也是可以的，但毕竟远水救不了近火，如果我们希望在正式解决这个问题之前，先尽可能地缓解，可以考虑怎么做呢？

首先可以考虑的就是我前面有提过的动态增大线程池中的线程数，但是假如说已经扩容过了，此时不应继续扩容，否则可能导致系统的吞吐量更低。在这种情况下，应当自行实现RejectedExecutionHandler，具体来说就是在实现类中，单独开一个单线程的线程池，然后调用原线程池的getQueue方法的put方法，将塞不进去的任务再次尝试塞进去。当然在队列满的时候是塞不进去的，但那至少也只是阻塞了这个单独的线程而已，并不影响主流程。

当然，这种方案是治标不治本的，面对流量激增这种场景其实业界有很多成熟的做法，只是单纯从线程池的角度来看的话，这种方式不失为一种临时的解决方案。



作者：非典型普通码农
链接：https://juejin.im/post/5d4e5c9de51d453c11684c25