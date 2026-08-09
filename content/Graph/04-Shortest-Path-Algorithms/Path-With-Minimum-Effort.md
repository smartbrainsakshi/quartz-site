---
title: "Path With Minimum Effort"
tags: [graph, shortest-path, dijkstra, grid, minimax]
videoLink: "https://www.youtube.com/playlist?list=PLgUwDviBIf0oE3gA41TKO2H5bHpPd7fzn"
difficulty: Hard
---

# Path With Minimum Effort

## 🎯 What problem are we solving?

> [!question]
> Given an `n x m` grid of heights, travel from the top-left cell to the bottom-right cell (4-directional moves). A route's **effort** is the **maximum** absolute height difference between any two *consecutive* cells along that route (not a sum). Find the route with the minimum possible effort.

## 💡 Intuition

> [!tip]
> This is still "shortest path"-shaped — minimize some end-to-end quantity between a source and destination — so Dijkstra's greedy structure still applies. What changes is what's being minimized: instead of a **sum** of edge weights, it's a **running maximum** of per-step differences.
>
> Redefine the "distance" recorded for a cell as *"the minimum possible effort needed to reach this cell."* Redefine relaxation accordingly: taking a step doesn't *add* to the effort so far — it only *raises* the effort if that step's height difference exceeds everything seen on the path up to that point:
> ```
> candidateEffort = max(currentEffort, abs(height[current] - height[neighbor]))
> ```
> Update the neighbor only if `candidateEffort < distance[neighbor]`. Because a min-heap still always expands the globally smallest known effort next, the moment the **destination is popped** from the priority queue, its associated effort is guaranteed to be the true minimum — return immediately, no need to drain the rest of the heap.

## 🖼️ Visualizing it

```
A route with heights 1,3,3,5 (diffs: |1-3|=2, |3-3|=0, |3-5|=2) has effort = max(2,0,2) = 2
A different route with heights 1,3,8,2,5 (diffs: 2,5,6,3) has effort = max(2,5,6,3) = 6

The first route is better -- lower max-diff, even though its individual steps aren't all smaller.
```

```
Dijkstra with effort-as-distance:
pop (0, source): relax each neighbor with candidateEffort = max(0, |height diff|)
  push each neighbor with its own individual step diff as its effort so far

pop (smallest pending effort, cell): relax onward, taking max(currentEffort, next diff)
  never adding -- only possibly raising the bar

... continues until destination is popped ...

pop (2, destination): return 2 immediately -- no need to check the rest of the heap
```

## 🛠️ Algorithm / Approach

> [!abstract]
> 1. Initialize an `effort` grid to infinity, except `effort[source] = 0`.
> 2. Push `(0, sourceRow, sourceCol)` into a min-heap.
> 3. While the heap isn't empty: pop `(currentEffort, row, col)`. For each of the 4 in-bounds neighbors: `candidateEffort = max(currentEffort, abs(height[row][col] - height[nr][nc]))`. If `candidateEffort < effort[nr][nc]`, update and push.
> 4. The instant `(row, col)` popped equals the destination, **return `currentEffort` immediately** — it's guaranteed minimal.

## 👨‍💻 Code

### Java

```java
import java.util.*;

public class PathWithMinimumEffort {

    public int minimumEffortPath(int[][] heights) {
        int n = heights.length, m = heights[0].length;
        int[][] effort = new int[n][m];
        for (int[] row : effort) Arrays.fill(row, Integer.MAX_VALUE);
        effort[0][0] = 0;

        PriorityQueue<int[]> pq = new PriorityQueue<>((a, b) -> a[0] - b[0]);
        pq.offer(new int[]{0, 0, 0});

        int[] deltaRow = {-1, 1, 0, 0};
        int[] deltaCol = {0, 0, -1, 1};

        while (!pq.isEmpty()) {
            int[] top = pq.poll();
            int currentEffort = top[0], row = top[1], col = top[2];

            if (row == n - 1 && col == m - 1) return currentEffort;   // destination reached

            for (int i = 0; i < 4; i++) {
                int nr = row + deltaRow[i], nc = col + deltaCol[i];
                boolean inBounds = nr >= 0 && nr < n && nc >= 0 && nc < m;
                if (!inBounds) continue;

                int diff = Math.abs(heights[row][col] - heights[nr][nc]);
                int candidateEffort = Math.max(currentEffort, diff);

                if (candidateEffort < effort[nr][nc]) {
                    effort[nr][nc] = candidateEffort;
                    pq.offer(new int[]{candidateEffort, nr, nc});
                }
            }
        }
        return 0;   // unreachable in practice: grid is always fully connected
    }
}
```

### Python

```python
import heapq

class Solution:
    def minimum_effort_path(self, heights):
        n, m = len(heights), len(heights[0])
        effort = [[float('inf')] * m for _ in range(n)]
        effort[0][0] = 0

        pq = [(0, 0, 0)]
        delta_row = [-1, 1, 0, 0]
        delta_col = [0, 0, -1, 1]

        while pq:
            current_effort, row, col = heapq.heappop(pq)

            if row == n - 1 and col == m - 1:
                return current_effort   # destination reached

            for dr, dc in zip(delta_row, delta_col):
                nr, nc = row + dr, col + dc
                if not (0 <= nr < n and 0 <= nc < m):
                    continue

                diff = abs(heights[row][col] - heights[nr][nc])
                candidate_effort = max(current_effort, diff)

                if candidate_effort < effort[nr][nc]:
                    effort[nr][nc] = candidate_effort
                    heapq.heappush(pq, (candidate_effort, nr, nc))

        return 0   # unreachable in practice: grid is always fully connected
```

## ⏱️ Complexity Analysis

> [!note]
> - **Time:** O(N × M × log(N × M)) — Dijkstra's E log V with `E = 4 × N × M` and `V = N × M`.
> - **Space:** O(N × M) for the `effort` grid and the heap.

## ⚠️ Common Pitfalls

> [!warning]
> - **Using `distance[node] + weight` (sum) instead of `max(currentEffort, diff)`.** This is the single most important adaptation for this problem — treating it like standard additive Dijkstra produces completely wrong answers, since the quantity being minimized is a bottleneck (max), not a total.
> - **Not returning immediately once the destination is popped.** While the correct minimal effort would eventually sit in `effort[destination]` even without an early return, stopping the moment it's popped avoids unnecessary further work and reflects the same greedy-finality guarantee Dijkstra's relies on generally.
> - **Assuming plain BFS/queue suffices here**, unlike [Shortest Distance in a Binary Maze](Shortest-Distance-in-a-Binary-Maze.md). Since each step's "cost" (the height difference) varies, a priority queue exploring smallest-effort-first is required — exactly the discriminator from [Why Priority Queue & Time Complexity](Dijkstra-Why-Priority-Queue-and-Time-Complexity.md).
> - **Bounds-check errors** when generating the four neighbor offsets — the same class of mistake as any grid-traversal problem.

## 🔗 Related Problems / Next Up

> [!success]
> - **Builds on:** [Shortest Distance in a Binary Maze](Shortest-Distance-in-a-Binary-Maze.md), [Dijkstra's Algorithm (Using a Priority Queue)](Dijkstra-Algorithm-Using-Priority-Queue.md)
> - **Contrast:** minimax/bottleneck shortest path vs. standard additive shortest path — same greedy heap machinery, different relaxation formula
> - [↑ Back to Shortest Path Algorithms Index](00-Shortest-Path-Index.md)
