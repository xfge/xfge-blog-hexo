---
title: "[LC45/55] Jump Game / 动态规划 / 贪心 / BFS"
date: 2018-06-13
tags: [leetcode, 动态规划, 贪心]
categories:
  - CS
  - LeetCode
---

给定一个数组，每个元素表示该位置可以跳跃的最远长度。从数组的第一个元素开始起跳，确定跳到最后一个元素所需的最小步数/确定能否跳到最后一个元素。

<!-- more -->

## 确定跳到最后一个元素的最小步数

[Jump Game II - LeetCode](https://leetcode.com/problems/jump-game-ii/description/)

> Given an array of non-negative integers, you are initially positioned at the first index of the array.
Each element in the array represents your maximum jump length at that position.
Your goal is to reach the last index in the minimum number of jumps.
```
Input: [2,3,1,1,4]
Output: 2
Explanation: The minimum number of jumps to reach the last index is 2.
    Jump 1 step from index 0 to 1, then 3 steps to the last index.
```
**Note:** You can assume that you can always reach the last index.

### 应用 BFS 算法

### 贪心算法



## 确定能否跳到最后一个元素
[Jump Game - LeetCode](https://leetcode.com/problems/jump-game/description/)
> ......
> Determine if you are able to reach the last index.
```
# Example 1
Input: [2,3,1,1,4]
Output: true
Explanation: Jump 1 step from index 0 to 1, then 3 steps to the last index.

# Example 2
Input: [3,2,1,0,4]
Output: false
Explanation: You will always arrive at index 3 no matter what. Its maximum
             jump length is 0, which makes it impossible to reach the last index.
```

### 动态规划的思路

以下我们称数组中的一个位置为 “good index”，如果从该位置能够最终到达最后一个元素；否则，称之为 “bad index”。问题转化为：位置 0 是否为 “good index”。

下面介绍用动态规划算法求解这个问题。通常，应用动态规划需要考虑以下步骤：
1. 尝试以回溯的方法解决问题。
2. 通过一个备忘表格来优化回溯算法（自顶向下法）。
3. 去掉不必要的递归部分。
4. 使用一些技巧来降低时间/空间复杂度。


[关于 LC32 的讨论](https://xfge.github.io/2017/10/29/longest-valid-parentheses/) 中已经介绍了**带备忘的自顶向下法** 和 **自底向上法**。本文再以本题为例重新应用这两种方法。

#### 从回溯开始

回溯算法的思路是：从第一个位置开始，对于每一个它能够到达的位置，递归地判断从每个位置是否能够到达最后一个位置。回溯终止的条件是该位置已经是最后一个位置。当无法再进行跳跃（回溯算法中一次循环结束了都还没有到达最后位置）时，算法回溯。

```java
public class Solution {
    public boolean canJumpFromPosition(int position, int[] nums) {
        if (position == nums.length - 1) {
            return true;
        }

        int furthestJump = Math.min(position + nums[position], nums.length - 1);
        for (int nextPosition = position + 1; nextPosition <= furthestJump; nextPosition++) {
            if (canJumpFromPosition(nextPosition, nums)) {
                return true;
            }
        }

        return false;
    }

    public boolean canJump(int[] nums) {
        return canJumpFromPosition(0, nums);
    }
}
```

也可以对这段代码进行优化——从右往左地检查 `nextPosition`。然而，即使这样调整，最坏情况所需进行的计算依然不变（以下表格给出了一个最坏情况）。直觉上来看，这样优化后的算法将进行尽可能更远的跳跃以更快到达终点。需要改变的代码是：

```java
// Old
for (int nextPosition = position + 1; nextPosition <= furthestJump; nextPosition++)
// New
for (int nextPosition = furthestJump; nextPosition > position; nextPosition--)
```

| Index | 0     | 1     | 2     | 3     | 4     | 5     | 6     |
| :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: |
| nums  | 5     | 4     | 3     | 2     | 1     | 0     | 0     |

以上回溯算法的时间复杂度是 $O(2^n)$。从第一个位置跳到最后一个位置共有 $2^n$ 种可能。空间复杂度是 $O(n)$，但回溯算法本身需要额外的栈空间。 


#### 自顶向下的动态规划

带备忘的自顶向下动态规划方法可以看做是回溯算法的一个优化。其思想是一旦我们确定了一个位置是 “good/bad index”，只要保存这些信息，就不再需要每次重新计算。

用数组 `memo` 作为存放每个位置是否为 “good/bad index” 的备忘，有三种类型：GOOD, BAD, UNKNOWN。以 `nums = [2, 4, 2, 1, 0, 2, 0]` 为例，它的备忘数组如下所示：

| Index | 0     | 1     | 2     | 3     | 4     | 5     | 6     |
| :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: |
| nums  | 2     | 4     | 2     | 1     | 0     | 2     | 0     |
| memo  | G     | G     | B     | B     | B     | G     | G     |

步骤是：
1. 刚开始时，除了最后一个元素为 GOOD，其他位置都为 UNKNOWN。
2. 这样修改回溯算法：每次迭代时首先检查该位置是否已知为 GOOD/BAD。
    1. 如果为 GOOD/BAD 那么返回 TRUE/FALSE。
    2. 否则仍进行递归回溯。
3. 一旦确定了当前位置的属性，将其记录在 `memo` 数组中。

```java
enum Index {
    GOOD, BAD, UNKNOWN
}

public class Solution {
    Index[] memo;

    public boolean canJumpFromPosition(int position, int[] nums) {
        if (memo[position] != Index.UNKNOWN) {
            return memo[position] == Index.GOOD ? true : false;
        }

        int furthestJump = Math.min(position + nums[position], nums.length - 1);
        for (int nextPosition = position + 1; nextPosition <= furthestJump; nextPosition++) {
            if (canJumpFromPosition(nextPosition, nums)) {
                memo[position] = Index.GOOD;
                return true;
            }
        }

        memo[position] = Index.BAD;
        return false;
    }

    public boolean canJump(int[] nums) {
        memo = new Index[nums.length];
        for (int i = 0; i < memo.length; i++) {
            memo[i] = Index.UNKNOWN;
        }
        memo[memo.length - 1] = Index.GOOD;
        return canJumpFromPosition(0, nums);
    }
}
```

自顶向下的动态规划算法的时间复杂度是 $O(n^2)$。对于每个位置，算法检查其右侧 `nums[i]` 个元素直到找到某个 GOOD 位置，`nums[i]` 至多为 $n$。空间复杂度为 $O(2n) = O(n)$。


#### 自底向上的动态规划

通过将自顶向下改为自底向上地检查，可以不用递归完成计算。直观地说，就是当从右向左检查时，每一个位置的右侧所有位置都已经经过检查且记录在 `memo` 中。时间复杂度仍为 $O(n^2)$。

```java
enum Index {
    GOOD, BAD, UNKNOWN
}

public class Solution {
    public boolean canJump(int[] nums) {
        Index[] memo = new Index[nums.length];
        for (int i = 0; i < memo.length; i++) {
            memo[i] = Index.UNKNOWN;
        }
        memo[memo.length - 1] = Index.GOOD;

        for (int i = nums.length - 2; i >= 0; i--) {
            int furthestJump = Math.min(i + nums[i], nums.length - 1);
            for (int j = i + 1; j <= furthestJump; j++) {
                if (memo[j] == Index.GOOD) {
                    memo[i] = Index.GOOD;
                    break;
                }
            }
        }

        return memo[0] == Index.GOOD;
    }
}
```

### 贪心算法

观察上述自底向上的动态规划算法，其实可以发现：在检查每个位置时，我们的目的是为了检查从该位置是否可以到达 GOOD 点。我们只需要用到最靠左侧的 GOOD index。也就是说，只要保存 “最左侧 GOOD index” 的索引，那么在每次检查时就不再需要挨个遍历寻找这个位置了。

从右至左遍历时，对于每个位置我们检查是否可能跳到一个 GOOD index，即最左侧 GOOD index 是否包含在最远跳跃的范围中（`currPosition + nums[currPosition] >= leftmostGoodIndex`）。如果存在这样的情况，那么当前位置也是 GOOD，且更新成为新的 “最左侧 GOOD index”。直到遍历到第一个位置，如果该位置也是 GOOD，那么返回 TRUE。

```java
public class Solution {
    public boolean canJump(int[] nums) {
        int lastPos = nums.length - 1;
        for (int i = nums.length - 1; i >= 0; i--) {
            if (i + nums[i] >= lastPos) {
                lastPos = i;
            }
        }
        return lastPos == 0;
    }
}
```

以 `nums = [9, 4, 2, 1, 0, 2, 0]` 为例说明。假定现在已经检查到最左侧位置。由于 `index == 1` 位置已经确定为 GOOD，我们就已经可以确定从最左侧为止可以跳到1位置。`nums[0]` 本身有多大已经无关紧要。

| Index | 0     | 1     | 2     | 3     | 4     | 5     | 6     |
| :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: |
| nums  | 9     | 4     | 2     | 1     | 0     | 2     | 0     |
| memo  | U     | G     | B     | B     | B     | G     | G     |


贪心算法的时间复杂度是 $O(n)$，空间复杂度是 $O(1)$。

### 一遍扫描确定最远距离

```python
def canJump(self, nums):
    m = 0
    for i, n in enumerate(nums):
        if i > m:
            return False
        m = max(m, i+n)
    return True
```