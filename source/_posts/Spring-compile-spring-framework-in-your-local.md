---
title: Spring compile spring framework in your local
date: 2021-10-21 15:27:12
categories:
- Spring
tags:
- 源码编译
---

最近看的 Spring 课程上面，那些讲师都会把 Spring 源码下载到本地然后在上面写 demo, 做笔记什么的，感觉这个方式很棒，实践一下。

spring-framework 有自己的官方 [git](git@github.com:spring-projects/spring-framework.git) 地址的，而且 readme 上也将本地编译的步骤写的很清楚了，再参考一下其他人的博客，难度应该不大。

## 实践

1. 下载 v5.2.x 源码进行编译(看 Supported Versions 里的信息，这个版本是支持 JDK8 的)
2. cd spring-framework 修改 gradle.build 文件配置，默认用的官方源会很慢
3. `./gradlew build` 构建，这个命令会下载 gradle.properties 中指定的 gradle 版本用于构建
4. 文章的末尾有说怎么导入 Idea, 先 `./gradlew :spring-oxm:compileTestJava`
5. 在 Idea 中新建项目，从 existing 的 code 中导入，选择 Gradle project，其他就依此点下去就行了，本地导入成功
6. 新建一个 gralde module: spring-debug 写一个简单的 Spring demo, 测试项目是否构建成功
7. 在 module 的 gradle 文件中 dependencies 下添加 spring-context 的依赖
8. 测试通过，删掉 .git 文件，重新上传到自己账户下做为笔记源文件

```build.gradle
compile(project(":spring-context"))
```

```conf
; -- gradle.build 修改 repo 信息如下 --

repositories {
			mavenCentral()
			maven { url "https://repo.spring.io/libs-spring-framework-build" }
			maven { url "https://repo.spring.io/snapshot" } // Reactor
			maven {url 'https://maven.aliyun.com/nexus/content/groups/public/'} //阿里云
			maven {url 'https://maven.aliyun.com/nexus/content/repositories/jcenter'}
		}
```

## Issues

1. 使用 JDK8 编译 v5.3.12 的 code 有三个 module 编译失败，"Execution failed for task ':spring-instrument:compileJava'. > invalid source release: 17"。将源码切换到 v5.2.18 成功，但是有几个 UT 挂了，直接删掉
2. lombok 不 work, 添加 @Data 之后对应的方法并没有生产，手写 getter/setter 可以 work

## 参考

* [CSDN](https://blog.csdn.net/java_lyvee/article/details/107300648)