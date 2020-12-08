---
title: 为什么我们要在打印 log 之前进行 isInfoEnabled 判断？
date: 2020-12-08 15:33:32
categories:
- 编程
tags:
- java
- logger
---

平时我们在代码中打印 log 基本都是直接调用 `logger.info()` 方法，官方推荐在外面再包一层 `if (logger.isInfoEnabled())` 判断，Why?

以 log4j 为例：

```java
private static final Logger logger = Logger.getLogger(ExampleBeanWithSetter.class);

if (logger.isInfoEnabled()) {
    logger.info("Hello world...");
}
```

如果没有加外层的 `isInfoEnabled` 判断，那个将会直接执行 `logger.info(String)` 方法， 相对于有判断的形式，多了一步拼接字符串的步骤。换句话说，最大的区别有两点：

1. 拼接字符串耗时
2. 这些字符串也会消耗内存空间

对一般的小系统当然是没什么影响，如果是高并发或者 log 很多的系统，可以作为一个优化的方向。