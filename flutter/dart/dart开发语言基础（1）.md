---
title: dart开发语言基础（1）
date: 2020-06-11 20:21:02
tags:
	- flutter
	- dart
categories: 
	- flutter
	- dart
---



##  一、重要概念

- 所有变量引用的都是 *对象*，每个对象都是一个 *类* 的实例。数字、函数以及 `null` 都是对象。所有的类都继承于 [Object](https://api.dart.dev/stable/dart-core/Object-class.html) 类。

- 尽管 Dart 是强类型语言，但是在声明变量时指定类型是可选的，因为 Dart 可以进行类型推断。在上述代码中，变量 `number` 的类型被推断为 `int` 类型。如果想显式地声明一个不确定的类型，可以[使用特殊类型 `dynamic`](https://dart.cn/guides/language/effective-dart/design#do-annotate-with-object-instead-of-dynamic-to-indicate-any-object-is-allowed)。

- Dart 支持泛型，比如 `List<int>`（表示一组由 int 对象组成的列表）或 `List<dynamic>`（表示一组由任何类型对象组成的列表）。
- Dart 支持顶级函数（例如 `main` 方法），同时还支持定义属于类或对象的函数（即 *静态* 和 *实例方法*）。你还可以在函数中定义函数（*嵌套* 或 *局部函数*）。
- Dart 支持顶级 *变量*，以及定义属于类或对象的变量（静态和实例变量）。实例变量有时称之为域或属性。
- Dart 没有类似于 Java 那样的 `public`、`protected` 和 `private` 成员访问限定符。如果一个标识符以下划线 (_) 开头则表示该标识符在库内是私有的。可以查阅 [库和可见性](https://dart.cn/guides/language/language-tour#libraries-and-visibility) 获取更多相关信息。
- *标识符* 可以以字母或者下划线 (_) 开头，其后可跟字符和数字的组合。
- Dart 中 *表达式* 和 *语句* 是有区别的，表达式有值而语句没有。比如[条件表达式](https://dart.cn/guides/language/language-tour#conditional-expressions) `expression condition ? expr1 : expr2` 中含有值 `expr1` 或 `expr2`。与 [if-else 分支语句](https://dart.cn/guides/language/language-tour#if-and-else)相比，`if-else` 分支语句则没有值。一个语句通常包含一个或多个表达式，但是一个表达式不能只包含一个语句。
- Dart 工具可以显示 *警告* 和 *错误* 两种类型的问题。警告表明代码可能有问题但不会阻止其运行。错误分为编译时错误和运行时错误；编译时错误代码无法运行；运行时错误会在代码运行时导致[异常](https://dart.cn/guides/language/language-tour#exceptions)。

## 二、关键字

下面的表格中列出了 Dart 语言所使用的关键字。

| [abstract](https://dart.cn/guides/language/language-tour#abstract-classes) 2 | [else](https://dart.cn/guides/language/language-tour#if-and-else) | [import](https://dart.cn/guides/language/language-tour#using-libraries) 2 | [super](https://dart.cn/guides/language/language-tour#extending-a-class) |
| ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| [as](https://dart.cn/guides/language/language-tour#type-test-operators) 2 | [enum](https://dart.cn/guides/language/language-tour#enumerated-types) | [in](https://dart.cn/guides/language/language-tour#for-loops) | [switch](https://dart.cn/guides/language/language-tour#switch-and-case) |
| [assert](https://dart.cn/guides/language/language-tour#assert) | [export](https://dart.cn/guides/libraries/create-library-packages) 2 | [interface](https://stackoverflow.com/questions/28595501/was-the-interface-keyword-removed-from-dart) 2 | [sync](https://dart.cn/guides/language/language-tour#generators) 1 |
| [async](https://dart.cn/guides/language/language-tour#asynchrony-support) 1 | [extends](https://dart.cn/guides/language/language-tour#extending-a-class) | [is](https://dart.cn/guides/language/language-tour#type-test-operators) | [this](https://dart.cn/guides/language/language-tour#constructors) |
| [await](https://dart.cn/guides/language/language-tour#asynchrony-support) 3 | [extension](https://dart.cn/guides/language/language-tour#extension-methods) 2 | [library](https://dart.cn/guides/language/language-tour#libraries-and-visibility) 2 | [throw](https://dart.cn/guides/language/language-tour#throw) |
| [break](https://dart.cn/guides/language/language-tour#break-and-continue) | [external](https://stackoverflow.com/questions/24929659/what-does-external-mean-in-dart) 2 | [mixin](https://dart.cn/guides/language/language-tour#adding-features-to-a-class-mixins) 2 | [true](https://dart.cn/guides/language/language-tour#booleans) |
| [case](https://dart.cn/guides/language/language-tour#switch-and-case) | [factory](https://dart.cn/guides/language/language-tour#factory-constructors) 2 | [new](https://dart.cn/guides/language/language-tour#using-constructors) | [try](https://dart.cn/guides/language/language-tour#catch)   |
| [catch](https://dart.cn/guides/language/language-tour#catch) | [false](https://dart.cn/guides/language/language-tour#booleans) | [null](https://dart.cn/guides/language/language-tour#default-value) | [typedef](https://dart.cn/guides/language/language-tour#typedefs) 2 |
| [class](https://dart.cn/guides/language/language-tour#instance-variables) | [final](https://dart.cn/guides/language/language-tour#final-and-const) | [on](https://dart.cn/guides/language/language-tour#catch) 1  | [var](https://dart.cn/guides/language/language-tour#variables) |
| [const](https://dart.cn/guides/language/language-tour#final-and-const) | [finally](https://dart.cn/guides/language/language-tour#finally) | [operator](https://dart.cn/guides/language/language-tour#overridable-operators) 2 | [void](https://medium.com/dartlang/dart-2-legacy-of-the-void-e7afb5f44df0) |
| [continue](https://dart.cn/guides/language/language-tour#break-and-continue) | [for](https://dart.cn/guides/language/language-tour#for-loops) | [part](https://dart.cn/guides/libraries/create-library-packages#organizing-a-library-package) 2 | [while](https://dart.cn/guides/language/language-tour#while-and-do-while) |
| [covariant](https://dart.cn/guides/language/sound-problems#the-covariant-keyword) 2 | [Function](https://dart.cn/guides/language/language-tour#functions) 2 | [rethrow](https://dart.cn/guides/language/language-tour#catch) | [with](https://dart.cn/guides/language/language-tour#adding-features-to-a-class-mixins) |
| [default](https://dart.cn/guides/language/language-tour#switch-and-case) | [get](https://dart.cn/guides/language/language-tour#getters-and-setters) 2 | [return](https://dart.cn/guides/language/language-tour#functions) | [yield](https://dart.cn/guides/language/language-tour#generators) 3 |
| [deferred](https://dart.cn/guides/language/language-tour#lazily-loading-a-library) 2 | [hide](https://dart.cn/guides/language/language-tour#importing-only-part-of-a-library) 1 | [set](https://dart.cn/guides/language/language-tour#getters-and-setters) 2 |                                                              |
| [do](https://dart.cn/guides/language/language-tour#while-and-do-while) | [if](https://dart.cn/guides/language/language-tour#if-and-else) | [show](https://dart.cn/guides/language/language-tour#importing-only-part-of-a-library) 1 |                                                              |
| [dynamic](https://dart.cn/guides/language/language-tour#important-concepts) 2 | [implements](https://dart.cn/guides/language/language-tour#implicit-interfaces) 2 | [static](https://dart.cn/guides/language/language-tour#class-variables-and-methods) 2 |                                                              |

应该避免使用这些单词作为标识符。但是，带有上标的单词可以在必要的情况下作为标识符：

- 带有上标 **1** 的关键字为 **上下文关键字**，只有在特定的场景才有意义，它们可以在任何地方作为有效的标识符。
- 带有上标 **2** 的关键字为 **内置标识符**，其作用只是在JavaScript代码转为Dart代码时更简单，这些关键字在大多数时候都可以作为有效的标识符，但是它们不能用作类名或者类型名或者作为导入前缀使用。
- 带有上标 **3** 的关键字为 Dart1.0 发布后用于[支持异步](https://dart.cn/guides/language/language-tour#asynchrony-support)相关的特性新加的。不能在由关键字 `async`、`async*` 或 `sync*` 标识的方法体中使用 `await` 或 `yield` 作为标识符。

其它没有上标的关键字为 **保留字**，均不能用作标识符。



## 三、语法

### 3.1 =>

语法 `=> *表达式*` 是 `{ return *表达式*; }` 的简写， `=>` 有时也称之为胖箭头语法。

```
bool isNoble(int atomicNumber) => _nobleGases[atomicNumber] != null;
```

### 3.2 可选参数

可选参数分为命名参数和位置参数，可在参数列表中任选其一使用，但两者不能同时出现在参数列表中。

#### 3.2.1 命名参数

```
enableFlags(bold: true, hidden: false);
```

虽然命名参数是可选参数的一种类型，但是你仍然可以使用 [@required](https://pub.flutter-io.cn/documentation/meta/latest/meta/required-constant.html) 注解来标识一个命名参数是必须的参数，此时调用者则必须为该参数提供一个值。例如：

```
const Scrollbar({Key key, @required Widget child})

```

#### 3.2.2 位置参数

```
String say(String from, String msg, [String device]) {
  var result = '$from says $msg';
  if (device != null) {
    result = '$result with a $device';
  }
  return result;
}
```

#### 3.2.3 默认参数值

可以用 `=` 为函数的命名和位置参数定义默认值，默认值必须为编译时常量，没有指定默认值的情况下默认值为 `null`

```
/// 设置 [bold] 和 [hidden] 标识……
/// Sets the [bold] and [hidden] flags ...
void enableFlags({bool bold = false, bool hidden = false}) {...}

// bold 的值将为 true；而 hidden 将为 false。
enableFlags(bold: true);
```

下一个示例将向你展示如何为位置参数设置默认值：

```
String say(String from, String msg,
    [String device = 'carrier pigeon', String mood]) {
  var result = '$from says $msg';
  if (device != null) {
    result = '$result with a $device';
  }
  if (mood != null) {
    result = '$result (in a $mood mood)';
  }
  return result;
}

assert(say('Bob', 'Howdy') ==
    'Bob says Howdy with a carrier pigeon');
```

##### 3.2.3.1默认参数是List

需要加入const

```
final List<double> data;
final List<String> legends;
const PieChart({Key key, this.data = const [4.3,5.2,1.6,3.0], this.legends=const ['第一季度','第二季度','第三季度','第四季度',]}) : super(key: key);
```

### 3.3 函数作为一级对象

可以将函数作为参数传递给另一个函数。例如：

```
void printElement(int element) {
  print(element);
}

var list = [1, 2, 3];

// 将 printElement 函数作为参数传递。
list.forEach(printElement);
```

你也可以将函数赋值给一个变量，比如：

```
var loudify = (msg) => '!!! ${msg.toUpperCase()} !!!';
assert(loudify('hello') == '!!! HELLO !!!');
```

### 3.4 匿名函数

大多数方法都是有名字的，比如 `main()` 或 `printElement()`。你可以创建一个没有名字的方法，称之为 *匿名函数*，或 *Lambda表达式* 或 *Closure闭包*。你可以将匿名方法赋值给一个变量然后使用它，比如将该变量添加到集合或从中删除。

匿名方法看起来与命名方法类似，在括号之间可以定义参数，参数之间用逗号分割。

```
后面大括号中的内容则为函数体：

([[类型] 参数[, …]]) {
  函数体;
};
```

下面代码定义了只有一个参数 `item` 且没有参数类型的匿名方法。List 中的每个元素都会调用这个函数，打印元素位置和值的字符串：

```
var list = ['apples', 'bananas', 'oranges'];
list.forEach((item) {
  print('${list.indexOf(item)}: $item');
});
```

### 3.5 词法作用域

Dart 是词法有作用域语言，变量的作用域在写代码的时候就确定了，大括号内定义的变量只能在大括号内访问，与 Java 类似。

下面是一个嵌套函数中变量在多个作用域中的示例：

```
bool topLevel = true;

void main() {
  var insideMain = true;

  void myFunction() {
    var insideFunction = true;

    void nestedFunction() {
      var insideNestedFunction = true;

      assert(topLevel);
      assert(insideMain);
      assert(insideFunction);
      assert(insideNestedFunction);
    }
  }
}
```

注意 `nestedFunction()` 函数可以访问包括顶层变量在内的所有的变量。

### 3.5 词法闭包

*闭包* 即一个函数对象，即使函数对象的调用在它原始作用域之外，依然能够访问在它词法作用域内的变量。

函数可以封闭定义到它作用域内的变量。接下来的示例中，函数 `makeAdder()` 捕获了变量 `addBy`。无论函数在什么时候返回，它都可以使用捕获的 `addBy` 变量。

```
/// 返回一个将 [addBy] 添加到该函数参数的函数。
/// Returns a function that adds [addBy] to the
/// function's argument.
Function makeAdder(int addBy) {
  return (int i) => addBy + i;
}

void main() {
  // 生成加 2 的函数。
  var add2 = makeAdder(2);

  // 生成加 4 的函数。
  var add4 = makeAdder(4);

  assert(add2(3) == 5);
  assert(add4(3) == 7);
}
```

### 3.6 测试函数是否相等

下面是顶级函数，静态方法和示例方法相等性的测试示例：

```
void foo() {} // 定义顶层函数 (A top-level function)

class A {
  static void bar() {} // 定义静态方法
  void baz() {} // 定义实例方法
}

void main() {
  var x;

  // 比较顶层函数是否相等。
  x = foo;
  assert(foo == x);

  // 比较静态方法是否相等。
  x = A.bar;
  assert(A.bar == x);

  // 比较实例方法是否相等。
  var v = A(); // A 的实例 #1
  var w = A(); // A 的实例 #2
  var y = w;
  x = w.baz;

  // 这两个闭包引用了相同的实例对象，因此它们相等。
  assert(y.baz == x);

  // 这两个闭包引用了不同的实例对象，因此它们不相等。
  assert(v.baz != w.baz);
}
```

### 3.7 返回值

所有的函数都有返回值。没有显示返回语句的函数最后一行默认为执行 `return null;`。

```
foo() {}

assert(foo() == null);
```

## 四、运算符

Dart定义了下表中显示的运算符。您可以覆盖许多这些运算符，如可覆盖运算符中所述。

| 描述           | 运算符                                                       |
| -------------- | ------------------------------------------------------------ |
| 一元后缀       | `*表达式*++` `*表达式*--` `()` `[]` `.` `?.`                 |
| 一元前缀       | `-*表达式*` `!*表达式*` `~*表达式*` `++*表达式*` `--*表达式*` |
| 乘除法         | `*` `/` `%` `~/`                                             |
| 加减法         | `+` `-`                                                      |
| 位运算         | `<<` `>>` `>>>`                                              |
| 二进制与       | `&`                                                          |
| 二进制异或     | `^`                                                          |
| 二进制或       | `|`                                                          |
| 关系和类型测试 | `>=` `>` `<=` `<` `as` `is` `is!`                            |
| 相等判断       | `==` `!=`                                                    |
| 逻辑与         | `&&`                                                         |
| 逻辑或         | `||`                                                         |
| 空判断         | `??`                                                         |
| 条件表达式     | `*表达式 1* ? *表达式 2* : *表达式 3*`                       |
| 级联           | `..`                                                         |
| 赋值           | `=` `*=` `/=` `+=` `-=` `&=` `^=` *等等……*                   |

一旦你使用了运算符，就创建了表达式。下面是一些运算符表达式的示例：

```
a++
a + b
a = b
a == b
c ? a : b
a is T
```

在[运算符表](https://dart.cn/guides/language/language-tour#operators) 中，运算符的优先级按先后排列，即第一行优先级最高，最后一行优先级最低，而同一行中，最左边的优先级最高，最右边的优先级最低。例如：`%` 运算符优先级高于 `==` ，而 `==` 高于 `&&`。根据优先级规则，那么意味着以下两行代码执行的效果相同：

```
// 括号提高了可读性。
// Parentheses improve readability.
if ((n % i == 0) && (d % i == 0)) ...

// 难以理解，但是与上面的代码效果一样。
if (n % i == 0 && d % i == 0) ...
```

### 4.1 算术运算符

Dart 支持常用的算术运算符：

| 运算符      | 描述                                       |
| ----------- | ------------------------------------------ |
| `+`         | 加                                         |
| `–`         | 减                                         |
| `-*表达式*` | 一元负, 也可以作为反转（反转表达式的符号） |
| `*`         | 乘                                         |
| `/`         | 除                                         |
| `~/`        | 除并取整                                   |
| `%`         | 取模                                       |

示例：

```
assert(2 + 3 == 5);
assert(2 - 3 == -1);
assert(2 * 3 == 6);
assert(5 / 2 == 2.5); // 结果是一个浮点数
assert(5 ~/ 2 == 2); // 结果是一个整数
assert(5 % 2 == 1); // 取余

assert('5/2 = ${5 ~/ 2} r ${5 % 2}' == '5/2 = 2 r 1');
```

Dart 还支持自增自减操作。

| Operator`++*var*` | `*var* = *var* + 1` (表达式的值为 `*var* + 1`) |
| ----------------- | ---------------------------------------------- |
| `*var*++`         | `*var* = *var* + 1` (表达式的值为 `*var*`)     |
| `--*var*`         | `*var* = *var* – 1` (表达式的值为 `*var* – 1`) |
| `*var*--`         | `*var* = *var* – 1` (表达式的值为 `*var*`)     |

示例：

```
var a, b;

a = 0;
b = ++a; // 在 b 赋值前将 a 增加 1。
assert(a == b); // 1 == 1

a = 0;
b = a++; // 在 b 赋值后将 a 增加 1。
assert(a != b); // 1 != 0

a = 0;
b = --a; // 在 b 赋值前将 a 减少 1。
assert(a == b); // -1 == -1

a = 0;
b = a--; // 在 b 赋值后将 a 减少 1。
assert(a != b); // -1 != 0
```

### 4.2 关系运算符

下表列出了关系运算符及含义：

| Operator`==` | 相等     |
| ------------ | -------- |
| `!=`         | 不等     |
| `>`          | 大于     |
| `<`          | 小于     |
| `>=`         | 大于等于 |
| `<=`         | 小于等于 |

要判断两个对象 x 和 y 是否表示相同的事物使用 `==` 即可。（在极少数情况下，可能需要使用 [identical()](https://api.dart.dev/stable/dart-core/identical.html) 函数来确定两个对象是否完全相同。）。下面是 `==` 运算符的一些规则：

1. 假设有变量 *x* 和 *y*，且 x 和 y 至少有一个为 null，则当且仅当 x 和 y 均为 null 时 x == y 才会返回 true，否则只有一个为 null 则返回 false。
2. `*x*.==(*y*)` 将会返回值，这里不管有没有 y，即 y 是可选的。也就是说 `==` 其实是 x 中的一个方法，并且可以被重写。详情请查阅[重写运算符](https://dart.cn/guides/language/language-tour#overridable-operators)。

下面的代码给出了每一种关系运算符的示例：

```
assert(2 == 2);
assert(2 != 3);
assert(3 > 2);
assert(2 < 3);
assert(3 >= 3);
assert(2 <= 3);
```

### 4.3 类型判断运算符

`as`、`is`、`is!` 运算符是在运行时判断对象类型的运算符。

| Operator`as` | 类型转换（也用作指定[类前缀](https://dart.cn/guides/language/language-tour#specifying-a-library-prefix))） |
| ------------ | ------------------------------------------------------------ |
| `is`         | 如果对象是指定类型则返回 true                                |
| `is!`        | 如果对象是指定类型则返回 false                               |

当且仅当 `obj` 实现了 `T` 的接口，`obj is T` 才是 true。例如 `obj is Object` 总为 true，因为所有类都是 Object 的子类。

仅当你确定这个对象是该类型的时候，你才可以使用 `as` 操作符可以把对象转换为特定的类型。例如：

```
(emp as Person).firstName = 'Bob';
```

如果你不确定这个对象类型是不是 `T`，请在转型前使用 `is T` 检查类型。

```
if (emp is Person) {
  // 类型检查
  emp.firstName = 'Bob';
}
```

你可以使用 `as` 运算符进行缩写：

```
(emp as Person).firstName = 'Bob';
```

> 上述两种方式是有区别的：如果 `emp` 为 null 或者不为 Person 类型，则第一种方式将会抛出异常，而第二种不会。

### 4.4 赋值运算符

可以使用 `=` 来赋值，同时也可以使用 `??=` 来为值为 null 的变量赋值。

```
// 将 value 赋值给 a (Assign value to a)
a = value;
// 当且仅当 b 为 null 时才赋值
b ??= value;
```

像 `+=` 这样的赋值运算符将算数运算符和赋值运算符组合在了一起。

| `=`  | `–=` | `/=`  | `%=`  | `>>=` | `^=` |
| ---- | ---- | ----- | ----- | ----- | ---- |
| `+=` | `*=` | `~/=` | `<<=` | `&=`  | `|=` |

下表解释了符合运算符的原理：

| 场景                      | 复合运算    | 等效表达式     |
| ------------------------- | ----------- | -------------- |
| **假设有运算符 \*op\*：** | `a *op*= b` | `a = a *op* b` |
| **示例：**                | `a += b`    | `a = a + b`    |

下面的例子展示了如何使用赋值以及复合赋值运算符：

```
var a = 2; // 使用 = 赋值 (Assign using =)
a *= 3; // 赋值并做乘法运算 Assign and multiply: a = a * 3
assert(a == 6);
```

### 4.5 逻辑运算符

使用逻辑运算符你可以反转或组合布尔表达式。

| 运算符      | 描述                                                      |
| ----------- | --------------------------------------------------------- |
| `!*表达式*` | 对表达式结果取反（即将 true 变为 false，false 变为 true） |
| `||`        | 逻辑或                                                    |
| `&&`        | 逻辑与                                                    |

下面是使用逻辑表达式的示例：

```
if (!done && (col == 0 || col == 3)) {
  // ...Do something...
}
```

### 4.6 按位和移位运算符

在 Dart 中，二进制位运算符可以操作二进制的某一位，但仅适用于整数。

| 运算符      | 描述                                        |
| ----------- | ------------------------------------------- |
| `&`         | 按位与                                      |
| `|`         | 按位或                                      |
| `^`         | 按位异或                                    |
| `~*表达式*` | 按位取反（即将 “0” 变为 “1”，“1” 变为 “0”） |
| `<<`        | 位左移                                      |
| `>>`        | 位右移                                      |

下面是使用按位和移位运算符的示例：

```
final value = 0x22;
final bitmask = 0x0f;

assert((value & bitmask) == 0x02); // 按位与 (AND)
assert((value & ~bitmask) == 0x20); // 取反后按位与 (AND NOT)
assert((value | bitmask) == 0x2f); // 按位或 (OR)
assert((value ^ bitmask) == 0x2d); // 按位异或 (XOR)
assert((value << 4) == 0x220); // 位左移 (Shift left)
assert((value >> 4) == 0x02); // 位右移 (Shift right)
```

### 4.7 条件表达式

Dart 有两个特殊的运算符可以用来替代 [if-else](https://dart.cn/guides/language/language-tour#if-和-else) 语句：

- `*condition* ? *expr1* : *expr2*`

  `*条件* ? *表达式 1* : *表达式 2*` ：如果条件为 true，执行表达式 1并返回执行结果，否则执行表达式 2 并返回执行结果。

- `*expr1* ?? *expr2*`

  `*表达式 1* ?? *表达式 2*`：如果表达式 1 为非 null 则返回其值，否则执行表达式 2 并返回其值。

**如果赋值是根据布尔表达式则考虑使用 `?:`。**

```
var visibility = isPublic ? 'public' : 'private';
```

**如果赋值是根据判定是否为 null 则考虑使用 `??`。**

```
String playerName(String name) => name ?? 'Guest';
```

上述示例还可以写成至少下面两种不同的形式，只是不够简洁：

```
// 相对使用 ?: 运算符来说稍微长了点。(Slightly longer version uses ?: operator).
String playerName(String name) => name != null ? name : 'Guest';

// 如果使用 if-else 则更长。
String playerName(String name) {
  if (name != null) {
    return name;
  } else {
    return 'Guest';
  }
}
```

### 4.8 级联运算符（..）

级联运算符（`..`）可以让你在同一个对象上连续调用多个对象的变量或方法。

比如下面的代码：

```
querySelector('#confirm') // 获取对象 (Get an object).
  ..text = 'Confirm' // 使用对象的成员 (Use its members).
  ..classes.add('important')
  ..onClick.listen((e) => window.alert('Confirmed!'));
```

第一个方法 `querySelector` 返回了一个 Selector 对象，后面的级联操作符都是调用这个 Selector 对象的成员并忽略每个操作的返回值。

上面的代码相当于：

```
var button = querySelector('#confirm');
button.text = 'Confirm';
button.classes.add('important');
button.onClick.listen((e) => window.alert('Confirmed!'));
```

级联运算符可以嵌套，例如：

```
final addressBook = (AddressBookBuilder()
      ..name = 'jenny'
      ..email = 'jenny@example.com'
      ..phone = (PhoneNumberBuilder()
            ..number = '415-555-0100'
            ..label = 'home')
          .build())
    .build();
```

在返回对象的函数中谨慎使用级联操作符。例如，下面的代码是错误的：

```
var sb = StringBuffer();
sb.write('foo')
  ..write('bar'); // 出错：void 对象中没有方法 write (Error: method 'write' isn't defined for 'void').
```

上述代码中的 `sb.write()` 方法返回的是 void，返回值为 `void` 的方法则不能使用级联运算符。

> 严格来说 `..` 级联操作并非一个运算符而是 Dart 的特殊语法。

### 4.9 其他运算符

大多数其它的运算符，已经在其它的示例中使用过：

| 运算符 | 名字         | 描述                                                         |
| ------ | ------------ | ------------------------------------------------------------ |
| `()`   | 使用方法     | 代表调用一个方法                                             |
| `[]`   | 访问 List    | 访问 List 中特定位置的元素                                   |
| `.`    | 访问成员     | 成员访问符                                                   |
| `?.`   | 条件访问成员 | 与上述成员访问符类似，但是左边的操作对象不能为 null，例如 foo?.bar，如果 foo 为 null 则返回 null ，否则返回 bar |

更多关于 `.`, `?.` 和 `..` 运算符介绍，请参考[类](https://dart.cn/guides/language/language-tour#classes).

## 五、流程控制语句

你可以使用下面的语句来控制 Dart 代码的执行流程：

- `if` 和 `else`
- `for` 循环
- `while` 和 `do`-`while` 循环
- `break` 和 `continue`
- `switch` 和 `case`
- `assert`

使用 `try-catch` 和 `throw` 也能影响控制流，详情参考[异常](https://dart.cn/guides/language/language-tour#exceptions)部分。

### 5.1 If 和 Else

Dart 支持 `if - else` 语句，其中 `else` 是可选的，比如下面的例子。你也可以参考[条件表达式](https://dart.cn/guides/language/language-tour#conditional-expressions)。

```
if (isRaining()) {
  you.bringRainCoat();
} else if (isSnowing()) {
  you.wearJacket();
} else {
  car.putTopDown();
}
```

与 JavaScript 不同的是，Dart 的 if 语句中的条件必须是一个布尔值，不能是其它类型。详情请查阅[布尔值](https://dart.cn/guides/language/language-tour#booleans)。

### 5.2 For 循环

你可以使用标准的 `for` 循环进行迭代。例如：

```
var message = StringBuffer('Dart is fun');
for (var i = 0; i < 5; i++) {
  message.write('!');
}
```

在 Dart 语言中，`for` 循环中的闭包会自动捕获循环的 **索引值** 以避免 JavaScript 中一些常见的陷阱。假设有如下代码：

```
var callbacks = [];
for (var i = 0; i < 2; i++) {
  callbacks.add(() => print(i));
}
callbacks.forEach((c) => c());
```

上述代码执行后会输出 `0` 和 `1`，但是如果在 JavaScript 中执行同样的代码则会输出两个 `2`。

如果要遍历的对象实现了 Iterable 接口，则可以使用 [forEach()](https://api.dart.dev/stable/dart-core/Iterable/forEach.html) 方法，如果不需要使用到索引，则使用 `forEach` 方法是一个非常好的选择：

```
candidates.forEach((candidate) => candidate.interview());
```

像 List 和 Set 等实现了 Iterable 接口的类还支持 `for-in` 形式的 [迭代](https://dart.cn/guides/libraries/library-tour#iteration)：

```
var collection = [0, 1, 2];
for (var x in collection) {
  print(x); // 0 1 2
}
```

### 5.3 While 和 Do-While

`while` 循环会在执行循环体前先判断条件：

```
while (!isDone()) {
  doSomething();
}
```

`do-while` 循环则会先执行一遍循环体 *再* 判断条件：

```
do {
  printLine();
} while (!atEndOfPage());
```

### 5.4 Break 和 Continue

使用 `break` 可以中断循环：

```
while (true) {
  if (shutDownRequested()) break;
  processIncomingRequests();
}
```

使用 `continue` 可以跳过本次循环直接进入下一次循环：

```
for (int i = 0; i < candidates.length; i++) {
  var candidate = candidates[i];
  if (candidate.yearsExperience < 5) {
    continue;
  }
  candidate.interview();
}
```

上述代码中的 candidates 如果像 List 或 Set 一样实现了 [Iterable](https://api.dart.dev/stable/dart-core/Iterable-class.html) 接口则可以简单地使用下述写法：

```
candidates
    .where((c) => c.yearsExperience >= 5)
    .forEach((c) => c.interview());
```

### 5.5 Switch 和 Case

Switch 语句在 Dart 中使用 `==` 来比较整数、字符串或编译时常量，比较的两个对象必须是同一个类型且不能是子类并且没有重写 `==` 操作符。 [枚举类型](https://dart.cn/guides/language/language-tour#enumerated-types)非常适合在 `Switch` 语句中使用。

 **备忘:**

> Dart 中的 Switch 语句仅适用于有限的情况，比如使用解释器和扫描器的场景。

每一个非空的 `case` 子句都必须有一个 `break` 语句，也可以通过 `continue`、`throw` 或者 `return` 来结束非空 `case` 语句。

当没有 `case` 语句匹配时，可以使用 `default` 子句来匹配这种情况：

```
var command = 'OPEN';
switch (command) {
  case 'CLOSED':
    executeClosed();
    break;
  case 'PENDING':
    executePending();
    break;
  case 'APPROVED':
    executeApproved();
    break;
  case 'DENIED':
    executeDenied();
    break;
  case 'OPEN':
    executeOpen();
    break;
  default:
    executeUnknown();
}
```

下面的例子忽略了 `case` 子句的 `break` 语句，因此会产生错误：

```
var command = 'OPEN';
switch (command) {
  case 'OPEN':
    executeOpen();
    // 错误: 没有 break

  case 'CLOSED':
    executeClosed();
    break;
}
```

但是，Dart 支持空的 `case` 语句，允许其以 fall-through 的形式执行。

```
var command = 'CLOSED';
switch (command) {
  case 'CLOSED': // case 语句为空时的 fall-through 形式。
  case 'NOW_CLOSED':
    // case 条件值为 CLOSED 和 NOW_CLOSED 时均会执行该语句。
    executeNowClosed();
    break;
}
```

在非空 `case` 语句中想要实现 fall-through 的形式，可以使用 `continue` 语句配合 lable 的方式实现:

```
var command = 'CLOSED';
switch (command) {
  case 'CLOSED':
    executeClosed();
    continue nowClosed;
  // 继续执行标签为 nowClosed 的 case 子句。

  nowClosed:
  case 'NOW_CLOSED':
    // case 条件值为 CLOSED 和 NOW_CLOSED 时均会执行该语句。
    executeNowClosed();
    break;
}
```

每个 `case` 子句都可以有局部变量且仅在该 case 语句内可见。

### 5.6 断言

在开发过程中，可以在条件表达式为 false 时使用 - `assert(*条件*, *可选信息*)`; - 语句来打断代码的执行。你可以在本文中找到大量使用 assert 的例子。下面是相关示例：

```
// 确保变量值不为 null (Make sure the variable has a non-null value)
assert(text != null);

// 确保变量值小于 100。
assert(number < 100);

// 确保这是一个 https 地址。
assert(urlString.startsWith('https'));
```

`assert` 的第二个参数可以为其添加一个字符串消息。

```
assert(urlString.startsWith('https'),
    'URL ($urlString) should start with "https".');
```

`assert` 的第一个参数可以是值为布尔值的任何表达式。如果表达式的值为 true，则断言成功，继续执行。如果表达式的值为 false，则断言失败，抛出一个 [AssertionError](https://api.dart.dev/stable/dart-core/AssertionError-class.html) 异常。

如何判断 assert 是否生效？assert 是否生效依赖开发工具和使用的框架：

- Flutter 在[调试模式](https://flutter.cn/docs/testing/debugging#debug-mode-assertions)时生效。
- 一些开发工具比如 [dartdevc](https://dart.cn/tools/dartdevc) 通常情况下是默认生效的。
- 其他一些工具，比如 [dart](https://dart.cn/server/tools/dart-vm) 以及 [dart2js](https://dart.cn/tools/dart2js) 通过在运行 Dart 程序时添加命令行参数 `--enable-asserts` 使 assert 生效。

在生产环境代码中，断言会被忽略，与此同时传入 `assert` 的参数不被判断。

## 六、异常

Dart 代码可以抛出和捕获异常。异常表示一些未知的错误情况，如果异常没有捕获则会被抛出从而导致抛出异常的代码终止执行。

与 Java 不同的是，Dart 的所有异常都是非必检异常，方法不一定会声明其所抛出的异常并且你也不会被要求捕获任何异常。

Dart 提供了 [Exception](https://api.dart.dev/stable/dart-core/Exception-class.html) 和 [Error](https://api.dart.dev/stable/dart-core/Error-class.html) 两种类型的异常以及它们一系列的子类，你也可以定义自己的异常类型。但是在 Dart 中可以将任何非 null 对象作为异常抛出而不局限于 Exception 或 Error 类型。

### 6.1 抛出异常

下面是关于抛出或者 *引发* 异常的示例：

```
throw FormatException('Expected at least 1 section');
```

你也可以抛出任意的对象：

```
throw 'Out of llamas!';
```

 **备忘:**

优秀的代码通常会抛出 [Error](https://api.dart.dev/stable/dart-core/Error-class.html) 或 [Exception](https://api.dart.dev/stable/dart-core/Exception-class.html) 类型的异常。

因为抛出异常是一个表达式，所以可以在 => 语句中使用，也可以在其他使用表达式的地方抛出异常：

```
void distanceTo(Point other) => throw UnimplementedError();
```

### 6.2 捕获异常

捕获异常可以避免异常继续传递（重新抛出异常除外）。捕获一个异常可以给你处理它的机会：

```
try {
  breedMoreLlamas();
} on OutOfLlamasException {
  buyMoreLlamas();
}
```

对于可以抛出多种异常类型的代码，也可以指定多个 catch 语句，每个语句分别对应一个异常类型，如果 catch 语句没有指定异常类型则表示可以捕获任意异常类型：

```
try {
  breedMoreLlamas();
} on OutOfLlamasException {
  // 指定异常
  buyMoreLlamas();
} on Exception catch (e) {
  // 其它类型的异常
  print('Unknown exception: $e');
} catch (e) {
  // // 不指定类型，处理其它全部
  print('Something really unknown: $e');
}
```

如上述代码所示可以使用 `on` 或 `catch` 来捕获异常，使用 `on` 来指定异常类型，使用 `catch` 来捕获异常对象，两者可同时使用。

你可以为 `catch` 方法指定两个参数，第一个参数为抛出的异常对象，第二个参数为栈信息 [StackTrace](https://api.dart.dev/stable/dart-core/StackTrace-class.html) 对象：

```dart
try {
  // ···
} on Exception catch (e) {
  print('Exception details:\n $e');
} catch (e, s) {
  print('Exception details:\n $e');
  print('Stack trace:\n $s');
}
```

关键字 `rethrow` 可以将捕获的异常再次抛出：

```dart
void misbehave() {
  try {
    dynamic foo = true;
    print(foo++); // 运行时错误
  } catch (e) {
    print('misbehave() partially handled ${e.runtimeType}.');
    rethrow; // 允许调用者查看异常。
  }
}

void main() {
  try {
    misbehave();
  } catch (e) {
    print('main() finished handling ${e.runtimeType}.');
  }
}
```

### 6.3 Finally

可以使用 `finally` 语句来包裹确保不管有没有异常都执行代码，如果没有指定 `catch` 语句来捕获异常，则在执行完 `finally` 语句后再抛出异常：

```
try {
  breedMoreLlamas();
} finally {
  // 总是清理，即便抛出了异常。
  cleanLlamaStalls();
}
```

`finally` 语句会在任何匹配的 `catch` 语句后执行：

```
try {
  breedMoreLlamas();
} catch (e) {
  print('Error: $e'); // 先处理异常。
} finally {
  cleanLlamaStalls(); // 然后清理。
}
```

更多详情，请参考你可以阅读 Dart 核心库概览的 [异常](https://dart.cn/guides/libraries/library-tour#exceptions) 章节获取更多相关信息。