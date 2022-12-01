---
title: Spring AOP 实现细节
date: 2022-10-28 14:00:50
categories:
- Spring
tags:
- AOP
---

* Join Point: In Spring AOP, a join point always represents a method execution. 事件触发点
* Advice: Action taken by an aspect at a particular join point. 遇到 join point 时会执行的动作，行为
* Pointcut: A predicate that matches join points. 匹配 method 的表达式
* Advisor: an Advisor is an aspect that contains only a single advice object associated with a pointcut expression. 只包含一个 advice 的 aspect
* Advised: However you create AOP proxies, you can manipulate them BY using the advised interface. 代理对象的配置信息

## AOP 模型

AOP 有两种注册方式 Xml 或者 Annotation, 做这两种设置时，底层其实就是注册了一个特殊的 BeanPostProcessor(DefaultAdvisorAutoProxyCreator/AnnotationAwareAspectJAutoProxyCreator)，剩下的气势和普通的 getBean() 流程一样，到处理 BeanPostProcessor 时，上面的说的两个 processor 会判断当前 bean 是不是需要代理，如果要的话，通过 ProxyFactory 创建代理并返回。