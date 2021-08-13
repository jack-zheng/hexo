---
title: Ex04 Tomcat 自带的 connector 实现解析
date: 2021-07-20 10:32:35
categories:
- Tomcat
tags:
- How Tomcat Works
---

> **Chapter4** presents Tomcat 4's default connector. 
> This connector has been deprecated in favor of a faster connector called Coyote. Nevertheless, the default connector is simpler and easier to understand.

Tomcat 的 connector 是一个独立的模块，现存的比较有名的 connector 有 Coyote, mod_jk, mod_jk2 和 mod_webapp. Tomcat 的 connector 实现 需要 follow 一下标准

* 必须实现 org.apache.catalina.Connector 接口
* 创建的 request 必须实现 org.apache.catalina.Request 接口
* 创建的 response 必须实现org.apache.catalina.Response 接口

Tomcat4 的默认 connector 做的事情和第三章的没什么区别，它会一直 stand by 等待 Http request 的到来，然后创建 request 和 response 对象，并通过调用 org.apache.catalina.Container 实现类的 invoke 方法传送给 container。

```java
public void invoke(
    org.apache.catalina.Request request,
    org.apache.catalina.Response response
    );
```

invoke 方法中，container 会加载 servlet，调用其 service 方法，管理 session，log 等事情

默认的 tomcat connector 和 ex03 有点不同，它提供了 pool 机制来减小创建对象的开销，同时更多的使用 char arry 代替 string。

PS: 这节里面的多线程操作，值得好好看一看，之前一直都没有机会接触相关的知识点 (●°u°●)​ 」

默认的 connector 实现了所有 HTTP 1.1 的特性，同时也支持老版本的 HTTP 协议，比如 0.9 和 1.0. 理解 1.1 的协议对下面理解 connector 实现原理很重要。之后我们辉介绍 tomcat 自定义的 connector 接口(org.apache.catalina.Connector).

## HTTP 1.1 New Features

下面介绍 HTTP 1.1 的新特性

### Persistent Connections

HTTP 1.1 之前的协议，在请求完后会关闭链接。但是现在一个网页请求中可能会包含很多资源，比如 images， applets 等。如果这些资源都通过不同的 connection 下载，那么整个过程会很慢。使用 persistent connection 之后，连接将被复用, 减小资源开销。

persistent connection 是 HTTP 1.1 的默认配置，你也可以通过 `connection: keep-alive` 属性显示的指定。

### Chunked Encoding

persistent connection 导致的一个结果是，发送方必须在发送 request 或 response 时指定自己发送的内容的长度。但是通常情况下服务器端并不能做到这一点, 服务器发送内容的时候，是准备一点，发送一点，所以很可能发送的时候根本不知道将要发送多少内容。比如 servlet 会在一些数据准备好后就发送，并不会等到所有数据都完备再开始。

HTTP 1.0 的时候并不需要指定这个长度属性，连接会一直保持直到接收到 -1 这个结束标志符。

HTTP 1.1 通过 transfer-encoding 这个标志位表示将要发送的流长度。每个 chunk 数据发送前都会先发送一个 16 进制长度 + CR/LF 的标志位。如下所示，我们以发送文字 `I'm as helpless as a kitten up a tree.` 为例

发送时，这段文字被分成 2 个 chunks，第一个 chunk 长度为 29 第二个 chunk 长度为 9 那么体现在实际的 request 中为如下情况

```txt
1D\r\n
I'm as helpless as a kitten u
9\r\n
p a tree.
0\r\n
```

1D 是 16 进制的 29， 表示第一个 chunk 包含 29 个 bytes. 0\r\n 表示通信结束。

### Use of the 100(Continue) Status

当客户端发送的 request body 很大时，他会在 header 中包含 100-continue 属性来和服务器端确认是否接收来提高效率，避免资源浪费(传到一半被拒绝)。服务器如果接收这种 request， 则返回 `HTTP/1.1 100 Continue`

## The Connector interface

Tomcat connector 必须实现 org.apache.catalina.Connector 接口，这个接口有很多方法，但是最主要方法有四个

* getContainer
* setContainer
* createReqeust
* createResponse

{% plantuml %}
Interface Request
Interface Response

Request <.."instantiates" Connector
Request <.."uses" Container
Response <.."instantiates" Connector
Response <.."uses" Container


Interface Connector
Connector <|.. HttpConnector

Interface Container
Container <|.. SimpleContainer

Connector "1"->"1" Container

HttpConnector "1"--"*" HttpProcessor

HttpProcessor "uses".d.> HttpHeader
HttpProcessor "uses".d.> RequestLine
HttpProcessor "uses".d.> SocketInputStream
HttpProcessor "uses".d.> StringManager
{% endplantuml %}

重点：Connector 和 Container 是 1 对 1 的关系，Connector 和 Processor 是 1 对多的关系

## The HttpConnector Class

{% plantuml %}
Interface Connector
Interface Lifecycle
Interface Runnable

Connector <|.. HttpConnector
Lifecycle <|.. HttpConnector
Runnable <|.. HttpConnector
{% endplantuml %}

实现 org.apache.catalina.Connector 接口使它能和 Catalina 整合

实现 java.lang.Runnable 使他能多线程运行

Lifecycle 接口用于管理每一个 catalina component 的生命周期，具体内容第六章介绍

### Creating a Server Socket

HttpConnector 的 initialize() 方法会调用 open 方法生成 serverSocket 对象。open 中通过工厂方法拿到 ServerSocket, 参见 ServerSocketFactory 和对应的实现 DefaultServerSocketFactory

```java
/**
* Return the server socket factory used by this Container.
*/
public ServerSocketFactory getFactory() {

    if (this.factory == null) {
        synchronized (this) {
            this.factory = new DefaultServerSocketFactory();
        }
    }
    return (this.factory);

}
```

lazy model 的方式拿到工厂实例。然后调用 factory.createSocket(port, acceptCount) 创建 socket.

### Maintaining HttpProcess Instances

HttpContainer 中声明了一个 java.io.Stack 类型的变量存储 processor 的实例，实现类似 pool 的效果。

```java
private Stack processor = new Stack();
```

HttpConnector 中定义了两个变量(minProcessors/maxProcessors)来控制这个 stack 的大小, 在启动的时候，服务器默认创建 minProcessors 数量的 processor 备用。随着 request 的增加，这个数量也会增加知道等于 maxProcessors。如果 request 再增加，之后的 request 都会被忽略。如果你想要访问数量没有限制，可以设置 maxProcessor 为负数。

PS: HttpConnector 中通过 curProcessor 这个变量表示当前可用的 processor 数量

```java
private Stack processors = new Stack();
// ...
// Create the specified minimum number of processors
while (curProcessors < minProcessors) {
    if ((maxProcessors > 0) && (curProcessors >= maxProcessors))
        break;
    HttpProcessor processor = newProcessor();
    recycle(processor);
}
// ...
void recycle(HttpProcessor processor) {
    processors.push(processor);
}
```

 processor 创建完后，通过调用 recycle 方法将 processor 放入栈中。processor 负责负责解析 request 内容，他的构造函数的参数中包含 HttpConnector, 在构造的过程中，会调用 connector 中创建 request 和 response 的方法。

### Serving HTTP Requests

HttpConnector 的主要逻辑都在 run 方法中，该方法通过 while 循环等待发送搞来的响应，知道服务器停止。

```java
public void run() {
        // Loop until we receive a shutdown command
        while (!stopped) {
            // Accept the next incoming connection from the server socket
            Socket socket = null;
            try {
                socket = serverSocket.accept();
                // ...
            }
            // ...
        }
        // ...
}
```

{% plantuml %}
(*) --> "accept socket"
if "create processor" then
--> [not null] "processor assign socket"
--> (*)
else
--> "log + close socket"
--> (*)
endif
{% endplantuml %}

createProcessor 工作流程

1. 如果 stack 中有，则返回
2. 如果没有，判断是否达到上限，没有就创建
3. 达到上限，返回并关闭 socket
4. 上限为 -1，创建 processor

processor 执行 assign() 方法后立即返回，后续工作由 processor 在单独的线程中完成

## The HttpProcessor Class

HttpProcessor 的功能和前一章中的 processor 是一样的，这章中的实现多了 assign 之后的多线程执行的功能。下面的内容将具体介绍他的实现原理。

和 HttpConnector 类似 HttpProcessor 也实现了 Runnable 和 Lifecycle 接口

{% plantuml %}
Interface Lifecycle
Interface Runnable

Lifecycle <|.. HttpProcessor
Runnable <|.. HttpProcessor
{% endplantuml %}

这里主要探究 processor 的 assign 方法是如何使用多线程来支持 tomcat 同时处理多个 request 的功能的

> For each HttpProcessor instance the HttpConnector creates, its start method is called, effectively starting the "processor thread" of the HttpProcessor instance.

HttpConnector 的 start() 方法被调用时，processor thread 立马就启动了

{% plantuml %}
(*) --> "get socket"
--> "process it"
--> "recycle, put back to stack"
--> (*)
{% endplantuml %}


processor 的 run 方法实现如下

```java
/**
* The background thread that listens for incoming TCP/IP connections and
* hands them off to an appropriate processor.
*/
public void run() {

    // Process requests until we receive a shutdown signal
    while (!stopped) {

        // Wait for the next socket to be assigned
        Socket socket = await();
        if (socket == null)
            continue;

        // Process the request from this socket, omit the try-catch
        process(socket);

        // Finish up this request
        connector.recycle(this);
    }

    // Tell threadStop() we have shut ourselves down successfully
    synchronized (threadSync) {
        threadSync.notifyAll();
    }
}
```

recycle 方法实现如下

```java
void recycle(HttpProcessor processor) {
    processors.push(processor);
}
```

当 connector 启动的时候 processor thread 也会一起启动，然后卡在 await 这里一直等待。让 HttpConnector 接受到 request 之后会调用 processor.assign(socket) 方法。

这里需要注意的是 assig() 方法是在 connector thread 中调用的，而 await() 方法是在 processor thread 中被调用的。这两者是怎么通信的呢？他们是通过 available flag 和 Object 自带的 wait(), notifyAll() 方法控制调度的。

PS: wait() 方法使得当前线程保持等待一直到另一个线程调用 notify() 或者 notifyAll() 方法

HttpConnector 中 assign 的实现

```java
/**
* Process an incoming TCP/IP connection on the specified socket.  Any
* exception that occurs during processing must be logged and swallowed.
* <b>NOTE</b>:  This method is called from our Connector's thread.  We
* must assign it to our own thread so that multiple simultaneous
* requests can be handled.
*
* @param socket TCP socket to process
*/
synchronized void assign(Socket socket) {

    // Wait for the Processor to get the previous Socket
    while (available) {
        try {
            wait();
        } catch (InterruptedException e) {
        }
    }

    // Store the newly available Socket and notify our thread
    this.socket = socket;
    available = true;
    notifyAll();

    if ((debug >= 1) && (socket != null))
        log(" An incoming request is being assigned");

}
```

HttpConnector 中 await 的实现

```java
/**
* Await a newly assigned Socket from our Connector, or <code>null</code>
* if we are supposed to shut down.
*/
private synchronized Socket await() {

    // Wait for the Connector to provide a new Socket
    while (!available) {
        try {
            wait();
        } catch (InterruptedException e) {
        }
    }

    // Notify the Connector that we have received this Socket
    Socket socket = this.socket;
    available = false;
    notifyAll();

    if ((debug >= 1) && (socket != null))
        log("  The incoming request has been awaited");

    return (socket);

}
```

TODO: connector thread 和 processor thread 的处理时间线 UML 图示

两个 thread 交互描述：

服务器奇松时 processor thread 一起启动并 block 在 wait() 方法处死循环等待唤醒。这时如果 connector thread 中接收到一个 request， connector 会从 stack 中取出一个可用的 processor 并调用 assing(socket) 方法。

assign 方法会判断 available flag, 初始值为 false， 跳过 while, 将 socket 复刻到成员变量，设置 available 为 true, 唤醒所有等待的线程。

这时 process thread 的 await() 方法从 wait() 中醒过来，跳出 while 循环将 socket 复刻到 local 变量中，并将 available 设置成 false，调用 notifyAll() 唤醒其他线程。

> 问题：
> Q：为什么 await 要使用 local variable 类型的 socket 而不是直接返回传入的 socket
> A: 没有用 local 的 socket 的时候， socket 还是 connector 中的那个 socket，我们用 local 的复刻之后返回，这个  socket 就可以用来处理下一个 request 了。
> Q: 为什么 await 需要调用 notifyAll() 方法
> A: 处理了第一个 request 之后，connector thread 如果接受到第二个 request，available 是 true，就会卡住，需要 notifyAll() 唤醒它

PS: 第二个问题不是很清楚，需要画图来帮助理解

PPS: 再看看创建多个 processor 的那部分代码，有助于理解这里的内容

## Request Objects / Request Objects

使用的是 catalina 自己定义的接口

## Process Requests

预先设置了一些 flag, 如 ok，finishResponse，stopped， keepAlive 等控制解析流程。主要就做了几件事

* parseConnection - 获取地址并塞到 request 中
* parseRequest - 同上节
* parseHeader - 解析 header 并塞到 request 中
* recycle response + request - 复用对象，相比于上一节完善了很多

## The Simple Container Application

这里的 container 是一个简易版本，实现了 catalina 中的 Container 接口以配合 connector 使用。只实现了 invoke 接口，里面的功能是加载 servlet class 并执行
