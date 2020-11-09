---
title: ASM 快速入门
date: 2020-09-07 15:29:05
categories:
- 编程
tags:
- java
- asm
---

通过本次实验对 ASM 这个字节码框架有一个基本的了解。实验必须是简单明了，方便重现的。引用一段话很好的概括了 ASM 的功能

> 可以负责任的告诉大家，ASM只不过是通过 “Visitor” 模式将 “.class” 类文件的内容从头到尾扫描一遍。因此如果你抱着任何更苛刻的要求最后都将失望而归。

实验平台信息：
    MacOS + IDEA + ASM Bytecode Outline 插件

## 输出 Class 方法

准备测试用 class，通过 ASM 输出 class 中的方法名称

```java

public interface MyInterface01 {}

public interface MyInterface02 {}

public class SayHello implements MyInterface01, MyInterface02 {
    public void say() {
        String name = "Jack";
        System.out.println("Hello" + name);
    }
}
```

右键准备的测试文件，选中 'Show bytecode outline' 选项，点击 Bytecode tab, 查看内容可以看到字节码如下

```bytecode
// class version 52.0 (52)
// access flags 0x21
public class sorra/tracesonar/mytest/SayHello implements sorra/tracesonar/mytest/MyInterface01 sorra/tracesonar/mytest/MyInterface02  {

  // compiled from: SayHello.java

  // access flags 0x1
  public <init>()V
   L0
    LINENUMBER 3 L0
    ALOAD 0
    INVOKESPECIAL java/lang/Object.<init> ()V
    RETURN
   L1
    LOCALVARIABLE this Lsorra/tracesonar/mytest/SayHello; L0 L1 0
    MAXSTACK = 1
    MAXLOCALS = 1

  // access flags 0x1
  public say()V
   L0
    LINENUMBER 5 L0
    LDC "Jack"
    ASTORE 1
   L1
    LINENUMBER 6 L1
    GETSTATIC java/lang/System.out : Ljava/io/PrintStream;
    NEW java/lang/StringBuilder
    DUP
    INVOKESPECIAL java/lang/StringBuilder.<init> ()V
    LDC "Hello"
    INVOKEVIRTUAL java/lang/StringBuilder.append (Ljava/lang/String;)Ljava/lang/StringBuilder;
    ALOAD 1
    INVOKEVIRTUAL java/lang/StringBuilder.append (Ljava/lang/String;)Ljava/lang/StringBuilder;
    INVOKEVIRTUAL java/lang/StringBuilder.toString ()Ljava/lang/String;
    INVOKEVIRTUAL java/io/PrintStream.println (Ljava/lang/String;)V
   L2
    LINENUMBER 7 L2
    RETURN
   L3
    LOCALVARIABLE this Lsorra/tracesonar/mytest/SayHello; L0 L3 0
    LOCALVARIABLE name Ljava/lang/String; L1 L3 1
    MAXSTACK = 3
    MAXLOCALS = 2
}
```

测试用例

```java
public class ASMTest {
    public static void main(String[] args) throws IOException {
        System.out.println("--- START ---");
        ClassReader cr = new ClassReader(SayHello.class.getName());
        cr.accept(new DemoClassVisitor(), 0);
        System.out.println("--- END ---");
    }
}

class DemoClassVisitor extends ClassVisitor {
    public DemoClassVisitor() {
        super(Opcodes.ASM5);
    }

    // Called when access file header, so it will called only once for each class
    @Override
    public void visit(int version, int access, String name, String signature, String superName, String[] interfaces) {
        super.visit(version, access, name, signature, superName, interfaces);
        System.out.println("invoke visit method, params: " + version + ", " + access + ", " + name + ", " + signature + ", " + superName + ", " + Arrays.toString(interfaces));
    }

    // Called when access method
    @Override
    public MethodVisitor visitMethod(int access, String name, String desc, String signature, String[] exceptions) {
        System.out.println("at Method " + name);
        //
        MethodVisitor superMV = super.visitMethod(access, name, desc, signature, exceptions);
        return new DemoMethodVisitor(superMV, name);
    }
}

class DemoMethodVisitor extends MethodVisitor {
    private String methodName;
    public DemoMethodVisitor(MethodVisitor mv, String methodName) {
        super(Opcodes.ASM5, mv);
        this.methodName = methodName;
    }
    public void visitCode() {
        System.out.println("at Method ‘" + methodName + "’ Begin...");
        super.visitCode();
    }

    @Override
    public void visitLocalVariable(String name, String desc, String signature, Label start, Label end, int index) {
        super.visitLocalVariable(name, desc, signature, start, end, index);
        System.out.println("Params in visitLocalVariable: " + name + ", " + desc + ", " + signature + ", " + start + ", " + end + ", " + index);
    }

    public void visitEnd() {
        System.out.println("at Method ‘" + methodName + "’End.");
        super.visitEnd();
    }
}
```

终端输出

```txt
--- START ---
invoke visit method, params: 52, 33, sorra/tracesonar/mytest/SayHello, null, java/lang/Object, [sorra/tracesonar/mytest/MyInterface01, sorra/tracesonar/mytest/MyInterface02]
at Method <init>
at Method ‘<init>’ Begin...
Params in visitLocalVariable: this, Lsorra/tracesonar/mytest/SayHello;, null, L662441761, L1618212626, 0
at Method ‘<init>’End.
at Method say
at Method ‘say’ Begin...
Params in visitLocalVariable: this, Lsorra/tracesonar/mytest/SayHello;, null, L1129670968, L1023714065, 0
Params in visitLocalVariable: name, Ljava/lang/String;, null, L2051450519, L1023714065, 1
at Method ‘say’End.
--- END ---
```

想要理解 ASM 运行方式，需要结合前面的 bytecode 内容。比如 `visitLocalVariable` 方法其实就是将 bytecode 里面对应的 LOCALVARIABLE 信息打印出来。

## MethodVisitor 的 visitMethodInsn 方法简单例子

根据查到的资料，该方法可以知道当前的方法调用了其他类的什么方法，设计用例如下: Class A 有 method a, Class B 有 method b, a 中包含对 b 的调用，使用 visitMethodInsn 解析 a 方法是应该可以拿到这层关系

```java
public class ClassA {
    ClassB b = new ClassB();

    public void methodA() {
        b.methodB();
    }
}

public class ClassB {
    public void methodB() {
        System.out.println("Method B called...");
    }
}
```

class A 的 bytecode 显示如下

```bytecode
// class version 52.0 (52)
// access flags 0x21
public class com/jzheng/asmtest/ClassA {

  // compiled from: ClassA.java

  // access flags 0x0
  Lcom/jzheng/asmtest/ClassB; b

  // access flags 0x1
  public <init>()V
   L0
    LINENUMBER 3 L0
    ALOAD 0
    INVOKESPECIAL java/lang/Object.<init> ()V
   L1
    LINENUMBER 4 L1
    ALOAD 0
    NEW com/jzheng/asmtest/ClassB
    DUP
    INVOKESPECIAL com/jzheng/asmtest/ClassB.<init> ()V
    PUTFIELD com/jzheng/asmtest/ClassA.b : Lcom/jzheng/asmtest/ClassB;
    RETURN
   L2
    LOCALVARIABLE this Lcom/jzheng/asmtest/ClassA; L0 L2 0
    MAXSTACK = 3
    MAXLOCALS = 1

  // access flags 0x1
  public methodA()V
   L0
    LINENUMBER 7 L0
    ALOAD 0
    GETFIELD com/jzheng/asmtest/ClassA.b : Lcom/jzheng/asmtest/ClassB;
    INVOKEVIRTUAL com/jzheng/asmtest/ClassB.methodB ()V
   L1
    LINENUMBER 8 L1
    RETURN
   L2
    LOCALVARIABLE this Lcom/jzheng/asmtest/ClassA; L0 L2 0
    MAXSTACK = 1
    MAXLOCALS = 1
}
```

可以看到在 `methodA()V` block 里有对 ClassB 的方法调用说明 `INVOKEVIRTUAL com/jzheng/asmtest/ClassB.methodB ()V`，通过它我们可以知道当前方法对其他类方法的调用

测试用例：

```java
public class ASMTest {
    public static void main(String[] args) throws IOException {
        System.out.println("--- START ---");
        ClassReader cr = new ClassReader(ClassA.class.getName());
        cr.accept(new DemoClassVisitor(), 0);
        System.out.println("--- END ---");
    }
}

class DemoClassVisitor extends ClassVisitor {
    public DemoClassVisitor() {
        super(Opcodes.ASM5);
    }

    // Called when access method
    @Override
    public MethodVisitor visitMethod(int access, String name, String desc, String signature, String[] exceptions) {
        System.out.println("at Method " + name);

        super.visitMethod(access, name, desc, signature, exceptions);
        return new MethodVisitor(Opcodes.ASM5) {
            @Override
            public void visitMethodInsn(int opcode, String owner, String name, String desc, boolean itf) {
                super.visitMethodInsn(opcode, owner, name, desc, itf);
                System.out.println(String.format("opcode: %s, owner: %s, name: %s, desc: %s, itf: %s", opcode, owner, name, desc, itf));
            }
        };
    }
}
// output:
// --- START ---
// at Method <init>
// opcode: 183, owner: java/lang/Object, name: <init>, desc: ()V, itf: false
// opcode: 183, owner: com/jzheng/asmtest/ClassB, name: <init>, desc: ()V, itf: false
// at Method methodA
// opcode: 182, owner: com/jzheng/asmtest/ClassB, name: methodB, desc: ()V, itf: false
// --- END ---
```

## 修改方法

实验内容：准备一个 HelloWorld.class 可以打印出 'Hello World' 字样。通过 ASM 框架使他在打印之前和之后都输出一些 debug 信息，调用时可以使用反射简化实验。

测试用 class

```java
public class HelloWorld {
    public void sayHello() {
        System.out.println("Hello World...");
    }
}
```

测试用例，通过反射拿到测试方法并调用查看输出。

```java
import java.lang.reflect.Method;

public class main {
    public static void main(String[] args) throws Exception {
        Class cls = Class.forName("sorra.tracesonar.main.aopsample.HelloWorld");
        
        Method sayHello = cls.getDeclaredMethod("sayHello");
        sayHello.invoke(cls.newInstance());
    }
}

// run and get output:
// Hello World...
```

预期目标：通过 ASM 修改目标 class 使得输出为 'Test start \n Hello World... \n Test end'，对应的 java code:

```java
public class Expected {
    public void sayHello() {
        System.out.println("Test start");
        System.out.println("Hello World...");
        System.out.println("Test end");
    }
}
```

选中 java 文件，右键 -> Show Bytecode Outline 选中 ASMifield tab 可以看到转化后的代码

```java
package asm.sorra.tracesonar.main.aopsample;

import java.util.*;

import org.objectweb.asm.*;

public class ExpectedDump implements Opcodes {

    public static byte[] dump() throws Exception {

        ClassWriter cw = new ClassWriter(0);
        FieldVisitor fv;
        MethodVisitor mv;
        AnnotationVisitor av0;

        cw.visit(52, ACC_PUBLIC + ACC_SUPER, "sorra/tracesonar/main/aopsample/Expected", null, "java/lang/Object", null);

        cw.visitSource("Expected.java", null);

        {
            mv = cw.visitMethod(ACC_PUBLIC, "<init>", "()V", null, null);
            mv.visitCode();
            Label l0 = new Label();
            mv.visitLabel(l0);
            mv.visitLineNumber(3, l0);
            mv.visitVarInsn(ALOAD, 0);
            mv.visitMethodInsn(INVOKESPECIAL, "java/lang/Object", "<init>", "()V", false);
            mv.visitInsn(RETURN);
            Label l1 = new Label();
            mv.visitLabel(l1);
            mv.visitLocalVariable("this", "Lsorra/tracesonar/main/aopsample/Expected;", null, l0, l1, 0);
            mv.visitMaxs(1, 1);
            mv.visitEnd();
        }
        {
            mv = cw.visitMethod(ACC_PUBLIC, "sayHello", "()V", null, null);
            mv.visitCode();
            Label l0 = new Label();
            mv.visitLabel(l0);
            mv.visitLineNumber(5, l0);
            mv.visitFieldInsn(GETSTATIC, "java/lang/System", "out", "Ljava/io/PrintStream;");
            mv.visitLdcInsn("Test start");
            mv.visitMethodInsn(INVOKEVIRTUAL, "java/io/PrintStream", "println", "(Ljava/lang/String;)V", false);
            Label l1 = new Label();
            mv.visitLabel(l1);
            mv.visitLineNumber(6, l1);
            mv.visitFieldInsn(GETSTATIC, "java/lang/System", "out", "Ljava/io/PrintStream;");
            mv.visitLdcInsn("Hello World...");
            mv.visitMethodInsn(INVOKEVIRTUAL, "java/io/PrintStream", "println", "(Ljava/lang/String;)V", false);
            Label l2 = new Label();
            mv.visitLabel(l2);
            mv.visitLineNumber(7, l2);
            mv.visitFieldInsn(GETSTATIC, "java/lang/System", "out", "Ljava/io/PrintStream;");
            mv.visitLdcInsn("Test end");
            mv.visitMethodInsn(INVOKEVIRTUAL, "java/io/PrintStream", "println", "(Ljava/lang/String;)V", false);
            Label l3 = new Label();
            mv.visitLabel(l3);
            mv.visitLineNumber(8, l3);
            mv.visitInsn(RETURN);
            Label l4 = new Label();
            mv.visitLabel(l4);
            mv.visitLocalVariable("this", "Lsorra/tracesonar/main/aopsample/Expected;", null, l0, l4, 0);
            mv.visitMaxs(2, 1);
            mv.visitEnd();
        }
        cw.visitEnd();

        return cw.toByteArray();
    }
}
```

其中类似如下的代码使一些行号和变量的处理，可以删掉不要，不影响结果

```java
Label l0 = new Label();
mv.visitLabel(l0);
mv.visitLineNumber(3, l0);
...
Label l4 = new Label();
mv.visitLabel(l4);
mv.visitLocalVariable("this", "Lsorra/tracesonar/main/aopsample/Expected;", null, l0, l4, 0);
```

将自动生成的文件里的冗余语句删掉，加一个 main 方法，生成文件并存放到根目录下

```java
public class ExpectedDump {
    public static byte[] dump() throws Exception {
        ...
        return cw.toByteArray();
    }

    public static void main(String[] args) throws Exception {
        byte[] updated = dump();

        try (FileOutputStream fos = new FileOutputStream("Expected.class")) {
            fos.write(updated);
        }
        System.out.println("Write success...");
    }
}
```

运行该 Java 文件，可以看到 project 的根目录下有生成一个名为 'Expected.class' 的文件，在 IDEA 里面浏览它，编辑器会自动给出反编译结果，可以发现，在目标语句前后已经加上了我们要的 'Test Start/End' 的 debug 语句了。