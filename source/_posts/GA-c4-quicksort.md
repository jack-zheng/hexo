---
title: 快速排序
date: 2021-04-08 12:45:01
categories:
 - Grokking Algorithms
tags:
- 快速排序
- quick sort
---

图解算法第四章 快排 读书笔记

分而治之（divide and conquer，D&C）- 种著名的递归式问题解决方法

## 4.1 分而治之

练习1: 有一块 1680 x 640 的土地，将它等分成一个个正方形，那么正方形面积最大时，边长为多少？

解决方案，根据 欧几里得 算法，我们先将长方形分解成 640x640 + 640x640 + 640x400，然后再将 640x400 的长方形按照同样的方式分解，到最后分解为正方形时就是我们要的答案了

```python
def getSquar(length, width): 
    if length%width == 0: 
        return width 
    else: 
        return getSquar(width, length%width) 

getSquar(1680, 640) 
# 80
```

重申一下D&C的工作原理：

1. 找出简单的基线条件
2. 确定如何缩小问题的规模，使其符合基线条件

练习2: 使用 D&C 的思路计算一个数组的和, sum = first + sum(sec...end)

```python
list = [sub for sub in range(1, 10)]
def sum(list):
    if list:
        return list[0] + sum(list[1:])
    return 0

sum(list)
# 45
```

## 快速排序

练习：准备一个无序数组，使用快排重新排序

快排原理：随机从数组中选择一个数作为基准，然后将数组分为 大于基准的数 + 基准 + 小于基准的数，在按照同样的思路对两堆数进行同样的排序，直到堆的 len < 2 为止排序结束

```python
list = random.sample(range(0, 100), 10)
# [71, 70, 99, 91, 12, 5, 80, 61, 17, 24]

def qsort(list): 
    if len(list) > 2: 
        pivot = list[0] 
        small_partition = [sub for sub in list[1:] if sub < pivot] 
        big_partition = [sub for sub in list[1:] if sub >= pivot] 
        return qsort(small_partition) + [pivot] + qsort(big_partition) 
    return list

# [5, 12, 17, 24, 61, 70, 71, 91, 80, 99]
# 看了下书上的答案基本一致 (●°u°●)​ 」
```

## 4.3 再谈大O表示法

常见大O运行时间

![常见大O运行时间](c4_01.png)

### 4.3.2  平均情况和最糟情况

当最糟情况时，n 个元素的数组，每层分析 n 次，栈深为 n, O(n) x O(n) = O(n<sup>2</sup>)

平均情况时，n 个元素的数组，每层分析 n 次，栈深为 log<sup>n</sup>, 时间复杂度为 O(n) x O(log<sup>n</sup>) = O(nlog<sup>n</sup>)