---
title: 装饰器模式
date: 2021-04-13 19:58:44
categories:
- 设计模式 
tags:
- Design Pattern
---

**Design Principle:** Classes should be open for extension, but closed for modification.

> **The Decorator Pattern** attaches additional responsibilities to an object dynamically. Decorators provide a flexible alternative to subclassing for extending functionality.
> 装饰器模式可以让你的对象动态添加特性

## 引子

Starbuzz 设计了一款软件卖咖啡，但是设计太烂了，导致类膨胀了。如果是你你会怎么整？

原始设计, 有一个基类 Beverage，然后各种子类比如浓缩咖啡，美式等。但是光咖啡还不够，我们还可以加入各种调味料，比如抹茶，奶盖等。每加入新的调料都是一种新的类，比如浓缩+抹茶。类数量成指数上升

```txt
                     +-----------------+                                                                                                              
                     |    Beverate     |                                                                                                              
                     |---------------- |                                                                                                              
                     | description     |                                                                                                              
                     |---------------- |                                                                                                              
                     | getDescription()|                                                                                                              
                     | cost()          |                                                                                                              
                     |                 |                                                                                                              
                     +-----------------+                                                                                                              
                       ^        ^   ^                                                                                                                 
                       |        |   |---------------                                                                                                  
                       |        |                  |                                                                                                  
          +--------------+   +--------------+      |                                                                                                  
          |  Espreesso   |   |  DarkRoast   |      |                                                                                                  
          |--------------|   |--------------|     ...                                                                                                 
          |  cost()      |   |  cost()      |                                                                                                         
          +--------------+   +--------------+                                                                                                         
            ^          ^                                                                                                                              
            |          |                                                                                                                              
            |          |                                                                                                                              
+-----------------+    |                                                                                                                              
|EspreessoWithMilk|    |                                                                                                                              
|-----------------|   ...                                                                                                                             
|  cost()         |                                                                                                                                   
+-----------------+                                                                                                                              
```

随之设计师又想到了另一种解决方案，可以将所有的属性和调味品设置成属性放到基类中，然后通过 flag 知道是否含有某种调味品，然后在子类中通过设置这些 flag 的值，定制 cost 结果

```txt
+----------------+                                                                                                                                 
|    Beverage    |                                                                                                                                 
|----------------|                                                                                                                                 
| description    |                                                                                                                                 
| soy            |                                                                                                                                 
| mocha          |                                                                                                                                 
| ...            |                                                                                                                                 
|----------------|                                                                                                                                 
| hasSoy()       |                                                                                                                                 
| hasMocha()     |                                                                                                                                 
| ...            |                                                                                                                                 
|                |                                                                                                                                 
|                |                                                                                                                                 
|                |                                                                                                                                 
|                |                                                                                                                                 
|                |                                                                                                                                 
+----------------+                                                                                                                                 
```

对应的代码实现

```java
class Beverage {
    // declear condiment
    public double cost() {
        double condimentCost = 0.0;
        if (hasMilk()) {
            codimentCost += milkCost;
        }
        if (hasSoy()) {
            // ...
        }
        // ...
    }
}

public class DarkRoast extends Beverage {
    public double cost() {
        return 1.00 + super.cost();
    }
}
```

这样做虽然避免的类爆炸式增长，但是导致了新的问题。比如每当调料价格变动，你就必须得改变老得代码。新加调料，你还得修改之前的 if 逻辑。而且如果有新的饮料，比如茶，那么这个继承关系在逻辑层面上就不是很合理了。为了解决类似的问题，我们引入装饰者模式

## UML

* 每个 Component 可以自己调用自己，也可以被 Decorator 包裹
* 每个 Decorator 都持有一个 Component 的引用
* ConcrateComponent 是 Component 的具体实现

```txt
                    +--------------+                            
                    |  Component   |                            
                    |--------------|------------------|         
                    | +operation() |                  |         
                    |              |                  |         
                    +--------------+                  |         
                    ^             ^                   |         
                    |             |                   |         
                    |             |                   |         
+---------------------+          +--------------+     |         
|  ConcreateComponent |          |  Decorator   |<>---|         
|---------------------|          |--------------|               
| +operation()        |          | +operation() |               
|                     |          |              |               
+---------------------+          +--------------+               
                                  ^         ^                   
                                  |         |                   
                                  |         |                   
              +---------------------+    +---------------------+
              | ConcreateDecoratorA |    | ConcreateDecoratorA |
              |---------------------|    |---------------------|
              | +operation()        |    | +operation()        |
              | +addBehavior()      |    | +addBehavior()      |
              +---------------------+    +---------------------+
```

## 实现

```java
// 基类实现
public abstract class Beverage {
    String description = "Unknown Beverage";

    public String getDescription() {
        return description;
    }

    public abstract double cost();
}

// 实例咖啡实现
public class Espresso extends Beverage {
    public Espresso() {
        description = "Espresso";
    }

    @Override
    public double cost() {
        return 1.99;
    }
}

// 装饰类的基类
public abstract class CondimentDecorator extends Beverage {
    @Override
    public abstract String getDescription();
}

// 装饰类实现，装饰类会持有基类引用，并对方法做扩展
public class Mocha extends CondimentDecorator {
    Beverage beverage;

    public Mocha(Beverage beverage) {
        this.beverage = beverage;
    }

    @Override
    public double cost() {
        return beverage.cost() + .20;
    }

    @Override
    public String getDescription() {
        return beverage.getDescription() + ", Mocha";
    }
}

public class Soy extends CondimentDecorator {
    Beverage beverage;
    public Soy(Beverage beverage) {
        this.beverage = beverage;
    }

    @Override
    public double cost() {
        return beverage.cost() + .15;
    }

    @Override
    public String getDescription() {
        return beverage.getDescription() + ", Soy";
    }
}

// 测试类
public class Client {
    public static void main(String[] args) {
        Beverage myBeverage = new Mocha(new Soy(new Espresso()));
        System.out.println(myBeverage.cost());
        System.out.println(myBeverage.getDescription());
    }
}

// 2.3400000000000003
// Espresso, Soy, Mocha
```

问，我现在如果加了双份的抹茶，description 输出时会现实 Mocha, Mocha。那如果我想要他输出 Double Mocha 的话需要怎么做?

按照装饰模式的思路，可以将 description 的实现改为容器，比如 list, 然后在最外层加入一个 CustomizedDescDecorator 截取 description 做整合

## 该模式在 JDK 中的应用

Java 的 I/O 包就使用了装饰器模式。IO 分两种，字节流（Input/OutputStream）和字符流（Reader/Writer）。

以输入字节流 InputStream 为例，继承关系如下

```tx
                                 +-------------+                                                 
                                 | InputStream |                                                 
                                 +-------------+                                                 
                                        ^                                                        
          ------------------------------|------------------------------------------------------  
         |                              |              |                       |              |  
         |                              |              |                       |              |  
 +-----------------+  +-------------------+  +-------------------+  +----------------------+  |  
 | FileInputStream |  | FilterInputStream |  | ObjectInputStream |  | ByteArrayInputStream |  ...
 +-----------------+  +-------------------+  +-------------------+  +----------------------+     
                                   ^                                                             
         --------------------------|-----------------------------------------------              
        |                          |                           |                  |              
        |                          |                           |                  |              
 +---------------------+   +---------------------+    +--------------------+      |              
 | BufferedInputStream |   | DataInputStream     |    | PushbakInputStream |     ...             
 +---------------------+   +---------------------+    +--------------------+                     
```

一开始看岔了，把 FilterInputStream 和 FileInputStream 看成同一个了，所以没能把它和装饰模式匹配起来。在 IO 的实现中，FileInputStream, ObjectInputStream 等即对饮了 ConcreateComponent, 是具体实现。

FilterInputStream 对应了 Decorator, 是修饰器的基类，持有了 inputStream 的引用，而 BufferedInputStream 则为装饰器的具体实现，起到包装 Component 的作用。

BufferedInputStream 使用示例如下：

```java
public class IoDecoratorDemo {
    public static void main(String[] args) {
        ClassLoader classloader = Thread.currentThread().getContextClassLoader();
        InputStream is = classloader.getResourceAsStream("c1_2.xml");
        try (BufferedInputStream bis = new BufferedInputStream(is)) {
            byte data;
            while ((data = (byte) bis.read()) != -1) {
                System.out.print((char) data);
            }

            System.out.println("Finish read...");
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```