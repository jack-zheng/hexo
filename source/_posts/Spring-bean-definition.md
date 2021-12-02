---
title: Spring bean definition
date: 2021-11-30 13:13:05
categories:
- Spring
tags:
- BeanDefinition
---

根据[官方](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#bean-overview)定义, 简单来说，bean definition 就是一个 bean 的定义

A Spring IoC container manages one or more beans. These beans are created with the configuration metadata that you supply to the container (for example, in the form of XML <bean/> definitions).

Within the container itself, these bean definitions are represented as BeanDefinition objects, which contain (among other information) the following metadata:
* A package-qualified class name: typically, the actual implementation class of the bean being defined.
* Bean behavioral configuration elements, which state how the bean should behave in the container (scope, lifecycle callbacks, and so forth).
* References to other beans that are needed for the bean to do its work. These references are also called collaborators or dependencies.
* Other configuration settings to set in the newly created object — for example, the size limit of the pool or the number of connections to use in a bean that manages a connection pool.

bean definition 包含：

* 全名限定，通常是实现类的全名限定
* bean 相关配置，包括 scope, lifecycle 等
* bean 的依赖
* 实例化的属性，比如 pool size 之类的

## 参考

* [Zhihu](https://zhuanlan.zhihu.com/p/189896257)
