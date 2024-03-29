---
title: 虚拟机类加载机制
date: 2021-09-03 18:50:36
categories:
- JVM
tags:
- 类加载
---

## 7.1 概述

Java 虚拟机把描述类的数据从 Class 文件加载到内存，并对数据进行校验，转换解析和初始化，最终形成可以被虚拟机直接使用的 Java 类型，这个过程被称作虚拟机的类加载机制。

## 7.2 类加载的时机

一个类从被加载到虚拟机内存中，到卸载为止，会经历七个步骤

* 加载 Loading
* 验证 Verification
* 准备 Preparation
* 解析 Resolution
* 初始化 Initialization
* 使用 Using
* 写在 Unloading

验证，准备，解析三部分统称为 链接 Linking

加载、验证、准备、初始化和卸载这五个阶段的顺序是确定的，而解析不一定，有些情况下可以在初始化后再开始，为了支持动态绑定

有且只有六种情况必须立即对类进行初始化

1. 遇到 new, getstatic, putstatic 或 invokestatic 这四条字节码指令时，如果类型没有进行过初始化，则需要先出发其初始化阶段。能够生产这四条指令的典型 Java 代码场景有：
   * 使用 new 关键字实例化对象的时候
   * 读取或设置一个类型的静态字段(被 final 修饰，已在编译期把结果放入常量池的静态字段除外)
   * 调用一个类型的静态方法的时候
2. 使用java.lang.reflect包的方法对类型进行反射调用的时候，如果类型没有进行过初始化，则需要先触发其初始化
3. 当初始化类的时候，如果发现其父类还没有进行过初始化，则需要先触发其父类的初始化
4. 当虚拟机启动时，用户需要指定一个要执行的主类（包含main()方法的那个类），虚拟机会先初始化这个主类
5. 当使用JDK 7新加入的动态语言支持时，如果一个java.lang.invoke.MethodHandle实例最后的解析结果为REF_getStatic、REF_putStatic、REF_invokeStatic、REF_newInvokeSpecial四种类型的方法句柄，并且这个方法句柄对应的类没有进行过初始化，则需要先触发其初始化
6. 当一个接口中定义了JDK 8新加入的默认方法（被default关键字修饰的接口方法）时，如果有这个接口的实现类发生了初始化，那该接口要在其之前被初始化

除此之外，所有引用类型的方式都不会出发初始化，称为被动引用。下面是三个被动引用的例子

```java
// 通过子类引用父类的静态字段，不会导致子类初始化
public class SuperClass {
    static {
        System.out.println("SuperClass init!");
    }

    public static int value = 123;
}

public class SubClass extends SuperClass {
    static {
        System.out.println("SubClass init!");
    }
}

public class NotInitialization {
    public static void main(String[] args) {
        System.out.println(SubClass.value);
    }
}

// SuperClass init!
// 123
```

只会输出 "SuperClass init!"，而不会输出 "SubClass init!"

```java
// 通过数组定义来引用类，不会出发此类的初始化

public class NotInitialization02 {
    public static void main(String[] args) {
        SuperClass[] sca = new SuperClass[10];
    }
}
```

这种情况并不会触发类的初始化，只会触发 "[Lclassloading.SuperClass" 的类初始化节点，它是虚拟机自动生成的，创建动作由 newarray 触发，代表一维数组。

第三个例子，对应第一条中的第二点，final 修饰的常量编译阶段通过常量传播优化，将值存入 NotInitialization03 类的常量池中，后面对 ConstClass.HELLOWORLD 的引用世纪都被转为 NotInitialization03 对自身常量池的引用。

```java
public class ConstClass {
    static {
        System.out.println("ConstClass init!");
    }

    public static final String HELLOWORLD = "hello world";
}

public class NotInitialization03 {
    public static void main(String[] args) {
        System.out.println(ConstClass.HELLOWORLD);
    }
}

// hello world
```

## 7.3 类加载的过程

下面具体介绍 加载，验证，准备，解析和初始化这五个阶段所执行的具体动作。

## 7.3.1 加载

加载是 类加载 的第一个阶段，会做三件事

1. 通过一个类的全限定名来获取定义此类的二进制字节流
2. 将这个字节流所代表的静态存储结构转化为方法区的运行时数据结构
3. 再内存中生成一个代表这个类的 java.lang.Class 对象，作为方法区这个类的各种数据的访问入口

PS: 对数组类型的加载，这里还有一些描述，以后有用到再看

### 7.3.1 验证

验证是链接阶段第一步，目的是确保 Class 文件的字节流中包含的信息符合 Java 虚拟机规范 的全部约束要求，保证这些信息被当作代码运行后不会危害虚拟机自身的安全。

验证阶段非常重要，这个阶段是否严谨直接决定虚拟机能否承受恶意代码的攻击。从代码量和耗费的执行性能角度讲，验证阶段工作量再虚拟机类加载过程中占了相当大的比重。

验证阶段大致会完成下面四个阶段的检测动作：文件格式验证，元数据验证，字节码验证和符号引用验证

**文件格式验证**

第一阶段要验证字节流是否符合 Class 文件格式规范，验证点包括

* 是否以魔数 0xCAFEBABE 开头
* 主、次版本号是否在当前Java虚拟机接受范围之内
* 常量池的常量中是否有不被支持的常量类型(检查常量tag标志)
* 指向常量的各种索引值中是否有指向不存在的常量或不符合类型的常量
* CONSTANT_Utf8_info型的常量中是否有不符合UTF-8编码的数据
* Class文件中各个部分及文件本身是否有被删除的或附加的其他信息
* ...

以上只列举了一部分

**元数据验证**

第二阶段是对字节码描述的信息进行语义分析，保证其描述的信息符合规范

* 这个类是否有父类（除了java.lang.Object之外，所有的类都应当有父类）
* 这个类的父类是否继承了不允许被继承的类（被final修饰的类）
* 如果这个类不是抽象类，是否实现了其父类或接口之中要求实现的所有方法
* 类中的字段、方法是否与父类产生矛盾（例如覆盖了父类的final字段，或者出现不符合规则的方法重载，例如方法参数都一致，但返回值类型却不同等
* ...

**字节码验证**

第三阶段是这个那个验证过程中最复杂的一个阶段，通过数据流分析和控制流分析，确定程序予以是合法的，符合逻辑的。

**符号引用验证**

最后一个阶段的校验行为发生在虚拟机将符号引用转化为直接引用的时候，这个转化动作将在链接的第三个阶段-解析阶段发生。符号引用验证可以看作是对类自身以外（常量池中的各种符号引用）的各类信息进行匹配性校验，通俗来说就是，该类是否缺少或者被禁止访问它依赖的某些外部类、方法、字段等资源。本阶段通常需要校验下列内容

### 7.3.2 准备

准备阶段是正式为类中定义的变量(即静态变量，被 static 修饰的变量)分配内存并设置类变量初始值的阶段，从概念上讲，这些变量所使用的内存都应该再方法区中进行分配，但方方法区本身就是一个逻辑上的区域。JDK7 之前 HotSpot 使用永久区，JDK8 之后类变量会和 Class 一起放在堆空间。这点再 4.3.1 验证过了。

准备阶段的赋值仅包括类变量，而不包括实例变量，实例变量将会在对象实例化时随着对象一起分配在 Java 堆中。其次，这里说的初始值指的是数据类型的零值，比如

```java
public static int value = 123;
```

准备阶段后，初始值为 0 而不是 123，此时尚未执行任何 Java 方法，把 123 赋值给 value 的方法存放在类构造器 <clinit>() 方法之中，所以赋值要到类初始化阶段才会被执行。Java 所有基本数据类型零值表如下

| data type | value    | data type | value |
| :-------- | :------- | :-------- | :---- |
| int       | 0        | boolean   | false |
| long      | 0L       | float     | 0.0f  |
| short     | (short)0 | double    | 0.0d  |
| char      | '\u0000' | reference | null  |
| byte      | (byte)0  |           |       |

例外情况是前面展示过的 final 的情况，比如

```java
public static final int value = 123;
```

编译时 Javac 将会为 value 生成 Constant Value 属性，在准备阶段虚拟机就会根据 Constant Value 的设置将value赋值为123。

### 7.3.4 解析

解析阶段是 Java 虚拟机将常量池内的符号引用替换为直接引用的过程

* 符号引用（Symbolic References）：符号引用以一组符号来描述所引用的目标，符号可以是任何形式的字面量，只要使用时能无歧义地定位到目标即可。
* 直接引用（Direct References）：直接引用是可以直接指向目标的指针、相对偏移量或者是一个能间接定位到目标的句柄。

PS: 中间穿插很多解析类型的解释，暂时用不到，跳过

### 7.3.5 初始化

类的初始化阶段是勒加载过程的最后一个步骤，之前介绍的几个动作，除了在加载阶段用户可以通过自定义类加载器的方式局部参与外，其余都由 Java 虚拟机来主导控制。直到初始化阶段，虚拟机才真正执行类中编写的 Java 程序代码，将主导权移交给应用程序。

初始化阶段就是执行类构造器 <clinit>() 方法的过程，它是Javac编译器的自动生成物。

<clinit>() 是由编译器自动收集类中所有类变量的赋值动作和静态语句块(static{})中的语句合并产生的，顺序由源文件中顺序决定。静态语句块只能访问它之前的变量。之后的变量可以赋值，但是不能访问。

```java
public class Test {
    static {
        i = 0;
        System.out.println(i); // 编译器提示 非法向前引用
    }

    static int i = 1;
}
```

<clinit>() 不需要显示调用父类构造器，虚拟机会保证在子类 <clinit>() 执行前，父类的 <clinit>() 已经执行完毕，所以 jvm 中第一个被执行的 <clinit>() 肯定是 java.lang.Object.

父类 <clinit>() 优先执行，即父类的静态语句块要优先于子类的变量赋值操作。下面例子中再子类使用变量之前，父类已经完成了静态块的调用

```java
public class Test {
    static class Parent {
        public static int A = 1;
        static {
            A = 2;
        }
    }

    static class Sub extends Parent {
        public static int B = A;
    }

    public static void main(String[] args) {
        System.out.println(Sub.B);
    }
}
// 2
```

<clinit>() 方法对于类或接口来说并不是必需的，如果一个类中没有静态语句块，也没有对变量的赋值操作，那么编译器可以不为这个类生成 <clinit>() 方法

接口仍有变量初始化赋值操作，也会生成 <clinit>() 方法。但与类不同，执行接口的 <clinit>() 不需要先执行弗雷接口的 <clinit>()。接口的实现类再初始化时也一样不会执行接口的 <clinit>() 方法。

Java虚拟机必须保证一个类的<clinit>()方法在多线程环境中被正确地加锁同步。如果一个类的 <clinit>() 方法有耗时很长的操作，很可能造成多个进程阻塞

```java
public class Test22 {
    static class DeadLoopClass {
        static {
            if (true) {
                System.out.println(Thread.currentThread() + " init DeadLoopClass");
                while(true) {}
            }
        }
    }

    public static void main(String[] args) {
        Runnable scritp = () -> {
            System.out.println(Thread.currentThread() + " start");
            new DeadLoopClass();
            System.out.println(Thread.currentThread() + " run over");
        };

        Thread thread1 = new Thread(scritp);
        Thread thread2 = new Thread(scritp);
        thread1.start();
        thread2.start();
    }
}
// Thread[Thread-0,5,main] start
// Thread[Thread-1,5,main] start
// Thread[Thread-0,5,main] init DeadLoopClass
```

## 7.4 类加载器

虚拟机设计团队有意把类加载阶段中 "通过一个类的全限定名来获取描述该类的二进制字节流" 的这个动作放到 Java 虚拟机外部去实现，以便让程序自己决定如何获取所需的类。实现这个动作的代码被称作-类加载器(Class Loader)。

这项技术原来是为了支持 Java Applet 而设计的，如今，Applet 已经淘汰了，但是类加载器却在 类层次划分，OSGi, 如部署，代码加密等领域大放异彩。

### 7.4.1 类与类加载器

类加载器虽然只用于实现类的加载动作，但他在 Java 程序中起到的作用却远超类加载阶段。任意类，必须由加载他的类加载器和这个类本身共同确立其在 jvm 中的唯一性，俄米格类加载器都拥有一个独立的类名称空间。

通俗讲：比较两个类是否相等，只有在两个类由同一个类加载器加载的前提下才有意义，否则，这两个类逼不想等。

这里所指的“相等”，包括代表类的Class对象的equals()方法，isAssignableFrom()方法、isInstance() 方法的返回结果，也包括了使用instanceof关键字做对象所属关系判定等各种情况

下面的例子中，我们自定义了一个 class loader 并加载当前测试类，生成实例。拿这个实例和默认类加载器的测试类进行 instanceof 的比较，结果为 false。

```java
public class ClassLoaderTest {
    public static void main(String[] args) throws Exception {
        ClassLoader myLoader = new ClassLoader() {
            @Override
            public Class<?> loadClass(String name) throws ClassNotFoundException {
                String fileName = name.substring(name.indexOf(".") + 1) + ".class";
                InputStream is = getClass().getResourceAsStream(fileName);
                if (is == null) {
                    return super.loadClass(name);
                }
                try {
                    byte[] b = new byte[is.available()];
                    is.read(b);
                    return defineClass(name, b, 0, b.length);
                } catch (IOException e) {
                    throw new ClassNotFoundException(name);
                }
            }
        };

        Object obj = myLoader.loadClass("classloading.ClassLoaderTest").newInstance();

        System.out.println(obj.getClass());
        System.out.println(obj instanceof classloading.ClassLoaderTest);
    }
}
// class classloading.ClassLoaderTest
// false
```

### 7.4.2 双亲委派模型

从虚拟机的角度看，只有两类加载器

* 启动类加载器(Bootstrap Class Loader): C++ 实现，虚拟机的一部分
* 其他加载器：Java 实现，独立与虚拟机外

开发人员角度看，可以分的更细致，可以分为三层类加载器

* 启动类加载器：加载存放在 <JAVA_HOME>\lib 或者 -Xbootstrapclasspth 参数指定的路径下，能被 JVM 识别的类库到内存中。不能被 Java 程序直接使用，编写自定义加载器时，返回 null 即可将加载委托给启动类加载器了
* 扩展类加载器(Extension Class Loader): 在类 sun.misc.Lanucher$ExtClassLoader 中，负责加载 <JAVA_HOME>\lib\ext 或者 java.ext.dirs 系统变量锁指定的路径中所有的类库。JDK9 之后，被模块化所替代。
* 应用程序类加载器(Application Class Loader): 由 sun.misc.Lanucher$AppClassLoader 实现。ClassLoader 的 getSystemClassLoader() 方法的返回值。记载用户类路径上所有类库。如果应用程序没有自定义过自己的类加载器，一般这个就是默认的类加载器。

{% plantuml %}
title 类加载器的双亲委派模型

node bcl [
启动类加载器
Bootstrap Class Loader
]

node ecl [
扩展类加载器
Extension Class Loader
]

node acl [
应用类加载器
Application Class Loader
]

node ucl1 [
自定义类加载器
User Class Loader
]

node ucl2 [
自定义类加载器
User Class Loader
]

bcl <-- ecl
ecl <-- acl

acl <-- ucl1
acl <-- ucl2
{% endplantuml%}

JDK9 之前的 Java 应用都是由这三类加载器互相配合来完成加载的。双亲委派模型要求，除了顶层的启动类加载器外，其余的类加载器都要有自己的父类加载器。这里的父子关系不是通过继承关系实现的，而是通过使用组合(Composition)关系来复用父加载器的代码。

双亲委派模型的工作过程是：如果一个类加载器收到了类加载请求，首先不会尝试自己去加载这个类，而是委派给父类加载器去完成，每次皆是如此。只有当父类无法实现这个加载请求时，子类才是尝试自己去加载。

使用亲委派模型的好处是具备了一种带有优先级的层次关系。例如 java.lang.Object, 无论那哪个加载器加载它，最终都会使用 rt.jar 下的 Object 定义。如果不用这个模型，用户在 ClassPath 下定义一个 java.lang.Object 类, 系统中就会出现多个不同的 Object 类，Java 类型体系中最基础的行为就无法得到保证了。

下面是 ClassLoader 中加载类的代码实现。先尝试查这个类是否已经被加载。再看是否有父加载器，如果没有则使用启动类加载器加载。如果还是没有找到类，则尝试用自己加载。

```java
protected Class<?> loadClass(String name, boolean resolve)
        throws ClassNotFoundException
    {
        synchronized (getClassLoadingLock(name)) {
            // First, check if the class has already been loaded
            Class<?> c = findLoadedClass(name);
            if (c == null) {
                long t0 = System.nanoTime();
                try {
                    if (parent != null) {
                        c = parent.loadClass(name, false);
                    } else {
                        c = findBootstrapClassOrNull(name);
                    }
                } catch (ClassNotFoundException e) {
                    // ClassNotFoundException thrown if class not found
                    // from the non-null parent class loader
                }

                if (c == null) {
                    // If still not found, then invoke findClass in order
                    // to find the class.
                    long t1 = System.nanoTime();
                    c = findClass(name);

                    // this is the defining class loader; record the stats
                    sun.misc.PerfCounter.getParentDelegationTime().addTime(t1 - t0);
                    sun.misc.PerfCounter.getFindClassTime().addElapsedTimeFrom(t1);
                    sun.misc.PerfCounter.getFindClasses().increment();
                }
            }
            if (resolve) {
                resolveClass(c);
            }
            return c;
        }
    }
```

### 7.4.3 破坏双亲委派模型

双亲委派模式是推荐方式，而不是强制性约束。直到 JDK9 的模块化为止，主要出现过 3 次较大规模的被破坏情况。

第一次是 JDK1.2 之前，双亲委派模型还没出现，但是类加载器的概念和抽象类已经引入了。

第二次被破坏是由于这个模型自身的缺陷导致的。这个模型很好的解决了各个类加载器协作时基础类型的一致性问题，但程序设计往往没有绝对不变的完美规则，如果基础类型又要调用回用户的代码，该如何。典型的例子便是 JNDI 服务。JDNI 现在已经是 Java 的标准服务，代码由启动类加载器来完成加载(JDK1.3 时加入到 rt.jar)，肯定属于 Java 中很基础的类型了。JNDI 目的是对资源进行查找和集中管理，它需要调用由其他厂商实现并部署在应用程序的 ClassPath 下的 JNDI 服务提供者接口(Service Provider Interface, SPI) 的代码，现在问题来了，启动类加载器是绝不可能认识，加载这些代码的。

为了解决这个困境，Java 设计团队引入了一个不太优雅的设计：线程上下文类加载器(Thread Context ClassLoader). 这个加载器可以通过 java.lang.Thread 的 setContextClassLoader() 方法进行设置，如果创建线程时未设置，将会从父类线程中继承一个，如果再应用程序的全局范围内都没有设置过的话，那这个类加载器默认就是应用程序类加载器。JNDI 使用这个线程上下文类加载器去加载所需的 SPI 服务代码，这个是一个类加载器去请求子类加载器完成类加载的行为，实际上打通了双亲委派模型的层次结构来逆向使用类加载器，Java 中涉及 SPI 的加载基本都采用这种方式，比如 JNDI，JDBC，JCE，JAXB 和 JBI 等。当 SPI 多于一个时，只能采用硬编码，为了消除这种不雅的实现，JDK6 提供了 java.util.ServiceLoader 配合 META-INF/services 中的配置信息，辅以责任链模式，才算给 SPI 提供了一种相对合理的解决方案。

第三次被破坏是由于用户对程序动态性的追求导致的。典型应用场景有：代码热部署，模块热部署等。即像电脑外设一般，在没有重新启动的情况下完成功能升级。现在比较热门的实现是 IBM 的 OSGi, Oracle 的 Jigsaw 。后面还有一些 OSGi 的介绍，但是没用过，看看就过了，用到再说。

### 7.5 Java 模块化系统

pass, 暂时用不到