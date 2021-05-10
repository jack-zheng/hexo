---
title: Python err str object is not callable
date: 2021-05-10 17:38:34
categories:
- python
tags:
- err
---

在 VSCode 上写代码的时候，终端突然不能做 int 到 String 的转化了

```python
# TypeError                                 Traceback (most recent call last)
# <ipython-input-40-5760e6fd64bf> in <module>
#      10     else:
#      11         tmpStr = re.sub('\s+', ' ', sub['preview'])
# ---> 12         allLines += "\n" + str(sub['lineNumber']) + " " + tmpStr
#      13 

# TypeError: 'str' object is not callable
```

查了下，str 是一个 global 的函数，如果之前有类似 `str = 'asdf'` 的赋值语句的话，后面对这个函数的调用就会出问题。。。。

回忆一下，貌似终端 debug 的时候有做过类似的操作 （；￣ェ￣）