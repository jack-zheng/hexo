---
title: 代理模式
date: 2020-10-12 14:05:30
categories:
- 设计模式 
tags:
- Design Pattern
---

记录一下代理模式的学习路径。代理模式常用的两种形式：静态代理，动态代理。其中，动态代理在两个国民级框架 mybatis 和 spring 中都有用到。

代理模式的定义：为其他对象提供一种代理以控制对这个对象的访问。在某些情况下，一个对象不适合或者不能直接引用另一个对象，而代理对象可以在客户端和目标对象之间起到中介的作用。

通过代理模式我们可以：

1. 隐藏委托类的具体实现
2. 客户和委托类解偶，在不改变委托类的情况下添加额外功能

插入类图 Here...

这里我们举一个生活中常见的例子，外卖小哥。在这个情境下，外卖小哥就是我们的代理。帮我们执行买餐这个动作。同时作为扩展，它还可以帮我们买烟买水，倒垃圾等。。。虽然我不提倡这种做法，只用于举例，无伤大雅。

## 静态代理

公共接口，用来点单

```java
public interface Order {
    void order();
}
```

客户实现，这个类代表叫外卖的人

```java
public class Customer implements Order {

    @Override
    public void order() {
        System.out.println("Order and pay money...");
    }
}
```

外卖小哥类

```java
public class DeliveryGuy implements Order {
    private Customer customer;

    public DeliveryGuy(Customer customer) {
        this.customer = customer;
    }

    @Override
    public void order() {
        customer.order();
    }
}
```

客户端调用


```java
public class Client {
    public static void main(String[] args) {
        Customer customer = new Customer();
        Order order = new DeliveryGuy(customer);
        order.order();
    }
}
```

优点：

1. 简单直接
2. 解偶
3. 代理类扩展业务方便
   
缺点：

每个业务都需要一个代理类，冗余代码很多

## 动态代理

常见的有两种方式：JDK 原生动态代理和 CGLib 动态代理，这里只介绍第一种。

JDK 根据代理模式的特性，制定了一套规范，参照他的规范，可以在很方便的在运行时产生代理类代码，而不需要在编译器写源码，更方便，当然代价就是增加了学习成本，代码不像之前那么一目了然了。实现时主要依赖两个 reflect 下的原生类 Proxy 和 InvocationHandler。

```java
public class LogHandler implements InvocationHandler {
    private Object target;

    public LogHandler(Object target) {
        this.target = target;
    }

    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        before();
        Object obj = method.invoke(target, args);
        after();
        return obj;
    }

    private void before() {
        System.out.println("buy something...");
    }

    private void after() {
        System.out.println("take out the trash...");
    }
}

public class Client {
    public static void main(String[] args) {
        LogHandler logHandler = new LogHandler(new Customer());
        Order order = (Order) (Proxy.newProxyInstance(Order.class.getClassLoader(), new Class[] {Order.class}, logHandler));
        order.order();
    }
}
```

LogHandler 的实现中 invok 的是要要特别注意一下，method.invoke 的参数是**target**。我一开始直接把 proxy 但参数传入了，排查了好久 （；￣ェ￣）

中的来说没什么难度，最花时间的部分是熟悉这种使用方式，第一次理解起来可能花点时间。


