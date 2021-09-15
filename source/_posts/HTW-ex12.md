---
title: HTW ex12
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

### The Invoke Method