---
layout: post
title: "[LC3] Longest Substring Without Repeating Characters"
tags: [leetcode]
---

[Longest Substring Without Repeating Characters - LeetCode](https://leetcode.com/problems/longest-substring-without-repeating-characters/)

> Given a string, find the length of the longest substring without repeating characters.<br>
>Examples:<br>
>Given `"abcabcbb"`, the answer is `"abc"`, which the length is 3.<br>
>Given `"bbbbb"`, the answer is `"b"`, with the length of 1.<br>
>Given `"pwwkew"`, the answer is `"wke"`, with the length of 3. Note that the answer must be a **substring**, `"pwke"` is a subsequence and not a substring.

## 直觉上的解法 [Accepted]

LeetCode的第三题，看到题目后首先想到的解法是通过一个整型数组`index[]`来表示 **当前位置往前数不含重复字符的最长字符串（LSWRC）中每个字符所在的位置**。从首个字符开始遍历，同时保存当前LSWRC长度以比较。最终输出LSWRC长度。

在第`i`轮迭代步骤中，如果`index[i]`不是-1（初始化值），则说明该字符 **在当前LSWRS中已经出现过一次**。记当前LSWRC起始位置为`begIndex`，那么需要将`index[]`中索引为`begIndex`到`i - 1`的所有值都置为-1；并将`begIndex`重新赋值为`i`。

## 过程优化

提交实现上述算法的代码运行结果超越了90%用户，但其时间复杂度应为 $$O(n^2)$$。如前所述，它嵌套了一层循环（重新赋值）。

[11-line simple Java solution, O(n) with explanation](https://leetcode.com/problems/longest-substring-without-repeating-characters/discuss/1729/11-line-simple-Java-solution-O(n)-with-explanation) 中给出了一种时间复杂度为 $$O(n)$$ 的简化算法：

```java
public int lengthOfLongestSubstring(String s) {
     if (s.length() == 0) return 0;
     HashMap<Character, Integer> map = new HashMap<Character, Integer>();
     int max=0;
     for (int i = 0, j = 0; i < s.length(); ++i){
         if (map.containsKey(s.charAt(i))){
             j = Math.max(j, map.get(s.charAt(i)) + 1);
         }
         map.put(s.charAt(i), i);
         max = Math.max(max, i - j + 1);
     }
     return max;
 }
```

其思想大致相同，但免去了「当遇到重复字符时逐个删除」的过程，而是直接将其赋值为 **字符最近出现的位置**。这里，其他所有 **跳过** 的字符不需修改，因为已经在 `j = Math.max(j, map.get(s.charAt(i)) + 1);` 一句中考虑了这一情况：
- 如果当前字符在 `j` 之前（即 **被跳过** 了），不影响 `j` 的值；
- 如果当前字符在 `j` 之后，则根据其位置修改 `j` 值。

根据这个思想修改首次提交代码：

```c++
int lengthOfLongestSubstring(string s) {
  int first[256];
  for (int i = 0; i < 256; i++)
    first[i] = -1;
  int max = 0;
  int indexBegin = 0;
  for (int i = 0; i < s.length(); i++) {
      if (first[s[i]] != -1) {
          if (first[s[i]] + 1 > indexBegin) {
              indexBegin = first[s[i]] + 1;
          }
      }
      first[s[i]] = i;
      if (i - indexBegin + 1 > max) {
          max = i - indexBegin + 1;
      }
  }
  return max;
}
```
