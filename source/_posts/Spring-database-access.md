---
title: Spring database access
date: 2022-11-16 13:33:35
categories:
- Spring
tags:
- database
---

## 统一的异常管理

1. DAO 模式通过接口编程隔离的 DB 实现
2. 实现的时候，由于 JDBC 规范将异常下方给供应商，导致接口代码会随着实现的改变而改变
3. DAO 模式描述的场景实在是诱人，我们通过统一异常来 apply DAO 模式
4. 不能直接在实现层吞了异常，所以把他们包装成 unchecked exception 抛出，这样客户端就不需要强制检查了
5. 各供应商的异常包装形式不同，比如错误信息存放位置，错误代码规定等。所以 Spring 提供了异常转译功能做统一处理

## JDBC API 最佳实践

* 通过模版方法封装重复代码
* 对 SQL Exception 进行转译