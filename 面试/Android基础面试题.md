---
title: Android基础面试题
date: 2020-11-06 17:40:37
tags:
---

### 一、屏幕适配

#### 1.1 问：什么是屏幕尺寸？

答：机对角线的物理尺寸 单位：英寸（inch），1英寸=2.54cm

Android手机常见的尺寸有5寸、5.5寸、6寸，6.5寸等等



#### 问：什么是屏幕分辨率？

答：手机在横向、纵向上的像素点数总和。例如：1080x1920，即宽度方向上有1080个像素点，在高度方向上有1920个像素点。

单位：px（pixel），1px=1像素点

Android手机常见的分辨率：320x480、480x800、720x1280、1080x1920



#### 1.3 问：什么是屏幕像素密度？如何计算？

答：每英寸的像素点数。单位：dpi（dots per ich）。例如：设备内每英寸有160个像素，那么该设备的屏幕像素密度=160dpi。

公式：屏幕的像素密度 = 公式：根号下（长的平方+高的平方）➗屏幕尺寸。

假如：一部手机的分辨率是1080×1920，屏幕大小是5英寸，通过宽1080和高1920，根据勾股定理，我们得出对角线的像素数大约是2203个，2203除以5，计算结果是440。440dpi就是屏幕密度

#### 

答：下表是Android手机的密度分类：

| 密度   | ldpi低分辨率 | mdpi中  | hdpi高  | xhdpi超高 | xxhdpi超超高分辨率 | xxxhdpi超超超高分辨率 |
| ------ | ------------ | ------- | ------- | --------- | ------------------ | --------------------- |
| 密度值 | 120          | 160     | 240     | 320       | 480                | 640                   |
| 分辨率 | 240*320      | 320*480 | 480*800 | 120*1280  | 1080*1920          |                       |

根据此表得到比例 ldpi:mdpi:hdpi:xhdpi:xxhdpi = 3:4:6:8:12

#### 1.4 问：什么是密度无关的像素单位？

答：dp，或者dip，使用dp作为像素单位，可以保证在不同的屏幕像素密度的手机上显示 很相似的效果。



#### 1.5 问：px是密度无关的像素单位吗？

答：根据屏幕像素密度工时可知，**px是和像素密度有直接关系**的像素单位。



#### 1.6 问：px和dp的换算公式？

答：px = dp * (dpi / 160)；如果有一个屏幕密度为 160dpi的手机，在它上面，1px=1dp；而如果是 320dpi的手机，则 1px = 0.5dp.



#### 1.7 问：什么是独立比例像素？

答：简称sp或者sip专门用于字体大小表示。

> 推荐使用`12sp`以上的偶数作为 字体大小, 不要使用奇数，或者浮点型小数，因为容易造成精度丢失。



#### 1.8 问：dp和sp 有什么区别？

答：通常情况下，dp和sp效果类似，但是有一点，如果用户 调整了手机字体，比如 从标准，变成了 超大，那么，1dp 原本等于1px依然不变，但是1sp就会从1px变成3px。



#### 1.9 问：说说Android中常用的限定符？

- 最小宽度限定符
  - 例如，如果布局要求屏幕区域的最小尺寸始终至少为 600dp，则可使用此限定符创建布局资源 `res/layout-sw600dp/`。仅当可用屏幕的最小尺寸至少为 600dp时。
- 可用宽度限定符
  - 指定资源应使用的最小可用屏幕宽度（以 `dp` 为单位，由 `<N>` 值定义）。当屏幕方向在横向和纵向之间切换时，此配置值也会随之变化，以匹配当前的实际宽度。例如`w720dp`，`w1024dp`等等
- 可用高度限定符
  - 指定资源应使用的最小可用屏幕高度（以“dp”为单位，由 `<N>` 值定义）。当屏幕方向在横向和纵向之间切换时，此配置值也会随之变化，以匹配当前的实际高度。示例：`h720dp h1024dp`等等
- 屏幕方向限定符
  - `port`：设备处于纵向（垂直）
  - `land`：设备处于横向状态（水平）
- 屏幕像素密度
  - `ldpi`：低密度屏幕；约为 120dpi。
  - `mdpi`：中等密度（传统 HVGA）屏幕；约为 160dpi。
  - `hdpi`：高密度屏幕；约为 240dpi。
  - `xhdpi`：超高密度屏幕；约为 320dpi。*此项为 API 级别 8 中的新增配置*
  - `xxhdpi`：绝高密度屏幕；约为 480dpi。*此项为 API 级别 16 中的新增配置*
  - `xxxhdpi`：极高密度屏幕使用（仅限启动器图标，请参阅*支持多种屏幕*中的[注释](https://developer.android.google.cn/guide/practices/screens_support#xxxhdpi-note)）；约为 640dpi。*此项为 API 级别 18 中的新增配置*
  - `nodpi`：可用于您不希望为匹配设备密度而进行缩放的位图资源。
  - `tvdpi`：密度介于 mdpi 和 hdpi 之间的屏幕；约为 213dpi。此限定符并非指“基本”密度的屏幕。它主要用于电视，且大多数应用都不使用该密度 — 大多数应用只会使用 mdpi 和 hdpi 资源，而且系统将根据需要对这些资源进行缩放。*此项为 API 级别 13 中的新增配置*
  - `anydpi`：此限定符适合所有屏幕密度，其优先级高于其他限定符。这非常适用于[矢量可绘制对象](https://developer.android.google.cn/training/material/drawables#VectorDrawables)。*此项为 API 级别 21 中的新增配置*
  - `nnndpi`：用于表示非标准密度，其中 `*nnn*` 是正整数屏幕密度。此限定符不适用于大多数情况。使用标准密度存储分区，可显著减少因支持市场上各种设备屏幕密度而产生的开销。
- 平台版本（API 级别）限定符
  - 示例：`v21 v4 v7`等等



#### 1.10 问：使用限定符的优缺点：

答：

优点：几乎可以解决所有的适配问题。

缺点：需要修改布局的时候，可能需要修改多套布局，这个要多恶心就多恶心，懂的人都懂.



#### 1.11 问：如何使用布局组件适配？

答：尽量直接通过一套布局解决所有的麻烦，这个就叫布局组件的适配。常见手段：

- 使用像素密度无关的单位 `dp sp`
- 杜绝使用绝对布局，多使用相对和线性布局
- 多使用 wrap_content match_parent 以及线性布局的权重
- 多用 minWidth minHeight,lines 等属性
- 使用多套限定的 dimens中定义的尺寸



#### 1.12 问：说说宽高限定符适配方案？

答：简单说，就是穷举市面上所有的Android手机的宽高像素值。

设定一个基准的分辨率，其他分辨率都根据这个基准分辨率来计算，在不同的尺寸文件夹内部，根据该尺寸编写对应的dimens文件。

比如以480x320为基准分辨率

- 宽度为320，将任何分辨率的宽度整分为320份，取值为x1-x320
- 高度为480，将任何分辨率的高度整分为480份，取值为y1-y480

那么对于800*480的分辨率的dimens文件来说，

- x1=(480/320)*1=1.5px*
- *x2=(480/320)*2=3px

这个时候，如果我们的UI设计界面使用的就是基准分辨率，那么我们就可以按照设计稿上的尺寸填写相对应的dimens引用了,而当APP运行在不同分辨率的手机中时，这些系统会根据这些dimens引用去该分辨率的文件夹下面寻找对应的值。这样基本解决了我们的适配问题，而且极大的提升了我们UI开发的效率，

缺点：需要精准命中才能适配，比如1920x1080的手机就一定要找到1920x1080的限定符，否则就只能用统一的默认的dimens文件了。而使用默认的尺寸的话，UI就很可能变形，简单说，就是容错机制很差

![屏幕快照 2020-11-10 下午3.54.49.png](https://upload-images.jianshu.io/upload_images/4118241-4084fff291042912.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### 1.13 问：说说最小宽度限定符适配？

答：也叫sw限定符适配，指的是Android会识别**屏幕可用高度和宽度的最小尺寸**的dp值（其实就是手机的宽度值），然后根据识别到的结果去资源文件中寻找对应限定符的文件夹下的资源文件。

这种机制和宽高限定符适配原理上是一致的，都是系统通过特定的规则来选择对应的文件。

![image.png](https://upload-images.jianshu.io/upload_images/4118241-a27dd78bf6c14a67.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

**最小宽度限定符适配和宽高限定符适配最大的区别在于，前者有很好的容错机制，如果没有value-sw360dp文件夹，系统会向下寻找，比如离360dp最近的只有value-sw350dp，那么Android就会选择value-sw350dp文件夹下面的资源文件。这个特性就完美的解决了上文提到的宽高限定符的容错问题。**



#### 1.14 问：说说今日头条适配方案原理？

答：通过修改density值，强行把所有不同尺寸分辨率的手机的宽度dp值改成一个统一的值，这样就解决了所有的适配问题。

比如，设计稿宽度是360px，那么开发这边就会把目标dp值设为360dp，在不同的设备中，动态修改density值，从而保证(手机像素宽度)px/density这个值始终是360dp,这样的话，就能保证UI在不同的设备上表现一致了。



### 二、Activity系列

#### 问：说下 Activity跟window，view之间的关系？

- Activity在创建时会调用 **attach()** 方法初始化一个**PhoneWindow(继承于Window)**，**每一个Activity都包含了唯一一个PhoneWindow**
- Activity通过**setContentView**实际上是调用的 **getWindow().setContentView**将View设置到PhoneWindow上，而PhoneWindow内部是通过 **WindowManager** 的**addView**、**removeView**、**updateViewLayout**这三个方法来管理View，**WindowManager本质是接口，最终由WindowManagerImpl实现**

**延伸**

- WindowManager为每个Window创建Surface对象，然后应用就可以通过这个Surface来绘制任何它想要绘制的东西。而对于WindowManager来说，这只不过是一块矩形区域而已
  - **Surface**其实就是一个持有像素点矩阵的对象，这个像素点矩阵是组成显示在屏幕的图像的一部分。我们看到显示的每个**Window**（包括对话框、全屏的**Activity**、状态栏等）都有他自己绘制的**Surface**。而最终的显示可能存在**Window**之间遮挡的问题，此时就是通过**SurfaceFlinger**对象渲染最终的显示，使他们以正确的**Z-order**显示出来。一般**Surface**拥有一个或多个缓存（一般2个），通过双缓存来刷新，这样就可以一边绘制一边加新缓存。
- **View**是**Window**里面用于交互的**UI**元素。**Window**只**attach**一个**View Tree（组合模式）**，当**Window**需要重绘（如，当**View**调用**invalidate**）时，最终转为**Window**的**Surface**，**Surface**被锁住（**locked**）并返回**Canvas**对象，此时**View**拿到**Canvas**对象来绘制自己。当所有**View**绘制完成后，**Surface**解锁（**unlock**），并且**post**到绘制缓存用于绘制，通过**Surface Flinger**来组织各个**Window**，显示最终的整个屏幕

推荐文章：[Activity、View、Window的理解一篇文章就够了](https://links.jianshu.com/go?to=https%3A%2F%2Fblog.csdn.net%2Fzane402075316%2Farticle%2Fdetails%2F69822438)



#### 问：说说Activity生命周期？

答：

![img](https://developer.android.com/guide/components/images/activity_lifecycle.png?hl=zh-cn)

#### 问：ActivityA启动ActivityB两个Activity的变化？

答：`A:onResume()-->A:onPause() --> B:onCreate() -->B: onStart() --> B:onResume() --> A:onStop()`

ActivityB的开启必须经过A的onPause方法，所有 onPause方法尽量不要做耗时操作，影响下一个界面的开启。

#### 连问：ActivityB 按back键呢？

答：` B.onPause－>A.onRestart－>A.onStart－>A.onResume－>B.onStop－>B.onDestory`



#### 问：下拉状态栏是否会对Activity的生命周期又影响？dialog呢？Toast时呢？

答：下拉状态栏对生命周期没有任何影响，弹出AlertDialog、Toast时都没有影响，重新理解下onPause()，应该修正为“被Activity遮挡”。



#### 问：了解哪些Activity常用的标记位Flags？

- **FLAG_ACTIVITY_NEW_TASK :** 对应singleTask启动模式，其效果和在XML中指定该启动模式相同；
- **FLAG_ACTIVITY_SINGLE_TOP :** 对应singleTop启动模式，其效果和在XML中指定该启动模式相同；
- **FLAG_ACTIVITY_CLEAR_TOP :** 具有此标记位的Activity，当它启动时，在同一个任务栈中所有位于它上面的Activity都要出栈。这个标记位一般会和singleTask模式一起出现，在这种情况下，被启动Activity的实例如果已经存在，那么系统就会回调onNewIntent。如果被启动的Activity采用standard模式启动，那么它以及连同它之上的Activity都要出栈，系统会创建新的Activity实例并放入栈中；
- **FLAG_ACTIVITY_EXCLUDE_FROM_RECENTS :** 具有这个标记的 Activity 不会出现在历史 Activity 列表中；



#### 问：Activity的启动方式有几种？

答：参数要跳转的activity中定义的action名，这个action是在androidManifest.sml中定义 

- 显式跳转
  - `startActivity(Intent intent)`Intent通过class跳转
  - `startActivity(Intent intent)`Intent通过包类名跳转
  - `ComponentName`跳转
- 隐式启动
  - Intent参数中定义的action名，这个action是在androidManifest.sml中定义。

#### 问：有什么方法可以启动一个没有在AndroidManifest.xml中注册过的Activity？

答：通过Hook AMS，插件化技术原理，用一个已经注册过的Activity去欺骗AMS和PMS的检查，然后真正创建Ativity和启动的时机替换成真的Activity。



#### 问：在Activity中可以多次调用setContentView方法吗？说说不同时机第二次调用setContentView会发生什么？

答：setContentView方法所指定的View，只有在onCreate方法返回后才会显示在界面上。因此，如果调用了两次setContentView方法，只有最后一次才是有效的。



#### 问：Activity 四种启动模式常见使用场景？

- Standard
  - 标准模式，每次都新建一个实例对象
- SingleTask
  - 如果在任务栈中发现了相同的实例，将其上面的任务终止并移除，重用该实例并`onNewIntent()`。否则新建实例并入栈
- SingleTop
  - 如果在任务栈顶发现了相同的实例则重用并回调`onNewIntent()`，否则新建并压入栈顶
- SingleInstance 允许不同应用，进程线程等共用一个实例，无论从何应用调用该实例都重用
  - 以singleInstance模式启动的Activity具有全局唯一性，即整个系统中只会存在一个这样的实例
  - 以singleInstance模式启动的Activity具有独占性，即它会独自占用一个任务，被他开启的任何activity都会运行在其他任务中（官方文档上的描述为，singleInstance模式的Activity不允许其他Activity和它共存在一个任务中）
  - 被singleInstance模式的Activity开启的其他activity，能够开启一个新任务，但不一定开启新的任务，也可能在已有的一个任务中开启

#### 问：简单说说taskAffinity属性？

答：affinity是指Activity的归属，Activity与Task的吸附关系，也就是该Activity属于哪个Task。一般情况下在同一个应用中，启动的Activity都在同一个Task中，它们在该Task中度过自己的生命。每个Activity都有taskAffinity属性，这个属性指出了它希望进入的Task。如果一个Activity没有显式的指明taskAffinity，那么它的这个属性就等于Application指明的taskAffinity，如果Application也没有指明，那么该taskAffinity的值就等于应用的包名。我们可以通过在元素中增加taskAffinity属性来为某一个Activity指定单独的affinity。这个属性的值是一个字符串，可以指定为任意字符串，但是必须至少包含一个”.”，否则会报错。



#### 问：内存不足时，怎么保持Activity的一些状态，在哪个方法里面做具体操作？

答：在onSaveInstanceState方法中保存Activity的状态，在onRestoreInstanceState或onCreate方法中恢复Activity的状态。



#### 问：onSaveInstanceState()被执行的场景有哪些？

- 当用户按下HOME键时
- 长按HOME键，选择运行其他的程序时
- 锁屏时
- 从activity A中启动一个新的activity时
- 屏幕方向切换时



### 三、BroadcastReceiver系列

#### 问：

#### 问：如何判断当前BroadcastReceiver接受到的是有序广播还是无序广播？

答：在BroadcastReceiver类中onReceiver()方法中，可以调用boolean b=isOrderedBroadcast()；该方法是BroadcastReceiver类中提供的方法，用于告诉我们当前的接收到的广播是否为有序广播。



#### 问：如何在manifest和代码中如何注册和使用BroadcastReceiver？

答：**在清单文件中注册广播接收者称为静态注册，在代码中注册称为动态注册。**

静态注册的广播接收者只要app在系统中运行则一直可以接收到广播消息，动态注册的广播接收者当注册的Activity或者Service销毁了那么就接收不到广播了。

- 静态注册：在清单文件中进行如下配置：

```
<receiver android:name=".BroadcastReceiver1" >
<intent-filter>
<action android:name="android.intent.action.CALL" >
</action>
</intent-filter>
</receiver>
```

- 动态态注册：在代码中进行如下注册

```
receiver = new BroadcastReceiver(); 
IntentFilter intentFilter = new IntentFilter(); 
intentFilter.addAction(CALL_ACTION); 
context.registerReceiver(receiver， intentFilter);
```



### 四、Service系列

#### 问：谈一谈Service的生命周期？

答：Service的生命周期涉及到六大方法

- **onCreate()**：如果service没被创建过，调用startService()后会执行onCreate()回调；如果service已处于运行中，调用startService()不会执行onCreate()方法。也就是说，onCreate()只会在第一次创建service时候调用，多次执行startService()不会重复调用onCreate()，此方法适合完成一些初始化工作；
- **onStartComand()**：服务启动时调用，此方法适合完成一些数据加载工作，比如会在此处创建一个线程用于下载数据或播放音乐；
- **onBind()**：服务被绑定时调用；
- **onUnBind()**：服务被解绑时调用；
- **onDestroy()**：服务停止时调用；

#### 问：Service 默认在哪个线程中执行, service 里面是否能执行耗时的操作？

答：默认情况,如果没有显示的指 service 所运行的进程, Service 和 activity 是运 行在当前 app 所在进程的 main thread(UI 主线程)里面。

service 里面不能执行耗时的操作(网络请求,拷贝数据库,大文件 )

#### 问：Service有几种启动方式？区别在哪？

答：两/三种种启动方式：**startService**和**bindService**或者同时调用。

- startService：
  - Service首次启动，则先调用onCreate()，在调用onStartCommand()
  - Service已经启动，则直接调用onStartCommand()
  - 当调用stopSelf()或者stopService()后，会执行onDestroy(),代表Service生命周期结束。
  - **startService方式启动Service不会调用到onBind()。 startService可以多次调用，每次调用都会执行onStartCommand()。** 
  - **不管调用多少次startService，只需要调用一次stopService就结束。**
- bindService：如果该服务之前**还没创建**，系统回调顺序为onCreate()→onBind()。如果调用bindService()方法前服务**已经被绑定**，多次调用bindService()方法不会多次创建服务及绑定。如果调用者希望与正在绑定的服务**解除绑定**，可以调用unbindService()方法，回调顺序为onUnbind()→onDestroy()；
- 同时调用：startService和bindService同时开启，需要分别调用stopService和unbindService，Service才会走onDestroy()。



#### 问：onStartCommand的返回值一共有几种？

答：三种。默认返回START_STICKY。

- START_STICKY= 1:service所在进程被kill之后，系统会保留service状态为开始状态。系统尝试重启service，当服务被再次启动，传递过来的intent可能为null，需要注意。
- START_NOT_STICKY= 2:service所在进程被kill之后，系统不再重启服务
- START_REDELIVER_INTENT= 3:系统自动重启service，并传递之前的intent





#### 问：IntentService与Service的区别？

答：IntentService是Service的子类，是一个异步的，会自动停止的服务，很好解决了传统的Service中处理完耗时操作忘记停止并销毁Service的问题。

- 会创建独立的worker线程来处理所有的Intent请求；
- 会创建独立的worker线程来处理onHandleIntent()方法实现的代码，无需处理多线程问题；
- 所有请求处理完成后，IntentService会自动停止，无需调用stopSelf()方法停止Service；
- 为Service的onBind()提供默认实现，返回null；
- 为Service的onStartCommand提供默认实现，将请求Intent添加到队列中；
- IntentService不会阻塞UI线程，而普通Serveice会导致ANR异常
- Intentservice若未执行完成上一次的任务，将不会新开一个线程，是等待之前的任务完成后，再执行新的任务，等任务完成后再次调用stopSelf()



### Fragment相关

#### 问：Fragment 和 生命周期和 Activity 对比

#### 问：Fragment 之间如何进行通信

#### 问：Fragment 的 的 startActivityForResult

#### 问：Fragment 重叠问题

#### 问：Fragment回退栈管理

#### 问：Fragment与Activity通信

#### 问：Fragment与ActionBar和MenuItem

#### 问：没有布局的Fragment—保存大量数据

#### 问：DialogFragment的使用

#### 问：Fragment的startActivityForResult .........



### Service 相关

#### 问：进程保活



#### 问：Service 的运行线程（生命周期方法全部在主线程）



#### 问：Service 启动方式以及如何停止



#### 问：ServiceConnection 里面的回调方法运行在哪个线程？





### Handler相关

#### 问：Handler Looper Message 关系是什么？

#### 问：Messagequeue 的数据结构是什么？为什么要用这个数据结构？

#### 问： 如何在子线程中创建 Handler?

#### 问：Handler post 方法原理？

#### 问：Android 消息机制的原理及源码解析？

#### 问：Android 消息机制？



### Android 布局优化之 ViewStub、include、merge

#### 问：用 什么情况下使用 ViewStub 、include 、merge ？



#### 问：他们的原理是什么？



#### 问：布局优化神器 include 、merge 、ViewStub标签详解



## Android 多线程

### HandlerThread和Thread

