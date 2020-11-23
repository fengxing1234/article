---
title: 复杂业务如何保证Flutter的高性能高流畅度
date: 2020-07-20 22:06:36
tags:
---

高性能高流畅度一直是Flutter团队宣传的一大亮点，也是当初闲鱼选择Flutter的重要因素之一，但是随着复杂业务的应用落地，通过Flutter页面和原生页面滑动流畅度对比，我们开始产生怀疑，因为部分Flutter页面流畅度明显低于Native，是Flutter的宣传言过其实还是我们开发人员使用姿势有问题，今天我们就来具体分析下。

### Flutter渲染原理简介

优化之前我们先来介绍下Flutter的渲染原理，通过这部分基础了解渲染流程以及主要耗时花费

flutter视图树包含了三颗树：Widget、Element、RenderObject

- Widget: 存放渲染内容、它只是一个配置数据结构，创建是非常轻量的，在页面刷新的过程中随时会重建
- Element: 同时持有Widget和RenderObject，存放上下文信息，通过它来遍历视图树，支撑UI结构
- RenderObject: 根据Widget的布局属性进行layout，paint ，负责真正的渲染

从创建到渲染的大体流程是：根据Widget生成Element，然后创建相应的RenderObject并关联到Element.renderObject属性上，最后再通过RenderObject来完成布局排列和绘制。

例如下面这段布局代码

```text
Container(
      color: Colors.blue,
      child: Row(
        children: <Widget>[
          Image.asset('image'),
          Text('text'),
        ],
      ),
    );
```

对应三棵树的结构如下图



![img](https://pic1.zhimg.com/80/v2-871d65414702552fd3daf3e9ed468632_1440w.jpg)



了解了这三棵树，我们再来看下页面刷新的时候具体做了哪些操作

当需要更新UI的时候，Framework通知Engine，Engine会等到下个Vsync信号到达的时候，会通知Framework进行animate, build，layout，paint，最后生成layer提交给Engine。Engine会把layer进行组合，生成纹理，最后通过Open Gl接口提交数据给GPU， GPU经过处理后在显示器上面显示，如下图：



![img](https://picb.zhimg.com/80/v2-1e5b51ac70061f9c36c5be080968e881_1440w.jpg)



结合前面的例子，如果text文本或者image内容发生变化会触发哪些操作呢？

Widget是不可改变，需要重新创建一颗新树，build开始，然后对上一帧的element树做遍历，调用他的updateChild，看子节点类型跟之前是不是一样，不一样的话就把子节点扔掉，创造一个新的，一样的话就做内容更新，对renderObject做updateRenderObject操作，updateRenderObject内部实现会判断现在的节点跟上一帧是不是有改动，有改动才会别标记dirty，重新layout、paint，再生成新的layer交给GPU，流程如下图：



![img](https://pic2.zhimg.com/80/v2-f76f7b02b7e2818a6ec32f1399f8dde3_1440w.jpg)



到这里大家对Flutter在渲染方面有基本的理解，作为后面优化部分内容理解的基础

### 性能分析工具及方法

下面来看下性能分析工具，注意，统计性能数据一定要在真机+profile模式下运行，拿到最接近真实的体验数据。

### performance overlay

平时常用的性能分析工具有performance overlay，通过他可以直观看到当前帧的耗时，但是他是UI线程和GPU线程分开展示的，UI Task Runner是Flutter Engine用于执行Dart root isolate代码，GPU Task Runner被用于执行设备GPU的相关调用。绿色的线表示当前帧，出现红色则表示耗时超过16.6ms，也就是发生丢帧现象



![img](https://picb.zhimg.com/80/v2-c7f72ec1facf96a953f447ea414f8be1_1440w.jpg)



### Dart DevTool

另一个工具是Dart DevTool ，就是早期的Observatory，官方提供的性能检测工具。它的 timeline 界面可以让逐帧分析应用的 UI 性能。但是目前还是预览版，存在一些问题。

profile模式下运行起来，点击android studio底部的菜单按钮，会弹出一个网页



![img](https://pic4.zhimg.com/80/v2-e13af00a4fdb29dc51fed89c4f629642_1440w.jpg)



点击顶部的Timeline菜单



![img](https://pic4.zhimg.com/80/v2-3c9361ddec4bc12116ac2b55eb123d5a_1440w.jpg)



这个时候滑动页面，每一帧的耗时会以柱形bar的形式显示在页面上，每条bar代表一个frame，同时用不同颜色区分UI/GPU线程耗时，这个时候我们要分析卡顿的场景就需要选中一条红色的bar（总耗时超过16.6ms），中间区域的Frame events chart显示了当前选中的frame的事件跟踪，UI和GPU事件是独立的事件流，但它们共享一个公共的时间轴。

选中Frame events chart中的某个事件，以上图为例Layout耗时最长，我们选中它，会在底部Flame chart区域显示一个自顶向下的堆栈跟踪，每个堆栈帧的宽度表示它消耗CPU的时长，消耗大量CPU时长的堆栈是我们首要分析的重点，后面就是具体分析堆栈，定位卡顿问题。

### debug调试工具

另外还有一些debug调试工具可以辅助查看更多信息，注意，只能在debug模式下使用分析，拿到的数据不能作为性能标准

debugProfileBuildsEnabled：向 Timeline 事件中添加每个widget的build 信息

debugProfilePaintsEnabled： 向 timeline 事件中添加每个renderObject的paint 信息

debugPaintLayerBordersEnabled：每个layer会出现一个边框，帮助区分layer层级

debugPrintRebuildDirtyWidgets：打印标记为dirty的widgets

debugPrintLayouts：打印标记为dirty的renderObjects

debugPrintBeginFrameBanner/debugPrintEndFrameBanner：打印每帧开始和结束

### 实例分析

了解这些工具下面我们来看个简单的demo具体分析下，一个由Column、Container、ListView嵌套的布局，其中有个定时器控制Text中显示的文本实时更新

```text
import 'dart:async';
import 'package:flutter/material.dart';
import 'package:flutter/widgets.dart';

class TestDemo extends StatefulWidget {
  @override
  State<StatefulWidget> createState() {
    return _TestDemoState();
  }
}

class _TestDemoState extends State<TestDemo> {
  int _count = 0;
  Timer _timer;
  @override
  void initState() {
    super.initState();
    _timer = Timer.periodic(Duration(milliseconds: 1000), (t) {
      setState(() {
        _count++;
      });
    });
  }
  @override
  void dispose() {
    if (_timer != null) {
      if (_timer.isActive) {
        _timer.cancel();
      }
    }
    super.dispose();
  }
  @override
  Widget build(BuildContext context) {
    return new Scaffold(
        appBar: new AppBar(
          title: new Text("Test Demo"),
        ),
        body: content()
    );
  }
  Widget content(){
    Widget result = Column(
      children: <Widget>[
        Container(
          margin: EdgeInsets.fromLTRB(10,10,10,5),
          height: 100,
          color: Color(0xff1fbfbf),
        ),
        Container(
          margin: EdgeInsets.fromLTRB(10,5,10,10),
          height: 100,
          color: Color(0xff1b8bdf),
        ),
        Container(
          height: 100,
          child: ListView.builder(
              scrollDirection: Axis.horizontal,
              itemCount: 5,
              itemBuilder: (context, index) {
                return Container(
                  width: 70,
                  height: 70,
                  child: Image.asset(
                    'assets/images/common_empty.png',
                    width: 50,
                    height: 50,
                  ),
                );
              }),
        ),

        Container(
            margin: EdgeInsets.fromLTRB(10,20,10,10),
            height: 100,
            width: 350,
            color: Colors.yellow,
            child: Center(
              child:
              Text(
                _count.toString(),
                style: TextStyle(fontSize: 18, fontWeight:FontWeight.bold),
              ),
            )
        ),
      ],
    );
    return result;
  }
}
```



![img](https://picb.zhimg.com/80/v2-60a9b01b819348ae851d13ad8675b8a5_1440w.jpg)



大部分widget都是静态的，只有黄色Container中包含一个内容一直刷新的Text，这个时候我们打开debugProfileBuildsEnabled，用Timeline分析下它的渲染耗时，可以通过Frame events chart中显示的build层级非常深



![img](https://pic2.zhimg.com/80/v2-f916161cd181b310e4372f66002c71d3_1440w.jpg)



结合第一部分渲染原理我们了解到，每次定时器刷新text数字的时候，整个页面widget树都会重新build，但其实只有最底层Container中的Text内容在改变，没有必要刷新整颗树，所以这里我们的优化方案是**提高build效率，降低Widget tree遍历的出发点，将setState刷新数据尽量下发到底层节点**，所以将Text单独抽取成独立的Widget，setState下发到抽取出的Widget内部

```text
class _TestDemoState extends State<TestDemo> {

  ...

  Widget content(){
    Widget result = Column(
      children: <Widget>[
        ...
        Container(
            margin: EdgeInsets.fromLTRB(10,20,10,10),
            height: 100,
            width: 350,
            color: Colors.yellow,
            child: Center(
              child:
                  CountText()
            )
        ),
      ],
    );
    return result;
  }
}

class CountText extends StatefulWidget {
  @override
  State<StatefulWidget> createState() {
    return _CountTextState();
  }
}

class _CountTextState extends State<CountText> {
  int _count = 0;
  Timer _timer;
  @override
  void initState() {
    super.initState();
    _timer = Timer.periodic(Duration(milliseconds: 1000), (t) {
      setState(() {
        _count++;
      });
    });
  }

  @override
  void dispose() {
    if (_timer != null) {
      if (_timer.isActive) {
        _timer.cancel();
      }
    }
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return Text(
      _count.toString(),
      style: TextStyle(fontSize: 18, fontWeight:FontWeight.bold),
    );
  }
}
```

修改后的Timeline显示如下图：



![img](https://pic3.zhimg.com/80/v2-41d1e54a7d152cfc1f63a7641c9dcdd6_1440w.jpg)



build层级明显减少，总耗时也明显降低

接下来分析下Paint过程有没有可以优化的部分，我们打开debugProfilePaintsEnabled变量分析可以看到Timeline显示的paint层级



![img](https://pic3.zhimg.com/80/v2-2f7d9472d96992170659e325b8a373e0_1440w.jpg)





![img](https://pic1.zhimg.com/80/v2-76c046b5f8fa38ca918bc8c92e9ed283_1440w.jpg)



通过`debugPaintLayerBordersEnabled = true;`显示layer边框可以看到不断变化的Text和其他Widget都是在同一个layer中的，这里我们想到的优化点是**利用RepaintBoundary提高paint效率，它为经常发生显示变化的内容提供一个新的隔离layer，新的layer paint不会影响到其他layer**

```text
RepaintBoundary(
          child: Container(
              margin: EdgeInsets.fromLTRB(10,20,10,10),
              height: 100,
              width: 350,
              color: Colors.yellow,
              child: Center(
                  child: CountText()
              )
          ),
        ),
```

看下优化后的效果



![img](https://picb.zhimg.com/80/v2-a0bec82598163aa8f9590f3a5ed275a0_1440w.jpg)





![img](https://pic2.zhimg.com/80/v2-92abea1bc18cbe7595134361d7a4b87b_1440w.jpg)



可以看到我们为黄色的Container建立了单独的layer，并且paint的层级减少很多

### 总结常见问题

1. 提高build效率，setState刷新数据尽量下发到底层节点
2. 提高paint效率，RepaintBoundry创建单独layer减少重绘区域

这两个我们之前的例子已经具体分析过

1. 减少build中逻辑处理，因为widget在页面刷新的过程中随时会通过build重建，build调用频繁，我们应该只处理跟UI相关的逻辑
2. 减少saveLayer（ShaderMask、ColorFilter、Text Overflow）、clipPath的使用，saveLayer会在GPU中分配一块新的绘图缓冲区，切换绘图目标，这个操作是在GPU中非常耗时的，clipPath会影响每个绘图指令,做相交操作，之外的部分剔除掉，所以这也是个耗时操作
3. 减少Opacity Widget 使用，尤其是在动画中，因为他会导致widget每一帧都会被重建，可以用 AnimatedOpacity 或 FadeInImage 进行代替

以上内容介绍了些Flutter常见的性能问题以及我们怎么用工具检测这个问题，在平时开发过程中要留意规避这类问题

### Flutter-DX案例分析

近期我们做了个Flutter端的动态化模板渲染方案Flutter-DX，它使用集团DinamicX的DSL，通过下发DSL模板，在Flutter侧实现动态解析渲染。具体介绍可以参考之前的文章：

[如何在Flutter上实现高性能的动态模板渲染](https://zhuanlan.zhihu.com/p/83331937)

[做一个高一致性、高性能的Flutter动态渲染，真的很难么？](https://zhuanlan.zhihu.com/p/90195812)

这里不再详细介绍。

尽管进行了一次渲染架构升级，很大程度上提升性能表现，但是通过高可用线上统计，发现在长列表场景下fps值没有达到预期值，所以需要进一步分析哪些操作导致的耗时问题。

以搜索页页面结构为例，外部是GridView的容器，里面都是一个个DX模板组成的宝贝card，滑动过程中发现流畅度要明显偏低

所以我们做了以下的优化措施

1.针对Sliver滑动的优化，sliver在滑动过程中，有一个超出屏幕上下250像素的一个缓存区



![img](https://picb.zhimg.com/80/v2-a7d4c4ef26c1474f9c201a9c865c2dac_1440w.jpg)



在列表滚动过程中，DX card不断的被重建和销毁，没有任何缓存机制，我们在其中加了个缓存池，流程如下，避免element不断的被销毁和创建，一定程度提高流畅度



![img](https://pic3.zhimg.com/80/v2-68962dcc3faf1d4d48fa47ad87279fe2_1440w.jpg)

\2. 通过Timeline分析发现TextPaint的layout耗时显著，进一步对比分析发现，同样的UI显示，带换行符的长文本长度layout耗时明显偏高，

后来确认带换行符的文本会影响布局效率，具体分析可以查看 [issue](https://link.zhihu.com/?target=https%3A//github.com/flutter/flutter/issues/52651)



![img](https://pic3.zhimg.com/80/v2-2f7104a6c8c7e4aefdd512b9e3f29d5d_1440w.jpg)



这里我们做的优化措施是在判断只有一行文本显示的情况下，截取换行符前的内容作为text文本，从而提升TextPaint layout效率。

除此之外，还有一些减少布局层级和简化build流程，预加载缓存等措施，实现将FPS提升3个点，达到一定程度的优化效果。

### 总结

以上内容分析了flutter的渲染原理以及遇到卡顿问题可以用哪些工具从哪些方向入手分析，Flutter虽然一直宣称流畅度是一大亮点，但也存在一定的优化空间，以及需要开发者掌握一定的开发技巧才能达到更丝滑的体验。