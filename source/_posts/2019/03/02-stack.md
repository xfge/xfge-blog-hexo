---
title: "[LeetCode] Stack / Queue / Heap"
date: 2019-03-02
tags: [leetcode, 栈]
categories:
  - CS
  - LeetCode
---

本篇总结了栈的使用，关键是能有使用栈的意识，以及确定栈中保存的是什么元素。

<!-- more -->

<!-- # 基本概念 -->


# 相关问题

## 394 Decode String

[394. Decode String](https://leetcode.com/problems/decode-string/)

将字符串用 `k[encoded_string]` 的方式编码，将通过这种方式编码的字符串解码。以下有几个例子。

```
s = "3[a]2[bc]", return "aaabcbc".
s = "3[a2[c]]", return "accaccacc".
s = "2[abc]3[cd]ef", return "abcabccdcdcdef".
```

### 使用两个栈

两个栈分别保存数字和字符串，数字栈保存字符串重复次数，字符串栈保存当前解码后的字符串。将待解码串从左到右地遍历：
1. 当遇到数字时，判断是否为多位数，将解析后的整数入数字栈；
2. 当遇到左括号时，先将已读取字符串 `res` 入栈，再将其清空，准备开始读取括号内部的新字符串；
3. 当遇到右括号时，数字栈出栈，栈顶元素为 `k`；字符串栈出栈，栈顶元素为 `temp`，将 `res` 赋值为 `temp` + k * `res`。
4. 最后 `res` 即为解码后的字符串。

```java
public class Solution {
    public String decodeString(String s) {
        String res = "";
        Stack<Integer> countStack = new Stack<>();
        Stack<String> resStack = new Stack<>();
        int idx = 0;
        while (idx < s.length()) {
            if (Character.isDigit(s.charAt(idx))) {
                int count = 0;
                while (Character.isDigit(s.charAt(idx))) {
                    count = 10 * count + (s.charAt(idx) - '0');
                    idx++;
                }
                countStack.push(count);
            }
            else if (s.charAt(idx) == '[') {
                resStack.push(res);
                res = "";
                idx++;
            }
            else if (s.charAt(idx) == ']') {
                StringBuilder temp = new StringBuilder (resStack.pop());
                int repeatTimes = countStack.pop();
                for (int i = 0; i < repeatTimes; i++) {
                    temp.append(res);
                }
                res = temp.toString();
                idx++;
            }
            else {
                res += s.charAt(idx++);
            }
        }
        return res;
    }
}
```


### 一个更直接的递归解法

```c++
class Solution {
public:
    string decodeString(const string& s, int& i) {
        string res;
        while (i < s.length() && s[i] != ']') {
            if (!isdigit(s[i]))
                res += s[i++];
            else {
                int n = 0;
                while (i < s.length() && isdigit(s[i]))
                    n = n * 10 + s[i++] - '0';
                    
                i++; // '['
                string t = decodeString(s, i);
                i++; // ']'
                
                while (n-- > 0)
                    res += t;
            }
        }
        return res;
    }

    string decodeString(string s) {
        int i = 0;
        return decodeString(s, i);
    }
};
```

使用递归的方法，每一次递归地调用 `decodeString(const string&, int&)` 都执行一次对形如 `k[encoded_string]` 的解码。注意该函数签名，两个参数都使用其引用，因此函数内部的 `i++` 对后续的 index 遍历都有影响。

以 `3[a2[c]]2[d]` 为例，`decodeString` 被依次调用时，返回的字符串分别是（TO CHECK）：
```
res = cc
res = acc
res = accaccacc
res = dd
res = accaccaccdd
```


## 739 Daily Temperatures

[739. Daily Temperatures](https://leetcode.com/problems/daily-temperatures/)

对给定的一组表示气温的数组，输出每天的「最近的更暖和的一天」距现在还有几天。For example, given the list of temperatures T = [73, 74, 75, 71, 69, 72, 76, 73], your output should be [1, 1, 4, 2, 1, 1, 0, 0].

### 使用栈记录

本题要求的输出与数组索引密切相关。对于某个气温 `T[i]`，如果能在访问它时确定最近一个高温天是哪项，可能需要从后往前遍历，维持一个严格递增的索引序列，能从该序列中得到 `T[i]` 的最近高温天。例如，假设 `T[10] = 70`, 序列为 `[20(60), 30(80), 40(90)]`，那么只需要将 20 出栈，就能得到最近高温天。注意，由于是从后往前遍历，`i = 20` 对应的那天已经赋值过，且 60 度在后续的遍历中不再需要（因为必会选择新入栈的 `10(70)`）。

从另一个角度考虑，如果对于某个气温 `T[i]`，能在访问它时，确定**与它相关**（即将它作为最近高温天的那些天）的所有索引，也能将其赋值，这种情况下，栈中保存的是一组**未确定**的数组索引。每访问到一个索引位置 `T[i]`，不断出栈并比较栈顶索引所对应的气温是否小于 `T[i]`，如果小于就知道 `T[i]` 是它的最近高温天。每次出栈就完成一次赋值，每个索引只会进栈出栈一次，因此直到最后所有索引都将被赋值（未赋值的都为0）。

总而言之，还是没能清楚地阐述这里的道理。以后再来填坑吧。

#### 从后往前地确定

```java
  public int[] dailyTemperatures(int[] T) {
      int[] ans = new int[T.length];
      Stack<Integer> stack = new Stack();
      for (int i = T.length - 1; i >= 0; --i) {
          while (!stack.isEmpty() && T[i] >= T[stack.peek()]) stack.pop();
          ans[i] = stack.isEmpty() ? 0 : stack.peek() - i;
          stack.push(i);
      }
      return ans;
  }
```

#### 从前往后地确定

```java
public int[] dailyTemperatures(int[] temperatures) {
    Stack<Integer> stack = new Stack<>();
    int[] ret = new int[temperatures.length];
    for(int i = 0; i < temperatures.length; i++) {
        while(!stack.isEmpty() && temperatures[i] > temperatures[stack.peek()]) {
            int idx = stack.pop();
            ret[idx] = i - idx;
        }
        stack.push(i);
    }
    return ret;
}
```


### 利用「温度可选范围小」的特点，遍历并不复杂

另一种方法也是从右到左地遍历，用另一个数组 `next[]` 保存每个温度在访问发生时的下一个索引位置。每遍历到一个新索引`i`，都检查从 `T[i]` 到 `T[100]` 在 `next` 数组中的索引值，取其最小。这种方法比较易懂，主要是利用了一个小技巧，就是温度可枚举（很少），而常数项对算法复杂度的影响不大。

```java
class Solution {
    public int[] dailyTemperatures(int[] T) {
        int[] ans = new int[T.length];
        int[] next = new int[101];
        Arrays.fill(next, Integer.MAX_VALUE);
        for (int i = T.length - 1; i >= 0; --i) {
            int warmer_index = Integer.MAX_VALUE;
            for (int t = T[i] + 1; t <= 100; ++t) {
                if (next[t] < warmer_index)
                    warmer_index = next[t];
            }
            if (warmer_index < Integer.MAX_VALUE)
                ans[i] = warmer_index - i;
            next[T[i]] = i;
        }
        return ans;
    }
}
```