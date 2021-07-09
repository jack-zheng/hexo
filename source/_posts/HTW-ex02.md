---
title: HTW ex02
date: 2021-07-09 19:35:15
- Tomcat
tags:
- How Tomcat Works
---

servlet container 会为 servlet 做如下事情

* 第一次调用 servlet 时，加载 servlet 并执行 init 方法
* 为每一个连接创建对应的 ServletRequest 和 ServletResponse 对象
* 调用 service 方法，传入前面声明的两个对象
* servlet 生命周期结束时，调用 destroy 方法

ex02 的第一个 container 不是完整功能的 server，没有实现 init 和 destroy 的 servlet 的功能，它主要 focus 在如下的功能点上

* 等待请求
* 构建 ServletRequest 和 ServletResponse 对象
* 如果求请求的是静态资源，调用 StaticResourceProcessor 相关方法
* 如果请求 servlet， 加载对应的 servlet 并调用 service 方法

TODO 添加 app01 UML 图