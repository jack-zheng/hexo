---
title: Design-pattern-singleton.md
date: 2021-08-11 13:31:03
categories:
- 设计模式 
tags:
- Design Pattern
---

目的：保证一个类只有一个实例，并提供一个访问他的全局访问点

关键代码：构造函数私有化

介绍几种单例模式的实现方式 - 有种回字的几种写法的意思，略无聊

## 懒汉式 - 线程不安全

```java
public class Singleton01 {
    private static Singleton01 instance;
    private Singleton01(){}

    public static Singleton01 getInstance() {
        if (instance == null) {
            instance = new Singleton01();
        }
        return instance;
    }
}
```

简单易懂，但是线程不安全

## 懒汉式 - 线程安全

```java
public class Singleton02 {
    private static Singleton02 instance;
    private Singleton02() {}

    public static synchronized Singleton02 getInstance() {
        if (instance == null) {
            instance = new Singleton02();
        }
        return instance;
    }
}
```

实现简单，线程安全，方法体加锁比较耗资源，当 getInstance() 调用不频繁时可以使用

## 饿汉式 - 线程安全

```java
public class Singleton03 {
    private static Singleton03 instance = new Singleton03();
    private Singleton03() {}

    public static Singleton03 getInstance() {
        return instance;
    }
}
```

实现简单，线程安全，通过 classloader 避免同步问题，缺点是类加载就初始化，浪费内存

## 双检锁/双重校验锁 DCL double-checked locking

```java
public class Singleton04 {
    private volatile static Singleton04 instance;
    private Singleton04() {}

    public static Singleton04 getInstance() {
        if (instance == null) {
            synchronized (Singleton04.class) {
                if (instance == null) {
                    instance = new Singleton04();
                }
            }
        }
        return instance;
    }
}
```

略显复杂，但是性能良好效率高，懒加载，线程安全 Java 1.5 后有效。

## 登记式/静态内部类

```java
public class Singleton05 {
    private static class SingletonHolder {
        private static final Singleton05 INSTANCE = new Singleton05();
    }
    private Singleton05() {}

    public static Singleton05 getInstance() {
        return SingletonHolder.INSTANCE;
    }
}
```

实现简单，线程安全，懒加载。效果和双检锁一致。但是由于只在被使用时才通过 classloader 加载，效率回更高

## 枚举

```java
public enum Singleton06 {
    INSTANCE;
    public void whateverMethod() {}
}
```

理论上来说最安全，最简单的实现，不过不流行。 Effective Java 作者推荐的写法