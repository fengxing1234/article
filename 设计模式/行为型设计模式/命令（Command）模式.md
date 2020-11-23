---
title: 命令（Command）模式
date: 2020-05-12 23:36:32
tags: 
	- Java
	- 面试
	- 基础
	- 设计模式
categories: 
	- 设计模式
	- 行为型设计模式
---

在软件开发系统中，常常出现“方法的请求者”与“方法的实现者”之间存在紧密的耦合关系。这不利于软件功能的扩展与维护。例如，想对行为进行“撤销、重做、记录”等处理都很不方便，因此“如何将方法的请求者与方法的实现者解耦？”变得很重要，命令模式能很好地解决这个问题。

在现实生活中，这样的例子也很多，例如，电视机遥控器（命令发送者）通过按钮（具体命令）来遥控电视机（命令接收者），还有计算机键盘上的“功能键”等。

**定义：**

命令（Command）模式的定义如下：将一个请求封装为一个对象，使发出请求的责任和执行请求的责任分割开。这样两者之间通过命令对象进行沟通，这样方便将命令对象进行储存、传递、调用、增加与管理。

**类型：**行为类模式

**类图：**

![命令模式的结构图](http://c.biancheng.net/uploads/allimg/181116/3-1Q11611335E44.gif)

**模式的结构：**

- 抽象命令类（Command）角色：声明执行命令的接口，拥有执行命令的抽象方法 execute()。
- 具体命令角色（Concrete  Command）角色：是抽象命令类的具体实现类，它拥有接收者对象，并通过调用接收者的功能来完成命令要执行的操作。
- 实现者/接收者（Receiver）角色：执行命令功能的相关操作，是具体命令对象业务的真正实现者。
- 调用者/请求者（Invoker）角色：是请求的发送者，它通常拥有很多的命令对象，并通过访问命令对象来执行相关请求，它不直接访问接收者。

所谓对命令的封装，说白了，无非就是把一系列的操作写到一个方法中，然后供客户端调用就行了，反映到类图上，只需要一个ConcreteCommand类和Client类就可以完成对命令的封装，即使再进一步，为了增加灵活性，可以再增加一个Command类进行适当地抽象，这个调用者和接收者到底是什么作用呢？其实大家可以换一个角度去想：假如仅仅是简单地把一些操作封装起来作为一条命令供别人调用，怎么能称为一种模式呢？命令模式作为一种行为类模式，首先要做到低耦合，耦合度低了才能提高灵活性，而加入调用者和接收者两个角色的目的也正是为此。

**Android中**：

**PackageManagerService**是Android系统的Service之一，主要功能是实现对应用包的解析、管理、卸载等操作。我们来看下具体的结构。

![image.png](https://upload-images.jianshu.io/upload_images/4118241-5207a689f3ee1d3b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

**HandlerParams是命令接口，即我们的Command角色。**

**具体的包的安装、移动以及包大小的测量分别在3个具体子类InstallParams、MoveParams和MeasureParams中实现。**

**而PackageHandler是Handler的子类，用来负责包相关消息的处理，不同的请求对应不同的命令对象，然后通过命令对象来执行具体操作。**

**解决的问题：**

**优点：**

- 降低系统的耦合度。命令模式能将调用操作的对象与实现该操作的对象解耦。
- 增加或删除命令非常方便。采用命令模式增加与删除命令不会影响其他类，它满足“开闭原则”，对扩展比较灵活。
- 可以实现宏命令。命令模式可以与组合模式结合，将多个命令装配成一个组合命令，即宏命令。
- 方便实现 Undo 和 Redo 操作。命令模式可以与备忘录模式结合，实现命令的撤销与恢复。

**缺点：**

- 可能产生大量具体命令类。因为计对每一个具体操作都需要设计一个具体命令类，这将增加系统的复杂性。

**适用场景：**

​    对于大多数请求-响应模式的功能，比较适合使用命令模式，正如命令模式定义说的那样，命令模式对实现记录日志、撤销操作等功能比较方便。

- 当系统需要将请求调用者与请求接收者解耦时，命令模式使得调用者和接收者不直接交互。
- 当系统需要随机请求命令或经常增加或删除命令时，命令模式比较方便实现这些功能。
- 当系统需要执行一组操作时，命令模式可以定义宏命令来实现该功能。
- 当系统需要支持命令的撤销（Undo）操作和恢复（Redo）操作时，可以将命令对象存储起来，采用备忘录模式来实现。

对于一个场合到底用不用模式，这对所有的开发人员来说都是一个很纠结的问题。有时候，因为预见到需求上会发生的某些变化，为了系统的灵活性和可扩展性而使用了某种设计模式，但这个预见的需求偏偏没有，相反，没预见到的需求倒是来了不少，导致在修改代码的时候，使用的设计模式反而起了相反的作用，以至于整个项目组怨声载道。这样的例子，我相信每个程序设计者都遇到过。所以，基于敏捷开发的原则，我们在设计程序的时候，如果按照目前的需求，不使用某种模式也能很好地解决，那么我们就不要引入它，因为要引入一种设计模式并不困难，我们大可以在真正需要用到的时候再对系统进行一下，引入这个设计模式。

拿命令模式来说吧，我们开发中，请求-响应模式的功能非常常见，一般来说，我们会把对请求的响应操作封装到一个方法中，这个封装的方法可以称之为命令，但不是命令模式。到底要不要把这种设计上升到模式的高度就要另行考虑了，因为，如果使用命令模式，就要引入调用者、接收者两个角色，原本放在一处的逻辑分散到了三个类中，设计时，必须考虑这样的代价是否值得。

## 案例

### UML实现

**抽象命令：**

```
public interface ICommand {
    void execute();
}
```

**具体命令：**

```
public class ConcreteCommandA implements ICommand {
    private ReceiverA receiverA = new ReceiverA();

    @Override
    public void execute() {
        System.out.println("具体命名A execute ...");
        receiverA.action();
    }
}

public class ConcreteCommandB implements ICommand {
    private ReceiverB receiverB = new ReceiverB();

    @Override
    public void execute() {
        System.out.println("具体命名B execute ...");
        receiverB.action();
    }
}
```

**接受者A**

```
public class ReceiverA {

    public void action() {
        System.out.println("接受者Receiver A action");
    }
}
```

**接收者B**

```
public class ReceiverB {

    public void action() {
        System.out.println("Receiver B action");
    }
}
```

**执行者**

```
public class Invoke {

    private ICommand command;

    public Invoke(ICommand command) {
        this.command = command;
    }

    public void setCommand(ICommand command) {
        this.command = command;
    }

    public void cell() {
        System.out.println("调用者执行命令cell...");
        command.execute();
    }
}
```

**客户端**

```
private static void commandMode() {
        ConcreteCommandA commandA = new ConcreteCommandA();
        Invoke invoke = new Invoke(commandA);
        invoke.cell();
        System.out.println("--------------");
        ConcreteCommandB commandB = new ConcreteCommandB();
        invoke.setCommand(commandB);
        invoke.cell();
    }
```

**结果：**

```
调用者执行命令cell...
具体命名A execute ...
接受者Receiver A action
--------------
调用者执行命令cell...
具体命名B execute ...
Receiver B action
```

### 用命令模式实现客户去餐馆吃早餐的实例。

分析：客户去餐馆可选择的早餐有肠粉、河粉和馄饨等，客户可向服务员选择以上早餐中的若干种，服务员将客户的请求交给相关的厨师去做。这里的**点早餐相当于“命令”**，**服务员相当于“调用者”**，**厨师相当于“接收者”，**所以用命令模式实现比较合适。

首先，定义一个早餐类（Breakfast），它是抽象命令类，有抽象方法 cooking()，说明要做什么；再定义其子类肠粉类（ChangFen）、馄饨类（HunTun）和河粉类（HeFen），它们是具体命令类，实现早餐类的 cooking() 方法，但它们不会具体做，而是交给具体的厨师去做；具体厨师类有肠粉厨师（ChangFenChef）、馄蚀厨师（HunTunChef）和河粉厨师（HeFenChef），他们是命令的接收者，最后，定义服务员类（Waiter），它接收客户的做菜请求，并发出做菜的命令。客户类是通过服务员类来点菜的。

![客户在餐馆吃早餐的结构图](http://c.biancheng.net/uploads/allimg/181116/3-1Q1161134341E.gif)

**定义接收者厨师接口，同一接到命令后做什么事：**

```
/**
 * 厨师接口
 * 接收者通用接口：表示接受到命令做什么操作
 */
public interface IChef {
    void cooking();
}
```

**具体接收者 河粉厨师，馄饨厨师，肠粉厨师：**

```
/**
 * 接收者：肠粉厨师
 */
public class ChangFenChef implements IChef {
    @Override
    public void cooking() {
        System.out.println("我是肠粉厨师 接到命令--- 我给你做肠粉");
    }
}

/**
 * 接收者：馄饨厨师
 */
public class HunTunChef implements IChef {
    @Override
    public void cooking() {
        System.out.println("我是馄饨厨师 接收到命令--- 我给你做馄饨");
    }
}

/**
 * 接收者：河粉厨师
 */
public class HeFenChef implements IChef {
    @Override
    public void cooking() {
        System.out.println("我是河粉厨师 接到命令--- 我给你做河粉");
    }
}

```

**定义抽象命令：点餐 **

```
/**
 * 点餐命令抽象类
 */
public interface IBreakfastCommand {
    void cooking();
}

```

**具体命令：点了什么东西**

```
/**
 * 河粉的具体命令
 */
public class HeFenCommand implements IBreakfastCommand {
    private HeFenChef heFenChef = new HeFenChef();

    @Override
    public void cooking() {
        System.out.println("命令河粉厨师 做河粉 ");
        heFenChef.cooking();
    }
}

/**
 * 肠粉的具体命令
 */
public class ChangHenCommand implements IBreakfastCommand {
    private ChangFenChef changFenChef = new ChangFenChef();

    @Override
    public void cooking() {
        System.out.println("命令肠粉厨师 做肠粉 ");
        changFenChef.cooking();
    }
}

/**
 * 馄饨的具体命令
 */
public class HunTunCommand implements IBreakfastCommand {
    private HunTunChef hunTunChef = new HunTunChef();

    @Override
    public void cooking() {
        System.out.println("命令馄饨厨师 做馄饨 ");
        hunTunChef.cooking();
    }
}

```

**调用者：服务员 **

```
/**
 * 服务员调用者
 */
public class Waiter {
    private IBreakfastCommand heFenCommand, changFenCommand, hunTunCommand;

    public Waiter() {

    }

    public void setHeFenCommand(IBreakfastCommand heFenCommand) {
        this.heFenCommand = heFenCommand;
    }

    public void setChangFenCommand(IBreakfastCommand changFenCommand) {
        this.changFenCommand = changFenCommand;
    }

    public void setHunTunCommand(IBreakfastCommand hunTunCommand) {
        this.hunTunCommand = hunTunCommand;
    }

    public void chooseHeFen() {
        System.out.println("服务员：这小子点了一份河粉 快点上菜");
        heFenCommand.cooking();
    }

    public void chooseChangFen() {
        System.out.println("服务员：这小子点了一份肠粉 快点上菜");
        changFenCommand.cooking();
    }

    public void chooseHunTun() {
        System.out.println("服务员：这小子点了一份馄饨 快点上菜");
        hunTunCommand.cooking();
    }
}

```

**客户端：我吃早餐**

```
private static void commandDemo() {
        System.out.println("过来服务员");
        Waiter waiter = new Waiter();
        System.out.println("给我来份肠粉");
        waiter.setChangFenCommand(new ChangHenCommand());
        waiter.chooseChangFen();
        System.out.println("给我来份河粉");
        waiter.setHeFenCommand(new HeFenCommand());
        waiter.chooseHeFen();
        System.out.println("听说你家馄饨不错 来份尝尝");
        waiter.setHunTunCommand(new HunTunCommand());
        waiter.chooseHunTun();

    }
```

**结果**

```
过来服务员
给我来份肠粉
服务员：这小子点了一份肠粉 快点上菜
命令肠粉厨师 做肠粉 
我是肠粉厨师 接到命令--- 我给你做肠粉
给我来份河粉
服务员：这小子点了一份河粉 快点上菜
命令河粉厨师 做河粉 
我是河粉厨师 接到命令--- 我给你做河粉
听说你家馄饨不错 来份尝尝
服务员：这小子点了一份馄饨 快点上菜
命令馄饨厨师 做馄饨 
我是馄饨厨师 接收到命令--- 我给你做馄饨
```

