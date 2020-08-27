---
title: Python 优雅的换行
date: 2020-07-08 15:16:28
categories:
- 编程
tags:
- python
---

记录一下查了无数遍的换行方法备用，总结一下就是使用 '' + \ + '' 类似的语法做链接，只可使用 'xxxx\xxx' 的话会出现空格

## 无缝连接

```python
a = '1111111'\
    '2222222'\
    '3333333'

print(a)
# '111111122222223333333'

print('aaaaaaaaa'
    'bbbbbbbbb'
    'ccccccccc')
# aaaaaaaaabbbbbbbbbccccccccc

print('aaaaaaaaa'\
        'bbbbbbbbb'\
        'ccccccccc')
# aaaaaaaaabbbbbbbbbccccccccc
```

## 有缝连接

```python
a = '''11111111
       22222222
       33333333'''
print(a)
# '11111111\n       22222222\n       33333333'

a = '11111111\
     22222222\
     33333333'
print(a)
# '11111111     22222222     33333333'

print('''55555555555
        66666666666
        77777777777''')
# 55555555555
#                66666666666
#                77777777777
```
