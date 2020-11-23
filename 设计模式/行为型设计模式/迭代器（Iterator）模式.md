---
title: 迭代器（Iterator）模式
date: 2020-05-13 21:48:51
tags:tags: 
	- Java
	- 面试
	- 基础
	- 设计模式
categories: 
	- 设计模式
	- 行为型设计模式
---

在现实生活以及程序设计中，经常要访问一个聚合对象中的各个元素，如数据结构中的链表遍历，通常的做法是将链表的创建和遍历都放在同一个类中，但这种方式不利于程序的扩展，如果要更换遍历方法就必须修改程序源代码，这违背了 “开闭原则”。

既然将遍历方法封装在聚合类中不可取，那么聚合类中不提供遍历方法，将遍历方法由用户自己实现是否可行呢？答案是同样不可取，因为这种方式会存在两个缺点：

- 暴露了聚合类的内部表示，使其数据不安全；
- 增加了客户的负担。

“迭代器模式”能较好地克服以上缺点，它在客户访问类与聚合类之间插入一个迭代器，这分离了聚合对象与其遍历行为，对客户也隐藏了其内部细节，且满足“单一职责原则”和“开闭原则”，如 Java中的 Collection、List、Set、Map 等都包含了迭代器。

**定义：**提供一种方法访问一个容器对象中各个元素，而又不暴露该对象的内部细节。

**类型：**行为类模式

**类图：**

![迭代器模式的结构图](http://c.biancheng.net/uploads/allimg/181116/3-1Q1161PU9528.gif)

**模式的结构：**

- 抽象聚合（Aggregate）角色：一般是一个接口，提供一个iterator()方法，例如java中的Collection接口，List接口，Set接口等,定义存储、添加、删除聚合对象以及创建迭代器对象的接口。
- 具体聚合（ConcreteAggregate）角色：实现抽象聚合类，返回一个具体迭代器的实例。就是抽象容器的具体实现类，比如List接口的有序列表实现ArrayList，List接口的链表实现LinkList，Set接口的哈希列表的实现HashSet等。
- 抽象迭代器（Iterator）角色：定义访问和遍历聚合元素的接口，通常包含 hasNext()、first()、next() 等方法。
- 具体迭代器（Concretelterator）角色：实现抽象迭代器接口中所定义的方法，完成对聚合对象的遍历，记录遍历的当前位置。

**解决的问题：**

​    如果要问java中使用最多的一种模式，答案不是单例模式，也不是工厂模式，更不是策略模式，而是迭代器模式，先来看一段代码吧：

```java
public static void print(Collection coll){
	Iterator it = coll.iterator();
	while(it.hasNext()){
		String str = (String)it.next();
		System.out.println(str);
	}
}
```

这个方法的作用是循环打印一个字符串集合，里面就用到了迭代器模式，java语言已经完整地实现了迭代器模式，Iterator翻译成汉语就是迭代器的意思。提到迭代器，首先它是与集合相关的，集合也叫聚集、容器等，我们可以将集合看成是一个可以包容对象的容器，例如List，Set，Map，甚至数组都可以叫做集合，而迭代器的作用就是把容器中的对象一个一个地遍历出来。

**优点：**

- 访问一个聚合对象的内容而无须暴露它的内部表示。
- 遍历任务交由迭代器完成，这简化了聚合类。
- 它支持以不同方式遍历一个聚合，甚至可以自定义迭代器的子类以支持新的遍历。
- 增加新的聚合类和迭代器类都很方便，无须修改原有代码。
- 封装性良好，为遍历不同的聚合结构提供一个统一的接口。

**缺点：**

**适用场景：**

迭代器模式是与集合共生共死的，一般来说，我们只要实现一个集合，就需要同时提供这个集合的迭代器，就像java中的Collection，List、Set、Map等，这些集合都有自己的迭代器。假如我们要实现一个这样的新的容器，当然也需要引入迭代器模式，给我们的容器实现一个迭代器。

​    但是，由于容器与迭代器的关系太密切了，所以大多数语言在实现容器的时候都给提供了迭代器，并且这些语言提供的容器和迭代器在绝大多数情况下就可以满足我们的需要，所以现在需要我们自己去实践迭代器模式的场景还是比较少见的，我们只需要使用语言中已有的容器和迭代器就可以了。

## 案例

### UML

```
/**
 * 抽象迭代器
 */
public interface IIterator {
    Object first();

    Object next();

    boolean hasNext();
}

/**
 * 具体迭代器
 */
public class ConcreteIterator implements IIterator {
    private List<Object> list;
    private int index = -1;

    public ConcreteIterator(List<Object> list) {
        this.list = list;
    }

    @Override
    public Object first() {
        index = 0;
        return list.get(index);
    }

    @Override
    public Object next() {
        if (hasNext()) {
            return list.get(++index);
        }
        return null;
    }

    @Override
    public boolean hasNext() {
        if (index < list.size() - 1) {
            return true;
        }
        return false;
    }
}

/**
 * 抽象聚合
 */
public interface IAggregate {
    void add(Object o);

    void remove(Object o);

    IIterator getIterator();
}

/**
 * 抽象聚合类
 */
public class ConcreteAggregate implements IAggregate {
    private List<Object> list = new ArrayList();

    @Override
    public void add(Object o) {
        list.add(o);
    }

    @Override
    public void remove(Object o) {
        list.remove(o);
    }

    @Override
    public IIterator getIterator() {
        return new ConcreteIterator(list);
    }
}

//客户端
private static void iterator() {
        ConcreteAggregate aggregate = new ConcreteAggregate();
        aggregate.add("1");
        aggregate.add("2");
        aggregate.add("3");
        IIterator iterator = aggregate.getIterator();
        System.out.print("聚合的内容有：");
        while (iterator.hasNext()) {
            Object ob = iterator.next();
            System.out.print(ob.toString() + "\t");
        }
        Object ob = iterator.first();
        System.out.println("\nFirst：" + ob.toString());

    }
```

**结果：**

```
聚合的内容有：1	2	3	
First：1
```

上面的代码中，Aggregate是容器类接口，大家可以想象一下Collection，List，Set等，Aggregate就是他们的简化版，容器类接口中主要有三个方法：添加对象方法add、删除对象方法remove、取得迭代器方法iterator。Iterator是迭代器接口，主要有两个方法：取得迭代对象方法next，判断是否迭代完成方法hasNext，大家可以对比java.util.List和java.util.Iterator两个接口自行思考。

