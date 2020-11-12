---
title: Springboot note
date: 2020-11-05 21:42:07
categories:
- 编程
tags:
- java
- spring
- springboot
---

Springboot 学习笔记，核心**自动配置**

社区版的 Idea 少了一些配置，从网上下下来的 initializr 直接倒入的还一些配置可能失效，可以看文件前缀判断

## HelloWorld web mode

1. 访问 [Initializr](https://start.spring.io/) 定制项目， dependencies 选 Spring Web 即可
2. 下载项目 jar 文件并解压，使用 Idea import，构建项目
3. Springboot 的项目结构和 Springmvc 基本一样，在 HelloWorldApplication 同级目录下创建 controller 包并添加 controller 类
4. Springboot 项目默认集成 tomcat，直接运行 application class 即可启动服务器
5. 该 tomcat 应该是优化过的，启动速度飞起，访问 `http://localhost:8080/hello` 可以看到返回 hello 字符串
6. 查看 idea 右边的 maven tab, 在 Lifecycle 下双击执行 package 打包项目 
7. 可以看到打包好的项目 `Building jar: ...\helloworld\target\helloworld-0.0.1-SNAPSHOT.jar`
8. 到对应的路径下，cmd 窗口输入 `java -jar helloworld-0.0.1-SNAPSHOT.jar` 可以直接启动

```java
@RestController
public class TestController {
    @GetMapping("/hello")
    public String hello() {
        return "hello";
    }
}
```

## HelloWorld idea mode

1. Idea -> file -> new -> project -> Spring Initialzr
2. 填入必要信息，可以修改 package 简化路径
3. 其他步骤和上面的练习一样

彩蛋：banner 替换，在 resources 下新建 banner.txt 文件，替换终端启动图标

## Autowired 替代方案

方案一 可以通过把注解放到对应的 setter 方法上绕过

```java
InitBean initBean;

@Autowired
public void setInitBean(InitBean initBean) {
    this.initBean = initBean;
}
```

方案二 放入构造函数中自动识别

```java
public class InitTraceSourceEventListener implements ApplicationListener<ApplicationReadyEvent> {
    InitBean bean;

    public InitTraceSourceEventListener(InitBean bean) {
        this.bean = bean;
    }
}
```

方案三 用 `@Resource` 代替

```java
@Resource
InitBean bean;
```

或者最粗暴的: Settings -> Editor -> Code Style -> Inspections -> Spring Core -> Code -> Field injection warning 选项 disable 掉

## 自动配置原理初探

1. 核心依赖都在父工程中
2. 写入依赖时不需要指定版本，父类 pom 已经管理了

**启动器**, 即 Springboot 的启动场景

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter</artifactId>
</dependency>
```

各种场景有对应的启动器，比如 `spring-boot-starter-web`, 开发时只需要找到对应的启动器即可

**主程序**

这部分需要 实操+完善 好几遍才行，流程有点长

```java
//@SpringBootApplication 标注这个类是一个 springboot 应用
@SpringBootApplication
public class Hello02Application {

    public static void main(String[] args) {
        // 启动应用
        SpringApplication.run(Hello02Application.class, args);
    }
}
```

主要注解关系

```txt
@SpringBootApplication 
    @SpringBootConfiguration - springboot 配置
        @Configuration - spring 配置类
            @Component - 说明时一个 spring 组件
    @EnableAutoConfiguration
        @AutoConfigurationPackage
            @Import(AutoConfigurationPackages.Registrar.class)
```

结论：Springboot 所有自动配置都是在启动的时候扫描并加载(spring.factories). 所有的自动配置配都在里面，但不一定生效。要判断条件是否成立，只有导入了对应的启动器(starter), 才会生效。

spring-boot-autoconfiguration.jar 包含所有的配置

SpringApplication.run() 完了可以深入了解一下，不过，前面的自动装备更重要

## YAML 给属性赋值

* yaml 和 properties 是可以共存的
* 共存时 properties 的优先级要高于 yaml
* yaml 的后缀可以是 yaml 或 yml 都可以生效

yaml 格式：

```yaml
server:
    port: 8081
```

可以直接给对象赋值

### 基本赋值用法

通过添加 @Value 实现

```java
import org.springframework.beans.factory.annotation.Value;
import org.springframework.stereotype.Component;

@Component
@Data
@NoArgsConstructor
@AllArgsConstructor
public class Dog {
    @Value("旺财")
    private String name;
    @Value("3")
    private int age;
}

@SpringBootTest
class DogTest {
    @Autowired
    private Dog dog;

    @Test
    public void test() {
        System.out.println(dog);
    }
}

// output: Dog(name=旺财, age=3)
```

### yaml 配置属性

配置 pom 依赖

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-configuration-processor</artifactId>
    <optional>true</optional>
</dependency>
```

类添加 `@ConfigurationProperties` 注解

```java
@Component
@Data
@NoArgsConstructor
@AllArgsConstructor
@ConfigurationProperties(prefix = "person")
public class Person {
    private String name;
    private int age;
    private boolean isHappy;
    private Date birth;
    private Map<String, Object> maps;
    private List<Object> lists;
    private Dog dog;
}
```

添加 `application.yaml` 文件并设置属性

```yaml
person:
  name: jack
  age: 30
  isHappy: true
  birth: 2020/01/01
  maps: {k1: v1, K2: v2}
  lists: [1, 2, 3]
  dog:
    name: 旺财
    age: 2
```

```java
@SpringBootTest
class PersonTest {
    @Autowired
    private Person person;

    @Test
    public void test() {
        System.out.println(person);
    }
}
// output: Person(name=jack, age=30, isHappy=false, birth=Wed Jan 01 00:00:00 CST 2020, maps={k1=v1, K2=v2}, lists=[1, 2, 3], dog=Dog(name=旺财, age=2))
```

PS: yaml 还支持各种随机占位符，一元表达式等，可扩展性要更强

### 通过 properties 配置

缺点：表示起来比较冗余

```java
@Component
@Data
@NoArgsConstructor
@AllArgsConstructor
@PropertySource(value="classpath:application.properties")
public class Person {
    @Value("${name}")
    private String name;
    private int age;
    private boolean isHappy;
    private Date birth;
    private Map<String, Object> maps;
    private List<Object> lists;
    private Dog dog;
}
```

## JSR 303 校验

spring 自带的验证注解，添加之后可以再给 bean 赋值的时候带上校验效果

添加依赖

```xml
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-validation</artifactId>
</dependency>
```

数据配置

```yaml
mailbox:
  email: 123
```

添加 `@Validated`, `@Email` 等注解
```java
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.stereotype.Component;
import org.springframework.validation.annotation.Validated;

import javax.validation.constraints.Email;

@Component
@Data
@NoArgsConstructor
@AllArgsConstructor
@ConfigurationProperties(prefix = "mailbox")
@Validated
public class MailBox {

    @Email(message = "Email format is incorrect!!!")
    private String email;
}

// 测试用例
@SpringBootTest
class MailBoxTest {
    @Autowired
    private MailBox box;

    @Test
    public void test() {
        System.out.println(box);
    }
}
// 抛异常：org.springframework.boot.context.properties.bind.validation.BindValidationException
```

## 默认配置文件优先级

方式一：root/config > root/. > classpath:/config > classpath:/.

方式二： 新建 `application-xx.properties`, 再 default 中的配置文件中通过 `spring.profile.active=xx` 指定激活的配置

方式三：yaml + ---

```yaml
server:
    port: 8081
spring:
    profiles:
        active: dev

---
server:
    port: 8082
spring:
    profiles: dev

---
server:
    port: 8083
spring:
    profiles: test
```

## application.properties 中支持的属性源码中在哪里写的

1. SpringBoot 启动会加载大量的配置类
2. 我们看需要的功能有没有在 SpringBoot 默认写好的自动配置类当中
3. 再看这个配置类中到底配置了哪些组件
4. 给容器中自动皮欸之类添加组件的时候，会从 properties 类中获取某些属性

xxxAutoConfiguration: 自动配置类，给容器添加组件

xxxProperties：封装配置文件中相关属性

debug=true 可以查看配置详情

## 静态资源加载原理

分析一波 WebMvcAutoConfiuration.java
    -> webjars， web 相关的包封装成 Java 模式，但是不建议这么做

优先级： resources > script > public

首页定制 getIndexHtml()

template 文件夹下的内容需要使用模板引擎，添加 dependency + 注解

## thymeleaf

导入 starter

```java
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-thymeleaf</artifactId>
</dependency>
```

在 template 下新建页面文件 test.html，新建 controller 文件夹并创建 controller

```html
<!DOCTYPE html>
<html lang="en" xmlns:th="http://www.thymeleaf.org">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
<div th:text="${msg}"></div>
</body>
</html>
```

```java
@Controller
public class TestController {
    @RequestMapping("/hello")
    public String test(Model model) {
        model.addAttribute("msg", "hello, springboot");
        return "test";
    }
}
```

启动服务，访问 localhost:8080/hello 可以看到新建的页面

## MVC 配置原理

https://docs.spring.io/spring-boot/docs/2.1.6.RELEASE/reference/html/boot-features-developing-web-applications.html

自定义视图解析器 @Configuration + implement WebMvcConfigurer 接口

```java
// 定制功能只需要鞋各组件，然后交给 springboot，他会帮我们自动装配
// dispatchservlet
@Configuration
public class MyMvcConfig implements WebMvcConfigurer {

    // ViewResolver 实现了视图解析器的接口类，我们可以把它看作是退解析器
    @Bean
    public ViewResolver myViewResolver() {
        return new MyViewResolver();
    }

    public static class MyViewResolver implements ViewResolver {

        @Override
        public View resolveViewName(String viewName, Locale locale) throws Exception {
            return null;
        }
    }
}
```

在 DispatcherServlet 的 doDispatch 方法打上断点，在 this 下的 viewResolver 变量中可以看到自定义的解析器

@Configuration 修饰的类可以帮你扩展功能