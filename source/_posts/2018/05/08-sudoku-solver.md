---
title: "[LC37/39/40/46-47] 用回溯算法解决问题"
date: 2018-05-08
tags: [leetcode, 回溯, 递归]
categories:
  - CS
  - LeetCode
---

LeetCode 中一系列问题使用回溯方法解决。本篇介绍一个通用的回溯方法框架。

<!-- more -->

## 回溯算法

### 概念

回溯算法系统地遍历搜索空间的所有可能。

以下以一种通用的方法说明。用向量 $a = (a_1, a_2, \ldots, a_n)$ 来表示一个 solution，其中的每个元素 $a_i$ 都来自一个有限有序集合 $S$。例如，该向量可以用来表示一个子集 $S$，如果 $S$ 中包括全集的第 $i$ 个元素那么 $a_i$ 为 `true`。该向量也可表示一组排列中的某一种安排。

在回溯算法的每一个迭代步骤，我们从一个「部分解决的」solution 开始（用 $a = (a_{1}, a_{2},..., a_{k})$ 表示）。然后，通过在末尾添加一个元素来 **扩展** 这个solution。扩展后，还需检查是否已经得到了一个完整 solution —— 如果已经得到，可以执行打印、计数等操作；否则，检查该扩展后的 部分解决的 solution 仍满足可扩展性到完整 solution 的约束。如果满足那么执行递归操作并继续迭代；否则，需要删除刚添加的元素并尝试其他可能直到穷尽。


### 基本框架

上述对于算法的描述可以用代码表示如下。其中 `finished` 用来标志「提前结束」的情况。

```c++
bool finished = FALSE;               /* found all solutions yet? */

backtrack(int a[], int k, data input) {
    int c[MAXCANDIDATES];        /* candidates for next position */
    int ncandidates;             /* next position candidate count */
    int i;                       /* counter */

    if (is_a_solution(a, k, input))
        process_solution(a, k, input);
    else {
        k = k + 1;
        construct_candidates(a, k, input, c, &ncandidates);
        for (i = 0; i < ncandidates; i++) {
            a[k] = c[i];
            make_move(a, k, input);
            backtrack(a, k, input);
            unmake_move(a, k, input);
            if (finished) return;   /* terminate early */
        }
    }
}
```

* `is_a_solution(a, k, input)` 判断当前的部分解向量 `a[1..k]` 是否是一个符合条件的解。
* `construct_candidates(a, k, input, c, ncandidates)` 根据目前状态，构造这一步可能的选择，存入 `c[]` 数组，其长度存入 `ncandidates`。
* `process_solution(a, k, input)` 对于符合条件的解进行处理，通常是输出、计数等。
* `make_move(a, k, input)` and `unmake_move(a, k, input)` 前者将所采用的选择更新到原数据结构中，后者把这一选择从原数据结构中删除。


## 37 Sudoku Solver

[Sudoku Solver - LeetCode](https://leetcode.com/problems/sudoku-solver/description/)

按照以上基本框架，替换相应的内容，下面给出的代码还有部分优化操作。在最初尝试解答此题的时候出现了错误，原因是没有在回溯方法中（调用方法本身一步后）清理使用空间。

比照我原先效率低下的代码，这份简洁代码中：
* 没有使用 `visited[][]` 来标记已访问。因为在数独数组中，只要元素是 `.` 就是未访问过，而且要注意访问完发现不满足时要及时 **撤销** 操作。`visited[][]` 记号源自回溯解决迷宫问题。在迷宫问题中，因为迷宫数组不可改变，需要另外设置标记符号。
* 直接在 `solve()` 中遍历 `board[][]`，以替代**计算下一个candidate**的操作。
* 用于检查数组合法性的 `isValid()` 很高效。我最初是采用标志位逐行逐列依次比较的方法（36题思路），而此题中只需一次循环比较特定位置是否满足就可以了。

```java
public class Solution {
    public void solveSudoku(char[][] board) {
        if(board == null || board.length == 0)
            return;
        solve(board);
    }
    
    public boolean solve(char[][] board){
        for(int i = 0; i < board.length; i++){
            for(int j = 0; j < board[0].length; j++){
                if(board[i][j] == '.'){
                    for(char c = '1'; c <= '9'; c++){ //trial. Try 1 through 9
                        if(isValid(board, i, j, c)){
                            board[i][j] = c; //Put c for this cell
                            if(solve(board))
                                return true; //If it's the solution return true
                            else
                                board[i][j] = '.'; //Otherwise go back
                        }
                    }
                    return false;
                }
            }
        }
        return true;
    }
    
    private boolean isValid(char[][] board, int row, int col, char c){
        for(int i = 0; i < 9; i++) {
            if(board[i][col] != '.' && board[i][col] == c) return false; //check row
            if(board[row][i] != '.' && board[row][i] == c) return false; //check column
            if(board[3 * (row / 3) + i / 3][ 3 * (col / 3) + i % 3] != '.' && 
board[3 * (row / 3) + i / 3][3 * (col / 3) + i % 3] == c) return false; //check 3*3 block
        }
        return true;
    }
}
```

## 39 Combination Sum

### 题目描述

[Combination Sum - LeetCode](https://leetcode.com/problems/combination-sum/)

> Given a **set** of candidate numbers (`candidates`) (without duplicates) and a target number (`target`), find all unique combinations in `candidates` where the candidate numbers sums to `target`.
The **same** repeated number may be chosen from `candidates` unlimited number of times.
**Note**:
All numbers (including `target`) will be positive integers.
The solution set must not contain duplicate combinations.
**Example 1**:
```
Input: candidates = [2,3,6,7], target = 7,
A solution set is:
[
  [7],
  [2,2,3]
]
```
>**Example 2**:
```
Input: candidates = [2,3,5], target = 8,
A solution set is:
[
  [2,2,2,2],
  [2,3,3],
  [3,5]
]
```

### 使用框架时进行优化

如果不加修改地使用原来的框架，也能够解决问题，但效率太低。存在这样几个问题：
* 如果不传入多余的参数，需要每次反复进行数组求和计算。巧妙的办法是：每次递归调用时，用 `target - candidates[i]` 作为参数，而不是不加处理地传入 `target`，可以省去很多计算操作。
* 如果每次选取（本轮计算的） `candidates` 时都默认从所有原始的 `candidates` 中选取，则会出现 `[2, 2, 3]` 和 `[2, 3, 2]` 等其他同一组数的排列结果都会加入结果集合中。一个提升效率的方法是，**比 `candidate[i]` 小的数字**不再作为新一轮递归调用的 `candidates`。注意，这不会导致漏选。
* 原先使用 HashSet 来消除 Solution List 中重复的数组，如果按上述第二条实现则无需考虑这一情况。

以下的实现来自 [LeetCode 的讨论版块](https://leetcode.com/problems/combination-sum/discuss/16502)。

```java
public List<List<Integer>> combinationSum(int[] nums, int target) {
    List<List<Integer>> list = new ArrayList<>();
    Arrays.sort(nums);
    backtrack(list, new ArrayList<>(), nums, target, 0);
    return list;
}

private void backtrack(List<List<Integer>> list, List<Integer> tempList, int [] nums, int remain, int start){
    if (remain < 0) return;
    else if (remain == 0) list.add(new ArrayList<>(tempList));
    else { 
        for (int i = start; i < nums.length; i++){
            tempList.add(nums[i]);
            backtrack(list, tempList, nums, remain - nums[i], i); // not i + 1 because we can reuse same elements
            tempList.remove(tempList.size() - 1);
        }
    }
}
```

## 40 Combination Sum II

### 题目描述

> Given a collection of candidate numbers (`candidates`) and a target number (`target`), find all unique combinations in `candidates` where the candidate numbers sums to `target`.
Each number in `candidates` may only be used **once** in the combination.
**Note**:
All numbers (including `target`) will be positive integers.
The solution set must not contain duplicate combinations.
**Example 1**:
```
Input: candidates = [10,1,2,7,6,1,5], target = 8,
A solution set is:
[
  [1, 7],
  [1, 2, 5],
  [2, 6],
  [1, 1, 6]
]
```
>**Example 2**:
```
Input: candidates = [2,5,2,1,2], target = 5,
A solution set is:
[
  [1,2,2],
  [5]
]
```

### 代码实现

与 39 Combination Sum 类似，不再具体分析。不过有两个值得一提的注意点：
* 为了避免由于 `candidates` 中某一个数重复从而重复输出同一结果，在某些情况下不执行回溯操作。注意，这不会导致 `[1, 1, 6]` 的漏解。
* 注意 `backtrack()` 调用时参数的设置：`startIndex = i + 1`。

```java
public List<List<Integer>> combinationSum2(int[] nums, int target) {
    List<List<Integer>> list = new ArrayList<>();
    Arrays.sort(nums);
    backtrack(list, new ArrayList<>(), nums, target, 0);
    return list;
    
}

private void backtrack(List<List<Integer>> list, List<Integer> tempList, int [] nums, int remain, int start){
    if(remain < 0) return;
    else if(remain == 0) list.add(new ArrayList<>(tempList));
    else{
        for(int i = start; i < nums.length; i++){
            if(i > start && nums[i] == nums[i-1]) continue; // skip duplicates
            tempList.add(nums[i]);
            backtrack(list, tempList, nums, remain - nums[i], i + 1);
            tempList.remove(tempList.size() - 1); 
        }
    }
} 
```

## 46 Permutations

```java
public List<List<Integer>> permute(int[] nums) {
   List<List<Integer>> list = new ArrayList<>();
   // Arrays.sort(nums); // not necessary
   backtrack(list, new ArrayList<>(), nums);
   return list;
}

private void backtrack(List<List<Integer>> list, List<Integer> tempList, int [] nums){
   if(tempList.size() == nums.length){
      list.add(new ArrayList<>(tempList));
   } else{
      for(int i = 0; i < nums.length; i++){ 
         if(tempList.contains(nums[i])) continue; // element already exists, skip
         tempList.add(nums[i]);
         backtrack(list, tempList, nums);
         tempList.remove(tempList.size() - 1);
      }
   }
} 
```

## 46 Permutations with duplicates

输出含有重复元素的全排列，不加处理地套用上一题的代码会得到：

```
[1, 1, 2]
[1, 2, 1]
[1, 1, 2]  x
[1, 2, 1]  x
[2, 1, 1]
[2, 1, 1]  x
```

需要对生成全排列的树形结构进行剪枝。具体来说，该树结构第一层节点是 `1 / 1 / 2`，需要剪去中间的一个 `1`；此外还需要剪去第一层节点 `2` 的孩子节点中的第二个节点。可以发现需要剪枝的节点是：**位于 `nums` 数组中非首次出现的节点**，还需注意这些节点需要满足：**与它们相同的前序节点还未排列**。这就可以通过修改上述代码来完成：

```java
public List<List<Integer>> permuteUnique(int[] nums) {
    List<List<Integer>> list = new ArrayList<>();
    Arrays.sort(nums);
    backtrack(list, new ArrayList<>(), nums, new boolean[nums.length]);
    return list;
}

private void backtrack(List<List<Integer>> list, List<Integer> tempList, int [] nums, boolean [] used){
    if(tempList.size() == nums.length){
        list.add(new ArrayList<>(tempList));
    } else{
        for(int i = 0; i < nums.length; i++){
            if(used[i] || i > 0 && nums[i] == nums[i-1] && !used[i - 1]) continue;
            used[i] = true; 
            tempList.add(nums[i]);
            backtrack(list, tempList, nums, used);
            used[i] = false; 
            tempList.remove(tempList.size() - 1);
        }
    }
}
```


## 附：回溯的经典应用

除了下面列出的几个经典应用，[这个博客](http://www.cnblogs.com/wuyuegb2312/p/3273337.html)还列举了几个其他应用：输出不重复数字的全排列、求解数独(剪枝的示范)、给定字符串生成其字母的全排列、求一个n元集合的k元子集、电话号码生成字符串、一摞烙饼的排序等。

### 列出所有子集

通过迭代地计算出 $2^n$ 个长度为 $n$ 的集合，构造出 $n$ 个元素的 $2^n$ 个子集：其中每一个元素 $a_i \in \{true, false\}$ 表示该元素是否出现在集合中。那么，$S_k = (true, false)，当 $k == n$ 时得到一个完整 solution。

```c++
is_a_solution(int a[], int k, int n){
        return (k == n);                /* is k == n? */
}

construct_candidates(int a[], int k, int n, int c[], int *ncandidates){
        c[0] = TRUE;
        c[1] = FALSE;
        *ncandidates = 2;
}

process_solution(int a[], int k){
        int i;                          /* counter */
        printf("{");
        for (i=1; i<=k; i++)
                if (a[i] == TRUE) printf(" %d",i);
        printf(" }\n");
}
Finally, we must instantiate the call to backtrack with the right arguments.


generate_subsets(int n){
        int a[NMAX];                    /* solution vector */
        backtrack(a,0,n);
}
```

### 列出所有排列

当计算第 $i$ 个元素的可能情况时，必须考虑该元素不能与前 $i - 1$ 个元素重复，因此在 `construct_candidates` 时需要排除这些情况。该案例中，$S_k = \{1, \ldots, n\} - a$，当 $k == n$ 时得到一个完整 solution。

```c++
construct_candidates(int a[], int k, int n, int c[], int *ncandidates){
        int i;                          /* counter */
        bool in_perm[NMAX];             /* who is in the permutation? */

        for (i=1; i<NMAX; i++) in_perm[i] = FALSE;
        for (i=0; i<k; i++) in_perm[ a[i] ] = TRUE;

        *ncandidates = 0;
        for (i=1; i<=n; i++) 
                if (in_perm[i] == FALSE) {
                        c[ *ncandidates] = i;
                        *ncandidates = *ncandidates + 1;
                }
}
```

其他函数的构造与上一案例类似。

### 「八皇后」问题

```c++
construct_candidates(int a[], int k, int n, int c[], int *ncandidates){
      int i,j;                        /* counters */
      bool legal_move;                /* might the move be legal? */

      *ncandidates = 0;
      for (i=1; i<=n; i++) {
          legal_move = TRUE;
          for (j=1; j<k; j++) {
                  if (abs((k)-j) == abs(i-a[j]))  /* diagonal threat */
                          legal_move = FALSE;
                  if (i == a[j])                  /* column threat */
                          legal_move = FALSE;
          }
          if (legal_move == TRUE) {
                  c[*ncandidates] = i;
                  *ncandidates = *ncandidates + 1;
          }
      }
}

process_solution(int a[], int k){
        int i;                          /* counter */
        solution_count ++;
}

is_a_solution(int a[], int k, int n){
        return (k == n);
}

nqueens(int n){
        int a[NMAX];                    /* solution vector */
        solution_count = 0;
        backtrack(a,0,n);
        printf("n=%d  solution_count=%d\n",n,solution_count);
}
```

