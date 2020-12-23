---
title: Java 拾遗之 泛型
date: 2020-08-07 17:36:37
categories:
- 编程
tags:
- 拾遗
- 泛型
---

记录一下泛型的定义，历史，使用案例等。素材主要来源于 On Java 8, Thinking in Java 和 Effective Java。

## 历史

1.5 版本引入，主要动机是支持 Collection 类

## 泛型方法

把这一块放到最前面时为了避免理解上的误区，泛型方法和泛型类，泛型接口没有从属关系，就算是普通的 Utils 方法也可以声明泛型方法

```java
public class GenericUtils {
    public static <T> void printParam(T t) {
        System.out.println(t);
    }
}

@Test
public void test_print() {
    GenericUtils.printParam("Jack");
    GenericUtils.printParam(1);
}

// Output:
// Jack
// 1
```

## 泛型类

即在声明类时添加类型声明，最常见的如 Collection 系列下的 ArrayList

```java
public class ArrayList<E> extends AbstractList<E> implements List<E>, RandomAccess, Cloneable, java.io.Serializable { ... }
```

## 泛型接口

泛型接口只是在接口定义的时候在接口名称后接上类型声明而已。使用 lang 包中自带的 `Supplier` 接口为例，接口在声明时指定类型，并在 get() 方法中指定返回类型。

```java
// 1.8 中引入的接口，充当工厂方法的角色
@FunctionalInterface
public interface Supplier<T> { T get(); }

Supplier<Integer> integerSupplier = () -> (new Random()).nextInt();
System.out.println(integerSupplier.get());
```

## 使用案例

记录一下工作生活中遇到的具体使用案例

### 代码重构

公司代码重构时遇到下面这种情况：

比如原来有个类叫 Background, 重构时为他抽了一个 interface IBackground。但是在替换一些集合相关的代码时出现了不兼容的问题。

```java
// before
List<Background> list = someClass.getBackgroundList();
// what I prefer to, but compile failed
List<IBackground> list = someClass.getBackgroundList();
// what I should do
List<IBackground> list = new ArrayList<>(someClass.getBackgroundList());
// or
List<? extends IBackground> list = someClass.getBackgroundList();
```

上面的这种转换失败就是由泛型转化异常造成的

### 指定泛型返回值为某个类的子类

可以使用泛型方法

```java
public void <T extends Sup>  T getSometing() {};
```

## 返回 Map 类型的泛型方法？

这种用法称为 multi-level wildcards，参考 [这篇](http://www.angelikalanger.com/GenericsFAQ/FAQSections/TypeArguments.html#FAQ101) 文章中的定义

子类现有方法为 

```java
public Map<String, List<Sub>> getResult() {
    new HashMap<String, List<Sub>>();
}
```

想要给他一个抽 interface 类似 `Map<String, List<? extends Sup>> getResult();` 但是会编译错误，需要怎么写？

* [Stack Overflow 精彩解答](https://stackoverflow.com/questions/22806202/java-nested-generic-type)

先说答案，可以使用 `Map<String, ? extends List<? extends Sup>> getResult();` 这样的语法来适配上面说的这种场景

关于这个问题的几个点：

1. List\<Sub\> 并不是 List\<Sup\> 的子类，想要表达子类的概念，Java 使用的是 `List<? extends Sup>` 这样的语法
2. `List<List<?>>` 适配所有的参数类型的 list
3. `List<? extends List<String>>` 适配任何 List 及其子类
4. 两者结合一下 `List<? extends List<?>>` 适配任何 list 及其子类，并且适配所有参数类型

Code sample:

```java
public interface Sup { }

public interface Sub extends Sup { }

public interface Father { 
    Map<String, ? extends List <? extends Sup>> getNestMap(); 
}

public class Son implements Father {
    @Override
    public Map<String, List<Sub>> getNestMap() {
        Map<String, List<Sub>> map =  new HashMap<>();
        List<Sub> list = new ArrayList<>();
        list.add(new Sub());
        map.put("Jack", list);
        return map;
    }
}
```

PS: List 前的 `? extends` 是不可少的，不然 Override 方法会编译错误，应为实现中是用 ArrayList 这个子类实现的，所以接口定义时语意上要有这个声明

### 工作中遇到的问题

```java
// 下面两个方法有没有区别？
public interface WildCardTest {
    <T extends Sup> List<T> getList01();

    List<? extends Sup> getList02();
}
```

没有。。。看了一下这两个方法编译出来的字节码是完全一样的，除了行号。

```bytecode
  // access flags 0x401
  // signature <T::Lcom/playground/genericsample/Sup;>()Ljava/util/List<TT;>;
  // declaration: java.util.List<T> getList01<T extends com.playground.genericsample.Sup>()
  public abstract getList01()Ljava/util/List;

  // access flags 0x401
  // signature ()Ljava/util/List<+Lcom/playground/genericsample/Sup;>;
  // declaration: java.util.List<? extends com.playground.genericsample.Sup> getList02()
  public abstract getList02()Ljava/util/List;
```

但是这两种表达方式在实现的时候还是有区别的，用界限符(?)的这种，要求在集合类型前面也加上界限符。。。

### 向 List<? extends Number> 中添加数据失败

`List<? extends Number> list = new ArrayList<>(); list.add(3);` 向该 list 中添加数据 3 有编译错误。这是应为通过 `List<? extends Number> list` 声明的 list 可以存储 Number 及其子类，效果上来看下面这些声明的集合只是 `? extends Number` 的一部分，那么我们加 3 这个行为在类型一致这个前提下就会有与以上的错误。一般这种声明方式拿到的结果只用于读操作。

```java
List<? extends Number> foo3 = new ArrayList<Number>();  // Number "extends" Number
List<? extends Number> foo3 = new ArrayList<Integer>(); // Integer extends Number
List<? extends Number> foo3 = new ArrayList<Double>();  // Double extends Number
```

### 方法中同时有 Class T 和 T bean 的情况怎么兼容

声明一个 list 类型是 `<? extends Number>` 我们还有一个方法参数列表 `Class<T> clz1, T clz2` 这种情况下怎么兼容。

```java
public class TestGeneric {
    public static void main(String[] args) {
        List<Class<? extends Number>> list = Arrays.asList(Integer.class, Double.class, Long.class);
        // !testGeneric(list.get(0), 1); // compile failed
        testGeneric(Integer.class, 1);
    }

    public static <T> void testGeneric(Class<T> clz1, T clz2) {
        // do something
    }
}
```