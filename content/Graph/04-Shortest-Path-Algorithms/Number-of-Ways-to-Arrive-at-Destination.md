---
title: "Number of Ways to Arrive at the Destination"
tags: [graph, shortest-path, dijkstra, counting, weighted-graph]
videoLink: "https://www.youtube.com/playlist?list=PLgUwDviBIf0oE3gA41TKO2H5bHpPd7fzn"
difficulty: Hard
---

# Number of Ways to Arrive at the Destination

## 🎯 What problem are we solving?

> [!question]
> Given an undirected weighted graph of city intersections `0` to `n-1`, count the **number of distinct shortest paths** from intersection `0` to intersection `n-1`, modulo `10^9 + 7`.

## 💡 Intuition

> [!tip]
> The tempting-but-wrong approach: whenever a node is reached with a distance tying the current best, just increment a counter by `1`. This breaks the moment a node *itself* was reachable via multiple equally-short paths — that multiplicity has to **propagate forward**. If node `X` can be reached in 2 distinct shortest ways, and `X` connects onward to `Y` along an edge that also lies on `Y`'s shortest path, then `Y` inherits **both** of `X`'s ways, not just one.
>
> The real quantity needed is `ways[node]` — "the number of distinct shortest paths from the source to this specific node" — and it composes additively across predecessors. Run Dijkstra as normal, but maintain a parallel `ways[]` array, `ways[source] = 1` (there's exactly one way to "reach" the source: don't move), everyone else `0`. During relaxation, for edge `node → neighbor`:
> - If a **strictly shorter** distance is found: this is a brand-new shortest route — **overwrite**: `ways[neighbor] = ways[node]` (discard whatever was counted through the now-obsolete longer path).
> - If an **equally short** distance is found (a tie with the current best): this is an *additional* shortest route — **accumulate**: `ways[neighbor] += ways[node]`.

## 🖼️ Visualizing it

```
Two predecessors A and B both lie on shortest paths into node X, with ways[A]=2 and ways[B]=3.
X inherits ways[X] = 2 + 3 = 5 -- five distinct shortest routes converge at X.

If X then connects onward to Y along an edge that's part of Y's shortest path too,
Y inherits ways[Y] = 5 as well (assuming this is Y's only shortest-path predecessor) --
even though Y is being reached "for the first time" with a fresh shortest distance, it
already carries a way-count greater than 1, inherited entirely from X's own multiplicity.
```

This is exactly why simply counting "how many predecessors does the destination have" is insufficient — the count has to recursively account for how multiply-reachable *those* predecessors were, all the way back to the source.

## 🛠️ Algorithm / Approach

> [!abstract]
> 1. Build a weighted undirected adjacency list.
> 2. Initialize `distance[]` to infinity except `distance[source] = 0`; initialize `ways[]` to `0` except `ways[source] = 1`.
> 3. Push `(0, source)` into a priority queue.
> 4. Pop `(d, node)`; for each `(neighbor, weight)` edge: `newDist = d + weight`.
>    - If `newDist < distance[neighbor]`: update `distance[neighbor] = newDist`, **overwrite** `ways[neighbor] = ways[node]`, and push `(newDist, neighbor)`.
>    - Else if `newDist == distance[neighbor]`: **accumulate** `ways[neighbor] = (ways[neighbor] + ways[node]) % MOD` (no need to re-push — this node's shortest distance was already finalized and queued earlier; only the way-count needs updating).
> 5. Return `ways[n-1] % MOD`.

## 👨‍💻 Code

### Java

```java
import java.util.*;

public class NumberOfWaysToArrive {

    private static final int MOD = 1_000_000_007;

    public int countPaths(int n, int[][] roads) {
        List<List<int[]>> adj = new ArrayList<>();
        for (int i = 0; i < n; i++) adj.add(new ArrayList<>());
        for (int[] r : roads) {
            adj.get(r[0]).add(new int[]{r[1], r[2]});
            adj.get(r[1]).add(new int[]{r[0], r[2]});
        }

        long[] distance = new long[n];
        long[] ways = new long[n];
        Arrays.fill(distance, Long.MAX_VALUE);
        distance[0] = 0;
        ways[0] = 1;

        PriorityQueue<long[]> pq = new PriorityQueue<>((a, b) -> Long.compare(a[0], b[0]));
        pq.offer(new long[]{0, 0});

        while (!pq.isEmpty()) {
            long[] top = pq.poll();
            long d = top[0];
            int node = (int) top[1];

            for (int[] edge : adj.get(node)) {
                int neighbor = edge[0], weight = edge[1];
                long newDist = d + weight;

                if (newDist < distance[neighbor]) {
                    distance[neighbor] = newDist;
                    ways[neighbor] = ways[node];
                    pq.offer(new long[]{newDist, neighbor});
                } else if (newDist == distance[neighbor]) {
                    ways[neighbor] = (ways[neighbor] + ways[node]) % MOD;
                }
            }
        }
        return (int) (ways[n - 1] % MOD);
    }
}
```

### Python

```python
import heapq

class Solution:
    def count_paths(self, n, roads):
        MOD = 10**9 + 7
        adj = [[] for _ in range(n)]
        for u, v, w in roads:
            adj[u].append((v, w))
            adj[v].append((u, w))

        distance = [float('inf')] * n
        ways = [0] * n
        distance[0] = 0
        ways[0] = 1

        pq = [(0, 0)]

        while pq:
            d, node = heapq.heappop(pq)

            for neighbor, weight in adj[node]:
                new_dist = d + weight

                if new_dist < distance[neighbor]:
                    distance[neighbor] = new_dist
                    ways[neighbor] = ways[node]
                    heapq.heappush(pq, (new_dist, neighbor))
                elif new_dist == distance[neighbor]:
                    ways[neighbor] = (ways[neighbor] + ways[node]) % MOD

        return ways[n - 1] % MOD
```

## ⏱️ Complexity Analysis

> [!note]
> - **Time:** O(E log V) — identical to standard Dijkstra; way-counting adds only O(1) extra work per relaxation.
> - **Space:** O(V + E) for the adjacency list, plus O(V) for `distance` and `ways`.

## ⚠️ Common Pitfalls

> [!warning]
> - **The single most common mistake:** naively counting "how many predecessors does the destination have" without recursively accounting for how many ways *those* predecessors themselves were reached — this undercounts whenever any earlier node in the graph had multiple equally-short incoming paths.
> - **Using `<=` instead of separate `<` and `==` branches.** `<` must **overwrite** `ways` (discarding stale counts from a now-suboptimal path), while `==` must **accumulate** — collapsing these into one careless comparison either double-counts or silently loses valid paths.
> - **Forgetting to apply the modulo on every addition**, not just at the final return — `ways[]` values can grow very large across many nodes before the answer is read, risking overflow in fixed-width integer types.
> - **Re-pushing a node into the priority queue on the `==` (tie) branch.** Unnecessary — that node's shortest distance was already finalized and queued during the earlier `<` update; only its `ways` value needs adjusting, not another heap entry.

## 🔗 Related Problems / Next Up

> [!success]
> - **Builds directly on:** [Dijkstra's Algorithm (Using a Priority Queue)](Dijkstra-Algorithm-Using-Priority-Queue.md)
> - **Contrast:** plain shortest-distance Dijkstra vs. this variant, which propagates a second piece of state (`ways[]`) alongside `distance[]`
> - This completes the Shortest Path Algorithms section — up next: [05 · MST & Disjoint Set](../05-MST-and-Disjoint-Set/00-MST-Disjoint-Set-Index.md)
> - [↑ Back to Shortest Path Algorithms Index](00-Shortest-Path-Index.md)
