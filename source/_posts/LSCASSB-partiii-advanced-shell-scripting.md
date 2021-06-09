---
title: Linux命令行与shell脚本编程大全 第三章 Shell 高级用法
date: 2021-06-05 16:38:52
categories:
- Shell
tags:
- Linux命令行与shell脚本编程大全 3rd
- TODO
---

## Creating Functions

### Basic Script Functions

#### Creating a function

方式一：

```sh
function name {
    commands
}
```

方式二：

```sh
name() {
    commands
}
```

#### Using functions

function 不一定要写在开头，但是你使用之前，他的定义必须已经声明了

```sh
cat test1
#!/usr/local/bin/bash
# using a function in a script

function func1 {
    echo "This is an example of a function"
}

count=1
while [ $count -le 4 ]
do
    func1
    count=$[ $count + 1 ]
done

echo "This is the end of the loop"
func1
echo "Now this is the end of the script"

./test1
# This is an example of a function
# This is an example of a function
# This is an example of a function
# This is an example of a function
# This is the end of the loop
# This is an example of a function
# Now this is the end of the script
```

### Returning a Value

bash shell 会在函数结束后给他一个返回值

#### The default exit status

默认情况下，exit status 是函数中最后一个命令的返回值，可以使用 `$?` 查看

下面实验中，我们在 func1 中 ls 一个不存在的文件，并打印 exit code

```sh
cat test4
#!/usr/local/bin/bash
# testing the exit status of a function

function func1 {
    echo "Trying to display a non-existent file"
    ls -l badfile
}

echo "testing the function"
func1
echo "The exit status is: $?"

./test4 
# testing the function
# Trying to display a non-existent file
# ls: badfile: No such file or directory
# The exit status is: 1

cat test4b
#!/usr/local/bin/bash
# testing the exit status of a function

function func1 {
    ls -l badfile
    echo "Trying to display a non-existent file"
}

echo "testing the function"
func1
echo "The exit status is: $?"


./test4b
# testing the function
# ls: badfile: No such file or directory
# Trying to display a non-existent file
# The exit status is: 0
```

#### Using the return command

你也可以用 return 关键子在函数末尾给他返回一个整数最 exit code

```sh
cat test5
#!/usr/local/bin/bash
# using the return command in a function

function db1 {
    read -p "Enter a value: " value
    echo "doubling the value"
    return $[ $value * 2 ]
}

db1
echo "The new value is $?"

./test5
# Enter a value: 10
# doubling the value
# The new value is 20
```

注意点：

* 记得在 function 执行完成后立刻取值备用
* exit code 返回只能在 0-255 之间

#### Using function output

你也可以将返回值包装在输出里面，通过 `result=$(db1)` 的形式拿到。和之前调用系统函数一个套路

```sh
cat test5b
#!/usr/local/bin/bash
# using the return command in a function

function db1 {
    read -p "Enter a value: " value
    echo $[ $value * 2 ]
}

result=$(db1)
echo "The new value is $result"

./test5b
# Enter a value: 20
# The new value is 40
```

**Caution** 如果函数中有多个 echo 他会将所有的 echo 内容合在一起作为 return 的值

```sh
cat freturn.sh
#!/usr/local/bin/bash
# multiple echo sentence in function

function multiecho {
  echo "return 1"
  echo "return 2"
}

./freturn.sh
# return 1
# return 2
```

**Question** 如果我把 echo 和 return 结合使用，他会拿什么当返回值？？？

结论：return 会被忽略

```sh
cat ./returnwithecho.sh
#!/usr/local/bin/bash
# multiple echo sentence and return in function

function multiecho {
  echo "return 1"
  echo "return 2"
  return 55
}

./freturn.sh
# return 1
# return 2
```

### Using Variables in Functions

#### Passing parameters to a function

```sh
cat test6
#!/usr/local/bin/bash
# passing parameters to a function

function addem {
    if [ $# -eq 0 ] || [ $# -gt 2 ]
    then
        echo -1
    elif [ $# -eq 1 ]
    then
        echo $[ $1 + $1 ]
    else
        echo $[ $1 + $2 ]
    fi
}

echo -n "Adding 10 and 15: "
value=$(addem 10 15)
echo $value
echo -n "Let's try adding just one number: "
value=$(addem 10)
echo $value
echo -n "Now trying adding no numbers: "
value=$(addem)
echo $value
echo -n "Finally, try adding three numbers: "
value=$(addem 10 15 20)
echo $value

./test6
# Adding 10 and 15: 25
# Let's try adding just one number: 20
# Now trying adding no numbers: -1
# Finally, try adding three numbers: -1
```

但是脚本文件中的参数并不会默认的应用到你定义的 function 中去，你需要显示的指定才能使他生效

下面实验中，当我们运行脚本是没有给参数，则直接运行了 else 的路径，如果给了参数，这个参数也不会传递给 badfunc1

```sh
cat badtest1
#!/usr/local/bin/bash
# Trying to access script parameters inside a function

function badfunc1 {
    echo $[ $1 * $2 ]
}

if [ $# -eq 2 ]
then
    value=$(badfunc1)
    echo "The result is $value"
else
    echo "Usage: badtest1 a b"
fi

./badtest1
# Usage: badtest1 a b
./badtest1 10 15
# ./badtest1: line 5: *  : syntax error: operand expected (error token is "*  ")
# The result is
```

如果要修复上面的脚本，你可以直接在实现中指定 `value=$(badfunc1 $1 $2)`

```sh
cat test7
#!/usr/local/bin/bash
# Trying to access script parameters inside a function

function func7 {
    echo $[ $1 * $2 ]
}

if [ $# -eq 2 ]
then
    value=$(func7 $1 $2)
    echo "The result is $value"
else
    echo "Usage: badtest1 a b"
fi

./test7
# Usage: badtest1 a b
./test7 10 15
# The result is 150
```

#### Handling variables in a function

Functions 使用两种类型的变量

* Global
* Local

Global 变量是在 script 中任何地方都可以访问的变量。在主程序中定义的 global 变量，function 中可以访问到。function 中定义的 global 变量，主程序也能访问到。一般来说你在脚本中定义的变量都是 global 的

```sh
cat test8
#!/usr/local/bin/bash
# Using a global variable to pass a value

function db1 {
    value=$[ $value * 2 ]
}

read -p "Enter a value: " value
db1
echo "The new value is: $value"

./test8
# Enter a value: 10
# The new value is: 20
```

如果你在 script 和 function 中都用到了同名的变量，就可能导致赋值出问题，是你的程序难以排错

```sh
cat badtest2
#!/usr/local/bin/bash
# Demonstrating a db use of variables

function func1 {
    temp=$[ $value + 5 ]
    result=$[ $temp * 2 ]
}

temp=4
value=6

func1
echo "The result is $result"
if [ $temp -gt $value ]
then
    echo "temp is larger"
else
    echo "temp is smaller"
fi

./badtest2
# The result is 22
# temp is larger
```

我们可以使用 local variables 避免这种问题，格式如下 `local temp`. 他让你的变量只在 function 中生效

```sh
cat test9
#!/usr/local/bin/bash
# Demonstrating the local keyword

function func1 {
    local temp=$[ $value + 5 ]
    result=$[ $temp * 2 ]
}

temp=4
value=6

func1
echo "The result is $result"
if [ $temp -gt $value ]
then
    echo "temp is larger"
else
    echo "temp is smaller"
fi


./test9
# The result is 22
# temp is smaller
```

### Array Variables and Functions

#### Passing arrays to functions

函数处理数组的方式有点特别，如果你传给函数一个数组，他默认只会取第一个元素作为参数

```sh
cat badtest3
#!/usr/local/bin/bash
# Trying to pass an array variable

function testit {
    echo "The parameters are: $@"
    thisarray=$1
    echo "The received array is ${thisarray[*]}"
}

myarray=(1 2 3 4 5)
echo "The original array is: ${myarray[*]}"
testit  $myarray

./badtest3
# The original array is: 1 2 3 4 5
# The parameters are: 1
# The received array is 1
```

你可以将数组 disassemble 之后传给函数，在使用时在 reassemble 即可. 书上给的例子不能运行，网上找了一个可用的表达式

```sh
cat test10
#!/usr/local/bin/bash
# array variable to function test

function testit {
    local newarray
    # 原始表达式为 newarray=(;'echo "$@"')
    newarray=($(echo "$@"))
    echo "The new array value is ${newarray[*]}"
}

myarray=(1 2 3 4 5)
echo "The original array is: ${myarray[*]}"
testit  ${myarray[*]}

./test10
# The original array is: 1 2 3 4 5
# The new array value is 1 2 3 4 5
```

#### Return arrays from functions

函数返回一个数组用了同样的技巧，函数中一个一个的 echo 值，接收方需要 reassemble 它们

```sh
cat test12
#!/usr/local/bin/bash
# returning an array value

function arraydblr {
    local origarray
    local newarray
    local elements
    local i
    origarray=($(echo "$@"))
    newarray=($(echo "$@"))
    elements=$[ $# - 1 ]
    for (( i=0; i<= $elements; i++ ))
    {
        newarray[$i]=$[ ${origarray[$i]} * 2 ]
    }
    echo ${newarray[*]}
}

myarray=(1 2 3 4 5)
echo "The original array is: ${myarray[*]}"
arg1=$(echo ${myarray[*]})
result=($(arraydblr $arg1))
echo "The new array is: ${result[*]}"

./test12
# The original array is: 1 2 3 4 5
# The new array is: 2 4 6 8 10
```

### Function Recursion

local function variable 提供 self-containment 功能。他使得函数可以实现 recursively 的效果

实现斐波那契数列

```sh
cat test13
#!/usr/local/bin/bash
# using recursion

function factorial {
    if [ $1 -eq 1 ]
    then 
        echo 1
    else
        local temp=$[ $1 - 1 ]
        local result=$(factorial $temp)
        echo $[ $result * $1 ]
    fi
}

read -p "Enter value: " value
result=$(factorial $value)
echo "The factorial of $value is: $result"

./test13
# Enter value: 5
# The factorial of 5 is: 120
```

### Creating a Library

建立自己的函数库，实现脚本文件中的可复用。简单来说，就是将所有的函数都写在文件中。在其他脚本文件中，通过 `. path/to/myfuncs` 的语法引入自己的库文件即可。这个点号是 `source` 的快捷方式叫做 dot operator.

```sh
cat myfuncs 
#!/usr/local/bin/bash
# my script functions

function addem {
    echo $[ $1 + $2 ]
}

function multem {
    echo $[ $1 * $2 ]
}

function divem {
    if [ $2 -ne 0 ]
    then
        echo $[ $1 / $2 ]
    else
        echo -1
    fi
}

cat test14
#!/usr/local/bin/bash
# using functions defined in a library file
. ./myfuncs

value1=10
value2=5
result1=$(addem $value1 $value2)
result2=$(multem $value1 $value2)
result3=$(divem $value1 $value2)
echo "The result of adding them is: $result1"
echo "The result of multiplying them is: $result2"
echo "The result of dividing them is: $result3"

./test14
# The result of adding them is: 15
# The result of multiplying them is: 50
# The result of dividing them is: 2
```

### Using Functions on the Command Line

#### Creating functions on the command line

方式一：终端一行定义

```sh
function doubleit { read -p "Enter value:" value; echo $[ $value * 2 ]; }
doubleit
# Enter value:3
6
```

方式二：多行定义，in this way, no need of semicolon

```sh
function multem {
> echo $[ $1 * $2 ]
> }
multem 2 5
# 10
```

**Caution** 如果你终端定义函数的时候和系统自带的函数重名了，那么这个函数会覆盖系统函数。

#### Defining functions in the .bashrc file

上面的这个方式，当终端退出时，函数就丢失了，你可以将函数写入 .bashrc 文件或使用 source 函数库的方式达到复用的效果。而且最方便的事，如果你将他们通过 bashrc 引入，你在写脚本的时候就不需要 source 了，直接可以调用

### Following a Practical Example

这章展示如何使用开源 shell 工具包

1. `wget ftp://ftp.gnu.org/gnu/shtool/shtool-2.0.8.tar.gz` 下载实验包并 `tar -zxvf shtool-2.0.8.tar.gz` 解压
2. cd 到解压后的文件夹，`./configure` + `make`, 当然你也可以用 `make test` 测试一下构建
3. `make install` 安装库文件

shtool 包含了一系列的工具集，可以使用 `shtool [options] [fucntion [options] [args]]` 来查看

The shtool Library Functions

| Function | Description                             |
| :------- | :-------------------------------------- |
| platform | Displays the platform identity          |
| Prop     | Dispalys an animated progress propeller |

只列出用到的几个

```sh
cat test16
#!/usr/local/bin/bash

shtool platform

./test16
# Mac OS X 11.4 (iX86)
```

带进度条的显示 `ls -al /usr/bin | shtool prop -p "waiting..."` 太快了，看不出效果

## Chapter 18: Writing Scripts for Graphical Desktops

和我这次看书的目标不符，跳过

## Chapter 19: Introducing sed and gawk

实际工作中，很多工作都是文字处理相关的。使用 shell 自带工具处理文字会显得很笨拙。这时候就要用到 sed 和 gawk 了。

### Manipulating Text

#### Getting to know the sed editor

sed 是一个流处理编辑器(stream editor)，你可以设定一系列的规则，然后通过这个流编辑器处理他。

sed 可以做如下事情

1. Reads one data line at a time from the input
2. Matches that data with the supplied editor commands
3. Changes data in the stream as specified in the commands
4. Outputs the new data to STDOUT

按行一次处理文件直到所有内容处理完毕结束，格式 `sed options script file`

The sed Command Options

| Option    | Description                                                                            |
| :-------- | :------------------------------------------------------------------------------------- |
| -e script | Adds commands specified in the script to the commands run while processing the input   |
| -f file   | Adds the commands specified in the file to the commands run while processing the input |
| -n        | Doesn't produce output for each command, but waits for the print command               |

#### Defining an editor command int the command line

```sh
echo "This is a test" | sed 's/test/big test/'
# This is a big test
```

`s` 表示替换(substitutes), 他会用后一个字符串替换前一个. 下面是替换文件内容的例子. sed 只会在输出内容中做修改，原文件还是保持原样

```sh
cat data1.txt
# The quick brown fox jumps over the lazy dog.
# The quick brown fox jumps over the lazy dog.
# The quick brown fox jumps over the lazy dog.
# The quick brown fox jumps over the lazy dog.
sed 's/dog/cat/' data1.txt 
# The quick brown fox jumps over the lazy cat.
# The quick brown fox jumps over the lazy cat.
# The quick brown fox jumps over the lazy cat.
# The quick brown fox jumps over the lazy cat.
```

#### Using mulitple editor commands int the command line

```sh
sed -e 's/brown/green/; s/dog/cat/' data1.txt 
# The quick green fox jumps over the lazy cat.
# The quick green fox jumps over the lazy cat.
# The quick green fox jumps over the lazy cat.
# The quick green fox jumps over the lazy cat.
```

如果不想写在一行，可以写在多行

```sh
sed -e '
> s/brown/green/
> s/fox/elephant/
> s/dog/cat/' data1.txt
# The quick green elephant jumps over the lazy cat.
# The quick green elephant jumps over the lazy cat.
# The quick green elephant jumps over the lazy cat.
# The quick green elephant jumps over the lazy cat.
```

#### Reading editor commands from a file

如果命令太多，也可以将他们放到文件中

```sh
cat script1.sed 
# s/brown/green/
# s/fox/elephant/
# s/dog/cat/
sed -f script1.sed data1.txt 
# The quick green elephant jumps over the lazy cat.
# The quick green elephant jumps over the lazy cat.
# The quick green elephant jumps over the lazy cat.
# The quick green elephant jumps over the lazy cat.
```

为了便于区分 shell 文件和 sed 条件，我们将 sed 条件文件存储为 `.sed` 结尾

#### Getting to know the gawk program

sed 让你动态改变文件内容，但是还是有局限性。gawk 提供一个更程序化的方式来处理文本信息. 这个工具包默认是没有的一般需要自己手动安装。gawk 是 GNU 版本的 awk, 通过它你可以

* Define variables to store data
* Use arithmetic and string operatiors to perate on data
* Use structured programming concepts, such as if-then statements and loops, to add logic to your data processing
* Generate formatted reports by extracting data elements within the data file and repositioning them in another order or format

第四点经常用来批量处理数据使之更具可读性，典型应用就是处理 log 文件。

#### Visiting the gawk command format

格式 `gawk options program file`

The gawk Options

| Option       | Description                                                        |
| :----------- | :----------------------------------------------------------------- |
| -F fs        | Specifies a file separator for delineating data fields in a line   |
| -f file      | Specifies a file name to read the program from                     |
| -v var=value | Defines a variable and default value used in the gawk program      |
| -mf N        | Specifies the maximum number of fields to process in the data file |
| -mr N        | Specifies the maximum record size in the data file                 |
| -W keyword   | Specifies the compatibility mode or warning level of gawk          |

gawk 最大的优势是可以用编程的手段，将数据重新格式化输出

#### Reading the program script from the command line

格式 `gawk '{ commands }'` 比如 `gawk '{print "Hello World!"}'` 这个 demo 命令并没有做什么文字处理，只是简单的接收标准输入然后打印 Hello World. 使用 Ctrl + D 结束对话

gawk 最主要的功能是提供操作文本中数据的功能，默认情况下，gawk 会提取如下变量

* $0 represents the entire line of text
* $1 represents the first data field in the line of text
* $2 represents the second data field in the line of text
* $n represents the nth data field in the line of text

gawk 会根据命令中提供的分割符做行的分割，下面是 gawk 读取文件并显示第一行的示例

```sh
cat data2.txt 
# One line of test text.
# Two lines of test text.
# Three lines of test text.

gawk '{ print $1 }' data2.txt 
# One
# Two
# Three
```

可以用 `-F` 指定分割符，比如你要处理 /etc/passwd 这个文件

```sh
cat /etc/passwd | tail -3
# _coreml:*:280:280:CoreML Services:/var/empty:/usr/bin/false
# _trustd:*:282:282:trustd:/var/empty:/usr/bin/false
# _oahd:*:441:441:OAH Daemon:/var/empty:/usr/bin/false

gawk -F: '{print $1}' /etc/passwd | tail -3 
# _coreml
# _trustd
# _oahd
```

#### Using multiple commands in the program script

使用分号分割 gawk 中想要运行的多个命令, 下面的命令会替换第四个 field 并打印整行

```sh
echo "My name is Rich" | gawk '{$4="Christine";print $0}'
# My name is Christine
```

多行表示也是 OK 的

```sh
gawk '{
> $4="Christine"
> print $0
> }'
my name is Rich
my name is Christine
```

#### Reading the program from a file

和 sed 一样，gawk 也支持从文件读取命令

```sh
cat script2.gawk 
# {print $1 "'s hoe directory is " $6}
gawk -F: -f script2.gawk /etc/passwd | tail -3
# _coreml's hoe directory is /var/empty
# _trustd's hoe directory is /var/empty
# _oahd's hoe directory is /var/empty
```

gawk 文件中包含多个命令的示例

```sh
cat script3.gawk 
# {
#     text = "'s home directory is "
#     print $1 text $6
# }
gawk -F: -f script3.gawk /etc/passwd | tail -3
# _coreml's home directory is /var/empty
# _trustd's home directory is /var/empty
# _oahd's home directory is /var/empty
```

#### Running scripts before processing data

gawk 提供了 BEGIN 关键字在处理文本前做一些操作

```sh
gawk 'BEGIN {print "Hello World!"}'
# Hello World!
```

BEGIN 处理文本的示例

```sh
cat data3.txt 
# Line 1
# Line 2
# Line 3
gawk 'BEGIN {print "The data3 File Contents: "}
> {print $0}' data3.txt
The data3 File Contents: 
# Line 1
# Line 2
# Line 3
```

#### Running scripts after processing data

和前面对应的还有一个 after 操作

```sh
gawk 'BEGIN {print "The data3 File Contents:"}
{print $0}
> END {print "End of File"}' data3.txt
# The data3 File Contents:
# Line 1
# Line 2
# Line 3
# End of File
```

如果过程多了，你还可以将这个步骤写到文件中

```sh
at script4.gawk 
BEGIN {
    print "The latest list of users and selles"
    print " UserID\t Shell"
    print "-------\t------"
    FS=":"
}

{
    print $1 "    \t " $7
}

END {
    print "This concludes the listing"
}

gawk -f script4.gawk  /etc/passwd
# The latest list of users and selles
#  UserID  Shell
# ------- ------
# nobody           /usr/bin/false
# ...
# _oahd            /usr/bin/false
# This concludes the listing
```

### Commanding at the sed Editor Basics

本章简要介绍一下 sed 的常规用法

#### Introducing more substitution options

##### Substituting flags

```sh
cat data4.txt 
# This is a test of the test script.
# This is the second test of the test script.
sed 's/test/trial/' data4.txt 
# This is a trial of the test script.
# This is the second trial of the test script.
```

默认情况下，sed 只会替换每行中第一个出现的位置，如果想要处理多个位置，需要指定 flags。格式为 `s/pattern/replacemnet/flags`

four types of substitution flags are available:

* A number, indicating the pattern occurrence for which new text should be substituted
* g, indicating that new text should be substituted for all occrurences of the existing text
* p, indicating that the contents of the original line should be printed
* w file, which means to write the results of the substitution to a file

替换指定位置的示例，下面示例中只替换了第二个位置的 test

```sh
sed 's/test/trial/2' data4.txt 
# This is a test of the trial script.
# This is the second test of the trial script.
```

全部替换示例

```sh
sed 's/test/trial/g' data4.txt 
This is a trial of the trial script.
This is the second trial of the trial script.
```

打印符合匹配条件的行

```sh
cat data5.txt 
# This is a test line.
# This is a different line.

sed -n 's/test/trial/p' data5.txt
# This is a trial line.
```

`-w` 指定 sed 结果输出到文件

```sh
sed 's/test/trial/w test.txt' data5.txt 
# This is a trial line.
# This is a different line.
cat test.txt 
# This is a trial line.
```

##### Replacing characters

Linux 系统中路径符号和 sed 中的符号是重的，也就是说，如果我要用 sed 替换路径的时候就必须用一中很累赘的写法, 比如 `sed 's/\/bin\/bash/\/bin\/csh/' /etc/passwd`

PS: Mac OS 语法和这个不一样

为了避免这么恶心的写法，我们可以用惊叹号(exclamation point) 代替原来的分割符 `sed 's!/bin/bash!/bin/csh!' /etc/passwd`

#### Using addresses

默认情况下 sed 会处理所有的行，如果你只需要处理特殊的几行，你可以使用 line address. line address 有两种模式

* A numberic range of lines
* A text pattern that filters out a line

两种模式的格式都是一样的 `[address] command` 你可以将多个命令组合到一起

```sh
address {
    command1
    command2
    command3
}
```

##### Addressing the numberic line

sed 会将 s 之前的内容当作行来处理, 下面的例子只替换第二行的内容

```sh
sed '2s/dog/cat/' data1.txt
# The quick brown fox jumps over the lazy dog.
# The quick brown fox jumps over the lazy cat.
# The quick brown fox jumps over the lazy dog.
# The quick brown fox jumps over the lazy dog.
```

替换多行

```sh
sed '2,3s/dog/cat/' data1.txt
# The quick brown fox jumps over the lazy dog.
# The quick brown fox jumps over the lazy cat.
# The quick brown fox jumps over the lazy cat.
# The quick brown fox jumps over the lazy dog.
```

从第 n 行还是到结束

```sh
sed '2,$s/dog/cat/' data1.txt
# The quick brown fox jumps over the lazy dog.
# The quick brown fox jumps over the lazy cat.
# The quick brown fox jumps over the lazy cat.
# The quick brown fox jumps over the lazy cat.
```

##### Using text pattern filters

使用 pattern 处理特定的行，格式 `/pattern/command`, 你需要在 pattern 前面指定一个斜杠作为开始

下面的示例中我们只将 root 的 sh 改为 csh

```sh
grep root /etc/passwd
# root:*:0:0:System Administrator:/var/root:/bin/sh
# daemon:*:1:1:System Services:/var/root:/usr/bin/false
# _cvmsroot:*:212:212:CVMS Root:/var/empty:/usr/bin/false
sed '/root/s/sh/csh/' /etc/passwd | grep root
# root:*:0:0:System Administrator:/var/root:/bin/csh
# daemon:*:1:1:System Services:/var/root:/usr/bin/false
# _cvmsroot:*:212:212:CVMS Root:/var/empty:/usr/bin/false
```

sed 是通过正则表达式来做内容匹配的。

##### Grouping commands

和 gawk 一样，sed 也可以在一个命令中处理做个匹配

```sh
sed '2{
s/fox/elephant/
s/dog/cat/
}' data1.txt
# The quick brown fox jumps over the lazy dog.
# The quick brown elephant jumps over the lazy cat.
# The quick brown fox jumps over the lazy dog.
# The quick brown fox jumps over the lazy dog.

sed '3,${
s/fox/elephant/
s/dog/cat/
}' data1.txt
# The quick brown fox jumps over the lazy dog.
# The quick brown fox jumps over the lazy dog.
# The quick brown elephant jumps over the lazy cat.
# The quick brown elephant jumps over the lazy cat.
```

##### Deleting lines

`d` 用来在输出内容中删除某一行, 如果没有指定删选内容，所有输出都会被删除

```sh
cat data6.txt 
# This is line number 1.
# This is line number 2.
# This is line number 3.
# This is line number 4.
sed 'd' data6.txt 

```

指定删除的行

```sh
sed '3d' data6.txt 
# This is line number 1.
# This is line number 2.
# This is line number 4.

sed '2,3d' data6.txt 
# This is line number 1.
# This is line number 4.

sed '3,$d' data6.txt 
# This is line number 1.
# This is line number 2.

sed '/number 1/d' data6.txt 
# This is line number 2.
# This is line number 3.
# This is line number 4.
```

PS: 这个删除只作用的输出，原文件保持不变

还有一种比较奇葩的删除方式，给两个匹配，删除会从第一个还是，第二个结束，删除内容包括当前行

```sh
'/1/,/3/d' data6.txt 
# This is line number 4.
```

这里有一个坑，这种方式是匹配删除，当第一匹配时，删除开始，第二个匹配找到时，删除结束。如果文件中有多个地方能匹配到开始，则可能出现意想不到的情况. 比如在 data7 中，第 5 行也能匹配到 1 这个关键字，但是后面就没有 4 了，则会导致后面的内容都删掉. 如果我指定一个不存在的停止符，则所有内容都不显示了

```sh
cat data7.txt 
# This is line number 1.
# This is line number 2.
# This is line number 3.
# This is line number 4.
# This is line number 1 again.
# This is text you want to keep.
# This is the last line in the file.

sed '/1/,/3/d' data7.txt 
# This is line number 4.

sed '/1/,/5/d' data7.txt
```

##### Inserting and appending text

sed 也允许你插入，续写内容，但是有一些特别的点

* The insert command(i) adds a new line before the specified line
* The append commadn(a) adds a new line after the specified line

特别的点在于，你需要新启一行写这些新加的行, 格式为

```sh
sed '[address]command\
new line'
```

示例如下, 不过 mac 上貌似有语法错误

```sh
echo "Test Line 2" | sed 'i\Test line 1'
# Test line 1
# Test Line 2

echo "Test Line 2" | sed 'a\Test line 1'
# Test Line 2
# Test line 1

echo "Test Line 2" | sed 'i\
> Test Line 1'
# Test Line 1
# Test Line 2
```

上面演示的是在全部内容之前/后添加新的行，那么怎么在特定行前后做类似的操作呢，你可以用行号指定。但是不能用 range 形式的，因为定义上，i/a 是单行操作

```sh
sed '3i\
This is an inserted line.' data6.txt
# This is line number1.
# This is line number2.
# This is an inserted line.
# This is line number3.
# This is line number4.
sed '3a\This is an appended line.' data6.txt
# This is line number1.
# This is line number2.
# This is line number3.
# This is an appended line.
# This is line number4.
```

插入文本末尾

```sh
sed '$a\
> This is a new line of text.' data6.txt
# This is line number1.
# This is line number2.
# This is line number3.
# This is line number4.
# This is a new line of text.
```

头部插入多行, 需要使用斜杠分割

```sh
sed '1i\
> This is one line of new text.\
> This is another line of new text.' data6.txt
# This is one line of new text.
# This is another line of new text.
# This is line number1.
# This is line number2.
# This is line number3.
# This is line number4.
```

如果不指定行号，他会每一行都 insert 啊，和之前的理解不一样. append 也是一样的效果

```sh
sed 'i\head insert' data6.txt
# head insert
# This is line number1.
# head insert
# This is line number2.
# head insert
# This is line number3.
# head insert
# This is line number4
```

也能指定 range... 前面的理解果断有问题

```sh
sed '1,2a\end append' data6.txt
# This is line number1.
# end append
# This is line number2.
# end append
# This is line number3.
# This is line number4.
```

##### Changing lines

改变行内容，用法和前面的 i/a 没什么区别

```sh
sed '3c\This is a chagned line of text.' data6.txt
# This is line number1.
# This is line number2.
# This is a chagned line of text.
# This is line number4.
```

pattern 方式替换

```sh
sed '/number3/c\
This is a changed line of text.' data6.txt
# This is line number1.
# This is line number2.
# This is a changed line of text.
# This is line number4.
```

pattern 替换多行

```sh
cat data8.txt
# This is line number1.
# This is line number2.
# This is line number3.
# This is line number4.
# This is line number1 again.
# This is yet another line.
# This is the last line in the line.

sed '/number1/c\This is a changed line of text.' data8.txt
# This is a changed line of text.
# This is line number2.
# This is line number3.
# This is line number4.
# This is a changed line of text.
# This is yet another line.
# This is the last line in the line.
```

指定行号替换的行为方式有点奇怪，他会将你指定的行中内容全部替换掉

```sh
cat data6.txt
# This is line number1.
# This is line number2.
# This is line number3.
# This is line number4.

sed '2,3c\This is a new line of text.' data6.txt
# This is line number1.
# This is a new line of text.
# This is line number4.
```

##### Transforming characters

transform(y) 是唯 sed 支持的唯一一个用于替换单个字符的参数，格式 `[address]y/inchars/outchars/`. inchars 和 outchars 必须是等长的，不然会报错。他是做一对一替换，比如 `y/123/789` 他会用 1 代替 7， 2 代替 8 依次类推

```sh
sed 'y/123/789/' data8.txt
# This is line number7.
# This is line number8.
# This is line number9.
# This is line number4.
# This is line number7 again.
# This is yet another line.
# This is the last line in the line.
```

而且他是全局替换，任何出现的地方都会被换掉

```sh
echo "This 1 is a test of 1 try." | sed 'y/123/456/'
# This 4 is a test of 4 try.
```

##### Printing revisited

和 p flag 类似的还有两个符号，表示如下

* The p command to print a text line
* The equal sign(=) command to print line numbers
* The l(lowercase L) command to list a line

p 是用案例, -n 可以强制只打印匹配的内容

```sh
echo "this is a test" | sed 'p'
# this is a test
# this is a test

cat data6.txt
# This is line number1.
# This is line number2.
# This is line number3.
# This is line number4.

sed -n '/number3/p' data6.txt
# This is line number3.

sed -n '2,3p' data6.txt
# This is line number2.
# This is line number3.
```

找到匹配的行，先打印原始值，再替换并打印。

```sh
sed -n '/3/{
> p
> s/line/test/p
> }' data6.txt
# This is line number3.
# This is test number3.
```

equals 相关的案例，输出行号

```sh
cat data1.txt 
# The quick brown fox jumps over the lazy dog.
# The quick brown fox jumps over the lazy dog.
# The quick brown fox jumps over the lazy dog.
# The quick brown fox jumps over the lazy dog.
sed '=' data1.txt 
# 1
# The quick brown fox jumps over the lazy dog.
# 2
# The quick brown fox jumps over the lazy dog.
# 3
# The quick brown fox jumps over the lazy dog.
# 4
# The quick brown fox jumps over the lazy dog.
```

搜索匹配的内容并打印行号

```sh
sed -n '/number 4/{
> =
> p
> }' data6.txt
# 4
# This is line number 4.
```

l - listing lines, 打印文字和特殊字符(non-printable characters). 下面的实验中，tab 符号答应失败了，可能什么设置问题把，不过结尾符 `$` 倒是么什么问题

```sh
cat data9.txt 
# This    line    contains    tabs.
sed -n 'l' data9.txt 
# This    line    contains    tabs.$
```

#### Using files with sed

##### Writing to a file

通过 w 将匹配的内容写到文件 `[address]w filename`, 使用 -n 只在屏幕上显示匹配部分

```sh
sed '1,2w test.txt' data6.txt 
# This is line number 1.
# This is line number 2.
# This is line number 3.
# This is line number 4.
cat test.txt 
# This is line number 1.
# This is line number 2.
```

这个技巧在筛选数据的时候格外好用

```sh
cat data11.txt 
# Blum, R       Browncoat
# McGuiness, A  Alliance
# Bresnahan, C  Browncoat
# Harken, C     Alliance

sed -n '/Browncoat/w Browncoats.txt' data11.txt 
cat Browncoats.txt 
# Blum, R       Browncoat
# Bresnahan, C  Browncoat
```

##### Reading data from a file

The read command(r) allows you to insert data contained in a separate file. format at: `[address]r filename`

filename 可以是相对路径，也可以是绝对路径。你不能使用 range of address for the read command. you can only specify a single line number or text pattern address.

读取目标文件中的内容并插入到指定位置**的后面**

```sh
cat data12.txt 
# This is an added line.
# This is the second added line.

cat data6.txt 
# This is line number 1.
# This is line number 2.
# This is line number 3.
# This is line number 4.

sed '3r data12.txt' data6.txt 
# This is line number 1.
# This is line number 2.
# This is line number 3.
# This is an added line.
# This is the second added line.
# This is line number 4.
```

pattern 同样支持

```sh
sed '/number 2/r data12.txt' data6.txt 
# This is line number 1.
# This is line number 2.
# This is an added line.
# This is the second added line.
# This is line number 3.
# This is line number 4.
```

添加到末尾

```sh
sed '$r data12.txt' data6.txt 
# This is line number 1.
# This is line number 2.
# This is line number 3.
# This is line number 4.
# This is an added line.
# This is the second added line.
```

将 read 和 delete 结合使用，我们就可以有类似于替换的效果了

下面例子中我们将名单用 LIST 这个单词做为占位符，将 data11.txt 中的内容替换进去

```sh
cat notice.std 
# Would the following people:
# LIST
# please report to the ship's captain.

sed '/LIST/{
> r data11.txt
> d
> }' notice.std
# Would the following people:
# Blum, R       Browncoat
# McGuiness, A  Alliance
# Bresnahan, C  Browncoat
# Harken, C     Alliance
# please report to the ship's captain.
```

## Chapter 20: Regular Expressions

### What Are Regular Expressions

#### A definition

A regular expression is a pattern template you define that a Linux utility users to filter text.

#### Types of regular expressions

Linux 系统中，一些不同的应用采用不同的正则表达式。正则表达式通过 regular expression engine 实现。Linux 世界中有两个爆款 engines:

* The POSIX Basic Regular Expression(BRE) engine
* The POSIX Extened Regular Expression(ERE) engine

大多数 Linux 工具都会适配 RRE，sed 除外，它的目标是尽可能快的处理，所以只识别部分 BRE。

### Defining BRE Patterns

#### Plain text

搜索的关键字是目标的一部分即可

#### Special characters

正则可以识别的特殊字符 `.*[]^${}\+?|()`, 如果你想要使用文本版的这些特殊字符，在他们前面加 backslash character(\)

```sh
echo 'The cost is $4.00' | sed -n '/\$/p'
# The cost is $4.00
```

#### Anchor characters

##### Starting at the beginning

caret character(^) 指代文本开头

```sh
echo "The book store" | sed -n '/^book/p'
# no matched
echo "Books are great" | sed -n '/^Book/p'
# Books are great
```

当符号出现在非开头位置，则他会被当作普通字符处理

```sh
echo "This ^ is a test" | sed -n '/s ^/p' 
# This ^ is a test
```

##### Looking for the ending

The dollar sign($) 指代了文本的结尾

```sh
echo "This is a good book" | sed -n '/book$/p'
# This is a good book
```

#### The dot character

dot 用于指代除换行外的任何字符, 如果 `.` 代表的位置没有东西，则匹配失败

```sh
cat data6 
# This is a test of a line.
# The cat is sleeping.
# That is a very nice hat.
# This test is at line four.
# at ten o'clock we'll go home.
sed -n '/.at/p' data6
# The cat is sleeping.
# That is a very nice hat.
# This test is at line four.
```

#### Character classes

character class 用于限定匹配的内容，使用 square brackets([]) 表示

```sh
sed -n '/[ch]at/p' data6
# The cat is sleeping.
# That is a very nice hat.
```

#### Negating character classes

和前面的相反，是不包含的意思

```sh
sed -n '/[^ch]at/p' data6
# This test is at line four.
```

#### Using ranges

`sed -n '/^[0-9][0-9][0-9][0-9][0-9]$/p' data8` 这个技巧也使用于字符

```sh
sed -n '/[c-h]at/p' data6
# The cat is sleeping.
# That is a very nice hat.
```

也可以指定非连续的字符集合

```sh
sed -n '/[a-ch-m]at/p' data6
# The cat is sleeping.
# That is a very nice hat.
echo "I'm getting too fat" | sed -n '/[a-ch-m]at/p'
# no matched
```

#### Special character classes

BRE Special Character classes

|Class|Description|
|:---|:---|
|[[:alpha:]]|Matches any alphabetical character, either upper or lower case|
|[[:alnum:]]|Matches any alphanumberic character 0-9, A-Z or a-z|
|[[:blank:]]|Matches a space or Tab character|
|[[:digit:]]|Matches a numberical digit from 0-9|
|[[:lower:]]|Matches any lowercase alphabetical character a-z|
|[[:print:]]|Matches any printable character|
|[[:punct:]]|Matches a punctuation character|
|[[:space:]]|Matches any whitespace character: space, Table, NL, FF, VT CR|
|[[:upper:]]|Matches any uppercase alphabetical character A-Z|

```sh
echo "abc" | sed -n '/[[:digit:]]/p'
# no matched
echo "abc" | sed -n '/[[:alpha:]]/p'
# abc
echo "abc123" | sed -n '/[[:digit:]]/p'
# abc123
echo "This is, a test" | sed -n '/[[:punct:]]/p'
# This is, a test
echo "This is a test" | sed -n '/[[:punct:]]/p'
# no matched
```

#### The asterisk

星号标识字符出现一次或 n 次

```sh
echo "ik" | sed -n '/ie*k/p'
# ik
echo "iek" | sed -n '/ie*k/p'
# iek
echo "ieeeek" | sed -n '/ie*k/p'
# ieeeek
```

星号也可以和 character class 结合使用

```sh
echo "baeeeet" | sed -n '/b[ae]*t/p'
# baeeeet
```

### Extended Regular Expressions

gawk 识别 ERE pattern，ERE 新增了一些符号来扩展功能

**Caution** sed 和 gawk 采用了不同的引擎，gawk 可以适配大部分的扩展功能。sed 不能，但是 sed 更快

#### The question mark

问号(?)表示出现 0 次或 1 次。

```sh
"bt" | gawk '/be?t/{print $0}'
# bt
echo "bet" | gawk '/be?t/{print $0}'
# bet
echo "beet" | gawk '/be?t/{print $0}'
# no matched
```

问号也可以结合 character class 使用

```sh
echo "bt" | gawk '/b[ae]?t/{print $0}'
# bt
echo "bat" | gawk '/b[ae]?t/{print $0}'
# bat
echo "bot" | gawk '/b[ae]?t/{print $0}'
# no matched
echo "bet" | gawk '/b[ae]?t/{print $0}'
# bet
echo "beaet" | gawk '/b[ae]?t/{print $0}'
# no matched
echo "baet" | gawk '/b[ae]?t/{print $0}'
# no matched
echo "beat" | gawk '/b[ae]?t/{print $0}'
# no matched
echo "beet" | gawk '/b[ae]?t/{print $0}'
# no matched
```

#### The plus sign

加号(+), 出现一次或多次

```sh
echo "beet" | gawk '/be+t/{print $0}'
# beet
echo "bet" | gawk '/be+t/{print $0}'
# bet
echo "bt" | gawk '/be+t/{print $0}'
# no matched
```

结合方括号使用

```sh
echo "bt" | gawk '/b[ae]+t/{print $0}'
# no matched
echo "bat" | gawk '/b[ae]+t/{print $0}'
# bat
echo "bet" | gawk '/b[ae]+t/{print $0}'
# bet
echo "baet" | gawk '/b[ae]+t/{print $0}'
# baet
echo "beet" | gawk '/b[ae]+t/{print $0}'
# beet
echo "beeet" | gawk '/b[ae]+t/{print $0}'
# beeet
```

#### Using braces

花括号表示重复多次

* m: The regular expression appears exactly m times
* m,n: The regular expression appears at least m times, but no more than n times

**Cautim** gawk 默认不识别这个模式，需要加上 --re-interval 参数增加这个功能

```sh
echo "bt" | gawk --re-interval '/be{1}t/{print $0}'
# no matched
echo "bet" | gawk --re-interval '/be{1}t/{print $0}'
# bet
echo "beet" | gawk --re-interval '/be{1}t/{print $0}'
# no matched
```

指定出现次数的区间

```sh
echo "bt" | gawk --re-interval '/be{1,2}t/{print $0}'
# no matched
echo "bet" | gawk --re-interval '/be{1,2}t/{print $0}'
# bet
echo "beet" | gawk --re-interval '/be{1,2}t/{print $0}'
# beet
echo "beeet" | gawk --re-interval '/be{1,2}t/{print $0}'
# no matched
```

同样适用于 character class

```sh
echo "bt" | gawk --re-interval '/b[ae]{1,2}t/{print $0}'
# no matched
echo "bat" | gawk --re-interval '/b[ae]{1,2}t/{print $0}'
# bat
echo "bet" | gawk --re-interval '/b[ae]{1,2}t/{print $0}'
# bet
echo "beat" | gawk --re-interval '/b[ae]{1,2}t/{print $0}'
# beat
echo "beet" | gawk --re-interval '/b[ae]{1,2}t/{print $0}'
# beet
echo "beeet" | gawk --re-interval '/b[ae]{1,2}t/{print $0}'
# no matched
```

#### The pipe symbol

pipe symbol 可以让你实现 OR 的逻辑，只要有一个匹配，就算 match 了 `expr1 | expr2 | ...`

```sh
echo "The cat is asleep" | gawk '/cat|dog/{print $0}'
# The cat is asleep
echo "The dog is asleep" | gawk '/cat|dog/{print $0}'
# The dog is asleep
echo "The sheep is asleep" | gawk '/cat|dog/{print $0}'
# no matched
```

结合 character class 使用

```sh
echo "He has a hat" | gawk '/[ch]at|dog/{print $0}'
# He has a hat
```

#### Grouping expressions

可以使用括号(parentheses)表示 group. 这个 group 会被当成一个基本字符对待。

```sh
echo "Sat" | gawk '/Sat(urday)?/{print $0}'
# Sat
echo "Saturday" | gawk '/Sat(urday)?/{print $0}'
# Saturday
```

group 经常和 pipe 结合使用来表示可能出现的组合

```sh
echo "cat" | gawk '/(c|b)a(b|t)/{print $0}'
# cat
echo "cab" | gawk '/(c|b)a(b|t)/{print $0}'
# cab
echo "bat" | gawk '/(c|b)a(b|t)/{print $0}'
# bat
```

### Regular Expressions in Action

实操，介绍一些使用案例

#### Counting directory files

统计环境变量中的可执行文件数量

步骤：echo $PATH 拿到路径，通过 `:` 分割，最后通过 `ls` 列出文件

```sh
cat countfiles.sh                                     
#!/usr/local/bin/bash
# Count number of files in you PATH

mypath=$(echo $PATH | sed 's/:/ /g')
count=0
for directory in $mypath
do
    check=$(ls $direcotry)
    for item in $check
    do
        count=$[ $count + 1 ]
    done
    echo "$directory - $count"
    count=0
done

./countfiles.sh 
# /usr/local/opt/mysql@5.7/bin - 4
# /Users/i306454/SAPDevelop/tools/maven/bin - 4
# ....
```

#### Validating a phone number

写一个脚本匹配电话号码，样本

```txt
000-000-0000
123-456-7890
212-555-1234
(317)555-1234
(202) 555-9876
33523
1234567890
234.123.4567
```

规则：如下四种格式是合法的，其他都不合法

```txt
(123)456-7890
(123) 456-7890
123-456-7890
123.456.7890
```

找规律:

* 可能以括号开头 `^\(?`
* 接下来三位数是 area codes, 第一位是非 0，1的数，后两位是 0-9的数 `[2-9][0-9]{2}`
* 可能存在的结束括号 `\)?`
* 间隔符，可以没有，可以是空格，点，横线 `(| |-|\.)` 使用 group 把它看作一个集合，使用竖线表示 or
* 三个 0-9 的整数 `[0-9]{3}`
* 空格(虽然例子上没显示)，横线或者点号 `( |-|\.)`
* 最后接四位整数作为结尾 `[0-9]{4}$`

完整表达式 `^\(?[2-9][0-9]{2}\)?(| |-|\.)[0-9]{3}( |-|\.)[0-9]{4}$`

测试:

```sh
cat isphone              
#!/usr/local/bin/bash
# Script to filter out bad phone numbers

gawk --re-interval '/^\(?[2-9][0-9]{2}\)?(| |-|\.)[0-9]{3}( |-|\.)[0-9]{4}$/{print $0}'

cat phonelist | ./isphone       
# 212-555-1234
# (317)555-1234
# (202) 555-9876
# 234.123.4567
```

PS: 中间那个过来间隔符的操作我之前是没有意识到的

#### Parsing an e-mail address

验证 email 的正则表达式

* 用户名部分，可以是任何数字，字母，下划线，横杠和加号 `^([a-zA-Z0-9_\-\.\+]+)@`
* hostname 和名字部分一样的规则 `([a-zA-Z0-9_\-\.\+]+)`
* 顶级域名值只能是字母，大于2个字符，小于5个字符 `\.([a-zA-Z]{2,5})$`

完整表达式 `^([a-zA-Z0-9_\-\.\+]+)@([a-zA-Z0-9_\-\.\+]+)\.([a-zA-Z]{2,5})$`

测试

```sh
cat isemail 
#!/usr/local/bin/bash
# Script to filter out bad email

gawk --re-interval '/^([a-zA-Z0-9_\-\.\+]+)@([a-zA-Z0-9_\-\.\+]+)\.([a-zA-Z]{2,5})$/{print $0}'

echo "rich@here.now" | ./isemail
# rich@here.now
echo "rich@here.now." | ./isemail
# no match
echo "rich.blum@here.now" | ./isemail
# rich.blum@here.now
```

## Advanced sed

### Looking at Multiline Commands

在前面的 sed 使用过程中，你可能已经察觉到了 sed 的一个限制，他只能按行处理。当 sed 拿到一个字符流时，他会将数据按照 newline characters 做分割，每次处理一行。

但是实际工作你总会遇到需要处理多行的情况，比如你要替换文件中的 `Linux System Administrators Group` 关键字，但是他可能分布在两行中，这时如果你安之前的 sed 做替换就会漏掉一些内容

为了应对这种情况，sed 提供了三个关键字来处理这种情况

* N add the next line in the data stream to create a multiline group for processing
* D delete a single line in multiline group
* P prints a single line in a multiline group

#### Navigating the next command

##### Using the single-line next command

下面的示例中，我们有5行文本，1，3，5有值，2，4为空。目标是通过 sed 只移除第二行

```sh
cat data1.txt
# This is the header line.

# This is a data line.

# This is the last line.
```

错误示范，会删掉所有空行

```sh
sed '/^$/d' data1.txt       
# This is the header line.
# This is a data line.
# This is the last line.
```

通过使用 `n` 这个关键字，可以将下一行也包括到搜索范围内

```sh
sed '/header/{n ; d}' data1.txt
# This is the header line.
# This is a data line.

# This is the last line
```

PS: MacOS 不支持，在 Ubantu 上做的实验

简单一句话就是，n 不会和前一句做合并处理. 说实话上面的例子还是不怎么理解，可能得另外找点书补充一下

##### Combining lines of text

The single-line next command moves the next line of text from the data stream into the processing space(called the pattern space) of the sed editor.

The multiline version of the next command(which uses a captial N) adds the next line of text to the text already in the pattern space.

大写的 N 可以将两行拼成一行处理，中间用换行符隔开

```sh
cat data2.txt
# This is the header line.
# This is the first data line.
# This is the second data line
# This is the last line

sed '/first/{N; s/\n/ /}' data2.txt
# This is the header line.
# This is the first data line. This is the second data line
# This is the last line
```

上面的例子中，我们找到包含 first 的行，然后将下一行接上一起处理，处理的时候，将换行替换为空格

再举一个例子

```sh
cat data3.txt
# On Tuesday, the Linux System
# Administrator's group meeting will be held.
# All System Administrators should attend.
# Thank you for your attendance.
```