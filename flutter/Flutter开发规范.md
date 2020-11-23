---
layout: flutter
title: 跨平台演进及架构开篇
date: 2020-06-04 18:17:47
tags:
	- flutter
categories: 
	- flutter
---

# Flutter开发规范

## 一、命名规范

### 1.1 **要** 使用 `UpperCamelCase` 风格命名类型。

- Classes（类名）
-  enums（枚举类型）
- typedefs（类型定义）
- type parameters（类型参数）

应该把每个单词的首字母都大写（包含第一个单词），不使用分隔符。

```dart
class SliderMenu { ... }

class HttpRequest { ... }

typedef Predicate<T> = bool Function(T value);
```

这里包括使用元数据注解的类。

```dart
class Foo {
  const Foo([arg]);
}

@Foo(anArg)
class A { ... }

@Foo()
class B { ... }
```

如果注解类的构造函数是无参函数，则可以使用一个 `lowerCamelCase` 风格的常量来初始化这个注解。

```dart
const foo = Foo();

@foo
class C { ... }
```

### 1.2 **要** 使用 `UpperCamelCase` 风格类型作为扩展名

与类型命名一样，扩展名也应大写每个单词的首字母（包括第一个单词），并且不使用分隔符。

```dart
extension MyFancyList<T> on List<T> { ... }

extension SmartIterable<T> on Iterable<T> { ... }
```

 **版本提示:**

> Extensions 是 Dart 语言 2.7 版本的新增功能。详情请参考 [extensions 设计文档。](https://github.com/dart-lang/language/blob/master/accepted/2.6/static-extension-members/feature-specification.md#dart-static-extension-methods-design)

### 1.3 **要** 在`库`，`包`，`文件夹`，`源文件`中使用 `lowercase_with_underscores` 方式命名。

一些文件系统不区分大小写，所以很多项目要求文件名必须是小写字母。使用分隔符这种形式可以保证命名的可读性。使用下划线作为分隔符可确保名称仍然是有效的Dart标识符，如果语言后续支持符号导入，这将会起到非常大的帮助。

```dart
//建议使用
library peg_parser.source_scanner;

import 'file_system.dart';
import 'slider_menu.dart';
//不建议
library pegparser.SourceScanner;

import 'file-system.dart';
import 'SliderMenu.dart';
```

### 1.4 **要** 用 `lowercase_with_underscores` 风格命名库和源文件名。

**建议**

```
import 'dart:math' as math;
import 'package:angular_components/angular_components'
    as angular_components;
import 'package:js/js.dart' as js;
```

**不建议**

```
import 'dart:math' as Math;
import 'package:angular_components/angular_components'
    as angularComponents;
import 'package:js/js.dart' as JS;
```

### 1.5 **要** 使用 `lowerCamelCase` 风格来命名其他的标识符。

- 类成员
- 顶级定义
- 变量
- 参数
- 命名参数等

*除了*第一个单词，每个单词首字母都应大写，并且不使用分隔符。

```dart
var item;

HttpRequest httpRequest;

void align(bool clearItems) {
  // ...
}
```

### 1.6 **推荐**使用 `lowerCamelCase` 来命名常量。

在新的代码中，使用 `lowerCamelCase` 来命名常量，包括枚举的值。

**建议**

```
const pi = 3.14;
const defaultTimeout = 1000;
final urlScheme = RegExp('^([a-z]+):');

class Dice {
  static final numberGenerator = Random();
}
```

**不建议**

```
const PI = 3.14;
const DefaultTimeout = 1000;
final URL_SCHEME = RegExp('^([a-z]+):');

class Dice {
  static final NUMBER_GENERATOR = Random();
}
```

**注意：** 我们一开始使用 Java `SCREAMING_CAPS` 风格来命名常量。我们之所以不再使用，是因为：

- `SCREAMING_CAPS` 很多情况下看起来比较糟糕，尤其类似于 CSS 颜色这类的枚举值。
- 常量常常被修改为 final 类型的非常量变量，这种情况你还需要修改变量的名字为小写字母形式。
- 在枚举类型中自动定义的 `values` 属性为常量并且是小写字母形式的。

### 1.7 把超过两个字母的首字母大写缩略词和缩写词当做一般单词来对待。

首字母大写缩略词比较难阅读，特别是多个缩略词连载一起的时候会引起歧义。例如，一个以 `HTTPSFTP` 开头的名字，没有办法判断它是指 HTTPS FTP 还是 HTTP SFTP。

为了避免上面的情况，缩略词和缩写词要像普通单词一样首字母大写，两个字母的单词除外。（像 ID 和 Mr. 这样的双字母*缩写词*仍然像一般单词一样首字母大写。）

**建议**

```
HttpConnectionInfo
uiHandler
IOStream
HttpRequest
Id
DB
```

**不建议**

```
HTTPConnection
UiHandler
IoStream
HTTPRequest
ID
Db
```

## 二、顺序(不强制要求，但不能太乱)

为了使文件前面部分保持整洁，我们规定了关键字出现顺序的规则。每个“部分”应该使用空行分割。

### 2.1 **要** 把 “dart:” 导入语句放到其他导入语句之前。

```
import 'dart:async';
import 'dart:html';

import 'package:bar/bar.dart';
import 'package:foo/foo.dart';
```

### 2.2 **要** 把 “package:” 导入语句放到项目相关导入语句之前。

```
import 'package:bar/bar.dart';
import 'package:foo/foo.dart';

import 'util.dart';
```

### 2.3 **要** 把导出（export）语句作为一个单独的部分放到所有导入语句之后。

```dart
import 'src/error.dart';
import 'src/foo_bar.dart';

export 'src/error.dart';
```

**不建议**

```
import 'src/error.dart';
export 'src/error.dart';
import 'src/foo_bar.dart';
```

### 2.4 **要** 按照字母顺序来排序每个部分中的语句。

```
import 'package:bar/bar.dart';
import 'package:foo/foo.dart';

import 'foo.dart';
import 'foo/foo.dart';
```

**不建议**

```
import 'package:foo/foo.dart';
import 'package:bar/bar.dart';

import 'foo/foo.dart';
import 'foo.dart';
```

## 三、格式化

和其他大部分语言一样， Dart 忽略空格。但是，*人*不会。具有一致的空格风格有助于帮助我们能够用编译器相同的方式理解代码。

### **3.1 要** 使用 `dartfmt` 格式化你的代码。

格式化是一项繁琐的工作，尤其在重构过程中特别耗时。庆幸的是，你不必担心。我们提供了一个名为 [dartfmt](https://github.com/dart-lang/dart_style) 的优秀的自动代码格式化程序，它可以为你完成格式化工作。我们有一些关于它适用的规则的 [文档](https://github.com/dart-lang/dart_style/wiki/Formatting-Rules) ， Dart 中任何官方的空格处理规则由 *dartfmt 生成*。

其余格式指南用于 dartfmt 无法修复的一些规则。

### 3.2 考虑 修改你的代码让格式更友好。

无论你扔给格式化程序什么样代码，它都会尽力去处理，但是格式化程序不会创造奇迹。如果代码里有特别长的标识符，深层嵌套的表达式，混合的不同类型运算符等。格式化输出的代码可能任然很难阅读。

当有这样的情况发生时，那么就需要重新组织或简化你的代码。考虑缩短局部变量名或者将表达式抽取为一个新的局部变量。换句话说，你应该做一些手动格式化并增加代码的可读性的修改。在工作中应该把 dartfmt 看做一个合作伙伴，在代码的编写和迭代过程中互相协作输出优质的代码。

### 3.3 避免 单行超过 80 个字符。

可读性研究表明，长行的文字不易阅读，长行文字移动到下一行的开头时，眼睛需要移动更长的距离。这也是为什么报纸和杂志会使用多列样式的文字排版。

如果你真的发现你需要的文字长度超过了 80 个字符，根据我们的经验，你的代码很可能过于冗长，而且有方式可以让它更紧凑。最常见的的一种情况就是使用 `VeryLongCamelCaseClassNames` （非常长的类名字和变量名字）。当遇到这种情况时，请自问一下：“那个类型名称中的每个单词都会告诉我一些关键的内容或阻止名称冲突吗？”，如果不是，考虑删除它。

注意，dartfmt 能自动处理 99% 的情况，但是剩下的 1% 需要你自己处理。 dartfmt 不会把很长的字符串字面量分割为 80 个字符的列，所以这种情况你**需要**自己手工确保每行不超过 80 个字符。

**例外：** 当情况出现在注释或字符串是（通常在导入和导出语句中），即使文字超出行限制，也可能会保留在一行中。这样可以更轻松地搜索给定路径的源文件。

### 3.4 **要** 对所有流控制结构使用花括号。

这样可以避免 [dangling else](https://en.wikipedia.org/wiki/Dangling_else)（else悬挂）的问题。

```dart
if (isWeekDay) {
  print('Bike to work!');
} else {
  print('Go dancing or read a book!');
}
```

这里有一个例外：一个没有 `else` 的 `if` 语句，并且这个 `if` 语句以及它的执行体适合在一行中实现。在这种情况下，如果您愿意，可以不用括号：

```dart
if (arg == null) return defaultValue;
```

但是，如果执行体包含下一行，请使用大括号：

```dart
if (overflowChars != other.overflowChars) {
  return overflowChars < other.overflowChars;
}
if (overflowChars != other.overflowChars)
  return overflowChars < other.overflowChars;
```

## 四、注释规范

### 4.1 **要** 像句子一样来格式化注释

```
// Not if there is nothing before it.
if (_chunks.isEmpty) return false;
```

### 4.2 **不要** 使用块注释作用作解释说明。

**建议**

```
greet(name) {
  // Assume we have a valid name.
  print('Hi, $name!');
}
```

**不建议**

```
greet(name) {
  /* Assume we have a valid name. */
  print('Hi, $name!');
}
```

## 五、文档注释

文档注释特别有用，应为通过 [dartdoc](https://github.com/dart-lang/dartdoc) 解析这些注释可以生成 [漂亮的文档网页](https://api.dart.dev/stable)。文档注释包括所有出现在声明之前并使用 `///` 语法的注释，这些注释使用使用 dartdoc 检索。

### 5.1 要 使用 `///` 文档注释来注释成员和类型。

使用文档注释可以让 [dartdoc](https://github.com/dart-lang/dartdoc) 来为你生成代码 API 文档。

```
/// The number of characters in this chunk when unsplit.
int get length => ...
```

由于历史原因，dartdoc 支持两种格式的文档注释：`///` (“C# 格式”) 和 `/** ... */` (“JavaDoc 格式”)。我们推荐使用 `///` 是因为其更加简洁。 `/**` 和 `*/` 在多行注释中间添加了开头和结尾的两行多余内容。 `///` 在一些情况下也更加易于阅读，例如，当文档注释中包含有使用 `*` 标记的列表内容的时候。

如果你现在还在使用 JavaDoc 风格格式，请考虑使用新的格式。

### **5.2 推荐** 为公开发布的 API 编写文档注释。

### 5.3 推荐 为私有API 编写文档注释。

文档注释不仅仅适用于外部用户使用你库的公开 API. 它也同样有助于理解那些私有成员，这些私有成员会被库的其他部分调用。

### 5.4 要在文档注释开头有一个单句总结。

注释文档要以一个以用户为中心，简要的描述作为开始。通常句子片段就足够了。为读者提供足够的上下文来定位自己，并决定是否应该继续阅读，或寻找其他解决问题的方法。

```dart
/// Deletes the file at [path] from the file system.
void delete(String path) {
  ...
}
```

### 5.5 **要** 让文档注释的第一句从段落中分开。

在第一句之后添加一个空行，将其拆分为自己的段落。如果不止一个解释句子有用，请将其余部分放在后面的段落中。

这有助于您编写一个紧凑的第一句话来总结文档。此外，像Dartdoc这样的工具使用第一段作为类和类成员列表等地方的简短摘要。

```dart
/// Deletes the file at [path].
///
/// Throws an [IOError] if the file could not be found. Throws a
/// [PermissionError] if the file is present but could not be deleted.
void delete(String path) {
  ...
}
```



## 五、项目通用工具

### 统一标题栏
/widgets/mcp_title_bar.dart

### 按钮样式
/res/styles.dart/BtnStyles.class

### 文字样式
/res/styles.dart/TextStyles.class

### 自定义TabBar 指示器
因TabBar默认无法满足UI需要，所以自定义指示器
/widgets/tab_bottom_shape_indicator.dart.dart