---
title: Lambda 使用时遇到的一些奇奇怪怪的问题
date: 2021-03-10 15:56:02
categories:
- java
tags:
- lambda
---

* `Predicate pre = Boolean::valueOf;` compile failed, 提示说: `Cannot resolve method 'valueOf'` 改为 `Predicate<Boolean> pre = Boolean::valueOf;` works
* `Predicate<Boolean> pre = Boolean::valueOf; pre.test(null);` 会抛出 NPE
* `Predicate<Boolean> pre2 = Objects::isNull; pre2.test(null);` 类似的调用 Objects 的 isNull 等方法却不会跑错

貌似无解，根据这个 [StackOverflow lambda](https://stackoverflow.com/questions/29143803/java-lambdas-how-it-works-in-jvm-is-it-oop) 相关的问题来看，JVM 解析 lambda 的时候，直接将我们写的表达式编译成字节码，然后 JVM 通过 `InvokeDynamic` 指令就执行了，如果是这样的还，我是我发看到他的中间状态的，上面的那些问题看来只能通过经验来解决了 （；￣ェ￣）
