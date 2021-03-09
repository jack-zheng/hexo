---
title: Java 集合类初探
date: 2021-01-14 19:17:49
categories:
- java
tags:
- collection
---

想要解决的问题：

1. 了解集合类的大致情况，包括名称，类关系
2. HashMap 和 Collection 的关系
3. 自己画一个关系图并和 TIJ4 做对比
4. 为什么 ArrayList 在继承了 AbstractList 之后还要 impl List 接口？意义上不是重复了吗

Answers:

4. 这样做，语义上没有改变，便于阅读，省的你再去一层层的去父类找接口实现， Stackoverflow 上是这么说的

基本上能将这一簇类的关系图画出来即可


