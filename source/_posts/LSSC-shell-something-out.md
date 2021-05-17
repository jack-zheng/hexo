---
title: 牛刀小试
date: 2021-05-17 19:27:05
categories:
- Linux Shell Script Cookbook
tags:
- Shell Something Out
---

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