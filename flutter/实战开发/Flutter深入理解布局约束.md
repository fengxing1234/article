---
title: Flutter深入理解布局约束
date: 2020-07-22 12:24:40
tags:
	- Flutter
categories: 
	- Flutter
	- Flutter实战
---

## 一、深入理解布局约束

我们会经常听到一些开发者在学习 Flutter 时的疑惑：为什么我设置了 `width:100`， 但是看上去却不是 100 像素宽呢。（注意，本文中的“像素”均指的是逻辑像素） 通常你会回答，将这个 Widget 放进 `Center` 中，对吧？

**别这么干。**

如果你这样做了，他们会不断找你询问这样的问题：为什么 `FittedBox` 又不起作用了？ 为什么 `Column` 又溢出边界，亦或是 `IntrinsicWidth` 应该做什么。

其实我们首先应该做的，是告诉他们 Flutter 的布局方式与 HTML 的布局差异相当大 （这些开发者很可能是 Web 开发），然后要让他们熟记这条规则：

- **首先，上层 widget 向下层 widget 传递约束条件。**
-   **然后，下层 widget 向上层 widget 传递大小信息。**
-   **最后，上层 widget 决定下层 widget 的位置。**

如果我们在开发时无法熟练运用这条规则，在布局时就不能完全理解其原理，所以越早掌握这条规则越好！

**更多细节：**

- Widget 会通过它的 **父级** 获得自身的约束。 约束实际上就是 4 个浮点类型的集合： 最大/最小宽度，以及最大/最小高度。
- 然后，这个 widget 将会逐个遍历它的 **children** 列表。向子级传递 **约束**（子级之间的约束可能会有所不同），然后询问它的每一个子级需要用于布局的大小。
- 然后，这个 widget 就会对它子级的 **children** 逐个进行布局。 （水平方向是 `x` 轴，竖直是 `y` 轴）
- 最后，widget 将会把它的大小信息向上传递至父 widget（包括其原始约束条件）。

例如，如果一个 widget 中包含了一个具有 padding 的 Column， 并且要对 Column 的子 widget 进行如下的布局：

![image.png](https://upload-images.jianshu.io/upload_images/4118241-257efca55201e1a3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

那么谈判将会像这样：

**Widget**: "嘿！我的父级。我的约束是多少？"

**Parent**: "你的宽度必须在 `80` 到 `300` 像素之间，高度必须在 `30` 到 `85` 之间。"

**Widget**: "嗯...我想要 `5` 个像素的内边距，这样我的子级能最多拥有 `290` 个像素宽度和 `75` 个像素高度。"

**Widget**: "嘿，我的第一个子级，你的宽度必须要在 `0` 到 `290`，高度在 `0` 到 `75` 之间。"

**First child**: "OK，那我想要 `290` 像素的宽度，`20` 个像素的高度。"

**Widget**: "嗯...由于我想要将我的第二个子级放在第一个子级下面，所以我们仅剩 `55` 个像素的高度给第二个子级了。"

**Widget**: "嘿，我的第二个子级，你的宽度必须要在 `0` 到 `290`，高度在 `0` 到 `55` 之间。"

**Second child**: "OK，那我想要 `140` 像素的宽度，`30` 个像素的高度。"

**Widget**: "很好。我的第一个子级将被放在 `x: 5` & `y: 5` 的位置， 而我的第二个子级将在 `x: 80` & `y: 25` 的位置。"

**Widget**: "嘿，我的父级，我决定我的大小为 `300` 像素宽度，`60` 像素高度。"

## 二、限制

正如上述所介绍的布局规则中所说的那样， Flutter 的布局引擎有一些重要限制：

- 一个 widget 仅在其父级给其约束的情况下才能决定自身的大小。 这意味着 widget 通常情况下 **不能任意获得其想要的大小**。
- 一个 widget **无法知道，也不需要决定其在屏幕中的位置**。 因为它的位置是由其父级决定的。
- 当轮到父级决定其大小和位置的时候，同样的也取决于它自身的父级。 所以，在不考虑整棵树的情况下，几乎不可能精确定义任何 widget 的大小和位置。

## 三、案例

转到[原文地址](https://juejin.im/post/5f01fe055188252e4a27dfaf#heading-3)

## 四、严格约束（Tight） vs 宽松约束（loose）

以后你经常会听到一些约束为严格约束或宽松约束， 你花点时间来弄明白它们是值得的。

严格约束给你了一种获得确切大小的选择。 换句话来说就是，它的最大/最小宽度是一致的，高度也一样。

如果你到 Flutter 的 `box.dart` 文件中搜索 `BoxConstraints` 构造器，你会发现以下内容：

```
BoxConstraints.tight(Size size)
   : minWidth = size.width,
     maxWidth = size.width,
     minHeight = size.height,
     maxHeight = size.height;
复制代码
```

如果你重新阅读 [样例 2](#heading-5)， 它告诉我们屏幕强制 `Container` 变得和屏幕一样大。 为何屏幕能够做到这一点， 原因就是给 `Container` 传递了严格约束。

一个宽松约束换句话来说就是设置了最大宽度/高度， 但是让允许其子 widget 获得比它更小的任意大小。 换句话来说，宽松约束的最小宽度/高度为 **0**。

```
BoxConstraints.loose(Size size)
   : minWidth = 0.0,
     maxWidth = size.width,
     minHeight = 0.0,
     maxHeight = size.height;
复制代码
```

如果你访问 [样例 3](#heading-6)， 它将会告诉我们 `Center` 让红色的 `Container` 变得更小， 但是不能超出屏幕。`Center` 能够做到这一点的原因就在于 给 `Container` 的是一个宽松约束。 总的来说，`Center` 起的作用就是从其父级（屏幕）那里获得的严格约束， 为其子级（`Container`）转换为宽松约束。

## 五、了解如何为特定 widget 制定布局规则

掌握通用布局是非常重要的，但这还不够。

应用一般规则时，每个 widget 都具有很大的自由度， 所以没有办法只看 widget 的名称就知道可能它长什么样。

如果你尝试推测，可能就会猜错。 除非你已阅读 widget 的文档或研究了其源代码， 否则你无法确切知道 widget 的行为。

布局源代码通常很复杂，因此阅读文档是更好的选择。 但是当你在研究布局源代码时，可以使用 IDE 的导航功能轻松找到它。

下面是一个例子：

- 在你的代码中找到一个 `Column` 并跟进到它的源代码。 为此，请在 (Android Studio/IntelliJ) 中使用 `command+B`（macOS）或 `control+B`（Windows/Linux）。 你将跳到 `basic.dart` 文件中。由于 `Column` 扩展了 `Flex`， 请导航至 `Flex` 源代码（也位于 `basic.dart` 中）。
- 向下滚动直到找到一个名为 `createRenderObject()` 的方法。 如你所见，此方法返回一个 `RenderFlex`。 它是 `Column` 的渲染对象， 现在导航到 `flex.dart` 文件中的 `RenderFlex` 的源代码。
- 向下滚动，直到找到 `performLayout()` 方法， 由该方法执行列布局。

![image.png](https://upload-images.jianshu.io/upload_images/4118241-88c13ddfb4b558dc.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)