---
title: TIJ4 interfaces
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

Interfaces may be nested within classes and within other interfaces. 3 This reveals a number
of interesting features:

```java
package reading.container;

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

The syntax for nesting an interface within a class is reasonably obvious. Just like non-nested
interfaces, these can have public or package-access visibility.

As an added twist, interfaces can also be private, as seen in A.D (the same qualification
syntax is used for nested interfaces as for nested classes). What good is a private nested
interface? You might guess that it can only be implemented as a private inner class as in
DImp, but A.DImp2 shows that it can also be implemented as a public class. However,
A.DImp2 can only be used as itself. You are not allowed to mention the fact that it
implements the private interface D, so implementing a private interface is a way to force
the definition of the methods in that interface without adding any type information (that is,
without allowing any upcasting).

The method getD( ) produces a further quandary concerning the private interface: It’s a
public method that returns a reference to a private interface. What can you do with the
return value of this method? In main( ), you can see several attempts to use the return
value, all of which fail. The only thing that works is if the return value is handed to an object
that has permission to use it—in this case, another A, via the receiveD( ) method.

Interface E shows that interfaces can be nested within each other. However, the rules about
interfaces—in particular, that all interface elements must be public—are strictly enforced
here, so an interface nested within another interface is automatically public and cannot be
made private. 

Nestinglnterfaces shows the various ways that nested interfaces can be implemented. In
particular, notice that when you implement an interface, you are not required to implement
any interfaces nested within. Also, private interfaces cannot be implemented outside of
their defining classes.

Initially, these features may seem like they are added strictly for syntactic consistency, but I
generally find that once you know about a feature, you often discover places where it is
useful. 
