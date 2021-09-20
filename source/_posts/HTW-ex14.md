---
title: Ex14 Server & Service
date: 2021-09-17 10:27:14
categories:
- Tomcat
tags:
- How Tomcat Works
---

> **Chapter 14** offers the server and service components. A server provides an elegant start and stop mechanism for the whole servlet container, a service serves as a holder for a container and one or more connectors. The application accompanying this chapter shows how to use a server and a service.

前面的实验中，一个 container 只能和一个 connector 关联，并且启动和停止时通过多条指令完成的，Server + Service 可以帮助你更优雅的管理这些服务。

## Server & Implementation

Server 表示的是 Catalina servlet container 以及所属的所有子 component。它提供了一种优雅的方式管理所有服务的开启和停止。

官方定义如下：A Server element represents the entire Catalina servlet container. 

对应的是现实 org.apache.catalina.core.StandardServer，同时还实现了 Lifecycle 接口(start/stop 方法)，以 initialize 方法为例

```java
/**
* Invoke a pre-startup initialization. This is used to allow connectors
* to bind to restricted ports under Unix operating environments.
*/
public void initialize()
throws LifecycleException {
    if (initialized)
        throw new LifecycleException (
            sm.getString("standardServer.initialize.initialized"));
    initialized = true;

    // Initialize our defined Services
    for (int i = 0; i < services.length; i++) {
        services[i].initialize();
    }
}
```

类似的还有 start, stop 方法，处理的逻辑都是类似的，先发送对应的 event，然后 for 循环调用 service 对应的方法

### The await Method

这个方法用来监听停止信号，当条件达成时跳出循环，这个方法是用来代替原来例子中的 `System.in.read()` 方法的

## Service & Implementation

> A Service is a group of one or more Connectors that share a single Container to process their incoming requests.

Service 可以持有多个 connector 和一个 container。多个 connector 可以用来匹配多种 protocol，比如 http 和 https。实现类为 org.apache.catalina.core.StandardService.

```java
private Connector connectors[] = new Connector[0];
private Container container = null;
```

service 在 setContainer 时会调用 container 的 start() 方法，并将它关联到 connector

```java
public void setContainer(Container container) {

    Container oldContainer = this.container;
    if ((oldContainer != null) && (oldContainer instanceof Engine))
        ((Engine) oldContainer).setService(null);
    this.container = container;
    if ((this.container != null) && (this.container instanceof Engine))
        ((Engine) this.container).setService(this);
    if (started && (this.container != null) &&
        (this.container instanceof Lifecycle)) {
        try {
            ((Lifecycle) this.container).start();
        } catch (LifecycleException e) {
            ;
        }
    }
    synchronized (connectors) {
        for (int i = 0; i < connectors.length; i++)
            connectors[i].setContainer(this.container);
    }
    if (started && (oldContainer != null) &&
        (oldContainer instanceof Lifecycle)) {
        try {
            ((Lifecycle) oldContainer).stop();
        } catch (LifecycleException e) {
            ;
        }
    }

    // Report this property change to interested listeners
    support.firePropertyChange("container", oldContainer, this.container);
}
```

同样的，在 addConnector() 的时候，会将它和 container 关联起来

```java
public void addConnector(Connector connector) {

    synchronized (connectors) {
        connector.setContainer(this.container);
        connector.setService(this);
        Connector results[] = new Connector[connectors.length + 1];
        System.arraycopy(connectors, 0, results, 0, connectors.length);
        results[connectors.length] = connector;
        connectors = results;

        if (initialized) {
            try {
                connector.initialize();
            } catch (LifecycleException e) {
                e.printStackTrace(System.err);
            }
        }

        if (started && (connector instanceof Lifecycle)) {
            try {
                ((Lifecycle) connector).start();
            } catch (LifecycleException e) {
                ;
            }
        }

        // Report this property change to interested listeners
        support.firePropertyChange("connector", null, connector);
    }

}
```

Service 也实现了 Lifecycle 接口，在 initialize() 和 start() 方法中会调用 connector 和 container 对应的方法。

## The Application

Bootstrap 实现和前面基本一致，最大的区别是在 Engine 声明之后，声明了这节介绍的 Server 和 Service 作为管理 Engine 的容器

```java
Service service = new StandardService();
service.setName("Stand-alone Service");
Server server = new StandardServer();
server.addService(service);
service.addConnector(connector);
```

并且使用 await 代替 `System.in.read();`