---
title: 适配器模式
date: 2021-04-09 12:40:47
categories:
- 设计模式 
tags:
- Design Pattern
---

> **The Adapter Pattern** converts the interface of a class into another interface the clients expect.  Adapter lets classes work together that couldn’t otherwise because of incompatible interfaces.
> 适配器模式将一种接口类型转化为另一种，他用来解决类中的类型适配问题

## 引子

举例一个现实生活中的案例：插头转换器。比如我们的手机是两脚插头，但是家里只有三脚插座，那怎么整？答案是找一个插口转化器，把两脚的转成三脚的即可。adapter 模式的工作方式和这种解决方案完全一致。

```txt
+--------------------+     +-------+     +-------------+                                                                                              
| Your existing      |     |Adaptor|     | Vendor      |                                                                                              
| system             |     |       |     | Class       |                                                                                              
|                    |---> |       |---->|             |                                                                                              
|                    |     |       |     |             |                                                                                              
|                    |     |       |     |             |                                                                                              
+--------------------+     +-------+     +-------------+                                                                                              
```

好了现在让我们用第一章的鸭子来举例子。话说有一天，PM 突然找到小码农说，我们的客户有一个特殊要求，想要一种有着火鸡内在的鸭子。小码农满脸的黑人问号，这也行？但是 PM 不管，说下周就要产品掩饰了，你自己看着办。

```java
/**
* 假设之前的鸭子实现是通过 interface 来做的, 我们现在已有了 Duck，Turkey 的接口定义，以及 Turkey 的实现
* 我们只需要新建一个适配器，实现目标接口的方法，并且适配器持有需要代理的实例。在对应的方法中通过调用实例方法即可
**/

public interface Duck {
    void quack();
    void fly();
}

public interface Turkey {
    void gobble();
    void fly();
}

public class WildTurkey implements Turkey {
    @Override
    public void gobble() {
        System.out.println("Gobble...");
    }

    @Override
    public void fly() {
        System.out.println("Fly a short distance...");
    }
}

public class DuckAdaptor implements Duck {
    private Turkey turkey;

    public DuckAdaptor(Turkey turkey) {
        this.turkey = turkey;
    }

    @Override
    public void quack() {
        turkey.gobble();
    }

    @Override
    public void fly() {
        turkey.fly();
        turkey.fly();
        turkey.fly();
    }
}

public class Client {
    public static void main(String[] args) {
        DuckAdaptor adaptor = new DuckAdaptor(new WildTurkey());
        adaptor.quack();
        adaptor.fly();
    }
}

// Gobble...
// Fly a short distance...
// Fly a short distance...
// Fly a short distance...
```

## UML

```txt
+-----------------+     +---------------+                                                                                                            
|                 |---> |<<Interface>>  |                                                                                                            
|    Client       |     |  Target       |                                                                                                            
|                 |     +---------------+                                                                                                            
+-----------------+             ^                                                                                                                    
                                |                                                                                                                    
                                |                                                                                                                    
                                |                                                                                                                    
                         +-------------+      +-----------------+                                                                                    
                         |   Adapter   |      |   Adaptee       |                                                                                    
                         |-------------| ---->|-----------------|                                                                                    
                         | request()   |      | specialRequest()|                                                                                    
                         |             |      |                 |                                                                                    
                         +-------------+      +-----------------+                                                                                    
```

这里展示的是类 adaptor, 还有一种 class adaptor，但是由于 Java 是单继承的，语法上就不能实现 class adaptor 的这种定义。不过支持多继承的语言是可以实现的。

```txt
+-----------------+      +-------------+     +-----------------+                                                                                     
|                 |--->  |   Target    |     |   Adaptee       |                                                                                     
|    Client       |      |-------------|     |-----------------|                                                                                     
|                 |      | request()   |     | specialRequest()|                                                                                     
+-----------------+      |             |     |                 |                                                                                     
                         +-------------+     +-----------------+                                                                                     
                                     ^         ^                                                                                                     
                                     |         |                                                                                                     
                                     |         |                                                                                                     
                                     |         |                                                                                                     
                                    +-------------+                                                                                                  
                                    |   Adapter   |                                                                                                  
                                    |-------------|                                                                                                  
                                    | request()   |                                                                                                  
                                    |             |                                                                                                  
                                    +-------------+                                                                                                  
```

## 项目中可能用到 Adapter 的地方

比如老代码中，很多地方会用到 Enumeration 类，但是在 JDK 1.2 时就推出了 Iterator 接口代替他。如果新的代码都时采用的 Iterator 做迭代，那么怎么兼容老的 Enumeration 呢。

可以新建一个 adapter 类，实现 iterator 接口，持有 enumeration 实现。在 client 中通过这个 adaptor 访问 enumeration。

```java
public class AdapterToEnum {
    public static void main(String[] args) {
        Vector<String> strings = new Vector<>();
        strings.add("a");
        strings.add("b");
        strings.add("c");
        strings.add("d");

        Enumeration<String> enumeration = strings.elements();

        IteratorAdapter adapter = new IteratorAdapter(enumeration);
        while (adapter.hasNext()) {
            System.out.println(adapter.next());
        }
    }
}

// PS: 书中视角 EnumerationAdapter, 但是我觉得不是应该叫 迭代器适配器 才合适吗？应该是我和作者对数据流向的理解相反，但是对这里的使用没有造成影响
class IteratorAdapter implements Iterator<String> {
    private final Enumeration<String> enumeration;

    public IteratorAdapter(Enumeration<String> enumeration) {
        this.enumeration = enumeration;
    }

    @Override
    public boolean hasNext() {
        return enumeration.hasMoreElements();
    }

    @Override
    public String next() {
        return enumeration.nextElement();
    }
}
```
