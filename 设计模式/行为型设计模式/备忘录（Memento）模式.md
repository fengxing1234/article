---
title: 备忘录（Memento）模式
date: 2020-05-14 00:27:28
tags:
	- Java
	- 面试
	- 基础
	- 设计模式
categories: 
	- 设计模式
	- 行为型设计模式
---

每个人都有犯错误的时候，都希望有种“后悔药”能弥补自己的过失，让自己重新开始，但现实是残酷的。在计算机应用中，客户同样会常常犯错误，能否提供“后悔药”给他们呢？当然是可以的，而且是有必要的。这个功能由“备忘录模式”来实现。

其实很多应用软件都提供了这项功能，如 Word、记事本、Photoshop、Eclipse 等软件在编辑时按 Ctrl+Z 组合键时能撤销当前操作，使文档恢复到之前的状态；还有在 IE 中的后退键、数据库事务管理中的回滚操作、玩游戏时的中间结果存档功能、数据库与操作系统的备份操作、棋类游戏中的悔棋功能等都属于这类。

**备忘录模式能记录一个对象的内部状态，当用户后悔时能撤销当前操作，使数据恢复到它原先的状态。**

**定义：**

备忘录（Memento）模式的定义：在不破坏封装性的前提下，捕获一个对象的内部状态，并在该对象之外保存这个状态，以便以后当需要时能将该对象恢复到原先保存的状态。该模式又叫快照模式。

**类型：**行为型设计模式

**类图：**

![备忘录模式的结构图](http://c.biancheng.net/uploads/allimg/181119/3-1Q119130413927.gif)

**模式的结构：**

- 发起人（Originator）角色：记录当前时刻的内部状态信息，提供创建备忘录和恢复备忘录数据的功能，实现其他业务功能，它可以访问备忘录里的所有信息。
- 备忘录（Memento）角色：负责存储发起人的内部状态，在需要的时候提供这些内部状态给发起人。
- 管理者（Caretaker）角色：对备忘录进行管理，提供保存与获取备忘录的功能，但其不能对备忘录的内容进行访问与修改。

**解决的问题：**解决没有后悔药的问题

**优点：**

- 提供了一种可以恢复状态的机制。当用户需要时能够比较方便地将数据恢复到某个历史的状态。
- 实现了内部状态的封装。除了创建它的发起人之外，其他对象都不能够访问这些状态信息。
- 简化了发起人类。发起人不需要管理和保存其内部状态的各个备份，所有状态信息都保存在备忘录中，并由管理者进行管理，这符合单一职责原则。

**缺点：**

- 资源消耗大。如果要保存的内部状态信息过多或者特别频繁，将会占用比较大的内存资源。

**适用场景：**

- 需要保存与恢复数据的场景，如玩游戏时的中间结果的存档功能
- 需要提供一个可回滚操作的场景，如 Word、记事本、Photoshop，Eclipse 等软件在编辑时按 Ctrl+Z 组合键，还有数据库中事务操作。

## 案例

### UML

```
/**
 * 管理者
 */
public class Caretaker {
    private Memento memento;

    public Memento getMemento() {
        return memento;
    }

    public void setMemento(Memento memento) {
        this.memento = memento;
    }
}

/**
 * 备忘录
 */
public class Memento {
    private String state;

    public Memento(String state) {
        this.state = state;
    }

    public String getState() {
        return state;
    }

    public void setState(String state) {
        this.state = state;
    }
}

/**
 * 发起人
 */
public class Originator {
    private String state;

    public Originator() {
    }

    public String getState() {
        return state;
    }

    public void setState(String state) {
        this.state = state;
    }

    public Memento createMemento() {
        return new Memento(state);
    }

    public void restoreMemento(Memento memento) {
        setState(memento.getState());
    }
}

//客户端使用
Originator originator = new Originator();
        Caretaker caretaker = new Caretaker();
        originator.setState("S0");
        System.out.println("初始状态:" + originator.getState());
        caretaker.setMemento(originator.createMemento()); //保存状态
        originator.setState("S1");
        System.out.println("新的状态:" + originator.getState());
        originator.restoreMemento(caretaker.getMemento()); //恢复状态
        System.out.println("恢复状态:" + originator.getState());
        
```

**结果：**

```
初始状态:S0
新的状态:S1
恢复状态:S0
```

### 案例

【例1】利用备忘录模式设计相亲游戏。

分析：假如有西施、王昭君、貂蝉、杨玉环四大美女同你相亲，你可以选择其中一位作为你的爱人；当然，如果你对前面的选择不满意，还可以重新选择，但希望你不要太花心；这个游戏提供后悔功能，用“备忘录模式”设计比较合适。

首先，先设计一个美女（Girl）类，它是备忘录角色，提供了获取和存储美女信息的功能；然后，设计一个相亲者（You）类，它是发起人角色，它记录当 前时刻的内部状态信息（临时妻子的姓名），并提供创建备忘录和恢复备忘录数据的功能；最后，定义一个美女栈（GirlStack）类，它是管理者角色，负责对备忘录进行管理，用于保存相亲者（You）前面选过的美女信息，不过最多只能保存 4 个，提供后悔功能。

![相亲游戏的结构图](http://c.biancheng.net/uploads/allimg/181119/3-1Q119130439230.gif)

**备忘录角色美女类**

```
/**
 * 备忘录
 */
public class Girl {
    //未婚妻
    public String fiancee;

    public Girl(String fiancee) {
        this.fiancee = fiancee;
    }

    public String getFiancee() {
        return fiancee;
    }

    public void setFiancee(String fiancee) {
        this.fiancee = fiancee;
    }
}

```

**发起人 You去相亲**

- create 保存一次状态到管理者
- restore 从管理者中恢复一次状态

```
/**
 * 发起人
 */
public class You {
    //未婚妻
    public String fiancee;

    public You() {
    }

    public String getFiancee() {
        return fiancee;
    }

    public void setFiancee(String fiancee) {
        this.fiancee = fiancee;
    }

    public Girl createGirl() {
        return new Girl(fiancee);
    }

    public void restoreGirl(Girl girl) {
        setFiancee(girl.getFiancee());
    }
}
```

**管理者**

保存状态的地方,push保存一次状态，pop取出一次状态。

```
/**
 * 管理者
 */
public class GirlStack {
    private Girl[] girls = new Girl[5];
    private int top = -1;

    public boolean push(Girl girl) {
        if (top > 4) {
            System.out.println("你太花心了，变来变去的！");
            return false;
        } else {
            girls[++top] = girl;
            return true;
        }
    }

    public Girl pop() {
        if (top <= 0) {
            System.out.println("美女栈空了！");
            return girls[0];
        } else return girls[--top];
    }
}
```

**客户端调用**

```
private static void mementoDemo() {
        You you = new You();
        GirlStack girlStack = new GirlStack();
        you.setFiancee("西施");
        //保存状态
        girlStack.push(you.createGirl());
        System.out.println("当前选择的美女是" + you.getFiancee());

        you.setFiancee("貂蝉");
        girlStack.push(you.createGirl());
        System.out.println("当前选择的美女是" + you.getFiancee());

        you.setFiancee("王昭君");
        girlStack.push(you.createGirl());
        System.out.println("当前选择的美女是" + you.getFiancee());

        you.restoreGirl(girlStack.pop());
        System.out.println("返回了选择上一个美女" + you.getFiancee());

        you.restoreGirl(girlStack.pop());
        System.out.println("返回了选择上一个美女" + you.getFiancee());

        you.restoreGirl(girlStack.pop());
        System.out.println("返回了选择上一个美女" + you.getFiancee());
    }
```

**结果**

```
当前选择的美女是西施
当前选择的美女是貂蝉
当前选择的美女是王昭君
返回了选择上一个美女貂蝉
返回了选择上一个美女西施
美女栈空了！
返回了选择上一个美女西施
```

