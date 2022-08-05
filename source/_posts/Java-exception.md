---
title: Java 异常处理
date: 2022-02-21 10:40:14
categories:
- java
tags:
- exception
- best practice
---

## 记录 log 并附带 stack trace

今天看到同事给我代码 review 的时候推荐使用 log.info(e.getMessage()) 时，不太清楚推荐的原因，特意 Google 了一下几种 log 记录方式

* info(ex)
* info(ex.getMessage())
* info("msg", ex)

总的来说，第三种最好，前两种只会记录当前类的异常抛出记录，之前的信息都 miss 掉了

PS: 在 SCIM API 项目中，实现也是只传了 e.getmessage() 而没有传递 ex 这个对象，导致追踪困难，引以为鉴

```java
public class ExpTest {
    public static int testMethod() {
        return 1 / 0;
    }
}

public class ExpClient {
    private static final Logger logger = Logger.getLogger(ExpClient.class);

    public static void main(String[] args) {
        try {
            ExpTest.testMethod();
        } catch (Exception e) {
            System.out.println("------------------------------> msg, e <------------------------------");
            logger.info("err...", e);
            System.out.println("------------------------------> e <------------------------------");
            logger.info(e);
            System.out.println("------------------------------> msg <------------------------------");
            logger.info(e.getMessage());
        }
    }
}

// 终端输出如下：
// ------------------------------> msg, e <------------------------------
//  INFO [main] (ExpClient.java:14) - err...
// java.lang.ArithmeticException: / by zero
// 	at sementic.ExpTest.testMethod(ExpTest.java:5)
// 	at sementic.ExpClient.main(ExpClient.java:11)
// ------------------------------> e <------------------------------
//  INFO [main] (ExpClient.java:16) - java.lang.ArithmeticException: / by zero
// ------------------------------> msg <------------------------------
//  INFO [main] (ExpClient.java:18) - / by zero
```

## try-catch 执行流程

今天在写一段补偿代码的时候突然想到一个问题，当异常发生时，后续代码是否还会被执行的问题。测试前根据主观猜测，感觉如果有 try-catch, 那么会继续执行；没有则直接跳出了，相当于 return.

## Scenario 1

```java
public class Test1 {
    public static void main(String[] args) {
        try {
            int a = 1/0;
        } catch (Exception e) {
            e.printStackTrace();
        }
        System.out.println("Log after exception");
    }
}

// > Task :idl-sfutil:idl-sfutil-service:Test1.main()
// Log after exception
```

抛错后继续执行

## Scenario 2

```java
public class Test1 {
    public static void main(String[] args) {
        int a = 1/0;
        System.out.println("Log after exception");
    }
}

// Exception in thread "main" java.lang.ArithmeticException: / by zero
// 	at com.sf.sfv4.util.Test1.main(Test1.java:5)
```

抛错后没有执行
