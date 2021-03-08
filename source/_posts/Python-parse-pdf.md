---
title: Python 解析 PDF
date: 2020-06-19 17:24:53
categories:
- python
tags:
- pdf
---

使用 python 解析 PDF 文件，提取文件中表格的数据。随便在网上找了一个 PDF 文件做样本。使用 `filetype:pdf 价格表格` 的到样本文件。

稍微检索了一下，当下貌似名为 camelot 的 python lib 很火，就用这个做实验吧

## 安装

这一步还挺复杂，需要安装挺多依赖，具体参考官方文档，这里只记录我本地环境的安装步骤

MacOS:

1. `brew install tcl-tk ghostscript`, 然后终端输入 `gs -version`, 在 python 命令行中输入 `import tkinter` 验证依赖是否安装成功
2. `pip3 install camelot-py[cv] --user` 安装报错，是 zsh 的锅，切换回 bash 安装即可

## 测试

运行了一下官方给的例子，成功。但是我自己下载的中文 pdf 有问题，查了下，是说 camelot 基于 PyPDF2，然后这个 lib 是不支持处理中文字符的，不过可以通过修改对应 lib 的源码实现支持，网上有教程。不过我暂时只处理英文文档，就不纠结了。

```python
import camelot
tables = camelot.read_pdf('foo.pdf')
tables[0].df
# 输出表格，foo.pdf 在官方教程中有给下载链接
```
