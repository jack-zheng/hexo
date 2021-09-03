---
title: Ex06 生命周期
date: 2021-07-29 16:36:06
categories:
- Tomcat
tags:
- How Tomcat Works
---

> **Chapter 6** explains the Lifecycle interface.
> This interface defines the lifecycle of a Catalina component and provides an elegant way of notifying other components of events that occur in that component.
> In addition, the Lifecycle interface provides an elegant mechanism for starting and stopping all the components in Catalina by one single start/stop.

Catalina 是由多个模块组成的，当 Catalina 启动时，这些模块也要一起启动，停止时也是。比如 destroy 所有的 servlet，将 session 存到二级缓存等。Catalina 通过 Lifecycle 接口管理这些事件。

实现了 Lifecycle 的组件可以出发以下事件

* BEFORE_START_EVENT
* START_EVENT
* AFTER_START_EVENT
* BEFORE_STOP_EVENT
* STOP_EVENT
* AFTER_STOP_EVENT

LifecycleEvent 这个接口表示上面这些事件。这些事件可以由 LifecycleListener 监听。本章会介绍上面这些类，介绍 LifecycleSupport，他可以帮助组件处理事件和监听器。

理解本章的关键点是理解如下概念

Lifecycle: 实现这个接口的 component 可以发送 event

LifecycleEvent: 代表具体的 event 事件，比如 开始，停止之类的

LifecycleListener: 监听事件的类

LifecycleSupport: Util 类，提供简化处理事件的方法. 一个实现了 Lifecycle 的 class 如果要添加 Listener 就得内部创建一些容器，比如 ArrayList 管理这些 Listener。LifecycleSupport 就是用来代替这些容器的。

## The Lifecycle Interface

Catalina 允许一个 component 包含另一个 component。比如 container 可以包含 loader，manager 等。父组件需要管理子组件的起止。

```java
public interface Lifecycle {
    String START_EVENT = "start";
    String BEFORE_START_EVENT = "before_start";
    String AFTER_START_EVENT = "after_start";
    String STOP_EVENT = "stop";
    String BEFORE_STOP_EVENT = "before_stop";
    String AFTER_STOP_EVENT = "after_stop";

    void addLifecycleListener(LifecycleListener var1);

    LifecycleListener[] findLifecycleListeners();

    void removeLifecycleListener(LifecycleListener var1);

    void start() throws LifecycleException;

    void stop() throws LifecycleException;
}
```

start 和 stop 是其中的核心方法，component 提供对应的实现，parent component 可以 调用start/stop 方法。其他三个方法是用来监听事件的。

## The LifecycleEvent Class

```java
public final class LifecycleEvent extends EventObject {

    public LifecycleEvent(Lifecycle lifecycle, String type) {
        this(lifecycle, type, null);
    }

    public LifecycleEvent(Lifecycle lifecycle, String type, Object data) {
        super(lifecycle);
        this.lifecycle = lifecycle;
        this.type = type;
        this.data = data;
    }

    private Object data = null;

    private Lifecycle lifecycle = null;

    private String type = null;

    public Object getData() {
        return (this.data);
    }

    public Lifecycle getLifecycle() {
        return (this.lifecycle);
    }

    public String getType() {
        return (this.type);
    }
}
```

## The LifecycleListener Interface

```java
public interface LifecycleListener {
    /**
     * Acknowledge the occurrence of the specified event.
     *
     * @param event LifecycleEvent that has occurred
     */
    public void lifecycleEvent(LifecycleEvent event);
}
```

## The LifecycleSupport Class

这个 support 类内部声明了一个 Array 变量存储要操作的 Listener: `private LifecycleListener listeners[] = new LifecycleListener[0];`. 主就三个方法:

* addLifecycleListener
* findLifecycleListeners
* fireLifecycleEvent
* removeLifecycleListener

addLifecycleListener(LifecycleListener listener) 被调用时，老的 listener list size + 1， 老的 listener 被拷贝，最后再加上新的这个 listener。

removeLifecycleListener(LifecycleListener listener): 遍历所有的 listener 如果有则删除，最后新建一个 size - 1 的 list，将原来的 list 拷贝进去

## The Application

实验环境是在 ex05 之上，删掉了额外 valves，context 实现了 Lifecycle 和 listener 接口。

{% plantuml %}
class LifecycleSupport
interface Lifecycle
interface LifecycleListener

LifecycleSupport <.. "uses" SimpleContext
LifecycleSupport <.. "uses" SimpleWrapper

Lifecycle <|.. SimpleContext
Lifecycle <|.. SimpleWrapper
Lifecycle <|.. SimplePipeline
Lifecycle <|.. SimpleLoader
Lifecycle <|.. SimpleContextMapper

LifecycleListener <|.. SimpleContextLifecycleListener

Lifecycle .> LifecycleListener
{% endplantuml %}

启动项目可以看到如下 log

```txt
HttpConnector Opening server socket on all host IP addresses
HttpConnector[8080] Starting background thread
SimpleContextLifecycleListener's event before_start
Starting SimpleLoader
Starting Wrapper Primitive
Starting Wrapper Modern
SimpleContextLifecycleListener's event start
Starting context.
SimpleContextLifecycleListener's event after_start

SimpleContextLifecycleListener's event before_stop
SimpleContextLifecycleListener's event stop
Stopping context.
Stopping wrapper Primitive
Stopping wrapper Modern
SimpleContextLifecycleListener's event after_stop
```

## 问题

Q: SimpleContext 中 fire 的 event 是什么时候被 listener 执行的，代码在哪里

A: 在 SimpleContext 的 start() 里。但是和我臆想的事件处理不同，感觉这最多是个伪事件。我本以为他会有一个多线程之类的东西。
实际处理的时候，start() 方法一开始就通过 support 发出了一个 before 的 event，support 发送的时候会调用 listener 实现对应的逻辑。说到底还是串行操作。

Q: 这个 event 和 listener 是不是用了观察这模式啊，复习一下

A: 貌似没有，没看出来

## 解构

复盘一下这个 Lifecycle 的原理。Tomcat 提供了一个机制，通过套娃的方式，让所有相关的组件知道某个事件发生了，并让他可以采取相应的动作。

PS: 解构的时候才注意到，start() 和 stop() 方法中操作的 component 对象，顺序上是相反的，666

这里当 event 发生时，有两种对象需要进行操作，一种是 Component，代表 Tomcat 里的 service 组件。另一种是 Listener，我的理解是，独立于 Tomcat 之外的一些 service，比如 log 之类的东西。

Listener 说白了就是定义了一个接口，接受 event 作为参数，实现中通过 if-else 判断 event 类型并采取对应的行为

```java
public interface LifecycleListener {
    /**
     * Acknowledge the occurrence of the specified event.
     *
     * @param event LifecycleEvent that has occurred
     */
    public void lifecycleEvent(LifecycleEvent event);
}
```

没有理解透彻，不能很顺利的重构出这个模型，但是理解层面的话已经做到了。