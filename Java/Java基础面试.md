---
title: Java面试基础
date: 2020-04-30 23:03:36
tags: 
  - 面试
  - 基础
  - 设计模式
categories: 
  - 面试
---
# 面向对象

​		面向对象简称OO（Object Oriented），一种基于面向过程的新思想，该思想就是站在对象的角度思考问题，把多个功能合理的放在不同的对象中，强调的是具备某些功能的对象。

​		具备某种功能的实体，称之为对象。面向对象最小的程序单位是**类**。面向对象更加符合常规的思维方式**稳定性好，可重用性强，有良好的可重用性**。



## 对象

所谓对象就是真实世界的实体，对象与实体一一对应，也就是说现实世界中每一个实体都是一个对象，它是一种具体的概念。对象有以下特点：



## 面向对象的三大特性

java中面向对象的三大特性：封装、继承、多态。

### 封装

**隐藏内部实现，提供公共方法进行访问**。所谓封装就是把数据和操作数据的方法封装起来，对数据的访问和操作只能通过已定义的接口进行访问。

### 继承

子类拥有父类的特征和行为，并可以对父类进行扩展。形容类的关系。

### 多态

同一种类型的对象，执行相同方法，有不同的表现。

**多态的必要条件**

- 继承
- 重写
- 向上转型

**向上转型**

父类指向子类的引用

# JVM

## JVM组成

JVM由三个主要的子系统构成：

-  类加载子系统：负责加载类或者接口
- 运行时数据去（内存结构）：包含方法区、Java堆、Java栈、本地方法栈、指令计数器及其他隐含寄存器
- 执行引擎：负责执行包含在已装载的类或接口中的指令

### 类加载系统

https://zhuanlan.zhihu.com/p/54693308

#### 什么是类加载

答：类的加载指的是将类的.class文件中的二进制数据读入到内存中，将其放在运行时数据区的方法区内，然后在堆区创建一个java.lang.Class对象，用来封装类在方法区内的数据结构。类的加载的最终产品是位于堆区中的Class对象，Class对象封装了类在方法区内的数据结构，并且向Java程序员提供了访问方法区内的数据结构的接口。



#### 类的生命周期

类的生命周期包括加载、连接、初始化、使用和卸载。

- 加载：查找并加载类的二进制数据，并在堆内存中创建对象。
- 连接：连接又分为三步：验证、准备、解析
  - 验证：验证文件格式、元数据、字节码、符号引用。
  - 准备：为类的静态变量分配内存并初始化默认值。
  - 解析：把类中的符号引用转换为直接引用

- 初始化：为类的静态变量赋予正确的初始值
- 使用：new出对象程序中使用
- 卸载：执行垃圾回收



#### 类加载器

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190603164758648.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzI5OTY2MjAz,size_16,color_FFFFFF,t_70)

- 启动类加载器：Bootstrap ClassLoader，负责加载存放在JDK\jre\lib(JDK代表JDK的安装目录，下同)下，或被-Xbootclasspath参数指定的路径中的，并且能被虚拟机识别的类库
- 扩展类加载器：Extension ClassLoader，该加载器由sun.misc.Launcher$ExtClassLoader实现，它负责加载DK\jre\lib\ext目录中，或者由java.ext.dirs系统变量指定的路径中的所有类库（如javax.*开头的类），开发者可以直接使用扩展类加载器。
- 应用程序类加载器：Application ClassLoader，该类加载器由sun.misc.Launcher$AppClassLoader来实现，它负责加载用户类路径（ClassPath）所指定的类，开发者可以直接使用该类加载器

##### 双亲委派模型

双亲委派的意思是如果一个类加载器需要加载类，那么首先它会把这个类请求委派给父类加载器去完成，每一层都是如此。一直递归到顶层，当父加载器无法完成这个请求时，子类才会尝试去加载。这里的双亲其实就指的是父类。父类也不是我们平日所说的那种继承关系，只是调用逻辑是这样。双亲委派模型能保证基础类仅加载一次，不会让jvm中存在重名的类。



##### ExtClassLoader为什么没有设置parent？

因为BootstrapClassLoader是由c++实现的，所以并不存在一个Java的类。







### 内存结构

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190603165013168.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzI5OTY2MjAz,size_16,color_FFFFFF,t_70)

方法区和堆是**所有线程共享**的内存区域；而java栈、本地方法栈和程序计数器是**运行时线程私有**的内存区域。

JVM把内存划分成了如下几个区域：

- 堆区（Heap）：存放**通过new创建的对象实例**，容易OutOfMemoryEroor。

- 方法区（Method Area）：存放类的信息（字段和方法的字节码、构造函数、接口定义等）、静态变量，常量，运行时常量池。

- 虚拟机栈（VM Stack）：用于存储局部变量表、操作数栈、动态链接、方法出口等。只要线程已结束栈就出栈，生命周期与线程一致，其内存管理如下：
  - 方法中的基础数据类型直接在栈空间分配
  - 方法中的引用数据类型，需要new来创建，既在栈中分配空间，也在堆中分配对象，栈中的地址指向堆内对象
  - 方法的形式参数直接在栈空间分配，方法调用完毕在栈空间回收
  - 方法中的引用参数，在栈中分配空间指向堆中对象，方法执行完毕回收栈空间。
  - 字符串常量,static静态变量在方法区分配空间。

- 本地方法栈（Native Method Stack）：可理解为java中jni调用。用于支持native方法执行

- 程序计数器（Program Counter Register）：每个方法在运行时都存储着一个独立的程序计数器，程序计数器是指定程序运行的行数指针。



### Object o=new Object()



![img](https://pic4.zhimg.com/80/v2-eddc430b991c58039dfc79dd6f3139cc_1440w.jpg)







### 堆内存分区

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190604161655494.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzI5OTY2MjAz,size_16,color_FFFFFF,t_70)

内存分为:

- 新生代(**1/3堆空间**)
  - Eden(伊甸园区)
  - Survivor From（幸存者区）
  - Survivor To（幸存者区）
- 老年代(**2/3堆空间**)
- 持久代。(**直接内存JDK1.8后**)



#### 新生代



新生代中98%的对象都是”朝生夕死”的，所以并不需要按照1 : 1的比例来划分内存空间，而是将内存(新生代内存)分为一块较大的Eden(伊甸园)空间和两块较小的Survivor(幸存者)空间（8:1:1），每次使用Eden和其中一块Survivor（两个Survivor区域一个称为From区，另一个称为To区域）。

**当进行垃圾回收时，将Eden和Survivor中还存活的对象一次性复制到另一块Survivor空间上，最后清理掉Eden和刚才用过的Survivor空间。(复制算法)**

HotSpot实现的复制算法流程如下:

- 当Eden区满的时候,会触发第一次Minor gc,把还活着的对象拷贝到Survivor From区；当Eden区再次触发Minor gc的时候,会扫描Eden区和From区域,对两个区域进行垃圾回收,经过这次回收后还存活的对象,则直接复制到To区域,并将Eden和From区域清空。
- 当后续Eden又发生Minor gc的时候,会对Eden和To区域进行垃圾回收,存活的对象复制到From区域,并将Eden和To区域清空。

- 部分对象会在From和To区域中复制来复制去,如此交换15次(由JVM参数MaxTenuringThreshold决定,这个参数默认是15),最终如果还是存活,就存入到老年代。

也有例外出现，对于一些比较大的对象（需要分配一块比较大的连续内存空间）则直接进入到老年代。一般在Survivor 空间不足的情况下发生。



#### 老年代

在年轻代中经历了N次垃圾回收后仍然存活的对象，就会被放到年老代中。因此，可以认为年老代中存放的都是一些生命周期较长的对象。内存比新生代也大很多(大概比例是1:2)，当老年代内存满时触发Major GC即Full GC，Full GC发生频率比较低，老年代对象存活时间比较长，存活率标记高。一般来说，大对象会被直接分配到老年代。所谓的大对象是指需要大量连续存储空间的对象。



#### 元数据

不属于堆内存，属于内存空间。真正与堆隔离。方法区是类逻辑上的一个抽象模板，而元空间是**方法区的实现**，是真实存在的内存。



#### 垃圾回收

- 虚拟机在进行MinorGC（新生代的GC）的时候，会判断要进入OldGeneration区域对象的大小，是否大于Old Generation剩余空间大小，如果大于就会发生Full GC。
- 刚分配对象在Eden中，如果空间不足尝试进行GC，回收空间，如果进行了MinorGC空间依旧不够就放入Old Generation，如果OldGeneration空间还不够就OOM了。
- 比较大的对象，数组等，大于某值（可配置）就直接分配到老年代，（避免频繁内存拷贝）
- 年轻代和年老代属于Heap空间的，Permanent Generation（永久代）可以理解成方法区，（它属于方法区）也有可能发生GC，例如类的实例对象全部被GC了，同时它的类加载器也被GC掉了，这个时候就会触发永久代中对象的GC。
- 如果OldGeneration满了就会产生FullGC。老年代满原因：
  - 1、from survive中对象的生命周期到一定阈值
  - 2、分配的对象直接是大对象
  - 3、由于To 空间不够，进行GC直接把对象拷贝到年老代（年老代GC时候采用不同的算法）

- 如果Young Generation大小分配不合理或空间比较小，这个时候导致对象很容易进入Old Generation中，而Old Generation中回收具体对象的时候速度是远远低于Young Generation回收速度。

  

#### 对象的分配

- 对象优先分配在Eden区，如果Eden区没有足够的空间时，虚拟机执行一次Minor GC。
- 大对象直接进入老年代（大对象是指需要大量连续内存空间的对象）。这样做的目的是避免在Eden区和两个Survivor区之间发生大量的内存拷贝（新生代采用复制算法收集内存）。
- 长期存活的对象进入老年代。虚拟机为每个对象定义了一个年龄计数器，如果对象经过了1次Minor GC那么对象会进入Survivor区，之后每经过一次Minor GC那么对象的年龄加1，直到达到阀值对象进入老年区。
- 动态判断对象的年龄。如果Survivor区中相同年龄的所有对象大小的总和大于Survivor空间的一半，年龄大于或等于该年龄的对象可以直接进入老年代。
- 空间分配担保。每次进行Minor GC时，JVM会计算Survivor区移至老年区的对象的平均大小，如果这个值大于老年区的剩余值大小则进行一次Full GC，如果小于检查HandlePromotionFailure设置，如果true则只进行Monitor GC,如果false则进行Full GC。

# Java中的堆栈

**栈内存**

在函数中定义的基本类型的变量和对象的引用变量都是在函数的栈内存中分配。

当在一段代码块中声明了一个变量时，java就会在栈内存中为这个变量分配内存空间，当超过变量的作用域之后，java也会自动释放为该变量分配的空间，而这个回收的空间可以即刻用作他用。

**堆内存**

堆内存用于存放由new创建的对象和数组。

在堆内存中分配的内存空间，由java虚拟机自动垃圾回收器来管理。在堆中产生了一个数组或者对象后，还可以在栈中定义一个特殊的变量，变量的值就等于数组或对象在堆内存中的首地址，而这个栈中的特殊变量，也就成为数组或对象的引用变量。以后可以在程序中使用栈内存中的引用变量访问堆内存中的数组或对象了。引用变量相当于是为数组或对象起的一个别名，或者是代号。

数组和对象在没有引用变量指向它的时候，才变成垃圾，不能被继续使用，但是仍然会占用堆内存空间，而后在一个不确定的时间内，由java虚拟机自动垃圾回收器回收，这也是java程序为什么会占用很大内存的原因。


# GC算法 垃圾回收

- 垃圾：无任何对象引用的对象。
- 回收：清理“垃圾”占用的内存空间而非对象本身。
- 发生地点：一般发生在堆内存中，因为大部分的对象都储存在堆内存中。
- 发生时间：程序空闲时间不定时回收。



## 对象的生命周期

### 创建阶段(Created)

在创建阶段系统通过下面的几个步骤来完成对象的创建过程：

- 为对象分配存储空间
  - 开始构造对象
  - 从超类到子类对static成员进行初始化
  - 超累成员变量按顺序初始化，递归调用超累的构造方法
  - 子类成员变量按顺序初始化，子类构造方法调用

- 应用阶段(In Use)：对象至少被一个强引用持有。
- 不可见阶段(Invisible)：简单说就是程序的执行已经超出了该对象的作用域了。
- 不可达阶段(Unreachable)：对象处于不可达阶段是指该对象不再被任何强引用所持有。
- 收集阶段(Collected)：
- 当垃圾回收器发现该对象已经处于“不可达阶段”并且垃圾回收器已经对该对象的内存空间重新分配做好准备时，则对象进入了“收集阶段”。如果该对象已经重写了finalize()方法，则会去执行该方法的终端操作。

- 终结阶段(Finalized)：当对象执行完finalize()方法后仍然处于不可达状态时，则该对象进入终结阶段。在该阶段是等待垃圾回收器对该对象空间进行回收。
- 对象空间重分配阶段(De-allocated)：垃圾回收器对该对象的所占用的内存空间进行回收或者再分配了，则该对象彻底消失了，称之为“对象空间重新分配阶段”。



## 判断对象是否是垃圾算法



#### 引用计数算法

堆中每个对象（不是引用）都有一个引用计数器。当一个对象被创建并初始化赋值后，该变量计数设置为1。每当有一个地方引用它时，计数器值就加1（a = b， b被引用，则b引用的对象计数+1）。当引用失效时（一个对象的某个引用超过了生命周期（出作用域后）或者被设置为一个新值时），计数器值就减1。任何引用计数为0的对象可以被当作垃圾收集。当一个对象被垃圾收集时，它引用的任何对象计数减1。



#### 根搜索算法（可达性分析算法）

![image.png](https://upload-images.jianshu.io/upload_images/4118241-bdc59e28c5775caf.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

从一个GC ROOT节点开始，寻找对应的引用节点，找到这个节点后，继续寻找这个节点的引用节点。当所有的引用节点寻找完毕后，剩余的节点则被认为是没有被引用到的节点，即无用的节点。



**所谓的GC根对象包括：**

- 虚拟机栈中引用的对象（本地变量表）
- 方法区中静态属性引用的对象
- 方法区中常亮引用的对象
- 本地方法栈中引用的对象（Native对象）

## GC算法

GC最基础的算法有三种：标记 -清除算法、复制算法、标记-整理算法，我们常用的垃圾回收器一般都采用分代收集算法。

### 标记 -清除算法

标记清除算法是最基础的收集算法，其他收集算法都是基于这种思想。标记清除算法分为“标记”和“清除”两个阶段：首先标记出需要回收的对象（**这一过程在可达性分析过程中进行**），标记完成之后统一清除对象。

**它的主要缺点：**

- 标记和清除过程效率不高 。
- 标记清除之后会产生大量不连续的内存碎片。



![image.png](https://upload-images.jianshu.io/upload_images/4118241-ab33b8257a3ed8bd.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 标记-整理算法

标记整理，标记操作和“标记-清除”算法一致，后续操作不只是直接清理对象，而是在清理无用对象完成后让所有存活的对象都向一端移动，并更新引用其对象的指针。

主要缺点：在标记-清除的基础上还需进行对象的移动，成本相对较高，好处则是不会产生内存碎片。

![image.png](https://upload-images.jianshu.io/upload_images/4118241-e02d7d3ebf1d75af.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 复制算法(新生代算法)



复制算法，它将可用内存容量划分为大小相等的两块，每次只使用其中的一块。当这一块用完之后，就将还存活的对象复制到另外一块上面，然后在把已使用过的内存空间一次理掉。这样使得每次都是对其中的一块进行内存回收，不会产生碎片等情况，只要移动堆订的指针，按顺序分配内存即可，实现简单，运行高效。



![image.png](https://upload-images.jianshu.io/upload_images/4118241-98d9dff8f994e8bf.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 垃圾回收器

Java中Stop-The-World机制简称STW，是在执行垃圾收集算法时，Java应用程序的其他所有线程都被挂起（除了垃圾收集帮助器之外）。Java中一种全局暂停现象，全局停顿，所有Java代码停止，native代码可以执行，但不能与JVM交互；这些现象多半是由于gc引起。

### 串行垃圾回收器

串行垃圾回收器通过持有应用程序所有的线程进行工作。它为单线程环境设计，只使用一个单独的线程进行垃圾回收，通过冻结所有应用程序线程进行工作，所以可能不适合服务器环境。它最适合的是简单的命令行程序（单CPU、新生代空间较小及对暂停时间要求不是非常高的应用）。是client级别默认的GC方式。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190605151624561.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzI5OTY2MjAz,size_16,color_FFFFFF,t_70)

### 并行垃圾回收器

并行垃圾回收器也叫做 throughput collector 。它是JVM的默认垃圾回收器。与串行垃圾回收器不同，它使用多线程进行垃圾回收。相似的是，当执行垃圾回收的时候它也会冻结所有的应用程序线程。

### 并发标记扫描垃圾回收器

并发标记垃圾回收使用多线程扫描堆内存，标记需要清理的实例并且清理被标记过的实例。并发标记垃圾回收器只会在下面两种情况持有应用程序所有线程。
（1）当标记的引用对象在Tenured区域；
（2）在进行垃圾回收的时候，堆内存的数据被并发的改变。
相比并行垃圾回收器，并发标记扫描垃圾回收器使用更多的CPU来确保程序的吞吐量。如果我们可以为了更好的程序性能分配更多的CPU，那么并发标记上扫描垃圾回收器是更好的选择相比并发垃圾回收器。

## GC调优

JVM调优，调的是什么？
每一次Full GC都会使JVM停止运行–>使Full GC不执行，使Minor GC尽可能少地执行

## GC日志分析

摘录GC日志一部分（前部分为年轻代gc回收；后部分为full gc回收）：

```
2016-07-05T10:43:18.093+0800: 25.395: [GC [PSYoungGen: 274931K->10738K(274944K)] 371093K->147186K(450048K), 0.0668480 secs] [Times: user=0.17 sys=0.08, real=0.07 secs]

2016-07-05T10:43:18.160+0800: 25.462: [Full GC [PSYoungGen: 10738K->0K(274944K)] [ParOldGen: 136447K->140379K(302592K)] 147186K->140379K(577536K) [PSPermGen: 85411K->85376K(171008K)], 0.6763541 secs] [Times: user=1.75 sys=0.02, real=0.68 secs]	
```

通过上面日志分析得出，PSYoungGen、ParOldGen、PSPermGen属于Parallel收集器。其中PSYoungGen表示gc回收前后年轻代的内存变化；ParOldGen表示gc回收前后老年代的内存变化；PSPermGen表示gc回收前后永久区的内存变化。young gc 主要是针对年轻代进行内存回收比较频繁，耗时短；full gc 会对整个堆内存进行回城，耗时长，因此一般尽量减少full gc的次数

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190603181024771.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzI5OTY2MjAz,size_16,color_FFFFFF,t_70)

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190603181041398.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzI5OTY2MjAz,size_16,color_FFFFFF,t_70)

### 减少GC开销的措施

- **不要显式调用System.gc()**：此函数建议JVM进行主GC,虽然只是建议而非一定,但很多情况下它会触发主GC,从而增加主GC的频率,也即增加了间歇性停顿的次数。
- **尽量减少临时对象的使用**：临时对象在跳出函数调用后,会成为垃圾,少用临时变量就相当于减少了垃圾的产生,从而延长了出现上述第二个触发条件出现的时间,减少了主GC的机会。
- **对象不用时最好显式置为Null**：一般而言,为Null的对象都会被作为垃圾处理,所以将不用的对象显式地设为Null,有利于GC收集器判定垃圾,从而提高了GC的效率。
- **尽量使用StringBuffer,而不用String来累加字符串**：由于String是固定长的字符串对象,累加String对象时,并非在一个String对象中扩增,而是重新创建新的String对象,如Str5=Str1+Str2+Str3+Str4,这条语句执行过程中会产生多个垃圾对象,因为对次作“+”操作时都必须创建新的String对象,但这些过渡对象对系统来说是没有实际意义的,只会增加更多的垃圾。避免这种情况可以改用StringBuffer来累加字符串,因StringBuffer是可变长的,它在原有基础上进行扩增,不会产生中间对象。
- **能用基本类型如Int,Long,就不用Integer,Long对象**：基本类型变量占用的内存资源比相应对象占用的少得多,如果没有必要,最好使用基本变量。
- **尽量少用静态对象变量**
- 静态变量属于全局变量,不会被GC回收,它们会一直占用内存。
- **分散对象创建或删除的时间**集中在短时间内大量创建新对象,特别是大对象,会导致突然需要大量内存,JVM在面临这种情况时,只能进行主GC,以回收内存或整合内存碎片,从而增加主GC的频率。集中删除对象,道理也是一样的。它使得突然出现了大量的垃圾对象,空闲空间必然减少,从而大大增加了下一次创建新对象时强制主GC的机会。

## Java代码编译和执行整个过程

开发人员编写Java代码(.java文件)，然后将之编译成字节码(.class文件)，再然后字节码被类加载器装入内存，一旦字节码进入虚拟机，它就会被解释器（执行引擎）解释执行。

**步骤1：Java代码编译是由Java源码编译器来完成，也就是Java代码到JVM字节码（.class文件）的过程。**

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190605130338211.png)

**步骤2：Java字节码的执行是由JVM执行引擎来完成，流程图如下所示：**

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190605130355535.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzI5OTY2MjAz,size_16,color_FFFFFF,t_70)

Java代码编译和执行的整个过程包含了三个重要机制：

- Java源码编译机制
- 类加载机制
- 类执行机制



## Java虚拟机和Dalvik虚拟机区别

- java虚拟机运行的是Java字节码，Dalvik虚拟机运行的是Dalvik字节码；传统的Java程序经过编译，生成Java字节码保存在class文件中，java虚拟机通过解码class文件中的内容来运行程序。而Dalvik虚拟机运行的是Dalvik字节码，所有的Dalvik字节码由Java字节码转换而来，并被打包到一个DEX(Dalvik Executable)可执行文件中Dalvik虚拟机通过解释Dex文件来执行这些字节码。
- Dalvik可执行文件体积更小。SDK中有一个叫dx的工具负责将java字节码转换为Dalvik字节码。
- java虚拟机与Dalvik虚拟机架构不同。java虚拟机基于栈架构。程序在运行时虚拟机需要频繁的从栈上读取或写入数据。这过程需要更多的指令分派与内存访问次数，会耗费不少CPU时间，对于像手机设备资源有限的设备来说，这是相当大的一笔开销。Dalvik虚拟机基于寄存器架构，数据的访问通过寄存器间直接传递，这样的访问方式比基于栈方式快的多.




# 基本数据类型和引用数据类型

**基本数据类型**

- byte
- short
- int
- long
- char
- float
- double
- boolean

**引用数据类型**

- 对象
- 数组

![img](https://img-blog.csdn.net/20140531091306906)

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190526183106748.jpeg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzI5OTY2MjAz,size_16,color_FFFFFF,t_70)

**区别:**

- 基本数据类型保存的是值，引用类型保存的是内存地址
- 基本数据类型存储在栈中，引用数据类型首先在栈上给引用分配内存空间，对象的具体信息存储在堆中。
- 基本数据类型的赋值其实是创建新的拷贝，而引用类型的赋值是传递引用。
- 基本数据类型的==其实是比较值，而引用数据类型比较的是地址（引用）。
- 一个方法不能修改一个基本类型的参数；一个方法可以修改引用类型参数中对象所指向的值。

[数据类型分析](https://blog.csdn.net/javazejian/article/details/51192130?utm_medium=distribute.pc_relevant.none-task-blog-OPENSEARCH-5&depth_1-utm_source=distribute.pc_relevant.none-task-blog-OPENSEARCH-5)



**int与Integer不同**

- Integer是int的包装类，int则是java的一种基本数据类型
- Integer变量必须实例化后才能使用，而int变量不需要
- Integer实际是对象的引用，当new一个Integer时，实际上是生成一个指针指向此对象；而int则是直接存储数据值
- Integer的默认值是null，int的默认值是0



**延伸：关于Integer和int的比较**

- 由于Integer变量实际上是对一个Integer对象的引用，所以两个通过new生成的Integer变量永远是不相等的（因为new生成的是两个对象，其内存地址不同）。

  ```
  Integer i = new Integer(100);
  Integer j = new Integer(100);
  System.out.print(i == j); //false
  ```

- Integer变量和int变量比较时，只要两个变量的值是相等的，则结果为true（因为包装类Integer和基本数据类型int比较时，java会自动拆包装为int，然后进行比较，实际上就变为两个int变量的比较）

  ```
  Integer i = new Integer(100);
  int j = 100；
  System.out.print(i == j); //true
  ```

- 非new生成的Integer变量和new Integer()生成的变量比较时，结果为false。（因为非new生成的Integer变量指向的是java常量池中的对象，而new Integer()生成的变量指向堆中新建的对象，两者在内存中的地址不同）

  ```
  Integer i = new Integer(100);
  Integer j = 100;
  System.out.print(i == j); //false
  ```

- 对于两个非new生成的Integer对象，进行比较时，如果两个变量的值在区间-128到127之间，则比较结果为true，如果两个变量的值不在此区间，则比较结果为false

  ```
  Integer i = 100;
  Integer j = 100;
  System.out.print(i == j); //true
  Integer i = 128;
  Integer j = 128;
  System.out.print(i == j); //false
  ```

  >原因：
  >java在编译Integer i = 100 ;时，会翻译成为Integer i = Integer.valueOf(100)；，而java API中对Integer类型的valueOf的定义如下：
  >
  >```
  >public static Integer valueOf(int i){
  >    assert IntegerCache.high >= 127;
  >    if (i >= IntegerCache.low && i <= IntegerCache.high){
  >        return IntegerCache.cache[i + (-IntegerCache.low)];
  >    }
  >    return new Integer(i);
  >}
  >```
  >
  

## 数组与链表的区别

- 数组从栈中分配空间，对程序员方便快速，自由度小。
- 链表从堆中分配内存。自由度大但申请管理比较麻烦。
- 数组必须事先定义固定的长度（元素个数），不能适应数据动态地增减情况（数据插入、删除比较麻烦）。当数据增加时，可能会超出数组的最大空间（越界）；当数据减少时，造成内存浪费。
- 链表动态地进行存储分配，可以适应数据动态地增减情况（数据插入删除简单）（数组中插入、删除数据项时，需要移动其他项），但链表查找元素时需要遍历整个链表。

# Java自动装箱与拆箱

## 什么是自动装箱、拆箱

装箱就是 自动将基本数据类型转换为包装器类型；拆箱就是 自动将包装器类型转换为基本数据类型。

```java
Integer i = 10;  //装箱
int n = i;   //拆箱
```

## 自动装箱的原理及使用场景

基本数据类型–>封装类

```
    Integer i = 10;
```

执行时实际上系统执行了

```
    Integer i = Integer.valueOf(10);
```

分析Integer的valueOf源码：

```
public static Integer valueOf(int i) {
    if(i >= -128 && i <= IntegerCache.high)　　// 没有设置的话，IngegerCache.high 默认是127
        return IntegerCache.cache[i + 128];
    else
        return new Integer(i);
}
```

对于–128到127（默认是127）之间的值，Integer.valueOf(int i) 返回的是缓存的Integer对象（并不是新建对象），而其他值，执行Integer.valueOf(int i) 返回的是一个新建的 Integer对象。装箱的过程会创建对应的对象，这个会消耗内存，所以装箱的过程会增加内存的消耗，影响性能。



## 自动拆箱的原理及使用场景

封装类–>基本数据类型

```
Integer i = 10; //装箱 
int t = i; //拆箱，实际上执行了 int t = i.intValue();
```

进行运算时也可以进行拆箱

```
Integer i = 10; 
System.out.println(i++);
```

Integer与int运算

```
int i = 10;
Integer integer1 = new Integer(10); 
System.out.println(i==integer1);//true,integer1自动拆箱
```

intValue函数很简单，直接返回value值即可

```
@Override
 public int intValue() {
    return value;
}
```





# String 类

- final类说明是一个不可变对象
- 内部使用value[]数组实现
- 声明int类型hash



**==与equals()**

答：

- ==
  - 若操作数的类型是基本数据类型，则该关系操作符判断的是左右两边操作数的值是否相等
  - 若操作数的类型是引用数据类型，则该关系操作符判断的是左右两边操作数的内存地址是否相同。		

- equals
  - 在Object类中，equals方法是用来比较两个对象的引用是否相等，即是否指向同一个对象。
  - 但是一般像String就重写了equels方法。



**定义为String类型的st1和st2是否相等，为什么**

答：常量池个特点，如果发现已经存在，就不在创建重复的对象

```
package string;
 
public class Demo2_String {
   public static void main(String[] args) {
     String st1 = "abc";
     String st2 = "abc";
     System.out.println(st1 == st2);
     System.out.println(st1.equals(st2)); 
   }
}
```

输出结果:

```
第一行：true

第二行：true
```

![img](https://img-blog.csdn.net/20180409133349472?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTE1NDE5NDY=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)



内存过程：

1. 运行编译，当前类class文件加载进方法区
2. 方法main压栈
3. 常量池创建“abc”对象，产生一个内存地址
4. 然后把内存地址执行st1，st1根据内存地址，指向了常量池中的“abc”。
5. 当代码执行`Stringst2 =”abc”`,发现常量池中存在"abc"对象所以不会创建对象，直接把"abc"内存地址复制给st2。
6. 最后st1和st2都指向了内存中同一个地址，所以两者是完全相同的。



**下面这句话在内存中创建了几个对象**

`String st1 = new String(“abc”);`

答：两个对象。一个是常量池“abc”对象，一个是堆内存String对象。

![img](https://img-blog.csdn.net/20180409133238683?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTE1NDE5NDY=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)



**判定以下定义为String类型的st1和st2是否相等**

```
package string;
public class Demo2_String {
   public static void main(String[] args) {
     String st1 = new String("abc");
     String st2 = "abc";
     System.out.println(st1 == st2);
     System.out.println(st1.equals(st2));
   }
}
```

答：false true

- ==比较的是地址值，st1指向堆内存中的string对象，st2指向常量池的‘abc’对象。
- equals默认也是比较地址值，但是String类中重写了此方法，实际比较的是值。（首先比较地址）

```
public boolean equals(Object anObject) {
        if (this == anObject) {
            return true;
        }
        if (anObject instanceof String) {
            String anotherString = (String)anObject;
            int n = length();
            if (n == anotherString.length()) {
                int i = 0;
                while (n-- != 0) {
                    if (charAt(i) != anotherString.charAt(i))
                            return false;
                    i++;
                }
                return true;
            }
        }
        return false;
    }
```



**判定以下定义为String类型的st1和st2是否相等**

```
package string;
 
public class Demo2_String {
 
   public static void main(String[] args) {
     String st1 = "a" + "b" + "c";
     String st2 = "abc";
     System.out.println(st1 == st2);
     System.out.println(st1.equals(st2));
   }
}
```

答：true 和 true

> “a”,”b”,”c”三个本来就是字符串常量，进行+符号拼接之后变成了“abc”，“abc”本身就是字符串常量（Java中有常量优化机制），所以常量池立马会创建一个“abc”的字符串常量对象，在进行st2=”abc”,这个时候，常量池存在“abc”，所以不再创建。所以，不管比较内存地址还是比较字符串序列，都相等。




**判断以下st2和st3是否相等**

```
package string;
 
public class Demo2_String {
 
   public static void main(String[] args) {
     String st1 = "ab";
     String st2 = "abc";
     String st3 = st1 + "c";
     System.out.println(st2 == st3);
     System.out.println(st2.equals(st3));
   }
}
```

答案：false 和 true

> s1和s2都会在常量池中创建对象，st3在编译时期不能确定st3的值，只能在运行期间确定`st3="abc"`,在运行期间就只能放在堆内存中。所以不是同一个引用。







# String、StringBuffer和StringBuilder的区别

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190526180828781.jpeg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzI5OTY2MjAz,size_16,color_FFFFFF,t_70)



# 不可变对象

不可变对象即对象一旦创建它的状态就不能发生改变（对象的数据，属性值），反之即为可变对象。

不可变对象的类即为不可变类，String、基本类型的包装类、BigInteger和BigDecimal等都是不可变类。



**不可变的对象必须满足的条件**

- 将类声明为final
- 声明属性为private 和 final
- 不要提供任何会修改对象状态的方法



**改变"不可变对象"**

现在我们已经知道了String的成员变量是private final 的，也就是初始化之后不可改变的。同时也提到value这个成员变量其实也是一个引用，指向真正的数组内存地址，不能改变它的引用指向，我们能不能直接改变内存数组中的数据呢，那么就需要获取到value，而value是私有的，可用反射获取



# transient关键字

java 的transient关键字为我们提供了便利，你只需要实现Serilizable接口，将不需要序列化的属性前添加关键字transient，序列化对象的时候，这个属性就不会序列化到指定的目的地中。

- 一旦变量被transient修饰，变量将不再是对象持久化的一部分，该变量内容在序列化后无法获得访问。
- transient关键字只能修饰变量，而不能修饰方法和类。注意，本地变量是不能被transient关键字修饰的。变量如果是用户自定义类变量，则该类需要实现Serializable接口。
- 被transient关键字修饰的变量不再能被序列化，一个静态变量不管是否被transient修饰，均不能被序列化。



# 序列化



对象序列化是一个用于将对象状态转换为字节流的过程，可以将其保存到磁盘文件中或通过网络发送到任何其他程序；从字节流创建对象的相反的过程称为反序列化。

序列化的作用就是为了保存java的类对象的状态，并将对象转换成可存储或者可传输的状态，用于不同jvm之间进行类实例间的共享。



## 序列化方式

实现序列化的两种方式：

- 实现Serializable接口：Java专用
- 实现Parcelable接口：Android专用

### Serializable 和Parcelable的对比

Android上应该尽量采用Parcelable，效率至上,Parcelable的速度比高十倍以上。

**Serializable**的迷人之处在于你只需要对某个类以及它的属性实现Serializable 接口即可。Serializable 接口是一种标识接口（marker interface），这意味着无需实现方法，Java便会对这个对象进行高效的序列化操作。这种方法的缺点是使用了反射，序列化的过程较慢。这种机制会在序列化的时候创建许多的临时对象，容易触发垃圾回收。

**Parcelable**方式的实现原理是将一个完整的对象进行分解，而分解后的每一部分都是Intent所支持的数据类型，这样也就实现传递对象的功能了



# 内部类

定义在类内部的类就被称为内部类。外部类按常规的类访问方式使用内部类，唯一的差别是内部类可以访问外部类的所有方法与属性，包括私有方法与属性。



## 静态内部类

定义在类内部的静态类

```
public class Out {
    private static int a;
    private int b;

    public static class Inner {
        public void print() {
            System.out.println(a);
        }
    }
}
```

Inner是静态内部类。静态内部类可以访问外部类所有静态变量和方法。静态内部类和一般类一致，可以定义静态变量、方法，构造方法等。

```
Out.Inner inner = new Out.Inner();
inner.print();
```



### 成员内部类

定义在类内部的非静态类称为成员内部类。

```
public class Out {
    private static int a;
    private int b;

    public class Inner {
        public void print() {
            System.out.println(a);
            System.out.println(b);
        }
    }
}
```

成员内部类可以访问外部类所有的变量和方法，包括静态和实例，私有和非私有。和静态内部类不同的是，每一个成员内部类的实例都依赖一个外部类的实例（成员内部类是依附外部类而存在的）。其它类使用内部类必须要先创建一个外部类的实例。

```
Out out = new Out();
Out.Inner inner = out.new Inner();
inner.print();
```

- 成员内部类不能定义静态方法和变量（final修饰的除外）。这是因为成员内部类是非静态的，类初始化的时候先初始化静态成员，如果允许成员内部类定义静态变量，那么成员内部类的静态变量初始化顺序是有歧义的。
- 成员内部类是依附于外围类的，所以只有先创建了外围类才能够创建内部类。
- 成员内部类与外部类可以拥有同名的成员变量或方法，默认情况下访问的是成员内部类的成员。如果要外部类的同名成员，需用下面的形式访问：

```
OutterClass(外部类).this.成员
```



**为什么Java中成员内部类可以访问外部类成员？**

- 成员内部类的创建需要外部类的对象
- 内部类对象持有指向外部类对象的引用。



**静态内部类与成员内部类对比**

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190528190801750.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzI5OTY2MjAz,size_16,color_FFFFFF,t_70)





### 局部内部类&闭包

定义在外部类方法中的类，叫局部类

```
public class Out {
    private static int a;
    private int b;

    public void test(final int c) {
        final int d = 1;
        class Inner {
            public void print() {
                System.out.println(a);
                System.out.println(b);
                System.out.println(c);
                System.out.println(d);
            }
        }
    }

    public static void testStatic(final int c) {
        final int d = 1;
        class Inner {
            public void print() {
                System.out.println(a);
                //定义在静态方法中的局部类不可以访问外部类的实例变量
                //System.out.println(b);
                System.out.println(c);
                System.out.println(d);
            }
        }
    }
}
```

局部类只能在定义该局部类的方法中使用。定义在实例方法中的局部类可以访问外部类的所有变量和方法，定义在静态方法中的局部类只能访问外部类的静态变量和方法。同时局部类还可以访问方法的参数和方法中的局部变量，这些参数和变量必须要声明为final的。否则会报错



### 匿名内部类

```
public class Out {
    private static int a;
    private int b;

    private Object obj = new Object() {
        private String name = "匿名内部类";
        @Override
        public String toString() {
            return name;
        }
    };

    public void test() {
        Object obj = new Object() {
            @Override
            public String toString() {
                System.out.println(b);
                return String.valueOf(a);
            }
        };
        System.out.println(obj.toString());
    }
}
```

匿名内部类可以访问外部类所有的变量和方法。



### 内部类特点



- 非静态内部类对象不仅指向该内部类，还指向实例化该内部类的外部类对象的内存。
- 内部类和普通类一样可以重写Object类的方法，如toString方法；并且有构造函数，执行顺序依旧是先初始化属性，再执行构造函数
- 在编译完之后，会出现（外部类.class）和（外部类﹩内部类.class）两个类文件名。
- 内部类可以被修饰为private，只能被外部类所访问。事实上一般也都是如此书写。
- 内部类可以被写在外部类的任意位置，如成员位置，方法内。



#### 内部类访问外部类

- 静态时，静态内部类只能访问外部类静态成员;非静态内部类都可以直接访问。（原因是：内部类有一个外部类名.this的指引）当访问外部类静态成员出现重名时，通过(外部类名.静态成员变量名)访问。如，Out.show();
- 重名情况下，非静态时，内部类访问自己内部类通过this.变量名。访问外部类通过（外部类名.this.变量名）访问 。如Out.this.show();
- 在没有重名的情况下，无论静态非静态，内部类直接通过变量名访问外部成员变量。

#### 外部类访问内部类

- 内部类为非静态时，外部类访问内部类，必须建立内部类对象。建立对象方法，如前所述。
- 内部类为静态时，外部类访问非静态成员，通过（外部类对象名.内部类名.方法名）访问，如new Out().In.function();
- 内部类为静态时，外部类访问静态成员时，直接通过（外部类名.内部类名.方法名），如 Out.In.funchtion();
- 当内部类中定义了静态成员时，内部类必须是静态的；当外部静态方法访问内部类时，内部类也必须是静态的才能访问。



# 静态（static）



把一个变量声明为静态变量通常基于以下三个目的：

- 作为共享变量
- 减少对象的创建
- 保留唯一副本

## 静态变量

- 静态变量在内存中只有一份拷贝，JVM只为静态分配一次内存，在类加载的过程中完成静态变量的内存分配。可用类名直接访问（方便），当然也可以通过对象来访问（但是这是不推荐的）。
- 实例变量，每次创建实例，都会对实例变量分配一次内存，实例变量可以在内存中有多个拷贝，互不影响（灵活）。

### 静态变量与实例变量区别

- 所属不同，静态变量属于类，普通成员变量所有当前对象。
- 存储区域不同，静态变量存在方法区，普通成员变量存在堆（成员变量存储在堆中的对象里面）。
- 生命周期不同，静态变量生周期与类相同；普通成员变量与当前对象相同。
- 序列化时，静态变量会被拆除在外。

### 静态方法

静态方法可以直接使用类名调用，静态方法中不能使用this、super，不能直接访问成员变量，成员方法，只能访问类下的静态变量、静态方法，因为实例成员和实例方法与当前对象关联，静态属于类。静态方法不能被抽象。



### 静态代码块

static代码块也叫静态代码块，是在类中独立于类成员的static语句块，可以有多个，位置可以随便放，它不在任何的方法体内，JVM加载类时会执行这些静态的代码块，如果static代码块有多个，JVM将按照它们在类中出现的先后顺序依次执行它们，每个代码块只会被执行一次。



### 静态方法和静态代码块的区别

- 如果有些代码必须在项目启动的时候就执行,就需要使用静态代码块,这种代码是主动执行的；
- 需要在项目启动的时候就初始化但是不执行,在不创建对象的情况下,可以供其他程序调用,而在调用的时候才执行，这需要使用静态方法,这种代码是被动执行的。 静态方法在类加载的时候 就已经加载 可以用类名直接调用。

一句话：类加载时初始化，静态代码块是自动执行的；静态方法是被调用的时候才执行的。



### 静态内部类和非静态内部类的区别

- 静态内部类可以有静态成员(方法，属性)，而非静态内部类则不能有静态成员(方法，属性)。
- 静态内部类只能够访问外部类的静态成员,而非静态内部类则可以访问外部类的所有成员(方法，属性)。
- 非静态内部类会持有外部类的引用；静态内部类不会持有外部类的引用。



### 父类的静态方法能不能被子类重写

#### 重写/重载

- 重写是子类对父类的允许访问的方法的实现过程进行重新编写, 返回值和形参都不能改变。即外壳不变，核心重写！
- 重载(overloading) 是在一个类里面，方法名字相同，而参数不同。返回类型可以相同也可以不同。每个重载的方法（或者构造函数）都必须有一个独一无二的参数类型列表。



#### 静态方法

所谓静态就是指：在编译之后所分配的内存会一直存在（不会被回收），直到程序退出内存才会释放这个空间。

在java中，所有的东西都是对象，对象的抽象就是类，对于一个类而言，如果要使用他的成员（类中的属性，方法等），一般情况下，必须先实例化对象后，通过对象的引用才能访问这些成员。但是，如果要使用的成员使用了static修饰，就可以不通过实例化获得该成员。

#### 父类的静态方法能不能被子类重写

不能

因为静态方法从程序开始运行后就已经分配了内存，也就是说已经写死了。所有引用到该方法的对象（父类的对象也好子类的对象也好）所指向的都是同一块内存中的数据，也就是该静态方法。子类中如果定义了相同名称的静态方法，并不会重写，而应该是在内存中又分配了一块给子类的静态方法，没有重写这一说。




# Java的异常体系

异常知识体系树如下图：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190530174605603.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzI5OTY2MjAz,size_16,color_FFFFFF,t_70)

- Error是程序代码无法处理的错误，比如OutOfMemoryError、ThreadDeath等。这些异常发生时，Java虚拟机（JVM）一般会选择线程终止退出，其表示程序在运行期间出现了十分严重、不可恢复的错误，应用程序只能中止运行。
- Exception分运行时异常和非运行时异常。
  - 运行时异常都是RuntimeException类及其子类异常，如NullPointerException、IndexOutOfBoundsException等，这些异常也是不检查异常，程序代码中自行选择捕获处理，也可以不处理。这些异常一般是由程序逻辑错误引起的，程序代码应该从逻辑角度尽可能避免这类异常的发生。
  - 所有继承Exception且不是RuntimeException的异常都是非运行时异常，也称检查异常，如上图中的IOException和ClassNotFoundException，编译器会对其作检查，故程序中一定会对该异常进行处理，处理方法要么在方法体中声明抛出checked Exception，要么使用catch语句捕获checked Exception进行处理，不然不能通过编译。

## 抛出异常的方式

- 使用 `throw` 抛出异常：throw` 总是出现在**函数体**中，用来抛出一个 `Throwable` 类型的异常，例如抛出一个` `IOException` 类的异常对象。

- 使用 `throws` 抛出异常：如果一个方法可能会出现异常，但没有能力处理这种异常，可以在**方法声明处**用 `throws` 子句来声明抛出异常。

  

**throw 和 throws 的区别？**

- throw用于方法内部，throws用于方法声明上
- throw后跟异常对象，throws后跟异常类型
- throw后只能跟一个异常对象，throws后可以一次声明多种异常类型

## final、finally、finalize 有什么区别

- final：是修饰符，如果修饰类，此类不能被继承；如果修饰方法和变量，则表示此方法和此变量不能在被改变，只能使用。
- finally：是 try{} catch{} finally{} 最后一部分，表示不论发生任何情况都会执行，finally 部分可以省略，但如果 finally 部分存在，则一定会执行 finally 里面的代码。
- finalize： 是 Object 类的一个方法，在垃圾收集器执行的时候会调用被回收对象的此方法。

## try-catch-finally 中哪个部分可以省略？

try-catch-finally 其中 catch 和 finally 都可以被省略，但是不能同时省略，也就是说有 try 的时候，必须后面跟一个 catch 或者 finally。

```
public static void omitFinally() {
		try {
			int i = 0;
			i += 1;
			System.out.println(i);
		} catch (Exception e) {
			e.printStackTrace();
		}
	}
	
	
	public static void omitCatch() {
		int i = 0;
		try {
			i += 1;
		} finally {
			i = 10;
		}
		System.out.println(i);
	}
```



## try-catch-finally 中，如果 catch 中 return 了，finally 还会执行吗？

finally 一定会执行，即使是 catch 中 return 了，catch 中的 return 会等 finally 中的代码执行完之后，才会执行。

**在以下 4 种特殊情况下，`finally` 块不会被执行：**

- 在 `finally` 语句块中发生了异常。
- 在前面的代码中用了 `System.exit()` 退出程序。
- 程序所在的线程死亡。
- 关闭 CPU。

## 常见的异常类有哪些？

- NullPointerException 当应用程序试图访问空对象时，则抛出该异常。
- SQLException 提供关于数据库访问错误或其他错误信息的异常。
- IndexOutOfBoundsException指示某排序索引（例如对数组、字符串或向量的排序）超出范围时抛出。
- NumberFormatException当应用程序试图将字符串转换成一种数值类型，但该字符串不能转换为适当格式时，抛出该异常。
- FileNotFoundException当试图打开指定路径名表示的文件失败时，抛出此异常。
- IOException当发生某种I/O异常时，抛出此异常。此类是失败或中断的I/O操作生成的异常的通用类。
- ClassCastException当试图将对象强制转换为不是实例的子类时，抛出该异常。
- ArrayStoreException试图将错误类型的对象存储到一个对象数组时抛出的异常。
- IllegalArgumentException 抛出的异常表明向方法传递了一个不合法或不正确的参数。
- ArithmeticException当出现异常的运算条件时，抛出此异常。例如，一个整数“除以零”时，抛出此类的一个实例。
- NegativeArraySizeException如果应用程序试图创建大小为负的数组，则抛出该异常。
- NoSuchMethodException无法找到某一特定方法时，抛出该异常。
- SecurityException由安全管理器抛出的异常，指示存在安全侵犯。
- UnsupportedOperationException当不支持请求的操作时，抛出该异常。
- RuntimeExceptionRuntimeException 是那些可能在Java虚拟机正常运行期间抛出的异常的超类。



# 反射

https://www.zhihu.com/question/24304289

![img](http://image.tengj.top/Javareflect.png)

- java代码在计算机中经历的阶段（三阶段）

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190901141812427.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzI5OTY2MjAz,size_16,color_FFFFFF,t_70)

![img](https://img-blog.csdn.net/20170513133210763)

Java反射机制是指在运行状态中，可以对任意类，都知道它的属性和方法并调用。这种动态获取的信息以及动态调用对象的方法的功能称为java语言的反射机制。一句话**反射可以在运行时知道类的属性和方法**。

反射就是把Java类中的各种成分映射成对象的Java类，然后对这个java类进行操作。（比如：类的成员属性Field、类的成员方法Method、类的构造方法）

- 反射的好处
  - 可以在程序运行时操作这些对象。
  - 可以解耦，提高程序可扩展性。
- 反射的缺点
  - 反射的效率很低
  - 会破坏封装，通过反射可以访问类的私有方法，不安全



## 字节码Class对象

### Class类对象——描述.class字节码文件

将java文件经过编译后变成class字节码文件通过类加载器加载到内存，通过java.lang.Class类对象对字节码文件进行描述。每一个类都是一个class类对象的实例。**class类对象是用来对类的描述，主要包括三个成员变量**：

- 类成员变量 Field[] fields
- 类成员方法 Method[] methods
- 类构造函数 Constructor[] constructors

### 获取Class对象的方式

Class类构造函数是私有的

```
 /*
     * Private constructor. Only the Java Virtual Machine creates Class objects.
     * This constructor is not used and prevents the default constructor being
     * generated.
     */
private Class() {}
```

- Class.forName(“全类名”)

  **在Source源代码阶段**，此时java类仍位于硬盘上。多用于配置文件，将类名定义在配置文件中。读取文件，并触发类构造器加载类。Class.forName() 方法如果写错类的路径会报 ClassNotFoundException 的异常。

- 类名.class：通过类名的属性class获取

  **在Class类对象阶段**，此时java类位于内存中，但没有实际对象。多用于参数的传递。通过这种方式时，只会加载Dog类，并不会触发其类构造器的初始化。

- 对象.getClass()：getClass（）方法在Object类中定义

  **在运行阶段**，此时已经获取类的实例对象，多用于对象的获取字节码的方式。

```
		// 方法1：Class.forName("全类名")
        try {
            Class cls1 = Class.forName("com.test.demo.Dog");
        } catch (ClassNotFoundException e) {}
        // 方法2：类名.class
        Class cls2 = Dog.class;
        // 方法3：对象.getClass()
        Dog dog = new Dog();
    	Class cls3 = dog.getClass();
		// 用 == 比较3个对象是否为同一个对象（指向同一物理地址）
		System.out.print(cls1 == cls2);	//	true
		System.out.print(cls1 == cls3);	//	true
```

同一个字节码文件（*.class）在一次程序运行过程中，只会被加载一次，不论通过哪一种方式获取的class对象都是同一个。**也就是说在运行期间，一个类，只有一个Class对象产生。**

## 反射机制



# 范型

## 什么是范型

用来规定类、接口、方法接受数据的类型。

泛型的本质就是利用编译器实现的Java语法糖，编译器将java文件转换为class文件前，会进行泛型擦除，所以在反编译的class文件中，是看不到泛型声明的

## 范型的优缺点

**范型的优点：**

-  **提高安全性:** 将运行期的错误转换到编译期. 如果我们在对一个对象所赋的值不符合其泛型的规定, 就会编译报错.
- **避免强转:** 比如我们在使用List时, 如果我们不使用泛型, 当从List中取出元素时, 其类型会是默认的Object, 我们必须将其向下转型为String才能使用。

**范型的缺点：**

- 类型擦除问题

### 范型擦除

在生成的Java字节代码中是不包含泛型中的类型信息的。使用泛型的时候加上的类型参数，会被编译器在编译的时候去掉。这个过程就称为类型擦除。



## 范型的使用

泛型有三种使用方式，分别为：

- 泛型类
- 泛型接口
- 泛型方法

范型使用规则：

- 泛型的类型参数只能是类类型（包括自定义类），不能是基本类型
- 泛型只在编译阶段有效。
- 在实例化泛型类时，必须指定具体类型，如果不传入泛型类型实参的话，不会起到限制的作用，在泛型类中使用泛型的方法或成员变量定义的类型可以为任何的类型。
- 不能对确切的泛型类型使用instanceof操作
- 当实现泛型接口的类，未传入泛型实参时，声明类需将泛型的声明也一起加到类中，如果不声明，编译器会报错。
- public 与 返回值中间<T>，可以理解为声明此方法为泛型方法。
- 只有声明了<T>的方法才是泛型方法，泛型类中的使用了泛型的成员方法并不是泛型方法。

### 范型类

泛型的类型参数只能是类类型（包括自定义类），不能是基本类型

```
public class Base<T> {

    private T t;

    public void setT(T t) {
        this.t = t;
    }

    public T getT() {
        return t;
    }
}
Base<Double> base = new Base<Double>();
```

### 范型接口

```
//定义一个泛型接口
public interface Generator<T> {
    public T next();
}
```

在实例化泛型类时，必须指定具体类型，如果不传入泛型类型实参的话，不会起到限制的作用，在泛型类中使用泛型的方法或成员变量定义的类型可以为任何的类型。

```
class FruitGenerator<T> implements Generator<T>{
    @Override
    public T next() {
        return null;
    }
}

public class FruitGenerator implements Generator<String> {

    private String[] fruits = new String[]{"Apple", "Banana", "Pear"};

    @Override
    public String next() {
        Random rand = new Random();
        return fruits[rand.nextInt(3)];
    }
}
```

### 范型方法

print是范型方法，其它的get set 不是范型方法。

```
public class Base<T> {

    private T t;

    public void setT(T t) {
        this.t = t;
    }

    public T getT() {
        return t;
    }

    public <E, K> K print(E e, K k) {

        return k;
    }
}
```



## 泛型中的通配符

范型中的通配符规定只允许某一部分类作为泛型。

### 通配符分类

- 无边界通配符(<?>)：让泛型能够接受未知类型的数据
- 固定上边界通配符(<? extends E>),即传入的类型实参必须是指定类型的子类型。注意: 这里虽然用的是extends关键字, 却不仅限于继承了父类E的子类, 也可以代指显现了接口E的类.
- 固定下边界通配符（? super E）即传入的类型实参必须是指定类型的父类型

## 关于泛型数组

看到了很多文章中都会提起泛型数组，经过查看sun的说明文档，在java中是**”不能创建一个确切的泛型类型的数组”**的。

也就是说下面的这个例子是不可以的：

```
List<String>[] ls = new ArrayList<String>[10];  
```

而使用通配符创建泛型数组是可以的，如下面这个例子：

```
List<?>[] ls = new ArrayList<?>[10];  
```

```
List<String>[] ls = new ArrayList[10];
```

下面使用[Sun](http://docs.oracle.com/javase/tutorial/extra/generics/fineprint.html)[的一篇文档](http://docs.oracle.com/javase/tutorial/extra/generics/fineprint.html)的一个例子来说明这个问题：

```
List<String>[] lsa = new List<String>[10]; // Not really allowed.    
Object o = lsa;    
Object[] oa = (Object[]) o;    
List<Integer> li = new ArrayList<Integer>();    
li.add(new Integer(3));    
oa[1] = li; // Unsound, but passes run time store check    
String s = lsa[1].get(0); // Run-time error: ClassCastException.
```

> 这种情况下，由于JVM泛型的擦除机制，在运行时JVM是不知道泛型信息的，所以可以给oa[1]赋上一个ArrayList而不会出现异常，但是在取出数据的时候却要做一次类型转换，所以就会出现ClassCastException，如果可以进行泛型数组的声明，上面说的这种情况在编译期将不会出现任何的警告和错误，只有在运行时才会出错。

而对泛型数组的声明进行限制，对于这样的情况，可以在编译期提示代码有类型安全问题，比没有任何提示要强很多。

下面采用通配符的方式是被允许的:**数组的类型不可以是类型变量，除非是采用通配符的方式**，因为对于通配符的方式，最后取出数据是要做显式的类型转换的。

```
List<?>[] lsa = new List<?>[10]; // OK, array of unbounded wildcard type.    
Object o = lsa;    
Object[] oa = (Object[]) o;    
List<Integer> li = new ArrayList<Integer>();    
li.add(new Integer(3));    
oa[1] = li; // Correct.    
Integer i = (Integer) lsa[1].get(0); // OK 
```



# 注解

## 什么是注解

注解（Annotation），也叫元数据，一种代码级别的说明。JDK1.5引入的特性，与类、接口、枚举是在同一个层次，它可以声明在包、类、字段、方法、局部变量、方法参数等的前面，用来对这些元素进行说明，注释。**注解的本质就是一个继承了 Annotation 接口的接口**，所有的注解类型都继承自Annotation。

## 元注解

元注解的作用就是负责注解其他注解。Java5.0定义了4个标准的meta-annotation类型，它们被用来提供对其它 annotation类型作说明。Java5.0定义的元注解：

- @Target：**用于描述注解的使用范围（即：被描述的注解可以用在什么地方）**

  ```
  public enum ElementType {
      /** 表示可以用来修饰类、接口、注解类型或枚举类型Class, interface (including annotation type), or enum declaration */
      TYPE,
      /** 可以用来修饰属性（包括枚举常量）Field declaration (includes enum constants) */
      FIELD,
      /** 可以用来修饰方法Method declaration */
      METHOD,
      /** 可以用来修饰参数 Formal parameter declaration */
      PARAMETER,
      /** 可以用来修饰构造器 Constructor declaration */
      CONSTRUCTOR,
      /** 可用来修饰局部变量Local variable declaration */
      LOCAL_VARIABLE,
      /** 可以用来修饰注解类型Annotation type declaration */
      ANNOTATION_TYPE,
      /** 可以用来修饰包Package declaration */
      PACKAGE,
      /**
       * 标注在类型参数上Type parameter declaration
       *
       * @since 1.8
       */
      TYPE_PARAMETER,
      /**
       * 用于标注任意类型(不包括class)Use of a type
       *
       * @since 1.8
       */
      TYPE_USE
  }
  ```

  

- @Retention：**定义了`Annotation`的生命周期**

  - 仅编译期：`RetentionPolicy.SOURCE`；

  - 仅class文件：`RetentionPolicy.CLASS`；

  - 运行期：`RetentionPolicy.RUNTIME`。注解信息将在运行期(JVM)也保留，因此可以通过反射机制读取注解的信息（源码、class文件和执行的时候都有注解的信息）

    **如果`@Retention`不存在，则该`Annotation`默认为`CLASS`。因为通常我们自定义的`Annotation`都是`RUNTIME`，所以，务必要加上`@Retention(RetentionPolicy.RUNTIME)`这个元注解：**

- @Documented：**用来描述注解是否被抽取到api文档中。在生成javadoc文档的时候将该Annotation也写入到文档中。**
- @Inherited：**如果一个使用了`@Inherited`修饰的`annotation`类型被用于一个class，则这个`annotation`将被用于该class的子类。并且仅针对`class`的继承，对`interface`的继承无效：**
- @Repeatable：它表示在同一个位置重复相同的注解。在没有该注解前，一般是无法在同一个类型上使用相同的注解的。

## 常见注解

- @Override：用于标记该方法是复写的父类中的某个方法。属于标记注解，不需要设置属性值；只能添加在方法的前面。

- @Deprecated：说明被修饰内容已被“废弃”，不再建议用户使用。

- @SuppressWarnings：忽略警告。

- @TargetApi：版本注解。

- @SuppressLint：避免在lint检查时报错

- **Nullable**：用于标记方法参数或者返回值可以为空；

- **@NonNull:**用于标记方法参数或者返回值不能为空，如果为空编译器会报警告；

- @IdRes：Android中一系列**资源类型注解**。

- 使用整型常量代替枚举类型

  ```
  public class IceCreamFlavourManager {
      private int flavour;
  
      public static final int VANILLA = 0;
      public static final int CHOCOLATE = 1;
      public static final int STRAWBERRY = 2;
  
      @IntDef({VANILLA, CHOCOLATE, STRAWBERRY})
      public @interface Flavour {
      }
  
      @Flavour
      public int getFlavour() {
          return flavour;
      }
  
      public void setFlavour(@Flavour int flavour) {
          this.flavour = flavour;
      }
  }
  ```

- @UiThread :等线程注解

- @Size、@IntRange、@FloatRange等值约束注解。

- 权限注解

- @CheckResult：**返回值注解**。

## 自定义注解

### 注解定义

```
public @interface 注解名 {定义体}


@Documented
@Target({ElementType.FIELD, ElementType.PARAMETER})
@Retention(RetentionPolicy.RUNTIME)
public @interface Test {
	String value() default "";
}
```



### 使用注解

#### 编译时期使用注解

编译时注解指的是@Retention的值为CLASS的注解。对于这类注解的解析，我们只需做好以下两件事儿：

- 自定义一个派生自 AbstractProcessor的“注解处理类”；
- 重写process 函数。

实际上，javac中包含的注解处理器在编译时会自动查找所有继承自 AbstractProcessor 的类，然后调用它们的 process 方法。因此我们只要做好上面两件事，编译器就会主动去解析我们的编译时注解。现在，我们把上面定义的TestAnnotation的Retention改为CLASS，我们就可以按照以下代码来解析它：

```
@SupportedAnnotationTypes("com.zhyen.com.TestAnnotation")
public class TestAnnotationClass extends AbstractProcessor {

    @Override
    public boolean process(Set<? extends TypeElement> annotations, RoundEnvironment roundEnv) {
        HashMap<String, String> map = new HashMap<>();
        for (TypeElement element : annotations) {
            for (Element e : roundEnv.getElementsAnnotatedWith(element)) {
                TestAnnotation annotation=e.getAnnotation(TestAnnotation.class);
                map.put(e.getEnclosingElement().toString(),annotation.value());
            }
        }
        return false;
    }
}
```

SupportedAnnotationTypes注解指出了MyProcessor向要解析的注解的完整名字（全限定名称）。process+函数的annotations参数表示待处理的注解集，通过env我们可以得到被特定注解所修饰的程序元素。process函数的返回值表示annotations中的注解是否被这个Processor接受。&oq=%40SupportedAnnotationTypes注解指出了MyProcessor向要解析的注解的完整名字（全限定名称）。process+函数的annotations参数表示待处理的注解集，通过env我们可以得到被特定注解所修饰的程序元素。process函数的返回值表示annotations中的注解是否被这个Processor接受

#### 运行时期使用直接

Class 类中提供了以下一些方法用于反射注解：

- getAnnotation：返回指定的注解

- isAnnotationPresent：判定当前元素是否被指定注解修饰

- getAnnotations：返回所有的注解

- getDeclaredAnnotation：返回本元素的指定注解

- getDeclaredAnnotations：返回本元素的所有注解，不包含父类继承而来的

**方法、字段中相关反射注解的方法基本是类似的**

### 案例一个

注解类：

```
@Documented
@Target({ElementType.METHOD, ElementType.FIELD})
@Retention(RetentionPolicy.RUNTIME)
public @interface TestAnnotation {
    String value() default "null";

    public String name() default " ";

    int index() default 1;
}

```

使用类：

```
public class TestAnnotationClass {

    @TestAnnotation(value = "汪汪汪", name = "dog", index = 100)
    public String dog;
    @TestAnnotation(value = "喵喵喵", name = "cat", index = 200)
    public String cat;

    @TestAnnotation(value = "爱爆炸", index = 10, name = "三星")
    public void phone() {

    }

    @TestAnnotation(value = "细", index =20, name = "ARM")
    public void CPU() {

    }
}
```

获取注解:

```
Class<TestAnnotationClass> clazz = TestAnnotationClass.class;
        Field[] fields = clazz.getFields();
        for (Field field : fields) {
            TestAnnotation annotation = field.getAnnotation(TestAnnotation.class);
            String name = annotation.name();
            int index = annotation.index();
            String value = annotation.value();
            Log.d(TAG, "name = " + name + " ,index = " + index + ", value = " + value);
        }

        TestAnnotation annotation = method.getAnnotation(TestAnnotation.class);
            String name = annotation.name();
            int index = annotation.index();
            String value = annotation.value();
            Log.d(TAG, "name = " + name + " ,index = " + index + ", value = " + value);
```

结果:

```
name = cat ,index = 200, value = 喵喵喵
name = dog ,index = 100, value = 汪汪汪
方法名: CPU 
name = ARM ,index = 20, value = 细
equals
方法名: getClass
方法名: hashCode
方法名: notify
方法名: phone
name = 三星 ,index = 10, value = 爱爆炸
...
```



# 抽象类（Abstract Class）

## 定义

如果一个类没有包含足够多的信息来描述一个具体的对象，这样的类就是抽象类。抽象类用abstract修饰。

## 特点

- 不能实例化，因为抽象类中含有无法具体实现的方法。
- 可在抽象类中定义公共成员变量、成员方法、构造方法等。
- 只要包含一个抽象方法的类，该类必须要定义成抽象类（抽象方法是一种特殊的方法，它只有声明但没有具体的实现，抽象方法必须为public或protected）。故可理解为抽象类是在普通类结构里增加抽象方法的组成部分。
- 如果子类继承于一个抽象类，则该子类可以有选择性决定是否覆写父类的抽象方法，如果子类没有实现父类的抽象方法，则必须将子类也定义为抽象类（抽象类可以继承抽象类）
- 继承只能单继承，一个子类只能继承一个抽象类。
- abstract不能与final并列修饰同一个类。
- abstract 不能与private、static、final或native并列修饰同一个方法。

## 实例

```
abstract class A{//定义一个抽象类
	
	public void fun(){//普通方法
		System.out.println("存在方法体的方法");
	}
	
	public abstract void print();//抽象方法，没有方法体，有abstract关键字做修饰
	
}
```



## 面试题

### 抽象类必须有抽象方法吗

不是

### 抽象类可以继承普通类吗

可以

### 抽象类有构造方法吗？

有

### 抽象类可以用final声明么？

不能，因为抽象类必须有子类，而final定义的类不能有子类；

### 抽象类能否使用static声明？

外部抽象类不允许使用static声明，而内部的抽象类运行使用static声明。使用static声明的内部抽象类相当于一个外部抽象类，继承的时候使用“外部类.内部类”的形式表示类名称。

### 可以直接调用抽象类中用static声明的方法么？

任何时候，如果要执行类中的static方法的时候，都可以在没有对象的情况下直接调用，对于抽象类也一样。



# 接口（Interface）

## 定义

官方解释：Java接口是一系列方法的声明，是一些方法特征的集合，**一个接口只有方法的特征没有方法的实现，因此这些方法可以在不同的地方被不同的类实现，而这些实现可以具有不同的行为（功能）**。

## 特点

- 接口指明了一个类必须要做什么和不能做什么，相当于类的蓝图。
- 接口的子类（如果不是抽象类），那么必须要覆写接口中的全部抽象方法；
- 一个接口可以继承于另一个接口，或者另一些接口，接口也可以继承，并且可以多继承。接口不能实现(implement)另一个接口
- 一个类如果要实现某个接口的话，那么它必须要实现这个接口中的所有方法。
- 不能直接去实例化一个接口，因为接口中的方法都是抽象的，是没有方法体的
- 接口没有构造方法
- 接口中所有的方法都是抽象的和public的，所有的属性都是public,static,final的。
- 接口支持多继承（一个类可以实现多个接口）
- 一个接口就是描述一种能力，比如“运动员”也可以作为一个接口，并且任何实现“运动员”接口的类都必须有能力实现奔跑这个动作（或者implement move()方法），所以接口的作用就是告诉类，你要实现我这种接口代表的功能，你就必须实现某些方法，我才能承认你确实拥有该接口代表的某种能力。
- 接口也被用来实现解耦。

## 实例

```
interface Door{
void open ();
void close();
}

Public class BigDoor implements Door {

void open (){
System.out.println("BigDoor is opening...");
};

void close(){
System.out.println("BigDoor is closing...");
};

}
```



## 接口的标识用法

虽然接口内部定义了一些抽象方法，但是并不是所有的接口内部都必须要有方法，比如Seriallizable接口，Seriallizable接口的作用是使对象能够“序列化”，但是Seriallizable接口中却没有任何内容，也就是说，如果有一个类需要实现“序列化”的功能，则这个类必须去实现Seriallizable接口，但是却并不用实现方法（因为接口中没有方法），此时，这个Serilizable接口就仅仅是一个“标识”接口，是用来标志一个类的，标志这个类具有这个“序列化”功能。

# 抽象类与接口区别

- 抽象类中可以有自己的方法实现。也可以有抽象方法。接口只有抽象方法。
- 一个子类只能继承一个父类，但可以实现多个接口。
- 子类使用extends关键字继承抽象类。子类可以选择性重写抽象类中需要使用的方法;子类使用implements关键字实现接口。子类需要提供接口中所有声明的方法的实现。
- 抽象类可以有构造方法，但接口没有构造方法。但抽象类的构造器不用于创造对象，而是让其子类调用这些构造器完成抽象类的初始化操作。
- 抽象方法比接口速度快。接口需要时间去寻找在类中实现的方法，故速度较慢。
- 抽象类是对事物的一种抽象，描述的是某一类特性的事物。表示 这个对象是什么。（is-a关系——强调所属关系）;接口是对行为功能的抽象，描述是否具备某种行为特征。表示 这个对象能做什么。（has-a关系——强调功能实现）

# 深拷贝和浅拷贝

## 对象的创建方式

### 使用new操作符创建一个对象

new操作符的本意是分配内存。程序执行到new操作符时， 首先去看new操作符后面的类型，因为知道了类型，才能知道要分配多大的内存空间。分配完内存之后，再调用构造函数，填充对象的各个域，这一步叫做对象的初始化，构造方法返回后，一个对象创建完毕，可以把他的引用（地址）发布到外部，在外部就可以使用这个引用操纵这个对象。

### 使用clone方法复制一个对象

clone在第一步是和new相似的， 都是分配内存，调用clone方法时，分配的内存和源对象（即调用clone方法的对象）相同，然后再使用原对象中对应的各个域，填充新对象的域， 填充完成之后，clone方法返回，一个新的相同的对象被创建，同样可以把这个新对象的引用发布到外部。

## clone的使用

- 实现Cloneable接口

- 重写Object中clone方法，定位为public

- 调用super.clone();
- 实现try catch 捕获异常
- clone默认实现的是浅拷贝。

```
public class Book implements Cloneable {
	@Override
    public Object clone() throws CloneNotSupportedException {
        return (Book)super.clone();
    }
}
```

### clone规则

- 如果变量是基本类型，克隆的是其**值**比如int、float等。
- 如果变量是实例对象，则拷贝其**地址引用**，也就是说新对象和原来对象是共用实例变量的。
- 若变量是string字符串，拷贝其地址引用，但是在修改时，它会从字符串池中重新生成一个新的字符串，原有的对象保持不变。
- clone默认实现的是浅拷贝。

### 浅拷贝

浅拷贝是按位拷贝对象，它会创建一个新对象，这个对象有着原始对象属性值的一份精确拷贝。如果属性是基本类型，拷贝的就是基本类型的值；如果属性是内存地址（引用类型），拷贝的就是内存地址 ，因此如果其中一个对象改变了这个地址，就会影响到另一个对象。

- 如果原型对象的成员变量是值类型，将复制一份给克隆对象，
- 如果原型对象的成员变量是引用类型，则将引用类型的地址复制一份给克隆对象。**也就是说原型对象和克隆对象的成员变量指向相同的内存地址**

![image.png](https://upload-images.jianshu.io/upload_images/4118241-8ec91452d46a20ae.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 深拷贝

在深拷贝中无论值类型还是引用类型都会复制一份给克隆对象，深拷贝将原型对象的所有引用对象也复制一份给克隆对象。

深拷贝会拷贝所有的属性,并拷贝属性指向的动态分配的内存。当对象和它所引用的对象一起拷贝时即发生深拷贝。深拷贝相比于浅拷贝速度较慢并且花销较大。

如果想要深拷贝一个对象， 这个对象必须要实现Cloneable接口，实现clone方法，并且在clone方法内部，把该对象引用的其他对象也要clone一份 ， 这就要求这个被引用的对象必须也要实现Cloneable接口并且实现clone方法。

![image.png](https://upload-images.jianshu.io/upload_images/4118241-c2b9f8dbacd2bffc.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![image.png](https://upload-images.jianshu.io/upload_images/4118241-76e0e5813178abab.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

# Java多线程之同步集合与并发集合

## 同步集合类

包括`Hashtable`、`Vector`、同步集合包装类，`Collections.synchronizedMap()`和`Collections.synchronizedList()`。

## 并发集合类

包括`ConcurrentHashMap`、`CopyOnWriteArrayList`、`CopyOnWriteHashSet`

同步集合比并发集合会慢得多，主要原因是锁，同步集合会对整个Map或List加锁。

- **ConcurrentHashMap：**把整个Map 划分成几个片段，只对相关的几个片段上锁，同时允许多线程访问其他未上锁的片段。
- **CopyOnWriteArrayList：**允许多个线程以非同步的方式读，当有线程写的时候它会将整个List复制一个副本给它。如果在读多写少这种对并发集合有利的条件下使用并发集合，这会比使用同步集合更具有可伸缩性。

一般不需要多线程的情况，只用到HashMap、ArrayList，只要真正用到多线程的时候就一定要考虑同步。所以这时候才需要考虑同步集合或并发集合。

### ConcurrentHashMap

ConcurrentHashMap是由Segment数组结构和HashEntry数组结构组成。Segment是一种可重入锁ReentrantLock，在ConcurrentHashMap里扮演锁的角色，HashEntry则用于存储键值对数据。一个ConcurrentHashMap里包含一个Segment数组，Segment的结构和HashMap类似，是一种数组和链表结构， 一个Segment里包含一个HashEntry数组，每个HashEntry是一个链表结构的元素， 每个Segment守护者一个HashEntry数组里的元素,当对HashEntry数组的数据进行修改时，必须首先获得它对应的Segment锁。

![image.png](https://upload-images.jianshu.io/upload_images/4118241-cefab31cd3461494.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### JDK1.7 & JDK1.8 对比

1. 数据结构：取消了Segment分段锁的数据结构，取而代之的是数组+链表+红黑树的结构。

2. 保证线程安全机制：JDK1.7采用segment的分段锁机制实现线程安全，其中segment继承自ReentrantLock。JDK1.8采用CAS+Synchronized保证线程安全。
3. 锁的粒度：原来是对需要进行数据操作的Segment加锁，现调整为对每个数组元素加锁（Node）。
4. 链表转化为红黑树:定位结点的hash算法简化会带来弊端,Hash冲突加剧,因此在链表节点数量大于8时，会将链表转化为红黑树进行存储。
5. 查询时间复杂度：从原来的遍历链表O(n)，变成遍历红黑树O(logN)。



### CopyOnWrite容器

CopyOnWrite容器即写时复制的容器。通俗的理解是当我们往一个容器添加元素的时候，不直接往当前容器添加，而是先将当前容器进行Copy，复制出一个新的容器，然后新的容器里添加元素，添加完元素之后，再将原容器的引用指向新的容器。这样做的好处是我们可以对CopyOnWrite容器进行并发的读，而不需要加锁，因为当前容器不会添加任何元素。所以CopyOnWrite容器也是一种读写分离的思想，读和写不同的容器。

#### CopyOnWriteArrayList

在添加的时候是需要加锁的，否则多线程写的时候会Copy出N个副本出来。

读的时候不需要加锁，如果读的时候有多个线程正在向ArrayList添加数据，读还是会读到旧的数据，因为写的时候不会锁住旧的ArrayList。



# 多线程开发

## 线程状态

![image.png](https://upload-images.jianshu.io/upload_images/4118241-98c472afe493c25d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![image.png](https://upload-images.jianshu.io/upload_images/4118241-891a1d42c08637a8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## wait 与 sleep 的区别与联系

wait和sleep均能使线程处于等待状态

- wait方法定义在Object里面，基于对象锁，所有的对象都能使用（Java里面每一个对象都有隐藏锁，也叫监视器(monitor)。当一个线程进入一个synchronized方法的时候它会获得一个当前对象的锁。）；sleep方法定义在Thread里面，是基于当前线程。
- wait必须在同步环境（synchronized方法）下使用，否则会报IllegalMonitorStateException异常
  sleep方法可在任意条件下使用
- wait/notify一起使用，用于线程间的通信。wait用于让线程进入等待状态，notify则唤醒正在等待的线程；sleep用于暂停当前线程的执行，它会在一定时间内释放CPU资源给其他线程执行，超过睡眠时间则会正常唤醒。
- 在同步环境中调用wait方法会释放当前持有的锁；调用sleep则不会释放锁，一直持有锁（直到睡眠结束）

## 线程阻塞的原因

- Thread.sleep(int millsecond) 调用 sleep 的线程会在一定时间内将 CPU 资源给其他线程执行，超过睡眠事件后唤醒。与是否持有同步锁无关。进程处于 TIMED_WAITING 状态
- 线程执行一段同步代码（Synchronic）代码，但无法获取同步锁：同步锁用于实现线程同步执行，未获得同步锁而无法进入同步块的线程处于 BLOCKED 状态
- 线程对象调用 wait 方法，进入同步块的线程发现运行条件不满足，此时会释放锁，并释放CPU，等待其他线程norify。此时线程处于 WAITING 状态
- 执行阻塞式I/O操作，等待相关I/O设备（如键盘、网卡等），为了节省CPU资源，释放CPU。此时线程处于RUNNABLE状态。

## 线程控制方法

JVM充分地利用现代多核处理器的强大性能。采用异步调用线程，提高使用性能，缺点就是会造成线程不安全。为了保证线程安全性，即确保Java内存模型的可见性、原子性和有序性。Java主要通过volatile、synchronized实现线程安全。

### Synchronized

synchronized 规定了同一个时刻只允许一条线程可以进入临界区（互斥性），同时还保证了共享变量的内存可见性。此规则决定了持有同一个对象锁的多个同步块只能串行执行。

Java中的每个对象都可以为锁：

- 普通同步方法，锁是当前实例对象。
- 静态同步方法，锁是当前类的class对象。
- 同步代码块，锁是括号中的对象。

wait/notify必须存在于synchronized块中。并且，这三个关键字针对的是同一个监视器（某个对象的监视器）。当某个线程并不持有监视器的使用权时，去wait或notify，会抛出java.lang.IllegalMonitorStateException。

当某个线程wait之后，其他执行该同步快的线程可以进入该同步块执行。

#### 锁的内部机制：从偏向锁到重量级锁

##### 对象头和monitor

Java对象在内存中的存储结构主要有一下三个部分：

- 对象头
- 实例数据
- 填充数据

monitor是线程私有的数据结构，每一个线程都有一个可用monitor列表，同时还有一个全局的可用列表，先来看monitor的内部。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190718141919602.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzI5OTY2MjAz,size_16,color_FFFFFF,t_70)

- Owner：初始时为NULL表示当前没有任何线程拥有该monitor，当线程成功拥有该锁后保存线程唯一标识，当锁被释放时又设置为NULL；
- EntryQ：关联一个系统互斥锁（semaphore），阻塞所有试图锁住monitor失败的线程。
- RcThis：表示blocked或waiting在该monitor上的所有线程的个数。
- Nest：用来实现重入锁的计数。
- HashCode：保存从对象头拷贝过来的HashCode值（可能还包含GC age）。
- Candidate：用来避免不必要的阻塞或等待线程唤醒，因为每一次只有一个线程能够成功拥有锁，如果每次前一个释放锁的线程唤醒所有正在阻塞或等待的线程，会引起不必要的上下文切换（从阻塞到就绪然后因为竞争锁失败又被阻塞）从而导致性能严重下降。Candidate只有两种可能的值：0表示没有需要唤醒的线程，1表示要唤醒一个继任线程来竞争锁。


在 java 虚拟机中，线程一旦进入到被synchronized修饰的方法或代码块时，指定的锁对象通过某些操作将对象头中的LockWord指向monitor 的起始地址与之关联，同时monitor 中的Owner存放拥有该锁的线程的唯一标识，确保一次只能有一个线程执行该部分的代码，线程在获取锁之前不允许执行该部分的代码。

##### 偏向锁

当线程执行到临界区（critical section）时，此时会利用CAS(Compare and Swap)操作，将线程ID插入到Markword中，同时修改偏向锁的标志位。

此时偏向锁标志位为1。

**偏向锁是jdk1.6引入的一项锁优化，其中的“偏”是偏心的偏。它的意思就是说，这个锁会偏向于第一个获得它的线程，在接下来的执行过程中，假如该锁没有被其他线程所获取，没有其他线程来竞争该锁，那么持有偏向锁的线程将永远不需要进行同步操作。**

**在此线程之后的执行过程中，如果再次进入或者退出同一段同步块代码，并不再需要去进行加锁或者解锁操作，而是会做以下的步骤：**

- Load-and-test，也就是简单判断一下当前线程id是否与Markword当中的线程id是否一致.
- 如果一致，则说明此线程已经成功获得了锁，继续执行下面的代码.
- 如果不一致，则要检查一下对象是否还是可偏向，即“是否偏向锁”标志位的值。
- 如果还未偏向，则利用CAS操作来竞争锁，也即是第一次获取锁时的操作。
- 如果此对象已经偏向了，并且不是偏向自己，则说明存在了竞争。此时可能就要根据另外线程的情况，可能是重新偏向，也有可能是做偏向撤销，但大部分情况下就是升级成轻量级锁了。

即偏向锁是针对于一个线程而言的，线程获得锁之后就不会进行解锁操作，节省了很多开销。为什么要这样做呢？因为经验表明，其实大部分情况下，都会是同一个线程进入同一块同步代码块的。这也是为什么会有偏向锁出现的原因。在Jdk1.6中，偏向锁的开关是默认开启的，适用于只有一个线程访问同步块的场景。

下述代码中，当线程访问同步方法method1时，会在对象头（SynchronizedTest.class对象的对象头）和栈帧的锁记录中存储锁偏向的线程ID，下次该线程在进入method2，只需要判断对象头存储的线程ID是否为当前线程，而不需要进行CAS操作进行加锁和解锁（因为CAS原子指令虽然相对于重量级锁来说开销比较小但还是存在非常可观的本地延迟）。

```
public class SynchronizedTest {
    private static Object lock = new Object();
    public static void main(String[] args) {
        method1();
        method2();
    }
    synchronized static void method1() {}
    synchronized static void method2() {}
}

```

##### 轻量级锁

当出现有两个线程来竞争锁的话，那么偏向锁就失效了，此时锁就会膨胀，升级为轻量级锁。锁撤销升级为轻量级锁之后，那么对象的Markword也会进行相应的的变化。下面先简单描述下锁撤销之后，升级为轻量级锁的过程：

- 线程在自己的栈桢中创建锁记录 LockRecord。
- 将锁对象的对象头中的MarkWord复制到线程的刚刚创建的锁记录中。
- 将锁记录中的Owner指针指向锁对象。
- 将锁对象的对象头的MarkWord替换为指向锁记录的指针。

轻量级锁主要是自旋锁。所谓自旋，就是指当有另外一个线程来竞争锁时，这个线程会在原地循环等待，而不是把该线程给阻塞，直到那个获得锁的线程释放锁之后，这个线程就可以马上获得锁的。注意，锁在原地循环的时候，是会消耗cpu的，就相当于在执行一个啥也没有的for循环。所以，轻量级锁适用于那些同步代码块执行的很快的场景，这样，线程原地等待很短很短的时间就能够获得锁了。自旋锁有一些问题：

- 如果同步代码块执行的很慢，需要消耗大量的时间，那么这个时侯，其他线程在原地等待空消耗cpu。
- 本来一个线程把锁释放之后，当前线程是能够获得锁的，但是假如这个时候有好几个线程都在竞争这个锁的话，那么有可能当前线程会获取不到锁，还得原地等待继续空循环消耗cup，甚至有可能一直获取不到锁。

基于这个问题，我们必须给线程空循环设置一个次数，当线程超过了这个次数，我们就认为，继续使用自旋锁就不适合了，此时锁会再次膨胀，升级为重量级锁。

##### 重量级锁

轻量级锁膨胀之后，就升级为重量级锁了。重量级锁是依赖对象内部的monitor锁来实现的，而monitor又依赖操作系统的MutexLock(互斥锁)来实现的，所以重量级锁也被成为互斥锁。

