---
title: "Topological Sort (BFS / Kahn's Algorithm)"
tags: [graph, topological-sort, bfs, dag, kahns-algorithm]
videoLink: "https://www.youtube.com/playlist?list=PLgUwDviBIf0oE3gA41TKO2H5bHpPd7fzn"
difficulty: Medium
---

# Topological Sort (BFS / Kahn's Algorithm)

## 🎯 What problem are we solving?

> [!question]
> Same problem as the DFS note — produce a valid topological ordering of a DAG's vertices (`u` before `v` for every edge `u → v`) — but solved iteratively using **BFS**. This specific technique is famously known as **Kahn's Algorithm**.

## 💡 Intuition

> [!tip]
> The DFS approach relies on recursion and "finish order." Kahn's algorithm instead uses a very concrete idea: **in-degree** — the number of incoming edges to a node.
>
> Any node with **in-degree 0** has no dependency blocking it — nothing needs to come before it — so it's always safe to place it next in the ordering right away. Since the graph is a DAG (no cycles), **there's always at least one node with in-degree 0** to start with.
>
> The algorithm: push all in-degree-0 nodes into a queue. Pop a node, add it to the result, then look at its neighbors and **decrement their in-degree by 1** — you're conceptually "removing" that node and its outgoing edges from the graph. Any neighbor whose in-degree just hit 0 (all its dependencies are now satisfied) gets pushed into the queue next. Repeat until the queue is empty.

## 🖼️ Visualizing it

```
Edges: 5→0, 4→0, 5→2, 2→3, 3→1, 4→1

In-degrees:  0: 2 (from 5, 4)   1: 2 (from 3, 4)   2: 1 (from 5)
             3: 1 (from 2)      4: 0               5: 0
```

```
queue = [4, 5]   (both start with in-degree 0)

pop 4 → result = [4]
  neighbor 0: in-degree 2→1
  neighbor 1: in-degree 2→1

pop 5 → result = [4, 5]
  neighbor 0: in-degree 1→0 → push 0
  neighbor 2: in-degree 1→0 → push 2

pop 0 → result = [4, 5, 0]
  no outgoing edges

pop 2 → result = [4, 5, 0, 2]
  neighbor 3: in-degree 1→0 → push 3

pop 3 → result = [4, 5, 0, 2, 3]
  neighbor 1: in-degree 1→0 → push 1

pop 1 → result = [4, 5, 0, 2, 3, 1]
  no outgoing edges

queue empty. Done.
```

**Result: `4, 5, 0, 2, 3, 1`** — a valid topological order. Every node that had an edge into it was only released once *all* its incoming dependencies had already been placed.

## 🛠️ Algorithm / Approach

> [!abstract]
> 1. Compute `inDegree[node]` for every node by scanning the adjacency list once (for every edge `u → v`, increment `inDegree[v]`).
> 2. Push every node with `inDegree == 0` into a queue.
> 3. While the queue isn't empty:
>    - Pop `node`, append it to the result list.
>    - For each `neighbor` of `node`: decrement `inDegree[neighbor]`. If it becomes `0`, push `neighbor` into the queue.
> 4. Return the result list — it's a valid topological order once the queue is exhausted.

## 👨‍💻 Code

### Java

```java
import java.util.*;

public class TopologicalSortBFS {

    public int[] topoSort(int n, List<List<Integer>> adj) {
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

        int[] result = new int[n];
        int idx = 0;

        while (!queue.isEmpty()) {
            int node = queue.poll();
            result[idx++] = node;

            for (int neighbor : adj.get(node)) {
                inDegree[neighbor]--;
                if (inDegree[neighbor] == 0) {
                    queue.offer(neighbor);
                }
            }
        }
        return result;
    }
}
```

### Python

```python
from collections import deque

class Solution:
    def topo_sort(self, n, adj):
        in_degree = [0] * n
        for u in range(n):
            for v in adj[u]:
                in_degree[v] += 1

        queue = deque([i for i in range(n) if in_degree[i] == 0])
        result = []

        while queue:
            node = queue.popleft()
            result.append(node)

            for neighbor in adj[node]:
                in_degree[neighbor] -= 1
                if in_degree[neighbor] == 0:
                    queue.append(neighbor)

        return result
```

## ⏱️ Complexity Analysis

> [!note]
> - **Time:** O(V + E) — computing in-degrees is O(V + E), and the BFS itself processes each node once and each edge once.
> - **Space:** O(N) for `inDegree` and the queue.

## ⚠️ Common Pitfalls

> [!warning]
> - **Forgetting to compute in-degrees for *all* nodes upfront** before starting the queue — nodes with in-degree 0 that were never scanned won't get pushed, and the algorithm silently produces an incomplete ordering.
> - **Not verifying the result covers all `n` nodes when the graph might have a cycle.** If the input isn't guaranteed to be a DAG, a shorter-than-`n` result means a cycle exists and blocked some nodes from ever reaching in-degree 0 (this is exactly the basis of BFS-based cycle detection in directed graphs).
> - **Decrementing in-degree but forgetting the `== 0` check** before pushing — pushing every decremented neighbor regardless of its resulting in-degree would push the same node multiple times.
> - **Confusing in-degree with out-degree.** Kahn's algorithm is driven entirely by in-degree; out-degree plays no role here (contrast with computing *terminal nodes*, which is about out-degree).

## 🔗 Related Problems / Next Up

> [!success]
> - **Companion:** [Topological Sort (DFS)](Topological-Sort-DFS.md) — same result, recursive/stack-based instead of iterative/queue-based
> - **Enables:** [Detect Cycle in a Directed Graph (BFS / Kahn's)](Detect-Cycle-Directed-BFS.md) — if Kahn's algorithm can't place all `n` nodes, the graph has a cycle
> - [↑ Back to Topological Sort Index](00-Topological-Sort-Index.md)
