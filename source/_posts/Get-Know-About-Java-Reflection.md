---
title: 初识 Java 反射
date: 2020-07-31 17:35:17
categories:
- 编程
tags:
- java
- 反射
- reflection
---

记录一些 Java 反射基础知识

## 准备测试 Bean

```java
package reflectiontest.bean;

public class TestUser {
  private String name;
  private int age;
  public  String gender;

  // Getter and Setter
}
```

## getFields VS getDeclaredFields

getFields 只会返回 public 类型的 fields, getDeclaredFields 会返回所有类型的 fieds

```java
@Test
public void get_class_field() {
Field[] fields = TestUser.class.getFields();
System.out.println("Output of getFields...");
for (Field f : fields) {
    System.out.println(f);
}
System.out.println("\n");

Field[] declareFields = TestUser.class.getDeclaredFields();
System.out.println("Output of getDeclaredFields...");
for (Field f : declareFields) {
    System.out.println(f);
}
}

// Output of getFields...
// public java.lang.String reflectiontest.bean.TestUser.gender


// Output of getDeclaredFields...
// private java.lang.String reflectiontest.bean.TestUser.name
// private int reflectiontest.bean.TestUser.age
// public java.lang.String reflectiontest.bean.TestUser.gender
```
