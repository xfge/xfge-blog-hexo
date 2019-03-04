---
title: "[LeetCode] Traversal"
date: 2018-12-10
tags: [leetcode, DFS, BFS]
categories:
  - CS
  - LeetCode
---

深度优先遍历（DFS）和广度优先遍历（BFS）是在树和图算法中至关重要。两者虽同样作为遍历算法，也有不同的应用场景。DFS 与栈密切相关；BFS 与队列密切相关。

<!-- more -->

# 基本概念


## BFS

### BFS 与队列

除了遍历树或图，BFS 的一个典型应用是**找到最短路径**。以下是一个典型的 BFS 代码框架，其中加入了对已访问节点的记录以防陷入死循环。如果确保遍历的图是无环图，或者希望多次访问节点，那么可以略去这一步骤。

```java
/**
 * Return the length of the shortest path between root and target node.
 */
int BFS(Node root, Node target) {
    Queue<Node> queue;  // store all nodes which are waiting to be processed
    Set<Node> used;     // store all the used nodes
    int step = 0;       // number of steps neeeded from root to current node
    // initialize
    add root to queue;
    add root to used;
    // BFS
    while (queue is not empty) {
        step = step + 1;
        // iterate the nodes which are already in the queue
        int size = queue.size();
        for (int i = 0; i < size; ++i) {
            Node cur = the first node in queue;
            return step if cur is target;
            for (Node next : the neighbors of cur) {
                if (next is not in used) {
                    add next to queue;
                    add next to used;
                }
            }
            remove the first node from queue;
        }
    }
    return -1;          // there is no path from root to target
}
```


## DFS

### DFS 与栈

多数情况下，在使用 BFS 的地方也可以使用 DFS，但它们的遍历顺序不同。DFS 所遍历得到的某一节点到另一节点的路径可能**不是最短路径**。

```java
/*
 * Return true if there is a path from cur to target.
 */
boolean DFS(Node cur, Node target, Set<Node> visited) {
    return true if cur is target;
    for (next : each neighbor of cur) {
        if (next is not in visited) {
            add next to visted;
            return true if DFS(next, target, visited) == true;
        }
    }
    return false;
}
```

事实上，在调用 DFS 时，并没有显式地使用栈，而是使用了系统提供的所谓**调用栈**。

### 时间戳

<div align="center"><img src="{{ site.baseurl }}/images/2018/12/dfs-struct.png" width="60%"></div>

每个节点的时间戳（发现时间、搜索完成时间）提供了图结构的重要信息。其所具有的括号化结构能够帮助我们判定各边属性。

#### 括号化定理

* 区间 `[u.d, u.f]` 和区间 `[v.d, v.f]` 完全分离，在深度优先森林中，节点 `u` 不是节点 `v` 的后代，反之也不是。
* 区间 `[u.d, u.f]` 完全包含在区间 `[v.d, v.f]` 内，在深度优先树中，节点 `u` 是节点 `v` 的后代。
* 区间 `[v.d, v.f]` 完全包含在区间 `[u.d, u.f]` 内，在深度优先树中，节点 `v` 是节点 `u` 的后代。

<div align="center"><img src="{{ site.baseurl }}/images/2018/12/dfs-time.png" width="50%"></div>

### DFS Tree

#### 边的类型

1. **树边**：深度优先森林中的边。如果节点 `v` 是因算法对边 `(u, v)` 的探索而首先发现，则 `(u, v)` 是一条树边。
2. **后向边**：后向边 `(u, v)` 是将节点 `u` 连接到其在深度优先树中（一个）祖先节点 `v` 的边。自循环也被认为是后向边。
3. **前向边**：是将节点 `u` 连接到其在深度优先树中一个后代节点 `v` 的边 `(u, v)`。
4. **横向边**：其他所有的边。这些边可以连接同一棵深度优先树中的节点，只要其中一个节点不是另外一个节点的祖先，也可以连接不同深度优先树中的两个节点。

当遇到某些边时，DFS 有足够的信息对边分类。第一次探索边 `(u, v)` 时，节点 `v` 的颜色决定了该边的分类：

1. 节点 `v` 为白色表明该条边 `(u, v)` 是一条树边。
2. 节点 `v` 为灰色表明该条边 `(u, v)` 是一条后向边。
3. 节点 `v` 为黑色表明该条边 `(u, v)` 是一条前向边或衡向边。其中，`u.d < v.d` 时为前向边，`u.d > v.d` 时为衡向边。

在对无向图的 DFS 中，不会出现前向边和横向边。



# 相关问题

## 310 Minimum Height Trees

[310. Minimum Height Trees](https://leetcode.com/problems/minimum-height-trees/)

> For an undirected graph with tree characteristics, we can choose any node as the root. The result graph is then a rooted tree. Among all possible rooted trees, those with minimum height are called minimum height trees (MHTs). Given such a graph, write a function to find all the MHTs and return a list of their root labels.
> **Format：**
> The graph contains `n` nodes which are labeled from `0` to `n - 1`. You will be given the number n and a list of undirected `edges` (each edge is a pair of labels).
> You can assume that no duplicate edges will appear in `edges`. Since all edges are undirected, `[0, 1]` is the same as `[1, 0]` and thus will not appear together in `edges`.
> **Example：**
```
Input: n = 6, edges = [[0, 3], [1, 3], [2, 3], [4, 3], [5, 4]]

     0  1  2
      \ | /
        3
        |
        4
        |
        5 

Output: [3, 4]
```

又是一道 Brute Force 很容易想到、但有多种优化方式的题。肯定不能每个节点遍历一遍去计算，就要想到 Minimum Height Trees (MHT) 的根节点有什么特点。这是个看上去很容易理解的道理：一根绳子从哪个位置拎起来下垂的长度更短？显然，答案是中间。对应到这棵树上，就是**最长路径**的中间节点（可能有两个）。

### 如何找到最长路径？

可以通过两次遍历（DFS/BFS）确定最长路径。第一次遍历前，从随意选取的某一个端点（可能不是最长路径的一端）`x` 开始遍历，找到以它为端点的最长路径 `x..u`，`u` 就是整棵树最长路径的一个端点。可以用反证法说明，注意：树中两个端点间只有一条路径。这时以 `u` 为起点执行第二次遍历，找到以 `u` 为端点的最长路径 `u..v`，这也是整棵树的最长路径。

```java
private void bfs(int start, int n, List<List<Integer>> neighbors, int[] parents, int[] levels) {
    // Normal bfs recording levels for each node
}

public List<Integer> findMinHeightTrees(int n, int[][] edges) {
    if (n == 1) {
        return Arrays.asList(0);
    }

    List<List<Integer>> neighbors = new ArrayList<>();
    for (int i = 0; i < n; i++) {
        neighbors.add(new ArrayList<>());
    }
    for (int i = 0; i < edges.length; i++) {
        neighbors.get(edges[i][0]).add(edges[i][1]);
        neighbors.get(edges[i][1]).add(edges[i][0]);
    }

    int[] parents = new int[n];
    int[] levels = new int[n];

    bfs(0, n, neighbors, parents, levels);
    int u = 0;
    for (int i = 0; i < n; i++) {
        if (levels[i] > levels[u]) {
            u = i;
        }
    }

    bfs(u, n, neighbors, parents, levels);
    int v = 0;
    for (int i = 0; i < n; i++) {
        if (levels[i] > levels[v]) {
            v = i;
        }
    }
    List<Integer> longestPath = new ArrayList<>();
    while (v != -1) {
        longestPath.add(v);
        v = parents[v];
    }

    int size = longestPath.size();
    return (size % 2 == 0) ? Arrays.asList(longestPath.get(size / 2 - 1), longestPath.get(size / 2)) : Arrays.asList(longestPath.get(size / 2));
}
```

### 通过一次遍历确定中间点

对于最简单的情形（有 `n` 个节点的线图），只需指定两个指针分别从两个端点出发，以相同速度靠近，如果两指针重合或只差一步，就找到了中间点（可能是两个）。

将这种「双指针」的思想应用于树，从每个**叶子节点**开始（叶子节点是度为1的节点），每个迭代步删去所有当前的叶子节点（对应于在邻接矩阵上删去这一节点），直到仅剩1个或2个节点——他们都是 MHT 的根节点。其实，在这一过程中被删去的节点都*不可能*是 MHT 的根节点，因为**它的邻居作为 MHT 的根节点时总能使树高度更短**。

<div align="center"><img src="{{ site.baseurl }}/images/2018/12/mht.jpeg" width="50%"></div>
<center>「农村包围城市」</center>

```java
public List<Integer> findMinHeightTrees(int n, int[][] edges) {
    if (n == 1) return Collections.singletonList(0);

    List<Set<Integer>> adj = new ArrayList<>(n);
    for (int i = 0; i < n; ++i) adj.add(new HashSet<>());
    for (int[] edge : edges) {
        adj.get(edge[0]).add(edge[1]);
        adj.get(edge[1]).add(edge[0]);
    }

    List<Integer> leaves = new ArrayList<>();
    for (int i = 0; i < n; ++i)
        if (adj.get(i).size() == 1) leaves.add(i);

    while (n > 2) {
        n -= leaves.size();
        List<Integer> newLeaves = new ArrayList<>();
        for (int i : leaves) {
            int j = adj.get(i).iterator().next();
            adj.get(j).remove(i);
            if (adj.get(j).size() == 1) newLeaves.add(j);
        }
        leaves = newLeaves;
    }
    return leaves;
}
```


## 417 Pacific Atlantic Water Flow

[417. Pacific Atlantic Water Flow](https://leetcode.com/problems/pacific-atlantic-water-flow/)

> Given an `m x n` matrix of non-negative integers representing the height of each unit cell in a continent, the "Pacific ocean" touches the left and top edges of the matrix and the "Atlantic ocean" touches the right and bottom edges.
> Water can only flow in four directions (up, down, left, or right) from a cell to another one with height equal or lower.
> Find the list of grid coordinates where water can flow to both the Pacific and Atlantic ocean.

> **Note**:
1. The order of returned grid coordinates does not matter.
2. Both m and n are less than 150.

> **Example**:
```
Given the following 5x5 matrix:

  Pacific ~   ~   ~   ~   ~ 
       ~  1   2   2   3  (5) *
       ~  3   2   3  (4) (4) *
       ~  2   4  (5)  3   1  *
       ~ (6) (7)  1   4   5  *
       ~ (5)  1   1   2   4  *
          *   *   *   *   * Atlantic

Return:

[[0, 4], [1, 3], [1, 4], [2, 2], [3, 0], [3, 1], [4, 0]] (positions with parentheses in above matrix).
```

### 确定遍历的起始点和方向

直接想到的方法是从**每个点**出发 DFS，如果到达 Pacific 或 Atlantic 就记录下来，最后扫描一遍检查符合要求的点。但这种方法 TLE。想到可以从另一个方向「反向」地 DFS。事实上，DFS 的起点是自定义的，在无向图中没有区分。仔细确定 DFS 执行的起点，可以影响算法的效率。

```java
class Solution {
    int[][] moves = new int[][] {{-1, 0}, {0, -1}, {0, 1}, {1, 0}};
    int m, n;
    public List<int[]> pacificAtlantic(int[][] matrix) {
        m = matrix.length;
        n = matrix[0].length;
        if (m == 0) {
            return new ArrayList<>();
        }
        boolean[][] reachA = new boolean[m][n];
        boolean[][] reachB = new boolean[m][n];

        for (int i = 0; i < m; i++) {
            dfs(i, 0, matrix, reachA);
            dfs(i, n - 1, matrix, reachB);
        }
        for (int j = 0; j < n; j++) {
            dfs(0, j, matrix, reachA);
            dfs(m - 1, j, matrix, reachB);
        }
        List<int[]> ret = new ArrayList<>();
        for (int i = 0; i < m; i++) {
            for (int j = 0; j < n; j++) {
                if (reachA[i][j] && reachB[i][j]) {
                    ret.add(new int[] {i, j});
                }
            }
        }
        return ret;
    }
    private void dfs(int r, int c, int[][] matrix, boolean[][] canReach) {
        canReach[r][c] = true;
        for (int[] move : moves) {
            int rMove = r + move[0], cMove = c + move[1];
            if (rMove >= 0 && rMove < m && cMove >= 0 && cMove < n && !canReach[rMove][cMove] && matrix[r][c] <= matrix[rMove][cMove]) {
                dfs(rMove, cMove, matrix, canReach);
            }
        }
    }
}
```

上述算法也可以改成 BFS。BFS 只需在起始时分别将海岸线**一次性地**加入队列中，算法执行时在每一层就**扩大**覆盖的区域的**边界**。

确定遍历起始点的重要性在另外一些题中也有体现。以下两题都反映了这种思想。


## 130 Surrounded Regions

[130. Surrounded Regions](https://leetcode.com/problems/surrounded-regions/)

> Given a 2D board containing `'X'` and `'O'` (the letter O), capture all regions surrounded by `'X'`.
> A region is captured by flipping all `'O'`s into `'X'`s in that surrounded region.

> **Example:**
```
X X X X
X O O X
X X O X
X O X X
```
> After running your function, the board should be:
```
X X X X
X X X X
X X X X
X O X X
```

> **Explanation:**
> Surrounded regions shouldn’t be on the border, which means that any 'O' on the border of the board are not flipped to 'X'. Any 'O' that is not on the border and it is not connected to an 'O' on the border will be flipped to 'X'. Two cells are connected if they are adjacent cells connected horizontally or vertically.

对边界上的 `O` 点执行 DFS，访问到的 `O` 点都更改为 `1`。所有边界上的点都探索并执行 DFS 完毕后，将标记过的点改为 `O` 其他点不变或改为 `X`。参考代码在[这里](https://leetcode.com/problems/surrounded-regions/discuss/41612)。

```
         X X X X           X X X X             X X X X
         X X O X  ->       X X O X    ->       X X X X
         X O X X           X 1 X X             X O X X
         X O X X           X 1 X X             X O X X
```


## 542 01 Matrix

[542. 01 Matrix](https://leetcode.com/problems/01-matrix/)

> Given a matrix consists of 0 and 1, find the distance of the nearest 0 for each cell.]
> The distance between two adjacent cells is 1.
> **Example 2**: 
> **Input**:
    ```
    0 0 0
    0 1 0
    1 1 1
    ```
> **Output**:
    ```
    0 0 0
    0 1 0
    1 2 1
    ```
> **Note**:
1. The number of elements of the given matrix will not exceed 10,000.
2. There are at least one 0 in the given matrix.
3. The cells are adjacent in only four directions: up, down, left and right.

### 用 BFS 计算最短距离

使用 BFS 解决这种需要计算**最短距离**的问题。总体思路是：
1. 在执行 BFS 之前，遍历矩阵将值为 0 的元素入列，其他元素值修改为 `Integer.MAX_VALUE`。
2. 在访问每个节点时，对四个相邻的点比较它们自身的值 `val` 和 **`cal_dist` = 当前点距离 + 1** 的大小，如果 `val > cal_dist` 就修改 `val` 为 `cal_dist`，并入队待下一轮访问。

```java
public int[][] updateMatrix(int[][] matrix) {
    int m = matrix.length;
    int n = matrix[0].length;
    
    Queue<int[]> queue = new LinkedList<>();
    for (int i = 0; i < m; i++) {
        for (int j = 0; j < n; j++) {
            if (matrix[i][j] == 0) {
                queue.offer(new int[] {i, j});
            }
            else {
                matrix[i][j] = Integer.MAX_VALUE;
            }
        }
    }
    
    int[][] dirs = {{-1, 0}, {1, 0}, {0, -1}, {0, 1}};
    while (!queue.isEmpty()) {
        int[] cell = queue.poll();
        for (int[] d : dirs) {
            int r = cell[0] + d[0];
            int c = cell[1] + d[1];
            if (r < 0 || r >= m || c < 0 || c >= n || 
                matrix[r][c] <= matrix[cell[0]][cell[1]] + 1) continue;
            queue.add(new int[] {r, c});
            matrix[r][c] = matrix[cell[0]][cell[1]] + 1;
        }
    }
    
    return matrix;
}
```

### 动态规划方法

动态规划分为两边整体扫描。用一个二维数组保存当前的 `distance`：
1. 第一遍扫描方向为从左到右，从上到下。扫描时在 `distance` 数组中赋值，如果当前点值不为 0，更新它为 `dis[i][j] = min(dis[i - 1][j], dis[i][j - 1]) + 1`。扫描完成后，每个节点都保存了距离左和上方向值为 0 的点的距离。
2. 第二遍扫描方向为从右到左，从下到上。如果当前点值不为 0，更新它为 `dis[i][j] = min(min(dis[i + 1][j], dis[i][j + 1]) + 1, dis[i][j])`。
3. 两边扫描结束后，`distance` 数组中保存了每个点的所求距离。

```
0 ... . ... 0
.     .     .
. ... 1 ... .
.     ~     .
0 *** * ... 0
```

可以如上图分别考虑「目标 0 点」在四个不同方向时，两边扫描都可以覆盖它们的最短路径（两条**折一次**的最短路径之一）。以左下角的 0 为例说明（假设 1 点对应的最近 0 点为左下角的点）：第一次扫描，可以确定 `distance` 数组中 `*` 路径上的点所对应的值为**最终值**；第二次扫描后可以确定 `~` 点对应值为最终值。经过两次扫描后，1 所在位置在数组中对应的值也为最终值。

```java
public int[][] updateMatrix(int[][] matrix) {
    if (matrix.length == 0 || matrix[0].length == 0) {
        return matrix;
    }
    int[][] dis = new int[matrix.length][matrix[0].length];
    int range = matrix.length * matrix[0].length;
    
    for (int i = 0; i < matrix.length; i++) {
        for (int j = 0; j < matrix[0].length; j++) {
            if (matrix[i][j] == 0) {
                dis[i][j] = 0;
            } else {
                int upCell = (i > 0) ? dis[i - 1][j] : range;
                int leftCell = (j > 0) ? dis[i][j - 1] : range;
                dis[i][j] = Math.min(upCell, leftCell) + 1;
            }
        }
    }
    
    for (int i = matrix.length - 1; i >= 0; i--) {
        for (int j = matrix[0].length - 1; j >= 0; j--) {
            if (matrix[i][j] == 0) {
                dis[i][j] = 0;
            } else {
                int downCell = (i < matrix.length - 1) ? dis[i + 1][j] : range;
                int rightCell = (j < matrix[0].length - 1) ? dis[i][j + 1] : range;
                dis[i][j] = Math.min(Math.min(downCell, rightCell) + 1, dis[i][j]);
            }
        }
    }
    
    return dis;
}
```

## 199 Binary Tree Right Side View

[199. Binary Tree Right Side View](https://leetcode.com/problems/binary-tree-right-side-view/)

> Given a binary tree, imagine yourself standing on the right side of it, return the values of the nodes you can see ordered from top to bottom.
```
Input: [1,2,3,null,5,null,4]
Output: [1, 3, 4]
Explanation:

   1            <---
 /   \
2     3         <---
 \     \
  5     4       <---
```

最容易想到的方法是通过 BFS 访问每一层的节点，自左向右地加入队列。在每一层计数，最后一个节点为 Right Side View 节点。也可称为层次遍历。

[另一种思路](https://leetcode.com/problems/binary-tree-right-side-view/discuss/56012)更为简洁，也很容易理解：使用递归的方式将当前节点的**深度**作为参数传递，函数的内部先递归地访问右子节点，再访问左子节点。在**每一个深度**，只有第一个访问的节点被加入到输出数组中。

```java
public List<Integer> rightSideView(TreeNode root) {
    List<Integer> result = new ArrayList<Integer>();
    rightView(root, result, 0);
    return result;
}
public void rightView(TreeNode curr, List<Integer> result, int currDepth){
    if(curr == null){
        return;
    }
    if(currDepth == result.size()){
        result.add(curr.val);
    }
    rightView(curr.right, result, currDepth + 1);
    rightView(curr.left, result, currDepth + 1);
}
```

## 116 Populating Next Right Pointers in Each Node

[116. Populating Next Right Pointers in Each Node](https://leetcode.com/problems/populating-next-right-pointers-in-each-node/)

> Given a binary tree
```
struct TreeLinkNode {
  TreeLinkNode *left;
  TreeLinkNode *right;
  TreeLinkNode *next;
}
```
> Populate each next pointer to point to its next right node. If there is no next right node, the next pointer should be set to `NULL`.
> Initially, all next pointers are set to `NULL`.

> **Note**:
- You may only use constant extra space.
- Recursive approach is fine, implicit stack space does not count as extra space for this problem.
- You may assume that it is a perfect binary tree (ie, all leaves are at the same level, and every parent has two children).

> Given the following perfect binary tree,
```
     1
   /  \
  2    3
 / \  / \
4  5  6  7
```
> After calling your function, the tree should look like:
```
     1 -> NULL
   /  \
  2 -> 3 -> NULL
 / \  / \
4->5->6->7 -> NULL
```

### 递归方法

递归地访问每个节点时，连接该节点的左子节点到右子节点，连接右子节点到该节点的 next 节点（如有）。

```java
public void connect(TreeLinkNode root) {
    if (root == null)
        return;
        
    if (root.left != null) {
        root.left.next = root.right;
        if (root.next != null)
            root.right.next = root.next.left;
    }
    
    connect(root.left);
    connect(root.right);
}
```

### 非递归方法

迭代的方法相当于按层次遍历每个节点，在访问每个节点时，连接其左右子节点。外层以层为单位，依次访问每一层的最左节点，由于访问一层的每个节点时，该层所有节点的 next 节点都已就绪，所以遍历时只需通过 `cur = cur.next` 访问下一个节点。

```java
    public void connect(TreeLinkNode root) {
        if (root == null)
            return;

        TreeLinkNode cur = root;
        while (cur.left != null) {
            TreeLinkNode nextLeftmost = cur.left;  // save the start of next level
            while (cur != null) {
                cur.left.next = cur.right;
                cur.right.next = cur.next == null ? null : cur.next.left;
                cur = cur.next;
            }
            cur = nextLeftmost;  // point to next level 
        }
    }
```

## 117 Populating Next Right Pointers in Each Node II

[117. Populating Next Right Pointers in Each Node II](https://leetcode.com/problems/populating-next-right-pointers-in-each-node-ii/)

> Given the following binary tree,
```
     1
   /  \
  2    3
 / \    \
4   5    7
```
> After calling your function, the tree should look like:
```
     1 -> NULL
   /  \
  2 -> 3 -> NULL
 / \    \
4-> 5 -> 7 -> NULL
```

### 非递归方法

在 116 题中非递归方法的算法基础上修改。

迭代地访问每一层节点，访问每个节点时，处理其左右子节点的连接。但由于每个节点的子节点情况不再固定，因此需要用其他方法**按层遍历节点**。两者另一显著区别是：本题中无法确定每（下）一层的**最左节点**。于是通过一个桩节点 `dummyNode` 来指向在每一层**第一个访问到**的节点。

`cur` 为当前层的移动指针；`p` 为下一层的移动指针。

```java
public void connect(TreeLinkNode root) {
    TreeLinkNode dummyNode = new TreeLinkNode(-1);
    TreeLinkNode cur = root;
    while (cur != null) {
        TreeLinkNode p = dummyNode;
        while (cur != null) {
            if (cur.left != null) {
                p.next = cur.left;  // link p.next
                p = p.next;  // jump to next node
            }
            if (cur.right != null) {
                p.next = root.right;  // link p.next
                p = p.next;  // jump to next node
            }
            cur = cur.next;
        }
        cur = dummyNode.next;  // move to next level left most
        dummyNode.next = null;
    }
}
```