---
title: Java 中 C 语言风格的参数声明
date: 2020-08-13 14:21:39
categories:
- 编程
tags:
- java
---

最近在调查一个 build issue 的时候发现有一段函数声明大致如下

```java
public class ParameterTest {

    public static void main(String[] args) {
        ParameterTest parameterTest = new ParameterTest();
        parameterTest.test(new String[]{"Jack"});
    }

    public void test(String list[]) {
        System.out.println(list);
    }
}
```

但是就感觉很好奇，`test(String list[])` 这样的声明竟然能通过编译检测。查了下资料，这中做法是合法的，是 C 语言中数组的声明方式，大概是早期为了让 C 程序员能更好的迁移过来做的兼容把，表达的语意和 `test(String[] list)` 是完全一样的。

果然一个老项目里面什么情况都能遇到 ╮(￣▽￣"")╭

