---
title: Java clone object
date: 2021-05-04 17:45:09
categories:
- java
tags:
- 拷贝
- clone
---

## 为什么要拷贝

因为方便，如果我有一个类有好多属性，固然可以用 new Object + set attribute 的方式达到拷贝的目的，但是麻烦。。。

## 如何拷贝

1. 实现 Cloneable 接口，否则会抛 CloneNotSupportedException 
2. 重写 clone 方法，修改修饰符为 public

```java
public class CloneTest {
    public static void main(String[] args) {
        Person person = new Person(1, "jack");
        System.out.println(person);
        Person clone = person.clone();
        System.out.println(clone);
    }
}

class Person implements Cloneable {
    int age;
    String name;

    // constructor + toString


    @Override
    public Person clone() {
        Person clone = null;
        try {
            clone = (Person)super.clone();
        } catch (CloneNotSupportedException e) {
            e.printStackTrace();
        }
        return clone;
    }
}

// Person{age=1, name='jack'}
// Person{age=1, name='jack'}
```

## 浅拷贝(shallow) Vs 深拷贝(deep)

> 浅拷贝

引用类型(接口，对象等复杂类型)的成员变量不会做拷贝，克隆对象会持有和原对象同一个引用

```java
/**
* 修改 Person 类定义，新增一个引用类型的成员变量 Address
* 为原型 person 设置 Address 变量，并设置值为 shanghai
* clone 一个 person，然后修改 Address 值为 hangzhou
* 由于是浅拷贝，持有的 Address 引用是同一个，所以两个 Person 实例的 Address 都改变了
*/
public class CloneTest {
    public static void main(String[] args) throws CloneNotSupportedException {
        Person person = new Person(1, "jack");
        Address addr = new Address("shanghai");
        person.addr = addr;

        Person clone = person.clone();
        addr.addr = "hangzhou";

        System.out.println(person);
        System.out.println(clone);
    }
}

class Person implements Cloneable {
    int age;
    String name;
    Address addr;

    // constructor + toString

    @Override
    public Person clone() {
        Person clone = null;
        try {
            clone = (Person)super.clone();
        } catch (CloneNotSupportedException e) {
            e.printStackTrace();
        }
        return clone;
    }
}

class Address {
    String addr;

    // constructor + toString 
}

```

上面这种表现方式和我们想要的有很大的出入, 我们想要的是连这些 field 也一起拷贝，深拷贝由此而来

> 深拷贝

在浅拷贝的基础上，你还需要做如下事情

1. 成员变量也需要实现 Cloneable 接口并重写 clone 方法
2. 在原来 clone 方法后面，添加成员变量的 clone 调用并赋值

```java
public class CloneTest {
    public static void main(String[] args) throws CloneNotSupportedException {
        Person person = new Person(1, "jack");
        Address addr = new Address("shanghai");
        person.addr = addr;

        Person clone = person.clone();
        addr.addr = "hangzhou";

        System.out.println(person);
        System.out.println(clone);
    }
}

class Person implements Cloneable {
    int age;
    String name;
    Address addr;

    // constructor + toString

    @Override
    public Person clone() {
        Person clone = null;
        try {
            clone = (Person)super.clone();
        } catch (CloneNotSupportedException e) {
            e.printStackTrace();
        }
        clone.addr = addr.clone(); // field 的 clone 调用和赋值
        return clone;
    }
}

class Address implements Cloneable {
    String addr;

    // constructor + toString

    @Override
    protected Address clone() {
        Address addr = null;
        try {
            addr = (Address) super.clone();
        } catch (CloneNotSupportedException e) {
            e.printStackTrace();
        }
        return addr;
    }
}

// Person{age=1, name='jack', addr=Address{addr='hangzhou'}}
// Person{age=1, name='jack', addr=Address{addr='shanghai'}}
```

## 解决多层 clone 问题

如果需要复制的对象包含很多成员变量，或者潜逃了很多层，那么重写 clone 也会变得相当麻烦，这个时候可以使用 Serializable 接口来简化序列化操作

这种实现不需要实现 Cloneable 接口，只需要相关的类实现 Serializable 接口即可，感觉就是代码看着会复杂一点，其他倒是没什么弊端

```java
public class CloneTest {
    public static void main(String[] args) throws CloneNotSupportedException {
        Person person = new Person(1, "jack");
        Address addr = new Address("shanghai");
        person.addr = addr;

        Person clone = person.clone();
        addr.addr = "hangzhou";

        System.out.println(person);
        System.out.println(clone);
    }
}

class Person implements Serializable {
    private static final long serialVersionUID = -926205369611773951L;
    int age;
    String name;
    Address addr;

    // constructor + toString

    @Override
    public Person clone() {
        Person clone = null;
        try {
            // 将该对象序列化成流,因为写在流里的是对象的一个拷贝，而原对象仍然存在于JVM里面。所以利用这个特性可以实现对象的深拷贝
            ByteArrayOutputStream baos = new ByteArrayOutputStream();
            ObjectOutputStream oos = new ObjectOutputStream(baos);
            oos.writeObject(this);
            // 将流序列化成对象
            ByteArrayInputStream bais = new ByteArrayInputStream(baos.toByteArray());
            ObjectInputStream ois = new ObjectInputStream(bais);
            clone = (Person) ois.readObject();
        } catch (IOException | ClassNotFoundException e) {
            e.printStackTrace();
        }
        return clone;
    }
}

class Address implements Serializable {
    private static final long serialVersionUID = -1360519918551295988L;
    String addr;

    // constructor + toString
}
// Person{age=1, name='jack', addr=Address{addr='hangzhou'}}
// Person{age=1, name='jack', addr=Address{addr='shanghai'}}
```
