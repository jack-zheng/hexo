---
title: 适配器模式和外观模式
date: 2021-04-09 12:40:47
categories:
- HFDP
tags:
- adapter pattern
- facade pattern
- 适配器模式
- 外观模式
---

> **The Adapter Pattern** converts the interface of a class into another interface the clients expect.  Adapter lets classes work together that couldn’t otherwise because of incompatible interfaces.
> 适配器模式将一种接口类型转化为另一种，他用来解决类中的类型适配问题

> **The Facade Pattern** provides a unified interface to a set of interfaces in a subsytem.  Facade defines a higher-level interface that makes the subsystem easier to use.
> 提供一套更 high-level 的接口简化子系统调用

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

## Facade(外观) Pattern

假设我们要组一套家庭影院，我们有好多设备，比如投影仪，音响，爆米花机，DVD 等。我们每次想要看一场电影需要做如下事情

1. 开启 爆米花 机
2. 开始爆米花
3. 开启影响
4. 设置音量
5. 开启投影仪
6. 摄制亮度
7. 开启 DVD
8. 塞入光盘
9. ...

而且等我们看完了，我们还需要逐个将上面的设备关掉，一套下来，可能以后再也不看电影了。

Facade 模式就是用来解决这种问题的。

> A facade not only simplifies an interface, it decouples a client from a subsystem of components.
> Facades and adapters may wrap multiple classes, but a facade’s intent is to simplify, while an adapter’s is to convert the interface to something different.
> 外观模式不仅仅是简化接口，同时他还将子系统和客户端解耦了
> Facade 和 Adapter 都会在类外面包一层，但是 Facade 是为了简化，而 Adapter 是为了转换

为了简化代码，我们一拿 DVD 和投影仪举例

```java
// Player 类表示 DVD 机的开/关/放电影功能
public class DvdPlayer {
    public void startPlayer() {
        System.out.println("Start DVD Player...");
    }

    public void endPlayer() {
        System.out.println("End DVD Player...");
    }

    public void playMovie(String name) {
        System.out.println("Show movie: " + name + " ...");
    }
}

// 表示屏幕功能
public class Screen {
    public void downScreen() {
        System.out.println("Down screen...");
    }

    public void upScreen() {
        System.out.println("Up screen...");
    }
}

// 家庭影院简化版
public class HomeTheaterFacade {
    private Screen screen;
    private DvdPlayer player;

    public HomeTheaterFacade(Screen screen, DvdPlayer player) {
        this.screen = screen;
        this.player = player;
    }

    public void startMovie(String name) {
        screen.downScreen();
        player.startPlayer();
        player.playMovie(name);
    }

    public void endMovie() {
        screen.upScreen();
        player.endPlayer();
    }
}

// 客户端播放和结束放映
public class Client {
    public static void main(String[] args) {
        HomeTheaterFacade facade = new HomeTheaterFacade(new Screen(), new DvdPlayer());
        facade.startMovie("<<NeZha>>");
        facade.endMovie();
    }
}

// Down screen...
// Start DVD Player...
// Show movie: <<NeZha>> ...
// Up screen...
// End DVD Player...
```

其实说是 Facade 模式，但是我这里平时经常会用到，只不过我一般把这种类型的东西叫做 Util 或者 Action 类。封装一些经常使用的方法，感觉效果上还是很相似的。

## The Principle of Least Knowledge

这个规则是说，我们在写代码的时候要尽量减少涉及到多种返回值类型的链式调用。

> Principle of Least Knowledge - talk only to your immediate friends.

在你的代码中，你只能调用下列对象的方法：

* 对象本身
* 通过方法参数传入的对象
* 任何在本类中创建的对象
* 任何本对象的 field

这样做可以减少两个对象之间的 dependencies 但是同时也有一个弊端，你需要写跟多的代码，项目会变得更大，还可能会性能下降。
