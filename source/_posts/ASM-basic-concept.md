---
title: ASM 中的一些基本概念
date: 2020-11-18 15:43:43
categories:
- 编程
tags:
- java
- asm
---

记录下收集到的 ASM 的基础概念和例子

## 参考资料

[ASM 官方 repo](https://gitlab.ow2.org/asm/asm/-/blob/master/asm/src/test/java/org/objectweb/asm/TypeTest.java)

## Type

Type 类代 field 和 method 的 descriptor 属性。就是类似 `(Ljava/lang/String;)V` 这种表达式。该类中所有的方法几乎都是静态的，提供的功能也基本一致，传入某种类型的参数，然后返回代表 descriptor 的 type 对象。当传入为 method 类型的参数时，type 中会包含 `参数 + 返回值` 类型信息。

```java
@Test
public void test() throws NoSuchMethodException {
    System.out.println(Type.getType(String.class));
    System.out.println(Type.getType(this.getClass().getMethod("testGetTypeFromDescriptor", String.class)));
    System.out.println(Arrays.toString(Type.getArgumentTypes("(Ljava/lang/String;)V")));
    System.out.println(Type.getType(this.getClass().getMethod("getList")));
    System.out.println(Type.getType(this.getClass().getMethod("getArr")));
}

public void testGetTypeFromDescriptor(final String descriptor) {}
public List<String> getList() {return Collections.emptyList();}
public String[] getArr() {return new String[]{"A"};}

// output:
// Ljava/lang/String;
// (Ljava/lang/String;)V
// [Ljava/lang/String;]
// ()Ljava/util/List;
// ()[Ljava/lang/String;
```

## Junit5 是怎么实现 Excutable 接口的？ 看不懂

```java
@Test
public void testConstructor_validApi() {
    Executable constructor = () -> new ClassVisitor(Opcodes.ASM4) {};

    assertDoesNotThrow(constructor);
}
```

## ClassReader

这个类可以看作字节码文件的读操作的入口，只负责读取，其他处理逻辑是在 XXXVisitor 里面实现的。

PS: 该类方法打印的类信息都是斜线 `/` 分割的

## 官方测试是怎么测试 ClassReader, ClassVisitor 等类的

ClassReader 其核心功能有两个，一个是解析文件流，拿到基本信息，比如编译版本，常量池信息等。另一个是定义 Visitor 接解析顺序。

测试时官方也分两类，第一类就是测试解析出来的信息，比如 'testGetClassName' 等，第二类就是测试流程的，比如 'testAccept_emptyVisitor'。

测试 Visitor 时也很简单，定义一个自己的 Visitor，官方示例中是定义一个 LogMethodVisitor，然后直接调用对应 visitor.visitMethodInsn 等方法，测试自定义在 LogMethodVisitor 里的逻辑是不是符合预期就行了。这个还是很有启发的，可以用这种方法来完善 TraceSonar 的项目。

## Class 文件的格式

以 ASM 官方例子的 `ClassVisitorTest.class` 为例，注意查看的是 `.class` 文件，不是 `.java` 文件。下载插件 BinEd 可以查看文件在不同进制下的内容。右键 class 文件 `open as Binary` 即可。

Class 文件在格式上有特殊的规定，比如以 16 进制打开 class 可以看到前 4 位值为：0xCAFE(B1100 1010 1111 1110/202 254)

```java
@Test
public void testReadByte() throws IOException {
    ClassReader classReader = new ClassReader(getClass().getName());
    System.out.println(bytesToHex(Arrays.copyOf(classReader.classFileBuffer, 2)));
}

private static final char[] HEX_ARRAY = "0123456789ABCDEF".toCharArray();
public static String bytesToHex(byte[] bytes) {
    char[] hexChars = new char[bytes.length * 2];
    for (int j = 0; j < bytes.length; j++) {
        int v = bytes[j] & 0xFF;
        hexChars[j * 2] = HEX_ARRAY[v >>> 4];
        hexChars[j * 2 + 1] = HEX_ARRAY[v & 0x0F];
    }
    return new String(hexChars);
}
// output: CAFE
```