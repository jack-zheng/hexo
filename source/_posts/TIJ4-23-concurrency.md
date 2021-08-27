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

中间中间好几节暂时不用，先放一放

## Sharing resources

并发需要解决的是多个线程操作共用资源的问题

### Improperly accessing resources

举一个多线程使用 int 生成器的例子。我们创建一个生成器的抽象接口,定义了抽象类中的方法。

* next() - 生成 int 结果
* cancel() - 设置 flag
* isCancel() - 返回 flag 结果

canceled 这个 flag 还被定义为 volatile 确保其他线程可见

cancel() 为 boolean 赋值语句，是一个原子操作

```java
public abstract class IntGenerator {
    private volatile boolean canceled = false;
    public abstract int next();
    public void cancel() { canceled = true; }
    public boolean isCanceled() { return canceled; }
}
```

定义 EvenChecker 并发检测奇偶情况.

* run() - 拿到生成的 int 值并判断，如果为奇数则停止线程
* test() - 重载了两个 test 方法，启动多个线程运行 run 方法

```java
public class EvenChecker implements Runnable {
    private IntGenerator generator;
    private final int id;

    public EvenChecker(IntGenerator g, int ident) {
        generator = g;
        id = ident;
    }

    @Override
    public void run() {
        while (!generator.isCanceled()) {
            int val = generator.next();
            if (val % 2 != 0) {
                System.out.println(val + " not event...");
                generator.cancel();
            }
        }
    }

    public static void test(IntGenerator gp, int count) {
        System.out.println("Ctrl + c to exit");
        ExecutorService exec = Executors.newCachedThreadPool();
        for (int i = 0; i < count; i++) {
            exec.execute(new EvenChecker(gp, i));
        }
        exec.shutdown();
    }

    public static void test(IntGenerator gp) {
        test(gp, 10);
    }
}
```

生成器的实现类 EvenGenerator, 声明一个初始值，并通过两个 ++ 运算，达到偶数次递增的目的

```java
public class EvenGenerator extends IntGenerator {
    private int currentEvenValue = 0;

    @Override
    public int next() {
        ++currentEvenValue;
        ++currentEvenValue;
        return currentEvenValue;
    }

    public static void main(String[] args) {
        EvenChecker.test(new EvenGenerator());
    }
// }
// Ctrl + c to exit
// 1515 not event...
// 1519 not event...
// 1517 not event...
```

运行后可以看到，三个线程检测到目标为奇数，停止了 generator。当执行 next() 时，有可能执行了一半，切到另一个线程执行 run() 中的判断了。这时，程序状态就会出错。你还可以在两个 ++ 操作中间通过新加 yield() 方法来加大重现频率。

还有一个需要注意的是 i++ 并不是一个原子操作，可能在执行间就切到另一个线程了。

### Resolving shared resource contention

这里举了一个挺有意思的例子，处理并发就像是 你坐在餐桌上，准备夹一块肉的时候，突然，肉没了(你的线程被暂停，同时其他线程操作了这个资源)

你可以通过加锁防止这种事情的发生，这样保证一个资源同一时间只能由一个 task 访问，其他 task 需要排队等待解锁。

代码层面，Java 通过 synchronized 关键字来实现这一功能。shared resource 同行来说是一片系统内存，以对象的形式表现出来。也可能是一个文件，或者 IO 端口或者一些设备，比如打印机之类的。

你需要将 class 中代表你要 lock 的对象声明为 private，并且所有和这个对象相关的方法前加关键字，示例如下

```java
synchronized void f() {}
synchronized void g() {}
```

当一个方法被调用的时候，其他加了关键字的方法都会被 lock 住，直到前一个方法执行完毕为止。

PS: 将用到的对象声明为 private 是很关键的一步，否则控制并发会失败。

上述对象在 JVM 有一个 field 来记录锁的数量，默认为 0，当 synchronized 方法被调用时，count + 1，当对应的 task 调用这个对象的两一个 synchronized 方法时，再 + 1.

此外还有 class level 的 lock 用来控制 static 方法的同步，保证同一时间只有一个 task 访问这个静态方法。

加锁的原则：This is an important point: Every method that accesses a critical shared resource must be synchronized or it won’t work right.

### Synchronizing the EvenGenerator

根据上一节的描述，我们通过给 next 方法加锁，将之前的 EvenGenerator 改为线程安全版本.

```java
public class SynchronizedEvenGenerator extends IntGenerator {
    private int currentEvenValue = 0;

    @Override
    public synchronized int next() {
        ++currentEvenValue;
        Thread.yield();
        ++currentEvenValue;
        return currentEvenValue;
    }

    public static void main(String[] args) {
        EvenChecker.test(new SynchronizedEvenGenerator());
    }
}
```

### Using explicit Lock objects

Java 5 的 concurrent 包中提供了一个 Lock 类来更精确的控制锁的范围，示例如下

```java
public class MutexEvenGenerator extends IntGenerator {
    private int currentEventValue = 0;
    private Lock lock = new ReentrantLock();

    @Override
    public int next() {
        lock.lock();
        try {
            ++currentEventValue;
            Thread.yield();
            ++currentEventValue;
            return currentEventValue;
        } finally {
            lock.unlock();
        }
    }

    public static void main(String[] args) {
        EvenChecker.test(new MutexEvenGenerator());
    }
}
```

用上面这种写法，你需要注意， 调用 lock() 方法之后，一定要用 try-finally 的语法，将 unlock 放到 finally 中，并切 try block 里面完成 return 的动作，防止指在外面被改动。

相比于传统的 synchronized 方式，try-finally 代码更多，但是它给你机会再程序出错时做出补救。

通过使用 concurrent 包下的方法，你可以实现 re-try 的机制

下面的例子中定义了两个方法，untimed/timed 功能都是一样的，尝试获取锁，并打印获取的情况。main 中一开始，顺序执行，两个方法可以拿到锁，并在使用完后释放。后面通过匿名类，启动一个新线程，获取锁并不释放，后面再次调用之前的方法，返回获取锁失败。

```java
public class AttemptLocking {
    private ReentrantLock lock = new ReentrantLock();
    public void untimed() {
        boolean captured = lock.tryLock();
        try {
            System.out.println("tryLock(): " + captured);
        } finally {
            if(captured)
                lock.unlock();
        }
    }

    public void timed() {
        boolean captured = false;
        try {
            captured = lock.tryLock(2, TimeUnit.SECONDS);
        } catch (InterruptedException e) {
            throw new RuntimeException(e);
        }
        try {
            System.out.println("tryLock(2, TimeUnit.SECONDS): " + captured);
        }finally {
            if (captured)
                lock.unlock();
        }
    }

    public static void main(String[] args) throws InterruptedException {
        final AttemptLocking al = new AttemptLocking();
        al.untimed();
        al.timed();
        new Thread() {
            {setDaemon(true);}

            @Override
            public void run() {
                al.lock.lock();
                System.out.println("acquired");
            }
        }.start();
        Thread.sleep(1000);
        al.untimed();
        al.timed();
    }
}
// tryLock(): true
// tryLock(2, TimeUnit.SECONDS): true
// tryLock(): true
// acquired
// tryLock(2, TimeUnit.SECONDS): false
```

### Atomicity and volatility

一个原子操作是指，如果这个操作开始了，那么只有在操作结束后，JVM 才会考虑进行上下文切换。考虑到原子操作这么冷门而且很危险，建议专家级别了再作原子操作代替 synchronized 的优化。

> The Goetz Test: If you can write a hight-perormance JVM for a modern microprocessor, then you are qualified to think about whether you can avoid synchronizing.

鼓励使用官方为你写的工具包(concurrent)，而不是自己造的轮子。

'simple operation' 和 除了 long/double 的 primitive 类型的数据操作都是原子操作。JVM 在处理 long/double 时会分成两个指令处理，但是如果你为这两类数据加上 volatile 修饰之后，可以保证原子性。

在多核处理器系统中，visibility 是比 atomicity 更容易出问题的点。一个 task 进行的一个原子操作，由于改动存放在本地处理器的 cache 中，导致其他 task 不知道这个改动，从而导致不同 task 之间 application 的状态不一致。synchronization 机制可以保证一个 task 的改动在其他 task 上也可以被观察到。

volatile 也可以保证这一点。如果你声明了一个 volatile 变量，一旦写操作执行了，那么所有要读他的地方会立即观察到这个变化，其实是 local cache 的情况也能保证。volatile 会保证 write 的动作立即反应到驻内存中。

atomicity 和 volatility 是两个概念，一个非 volatile 的原子操作并不会被 flush 到主内存中，如果多个 task 对他进行操作，会产生不一致。如果多个 task 都要访问一个 field，那么他就需要声明为 volatile 类型，或者用 synchronization 来管理他。如果用了 synchronizaiton 管理，就不需要用 volatile 修饰了

优先考虑用 synchronization，这个是最安全的解决方案。

Java 中赋值和返回语句是原子操作，自增/减不是。。。Java 反编译自增代码如下

```java
//: concurrency/Atomicity.java
// {Exec: javap -c Atomicity}
public class Atomicity {
    int i;
    void f1() { i++; }
    void f2() { i += 3; } 
} 
/* Output: (Sample) 
... void f1();
   Code:
    0:        aload_0    
    1:        dup    
    2:        getfield        #2; //Field i:I    
    5:        iconst_1    
    6:        iadd    
    7:        putfield        #2; //Field i:I    
    10:        return  
void f2();
   Code:    
   0:        aload_0    
   1:        dup    
   2:        getfield        #2; //Field i:I    
   5:        iconst_3    
   6:        iadd    
   7:        putfield        #2; //Field i:I    
   10:        return
```

可以看到在 get 和 put 指令中间，还会执行一些其他的指令。上下文切换有可能发生在执行这些指令的时候，所以并不是原子行的。

即使是 getValue() 这样的操作，虽然说他是原子操作，但是如果不加 synchronized 也可能会出问题. 下面的程序中，我们实现了一个自增函数 evenIncrement 并用 synchronized 修饰，在 run 中让他一直运行。在 main 中启动这个线程，并打印当前 i 的值。可以看到还是会出问题。问题有两个

* 变量没有用 volatile 修饰
* getValue 没有用 synchronized 修饰

试了一下，即使 i 用 volatile 修饰了还是会出问题的

没有 synchronized 修饰时，程序允许 getValue 在状态不确定的情况下访问变量，所以会出问题。

```java
public class AtomicityTest implements Runnable{
    private int i = 0;
    public int getValue() { return i; }
    private synchronized void evenIncrement() { i++; i++; }

    @Override
    public void run() {
        while(true)
            evenIncrement();
    }

    public static void main(String[] args) {
        ExecutorService exec = Executors.newCachedThreadPool();
        AtomicityTest at = new AtomicityTest();
        exec.execute(at);

        while(true) {
            int val = at.getValue();
            if(val %2 !=0) {
                System.out.println(val);
                System.exit(0);
            }
        }
    }
}
// 423
```

下面是一个更简单的例子，一个生成器给出一系列的数字

```java
public class SerialNumberGenerator {
    private static volatile int serialNumber = 0;
    public static int nextSerialNumber() {
        return serialNumber++;  // Not thread-safe
    }
}
```

将变量声明为 volatile 可以告诉编译器不要对它进行优化。读写会被直接反应到内存中，而不是 cache. 而且还能防止指令重排。但是它并不表示这个自增是一个原子操作。

通常来说，如果有多个 task 操作一个 field，并且至少有一个会对他进行写操作，那么你就要讲他声明为 volatile。比如 flag 的操作。

为了测试上面的这个类，我们创建了如下的测试类

CircularSet 是一个容器类，用来存储生成器产生的数据。定义了一个数组，可以指定大小。 add/contains 使用 synchronized 修饰。主线程中，启动 10 个线程生产数据并存到容器中。理论上来说，如果线程安全，容器中不会有重复数据，如果有，则报错，结束进程。

这里一个比较巧妙的设置是，CircularSet 会存储一个下标，如果生产的值超出容量，他会循环利用之前的位置，覆盖之前的值。

可以在 SerialNumberGenerator 的 nextSerialNumber 方法前添加 synchronized 修饰修复这个问题。

```java
public class SerialNumberChecker {
    private static final int SIZE = 10;
    private static CircularSet serials = new CircularSet(1000);
    private static ExecutorService exec = Executors.newCachedThreadPool();

    static class SerialChecker implements Runnable {
        @Override
        public void run() {
            while(true) {
                int serial = SerialNumberGenerator.nextSerialNumber();
                if (serials.contains(serial)) {
                    System.out.println("Duplicate: " + serial);
                    System.exit(0);
                }
                serials.add(serial);
            }
        }
    }

    public static void main(String[] args) throws InterruptedException {
        for (int i = 0; i < SIZE; i++) {
            exec.execute(new SerialChecker());
        }

        if (args.length > 0) {
            TimeUnit.SECONDS.sleep(new Integer(args[0]));
            System.out.println("No duplicates detected");
            System.exit(0);
        }
    }
}

class CircularSet {
    private int[] array;
    private int len;
    private int index = 0;
    public CircularSet(int size) {
        array = new int[size];
        len = size;
        // Initialize to a value not produced by the SerialNumberChecker
        for (int i = 0; i < size; i++) {
            array[i] = -1;
        }
    }

    public synchronized void add(int i) {
        array[index] = i;
        // Wrap index and write over old elements
        index = ++index % len;
    }

    public synchronized boolean contains(int val) {
        for (int i = 0; i < len; i++) {
            if (array[i] == val) return true;
        }
        return false;
    }
}
// Duplicate: 47
```

### Atomic classes

Java 5 引入了原子类，如 AtomicInteger, AtomicLong 和 AtomicReference 等提供原子级别的更新操作。

> boolean compareAndSet(expectedValue, updateValue);

他们做过优化，可以保证机器层面的原子性，一般来说，你可以放心使用。下面我们用他们来重写之前的测试类 AtomicityTest.java. 内部逻辑和之前一样，我们将 volatile 和 synchronized 关键字都去了，同时定义一个 main 函数，在里面使用 Timer 设置程序 5s 之后退出，到程序结束为止一切运行正常。

```java
public class AtomicIntegerTest implements Runnable {
    private AtomicInteger i = new AtomicInteger(0);

    public int getValue() {
        return i.get();
    }

    private void evenIncrement() {
        i.addAndGet(2);
    }

    @Override
    public void run() {
        while (true)
            evenIncrement();
    }

    public static void main(String[] args) {
        new Timer().schedule(new TimerTask() {
            @Override
            public void run() {
                System.out.println("Aborting");
                System.exit(0);
            }
        }, 5000);
        ExecutorService exec = Executors.newCachedThreadPool();
        AtomicIntegerTest ait = new AtomicIntegerTest();
        exec.execute(ait);
        while (true) {
            int val = ait.getValue();
            if (val % 2 != 0) {
                System.out.println(val);
                System.exit(0);
            }
        }
    }
}
```

同样的，我们使用 AtomicInteger 重写 EvenGenerator

```java
public class AtomicEvenGenerator extends IntGenerator {
    private AtomicInteger currentEvenValue = new AtomicInteger(0);

    @Override
    public int next() {
        return currentEvenValue.addAndGet(2);
    }

    public static void main(String[] args) {
        EvenChecker.test(new AtomicEvenGenerator());
    }
}
```

虽然 Atomic class 可以解决原子行问题，但是还是强烈推荐使用锁机制。

### Critial sections

我们可以通过 critical sections 的方式，只对一段代码进行保护而不是整个方法

```java
synchronized(syncObject) {
    // This code can be accessed
    // by only one task at a time
}
```

这种方式也叫做 synchronized block. 如果此时 syncObject 被其他 task lock 了，那么当前 task 会一直等待，直到 lock 被释放。以下示例对两种 lock 方式进行性能比较

Pair 是我们要操作的对象，线程不安全，这个模型就是内部存两个 int 变量，我们的目标是保证这两个变量想等。还有一个 checkState 方法，如果两个变量值不同，则抛异常。

PairManager 是一个抽象类，里面声明了一个用来存储 pair 的 list, 使用 Collections.synchronizedList() 得到，所以线程安全。 getPair() 也用 synchronized 修饰，线程安全。 store 虽然没有修饰但是只在实现类中调用，调用的时候会加 synchronized 限制。

PairManager1， PairManager2 都是 PairManager 的实现，区别是一个用了 method level 的 lock， 一个用了 block level 的 lock

PairManipulator 代表使用 PairManager 的 task, 我们通过它来启动多线程实现 PM 的 increment 调用

PairChecker 也是一个多线程的 task 它用来检测 PM 的状态并记录检测次数

CriticalSection 相当于 client，将上面说的这些元素整合并调用

```java
public class CriticalSection {
    // Test the two different approaches:
    static void testApproaches(PairManager pman1, PairManager pman2) {
        ExecutorService exec = Executors.newCachedThreadPool();
        PairManipulator
                pm1 = new PairManipulator(pman1),
                pm2 = new PairManipulator(pman2);
        PairChecker
                pcheck1 = new PairChecker(pman1),
                pcheck2 = new PairChecker(pman2);
        exec.execute(pm1);
        exec.execute(pm2);
        exec.execute(pcheck1);
        exec.execute(pcheck2);
        try {
            TimeUnit.MILLISECONDS.sleep(500);
        } catch (InterruptedException e) {
            System.out.println("Sleep interrupted");
        }
        System.out.println("pm1: " + pm1 + "\npm2: " + pm2);
        System.exit(0);
    }

    public static void main(String[] args) {
        PairManager
                pman1 = new PairManager1(),
                pman2 = new PairManager2();
        testApproaches(pman1, pman2);
    }
}

class Pair {
    private int x, y;

    public Pair(int x, int y) {
        this.x = x;
        this.y = y;
    }

    public Pair() {
        this(0, 0);
    }

    public int getX() {
        return x;
    }

    public int getY() {
        return y;
    }

    public void incrementX() {
        x++;
    }

    public void incrementY() {
        y++;
    }

    @Override
    public String toString() {
        return "x: " + x + ", y: " + y;
    }

    public class PairValuesNotEqualException extends RuntimeException {
        public PairValuesNotEqualException() {
            super("Pair values not equal: " + Pair.this);
        }
    }

    public void checkState() {
        if (x != y)
            throw new PairValuesNotEqualException();
    }
}

abstract class PairManager {
    AtomicInteger checkCounter = new AtomicInteger(0);
    protected Pair p = new Pair();
    private List<Pair> storage = Collections.synchronizedList(new ArrayList<>());

    public synchronized Pair getPair() {
        // Make a copy to keep the original safe:
        return new Pair(p.getX(), p.getY());
    }

    // Assme this is a time consuming peration
    protected void store(Pair p) {
        storage.add(p);
        try {
            TimeUnit.MILLISECONDS.sleep(50);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }

    public abstract void increment();
}

class PairManager1 extends PairManager {
    @Override
    public synchronized void increment() {
        p.incrementX();
        p.incrementY();
        store(getPair());
    }
}

// Use a critical section
class PairManager2 extends PairManager {
    @Override
    public void increment() {
        Pair temp;
        synchronized (this) {
            p.incrementX();
            p.incrementY();
            temp = getPair();
        }
        store(temp);
    }
}

class PairManipulator implements Runnable {
    private PairManager pm;

    public PairManipulator(PairManager pm) {
        this.pm = pm;
    }

    @Override
    public void run() {
        while (true)
            pm.increment();
    }

    @Override
    public String toString() {
        return "Pair: " + pm.getPair() + " checkCounter = " + pm.checkCounter.get();
    }
}

class PairChecker implements Runnable {
    private PairManager pm;

    public PairChecker(PairManager pm) {
        this.pm = pm;
    }

    @Override
    public void run() {
        while (true) {
            pm.checkCounter.incrementAndGet();
            pm.getPair().checkState();
        }
    }
}

// pm1: Pair: x: 42, y: 42 checkCounter = 2
// pm2: Pair: x: 42, y: 42 checkCounter = 2133930
```

PS: Note that the synchronized keyword is not part of the method signature and thus may be added during overriding. 

synchronized 不是方法签名的一部分！！

从输出的实验结果可以看到方法锁的可使用率要比 block 锁低很多，完全是碾压级别的差距。block 类型的锁可以提供更多的 unlock time.

下面通过使用 Lock 类来进行精确锁, 共能和之前的例子类似，只不过新实现了两个 PairManager 类，一个还是用方法级别的锁， ExplicitPairManager1 由于已经加了 synchronized 的了，里面的 lock 其实没什么用，去掉也不影响结果

ExplicitPairManager1 中直接使用了 Lock 类进行 block level 的锁。运行结果失败，会抛异常

```java
public class ExplicitCriticalSection {
    public static void main(String[] args) {
        PairManager
                pman1 = new ExplicitPairManager1(),
                pman2 = new ExplicitPairManager2();
        CriticalSection.testApproaches(pman1, pman2);
    }
}

class ExplicitPairManager1 extends PairManager {
    private Lock lock = new ReentrantLock();

    @Override
    public synchronized void increment() {
        lock.lock();
        try {
            p.incrementX();
            p.incrementY();
            store(getPair());
        } finally {
            lock.unlock();
        }
    }
}

class ExplicitPairManager2 extends PairManager {
    private Lock lock = new ReentrantLock();
    public void increment() {
        Pair temp;
        lock.lock();
        try {
            p.incrementX();
            p.incrementY();
            temp = getPair();
        } finally {
            lock.unlock();
        }
        store(temp);
    }
    //
    // @Override
    // public Pair getPair() {
    //     lock.lock();
    //     try {
    //         return new Pair(p.getX(), p.getY());
    //     } finally {
    //         lock.unlock();
    //     }
    // }
}
// Exception in thread "pool-1-thread-4" org.jz.c23.Pair$PairValuesNotEqualException: Pair values not equal: x: 2, y: 1
// 	at org.jz.c23.Pair.checkState(CriticalSection.java:83)
// 	at org.jz.c23.PairChecker.run(CriticalSection.java:163)
// pm1: Pair: x: 127, y: 127 checkCounter = 3
// pm2: Pair: x: 127, y: 127 checkCounter = 1816160
```

搜索了一下，发先这个[博客](https://blog.nex3z.com/2016/07/03/%E6%B7%B7%E7%94%A8%E5%90%8C%E6%AD%A5%E5%9D%97%E5%92%8C%E5%90%8C%E6%AD%A5%E6%96%B9%E6%B3%95%E6%97%B6%E7%9A%84%E9%97%AE%E9%A2%98/)说的挺好. 总结一下就是 getPair 用的 synchronized 语法，而 increment 用的 Lock 类的方法，两个都是锁，但是拿到的锁是不一样的，将 getPair 重写一下，也用同样的 Lock 类提供的锁即可修复。

### Synchronizing on other objects

synchronized block 的写法中，需要给出一个 lock 的对象，一般来说我们都会使用 this 作为参数，表示持有这个方法的对象就是我们要 lock 的对象。当然你也可以指定另一个对象，但是你一定要理清楚自己的业务逻辑，知道你要 lock 的对象是哪一个

下面例子中声明了一个 DualSynch 类，里面有两个方法，f(), g() 分别打印 5 次，f() 是 method lock，就是锁住自己的意思，g() 在内部指定锁住一个内部成员变量。在主函数中，我们通过新起线程调用 f()，在主函数中调用 g()。可以看到虽然外部 class 实体也有 lock 但是和内部的变量是不冲突的，两个 task 可以一起执行

```java
public class SyncObject {
    public static void main(String[] args) {
        final DualSynch ds = new DualSynch();
        new Thread(ds::f).start();
        ds.g();
    }
}

class DualSynch {
    private Object syncObject = new Object();
    public synchronized void f() {
        for (int i = 0; i < 5; i++) {
            System.out.println("f()");
            Thread.yield();
        }
    }

    public void g() {
        synchronized (syncObject) {
            for (int i = 0; i < 5; i++) {
                System.out.println("g()");
                Thread.yield();
            }
        }
    }
}
// g()
// f()
// f()
// g()
// f()
// f()
// f()
// g()
// g()
// g()
```

### Thread local storage

Java 还提供了另一种解决多线程使用共享资源时的冲突问题，叫做 ThreadLocal.

对与被管理的变量，Thread local storage 会在不同的 thread 总为变量创建单独的副本，所以各个 thread 彼此不会被影响到。

下面的例子中 Accessor 是一个具体的 task，他会通过 while 循环不停的调用 holder 的 increment 方法并答应对应的值。

ThreadLocalVariableHolder 中持有一个 Integer 类型的 ThreadLocal 变量，提供自增长方法，在 main 函数中，启动五个线程，调用自增方法并打印。

可以看到每个线程中拿到的 integer 值都是不一样的，而且相互不影响。

```java
public class ThreadLocalVariableHolder {
    private static ThreadLocal<Integer> value = new ThreadLocal<Integer>() {
        private Random rand = new Random(47);

        protected synchronized Integer initialValue() {
            return rand.nextInt(10000);
        }
    };

    public static void increment() {
        value.set(value.get() + 1);
    }

    public static int get() {
        return value.get();
    }

    public static void main(String[] args) throws InterruptedException {
        ExecutorService exec = Executors.newCachedThreadPool();
        for (int i = 0; i < 5; i++) {
            exec.execute(new Accessor(i));
        }
        TimeUnit.SECONDS.sleep(3);
        exec.shutdown();
    }
}

class Accessor implements Runnable {
    private final int id;

    public Accessor(int idn) {
        id = idn;
    }

    @Override
    public void run() {
        while (!Thread.currentThread().isInterrupted()) {
            ThreadLocalVariableHolder.increment();
            System.out.println(this);
            Thread.yield();
        }
    }

    @Override
    public String toString() {
        return "#" + id + ": " + ThreadLocalVariableHolder.get();
    }
}
// #0: 9259
// #1: 556
// #2: 6694
// #0: 9260
// #2: 6695
// #1: 557
// ...
```

## Terminating tasks

本章介绍如何外部结束 task

### The ornamental garden

下面的例子模拟一个植物园的场景，植物园入口处有闸机，通过统计闸机记述统计园内总人数。只做演示用，没有其他深意。

Count 用来管理总人数，提供 increment 和 value 方法，且都是 synchronized 修饰的。increment 中还包含一个 yield 方法用来提高多线程问题出发的概率。

Entrance 表示入口，他持有一个 Count 的静态变量，用来合计总人数。同时还申明了一个 number 的成员变量，用来审计从这个门进入的游客数量。声明 entrances 这个静态变量，用于线程结束后的统计，cancel 声明为 volatile 用来控制 task 的结束。

OrnamentalGarden 为 client 端，他的 main 函数会启动五个线程模拟入园操作。3s 后结束，分别答应 number 加和以及 count 值做统计，两个值应该是一样的。

```java
public class OrnamentalGarden {
    public static void main(String[] args) throws InterruptedException {
        ExecutorService exec = Executors.newCachedThreadPool();
        for (int i = 0; i < 5; i++) {
            exec.execute(new Entrance(i));
        }
        // Run for a while then stop and collect the data:
        TimeUnit.SECONDS.sleep(3);
        Entrance.cancel();
        exec.shutdown();
        if (!exec.awaitTermination(250, TimeUnit.MILLISECONDS))
            System.out.println("Some task were not terminated!");
        System.out.println("Total: " + Entrance.getTotalCount());
        System.out.println("Sum of Entrances: " + Entrance.sumEntrances());
    }
}

class Count {
    private int count = 0;
    private Random rand = new Random(47);

    // Remove the synchronized keyword to see counting fail:
    public synchronized int increment() {
        int temp = count;
        if (rand.nextBoolean()) // Yield half the time
            Thread.yield();
        return (count = ++temp);
    }

    public synchronized int value() {
        return count;
    }
}

class Entrance implements Runnable {
    private static Count count = new Count();
    private static List<Entrance> entrances = new ArrayList<>();
    private int number = 0;
    // Doesn't need synchronization to read:
    private final int id;
    private static volatile boolean canceled = false;

    // Atomic operation on a volatile field:
    public static void cancel() {
        canceled = true;
    }

    public Entrance(int id) {
        this.id = id;
        // Keep this task in a list. Also prevents garbage collection of dead tasks:
        entrances.add(this);
    }

    @Override
    public void run() {
        while (!canceled) {
            synchronized (this) {
                ++number;
            }
            System.out.println(this + " Total: " + count.increment());
            try {
                TimeUnit.MILLISECONDS.sleep(100);
            } catch (InterruptedException e) {
                System.out.println("Sleep interrupted");
            }
        }
        System.out.println("Stopping " + this);
    }

    public synchronized int getValue() {
        return number;
    }

    @Override
    public String toString() {
        return "Entrance " + id + ": " + getValue();
    }

    public static int getTotalCount() {
        return count.value();
    }

    public static int sumEntrances() {
        int sum = 0;
        for (Entrance entrance : entrances) {
            sum += entrance.getValue();
        }
        return sum;
    }
}
// Entrance 0: 1 Total: 1
// Entrance 2: 1 Total: 3
// Entrance 1: 1 Total: 2
// Entrance 3: 1 Total: 4
// Entrance 4: 1 Total: 5
// ...
// Entrance 2: 30 Total: 150
// Entrance 1: 30 Total: 148
// Stopping Entrance 3: 30
// Stopping Entrance 4: 30
// Stopping Entrance 1: 30
// Stopping Entrance 2: 30
// Stopping Entrance 0: 30
// Total: 150
// Sum of Entrances: 150
```

### Terminating when blocked

#### Thread states

一个 Thread 可能处于四种状态中的任意一种

1. New: 这种状态很短暂，在创建线程的时候出现。系统为他配置所需要的资源，完成后，scheduler 会把它置于 runnable 或者 blocked
2. Runnable: 当 CPU 有空闲时就可以运行它
3. Blocked: 可以运行，但是被阻止了。CPU 会直接跳过它。
4. Dead: task 结束了，不会再被 schedule。从 run() 中返回，或者被 interrupted 时会处于这种状态。

#### Becoming blocked

一下情况会导致 task 进入 block 状态

* 调用 sleep() 方法
* 调用 wait() 方法，可以调用 notify()/notifyAll() 解除
* 等待 I/O 完成
* 调用其他被 lock 的方法时

#### Interruption

和你预期的一样，在 thread 中间打断它要比等到它出来，判断 cancel flag 结束要复杂的多，你打断 thread 的时候可能要处理很多 clean up 的操作。

你可以通过 Thread.interrupt() 方法打断线程，这个方法会将线程设置为 interrupted 状态，然后这个线程就会抛出 InterruptedException. 这个状态会在异常抛出或者调用 Thread.interrupted() 方法的时候置位。interrupted() 是另一种结束 run() 而不抛异常的方法。

为了调用 interrupt() 方法，你需要持有 Thread 对象。Java 提供的 concurrent 包让你避免直接使用 Thread，你可以用 Executor 来完成这类工作。shutdownNow() 会向它开启的所有线程发送 interrupt() 指令。如果你想单独控制某个 task 你可以使用 Executor 的 submit() 方法，它会返回 Feature 对象，你可以调用 feature.cancel(true) 来给对应的 task 传递 interrupt 指令。

下面是通过 feature.cancel() 来中断线程的测试, 定义了三种 block

SleepBlocked: 普通的 Runnable 实现，在 run 方法中，sleep 100s 作为 block

IOBlocked: 普通的 Runnable 实现，run 中读取输入流的内容

SynchronizedBlocked: f() 中无限循环调用 yield, 在构造函数中新起一个线程，调用 f(), 然后 main 中通过 test 起新线程制造 lock

Interrupting: 测试类， 写了一个 test 方法，接收 Runnable 实现，并通过 submit() 运行，然后通过 cancel(true) 中断 task. main 函数中将之前定义的 block 分别进行 test。

从输出可以看出，你可以 interrupt sleep 类型的 block，但是不能打断 IO 或者 Synchronized 类型的锁

```java
public class Interrupting {
    private static ExecutorService exec = Executors.newCachedThreadPool();
    static void test(Runnable r) throws InterruptedException {
        Future<?> f = exec.submit(r);
        TimeUnit.MILLISECONDS.sleep(100);
        System.out.println("Interrupting " + r.getClass().getName());
        f.cancel(true); // Interrupts if running
        System.out.println("Interrupt sent to " + r.getClass().getName());
    }

    public static void main(String[] args) throws InterruptedException {
        test(new SleepBlocked());
        test(new IOBlocked(System.in));
        test(new SynchronizedBlocked());
        TimeUnit.SECONDS.sleep(3);
        System.out.println("Aborting with System.exit(0)");
        System.exit(0);
    }
}

class SleepBlocked implements Runnable {
    @Override
    public void run() {
        try {
            TimeUnit.SECONDS.sleep(100);
        } catch (InterruptedException e) {
            System.out.println("InterruptedException");
        }
        System.out.println("Existing SleepBlocked.run()");
    }
}

class IOBlocked implements Runnable {
    private InputStream in;

    public IOBlocked(InputStream in) {
        this.in = in;
    }

    @Override
    public void run() {
        try {
            System.out.println("Waiting for read(): ");
            in.read();
        } catch (IOException e) {
            if (Thread.currentThread().isInterrupted()) {
                System.out.println("Interrupted from blocked I/O");
            } else {
                throw new RuntimeException(e);
            }
        }
        System.out.println("Exiting IOBlocked.run()");
    }
}

class SynchronizedBlocked implements Runnable {
    public synchronized void f() {
        while(true)
            Thread.yield();
    }

    public SynchronizedBlocked() {
        new Thread() {
            public void run() {
                f(); // Lock acquired by this thread
            }
        }.start();
    }

    public void run () {
        System.out.println("Trying to call f()");
        f();
        System.out.println("Exiting SynchronizedBlocked.run()");
    }
}

// Interrupting org.jz.c23.SleepBlocked
// Interrupt sent to org.jz.c23.SleepBlocked
// InterruptedException
// Existing SleepBlocked.run()
// Waiting for read(): 
// Interrupting org.jz.c23.IOBlocked
// Interrupt sent to org.jz.c23.IOBlocked
// Trying to call f()
// Interrupting org.jz.c23.SynchronizedBlocked
// Interrupt sent to org.jz.c23.SynchronizedBlocked
// Aborting with System.exit(0)
```

有时你可以通过关闭底层的 resource 来中断 IO, 从输出可以看到， Socket 的输入流是通过异常关闭的，而 System.in 不是。

```java
public class CloseResource {
    public static void main(String[] args) throws IOException, InterruptedException {
        ExecutorService exec = Executors.newCachedThreadPool();
        ServerSocket server  = new ServerSocket(8080);
        InputStream socketInput = new Socket("localhost", 8080).getInputStream();

        exec.execute(new IOBlocked(socketInput));
        exec.execute(new IOBlocked(System.in));

        TimeUnit.MILLISECONDS.sleep(100);
        System.out.println("Shutting down all threads");
        exec.shutdownNow();

        TimeUnit.SECONDS.sleep(1);
        System.out.println("Closing " + socketInput.getClass().getName());
        socketInput.close(); // Releases blocked thread

        TimeUnit.SECONDS.sleep(1);
        System.out.println("Closing " + System.in.getClass().getName());
        System.in.close();
    }
}

// Waiting for read(): 
// Waiting for read(): 
// Shutting down all threads
// Closing java.net.SocketInputStream
// Interrupted from blocked I/O
// Exiting IOBlocked.run()
// Closing java.io.BufferedInputStream
// Exiting IOBlocked.run()
```

好消息是 nio 相关的类有提供更好的终端 IO 的方法. blocked nio channels 会自动相应 interrupt 信号。

从输出可以看到，关闭底层的 channel 会释放 block，虽然这种方式和少用到。

```java
public class NIOInterruption {
    public static void main(String[] args) throws IOException, InterruptedException {
        ExecutorService exec = Executors.newCachedThreadPool();
        ServerSocket server = new ServerSocket(8080);
        InetSocketAddress isa = new InetSocketAddress("localhost", 8080);
        SocketChannel sc1 = SocketChannel.open(isa);
        SocketChannel sc2 = SocketChannel.open(isa);
        Future<?> f = exec.submit(new NIOBlocked(sc1));
        exec.execute(new NIOBlocked(sc2));
        exec.shutdown();

        TimeUnit.SECONDS.sleep(1);
        // Produce an interrupt via cancel;
        f.cancel(true);

        TimeUnit.SECONDS.sleep(1);
        // Release the block by closing the channel
        sc2.close();
    }
}

class NIOBlocked implements Runnable {
    private final SocketChannel sc;

    public NIOBlocked(SocketChannel sc) {
        this.sc = sc;
    }

    @Override
    public void run() {
        try {
            System.out.println("Waiting for read() in " + this);
            sc.read(ByteBuffer.allocate(1));
        } catch (ClosedByInterruptException e) {
            System.out.println("ClosedByInterruptException");
        } catch (AsynchronousCloseException e) {
            System.out.println("AsynchronousCloseException");
        } catch (IOException e) {
            throw new RuntimeException(e);
        }
        System.out.println("Exiting NIOBlocked.run() " + this);
    }
}

// Waiting for read() in org.jz.c23.NIOBlocked@2bc68519
// Waiting for read() in org.jz.c23.NIOBlocked@50b0c91f
// ClosedByInterruptException
// Exiting NIOBlocked.run() org.jz.c23.NIOBlocked@2bc68519
// AsynchronousCloseException
// Exiting NIOBlocked.run() org.jz.c23.NIOBlocked@50b0c91f
```

#### Blocked by a mutex

从 Interrupting.java 的例子可以看到，如果我们调用一个对象的 synchronized 方法，如果该方法的 lock 已经被获取了，那么这个 task 会 block 并等到 lock 被释放后再调用。下面的例子展示了同一个 task 如何多次获取同一个对象的锁

MultiLock 有两个方法 f, g 分别调用对方，并用 synchronized 关键字修饰。每次方法被调用时，方法的实例都会被 lock 一次。

举一个

```java
public class MultiLock {
    public synchronized void f1(int count) {
        if (count-- > 0) {
            System.out.println("f1() calling f2() with count " + count);
            f2(count);
        }
    }

    public synchronized void f2(int count) {
        if (count-- > 0) {
            System.out.println("f2() calling f1() with count " + count);
            f1(count);
        }
    }

    public static void main(String[] args) {
        final MultiLock multiLock = new MultiLock();
        new Thread(() -> multiLock.f1(10)).start();
    }
}
// f1() calling f2() with count 9
// f2() calling f1() with count 8
// f1() calling f2() with count 7
// f2() calling f1() with count 6
// f1() calling f2() with count 5
// f2() calling f1() with count 4
// f1() calling f2() with count 3
// f2() calling f1() with count 2
// f1() calling f2() with count 1
// f2() calling f1() with count 0
```

这个例子的说明不是很懂，但是大概就是 concurrency lib 的 RenntrantLocks 提供了一种打断机制

BlockedMutex 持有 ReentrantLock 变量，并在构造函数中 lock，提供 f() 方法，调用 lock.lockInterruptibly(); 由于构造中的 lock，这个方法会一直 block。

Blocked2 新建 BlockedMutex 对象，导致上锁。然后调用 f() 被 block。

Interrupting2 在启动 Blocked2 之后 1s 进行打断，interrupt 成功

```java
public class Interrupting2 {
    public static void main(String[] args) throws InterruptedException {
        Thread t = new Thread(new Blocked2());
        t.start();
        TimeUnit.SECONDS.sleep(1);
        System.out.println("Issuing t.interrupt()");
        t.interrupt();
    }
}

class BlockedMutex {
    private Lock lock = new ReentrantLock();

    public BlockedMutex() {
        // Acquire it right away, to demonstrate interruption of a task blocked on a ReentrantLock
        lock.lock();
    }

    public void f() {
        try {
            // This will never be available to a second task
            lock.lockInterruptibly();
            System.out.println("lock acquired in f()");
        } catch (InterruptedException e) {
            System.out.println("Interrupted from lock acquisition in f()");
        }
    }
}

class Blocked2 implements Runnable {
    BlockedMutex blocked = new BlockedMutex();

    @Override
    public void run() {
        System.out.println("Waiting for f() in BlockedMutex");
        blocked.f();
        System.out.println("Broken out of blocked call");
    }
}

// Waiting for f() in BlockedMutex
// Issuing t.interrupt()
// Interrupted from lock acquisition in f()
// Broken out of blocked call
```

### Checking for an interrupt

TBD

## Cooperation between tasks

通过之前的章节，我们知道可以通过互斥锁来控制多个 task 对一个资源的访问。

这章我们会学习如何通过内置方法，协调多个 task 之间对一个资源的调用。Object 提供了 wait()/notifyAll()，concurrent lib 提供了 await()/signal() 来完成这些功能。

### wait() and notifyAll()

下面将 wait/notifyAll 应用在汽车打蜡的场景

Car 代表将会被 lock 的类, 提供了四个方法，使用 synchronized 修饰，分别是打蜡，抛光，等待上蜡，等待抛光。

WaxOn 代表上蜡的 task, 接收一个 car 做参数，在 run 中，打印状态并将 car 的 flag 置位 true，等待抛光

WaxOff 代表抛光的 task, 接收一个 car 做参数，在 run 中，打印状态并将 car 的 flag 置位 false, 并等待打蜡

WaxOMatic 新建 car 对象，并启动两个 task 让他们轮流操作 car, 并在一定时间后结束操作

```java
public class WaxOMatic {
    public static void main(String[] args) throws InterruptedException {
        Car car = new Car();
        ExecutorService exec = Executors.newCachedThreadPool();
        exec.execute(new WaxOff(car));
        exec.execute(new WaxOn(car));
        TimeUnit.SECONDS.sleep(2);
        exec.shutdownNow();
    }
}

class Car {
    private boolean waxOn = false;

    public synchronized void waxed() {
        waxOn = true; // Ready to buff
        notifyAll();
    }

    public synchronized void buffed() {
        waxOn = false; // Ready for another coat of wax
        notifyAll();
    }

    public synchronized void waitForWaxing() throws InterruptedException {
        while (waxOn == false)
            wait();
    }

    public synchronized void waitForBuffing() throws InterruptedException {
        while (waxOn == true)
            wait();
    }
}

class WaxOn implements Runnable {
    private Car car;

    public WaxOn(Car c) {
        car = c;
    }

    @Override
    public void run() {
        try {
            while (!Thread.interrupted()) {
                System.out.println("Wax On!");
                TimeUnit.MILLISECONDS.sleep(200);
                car.waxed();
                car.waitForBuffing();
            }
        } catch (InterruptedException e) {
            System.out.println("Exiting via interrupt");
        }
        System.out.println("Ending Wax On task");
    }
}

class WaxOff implements Runnable {
    private Car car;

    public WaxOff(Car c) {
        car = c;
    }

    @Override
    public void run() {
        try {
            while (!Thread.interrupted()) {
                car.waitForWaxing();
                System.out.println("Wax Off!");
                TimeUnit.MILLISECONDS.sleep(200);
                car.buffed();
            }
        } catch (InterruptedException e) {
            System.out.println("Exiting via interrupt");
        }
        System.out.println("Ending Wax Off task");


    }
}
// Wax On!
// Wax Off!
// Wax On!
// Wax Off!
// Wax On!
// Wax Off!
// Wax On!
// Wax Off!
// Wax On!
// Wax Off!
// Exiting via interrupt
// Ending Wax Off task
// Exiting via interrupt
// Ending Wax On task
```

上面的例子中需要强调的一点是，你必须将 wait() 用 while 包裹起来，因为

* 多个 task 等待同一个 lock 时，前面的 task 可能会改变某些条件，当前的 task 需要 block 住知道条件允许
* 当前 task 唤醒后，可能条件不允许，他要继续等待
* 当前 task 唤醒后，可能操作的对象还在 block 中，那它就要继续 wait

#### Missed Singals

当两个 task 在通过 notify()/wait() 或者 notifyAll()/wait() 协调工作时，有可能错过一些指令，比如下面的例子

<setup condition for T2> 会组织 T2 调用 wait

```txt
T1: 
synchronized(sharedMonitor) {
    <setup condition for T2>
    sharedMonitor.notify(); 
}

T2: 
while(someCondition) {
    // Point 1   
    synchronized(sharedMonitor) 
    {     
        sharedMonitor.wait();   
    } 
} 
```

假设 T2 通过了 someCondition 的验证，在 Point1 时，切换到 T1，然后 T1 拿到锁并运行就结束，发出 notify() 这时切换回 T2 继续运行，发现 T2 一直等待，就死锁了。

T2 的正确写法应该是

```txt
synchronized(sharedMonitor) {
    while(someCondition)
        sharedMonitor.wait();
}
```

如果 T1 先运行，当返回到 T2 时，回判断 condition 不满足，将不会进入等待状态。相反，当 T2 先运行，他会进如 wait, 等待 T1 唤醒

PS: 看的很迷，我的逻辑应该是错的，但是我放弃思考了。。。

### notify() vs notifyAll()

notify 是 notifyAll 的一个优化，由于 notify 只会唤醒一个 task. 如果你要使用它，请确保，在你调用的时候只有想要调用的那个 task 是处于等待状态的。

notifyAll() 并不会 wake up "all waiting tasks", only the tasks that are waiting on a particular lock are awoken when notifyAll() is called/or that lock

实验说明如下：

Blocker 是操作的对象类，提供三个方法，waitingCall 用来停留在 wait() 状态，prod/prodAll 分别是唤醒单个和唤醒全部线程。

Task1/2 分别是两个 task, 功能一样，唯一的作用是提供提供两个操作类进行实验。

NotifyVsNotifyAll 为 client 类，先启动 5 个 Task1，再启动 1 个 Task2. 6 个线程都停留在 wait 状态。然后通过 timer 分别出发 Task1 的 prod 和 prodAll 方法，从输出可以看到，当调用 notify 时只有一个 Task1 被唤醒，当调用 notifyAll 时，所有 Task1 都醒了，Task2 毫无反应。只有最后结束时调用了 Task2 的 prodAll 时，Task2 对应的线程被唤醒。

```java
public class NotifyVsNotifyAll {
    public static void main(String[] args) throws InterruptedException {
        ExecutorService exec = Executors.newCachedThreadPool();
        for (int i = 0; i < 5; i++) {
            exec.execute(new Task());
        }
        exec.execute(new Task2());
        Timer timer = new Timer();
        timer.scheduleAtFixedRate(new TimerTask() {
            boolean prod = true;

            @Override
            public void run() {
                if (prod) {
                    System.out.println("\nnotify() ");
                    Task.blocker.prod();
                    prod = false;
                } else {
                    System.out.println("\nnotifyAll() ");
                    Task.blocker.prodAll();
                    prod = true;
                }
            }
        }, 400, 400);
        TimeUnit.SECONDS.sleep(5);
        timer.cancel();
        System.out.println("\nTimer canceled");
        TimeUnit.MILLISECONDS.sleep(500);
        System.out.println("Task2.blocker.prodAll() ");
        Task2.blocker.prodAll();
        TimeUnit.MILLISECONDS.sleep(500);
        System.out.println("\nShutting down");
        exec.shutdownNow();// Interrupt all tasks
    }
}

class Blocker {
    synchronized void waitingCall() {
        try {
            while (!Thread.interrupted()) {
                wait();
                System.out.println(Thread.currentThread() + " ");
            }
        } catch (InterruptedException e) {
            // OK to exit this way
        }
    }

    synchronized void prod() {
        notify();
    }

    synchronized void prodAll() {
        notifyAll();
    }
}

class Task implements Runnable {
    static Blocker blocker = new Blocker();

    @Override
    public void run() {
        blocker.waitingCall();
    }
}

class Task2 implements Runnable {
    // A separate Blocker object:
    static Blocker blocker = new Blocker();

    @Override
    public void run() {
        blocker.waitingCall();
    }
}
// notify() 
// Thread[pool-1-thread-1,5,main] 

// notifyAll() 
// Thread[pool-1-thread-2,5,main] 
// Thread[pool-1-thread-1,5,main] 
// Thread[pool-1-thread-5,5,main] 
// Thread[pool-1-thread-4,5,main] 
// Thread[pool-1-thread-3,5,main] 

// ....

// Timer canceled
// Task2.blocker.prodAll() 
// Thread[pool-1-thread-6,5,main] 

// Shutting down
```

暂时就先看到这儿把，耐心已经磨光了，以后有动力了再接着看