---
title: "Rotting Oranges (Multi-Source BFS)"
tags: [graph, bfs-dfs-problems, grid, bfs, multi-source-bfs]
videoLink: "https://www.youtube.com/playlist?list=PLgUwDviBIf0oE3gA41TKO2H5bHpPd7fzn"
difficulty: Medium
---

# Rotting Oranges (Multi-Source BFS)

## 🎯 What problem are we solving?

> [!question]
> A grid contains `0` (empty), `1` (fresh orange), and `2` (rotten orange). Every unit of time, **every** rotten orange simultaneously rots its 4-directional fresh-orange neighbors. Find the **minimum time** for all oranges to rot — or `-1` if some fresh orange can never be reached.
>
> This introduces a genuinely new pattern: **multi-source BFS** — starting the traversal from *multiple* nodes at once, all at the same "time."

## 💡 Intuition

> [!tip]
> The key phrase is **"simultaneously."** All rotten oranges spread to their neighbors *in the same unit of time*, in parallel — exactly the same way BFS visits an entire level before moving to the next one.
>
> > If you used DFS instead, you'd go deep down one path first (1 sec, 2 sec, 3 sec...) before ever touching oranges that should have rotted on minute 1 from a completely different rotten orange. DFS doesn't respect "simultaneous spreading" — only BFS's level-by-level nature does.
>
> The twist versus single-source BFS (G-4): instead of starting from *one* node, **every initially-rotten orange is a starting point, all pushed into the queue at once, all at time 0.** From there, it's the same mechanics — just track the time alongside each cell as it gets pushed.

## 🖼️ Visualizing it

```
2 1 1        Time 0: rotten = {(0,0)}
1 1 0
0 1 1
```

```
Time 0 → queue: [(0,0,t=0)]
Pop (0,0): neighbors (0,1) fresh→rot, t=1; (1,0) fresh→rot, t=1
  queue: [(0,1,1), (1,0,1)]

Pop (0,1): neighbors (0,2) fresh→rot, t=2; (1,1) fresh→rot, t=2
  queue: [(1,0,1), (0,2,2), (1,1,2)]

Pop (1,0): neighbors (1,1) already queued/rotten, (2,0) is 0(empty, skip)
  queue: [(0,2,2), (1,1,2)]

Pop (0,2): neighbors (1,2) fresh→rot, t=3
  queue: [(1,1,2), (1,2,3)]

Pop (1,1): neighbors all already rotten/empty → nothing new

Pop (1,2): neighbors (2,2) fresh→rot, t=4
  queue: [(2,2,4)]

Pop (2,2): neighbors (2,1) fresh→rot, t=5
  queue: [(2,1,5)]

Pop (2,1): no new fresh neighbors → done
```

Maximum time seen = **5** → wait, let's double check against a fresh count: every cell got rotten, so the answer is the max time recorded = **5** (track the running max as you pop, rather than computing it some other way).

If, instead, some fresh orange were sitting in a region with no path to any rotten orange (or isolated by `0`s), it would never get pushed into the queue — at the end, scan the grid (or keep a running "fresh remaining" counter) to detect this and return `-1`.

### Two ways to check "did everyone rot?"
1. **Post-scan:** after BFS finishes, loop the grid once more — if any cell is still `1`, return `-1`.
2. **Running counter:** count all fresh oranges up front; decrement every time one gets rotted during BFS; if the counter isn't `0` at the end, return `-1`. (Avoids a second full grid scan.)

## 🛠️ Algorithm / Approach

> [!abstract]
> 1. Scan the grid once: push every `(row, col, time=0)` where `grid[row][col] == 2` into the queue, and mark it visited/rotten. Optionally count fresh oranges (`freshCount`).
> 2. `maxTime = 0`.
> 3. While the queue isn't empty:
>    - Pop `(row, col, time)`. Update `maxTime = max(maxTime, time)`.
>    - For each of the 4 neighbors: if in bounds, **fresh** (`grid[nr][nc] == 1`), and not visited → mark visited/rotten, push `(nr, nc, time + 1)`, decrement `freshCount`.
> 4. If `freshCount != 0` (or a post-scan finds a leftover `1`) → return `-1`.
> 5. Else return `maxTime`.

## 👨‍💻 Code

### Java

```java
import java.util.*;

public class RottingOranges {

    public int orangesRotting(int[][] grid) {
        int n = grid.length, m = grid[0].length;
        Queue<int[]> queue = new LinkedList<>();   // {row, col, time}
        int freshCount = 0;

        for (int i = 0; i < n; i++) {
            for (int j = 0; j < m; j++) {
                if (grid[i][j] == 2) {
                    queue.offer(new int[]{i, j, 0});
                } else if (grid[i][j] == 1) {
                    freshCount++;
                }
            }
        }

        int[] deltaRow = {-1, 0, 1, 0};
        int[] deltaCol = {0, 1, 0, -1};
        int maxTime = 0;

        while (!queue.isEmpty()) {
            int[] cell = queue.poll();
            int row = cell[0], col = cell[1], time = cell[2];
            maxTime = Math.max(maxTime, time);

            for (int i = 0; i < 4; i++) {
                int nr = row + deltaRow[i];
                int nc = col + deltaCol[i];
                boolean inBounds = nr >= 0 && nr < n && nc >= 0 && nc < m;

                if (inBounds && grid[nr][nc] == 1) {
                    grid[nr][nc] = 2;              // mark rotten (acts as visited)
                    freshCount--;
                    queue.offer(new int[]{nr, nc, time + 1});
                }
            }
        }

        return freshCount == 0 ? maxTime : -1;
    }
}
```

### Python

```python
from collections import deque

class Solution:
    def orangesRotting(self, grid: list[list[int]]) -> int:
        n, m = len(grid), len(grid[0])
        queue = deque()
        fresh_count = 0

        for i in range(n):
            for j in range(m):
                if grid[i][j] == 2:
                    queue.append((i, j, 0))
                elif grid[i][j] == 1:
                    fresh_count += 1

        delta_row = [-1, 0, 1, 0]
        delta_col = [0, 1, 0, -1]
        max_time = 0

        while queue:
            row, col, time = queue.popleft()
            max_time = max(max_time, time)

            for i in range(4):
                nr, nc = row + delta_row[i], col + delta_col[i]
                in_bounds = 0 <= nr < n and 0 <= nc < m

                if in_bounds and grid[nr][nc] == 1:
                    grid[nr][nc] = 2          # mark rotten (acts as visited)
                    fresh_count -= 1
                    queue.append((nr, nc, time + 1))

        return max_time if fresh_count == 0 else -1
```

## ⏱️ Complexity Analysis

> [!note]
> - **Time:** O(N × M) — the initial scan is O(N×M), and across the entire BFS, every cell is pushed/popped at most once, with each pop checking 4 neighbors (constant work) → total O(N×M).
> - **Space:** O(N × M) worst case for the queue (if every cell is rotten/fresh).

## ⚠️ Common Pitfalls

> [!warning]
> - **Using DFS instead of BFS.** DFS would still eventually rot every reachable orange, but the *time* values it computes would be wrong — DFS doesn't process all same-distance cells together, so it can't correctly answer a "minimum simultaneous time" question.
> - **Pushing only one rotten orange as the starting point.** This is **multi-source** BFS — every initially-rotten cell must be in the queue from the start, all at `time = 0`, not just the first one found.
> - **Computing the answer as the number of BFS "rounds" via a separate outer loop instead of tracking time per-cell.** Storing `time` directly alongside each queue entry (as a triple) is simpler and less error-prone than trying to process the queue in synchronized batches.
> - **Forgetting the "unreachable fresh orange" case.** Always verify every fresh orange actually got rotted — either with a fresh counter or a final grid scan — before returning the time; otherwise you'll return a time that doesn't actually represent "all oranges rotten."
> - **Mutating `grid` to mark visited without considering whether the caller needs the original.** It's done here for simplicity (`grid[nr][nc] = 2` doubles as marking visited), but as with Flood Fill, copy first if preserving input matters in your context.

## 🔗 Related Problems / Next Up

> [!success]
> - **Builds on:** [Flood Fill](Flood-Fill.md), [G-4 — BFS Traversal](../01-Foundations/G4-BFS-Traversal.md)
> - **New pattern introduced:** Multi-source BFS — starting from many nodes simultaneously, useful whenever multiple sources "spread" at the same rate (01 Matrix, Walls and Gates)
> - [↑ Back to BFS/DFS Problems Index](00-BFS-DFS-Index.md)

