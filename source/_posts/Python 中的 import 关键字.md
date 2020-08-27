---
title: Python 中的 import 关键字
date: 2020-01-23 13:45:22
categories:
- 编程
tags:
- python
- import
- module
- package
---
看 you-get 源码时卡在了 import package 这个点，特此记录一下搜索资料的结果

## Import Of Python

你在看 python 代码的时候经常会在文件头部发现一串代码，格式类似 `import xxx` 或者 `from xxx import xxx`。功能都是一样的，引入代码重复利用。分两种，一种是引入 module，另一种是映入 package。

- module 简单理解就是组织好的 python 文件
- package 即使用文件夹形式组织 python 文件，在 package 的更目录下会有一个 `__init__.py` 文件作为 package 的入口

## 相对引用

clone 了 rich 的源码通过 `python ./styled.py` 运行时报错

```bash
(rich-2qeSub0j-py3.7)  i306454@C02TW719HTD5  ~/gitStore/rich/rich   master  python ./styled.py
Traceback (most recent call last):
  File "./styled.py", line 3, in <module>
    from .measure import Measurement
ImportError: attempted relative import with no known parent package
```

这是因为对应的文件中采用了相对引用就是类似 `from .style import StyleType` 的语法，我们可以通过在上一级目录下输入 `python -m rich.styled` 运行。注意命令没有 `.py` 后缀

## Reference

- [cnlogs - Python学习者](https://www.cnblogs.com/yan-lei/p/7828871.html)
