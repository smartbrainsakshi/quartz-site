---
title: "Cheapest Flights Within K Stops"
tags: [graph, shortest-path, bfs, constrained-shortest-path, bellman-ford-style]
videoLink: "https://www.youtube.com/playlist?list=PLgUwDviBIf0oE3gA41TKO2H5bHpPd7fzn"
difficulty: Hard
---

# Cheapest Flights Within K Stops

## 🎯 What problem are we solving?

> [!question]
> Given `n` cities and a list of directed flights `(from, to, price)`, find the cheapest price to fly from `src` to `dst` using **at most `k` stops** (intermediate cities — the source and destination don't count as stops). Return `-1` if no such route exists within the stop budget.

## 💡 Intuition

> [!tip]
> This looks like a job for Dijkstra's (source, destination, cheapest = shortest) — but Dijkstra's core assumption is exactly what breaks here. Dijkstra's greedily **finalizes** a node's distance the moment it's popped with the smallest pending cost, and never revisits it. But here, a node reached *cheaply* might have used up too many stops to have any budget left for continuing onward — while a *more expensive* arrival at that same node might still have stops to spare and could legally continue to the destination. Cost and stop-count don't trade off cleanly enough for Dijkstra's "cheapest always wins, permanently" rule to hold.
>
> The fix: **stop ordering by cost entirely — order by number of stops used instead.** Since stops increase in perfectly uniform `+1` steps (same structural insight as unit-weight BFS problems), a **plain FIFO queue** — not a priority queue — storing `(stops, node, costSoFar)` naturally processes everything with `stops` stops before anything with `stops + 1`. This is closer to a bounded-level Bellman-Ford than true Dijkstra. Critically: **different arrivals at the same node with different remaining stop budgets must all be allowed to proceed independently** — a node must never be "locked out" just because it was reached more cheaply by some other, stop-costlier path.

## 🖼️ Visualizing it

```
Cities 0-3. Directed edges (price): 0->1(100), 1->2(100), 2->0(100), 1->3(600), 2->3(200)
src=0, dst=3, k=1 (at most 1 stop)
```

```
queue = [(stops=0, node=0, cost=0)]

pop (0, 0, 0): stops (0) <= k (1), OK to expand
  edge 0->1(100): newCost=100 < distance[1]=inf -> update distance[1]=100, push (1, 1, 100)

pop (1, 1, 100): stops (1) <= k (1), OK to expand
  edge 1->2(100): newCost=200 < distance[2]=inf -> update, push (2, 2, 200)
    BUT stops for this push would be 2, which EXCEEDS k=1 -- this path is only used for
    comparison bookkeeping, it will be skipped once popped since stops > k
  edge 1->3(600): newCost=700 < distance[3]=inf -> update distance[3]=700, push (2, 3, 700)

pop (2, 2, 200): stops (2) > k (1) -- do not expand further, this path used too many stops

pop (2, 3, 700): this IS the destination -- but we don't special-case-stop here, distance[3]
  is already recorded as 700
```

**Result: 700** via `0 → 1 → 3` (1 stop, exactly at the budget). The cheaper 400-cost route `0 → 1 → 2 → 3` uses 2 stops, which exceeds `k=1`, so it's correctly excluded.

## 🛠️ Algorithm / Approach

> [!abstract]
> 1. Build a **directed** weighted adjacency list.
> 2. Initialize `distance[]` to infinity except `distance[source] = 0` (used only as a pruning bound, not as a Dijkstra-style finalized value). Push `(stops=0, source, cost=0)` into a **plain queue**.
> 3. While the queue isn't empty: pop `(stops, node, cost)`. If `stops > k`, skip — don't expand further.
> 4. Otherwise, for each `(neighbor, price)` edge from `node`: `newCost = cost + price`. If `newCost < distance[neighbor]`, update `distance[neighbor] = newCost` and push `(stops + 1, neighbor, newCost)`.
> 5. Return `distance[destination]` if it's not infinity, else `-1`.

## 👨‍💻 Code

### Java

```java
import java.util.*;

public class CheapestFlightsWithinKStops {

    public int findCheapestPrice(int n, int[][] flights, int src, int dst, int k) {
        List<List<int[]>> adj = new ArrayList<>();
        for (int i = 0; i < n; i++) adj.add(new ArrayList<>());
        for (int[] f : flights) adj.get(f[0]).add(new int[]{f[1], f[2]});

        int[] distance = new int[n];
        Arrays.fill(distance, Integer.MAX_VALUE);
        distance[src] = 0;

        Queue<int[]> queue = new LinkedList<>();
        queue.offer(new int[]{0, src, 0});   // {stops, node, cost}

        while (!queue.isEmpty()) {
            int[] top = queue.poll();
            int stops = top[0], node = top[1], cost = top[2];

            if (stops > k) continue;

            for (int[] edge : adj.get(node)) {
                int neighbor = edge[0], price = edge[1];
                int newCost = cost + price;

                if (newCost < distance[neighbor]) {
                    distance[neighbor] = newCost;
                    queue.offer(new int[]{stops + 1, neighbor, newCost});
                }
            }
        }
        return distance[dst] == Integer.MAX_VALUE ? -1 : distance[dst];
    }
}
```

### Python

```python
from collections import deque

class Solution:
    def find_cheapest_price(self, n, flights, src, dst, k):
        adj = [[] for _ in range(n)]
        for u, v, price in flights:
            adj[u].append((v, price))

        distance = [float('inf')] * n
        distance[src] = 0

        queue = deque([(0, src, 0)])   # (stops, node, cost)

        while queue:
            stops, node, cost = queue.popleft()

            if stops > k:
                continue

            for neighbor, price in adj[node]:
                new_cost = cost + price
                if new_cost < distance[neighbor]:
                    distance[neighbor] = new_cost
                    queue.append((stops + 1, neighbor, new_cost))

        return distance[dst] if distance[dst] != float('inf') else -1
```

## ⏱️ Complexity Analysis

> [!note]
> - **Time:** roughly O(E × K) — each edge can be relaxed up to `K + 1` times in the worst case, once per stop level.
> - **Space:** O(V + E) for the adjacency list, plus O(V) for `distance` and the queue.

## ⚠️ Common Pitfalls

> [!warning]
> - **Using standard Dijkstra (finalizing nodes by cheapest cost, never revisiting).** This is *the* defining trap of this exact problem — a node reached cheaply but with too many stops must not be allowed to permanently block a costlier-but-within-budget arrival at that same node.
> - **Using a priority queue ordered by cost instead of a plain queue ordered by stops.** This reintroduces the same bug — exploration order must respect the stop-count constraint, not price, since stops (not cost) is what increases in the clean, uniform steps that make ordering-by-level meaningful here.
> - **Forgetting to stop expanding once `stops > k`.** Continuing wastes time exploring paths that can never be valid answers.
> - **Returning an infinity sentinel directly instead of `-1`** when no valid route exists within the stop budget.

## 🔗 Related Problems / Next Up

> [!success]
> - **Contrast with:** [Dijkstra's Algorithm (Using a Priority Queue)](Dijkstra-Algorithm-Using-Priority-Queue.md) — this is the classic "why does Dijkstra fail here" interview follow-up
> - **Conceptually related to:** Bellman-Ford's layer-by-layer relaxation (covered later in the series)
> - [↑ Back to Shortest Path Algorithms Index](00-Shortest-Path-Index.md)
