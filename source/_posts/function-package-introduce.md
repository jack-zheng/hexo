---
title: function 包简介
date: 2020-06-03 21:02:15
categories:
- 编程
tags:
- java
- lambda
---
对 java 的 lang 包下的 function 包做一下简要的总结， 写本篇文章时参考的 java 11 的源代码。说实话，我总觉得函数接口的定义，语义上很奇葩，不怎么读的懂，比如源码中 Predicate 的 isEquals 方法是这样定义的

```java
static <T> Predicate<T> isEqual(Object targetRef) {
    return (null == targetRef)
            ? Objects::isNull
            : object -> targetRef.equals(object);
}
```

简直是看的我一脸的黑人问号啊 ？？？ 这 TM 什么鬼，有空再研究一下怎么自定义函数接口。

## 概况

function 包是函数接口的集合，包路径为： `java.util.function.*`， 接口可以大致分为 5 类

| 类型      | 作用                            | 个数 |
| --------- | ------------------------------- | ---- |
| Consumer  | 接收参数做计算，无返回          | 8    |
| Supplier  | 生成数据，对象                  | 5    |
| Predicate | 根据参数做返回 Boolean 值的计算 | 5    |
| Function  | 接受参数，返会计算值            | 17   |
| Operator  | 接受数据并返回计算值            | 8    |

## 示例

### Consumer

拿到参数， 运算， 无返回值：

```java
Stream.of("a", "b").forEach(System.out::println);
```

它有一些变种，比如 BiConsumer, Bi 是 Binary 的缩写，表示复数， 两个的意思。这里表示 Consumer 接收两个参数：

```java
BiConsumer<String, String> biConsumer = (name, action) -> {
    System.out.println(String.format("Name: %s is %s ing...", name, action));
};

biConsumer.accept("Jack", "run");
```

其他变种，比如 DoubleConsumer, 只接受 Double 做参数， 类似的还有 LongConsumer, IntConsumer 等，限制一样：

```java
DoubleConsumer consumer = (val) -> {
    System.out.println("Val: " + val);
};

consumer.accept(1.0); // output: Val: 1.0
// consumer.accept("test"); - 编译报错
// consumer.accept(1.0L); - 编译报错
```

还有一类变种，比如 ObjDoubleConsumer, ObjIntConsumer 和 ObjLongConsumer， 表示接收两个参数，但是其中一个是对象类型的：

```java
// 设计一个 lambda， 接受 person 对象和 int 值，并用 int 对 person 的年龄 field 赋值

class Person {
    String name;
    int age;
    // 省略 getter/setter/toString 方法
}

ObjIntConsumer<Person> consumer = Person::setAge;
Person p = new Person("Jack");
consumer.accept(p, 30);
System.out.println(p);
// output: Person{name='Jack', age=30}
```

大多数 *Consumer 接口看书中还有另外一个方法，叫 `andThen()` 可以达到组合拳的效果

```java
// 定义两个 lambda, 一个做大写转化，一个做小写转化。 就是他的这个级联的语法总有一种很奇葩的感觉，要先写 andThan 再写 accept 才合法
Consumer<String> consumer01 = (val) -> {
    String toUp = val.toUpperCase();
    System.out.print(toUp);
};

Consumer<String> consumer02 = (val) -> {
    String toUp = val.toLowerCase();
    System.out.println(toUp);
};

Stream.of("a", "B").forEach(consumer01.andThen(consumer02).accept);
// output:
// Aa
// Bb
```

### Supplier

生成数据并返回，和工厂方法很像

```java
Supplier supplier = Math::random;
System.out.println(supplier.get());
System.out.println(supplier.get());
// output: 0.21716622238340733
// output: 0.06868488591912514
```

和 Consumer 一样，他也有指定返回类型的 type, 像 BooleanSupplier, DoubleSupplier, IntSupplier 和 LongSupplier

### Predicate

接受参数，然后再 lambda 中计算，得出一个 Boolean 的结果值

```java
// 对准备的 3 个 string 做过滤，输出空字串的个数
long count = Stream.of("a", "  ", "B").filter(String::isBlank).count();
System.out.println(count);
// output: 1
```

filter 中接受的就是 Predicate 类型的表达式，如果计算结果为 true，则保留参数对象，否则过滤掉。

对应的它也有多个变种形式，变种的处理方式和前面的雷同： BiPredicate, DoublePredicate, IntPredicate 和 LongPredicate。

除此之外，大多数的 *Predicate 接口中除了 test() 外还有 and(), negate()， or() 和 isEqual() 方法

```java
Predicate<String> checkLength = val -> val.length() > 5;
Predicate<String> startWith = val -> val.startsWith("B");
List<String> ret = Stream.of("Apple", "Banana", "BBC").filter(checkLength.and(startWith)).collect(Collectors.toList());
System.out.println("-------------- Test AND result--------------");
ret.forEach(System.out::println);

ret = Stream.of("Apple", "Banana", "BBC").filter(checkLength.or(startWith)).collect(Collectors.toList());
System.out.println("-------------- Test OR result--------------");
ret.forEach(System.out::println);

ret = Stream.of("Apple", "Banana", "BBC").filter(checkLength.negate()).collect(Collectors.toList());
System.out.println("-------------- Test NEGATE result--------------");
ret.forEach(System.out::println);

ret = Stream.of("Apple", "Banana", "BBC").filter(Predicate.not(checkLength)).collect(Collectors.toList());
System.out.println("-------------- Test NOT result--------------");
ret.forEach(System.out::println);

System.out.println("-------------- Test EQUALS result--------------");
System.out.println(Predicate.isEqual("abc").test("abc"));

// output:
-------------- Test AND result--------------
Banana
-------------- Test OR result--------------
Banana
BBC
-------------- Test NEGATE result--------------
Apple
BBC
-------------- Test NOT result--------------
Apple
BBC
-------------- Test EQUALS result--------------
true
```

### Function

接受参数，计算并返回所得的值

```java
// 接收一个字符串并将其转化成 integer 类型
Function<String, Integer> func = Integer::valueOf;
int ret = func.apply("100");
System.out.println(ret);
// output: 100
```

同样，他也有变种，而且特别多

| name                 | 作用                          |
| -------------------- | ----------------------------- |
| BiFunction           | 接收两个参数, 返回值类型自定  |
| DoubleFunction       | 接收 Double 参数，返回值自定  |
| IntFunction          | 接收 Int 参数，返回值自定     |
| LongFunction         | 接收 Long 参数，返回值自定    |
| DoubleToIntFunction  | 接收 Double 返回 Int          |
| DoubleToLongFunction | 接收 Double 返回 Long         |
| IntToDoubleFunction  | 接收 Int 返回 Double          |
| IntToLongFunction    | 接收 Int 返回 Long            |
| LongToDoubleFunction | 接收 Long 返回 Double         |
| LongToIntFunction    | 接收 Long 返回 Int            |
| ToDoubleFunction     | 接收参数类型自定，返回 Double |
| ToIntFunction        | 接收参数类型自定，返回 Int    |
| ToLongFunction       | 接收参数类型自定，返回 Long   |
| ToIntBiFunction      | 接收两个参数, 返回 Int        |
| ToLongBiFunction     | 接收两个参数, 返回 Long       |
| ToDoubleBiFunction   | 接收两个参数, 返回 Double     |

相比其他的几个 *Function 接口， BiFunction 和 Function 要更特殊一点，他们除了最基本的 apply() 之外还有一些额外的方法。

Function 和 BiFunction 还有一个相同的 `andThen()` 方法，他会再前一个返回值的基础上再做计算

```java
// 对 Stream 中的数进行 x 100 -10 操作
Function<Integer, Integer> funcx100 = val -> val * 100;
Function<Integer, Integer> funcMinus10 = val -> val - 10;
List<Integer> ret = Stream.of(1, 2).map(funcx100.andThen(funcMinus10)).collect(Collectors.toList());
ret.forEach(System.out::println);
// output:
// 90
// 190
```

初此之外 Function 还有几个特殊的方法， compose() 他是在 apply() 之前执行的，注意泛型返回值的承接

```java
// 设计两个 lambda 函数，将测试字符串中的数字部分抽出来，并格式化
Function<String, Integer> funcComp = val -> {String intVal = val.split(":")[1];
return Integer.valueOf(intVal);};
Function<Integer, String> func = val -> "[" + val + "]";
List<String> ret = Stream.of("Jack:30", "Jerry:18").map(func.compose(funcComp)).collect(Collectors.toList());
ret.forEach(System.out::println);
```

Function 还有一个 identity() 方法，传入什么返回什么，完全不能领会它有什么用，到是网上一些例子中，可以用来快速生成 map 的用法让人挺印象深刻的

```java
Map<String, Integer> map = Stream.of("i", "love", "you").collect(Collectors.toMap(Function.identity(), String::length));
System.out.println(map);
// output:
// {love=4, i=1, you=3}
```

### Operator

一个计算表达式，最基本的类型为 UnaryOperator， 翻译为 `一元表达式`, 它是 Function 的一个子类, 可以看成是定制版/特殊形式的 Function，只用于计算，看网上的例子貌似是这样

```java
UnaryOperator<Integer> operator = val -> val ^ 2;
System.out.println(operator.apply(4));
// output: 6
```
与之类似的还有 LongUnaryOperator 表示 long 类型的一元运算， 同理推至 IntUnaryOperator， DoubleUnaryOperator。

DoubleBinaryOperator 两个 Double 类型数据的运算，同理推至 IntBinaryOperator 和 LongBinaryOperator。

在 Operator 一族中，比较特殊的是 BinaryOperator, 他的方法中有两个计算最值的方法 `minBy()` 和 `maxBy()`

```java
BinaryOperator<Integer> max = BinaryOperator.maxBy(Comparator.naturalOrder());
System.out.println(max.apply(1,2));

BinaryOperator<Integer> min = BinaryOperator.minBy(Comparator.naturalOrder());
System.out.println(min.apply(1, 2));
//output: 2 1
```




