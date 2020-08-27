---
title: 创建 Date 对象
date: 2020-01-03 17:43:50
categories:
- 编程
tags:
- java
- date
---
简单记录一下 java 中 Date 类的使用

### 通过 Date 创建

默认构造函数会创建当前时间点的 Date 对象, 另外还可以通过 `Date(long milliseconds)` 的构造器创建指定时间的日期对象

主要方法：

* getTime() - return milliseconds
* before(Date) - if date is before target date
* after(Date)

```java
public static void main(String[] args) {
    Date date = new Date();
    // The default date fromat is: "EEE MMM dd HH:mm:ss zzz yyyy";
    System.out.println(date);
    System.out.println(date.getTime());
}

// Output:
// Fri Jan 03 17:47:16 CST 2020
// 1578045256817
```

### 通过 SimpleDateFormat 创建

相比于上一种方式，这种更易懂一点，而且可以指定输出格式呦(´▽｀)

```java
public static void main(String[] args) throws ParseException {
    Date simpleDate = new SimpleDateFormat("yyyy-MM-dd").parse("2020-01-03");

    System.out.println(simpleDate);
    System.out.println(new SimpleDateFormat("MM-dd-yyyy").format(simpleDate));
}

// Output:
// Fri Jan 03 00:00:00 CST 2020
// 01-03-2020
```
