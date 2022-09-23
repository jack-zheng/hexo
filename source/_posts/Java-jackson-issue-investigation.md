---
title: Java jackson issue investigation
date: 2022-09-08 15:24:02
categories:
tags:
---

在测试 jackson API 的时候遇到一个很奇怪的问题，ObjectMapper 调用 print 之后再设置就不起作用了。下面的代码，最后的打印语句不能输出 protected 的 field，如果将第一个打印语句注释掉就可以。谷歌了半天没看到有人解答这个问题，汗，看来得自己看源码了。

```java
public class SetVisibilityTest {
    public String attr1 = "public attr";
    protected String attr5 = "protected str";

    public static void main(String[] args) throws Exception {
        SetVisibilityTest bean = new SetVisibilityTest();

        ObjectMapper mapper = new ObjectMapper();
        System.out.println(mapper.writerWithDefaultPrettyPrinter().writeValueAsString(bean));
        mapper.setVisibility(PropertyAccessor.FIELD, JsonAutoDetect.Visibility.ANY);
        System.out.println(mapper.writerWithDefaultPrettyPrinter().writeValueAsString(bean));
    }
}
```

呃。。。下了源码，读了 ObjectMapper 的文档，至少从使用方面找到了这个限制。

> Mapper instances are fully thread-safe provided that ALL configuration of the
> instance occurs before ANY read or write calls. If configuration of a mapper instance
> is modified after first usage, changes may or may not take effect, and configuration
> calls themselves may fail.

解决方案有两种，一个是使用 ObjectReader/Writer 代替，另一个是新建一个 mapper 或者使用 mapper.copy() 效果一样。不过我对为什么会有这种 behavior 还是很好奇，可以继续看看源码

## Jackson 基本信息

官网 [Github - jackson](https://github.com/FasterXML/jackson)

主要由三部分组成

* Streaming, jackson-core 底层实现
* Annotation, jackson-annotation 注解相关
* Databind, jackson-databind 我们直接使用的层，做 POJO 到 Json 的 map

貌似它里面还有关于对象树的处理，我对这种数据结构挺感兴趣，刚好看看他是怎么实现的。