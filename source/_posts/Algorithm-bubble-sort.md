---
title: 冒泡排序
date: 2021-04-20 13:05:19
- 算法
tags:
- 冒泡排序
---

* [示例带图](https://zhuanlan.zhihu.com/p/42586566)

算法描述

1. 遍历数组，比较相邻的两个元素的大小，如果前一个比后一个大就交换位置，如此循环，最后一个位置即最大值
2. 重复上述过程，对前 n-1 个元素排序
3. 重复直到所有元素都完成排序

操作没问题，但是你难道不觉得，这个比较中间过程中的交换过程很浪费资源吗？为了省去中间的交换过程，我们有了选择排序。

时间复杂度：O(n<sup>2</sup>)

## 实现

```java
/**
 * 1. 遍历数组，比较相邻的两个元素的大小，如果前一个比后一个大就交换位置，如此循环，最后一个位置即最大值
 * 2. 重复上述过程，对前 n-1 个元素排序
 * 3. 重复直到所有元素都完成排序
 */
public class BubbleSortDemo {
    public static void main(String[] args) {
        int[] sample = new Random().ints(0, 100).limit(10).toArray();
        System.out.println("Origin: " + Arrays.toString(sample));
        bubbleSort(sample);
        System.out.println("After:  " + Arrays.toString(sample));
    }

    private static void bubbleSort(int[] sample) {
        for (int i = sample.length - 1; i > 0; i--) { // 这里使用 i=length-1 的表达方式，第二层直接 j < i, 书写起来比较好看
            for (int j = 0; j < i; j++) {
                if (sample[j] > sample[j + 1]) {
                    swap(sample, j, j + 1);
                }
            }
        }
    }

    private static void swap(int[] arr, int pos1, int pos2) {
        int tmp = arr[pos1];
        arr[pos1] = arr[pos2];
        arr[pos2] = tmp;
    }
}
// Origin: [62, 39, 37, 64, 84, 27, 68, 90, 55, 63]
// After:  [27, 37, 39, 55, 62, 63, 64, 68, 84, 90]
```