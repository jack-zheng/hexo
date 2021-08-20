---
title: TIJ4 23 concurrency
date: 2021-08-17 18:59:59
categories:
- TIJ4
tags:
- concurrency
---

## The manay faces of concurrency

并发看上去很让人摸不着头脑，主要是应为我们需要解决多个问题，并且解决问题的方法很多。这两者间也没有明确的匹配关系。所以你必须要全面的理解各种问题和场景才能更高效的使用并发。

使用并发可以解决的问题可以粗略归纳为两类

### Faster execution

多处理器系统通过并发可以提高效率，这很容易理解。但是有时但处理器系统通过并发也能提高效率，听上去可能有点反直觉，但是确实如此。一般来说，在但处理器系统中使用多线程，会有上下文切换(context switch)开销导致性能下降。但是如果场景中有较多的 IO 操作，则可能开销不增反降。

### Improving code design

单核系统中实现多线程，本质上，一个时间点也只能做一件事，理论上，我们可以将这个多线程转化为单线程实现。但是有时多线程可以提供更好的组织方式，比如在模拟动画的场景上。

Java 中多线程是有优先级的。通过这个优先级，JVM 会分配不同的时间片给程序执行。

## Basic threading

并发可以帮你讲你的程序分成独立的 task，每个 task 可以通过 processor 中的一个 thread 执行。每个 thread 可以线性的执行程序。通过这种方式单核 CUP 也能执行多线程，而且这对使用者是透明的，你不需要关心他的具体实现。

### Defining tasks

并发中一个 thread 对应一个 task, 我们通过实现 Runnable 接口并实现 run() 方法的形式实现并发。

示例说明:

多线程打印变量指，run() 方法中有个 while 循环让 countDown 值递减，然后调用 print 方法打印状态信息。countDown = 0 时结束 task 并推出。

Thread.yield() 是 Thread 类自带的方法，作用是告诉 CPU 现在是时候让渡时间片了。

```java
public class LiftOff implements Runnable {
    protected int countDown = 10; // Default
    private static int taskCount = 0;
    private final int id = taskCount++;

    public LiftOff() {
    }

    public LiftOff(int countDown) {
        this.countDown = countDown;
    }

    public String status() {
        return "#" + id + "(" + (countDown > 0 ? countDown : "Liftoff!") + "), ";
    }

    public void run() {
        while (countDown-- > 0) {
            System.out.print(status());
            Thread.yield();
        }
    }
}

public class MainThread {
    public static void main(String[] args) {
        LiftOff launch = new LiftOff();
        launch.run();
    }
}

// #0(9), #0(8), #0(7), #0(6), #0(5), #0(4), #0(3), #0(2), #0(1), #0(Liftoff!),
```

除了调用 run() 方法，还可以将 Runnable 类传给 Thread 类并调用 start() 方法启动线程。下面的例子中，我们在主函数中新建五个线程

```java
public class MoreBasicThreads {
    public static void main(String[] args) {
        for (int i = 0; i < 5; i++)
            new Thread(new LiftOff()).start();
        System.out.println("Waiting for LiftOff...");
    }
}

// #0(9), #1(9), #2(9), #0(8), #2(8), #1(8), #2(7), #0(7), #2(6),
// #3(9), #2(5), #3(8), #4(9), #3(7), #0(6), #3(6), Waiting for LiftOff...
// #1(7), #3(5), #0(5), #4(8), #2(4), #4(7), #0(4), #4(6), #3(4),
// #1(6), #3(3), #1(5), #4(5), #1(4), #0(3), #2(3), #1(3), #4(4),
// #2(2), #4(3), #3(2), #4(2), #2(1), #1(2), #0(2), #2(Liftoff!),
// #0(1), #4(1), #0(Liftoff!), #3(1), #4(Liftoff!), #1(1),
// #3(Liftoff!), #1(Liftoff!),
```

从输出我们可以看出来，各个线程的 task 是混合执行的，通过 thread scheduler 调度。如果你使用的是多核系统，scheduler 会帮你讲这些 task 分配到不同核上计算。

main() 函数并不会持有创建出来的 Thread 的引用。对普通的对象来说，这会影响到垃圾回收，但是 Thread 有特殊的机制保证这一点。他会一直存在知道 run() 方法结束为止。

### Using Executors

Java SE5 以来，提供了另一种执行并发的方式 - Executor. 通过它你就不需要在 Client 中新建 Thread 来执行这个 task 了。Executor 是 Java5/6 中提倡的运行并发的方式。

Executor 提供了多种运行方式，有 CachedThreadPool, FixedThreadPool 和 SingleThreadPool.

CachedThreadPool 的例子如下

```java
public class CacheThreadPool {
    public static void main(String[] args) {
        ExecutorService exec = Executors.newCachedThreadPool();
        for (int i = 0; i < 5; i++) {
            exec.execute(new LiftOff());
        }
        exec.shutdown();
    }
}

// #0(9), #1(9), #2(9), #0(8), #2(8), #1(8), #3(9), #2(7), 
// #0(7), #2(6), #3(8), #2(5), #1(7), #3(7), #2(4), #4(9), 
// #4(8), #0(6), #4(7), #2(3), #3(6), #1(6), #3(5), #1(5), 
// #2(2), #0(5), #0(4), #4(6), #0(3), #2(1), #1(4), #3(4), 
// #2(Liftoff!), #3(3), #0(2), #4(5), #0(1), #3(2), #1(3), 
// #3(1), #0(Liftoff!), #3(Liftoff!), #4(4), #1(2), #4(3), 
// #1(1), #4(2), #1(Liftoff!), #4(1), #4(Liftoff!),
```

task execute 之后需要调用 shutdown 方法防止新的 task 被提交到 Executor 中。

下面是 FixedThreadPool 的使用方法，和前面基本一样，我们可以自定义 pool size

```java
public class FixedThreadPool {
    public static void main(String[] args) {
        ExecutorService exec = Executors.newFixedThreadPool(5);
        for (int i = 0; i < 5; i++) {
            exec.execute(new LiftOff());
        }
        exec.shutdown();
    }
}
```

相比于 CachedThreadPool, FixedThreadPool 会在开始前一并将制定的 Thread 都创建完以节省创建成本。同时由于指定了线程数，可以防止资源滥用。

书中例子都是用的 CachedTheadPool, 因为方便，他的机制是，更具使用情况创建线程，如果之前的线程用完了，会重用。产品代码还是尽量使用 FixedThreadPool 为好。

SingleThreadPool 和 FixedThreadPool 很像，只不过限定只能是单线程。这种情况在 常驻线程 和 临时线程 的情况下很有用。如果多个 task 被提交到 SingleThreadPool 中的话，他会顺序执行所有的线程。

```java
public class SingleThreadExecutor {
    public static void main(String[] args) {
        ExecutorService exec = Executors.newSingleThreadExecutor();
        for (int i = 0; i < 5; i++) {
            exec.execute(new LiftOff());
        }
        exec.shutdown();
    }
}

// #0(9), #0(8), #0(7), #0(6), #0(5), #0(4), #0(3), #0(2), 
// #0(1), #0(Liftoff!), #1(9), #1(8), #1(7), #1(6), #1(5), 
// #1(4), #1(3), #1(2), #1(1), #1(Liftoff!), #2(9), #2(8), 
// #2(7), #2(6), #2(5), #2(4), #2(3), #2(2), #2(1), #2(Liftoff!), 
// #3(9), #3(8), #3(7), #3(6), #3(5), #3(4), #3(3), #3(2), #3(1), 
// #3(Liftoff!), #4(9), #4(8), #4(7), #4(6), #4(5), #4(4), #4(3), 
// #4(2), #4(1), #4(Liftoff!),
```

通过 SingleThreadExectuor 你可以确保同一时间只有一个线程占用某个资源。如果读写文件系统时，可以通过这中 executor 避免死锁。当然最常见的还是给资源加锁，后面有介绍。

### Producing return values from tasks

Runnable 的方式，当 run 结束时即退出，是没有返回值的，如果想要在 task 结束后返回值，可以使用 Callable 接口，从 Java 5 开始支持这个接口。他只能通过 ExecutorService 的 submit 进行调用。

```java
public class CallableDemo {
    public static void main(String[] args) throws ExecutionException, InterruptedException {
        ExecutorService exec = Executors.newCachedThreadPool();
        ArrayList<Future<String>> results = new ArrayList<>();
        for (int i = 0; i < 10; i++) {
            results.add(exec.submit(new TaskWithResult(i)));
        }
        exec.shutdown();
        for (Future<String> fs : results) {
            System.out.println(fs.get());
        }
    }
}

// result of TaskWithResult 0
// result of TaskWithResult 1
// result of TaskWithResult 2
// result of TaskWithResult 3
// result of TaskWithResult 4
// result of TaskWithResult 5
// result of TaskWithResult 6
// result of TaskWithResult 7
// result of TaskWithResult 8
// result of TaskWithResult 9
```

submit() 方法会产生一个 Future 对象来存储 Callable 的结果。Future 提供 isDone() 方法用以检测 task 是否执行结束。结束后可以执行 get() 方法得到结果。如果直接调用 get() 但是 task 还没有 done, 那 get() 就会 block 直到 task 完成，你也可以为 get() 设置 timeout。

### Sleeping

在 task 中，你可以通过调用 sleep() 方法来影响 task 的执行。

```java
public class SleepingTask extends LiftOff {
    @Override
    public void run() {
        try {
            while (countDown-- > 0) {
                System.out.println(status());
                // Old-style: Thread.sleep(100);
                // Java SE5/6-style
                TimeUnit.MILLISECONDS.sleep(100);
            }
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }

    public static void main(String[] args) {
        ExecutorService exec = Executors.newCachedThreadPool();
        for (int i = 0; i < 5; i++) {
            exec.execute(new SleepingTask());
        }
        exec.shutdown();
    }
}
// #0(9), #2(9), #1(9), #3(9), #4(9), 
// #0(8), #2(8), #1(8), #4(8), #3(8), 
// #4(7), #1(7), #3(7), #2(7), #0(7), 
// #1(6), #4(6), #3(6), #0(6), #2(6), 
// #1(5), #4(5), #3(5), #0(5), #2(5), 
// #4(4), #0(4), #2(4), #3(4), #1(4), 
// #4(3), #0(3), #1(3), #3(3), #2(3), 
// #3(2), #2(2), #0(2), #4(2), #1(2), 
// #2(1), #3(1), #4(1), #0(1), #1(1), 
// #4(Liftoff!), #3(Liftoff!), #2(Liftoff!), #0(Liftoff!), #1(Liftoff!),
```

sleep() 会抛出 InterruptedException 异常，你必须在 run() 方法中处理他，因为异常是不能被传递到 main 中的。 TimeUnit 是对 Thread.sleep() 更精确的处理方式。

从输出内容我们可以看到，加了 sleep 之后 task 以轮训的形式输出，但是这个是不能保证的，不同的操作系统可能有不同的行为。

### Priority

priority 表示 thread 在 scheduler 总的重要程度。scheduler 会倾向于更频繁的调用 priority 高的 thread。

一般来说，你不需要人为的制定 thread priority，系统会自动为你分配。你可以调用 getPriority()/setPriority() 查看，指定优先级

示例如下，和前面的 LiftOff 基本一致，只是 run() 中的实现改为 10w 浮点计算

```java
public class SimplePriorities implements Runnable {
    private int countDown = 5;
    private volatile double d; // No optimization
    private int priority;

    public SimplePriorities(int priority) {
        this.priority = priority;
    }

    @Override
    public String toString() {
        return Thread.currentThread() + ": " + countDown;
    }

    @Override
    public void run() {
        Thread.currentThread().setPriority(priority);
        while (true) {
            // An expensive, interruptable operation
            for (int i = 0; i < 100000; i++) {
                d += (Math.PI + Math.E) / (double) i;
                if (i % 1000 == 0)
                    Thread.yield();
            }
            System.out.println(this);
            if (--countDown == 0) return;
        }
    }

    public static void main(String[] args) {
        ExecutorService exec = Executors.newCachedThreadPool();
        for (int i = 0; i < 5; i++) {
            exec.execute(new SimplePriorities(Thread.MIN_PRIORITY));
        }
        exec.execute(new SimplePriorities(Thread.MAX_PRIORITY));
        exec.shutdown();
    }
}
// Thread[pool-1-thread-2,1,main]: 5
// Thread[pool-1-thread-3,1,main]: 5
// Thread[pool-1-thread-5,1,main]: 5
// Thread[pool-1-thread-1,1,main]: 5
// Thread[pool-1-thread-4,1,main]: 5
// Thread[pool-1-thread-6,10,main]: 5
// Thread[pool-1-thread-3,1,main]: 4
// Thread[pool-1-thread-2,1,main]: 4
// Thread[pool-1-thread-5,1,main]: 4
// Thread[pool-1-thread-4,1,main]: 4
// Thread[pool-1-thread-1,1,main]: 4
// Thread[pool-1-thread-6,10,main]: 4
// Thread[pool-1-thread-3,1,main]: 3
// Thread[pool-1-thread-1,1,main]: 3
// Thread[pool-1-thread-5,1,main]: 3
// Thread[pool-1-thread-2,1,main]: 3
// Thread[pool-1-thread-4,1,main]: 3
// Thread[pool-1-thread-3,1,main]: 2
// Thread[pool-1-thread-6,10,main]: 3
// Thread[pool-1-thread-1,1,main]: 2
// Thread[pool-1-thread-5,1,main]: 2
// Thread[pool-1-thread-3,1,main]: 1
// Thread[pool-1-thread-6,10,main]: 2
// Thread[pool-1-thread-4,1,main]: 2
// Thread[pool-1-thread-2,1,main]: 2
// Thread[pool-1-thread-1,1,main]: 1
// Thread[pool-1-thread-6,10,main]: 1
// Thread[pool-1-thread-5,1,main]: 1
// Thread[pool-1-thread-4,1,main]: 1
// Thread[pool-1-thread-2,1,main]: 1
```

Thread 的 toString 方法有自定义过，输出时会打印 thread name + priority + group name. 

Mac 上跑这个实验效果并不明显，期望值应该是 CPU 会优先执行 priority 为 5 的线程才对。。。

下面解释 d 变量增加这个 volatile 就是为了防止优化，不然看不到预期结果。难道 mac 上这个设置失效了？！之后调用 yield 释放线权。

JDK 有 10 个等级的优先级设置，不一定和操作系统匹配，比如 Windows 只有 7 级而 Linux 系统有 23 级。

### Yielding

通过使用 yield 可以在 task 进行过程中，让渡 CPU 给其他同级别的 task，但是这个让渡并不能被保证，你不能通过他来严格控制 task 的执行顺序。

### Daemon threads

守护进程可以在成勋运行时在后台提供一些其他的基础服务，但是这个服务和程序没关系。当所有 非守进程 的程序结束后，守护进程也会被杀死，程序退出。反之，只要有 非守护进程 没有结束，那么守护进程就不会结束。

示例演示

run 方法总我们指定新建的 thread 为 守护进程。当 main 结束时，守护进程也一起结束。

```java
public class SimpleDaemons implements Runnable {
    @Override
    public void run() {
        try {
            while(true) {
                TimeUnit.MILLISECONDS.sleep(100);
                System.out.println(Thread.currentThread() + " " + this);
            }
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }

    public static void main(String[] args) throws InterruptedException {
        for (int i = 0; i < 10; i++) {
            Thread daemon = new Thread(new SimpleDaemons());
            daemon.setDaemon(true);
            daemon.start();
        }
        System.out.println("All daemons started");
        TimeUnit.MILLISECONDS.sleep(175);
    }
}

// All daemons started
// Thread[Thread-5,5,main] org.jz.c23.SimpleDaemons@501a67ca
// Thread[Thread-8,5,main] org.jz.c23.SimpleDaemons@22e706f4
// Thread[Thread-2,5,main] org.jz.c23.SimpleDaemons@4eba943c
// Thread[Thread-0,5,main] org.jz.c23.SimpleDaemons@c0a422e
// Thread[Thread-7,5,main] org.jz.c23.SimpleDaemons@317704a2
// Thread[Thread-3,5,main] org.jz.c23.SimpleDaemons@5ed3479d
// Thread[Thread-6,5,main] org.jz.c23.SimpleDaemons@2482ac6c
// Thread[Thread-9,5,main] org.jz.c23.SimpleDaemons@205caa11
// Thread[Thread-1,5,main] org.jz.c23.SimpleDaemons@71537d82
// Thread[Thread-4,5,main] org.jz.c23.SimpleDaemons@1b02d47b
```

我们还可以通过定制 ThreadFactory 来生成 Thread. 然后通过 Executors 来做并发

示例说明：

main 中新建了一个 ExecutorService 并用 DaemonThreadFactory 作为参数。通过这种方式，所有创建的并发线程都是守护进程。他们会每隔 100ms 打印一次信息。同时 main 会 sleep 500ms 然后退出。同时守护进程全部结束。

```java
public class DaemonThreadFactory implements ThreadFactory  {
    @Override
    public Thread newThread(Runnable r) {
        Thread t = new Thread(r);
        t.setDaemon(true);
        return t;
    }
}

public class DaemonFromFactory implements Runnable {
    @Override
    public void run() {
        try {
            while(true) {
                TimeUnit.MILLISECONDS.sleep(100);
                System.out.println(Thread.currentThread() + " " + this);
            }
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }

    public static void main(String[] args) throws InterruptedException {
        ExecutorService exec = Executors.newCachedThreadPool(new DaemonThreadFactory());
        for (int i = 0; i < 10; i++) {
            exec.execute(new DaemonFromFactory());
        }
        System.out.println("All daemons started");
        TimeUnit.MILLISECONDS.sleep(500);
    }
}

// All daemons started
// Thread[Thread-9,5,main] org.jz.c23.DaemonFromFactory@627a6c14
// Thread[Thread-6,5,main] org.jz.c23.DaemonFromFactory@43ca4e8
// ....
```

我们还可以使用 ThreadPoolExecutor 来简化上面的操作

```java
public class DaemonThreadPoolExecutor extends ThreadPoolExecutor {
    public DaemonThreadPoolExecutor() {
        super(0, Integer.MAX_VALUE, 60L, 
        TimeUnit.SECONDS, new SynchronousQueue<>(), new DaemonThreadFactory());
    }
}
```

你可以通过调用 isDaemon() 方法查看线程是否为 守护进程，由 守护进程 创建的所有 thread 都会自动变为 守护进程。

示例说明：

main 函数中为 Daemon 创建线程，并制定类型为 守护进程

Daemon 这个进程中会新建并启动十个 thread，逻辑都一样，就是生产后一直空转

打印这十个空转进程的类型，可以看到也是 守护进程

```java
public class Daemons {

    public static void main(String[] args) throws InterruptedException {
        Thread d = new Thread(new Daemon());
        d.setDaemon(true);
        d.start();
        System.out.println("d.isDaemon() = " + d.isDaemon() + ", ");
        // Allow the daemon threads to finish their startup processes
        TimeUnit.SECONDS.sleep(1);
    }

}

class DaemonSpawn implements Runnable {
    @Override
    public void run() {
        while (true) {
            Thread.yield();
        }
    }
}

class Daemon implements Runnable {
    private Thread[] t = new Thread[10];

    @Override
    public void run() {
        for (int i = 0; i < t.length; i++) {
            t[i] = new Thread(new DaemonSpawn());
            t[i].start();
            System.out.println("DaemonSpawn " + i + " started, ");
        }
        for (int i = 0; i < t.length; i++) {
            System.out.println("t[" + i + "].isDaemon() = " + t[i].isDaemon() + ", ");
        }
        while (true)
            Thread.yield();
    }
}
// d.isDaemon() = true, 
// DaemonSpawn 0 started, 
// DaemonSpawn 1 started, 
// DaemonSpawn 2 started, 
// DaemonSpawn 3 started, 
// DaemonSpawn 4 started, 
// DaemonSpawn 5 started, 
// DaemonSpawn 6 started, 
// DaemonSpawn 7 started, 
// DaemonSpawn 8 started, 
// DaemonSpawn 9 started, 
// t[0].isDaemon() = true, 
// t[1].isDaemon() = true, 
// t[2].isDaemon() = true, 
// t[3].isDaemon() = true, 
// t[4].isDaemon() = true, 
// t[5].isDaemon() = true, 
// t[6].isDaemon() = true, 
// t[7].isDaemon() = true, 
// t[8].isDaemon() = true, 
```

注意：守护进程是可以在不执行 finally 的情况下退出的

示例说明：

主函数中新建一个 ADaemon 的 thread 并启动。 ADaemon 的 run 方法会答应 "Starting ADaemon" 并在 1s 后打印 "This should always run?"

祝函数启动并结束后，守护进程的 finally 中的语句并没有打印。

```java
class ADaemon implements Runnable {
    @Override
    public void run() {
        try {
            System.out.println("Starting ADaemon");
            TimeUnit.SECONDS.sleep(1);
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            System.out.println("This should always run?");
        }
    }
}

public class DaemonsDontRunFinally {
    public static void main(String[] args) {
        Thread t = new Thread(new ADaemon());
        t.setDaemon(true);
        t.start();
    }
}
// Starting ADaemon
```

如果我们将 t.setDaemon(true); 注释掉，则 finally 中的内容会被打印出来。

守护进程在没有其他 非守护进程 的时候会立即别 JVM 杀掉。所以一般来说鼓励创建 非守护进程。我们可以通过 Executor 关闭 非守护进程，后面会介绍。

### Coding variations

到现在为止，我们都用 Runnable 实现并发，其实并发还可以通过很多其他不同的方式实现

通过继承 Thread 类, 和 Runnable 的区别：

* Runnable 需要通过 Thread 类或者 Executor 才能启动，Thread 自己就可以启动
* Runnable 启动时需要调用 start() 方法，Thread 不需要，new 完就启动了

```java
public class SimpleThread extends Thread {
    private int countDown = 5;
    private static int threadCount = 0;

    public SimpleThread() {
        super(Integer.toString(++threadCount));
        start();
    }

    @Override
    public String toString() {
        return "#" + getName() + "(" + countDown + "), ";
    }

    @Override
    public void run() {
        while(true) {
            System.out.println(this);
            if (--countDown == 0)
                return;
        }
    }

    public static void main(String[] args) {
        for (int i = 0; i < 5; i++) {
            new SimpleThread();
        }
    }
}
// #1(5), #1(4), #3(5), #3(4), #3(3), #3(2), #3(1), #4(5), #4(4), 
// #2(5), #4(3), #4(2), #1(3), #1(2), #1(1), #4(1), #5(5), #5(4), 
// #2(4), #5(3), #2(3), #2(2), #2(1), #5(2), #5(1), 
```

还可以在 Runnable 的实现中自启动, 这种写法的特点是，在成员变量中，将本身作为参数传给 Thread 的构造函数。在自己的构造函数中，调用 Thread 的 start 方法

```java
public class SelfManaged implements Runnable {
    private int countDown = 5;
    private Thread t = new Thread(this);

    public SelfManaged() {
        t.start();
    }

    @Override
    public String toString() {
        return Thread.currentThread().getName() + "(" + countDown + "), ";
    }

    @Override
    public void run() {
        while(true) {
            System.out.println(this);
            if (--countDown == 0)
                return;
        }
    }

    public static void main(String[] args) {
        for (int i = 0; i < 5; i++) {
            new SelfManaged();
        }
    }
}
// Thread-0(5), Thread-2(5), Thread-1(5), Thread-3(5), Thread-2(4), Thread-2(3), 
// Thread-4(5), Thread-4(4), Thread-0(4), Thread-4(3), Thread-4(2), Thread-2(2), 
// Thread-3(4), Thread-3(3), Thread-3(2), Thread-3(1), Thread-1(4), Thread-2(1), 
// Thread-4(1), Thread-0(3), Thread-0(2), Thread-1(3), Thread-0(1), Thread-1(2), 
// Thread-1(1),
```

PS: 这个例子中使用场景很简单，所以可能没什么风险，但是在构造函数中启动线程可能会很危险。其他 task 可能在这个 task 初始化结束之前就开始使用这个 task 对应的 thread，这个对象此时处于一个不稳定状态。这就是我们更倾向于使用 Executor 的原因

有时候，你并不想对外暴露并发类的实现，此时，你可以使用内部类的方式。以下是几种典型应用方式

通过显示的内部类实现

* InnerThread1 包含内部类 Inner，内部类继承 Thread
* Inner 的构造函数会调用 start 方法启动线程
* InnerThread1 有构造函数，调用 Inner 的构造函数

```java
// Using a named inner class
class InnerThread1 {
    private int countDown = 5;
    private Inner inner;

    private class Inner extends Thread {
        Inner(String name) {
            super(name);
            start();
        }

        @Override
        public void run() {
            try {
                while (true) {
                    System.out.println(this);
                    if (--countDown == 0) return;
                    sleep(10);
                }
            } catch (InterruptedException e) {
                System.out.println("Interrupted");
            }
        }

        public String toString() {
            return getName() + ": " + countDown;
        }
    }

    public InnerThread1(String name) {
        inner = new Inner(name);
    }
}
```

通过显示的内部类实现, 和上面的例子大同小异，只是把  Thread 类的声明塞到了 InnerThread2 构造函数中，然后直接调用 start() 开启线程

```java
// Using an anonymous inner class
class InnerThread2 {
    private int countDown = 5;
    private Thread t;

    public InnerThread2(String name) {
        t = new Thread(name) {
            @Override
            public void run() {
                try {
                    while (true) {
                        System.out.println(this);
                        if (--countDown == 0) return;
                        sleep(10);
                    }
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }

            @Override
            public String toString() {
                return getName() + ": " + countDown;
            }
        };
        t.start();
    }
}
```

内部类 + Runnable, 和第一个例子类似，只不过通过 Runnable 接口做实现，start 的调用直接放在 Inner 的构造函数中


```java
// Using a named Runnable implementation
class InnerRunnable1 {
    private int countDown = 5;
    private Inner inner;

    private class Inner implements Runnable {
        Thread t;

        public Inner(String name) {
            t = new Thread(this, name);
            t.start();
        }

        @Override
        public void run() {
            try {
                while (true) {
                    System.out.println(this);
                    if (--countDown == 0) return;
                    TimeUnit.MILLISECONDS.sleep(10);
                }
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }

        @Override
        public String toString() {
            return t.getName() + ": " + countDown;
        }
    }

    public InnerRunnable1(String name) {
        inner = new Inner(name);
    }
}
```

内部类 + Runnable, Runnable 实现放在构造函数中

```java
// Using an anonymous Runnable implementation
class InnerRunnable2 {
    private int countDown = 5;
    private Thread t;

    public InnerRunnable2(String name) {
        t = new Thread(new Runnable() {
            @Override
            public void run() {
                try {
                    while (true) {
                        System.out.println(this);
                        if (--countDown == 0) return;
                        TimeUnit.MILLISECONDS.sleep(10);
                    }
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }

            @Override
            public String toString() {
                return Thread.currentThread().getName() + ": " + countDown;
            }
        }, name);
        t.start();
    }
}
```

将 Runnable 放到方法中做实现

```java
// A separate method tu run some code as a task
class ThreadMethod{
    private int countDown = 5;
    private Thread t;
    private String name;

    public ThreadMethod(String name) {
        this.name = name;
    }

    public void runTask() {
        if (t == null) {
            t = new Thread(name) {
                @Override
                public void run() {
                    try {
                        while(true) {
                            System.out.println(this);
                            if (--countDown == 0) return;
                            sleep(10);
                        }
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
            };
            t.start();
        }
    }
}
```

客户端调用

```java
public class ThreadVariations {
    public static void main(String[] args) {
        new InnerThread1("InnerThread1");
        new InnerThread2("InnerThread2");
        new InnerRunnable1("InnerRunnable1");
        new InnerRunnable2("InnerRunnable2");
        new ThreadMethod("ThreadMethod").runTask();
    }
}
```

### Terminology

术语解释，没发现什么有趣的点

### Joining a thread

在线程 A 的执行中，如果你 call 了线程 B 的 join() 方法，那么，线程 A 会等待线程 B 结束后再执行。

上面的 join 的行为可以通过调用 B 的 interrup() 方法进行打断

示例解析：

Sleeper 继承自 Thread 通过构造函数指定线程名称和休眠时间。当被打断时输出日志。

Joiner 继承自 Thread, 通过参数指定 Thread name 和将要 join 的 thread

主程序中，创建两个 Sleeper 类，再创建两个 Joiner 类并将 Sleeper 分别传给他们。

Joiner 执行的时候，会等待传入的 Sleeper 执行结束再继续执行。其中一个 Sleper 调用 interrupt() 方法，中途中断，对应的 Joiner 继续执行

PS: 当现场被打断时，isInterrupted 被设置为 true, 当异常被捕获时，重制为 false。

```java
public class joining {
    public static void main(String[] args) {
        Sleeper
                sleepy = new Sleeper("Sleepy", 1500),
                grumpy = new Sleeper("Grumpy", 1500);
        Joiner
                dopey = new Joiner("Dopey", sleepy),
                doc = new Joiner("Doc", grumpy);
        grumpy.interrupt();
    }
}

class Sleeper extends Thread {
    private final int duration;
    public  Sleeper (String name, int sleepTime) {
        super(name);
        duration = sleepTime;
        start();
    }

    @Override
    public void run() {
        try {
            sleep(duration);
        }catch (InterruptedException e) {
            System.out.println(getName() + " was interrupted. " + "isInterrupted(): " + isInterrupted());
            return;
        }
        System.out.println(getName() + " has awakened");
    }
}

class Joiner extends Thread {
    private final Sleeper sleeper;

    public Joiner(String name, Sleeper sleeper) {
        super(name);
        this.sleeper = sleeper;
        start();
    }

    @Override
    public void run() {
        try {
            sleeper.join();
        }catch(InterruptedException e) {
            System.out.println("Interrupted");
        }
        System.out.println(getName() + " join completed");
    }
}

// Grumpy was interrupted. isInterrupted(): false
// Doc join completed
// Sleepy has awakened
// Dopey join completed
```

### Creating responsive user interface

模拟图形界面，但是不太能 get 到他的点，pass

### Thread groups

Thread group 是一个失败的作品，你最好忘记他的存在

### Catching exceptions

由于 Thread 的特性，run() 中抛出的异常，你不能在 main 中 catch。示例如下，我们新建一个 Runnable 的实现类，并让跑抛异常

```java
public class ExceptionThread implements Runnable {
    @Override
    public void run() {
        throw new RuntimeException();
    }

    public static void main(String[] args) {
        ExecutorService exec = Executors.newCachedThreadPool();
        exec.execute(new ExceptionThread());
    }
}
// Exception in thread "pool-1-thread-1" java.lang.RuntimeException
// 	at org.jz.c23.ExceptionThread.run(ExceptionThread.java:9)
// 	at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1149)
// 	at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:624)
// 	at java.lang.Thread.run(Thread.java:836)
```

我们在 executor 外面添加 try-catch 试图捕获异常

```java
public class NaiveExceptionHandling {
    public static void main(String[] args) {
        try {
            ExecutorService exec = Executors.newCachedThreadPool();
            exec.execute(new ExceptionThread());
        }catch (RuntimeException e) {
            System.out.println("Exception has been handled...");
        }
    }
}
// Exception in thread "pool-1-thread-1" java.lang.RuntimeException
// 	at org.jz.c23.ExceptionThread.run(ExceptionThread.java:9)
// 	at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1149)
// 	at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:624)
// 	at java.lang.Thread.run(Thread.java:836)
```

然并卵。。。。这时，你可以结合 Executor 使用它，在 new pool 的时候指定 factory， 并在 factory 的实现中指定异常的处理器

```java
public class CaptureUncaughtException {
    public static void main(String[] args) {
        ExecutorService exec = Executors.newCachedThreadPool(new HandlerThreadFactory());
        exec.execute(new ExceptionThread2());
    }
}

class ExceptionThread2 implements Runnable {
    @Override
    public void run() {
        Thread t = Thread.currentThread();
        System.out.println("run() by " + t);
        System.out.println("eh = " + t.getUncaughtExceptionHandler());
        throw new RuntimeException();
    }
}

class MyUncaughtExceptionHandler implements Thread.UncaughtExceptionHandler {
    @Override
    public void uncaughtException(Thread t, Throwable e) {
        System.out.println("caught " + e);
    }
}

class HandlerThreadFactory implements ThreadFactory {
    @Override
    public Thread newThread(Runnable r) {
        System.out.println(this + " creating new thread");
        Thread t = new Thread(r);
        System.out.println("create " + t);
        t.setUncaughtExceptionHandler(new MyUncaughtExceptionHandler());
        System.out.println("eh = " + t.getUncaughtExceptionHandler());
        return t;
    }
}
// org.jz.c23.HandlerThreadFactory@39a054a5 creating new thread
// create Thread[Thread-0,5,main]
// eh = org.jz.c23.MyUncaughtExceptionHandler@6ed3ef1
// run() by Thread[Thread-0,5,main]
// eh = org.jz.c23.MyUncaughtExceptionHandler@6ed3ef1
// org.jz.c23.HandlerThreadFactory@39a054a5 creating new thread
// create Thread[Thread-1,5,main]
// eh = org.jz.c23.MyUncaughtExceptionHandler@78463d45
// caught java.lang.RuntimeException
```

PS: 不知道为毛会有两次创建 handler 的动作，好奇怪

如果所有的异常都是一个 handler 处理的，还可以直接将 handler 设置给 Thread 简化操作

```java
public class SettingDefaultHandler {
    public static void main(String[] args) {
        Thread.setDefaultUncaughtExceptionHandler(new MyUncaughtExceptionHandler());
        ExecutorService exec = Executors.newCachedThreadPool();
        exec.execute(new ExceptionThread());
    }
}
// caught java.lang.RuntimeException
```