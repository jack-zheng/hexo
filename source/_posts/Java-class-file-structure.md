---
title: Java class 文件架构摘要
date: 2020-11-19 10:45:30
categories:
- 编程
tags:
- todo
- jvm
- asm
---

目标：通过阅读 深入理解 JVM 虚拟机 第三版 的第 6 章，结合 ASM 里的 Reader 和 Visitor 对 class 文件有个一比较深入的了解。

* [bilibili 参考视频](https://www.bilibili.com/video/BV12y4y1B7pn?p=12)，很棒，白嫖

## 6.2 无关性的基石

Java 虚拟机不与包括 Java 语言在内的任何语言绑定，它只与 Class 文件这种特定的二进制文件格式所关联，Class 文件中包含了 Java 虚拟机指令集，符号表以及若干其他辅助信息。

## 6.3 Class 类文件的结构

> Idea 安装 BinEd 插件可以查看 Class 文件在各种进制下的值

Class 文件是一组以 8 个字节为基础单位的二进制流，各个数据项目严格按照顺序紧凑排列在文件之中，中间没有添加任何分隔符，这使得整个 Class 文件中存储的内容几乎全部是程序运行的必要数据，没有空隙存在。

Class 文件结构中只有两种数据类型：无符号数 + 表

* 无符号数：是基本数据类，以 u1, u2, u4, u8 分别表示 1，2，4，8 个字节的无符号数。无符号数可以用来描述数字，索引引用，数量值或者按照 UTF-8 编码构成的字符串值
* 表：n 个无符号数 + 其他表构成的复合数据类型，命名习惯性的以 _info 结尾。表用于描述有层次关系的复合结构数据，整个 Class 可以看作一张表。

Class 文件格式表：

| Type           | Name                | Count                 |
| :------------- | :------------------ | :-------------------- |
| u4             | magic               | 1                     |
| u2             | minor_version       | 1                     |
| u2             | major_version       | 1                     |
| u2             | constant_pool_count | 1                     |
| cp_info        | constant_pool       | constant_pool_count-1 |
| u2             | access_flags        | 1                     |
| u2             | this_class          | 1                     |
| u2             | super_class         | 1                     |
| u2             | interfaces_count    | 1                     |
| u2             | interfaces          | interfaces_count      |
| u2             | fields_count        | 1                     |
| field_info     | fields              | fields_count          |
| u2             | methods_count       | 1                     |
| method_info    | methods             | methods_count         |
| u2             | attributes_count    | 1                     |
| attribute_info | attributes          | attributes_count      |

### 6.3.1 魔数与 Class 文件的版本

Class 文件前 4 个字节被称为魔数，值为 0xCAFEBABE，用来表明文件类型。紧接着 4 个字节为主次版本号。

Java 虚拟机规范在 Class 文件校验部分明确要求，即使文件格式并未发生任何变化，虚拟机也必须**拒绝执行**超过其版本号的 Class 文件。

| JDK version | version number |
| :---------- | :------------- |
| JDK 13      | 57             |
| JDK 12      | 56             |
| JDK 11      | 55             |
| JDK 10      | 54             |
| JDK 9       | 53             |
| JDK 8       | 52             |
| JDK 7       | 51             |
| JDK 6.0     | 50             |
| JDK 5.0     | 49             |
| JDK 1.4     | 48             |
| JDK 1.3     | 47             |
| JDK 1.2     | 46             |
| JDK 1.1     | 45             |

仿照参考书写下测试代码, 不知道是不是编译器版本不一样，结果从常量池开始有些许偏差，不过无伤大雅，学习路径，方法还是一样的。

```java
package c631;

public class TestClass {
    private int m;

    public int inc() {
        return m + 1;
    }
}
```

Class 文件 16 进制表达式

|    line    | 00 01 02 03 04 05 06 07 08 09 0a 0b 0c 0d 0e 0f | hex value        |
| :--------: | :---------------------------------------------- | :--------------- |
| 0000000000 | ca fe ba be 00 00 00 32 00 16 0a 00 04 00 12 09 | .......2........ |
| 0000000010 | 00 03 00 13 07 00 14 07 00 15 01 00 01 6d 01 00 | .............m.. |
| 0000000020 | 01 49 01 00 06 3c 69 6e 69 74 3e 01 00 03 28 29 | .I...<init>...() |
| 0000000030 | 56 01 00 04 43 6f 64 65 01 00 0f 4c 69 6e 65 4e | V...Code...LineN |
| 0000000040 | 75 6d 62 65 72 54 61 62 6c 65 01 00 12 4c 6f 63 | umberTable...Loc |
| 0000000050 | 61 6c 56 61 72 69 61 62 6c 65 54 61 62 6c 65 01 | alVariableTable. |
| 0000000060 | 00 04 74 68 69 73 01 00 10 4c 63 36 33 31 2f 54 | ..this...Lc631/T |
| 0000000070 | 65 73 74 43 6c 61 73 73 3b 01 00 03 69 6e 63 01 | estClass;...inc. |
| 0000000080 | 00 03 28 29 49 01 00 0a 53 6f 75 72 63 65 46 69 | ..()I...SourceFi |
| 0000000090 | 6c 65 01 00 0e 54 65 73 74 43 6c 61 73 73 2e 6a | le...TestClass.j |
| 00000000a0 | 61 76 61 0c 00 07 00 08 0c 00 05 00 06 01 00 0e | ava............. |
| 00000000b0 | 63 36 33 31 2f 54 65 73 74 43 6c 61 73 73 01 00 | c631/TestClass.. |
| 00000000c0 | 10 6a 61 76 61 2f 6c 61 6e 67 2f 4f 62 6a 65 63 | .java/lang/Objec |
| 00000000d0 | 74 00 21 00 03 00 04 00 00 00 01 00 02 00 05 00 | t.!............. |
| 00000000e0 | 06 00 00 00 02 00 01 00 07 00 08 00 01 00 09 00 | ................ |
| 00000000f0 | 00 00 2f 00 01 00 01 00 00 00 05 2a b7 00 01 b1 | ../........*.... |
| 0000000100 | 00 00 00 02 00 0a 00 00 00 06 00 01 00 00 00 03 | ................ |
| 0000000110 | 00 0b 00 00 00 0c 00 01 00 00 00 05 00 0c 00 0d | ................ |
| 0000000120 | 00 00 00 01 00 0e 00 0f 00 01 00 09 00 00 00 31 | ...............1 |
| 0000000130 | 00 02 00 01 00 00 00 07 2a b4 00 02 04 60 ac 00 | ........*....`.. |
| 0000000140 | 00 00 02 00 0a 00 00 00 06 00 01 00 00 00 07 00 | ................ |
| 0000000150 | 0b 00 00 00 0c 00 01 00 00 00 07 00 0c 00 0d 00 | ................ |
| 0000000160 | 00 00 01 00 10 00 00 00 02 00 11                | ...........      |

魔数值 cafe, 版本号 `00 00 00 32` 转化后位 50 和我在 pom 指定的 1.6 版本 JDK 编译一致

### 6.3.2 常量池

第 9-8 个字节表示常量池。常量池是从 1 开始的。示例中对应的值位 `00 16 - 22` 表明常量池总共有 21 个值。

PS: 常量池的 0 位空余，是为了考虑特殊情况。当指向常量池的数据需要表达 `不引用任何常量池中的项目` 这样的意思时，可以将索引值设置位 0 表示。

常量池主要存放两大类的常量：字面量 Literal + 符号引用 Symbolic References。字面量接近于 Java 语言层面的常量概念，符号引用则属于编译原理的概念主要包括下面几类常量：

1. 被模块导出或者开放的包 package
2. 类和接口的全名限定 Fully Qualified Name
3. 字段名称和描述符 Desciptor
4. 方法名称和描述符
5. 方法句柄和方法类型 Method Handle, Mehtod Type, Invoke Dynamic
6. 动态调用点和动态常量 Dynamically-Computed Call Site, Dynamically-Computed Constant

Class 文件中没有类似 C 语言中的链接，只有当 Class 文件在虚拟机中加载后才能确定内存分布。

常量池中每一项都是一个表，到 JDK13 为止有 17 种表结构

| Type                             | Flag | Desc                           |
| :------------------------------- | :--- | :----------------------------- |
| CONSTANT_Utf8_info               | 1    | UTF-8 编码的字符串             |
| CONSTANT_Integer_info            | 3    | 整型字面量                     |
| CONSTANT_Float_info              | 4    | 浮点型字面量                   |
| CONSTANT_Long_info               | 5    | 长整型型字面量                 |
| CONSTANT_Class_info              | 7    | 类或接口的符号引用             |
| CONSTANT_String_info             | 8    | 字符串类型字面量               |
| CONSTANT_Fieldref_info           | 9    | 字段的符号引用                 |
| CONSTANT_Methodref_info          | 10   | 类中方法的符号引用             |
| CONSTANT_InterfaceMethodref_info | 11   | 接口中方法的符号引用           |
| CONSTANT_NameAndType_info        | 12   | 字段或方法的部分符号引用       |
| CONSTANT_MethodHandle_info       | 15   | 表示方法句柄                   |
| CONSTANT_MethodType_info         | 16   | 表示方法类型                   |
| CONSTANT_Dynamic_info            | 17   | 表示一个动态计算常量           |
| CONSTANT_InvokeDynamic_info      | 18   | 表示一个动态方法调用点         |
| CONSTANT_Module_info             | 19   | 表示一个模块                   |
| CONSTANT_Package_info            | 20   | 表示一个模块中开放或者导出的包 |

常量池第一项以 `0a - 10` 开头，查看上表得知为 CONSTANT_Methodref_info 类型的表，查询可知对应的表结构为

| Const Name              | Item  | Length | desc                                                |
| :---------------------- | :---- | :----- | :-------------------------------------------------- |
| CONSTANT_Methodref_info | tag   | u1     | 值为 10                                             |
| -                       | index | u2     | 指向声明方法的类描述符 CONSTANT_Class_info 的索引项 |
| -                       | index | u2     | 指向名称及类型描述符 CONSTANT_NameAndType 的索引项  |

所以第一个常量值总共 5 个字节组成 `0a 00 04 00 12`，表示的是方法引用，类引用地址为 4，方法名称和类型地址为 18。

为了反向验证这样分析是否正确可以通过反编译命令 `javap -verbose TestClass` 查看 class 文件。

第一个常量值内容为 `#1 = Methodref          #4.#18         // java/lang/Object."<init>":()V` 和分析结果一致

```txt
C:\Users\jack\Downloads\helloworld\understanding-the-jvm\c6-file-structure\target\classes\c631>javap -verbose TestClass
警告: 文件 .\TestClass.class 不包含类 TestClass
Classfile /C:/Users/jack/Downloads/helloworld/understanding-the-jvm/c6-file-structure/target/classes/c631/TestClass.class
  Last modified 2020年11月19日; size 363 bytes
  MD5 checksum 16826804824a30e99e96960a47c3a47a
  Compiled from "TestClass.java"
public class c631.TestClass
  minor version: 0
  major version: 50
  flags: (0x0021) ACC_PUBLIC, ACC_SUPER
  this_class: #3                          // c631/TestClass
  super_class: #4                         // java/lang/Object
  interfaces: 0, fields: 1, methods: 2, attributes: 1
Constant pool:
   #1 = Methodref          #4.#18         // java/lang/Object."<init>":()V
   #2 = Fieldref           #3.#19         // c631/TestClass.m:I
   #3 = Class              #20            // c631/TestClass
   #4 = Class              #21            // java/lang/Object
   #5 = Utf8               m
   #6 = Utf8               I
   #7 = Utf8               <init>
   #8 = Utf8               ()V
   #9 = Utf8               Code
  #10 = Utf8               LineNumberTable
  #11 = Utf8               LocalVariableTable
  #12 = Utf8               this
  #13 = Utf8               Lc631/TestClass;
  #14 = Utf8               inc
  #15 = Utf8               ()I
  #16 = Utf8               SourceFile
  #17 = Utf8               TestClass.java
  #18 = NameAndType        #7:#8          // "<init>":()V
  #19 = NameAndType        #5:#6          // m:I
  #20 = Utf8               c631/TestClass
  #21 = Utf8               java/lang/Object
{
  public c631.TestClass();
    descriptor: ()V
    flags: (0x0001) ACC_PUBLIC
    Code:
      stack=1, locals=1, args_size=1
         0: aload_0
         1: invokespecial #1                  // Method java/lang/Object."<init>":()V
         4: return
      LineNumberTable:
        line 3: 0
      LocalVariableTable:
        Start  Length  Slot  Name   Signature
            0       5     0  this   Lc631/TestClass;

  public int inc();
    descriptor: ()I
    flags: (0x0001) ACC_PUBLIC
    Code:
      stack=2, locals=1, args_size=1
         0: aload_0
         1: getfield      #2                  // Field m:I
         4: iconst_1
         5: iadd
         6: ireturn
      LineNumberTable:
        line 7: 0
      LocalVariableTable:
        Start  Length  Slot  Name   Signature
            0       7     0  this   Lc631/TestClass;
}
SourceFile: "TestClass.java"
```

第二个常量为 09 开头，查表可知为 field 的引用

| Const Name             | Item  | Length | desc                                                        |
| :--------------------- | :---- | :----- | :---------------------------------------------------------- |
| CONSTANT_Fieldref_info | tag   | u1     | 值为 9                                                      |
| -                      | index | u2     | 指向声明字段的类或接口类描述符 CONSTANT_Class_info 的索引项 |
| -                      | index | u2     | 指向字段描述符 CONSTANT_NameAndType 的索引项                |

值为 `09 00 03 00 13` 对应 `#2 = Fieldref           #3.#19         // c631/TestClass.m:I`

第三个常量为 07 开头，为 Class 常量表

| Const Name          | Item  | Length | desc                     |
| :------------------ | :---- | :----- | :----------------------- |
| CONSTANT_Class_info | tag   | u1     | 值为 7                   |
| -                   | index | u2     | 指向全限定名常量的索引项 |

`07 00 14` 对应 `#3 = Class              #20            // c631/TestClass`

第四个也是 07 开头

`07 00 15` - `#4 = Class              #21            // java/lang/Object`

第五个为 01 开头, 表示 Utf8 类型的常量

| Const Name         | Item  | Type | desc                              |
| :----------------- | :---- | :--- | :-------------------------------- |
| CONSTANT_Utf8_info | tag   | u1   | 值为 1                            |
| -                  | index | u2   | UTF-8 编码的字符串占用的字节数    |
| -                  | bytes | u1   | 长度为 length 的 UTF-8 编码字符串 |

`01 00 01 6d`, 占用字节数 1，内容为 6d 的 UTF 内容 `m`，对应关系可以通过各种在线工具查看，很常用 `#5 = Utf8               m`

第六个常量 `01 00 01 49` - `#6 = Utf8               I`

第七个常量 `01 00 06 3c 69 6e 69 74 3e` 占用字节数 6 个 - `#7 = Utf8               <init>`

第八个 `01 00 03 28 29 56` - `#8 = Utf8               ()V`

第九个 `01 00 04 43 6f 64 65` - `#9 = Utf8               Code` 

第十个 `01 00 0f 4c 69 6e 65 4e 75 6d 62 65 72 54 61 62 6c 65` - `#10 = Utf8               LineNumberTable`

第十一个 `01 00 12 4c 6f 63 61 6c 56 61 72 69 61 62 6c 65 54 61 62 6c 65` - `#11 = Utf8               LocalVariableTable`

第十二个 `01 00 04 74 68 69 73` - `#12 = Utf8               this`

第十三个 `01 00 10 4c 63 36 33 31 2f 54 65 73 74 43 6c 61 73 73 3b` - `#13 = Utf8               Lc631/TestClass;`

第十四个 `01 00 03 69 6e 63` - `#14 = Utf8               inc`

第十五个 `01 00 03 28 29 49` - ` #15 = Utf8               ()I`

第十六个 `01 00 0a 53 6f 75 72 63 65 46 69 6c 65` - `#16 = Utf8               SourceFile`

第十七个 `01 00 0e 54 65 73 74 43 6c 61 73 73 2e 6a 61 76 61` - `#17 = Utf8               TestClass.java`

第十八个 `0c` 开头，为 NameAndType 类型

| Const Name                | Item  | Length | desc                                     |
| :------------------------ | :---- | :----- | :--------------------------------------- |
| CONSTANT_NameAndType_info | tag   | u1     | 值为 12                                  |
| -                         | index | u2     | 指向该字段或方法**名称**常量项的索引项   |
| -                         | index | u2     | 指向该字段或方法**描述符**常量项的索引项 |

`0c 00 07 00 08` - `#18 = NameAndType        #7:#8          // "<init>":()V`

第十九 `0c 00 05 00 06` - `#19 = NameAndType        #5:#6          // m:I`

第二十 `01 00 0e 63 36 33 31 2f 54 65 73 74 43 6c 61 73 73` - `#20 = Utf8               c631/TestClass`

第二十一 `01 00 10 6a 61 76 61 2f 6c 61 6e 67 2f 4f 62 6a 65 63 74` - `#21 = Utf8               java/lang/Object`

到此为止，常量池分析完毕

### 6.3.3 访问标志