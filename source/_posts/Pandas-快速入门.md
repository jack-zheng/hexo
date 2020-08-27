---
title: Pandas 快速入门
date: 2020-06-29 17:08:08
categories:
- 编程
tags:
- python
- 数据分析
---

记录 Pandas 常用方法作为快速入门导航

## 常用方法

```python
path = '/Users/i306454/Downloads/dump.json'
dump = pandas.read_json(path)
# 输出一个二维数组

# 显示每一列的基本信息，包括类型，是否空等
dump.info()
# <class 'pandas.core.frame.DataFrame'>
# RangeIndex: 18846 entries, 0 to 18845
# Data columns (total 6 columns):
#  #   Column        Non-Null Count  Dtype
# ---  ------        --------------  -----
#  0   subject       18846 non-null  object
#  1   receive_from  18846 non-null  object
# ...

# 显示可计算列的统计信息，最值，方差，分布等
dump.describe()
#                size
# count  1.884600e+04
# mean   5.790807e+04
# ...

# 显示前三条，用作预览
dump.head(3)

# 取值 loc/iloc, loc 通过名字，iloc 通过数字标签
# 取1,2 行, size 到 receive_data 矩阵, 和 python 的语法不一样的是这个表达式会包含第二行
dump.loc[1:2, 'size':]
#     size         receive_date
# 1  11593  2015-08-06T08:36:19
# 2  15863  2017-08-06T08:09:36

# 直接到方括号可以选择列
dump['size']

# 查看矩阵大小
dump.shape
# (15, 6)

# 查看 size > 3M 的行
dump[dump['size'] > 2000000]

# 画直方图, bins 如果是数字的话表示你想分成几个 bar
dump['size'].hist(bins=20)

# pandas 的直方图可选项比较少，画图可以用 matplotlib
import matplotlib.pyplot as plt

x = [2500*sub for sub in range(0, 22)]
plt.figure(figsize=(20, 8), dpi=80)
plt.hist(main['size'].values, bins=range(0, 50000+1, 2500))
plt.grid()
plt.xticks(x, ['{:.1f}KB'.format(sub/1000) for sub in x])
plt.show()

# 生产次云, 通过 scale 控制清晰度
titles = dump['subject']
titletxt = ' '.join(titles)
wordcloud = WordCloud(scale=10).generate(titletxt)
plt.figure(figsize=(20, 8), dpi=80)
plt.imshow(wordcloud, interpolation='bilinear')
plt.axis("off")
plt.show()
```

## 修改 pandas describe 格式

```python
pd.set_option('display.float_format', lambda x: '{:.2f}KB'.format(x/1000))
dump.describe()
#            size
# count   18.85KB
# mean    57.91KB
# std    143.28KB
# min      3.21KB
# 25%     11.09KB
# 50%     23.72KB
# 75%     42.62KB
# max   5235.50KB
```
