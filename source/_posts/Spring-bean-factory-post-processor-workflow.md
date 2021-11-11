---
title: Spring bean factory post processor workflow
date: 2021-10-27 13:50:47
categories:
- Spring
tags:
- BeanFactoryPostProcessor
---

Spring 中 BeanFactoryPostProcessor 相关代码的源码解析，涉及到的主要 class 整理如下

* BeanFactory: bean 工厂，生产单个 bean,  典型方法 getBean/containsBean/isSingleton/getType etc
* ListableBeanFactory: 继承自 BeanFactory，额外拥有枚举 bean 的能力，即批量返回
* HierarchicalBeanFactory: 继承自 BeanFactory，额外拥**父子**容器的概念
* AutowireCapableBeanFactory: 继承自 BeanFactory, 额外提供管理非自动装配 bean 的能力
* SingletonBeanRegistry: 保证单例的注册器
* ConfigurableBeanFactory: 继承自 HierarchicalBeanFactory + SingletonBeanRegistry, 额外的配置能力
* ConfigurableListableBeanFactory: 整合 listale, hierarch 和 autowire 的能力，额外提供分析修改 bean definition 的能力，还有 pre-instantiate singletons 的能力
* AliasRegistry: alias 管理接口，典型方法 registerAlias/removeAlias etc
* BeanDefinitionRegistry: 继承自 AliasRegistry，增加了持有 bean definition 的功能，是 Spring factory 包下唯一一个有这种能力的接口
* SimpleAliasRegistry: AliasRegistry 的简单实现**类**，内部通过 map 存储 name 和 alias 的对应关系
* DefaultSingletonBeanRegistry: 类实现，强调 registry 和 singleton 属性，不包含任何 bean definition 的概念
* FactoryBean: 不是普通的 bean, 这个 bean 的作用类似工厂，用来生产其他 bean 的. 通常用在 infrastructure code 中
* FactoryBeanRegistrySupport: FactoryBeanRegistrySupport 派生类, 专门处理 FactoryBean 的 instance
* AbstractBeanFactory: BeanFactory 抽象实现，并不具备 Listable 的功能，具备从 resource 中提取 bean definition 的功能，核心方法 getBeanDefinition
* AbstractAutowireCapableBeanFactory: 实现了默认的 bean 创建逻辑，没有 bean definition 注册能力