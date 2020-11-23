---
title: UML中类的关系
date: 2020-05-10 21:15:22
tags: 
	- UML
categories: 
	- UML
---

**类图知识点：**

![image.png](https://upload-images.jianshu.io/upload_images/4118241-3f8577784f7dfdc6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- 类图分为三部分，依次是类名、属性、方法

- 以<<开头和以>>结尾的为注释信息

- 修饰符+代表public，-代表private，#代表protected，什么都没有代表包可见。

- 带下划线的属性或方法代表是静态的。



在java以及其他的面向对象设计模式中，类与类之间主要有6种关系，他们分别是：

- 依赖
- 关联
- 聚合
- 组合
- 继承
- 实现

他们的耦合度依次增强

# 依赖![image.png](https://upload-images.jianshu.io/upload_images/4118241-b8cac23467b8431c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![image.png](https://upload-images.jianshu.io/upload_images/4118241-46ead55a043b1c3f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

依赖关系的定义为：对于两个相对独立的对象，当一个对象负责构造另一个对象的实例，或者依赖另一个对象的服务时，这两个对象之间主要体现为依赖关系。定义比较晦涩难懂，但在java中的表现还是比较直观的：**类A当中使用了类B，其中类B是作为类A的方法参数、方法中的局部变量、或者静态方法调用**。类上面的图例中：People类依赖于Book类和Food类，Book类和Food类是作为类中方法的参数形式出现在People类中的。

```
public class People{
    //Book作为read方法的形参
     public void read(Book book){
        System.out.println(“读的书是”+book.getName());
    }
}
```

# 关联![image.png](https://upload-images.jianshu.io/upload_images/4118241-0df900778ab7f3af.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)![image.png](/Users/fengxing/blogs/source/image/设计模式/类的关系/关联.png)![image.png](https://upload-images.jianshu.io/upload_images/4118241-3304f5652e0e10ba.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- 关联关系中作为成员变量的类一般会在类中赋值

**双向的关联可以有两个箭头或者没有箭头，单向的关联有一个箭头。**

 对于两个相对独立的对象，当一个对象的实例与另一个对象的一些特定实例存在固定的对应关系时，这两个对象之间为关联关系。关联关系分为单向关联和双向关联。在java中，单向关联表现为：类A当中使用了类B，其中类B是作为类A的成员变量。双向关联表现为：类A当中使用了类B作为成员变量；同时类B中也使用了类A作为成员变量。

**单项关联**

![这里写图片描述](https://img-blog.csdn.net/20160507192549756)

```
public class Person{
    private Car car = new Car();
}
```



**双向关联**

![这里写图片描述](https://img-blog.csdn.net/20160507192628366)

```
public class Husband{
    private Wife wife=new Wife();

    public void say(){
        System.out.println("my wife name:"+wife.getName());
    }

}

public class Wife{
    private Husband husband=new Husband();

    public void say(){
        System.out.println("my husband name:"+husband.getName());
    }

}
```

# 聚合![image.png](https://upload-images.jianshu.io/upload_images/4118241-2e286214e4402d08.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- 从`set`方法中传递过来的叫聚合

![image.png](https://upload-images.jianshu.io/upload_images/4118241-16c2707d4666de18.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

聚合关系是关联关系的一种，耦合度强于关联，他们的代码表现是相同的，仅仅是在语义上有所区别：关联关系的对象间是相互独立的，而聚合关系的对象之间存在着包容关系，他们之间是“整体-个体”的相互关系。

```
public class People{
    Car car;
    House house; 
    //聚合关系中作为成员变量的类一般使用set方法赋值
     public void setCar(Car car){
        This.car = car;
    }
    public void setHouse(House house){
        This.house = house;
    }
 
    public void driver(){
        System.out.println(“车的型号：”+car.getType());
    }
    public void sleep(){
        System.out.println(“我在房子里睡觉：”+house.getAddress());
    }
}
```

# 组合![image.png](https://upload-images.jianshu.io/upload_images/4118241-16bf5e2669ef869d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- 在构造方法中实例化其他类叫组合

![image.png](https://upload-images.jianshu.io/upload_images/4118241-a87c3182e0dd1052.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

相比于聚合，组合是一种耦合度更强的关联关系。存在组合关系的类表示“整体-部分”的关联关系，“整体”负责“部分”的生命周期，他们之间是共生共死的；并且“部分”单独存在时没有任何意义。在下图的例子中，People与Soul、Body之间是组合关系，当人的生命周期开始时，必须同时有灵魂和肉体；当人的生命周期结束时，灵魂肉体随之消亡；无论是灵魂还是肉体，都不能单独存在，他们必须作为人的组成部分存在。

```
Public class People{
    Soul soul;
    Body body; 
    //组合关系中的成员变量一般会在构造方法中赋值
     Public People(Soul soul, Body body){ 
        This.soul = soul;
        This.body = body;
    }
 
    Public void study(){
        System.out.println(“学习要用灵魂”+soul.getName());
    }
    Public void eat(){
        System.out.println(“吃饭用身体：”+body.getName());
    }
}
```



# 继承![image.png](https://upload-images.jianshu.io/upload_images/4118241-b67f94bc3650ad16.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![image.png](https://upload-images.jianshu.io/upload_images/4118241-b2e030a7da785f79.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

​    继承表示类与类（或者接口与接口）之间的父子关系。在java中，用关键字extends表示继承关系。UML图例中，继承关系用实线+空心箭头表示，箭头指向父类。

# 实现![image.png](https://upload-images.jianshu.io/upload_images/4118241-6af21432d37bf5da.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![image.png](https://upload-images.jianshu.io/upload_images/4118241-a80ee9fdf19a635d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

​     表示一个类实现一个或多个接口的方法。接口定义好操作的集合，由实现类去完成接口的具体操作。在java中使用implements表示。UML图例中，实现关系用虚线+空心箭头表示，箭头指向接口。