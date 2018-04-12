---
layout: post
title: "[LC11/19] Container With Most Water / 双指针"
date: 2017-09-04
tags: [leetcode, 双指针]
categories:
  - CS
  - LeetCode
---

本篇总结了两个使用「双指针」解决问题的方法。都达到了扫描一遍计算结果的效果，区别在于，一个是「相遇问题」，一个是「追及问题」。

<!-- more -->

# 11 Container With Most Water

[Container With Most Water - LeetCode](https://leetcode.com/problems/container-with-most-water/description/)

> Given n non-negative integers a1, a2, ..., an, where each represents a point at coordinate (i, ai). n vertical lines are drawn such that the two endpoints of line i is at (i, ai) and (i, 0). Find two lines, which together with x-axis forms a container, such that the container contains the most water.
Note: You may not slant the container and n is at least 2.


此问题用 Brute Force 方法的时间复杂度是 $O(n^2)$。

## 双指针方法

容易想到「木桶原理」。此题基于两个直觉：两条直线形成的（矩形）面积受制于较短的线；而且，如果长度一定，两条线的距离越远，所形成的面积越大。

用双指针方法解决此问题：
1. 分别初始化`i`和`j`为`0`和`height.length - 1`；
2. 维护一个变量 `maxArea` 来存储当前最大的面积；
3. 在循环的每一步，计算当前由`height[i]`和`height[j]`组成的面积，如果比`maxArea`大，则替换它；
4. 移动 **较短的线所对应的指针**（`i`或`j`）到下一个位置。

## 正确性说明

考虑当前`i`和`j`当前指向最外侧的两线。为了最大化面积，自然想到所取两线的长度应该尽可能长。在某一时刻，假设我们移动 **较长的线所对应的指针** 到下一位置，**不会获得任何增益**（因为面积仍受制于较短的线，面积只会减小不会增加）。但是，**移动较短的线的指针** 尽管减少了宽度，但可能增加面积——因为通过移动较短长度的指针而获得的相对较长的height可以克服由width减小引起的面积减小。

[LeetCode 上的一则讨论](https://leetcode.com/problems/container-with-most-water/discuss/6099)给出了一种更为直观的解释。以下解释所绘制矩阵的「行」指的是第一根线（其长度用LL表示），「列」指的是第二根线（其长度用RL表示）；矩阵中，根据对称性易得，「x」所在的位置不需计算。例子中`n=6`。

从`(1,6)`开始计算，**如果`LL<RL`，那么`(1,6)`左侧所指的面积不需计算**，因为它们必将比此时计算的面积更小。
```java
  1 2 3 4 5 6
1 x ------- o
2 x x
3 x x x
4 x x x x
5 x x x x x
6 x x x x x x
```

下面计算`(2,6)`，此时 **如果`LL>RL`，那么`(2,6)`以下所指的面积不需计算**。
```java
  1 2 3 4 5 6
1 x ------- o
2 x x       o
3 x x x     |
4 x x x x   |
5 x x x x x |
6 x x x x x x
```

不论 `o` 的走向如何，只要沿途不断找出最大值，最终只需`n-1`次计算即可求出最大面积。
```java
  1 2 3 4 5 6
1 x ------- o
2 x x - o o o
3 x x x o | |
4 x x x x | |
5 x x x x x |
6 x x x x x x
```

# 19 Remove Nth Node From End of List

[Remove Nth Node From End of List - LeetCode](https://leetcode.com/problems/remove-nth-node-from-end-of-list)
> Given a linked list, remove the nth node from the end of list and return its head.

此题刚开始看到觉得好奇怪啊：扫描一遍无论如何也无法实现同时获得链表数目以及确定删除点。然后参考了讨论版的解答。

一趟搜索的实现是：初始化两个指针 `fast` 和 `slow`，其中 `fast - slow = n`。这样，经过若干次「指针后移」操作后，当 `fast == null` 时， `slow` 指针指向的就是待删除的节点。

由于题中说明 `n` 总是有效的，所以无需做额外的有效性检查。