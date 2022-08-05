---
title: Effective Java 异常
date: 2022-08-05 10:44:32
categories:
- Effective Java
tags:
- Exception
---

异常章节摘抄

* 只针对异常的情况才使用异常
* 对可恢复的情况使用 checked exception, 不可恢复的使用 runtime exception
* 避免滥用 checked exception
* 优先使用 JDK 提供的标准异常
* 抛出与上下文匹配的异常
* 在方法注释上，为每个抛出的异常编写文档
* 异常信息中包含失败信息
* 让失败保持源字形
* 不要忽略异常

## 对可恢复的情况使用 checked exception, 不可恢复的使用 runtime exception

* 如果期望调用者能够适当的恢复，对于这种情况应该使用 checked exception
* runtime exception 和 error 行为上是等同的，没有捕获，当前线程停止
* runtime exception 表明程序错误

## 避免滥用 checked exception

滥用 checked exception 会使你的 API 调用极其麻烦。一旦抛出 checked exception，调用者就必须 catch 他或者在方法标签中 throw，传播出去，无论哪种处理方式，都是对调用者不可忽视的负担。

绕过 checked exception 的方法：

* 使用 runtime exception 代替
* 重构逻辑

## 优先使用 JDK 提供的标准异常

常见的标准异常

* IllegalArgmentException
* IllegalStateException
* NullPointException
* IndexOutOfBoundsException
* ConcurrentModificationException
* UnsupportedOperaitonException

## 抛出与上下文匹配的异常

底层异常，传递到上层是可能和当前上下文不匹配，污染 API。这是需要在上层转译一下，重新抛出异常。如果异常栈对错误排查很有帮助，可以将 exception chain 一并传递给新的异常(Throwable 的 initCause 方法)

异常转译比直接传递底层异常好，但更好的处理方式应该是调用底层方法之前确保他能执行成功，避免抛出一场。有时调用底层之前，先做一下参数监测可能是更好的方式。

如果无法避免底层异常，次选方案是让上层悄悄绕开这些异常，从而使上层调用者和底层问题隔离开来，通常这是还会打个 log 记录，方便回溯问题。