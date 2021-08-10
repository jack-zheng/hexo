---
title: HTW ex03
date: 2021-07-13 17:23:41
categories:
- Tomcat
tags:
- How Tomcat Works
---

> **Chapter3** presents a simplified version of Tomcat 4's default connector.
> The application built in this chapter serves as a learning tool to understand the connector discussed in Chapter4

PS: 这个 project 有点老了，其中用到的 Catalina 包比较老, 找了半天

```xml
<dependency>
    <groupId>tomcat</groupId>
    <artifactId>catalina</artifactId>
    <version>4.0.4</version>
</dependency>
```

相比于 ex02 这章节实现的服务器多了如下功能

* connector parse request headers
* servlet can obtain headers, cookies parameter name/values, etc
* enhance response's getWriter

完成上述功能后，这个就是简化版的 Tomcat4 的 connector 了。Tomcat 的默认 connector 在 Tomcat4 时被 deprecated 了，不过还是有参考价值的。


## StringManager

开篇先介绍了一个用于做类似国际化的类 org.apache.catalina.util.StringManager. 原理很简单，就是这个类通过单例模式生成唯一对象，加载预先定义好的 properties，通过 getString 方法拿到对应语言的翻译。

StringManager 底层使用两个 Java 基础类做实现，一个是 ResourceBundle 另一个是 MessageFormat. ResourceBundle 可以通过 properties 加载多语言支持，MessageFormat 则用于格式化打印信息。

为了节省资源，StringManager 内部通过 Hashtable 存储多语言，并通过单例模式创建这个 field

```java
private static Hashtable managers = new Hashtable();

/**
    * Get the StringManager for a particular package. If a manager for
    * a package already exists, it will be reused, else a new
    * StringManager will be created and returned.
    *
    * @param packageName
    */

public synchronized static StringManager getManager(String packageName) {
    StringManager mgr = (StringManager)managers.get(packageName);
    if (mgr == null) {
        mgr = new StringManager(packageName);
        managers.put(packageName, mgr);
    }
    return mgr;
}
```

每一个 package 下的 LocalStrings 都有一个对象。

相比之前的 project，这章开始，代码开始分包

```txt
.
├── ServletProcessor.java
├── StaticResourceProcessor.java
├── connector
│   ├── RequestStream.java
│   ├── ResponseStream.java
│   ├── ResponseWriter.java
│   └── http
│       ├── Constants.java
│       ├── HttpConnector.java
│       ├── HttpHeader.java
│       ├── HttpProcessor.java
│       ├── HttpRequest.java
│       ├── HttpRequestFacade.java
│       ├── HttpRequestLine.java            拆出来一个单独的类代表 request 的第一行，包括请求类型，URI，协议等信息
│       ├── HttpResponse.java
│       ├── HttpResponseFacade.java
│       ├── LocalStrings.properties
│       ├── LocalStrings_es.properties
│       ├── LocalStrings_ja.properties
│       └── SocketInputStream.java
└── startup
    └── Bootstrap.java                      启动类，实例化 HttpConnector 并调用 start() 方法
```

Bootstrap.java 为启动类，内容很简单，就是 new 一个 connector 然后执行 start 方法，让 connector 常驻。

connector 下的类可以分为五类

* connect 即该类的辅助类 HttpConnector + HttpProcessor
* 代表 Http Request 的类即其辅助类
* 代表 Http Response 的类即其辅助类
* Facade 类
* Constant 常量类

类关系图

{% plantuml %}
HttpConnector "1"-"1" HttpProcessor

HttpProcessor "uses".up.> StringManager
HttpProcessor "uses".up.> SocketInputStream

HttpProcessor "uses".up.> HttpHeader
HttpProcessor "uses".up.> HttpRequestLine

HttpProcessor "1"*.down."1" ServletProcessor
HttpProcessor "1"*.down."1" StaticResourceProcessor

ServletProcessor "uses".down.> HttpRequest
ServletProcessor "uses".down.> HttpResponse
StaticResourceProcessor "uses".down.> HttpRequest
StaticResourceProcessor "uses".down.> HttpResponse
{% endplantuml %}

和 ex02 比，这里将 HttpServer 拆成了 HttpConnector 和 HttpProcessor 两个类。HttpConnector 等待 request， HttpProcessor 负责 request/response 的生成和处理。

为了提高 connector 的效率，设计的时候将 request 中的 parse 的行为尽可能的延后了(比如有些 servlet 根本不需要 request 中的参数，这样 parse 就显得很多余，白白浪费了时间)。

TODO：connector 中的 SocketInputStream 有很方便的处理 request line 的方法，明天有机会可以测试一波

HttpProcessor 新建 request 并填充信息，比如 header 之类的，具体到 url 参数的解析，则由 request 类自己负责。

HttpRequest 的继承关系图如下

{% plantuml %}
skinparam linetype ortho
interface javax.servlet.http.HttpServletRequest

HttpRequestFacade .up.|> javax.servlet.http.HttpServletRequest
HttpRequest .up.|> javax.servlet.http.HttpServletRequest


interface javax.servlet.ServletInputStream
RequestStream .up.|> javax.servlet.ServletInputStream

RequestStream "1"-*"1" HttpRequest
{% endplantuml %}

servlet/JSP 程序中通过 JsessionId 指代 session。 解析 request 相关的内容时，需要解析 cookie 中的这个值，如果客户端没有 enable cookie 还需要将它 append 到 URL 中

关于本章中用到的 HttpHeader 你暂时只需要知道如下几点就行

* 可以使用无参数构造器创建实例
* SocketInputStream 的 readHeader 会将流中的 header 部分解析并填充进指定的 HttpHeader 对象
* 通过 header.name 和 header.nameEnd 可以方便的提取属性名和值

HttpReponse 类图

{% plantuml %}
skinparam linetype ortho

Interface javax.servlet.http.HttpServletResponse
Interface javax.servlet.ServletOutputStream
Interface java.io.PrintWriter

javax.servlet.http.HttpServletResponse <|.. HttpResponseFacade
javax.servlet.http.HttpServletResponse <|.. HttpResponse


javax.servlet.ServletOutputStream <|.. ResponseStream


java.io.PrintWriter <|.. ResponseWriter


HttpResponse "1"*-l->"1" ResponseStream
HttpResponse "uses"-r-> ResponseWriter
{% endplantuml %}

通过设置 PrintWriter 的 auto flush 功能，之前打印的 behavior 才修复了，不然只会打印第一句话。为了了解这里说的东西，你需要查一下 Writer 相关的知识点。

## 深入了解各个类的细节

### HttpConnector

服务器的主体部分，负责指定 ip 和 port 并 stand by. 每当有 request 过来就新建一个 HttpProcessor 对象处理对应的 socket。 

### HttpRequestLine

代表的是 request 的第一行的内容，示例如下 `GET /servlet/ModernServlet?userName=tarzan&password=pwd HTTP/1.1` 不过它的实现比较有意思，它为这一样中的各个部分声明了一个存储的 char 数组，并标识了结束地址 `char[] method, int methodEnd`

{% plantuml %}
Class HttpRequestLine {
    +char[] method;
    +int methodEnd;
    +char[] uri;
    +int uriEnd;
    +char[] protocol;
    +int protocolEnd;

    +HttpRequestLine();
    +HttpRequestLine(method, methodEnd, uri, uriEnd, protocol, protocolEnd);
}
{% endplantuml %}

SocketInputStream 是处理 HttpRequestLine 的类，主要涉及的方法 

* readRequestLine(HttpRequestLine) 填充 line 对象的方法入口
* fill() 使用 buffer 的方式读取输入流中的内容，这个过程中会初始化 pos 和 count 的值。pos 表示当前位置，count 表示流中内容长度
* read() 放回 pos 位置上的内容

SocketInputStream 的 read() 方法有一个很有意思的处理方式

```java
/**
* Read byte.
*/
public int read()
    throws IOException {
    if (pos >= count) {
        fill();
        if (pos >= count)
            return -1;
    }
    return buf[pos++] & 0xff;
}
```

可以看到最后的处理方式是返回 `buf[n] & 0xff` 0xff 即 0000 0000 0000 1111 做与操作可以将前面的值置位

readRequestLine 中用了三个 while 循环通过判断空格和行结束符将首行的信息提取出来。很雷同的还有一个叫 readHeader() 的方法处理解析 request 中的 headers.

### HttpProcessor

HttpProcessor 的主体方法是 process， 只做了几件事

* 解析 request line
* 解析 headers
* 根据 uri 调用对应的 processor

{% plantuml %}
(*) --> "init request, response"
--> "parse request line"
--> "parse headers"
if "uri contains '/servlet/'" then
    --> "init ServletProcessor"
    --> "invoke process()"
else
    --> "init StaticProessor"
    --> "invoke process()"
endif
--> (*)
{% endplantuml %}

parseReauest 过程如下

{% plantuml %}
(*) --> "populate to HttpRequestLine obj"
if "requestLine contains '?'" then
    --> [true] "request.setQueryString() + init uri"
    --> "absolute uri check"
else
    --> "init uri"
endif
--> "absolute uri check"
if "uri contains jsession" then
    --> [true] "parse + reqeust.setReqeustedSessionId() + set flag"
    --> "normalize uri"
else
--> "set flag"
endif
--> "normalize uri"
--> (*)
{% endplantuml %}

normalize 即处理一些特殊字符，比如 `..` 是上一级目录

parseHeaders 的过程和 parseRequest 雷同，while 循环逐行解析并向 request 中设置值。不过 ex03 中只处理了 cookie, content-length 和 content-type 几种类型的 header

### HttpResponse

实现了 HttpServletResponse 接口，包含各种常用的状态量，我最看不过的是作者将处理静态请求的部分也放在 response 的实现类中了。。。

## 问题

> server 启动后访问 URL 抛异常 `Exception in thread "Thread-0" java.util.MissingResourceException: Can't find bundle for base name com.jzheng.connector.http.LocalStrings, locale en_US`

查看了一下编译后的 target 文件加，其中咩有 properties 文件，怀疑是一些类型的文件编译时没有同步过去，试着在 pom 中添加以前项目中用过的 build 代码，问题解决

```xml
<build>
    <plugins>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-compiler-plugin</artifactId>
            <configuration>
                <source>1.8</source>
                <target>1.8</target>
            </configuration>
        </plugin>
    </plugins>
    <!-- 在 build 的时候将工程中的配置文件也一并 copy 到编译文件中，即 target 文件夹下 -->
    <resources>
        <resource>
            <directory>src/main/resources</directory>
            <includes>
                <include>**/*.properties</include>
                <include>**/*.xml</include>
            </includes>
        </resource>
        <resource>
            <directory>src/main/java</directory>
            <includes>
                <include>**/*.properties</include>
                <include>**/*.xml</include>
            </includes>
            <filtering>true</filtering>
        </resource>
    </resources>
</build>
```