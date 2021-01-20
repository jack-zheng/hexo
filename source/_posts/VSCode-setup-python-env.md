---
title: VSCode setup python 环境
date: 2020-06-12 15:43:34
categories:
- vscode
tags:
- python
---

VSCode setup python 独立运行环境

## 安装依赖

1. 安装 pipenv `pip install pipenv --user`
2. 创建独立环境 `pipenv shell`, 还可以通过 `pipenv --three/two` 指定 python 版本
3. 修改 pipfile, 使用国内源加速
4. 安装依赖 `pipenv install pdfminer.six`

```pipfile
[[source]]
name = "pypi"
url = "https://pypi.douban.com/simple"
verify_ssl = true
```

查看 VSCode 左下角的运行环境是不是你新建的那个，不是的话 `pipenv --venv` 查看新建 venv 路径， `Ctrl + Shift + p` 搜索 `python: select interpreter` 选择你新建的那个 env

## reload module after update

如果某些方法正在进行中，可能频繁修改，在 ipython 中调试的时候可以用 reload 来重新加载，也可以指定 ipython 到自动重加载模式 [autoreload mode](https://ipython.org/ipython-doc/stable/config/extensions/autoreload.html)

```python
import importlib
importlib.reload(PDFParser)
```
