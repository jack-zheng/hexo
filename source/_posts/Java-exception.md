---
title: Java 异常处理流程
date: 2022-02-21 10:40:14
categories:
- java
tags:
- exception
---

今天在写一段补偿代码的时候突然想到一个问题，当异常发生时，后续代码是否还会被执行的问题。测试前根据主管猜测，感觉如果有 try-catch, 那么会继续执行；没有则直接跳出了，相当于 return.

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
