---
title: "Bipartite Graph (BFS)"
tags: [graph, bfs-dfs-problems, bipartite, bfs, graph-coloring]
videoLink: "https://www.youtube.com/playlist?list=PLgUwDviBIf0oE3gA41TKO2H5bHpPd7fzn"
difficulty: Medium
---

# Bipartite Graph (BFS)

## 🎯 What problem are we solving?

> [!question]
> Same problem as the DFS note — determine whether a graph is **bipartite** (2-colorable such that no two adjacent nodes share a color) — but solved with **BFS** instead of recursion. The coloring rule is identical; only the traversal mechanics change.

## 💡 Intuition

> [!tip]
> This is plain brute force: push a starting node into the queue with color `0`. Every time you pop a node, look at its neighbors — any **uncolored** neighbor gets the **opposite** color and gets pushed into the queue. Any neighbor that's **already colored** gets checked: if it matches the current node's color, that's a clash — two adjacent nodes ended up the same color, so the graph isn't bipartite. There's no deeper insight here beyond "fill in colors level by level and bail the moment two neighbors collide."

## 🖼️ Visualizing it

```
Graph:  1 - 2 - 3 - 4
                |
            6 - 5
```

Adjacency: 1-2, 2-3, 2-6, 3-4, 4-5, 5-6

```
queue = [1], color[1] = 0

pop 1: neighbor 2 uncolored → color[2] = 1, push 2

pop 2: neighbor 1 colored 0, opposite of 2's color 1 → fine
       neighbor 3 uncolored → color[3] = 0, push 3
       neighbor 6 uncolored → color[6] = 0, push 6

pop 3: neighbor 2 colored 1, opposite → fine
       neighbor 4 uncolored → color[4] = 1, push 4

pop 6: neighbor 2 colored 1, opposite → fine
       neighbor 5 uncolored → color[5] = 1, push 5

pop 4: neighbor 3 colored 0, opposite → fine
       neighbor 5 colored 1 — SAME as node 4's color (1) → CLASH → return false
```

Node 4 (color `1`) and node 5 (color `1`, colored earlier via the 6-5 edge) turn out to be adjacent — that's the odd cycle `2-3-4-5-6-2` (length 5) showing up again, just detected level-by-level instead of via recursion. **Not bipartite.**

## 🛠️ Algorithm / Approach

> [!abstract]
> 1. Initialize a `color` array of size `n`, all `-1` (uncolored).
> 2. For every node `1..n` not yet colored (handles disconnected components): push it into a queue with color `0`, set `color[start] = 0`.
> 3. While the queue isn't empty:
>    - Pop `node`.
>    - For each `neighbor` of `node`:
>      - If **uncolored**: assign `color[neighbor] = 1 - color[node]`, push it into the queue.
>      - Else if `color[neighbor] == color[node]`: **return `false`** — clash found.
> 4. If every component finishes with no clash, return `true`.

## 👨‍💻 Code

### Java

```java
import java.util.*;

public class BipartiteGraphBFS {

    private boolean check(int start, List<List<Integer>> adj, int[] color) {
        Queue<Integer> queue = new LinkedList<>();
        queue.offer(start);
        color[start] = 0;

        while (!queue.isEmpty()) {
            int node = queue.poll();

            for (int neighbor : adj.get(node)) {
                if (color[neighbor] == -1) {
                    color[neighbor] = 1 - color[node];
                    queue.offer(neighbor);
                } else if (color[neighbor] == color[node]) {
                    return false;   // adjacent node has the same color -> clash
                }
            }
        }
        return true;
    }

    public boolean isBipartite(int n, List<List<Integer>> adj) {
        int[] color = new int[n];
        Arrays.fill(color, -1);

        for (int i = 0; i < n; i++) {
            if (color[i] == -1) {
                if (!check(i, adj, color)) return false;
            }
        }
        return true;
    }
}
```

### Python

```python
from collections import deque

class Solution:
    def is_bipartite(self, n, adj):
        color = [-1] * n

        def check(start):
            queue = deque([start])
            color[start] = 0

            while queue:
                node = queue.popleft()
                for neighbor in adj[node]:
                    if color[neighbor] == -1:
                        color[neighbor] = 1 - color[node]
                        queue.append(neighbor)
                    elif color[neighbor] == color[node]:
                        return False   # adjacent node has the same color -> clash
            return True

        for i in range(n):
            if color[i] == -1:
                if not check(i):
                    return False
        return True
```

## ⏱️ Complexity Analysis

> [!note]
> - **Time:** O(N + 2E) — standard BFS traversal cost across all components combined.
> - **Space:** O(N) for the `color` array plus O(N) worst-case for the queue.

## ⚠️ Common Pitfalls

> [!warning]
> - **Only testing starting from node `0` / a single fixed node.** Graphs with multiple disconnected components need the outer loop over all nodes — otherwise components after the first are silently never checked, and larger test cases with disconnected pieces will fail.
> - **Initializing `color` to `0` instead of `-1`.** Since `0` is a legitimate color, you need `-1` as the "uncolored" sentinel to tell the two apart.
> - **Forgetting to check the clash condition for already-colored neighbors** — it's not enough to only color uncolored neighbors; every already-colored neighbor must be checked for a same-color collision too.
> - **Stopping at the first clash inside the wrong scope.** The `false` needs to propagate all the way out of `check()` and then out of the outer loop in `isBipartite()` — a `false` swallowed only in the inner loop doesn't stop the outer component loop.

## 🔗 Related Problems / Next Up

> [!success]
> - **Companion:** [Bipartite Graph (DFS)](Bipartite-Graph-DFS.md) — identical coloring logic and identical odd-cycle intuition, recursive instead of queue-based
> - [↑ Back to BFS/DFS Problems Index](00-BFS-DFS-Index.md)
