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

紧跟在常量池之后，由两个字节组成，有 16 个标志位，当前只定义了 9 种。

| Name           | flag value | 含义                                                                                                                                                                            |
| :------------- | :--------- | :------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| ACC_PUBLIC     | 0x0001     | 是否为 public 类型                                                                                                                                                              |
| ACC_FINAL      | 0x0010     | 是否为 final 类型, 只有类可设置                                                                                                                                                 |
| ACC_SUPER      | 0x0020     | 是否允许使用 invokespecial 字节码指定的新语义，<BR>invokespecial 语义在 JDK 1.0.2 发生过改变，<br>为了区别这条指令使用哪种语义， <BR>JDK 1.0.2 之后编译出来的类这个标志必须为真 |
| ACC_INTERFACE  | 0x0200     | 是否是一个接口                                                                                                                                                                  |
| ACC_ABSTRACT   | 0x0400     | 是否为 abstract 类型，对于接口或者抽象类来说，此标志必须为真，其他类型为假                                                                                                      |
| ACC_SYNTHETIC  | 0x1000     | 表示这个类并非由用户代码产生                                                                                                                                                    |
| ACC_ANNOTATION | 0x2000     | 标识这是一个注解                                                                                                                                                                |
| ACC_ENUM       | 0x4000     | 标识这是一个枚举                                                                                                                                                                |
| ACC_MODULE     | 0x8000     | 标识这是一个模块                                                                                                                                                                |

示例种值为 `00 21` 即 0020 & 0001 所以是 public + super 类型

### 6.3.4 类索引，父索引和接口索引集合

* 类索引（this_class） - u2 类型数据
* 父索引（super_class） - u2 类型数据
* 接口索引集合（super_class） - u2 类型数据

这些所以确定类的继承关系，实例中数据 `00 03 00 04 00 00` 表示 类所以指向常量池第三个常量，父索引指向第四个常量，接口集合数量为 0 

`#3 = Class              #20            // c631/TestClass `

`#4 = Class              #21            // java/lang/Object`

### 6.3.5 字段表集合

用来描述接口或类中声明的变量。这里的变量只包括**类级**变量以及**实例级**变量，不包含局部变量。

字段表结构

| 类型           | 名称             | 数量            |
| :------------- | :--------------- | :-------------- |
| u2             | access_flags     | 1               |
| u2             | name_index       | 1               |
| u2             | descriptor_index | 1               |
| u2             | attribute_count  | 1               |
| attribute_info | attributes       | attribute_count |

字段修饰符 access_flags 和类的访问修饰符很想都由一个 u2 的数据类型表示

| 名称          | 标志值 | 含义                 |
| :------------ | :----- | :------------------- |
| ACC_PUBLIC    | 0x0001 | 字段是否 public      |
| ACC_PRIVATE   | 0x0002 | 字段是否 private     |
| ACC_PROTECTED | 0x0004 | 字段是否 protected   |
| ACC_STATIC    | 0x0008 | 字段是否 static      |
| ACC_FINAL     | 0x0010 | 字段是否 final       |
| ACC_VOLATILE  | 0x0040 | 字段是否 volatile    |
| ACC_TRANSIENT | 0x0080 | 字段是否 transient   |
| ACC_SYNTHTIC  | 0x0100 | 字段是否由编译器产生 |
| ACC_ENUM      | 0x0400 | 字段是否 enum        |

* 作用域修饰符： public/private/protected
* 是否是类级字段：static
* 是否可变：final
* 是否强制主从内存读写：volatile
* 是否可序列化：transient

name_index 和 descriptor_index 都指向常量池引用，表示字段简单名称以及字段和方法描述符。

* 全名限定：用斜线分割的 路径+类名
* 简单名称：只有名字，没有路径信息
* 方法和字段描述符：参数列表+返回值类型，例如 ()V, (Lcom/lang/Object;)V

基本数据类型含义表

| 字符 | 含义     |
| :--- | :------- |
| B    | byte     |
| C    | char     |
| D    | double   |
| F    | float    |
| I    | int      |
| J    | long     |
| S    | short    |
| Z    | boolean  |
| V    | void     |
| L    | 对象类型 |

表示数组类型时，每一维度将使用一个前置的 `[` 字符描述，比如 String[][] 表示为 `[[Ljava/lang/String;`, 整形数组 int[] 表示为 `[I`。

实例中对应的字段表集合内容为 `00 01 00 02 00 05 00 06 00 00`， interface 之后紧接着为 fields_count 的表示位， `00 01`， 表示只有一个 field。

`00 02` 表示方位权限 private，`00 05` 表示名字指向常量池第五个常量 `m`, `00 06` 表示描述符指向第六个常量 `I`，`00 00` 属性表个数位 0 个。

### 6.3.6 方法表集合

方法表和之前的属性表，class 表是一个套路的, 方法表结构如下

| 类型           | 名称             | 数量            |
| :------------- | :--------------- | :-------------- |
| u2             | access_flags     | 1               |
| u2             | name_index       | 1               |
| u2             | descriptor_index | 1               |
| u2             | attribute_count  | 1               |
| attribute_info | attributes       | attribute_count |

方法表的 access_flag 相对 field 少了 volatile 和 trasient, 多了 synchronized, native, strictfp 和 abstract

| 名称             | 标志值 | 含义                             |
| :--------------- | :----- | :------------------------------- |
| ACC_PUBLIC       | 0x0001 | 方法是否 public                  |
| ACC_PRIVATE      | 0x0002 | 方法是否 private                 |
| ACC_PROTECTED    | 0x0004 | 方法是否 protected               |
| ACC_STATIC       | 0x0008 | 方法是否 static                  |
| ACC_FINAL        | 0x0010 | 方法是否 final                   |
| ACC_SYNCHRONIZED | 0x0020 | 方法是否 synchronized            |
| ACC_BRIDGE       | 0x0040 | 方法是否是由编译器产生的桥接方法 |
| ACC_VARARGS      | 0x0080 | 方法是否接收不定长参数           |
| ACC_NATIVE       | 0x0100 | 方法是否为 native                |
| ACC_ABSTRACT     | 0x0400 | 字段是否 abstract                |
| ACC_STRICT       | 0x0800 | 字段是否 strictfp                |
| ACC_SYNTHETIC    | 0x1000 | 字段是否由编译器自动产生         |

方法中的具体实现经过 javac 编译成字节码指令后存在属性表集合中一个名为 Code 的属性里面。

实例内容 `00 02 00 01 00 07 00 08 00 01 00 09`

* 00 02 - 有两个方法
* 00 01 - public 类型的方法
* 00 07 - name 指向常量池7 - <init>
* 00 08 - 描述符指向8 - ()V
* 00 01 - 属性数量 1
* 00 09 - 属性表索引 9，指向 Code

方法签名：Java 语法中的方法签名可以从重载(Overload)理解。Java 中重载要求方法名一致，参数列表及参数类型不同。返回值并不在比较范围内。方法除了返回值不同的重载是会编译错误的。但是在字节码的语义中，只有返回值不同的重载是合法的。

### 6.3.7 属性表集合

属性表集合的限制比前面那些结构要宽松一些，对虚拟机不认识的属性，会自动跳过。到 java 12 一共有 29 种预定义的属性

| 属性名称                             | 使用位置                     | 含义                                                                                    |
| :----------------------------------- | :--------------------------- | :-------------------------------------------------------------------------------------- |
| Code                                 | 方法表                       | Java代码编译成的自己吗指令                                                              |
| ConstantValue                        | 字段表                       | 由 final 关键字定义的常量值                                                             |
| Deprecated                           | 类，方法，字段表             | 被声明为 deprecated 的方法和字段                                                        |
| Exceptions                           | 方法表                       | 方法抛出的异常列表                                                                      |
| EnclosingMethod                      | 类文件                       | 仅当一个类为局部类或匿名类是才拥有这个属性，用于标识这个类所在的外围方法                |
| InnerClasses                         | 类文件                       | 内部类列表                                                                              |
| LineNumberTable                      | Code属性                     | Java 源码的行号与字节码指令的对应关系                                                   |
| LocalVariableTable                   | Code属性                     | 方法的局部变量描述                                                                      |
| StackMapTable                        | Code属性                     | JDK6 新增，供新的类型检查验证器检查和处理目标方法的局部变量和操作数栈所需的类型是否匹配 |
| Signature                            | 类，方法表和字段表           | JDK5新增，用于支持泛型情况下的方法签名                                                  |
| SourceFile                           | 类文件                       | 记录源文件名称                                                                          |
| SourceDebugExtension                 | 类文件                       | JDK5新增，存储额外的调试信息                                                            |
| Synthetic                            | 类，方法表，字段表           | 标识是否由编译器产生                                                                    |
| LocalVariableTypeTable               | 类                           | JDK5新增，使用特征签名代替描述符，为了支持泛型                                          |
| RuntimeVisibleAnnotations            | 类，方法表，字段表           | JDK5新增，为动态注解提供支持                                                            |
| RuntimeInVisibleAnnotations          | 类，方法表，字段表           | JDK5新增，为动态注解提供支持,标识不可见                                                 |
| RuntimeVisibleParameterAnnotations   | 方法表                       | JDK5新增，作用对象为方法参数                                                            |
| RuntimeInvisibleParameterAnnotations | 方法表                       | JDK5新增，作用对象为方法参数                                                            |
| AnnotationDefault                    | 方法表                       | JDK5新增，注解类元素默认值                                                              |
| BootstrapMethods                     | 类文件                       | JDK7新增，保存 invokedynamic 指令引用的引导犯法限定符                                   |
| RuntimeVisibleTypeAnnotations        | 类，方法表，字段表, Code属性 | JDK8新增                                                                                |
| RuntimeInvisibleTypeAnnotations      | 类，方法表，字段表, Code属性 | JDK8新增                                                                                |
| MethodParameters                     | 方法表                       | JDK8新增                                                                                |
| Module                               | 类                           | JDK9新增                                                                                |
| ModulePackages                       | 类                           | JDK9新增                                                                                |
| ModuleMainClass                      | 类                           | JDK9新增                                                                                |
| NestHost                             | 类                           | JDK11新增                                                                               |
| NestMembers                          | 类                           | JDK11新增                                                                               |

属性表结构

| 类型 | 名称                 | 数量             |
| :--- | :------------------- | :--------------- |
| u2   | attribute_name_index | 1                |
| u4   | attribute_length     | 1                |
| u1   | info                 | attribute_length |

attribute_name_index 指向常量池中的一个引用，属性值结构完全自定义，attribute_length 说明属性值所占的位数。

#### Code

Java 方法体种的代码经过 javac 编译器处理之后都会转化为字节码指令存储在 Code 属性内。Code 属性出现在方法表的属性集合中，但并非所有方法表都必须存在这个属性，比如抽象方法或接口中就可以不存在 Code 属性。

Code 属性表的结构

| 类型           | 名称                   | 数量                   | 含义                                              |
| :------------- | :--------------------- | :--------------------- | :------------------------------------------------ |
| u2             | attribute_name_index   | 1                      | 指向 CONSTANT_Utf8_info 常量的索引，为固定值 Code |
| u4             | attribute_length       | 1                      | 属性值长度                                        |
| u2             | max_stack              | 1                      | 操作数栈深度的最大值                              |
| u2             | max_locals             | 1                      | 局部变量表存储空间，单位-变量槽(Slot)             |
| u4             | code_length            | 1                      | 编译后字节码指令个数                              |
| u1             | code                   | code_length            | 编译后字节码指令                                  |
| u2             | exception_table_length | 1                      | -                                                 |
| exception_info | exception_table        | exception_table_length | -                                                 |
| u2             | attribute_count        | 1                      | -                                                 |
| attribute_info | attributes             | attribute_count        | -                                                 |

对于 byte, char, float, int, short, boolean 和 returnAddress 等长度不超过 32 byte 的数据类型，每个局部变量占用一个变量槽，double, long 这两个 64 位的占两个槽。

同时生存的最大局部变量和类型计算出 max_locals

字节码指令长度 u1。u1 可以最多表达 255 个指令，现在大约已经定义了 200 条。

测试案例中 init 方法对应的 code 代码块为 `00 09 00 00 00 2f 00 01 00 01 00 00 00 05 2a b7 00 01 b1 00 00 00 02`

`00 09` 前面已经说过，指向固定的 Code 字符地址

`00 00 00 31` 属性表长度 3*16 + 1 = 49

`00 01` 栈深 1

`00 01` 本地变量表大小 1

`00 00 00 05` code 长度 5

`2a b7 00 01 b1` code 内容

* `2a`: aload_0 将第一个变量推送至栈顶
* `b7` invokespecial, 后面接一个 u2 类型引用数据，执行构造方法或 private 方法，或它的父类方法
* `00 01` 方法引用，指向 init
* `b1` return 指令

对应的 javap 代码

```
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
```

`args_size=1` 方法虽然没有参数，但是 Java 编译时会把 this 作为第一个默认参数塞入 code 代码块中。

`00 00 00 02` 异常表长度 0， 属性表长度 2

异常表结构

| 类型 | 名称       | 数量 |
| :--- | :--------- | :--- |
| u2   | start_pc   | 1    |
| u2   | end_pc     | 1    |
| u2   | handler_pc | 1    |
| u2   | catch_type | 1    |

异常代码案例

```java
public int inc() {
      int x;
      try {
          x = 1;
          return x;
      } catch (Exception e) {
          x = 2;
          return x;
      } finally {
          x = 3;
      }
  }
```

对应的 javap 代码

```txt
public int inc();
    descriptor: ()I
    flags: (0x0001) ACC_PUBLIC
    Code:
      stack=1, locals=5, args_size=1
         0: iconst_1
         1: istore_1
         2: iload_1
         3: istore_2
         4: iconst_3
         5: istore_1
         6: iload_2
         7: ireturn
         8: astore_2
         9: iconst_2
        10: istore_1
        11: iload_1
        12: istore_3
        13: iconst_3
        14: istore_1
        15: iload_3
        16: ireturn
        17: astore        4
        19: iconst_3
        20: istore_1
        21: aload         4
        23: athrow
      Exception table:
         from    to  target type
             0     4     8   Class java/lang/Exception
             0     4    17   any
             8    13    17   any
            17    19    17   any
```

和书上的结果略有差别，但基本一致

#### Exceptions 属性

和 Code 平级的概念，并不是上一章节里 Code 下面的 exception 表。这里表示的是方法可能抛出的异常，就是 throws 后面的那些东西。属性结构如下:

| type | name                  | count                |
| :--- | :-------------------- | :------------------- |
| u2   | attribute_name_index  | 1                    |
| u4   | attribute_length      | 1                    |
| u2   | number_of_exceptions  | 1                    |
| u2   | exception_index_table | number_of_exceptions |

number_of_exceptions: 可能抛出的受检测的异常类型
exception_index_table: 指向常量池中的 CONSTANT_Class_info 索引

#### LineNumberTable 属性

描述 Java 源码行号和字节码行号之间的对应关系。可以在编译时指定不生成行号，但是会影响异常信息显示和 debug, 表结构如下:

| type             | name                     | count                    |
| :--------------- | :----------------------- | :----------------------- |
| u2               | attribute_name_index     | 1                        |
| u4               | attribute_length         | 1                        |
| u2               | line_number_table_length | 1                        |
| line_number_info | line_number_table        | line_number_table_length |

line_number_info: 包含 start_pc 和 line_number 两个 u2 类型的数据项，前者是字节码行号，后者是 Java 源码行号。

#### LocalVarableTable 及 LocalVarableTypeTable 属性

LocalVarableTable 描述局部变量表的变量与 Java 源码中定义的变量之间的关系。非必须，可以指定 javac 参数去除且不影响运行。但是去除后方法参数名称会变为类似 arg0, arg1 的表示，不方便，表结构如下：

| type                | name                        | count                       |
| :------------------ | :-------------------------- | :-------------------------- |
| u2                  | attribute_name_index        | 1                           |
| u4                  | attribute_length            | 1                           |
| u2                  | local_variable_table_length | 1                           |
| local_variable_info | local_variable_table        | local_variable_table_length |

local_variable_info 代表栈帧与源码中局部变量的关联，结构如下：

| type | name             | count |
| :--- | :--------------- | :---- |
| u2   | start_pc         | 1     |
| u2   | length           | 1     |
| u2   | name_index       | 1     |
| u2   | descriptor_index | 1     |
| u2   | index            | 1     |

* start_pc + length: 限定了局部变量的作用范围，即作用域
* name_index + descriptor_index: 指向常量池中 CONSTANT_Utf8_info 类型索引
* index: 栈帧局部变量槽位置，当数据类型为 64 位则占用 index 和 index+1 两个

LocalVarableTypeTable 是 JDK5 时为了支持范型而引入的，基本功能和 LocalVarableTable 一样。

#### SourceFile 及 SourceDebugExtension 属性

SourceFile 记录生成 Class 文件的源码文件名称，可选，通常与类名同，特殊情况除外(如内部类)。表结构如下:

| type | name                 | count |
| :--- | :------------------- | :---- |
| u2   | attribute_name_index | 1     |
| u4   | attribute_length     | 1     |
| u2   | sourcefile_index     | 1     |

sourcefile_index: 指向常量池中 CONSTANT_Utf8_info 型常量的索引，值问文件名。

SourceDebugExtension 是 JDK5 中加入的新特性，存储额外调试信息，支持类似 JSP 这种使用 Java 编译器但是语法不同的语言，类中最多只允许一个该属性。表结构如下：

| type | name                              | count |
| :--- | :-------------------------------- | :---- |
| u2   | attribute_name_index              | 1     |
| u4   | attribute_length                  | 1     |
| u2   | debug_extension[attribute_length] | 1     |

#### ConstantValue 属性

ConstantValue 通知虚拟机自动为静态变量赋值。只有被 static 修饰的变量才能使用这个属性。虚拟机中对非 static 变量在 <init>() 方法总进行，对于静态变量则有两种方式，一种是构造器 <clinit>() 另一种是 ConstantValue。Oracle 的 javac 中的实现方式为：static + final + 基本类型/String 在 ConstantValue 中赋值， 没有 final 或者是其他数据类型则在 <clinit>() 中赋值。表结构如下：

| type | name                 | count |
| :--- | :------------------- | :---- |
| u2   | attribute_name_index | 1     |
| u4   | attribute_length     | 1     |
| u2   | constantvalue_index  | 1     |

constantvalue_index: 指向常量池中一个引用，可选类型有 CONSTANT_Long_info, CONSTANT_Float_info, CONSTANT_Double_info, CONSTANT_Integer_info 和 CONSTANT_String_info。

#### InnerClasses 属性

InnerClasses 记录内部类与宿主类之间的关联。结构如下：

| type               | name                 | count             |
| :----------------- | :------------------- | :---------------- |
| u2                 | attribute_name_index | 1                 |
| u4                 | attribute_length     | 1                 |
| u2                 | number_of_classes    | 1                 |
| inner_classes_info | inner_classes        | number_of_classes |

number_of_classes: 内部类个数

inner_classes_info 结构如下

| type | name                     | count |
| :--- | :----------------------- | :---- |
| u2   | inner_class_info_index   | 1     |
| u2   | outer_class_info_index   | 1     |
| u2   | inner_name_index         | 1     |
| u2   | inner_class_access_flags | 1     |

inner_class_info_index, outer_class_info_index：指向常量池中 CONSTANT_Class_info 常量索引，分别代表内部类和宿主类

inner_name_index：指向常量池 CONSTANT_Utf8_info 引用，代表内部类名称，如果是匿名内部类，值为 0

inner_class_access_flags：和 class 定义相似，类的访问标示符，取值范围如下

| 标志名称       | 标志值 | 含义                         |
| :------------- | :----- | :--------------------------- |
| ACC_PUBLIC     | 0x0001 | 内部类是否为 public          |
| ACC_PRIVATE    | 0x0002 | 内部类是否为 private         |
| ACC_PROTECTED  | 0x0004 | 内部类是否为 protected       |
| ACC_STATIC     | 0x0008 | 内部类是否为 static          |
| ACC_FINAL      | 0x0010 | 内部类是否为 final           |
| ACC_INTERFACE  | 0x0020 | 内部类是否为 接口            |
| ACC_ABSTRACT   | 0x0400 | 内部类是否为 abstract        |
| ACC_SYNTHETIC  | 0x1000 | 内部类是否并非由用户代码产生 |
| ACC_ANNOTATION | 0x2000 | 内部类是否为一个注解         |
| ACC_ENUM       | 0x4000 | 内部类是否为一个枚举         |

#### Deprecated 及 Synthetic 属性

都是标志符类型的布尔属性，只有存在有和没有的区别，没有属性概念。Deprecated 对应 @deprecated 注解，表示不推荐使用。

Synthetic 标示字段或方法由编译器产生，JDK5之后同样的功能可以通过设置 ACC_SYNTHETIC 标志位达到。通过这种方式甚至可以越权访问或绕开语言限制功能。典型例子是枚举类中自动生成枚举元素数组和嵌套类的桥接方法(Bridge Method)。

| type | name                 | count |
| :--- | :------------------- | :---- |
| u2   | attribute_name_index | 1     |
| u4   | attribute_length     | 1     |

attribute_length 必须为 0x00000000，因为诶呦任何属性需要设置。

#### StackMapTable 属性

JDK6 增加到 Class 文件规范，一个相当复杂的变长属性，位于 Code 属性表中，用来代替原来的类型检查验证器，提升性能。实现很复杂，Java SE7 新增 120 页篇幅讲解描述。

| type            | name                    | count             |
| :-------------- | :---------------------- | :---------------- |
| u2              | attribute_name_index    | 1                 |
| u4              | attribute_length        | 1                 |
| u2              | number_of_entries       | 1                 |
| stack_map_frame | stack_map_frame entries | number_of_entries |

SE7 之后规定，版本号 >= 50.0 的 class 文件都必须带有 StackMapTable 属性。一个 Code 属性最多只能有一个 StackMapTable 不然抛错 ClassFormatError。

