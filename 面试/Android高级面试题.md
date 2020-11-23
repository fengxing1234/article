---
title: Android高级面试题
date: 2020-11-10 22:51:11
tags:
---

1. ### Activity的管理机制

   

   #### 问：activity的启动流程？

   1. 先还是得当前系统中有没有拥有这个 Application 的进程。如果没有，则需要处理 APP 的启动过 程。在经过创建进程、绑定 Application 步骤后，才真正开始启动 Activity 的⽅法。 startActivity() ⽅ 法最终还是调⽤的 startActivityForResult()。
   2. 在 startActivityForResult() 中，真正去打开 Activity 的实现是在 Instrumentation 的 execStartActivivity() ⽅法中。
   3. 在 execStartActivity() 中采⽤ checkStartActivityResult() 检查在 manifest 中是否已经注册，如果没 有注册则抛出异常。否则把打开 Activity 的任务交给 ActivityThread 的内部类 ApplicationThread， 该类实现了 IApplicationThread 接⼝。这个类完全搞定了 onCreate()、onStart() 等 Activity 的⽣命 周期回调⽅法。
   4. 在 ApplicationThread 类中，有⼀个⽅法叫 scheduleLaunchActivity()，它可以构造⼀个 Activity 记 录，然后发送⼀个消息给事先定义好的 Handler。 这个 Handler 负责根据 LAUNCH_ACTIVITY 的类型来做不同的 Activity 启动⽅式。其中有⼀个᯿要的 ⽅法 handleLaunchActivity() 。
   5. 在 handleLaunchActivity() 中，会把启动 Activity 交给 performLaunchActivity() ⽅法。 在 performLaunchActivity() ⽅法中，⾸先从 Intent 中解析出⽬标 Activity 的启动参数，然后⽤ ClassLoader 将⽬标 Activity 的类通过类名加载出来并⽤ newInstance() 来实例化⼀个对象。 创建完毕后， 开始调⽤ Activity 的 onCreate() ⽅法，⾄此，Activity 被成功启动。

   ###  MVP MVVM原理和区别

   #### 问：介绍一下mvp与mvvm有什么区别及优缺点

   

   1. MVP：P是指Presenter，即实现者，功能与Controller类似。Presenter实质为Interface类的运用，用于降低M、V、C间的耦合度，主旨是为Activity或Fragment瘦身。MVP模式容易维护，可拆卸，可扩展，耦合性叫MVC较小，结构清晰。

      MVP的缺点，在于开发开销相对较大。与MVC相比，需要维护更多的接口。

   2. MVVM：MVVM由三部分组成，M，V，VM，分别代表Modle，View，ViewModle。谈论MVVM的前提是需要了解DataBinding，即Modle与View的进行绑定，包括单向绑定（Modle影响View），双向绑定（Modle与View相互影响）。

      MVVM的侧重点在于数据与UI的联动，自动更新，而非降低耦合度。对于耦合度的问题，其实还是需要结合MVP模式来解决。

      

   ### HTTP与HTTPS的关系

   

   #### 问：HTTP与HTTPS区别以及如何实现安全性？

   1. HTTP是明文传输，传输内容容易被篡改或者被窃取；HTTPS是密文传输，https相当于包装了SSL\TLS协议的HTTP。
   2. HTTPS需要申请CA证书，用于验证公钥这个证书收费，HTTP不用。主要通过非对称加密+对称加密+CA证书来保证请求安全的。

   ### Retrofit

   #### 问：简单介绍一下Retrofit

   1. Retrofit 是基于OKhttp网络请求框架的二次封装，本质是OKhttp。所以说Retrofit并不是一个网络框架、它只是一个网络框架封装。

      Android AsyncHttp 基于HttpClient ，已经停止维护，Android5.0之后不再使用HttpClient，不推荐应用。

      Volley 是google推出的基于HttpUrlConnection 的适合轻量级数据传输的网路库，不适合大文件的上传和下载。

      Retrofit优点：API设计简洁易用、注解化配置高度解耦、支持多种解析器、支持Rxjava。

   #### 问：说几个Retrofit中的注解

   1. @GET、@POST：确定请求方式、
   2. @Body：添加实体类对象、
   3. @Query:要传递的参数、
   4. @QueryMap:包含多个@Query注解参数

   ### 内存泄漏

   #### 问：如何解决Handler 引起的内存泄漏

   答：将Handler声明为静态内部类，就不会持有外部类SecondActivity的引用，其生命周期就和外部类无关，

   如果Handler里面需要context的话，可以通过弱引用方式引用外部类。

   #### 问：如何解决单例模式引起的内存泄漏

   答：使用的Context是ApplicationContext，由于ApplicationContext的生命周期是和app一致的，不会导致内存泄漏。

   ###  Apk 优化

   #### 问：一般如何去减小apk体积？

   1. 代码方面：统一三方库，删除无用代码。
   2. 资源方面：无用资源移除，资源混淆，适配主流手机。
   3. so：只保留Armeabi。

   ### 其他

   

   #### 问：SharedPreferences是如何保证线程安全的，其内部的实现用到了哪些锁？

   

   1. SharedPreferences的本质是用键值对的方式保存数据到xml文件，然后对文件进行读写操作。
   2. 对于读操作，加一把锁就够了：

   ```java
   public String getString(String key, @Nullable String defValue) {
       synchronized (mLock) {
           String v = (String)mMap.get(key);
           return v != null ? v : defValue;
       }
   }
   ```

   - 对于写操作，由于是两步操作，一个是editor.put，一个是commit或者apply所以其实是需要两把锁的：

     ```java
     //第一把锁，操作Editor类的map对象
     public final class EditorImpl implements Editor {
       @Override
       public Editor putString(String key, String value) {
           synchronized (mEditorLock) {
               mEditorMap.put(key, value);
               return this;
           }
       }
     }
     
     
     //第二把锁，操作文件的写入
     synchronized (mWritingToDiskLock) {
         writeToFile(mcr, isFromSyncCommit);
     }
     ```