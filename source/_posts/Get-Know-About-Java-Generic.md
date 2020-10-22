---
title: Java 拾遗之 泛型
date: 2020-08-07 17:36:37
categories:
- 编程
tags:
- 拾遗
- 泛型
---

记录一下泛型的定义，历史，使用案例等。素材只要来源于 On Java 8, Thinking in Java 和 Effective Java。

## 历史

1.5 版本引入，主要动机是支持 Collection 类

## 泛型方法

把这一块放到最前面时为了避免理解上的误区，泛型方法和泛型类，泛型接口没有从属关系，就算时普通的 Utils 方法也可以声明泛型方法

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

## 工作中遇到的问题

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