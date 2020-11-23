---
title: Thread类分析
date: 2020-05-27 14:53:16
tags:
	- Thread
---

# Thead类

写的不好 别看了，对注释的解释。

## Thread类 解释

线程是程序中的执行线程。 Java虚拟机允许应用程序具有多个并发运行的执行线程。

每个线程都有一个优先级。 具有较高优先级的线程**优先**于具有较低优先级的线程执行。 也可以把线程标记为守护线程。 在某个线程中运行的代码创建新的`<code> Thread </ code>`对象时，新线程的优先级和创建线程的优先级相同。当且仅当创建线程是守护程序时，它才是守护程序线程。

当Java虚拟机启动时，通常只有一个非守护程序线程（通常调用某些指定类的名为`<code> main </ code>`的方法）。Java虚拟机将继续执行线程，直到发生以下任何一种情况：

- 已调用类`<code> Runtime </ code>`的`<code> exit </ code>`方法，并且安全管理器已允许进行退出操作。
- 所有的非守护线程都已死亡，要么通过从调用返回到`<code> run </ code>`方法，要么抛出异常传播到<code> run `</ code>`方法之外。

有两种方法可以创建新的执行线程。 一种是将一个类声明为<code> Thread </ code>的子类。 该子类应重写类<code> Thread </ code>的<code> run </ code>方法。 然后可以分配并启动子类的实例。 例如，计算素数大于指定值的线程可以编写如下：

```
class PrimeThread extends Thread {
 *         long minPrime;
 *         PrimeThread(long minPrime) {
 *             this.minPrime = minPrime;
 *         }
 *
 *         public void run() {
 *             // 计算大于minPrime的素数
 *             &nbsp;.&nbsp;.&nbsp;.
 *         }
 *     }
```

下面的代码将创建一个线程并开始运行：

```
PrimeThread p = new PrimeThread(143);
p.start();
```

创建线程的另一种方法是声明一个实现<code> Runnable </ code>接口的类。 然后，该类实现<code> run </ code>方法。 然后可以分配该类的实例，在创建<code> Thread </ code>时将其作为参数传递并启动。 其他样式的相同示例如下所示：

```
class PrimeRun implements Runnable {
 *         long minPrime;
 *         PrimeRun(long minPrime) {
 *             this.minPrime = minPrime;
 *         }
 *
 *         public void run() {
 *             // compute primes larger than minPrime
 *             &nbsp;.&nbsp;.&nbsp;.
 *         }
 *     }
```

下面的代码将创建一个线程并开始运行：

```
 PrimeRun p = new PrimeRun(143);
 new Thread(p).start();
```

每个线程都有一个名称供识别。 一个以上的线程可能具有相同的名称。 如果在创建线程时未指定名称，则会为其生成一个新名称。

除非另有说明，否则将{@code null}参数传递给此类中的构造函数或方法将导致抛出{@link NullPointerException}。



## Thread 常量与属性

```
class Thread implements Runnable {
/ *确保registerNatives是<clinit>要做的第一件事。 * /

		*负责此线程的join / sleep / park操作的同步对象。
    private final Object lock = new Object();
 private volatile long nativePeer;

    boolean started = false;

    private volatile String name;

    private int         priority;
    private Thread      threadQ;
    private long        eetop;

    /* Whether or not to single_step this thread. */
    / *是否单步执行此线程。 * /
    private boolean     single_step;

    /* Whether or not the thread is a daemon thread. */
    / *线程是否是守护程序线程。 * /
    private boolean     daemon = false;

    /* JVM state */
    //JVM 状态
    private boolean     stillborn = false;

    /* What will be run. */
    //将要被运行的内容
    private Runnable target;

    /* The group of this thread */
    //当前线程的组
    private ThreadGroup group;

    /* The context ClassLoader for this thread */
    //此线程的上下文ClassLoader
    private ClassLoader contextClassLoader;

    /* The inherited AccessControlContext of this thread */
    //该线程的继承的AccessControlContext
    private AccessControlContext inheritedAccessControlContext;

    /* For autonumbering anonymous threads. */
    //用于自动编号匿名线程。
    private static int threadInitNumber;
    
    private static synchronized int nextThreadNum() {
        return threadInitNumber++;
    }

    /* ThreadLocal values pertaining to this thread. This map is maintained
     * by the ThreadLocal class. */
     //与此线程有关的ThreadLocal值。 该映射由ThreadLocal类维护。 * /
    ThreadLocal.ThreadLocalMap threadLocals = null;

    /*
     * InheritableThreadLocal values pertaining to this thread. This map is
     * maintained by the InheritableThreadLocal class.
     */
     //与此线程有关的InheritableThreadLocal值。 该映射由InheritableThreadLocal类维护。
    ThreadLocal.ThreadLocalMap inheritableThreadLocals = null;

    /*
     * The requested stack size for this thread, or 0 if the creator did
     * not specify a stack size.  It is up to the VM to do whatever it
     * likes with this number; some VMs will ignore it.
     */
     //此线程请求的堆栈大小，如果创建者未指定堆栈大小，则为0。 由VM决定使用此编号执行任何操作。 一些虚拟机将忽略它。
    private long stackSize;

    /*
     * JVM-private state that persists after native thread termination.
     */
     //JVM私有状态在本地线程终止后仍然存在。
    private long nativeParkEventPointer;

    /*
     * Thread ID
     */
    private long tid;

    /* For generating thread ID */
    //用于生成线程ID
    private static long threadSeqNumber;

    /* Java thread status for tools,
     * initialized to indicate thread 'not yet started'
     */
		//Java线程状态，已初始化以指示线程“尚未启动”
    private volatile int threadStatus = 0;

/**
     * The argument supplied to the current call to
     * java.util.concurrent.locks.LockSupport.park.
     * Set by (private) java.util.concurrent.locks.LockSupport.setBlocker
     * Accessed using java.util.concurrent.locks.LockSupport.getBlocker
     */
    volatile Object parkBlocker;
    
    / **
      *线程可以具有的最低优先级。
      * /
     public final static int MIN_PRIORITY = 1;

    / **
      *分配给线程的默认优先级。
      * /
     public final static int NORM_PRIORITY = 5;

     / **
      *线程可以具有的最大优先级。
      * /
     public final static int MAX_PRIORITY = 10;
}
```



## 构造方法

```
public Thread() {
        init(null, null, "Thread-" + nextThreadNum(), 0);
    }
    
public Thread(Runnable target) {
        init(null, target, "Thread-" + nextThreadNum(), 0);
    }
    
public Thread(ThreadGroup group, Runnable target) {
        init(group, target, "Thread-" + nextThreadNum(), 0);
    }   

 public Thread(String name) {
        init(null, null, name, 0);
    }
    
    public Thread(ThreadGroup group, String name) {
        init(group, null, name, 0);
    }
    
    Thread(ThreadGroup group, String name, int priority, boolean daemon) {
        this.group = group;
        this.group.addUnstarted();
        // Must be tolerant of threads without a name.
        if (name == null) {
            name = "Thread-" + nextThreadNum();
        }

        // NOTE: Resist the temptation to call setName() here. This constructor is only called
        // by the runtime to construct peers for threads that have attached via JNI and it's
        // undesirable to clobber their natively set name.
        this.name = name;

        this.priority = priority;
        this.daemon = daemon;
        init2(currentThread());
        tid = nextThreadID();
    }

```

前面5个构造都会调用init方法，

## init()

```
/**
     * Initializes a Thread.
     * 创建Thread的公有构造函数，都调用的都是这个私有的init()方法。
     * @param g the Thread group
     * @param target the object whose run() method gets called
     * @param name the name of the new Thread
     * @param stackSize 指定占堆的大小
     */
    private void init(ThreadGroup g, Runnable target, String name, long stackSize) {
				// 获取当前正在运行的线程
        // 当前正在运行的线程就是该我们要创建的线程的父线程
        // 我们要创建的线程会从父线程那继承一些参数过来
        // 注意哦，此时仍然是在原来的线程，新线程此时还没有创建的哦！
				Thread parent = currentThread();
        if (g == null) {
        //如果没有指定和父类一组
            g = parent.getThreadGroup();
        }

			 //将ThreadGroup中的就绪线程计数器增加一。注意，此时线程还并没有被真正加入到ThreadGroup中。
        g.addUnstarted();
        //将Thread实例的group赋值。从这里开始线程就拥有ThreadGroup了。
        this.group = g;

        this.target = target;
        //和开启线程的优先级相同
        this.priority = parent.getPriority();
        //根据父线程是否是守护线程来确定Thread实例是否是守护线程。
        this.daemon = parent.isDaemon();
        setName(name);

        init2(parent);

        /* Stash the specified stack size in case the VM cares */
        this.stackSize = stackSize;
        tid = nextThreadID();
    }
```

在Thread的init()方法中，比较重要的是会通过一个`currentThread()`这样的native函数通过底层从虚拟机中获取到当前运行的线程。

```
public static native Thread currentThread();
```

所以在Thread初始化的时候，仍然是在创建它的线程中。不难猜测出，其实Java层的Thread只是对底层的封装而已。

## init2()

```
private void init2(Thread parent) {
				//设置ClassLoader成员变量
        this.contextClassLoader = parent.getContextClassLoader();
        //设置访问权限控制环境
        this.inheritedAccessControlContext = AccessController.getContext();      
        if (parent.inheritableThreadLocals != null) {
         //创建Thread实例的ThreadLoacaleMap。需要用到父线程的ThreadLocaleMap，目的是为了将父线程中的变量副本拷贝一份到当前线程中。
            //ThreadLocaleMap是一个Entry型的数组，Thread实例会将变量副本保存在这里面。
            this.inheritableThreadLocals = ThreadLocal.createInheritedMap(
                    parent.inheritableThreadLocals);
        }
    }
```

至此，我们的Thread就初始化完成了，Thread的几个重要成员变量都赋值了。

## start()

```
    /**
     * Causes this thread to begin execution; the Java Virtual Machine
     * calls the <code>run</code> method of this thread.
     * <p>
     * The result is that two threads are running concurrently: the
     * current thread (which returns from the call to the
     * <code>start</code> method) and the other thread (which executes its
     * <code>run</code> method).
     * <p>
     * It is never legal to start a thread more than once.
     * In particular, a thread may not be restarted once it has completed
     * execution.
     *
     * @exception  IllegalThreadStateException  if the thread was already
     *               started.
     * @see        #run()
     * @see        #stop()
     */
    public synchronized void start() {
        /**
         * This method is not invoked for the main method thread or "system"
         * group threads created/set up by the VM. Any new functionality added
         * to this method in the future may have to also be added to the VM.
         *
         * A zero status value corresponds to state "NEW".
         */
        // Android-changed: throw if 'started' is true
         //检查线程状态是否为0，为0表示是一个新状态，即还没被start()过。不为0就抛出异常。
        //就是说，我们一个Thread实例，我们只能调用一次start()方法。
        if (threadStatus != 0 || started)
            throw new IllegalThreadStateException();

        /* Notify the group that this thread is about to be started
         * so that it can be added to the group's list of threads
         * and the group's unstarted count can be decremented. */
         //从这里开始才真正的线程加入到ThreadGroup组里。再重复一次，前面只是把nUnstartedThreads这个计数器进行了增量，并没有添加线程。
        //同时，当线程启动了之后，nUnstartedThreads计数器会-1。因为就绪状态的线程少了一条啊！
        group.add(this);

        started = false;
        try {
            //又是个Native方法。这里交由JVM处理，会调用Thread实例的run()方法。
            nativeCreate(this, stackSize, daemon);
            started = true;
        } finally {
            try {
                if (!started) {
                //如果没有被启动成功，Thread将会被移除ThreadGroup，同时，nUnstartedThreads计数器又增量1了。
                    group.threadStartFailed(this);
                }
            } catch (Throwable ignore) {
                /* do nothing. If start0 threw a Throwable then
                  it will be passed up the call stack */
            }
        }
    }
    
    private native static void nativeCreate(Thread t, long stackSize, boolean daemon);
```

使该线程开始执行； Java虚拟机将调用此线程的<code> run </ code>方法。

如我们所见，这个方法是加了锁的。原因是避免开发者在其它线程调用同一个Thread实例的这个方法，从而尽量避免抛出异常。这个方法之所以能够执行我们传入的Runnable里的run()方法，是应为JVM调用了Thread实例的run()方法。

最精华的函数是`nativeCreate(this, stackSize, daemon)`，会去调用底层的JNI函数`Thread_nativeCreate()`，进一步的会调用底层的Thread类的`Thread::CreateNativeThread()`函数。

`Thread::CreateNativeThread()`函数在`/art/runtime/thread.cc`文件中（注：CoorChice用的是6.0.0-r1的源码）。它会在去创建一个c/c++层的Thread对象，并且会关联Java层的Thread对象（其实就是保存一个Java层Thread对象的引用而已）。接着，会通过c/c++层的`pthread_create()`函数去创建并启动一个新线程。这条代码必须要看看了：

```php
pthread_create_result = pthread_create(&new_pthread, &attr, 
    Thread::CreateCallback, child_thread);
```

这里我们需要注意第三个参数位置的`Thread::CreateCallback`，它会返回一个Java层Thread类的run()方法指针，在Linux层的pthread线程创建成功后，将会调用这个run()方法。这就是为什么我们调用start()方法后，run()方法会被调用的原因。

从上面的分析我们可以知道，其实Java的线程Thread还是用的Linux那一套 `pthread` 的东西，并且一条线程真正创建并运行在虚拟机中时，是在调用start()方法之后。所以，如果你创建了一条线程，但是从没调用过它的start()方法，就不会有条新线程生成，此时的Thread对象和主线程里的一个普通对象没什么区别。如果你企图调用 `run()` 方法去试图启动你的线程，那真是大错特错了！这样不过相当于在主线程中调用了一个Java方法而已。

所以，Java中的线程在Android中实际上走的还是Linux的pthread那一套。

```
//没错，就是这么简单！仅仅调用了Runnable类型的成员变量target的run()方法。至此，我们需要执行的代码就执行起来了。
//至于这个@Overrid的存在，完全是因为Thread本身也是一个Runnable！就是说，我们的Thread也可以作为一个Runnable来使用。
@Override
public void run() {
        if (target != null) {
            target.run();
        }
    }
```

# 常用的方法

## Thread.sleep()

sleep使当前线程暂停，并没有释放锁，经过传入的time时间后自动继续执行。

```
    /**
     * Causes the currently executing thread to sleep (temporarily cease
     * execution) for the specified number of milliseconds, subject to
     * the precision and accuracy of system timers and schedulers. The thread
     * does not lose ownership of any monitors.
     *
     * @param  millis
     *         the length of time to sleep in milliseconds
     *
     * @throws  IllegalArgumentException
     *          if the value of {@code millis} is negative
     *
     * @throws  InterruptedException
     *          if any thread has interrupted the current thread. The
     *          <i>interrupted status</i> of the current thread is
     *          cleared when this exception is thrown.
     如果任何线程中断了当前线程。 抛出此异常时，将清除当前线程的<i>中断状态</ i>。
     */
     //根据系统计时器和调度程序的精度和准确性，使当前正在执行的线程进入休眠状态（暂时停止执行）达指定的毫秒数。 该线程不会失去任何监视器的所有权。
    public static void sleep(long millis) throws InterruptedException {
        Thread.sleep(millis, 0);
    }

    @FastNative
    private static native void sleep(Object lock, long millis, int nanos)
        throws InterruptedException;

    /**
     * Causes the currently executing thread to sleep (temporarily cease
     * execution) for the specified number of milliseconds plus the specified
     * number of nanoseconds, subject to the precision and accuracy of system
     * timers and schedulers. The thread does not lose ownership of any
     * monitors.
     *
     * @param  millis
     *         the length of time to sleep in milliseconds
     *
     * @param  nanos
     *         {@code 0-999999} additional nanoseconds to sleep
     *					额外的纳秒睡眠时间
     * @throws  IllegalArgumentException
     *          if the value of {@code millis} is negative, or the value of
     *          {@code nanos} is not in the range {@code 0-999999}
     *
     * @throws  InterruptedException
     *          if any thread has interrupted the current thread. The
     *          <i>interrupted status</i> of the current thread is
     *          cleared when this exception is thrown.
     */
    public static void sleep(long millis, int nanos)
    throws InterruptedException {
        if (millis < 0) {
            throw new IllegalArgumentException("millis < 0: " + millis);
        }
        if (nanos < 0) {
            throw new IllegalArgumentException("nanos < 0: " + nanos);
        }
        if (nanos > 999999) {
            throw new IllegalArgumentException("nanos > 999999: " + nanos);
        }

        // The JLS 3rd edition, section 17.9 says: "...sleep for zero
        // time...need not have observable effects."
        if (millis == 0 && nanos == 0) {
            // ...but we still have to handle being interrupted.
            //当睡眠时间为0时，检测线程是否中断，并清除线程的中断状态标记。这是个Native的方法。
            if (Thread.interrupted()) {
            //如果线程被设置了中断状态为true了(调用Thread.interrupt())。那么他将抛出异常。如果在catch住这个异常之后return线程，那么线程就停止了。  
              throw new InterruptedException();
           //需要注意，在调用了Thread.sleep()之后，再调用isInterrupted()得到的结果永远是False。别忘了Thread.interrupted()在检测的同时还会清除标记位置哦！
            }
            return;
        }
 //类似System.currentTimeMillis()。但是获取的是纳秒，可能不准。
        long start = System.nanoTime();
        long duration = (millis * NANOS_PER_MILLI) + nanos;
				 //获得当前线程的锁。
        Object lock = currentThread().lock;

        // Wait may return early, so loop until sleep duration passes.
//        等待可能会提前返回，因此请循环直到睡眠持续时间过去。
        synchronized (lock) {
            while (true) {
                sleep(lock, millis, nanos);

                long now = System.nanoTime();
                long elapsed = now - start;
								////如果当前睡眠时长，已经满足我们的需求，就退出循环，睡眠结束。
                if (elapsed >= duration) {
                    break;
                }

                duration -= elapsed;
                start = now;
                millis = duration / NANOS_PER_MILLI;
                nanos = (int) (duration % NANOS_PER_MILLI);
            }
        }
    }
```

Android为了确保休眠的准确性，在这里还使用了一个`while()`循环，在每次线程从底层被唤醒后，检查一下是否休眠够了足够的时长。如果不够就让它继续休眠。

## yield()

这个方法是Native的。调用这个方法可以提示cpu，当前线程将放弃目前cpu的使用权，和其它线程重新一起争夺新的cpu使用权限。当前线程可能再次获得执行，也可能没获得。

    /**
     * A hint to the scheduler that the current thread is willing to yield
     * its current use of a processor. The scheduler is free to ignore this
     * hint.
     *
     * <p> Yield is a heuristic attempt to improve relative progression
     * between threads that would otherwise over-utilise a CPU. Its use
     * should be combined with detailed profiling and benchmarking to
     * ensure that it actually has the desired effect.
     *
     * <p> It is rarely appropriate to use this method. It may be useful
     * for debugging or testing purposes, where it may help to reproduce
     * bugs due to race conditions. It may also be useful when designing
     * concurrency control constructs such as the ones in the
     * {@link java.util.concurrent.locks} package.
     */
    public static native void yield();
给调度程序的提示是当前线程愿意放弃当前使用的处理器。 调度程序可以随意忽略此提示。

Yield是一种启发式尝试，旨在提高线程之间的相对进程，否则将过度利用CPU。 应将其使用与详细的性能分析和基准测试结合起来，以确保它实际上具有所需的效果。

很少适合使用此方法。 对于调试或测试目的，它可能很有用，因为它可能有助于重现由于竞争条件而产生的错误。 在设计并发控制结构（例如@link java.util.concurrent.locks}包中的结构）时，它也可能很有用。

## join() 

join方法是等待该线程执行，直到超时或者终止，可以作为线程通信的一种方式，A线程调用B线程的join（阻塞），等待B完成后再往下执行。

```
public final void join(long millis, int nanos)
    throws InterruptedException {
        synchronized(lock) {
        if (millis < 0) {
            throw new IllegalArgumentException("timeout value is negative");
        }

        if (nanos < 0 || nanos > 999999) {
            throw new IllegalArgumentException(
                                "nanosecond timeout value out of range");
        }

        if (nanos >= 500000 || (nanos != 0 && millis == 0)) {
            millis++;
        }

        join(millis);
        }
    }
    
    public final void join(long millis) throws InterruptedException {
        synchronized(lock) {
               //得到当前的系统给时间
        long base = System.currentTimeMillis();
        long now = 0;

        if (millis < 0) {
            throw new IllegalArgumentException("timeout value is negative");
        }

        if (millis == 0) {
           // 如果millis == 0 线程将一直等待下去
           //如果是活跃的的
            while (isAlive()) {
                lock.wait(0);
            }
        } else {
           // 指定了millis ，等待指定时间以后，会break当前线程
            while (isAlive()) {
                long delay = millis - now;
                if (delay <= 0) {
                    break;
                }
                 //有限期的进行等待
                lock.wait(delay);
                now = System.currentTimeMillis() - base;
            }
        }
        }
    }

```

## interrupt()方法

interrupt()方法是中断当前的线程

```
    /**
     * Interrupts this thread.
     *
     * <p> Unless the current thread is interrupting itself, which is
     * always permitted, the {@link #checkAccess() checkAccess} method
     * of this thread is invoked, which may cause a {@link
     * SecurityException} to be thrown.
     *
     * <p> If this thread is blocked in an invocation of the {@link
     * Object#wait() wait()}, {@link Object#wait(long) wait(long)}, or {@link
     * Object#wait(long, int) wait(long, int)} methods of the {@link Object}
     * class, or of the {@link #join()}, {@link #join(long)}, {@link
     * #join(long, int)}, {@link #sleep(long)}, or {@link #sleep(long, int)},
     * methods of this class, then its interrupt status will be cleared and it
     * will receive an {@link InterruptedException}.
     *
     * <p> If this thread is blocked in an I/O operation upon an {@link
     * java.nio.channels.InterruptibleChannel InterruptibleChannel}
     * then the channel will be closed, the thread's interrupt
     * status will be set, and the thread will receive a {@link
     * java.nio.channels.ClosedByInterruptException}.
     *
     * <p> If this thread is blocked in a {@link java.nio.channels.Selector}
     * then the thread's interrupt status will be set and it will return
     * immediately from the selection operation, possibly with a non-zero
     * value, just as if the selector's {@link
     * java.nio.channels.Selector#wakeup wakeup} method were invoked.
     *
     * <p> If none of the previous conditions hold then this thread's interrupt
     * status will be set. </p>
     *
     * <p> Interrupting a thread that is not alive need not have any effect.
     *
     * @throws  SecurityException
     *          if the current thread cannot modify this thread
     *
     * @revised 6.0
     * @spec JSR-51
     */
    public void interrupt() {
        if (this != Thread.currentThread())
            checkAccess();

        synchronized (blockerLock) {
            Interruptible b = blocker;
            if (b != null) {
                nativeInterrupt();
                b.interrupt(this);
                return;
            }
        }
        nativeInterrupt();
    }

    /**
     * Tests whether the current thread has been interrupted.  The
     * <i>interrupted status</i> of the thread is cleared by this method.  In
     * other words, if this method were to be called twice in succession, the
     * second call would return false (unless the current thread were
     * interrupted again, after the first call had cleared its interrupted
     * status and before the second call had examined it).
     *
     * <p>A thread interruption ignored because a thread was not alive
     * at the time of the interrupt will be reflected by this method
     * returning false.
     *
     * @return  <code>true</code> if the current thread has been interrupted;
     *          <code>false</code> otherwise.
     * @see #isInterrupted()
     * @revised 6.0
     */
    @FastNative
    public static native boolean interrupted();

    /**
     * Tests whether this thread has been interrupted.  The <i>interrupted
     * status</i> of the thread is unaffected by this method.
     *
     * <p>A thread interruption ignored because a thread was not alive
     * at the time of the interrupt will be reflected by this method
     * returning false.
     *
     * @return  <code>true</code> if this thread has been interrupted;
     *          <code>false</code> otherwise.
     * @see     #interrupted()
     * @revised 6.0
     */
    @FastNative
    public native boolean isInterrupted();

```

## 线程的状态

```
public enum State {
        /**
         * Thread state for a thread which has not yet started.
         */
        NEW,

        /**
         * Thread state for a runnable thread.  A thread in the runnable
         * state is executing in the Java virtual machine but it may
         * be waiting for other resources from the operating system
         * such as processor.
         */
        RUNNABLE,

        /**
         * Thread state for a thread blocked waiting for a monitor lock.
         * A thread in the blocked state is waiting for a monitor lock
         * to enter a synchronized block/method or
         * reenter a synchronized block/method after calling
         * {@link Object#wait() Object.wait}.
         */
        BLOCKED,

        /**
         * Thread state for a waiting thread.
         * A thread is in the waiting state due to calling one of the
         * following methods:
         * <ul>
         *   <li>{@link Object#wait() Object.wait} with no timeout</li>
         *   <li>{@link #join() Thread.join} with no timeout</li>
         *   <li>{@link LockSupport#park() LockSupport.park}</li>
         * </ul>
         *
         * <p>A thread in the waiting state is waiting for another thread to
         * perform a particular action.
         *
         * For example, a thread that has called <tt>Object.wait()</tt>
         * on an object is waiting for another thread to call
         * <tt>Object.notify()</tt> or <tt>Object.notifyAll()</tt> on
         * that object. A thread that has called <tt>Thread.join()</tt>
         * is waiting for a specified thread to terminate.
         */
        WAITING,

        /**
         * Thread state for a waiting thread with a specified waiting time.
         * A thread is in the timed waiting state due to calling one of
         * the following methods with a specified positive waiting time:
         * <ul>
         *   <li>{@link #sleep Thread.sleep}</li>
         *   <li>{@link Object#wait(long) Object.wait} with timeout</li>
         *   <li>{@link #join(long) Thread.join} with timeout</li>
         *   <li>{@link LockSupport#parkNanos LockSupport.parkNanos}</li>
         *   <li>{@link LockSupport#parkUntil LockSupport.parkUntil}</li>
         * </ul>
         */
        TIMED_WAITING,

        /**
         * Thread state for a terminated thread.
         * The thread has completed execution.
         */
        TERMINATED;
    }

    /**
     * Returns the state of this thread.
     * This method is designed for use in monitoring of the system state,
     * not for synchronization control.
     *
     * @return this thread's state.
     * @since 1.5
     */
    public State getState() {
        // get current thread state
        return State.values()[nativeGetStatus(started)];
    }
```

- NEW 线程刚刚创建，尚未启动
- RUNNABLE 线程正在运行 
- BLOCKED 线程堵塞状态，等待同步块的锁的释放，如果获得锁，就会自动进入运行状态
- WAITING 线程处于等待状态，需要被其他线程唤醒（notify、notifyAll方法的调用）。线程在调用了join之后，也会进入等待状态，直到被join的线程执行结束才会被唤醒
- TIMED_WAITING 有限时间的等待，一般出现在调用sleep(time),join(time),sleep(time)后
- TERMINATED 线程运行结束

## Object.wait()

根据文档的描述，wait()配合notify()和notifyAll()能够实现线程间通讯，即同步。在线程中调用wait()必须在同步代码块中调用，否则会抛出IllegalMonitorStateException异常。因为wait()函数需要释放相应对象的锁。当线程执行到wait()时，对象会把当前线程放入自己的线程池中，并且释放锁，然后阻塞在这个地方。直到该对象调用了notify()或者notifyAll()后，该线程才能重新获得，或者有可能获得对象的锁，然后继续执行后面的语句。

```
 /**
     * Causes the current thread to wait until another thread invokes the
     * {@link java.lang.Object#notify()} method or the
     * {@link java.lang.Object#notifyAll()} method for this object, or
     * some other thread interrupts the current thread, or a certain
     * amount of real time has elapsed.
     * <p>
     * This method is similar to the {@code wait} method of one
     * argument, but it allows finer control over the amount of time to
     * wait for a notification before giving up. The amount of real time,
     * measured in nanoseconds, is given by:
     * <blockquote>
     * <pre>
     * 1000000*timeout+nanos</pre></blockquote>
     * <p>
     * In all other respects, this method does the same thing as the
     * method {@link #wait(long)} of one argument. In particular,
     * {@code wait(0, 0)} means the same thing as {@code wait(0)}.
     * <p>
     * The current thread must own this object's monitor. The thread
     * releases ownership of this monitor and waits until either of the
     * following two conditions has occurred:
     * <ul>
     * <li>Another thread notifies threads waiting on this object's monitor
     *     to wake up either through a call to the {@code notify} method
     *     or the {@code notifyAll} method.
     * <li>The timeout period, specified by {@code timeout}
     *     milliseconds plus {@code nanos} nanoseconds arguments, has
     *     elapsed.
     * </ul>
     * <p>
     * The thread then waits until it can re-obtain ownership of the
     * monitor and resumes execution.
     * <p>
     * As in the one argument version, interrupts and spurious wakeups are
     * possible, and this method should always be used in a loop:
     * <pre>
     *     synchronized (obj) {
     *         while (&lt;condition does not hold&gt;)
     *             obj.wait(timeout, nanos);
     *         ... // Perform action appropriate to condition
     *     }
     * </pre>
     * This method should only be called by a thread that is the owner
     * of this object's monitor. See the {@code notify} method for a
     * description of the ways in which a thread can become the owner of
     * a monitor.
     *
     * @param      millis   the maximum time to wait in milliseconds.
     * @param      nanos      additional time, in nanoseconds range
     *                       0-999999.
     * @throws  IllegalArgumentException      if the value of timeout is
     *                      negative or the value of nanos is
     *                      not in the range 0-999999.
     * @throws  IllegalMonitorStateException  if the current thread is not
     *               the owner of this object's monitor.
     * @throws  InterruptedException if any thread interrupted the
     *             current thread before or while the current thread
     *             was waiting for a notification.  The <i>interrupted
     *             status</i> of the current thread is cleared when
     *             this exception is thrown.
     */
public final void wait(long millis) throws InterruptedException {
        wait(millis, 0);
    }
    
    public final native void wait(long millis, int nanos) throws InterruptedException;
```

使当前线程等待，直到另一个线程为此对象调用{@link java.lang.Object＃notify（）}方法或{@link java.lang.Object＃notifyAll（）}方法，或其他一些线程中断 当前线程，或一定数量的实时时间已经过去。