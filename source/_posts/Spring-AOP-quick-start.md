---
title: Spring AOP quick start
date: 2022-06-27 14:45:45
categories:
- Spring
tags:
- AOP
---

快速了解 AOP 必要只是并写出 demo。这篇文上是在我系统学了 AOP 的基本使用之后再写的，之前遇到的很多痛点都体现不出来了。花了挺多时间读官方文档的时候，很多问题都迎刃而解了，就是比较花时间。之前失败的主要原因是 pointcut 的表达式写错了，tomcat 起了，但是服务都失败了。。。。

## 简单概括什么是 AOP

不破坏代码结构的情况下，为代码添加功能，典型案例如打印方法的执行时间。

## AOP 涉及的专有名词

* Joinpoint - 要匹配的方法
* Pointcut - 匹配方法的规则
* Advice - 检测到匹配方法的时候要执行的 额外 逻辑
* Aspect - 带 @Aspect 的那个类

## 怎么写

如果 Spring 整体环境已经搭建完成，如果你想要新建一个 Aspect 只需要做两件事

1. 写 Aspect 文件，最简单的方式是，写 Advice 方法，并在注解中添加 pointcut 表达式
1. 注册 Aspect, 不过如果是已经运行的系统，这步应该不用做，都是自动扫描的
   1. Xml
   2. Annotation

```java
@Aspect
@Component
public class IssueAspect {
    @Before("within(issues.*)")
    public void doBeforeLog(JoinPoint joinPoint) {
        String methodName = joinPoint.getSignature().getName();
        System.out.printf("Before Advice of method %s is called...%n", methodName);
    }
}
```