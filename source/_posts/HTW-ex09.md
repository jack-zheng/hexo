---
title: Ex09 Session 管理
date: 2021-09-10 17:48:20
categories:
- Tomcat
tags:
- How Tomcat Works
---

> **Chapter 9** discusses the manager, the component that manages sessions in session management. It explains the various types of managers and how a manager can persist session objects into a store. At the end of the chapter, you will learn how to build an application that uses a StandardManager instance to run a servlet that uses session objects to store values. 

PS: 本节实验失败了，页面显示不出来，追踪了一下，servlet 可以正常加载，但是执行 init() 方法的时候报错了。感觉可以先用之前的 javaweb 项目把这个页面显示出来，在看看问题。有可能是依赖有问题。

主要知识点

* Session 相关的接口关系
* 通过 Manager 管理 session
* 实现了 Lifecycle 接口
* 提供 swap out 功能，节省内存资源 - 长时间不用的 session 暂存
* 持久化

没什么成就感，暂时先记怎么多把