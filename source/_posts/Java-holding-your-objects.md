---
title: Java holding your objects
date: 2020-12-22 19:55:40
categories:
- 编程
tags:
- TIJ4
---

- [前述](#前述)
- [Generics and type-safe containers](#generics-and-type-safe-containers)
- [Basic concepts](#basic-concepts)
- [Adding groups of elements](#adding-groups-of-elements)
- [Printing containers](#printing-containers)
- [List](#list)
- [Iterator](#iterator)

遇到 Iterator 相关的问题，重新看一遍 Holding Your Objects 章节

想要解决的问题：

- [ ] 这个章节具体讲了什么东西
- [ ] iterable/iterator/forEach 之前的关系和区别

## 前述

如果程序中只包含长度一定的，生命周期可知的对象，那这个程序确实足够简单了。

Array 是持有对象的最高效的方式，但是长度限制死了。

Java 中使用 'collection classes' 来解决可变长容器的问题，因为 Collection 在 Java 中已经有对应的类了，所以这个概念又被叫做容器(Container)。

这章只是介绍基本用法，后面有一节 Containers in Depth 会深入介绍

## Generics and type-safe containers

## Basic concepts

## Adding groups of elements

## Printing containers

## List

有序的一个数据序列，在 Collection 的基础上添加了一些方法来达到在 list 中间插入，删除元素的效果。

* ArrayList: 注重随机读写，但是插入删除性能比较慢
* LinkedList: 注重顺序读写，插入删除很快，随机读写很慢，功能上比 ArrayList 多

## Iterator

容器设计出来的主要作用：持有对象

Iteractor 是集合中的一个轻量级对象，可以很方便的在容器类之间做兼容，常见用法：

1. 用 Collection 对象调用 iterator() 方法拿到 Iterator 对象，它已经可以为你返回第一个对象了。
2. 调用 next() 返回下一个对象
3. 查看是否有跟多的对象过 hasNext()
4. 调用 remove() 删除之前的使用的对象

这个章节虽然简单，但是例子都是在 Type Information 里面的，得先看这个，不然看的没什么头绪。