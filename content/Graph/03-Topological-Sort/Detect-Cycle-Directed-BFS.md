---
title: "Detect Cycle in a Directed Graph (BFS / Kahn's Algorithm)"
tags: [graph, topological-sort, bfs, cycle-detection, directed-graph, kahns-algorithm]
videoLink: "https://www.youtube.com/playlist?list=PLgUwDviBIf0oE3gA41TKO2H5bHpPd7fzn"
difficulty: Medium
---

# Detect Cycle in a Directed Graph (BFS / Kahn's Algorithm)

## 🎯 What problem are we solving?

> [!question]
> Same problem as the earlier DFS note — detect whether a directed graph has a cycle — but this time using **BFS**, by repurposing **Kahn's topological sort algorithm**.

## 💡 Intuition

> [!tip]
> The DFS approach used `visited` + `pathVisited`, resetting `pathVisited` on the way back up out of the recursion. But **BFS has no backtracking** — there's no "way back up" to unmark anything — so that exact trick doesn't translate.
>
> The fix: **just run Kahn's algorithm and see what happens.** Kahn's algorithm only works cleanly on a DAG — a graph with **no** cycle. If the graph *does* have a cycle, every node in that cycle (and depending on it) has an in-degree that **never reaches 0**, because their incoming edges all come from other nodes stuck in the same cyclic dependency — so those nodes simply never get pushed into the queue.
>
> The consequence: **topological sort produces exactly `n` elements if and only if the graph has no cycle.** If Kahn's algorithm terminates with fewer than `n` nodes processed, the missing nodes are exactly the ones tangled in a cycle — so a shorter-than-`n` result is proof of a cycle.

## 🖼️ Visualizing it

```
Edges: 1→2, 2→3, 3→1     (a pure 3-node cycle, n = 3)

In-degrees: 1: 1 (from 3)   2: 1 (from 1)   3: 1 (from 2)
```

```
queue = []   -- no node has in-degree 0!

Kahn's algorithm never even starts. Result has 0 elements.

0 != n (which is 3) → the graph has a cycle.
```

Every node in this graph depends on another node in the same loop — none of them ever reaches in-degree 0, so the queue starts empty and the topological sort produces nothing at all. Compare that count (`0`) to `n` (`3`): they don't match, so a cycle is confirmed.

## 🛠️ Algorithm / Approach

> [!abstract]
> 1. Run standard Kahn's algorithm (BFS-based topological sort): compute in-degrees, push all in-degree-0 nodes into a queue, repeatedly pop and decrement neighbors' in-degrees, pushing any that reach 0.
> 2. Keep a **counter** of how many nodes were successfully popped/processed (no need to store the actual ordering if you only care about cycle detection).
> 3. After the queue empties, compare the counter to `n` (total number of nodes):
>    - If `counter == n`: no cycle — the graph is a DAG.
>    - If `counter < n`: a cycle exists among the unprocessed nodes.

## 👨‍💻 Code

### Java

```java
import java.util.*;

public class DetectCycleDirectedBFS {

    public boolean isCyclic(int n, List<List<Integer>> adj) {
        int[] inDegree = new int[n];
        for (int i = 0; i < n; i++) {
            for (int neighbor : adj.get(i)) {
                inDegree[neighbor]++;
            }
        }

        Queue<Integer> queue = new LinkedList<>();
        for (int i = 0; i < n; i++) {
            if (inDegree[i] == 0) queue.offer(i);
        }

        int count = 0;
        while (!queue.isEmpty()) {
            int node = queue.poll();
            count++;

            for (int neighbor : adj.get(node)) {
                inDegree[neighbor]--;
                if (inDegree[neighbor] == 0) {
                    queue.offer(neighbor);
                }
            }
        }

        return count != n;   // fewer than n processed -> a cycle exists
    }
}
```

### Python

```python
from collections import deque

class Solution:
    def is_cyclic(self, n, adj):
        in_degree = [0] * n
        for u in range(n):
            for v in adj[u]:
                in_degree[v] += 1

        queue = deque([i for i in range(n) if in_degree[i] == 0])
        count = 0

        while queue:
            node = queue.popleft()
            count += 1

            for neighbor in adj[node]:
                in_degree[neighbor] -= 1
                if in_degree[neighbor] == 0:
                    queue.append(neighbor)

        return count != n   # fewer than n processed -> a cycle exists
```

## ⏱️ Complexity Analysis

> [!note]
> - **Time:** O(V + E) — identical to plain Kahn's algorithm; only a counter comparison is added at the end.
> - **Space:** O(N) for `inDegree` and the queue — no `pathVisited` array needed, unlike the DFS approach.

## ⚠️ Common Pitfalls

> [!warning]
> - **Comparing the wrong quantities.** The check is `count != n` (nodes *processed*), not queue size or list length at some intermediate point — a cycle can leave the queue empty long before all nodes are handled.
> - **Assuming an empty queue always means success.** An empty queue simply means the BFS is *done*; whether that's a success (all `n` processed) or a cycle (fewer than `n` processed) depends entirely on the final count.
> - **Recomputing in-degrees incorrectly for a graph that already has an in-degree array from elsewhere** — always compute in-degree fresh for the actual graph being checked, since a stale or partially-filled in-degree array causes false cycle results.
> - **Forgetting this requires the prerequisite Kahn's algorithm to be understood first** — if the topological sort mechanics themselves aren't clear, debugging *why* a cycle causes a short result is much harder.

## 🔗 Related Problems / Next Up

> [!success]
> - **Builds on:** [Topological Sort (BFS / Kahn's Algorithm)](Topological-Sort-BFS-Kahns-Algorithm.md)
> - **Contrast with:** [Detect Cycle in a Directed Graph (DFS)](../02-BFS-DFS-Problems/Detect-Cycle-Directed-DFS.md) — same problem, `visited`/`pathVisited` recursion instead of in-degree BFS
> - **Used by:** [Course Schedule I](Course-Schedule-I.md) — "is it possible to finish all courses" is exactly "does this prerequisite graph have a cycle"
> - [↑ Back to Topological Sort Index](00-Topological-Sort-Index.md)
