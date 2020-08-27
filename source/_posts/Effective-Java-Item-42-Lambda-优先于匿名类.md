---
title: Effective Java Item 42 Lambda 优先于匿名类
date: 2020-06-04 21:41:24
categories:
- 编程
tags:
- java
- effective java
- Lambda和Stream
---

本节要点：

* 使用 lambda 代替匿名函数
* 不要指定 lambda 中的数据类型，除非报错
* 主要长度，最多三行

名词对照表

| EN                 | CN       |
| ------------------ | -------- |
| function type      | 函数类型 |
| function object    | 函数对象 |
| function interface | 函数接口 |
| type inference     | 类型推导 |
| raw type           | 原生类型 |

自从 java 1.1 发布依赖，如果我们想要创建一个方法对象那么就需要使用到匿名函数

```java
List<String> list = Arrays.asList("jerry", "tom");
Collections.sort(list, new Comparator<String>() {
    @Override
    public int compare(String o1, String o2) {
        return Integer.compare(o1.length(), o2.length());
    }
});
list.forEach(System.out::println);
// output:
// tom
// jerry
```

这种表述方式可以实现我们的需求，但是实现繁琐并且语义表达不顺畅， 在 java 8 中，我们可以使用 lambda 来代替匿名函数

```java
List<String> list = Arrays.asList("jerry", "tom");
Collections.sort(list, (s1, s2) -> Integer.compare(s1.length(), s2.length()));
list.forEach(System.out::println);

// 甚至可以简写为
Collections.sort(list, Comparator.comparingInt(String::length));

// 或者更甚
list.sort(Comparator.comparingInt(String::length));
```

再使用 lambda 的时候有一条原则**去掉 lambda 中的所有参数类型，除非它能使你的表达更清楚**。默认情况下，程序会根据上下文推断出类型，实在不行它会报错的，那个时候你再自己修不迟。

Operator 枚举类优化，可以将参数使用 DoubleBinaryOperator 这个方法接口做优化

```java
// 原始代码
enum Operation {
    PLUS("+") {
        @Override
        public double apply(double x, double y) {
            return x + y;
        }
    },
    MINUS("-") {
        @Override
        public double apply(double x, double y) {
            return x - y;
        }
    },
    TIMES("*") {
        @Override
        public double apply(double x, double y) {
            return x * y;
        }
    },
    DIVIDE("/") {
        @Override
        public double apply(double x, double y) {
            return 0;
        }
    };
    private final String symbol;

    Operation(String symbol) { this.symbol = symbol; }
    @Override
    public String toString() { return symbol; }
    public abstract double apply(double x, double y);
}
```

通过将上面的 apply() 方法抽象，这个 Operation 的枚举中的行为可以看作是传入两个数，进行计算， 我们将计算抽象，得到如下的简化形式

```java
enum Operation {
    PLUS("+", (x, y) -> x + y),
    MINUS("-", (x, y) -> x - y),
    TIMES("*", (x, y) -> x * y),
    DIVIDE("/", (x, y) -> x / y);
    private final String symbol;
    private final DoubleBinaryOperator operator;

    Operation(String symbol, DoubleBinaryOperator operator) {
        this.symbol = symbol;
        this.operator = operator;
    }

    @Override
    public String toString() { return symbol; }

    public double apply(double x, double y) {
        return operator.applyAsDouble(x, y);
    }
}
```

缺点：lambda 没有名字和文档，如果一段算法不能自我描述，或者超出了几行，就别把他放到一个 lambda 函数中。 

lambda 注意点：

* 一个 lambda 一行是最理想的，最多不能超过三行！
* 绝大多视情况下，使用 lambda 代替匿名函数，但是如果是对抽象类的实现，还是得依靠匿名函数， lambda 并不能完成这样的功能。
* lambda 不能获取自身引用， 在 lambda 中 this 指代的是外围示例，匿名类中 this 指自己
* 可能的话，别去序列化 lambda 和 匿名函数
* lambda 是小函数的最佳表现方式，除非万不得已，不然就别用匿名类实现函数接口
  