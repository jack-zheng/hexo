---
title: Java Autocloseable
date: 2022-07-22 17:51:52
categories:
- java
tags:
- AutoCloseable
---

最近在看其他 team 的代码时，看到一个在 Filter 中添加 AutoCloseable 的方式来达到记录 perf log 的目的，听新颖的，记录一下并熟悉一下 AutoCloseable 接口的用法

## 示例

在这之前，我一直以为，想要在方法(request)执行前后记录执行时间，只能通过新建一个 拦截器 或者使用类似 Aspect 的技术。API team 的这个实现着实让我对 AutoCloseable 的使用有了新认识，之前对这个接口只是看到过的程度，哈哈。大致模型如下

```java
// perf 类实现 AutoCloseable 接口，在 close
public class SimpleAutoCloseable implements AutoCloseable {

    public SimpleAutoCloseable(String name) {
        System.out.println("Start to record time cost for scope: " + name);
    }

    @Override
    public void close() throws Exception {
        System.out.println("Stop record, print time cost: xx ms");
    }
}

public class Client {
    public void doFilter() throws Exception {
        try(SimpleAutoCloseable auto = new SimpleAutoCloseable("Request")) {
            System.out.println("special filter logic...");
            System.out.println("filterChain.doFilter(request, response)");
        }
    }
}
```

测试类中，我们调用 client 的方法触发 try-with-resource 流程，查看 console log, 可以看到当 chain 的 doFilter 整体执行结束之后，自动打印了 perf 信息，666

```java
@Test
public void test_AutoCloseable_class_will_execute_close_after_try_block() throws Exception {
    Client client = new Client();
    client.doFilter();
}

// Start to record time cost for scope: Request
// special filter logic...
// filterChain.doFilter(request, response)
// Stop record, print time cost: xx ms
```
