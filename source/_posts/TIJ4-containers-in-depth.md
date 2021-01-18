---
title: TIJ4 containers in depth
date: 2021-01-15 19:30:27
categories:
- 编程
tags:
- TIJ4
---

[ ] 结合 Filling containers 章节复习 Adapter 设计模式

## 前言

本章需要些许 Generic 章节的知识，所以看之前最好过一遍前章。

## Full container taxonomy 分类学

Java5 中新加的内容：

* Queue 接口以及对应的实现 PriorityQueue，BlockingQueue 将在 Concurrency 章节介绍
* ConcurrentMap 以及对应实现 ConcurrentHashMap 也放到 Concurrency
* CopyOnWriteArrayList， CopyOnWriteArraySet 同上
* EnumSet and EnumMap, special implementations of Set and Map for use with enums, and shown in the Enumerated Types chapter.
* Collectipns 中的一些单元方法

你可以看到这个整个集合体系中有几个类是以 Abstract 开头的，这些用虚线框包裹的类表示抽象类。他们实现了对应接口的部分功能，比如当你想要实现一个 Set 接口的时候, 你不会想要实现 Set 接口并且实现里面的所有方法。一般来说，我们会通过继承 AbstractSet 类做一个最小实现。但是说实话现在集合类基本上能满足你的需求了，一般不需要自己做扩展。

## Filling containers

类似于 Arrays 这个 util 类，针对集合类，也有一个 util 类叫做 Collections。它里面有一个 fill() 方法可以将一个对象复制填充满整个容器，执行结果返回一个 List, 我们可以将整个 list 传给其他构造函数或者调用 addAll() 方法。

```java
import java.util.ArrayList;
import java.util.Collections;
import java.util.List;

class StringAddress {
    private String s;

    public StringAddress(String s) {
        this.s = s;
    }

    public String toString() {
        return super.toString() + " " + s;
    }
}

public class FillingLists {
    public static void main(String[] args) {
        List<StringAddress> list = new ArrayList<>(
                Collections.nCopies(4, new StringAddress("Hello")));
        System.out.println(list);
        Collections.fill(list, new StringAddress("World!"));
        System.out.println(list);
    }
}

// output:
// [reading.container.StringAddress@7c53a9eb Hello, reading.container.StringAddress@7c53a9eb Hello, reading.container.StringAddress@7c53a9eb Hello, reading.container.StringAddress@7c53a9eb Hello]
// [reading.container.StringAddress@ed17bee World!, reading.container.StringAddress@ed17bee World!, reading.container.StringAddress@ed17bee World!, reading.container.StringAddress@ed17bee World!]
```

如上例所示，给出了两种方法填充集合，一种是调用 Collections.nCopies() 另一种是调用 Collections.fill()，第一种扩展性更好。第二种方式只支持替换，不能扩容。

## A Generator solution

Collection 子类都有一个接收其他 Collection 的构造器。为了实验方便，我们创建一个构造器，接收类型和数量做参数

```java
import java.util.ArrayList;

interface Generator<T> { T next(); }

public class CollectionData<T> extends ArrayList<T> {
    public CollectionData(Generator<T> gen, int quantity) {
        for (int i = 0; i < quantity; i++)
            add(gen.next());
    }

    // A generic convenience method:
    public static <T> CollectionData<T> list(Generator<T> gen, int quantity) {
        return new CollectionData<>(gen, quantity);
    }
}
```

通过上面的代码我们可以构造一个任意容量的容器，创建的对象也可以作为参数传给任意 Collection 子类。同时结合自带的 addAll() 方法也可以用于填充容器。

CollectionData is an example of the Adapter design pattern;1 it adapts a Generator to the constructor for a Collection.

Here’s an example that initializes a LinkedHashSet: 

```java
import java.util.LinkedHashSet;
import java.util.Set;

class Government implements Generator<String> {
    String[] foundation = ("strange women lying in ponds " +
            "distributing swords is no basis for a system of " +
            "government").split(" ");
    private int index;

    public String next() {
        return foundation[index++];
    }
}

public class CollectionDataTest {
    public static void main(String[] args) {
        Set<String> set = new LinkedHashSet<>(
                new CollectionData<>(new Government(), 15));
        // Using the convenience method:
        set.addAll(CollectionData.list(new Government(), 15));
        System.out.println(set);
    }
}

// output:
// [strange, women, lying, in, ponds, distributing, swords, is, no, basis, for, a, system, of, government]
```

由于 LinkedHashSet 的关系，所以容器中元素的顺序不变。

TODO: 后面的例子需要 Arrays 章节的内容，没看过，直接继续感觉很不爽，我回头看看先。Arrays 只有 30+ pages 应该挺快的