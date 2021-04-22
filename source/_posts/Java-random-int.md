---
title: Java 生成随机数
date: 2021-04-21 11:14:00
categories:
- java
tags:
- 随机数
---

遇到一个问题，在写测试的时候需要产生一串随机数，找了一下解决方案，记录一下

## 示例

需求：

1. 字符串以 Test 开头
2. 中间加指定格式的日期
3. 结尾加上前面补0的4位随机整数

```java
public class RandomDemo {
    public static void main(String[] args) {
        String prefix = "Test";

        SimpleDateFormat sdfDate = new SimpleDateFormat("yyMMdd");
        String mid = sdfDate.format(new Date());

        String suffix = String.format("%04d", new Random().nextInt(10000));

        System.out.println(prefix + mid + suffix);
    }
}
// Test2104215709
```

%04d 的含义：

* 0: 前面补0
* 4: 长度为4
* d: 对整形做操作