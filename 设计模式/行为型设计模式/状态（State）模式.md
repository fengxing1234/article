---
title: 状态（State）模式
date: 2020-05-14 12:55:55
tags: 
	- Java
	- 面试
	- 基础
	- 设计模式
categories: 
	- 设计模式
	- 行为型设计模式
---

在软件开发过程中，应用程序中的有些对象可能会根据不同的情况做出不同的行为，我们把这种对象称为有状态的对象，而把影响对象行为的一个或多个动态变化的属性称为状态。当有状态的对象与外部事件产生互动时，其内部状态会发生改变，从而使得其行为也随之发生改变。如人的情绪有高兴的时候和伤心的时候，不同的情绪有不同的行为，当然外界也会影响其情绪变化。

对这种有状态的对象编程，传统的解决方案是：将这些所有可能发生的情况全都考虑到，然后使用 if-else 语句来做状态判断，再进行不同情况的处理。但当对象的状态很多时，程序会变得很复杂。而且增加新的状态要添加新的 if-else 语句，这违背了“开闭原则”，不利于程序的扩展。

以上问题如果采用“状态模式”就能很好地得到解决。状态模式的解决思想是：当控制一个对象状态转换的条件表达式过于复杂时，把相关“判断逻辑”提取出来，放到一系列的状态类当中，这样可以把原来复杂的逻辑判断简单化。

**定义：**

状态（State）模式的定义：对有状态的对象，把复杂的“判断逻辑”提取到不同的状态对象中，允许状态对象在其内部状态发生改变时改变其行为。

**类型：**行为型设计模式

**类图：**

![状态模式的结构图](http://c.biancheng.net/uploads/allimg/181116/3-1Q11615412U55.gif)

**模式的结构：**

**解决的问题：**

**优点：**

- 状态模式将与特定状态相关的行为局部化到一个状态中，并且将不同状态的行为分割开来，满足“单一职责原则”。
- 减少对象间的相互依赖。将不同的状态引入独立的对象中会使得状态转换变得更加明确，且减少对象间的相互依赖。
- 有利于程序的扩展。通过定义新的子类很容易地增加新的状态和转换。

**缺点：**

- 状态模式的使用必然会增加系统的类与对象的个数。
- 状态模式的结构与实现都较为复杂，如果使用不当会导致程序结构和代码的混乱。

**适用场景：**

## 案例

### UML

```
public abstract class AbstractState {
    public abstract void handle(StateContext context);
}

public class ConcreteStateA extends AbstractState {
    @Override
    public void handle(StateContext context) {
        System.out.println("当前状态是 A.");
        context.setState(new ConcreteStateB());
    }
}

public class ConcreteStateB extends AbstractState {
    @Override
    public void handle(StateContext context) {
        System.out.println("当前状态是 B.");
        context.setState(new ConcreteStateA());
    }
}

public class StateContext {

    private AbstractState state;

    public StateContext() {
        this.state = new ConcreteStateA();
    }

    public AbstractState getState() {
        return state;
    }

    public void setState(AbstractState state) {
        this.state = state;
    }

    public void handle() {
        state.handle(this);
    }
}

//客户端段调用
StateContext context = new StateContext();    //创建环境
        context.handle();    //处理请求
        context.handle();
        context.handle();
        context.handle();
```

**结果：**

```
当前状态是 A.
当前状态是 B.
当前状态是 A.
当前状态是 B.
```

### 【例1】用“状态模式”设计一个学生成绩的状态转换程序。

分析：本实例包含了“不及格”“中等”和“优秀” 3 种状态，当学生的分数小于 60 分时为“不及格”状态，当分数大于等于 60 分且小于 90 分时为“中等”状态，当分数大于等于 90 分时为“优秀”状态，我们用状态模式来实现这个程序。

首先，定义一个抽象状态类（AbstractState），其中包含了环境属性、状态名属性和当前分数属性，以及加减分方法 addScore(intx) 和检查当前状态的抽象方法 checkState()；然后，定义“不及格”状态类 LowState、“中等”状态类 MiddleState 和“优秀”状态类 HighState，它们是具体状态类，实现 checkState() 方法，负责检査自己的状态，并根据情况转换；最后，定义环境类（ScoreContext），其中包含了当前状态对象和加减分的方法 add(int score)，客户类通过该方法来改变成绩状态。图 2 所示是其结构图。

![学生成绩的状态转换程序的结构图](http://c.biancheng.net/uploads/allimg/181116/3-1Q11615425V39.gif)

```
//环境类
public class ScoreContext {
    private AbstractState state;

    public ScoreContext() {
        state = new LowState(this);
    }

    public AbstractState getState() {
        return state;
    }

    public void setState(AbstractState state) {
        this.state = state;
    }

    public void add(int score) {
        state.addScore(score);
    }
}

// 抽象状态类
public abstract class AbstractState {
    protected ScoreContext context;
    protected String stateName;
    protected int score;

    public abstract void checkState();

    public void addScore(int x) {
        score += x;
        System.out.print("加上：" + x + "分，\t当前分数：" + score);
        checkState();
        System.out.println("分，\t当前状态：" + context.getState().stateName);
    }
}

//具体状态类
public class HighState extends AbstractState {
    public HighState(AbstractState state) {
        context = state.context;
        stateName = "优秀";
        score = state.score;
    }

    @Override
    public void checkState() {
        if (score < 60) {
            context.setState(new LowState(this));
        } else if (score < 90) {
            context.setState(new MiddleState(this));
        }
    }
}


public class MiddleState extends AbstractState {

    public MiddleState(AbstractState state) {
        context = state.context;
        stateName = "中等";
        score = state.score;
    }

    @Override
    public void checkState() {
        if (score < 60) {
            context.setState(new LowState(this));
        } else if (score >= 90) {
            context.setState(new HighState(this));
        }
    }
}

public class LowState extends AbstractState {

    public LowState(ScoreContext c) {
        super.context = c;
        stateName = "不及格";
        score = 0;
    }

    public LowState(AbstractState state) {
        context = state.context;
        stateName = "不及格";
        score = state.score;
    }

    @Override
    public void checkState() {
        if (score >= 90) {
            context.setState(new HighState(this));
        } else if (score >= 60) {
            context.setState(new MiddleState(this));
        }
    }
}
//客户端调用
ScoreContext account = new ScoreContext();
        System.out.println("学生成绩状态测试：");
        account.add(30);
        account.add(40);
        account.add(25);
        account.add(-15);
        account.add(-25);
```

**结果**

```
学生成绩状态测试：
加上：30分，	当前分数：30分，	当前状态：不及格
加上：40分，	当前分数：70分，	当前状态：中等
加上：25分，	当前分数：95分，	当前状态：优秀
加上：-15分，	当前分数：80分，	当前状态：中等
加上：-25分，	当前分数：55分，	当前状态：不及格
```

### 【例2】用“状态模式”设计一个多线程的状态转换程序。

分析：多线程存在 5 种状态，分别为新建状态、就绪状态、运行状态、阻塞状态和死亡状态，各个状态当遇到相关方法调用或事件触发时会转换到其他状态，其状态转换规律如图 3 所示。

![线程状态转换图](http://c.biancheng.net/uploads/allimg/181116/3-1Q116154332451.gif)

现在先定义一个抽象状态类（TheadState），然后为图 3 所示的每个状态设计一个具体状态类，它们是新建状态（New）、就绪状态（Runnable ）、运行状态（Running）、阻塞状态（Blocked）和死亡状态（Dead），每个状态中有触发它们转变状态的方法，环境类（ThreadContext）中先生成一个初始状态（New），并提供相关触发方法，图 4 所示是线程状态转换程序的结构图。

![线程状态转换程序的结构图](http://c.biancheng.net/uploads/allimg/181116/3-1Q116154432352.gif)

```
//抽象状态类
public abstract class ThreadState {
    protected String stateName;
}
//具体状态类
package com.zhyen.base.design_mode.state_mode.thread_state_demo;

public class NewState extends ThreadState {

    public static final String NEW_STATE = "新建状态";

    public NewState() {
        stateName = NEW_STATE;
        System.out.println("当前线程处于：" + stateName);
    }

    public void start(ThreadContext context) {
        if (NEW_STATE.equals(stateName)) {
            System.out.print("调用start()方法-->");
            context.setState(new RunnableState());
        } else {
            System.out.println("当前线程不是新建状态，不能调用start()方法.");
        }
    }
}

package com.zhyen.base.design_mode.state_mode.thread_state_demo;

public class RunnableState extends ThreadState {
    public static final String RUNNABLE_STATE = "就绪状态";

    public RunnableState() {
        stateName = RUNNABLE_STATE;
        System.out.println("当前线程处于：" + stateName);
    }
    public void getCPU(ThreadContext context){
        if(RUNNABLE_STATE.equals(stateName)){
            System.out.print("getCPU()方法-->");
            context.setState(new RunningState());
        }else{
            System.out.println("当前线程不是新建状态，不能调用start()方法.");
        }
    }
}
package com.zhyen.base.design_mode.state_mode.thread_state_demo;

public class RunningState extends ThreadState {
    public static final String RUNNING_STATE = "运行状态";

    public RunningState() {
        stateName = RUNNING_STATE;
        System.out.println("当前线程处于：" + stateName);
    }

    public void suspend(ThreadContext hj) {
        System.out.print("调用suspend()方法-->");
        if (RUNNING_STATE.equals(stateName)) {
            hj.setState(new BlockState());
        } else {
            System.out.println("当前线程不是运行状态，不能调用suspend()方法.");
        }
    }

    public void stop(ThreadContext hj) {
        System.out.print("调用stop()方法-->");
        if (RUNNING_STATE.equals(stateName)) {
            hj.setState(new DeadState());
        } else {
            System.out.println("当前线程不是运行状态，不能调用stop()方法.");
        }
    }
}


package com.zhyen.base.design_mode.state_mode.thread_state_demo;

public class BlockState extends ThreadState {
    public BlockState() {
        stateName = "阻塞状态";
        System.out.println("当前线程处于：阻塞状态.");
    }

    public void resume(ThreadContext hj) {
        System.out.print("调用resume()方法-->");
        if (stateName.equals("阻塞状态")) {
            hj.setState(new RunnableState());
        } else {
            System.out.println("当前线程不是阻塞状态，不能调用resume()方法.");
        }
    }
}


package com.zhyen.base.design_mode.state_mode.thread_state_demo;

public class DeadState extends ThreadState {
    public DeadState() {
        stateName = "死亡状态";
        System.out.println("当前线程处于：死亡状态.");
    }
}

//环境类
package com.zhyen.base.design_mode.state_mode.thread_state_demo;

public class ThreadContext {
    private ThreadState state;

    public ThreadContext() {
        this.state = new NewState();
    }

    public ThreadState getState() {
        return state;
    }

    public void setState(ThreadState state) {
        this.state = state;
    }

    public void start() {
        ((NewState) state).start(this);
    }

    public void getCPU() {
        ((RunnableState) state).getCPU(this);
    }

    public void suspend() {
        ((RunningState) state).suspend(this);
    }

    public void stop() {
        ((RunningState) state).stop(this);
    }

    public void resume() {
        ((BlockState) state).resume(this);
    }
}


//客户端调用
ThreadContext context=new ThreadContext();
        context.start();
        context.getCPU();
        context.suspend();
        context.resume();
        context.getCPU();
        context.stop();
```

```
当前线程处于：新建状态
调用start()方法-->当前线程处于：就绪状态
getCPU()方法-->当前线程处于：运行状态
调用suspend()方法-->当前线程处于：阻塞状态.
调用resume()方法-->当前线程处于：就绪状态
getCPU()方法-->当前线程处于：运行状态
调用stop()方法-->当前线程处于：死亡状态.
```

