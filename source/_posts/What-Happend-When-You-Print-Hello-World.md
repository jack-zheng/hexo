---
title: 当你打印 Hello World 的时候到底发生了什么？
date: 2020-08-24 16:59:59
categories:
- 编程
tags:
- java
- jvm
---

一个最简单的例子，当我们在 IDE 中写入 Hello World 代码，并右键运行后，控制台会打印出来 `Hello World!` 的字符串，那么这中间到底发生了什么？

```java
public class Hello {
    String name = "Jack";
    public static void main(String[] args) {
        Hello test = new Hello();
        System.out.println("Hello " + test.name);
        while (true) {}
    }
}
```

写入 IDE 里的代码都是存到 `.java` 文件中的，在保存后 IDE 会将它编译为 `.class` 文件。这个文件也叫字节码文件，有自己的一套规则。之后当我们运行这个字节码文件时，一个 JVM 虚拟机被启动，解析这个文件，将文件中的各种变量，方法分配到虚拟机的各功能区。运行代码中的打印逻辑，并输出到终端。

## java -> class

从 java 文件到 class 的功能可以简单概括为 javac 命令的功能。其中主要涉及到编译器的相关只是，可以参考 编译原理 加深了解。简单概括步骤有：词法分析 -> 语法分析 -> 语义分析 -> 字节码生成

## 运行

运行主要涉及到 JVM 启动，类加载，逻辑执行，可以通过看 深入理解JVM虚拟机 加深了解

### jmap 查看堆中对象分布

运行示例代码，通过 `ps -ef | grep Hello` 拿到线程 pid. 然后使用 `jmap -heap <pid>` 查看对象情况。结果失败。。。

PS: jps 可以很方便的查看 java 程序 pid

```bash
Jack > ~ > jmap -heap 68202
Attaching to process ID 68202, please wait...
ERROR: attach: task_for_pid(68202) failed: '(os/kern) failure' (5)
Error attaching to process: sun.jvm.hotspot.debugger.DebuggerException: Can't attach to the process. Could be caused by an incorrect pid or lack of privileges.
sun.jvm.hotspot.debugger.DebuggerException: sun.jvm.hotspot.debugger.DebuggerException: Can't attach to the process. Could be caused by an incorrect pid or lack of privileges.
```

操作系统为 MacOS, java1.8。一开始说是权限问题，但是用了 root 也不顶用，然后说是 1.8 以前不支持。本地安装 j14 然后按照之前的步骤运行 cmd 还是一样的错误。在 14 版本中，命令变了，j9 之后需要使用 `jhsdb jmap --heap --pid  68633` 做查询。难道是 MacOS 需要什么特殊设置 (´Д` ) 容我找太其他系统的机子试试水先。。。

可能就是系统问题把，或者公司的机子有什么限制？用家里的 Windows 试了下是可以 work 的

```bash
PS C:\Users\jack> jhsdb jmap --heap --pid 10648
Attaching to process ID 10648, please wait...
Debugger attached successfully.
Server compiler detected.
JVM version is 11.0.6+8-LTS

using thread-local object allocation.
Garbage-First (G1) GC with 4 thread(s)

Heap Configuration:
   MinHeapFreeRatio         = 40
   MaxHeapFreeRatio         = 70
   MaxHeapSize              = 2118123520 (2020.0MB)
   NewSize                  = 1363144 (1.2999954223632812MB)
   MaxNewSize               = 1270874112 (1212.0MB)
   OldSize                  = 5452592 (5.1999969482421875MB)
   NewRatio                 = 2
   SurvivorRatio            = 8
   MetaspaceSize            = 21807104 (20.796875MB)
   CompressedClassSpaceSize = 1073741824 (1024.0MB)
   MaxMetaspaceSize         = 17592186044415 MB
   G1HeapRegionSize         = 1048576 (1.0MB)

Heap Usage:
G1 Heap:
   regions  = 2020
   capacity = 2118123520 (2020.0MB)
   used     = 1048576 (1.0MB)
   free     = 2117074944 (2019.0MB)
   0.04950495049504951% used
G1 Young Generation:
Eden Space:
   regions  = 1
   capacity = 15728640 (15.0MB)
   used     = 1048576 (1.0MB)
   free     = 14680064 (14.0MB)
   6.666666666666667% used
Survivor Space:
   regions  = 0
   capacity = 0 (0.0MB)
   used     = 0 (0.0MB)
   free     = 0 (0.0MB)
   0.0% used
G1 Old Generation:
   regions  = 0
   capacity = 118489088 (113.0MB)
   used     = 0 (0.0MB)
   free     = 118489088 (113.0MB)
   0.0% used
```

使用 histo 参数查看对象大小

```bash
PS C:\Users\jack> jhsdb jmap --histo --pid 10648
Attaching to process ID 10648, please wait...
Debugger attached successfully.
Server compiler detected.
JVM version is 11.0.6+8-LTS
Iterating over heap. This may take a while...
Object Histogram:

num       #instances    #bytes  Class description
--------------------------------------------------------------------------
1:              551     357184  char[]
2:              3641    258616  byte[]
3:              1629    98808   java.lang.Object[]
...
304:            1       16      float[]
305:            1       16      boolean[]
306:            1       16      Hello
...
```