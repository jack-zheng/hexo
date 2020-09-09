---
title: Spring5 note
date: 2020-09-08 21:28:47
categories:
- 编程
tags:
- java
- spring
---

[B 站狂神 Spring5 教程笔记](https://www.bilibili.com/video/BV1WE411d7Dv)

## Spring 基本概念

七大组成

1. AOP
2. ORM
3. Web
4. DAO
5. Context
6. Web MVC
7. Core

* Spring Boot
  * 快速开发脚手架
  * 快速开发单个微服务
  * 约定大于配置
* Spring Cloud
  * 基于 SpringBoot 实现的

弊端：发展太久，违背原来的理念。配置繁琐，人称 '配置地狱'

## IoC 理论推导 （Inversion of Control）

原来的实现

1. UserDao 接口
2. UserDaoImpl 实现类
3. UserService 业务接口
4. UserServiceImpl 业务实现类

用户的需求可能影响到原来的代码，我们需要根据用户需求修改源代码（修改 UserDaoImpl 中的 Dao 生成）

通过 set 方法主入后，实现被动接受对象，需求由外部决定。不在管理对象创建，专注于扩展业务。

```java
private UserDao userDao;

// 利用 set 动态注入实现
public void setUserDao(UserDao userDao) {
    this.userDao = userDao;
}
```

## IoC 的本质

控制反转是一种**设计思想**，DI（Dependency Injection） 是 IoC 的一种实现方式，将对象的创建交给第三方，获取对象的方式的反转。

Spring 是一种实现控制反转的 IoC 容器，常见的有两种对象控制方式，XML 和 注解。XML 配置 Bean, 定义和实现是分离的。注解方式则把两者结合在了一起，从而达到零配置。

[Spring Framework 官方文档](https://docs.spring.io/spring/docs/current/spring-framework-reference/core.html#beans-factory-metadata)

## IoC创建对象的方式

1. 默认使用无参构造创建对象
2. 通过 constructor-arg 标签实现带参构造器功能

在 xml 加载完后，配置的对象就已经被创建了

## Spring 配置说明

1. alias 别名，和 bean 的 name 属性重复，而且 name 更灵活
2. bean 对象生成配置
3. import 合并多个 xml 配置文件

## DI - 依赖注入

1. 构造器注入
2. Set方式注入 - 即依赖注入
3. 其他注入

依赖： bean 对象的创建依赖容器
注入： bean 对象的所有属性由容器来注入

## P/C命名空间注入

在 xml 中导入约束即可使用

```xml
xmlns:p="http://www.springframework.org/schema/p"
 xmlns:c="http://www.springframework.org/schema/c"
```

P 可以扩展属性注入，一个 tag 解决，不用嵌套xml了

C 可以扩展构造器

## Bean 的 作用域(scope)

1. singleton - 默认域
2. prototype - 每次取 bean 都会产生新对象

```xml
<bean id="accountService" class="com.something.DefaultAccountService" scope="singleton"/>
<bean id="accountService" class="com.something.DefaultAccountService" scope="prototype"/>
```

## Bean 的自动装配

* 自动装配是 Spring 满足 bean 依赖的一种方式
* Spring 在上下文中自动寻找，并自动给 bean 装配属性

Spring 三种装配方式：

1. xml
2. 注解
3. 隐式的自动装配 bean