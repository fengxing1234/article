---
title: 中介者（Mediator）模式
date: 2020-05-12 16:52:09
tags: 
	- Java
	- 面试
	- 基础
	- 设计模式
categories: 
	- 设计模式
	- 行为型设计模式
---

在现实生活中，常常会出现好多对象之间存在复杂的交互关系，这种交互关系常常是“网状结构”，它要求每个对象都必须知道它需要交互的对象。例如，每个人必须记住他（她）所有朋友的电话；而且，朋友中如果有人的电话修改了，他（她）必须告诉其他所有的朋友修改，这叫作“牵一发而动全身”，非常复杂。

**定义：**中介者（Mediator）模式：定义一个中介对象来封装一系列对象之间的交互，使原有对象之间的耦合松散，且可以独立地改变它们之间的交互。中介者模式又叫调停模式，它是迪米特法则的典型应用。

**类型：**行为设计模式

**类图：**

![image.png](https://upload-images.jianshu.io/upload_images/4118241-332a4b17960cd27b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![中介者模式的结构图](http://c.biancheng.net/uploads/allimg/181116/3-1Q1161I532V0.gif)

**模式的结构：**

-  抽象中介者：定义好同事类对象到中介者对象的接口，用于各个同事类之间的通信。一般包括一个或几个抽象的事件方法，并由子类去实现。
- 中介者实现类：从抽象中介者继承而来，实现抽象中介者中定义的事件方法。从一个同事类接收消息，然后通过消息影响其他同时类。
- 抽象同事类（Colleague）角色：定义同事类的接口，保存中介者对象，提供同事对象交互的抽象方法，实现所有相互影响的同事类的公共功能。
- 同事类：如果一个对象会影响其他的对象，同时也会被其他对象影响，那么这两个对象称为同事类。在类图中，同事类只有一个，这其实是现实的省略，在实际应用中，同事类一般由多个组成，他们之间相互影响，相互依赖。同事类越多，关系越复杂。并且，同事类也可以表现为继承了同一个抽象类的一组实现组成。在中介者模式中，同事类之间必须通过中介者才能进行消息传递。


**解决的问题：**

一般来说，同事类之间的关系是比较复杂的，多个同事类之间互相关联时，他们之间的关系会呈现为复杂的网状结构，这是一种过度耦合的架构，即不利于类的复用，也不稳定。例如在下图中，有六个同事类对象，假如对象1发生变化，那么将会有4个对象受到影响。如果对象2发生变化，那么将会有5个对象受到影响。也就是说，同事类之间直接关联的设计是不好的。
![image.png](https://upload-images.jianshu.io/upload_images/4118241-1899a94b9afe9deb.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![image.png](https://upload-images.jianshu.io/upload_images/4118241-34a9a234acb32f7d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

​    如果引入中介者模式，那么同事类之间的关系将变为星型结构，从图中可以看到，任何一个类的变动，只会影响的类本身，以及中介者，这样就减小了系统的耦合。一个好的设计，必定不会把所有的对象关系处理逻辑封装在本类中，而是使用一个专门的类来管理那些不属于自己的行为。

![image.png](https://upload-images.jianshu.io/upload_images/4118241-bb8db9fd250f5d4a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



**优点：**

- 适当地使用中介者模式可以避免同事类之间的过度耦合，使得各同事类之间可以相对独立地使用。
- 使用中介者模式可以将对象间一对多的关联转变为一对一的关联，使对象间的关系易于理解和维护。
- 使用中介者模式可以将对象的行为和协作进行抽象，能够比较灵活的处理对象间的相互作用。

**缺点：**

- 当同事类太多时，中介者的职责将很大，它会变得复杂而庞大，以至于系统难以维护。

**Android中**

具体的例子是KeyGuardViewMediator，这个类是负责锁屏控制的一个类。 在我们的开屏和锁屏的时候要注意会需要很多的控制，包括音效和状态栏，通知等，这些都对应AudioManager等具体的系统服务。所以需要一个统一的中介类来协调和控制各个管理服务的状态。

**适用场景：**

- 当对象之间存在复杂的网状结构关系而导致依赖关系混乱且难以复用时。
- 当想创建一个运行于多个类之间的对象，又不想生成新的子类时。

在 MVC 框架中，控制器（C）就是模型（M）和视图（V）的中介者。

在面向对象编程中，一个类必然会与其他的类发生依赖关系，完全独立的类是没有意义的。一个类同时依赖多个类的情况也相当普遍，既然存在这样的情况，说明，一对多的依赖关系有它的合理性，适当的使用中介者模式可以使原本凌乱的对象关系清晰，但是如果滥用，则可能会带来反的效果。一般来说，只有对于那种同事类之间是网状结构的关系，才会考虑使用中介者模式。可以将网状结构变为星状结构，使同事类之间的关系变的清晰一些。

   中介者模式是一种比较常用的模式，也是一种比较容易被滥用的模式。对于大多数的情况，同事类之间的关系不会复杂到混乱不堪的网状结构，因此，大多数情况下，将对象间的依赖关系封装的同事类内部就可以的，没有必要非引入中介者模式。滥用中介者模式，只会让事情变的更复杂。

## 案例

**抽象中介者：**

```
/**
 * 抽象中介者
 */
public abstract class AbstractMediator {
    public abstract void register(AbstractColleague colleague);

    //转发
    public abstract void relay(AbstractColleague colleague);
}

```

**具体中介者：**

```
/**
 * 具体中介者
 */
public class ConcreteMediator extends AbstractMediator {

    private List<AbstractColleague> colleagues = new ArrayList<>();

    @Override
    public void register(AbstractColleague colleague) {
        System.out.println("具体中介者 register");
        if (!colleagues.contains(colleague)) {
            colleagues.add(colleague);
            colleague.setMediator(this);
        }
    }

    //转发
    @Override
    public void relay(AbstractColleague colleague) {
        System.out.println("具体中介者 转发");
        for (AbstractColleague ob : colleagues) {
            if (!ob.equals(colleague)) {
                ob.receive();
            }
        }
    }
}
```

**抽象同事类**

```
/**
 * 抽象同时类
 */
public abstract class AbstractColleague {
    protected AbstractMediator mediator;

    protected void setMediator(AbstractMediator mediator) {
        System.out.println("抽象同时类 setMediator");
        this.mediator = mediator;
    }

    public abstract void receive();

    public abstract void send();

}
```

**具体同事类：**

```
public class ConcreteColleagueA extends AbstractColleague {
    @Override
    public void receive() {
        System.out.println("具体同时类A receive");
    }

    @Override
    public void send() {
        System.out.println("具体同时类A send");
        mediator.relay(this); //请中介者转发
    }
}


/**
 * 具体同时类B
 */
public class ConcreteColleagueB extends AbstractColleague {
    @Override
    public void receive() {
        System.out.println("具体同时类B receive");
    }

    @Override
    public void send() {
        System.out.println("具体同时类B send");
        mediator.relay(this); //请中介者转发

    }
}
```

**客户端使用**

```
public static void mediatorMode() {
        //创建具体中介者
        AbstractMediator concreteMediator = new ConcreteMediator();
        //创建具体同事类A
        ConcreteColleagueA colleagueA = new ConcreteColleagueA();
        //创建具体同事类A
        ConcreteColleagueB colleagueB = new ConcreteColleagueB();
        concreteMediator.register(colleagueA);
        concreteMediator.register(colleagueB);
        System.out.println("--------------------");
        colleagueA.send();
        System.out.println("--------------------");
        colleagueB.send();
    }
```

**结果：**

```
具体中介者 register
抽象同时类 setMediator
具体中介者 register
抽象同时类 setMediator
--------------------
具体同时类A send
具体中介者 转发
具体同时类B receive
--------------------
具体同时类B send
具体中介者 转发
具体同时类A receive
```