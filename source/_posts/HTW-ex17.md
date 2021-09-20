---
title: Ex17 Tomcat Startup
date: 2021-09-20 16:28:19
categories:
- Tomcat
tags:
- How Tomcat Works
---

> **Chapter 17** discusses the starting and stopping of Tomcat through the use of batch files and shell scripts. 

Tomcat 中通过两个 class 管理容器的启动分别是 Catalina 和 Bootstrap。Catalina 用来管理服务器的开启和停止，同时负责解析 server.xml 配置文件。 Bootstrap 用来创建 Catalina 实体并运行。理论上来说，这两个类可以合并，但是为了支持多模式运行，还是分开了。

这章先介绍了 Catalina 和 Bootstrap，然后介绍了管理 Tomcat 的 dot 和 shell 文件的编写

## The Catalina Class

管理主入口为 process 方法，通过 Bootstrap 管理，但其实它本身就包含一个 main() 函数，调用方式和 Bootstrap 一样

```java
public void process(String args[]) {

    setCatalinaHome();
    setCatalinaBase();
    try {
        if (arguments(args))
            execute();
    } catch (Exception e) {
        e.printStackTrace(System.out);
    }
}
```

### The start Mehtod

start 方法中，会做如下事情

1. 创建对应的 digester 对象处理 server.xml
2. 调用解析得到的 server 容器的 start 方法
3. 注册 shutdown hook
4. 卡在 await 等待结束信号
5. 正常结束时会先移掉 hook 避免重复执行

### The stop Mehtod

执行原理和逻辑和 start 一致，跳过。

### Start/Stop Digester

逻辑一致，创建了一个 Digester 对象，主要的逻辑都在 rule set 那一部分，很直观

## The Bootstrap Class

Tomcat 通过 Bootstrap 调用 Catalina，我们平时启动用的 dot/shll 文件也是直接调用的 Bootstrap 类。其中的 main 方法创建了三个 class loader 并对 Catalina 做了实例话，最后调用了他的 process 方法启动服务器。

PS: 这部分后面可以结合第八章仔细看看，应该挺有意思

## Running Tomcat on Windows/Linux

略