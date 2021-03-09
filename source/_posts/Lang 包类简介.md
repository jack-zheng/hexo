---
title: lang 包类简介
date: 2020-01-08 15:25:37
categories:
- java
tags:
- lang
- class
---
用人话解释我用过的 Class 类中的方法

```java
/**
* 食用方法：classA.isAssignableFrom(classB)
* 表达的意思：classB 是不是 classA 的子类/接口 或 本身
*/
public native boolean isAssignableFrom(Class<?> cls);

// Samples, all tests passed.
import org.testng.Assert;
import org.testng.annotations.Test;

public class TestIsAssignableFrom {
    @Test
    public void test_isAssignableFrom (){
        // 对自己使用，返回 true
        Assert.assertTrue(ClassA.class.isAssignableFrom(ClassA.class));
        // 父类对子类使用，返回 true
        Assert.assertTrue(ClassA.class.isAssignableFrom(ClassB.class));
        // 子类对父类使用，返回 false
        Assert.assertFalse(ClassB.class.isAssignableFrom(ClassA.class));
        // 父接口对自接口使用，返回 true
        Assert.assertTrue(InterfaceC.class.isAssignableFrom(InterfaceD.class));
        // 子接口对父接口使用，返回 false
        Assert.assertFalse(InterfaceD.class.isAssignableFrom(InterfaceC.class));
        // 接口对实现了自己的类使用，返回 true
        Assert.assertTrue(InterfaceC.class.isAssignableFrom(ClassB.class));
    }
}

class ClassA {}

class ClassB extends ClassA implements InterfaceC {}

interface InterfaceC {}

interface InterfaceD extends InterfaceC {}
```
