---
title: 适配器模式（Adapter）
date: 2020-05-14 20:17:58
tags: 
	- Java
	- 面试
	- 基础
	- 设计模式
categories: 
	- 设计模式
	- 结构型设计模式
---

在现实生活中，经常出现两个对象因接口不兼容而不能在一起工作的实例，这时需要第三者进行适配。例如，讲中文的人同讲英文的人对话时需要一个翻译，用直流电的笔记本电脑接交流电源时需要一个电源适配器，用计算机访问照相机的 SD 内存卡时需要一个读卡器等。

在软件设计中也可能出现：需要开发的具有某种业务功能的组件在现有的组件库中已经存在，但它们与当前系统的接口规范不兼容，如果重新开发这些组件成本又很高，这时用适配器模式能很好地解决这些问题。

**定义：**将一个类的接口转换成客户希望的另外一个接口，使得原本由于接口不兼容而不能一起工作的那些类能一起工作。

适配器模式分为类结构型模式和对象结构型模式两种，前者类之间的耦合度比后者高，且要求程序员了解现有组件库中的相关组件的内部结构，所以应用相对较少些。

**类图：**

- 类适配器模式的结构图

![类适配器模式的结构图](http://c.biancheng.net/uploads/allimg/181115/3-1Q1151045351c.gif)

- 对象适配器模式的结构图

![对象适配器模式的结构图](http://c.biancheng.net/uploads/allimg/181115/3-1Q1151046105A.gif)

**模式的结构：**

- 目标（Target）接口：当前系统业务所期待的接口，它可以是抽象类或接口。
- 适配者（Adaptee）类：它是被访问和适配的现存组件库中的组件接口。
- 适配器（Adapter）类：它是一个转换器，通过继承或引用适配者的对象，把适配者接口转换成目标接口，让客户按目标接口的格式访问适配者。

**解决的问题：**

**优点：**

- 客户端通过适配器可以透明地调用目标接口。
- 复用了现存的类，程序员不需要修改原有代码而重用现有的适配者类。
- 将目标类和适配者类解耦，解决了目标类和适配者类接口不一致的问题。

**缺点：**

- 对类适配器来说，更换适配器的实现过程比较复杂。

**适用场景：**

- 以前开发的系统存在满足新系统功能需求的类，但其接口同新系统的接口不一致。
- 使用第三方提供的组件，但组件接口定义和自己要求的接口定义不同。

## 案例

### UML

```
//目标接口
public interface ITarget {
    void request();
}
//适配者
public class Adapter {
    public void specificRequest() {
        System.out.println("适配者中的业务代码被调用！");
    }
}
//适配器
public class ClassAdapter extends Adapter implements ITarget {
    @Override
    public void request() {
        specificRequest();
    }
}
//客户端
private static void adapterMode() {
        ClassAdapter adapter = new ClassAdapter();
        adapter.request();
    }

```

**结果：**

```
适配者中的业务代码被调用!
```

### 用适配器模式（Adapter）模拟新能源汽车的发动机。

分析：新能源汽车的发动机有电能发动机（Electric Motor）和光能发动机（Optical Motor）等，各种发动机的驱动方法不同，例如，电能发动机的驱动方法 electricDrive() 是用电能驱动，而光能发动机的驱动方法 opticalDrive() 是用光能驱动，它们是适配器模式中被访问的适配者。

客户端希望用统一的发动机驱动方法 drive() 访问这两种发动机，所以必须定义一个统一的目标接口 Motor，然后再定义电能适配器（Electric Adapter）和光能适配器（Optical Adapter）去适配这两种发动机。

![发动机适配器的结构图](http://c.biancheng.net/uploads/allimg/181115/3-1Q115104I22F.gif)

```
/**
 * 目标：发动机
 */
public interface Motor {
    void drive();
}

//适配者1：电能发动机
public class ElectricMotor {
    public void electricDrive() {
        System.out.println("电能发动机驱动汽车！");
    }
}

//适配者2：光能发动机
public class OpticalMotor {

    public void opticalMotor() {
        System.out.println("光能发动机驱动汽车！");
    }
}

//电能适配器
public class ElectricAdapter implements Motor {
    private ElectricMotor electricMotor;

    public ElectricAdapter() {
        electricMotor = new ElectricMotor();
    }

    @Override
    public void drive() {
        electricMotor.electricDrive();
    }
}


//光能适配器
public class OpticalAdapter implements Motor {
    private OpticalMotor opticalMotor;

    public OpticalAdapter() {
        opticalMotor = new OpticalMotor();
    }

    @Override
    public void drive() {
        opticalMotor.opticalMotor();
    }
}

//客户端
        ElectricAdapter adapter = new ElectricAdapter();
        adapter.drive();
        OpticalAdapter opticalAdapter = new OpticalAdapter();
        opticalAdapter.drive();
```

```
电能发动机驱动汽车！
光能发动机驱动汽车！
```

