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