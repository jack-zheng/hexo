---
title: TIJ4 - Type Information 笔记
date: 2020-12-22 22:18:06
categories:
- 编程
tags:
- TIJ4
---

- [前述](#前述)
- [The need for RTTI](#the-need-for-rtti)
- [The Class object](#the-class-object)
  - [Class literals(字面量)](#class-literals字面量)
  - [Generic class references](#generic-class-references)
  - [New cast syntax](#new-cast-syntax)
- [Checking before a cast](#checking-before-a-cast)
  - [Using class literals](#using-class-literals)
  - [A dynamic instanceof](#a-dynamic-instanceof)
  - [Counting recursively](#counting-recursively)
- [Registered factories](#registered-factories)
- [instanceof vs. Class equivalence](#instanceof-vs-class-equivalence)

## 前述

Runtime type information(RTTI) allows you to discover and use type information while a program is running.

两种使用方式：

1. 传统模式，假定你在编译期就知道所有用到的类型
2. 反射模式，你只在运行时才知道类信息

## The need for RTTI

简单的继承关系示例：

基类：Shape 包含方法 draw()， 子类：Circle, Square, Triangle

```java
import java.util.Arrays;
import java.util.List;

abstract class Shape {
    void draw() {
        System.out.println(this + ".draw()");
    }

    abstract public String toString();
}

class Circle extends Shape {
    public String toString() {
        return "Circle";
    }
}

class Square extends Shape {
    public String toString() {
        return "Square";
    }
}

class Triangle extends Shape {
    public String toString() {
        return "Triangle";
    }
}

public class Shapes {
    public static void main(String[] args) {
        List<Shape> shapeList = Arrays.asList(new Circle(), new Square(), new Triangle());
        for (Shape shape : shapeList) shape.draw();
    }
}
// output:
// Circle.draw()
// Square.draw()
// Triangle.draw()
```

在上面的例子中，我本将子类结合 List 强转成父类，然后统一做操作，这种做法更易读，容易维护。这也是面向对象的目标之一，但是如果我想在运行时得知这个对象的具体类型，应该怎么做？

## The Class object

在 Java 中有一个神奇的类他叫 Class 类，所有创建类实例的行为都和他有关。 Java 的 RTTI 特性也是通过它来实现的。当你编译一个类的时候，JVM 会通过 class 创建一个对应的 Class 类来存储对应的信息。

类加载器可以有一组 class loaders 组成，但是已有一个 primordial class loader，他时 JVM 的一部分，他也被叫做 trusted classes。通常你不需要自己新家 class loader 但是如果由特殊需要，想加也是可以的。

只有当第一次使用的时候，JVM 才会加载对应的 class。这个行为发生在类第一次关联到 static 实体， 构造函数也是一个特殊的 static method，换句话说，当我们 new 一个对象的时候，加载器就会加载对应的 class。

Java 中只允许 Class 加载一次，加载完成之后，以后所有这个 class 对应的实体都是通过它来创建的。

PS：这里翻译很生硬，缺少很多类加载的相关知识，可以看过 JVM 那本书之后，再来完善一下。

```java
class Candy {
    static {
        System.out.println("Loading Candy");
    }
}

class Gum {
    static {
        System.out.println("Loading Gum");
    }
}

class Cookie {
    static {
        System.out.println("Loading Cookie");
    }
}

public class SweetShop {
    public static void main(String[] args) {
        System.out.println("inside main");
        new Candy();
        System.out.println("After creating Candy");
        try {
            Class.forName("Gum");
        } catch (ClassNotFoundException e) {
            System.out.println("Couldn’t find Gum");
        }
        System.out.println("After Class.forName(\"Gum\")");
        new Cookie();
        System.out.println("After creating Cookie");
    }
}
// output:
// inside main
// Loading Candy
// After creating Candy
// Couldn’t find Gum
// After Class.forName("Gum")
// Loading Cookie
// After creating Cookie
```

当各个类在第一次调用时对应的静态代码块就会被调用，输出我们定制的信息。上例有一个比较特殊的语法 `forName()` 我们可以通过这个方法拿到对应的 Class 引用，当然如果找不到会抛 `ClassNotFoundExcepiton`。如果实体类已经创建了，你也可以通过 Object.getClass() 来拿到对应的类应用。

下面这个示例展示了部分 Class 中的常用方法：

```java
package samples;

interface HasBatteries {
}

interface Waterproof {
}

interface Shoots {
}

class Toy {
    // Comment out the following default constructor
    // to see NoSuchMethodError from (*1*)
    Toy() {
    }

    Toy(int i) {
    }
}

class FancyToy extends Toy
        implements HasBatteries, Waterproof, Shoots {
    FancyToy() {
        super(1);
    }
}

public class ToyTest {
    static void printlnInfo(Class cc) {
        System.out.println("Class name: " + cc.getName() +
                " is interface? [" + cc.isInterface() + "]");
        System.out.println("Simple name: " + cc.getSimpleName());
        System.out.println("Canonical name : " + cc.getCanonicalName());
    }

    public static void main(String[] args) {
        Class c = null;
        try {
            c = Class.forName("samples.FancyToy");
        } catch (ClassNotFoundException e) {
            System.out.println("Can’t find FancyToy");
            System.exit(1);
        }
        printlnInfo(c);
        for (Class face : c.getInterfaces())
            printlnInfo(face);
        Class up = c.getSuperclass();
        Object obj = null;
        try {
            // Requires default constructor:
            obj = up.newInstance();
        } catch (InstantiationException e) {
            System.out.println("Cannot instantiate");
            System.exit(1);
        } catch (IllegalAccessException e) {
            System.out.println("Cannot access");
            System.exit(1);
        }
        printlnInfo(obj.getClass());
    }
}
// output:
// Class name: samples.FancyToy is interface? [false]
// Simple name: FancyToy
// Canonical name : samples.FancyToy
// Class name: samples.HasBatteries is interface? [true]
// Simple name: HasBatteries
// Canonical name : samples.HasBatteries
// Class name: samples.Waterproof is interface? [true]
// Simple name: Waterproof
// Canonical name : samples.Waterproof
// Class name: samples.Shoots is interface? [true]
// Simple name: Shoots
// Canonical name : samples.Shoots
// Class name: samples.Toy is interface? [false]
// Simple name: Toy
// Canonical name : samples.Toy
```

* getSimpleName(): 输出类名
* getCanonicalName(): 输出全路径名
* islnterface(): 是否是接口
* getlnterfaces(): 拿到类实现的接口
* getSuperclass(): 拿到父类 Class 引用
* newlnstance(): 创建实例，但是这个方法要求对应的类必须有**默认构造函数**

### Class literals(字面量)

出了上面的方法，你还可以通过使用类的字面量来拿到 Class 引用，相比与 forName() 的形式，它更简单，安全，不需要 try-catch 块，效率也更高。

普通类，接口，数组和基本数据类型都可以使用这个语法，对于包装类，它内部有一个 TYPE field 可以指向对应的 Class。

| primitive     | wrapper      |
| :------------ | :----------- |
| boolean.class | Boolean.TYPE |
| char.class    | Char.TYPE    |
| byte.class    | Byte.TYPE    |
| short.class   | Short.TYPE   |
| int.class     | Integer.TYPE |
| long.class    | Long.TYPE    |
| float.class   | Float.TYPE   |
| double.class  | Double.TYPE  |
| void.class    | Void.TYPE    |

通常建议使用 '.class' 的这种语法，它和我们平时的使用方式更统一。调用 '.class' 的时候并不会自动初始化一个 Class 对象。在初始化 Class 时有三个步骤：

1. Loading, which is performed by the class loader. 找到生成 Class 对应的字节码
2. Linking. 验证字节码，为静态变量分配空间，解决依赖问题
3. Initialization. 如果还有父类，父类会先初始化，然后执行静态构造器和代码块

初始化会延期，直到确定第一个到静态方法(构造函数是一个隐式的静态方法)或非常量的静态 field 引用：

```java
package review;

import java.util.Random;

class Initable {
    static final int staticFinal = 47;
    static final int staticFinal2 =
            ClassInitialization.rand.nextInt(1000);

    static {
        System.out.println("Initializing Initable");
    }
}

class Initable2 {
    static int staticNonFinal = 147;

    static {
        System.out.println("Initializing Initable2");
    }
}

class Initable3 {
    static int staticNonFinal = 74;

    static {
        System.out.println("Initializing Initable3");
    }
}

public class ClassInitialization {
    public static Random rand = new Random(47);

    public static void main(String[] args) throws Exception {
        Class initable = Initable.class;
        System.out.println("After creating Initable ref");
        // Does not trigger initialization:
        System.out.println(Initable.staticFinal);
        // Does trigger initialization:
        System.out.println(Initable.staticFinal2);
        // Does trigger initialization:
        System.out.println(Initable2.staticNonFinal);
        Class initable3 = Class.forName("review.Initable3");
        System.out.println("After creating Initable3 ref");
        System.out.println(Initable3.staticNonFinal);
    }
}
// output:
// After creating Initable ref
// 47
// Initializing Initable
// 258
// Initializing Initable2
// 147
// Initializing Initable3
// After creating Initable3 ref
// 74
```

实际上，初始化尽可能完的执行。从 initable 的例子可以看出，调用 '.class' 语法并不会导致一个 Class 的初始化，但是从 initable3 可以看出 Class.forName() 会直接导致初始化。

如果调用的是 static final 这种编译期常量，如 Initable.staticFinal 所示，那么该值也可以在类为初始化时就可用。就算有 static 和 final 也不能保证不需要初始化 Class，如 Initable.staticFinal2 所示，如果对应的变量不是编译期常量，还是想要初始化 Class 的。

如果变量不是 final 类型的，那么在访问之前就需要进行 link 和 initialization 动作，如 Initable2.staticNonFinal 所示。

### Generic class references

具体的 Class 对象包含了静态变量，方法等创建一个对象所需要的所有必要元素。但是在 Java 5 之前这种 reference 都是 object 类型的，但是 Java 5 之后，通过引入泛型，我们可以用一种更特殊的方式指代它。

实例说明：

同样是持有 int 的 class reference，如果没有采用泛型，reference 之间可以随便关联，如果带有泛型则会进行类型检测。

```java
public class GenericClassReferences {
    public static void main(String[] args) {
        Class intClass = int.class;
        Class<Integer> genericIntClass = int.class;
        genericIntClass = Integer.class; // Same thing
        intClass = double.class;
        // genericIntClass = double.class; // Illegal
    }
}
```

如果你想要更宽松的类型检测，可以使用类似 `Class<? extends Number> genericNumberClass = int.class;` 的语法。

在 Java 5 中 Class<?> 效果上和 Class 等价，但是前者没有 warning, 因语义上他更清晰的表明，这个 Class 不是一个 non-specific 的类。

Class 声明中加入泛型语法的唯一作用就是在编译期进行类型检测。

实例说明：

简单的泛型使用案例，就是没什么逻辑，只是使用，看起来感觉没什么目的，比较难记忆。CountedInteger 的 counter 属性是一个静态变量，充当实例的 id 的角色。

```java
class CountedInteger {
    private static long counter;
    private final long id = counter++;

    public String toString() {
        return Long.toString(id);
    }
}

public class FilledList<T> {

    private Class<T> type;

    public FilledList(Class<T> type) {
        this.type = type;
    }

    public List<T> create(int nElements) {
        List<T> result = new ArrayList<T>();
        try {
            for (int i = 0; i < nElements; i++)
                result.add(type.newInstance());
        } catch (Exception e) {
            throw new RuntimeException(e);
        }
        return result;
    }

    public static void main(String[] args) {
        FilledList<CountedInteger> fl =
                new FilledList<CountedInteger>(CountedInteger.class);
        System.out.println(fl.create(15));
    }
}
// output:
// [0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14]
```

注意点：

1. CountedInteger 必须有默认的无参构造函数，不然嗲用 nweInstance 会抛错
2. 如果带有泛型，newInstance 直接返回对应的对象，而不是 Object 对象

对应下面的例子，up 在指定类型的时候用的是 `<? super FancyToy>` (FancyToy 的超类)，并不能表示为直接父类，所以下面的 newInstanc() 对应的类型为 Object。

```java
public class GenericToyTest {
    public static void main(String[] args) throws Exception {
        Class<FancyToy> ftClass = FancyToy.class;
        // Produces exact type:
        FancyToy fancyToy = ftClass.newInstance();
        Class<? super FancyToy> up = ftClass.getSuperclass();
        // This won’t compile:
        // Class<Toy> up2 = ftClass.getSuperclass();
        // Only produces Object:
        Object obj = up.newInstance();
    }
} 
```

### New cast syntax

Java 5 中还为 Class 添加了 cast(object) 方法用于将参数强转为 Class 对应的类实例。

```java
class Building {}

class House extends Building {}

public class ClassCasts {
    public static void main(String[] args) {
        Building b = new House();
        Class<House> houseType = House.class;
        House h = houseType.cast(b);
        h = (House) b; // ... or just do this.
    }
}
```

使用括号的那种强转方式要比调用方法的方便很多，需要注意的是，一开始 new 的时候对应的实现是 House 才能在后面进行 b 强转成 a 的，如果你一开始声明为 new Building() 而且两边的方法有出入，运行时会抛异常

```java
class Building {}

class House extends Building {
    public void method01() {
        System.out.println("m1...");
    }
}

public class ClassCasts {
    public static void main(String[] args) {
        Building b = new Building();
        ((House)b).method01();
    }
}

// output:
// Exception in thread "main" java.lang.ClassCastException
// at review.ClassCasts.main(ClassCasts.java:15)
```

## Checking before a cast

父类强转到子类的过程叫做 downcast, java 中如果没有显示的 check 的话，这种强转是不允许的。这里就要提到 RTTI 的第三种模式 instanceof 语法。`if (x instanceof Dog) ((Dog)x).bark();`

实例说明：

1. Individual 具体 code 在 Containers in Depth 那一个章节，只需要知道它里面有一个 id() 方法可以给每个继承这个累的对象一个唯一的值做 id, 构造函数的参数则是自定义的 name, 可以重复的
2. 定义了一大串有继承关系的类，继承关系如下
3. 定义一个抽象构造器 PetCreator 创建声明的这些类
4. 实现这个抽象构造器 ForNameCreator，其实就是将声明的类通过 Class.forName 拿到引用再塞到 type() 方法的返回值中
5. 便携测试程序 PetCount, 创建一个内部类 PetCounter 统计 pet 的出现次数，有一个静态方法 countPets 传入 PetCreater 用来随机创建 Pet 对象

individual -> person
        |-> pet -> dog - mutt(串串)
            ｜      ｜-> pug(哈巴狗)
            ｜-> cat -> EgyptianMau (埃及猫)
            ｜    ｜-> Manx(曼岛猫) -> Cymric(威尔士猫)
            ｜-> Rodent 啮齿动物 -> Rat 鼠 -> Mouse 小鼠
                    |-> Hamster 仓鼠

```java
class Individual implements Comparable<Individual> {
    private static long counter = 0;
    private final long id = counter++;
    private String name;

    public Individual(String name) {
        this.name = name;
    }

    // ‘name’ is optional:
    public Individual() {
    }

    public String toString() {
        return getClass().getSimpleName() +
                (name == null ? "" : " " + name);
    }

    public long id() {
        return id;
    }

    public boolean equals(Object o) {
        return o instanceof Individual &&
                id == ((Individual) o).id;
    }

    public int hashCode() {
        int result = 17;
        if (name != null)
            result = 37 * result + name.hashCode();
        result = 37 * result + (int) id;
        return result;
    }

    public int compareTo(Individual arg) {
        // Compare by class name first:
        String first = getClass().getSimpleName();
        String argFirst = arg.getClass().getSimpleName();
        int firstCompare = first.compareTo(argFirst);
        if (firstCompare != 0)
            return firstCompare;
        if (name != null && arg.name != null) {
            int secondCompare = name.compareTo(arg.name);
            if (secondCompare != 0)
                return secondCompare;
        }
        return (arg.id < id ? -1 : (arg.id == id ? 0 : 1));
    }
}

class Person extends Individual {
    public Person(String name) {
        super(name);
    }
}

class Pet extends Individual {
    public Pet(String name) {
        super(name);
    }

    public Pet() {
        super();
    }
}

class Dog extends Pet {
    public Dog(String name) {
        super(name);
    }

    public Dog() {
        super();
    }
}

class Mutt extends Dog {
    public Mutt(String name) {
        super(name);
    }

    public Mutt() {
        super();
    }
}

class Pug extends Dog {
    public Pug(String name) {
        super(name);
    }

    public Pug() {
        super();
    }
}

class Cat extends Pet {
    public Cat(String name) {
        super(name);
    }

    public Cat() {
        super();
    }
}

class EgyptianMau extends Cat {
    public EgyptianMau(String name) {
        super(name);
    }

    public EgyptianMau() {
        super();
    }
}

class Manx extends Cat {
    public Manx(String name) {
        super(name);
    }

    public Manx() {
        super();
    }
}

class Cymric extends Manx {
    public Cymric(String name) {
        super(name);
    }

    public Cymric() {
        super();
    }
}

class Rodent extends Pet {
    public Rodent(String name) {
        super(name);
    }

    public Rodent() {
        super();
    }
}

class Rat extends Rodent {
    public Rat(String name) {
        super(name);
    }

    public Rat() {
        super();
    }
}

class Mouse extends Rodent {
    public Mouse(String name) {
        super(name);
    }

    public Mouse() {
        super();
    }
}

class Hamster extends Rodent {
    public Hamster(String name) {
        super(name);
    }

    public Hamster() {
        super();
    }
}

abstract class PetCreator {
    private Random rand = new Random(47);

    // 模版方法 模式
    // The List of the different types of Pet to create:
    public abstract List<Class<? extends Pet>> types();

    public Pet randomPet() { // Create one random Pet
        int n = rand.nextInt(types().size());
        try {
            return types().get(n).newInstance();
        } catch (InstantiationException e) {
            throw new RuntimeException(e);
        } catch (IllegalAccessException e) {
            throw new RuntimeException(e);
        }
    }

    public Pet[] createArray(int size) {
        Pet[] result = new Pet[size];
        for (int i = 0; i < size; i++)
            result[i] = randomPet();
        return result;
    }

    public ArrayList<Pet> arrayList(int size) {
        ArrayList<Pet> result = new ArrayList<Pet>();
        Collections.addAll(result, createArray(size));
        return result;
    }
}

class ForNameCreator extends PetCreator {
    private static List<Class<? extends Pet>> types =
            new ArrayList<Class<? extends Pet>>();
    // Types that you want to be randomly created:
    private static String[] typeNames = {
            "review.Mutt",
            "review.Pug",
            "review.EgyptianMau",
            "review.Manx",
            "review.Cymric",
            "review.Rat",
            "review.Mouse",
            "review.Hamster"
    };

    @SuppressWarnings("unchecked")
    private static void loader() {
        try {
            for (String name : typeNames)
                types.add((Class<? extends Pet>) Class.forName(name));
        } catch (ClassNotFoundException e) {
            throw new RuntimeException(e);
        }
    }

    static {
        loader();
    }

    public List<Class<? extends Pet>> types() {
        return types;
    }
}

public class PetCount {
    static class PetCounter extends HashMap<String,Integer> {
        public void count(String type) {
            Integer quantity = get(type);
            if(quantity == null)
                put(type, 1);
            else
                put(type, quantity + 1);
        }
    }
    public static void countPets(PetCreator creator) {
        PetCounter counter= new PetCounter();
        for(Pet pet : creator.createArray(20)) {
            // List each individual pet:
            System.out.println(pet.getClass().getSimpleName() + " ");
            if(pet instanceof Pet)
                counter.count("Pet");
            if(pet instanceof Dog)
                counter.count("Dog");
            if(pet instanceof Mutt)
                counter.count("Mutt");
            if(pet instanceof Pug)
                counter.count("Pug");
            if(pet instanceof Cat)
                counter.count("Cat");
            if(pet instanceof Manx)
                counter.count("EgyptianMau");
            if(pet instanceof Manx)
                counter.count("Manx");
            if(pet instanceof Manx)
                counter.count("Cymric");
            if(pet instanceof Rodent)
                counter.count("Rodent");
            if(pet instanceof Rat)
                counter.count("Rat");
            if(pet instanceof Mouse)
                counter.count("Mouse");
            if(pet instanceof Hamster)
                counter.count("Hamster");
        }
        // Show the counts:
        System.out.println();
        System.out.println(counter);
    }
    public static void main(String[] args) {
        countPets(new ForNameCreator());
    }
}

// output:
// Rat 
// Manx 
// Cymric 
// Mutt 
// Pug 
// Cymric 
// Pug 
// Manx 
// Cymric 
// Rat 
// EgyptianMau 
// Hamster 
// EgyptianMau 
// Mutt 
// Mutt 
// Cymric 
// Mouse 
// Pug 
// Mouse 
// Cymric 

// {EgyptianMau=7, Pug=3, Rat=2, Cymric=7, Mouse=2, Cat=9, Manx=7, Rodent=5, Mutt=3, Dog=6, Pet=20, Hamster=1}
```

提示：当你的程序中充满了大量的 instanceof 判断，那么你的成程序很可能有缺陷

### Using class literals

如果我们使用 PetCreator 的类字面量(.class)来重构它的实现，省去了 try-catch block, 而且表达的语义更清晰, 程序会更明了：

```java
import java.util.Arrays;
import java.util.Collections;
import java.util.List;

public class LiteralPetCreator extends PetCreator {
    // No try block needed.
    @SuppressWarnings("unchecked")
    public static final List<Class<? extends Pet>> allTypes =
            Collections.unmodifiableList(Arrays.asList(
                    Pet.class, Dog.class, Cat.class, Rodent.class,
                    Mutt.class, Pug.class, EgyptianMau.class, Manx.class,
                    Cymric.class, Rat.class, Mouse.class, Hamster.class));
    // Types for random creation:
    private static final List<Class<? extends Pet>> types =
            allTypes.subList(allTypes.indexOf(Mutt.class), allTypes.size());

    public List<Class<? extends Pet>> types() {
        return types;
    }

    public static void main(String[] args) {
        System.out.println(types);
    }
}
// output:
// [class review.Mutt, class review.Pug, class review.EgyptianMau, class review.Manx, class review.Cymric, class review.Rat, class review.Mouse, class review.Hamster]
```

新建一个 Pets 工具类用来创建创建 pet, 书上管这种方式叫做 Facade 模式。

```java
import java.util.ArrayList;

public class Pets {
    public static final PetCreator creator =
            new LiteralPetCreator();

    public static Pet randomPet() {
        return creator.randomPet();
    }

    public static Pet[] createArray(int size) {
        return creator.createArray(size);
    }

    public static ArrayList<Pet> arrayList(int size) {
        return creator.arrayList(size);
    }
}
```

This also provides indirection to randomPet( ), createArray( ) and arrayList( ).
Because PetCount.countPets( ) takes a PetCreator argument, we can easily test the
LiteralPetCreator (via the above Facade): 

```java
public class PetCount2 {
    public static void main(String[] args) {
        PetCount.countPets(Pets.creator);
    }
}
```

### A dynamic instanceof

新建一个 PetCount 继承自 LinedHashMap, 这种从现成的 Map 对象继承的做法我以前到是没怎么见过，也没怎么用过，长见识了，而且用起来挺方便的。

`Class.isInstance()` 效果上和 `instanceof` 等价，继承 map 之后通过调用 entrySet() 拿到所有的 entry, 然后通过范型遍历，节省了很多代码，和之前那一串 forName 相比干净了很多。

```java
public class PetCount3 {
    static class PetCounter extends LinkedHashMap<Class<? extends Pet>,Integer> {
        public PetCounter() {
            super(LiteralPetCreator.allTypes.stream().collect(Collectors.toMap(Function.identity(), x->0)));
        }
        public void count(Pet pet) {
            // Class.isInstance() eliminates instanceof:
            for(Map.Entry<Class<? extends Pet>, Integer> pair : entrySet())
                if(pair.getKey().isInstance(pet))
                    put(pair.getKey(), pair.getValue() + 1);
        }
        public String toString() {
            StringBuilder result = new StringBuilder("{");
            for(Map.Entry<Class<? extends Pet>,Integer> pair
                    : entrySet()) {
                result.append(pair.getKey().getSimpleName());
                result.append("=");
                result.append(pair.getValue());
                result.append(", ");
            }
            result.delete(result.length()-2, result.length());
            result.append("}");
            return result.toString();
        }
    }
    public static void main(String[] args) {
        PetCounter petCount = new PetCounter();
        for(Pet pet : Pets.createArray(20)) {
            System.out.println(pet.getClass().getSimpleName() + " ");
            petCount.count(pet);
        }
        System.out.println();
        System.out.println(petCount);
    }
}
```

### Counting recursively

除了 Class.isInstance() 还可以使用 isAssignFrom 来做类型判断，示例如下

```java
public class TypeCounter extends HashMap<Class<?>,Integer> {
    private Class<?> baseType;
    public TypeCounter(Class<?> baseType) {
        this.baseType = baseType;
    }
    public void count(Object obj) {
        Class<?> type = obj.getClass();
        if(!baseType.isAssignableFrom(type))
            throw new RuntimeException(obj + " incorrect type: "
                    + type + ", should be type or subtype of "
                    + baseType);
        countClass(type);
    }
    private void countClass(Class<?> type) {
        Integer quantity = get(type);
        put(type, quantity == null ? 1 : quantity + 1);
        Class<?> superClass = type.getSuperclass();
        if(superClass != null &&
                baseType.isAssignableFrom(superClass))
            countClass(superClass);
    }
    public String toString() {
        StringBuilder result = new StringBuilder("{");
        for(Map.Entry<Class<?>,Integer> pair : entrySet()) {
            result.append(pair.getKey().getSimpleName());
            result.append("=");
            result.append(pair.getValue());
            result.append(", ");
        }
        result.delete(result.length()-2, result.length());
        result.append("}");
        return result.toString();
    }
}

public class PetCount4 {
    public static void main(String[] args) {
        TypeCounter counter = new TypeCounter(Pet.class);
        for(Pet pet : Pets.createArray(20)) {
            System.out.println(pet.getClass().getSimpleName() + " ");
            counter.count(pet);
        }
        System.out.println();
        System.out.println(counter);
    }
}
```

这几个示例其实就说明了一个点，使用 isInstance() 和 isAssignFrom() 可以绕开 forName 使得代码整洁，好看很多。整洁好看也就意味着更少的维护成本。

## Registered factories

上面的例子有一个问题，就是每次你新建一个 Pets 的子类，你必须去 LiteralPetCreator 中将这个新建的 Class 手动添加进去，未免有点累赘。这里有两种解决方案，一种就是新写一个工具类遍历代码，找到 Pets 的子类统一处理，另一种方案就是将所有的类放到一个地方统一管理，基类就是很好的一个地方，示例如下： 

```java
import java.util.ArrayList;
import java.util.List;
import java.util.Random;

interface Factory<T> { T create(); } 

class Part {
    public String toString() {
        return getClass().getSimpleName();
    }

    static List<Factory<? extends Part>> partFactories =
            new ArrayList<>();

    static {
        // Collections.addAll() gives an "unchecked generic
        // array creation ... for varargs parameter" warning.
        partFactories.add(new FuelFilter.Factory());
        partFactories.add(new AirFilter.Factory());
        partFactories.add(new CabinAirFilter.Factory());
        partFactories.add(new OilFilter.Factory());
        partFactories.add(new FanBelt.Factory());
        partFactories.add(new PowerSteeringBelt.Factory());
        partFactories.add(new GeneratorBelt.Factory());
    }

    private static Random rand = new Random(47);

    public static Part createRandom() {
        int n = rand.nextInt(partFactories.size());
        return partFactories.get(n).create();
    }
}

class Filter extends Part {
}

class FuelFilter extends Filter {
    // Create a Class Factory for each specific type:
    public static class Factory implements review.Factory<FuelFilter> {
        public FuelFilter create() {
            return new FuelFilter();
        }
    }
}

class AirFilter extends Filter {
    public static class Factory
            implements review.Factory<AirFilter> {
        public AirFilter create() {
            return new AirFilter();
        }
    }
}

class CabinAirFilter extends Filter {
    public static class Factory
            implements review.Factory<CabinAirFilter> {
        public CabinAirFilter create() {
            return new CabinAirFilter();
        }
    }
}

class OilFilter extends Filter {
    public static class Factory
            implements review.Factory<OilFilter> {
        public OilFilter create() {
            return new OilFilter();
        }
    }
}

class Belt extends Part {
}

class FanBelt extends Belt {
    public static class Factory
            implements review.Factory<FanBelt> {
        public FanBelt create() {
            return new FanBelt();
        }
    }
}

class GeneratorBelt extends Belt {
    public static class Factory
            implements review.Factory<GeneratorBelt> {
        public GeneratorBelt create() {
            return new GeneratorBelt();
        }
    }
}

class PowerSteeringBelt extends Belt {
    public static class Factory
            implements review.Factory<PowerSteeringBelt> {
        public PowerSteeringBelt create() {
            return new PowerSteeringBelt();
        }
    }
}

public class RegisteredFactories {
    public static void main(String[] args) {
        for (int i = 0; i < 10; i++)
            System.out.println(Part.createRandom());
    }
}

// output:
// GeneratorBelt
// CabinAirFilter
// GeneratorBelt
// AirFilter
// PowerSteeringBelt
// CabinAirFilter
// FuelFilter
// PowerSteeringBelt
// PowerSteeringBelt
// FuelFilter
```

## instanceof vs. Class equivalence