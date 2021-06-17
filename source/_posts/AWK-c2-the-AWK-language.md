---
title: AWK c2 the AWK language
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

有时 pattern 会省略，有时 action 会省略。当 awk 检查完程序段没有语法错误后，他会一句一句的执行。pattern 没有写即表示匹配每一行。

本章第一节会介绍 pattern， 后面会介绍表达式，赋值等，剩余部分则是介绍函数等信息。

这里的准备文件是有讲究的，直接用 vscode 准备可能会出问题，最好在终端使用 echo + \t 的方式手动打一遍

PS: 试着用 cat 和 sed -n 'l' 观察一下文件

## Patterns

pattern 控制着 action 的执行，下面介绍六种 pattern 类型

* BEGIN { statements }
* END { statements }
* expression { statements }, 当 expression 为 true， statements 会被执行
* /regular expression/{ statements }
* compound pattern { statements }, pattern 通过 &&, ||, !, () 链接
* pattern1, pattern2 { statements }

### BEGIN and END

BEGIN 经常用来改变 field 的分隔符，默认的分隔符通过 FS 这个内置变量控制，默认的值有空格， / 和 tab.