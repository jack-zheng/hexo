---
title: SpringMVC 视频笔记
date: 2020-10-14 21:17:02
categories:
- Spring
tags:
- springmvc
- 视频笔记
---

[B 站狂神 SpringMVC 教程笔记](https://www.bilibili.com/video/BV1aE41167Tu?p=3)

## 01 Servlet review

### 准备工作 提前本地安装 Tomcat

1. 访问[官网](https://tomcat.apache.org/download-90.cgi)下载安装包
2. 点击 Binary Distributions 下的 `32-bit/64-bit Windows Service Installer (pgp, sha512)` 下载 exe 可执行文件
3. 点击，傻瓜式安装
4. 点击提示框，启动 访问 `http://localhost:8080/` 看到页面则安装成功

配置环境变量：通过上面的傻瓜式安装，Tomcat 默认安装在 `C:\Program Files\Apache Software Foundation\Tomcat 9.0` 这个路径下

我的电脑 -> 属性 -> 高级系统设置 -> 高级 -> 环境变量, 在系统变量中添加：

| 变量名        | 值                                                     |
| :------------ | :----------------------------------------------------- |
| TOMCAT_HOME   | C:\Program Files\Apache Software Foundation\Tomcat 9.0 |
| CATALINA_HOME | C:\Program Files\Apache Software Foundation\Tomcat 9.0 |

修改变量Path, 在原来的值后面添加 `;%TOMCAT_HOME%\bin;%CATALINA_HOME%\lib`

### 子项目创建

正常步骤建项目，创建 maven 子 module，然后 module 上邮件选中 Add Framework Support -> Web Application 来创建 web app 会比较省事。可以看到在目录中新增了名为 web 的文件夹

在 src 下新建一个测试用 servlet 文件

```java
public class HelloServlet extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        // 1. 获取前端参数
        String method = req.getParameter("method");
        if (method.equals("add")) {
            req.getSession().setAttribute("msg", "execute Add...");
        }

        if (method.equals("delete")) {
            req.getSession().setAttribute("msg", "execute Delete...");
        }
        // 2. 调用业务层
        // 3. 试图转发或重定向
        req.getRequestDispatcher("/WEB-INF/jsp/test.jsp").forward(req,resp);
    }

    @Override
    protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        doGet(req, resp);
    }
}
```

web -> WEB-INF 下新建 jsp 文件夹，创建 test.jsp

```jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>Title</title>
</head>
<body>

${msg}

</body>
</html>
```

修改 WEB-INF 下的 web.xml 配置路由

```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee http://xmlns.jcp.org/xml/ns/javaee/web-app_4_0.xsd"
         version="4.0">
    <servlet>
        <servlet-name>hello</servlet-name>
        <servlet-class>com.jzheng.servlet.HelloServlet</servlet-class>
    </servlet>
    
    <servlet-mapping>
        <servlet-name>hello</servlet-name>
        <url-pattern>/hello</url-pattern>
    </servlet-mapping>
</web-app>
```

工具栏 -> edit configuration -> 点击 + 号，选中 Tomcat 配置本地 tomcat， 点击 fix -> 启动服务器。访问 `http://localhost:8080/springmvn_01_servlet_war_exploded/hello?method=add` 可以看 msg 显示在页面上。

PS：应该是哪里配置有问题，视频上面直接访问 `http://localhost:8080/hello?method=add` 即可，回头看一下前面的 JavaWeb 项目应该就知道了，暂时没什么关系，无伤大雅

## SpringMVC start

PS: 在官方文档页面，修改 current 为其他版本可以访问老版本的文档,例如 `https://docs.spring.io/spring-framework/docs/4.3.24.RELEASE/spring-framework-reference/`

PPS: 这个可以在 Tomcat 配置页面的 Deployment tab 下，将 Application context 内容直接改为 `/` 即可

特点：

1. 轻量
2. 基于响应
3. 兼容 Spring
4. 约定优于配置
5. 功能强大
6. 简介灵活
7. 用的人多

1. 创建子项目，配置为 web app
2. Porject Structure -> Artifacts，选中项目 -> WEB-INF 下新建 lib 包手动把包导进去（idea 的bug）-> 点击 + 号 -> Library files 全选
3. resource 下新建 xml 文件，选择 Spring config 类型，可以自带配置信息
4. 配置 WEB-INF 下的 web.xml
5. 创建 Controller 添加业务逻辑
6. 配置启动 Tomcat，访问 URL 看结果

这部分主要是为了讲解 SpringMVC 的原理，真实环境都用注解开发，会方便很多。

## SpringMVC 注解版

1. 创建工程转化为 web app
2. 在 web 创建 jsp 目录，配置 web.xml 配置内容和之前完全一样
3. 配置 springmvc-config.xml, 指定注解扫描路径，handler 和视图解析器
4. 创建 controller 添加注解

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:mvc="http://www.springframework.org/schema/mvc"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/context https://www.springframework.org/schema/context/spring-context.xsd http://www.springframework.org/schema/mvc https://www.springframework.org/schema/mvc/spring-mvc.xsd">

    <!-- 自动扫包，让指定包下的注解生效，IOC 容器统一管理 -->
    <context:component-scan base-package="com.jzheng.controller"/>

    <!-- 让 spring MVC 不处理静态资源（.css .js .html...） -->
    <mvc:default-servlet-handler/>

    <!-- 代替 HandlerMapping 和 HandlerAdapter-->
    <mvc:annotation-driven/>

    <!-- 视图解析器: 模板引擎 Thymeleaf, Freemaker 等 -->
    <bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
        <property name="prefix" value="/WEB-INF/jsp/"/>
        <property name="suffix" value=".jsp"/>
    </bean>
</beans>
```

```java
@Controller
public class HelloController {

    // 指定 URL
    @RequestMapping("/hello")
    public String hello(Model model) {
        // 封装数据
        model.addAttribute("msg", "Hello, SpringMVC");
        return "hello"; // 被视图解析器处理
    }
}
```

## 04 回顾

回顾了两种添加 Controller 的方法，还有 RequestMapping 添加在 class 和 method 上的区别

### Restful 风格

* @PathVariable 配置变量
* @RequestMapping(value = "/add/{a}/{b}", method = RequestMethod.POST) 配置请求方式
* @GetMapping(value = "/add/{a}/{b}") 请求方式简写

### 专发和重定向

forward，redirect

### 接受前端参数

简单类型传递

```java
public String test(@RequestParam("username") String name, Model model) {}
```

复杂类型

```java
 // http://localhost:8080/t2?username=jack&id=jjjj&age=3
// 终端能打印出对象
@GetMapping("/t2")
public String complexType(User user) {
    System.out.println(user);
    return "test";
}
```

### 乱码

web 文件夹下添加测试用的 jsp

```jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>Title</title>
</head>
<body>

<form action="/e/t1" method="post">
    <input type="text" name="name">
    <input type="submit">
</form>

</body>
</html>
```

创建测试 controller

```java
@Controller
public class EncodingController {

    @PostMapping("/e/t1")
    public String test1(String name, Model model) {
        System.out.println("output: " + name);
        model.addAttribute("msg", name);
        return "test";
    }
}
```

访问 `localhost:8080/form.jsp` 输入中文，可以看到输出乱码。

解决方案：过滤器

#### 自建过滤器

新建 filter 文件夹，添加过滤器

```java
public class EncodingFilter implements Filter {
    @Override
    public void init(FilterConfig filterConfig) throws ServletException {

    }

    @Override
    public void doFilter(ServletRequest servletRequest, ServletResponse servletResponse, FilterChain filterChain) throws IOException, ServletException {
        servletRequest.setCharacterEncoding("utf-8");
        servletResponse.setCharacterEncoding("utf-8");

        filterChain.doFilter(servletRequest, servletResponse);
    }

    @Override
    public void destroy() {

    }
}
```

`web.xml` 下配置过滤器

```xml
<filter>
    <filter-name>encoding</filter-name>
    <filter-class>com.jzheng.filter.EncodingFilter</filter-class>
</filter>
<filter-mapping>
    <filter-name>encoding</filter-name>
    <url-pattern>/*</url-pattern>
</filter-mapping>
```

#### 框架自带过滤器

```xml
<!-- 框架自带的过滤器 -->
<filter>
    <filter-name>build_in_encoding_filter</filter-name>
    <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
    <init-param>
        <param-name>encoding</param-name>
        <param-value>utf-8</param-value>
    </init-param>
</filter>
<filter-mapping>
    <filter-name>build_in_encoding_filter</filter-name>
    <url-pattern>/*</url-pattern>
</filter-mapping>
```

### Json

简单介绍一下 js 对象和字符串的转化

```js
<script type="text/javascript">
    var user = {
        name: "jack",
        age: 3,
        gender: "男"
    };


    console.log(user);

    // js 对象转化为 json 对象
    var json  = JSON.stringify(user);
    console.log(json);

    // json 对象转化为 JavaScript 对象
    var obj = JSON.parse(json);
    console.log(obj);
</script>
```

### jackson

1. 引入 jackson-databind 包
2. 创建测试类 User

```java
@Data
@AllArgsConstructor
@NoArgsConstructor
public class User {
    private String name;
    private int age;
    private String gender;
}
```

```java
// UserController
@Controller
public class UserController {

    @RequestMapping(value="/j1", produces="application/json;charset=utf-8")
    @ResponseBody // 不走视图解析器，直接返回字符串
    public String json1() throws JsonProcessingException {
        // jackson - ObjectMapper
        ObjectMapper objectMapper = new ObjectMapper();

        // 创建一个对象
        User user = new User("杰克", 1, "man");

        String ret = objectMapper.writeValueAsString(user);
        return ret;
    }
}

// user.toString() 返回 User(name=Jack01, age=1, gender=man)
// 访问 /j1 输出：{"name":"Jack01","age":1,"gender":"man"}
```

结果中包含中文会乱码，这时可以配置 RequestMapping 注解也可以配置 springmvc 配置文件

```xml
<!--
    beans 头里面确认包含
    http://www.springframework.org/schema/mvc
    http://www.springframework.org/schema/mvc/spring-mvc.xsd
    不然会抛错：通配符的匹配很全面, 但无法找到元素 'mvc:annotation-driven' 的声明
-->
<!-- Jackson 乱码问题 -->
<mvc:annotation-driven>
    <mvc:message-converters>
        <bean class="org.springframework.http.converter.StringHttpMessageConverter">
            <constructor-arg value="UTF-8"/>
        </bean>
        <bean class="org.springframework.http.converter.json.MappingJackson2HttpMessageConverter">
            <property name="objectMapper">
                <bean class="org.springframework.http.converter.json.Jackson2ObjectMapperFactoryBean">
                    <property name="failOnEmptyBeans" value="false"/>
                </bean>
            </property>
        </bean>
    </mvc:message-converters>
</mvc:annotation-driven>
```

## SSM 整合

```sql
-- 建表
CREATE DATABASE ssmbuild;
USE ssmbuild;

CREATE TABLE `books`(
`bookID` INT NOT NULL AUTO_INCREMENT COMMENT '书id',
`bookName` VARCHAR(100) NOT NULL COMMENT '书名',
`bookCounts` INT NOT NULL COMMENT '数量',
`detail` VARCHAR(200) NOT NULL COMMENT '描述',
KEY `bookID`(`bookID`)
)ENGINE=INNODB DEFAULT CHARSET=utf8;

INSERT INTO `books`(`bookID`,`bookName`,`bookCounts`,`detail`)VALUES
(1,'Java',1,'从入门到放弃'),
(2,'MySQL',10,'从删库到跑路'),
(3,'Linux',5,'从进门到进牢');
```

#### Issues

启动报错 `一个或多个筛选器启动失败。完整的详细信息将在相应的容器日志文件中找到`, 这个是适配 web support 的时候没有添加 lib 包导致的

Tomcat 下的 catalina.properties 修改了配置 'tomcat.util.scan.StandardJarScanFilter.jarsToSkip=*.jar' 导致 jstl 解析出问题抛异常 `无法在web.xml或使用此应用程序部署的jar文件中解析绝对uri` 改回到默认配置，修复。花了2个小时排错，之前告诉我这个该法的人真想把它拖出去枪毙18遍！参考 [cnblog](https://www.cnblogs.com/tioxy/p/13291574.html)

## Ajax

## 拦截器

拦截器之访问 controller 方法，不会拦截静态资源（js 等）

项目突然坏了，干！！！ 关了，最后三讲直接云上课看看完了