---
title: "[LC279] Perfect Squares / 动态规划和BFS"
date: 2019-03-05
tags: [leetcode, 动态规划, BFS]
categories:
  - CS
  - LeetCode
---

第一次做这道题的时候根据标签🏷很容易想到了BFS的解法，而第二次看的时候却想不起来了。将动态规划应用于这道题的时候，使用的小技巧「看似遍历但实则复杂度并不大」。

<!-- more -->

# 问题描述

[279. Perfect Squares](https://leetcode.com/problems/perfect-squares/)

Given a positive integer n, find the least number of perfect square numbers (for example, 1, 4, 9, 16, ...) which sum to n.

**Example:**
```
Input: n = 13
Output: 2
Explanation: 13 = 4 + 9.
```

# 解法

## BFS

这个我第二次遇到的时候已经想不起来的 BFS 确实很有意思。它将 `0..n` 作为树节点，将问题转化为求 `n` 到 `0` 的最短路径。而 BFS 的一类应用就是最短路径求解。以节点 `0` 为例，它的相邻边有 1, 4, 9, ... 当然，过大的节点我们实际是不需要考虑的。

实际代码中，便利方向为 `n -> 0`。这样，当遍历到 `i` 时，只需考虑 `1..sqrt(i)`，将它们加入队列即可。

```java
public int numSquares(int n) {
    int level = 0;
    Queue<Integer> q = new LinkedList<>();
    q.offer(n);

    while (!q.isEmpty()) {
        int size = q.size();
        while (size > 0) {
            int c = q.poll();
            if (c == 0) {
                return level;
            }

            for (int i = 1; i <= Math.sqrt(c); i++) {
                q.offer(c - i * i);
            }
            size--;
        }
        level++;
    }
    return level;
}
```

## 动态规划

动态规划的思路是：`dp[0..n]` 中的 `dp[i]` 表示平方数之和为 `i` 的最小数目。也即，从 `0` 遍历到 `n` 后，`dp[n]` 即为所求结果，在这一过程中，每个步骤需要计算 `dp[i]` 都需要借助已经计算好的数字。例如，计算 `dp[18]` 时，需要找到 `{dp[18 - 1], dp[18 - 4], dp[18 - 9], dp[18 - 16]}` 中的最小值。

```c++
class Solution {
public:
    int numSquares(int n) {
        if (n <= 0) {
            return 0;
        }
        
        // Note that cntPerfectSquares[0] is 0.
        vector<int> cntPerfectSquares(n + 1, INT_MAX);
        cntPerfectSquares[0] = 0;
        for (int i = 1; i <= n; i++) {
            for (int j = 1; j*j <= i; j++) {
                cntPerfectSquares[i] = 
                    min(cntPerfectSquares[i], cntPerfectSquares[i - j*j] + 1);
            }
        }
        
        return cntPerfectSquares.back();
    }
};
```