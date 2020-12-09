---
title: Java inner class
date: 2020-12-09 15:32:00
categories:
- 编程
tags:
- thing in java
---

- [前述](#前述)
- [Creating inner classes](#creating-inner-classes)
- [The link to the outer class](#the-link-to-the-outer-class)
- [Using .this and .new](#using-this-and-new)
- [Inner classes and upcasting](#inner-classes-and-upcasting)
- [Inner classes in methods and scopes](#inner-classes-in-methods-and-scopes)

最近在看 Spring Core 文档的以后，刚好遇到一个 Inner Class 相关的问题，回忆以下突然发现对他基本没有什么很深入的理解，特此重新阅读以下 Think in Java 4th 看看能不能有什么特别的收获。

想要解决的问题：

1. 什么是内部类
2. 静态/非静态内部类有什么区别
3. 内部类有什么用
4. 字节码层面是怎么表现的

## 前述

Java 语法是支持在一个 class 内部再放入另一个 class 的定义的，这种做法叫做 内部类(Inner Class)。

Inner class 是一个很有价值的功能，他让你可以把两个逻辑上共存的 class 放到一起，并让他们之间有了一层可见性控制的功能。

## Creating inner classes

创建内部类的做法只需要直接将内部类定义放到外部类里面就行了， 很直接了当, 一般来说，外部类还有会有一些方法用来返回内部类引用，比如 to() 和 contents()。如果是 `非静态`内部类，你需要先新建外部类，然后才能创建内部类。如果是 `静态`内部类，你可以直接创建内部类对象。

注意内部类创建的声明方式 `Surrounding.Inner inner = surrounddingInstance.method();` 这个还是挺特别的。

```java
public class Parcel2 {
    class Contents {
        private int i = 11;

        public int value() {
            return i;
        }
    }

    class Destination {
        private String label;

        Destination(String whereTo) {
            label = whereTo;
        }

        String readLabel() {
            return label;
        }
    }

    public Destination to(String s) {
        return new Destination(s);
    }

    public Contents contents() {
        return new Contents();
    }

    public void ship(String dest) {
        Contents c = contents();
        Destination d = to(dest);
        System.out.println(d.readLabel());
    }

    public static void main(String[] args) {
        Parcel2 p = new Parcel2();
        p.ship("Tasmania");
        Parcel2 q = new Parcel2();
        // Defining references to inner classes:
        Parcel2.Contents c = q.contents();
        Parcel2.Destination d = q.to("Borneo");
    }
}

// output: Tasmania
```

## The link to the outer class

内部类最显著的特点：Inner class 创建的时候会持有一个外边引用的联结，这使得他能没有限制的访问外部类成员变量和方法。

下面的例子，我们声明了一个接口 Selector, 它定义了三个方法， Sequence 是一个可变长的数组容器，只实现了构造函数和 add() 方法。他有一个内部类 SequenceSelector 实现了 Selector 接口。功能上，Sequence 负责表明对象实例，SequenceSelector 作用类似游标，用来遍历 Sequence 里的子元素。

重点：SequenceSelector **可以访问** Sequence 的**私有**变量而不受限制

```java
interface Selector {
    boolean end();

    Object current();

    void next();
}

public class Sequence {
    private Object[] items;
    private int next = 0;

    public Sequence(int size) {
        items = new Object[size];
    }

    public void add(Object x) {
        if (next < items.length) items[next++] = x;
    }

    private class SequenceSelector implements Selector {
        private int i = 0;

        public boolean end() {
            return i == items.length;
        }

        public Object current() {
            return items[i];
        }

        public void next() {
            if (i < items.length) i++;
        }
    }

    public Selector selector() {
        return new SequenceSelector();
    }

    public static void main(String[] args) {
        Sequence sequence = new Sequence(10);
        for (int i = 0; i < 10; i++) sequence.add(Integer.toString(i));
        Selector selector = sequence.selector();
        while (!selector.end()) {
            System.out.print(selector.current() + " ");
            selector.next();
        }
    }
}
```

## Using .this and .new

内部类中，你可以使用 `外部类.this` 的方式得到外部类的引用。下面的例子中，inner class 的 `outer()` 通过 `DotThis.this` 返回了外部类的引用，并调用 `f()` 打印结果

```java
public class DotThis {
    void f() {
        System.out.println("DotThis.f()");
    }

    public class Inner {
        public DotThis outer() {
            return DotThis.this;
            // A plain "this" would be Inner’s "this"
        }
    }

    public Inner inner() {
        return new Inner();
    }

    public static void main(String[] args) {
        DotThis dt = new DotThis();
        DotThis.Inner dti = dt.inner();
        dti.outer().f();
    }
}

// output: DotThis.f()
```

如果你想创建内部类，那么你可以通过 `外部类实例.new` 的形式创建。

创建是你不需要为 `Inner()` 指定前缀 class，这个挺方便的。本来还以为需要用 `dn.new DotNew.Inner();` 的语法，后来试过发现编译会报错。

```java
public class DotNew {
    public class Inner {
    }

    public static void main(String[] args) {
        DotNew dn = new DotNew();
        DotNew.Inner dni = dn.new Inner();
    }
}
```

PS: \[Attention\] 书上将非静态内部类叫做 inner class，静态内部类叫做 nested class 或 static inner class, 有点意思

想要创建内部类你必须要先创建外部类，这是因为，创建内部类需要外部类的引用，这更像是一个先决条件。如果想脱钩，可以使用 nested class，静态内部类。

## Inner classes and upcasting

Inner class 和接口结合，可以达到隐藏自己实现的目的，这个特性很厉害。下面的例子里 Parcel4 通过 destination() 和 contents() 拿到内部类的引用。但是由于返回的是接口类型的，而且内部类是 private 和 protected 的所以包外的类压根就不能访问，也不知道他的实现细节。这就很阴霸，起到了和强的隔离效果 （；￣ェ￣）

```java
public class TestParcel {
    public static void main(String[] args) {
        Parcel4 p = new Parcel4();
        Contents c = p.contents();
        Destination d = p.destination("Tasmania");
        // Illegal -- can’t access private class:
        // ! Parcel4.PContents pc = p.new PContents();
    }
}

interface Destination {
    String readLabel();
}

interface Contents {
    int value();
}

class Parcel4 {
    private class PContents implements Contents {
        private int i = 11;

        public int value() {
            return i;
        }
    }

    protected class PDestination implements Destination {
        private String label;

        private PDestination(String whereTo) {
            label = whereTo;
        }

        public String readLabel() {
            return label;
        }
    }

    public Destination destination(String s) {
        return new PDestination(s);
    }

    public Contents contents() {
        return new PContents();
    }
}
```

> Exc8: 确认下外部类是否能访问内部类变量? Determine whether an outer class has access to the private elements of its inner class.
> 看调用方式，如果是外部类方法直接调用内部类成员变量，不能，外部类实例化后，内部类可能压根就没有实例化，访问个毛线
> 如果是实例化了，就可以调用，习题答案如下

```java
class Outer8 {	
	class Inner {
		private int ii1 = 1;
		private int ii2 = 2;
		private void showIi2() { System.out.println(ii2); }
		private void hi() { System.out.println("Inner hi"); }
		}
	// Need to create objects to access private elements of Inner:
	int oi = new Inner().ii1;
	void showOi() { System.out.println(oi); }
	void showIi2() { new Inner().showIi2(); } 
	void outerHi() { new Inner().hi(); }
	public static void main(String[] args) {
		Outer8 out = new Outer8();
		out.showOi();
		out.showIi2();
		out.outerHi();
	}
}
```

## Inner classes in methods and scopes