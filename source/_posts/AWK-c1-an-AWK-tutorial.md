---
title: 第一章 an AWK tutorial
date: 2021-06-16 19:08:44
categories:
- AWK
tags:
- shell
- awk
---


## Getting Started

通过例子快速入门, 在如下结构化的数据中

* 计算工作时长 > 0 的总薪资
* 显示时常 = 0 的用户

```sh
cat emp.data
#name, pay rate per hour, work time(hour)
Beth   4.00 0
Dan    3.75 0
Kathy  4.00 10
Mark   5.00 20
Mary   5.50 22
Susie  4.25 18

awk '$3 == 0 {print $1}' emp.data
Beth
Dan

awk '$3 > 0 {print $1, $2 * $3}' emp.data 
Kathy 40
Mark 100
Mary 121
Susie 76.5
```

### The Structure of an AWK Program

格式 `pattern { action }`, pattern 如果结果为 true 则执行 action 中定义的行为。然后执行下一行，一直到文本结束。

### Running an AWK Program

`awk 'program' input files`, 多文件 `awk 'program' input file1 file2`, 不接文件，则从标准输入拿内容

可以将程序部分写到文件中 `awk -f progfile optional_list_of_input_files`

## Simple Output

**Print Event Line** 没有写 pattern 表示每行都输出，`{ print }` 和 `{ print $0 }` 等价，都是打印全部

**Print Certain Fields** `{ print $1, $3 }`

**NF, the Number of Fields** 内置变量 NF 表示最后一个 field 的 number, 下面的例子答应 field 数量 + 名字 + 最后一个 field

```sh
awk '{print NF, $1, $NF}' emp.data
3 Beth 0
3 Dan 0
...
```

**Computing and Printing** field 可以直接用于计算 `{ print $1, $2 * $3 }`

**Printing Line Numbers** NR 代表行号, number of row(record?)

```sh
awk '{print NR, $0}' emp.data
1 Beth 4.00 0
2 Dan 3.75 0
...
```

**Putting Text in the Output** 输出内容中加入自己的文本

```sh
awk '{print "total pay for", $1, "is", $2 * $3}' emp.data
total pay for Beth is 0
total pay for Dan is 0
total pay for Kathy is 40
total pay for Mark is 100
total pay for Mary is 121
total pay for Susie is 76.5
```

## Fancier Output

print 只是简单的打印内容，如果想要输出更丰富，使用 printf

**Lining Up Fields** printf 格式 `printf(format, value1, value2..., valuen)`

```sh
awk '{printf("total pay for %s is %.2f\n", $1, $2 * $3)}' emp.data
total pay for Beth is 0.00
total pay for Dan is 0.00
total pay for Kathy is 40.00
total pay for Mark is 100.00
total pay for Mary is 121.00
total pay for Susie is 76.50

awk '{printf("%-8s $%6.2f\n", $1, $2 * $3)}' emp.data
Beth     $  0.00
Dan      $  0.00
Kathy    $ 40.00
Mark     $100.00
Mary     $121.00
Susie    $ 76.50
```

**Sorting the Output** 结合 pipe 进行排序

```sh
awk '{printf("%6.2f %s\n", $2 * $3, $0)}' emp.data | sort
  0.00 Beth 4.00 0
  0.00 Dan 3.75 0
 40.00 Kathy 4.00 10
 76.50 Susie 4.25 18
100.00 Mark 5.00 20
121.00 Mary 5.50 22
```

## Selection

Awk 的 pattern 可用于筛选数据

**Selection by Comparison** 通过比较筛选

```sh
# 时薪大于5
awk '$2 >= 5' emp.data
Mark 5.00 20
Mary 5.50 22
```

**Selection by Computation** 结合计算筛选

```sh
awk '$2 * $3 >= 50 { printf("$%.2f for %s\n", $2 * $3, $1)}' emp.data
$100.00 for Mark
$121.00 for Mary
$76.50 for Susie
```

**Selection by Text Content** 文本匹配选择, == 精确匹配，/match/ 包含

```sh
awk '$1 == "Susie"' emp.data
Susie  4.25 18

awk '/Susie/' emp.data
Susie  4.25 18
```

**Combinations of Patterns** 使用 ||, && 做逻辑操作

```sh
awk '$2 >= 4 || $3 >= 20' emp.data
Beth   4.00 0
Kathy  4.00 10
Mark   5.00 20
Mary   5.50 22
Susie  4.25 18
```

如果不加逻辑操作符号，则符合条件的语句会重复输出

'$2 >= 4 || $3 >= 20' 等价于 !($2 < 4 && $3 < 20)

**Data Validation** Awk 是一款很优秀的数据校验工具

```sh
# 打印 field 不等于 3 的行
awk 'NF != 3 {print $0, "number of fields is not equal to 3"}' emp.data
# 打印时薪小于 3.35 的行
awk '$2 < 3.35 { print $0, "rate is below minimum wage" }' emp.data
```

**BEGIN and END** 类似 before/after class 的操作

```sh
awk 'BEGIN { print "NAME   RATE    HOURS"; print ""} { print }' emp.data 
NAME   RATE    HOURS

Beth   4.00 0
Dan    3.75 0
Kathy  4.00 10
Mark   5.00 20
Mary   5.50 22
Susie  4.25 18
```

## Computing with AWK

`pattern { action }` 中的 action 是一系列用换行或冒号分割的语句。这节介绍一些字符，数字操作，一些内置变量和自定义变量。自定义变量不需要声明。

**Counting** 统计时长大于 15 的用户

```sh
awk ' $3 > 15 { emp++; } END{ print emp, "employees worked more than 15 hours" }' emp.data 
3 employees worked more than 15 hours
```

**Computing Sums and Averages** 计算总值和平均值

```sh
awk 'END { print NR, "employees" }' emp.data                                              
6 employees

awk '{ pay = pay + $2 * $3 }
END {
print NR, "employees"
print "total pay is", pay
print "average pay is", pay/NR
}' emp.data
6 employees
total pay is 337.5
average pay is 56.25
```

**Handling Text** Awk 有处理文字的能力， awk 中的变量可以持有数字和字符串，下面的例子显示报酬最多的用户

```sh
awk '                       
$2 > maxrate { maxrate = $2; maxemp = $1 }
END { print "highest hourly rate:", maxrate, "for", maxemp }
' emp.data 
highest hourly rate: 5.50 for Mary
```

可以看出来，默认的变量初始值为 0

**String Concatenation** 字符拼接

```sh
awk '{ names = names $1 "  " }
END { print names }' emp.data 
Beth  Dan  Kathy  Mark  Mary  Susie 
```

names 变量的初始值为 null

**Printing the Last Input Line** 打印最后一行

```sh
awk '{ last = $0 } END { print last }' emp.data 
Susie  4.25 18
```

**Built-in Functions** awk 提供了很多内置函数，比如 length 计算字符串长度

```sh
awk '{ print $1, length($1) }' emp.data 
Beth 4
Dan 3
Kathy 5
Mark 4
Mary 4
Susie 5
```

**Counting Lines, Words and Characters** 通过使用 length，NF, NR 统计行基本信息，为了便于计算，我们将每一个 field 都当作 String 对待

```sh
awk '{                                 
    nc = nc + length($0) + 1
    nw = nw + NF     
}
END { print NR, "lines,", nw, "words,", nc, "characters"}' emp.data 
6 lines, 18 words, 88 characters
```

`nc = nc + length($0) + 1` 1 代表换行符

## Control-Flow Statemnets

awk 中的流程控制和 C 语言中基本一直，这些控制语句只能用在 action 中

### If-Else Statement

统计时薪大于 6 的所有人的总收入及平均收入, 通过 if-else 控制打印的 loop

```sh
awk '    
$2 > 6 { n = n+1; pay = pay+$2*$3 }
END {
  if (n > 0)
    print n, "employees, total pay is", pay, "average pay is", pay/n
  else     
    print "no employees are paid more than $6/hour"
}' emp.data 
no employees are paid more than $6/hour
```

### While Statement

while = condition + body. 下面实现一个计算存款的功能，表达式可以概括为 value = amount (1 + rate)<sup>years</sup>

```sh
cat interest1 
# interest1 - compute compound interest
#   input: amount rate years
#   output: compounded value at the end of each year

{
    i = 1
    while (i <= $3) {
        printf("\t%.2f\n", $1 * (1 + $2) ^ i)
        i = i + 1
    }
}

awk -f interest1
1000 .06 5
        1060.00
        1123.60
        1191.02
        1262.48
        1338.23
```

### For Statement

同样的计算，用 for 实现

```sh
cat interest2  
# interest2 - compute compound interest
#   input: amount rate years
#   output: compounded value at the end of each year

{
    for (i=1; i<=$3; i++)
        printf("\t%.2f\n", $1 * (1+$2) ^ i)
}

awk -f interest2
1000 .06 5
        1060.00
        1123.60
        1191.02
        1262.48
        1338.23
```

## Arrays

awk 支持数组。下面的实验中，我们在 action 中将行信息存到数组中，在 END 中通过 while 倒序输出

```sh
awk '           
{ line[NR] = $0 }
END {
  i = NR  
  while (i>0) {                                
    print line[i]                            
    i = i-1 
  }
}' emp.data 
Susie  4.25 18
Mary   5.50 22
Mark   5.00 20
Kathy  4.00 10
Dan    3.75 0
Beth   4.00 0
```

## A Handful of Useful "One-liners"

摘录一些简短但是令人印象深刻的 awk 脚本

* print the total number of input lines `awk 'END { print NR }' emp.data`
* 打印第三行 `awk 'NR == 3' emp.data `
* 打印每行最后一个 field `awk '{ print $NF }' emp.data`
* 打印最后一行的最后一个 field `awk '{ field = $NF } END { print field }' emp.data`
* 打印 field 数量大于 4 的行 `awk 'NF > 4' emp.data`
* 打印最后一个 field 大于 4 的行 `awk '$NF > 4' emp.data`
* 用行号代替第一个 field `awk '{ $1 = NR; print }' emp.data`
* 抹去第二个 field `awk '{ $2=""; print }' emp.data`
* 倒序打印每一行 `awk '{for(i=NF; i>0;i--) printf("%s", $i); printf("\n")}' emp.data`