---
title: 访问者（Visitor）模式
date: 2020-05-12 20:08:21
tags: 
	- Java
	- 面试
	- 基础
	- 设计模式
categories: 
	- 设计模式
	- 行为型设计模式
---

在现实生活中，有些集合对象中存在多种不同的元素，且每种元素也存在多种不同的访问者和处理方式。例如，公园中存在多个景点，也存在多个游客，不同的游客对同一个景点的评价可能不同；医院医生开的处方单中包含多种药元素，査看它的划价员和药房工作人员对它的处理方式也不同，划价员根据处方单上面的药品名和数量进行划价，药房工作人员根据处方单的内容进行抓药。

这样的例子还有很多，例如，电影或电视剧中的人物角色，不同的观众对他们的评价也不同；还有顾客在商场购物时放在“购物车”中的商品，顾客主要关心所选商品的性价比，而收银员关心的是商品的价格和数量。

这些被处理的数据元素相对稳定而访问方式多种多样的数据结构，如果用“访问者模式”来处理比较方便。访问者模式能把处理方法从数据结构中分离出来，并可以根据需要增加新的处理方法，且不用修改原来的程序代码与数据结构，这提高了程序的扩展性和灵活性。

**定义：**

访问者（Visitor）模式的定义：**将作用于某种数据结构中的各元素的操作分离出来封装成独立的类，使其在不改变数据结构的前提下可以添加作用于这些元素的新的操作，为数据结构中的每个元素提供多种访问方式。**它将对数据的操作与数据结构进行分离，是行为类模式中最复杂的一种模式。

**类型：**行为型模式

**类图：**

![访问者（Visitor）模式的结构图](http://c.biancheng.net/uploads/allimg/181119/3-1Q11910135Y25.gif)

**模式的结构：**

**访问者（Visitor）模式实现的关键是如何将作用于元素的操作分离出来封装成独立的类**，其基本结构与实现方法如下。

-  **抽象访问者：**抽象类或者接口，声明访问者可以访问哪些元素，具体到程序中就是visit方法中的参数定义哪些对象是可以被访问的。
- **访问者：**实现抽象访问者所声明的方法，它影响到访问者访问到一个类后该干什么，要做什么事情。
- **抽象元素类：**接口或者抽象类，声明接受哪一类访问者访问，程序上是通过accept方法中的参数来定义的。抽象元素一般有两类方法，一部分是本身的业务逻辑，另外就是允许接收哪类访问者来访问。
- **元素类：**实现抽象元素类所声明的accept方法，通常都是visitor.visit(this)，基本上已经形成一种定式了。
- **结构对象：**一个元素的容器，一般包含一个容纳多个不同类、不同接口的容器，如List、Set、Map等，在项目中一般很少抽象出这个角色。

**解决的问题：**

**优点：**

- 扩展性好。能够在不修改对象结构中的元素的情况下，为对象结构中的元素添加新的功能。
- 复用性好。可以通过访问者来定义整个对象结构通用的功能，从而提高系统的复用程度。
- 灵活性好。访问者模式将数据结构与作用于结构上的操作解耦，使得操作集合可相对自由地演化而不影响系统的数据结构。
- **符合单一职责原则：**凡是适用访问者模式的场景中，元素类中需要封装在访问者中的操作必定是与元素类本身关系不大且是易变的操作，使用访问者模式一方面符合单一职责原则，另一方面，因为被封装的操作通常来说都是易变的，所以当发生变化时，就可以在不改变元素类本身的前提下，实现对变化部分的扩展。

**缺点：**

- 增加新的元素类很困难。在访问者模式中，每增加一个新的元素类，都要在每一个具体访问者类中增加相应的具体操作，这违背了“开闭原则”。
- 破坏封装。访问者模式中具体元素对访问者公布细节，这破坏了对象的封装性。
- 违反了依赖倒置原则。访问者模式依赖了具体类，而没有依赖抽象类。

**适用场景：**

- 对象结构相对稳定，但其操作算法经常变化的程序。

- 假如一个对象中存在着一些与本对象不相干（或者关系较弱）的操作，为了避免这些操作污染这个对象，则可以使用访问者模式来把这些操作封装到访问者中去。
- 假如一组对象中，存在着相似的操作，为了避免出现大量重复的代码，也可以将这些重复的操作封装到访问者中去。

## 案例

**抽象元素类：**

```
/**
 * 抽象元素类
 */
public interface IElement {
    void accept(IVisitor visitor);
}
```



**具体元素类：**

```
public class ConcreteElementA implements IElement {

    @Override
    public void accept(IVisitor visitor) {
        visitor.visitor(this);
    }

    public String operation() {
        return " 访问元素 A";
    }
}
public class ConcreteElementB implements IElement {
    @Override
    public void accept(IVisitor visitor) {
        visitor.visitor(this);
    }

    public String operation() {
        return " 访问元素 B";
    }
}
```

**抽象访问者：**

```
/**
 * 抽象访问者
 */
public interface IVisitor {
    void visitor(ConcreteElementA concreteElementA);

    void visitor(ConcreteElementB concreteElementB);
}
```

**具体访问者：**

```
/**
 * 具体访问者A
 */
public class ConcreteVisitorA implements IVisitor {
    @Override
    public void visitor(ConcreteElementA concreteElementA) {
        String operation = concreteElementA.operation();
        System.out.println("具体访问者 A" + operation);
    }

    @Override
    public void visitor(ConcreteElementB concreteElementB) {
        String operation = concreteElementB.operation();
        System.out.println("具体访问者 A" + operation);
    }
}

public class ConcreteVisitorB implements IVisitor {
    @Override
    public void visitor(ConcreteElementA concreteElementA) {
        String operation = concreteElementA.operation();
        System.out.println("具体访问者 B" + operation);
    }

    @Override
    public void visitor(ConcreteElementB concreteElementB) {
        String operation = concreteElementB.operation();
        System.out.println("具体访问者 B" + operation);
    }
}
```

**结构对象：**

```
/**
 * 对象结构类
 */
public class ObjectStructure {
    private List<IElement> list = new ArrayList<>();

    public void accept(IVisitor visitor) {
        Iterator<IElement> iterator = list.iterator();
        while (iterator.hasNext()) {
            IElement next = iterator.next();
            next.accept(visitor);
        }
    }

    public void add(IElement element) {
        list.add(element);
    }

    public void remote(IElement element) {
        list.remove(element);
    }

}
```

**客户端调用：**

```
ObjectStructure structure = new ObjectStructure();
        structure.add(new ConcreteElementA());
        structure.add(new ConcreteElementB());
        IVisitor concreteVisitorA = new ConcreteVisitorA();
        structure.accept(concreteVisitorA);
        System.out.println("------------------------");
        ConcreteVisitorB visitorB = new ConcreteVisitorB();
        structure.accept(visitorB);
```

**结果：**

```
具体访问者 A 访问元素 A
具体访问者 A 访问元素 B
------------------------
具体访问者 B 访问元素 A
具体访问者 B 访问元素 B
```

