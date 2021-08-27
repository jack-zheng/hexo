---
title: Ex01 创建一个简易的 Server
date: 2021-07-05 12:51:24
categories:
- Tomcat
tags:
- How Tomcat Works
---

## Introduce

Basically there are three things that a servlet container does to service a request for a servlet:

* 创建 request 对象并塞入后续要用到的信息，比如 parameters，cookies 等。request 是 javax.servlet(.http).ServletRequest 的一个实现
* 创建一个 response 对象返回给 web client. response 是 javax.servlet(.http).ServletResponse 的一个实现
* 调用 service 方法，service 中接受 request 的参数，处理结果塞入 response 中

### Catalina Block Diagram

Catalina 很复杂，但是他的设计很优雅，采用模块化的思想。主要可以分为两部分 Connector 和 Container，关系如下

{% plantuml %}
@startuml
Connector "*"*-"1" Container
{% endplantuml %}

connector 主要作用是构建 request/response 并传递给 container 处理，这里只是简化的模型。container 除了处理 request 还有很多东西需要做，比如加载 servlet，更新 session 等。

## A Simple Web Server

> **Chapter1** starts this book by presenting a simple HTTP server.
> To build a working HTTP server, you need to know the internal workings of two classes in the java.net package: Socket and ServerSocket.
> There is sufficient background information in this chapter about these two classes for you to understand how the accompanying application works.


第一个练习的目标，创建一个简单的 web server. 服务启动后，浏览器输入地址，server 返回请求的静态资源.

逻辑层面上来说模型可以像下面这样展示，但是代码层面上却不行。

{% plantuml %}
node Client
node Server

Client -> Server : "request"
Server -> Client : "resp"
{% endplantuml %}

按照上面的图示，难道 client 是直接 new 一个 request 和 web server 进行交互吗？难道 web server 会 new 一个 response 发送给 client 吗? 非也。模型画成下面的样子应该更合适

{% plantuml %}
left to right direction
skinparam packageStyle rectangle

(Client)
rectangle "Web Server" {
    cloud "service" {
        (req)
        (resp)
    }

    (Client) <--> service : socket
}
{% endplantuml %}

Client 和 Web Server 之间通过 socket 进行交互。在 server 内部，会将 socket 分化为 input 和 output 两个 IO 流，分别对应读取 Client 发送的数据和发送给 Client 相应

项目目录设置如下

```txt
how-tomcat-works
├── ex01
│   ├── pom.xml
│   └── src
│       └── ex01
│           ├── HttpServer.java
│           ├── Request.java
│           └── Response.java
├── pom.xml
└── webroot
    ├── images
    │   └── logo.gif
    └── index.html
```

HttpServer 表示服务器类，内部实现主要依赖于 ServerSocket 这个 java.net 包下的类，他在绑定本地端口并死循环等待 Client 端的 socket 访问

```java
public class HttpServer {
    // 指定静态资源目录
    public static final String WEB_ROOT = System.getProperty("user.dir") + File.separator + "webroot";
    // shutdown command
    private static final String SHUTDOWN_COMMAND = "/SHUTDOWN";
    // the shutdown command received
    private boolean shutdown = false;

    public static void main(String[] args) {
        HttpServer server = new HttpServer();
        server.await();
    }

    public void await() {
        ServerSocket serverSocket = null;
        int port = 8080;
        // 绑定端口和地址
        serverSocket = new ServerSocket(port, 1, InetAddress.getByName("127.0.0.1"));
        // 死循环等待 request
        while (!shutdown) {
            Socket socket = null;
            InputStream input = null;
            OutputStream output = null;

            // 拿到 socket
            socket = serverSocket.accept();
            // 分化 socket
            input = socket.getInputStream();
            output = socket.getOutputStream();
            // create Request object and parse
            Request request = new Request(input);
            request.parse();

            // create Response object
            Response response = new Response(output);
            response.setRequest(request);
            response.sendStaticResource();

            // Close the socket
            socket.close();
            // check if the previous URI is a shutdown command
            shutdown = request.getUri().equals(SHUTDOWN_COMMAND);
        }
    }
}
```

Request 类的主要职责是拿到 socket 的输入流，解析并提取 Client 发送过来的信息，这个例子中主要是提取请求的资源路径

```java
public class Request {
    private InputStream input;
    private String uri;

    public Request(InputStream input) {
        this.input = input;
    }

    public void parse() {
        // Read a set of characters from the socket
        StringBuilder request = new StringBuilder(2048);
        int i;
        byte[] buffer = new byte[2048];
        i = input.read(buffer);
        
        for (int j = 0; j < i; j++) {
            request.append((char) buffer[j]);
        }
        // 打印请求信息
        System.out.print(request);
        uri = parseUri(request.toString());
    }

    // 解析请求，拿到请求的路径
    private String parseUri(String requestString) {
        int index1, index2;
        index1 = requestString.indexOf(' ');
        if (index1 != -1) {
            index2 = requestString.indexOf(' ', index1 + 1);
            if (index2 > index1)
                return requestString.substring(index1 + 1, index2);
        }
        return null;
    }

    public String getUri() {
        return uri;
    }
}
```

Response 负责向 Client 返回信息

```java
public class Response {
    private static final int BUFFER_SIZE = 1024;
    Request request;
    OutputStream output;

    public Response(OutputStream output) {
        this.output = output;
    }

    public void setRequest(Request request) {
        this.request = request;
    }

    public void sendStaticResource() throws IOException {
        byte[] bytes = new byte[BUFFER_SIZE];
        FileInputStream fis = null;
        try {
            File file = new File(HttpServer.WEB_ROOT, request.getUri());
            if (file.exists()) {
                // 书上这段是没有的，当时 browser 比较老，不检测格式，现在浏览器不加这个页面会不能显示，curl 测试倒是不受影响
                String header = "HTTP/1.1 200 OK\r\n" +
                        "Content-Type: text/html\r\n" +
                        "\r\n";
                output.write(header.getBytes());

                fis = new FileInputStream(file);
                int ch = fis.read(bytes, 0, BUFFER_SIZE);
                while (ch != -1) {
                    output.write(bytes, 0, ch);
                    ch = fis.read(bytes, 0, BUFFER_SIZE);
                }
            } else {
                // file not found
                String errorMessage = "HTTP/1.1 404 File Not Found\r\n"
                        + "Content-Type: text/html\r\n"
                        + "Content-Length: 23\r\n" + "\r\n"
                        + "<h1>File Not Found</h1>";
                output.write(errorMessage.getBytes());
            }
        } catch (Exception e) {
            // thrown if cannot instantiate a File object
            System.out.println(e.toString());
        } finally {
            if (fis != null) {
                fis.close();
            }
        }
    }
}
```

结合计算机网络的知识做下分层的整理：Tomcat 处理的问题属于 Application 层，HTTP 规范是在处理 socket 的时候体现的. 那么写 reponse 的时候，需要特殊指定 HTTP 版本，header 等信息，这些参数都是服务器端指定的。