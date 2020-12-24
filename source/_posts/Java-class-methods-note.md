---
title: Class 这个类中的方法使用记录
date: 2020-12-24 16:37:33
categories:
- 编程
tags:
- java
- 语法
---

最近在使用 Class 这个类的时候遇到一些问题，顺便记录一下这个类中方法的使用案例

## isAssignFrom

简单来说就是测试传入的 Class 是不是前面的 Class 本身或子类, 同时适用于接口实现的情况。

```java
System.out.println("Number isAssignableFrom Number.class: " + Number.class.isAssignableFrom(Number.class));
System.out.println("Number isAssignableFrom Integer.class: " + Number.class.isAssignableFrom(Integer.class));
System.out.println("Integer.class isAssignableFrom Number: " + Integer.class.isAssignableFrom(Number.class));
System.out.println("Collection.class isAssignableFrom ArrayList.class: " + Collection.class.isAssignableFrom(ArrayList.class));

// output:
// Number isAssignableFrom Number.class: true
// Number isAssignableFrom Integer.class: true
// Integer.class isAssignableFrom Number: false
// Collection.class isAssignableFrom ArrayList.class: true
```