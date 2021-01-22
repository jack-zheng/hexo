---
title: TIJ4 通过异常处理错误 error handing with exceptions
date: 2021-01-21 13:49:44
categories:
- TIJ4
tags:
- exception
---

刚好这几天要修一个异常处理的问题，回顾以下异常相关的知识点

想要解决的问题：

[ ] runtime 异常和其他的异常有什么区别
[ ] 异常处理的最佳实践是什么
[ ] catch 里加一个 return 会怎么样

## 前述

Java 的一个基本规范：组织糟糕的代码不会被运行。最理想的异常捕捉点是编译器，代码运行之前。但是并不是所有的 errors 都能在编译器就被捕捉到。剩下的这些问题，我们需要在运行时让异常产生者将错误信息提交给接受者做适当的处理。

提升异常回复你是增加代码健壮性的重要方式。异常恢复是每个码农都需要考虑的问题，对于 Java 这种旨在为他人提供接口的语言来说，这个就尤为重要了。通过提供 error-reporting 这个错误处理模式，Java 允许组建代码将异常传给客户端代码处理。

Java 中的异常处理机制旨在以尽量少的代码完成经可能多的功能，并且同时让你的程序能覆盖尽可能多的异常情况。异常机制不难学，并且学会了他能马上对你的项目产生益处，因为他是唯一官方指定的处理异常的方式，并且是编译器强制检查的。

本章将介绍一些必须要用到异常的代码以及遇到异常时如何处理

## Concepts

C 和其他一些早期语言经常会有多中处理异常的方式，这些方式基本都是便宜形式，并且不是语言的一部分。典型的案例就是放回一个特殊的值或者设置一个特殊的 flag，接收方根据这个返回值来进行相应的处理。但是渐渐的码农们突然意识到，自己定义的异常情况可能是不充分的，并且为了覆盖这些没有覆盖到的情况，代码会变得越来越难以维护。

解决方案是将异常处理正规化，这种思路的出现是一个很长过的过程，可以追溯到 1960 年。

exception 表示：我处理这个异常是为了。。。当异常发生时，你可能并不清楚你需要做什么，但是你知道，你不能在正常的执行下去了，你应该做点什么。你就可以将这个异常提交给上层处理，那里可能具备足够的知识处理它。

异常处理机制的另一个明显的好处是，它可以简化你的异常处理代码。如果没有他你必须在多个地方编写检测代码。有了它之后，就不必这么做了，exception 会保证，有人在合适的地方做检查。你只需要在一个地方处理即可。这种机制简化了你的代码，代码被分为两个分之，正常的分支和异常分支，同时你阅读，书写，调试代码也会变的更方便。

## Basic exceptions

发生异常的条件下，程序会停止执行现有的方法。区分正常程序和异常程序是很重要的，当异常产生时，你的程序由于缺少一些信息，已经不能继续执行程序了，他能做的就是跳出当前的上下文并且将执行权交给上层。

除 0 就是这么一种情况，当代码中出现了除 0 的情况，就需要检查一下了。可能你知道为什么要除 0，可能是你业务逻辑的需要，你知道在这种情况下需要做什么。但是如果这是一个意外情况，你必须停止当前方法并且抛出一个异常。

当你跑出一个异常时，有几件事情会发生。首先和其他对象一样，一个异常对象会在堆上通过 new 的方式被创建出来。然后当前的执行路径被阻断，异常对象被当前上下问弹出。exception-handling 处理机制开始接收这个异常，并试图妥善的处理它。exception handler 就是处理异常的地方，他会判断是否继续执行，或者寻求其他解决路径。

想象一个简单的场景，比如你有一个对象引用叫做 `t`，他可能没有被初始化过，所以你想在使用前检查一下。你可以将检查的错误通过一个对象包裹起来并 thorwing 出去，这种做法就叫做跑出异常，代码表示如下：

```java
if(t == null)
 throw new NullPointerException();
```

这种做法可以让你为未来做打算，他会在之后的什么地方被处理，你很快能看到。

Exceptions 让你能够以 transaction 为单位处理问题， 你也可以将它想象成一个 undo 系统，你可以设置多个恢复点，当你的程序抛出异常时，他可以将程序恢复到某个稳定的节点。

Exceptions 最重要的一个点就是，当异常发生时，他阻止程序继续执行下去。这种情况在 C 总尤为明显，C 是没有打断机制的，这种情况下，程序可能进入到一个完全错误的状态。

## Exception arguments

和其他 Java 中的对象一样，你可以通过 new 在堆上创建一个 exception 对象，它有两种构造函数，一种是无参的，另一种是带字符串的，比如：

```java
throw new NullPointerException("t = null");
```

当然这个信息字符串也可以在之后通过调用方法设置，之后你会看到这种用法。

`throw` 这个关键词可以产生几种很有趣的结果。当你使用 new 创建一个 exception 的时候，你指定了 throw 的对象。虽然这个对象和你方法的返回值类型不一样，但是它还是会被这个方法返回。由此，你可以将异常处理看作一种特殊的 return 机制。返回的同时，方法和 scope 将会推出(出栈)。

和普通方法的共同点到此为止了，接下来的处理方式将迥异于普通方法。异常将会在 exception handler 中被处理。

虽然你可以在处理异常时抛出任何 Throwable 的子类，但是一般来说，我本会更具 error 的类型来指定它。error 的信息可以从他的名字和内容体现出来，但是更常见的情况是异常只包含名字而没有其他什么内容。

## Catching an exception

在理解异常捕捉之前，你先得理解**守护区域**的概念。它代表了一段可能抛出异常的代码段，这段代码之后会紧接着一段异常处理代码。

### The try block

如果你在方法体中抛出一个异常(或者方法体中调用的其他方法抛出异常)，那么这个方法体在执行完 throwing 之后就结束了。如果你不想就这个结束，你可以在这些代码外面加一个 try block

```java
try {
 // Code that might generate exceptions
}
```

在不提供异常处理机制的语言中，如果你写代码很仔细的话，你可能需要为每一个方法添加异常处理，但是通过 try block 你只需要将他们全部包裹起来即可。这样你的代码会更容易阅读。

### Exception handlers

当然，被抛出的异常都需要有一个地方来处理，这个地方就是 exception handler。它紧跟着 try block 通过关键字 catch 引出

```java
try {
 // Code that might generate exceptions
} catch(Type1 id1)|{
 // Handle exceptions of Type1
} catch(Type2 id2) {
 // Handle exceptions of Type2
} catch(Type3 id3) {
 // Handle exceptions of Type3
}
// etc...
```

每一个 catch 就是一个小的方法体，只接收一个参数。有时你甚至不需要这个参数，仅仅根据异常的名字就可以写完处理逻辑。

如果异常被抛出，exception-handling 机制会搜寻第一个匹配的 catch 分支并进入，当 catch 分支走完后，异常处理被视为结束。不像 switch，catch 分支不需要 break 关键字，执行完直接返回。

## Termination vs. resumption (中断还是继续？)

异常处理有两种模型，Java 采用的是中断，他认为当异常发生后，你不能在回到异常发生的节点。

另一种是 resumption(继续)，他表示异常发生后，我们可以做一些补救措施，并且尝试重新执行失败的方法。采用这个方式意味着你在异常产生后依旧希望继续执行程序。

如果你想要 resumption 的处理方法，你不能在 error 发生的地方抛异常，或者你可以把你抛异常的代码放到一个循环中，多次运行，知道结果符合你预期。

历史上，码农们有尝试过使用 resumption 机制的操作系统，但最终回归到了 termination 机制。虽然 resumption 机制乍一听上去很美，但是并不是这么实用。可能是应为这种机制下你写的代码不能很通用，导致维护困难，特别是在写一些大型项目的时候。

## Creating your own exceptions

Java 允许你自己定制异常，你需要做的只是继承一个已有的异常类即可。当然继承的时候如果有的话，选一个最贴近你异常类的，那是极好的。创建时只需要用它的默认构造函数即可，代码很简单：

```java
class SimpleException extends Exception {
}

public class InheritingExceptions {
    public void f() throws SimpleException {
        System.out.println("Throw SimpleException from f()");
        throw new SimpleException();
    }

    public static void main(String[] args) {
        InheritingExceptions sed = new InheritingExceptions();
        try {
            sed.f();
        } catch (SimpleException e) {
            System.out.println("Caught it!");
        }
    }
}
// output:
// Throw SimpleException from f()
// Caught it!
```

用以上的方式创建的异常会自带默认的构造函数，这个默认的构造函数是无参的，在自定义异常时，最重要的是要有个贴切的名字。

下面是带带参构造的例子

```java
class MyException extends Exception {
    public MyException() {
    }

    public MyException(String msg) {
        super(msg);
    }
}

public class FullConstructors {
    public static void f() throws MyException {
        System.out.println("Throwing MyException from f()");
        throw new MyException();
    }

    public static void g() throws MyException {
        System.out.println("Throwing MyException from g()");
        throw new MyException("Originated in g()");
    }

    public static void main(String[] args) {
        try {
            f();
        } catch (MyException e) {
            e.printStackTrace(System.out);
        }
        try {
            g();
        } catch (MyException e) {
            e.printStackTrace(System.out);
        }
    }
}
// output
// Throwing MyException from f()
// reading.container.MyException
// 	at reading.container.FullConstructors.f(FullConstructors.java:15)
// 	at reading.container.FullConstructors.main(FullConstructors.java:25)
// Throwing MyException from g()
// reading.container.MyException: Originated in g()
// 	at reading.container.FullConstructors.g(FullConstructors.java:20)
// 	at reading.container.FullConstructors.main(FullConstructors.java:30)
```

只需要少量的新加 code 就能实现带参构造，声明的时候用上 `super` 关键字即可。

在 exception handler 中你可以看到一个方法调用叫做 `e.printStackTrace(System.out)`。他可以将异常信息输出。

## Exercises

Exercise 1: (2) Create a class with a main( ) that throws an object of class Exception
inside a try block. Give the constructor for Exception a String argument. Catch the
exception inside a catch clause and print the String argument. Add a finally clause and
print a message to prove you were there.

```java
class Exe1Exception extends Exception {
    Exe1Exception(String msg) {
        super(msg);
    }
}

public class Exercise1 {
    public static void main(String[] args) {
        try {
            throw new Exe1Exception("my exception msg...");
        } catch (Exe1Exception e) {
            System.out.println(e.getMessage());
        } finally {
            System.out.println("Into final cluster...");
        }
    }
}

// output
// my exception msg...
// Into final cluster...
```

Exercise 2: (1) Define an object reference and initialize it to null. Try to call a method
through this reference. Now wrap the code in a try-catch clause to catch the exception.

```java
public class Exercise1 {
    public static void main(String[] args) {
        try {
            String str = null;
            System.out.println(str.isEmpty());
        } catch (NullPointerException e) {
            System.out.println("Invoked object is null...");
        }
    }
}
// output
// Invoked object is null...
```

Exercise 3: (1) Write code to generate and catch an
ArraylndexOutOfBoundsException.

```java
public class Exercise1 {
    public static void main(String[] args) {
        try {
            String[] strArr = new String[0];
            strArr[0] = "str";
        } catch (ArrayIndexOutOfBoundsException e) {
            System.out.println("ArrayIndexOutOfBoundsException caught...");
        }
    }
}
```

Exercise 4: (2) Create your own exception class using the extends keyword. Write a
constructor for this class that takes a String argument and stores it inside the object with a
String reference. Write a method that displays the stored String. Create a try-catch clause
to exercise your new exception.

```java
class ExeException extends Exception {
    ExeException(String msg) {
        super(msg);
    }

    public String getExeExceptionMsg() {
        return getMessage();
    }
}

public class Exercise1 {
    public static void main(String[] args) {
        try {
            throw new ExeException("exe exception...");
        } catch (ExeException e) {
            System.out.println("Caught ExeException, msg: " + e.getExeExceptionMsg());
        }
    }
}
// output
// Caught ExeException, msg: exe exception...
```

Exercise 5: (3) Create your own resumption-like behavior using a while loop that
repeats until an exception is no longer thrown. 

```java
class ExeException extends Exception {}

public class Exercise1 {
    public static void main(String[] args) {
        int index = 0;
        while (index < 3) {
            try {
                System.out.println(index / 0);
            } catch (ArithmeticException e) {
                System.out.println("Arithmetic exception when index is " + index);
                index ++;
            }
        }
        System.out.println("end program...");
    }
}

// output
// Arithmetic exception when index is 0
// Arithmetic exception when index is 1
// Arithmetic exception when index is 2
// end program...
```

## Exceptions and logging