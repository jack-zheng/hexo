---
title: forward Vs redirect
date: 2021-07-09 11:00:04
categories:
- java
tags:
- servlet
- forward
- redirect
---

问题：

* forward 和 redirect 时 request 传递的区别
* forward 和 redirect 时 url 传递的区别

## 问: forward 和 redirect 时 request 传递的区别

forward 可以将 request 传递下去，而 redirect 不能。其实写了 code 之后才意识到，forward 的传递性是因为它直接把之前的 request 当参数传递了，当然是一致的。而 redirect 是不带 request 参数的。

新建一个 ForwardServlet 在这个 servlet 中我们向 request 中设置 name 属性，然后 forward 到 ForwardedServlet. 在 ForwardServlet 中打印处之前设置的属性

```java
public class ForwardServlet extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        req.setAttribute("name", "jack");
        RequestDispatcher dispatcher = getServletContext().getRequestDispatcher("/forwarded");
        dispatcher.forward(req, resp);
    }
}

public class ForwardedServlet extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) {
        System.out.println("Name in request: " + req.getAttribute("name"));
    }
}
```

在 web.xml 中配置 mapping 关系

```xml
<servlet>
    <servlet-name>Forward</servlet-name>
    <servlet-class>com.jzheng.servlet.ForwardServlet</servlet-class>
</servlet>
<servlet-mapping>
    <servlet-name>Forward</servlet-name>
    <url-pattern>/forward</url-pattern>
</servlet-mapping>

<servlet>
    <servlet-name>Forwarded</servlet-name>
    <servlet-class>com.jzheng.servlet.ForwardedServlet</servlet-class>
</servlet>
<servlet-mapping>
    <servlet-name>Forwarded</servlet-name>
    <url-pattern>/forwarded</url-pattern>
</servlet-mapping>
```

name 在终端正确显示。其实光看 dispatch 部分的代码应该就有数了 `dispatcher.forward(req, resp);` 会将 request 传递下去，能拿到也不奇怪

同样的思路我们设计一个 redirect 的测试。新建 RedirectServlet 并在 request 对象中设置 name 属性，接着 redirect 到 RedirectedServlet 并打印 name

```java
public class RedirectServlet extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws IOException {
        req.setAttribute("name", "jack");
        System.out.println("context path: " + req.getContextPath());
        resp.sendRedirect(req.getContextPath() + "/redirected");
    }
}

public class RedirectedServlet extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) {
        System.out.println("Name in request: " + req.getAttribute("name"));
    }
}
```

name 为 null. 原因也很简单 `resp.sendRedirect(req.getContextPath() + "/redirected");` 并没有传递 request 参数。

## 问: forward 和 redirect 时 url 传递的区别

forward 方式不会改变 URL，redirect 会变。上面的实验中，我们访问 `/redirect` 时，最终浏览器显示的是 `/redirected`