---
title: Algorithm merge sort
date: 2021-04-28 14:02:39
categories:
- 算法
tags:
- 归并排序
---

采用分治(Divide and Conquer)的思想，解题思路如下：

1. 先将整个数组分为有序数组(对半分，直到长度为1)
2. 两两合并，并确保合并后的数组有序 - 二路归并
3. 重复直到所有数组合并完成