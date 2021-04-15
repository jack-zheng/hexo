---
title: Idea tomcat log 乱码
date: 2020-11-02 16:29:02
categories:
- 配置
tags:
- idea
- 乱码
---

Windows 平台练习 spring 项目的时候，idea 终端 tomcat 乱码，不方便调试排错，可以改动如下

先确定 tomcat 本身是不是有乱码。先启动一个 tomcat，查看 Windows 终端的显示情况，这部分可以查看 tomcat 安装目录的配置文件 `C:\Program Files\Apache Software Foundation\Tomcat 9.0\conf\logging.properties`，所有的编码设置成 UTF-8 `encoding = UTF-8` 

配置 Idea 编码选项

1. Editor -> File Encoding， Global 和 Project 都设置成 UTF-8
2. Java compiler 页面的 'Additional command line parameters:' 添加 `-encoding utf-8`
3. 项目的 tomcat 服务器 VM options: 添加 `-Dfile.encoding=UTF-8`
4. Help -> `Edit Custom VM Options...` 添加配置 `-Dfile.encoding=utf-8`

