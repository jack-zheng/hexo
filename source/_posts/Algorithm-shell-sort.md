---
title: 希尔排序
date: 2021-05-04 13:05:25
categories:
- 算法
tags:
- 希尔排序
---

希尔排序是快排的一个变种，是首个突破 O(n<sup>2</sup>) 的排序算法。但是为什么说他快，看了一些知乎上的解释，这是一种实现简单，但是证明很难的算法。要用到很多数学概念，这里就算了。

算法说明：

1. 取一个固定的步长(通常为待排数组长度的一半), 将数组分组，进行插入排序。
   1. 这里的分组并不是对半分，举例如下：假设有数组 [0, 1, 2, 3, 4, 5, 6, 7] 取步长 4，则 0，4 为一组，1，5为一组依次类推
   2. 分组后的排序，因为只有两个数比较，我也看到有直接用比较之后交换的，不一定是插入排序
2. 将固定步长取半重复之前的运算规则
3. 直到排序完步长为 1 的情况，排序结束

时间复杂度：

平均和最好： O(nlong<sup>n</sup>)
最坏： O(n<sup>2</sup>)

## 实现

说实话，道理我都懂，但是在看 Java 版本的希尔排序实现的时候，我看是看了半天，加上搜索视频教程和本地 debug 才理解下来的，他别是在这个交替进行的点，奇数个元素分组规则上，卡了挺久的。

```java
public class ShellSort {
    public static void main(String[] args) {
        int[] sample = new Random().ints(0, 100).limit(11).toArray();
        System.out.println("Origin: " + Arrays.toString(sample));
        shellSort(sample);
        System.out.println("After:  " + Arrays.toString(sample));
    }

    private static void shellSort(int[] sample) {
        // 步长操作
        for (int step = sample.length / 2; step > 0; step /= 2) {
            //分组进行插入排序，这里需要注意的是，这个插入排序是交替进行的，这里困惑了很久
            for (int i = step; i < sample.length; i++) {
                // 以步长为基本单位做插入排序
                int j = i - step;
                // j >= 0, 0 的时候也是合法的
                while (j >= 0 && sample[j] > sample[j + step]) {
                    // 典型的通过中间变量交换值的逻辑
                    int temp = sample[j + step];
                    sample[j + step] = sample[j];
                    sample[j] = temp;
                    j = j - step;
                }
            }
        }
    }
}

// Origin: [84, 32, 55, 3, 57, 68, 75, 71, 12, 43, 50]
// After:  [3, 12, 32, 43, 50, 55, 57, 68, 71, 75, 84]
```

## 细节分析

以 [84, 32, 55, 3, 57, 68, 75, 71, 12, 43, 50] 为例子分析

第一次分组，对半分，由于元素格式 11 个，步长 5，将整个数组分为了五组

|  num  |  elements  |
| :---: | :--------: |
|   1   | 84, 68, 50 |
|   2   |   32, 75   |
|   3   |   55, 71   |
|   4   |   3, 12    |
|   5   |   57, 43   |

我一开始以为代码实现的时候会先将 1 组中的各个元素抽出来，然后做一次插入排序，然后进行下第二组排序，一次类推。但实际上，在实现过程中，他会先将一组中第1，2 个元素用插入排序排完，然后直接跳到第二组，对1，2号元素排序，依次类推。等五个组的1，2 号元素都排完了，再回到第一组，对3号元素进行插入排序