---
title: Effective Java chapter 2 creating and destorying objects
date: 2020-10-20 10:48:50
categories:
- 编程
tags:
- java
- effective java
---

第二章 对象的生成和销毁 读书笔记

## 实体类有很多构造函数的时候，使用 Builder

> In summary, the Builder pattern is a good choice when designing classes whose constructors or static factories would have more than a handful of parameters, especially if many of the parameters are optional or of identical type. Client code is much easier to read and write with builders than with telescoping constructors, and builders are much safer than JavaBeans.
> 
> 总的来说，builder 模式适用于实体类有多个构造函数并且参数大于 5 个，参数可选并且参数类型相同的情况。

```java
/**
 * 简单的多构造函数实体类例子
 */
public class NutritionFacts {
    private final int servingSize;  // (mL)            required
    private final int servings;     // (per container) required
    private final int calories;     // (per serving)   optional
    private final int fat;          // (g/serving)     optional
    private final int sodium;       // (mg/serving)    optional
    private final int carbohydrate; // (g/serving)     optional

    public NutritionFacts(int servingSize, int servings) {
        this(servingSize, servings, 0);
    }

    public NutritionFacts(int servingSize, int servings, int calories) {
        this(servingSize, servings, calories, 0);
    }

    public NutritionFacts(int servingSize, int servings, int calories, int fat) {
        this(servingSize, servings, calories, fat, 0);
    }

    public NutritionFacts(int servingSize, int servings, int calories, int fat, int sodium) {
        this(servingSize, servings, calories, fat, sodium, 0);
    }

    public NutritionFacts(int servingSize, int servings, int calories, int fat, int sodium, int carbohydrate) {
        this.servingSize = servingSize;
        this.servings = servings;
        this.calories = calories;
        this.fat = fat;
        this.sodium = sodium;
        this.carbohydrate = carbohydrate;
    }
}
```

多构造函数能够工作，但是对客户端来说多个参数的构造函数在调用和阅读上比较容易出错。

为了解决上面的多构造器模式的弊端，还有一种解决方案是采用 简单构造函数 + setter 的方式

```java

/**
 * 简单构造函数 + Setter 的模式
 */
public class NutritionFactsV2 {
    // Parameters initialized to default values (if any)
    private int servingSize = -1; // Required; no default value
    private int servings = -1; // Required; no default value
    private int calories = 0;
    private int fat = 0;
    private int sodium = 0;
    private int carbohydrate = 0;

    public NutritionFactsV2() {
    }

    // Setter
    public void setServingSize(int val) {
        servingSize = val;
    }

    public void setServings(int val) {
        servings = val;
    }

    public void setCalories(int val) {
        calories = val;
    }

    public void setFat(int val) {
        fat = val;
    }

    public void setSodium(int val) {
        sodium = val;
    }

    public void setCarbohydrate(int val) {
        carbohydrate = val;
    }
}
```

弊端：调用是分散的，类属性可能存在不一致；这种模式不可能把一类做成不可变，需要额外的努力来确保线程安全。

以下是建造者模式的实现

```java
public class NutritionFacts {
    private final int servingSize;
    private final int servings;
    private final int calories;
    private final int fat;
    private final int sodium;
    private final int carbohydrate;

    public static class Builder {
        // Required parameters
        private final int servingSize;
        private final int servings;

        // Optional parameters - initialized to default values
        private int calories = 0;
        private int fat = 0;
        private int sodium = 0;
        private int carbohydrate = 0;

        public Builder(int servingSize, int servings) {
            this.servingSize = servingSize;
            this.servings = servings;
        }

        public Builder calories(int val) {
            calories = val;
            return this;
        }

        public Builder fat(int val) {
            fat = val;
            return this;
        }

        public Builder sodium(int val) {
            sodium = val;
            return this;
        }

        public Builder carbohydrate(int val) {
            carbohydrate = val;
            return this;
        }

        public NutritionFacts build() {
            return new NutritionFacts(this);
        }
    }

    private NutritionFacts(Builder builder) {
        servingSize = builder.servingSize;
        servings = builder.servings;
        calories = builder.calories;
        fat = builder.fat;
        sodium = builder.sodium;
        carbohydrate = builder.carbohydrate;
    }
}
```

客户端调用代码如下

```java
NutritionFacts cocaCola = new NutritionFacts.Builder(240, 8).calories(100).sodium(35).carbohydrate(27).build();
```

多态下使用 Builder 模式

```java
// 父类
public abstract class Pizza {
    public enum Topping {HAM, MUSHROOM, ONION, PEPPER, SAUSAGE}

    final Set<Topping> toppings;

    abstract static class Builder<T extends Builder<T>> {
        private EnumSet<Topping> toppings = EnumSet.noneOf(Topping.class);

        public T addTopping(Topping topping) {
            toppings.add(Objects.requireNonNull(topping));
            return self();
        }

        abstract Pizza build();

        // Subclasses must override this method to return "this"
        protected abstract T self();
    }

    Pizza(Builder<?> builder) {
        toppings = builder.toppings.clone(); // See Item 50}
    }
}

// 子类其一
public class NYPizza extends Pizza {
    public enum Size {SMALL, MEDIUM, LARGE}

    private final Size size;

    private NYPizza(Builder builder) {
        super(builder);
        this.size = builder.size;
    }

    @Override
    public String toString() {
        return "NYPizza{" +
                "size=" + size +
                ", toppings=" + toppings +
                '}';
    }

    public static class Builder extends Pizza.Builder<Builder> {
        private final Size size;

        public Builder(Size size) {
            this.size = Objects.requireNonNull(size);
        }

        @Override
        NYPizza build() {
            return new NYPizza(this);
        }

        @Override
        protected Builder self() {
            return this;
        }
    }
}

// 子类其二
public class Calzone extends Pizza {
    private final boolean sauceInside;

    private Calzone(Builder builder) {
        super(builder);
        this.sauceInside = builder.sauceInside;
    }

    @Override
    public String toString() {
        return "Calzone{" +
                "sauceInside=" + sauceInside +
                ", toppings=" + toppings +
                '}';
    }

    public static class Builder extends Pizza.Builder<Builder> {
        private boolean sauceInside = false; // Default

        public Builder sauceInside() {
            sauceInside = true;
            return this;
        }

        @Override
        Calzone build() {
            return new Calzone(this);
        }

        @Override
        protected Builder self() {
            return this;
        }
    }
}

// 调用
public class Client {

    public static void main(String[] args) {
        NYPizza nyPizza = new NYPizza.Builder(NYPizza.Size.LARGE).addTopping(Pizza.Topping.ONION).addTopping(Pizza.Topping.HAM).build();
        System.out.println(nyPizza);

        Calzone calzone = new Calzone.Builder().sauceInside().addTopping(Pizza.Topping.HAM).addTopping(Pizza.Topping.SAUSAGE).build();
        System.out.println(calzone);
    }
}
```

Builder 模式出了比较冗长之外没有其他坏处，而且扩展性好。

## 问题

* 使用 final 修饰属性有什么好处?
  * 可以保证对应的变量只被赋值一次，并且 final 修饰的变量必须得赋值

PS: 外部类可以访问内部类的私有变量，这个我之前倒是没有想到的

* `Builder<T extends Builder<T>>` 语法
  * 这种语法叫做 递归类型参数(recursive type parameter), 是泛型的一种，指代泛型参数必须是自己的子类，不理解可以先记着

* `protected abstract T self();` 为什么不直接返回 `this`?
  * 亲自写一下就会发现，builder 本身是个抽象类，所以是没有 `this` 这个指代的

* 父类的构造器是 `protected` 的，不然子类无法继承, 同理 `toppings` 也应该是 protected 的，不然子类根本就访问不了 (´Д` )


  