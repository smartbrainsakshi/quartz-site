---
title: "Flood Fill"
tags: [graph, bfs-dfs-problems, grid, dfs, bfs]
videoLink: "https://www.youtube.com/playlist?list=PLgUwDviBIf0oE3gA41TKO2H5bHpPd7fzn"
difficulty: Easy
---

# Flood Fill

## 🎯 What problem are we solving?

> [!question]
> You're given a 2D grid of pixel values, a starting pixel `(sr, sc)`, and a `newColor`. Starting from that pixel, recolor it **and every pixel reachable from it through 4-directional neighbors that share the original starting color** — exactly like the "paint bucket" tool in MS Paint. This is the same connected-component-marking idea as Number of Islands, but now we're *writing* a new value instead of just counting.

## 💡 Intuition

> [!tip]
> The condition for "two pixels are connected" is stricter than before: a neighbor only joins the flood-fill region if it's 4-directionally adjacent **and has the exact same color as the starting pixel's original color**. The moment a neighbor has a different color, the fill stops spreading in that direction — it acts as a wall.
>
> This makes Flood Fill a textbook DFS (or BFS) problem:
> 1. Remember the **starting/original color** before you change anything.
> 2. Recolor the current pixel to `newColor`.
> 3. Recurse/queue into each of the 4 neighbors — but only visit a neighbor if it's in bounds, **still has the original color** (not yet recolored), and hasn't been visited.
>
> > **Why check "still has the original color" instead of a separate visited array?** Once a pixel is recolored to `newColor`, it naturally no longer matches the original color — so the color-check *doubles* as the visited check, as long as `newColor != originalColor`. (If they happen to be equal, you'd infinite-loop without an explicit visited check — see Pitfalls.)

## 🖼️ Visualizing it

```
Grid:                  Start: (2, 0), newColor = 3

1 1 1
1 1 0
1 1 1
```

Starting pixel `(2,0)` has color `1` (the **original color**). DFS from `(2,0)`:

```
(2,0)=1 → recolor to 3 → neighbors: up(1,0)=1 ✓, right(2,1)=1 ✓
  (1,0)=1 → recolor to 3 → neighbors: up(0,0)=1 ✓, down(2,0)=3 (already new color, skip), right(1,1)=1 ✓
    (0,0)=1 → recolor to 3 → neighbors: down(1,0)=3(skip), right(0,1)=1 ✓
      (0,1)=1 → recolor to 3 → neighbors: left(0,0)=3(skip), right(0,2)=1 ✓, down(1,1)=1(will visit/has been queued)✓
        (0,2)=1 → recolor to 3 → neighbors: left(0,1)=3(skip), down(1,2)=0 ✗(different color, wall) → dead end, backtrack
    (1,1)=1 → recolor to 3 → neighbors: up(0,1)=3(skip), down(2,1)=1 ✓, left(1,0)=3(skip), right(1,2)=0 ✗(wall)
      (2,1)=1 → recolor to 3 → neighbors: left(2,0)=3(skip), right(2,2)=1 ✓, up(1,1)=3(skip)
        (2,2)=1 → recolor to 3 → neighbors: up(1,2)=0 ✗(wall), left(2,1)=3(skip)
  (2,1) already handled above
```

Final grid:
```
3 3 3
3 3 0
3 3 3
```
The single `0` acts as a wall — it's never touched because it never matched the original color `1`.

## 🛠️ Algorithm / Approach

> [!abstract]
> 1. Record `originalColor = image[sr][sc]`.
> 2. **Edge case:** if `originalColor == newColor`, do nothing and return immediately (otherwise you'd infinite-loop — see Pitfalls).
> 3. Make a copy of the image if you don't want to mutate the input (common good practice — don't tamper with given data unless explicitly told to).
> 4. DFS/BFS from `(sr, sc)`:
>    - Recolor the current cell to `newColor`.
>    - For each of the 4 directions: if in bounds **and** `image[nr][nc] == originalColor` → recurse/enqueue.
> 5. Return the modified image.

## 👨‍💻 Code

### Java

```java
import java.util.*;

public class FloodFill {

    public int[][] floodFill(int[][] image, int sr, int sc, int newColor) {
        int originalColor = image[sr][sc];
        if (originalColor == newColor) return image;   // nothing to do, avoid infinite recursion

        int[][] answer = new int[image.length][];
        for (int i = 0; i < image.length; i++) answer[i] = image[i].clone();   // don't mutate input

        dfs(sr, sc, answer, originalColor, newColor);
        return answer;
    }

    private void dfs(int row, int col, int[][] image, int originalColor, int newColor) {
        image[row][col] = newColor;

        int[] deltaRow = {-1, 0, 1, 0};
        int[] deltaCol = {0, 1, 0, -1};

        for (int i = 0; i < 4; i++) {
            int nr = row + deltaRow[i];
            int nc = col + deltaCol[i];
            boolean inBounds = nr >= 0 && nr < image.length && nc >= 0 && nc < image[0].length;

            if (inBounds && image[nr][nc] == originalColor) {
                dfs(nr, nc, image, originalColor, newColor);
            }
        }
    }
}
```

### Python

```python
class Solution:
    def floodFill(self, image: list[list[int]], sr: int, sc: int, newColor: int) -> list[list[int]]:
        original_color = image[sr][sc]
        if original_color == newColor:
            return image   # nothing to do, avoid infinite recursion

        n, m = len(image), len(image[0])
        answer = [row[:] for row in image]   # don't mutate input

        delta_row = [-1, 0, 1, 0]
        delta_col = [0, 1, 0, -1]

        def dfs(row, col):
            answer[row][col] = newColor
            for i in range(4):
                nr, nc = row + delta_row[i], col + delta_col[i]
                in_bounds = 0 <= nr < n and 0 <= nc < m
                if in_bounds and answer[nr][nc] == original_color:
                    dfs(nr, nc)

        dfs(sr, sc)
        return answer
```

## ⏱️ Complexity Analysis

> [!note]
> - **Time:** O(N × M) — in the worst case (every pixel shares the same original color), every cell gets visited exactly once, and each visit checks a constant 4 neighbors.
> - **Space:** O(N × M) for the output copy, plus O(N × M) worst-case recursion stack depth (e.g. a long snake-shaped region of the same color forces deep recursion).

## ⚠️ Common Pitfalls

> [!warning]
> - **Not handling `originalColor == newColor`.** Without this check, the color-equality test never distinguishes "already filled" from "not yet filled," and the DFS recurses forever between already-colored neighbors — an infinite loop / stack overflow.
> - **Checking visited via a separate boolean array AND relying on the color check** — redundant, and easy to get out of sync. Once you guarantee `originalColor != newColor`, the color check alone is sufficient and simpler.
> - **Mutating the input image when the problem (or your own code style) expects the original preserved.** Many implementations recolor in place — that's fine and common — but be deliberate about it; copying first is the safer default when you're not sure if the caller still needs the original.
> - **Using 8-directional deltas out of habit** (carried over from Number of Islands) — Flood Fill is explicitly **4-directional only**. Mixing this up silently changes the answer.

## 🔗 Related Problems / Next Up

> [!success]
> - **Builds on:** [Number of Islands](Number-of-Islands.md) — same grid-as-graph idea, but writing instead of counting, and 4-directional instead of 8
> - **Pattern reused in:** Rotting Oranges, 01 Matrix, Surrounded Regions
> - [↑ Back to BFS/DFS Problems Index](00-BFS-DFS-Index.md)

