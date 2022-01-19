---
title: Spring XML 转化为 bean definition
date: 2022-01-04 15:04:23
categories:
- Spring
tags:
- IoC
---

本文主要 focus 在 xml 到 bean definition 转化的过程，即 AbstractApplicationContext 的 refresh 方法的 obtainFreshBeanFactory()。转化过程是这个方法的一部分。本文会先介绍一些这个过程中用到的一些底层类实现，最后再总的将这部分内容捋一遍。

## Document

## BeanDefinitionHolder

BeanDefinitionHolder 是解析 xml 时的一个中间类，不对外暴露。只包含了三个属性 beanDefinition, beanName 和 alias。代表一个被解析的 bean 节点。

