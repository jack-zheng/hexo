---
title: TIJ4 内部类
date: 2020-12-09 15:32:00
categories:
- TIJ4
tags:
- inner class
---

- [Intro](#intro)
- [Creating inner classes](#creating-inner-classes)
- [The link to the outer class](#the-link-to-the-outer-class)
- [Using .this and .new](#using-this-and-new)
- [Inner classes and upcasting](#inner-classes-and-upcasting)
- [Inner classes in methods and scopes](#inner-classes-in-methods-and-scopes)
- [Anonymous inner classes](#anonymous-inner-classes)
  - [Factory Method revisited](#factory-method-revisited)
- [Nested classes](#nested-classes)
  - [Classes inside interfaces](#classes-inside-interfaces)
  - [Reaching outward from a multiply nested class](#reaching-outward-from-a-multiply-nested-class)
- [Why inner classes?](#why-inner-classes)
  - [Closures & callbacks](#closures--callbacks)
  - [Inner classes & control frameworks](#inner-classes--control-frameworks)
- [Inheriting from inner classes](#inheriting-from-inner-classes)
- [Can inner classes be overridden?](#can-inner-classes-be-overridden)
- [Local inner classes](#local-inner-classes)
- [Inner-class identifiers](#inner-class-identifiers)
- [Summary](#summary)
- [实践出真知](#实践出真知)

最近在看 Spring Core 文档, 刚好遇到一个 Inner Class 相关的问题, 回忆一下突然发现对他基本没有什么很深入的理解, 特此重新阅读一下 Think in Java 4th 相关章节看看能不能有什么特别的收获. 

想要解决的问题:

- [x] 什么是内部类 - 将 class 定义嵌入另一个 class 内部, 我们就得到了一个内部类
- [x] 静态/非静态内部类有什么区别 - 前者可以单独使用, 后者需要持有外部类(enclosing class)的实例才能使用
- [x] 内部类有什么用 - 更好的闭包
- [x] 字节码层面是怎么表现的 - class 分别编译, 外层会持有内层的 class reference

## Intro

Java 语法是支持在一个 class 内部再放入另一个 class 的定义的, 这种做法叫做 内部类(Inner Class). 

Inner class 是一个很有价值的功能, 他让你可以把两个逻辑上共存的 class 放到一起, 并让他们之间有可见性控制的特性. 

## Creating inner classes

如果想要创建一个内部类你只需要直接将内部类的定义放到外部类里面就行了. 外部类一般会有一些方法用来返回内部类引用, 比如下面例子中的 `to()` 和 `contents()` 方法. 如果是 `非静态` 内部类, 你需要先新建外部类, 然后才能创建内部类. 如果是 **静态** 内部类, 则你可以直接通过 class 引用创建内部类对象. 

示例说明:

创建静态/非静态内部类

```java
// 类结构
public class OuterClass {
    public static class NestedClass {}
    public class InnerClass {}
}

public class TestInner {
    public static void main(String[] args) {
        // 静态内部类的情况
        OuterClass.NestedClass nestedClass = new OuterClass.NestedClass();

        // 非静态内部类的情况
        OuterClass outer = new OuterClass();
        OuterClass.InnerClass innerClass = outer.new InnerClass();

        // 或者合二为一
        OuterClass.InnerClass innerClass2 = new OuterClass().new InnerClass();
    }
}
```

示例说明:

parcel = 包裹, 该示例以包裹运输为场景, 用包裹数量, 目的地等属性展示了内部类的应用

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

官方给的例子有点不太好记忆, 本人更倾向于简单的直接 Outer/Inner 这中名字来命名 class

```java
public class Outer {
    class Inner {}

    public Inner getInner() {
        return new Inner();
    }

    public static void main(String[] args) {
        Outer.Inner inner = new Outer().getInner();
    }
}
```

## The link to the outer class

内部类最显著的特点: Inner class 创建的时候会持有一个外部类的引用, 概念上类似指针, 这使得他能没有限制的访问外部类成员变量和方法. 

示例说明:

我们声明了一个接口 Selector, 它定义了三个方法, 表示类似游标的能力, 这个接口可以让我们

1. 得到当前量
2. 判断是不是最后一个元素
3. 移动到下一个元素

Sequence(次序) 是一个可变长的数组容器, 实现了构造函数和 add() 方法. 通过构造函数我们可以指定他的容量, 通过 add() 方法可以向容器中添加元素. 

Sequence 中我们声明了一个内部类 SequenceSelector 实现了 Selector 接口. 通过这种组合方式, 我们把删选能力和容器分隔开, 避免让 Sequence 直接实现 Selector 这种在语义上有歧义的做法. 

重点: SequenceSelector **可以访问** Sequence 的**私有**变量而不受限制

PS: 看了上面的这些描述, 这绝逼就是 Iterator 的概念, Iterator Pattern 章节的内容还历历在目 (●°u°●)​ 」

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

每次我们调用 selector() 方法时都会产生一个内部类的实体, 而且各个实体之间是相互独立的, 很赞. 

自己写的内部类持有外部引用的例子, 光这个点的话, 官方的例子有点累赘, 不过这种设计思路很喜欢

```java
public class Outer {
    private String name = "outer";
    class Inner {
        public String getOuterName() {
            return Outer.this.name;
        }
    }

    public Inner getInner() {
        return new Inner();
    }

    public static void main(String[] args) {
        Outer.Inner inner = new Outer().getInner();
        System.out.println(inner.getOuterName());
    }
}
// output: outer
```

## Using .this and .new

内部类中, 你可以使用 `外部类.this` 的方式得到外部类的引用. 下面的例子中, inner class 的 `outer()` 通过 `DotThis.this` 返回了外部类的引用, 并调用 `f()` 打印结果. 

这个例子中的调用链有点别扭, 但是主旨是为了说明我们可以通过 `outer.this` 这个关键字拿到外部类的引用, 仅此而已. 

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

如果你想创建内部类, 那么你可以通过 `外部类实例.new` 的形式创建. 

创建时你不需要为 `Inner()` 指定前缀 class, 这个挺方便的. 本来还以为需要用 `dn.new DotNew.Inner();` 的语法, 后来试过发现编译会报错. 

```java
public class DotNew {
    public class Inner {}

    public static void main(String[] args) {
        DotNew dn = new DotNew();
        DotNew.Inner dni = dn.new Inner();
    }
}
```

PS: \[Attention\] 书上将非静态内部类叫做 inner class, 静态内部类叫做 nested class 或 static inner class, 有点意思

想要创建内部类你必须要先创建外部类, 这是因为**创建内部类需要外部类的引用**, 这更像是一个先决条件. 如果想脱钩, 可以使用 nested class(静态内部类). 

## Inner classes and upcasting

标题可以理解为 内部类和强转. Inner class 和接口结合, 可以达到隐藏自己实现的目的, 这个特性很厉害. 

示例说明:

下面的例子里 Parcel4 声明了两个内部类, 分别实现 Destination 和 Contents 接口, 而且类修饰符为 private 和 protect 限制的包外的访问. 

然后按照常用套路, 为这两个内部类创建了方法(destination(str) 和 contents())返回对应的实现. 但是由于方法返回的是接口类型的, 所以包外压根就不知道他的实现细节. 这就很 imba, 起到了很强的隔离效果 （；￣ェ￣）

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
> 看调用方式, 如果是外部类方法直接调用内部类成员变量, 不能, 外部类实例化后, 内部类可能压根就没有实例化, 访问个毛线
> 如果是实例化了, 就可以调用, 习题答案如下

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

前面那些例子都很直白易懂, 但是 Inner class 还有一些变种, 格式很放飞自我, 该变种适用如下情况

1. 你只是想要实现某个接口, 并返回这个接口引用
2. 你在解决某个复杂问题时, 临时需要创建一个 class 以解决问题, 但是不想暴露它的实现

下面我们会将前面的 Parcel 例子转化为以下几种方式:

1.A class defined within a method - 在方法体内定义类
2.A class defined within a scope inside a method - 在方法的某个更小的 scope 中声明类, 比如方法的 if 条件语句中
3.An anonymous class implementing an interface - 匿名内部类实现接口
4.An anonymous class extending a class that has a non-default constructor - 匿名内部类继承抽象类 + 自定义构造函数
5.An anonymous class that performs field initialization  - 匿名内部类 + field 初始化
6.An anonymous class that performs construction using instance initialization (anonymous inner classes cannot have constructors) - 匿名内部类 + 构造代码块

对应 item1: A class defined within a method, 我们将 class 创建在方法体内部, 这种做法也叫 本地内部类(local inner class):

```java
public class Parcel5 {
    public Destination destination(String s) {
        class PDestination implements Destination {
            private String label;

            private PDestination(String whereTo) {
                label = whereTo;
            }

            public String readLabel() {
                return label;
            }
        }
        return new PDestination(s);
    }

    public static void main(String[] args) {
        Parcel5 p = new Parcel5();
        Destination d = p.destination("Tasmania");
    }
}
```

PDestination 在 destination() 方法内而不在 Parcels 内, 所以 PDestination 只在方法体 destination() 内可见. 这种用法还允许你在这个类的其他方法中创建**同名**的内部类而没有冲突. 

对应 item2: A class defined within a scope inside a method, 在方法内更小的 scope 中创建内部类:  

```java
public class Parcel6 {
    private void internalTracking(boolean b) {
        if (b) {
            class TrackingSlip {
                private String id;

                TrackingSlip(String s) {
                    id = s;
                }

                String getSlip() {
                    return id;
                }
            }
            TrackingSlip ts = new TrackingSlip("slip");
            String s = ts.getSlip();
        }
        // Can’t use it here! Out of scope:
        // ! TrackingSlip ts = new TrackingSlip("x");
    }

    public void track() {
        internalTracking(true);
    }

    public static void main(String[] args) {
        Parcel6 p = new Parcel6();
        p.track();
    }
}
```

TrackingSlip 嵌在 `if` 语句中, 只在 `if` 里生效, 出了这个范围就失效了, 除此之外和其他内部类没什么区别. 

## Anonymous inner classes

对应 item3: An anonymous class implementing an interface 定义匿名内部类 和 item4: An anonymous class extending a class that has a non-default constructor 使用默认构造函数

```java
interface Contents {
    int value();
}

public class Parcel7 {
    public Contents contents() {
        return new Contents() {
            // Insert a class definition
            private int i = 11;

            public int value() {
                return i;
            }
        };// Semicolon required in this case
    }

    public static void main(String[] args) {
        Parcel7 p = new Parcel7();
        Contents c = p.contents();
    }
}
```

`contents()` 将类定义和 return 结合在了一起. 除此之外, 该类还是匿名的, 返回时该类自动转换为基类类型. 上面的实现和下面的是等价的, 不过上面的更简洁. 

```java
public class Parcel7b {
    class MyContents implements Contents {
        private int i = 11;

        public int value() {
            return i;
        }
    }

    public Contents contents() {
        return new MyContents();
    }

    public static void main(String[] args) {
        Parcel7b p = new Parcel7b();
        Contents c = p.contents();
    }
}
```

看文章顺序这个应该是对应 item4: An anonymous class extending a class that has a non-default constructor 的但是总感觉他这种说法不贴切, 可能是我笔记有问题, 按理说, 下面的 instance initialization 更贴切才对. 

上面例子中, 内部类使用默认构造函数实例化, 如果你需要一个特殊的构造函数, 你可以参考下面的例子. Wrapping 是一个普通的类, 我们在 Parcel8 中的 wrapping 方法中调用了 Wrapping 的带参构造函数, 并且返回时重写了其中的 value 方法. 和之前的那些返回内部类的方式异曲同工. 

```java
public class Parcel8 {
    public Wrapping wrapping(int x) {
        // Base constructor call:
        return new Wrapping(x) { 
            // Pass constructor argument.
            public int value() {
                return super.value() * 47;
            }
        }; // Semicolon required
    }

    public static void main(String[] args) {
        Parcel8 p = new Parcel8();
        Wrapping w = p.wrapping(10);
        System.out.println(wrapping.value());
    }
}

public class Wrapping {
    private int i;

    public Wrapping(int x) {
        i = x;
    }

    public int value() {
        return i;
    }
}

// output: 470
```

对应 item5: An anonymous class that performs field initialization 你可以在内部类中定义, 使用 field, field 如果是作为参数传入, 必须是 final 类型的:

> 再看一遍才发现, 他的特殊之处是内部类有一个 field 声明, 对应的值是直接从方法参数里面拿的！！这种用法以前没注意到过 （；￣ェ￣）

```java
public class Parcel9 {
    // Argument must be final to use inside
    // anonymous inner class:
    public Destination destination(final String dest) {
        return new Destination() {
            private String label = dest;

            public String readLabel() {
                return label;
            }
        };
    }

    public static void main(String[] args) {
        Parcel9 p = new Parcel9();
        Destination d = p.destination("Tasmania");
    }
}
```

If you’re defining an anonymous inner class and want to use an object that’s defined outside the anonymous inner class, the compiler requires that the argument reference be **final**, as you see in the argument to destination(). If you forget, you’ll get a compile-time error message. 

内部匿名类会调用基类的构造器, 但是如果你在实例里需要定制一些行为, 但是由于你没有名字, 没有自己的构造器, 那该怎么办？

对应 item6: An anonymous class that performs construction using instance initialization, 这种情况下, 你可以使用 构造代码块(instance initializaiton) 实现通用的功能. 

```java
abstract class Base {
    public Base(int i) {
        System.out.println("Base constructor, i = " + i);
    }

    public abstract void f();
}

public class AnonymousConstructor {
    public static Base getBase(int i) {
        return new Base(i) {
            {
                System.out.println("Inside instance initializer");
            }

            public void f() {
                System.out.println("In anonymous f()");
            }
        };
    }

    public static void main(String[] args) {
        Base base = getBase(47);
        base.f();
    }
}

// output:
// Base constructor, i = 47
// Inside instance initializer
// In anonymous f()
```

上例中 i 作为构造器参数传入, 但是并没有在内部类中被直接使用, 使用他的是基类的构造函数. 所以不用像前面的 local inner class 那样, 使用 final 修饰. 

Note that the arguments to destination() must be final since they are used within the anonymous class:

```java
public class Parcel10 {
    public Destination destination(final String dest, final float price) {
        return new Destination() {
            private int cost;

            // Instance initialization for each object:      
            {
                cost = Math.round(price);
                if (cost > 100) System.out.println("Over budget!");
            }

            private String label = dest;

            public String readLabel() {
                return label;
            }
        };
    }

    public static void main(String[] args) {
        Parcel10 p = new Parcel10();
        Destination d = p.destination("Tasmania", 101.395F);
    }
}
// output: Over budget!
```

在内部类的使用中, 代码块可以看作是内部类的构造函数

和其他普通的类相比, 你可以使用匿名内部类来扩展类或接口, 但只能选其一, 而且数量只能是一个. 

### Factory Method revisited

> 这部分是使用 inner class 重构之前 factory/interface 相关的代码, 有机会回头再瞅一眼

Look at how much nicer the interfaces/Factories.java example comes out when you use anonymous inner classes:

示例说明:

通过内部类实现工厂方法, 并且在示例中将 outer 类的构造函数设置成 private, 将新建的动作限制到只能通过 factory 实现, 6 的飞起. 

```java
interface Service {
    void method1();

    void method2();
}

interface ServiceFactory {
    Service getService();
}

class Implementation1 implements Service {
    private Implementation1() {}

    public void method1() {
        System.out.println("Implementation1 method1");
    }

    public void method2() {
        System.out.println("Implementation1 method2");
    }

    public static ServiceFactory factory = new ServiceFactory() {
        public Service getService() {
            return new Implementation1();
        }
    };
}

class Implementation2 implements Service {
    private Implementation2() {}

    public void method1() {
        System.out.println("Implementation2 method1");
    }

    public void method2() {
        System.out.println("Implementation2 method2");
    }

    public static ServiceFactory factory = new ServiceFactory() {
        public Service getService() {
            return new Implementation2();
        }
    };
}

public class Factories {
    public static void serviceConsumer(ServiceFactory fact) {
        Service s = fact.getService();
        s.method1();
        s.method2();
    }

    public static void main(String[] args) {
        serviceConsumer(Implementation1.factory);
        // Implementations are completely interchangeable:
        serviceConsumer(Implementation2.factory);
    }
}

// output:
// Implementation1 method1
// Implementation1 method2
// Implementation2 method1
// Implementation2 method2
```

通过为 Factory 提供 inner class 的实现, 我们可以将上例中的 Implementation1 和 Implementation2 的构造函数设置成私有, 缩小了 Service 实现的作用域. 同时不需要为工厂类提供单独的实现. 从语法上这样的解决方案更合理. 

interfaces/Games.java 的例子也可以使用 inner class 做类似的优化:

```java
interface Game {
    boolean move();
}

interface GameFactory {
    Game getGame();
}

class Checkers implements Game {
    private Checkers() {
    }

    private int moves = 0;
    private static final int MOVES = 3;

    public boolean move() {
        System.out.println("Checkers move " + moves);
        return ++moves != MOVES;
    }

    public static GameFactory factory = new GameFactory() {
        public Game getGame() {
            return new Checkers();
        }
    };
}

class Chess implements Game {
    private Chess() {
    }

    private int moves = 0;
    private static final int MOVES = 4;

    public boolean move() {
        System.out.println("Chess move " + moves);
        return ++moves != MOVES;
    }

    public static GameFactory factory = new GameFactory() {
        public Game getGame() {
            return new Chess();
        }
    };
}

public class Games {
    public static void playGame(GameFactory factory) {
        Game s = factory.getGame();
        while (s.move()) ;
    }

    public static void main(String[] args) {
        playGame(Checkers.factory);
        playGame(Chess.factory);
    }
}
```

Remember the advice given at the end of the last chapter: Prefer classes to interfaces. If your design demands an interface, you’ll know it. Otherwise, don’t put it in until you are forced to. 

> 这个建议是从上一章节 Interface 那边出来了, 具体得完那一章才知道. 建议就是先用 class, 等你完全定下来再 refactor 成 interface, 现在 interface 一般都是被滥用的. 

## Nested classes 

如果你不想要内部类和外部类的关系, 你可以把内部类静态化, 这种做法叫 nested class(静态内部类). 普通的内部类会持有一个外部类的引用, 静态内部类则不会. 静态内部类有如下特点:

1. You don’t need an outer-class object in order to create an object of a nested class. 独立于外部类实例存在
2. You can’t access a non-static outer-class object from an object of a nested class. 不能通过它访问非静态的外部类
 
除此之外的区别还有, 普通内部类还不能持有静态变量, 方法. 

```java
public class Parcel11 {
    private static class ParcelContents implements Contents {
        private int i = 11;

        public int value() {
            return i;
        }
    }

    protected static class ParcelDestination implements Destination {
        private String label;

        private ParcelDestination(String whereTo) {
            label = whereTo;
        }

        public String readLabel() {
            return label;
        }        

        // Nested classes can contain other static elements:
        public static void f() {
        }

        static int x = 10;

        static class AnotherLevel {
            public static void f() {
            }

            static int x = 10;
        }
    }

    public static Destination destination(String s) {
        return new ParcelDestination(s);
    }

    public static Contents contents() {
        return new ParcelContents();
    }

    public static void main(String[] args) {
        Contents c = contents();
        Destination d = destination("Tasmania");
    }
}
```

由于使用了静态的内部类, 外部类也可以使用静态方法返回内部类实例. 在 main() 中调用时就可以直接 call 方法而不用外部类实例了. 

### Classes inside interfaces 

一般来说, 在 interface 里放 class 是不允许的, 但是 nested class 是个例外. 任何放到 interface 里的 code 都会有 public 和 static 的属性, 所以下面代码中声明的 class `class Test implements ClassInInterface` 其实就是一个静态内部类. You can even implement the surrounding interface in the inner class, like this: 

```java
public interface ClassInInterface {
    void howdy();

    class Test implements ClassInInterface {
        public void howdy() {
            System.out.println("Howdy!");
        }

        public static void main(String[] args) {
            new Test().howdy();
        }
    }
}

// output Howdy!
```

通过这种方式我们可以很方便的在接口使用方分享一些公用代码. 

在这本书的前面几章, 有建议说在每个 class 里面加一个 main() 方法来存放测试代码, 但是这会增加需要编译的代码量. 这里我们可以将测试放到 nested class 中:

```java
public class TestBed {
    public void f() {
        System.out.println("f()");
    }

    public static class Tester {
        public static void main(String[] args) {
            TestBed t = new TestBed();
            t.f();
        }
    }
}
// output f()
```

编译之后测试会放到单独的 class `TestBed$Tester` 中, 它可以用来测试, 当要部署到产品环境时, 可以把这部分代码 exclude 掉. 

> 现在应该不用了, 我们都是通过在测试 folder 下新建测试 UT 来完成这部分功能的

### Reaching outward from a multiply nested class

不管 inner class 嵌套的有多深, 内部类都可以不受限制的访问外部类, 如下:

```java
class MNA {
    private void f() {
    }

    class A {
        private void g() {
        }

        public class B {
            void h() {
                g();
                f();
            }
        }
    }
}

public class MultiNestingAccess {
    public static void main(String[] args) {
        MNA mna = new MNA();
        MNA.A mnaa = mna.new A();
        MNA.A.B mnaab = mnaa.new B();
        mnaab.h();
    }
} 
```

上例中MNAAB 可以访问外部的私有方法 g(), f(). 同时也演示了, 在 main() 中你如果要新建内部类, 需要先实例化他的外部类. 

## Why inner classes?

为什么 Java 要支持 inner class 这种语法？

从典型的使用方式上看, 内部类会继承 class 或者 实现接口, 然后操作外部类的属性. 所以我们可以说**内部类提供了一个外部类的访问窗口**. 

Inner class 存在的最合理的解释:

> 内部类都可以独立的实现一个继承. 即不管外部类是否已经继承了一个实现这对 inner class 毫无影响. 

换个角度看, inner class 可以看作是**多重继承**的一种解决方案. 在这方面, interface 可以解决一部分问题, 但是 inner class 效率更高. 

就上面的问题, 下面我们举例子来说明, 比如我们想要在一个类里实现两个接口, 你有两种选择, 一个 class + 2*interface 或者 class + inner class + 1*interface

```java
interface A {}

interface B {}

class X implements A, B {}

class Y implements A {
    B makeB() {
        // Anonymous inner class:
        return new B() {};
    }
}

public class MultiInterfaces {
    static void takesA(A a) {}

    static void takesB(B b) {}

    public static void main(String[] args) {
        X x = new X();
        Y y = new Y();
        takesA(x);
        takesA(y);
        takesB(x);
        takesB(y.makeB());
    }
}
```

示例中我们有 A, B 两个接口, X 实现两个接口, Y 实现一个接口 + 一个 inner class. X, Y 虽然实现方式不太一样, 但是目的都达到了, 两个接口都实现了. 

但是, 如果是抽象类或者实体类, 多重继承就会受到限制. 

```java
class D {}

abstract class E {}

class Z extends D {
    E makeE() {
        return new E() {
        };
    }
}

public class MultiImplementation {
    static void takesD(D d) {
    }

    static void takesE(E e) {
    }

    public static void main(String[] args) {
        Z z = new Z();
        takesD(z);
        takesE(z.makeE());
    }
}
```

> 作者这里的继承说的是具有基类的某种能力, 而不是限制在继承类的语法表现, 这个对我理解继承还是有点启发的. 通过**内部类**我可以得到**基类**的实例, 说我继承了它, 也说的过去. 

通过 inner class, 你可以具备以下附加功能:

1. 内部类可以有多个实例, 并且相互独立, 和外部类也相互独立
2. In a single outer class you can have several inner classes, each of which implements the same interface or inherits from the same class in a different way. An example of this will be shown shortly.
3. The point of creation of the inner-class object is not tied to the creation of the outer-class object.  
4. There is no potentially confusing "is-a" relationship with the inner class; it’s a separate entity. 

就第四点, 可以那前面的 `Sequence.java` 为例. Sequence 语义上来说是一个容器, 而 Selector 接口代表了选择这种能力. 我们通过内部创建一个 SequenceSelector 实现这中能力, 在语义上会更合理. 

### Closures & callbacks

Closure(闭包) 即一个可调用对象, 保留了创建它的作用域的信息. Inner class 就是 OO 概念上的一个闭包, 他持有外部类的引用, 访问不受限. 

Java 支持部分指针机制, 其中之一就是 callback(回调). 在回调中, 一些对象给出自身的一部分信息(引用), 通过这部分信息, 其他对象可以操作这个对象. 

inner class 的闭包特性比之与指针, 扩展性更强, 更安全. 

示例说明:

下面这个例子只为了一个目的, 就是凸显出内部类可以拿到外部类的引用(Callee2.this), 并且没有任何访问限制. 

我们声明一个 Incremnentable 接口, 其中有一个方法 `increment()`, Callee1 实现了 Incrementable. 

再声明 MyIncrement 基类, 也有一个 `increment()` 方法, 然后声明 Callee2 继承了 MyIncrement 那么相应的他就自带了 increment() 方法, 无法再实现 Incrementable 接口, 这里通过内部类 Closure 实现接口, 在通过 getCallbackReference() 拿到引用, 变相的达到了多重继承的效果. 在主函数中, Caller 通过构造函数统一对 Incrementable 做操作. 

PS:个人感觉这个例子中 MyIncrement 这个类对说明 callback 这个特性反而起了误导的作用, 让整个示例反觉更繁琐了. 整个例子只需要保留 Callee2 + Caller 部分即. 我们可以通过 caller 的 go() 方法调用 Callee2 中的方法, 改变内部变量值. 

PPS: 这个例子确实多余, 这里表现出来的特性不就是 `Outer.this` 这个属性吗, 绕了一大圈. 

```java
interface Incrementable {
    void increment();
}

// Very simple to just implement the interface:
class Callee1 implements Incrementable {
    private int i = 0;

    public void increment() {
        i++;
        System.out.println(i);
    }
}

class MyIncrement {
    public void increment() {
        System.out.println("Other operation");
    }

    static void f(MyIncrement mi) {
        mi.increment();
    }
}

// If your class must implement increment() in
// some other way, you must use an inner class:
class Callee2 extends MyIncrement {
    private int i = 0;

    @Override
    public void increment() {
        super.increment();
        i++;
        System.out.println(i);
    }

    private class Closure implements Incrementable {
        public void increment() {
            // Specify outer-class method, otherwise
            // you’d get an infinite recursion:
            Callee2.this.increment();
        }
    }

    Incrementable getCallbackReference() {
        return new Closure();
    }
}

class Caller {
    private Incrementable callbackReference;

    Caller(Incrementable cbh) {
        callbackReference = cbh;
    }

    void go() {
        callbackReference.increment();
    }
}

public class Callbacks {

    public static void main(String[] args) {
        Callee1 c1 = new Callee1();
        Callee2 c2 = new Callee2();
        MyIncrement.f(c2);
        Caller caller1 = new Caller(c1);
        Caller caller2 = new Caller(c2.getCallbackReference());
        caller1.go();
        caller1.go();
        caller2.go();
        caller2.go();
    }
}

// output
// Other operation
// 1
// 1
// 2
// Other operation
// 2
// Other operation
// 3
```

### Inner classes & control frameworks

> List<Event> (pronounced "List of Event") 原来带类型的 collection 这么发音的吗, 学到了同时感觉很合理

> 本章主要例子中用到了 Command pattern 不过我已经忘了那是个什么东西了, 又要复习了 （；￣ェ￣）

control framework 是一种用于处理 event 的应用框架. 下面是书中 GreenHouse 的例子. 我们先创建一个 abstract 的类代表我们要处理的 event

```java
public abstract class Event {
    private long eventTime;
    protected final long delayTime;

    public Event(long delayTime) {
        this.delayTime = delayTime;
        start();
    }

    public void start() { // Allows restarting
        eventTime = System.nanoTime() + delayTime;
    }

    public boolean ready() {
        return System.nanoTime() >= eventTime;
    }

    public abstract void action();
}
```

`start()` 单独抽离, 方便以后实现 restart 功能, `ready()` 即判断是否已经可以执行事件, `action()` 是我们要执行事件的内容. 

以下是 Controller 代码, Controller 实体持有事件列表, 然后通过 while 遍历 event 并执行. 处理时将变量 list 备份以防止动态改变 list 的值. 

```java
public class Controller {
    // A class from java.util to hold Event objects:
    private List<Event> eventList = new ArrayList<Event>();

    public void addEvent(Event c) {
        eventList.add(c);
    }

    public void run() {
        while (eventList.size() > 0)
            // Make a copy so you’re not modifying the list
            // while you’re selecting the elements in it:
            for (Event e : new ArrayList<Event>(eventList))
                if (e.ready()) {
                    System.out.println(e);
                    e.action();
                    eventList.remove(e);
                }
    }
}
```

在遍历 event 时, 我们并不知道 event 具体是什么, 这正是框架的目的, 我们并不关心某个具体的对象. 而这恰恰是 inner class 擅长的地方. 通过使用它我们可以在两方面优化上面的代码. 

1. 我们可以把 event 和 controller 合二为一, 将各个 event 特有的 action() 封装在内部类中
2. 内部类让你的实现对外不可见

使用**内部类**实现代码如下

```java
public class GreenhouseControls extends Controller {
    private boolean light = false;

    public class LightOn extends Event {
        public LightOn(long delayTime) {
            super(delayTime);
        }

        public void action() {
            // Put hardware control code here to
            // physically turn on the light.
            light = true;
        }

        public String toString() {
            return "Light is on";
        }
    }

    public class LightOff extends Event {
        public LightOff(long delayTime) {
            super(delayTime);
        }

        public void action() {
            // Put hardware control code here to
            // physically turn off the light.
            light = false;
        }

        public String toString() {
            return "Light is off";
        }
    }

    private boolean water = false;

    public class WaterOn extends Event {
        public WaterOn(long delayTime) {
            super(delayTime);
        }

        public void action() {
            // Put hardware control code here.
            water = true;
        }

        public String toString() {
            return "Greenhouse water is on";
        }
    }

    public class WaterOff extends Event {
        public WaterOff(long delayTime) {
            super(delayTime);
        }

        public void action() {
            // Put hardware control code here.
            water = false;
        }

        public String toString() {
            return "Greenhouse water is off";
        }
    }

    private String thermostat = "Day";

    public class ThermostatNight extends Event {
        public ThermostatNight(long delayTime) {
            super(delayTime);
        }

        public void action() {
            // Put hardware control code here.
            thermostat = "Night";
        }

        public String toString() {
            return "Thermostat on night setting";
        }
    }

    public class ThermostatDay extends Event {
        public ThermostatDay(long delayTime) {
            super(delayTime);
        }

        public void action() {
            // Put hardware control code here.
            thermostat = "Day";
        }

        public String toString() {
            return "Thermostat on day setting";
        }
    }

    // An example of an action() that inserts a
    // new one of itself into the event list:
    public class Bell extends Event {
        public Bell(long delayTime) {
            super(delayTime);
        }

        public void action() {
            addEvent(new Bell(delayTime));
        }

        public String toString() {
            return "Bing!";
        }
    }

    public class Restart extends Event {
        private Event[] eventList;

        public Restart(long delayTime, Event[] eventList) {
            super(delayTime);
            this.eventList = eventList;
            for (Event e : eventList)
                addEvent(e);
        }

        public void action() {
            for (Event e : eventList) {
                e.start(); // Rerun each event
                addEvent(e);
            }
            start(); // Rerun this Event
            addEvent(this);
        }

        public String toString() {
            return "Restarting system";
        }
    }

    public static class Terminate extends Event {
        public Terminate(long delayTime) {
            super(delayTime);
        }

        public void action() {
            System.exit(0);
        }

        public String toString() {
            return "Terminating";
        }
    }
}
```

代码结构很简单, 分别声明了一些事件类型 lightOn/Off, waterOn/Off 等, 内部类继承 Event, 实现个则的抽象方法即可. 

Bell 和 Restart 有别于其他的 event 内部类, 它还会调用 Outer class 的其他方法. 

以下是 GreenhouseController 执行函数

```java
public class GreenhouseController {
    public static void main(String[] args) {
        GreenhouseControls gc = new GreenhouseControls();
        // Instead of hard-wiring, you could parse
        // configuration information from a text file here:
        gc.addEvent(gc.new Bell(900));
        Event[] eventList = {
                gc.new ThermostatNight(0), 
                gc.new LightOn(200), 
                gc.new LightOff(400), 
                gc.new WaterOn(600), 
                gc.new WaterOff(800), 
                gc.new ThermostatDay(1400)
        };
        gc.addEvent(gc.new Restart(2000, eventList));
        gc.addEvent(new GreenhouseControls.Terminate(new Integer(5000)));
        gc.run();
    }
}
// output:
// Bing!
// Thermostat on night setting
// Light is on
// Light is off
// Greenhouse water is on
// Greenhouse water is off
// Thermostat on day setting
// Restarting system
// Terminating
```

## Inheriting from inner classes

如果想要继承一个内部类, 语法稍微有点特殊, 由于内部类需要借助外部类才能实例化, 所以构造函数中需要调用 `outer.super()` 实例如下:

```java
class WithInner {
    class Inner {
    }
}

public class InheritInner extends WithInner.Inner {
    //! InheritInner() {} // Won’t compile
    InheritInner(WithInner wi) {
        wi.super();
    }

    public static void main(String[] args) {
        WithInner wi = new WithInner();
        InheritInner ii = new InheritInner(wi);
    }
}
```

InheritInner 继承自内部类, 在构造函数中需要外部类实体做参数. 内部类是以外部类为基础的, 所以这样做也挺合理.

## Can inner classes be overridden?

内部类并不能像方法那样被重写. 我们准备一个 class Egg, 里面声明一个内部类 Yolk 并在构造函数中调用它. 我们再新建一个类 GigEgg 继承 Egg, 在里面声明一个同名的内部类, 试图用类似方法重写的方式覆盖他. 示例如下:

```java
class Egg {
    private Yolk y;

    protected class Yolk {
        public Yolk() {
            System.out.println("Egg.Yolk()");
        }
    }

    public Egg() {
        System.out.println("New Egg()");
        y = new Yolk();
    }
}

public class BigEgg extends Egg {
    public class Yolk {
        public Yolk() {
            System.out.println("BigEgg.Yolk()");
        }
    }

    public static void main(String[] args) {
        new BigEgg();
    }
}

// output:
// New Egg()
// Egg.Yolk()
```

默认的构造函数会在编译时指定调用基类中的 Yolk 对象. 这个例子表明 JVM 在处理内部类时并没有做什么特殊的操作, 基类和子类中的内部函数时完全隔离的. 

This example shows that there isn’t any extra inner-class magic going on when you inherit
from the outer class. The two inner classes are completely separate entities, each in its own
namespace. However, it’s still possible to explicitly inherit from the inner class: 

```java
class Egg2 {
    protected class Yolk {
        public Yolk() {
            System.out.println("Egg2.Yolk()");
        }

        public void f() {
            System.out.println("Egg2.Yolk.f()");
        }
    }

    private Yolk y = new Yolk();

    public Egg2() {
        System.out.println("New Egg2()");
    }

    public void insertYolk(Yolk yy) {
        y = yy;
    }

    public void g() {
        y.f();
    }
}

public class BigEgg2 extends Egg2 {
    public class Yolk extends Egg2.Yolk {
        public Yolk() {
            System.out.println("BigEgg2.Yolk()");
        }

        public void f() {
            System.out.println("BigEgg2.Yolk.f()");
        }
    }

    public BigEgg2() {
        insertYolk(new Yolk());
    }

    public static void main(String[] args) {
        Egg2 e2 = new BigEgg2();
        e2.g();
    }
}

// output
// Egg2.Yolk() <- 初始化子类时调用基类构造, 先初始化基类中的 field
// New Egg2() <- 基类构造
// Egg2.Yolk() <- 子类 new Yolk() 先调用 基类 中的 Yolk 构造
// BigEgg2.Yolk() <- 子类构造调用
// BigEgg2.Yolk.f() <- 子类调用 g 方法
```

在上面的例子里面, 我们显示的指定 BigEgg2 中的 Yolk 继承自 Egg2 中的 Yolk, 然后基类中还提供了一个 `insertYolk()` 来修改基类中内部类的引用. 

## Local inner classes

内部类可以创建在代码块中, 一般常见的是创建在方法里面. 我们无法访问方法体里面的内部类, 因为他并不是 outer class 的一部分. 但是这个内部类还是可以毫无限制的访问外部类的各种信息. 

下面是 local inner class 和匿名内部类的对比例子:

```java
interface Counter {
    int next();
}

public class LocalInnerClass {
    private int count = 0;

    Counter getCounter(final String name) {
        // A local inner class:
        class LocalCounter implements Counter {
            public LocalCounter() {
                // Local inner class can have a constructor
                System.out.println("LocalCounter()");
            }

            public int next() {
                System.out.print(name); // Access local final
                return count++;
            }
        }
        return new LocalCounter();
    }

    // The same thing with an anonymous inner class:
    Counter getCounter2(final String name) {
        return new Counter() {
            // Anonymous inner class cannot have a named
            // constructor, only an instance initializer:
            {
                System.out.println("Counter()");
            }

            public int next() {
                System.out.print(name); // Access local final
                return count++;
            }
        };
    }

    public static void main(String[] args) {
        LocalInnerClass lic = new LocalInnerClass();
        Counter
                c1 = lic.getCounter("Local inner "), 
                c2 = lic.getCounter2("Anonymous inner ");
        for (int i = 0; i < 5; i++)
            System.out.println(c1.next());
        for (int i = 0; i < 5; i++)
            System.out.println(c2.next());
    }
} 

// output
// LocalCounter()
// Counter()
// Local inner 0
// Local inner 1
// Local inner 2
// Local inner 3
// Local inner 4
// Anonymous inner 5
// Anonymous inner 6
// Anonymous inner 7
// Anonymous inner 8
// Anonymous inner 9
```

上面的例子中, Counter 接口会依次返回 count 值. local inner class 和 匿名内部类都实现了这个接口. 两个内部类逻辑和功能也都一样, 唯一区别是, 匿名内部类他是没有构造函数的, 需要用代码块代替. 

如果你需要创建多个实例的话, 你也要使用 local inner class, 你用 anonymous 是建不出来多个实例的. 

## Inner-class identifiers

每个类在编译后都会生成一个 `.class` 文件保存对应的类信息. 内部类也一样, 格式为 `外部类$内部类` 下面是 LocalInnerClass.java 编译后的文件:

```txt
Counter.class
LocalInnerClass$l.class
LocallnnerClassSlLocalCounter.class
LocallnnerClass.class
```

如果是内部匿名类, 类名由数字代替. 如果是多层嵌套的内部类, 类名间链接多个 `$` 符号. 

## Summary

接口和内部类是 Java 特有的, 你在 C++ 中找不到类似的概念, 他们帮助我们实现多重继承的问题而且实现上要比 C++ 的优雅. 

## 实践出真知

> 2021-04-22 想要在代码中使用简化版的 Builder 模式，但是发现 nested static class 中声明的对象不能调用 set method. 为啥？

```java
public class Person {
    private String name;

    public void setName(String name) {
        this.name = name;
    }

    public static class Builder {
        Person p = new Person();
        //? p.name = "jack"; 不能操作

        public Builder setName(String name) {
            p.name = name;   // 可操作
            // p.setName(name); 可操作
            return this;
        }

        public Person build() {
            return p;
        }
    }
}
```
