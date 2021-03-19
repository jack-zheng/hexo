---
title: 变量赋值，跌进了坑中
date: 2021-03-19 21:24:05
categories:
- java
tags:
- bug
---

今天在修一个 feature 的时候，掉进了一个很初级的坑中而不自觉，debug 了好久才发现的，汗颜

## 问题简化

求解

```java
class Person {
    private Map<String , Integer> map;

    public Map<String, Integer> getMap() {
        return map;
    }
}

public class TestScope {
    public static void main(String[] args) {
        Person person = new Person();

        Map<String, Integer> tmp = person.getMap();
        System.out.println(tmp); // null
        System.out.println(person.getMap()); // null

        tmp = new HashMap<>();
        tmp.put("jack", 31);

        System.out.println(tmp); // {jack=31}
        System.out.println(person.getMap()); // null
    }
}
```

debug 的时候还一直纳闷，怎么 person 引用没有被赋值。。。作为对比看下面的例子应该就很清楚了

```java
class Person {
    private Map<String , Integer> map = new HashMap<>(); // 带初始化引用类型的

    public Map<String, Integer> getMap() {
        return map;
    }
}

public class TestScope {
    public static void main(String[] args) {
        Person person = new Person();

        Map<String, Integer> tmp = person.getMap();
        System.out.println(tmp); // {}
        System.out.println(person.getMap()); // {}

        tmp.put("jack", 31);

        System.out.println(tmp); // {jack=31}
        System.out.println(person.getMap()); // {jack=31}
    }
}
```

在特殊化一下

```java
public class TestScope {
    public static void main(String[] args) {
        Person person = new Person();

        Map<String, Integer> tmp = person.getMap();
        System.out.println(tmp); // {}
        System.out.println(person.getMap()); // {}

        tmp.put("jack", 31);
        tmp = new HashMap<>();
        tmp.put("jerry", 21);

        System.out.println(tmp); // {jack=21}
        System.out.println(person.getMap()); // {jack=31}
    }
}
```

总结一下就是，改变引用类型的值没什么问题，但是如果一开始是 null 的话它就不会随着一起改了