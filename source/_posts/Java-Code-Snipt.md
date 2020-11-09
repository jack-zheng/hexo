---
title: Java 代码块记录
date: 2020-08-10 15:19:26
categories:
- 编程
tags:
- java
- 代码段
---

记录平时遇到的一些精巧的小代码段

## 创建元组

摘自 On Java 8 泛型章节。元组的定义：用户只能取值而不能设置值，所以这里没有使用 getter/setter 的封装形式，而是使用 public + final 关键字实现了该功能。

```java
public class Tuple2<A, B> {
    public final A a1;
    public final B a2;
    public Tuple2(A a, B b) { a1 = a; a2 = b; }
    public String rep() { return a1 + ", " + a2; }

    @Override
    public String toString() {
        return "(" + rep() + ")";
    }
}
```

## lambda 实现 interface

```java
public class TestLambda {
    private MyPrint print;

    @Test
    public void test() {
        print = System.out::println;
        print.print("jack");
    }
}

interface MyPrint {
    void print(String name);
}
// output: jack
```