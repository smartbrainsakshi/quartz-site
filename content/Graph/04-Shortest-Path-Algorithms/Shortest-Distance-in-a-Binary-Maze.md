---
title: "Shortest Distance in a Binary Maze"
tags: [graph, shortest-path, bfs, grid, unweighted-graph]
videoLink: "https://www.youtube.com/playlist?list=PLgUwDviBIf0oE3gA41TKO2H5bHpPd7fzn"
difficulty: Medium
---

# Shortest Distance in a Binary Maze

## 🎯 What problem are we solving?

> [!question]
> Given an `n x m` binary grid (each cell `0` or `1`), a source cell and a destination cell, find the shortest distance between them, moving only through cells valued `1`, in the 4 cardinal directions. If no path exists, return `-1`.

## 💡 Intuition

> [!tip]
> This is shortest-path-shaped (source, destination, minimize distance), which might suggest Dijkstra's — but every single move here costs exactly the same: `1`. That's the exact condition under which a plain **BFS** already computes shortest distances correctly, no priority queue required (the same insight as [Shortest Path in an Undirected Graph with Unit Weights](Shortest-Path-Undirected-Unit-Weights.md), just applied to an implicit grid graph instead of an explicit adjacency list).
>
> Treat each grid cell as a node, with edges to its ≤4 in-bounds, value-`1` neighbors, each edge weighted `1`. Run BFS from the source, maintaining a `distance` grid instead of a plain `visited` grid, relaxing each neighbor exactly as in the general unit-weight BFS pattern.

## 🖼️ Visualizing it

```
Source = (0,1), Destination = (2,2). Grid (1 = passable, 0 = wall):
1 1 0
0 1 0
0 1 1
```

```
distance[(0,1)] = 0. queue = [((0,1))]

pop (0,1) [dist 0]: neighbors (0,0)=1 dist1, (0,2)=0 wall skip, (1,1)=1 dist1
  distance[(0,0)]=1, distance[(1,1)]=1, both pushed

pop (0,0) [dist 1]: neighbors already visited or walls -> nothing new

pop (1,1) [dist 1]: neighbors (0,1) already better, (2,1)=1 dist2, (1,0)=0 wall, (1,2)=0 wall
  distance[(2,1)]=2, pushed

pop (2,1) [dist 2]: neighbors (1,1) already better, (2,0)=0 wall, (2,2)=1 dist3
  distance[(2,2)]=3, pushed  <- this is the destination

queue continues to drain but nothing further improves anything.
```

**Result: 3** — path `(0,1) → (1,1) → (2,1) → (2,2)`.

## 🛠️ Algorithm / Approach

> [!abstract]
> 1. Initialize a `distance` grid the same size as the input, all infinity, except `distance[source] = 0`.
> 2. Push `source` into a plain FIFO queue.
> 3. While the queue isn't empty: pop `(row, col)`. For each of the 4 neighbors: if in bounds, the grid value there is `1`, and `distance[row][col] + 1 < distance[neighbor]`, update the neighbor's distance and push it.
> 4. Return `distance[destination]` if it's not infinity, else `-1`.
> 5. (Defensive edge case: if source equals destination, `distance[source] = 0` is already correct before the loop even starts.)

## 👨‍💻 Code

### Java

```java
import java.util.*;

public class ShortestDistanceBinaryMaze {

    public int shortestPath(int[][] grid, int[] source, int[] destination) {
        int n = grid.length, m = grid[0].length;
        int[][] distance = new int[n][m];
        for (int[] row : distance) Arrays.fill(row, Integer.MAX_VALUE);

        int sr = source[0], sc = source[1];
        distance[sr][sc] = 0;

        Queue<int[]> queue = new LinkedList<>();
        queue.offer(new int[]{sr, sc});

        int[] deltaRow = {-1, 1, 0, 0};
        int[] deltaCol = {0, 0, -1, 1};

        while (!queue.isEmpty()) {
            int[] cell = queue.poll();
            int row = cell[0], col = cell[1];

            for (int i = 0; i < 4; i++) {
                int nr = row + deltaRow[i], nc = col + deltaCol[i];
                boolean inBounds = nr >= 0 && nr < n && nc >= 0 && nc < m;

                if (inBounds && grid[nr][nc] == 1 && distance[row][col] + 1 < distance[nr][nc]) {
                    distance[nr][nc] = distance[row][col] + 1;
                    queue.offer(new int[]{nr, nc});
                }
            }
        }

        int dr = destination[0], dc = destination[1];
        return distance[dr][dc] == Integer.MAX_VALUE ? -1 : distance[dr][dc];
    }
}
```

### Python

```python
from collections import deque

class Solution:
    def shortest_path(self, grid, source, destination):
        n, m = len(grid), len(grid[0])
        distance = [[float('inf')] * m for _ in range(n)]

        sr, sc = source
        distance[sr][sc] = 0
        queue = deque([(sr, sc)])

        delta_row = [-1, 1, 0, 0]
        delta_col = [0, 0, -1, 1]

        while queue:
            row, col = queue.popleft()

            for dr, dc in zip(delta_row, delta_col):
                nr, nc = row + dr, col + dc
                in_bounds = 0 <= nr < n and 0 <= nc < m

                if in_bounds and grid[nr][nc] == 1 and distance[row][col] + 1 < distance[nr][nc]:
                    distance[nr][nc] = distance[row][col] + 1
                    queue.append((nr, nc))

        dr, dc = destination
        return distance[dr][dc] if distance[dr][dc] != float('inf') else -1
```

## ⏱️ Complexity Analysis

> [!note]
> - **Time:** O(N × M) — each cell is processed once, with 4 constant-time neighbor checks.
> - **Space:** O(N × M) for the `distance` grid and the queue.

## ⚠️ Common Pitfalls

> [!warning]
> - **Reaching for a priority queue here.** It wouldn't be wrong, just unnecessarily costs an extra log factor — recognizing the uniform edge weight (every move costs exactly `1`) is what allows dropping straight to plain BFS.
> - **Not checking `grid[nr][nc] == 1` before moving into a cell.** Cells valued `0` are impassable walls and must never be entered, regardless of how short a path through them might otherwise be.
> - **Forgetting bounds checks before indexing the grid**, causing out-of-bounds errors on edge/corner cells.
> - **Using a `visited` boolean instead of a `distance` grid.** Since this problem may be reused as a stepping stone toward weighted grid-Dijkstra variants (see [Path With Minimum Effort](Path-With-Minimum-Effort.md)), building the habit of tracking distance explicitly (with relaxation) pays off even when a plain visited check would technically suffice here.

## 🔗 Related Problems / Next Up

> [!success]
> - **Builds on:** [Shortest Path in an Undirected Graph with Unit Weights](Shortest-Path-Undirected-Unit-Weights.md), grid-BFS patterns from [Flood Fill](../02-BFS-DFS-Problems/Flood-Fill.md) and [Rotting Oranges](../02-BFS-DFS-Problems/Rotting-Oranges.md)
> - **Next up:** [Path With Minimum Effort](Path-With-Minimum-Effort.md) — same grid shape, but the quantity being minimized isn't a uniform per-step cost, requiring Dijkstra's again
> - [↑ Back to Shortest Path Algorithms Index](00-Shortest-Path-Index.md)
