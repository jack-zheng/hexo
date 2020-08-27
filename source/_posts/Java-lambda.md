---
title: Java 函数式编程
date: 2020-05-28 21:15:28
categories:
- 编程
tags:
- java
- 函数式
- lambda
---
Java 8 函数式编程读书笔记

## 第一章 简介

* Java 8 增加 Lambda 表达式来支持对大型数据的并发操作 - 核实一下
* 面向对象式对数据进行抽象，函数式编程时对行为进行抽象

## 第二章 Lambda 表达式

以 Swing 为例，传统做法中监听事件需要如下代码：

```java
button.addActionListener(
    new ActionListener() {
        public void actionPerformed(ActionEvent event) {
            System.out.println("button clicked");
        }
    }
);
```

当我们使用 lambda 表达式时可以简写为 `button.addActionListener(event -> System.out.println("button clicked"));`

Lambda 表达式中的类型都是由编译器推断出来的，但你也可以显示的声明参数类型。

Lambda 表达式引用的是值，而不是变量。

| 接口              | 参数   | 返回类型 | 示例                  |
| ----------------- | ------ | -------- | --------------------- |
| Predicate\<T\>      | T      | boolean  | 唱片是否发行          |
| Consumer\<T\>       | T      | void     | 输出一个值            |
| Function<T, R>    | T      | R        | 获取Artist 对象的名字 |
| Supplier\<T\>       | None   | T        | 工厂方法              |
| UnaryOperator\<T\>  | T      | T        | 逻辑非(!)             |
| BinaryOperator\<T\> | (T, T) | T        | 求连个数的乘积(*)     |

在复杂的情况下需要指定泛型类型才能使编译通过

```java
// 如果省略掉 <Long> 编译报错：Operator'&#x002B;'cannotbeappliedtojava.lang.Object,java.lang.Object.
BinaryOperator<Long> addLongs = (x, y) -> x + y;
```

Predicate 可用于检测对象是否符合要求

```java
// 检测字符串是否以 J 开头
List<String> names = Arrays.asList("Jerry", "Tom");
List<String> ret = names.stream().filter(name -> name.charAt(0) == 'J').collect(Collectors.toList());

// filter 中的部分就是 Predicate 表达式，也可以分开定义写成如下形式
Predicate<String> filterTom = input -> input.equals("Tom");
ret = names.stream().filter(filterTom).collect(Collectors.toList());
```

Consumer 对传入的参数做操作，没有返回值，例如可以用它实现打印，断言等操作

```java
ret.forEach(System.out::print);
names.forEach(name -> Assert.assertEquals(name, "Jerry"));
```

Function 对传入的对象操作并放回结果

```java
List<String> ret = names.stream().map(String::toUpperCase).collect(Collectors.toList());
ret.forEach(System.out::println);
```

Supplier 可以帮你生产数据, 但是只能使用应用于无参的 constructor，不支持传入参数

```java
Supplier<User> userSupplier = User::new;
```

## 第三章 流

```java
// for 处理集合模板
int count=0;
for (Artist artist : allArtists) {
    if (artist.isFrom("London")) {
        count++;
    }
}
``

外部迭代方式： 通过拿到 iterator， 然后通过 hasNext(), next() 方法完成迭代

```java
int count=0;
Iterator<Artist> iterator = allArtists.iterator();
while (iterator.hasNext()) {
    Artistartist = iterator.next();
    if (artist.isFrom("London")) {
        count++;
    }
}
```

内部迭代：通过 Steam 对集合类进行复杂操作

```java
// filter：只保留通过某项测试的对象，整个过程被分为两步，过滤和计算
long count = allArtists.stream().filter(artist -> artist.isFrom("London")).count();
```

filter 中的表达式是惰性求值方法，count 是及早求职方法，惰性求值并不会真正执行

```java
// 此实例中并不会在控制台打印文字
allArtists.stream().filter(artist -> {
            System.out.println("print artist's location: " + artist.location);
            return artist.isFrom("London");
        });
```

如果返回值是 Stream 则为 惰性求值；如果返回值是另一个值或为空则是 及早求值。使用这些操作的理想方式就是形成一个惰性求值的链，最后用一个及早求值的操作返回想要的结果，这正是它的合理之处。

### 常用的流操作

| 操作    | 用途            | 示例                                                                                          |
| ------- | --------------- | --------------------------------------------------------------------------------------------- |
| collect | 生成集合        | Stream.of("a", "b", "c").collect(Collectors.toList());                                        |
| map     | 类型转换        | Stream.of("a").map(string -> string.toUpperCase()).collect(Collectors.toList());              |
| filter  | 检查过滤        | Stream.of("a", "12b").filter(val -> isDigit(val.charAt(0))).(Collectors.toList());            |
| flatMap | 拼接多个 Stream | Stream.of(asList(1, 2), asList(3, 4)).flatMap(numbers -> numbers.stream()).collect(toList()); |
| max/min | 最值            | tracks.stream().min(Comparator.comparing(track -> track.getLength())).get();                  |
| reduce  | 提供计算功能    | Stream.of(1,2,3).reduce(0, (acc, ele) - > acc + ele);                                         |

## 第四章 类库

日志优化

```java
Logger logger = new Logger();
if (logger.isDebugEnabled()) {
    logger.debug("Look at this:" + expensiveOperation());
}
```

使用 Lambda 优化

```java
Logger logger = new Logger();
logger.debug(() -> "Look at this:" + expensiveOperation());

// 在 Logger 类中添加方法
public void debug(Supplier<String> message) {
    if (isDebugEnabled()) {
        debug(message.get());
    }
}
```

Supplier -> get(), Predicate -> test, Function -> apply.

如果可以的话，在流中尽量使用对基本类型的操作，而不是封装类型。 mapToInt 之类的操作还提供了很多简便操作得到最值和平均值。

**Optional**是一个新设计的数据类新来替换 null 值。 使用它有两个目的：

* Optional 对象鼓励程序员适时检测变量是否为空，以避免代码缺陷
* 将一个类的 API 中可能为空的值文档化，这比阅读实现代码要简单的很多

## 第五章 高级集合类和收集器

`方法引用`语法， artist -> artist.getName() 等价于 Artist::getName, 标准语法为 Classname::methodName. 由此，新建 Artist 对象的代码可以由 （name, nationality） -> new Artist(name, nationality) 简化为 Artist::new, 类似的可以通过 String[]::new 创建新的数组。

`stream.collect()` 可以生成你想要的集合形式。例如：`stream.collect(toCollection(TreeSet::new));`

`partitioningBy` 收集器可用于分流, 与之类似的还有 `groupingBy` 关键字。

```java
public Map<Boolean, List<Artist>> bandsAndSolo(Stream<Artist> artists) {
    return artist.collect(partitioningBy(artist -> artist.isSolo())); // artist -> artist.isSolo() 可替换为 Artist::isSolo
}
```

字符串流操作示例：

```java
String ret = artists.steam().map(Artist::getName).collect(Collectors.joining(",", "[", "]"));
```

查询并加入 map 的简化操作：

```java
// before
public Artist getArtist(String name) {
    Artist artist = artistCache.get(name);
    if (artist == null) {
        artist = readArtistFromDB(name);
        artistCache.put(name, artist);
    }
    return artist;
}

// after
public Artist getArtist(String name) {
    return artistCache.computeIfAbsent(name, this::readArtistFromDB);
}
```

通过 forEach 简化 map 的统计操作：

```java
// before
Map<Artist, Integer> countOfAlbums = new HashMap<>();
for(Map.Entry<Artist, List<Album>> entry : albumsByArtist.entrySet()) {
    Artist artist = entry.getKey();
    List<Album> albums = entry.getValue();
    countOfAlbums.put(artist, albums.size());
}
// after
Map<Artist, Integer> countOfAlbums = new HashMap<>();
albumsByArtist.forEach((artist, albums) -> {
    countOfAlbums.put(artist, albums.size());
});
```

## 第六章 数据并行化

并行化：同一任务拆分，多核执行
并发化：单核多任务

实现上只需要在调用方法时将 `.stream()` 改为 `.parallelStream()` 就行了。但是并不是并行了就快，取决于处理量等其他因素。

影响因素：数据大小， 源数据结构， 装箱， 核的数量， 单元处理开销， 底层还是使用了 fork/join 的模式。

数据结构并行性能：ArrayList, 数组， IntStream.range > HashSet, Treeset > LinkedList, Streams.iterate, BufferedReader.lines

为 array 赋初值：

```java
int[] a = new int[100];
Arrays.setAll(a, i->i);
// 输出：0，1，3.。。99
```

## 第七章 测试，调试和重构

ThreadLocal 优化：

```java
// before
ThreadLocal<album> thisAlbum = new ThreadLocal<Album> () {
    @Overrride protected Album initialValue() {
        return database.lookupCurrentAlbum();
    }
}
// after
ThreadLocal<Album> thisAlbum = ThreadLocal.withInitial(() -> database.lookupCurrentAlbum());
```

可以使用 peek 进行流的调试

## 第八章 设计和架构的原则

列举了 Lambda 和 设计模式， DSL 的结合的例子，和我看这本书的初衷有点远了，先跳过。

## 第九章 使用 Lambda 表达式编写并发程序

使用 Vertx 框架结合 Lambda 的知识点，实现一个聊天室，跳过。但是它的这个框架我倒是感觉很有意思，灵感是从 NodeJS 那边来的，支持并发。

## 工作中遇到的一些例子

### Map -> Map 转化

Map\<String, List\<Obj\>\> 对 list 中的值进行修改，案例简化为 Map<String, List\<String>\> 将 list 中的 String 转化为大写

第一步先熟悉 list -> list 转化方式

```java
List<String> test = Arrays.asList("a", "c");
List<String> answer = test.stream().map(String::toUpperCase).collect(Collectors.toList());
System.out.println(answer);
//output: [A, C]
```

熟悉 map 转化方式并结合 list 转化

```java
Map<String, List<String>> origin = new HashMap<>();
origin.put("a", Arrays.asList("a", "n"));
origin.put("b", Arrays.asList("b"));
origin.put("c", Arrays.asList("c"));

System.out.println(origin);

Map<String, List<String>> after = origin.entrySet().stream().
        collect(Collectors.toMap(
                Map.Entry::getKey, (entry) -> entry.getValue().stream().map(String::toUpperCase).collect(Collectors.toList()))
        );
System.out.println(after);
// output:
// {a=[a, n], b=[b], c=[c]}
// {a=[A, N], b=[B], c=[C]}
```
