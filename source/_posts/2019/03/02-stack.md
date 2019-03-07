---
title: "[LeetCode] Stack"
date: 2019-03-02
tags: [leetcode, 栈]
categories:
  - CS
  - LeetCode
---

本篇总结了栈的使用，关键是能有使用栈的意识，以及确定栈中保存的是什么元素。栈的使用也与许多树操作有关。

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


## 341 Flatten Nested List Iterator

[341. Flatten Nested List Iterator](https://leetcode.com/problems/flatten-nested-list-iterator/)

Given a nested list of integers, implement an iterator to flatten it.

Each element is either an integer, or a list -- whose elements may also be integers or other lists.

**Example 1:**

Input: [[1,1],2,[1,1]]
Output: [1,1,2,1,1]
Explanation: By calling next repeatedly until hasNext returns false, 
             the order of elements returned by next should be: [1,1,2,1,1].


这道题与 173 题极其类似，都是实现一个迭代器。刚做完 173 题时看到这题很快就想到只要倒序入栈，访问 next 元素时只要出栈并处理就可以了。但一开始犯了一个小错误，就是仍与 173 题一样通过 `stack.isEmpty()` 来决定 `hasNext()`，这就使一些很难处理的情况成为漏网之鱼：`[[[]],[]]`，这其实是空的，但栈中不空！

其实只要简单修改一下，让主要的判断操作放在 `hasNext()` 中，在该函数中就将需要 flatten 的 List 都 flatten，（对于 `true` 的情况只要）使栈顶保持是一个整数即可。此时也就更方便地排除 `[[[]],[]]` 这种情况了：这时只要经过判断返回 `false`，`next()` 就不会执行。注意最外层的 `while` 循环。

```java
public class NestedIterator implements Iterator<Integer> {

    Stack<NestedInteger> stack = new Stack<>();
    public NestedIterator(List<NestedInteger> nestedList) {
        for (int i = nestedList.size() - 1; i >= 0; i--) {
            stack.push(nestedList.get(i));
        }
    }

    @Override
    public Integer next() {
        return stack.pop().getInteger();
    }

    @Override
    public boolean hasNext() {
        while (!stack.isEmpty()) {
            if (stack.peek().isInteger()) {
                return true;
            }
            else {
                NestedInteger top = stack.pop();
                List<NestedInteger> list = top.getList();
                for (int i = list.size() - 1; i >= 0; i--) {
                    stack.push(list.get(i));
                }
            }
        }
        return false;
    }
}
```

## 385 Mini Parser

[385. Mini Parser](https://leetcode.com/problems/mini-parser/)

Given a nested list of integers represented as a string, implement a parser to deserialize it.

Each element is either an integer, or a list -- whose elements may also be integers or other lists.

Note: You may assume that the string is well-formed:

* String is non-empty.
* String does not contain white spaces.
* String contains only digits 0-9, [, - ,, ].


### 迭代式方法：使用栈

这道题与 394. Decode String 有点类似，都可以使用栈解决。只不过题目中有些地方没有明确，比如 `[[]]` 这种 corner case，很恶心。

刚拿到题的时候，因为刚做了 Decode String，所以理所当然地想到用 String 栈来存储 token（也可能是受到栈的景点应用——算术表达式处理——的影响），将 `[`、`]` 和数字分别存到字符串栈中，再用另一个栈保存 NestedInteger，且没有考虑清除在何时将数字进栈（最初是想在每次遇到 `]` 时将栈中 `[` 之前的数字逐一进栈，但这样就忽略了已经形成列表的 NestedInteger），总之思路变得异常繁琐。做了一百题下来，我发现**思路不清楚让解法变得很啰嗦的时候很有可能就是错了！**

正确的迭代式解法只需要一个栈，栈中元素是 NestedInteger。依次访问字符串中的每个字符，当遇到 `[` 时创建一个新的 NestedInteger 对象，类型是一个 empty nested list。当遇到 `,` 或 `]` 时表示（可能）结束了一个数字，将数字添加到之前的 NesteInteger 中也在此刻进行。需要额外考虑的是 `]` 的情况：将 list 添加到其所属 list 的操作也在此刻一并解决（可以思考一下这种情况）。这样分解下来看，其实每个步骤都很简单。最后再加上特殊情况的考虑。

这才是正确解法该有的样子😂。


```java
/**
 * // This is the interface that allows for creating nested lists.
 * // You should not implement it, or speculate about its implementation
 * public interface NestedInteger {
 *     // Constructor initializes an empty nested list.
 *     public NestedInteger();
 *
 *     // Constructor initializes a single integer.
 *     public NestedInteger(int value);
 *
 *     // @return true if this NestedInteger holds a single integer, rather than a nested list.
 *     public boolean isInteger();
 *
 *     // @return the single integer that this NestedInteger holds, if it holds a single integer
 *     // Return null if this NestedInteger holds a nested list
 *     public Integer getInteger();
 *
 *     // Set this NestedInteger to hold a single integer.
 *     public void setInteger(int value);
 *
 *     // Set this NestedInteger to hold a nested list and adds a nested integer to it.
 *     public void add(NestedInteger ni);
 *
 *     // @return the nested list that this NestedInteger holds, if it holds a nested list
 *     // Return null if this NestedInteger holds a single integer
 *     public List<NestedInteger> getList();
 * }
 */
class Solution {
    public NestedInteger deserialize(String s) {
        if (s.charAt(0) != '[') {
            return new NestedInteger(Integer.parseInt(s));
        }
        Stack<NestedInteger> stack = new Stack<>();
        int start = 0;
        for (int i = 0; i < s.length(); i++) {
            char ch = s.charAt(i);
            if (ch == '[') {
                stack.push(new NestedInteger());
                start = i + 1;
            }
            else if (ch == ']' || ch == ',') {
                if (i - start > 0) {
                    stack.peek().add(new NestedInteger(Integer.parseInt(s.substring(start, i))));
                }
                if (ch == ']' && stack.size() > 1) {
                    NestedInteger top = stack.pop();
                    stack.peek().add(top);
                }
                start = i + 1;
            }
        }
        return stack.pop();
    }
}
```

### 递归方法

递归方法是没想起来，不过看了解答之后觉得整个思路也挺清晰的。

在递归函数的内部，首先是停止条件，有两个：
1. 字符串为空的时候（对应的是 `[]` 的情况），直接返回 empty list。
2. 字符串是单个数字的时候（只需判断首字符是否为 `[`），返回解析后的整数类型的 NestedInteger 对象。

其他情况就都需要递归了，递归时，首先要做的是分隔「当前层」（最外层）的每个外层 token（指的是整数或列表类型的 NestedInteger），当访问到一个逗号且此刻并未进入某一组 `[]` 时，就分割出了一个「外层 token」，对每个 token 递归调用函数本身，将返回的 NestedInteger 加入到返回变量中即可。至于如何确定每个外层 token 的起始位置，则利用与迭代式方法中类似的 `start` 变量来记录起始位置，该变量只当处理过上一个 token 之后才被刷新。

```java
public class Solution {
    public NestedInteger deserialize(String s) {
        NestedInteger ret = new NestedInteger();
        if (s == null || s.length() == 0) return ret;
        if (s.charAt(0) != '[') {
            ret.setInteger(Integer.parseInt(s));
        }
        else if (s.length() > 2) {
            int start = 1, count = 0;
            for (int i = 1; i < s.length(); i++) {
                char c = s.charAt(i);
                if (count == 0 && (c == ',' || i == s.length() - 1)) {
                    ret.add(deserialize(s.substring(start, i)));
                    start = i + 1;
                }
                else if (c == '[') count++;
                else if (c == ']') count--;
            }
        }
        return ret;
    }
}
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

## 173 Binary Search Tree Iterator

[173. Binary Search Tree Iterator](https://leetcode.com/problems/binary-search-tree-iterator/)

Implement an iterator over a binary search tree (BST). Your iterator will be initialized with the root node of a BST.

Calling `next()` will return the next smallest number in the BST.

**Note:**

`next()` and `hasNext()` should run in average O(1) time and uses O(h) memory, where h is the height of the tree.
You may assume that `next()` call will always be valid, that is, there will be at least a next smallest number in the BST when `next()` is called.

##

又是一道不看标签🏷想不到用栈实现的题（当然，即使知道用栈也不知道怎么做）。[这则帖子](https://leetcode.com/problems/binary-search-tree-iterator/discuss/52526)其实提示了一些通过题目中的注意点来得到解题启发的想法。比如，「in average O(1) time」可能想到：`next()` 调用通常可以直接访问到下一个节点，但偶尔允许通过 path 访问子节点的方式在 O(h) 下解决。（只是思考）

接下来思考如何实现 `next()` 函数，对于某个节点，如果它存在右分支，那么通过访问它的右分支再循环访问其左分支，最后得到的就是 next 节点。这其实和「通过迭代方式中序遍历树」很相似。然而，当它不存在右分支时，（注意左分支元素都比当前元素小）如何得到父节点（以继续寻找 next 节点）？可以发现 next 节点要么在右分支下，要么在「根节点到该节点」的通路上。而在最初访问该节点的遍历过程中，将访问过的节点依次保存到栈中，便可按逆序出栈。这也是中序遍历过程。

```java
public class BSTIterator {
    
    private Stack<TreeNode> stack;
    public BSTIterator(TreeNode root) {
        stack = new Stack<>();
        TreeNode cur = root;
        while(cur != null){
            stack.push(cur);
            if(cur.left != null)
                cur = cur.left;
            else
                break;
        }
    }

    /** @return whether we have a next smallest number */
    public boolean hasNext() {
        return !stack.isEmpty();
    }

    /** @return the next smallest number */
    public int next() {
        TreeNode node = stack.pop();
        TreeNode cur = node;
        // traversal right branch
        if(cur.right != null){
            cur = cur.right;
            while(cur != null){
                stack.push(cur);
                if(cur.left != null)
                    cur = cur.left;
                else
                    break;
            }
        }
        return node.val;
    }
}
```


## 331 Verify Preorder Serialization of a Binary Tree

[331. Verify Preorder Serialization of a Binary Tree](https://leetcode.com/problems/verify-preorder-serialization-of-a-binary-tree/)

One way to serialize a binary tree is to use pre-order traversal. When we encounter a non-null node, we record the node's value. If it is a null node, we record using a sentinel value such as #.

```
     _9_
    /   \
   3     2
  / \   / \
 4   1  #  6
/ \ / \   / \
# # # #   # #
```

For example, the above binary tree can be serialized to the string "9,3,4,#,#,1,#,#,2,#,6,#,#", where # represents a null node.

Given a string of comma separated values, verify whether it is a correct preorder traversal serialization of a binary tree. Find an algorithm without reconstructing the tree.

Each comma separated value in the string must be either an integer or a character '#' representing null pointer.

You may assume that the input format is always valid, for example it could never contain two consecutive commas such as "1,,3".

**Example 1:**

Input: "9,3,4,#,#,1,#,#,2,#,6,#,#"
Output: true

**Example 2:**

Input: "1,#"
Output: false

**Example 3:**

Input: "9,#,#,1"
Output: false

## 使用栈不断收缩

<div align="center"><img src="{{ site.baseurl }}/images/2019/03/04-target-sum-dp1.png" width="40%"></div>

在 “stack” 标签的提示下容易想到这种解法。其实与树的迭代式遍历相关的问题都会与栈有关。不过一开始做的时候在「遍历时套循环来解决栈不断弹出」部分的时候遇到了问题，当时的思路是一遍一遍地遍历 LinkedList 处理 “X##” 类型的连续串知道串不在减小，好像也可以，但是显得很复杂。要灵活用好循环结构。不过下面这段代码其实有一些地方值得优化，但优化后的代码虽更高效但也更复杂一些。

```java
    public boolean isValidSerialization(String preorder) {
        Stack<String> stack = new Stack<>();
        String[] list = preorder.split(",");
        for (int i = 0; i < list.length; i++) {
            stack.push(list[i]);
            while (stack.size() >= 3 && stack.peek().equals("#") && stack.elementAt(stack.size() - 2).equals("#") && !stack.elementAt(stack.size() - 3).equals("#")) {
                stack.pop();
                stack.pop();
                stack.pop();
                stack.push("#");
            }
        }
        return (stack.size() == 1 && stack.peek().equals("#"));
    }
```

## 另一种解法：出度和入度

如果将二叉树的边看做是从父节点到子节点的有向边，利用图论中「出度和入度」的定义，可知：
* 非叶节点（非“#”节点）的出度为2，入度为1；
* 叶节点（“#”节点）的出度为0，入度为1。

即每个节点的入读相同，都为1。在通过前序遍历来重绘树结构时，跟踪一个变量 `diff = out_degree - in_degree`。此外，**为了让每个点都有1个入度**，给根节点一个“悬浮”的入边（使其与其他节点保持一致）。这样，当树重绘完成后，所有节点的出度与入度之和应相等（加上根节点的“悬浮”入边对应的点），且在遍历访问每个节点时，diff 都不会为负（那样意味着在没有相应入边时就绘制了点）。

[YouTube 上的一则视频](https://www.youtube.com/watch?v=_mbnPPHJmTQ)用例子分步讲解了算法过程。

```java
public boolean isValidSerialization(String preorder) {
    String[] nodes = preorder.split(",");
    int diff = 1;
    for (String node: nodes) {
        if (--diff < 0) return false;
        if (!node.equals("#")) diff += 2;
    }
    return diff == 0;
}
```

## 402 Remove K Digits

[402. Remove K Digits](https://leetcode.com/problems/remove-k-digits/)

Given a non-negative integer num represented as a string, remove k digits from the number so that the new number is the smallest possible.

Note:
* The length of num is less than 10002 and will be ≥ k.
* The given num does not contain any leading zero.

**Example 1:**
```
Input: num = "1432219", k = 3
Output: "1219"
Explanation: Remove the three digits 4, 3, and 2 to form the new number 1219 which is the smallest.
```

这道题还是比较容易想到的。只要「每次删的字符能保证是所有可能情况中最小的」就可以了——其实是一种贪心思想。我在实现的时候是在循环中，找到一个比后一位大的数字就删除，并 break 进入下一次循环。这一过程中出现了一些冗余：看了其他人的解答一下子反应过来可以用栈（考虑到最后需要删除字符串首的0，可以用双端队列）来实现。下面的解法使用了双端队列。


```java
public String removeKdigits(String num, int k) {        
    if(k >= num.length()) return "0";

    Deque<Character> stack = new ArrayDeque<>();
    for(char c : num.toCharArray()) {
        while(k > 0 && !stack.isEmpty() && stack.peekLast() > c) {
            stack.removeLast();
            k--;
        }
        stack.addLast(c);
    }
    
    while(k>0) {
        stack.removeLast();
        k--;
    }
    
    // Remove all zeros from the front of the stack and then if stack is empty, return "0"
    while(!stack.isEmpty() && stack.peekFirst()== '0') stack.removeFirst();
    if(stack.isEmpty()) return "0";

    // build the number from the stack
    StringBuilder sb = new StringBuilder();
    while(!stack.isEmpty()) {
        sb.append(stack.removeFirst());
    }
    return sb.toString();
}
```