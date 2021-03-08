---
title: Python re lib method abstrct
date: 2021-03-03 19:28:29
categories:
- python
tags:
- re
---

Python 的 re 包里面的 search 和 match 经常搞不清楚，特意整理记录下 re 包支持的方法

**re.search(pattern, string, flags=0)** 字符串任意位置匹配

Scan through string looking for the first location where the regular expression pattern produces a match, and return a corresponding match object. Return None if no position in the string matches the pattern; note that this is different from finding a zero-length match at some point in the string.

```python
ret = re.search(r'[\d]+', 'aaa123bbbb')
ret.group(0)

# '123'
```


**re.match(pattern, string, flags=0)** 从头开始匹配

If zero or more characters at the beginning of string match the regular expression pattern, return a corresponding match object. Return None if the string does not match the pattern; note that this is different from a zero-length match.

Note that even in MULTILINE mode, re.match() will only match at the beginning of the string and not at the beginning of each line.

If you want to locate a match anywhere in string, use search() instead (see also search() vs. match()).

```python
re.match(r'[\d]+', 'aaa123aaa')
# null

re.match(r'[\d]+', '123aaa') 
<re.Match object; span=(0, 3), match='123'>
```