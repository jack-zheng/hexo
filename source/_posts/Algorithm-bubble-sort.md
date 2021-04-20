---
title: Algorithm bubble sort
date: 2021-04-20 13:05:19
- 算法
tags:
- 插入排序
---

* [示例带图](https://zhuanlan.zhihu.com/p/42586566)

算法描述

1. 从第一个元素开始，一次比较相邻两个元素大小，根据比较条件交换。经过一次遍历后，可以得到队尾的值即最值
2. 按照上面的步骤，对剩下的 n-1 个元素做同样的操作

时间复杂度：O(n<sup>2</sup>)

## 实现

```java
public class BubbleSortDemo {
    public static void main(String[] args) {
        int[] sample = new Random().ints(0, 100).limit(10).toArray();
        System.out.println("Origin: " + Arrays.toString(sample));
        bubbleSort(sample);
        System.out.println("After:  " + Arrays.toString(sample));
    }

    private static void bubbleSort(int[] arr) {
        for (int i = 0; i < arr.length - 1; i++) {
            for (int j = 0; j < arr.length - 1 - i; j++) {
                if (arr[j] > arr[j + 1]) {
                    swap(arr, j, j + 1);
                }
            }
        }
    }

    private static void swap(int[] arr, int idx1, int idx2) {
        int tmp = arr[idx1];
        arr[idx1] = arr[idx2];
        arr[idx2] = tmp;
    }
}
// Origin: [62, 39, 37, 64, 84, 27, 68, 90, 55, 63]
// After:  [27, 37, 39, 55, 62, 63, 64, 68, 84, 90]
```