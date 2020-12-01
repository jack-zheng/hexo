---
title: Java methods equals and hashcode
date: 2020-12-01 17:25:15
categories:
- 编程
tags:
- java
---

刚好最近遇到需要重写 equals 和 hashcode 的情况，总结记录一下，加深印象。

## 官方对 hashCode 方法的描述

```txt
Returns a hash code value for the object. This method is supported for the benefit of hash tables such as those provided by HashMap.

The general contract of hashCode is:

1. Whenever it is invoked on the same object more than once during an execution of a Java application, the hashCode method must consistently return the same integer, provided no information used in equals comparisons on the object is modified. This integer need not remain consistent from one execution of an application to another execution of the same application.
2. If two objects are equal according to the equals(Object) method, then calling the hashCode method on each of the two objects must produce the same integer result.
3. It is not required that if two objects are unequal according to the equals(java.lang.Object) method, then calling the hashCode method on each of the two objects must produce distinct integer results. However, the programmer should be aware that producing distinct integer results for unequal objects may improve the performance of hash tables.

As much as is reasonably practical, the hashCode method defined by class Object does return distinct integers for distinct objects. (This is typically implemented by converting the internal address of the object into an integer, but this implementation technique is not required by the JavaTM programming language.)
```

概括起来就是 equals 相等的两个对象 hashcode 必须相同，反过来，hashcode 相等的像个对象，equals 可以不想等。

hashcode 是需要结合集合类才能体现出来的。试想一下，如果没有 hashcode, 那么我们在一个存了 1000 个对象的 HashSet 中添加一个新的对象就要进行 1000 次的 equals 比较，这样的性能消耗无疑是巨大的。所以 HashXXX 的数据结构引入 hash 算法来简化比较。

hash 的数据结构中是允许存在 hash 值相同的对象的，这种情况下，他会在 hash 的地址位置创建一个链表存储 hash 值相同的对象。

当集合存入一个对象时，他会先根据 hash 值判断是否有重复的元素， 如果 hash 值已经存在，那么他会找到对应的链表然后一次进行对象的 equals 判断重复。好的 hash 算法要尽量减少 hash 冲突来提高检索效率。