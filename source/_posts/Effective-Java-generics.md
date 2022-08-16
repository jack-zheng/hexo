---
title: Effective Java 异常
date: 2022-08-05 10:44:32
categories:
- Effective Java
tags:
- 泛型
---

章节摘抄

* 新写的代码中有泛型支持的要加上检测，别去掉。e.g. List<String> 而不是直接 List list=...
* 消除非受检警告
* 列表优先于数组
* 优先考虑泛型 - 这个建议中给出的 Stack 例子，属实没有 get 到他要讲的点

泛型那段读不下去了，很多概念性的东西读起来很吃力，需要再重新看看泛型相关的知识点了，应该可以结合 Collection 来看效果会比较好。

## 请不要在新代码中使用原生态类型

原生态类型指的是原本可以使用泛型的地方，没有指定，比如 List<String> 写出 List list =... 的情况

泛型给了你在编译器检测类型匹配的能力

List<String> 是 List 的子类，而不是 List<Object> 的子类，语法上是这么规定的，挺神奇。

现在还能使用原生态类型是为了向下兼容，Java 1.5 才出的泛型

无限制通配符类型，比如 Set<E> 的无限制形式 Set<?> 读作 '某个类型的集合'

Java 1.5 之后，只有两种情况允许使用原生态类型，一种是类 class 操作的时候，比如 List.class, String[].class, int.class。另一种是与 instanceof 相关的操作，比如 `if(o instance of Set)`

## 消除非受检警告

使用泛型是会碰到很多 unchecked warning，把他们都修复，这个可以保证代码运行时不会出现 ClassCastException.

## 列表优先于数组

数组是协变的(covariant)，如果 sub 是 super 的子类，那么 sub[] 就是 super[] 的子类。

泛型是不可变的(invariant), 如果 Type1 是 Type2 的子类，List<Type1> 和 List<Type2> 没有继承关系。

作则认为上面的这种特性是数组的缺点。举个例子，下面两种变量都不能将 String 放入变量中，第一种要运行时才能发现，第二种编译期就抛错了。

```java
Object[] a = new Long[1];
a[0]="I don't fit it"; // 可以添加，运行时出错

List<Object> b = new ArrayList<Long>(); // 编译时就会抛错
b.add("I don't fit it");
```

数组是具体化的，泛型则通过擦除机制保证代码的通用性。应为数组的类型不安全的，所以不能创建泛型数组，这种做法和泛型的定义相违背。

像 E, List<E>, List<String> 这样的类型称作不可具体化的(non-reifiable)类型，是运行时表示法包含的信息比编译时表示法包含的信息更好的类型。唯一可以具体化的参数化类型是无限制通配符，但是不常用

当遇到泛型数组创建错误时，最好的方案是用集合类型List<E> 代替数组类型 E[], 这样可能会损失一些性能或者简洁性，但是却有更高的安全性和通用性。

一般来说，数组和泛型不能混用，如果混合使用时出现编译错误或警告，第一反应因该是用列表代替数组。