---
title: Java syntax track
date: 2021-11-24 15:52:01
categories:
- Java
tags:
- 语法
---

## 如果是 null 返回默认值

```java
System.out.println(Optional.ofNullable("a").orElse("b")); // a
System.out.println(Optional.ofNullable(null).orElse("b")); // b

// 如果是 JDK9 还可以使用 Objects
Objects.requireNonNullElse(T obj, T defaultObj);
```
