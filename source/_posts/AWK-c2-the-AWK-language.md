---
title: 第二章 the AWK language
date: 2021-06-17 19:45:18
categories:
- AWK
tags:
- shell
- awk
---

最简单的 awk 程序是由一系列 pattern-action 组成的

```sh
pattern { action }
pattern { action }
...
```

有时 pattern 会省略，有时 action 会省略。当 awk 检测程序段没有语法错误后，他会一句一句的执行。pattern 没有写即表示匹配每一行。

本章第一节会介绍 pattern， 后面会介绍表达式，赋值等，剩余部分则是介绍函数等信息。

这里的准备文件是有讲究的，直接用 vscode 准备会出问题，最好在终端使用 echo + \t 的方式手动打一遍

```sh
cat countries                  
USSR    8649    275     Asia
Canada  3852    25      North America
China   3705    1032    Asia
USA     3615    237     North America
Brazil  286     134     South America
India   1267    746     Asia
Mexico  762     78      North America
France  211     55      Europe
Japan   144     120     Asia
Germany 96      61      Europe
England 94      56      Europe

sed -n 'l' countries 
USSR\t8649\t275\tAsia$
Canada\t3852\t25\tNorth America$
China\t3705\t1032\tAsia$
USA\t3615\t237\tNorth America$
Brazil\t286\t134\tSouth America$
India\t1267\t746\tAsia$
Mexico\t762\t78\tNorth America$
France\t211\t55\tEurope$
Japan\t144\t120\tAsia$
Germany\t96\t61\tEurope$
England\t94\t56\tEurope$

bat -A countries 
   1   │ USSR├──┤8649├──┤275├──┤Asia␊
   2   │ Canada├──┤3852├──┤25├──┤North·America␊
   3   │ China├──┤3705├──┤1032├──┤Asia␊
   4   │ USA├──┤3615├──┤237├──┤North·America␊
   5   │ Brazil├──┤286├──┤134├──┤South·America␊
   6   │ India├──┤1267├──┤746├──┤Asia␊
   7   │ Mexico├──┤762├──┤78├──┤North·America␊
   8   │ France├──┤211├──┤55├──┤Europe␊
   9   │ Japan├──┤144├──┤120├──┤Asia␊
  10   │ Germany├──┤96├──┤61├──┤Europe␊
  11   │ England├──┤94├──┤56├──┤Europe␊
```

## Patterns

pattern 控制着 action 的执行，下面介绍六种 pattern 类型

* BEGIN { statements }
* END { statements }
* expression { statements }, 当 expression 为 true， statements 会被执行
* /regular expression/{ statements }
* compound pattern { statements }, pattern 通过 &&, ||, !, () 链接
* pattern1, pattern2 { statements }

### BEGIN and END

BEGIN 经常用来改变 field 的分隔符，默认的分隔符通过 FS 这个内置变量控制，默认的值有空格， / 和 tab. 下面的例子中，我们将分隔符改为 tab 并统计所有地区的面积和人口总数

```sh
awk '
BEGIN { 
    FS = "\t"
    printf("%10s %6s %5s  %s\n\n", "COUNTRY", "AREA", "POP", "CONTINENT")
}
{
    printf("%10s %6s %5d  %s\n", $1, $2, $3, $4)
    area = area + $2
    pop = pop + $3
}
END { printf("\n%10s %6d %5d\n", "TOTAL", area, pop) }' countries
   COUNTRY   AREA   POP  CONTINENT

      USSR   8649   275  Asia
    Canada   3852    25  North America
     China   3705  1032  Asia
       USA   3615   237  North America
    Brazil    286   134  South America
     India   1267   746  Asia
    Mexico    762    78  North America
    France    211    55  Europe
     Japan    144   120  Asia
   Germany     96    61  Europe
   England     94    56  Europe

     TOTAL  22681  2819
```

### Expressions as Patterns

Expressions 是指运算表达式，由 数字，字符串和符号组成。本书中的 string 指的是 0-n 的字符序列。空字串 "" 在 awk 中被称为 null string. 每个 string 中都包含有 null string.

如果操作符需要一个 string 类型的参数，但是你给了一个 numberic 的参数，则 awk 会将数字类型转为字符类型。同样的，如果操作符需要数字类型，给了字符类型，也会自动做转化。

常见的比较符号

| OPERATOR | MEANING                  |
| :------- | :----------------------- |
| <        | less than                |
| <=       | less than or equal to    |
| ==       | equal to                 |
| !=       | not equal to             |
| >=       | greater than or equal to |
| >        | greater than             |
| ~        | matched by               |
| !~       | not matched by           |

举例：

* NF>10: 选择 field 大于 10 的行
* NF: 直接数字, match when it's numberic value is nonzero
* string, match when value of expression is nonnull
* num operator num, 做计算
* num operator str, 都转化为 string 做计算
* string operator string, 按位比较顺序. "Canada" < "China"

```sh
awk '$0 >= "M"' countries                                                     
USSR    8649    275     Asia
USA     3615    237     North America
Mexico  762     78      North America
```

### String-matching Patterns

Awk 支持 regular expressions

String-Matching Pattern

* /regexpr/: 目标是行内容的一部分
* expression ~ /regexpr/: matches if the string value of expression contains a substring matched by regexpr
* expression !~ /regexpr/: 和上面的相反

```sh
awk '$4 ~ /Asia/' countries 
USSR    8649    275     Asia
China   3705    1032    Asia
India   1267    746     Asia
Japan   144     120     Asia

awk '$4 !~ /Asia/' countries
Canada  3852    25      North America
USA     3615    237     North America
Brazil  286     134     South America
Mexico  762     78      North America
France  211     55      Europe
Germany 96      61      Europe
England 94      56      Europe
```

这部分可以这样理解，基本的 awk 格式是 `pattern { action }`, 而这部分 match 是 pattern 的扩展。pattern = expression (!~) /regexpr/

### Regular Expressions

这部分自信已经不用笔记了。。。

只记一个 `(Asian|European|North American)` 表示单词级别的选择关系

ESCAPE SEQUENCES

| SEQUENCE | MEANING                                              |
| :------- | :--------------------------------------------------- |
| \b       | backspace                                            |
| \f       | formfeed                                             |
| \n       | newline (line feed)                                  |
| \r       | carriage return                                      |
| \t       | tab                                                  |
| \ddd     | octal value ddd, where ddd is 1-3 digits between 0-7 |
| \c       | any other character c literally                      |

### Compound Patterns

混合模式，即多个表示式通过逻辑运算符组合

```sh
awk '$4 == "Asia" && $3 > 500' countries
China   3705    1032    Asia
India   1267    746     Asia

awk '$4 == "Asia" || $4 == "Europe"' countries
USSR    8649    275     Asia
China   3705    1032    Asia
India   1267    746     Asia
France  211     55      Europe
Japan   144     120     Asia
Germany 96      61      Europe
England 94      56      Europe
```

上面的例子是字符比较，也可以使用正则

```sh
awk '$4 ~ /^(Asia|Europe)$/' countries 
USSR    8649    275     Asia
China   3705    1032    Asia
India   1267    746     Asia
France  211     55      Europe
Japan   144     120     Asia
Germany 96      61      Europe
England 94      56      Europe
```

如果其他 field 不包含这两个关键字，还可以用逻辑或筛选

```sh
# 等价于 awk '/Asia|Europe/' countries
awk '/Asia/||/Europe/' countries      
USSR    8649    275     Asia
China   3705    1032    Asia
India   1267    746     Asia
France  211     55      Europe
Japan   144     120     Asia
Germany 96      61      Europe
England 94      56      Europe
```

优先级：! > && > ||

同优先级(|| + &&)的操作 从左到右 的顺序计算

### Range Patterns

即两个 pattern 用逗号间隔, `pat1， pat2` 表示取匹配的 1 行或 n 行内容. 如下面的例子，选取 Canada 出现到 USA 之间的所有行

```sh
awk '/Canada/, /USA/' countries 
Canada  3852    25      North America
China   3705    1032    Asia
USA     3615    237     North America
```

如果后一个 pattern 没有匹配的内容，则匹配到末尾

```sh
awk '/Europe/, /Africa/' countries
France  211     55      Europe
Japan   144     120     Asia
Germany 96      61      Europe
England 94      56      Europe
```

* FNR: the number of the line just read from current input file
* FILENAME: the filename itself

打印第一行到第五行

```sh
awk 'FNR == 1, FNR == 5 { print FILENAME ":" $0 }' countries 
countries:USSR  8649    275     Asia
countries:Canada        3852    25      North America
countries:China 3705    1032    Asia
countries:USA   3615    237     North America
countries:Brazil        286     134     South America
```

同样的效果还可以写成

```sh
awk 'FNR <= 5 { print FILENAME ": " $0 }' countries         
countries: USSR 8649    275     Asia
countries: Canada       3852    25      North America
countries: China        3705    1032    Asia
countries: USA  3615    237     North America
countries: Brazil       286     134     South America
```

### Summary of Patterns

总结 pattern 支持的格式

| PATTERN         | EXAMPLE                  | MATCHES                                           |
| :-------------- | :----------------------- | :------------------------------------------------ |
| BEGIN           | BEGIN                    | before any input has been read                    |
| END             | END                      | after all input has been read                     |
| expression      | $3 < 100                 | third field less than 100                         |
| string-matching | /Asia/                   | lines that contain Asia                           |
| compound        | $3 < 100 && $4 == "Asia" | thrid fields less than 100 + fourth field is Asia |
| range           | NR==10, NR==20           | tenth to twentieth lines of input inclusive       |

## Actions

在 pattern-action 的格式中，pattern 决定了是否执行 action。action 可以很简单，比如打印；也可以很复杂，比如多语句操作或者包含控制流什么的。下面章节会介绍自定义函数和输入，输出的一些语法。

actions 中可以包含下列语法

* 包含常量，变量，赋值函数调用的 expressions
* print
* printf
* if 语句
* if - else
* while
* for (expression; expression; expression) statement
* for (variable in array) statement
* do statement while (expression)
* break
* continue
* next
* exit
* exit expression
* { statement }

### Expressions

expression 是最简单的语句，expression 之间可以通过 operators 连接，有五种 operators

* arthmetic
* comparison
* logical
* conditional
* assignment

#### Constants

两种常数类型：string and numberic

string = 双引号 + 字符 + 双引号，字符包括转义字符

numberic 都是由浮点类型的值表示的，可以有不同的形态，但是内存中都是浮点表示，比如 1e6, 1.00E6 等形式

#### Variables

* user-defined
* built-in
* fields

由于变量的类型是没有声明的，所以 awk 会根据上下文推断变量类型，必要时它会做 string 和 numberic 之间的转换。

还没有初始化的时候 string 默认是 "" (the null string), numberic 默认是 0

#### Built-in Variables

下面是一些自带的变量，FILENAME 在每次读文件时都会自动赋值。FNR，NF 和 NR 在每次读入一行时重置。

| VARIABLE | MEANING                                    | DEFAULT |
| :------- | :----------------------------------------- | :------ |
| ARGC     | number of command line arguments           | -       |
| ARGV     | array of command line arguments            | -       |
| FILENAME | name of current input file                 | -       |
| FNR      | record number in current file              | -       |
| FS       | controls the input field separator         | " "     |
| NF       | number of fields in current record         | -       |
| NR       | number of records read so far              | -       |
| OFMT     | output format for numbers                  | "%.6g"  |
| OFS      | output field separator                     | " "     |
| ORS      | output record separator                    | "\n"    |
| RLENGTH  | length of string matched by match function | -       |
| RS       | controls the input recrod separator        | "\n"    |
| RSTART   | start of string matched by match function  | -       |
| SUBSEP   | subscript separator                        | "\034"  |

#### Field Variables

表示当前行的 field 参数，从 $1 - $NF, $0 表示整行。运行一些例子找找感觉

```sh
# 第二个 field 值缩小 1000 倍并打印
awk '{ $2 = $2 / 1000; print }' countries 

USSR 8.649 275 Asia
Canada 3.852 25 North America
China 3.705 1032 Asia
...
```

将 North America 和 South America 替换为简写

```sh
awk 'BEGIN { FS = OFS = "\t" }
$4 == "North America" { $4 = "NA" }
$4 == "South America" { $4 = "SA" } 
{print}
' countries
USSR    8649    275     Asia
Canada  3852    25      NA
China   3705    1032    Asia
USA     3615    237     NA
Brazil  286     134     SA
India   1267    746     Asia
Mexico  762     78      NA
France  211     55      Europe
Japan   144     120     Asia
Germany 96      61      Europe
England 94      56      Europe
```

PS: 这里之前我倒是没有意识到，上面的做法其实就是多种情况替换的案例了

还有一些比较神奇的使用方式，比如 $(NF - 1) 可以取得倒数第二个 field。如果 field 不存在，默认值为 null string, 比如 $(NF + 1), 一个新的 field 可以通过赋值得到，比如下面的例子是在原有的数据后面添加第五列元素

```sh
awk 'BEGIN { FS = OFS = "\t" }; { $5 = 1000 * $3 / $2; print }' countries 
USSR    8649    275     Asia    31.7956
Canada  3852    25      North America   6.49013
...
```

#### Arthmetic Operators

awk 提供常规计算 +, -, *, %, ^.

#### Comparison Operators

支持常见的比较操作：<, <=, ==, !=, >=, >。还有匹配符号 ～ 和 ！～。比较的结果为 1(true)/0(false) 二选一。

#### Logical Operators

逻辑运算符有：&&, ||, !

#### Condition Expressions

expr1 ? expr2 : expr3 效果和 Java 中的一致. 下面的例子会打印 $1 的倒数，如果 $1 为 0 则打印提示信息

```sh
awk '{ print ($1 !=0 ? 1/$1 : "$1 is zero, line " NR) }' 
```

#### Assignment Operators

var = expr, 下面的例子计算所有亚洲国家的人口和

```sh
awk '$4 == "Asia" { pop = pop + $3; n = n + 1}
END { print "Total population of the ", n, "Asian countries is", pop, "million"}' countries 
Total population of the  4 Asian countries is 2173 million
```

统计人口最多的国家

```sh
awk '$3 > maxpop {maxpop = $3; country = $1}
END { print "country with largest population:", country, maxpop }' countries
country with largest population: China 1032
```

#### Increment and Decrement Oerators

n = n + 1 通常简写为 ++n 或者 n++, 区别是，如果有赋值，则 n++ 会将原始值赋给变量再自增，++n 则先自增再赋值

```sh
awk 'BEGIN { n=1; i=n++ }; END { print i }' countries
1
awk 'BEGIN { n=1; i=++n }; END { print i }' countries
2
```

#### Built-In Arithmetic Functions

| FUNCTION    | VALUE RETURNED                                    |
| :---------- | :------------------------------------------------ |
| atan2(y, x) | arctangent of y/x in the range -pi to pi          |
| cos(x)      | cosine of x, with x in radians                    |
| exp(x)      | exponential function of x, e<sup>x</sup>          |
| int(x)      | integer part of x; truncated towards 0 when x > 0 |
| log(x)      | natural (base e) logarithm of x                   |
| rand()      | random number r, where 0 <= r < 1                 |
| sin(x)      | sine of x, with x in radians                      |
| sqrt(x)     | square root of x                                  |
| srand(x)    | x is new seed for rand ()                         |

#### String Operators

awk 只支持一种字符串操作 - 拼接。拼接不需要任何的连接符, 比如下面的例子是在每行前面打印行号 + 冒号的前缀

```sh
awk '{ print NR ":" $0 }' countries                  
1:USSR  8649    275     Asia
2:Canada        3852    25      North America
...
```

#### Strings as Regular Expressions

`awk 'BEGIN { digits = "^[0-9]+$" }; $2 ~ digits' countries`, 表达式可以动态拼装，所以下面的例子也是合法的

```sh
BEGIN {
  sign = "[+-]?"
  decimal= "[0-9]+[.]?[0-9]*"
  fraction= "[.][0-9]+"
  exponent= "([eEl" sign "[0-9]+)?"
  number= "^" sign "(" decimal "|" fraction ")" exponent "$"
}
$0 .. number
```

#### Built-In String Functions

| Function                  | Description                                                                                            |
| :------------------------ | :----------------------------------------------------------------------------------------------------- |
| gsub(r,s)                 | substitute s for r globally in $0, return number of substitutions made                                 |
| gsub(r ,s ,t)             | substitutes for r globally in string t, return number of substitutions made                            |
| index(s ,t)               | return first position of string t in s, or 0 if t is not present                                       |
| length(s)                 | return number of characters in s                                                                       |
| match(s ,r)               | test whether s contains a substring matched by r,return index or 0; sets RSTART and RLENGTH            |
| split(s ,a)               | split s into array a on FS, return number of fields                                                    |
| split(s ,a ,fs)           | splits into array a on field separator fs, return number of fields                                     |
| sprintf(fmt, expr -list ) | return expr -list formatted according to format string fmt                                             |
| sub(r ,s)                 | substitutes for the leftmost longest substring of $0 matched by r, return number of substitutions made |
| sub(r ,s ,t)              | substitute s for the leftmost longest substring of t matched by r, return number of substitutions made |
| substr (s ,p)             | return suffix of s starting at position p                                                              |
| substr (s ,p ,n)          | return substring of s of length n starting at position p                                               |

p42