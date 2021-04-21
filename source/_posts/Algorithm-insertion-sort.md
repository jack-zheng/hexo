---
title: 插入排序
date: 2021-04-20 10:57:12
categories:
- 算法
tags:
- 插入排序
---

* 参考[动图](https://zhuanlan.zhihu.com/p/122293204)

插入排序是一个很基础的排序方法，基本解法思路为：

将待排序序列第一个元素看做一个有序序列，把第二个元素到最后一个元素当成是未排序序列。从头到尾依次扫描未排序序列，将扫描到的每个元素插入有序序列的适当位置。（如果待插入的元素与有序序列中的某个元素相等，则将待插入元素插入到相等元素的后面。）

时间复杂度：O(n<sup>2</sup>)

## 实现

```java
/**
 * 1. 第一次计算，拿二号元素和一号元素比较，如果二号小于一号，交换位置。计算后前两个元素为规则元素
 * 2. 第二次计算，拿三号元素依次和二号，一号做比较，如果三号小于其中某个元素，交换位置
 * 3. 重复以上规则对剩余元素进行排序
 */
public class InsertionSortDemo {
    public static void main(String[] args) {
        int[] sample = new Random().ints(0, 100).limit(10).toArray();
        System.out.println("Origin: " + Arrays.toString(sample));

        insertionSort(sample);
        System.out.println("After:  " + Arrays.toString(sample));
    }

    private static void insertionSort(int[] sample) {
        for (int i = 1; i < sample.length; i++) {
            for (int j = i; j > 0; j--) {
                if (sample[j] < sample[j-1]) {
                    swap(sample, j, j-1);
                }
            }
        }
    }

    private static void swap(int[] sample, int pos1, int pos2) {
        int tmp = sample[pos1];
        sample[pos1] = sample[pos2];
        sample[pos2] = tmp;
    }
}
// Origin: [54, 66, 34, 28, 0, 10, 6, 61, 42, 97]
// After:  [0, 6, 10, 28, 34, 42, 54, 61, 66, 97]
```