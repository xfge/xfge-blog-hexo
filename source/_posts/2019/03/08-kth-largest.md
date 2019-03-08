---
title: "[LC215] Kth Largest Element in an Array"
date: 2019-03-08
tags: [leetcode, 分治]
categories:
  - CS
  - LeetCode
---

在数组中找到第 K 大元素时一个典型题。类似的还有找到前 K 大元素。这道题的多种解法涵盖了堆排序思想和快排思想。

<!-- more -->

# 问题描述

[215. Kth Largest Element in an Array](https://leetcode.com/problems/kth-largest-element-in-an-array/)

Find the kth largest element in an unsorted array. Note that it is the kth largest element in the sorted order, not the kth distinct element. You may assume k is always valid, 1 ≤ k ≤ array's length.

# 解法

## 1 Brute Force

最容易想到的方法是使用一种排序算法，取排序结果的第 K 位。这种方法的时间复杂度是 $O(n\log{n})$，这种方法就没有使用好 “K” 的作用。

## 2 优先队列·堆排序算法

优先队列是一种高效的抽象数据类型，它支持的主要操作是「删除最大元素」和「插入元素」，最经典的实现是基于**二叉堆**（堆），能够实现时间复杂度为 $O(N\log{M})$ 的查询操作。如果只是需要找到前一组海量数据的前若干项元素，使用栈实现的优先队列很有帮助。避免了排序的巨大开销。优先队列也可以通过无序或有序的数组进行简陋的实现，时间复杂度略高（$O(NM)$）。

堆排序由两个阶段步骤（构造堆、下沉排序）组成，堆的实现通常是依据完全二叉树的特性（父子节点的索引特性）由数组直接实现，这使得它的效率非常高。在建堆步骤中，可以通过**对一半的元素（忽略只有一个元素的堆）**从右到左地执行下沉操作来简化通常的做法（从左到右上浮操作），实现效率的提升。在堆形成后的任何时候，都可直接获取堆中所有元素的最大（或最小）值。

利用该想法可以在 $O(N\log{K})$ 内找到第 K 大的元素。只需要将优先队列的大小控制在 **k + 1** 内（允许某个时刻为 k + 1 个元素，但随即栈顶元素被删除），这样，前 K 大的元素就都会「沉淀」在最小堆的内部，新进入的比它们小的元素都会被依次上浮到堆顶（此时有 k + 1 个元素）并随即被删除。

```java
public int findKthLargest(int[] nums, int k) {

    final PriorityQueue<Integer> pq = new PriorityQueue<>();
    for(int val : nums) {
        pq.offer(val);

        if(pq.size() > k) {
            pq.poll();
        }
    }
    return pq.peek();
}
```

## 3 只确定第 K 大元素的位置

回顾快速排序，在排序函数的内部，根据某个「支点」，分治地处理支点左边和右边的两个子数组。在支点设立之时，左数组的所有元素均小于支点元素，右数组均大于支点元素。支点的设置由 partition 函数进行，在该函数内部，先**随意**（或根据某些规则）选取支点，在以该支点对整个数组的所有元素进行遍历式访问，用两个指针（从左到右地移动或从两端相向移动）在访问的时候通过交换操作最终使得比支点元素小的元素和比它大的元素位于某个位置两侧。这个位置就是支点的位置。partition 最终返回支点索引。

使用与 partition 一样的思路，在每次确定新的支点位置的时候，如果该索引位置正好为 K（或 N-K，与升降序有关），那么该支点元素在数组中的位置其实已经确定，即，它就是第 K 大元素。

该算法的时间复杂度：在最好情况下为 $O(N)$，在最坏情况下为 $O(N^2)$。如果能加入对输入数据的随机化，即可有效避免最坏情况的出现，使时间复杂度保证为 $O(N)$。

```java
class Solution {
    public int findKthLargest(int[] nums, int k) {
        int len = nums.length;
        int i = 0, j = len - 1;
        while (true) {
            int p = partition(nums, i, j);
            if (p == len - k) {
                return nums[p];
            }
            else if (p < len - k) {
                i = p + 1;
            }
            else {
                j = p - 1;
            }
        }
    }

    public int partition(int[] a, int l, int r) {
        int pivot = a[r];
        int i = l - 1;
        for (int j = l; j <= r - 1; j++) {
            if (a[j] < pivot) {
                swap(a, ++i, j);
            }
        }
        swap(a, ++i, r);
        return i;
    }
    
    public void swap(int[] a, int i, int j) {
        int tmp = a[i];
        a[i] = a[j];
        a[j] = tmp;
    }
}
```

《算法》版本的 partition （两指针相向移动）如下。

```java
public int findKthLargest(int[] nums, int k) {
    k = nums.length - k;
    int lo = 0;
    int hi = nums.length - 1;
    while (lo < hi) {
        final int j = partition(nums, lo, hi);
        if(j < k) {
            lo = j + 1;
        } else if (j > k) {
            hi = j - 1;
        } else {
            break;
        }
    }
    return nums[k];
}

private int partition(int[] a, int lo, int hi) {

    int i = lo;
    int j = hi + 1;
    while(true) {
        while(i < hi && less(a[++i], a[lo]));
        while(j > lo && less(a[lo], a[--j]));
        if(i >= j) {
            break;
        }
        exch(a, i, j);
    }
    exch(a, lo, j);
    return j;
}

private void exch(int[] a, int i, int j) {
    final int tmp = a[i];
    a[i] = a[j];
    a[j] = tmp;
}

private boolean less(int v, int w) {
    return v < w;
}
```