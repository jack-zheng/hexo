---
title: Ex02 创建一个简易的 Servlet Container
date: 2021-07-09 19:35:15
categories:
- Tomcat
tags:
- How Tomcat Works
---

> **Chapter2** explains how simple servlet containers work. 
> This chapter comes with two servlet container applications that can service requests for static resources as well as very simple servlets. 
> In particular, you will learn how you can create request and response objects and pass them to the requested servlet's service method. 
> There is also a servlet that can be run inside the servlet containers and that you can invoke from a web browser.


servlet container 会为 servlet 做如下事情

* 第一次调用 servlet 时，加载 servlet 并执行 init 方法
* 接收 server 创建的 ServletRequest 和 ServletResponse 对象
* 调用 service 方法，传入前面声明的两个对象
* servlet 生命周期结束时，调用 destroy 方法

## 练习01

ex02 的第一个 demo 并不是完整功能的 server，没有实现 init 和 destroy 的功能，它主要关注以下几个方面

* 等待请求
* 构建 ServletRequest 和 ServletResponse 对象
* 如果请求静态资源，调用 StaticResourceProcessor 相关方法
* 如果请求 servlet， 加载 servlet 并调用 service 方法

![Figure 2.1](F2.1.png)

主要类介绍

HttpServer1 即服务器主体类，通过 while 死循环等待连接

```java
public class HttpServer1 {
    public static void main(String[] args) {
        HttpServer1 server = new HttpServer1();
        server.await();
    }

    public void await() {
        ServerSocket serverSocket = null;
        int port = 8080;
        serverSocket = new ServerSocket(port, 1, InetAddress.getByName("127.0.0.1"));

        // Loop waiting for a request
        while (!shutdown) {
            Socket socket = serverSocket.accept();
            InputStream input = socket.getInputStream();
            OutputStream output = socket.getOutputStream();

            // create Request object and parse
            Request request = new Request(input);
            request.parse();

            // create Response object
            Response response = new Response(output);
            response.setRequest(request);

            // check if this is a request for a servlet or a static resource
            // a request for a servlet begins with "/servlet/"
            if (request.getUri().startsWith("/servlet/")) {
                ServletProcessor1 processor = new ServletProcessor1();
                processor.process(request, response);
            } else {
                StaticResourceProcessor processor = new StaticResourceProcessor();
                processor.process(request, response);
            }

            // Close the socket
            socket.close();
            //check if the previous URI is a shutdown command
            shutdown = request.getUri().equals(SHUTDOWN_COMMAND);
        }
    }
}
```

Request 和 Response 与 ex01 唯一的不同是，它们分别实现了 servlet 包中定义的 ServletReqeust 和 ServletResponse 的接口，需要实现很多额外的方法，但是本次练习中基本都是使用默认实现，不做介绍

Response 稍微多了一点逻辑，作者将处理静态页面的逻辑放到了 response 中，我觉得这个不好，应该写到 StaticResourceProcessor 中去的，和 ServletProcessor1 相对应，职责更分明才好

```java
// Response
public class Response implements ServletResponse {
    // constructor ...

    /* This method is used to serve a static page */
    public void sendStaticResource() throws IOException {
        // write response header
        String header = "HTTP/1.1 200 OK\r\n" +
                "Content-Type: text/html\r\n" +
                "\r\n";
        output.write(header.getBytes());

        byte[] bytes = new byte[BUFFER_SIZE];
        FileInputStream fis = null;
        try {
            /* request.getUri has been replaced by request.getRequestURI */
            File file = new File(Constants.WEB_ROOT, request.getUri());
            fis = new FileInputStream(file);
      /*
         HTTP Response = Status-Line
           *(( general-header | response-header | entity-header ) CRLF)
           CRLF
           [ message-body ]
         Status-Line = HTTP-Version SP Status-Code SP Reason-Phrase CRLF
      */
            int ch = fis.read(bytes, 0, BUFFER_SIZE);
            while (ch != -1) {
                output.write(bytes, 0, ch);
                ch = fis.read(bytes, 0, BUFFER_SIZE);
            }
        } catch (FileNotFoundException e) {
            String errorMessage = "HTTP/1.1 404 File Not Found\r\n" +
                    "Content-Type: text/html\r\n" +
                    "Content-Length: 23\r\n" +
                    "\r\n" +
                    "<h1>File Not Found</h1>";
            output.write(errorMessage.getBytes());
        } finally {
            if (fis != null)
                fis.close();
        }
    }
    // other required methods defined in ServletResponse
}
```

StaticResourceProcessor 很简单的一个类，其实就是提供了一个 process 方法，但是实现只是调用了 response 中定义的方法

```java
public class StaticResourceProcessor {
    public void process(Request request, Response response) {
        response.sendStaticResource();
    }
}
```

ServletProcessor1 即处理 servlet 的类，作者参考了 tomcat 中的处理方式，通过 url 解析出来请求的 servlet 名字，再通过 URLClassLoader 做类加载，最后生成实体类并调用 service 方法

```java
public class ServletProcessor1 {
    public void process(Request request, Response response) {
        // 解析 servlet 名字
        String uri = request.getUri();
        String servletName = uri.substring(uri.lastIndexOf("/") + 1);

        // create a URLClassLoader
        URL[] urls = new URL[1];
        URLStreamHandler streamHandler = null;
        File classPath = new File(Constants.WEB_ROOT);

        // the forming of repository is taken from the createClassLoader
        // method in org.apache.catalina.startup.ClassLoaderFactory
        String repository = (new URL("file", null, classPath.getCanonicalPath() + File.separator)).toString();

        // the code for forming the URL is taken from the addRepository
        // method in org.apache.catalina.loader.StandardClassLoader class.
        urls[0] = new URL(null, repository, streamHandler);
        URLClassLoader loader = new URLClassLoader(urls);
       

        Class<?> myClass = myClass = loader.loadClass(servletName);
        Servlet servlet = (Servlet) myClass.newInstance();
        servlet.service(request, response);
    }
}
```

启动 server 通过终端访问测试地址. 现在的浏览器对 response 格式要求都挺严格的，默认的代码访问会因为 header 不全被 block，通过终端访问则没这么多问题。

```txt
curl 'http://localhost:8080/servlet/PrimitiveServlet'
Hello. Roses are red.

curl 'http://localhost:8080/index.html'
<html>
<head>
    <title>Welcome to BrainySoftware</title>
</head>
<body>
<img src="images/logo.gif">
<br>
Welcome to BrainySoftware.
</body>
</html>
```

## 练习 02

这一节练习主要解决一个安全问题-恶意强转。上面的例子中，在 ServletProcessor1 中，我们直接将 request/response 对象作为参数传入 service 方法中，如果使用方知道我们对应的实现，就可以做对象强转并调用 public 方法，比如调用解析静态资源的 sendStaticResource() 方法，这是不安全的，所以我们通过 Facade 的模式，将 request/response 作为内部私有变量持有并调用，以达到隐藏实现的目的。

Facade 和 request/response 的关系

{% plantuml %}
interface javax.servlet.ServletRequest
javax.servlet.ServletRequest <|.. Request
javax.servlet.ServletRequest <|.. RequestFacade

interface javax.servlet.ServletResponse
javax.servlet.ServletResponse <|.. Response
javax.servlet.ServletResponse <|.. ResponseFacade
{% endplantuml %}

ServletProcessor1 和 ServletProcessor2 最大的区别只有下面这些

```java
    // ...
    Servlet servlet = null;
    RequestFacade requestFacade = new RequestFacade(request);
    ResponseFacade responseFacade = new ResponseFacade(response);
    try {
        servlet = (Servlet) myClass.newInstance();
        servlet.service((ServletRequest) requestFacade, (ServletResponse) responseFacade);
    } catch (Exception e) {
        System.out.println(e.toString());
    } catch (Throwable e) {
        System.out.println(e.toString());
    }
    // ...
```

## 思考题

> Servlet 到底是什么？

先看这个英文单词的意思：小服务程序。Javaweb 三个组建之一(其二为 Listener 和 Filter)，定义了一类特殊的 Java class 可以在访问特定的地址时执行特定的业务逻辑。我觉得理解到这里就足够了。

## 遇到的问题

> 启动 server 之后访问 URL 报错 `Exception in thread "main" java.lang.NoClassDefFoundError: javax/servlet/ServletRequest`。检查了 pom.xml 什么都正常，Google 了一下，发现是 pom 中包的 scope 有问题.

maven 网站上 copy 时，scope 是 provided 需要改成 compile

* compile，缺省值，适用于所有阶段，会随着项目一起发布。 
* provided，类似compile，期望JDK、容器或使用者会提供这个依赖。
* runtime，只在运行时使用，如JDBC驱动，适用运行和测试阶段。 
* test，只在测试时使用，用于编译和运行测试代码。不会随项目发布。 
* system，类似provided，需要显式提供包含依赖的jar，Maven不会在Repository中查找它。

> PrimitiveServlet 访问异常

idea 抄过来的会带包名的，所以会出问题。从成功项目 copy 过来的不带，所以能 work

> Servlet 中的内容 browser 访问出问题

和之前静态页面的问题一样，不过 servlet 文件我拿到的源码就是 class 的不好改