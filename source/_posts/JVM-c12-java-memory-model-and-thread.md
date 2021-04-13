---
title: Java内存模型与线程
date: 2021-04-01 18:45:44
categories:
- JVM
tags:
- 并发
- JMM
- 内存模型
- concurrent
---

深入理解 Java 虚拟机第 12 章Java内存模型与线程笔记

## 12.3 Java 内存模型

Java 内存模型(JMM) 用来屏蔽各种硬件和操作系统的内存访问差异，已实现 Java 程序在各种平台下都能达到一致的内存访问效果。 PS：看起来这就是传说中的一次编译，到处运行的功能吧。

### 12.3.1 主内存和工作内存

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

### 12.3.2 内存间交互操作

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

### 12.3.3 对于 volatile 型变量的特殊规则

volatile 是 JVM 提供的最轻量级的同步机制，JMM 专门为他定义了一些特殊规则。当一个变量定义为 volatile 后，它具备两个特性：

1. 保证变量对所有线程的可见性，当一条线程修改了这个变量的值，新值对于其他线程立即可知
2. 禁止指令重排序优化 - PS: java 1.5 之前这个关键字有问题，并不能保证可见性

```java
/**
* 实验 #1 volatile 虽然是线程可见的，但是多线程同时写操作时依然线程不安全
*
* 声明一个 volatile 的 int 变量，给初始值 0，起 20 个线程，每个线程都对变量做 10000 次加 1 操作，统计最终计算结果。
*
* 结论：虽然 volatile 保证了变量的线程可见性，但是由于 race++ 是非原子性的。具体情况可能如下：
* 线程A：进行累加操作，取得计算前的值 100，并进行累加操作
* 线程B：取得累加前的值 100 进行操作
* 线程A：完成操作 101 并赋值给主内存
* 线程B：完成操作 101 并赋值给主内存
* 所以计算结果总是小于理论值 20 0000
**/
public class VolatileTest {
    public static volatile int race = 0;

    public static void main(String[] args) {
        for (int i = 0; i < 20; i++) {
            new Thread(() -> {
                for (int i = 0; i < 10000; i++) {
                    race++;
                }
            }).start();
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

// 34490
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

### 12.3.4 针对 long 和 double 型变量的特殊规则

虚拟机允许将没有被 volatile 修饰的 64 位数据的读写操作分为两次 32 位的操作进行，即不保证 64 位数据类型的 load, store, read, write 的原子性。这就是所谓的 'long 和 double 的非原子性协定'， 目前主流商用 64 位虚拟机并不会出现非原子访问。

### 12.3.5 原子性，可见性与有序性

**原子性**：JMM 直接保证 read, load, assign, use, store 和 write 操作的原子性，对于一个更大范围的原子性保证，JMM 提供了 lock 和 unlock 这两个操作，反应到字节码就是 monitorenter/monitorexit，Java 代码层面就是 synchronized 关键字了

**可见性**：当一个线程修改了共享变量值时，其他线程能够立即得知这个修改。对于 volatile 类型的数据，JMM 通过修改后立即同步回主内存，在变量读取前刷新变量值以保证可见性，普通变量不是立即执行的。

除了 volatile 外，synchronized 和 final 也能实现可见性。synchronized 通过规则：在 unlock 之前必须把此变量同步回主内存中(执行 store，write)来达到目的。final 的可见性指：被 final 修饰的字段在构造器中一旦被初始化完成，并且构造器没有吧 this 引用传递出去，那么其他线程就能看到 final 字段的值。

```java
/**
* i， j 都具备可见性，不许同步就可以被其他线程访问
**/
public static final int i;
public final int j;

static {
    i = 0;
    // 后续省略
}

{
    // 可也以在构造函数中初始化
    j = 0;
    // 后续省略
}
```

**有序性**：如果在本线程内观察，所有的操作都是有序的；如果在一个线程中观察另一个线程，所有操作都是无序的。volatile 和 synchronized 关键字可用于保证线程操作之间的有序性，volatile 本省包含禁止指令重排的语义，而 synchronized 则是由 ‘一个变量在同一时刻只允许一条线程对其进行 lock 操作’ 的这条规则决定。