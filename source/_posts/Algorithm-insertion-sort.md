---
title: Algorithm insertion sort
date: 2021-04-20 10:57:12
categories:
- 算法
tags:
- 插入排序
---

* 参考[动图](https://zhuanlan.zhihu.com/p/122293204)

插入排序是一个很基础的排序方法，基本解法思路为：

1. 准备一个测试样本数组
2. 从第二个开始排序，以第二个为例。二和一比较，如果符合条件，则交换。所以每次处理完后，前 n 个元素都是有序的

时间复杂度：O(n<sup>2</sup>)

## 实现

```java
public class InsertionSortDemo {
    public static void main(String[] args) {
        int[] sample = new int[10];
        Random random = new Random();
        for (int i = 0; i < 10; i++) {
            sample[i] = random.nextInt(100);
        }

        System.out.println("Origin: " + Arrays.toString(sample));

        insertionSort(sample);

        System.out.println("After:  " + Arrays.toString(sample));
    }

    private static void insertionSort(int[] arr) {
        for (int i = 1; i < arr.length; i++) {
            for (int j = i; j > 0; j--) {
                if (arr[j] < arr[j - 1]) {
                    swap(arr, j, j - 1);
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
// Origin: [54, 66, 34, 28, 0, 10, 6, 61, 42, 97]
// After:  [0, 6, 10, 28, 34, 42, 54, 61, 66, 97]
```