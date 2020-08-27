---
title: Stream 类概览
date: 2020-06-07 16:45:29
categories:
- 编程
tags:
- java
- effective java
- 枚举和注解
- stream
---

概要：

* Stream 不是数据结构，更像是算法的集合
* 在流操作过程中不会修改元数据
* 以 lambda 表达式为参数
* 惰性
* 免费提供并行计算能力
* 元数据可以无限大
* 类型确定时使用 IntStream 之类的 class 可以提高效率

## API 简介

在 Java 8 的 API 中， Stream 内置了 39 个方法。

匹配，检测 source 中是否有符合条件的元素

| name                                      | return  | 简述                |
| ----------------------------------------- | ------- | ------------------- |
| allMatch(Predicate<? super T> predicate)  | boolean | 全部匹配返回 true   |
| anyMatch(Predicate<? super T> predicate)  | boolean | 只要有一个匹配 true |
| noneMatch(Predicate<? super T> predicate) | boolean | 全部匹配返回 true   |

```java
// 检测Stream 中是否有数能被 2 整除
boolean ret = Stream.of(1, 2, 3, 4, 5).anyMatch(x -> x % 2 == 0);
System.out.println(ret);
// output: true
```

用于产生流对象的方法

| name                                                 | return                       | 简述                                           |
| ---------------------------------------------------- | ---------------------------- | ---------------------------------------------- |
| builder()                                            | static <T> Stream.Builder<T> | 返回一个流的构造器                             |
| concat(Stream<? extends T> a, Stream<? extends T> b) | static <T> Stream<T>         | 拼接多个流并一起操作                           |
| empty()                                              | static <T> Stream<T>         | 创建一个空的流对象                             |
| generate(Supplier<T> s)                              | static <T> Stream<T>         | 传入一个 Supplier 构造器，返回构造器指定的对象 |
| iterate(T seed, UnaryOperator<T> f)                  | static <T> Stream<T>         | seed 为初始值，UnaryOperator 为算法            |
| limit(long maxSize)                                  | Stream<T>                    | 配合其他生成方法指定生成个数                   |
| skip(long n)                                         | Stream<T>                    | 跳过几个元素，可以结合 iterate, generate 使用  |
| of(T... values)                                      | static <T> Stream<T>         | 生成一个流                                     |
| of(T t)                                              | static <T> Stream<T>         | /                                              |

```java
Stream.Builder<String> builder = Stream.builder();
Stream<String> stream = builder.add("Jerry").add("Tom").build();
stream.forEach(System.out::println);
// output: Jerry Tom

// concat sample, concat 中为需要拼接的流对象
Stream.concat(Stream.of("Jerry"), Stream.of("Tom")).forEach(System.out::println);

// 随机产生 3 个整形
Stream<Integer> ret = Stream.generate(new Random()::nextInt).limit(3);

// iterate sample, 0 作为初始值，每次返回值 +1， 返回 3 次
Stream.iterate(0, x -> x+1).limit(3).forEach(System.out::print);
// output: 012
```

常用的查找函数 max/min/distinct

| name                                  | return      | 简述       |
| ------------------------------------- | ----------- | ---------- |
| distinct()                            | Stream<T>   | 去重       |
| max(Comparator<? super T> comparator) | Optional<T> | 查找最大值 |
| min(Comparator<? super T> comparator) | Optional<T> | 查找最小值 |

```java
// distinct sample
Stream.of(1,2,3,1,2,3).distinct().forEach(System.out::print);
// output: 123

// 如果传入的时对象，那个会更具 equals, hashCode 来判断是不是重复
class Person {
    private String name;
    private int age;
    // Constructor, getter and setter

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;
        Person person = (Person) o;
        return age == person.age &&
                Objects.equals(name, person.name);
    }

    @Override
    public int hashCode() {
        return Objects.hash(age, name);
    }

    @Override
    public String toString() {
        return "Person{" + "name='" + name + '\'' + ", age=" + age + '}';
    }
}

Stream.of(new Person("Jack", 30), new Person("Jack", 30), new Person("Jack", 20)).distinct().forEach(System.out::print);
// output: Person{name='Jack', age=30} Person{name='Jack', age=20}
```

生成指定类型的 Stream 对象

| name                                            | return       | 简述                      |
| ----------------------------------------------- | ------------ | ------------------------- |
| map(Function<? super T,? extends R> mapper)     | Stream<T>    | 返回指定类型的 Stream     |
| mapToDouble(ToDoubleFunction<? super T> mapper) | DoubleStream | 返回 Double 类型的 Stream |
| mapToInt(ToIntFunction<? super T> mapper)       | IntStream    | 返回 Int 类型的 Stream    |
| mapToLong(ToLongFunction<? super T> mapper)     | LongStream   | 返回 Long 类型的 Stream   |

```java
 Stream.of("Jack", "Tom").map(x -> "Name: " + x).forEach(System.out::println);
 // output: Name: Jack Name: Tom

 // 其他几个类似，只不过把返回类型指定了
```

将流中的处理结果整合输出到集合中

| name                                                                                         | return  | 简述 |
| -------------------------------------------------------------------------------------------- | ------- | ---- |
| collect(Collector<? super T,A,R> collector)                                                  | <R,A> R | /    |
| collect(Supplier<R> supplier, BiConsumer<R,? super T> accumulator, BiConsumer<R,R> combiner) | <R> R   | /    |

```java
// 将流中的值连接起来
Stream.of("Jack", "Tom").collect(Collectors.joining(", "));

// 将字符串组成 string, length 的键值对
Map<String, Integer> ret = Stream.of("Jack", "Tom").collect(Collectors.toMap(Function.identity(), String::length));
```

map 及类似的操作

| name                                                               | return        | 简述                          |
| ------------------------------------------------------------------ | ------------- | ----------------------------- |
| map(Function<? super T,? extends R> mapper)                        | <R> Stream<R> | 对流中的元素逐个操作          |
| mapToDouble(ToDoubleFunction<? super T> mapper)                    | DoubleStream  | /                             |
| mapToInt(ToIntFunction<? super T> mapper)                          | IntStream     | /                             |
| mapToLong(ToLongFunction<? super T> mapper)                        | LongStream    | /                             |
| flatMap(Function<? super T,? extends Stream<? extends R>> mapper)  | <R> Stream<R> | 和 map 主要的区别时**扁平化** |
| flatMapToDouble(Function<? super T,? extends DoubleStream> mapper) | DoubleStream  | /                             |
| flatMapToInt(Function<? super T,? extends IntStream> mapper)       | IntStream     | /                             |
| flatMapToLong(Function<? super T,? extends LongStream> mapper)     | LongStream    | /                             |

```java
// 扁平化就是将集合中的集合拆散成基本元素，下例中将 list 中的最基本的元素做平方操作
List<List<Integer>> listOfList = Arrays.asList(Arrays.asList(1,2), Arrays.asList(3,4));
listOfList.stream().flatMap(Collection::stream).map(x -> x*x).forEach(System.out::println);
// output: 1, 4, 9 ,16
```

过滤

| name                                                                                  | return      | 简述                              |
| ------------------------------------------------------------------------------------- | ----------- | --------------------------------- |
| filter(Predicate<? super T> predicate)                                                | Stream<T>   | 根据 predicate 过滤               |
| reduce(BinaryOperator<T> accumulator)                                                 | Optional<T> | 从多个元素中产生一个结果          |
| reduce(T identity, BinaryOperator<T> accumulator)                                     | T           | identity - 初始值                 |
| reduce(U identity, BiFunction<U,? super T,U> accumulator, BinaryOperator<U> combiner) | <U> U       | combiner 是并行运算时需要指定的值 |

```java
Optional<String> ret = Stream.of("Jack", "Tom").reduce(String::concat);
System.out.println(ret.get());
// output: JackTom

// 使用 identity 作为初始值
Optional<String> ret = Optional.ofNullable(Stream.of("Jack", "Tom").reduce("Name Ret:", String::concat));
System.out.println(ret.get());
// output: Name Ret:JackTom
```

Find*

| name        | return      | 简述                                                                   |
| ----------- | ----------- | ---------------------------------------------------------------------- |
| findAny()   | Optional<T> | 随机返回一个值，并不关心值的内容，在单线程中一般返回第一个，但是不保证 |
| findFirst() | Optional<T> | 返回第一个                                                             |

```java
System.out.println(Stream.of(1,2,3,4).findFirst().get());
```

forEach*

| name                                       | return | 简述                                   |
| ------------------------------------------ | ------ | -------------------------------------- |
| forEach(Consumer<? super T> action)        | void   | 遍历不保证顺序(多线程下可能会顺序不定) |
| forEachOrdered(Consumer<? super T> action) | void   | 遍历保证顺序                           |

count

| name                                     | return    | 简述                   |
| ---------------------------------------- | --------- | ---------------------- |
| count()                                  | long      | 输出元素个数           |
| peek(Consumer<? super T> action)         | Stream<T> | 得到流对象，可用于调试 |
| sorted()                                 | Stream<T> | 使用自然排序           |
| sorted(Comparator<? super T> comparator) | Stream<T> | 定制排序               |
| toArray()                                | Object[]  | 生成数组               |
| toArray(IntFunction<A[]> generator)      | <A> A[]   | /                      |

```java
// steam 转化为 array
Stream<String> stringStream = Stream.of("a", "b", "c");
String[] stringArray = stringStream.toArray(String[]::new);
Arrays.stream(stringArray).forEach(System.out::println);
```

## 语法糖

创建实例或者调用方法时可以使用 `::` 两个冒号的形式调用

## Supplier 使用举例

```java
DateTimeFormatter dtf = DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss");
Supplier<LocalDateTime> s = LocalDateTime::now;
System.out.println(s.get());

Supplier<String> s1 = () -> dtf.format(LocalDateTime.now());
System.out.println(s1.get());
```