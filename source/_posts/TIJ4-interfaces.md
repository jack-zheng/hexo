---
title: TIJ4 接口
date: 2021-03-09 19:17:36
categories:
- TIJ4
tags:
- interface
---

- [Intro](#intro)
- [Abstract classes and methods](#abstract-classes-and-methods)
- [Interfaces](#interfaces)
- [Complete decoupling](#complete-decoupling)
- ["Multiple inheritance" in Java](#multiple-inheritance-in-java)
- [Nesting interfaces](#nesting-interfaces)

## Intro

Interfaces and abstract classes provide more structured way to
separate interface from implementation.

## Abstract classes and methods

描述了前几章乐器的例子，其中使用 abstract class 会更符合题意

## Interfaces

interface 是 abstract 在抽象上的进一步体现，他允让你决定方法名，参数列表和返回值，并且你不用实现它，结果只做了规范，但不需要实现。

通过 interface 你仿佛在说：所有实现了这个接口的类都应该长这样。其他编程语言中也叫**protocol**

除此之外，interface 变相的让你的类实现了多重继承，你的类可以转化为多个基类。

interface 可以用 public 修饰，也可以不写(包可见). 在 interface 中声明的 field 都是默认 static + final 的。interface 中声明的方法即使你没有显示的指定访问修饰符也是默认是 public 的。

// TODO，这里也给了例子，不过补上要等看了前面的章节再说了

## Complete decoupling

设想这么一种场景，我们创建一个处理器类，他有两个方法 name() 和 process()。process() 可以接收字符串并处理。我们再声明一个 Apply 类作为 client 端声明方法接收 Processor 类，调用 Processor 方法。这个没记错的话就是策略模式了。作者的行为中也这么指出。

```java
class Processor {
    public String name() {
        return getClass().getSimpleName();
    }
    Object process(Object input) { return input; }
}

class Upcase extends Processor {
    String process(Object input) { // Covariant return
        return ((String)input).toUpperCase();
    }
}

class Downcase extends Processor {
    String process(Object input) {
        return ((String)input).toLowerCase();
    }
}

class Splitter extends Processor {
    String process(Object input) {
        // The split() argument divides a String into pieces:
        return Arrays.toString(((String)input).split(" "));
    }
}

public class Apply {
    public static void process(Processor p, Object s) {
        System.out.println("Using Processor " + p.name());
        System.out.println(p.process(s));
    }
    public static String s =
            "Disagreement with beliefs is by definition incorrect";
    public static void main(String[] args) {
        process(new Upcase(), s);
        process(new Downcase(), s);
        process(new Splitter(), s);
    }
}

// Using Processor Upcase
// DISAGREEMENT WITH BELIEFS IS BY DEFINITION INCORRECT
// Using Processor Downcase
// disagreement with beliefs is by definition incorrect
// Using Processor Splitter
// [Disagreement, with, beliefs, is, by, definition, incorrect]
```

现在我们有另一批过滤器类

```java
public class Waveform {
    private static long counter;
    private final long id = counter++;

    public String toString() {
        return "Waveform " + id;
    }
}

public class Filter {
    public String name() {
        return getClass().getSimpleName();
    }

    public Waveform process(Waveform input) {
        return input;
    }
}

public class LowPass extends Filter {
    double cutoff;

    public LowPass(double cutoff) {
        this.cutoff = cutoff;
    }

    public Waveform process(Waveform input) {
        return input; // Dummy processing
    }
}

public class HighPass extends Filter {
    double cutoff;

    public HighPass(double cutoff) {
        this.cutoff = cutoff;
    }

    public Waveform process(Waveform input) {
        return input;
    }
}

public class BandPass extends Filter {
    double lowCutoff, highCutoff;

    public BandPass(double lowCut, double highCut) {
        lowCutoff = lowCut;
        highCutoff = highCut;
    }

    public Waveform process(Waveform input) {
        return input;
    }
}
```

他的行为模式和前面的 Processor 是很相似的，理论上来说我们可以将 Filter 看作是一个算法的集合，然后 Apply 中接收 Filter 这族算法，同时接收一个 Waveform 作为输入，process() 方法产生输出即可。但是由于 Apply 定义的方法指定了 Processor 为参数，导致兼容 Filter 失败了

这时我们改一下 Processor 的代码，将其定义为一个接口

```java
public interface Processor {
    String name();
    Object process(Object input);
}

public class Apply {
    public static void process(Processor p, Object s) {
        System.out.println("Using Processor " + p.name());
        System.out.println(p.process(s));
    }
}
```

同时包装一个处理 String 的 Processor 基类，并实现各种具体的处理器

```java
// 虽然它这种 abstract 类中直接写具体调用的做法我总感觉很飘逸，但是理解不难
public abstract class StringProcessor implements Processor{
    public String name() {
        return getClass().getSimpleName();
    }
    public abstract String process(Object input);
    public static String s = "If she weighs the same as a duck, she’s made of wood";
    public static void main(String[] args) {
        Apply.process(new Upcase(), s);
        Apply.process(new Downcase(), s);
        Apply.process(new Splitter(), s);
    }
}
class Upcase extends StringProcessor {
    public String process(Object input) { 
        // Covariant return
        return ((String)input).toUpperCase();
    }
}
class Downcase extends StringProcessor {
    public String process(Object input) {
        return ((String)input).toLowerCase();
    }
}
class Splitter extends StringProcessor {
    public String process(Object input) {
        return Arrays.toString(((String)input).split(" "));
    }
}

// Using Processor Upcase
// IF SHE WEIGHS THE SAME AS A DUCK, SHE’S MADE OF WOOD
// Using Processor Downcase
// if she weighs the same as a duck, she’s made of wood
// Using Processor Splitter
// [If, she, weighs, the, same, as, a, duck,, she’s, made, of, wood]
```

现在轮到重构 Filter 部分了，和我原本料想的直接改代码不同，作者直接假定，这部分 Filter 类的代码就是第三方方法，你无权改动，这个时候怎么办？他又引入了 Adaptor 模式。好在这两个模式我都还挺熟悉，不过没记录过，明天花点时间写一下 ╮(￣▽￣"")╭

通过实现 Processor 实现一个 Filter 的包装类, 然后将 Filter 传给包装类，并把包装类作为 Apply.process() 的方式实现了曲线救国，秀啊，小老弟。

```java
class FilterAdapter implements Processor {
    Filter filter;

    public FilterAdapter(Filter filter) {
        this.filter = filter;
    }

    public String name() {
        return filter.name();
    }

    public Waveform process(Object input) {
        return filter.process((Waveform) input);
    }
}

public class FilterProcessor {
    public static void main(String[] args) {
        Waveform w = new Waveform();
        Apply.process(new FilterAdapter(new LowPass(1.0)), w);
        Apply.process(new FilterAdapter(new HighPass(2.0)), w);
        Apply.process(new FilterAdapter(new BandPass(3.0, 4.0)), w);
    }
}

// Using Processor LowPass
// Waveform 0
// Using Processor HighPass
// Waveform 0
// Using Processor BandPass
// Waveform 0
```

## "Multiple inheritance" in Java

TBD

## Nesting interfaces

Interfaces 可以内嵌到 class 或者其他 interface 内部，这种做法可以引入一些有趣的特性。

```java
class A {
    interface B {
        void f();
    }

    public class BImp implements B {
        public void f() {
        }
    }

    private class BImp2 implements B {
        public void f() {
        }
    }

    public interface C {
        void f();
    }

    class CImp implements C {
        public void f() {
        }
    }

    private class CImp2 implements C {
        public void f() {
        }
    }

    private interface D {
        void f();
    }

    private class DImp implements D {
        public void f() {
        }
    }

    public class DImp2 implements D {
        public void f() {
        }
    }

    public D getD() {
        return new DImp2();
    }

    private D dRef;

    public void receiveD(D d) {
        dRef = d;
        dRef.f();
    }
}

interface E {
    interface G {
        void f();
    }

    // Redundant "public":
    public interface H {
        void f();
    }

    void g();
    // Cannot be private within an interface:
    // ! private interface I {}
}

public class NestingInterfaces {
    public class BImp implements A.B {
        public void f() {
        }
    }

    class CImp implements A.C {
        public void f() {
        }
    }

    // Cannot implement a private interface except
    // within that interface’s defining class:
    // ! class DImp implements A.D {
    // ! public void f() {}
    // ! }
    class EImp implements E {
        public void g() {
        }
    }

    class EGImp implements E.G {
        public void f() {
        }
    }

    class EImp2 implements E {
        public void g() {
        }

        class EG implements E.G {
            public void f() {
            }
        }
    }

    public static void main(String[] args) {
        A a = new A();
        // Can’t access A.D:
        // ! A.D ad = a.getD();
        // Doesn’t return anything but A.D:
        // ! A.DImp2 di2 = a.getD();
        // Cannot access a member of the interface:
        // ! a.getD().f();
        // Only another A can do anything with getD():
        A a2 = new A();
        a2.receiveD(a.getD());
    }
}
```

这种嵌套 interface 的语法是合理的，和普通的 interface 一下，所有访问修饰符都用在嵌套接口上。

内嵌接口在使用上和 内部类并没有什么不同. 那么 private 的 interface 有什么价值呢？如果你以为 nested private interface 的实现只能是 private 的，那么你就错了。看看 DImp2 就可知，其实现可以是任意访问类型的。

```
! 后面的这段感觉翻译不过去，模模糊糊，看了中文版，貌似没有相关的章节。。。。汗

but A.DImp2 shows that it can also be implemented as a public class. However,
A.DImp2 can only be used as itself. You are not allowed to mention the fact that it
implements the private interface D, so implementing a private interface is a way to force
the definition of the methods in that interface without adding any type information (that is,
without allowing any upcasting).

个人理解为 private 接口将接口的实现和定义限制在了定义类里面。而且一般使用的时候都是会返回接口类，像上面的 D getD(), 而 D 又是 private 的，限制了他的使用，这应该就是原文中 'A.DImp2 can only be used as itself' 的意思吧
```

`getD()` 方法是 private 修饰的嵌套接口的更特殊的使用方式，在 `main()` 中，我们 comment 了很多对 D 接口的引用，但是这些用法都有编译错误。唯一的使用方式是新建一个 A 对象，调用以 D 为参数的方法。

Interface E 的例子想要说明的事，接口内部也能声明接口，但是秉承接口内部元素必须都是规则，内嵌的接口也**必须只能**是 public 的

Nestinglnterfaces 类中给出了嵌套接口更多的实现，当我们实现一个嵌套接口的外部接口(E)时，是不需要我们实现对应的嵌套接口的。

private interface 在外部是不能访问的，只能在声明他的类内部做实现
