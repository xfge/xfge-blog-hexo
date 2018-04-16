---
title: "[LC24/25] Swap Nodes in k-Group / 递归"
date: 2017-09-09
tags: [leetcode, 递归]
categories:
  - CS
  - LeetCode
---

24和25两题都是练习链表的相关操作，涉及到对链表节点的翻转。是显而易见的对经典方法——递归的使用。

<!-- more -->

## 24 Swap Nodes in Pairs

[Swap Nodes in Pairs - LeetCode](https://leetcode.com/problems/swap-nodes-in-pairs/)

> Given a linked list, swap every two adjacent nodes and return its head.
>For example,
Given `1->2->3->4`, you should return the list as `2->1->4->3`.
Your algorithm should use only constant space. You may not modify the values in the list, only nodes itself can be changed.

### 很直观的解法

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

### 递归

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

## 25 Reverse Nodes in k-Group

### 用了递归，但还是很复杂的代码。

在整理这篇笔记时，又重新写了一下这道题的代码（18/4/14），因为之前总结过递归的一些例题，所以想到用递归来实现。而且，翻阅7个月之前的提交记录，当时是通过 while 循环来实现的。虽然在此题中两者没有什么差距，但有点小高兴的是，使用递归方法的代码（虽然仍很冗长）当前已经超过99.91%的提交。😊

| Time Submitted        | Status    |  Runtime  |
| --------   | -----  | ---- |
| 0        | Accepted      |   7 ms    |
| 7 months ago        | Accepted      |   8 ms    |

下面是第二次提交的代码。去年首次提交的代码由于太过冗长就不放这里了。

```java
class Solution {
    public ListNode reverseKGroup(ListNode head, int k) {
        ListNode dummy = head;
        int cnt = 0;
        while (dummy != null) {
            if (cnt == k) {
                break;
            }
            cnt++;
            dummy = dummy.next;
        }
        if (cnt < k) {
            return head;
        }

        ListNode lastHead = head;
        ListNode last = head;
        ListNode current = head;
        ListNode next = head.next;

        cnt = k - 1;
        while (cnt != 0) {
            current = next;
            next = next.next;
            current.next = last;
            last = current;
            cnt--;
        }
        lastHead.next = reverseKGroup(next, k);
        return current;

    }
}
```

### 借鉴简洁代码

```java
public ListNode reverseKGroup(ListNode head, int k) {
    ListNode curr = head;
    int count = 0;
    while (curr != null && count != k) { // find the k+1 node
        curr = curr.next;
        count++;
    }
    if (count == k) { // if k+1 node is found
        curr = reverseKGroup(curr, k); // reverse list with k+1 node as head
        // head - head-pointer to direct part, 
        // curr - head-pointer to reversed part;
        while (count-- > 0) { // reverse current k-group: 
            ListNode tmp = head.next; // tmp - next head in direct part
            head.next = curr; // preappending "direct" head to the reversed list 
            curr = head; // move head of reversed part to a new node
            head = tmp; // move "direct" head to the next node in direct part
        }
        head = curr;
    }
    return head;
}
```

代码中两次循环分别对过程简化。第一个循环用于确定 k+1 个节点，第二个循环是用于将链表重组，且合为一个完整过程。尤其注意其从每一组待翻转的链表的末端的下一位（`curr`）开始，直接将其链接到原先 `head` 的 `next` 属性上。然后修改 `curr` 使其指向当前的 `head`，用计数器计数直到翻转完毕。

要关注代码简化思想。