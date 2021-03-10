---
title: TIJ4 接口 读书笔记 
date: 2021-03-09 19:17:36
categories:
- TIJ4
tags:
- interface
---

TIJ4 中 Interfaces 章节读书笔记

- [Intro](#intro)
- [Nesting interfaces](#nesting-interfaces)

## Intro

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
