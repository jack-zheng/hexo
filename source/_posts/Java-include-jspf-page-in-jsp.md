---
title: JSP 中 include jspf
date: 2021-07-09 13:56:04
categories:
- java
tags:
- jsp
- jspf
---

问题：

* 父 JSP 页面中声明了一个变量，子 JSPF 文件中不显示的声明能直接使用这个变量吗
* 上面的情况如果是子 JSP 又如何

## 实验

新建一个 servlet 在 request 中传入 name 属性，然后 forward 到 parent.jsp 页面中。parent 页面包含三个子页面，分别是 sub.jspf, sub2.jsp 和 sub3.jsp. 前两个通过 `<%@ include file="xxx" %>` 引入，sub3.jsp 通过 `<jsp:include page="xxx"/>` 引入。目录结构如下

```txt
.
├── java
│   └── com
│       └── jzheng
│           └── servlet
│               └── ParentServlet.java
└── webapp
    ├── WEB-INF
    │   └── web.xml
    ├── parent.jsp
    ├── sub.jspf
    ├── sub2.jsp
    └── sub3.jsp
```

代码实现如下

```java
public class ParentServlet extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        req.setAttribute("name", "jack");
        getServletContext().getRequestDispatcher("/parent.jsp").forward(req, resp);
    }
}
```

parent.jsp 页面取得 request 中的属性，并重命名为 myname

```jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>Title</title>
</head>
<body>

<%
    String myname = (String)request.getAttribute("name");
%>

<hr>
<h1> include jspf file </h1>
<%@ include file="sub.jspf" %>

<hr>
<h1> include jsp file </h1>
<%@ include file="sub2.jsp" %>

<hr>
<h1> jsp:include jsp file </h1>
<jsp:include page="sub3.jsp"/>

</body>
</html>
```

sub 页面内容如下

```jsp
<!-- sub.jsp -->
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<h2> Name: <%= myname%> <h2/>

<!-- sub2.jsp -->
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<h2> Name: <%=myname%> </h2>

<!-- sub3.jsp -->
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<h2>Name: ${name}</h2>
```

PS: 这里插入一个语法点，`<%= myname%>` 这种语法是结合 `String myname = (String)request.getAttribute("name");` 只有 jsp 中声明的变量可以这么用，其实这种写法有点累赘，可以通过 EL 表示式 `${name}` 直接从内置对象中取值。

配置 web.xml

```xml
<servlet>
    <servlet-name>ParentServlet</servlet-name>
    <servlet-class>com.jzheng.servlet.ParentServlet</servlet-class>
</servlet>
<servlet-mapping>
    <servlet-name>ParentServlet</servlet-name>
    <url-pattern>/parent</url-pattern>
</servlet-mapping>
```

启动服务器访问 `/parent` 可以看到如下结果, 几种方式都能 work.

![parent](parent.png)

## 深入理解

查看编译生成的 JSP 文件，可以看到 parent.jsp 和 sub3.jsp 被编译成了 java/class, sub.jsp 和 sub2.jsp 没有。原因是 `<%@ include file="xxx" %>` 会将页面直接整合到父页面中，而 `<jsp:include page="xxx"/>` 则是通过 request 转发达到这个效果的。查看 parent_jsp.java 文件可以更清晰一点. sub 页面处理部分已表明。所以 `<%@ include file="xxx" %>` 中直接使用父页面定义的变量是可以，这种做法更像是定义了一写通用脚本做包含。但是这些变量都会爆红，非常的不爽

```java
response.setContentType("text/html;charset=UTF-8");
pageContext = _jspxFactory.getPageContext(this, request, response,
        null, true, 8192, true);
_jspx_page_context = pageContext;
application = pageContext.getServletContext();
config = pageContext.getServletConfig();
session = pageContext.getSession();
out = pageContext.getOut();
_jspx_out = out;

out.write("\n");
out.write("\n");
out.write("<html>\n");
out.write("<head>\n");
out.write("    <title>Title</title>\n");
out.write("</head>\n");
out.write("<body>\n");
out.write("\n");

String myname = (String)request.getAttribute("name");

out.write("\n");
out.write("\n");
out.write("<hr>\n");
out.write("<h1> include jspf file </h1>\n"); // <---------- sub.jsp part
out.write('\n');
out.write('\n');
out.write("\n");
out.write("<h2> Name: ");
out.print( myname);
out.write(" <h2/>");
out.write("\n");
out.write("\n");
out.write("<hr>\n");
out.write("<h1> include jsp file </h1>\n"); // <---------- sub2.jsp part
out.write("\n");
out.write("<h2> Name: ");
out.print(myname);
out.write(" </h2>\n");
out.write("\n");
out.write("\n");
out.write("<hr>\n");
out.write("<h1> jsp:include jsp file </h1>\n"); // <---------- sub3.jsp part
org.apache.jasper.runtime.JspRuntimeLibrary.include(request, response, "sub3.jsp", out, false);
out.write("\n");
out.write("\n");
out.write("</body>\n");
out.write("</html>\n");
```