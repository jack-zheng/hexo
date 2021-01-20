---
title: Poetry 快速入门
date: 2020-07-17 17:59:45
categories:
- python
tags:
- poetry
---

Poetry 类 pipenv 工具，据说 lock 什么的速度更快，而且有集成发布功能，刚好 rich 这个项目有用这个，刚好在看源码的时候体验一把

## 安装

### Win

```bash
# powershell 输入
(Invoke-WebRequest -Uri https://raw.githubusercontent.com/python-poetry/poetry/master/get-poetry.py -UseBasicParsing).Content | python

# 提示 error, 原因是 DNS 污染
Invoke-WebRequest : 未能解析此远程名称: 'raw.githubusercontent.com'
所在位置 行:1 字符: 2
+ (Invoke-WebRequest -Uri https://raw.githubusercontent.com/python-poet ...
+  ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
    + CategoryInfo          : InvalidOperation: (System.Net.HttpWebRequest:HttpWebRequest) [Invoke-WebRequest]，WebExce
    ption
    + FullyQualifiedErrorId : WebCmdletWebResponseException,Microsoft.PowerShell.Commands.InvokeWebRequestCommand

# 解决方案：修改 host 文件
# 目录：C:/Windows/System32/drivers/etc/
# 管理员模式打开，添加文本: 151.101.0.133 raw.githubusercontent.com
# 刷新DNS
ipconfig /flushdns

# 链接成功，但是报其他错误
Invoke-WebRequest : 基础连接已经关闭: 发送时发生错误。
所在位置 行:1 字符: 2
+ (Invoke-WebRequest -Uri https://raw.githubusercontent.com/python-poet ...
+  ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
    + CategoryInfo          : InvalidOperation: (System.Net.HttpWebRequest:HttpWebRequest) [Invoke-WebRequest]，WebExce
    ption
    + FullyQualifiedErrorId : WebCmdletWebResponseException,Microsoft.PowerShell.Commands.InvokeWebRequestCommand

# 改完之后各种报错，烦躁。这个命令就是下载一个 get-poetry.py 的 raw 文件，然后使用 python get-poetry.py 安装。我直接下载这个文件然后安装了。。。

# 尼玛，被墙了安装超级慢 (╬▔皿▔)╯ 最后用小飞机开启全局代理， 再 CMD 窗口 python get-poetry.py 安装成功

Retrieving Poetry metadata

Before we start, please answer the following questions.
You may simply press the Enter key to leave unchanged.
Modify PATH variable? ([y]/n)

# Welcome to Poetry!
This will download and install the latest version of Poetry,
a dependency and package manager for Python.
It will add the `poetry` command to Poetry's bin directory, located at:
%USERPROFILE%\.poetry\bin
This path will then be added to your `PATH` environment variable by
modifying the `HKEY_CURRENT_USER/Environment/PATH` registry key.
You can uninstall at any time by executing this script with the --uninstall option,
and these changes will be reverted.

Installing version: 1.0.10
  - Downloading poetry-1.0.10-win32.tar.gz (11.96MB)

Poetry (1.0.10) is installed now. Great!
To get started you need Poetry's bin directory (%USERPROFILE%\.poetry\bin) in your `PATH`
environment variable. Future applications will automatically have the
correct environment, but you may need to restart your current shell.

# 重启一下终端，输入命令检测安装
poetry --version

# 但是在 vscode 的终端中还是不能识别，手动将 user\.poetry\bin 添加到系统 path 中重启 vscode, 识别成功
```

PS: 国内安装各种软件有助于增长火气！！！

## 常用 Command

### poetry new project-name

初始化项目, 创建必要文件。你可以在 git 上先建一个空的仓库然后，本地做完 poetry init 和 git init 之后 match 一下

初始化后目录为

```bash
job-spider
├── pyproject.toml
├── README.rst
├── job_spider
│   └── __init__.py
└── tests
    ├── __init__.py
    └── test_job_spider.py
```

通过配置 toml 文件指定国内源加速

```toml
[[tool.poetry.source]]
name = "douban"
url = "https://pypi.doubanio.com/simple/"
```

### poetry config --list

查看配置，比如 virtualenv 会创建在哪里之类的。这个 cmd 还是很有帮助的，可以通过它知道你的虚拟环境创建在哪里，是不是要在 project 创建 venv 等

```bash
$ poetry config --list
cache-dir = "/Users/jack/Library/Caches/pypoetry"
virtualenvs.create = true
virtualenvs.in-project = false
virtualenvs.path = "{cache-dir}/virtualenvs" # /Users/jack/Library/Caches/pypoetry/virtualenvs
```

通过指定 `poetry config virtualenvs.in-project true` 可以指定将虚拟环境创建到 project 目录下面，方便管理

### poetry shell

激活环境, 如果还没有创建过虚拟环境，他还会根据 toml 文件新建一个

### poetry install

并不是安装依赖，而是根据 toml 文件安装项目依赖，对标 `pipenv sync`

### poetry add

对标 pipenv 中的 `pipenv install`, 使用 `add --dev/-D flask` 安装 dev 相关的包

### poetry env info

`poetry env info`: 显示运行环境信息，包括本地 OS 和虚拟环境

```bash
Virtualenv
Python:         3.7.5
Implementation: CPython
Path:           /Users/jack/gitStore/mycommands/.venv
Valid:          True

System
Platform: darwin
OS:       posix
Python:   /Library/Frameworks/Python.framework/Versions/3.7
```

`poetry env list`: 显示可用的 env 列表

官方推荐 poetry 结合 pyenv 管理各种版本的虚拟环境

## poetry show

显示已安装的依赖

```bash
poetry show
atomicwrites       1.4.0  Atomic file writes.
attrs              19.3.0 Classes Without Boilerplate
click              7.1.2  Composable command line interface toolkit
flask              1.1.2  A simple framework for building complex web applications.
...
```

## 遇到的问题

### Resolving dependency 挺慢

在安装更新的时候 resolving dependency 挺慢的，等了好一会儿，一度认为进程死了。但是第二次就快多了，可能是有 cache

```bash
C:\Users\jack\gitStore\job-spider\job_spider>poetry install --verbose
Updating dependencies
Resolving dependencies...
```

### 编译器识别有问题

观察 VSCode 的左下角，python 编译器经常选择有问题，会找不到自己创建的虚拟环境路径。可以点击它，然后根据 poetry shell 的提示手动设置，路径如 `C:\Users\jack\AppData\Local\pypoetry\Cache\virtualenvs\job-spider-UlnXzhyt-py3.7` 做完后他会自动保存到 `.vscode` 的工程文件夹下。但是我默认这个文件是不 check in 的，所以然并卵 ┑(￣Д ￣)┍

### Win 启动 flask 失败

新建了一个 flask demo，启动的时候报错

```bash
PS C:\Users\jack\gitStore\job-spider> poetry run .\job_spider\main.py

[OSError]
[WinError 193] %1 不是有效的 Win32 应用程序。
```

据说是 windows 上安装了 64 位的 python， 调用了 32 位的 dll 会报这个错，换个 32 位的 python 就能解决。将原有的 64 位卸载，删除各种环境变量，重新安装 32 位 python，然并卵，要自闭了 (￣ε(#￣)

暂时没有什么其他更好的解决方案，打算用虚拟机或者在 MacOS 上完成开发以节省时间

今天在 Mac 上用 3.7.8 的版本也会抛同样的错误！！！难道是版本有问题？果断用 `3.6.6`, `3.7.3` 试试，可行。。。。回去再到 Windows 的机子上试试这个版本。

在 Win 上换 3.7.3 之后一切正常 ╰(艹皿艹 )

### MacOS poetry install 报错

切换到 3.6.5 之后 poetry install 报错

```bash
[EnvCommandError]
Command ['/Users/jack/gitStore/splunk-collector/.venv/bin/pip', 'install', '--no-deps', 'zipp==3.1.0'] errored with the following return code 1, and output:
pip is configured with locations that require TLS/SSL, however the ssl module in Python is not available.
...
...
```

是 OpenSSL 包缺失导致的

```bash
# 修复，第一行可以不运行，下载包经常卡住
brew update && brew upgrade
brew uninstall --ignore-dependencies openssl; brew install https://github.com/tebelorg/Tump/releases/download/v1.0.0/openssl.rb

brew reinstall python
```

这之后还重新将 pyenv 管理的 python 重新卸载安装了一下，问题解决

### MacOS poetry run

```bash
 poetry run splunk_collector/main.py

[PermissionError]
[Errno 13] Permission denied
```

运行 flask demo, permission 报错。完全搞错了。。。。flask 并不是那样运行的。保存完文件之后, 通过如下方式运行，而不是直接用 poetry 或者 python 运行，我 凸^-^凸

```bash
$ export FLASK_APP=hello.py
$ flask run
 * Running on http://127.0.0.1:5000/
```
