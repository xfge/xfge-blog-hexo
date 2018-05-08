---
title: "[LC37] Sudoku Solver / 回溯"
date: 2018-05-08
tags: [leetcode, 回溯, 递归]
categories:
  - CS
  - LeetCode
---

Sodoku Solver 使用回溯方法解决。本篇介绍一个通用的回溯方法框架。

<!-- more -->

[Sudoku Solver - LeetCode](https://leetcode.com/problems/sudoku-solver/description/)

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


## 本题解法

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


## 回溯的经典应用

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

