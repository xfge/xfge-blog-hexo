---
title: "[LC60/62] 用优化的方法替代回溯算法"
date: 2018-07-27
tags: [leetcode]
categories:
  - CS
  - LeetCode
---

回溯（Back Tracking）算法相当于使用 BFS 将所有可能的情况遍历一遍，由于使用了递归栈，效率不高。在有些问题中，可以用更优化的方法解决而不需要递归地遍历所有情形。

<!-- more -->

## 60 Permutation Sequence
[Permutation Sequence - LeetCode](https://leetcode.com/problems/permutation-sequence/description/)

>The set `[1, 2, 3, ..., n]` contains a total of $n!$ unique permutations.
By listing and labeling all of the permutations in order, we get the following sequence for $n$ = 3:
`["123", "132", "213", "231", "312", "321"]`
Given $n$ and $k$, return the $k^{th}$ permutation sequence.

当解决另一个比较类似的问题，即「按序穷举字符组合的所有排列」时，使用回溯算法/BFS算法。此题亦可使用该方法解决。但注意到如果能归纳出序列中每个位置上数字排列所蕴含的「规律」，也就不必穷举出此前所有的序列了。那么，给定 $k$ 可以根据规律输出其排序。

下面以 $n = 4$ 为例来说明这一想法。

$n = 4$ 所列出的所有排序可以归纳为：

* 1 + (permutations of 2, 3, 4)
* 2 + (permutations of 1, 3, 4)
* 3 + (permutations of 1, 2, 4)
* 4 + (permutations of 1, 2, 3)

上述的每一「组」排列情况（的集合），有 $n! / 4 = 6$ 个元素。如果我们要找 $k = 14$ 所对应的排列，它应该在第 `(k - 1) / 6` 个集合中，即「3 + (permutations of 1, 2, 4)」中。**那么就确定了第一个数字为 3**。（注意，使用 `k - 1` 的原因是我们一直以 0 开始记位。）

对剩下的数字 (1, 2, 4) 作类似的分析。此时注意更新 $k$ 的值为 `(k - 1) - 2 * 6 = 1`，即减去了前面两个排列集合中总的排列个数。按照与上述类似的方法可以确定**第二个数字为 1**。

这样就剩下两个数字 (2, 4)，总共两种排列情况。根据当前 $k$ 的值确定第三个数字为 4；确定最后一个数字为剩下的 2。

这是一种比较直观的想法，关键在于如何编程实现对 `k` 值的反复更新和递归计算。

```java
public String getPermutation(int n, int k) {
    int pos = 0;
    List<Integer> numbers = new ArrayList<>();
    int[] factorial = new int[n+1];
    StringBuilder sb = new StringBuilder();
    
    // create an array of factorial lookup
    int sum = 1;
    factorial[0] = 1;
    for(int i=1; i<=n; i++){
        sum *= i;
        factorial[i] = sum;
    }
    // factorial[] = {1, 1, 2, 6, 24, ... n!}
    
    // create a list of numbers to get indices
    for(int i=1; i<=n; i++){
        numbers.add(i);
    }
    // numbers = {1, 2, 3, 4}
    
    k--;
    
    for(int i = 1; i <= n; i++){
        int index = k/factorial[n-i];
        sb.append(String.valueOf(numbers.get(index)));
        numbers.remove(index);
        k-=index*factorial[n-i];
    }
    
    return String.valueOf(sb);
}
```

## 62 Unique Paths
[Unique Paths - LeetCode](https://leetcode.com/problems/unique-paths/description/)

这道题用 BFS 的方法做就属于 Brute Force 了。其实，只要稍微一想就能发现简单的动态规划公式：`P[i][j] = P[i - 1][j] + P[i][j - 1]`。其中，用 `P[i][j]` 记录到达 (i, j) 点的路径个数，而到达每一个点的路径个数为「到达上方点的路径个数」加上「到达左边点的路径个数」。将首行和首列初始化为 1，然后从下一行/列开始计算。

```c++
    int uniquePaths(int m, int n) {
        vector<vector<int> > path(m, vector<int> (n, 1));
        for (int i = 1; i < m; i++)
            for (int j = 1; j < n; j++)
                path[i][j] = path[i - 1][j] + path[i][j - 1];
        return path[m - 1][n - 1];
    }
```

注意，该算法的时间复杂度是 $O(n^2)$，空间复杂度是 $O(mn)$。[LeetCode 的讨论帖](https://leetcode.com/problems/unique-paths/discuss/22954)还提供了一种优化空间复杂度的思路。如果在绘制 DP 表格时按照从上往下、从左往右的方向计算，可以只记录**当前列**和**左边一列**。这就将空间复杂度降为 $O(min(m, n))$。

```c++
    int uniquePaths(int m, int n) {
        if (m > n) return uniquePaths(n, m); 
        vector<int> pre(m, 1);
        vector<int> cur(m, 1);
        for (int j = 1; j < n; j++) {
            for (int i = 1; i < m; i++)
                cur[i] = cur[i - 1] + pre[i];
            swap(pre, cur);
        }
        return pre[m - 1];
    }
```

更进一步，上一个方法中使用两个数组来记录两个列的所有数字。但 `pre` 中真正需要的数字仅仅是 `pre[i]`，而这一数字就是 `cur[i]` 在更新之前的数字。完全可以用 `cur[i] += cur[i - 1]`。只需维护 `cur` 这一个数组就可以了。

```c++
    int uniquePaths(int m, int n) {
        if (m > n) return uniquePaths(n, m);
        vector<int> cur(m, 1);
        for (int j = 1; j < n; j++)
            for (int i = 1; i < m; i++)
                cur[i] += cur[i - 1]; 
        return cur[m - 1];
    }
```

但上面这一个算法已经无法再进一步简化为 $O(1)$ 了，因为至少必须记录一整列的数字用于后续一列的更新。