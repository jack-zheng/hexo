---
title: Algorithm selection sort
date: 2021-04-20 13:13:54
categories:
- 算法
tags:
- 选择排序
---

算法描述

1. 从整个数组总选择最大的元素，和首元素交换
2. 按照上面的逻辑，对剩余 n-1 个元素排序，放到第二个位置
3. 依次进行 n-1 次循环，得到有序数组

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
