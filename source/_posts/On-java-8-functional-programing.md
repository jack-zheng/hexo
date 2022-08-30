---
title: On java 8 函数式编程
date: 2022-08-12 16:48:53
categories:
- On Java 8
tags:
- 函数式编程
---

OO（object oriented，面向对象）是抽象数据，FP（functional programming，函数式编程）是抽象行为。

纯粹的函数是编程有额外约束，数据必须不可变：设置一次，永不改变。这种方式解决了多处理器时数据竞争的问题

Lambda 表达式是使用最小可能语法编写的函数定义

* Lambda表达式产生函数，而不是类
* Lambda语法尽可能少，这正是为了使Lambda易于编写和使用

Lambda表达式基本语法：

1. 参数
2. -> 可视为产出
3. -> 之后的内容都是方法体

Lambda 表达式通常比匿名内部类产生更容易读懂的代码

方法引用：类名或对象名::方法名

函数式接口，只能包含一个方法，如果出现多个，会抛 exception(这个结论有问题 Function 这个接口就有多个方法)。这个接口可以添加 @FunctionalInterface 注释，但是可选项，不加也没问题。

java.util.function 包旨在创建一组完整的目标接口，使得我们一般情况下不需要再定义自己的接口。他们的命名规则如下：

1. 如果处理对象而非基本类型，名称则为 Function, Consumer, Predicate 等。参数类型通过泛型添加。
2. 如果接受基本类型，则由名称的第一部分表示，如 LongConsumer, DoubleFunction，IntPredicate 等，返回基本类型的 Supplier 例外
3. 如果返回值为基本类型，则用 To 表示,如 ToLongFunction<T> 和 IntToLongFunction
4. 返回类型与参数类型一致，则是一个运算符，单个参数用UnaryOperator,两个参数用BinaryOperator
5. 接收两个参数，返回布尔值用 Predicate
6. 加收的两个参数类型不同，则名字中有一个 Bi

使用函数接口时，方法名无关紧要，只要参数类型和返回值相同即可。

高阶函数：指一个消费或产生函数的函数。

闭包：讲了一写变量作用域的问题，不过不太看的懂。大概是函数中的变量都相当于 final 吧，大概这个意思。

函数组合：多个函数组成新的函数，是函数式编程的基本组成部分。

柯里化：将一个多参的函数转化为一系列单参函数, 套娃表达式，理解上有点难读懂

纯函数式编程：Java支持并发，但是如果你的业务核心部分都是并发的，应该考虑使用 Scala或者 Clojure， 他们在一开始就为保持不变性而设计的。