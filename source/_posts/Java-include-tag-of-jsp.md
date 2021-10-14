---
title: JSP 页面中 include 的自页面是否能访问父业页面的 request
date: 2021-07-09 12:45:04
categories:
- java
tags:
- servlet
- jsp
---

问题：

* jsp 中如果 include 了其他的 jsp 页面时 request 属性是否能传递到 include 的页面中

## 测试

新建一个 IndexServlet 为 request 设置 name 属性并 forward 到 index.jsp. index.jsp 包含一个子页面 mypage.jsp 这个页面会拿尝试那 index.jsp 中的 request 属性并显示

```java
public class IndexServlet extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        req.setAttribute("name", "jack");
        RequestDispatcher dispatcher = getServletContext().getRequestDispatcher("/index.jsp");
        dispatcher.forward(req, resp);
    }
}
```

jsp 页面代码如下

```jsp
<!-- index.jsp -->
<html>
<body>
<h2>Hello World!</h2>

<jsp:include page="mypage.jsp"/>
</body>
</html>

<!-- mypage.jsp -->
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<h2>Name: <%=request.getAttribute("name")%></h2>
```

配置 web.xml 指定路由

```xml
<servlet>
    <servlet-name>IndexServlet</servlet-name>
    <servlet-class>com.jzheng.servlet.IndexServlet</servlet-class>
</servlet>
<servlet-mapping>
    <servlet-name>IndexServlet</servlet-name>
    <url-pattern>/indexServlet</url-pattern>
</servlet-mapping>
```

启动 server 时跳出来的首页，name 为 null, 访问 `/indexServlet` 显示的首页 name 属性可以被拿到

```bash
curl 'http://localhost:8080/'
<html>
<body>
<h2>Hello World!</h2>


<h2>Name: null</h2>

</body>
</html>

curl 'http://localhost:8080/indexServlet'
<html>
<body>
<h2>Hello World!</h2>


<h2>Name: jack</h2>

</body>
</html>
```

## 探究

Mac 环境下配置的 Tomcat 服务，jsp 编译的页面会放在 `/Users/jack/Library/Caches/JetBrains/IntelliJIdea2021.1/tomcat/a0a1bd57-0f17-404d-9ff8-9f185e6d4d97/work/Catalina/localhost/ROOT/org/apache/jsp` 这个路径下

```txt
.
├── index_jsp.class
├── index_jsp.java
├── mypage_jsp.class
└── mypage_jsp.java
```

index_jsp.java 和 mypage_jsp.java 核心代码显示如下

```java
// index_jsp.java
response.setContentType("text/html");
pageContext = _jspxFactory.getPageContext(this, request, response, null, true, 8192, true);
_jspx_page_context = pageContext;
application = pageContext.getServletContext();
config = pageContext.getServletConfig();
session = pageContext.getSession();
out = pageContext.getOut();
_jspx_out = out;

out.write("<html>\n");
out.write("<body>\n");
out.write("<h2>Hello World!</h2>\n");
out.write("\n");
org.apache.jasper.runtime.JspRuntimeLibrary.include(request, response, "mypage.jsp", out, false);
out.write("\n");
out.write("</body>\n");
out.write("</html>\n");

// mypage_jsp.java
response.setContentType("text/html;charset=UTF-8");
pageContext = _jspxFactory.getPageContext(this, request, response, null, true, 8192, true);
_jspx_page_context = pageContext;
application = pageContext.getServletContext();
config = pageContext.getServletConfig();
session = pageContext.getSession();
out = pageContext.getOut();
_jspx_out = out;

out.write("\n");
out.write("<h2>Name: ");
out.print(request.getAttribute("name"));
out.write("</h2>\n");
```

可以看到父页面中通过 `org.apache.jasper.runtime.JspRuntimeLibrary.include(request, response, "mypage.jsp", out, false);` 包含页面，这个过程中是会把 request 当成参数传递的，自然在自页面中是能访问到这个对象的。

PS: 在配置 Tomcat 服务器时有两种 type, Tomcat 和 TomEE 之前选错了 TomEE 起不来 server （；￣ェ￣）
