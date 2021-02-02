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

[x] runtime 异常和其他的异常有什么区别
[x] 异常处理的最佳实践是什么
[x] catch 里加一个 return，final 还会执行吗

Answers:

1. RuntimeException 可以不用写 try-catch 处理，而 checked exception 你必须添加 try-catch block
2. 本章的后半截有介绍，首先是根据当前节点是否有能力处理对应的异常，倒数第二章还有介绍一些规则。
3. final 还是会执行

## 前述

最理想的异常捕捉点是编译期，即代码运行之前。但是并不是所有的 errors 都能在编译器就被捕捉到。剩下的这些问题，我们需要在运行时让异常产生者将错误信息提交给接受者做适当的处理。

提升异常处理是增加代码健壮性的重要方式。异常恢复是每个码农都需要考虑的问题，对于 Java 这种旨在为他人提供接口的语言来说，这点尤为重要。通过提供 error-reporting 错误处理模式，Java 允许组件代码将异常传给客户端代码处理。

Java 中的异常处理机制旨在以尽量少的代码完成经可能多的功能的同时让你的程序覆盖尽可能多的异常情况。异常机制不难学，并且学会了他能马上对你的项目产生益处，他是唯一官方指定的处理异常的方式并且编译器强制检查的。

本章将介绍一些必须要用到异常的代码以及遇到异常时的处理方案。

## Concepts

C 和其他一些早期语言经常会有多中处理异常的方式，这些方式基本都是便宜形式，并且不是语言的一部分。典型的案例就是放回一个特殊的值或者设置一个特殊的 flag，接收方根据这个返回值来进行相应的处理。但是渐渐的码农们突然意识到，自己定义的异常情况可能是不充分的，并且为了覆盖这些没有覆盖到的情况，代码会变得越来越难以维护。

解决方案是将异常处理正规化，这种思路的出现是一个很长过的过程，可以追溯到 1960 年。

exception 表示：我处理这个异常是为了。。。当异常发生时，你可能并不清楚你需要做什么，但是你知道，你不能在正常的执行下去了，你应该做点什么。你就可以将这个异常提交给上层处理，那里可能具备足够的知识处理它。

异常处理机制的另一个明显的好处是，它可以简化你的异常处理代码。如果没有他你必须在多个地方编写检测代码。有了它之后，就不必这么做了，exception 会保证，有人在合适的地方做检查。你只需要在一个地方处理即可。这种机制简化了你的代码，代码被分为两个分之，正常的分支和异常分支，同时你阅读，书写，调试代码也会变的更方便。

## Basic exceptions

发生异常的条件下，程序会停止执行当前方法。当代码发生异常时，程序由于缺少某些信息，已经不能继续执行程序了，他能做的就是跳出当前的上下文并且将执行权交给上层。

`1\0` 就是这么一种情况，当代码中出现了除 0 的情况，就需要检查一下了。可能你知道为什么要除 0，可能是你业务逻辑的需要，你知道在这种情况下需要做什么。但是如果这是一个意外情况，你必须停止当前方法并且抛出一个异常。

当你跑出一个异常时，有几件事情会发生。首先一个异常对象会在堆上通过 new 的方式被创建出来。然后当前的执行路径被阻断，异常对象被当前上下文弹出。exception-handling 处理机制开始接收这个异常，并试图妥善的处理它。

Exception handler 就是处理异常的地方，他会判断是否继续执行或者寻求其他解决路径。

想象一个简单的场景，比如你有一个对象引用叫做 `t`，他可能没有被初始化过，所以你想在使用前检查一下。你可以将检查的错误通过一个对象包裹起来并 thorwing 出去，这种做法就叫做抛出异常，代码如下：

```java
if(t == null)
 throw new NullPointerException();
```

这种做法可以让你为以后做打算，他会在之后的什么地方被处理，你很快能看到。

Exceptions 让你能够以 事务(transaction) 为单位处理问题， 你也可以将它想象成一个 undo 系统，你可以设置多个恢复点，当你的程序抛出异常时，他可以将程序恢复到某个稳定的节点。

Exceptions 最重要的一个点就是，当异常发生时，他阻止程序继续执行下去。C 语言在这方面就很糟糕，C 语言中是没有打断机制的，这发生异常时，你都不能预期它会执行到什么状态。

## Exception arguments

和其他 Java 中的对象一样，你可以通过 new 关键字创建一个 exception 对象，它有两种构造函数，一种是无参的，另一种是带字符串的，比如：

```java
throw new NullPointerException("t = null");
```

当然这个信息字符串也可以在之后通过调用 `set` 方法设置，之后有该种示例。

`throw` 这个关键词可以产生几种很有趣的结果。当你使用 new 创建一个 exception 的时候，你指定了 throw 的对象。虽然这个对象和你方法的返回值类型不一样，但是它还是会被这个方法返回。由此，你可以将异常处理看作一种特殊的 return 机制。返回的同时，方法和 scope 将会弹出(出栈)。

和普通方法的共同点到此为止了，接下来的处理方式将迥异于普通方法。异常将会在 exception handler 中被处理。

虽然你可以在处理异常时抛出任何 Throwable 的子类，但是一般来说，我们本会更具 error 的具体类型来指定它。error 的信息可以从他的名字和内容体现出来，但是通常来说异常只包含类名而没有其他什么内容。

## Catching an exception

在理解异常捕捉之前，你先得理解**守护区域**的概念，就是被 try 包裹的部分。它代表了一段可能抛出异常的代码段，这段代码之后会紧接着一段异常处理代码。

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

如果异常被抛出，exception-handling 机制会搜寻第一个匹配的 catch 分支并进入，当 catch 分支走完后，异常处理被视为结束。不像 switch，catch 分支**不需要 break 关键字**，执行完直接返回。

## Termination vs. resumption (中断还是继续?)

异常处理有两种模型，Java 采用的是中断，他认为当异常发生后，你不能在回到异常发生的节点。

另一种是 resumption(继续)，他表示异常发生后，我们可以做一些补救措施，并且尝试重新执行失败的方法。采用这个方式意味着你在异常产生后依旧希望继续执行程序。

如果你想要 resumption 的处理方法，你不能在 error 发生的地方抛异常，或者你可以把你抛异常的代码放到一个循环中，多次运行，知道结果符合你预期。

历史上，码农们有尝试过使用 resumption 机制的操作系统，但最终回归到了 termination 机制。虽然 resumption 机制乍一听上去很美，但是并不是这么实用。可能是因为这种机制下你写的代码不能很通用，难以维护，特别是在写一些大型项目的时候，最后没有保留下来。

## Creating your own exceptions

Java 允许你自己定制异常，你需要做的只是继承一个已有的异常类即可。当然继承的时候如果可能的话，选一个最贴近你异常类的，那是极好的。创建时只需要用它的默认构造函数即可，代码很简单：

自定义个一个异常 SimpleException 继承自 Exception，其他什么都没有。这是很常见的定义异常的方式，它调用默认的无参构造函数。定义异常时，取一个见名知意的名字显得尤为重要。

```java
class SimpleException extends Exception {}

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

下面是调用带参构造的例子，只需要少量的新加 code 就能实现带参构造，声明的时候用上 `super` 关键字即可。

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

使用 logging 工具类记录信息，`Logger.getLogger()` 接收一个字符串作为参数创建 Logger 对象，如果没有其他设置，他回把对应的信息输出到 `System.err` 中去。`printStackTrace()` 接收一个 PrintWriter 作为参数，再通过调用 `logger.severe()` 将信息输出。

上面这种处理方式将所有的 logging 相关动作封装在了异常中，所以很简便，但是更常见的处理方式是将你要处理的 log 在 exception handler 中进行封装。

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

下面是在 catch 中 log 异常信息的例子。自定义一个异常 MyException2，重写三种构造函数，分别是默认，带一个字符串，带字符串和数字三种形式。三种方式会分别在异常对象中多设置一个属性。

ExtraFeatures 中声明三个方法，调用三种异初始化函数，并抛出。主函数中，捕获异常并处理。

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
        return "Detail Message: " + x + " " + super.getMessage();
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

exception 也是一个 Java 对象，你可以继续扩展这个类，但是值得注意的是，你的包装可能被其他人忽略，因为他们在使用的时候可能只想找一个贴切的异常并丢出去。

## The exception specification

在 Java 中，你需要告知调用者你的方法可能会抛出什么异常，而且这是强制的。这种语法使用 throw 作为关键字，后面接需要 catch 的异常 `void f() throws TooBig, TooSmall, DivZero { //...`

如果方法声明只是简单的 `void f() { //...` 这表示没有异常从这个方法中抛出。 `{except` 表示异常继承自 `RuntimeException`，这个异常可以在任何地方抛出。在异常声明中，你不能作弊。如果你方法中有抛出异常，但是你没有处理的话，编译器就会监测到并给你提示你要么抛出它要么在方法签名上给出提示。通过自顶向下的约束异常声明，Java 保证了在编译期的异常检测。

有一个特别的地方是，你可以在没有对应实现的情况下抛出异常。这种处理方式可以看作是一个预先打桩，为你将来的实现做准备，而且省去了以后改应用代码的麻烦。

在编译期强制做检测的这种异常叫做 Checked Exception(受检异常)。

## Catching any exception

在异常处理中，声明一个 catch 来捕获 Exception 以达到捕获几乎所有异常的基类的目的，这是可行的，而且很常见。

```java
catch(Exception e) {
 System.out.println("Caught an exception");
}
```

他会处理任何异常，所以确保将它放到你的 catch 列表的末位。由于他是一个基类，所以你一般不能得到什么很特殊的信息，但是你还是可以调用那些基于 Throwable 的方法，比如

```java
String getMessage( )  
String getLocalizedMessage( )  
```

获取 message，或者是基于本地化的 message。

`String toString( )` 返回一个简短的关于 Throwable 类的描述，如果这个类有详细信息的话，也会包含在其中。

```java
void printStackTrace( )  
void printStackTrace(PrintStream)  
void printStackTrace(java.io.PrintWriter)  
```

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

在你的代码中总有一些动作是你无论如何都要做的，不管是否有异常发生， 为了应对这写问题，我们在所有的 catch 结束后引入了 `finally` 这个关键字。完成的表现形式如下：

```java
try {
 // The guarded region: Dangerous activities
 // that might throw A, B, or C
} catch(A a1) {
 // Handler for situation A
} catch(B b1) {
 // Handler for situation B
} catch(C c1) {
 // Handler for situation C
} finally {
 // Activities that happen every time
} 
```

为了表明 final 是一个必定执行的分支，我们创建了一下示例：

```java
class ThreeException extends Exception {}

public class FinallyWorks {
    static int count = 0;

    public static void main(String[] args) {
        while (true) {
            try {
                // Post-increment is zero first time:
                if (count++ == 0)
                    throw new ThreeException();
                System.out.println("No exception");
            } catch (ThreeException e) {
                System.out.println("ThreeException");
            } finally {
                System.out.println("In finally clause");
                if (count == 2) break; // out of "while"
            }
        }
    }
}

// output
// ThreeException
// In finally clause
// No exception
// In finally clause
```

从输出的 log 我们可以看到，无论是否有异常抛出，finally 里面的内容都会被执行。

这个代码段同时也提示我们，Java 是不允许我们回到异常点的，如果你将你的 try block 放到一个循环中，你可以设定条件来重复执行他。

### What’s finally for? 

在一个没有垃圾回收和和自动解构的语言中，finally 是很重要的，但是 Java 语言体系中已经默认给你提供了这些功能，那么 fianlly 又是用来做什么的呢？

finally 可以用于重制除内存以外的对象，比如关闭文件，或者网络链接之类的东西。

示例说明：

下面这个例子，我们想要达到的效果是无论如果要在程序结束时将开关关闭。

我们声明了两个异常 OnOffException1， OnOffException2，但是如果要将关闭的动作放到 catch 中，会出现很多重复的代码。

```java
class OnOffException1 extends Exception {}

class OnOffException2 extends Exception {}

public class Switch {
    private boolean state = false;

    public boolean read() {
        return state;
    }

    public void on() {
        state = true;
        System.out.println(this);
    }

    public void off() {
        state = false;
        System.out.println(this);
    }

    public String toString() {
        return state ? "on" : "off";
    }
}

public class OnOffSwitch {
    private static Switch sw = new Switch();

    public static void f()
            throws OnOffException1, OnOffException2 {
    }

    public static void main(String[] args) {
        try {
            sw.on();
            // Code that can throw exceptions...
            f();
            sw.off();
        } catch (OnOffException1 e) {
            System.out.println("OnOffException1");
            sw.off();
        } catch (OnOffException2 e) {
            System.out.println("OnOffException2");
            sw.off();
        }
    }
}

// output
// on
// off
```

我们可以加一个 finally 来统一处理，去除重复代码。

```java
public class WithFinally {
    static Switch sw = new Switch();

    public static void main(String[] args) {
        try {
            sw.on();
            // Code that can throw exceptions...
            OnOffSwitch.f();
        } catch (OnOffException1 e) {
            System.out.println("OnOffException1");
        } catch (OnOffException2 e) {
            System.out.println("OnOffException2");
        } finally {
            sw.off();
        }
    }
}

// output
// on
// off
```

现在无论异常是否抛出，switch 都会被 turn off。

下面是一个更加深入的例子：

```java
class FourException extends Exception {}

public class AlwaysFinally {
    public static void main(String[] args) {
        System.out.println("Entering first try block");
        try {
            System.out.println("Entering second try block");
            try {
                throw new FourException();
            } finally {
                System.out.println("finally in 2nd try block");
            }
        } catch (FourException e) {
            System.out.println(
                    "Caught FourException in 1st try block");
        } finally {
            System.out.println("finally in 1st try block");
        }
    }
}

// output
// Entering first try block
// Entering second try block
// finally in 2nd try block
// Caught FourException in 1st try block
// finally in 1st try block
```

即使有 break 或者 continue 关键字，finally 还是会被执行，它的出现结束了 goto 关键字的使用。

### Using finally during return

由于 finally 是保证会被执行的，这就是的一段程序中有两个返回点变为可能，而且一些重要的 cleanup 必定会被执行。

```java
public class MultipleReturns {
    public static void f(int i) {
        System.out.println("Initialization that requires cleanup");
        try {
            System.out.println("Point 1");
            if (i == 1) return;
            System.out.println("Point 2");
            if (i == 2) return;
            System.out.println("Point 3");
            if (i == 3) return;
            System.out.println("End");
            return;
        } finally {
            System.out.println("Performing cleanup");
        }
    }

    public static void main(String[] args) {
        for (int i = 1; i <= 4; i++)
            f(i);
    }
}

// Initialization that requires cleanup
// Point 1
// Performing cleanup
// Initialization that requires cleanup
// Point 1
// Point 2
// Performing cleanup
// Initialization that requires cleanup
// Point 1
// Point 2
// Point 3
// Performing cleanup
// Initialization that requires cleanup
// Point 1
// Point 2
// Point 3
// End
// Performing cleanup
```

我们可以看到，不管 return 在哪里，finally 都会被执行到。

### Pitfall: the lost exception

Java 异常处理终有一个缺陷可能导致我们漏掉异常。这种情况和 finally 有关，示例如下：

```java
class VeryImportantException extends Exception {
    public String toString() {
        return "A very important exception!";
    }
}

class HoHumException extends Exception {
    public String toString() {
        return "A trivial exception";
    }
}


public class LostMessage {
    void f() throws VeryImportantException {
        throw new VeryImportantException();
    }

    void dispose() throws HoHumException {
        throw new HoHumException();
    }

    public static void main(String[] args) {
        try {
            LostMessage lm = new LostMessage();
            try {
                lm.f();
            } finally {
                lm.dispose();
            }
        } catch (Exception e) {
            System.out.println(e);
        }
    }
}
// A trivial exception
```

我们可以看到在上面的例子中 VerylmportantException 被吞了，只有 HoHumException 被捕获了。这种缺陷很严重，异常被完全抹去了，而且很难找到 root cause。

下面是一个更直接的例子, 如果我们在 final 中加了一个 return，那么所有的 try 中的异常都会被 skip 掉。

```java
public class ExceptionSilencer {
    public static void main(String[] args) {
        try {
            throw new RuntimeException();
        } finally {
            // Using ‘return’ inside the finally block
            // will silence any thrown exception.
            return;
        }
    }
}
```

## Exception restrictions

当你 重写 一个方法的时候，你只能抛出基类方法中规定的异常，这个限制很有用，通过这样的限制，就能使重写的方法可以在原方法出现的地方生效。下面的方法展示了异常在继承体系中的限制情况

示例说明：

Inning 为基类， Storm 为接口， StormyInning 为测试类，继承 Inning 并且实现 Storm 接口。接口和基类中抛出的异常不同，并且接口和异常中有一个方法是同名的。总结规则如下

1. 构造函数和一般方法相比，比较特别，它要求必须抛出和基类构造函数一致的异常的同时，可以新增异常
2. 方法只在一个超类(接口或者基类)中出现，那么子类中的异常 <= 超类，甚至可以不抛出异常
3. 如果方法在接口和基类中都出现，则一基类为准
4. 从 main 方法中可以看出，程序会根据你的类信息来处理对应的异常。StormyInning 时处理一类，转化为 Inning 时需要处理的异常就改变了。

子类不能抛出基类没有定义的异常的理由：比如在框架层级的代码中，你写了一个 flow, 将所有的一族类统一到一个流程中，如果允许子类可以抛出基类没有的异常，那么就有可能 flow 中没有子类的异常处理逻辑。

```java

class BaseballException extends Exception {}

class Foul extends BaseballException {}

class Strike extends BaseballException {}

abstract class Inning {
    public Inning() throws BaseballException {}

    public void event() throws BaseballException {
        // Doesn’t actually have to throw anything
    }

    public abstract void atBat() throws Strike, Foul;

    public void walk() {} // Throws no checked exceptions
}

class StormException extends Exception {}

class RainedOut extends StormException {}

class PopFoul extends Foul {}

interface Storm {
    public void event() throws RainedOut;

    public void rainHard() throws RainedOut;
}

public class StormyInning extends Inning implements Storm {
    // OK to add new exceptions for constructors, but you
    // must deal with the base constructor exceptions:
    public StormyInning()
            throws RainedOut, BaseballException {
    }

    public StormyInning(String s)
            throws Foul, BaseballException {
    }

    // Regular methods must conform to base class:
    // !public void walk() throws PopFoul {} //Compile error
    // Interface CANNOT add exceptions to existing
    // methods from the base class:
    // !public void event() throws RainedOut {}
    // If the method doesn’t already exist in the
    // base class, the exception is OK:
    public void rainHard() throws RainedOut {}

    // You can choose to not throw any exceptions,
    // even if the base version does:
    public void event() {}

    // Overridden methods can throw inherited exceptions:
    public void atBat() throws PopFoul {}

    public static void main(String[] args) {
        try {
            StormyInning si = new StormyInning();
            si.atBat();
        } catch (PopFoul e) {
            System.out.println("Pop foul");
        } catch (RainedOut e) {
            System.out.println("Rained out");
        } catch (BaseballException e) {
            System.out.println("Generic baseball exception");
        }
        // Strike not thrown in derived version.
        try {
            // What happens if you upcast?
            Inning i = new StormyInning();
            i.atBat();
            // You must catch the exceptions from the
            // base-class version of the method:
        } catch (Strike e) {
            System.out.println("Strike");
        } catch (Foul e) {
            System.out.println("Foul");
        } catch (RainedOut e) {
            System.out.println("Rained out");
        } catch (BaseballException e) {
            System.out.println("Generic baseball exception");
        }
    }
}
```

虽然异常签名会在继承时对你做一些语法上的要求，但是它并不是方法的一部分。方法签名只和 方法名，方法参数有关。所以你在重写方法的时候是不能基于异常类型的。

## Constructors

时常问自己一句，“如果有异常发生，是不是所有的东西都会被清理干净” 是很重要的。大部分情况下你是安全的，但是在构造函数中，这就是个问题了。构造函数中，对象一开始是安全的，但是随着程序的进行，比如打开了一个文件。但是只有在完成读写后他才会关闭这个流。如果在构造函数中抛出异常，那么这个清理工作可能不能顺利完成。这就意味着，在处理构造函数的时候，你必须格外小心。

你可能会想，我们可以用 finally 来处理这种情况，但是情况可能并没有这么简单，因为 finally 是每次都会执行的。如果构造函数中途挂了，一些对象可能没有被正确的创建出来，那么对应的 finally 执行也可能出问题。

下面的例子中，我们用一个 I/O 的例子做示范：

```java
public class InputFile {
    private BufferedReader in;

    public InputFile(String fname) throws Exception {
        try {
            in = new BufferedReader(new FileReader(fname));
            // Other code that might throw exceptions
        } catch (FileNotFoundException e) {
            System.out.println("Could not open " + fname);
            // Wasn’t open, so don’t close it
            throw e;
        } catch (Exception e) {
            // All other exceptions must close it
            try {
                in.close();
            } catch (IOException e2) {
                System.out.println("in.close() unsuccessful");
            }
            throw e; // Rethrow
        } finally {
            // Don’t close it here!!!
        }
    }

    public String getLine() {
        String s;
        try {
            s = in.readLine();
        } catch (IOException e) {
            throw new RuntimeException("readLine() failed");
        }
        return s;
    }

    public void dispose() {
        try {
            in.close();
            System.out.println("dispose() successful");
        } catch (IOException e2) {
            throw new RuntimeException("in.close() failed");
        }
    }
}
```

构造函数里，InputFile 用 String 表示文件名，并在一个 try block 创建一个 FileReader。InputFile 并没有什么特别的地方，它最大的作用是将 FileReader 和 BufferedReader 结合在了一起。

如果 FileReader 的构造失败了，就会抛出 FileNotFoundException。这种情况下，IO 流并没有被正常的开启，我们在异常处理时不需要关闭这个流。除这个异常外的其他异常，则要求我们关闭文件流。因为如果是其他的异常，则当时文件流已经被打开了，我们就需要在处理异常后关闭文件流。close() 方法也会抛出异常，对应的，我们在 catch 中也添加一个 try-catch 处理。处理完这些异常后我们再将这个异常抛出去。

上面的示例中，由于 finally 是每次比执行的，所以不是一个处理 close() 的好地方。

getLine() 会返回下一行内容。底层是调用了 readLine() 方法，它会抛出一个异常，但是这个异常被捕获并转化为 RuntimeException了，所以方法签名中不需要处理该异常。

当 InputFile 对象使用完毕后，调用 dispose() 方法释放资源。你可能想要将这个动作放到 finalize() 方法中，但是 Java 语言体系是不支持这种操作的，算是 Java 的缺陷之一。

像这种例子，最安全的做法应该是在原来的 try block 中再嵌套一个 try block。

```java
public class Cleanup {
    public static void main(String[] args) {
        try {
            InputFile in = new InputFile("Cleanup.java");
            try {
                String s;
                int i = 1;
                while ((s = in.getLine()) != null)
                    ; // Perform line-by-line processing here...
            } catch (Exception e) {
                System.out.println("Caught Exception in main");
                e.printStackTrace(System.out);
            } finally {
                in.dispose();
            }
        } catch (Exception e) {
            System.out.println("InputFile construction failed");
        }
    }
}
// dispose() successful
```

上例中，我们有通过两个 try block 进行嵌套，一个处理文件流的构造，一个处理文件流读写。构造方法失败无需调用 dispose()，读写失败则需要调用读写。

这种通用的 cleanup 即使是那些不会抛异常的构造函数，也可以适用。基本规则是：当你创建了需要 cleanup 的对象，你就可以开始适用 try-finally 了。

```java
class NeedsCleanup { // Construction can’t fail
    private static long counter = 1;
    private final long id = counter++;

    public void dispose() {
        System.out.println("NeedsCleanup " + id + " disposed");
    }
}

class ConstructionException extends Exception {}

class NeedsCleanup2 extends NeedsCleanup {
    // Construction can fail:
    public NeedsCleanup2() throws ConstructionException {
    }
}

public class CleanupIdiom {
    public static void main(String[] args) {
        // Section 1:
        NeedsCleanup nc1 = new NeedsCleanup();
        try {
            // ...
        } finally {
            nc1.dispose();
        }

        // Section 2:
        // If construction cannot fail you can group objects:
        NeedsCleanup nc2 = new NeedsCleanup();
        NeedsCleanup nc3 = new NeedsCleanup();
        try {
            // ...
        } finally {
            nc3.dispose(); // Reverse order of construction
            nc2.dispose();
        }

        // Section 3:
        // If construction can fail you must guard each one:
        try {
            NeedsCleanup2 nc4 = new NeedsCleanup2();
            try {
                NeedsCleanup2 nc5 = new NeedsCleanup2();
                try {
                    // ...
                } finally {
                    nc5.dispose();
                }
            } catch (ConstructionException e) { // nc5 constructor
                System.out.println(e);
            } finally {
                nc4.dispose();
            }
        } catch (ConstructionException e) { // nc4 constructor
            System.out.println(e);
        }
    }
}
// NeedsCleanup 1 disposed
// NeedsCleanup 3 disposed
// NeedsCleanup 2 disposed
// NeedsCleanup 5 disposed
// NeedsCleanup 4 disposed
```

在 main() 函数中， section 1 的内容很直截了当，如果构造失败，就不需要 try-finally 了。

section 2 中，如果构造成功，我们可以在一个 try-finally 中同时关闭两个对象。

section 3 中，由于构造函数本身会跑出异常，所以每执行一个对象的初始化，你就需要有一个 try-finally 处理它，对应的代码会变得混乱。在这种情况下，强烈建议你将初始化的代码段也纳入 try 中，虽然有点容于，但是更好维护。

如果 dispose() 也会抛出异常，你需要额外的 try 来处理它，总之你必须处理所有可能出现的情况。

## Exception matching

当抛出异常时，异常处理系统会查找最近的处理器。如果找到一个匹配的，那么就视作异常已经被处理，不会找下一个了。

搜索异常是并不会精确匹配，子类异常可以匹配到基类异常处理器。

在下面的例子中，第一个 try block 中 `Sneeze` 异常被第一个 catch block 捕获，这个合理。但是当我们将第一个 catch 移除时，异常也可以被 Annoyance 这个基类异常捕获。

如果你将第一个示例中的 Annoyance 提前，会有**编译错误**指出 Sneeze 已经被捕获，不需要再处理了。

```java
class Annoyance extends Exception {}

class Sneeze extends Annoyance {}

public class Human {
    public static void main(String[] args) {
        // Catch the exact type:
        try {
            throw new Sneeze();
        } catch (Sneeze s) {
            System.out.println("Caught Sneeze");
        } catch (Annoyance a) {
            System.out.println("Caught Annoyance");
        }
        // Catch the base type:
        try {
            throw new Sneeze();
        } catch (Annoyance a) {
            System.out.println("Caught Annoyance");
        }
    }
}
// output
// Caught Sneeze
// Caught Annoyance
```

## Alternative approaches

异常处理系统提供了一个在程序异常时的分支。异常状态表示当前程序不能被处理，异常系统开发的初衷是为了给程序员处理异常情况提供便利性。

异常处理的准则之一：不要捕获那些你不知道怎么处理的异常。其实，异常处理的主要目标之一是将异常处理代码从当前节点移除。这样你就可以将你的主要逻辑集中到一个地方，而在不远处的 catch 中集中处理异常代码。这样代码更容易理解和维护。

一个 handler 可以处理多种异常，减少了处理代码的量

Checked exception 会强制你写 catch block 这个有违于 'harmful if swallowed' 原则。

```java
try {
// ... to do something useful
} catch(ObligatoryException e) {} // Gulp! 
```

码农们仅仅做了捕获，并不处理。但是编译器视这种做法合理，所以除非你重新回顾这段代码，不然这个异常就丢失了。当异常发生时，它被吞了。这是最简单但也可能是最糟糕的一种处理方式了。

第二版中，做了一些改进，我们在处理异常时打印了对应的 log。但是我们在那个时间点还是不知道应该怎么处理它。这个章节我们将提供几个处理异常时的可选项。

### History

各种语言的异常发展历史，pass

### Perspectives

各种语言的异常发展历史，pass

### Passing exceptions to the console

在简单的代码段中，最简单的异常处理可能就是将异常抛出不处理。比如我们要打开/关闭一个文件，我们就会需要处理一些 IO Exception。

```java
public class MainException {
    // Pass all exceptions to the console:
    public static void main(String[] args) throws Exception {
        // Open the file:
        FileInputStream file = new FileInputStream("MainException.java");
        // Use the file ...
        // Close the file:
        file.close();
    }
}
```

main() 也是一个可以携带异常签名的方法，上面的例子中，它带有一个异常 Exception，是所有受检异常的基类。通过把它在方法签名中抛出我们就不用在代码段中写 try-catch 了。

### Converting checked to unchecked exceptions

从 main() 中抛出异常很方面，但是不实用。很多时候，你在调用其他方法的时候，你会想，我不知道怎么处理该异常，但是我又不想只是打印一条信息。这时我们可以简单的在这个异常外面套一个壳变成 Runtimexception。

```java
try {
 // ... to do something useful
} catch(IDontKnowWhatToDoWithThisCheckedException e) {
 throw new RuntimeException(e);
} 
```

这中处理方式看上去很美好，他可以让你从 checked exception 中解放出来，你没有吞掉它。并且你通过异常链串起来，起初的异常也没有丢失。

虽然不用再写 try-catch 了，但是你还是可以通过 getCause() 处理。

下例中，WrapCheckedException.throwRuntimeException() 可以抛出不同的异常。他们被包裹在 RuntimeException 异常中的 cause 中。TurnOffChecking 中你在调用 throwRuntimeException 时可以不处理 try。

但是如果你想要处理，你也可以在方法调用外面包裹 try 并通过 getCause() 得到原始异常并处理。

```java
class WrapCheckedException {
    void throwRuntimeException(int type) {
        try {
            switch (type) {
                case 0:
                    throw new FileNotFoundException();
                case 1:
                    throw new IOException();
                case 2:
                    throw new RuntimeException("Where am I?");
                default:
                    return;
            }
        } catch (Exception e) { // Adapt to unchecked:
            throw new RuntimeException(e);
        }
    }
}

class SomeOtherException extends Exception {}

public class TurnOffChecking {
    public static void main(String[] args) {
        WrapCheckedException wce = new WrapCheckedException();
        // You can call throwRuntimeException() without a try
        // block, and let RuntimeExceptions leave the method:
        wce.throwRuntimeException(3);
        // Or you can choose to catch exceptions:
        for (int i = 0; i < 4; i++)
            try {
                if (i < 3)
                    wce.throwRuntimeException(i);
                else
                    throw new SomeOtherException();
            } catch (SomeOtherException e) {
                System.out.println("SomeOtherException: " + e);
            } catch (RuntimeException re) {
                try {
                    throw re.getCause();
                } catch (FileNotFoundException e) {
                    System.out.println("FileNotFoundException: " + e);
                } catch (IOException e) {
                    System.out.println("IOException: " + e);
                } catch (Throwable e) {
                    System.out.println("Throwable: " + e);
                }
            }
    }
}
// FileNotFoundException: java.io.FileNotFoundException
// IOException: java.io.IOException
// Throwable: java.lang.RuntimeException: Where am I?
// SomeOtherException: reading.container.SomeOtherException
```

当然你也可以包装一个 RuntimeException 的子类，并用它包装你捕获的受检异常。

## Exception guidelines

1. 在恰当的地方处理异常(Avoid catching exceptions unless you know what to do with them.)
2. Fix the problem and call the method that caused the exception again.
3. 修复问题，不要用 retry
4. Calculate some alternative result instead of what the method was supposed to produce.
5. Do whatever you can in the current context and rethrow the same exception to a higher context.
6. Do whatever you can in the current context and throw a different exception to a higher context.
7. Terminate the program.
8. Simplify. (If your exception scheme makes things more complicated, then it is painful and annoying to use.)
9. Make your library and program safer. (This is a short-term investment for debugging, and a long-term investment for application robustness.) 

## Summary

Exception 是 Java 的一部分，如果你不了解他，那么你能做的事情就非常有限了。

异常处理的一大好处是，你可以将你的业务逻辑和异常处理代码分开。已经异常处理涵盖两部分，报告异常和修复程序，但是从遗忘的经验中来看，修复这个功能貌似都是难以实现的，或者压根不可能实现。

不管怎么说，我始终相信，报告异常才是异常处理的主要职责。通过异常机制，你可以将更多的精力集中到更有趣，有挑战的部分。
