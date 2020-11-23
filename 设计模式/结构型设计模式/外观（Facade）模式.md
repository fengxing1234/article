---
title: 外观（Facade）模式
date: 2020-05-15 00:04:50
tags:
	- Java
	- 面试
	- 基础
	- 设计模式
categories: 
	- 设计模式
	- 结构型设计模式
---

在现实生活中，常常存在办事较复杂的例子，如办房产证或注册一家公司，有时要同多个部门联系，这时要是有一个综合部门能解决一切手续问题就好了。

软件设计也是这样，当一个系统的功能越来越强，子系统会越来越多，客户对系统的访问也变得越来越复杂。这时如果系统内部发生改变，客户端也要跟着改变，这违背了“开闭原则”，也违背了“迪米特法则”，所以有必要为多个子系统提供一个统一的接口，从而降低系统的耦合度，这就是外观模式的目标。

给出了客户去当地房产局办理房产证过户要遇到的相关部门。

![办理房产证过户的相关部门](http://c.biancheng.net/uploads/allimg/181115/3-1Q11515205E60.gif)

**定义：**

外观（Facade）模式的定义：是一种通过为多个复杂的子系统提供一个一致的接口，而使这些子系统更加容易被访问的模式。该模式对外有一个统一接口，外部应用程序不用关心内部子系统的具体的细节，这样会大大降低应用程序的复杂度，提高了程序的可维护性。

**类图：**

![外观模式的结构图](http://c.biancheng.net/uploads/allimg/181115/3-1Q115152143509.gif)

**模式的结构：**

- 外观（Facade）角色：为多个子系统对外提供一个共同的接口。
- 子系统（Sub System）角色：实现系统的部分功能，客户可以通过外观角色访问它。
- 客户（Client）角色：通过一个外观角色访问各个子系统的功能。

**解决的问题：**

**优点：**

- 降低了子系统与客户端之间的耦合度，使得子系统的变化不会影响调用它的客户类。
- 对客户屏蔽了子系统组件，减少了客户处理的对象数目，并使得子系统使用起来更加容易。
- 降低了大型软件系统中的编译依赖性，简化了系统在不同平台之间的移植过程，因为编译一个子系统不会影响其他的子系统，也不会影响外观对象。

**缺点：**

- 不能很好地限制客户使用子系统类。
- 增加新的子系统可能需要修改外观类或客户端的源代码，违背了“开闭原则”。

**适用场景：**

- 对分层结构系统构建时，使用外观模式定义子系统中每层的入口点可以简化子系统之间的依赖关系。
- 当一个复杂系统的子系统很多时，外观模式可以为系统设计一个简单的接口供外界访问。
- 当客户端与多个子系统之间存在很大的联系时，引入外观模式可将它们分离，从而提高子系统的独立性和可移植性。

## 案例

```
//外观角色
public class Facade {
    private SubSystem01 obj1 = new SubSystem01();
    private SubSystem02 obj2 = new SubSystem02();
    private SubSystem03 obj3 = new SubSystem03();

    public void method() {
        obj1.method1();
        obj2.method2();
        obj3.method3();
    }
}

/**
 * 子系统01
 */
public class SubSystem01 {

    public void method1() {
        System.out.println("子系统01的method1()被调用！");
    }
}

public class SubSystem02 {


    public void method2() {
        System.out.println("子系统02的method2()被调用！");
    }
}

public class SubSystem03 {
    public void method3() {
        System.out.println("子系统03的method3()被调用！");
    }
}

//客户端
private static void facadeMode() {
        Facade facade = new Facade();
        facade.method();
    }
```

**结果：**

```
子系统01的method1()被调用！
子系统02的method2()被调用！
子系统03的method3()被调用！
```

