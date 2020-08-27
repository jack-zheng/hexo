---
title: Python 版本管理-pyenv
date: 2020-07-29 13:20:05
categories:
- 编程
tags:
- python
- pyenv
---

poetry 推荐使用 pyenv 进行本地 python 的多版本管理，以前用过，但是也没什么特别的印象了，特此记录一下使用情况

## 安装

* [官方教程](https://github.com/pyenv/pyenv)

Win 平台不支持这个工具，残念。。。

通过 brew 安装, brew 加速的教程在另一篇教程里有提到

```bash
brew update
brew install pyenv
```

在 profile 中添加配置使能，我本地用的 zsh, 各版本的 shell 稍有区别，指定的文件不一样

```bash
echo -e 'if command -v pyenv 1>/dev/null 2>&1; then\n  eval "$(pyenv init -)"\nfi' >> ~/.zshrc
```

安装推荐的工具机，各种系统不一样

For MacOS, install Xcode Command Line Tools (xcode-select --install) and HomeBrew, then optional but best install

```bash
brew install openssl readline sqlite3 xz zlib
```

在系统中可以通过输入 `echo $(pyenv root)` 拿到目录地址

## 常用命令

直接输入 `pyenv` 查看所有的 cmd 信息

### 安装某个版本的 python

```bash
pyenv install 3.7.8
```

如果没打全，他会给提示可用的版本，很人性化。安装的 python 版本会被放到 `~/.pyenv/versions/` 管理

### 删除对应版本

`pyenv uninstall 3.7.8` 或直接去 versions 文件夹下删除

### 显示可用版本

`pyenv versions`

```bash
pyenv versions
* system (set by /Users/jack/.python-version)
  3.6.5
  3.7.8
```

### 切换版本

多用 `pyenv version` 查看当前的环境版本信息

使用前的情况：系统自带 python 版本 2.7.16， pyenv 可用版本 3.6.5 和 3.7.8。此时 cmd 输入 `python -V` 给出版本 `2.7.16`

全局切换版本 `pyenv gloabl 3.7.8`，他会将这个版本存放到 `.pyenv/version` 文件中，再打开终端查看版本，变为 `3.7.8`。

`pyenv local 3.6.5` 可以指定 folder 下的 python 版本，他会将版本信息写入当前目录下的 `.python-version` 文件中

如果想要指定终端的 python 版本，可以用 `pyenv shell xxx`, 这个我到时没有亲测

作用范围和其编程语言一样，范围最小的那个生效 `shell > local > gloabl`

### 查看 python 路径

`pyenv which python`

### 更新

每次新安装版本，记得跑一下 `pyenv rehash` 更新信息

## Issues

pyenv install 下载失败, 报错

```bash
 Jack > ~ > pyenv install 3.7.3
python-build: use openssl@1.1 from homebrew
python-build: use readline from homebrew
Downloading Python-3.7.3.tar.xz...
-> https://www.python.org/ftp/python/3.7.3/Python-3.7.3.tar.xz
error: failed to download Python-3.7.3.tar.xz

BUILD FAILED (OS X 10.15.6 using python-build 20180424)
```

可以自行下载对应的 tar.xz 文件然后放到 pyenv 的 cache 文件夹下，pyenv install 的时候会取对应的安装包进行安装

```bash
wget -P $(pyenv root)/cache https://www.python.org/ftp/python/3.7.3/Python-3.7.3.tar.xz
```


## Refer

* [参考](http://einverne.github.io/post/2017/04/pyenv.html)
