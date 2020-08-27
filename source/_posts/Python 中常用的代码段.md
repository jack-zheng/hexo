---
title: Python 中常用的代码段
date: 2020-01-23 14:11:18
categories:
- 编程
tags:
- python
- tips
---
记录一些我经常查找的 python 方法作为备忘

## Generate random int list, or just a requirement of loop N times

it's a common requirement and some guys achieve this goal by using Numpy lib, but it's too heavy. you can do in this way:

```python
import random

for _ in range(10)
  print(random.randint(0, 100))
  
# the _ is from 0 - 9
```

## Get index and val at the same time

```python
a = ['a','b','c','d']
for idx, val in enumerate(a):
   print(f'idx = {idx}, val = {val}')


# Output:
# idx = 0, val = a
# idx = 1, val = b
# idx = 2, val = c
# idx = 3, val = d
```

if you want to specify the start index, you can add a second parameter to enumerate func

```python

# in this case, idx would start from 3
a = ['a','b','c','d']
for idx, val in enumerate(a, 3):
  print(f'idx = {idx}, val = {val}')
  
# Output:
# idx = 3, val = a
# idx = 4, val = b
# idx = 5, val = c
# idx = 6, val = d
```

## Ipython 交互界面重新引入修改后的包

```python
import importlib
importlib.reload(some_module)
```

## for loop one line mode

```python
user_ids = [record['login'] for record in resp]

# if you need if condition
list = [1,2,3,4,5,6]
filter = [str(sub + "tt") for sub in list if sub >= 3]
```

## __repr__ Vs __str__

* 只重写 __str__ 只定制在 print() 时的输出
* 只重写 __repr__ print() 和 调用都输出定制内容
* 重写 __str__ + __repr__ print() 输出 str 定制内容，调用输出 repr 内容

```python
class N1:
    def __init__(self, data):
        self.data = data

    def __str__(self):
        return 'N1: data=%s' % self.data

class N2:
    def __init__(self, data):
        self.data = data

    def __repr__(self):
        return 'N2: data=%s' % self.data

class N3:
    def __init__(self, data):
        self.data = data

    def __repr__(self):
        return 'N3 repr: data=%s' % self.data

    def __str__(self):
        return 'N3 str: data=%s' % self.data

'''output
n1 = N1(1)
# In [30]: n1
# Out[30]: <BinaryTree.N1 at 0x10853fd30>

print(n1)
# N1: data=1

n2 = N2(2)
# In [33]: n2
# Out[33]: N2: data=2

print(n2)
# N2: data=2

n3 = N3(3)
# Out[36]: N3 repr: data=3

print(n3)
# N3 str: data=3
'''
```

## How to print in string formant

```python
'{{ Test-{} }}'.format('output')

# output: { Test-output }
```

## 遍历子目录

```python
import os

for root, dirs, files in os.walk('.'):
    for sub in files:
        print('name: %s' %(os.path.join(root, sub)))

# 或者也可以使用 glob
import glob

glob.glob('./**/*.png', recursive=True)
```
