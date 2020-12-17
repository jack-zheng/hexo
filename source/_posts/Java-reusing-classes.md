---
title: Java 代码重用
date: 2020-12-17 18:51:46
categories:
- 编程
tags:
- TIJ4
---

最近看 Code 经常看到有使用 final 参数的例子，但是对这点没有系统的认识，重新认真读一遍 Think in Java 4th 相关章节并做笔记。

想要解决的问题：

- [ ] local inner class 中如果用到方法中的参数，为什么要用 final 修饰？

F1: java 编译器在实现 Q1 中描述的问题时，用的是值拷贝，而不是 reference 拷贝，为了防止内外值不一致，只能强制用 final 把它定为一个常量，不改变他的值

## The final keyword

Java 里面 final 这个关键字的含义会根据上下文不同而有所区别，但是大体上来说，他都会表达出一个 '不允许改变' 的含义。你会出于两种目的阻止他改变，一种是设计上另一种是效率上。这两种目的很不一样，所以可能存在误用的情况。

接下来我们会例举三种 final 的使用场景：data, method, class

### final data

在两种情况下你以将变量声明为常量：

1. 编译期常量，不能被改变
2. 在运行时赋值并且不能被改变

编译时常量有一个好处是在编译期间就将常量相关的运算做了，可以节省运行时的计算时间。这种情况下，对应的常量类型必须是 final 修饰的 primitive 类型，声明变量时就得赋值。

static + final 表明系统中只有一块内存空间存储相应的值。这种变量是有**命名规范**的，全部大些，中间用下划线分隔。

通过使用 final 修饰 class 表明这个类的 reference 是一个常量。

示例说明：

声明一个 Value class 用做演示 final 修饰对象情况的素材。

valueOne: 演示 final 修饰的基本数据类型不能改变值

VALUE_TWO/VALUE_THREE: 演示 static final 的常见用法和命名规范

i4/INT_5: final 修饰的变量可以通过表达式赋值，不一定需要直接赋值

v1/v2/VAL_3: final 修饰的对象 reference 不能改，但对应的对象可以改变

a: 数组也是一种对象，符合上一条行为规范

```java
import java.util.Random;

class Value {
    int i; // Package access

    public Value(int i) {
        this.i = i;
    }
}

public class FinalData {
    private static Random rand = new Random(47);
    private String id;

    public FinalData(String id) {
        this.id = id;
    }

    // Can be compile-time constants:
    private final int valueOne = 9;
    private static final int VALUE_TWO = 99;
    // Typical public constant:
    public static final int VALUE_THREE = 39;
    // Cannot be compile-time constants:
    private final int i4 = rand.nextInt(20);
    static final int INT_5 = rand.nextInt(20);
    private Value v1 = new Value(11);
    private final Value v2 = new Value(22);
    private static final Value VAL_3 = new Value(33);
    // Arrays:
    private final int[] a = {1, 2, 3, 4, 5, 6};

    public String toString() {
        return id + ": " + "i4 = " + i4 + ", INT_5 = " + INT_5;
    }

    public static void main(String[] args) {
        FinalData fd1 = new FinalData("fd1");
        //! fd1.valueOne++; // Error: can’t change value
        fd1.v2.i++; // Object isn’t constant!
        fd1.v1 = new Value(9); // OK -- not final
        for (int i = 0; i < fd1.a.length; i++)
            fd1.a[i]++; // Object isn’t constant!
        //! fd1.v2 = new Value(0); // Error: Can’t
        //! fd1.VAL_3 = new Value(1); // change reference
        //! fd1.a = new int[3];
        System.out.println(fd1);
        System.out.println("Creating new FinalData");
        FinalData fd2 = new FinalData("fd2");
        System.out.println(fd1);
        System.out.println(fd2);
    }
} 
```

public so they’re usable outside the package, static to emphasize that there’s only one, and final to say that it’s a constant.

public: 包外可访问；static：强调只有一份空间；final：常量

### Blank finals

变量声明为 final 类型但是没有给初始值的情况叫做 Blank finals。但是在这个变量使用前，它必须被初始化。

归结为两种情况为 final 变量赋值，一种就是声明时赋值，另一种是构造函数内赋值。不做的话会有编译错误。提供第二种赋值方式之后，对于同一个变量，每个类都可以有自己不同 final 变量值了。

```java
class Poppet {
    private int i;

    Poppet(int ii) {
        i = ii;
    }
}

public class BlankFinal {
    private final int i = 0; // Initialized final
    private final int j; // Blank final
    private final Poppet p; // Blank final reference

    // Blank finals MUST be initialized in the constructor:
    public BlankFinal() {
        j = 1; // Initialize blank final
        p = new Poppet(1); // Initialize blank final reference
    }

    public BlankFinal(int x) {
        j = x; // Initialize blank final
        p = new Poppet(x); // Initialize blank final reference
    }

    public static void main(String[] args) {
        new BlankFinal();
        new BlankFinal(47);
    }
}
```

### final arguments

你还可以在方法的参数列表中，将变量类型指定为 final，标示在方法体内你不能改变参数的 reference。

示例说明：

with/without: 表明 final 修饰对参数不能内改变 reference

f()/g(): 表明 final 修饰的基本数据类型值不能被修改

```java

class Gizmo {
    public void spin() {
    }
}

public class FinalArguments {
    void with(final Gizmo g) {
        //! g = new Gizmo(); // Illegal -- g is final
    }

    void without(Gizmo g) {
        g = new Gizmo(); // OK -- g not final
        g.spin();
    }

    // void f(final int i) { i++; } // Can’t change
    // You can only read from a final primitive:
    int g(final int i) {
        return i + 1;
    }

    public static void main(String[] args) {
        FinalArguments bf = new FinalArguments();
        bf.without(null);
        bf.with(null);
    }
}
```

### final methods

fianl 修饰 method 有两种作用，一种是表达了你不想被修饰的方法在子类中被重写而改变语义；另一种是提升执行效率。但是第二种功能在 Java 5/6 时已经包含在 JVM 优化中了，所以现在只推荐在第一种意图是使用该语法。

```java
class MyFinal {
    public final void method01(){};
}

public class MyFinalTest extends MyFinal{
    // ! public final void method01(){}; // compile error
}
```

### final and private

类中的所有 private 方法其实都是默认有 final 修饰的，只不过你显示的加了也没什么额外的作用。

private 方法代表的意思不就是外部不能访问，当然也不能修改这个方法吗，没毛病。

示例说明：

下面的例子中，我们在基类中声明了两个方法 f()/g() 分别显示和隐示的加上 final 关键字。虽然你可以在它的子类中重写这个方法，但是只有在最末端的子类中可以调用，且调用的还是子类自己的实现。

如果你在子类的实现上加上 `Override` 标签，还会有编译错误，因为 private 方法自带 final, 表明的含义就是不能被重写。

```java
class WithFinals {
    // Identical to "private" alone:
    private final void f() {
        System.out.println("WithFinals.f()");
    }

    // Also automatically "final":
    private void g() {
        System.out.println("WithFinals.g()");
    }
}

class OverridingPrivate extends WithFinals {
    private final void f() {
        System.out.println("OverridingPrivate.f()");
    }

    private void g() {
        System.out.println("OverridingPrivate.g()");
    }
}

class OverridingPrivate2 extends OverridingPrivate {
    public final void f() {
        System.out.println("OverridingPrivate2.f()");
    }

    public void g() {
        System.out.println("OverridingPrivate2.g()");
    }
}

public class FinalOverridingIllusion {
    public static void main(String[] args) {
        OverridingPrivate2 op2 = new OverridingPrivate2();
        op2.f();
        op2.g();
        // You can upcast:
        OverridingPrivate op = op2;
        // But you can’t call the methods:
        //! op.f();
        //! op.g();
        // Same here:
        WithFinals wf = op2;
        //! wf.f();
        //! wf.g();
    }
} 
// output:
// OverridingPrivate2.f()
// OverridingPrivate2.g()
```

If a method is private, it isn’t part of the base-class interface. It is just some code that’s hidden away inside the class, and it just happens to have that name.

私有方法并不是基类的一部分，它是该类中的隐藏代码，只不过恰巧有了名字。

### final class

final 修饰的 class 表明，不管出于什么目的，你不想你的这个 class 被继承。

```java
class SmallBrain {
}

final class Dinosaur {
    int i = 7;
    int j = 1;
    SmallBrain x = new SmallBrain();

    void f() {
    }
}

//! class Further extends Dinosaur {}
// error: Cannot extend final class ‘Dinosaur’
public class Jurassic {
    public static void main(String[] args) {
        Dinosaur n = new Dinosaur();
        n.f();
        n.i = 40;
        n.j++;
    }
}
```

final class 的 field 可以不是 final 的，但是 final class 里面的 method 都隐示为 final method。因为 final class 就是为了防止被继承，都不被继承了，对应的方法都不能重写也是合理的。

### final caution 

例举了一些老的 Java lib 实现 Vector 和 Hashtable 说明，使用 final 修饰方法的时候要谨慎，你完全不知道其他人会怎样使用你的代码。
