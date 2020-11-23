---
title: 享元（Flyweight）模式
date: 2020-05-15 11:17:59
tags:
	- Java
	- 面试
	- 基础
	- 设计模式
categories: 
	- 设计模式
	- 结构型设计模式
---

在面向对象程序设计过程中，有时会面临要创建大量相同或相似对象实例的问题。创建那么多的对象将会耗费很多的系统资源，它是系统性能提高的一个瓶颈。例如，围棋和五子棋中的黑白棋子，图像中的坐标点或颜色，局域网中的路由器、交换机和集线器，教室里的桌子和凳子等。这些对象有很多相似的地方，如果能把它们相同的部分提取出来共享，则能节省大量的系统资源，这就是享元模式的产生背景。

**定义：**

享元（Flyweight）模式的定义：运用共享技术来有効地支持大量细粒度对象的复用。它通过共享已经存在的又橡来大幅度减少需要创建的对象数量、避免大量相似类的开销，从而提高系统资源的利用率。

**类图：**

下图是享元模式的结构图。图中的 UnsharedConcreteFlyweight 是与淳元角色，里面包含了非共享的外部状态信息 info；而 Flyweight 是抽象享元角色，里面包含了享元方法 operation(UnsharedConcreteFlyweight state)，非享元的外部状态以参数的形式通过该方法传入；ConcreteFlyweight 是具体享元角色，包含了关键字 key，它实现了抽象享元接口；FlyweightFactory 是享元工厂角色，它逝关键字 key 来管理具体享元；客户角色通过享元工厂获取具体享元，并访问具体享元的相关方法。

![享元模式的结构图](http://c.biancheng.net/uploads/allimg/181115/3-1Q115161342242.gif)

**模式的结构：**

享元模式中存在以下两种状态：

- 内部状态，即不会随着环境的改变而改变的可共享部分；
- 外部状态，指随环境改变而改变的不可以共享的部分。享元模式的实现要领就是区分应用中的这两种状态，并将外部状态外部化。下面来分析其基本结构和实现方法。

享元模式的主要角色有如下。

- 抽象享元角色（Flyweight）:是所有的具体享元类的基类，为具体享元规范需要实现的公共接口，非享元的外部状态以参数的形式通过方法传入。
- 具体享元（Concrete Flyweight）角色：实现抽象享元角色中所规定的接口。
- 非享元（Unsharable Flyweight)角色：是不可以共享的外部状态，它以参数的形式注入具体享元的相关方法中。
- 享元工厂（Flyweight Factory）角色：负责创建和管理享元角色。当客户对象请求一个享元对象时，享元工厂检査系统中是否存在符合要求的享元对象，如果存在则提供给客户；如果不存在的话，则创建一个新的享元对象。

**解决的问题：**

**优点：**

- 相同对象只要保存一份，这降低了系统中对象的数量，从而降低了系统中细粒度对象给内存带来的压力。

**缺点：**

- 为了使对象可以共享，需要将一些不能共享的状态外部化，这将增加程序的复杂性。
- 读取享元模式的外部状态会使得运行时间稍微变长。

**适用场景：**

- 系统中存在大量相同或相似的对象，这些对象耗费大量的内存资源。
- 大部分的对象可以按照内部状态进行分组，且可将不同部分外部化，这样每一个组只需保存一个内部状态。
- 由于享元模式需要额外维护一个保存享元的数据结构，所以应当在有足够多的享元实例时才值得使用享元模式。

## 案例

```
package com.zhyen.base.design_mode.flyweight_mode;

/**
 * 非享元（Unsharable Flyweight)角色：
 * 是不可以共享的外部状态，它以参数的形式注入具体享元的相关方法中。
 */
public class UnsharedConcreteFlyweight {
    private String info;

    public UnsharedConcreteFlyweight(String info) {
        this.info = info;
    }

    public String getInfo() {
        return info;
    }

    public void setInfo(String info) {
        this.info = info;
    }
}


package com.zhyen.base.design_mode.flyweight_mode;

/**
 * 抽象享元角色（Flyweight）:
 * 是所有的具体享元类的基类，为具体享元规范需要实现的公共接口，非享元的外部状态以参数的形式通过方法传入
 */
public interface IFlyweight {

    void operation(UnsharedConcreteFlyweight flyweight);
}


package com.zhyen.base.design_mode.flyweight_mode;

public class ConcreteFlyweightA implements IFlyweight {
    private String key;

    public ConcreteFlyweightA(String key) {
        this.key = key;
        System.out.println("具体享元 A " + key + "被创建！");
    }

    @Override
    public void operation(UnsharedConcreteFlyweight flyweight) {
        System.out.print("具体享元" + key + "被调用，");
        System.out.println("非享元信息是:" + flyweight.getInfo());
    }
}


package com.zhyen.base.design_mode.flyweight_mode;

import java.util.HashMap;

/**
 * 享元工厂（Flyweight Factory）角色：
 * 负责创建和管理享元角色。
 * 当客户对象请求一个享元对象时，享元工厂检査系统中是否存在符合要求的享元对象，
 * 如果存在则提供给客户；如果不存在的话，则创建一个新的享元对象。
 */
public class FlyweightFactory {
    private HashMap<String, IFlyweight> flyweights = new HashMap<String, IFlyweight>();

    public IFlyweight getFlyweight(String key) {
        IFlyweight flyweight = (IFlyweight) flyweights.get(key);
        if (flyweight != null) {
            System.out.println("具体享元" + key + "已经存在，被成功获取！");
        } else {
            flyweight = new ConcreteFlyweightA(key);
            flyweights.put(key, flyweight);
        }
        return flyweight;
    }
}

//客户端使用
    FlyweightFactory factory = new FlyweightFactory();
        IFlyweight f01 = factory.getFlyweight("1");
        IFlyweight f02 = factory.getFlyweight("1");
        IFlyweight f03 = factory.getFlyweight("1");
        IFlyweight f11 = factory.getFlyweight("b");
        IFlyweight f12 = factory.getFlyweight("b");
        f01.operation(new UnsharedConcreteFlyweight("第1次调用a。"));
        f02.operation(new UnsharedConcreteFlyweight("第2次调用a。"));
        f03.operation(new UnsharedConcreteFlyweight("第3次调用a。"));
        f11.operation(new UnsharedConcreteFlyweight("第1次调用b。"));
        f12.operation(new UnsharedConcreteFlyweight("第2次调用b。"));
```

**结果：**

```
具体享元 A1被创建！
具体享元1已经存在，被成功获取！
具体享元1已经存在，被成功获取！
具体享元 Ab被创建！
具体享元b已经存在，被成功获取！
具体享元1被调用，非享元信息是:第1次调用a。
具体享元1被调用，非享元信息是:第2次调用a。
具体享元1被调用，非享元信息是:第3次调用a。
具体享元b被调用，非享元信息是:第1次调用b。
具体享元b被调用，非享元信息是:第2次调用b。
```

