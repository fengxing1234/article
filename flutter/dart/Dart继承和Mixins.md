---
title: Dart继承和Mixins
date: 2020-07-31 23:18:08
tags:
---

### 一、类的单继承

#### 1、基本介绍

Dart中的单继承和其他语言中类似，都是通过使用 **`extends`** 关键字来声明。例如

```
class Student extends Person {//Student类称为子类或派生类，Person类称为父类或基类或超类。这一点和Java中是一致的。
    ...
}
复制代码
```

#### 2、继承中的构造函数

- **子类中构造函数会默认调用父类中无参构造函数(一般为主构造函数)**。

```
class Person {
  String name;
  String age;

  Person() {
    print('person');
  }
}

class Student extends Person {
  String classRoom;
  Student() {
    print('Student');
  }
}

main(){
  var student = Student();//构造Student()时会先调用父类中无参构造函数，再调用子类中无参构造函数
}
复制代码
```

输出结果:

```
person
Student

Process finished with exit code 0
复制代码
```

- **若父类中没有默认无参的构造函数，则需要显式调用父类的构造函数(可以是命名构造函数也可以主构造函数或其他), 并且在初始化列表的尾部显式调用父类中构造函数, 也即是类构造函数 `:`后面列表的尾部**。

```
class Person {
  String name;
  int age;

  Person(this.name, this.age); //指定了带参数的构造函数为主构造函数，那么父类中就没有默认无参构造函数
  //再声明两个命名构造函数
  Person.withName(this.name);

  Person.withAge(this.age);
}

class Student extends Person {
  String classRoom;

  Student(String name, int age) : super(name, age) { //显式调用父类主构造函数
    print('Student');
  }

  Student.withName(String name) : super.withName(name) {} //显式调用父类命名构造函数withName

  Student.withAge(int age) : super.withAge(age) {} //显式调用父类命名构造函数withAge
}

main() {
  var student1 = Student('mikyou', 18);
  var student2 = Student.withName('mikyou');
  var student3 = Student.withAge(18);
}
复制代码
```

- **父类的构造函数在子类构造函数体开始执行的位置调用，如果有初始化列表，初始化列表会在父类构造函数执行之前执行**。

```
class Person {
  String name;
  int age;

  Person(this.name, this.age); //指定了带参数的构造函数为主构造函数，那么父类中就没有默认无参构造函数
}

class Student extends Person {
  final String classRoom;

  Student(String name, int age, String room) : classRoom = room, super(name, age) {//注意super(name, age)必须位于初始化列表尾部
    print('Student');
  }
}

main() {
  var student = Student('mikyou', 18, '三年级八班');
}
复制代码
```

### 二、基于Mixins的多继承

除了上面和其他语言类似的单继承外，在Dart中还提供了另一继承的机制就是基于**Mixins的多继承**，但是它**不是真正意义上类的多继承，它始终还是只能有一个超类(基类)**。

#### 1、为什么需要Mixins?

为什么需要Mixins多继承？它实际上为了解决单继承所带来的问题，我们很多语言中都是采用**了单继承+接口多实现**的方式。但是这种方式并不能很好适用于所有场景。

假设一下下面场景，我们把车进行分类，然后下面的颜色条表示各种车辆具有的能力。



![img](https://user-gold-cdn.xitu.io/2019/11/12/16e5ed87e01b81d4?imageslim)

我们通过上图就可以看到，这些车辆都有一个共同的父类 **`Vehicle`**,然后它又由两个抽象的子类: **`MotorVehicle`** 和 **`NonMotorVehicle`** 。有些类是具有相同的行为和能力，但是有的类又有自己独有的行为和能力。比如公交车 **`Bus`** 和摩托车 **`Motor`** 都能使用汽油驱动，但是摩托车 **`Motor`** 还能载货公交车 **`Bus`** 却不可以。

如果**仅仅是单继承模型下**，**无法把部分子类具有相同行为和能力抽象放到基类，因为对于不具有该行为和能力的子类来说是不妥的，所以只能在各自子类另外实现**。那么就问题来了，部分具有相同能力和行为的子类中都要保留一份相同的代码实现。这就是产生冗余，突然觉得单继承模型有点鸡肋，食之无味弃之可惜。

```
//单继承模型的普通实现
abstract class Vehicle {}

abstract class MotorVehicle extends Vehicle {}

abstract class NonMotorVehicle extends Vehicle {}

class Motor extends MotorVehicle {
  void petrolDriven() => print("汽油驱动");

  void passengerService() => print('载人');

  void carryCargo() => print('载货');
}

class Bus extends MotorVehicle {
  void petrolDriven() => print("汽油驱动");

  void electricalDriven() => print("电能驱动");

  void passengerService() => print('载人');
}

class Truck extends MotorVehicle {
  void petrolDriven() => print("汽油驱动");

  void carryCargo() => print('载货');
}

class Bicycle extends NonMotorVehicle {
  void electricalDriven() => print("电能驱动");

  void passengerService() => print('载人');
}

class Bike extends NonMotorVehicle {
  void passengerService() => print('载人');
}
复制代码
```

可以从上述实现代码来看发现，有很多相同冗余代码实现，请注意这里所说的相同代码是连具体实现都相同的。很多人估计想到一个办法那就是将各个能力提升成接口，然后各自的选择去实现相应能力。但是我们知道即使抽成了接口，各个实现类中还是需要写对应的实现代码，冗余还是无法摆脱。不妨我们来试试用接口:

```
//单继承+接口多实现
abstract class Vehicle {}

abstract class MotorVehicle extends Vehicle {}

abstract class NonMotorVehicle extends Vehicle {}

//将各自的能力抽成独立的接口，这样的好处就是可以从抽象角度对不同的实现类赋予不同接口能力，
// 职责更加清晰,但是这个只是一方面的问题，它还是无法解决相同能力实现代码冗余的问题
abstract class PetrolDriven {
  void petrolDriven();
}

abstract class PassengerService {
  void passengerService();
}

abstract class CargoService {
  void carryCargo();
}

abstract class ElectricalDriven {
  void electricalDriven();
}

//对于Motor赋予了PetrolDriven、PassengerService、CargoService能力
class Motor extends MotorVehicle implements PetrolDriven, PassengerService, CargoService {
  @override
  void carryCargo() => print('载货');//仍然需要重写carryCargo

  @override
  void passengerService() => print('载人');//仍然需要重写passengerService

  @override
  void petrolDriven() => print("汽油驱动");//仍然需要重写petrolDriven
}

//对于Bus赋予了PetrolDriven、ElectricalDriven、PassengerService能力
class Bus extends MotorVehicle implements PetrolDriven, ElectricalDriven, PassengerService {
  @override
  void electricalDriven() => print("电能驱动");//仍然需要重写electricalDriven

  @override
  void passengerService() => print('载人');//仍然需要重写passengerService

  @override
  void petrolDriven() => print("汽油驱动");//仍然需要重写petrolDriven
}

//对于Truck赋予了PetrolDriven、CargoService能力
class Truck extends MotorVehicle implements PetrolDriven, CargoService {
  @override
  void carryCargo() => print('载货');//仍然需要重写carryCargo

  @override
  void petrolDriven() => print("汽油驱动");//仍然需要重写petrolDriven
}

//对于Bicycle赋予了ElectricalDriven、PassengerService能力
class Bicycle extends NonMotorVehicle implements ElectricalDriven, PassengerService {
  @override
  void electricalDriven() => print("电能驱动");//仍然需要重写electricalDriven

  @override
  void passengerService() => print('载人');//仍然需要重写passengerService
}

//对于Bike赋予了PassengerService能力
class Bike extends NonMotorVehicle implements PassengerService {
  @override
  void passengerService() => print('载人');//仍然需要重写passengerService
}

复制代码
```

针对相同实现代码冗余的问题，使用Mixins就能很好的解决。**它能复用类中某个行为的具体实现**，**而不是像接口仅仅从抽象角度规定了实现类具有哪些能力，至于具体实现接口方法都必须重写，也就意味着即使是相同的实现还得重新写一遍**。一起看下Mixins改写后代码:

```
//mixins多继承模型实现
abstract class Vehicle {}

abstract class MotorVehicle extends Vehicle {}

abstract class NonMotorVehicle extends Vehicle {}

//将各自的能力抽成独立的Mixin类
mixin PetrolDriven {//使用mixin关键字代替class声明一个Mixin类
  void petrolDriven() => print("汽油驱动");
}

mixin PassengerService {//使用mixin关键字代替class声明一个Mixin类
  void passengerService() => print('载人');
}

mixin CargoService {//使用mixin关键字代替class声明一个Mixin类
  void carryCargo() => print('载货');
}

mixin ElectricalDriven {//使用mixin关键字代替class声明一个Mixin类
  void electricalDriven() => print("电能驱动");
}

class Motor extends MotorVehicle with PetrolDriven, PassengerService, CargoService {}//利用with关键字使用mixin类

class Bus extends MotorVehicle with PetrolDriven, ElectricalDriven, PassengerService {}//利用with关键字使用mixin类

class Truck extends MotorVehicle with PetrolDriven, CargoService {}//利用with关键字使用mixin类

class Bicycle extends NonMotorVehicle with ElectricalDriven, PassengerService {}//利用with关键字使用mixin类

class Bike extends NonMotorVehicle with PassengerService {}//利用with关键字使用mixin类
复制代码
```

可以对比发现Mixins类能真正地解决相同代码冗余的问题，并能实现很好的复用；所以使用Mixins多继承模型可以很好地解决单继承模型所带来冗余问题。

#### 2、Mixins是什么?

用dart官网一句话来概括: **Mixins是一种可以在多个类层次结构中复用类代码的方式**。

- 基本语法

**方式一:** Mixins类使用关键字 **`mixin`** 声明定义

```
mixin PetrolDriven {//使用mixin关键字代替class声明一个Mixin类
  void petrolDriven() => print("汽油驱动");
}

class Motor extends MotorVehicle with PetrolDriven {//使用with关键字来使用mixin类
    ...
}

class Petrol extends PetrolDriven{//编译异常，注意:mixin类不能被继承
    ...
}

main() {
    var petrolDriven = PetrolDriven()//编译异常，注意:mixin类不能实例化
}
复制代码
```

**方式二:** Dart中的普通类当作Mixins类使用

```
class PetrolDriven {
    factory PetrolDriven._() => null;//主要是禁止PetrolDriven被继承以及实例化
    void petrolDriven() => print("汽油驱动");
}

class Motor extends MotorVehicle with PetrolDriven {//普通类也可以作为Mixins类使用
    ...
}
复制代码
```

#### 3、使用Mixins多继承的场景

那么问题来了，什么时候去使用Mixins呢？

**当想要在不同的类层次结构中多个类之间共享相同的行为时或者无法合适抽象出部分子类共同的行为到基类中时**. 比如说上述例子中在 **`MotorVehicle`** (机动车)和 **`Non-MotorVehicle`** (非机动车)两个不同类层次结构中，其中 **`Bus`** (公交车)和 **`Bicycle`** (电动自行车)都有相同行为 **`ElectricalDriven`** 和 **`PassengerService`.**  但是很明显你无法把这个两个共同的行为抽象到基类 **`Vehicle`** 中，因为这样的话 **`Bike`** (自行车)继承 **`Vehicle`** 会自动带有一个 **`ElectricalDriven`** 行为就比较诡异。所以这种场景下**mixins就是一个不错的选择**，**可以跨类层次之间复用相同行为的实现**。

#### 4、Mixins的线性化分析

在说Mixins线性化分析之前，一起先来看个例子

```
class A {
   void printMsg() => print('A'); 
}
mixin B {
    void printMsg() => print('B'); 
}
mixin C {
    void printMsg() => print('C'); 
}

class BC extends A with B, C {}
class CB extends A with C, B {}

main() {
    var bc = BC();
    bc.printMsg();
    
    var cb = CB();
    cb.printMsg();
}
复制代码
```

不妨考虑下上述例子中应该输出啥呢?

输出结果:

```
C
B

Process finished with exit code 0
复制代码
```

为什么会是这样的结果？实际上可以通过线性分析得到输出结果。理解Mixin线性化分析有一点很重要就是: **在Dart中Mixins多继承并不是真正意义上的多继承，实际上还是单继承；而每次Mixin都是会创建一个新的中间类。并且这个中间类总是在基类的上层**。

关于上述结论可能有点难以理解，下面通过一张mixins继承结构图就能清晰明白了:



![img](https://user-gold-cdn.xitu.io/2019/11/14/16e6a66c88da4808?imageslim)

通过上图，我们可以很清楚发现**Mixins并不是经典意义上获得多重继承的方法。 Mixins是一种抽象和复用一系列操作和状态的方式，而是生成多个中间的mixin类**(比如生成ABC类，AB类，ACB类，AC类)。它类似于从扩展类获得的复用，但**由于它是线性的，因此与单继承兼容**。

上述mixins代码在语义理解上可以转化成下面形式:

```
//A with B 会产生AB类混合体，B类中printMsg方法会覆盖A类中的printMsg方法,那么AB中间类保留是B类中的printMsg方法
class AB = A with B;
//AB with C 会产生ABC类混合体,C类中printMsg方法会覆盖AB混合类中的printMsg方法，那么ABC中间类保留C类中printMsg方法
class ABC = AB with C;
//最终BC类相当于继承的是ABC类混合体，最后调用的方法是ABC中间类保留C类中printMsg方法，最后输出C
class BC extends ABC {}

//A with C 会产生AC类混合体，C类中printMsg方法会覆盖A类中的printMsg方法,那么AC中间类保留是C类中的printMsg方法
class AC = A with C;
//AC with B 会产生ACB类混合体,B类中printMsg方法会覆盖AC混合类中的printMsg方法，那么ACB中间类保留B类中printMsg方法
class ACB = AC with B;
//最终CB类相当于继承的是ACB类混合体，最后调用的方法是ACB中间类保留B类中printMsg方法，最后输出B
class CB extends ACB {}

复制代码
```

#### 5、Mixins中的类型

有了上面的探索，不妨再来考虑一个问题，mixin的实例对象是什么类型呢? 我们都知道它肯定是它基类的子类型。 一起来看个例子：

```
class A {
   void printMsg() => print('A'); 
}
mixin B {
    void printMsg() => print('B'); 
}
mixin C {
    void printMsg() => print('C'); 
}

class BC extends A with B, C {}
class CB extends A with C, B {}

main() {
    var bc = BC();
    print(bc is A);
    print(bc is B);
    print(bc is C);
    
    var cb = CB();
    print(cb is A);
    print(cb is B);
    print(cb is C);
}
复制代码
```

输出结果:

```
true
true
true
true
true
true

Process finished with exit code 0
复制代码
```

可以看到输出结果全都是`true`, 这是为什么呢？

其实通过上面那张图就能找到答案，我们知道mixin with后就会生成新的中间类，比如中间类`AB、ABC`. 同时也会生成 一个新的接口`AB、ABC`(因为所有Dart类都定义了对应的接口, Dart类是可以直接当做接口实现的)。新的中间类`AB`继承基类`A`以及`A`类和`B`类混合的成员(方法和属性)副本，但它也实现了`A`接口和`B`接口。那么就很容易理解，`ABC`混合类最后会实现了`A`、`B`、`C`三个接口，最后`BC`类继承`ABC`类实际上也相当于间接实现了`A`、`B`、`C`三个接口，所以`BC`类肯定是`A`、`B`、`C`的子类型，故所有输出都是`true`。

其实一般情况mixin中间类和接口都是不能直接引用的，比如这种情况:

```
class BC extends A with B, C {}//这种情况我们无法直接引用中间AB混合类和接口、ABC混合类和接口
复制代码
```

但是如果这么写，就能直接引用中间混合类和接口了:

```
class A {
  void printMsg() => print('A');
}
mixin B {
  void printMsg() => print('B');
}
mixin C {
  void printMsg() => print('C');
}

class AB = A with B;

class ABC = AB with C;

class D extends AB {}

class E implements AB {
  @override
  void printMsg() {
    // TODO: implement printMsg
  }
}

class F extends ABC {}

class G implements ABC {
  @override
  void printMsg() {
    // TODO: implement printMsg
  }
}

main() {
  var ab = AB();
  print(ab is A);
  print(ab is B);

  var e = E();
  print(e is A);
  print(e is B);
  print(e is AB);

  var abc = ABC();
  print(abc is A);
  print(abc is B);
  print(abc is C);

  var f = F();
  print(f is A);
  print(f is B);
  print(f is C);
  print(f is AB);
  print(f is ABC);
}
复制代码
```

输出结果:

```
true
true
true
true
true
true
true
true
true
true
true
true
true

Process finished with exit code 0
复制代码
```

### 参考资料

- [Dart: What are mixins?](https://medium.com/flutter-community/dart-what-are-mixins-3a72344011f3)


作者：极客熊猫
链接：https://juejin.im/post/6844903997438951431
来源：掘金
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。