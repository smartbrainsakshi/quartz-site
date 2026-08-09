---
title: "Minimum Multiplications to Reach End"
tags: [graph, shortest-path, bfs, implicit-graph, modular-arithmetic]
videoLink: "https://www.youtube.com/playlist?list=PLgUwDviBIf0oE3gA41TKO2H5bHpPd7fzn"
difficulty: Hard
---

# Minimum Multiplications to Reach End

## 🎯 What problem are we solving?

> [!question]
> Given a `start` value, an `end` value, and an array of allowed multipliers: at each step, multiply the current value by any one array element, then take the result `mod 10^5`, to get the new current value. Find the **minimum number of steps** to transform `start` into `end`. Return `-1` if it's impossible.

## 💡 Intuition

> [!tip]
> Same "implicit graph" trick as Word Ladder: there's no explicit adjacency list, but every reachable value **is** a node, and applying one multiplier **is** an edge of weight `1` to whatever new value results. The insight that makes this tractable at all: because of the `mod 10^5` operation, **every possible value is bounded to the range `[0, 99999]`** — regardless of how large the raw products would otherwise grow, the graph never has more than 100,000 possible nodes.
>
> Since every single multiplication costs exactly `1` step, this is — once again — a **uniform-edge-weight shortest path problem**, solvable with plain BFS, no priority queue needed (the same reasoning as [Word Ladder I](Word-Ladder-I.md) and the unit-weight grid/graph problems).

## 🖼️ Visualizing it

```
start = 3, end = 30, multipliers = [2, 5, 7]
```

```
distance[3] = 0. queue = [3]

pop 3 (steps=0): 3*2=6 (new, dist=1), 3*5=15 (new, dist=1), 3*7=21 (new, dist=1)
  push 6, 15, 21

pop 6 (steps=1): 6*2=12, 6*5=30 -- THIS IS end! return steps+1 = 2 immediately
```

**Result: 2** — `3 → 6 → 30` (multiply by 2, then by 5).

## 🛠️ Algorithm / Approach

> [!abstract]
> 1. Initialize a `distance` array of size `10^5`, all infinity, except `distance[start] = 0`.
> 2. Push `(start, 0)` into a plain FIFO queue.
> 3. While the queue isn't empty: pop `(node, steps)`. For each multiplier `m` in the array: `newNode = (node * m) % 100000`. If `steps + 1 < distance[newNode]`: update `distance[newNode]`; if `newNode == end`, return `steps + 1` immediately; otherwise push `(newNode, steps + 1)`.
> 4. If the queue empties without ever reaching `end`, return `-1`.

## 👨‍💻 Code

### Java

```java
import java.util.*;

public class MinimumMultiplicationsToReachEnd {

    private static final int MOD = 100000;

    public int minimumMultiplications(int[] arr, int start, int end) {
        int[] distance = new int[MOD];
        Arrays.fill(distance, Integer.MAX_VALUE);
        distance[start] = 0;

        Queue<int[]> queue = new LinkedList<>();
        queue.offer(new int[]{start, 0});

        while (!queue.isEmpty()) {
            int[] top = queue.poll();
            int node = top[0], steps = top[1];

            for (int m : arr) {
                int newNode = (int) (((long) node * m) % MOD);

                if (steps + 1 < distance[newNode]) {
                    distance[newNode] = steps + 1;
                    if (newNode == end) return steps + 1;
                    queue.offer(new int[]{newNode, steps + 1});
                }
            }
        }
        return -1;
    }
}
```

### Python

```python
from collections import deque

class Solution:
    def minimum_multiplications(self, arr, start, end):
        MOD = 100000
        distance = [float('inf')] * MOD
        distance[start] = 0

        queue = deque([(start, 0)])

        while queue:
            node, steps = queue.popleft()

            for m in arr:
                new_node = (node * m) % MOD

                if steps + 1 < distance[new_node]:
                    distance[new_node] = steps + 1
                    if new_node == end:
                        return steps + 1
                    queue.append((new_node, steps + 1))

        return -1
```

## ⏱️ Complexity Analysis

> [!note]
> - **Time:** loosely bounded by O(10^5 × len(arr)) in the theoretical worst case where all 100,000 nodes get generated — the video notes this worst case is practically near-impossible given typical constraints on the multiplier array's size, so real performance is far better than this bound suggests.
> - **Space:** O(10^5) for the `distance` array and the queue.

## ⚠️ Common Pitfalls

> [!warning]
> - **Forgetting to apply `mod 10^5` after every multiplication.** Without it, intermediate values can grow arbitrarily large (overflow risk in fixed-width integer types) and the whole algorithm's tractability — bounding the graph to 100,000 nodes — depends entirely on this step.
> - **Using a priority queue instead of a plain queue.** Unnecessary overhead, since every operation costs exactly `1` step (same lesson as the other unit-weight problems in this section).
> - **Checking for the target only after popping it in a later iteration**, instead of the moment it's generated. Not incorrect, but inconsistent placement of this check is a common source of subtle bugs.
> - **Overestimating the real-world time complexity.** The theoretical worst case (generating all 100,000 nodes) is a hypothetical upper bound, not something that realistically occurs given the array size constraints typical of this problem — worth explicitly noting if an interviewer pushes on Big-O.

## 🔗 Related Problems / Next Up

> [!success]
> - **Builds on:** [Word Ladder I](Word-Ladder-I.md) — same "implicit graph + uniform edge weight → plain BFS" pattern
> - **Builds on:** [Shortest Path in an Undirected Graph with Unit Weights](Shortest-Path-Undirected-Unit-Weights.md)
> - [↑ Back to Shortest Path Algorithms Index](00-Shortest-Path-Index.md)
