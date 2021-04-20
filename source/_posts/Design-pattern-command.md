---
title: 命令模式
date: 2021-03-16 15:31:47
- 设计模式
tags:
- command pattern
- 命令模式
---

**Command** is a behavioral design pattern that turns a request into a stand-alone object that contains all information about the request. This transformation lets you parameterize methods with different requests, delay or queue a request’s execution, and support undoable operations.

> 命令模式是行为模式的一种，它将请求包裹在一个单独的对象里面，包含了一个请求的所有信息，通过这种方式，它让你的处理方法有一个统一的参数格式，并且这种处理方式为处理延迟，队列和回滚等操作提供可行性

优点：

* 较容易的设计一个命令队列
* 容易将命令记入日志
* 允许请求放取消
* 容易的实现撤销和重做
* 加入新的命令不会影响已有的实现
* 将 command 从执行类中剥离出来，独立存在

## 解释

角色定义：

* Client: 终端，初始化对象，类似执行环境，可以代表一个人
* Command: 代表一个命令的接口
* Concrete Command: 具体的命令，会持有一个 Receiver 引用
* Receiver: 有能力**执行**具体命令的主体
* Invoker(Sender): **持有**命令的类似容器的东西

PS: 感觉上，写 demo 的时候可以把 Client 和 Invoker 合并成一个东西

```java
public class Receiver {
    public void doSomething() { System.out.println("Receiver do something ..."); }
}

public interface Command {
    void execute();
}

public class ConcreteCommand implements Command {
    Receiver receiver;

    public ConcreteCommand(Receiver receiver) { this.receiver = receiver; }

    @Override
    public void execute() { receiver.doSomething(); }
}

public class Invoker {
    private Command cmd;

    public Invoker(Command cmd) { this.cmd = cmd; }

    public void action() { cmd.execute(); }
}

public class Client {
    public static void main(String[] args) {
        Receiver receiver = new Receiver();
        Command cmd = new ConcreteCommand(receiver);
        Invoker invoker = new Invoker(cmd);

        invoker.action();
    }
}

// Receiver do something ...
```

## 案例分析

TIJ4 中 inner class 下的 Inner classes & control frameworks 小结说的 GreenHouse 采用了 Command Pattern，说实话我不是理解，应该是我对这个模式的使用太刻板了，不够变通吧

### 大话设计模式

用烧烤摊和烧烤店来举例子，烧烤摊客户直接和摊主紧耦合，混乱。烧烤店中有服务员做中介，效率高

```java
public abstract class Command {
    protected Barbecuer barbecuer;
    public Command(Barbecuer barbecuer) {
        this.barbecuer = barbecuer;
    }

    public abstract void execute() ;
}

public class BakeChickenWingCommand extends Command {
    public BakeChickenWingCommand(Barbecuer barbecuer) {
        super(barbecuer);
    }

    @Override
    public void execute() {
        barbecuer.bakeChickenWing();
    }
}

public class BakeMuttonCommand extends Command {
    public BakeMuttonCommand(Barbecuer barbecuer) {
        super(barbecuer);
    }

    @Override
    public void execute() {
        barbecuer.bakeMutton();
    }
}

public class Barbecuer {
    public void bakeMutton() {
        System.out.println("Bake mutton...");
    }

    public void bakeChickenWing() {
        System.out.println("Bake chicken wing...");
    }
}

public class Waiter {
    private List<Command> cmds = new ArrayList<>();

    public void setOrder(Command cmd) {
        cmds.add(cmd);
        System.out.println("Set order: " + cmd.getClass().getSimpleName() + ", Time: " + new Date());
    }

    public void cancelOrder(Command cmd) {
        cmds.remove(cmd);
        System.out.println("Cancel order: " + cmd.getClass().getSimpleName() + ", Time: " + new Date());
    }

    public void noteBaker() {
        for (Command cmd : cmds) {
            System.out.println("Process order: " + cmd.getClass().getSimpleName() + ", Time: " + new Date());
        }
    }
}

public class Client {
    public static void main(String[] args) {
        Barbecuer baker = new Barbecuer();
        Command bakeWing = new BakeChickenWingCommand(baker);
        Command bakeMutton = new BakeMuttonCommand(baker);

        Waiter waiter = new Waiter();
        waiter.setOrder(bakeMutton);
        waiter.setOrder(bakeWing);
        waiter.setOrder(bakeWing);
        waiter.setOrder(bakeMutton);
        waiter.cancelOrder(bakeMutton);

        waiter.noteBaker();
    }
}
```

**Head First Design Pattern**

支持宏模式的 command pattern 它管这种模式叫 Meta Command Pattern

## 资料

* [Refactoring Guru](https://refactoring.guru/design-patterns/command)
* [Jianshu sample](https://www.jianshu.com/p/5901e76a6348)