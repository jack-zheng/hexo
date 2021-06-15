---
title: Linux命令行与shell脚本编程大全 第三章 Shell 高级用法
date: 2021-06-05 16:38:52
categories:
- Shell
tags:
- Linux命令行与shell脚本编程大全 3rd
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

Mac 自带的 sed 工具和书上的是不一样的，好像做了很多裁剪，很多 flag 是不支持的，可以通过 brew 重新装一个

```sh
brew install gnu-sed
# 会给出添加 PATH 的提示，按照提示添加到配置文件中(.zshrc)
brew info gnu-sed
# ==> Caveats
# GNU "sed" has been installed as "gsed".
# If you need to use it as "sed", you can add a "gnubin" directory
# to your PATH from your bashrc like:

#     PATH="/usr/local/opt/gnu-sed/libexec/gnubin:$PATH"
```

#### Getting to know the sed editor

sed 是一个流处理编辑器(stream editor)，你可以设定一系列的规则，然后通过这个流编辑器处理他。

sed 可以做如下事情

1. Reads one data line at a time from the input
2. Matches that data with the supplied editor commands
3. Changes data in the stream as specified in the commands
4. Outputs the new data to STDOUT

按行依次处理文件直到所有内容处理完毕结束，格式 `sed options script file`

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

和 gawk 一样，sed 也可以在一个命令中处理多个匹配

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

特别的点在于，你需要新起一行写这些新加的行, 格式为

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

p 使用案例, -n 可以强制只打印匹配的内容

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

| Class       | Description                                                    |
| :---------- | :------------------------------------------------------------- |
| [[:alpha:]] | Matches any alphabetical character, either upper or lower case |
| [[:alnum:]] | Matches any alphanumberic character 0-9, A-Z or a-z            |
| [[:blank:]] | Matches a space or Tab character                               |
| [[:digit:]] | Matches a numberical digit from 0-9                            |
| [[:lower:]] | Matches any lowercase alphabetical character a-z               |
| [[:print:]] | Matches any printable character                                |
| [[:punct:]] | Matches a punctuation character                                |
| [[:space:]] | Matches any whitespace character: space, Table, NL, FF, VT CR  |
| [[:upper:]] | Matches any uppercase alphabetical character A-Z               |

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

再举一个需要测试的数据落在两个段落中的例子

```sh
cat data3.txt
# On Tuesday, the Linux System
# Administrator's group meeting will be held.
# All System Administrators should attend.
# Thank you for your attendance.
```

第一个关键字替换失败，第二个成功，因为第一个用的换行，匹配用的空格

```sh
sed 'N;s/System Administrator/Desktop User/' data3.txt
# On Tuesday, the Linux System
# Administrator's group meeting will be held.
# All Desktop Users should attend.
# Thank you for your attendance
```

替换成功不过换行消失了

```sh
sed 'N;s/System.Administrator/Desktop User/' data3.txt
# On Tuesday, the Linux Desktop User's group meeting will be held.
# All Desktop Users should attend.
# Thank you for your attendance.
```

使用两个替换分别应对换行和空格的情况

```sh
sed 'N
> s/System\nAdministrator/Desktop\nUser/
> s/System Administrator/Desktop User/
> ' data3.txt
# On Tuesday, the Linux Desktop
# User's group meeting will be held.
# All Desktop Users should attend.
# Thank you for your attendance.
```

这里还有一个小问题，由于命令是 N 开头，他会先拿下一行到 pattern space，当处理最后一行时，下一行为空，直接结束了，如果要替换的目标在最后一行就会有遗漏

```sh
cat data4.txt
# On Tuesday, the Linux System
# Administrator's group meeting will be held.
# All System Administrators should attend.

sed 'N
s/System\nAdministrator/Desktop\nUser/
s/System Administrator/Desktop User/
' data4.txt
# On Tuesday, the Linux Desktop
# User's group meeting will be held.
# All System Administrators should attend.
```

这时你可以换一下顺序

```sh
sed '
> s/System Administrator/Desktop User/
> N
> s/System\nAdministrator/Desktop\nUser/
> ' data4.txt
# On Tuesday, the Linux Desktop
# User's group meeting will be held.
# All Desktop Users should attend.
```

（；￣ェ￣）简直无情，太繁琐了

#### Navigating the multiline delete command

当使用 N 的方式做 delete 的时候，它会将匹配到的两行内容全部删掉

```sh
cat data4.txt
# On Tuesday, the Linux System
# Administrator's group meeting will be held.
# All System Administrators should attend.

sed 'N ; /System\nAdministrator/d' data4.txt
# All System Administrators should attend.
```

sed 提供了一个只删除第一行内容的 flag - D

```sh
sed 'N ; /System\nAdministrator/D' data4.txt
# Administrator's group meeting will be held.
# All System Administrators should attend.
```

类似的技巧可以用来删除文章开头的空行

```sh
cat -n data1.txt
    #  1
    #  2	This is the header line.
    #  3
    #  4	This is a data line.
    #  5
    #  6	This is the last line

sed '/^$/{N;/header/D}' data1.txt | cat -n
    #  1	This is the header line.
    #  2
    #  3	This is a data line.
    #  4
    #  5	This is the last line
```

#### Navigating the multiline print command

和 p 对应的还有一个 P， 用法和上面的 D 一样，如果用 p 会打印两行，而用 P 则只打印第一行

```sh
sed -n 'N ; /System\nAdministrator/P' data3.txt
# On Tuesday, the Linux System
sed -n 'N ; /System\nAdministrator/p' data3.txt
# On Tuesday, the Linux System
# Administrator's group meeting will be held.
```

### Holding Space

pattern space 是 sed 用于存放正在的处理文本的空间。但是这并不是存放文本的唯一的地方，还有一个叫做 hold space. 下列是五个可以操作 hold space 的命令

The sed Editor Hold Space Commands

| Command | Description                                   |
| :------ | :-------------------------------------------- |
| h       | Copies pattern space to hold space            |
| H       | Appends pattern space to hold space           |
| g       | Copies hold space to pattern sapce            |
| G       | Appends hold space to pattern space           |
| x       | Exchanges contents of pattern and hold spaces |

这些命令可以让 pattern space 空出来处理其他文本。一般来说，你在通过 h/H 将 pattern space 的内容移动到 hold space 之后，都会再通过 g/G/x 将内容在放回到 pattern space 中。

```sh
cat data2.txt
# This is the header line.
# This is the first data line.
# This is the second data line
# This is the last line

sed -n '/first/ {h ; p ; n ; p ; g ; p }' data2.txt
# This is the first data line.
# This is the second data line
# This is the first data line.
```

解析上面的命令

1. sed 通过 RE 过滤包含 first 的语句
2. 匹配到目标语句后，开始执行 {} 中的内容，h 会将语句 copy 到 hold space 中
3. 第一个 p 打印当前 pattern space 中内容
4. n 提取下一行内容并放到 pattern space
5. 第二个 p 打印当前 pattern space 中内容, 即包含 second 的语句
6. g 将 hold space 中的内容再 copy 回去
7. 第三个 p 打印当前 pattern space 中内容, 即包含 first 的语句

### Negating a Command

使用叹号(!)对操作取反

```sh
cat data2.txt
# This is the header line.
# This is the first data line.
# This is the second data line
# This is the last line
sed -n '/header/p' data2.txt
# This is the header line.
sed -n '/header/!p' data2.txt
# This is the first data line.
# This is the second data line
# This is the last line
```

N 也有取反操作, 之前的例子

```sh
sed 'N
s/System\nAdministrator/Desktop\nUser/
s/System Administrator/Desktop User/
' data4.txt
# On Tuesday, the Linux Desktop
# User's group meeting will be held.
# All System Administrators should attend.

sed '$!N
> s/System\nAdministrator/Desktop\nUser/
> s/System Administrator/Desktop User/
> ' data4.txt
# On Tuesday, the Linux Desktop
# User's group meeting will be held.
# All Desktop Users should attend.
```

`$!N` 消失 最后一行不执行 N 的操作。。。

通过上面介绍的这些技巧，你可以利用 hold space 做文本倒序的功能

1. Place a line in the pattern space
2. Place the line from the pattern space to the hold space
3. Put the next line of text in the pattern space
4. Append the hold space to the pattern space
5. Place everything in the pttern space into the hold space
6. Repeat step 3-5 until you've put all the lines in reverse oder in the hold space
7. Retrieve the lines, and print them

```sh
cat -n data2.txt
    #  1	This is the header line.
    #  2	This is the first data line.
    #  3	This is the second data line
    #  4	This is the last line

sed -n '{1!G; h; $p}' data2.txt | cat -n
    #  1	This is the last line
    #  2	This is the second data line
    #  3	This is the first data line.
    #  4	This is the header line.
```

* `1!G` 第一行时不用将 hold space 的内容 append 过来, 不加的话会多一个空行
*  `h` copy to hold space
*  `$p` 最后一行的话 打印

这尼玛也太精巧了把，我感觉我想不出来 （；￣ェ￣）

PS: 如果真要倒序，直接用 tac 即可， cat 的倒写

### Changing the Flow

默认情况下 sed 是从头到尾的处理的，但是他也提供了方法改变处理顺序，感觉像有点像结构化语言

#### Branching

效果和叹号一样，只不过他是会根据 address 的标识批量操作而已

branch command: [address]b [label]

下面的例子中， sed 在做替换是根据 `2,3b` 跳过了第 2-3 行

```sh
cat data2.txt
# This is the header line.
# This is the first data line.
# This is the second data line.
# This is the last line.

sed '{2,3b; s/This is/Is this/; s/line./test?/}' data2.txt
# Is this the header test?
# This is the first data line.
# This is the second data line.
# Is this the last test?
```

label 的作用是设置一个跳点，本来看了第一个例子我还想说它很像 if condition 但是感觉上说他是 goto 还更恰当一点。 label 最长为 7 个字符

下面的例子，jump1 更像是 if, 如果 match 则跳过条件直接执行 :jump1 之后的命令

```sh
cat data2.txt
# This is the header line.
# This is the first data line.
# This is the second data line.
# This is the last line.

sed '{/first/b jump1; s/This is the/No jump on/
> :jump1
> s/This is the/Jump here on/}' data2.txt
# No jump on header line.
# Jump here on first data line.
# No jump on second data line.
# No jump on last line.
```

当 b 匹配的内容出现，则跳过第一个替换，直接执行后一个。更骚的操作是下面的这个循环替换逗号的操作

```sh
echo "This, is, a, test, to, remove, commas." | sed -n '{
> :start
> s/,//1p
> b start
> }'
# This is, a, test, to, remove, commas.
# This is a, test, to, remove, commas.
# This is a test, to, remove, commas.
# This is a test to, remove, commas.
# This is a test to remove, commas.
# This is a test to remove commas.
# ^C
```

这个例子大致意思我是懂得，但是不清楚为什么执行操作的时候文本一直有效，不会被冲掉吗？可能要深入了解一下 pattern space 才能直到原因。这个cmd 需要 Ctrl + C 才能强制结束, 下面是改进版本

```sh
echo "This, is, a, test, to, remove, commas." | sed -n '{
:start
s/,//1p
/,/b start
}'
# This is, a, test, to, remove, commas.
# This is a, test, to, remove, commas.
# This is a test, to, remove, commas.
# This is a test to, remove, commas.
# This is a test to remove, commas.
# This is a test to remove commas.
```

#### Testing

语法和 branch 很像 `[address]t [label]`

test command provide a cheap way to perform a basic if-then statement on the text in the data stream

```sh
cat data2.txt
# This is the header line.
# This is the first data line.
# This is the second data line.
# This is the last line.
sed '{
> s/first/matched/
> t
> s/This is the/No match on/
> }' data2.txt
# No match on header line.
# This is the matched data line.
# No match on second data line.
# No match on last line.
```

如果 t 前面的 cmd 匹配则执行，不然直接执行后一个命令。之前循环替换逗号的例子用 t 的形式

```sh
echo "This, is, a, test, to, remove, commas." | sed -n '{
:start
s/,//1p
t start
}'
# This is, a, test, to, remove, commas.
# This is a, test, to, remove, commas.
# This is a test, to, remove, commas.
# This is a test to, remove, commas.
# This is a test to remove, commas.
# This is a test to remove commas.
```

### Replacing via a Pattern

通过 sed 做精确替换还是简单的, 比如下面的例子要在 cat 外面添加双引号

```sh
echo "The cat sleeps in his hat." | sed 's/cat/"cat"/'
# The "cat" sleeps in his hat.
```

但是如果你想要在所有 .at 外面加双引号可能有会遇到问题了

```sh
echo "The cat sleeps in his hat." | sed 's/.at/".at"/g'
# The ".at" sleeps in his ".at".
```

#### Using the ampersand

为了解决上面的问题，sed 提供了 `&` 符号指代匹配的字符

```sh
echo "The cat sleeps in his hat." | sed 's/.at/"&"/g'
# The "cat" sleeps in his "hat".
```

#### Replacing individual words

如果你只想替换一部分内容，说人话就是支持 group 的模式减少 typing

```sh
echo "This System Administractor manual" | sed '
s/\(System\) Administractor/\1 User/'
# This System User manual
```

* group 需要用反斜线
* 指代 group 用反斜线加数子

下面的例子中我们用原句中的一部分代替原有部分

```sh
echo "That furry cat is pretty" | sed 's/furry \(.at\)/\1/'
# That cat is pretty
```

这个技巧在插入值的时候很好用

```sh
echo "1234567" | sed '{                                    
:start
s/\(.*[0-9]\)\([0-9]\{3\}\)/\1,\2/
t start
}'
# 1,234,567
```

有两个分组

* .*[0-9]
* [0-9]{3}

第一次替换结果为 1234,567，第二次 1,234,567

### Placing sed Commands in Script

展示一些脚本中使用 sed 的技巧

#### Using wrappers

每次使用 sed 的时候现打会很累赘，你可以将他们写到脚本中并调用

下面的例子中，我们将之前实现的 reverse 功能通过脚本调用

```sh
cat reverse.sh
#!/bin/bash
# Shell wrapper for sed editor script.
#           to reverse text file lines
sed -n '{1!G; h; $p}' $1

cat data2.txt
# This is the header line.
# This is the first data line.
# This is the second data line.
# This is the last line.

./reverse.sh data2.txt
# This is the last line.
# This is the second data line.
# This is the first data line.
# This is the header line.
```

#### Redirecting sed output

sed 操作后的输出可以用 $() 包裹起来作为结果引用

下面的例子中我们计算斐波那契额数列并用之前写的 sed 表达式为它加上逗号分割

```sh
cat fact.sh                        
#!/usr/local/bin/bash
# Add commas to number in factorial answer

factorial=1
counter=1
number=$1
#
while [ $counter -le $number ]
do 
    factorial=$[ $factorial * $counter ]
    counter=$[ $counter + 1 ]
done
#
result=$(echo $factorial | sed '{
:start
s/\(.*[0-9]\)\([0-9]\{3\}\)/\1,\2/
t start
}')
#
echo "The result is $result"
#

./fact.sh 20      
# The result is 2,432,902,008,176,640,000
```

### Creating sed Utilities 

分享一些数据处理函数

#### Spacing with double lines

为文本中的每一行后面新家一个空行, 如果最后一行不想加空格，可以用叹号取反

```sh
sed 'G' data2.txt  | cat -n
    #  1	This is the header line.
    #  2
    #  3	This is the first data line.
    #  4
    #  5	This is the second data line.
    #  6
    #  7	This is the last line.
    #  8
 sed '$!G' data2.txt  | cat -n
    #  1	This is the header line.
    #  2
    #  3	This is the first data line.
    #  4
    #  5	This is the second data line.
    #  6
    #  7	This is the last line.
```

#### Spacing files that may have blanks

如果你的文件中已经存在空行了，那么用上面的技巧，你的文件中可能出现多个空行，怎么保证空行数只有一个呢

```sh
sed '$!G' data6.txt | cat -n
    #  1	This is line number1.
    #  2
    #  3	This is line number2.
    #  4
    #  5
    #  6
    #  7	This is line number3.
    #  8
    #  9	This is line number4.
```

解决方案，现将所有空格去了，再做加空行的操作

```sh
sed '/^$/d; $!G;' data6.txt  | cat  -A
# This is line number 1.$
# $
# This is line number 2.$
# $
# This is line number 3.$
# $
# This is line number 4.$
```

PS: 使用 `i\` 的语法添加空行会带有一个空格，推荐使用 `1G` 的方式

#### Numbering lines in a file


19 章时我们介绍过用 `=` 显示行号的操作

```sh
sed '=' data2.txt 
1
This is the header line.
2
This is the first data line.
3
This is the second data line.
4
This is the last line.
```

这种格式有点奇怪，更友好的方式应该时行号和字串在一行中，这里可以用 N

```sh
sed '=' data2.txt | sed 'N; s/\n/ /'
# 1 This is the header line.
# 2 This is the first data line.
# 3 This is the second data line.
# 4 This is the last line.
```

这中方式最大的好处是，没有加行额外的空格，一些其他的工具，比如 nl, cat -n 会在结果前面添加一些空格


```sh
nl data2.txt
    #  1	This is the header line.
    #  2	This is the first data line.
    #  3	This is the second data line.
    #  4	This is the last line.

cat -n data2.txt
    #  1	This is the header line.
    #  2	This is the first data line.
    #  3	This is the second data line.
    #  4	This is the last line.
```

#### Printing last lines

只打印最后一行

```sh
sed -n '$p' data2.txt
# This is the last line.
```

使用类似的技巧，你可以显示末尾几行数据，这种做法叫做 rolling window

rolling window 中我们结合使用 N，将整块的文本存储到 pattern space 中

下面的例子中我们将用 sed 显示文本最后10行的内容

```sh
cat data7.txt
# This is line 1.
# This is line 2.
# This is line 3.
# This is line 4.
# This is line 5.
# This is line 6.
# This is line 7.
# This is line 8.
# This is line 9.
# This is line 10.
# This is line 11.
# This is line 12.
# This is line 13.
# This is line 14.
# This is line 15.

sed '{
:start
$q; N; 11,$D
b start
}' data7.txt
# This is line 6.
# This is line 7.
# This is line 8.
# This is line 9.
# This is line 10.
# This is line 11.
# This is line 12.
# This is line 13.
# This is line 14.
# This is line 15.
```

* `$q` 退出
* `N` 将下一行 append 到 pattern space
*  `11,$D` 如果当前为 10 行以后，删除第一行

#### Deleting lines

这节将介绍一些快速移除空白行的操作

##### Deleting consecutive blank lines

删除多余空行，这里用了另外一种解决方案。这里的规则是，起始于任何非空行，终止于空行的内容都不会被删除

```sh
cat -n data8.txt   
    #  1  This is the header line.
    #  2
    #  3
    #  4  This is the first data line.
    #  5
    #  6  This is the second data line.
    #  7
    #  8
    #  9  This is the last line.
    # 10

sed '/./,/^$/!d' data8.txt | cat -n
    #  1  This is the header line.
    #  2
    #  3  This is the first data line.
    #  4
    #  5  This is the second data line.
    #  6
    #  7  This is the last line.
    #  8
```

##### Deleting leading blank lines

删除开头部分的空行

```sh
cat -n data9.txt 
    #  1
    #  2
    #  3  This is line one.
    #  4
    #  5  This is line two.

sed '/./,$!d' data9.txt | cat -n
    #  1  This is line one.
    #  2
    #  3  This is line two.
```

任何有内容的行开始，到结束不删除

##### Deleting trailing blank lines

删除结尾部分的空行要比删除开头部分的空行麻烦一点，需要一些技巧和循环

```sh
cat -n data10.txt
    #  1	This is the first line.
    #  2	This is the second line.
    #  3
    #  4
    #  5

sed '{
:start
/^\n*$/{$d; N; b start}
}' data10.txt | cat -n
    #  1	This is the first line.
    #  2	This is the second line.
```

匹配任何只包含换行的 line, 如果是最后一行，则删除，如果不是就再次执行

#### Removing HTML tags

```sh
cat data11.txt
# <html>
# <head>
# <title>This is the page title</title>
# </head>
# <body>
# <p>
# This is the <b>first</b> line in the Web page.
# This should provide some <i>useful</i>
# information to use in our sed script.
# </p>
# </body>
# </html>
```

如果直接使用 `s/<.*>//g` 会出问题，一些文本类似 `<b>abc</b>` 也会被删除

```sh
sed 's/<.*>//g' data11.txt | cat -n 
    #  1
    #  2   
    #  3    
    #  4   
    #  5   
    #  6    
    #  7     This is the  line in the Web page.
    #  8     This should provide some 
    #  9     information to use in our sed script.
    # 10    
    # 11   
    # 12
```

这个是由于 sed 将内嵌的 `>` 识别为 `.*` 的一部分了，可以使用 `s/<[^>]*>//g` 修复, 再结合删除空格的语法

```sh
sed 's/<[^>]*>//g; /^$/d' data11.txt             
# This is the page title
# This is the first line in the Web page.
# This should provide some useful
# information to use in our sed script.
```

## Chapter 22: Advanced gawk

### Using Variables

gawk 支持两种不同类型的变量

* Built-in variables
* User-defined variables

#### Built-in variables

这一节将展示 gawk 自带变量的使用办法

##### The field and record separator variables

The gawk Data Field and REcord Variables

| Variable    | Description                                                                              |
| :---------- | :--------------------------------------------------------------------------------------- |
| FIELDWIDTHS | A space-separated list of numbers defining the exact width(in spaces) of each data field |
| FS          | Input field separator character                                                          |
| RS          | Input record separator character                                                         |
| OFS         | Output field separator character                                                         |
| ORS         | Output record separator character                                                        |

下面的例子展示了 FS 的使用方法, 通过 FS 指定分割符，只输出每行前三组数据

```sh
cat data1
# data11,data12,data13,data14,data15
# data21,data22,data23,data24,data25
# data31,data32,data33,data34,data35

gawk 'BEGIN{FS=","} {print $1, $2, $3}' data1
# data11 data12 data13
# data21 data22 data23
# data31 data32 data33
```

OFS 指定输出时候的分割符

```sh
gawk 'BEGIN{FS=","; OFS="--"} {print $1, $2, $3}' data1
# data11--data12--data13
# data21--data22--data23
# data31--data32--data33
```

有些数据并不是用固定的分割符做刷剧划分的，而是用的位置，这个时候你就要用到 FIELDWIDTHS 了

```sh
cat data1b                                                               
# 1005.3247596.37
# 115-2.349194.00
# 05810.1298100.1

gawk 'BEGIN{FIELDWIDTHS="3 5 2 5"} {print $1, $2, $3 $4}' data1b
# 100 5.324 7596.37
# 115 -2.34 9194.00
# 058 10.12 98100.1
```

PS: FIEDLWIDTHS 必须是常数，变量是不支持的

RS/ORS 用于行数据，默认的 RS 即为换行符

下面是一个解析电话号码的例子，我们想要解析出用户和对应的电话号码

```sh
cat data2    
# Riley Mullen
# 123 Main Street
# Chicago, IL  60601
# (312)555-1234

# Frank Williams
# 456 Oak Street
# Indianapolis, IN  46201
# (317)555-9876

# Haley Snell
# 4231 Elm Street
# Detroit, MI 48201
# (313)555-4938
```

如果我们还是用默认的换行符，则不能解析。我们可以将 `\n` 设置为字段分割符，将空行作为行分割符

```sh
gawk 'BEGIN{FS="\n"; RS=""} {print $1, $4}' data2 
# Riley Mullen (312)555-1234
# Frank Williams (317)555-9876
# Haley Snell (313)555-4938
```

##### Data variables

More gawk Built-In Variables

| Variable | Description                                                                                |
| :------- | :----------------------------------------------------------------------------------------- |
| ARGC     | The number of command line parameters present                                              |
| ARGIND   | The index in ARGV of the current file being proecssed                                      |
| ARGV     | An array of command line parameters                                                        |
| CONVFMT  | The conversion format for numbers(see the printf statement), with a default value of %.6 g |
| ENVIRON  | An associative array of the current shell environment variables and their values           |
| FNR      | The current record number in the data file                                                 |
| NF       | The total number of data fields in the data file                                           |
| NR       | The number of input records processed                                                      |

其他懒得打了

ARGC, ARGV 和之前 shell 变量概念很想，不过 ARGV 是不会将 script 统计在内的，这个有点不一样

```sh
gawk 'BEGIN{print ARGC, ARGV[0], ARGV[1]}' data1
# 2 gawk data1
```

PS: 表达式也有点不一样，变量前不需要加 $

获取环境变量

```sh
gawk '                                          
quote> BEGIN{
quote> print ENVIRON["HOME"]                    
quote> print ENVIRON["PATH"]                    
quote> }'
# /Users/i306454
# ...
```

FNR, NF, NR 可以标记 field 的位置。 NF 可以让你在不清楚 field 数量的情况下处理最后一个 field

```sh
gawk 'BEGIN{FS=":"; OFS="--"} {print $1,$NF}' /etc/passwd
# _nearbyd--/usr/bin/false
# ...
cat /etc/passwd | tail -3                                
# _coreml:*:280:280:CoreML Services:/var/empty:/usr/bin/false
# _trustd:*:282:282:trustd:/var/empty:/usr/bin/false
# _oahd:*:441:441:OAH Daemon:/var/empty:/usr/bin/false
```

FNR 表示当前 field 的序号. 下面的例子中，我们传入两个 data file, 每个文件处理完后 FNR 会重制

```sh
gawk 'BEGIN{FS=","}{print $1, "FNR="FNR}' data1 data1    
# data11 FNR=1
# data21 FNR=2
# data31 FNR=3
# data11 FNR=1
# data21 FNR=2
# data31 FNR=3
```

NR 则是将所有的传入数据一起统计的

```sh
gawk 'BEGIN{FS=","}                            
quote> {print $1, "FNR="FNR, "NR="NR}
quote> END{print "There were", NR, "records processed"}' data1 data1
# data11 FNR=1 NR=1
# data21 FNR=2 NR=2
# data31 FNR=3 NR=3
# data11 FNR=1 NR=4
# data21 FNR=2 NR=5
# data31 FNR=3 NR=6
# There were 6 records processed
```

单个文件处理时 FNR 和 NR 是一致的，多个文件处理时不一样

#### User-defined variables

gawk 自第一变量不能以数字开头，大小写敏感

##### Assigning variables in scripts

使用方式了 shell 中一致, 下面例子中我们将数字，字符赋值给变量，而且支持计算

```sh
gawk '             
quote> BEGIN{
quote> testing="This is a test"
quote> print testing                            
quote> testing=45        
quote> print testing                            
quote> }'
# This is a test
# 45
gawk 'BEGIN{x=4; x=x*2+3; print x}'
# 11
```

##### Assigning variables on the command line

支持从终端接受参数的形式

```sh
cat script1               
# BEGIN{FS=","}
# {print $n}

gawk -f script1 n=2 data1                                      
# data12
# data22
# data32
gawk -f script1 n=3 data1
# data13
# data23
# data33
```

开起来挺好的，但是这里有一个问题，终端传入的参数，BEGIN 里面是访问不到的

```sh
cat script2                    
# BEGIN{print "The starting value is", n; FS=","}
# {print $n}

gawk -f script2 n=3 data1
# The starting value is 
# data13
# data23
# data33
```

你可以用 -v 参数解决这个问题

```sh
gawk -v n=3 -f script2 data1
# The starting value is 3
# data13
# data23
# data33
```

### Working with Arrays

和很多其他语言一样，gawk 提供了 array 相关的功能，叫做 associative arrays. 个人感觉可以叫做 map, 行为方式是根据键拿值

#### Defining array variables

格式 `var[index] = element`

```sh
gawk 'BEGIN{
captial["Illinois"] = "Springfield"
print captial["Illinois"]
}'
# Springfield
```

对数字也有效

```sh
gawk 'BEGIN{
quote> var[1] = 34                         
quote> var[2] = 3                          
quote> total = var[1] + var[2]
quote> print total                              
quote> }'
# 37
```

#### Iterating through array variables

gawk 中遍历 map 的语法

```sh
for (var in arry)
{
    statements
}
```

遍历 map 的示例

```sh
gawk 'BEGIN{
quote> var["a"] = 1                        
quote> var["g"] = 2
quote> var["m"] = 3
quote> var["u"] = 4
quote> for (test in var)            
quote> {
quote> print "index:", test, " - value:", var[test]
quote> }
quote> }'
# index: u  - value: 4
# index: m  - value: 3
# index: a  - value: 1
# index: g  - value: 2
```

背后的实现和 hash 一样，不保证顺序

#### Deleting array variables

语法：delete array[index]

```sh
gawk 'BEGIN{
var["a"] = 1
var["g"] = 2
for (test in var)
{
    print "Index:", test, " - Value:", var[test]
}
delete var["g"]             
print "---"                              
for (test in var)   
    print "Index:", test, " - Value:", var[test]
}'
# Index: a  - Value: 1
# Index: g  - Value: 2
# ---
# Index: a  - Value: 1
```

PS: 原来他也支持单行 for 循环吗。。。

### Using Patterns

本章介绍如何定义 pattern

#### Regular expressions

gawk 同时支持 BRE 和 ERE，正则要保证出现在 program script 之前

```sh
gawk 'BEGIN{FS=","} /11/{print $1}' data1    
# data11

gawk 'BEGIN{FS=","} /,d/{print $1}' data1
# data11
# data21
# data31
```

#### The matching operator

gawk 使用波浪线(~)表示匹配的动作，格式 `$1 ~ /^data/`. 下面的例子中，我们适配所有出现在 $2 这个位置上的 field， 以 data2 开头的即使我们寻在的目标

```sh
gawk 'BEGIN{FS=","} $2 ~ /^data2/{print $0}' data1
# data21,data22,data23,data24,data25
```

这个技巧在 gawk 中经常被用到, 下面是在 passwd 文件中寻找包含 root 关键字的行

```sh
gawk -F: '$1 ~/root/{print $1, $NF}' /etc/passwd               
# root /bin/sh
# _cvmsroot /usr/bin/false
```

这个操作还支持取反 `$1 !~ /expression/`

```sh
gawk 'BEGIN{FS=","} $2 !~ /^data2/{print $0}' data1
# data11,data12,data13,data14,data15
# data31,data32,data33,data34,data35
```

#### Mathematical expressions

gawk 还支持直接在表达式中做计算的, 下面的例子中我们统计 group 等于 0 的用户

```sh
gawk -F: '$4 == 0{print $1}' /etc/passwd           
# root
```

支持的算数表达式

* x == y: Value x is equal to y
* x <= y
* x < y
* x >=y
* x > y

`==` 也可以用于文字表示式，但是表示的是精确匹配

```sh
gawk -F, '$1 == "data"{print $1}' data1  
# no match
gawk -F, '$1 == "data11"{print $1}' data1
# data11
```

### Structured Commands

结构化脚本

#### The if statement

支持 if-then-else 语法

```sh
if (condition)
    statement1

# 一行的格式也 OK
if (condition) statement1
```

```sh
cat data4 
# 10
# 5
# 13
# 50
# 34
gawk '{if ($1 > 20) print $1}' data4               
# 50
# 34
```

如果 if 有多个条件，需要用花括号包裹

```sh
gawk '{                             
quote> if ($1 > 20)             
quote> { 
quote> x = $1 * 2                              
quote> print x                                  
quote> }
quote> }' data4
# 100
# 68
```

带 else 的例子

```sh
gawk '{
quote> if ($1 > 20)             
quote> {
quote> x = $1 * 2                              
quote> print x                                  
quote> } else 
quote> {
quote> x = $1 / 2                              
quote> print x                                  
quote> }}' data4
# 5
# 2.5
# 6.5
# 100
# 68
```

也可以写在一行, 格式 `if (condition) statement1; else statement2`

```sh
gawk '{if ($1 > 20) print $1 * 2; else print $1 /2}' data4
# 5
# 2.5
# 6.5
# 100
# 68
```

#### The while statement

格式：

```sh
while (condition)
{
    statements
}
```

```sh
cat data5 
# 130 120 135
# 160 113 140
# 145 170 215

gawk '{
total = 0
i = 1
while (i<4)
{
total += $i
i++
}
avg = total / 3
print "Average:", avg
}' data5
# Average: 128.333
# Average: 137.667
# Average: 176.667
```

支持 break，continue 打断循环

```sh
gawk '{
quote> total = 0      
quote> i = 1   
quote> while (i<4)                                  
quote> {
quote>      total += $i    
quote>      if (i == 2)              
quote>           break             
quote>      i++     
quote> }
quote> avg = total/2                                                          
quote> print "The average of the first two data is:" , avg
quote> }' data5
# The average of the first two data is: 125
# The average of the first two data is: 136.5
# The average of the first two data is: 157.5
```

#### The do-while statement

格式：

```sh
do
{
    statemnets
} while (condition)
```

```sh
gawk '{
quote> total = 0      
quote> i = 1   
quote> do                          
quote> {
quote> total += $i    
quote> i++     
quote> } while (total < 150)
quote> print total }' data5                     
# 250
# 160
# 315
```

#### The for statement

格式 `for( variable assignment; condition; iteration process)`

```sh
gawk '{
quote> total = 0      
quote> for  (i=1; i<4; i++)
quote> {
quote> total += $i    
quote> }
quote> avg = total /3                                                           
quote> print "Average:", avg                    
quote> }' data5
# Average: 128.333
# Average: 137.667
# Average: 176.667
```

### Formatted Printing

gawk 使用 printf 格式化输出 `printf "format string", var1, var2...`

format string 是格式化输出的关键，采用的 C 语言相同的 printf 功能。格式为 `%[modifier]control-letter`

Format Specifier Control Letters

| Control Letter | Desciption                                                                  |
| :------------- | :-------------------------------------------------------------------------- |
| c              | Displays a number as an ASCII character                                     |
| d              | Displays an integer value                                                   |
| i              | Displays an integer value(same as d)                                        |
| e              | Displays a number in scientific notation                                    |
| f              | Displays a floating-point value                                             |
| g              | Displays eigher scientific notation or floating point, whichever is shorter |
| o              | Displays an octal value                                                     |
| s              | Displays a text string                                                      |
| x              | Displays a hexadecimal value                                                |
| X              | Displays a hexadecimal value, but using capital letters for A through F     |

```sh
gawk 'BEGIN{                                       
quote> x = 1 * 100                              
quote> printf "The answer is: %e\n", x
quote> }'
# The answer is: 1.000000e+02
```

出来上面的控制符外，printf 还提供了另外三个控制项

* width, 控制宽度，小于设定值，给出空格补全，大于则用实际值覆盖
* prec, 
* -(minus sign) 强制左对齐

```sh
gawk 'BEGIN{FS="\n"; RS=""} {print $1, $4}' data2
# Riley Mullen (312)555-1234
# Frank Williams (317)555-9876
# Haley Snell (313)555-4938

gawk 'BEGIN{FS="\n"; RS=""} {printf "%s %s \n", $1, $4}' data2
# Riley Mullen (312)555-1234 
# Frank Williams (317)555-9876 
# Haley Snell (313)555-4938
```

如果用 printf 需要自己打印换行符号，这种设定当你想讲多行数据放在一行的时候就很好使

```sh
gawk 'BEGIN{FS=","} {printf "%s ", $1} END{printf "\n"}' data1
# data11 data21 data31
```

下面的例子我们通过 modifier 格式化名字这个字段

```sh
gawk 'BEGIN{FS="\n"; RS=""} {printf "%16s %s\n ", $1, $4}' data2
#     Riley Mullen (312)555-1234
#    Frank Williams (317)555-9876
#       Haley Snell (313)555-4938
```

默认是右对齐的，可以使用 minus sign 来左对齐

```sh
# gawk 'BEGIN{FS="\n"; RS=""} {printf "%-16s %s\n ", $1, $4}' data2Riley Mullen     (312)555-1234
# Frank Williams   (317)555-9876
# Haley Snell      (313)555-4938
```

格式化浮点类型

```sh
gawk '{                                                          
quote> total = 0      
quote> for (i=0; i<4; i++)          
quote> {
quote> total += $i    
quote> }
quote> avg = total / 3                                                          
quote> printf "Average: %5.1f\n", avg
quote> }' data5
# Average: 171.7
# Average: 191.0
# Average: 225.0
```

### Built-In Functions

gawk 提供了不少的内建函数帮助你完成一些特定功能。

#### Mathematical functions

The gawk Mathematical Functions

| Function    | Description                                                  |
| :---------- | :----------------------------------------------------------- |
| atan2(x, y) | The arctangent of x/y, with x and y specified in radians     |
| cos(x)      | The cosine of x, with x specified in radians                 |
| exp(x)      | The exponential of x                                         |
| int(x)      | The integer part of x, truncated toward 0                    |
| log(x)      | The natural logarithm of x                                   |
| rand()      | A random floating point value larger thant 0 and less than 1 |
| sin(x)      | The sine of x, with x specified in radians                   |
| sqrt(x)     | The square root of x                                         |
| srand(x)    | Specifies a seed value for calculating random numbers        |

gawk 是有计算上线的，比如 exp(1000) 就会抛错

gawk 还提供了位运算

* and(v1, v2)
* compl(val) 补全
* lshift(val, count) 左移
* or(v1, v2)
* rshift(val, count)
* xor(v1, v2) 异或

#### String functions

支持一些常规的字符操作，比如排序，截取，匹配，分割等

| Function         | Description                                                                                                                          |
| :--------------- | :----------------------------------------------------------------------------------------------------------------------------------- |
| split(s, r [,a]) | This function splits s into array a using the FS character, or the regular expression r if supplied. It returns the number of fields |

```sh
gawk 'BEGIN{x="testing"; print toupper(x); print length(x)}'
# TESTING
# 7
```

sort 比较复杂, 下面的 asort 例子中，我们将原始 map 和输出结果 test 传给 asort 然后遍历打印 test. 打印时可以看到，原来的字母 index 被替换成了数字

```sh
gawk 'BEGIN{
var["a"] = 1
var["g"] = 2
var["m"] = 3
var["u"] = 4
asort(var, test)
for (i in test)
print "Index:", i, " - value:", test[i]
}'
# Index: 1  - value: 1
# Index: 2  - value: 2
# Index: 3  - value: 3
# Index: 4  - value: 4
```

下面是 split 的测试

```sh
cat data1
# data11,data12,data13,data14,data15
# data21,data22,data23,data24,data25
# data31,data32,data33,data34,data35
gawk 'BEGIN{FS=","}{
split($0, var)
    print var[1], var[5]                     
}' data1
# data11 data15
# data21 data25
# data31 data35
```

#### Time functions

|Function|Description|
|:----|:----|
|mktime(datespec)|Converts a date specified in the format YYYY NN DD HH MM SS[DST] into a timestamp value|
|strftime(format[,timestamp])|Formats either the current time of day timestamp, or timestamp if provided, into a formatted data and date, using the date() shell function format|
|systime()|Returns the timestamp for the current time of day|

时间函数在处理带时间相关的 log 文件时很有用

```sh
gawk 'BEGIN{        
quote> date = systime()            
quote> day = strftime("%A, %B %d, %Y", date)
quote> print day                                
quote> }'
# Tuesday, June 15, 2021
```

### User-Defined Functions

#### Defining a function

语法

```sh
function name([variables])
{
    statements
}
```

```sh
function printthird()
{
    print $3
}
```

允许返回值 `return value`

```sh
function myrand(limit)
{
    return int(limit * rand())
}
```

#### Using your functions

当你定义一个函数的时候，它必须在最开始部分(before BEGIN).

```sh
gawk '      
quote> function myprint()                 
quote> {
quote> printf "%-16s - %s\n", $1, $4                                          
quote> }
quote> BEGIN{FS="\n"; RS=""}
quote> {
quote> myprint()
quote> }' data2
# Riley Mullen     - (312)555-1234
# Frank Williams   - (317)555-9876
# Haley Snell      - (313)555-4938
```

#### Creating a function library

1. 为自定义还是创建库
2. 将 gawk 脚本也存到文件中
3. 在终端同时调用两个脚本

```sh
cat funclib 
# function myprint()
# {
#     printf "%-16s - %s\n", $1, $4
# }
# function myrand(limit)
# {
#     return int(limit * rand())
# }
# function printthird()
# {
#     print $3
# }
cat script4
# BEGIN{ FS="\n"; RS=""}
# {
#     myprint()
# }
gawk -f funclib -f script4 data2         
# Riley Mullen     - (312)555-1234
# Frank Williams   - (317)555-9876
# Haley Snell      - (313)555-4938
```

### Working through a Practical Example

When work with data files, the key is to first group related data records together and then perform any calculations required on the related data.

下面是一个保龄球得分统计的例子, 每一行分别包含 名字，组名，得分 的信息

```sh
cat scores.txt 
# Rich Blum,team1,100,115,95
# Barbara Blum,team1,110,115,100
# Christine Bresnahan,team2,120,115,118
# Tim Bresnahan,team2,125,112,116
```

目标：统计每个 team 的总分以及平均分

```sh
cat bowling.sh                               
#!/usr/local/bin/bash

for team in $(gawk -F, '{print $2}' scores.txt | uniq)
do
    gawk -v team=$team '
    BEGIN{ FS=","; total=0 }
    {
        if ($2==team)
        {
            total += $3 + $4 + $5;
        }
    }
    END {
        avg = total/6;
        print "Total for", team, "is", total, ", the average is",avg
    }
    ' scores.txt
done

./bowling.sh 
# Total for team1 is 635 , the average is 105.833
# Total for team2 is 706 , the average is 117.667
```

计算方法：先遍历文件，取得所有的组名，然后再每个组名遍历一遍文件统计一次，打印一次。一开始我还以为可以一次对结果做分类统计的，如果是每次循环的话，还是很容易理解的

## Working with Alternative Shells

介绍除 bash 外其他一些常见的 shell, 暂时不关心，pass