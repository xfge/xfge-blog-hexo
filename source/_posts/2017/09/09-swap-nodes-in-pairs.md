---
title: "[LC24] Swap Nodes in Pairs / 递归"
date: 2017-09-09
tags: [leetcode, 递归]
categories:
  - CS
  - LeetCode
---

[Swap Nodes in Pairs - LeetCode](https://leetcode.com/problems/swap-nodes-in-pairs/)

> Given a linked list, swap every two adjacent nodes and return its head.

<!-- more -->

>For example,
Given `1->2->3->4`, you should return the list as `2->1->4->3`.
Your algorithm should use only constant space. You may not modify the values in the list, only nodes itself can be changed.

## 很直观的解法

这题较为容易，且题目所要求的常数空间可以很容易地达到。只要通过循环依次两个两个地交换数字即可。

```java
public ListNode swapPairs(ListNode head) {
    ListNode dummy = new ListNode(0);
    dummy.next = head;
    ListNode current = dummy;
    while (current.next != null && current.next.next != null) {
        ListNode first = current.next;
        ListNode second = current.next.next;
        first.next = second.next;
        current.next = second;
        current.next.next = first;
        current = current.next.next;
    }
    return dummy.next;
}
```

## 递归

将此题单独列出是因为看到 Discussion 板块有人提供了一种使用递归解决问题的方法。将代码简洁到了极致，让我大开眼界。仔细考虑一下该题确实隐含着递归的思想，虽然未能满足常数空间，但非常值得参考。

```java
public class Solution {
    public ListNode swapPairs(ListNode head) {
        if ((head == null)||(head.next == null))
            return head;
        ListNode n = head.next;
        head.next = swapPairs(head.next.next);
        n.next = head;
        return n;
    }
}
```