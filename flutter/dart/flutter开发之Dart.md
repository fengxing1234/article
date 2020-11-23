---
title: flutter开发之Dart
date: 2020-07-02 21:53:03
tags:
---

#  flutter开发之Dart[必读篇]💯

> 🥇 
>
> 🚩Flutter应用程序使用Dart语言开发，Dart是面向对象的、类定义的、单继承的语言。它的语法类似C语言，可以转译为JavaScript，支持接口(interfaces)、混入(mixins)、抽象类(abstract classes)、具体化泛型、可选类型 。
>
> ”

我相信本篇文章肯定会给你带来一定的收获👉

# 前言

```
下面这一段是笔者从Dart官网摘抄过来的，笔者认为在学习Dart之前，先要了解以下Dart相关概念：
```

- 任何保存在变量中的都是一个 对象 ， 并且所有的对象都是对应一个 类 的实例。 无论是数字，函数和 null 都是对象。所有对象继承自 Object 类。
- 尽管 Dart 是强类型的，但是 Dart 可以推断类型，所以类型注释是可选的。 如果要明确说明不需要任何类型， 需要使用特殊类型 dynamic 。
- Dart 支持泛型，如 List （整数列表）或 List （任何类型的对象列表）。
- Dart 支持顶级函数（例如 `main()` ）， 同样函数绑定在类或对象上（分别是 静态函数 和 实例函数 ）。 以及支持函数内创建函数 （ 嵌套 或 局部函数 ） 。
- Dart 支持顶级 变量 ， 同样变量绑定在类或对象上（静态变量和实例变量）。 实例变量有时称为字段或属性。
- 与 Java 不同，Dart 没有关键字 “public” ， “protected” 和 “private” 。 如果标识符以下划线（_）开头，则它相对于库是私有的。
- 标识符 以字母或下划线（_）开头，后跟任意字母和数字组合。
- Dart 语法中包含 表达式（ expressions ）（有运行时值）和 语句（ statements ）（没有运行时值）。 例如，条件表达式 condition ? expr1 : expr2 的值可能是 expr1 或 expr2 。 将其与 if-else 语句 相比较，if-else 语句没有值。 一条语句通常包含一个或多个表达式，相反表达式不能直接包含语句。
- Dart 工具提示两种类型问题：警告_和_错误。 警告只是表明代码可能无法正常工作，但不会阻止程序的执行。 错误可能是编译时错误或者运行时错误。 编译时错误会阻止代码的执行; 运行时错误会导致代码在执行过程中引发 [异常]（#exception）。

# Dart 命名规则

main() 程序开始执行函数，该函数是特定的、必须的、顶级函数。

## **Dart 变量**

```
Dart本身是一个强类型语言，任何变量都是有确定类型的。
```

**1. 使用var创建一个变量并初始化**

```
var name = 'lisa'; //name 变量的类型被推断为 String
复制代码
```

在Dart中，当用var声明一个变量后，`Dart在编译时会根据第一次赋值数据的类型来推断其类型，编译结束后其类型就已经被确定。`比如上面的name已经被确定为String类型，不能在复制其他的类型，否则会报错。但是也可以通过指定类型的方式，来改变变量类型。比如：

```
name = 123; //报错
name = '张三' // 不报错
复制代码
```

**2. 显式声明可以推断出的类型：**

比如： 使用String ,name被确定为String类型了。

```
String name = '张三';
复制代码
```

**3. dynamic 它的类型可以随意改变。**

如果要明确说明不需要任何类型， 需要使用特殊类型 dynamic。

![img](https://user-gold-cdn.xitu.io/2020/7/25/173854e888d1f4e4?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)**4. 默认值——未初始化的变量默认值是 null。**

即使变量是数字 类型默认值也是 null，因为在 Dart 中一切都是对象，数字类型 也不例外。

![img](https://user-gold-cdn.xitu.io/2020/7/25/173854e88a3c1d62?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

## **Dart 常量：**

使用过程中从来不会被修改的变量， 可以使用 final 或 const, 而不是 var 或者其他类型.

```
使用final或者const修饰的变量，变量类型可以省略.
```

**final**

```
final name = 'Bob';
final String nickname = 'Bobby';
复制代码
```

**const**

```
const bar = 1000000; 
const double atm = 1.01325 * bar; 
复制代码
```

**final和const的区别：**

```
final a=new DateTime.now();
print(a);   //2019-05-10 15:59:02.966122
//const a=new DateTime.now();   //报错了
复制代码
```

两者区别在于：const 变量是一个编译时常量，final变量在第一次使用时被初始化。

## **Dart的命名规则：**

1. 变量名称必须由数字、字母、下划线和美元符($)组成。
2. 注意：标识符开头不能是数字
3. 标识符不能是保留字和关键字。
4. 变量的名字是区分大小写的如: age和Age是不同的变量。在实际的运用中,也建议,不要用一个单词大小写区分两个变量。
5. 标识符(变量名称)一定要见名思意 ：变量名称建议用名词，方法名称建议用动词 。

# Dart 数据类型

## **字符串类型**

1、字符串定义的几种方式

```
var str1='this is str1';
String str1='this is str1';
复制代码
```

2、字符串的拼接

```
String str1='你好';

String str2='Dart';

print("$str1 $str2"); //你好Dart
  
//或者使用 +运算符
print(str1+str2); //你好Dart
复制代码
```

1. 使用连续三个单引号或者三个双引号实现多行字符串对象的创建：

![img](https://user-gold-cdn.xitu.io/2020/7/25/173854e88a974185?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

## **数值类型**

**Number类型分为int类型和double类型。**

int 和 double 都是 num. 的亚类型。 num 类型包括基本运算 +， -， /， 和 *， 以及 abs()， ceil()， 和 floor()， 等函数方法.

**1、int 必须是整型**

```
int a=123;

a=45;

print(a);
复制代码
```

**2、double 既可以是整型 也可是浮点型**

```
double b=23.5;

b=24;

print(b);
复制代码
```

## **布尔类型**

Dart 使用 bool 类型表示布尔值。 Dart 只有字面量 true and false 是布尔类型.

```
bool flag1=true;

print(flag1);

bool flag2=false;

print(flag2);
复制代码
 bool flag=true;

if(flag){
  print('真');
}else{
  print('假');
}
复制代码
```

## **List集合类型**

在 Dart 中的 Array 就是 List 对象， 通常称之为 List 。

**1、第一种定义List的方式**

```
var arr1=['aaa','bbbb','cccc'];

print(arr1);

print(arr1.length);

print(arr1[1]);
复制代码
```

**2、第二种定义List的方式**

元素的类型不确定。

```
var arr2=new List();
arr2.add('张三');
arr2.add('李四');
arr2.add('王五');

print(arr2);
print(arr2[2]);
  
//也可以使用下面这种方法 
  
var arr3=new List<dynamic>();

arr3.add('张三');

arr3.add(123);

print(arr3);
复制代码
```

**3、定义List指定类型** 指定List的每一个元素都是String类型。

```
main() {
  var arr3=new List<String>();

  arr3.add('张三');

  arr3.add('lisa');

  print(arr3);

}

复制代码
```

## **Map 类型**

通常来说， Map 是用来关联 keys 和 values 的对象。 keys 和 values 可以是任何类型的对象。在一个 Map 对象中一个 key 只能出现一次。 但是 value 可以出现多次。 Dart 中 Map 通过 Map 字面量 和 Map 类型来实现。

**第一种定义 Map的方式**

```
var person={
"name":"张三",
"age":20,
"work":["程序员","送外卖"]
};

print(person);

print(person["name"]);

print(person["age"]);

print(person["work"]);
复制代码
```

**第二种定义 Map的方式**

```
var p=new Map();

p["name"]="李四";
p["age"]=22;
p["work"]=["程序员","送外卖"];
print(p);

print(p["age"]);
复制代码
```

## **Set类型**

Set是没有顺序且不能重复的集合，所以不能通过索引去获取值

```
var s=new Set();
s.add('香蕉');
s.add('苹果');
s.add('苹果');

print(s);   //{香蕉, 苹果}
print(s.toList()); //[香蕉, 苹果] 


  List myList=['香蕉','苹果','西瓜','香蕉','苹果','香蕉','苹果'];

  var s=new Set();
复制代码
```

可以用来数组去重

```
 List myList=['香蕉','苹果','西瓜','香蕉','苹果','香蕉','苹果'];

  var s=new Set();

  s.addAll(myList);

  print(s); //{香蕉, 苹果, 西瓜}

  print(s.toList());  //[香蕉, 苹果, 西瓜]
复制代码
```

## **其他类型**

**Runes**

Rune是UTF-32编码的字符串。它可以通过文字转换成符号表情或者代表特定的文字。

**Symbols**

Symbol对象表示在Dart程序中声明的运算符或标识符。

# Dart 常用操作符

## **算数运算符**

```
int a=13;
int b=5;

print(a+b);   //加
print(a-b);   //减
print(a*b);   //乘
print(a/b);   //除
print(a%b);   //取余
print(a~/b);  //取整
复制代码
```

## **关系运算符**

```
  int a=5;
  int b=3;

  print(a==b);   //判断是否相等
  print(a!=b);   //判断是否不等
  print(a>b);   //判断是否大于
  print(a<b);   //判断是否小于
  print(a>=b);   //判断是否大于等于
  print(a<=b);   //判断是否小于等于
复制代码
```

## **赋值运算符**

使用 = 为变量赋值。 使用 ??= 运算符时，只有当被赋值的变量为 null 时才会赋值给它。

```
int b=6;
b??=23;
print(b); //6


int b;
b??=23;
print(b); //23
复制代码
```

如果a为空，则b=10;

```
var a=22;
var b= a ?? 10;

print(b); //22
复制代码
```

## **类型转换**

```
Number类型转换成String类型 toString()
String类型转成Number类型 int.parse()
```

**字符串转化为数值**

```
String str='123';

var myNum=int.parse(str);

print(myNum is int);



String str='123.1';

var myNum=double.parse(str);

print(myNum is double);


String price='12';

var myNum=double.parse(price);

print(myNum);

print(myNum is double);     
复制代码
```

**可以通过try ...catch做判断**

```
String price='';
try{
  var myNum=double.parse(price);

  print(myNum);

}catch(err){
     print(0);
} 

复制代码
```

**数值转化为字符串**

```
var myNum=12;

var str=myNum.toString();

print(str is String);
复制代码
```

**其他类型转换成Booleans类型**

```
//isEmpty:判断字符串是否为空  

var str='';
if(str.isEmpty){
  print('str空');
}else{
  print('str不为空');
}


var myNum=123;

if(myNum==0){
   print('0');
}else{
  print('非0');
}


var myNum;

if(myNum==0){
   print('0');
}else{
  print('非0');
}



var myNum;
if(myNum==null){
   print('空');
}else{
  print('非空');
}


复制代码
```

# Dart 集合类型详解

## **List里面常用的属性和方法：**

**List里面的属性：**

```
List myList=['香蕉','苹果','西瓜'];
    print(myList.length);
    print(myList.isEmpty);
    print(myList.isNotEmpty);
    print(myList.reversed);  //对列表倒序排序
    var newMyList=myList.reversed.toList();
    print(newMyList);
复制代码
```

**List里面的方法：**

```
List myList=['香蕉','苹果','西瓜'];
myList.add('桃子');   //增加数据  增加一个

myList.addAll(['桃子','葡萄']);  //拼接数组

print(myList);

print(myList.indexOf('苹x果'));    //indexOf查找数据 查找不到返回-1  查找到返回索引值


myList.remove('西瓜'); //删除  传入具体值

myList.removeAt(1);  //删除  传入索引值

print(myList);

myList.fillRange(1, 2,'aaa');  //修改

myList.fillRange(1, 3,'aaa');  


myList.insert(1,'aaa');      // 指定位置插入  插入一个

myList.insertAll(1, ['aaa','bbb']);  //插入 多个 指定位置插入List

var str=myList.join('-');   //list转换成字符串

print(str);

print(str is String);  //true
var list=str.split('-'); // 字符串转化成List

var arr = (11,22,33);
var arr2 = arr.toList(); //其他类型转换成List 
复制代码
遍历List的方法：forEach map where any every，用法和ES6一样，此处不再赘述。
```

## **Set:**

用它最主要的功能就是去除数组重复内容; Set是没有顺序且不能重复的集合，所以不能通过索引去获取值;

```
var s=new Set();
s.add('香蕉');
s.add('苹果');
s.add('苹果');

print(s);   //{香蕉, 苹果}

print(s.toList()); 
复制代码
List myList=['香蕉','苹果','西瓜','香蕉','苹果','香蕉','苹果'];

var s=new Set();

s.addAll(myList);

print(s);

print(s.toList());
复制代码
s.forEach((value)=>print(value));
复制代码
```

## **Map: 映射(Map)是无序的键值对：**

**常用属性：**

```
Map person={
"name":"张三",
"age":20,
"sex":"男"
};

print(person.keys.toList());
print(person.values.toList());
print(person.isEmpty); //是否为空
print(person.isNotEmpty); //是否不为空
复制代码
```

**常用方法：**

```
Map person={
  "name":"张三",
  "age":20,
  "sex":"男"
};

person.addAll({ //合并映射  给映射内增加属性
  "work":['敲代码','送外卖'],
  "height":160
});

print(person);

person.remove("sex"); //删除指定key的数据
print(person);
 
print(person.containsValue('张三')); //查看映射内的值  返回true/false

person.forEach((key,value){            
    print("$key---$value");
});
复制代码
```

# Dart 函数

Dart 是一门真正面向对象的语言， 甚至其中的函数也是对象，并且有它的类型 Function 。

## **自定义方法**

**无返回值** 前面加void关键字表示改方法无返回值。

```
void printInfo(){
  print('我是一个自定义方法');
}
复制代码
```

**返回一个int类型的数值**

```
int getNum(){
  var myNum=123;
  return myNum;
}
复制代码
```

**返回字符串类型**

```
String printUserInfo(){
  return 'this is str';
}
复制代码
```

**返回List类型**

```
List getList(){
  return ['111','2222','333'];
}
复制代码
```

**main() 函数**

任何应用都必须有一个顶级 main() 函数，作为应用服务的入口。 main() 函数返回值为空，参数为一个可选的 List 。

```
void main(){

  print('调用系统内置的方法');

  printInfo();
  var n=getNum();
  print(n);


  print(printUserInfo());


  print(getList());

  print(getList());
}
复制代码
```

## **方法的作用域**

```
  void get(){

      aaa(){

          print(getList());
          print('aaa');
      }
      aaa();
  }

  // aaa();  错误写法 

  get();  //调用方法
}
复制代码
```

## **方法传参**

定义一个方法 求1到这个数的所有数的和 ，返回值是一个int类型。

```
int sumNum(int n){ //接收的参数必须事int类型，否则报错
  var sum=0;
  for(var i=1;i<=n;i++)
  {
    sum+=i;
  }
  return sum;
} 

var sum=sumNum(5);

复制代码
```

定义一个方法然后打印用户信息,返回值是String类型， 穿的参数是String和int类型。

```
String printUserInfo(String username,int age){  //行参
    return "姓名:$username---年龄:$age";
}

print(printUserInfo('张三',20)); //实参
复制代码
```

## **定义一个带可选参数的方法**

包装一组函数参数，用[]标记为可选的位置参数，并放在参数列表的最后面：

```
String printUserInfo(String username,[int age]){  //行参

  if(age!=null){
    return "姓名:$username---年龄:$age";
  }
  return "姓名:$username---年龄保密";

}

print(printUserInfo('张三',21)); //实参

print(printUserInfo('张三'));
复制代码
```

## **定义一个带默认参数的方法**

如果调用该方法时，不传可选参数，可以使用默认参数。

```
String printUserInfo(String username,[String sex='男',int age]){  //行参

  if(age!=null){
    return "姓名:$username---性别:$sex--年龄:$age";
  }
  return "姓名:$username---性别:$sex--年龄保密";

}

print(printUserInfo('张三'));

print(printUserInfo('小李','女'));

print(printUserInfo('小李','女',30));
复制代码
```

## **定义一个命名参数的方法**

定义函数时，使用{param1, param2, …}，放在参数列表的最后面，用于指定命名参数。例如：

**调用函数时，可以使用指定命名参数。例如：paramName: value**

```
String printUserInfo(String username,{int age,String sex='男'}){  //行参

  if(age!=null){
    return "姓名:$username---性别:$sex--年龄:$age";
  }
  return "姓名:$username---性别:$sex--年龄保密";
}

print(printUserInfo('张三',age:20,sex:'未知'));
复制代码
```

命名参数的方法和可选参数的方法区别：

1. 命名参数方法使用的是{};
2. 可选参数使用的是[];
3. 命名参数方法必须根据形参的字段名字传值;

```
可选参数可以是命名参数或者位置参数，但一个参数只能选择其中一种方式修饰。
```

## **函数是一等对象**

一个函数可以作为另一个函数的参数。例如：

```
//方法
fn1(){
  print('fn1');
}

//方法
fn2(fn){

  fn();
}

//调用fn2这个方法 把fn1这个方法当做参数传入
fn2(fn1);
复制代码
```

## **匿名方法**

多数函数是有名字的， 比如 main() 和 getInfo()。 也可以创建没有名字的函数，这种函数被称为 匿名函数.

匿名函数和命名函数看起来类似 - 在括号里面可以传参，参数使用都好分割。

```
var printNum=(int n, int m){

  print(m*n);
};

printNum(12,20); //240
复制代码
```

## **自执行方法**

```
((int n){
  print(n);
  print('我是自执行方法');
})(12);
复制代码
```

## **方法的递归**

请求100以内数值的和

```
var sum=0;
fn(int n){

    sum+=n;

    if(n==0){
      return;
    }
    fn(n-1);
}

fn(100);
print(sum);

复制代码
```

## **闭包**

闭包 即一个函数对象，即使函数对象的调用在它原始作用域之外， 依然能够访问在它词法作用域内的变量。

```
/// 返回一个函数，返回的函数参数与 [addBy] 相加。
Function makeAdder(num addBy) {
  return (num i) => addBy + i;
}

void main() {
  // 创建一个加 2 的函数。
  var add2 = makeAdder(2);

  // 创建一个加 4 的函数。
  var add4 = makeAdder(4);

  print(add2(3) == 5); //true
  print(add4(3) == 7); //true
}
复制代码
```

# Dart 中的类

```
Dart所有的东西都是对象，所有的对象都继承自Object类。
```

Dart是一门使用类和单继承的面向对象语言，所有的对象都是类的实例，并且所有的类都是Object的子类。

## **Dart 中定义类 使用类**

```
class Person{
  String name="张三";
  int age=23;
  void getInfo(){
      // print("$name----$age");
      //等同于下面
      print("${this.name}----${this.age}");
  }
  void setInfo(int age){
    this.age=age;
  }
}
  
void main() {
  //实例化类
  Person p1=new Person();
  print(p1.name); //张三

  p1.setInfo(28);
  p1.getInfo(); //张三----28
}
复制代码
```

## **Dart 中类的默认构造函数**

**定义一个默认的构造函数**

```
class Person{
  String name='张三';
  int age=20; 
  
  //默认构造函数
  Person(){
    print('这是构造函数里面的内容  这个方法在实例化的时候触发');
  }
  
  void printInfo(){   
    print("${this.name}----${this.age}");
  }
}
void main() {
  //实例化类
  Person p1=new Person();
  print(p1.name);

  p1.printInfo();
}
复制代码
```

**给构造函数传参**

```
class Person{
  String name;
  int age; 
  
  //默认构造函数
  Person(String name,int age){
      this.name=name;
      this.age=age;
  }
  
  void printInfo(){   
    print("${this.name}----${this.age}");
  }
}

void main() {
  //实例化类
  Person p1=new Person('张三',20);
  print(p1.name);  //张三

  p1.printInfo(); //张三----20
}
复制代码
```

**构造函数的简写方法：**

```
class Person{
  String name;
  int age;
  
  //默认构造函数的简写
  Person(this.name,this.age);
  
  void printInfo(){   
    print("${this.name}----${this.age}");
  }
}

void main() {
  //实例化类
  Person p1=new Person('张三',20);
  print(p1.name);  //张三

  p1.printInfo(); //张三----20
}
复制代码
```

## **Dart中的命名构造函数**

dart里面默认构造函数只有一个，但是命名构造函数可以有多个。

```
class Person{
  String name;
  int age; 
  
  //默认构造函数的简写
  Person(this.name, this.age);
  
  //我是命名构造函数
  Person.now(){
    print('我是命名构造函数');
  }

  //给命名构造函数传参
  Person.setInfo(String name, int age){
    this.name=name;
    this.age=age;
  }

  void printInfo(){   
    print("${this.name}----${this.age}");
  }
}

void main() {
  Person p1=new Person('张三', 20);   //默认实例化类的时候调用的是 默认构造函数

  Person p2=new Person.now();   //命名构造函数
  
  Person p3=new Person.setInfo('李四',30);
  p1.printInfo(); //张三----20
  p3.printInfo(); //李四----30

}
复制代码
```

## **Dart中的私有方法 和私有属性**

Dart和其他面向对象语言不一样，Data中没有 public private protected这些访问修饰符合

但是我们可以使用_把一个属性或者方法定义成私有。

```
新建一个单独的 Animal类
class Animal{

  String _name;   //私有属性
  int age; 
  
  //默认构造函数的简写
  Animal(this._name,this.age);

  void printInfo(){   
    print("${this._name}----${this.age}");
  }

  String getName(){ 
    return this._name;
  } 
  void _run(){
    print('这是一个私有方法');
  }

  execRun(){
    this._run();  //类里面方法的相互调用
  }
}
复制代码
```

//调用私有方法

```
import 'lib/Animal.dart';

void main(){
 
 Animal a=new Animal('小狗', 3);

 print(a.getName());

  a.execRun();   //间接的调用私有方法
 

}
复制代码
```

## **类中的getter和setter修饰符的用法**

```
class Rect{
  num height;
  num width; 
  
  Rect(this.height,this.width);
  get area{
    return this.height*this.width;
  }
  set areaHeight(value){
    this.height=value;
  }
}

void main(){
  Rect r=new Rect(10,4);
  // print("面积:${r.area()}");   
  r.areaHeight=6;

  print(r.area); //24

}
复制代码
```

## **Dart中 初始化实例变量**

Dart中我们也可以在构造函数体运行之前初始化实例变量。

```
class Rect{

  int height;
  int width;
  Rect():height=2,width=10{
    
    print("${this.height}---${this.width}"); //2---10
  }
  getArea(){
    return this.height*this.width;
  } 
}

void main(){
  Rect r=new Rect();
  print(r.getArea());  //10
   
}
复制代码
```

## **Dart 一个类实现多个接口**

定义2个抽象类A和B,在抽象类中定义的属性和方法，在接口中必须实现。

```
abstract class A{
  String name;
  printA();
}

abstract class B{
  printB();
}

class C implements A,B{  
  @override
  String name;  
  
  @override
  printA() {
    print('printA');
  }
  
  @override
  printB() {
    // TODO: implement printB
    return null;
  }

  
}

复制代码
void main(){

  C c=new C();
  
  c.printA();  //printA

}
复制代码
```

## **Dart中的mixins**

在Dart中可以使用mixins实现类似多继承的功能。

因为mixins使用的条件[Dart2.x版本]：

1、作为mixins的类只能继承自Object，不能继承其他类；

2、作为mixins的类不能有构造函数；

3、一个类可以mixins多个mixins类；

4、mixins绝不是继承，也不是接口，而是一种全新的特性

```
class A {
  String info="this is A";
  void printA(){
    print("A");
  }
}

class B {
  void printB(){
    print("B");
  }
}

class C with A,B{
  
}

void main(){
  
  var c=new C();  
  c.printA();
  c.printB();
  print(c.info);


}
复制代码
```

# Dart中类的 静态成员 操作符 类的继承

## **Dart 类中的静态成员 静态方法**

Dart中的静态成员:

1、使用static 关键字来实现类级别的变量和函数

2、静态方法不能访问非静态成员，非静态方法可以访问静态成员

**定义一个有静态属性和静态方法的类**

```
class Person {
  static String name = '张三';
  static void show() {
    print(name);
  }
}

main(){
  print(Person.name);
  Person.show();  
}
复制代码
```

## **Dart中的对象操作符**

1. ? 条件运算符 （了解）
2. as 类型转换
3. is 类型判断
4. .. 级联操作

```
class Person {
  String name;
  num age;
  Person(this.name,this.age);
  void printInfo() {
    print("${this.name}---${this.age}");  
  }
  
}
复制代码
```

**? 条件运算符**

如果p存在，就调用printInfo方法。

```
Person p=new Person('张三', 20);
   p?.printInfo();
复制代码
```

**is 判断类型**

```
Person p=new Person('张三', 20);

if(p is Person){
    p.name="李四";
} 
p.printInfo();
print(p is Object);
复制代码
```

**as 类型转换**

```
 p1=new Person('张三1', 20);

// p1.printInfo();

(p1 as Person).printInfo();
复制代码
```

**.. 级联操作 （连缀）**

```
 Person p1=new Person('张三1', 20);

   p1.printInfo();

   p1.name='张三222';
   p1.age=40;
   p1.printInfo();
复制代码
等同于
Person p1=new Person('张三1', 20);

p1.printInfo();

p1..name="李四"
..age=30
..printInfo();
复制代码
```

## **Dart 类的继承-简单继承**

Dart中的类的继承：
1、子类使用extends关键词来继承父类

2、子类会继承父类里面可见的属性和方法 但是不会继承构造函数

3、子类能复写父类的方法 getter和setter

```
class Person {
  String name='张三';
  num age=20; 
  void printInfo() {
    print("${this.name}---${this.age}");  
  } 
}
class Web extends Person{

}

main(){   

  Web w=new Web();
  print(w.name);
  w.printInfo();
 
}
复制代码
```

## **给父类构造函数传参**

```
class Person {
  String name;
  num age; 
  Person(this.name,this.age);
  void printInfo() {
    print("${this.name}---${this.age}");  
  }
}


class Web extends Person{
  Web(String name, num age) : super(name, age){

  }
  
}

main(){ 
  Web w=new Web('张三', 12);

  w.printInfo();


}
复制代码
```

继承的时候也可以给自己的类传参

```
class Web extends Person{
  String sex;
  Web(String name, num age,String sex) : super(name, age){
    this.sex=sex;
  }
  run(){
   print("${this.name}---${this.age}--${this.sex}");  
  }
  
}

Web w=new Web('张三', 12,"男");

w.printInfo();
  
复制代码
```

## **给命名构造函数传参**

```
class Person {
  String name;
  num age; 
  Person(this.name,this.age);
  Person.xxx(this.name,this.age);
  void printInfo() {
    print("${this.name}---${this.age}");  
  }
}


class Web extends Person{
  String sex;
  Web(String name, num age,String sex) : super.xxx(name, age){
    this.sex=sex;
  }
  run(){
   print("${this.name}---${this.age}--${this.sex}");  
  }
  
}

main(){ 
  Web w=new Web('张三', 12,"男");

  w.printInfo();

  w.run();

}
复制代码
```

## **Dart 类的继承 覆写父类的方法**

```
class Person {
  String name;
  num age; 
  Person(this.name,this.age);
  void printInfo() {
    print("${this.name}---${this.age}");  
  }
  work(){
    print("${this.name}在工作...");
  }
}

class Web extends Person{
  Web(String name, num age) : super(name, age);

  run(){
    print('run');
  }
  //覆写父类的方法
  @override       //可以写也可以不写  建议在覆写父类方法的时候加上 @override 
  void printInfo(){
     print("姓名：${this.name}---年龄：${this.age}"); 
  }
  @override
  work(){
    print("${this.name}的工作是写代码");
  }

}
main(){ 

  Web w=new Web('李四',20);

  w.printInfo();

  w.work();
 
}
复制代码
```

## **Dart 自类里面调用父类的方法**

```
class Person {
  String name;
  num age; 
  Person(this.name,this.age);
  void printInfo() {
    print("${this.name}---${this.age}");  
  }
  work(){
    print("${this.name}在工作...");
  }
}

class Web extends Person{
  Web(String name, num age) : super(name, age);

  run(){
    print('run');
    super.work();  //自类调用父类的方法
  }
  //覆写父类的方法
  @override       //可以写也可以不写  建议在覆写父类方法的时候加上 @override 
  void printInfo(){
     print("姓名：${this.name}---年龄：${this.age}"); 
  }

}


main(){ 

  Web w=new Web('李四',20);

  // w.printInfo();

  w.run();
 
}
复制代码
```

# Dart中的抽象类 多态 以及接口

**extends抽象类 和 implements的区别：**

1、如果要复用抽象类里面的方法，并且要用抽象方法约束自类的话我们就用extends继承抽象类

2、如果只是把抽象类当做标准的话我们就用implements实现抽象类

## **Dart中抽象类**

Dart抽象类主要用于定义标准，子类可以继承抽象类，也可以实现抽象类接口。

1、抽象类通过abstract 关键字来定义

2、Dart中的抽象方法不能用abstract声明，Dart中没有方法体的方法我们称为抽象方法。

3、如果子类继承抽象类必须得实现里面的抽象方法

4、如果把抽象类当做接口实现的话必须得实现抽象类里面定义的所有属性和方法。

5、抽象类不能被实例化，只有继承它的子类可以

```
abstract class Animal{
  eat();   //抽象方法
  run();  //抽象方法  
  printInfo(){
    print('我是一个抽象类里面的普通方法');
  }
}

class Dog extends Animal{
  @override
  eat() {
     print('小狗在吃骨头');
  }

  @override
  run() {
    // TODO: implement run
    print('小狗在跑');
  }  
}
class Cat extends Animal{
  @override
  eat() {
    // TODO: implement eat
    print('小猫在吃老鼠');
  }

  @override
  run() {
    // TODO: implement run
    print('小猫在跑');
  }

}

main(){

  Dog d=new Dog();
  d.eat();
  d.printInfo();

   Cat c=new Cat();
  c.eat();

  c.printInfo();


  // Animal a=new Animal();   //抽象类没法直接被实例化

}
复制代码
```

## **Dart中多态**

1. 允许将子类类型的指针赋值给父类类型的指针, 同一个函数调用会有不同的执行效果 。
2. 子类的实例赋值给父类的引用。
3. `多态就是父类定义一个方法不去实现，让继承他的子类去实现，每个子类有不同的表现`。

```
abstract class Animal{
  eat();   //抽象方法 
}

class Dog extends Animal{
  @override
  eat() {
     print('小狗在吃骨头');
  }
  run(){
    print('run');
  }
}
class Cat extends Animal{
  @override
  eat() {   
    print('小猫在吃老鼠');
  }
  run(){
    print('run');
  }
}

main(){
  Animal d=new Dog();

  d.eat(); 

  Animal c=new Cat();

  c.eat();

}
复制代码
```

## **Dart中的接口**

```
dart的接口没有interface关键字定义接口，而是普通类或抽象类都可以作为接口被实现。
```

同样使用implements关键字进行实现。

但是dart的接口有点奇怪，如果实现的类是普通类，会将普通类和抽象中的属性的方法全部需要覆写一遍。

而因为抽象类可以定义抽象方法，普通类不可以，所以一般如果要实现像Java接口那样的方式，一般会使用抽象类。

建议使用抽象类定义接口。

```
定义一个DB库 支持 mysql mssql mongodb ， mysql mssql mongodb三个类里面都有同样的方法
abstract class Db{   //当做接口   接口：就是约定 、规范
    String uri;      //数据库的链接地址
    add(String data);
    save();
    delete();
}

class Mysql implements Db{
  
  @override
  String uri;

  Mysql(this.uri);

  @override
  add(data) {
    // TODO: implement add
    print('这是mysql的add方法'+data);
  }

  @override
  delete() {
    // TODO: implement delete
    return null;
  }

  @override
  save() {
    // TODO: implement save
    return null;
  }

  remove(){
      
  }

  
}

class MsSql implements Db{
  @override
  String uri;
  @override
  add(String data) {
    print('这是mssql的add方法'+data);
  }

  @override
  delete() {
    // TODO: implement delete
    return null;
  }

  @override
  save() {
    // TODO: implement save
    return null;
  }

}

main() {

  Mysql mysql=new Mysql('xxxxxx');

  mysql.add('1243214');  
}
复制代码
```

# Dart中一个类实现多个接口 以及Dart中的Mixins

## **Dart中一个类实现多个接口：**

```
abstract class A{
  String name;
  printA();
}

abstract class B{
  printB();
}

class C implements A,B{  
  @override
  String name; 
  
  @override
  printA() {
    print('printA');
  }
  
  @override
  printB() {
    // TODO: implement printB
    return null;
  }
}

void main(){

  C c=new C();
  c.printA();
}
复制代码
```

## **Dart中的mixins**

```
Mixin 是复用类代码的一种途径， 复用的类可以在不同层级，之间可以不存在继承关系。
通过 with 后面跟一个或多个混入的名称，来 使用 Mixin 。
```



因为mixins使用的条件，随着Dart版本一直在变，这里讲的是Dart2.x中使用mixins的条件：

1、作为mixins的类只能继承自Object，不能继承其他类;

2、作为mixins的类不能有构造函数;

3、一个类可以mixins多个mixins类;

4、mixins绝不是继承，也不是接口，而是一种全新的特性;

```
class A {
  String info="this is A";
  void printA(){
    print("A");
  }
}

class B {
  void printB(){
    print("B");
  }
}

class C with A,B{
  
}

void main(){  
  var c=new C();  
  c.printA();
  c.printB();
  print(c.info);
}
复制代码
```

**给父类传参**

```
class Person{
  String name;
  num age;
  Person(this.name,this.age);
  printInfo(){
    print('${this.name}----${this.age}');
  }
  void run(){
    print("Person Run");
  }
}

class A {
  String info="this is A";
  void printA(){
    print("A");
  }
  void run(){
    print("A Run");
  }
}

class B {  
  void printB(){
    print("B");
  }
  void run(){
    print("B Run");
  }
}

class C extends Person with B,A{

  //给父类传参
  C(String name, num age) : super(name, age);
}

void main(){  
  var c=new C('张三',20);  
  c.printInfo();
  c.run();
}
复制代码
```

## **Dart中的mixins 的类型**

```
mixins的类型就是其超类的子类型。
class A {
  String info="this is A";
  void printA(){
    print("A");
  }
}

class B {
  void printB(){
    print("B");
  }
}

class C with A,B{
  
}

void main(){  
  var c=new C();  
  print(c is C);    //true
  print(c is A);    //true
  print(c is B);   //true
}
复制代码
```

# Dart 中的泛型

```
通俗理解：泛型就是解决 类 接口 方法的复用性、以及对不特定数据类型的支持(类型校验)
```

## **Dart中的泛型方法**

**只能返回string类型的数据**

```
String getData(String value){
      return value;
  }
复制代码
```

**同时支持返回 string类型 和int类型 （代码冗余）**

```
 String getData1(String value){
      return value;
  }

  int getData2(int value){
      return value;
  }
复制代码
```

**同时返回 string类型 和number类型 不指定类型可以解决这个问题**

```
getData(value){
      return value;
  }
复制代码
```

不指定类型放弃了类型检查。我们现在想实现的是传入什么 返回什么。比如:传入number 类型必须返回number类型 传入 string类型必须返回string类型

```
 getData<T>(T value){
      return value;
  }
  
print(getData<int>(12)); //传入类型和返回类型一致
复制代码
```

## **Dart中的泛型类**

可以给数组添加int类型，也可以添加String类型的元素。

```
class PrintClass<T>{
      List list=new List<T>();
      void add(T value){
          this.list.add(value);
      }
      void printInfo(){          
          for(var i=0;i<this.list.length;i++){
            print(this.list[i]);
          }
      }
 }
 
 PrintClass p=new PrintClass<int>();

p.add(12);

p.add(23);

p.printInfo();
复制代码
```

## **Dart中的泛型接口**

实现数据缓存的功能：有文件缓存、和内存缓存。内存缓存和文件缓存按照接口约束实现。

1、定义一个泛型接口 约束实现它的子类必须有getByKey(key) 和 setByKey(key,value)

2、要求setByKey的时候的value的类型和实例化子类的时候指定的类型一致

```
abstract class Cache<T>{
  getByKey(String key);
  void setByKey(String key, T value);
}

class FlieCache<T> implements Cache<T>{
  @override
  getByKey(String key) {    
    return null;
  }

  @override
  void setByKey(String key, T value) {
   print("我是文件缓存 把key=${key}  value=${value}的数据写入到了文件中");
  }
}

class MemoryCache<T> implements Cache<T>{
  @override
  getByKey(String key) {   
    return null;
  }

  @override
  void setByKey(String key, T value) {
       print("我是内存缓存 把key=${key}  value=${value} -写入到了内存中");
  }
}


void main(){

 MemoryCache m=new MemoryCache<Map>();

 m.setByKey('index', {"name":"张三","age":20});
}
复制代码
```

# Dart的异步

Dart是基于单线程模型的语言。在Dart也有自己的进程（或者叫线程）机制，名叫isolate。APP的启动入口main函数就是一个isolate。

Dart线程中有一个消息循环机制（event loop）和两个队列（event queue和microtask queue）。

- event queue包含所有外来的事件：I/O，mouse events，drawing events，timers，isolate之间的message等。任意isolate中新增的event（I/O，mouse events，drawing events，timers，isolate的message）都会放入event queue中排队等待执行。
- microtask queue只在当前isolate的任务队列中排队，优先级高于event queue.

如果在event中插入microtask，当前event执行完毕即可插队执行microtask。如果没有microtask，就没办法插队了，也就是说，microtask queue的存在为Dart提供了给任务队列插队的解决方案。

当main方法执行完毕退出后，event loop就会以FIFO(先进先出)的顺序执行microtask，当所有microtask执行完后它会从event queue中取事件并执行。如此反复，直到两个队列都为空，如下流程图：

![img](https://user-gold-cdn.xitu.io/2020/7/25/173868104aadc2b3?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

## **异步执行**

那么在Dart中如何让你的代码异步执行呢？很简单，把要异步执行的代码放在微任务队列或者事件队列里就行了。

- 可以调用scheduleMicrotask来让代码以微任务的方式异步执行

```
 scheduleMicrotask((){
        print('a microtask');
    });
复制代码
```

- 可以调用Timer.run来让代码以Event的方式异步执行

```
Timer.run((){
       print('a event');
   });
复制代码
```

和JS一样，仅仅使用回调函数来做异步的话很容易陷入“回调地狱（Callback hell）”，为了避免这样的问题，JS引入了Promise。同样的, Dart引入了Future。

## **Future**

Future与JavaScript中的Promise非常相似，表示一个异步操作的最终完成（或失败）及其结果值的表示。简单来说，它就是用于处理异步操作的，异步处理成功了就执行成功的操作，异步处理失败了就捕获错误或者停止后续操作。一个Future只会对应一个结果，要么成功，要么失败。

要使用Future的话需要引入`dart.async`

```
import 'dart:async';
复制代码
```

创建一个立刻在事件队列里运行的Future:

```
Future(() => print('立刻在Event queue中运行的Future'));
复制代码
```

**Future.then**

使用Future.delayed 创建了一个延时任务（实际项目可能是一个真正的耗时任务，比如一次网络请求），即2秒后返回结果字符串"hi world!"，然后我们在then中接收异步结果并打印结果：

```
Future.delayed(new Duration(seconds: 2),(){
   return "hi world!";
}).then((data){
   print(data);
});
复制代码
```

**Future.catchError**

如果异步任务发生错误，我们可以在catchError中捕获错误，我们将上面示例改为：

```
Future.delayed(new Duration(seconds: 2),(){
   //return "hi world!";
   throw AssertionError("Error");  
}).then((data){
   //执行成功会走到这里  
   print("success");
}).catchError((e){
   //执行失败会走到这里  
   print(e);
});
复制代码
then方法还有一个可选参数onError，我们也可以它来捕获异常：
Future.delayed(new Duration(seconds: 2), () {
    //return "hi world!";
    throw AssertionError("Error");
}).then((data) {
    print("success");
}, onError: (e) {
    print(e);
});
复制代码
```

**Future.whenComplete**

有些时候，我们会遇到无论异步任务执行成功或失败都需要做一些事的场景。

```
Future.delayed(new Duration(seconds: 2),(){
   //return "hi world!";
   throw AssertionError("Error");
}).then((data){
   //执行成功会走到这里 
   print(data);
}).catchError((e){
   //执行失败会走到这里   
   print(e);
}).whenComplete((){
   //无论成功或失败都会走到这里
});
复制代码
```

**Future.wait**

有些时候，我们需要等待多个异步任务都执行结束后才进行一些操作。

我们通过模拟Future.delayed 来模拟两个数据获取的异步任务，等两个异步任务都执行成功时，将两个异步任务的结果拼接打印出来。

```
Future.wait([
  // 2秒后返回结果  
  Future.delayed(new Duration(seconds: 2), () {
    return "hello";
  }),
  // 4秒后返回结果  
  Future.delayed(new Duration(seconds: 4), () {
    return " world";
  })
]).then((results){
  print(results[0]+results[1]);
}).catchError((e){
  print(e);
});
复制代码
```

## **Async/await**

Dart中的async/await 和JavaScript中的async/await功能和用法是一模一样的。

通过Future回调中再返回Future的方式虽然能避免层层嵌套，但是还是有一层回调，有没有一种方式能够让我们可以像写同步代码那样来执行异步任务而不使用回调的方式？答案是肯定的，这就要使用async/await了，下面我们先直接看代码，然后再解释，代码如下：

```
task() async {
   try{
    String id = await login("alice","******");
    String userInfo = await getUserInfo(id);
    await saveUserInfo(userInfo);
    //执行接下来的操作   
   } catch(e){
    //错误处理   
    print(e);   
   }  
}
复制代码
```

- async用来表示函数是异步的，定义的函数会返回一个Future对象，可以使用then方法添加回调函数。
- await 后面是一个Future，表示等待该异步任务完成，异步完成后才会往下走；await必须出现在 async 函数内部。