---
title: Ex12 StandardContext
date: 2021-09-15 19:52:57
categories:
- Tomcat
tags:
- How Tomcat Works
---

> **Chapter 12** covers the org.apache.catalina.core.StandardContext class that represents a web application. In particular this chapter discusses how a StandardContext object is configured, what happens in it for each incoming HTTP request, how it supports automatic reloading, and how Tomcat 5 shares a thread that executes periodic tasks in its associated components.

本章沿用 11 章的代码，主要介绍一下内容

* StanadradContextMapper 和 ContextConfig
* Http request 接收到之后的调用链
* StandardContext 中的重要属性
* Tomcat 5 中的 backgroundProcess

## StandardContext Configuration

创建完 StandardContext 的实例后必须调用一下他的 start() 方法，这个过程中他会做以下事情

* 置位 available flag
* 读取 CATALINA_HOME/conf 路径下的配置文件
* 在 listener 中进行 context 的配置

细节在 15 章中再讲

### StandardContext Class's Constructor

构造函数，设置默认的 valve

```java
public StandardContext() {
    super();
    pipeline.setBasic(new StandardContextValve());
    namingResources.setContainer(this);
}
```

### Starting StandardContext

只要做了 flag 置位和 listener 的处理，代码很清楚，主要做了如下事情

* fire BEFORE_START event
* availability flag 置位
* configured flag 置位
* set resources
* set manager
* init character set manager
* 启动 context 相关联的其他 component
* 启动子 container
* 启动 pipeline
* 启动 manager
* fire START event, listener(ContextConfig) 会进行一些配置，成功了之后 configured flag 置位
* 如果 configured 为 true，进行一些其他配置工作
* fire AFTER_START event

PS: Tomcat 5 中逻辑基本一致，此外还增加了 JMX 相关的代码

## StandardContextMapper

StandardContext 的 invoke 方法调用时，会调用自己的 StandardContextValve 的 invoke 方法，它做的第一件事是拿到处理 request 匹配的 wrapper。

ContainerBase 类中有 addDefaultMapper() 方法，实现如下

```java
protected void addDefaultMapper(String mapperClass) {
    // Do we need a default Mapper?
    if (mapperClass == null)
        return;
    if (mappers.size() >= 1)
        return;

    // Instantiate and add a default Mapper
    try {
        Class clazz = Class.forName(mapperClass);
        Mapper mapper = (Mapper) clazz.newInstance();
        mapper.setProtocol("http");
        addMapper(mapper);
    } catch (Exception e) {
        log(sm.getString("containerBase.addDefaultMapper", mapperClass),
            e);
    }
}
```

StandardContext 中定义了对应的 mapperClass 类

```java
private String mapperClass = "org.apache.catalina.core.StandardContextMapper"
```

Mapper 中最核心的方法为 map 方法 `public Container map(Request request, boolean update)` 返回 request 对应的 wrapper。mapper 中会一次通过四种筛选条件过滤出目标 wrapper

* 精确匹配
* 前缀匹配
* 后缀匹配
* 默认匹配

如果还是没找到，就返回 null. Tomcat 5 中做了改进，直接从 request 中可以拿到对应的 wrapper

```java
Wrapper wrapper = request.getWrapper();
```

## Support for Reloading

StandardContext 中又一个 reloadable 属性，当 web.xml 改变或者 WEB-INF/classes 文件夹下文件发生改变时，这个 flag 会置位

StandardContext 的 loader - WebappLoader 在执行 setContainer 时会启动一个线程，当上述目录的文件发生改变，loader 会重新加载 application

```java
/**
    * Set the Container with which this Logger has been associated.
    *
    * @param container The associated Container
    */
public void setContainer(Container container) {

    // Deregister from the old Container (if any)
    if ((this.container != null) && (this.container instanceof Context))
        ((Context) this.container).removePropertyChangeListener(this);

    // Process this property change
    Container oldContainer = this.container;
    this.container = container;
    support.firePropertyChange("container", oldContainer, this.container);

    // Register with the new Container (if any)
    if ((this.container != null) && (this.container instanceof Context)) {
        setReloadable( ((Context) this.container).getReloadable() );
        ((Context) this.container).addPropertyChangeListener(this);
    }

}
```

最后一段的 setReloadable() 方法实现如下

```java
/**
* Set the reloadable flag for this Loader.
*
* @param reloadable The new reloadable flag
*/
public void setReloadable(boolean reloadable) {

// Process this property change
boolean oldReloadable = this.reloadable;
this.reloadable = reloadable;
support.firePropertyChange("reloadable",
                            new Boolean(oldReloadable),
                            new Boolean(this.reloadable));

// Start or stop our background thread if required
if (!started)
    return;
if (!oldReloadable && this.reloadable)
    threadStart();
else if (oldReloadable && !this.reloadable)
    threadStop();

}
```

threadStart() 会启动一个线程，持续监测 WEB-INF 文件夹下文件的时间戳，threadStop() 则用来停止这个线程

PS：Tomcat 5 中这个监测过程使用专门的 backgroundProcess 来完成

## The backgroundProcess Method

略

PS：这本书看完之后，为了巩固他，可以看看 Stackoverflow 上 Tomcat 相关的问题，找找灵感
