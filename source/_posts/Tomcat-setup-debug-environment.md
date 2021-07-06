---
title: Tomcat setup debug environment
date: 2021-07-05 18:54:11
- How tomcat works
tags:
- setup
---

本来像原滋原味的 setup tomcat4/5 的环境的，但是找了一圈没现成资源，还是拿了别人已经搞过的 Tomcat 8 过来，先搞起来再说，有必要再找 4/5 版本的代码。

1. 访问[官网](https://tomcat.apache.org/download-80.cgi), 选择 Source Code Distributions 下的 zip 或者 tar.gz，下载。瞄了一眼 README 貌似两者的区别是 zip 是 CRLF 换行而 tar 是 LF 换行。应该对应 windows 和 Linux 系统。
2. 下载后得到一个 apache-tomcat-8.5.68-src.tar.gz 压缩包，解压它
3. 在解压后目录中新建 catalina-home 文件夹，并将源码中的 conf, webapps 文件夹拷贝进去。webapps 下的 example 文件夹删掉，不然后面启动会抛异常
4. 在 catalina-home 下新建四个文件夹：lib, temp, work, logs
5. 在源文件目录下新建 pom.xml 文件并添加依赖
6. 删除源文件中的 test 目录避免一些不必要的错误
7. 打开 Idea, 导入项目，run/debug 处配置启动参数如下
8. 修改 ContextConfig 文件，在 `webConfig();` 后添加 `context.addServletContainerInitializer(new JasperInitializer(), null);` 不然访问页面会抛 `org.apache.jasper.JasperException: java.lang.NullPointerException` 的异常
9. 点击 Idea 上的 run 测试启动，成功
   

```config
<!-- 启动参数 -->
main class: org.apache.catalina.startup.Bootstrap
vm options: 
-Dcatalina.home="/Users/i306454/IdeaProjects/apache-tomcat-8.5.68-src/catalina-home"
-Duser.language=en
-Duser.region=US
-Dfile.encoding=UTF-8
```

PS: Idea 2021.1.3 版本的 vm options 需要点击 Modify Options 自己添加，默认不显示

![application](application.png)

最终，项目的目录结构如下

```txt
.
└── apache-tomcat-8.5.68-src
    ├── catalina-home
    │   ├── conf
    │   ├── lib
    │   ├── logs
    │   ├── temp
    │   ├── webapps
    │   └── work
    └── pom.xml
```

PS: [Tomcat 5 源码地址](https://archive.apache.org/dist/tomcat/tomcat-5/)