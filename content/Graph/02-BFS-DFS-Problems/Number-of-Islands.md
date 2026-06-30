---
title: "Number of Islands (Connected Components in a Grid)"
tags: [graph, bfs-dfs-problems, connected-components, grid, bfs, dfs]
videoLink: "https://www.youtube.com/playlist?list=PLgUwDviBIf0oE3gA41TKO2H5bHpPd7fzn"
difficulty: Medium
---

# Number of Islands (Connected Components in a Grid)

## 🎯 What problem are we solving?

> [!question]
> Given an `n × m` grid of `0`s (water) and `1`s (land), find the number of **islands** — where an island is a group of `1`s connected horizontally, vertically, **or diagonally** (all 8 directions). This is the exact same "count connected components" idea as Number of Provinces — just disguised as a 2D grid instead of an explicit graph.

## 💡 Intuition

> [!tip]
> The trick is realizing **a grid *is* a graph** — you just haven't drawn it that way yet:
>
> > Every cell `(row, col)` is a node. Two cells are "neighbors" (connected by an edge) if they're adjacent in any of the 8 directions **and both are land**.
>
> Once you see it that way, this becomes the *exact* same algorithm as Number of Provinces:
>
> 1. Loop over every cell in the grid.
> 2. If a cell is land **and unvisited**, that means you've found a brand-new island — increment your counter and run BFS/DFS from it to mark the *entire* connected blob of land as visited.
> 3. Skip any cell that's water or already visited.
>
> The only genuinely new piece is: **since nodes are now `(row, col)` pairs instead of single integers, "visited" and "the queue" both need to store pairs, not single numbers.**

## 🖼️ Visualizing it

```
1 1 0 0 0
0 1 0 1 0
0 0 1 1 0
0 0 0 0 0
1 0 0 1 1
```

Tracing the scan order (row 0 → row n-1, left to right):
- `(0,0)` is land, unvisited → **island #1** → BFS visits `(0,0),(0,1),(1,1)` (and via diagonal, `(0,2)→no it's water; (1,2)→water; (1,0)? water`). Then from `(1,1)` diagonally we reach `(2,2)` and `(1,3)`? Let's just say BFS expands through every 8-directionally-adjacent land cell reachable from `(0,0)` — ending up covering `{(0,0),(0,1),(1,1),(2,2),(2,3),(1,3)}` since diagonal connectivity bridges them all into one blob.
- Continue scanning... `(4,0)` is land, unvisited → **island #2** → BFS visits just `{(4,0)}` (no land neighbors in any of 8 directions).
- `(4,3)` is land, unvisited → **island #3** → BFS visits `{(4,3),(4,4)}`.

**Answer: 3 islands.**

### Finding the 8 neighbors without writing 8 separate lines
Instead of manually listing all 8 neighbor offsets, use a **delta loop**:

```
for delta_row in [-1, 0, 1]:
    for delta_col in [-1, 0, 1]:
        neighbor_row = row + delta_row
        neighbor_col = col + delta_col
        # (delta_row=0, delta_col=0) is the cell itself — already visited, naturally skipped
```

This generates all 9 combinations of `(-1,0,1) × (-1,0,1)` — 8 real neighbors plus the cell itself (which gets skipped automatically since it's already marked visited).

## 🛠️ Algorithm / Approach

> [!abstract]
> 1. Get `n = grid.length`, `m = grid[0].length`. Create a 2D `visited` array of the same size, all `false`.
> 2. Loop `row` from `0` to `n-1`, `col` from `0` to `m-1`:
>    - If `grid[row][col] == 1` and `!visited[row][col]`: increment island count, run BFS/DFS from `(row, col)`.
> 3. **BFS/DFS from `(row, col)`:**
>    - Mark `(row, col)` visited, push it into the queue (BFS) or recurse (DFS).
>    - For each of the 8 deltas: compute `(nr, nc)`. If `nr, nc` are within grid bounds, `grid[nr][nc] == 1`, and not visited → mark visited, push/recurse.
> 4. Return the island count.

## 👨‍💻 Code

### Java

```java
import java.util.*;

public class NumberOfIslands {

    public int numIslands(int[][] grid) {
        int n = grid.length, m = grid[0].length;
        boolean[][] visited = new boolean[n][m];
        int count = 0;

        for (int row = 0; row < n; row++) {
            for (int col = 0; col < m; col++) {
                if (grid[row][col] == 1 && !visited[row][col]) {
                    bfs(row, col, grid, visited, n, m);
                    count++;
                }
            }
        }
        return count;
    }

    private void bfs(int row, int col, int[][] grid, boolean[][] visited, int n, int m) {
        Queue<int[]> queue = new LinkedList<>();
        queue.offer(new int[]{row, col});
        visited[row][col] = true;

        while (!queue.isEmpty()) {
            int[] cell = queue.poll();
            int r = cell[0], c = cell[1];

            for (int dr = -1; dr <= 1; dr++) {
                for (int dc = -1; dc <= 1; dc++) {
                    int nr = r + dr, nc = c + dc;
                    boolean inBounds = nr >= 0 && nr < n && nc >= 0 && nc < m;

                    if (inBounds && grid[nr][nc] == 1 && !visited[nr][nc]) {
                        visited[nr][nc] = true;
                        queue.offer(new int[]{nr, nc});
                    }
                }
            }
        }
    }
}
```

### Python

```python
from collections import deque

class Solution:
    def numIslands(self, grid: list[list[int]]) -> int:
        n, m = len(grid), len(grid[0])
        visited = [[False] * m for _ in range(n)]
        count = 0

        def bfs(start_row, start_col):
            queue = deque([(start_row, start_col)])
            visited[start_row][start_col] = True

            while queue:
                r, c = queue.popleft()
                for dr in (-1, 0, 1):
                    for dc in (-1, 0, 1):
                        nr, nc = r + dr, c + dc
                        in_bounds = 0 <= nr < n and 0 <= nc < m
                        if in_bounds and grid[nr][nc] == 1 and not visited[nr][nc]:
                            visited[nr][nc] = True
                            queue.append((nr, nc))

        for row in range(n):
            for col in range(m):
                if grid[row][col] == 1 and not visited[row][col]:
                    bfs(row, col)
                    count += 1

        return count
```

## ⏱️ Complexity Analysis

> [!note]
> - **Time:** O(N × M × 9) ≈ **O(N × M)** — the outer double loop scans every cell once (N×M), and every cell that *does* trigger a BFS only ever gets processed once overall (same "partial traversals sum to one full traversal" argument as Number of Provinces); each cell's processing checks 9 deltas (8 neighbors + itself) — a constant factor, so it doesn't change the asymptotic complexity.
> - **Space:** O(N × M) for the `visited` grid, plus O(N × M) worst-case for the BFS queue (if the entire grid is one giant island).

## ⚠️ Common Pitfalls

> [!warning]
> - **Forgetting `(row, col)` is now a pair**, not a single integer — both the `visited` structure and the queue need to store/handle pairs, not plain integers like in earlier 1D graph problems.
> - **Skipping the bounds check before the land/visited checks** — accessing `grid[nr][nc]` when `nr` or `nc` is out of range throws an index error. Always check bounds *first* (short-circuit evaluation), then land, then visited.
> - **Using only 4-directional connectivity when the problem asks for 8.** This specific problem explicitly allows diagonal connections — many "Number of Islands" variants (e.g. the classic LeetCode 200) only use 4 directions (up/down/left/right). **Always check the problem statement** before reusing this template; using the wrong direction count silently gives wrong island counts.
> - **Including the `(delta_row=0, delta_col=0)` case as a "neighbor."** It isn't a neighbor — it's the cell itself. It's harmless here only because the cell is already marked visited by the time the delta loop runs, but it's worth understanding why it doesn't cause a bug rather than assuming it "just works."

## 🔗 Related Problems / Next Up

> [!success]
> - **Builds on:** [Number of Provinces](Number-of-Provinces.md) — same pattern, grid instead of explicit graph
> - **Pattern reused in:** Flood Fill, Rotting Oranges, Max Area of Island, 01 Matrix
> - [↑ Back to BFS/DFS Problems Index](00-BFS-DFS-Index.md)

