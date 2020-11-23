---
title: 策略（Strategy）模式
date: 2020-05-13 17:54:29
tags: 
	- Java
	- 面试
	- 基础
	- 设计模式
categories: 
	- 设计模式
	- 行为型设计模式
---

在现实生活中常常遇到实现某种目标存在多种策略可供选择的情况，例如，出行旅游可以乘坐飞机、乘坐火车、骑自行车或自己开私家车等，超市促销可以釆用打折、送商品、送积分等方法。

在软件开发中也常常遇到类似的情况，当实现某一个功能存在多种算法或者策略，我们可以根据环境或者条件的不同选择不同的算法或者策略来完成该功能，如数据排序策略有冒泡排序、选择排序、插入排序、二叉树排序等。

如果使用多重条件转移语句实现（即硬编码），不但使条件语句变得很复杂，而且增加、删除或更换算法要修改原代码，不易维护，违背开闭原则。如果采用策略模式就能很好解决该问题。

**定义：**定义一组算法，将每个算法都封装起来，并且使他们之间可以互换。

策略（Strategy）模式的定义：该模式定义了一系列算法，并将每个算法封装起来，使它们可以相互替换，且算法的变化不会影响使用算法的客户。策略模式属于对象行为模式，它通过对算法进行封装，把使用算法的责任和算法的实现分割开来，并委派给不同的对象对这些算法进行管理。

**类型：**行为类模式

**类图：**

![策略模式的结构图](http://c.biancheng.net/uploads/allimg/181116/3-1Q116103K1205.gif)

**模式的结构：**

- 抽象策略（Strategy）类：定义了一个公共接口，各种不同的算法以不同的方式实现这个接口，环境角色使用这个接口调用不同的算法，一般使用接口或抽象类实现。
- 具体策略（Concrete Strategy）类：实现了抽象策略定义的接口，提供具体的算法实现。
- 环境（Context）类：持有一个策略类的引用，最终给客户端调用。

**解决的问题：**

**优点：**

- 多重条件语句不易维护，而使用策略模式可以避免使用多重条件语句。
- 策略模式提供了一系列的可供重用的算法族，恰当使用继承可以把算法族的公共代码转移到父类里面，从而避免重复的代码。
- 策略模式可以提供相同行为的不同实现，客户可以根据不同时间或空间要求选择不同的。
- 策略模式提供了对开闭原则的完美支持，可以在不修改原代码的情况下，灵活增加新算法。
- 策略模式把算法的使用放到环境类中，而算法的实现移到具体策略类中，实现了二者的分离。

**缺点：**

- 客户端必须理解所有策略算法的区别，以便适时选择恰当的算法类。
- 策略模式造成很多的策略类。
- 必须对客户端（调用者）暴露所有的策略类，因为使用哪种策略是由客户端来决定的，因此，客户端应该知道有什么策略，并且了解各种策略之间的区别，否则，后果很严重。例如，有一个排序算法的策略模式，提供了快速排序、冒泡排序、选择排序这三种算法，客户端在使用这些算法之前，是不是先要明白这三种算法的适用情况？再比如，客户端要使用一个容器，有链表实现的，也有数组实现的，客户端是不是也要明白链表和数组有什么区别？就这一点来说是有悖于迪米特法则的。


**Android中**

在动画执行过程中，我们需要一些动态效果，有点类似电影的慢镜头，有时需要慢一点，有时需要快一点，这样动画变得灵动起来，这些动态效果就是通过插值器（TimeInterpolator）实现的，我们只需要对Animation对象设置不同的插值器就可以实现不同的动态效果，该效果则使用的就是策略模式实现。

- AccelerateDecelerateInterpolator 在动画开始与介绍的地方速率改变比较慢，在中间的时候加速
- AccelerateInterpolator 在动画开始的地方速率改变比较慢，然后开始加速
- AnticipateInterpolator 开始的时候向后然后向前甩
- AnticipateOvershootInterpolator 开始的时候向后然后向前甩一定值后返回最后的值
- BounceInterpolator 动画结束的时候弹起
- CycleInterpolator 动画循环播放特定的次数，速率改变沿着正弦曲线
- DecelerateInterpolator 在动画开始的地方快然后慢
- LinearInterpolator 以常量速率改变
- OvershootInterpolator 向前甩一定值后再回到原来位置
- PathInterpolator 路径插值器

**适用场景**

做面向对象设计的，对策略模式一定很熟悉，因为它实质上就是面向对象中的继承和多态，在看完策略模式的通用代码后，我想，即使之前从来没有听说过策略模式，在开发过程中也一定使用过它吧？至少在在以下两种情况下，大家可以考虑使用策略模式，

- 几个类的主要逻辑相同，只在部分逻辑的算法和行为上稍有区别的情况。
- 有几种相似的行为，或者说算法，客户端需要动态地决定使用哪一种，那么可以使用策略模式，将这些算法封装起来供客户端调用。

策略模式是一种简单常用的模式，我们在进行开发的时候，会经常有意无意地使用它，一般来说，策略模式不会单独使用，跟模版方法模式、工厂模式等混合使用的情况比较多。 

## 案例

### UML

```
//抽象策略接口
public interface IStrategy {
    void strategyMethod();
}

//具体实现策略类
public class ConcreteStrategyA implements IStrategy {
    @Override
    public void strategyMethod() {
        System.out.println("使用实现类A的方法进行计算");
    }
}

public class ConcreteStrategyB implements IStrategy {
    @Override
    public void strategyMethod() {
        System.out.println("使用实现类B的方法进行计算");
    }
}

//环境类
public class StrategyContext {
    private IStrategy strategy;

    public IStrategy getStrategy() {
        return strategy;
    }

    public void setStrategy(IStrategy strategy) {
        this.strategy = strategy;
    }

    public void strategy(){
        strategy.strategyMethod();
    }
}

//客户端使用
private static void strategyUML() {
        StrategyContext strategy = new StrategyContext();
        strategy.setStrategy(new ConcreteStrategyA());
        strategy.strategy();
        System.out.println("---------");
        strategy.setStrategy(new ConcreteStrategyB());
        strategy.strategy();
    }
```

**结果：**

```
使用实现类A的方法进行计算
---------
使用实现类B的方法进行计算
```

