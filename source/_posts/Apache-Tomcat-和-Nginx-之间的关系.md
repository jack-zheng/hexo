---
title: 'Apache, Tomcat 和 Nginx 之间的关系'
date: 2020-07-03 13:47:56
categories:
- 科普
tags:
- 解释
---
想要解决的问题：

1. Apache, Tomcat 和 Nginx 的定义/区别
2. Server 搭配拓扑图

## Apache, Tomcat 和 Nginx 的定义/区别

名词解释：

* 静态服务器，就是每次访问同一个地址只能返回同样的内容，不会改变

Apache

> 这里说的 Apache 指的是 Apache Http Server。静态服务器的一种，老牌(始于1995)，曾经的王者，近年来市场占有率下降。
> 模块多，性能稳定，rewrite 性能搞，配置相对复杂

Nginx

> 毛子出品，2004年首发，声势迅猛。如今是三巨头之一(另两个是Microsoft, Apache)，和 Apache 是同类产品。
> 支持反向代理，轻量级，非阻塞，高并发，社区活跃，bug 多

Tomcat

> 全名是 Apache Tomcat，Application Server 的一种，用来提供动态支持，和前面的不是一种类型。

## Server 搭配拓扑图

```txt
                                                       +-----------+
                                              -------->|  Tomcat01 |
                                              |        |           |
+--------------+          +------------+      |        +-----------+
| Client       |          |Apache/Nginx|      |
|              |--------> |            |------|
|              |          |            |      |
+--------------+          +------------+      |
                                              |        +-----------+
                                              -------->| Tomcat02  |
                                                       |           |
                                                       +-----------+
```
