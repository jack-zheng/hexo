---
title: 工厂模式
date: 2021-05-08 17:14:36
categories:
- 设计模式
- HFDP 
tags:
- factory pattern
- 工厂模式
---

* 简单工厂并不是一种设计模式，而是一种良好的编码习惯

> **The Factory Method Pattern** defines an interface for creating an object, but lets subclasses decide which class to instantiate. Factory Method lets a class defer instantiation to subclasses.
> 工厂方法 模式，这个名字可以说是很形象了，哈哈。它定义一个方法来创建对象，并把创建对象的逻辑托付给子类。

简单工厂，工厂只是一个其他的对象，而 工厂方法 这个设计模式中，工厂是一个子类

SimpleFactory, which gives you a way to encapsulate object creation, but not give you the flexibility of the Factory Method