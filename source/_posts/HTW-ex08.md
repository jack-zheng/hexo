---
title: HTW ex08
date: 2021-08-02 11:14:13
categories:
- Tomcat
tags:
- How Tomcat Works
---

> **Chapter 8** explains about loaders. 
> A loader is an important Catalina module responsible for loading servlet and other classes that a web application uses. 
> This chapter also shows how application reloading is achieved.

之前章节我们已经给出了一个简单的 loader 实现用于加载 servlet。这章我们将介绍 tomcat 的 standard web application loader. servlet container 必须实现自己的 loader，而不能使用系统自带的那个。因为它不能相信 运行的 servlets。如果它像我们之前的例子那样使用默认的类加载器，那么 servlet 将可以访问任何 JVM classpath 下的 class 和 lib，这和 security 的规则相违背。

一个 servlet 只允许加载 WEB-INF/classes 和 WEB-INF/lib 文件夹下的内容, 那个 web application(context) 需要有它自己的 loader。Catalina 中，org.apache.catalina.Loader 表示 loader 类。

另一个 tomcat 需要自己的 loader 的原因是它需要支持自动加载的功能。当 WEB-INF/classes 和 lib 下的内容发生改变时，这个 loader 需要自动检测并重新加载。Tomcat 新起一个线程完成这个功能。reload 的接口为 org.apache.catalina.loader.Reloader

本章第一部分介绍 Java 中的类加载机制。之后介绍 Loader 接口，最后延时 tomcat 的 loader 使用案例

本章中两个术语 repository 表示 class loader 会搜索的地方，resources 表示 DirContext，它指向 context 的 document 目录。

PS: 看这章的介绍感觉可以复习一下类加载机制相关章节了，中间说的用同一个 class loader 会有安全问题的事情不是很清楚

## Java Class Loader

每当你创建一个 Java 实例时，对应的类必须加载到内存中。JVM 会使用 class loader 搜索 Java 的核心类库和 classpath 中包含的环境变量。如果没有找到对应的实例，则会跑出 ClassNotFoundException。

Java 1.2 开始，JVM 通过三个类加载类文件，他们是 bootstrap class loader, extension class loader 和 system class loader。三者是父子关系，bootstrap class loader 是父节点，system class loader 是最后的子节点。

bootstrap class loader 用来 bootstrap JVM，它用来加载 JVM 启动所需的类，通过 native code 实现。它还负责加载所有的 Java 核心类，比如 java.lang 和 i。 也加载 rt.jar 和 i8n.jar 等 lib.

extension class loader 负责加载 standard extension 文件夹下的内容，各个供应商的文件夹可能不一样，Sun 的标准扩展文件夹是 /jdk/jre/lib/ext

system class loader 负责加载classpath 中的环境变量

为了保证安全，JVM 使用 delegation model 来加载类。每当一个类需要被加载时，他会先委托给 system class loader 然后再委托给 extension class loader 最后给 bootstrap class loader。当 bootstrap 找不到时，extension class loader 再找，最后是 system class loader。如果都没找到，抛出 ClassNotFoundException.

PS: 感觉这三者唯一的区别就是 load class 的路径不同，是不是可以理解为我在指定路径下加载类，保证特定的类是安全的

举例这种安全模型的作用，比如我是黑客，像破坏 Java 系统，我自己写了一个 java.lang.Object 的 class 来代替官方的这个类。但是由于前面说的这种代理机制，JVM 会到指定路径下加载 Object 类，所以我的企图不能实现。

Java 允许你定制自己的 class loader，只需继承 java.lang.ClassLoader 即可。以下是 tomcat 需要定制 loader 的原因

* To specify certain rules in loading classes.
* To cache the previously loaded classes.
* To pre-load classes so they are ready to use.

