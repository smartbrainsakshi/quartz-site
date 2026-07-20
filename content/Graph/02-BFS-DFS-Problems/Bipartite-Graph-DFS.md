---
title: "Bipartite Graph (DFS)"
tags: [graph, bfs-dfs-problems, bipartite, dfs, graph-coloring]
videoLink: "https://www.youtube.com/playlist?list=PLgUwDviBIf0oE3gA41TKO2H5bHpPd7fzn"
difficulty: Medium
---

# Bipartite Graph (DFS)

## 🎯 What problem are we solving?

> [!question]
> A graph is **bipartite** if its nodes can be colored using **exactly two colors** such that no two adjacent nodes share the same color. Given a graph, determine whether it is bipartite — using **DFS** to attempt the coloring.
>
> Key structural fact: a graph is bipartite **if and only if it contains no odd-length cycle**. Any graph with only even-length cycles (or no cycles at all — e.g. a straight line / tree) is always 2-colorable; any graph with an odd-length cycle can never be 2-colored, because you'll always end up forcing two adjacent nodes into the same color somewhere along that odd loop.

## 💡 Intuition

> [!tip]
> Since a bipartite graph is just "2-colorable such that neighbors differ," the natural approach is to **actually try coloring it** and see if you ever get stuck.
>
> Start anywhere, give it color `0`. Every neighbor of a `0`-colored node must be `1`, and every neighbor of a `1`-colored node must be `0` — colors must **alternate** with each DFS step. The moment DFS reaches a node that's **already colored** and that color happens to **match** the color it's about to be assigned, that's a contradiction: two adjacent nodes ended up the same color, which is only possible if the graph has an odd cycle. In that case, immediately report "not bipartite." If the entire DFS finishes without ever finding such a clash, the coloring succeeded and the graph is bipartite.

## 🖼️ Visualizing it

```
Graph:  1 - 2 - 3 - 4
            |   |
            6 - 5 - 7 - 8
```

Adjacency (undirected): 1-2, 2-3, 2-6, 3-4, 4-5, 5-6, 5-7, 7-8

```
dfs(1, color=0): color[1]=0
  neighbor 2 (uncolored) → dfs(2, color=1): color[2]=1
    neighbor 1: colored 0, opposite of 1 → fine, already visited, skip
    neighbor 3 (uncolored) → dfs(3, color=0): color[3]=0
      neighbor 2: colored 1, opposite → fine
      neighbor 4 (uncolored) → dfs(4, color=1): color[4]=1
        neighbor 3: colored 0, opposite → fine
        neighbor 7 (uncolored) → dfs(7, color=0): color[7]=0
          neighbor 4: colored 1, opposite → fine
          neighbor 8 (uncolored) → dfs(8, color=1): color[8]=1
            neighbor 7: colored 0, opposite → fine. No more neighbors → done, no clash
          neighbor 5 (uncolored) → dfs(5, color=1): color[5]=1
            neighbor 4: colored 1, opposite → fine
            neighbor 6 (uncolored) → dfs(6, color=0): color[6]=0
              neighbor 2: colored 1, opposite → fine
              neighbor 5: colored 1, opposite → fine. Done.
            neighbor 7: colored 0, opposite → fine. Done.
      neighbor 6: colored 0 — SAME as node 3's color (0) → CLASH → return false
```

The DFS finds that node **6** (color `0`) is adjacent to node **3** (also color `0`) via a different path than the one that colored it — that's an odd cycle (`2-3-4-5-6-2`, length 5), so the whole call chain unwinds returning `false`: **not bipartite**.

## 🛠️ Algorithm / Approach

> [!abstract]
> **`dfs(node, color, adj, colorArr) → boolean`:**
> 1. Assign `colorArr[node] = color`.
> 2. For each `neighbor` of `node`:
>    - If `neighbor` is **uncolored** (`colorArr[neighbor] == -1`): recursively call `dfs(neighbor, 1 - color, adj, colorArr)`. **If that call returns `false`, immediately return `false`.**
>    - Else if `colorArr[neighbor] == color` (same color as current node): **return `false`** — clash found.
> 3. If the loop finishes with no clash, return `true`.
>
> **Multi-component handling:** initialize `colorArr` to `-1` for every node. Loop over every node `1..n`; if still uncolored, call `dfs(i, 0, ...)`. If **any** component's DFS returns `false`, the whole graph is not bipartite.

## 👨‍💻 Code

### Java

```java
import java.util.*;

public class BipartiteGraphDFS {

    private boolean dfs(int node, int color, List<List<Integer>> adj, int[] colorArr) {
        colorArr[node] = color;

        for (int neighbor : adj.get(node)) {
            if (colorArr[neighbor] == -1) {
                if (!dfs(neighbor, 1 - color, adj, colorArr)) {
                    return false;   // propagate the clash upward immediately
                }
            } else if (colorArr[neighbor] == color) {
                return false;       // adjacent node has the same color -> clash
            }
        }
        return true;
    }

    public boolean isBipartite(int n, List<List<Integer>> adj) {
        int[] colorArr = new int[n];
        Arrays.fill(colorArr, -1);

        for (int i = 0; i < n; i++) {
            if (colorArr[i] == -1) {
                if (!dfs(i, 0, adj, colorArr)) return false;
            }
        }
        return true;
    }
}
```

### Python

```python
class Solution:
    def is_bipartite(self, n, adj):
        color = [-1] * n

        def dfs(node, c):
            color[node] = c
            for neighbor in adj[node]:
                if color[neighbor] == -1:
                    if not dfs(neighbor, 1 - c):
                        return False   # propagate the clash upward immediately
                elif color[neighbor] == c:
                    return False       # adjacent node has the same color -> clash
            return True

        for i in range(n):
            if color[i] == -1:
                if not dfs(i, 0):
                    return False
        return True
```

## ⏱️ Complexity Analysis

> [!note]
> - **Time:** O(N + 2E) — standard DFS traversal cost: each node visited once, each edge examined at most twice (once from each endpoint).
> - **Space:** O(N) for the `colorArr`, plus O(N) worst-case recursion stack depth.

## ⚠️ Common Pitfalls

> [!warning]
> - **Initializing colors to `0` instead of `-1`.** `0` is a valid color — you need a distinct sentinel (`-1`) to distinguish "uncolored" from "colored with color 0."
> - **Forgetting to immediately return `false` up the call chain** the instant a clash is found, instead of letting the recursion continue checking unrelated neighbors.
> - **Not handling disconnected graphs** — each component must be colored independently starting fresh from color `0`; skipping the outer loop over all nodes will silently miss components after the first.
> - **Assuming any cycle makes a graph non-bipartite.** Only **odd-length** cycles do — even-length cycles are perfectly 2-colorable, so don't special-case "has a cycle" as an automatic fail.

## 🔗 Related Problems / Next Up

> [!success]
> - **Builds on:** [G5. DFS Traversal](../01-Foundations/G5-DFS-Traversal.md)
> - **Companion:** [Bipartite Graph (BFS)](Bipartite-Graph-BFS.md) — identical coloring logic, level-by-level instead of recursive
> - [↑ Back to BFS/DFS Problems Index](00-BFS-DFS-Index.md)
