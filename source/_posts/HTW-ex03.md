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

PS: 它这里用的是饿汉式的声明，类加载的时候就创建了对象，调用 getManager() 的时候通过 synchronized 加锁保证线程安全。每一个 package 下的 LocalStrings 都回创建一个对象存储多语言信息。

## The Application

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

* connect 即该类的辅助类(HttpConnector + HttpProcessor)
* 代表 Http Request 的类(HttpRequest)及其辅助类
* 代表 Http Response 的(HttpResponse)及其辅助类
* Facade 类(HttpRequestFacade + HttpResponseFacade)
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

###  The Connector

HttpConnector 表示 connector 的实体类，他负责创建 server socket 并等待 Http request 的到来。HttpConnector 实现 runnable 接口，当 start() 被调用时，HttpConnector 被创建并运行。

connector 运行时回做如下几件事情

* 等待 HTTP requests
* 为每个 request 创建 HttpProcessor
* 调用 processor 的 process 方法

HttpProcessor 的 process 方法在拿到 socket 后，会做如下事情

* Create an HttpRequest/HttpResponse object
* Parse request first line and headers, populate to HttpRequest object
* Pass HttpRequest, HttpResponse to Processor(servlet process/static processor)

#### Create an HttpRequest Object

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

本章的 HttpRequest 实现的很多方法都留空了，下一章会有具体实现。但是 header，cookies 等主要属性的提取已经实现了。由于 HttpRequst 的解析比较复杂，下面会分几个小节介绍

#### Reading the Socket's input Stream

我们从 Tomcat4 中 copy 了 SocketInputStream 的实现，他负责解析从 socket 中获取的 input steam。

#### Parsing the Request Line

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

parseRequest 过程如下

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

request line 就是 inputStream 中的第一行内容，下面是示例

> GET /myApp/ModernServlet?userName=tarzan&password=pwd HTTP/1.1

各部分称谓如下

* GET - method
* /myApp/ModernServlet - URI
* userName=tarzan&password=pwd - query string
* parameters - userName/tarzan;password/pwd 成对出现

servlet/JSP 程序中通过 JsessionId 指代 session。 session 标识符通常通过 cookies 存储，如果客户端没有 enable cookie 还需要将它 append 到 URL 中

HttpProcessor 的 process 方法会将上面提到的对象重 inputStream 中提取出来并塞到对应的对象中

```java
private void parseRequest(SocketInputStream input, OutputStream output) throws IOException, ServletException {
    // Parse the incoming request line
    input.readRequestLine(requestLine);
    String method = new String(requestLine.method, 0, requestLine.methodEnd);
    String uri = null;
    String protocol = new String(requestLine.protocol, 0,
            requestLine.protocolEnd);

    // Validate the incoming request line
    if (method.length() < 1) {
        throw new ServletException("Missing HTTP request method");
    } else if (requestLine.uriEnd < 1) {
        throw new ServletException("Missing HTTP request URI");
    }
    // Parse any query parameters out of the request URI
    int question = requestLine.indexOf("?");
    if (question >= 0) {
        request.setQueryString(new String(requestLine.uri, question + 1,
                requestLine.uriEnd - question - 1));
        uri = new String(requestLine.uri, 0, question);
    } else {
        request.setQueryString(null);
        uri = new String(requestLine.uri, 0, requestLine.uriEnd);
    }

    // Checking for an absolute URI (with the HTTP protocol)
    if (!uri.startsWith("/")) {
        int pos = uri.indexOf("://");
        // Parsing out protocol and host name
        if (pos != -1) {
            pos = uri.indexOf('/', pos + 3);
            if (pos == -1) {
                uri = "";
            } else {
                uri = uri.substring(pos);
            }
        }
    }

    // Parse any requested session ID out of the request URI
    String match = ";jsessionid=";
    int semicolon = uri.indexOf(match);
    if (semicolon >= 0) {
        String rest = uri.substring(semicolon + match.length());
        int semicolon2 = rest.indexOf(';');
        if (semicolon2 >= 0) {
            request.setRequestedSessionId(rest.substring(0, semicolon2));
            rest = rest.substring(semicolon2);
        } else {
            request.setRequestedSessionId(rest);
            rest = "";
        }
        request.setRequestedSessionURL(true);
        uri = uri.substring(0, semicolon) + rest;
    } else {
        request.setRequestedSessionId(null);
        request.setRequestedSessionURL(false);
    }

    // Normalize URI (using String operations at the moment)
    String normalizedUri = normalize(uri);

    // Set the corresponding request properties
    ((HttpRequest) request).setMethod(method);
    request.setProtocol(protocol);
    if (normalizedUri != null) {
        ((HttpRequest) request).setRequestURI(normalizedUri);
    } else {
        ((HttpRequest) request).setRequestURI(uri);
    }

    if (normalizedUri == null) {
        throw new ServletException("Invalid URI: " + uri + "'");
    }
}
```

它的实现比较有意思，它为这一样中的各个部分声明了一个存储的 char 数组，并标识了结束地址 `char[] method, int methodEnd`

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


#### Parsing Headers

request 的 header 部分由 HttpHeader 这个类表示。将在第四章介绍具体实现，目前只需要了解一下几点

* 可以使用无参数构造器创建实例
* 通过调用 readHeader 方法 SocketInputStream 中的 header 部分解析并填充进指定的 HttpHeader 对象
* 通过 String name = new String(header.name, 0, header.nameEnd) 拿到 header 的 name, 同理获取 value

由于一个 request 中可能包含多个 header，所以通过 while 循环解析，解析完后通过 addHeader 塞入 request 中

```java
private void parseHeaders(SocketInputStream input) throws IOException, ServletException {
    while (true) {
        HttpHeader header = new HttpHeader();
        ;

        // Read the next header
        input.readHeader(header);
        if (header.nameEnd == 0) {
            if (header.valueEnd == 0) {
                return;
            } else {
                throw new ServletException(
                        sm.getString("httpProcessor.parseHeaders.colon"));
            }
        }

        String name = new String(header.name, 0, header.nameEnd);
        String value = new String(header.value, 0, header.valueEnd);
        request.addHeader(name, value);
        // do something for some headers, ignore others.
        if (name.equals("cookie")) {
            Cookie cookies[] = RequestUtil.parseCookieHeader(value);
            for (int i = 0; i < cookies.length; i++) {
                if (cookies[i].getName().equals("jsessionid")) {
                    // Override anything requested in the URL
                    if (!request.isRequestedSessionIdFromCookie()) {
                        // Accept only the first session id cookie
                        request.setRequestedSessionId(cookies[i].getValue());
                        request.setRequestedSessionCookie(true);
                        request.setRequestedSessionURL(false);
                    }
                }
                request.addCookie(cookies[i]);
            }
        } else if (name.equals("content-length")) {
            int n = -1;
            try {
                n = Integer.parseInt(value);
            } catch (Exception e) {
                throw new ServletException(
                        sm.getString("httpProcessor.parseHeaders.contentLength"));
            }
            request.setContentLength(n);
        } else if (name.equals("content-type")) {
            request.setContentType(value);
        }
    } // end while
}
```

#### Parsing Cookies

随便访问了一下网页，下面是一个 cookie 的例子

`cookie: fontstyle=null; loginMethodCookieKey=PWD; bizxThemeId=lightGrayPlacematBlueAccentNoTexture; route=133abdfd8b5240fdc3330810e535ae4c79433a08; zsessionid=45641c6c-9dff-4d67-8893-b0764636ee1f; JSESSIONID=D8477F13FD4A9257B98731F666694D91.mo-bce0c171c`

在前面的 parseHeaders 方法中，处理 cookie 的部分，通过 RequestUtil.parseCookieHeader(value) 解析 cookie

```java
/**
* Parse a cookie header into an array of cookies according to RFC 2109.
*
* @param header Value of an HTTP "Cookie" header
*/
public static Cookie[] parseCookieHeader(String header) {

    if ((header == null) || (header.length() < 1))
        return (new Cookie[0]);

    ArrayList cookies = new ArrayList();
    while (header.length() > 0) {
        int semicolon = header.indexOf(';');
        if (semicolon < 0)
            semicolon = header.length();
        if (semicolon == 0)
            break;
        String token = header.substring(0, semicolon);
        if (semicolon < header.length())
            header = header.substring(semicolon + 1);
        else
            header = "";
        try {
            int equals = token.indexOf('=');
            if (equals > 0) {
                String name = token.substring(0, equals).trim();
                String value = token.substring(equals+1).trim();
                cookies.add(new Cookie(name, value));
            }
        } catch (Throwable e) {
            ;
        }
    }

    return ((Cookie[]) cookies.toArray(new Cookie[cookies.size()]));

}
```

#### Obtaining Parameters

解析 parameter 的动作放在 HttpRequest 的 parseParameter 方法中。在调用 parameter 相关的方法，比如 getParameterMap, getParameterNames 等时，会先调用 parseParameter 方法解析他，而且只需要解析一次即可，再次调用是，使用之前解析的结果。

```java
/**
* Parse the parameters of this request, if it has not already occurred.
* If parameters are present in both the query string and the request
* content, they are merged.
*/
protected void parseParameters() {
    if (parsed)
        return;
    ParameterMap results = parameters;
    if (results == null)
        results = new ParameterMap();
    results.setLocked(false);
    String encoding = getCharacterEncoding();
    if (encoding == null)
        encoding = "ISO-8859-1";

    // Parse any parameters specified in the query string
    String queryString = getQueryString();
    try {
        RequestUtil.parseParameters(results, queryString, encoding);
    }
    catch (UnsupportedEncodingException e) {
        ;
    }

    // Parse any parameters specified in the input stream
    String contentType = getContentType();
    if (contentType == null)
        contentType = "";
    int semicolon = contentType.indexOf(';');
    if (semicolon >= 0) {
        contentType = contentType.substring(0, semicolon).trim();
    }
    else {
        contentType = contentType.trim();
    }
    if ("POST".equals(getMethod()) && (getContentLength() > 0)
        && "application/x-www-form-urlencoded".equals(contentType)) {
        try {
        int max = getContentLength();
        int len = 0;
        byte buf[] = new byte[getContentLength()];
        ServletInputStream is = getInputStream();
        while (len < max) {
            int next = is.read(buf, len, max - len);
            if (next < 0 ) {
            break;
            }
            len += next;
        }
        is.close();
        if (len < max) {
            throw new RuntimeException("Content length mismatch");
        }
        RequestUtil.parseParameters(results, buf, encoding);
        }
        catch (UnsupportedEncodingException ue) {
        ;
        }
        catch (IOException e) {
        throw new RuntimeException("Content read fail");
        }
    }

    // Store the final results
    results.setLocked(true);
    parsed = true;
    parameters = results;
    }
```

在 GET 类型的 request 中，所有的 parameter 都是存在 URL 中的，POST 类型的 request，parameter 是存在 body 中的。解析的 parameter 会存在特殊的 Map 中，这个 map 不允许改变存放的 parameter 的值。对应的实现是 org.apache.catalina.util.ParameterMap. 看了一下具体的实现类代码，其实就是一个 HashMap, 最大的特点是他新加了一个 locked 的 boolean 属性，在增删改的时候都会先检查一下这个 flag 如果 flag 为 false 则抛异常。

### Creating a HttpResponse Object

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