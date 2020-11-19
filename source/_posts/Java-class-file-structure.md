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

Class 文件时一组以 8 个字节为基础单位的二进制流，各个数据项目严格按照顺序紧凑排列在文件之中，中间没有添加任何分隔符，这使得整个 Class 文件中存储的内容几乎全部是程序运行的必要数据，没有空隙存在。

Class 文件结构中只有两种数据类型：无符号数 + 表

* 无符号数：是基本数据类，以 u1, u2, u4, u8 分别表示 1，2，4，8 个字节的无符号数。无符号数可以用来描述数字，索引引用，数量值或者按照 UTF-8 编码构成的字符串值
* 表：n 个无符号数 + 其他表构成的复合数据类型，命名习惯性的以 _info 结尾。表用于描述有层次关系的复合结构数据，整个 Class 可以看作一张表。

Class 文件格式表：

| Type           | Name                | Count                 | Position |
| :------------- | :------------------ | :-------------------- | :------- |
| u4             | magic               | 1                     | 0-3      |
| u2             | minor_version       | 1                     | 4-5      |
| u2             | major_version       | 1                     | 6-7      |
| u2             | constant_pool_count | 1                     | 8-9      |
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

Java 虚拟机规范在 Class 文件校验部分明确要求，即使文件格式并未发生任何变化，虚拟机也必须拒绝执行超过其版本号的 Class 文件。

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

### 6.3.2 常量池

第 9-8 个字节表示常量池。常量池是从 1 开始的。案例中对应的十六进制为 0x0016，转化为 10 进制为 22。表明有 21 个常量。

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
