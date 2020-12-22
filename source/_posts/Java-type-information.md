---
title: Java type information
date: 2020-12-22 22:18:06
categories:
- 编程
tags:
- TIJ4
---

## 前述

Runtime type information(RTTI) allows you to discover and use type information while a program is running.

两种使用方式：

1. 传统模式，假定你在编译期就知道所有用到的类型
2. 反射模式，你只在运行时才知道类信息

## The need for RTTI

简单的继承关系示例：

基类：Shape 包含方法 draw()， 子类：Circle, Square, Triangle

```java
import java.util.Arrays;
import java.util.List;

abstract class Shape {
    void draw() {
        System.out.println(this + ".draw()");
    }

    abstract public String toString();
}

class Circle extends Shape {
    public String toString() {
        return "Circle";
    }
}

class Square extends Shape {
    public String toString() {
        return "Square";
    }
}

class Triangle extends Shape {
    public String toString() {
        return "Triangle";
    }
}

public class Shapes {
    public static void main(String[] args) {
        List<Shape> shapeList = Arrays.asList(new Circle(), new Square(), new Triangle());
        for (Shape shape : shapeList) shape.draw();
    }
}
// output:
// Circle.draw()
// Square.draw()
// Triangle.draw()
```

在上面的例子中，我本将子类结合 List 强转成父类，然后统一做操作，这种做法更易读，容易维护。这也是面向对象的目标之一，但是如果我想在运行时得知这个对象的具体类型，应该怎么做？

## The Class object

在 Java 中有一个神奇的类他叫 Class 类，所有创建类实例的行为都和他有关。 Java 的 RTTI 特性也是通过它来实现的。当你编译一个类的时候，JVM 会通过 class 创建一个对应的 Class 类来存储对应的信息。

类加载器可以有一组 class loaders 组成，但是已有一个 primordial class loader，他时 JVM 的一部分，他也被叫做 trusted classes。通常你不需要自己新家 class loader 但是如果由特殊需要，想加也是可以的。

只有当第一次使用的时候，JVM 才会加载对应的 class。这个行为发生在类第一次关联到 static 实体， 构造函数也是一个特殊的 static method，换句话说，当我们 new 一个对象的时候，加载器就会加载对应的 class。

Java 中只允许 Class 加载一次，加载完成之后，以后所有这个 class 对应的实体都是通过它来创建的。

PS：这里翻译很生硬，缺少很多类加载的相关知识，可以看过 JVM 那本书之后，再来完善一下。

```java
class Candy {
    static {
        System.out.println("Loading Candy");
    }
}

class Gum {
    static {
        System.out.println("Loading Gum");
    }
}

class Cookie {
    static {
        System.out.println("Loading Cookie");
    }
}

public class SweetShop {
    public static void main(String[] args) {
        System.out.println("inside main");
        new Candy();
        System.out.println("After creating Candy");
        try {
            Class.forName("Gum");
        } catch (ClassNotFoundException e) {
            System.out.println("Couldn’t find Gum");
        }
        System.out.println("After Class.forName(\"Gum\")");
        new Cookie();
        System.out.println("After creating Cookie");
    }
}
// output:
// inside main
// Loading Candy
// After creating Candy
// Couldn’t find Gum
// After Class.forName("Gum")
// Loading Cookie
// After creating Cookie
```

当各个类在第一次调用时对应的静态代码块就会被调用，输出我们定制的信息。上例有一个比较特殊的语法 `forName()` 我们可以通过这个方法拿到对应的 Class 引用，当然如果找不到会抛 `ClassNotFoundExcepiton`。如果实体类已经创建了，你也可以通过 Object.getClass() 来拿到对应的类应用。

下面这个示例展示了部分 Class 中的常用方法：

```java
package samples;

interface HasBatteries {
}

interface Waterproof {
}

interface Shoots {
}

class Toy {
    // Comment out the following default constructor
    // to see NoSuchMethodError from (*1*)
    Toy() {
    }

    Toy(int i) {
    }
}

class FancyToy extends Toy
        implements HasBatteries, Waterproof, Shoots {
    FancyToy() {
        super(1);
    }
}

public class ToyTest {
    static void printlnInfo(Class cc) {
        System.out.println("Class name: " + cc.getName() +
                " is interface? [" + cc.isInterface() + "]");
        System.out.println("Simple name: " + cc.getSimpleName());
        System.out.println("Canonical name : " + cc.getCanonicalName());
    }

    public static void main(String[] args) {
        Class c = null;
        try {
            c = Class.forName("samples.FancyToy");
        } catch (ClassNotFoundException e) {
            System.out.println("Can’t find FancyToy");
            System.exit(1);
        }
        printlnInfo(c);
        for (Class face : c.getInterfaces())
            printlnInfo(face);
        Class up = c.getSuperclass();
        Object obj = null;
        try {
            // Requires default constructor:
            obj = up.newInstance();
        } catch (InstantiationException e) {
            System.out.println("Cannot instantiate");
            System.exit(1);
        } catch (IllegalAccessException e) {
            System.out.println("Cannot access");
            System.exit(1);
        }
        printlnInfo(obj.getClass());
    }
}
// output:
// Class name: samples.FancyToy is interface? [false]
// Simple name: FancyToy
// Canonical name : samples.FancyToy
// Class name: samples.HasBatteries is interface? [true]
// Simple name: HasBatteries
// Canonical name : samples.HasBatteries
// Class name: samples.Waterproof is interface? [true]
// Simple name: Waterproof
// Canonical name : samples.Waterproof
// Class name: samples.Shoots is interface? [true]
// Simple name: Shoots
// Canonical name : samples.Shoots
// Class name: samples.Toy is interface? [false]
// Simple name: Toy
// Canonical name : samples.Toy
```

* getSimpleName(): 输出类名
* getCanonicalName(): 输出全路径名
* islnterface(): 是否是接口
* getlnterfaces(): 拿到类实现的接口
* getSuperclass(): 拿到父类 Class 引用
* newlnstance(): 创建实例，但是这个方法要求对应的类必须有**默认构造函数**

