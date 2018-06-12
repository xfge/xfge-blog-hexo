---
title: "[LC53] Maximum Subarray / 动态规划 / 分治"
date: 2018-06-12
tags: [leetcode, 动态规划, 分治]
categories:
  - CS
  - LeetCode
---

最大子数组之和问题是 CLRS 上讲解分治算法的一道例题。由 $O(n\log(n))$ 的分治算法再到 $O(n)$ 的动态规划算法，都有很好的示范作用。

<!-- more -->

[Maximum Subarray - LeetCode](https://leetcode.com/problems/maximum-subarray/description/)

> Given an integer array `nums`, find the contiguous subarray (containing at least one number) which has the largest sum and return its sum.

```
Input: [-2,1,-3,4,-1,2,1,-5,4],
Output: 6
Explanation: [4,-1,2,1] has the largest sum = 6.
```

## 动态规划 (Kadane's Algorithm)

可以在线性时间内解决问题。在动态规划算法中如何定义递推关系式很重要。在本题中，定义 `dp[i]` 为 **以`nums[i]`为结尾的串的最大子数组之和**，在从 `0` 到 `i-1` 的遍历过程中只需要维护一个 `globalMax` 就可以在计算完成后得到 `nums` 的最大子数组之和。计算 `dp[i]` 时，没有必要依次比较 `0..i`，`1..i` ... 只需根据公式 `dp[i] = max(dp[i-1] + nums[i], nums[i])` 即可。注意，可以通过反证法证明该结论成立。


## 分治

### 1 CLRS 上的解法

CLRS 在讲解分治问题时将这一问题作为例题。分治算法在计算 `A[low..high]` 的最大子数组之和时，将其分为 `A[low..mid]` 和 `A[mid+1..high]` 两个数组分别计算。在合并时，考虑 `A[low..high]` 的最大子数组必为以下三种情况之一：

* 最大子数组完全落在 `A[low..mid]􏰀` 中。
* 最大子数组完全落在 `A[mid+1..high]􏰀` 中。
* 最大子数组左右端点跨过中间点 `mid]􏰀`。

前两种情况只需递归调用即可；第三种情况需要单独处理。需要从 `mid` 开始分别向两个方向不断求和，分别计算出两个方向的最大值（两者的端点其中之一均为中间点），求和就得到第三种情况的最大值。

取三种情况计算出的最大值的最大者为 `A[low..high]` 的最大子数组之和。

<div align="center"><img src="{{ site.baseurl }}/images/2018/06/find-max-crossing-subarray.png" width="60%"></div>
<div align="center"><img src="{{ site.baseurl }}/images/2018/06/find-maximum-subarray.png" width="80%"></div>

### 2 一种优化的分治方法

以下方法给出了一个优化后的分治算法，可以在 `O(n)` 时间复杂度内解决。来源于 [LeetCode 讨论版块](https://leetcode.com/problems/maximum-subarray/discuss/20360)。

其中，
* `mx` 表示当前数组的最大子数组之和
* `lmx` 表示以当前数组最左元素为左端点的所有子数组的最大和
* `rmx` 表示以当前数组最右元素为右端点的所有子数组的最大和
* `sum` 表示当前数组元素之和。

```c++
    void maxSubArray(vector<int>& nums, int l, int r, int& mx, int& lmx, int& rmx, int& sum) {
        if (l == r) {
            mx = lmx = rmx = sum = nums[l];
        }
        else {
            int m = (l + r) / 2;
            int mx1, lmx1, rmx1, sum1;
            int mx2, lmx2, rmx2, sum2;
            maxSubArray(nums, l, m, mx1, lmx1, rmx1, sum1);
            maxSubArray(nums, m + 1, r, mx2, lmx2, rmx2, sum2);
            mx = max(max(mx1, mx2), rmx1 + lmx2);
            lmx = max(lmx1, sum1 + lmx2);
            rmx = max(rmx2, sum2 + rmx1);
            sum = sum1 + sum2;
        }
    }
    int maxSubArray(vector<int>& nums) {
        if (nums.size() == 0) {
            return 0;
        }
        int mx, lmx, rmx, sum;
        maxSubArray(nums, 0, nums.size() - 1, mx, lmx, rmx, sum);
        return mx;
    }
```

在 $O(n\log(n))$ 的算法中，包括了 $O(n)$ 的计算 crossing mid 的情况。而在此算法中，这一种情况所对应的最大子数组之和由 `rmx1 + lmx2` 得到——如果已知分割后的两个数组的 `lmx` 和 `rmx`，就可以简化计算。利用已知信息计算 crossing mid 对应的最大子数组之和。

分析该算法效率，根据其递归关系式：$T(n) = 2T(n / 2) + O(1)$，结合**主定理**，得出其时间复杂度是 $O(n)$。