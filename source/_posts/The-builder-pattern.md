---
title: 生成器/建造者模式
date: 2020-10-13 10:05:37
categories:
- 编程
tags:
- 设计模式
---

在看 mybatis 源码的 xml 解析部分的时候，发现里面重度使用了生成器模式，特此整理一下。

定义：将一个复杂对象的构建与它的表示分离，使得同样的构建过程可以创建不同的表示。建造者模式是一种对象创建型模式。

适用场景：构造函数多，且参数可选的情况下，建议使用。

案例，创建一个有五个属性的 computer 对象

## Java 简化版

创建方案有两种：

```java
// 方案一， 重载构造函数
public class Computer {
    public Computer(String cpu, String ram) {
        this(cpu, ram, 0);
    }

    public Computer(String cpu, String ram, int usbCount) {
        this(cpu, ram, usbCount, "罗技键盘");
    }

    public Computer(String cpu, String ram, int usbCount, String keyboard) {
        this(cpu, ram, usbCount, keyboard, "三星显示器");
    }

    public Computer(String cpu, String ram, int usbCount, String keyboard, String display) {
        this.cpu = cpu;
        this.ram = ram;
        this.usbCount = usbCount;
        this.keyboard = keyboard;
        this.display = display;
    }
    //...
}

// 方案二， 使用 new + set
public class Computer {
    public void setCpu(String cpu) {
        this.cpu = cpu;
    }
    // 省略其他 set 方法
}
```

1. 使用构造函数 - 弊端：参数过多，增加阅读，调用复杂度
2. 使用 new + set - 弊端：不连续，可能少设置属性什么的

Java 简化版方案：

1. 在对象内部创建一个 public 的内部静态类 Builder
2. 复制一份对象的属性到 Builder 中
3. Builder 提供 set 方法
4. 在对象内部添加一个私有的构造函数，参数为 Builder
5. 通过链式调用 Builder 创建对象

```java
@Data
public class Computer {
    private String cpu;//必须
    private String ram;//必须
    private int usbCount;//可选
    private String keyboard;//可选
    private String display;//可选

    private Computer(Builder builder) {
        this.cpu = builder.cpu;
        this.ram = builder.ram;
        this.usbCount = builder.usbCount;
        this.keyboard = builder.keyboard;
        this.display = builder.display;
    }

    public static class Builder {
        private String cpu;//必须
        private String ram;//必须
        private int usbCount;//可选
        private String keyboard;//可选
        private String display;//可选

        public Builder(String cup, String ram) {
            this.cpu = cup;
            this.ram = ram;
        }

        public Builder setUsbCount(int usbCount) {
            this.usbCount = usbCount;
            return this;
        }

        public Builder setKeyboard(String keyboard) {
            this.keyboard = keyboard;
            return this;
        }

        public Builder setDisplay(String display) {
            this.display = display;
            return this;
        }

        public Computer build() {
            return new Computer(this);
        }
    }
}
```

## 传统方式

{% asset_img builder_pattern.png builder_pattern UML %}

涉及到的角色：

* Builder: 抽象接口，定义了一系列需要实现的接口
* ConcreateBuilder: 具体的 Builder 实现类
* Production：生成的产品
* Director：具体 Builder 调用方法顺序的类

和上面的 Java 简化版相比，传统模式只不过是把类内部的 Builder 实现独立出来了而已，并没有什么其他很骚的操作。不过相比于简单的版本，它提供了 Builder 的扩展性，在这个实现里， ConcreateBuilder 可以有多个版本的实现，客户端可以根据实际需求调用所需要的 Builder。



```java
// production 类
public class Computer {
    private String cpu;//必须
    private String ram;//必须
    private int usbCount;//可选
    private String keyboard;//可选
    private String display;//可选

    public Computer(String cpu, String ram) {
        this.cpu = cpu;
        this.ram = ram;
    }

    public void setUsbCount(int usbCount) {
        this.usbCount = usbCount;
    }

    public void setKeyboard(String keyboard) {
        this.keyboard = keyboard;
    }

    public void setDisplay(String display) {
        this.display = display;
    }

    @Override
    public String toString() {
        return "Computer{" +
                "cpu='" + cpu + '\'' +
                ", ram='" + ram + '\'' +
                ", usbCount=" + usbCount +
                ", keyboard='" + keyboard + '\'' +
                ", display='" + display + '\'' +
                '}';
    }
}

// Builder 接口
public interface ComputerBuilder {
    void setUsbCount();

    void setKeyboard();

    void setDisplay();

    Computer getComputer();
}

// 具体实现类，分别组装两种品牌的电脑
public class MacBuilder implements ComputerBuilder {
    private Computer computer;

    public MacBuilder(String cpu, String ram) {
        this.computer = new Computer(cpu, ram);
    }

    @Override
    public void setUsbCount() {
        this.computer.setUsbCount(2);
    }

    @Override
    public void setKeyboard() {
        this.computer.setKeyboard("Mac Keyboard");
    }

    @Override
    public void setDisplay() {
        this.computer.setDisplay("Mac Display");
    }

    @Override
    public Computer getComputer() {
        return this.computer;
    }
}

public class LenovoBuilder implements ComputerBuilder {
    private Computer computer;

    public LenovoBuilder(String cpu, String ram) {
        this.computer = new Computer(cpu, ram);
    }

    @Override
    public void setUsbCount() {
        this.computer.setUsbCount(3);
    }

    @Override
    public void setKeyboard() {
        this.computer.setKeyboard("Logic");
    }

    @Override
    public void setDisplay() {
        this.computer.setDisplay("ThinkVision");
    }

    @Override
    public Computer getComputer() {
        return this.computer;
    }
}

// director 控制流程

public class ComputerDirector {
    public void makeComputer(ComputerBuilder builder) {
        // 定制组装顺序
        builder.setDisplay();
        builder.setKeyboard();
        builder.setUsbCount();
    }
}

// 测试类
@Test
public void test_builder() {
    ComputerDirector director = new ComputerDirector();

    MacBuilder macBuilder = new MacBuilder("I5", "Sansong 4G");
    director.makeComputer(macBuilder);
    System.out.println(macBuilder.getComputer());

    LenovoBuilder lenovoBuilder = new LenovoBuilder("I7", "Kingston 8G");
    director.makeComputer(lenovoBuilder);
    System.out.println(lenovoBuilder.getComputer());
}

// output
// Computer{cpu='I5', ram='Sansong 4G', usbCount=2, keyboard='Mac Keyboard', display='Mac Display'}
// Computer{cpu='I7', ram='Kingston 8G', usbCount=3, keyboard='Logic', display='ThinkVision'}
```

## 建造者模式在 StringBuilder 中的应用

TODO

## 参考文档

* [逼乎](https://zhuanlan.zhihu.com/p/58093669)