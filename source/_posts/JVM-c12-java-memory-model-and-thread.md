---
title: Java内存模型与线程
date: 2021-04-01 18:45:44
categories:
- JVM
tags:
- 并发
- concurrent
---

深入理解 Java 虚拟机第 12 章Java内存模型与线程笔记

## 12.3 Java 内存模型

Java 内存模型(JMM) 用来屏蔽各种硬件和操作系统的内存访问差异，已实现 Java 程序在各种平台下都能达到一致的内存访问效果。 PS：看起来这就是传说中的一次编译，到处运行的功能吧。

## 12.3.1 主内存和工作内存

JMM 规定，

* 所有变量都存储在主内存(Main Memory)中
* 每条线程有自己的工作内存(Working Memory)
* 线程对变量的所有操作都必须在工作内存中进行，而不能直接读写主内存中的数据
* 不同线程不能访问对方的工作内存中的变量
* 线程间值传递需要通过主内存完成

```txt
  +-----------+          +-----------+           +-------+                                                                                             
  | Java      |  <-----> | Working   |  <----->  |       |                                                                                             
  | Thread    |          | Memory    |           |       |         +---------------+                                                                   
  +-----------+          +-----------+           | Save  |         |               |                                                                   
                                                 | &     |         |  Main Memory  |                                                                   
  +-----------+          +-----------+           | Load  | <-----> |               |                                                                   
  | Java      |          | Working   |           |       |         |               |                                                                   
  | Thread    |  <-----> | Memory    |  <----->  |       |         |               |                                                                   
  +-----------+          +-----------+           |       |         |               |                                                                   
                                                 |       |         |               |                                                                   
  +-----------+          +-----------+           |       |         +---------------+                                                                   
  | Java      |          | Working   |           |       |                                                                                             
  | Thread    |  <-----> | Memory    |  <----->  |       |                                                                                             
  +-----------+          +-----------+           +-------+                                
```

## 12.3.2 内存间交互操作

JMM 规定了主内存和工作内存之间的具体交互协议，定义了 8 中操作，这些操作都要求是原子的，不可再分的(对 double/long 类型的变量来说， laod, store,read,write 操作在某些平台上允许例外)

* lock - 作用主内存，把变量标识为线程独占
* unlock - 作用主内存，把变量从锁定状态释放
* read - 作用主内存，将变量从主内存传输到工作内存
* load - 作用工作内存，把 read 操作得到的变量放入工作内存的变量副本中
* use - 作用工作内存，把变量传给执行引擎
* assign - 作用工作内存，将工作引擎计算结果赋值给变量
* store - 作用工作内存，将工作内存变量值传给主内存
* write - 作用主内存，把 store 操作传过来的变量放入主内存

JMM 还对这八种操作做了一些限制，比如 store 和 write 必须顺序执行，不允许一个线程丢弃它最近的 assign 操作等，这些细节这里就不深究了，感觉是很理论的东西，以后真的用到了，再回头看啊。

## 12.3.3 对于 volatile 型变量的特殊规则

volatile 是 JVM 提供的最轻量级的同步机制，JMM 专门为他定义了一些特殊规则。当一个变量定义为 volatile 后，它具备两个特性：

1. 保证变量对所有线程的可见性，当一条线程修改了这个变量的值，新值对于其他线程立即可知
2. 禁止指令重排序优化 - PS: java 1.5 之前这个关键字有问题，并不能保证可见性

```java
/**
* 实验 #1
* 声明一个 volatile 的 int 变量，然后通过多线程进行累加，统计最终计算结果。虽然 volatile 保证了变量的线程可见性，但是由于 race++ 是非原子性的，所以计算结果是错误的。
**/
public class VolatileTest {
    public static volatile int race = 0;

    public static void increase() {
        race++;
    }

    private static final int THREADS_COUNT = 20;

    public static void main(String[] args) {
        Thread[] threads = new Thread[THREADS_COUNT];
        for (int i = 0; i < THREADS_COUNT; i++) {
            threads[i] = new Thread(new Runnable() {
                @Override
                public void run() {
                    for (int i = 0; i < 10000; i++) {
                        increase();
                    }
                }
            });
            threads[i].start();
        }

        /*
        * 这个实验用 Idea 会失败，Idea 在起线程的时候会通过守护进程的方式，所以 activeCount 一直为 2, 死循环。使用 Eclipse 则能正常工作。
        */

        while (Thread.activeCount() > 2) {
            Thread.yield();
        }
        System.out.println(race);
    }
}

// 49474
```

volatile 只保证可见性，在不符合以下两条规则的运算场景中，还是需要通过加锁保证原子性：

* 运算结果并不依赖变量的当前值，或者能够保证只有单一的线程修改变量值
* 变量不需要与其他的状态变量共同参与不变约束

```java
/**
* 实验 #2-1
* 禁止指令重排序优化为代码。在没有添加 volatile 修饰的时候，由于指令重排序优化，initialized = true 可能被提前执行，导致线程 B 执行异常
**/
Map configOptions;
char[] configText;
// 此变量必须定义为 volatile
volatile boolean initialized = false;

// 假设以下代码在线程 A 中进行
// 模拟读取配置星系，当读取完成后
// 将 initialized 设置为 true，同志其他线程配置可用
configOptions = new HashMap();
configText = readConfigFile(fileName);
processConfigOptions(configText, configOptions);
initialized = true

// 假设以下代码在线程 B 中进行
// 等待 initialized 为 true，代表 A 已完成初始化
while(!initialized) {
    sleep();
}
// 使用线程 A 中初始化好的配置信息
doSomethingWithConfig();
```

```java
/**
* 实验 #2-2
* DCL 双锁检测
**/
public class Singleton {
    private static volatile Singleton instance;

    private Singleton() {}

    public static Singleton getInstance() {
        if (instance == null) {
            syncronized(Singleton.class) {
                if (instance == null) {
                    return new Singleton();
                }
            }
        }
    }

    public static void main(String[] args) {
        Singleton.getInstance();
    }
}
```

本章最后还介绍了 volatile 底层实现和 JMM 中定义的规则，暂时就不深入了。