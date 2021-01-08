---
title: 记录一些平时发现的好的编程习惯
date: 2021-01-08 19:07:16
categories:
- 编程
tags:
- 习惯
---

## 更抽象的命名

今天在写一个 Java bean 的时候，为了 merge bean 的属性，特意给这个 bean 写了一个 mergeUpdatedProperties(Properties props) 方法，同时 review 之后提出，直接用 merg(Bean bean) 的方式会更有扩展性，深以为然。

