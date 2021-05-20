---
title: Spring usage collections
date: 2021-05-20 16:47:39
categories:
- Spring
tags:
- Spring
---

## 通过工行方法 + 注解注入 bean

今天接到一个任务，需要通过工厂方法注入一个 bean 到 context 中，抄了别人的代码，但是还是不生效，再三测试，发现是类型赋错了。简化后，示例如下

```java
person

student -> person

factory::getStudent@Bean, return Person

返回值错了，导致注入失败
```

## spring 如何加载 bean？

对于公司这个老项目，它又是如何加载的？