---
title: NoClassDefFoundError 记录
date: 2020-01-06 10:37:39
categories:
- 编程
tags:
- java
- exception
---
写 UT 的时候遇到一个 NoClassDefFoundError, 以前没碰到过，记一笔

### Root Cause

编译时能找到 class 但是运行时对应的类找不到了，听上去可能不点矛盾

### 与 ClassNotFoundException 的区别

ClassNotFoundException 的场景更多的是我们给出 class name, 然后 JVM 根据名字去 load 的时候找不到就会跑抛出这个异常

NoClassDefFoundError 则是在编译期，JVM 是能找到对应的类的，但是等运行期时找不到了

### 怎么修复

1. 检测 Classpath 是不是缺少你需要的 jar 包，缺少就加一下。我本地就是这个问题，测试的 dependency 中没有类的引用，挂了
2. 检查 error exception stack, 看看是不是类初始化时 static 部分出错了

### 打印 Classpath 调试

通过打印 classpath 输出当前运行环境是否缺少需要的 jar 包。

```java
import java.net.URL;
import java.net.URLClassLoader;

public class PrintClassPath {
    public static void main(String[] args) {
        ClassLoader cl = ClassLoader.getSystemClassLoader();

        URL[] urls = ((URLClassLoader)cl).getURLs();

        for(URL url: urls){
            System.out.println(url.getFile());
        }
    }
}

// Output:
// /Library/Java/JavaVirtualMachines/jdk1.8.0_144.jdk/Contents/Home/jre/lib/charsets.jar
// /Library/Java/JavaVirtualMachines/jdk1.8.0_144.jdk/Contents/Home/jre/lib/deploy.jar
// /Library/Java/JavaVirtualMachines/jdk1.8.0_144.jdk/Contents/Home/jre/lib/ext/cldrdata.jar
// ...
```

### Refer

[很全面的一个 NoClassDefFoundError 异常分析博文](https://javarevisited.blogspot.com/2011/06/noclassdeffounderror-exception-in.html)
