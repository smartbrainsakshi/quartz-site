---
title: "Find Eventual Safe States (BFS / Topological Sort)"
tags: [graph, topological-sort, bfs, kahns-algorithm, directed-graph]
videoLink: "https://www.youtube.com/playlist?list=PLgUwDviBIf0oE3gA41TKO2H5bHpPd7fzn"
difficulty: Medium
---

# Find Eventual Safe States (BFS / Topological Sort)

## 🎯 What problem are we solving?

> [!question]
> Same problem as the [earlier DFS note](../02-BFS-DFS-Problems/Eventual-Safe-States-DFS.md): a node is **safe** if every path starting from it eventually reaches a **terminal node** (out-degree 0). Return all safe nodes, sorted ascending — this time solved with **BFS**, by reversing the graph and reusing **Kahn's algorithm**.

## 💡 Intuition

> [!tip]
> Kahn's algorithm is naturally built around **in-degree** and **terminal-ness** (in the sense that a node with in-degree 0 is "ready"). But this problem is defined in terms of **out-degree** (a terminal node has no *outgoing* edges). To make Kahn's algorithm directly applicable, **reverse every edge in the graph first**. After reversing, a node that used to be terminal (out-degree 0 in the original graph) now has **in-degree 0** in the reversed graph — exactly the starting condition Kahn's algorithm needs.
>
> Then just run Kahn's algorithm on the reversed graph: start from the (formerly-terminal) in-degree-0 nodes, and repeatedly "peel off" a node, decrementing the in-degree of whoever it points to (in the reversed graph — i.e., whoever used to point *to* it originally). Any node whose reversed in-degree drops to 0 means **all of its (original) outgoing paths have now been accounted for and confirmed safe** — so it gets added to the queue and marked safe too. This is the same propagation idea as the DFS version ("anyone whose every path is settled is safe"), just executed as an iterative peeling process instead of recursion.

## 🖼️ Visualizing it

```
Original edges: 0→1, 0→2, 1→2, 2→3, 4→5, 5→6, 6→7, 7→5,
                8→1, 8→9, 9→10, 9→11, 10→8
Terminal node in original graph (out-degree 0): 7 has an edge to 5, so not terminal here —
use the video's actual reduced example instead:
```

```
Reversed-graph in-degrees (after flipping every edge):
node 7 (terminal in original, out-degree 0) → in-degree 0 in reversed graph

queue = [7]   (only node with reversed in-degree 0)

pop 7 → safe. In the reversed graph, 7's neighbor is 6 (since original was 6→7).
  reduce reversed in-degree of 6 → becomes 0 → push 6

pop 6 → safe. Reversed neighbors: 5 and 4 (original edges were 5→6, 4→6).
  reduce in-degree of 5 → 0 → push 5
  reduce in-degree of 4 → 0 → push 4

pop 5 → safe. Reversed neighbor: 2 (original 2→5).
  reduce in-degree of 2 → still not 0 yet (2 has other incoming original edges to account for)

pop 4 → safe. Reversed neighbors: 2, 3 (original 4→2, 4→3).
  reduce in-degree of 2 → now 0 → push 2
  reduce in-degree of 3 → 0 → push 3

pop 2 → safe. Reversed neighbor: 1 (original 1→2).
  reduce in-degree of 1 → 0 → push 1

pop 3 → safe. Reversed neighbor: 1 (original 1→3).
  (1 already fully accounted for above)

pop 1 → safe. Reversed neighbor: 0 (original 0→1).
  reduce in-degree of 0 → 0 → push 0

pop 0 → safe. No more reversed neighbors.

queue empty. Safe nodes: {7, 6, 5, 4, 2, 3, 1, 0} → sorted: [0,1,2,3,4,5,6,7]
```

Every node in this component gets peeled off starting from the terminal node backward — exactly mirroring the DFS version's result, just computed by working from terminals outward via reversed edges instead of recursing forward and backtracking. Nodes involved in an actual cycle (like `8, 9, 10` with edge `10→8`) never reach reversed in-degree 0 and are correctly excluded from the safe set.

## 🛠️ Algorithm / Approach

> [!abstract]
> 1. Build the **reversed** adjacency list: for every original edge `u → v`, add `v → u` in the reversed graph.
> 2. Compute in-degrees on the **reversed** graph.
> 3. Push every node with reversed in-degree `0` into a queue (these are exactly the original terminal nodes).
> 4. Run standard Kahn's algorithm on the reversed graph: pop a node, mark it safe, decrement the reversed in-degree of its reversed neighbors, push any that hit `0`.
> 5. Collect and **sort** all nodes marked safe — that's the answer.

## 👨‍💻 Code

### Java

```java
import java.util.*;

public class EventualSafeStatesBFS {

    public List<Integer> eventualSafeNodes(int n, List<List<Integer>> adj) {
        List<List<Integer>> reverseAdj = new ArrayList<>();
        for (int i = 0; i < n; i++) reverseAdj.add(new ArrayList<>());

        int[] inDegree = new int[n];
        for (int u = 0; u < n; u++) {
            for (int v : adj.get(u)) {
                reverseAdj.get(v).add(u);   // reverse the edge
                inDegree[u]++;              // in-degree on the REVERSED graph
            }
        }

        Queue<Integer> queue = new LinkedList<>();
        for (int i = 0; i < n; i++) {
            if (inDegree[i] == 0) queue.offer(i);   // originally terminal nodes
        }

        boolean[] safe = new boolean[n];
        while (!queue.isEmpty()) {
            int node = queue.poll();
            safe[node] = true;

            for (int neighbor : reverseAdj.get(node)) {
                inDegree[neighbor]--;
                if (inDegree[neighbor] == 0) {
                    queue.offer(neighbor);
                }
            }
        }

        List<Integer> result = new ArrayList<>();
        for (int i = 0; i < n; i++) {
            if (safe[i]) result.add(i);
        }
        return result;   // already ascending since we iterate 0..n-1
    }
}
```

### Python

```python
from collections import deque

class Solution:
    def eventual_safe_nodes(self, n, adj):
        reverse_adj = [[] for _ in range(n)]
        in_degree = [0] * n

        for u in range(n):
            for v in adj[u]:
                reverse_adj[v].append(u)   # reverse the edge
                in_degree[u] += 1           # in-degree on the REVERSED graph

        queue = deque([i for i in range(n) if in_degree[i] == 0])   # originally terminal nodes
        safe = [False] * n

        while queue:
            node = queue.popleft()
            safe[node] = True

            for neighbor in reverse_adj[node]:
                in_degree[neighbor] -= 1
                if in_degree[neighbor] == 0:
                    queue.append(neighbor)

        return [i for i in range(n) if safe[i]]   # already ascending
```

## ⏱️ Complexity Analysis

> [!note]
> - **Time:** O(V + E) — building the reversed graph is O(V + E), and Kahn's algorithm on it is also O(V + E).
> - **Space:** O(V + E) for the reversed adjacency list, plus O(V) for in-degree, the queue, and the `safe` array. Slightly more overhead than the DFS version (which needs no reversed graph), plus O(N log N) if sorting is needed separately (here it's avoided by iterating `0..n-1` in order).

## ⚠️ Common Pitfalls

> [!warning]
> - **Computing in-degree on the original graph instead of the reversed one.** The entire trick only works because reversing the edges turns "out-degree 0" (terminal) into "in-degree 0" (Kahn's starting condition) — skip the reversal and the algorithm starts from the wrong nodes entirely.
> - **Forgetting to build the reversed adjacency list as a *separate* structure** — traversing the original graph "backwards" isn't something you can fake without actually constructing the reverse edges (or a reverse index), since adjacency lists only store outgoing edges by default.
> - **Marking a node safe before it's popped from the queue** (e.g. marking on push instead of pop) — while this happens to work out fine for this specific algorithm since push only happens when in-degree hits 0, it's a common source of bugs if adapted carelessly to other in-degree-based algorithms; keep the mark on pop for consistency with the general Kahn's pattern.
> - **Not sorting the final result** if collecting nodes in a different order (e.g. via a `Set` or non-sequential iteration) — iterating `0..n-1` in order sidesteps this, but any other collection strategy needs an explicit sort before returning.

## 🔗 Related Problems / Next Up

> [!success]
> - **Companion:** [Find Eventual Safe States (DFS)](../02-BFS-DFS-Problems/Eventual-Safe-States-DFS.md) — same result, computed via cycle-detection recursion instead of reversed-graph Kahn's algorithm.
> - **Builds on:** [Topological Sort (BFS / Kahn's Algorithm)](Topological-Sort-BFS-Kahns-Algorithm.md)
> - [↑ Back to Topological Sort Index](00-Topological-Sort-Index.md)
