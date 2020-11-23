---
title: dart开发语言基础（2）
date: 2020-06-11 22:21:02
tags:
	- flutter
	- dart
categories: 
	- flutter
	- dart

---



## 一、类

Dart 是支持基于 mixin 继承机制的面向对象语言，所有对象都是一个类的实例，而所有的类都继承自 [Object](https://api.dart.dev/stable/dart-core/Object-class.html) 类。基于 *mixin 的继承* 意味着每个除 Object 类之外的类都只有一个超类，一个类的代码可以在其它多个类继承中重复使用。 [Extension 方法](https://dart.cn/guides/language/language-tour#extension-methods) 是一种在不更改类或创建子类的情况下向类添加功能的方式。

### 1.1 使用类的成员

对象的 *成员* 由函数和数据（即 *方法* 和 *实例变量*）组成。方法的 *调用* 要通过对象来完成，这种方式可以访问对象的函数和数据。

使用（`.`）来访问对象的实例变量或方法：

```
var p = Point(2, 2);

// 为实例变量 y 赋值。
p.y = 3;

// 获取 y 的值。
assert(p.y == 3);

// 调用变量 p 的 distanceTo() 方法。
double distance = p.distanceTo(Point(4, 4));
```

使用 `?.` 代替 `.` 可以避免因为左边表达式为 null 而导致的问题：

```
// If p is non-null, set its y value to 4.
// 如果 p 为非空则将其属性 y 的值设为 4
p?.y = 4;
```

### 1.2 使用构造函数

可以使用 *构造函数* 来创建一个对象。构造函数的命名方式可以为 `*类名*` 或 `*类名* . *标识符* `的形式。例如下述代码分别使用 `Point()` 和 `Point.fromJson()` 两种构造器创建了 `Point` 对象：

```
var p1 = Point(2, 2);
var p2 = Point.fromJson({'x': 1, 'y': 2});
```

以下代码具有相同的效果，但是构造函数名前面的的 `new` 关键字是可选的：

```
var p1 = new Point(2, 2);
var p2 = new Point.fromJson({'x': 1, 'y': 2});
```

 **版本提示:**

从 Dart 2 开始，`new` 关键字是可选的。

一些类提供了[常量构造函数](https://dart.cn/guides/language/language-tour#constant-constructors)。使用常量构造函数，在构造函数名之前加 `const` 关键字，来创建编译时常量时：

```
var p = const ImmutablePoint(2, 2);
```

两个使用相同构造函数相同参数值构造的编译时常量是同一个对象：

```
var a = const ImmutablePoint(1, 1);
var b = const ImmutablePoint(1, 1);

assert(identical(a, b)); // 它们是同一个实例 (They are the same instance!)
```

根据使用 *常量上下文* 的场景，你可以省略掉构造函数或字面量前的 `const` 关键字。例如下面的例子中我们创建了一个常量 Map：

```
// Lots of const keywords here.
// 这里有很多 const 关键字
const pointAndLine = const {
  'point': const [const ImmutablePoint(0, 0)],
  'line': const [const ImmutablePoint(1, 10), const ImmutablePoint(-2, 11)],
};
```

根据上下文，你可以只保留第一个 `const` 关键字，其余的全部省略：

```
// Only one const, which establishes the constant context.
// 只需要一个 const 关键字，其它的则会隐式地根据上下文进行关联。
const pointAndLine = {
  'point': [ImmutablePoint(0, 0)],
  'line': [ImmutablePoint(1, 10), ImmutablePoint(-2, 11)],
};
```

但是如果无法根据上下文判断是否可以省略 `cosnt`，则不能省略掉 `const` 关键字，否则将会创建一个 **非常量对象** 例如：

```
var a = const ImmutablePoint(1, 1); // 创建一个常量 (Creates a constant)
var b = ImmutablePoint(1, 1); // 不会创建一个常量 (Does NOT create a constant)

assert(!identical(a, b)); // 这两变量并不相同 (NOT the same instance!)
```

 **版本提示:**

只有从 Dart 2 开始才能根据上下文判断省略 `const` 关键字。

### 1.3 获取对象的类型

可以使用 Object 对象的 `runtimeType` 属性在运行时获取一个对象的类型，该对象类型是 [Type](https://api.dart.dev/stable/dart-core/Type-class.html) 的实例。

```
print('The type of a is ${a.runtimeType}');
```

到目前为止，我们已经解了如何 *使用* 类。本节的其余部分将向你介绍如何 *实现* 一个类。

### 1.4 实例变量

下面是声明实例变量的示例：

```
class Point {
  double x; // 声明 double 变量 x 并初始化为 null。
  double y; // 声明 double 变量 y 并初始化为 null
  double z = 0; // 声明 double 变量 z 并初始化为 0。
}
```

所有未初始化的实例变量其值均为 `null`。

所有实例变量均会隐式地声明一个 *Getter* 方法，非 final 类型的实例变量还会隐式地声明一个 *Setter* 方法。你可以查阅 [Getter 和 Setter](https://dart.cn/guides/language/language-tour#getters-and-setters) 获取更多相关信息。

```
class Point {
  double x;
  double y;
}

void main() {
  var point = Point();
  point.x = 4; // 使用 x 的 Setter 方法。
  assert(point.x == 4); // 使用 x 的 Getter 方法。
  assert(point.y == null); // 默认值为 null。
}
```

如果你在声明一个实例变量的时候就将其初始化（而不是在构造函数或其它方法中），那么该实例变量的值就会在对象实例创建的时候被设置，该过程会在构造函数以及它的初始化器列表执行前。

### 1.5 构造函数

声明一个与类名一样的函数即可声明一个构造函数（对于[命名式构造函数](https://dart.cn/guides/language/language-tour#named-constructors) 还可以添加额外的标识符）。大部分的构造函数形式是生成式构造函数，其用于创建一个类的实例：

```
class Point {
  double x, y;

  Point(double x, double y) {
    // 还会有更好的方式来实现此逻辑，敬请期待。
    this.x = x;
    this.y = y;
  }
}
```

使用 `this` 关键字引用当前实例。

 **备忘:**

> 当且仅当命名冲突时使用 `this` 关键字才有意义，否则 Dart 会忽略 `this` 关键字。

对于大多数编程语言来说在构造函数中为实例变量赋值的过程都是类似的，而 Dart 则提供了一种特殊的语法糖来简化该步骤：

```
class Point {
  double x, y;

  // 在构造函数体执行前用于设置 x 和 y 的语法糖。
  Point(this.x, this.y);
}
```

#### 1.5.1 默认构造函数

如果你没有声明构造函数，那么 Dart 会自动生成一个无参数的构造函数并且该构造函数会调用其父类的无参数构造方法。

#### 1.5.2 构造函数不被继承

子类不会继承父类的构造函数，如果子类没有声明构造函数，那么只会有一个默认无参数的构造函数。

#### 1.5.3 命名式构造函数

可以为一个类声明多个命名式构造函数来表达更明确的意图：

```dart
class Point {
  double x, y;

  Point(this.x, this.y);

  // 命名式构造函数
  Point.origin() {
    x = 0;
    y = 0;
  }
}
```

记住构造函数是不能被继承的，这将意味着子类不能继承父类的命名式构造函数，如果你想在子类中提供一个与父类命名构造函数名字一样的命名构造函数，则需要在子类中显式地声明。

#### 1.5.4 调用父类非默认构造函数

默认情况下，子类的构造函数会调用父类的匿名无参数构造方法，并且该调用会在子类构造函数的函数体代码执行前，如果子类构造函数还有一个 [初始化列表](https://dart.cn/guides/language/language-tour#initializer-list)，那么该初始化列表会在调用父类的该构造函数之前被执行，总的来说，这三者的调用顺序如下：

1. 初始化列表
2. 父类的无参数构造函数
3. 当前类的构造函数

如果父类没有匿名无参数构造函数，那么子类必须调用父类的其中一个构造函数，为子类的构造函数指定一个父类的构造函数只需在构造函数体前使用（`:`）指定。

下面的示例中，Employee 类的构造函数调用了父类 Person 的命名构造函数。点击运行按钮执行示例代码。

<iframe src="https://dartpad.cn/embed-inline.html?id=e57aa06401e6618d4eb8&amp;split=90&amp;ga_id=non_default_superclass_constructor" width="100%" height="500px" style="box-sizing: border-box; color: rgb(33, 37, 41); font-family: &quot;Noto Sans SC&quot;, Roboto, &quot;Helvetica Neue&quot;, Helvetica, Arial, &quot;PingFang SC&quot;, &quot;Hiragino Sans GB&quot;, &quot;Heiti SC&quot;, &quot;Microsoft YaHei&quot;, &quot;WenQuanYi Micro Hei&quot;, sans-serif; font-size: 14px; font-style: normal; font-variant-ligatures: normal; font-variant-caps: normal; font-weight: 300; letter-spacing: normal; orphans: 2; text-align: left; text-indent: 0px; text-transform: none; white-space: normal; widows: 2; word-spacing: 0px; -webkit-text-stroke-width: 0px; background-color: rgb(255, 255, 255); text-decoration-style: initial; text-decoration-color: initial; border: 1px solid rgb(204, 204, 204);"></iframe>

因为参数会在子类构造函数被执行前传递给父类的构造函数，因此该参数也可以是一个表达式，比如一个函数：

```
class Employee extends Person {
  Employee() : super.fromJson(defaultData);
  // ···
}
```

 **请注意:**

> 传递给父类构造函数的参数不能使用 `this` 关键字，因为在参数传递的这一步骤，子类构造函数尚未执行，子类的实例对象也就还未初始化，因此所有的实例成员都不能被访问，但是类成员可以。

#### 1.5.5 初始化列表

除了调用父类构造函数之外，还可以在构造函数体执行之前初始化实例变量。每个实例变量之间使用逗号分隔。

```
// Initializer list sets instance variables before
// the constructor body runs.
// 使用初始化列表在构造函数体执行前设置实例变量。
Point.fromJson(Map<String, double> json)
    : x = json['x'],
      y = json['y'] {
  print('In Point.fromJson(): ($x, $y)');
}
```

 **请注意:**

> 初始化列表表达式 = 右边的语句不能使用 `this` 关键字。

在开发模式下，你可以在初始化列表中使用 `assert` 来验证输入数据：

```dart
Point.withAssert(this.x, this.y) : assert(x >= 0) {
  print('In Point.withAssert(): ($x, $y)');
}
```

初始化列表用来设置 `final` 字段是非常好用的，下面的示例中就使用初始化列表来设置了三个 `final` 变量的值。点击运行按钮执行示例代码。

<iframe src="https://dartpad.cn/embed-inline.html?id=7a9764702c0608711e08&amp;split=90&amp;ga_id=initializer_list" width="100%" height="420px" style="box-sizing: border-box; color: rgb(33, 37, 41); font-family: &quot;Noto Sans SC&quot;, Roboto, &quot;Helvetica Neue&quot;, Helvetica, Arial, &quot;PingFang SC&quot;, &quot;Hiragino Sans GB&quot;, &quot;Heiti SC&quot;, &quot;Microsoft YaHei&quot;, &quot;WenQuanYi Micro Hei&quot;, sans-serif; font-size: 14px; font-style: normal; font-variant-ligatures: normal; font-variant-caps: normal; font-weight: 300; letter-spacing: normal; orphans: 2; text-align: left; text-indent: 0px; text-transform: none; white-space: normal; widows: 2; word-spacing: 0px; -webkit-text-stroke-width: 0px; background-color: rgb(255, 255, 255); text-decoration-style: initial; text-decoration-color: initial; border: 1px solid rgb(204, 204, 204);"></iframe>

#### 1.5.6 重定向构造函数

有时候类中的构造函数会调用类中其它的构造函数，该重定向构造函数没有函数体，只需在函数签名后使用（:）指定需要重定向到的其它构造函数即可：

```
class Point {
  double x, y;

  // 该类的主构造函数。
  Point(this.x, this.y);

  // 委托实现给主构造函数。
  Point.alongXAxis(double x) : this(x, 0);
}
```

#### 1.5.7 常量构造函数

如果类生成的对象都是不会变的，那么可以在生成这些对象时就将其变为编译时常量。你可以在类的构造函数前加上 `const` 关键字并确保所有实例变量均为 `final` 来实现该功能。

```
class ImmutablePoint {
  static final ImmutablePoint origin =
      const ImmutablePoint(0, 0);

  final double x, y;

  const ImmutablePoint(this.x, this.y);
}
```

常量构造函数创建的实例并不总是常量，具体可以参考[使用构造函数](https://dart.cn/guides/language/language-tour#using-constructors)章节。

#### 1.5.8 工厂构造函数

使用 `factory` 关键字标识类的构造函数将会令该构造函数变为工厂构造函数，这将意味着使用该构造函数构造类的实例时并非总是会返回新的实例对象。例如，工厂构造函数可能会从缓存中返回一个实例，或者返回一个子类型的实例。

以下示例演示了从缓存中返回对象的工厂构造函数：

```
class Logger {
  final String name;
  bool mute = false;

  // _cache 变量是库私有的，因为在其名字前面有下划线。
  static final Map<String, Logger> _cache =
      <String, Logger>{};

  factory Logger(String name) {
    return _cache.putIfAbsent(
        name, () => Logger._internal(name));
  }

  Logger._internal(this.name);

  void log(String msg) {
    if (!mute) print(msg);
  }
}
```

 **备忘:**

> 在工厂构造函数中无法访问 `this`。

工厂构造函的调用方式与其他构造函数一样：

```
var logger = Logger('UI');
logger.log('Button clicked');
```

### 1.6 方法

方法是对象提供行为的函数。

#### 1.6.1 实例方法

对象的实例方法可以访问实例变量和 `this`。下面的 `distanceTo()` 方法就是一个实例方法的例子：

```
import 'dart:math';

class Point {
  double x, y;

  Point(this.x, this.y);

  double distanceTo(Point other) {
    var dx = x - other.x;
    var dy = y - other.y;
    return sqrt(dx * dx + dy * dy);
  }
}
```

#### 1.6.2 Getter 和 Setter

Getter 和 Setter 是一对用来读写对象属性的特殊方法，上面说过实例对象的每一个属性都有一个隐式的 Getter 方法，如果为非 final 属性的话还会有一个 Setter 方法，你可以使用 `get` 和 `set` 关键字为额外的属性添加 Getter 和 Setter 方法：

```
class Rectangle {
  double left, top, width, height;

  Rectangle(this.left, this.top, this.width, this.height);

  // 定义两个计算产生的属性：right 和 bottom。
  double get right => left + width;
  set right(double value) => left = value - width;
  double get bottom => top + height;
  set bottom(double value) => top = value - height;
}

void main() {
  var rect = Rectangle(3, 4, 20, 15);
  assert(rect.left == 3);
  rect.right = 12;
  assert(rect.left == -8);
}
```

使用 Getter 和 Setter 的好处是，你可以先使用你的实例变量，过一段时间过再将它们包裹成方法且不需要改动任何代码，即先定义后更改且不影响原有逻辑。

 **备忘:**

> 像自增（++）这样的操作符不管是否定义了 Getter 方法都会正确地执行。为了避免一些不必要的异常情况，运算符只会调用 Getter 一次，然后将其值存储在一个临时变量中。

### 1.7 抽象类

使用关键字 `abstract` 标识类可以让该类成为 *抽象类*，抽象类将无法被实例化。抽象类常用于声明接口方法、有时也会有具体的方法实现。如果想让抽象类同时可被实例化，可以为其定义 [工厂构造函数](https://dart.cn/guides/language/language-tour#工厂构造函数)。

抽象类常常会包含 [抽象方法](https://dart.cn/guides/language/language-tour#abstract-methods)。下面是一个声明具有抽象方法的抽象类示例：

```
// This class is declared abstract and thus
// can't be instantiated.
// 该类被声明为抽象的，因此它不能被实例化。
abstract class AbstractContainer {
  // 定义构造函数、字段、方法等……

  void updateChildren(); // 抽象方法。
```

### 1.8 隐式接口

每一个类都隐式地定义了一个接口并实现了该接口，这个接口包含所有这个类的实例成员以及这个类所实现的其它接口。如果想要创建一个 A 类支持调用 B 类的 API 且不想继承 B 类，则可以实现 B 类的接口。

一个类可以通过关键字 `implements` 来实现一个或多个接口并实现每个接口定义的 API：

```
// A person. The implicit interface contains greet().
// Person 类的隐式接口中包含 greet() 方法。
class Person {
  // _name 变量同样包含在接口中，但它只是库内可见的。
  final _name;

  // 构造函数不在接口中。
  Person(this._name);

  // greet() 方法在接口中。
  String greet(String who) => '你好，$who。我是$_name。';
}

// Person 接口的一个实现。
class Impostor implements Person {
  get _name => '';

  String greet(String who) => '你好$who。你知道我是谁吗？';
}

String greetBob(Person person) => person.greet('小芳');

void main() {
  print(greetBob(Person('小芸')));
  print(greetBob(Impostor()));
}
```

如果需要实现多个类接口，可以使用逗号分割每个接口类：

```
class Point implements Comparable, Location {...}
```

```
class Point implements Comparable, Location {...}
```

### 1.8 扩展一个类

使用 `extends` 关键字来创建一个子类，并可使用 `super` 关键字引用一个父类：

```dart
class Television {
  void turnOn() {
    _illuminateDisplay();
    _activateIrSensor();
  }
  // ···
}

class SmartTelevision extends Television {
  void turnOn() {
    super.turnOn();
    _bootNetworkInterface();
    _initializeMemory();
    _upgradeApps();
  }
  // ···
}
```

#### 1.8.1 重写类成员

子类可以重写父类的实例方法、Getter 以及 Setter 方法。你可以使用 `@override` 注解来表示你重写了一个成员：

```dart
class SmartTelevision extends Television {
  @override
  void turnOn() {...}
  // ···
}
```

限定方法参数以及实例变量的类型可以让代码更加 [类型安全](https://dart.cn/guides/language/sound-dart)，你可以使用 [协变关键字](https://dart.cn/guides/language/sound-problems#the-covariant-keyword)。

#### 重写运算符

可以在一个类中重写下表所罗列出的所有运算符。比如如果定一个 Vector 表示矢量的类，那么可以考虑重写 `+` 操作符来处理两个矢量的相加。

| `<`  | `+`  | `|`  | `[]`  |
| ---- | ---- | ---- | ----- |
| `>`  | `/`  | `^`  | `[]=` |
| `<=` | `~/` | `&`  | `~`   |
| `>=` | `*`  | `<<` | `==`  |
| `–`  | `%`  | `>>` |       |

 **备忘:**

必须要注意的是 `!=` 操作符并不是一个可被重写的操作符。表达式 `e1 != e2` 仅仅是 `!(e1 == e2)` 的一个语法糖。

下面是重写 `+` 和 `-` 操作符的例子：

```
class Vector {
  final int x, y;

  Vector(this.x, this.y);

  Vector operator +(Vector v) => Vector(x + v.x, y + v.y);
  Vector operator -(Vector v) => Vector(x - v.x, y - v.y);

  // 运算符 == 和 hashCode 的实现未在这里展示，详情请查看下方说明。
  // ···
}

void main() {
  final v = Vector(2, 3);
  final w = Vector(2, 2);

  assert(v + w == Vector(4, 5));
  assert(v - w == Vector(0, 1));
}
```

如果重写 `==` 操作符，必须也同时重写对象 `hashCode` 的 Getter 方法。你可以查阅 [实现映射键](https://dart.cn/guides/libraries/library-tour#implementing-map-keys) 获取更多关于重写的 `==` 和 `hashCode` 的例子。

你也可以查阅 [扩展一个类](https://dart.cn/guides/language/language-tour#extending-a-class)获取更多关于重写的信息。

#### noSuchMethod()

如果调用了对象上不存在的方法或实例变量将会触发 `noSuchMethod` 方法，你可以重写 `noSuchMethod` 方法来追踪和记录这一行为：

```dart
class A {
  // 除非你重写 noSuchMethod，否则调用一个不存在的成员会导致 NoSuchMethodError。
  @override
  void noSuchMethod(Invocation invocation) {
  print('你尝试使用一个不存在的成员：' +
  '${invocation.memberName}');
  }
}
```

你不能调用一个未实现的方法除非下面其中的一个条件成立：

- 接收方是静态的 `dynamic` 类型。
- 接收方具有静态类型，定义了未实现的方法（抽象亦可），并且接收方的动态类型实现了 `noSuchMethod` 方法且具体的实现与 `Object` 中的不同。

你可以查阅 [noSuchMethod 转发规范](https://github.com/dart-lang/sdk/blob/master/docs/language/informal/nosuchmethod-forwarding.md)获取更多相关信息。

### 1.9 Extension 方法

Dart 2.7 中引入的 Extension 方法是向现有库添加功能的一种方式。你可能甚至都不知道有 Extension 方法。例如，当您在 IDE 中使用代码完成功能时，它建议将 Extension 方法与常规方法一起使用。

这里是一个在 `String` 中使用 extension 方法的样例，我们取名为 `parseInt()`，它在 `string_apis.dart` 中定义：

```
import 'string_apis.dart';
...
print('42'.padLeft(5)); // Use a String method.
print('42'.parseInt()); // Use an extension method.
```

有关使用以及实现 extension 方法的详细信息，请参阅 [extension methods 页面](https://dart.cn/guides/language/extension-methods).

### 1.10 使用枚举

使用关键字 `enum` 来定义枚举类型：

```
enum Color { red, green, blue }
```

每一个枚举值都有一个名为 `index` 成员变量的 Getter 方法，该方法将会返回以 0 为基准索引的位置值。例如，第一个枚举值的索引是 0 ，第二个枚举值的索引是 1。以此类推。

```
assert(Color.red.index == 0);
assert(Color.green.index == 1);
assert(Color.blue.index == 2);
```

可以使用枚举类的 `values` 方法获取一个包含所有枚举值的列表：

```
List<Color> colors = Color.values;
assert(colors[2] == Color.blue);
```

你可以在 [Switch 语句](https://dart.cn/guides/language/language-tour#switch-和-case)中使用枚举，但是需要注意的是必须处理枚举值的每一种情况，即每一个枚举值都必须成为一个 case 子句，不然会出现警告：

```
var aColor = Color.blue;

switch (aColor) {
  case Color.red:
    print('红如玫瑰！');
    break;
  case Color.green:
    print('绿如草原！');
    break;
  default: // 没有该语句会出现警告。
    print(aColor); // 'Color.blue'
}
```

枚举类型有如下两个限制：

- 枚举不能成为子类，也不可以 mix in，你也不可以实现一个枚举。
- 不能显式地实例化一个枚举类。

你可以查阅 [Dart 编程语言规范][]获取更多相关信息。

### 1.11 使用 Mixin 为类添加功能

Mixin 是一种在多重继承中复用某个类中代码的方法模式。

使用 `with` 关键字并在其后跟上 Mixin 类的名字来使用 Mixin 模式：

```dart
class Musician extends Performer with Musical {
  // ···
}

class Maestro extends Person
    with Musical, Aggressive, Demented {
  Maestro(String maestroName) {
    name = maestroName;
    canConduct = true;
  }
}
```

定义一个类继承自 Object 并且不为该类定义构造函数，这个类就是 Mixin 类，除非你想让该类与普通的类一样可以被正常地使用，否则可以使用关键字 `mixin` 替代 `class` 让其成为一个单纯的 Mixin 类：

```
mixin Musical {
  bool canPlayPiano = false;
  bool canCompose = false;
  bool canConduct = false;

  void entertainMe() {
    if (canPlayPiano) {
      print('Playing piano');
    } else if (canConduct) {
      print('Waving hands');
    } else {
      print('Humming to self');
    }
  }
}
```

可以使用关键字 `on` 来指定哪些类可以使用该 Mixin 类，比如有 Mixin 类 A，但是 A 只能被 B 类使用，则可以这样定义 A：

```
mixin MusicalPerformer on Musician {
  // ···
}
```

 **版本提示:**

`mixin` 关键字在 Dart 2.1 中才被引用支持。早期版本中的代码通常使用 `abstract class` 代替。你可以查阅 [Dart SDK 变](https://github.com/dart-lang/sdk/blob/master/CHANGELOG.md)

### 1.12 类变量和方法

使用关键字 `static` 可以声明类变量或类方法。

#### 1.12.1静态变量

静态变量（即类变量）常用于声明类范围内所属的状态变量和常量：

```
class Queue {
  static const initialCapacity = 16;
  // ···
}

void main() {
  assert(Queue.initialCapacity == 16);
}
```

静态变量在其首次被使用的时候才被初始化。

#### 1.12.2 静态方法

静态方法（即类方法）不能被一个类的实例访问，同样地，静态方法内也不可以使用 `this`：

```
import 'dart:math';

class Point {
  double x, y;
  Point(this.x, this.y);

  static double distanceBetween(Point a, Point b) {
    var dx = a.x - b.x;
    var dy = a.y - b.y;
    return sqrt(dx * dx + dy * dy);
  }
}

void main() {
  var a = Point(2, 2);
  var b = Point(4, 4);
  var distance = Point.distanceBetween(a, b);
  assert(2.8 < distance && distance < 2.9);
  print(distance);
}
```

 **备忘:**

对于一些通用或常用的静态方法，应该将其定义为顶级函数而非静态方法。

可以将静态方法作为编译时常量。例如，你可以将静态方法作为一个参数传递给一个常量构造函数。

## 二、泛型

如果你查看数组的 API 文档，你会发现数组 [List](https://api.dart.dev/stable/dart-core/List-class.html) 的实际类型为 `List<E>`。 <…> 符号表示数组是一个 *泛型*（或 *参数化类型*） [通常](https://dart.cn/guides/language/effective-dart/design#do-follow-existing-mnemonic-conventions-when-naming-type-parameters) 使用一个字母来代表类型参数，比如E、T、S、K 和 V 等等。

### 2.1 为什么使用泛型？

泛型常用于需要要求类型安全的情况，但是它也会对代码运行有好处：

- 适当地指定泛型可以更好地帮助代码生成。
- 使用泛型可以减少代码重复。

比如你想声明一个只能包含 String 类型的数组，你可以将该数组声明为 `List<String>`（读作“字符串类型的 list”），这样的话就可以很容易避免因为在该数组放入非 String 类变量而导致的诸多问题，同时编译器以及其他阅读代码的人都可以很容易地发现并定位问题：

```
var names = List<String>();
names.addAll(['Seth', 'Kathy', 'Lars']);
names.add(42); // Error
```

另一个使用泛型的原因是可以减少重复代码。泛型可以让你在多个不同类型实现之间共享同一个接口声明，比如下面的例子中声明了一个类用于缓存对象的接口：

```
abstract class ObjectCache {
  Object getByKey(String key);
  void setByKey(String key, Object value);
}
```

不久后你可能又会想专门为 String 类对象做一个缓存，于是又有了专门为 String 做缓存的类：

```
abstract class StringCache {
  String getByKey(String key);
  void setByKey(String key, String value);
}
```

如果过段时间你又想为数字类型也创建一个类，那么就会有很多诸如此类的代码……

这时候可以考虑使用泛型来声明一个类，让不同类型的缓存实现该类做出不同的具体实现即可：

```
abstract class Cache<T> {
  T getByKey(String key);
  void setByKey(String key, T value);
}
```

在上述代码中，T 是一个替代类型。其相当于类型占位符，在开发者调用该接口的时候会指定具体类型。

### 2.2 使用集合字面量

List、Set 以及 Map 字面量也可以是参数化的。定义参数化的 List 只需在中括号前添加 `<*type*>`；定义参数化的 Map 只需要在大括号前添加 `<*keyType*, *valueType*>`：

```
var names = <String>['小芸', '小芳', '小民'];
var uniqueNames = <String>{'小芸', '小芳', '小民'};
var pages = <String, String>{
  'index.html': '主页',
  'robots.txt': '网页机器人提示',
  'humans.txt': '我们是人类，不是机器'
};
```

### 2.3 使用类型参数化的构造函数

在调用构造方法时也可以使用泛型，只需在类名后用尖括号（`<...>`）将一个或多个类型包裹即可：

```
var nameSet = Set<String>.from(names);
```

下面代码创建了一个键为 Int 类型，值为 View 类型的 Map 对象：

```
var views = Map<int, View>();
```

### 2.4 泛型集合以及它们所包含的类型

Dart的泛型类型是 *固化的*，这意味着即便在运行时也会保持类型信息：

```
var names = List<String>();
names.addAll(['小芸', '小芳', '小民']);
print(names is List<String>); // true
```

 **备忘:**

> 与 Java 不同的是，Java 中的泛型是类型 *擦除* 的，这意味着泛型类型会在运行时被移除。在 Java 中你可以判断对象是否为 List 但不可以判断对象是否为 `List<String>`。

### 2.5 限制参数化类型

有时使用泛型的时候可能会想限制泛型的类型范围，这时候可以使用 `extends` 关键字：

```dart
class Foo<T extends SomeBaseClass> {
  // 具体实现……
  String toString() => "'Foo<$T>' 的实例";
}

class Extender extends SomeBaseClass {...}
```

这时候就可以使用 `SomeBaseClass` 或者它的子类来作为泛型参数：

```dart
var someBaseClassFoo = Foo<SomeBaseClass>();
var extenderFoo = Foo<Extender>();
```

这时候也可以指定无参数的泛型，这时无参数泛型的类型则为 `Foo<SomeBaseClass>`：

```
var foo = Foo();
print(foo); // 'Foo<SomeBaseClass>' 的实例 (Instance of 'Foo<SomeBaseClass>')
```

将非 `SomeBaseClass` 的类型作为泛型参数则会导致编译错误：

```dart
var foo = Foo<Object>();
```

### 2.6 使用泛型方法

起初 Dart 只支持在类的声明时指定泛型，现在同样也可以在方法上使用泛型，称之为 *泛型方法*：

```dart
T first<T>(List<T> ts) {
  // 处理一些初始化工作或错误检测……
  T tmp = ts[0];
  // 处理一些额外的检查……
  return tmp;
}
```

方法 `first<T>` 的泛型 `T` 可以在如下地方使用：

- 函数的返回值类型 (`T`)。
- 参数的类型 (`List<T>`)。
- 局部变量的类型 (`T tmp`)。

你可以查阅[使用泛型函数](https://github.com/dart-lang/sdk/blob/master/pkg/dev_compiler/doc/GENERIC_METHODS.md)获取更多关于泛型的信息。

## 三、库和可见性

`import` 和 `library` 关键字可以帮助你创建一个模块化和可共享的代码库。代码库不仅只是提供 API 而且还起到了封装的作用：以下划线（_）开头的成员仅在代码库中可见。*每个 Dart 程序都是一个库*，即便没有使用关键字 `library` 指定。

Dart 的库可以使用[包](https://dart.cn/guides/packages)工具来发布和部署。

### 3.1 使用库

使用 `import` 来指定命名空间以便其它库可以访问。

比如你可以导入代码库 [dart:html](https://api.dart.dev/stable/dart-html) 来使用 Dart Web 中相关 API：

```
import 'dart:html';
```

`import` 的唯一参数是用于指定代码库的 URI，对于 Dart 内置的库，使用 `dart:xxxxxx` 的形式。而对于其它的库，你可以使用一个文件系统路径或者以 `package:xxxxxx` 的形式。`package:xxxxxx` 指定的库通过包管理器（比如 pub 工具）来提供：

```
import 'package:test/test.dart';
```

 **备忘:**

*URI* 代表统一资源标识符。

*URL*（统一资源定位符）是一种常见的URI。

#### 3.1.1 指定库前缀

如果你导入的两个代码库有冲突的标识符，你可以为其中一个指定前缀。比如如果 library1 和 library2 都有 Element 类，那么可以这么处理：

```
import 'package:lib1/lib1.dart';
import 'package:lib2/lib2.dart' as lib2;

// 使用 lib1 的 Element 类。
Element element1 = Element();

// 使用 lib2 的 Element 类。
lib2.Element element2 = lib2.Element();
```

#### 3.1.2 导入库的一部分

如果你只想使用代码库中的一部分，你可以有选择地导入代码库。例如：

```
// 只导入 lib1 中的 foo。(Import only foo).
import 'package:lib1/lib1.dart' show foo;

// 导入 lib2 中除了 foo 外的所有。
import 'package:lib2/lib2.dart' hide foo;
```

#### 3.1.3 延迟加载库

*延迟加载*（也常称为 *懒加载*）允许应用在需要时再去加载代码库，下面是可能使用到延迟加载的场景：

- 为了减少应用的初始化时间。
- 处理 A/B 测试，比如测试各种算法的不同实现。
- 加载很少会使用到的功能，比如可选的屏幕和对话框。

> **目前只有 dart2js 支持延迟加载** Flutter、Dart VM以及 DartDevc 目前都不支持延迟加载。你可以查阅 [issue #33118](https://github.com/dart-lang/sdk/issues/33118) 和 [issue #27776](https://github.com/dart-lang/sdk/issues/27776) 获取更多的相关信息。

使用 `deferred as` 关键字来标识需要延时加载的代码库：

```
import 'package:greetings/hello.dart' deferred as hello;
```

当实际需要使用到库中 API 时先调用 `loadLibrary` 函数加载库：

```
Future greet() async {
  await hello.loadLibrary();
  hello.printGreeting();
}
```

在前面的代码，使用 `await` 关键字暂停代码执行直到库加载完成。更多关于 `async` 和 `await` 的信息请参考[异步支持](https://dart.cn/guides/language/language-tour#asynchrony-support)。

`loadLibrary` 函数可以调用多次也没关系，代码库只会被加载一次。

当你使用延迟加载的时候需要牢记以下几点：

- 延迟加载的代码库中的常量需要在代码库被加载的时候才会导入，未加载时是不会导入的。
- 导入文件的时候无法使用延迟加载库中的类型。如果你需要使用类型，则考虑吧接口类型转移到另一个库中然后让两个库都分别导入这个接口库。
- Dart会隐式地将 `loadLibrary` 方法导入到使用了 `deferred as *命名空间*` 的类中。`loadLibrary` 函数返回的是一个 [Future](https://dart.cn/guides/libraries/library-tour#future)。

### 3.2 实现库

查阅[创建依赖库包](https://dart.cn/guides/libraries/create-library-packages)可以获取有关如何实现库包的建议，包括：

- 如何组织库的源文件。
- 如何使用 `export` 命令。
- 何时使用 `part` 命令。
- 何时使用 `library` 命令。
- 如何使用倒入和导出命令实现多平台的库支持。

## 四、异步支持

Dart 代码库中有大量返回 [Future](https://api.dart.dev/stable/dart-async/Future-class.html) 或 [Stream](https://api.dart.dev/stable/dart-async/Stream-class.html) 对象的函数，这些函数都是 *异步* 的，它们会在耗时操作（比如I/O）执行完毕前直接返回而不会等待耗时操作执行完毕。

`async` 和 `await` 关键字用于实现异步编程，并且让你的代码看起来就像是同步的一样。

### 4.1 处理 Future

可以通过下面两种方式，获得 Future 执行完成的结果：

- 使用 `async` 和 `await`；
- 使用 Future API，具体描述，参考[库概览](https://dart.cn/guides/libraries/library-tour#future)。

使用 `async` 和 `await` 的代码是异步的，但是看起来有点像同步代码。例如，下面的代码使用 `await` 等待异步函数的执行结果。

```
await lookUpVersion();
```

必须在带有 async 关键字的 *异步函数* 中使用 `await`：

```dart
Future checkVersion() async {
  var version = await lookUpVersion();
  // 使用 version 继续处理逻辑
}
```

 **备忘:**

> 尽管异步函数可以处理耗时操作，但是它并不会等待这些耗时操作完成，异步函数执行时会在其遇到第一个 `await` 表达式（[详情见](https://github.com/dart-lang/sdk/blob/master/docs/newsletter/20170915.md#synchronous-async-start)）的时候返回一个 Future 对象，然后等待 await 表达式执行完毕后继续执行。

使用 `try`、`catch` 以及 `finally` 来处理使用 `await` 导致的异常：

```
try {
  version = await lookUpVersion();
} catch (e) {
  // 无法找到版本时做出的反应
}
```

你可以在异步函数中多次使用 `await` 关键字。例如，下面代码中等待了三次函数结果：

```
var entrypoint = await findEntrypoint();
var exitCode = await runExecutable(entrypoint, args);
await flushThenExit(exitCode);
await *表达式的返回值通常是一个 Future 对象；如果不是的话也会自动将其包裹在一个 Future 对象里。Future 对象代表一个“承诺”，`await \*表达式\*`会阻塞直到需要的对象返回。*
```

**如果在使用 `await` 时导致编译错误，请确保 `await` 在一个异步函数中使用**。例如，如果想在 main() 函数中使用 `await`，那么 `main()` 函数就必须使用 `async` 关键字标识。

```dart
Future main() async {
  checkVersion();
  print('在 Main 函数中执行：版本是 ${await lookUpVersion()}');
}
```

### 4.2 声明异步函数

定义 *异步函数* 只需在普通方法上加上 `async` 关键字即可。

将关键字 `async` 添加到函数并让其返回一个 Future 对象。假设有如下返回 String 对象的方法：

```
String lookUpVersion() => '1.0.0';
```

将其改为异步函数，返回值是 Future：

```
Future<String> lookUpVersion() async => '1.0.0';
```

注意，函数体不需要使用 Future API。如有必要，Dart 会创建 Future 对象。

如果函数没有返回有效值，需要设置其返回类型为 `Future<void>`。

关于 futures、`async` 和 `await` 的使用介绍，可以参见这个 codelab: [asynchronous programming codelab](https://dart.cn/codelabs/async-await)。

### 4.3 处理 Stream

如果想从 Stream 中获取值，可以有两种选择：

- 使用 `async` 关键字和一个 *异步循环*（使用 `await for` 关键字标识）。
- 使用 Stream API。详情参考[库概览](https://dart.cn/guides/libraries/library-tour#stream)。

 **备忘:**

在使用 `await for` 关键字前，确保其可以令代码逻辑更加清晰并且是真的需要等待所有的结果执行完毕。例如，通常不应该在 UI 事件监听器上使用 `await for` 关键字，因为 UI 框架发出的事件流是无穷尽的。

使用 await for 定义异步循环看起来是这样的：

```
await for (varOrType identifier in expression) {
  // 每当 Stream 发出一个值时会执行
}
```

`*表达式*` 的类型必须是 Stream。执行流程如下：

1. 等待直到 Stream 返回一个数据。
2. 使用 1 中 Stream 返回的数据执行循环体。
3. 重复 1、2 过程直到 Stream 数据返回完毕。

使用 `break` 和 `return` 语句可以停止接收 Stream 数据，这样就跳出了循环并取消注册监听 Stream。

**如果在实现异步 for 循环时遇到编译时错误，请检查确保 `await for` 处于异步函数中。** 例如，要在应用程序的 `main()` 函数中使用异步 for 循环，`main()` 函数体必须标记为 `async`：

```dart
Future main() async {
  // ...
  await for (var request in requestServer) {
    handleRequest(request);
  }
  // ...
}
```

你可以查阅库概览中有关 [dart:async](https://dart.cn/guides/libraries/library-tour#dartasync---asynchronous-programming) 的部分获取更多有关异步编程的信息。

## 五、生成器

当你需要延迟地生成一连串的值时，可以考虑使用 *生成器函数*。Dart 内置支持两种形式的生成器方法：

- **同步** 生成器：返回一个 **[Iterable](https://api.dart.dev/stable/dart-core/Iterable-class.html)** 对象。
- **异步** 生成器：返回一个 **[Stream](https://api.dart.dev/stable/dart-async/Stream-class.html)** 对象。

通过在函数上加 `sync*` 关键字并将返回值类型设置为 Iterable 来实现一个 **同步** 生成器函数，在函数中使用 `yield` 语句来传递值：

```
Iterable<int> naturalsTo(int n) sync* {
  int k = 0;
  while (k < n) yield k++;
}
```

实现 **异步** 生成器函数与同步类似，只不过关键字为 `async*` 并且返回值为 Stream：

```
Stream<int> asynchronousNaturalsTo(int n) async* {
  int k = 0;
  while (k < n) yield k++;
}
```

如果生成器是递归调用的，可是使用 `yield*` 语句提升执行性能：

```
Iterable<int> naturalsDownFrom(int n) sync* {
  if (n > 0) {
    yield n;
    yield* naturalsDownFrom(n - 1);
  }
}
```

## 六、可调用类

通过实现类的 `call()` 方法，允许使用类似函数调用的方式来使用该类的实例。

在下面的示例中，`WannabeFunction` 类定义了一个 call() 函数，函数接受三个字符串参数，函数体将三个字符串拼接，字符串间用空格分割，并在结尾附加了一个感叹号。单击运行按钮执行代码。

<iframe src="https://dartpad.cn/embed-inline.html?id=3723fcf3915ca935d13393b8a9f86fd5&amp;ga_id=callable_classes" width="100%" height="350px" style="box-sizing: border-box; color: rgb(33, 37, 41); font-family: &quot;Noto Sans SC&quot;, Roboto, &quot;Helvetica Neue&quot;, Helvetica, Arial, &quot;PingFang SC&quot;, &quot;Hiragino Sans GB&quot;, &quot;Heiti SC&quot;, &quot;Microsoft YaHei&quot;, &quot;WenQuanYi Micro Hei&quot;, sans-serif; font-size: 14px; font-style: normal; font-variant-ligatures: normal; font-variant-caps: normal; font-weight: 300; letter-spacing: normal; orphans: 2; text-align: left; text-indent: 0px; text-transform: none; white-space: normal; widows: 2; word-spacing: 0px; -webkit-text-stroke-width: 0px; background-color: rgb(255, 255, 255); text-decoration-style: initial; text-decoration-color: initial; border: 1px solid rgb(204, 204, 204);"></iframe>



## 七、隔离区

大多数计算机中，甚至在移动平台上，都在使用多核 CPU。为了有效利用多核性能，开发者一般使用共享内存的方式让线程并发地运行。然而，多线程共享数据通常会导致很多潜在的问题，并导致代码运行出错。

为了解决多线程带来的并发问题，Dart 使用 isolates 替代线程，所有的 Dart 代码均运行在一个 *isolates* 中。每一个 isolates 有它自己的堆内存以确保其状态不被其它 isolates 访问。

你可以查阅下面的文档获取更多相关信息：

- [Dart 异步编程：隔离区和事件循环](https://medium.com/dartlang/dart-asynchronous-programming-isolates-and-event-loops-bffc3e296a6a)
- [dart:isolate API 参考](https://api.dart.dev/stable/dart-isolate)介绍了 [Isolate.spawn()](https://api.dart.dev/stable/dart-isolate/Isolate/spawn.html) 和 [TransferableTypedData](https://api.dart.dev/stable/dart-isolate/TransferableTypedData-class.html) 的用法
- [Background parsing](https://flutter.cn/docs/cookbook/networking/background-parsing) cookbook on the Flutter site
- Flutter 网站上关于[后台解析](https://flutter.cn/docs/cookbook/networking/background-parsing)的 Cookbook

## 八、类型定义(重要)

在 Dart 语言中，函数与 String 和 Number 一样都是对象，可以使用 *类型定义*（或者叫 *方法类型别名*）来为函数的类型命名。使用函数命名将该函数类型的函数赋值给一个变量时，类型定义将会保留相关的类型信息。

比如下面的代码没有使用类型定义：

```
class SortedCollection {
  Function compare;

  SortedCollection(int f(Object a, Object b)) {
    compare = f;
  }
}

// 简单的不完整实现。
int sort(Object a, Object b) => 0;

void main() {
  SortedCollection coll = SortedCollection(sort);

  // 我们知道 compare 是一个函数类型的变量，但是具体是什么样的函数却不得而知。
  assert(coll.compare is Function);
}
```

上述代码中，当将参数 `f` 赋值给 `compare` 时，函数的类型信息丢失了，这里 `f` 这个函数的类型为 `(Object, Object) → int`（→ 代表返回），当然该类型也是一个 Function 的子类，但是将 `f` 赋值给 `compare` 后，`f` 的类型 `(Object, Object) → int` 就会丢失。我们可以使用 `typedef` 显式地保留类型信息：

```
typedef Compare = int Function(Object a, Object b);

class SortedCollection {
  Compare compare;

  SortedCollection(this.compare);
}

// 简单的不完整实现。
int sort(Object a, Object b) => 0;

void main() {
  SortedCollection coll = SortedCollection(sort);
  assert(coll.compare is Function);
  assert(coll.compare is Compare);
}
```

 **备忘:**

> 目前类型定义只能用在函数类型上，但是将来可能会有变化。

因为类型定义只是别名，因此我们可以使用它判断任意函数类型的方法：

```
typedef Compare<T> = int Function(T a, T b);

int sort(int a, int b) => a - b;

void main() {
  assert(sort is Compare<int>); // True!
}
```

## 九、元数据

使用元数据可以为代码增加一些额外的信息。元数据注解以 `@` 开头，其后紧跟一个编译时常量（比如 `deprecated`）或者调用一个常量构造函数。

Dart 中有两个注解是所有代码都可以使用的：`@deprecated` 和 `@override`。你可以查阅[扩展一个类](https://dart.cn/guides/language/language-tour#extending-a-class)获取有关 `@override` 的使用示例。下面是使用 `@deprecated` 的示例：

```dart
class Television {
  /// _弃用: 使用 [turnOn] 替代_
  @deprecated
  void activate() {
    turnOn();
  }

  /// 打开 TV 的电源。
  void turnOn() {...}
}
```

可以自定义元数据注解。下面的示例定义了一个带有两个参数的 @todo 注解：

```
library todo;

class Todo {
  final String who;
  final String what;

  const Todo(this.who, this.what);
}
```

使用 @todo 注解的示例：

```
import 'todo.dart';

@Todo('seth', 'make this do something')
void doSomething() {
  print('do something');
}
```

元数据可以在 library、class、typedef、type parameter、constructor、factory、function、field、parameter 或者 variable 声明之前使用，也可以在 import 或 export 之前使用。可使用反射在运行时获取元数据信息。

## 十、注释

Dart 支持单行注释、多行注释和文档注释。

### 10.1 单行注释

单行注释以 `//` 开始。所有在 `//` 和该行结尾之间的内容被编译器忽略。

```
void main() {
  // TODO: refactor into an AbstractLlamaGreetingFactory?
  print('Welcome to my Llama farm!');
}
```

### 10.2 多行注释

多行注释以 `/*` 开始，以 `*/` 结尾。所有在 `/*` 和 `*/` 之间的内容被编译器忽略（不会忽略文档注释）。多行注释可以嵌套。

```
void main() {
  /*
   * This is a lot of work. Consider raising chickens.

  Llama larry = Llama();
  larry.feed();
  larry.exercise();
  larry.clean();
   */
}
```

### 10.3 文档注释

文档注释可以是多行注释，也可以是单行注释，文档注释以 `///` 或者 `/**` 开始。在连续行上使用 `///` 与多行文档注释具有相同的效果。

在文档注释中，除非用中括号括起来，否则 Dart 编译器会忽略所有文本。使用中括号可以引用类、方法、字段、顶级变量、函数、和参数。括号中的符号会在已记录的程序元素的词法域中进行解析。

下面是一个引用其他类和成员的文档注释：

```
/// A domesticated South American camelid (Lama glama).
///
/// Andean cultures have used llamas as meat and pack
/// animals since pre-Hispanic times.
class Llama {
  String name;

  /// Feeds your llama [Food].
  ///
  /// The typical llama eats one bale of hay per week.
  void feed(Food food) {
    // ...
  }

  /// Exercises your llama with an [activity] for
  /// [timeLimit] minutes.
  void exercise(Activity activity, int timeLimit) {
    // ...
  }
}
```

在生成的文档中，`[Food]` 会成为一个链接，指向 Food 类的 API 文档。

解析 Dart 代码并生成 HTML 文档，可以使用 SDK 中的[文档生成工具。](https://github.com/dart-lang/dartdoc#dartdoc)关于生成文档的实例，请参考 [Dart API documentation.](https://api.dart.dev/stable)关于文档结构的建议，请参考[Guidelines for Dart Doc Comments.](https://dart.cn/guides/language/effective-dart/documentation)