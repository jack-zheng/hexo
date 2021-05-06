---
title: Java getEnclosingClass in Class
date: 2021-05-06 18:45:35
categories:
- java
tags:
- bugs
---

今天在查找某个 bug 的 root cause 的时候，发现一个 Class.getEnclosingClass() 的调用。从来没有用个这玩意儿，做下笔记。

## 定义

人话就是：返回这个类的外部类。

```java
/**
 * Returns the immediately enclosing class of the underlying
 * class.  If the underlying class is a top level class this
 * method returns {@code null}.
 * @return the immediately enclosing class of the underlying class
 * @exception  SecurityException
 *             If a security manager, <i>s</i>, is present and the caller's
 *             class loader is not the same as or an ancestor of the class
 *             loader for the enclosing class and invocation of {@link
 *             SecurityManager#checkPackageAccess s.checkPackageAccess()}
 *             denies access to the package of the enclosing class
 * @since 1.5
 */
@CallerSensitive
public Class<?> getEnclosingClass() throws SecurityException
```

## 案件重现

以前有一段 code 组织形式如下，定义了一个 MyOuterClass，它有一个 field 动态实现了 MyAbsClass 这个抽象类。那么，当我们调用 MyOuterClass 实例的 `field.getClass().getEcloseingClass()` 的时候会返回外部类的 class name

```java
public class Test {
    public static void main(String[] args) {
        System.out.println((new MyOuterClass()).field.getClass().getEnclosingClass().getSimpleName());
    }
}

abstract class MyAbsClass {}

class MyOuterClass {
    MyAbsClass field = new MyAbsClass() {};
}
// MyOuterClass
```

然后有个德国的 team 将这段代码重构了，形式如下。他们新建了一个类，继承了抽象类，并将原先的类成员变量声明的地方用这个新建类代替了。这种情况下，getEnclosingClass() 的调用方就变为一个 top level class 了，返回 null，随之导致 getSimpleName() 抛出 NPE 了。

```java
public class Test {
    public static void main(String[] args) {
        System.out.println((new MyOuterClass()).field.getClass().getEnclosingClass().getSimpleName());
    }
}

abstract class MyAbsClass {}

class MyAbsClassIns extends MyAbsClass {}

class MyOuterClass {
    MyAbsClass field = new MyAbsClassIns();
}
// Exception in thread "main" java.lang.NullPointerException
// 	at reading.container.Test.main(Test.java:5)
```

## 其他的使用测试

```java
public class Test {
    public static void main(String[] args) {
        new Person();
    }
}

class Person {
    {
        System.out.println("log when init person: " + getClass().getEnclosingClass());
        new Outer();
    }

    class Outer {
        {
            System.out.println("log when init Outer: " + getClass().getEnclosingClass());
            new Inner();
        }

        class Inner {
            {
                System.out.println("log when init Inner: " + getClass().getEnclosingClass());
            }
        }
    }
}
// log when init person: null
// log when init Outer: class reading.container.Person
// log when init Inner: class reading.container.Person$Outer
```