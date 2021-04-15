---
title: jenv 快速上手
date: 2020-08-25 13:25:21
categories:
- 弹射起步
tags:
- jenv
---

jenv 是和 pyenv 一个类型的工具，应对多版本 java 的需求进行管理。简单记录一下 jenv 安装使用方法。[官方教程](https://www.jenv.be/)。需要注意的是在配置文件里添加完设置之后需要重起终端，不然文件夹什么还没有创建出来。

## CMDs

```bash
# 安装
brew install jenv

# bashrc/zshrc 中添加配置
export PATH="$HOME/.jenv/bin:$PATH"
eval "$(jenv init -)"

# local 已经安装的版本检测
which java

# 查看 brew 安装的 Java 路径
brew list java 

# 可以看到安装的路径是 /usr/local/Cellar/openjdk/XXX
# 默认就是从 openjdk repo 下载的
# 如果想安装其他版本可以 special 一下 version: brew list openjdk@11

# jenv 添加 home 路径
jenv add /usr/local/Cellar/openjdk@11/11.0.9

# 查看可用版本
jenv versions

# 如果想要只在某个路径下面指定 java 版本，可以 cd 到目标目录下，使用
jenv local 14

# 删除某个版本
jenv remove 14
```
