---
title: "Shortest Path in an Undirected Graph with Unit Weights"
tags: [graph, shortest-path, bfs, unweighted-graph]
videoLink: "https://www.youtube.com/playlist?list=PLgUwDviBIf0oE3gA41TKO2H5bHpPd7fzn"
difficulty: Medium
---

# Shortest Path in an Undirected Graph with Unit Weights

## 🎯 What problem are we solving?

> [!question]
> Given an **undirected** graph where every edge has the same weight (unit weight, i.e. `1`), and a source node, find the shortest distance from the source to **every** other node. If a node is unreachable, report `-1` for it.

## 💡 Intuition

> [!tip]
> When every edge costs exactly the same, **plain BFS already computes shortest distances** — no need for Dijkstra's or any priority queue. BFS naturally explores the graph level by level: everything at "hop distance 1" from the source is discovered before anything at "hop distance 2," which is discovered before "hop distance 3," and so on. Since every edge costs the same unit weight, hop distance *is* shortest distance.
>
> The only twist versus a plain BFS traversal: instead of a `visited` boolean array, keep a **distance array** initialized to infinity. When popping a node and examining a neighbor, if going through the current node offers a **shorter** distance than what's currently recorded for that neighbor (`distance[node] + 1 < distance[neighbor]`), update it and push the neighbor into the queue. Because BFS processes nodes in strictly non-decreasing distance order, the very first time a node's distance is set, it's already optimal — but keeping the explicit `<` comparison (rather than a one-time visited check) makes the logic robust and mirrors the general relaxation pattern used in weighted shortest-path algorithms.

## 🖼️ Visualizing it

```
Source = 0. Undirected edges (unit weight): 0-1, 0-3, 1-2, 1-3, 3-4, 2-6, 4-5, 4-6, 6-7, 6-8, 5-6, 5-8, 7-8
```

```
distance[0] = 0, everyone else = infinity. queue = [(0, 0)]

pop (0, 0): neighbors 1, 3 -> distance 0+1=1, both better than infinity -> distance[1]=1, distance[3]=1
            push (1,1), (3,1)

pop (1, 1): neighbors 0, 2, 3
            0: 1+1=2, but distance[0]=0 already better -> skip
            2: 1+1=2, better than infinity -> distance[2]=2, push (2,2)
            3: 1+1=2, but distance[3]=1 already better -> skip

pop (3, 1): neighbors 0, 1, 4
            0, 1 already better -> skip
            4: 1+1=2, better than infinity -> distance[4]=2, push (4,2)

pop (2, 2): neighbor 6 -> distance 2+1=3, better than infinity -> distance[6]=3, push (6,3)

pop (4, 2): neighbors 3, 5, 6
            3 already better -> skip
            5: 2+1=3, better than infinity -> distance[5]=3, push (5,3)
            6: 2+1=3, but distance[6]=3 already equal (not strictly better) -> skip

... continues until the queue is empty, filling in the remaining distances.
```

Every distance gets finalized the first time it's reached, because BFS explores strictly in order of increasing hop-count.

## 🛠️ Algorithm / Approach

> [!abstract]
> 1. Build an undirected adjacency list (add both directions for each edge).
> 2. Initialize a `distance` array to infinity for all nodes; set `distance[source] = 0`.
> 3. Push `source` into a queue.
> 4. While the queue isn't empty: pop `node`. For each `neighbor` of `node`: if `distance[node] + 1 < distance[neighbor]`, update `distance[neighbor]` and push `neighbor` into the queue.
> 5. Build the final answer: for each node, if its distance is still infinity, output `-1` (unreachable); otherwise output the computed distance.

## 👨‍💻 Code

### Java

```java
import java.util.*;

public class ShortestPathUnitWeights {

    public int[] shortestPath(int n, int[][] edges, int source) {
        List<List<Integer>> adj = new ArrayList<>();
        for (int i = 0; i < n; i++) adj.add(new ArrayList<>());
        for (int[] edge : edges) {
            adj.get(edge[0]).add(edge[1]);
            adj.get(edge[1]).add(edge[0]);   // undirected
        }

        int[] distance = new int[n];
        Arrays.fill(distance, Integer.MAX_VALUE);
        distance[source] = 0;

        Queue<Integer> queue = new LinkedList<>();
        queue.offer(source);

        while (!queue.isEmpty()) {
            int node = queue.poll();
            for (int neighbor : adj.get(node)) {
                if (distance[node] + 1 < distance[neighbor]) {
                    distance[neighbor] = distance[node] + 1;
                    queue.offer(neighbor);
                }
            }
        }

        int[] result = new int[n];
        for (int i = 0; i < n; i++) {
            result[i] = distance[i] == Integer.MAX_VALUE ? -1 : distance[i];
        }
        return result;
    }
}
```

### Python

```python
from collections import deque

class Solution:
    def shortest_path(self, n, edges, source):
        adj = [[] for _ in range(n)]
        for u, v in edges:
            adj[u].append(v)
            adj[v].append(u)   # undirected

        distance = [float('inf')] * n
        distance[source] = 0

        queue = deque([source])
        while queue:
            node = queue.popleft()
            for neighbor in adj[node]:
                if distance[node] + 1 < distance[neighbor]:
                    distance[neighbor] = distance[node] + 1
                    queue.append(neighbor)

        return [d if d != float('inf') else -1 for d in distance]
```

## ⏱️ Complexity Analysis

> [!note]
> - **Time:** O(V + 2E) — standard BFS traversal cost on an undirected graph.
> - **Space:** O(V) for the `distance` array and the queue, plus O(V + 2E) for the adjacency list.

## ⚠️ Common Pitfalls

> [!warning]
> - **Using a `visited` boolean array instead of a `distance` array.** While a plain visited check technically works for unit-weight BFS (the first visit is always shortest), the distance-array-with-relaxation pattern generalizes better and matches the mental model needed for weighted shortest-path algorithms later — build the right habit early.
> - **Forgetting this only works because all edges have equal weight.** The moment edges have different weights, plain BFS no longer guarantees shortest distances — that's exactly when Dijkstra's algorithm becomes necessary.
> - **Not initializing unreachable nodes to `-1` in the final output.** A node whose distance stayed at infinity throughout the BFS was never reached — report that clearly instead of leaving a sentinel/garbage value.
> - **Forgetting to add edges in both directions** since the graph is undirected — adding only `u → v` and not `v → u` silently turns it into a directed graph and produces wrong distances.

## 🔗 Related Problems / Next Up

> [!success]
> - **Builds on:** [G4. BFS Traversal](../01-Foundations/G4-BFS-Traversal.md)
> - **Contrast with:** [Shortest Path in a DAG](Shortest-Path-DAG.md) — same relaxation idea, but processed in topological order instead of BFS order, to handle non-uniform edge weights
> - [↑ Back to Shortest Path Algorithms Index](00-Shortest-Path-Index.md)
