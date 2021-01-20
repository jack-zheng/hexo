---
title: TIJ4 数组 Arrays
date: 2021-01-20 19:45:19
categories:
- TIJ4
tags:
- arrays
---

Arrays 章节读书笔记

## 前述

这个章节将深入讲解 Arrays 的使用

## Why arrays are special

array 持有对象的特点：高效，可指定类型，可以存储原始类型的数据

* 数组是最高效的存储结构，代价是容量固定，不可变
* ArrayList 可以自动扩容，但是每次扩容都需要重新拷贝引用
* 泛型没出来前，Array 有类型检查的优势
* 数组可以装载原始数据类型，容器则需要通过自动开箱，装箱实现

以下示例对比数组和容器

```java
import java.util.ArrayList;
import java.util.Arrays;
import java.util.List;


class BerylliumSphere {
    private static long counter;
    private final long id = counter++;

    public String toString() { return "Sphere " + id; }
}

public class ContainerComparison {
    public static void main(String[] args) {
        BerylliumSphere[] spheres = new BerylliumSphere[10];
        for (int i = 0; i < 5; i++)
            spheres[i] = new BerylliumSphere();
        System.out.println(Arrays.toString(spheres));
        System.out.println(spheres[4]);

        List<BerylliumSphere> sphereList = new ArrayList<>();
        for (int i = 0; i < 5; i++)
            sphereList.add(new BerylliumSphere());
        System.out.println(sphereList);
        System.out.println(sphereList.get(4));

        int[] integers = {0, 1, 2, 3, 4, 5};
        System.out.println(Arrays.toString(integers));
        System.out.println(integers[4]);

        List<Integer> intList = new ArrayList<>(Arrays.asList(0, 1, 2, 3, 4, 5));
        intList.add(97);
        System.out.println(intList);
        System.out.println(intList.get(4));
    }
}
```

两种方式都会做类型检测，唯一的不同体现在，使用数组时我们通过 `[ ]` 访问元素，使用容器时我们通过 `add( )` 和 `get( )` 来访问。两者很类似是有意为之，为了减少两者之间 migration 的effort。

容器支持的功能更多，现在使用数组的唯一理由就是**快**。但是当你需要应付一些复杂的情况时，数组的这种严格限制就会制约你，你可能需要用容器来代替它。
