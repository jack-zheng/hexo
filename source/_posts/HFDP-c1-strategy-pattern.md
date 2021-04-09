---
title: 策略模式
date: 2021-04-09 12:39:57
categories:
- HFDP 
tags:
- strategy pattern
---

Design Principle: 

* Identify the aspects of your application that vary and separate them from what stays the same - 将频繁改变的部分从系统中抽离出来
* Program to an interface, not an implementation - 面向接口编程而非实现
* Favor composition over inheritance - 组合优于继承

> The **Strategy Pattern** defines a family of algorithms, encapsulates each one, and makes them interchangeable.  Strategy lets the algorithm vary independently from clients that use it.  
> 
> 定义了一族算法，将它和调用方解偶

## 案例

已有一段鸭子模拟器代码，所有鸭子都有一个基类，基类中定义了一些常用方法，并给了实现。只有 display() 是需要每个类定制的，声明成了 abstract 并在子类中各自实现

```txt
            +--------------------+                                                                                                                  
            |  Duck              |                                                                                                                  
            |--------------------|                                                                                                                  
            | swim()             |                                                                                                                  
            | display()          |                                                                                                                  
            | quack()            |                                                                                                                  
            | //other methods    |                                                                                                                  
            +--------------------+                                                                                                                  
               ^                                                                                                                                    
               |                                                                                                                                    
               |---------------                                                                                                                     
               |              |                                                                                                                     
               |              |                                                                                                                     
+----------------+     +----------------+                                                                                                           
|  MallardDuck   |     |  RedHeadDuck   |                                                                                                           
|--------------- |     |--------------- |                                                                                                           
|display()       |     |display()       |                                                                                                           
|                |     |                |                                                                                                           
|                |     |                |                                                                                                           
+----------------+     +----------------+                                                                                                           
```

某个 release，客户突然想要看到鸭子能飞。。。小码农一拍脑袋：嗨，这还不简单。就直接在基类里添加了 fly() 的实现。然后转头就去写其他功能了。

```txt
            +--------------------+                                                                                                                  
            |  Duck              |                                                                                                                  
            |--------------------|                                                                                                                  
            | swim()             |                                                                                                                  
            | display()          |                                                                                                                  
            | quack()            |                                                                                                                  
            | fly()              |                                                                                                                  
            | //other methods    |                                                                                                                  
            +--------------------+                                                                                                                  
               ^                                                                                                                                    
               |                                                                                                                                    
               |---------------                                                                                                                     
               |              |                                                                                                                     
               |              |                                                                                                                     
+----------------+     +----------------+                                                                                                           
|  MallardDuck   |     |  RedHeadDuck   |                                                                                                           
|--------------- |     |--------------- |                                                                                                           
|display()       |     |display()       |                                                                                                           
|                |     |                |                                                                                                           
|                |     |                |                                                                                                           
+----------------+     +----------------+                                                                                                           
```

三天后，Sales 一个夺命连环 call 打到小码农这儿，说自己给客户演示功能的时候，突然发现，程序里的橡皮鸭子起飞了！！！这还了得，小码农打开了 IDE 看了橡皮鸭子的代码

```txt
            +--------------------+                                                                                                                  
            |  Duck              |                                                                                                                  
            |--------------------|                                                                                                                  
            | swim()             |                                                                                                                  
            | display()          |                                                                                                                  
            | quack()            |                                                                                                                  
            | fly()              |                                                                                                                  
            | //other methods    |                                                                                                                  
            +--------------------+                                                                                                                  
               ^                                                                                                                                    
               |                                                                                                                                    
               |----------------------------------------                                                                                            
               |              |                        |                                                                                            
               |              |                        |                                                                                            
+----------------+     +----------------+     +----------------+                                                                                    
|  MallardDuck   |     |  RedHeadDuck   |     |  RubberDuck    |                                                                                    
|--------------- |     |--------------- |     |--------------- |                                                                                    
|display()       |     |display()       |     |display()       |                                                                                    
|                |     |                |     |quack(){        |                                                                                    
|                |     |                |     |override quack  |                                                                                    
+----------------+     +----------------+     |}               |                                                                                    
                                              |                |                                                                                    
                                              +----------------+                                                                                    
```

好嘛，原来基类加完 fly() 方法之后忘了重写橡皮鸭子这个子类了，还能咋整，要不重写一下 fly() 呗

```txt
+-------------------------+                                                                                                                           
|    RubberDuck           |                                                                                                                           
| ----------------------  |                                                                                                                           
|                         |                                                                                                                           
| display()               |                                                                                                                           
| quack(){// rubber quck }|                                                                                                                           
| fly(){ // do nothing }  |                                                                                                                           
|                         |                                                                                                                           
+-------------------------+ 
```

不过转念一下，这也不是办法啊，不然哪天要新加一个什么木头鸭子，不是还得和相比鸭子一样再来一遍？于是小码农就寻思着，把 quack 和 fly 整成接口？

把这个想法和同事小美一说，小美立马就不乐意了，这个那个产品老几十个类，把这两个玩意儿整成接口，你不得在这几十个类里面都改一遍，你改啊？！

小码农想也是，但是没什么法子，就去问自己的师傅大能耐了。小码农把情况和大能耐说了一遍。

'耐哥，就上面的情况，听说你精通设计模式，这种情况，有法儿吗'

'咱先不说设计模式的事儿，写代码最起码得掌握一个原则：先把频繁变动的代码和不变的拆分开了后我们再说事儿'

按照耐哥的思路，小码农把 quack 和 fly 从鸭子类里面单独拿出来写成接口，并为他们做了各种实现

```txt
                  +-------------------------+                                                                                                       
                  |    <<Interface>>        |                                                                                                       
                  |    FlyBehavior          |                                                                                                       
                  | ----------------------  |                                                                                                       
                  | fly()                   |                                                                                                       
                  |                         |                                                                                                       
                  +-------------------------+                                                                                                       
                        ^                                                                                                                           
                        |---------------------                                                                                                      
                        |                    |                                                                                                      
                        |                    |                                                                                                      
 +-------------------------+   +-------------------------+                                                                                          
 |    FlyWithWings         |   |    FlyNoWay             |                                                                                          
 | ----------------------  |   | ----------------------  |                                                                                          
 | fly(){ // duck flying } |   | fly(){ // can't fly }   |                                                                                          
 |                         |   |                         |                                                                                          
 +-------------------------+   +-------------------------+ 
```

既然将这两个属性从基类中抽出来了，那原来的基类中我们再用这两个新的接口代替，并新建方法调用接口实现 fly 和 quack 的功能

```txt
 +--------------------+                                                                                                                             
 |  Duck              |                                                                                                                             
 |--------------------|                                                                                                                             
 | FlyBehavior fb     |                                                                                                                             
 | QuackBehavior qb   |                                                                                                                             
 |                    |                                                                                                                             
 |--------------------|                                                                                                                             
 | swim()             |                                                                                                                             
 | display()          |                                                                                                                             
 | perormQuack()  <----------- qb.quack();                                                                                                          
 | performFly()   <----------- fb.fly();                                                                                                            
 | //other methods    |                                                                                                                             
 |                    |                                                                                                                             
 +--------------------+                     
```

按照这个逻辑，小码农重构了以后的代码

```java
// Fly 接口及其实现
public interface FlyBehavior {
    void fly();
}

public class FlyNoWay implements FlyBehavior {
    public void fly() { System.out.println("I can't fly..."); }
}

public class FlyWithWings implements FlyBehavior {
    public void fly() { System.out.println("I can fly..."); }
}

// Quack 接口及其实现
public interface QuackBehavior {
    void quack();
}

public class Quack implements QuackBehavior {
    public void quack() { System.out.println("Quack...Quack..."); }
}

public class Squack implements QuackBehavior {
    public void quack() { System.out.println("Squack..."); }
}

public class MuteQuack implements QuackBehavior {
    public void quack() { System.out.println("<< Silence >>"); }
}

// Duck 抽象方法
public abstract class Duck {
    QuackBehavior quackBehavior;
    FlyBehavior flyBehavior;

    public void performQuack() { quackBehavior.quack(); }
    public void performFly() { flyBehavior.fly(); }
    public void swim() { System.out.println("All ducks float, even decoys!"); }
    // other shared methods
    public abstract void display();
}

// 具体种类的鸭子类
public class MallardDuck extends Duck {
    public MallardDuck() {
        this.flyBehavior = new FlyWithWings();
        this.quackBehavior = new Quack();
    }

    @Override
    public void display() {
        System.out.println("I'm a Mallard duck...");
    }
}

// 测试类
public class MiniDuckSimulator {
    public static void main(String[] args) {
        MallardDuck mallardDuck = new MallardDuck();
        mallardDuck.performQuack();
        mallardDuck.performFly();
    }
}

// Quack...Quack...
// I can fly...
```

大功告成，小码农拿着自己的成果找了大能耐 review 代码

'不错不错，不过你有没有想过你的代码还可以更灵活，只要在 Duck 里面添加一个 set 方法，你就可以动态的改变鸭子的行为了哟'

小码农一想，立马知道了其中的关键，修改了代码

```txt
 +--------------------+                                                                                                                             
 |  Duck              |                                                                                                                             
 |--------------------|                                                                                                                             
 | FlyBehavior fb     |                                                                                                                             
 | QuackBehavior qb   |                                                                                                                             
 |                    |                                                                                                                             
 |--------------------|                                                                                                                             
 | swim()             |                                                                                                                             
 | display()          |                                                                                                                             
 | perormQuack()  <----------- qb.quack();                                                                                                          
 | performFly()   <----------- fb.fly();                                                                                                            
 | //other methods    |                                                                                                                             
 | setFlyBehavior()   |                                                                                                                             
 | setQuackBehavior() |                                                                                                                             
 |                    |                                                                                                                             
 +--------------------+                     
```

```java
public abstract class Duck {
    QuackBehavior quackBehavior;
    //...
    //...
    public void setQuackBehavior(QuackBehavior quackBehavior) { this.quackBehavior = quackBehavior; }
    public void setFlyBehavior(FlyBehavior flyBehavior) { this.flyBehavior = flyBehavior; }
}
```

正巧项目组想要一个新的鸭子模型，能够动态的改变飞行模式，在氪金以前是不能飞的，但是氪金以后能弹射起步。。。正好赶巧了，小码农三下五除二就实现了功能

```java
public class RocketFly implements FlyBehavior {
    public void fly() {  System.out.println("Fly in rocket speed..."); }
}

public class MiniDuckSimulator {
    public static void main(String[] args) {
        ModelDuck modelDuck = new ModelDuck();
        modelDuck.performFly();

        modelDuck.setFlyBehavior(new RocketFly());
        modelDuck.performFly();
    }
}

// I can't fly...
// Fly in rocket speed...
```

小码农哼着小曲儿锁了屏，转头跑到走廊上掏出手机开启了王者农药。。。
