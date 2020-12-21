---
title: Java 泛型读书笔记
date: 2020-12-17 17:42:08
categories:
- 编程
tags:
- TIJ4
---

项目重构的时候刚好遇到一些泛型相关的问题，发现这块掌握的确实有点浅薄，重新认真读一遍 Think in Java 4th 相关章节并做笔记，顺便一提，100 页的内容有点心虚。

- [前述](#前述)
- [Comparison with C++](#comparison-with-c)
- [Simple generics](#simple-generics)
  - [A tuple library](#a-tuple-library)
  - [A stack class](#a-stack-class)
  - [RandomList](#randomlist)
- [Generic interfaces](#generic-interfaces)
- [Generic methods](#generic-methods)

想要解决的问题：

- [ ] 泛型的定义是什么
- [ ] 为什么需要泛型
- [ ] 集合类中使用泛型的注意点

## 前述

范型提供了一种比 interface 更高的通用性，它代表的语义是：对一批 unspecified type 的对象生效，而不单单是某一类 class 或者接口。

Java 的范型实现看作者的意思好像还不如 C++ 里实现的好。

## Comparison with C++

Java 灵感来源于 C++, 通过比较 C++ 的泛型(template)可以让你更清楚 Java 泛型的极限。

## Simple generics

generic 这个概念最初被提出来是为了创建容器 class，这是有别于 arrays 的一个概念，它会提供更高的扩展性，跟多信息要看 Holding your object 章节和后面的章节。(他这里提到的容器我估摸着就是 Collection 那一族类了)

下面是一个很常见的 class 持有 object 的例子, 里面有一个 Automobile 类，还有一个 Holder1 类通过构造函数持有它：

```java
class Automobile {}

public class Holder1 {
    private Automobile a;

    public Holder1(Automobile a) {
        this.a = a;
    }

    Automobile get() {
        return a;
    }
}
```

但是这种做法限制了你能传入的类型，在 Java 5 之前，如果你想把它变得更通用，你只能将它的参数类型改为 Object.

```java
public class Holder2 {
    private Object a;

    public Holder2(Object a) {
        this.a = a;
    }

    public void set(Object a) {
        this.a = a;
    }

    public Object get() {
        return a;
    }

    public static void main(String[] args) {
        Holder2 h2 = new Holder2(new Automobile());
        Automobile a = (Automobile) h2.get();
        h2.set("Not an Automobile");
        String s = (String) h2.get();
        h2.set(1); // Autoboxes to Integer
        Integer x = (Integer) h2.get();
    }
} 
```

上例中 Holder2 持有了三种不同类型的数据，但是通常来说我们只希望容器持有一种特殊类型的数据就行了，指定之后，这种特殊性可以在编译期就被检测出来。这种语法就是范型，我们在 class 名字后面接一个 尖括号+字母 的形式表示

```java
public class Holder3<T> {
    private T a;

    public Holder3(T a) {
        this.a = a;
    }

    public void set(T a) {
        this.a = a;
    }

    public T get() {
        return a;
    }

    public static void main(String[] args) {
        Holder3<Automobile> h3 = new Holder3<Automobile>(new Automobile());
        Automobile a = h3.get(); // No cast needed
        // h3.set("Not an Automobile"); // Error
        // h3.set(1); // Error
    }
} 
```

现在你在创建 Holder 的时候必须在尖括号中指定你想要的类型，当你从容器中取值的时候，jvm 会自动帮你完成类型转化。

### A tuple library

Java 语法限制一个 method 只能返回一个值，那么如果你想返回多个，怎么办？ 这种情况下我们可以定一个对象里面持有多个值，并且只读不能写。这种对象有个名字，叫做 Data Transfer Object/Message 也叫元组

元组长度可以是任意的，但是类型必须是确定的，这里我们可以用泛型绕过去，对于多个元素的问题，我们可以创建不同的元组来做兼容。

```java
public class TwoTuple<A, B> {
    public final A first;
    public final B second;

    public TwoTuple(A a, B b) {
        first = a;
        second = b;
    }

    public String toString() {
        return "(" + first + ", " + second + ")";
    }
}
```

精髓：通过 final 关键字 代替 getXXX method, 代码更简单明了，如果你想要一个三个变量的元组，你可以继承这个 class

```java
public class ThreeTuple<A, B, C> extends TwoTuple<A, B> {
    public final C third;

    public ThreeTuple(A a, B b, C c) {
        super(a, b);
        third = c;
    }

    public String toString() {
        return "(" + first + ", " + second + ", " + third + ")";
    }
} 
```

更多变量的元组以此类推, 测试如下

```java
class Amphibian {}

public class TupleTest {
    static TwoTuple<String, Integer> f() {
        // Autoboxing converts the int to Integer:
        return new TwoTuple<String, Integer>("hi", 47);
    }

    static ThreeTuple<Amphibian, String, Integer> g() {
        return new ThreeTuple<Amphibian, String, Integer>(
                new Amphibian(), "hi", 47);
    }

    public static void main(String[] args) {
        TwoTuple<String, Integer> ttsi = f();
        System.out.println(ttsi);
        // ttsi.first = "there"; // Compile error: final
        System.out.println(g());
    }
}
// output:
// (hi, 47)
// (generic.Amphibian@7c53a9eb, hi, 47)
```

通过泛型我们可以很轻松的指定 tuple 中成员的类型，通过 new 来新建对象还是略显繁琐，后面有改进型。

### A stack class

这里回顾了一下 Holding Your Objects 章节的 LinkedList 例子，然并卵我并没有看过 ╮(￣▽￣"")╭

下面是我们自己实现的带有 linked 存储机制的类：

```java
public class LinkedStack<T> {
    private static class Node<U> {
        U item;
        Node<U> next;

        Node() {
            item = null;
            next = null;
        }

        Node(U item, Node<U> next) {
            this.item = item;
            this.next = next;
        }

        boolean end() {
            return item == null && next == null;
        }
    }

    private Node<T> top = new Node<>(); // End sentinel

    public void push(T item) {
        top = new Node<>(item, top);
    }

    public T pop() {
        T result = top.item;
        if (!top.end())
            top = top.next;
        return result;
    }

    public static void main(String[] args) {
        LinkedStack<String> lss = new LinkedStack<>();
        for (String s : "Phasers on stun!".split(" "))
            lss.push(s);
        String s;
        while ((s = lss.pop()) != null)
            System.out.println(s);
    }
}
// output
// stun!
// on
// Phasers
```

这里通过内部静态类创建了一个 Node class 代表一个节点。这个带泛型的 Node 节点是一种很经典的数据结构，将数据通过泛型封装，结构体现在 Node 中。

LinkedStack 初始化时会声明一个内容为空的节点，在后续的 pop 方法中，通过判断节点的这两个内容是不是空来断定容器是否为空。

### RandomList

设计一个数据结构，每次调用 list 的 select 方法的时候会随机返回一个元素：

```java
public class RandomList<T> {
    private ArrayList<T> storage = new ArrayList<T>();
    private Random rand = new Random(47);

    public void add(T item) {
        storage.add(item);
    }

    public T select() {
        return storage.get(rand.nextInt(storage.size()));
    }

    public static void main(String[] args) {
        RandomList<String> rs = new RandomList<String>();
        for (String s : ("The quick brown fox jumped over " +
                "the lazy brown dog").split(" "))
            rs.add(s);
        for (int i = 0; i < 11; i++)
            System.out.print(rs.select() + " ");
    }
}

// output
// brown over fox quick quick dog brown The brown lazy brown
```

数据结构很简单，随机性由 Random 对象提供 `random.nextInt(x)` 可以给出 0-x 返回内的整数。RandomList 里面新建一个 ArrayList 作为数据存储容器。

## Generic interfaces 

接口也可以由泛型配置。Generator(生成器) 是一种特殊的工厂方法，他可以在不接受任何参数的情况下，创建你需要的对象。这里我们为产生对象的方法取名为 `next()`。

示例说明：

1. 声明一个 Generator 接口带有泛型参数，只有一个方法 next 返回类型为泛型
2. 创建产品基类 Coffee 并创建对应的实体类
3. 创建生成器实体类 CoffeeGenerator，他实现了 Generator 接口和 Iterable 接口，前者用于一次生成一个的模式，后者用于一次性生成多个的模式，有了 Iterable 就可以支持 foreach 语法了

这里面唯一我想不到的是他通过 `Class.newInstance()` 直接生成对象的，就感觉很突然，很直球 (´Д` )

而且他在实现里使用 Iterable + 内部类实现 Iterator 的方式，我对这个也听陌生的，虽然知道有这种用法。。。感觉又可以开坑了 （；￣ェ￣）

```java
public interface Generator<T> {T next();}

public class Coffee {
    private static long counter = 0;
    private final long id = counter++;

    public String toString() {
        return getClass().getSimpleName() + " " + id;
    }
}

class Latte extends Coffee {}

class Mocha extends Coffee {}

class Cappuccino extends Coffee {}

class Americano extends Coffee {}

class Breve extends Coffee {}

public class CoffeeGenerator
        implements Generator<Coffee>, Iterable<Coffee> {
    private Class[] types = {Latte.class, Mocha.class, Cappuccino.class, Americano.class, Breve.class,};
    private static Random rand = new Random(47);

    public CoffeeGenerator() {
    }

    // For iteration:
    private int size = 0;

    public CoffeeGenerator(int sz) {
        size = sz;
    }

    public Coffee next() {
        try {
            return (Coffee)types[rand.nextInt(types.length)].newInstance();
            // Report programmer errors at run time:
        } catch (Exception e) {
            throw new RuntimeException(e);
        }
    }

    class CoffeeIterator implements Iterator<Coffee> {
        int count = size;

        public boolean hasNext() {
            return count > 0;
        }

        public Coffee next() {
            count--;
            return CoffeeGenerator.this.next();
        }

        public void remove() { // Not implemented
            throw new UnsupportedOperationException();
        }
    }

    public Iterator<Coffee> iterator() {
        return new CoffeeIterator();
    }

    public static void main(String[] args) {
        CoffeeGenerator gen = new CoffeeGenerator();
        for (int i = 0; i < 5; i++)
            System.out.println(gen.next());
        for (Coffee c : new CoffeeGenerator(5))
            System.out.println(c);
    }
}
// output
// Americano 0
// Latte 1
// Americano 2
// Mocha 3
// Mocha 4
// Breve 5
// Americano 6
// Latte 7
// Cappuccino 8
// Cappuccino 9
```

下面是使用泛型接口实现斐波那契额的例子

算法这一块，不是我吹逼，我真的太弱了 （；￣ェ￣） 老是忘记

这里 class 内部持有一个 count 变量，每次调用 next() 方法，都会使得 count+1, 第 n 次调用就相当于打印 fib(n) 的值，fib 是一个基本的递归函数。

```java
public class Fibonacci implements Generator<Integer> {
    private int count = 0;

    public Integer next() {
        return fib(count++);
    }

    private int fib(int n) {
        if (n < 2) return 1;
        return fib(n - 2) + fib(n - 1);
    }

    public static void main(String[] args) {
        Fibonacci gen = new Fibonacci();
        for (int i = 0; i < 18; i++)
            System.out.print(gen.next() + " ");
    }
}
// output: 1 1 2 3 5 8 13 21 34 55 89 144 233 377 610 987 1597 2584
```

泛型参数不支持基本数据类型，必须是包装型的。

下面我们用 Iterable 接口 + adapter 模式扩展一下上面的斐波那契数列，说实话，这个 adapter 模式和我印象中的不一样，又得复习一下对应的那块设计模式 code 了 （；￣ェ￣）

```java
public class IterableFibonacci extends Fibonacci implements Iterable<Integer> {
    private int n;

    public IterableFibonacci(int count) {
        n = count;
    }

    @Override
    public Iterator<Integer> iterator() {
        return new Iterator<Integer>() {
            public boolean hasNext() {
                return n > 0;
            }

            public Integer next() {
                n--;
                return IterableFibonacci.this.next();
            }

            public void remove() { // Not implemented
                throw new UnsupportedOperationException();
            }
        };
    }

    public static void main(String[] args) {
        for (int i : new IterableFibonacci(18))
            System.out.print(i + " ");
    }
}
// output: 1 1 2 3 5 8 13 21 34 55 89 144 233 377 610 987 1597 2584
```

## Generic methods

TBD...