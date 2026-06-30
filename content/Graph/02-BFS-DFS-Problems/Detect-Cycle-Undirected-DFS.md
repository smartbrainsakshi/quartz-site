---
title: "Detect Cycle in an Undirected Graph (DFS)"
tags: [graph, bfs-dfs-problems, cycle-detection, dfs, undirected-graph]
videoLink: "https://www.youtube.com/playlist?list=PLgUwDviBIf0oE3gA41TKO2H5bHpPd7fzn"
difficulty: Medium
---

# Detect Cycle in an Undirected Graph (DFS)

## 🎯 What problem are we solving?

> [!question]
> Same problem as the previous note — detect whether an undirected graph has a cycle — but solved with **DFS** instead of BFS. The core idea (track each node's parent, flag a collision with a non-parent visited node) carries over directly; only the mechanics of *how* the collision propagates back up change, since DFS uses recursion instead of a queue.

## 💡 Intuition

> [!tip]
> DFS commits to a single path and goes as deep as possible before backtracking (G-5). If, while going deep, DFS ever lands on a node that's **already visited** — and that node isn't the one it just came from (its parent) — it means **the current path looped back onto itself**, which is exactly a cycle.
>
> > Same rule as BFS: while exploring neighbor `nbr` of node `cur`, if `nbr` is visited and `nbr != parent`, there's a cycle.
>
> The new piece with DFS/recursion: a cycle found deep in the recursion needs to **propagate back up** through every parent call so the original caller knows about it. This is done by having `dfs(...)` **return a boolean** — `true` the instant a cycle is found, and every calling frame immediately returns `true` too without bothering to check further neighbors.

## 🖼️ Visualizing it

Same graph as the BFS note: edges (1,2), (1,3), (2,5), (3,6), (3,4), (5,7), (6,7).

```
        1
       / \
      2   3
      |   |\
      5   6 4
       \ /
        7
```

**DFS from node 1, parent = -1:**

```
dfs(1, parent=-1): mark visited
  neighbor 2: unvisited → dfs(2, parent=1)
    mark visited. neighbor 1: visited, but 1 == parent(2) → skip (just backtracking, not a cycle)
    neighbor 5: unvisited → dfs(5, parent=2)
      mark visited. neighbor 2: visited, == parent(5) → skip
      neighbor 7: unvisited → dfs(7, parent=5)
        mark visited. neighbor 5: visited, == parent(7) → skip
        neighbor 6: unvisited → dfs(6, parent=7)
          mark visited. neighbor 7: visited, == parent(6) → skip
          neighbor 3: unvisited → dfs(3, parent=6)
            mark visited. neighbor 6: visited, == parent(3) → skip
            neighbor 4: unvisited → dfs(4, parent=3)
              mark visited. neighbor 3: visited, == parent(4) → skip. No more neighbors → return false
            neighbor 1: VISITED, and 1 != parent(3) [parent(3) is 6] → CYCLE → return true
          dfs(3,...) returned true → dfs(6,...) returns true immediately, no need to check more
        dfs(6,...) returned true → dfs(7,...) returns true immediately
      dfs(7,...) returned true → dfs(5,...) returns true immediately
    dfs(5,...) returned true → dfs(2,...) returns true immediately
  dfs(2,...) returned true → dfs(1,...) returns true immediately (never even tries neighbor 3)
```

**Result: `true` — the graph has a cycle.** Notice how the moment `3`'s call detects the collision with `1`, **every single parent frame above it short-circuits and returns `true` without doing any further work** — that's the "propagate up the instant you find it" behavior.

## 🛠️ Algorithm / Approach

> [!abstract]
> **`dfs(node, parent, adj, visited) → boolean`:**
> 1. Mark `node` visited.
> 2. For each `neighbor` of `node`:
>    - If `neighbor` is **not visited**: recursively call `dfs(neighbor, node, adj, visited)`. **If that call returns `true`, immediately return `true`** (don't check remaining neighbors).
>    - Else if `neighbor != parent`: this neighbor was visited by some *other* path → **return `true`** (cycle found).
> 3. If the loop finishes with no cycle found among any neighbor → return `false`.
>
> **Multi-component handling** (same as BFS version): loop every node `1..n`; if unvisited, call `dfs(i, -1, ...)`. If any call returns `true`, the graph has a cycle.

## 👨‍💻 Code

### Java

```java
import java.util.*;

public class DetectCycleUndirectedDFS {

    private boolean dfs(int node, int parent, List<List<Integer>> adj, boolean[] visited) {
        visited[node] = true;

        for (int neighbor : adj.get(node)) {
            if (!visited[neighbor]) {
                if (dfs(neighbor, node, adj, visited)) {
                    return true;   // propagate the cycle finding upward immediately
                }
            } else if (neighbor != parent) {
                return true;       // visited, and not where we came from -> cycle
            }
        }
        return false;
    }

    public boolean isCycle(int n, List<List<Integer>> adj) {
        boolean[] visited = new boolean[n + 1];   // adjust if 0-indexed

        for (int i = 1; i <= n; i++) {
            if (!visited[i]) {
                if (dfs(i, -1, adj, visited)) return true;
            }
        }
        return false;
    }
}
```

### Python

```python
class Solution:
    def is_cycle(self, n, adj):
        visited = [False] * (n + 1)   # adjust if 0-indexed

        def dfs(node, parent):
            visited[node] = True
            for neighbor in adj[node]:
                if not visited[neighbor]:
                    if dfs(neighbor, node):
                        return True   # propagate the cycle finding upward immediately
                elif neighbor != parent:
                    return True       # visited, and not where we came from -> cycle
            return False

        for i in range(1, n + 1):
            if not visited[i]:
                if dfs(i, -1):
                    return True
        return False
```

## ⏱️ Complexity Analysis

> [!note]
> - **Time:** O(N + 2E) — same reasoning as the BFS version: across all calls (the outer loop plus every recursive `dfs`), each node is visited exactly once and each edge examined at most twice.
> - **Space:** O(N) for the `visited` array, **plus O(N) worst-case recursion stack** (the DFS-specific cost, versus the queue used by BFS).

## ⚠️ Common Pitfalls

> [!warning]
> - **Forgetting to immediately return `true` up the call chain.** If a recursive call detects a cycle but the caller keeps checking its *other* neighbors instead of returning immediately, you risk overwriting the `true` result or doing unnecessary extra work — always short-circuit the moment a deeper call returns `true`.
> - **Confusing "parent" with "already visited at some earlier, unrelated point in the traversal."** The parent check must be the *immediate* node you recursed from, not any arbitrary previously-seen node.
> - **Mixing up BFS-queue-style parent tracking with DFS-recursion-style parent tracking.** In DFS, the parent is simply the previous function argument — no need for a separate `(node, parent)` pair structure in a queue, since the call stack naturally carries that context.
> - **Not handling disconnected graphs** — same as the BFS version, you must loop over all nodes and trigger a fresh DFS from each unvisited one, since a cycle could exist in any component.

## 🔗 Related Problems / Next Up

> [!success]
> - **Builds on:** [Detect Cycle in an Undirected Graph (BFS)](Detect-Cycle-Undirected-BFS.md), [G-5 — DFS Traversal](../01-Foundations/G5-DFS-Traversal.md)
> - **Contrast with:** cycle detection in a *directed* graph (covered later) — the parent-exclusion trick alone isn't enough there; it needs a separate "currently in recursion stack" marker instead.
> - [↑ Back to BFS/DFS Problems Index](00-BFS-DFS-Index.md)

