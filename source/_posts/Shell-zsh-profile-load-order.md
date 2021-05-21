---
title: Zsh 配置文件的 load 顺序
date: 2021-05-07 15:59:38
categories:
- Shell
tags:
- zsh
---

今天在看 sourcegraph 这个开源工具的时候，发现他有提供终端查询接口的，但是要配置参数到 `.zprofile` 中, 突然想问一句 `.zshrc` 和 `.zprofile` 有什么关系？

* [stackexchange](https://unix.stackexchange.com/questions/71253/what-should-shouldnt-go-in-zshenv-zshrc-zlogin-zprofile-zlogout)

> `.zprofile` is basically the same as `.zlogin` except that it's sourced before `.zshrc` while `.zlogin` is sourced after `.zshrc`.
>  According to the zsh documentation, ".zprofile is meant as an alternative to .zlogin for ksh fans; the two are not intended to be used together, although this could certainly be done if desired."

PS: Ksh 是很早就出现的一种 shell

由上面的描述我们可以直到，zsh 的 config file 生效顺序为 zprofile -> zsh -> zlogin

## 实验

```sh
# shell 的 config file 是可以直接执行 cmd 的，在 .zprofile 和 .zshrc 中分别加上 "echo hello from .zprofile"
# 和 "echo hello from .zshrc". 新起一个终端会加载这两个配置，观察输出结果，和之前的描述一致

hello from .zprofile
hello from .zshrc
```