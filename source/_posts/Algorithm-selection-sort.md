---
title: 选择排序
date: 2021-04-20 13:13:54
categories:
- 算法
tags:
- 选择排序
---

算法描述

1. 首先在未排序序列中找到最小（大）元素，存放到排序序列的起始位置。
2. 再从剩余未排序元素中继续寻找最小（大）元素，然后放到已排序序列的末尾。
3. 重复第二步，直到所有元素均排序完毕。

## 实现

```java
public class SelectionSortDemo {
    public static void main(String[] args) {
        int[] sample = new Random().ints(0, 100).limit(10).toArray();
        System.out.println("Origin: " + Arrays.toString(sample));

        selectionSort(sample);
        System.out.println("After:  " + Arrays.toString(sample));
    }

    private static void selectionSort(int[] sample) {
        for (int i = 0; i < sample.length - 1; i++) {
            int max_idx = i;
            for (int j = i; j < sample.length; j++) { // 这里注意一下，是对整个数组做 selection，所以为 sample.length, 如果写为 sample.length-1 则最后一个元素会跳过排序
                if (sample[max_idx] < sample[j]) {
                    max_idx = j;
                }

            }
            if (max_idx != i) {
                swap(sample, i, max_idx);
            }
        }
    }

    private static void swap(int[] sample, int i, int j) {
        int tmp = sample[i];
        sample[i] = sample[j];
        sample[j] = tmp;
    }
}
// Origin: [32, 47, 97, 16, 3, 81, 61, 78, 43, 65]
// After:  [97, 81, 78, 65, 61, 47, 43, 32, 16, 3]
```
