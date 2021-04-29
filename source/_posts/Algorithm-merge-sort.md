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

时间复杂度 O(nlog<sup>n</sup>)

## 实现

```java
import java.util.Arrays;
import java.util.Random;

public class MergeSort {
    public static void main(String[] args) {
        int[] sample = new Random().ints(0, 100).limit(10).toArray();
        System.out.println("Origin: " + Arrays.toString(sample));

        mergeSort(sample, 0, sample.length - 1);
        System.out.println("Origin: " + Arrays.toString(sample));
    }

    private static void mergeSort(int[] sample, int low, int high) {
        // 结束条件
        if (low >= high)
            return;
        // 对半分
        int mid = (low + high) / 2;
        // 分别对两个子数组做归并排序
        mergeSort(sample, low, mid);
        mergeSort(sample, mid + 1, high);
        // 合并两个子数组
        merge(sample, low, mid, high);
    }

    private static void merge(int[] sample, int low, int mid, int high) {
        // 声明零时数组存放排序结果
        int[] tmp = new int[high - low + 1];
        int i = low, j = mid + 1;
        int index = 0;
        // 比较两个子数组的最值，并放到临时数组中
        while (i <= mid && j <= high) {
            if (sample[i] < sample[j]) {
                tmp[index++] = sample[i++];
            } else {
                tmp[index++] = sample[j++];
            }
        }

        // 将子数组剩余值放入临时数组中，经过上一步之后其中一个数组已经空了，所以下面两个 while 先后顺序没关系
        while (i <= mid) {
            tmp[index++] = sample[i++];
        }

        while (j <= high) {
            tmp[index++] = sample[j++];
        }

        // 将临时数组中的结果覆盖到原数组中
        for (int pos = 0; pos < tmp.length; pos++) {
            sample[low + pos] = tmp[pos];
        }
    }
}
```