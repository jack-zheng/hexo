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

## Arrays are first-class objects

抛开数组的类型不说，数组修饰符是指向对内对象的引用。它存储了其他对象的引用，你可以通过数组初始化语句隐式创建它，也可以通过 new 的方式显示的创建。整个数组对象基本上只提供一个只读属性(length)给你使用，你还可以通过 `[ ]` 语法访问数组元素。

下面的例子展示了初始化数组的各种方式，和各种赋值方法。存储原始类型和对象类型的数据基本上一致的，唯一的不同是，如果存储的是对象，那么数组持有的是对象的引用。

```java
import java.util.Arrays;

class BerylliumSphere {
    private static long counter;
    private final long id = counter++;

    public String toString() {
        return "Sphere " + id;
    }
}

public class ArrayOptions {
    public static void main(String[] args) {
        // Arrays of objects:
        BerylliumSphere[] a; // Local uninitialized variable
        BerylliumSphere[] b = new BerylliumSphere[5];
        // The references inside the array are
        // automatically initialized to null:
        System.out.println("b: " + Arrays.toString(b));
        BerylliumSphere[] c = new BerylliumSphere[4];
        for (int i = 0; i < c.length; i++)
            if (c[i] == null) // Can test for null reference
                c[i] = new BerylliumSphere();
        // Aggregate initialization:
        BerylliumSphere[] d = {new BerylliumSphere(),
                new BerylliumSphere(), new BerylliumSphere()
        };
        // Dynamic aggregate initialization:
        a = new BerylliumSphere[]{
                new BerylliumSphere(), new BerylliumSphere(),
        };
        // (Trailing comma is optional in both cases)
        System.out.println("a.length = " + a.length);
        System.out.println("b.length = " + b.length);
        System.out.println("c.length = " + c.length);
        System.out.println("d.length = " + d.length);
        a = d;
        System.out.println("a.length = " + a.length);
        // Arrays of primitives:
        int[] e; // Null reference
        int[] f = new int[5];
        // The primitives inside the array are
        // automatically initialized to zero:
        System.out.println("f: " + Arrays.toString(f));
        int[] g = new int[4];
        for (int i = 0; i < g.length; i++)
            g[i] = i * i;
        int[] h = {11, 47, 93};
        // Compile error: variable e not initialized:
        //!System.out.println("e.length = " + e.length);
        System.out.println("f.length = " + f.length);
        System.out.println("g.length = " + g.length);
        System.out.println("h.length = " + h.length);
        e = h;
        System.out.println("e.length = " + e.length);
        e = new int[]{1, 2};
        System.out.println("e.length = " + e.length);
    }
}

// output
// b: [null, null, null, null, null]
// a.length = 2
// b.length = 5
// c.length = 4
// d.length = 3
// a.length = 3
// f: [0, 0, 0, 0, 0]
// f.length = 5
// g.length = 4
// h.length = 3
// e.length = 3
// e.length = 2
```

* 数组 a 未初始化，编译器在你赋值之前会阻止你做任何操作
* 