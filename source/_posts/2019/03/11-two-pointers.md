---
title: "[LeetCode] Two Pointers"
date: 2019-03-11
tags: [leetcode, 双指针]
categories:
  - CS
  - LeetCode
---

本篇总结了双指针算法的应用。

<!-- more -->

# 75. Sort Colors

Given an array with n objects colored red, white or blue, sort them in-place so that objects of the same color are adjacent, with the colors in the order red, white and blue.

Here, we will use the integers 0, 1, and 2 to represent the color red, white, and blue respectively.

**Note:** You are not suppose to use the library's sort function for this problem.

**Example:**

Input: [2,0,2,1,1,0]
Output: [0,0,1,1,2,2]

**Follow up:**

* A rather straight forward solution is a two-pass algorithm using counting sort. First, iterate the array counting number of 0's, 1's, and 2's, then overwrite array with total number of 0's, then 1's and followed by 2's.
* Could you come up with a one-pass algorithm using only constant space?

## Brute Force 计数排序

题目中已经提示了，使用最直接的方法是通过一次循环计数 0、1、2 分别出现的次数，然后按它们的次数给数组赋值。这种方法是两次数组遍历。

## 双指针/多指针：用指针标志特殊位置（边界）

数组 + one-pass 排序很容易联想到双指针。但刚开始想到的是首尾各一个指针，怎样利用这两个指针完成 0、1、2 三种数字的 in-place 摆放。因为是三种数字事实上涉及到数组的三块「分隔」的区域，不经过一次遍历是不可能确定这三块区域的分隔边界。

这里采用的其实是「三指针」，用 [YouTube 上的一个讲解视频](https://www.youtube.com/watch?v=yTwW8WiGrKw) 中的话说，是「用三块挡板分隔 0区域、1区域、未定区域、2区域」，这样在不断移动元素、缩小「未定区域」的范围时，经过一次遍历（每个元素只需访问一次），就能将未定区域缩小为0。由于要确定四块区域，用三个指针可以指向「三块挡板」。这四个区域分别是：

* `[0, i - 1]` 0区域
* `[i, j - 1]` 1区域
* `[j, k]` 未定区域（`j` 一直指向遍历的当前位置）
* `[k + 1, n - 1]` 2区域

在每个时刻（每次移动指针时），都能保证上面的区间与区域的对应性。例如，下面是某个字符串在某时刻的指针位置。

```
    i           k
0 0 1 1 1 . . . . 2 2
          j  
```

事实上，当 `j` 访问到新数字时，
* 如果是 0 可将 `nums[j]` 与 `nums[i]` 互换，相当于扩展了 0 区域，1 区域不变，缩小了未定区域，2 区域不变。
* 如果是 2 可将 `nums[j]` 与 `nums[k]` 互换，相当于 0、1 区域不变，缩小了未定区域和 2 区域，
* 如果是 1 ，只需将 j 前进一位，无需改动 i 和 k，缩小了未定区域，扩展了 1 区域，其他区域不变。

上述每种情况下，都是通过调整 `i`, `j`, `k` 以满足互换后区间区域对应性的恒成立。


```java
    public void sortColors(int[] nums) {
        int n = nums.length;
        int i = 0, j = 0, k = n - 1;

        while (j <= k) {
            if (nums[j] == 0) {
                swap(nums, i++, j++);
            }
            else if (nums[j] == 1) {
                j++;
            }
            else if (nums[j] == 2) {
                swap(nums, j, k--);
            }
        }
    }

    private void swap(int[] nums, int i, int j) {
        int tmp = nums[i];
        nums[i] = nums[j];
        nums[j] = tmp;
    }
```

注意，在初始时，如果不满足区间对应性的话（比如 `0 0 1 ... 1 2` 这种情况），在这段代码中不需要特殊处理，因为：
* 对于前部的 0，会在第一次扫描到 `nums[j] == 0` 时就不断递增指针，直到 i 和 j 都处在第一个非 0 位置。
* 对于后部的 2，会在第一次扫描到 `nums[j] == 2` 时将后部的首个 2 交换到 j 的位置，由于此时已经调整了 k 指针为 k - 1，那么马上（看起来被错误）交换到前面的 2 就会被换到 k - 1 的位置然后继续接下来的操作。每次 k 位置指向 2 时都会进行这样的重复交换，但这保证了代码的简洁。