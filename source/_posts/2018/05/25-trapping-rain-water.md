---
title: "[LC42/84] Trapping Rain Water / Largest Rectangle in Histogram / 动态规划 / 双指针"
date: 2018-05-25
tags: [leetcode, 动态规划, 双指针]
categories:
  - CS
  - LeetCode
---

这两题分别有不同的解法，但它们都可以用一种基于栈的方法来实现一次遍历计算结果。这种方法刚看到的时候不是很理解，思考后总结出其核心思想——确定迭代的每个步骤的「子任务」。这个选取的子任务通过循环迭代计算后，就可综合为最终结果（例如取最大/小值）。

<!-- more -->

# 42 Trapping Rain Water

[Trapping Rain Water - LeetCode](https://leetcode.com/problems/trapping-rain-water/description/)

Given n non-negative integers representing an elevation map where the width of each bar is 1, compute how much water it is able to trap after raining.

<div align="center"><img src="{{ site.baseurl }}/images/2018/05/rain-water-trap.png" width="60%"></div>

```
Input: [0,1,0,2,1,0,1,3,2,1,2,1]
Output: 6
```

## 一开始的错误考虑

因为联想到之前的 Container With Most Water，该题是每个坐标对应一根线（杆），找到能够容纳更多水的两个线框包围的区域。考虑到一个直观的想法：在每一「行」，找到最左端点和最右端点，中间的空白区域即为容纳水的部分。

这个算法的时间复杂度是 $O(n \times maxHeight)$。在最后一个较多数据的测试样例上超时了。

本题给出的参考答案思路比较清楚，且很容易能想到通过动态规划来**避免重复计算**。

## Brute Force

从每一个坐标位置（`i = 0..n-1`）来看，每个坐标（视为墙壁）上水的多少取决于左边最高墙壁**或**右边最高墙壁的高度。注意，根据木桶原理，取稍低的那一个的高度 `maxHeight`，水的容量等于 `maxHeight - height[i]`。

## 动态规划

可以发现，上述思路从 `i` 迭代到 `i+1` 时，重复考虑了 `0..i` 的长度，且可以利用 `leftMax[i-1]` 来得到 `leftMax[i] = max(leftMax[i-1], height[i])`。右侧同理。这就是动态规划的思想。

<div align="center"><img src="{{ site.baseurl }}/images/2018/05/rain-water-dp.png" width="100%"></div>

```c++
int trap(vector<int>& height) {
    if(height == null) return 0;
    int ans = 0;
    int size = height.size();
    vector<int> left_max(size), right_max(size);
    left_max[0] = height[0];
    for (int i = 1; i < size; i++) {
        left_max[i] = max(height[i], left_max[i - 1]);
    }
    right_max[size - 1] = height[size - 1];
    for (int i = size - 2; i >= 0; i--) {
        right_max[i] = max(height[i], right_max[i + 1]);
    }
    for (int i = 1; i < size - 1; i++) {
        ans += min(left_max[i], right_max[i]) - height[i];
    }
    return ans;
}
```

## 双指针

回顾「动态规划」一节的图示，可以发现红色部分和绿色部分的区别在于：当 `right_max[i]>left_max[i]` 时（坐标位置为0-6），容水量取决于 `left_max[i]`，右侧相反。

下图能更直观地显示这一区别。图中，中间两个位置分别代表 `i` 和 `j`，最左侧和最右侧分别指向的是 `i` 的左侧最大值 `leftMax` 和 `j` 的右侧最大值 `rightMax`。容易得到结论：如果如图中所示 `leftMax < rightMax`，那么 `i` 位置的容水量取决于 `leftMax`，为 `leftMax - height[i]`，此时 `j` 位置的容水量不能确定；反之。

<div align="center"><img src="{{ site.baseurl }}/images/2018/05/rain-water-2-pointers.png" width="60%"></div>

左右各设一个指针，向中间靠拢。用 `leftMax` 和 `rightMax` 的大小关系作为处理 `i` 还是 `j` 的判断依据，最后两者都将往最高处靠拢。

```java
    public int trap(int[] height) {
        int i = 0;
        int j = height.length - 1;
        int max = 0;
        int leftMax = 0;
        int rightMax = 0;
        while (i <= j) {
            leftMax = Math.max(leftMax, height[i]);
            rightMax = Math.max(rightMax, height[j]);
            if (leftMax < rightMax) {
                max += (leftMax - height[i]);
                i++;
            } else {
                max += (rightMax - height[j]);
                j--;
            }
        }
        return max;
    }
```

# 84 Largest Rectangle in Histogram

[Largest Rectangle in Histogram - LeetCode](https://leetcode.com/problems/largest-rectangle-in-histogram/description/)

Given n non-negative integers representing the histogram's bar height where the width of each bar is 1, find the area of largest rectangle in the histogram.

<div align="center"><img src="{{ site.baseurl }}/images/2018/05/histogram-area.png" width="30%"></div>

The largest rectangle is shown in the shaded area, which has area = `10` unit.

```
Input: [2,1,5,6,2,3]
Output: 10
```

## 选择用于迭代步计算的子任务

下面给出一种基于栈的解法，但**使用栈不是最终目的**，而是为了保存一些计算必要的中间结果。

循环的每个步骤，其实都在完成一个子任务。例如，本题的双重循环（Brute Force）解法将会在第一层的每次循环后，计算出第 $0..i$ 条 bar 为左边界的矩形面积的最小值。那么，当循环结束后，所有矩形面积的最小值就顺势推出了。

既然要算以 `x` 为矩形中最低的一条时矩形的面积，那就要找到这个矩形的左右边界（记为 `lb`, `rb`）。边界情况是： `lb` 相邻左边的一条长度比 `x` 小，`rb` 相邻右边的一条长度比 `x` 小。换句话说，就是在计算时需要知道这两条 bar 的索引（记为 `fli`, `fri`）。在这种基于栈的解法中，由于每条 bar 只进栈一次，那么当它出出栈的时候计算以它为最短 bar 所形成的矩形面积。循环标志 `i` 表示在这一循环内分别考虑所有以 `i` 为右边界的矩形。

通过「栈」来保存并获得这些信息：在从左到右地遍历 bar 时，每个 bar 仅进栈一次。当**当前 bar 的长度大于栈顶元素所表示的 bar 的长度**时，栈顶元素 `top` 出栈。这时，将 `top` 视作最短 bar，且当前循环的索引 `i` 代表它对应的 `fri`，原先栈中 `top` 之前的元素代表 `fli`。

这样，当循环结束后，也能够得到所有矩形面积的最小值。因为，**该结果必定包含在循环的计算中**。

具体算法如下：

1. 建栈。
2. 在每次循环中，`i=0..n-1`：
    1.  如果栈为空或者 `hist[i] >= hist[stack.top]`，`i` 进栈。
    2.  否则不断地将栈顶元素出栈只要栈顶元素更大，它代表的 bar 记为 `hist[tp]`。计算以 `hist[tp]` 为最短 bar 的矩形面积。`fli` 对应栈中 `tp` 的前一个元素，`fri` 对应当前循环的索引 `i`。
3. 如果栈不为空, 那么依次出栈并且都要重复上述循环中第二个步骤。

```java
public class Solution {
    public int largestRectangleArea(int[] height) {
        int len = height.length;
        Stack<Integer> s = new Stack<Integer>();
        int maxArea = 0;
        for(int i = 0; i <= len; i++){
            int h = (i == len ? 0 : height[i]);
            if(s.isEmpty() || h >= height[s.peek()]){   // 注意此处的符号
                s.push(i);
            }else{
                int tp = s.pop();
                maxArea = Math.max(maxArea, height[tp] * (s.isEmpty() ? i : i - 1 - s.peek()));
                i--;    // 注意
            }
        }
        return maxArea;
    }
}
```

Trapping Rain Water 问题可以依葫芦画瓢给出基于栈的解法。

```java
public int trap(int[] A) {
        if (A==null) return 0;
        Stack<Integer> s = new Stack<>();
        int i = 0, maxWater = 0, maxBotWater = 0;
        while (i < A.length){
            if (s.isEmpty() || A[i] <= A[s.peek()]){
                s.push(i++);
            }
            else {
                int bot = s.pop();
                maxBotWater = s.isEmpty() ? 0 : (Math.min(A[s.peek()], A[i])-A[bot]) * (i - 1 - s.peek());
                maxWater += maxBotWater;
            }
        }
        return maxWater;
    }
```

这是该解法的另一种写法。