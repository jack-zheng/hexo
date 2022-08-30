---
title: 泛型
date: 2020-12-17 17:42:08
categories:
- On Java 8
tags:
- 泛型
---

## 简单泛型

多态是一种面向对象思想的泛化机制。你将方法参数设置为基类，这样方法就能接受任何派生类作为参数。不过拘泥于单一继承体系太过局限，如果以接口而不是类作为参数，限制就宽松多了。但即便是接口也有诸多限制，一旦制定了接口，就要求你的代码使用特定接口。Java 5 引入了泛型，实现了参数化类型的效果。泛型这个术语的含义是**适用于很多类**，初衷是通过结偶类或方法与所使用的类型之间的约束，使得类或方法具备最宽泛的表达力。

泛型出现的最主要动机之一是为了创建集合类。一个集合中存储多种不同类型的对象的情况很少见，通常我们只会用集合存储同一种类型的对象。泛型的主要目的之一就是用来约定集合要存储什么类型的对象，并且通过编译器确保规约得以满足。

泛型类基本语法，通过类型参数限定使用的类：

```java
// class 类名称 <泛型标识:可以是任意标识符>{
//   private 泛型标识 var; 
//   .....
//   }
// }
public class ArrayList<E> extends AbstractList<E> {}
```

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

Java 语法限制一个 method 只能返回一个值，那么如果你想返回多个，怎么办？ 这种情况下我们可以定一个对象里面持有多个值，并且只读不能写。这种对象有个名字，叫做 Data Transfer Object(DTO)/Message 也叫元组

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

## 泛型方法

类本身可能是泛型的，也可能不是，不过这与她的方法是否是泛型的并没有关系。泛型方法独立于类而改变方法，尽可能使用泛型方法，通常将单个方法泛型化要比将整个类泛型化更清晰易懂。

泛型方法定义：将泛型参数列表放置在返回值之前

## 泛型擦除

这个擦除好像只和集合有关系。

在泛型代码的内部，无法获取任何有关泛型参数类型的信息。你可以知道泛型参数标识符和泛型边界的信息，但无法知道实际类型参数从而创建特定的实例。

泛型只有在类型参数比某个具体类型更加泛化，代码能跨多个类工作时才有用。类型参数和他们在有用的泛型代码中的应用，通常比简单的类替换更加复杂。但不能因此认为使用 <T extends HasF> 形式就是有缺陷的。比如某个类有一个返回 T 的方法，此时泛型就帮你检测了返回值的类型。

Java 的泛型不是一种语言特性，而是一种妥协。为了提供迁移兼容性。

泛型类型只有在静态类型监测期间才出现，此后程序中的所有泛型类型都将被擦出，替换为他们的非泛型上界。

泛型的所有动作都发生在边界处，即入参的编译器检查和对返回值的转型。

## 补偿擦除

为了创建泛型实例，书中的实例通过在构造函数中传入类型的 class 来解决这个问题, 但只对对象有无参构造是有用。这样的话一些错误不能在编译时捕获，语言创建团队建议使用显示工厂(Suplier)并约束类型

我们无法创建泛型数组，通用解决办法是使用 ArrayList 代替

这里很多关于数组类型的讲解，概念感觉很模糊，什么类型不能改变之类的，我觉得，这些定义作者可能在之前的数组篇章里有介绍，有机会看一看。

## 边界

边界潜在更重要的效果是我们可以在绑定的类型中调用方法

## 通配符

flist 现在是 List<? extends Fruit>， 可以读作 '一个具有任何从Fruit集成的类型的列表'，然而这并不意味着这个List将持有任何类型的Fruit。通配符引用的是明确的类型，因此它意味着 '某种flist引用没有指定的具体类型'

```java
// 表示 任何Fruit子类的list
List<? extends Fruit> flist = new ArrayList<Apple>();
// 不能确定具体子类，所以不能 add, null 除外
// flist.add(new Apple());
// 可以 get
Fruit f = flist.get(0);
```

### 逆变

<? super MyClass> 的这种表现形式

```java
// 表示 任何Fruit超类的list
List<? super Fruit> flist = new ArrayList<Fruit>();
// 所以至少是个 Fruit，不确定具体类型，不能 get, 可以 add
flist.add(new Apple());
// 不可以 get
// Fruit f = flist.get(0);
```

这个和上面的 extend 贼TM绕

### 无界通配符

读起来很懵逼，找不到重点

使用确切类型来代替通配符类型的好处是，可以用泛型参数来做更多的事情。使用通配符使得你必须接受范围更宽的参数化类型作为参数。因此，必须逐个情况的权衡利弊，找到更适合你的需求的方法。

### 捕获转换

TBD

## 问题

### 基本类型不能作为类型参数

类似 List<int> 的写法是不允许的。解决办法，可以使用包装类型，自动装箱机制将自动实现类型转化

### 实现参数化接口

```java
interface Payable<T> {}

class Employee2 implements Payable<Employee2> {}

// 编译失败
class Hourly extends Employee2 implements Payable<Hourly> {}
```

这样的声明会有编译错误，第三局声明中，由于类型擦出，导致重复声明

### 转型和警告

### 重载

由于擦除机制，下面两条语句语意是相同的，所以不能编译,通过修改方法名可以修复这个问题

```java
public class UseList<W, T> {
    void f(List<T> v) {}
    void f(List<W> v) {}
}
```

### 基类劫持接口

比如常见的 Comparable 接口，我们如果想将泛型范围限制到更小的范围，但是 Cat 的声明会有编译错误

```java
public class ComparablePet implements Comparable<ComparablePet> {
    @Override
    public int compareTo(ComparablePet o) {
        return 0;
    }
}

// 由于泛型指定的类型不同，不能编译。
//class Cat extends ComparablePet implements Comparable<Cat> {
//}
```

## 自限定的类型

自限定指的是 `class SelfBounded<T extends SelfBounded<T>> {` 这样的定义，强调的是当 extends 关键字用于边界

### 古怪的循环泛型

### 自限定

他可以保证类型参数必须与被自定义的类相同。如果使用自限定，这个累所用的类型参数将与使用这个参数的类具有相同的基本类型。这回强制要求使用这个累的每个人都要遵循这种形式。

自限定还可以用于泛型方法。

### 参数协变

## 对缺乏潜在类型机制的补偿

### 反射

这个我之前在自己代码中就用过，就是在方法调用中通过 method name 来调用

### 将一个方法应用于序列

反射将类型监测移到了运行时，但是我们通常更希望在编译期就实现类型检测，更早的发现问题。