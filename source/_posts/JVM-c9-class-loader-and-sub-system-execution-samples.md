---
title: JVM c9 class loader and sub system execution samples
date: 2021-09-10 10:32:15
categories:
- JVM
tags:
- 类加载
---

代码编译的结果从本地机器码转为字节码，是存储格式发展的一小步，却是编程语言发展的一大步。

## 9.1 概述

在 Class 文件格式与执行引擎这部分里，用户的程序能直接参与的内容并不多，Class 以何种格式存储，类型和施加在，如何连接，以及虚拟机如何执行字节码指令等都是由虚拟机直接控制的行为，用户程序无法对其进行改变。能操作的，主要是字节码生成与类加载两个部分的功能，但仅仅在如何处理这两点上，就已经出现了许多值得欣赏借鉴的思路，这些思路后来成为很多常用功能和程序实现的基础。

## 9.2 案例分析

四个案例，关于类加载和字节码各两个。

### 9.2.1 Tomcat: 正统的类加载架构

主流的 Java Web 服务器，如 Tomcat, Jetty 等都实现了自己定义的类加载器，而且还不止一个。因为一个功能健全的 Web 服务器，都要解决如下这些问题：

* 部署在同一个服务器上的两个 web 应用程序所使用的 Java 类库可以实现互相隔离。两个不同应用可能依赖同一个第三方类库的不同版本，不能要求每个类库在一个服务器中只能有一份，服务器应该能够保证两个独立应用程序的类库可以互相独立使用。
* 部署在同一个服务器上的两个 web 应用所使用的 Java 类库可以互相共享。与前一个相反，但很常见，如用户可能有 10 个使用 Spring 的应用部署在统一台服务器，如果把 10 份 Spring 分别存在应用的隔离目录，将会很大的浪费资源。磁盘空间是其次，主要是良妃内存，很容易造成方法去过度膨胀的风险。
* 服务器需要尽可能保证自身的安全不受部署的 Web 应用程序的影响。一般来说，给予安全考虑，服务器所使用的类库应该与程序类库相互独立。
* 只是 JSP 应用的 Web 服务器，十有八九都需要支持 HotSwap 功能。JSP 由于其纯文本特性，修改几率远大于第三方类库和自己的 Class 文件。ASP，PHP 和 JSP 这些网页应用也将修改后无需重启作为优势来看待，因此，主流 Web 服务器都会支持 JSP 生成类的热替换。

由于以上问题，不是 web 应用时，单独一个 ClassPath 就不能满足要求了，所以各种 web 服务器不约而同的提供了好几个不同还以的 ClassPath 路径供用户存放第三方类库。一般这些路径都以 lib 或 classes 命名。不同路径中的类库，具备不同的访问范围和服务对象，通常每个目录都会对应一个自定义类加载器去加载防止在里面的 Java 类库。下面以 Tomcat 为例，分析其规划。

Tomcat 目录结构中，有三组目录可以设置，一组默认，供4组。分别是

* /common 目录，类库可被 Tomcat 和所有 Web 应用共同使用
* /server 目录，类库可被 Tomcat 使用，对所有 Web 应用不可见
* /shared 目录，类库可以被所有 Web 应用共同使用，对 Tomcat 不可见
* /WebApp/WEB-INF 目录，仅被该 Web 应用使用，对 Tomcat 和其他应用不可见

{% plantuml %}
title Tomcat服务器的类加载架构

node bcl [
启动类加载器
Bootstrap Class Loader
]

node ecl [
扩展类加载器
Extension Class Loader
]

node acl [
应用类加载器
Application Class Loader
]

node ccl #aliceblue;line:blue;line.dotted;text:blue [
Common类加载器
CommonClassLoader
] 

node catalinacl #aliceblue;line:blue;line.dotted;text:blue [
Catalina类加载器
Catalina Class Loader
]

node scl #aliceblue;line:blue;line.dotted;text:blue [
Shared类加载器
Shared Class Loader
]

node webappcl #aliceblue;line:blue;line.dotted;text:blue [
WebApp类加载器
WebApp Class Loader
]

node jspcl #aliceblue;line:blue;line.dotted;text:blue [
Jsp类加载器
Jsper Class Loader
]

bcl <-- ecl
ecl <-- acl

acl <-- ccl

ccl <-- catalinacl
ccl <-- scl

scl <-- webappcl
webappcl <-- jspcl
{% endplantuml %}

实线节点是 JDK 自带加载起，虚线是 Tomcat 自建的加载器。Common类加载器，Catalina类加载器(也称为Server类加载器)，Shared类加载器和Webapp类加载器则是 Tomcat 自定义的类加载器，分别加载 /common/*, /server/*, shared/* 和 /WebApp/WEB-INF/* 中的 Java 类库。WebApp 类加载器和 JSP 类加载器通常还会有多个实例，每个 Web 应用对应一个 WebApp 类加载器，每个 JSP 文件对应一个 JasperLoader 类加载器。

从图可以看出，Common 类加载器能加载的类都可以被 Catalina 类加载器和 Shared 类加载器使用，而 Catalina 类加载器和 Shared 类加载器自己能加载的类则与对方相互隔离。WebApp类加载器可以使用Shared类加载器加载到的类，但各个WebApp类加载器实例之间相互隔离。而JasperLoader的加载范围仅仅是这个JSP文件所编译出来的那一个Class文件，它存在的目的就是为了被丢弃：当服务器检测到JSP文件被修改时，会替换掉目前的JasperLoader的实例，并通过再建立一个新的JSP类加载器来实现JSP文件的HotSwap功能。

上面讲的是 Tomcat6 之前的加载器架构，Tomcat6 之后对默认的目录结构做了简化，只有指定 tomcat/conf/catalina.properties 的 server.loader 和 share.loader 后才会真正建立 Catalina类加载器和Shared类加载器实例，否则用到的地方都会用 Common 类加载器实例代替，而默认的配置文件中没有设置这两项，所以 Tomcat6 之后顺理成章的把 /common, /server 和 /shared 三个目录合并在一起变成 /lib 目录，相当于之前的 /common 目录的作用，是 Tomcat 团队简化部署的一项改动。

### 9.2.2 OSGi：灵活的类加载器架构

没兴趣，用到再看

### 9.2.3 字节码生成技术与动态代理的实现

有兴趣，等 Tomcat 完结了再看，不过应该要先看完第八章的内容才行
