---
title: Java 面试之多线程
date: 2021-03-26 16:23:12
categories:
- 面试题
tags:
- 多线程
---

备忘一下多线程相关的面试题

## Java 如何开启线程，怎么保证线程安全

进程是操作系统分配**资源**的最小系统，线程是系统进行分配**任务**的最小单元，线程隶属于进程。

如何开启：

1. 继承 Thread, 重写 run 方法
2. 实现 Runable 接口，实现 run 方法
3. 实现 Callable 接口，实现 call 方法。通过 FeatureTask 创建一个线程，获取线程执行返回结果
4. 通过线程池开启线程

3，4 有对应的只是储备可以延伸一下，不然就别提了, 这两个都是在 concurrent 包下的，等学习那个包的时候再看

为什么要有以上两种方式：Java 采用单继承，多实现的设计模式

```java
// extends class 实现
public class ThreadDemo extends Thread {
    @Override
    public void run() {
        for (int i = 0; i < 1000; i++) {
            System.out.println("ThreadDemo print count: " + i)
        }

    }

    public static void main(String[] args) {
        ThreadDemo threadDemo1 = new ThreadDemo();
        ThreadDemo threadDemo2 = new ThreadDemo();
        ThreadDemo threadDemo3 = new ThreadDemo();
        threadDemo1.start();
        threadDemo2.start();
        threadDemo3.start();
    }
}

// 也可以用 lambda 简写, 但是 lambda 的形式是不能复用的，一次性产品
new Thread(() -> System.out.println("ThreadDemo print count: " + i)).start();

// implement class 实现
public class RunnableDemo implements Runnable {
    @Override
    public void run() {
        for (int i = 0; i < 1000; i++) {
            System.out.println(Thread.currentThread().getName() + " - RunnableDemo count: " + i);
        }
    }

    public static void main(String[] args) {
        RunnableDemo runnableDemo = new RunnableDemo();

        new Thread(runnableDemo, "1").start();
        new Thread(runnableDemo, "2").start();
        new Thread(runnableDemo, "3").start();
    }
}
```

怎么保证线程安全：加锁

1. JVM 锁， Synchronized
2. JDK 锁， Lock

## Volatile 和 Synchronized 有什么区别？ Volatile 能不能保证线程安全？ DCL(Double Check Lock) 单例为什么要加 Volatile?

Syncronized 用于加锁。Volatile 只保持变量的线程可见性，通常用于一个线程写，多个线程读取的情况

Volatile 不能保证线程安全，只保证可见行，不保证原子性

示例：

```java
public class VolatileDemo {
    public static /**volatile**/ boolean flag = true;

    public static void main(String[] args) throws InterruptedException {
        new Thread(() -> {
            while (flag) {
            }
            System.out.println("-------- End Thread --------");
        }).start();

        Thread.sleep(100);
        System.out.println("-------- Set flag to false --------");
        flag = false;
    }
}

// 当注释掉 volatile 时，终端只输出 set flag 提示并挂起
// -------- Set flag to false --------
// 当使用 volatile 时，程序正常结束
// -------- Set flag to false --------
// -------- End Thread --------
```

TODO: 补一张主/从内存 copy 图

DCL(Double Check Lock) 单例为什么要加 Volatile：防止指令重排，防止高并发下指令重拍导致的线程安全问题。

DCL 示例：

```java
public class SingleDemo {
    private static SingleDemo singleDemo;

    private SingleDemo() {}

    public static SingleDemo getInstance() {
        if (null == singleDemo) {
            synchronized (SingleDemo.class) {
                if (null == singleDemo) {
                    singleDemo = new SingleDemo();
                }
            }
        }
        return singleDemo;
    }
}
```

TODO: 补图

## Java 线程锁机制是什么怎么样的？偏向锁，轻量级锁，重量级锁有什么区别，锁机制如何升级

和视频给的结果不一样，视频中，只有 markword 不一样，体现出这个标志位表示了锁状态，表示很清楚问什么，难道是平台问题，回去后用 Windows 试一下
```xml
<!-- https://mvnrepository.com/artifact/org.openjdk.jol/jol-core -->
<dependency>
    <groupId>org.openjdk.jol</groupId>
    <artifactId>jol-core</artifactId>
    <version>0.14</version>
</dependency>
<!-- 可以打印出 Java 对象在内存中的分布情况 -->
```

```java
import org.openjdk.jol.info.ClassLayout;

public class JolDemo {
    public static void main(String[] args) {
        Object o = new Object();
        System.out.println(ClassLayout.parseInstance(o).toPrintable());

        synchronized (o) {
            System.out.println(ClassLayout.parseInstance(o).toPrintable());
        }
    }
}

// java.lang.Object object internals:
//  OFFSET  SIZE   TYPE DESCRIPTION                               VALUE
//       0     4        (object header)                           01 00 00 00 (00000001 00000000 00000000 00000000) (1)
//       4     4        (object header)                           00 00 00 00 (00000000 00000000 00000000 00000000) (0)
//       8     4        (object header)                           e5 01 00 f8 (11100101 00000001 00000000 11111000) (-134217243)
//      12     4        (loss due to the next object alignment)
// Instance size: 16 bytes
// Space losses: 0 bytes internal + 4 bytes external = 4 bytes total

// java.lang.Object object internals:
//  OFFSET  SIZE   TYPE DESCRIPTION                               VALUE
//       0     4        (object header)                           d0 a9 41 0d (11010000 10101001 01000001 00001101) (222407120)
//       4     4        (object header)                           00 70 00 00 (00000000 01110000 00000000 00000000) (28672)
//       8     4        (object header)                           e5 01 00 f8 (11100101 00000001 00000000 11111000) (-134217243)
//      12     4        (loss due to the next object alignment)
// Instance size: 16 bytes
// Space losses: 0 bytes internal + 4 bytes external = 4 bytes total
```

Java 的锁就是在对象的 Markword 中记录的一种状态，无锁，偏向锁，轻量级锁，重量级锁 对应标志位的不同状态

Java 的锁机制就是根据资源竞争的激烈程度不断进行锁升级的过程

TODO：补图

JVM 锁优化：休眠 5s 之后对象自带偏向锁
```java
public class JolDemo {
    public static void main(String[] args) throws InterruptedException {
        Object o = new Object();
        System.out.println(ClassLayout.parseInstance(o).toPrintable());

        // -XX:UseBiasedLocking 是否打开偏向所，默认关闭
        // -XX:BiasedLockingStartupDelay 默认开启
        Thread.sleep(5000);
        Object o2 = new Object();
        System.out.println(ClassLayout.parseInstance(o2).toPrintable());
    }
}

// java.lang.Object object internals:
//  OFFSET  SIZE   TYPE DESCRIPTION                               VALUE
//       0     4        (object header)                           01 00 00 00 (00000001 00000000 00000000 00000000) (1)
//       4     4        (object header)                           00 00 00 00 (00000000 00000000 00000000 00000000) (0)
//       8     4        (object header)                           e5 01 00 f8 (11100101 00000001 00000000 11111000) (-134217243)
//      12     4        (loss due to the next object alignment)
// Instance size: 16 bytes
// Space losses: 0 bytes internal + 4 bytes external = 4 bytes total

// java.lang.Object object internals:
//  OFFSET  SIZE   TYPE DESCRIPTION                               VALUE
//       0     4        (object header)                           05 00 00 00 (00000101 00000000 00000000 00000000) (5)
//       4     4        (object header)                           00 00 00 00 (00000000 00000000 00000000 00000000) (0)
//       8     4        (object header)                           e5 01 00 f8 (11100101 00000001 00000000 11111000) (-134217243)
//      12     4        (loss due to the next object alignment)
// Instance size: 16 bytes
// Space losses: 0 bytes internal + 4 bytes external = 4 bytes total
```

## 谈谈对 AQS 的理解

## 有 ABC 三个线程，如何保证三个线程同时执行，如何在并发情况下保证三个线程一次执行，如何保证三个线程有序交错进行

## 如何对一个字符串快速进行排序（多线程快排）
