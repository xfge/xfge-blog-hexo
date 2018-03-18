---
layout: post
title: "[LC32] Longest Valid Parentheses / 动态规划"
date: 2017-09-29
tags: [leetcode, 动态规划]
categories:
  - CS
  - LeetCode
---

<a href="https://leetcode.com/problems/longest-valid-parentheses/">Longest Valid Parentheses - LeetCode</a>

> Given a string containing just the characters `'('` and `')'`, find the length of the longest valid (well-formed) parentheses substring.
For `"(()"`, the longest valid parentheses substring is `"()"`, which has length = 2.
Another example is `")()())"`, where the longest valid parentheses substring is `"()()"`, which has length = 4.

<!-- more -->

这道题 LeetCode 标注的 tag 是动态规划。琢磨了很久还是想不出和动态规划有关的算法。于是趁机复习了一遍动态规划，并重温了几个经典的动态规划问题的例子：最长公共子序列（longest-common-subsequence problem），最长公共子串（longest-common-substring problem），钢条切割问题（rod-cutting problem）等。

## 1. 动态规划

动态规划通常用来求解 **最优化问题**。这类问题可能有很多可行解，所以我们称这样的解为 _一个最优解_，而不是最优解。动态规划应用于子问题重叠的情况，对这些子问题只求解一次，将其解保存在 **一个表格** 中，从而避免重复计算。

通常按照以下4个步骤来设计一个动态规划算法：
1. 刻画一个最优解的结构特征。
2. 递归地定义最优解的值。
3. 计算最优解的值，通常采用自底向上的方法。
4. 利用计算出的信息构造一个最优解。

### 经典问题：钢条切割

钢条切割问题是：给定一段长度为 $n$ 英寸的钢条和一个价格表 $p_i (i=1, 2, \ldots, n)$，求切割钢条的方案，使得销售收益 $r_n$ 最大。

#### 递归实现

考虑一般情况，对于 $r_n (n \geqslant 1)$，我们用 _更短的钢条的最优切割收益_ 描述：

$$r_n = {\rm max}(p_n, r_1 + r_{n-1}, r_2 + r_{n-2}, \ldots, r_{n-1} + r_1)$$

这就是钢条切割问题的 **最优子结构** 性质：问题的最优解由相关子问题的最优解组合而成，而这些子问题可以独立求解。

至此，我们可以根据公式 $r_n = \underset{1 \leqslant i \leqslant n}{max}(p_i + r_{n-i})$ 实现一个 **自顶向下的递归实现**。下图所示的递归调用树显示了 $n=4$ 时，这样实现的递归函数的调用过程。由此可见，该递归函数的运行时间为 $n$ 的指数函数。

<div align="center"><img src="{{ site.baseurl }}/images/2017/10/rod-cutting-recursive.png" width="50%"></div>

#### 用动态规划求解

动态规划仔细安排求解顺序，使得每个子问题只需求解一次，并将结果保存下来。这是典型的 _时空权衡_ 的例子。如果子问题的数量是输入规模的多项式函数，而我们可以再多项式时间内求解出每个子问题，那么动态规划方法的总运行时间就是多项式阶的。

##### 两种实现方法

动态规划有两种等价的实现方法—— **带备忘的自顶向下法** 和 **自底向上法**。

带备忘的自顶向下法仍按自然的递归形式编写过程，但过程会保存每个子问题的解（用数组或散列表）。需要解的时候，如果已经保存则直接返回，否则再按通常方法计算。以下是钢条切割问题的自顶向下 CUT-ROD 过程的伪代码，加入了备忘机制。

<div align="center"><img src="{{ site.baseurl }}/images/2017/10/rod-cutting-memoization.png" width="70%"></div>


自底向上法需要恰当定义子问题“规模”的概念，使得任何子问题的求解都只依赖与“更小的”子问题的求解。当求解某个子问题时，它所依赖的更小的子问题都已求解完毕。每个子问题只需求解一次。以下是钢条切割求解的自底向上版本。

<div align="center"><img src="{{ site.baseurl }}/images/2017/10/rod-cutting-bottom-up.png" width="40%"></div>

上述的求解过程依次求解规模为 $j=1, 2, \ldots, n$ 的子问题。

两种方法具有相同的渐进运行时间—— $\Theta(n^2)$。不过，自底向上方法的时间复杂性函数通常具有更小的系数。

##### 子问题图

思考动态规划问题时，应该弄清涉及的子问题之间的依赖关系。下图是 $n=4$ 时钢条切割问题的 **子问题图**。

<div align="center"><img src="{{ site.baseurl }}/images/2017/10/rod-cutting-subproblem-graph.png" width="20%"></div>

值得一提的是，Youtube 的一位 Coder 播主 Tushar Roy 发布的算法讲解视频中有<a href="https://youtu.be/IRwVmTmN6go">这个经典案例</a>，相较于《CLRS》中琐细的解释，他给出表格式的解决方案更加清晰，我非常喜欢。看了许多他的视频，顺便治好了我的「印度🇮🇳口音恐惧症」。（逃

<div align="center"><iframe width="560" height="315" src="https://www.youtube.com/embed/IRwVmTmN6go" frameborder="0" allowfullscreen></iframe></div>

他列出了一个 $m \times n$ 的表格。这个表格中，第 $i$ 行第 $j$ 列（$i=1, 2, \ldots, m；j=0, 1, 2, \ldots, n$）表示：**将长度为 $j$ 的钢条（仅）使用 $1, 2, \ldots, i$ 长度切割（可能不包括每种长度）时所能取得的最优收益**。那么，计算到第 $m$ 行第 $n$ 列时就求得结果了。注意第 $0$ 列是为了便于下面的求解公式计算）。

这个行列的设定非常精妙，需要好好琢磨。

动态规划中，最优解的递归定义是重要的环节。钢条切割问题最优收益的 **递归求解公式** 为：

$$
T[i, j] = \begin{cases}
{\rm max}(T[i-1, j], p_i+T[i, j-i]) & \text{ if } j \geqslant  1 \\\\
T[i-1, j] & \text{ otherwise }
\end{cases}
$$

有了这个公式，代码编写就不算个事儿了。


## 2. 这道题

### 问题描述
> Given a string containing just the characters <code>'('</code> and <code>')'</code>, find the length of the longest valid (well-formed) parentheses substring.
<br>
For <code>"(()"</code>, the longest valid parentheses substring is <code>"()"</code>, which has length = 2.
<br>
Another example is <code>")()())"</code>, where the longest valid parentheses substring is <code>"()()"</code>, which has length = 4.


### 解决方案

#### 动态规划

基于前一节对动态规划的分析，只要求解出这道题的递归公式就可以了。

首先，定义一个一维数组`L[n]`，`L[i]`记录 **在第i位为结尾的最长有效子串的长度**。`L[n]`每一项初始化为 $0$。这样，就可以通过扫描一遍数组，在求得`L[n]`后，通过遍历`L[n]`（或在过程中记录）得到最优解。事实上，在遍历时我们只需检查`s[n] == ')'`的情况，因为`s[n] == '('`时条件总不会满足。

这个算法的运行时间为 $O(n)$，思路是：

$$
L[i] = \begin{cases}
L[i-2] + 2 & \text{ if } s[i] = ')', s[i-1] = '(' \\\\
L[i-1] + 2 + L[i - L[i-1] - 2] & \text{ if } s[i] = ')', s[i-1] = ')', s[i - L[i-1] - 1] = '(' \\\\
0 & \text{ otherwise }
\end{cases}
$$

> 注意：<br>
> 1. $i - L[i - 1] - 1 \leqslant 0$ 属于其他情况。<br>
> 2. 满足第二种情况时，字符串形式为：`"..*(....))"`，`*`后的`'('` _必然_ 与`i - 1`位的`')'`匹配。因此，检查`i - L[i - 1] - 1`位（即`*`所示位置）是否为`'('`，如果是，则完成了与`i`位`')'`的匹配。需要再加上`L[i - L[i - 1] - 2]`。

LeetCode 在该题的解答中给出了<a href="https://leetcode.com/articles/longest-valid-parentheses/#approach-2-using-dynamic-programming-accepted">较为简洁的代码</a>：

```java
public class Solution {
    public int longestValidParentheses(String s) {
        int maxans = 0;
        int dp[] = new int[s.length()];
        for (int i = 1; i < s.length(); i++) {
            if (s.charAt(i) == ')') {
                if (s.charAt(i - 1) == '(') {
                    dp[i] = (i >= 2 ? dp[i - 2] : 0) + 2;
                } else if (i - dp[i - 1] > 0 && s.charAt(i - dp[i - 1] - 1) == '(') {
                    dp[i] = dp[i - 1] + ((i - dp[i - 1]) >= 2 ? dp[i - dp[i - 1] - 2] : 0) + 2;
                }
                maxans = Math.max(maxans, dp[i]);
            }
        }
        return maxans;
    }
}
```

#### 利用「栈」

一种思想是使用「栈」数据结构。最初的想法是在栈中放置字符 <code>'('</code> 或 <code>')'</code> 。遍历字符串的每个字符时，如果当前字符是 <code>'('</code> ，就进栈；如果是 <code>')'</code> ，就出栈。这种想法源自于「算术表达式求值」问题。不过，在那个问题中，处理的不仅有括号，还有运算符，因此栈中元素选用字符。

而在这道题中，为了计算满足条件的最长子串的长度，一定要知道该子串的 **起始位置** 和 **结束位置** ，因此，栈的元素类型使用`Integer`。思路是这样的：

1. 从头至尾地扫描字符串。
2. 如果当前字符是 <code>'('</code> ，那么将它的索引值 **入栈**。如果当前字符是 <code>')'</code> 并且当前栈顶元素是 <code>'('</code> ，表示我们找到了一对相互匹配的括号，因此采取 **出栈** 操作。其他情况下，将 <code>')'</code> 的索引值 **入栈**。
3. 扫描完成后，栈中所有元素表示的是 _没有匹配成功的括号的索引值_。也即，每相邻的两个索引值之间的字符串为 _有效的字符串_。（注意首个索引值和最后一个索引值表示的含义）
4. 如果栈为空，整个字符串就是有效字符串。否则，还需要对栈中元素进行一次遍历，以求得最长子串的长度。

> 注意：第4步的遍历也可以替换为：在每次遇到`)`进栈时维护一个当前最长有效子串长度值。

##### 一个导致 TLE 的问题

用该方法第一次提交时，出现 TLE 错误，检查代码后发现原因出在上述第3步遍历栈元素时：

```java
int currentIndex = n;
while (!st.empty()) {
    int lastIndex = st.pop();
    result = Integer.max(currentIndex - lastIndex - 1, result);
    currentIndex = lastIndex;
}
result = Integer.max(currentIndex, result);
```

反复地声明新的变量导致超时。因此，此类反复使用的变量应该在循环外面声明。


#### 不需要额外空间的算法

LeetCode 还提供了一种空间复杂度为 $O(1)$ ，时间复杂度为 $O(n)$ 的<a href="https://leetcode.com/articles/longest-valid-parentheses/#approach-4-without-extra-space-accepted">算法</a>。通过维护两个变量`left`和`right`，分别左右遍历一次，就可求得最优解。思路如下：

1. 初始化`left`和`right`为0。
2. 首先从左至右地遍历每一个字符，如果遇到`'('`，则`left`自增1；如果遇到`')'`，则`'right'`自增1。
3. 当`left == right`时，当前匹配到一个有效子串，更新`currentMaxLenth`。当`right > left`时，重新将`left`和`right`置为0。
4. 从右至左地进行类似操作。
