---
title: Java 拾遗之 final 关键字
date: 2020-10-22 16:23:35
categories:
- 编程
tags:
- 拾遗
- 泛型
---

从定义上来说 final 表达的是只能赋值一次，赋之后不能改变的意思。下面介绍几种常见使用形式。

## final + attr

这种形式即实体类有 final 变量

```java
public class Test {
    private final String name;

    public Test() {
        this.name = "default";
    }

    public Test(String name) {
        this.name = name;
    }

    // public Test(int i) {} -- compile fail
}
```

由于 name 由 final 修饰，所以要求每个构造函数都要有 name 的初始化，或者声明时直接赋值。不然不就等于允许在运行时改变 name 值了

## statuc + final + attr

这种形式常见于 class 属性，声明时就得赋值，static block 都不好使

```java
class Test2 {
    private static final String name = "Jack";
}
```

## final + method

> `final` is used with a Java method to mark that the method can't be overridden (for object scope) or hidden (for static). This allows the original developer to create functionality that cannot be changed by subclasses, and that is all the guarantee it provides.
> 
> 方法不能被重写，防止继承之后方法语意发生变化

```java
class Test3 {
    public static final void printT3 () {
        System.out.println("printT3...");
    }
}

class Sub3 extends Test3 {
    @Override // compile error, can't override final method
    public final void printT3 () {
        System.out.println("sub3...");
    }
}
```
