---
title: drat新颖语法糖
date: 2020-06-12 13:53:54
tags:
	- flutter
	- dart
categories: 
	- flutter
	- dart
---

### 避空运算符

Dart 提供了一系列方便的运算符用于处理可能会为空值的变量。其中一个是 `??=` 赋值运算符，仅当该变量为空值时才为其赋值：

```
int a; // The initial value of any object is null.
a ??= 3;
print(a); // <-- Prints 3.

a ??= 5;
print(a); // <-- Still prints 3.
```

另外一个避空运算符是 `??`，如果该运算符左边的表达式返回的是空值，则会计算并返回右边的表达式。

```
print(1 ?? 3); // <-- Prints 1.
print(null ?? 12); // <-- Prints 12.
```

### 条件属性访问

要保护可能会为空的属性的正常访问，请在点（`.`）之前加一个问号（`?`）。

```
myObject?.someProperty
```

上述代码等效于以下内容：

```dart
(myObject != null) ? myObject.someProperty : null
```

你可以在一个表达式中连续使用多个 `?.`：

```dart
myObject?.someProperty?.someMethod()
```

如果 `myObject` 或 `myObject.someProperty` 为空，则前面的代码返回 null（并不再调用 `someMethod`）。

### 集合字面量

Dart 内置了对 list、map 以及 set 的支持。你可以通过字面量直接创建它们：

```
final aListOfStrings = ['one', 'two', 'three'];
final aSetOfStrings = {'one', 'two', 'three'};
final aMapOfStringsToInts = {
  'one': 1,
  'two': 2,
  'three': 3,
};
```

Dart 的类型推断可以自动帮你分配这些变量的类型。在这个例子中，推断类型是 `List<String>`、`Set<String>`和 `Map<String, int>`。

你也可以手动指定类型：

```dart
final aListOfInts = <int>[];
final aSetOfInts = <int>{};
final aMapOfIntToDouble = <int, double>{};
```

在使用子类型的内容初始化列表，但仍希望列表为 `List <BaseType>` 时，指定其类型很方便：

```dart
final aListOfBaseType = <BaseType>[SubType(), SubType()];
```

### 箭头语法

你也许已经在 Dart 代码中见到过 `=>` 符号。这种箭头语法是一种定义函数的方法，该函数将在其右侧执行表达式并返回其值。

例如，考虑调用这个 `List` 类中的 `any` 方法：

```dart
bool hasEmpty = aListOfStrings.any((s) {
  return s.isEmpty;
});
```

这里是一个更简单的代码实现：

```dart
bool hasEmpty = aListOfStrings.any((s) => s.isEmpty);
```

### 级连

要对同一对象执行一系列操作，请使用级联（`..`）。我们都看到过这样的表达式：

```dart
myObject.someMethod()
```

它在 `myObject` 上调用 `someMethod` 方法，而表达式的结果是 `someMethod` 的返回值。

下面是一个使用级连语法的相同表达式：

```dart
myObject..someMethod()
```

虽然它仍然在 `myObject` 上调用了 `someMethod`，但表达式的结果却**不是**该方法返回值，而是是 `myObject` 对象的引用！使用级联，你可以将需要单独操作的语句链接在一起。例如，请考虑以下代码：

```dart
var button = querySelector('#confirm');
button.text = 'Confirm';
button.classes.add('important');
button.onClick.listen((e) => window.alert('Confirmed!'));
```

使用级连能够让代码变得更加简洁，而且你也不再需要 `button` 变量了。

```dart
querySelector('#confirm')
..text = 'Confirm'
..classes.add('important')
..onClick.listen((e) => window.alert('Confirmed!'));
```

### Getters and setters

任何需要对属性进行更多控制而不是允许简单字段访问的时候，你都可以自定义 getter 和 setter。

例如，你可以用来确保属性值合法：

```dart
class MyClass {
  int _aProperty = 0;

  int get aProperty => _aProperty;

  set aProperty(int value) {
    if (value >= 0) {
      _aProperty = value;
    }
  }
}
```

你还可以使用 getter 来定义计算属性：

```dart
class MyClass {
  List<int> _values = [];

  void addValue(int value) {
    _values.add(value);
  }

  // A computed property.
  int get count {
    return _values.length;
  }
}
```

### 可选位置参数

Dart 有两种传参方法：位置参数和命名参数。位置参数你可能会比较熟悉：

```dart
int sumUp(int a, int b, int c) {
  return a + b + c;
}

int total = sumUp(1, 2, 3);
```

在 Dart 中，你可以通过将这些参数包裹在大括号中使其变成可选位置参数：

```dart
int sumUpToFive(int a, [int b, int c, int d, int e]) {
  int sum = a;
  if (b != null) sum += b;
  if (c != null) sum += c;
  if (d != null) sum += d;
  if (e != null) sum += e;
  return sum;
}

int total = sumUptoFive(1, 2);
int otherTotal = sumUpToFive(1, 2, 3, 4, 5);
```

可选位置参数永远放在方法参数列表的最后。除非你给它们提供一个默认值，否则默认为 null。

```dart
int sumUpToFive(int a, [int b = 2, int c = 3, int d = 4, int e = 5]) {
  ...
}

int newTotal = sumUpToFive(1);
print(newTotal); // <-- p
```

### 异常

Dart 代码可以抛出和捕获异常。与 Java 相比，Dart 的所有异常都是 unchecked exception。方法不会声明它们可能抛出的异常，你也不需要捕获任何异常。

虽然 Dart 提供了 Exception 和 Error 类型，但是你可以抛出任何非空对象：

```dart
throw Exception('Something bad happened.');
throw 'Waaaaaaah!';
```

使用 `try`、`on` 以及 `catch` 关键字来处理异常：

```dart
try {
  breedMoreLlamas();
} on OutOfLlamasException {
  // A specific exception
  buyMoreLlamas();
} on Exception catch (e) {
  // Anything else that is an exception
  print('Unknown exception: $e');
} catch (e) {
  // No specified type, handles all
  print('Something really unknown: $e');
}
```

`try` 关键字作用与其他大多数语言一样。使用 `on` 关键字按类型过滤特定异常，而 `catch` 关键字则能够获取捕捉到的异常对象的引用。

如果你无法完全处理该异常，请使用 `rethrow` 关键字再次抛出异常：

```dart
try {
  breedMoreLlamas();
} catch (e) {
  print('I was just trying to breed llamas!.');
  rethrow;
}
```

要执行一段无论是否抛出异常都会执行的代码，请使用 `finally`：

```dart
try {
  breedMoreLlamas();
} catch (e) {
  … handle exception ...
} finally {
  // Always clean up, even if an exception is thrown.
  cleanLlamaStalls();
}
```

### 在构造方法中使用 `this`

Dart 提供了一个方便的快捷方式，用于为构造方法中的属性赋值：在声明构造方法时使用 `this.propertyName`。

```dart
class MyColor {
  int red;
  int green;
  int blue;

  MyColor(this.red, this.green, this.blue);
}

final color = MyColor(80, 80, 128);
```

此技巧同样也适用于命名参数。属性名为参数的名称：

```dart
class MyColor {
  ...

  MyColor({this.red, this.green, this.blue});
}

final color = MyColor(red: 80, green: 80, blue: 80);
```

对于可选参数，默认值为期望值：

```dart
MyColor([this.red = 0, this.green = 0, this.blue = 0]);
// or
MyColor({this.red = 0, this.green = 0, this.blue = 0});
```

### Initializer lists

有时，当你在实现构造函数时，您需要在构造函数体执行之前进行一些初始化。例如，final 修饰的字段必须在构造函数体执行之前赋值。在初始化列表中执行此操作，该列表位于构造函数的签名与其函数体之间：

```dart
Point.fromJson(Map<String, num> json)
    : x = json['x'],
      y = json['y'] {
  print('In Point.fromJson(): ($x, $y)');
}
```

初始化列表也是放置断言的便利位置，它仅会在开发期间运行：

```dart
NonNegativePoint(this.x, this.y)
    : assert(x >= 0),
      assert(y >= 0) {
  print('I just made a NonNegativePoint: ($x, $y)');
}
```

### 命名构造方法

为了允许一个类具有多个构造方法，Dart 支持命名构造方法：

```dart
class Point {
  num x, y;

  Point(this.x, this.y);

  Point.origin() {
    x = 0;
    y = 0;
  }
}
```

为了使用命名构造方法，请使用全名调用它：

```dart
final myPoint = Point.origin();
```

### 工厂构造方法

Dart 支持工厂构造方法。它能够返回其子类甚至 null 对象。要创建一个工厂构造方法，请使用 `factory` 关键字。

```dart
class Square extends Shape {}

class Circle extends Shape {}

class Shape {
  Shape();

  factory Shape.fromTypeName(String typeName) {
    if (typeName == 'square') return Square();
    if (typeName == 'circle') return Circle();

    print('I don\'t recognize $typeName');
    return null;
  }
}
```

### 重定向构造方法

有时一个构造方法仅仅用来重定向到该类的另一个构造方法。重定向方法没有主体，它在冒号（`:`）之后调用另一个构造方法。

```dart
class Automobile {
  String make;
  String model;
  int mpg;

  // The main constructor for this class.
  Automobile(this.make, this.model, this.mpg);

  // Delegates to the main constructor.
  Automobile.hybrid(String make, String model) : this(make, model, 60);

  // Delegates to a named constructor
  Automobile.fancyHybrid() : this.hybrid('Futurecar', 'Mark 2');
}
```

### Const 构造方法

如果你的类生成的对象永远都不会更改，则可以让这些对象成为编译时常量。为此，请定义 `const` 构造方法并确保所有实例变量都是 final 的。

```dart
class ImmutablePoint {
  const ImmutablePoint(this.x, this.y);

  final int x;
  final int y;

  static const ImmutablePoint origin =
      ImmutablePoint(0, 0);
}
```

