---
title: 桥接（Bridge）模式
date: 2020-05-14 21:43:17
tags: 
	- Java
	- 面试
	- 基础
	- 设计模式
categories: 
	- 设计模式
	- 结构型设计模式
---

在现实生活中，某些类具有两个或多个维度的变化，如图形既可按形状分，又可按颜色分。如何设计类似于 Photoshop 这样的软件，能画不同形状和不同颜色的图形呢？如果用继承方式，m 种形状和 n 种颜色的图形就有 m×n 种，不但对应的子类很多，而且扩展困难。

当然，这样的例子还有很多，如不同颜色和字体的文字、不同品牌和功率的汽车、不同性别和职业的男女、支持不同平台和不同文件格式的媒体播放器等。如果用桥接模式就能很好地解决这些问题。

**定义：**

桥接（Bridge）模式的定义如下：将抽象与实现分离，使它们可以独立变化。它是用组合关系代替继承关系来实现，从而降低了抽象和实现这两个可变维度的耦合度。

**类图：**

![桥接模式的结构图](http://c.biancheng.net/uploads/allimg/181115/3-1Q115125253H1.gif)

**模式的结构：**

- 抽象化（Abstraction）角色：定义抽象类，并包含一个对实现化对象的引用。
- 扩展抽象化（Refined  Abstraction）角色：是抽象化角色的子类，实现父类中的业务方法，并通过组合关系调用实现化角色中的业务方法。
- 实现化（Implementor）角色：定义实现化角色的接口，供扩展抽象化角色调用。
- 具体实现化（Concrete Implementor）角色：给出实现化角色接口的具体实现。

**解决的问题：**

**优点：**

- 由于抽象与实现分离，所以扩展能力强；
- 其实现细节对客户透明。

**缺点：**

- 由于聚合关系建立在抽象层，要求开发者针对抽象化进行设计与编程，这增加了系统的理解与设计难度。

**适用场景：**

- 当一个类存在两个独立变化的维度，且这两个维度都需要进行扩展时。
- 当一个系统不希望使用继承或因为多层次继承导致系统类的个数急剧增加时。
- 当一个系统需要在构件的抽象化角色和具体化角色之间增加更多的灵活性时。

## 案例

【例1】用桥接（Bridge）模式模拟女士皮包的选购。

分析：女士皮包有很多种，可以按用途分、按皮质分、按品牌分、按颜色分、按大小分等，存在多个维度的变化，所以采用桥接模式来实现女士皮包的选购比较合适。

本实例按用途分可选钱包（Wallet）和挎包（HandBag），按颜色分可选黄色（Yellow）和红色（Red）。可以按两个维度定义为颜色类和包类。

颜色类（Color）是一个维度，定义为实现化角色，它有两个具体实现化角色：黄色和红色，通过 getColor() 方法可以选择颜色；包类（Bag）是另一个维度，定义为抽象化角色，它有两个扩展抽象化角色：挎包和钱包，它包含了颜色类对象，通过 getName() 方法可以选择相关颜色的挎包和钱包。

![女士皮包选购的结构图](http://c.biancheng.net/uploads/allimg/181115/3-1Q11512532X54.gif)

```
public interface IColor {
    String getColor();
}

public abstract class Bag {
    protected IColor color;

    public void setColor(IColor color) {
        this.color = color;
    }

    public abstract String getName();
}

public class Wallet extends Bag {
    @Override
    public String getName() {
        return color.getColor() + "钱包";
    }
}

package com.zhyen.base.design_mode.bridge_mode;

public class Red implements IColor {
    @Override
    public String getColor() {
        return "红色";
    }
}
package com.zhyen.base.design_mode.bridge_mode;

public class Yellow implements IColor {
    @Override
    public String getColor() {
        return "黄色";
    }
}

package com.zhyen.base.design_mode.bridge_mode;

public class HandBag extends Bag {
    @Override
    public String getName() {
        return color.getColor() + "挎包";
    }
}

//客户端
Wallet wallet = new Wallet();
        wallet.setColor(new Red());
        System.out.println(wallet.getName());
        HandBag handBag = new HandBag();
        handBag.setColor(new Yellow());
        handBag.getName();
        System.out.println(handBag.getName());
```

**结果：**

```
红色钱包
黄色挎包
```

