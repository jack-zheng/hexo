---
title: 基数排序
date: 2021-05-10 18:52:52
categories:
- 算法
tags:
- 基数排序
- radix sort
---

算法描述：

1. 取得数组中的最大数，并取得位数
2. arr为原始数组，从最低位开始取每个位组成radix数组
3. 对radix进行计数排序（利用计数排序适用于小范围数的特点）

稳定，时间复杂度 O(d*2n)

## 课外小知识

java 中的 n 次方计算

```java
System.out.println(Math.pow(2,3));
System.out.println(Math.pow(3,2));
// 8.0
// 9.0
```

## 实现

```java
public class RadixSort {
    public static void main(String[] args) {

        int[] arr = new int[]{67, 75, 61, 18, 60, 30, 90, 19, 89, 35};
        System.out.println(Arrays.toString(arr));
        int max = getMaxDigit(arr);
        System.out.println("Max: " + max);
        int[] ret = radixSort(arr, max);
        System.out.println(Arrays.toString(ret));
    }

    private static int[] radixSort(int[] arr, int length) {
        int mod = 10;
        int dev = 1;
        int[] result = new int[arr.length];
        int[] bucket = new int[10];
        for (int i = 0; i < length; i++, mod *= 10, dev *= 10) {
            int division = (int) Math.pow(10, i);
            for (int k : arr) {
                int remainder = k / division % 10;
                bucket[remainder]++;
            }
            System.out.println("bucket: " + Arrays.toString(bucket));

            for (int j = 1; j < bucket.length; j++) {
                bucket[j] = bucket[j] + bucket[j - 1];
            }
            System.out.println("bucket: " + Arrays.toString(bucket));

            for (int j = arr.length - 1; j >= 0; j--) { // 等于 0 时也要计算
                System.out.println("arr[j]: " + arr[j] + "; division: " + division + "; reminder: " + arr[j] / division % 10);
                int index = bucket[arr[j] / division % 10];
                System.out.println("index: " + index);
                result[index - 1] = arr[j];
                bucket[arr[j] / division % 10]--;
                System.out.println(Arrays.toString(result));
            }
            System.arraycopy(result, 0, arr, 0, arr.length);
            System.out.println("Reset arr: " + Arrays.toString(arr));
            Arrays.fill(bucket, 0);
            System.out.println("Reset ret: " + Arrays.toString(result));
        }
        return result;
    }

    private static int getMaxDigit(int[] arr) {
        // get max value
        int max = arr[0];
        for (int i : arr) {
            if (max < i)
                max = i;
        }

        // get max value length
        return String.valueOf(max).length();
    }
}
```

这种从低位开始的排序叫做：Least significant digital

## MSD

Most signficat digital

## 考虑负数

## 字符串排序