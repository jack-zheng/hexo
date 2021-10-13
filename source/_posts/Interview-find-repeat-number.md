---
title: 找出数组中重复的数字
date: 2021-10-11 14:30:29
categories:
- Leetcode
tags:
- 应试
- 数组
- 剑指 offer
---

在一个长度为 n 的数组 nums 里的所有数字都在 0～n-1 的范围内。数组中某些数字是重复的，但不知道有几个数字重复了，也不知道每个数字重复了几次。请找出数组中任意一个重复的数字。

示例 1：

输入：
[2, 3, 1, 0, 2, 5, 3]
输出：2 或 3 
 
## 解题 1

通过 Set 的特性解题，将元素一次存入 set, 如果 add 失败，则为重复元素，返回

```java
class Solution {

    public static void main(String[] args) {
        int[] arr = new int[]{ 2, 3, 1, 0, 2, 5, 3 };
        System.out.println(new Solution().findRepeatNumber(arr));
    }

    public int findRepeatNumber(int[] nums) {
        Set<Integer> set = new HashSet<>();
        for (int num : nums) {
            if (!set.add(num)) {
                return num;
            }
        }
        return -1;
    }
}
```

我很喜欢这个解法，很直接了当，时间/空间 复杂度都是 O(n)

## 解题 2

活用题中另一个条: 件数字范围 0-(n-1), 如果将数组中的值一次排开，重复数字会在相同的下标下有冲突这一特性解题，时间复杂度 O(n), 空间复杂度 O(1)

```java
class Solution2 {

    public static void main(String[] args) {
        int[] arr = new int[]{2, 3, 1, 0, 2, 5, 3};
        System.out.println(new Solution2().findRepeatNumber(arr));
    }

    public int findRepeatNumber(int[] nums) {
        int i = 0;
        while(i < nums.length) {
            if (i == nums[i]) {
                i ++;
                continue;
            }
            if (nums[i] == nums[nums[i]]) return nums[i];

            int tmp = nums[i];
            nums[i] = nums[tmp]; // 使用 tmp 做索引，nums[i] 会改变
            nums[tmp] = tmp;
        }
        return -1;
    }
}
```