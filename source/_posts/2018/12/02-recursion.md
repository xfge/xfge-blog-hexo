---
title: "[LeetCode] Recursion"
date: 2018-12-02
tags: [leetcode, 递归]
categories:
  - CS
  - LeetCode
---

本篇总结了 LeetCode 中用递归方法解决的问题，大多数这样的题都对应一种（非递归）迭代方式。最典型的是中序遍历序列问题。

<!-- more -->

## 101 Symmetric Tree

[101. Symmetric Tree](https://leetcode.com/problems/symmetric-tree/)

> Given a binary tree, check whether it is a mirror of itself (ie, symmetric around its center).
> For example, this binary tree `[1,2,2,3,4,4,3]` is symmetric:
```
    1
   / \
  2   2
 / \ / \
3  4 4  3
```

这道题不是直觉上简单地写一个递归函数作用于根节点就可以了。其实更关键的是判断 `root` 节点的两个孩子节点 `left` 、 `right` 是否是「镜像」的。而是否镜像这一判断在递归时，需要对恰当的节点进行比较。比如：

* `left.left` 与 `right.right` 互为镜像
* `left.right` 与 `right.left` 互为镜像

实现 `isMirror` 函数进行判断，函数内部就不需再判断是否对称了。

### 递归方法

```java
public boolean isSymmetric(TreeNode root) {
    return isMirror(root, root);
}

public boolean isMirror(TreeNode t1, TreeNode t2) {
    if (t1 == null && t2 == null) return true;
    if (t1 == null || t2 == null) return false;
    return (t1.val == t2.val)
        && isMirror(t1.right, t2.left)
        && isMirror(t1.left, t2.right);
}
```

### 迭代方法

```java
public boolean isSymmetric(TreeNode root) {
    Queue<TreeNode> q = new LinkedList<>();
    q.add(root);
    q.add(root);
    while (!q.isEmpty()) {
        TreeNode t1 = q.poll();
        TreeNode t2 = q.poll();
        if (t1 == null && t2 == null) continue;
        if (t1 == null || t2 == null) return false;
        if (t1.val != t2.val) return false;
        q.add(t1.left);
        q.add(t2.right);
        q.add(t1.right);
        q.add(t2.left);
    }
    return true;
}
```

## 94 Binary Tree Inorder Traversal

[94. Binary Tree Inorder Traversal](https://leetcode.com/problems/binary-tree-inorder-traversal/)

> Given a binary tree, return the inorder traversal of its nodes' values.
> **Example:**
```
Input: [1,null,2,3]
   1
    \
     2
    /
   3

Output: [1,3,2]
```

### 递归方法

递归方法很直观。大多数递归方法、回溯方法都需要另一个 `helper` 函数执行自身递归的实现，主函数调用它。

```java
public List<Integer> inorderTraversal(TreeNode root) {
    List<Integer> res = new ArrayList<>();
    helper(root, res);
    return res;
}

public void helper(TreeNode root, List<Integer> res) {
    if (root != null) {
        if (root.left != null) {
            helper(root.left, res);
        }
        res.add(root.val);
        if (root.right != null) {
            helper(root.right, res);
        }
    }
}
```

### 使用 Stack 的迭代方法

非递归方法需要使用栈来存储沿途还未加入到中序遍历序列的节点，并用于记录从右分支返回时的下一个访问节点。

```java
public List<Integer> inorderTraversal(TreeNode root) {
    List<Integer> res = new ArrayList<>();
    Stack<TreeNode> stack = new Stack<>();
    TreeNode curr = root;
    while (curr != null || !stack.isEmpty()) {
        while (curr != null) {
            stack.push(curr);
            curr = curr.left;
        }
        curr = stack.pop();
        res.add(curr.val);
        curr = curr.right;
    }
    return res;
}
```

### Morris Traversal

在 Morris Traversal 中，使用一种新的数据结构（Threaded Binary Tree）来实现中序遍历，具体策略是：

```
Step 1: Initialize current as root
Step 2: While current is not NULL,
If current does not have left child
    a. Add current’s value
    b. Go to the right, i.e., current = current.right
Else
    a. In current's left subtree, make current the right child of the rightmost node
    b. Go to this left child, i.e., current = current.left
```

如果一颗二叉树原来形如

```
     X
   /   \
  Y     Z
 / \   / \
A   B C   D
```

那么将其线性化，可以得到：

```
 A
  \
   Y
  / \
(A)  B
      \
       X
      / \
    (Y)  Z
        / \
       C   D
```

Morris Traversal 不需要递归。递归与栈紧密相关。而在 Morris Traversal 中，不再需要**记录序列中前一个节点**（previous node），因为每个树节点的父节点都蕴含在遍历过程中——它们可以在通常的迭代式遍历过程中被访问到。

[Stack Overflow 上的一个回答](https://stackoverflow.com/questions/5502916/explain-morris-inorder-tree-traversal-without-using-stacks-or-recursion)给出了详细的解释，而 [LeetCode 的 Solution](https://leetcode.com/problems/binary-tree-inorder-traversal/solution/) 是一个略微简化版的算法实现，与前者的区别在于，后者**不能**保留原始树结构。

```java
public List<Integer> inorderTraversal(TreeNode root) {
    List<Integer> nodes = new ArrayList<>();
    TreeNode cur = root;
    while (cur != null) {
        if (cur.left != null) {
            // Find the inorder predecessor of current
            TreeNode pre = cur.left;
            while ((pre.right != null) && (pre.right != cur)) {
                pre = pre.right;
            }
            // Make current as right child of its inorder predecessor
            if (pre.right == null) {
                pre.right = cur;
                cur = cur.left;
            }
            // MAGIC OF RESTORING the Tree happens here
            else { 
                pre.right = null;  // Cut the temp link
                nodes.add(cur.val);
                cur = cur.right;
            }
        } else {
            nodes.add(cur.val);
            cur = cur.right;
        }
    }
    return nodes;
}
```

下面的问题 [114 Flatten Binary Tree to Linked List](http://localhost:4000/2018/12/02/recursion/#114-Flatten-Binary-Tree-to-Linked-List) 也可用 Morris Traversal 算法解决。


## 98 Validate Binary Search Tree

[98. Validate Binary Search Tree](https://leetcode.com/problems/validate-binary-search-tree/)

> Given a binary tree, determine if it is a valid binary search tree (BST).
> Assume a BST is defined as follows:
* The left subtree of a node contains only nodes with keys less than the node's key.
* The right subtree of a node contains only nodes with keys greater than the node's key.
* Both the left and right subtrees must also be binary search trees.


### 递归方法

本题的递归解决方法是通过传递两个参数 `minVAL` 和 `maxVal` 用于每个节点的判断。判断时，节点必须介于 `minVal` 和 `maxVal` 之间才能继续递归。

```java
public class Solution {
    public boolean isValidBST(TreeNode root) {
        return isValidBST(root, Long.MIN_VALUE, Long.MAX_VALUE);
    }
    
    public boolean isValidBST(TreeNode root, long minVal, long maxVal) {
        if (root == null) return true;
        if (root.val >= maxVal || root.val <= minVal) return false;
        return isValidBST(root.left, minVal, root.val) && isValidBST(root.right, root.val, maxVal);
    }
}
```

### 迭代方法

非递归方式的解决方法用到了二叉树中序遍历的特性。考虑在本题的情境下，BST 的中序遍历一定是升序数组。修改上述使用 Stack 的迭代方法获取中序遍历的代码：

```java
public boolean isValidBST(TreeNode root) {
   if (root == null) return true;
   Stack<TreeNode> stack = new Stack<>();
   TreeNode pre = null;
   while (root != null || !stack.isEmpty()) {
      while (root != null) {
         stack.push(root);
         root = root.left;
      }
      root = stack.pop();
      if(pre != null && root.val <= pre.val) return false;
      pre = root;
      root = root.right;
   }
   return true;
}
```

## 114 Flatten Binary Tree to Linked List

[114. Flatten Binary Tree to Linked List](https://leetcode.com/problems/flatten-binary-tree-to-linked-list/)

> Given a binary tree, flatten it to a linked list in-place.
> For example, given the following tree:
```
    1
   / \
  2   5
 / \   \
3   4   6
```
> The flattened tree should look like:
```
1
 \
  2
   \
    3
     \
      4
       \
        5
         \
          6
```

### 递归方法

```java
public void flatten(TreeNode root) {
    if (root == null) return;
    
    TreeNode left = root.left;
    TreeNode right = root.right;
    
    root.left = null;
    
    flatten(left);
    flatten(right);
    
    root.right = left;
    TreeNode cur = root;
    while (cur.right != null) {
        cur = cur.right;
    }
    cur.right = right;
}
```

### 基于 Morris Traversal 算法的非递归方法

以下是基于 Morris Traversal 算法的修改，请结合[原版本](#Morris-Traversal)对比查看。当 `pre.right != null` 时，调整 Morris Traversal 原本的顺序（中序遍历）为前序遍历，就像**在链表中调整顺序**一样。

最好将[原版本](#Morris-Traversal)的例子在纸上画一下并搞清楚 `prev`、`cur.left`、`cur.right` 的取值以及更改前后的变化。上述例子足以用于这两个不同的情景。

```java
public void flatten(TreeNode root) {
    TreeNode cur = root, pre = root;
    while (cur != null) {
        if (cur.left != null) {
            pre = cur.left;
            while (pre.right != null && pre.right != cur) {
                pre = pre.right;
            }
            if (pre.right == null) {
                pre.right = cur;
                cur = cur.left;
            } else {
                // Instead of cut the temp link in Morris traversal, link it to right subtree
                TreeNode right = cur.right;
                cur.right = cur.left;
                cur.left = null;
                pre.right = right;
                cur = right;
            }
        } else {
            cur = cur.right;
        }
    }
}
```

### 基于“逆”前序遍历算法

```java
private TreeNode prev = null;
public void flatten(TreeNode root) {
    if (root == null)
        return;
    flatten(root.right);
    flatten(root.left);
    root.right = prev;
    root.left = null;
    prev = root;
}
```

观察 flattened tree 的节点顺序，即“前序遍历”。上面的算法巧妙地利用了一个特性：一棵树 $T$ 的**前序遍历**是其镜像树 $T'$ 的**后序遍历**的逆序。很有意思的一个特性。

这样，在镜像树上按照后序遍历的顺序，将每个访问过的节点用一个全局变量保存下来用于下一个访问节点的赋值，就构造出 flattened tree。


## 337 House Robber III

[337. House Robber III](https://leetcode.com/problems/house-robber-iii/)

The thief has found himself a new place for his thievery again. There is only one entrance to this area, called the "root." Besides the root, each house has one and only one parent house. After a tour, the smart thief realized that "all houses in this place forms a binary tree". It will automatically contact the police if two directly-linked houses were broken into on the same night.

Determine the maximum amount of money the thief can rob tonight without alerting the police.

Example 1:
```
Input: [3,2,3,null,3,null,1]

     3
    / \
   2   3
    \   \ 
     3   1

Output: 7 
Explanation: Maximum amount of money the thief can rob = 3 + 3 + 1 = 7.
```

### 返回两个数值

与一些典型的二叉树问题类似，该问题直觉上可以用递归方法解决。但是因为要避免两个相邻的节点同时纳入计算，就让问题稍微复杂了一些——计算不同的节点并比较其和。[这则帖子](https://leetcode.com/problems/house-robber-iii/discuss/79330/Step-by-step-tackling-of-the-problem)对这一问题分析的较为详尽，其包括了最粗糙的递归算法、简化后的动态规划算法以及与下述类似的优化后的递归算法。

下面经过优化后的算法展示了一个重要的特性：其通过返回两个整数区分了参数树节点「被计算」和「不被计算」两种情况，而避免了[最粗糙算法](https://leetcode.com/problems/house-robber-iii/discuss/79330/Step-by-step-tackling-of-the-problem)中在递归后反复重复计算的弊端：解构后的算法在一次递归函数中只需递归调用自身两次，所需要的值已经返回，不再需要重复计算。

```java
/**
 * Definition for a binary tree node.
 * public class TreeNode {
 *     int val;
 *     TreeNode left;
 *     TreeNode right;
 *     TreeNode(int x) { val = x; }
 * }
 */
class Solution {
    public int rob(TreeNode root) {
        return Integer.max(maxRobValue(root)[0], maxRobValue(root)[1]);
    }

    public int[] maxRobValue(TreeNode root) {
        int[] result = new int[]{0, 0};
        if (root == null) {
            return result;
        }

        int[] leftRobValues = maxRobValue(root.left);
        int[] rightRobValues = maxRobValue(root.right);

        result[0] = root.val + leftRobValues[1] + rightRobValues[1];
        result[1] = Integer.max(leftRobValues[0], leftRobValues[1]) + Integer.max(rightRobValues[0], rightRobValues[1]);

        return result;
    }
}
```