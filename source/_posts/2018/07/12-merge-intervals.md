---
title: "[LC56] Merge Intervals 图连通性 / 排序"
date: 2018-07-12
tags: [leetcode]
categories:
  - CS
  - LeetCode
---

将给定的一组区间表示合并起来——将部分重叠或端点重合的区间合并起来。

<!-- more -->

[Merge Intervals - LeetCode](https://leetcode.com/problems/merge-intervals/description/)

Given a collection of intervals, merge all overlapping intervals.
**Example 1:**
```
Input: [[1,3],[2,6],[8,10],[15,18]]
Output: [[1,6],[8,10],[15,18]]
Explanation: Since intervals [1,3] and [2,6] overlaps, merge them into [1,6].
```
**Example 2:**
```
Input: [[1,4],[4,5]]
Output: [[1,5]]
Explanation: Intervals [1,4] and [4,5] are considerred overlapping.
```

上次看完这道题没有思路，又恰逢期末考试，就迟迟没有动笔。刚看到题目，先是考虑将所有元素依次排序，再根据原始位置与排序后的新位置的变化关系找规律。但好像没有什么结论。这反映了我还是缺乏将算法理论用于实际问题的转换能力。

本题给出的答案一个是从连通分量的思路出发，另外两者都是将 intervals 通过某个规则排序后进行一次扫描处理。

## 连通分量

将每个 interval 视作图中的一个顶点；如果两个 interval 存在重叠的情况，添加两者间的一条边。在实际实现时，用邻接表（Adjacency List）来储存图的相邻关系。用顶点间的两条有向边表示一条无向边：在邻接表中分别添加 `v1 -> v2` 和 `v2 -> v1`。

如何给出 “两个 inverval 存在重叠” 的条件？考虑相反的条件：两个 interval 不重叠。如果用 `s1/e1` 和 `s2/e2` 分别表示两个顶点的两个端点，那么必有 `e1 < s2 || e2 < s1`，注意，补全其中的隐含关系将得到（以前者为例）：`s1 < e1 < s2 < e2`，我们只需考虑中间的不等关系。

那么，将上述条件取反就可得到 `e1 >= s2 && e2 >= s1`，可用这个条件作为检查重叠的依据。

然后就要检查每个顶点所属的连通分量，并将该连通分量中的所有顶点所表示的 interval 进行合并。探索连通分量时，使用 DFS 或 BFS 并通过一个集合来记录所有访问过的顶点（interval）。合并时，只需将集合中所有的 start point 的最小值和 end point 的最大值作为该连通分量所对应的新 interval 即可。

<div align="center"><img src="{{ site.baseurl }}/images/2018/07/connected-component.png" width="50%"></div>

下面的代码可作为连通分量一类问题的参考。

该算法的时间复杂度和空间复杂度都是 $O(n^2)$。

```java
class Solution {
    private Map<Interval, List<Interval> > graph;
    private Map<Integer, List<Interval> > nodesInComp;
    private Set<Interval> visited;

    // return whether two intervals overlap (inclusive)
    private boolean overlap(Interval a, Interval b) {
        return a.start <= b.end && b.start <= a.end;
    }

    // build a graph where an undirected edge between intervals u and v exists
    // iff u and v overlap.
    private void buildGraph(List<Interval> intervals) {
        graph = new HashMap<>();
        for (Interval interval : intervals) {
            graph.put(interval, new LinkedList<>());
        }

        for (Interval interval1 : intervals) {
            for (Interval interval2 : intervals) {
                if (overlap(interval1, interval2)) {
                    graph.get(interval1).add(interval2);
                    graph.get(interval2).add(interval1);
                }
            }
        }
    }

    // merges all of the nodes in this connected component into one interval.
    private Interval mergeNodes(List<Interval> nodes) {
        int minStart = nodes.get(0).start;
        for (Interval node : nodes) {
            minStart = Math.min(minStart, node.start);
        }

        int maxEnd = nodes.get(0).end;
        for (Interval node : nodes) {
            maxEnd= Math.max(maxEnd, node.end);
        }

        return new Interval(minStart, maxEnd);
    }

    // use depth-first search to mark all nodes in the same connected component
    // with the same integer.
    private void markComponentDFS(Interval start, int compNumber) {
        Stack<Interval> stack = new Stack<>();
        stack.add(start);

        while (!stack.isEmpty()) {
            Interval node = stack.pop();
            if (!visited.contains(node)) {
                visited.add(node);

                if (nodesInComp.get(compNumber) == null) {
                    nodesInComp.put(compNumber, new LinkedList<>());
                }
                nodesInComp.get(compNumber).add(node);

                for (Interval child : graph.get(node)) {
                    stack.add(child);
                }
            }
        }
    }

    // gets the connected components of the interval overlap graph.
    private void buildComponents(List<Interval> intervals) {
        nodesInComp = new HashMap();
        visited = new HashSet();
        int compNumber = 0;

        for (Interval interval : intervals) {
            if (!visited.contains(interval)) {
                markComponentDFS(interval, compNumber);
                compNumber++;
            }
        }
    }

    public List<Interval> merge(List<Interval> intervals) {
        buildGraph(intervals);
        buildComponents(intervals);

        // for each component, merge all intervals into one interval.
        List<Interval> merged = new LinkedList<>();
        for (int comp = 0; comp < nodesInComp.size(); comp++) {
            merged.add(mergeNodes(nodesInComp.get(comp)));
        }

        return merged;
    }
}
```

## 使用排序方法简化计算

使用排序方法解决的关键问题是如何排序以及排序后如何判定重叠？下面两个方法给出了略微不同的思路。

### 按照 start 值排序

首先将 intervals 按照 start 值排序。排序后，一次扫描 intervals 就能完成合并。用 mergedInterval 表示当前扫描到的 interval 所属的 interval。一旦**后继 interval** 的起始端点大于**当前 mergedInterval** 的终止端点，那么后续不再存在与**当前 mergedInterval** 重合的 interval。

上述说法有些啰嗦，其实是在解决如何判定重叠的问题。我刚开始实现这一方法时，通过 `intervals.get(i).end < intervals.get(i + 1).start` 判断重叠，但遇到了反例（见下图6️⃣）。于是想到，用于比较的应该是当前 mergedInterval 的 end 值**而不是前一个 interval 的 end 值**。用中间变量 `currentEnd` 保存该值，每当重叠发生时，用较大的 end 值覆盖旧值。

<div align="center"><img src="{{ site.baseurl }}/images/2018/07/overlapping-cases.png" width="50%"></div>

> 能够考虑多种可能的情况是必要的。同时也应该注意要想比较不一定是通过纸面上的数值，有些时候需要保存一些之前的值（动态规划思想）。

如何说明该方法的正确性？毕竟，只是不完备地穷举出可能的情况不能作为证明。

LeetCode 给出一种用反证法说明正确性的叙述。首先，假设在某个时刻算法并未将两个本应合并的 intervals 合并（注：这两个 interval 必定不连续，因为这种情况我们已经完备地处理了）。那就表明存在三个数组索引 $i, j$ 和 $k$ 满足 $i < j < k$ 且 `(intervals[i], intervals[k])` 满足合并条件但 `(intervals[i], intervals[j])` 和 `(intervals[j], intervals[k])` 均不满足。那么，有不等式：
$$
ints[i].end < ints[j].start \\\\
ints[j].end < ints[k].start \\\\
ints[i].end \geq ints[k].start \\\\
$$

那么将会导出矛盾（与上面第三个不等式矛盾）：
$$
ints[i].end < ints[j].start < ints[j].end < ints[k].start
$$

算法的时间复杂度是 $O(n\log(n))$。以下给出两种略有不同的实现方式。前者在确定一个完整的 merged interval 后才将其放到结果中；后者每次先将其加入到结果中，后续迭代时再比较并修改 end value of last interval。

```java
    public List<Interval> merge(List<Interval> intervals) {
        Collections.sort(intervals, new Comparator<Interval>() {
            @Override
            public int compare(Interval o1, Interval o2) {
                return o1.start - o2.start;
            }
        });

        List<Interval> result = new ArrayList<>();
        int j = 0;
        int currentEnd = 0;
        for (int i = 0; i < intervals.size(); i++) {
            currentEnd = Integer.max(currentEnd, intervals.get(i).end);
            if (i == intervals.size() - 1 || currentEnd < intervals.get(i + 1).start) {
                result.add(new Interval(intervals.get(j).start, currentEnd));
                j = i + 1;
            }
        }

        return result;
    }
```

```java
    public List<Interval> merge(List<Interval> intervals) {
        Collections.sort(intervals, new Comparator<Interval>() {
            @Override
            public int compare(Interval o1, Interval o2) {
                return o1.start - o2.start;
            }
        });

        LinkedList<Interval> merged = new LinkedList<Interval>();
        for (Interval interval : intervals) {
            // if the list of merged intervals is empty or if the current
            // interval does not overlap with the previous, simply append it.
            if (merged.isEmpty() || merged.getLast().end < interval.start) {
                merged.add(interval);
            }
            // otherwise, there is overlap, so we merge the current and previous
            // intervals.
            else {
                merged.getLast().end = Math.max(merged.getLast().end, interval.end);
            }
        }

        return merged;
    }
```

### 分别按照 start/end 排序后处理

```java
public List<Interval> merge(List<Interval> intervals) {
	// sort start&end
	int n = intervals.size();
	int[] starts = new int[n];
	int[] ends = new int[n];
	for (int i = 0; i < n; i++) {
		starts[i] = intervals.get(i).start;
		ends[i] = intervals.get(i).end;
	}
	Arrays.sort(starts);
	Arrays.sort(ends);
	// loop through
	List<Interval> res = new ArrayList<Interval>();
	for (int i = 0, j = 0; i < n; i++) { // j is start of interval.
		if (i == n - 1 || starts[i + 1] > ends[i]) {
			res.add(new Interval(starts[j], ends[i]));
			j = i + 1;
		}
	}
	return res;
}
```

算法比较简洁高效，但我难以说出它的正确性。借助这张图，该算法似乎是将所有“交替”/“包含”的情况都处理成交错一种了。这样也就解决了上一种方法中出现的反例情况（上图6️⃣）了。

<div align="center"><img src="{{ site.baseurl }}/images/2018/07/sort-start-end-respectively.png" width="60%"></div>