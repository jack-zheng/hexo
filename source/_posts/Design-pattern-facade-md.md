---
title: 外观模式
date: 2021-08-10 12:39:00
categories:
- 设计模式 
tags:
- Design Pattern
---

> **The Facade Pattern** provides a unified interface to a set of interfaces in a subsytem.  Facade defines a higher-level interface that makes the subsystem easier to use.
> 提供一套更 high-level 的接口简化子系统调用

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
