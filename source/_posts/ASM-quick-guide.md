---
title: ASM quick guide
date: 2020-09-07 15:29:05
categories:
- 编程
tags:
- java
- asm
---

通过本次实验对 ASM 这个字节码框架有一个基本的了解。实验必须是简单明了的，便于重复的。引用一段话很好的概括了 ASM 的功能

> 可以负责任的告诉大家，ASM只不过是通过 “Visitor” 模式将 “.class” 类文件的内容从头到尾扫描一遍。因此如果你抱着任何更苛刻的要求最后都将失望而归。

实验平台信息：
    MacOS + IDEA + ASM Bytecode Outline 插件

## 输出 Class 方法

准备测试用 class，通过 ASM 输出 class 中的方法名称



## 修改方法

实验内容：准备一个 HelloWorld.class 可以打印出 'Hello World' 字样。通过 ASM 框架使他在答应之前，之后都输出一些 debug 信息，调用时可以使用反射简化实验。

HelloWorld.java 文件内容

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

我们想通过 ASM 修改拿到的样板输出为 'Test start \n Hello World... \n Test end'，对应的 java code:

```java
public class Expected {
    public void sayHello() {
        System.out.println("Test start");
        System.out.println("Hello World...");
        System.out.println("Test end");
    }
}
```

选中文件，右键 -> Show Bytecode Outline 选中 ASMifield tab 可以看到转化后的代码

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

其中类似如下的代码使一些行号和变量的处理，可以删掉不要，不影响就过

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

运行后找到目标文件，在 IDEA 里面浏览一个，编辑器会自动给出反编译结果，可以发现，在目标语句前后已经加上了我们要的 'Test Start/End' 的 debug 语句了。