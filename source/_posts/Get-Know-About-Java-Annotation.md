---
title: Get Know About Java Annotation
date: 2020-08-05 17:45:41
categories:
- 编程
tags:
- java
- annotation
---

记录一下 Java 注解的学习过程

## 快速入门

设计一个测试案例，创建一个名为 Marked 的注解类，该注解可以添加在 method 上用来表示方法是否被标记过。在测试用力中遍历被标记的类并打印信息

测试类：

```java
public class Person {
    private String name;
    private int age;
    private boolean isSelected;

    // Getter/Setter methods

    @Marked
    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    @Marked(value = true)
    public int getAge() {
        return age;
    }
    // ...
}
```

自己创建的注解类：

```java
@Retention(RetentionPolicy.RUNTIME)
public @interface Marked {
    boolean value() default false;
}
```

测试用例：

```java
@Test
public void test_print_by_anno() {
    Method[] methods = Person.class.getDeclaredMethods();
    for (Method m : methods) {
        Marked myAnno = m.getAnnotation(Marked.class);
        if (myAnno != null) {
            System.out.println("Method: " + m.getName() + " has marked annotation.");
            System.out.println("Marked value: " + myAnno.value());
        } else {
            System.out.println("Method: " + m.getName() + " don't has marked annotation.");
        }
    }
}
```

终端打印：

```txt
Method: getName has marked annotation.
Marked value: false
Method: setName don't has marked annotation.
Method: getAge has marked annotation.
Marked value: true
Method: setAge don't has marked annotation.
Method: isSelected don't has marked annotation.
Method: setSelected don't has marked annotation.
```

从这个例子可以看出来，Annotation 都是处理 class level 的问题的，和类延伸出来的实例基本没关系了
