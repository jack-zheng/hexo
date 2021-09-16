---
title: Ex13 Engine 和 Host
date: 2021-09-16 14:12:43
categories:
- Tomcat
tags:
- How Tomcat Works
---

> **Chapter 13** presents the two other containers: host and engine. You can also find the standard implementation of these two containers: org.apache.catalina.core.StandardHost and org.apache.catalina.core.StandardEngine.

介绍了另外两个 Container 概念：engine 和 host。如果你的系统需要有一个以上的 context，那你就需要 host 了。如果只有一个 context，理论上可以不用 host。

## The Host Interface

Container 的一个接口实现，最重要的方法为 map

```java
/**
* Return the Context that would be used to process the specified
* host-relative request URI, if any; otherwise return <code>null</code>.
*
* @param uri Request URI to be mapped
*/
public Context map(String uri);
```

## StandardHost

Host 的具体实现，也没什么新鲜的，还是 Container + pipeline + valve 三件套。

```java
public StandardHost() {
    super();
    pipeline.setBasic(new StandardHostValve());
}
```

在它的 start() 方法中还会添加两个新的 valve 进 pipeline, 一个是 ErrorReportValve 另一个是 ErrorDispatcherValve

StandardHost 的 map() 用来寻找匹配的 context。

PS: Tomcat 5 已经不用 Mapper 机制的，直接从 request 中找到正确的 context

## StandardHostMapper

略

## StandardHostValve

里面有涉及 Session 的操作，挺有意思

## Why You Cannot Live without a Host

如果你的应用使用了默认的 ContextConfig 作为配置的对象，那你必须创建 Host。因为它的加载配置文件的方法 applicationConfig() 的实现如下

```java
URL url = servletContext.getResource(Constants.ApplicationWebXml);

InputSource is = new InputSource(url.toExternalForm());
is.setByteStream(stream);
//...
webDigester.parse(is);
```

servletContext 的实现如下

```java
public URL getResource(String path) throws MalformedURLException {
    DirContext resources = context.getResources();
    if (resources != null) {
        String fullPath = context.getName() + path;

        // this is the problem. Host must not be null
        String hostName = context.getParent().getName();
        //...
```

最后一行表示，当使用 ContextConfig 时，你必须有一个 Host 类型的 parent

## Bootstrap1

使用 Host 的案例

## The Engine Interface

Engin 表示整个 Catalina servlet engine, 让你想要你的应用有多个 Host 的时候可以使用它。

对应的实现是 org.apache.catalina.core.StandardEngine。和其他 Container 实现相比，StandardEngine 要单薄的多。套路还是一样，结合 valve 使用。在 addChild() 时，如果不是 Host 类型的 container 会抛异常。 setParent() 时，由于它是顶层 Container，setParent() 会抛异常。

## 问题

按理来说，学到这里应该就可以设置一个网站，多个域名了，怎么实现？