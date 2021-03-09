---
title: Java HashMap things
date: 2021-03-09 15:16:54
categories:
- java
tags:
- collection
- HashMap
---

今天再看一个 defect 的时候涉及到 HashMap 存储的问题，回头想一下发现自己只对 HashMap 以 key 的 Hash 作为依存储依据这点比较清楚外，其他的印象很模糊，写这篇文章记录一下。想要了解的问题如下：

1. HashMap 再存储时是否只用 key 的 hash 做依据，和 value 有关系吗
2. HashMap 底层使用什么数据结构存储的
3. HashMap 的类继承关系

以后对 HashMap 的知识点都可以考虑在这篇中做扩展，做成一个总集篇