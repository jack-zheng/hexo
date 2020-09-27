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

## 使用注解开发

Spring4 之后，要使用注解需要保证 AOP 包已经导入。XML 也需要添加特殊的约束 `<context:annotation-config/>`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context
        https://www.springframework.org/schema/context/spring-context.xsd">

    <!-- 设置扫描路径 -->
    <context:component-scan base-package="com.jzheng.pojo"/>
    <context:annotation-config/>

</beans>
```

1. bean - @Component
2. 属性 - @Value
3. 衍生的注解 - @Repository - for dao/@Service - for service/@Controller - for controller 作用都是将对象注入到容器
4. 自动装配
   1. @Autowired 通过类型，名字装配。如果不能自动装配属性，可以通过 @Qualifier(value="xxx)
   2. @Nullable，允许为空
   3. @Resource，通过名字，类型装配
5. 作用域 - @Scope
6. 小结: XML 更加万能，使用任何场合；注解只能在自己的class 里使用。

推荐做法：XML 用来管理 Bean，注解只用来注入属性

## 使用 Java 的方式配置 Spring

JavaConfig 式 Spring 一个子项目， Spring4 之后成为核心项目。通过 @Configuration 注解来实现，可以代替 xml。也有像 Import 这样的东西，可以包含其他配置类。

## 代理模式

Spring 必问题 - SpringAOP 和 SpringMVC

代理模式分类

* 静态代理
* 动态代理

### 静态代理

角色分析

* 抽象角色：一般是接口或抽象类
* 真实角色：被代理的角色
* 代理角色：代理真实角色，代理后做一些操作
* 客户：访问代理对象的人

优点：

* 使真实对象操作更纯粹，不用去关注公共业务
* 公共业务交给代理，业务分工
* 公共业务扩展方便

缺点： 一个真实角色产生一个代理角色，代码量翻倍

### 动态代理

* 动态代理和静态代理角色一样
* 动态代理的代理类使动态生成，不是直接写好的
* 动态代理分两大类：基于接口的动态代理/基于类的动态代理
  * 接口 - JDK动态代理
  * 类 - cglib
  * java字节码 - javasist

两个类： Proxy / InvocationHandler

Proxy: 在 handler 中被调用，产生代理的实例

InvocationHandler: 自定义调用过程，返回执行结果

优点：静态的有点 + 一个动态代理类代理的使一个接口，一般对应一类业务

## AOP - 横向扩展功能

### 配置实现01

目标业务点：pointcut, 需要额外添加的附属动作：adviser(MethodBeforeAdvice/AfterReturningAdvice)

然后添加配置文件

```xml
<bean id="userService" class="com.jzheng.service.UserServiceImpl"/>
<bean id="log" class="com.jzheng.log.Log"/>
<bean id="afterLog" class="com.jzheng.log.AfterLog"/>

<!--config AOP -->
<aop:config>
    <!-- point cut -->
    <aop:pointcut id="pointcut" expression="execution(* com.jzheng.service.UserServiceImpl.*(..))"/>
    <!-- 执行环绕增强 -->
    <aop:advisor advice-ref="log" pointcut-ref="pointcut"/>
    <aop:advisor advice-ref="afterLog" pointcut-ref="pointcut"/>
</aop:config>
```

### 配置实现02

也可以用自定义类，使用更简单，但是功能比之前的弱，不能操作 Method 之类的属性

```java
public class DiyPointCut {
    public void before() {
        System.out.println("----------> before method");
    }

    public void after() {
        System.out.println("-----------> after method");
    }
}
```

```xml
<bean id="diy" class="com.jzheng.diy.DiyPointCut"/>
<aop:config>
    <!-- 自定义切面 ref 要引用的类-->
    <aop:aspect ref="diy">
        <!-- 切入点 -->
        <aop:pointcut id="point" expression="execution(* com.jzheng.service.UserServiceImpl.*(..))"/>
        <!-- 通知 -->
        <aop:before method="before" pointcut-ref="point"/>
        <aop:after method="after" pointcut-ref="point"/>
    </aop:aspect>
</aop:config>
```

### 注解实现

类添加 Aspect 注解， 在方法上添加注解， 方法注解中可以指定切点。执行顺序：环绕前 -> 方法前 -> 方法 -> 环绕后 -> 方法后

```java
@Aspect
public class AnnotationPointCut {
    @Before("execution(* com.jzheng.service.UserServiceImpl.*(..))")
    public void before() {
        System.out.println("-------------> before [Anno type]");
    }

    @After("execution(* com.jzheng.service.UserServiceImpl.*(..))")
    public void after() {
        System.out.println("-------------> after [Anno type]");
    }

    @Around("execution(* com.jzheng.service.UserServiceImpl.*(..))")
    public void around(ProceedingJoinPoint proceedingJoinPoint) throws Throwable {
        System.out.println("-------------> around before [Anno type]");
        System.out.println("Signature: " + proceedingJoinPoint.getSignature());
        Object proceed = proceedingJoinPoint.proceed();
        System.out.println("-------------> around after [Anno type]");
    }
}
```

```xml
<!-- 方式3， 注解方式 -->
<bean id="annotationPointCut" class="com.jzheng.diy.AnnotationPointCut"/>
<!-- 开启注解支持 -->
<aop:aspectj-autoproxy/>
```

## 整合 Mybatis

1. 导入包
   1. junit
   2. mybatis
   3. mysql数据库
   4. spring相关的jar
   5. aop织入
   6. mybatis-spring [new]
2. 编写配置文件
3. 测试

## Mybatis

学到第 23 课，跳出去先把 Mybatis 看完再回来。。。。

1. 导包
   * junit
   * mybatis
   * mysql
   * spring
   * aop织入
   * mybatis-spring
2. 配置文件
3. 写测试

### mybatis 回忆

1. 编写实体类
2. 编写核心配置文件
3. 编写接口
4. 编写 Mapper.xml
5. 测试

### mybatis-spring

1. 编写数据源配置
2. sqlSessionFactory
3. sqlSessionTemplate

## 事务回顾

* 要么都成功，要么都失败
* 十分重要，涉及一致性，不能马虎
* 确保完整性和一致性

事务 ACID 原则：

* 原子性
* 一致性
* 隔离性，多个业务可能操作一个资源，防止数据损坏
* 持久性，十五一旦提交，无论系统发生什么问题，结果都不会被影响，被持久化的写到存储器中

### spring 中的事务管理

* 声明式事务 - AOP
* 编程式事务 - 需要在代码中进行事务管理，侵入性太强，不推荐

为什么需要事务：

1. 不添加事务管理，可能存在事务提交不一致
2. 如果不在 spring 中配置声明式事务，我们就需要在代码中手动配置事务
3. 在项目的开发中十分重要，涉及到数据的一致性