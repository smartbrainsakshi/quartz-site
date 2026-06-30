---
title: "Detect Cycle in an Undirected Graph (BFS)"
tags: [graph, bfs-dfs-problems, cycle-detection, bfs, undirected-graph]
videoLink: "https://www.youtube.com/playlist?list=PLgUwDviBIf0oE3gA41TKO2H5bHpPd7fzn"
difficulty: Medium
---

# Detect Cycle in an Undirected Graph (BFS)

## 🎯 What problem are we solving?

> [!question]
> Given an undirected graph, determine whether it contains a **cycle** — a path that starts at a node and, through some sequence of edges, returns to that same node (the definition from G-1). This is the first "yes/no property of the graph" problem in the series, and it introduces a technique (tracking each node's *parent* during traversal) that reappears constantly in later cycle-detection and tree problems.

## 💡 Intuition

> [!tip]
> Think about what BFS actually does: from a starting node, it fans out level by level, and **every node (except the start) gets discovered by exactly one "parent" edge** — *unless the graph has a cycle.*
>
> If a graph has no cycle, every node you reach during BFS is being visited for the first time — it has exactly one path back to the root through its parent chain. **But if BFS ever tries to visit a node that's already been visited, and that node is *not* the parent of the node you're currently standing on** — that's only possible because *two different paths* converged on the same node. Two distinct paths reaching the same node is exactly the definition of a cycle.
>
> > **The core rule:** while exploring neighbor `nbr` of node `cur`, if `nbr` is already visited **and** `nbr != parent(cur)`, the graph has a cycle.
>
> Why exclude the parent? Because BFS naturally "looks back" at the node it just came from (every undirected edge stores both directions in the adjacency list) — that's not a cycle, that's just the edge you arrived on. Only a *different* already-visited node colliding with your path signals an actual cycle.

## 🖼️ Visualizing it

```
        1
       / \
      2   3
      |   |\
      5   6 4
       \ /
        7
```
Edges: (1,2), (1,3), (2,5), (3,6), (3,4), (5,7), (6,7)

**BFS from node 1** (storing `(node, parent)` pairs):

```
Queue: [(1, -1)]                      Visited: {1}

Pop (1, -1): neighbors 2, 3 — both unvisited → push (2,1), (3,1)
Queue: [(2,1), (3,1)]                 Visited: {1,2,3}

Pop (2, 1): neighbors 1(=parent, skip), 5(unvisited) → push (5,2)
Queue: [(3,1), (5,2)]                 Visited: {1,2,3,5}

Pop (3, 1): neighbors 1(=parent, skip), 6(unvisited), 4(unvisited) → push (6,3), (4,3)
Queue: [(5,2), (6,3), (4,3)]          Visited: {1,2,3,5,6,4}

Pop (5, 2): neighbors 2(=parent, skip), 7(unvisited) → push (7,5)
Queue: [(6,3), (4,3), (7,5)]          Visited: {1,2,3,5,6,4,7}

Pop (6, 3): neighbors 3(=parent, skip), 7(VISITED, and parent of 6 is 3, not 7!) → CYCLE FOUND
```

`6`'s neighbor `7` is already visited, but `7` is not `6`'s parent (`6`'s parent is `3`) — this collision is exactly two different BFS paths (`1→2→5→7` and `1→3→6→7`) meeting at node 7. **Cycle confirmed.**

## 🛠️ Algorithm / Approach

> [!abstract]
> **`hasCycleBFS(source, adj, visited)`:**
> 1. Mark `source` visited. Push `(source, parent = -1)` into the queue.
> 2. While the queue isn't empty:
>    - Pop `(cur, parent)`.
>    - For each `neighbor` of `cur`:
>      - If `!visited[neighbor]`: mark visited, push `(neighbor, cur)`.
>      - Else if `neighbor != parent`: **cycle detected** → return `true`.
> 3. Queue exhausted with no collision → return `false` (no cycle reachable from `source`).
>
> **Handling multiple components** (same G-3 pattern):
> 4. Loop every node `1..n`; if unvisited, call `hasCycleBFS` from it. If any call returns `true`, the graph has a cycle. If every component finishes with no cycle, the graph is acyclic.

## 👨‍💻 Code

### Java

```java
import java.util.*;

public class DetectCycleUndirectedBFS {

    private boolean hasCycleBFS(int source, List<List<Integer>> adj, boolean[] visited) {
        visited[source] = true;
        Queue<int[]> queue = new LinkedList<>();   // {node, parent}
        queue.offer(new int[]{source, -1});

        while (!queue.isEmpty()) {
            int[] cur = queue.poll();
            int node = cur[0], parent = cur[1];

            for (int neighbor : adj.get(node)) {
                if (!visited[neighbor]) {
                    visited[neighbor] = true;
                    queue.offer(new int[]{neighbor, node});
                } else if (neighbor != parent) {
                    return true;   // visited, and not where we came from -> cycle
                }
            }
        }
        return false;
    }

    public boolean isCycle(int n, List<List<Integer>> adj) {
        boolean[] visited = new boolean[n + 1];   // adjust if 0-indexed

        for (int i = 1; i <= n; i++) {
            if (!visited[i]) {
                if (hasCycleBFS(i, adj, visited)) return true;
            }
        }
        return false;
    }
}
```

### Python

```python
from collections import deque

class Solution:
    def is_cycle(self, n, adj):
        visited = [False] * (n + 1)   # adjust if 0-indexed

        def has_cycle_bfs(source):
            visited[source] = True
            queue = deque([(source, -1)])   # (node, parent)

            while queue:
                node, parent = queue.popleft()
                for neighbor in adj[node]:
                    if not visited[neighbor]:
                        visited[neighbor] = True
                        queue.append((neighbor, node))
                    elif neighbor != parent:
                        return True   # visited, and not where we came from -> cycle
            return False

        for i in range(1, n + 1):
            if not visited[i]:
                if has_cycle_bfs(i):
                    return True
        return False
```

## ⏱️ Complexity Analysis

> [!note]
> - **Time:** O(N + 2E) — same bound as plain BFS (G-4). Even though `isCycle` calls `hasCycleBFS` once per unvisited starting node, the *combined* work across all those calls still equals exactly one full traversal of the graph — same argument as in Number of Provinces.
> - **Space:** O(N) for the `visited` array + O(N) for the queue (each entry now stores a pair, but that's still O(1) extra per node, not asymptotically more).

## ⚠️ Common Pitfalls

> [!warning]
> - **Forgetting to track/check the parent**, and instead flagging *any* already-visited neighbor as a cycle. This is wrong — every undirected edge appears in both directions in the adjacency list, so the node you just came from will always show up as an already-visited neighbor; that's not a cycle, just backtracking.
> - **Comparing against the wrong parent.** The parent must be tracked *per queue entry* (i.e., "the node I popped *from* to reach this neighbor"), not some single global "previous node" variable — that breaks the moment BFS branches into multiple directions.
> - **Stopping the multi-component loop after the first component finds no cycle.** A graph can have several disconnected pieces, and a cycle could exist in *any* of them — you must check every unvisited node as a potential new starting point (same as Number of Provinces), not just the first one.
> - **Re-scanning an already-fully-explored component for a cycle "just to be sure."** Once a component's BFS completes without finding a collision, there's no need to recheck it — the visited array already guarantees you'll skip it naturally on subsequent loop iterations.

## 🔗 Related Problems / Next Up

> [!success]
> - **Builds on:** [G-4 — BFS Traversal](../01-Foundations/G4-BFS-Traversal.md), [G-3 — Connected Components](../01-Foundations/G3-Connected-Components.md)
> - **Next:** Detect Cycle in an Undirected Graph (DFS version — same idea, different mechanics for tracking parent)
> - [↑ Back to BFS/DFS Problems Index](00-BFS-DFS-Index.md)

