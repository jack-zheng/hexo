---
title: Ex05 自定义 Container
date: 2021-07-22 15:47:11
categories:
- Tomcat
tags:
- How Tomcat Works
---

> **Chapter 5** discusses the container module.
> A container is represented by the org.apache.catalina.Container interface and there are four types of containers: engine, host, context, and wrapper.
> This chapter offers two applications that work with contexts and wrappers.

Tomcat 中有 4 种 container: Wrapper, Context, Engine and Host. container module 的主要作用是用来处理 request 并向 response 中填充处理结果。这节主要介绍前两种，后两种将在 13 节中介绍。

这节的主要目标是理解一下几个概念分别代表了什么

Container: 这个类注释解释的挺好的。container 表示的是可以执行 client 传过来的 request 的类。container 会将他的 invoke 委托给 pipeline 执行

> A Container is an object that can execute requests received from a client, and return responses based on those requests.

Pipeline: 表示装载着 container 将会 invoke 的 tasks 的容器

Valve: 表示将会被执行的 task, 有一个特殊的 valve 叫做 basic valve, 要求最后执行

* ValveContext

## The Container Interface

container 必须实现 catalina 的 Container 接口。connector 会调用实现类的 invoke 逻辑(怎么调用的我一直没看到，估摸着应该在 Lifecycle 部分才涉及到)。按处理的业务来分，有四种类型的 Container

* Engine: 代表整个 Catalina servlet engine - 虽然我对这里说的 engine 是个什么东西不怎么清楚
* Host: 代表了含有 contexts 的虚拟 host 
* Context: 代表了一个 web application, 一个 application 可以包含一个或多个 wrapper
* Wrapper: 代表一个独立的 servlet

以上四种概念的接口定义放在 org.apache.catalina 包下，对应的实现放在 org.apache.catalina.core 下

{% plantuml %}
Interface Container
Interface Engine
Interface Host
Interface Context
Interface Wrapper
Class ContainerBase

Container <|-- Engine
Container <|-- Host
Container <|-- Context
Container <|-- Wrapper
Container <|.. ContainerBase

Engine -[hidden]> Host
Host -[hidden]> Context
Context -[hidden]> Wrapper
Wrapper -[hidden]> ContainerBase

Class StandardEngine
Class StandardHost
Class StandardContext
Class StandardWrapper

Engine <|.. StandardEngine
Host <|.. StandardHost
Context <|.. StandardContext
Wrapper <|.. StandardWrapper

ContainerBase <-- StandardEngine
ContainerBase <-- StandardHost
ContainerBase <-- StandardContext
ContainerBase <-- StandardWrapper

StandardEngine "1" o- "*" StandardHost
StandardHost "1" o- "*" StandardContext
StandardContext "1" o- "*" StandardWrapper
{% endplantuml %}

一个可用的 container 并不需要具备四种 container 类型，最简单的案例只需要一个 wrapper 即可。wrapper 是最低等级的 container，不能再包含其他 container。container 支持的常规操作

1. addChild() - wrapper 除外
2. removeChild()
3. findChild()
4. findChildren()

container 还可以包含其他辅助类，比如 Loader, Logger, Manager, Realm 和 Resources. container 还可以通过配置 server.xml 使他在启动服务器时达到动态指定的效果。这种特性是通过 pipeline 和 valves 达到的。

## Pipeline Tasks

这节介绍当 container 的 invoke 方法被调用的时候会发生什么。主要涉及到四个接口 Pipeline, Valve, ValveContext 和 Contained. pipeline 包含了 container 将要执行的 tasks, valve 即将要执行的 task. container 默认有一个 valves 但是我们可以自己添加任意多个自定义的 valves. valves 也可以通过配置 server.xml 指定。

这里有个图，但是没显示，我猜是这种责任链的图

![pipeline and valves](pipeline_valve.png)

pipeline 的工作原理和 servlet 的 filter 是一样的, 使用责任链模式。pipeline 相当于链，valves 相当去 filter。当一个 valve 执行完了之后，会调用下一个 valve 继续执行。自带的那个 basic valve 总是在最后才被调用。

按上面的逻辑，你可能会用如下方式实现 pipeline

```java
// invoke each valve added to the pipeline
for(int n=0; n<valves.length; n++) {
    valves[n].invoke(...);
}
// then, invoke the basic valve
basicValve.invoke(...);
```

但是 Tomcat 的设计者通过引入 ValveContext 这个 interface 来解决这个问题，工作原理如下

Container 的 invoke() 方法被调用的时候，并不是 hard code 需要做的事情，而是通过调用 pipeline 的 invoke() 方法。pipeline 和 container 的 invoke() 方法定义如下

```java
public interface Pipeline {
    public void invoke(Request request, Response response)
        throws IOException, ServletException;
}

public interface Container {
        public void invoke(Request request, Response response)
        throws IOException, ServletException;
}

// container 的实现
public abstract class ContainerBase implements Container, Lifecycle, Pipeline {
    protected Pipeline pipeline = new StandardPipeline(this);

    public void invoke(Request request, Response response)
        throws IOException, ServletException {

        pipeline.invoke(request, response);

    }
}
```

pipeline 需要保证所有被添加进来的 valves 和 basic valve 只被调用一次。pipeline 是通过 ValveContext 这个接口实现该功能的。ValveContext 是 pipeline 的一个内部类(innerClass)，通过这种定义使得 ValveContext 可以访问 pipeline 中的所有对象。ValveContext 中最重要的方法是 invokeNext

```java
public interface ValveContext {
    public void invokeNext(Request request, Response response)
        throws IOException, ServletException;
}

public class SimplePipeline implements Pipeline {
    protected Valve valves[] = new Valve[0];

    public void addValve(Valve valve) {...}
    
    protected class SimplePipelineValveContext implements ValveContext {
        protected int stage = 0;

        public String getInfo() {
            return null;
        }

        public void invokeNext(Request request, Response response)
                throws IOException, ServletException {
            // subscript: 下标 + stage 用来标记被调用的 valve
            int subscript = stage;
            stage = stage + 1;
            // Invoke the requested Valve for the current request thread
            if (subscript < valves.length) {
                valves[subscript].invoke(request, response, this);
            } else if ((subscript == valves.length) && (basic != null)) {
                basic.invoke(request, response, this);
            } else {
                throw new ServletException("No valve");
            }
        }
    }
}
```

ValveContxt 会调用第一个 valve 的 invoke 方法，第一个 valve 会调用第二个 valve 的 invoke 方法。Valve 的 invoke 方法的参数列表中包含 ValveContext 方便调用 invokeNext 方法。

```java
public interface Valve {
    public void invoke(Request request, Response response, ValveContext context) throws IOException, ServletException;
}
```

### The Pipeline Interface

Pipeline 接口定义, container 通过调用它来处理 valves 和 basic valve.

```java
public interface Pipeline {
    // 操作 basic valve
    public Valve getBasic();
    public void setBasic(Valve valve);

    public void addValve(Valve valve);
    public Valve[] getValves();

    public void invoke(Request request, Response response)
        throws IOException, ServletException;

    public void removeValve(Valve valve);
}
```

### The Valve Interface

这个 component 用于处理一个 request，只有两个方法 invoke 和 getInfo

```java
public interface Valve {
    public String getInfo();

    public void invoke(Request request, Response response, ValveContext context)
        throws IOException, ServletException;
}
```

### The ValveContext Interface

只有 invokeNext 和 getInfo 两个方法

```java
public interface ValveContext {

    public String getInfo();

    public void invokeNext(Request request, Response response)
        throws IOException, ServletException;
}
```

### The Contained Interface

valve 可以选择性的实现 Contained 接口，这个接口表明对应的实现最多只能和一个 container 有关系

```java
public interface Contained {
    public Container getContainer();
    public void setContainer(Container container);
}
```

## The Wrapper Interface

Wrapper 表示一个独立的 servlet 定义。Wrapper 的实现类负责管理 servlet 的生命周期。比如调用 init, service 和 destroy 方法。它是最底层的 container 实现，所以不能添加 child, 添加会抛错

```java
 public void addChild(Container child) {
    throw new IllegalStateException(sm.getString("standardWrapper.notChild"));
}
```

其他一些比较重要的方法比如 allocate 和 load

```java
public interface Wrapper extends Container {
    public Servlet allocate() throws ServletException;
    public void load() throws ServletException;
}
```

allocate 用于指定 wrapper 指代的 servlet，load 用于加载 servlet 的实例。

## The Context Interface

Context 指代一个 web application. 一个 context 可以包含一个或多个 wrapper

## The Wrapper Application

下面是本章的第一个例子，一个简单的 container，只由一个 wrapper 来充当 container 主体。包含七个类

* SimpleWrapper: Wrapper 的实现类，包含一个 Pipeline, 通过一个 Loader 来加载 servlet。
* SimplePipeline: Pipeline 的实现类，包含一个 basic valve 和两个额外 valve
* SimpleLoader: 用于加载 servlet
* SimpleWrapperValve: basic valve 的实现类
* ClientIPLoggerValve, HeaderLoggerValve: 额外 valve 的实现类
* Bootstrap1: 启动类

SimpleWrapperValve 和额外的 Valve 最大的区别是，SimpleWrapperValve 没有在调用 invkeNext, 因为规则上来说，它是最后一个需要调用的 valve 了。

{% plantuml %}
interface Loader
Loader <|.. SimpleLoader

interface Container
Loader "1"-r-*"1" Container

interface Wrapper
Container <|-- Wrapper

Wrapper <|.. SimpleWrapper

interface Pipeline
Wrapper -[hidden] Pipeline

Pipeline <|.. SimpleWrapper
Pipeline <|.. SimplePipeline

interface Valve
Pipeline "1" o- "n" Valve

Valve "previous"-> Valve
Valve "next"-> Valve

Valve <|.. SimpleWrapperValve
Valve <|.. ClientIPLoggerValue
Valve <|.. HeaderLoggerValue
{% endplantuml %}

### Running the Application

启动服务器，访问 `http://localhost:8080` 终端显内容如下

```txt
ModernServlet -- init
Client IP Logger Valve
0:0:0:0:0:0:0:1
------------------------------------
Header Logger Valve
host:localhost:8080
connection:keep-alive
sec-ch-ua:"Chromium";v="92", " Not A;Brand";v="99", "Google Chrome";v="92"
sec-ch-ua-mobile:?0
upgrade-insecure-requests:1
user-agent:Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/92.0.4515.107 Safari/537.36
accept:text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9
sec-fetch-site:none
sec-fetch-mode:navigate
sec-fetch-user:?1
sec-fetch-dest:document
accept-encoding:gzip, deflate, br
accept-language:en-US,en;q=0.9,zh-CN;q=0.8,zh;q=0.7,zh-TW;q=0.6
cookie:loginMethodCookieKey=PWD; bizxThemeId=2wyfwkupsp; fontstyle=null; _pk_id.1.1fff=9a13b6b396550ca6.1619008137.; Idea-12b942b0=d6d70da6-7368-4631-b3f7-a91eafcb1e9f; Idea-cead54cd=ce25569f-051c-4d11-b8f4-6b99d512503d; JSESSIONID=B3BD850D653D266A9BB2346D7D97B8A2
------------------------------------
```

## The Context Application

当服务器需要处理多个 servlet 时，就需要用到 context 和 mapper 了。mapper 帮助父容器选择子 container 处理 request。

PS: mapper 只在 Tomcat4 中使用，到 Tomcat5 就使用其他技术了。

一个 container 可以使用多个 mapper 支持多种 protocols. 这个例子中只处理一种。比如一个 container 可以配置一个 mapper 处理 http 请求，配置另一个 mapper 处理 https.

```java
public interface Mapper {
    public Container getContainer();
    public void setContainer(Container container);
    public String getProtocol();
    public void setProtocol(String protocol);
    public Container map(Request request, boolean update);
}
```

{% plantuml %}
interface Container
interface Loader
interface Mapper

Mapper "*"-*"1" Container
Container "1"-"1" Loader

Mapper <|.. SimpleContextMapper
Loader <|.. SimpleLoader

interface Context
interface Wrapper
interface Pipeline
interface Valve

Context -[hidden] Wrapper
Wrapper -[hidden] Pipeline
Pipeline -[hidden] Valve

Pipeline "1" *- Valve

Container <|-- Context
Container <|-- Wrapper


Context <|.. SimpleContext
Wrapper <|.. SimpleWrapper

Pipeline <|.. SimplePipeline
Pipeline <|.. SimpleContext
Pipeline <|.. SimpleWrapper

SimpleContext -[hidden] SimpleWrapper

Valve <|.. SimpleContextValve
Valve <|.. SimpleWrapperValve
Valve <|.. ClientIPLoggerValve
Valve <|.. HeaderLoggerValve

SimpleContext "1" o-- "*" SimpleWrapper: "contains"
{% endplantuml %}

过程：

1. SimpleContext 调用 pipeline 的 invoke 方法
2. pipeline 的 invoke 方先调用额外 valves 再调用 basic valve
3. basic valve 的 invoke 方法会调用 map 方法找到子 wrapper，如果存在则调用其 invoke 方法