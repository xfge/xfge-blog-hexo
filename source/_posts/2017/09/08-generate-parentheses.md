---
title: "[LC22] Generate Parentheses / 回溯"
date: 2017-09-08
tags: [leetcode, 回溯]
categories:
  - CS
  - LeetCode
---

[Generate Parentheses - LeetCode](https://leetcode.com/problems/generate-parentheses/)

>Given n pairs of parentheses, write a function to generate all combinations of well-formed parentheses.

<!-- more -->

>For example, given $n = 3$, a solution set is:
```
[
  "((()))",
  "(()())",
  "(())()",
  "()(())",
  "()()()"
]
```

## 错误理解

因为题目中给了 $n = 3$ 的例子，因此，思维定式地考虑：对于所有 $n=k$ 的括号串 `s_i`，分别构造 `()s_i`、`s_i()` 和 `(s_i)`，形成 $n=k+1$ 的所有可能括号串。

显然，如果验证一次 $n=4$ 就容易想到这种思路是错误的，因为没有考虑到这三种情况未能包括的其他情况，如：`(())(())`，这可以视为两个 $n=2$ 的括号串的拼接。

## Brute Force

Brute Force 方法是将 $2^2n$ 种由 `(` 和 `)` 组成的括号串依次检验其是否符合规则。检验时根据 `# of ( - # of ) >= 0` 来判断。

不再赘述。

## 解决两个子问题

上述错误思路实际上是来源于一个（正确的）思路：即解决一个**基数**较小的不相交子集的「总和」。思考如何用 $n<k$ 的已经计算的符号串集合来求解 $n=k$ 时的问题。

我最终提交的 AC 答案思路是：先将 $n=k-1$ 的所有符号串构造成 `(...)` 并加入 `ans`。另外考虑 `i = 1..k-1` 时所形成的每一个符号串 `left`，与 `j = n - i` 所形成的每个符号串 `right`，将两者拼接起来形成一种可能的符号串，并加入 `ans`。通过双层循环语句实现。

本题的 Solution 部分给出了这种方法的一个更简洁的描述。

* 定义一个有效括号串 `S` 的 closure number 为：使得`S[0]`, `S[1]`, ..., `S[2*index+1]` 有效的 `index >= 0` 的**最小值**。每个括号串有唯一的 closure number。
* 对每个括号串，考虑其 closure number `c`：`S[0]` 和 `S[2*c + 1]` 一定分别为 `(` 和 `)`，而且**其间的 `2*c` 个括号形成的串一定也有效**。加上剩余部分（也是有效的括号串），就形成了完整的括号串。

```java
class Solution {
    public List<String> generateParenthesis(int n) {
        List<String> ans = new ArrayList();
        if (n == 0) {
            ans.add("");
        } else {
            for (int c = 0; c < n; ++c)
                for (String left: generateParenthesis(c))
                    for (String right: generateParenthesis(n-1-c))
                        ans.add("(" + left + ")" + right);
        }
        return ans;
    }
}
```

时间和空间复杂度均为 $O \left ( \frac{4^n}{\sqrt{n}} \right )$。

## 回溯

回溯方法将一个括号串添加到 `ans` 中当且仅当这个括号串有效。因此，回溯方法需要维持已经放置的 `# of (` 和 `# of )`。

1. 如果**仍可放置** `(`，那么放置 `(`；
2. 如果 `# of ( > # of )`，那么放置 `)`。

```java
class Solution {
    public List<String> generateParenthesis(int n) {
        List<String> ans = new ArrayList();
        backtrack(ans, "", 0, 0, n);
        return ans;
    }

    public void backtrack(List<String> ans, String cur, int open, int close, int max){
        if (str.length() == max * 2) {
            ans.add(cur);
            return;
        }

        if (open < max)
            backtrack(ans, cur+"(", open+1, close, max);
        if (close < open)
            backtrack(ans, cur+")", open, close+1, max);
    }
}
```

时间复杂度与上述方法相同。