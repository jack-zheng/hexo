---
title: 计数排序
date: 2021-05-07 18:47:57
categories:
- 算法
tags:
- 计数排序
---

计数排序不是比较排序，他将数据的值转化为键存储在额外空间中，时间复杂度是线性的。他要求输入的数据必须是有确定范围的整数。

算法说明：

1. 找出待排数组中的最大，最小元素
2. 统计数组中每个值为 i 的元素出现的次数，存入索引数组的第 i 相中
3. 对所有计数累加
4. 反向填充目标数组：元素 i 放到新数组第 C[i] 位置，美方一个元素 C[i] - 1

时间/空间复杂度：O(n+k)

## 普通实现

```java
public class CountingSort {
    public static void main(String[] args) {
        int[] arr = new Random().ints(0, 100).limit(10).toArray();
        System.out.println("Origin: " + Arrays.toString(arr));
        int[] result = countingSort(arr);
        System.out.println("After:  " + Arrays.toString(result));
    }

    private static int[] countingSort(int[] arr) {
        // 遍历数组，得到最大值
        int max = arr[0];  // 这里之前写的是 max = 0; 如果原数组包含负数就挂了。。
        for (int j : arr) {
            if (max < j) {
                max = j;
            }
        }
        
        // 根据这个值新建一个用于计数的数组, 比如原数组最大值为 2，新建的计数数组为 [0, 1, 2]，所以声明长度的时候为 max + 1
        int[] bucket = new int[max + 1];
        // 再次遍历数组，将对应的 bucket 下标坐 ++ 操作
        for (int i : arr) {
            bucket[i]++;
        }

        // 遍历 bucket 数组，将统计结果塞到新建结果集中
        int[] result = new int[arr.length];
        int index = 0;
        for (int i = 0; i < bucket.length; i++) {
            while (bucket[i] > 0) {
                result[index] = i;
                index++;
                bucket[i]--;
            }
        }
        return result;
    }
}
// Origin: [36, 7, 70, 84, 69, 65, 88, 11, 16, 87]
// After:  [7, 11, 16, 36, 65, 69, 70, 84, 87, 88]
```

步骤和思路很都简单明了，容易理解

## 稳定性

但是上面的实现有一个弊端，就是，他是不稳定的。。。。

所谓的稳定性，简单来说，如果原数组中，有两个相等的值 arr[i], arr[j]，在排序后他们的前后顺序还保持一致，那么就是稳定的。

好处：稳定的算法，第一个键排序的结果可以当作第二次排序的输入使用

PS：这个说法我大致有感觉，但是具体还是没把握。。。

## 保证稳定性的实现

```java
public class CountingSort {
    public static void main(String[] args) {
        int[] arr = new Random().ints(0, 10).limit(10).toArray();
        System.out.printf("%-10s: %s%n", "Origin", Arrays.toString(arr));
        int[] result = countingSort(arr);
        System.out.printf("%-10s: %s%n", "After", Arrays.toString(result));
    }

    private static int[] countingSort(int[] arr) {
        int max = getMax(arr);

        // 根据这个值新建一个用于计数的数组, 比如原数组最大值为 2，新建的计数数组为 [0, 1, 2]，所以声明长度的时候为 max + 1
        int[] bucket = new int[max + 1];
        // 再次遍历数组，将对应的 bucket 下标做 ++ 操作
        for (int i : arr) {
            bucket[i]++;
        }
        System.out.printf("%-10s: %s%n", "Bucket", Arrays.toString(bucket));

        // 新建一个数组存储下标结束的值，用于保证排序稳定性
        int[] endIndex = new int[bucket.length];
        int sum = 0;
        for (int i = 0; i < endIndex.length; i++) {
            sum += bucket[i];
            endIndex[i] = sum;
        }
        System.out.printf("%-10s: %s%n", "endIndex", Arrays.toString(endIndex));

        // 遍历 bucket 数组，将统计结果塞到新建结果集中
        // 这一段实现踩了好多坑
        //  1. i 结束条件 >= 0，第一次实现没有把 = 纳入
        //  2. 后两句的 endIndex[arr[i]] 老是写成 endIndex[i]，对应关系还是没有写清楚
        int[] result = new int[arr.length];
        for (int i = arr.length - 1; i >= 0; i--) {
            result[endIndex[arr[i]] - 1] = arr[i];
            endIndex[arr[i]]--;
        }
        return result;
    }

    private static int getMax(int[] arr) {
        // 遍历数组，得到最大值
        int max = arr[0];
        for (int j : arr) {
            if (max < j) {
                max = j;
            }
        }
        return max;
    }
}

// Origin    : [4, 6, 8, 3, 8, 0, 0, 4, 3, 5]
// Bucket    : [2, 0, 0, 2, 2, 1, 1, 0, 2]
// endIndex  : [2, 2, 2, 4, 6, 7, 8, 8, 10]
// After     : [0, 0, 3, 3, 4, 4, 5, 6, 8, 8]
```

## 不是从 0 开始的实现

之前的实现还是有瑕疵，作为统计数组的下标都是从 0 开始的，在一开始统计最值的时候可以同时统计最小值，缩小样本空间，减少空间消耗

在之前的练习中理解了下标，值之间的对应关系，修改一下并不十分麻烦，挺好理解的。

有的示例中会把 bucket 和 endIndex 合并来减少内存开销，我觉得都 OK，这样写思路会清晰一点

```java
public class CountingSort {
    public static void main(String[] args) {
        int[] arr = new Random().ints(0, 10).limit(10).toArray();
        System.out.printf("%-10s: %s%n", "Origin", Arrays.toString(arr));
        int[] result = countingSort(arr);
        System.out.printf("%-10s: %s%n", "After", Arrays.toString(result));
    }

    private static int[] countingSort(int[] arr) {
        // 遍历数组，得到最值
        int max = arr[0];
        int min = arr[0];
        for (int j : arr) {
            if (max < j) {
                max = j;
            }
            if (min > j) {
                min = j;
            }
        }

        // 根据最值新建一个用于计数的数组, 比如原数组最大值为 4，最小值为 2，新建的计数数组为 [2，3，4]，所以声明长度的时候为 max - min + 1
        int[] bucket = new int[max - min + 1];
        // 再次遍历数组，将对应的 bucket 下标做 ++ 操作
        for (int i : arr) {
            bucket[i - min]++;
        }
        System.out.printf("%-10s: %s%n", "Bucket", Arrays.toString(bucket));

        // 新建一个数组存储下标结束的值，用于保证排序稳定性
        int[] endIndex = new int[bucket.length];
        int sum = 0;
        for (int i = 0; i < endIndex.length; i++) {
            sum += bucket[i];
            endIndex[i] = sum;
        }
        System.out.printf("%-10s: %s%n", "endIndex", Arrays.toString(endIndex));

        // 遍历 bucket 数组，将统计结果塞到新建结果集中
        int[] result = new int[arr.length];
        for (int i = arr.length - 1; i >= 0; i--) {
            result[endIndex[arr[i] - min] - 1] = arr[i];
            endIndex[arr[i] - min]--;
        }
        return result;
    }
}
// Origin    : [5, 1, 7, 6, 1, 0, 5, 2, 0, 3]
// Bucket    : [2, 2, 1, 1, 0, 2, 1, 1]
// endIndex  : [2, 4, 5, 6, 6, 8, 9, 10]
// After     : [0, 0, 1, 1, 2, 3, 5, 5, 6, 7]
```

## 继续演进

既然都写到这里了，直接在源代码基础上，直接将 bucket 和 endIndex 合并了，so easy

```java
public class CountingSort {
    public static void main(String[] args) {
        int[] arr = new Random().ints(0, 10).limit(10).toArray();
        System.out.printf("%-10s: %s%n", "Origin", Arrays.toString(arr));
        int[] result = countingSort(arr);
        System.out.printf("%-10s: %s%n", "After", Arrays.toString(result));
    }

    private static int[] countingSort(int[] arr) {
        // 遍历数组，得到最值
        int max = arr[0];
        int min = arr[0];
        for (int j : arr) {
            if (max < j) {
                max = j;
            }
            if (min > j) {
                min = j;
            }
        }

        // 根据最值新建一个用于计数的数组, 比如原数组最大值为 4，最小值为 2，新建的计数数组为 [2，3，4]，所以声明长度的时候为 max - min + 1
        int[] bucket = new int[max - min + 1];
        // 再次遍历数组，将对应的 bucket 下标做 ++ 操作
        for (int i : arr) {
            bucket[i - min]++;
        }
        System.out.printf("%-10s: %s%n", "Bucket", Arrays.toString(bucket));

        // 新建一个数组存储下标结束的值，用于保证排序稳定性
        int sum = 0;
        for (int i = 0; i < bucket.length; i++) {
            sum += bucket[i];
            bucket[i] = sum;
        }
        System.out.printf("%-10s: %s%n", "Index", Arrays.toString(bucket));

        // 遍历 bucket 数组，将统计结果塞到新建结果集中
        int[] result = new int[arr.length];
        for (int i = arr.length - 1; i >= 0; i--) {
            result[bucket[arr[i] - min] - 1] = arr[i]; // 下标计算注意一下，arr[i] - min 取出 bucket 存储的位置值，但是这个值比下标大1
            bucket[arr[i] - min]--;
        }
        return result;
    }
}
// Origin    : [1, 4, 6, 3, 9, 5, 0, 0, 9, 8]
// Bucket    : [2, 1, 0, 1, 1, 1, 1, 0, 1, 2]
// Index     : [2, 3, 3, 4, 5, 6, 7, 7, 8, 10]
// After     : [0, 0, 1, 3, 4, 5, 6, 8, 9, 9]
```