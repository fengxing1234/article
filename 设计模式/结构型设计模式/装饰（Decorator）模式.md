---
title: 装饰（Decorator）模式
date: 2020-05-14 22:43:15
tags:
	- Java
	- 面试
	- 基础
	- 设计模式
categories: 
	- 设计模式
	- 结构型设计模式
---

在现实生活中，常常需要对现有产品增加新的功能或美化其外观，如房子装修、相片加相框等。在软件开发过程中，有时想用一些现存的组件。这些组件可能只是完成了一些核心功能。但在不改变其结构的情况下，可以动态地扩展其功能。所有这些都可以釆用装饰模式来实现。

**定义：**

装饰（Decorator）模式的定义：指在不改变现有对象结构的情况下，动态地给该对象增加一些职责（即增加其额外功能）的模式，它属于对象结构型模式。

**类图：**

![装饰模式的结构图](http://c.biancheng.net/uploads/allimg/181115/3-1Q115142115M2.gif)

**模式的结构：**

- 抽象构件（Component）角色：定义一个抽象接口以规范准备接收附加责任的对象。
- 具体构件（Concrete  Component）角色：实现抽象构件，通过装饰角色为其添加一些职责。
- 抽象装饰（Decorator）角色：继承抽象构件，并包含具体构件的实例，可以通过其子类扩展具体构件的功能。
- 具体装饰（ConcreteDecorator）角色：实现抽象装饰的相关方法，并给具体构件对象添加附加的责任。

**Android中的使用：**

Java I/O 标准库的设计了。例如，InputStream 的子类 FilterInputStream，OutputStream 的子类 FilterOutputStream，Reader 的子类 BufferedReader 以及 FilterReader，还有 Writer 的子类 BufferedWriter、FilterWriter 以及 PrintWriter 等，它们都是抽象装饰类。

**解决的问题：**

**优点：**

- 采用装饰模式扩展对象的功能比采用继承方式更加灵活。
- 可以设计出多个不同的具体装饰类，创造出多个不同行为的组合。

**缺点：**

- 装饰模式增加了许多子类，如果过度使用会使程序变得很复杂。

**适用场景：**

- 当需要给一个现有类添加附加职责，而又不能采用生成子类的方法进行扩充时。例如，该类被隐藏或者该类是终极类或者采用继承方式会产生大量的子类。
- 当需要通过对现有的一组基本功能进行排列组合而产生非常多的功能时，采用继承关系很难实现，而采用装饰模式却很好实现。
- 当对象的功能要求可以动态地添加，也可以再动态地撤销时。

## 案例

```
package com.zhyen.base.design_mode.decorator_mode;

public interface IComponent {
    public void operation();
}



package com.zhyen.base.design_mode.decorator_mode;

public class ConcreteComponent implements IComponent {

    public ConcreteComponent() {
        System.out.println("创建具体构件角色");
    }

    @Override
    public void operation() {
        System.out.println("调用具体构件角色的方法operation()");
    }
}



package com.zhyen.base.design_mode.decorator_mode;

public class Decorator implements IComponent {
    private IComponent component;

    public Decorator(IComponent component) {
        this.component = component;
    }

    @Override
    public void operation() {
        component.operation();
    }
}



package com.zhyen.base.design_mode.decorator_mode;

public class ConcreteDecorator extends Decorator {

    public ConcreteDecorator(IComponent component) {
        super(component);
    }

    public void operation() {
        super.operation();
        addedFunction();
    }

    public void addedFunction() {
        System.out.println("为具体构件角色增加额外的功能addedFunction()");
    }
}
//客户端调用
 IComponent p = new ConcreteComponent();
        p.operation();
        System.out.println("---------------------------------");
        IComponent d = new ConcreteDecorator(p);
        d.operation();
```

```
创建具体构件角色
调用具体构件角色的方法operation()
---------------------------------
调用具体构件角色的方法operation()
为具体构件角色增加额外的功能addedFunction()
```

