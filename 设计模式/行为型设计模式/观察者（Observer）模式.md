---
title: 观察者（Observer）模式
date: 2020-05-12 19:07:13
tags: 
	- Java
	- 面试
	- 基础
	- 设计模式
categories: 
	- 设计模式
	- 行为型设计模式
---

在现实世界中，许多对象并不是独立存在的，其中一个对象的行为发生改变可能会导致一个或者多个其他对象的行为也发生改变。例如，某种商品的物价上涨时会导致部分商家高兴，而消费者伤心；还有，当我们开车到交叉路口时，遇到红灯会停，遇到绿灯会行。这样的例子还有很多，例如，股票价格与股民、微信公众号与微信用户、气象局的天气预报与听众、小偷与警察等。

**定义：**

观察者（Observer）模式的定义：指多个对象间存在一对多的依赖关系，当一个对象的状态发生改变时，所有依赖于它的对象都得到通知并被自动更新。这种模式有时又称作发布-订阅模式、模型-视图模式。

**类型：**行为类模式

**类图：**

![image.png](https://upload-images.jianshu.io/upload_images/4118241-d5b6f068df5001d2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

**模式的结构：**

- **被观察者：它提供了一个用于保存观察者对象的聚集类和增加、删除观察者对象的方法，以及通知所有观察者的抽象方法。**从类图中可以看到，类中有一个用来存放观察者对象的Vector容器（之所以使用Vector而不使用List，是因为多线程操作时，Vector在是安全的，而List则是不安全的），这个Vector容器是被观察者类的核心，另外还有三个方法：attach方法是向这个容器中添加观察者对象；detach方法是从容器中移除观察者对象；notify方法是依次调用观察者对象的对应方法。这个角色可以是接口，也可以是抽象类或者具体的类，因为很多情况下会与其他的模式混用，所以使用抽象类的情况比较多。
- **观察者：**观察者角色一般是一个接口，它只有一个update方法，在被观察者状态发生变化时，这个方法就会被触发调用。
- **具体的被观察者：**使用这个角色是为了便于扩展，可以在此角色中定义具体的业务逻辑。
- **具体的观察者：**观察者接口的具体实现，在这个角色中，将定义被观察者对象状态发生变化时所要处理的逻辑。

**解决的问题：**

**优点：**

- 目标与观察者之间建立了一套触发机制。

- 降低了目标与观察者之间的耦合关系，两者之间是抽象耦合关系。

**缺点：**

- 目标与观察者之间的依赖关系并没有完全解除，而且有可能出现循环引用。
- 当观察者对象很多时，通知的发布会花费很多时间，影响程序的效率。

**Android中使用**

ListView 是 Android 中很常用的控件，我们在使用 ListView 添加数据后会调用 Adapter 的 notifyDataSetChanged()方法。

监听器、点击事件、滚动事件
rxjava、eventbus、广播

**适用场景：**

- 对象间存在一对多关系，一个对象的状态发生改变会影响其他对象。
- 当对一个对象的改变需要同时改变其它对象, 而不知道具体有多少对象有待改变。

## 案例

抽象被观察者

```
import java.util.ArrayList;
import java.util.List;

/**
 * 抽象被观察者
 */
public abstract class AbstractSubject {
    protected List<IObserver> list = new ArrayList();

    public void attach(IObserver observer) {
        list.add(observer);
    }

    public void detach(IObserver observer) {
        list.remove(observer);
    }

    protected void notifyObserver() {
        if (list.size() == 0) {
            System.out.println("没有更多的观察者了");
        }
        for (IObserver observer : list) {
            observer.update();
        }
    }

    public abstract void doSomething();
}
```

具体被观察者

```
/**
 * 具体被观察者
 */
public class ConcreteSubject extends AbstractSubject {

    @Override
    public void doSomething() {
        System.out.println("处理一些事情 处理完毕 需要告诉 观察者更新数据");
        notifyObserver();
    }
}
```

抽象观察者

```
public interface IObserver {
    void update();
}
```

具体观察者

```
public class ObserverA implements IObserver {
    @Override
    public void update() {
        System.out.println("A 收到了更新请求");
    }
}


/**
 * 具体的观察者
 */
public class ObserverB implements IObserver {
    @Override
    public void update() {
        System.out.println("B 收到了更新请求");
    }
}
```

客户端调用

```
public static void observerMode() {
        ConcreteSubject subject = new ConcreteSubject();
        ObserverA observerA = new ObserverA();
        ObserverB observerB = new ObserverB();
        subject.attach(observerA);
        subject.attach(observerB);
        subject.doSomething();
        System.out.println("-------------");
        subject.detach(observerA);
        subject.detach(observerB);
        subject.doSomething();
    }
```

**结果：**

```
处理一些事情 处理完毕 需要告诉 观察者更新数据
A 收到了更新请求
B 收到了更新请求
-------------
处理一些事情 处理完毕 需要告诉 观察者更新数据
没有更多的观察者了
```

