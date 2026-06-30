---
title: "Distance of Nearest Cell Having 1 (01 Matrix)"
tags: [graph, bfs-dfs-problems, grid, bfs, multi-source-bfs]
videoLink: "https://www.youtube.com/playlist?list=PLgUwDviBIf0oE3gA41TKO2H5bHpPd7fzn"
difficulty: Medium
---

# Distance of Nearest Cell Having 1 (01 Matrix)

## 🎯 What problem are we solving?

> [!question]
> Given a binary grid of `0`s and `1`s, for **every** cell, find the distance to the nearest `1` (where distance = `|row1 - row2| + |col1 - col2|`, i.e. only up/down/left/right moves count — no diagonals). A `1` cell's distance to its nearest `1` is trivially `0` (itself).
>
> This is the **same multi-source BFS pattern as Rotting Oranges** — just computing a distance value for every cell instead of a single "time until all rotten" answer. (Note: LeetCode's "01 Matrix" is the mirror version — distance to nearest `0` — but the technique is identical either way.)

## 💡 Intuition

> [!tip]
> Why BFS and not DFS? Because **multiple `1`s can be the "nearest" source for different cells**, and we need everyone to expand outward *at the same rate* to guarantee the *first* time a cell is reached really is its *minimum* distance.
>
> > If all the `1`s simultaneously take one step outward, every `0` they touch in that step is **guaranteed** to be at distance exactly 1 from its nearest `1`. Take another simultaneous step from those newly-touched cells, and everything newly reached is at distance exactly 2. This only works because BFS processes everything at the current distance before moving further — DFS would dive deep along one path and report wrong (too large) distances for cells that actually had a much closer `1` via a different source.
>
> This is **multi-source BFS**, exactly like Rotting Oranges: push every `1` into the queue at once, all at distance `0`, then expand level by level, recording the distance the first time each `0` is reached.

## 🖼️ Visualizing it

```
1 0 1
1 1 0
0 0 0
```

All `1`s — `(0,0), (0,2), (1,0), (1,1)` — start in the queue at distance `0`.

```
Step 1 (distance 1): from the four 1's, expand outward —
  (0,0) → (0,1) dist=1
  (0,2) → (0,1) already queued/visited from above, (1,2) dist=1
  (1,0) → (2,0) dist=1
  (1,1) → (1,2) already touched, (2,1) dist=1

Step 2 (distance 2): from the newly-touched cells —
  (0,1) → no new cells (neighbors already visited)
  (1,2) → no new cells
  (2,0) → (2,1) already visited
  (2,1) → (2,2) dist=2
```

**Final distance matrix:**
```
0 1 0
0 0 1
1 1 2
```

Every `0` was reached on its very first BFS "touch" — and because all sources expand in lockstep, that first touch is guaranteed to be the shortest possible distance.

## 🛠️ Algorithm / Approach

> [!abstract]
> 1. Create a `visited` grid and a `distance` grid (both same size as input), all initialized appropriately — `distance` can start at `0` everywhere since it'll be overwritten as cells are reached.
> 2. Scan the grid once: for every cell where `grid[i][j] == 1`, push `(i, j, step=0)` into the queue and mark visited. (This is the **multi-source** initialization — all `1`s start together.)
> 3. While the queue isn't empty:
>    - Pop `(row, col, step)`. Record `distance[row][col] = step`.
>    - For each of the 4 neighbors: if in bounds and not visited → mark visited, push `(nr, nc, step + 1)`.
> 4. Return the `distance` grid.

## 👨‍💻 Code

### Java

```java
import java.util.*;

public class NearestCellDistance {

    public int[][] nearestOne(int[][] grid) {
        int n = grid.length, m = grid[0].length;
        boolean[][] visited = new boolean[n][m];
        int[][] distance = new int[n][m];
        Queue<int[]> queue = new LinkedList<>();   // {row, col, step}

        // multi-source initialization: every 1 starts the BFS at step 0
        for (int i = 0; i < n; i++) {
            for (int j = 0; j < m; j++) {
                if (grid[i][j] == 1) {
                    queue.offer(new int[]{i, j, 0});
                    visited[i][j] = true;
                }
            }
        }

        int[] deltaRow = {-1, 0, 1, 0};
        int[] deltaCol = {0, 1, 0, -1};

        while (!queue.isEmpty()) {
            int[] cell = queue.poll();
            int row = cell[0], col = cell[1], step = cell[2];
            distance[row][col] = step;

            for (int i = 0; i < 4; i++) {
                int nr = row + deltaRow[i];
                int nc = col + deltaCol[i];
                boolean inBounds = nr >= 0 && nr < n && nc >= 0 && nc < m;

                if (inBounds && !visited[nr][nc]) {
                    visited[nr][nc] = true;
                    queue.offer(new int[]{nr, nc, step + 1});
                }
            }
        }
        return distance;
    }
}
```

### Python

```python
from collections import deque

class Solution:
    def nearest_one(self, grid: list[list[int]]) -> list[list[int]]:
        n, m = len(grid), len(grid[0])
        visited = [[False] * m for _ in range(n)]
        distance = [[0] * m for _ in range(n)]
        queue = deque()

        # multi-source initialization: every 1 starts the BFS at step 0
        for i in range(n):
            for j in range(m):
                if grid[i][j] == 1:
                    queue.append((i, j, 0))
                    visited[i][j] = True

        delta_row = [-1, 0, 1, 0]
        delta_col = [0, 1, 0, -1]

        while queue:
            row, col, step = queue.popleft()
            distance[row][col] = step

            for i in range(4):
                nr, nc = row + delta_row[i], col + delta_col[i]
                in_bounds = 0 <= nr < n and 0 <= nc < m

                if in_bounds and not visited[nr][nc]:
                    visited[nr][nc] = True
                    queue.append((nr, nc, step + 1))

        return distance
```

## ⏱️ Complexity Analysis

> [!note]
> - **Time:** O(N × M) — the initial scan is O(N×M), and the BFS itself visits each cell exactly once with 4 constant-time neighbor checks per cell.
> - **Space:** O(N × M) for `visited` and `distance`, plus O(N × M) worst case for the queue.

## ⚠️ Common Pitfalls

> [!warning]
> - **Using single-source BFS or running BFS separately from each `1` and taking the minimum.** This technically gives the correct answer but is far slower (O(N×M) sources × O(N×M) BFS each = O((N×M)²)) — always push **all** sources into the queue together at the start, exactly once.
> - **Marking visited only by checking `distance != 0`** instead of a proper visited array — `0` is a legitimate computed distance for some cells, so you can't distinguish "not yet visited" from "distance happens to be 0" this way. Always use a dedicated visited structure.
> - **Computing Manhattan distance directly (`|r1-r2| + |c1-c2|`) for every `(0, nearest 1)` pair via brute force.** This works but is O((N×M)²) in the worst case — multi-source BFS achieves the same result in O(N×M).
> - **Forgetting cells that are themselves `1` should report distance `0`.** This falls out naturally from the algorithm above (they're popped at `step = 0`), but it's worth explicitly verifying when testing your code.

## 🔗 Related Problems / Next Up

> [!success]
> - **Builds on:** [Rotting Oranges (Multi-Source BFS)](Rotting-Oranges.md) — nearly identical pattern, computing per-cell distance instead of a single global time
> - **Variant:** LeetCode 542 "01 Matrix" — same technique, just measuring distance to nearest `0` instead of nearest `1`
> - [↑ Back to BFS/DFS Problems Index](00-BFS-DFS-Index.md)

