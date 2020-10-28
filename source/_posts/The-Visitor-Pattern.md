---
title: 访问者模式
date: 2020-09-09 15:37:59
categories:
- 编程
tags:
- java
- 设计模式
---

> GoF 定义: Allows for one or more operation to be applied to a set of objects at runtime, decoupling the operations from the object structure. 

访问者模式讲的是表示一个作用于某对象结构中的各元素的操作。它使你可以在不改变各元素的类的前提下定义作用于这些元素的新操作。

行为模式之一，目的是将**行为**和**对象**分开。

缺点：**每增**加一种支持的 object，你就必须在 visitor 及其实现类中添加新的方法支持这个改动。

## 描述

被访问者就是上文中的 object，他持有数据，我们想把他和数据运算分离，保持其独立性

访问者代表着 operations，通过它可以实现数据运算

## UML

TODO

## 实例

### From DZone

* [DZone - Visitor Pattern](https://dzone.com/articles/design-patterns-visitor)

抽象一个邮寄业务，计算购物车中所有的物件总的邮费。每样物件都有自己的属性，比如价格，重量之类的。我们将邮费计算的规则单独封装在 Visitor 中，在物件类中通过调用 accept 实现计算。

```java
// 代表 object 的接口
public interface Visitable {
    void accept(Visitor visitor);
}

// 实现 accept 的实例
public class Book implements Visitable {
    private double price = 8.0;
    private double weight = 3.2;

    @Override
    public void accept(Visitor visitor) {
        visitor.visit(this);
    }

    public double getPrice() {
        return price;
    }

    public double getWeight() {
        return weight;
    }
}

// visitor 接口
public interface Visitor {
    void visit(Book book);

    void visit(Shoes shoes);
}

// visitor 实例
public class PostageVisitor implements Visitor {
    private double totalPostageForCart;

    @Override
    public void visit(Book book) {
        // rule to calculate book postage cost
        // if price over 10, free postage.
        if(book.getPrice() < 10.0) {
            totalPostageForCart += book.getWeight() * 2;
        }
    }

    @Override
    public void visit(Shoes shoes) { //TODO }

    public double getTotalPostageForCart() {
        return this.totalPostageForCart;
    }
}

// 客户端调用
public class Client {
    public static void main(String[] args) {
        Book book = new Book();
        Shoes shoes = new Shoes();
        PostageVisitor postageVisitor = new PostageVisitor();

        book.accept(postageVisitor);
        shoes.accept(postageVisitor);

        System.out.println("Total cost: " + postageVisitor.getTotalPostageForCart());
    }
}
```

### From Refactoring Guru

* [重构大师](https://refactoringguru.cn/design-patterns/visitor/java/example)

根据定义的图形打印信息到 XML 文件中，这个例子本质上和前一个没什么区别，但是他提供了组合类型的 object 支持，并且输出 xml, 还有 format 都让我眼前一亮。反正感觉很赞！

```java
// 定义持有 accept 的接口
public interface Shape {
    void move(int x, int y);

    void draw();

    String accept(Visitor visitor);
}

// 定义接口实现
public class Dot implements Shape {
    private int id;
    private int x;
    private int y;

    public Dot() {
    }

    public Dot(int id, int x, int y) {
        this.id = id;
        this.x = x;
        this.y = y;
    }

    @Override
    public void move(int x, int y) {
        // move shape
    }

    @Override
    public void draw() {
        // draw shape
    }

    public String accept(Visitor visitor) {
        return visitor.visitDot(this);
    }
    // getter + setter
}

// 定义组合类型的实现
public class CompoundShape implements Shape {
    public int id;
    public List<Shape> children = new ArrayList<>();

    public CompoundShape(int id) {
        this.id = id;
    }

    @Override
    public void move(int x, int y) {
        // move shape
    }

    @Override
    public void draw() {
        // draw shape
    }

    public int getId() {
        return id;
    }

    @Override
    public String accept(Visitor visitor) {
        return visitor.visitCompoundGraphic(this);
    }

    public void add(Shape shape) {
        children.add(shape);
    }
}

// 定义 visitor 接口
public interface Visitor {
    String visitDot(Dot dot);

    String visitCircle(Circle circle);

    String visitRectangle(Rectangle rectangle);

    String visitCompoundGraphic(CompoundShape cg);
}

// visitor 实现
public class XMLExportVisitor implements Visitor {

    public String export(Shape... args) {
        StringBuilder sb = new StringBuilder();
        sb.append("<?xml version=\"1.0\" encoding=\"utf-8\"?>" + "\n");
        for (Shape shape : args) {
            sb.append(shape.accept(this)).append("\n");
        }
        return sb.toString();
    }

    public String visitDot(Dot d) {
        return "<dot>" + "\n" +
                "    <id>" + d.getId() + "</id>" + "\n" +
                "    <x>" + d.getX() + "</x>" + "\n" +
                "    <y>" + d.getY() + "</y>" + "\n" +
                "</dot>";
    }

...

    public String visitCompoundGraphic(CompoundShape cg) {
        return "<compound_graphic>" + "\n" +
                "   <id>" + cg.getId() + "</id>" + "\n" +
                _visitCompoundGraphic(cg) +
                "</compound_graphic>";
    }

    private String _visitCompoundGraphic(CompoundShape cg) {
        StringBuilder sb = new StringBuilder();
        for (Shape shape : cg.children) {
            String obj = shape.accept(this);
            // Proper indentation for sub-objects.
            obj = "    " + obj.replace("\n", "\n    ") + "\n";
            sb.append(obj);
        }
        return sb.toString();
    }

}

// 客户端调用
public class Client {
    public static void main(String[] args) {
        Dot dot = new Dot(1, 10, 55);
        Circle circle = new Circle(2, 23, 15, 10);
        Rectangle rectangle = new Rectangle(3, 10, 17, 20, 30);

        CompoundShape compoundShape = new CompoundShape(4);
        compoundShape.add(dot);
        compoundShape.add(circle);
        compoundShape.add(rectangle);

        CompoundShape c = new CompoundShape(5);
        c.add(dot);
        compoundShape.add(c);

        export(circle, compoundShape);
    }

    private static void export(Shape... shapes) {
        XMLExportVisitor exportVisitor = new XMLExportVisitor();
        System.out.println(exportVisitor.export(shapes));
    }
}
```