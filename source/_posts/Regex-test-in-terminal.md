---
title: Regex test in terminal
date: 2021-05-11 11:03:55
categories:
- regex
tags:
- test
---

终端如何测试一个正则表达式

```bash
echo 'aaabbbccc' | grep -E bc       # bc 会被标红
# aaabbbccc

echo 'aaabbbccc' | grep -E -o bc    # 只输出匹配的字段
# bc

echo 'a1b a2b acb' | grep -E -o 'a[^0-9]b'      # 当方括号中有 ^ 时，表示除外的意思
# acb

echo 'a1b a2b acb' | grep -G 'a\w+'     # 使用 basic regex 匹配的时候没有适配的结果

echo 'a1b a2b acb' | grep -G 'a\w\+'    # 需要在 '+' 的前面也添加转义符才能生效
```

## BRE ERE

`man grep` 可以看到它支持的一些可选正则模式, 这几种模式有什么区别？

```txt
-E, --extended-regexp     PATTERN is an extended regular expression (ERE)
-G, --basic-regexp        PATTERN is a basic regular expression (BRE)
-P, --perl-regexp         PATTERN is a Perl regular expression
-e, --regexp=PATTERN      use PATTERN for matching
```

> In GNU sed, the only difference between basic and extended regular expressions is in the behavior of a few special characters: ‘?’, ‘+’, parentheses('()'), braces('{}'), and ‘|’.

| Type  |                         Desc                          |
| :---: | :---------------------------------------------------: |
|  BRE  | character like () {} + ? \| need use escape character |
|  ERE  | no need to use escape caharcter before + ? ( ) { } \| |
|  PRE  |            same as ERE, and add other func            |

BRE、ERE可以使用 POSIX 字符集来操作

使用支持
grep
支持BRE,通过参数控制,默认BRE, -P开启PRE, -E开启ERE

sed
支持BRE,默认BRE,-r开启ERE

awk
支持ERE,默认ERE。

## 其他一些知识点

元字符(Metacharacter), 指SHELL直译器或正则表达式（regex）引擎等计算机程序中具有特殊意义的字符。

在 POSIX 扩展正则表达式里,定义了14个元字符,它们被作为一般的字符使用时,必须要通过 "转义"（前面加一个反斜杠"\"）来去除他们本身的特殊意义,这些元字符包括：

开和闭方括号："["和"]"
反斜线："\"
脱字符："^"
美元符号："$"
句号/点："."
竖线/管道符："|"
问号："?"
星号："*"
加号："+"
开和闭 花括号："{"和"}"
开和闭 小括号："("和")"
