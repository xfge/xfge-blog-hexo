---
title: "[LC21/23] Merge k Sorted Lists"
date: 2017-09-07
tags: [leetcode]
categories:
  - CS
  - LeetCode
---

本篇梳理「归并多个有序列表」问题。求和问题是指一类从数组中找出 K 个数满足其和为给定整数的问题。

<!-- more -->


## 用递归方法解决 Merge 2 Sorted Lits

[Merge Two Sorted Lists - LeetCode](https://leetcode.com/problems/merge-two-sorted-lists)
> Merge two sorted linked lists and return it as a new list. The new list should be made by splicing together the nodes of the first two lists.

显然，可以通过两个指针分别指向链表头，依次比较并移动指针来实现归并。这是一种基本思路，实现起来容易但有点繁琐。

### 用递归解决

递归方法写出的代码就简洁多了。递归终止条件是 「`l1 == null` 或者 `l2 == null`」，其他情况只需将递归比较所返回的结果置为 `[l1|l2].next` 即可。

这种实现并不是[尾递归](https://zh.wikipedia.org/wiki/尾调用)，因此当链表规模过大的时候可能导致「栈溢出」。

```c++
class Solution {
public:
    ListNode *mergeTwoLists(ListNode *l1, ListNode *l2) {
        if(l1 == NULL) return l2;
        if(l2 == NULL) return l1;
        
        if(l1->val < l2->val) {
            l1->next = mergeTwoLists(l1->next, l2);
            return l1;
        } else {
            l2->next = mergeTwoLists(l2->next, l1);
            return l2;
        }
    }
};
```

## Merge k Sorted Lists
