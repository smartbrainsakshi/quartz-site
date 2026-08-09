---
title: "Print the Shortest Path (Dijkstra with Path Reconstruction)"
tags: [graph, shortest-path, dijkstra, path-reconstruction, weighted-graph]
videoLink: "https://www.youtube.com/playlist?list=PLgUwDviBIf0oE3gA41TKO2H5bHpPd7fzn"
difficulty: Hard
---

# Print the Shortest Path (Dijkstra with Path Reconstruction)

## 🎯 What problem are we solving?

> [!question]
> Given a weighted undirected graph with `n` vertices (1-indexed) and `m` edges, source fixed at `1` and destination fixed at `n`, **print the actual shortest path** (the sequence of nodes), not just its total distance. Return a list containing only `-1` if no path exists.

## 💡 Intuition

> [!tip]
> Dijkstra's already computes the shortest *distance* to every node — the missing piece is remembering *how* each node achieved that distance. Add a **`parent[]` array**: every time a node's distance is improved during relaxation, record `parent[neighbor] = node` — "the last hop before reaching `neighbor`, on its current-best path."
>
> Once Dijkstra finishes, reconstruct the path by walking **backward** from the destination: look up `parent[destination]`, then `parent[parent[destination]]`, and so on, until reaching a node that is its own parent (the source, whose `parent[]` entry is initialized to point to itself as a stopping sentinel). Collect nodes along this backward walk, then **reverse** the collected list to get the path in forward (source-to-destination) order.

## 🖼️ Visualizing it

```
n=5, source=1, destination=5. Edges: 1-2(2), 1-4(1), 4-3(3), 2-3(4), 2-5(5), 3-5(1)
```

```
distance: 1:0, 2:2 (via 1), 4:1 (via 1), 3:4 (via 4: 1+3), 5:5 (via 3: 4+1, better than via 2: 2+5=7)

parent: parent[1]=1 (self, sentinel)
        parent[2]=1
        parent[4]=1
        parent[3]=4
        parent[5]=3

Reconstruction, starting at 5:
  node=5: parent[5]=3, not self -> append 5, move to 3
  node=3: parent[3]=4, not self -> append 3, move to 4
  node=4: parent[4]=1, not self -> append 4, move to 1
  node=1: parent[1]=1, IS self -> stop, append 1 manually

collected (backward order): [5, 3, 4, 1]
reversed (forward order):   [1, 4, 3, 5]
```

**Result: `1 → 4 → 3 → 5`**, distance 5 — matches the problem's own example.

## 🛠️ Algorithm / Approach

> [!abstract]
> 1. Build a weighted undirected adjacency list (size `n+1` for 1-indexing).
> 2. Initialize `distance[]` to infinity except `distance[source] = 0`; initialize `parent[i] = i` for every node (self-pointing sentinel).
> 3. Run standard Dijkstra with a priority queue. Every time `distance[neighbor]` improves via `node`, also set `parent[neighbor] = node`.
> 4. If `distance[destination]` is still infinity after Dijkstra finishes, return `[-1]` — unreachable.
> 5. Otherwise, reconstruct: start `node = destination`, and while `parent[node] != node`, append `node` to a list and move to `parent[node]`. After the loop, manually append the source (the loop stops *before* including it).
> 6. Reverse the collected list and return it.

## 👨‍💻 Code

### Java

```java
import java.util.*;

public class PrintShortestPath {

    public List<Integer> shortestPath(int n, int m, int[][] edges) {
        List<List<int[]>> adj = new ArrayList<>();
        for (int i = 0; i <= n; i++) adj.add(new ArrayList<>());
        for (int[] e : edges) {
            adj.get(e[0]).add(new int[]{e[1], e[2]});
            adj.get(e[1]).add(new int[]{e[0], e[2]});
        }

        int[] distance = new int[n + 1];
        int[] parent = new int[n + 1];
        Arrays.fill(distance, Integer.MAX_VALUE);
        for (int i = 1; i <= n; i++) parent[i] = i;

        int source = 1, destination = n;
        distance[source] = 0;

        PriorityQueue<int[]> pq = new PriorityQueue<>((a, b) -> a[0] - b[0]);
        pq.offer(new int[]{0, source});

        while (!pq.isEmpty()) {
            int[] top = pq.poll();
            int d = top[0], node = top[1];

            for (int[] edge : adj.get(node)) {
                int neighbor = edge[0], weight = edge[1];
                if (d + weight < distance[neighbor]) {
                    distance[neighbor] = d + weight;
                    parent[neighbor] = node;
                    pq.offer(new int[]{distance[neighbor], neighbor});
                }
            }
        }

        if (distance[destination] == Integer.MAX_VALUE) {
            return List.of(-1);
        }

        List<Integer> path = new ArrayList<>();
        int node = destination;
        while (parent[node] != node) {
            path.add(node);
            node = parent[node];
        }
        path.add(source);
        Collections.reverse(path);
        return path;
    }
}
```

### Python

```python
import heapq

class Solution:
    def shortest_path(self, n, m, edges):
        adj = [[] for _ in range(n + 1)]
        for u, v, w in edges:
            adj[u].append((v, w))
            adj[v].append((u, w))

        distance = [float('inf')] * (n + 1)
        parent = list(range(n + 1))   # self-pointing sentinel

        source, destination = 1, n
        distance[source] = 0
        pq = [(0, source)]

        while pq:
            d, node = heapq.heappop(pq)

            for neighbor, weight in adj[node]:
                if d + weight < distance[neighbor]:
                    distance[neighbor] = d + weight
                    parent[neighbor] = node
                    heapq.heappush(pq, (distance[neighbor], neighbor))

        if distance[destination] == float('inf'):
            return [-1]

        path = []
        node = destination
        while parent[node] != node:
            path.append(node)
            node = parent[node]
        path.append(source)

        return path[::-1]
```

## ⏱️ Complexity Analysis

> [!note]
> - **Time:** O(E log V + N) — standard Dijkstra plus an O(path length) backward walk, bounded by O(N).
> - **Space:** O(N) for `distance` and `parent`, plus O(N) for the reconstructed path.

## ⚠️ Common Pitfalls

> [!warning]
> - **Not initializing `parent[i] = i`.** This self-pointing sentinel is what lets the reconstruction loop know when to stop (having reached the source) — using `-1` or `0` instead risks confusing a sentinel with a real 0-indexed node, or requires an extra explicit source check.
> - **Checking `distance[destination] == infinity` *after* attempting reconstruction instead of before.** If unreachable, `parent[destination]` never got updated and still equals itself — reconstruction would silently "succeed" with a single-node, wrong path unless the infinity check runs first.
> - **Forgetting to manually append the source** after the reconstruction loop — the `while parent[node] != node` condition excludes the source by design, since the loop stops the moment it detects the self-pointing sentinel.
> - **Forgetting to reverse the collected path** before returning — it's built backward (destination to source) and must be flipped to read as a proper source-to-destination route.

## 🔗 Related Problems / Next Up

> [!success]
> - **Builds on:** [Dijkstra's Algorithm (Using a Priority Queue)](Dijkstra-Algorithm-Using-Priority-Queue.md)
> - **Similar parent-tracking pattern:** [Shortest Path in a DAG](Shortest-Path-DAG.md) used topological order instead of a heap, but the same "remember where you came from" idea generalizes across shortest-path variants
> - [↑ Back to Shortest Path Algorithms Index](00-Shortest-Path-Index.md)
