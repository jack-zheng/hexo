---
title: jenv 快速上手
date: 2020-08-25 13:25:21
categories:
- 小工具
tags:
- java
---

jenv 是和 pyenv 一个类型的工具，应对多版本 java 的需求进行管理。简单记录一下 jenv 安装使用方法。[官方教程](https://www.jenv.be/)。需要注意的是在配置文件里添加完设置之后需要重起终端，不然文件夹什么还没有创建出来。

## CMDs

```bash
# 安装
brew install jenv

# local 已经安装的版本检测
which java

# 安装多版本 java, 可以通过 java11, java12 等指定版本。默认最新版
brew cask install java
# 安装成功会给出路径信息：Moving Generic Artifact 'jdk-14.0.2.jdk' to '/Library/Java/JavaVirtualMachines/openjdk-14.0.2.jdk'.

# jenv 添加 home 路径
jenv add /Library/Java/JavaVirtualMachines/openjdk-14.0.2.jdk/Contents/Home

# 查看可用版本
jenv versions

# 如果想要只在某个路径下面指定 java 版本，可以使用
jenv local 14
```
