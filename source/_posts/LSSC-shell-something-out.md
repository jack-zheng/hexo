---
title: 牛刀小试
date: 2021-05-17 19:27:05
categories:
- Linux Shell Script Cookbook
tags:
- bash
- shell
---

## Introduce

以下实验均基于 bas

* 提示符 `$` 表示普通用户， `#` 表示 root 用户
* shebang: shell 文件首部的，以 `#!` 开头指定 shell 版本的语句
* 两种运行方式，终端对话框运行 or 运行脚本文件 `bash /path/to/file.script.sh`
* 历史记录存储在 `.bash_history` 文件中
* 两个 cmd 可以写在同一行，但是需要用 `;` 隔开， `echo a ; echo b` 等价于 `echo a \n echo b`

> 什么是 login shell
> A login shell is the shell which you get just after logging in to a machine. However, if you open up a shell while logged in to a graphical environment (such as GNOME, KDE, and so on), then it is not a login shell.

## Printing in the terminal

```bash
# 打印，自带结尾换行效果
echo "Welcome to Bash"
# Welcome to Bash

# 不带引号也可以
echo Welcome to Bash
# Welcome to Bash

# 单引号也可以
echo 'text in quotes'
# text in quotes
```

从上面效果看，都能打印，而且结果都一致，但是这几种使用方式还是有区别的

```bash
# " 中不能以 ！ 结尾, 在 bash 中 ！有特殊含义，表示前一个
echo "hi !"
# bash: !": event not found

# 调用之前命令测试
echo aaa
# aaa
!echo
# echo aaa
# aaa

# 但是如果 ！ 后面接空格则正常
echo "hi ! "
# hi !
```

echo 的几种用法的副作用：

1. 光杆 echo, 后面不能接 `;` 不然命令断裂了
2. 单引号 echo, 变量复制不起作用

echo 的其他用法：

1. `echo -n "hi"` 使用 `-n` 取出换行
2. `echo -e "1\t2\t3"` 使用 `-e` 打印转义符

echo 输出变色：

```bash
# \e[1;31m 设置字体为红色, \e[0m 重制
echo -e "\e[1;31m This is red text \e[0m"
# This is red text

# \e[1;42m 设置背景为绿色, \e[0m 重制
echo -e "\e[1;42m Green Background \e[0m"
# Green Background

# 混合使用也是 OK 的
echo -e "\e[1;42m\e[1;31m Green Background \e[0m"
```

```bash
# printf 和 C 语言中用法一致，默认不带换行
bash-3.2$ printf "hello world"
hello worldbash-3.2$

# 打印示例
printf  "%-5s %-10s %-4s\n" No Name  Mark
printf  "%-5s %-10s %-4.2f\n" 1 Sarath 80.3456
printf  "%-5s %-10s %-4.2f\n" 2 James 90.9989
printf  "%-5s %-10s %-4.2f\n" 3 Jeff 77.564
# No    Name       Mark
# 1     Sarath     80.35
# 2     James      91.00
# 3     Jeff       77.56
```

PS: -5, 左对齐，占 5 个位置

## Session 3 Playing with variables and environment variables

赋值语句 `var=value`，需要注意的是，`var = value` 是不对的，后者是等于运算。如果 value 中间没有空格，则直接写，不然需要用引号包裹。当使用 echo 或者 prinf 输出变量时，需要用**双引号**

```shell
name=jack
echo $name # or `echo ${name}`
# jack

name2 = jaack
# zsh: command not found: name2

name="jack zheng"
echo ${name}
# jack zheng

name='jack zheng'
echo ${name}
# jack zheng

fruit=apple
count=5 
echo "We have $count ${fruit}(s)"
# We have 5 apple(s)
echo 'We have $count ${fruit}(s)'
# We have $count ${fruit}(s)

# 输出变量长度
length=${#name}
echo $length
# 10 - it's the length of 'jack zheng'

# 显示当前的 shell 种类
echo $SHELL # or `echo $0`
# /bin/zsh

# 如果运行 script 的用户不是 root，打印提示信息
# 注意点：
#   1. 书上的例子首字母写错了
#   2. if [ 之间都有空格的，不然会抛语法错误
#   3. root user 的 UID 为 0
if [ $UID -ne 0 ]; then
    echo Non root user. Please run as root.
else
    echo Root user
fi

# 修改提示符内容
# 可以通过修改 .bashrc 中的 PS1 变量来达到该效果，不过我本地用的 zsh 和书上说的略有不同，就不是试了
```

## Function to prepend to environmnet variables

假设我们安装了新的应用到 `/opt/myapp`，这个应用包含 `bin` 和 `lib` 目录。为了使他生效，我们需要做如下设置

```bash
export PATH=/opt/myapp/bin:$PATH
export LD_LIBRARY_PATH=/opt/myapp/lib;$LD_LIBRARY_PATH
```

如果想简化上面的设置的话，我们可以在 `.bashrc` 里自定义一些函数

```bash
prepend() { [ -d "$2" ] && eval $1=\"$2':'\$$1\" && export $1; }
# $1=\"$2':'\$$1\" 人话说就是 PATH="New_Path:$PATH"
# 可以通过 echo $1=\"$2':'\$$1\" 测试

# 之前的命令可以简化为
prepend PATH /opt/myapp/bin
prepend LD_LIBRARY_PATH /opt/myapp/lib
```

但是上面的脚本有瑕疵，当原始 PATH 为 null 时，新生成的 PATH 就会以 `:` 结果，我们可以改进如下

```bash
prepend() { [ -d "$2" ] && eval $1=\"$2\$\{$1:+':'\$$1\}\" && export $1 ; }
```

PS: shell 中定义变量的格式 `${parameter:+expression}`

## 上例中语法解释

```bash
# 如何定义函数？
# 格式如下，写在 .zshrc 中，source 一下即可调用
demoFun() {
  echo "This is a function..."
}
demoFun   
# This is a function...

# 如何取得调用时传入的参数？
# 当位数超过 10 时需要用 ${n} 获取参数
# 其他特殊字符表示含义：
#   * $#: 传入参数的个数
#   * $*: 以一个字符串显示所有参数
#   * $@: 同上，但是需要加引号
#   * $$: 脚本运行的当前进程 ID
#   * $!: 后台运行的最后一个进程的 ID
#   * $?: 显示最后命令的退出状态，0 表示没错误，其他值表示有错误
funWithParam(){
    echo "第一个参数为 $1 !"
    echo "第二个参数为 $2 !"
    echo "第十个参数为 $10 !"
    echo "第十个参数为 ${10} !"
    echo "第十一个参数为 ${11} !"
    echo "参数总数有 $# 个!"
    echo "作为一个字符串输出所有参数 $* !"
}

funWithParam 1 2 3 4 5 6 7 8 9 34 73
# 第一个参数为 1 !
# 第二个参数为 2 !
# 第十个参数为 34 !
# 第十个参数为 34 !
# 第十一个参数为 73 !
# 参数总数有 11 个!
# 作为一个字符串输出所有参数 1 2 3 4 5 6 7 8 9 34 73 !

# [] 什么意思？
# shell 中方括号和 Test 等价

# -d 什么意思？
# -d filename 表示是否为目录，但是总感觉用在这里词不达意
# 类似的还有 -eq，判等

# eval 什么意思？
# eval 如果后面直接跟命令，直接运行，如果命令中包含变量，则计算变量后运行
set 11 22 33 44
# 假设我们不知道参数长度，可以如下输出最后一个参数
echo $#
# 4
echo "\$$#"
# $4
eval echo "\$$#"
# 44

# 修改后的 \$\{$1:+':'\$$1\} 是什么意思？
# :+ 表示覆盖缺省值
# 只有当var不是空的时候才替换成string, 若var为空时则不替换或者说是替换成变量var的值,即空值
COMPANY="Nightlight Inc."
echo "${COMPANY:+Company has been overridden}"
# Company has been overridden
COMPANY=
echo "${COMPANY:+Company has been overridden}"
# 打印空行

# && 什么意思
# 逻辑与，前面条件成立才能执行后面的逻辑
```

## Math with the shell

体验下来就一个感觉，bash 下的计算略微繁琐

```bash
# 准备变量
no1=4; no2=5

# 使用 let 关键字
# 整个表达式没有空格，不然报错！！！
let result=no1+no2
echo $result
# 9

# 支持自增/减
let no1++ # 也支持写成 no1+=1
let no2-- # 也支持写成 no2-=1
echo $no1 $no2
# 6 4

# 和 let 功能相同的还有 []
result=$[ no1 + no2 ]   # 等号两边不能有空格，中括号中间无所谓
echo $result
# 10

# 方括号中间的变量可以带 $，不影响结果
result=$[ $no1 + 100 ]
echo $result
# 106

# 相同的功能还有 (()), 用法和 [] 一致
result=$(( $no1 + 50 ))
echo $result
# 56

# 相同功能的还有 expr
result=`expr $no1 + 4`
echo $result
# 10

# 但是它们貌似都只能做整数计算
let result=3+4.0
# bash: let: result=3+4.0: syntax error: invalid arithmetic operator (error token is ".0")
result=$[expr $no1 + 4.0]
# bash: expr 6 + 4.0: syntax error in expression (error token is "6 + 4.0")
bash-3.2$ result=$((expr $no1 + 4.0))
# bash: expr 6 + 4.0: syntax error in expression (error token is "6 + 4.0")
result=`expr $no1 + 4.0`
# expr: not a decimal number: '4.0'
```

如果想要有精度的计算，可以使用 `bc`, bc 是basic calculator/bench calculator的简称。其语法类似于C语言，支持加减乘除还有更多复杂的运算。

```bash
>>> bc
1+2
# 3
quit

echo "4 * 0.56" | bc
# 2.24

result=`echo "$no * 1.5" | bc`
echo $result
# 81.0

# 可以指定精确的位数
echo "scale=2;3/8" | bc
# .37

# 可以做进制转化, 而且必须先设置 obase 在设置 ibase
no=100
echo "obase=2;$no" | bc
# 1100100
echo "obase=10;ibase=2;1100100" | bc
100

# echo 也可以达到进制转化的效果，不过只能转成 10 进制
echo $[8#11] # 效果等同与 $((8#11))
# 11

# bash 中也有自带的函数，比如次方和开方
echo "10^2" | bc
# 100
echo "sqrt(100)" | bc
# 10
```

> man page 说明如下: 
> There are four special variables, scale, ibase, obase, and last. scale defines how some operations use digits after the decimal point. The default value of scale is 0.
> ibase and obase define the conversion base for input and output numbers. The default for both input and output is base 10.
> last (an extension) is a variable that has the value of the last printed number. These will be discussed in further detail where appropriate.
> All of these variables may have values assigned to them as well as used in expressions.

## Playing with file descriptors and redirection

文件描述符：

* 0: stdin
* 1: stdout
* 2: stderr

```bash
# 基操，输入到文件，如果原来有值，会覆盖
echo "This is a sample text 1" > tmp.txt
ls
# tmp.txt

# append 内容
echo "This is a sample text 2" >> tmp.txt
cat tmp.txt
# This is a sample text 1
# This is a sample text 2

# 重定向错误信息
ls +
# ls: +: No such file or directory
# 打印命令返回值，非 0 都是失败
echo $?
# 1

# 尝试将 err 导入文件
ls + > out.txt
# ls: +: No such file or directory
cat out.txt
# 无内容

# stderr 标识符 2，所以 2> 表示把错误信息导入到某个流处理的意思
ls + 2> out.txt
cat out.txt 
# ls: +: No such file or directory

# 还可以分别指定流出口
cmd 2>stderr.txt 1>stdout.txt
cat stderr.txt 
# bash: cmd: command not found

# 将标准输出和错误信息一起写入文件
cmd > out.txt 2>&1
# 他还有一个简写方式
cmd &> out.txt

# 测试 /dev/null
# 这个测试感觉上不怎么贴切，但是，它用来生产 err 的方式还是很可以的
# 测试描述：生产三个文件，将其中一个权限变为 000, cat 这三个文件就会抛异常
echo a1 > a1; cp a1 a2; cp a1 a3; chmod 000 a1;
cat a* 2> err.txt
# a1
# a1
cat err.txt
# cat: a1: Permission denied
# 将 err 导向 /dev/null 则没有输出
cmd 2> /dev/null

# tee 将输出写入文件的同时，给一份到 stdout
# 默认情况下 tee 会覆盖原文件，用 tee -a 可以达到 append 的效果
cat a* | tee out.txt | cat -n
# cat: a1: Permission denied
#      1	a1
#      2	a1
cat out.txt
# a1
# a1

# tee 后接多个 file，内容都是重复的
echo aaa | tee f1 f2
cat f1 f2
# aaa
# aaa

# `-` 代表标准输入, 
echo who is this | tee -
# who is this
```

`>` Vs `>>`: 前者是覆盖，后者是续接

> ./dev/null is a special device file where any data received by the file is discarded. The null device is often known as a black hole as all the data that goes into it is lost forever.
> /dev/null 是一个特殊的设备，可以将它看作一个黑洞，所有进去的东西都没了 

```bash
# 终端多行输入
cat << EOF > log.txt
LOG FILE HEADER
This is a test log file
Function: System statistics
EOF

cat log.txt
# LOG FILE HEADER
# This is a test log file
# Function: System statistics
```

自定义文件描述符

可用模式：Read mode, Write with truncate mode, Write with append mode

```bash
# `<` 用于将文件导向 stdin
echo this is a test line > input.txt
exec 3<input.txt
cat <&3
# this is a test line
cat <&3
# 没有输出，这种方式不能复用，需要重新赋值
```

为写操作自定义文件描述符

```bash
exec 4> output.txt
echo newline >&4
cat output.txt
# newline
# 书上说这种方式再次调用会覆盖的才对，我这边测试是以 append 方式附加的
echo aaa >&4
cat output.txt
# newline
# aaa
```

为 append 模式的 write 定义文件描述符

```bash
exec 5>>out.txt
echo appended line >&5
cat out.txt
# appended line
echo appended line2 >&5
cat out.txt
# appended line
# appended line2
```

## Arrays and associative arrays

```bash
# 数组声明及调用
array_var=(1 2 3 4 5 6)
echo ${array_var[0]}
# 1
# 输出全部
echo ${array_var[*]}  # or echo ${array_var[@]}
# 1 2 3 4 5 6
# 打印长度
echo ${#array_var[*]}
# 6

# 输出 index 下标
echo ${!array_var[*]} # or echo ${!array_var[@]}
# 0 1 2 3 4 5
```

Associative arrays: 关系型数组，普通数组只能存整形，但是关系型数组可以存储混合的，任何 text 格式的数据。这种数据类型是在 Bash 4.0 引入的

Mac 升级 Bash 版本

```bash
# 查看当前版本
bash --version
# GNU bash, version 3.2.57(1)-release (x86_64-apple-darwin20)
# Copyright (C) 2007 Free Software Foundation, Inc.

# 小八卦：Bash 在 3.2 后的版本改了协议，开始使用 GPLv3 许可，Apple 不想支持，所以 Mac 上默认 Bash 只到 3.2 为止
# 新的 Bash 有很强的 tab 补全，值得期待

# 查看以安装 bash 路径
which bash
# /bin/bash

brew install bash
# Updating Homebrew...
# ==> Downloading https://mirrors.ustc.edu.cn/homebrew-bottles/bash-5.1.8.big_sur.bottle.tar.gz
# ######################################################################## 100.0%
# ==> Pouring bash-5.1.8.big_sur.bottle.tar.gz
# /usr/local/Cellar/bash/5.1.8: 157 files, 10.9MB

which bash
/usr/local/bin/bash

bash --version
# GNU bash, version 5.1.8(1)-release (x86_64-apple-darwin20.3.0)
# Copyright (C) 2020 Free Software Foundation, Inc.
# License GPLv3+: GNU GPL version 3 or later <http://gnu.org/licenses/gpl.html>

# This is free software; you are free to change and redistribute it.
# There is NO WARRANTY, to the extent permitted by law.

## 安装结束
```

Associative arrays 使用案例

```bash
declare -A fruits_value
fruits_value=([apple]='100 dollars' [orange]='150 dollars')
echo "Apple costs ${fruits_value[apple]}"
# Apple costs 100 dollars

# 输出下标，或者索引更贴切
echo ${!fruits_value[*]}
# orange apple
```

PS: 感觉被骗了，这 TM 叫数组？！！命名就是字典嘛

## Visiting aliases

alias: 将很长的命令用一个简写来代替， 声明形式 `alias new_command='command sequence'`, 例如 `alias install='sudo apt-get install'`

终端声明的 alias 是零时的，重启终端后失效，可以将其写入 rc 文件 `echo 'alias cmd="command seq"' >> ~/.bashrc`

```bash
# 显示所有可用的别名
alias 

# 重新定义 rm 行为，删除文件时将它备份
# 测试失败。。。
alias rm='cp $@ ~/backup && rm $@'

# 这应该就是持续失败的原因，参数为止调换了
# 搜了下好像每有类似的问题，难道我系统坏了？？
alias mycp='echo "cp $@ ~/backup"'
mycp aaa.txt
# cp  ~/backup aaa.txt

# disable alias, 前面加一个反斜杠
\command

# 显示定义的 alias
alias rm
# alias rm='cp $@ ~/backup && rm $@'

# 删除
unalias rm
alias rm
# bash: alias: rm: not found
```