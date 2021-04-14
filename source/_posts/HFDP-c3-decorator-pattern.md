---
title: 装饰器模式
date: 2021-04-13 19:58:44
categories:
- HFDP 
tags:
- decorator pattern
- 装饰器模式
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

## 实现

## 该模式在 JDK 中的应用
