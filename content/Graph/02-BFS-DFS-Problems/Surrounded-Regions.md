---
title: "Surrounded Regions"
tags: [graph, bfs-dfs-problems, grid, dfs, boundary-traversal]
videoLink: "https://www.youtube.com/playlist?list=PLgUwDviBIf0oE3gA41TKO2H5bHpPd7fzn"
difficulty: Medium
---

# Surrounded Regions

## 🎯 What problem are we solving?

> [!question]
> Given an `n x m` matrix where every cell is `'X'` or `'O'`, replace every `'O'` that is **surrounded** by `'X'` with an `'X'`. An `'O'` (or a connected group of `O`s) counts as surrounded only if it has **no path to the border** of the matrix through other `O`s — moving up/down/left/right only, no diagonals. `O`s that do reach the border, directly or through a chain of connected `O`s, must stay as `O`.

## 💡 Intuition

> [!tip]
> Trying to directly check "is this `O` surrounded on all four sides" doesn't scale — a whole connected blob of `O`s can wander around and still be fully enclosed, or one of them can sneak out to the edge and save the entire group.
>
> The trick is to **flip the question**: instead of hunting for surrounded `O`s, find the `O`s that can **never** be captured — the ones connected (directly or transitively) to a boundary `O`. Any `O` reachable from the border by a chain of `O`s is safe forever, no matter how far inside the matrix that chain wanders. Everything else — every `O` untouched by that traversal — is guaranteed to be surrounded, and gets converted to `X`.
>
> So the algorithm becomes: start a DFS/BFS from every boundary cell that is `O`, mark everything it touches as "safe," then sweep the whole grid and flip every unmarked `O` to `X`.

## 🖼️ Visualizing it

```
X X X X
X O O X
X X O X
X O X X
```

Boundary scan finds one `O` on the border: row 3, column 1 (`X O X X` row).

```
DFS from (3,1):
  mark (3,1) visited
  neighbors: (2,1)=X skip, (3,0)=X skip, (3,2)=X skip — no up? check (2,1) is X
  actually (3,1)'s up-neighbor is (2,1) which is X, so DFS stops here
  → only (3,1) marked safe
```

The blob `(1,1), (1,2), (2,2)` is never touched by any boundary DFS — it's fully enclosed — so it gets converted:

```
X X X X
X X X X      <- (1,1),(1,2) flipped to X
X X X X      <- (2,2) flipped to X
X O X X      <- (3,1) stays O (reached from boundary)
```

Every `O` that survives is one the boundary traversal actually walked over — everything else was, by definition, unreachable from the outside.

## 🛠️ Algorithm / Approach

> [!abstract]
> 1. Create a `visited` grid the same size as the input, all `false`.
> 2. Walk the **first row** and **last row**: for every cell that is `'O'` and unvisited, run DFS (or BFS) marking it and every `O` connected to it as visited.
> 3. Walk the **first column** and **last column**: same thing — for every unvisited `'O'`, run DFS marking its whole connected component.
> 4. After all four borders have been processed, scan the entire matrix: if a cell is `'O'` and still **not visited**, flip it to `'X'`. If it's `'O'` and visited, leave it as `'O'`.
> 5. Return the (in-place modified) matrix.

## 👨‍💻 Code

### Java

```java
public class SurroundedRegions {

    private int[] deltaRow = {-1, 0, 1, 0};
    private int[] deltaCol = {0, 1, 0, -1};

    public void solve(char[][] matrix) {
        int n = matrix.length, m = matrix[0].length;
        boolean[][] visited = new boolean[n][m];

        // traverse first & last row
        for (int j = 0; j < m; j++) {
            if (!visited[0][j] && matrix[0][j] == 'O') dfs(0, j, visited, matrix);
            if (!visited[n - 1][j] && matrix[n - 1][j] == 'O') dfs(n - 1, j, visited, matrix);
        }

        // traverse first & last column
        for (int i = 0; i < n; i++) {
            if (!visited[i][0] && matrix[i][0] == 'O') dfs(i, 0, visited, matrix);
            if (!visited[i][m - 1] && matrix[i][m - 1] == 'O') dfs(i, m - 1, visited, matrix);
        }

        // anything not reachable from the boundary gets captured
        for (int i = 0; i < n; i++) {
            for (int j = 0; j < m; j++) {
                if (!visited[i][j] && matrix[i][j] == 'O') {
                    matrix[i][j] = 'X';
                }
            }
        }
    }

    private void dfs(int row, int col, boolean[][] visited, char[][] matrix) {
        visited[row][col] = true;
        int n = matrix.length, m = matrix[0].length;

        for (int i = 0; i < 4; i++) {
            int nRow = row + deltaRow[i];
            int nCol = col + deltaCol[i];

            boolean inBounds = nRow >= 0 && nRow < n && nCol >= 0 && nCol < m;
            if (inBounds && !visited[nRow][nCol] && matrix[nRow][nCol] == 'O') {
                dfs(nRow, nCol, visited, matrix);
            }
        }
    }
}
```

### Python

```python
class Solution:
    def solve(self, matrix: list[list[str]]) -> None:
        n, m = len(matrix), len(matrix[0])
        visited = [[False] * m for _ in range(n)]
        delta_row = [-1, 0, 1, 0]
        delta_col = [0, 1, 0, -1]

        def dfs(row: int, col: int) -> None:
            visited[row][col] = True
            for dr, dc in zip(delta_row, delta_col):
                nr, nc = row + dr, col + dc
                in_bounds = 0 <= nr < n and 0 <= nc < m
                if in_bounds and not visited[nr][nc] and matrix[nr][nc] == 'O':
                    dfs(nr, nc)

        # traverse first & last row
        for j in range(m):
            if not visited[0][j] and matrix[0][j] == 'O':
                dfs(0, j)
            if not visited[n - 1][j] and matrix[n - 1][j] == 'O':
                dfs(n - 1, j)

        # traverse first & last column
        for i in range(n):
            if not visited[i][0] and matrix[i][0] == 'O':
                dfs(i, 0)
            if not visited[i][m - 1] and matrix[i][m - 1] == 'O':
                dfs(i, m - 1)

        # anything not reachable from the boundary gets captured
        for i in range(n):
            for j in range(m):
                if not visited[i][j] and matrix[i][j] == 'O':
                    matrix[i][j] = 'X'
```

## ⏱️ Complexity Analysis

> [!note]
> - **Time:** O(N × M) — the four boundary scans are O(N + M) total, and the DFS across all boundary-connected components visits each cell at most once with 4 constant-time neighbor checks, so it's bounded by O(N × M).
> - **Space:** O(N × M) for the `visited` grid, plus O(N × M) worst-case recursion stack depth if nearly the whole grid is one connected `O` component.

## ⚠️ Common Pitfalls

> [!warning]
> - **Only checking the first row/column and forgetting the last row/column.** All four borders can have `O`s that need to seed a traversal — missing any one of them silently mis-captures cells that should have stayed `O`.
> - **Starting DFS only from the first boundary `O` found and stopping.** There can be multiple disconnected boundary components — every unvisited boundary `O` needs its own DFS call.
> - **Flipping cells to `X` *during* the boundary DFS** instead of using a separate `visited` array. If you mutate the matrix while still trying to detect "is this an `O`" for the traversal, you break your own connectivity checks.
> - **Considering diagonal neighbors as connected.** Only up/down/left/right count — a diagonal `O` does not save a surrounded region.

## 🔗 Related Problems / Next Up

> [!success]
> - **Builds on:** [Number of Islands (Grid Connected Components)](Number-of-Islands.md) and [Flood Fill](Flood-Fill.md) — same grid-DFS mechanics, but inverted: mark what *can't* be converted instead of what should be.
> - **Next up:** Number of Enclaves (DFS/BFS) — near-identical boundary-DFS pattern, counting untouched land cells instead of flipping a matrix.
> - [↑ Back to BFS/DFS Problems Index](00-BFS-DFS-Index.md)
