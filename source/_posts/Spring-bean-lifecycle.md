---
title: Spring bean 生命周期
date: 2022-10-14 11:15:29
categories:
- Spring
tags:
- lifecycle
---

IoC容器的两个主要阶段：容器启动阶段 + Bean实例化阶段

1. bean def load
2. post process bean definition - BeanFactoryPostProcessor
3. instantiate beans
4. populate property
5. before bean post processor
6. initializer
7. init-method
8. after bean post processor
9. ready to use bean
10. destory

如何记忆：

* 1-2 属于 def 部分
* 3-4 属于创建 bean
* 5-8 属于修改 bean, 根据扩展点不同所以有这么多接口，主旨都是一样的
* 9 容器中的状态
* 10 销毁对象的时候会执行，很少接触

## BeanFactoryPostProcessor - bfpp

在加载 def 的时候更具配置来修改 def 属性的值。比如可以在 xml 中通过使用 `${}` 的语法来指代配置文件中的内容。典型应用如 PropertyOverrideConfigurer 和 PropertySourcesPlaceholderConfigurer。如果是想改变 bean 实例属性，使用 BeanPostProcessor。

在 Spring doc 的这个章节出现了 eagerly 的描述，目的是说，这种 processor 是用来对 def, bean 做操作的，所以 container 找到后会立即加载，lazy-load 也不生效的。这是从 processor 的定义决定的。

如果使用 BeanFactory + bfpp 的方式，需要显示的注册 bfpp `cfg.postProcessBeanFactory(factory);` 不是很方便，所以一般都直接用 ApplicationContext 来操作，自动实现 processor 的注册

ApplicationContext 中的 refresh() 方法中的

* obtainFreshBeanFactory() 方法包含加载 bean definition 的步骤
* invokeBeanFactoryPostProcessors() 方法就是处理 bfpp 的地方

## 对象实例化

refresh() 的 finishBeanFactoryInitialization() 方法负责实现 bean 的实例化，具体逻辑托管给 doGetBean() 方法。

* AbstractAutowireCapableBeanFactory::createBeanInstance 负责创建空壳 bean
* AbstractAutowireCapableBeanFactory::populateBean 填充属性
* AbstractAutowireCapableBeanFactory::initializeBean 初始化，包括处理 processor, init, init-method
* AbstractAutowireCapableBeanFactory::registerDisposableBeanIfNecessary 注册 disposable 方法备用
* initializeBean::invokeAwareMethods, 部分 aware 接口注入(BeanName, ClassLoader, BeanFactory)
* initializeBean::applyBeanPostProcessorsBeforeInitialization 处理 processor before 部分
* initializeBean::invokeInitMethod，分两块
  * 第一部分对应 InitializingBean 接口的方法
  * 第二部分对应 init-method 的方法, 使得 bean 执行 init-method 方法作为初始化的一步，可以通过 xml 和 @Bean 配置
* initializeBean::applyBeanPostProcessorsAfterInitialization