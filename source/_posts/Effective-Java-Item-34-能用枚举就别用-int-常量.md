---
title: Effective Java Item 34 能用枚举就别用 int 常量
date: 2020-06-05 23:14:50
categories:
- 编程
tags:
- java
- effective java
- 枚举和注解
---
本节要点：

* 使用 enum 代替 整型/字符型枚举模式
* enum 是 final，单例的安全
* 在 enum 内部使用 abstract 方法使得实例和方法绑定
* 用 values() 遍历，用 valueOf() 反向索取
* 使用策略枚举来封装算法

在枚举类加入到 java 大家族之前，为了表达达到枚举的效果，我们使用整形常量来表示，这种表达方式被叫做： int 枚举模式(int enum pattern), 例如：

```java
public static final int APPLE_FUJI = 0;
public static final int APPLE_PIPPIN = 1;
public static final int APPLE_GRANNY_SMITH = 2;

public static final int ORANGE_NAVEL = 0;
public static final int ORANGE_TEMPLE = 1;
public static final int ORANGE_BLOOD = 2;
```

缺点： 类型不安全 + 描述性不好， 与之类似的还有 String 枚举模式(String enum pattern)。就是用 String 来代替上例中的 int, 这种做法更糟糕，就算拼写错误也能编译通过，很容易引入 bug。

枚举中每个实例都是单例的，是 public static final 的 field。 枚举没有可访问的构造器，所以不能被继承，是真正的 final 类型的 class。enum 提供了一个命名空间，所以不同 enum 中重名是允许的。示例：

```java
public enum Apple { FUJI, PIPPIN, GRANNY_SMITH }
public enum Orange { NAVEL, TEMPLE, BLOOD }
```

太阳系八大行星枚举示例：

```java
public enum Planet {
    MERCURY(3.302e+23, 2.439e6),
    VENUS(4.869e+24, 6.052e6),
    EARTH(5.975e+24, 6.378e6), 
    MARS(6.419e+23, 3.393e6),
    JUPITER(1.899e+27, 7.149e7), 
    SATURN(5.685e+26, 6.027e7),
    URANUS(8.683e+25, 2.556e7), 
    NEPTUNE(1.024e+26, 2.477e7);
    private final double mass;           // In kilograms
    private final double radius;         // In meters
    private final double surfaceGravity; // In m / s^2
    // Universal gravitational constant in m^3 / kg s^2
    private static final double G = 6.67300E-11;

    // Constructor
    Planet(double mass, double radius) {
        this.mass = mass;
        this.radius = radius;
        surfaceGravity = G * mass / (radius * radius);
    }

    public double mass() {
        return mass;
    }

    public double radius() {
        return radius;
    }

    public double surfaceGravity() {
        return surfaceGravity;
    }

    public double surfaceWeight(double mass) {
        return mass * surfaceGravity;  // F = ma}
    }
}
```

枚举中所有的 field 都应该是 final 的。枚举都有 values() 静态方法， 按照声明顺序返回枚举值。

根据枚举类的适用范围制定他的访问权限，如果是普适的，就把他定义成顶层类，比如 math 中控制舍入模式的 RoundingMode 类。

枚举绑定行为的最佳实践：

```java
// 普通表示
public enum Operation {
    PLUS, MINUS, TIMES, DIVIDE;

    public double apply(double x, double y) {
        switch (this) {
            case PLUS:
                return x + y;
            case MINUS:
                return x - y;
            case TIMES:
                return x * y;
            case DIVIDE:
                return x / y;
        }
        throw new AssertionError("Unknown op: " + this);
    }
}
```

缺点：

* 没有 throw exception 会编译失败
* 代码脆弱，在添加新操作，如果没有添加 switch 分支的话，新操作不能生效

改进版：

```java
public enum Operation {
    PLUS("+") {
        @Override
        public double apply(double x, double y) {
            return x + y;
        }
    }, MINUS("-") {
        @Override
        public double apply(double x, double y) {
            return x - y;
        }
    }, TIMES("*") {
        @Override
        public double apply(double x, double y) {
            return x * y;
        }
    }, DIVIDE("/") {
        @Override
        public double apply(double x, double y) {
            return x / y;
        }
    };

    private final String symbol;

    Operation(String symbol) {
        this.symbol = symbol;
    }

    @Override
    public String toString() {
        return symbol;
    }

    public abstract double apply(double x, double y);
}
```

通过将 apply 方法声明为 abstrct 类型迫使枚举类的每个 field 都必须实现自己的 apply 方法达到绑定的效果，这种做法称为：constant-specific method implementation。

通过使用 values() 方法，可以很方便的实现迭代

```java
double x = 2.0;
double y = 4.0;
for (Operation op : Operation.values()) {
    System.out.printf("%f %s %f = %f%n", x, op , y, op.apply(x, y));
}
```

如果 enum 的 toString 方法被重写了，可以订制 fromString() 方法实现字符到枚举的转化

```java
// 将枚举的名称和枚举类型配对，存到 map 中
private static final Map<String, Operation> stringToEnum = Stream.of(Operation.values()).collect(Collectors.toMap(Object::toString, e-> e));
// 新增 fromString 方法根据 toString 的值到 map 中取数据
public static Optional<Operation> fromString(String symbol) {
    return Optional.ofNullable(stringToEnum.get(symbol));
}

System.out.println(fromString("a"));
System.out.println(fromString("-"));
// output:
// Optional.empty
// Optional[-]
```

通过 switch 来控制 enum 中的条件选择的例子, 该例用于计算薪资，根据工作日和休息日采取不同的薪资计算。在这个例子中周末工资的理解很有意思，它等于**基本工资 + 从一开始就累加的加班工资**，这样想的话这个例子理解起来会容易一点。

```java
enum PayrollDay {
    MONDAY, TUESDAY, WEDNESDAY, THURSDAY, FRIDAY, SATURDAY, SUNDAY;

    private static final int MINS_PER_SHIFT = 8 * 60;

    int pay(int minutesWorked, int payRate) {
        int basePay = minutesWorked * payRate;

        int overtimePay;
        switch (this) {
            // weekends
            case SATURDAY:
            case SUNDAY:
                overtimePay = basePay / 2;
                break;
            // work day
            default:
                overtimePay = minutesWorked <= MINS_PER_SHIFT ? 0 : (minutesWorked - MINS_PER_SHIFT) * payRate / 2;
        }
        return basePay + overtimePay;
    }
}
```

在 enum 中使用 switch 有一个弊端， 新添加的类型，比如我想加一个国亲节加班的薪资计算，如果忘了在 switch 中添加相应的分支， 虽然编译能过，然是薪资计算的规则已经出错了。我们通过在该 enum 中添加一个策略枚举来改善它

```java
enum PayrollDay {
    MONDAY, TUESDAY, WEDNESDAY, THURSDAY, FRIDAY,
    SATURDAY(PayType.WEEKEND), SUNDAY(PayType.WEEKEND);

    private final PayType payType;

    private static final int MINS_PER_SHIFT = 8 * 60;

    PayrollDay(PayType payType) {
        this.payType = payType;
    } // constructor for weekend

    PayrollDay() {
        this(PayType.WEEKDAY);
    } // constructor for weekday

    int pay(int minutesWorked, int payRate) {
        return payType.pay(minutesWorked, payRate);
    }

    private enum PayType {
        WEEKDAY {
            @Override
            int overtimePay(int minsWorked, int payRate) {
                return minsWorked <= MINS_PER_SHIFT ? 0 : (minsWorked - MINS_PER_SHIFT) * payRate / 2;
            }
        },
        WEEKEND {
            @Override
            int overtimePay(int minsWorked, int payRate) {
                return minsWorked * payRate / 2;
            }
        };

        abstract int overtimePay(int minsWorked, int payRate);

        private static final int MINS_PER_SHIFT = 8 * 60;

        int pay(int minsWorked, int payRate) {
            int basePay = minsWorked * payRate;
            return basePay + overtimePay(minsWorked, payRate);
        }
    }
}
```

所以总结起来就是在枚举内部别用 switch， 在外部鼓励使用。枚举在性能上与 int 相当，但是由于包装成对象形肯定要略差的，但是使用上感觉不出来。所以**每当需要一组固定常量，并且在编译时就知道其成员的时候，就应该使用枚举**

多个枚举共享行为是可以用**策略枚举**的形式

枚举中的常量集并不一定要始终保持不变(?不是很清楚怎么理解，没碰到过这种情况)