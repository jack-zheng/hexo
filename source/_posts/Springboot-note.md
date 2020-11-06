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

Springboot 学习笔记

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