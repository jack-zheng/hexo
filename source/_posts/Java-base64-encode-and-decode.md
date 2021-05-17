---
title: Java 中实现 base64 编码/解码
date: 2021-05-17 18:56:05
categories:
- java
tags:
- base64
---

Java util 包中自带了 base 64 的编码/解码方法，示例如下

```java
import java.util.Base64;

public class Base64Test {
    public static void main(String[] args) {
        String origin = "HelloWorld";
        String encodeRet = Base64.getEncoder().encodeToString(origin.getBytes());
        System.out.println("Encode result: " + encodeRet);

        System.out.println("Decode result: " + new String(Base64.getDecoder().decode(encodeRet)));
    }
}

// Encode result: SGVsbG9Xb3JsZA==
// Decode result: HelloWorld
```