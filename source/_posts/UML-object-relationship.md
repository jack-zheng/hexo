---
title: UML 常见关系表示
date: 2020-10-13 17:39:19
categories:
- 编程
tags:
- UML
---

例举 UML 图中常见的关系及其表示方式

## 泛化 Generalization

对应 Java 中的继承，实线 + 实心三角指向父类

{% asset_img Generalization.png Generalization 关系图 %}

## 实现 Realization

对应 Java 中的实现，虚线 + 空心三角指向接口

{% asset_img Realization.png Realization 关系图 %}

## 关联 Association

对应 Java 中的成员变量，拥有关系，使一个类知道另一个类的属性和方法，可单向可双向。实心线 + 普通箭头指向被拥有者

{% asset_img Association.png Association 关系图 %}

## 聚合 Aggregation

整体与部分的关系，比如车和轮胎。他是一种强关联关系。空心菱形指向整体 + 实线 + 普通箭头指向部分

{% asset_img Aggregation.png Aggregation 关系图 %}

## 组合 Composition

整体与部分的关系，程度比聚合还要强的关联关系。实心菱形指向整体 + 实线 + 普通箭头指向部分

{% asset_img Composition.png Composition 关系图 %}

## 依赖 Dependency

对应 Java 中的局部变量，方法参数和静态方法调用，是一种使用的关系,所以要尽量不使用双向的互相依赖。虚线 + 普通箭头指向被使用者

{% asset_img Dependency.png Dependency 关系图 %}

## 参考文档

* [CSDN](https://blog.csdn.net/tianhai110/article/details/6339565)