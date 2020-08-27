---
title: Effective Java Item 55 谨慎返回 Optional
date: 2020-06-08 22:17:59
categories:
- 编程
tags:
- java
- effective java
- 方法
---

在 Java 8 中，引入了 Optional class 给我们在处理无法返回任何值的情况下，有了第三种选择。

## Optional 概览，基于 Java 1

他是一个 final 类， 方法列表如下

| name                                                                 | 返回值      | 简述                                                        |
| -------------------------------------------------------------------- | ----------- | ----------------------------------------------------------- |
| empty()                                                              | Optional<T> | 返回一个空的实例                                            |
| filter(Predicate<? super T> predicate)                               | Optional<T> | 过滤                                                        |
| flatMap(Function<? super T, ? extends Optional<? extends U>> mapper) | Optional<U> | 扁平化操作                                                  |
| get()                                                                | T           | 取值                                                        |
| ifPresent(Consumer<? super T> action)                                | void        | 如果值存在，执行给定的操作                                  |
| ifPresentOrElse(Consumer<? super T> action, Runnable emptyAction)    | void        | 如果存在，执行给定操作，否则运行 empty-base action          |
| isEmpty()                                                            | boolean     | 是否为空                                                    |
| isPresent()                                                          | boolean     | 是否有值                                                    |
| map(Function<? super T, ? extends U> mapper)                         | Optional<U> | 对每个元素操作                                              |
| of(T value)                                                          | Optional<T> | 生成对象                                                    |
| ofNullable(T value)                                                  | Optional<T> | 生成 empty 或 有值的 optional 对象                          |
| or(Supplier<? extends Optional<? extends T>> supplier)               | Optional<T> | present 返回自己，否则返回 supplier 生成的对象              |
| orElse(T other)                                                      | T           | present 返回自己，否则返回 else 中指定的值                  |
| orElseGet(Supplier<? extends T> supplier)                            | T           | present 返回自己，否则返回 else 中指定的 spplier 生成的对象 |
| orElseThrow()                                                        | T           | 存在值，返回，否则抛 NoSuchElementException                 |
| stream()                                                             | Stream<T>   | 产生流                                                      |
| toString()                                                           | String      | /                                                           |

```java
Optional<String> op = Optional.of("test");
System.out.println(op.isEmpty()); // false
System.out.println(op.isPresent()); // true
Optional<String> op2 = Optional.empty();
op2.get(); // Exception in thread "main" java.util.NoSuchElementException
String ret = op2.orElse("backup"); // backup
```

or vs orElseGet: 返回值不同，前者返回 Optional 对象，后者返回的泛型指定的值

## item 55

本节要点：

* Optional 强制客户端对返回值做校验
* 如果不能从 Optional 中 get 值，会抛 NoSuchElementException
* 永远不要通过返回 Optional 的方法返回 null, 这违背了设计的本意
* Optional 本质上与受检测异常相似
* 容器类，比如 map, stream, 数组和 optional 都不应该装载在 optional 中，你可以返回空的容器，比如空的数组
* 不要返回基本包装类型的 Option， 有其他的替代品比如 OptionalInt
* Optional 不要作为map， set 中的键元素，数组也不行
* Optional 相对而言还是比较消耗资源的，性能要求高的场景谨慎使用

示例代码：

```java
public static <E extends Comparable<E>> E max(Collection<E> c) {
    if (c.isEmpty()) {
        throw new IllegalArgumentException("Empty collection");
    }

    E result = null;
    for (E e : c) {
        if (result == null || e.compareTo(result) > 0) {
            result =  Objects.requireNonNull(e);
        }
    }
    return result;
}
```

使用 Optional 优化

```java
public static <E extends Comparable<E>> Optional<E> max(Collection<E> c) {
    if (c.isEmpty()) {
        return Optional.empty();
    }

    E result = null;
    for (E e : c) {
        if (result == null || e.compareTo(result) > 0) {
            result =  Objects.requireNonNull(e);
        }
    }
    return Optional.of(result);
}
```

使用 Stream 优化

```java
public static <E extends Comparable<E>> Optional<E> max(Collection<E> c) {
    return c.stream().max(Comparator.naturalOrder());
}
```

其他示例

```java
// 如果没有返回备选
max(words).orElse("other words...");

// 如果没有，抛出异常
max(toys).orElseThrow(TmperTantrumException::new);

ph.parent().map(h -> String.valueIf(h.pid())).orElse("N/A");

// 过滤非空的 Optional 集合
List<Optional<String>> listOfOptionals = Arrays.asList(Optional.empty(), Optional.of("foo"), Optional.empty(), Optional.of("bar"));
// Java 8 
List<String> filteredList = listOfOptionals.stream()
  .filter(Optional::isPresent)
  .map(Optional::get)
  .collect(Collectors.toList());
// Java9 中可以简化为
List<String> filteredList = listOfOptionals.stream()
  .flatMap(Optional::stream)
  .collect(Collectors.toList());
```