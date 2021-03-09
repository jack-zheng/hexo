---
title: Java 通过 ClassLoader 获取 resources
date: 2020-11-06 19:22:08
categories:
- java
tags:
- snip
---

今天突然发现了一个 class loader 的新用法

```java
Enumeration<URL> urls = Thread.currentThread().getContextClassLoader().getResources("my.xml");
```

`getResources` 竟然连 jar 包中的资源文件也会加载，以前一只以为只会加载当前项目的资源的，孤陋寡闻了，哈哈哈哈 ε-(´∀｀; )

