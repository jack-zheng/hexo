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

使用 logging 工具类记录信息

```java
import java.util.logging.*;
import java.io.*;

class LoggingException extends Exception {
    private static Logger logger = Logger.getLogger("LoggingException");

    public LoggingException() {
        StringWriter trace = new StringWriter();
        printStackTrace(new PrintWriter(trace));
        logger.severe(trace.toString());
    }
}

public class LoggingExceptions {
    public static void main(String[] args) {
        try {
            throw new LoggingException();
        } catch (LoggingException e) {
            System.err.println("Caught " + e);
        }
        try {
            throw new LoggingException();
        } catch (LoggingException e) {
            System.err.println("Caught " + e);
        }
    }
}
// output
// Jan 25, 2021 10:57:56 AM reading.container.LoggingException <init>
// SEVERE: reading.container.LoggingException
// 	at reading.container.LoggingExceptions.main(LoggingExceptions.java:20)

// Caught reading.container.LoggingException
// Jan 25, 2021 10:57:56 AM reading.container.LoggingException <init>
// SEVERE: reading.container.LoggingException
// 	at reading.container.LoggingExceptions.main(LoggingExceptions.java:25)

// Caught reading.container.LoggingException
```

`Logger.getLogger()` 接收一个字符串作为参数创建 Logger 对象，如果没有其他设置，他回把对应的信息输出到 `System.err` 中去。`printStackTrace()` 接收一个 PrintWriter 作为参数，再通过调用 `logger.severe()` 将信息输出。

上面这种处理方式将所有的 logging 相关动作封装在了异常中，所以很简便，但是更常见的处理方式是将你要处理的 log 在 exception handler 中进行封装。

```java
class MyException2 extends Exception {
    private int x;
    public MyException2() {}
    public MyException2(String msg) { super(msg); }
    public MyException2(String msg, int x) {
        super(msg);
        this.x = x;
    }
    public int val() { return x; }
    public String getMessage() {
        return "Detail Message: "+ x + " "+ super.getMessage();
    }
}

public class ExtraFeatures {
    public static void f() throws MyException2 {
        System.out.println("Throwing MyException2 from f()");
        throw new MyException2();
    }
    public static void g() throws MyException2 {
        System.out.println("Throwing MyException2 from g()");
        throw new MyException2("Originated in g()");
    }
    public static void h() throws MyException2 {
        System.out.println("Throwing MyException2 from h()");
        throw new MyException2("Originated in h()", 47);
    }
    public static void main(String[] args) {
        try {
            f();
        } catch(MyException2 e) {
            e.printStackTrace(System.out);
        }
        try {
            g();
        } catch(MyException2 e) {
            e.printStackTrace(System.out);
        }
        try {
            h();
        } catch(MyException2 e) {
            e.printStackTrace(System.out);
            System.out.println("e.val() = " + e.val());
        }
    }
}
// output
// Throwing MyException2 from f()
// reading.container.MyException2: Detail Message: 0 null
// 	at reading.container.ExtraFeatures.f(ExtraFeatures.java:20)
// 	at reading.container.ExtraFeatures.main(ExtraFeatures.java:32)
// Throwing MyException2 from g()
// reading.container.MyException2: Detail Message: 0 Originated in g()
// 	at reading.container.ExtraFeatures.g(ExtraFeatures.java:24)
// 	at reading.container.ExtraFeatures.main(ExtraFeatures.java:37)
// Throwing MyException2 from h()
// reading.container.MyException2: Detail Message: 47 Originated in h()
// 	at reading.container.ExtraFeatures.h(ExtraFeatures.java:28)
// 	at reading.container.ExtraFeatures.main(ExtraFeatures.java:42)
// e.val() = 47
```

我们在自定义异常中添加了一个新的 field 并给出了对应的 getter 方法，重写了 `Throwable.getMessage( )` 方法。

exception 也是一个 Java 对象，你可以继续扩展这个类，但是值得注意的是，你的包装可能被其他人忽略，因为他们在使用的时候可能只想找一个贴切的异常并丢出去。

## The exception specification

在 Java 中，你需要告知调用者你的方法可能会抛出什么异常，而且这是强制的语法。这种语法使用 throw 作为关键字，后面接需要 catch 的异常 `void f() throws TooBig, TooSmall, DivZero { //...`

如果方法声明只是简单的 `void f() { //...` 这表示没有异常从这个方法中抛出。 `{except` 表示异常继承自 `RuntimeException`，这个异常可以在任何地方抛出。在异常声明中，你不能作弊。如果你方法中有抛出异常，但是你没有处理的话，编译器就会监测到并给你提示，要跑处理它，要跑继续抛出它。通过自顶向下的约束异常声明，Java 保证了在编译期的异常检测。

有一个特别的地方是，你可以在没有对应实现的情况下抛出异常。这种处理方式可以看作是一个预先打桩，为你将来的实现做预备，而且省去了你以后还要改应用代码的麻烦。

在编译期强制你检测的这种异常，叫做 Checked Exception。

## Catching any exception

在异常处理中，声明一个 catch 来捕捉所有的异常是可行的。你可以 catch Exception 来实现，他是几乎所有异常的基类

```java
catch(Exception e) {
 System.out.println("Caught an exception");
}
```

他会处理任何异常，所以确保将它放到你的 catch 列表的末位。由于他是一个基类，所以你一般不能得到什么很特殊的信息，但是你还是可以调用那些基于 Throwable 的方法，比如

String getMessage( )  
String getLocalizedMessage( )  

获取 message，或者是基于本地化的 message。

`String toString( )` 返回一个间断的关于 Throwable 类的描述，如果这个类有详细信息的话，也会包含在其中。

void printStackTrace( )  
void printStackTrace(PrintStream)  
void printStackTrace(java.io.PrintWriter)  

打印 Throwable 以及对应的调用栈信息。栈信息会告诉你异常发生的点。第一种方式会将异常输出到 standard error, 第二和三种方式会输出到对应的流。

Throwable 还有很多其他的方法可以调用，比如 `getClass()`， 它能返回一个异常对象，`getName()` 返回类信息，包含路径名，`getSimpleName()` 只含有类名。

```java
public class ExceptionMethods {
    public static void main(String[] args) {
        try {
            throw new Exception("My Exception");
        } catch (Exception e) {
            System.out.println("Caught Exception");
            System.out.println("getMessage():" + e.getMessage());
            System.out.println("getLocalizedMessage():" +
                    e.getLocalizedMessage());
            System.out.println("toString():" + e);
            System.out.println("System.out.printlnStackTrace():");
            e.printStackTrace(System.out);
        }
    }
}
// output
// Caught Exception
// getMessage():My Exception
// getLocalizedMessage():My Exception
// toString():java.lang.Exception: My Exception
// System.out.printlnStackTrace():
// java.lang.Exception: My Exception
// 	at reading.container.ExceptionMethods.main(ExceptionMethods.java:6)
```

### The stack trace

`printStackTrace( )` 中的信息也可以通过 `getStackTrace( )` 得到，他会返回一个信息栈。下面是一个示例，可以看到，root cause 是在第一行打印的，最外层的异常点在最后打印。

```java
public class WhoCalled {
    static void f() {
        // Generate an exception to fill in the stack trace
        try {
            throw new Exception();
        } catch (Exception e) {
            for (StackTraceElement ste : e.getStackTrace())
                System.out.println(ste.getMethodName());
        }
    }

    static void g() {
        f();
    }

    static void h() {
        g();
    }

    public static void main(String[] args) {
        f();
        System.out.println("--------------------------------");
        g();
        System.out.println("--------------------------------");
        h();
    }
}

// output
// f
// main
// --------------------------------
// f
// g
// main
// --------------------------------
// f
// g
// h
// main
```

### Rethrowing an exception

有时你在捕捉到异常之后会想要再一次 throw 它，比如之前提到的，通过 Exception 捕捉到异常的情况。这时你只需要在 handler 里面再 throw 即可

```java
catch(Exception e) {
    System.out.println("An exception was thrown");
    throw e;
}
```

Rethrowing 会将异常交由更高的 context 处理，这个过程中，异常对象的所有信息都会被保存下来，如果你想要创建一个新的异常对象，你可以使用 `fillInStackTrace( )` 方法，示例如下：

```java
public class Rethrowing {
    public static void f() throws Exception {
        System.out.println("originating the exception in f()");
        throw new Exception("thrown from f()");
    }

    public static void g() throws Exception {
        try {
            f();
        } catch (Exception e) {
            System.out.println("Inside g(),e.printStackTrace()");
            e.printStackTrace(System.out);
            throw e;
        }
    }

    public static void h() throws Exception {
        try {
            f();
        } catch (Exception e) {
            System.out.println("Inside h(),e.printStackTrace()");
            e.printStackTrace(System.out);
            throw (Exception) e.fillInStackTrace();
        }
    }

    public static void main(String[] args) {
        try {
            g();
        } catch (Exception e) {
            System.out.println("main: printStackTrace()");
            e.printStackTrace(System.out);
        }
        try {
            h();
        } catch (Exception e) {
            System.out.println("main: printStackTrace()");
            e.printStackTrace(System.out);
        }
    }
}

// originating the exception in f()
// Inside g(),e.printStackTrace()
// java.lang.Exception: thrown from f()
// 	at reading.container.Rethrowing.f(Rethrowing.java:6)
// 	at reading.container.Rethrowing.g(Rethrowing.java:11)
// 	at reading.container.Rethrowing.main(Rethrowing.java:31)
// main: printStackTrace()
// java.lang.Exception: thrown from f()
// 	at reading.container.Rethrowing.f(Rethrowing.java:6)
// 	at reading.container.Rethrowing.g(Rethrowing.java:11)
// 	at reading.container.Rethrowing.main(Rethrowing.java:31)
// /--------------------- Dash --------------------/
// originating the exception in f()
// Inside h(),e.printStackTrace()
// java.lang.Exception: thrown from f()
// 	at reading.container.Rethrowing.f(Rethrowing.java:6)
// 	at reading.container.Rethrowing.h(Rethrowing.java:21)
// 	at reading.container.Rethrowing.main(Rethrowing.java:38)
// main: printStackTrace()
// java.lang.Exception: thrown from f()
// 	at reading.container.Rethrowing.h(Rethrowing.java:25)
// 	at reading.container.Rethrowing.main(Rethrowing.java:38)
```

`f()` 中通过 `fillInStackTrace( )` 改变了异常原点，相比于之前的调用 `g()` 的方法信息没有了。当然你也可以用 throw 新的 Exception 来实现和 `fillInStackTrace()` 同样的功能

```java
class OneException extends Exception {
    public OneException(String s) {
        super(s);
    }
}

class TwoException extends Exception {
    public TwoException(String s) {
        super(s);
    }
}

public class RethrowNew {
    public static void f() throws OneException {
        System.out.println("originating the exception in f()");
        throw new OneException("thrown from f()");
    }

    public static void main(String[] args) {
        try {
            try {
                f();
            } catch (OneException e) {
                System.out.println(
                        "Caught in inner try, e.printStackTrace()");
                e.printStackTrace(System.out);
                throw new TwoException("from inner try");
            }
        } catch (TwoException e) {
            System.out.println(
                    "Caught in outer try, e.printStackTrace()");
            e.printStackTrace(System.out);
        }
    }
}

// originating the exception in f()
// Caught in inner try, e.printStackTrace()
// reading.container.OneException: thrown from f()
// 	at reading.container.RethrowNew.f(RethrowNew.java:18)
// 	at reading.container.RethrowNew.main(RethrowNew.java:24)
// Caught in outer try, e.printStackTrace()
// reading.container.TwoException: from inner try
// 	at reading.container.RethrowNew.main(RethrowNew.java:29)
```

最后的 exception handler 只知道异常来源于 inner try block 而不知道任何关于 f() 的信息。你完全不用关心异常的清理问题，他们都是基于堆创建的对象，垃圾回收机制会负责清理他们。

### Exception chaining

通常来说，当你抛出自己的异常时，你都会希望这个异常带有原始异常的信息。在 Java 1.4 以前，码农们需要自己处理这个问题，但是之后的版本中，你可以通过在构造函数中传入异常类来实现这个功能。

Throwable 的子类中只有三个提供这个功能，分别是 Error(用于记录 JVM 异常)，Exception 和 RuntimeException。如果其他类型的异常，你也想串联起来的话，你可以调用 `initCause()` 方法，示例如下：

```java
class DynamicFieldsException extends Exception {}

public class DynamicFields {
    private Object[][] fields;

    public DynamicFields(int initialSize) {
        fields = new Object[initialSize][2];
        for (int i = 0; i < initialSize; i++)
            fields[i] = new Object[]{null, null};
    }

    public String toString() {
        StringBuilder result = new StringBuilder();
        for (Object[] obj : fields) {
            result.append(obj[0]);
            result.append(": ");
            result.append(obj[1]);
            result.append("\n");
        }
        return result.toString();
    }

    private int hasField(String id) {
        for (int i = 0; i < fields.length; i++)
            if (id.equals(fields[i][0]))
                return i;
        return -1;
    }

    private int
    getFieldNumber(String id) throws NoSuchFieldException {
        int fieldNum = hasField(id);
        if (fieldNum == -1)
            throw new NoSuchFieldException();
        return fieldNum;
    }

    private int makeField(String id) {
        for (int i = 0; i < fields.length; i++)
            if (fields[i][0] == null) {
                fields[i][0] = id;
                return i;
            }
        // No empty fields. Add one:
        Object[][] tmp = new Object[fields.length + 1][2];
        for (int i = 0; i < fields.length; i++)
            tmp[i] = fields[i];
        for (int i = fields.length; i < tmp.length; i++)
            tmp[i] = new Object[]{null, null};
        fields = tmp;
        // Recursive call with expanded fields:
        return makeField(id);
    }

    public Object getField(String id) throws NoSuchFieldException {
        return fields[getFieldNumber(id)][1];
    }

    public Object setField(String id, Object value)
            throws DynamicFieldsException {
        if (value == null) {
            // Most exceptions don’t have a "cause" constructor.
            // In these cases you must use initCause(),
            // available in all Throwable subclasses.
            DynamicFieldsException dfe = new DynamicFieldsException();
            dfe.initCause(new NullPointerException());
            throw dfe;
        }
        int fieldNumber = hasField(id);
        if (fieldNumber == -1)
            fieldNumber = makeField(id);
        Object result = null;
        try {
            result = getField(id); // Get old value
        } catch (NoSuchFieldException e) {
            // Use constructor that takes "cause":
            throw new RuntimeException(e);
        }
        fields[fieldNumber][1] = value;
        return result;
    }

    public static void main(String[] args) {
        DynamicFields df = new DynamicFields(3);
        System.out.println(df);
        try {
            df.setField("d", "A value for d");
            df.setField("number", 47);
            df.setField("number2", 48);
            System.out.println(df);
            df.setField("d", "A new value for d");
            df.setField("number3", 11);
            System.out.println("df: " + df);
            System.out.println("df.getField(\"d\") : " + df.getField("d"));
            Object field = df.setField("d", null); // Exception
        } catch (NoSuchFieldException e) {
            e.printStackTrace(System.out);
        } catch (DynamicFieldsException e) {
            e.printStackTrace(System.out);
        }
    }
}

// output
// null: null
// null: null
// null: null

// d: A value for d
// number: 47
// number2: 48

// df: d: A new value for d
// number: 47
// number2: 48
// number3: 11

// df.getField("d") : A new value for d
// reading.container.DynamicFieldsException
// 	at reading.container.DynamicFields.setField(DynamicFields.java:68)
// 	at reading.container.DynamicFields.main(DynamicFields.java:98)
// Caused by: java.lang.NullPointerException
// 	at reading.container.DynamicFields.setField(DynamicFields.java:69)
// 	... 1 more
```

示例说明：

* 自定义一个异常 DynamicFieldsException
* DynamicFields 为测试类，包含一个需要处理的 field 叫做 fields，他是一个二维数组
* fields 初始化时可以给定长度，宽度为固定值 2, 也就是 n*2 的矩阵
* fields 的子单元值为对象，不能填充原始类型的值
* 自定义 toString 方法可以答应矩阵值
* setField 可以设置一行的值，如果超出容量，自动 copy + append, 设置的值不能为 null 否则报错
* getField 返回对应行的值，如果没有抛异常

在 `setField()` 方法中，我们我们为 DynamicFieldsException 通过调用 initCause 设置了 NPE 为 root

## Standard Java exceptions

Java 的 Throwable 类代表了所有可 throw 类，通常有两个常用子类 Error 和 Exception。Error 表示 compile-time 和系统错误，这些是你不需要关心的。另一类是 Exception，这些是码农需要关心的。

想要对 Exception 有一个概览，最好就去看一下 JDK 文档，这可以给你找找感觉，但是当你看了之后，你会发现，这些异常，出了名字不同外，其他基本都是一样的。如果你是用第三方包，那么很大概率会遇到他们自定义的异常。所以最重要的事是来哦姐他的定义，还有就是知道当你遇到它时你需要做什么。

异常的名字就代表了它处理的场景，异常的命名要求贴切明了。异常并不是全都定义在 java.lang 下，其他一些包，比如 util, net 和 io 也都有自己的异常类。你可以通过查看他们的包路径知道这些信息。比如所有的 I/O 异常都是继承于 java.io.IOException。

### Special case: RuntimeException

下面是第一个示例

```java
if (t == null) {
    throw new NullPointException();
}
```

如果代码中每个可能有 null 引用的地方都需要做 NPE 检测，那想象就很刺激。所幸，这个检测 Java 会替你完成，所以上述的代码中的 NPE 检查是多余的。

JDK 中有一族异常处理类似的问题，Java 代码中会自动抛出，自动处理这些异常签名。他们有一个基类叫做 `RuntimeException`, 由它派生出来的异常都不需要在声明中他别指出来。他们也被叫做 unchecked exceptions(非受检异常)。虽然你不需要检测 RuntimeException, 都是你在写代码的过程中可能会想要抛出这个异常。

下面是一个没有捕获 RuntimeException 的例子：

```java
public class NeverCaught {
    static void f() {
        throw new RuntimeException("From f()");
    }

    static void g() {
        f();
    }

    public static void main(String[] args) {
        g();
    }
}

// output
// Exception in thread "main" java.lang.RuntimeException: From f()
// 	at reading.container.NeverCaught.f(NeverCaught.java:5)
// 	at reading.container.NeverCaught.g(NeverCaught.java:9)
// 	at reading.container.NeverCaught.main(NeverCaught.java:13)
```

你可以看到，即使你在 f() 中 throw 了这个异常，但是你在调用它的位置也不需要用异常签名标识它。

时刻牢记，只有 运行时异常 可以这么处理， checked exception 不行，因为 Java 语法中，将运行时异常当作系统错误处理，系统错误的定义：

1. 那些你不能预料的异常，比如 null reference
2. 那种作为作者，你在程序中应该检查的错误，比如 ArraylndexOutOfBoundsException

## Performing cleanup with finally

### What’s finally for? 

### Using finally during return

### Pitfall: the lost exception

## Exception restrictions

## Constructors

## Exception matching

## Alternative approaches

### History

### Perspectives

### Passing exceptions to the console

### Converting checked to unchecked exceptions

## Exception guidelines

## Summary