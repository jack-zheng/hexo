---
title: Design-pattern-chain-of-responsibility
date: 2021-07-23 13:47:46
categories:
- 设计模式 
tags:
- Design Pattern
---

> 避免请求发送者和接受者的耦合，让多个对象都有可能接受请求，将这些对象连成一条链，并且沿着这条链传递请求，知道有对象处理它为止

典型应用：

* JS 中的冒泡事件
* tomcat 中对 Encoding 的处理
* servlet 的 filter

{% plantuml %}
class Handler {
	-Handler successor
    +handleRequest()
}

Client .> Handler
Handler "successor" o-> Handler

Handler <|-- ConcreateHandlerA
Handler <|-- ConcreateHandlerB
{% endplantuml %}

* Handler(抽象处理者): 定义一个处理请求的接口，提供对后续处理者的引用
* ConcreteHandler(具体处理者): 抽象处理者的子类，处理用户请求，可选将请求处理掉还是传给下家；在具体处理者中可以访问链中下一个对象，以便请求的转发。

## Handler 示例

示例描述：定义一个 handler 的抽象类以及三个对应的实现类。抽象类中定义 handleRequest() 方法作为统一的处理入口，最后创建一个 ChainClient 建立处理链并测试

```java
public abstract class AbstractHandler {
    private AbstractHandler handler;

    public abstract void handleRequest(String condition);

    public AbstractHandler getHandler() {
        return handler;
    }

    public void setHandler(AbstractHandler handler) {
        this.handler = handler;
    }
}

public class ConcreteHandlerA extends AbstractHandler {
    @Override
    public void handleRequest(String condition) {
        if (condition.equals("A")) {
            System.out.println("Concrete Handler A processed...");
        } else {
            System.out.println("Concrete Handler A can't process, call other handler...");
            getHandler().handleRequest(condition);
        }
    }
}

public class ConcreteHandlerB extends AbstractHandler {
    @Override
    public void handleRequest(String condition) {
        if (condition.equals("B")) {
            System.out.println("Concrete Handler B processed...");
        } else {
            System.out.println("Concrete Handler B can't process, call other handler...");
            getHandler().handleRequest(condition);
        }
    }
}

public class ConcreteHandlerZ extends AbstractHandler {
    @Override
    public void handleRequest(String condition) {
        // 一般就是最后一个处理器
        System.out.println("Concrete handler z processed...");
    }
}

public class ChainClient {
    public static void main(String[] args) {
        AbstractHandler handlerA = new ConcreteHandlerA();
        AbstractHandler handlerB = new ConcreteHandlerB();
        AbstractHandler handlerZ = new ConcreteHandlerZ();

        // 设置链顺序
        handlerA.setHandler(handlerB);
        handlerB.setHandler(handlerZ);

        System.out.println("--------------- handle A ---------------");
        handlerA.handleRequest("A");
        System.out.println("--------------- handle B ---------------");
        handlerA.handleRequest("B");
        System.out.println("--------------- handle Z ---------------");
        handlerA.handleRequest("Z");
    }
}

// 执行结果
//
// --------------- handle A ---------------
// Concrete Handler A processed...
// --------------- handle B ---------------
// Concrete Handler A can't process, call other handler...
// Concrete Handler B processed...
// --------------- handle Z ---------------
// Concrete Handler A can't process, call other handler...
// Concrete Handler B can't process, call other handler...
// Concrete handler z processed...
```

## Log 示例

目标：实现一个 log 机制，当 log level 比当前 log 类的 level 高是，记录它

这个示例的套路和上一个很类似，但是它把处理逻辑固定了，在抽象类中做了实现，每个具体的类只定制自己的 write 行为即可，和上一个在思路上有些许的不同

```java
public abstract class AbstractLogger {
    public static int INFO = 1;
    public static int DEBUG = 2;
    public static int ERROR = 3;

    protected int level;

    //责任链中的下一个元素
    protected AbstractLogger nextLogger;

    public void setNextLogger(AbstractLogger nextLogger){
        this.nextLogger = nextLogger;
    }

    public void logMessage(int level, String message){
        if(this.level <= level){
            write(message);
        }
        if(nextLogger !=null){
            nextLogger.logMessage(level, message);
        }
    }

    abstract protected void write(String message);
}

public class ConsoleLogger extends AbstractLogger {
    public ConsoleLogger(int level) {
        this.level = level;
    }

    @Override
    protected void write(String message) {
        System.out.println("Standard Console::Logger: " + message);
    }
}

public class ErrorLogger  extends AbstractLogger {
    public ErrorLogger(int level){
        this.level = level;
    }

    @Override
    protected void write(String message) {
        System.out.println("Error Console::Logger: " + message);
    }
}

public class FileLogger extends AbstractLogger {
    public FileLogger(int level) {
        this.level = level;
    }

    @Override
    protected void write(String message) {
        System.out.println("File::Logger: " + message);
    }
}

public class ChainPatternDemo {
    private static AbstractLogger getChainOfLoggers(){

        AbstractLogger errorLogger = new ErrorLogger(AbstractLogger.ERROR);
        AbstractLogger fileLogger = new FileLogger(AbstractLogger.DEBUG);
        AbstractLogger consoleLogger = new ConsoleLogger(AbstractLogger.INFO);

        errorLogger.setNextLogger(fileLogger);
        fileLogger.setNextLogger(consoleLogger);

        return errorLogger;
    }

    public static void main(String[] args) {
        AbstractLogger loggerChain = getChainOfLoggers();

        System.out.println("--------------- handle info ---------------");
        loggerChain.logMessage(AbstractLogger.INFO, "This is an information.");

        System.out.println("--------------- handle debug ---------------");
        loggerChain.logMessage(AbstractLogger.DEBUG, "This is a debug level information.");

        System.out.println("--------------- handle Error ---------------");
        loggerChain.logMessage(AbstractLogger.ERROR, "This is an error information.");
    }
}

// --------------- handle info ---------------
// Standard Console::Logger: This is an information.
// --------------- handle debug ---------------
// File::Logger: This is a debug level information.
// Standard Console::Logger: This is a debug level information.
// --------------- handle Error ---------------
// Error Console::Logger: This is an error information.
// File::Logger: This is an error information.
// Standard Console::Logger: This is an error information.
```