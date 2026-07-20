---
title: "Shortest Path in a Directed Acyclic Graph (DAG)"
tags: [graph, shortest-path, topological-sort, dfs, dag, weighted-graph]
videoLink: "https://www.youtube.com/playlist?list=PLgUwDviBIf0oE3gA41TKO2H5bHpPd7fzn"
difficulty: Hard
---

# Shortest Path in a Directed Acyclic Graph (DAG)

## 🎯 What problem are we solving?

> [!question]
> Given a **weighted DAG** (directed, acyclic, with edge weights that may not all be equal), find the shortest distance from a given source node to **every** other node.

## 💡 Intuition

> [!tip]
> Dijkstra's or Bellman-Ford would both work here, but a DAG has a special structure that allows something much cheaper: **topological order**. A topological sort guarantees that by the time you "arrive" at any node during a left-to-right scan of that order, **every node that could possibly shorten the path to it has already been finalized** — because a DAG has no cycles, nothing "ahead" in the topo order can ever point back to something "behind."
>
> So the algorithm is: **first compute a topological sort of the DAG**, then process nodes **in that exact order**, "relaxing" every outgoing edge as you go — i.e., for each neighbor, checking if going through the current node gives a shorter distance than what's currently recorded, and updating if so. Since nodes are handled in dependency order, once you reach a node in the topo sort, its recorded distance is already guaranteed to be final and correct — no future relaxation could ever improve it further, because nothing earlier in the topo order still needs to be processed.

## 🖼️ Visualizing it

```
Source = 6. Edges (u, v, weight): 6→4(2), 6→5(3), 5→4(1), 5→3(3), 4→3(5), 4→2(1), 3→2(2), 3→1(3), 2→0(3), 0→1(2), 1→3(1)
```

```
Step 1: Topological sort (DFS-based) -> 6, 5, 4, 2, 0, 1, 3

Step 2: distance[6] = 0, everyone else = infinity. Process in topo order:

pop 6 (dist 0): relax 6->4 (0+2=2 < inf) -> dist[4]=2
                relax 6->5 (0+3=3 < inf) -> dist[5]=3
pop 5 (dist 3): relax 5->4 (3+1=4, but dist[4] is already 2) -> skip, not better
                relax 5->3 (3+3=6 < inf) -> dist[3]=6
pop 4 (dist 2): relax 4->3 (2+5=7, but dist[3] is 6) -> skip
                relax 4->2 (2+1=3 < inf) -> dist[2]=3
pop 2 (dist 3): relax 2->0 (3+3=6 < inf) -> dist[0]=6
pop 0 (dist 6): relax 0->1 (6+2=8 < inf) -> dist[1]=8
pop 1 (dist 8): relax 1->3 (8+1=9, but dist[3] is 6) -> skip
pop 3 (dist 6): no outgoing edges
```

**Final distances from 6:** `0:6, 1:8, 2:3, 3:6, 4:2, 5:3, 6:0` — each one finalized the moment its node was popped, since every predecessor that could improve it had already been processed.

## 🛠️ Algorithm / Approach

> [!abstract]
> 1. Build the adjacency list storing `(neighbor, weight)` pairs.
> 2. Compute a topological sort of the DAG (DFS-based: push to a stack on finishing each node, then pop for the order).
> 3. Initialize `distance[]` to infinity for all nodes, `distance[source] = 0`.
> 4. Process nodes **in topological order**: for the current node (skip if its distance is still infinity — unreachable), relax every outgoing edge: if `distance[node] + weight < distance[neighbor]`, update `distance[neighbor]`.
> 5. Return the `distance` array.

## 👨‍💻 Code

### Java

```java
import java.util.*;

public class ShortestPathDAG {

    private void dfs(int node, List<List<int[]>> adj, boolean[] visited, Deque<Integer> stack) {
        visited[node] = true;
        for (int[] edge : adj.get(node)) {
            int neighbor = edge[0];
            if (!visited[neighbor]) dfs(neighbor, adj, visited, stack);
        }
        stack.push(node);
    }

    public int[] shortestPath(int n, int source, List<List<int[]>> adj) {
        boolean[] visited = new boolean[n];
        Deque<Integer> stack = new ArrayDeque<>();

        for (int i = 0; i < n; i++) {
            if (!visited[i]) dfs(i, adj, visited, stack);
        }

        int[] distance = new int[n];
        Arrays.fill(distance, Integer.MAX_VALUE);
        distance[source] = 0;

        while (!stack.isEmpty()) {
            int node = stack.pop();
            if (distance[node] != Integer.MAX_VALUE) {
                for (int[] edge : adj.get(node)) {
                    int neighbor = edge[0], weight = edge[1];
                    if (distance[node] + weight < distance[neighbor]) {
                        distance[neighbor] = distance[node] + weight;
                    }
                }
            }
        }
        return distance;
    }
}
```

### Python

```python
class Solution:
    def shortest_path(self, n, source, adj):
        visited = [False] * n
        stack = []

        def dfs(node):
            visited[node] = True
            for neighbor, _ in adj[node]:
                if not visited[neighbor]:
                    dfs(neighbor)
            stack.append(node)

        for i in range(n):
            if not visited[i]:
                dfs(i)

        distance = [float('inf')] * n
        distance[source] = 0

        while stack:
            node = stack.pop()
            if distance[node] != float('inf'):
                for neighbor, weight in adj[node]:
                    if distance[node] + weight < distance[neighbor]:
                        distance[neighbor] = distance[node] + weight

        return distance
```

## ⏱️ Complexity Analysis

> [!note]
> - **Time:** O(V + E) — topological sort is O(V + E), and relaxing every edge exactly once during the second pass is also O(V + E). Notably faster than Dijkstra's O((V + E) log V) since no priority queue is needed.
> - **Space:** O(V) for `visited`, the stack, and the `distance` array, plus O(V) recursion stack for the DFS.

## ⚠️ Common Pitfalls

> [!warning]
> - **Skipping the "is this node reachable" check before relaxing its edges.** If `distance[node]` is still infinity, relaxing its outgoing edges would incorrectly propagate an "infinity + weight" overflow/garbage value — always guard against processing unreachable nodes.
> - **Using this technique on a graph with a cycle.** This algorithm's correctness depends entirely on the DAG property — a cyclic graph has no valid topological order, so the whole "process in dependency order" guarantee breaks down.
> - **Forgetting this only works because edge weights can be arbitrary (not necessarily positive)** — unlike Dijkstra's, this technique handles negative weights correctly too, precisely because it never revisits a node once processed (topological order guarantees finality), sidestepping the assumption Dijkstra's relies on.
> - **Reaching for Dijkstra's by default.** On a DAG specifically, this topo-sort-based approach is strictly simpler and faster — recognize the DAG structure before reaching for a heavier general-purpose algorithm.

## 🔗 Related Problems / Next Up

> [!success]
> - **Builds on:** [Topological Sort (DFS)](../03-Topological-Sort/Topological-Sort-DFS.md)
> - **Contrast with:** Dijkstra's algorithm (covered next) — needed when the graph isn't a DAG (has cycles), where topological order doesn't exist
> - [↑ Back to Shortest Path Algorithms Index](00-Shortest-Path-Index.md)
