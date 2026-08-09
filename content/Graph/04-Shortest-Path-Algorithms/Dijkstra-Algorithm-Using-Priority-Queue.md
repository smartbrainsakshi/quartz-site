---
title: "Dijkstra's Algorithm (Using a Priority Queue)"
tags: [graph, shortest-path, dijkstra, priority-queue, weighted-graph]
videoLink: "https://www.youtube.com/playlist?list=PLgUwDviBIf0oE3gA41TKO2H5bHpPd7fzn"
difficulty: Medium
---

# Dijkstra's Algorithm (Using a Priority Queue)

## 🎯 What problem are we solving?

> [!question]
> Given a weighted graph (all edge weights **non-negative**, no negative-weight cycles) and a source node, find the shortest distance from that source to **every** other node.

## 💡 Intuition

> [!tip]
> This is a **greedy** algorithm: always expand the node whose currently-known distance is smallest among everything left to process. The reasoning for why this is safe: once a node is popped with the smallest pending distance, **no future discovery could ever produce a shorter path to it** — any other path would have to go through some node with a distance ≥ the one just finalized, which can only make things worse (since weights are non-negative). So that popped distance is locked in as final.
>
> To always get the smallest pending distance next, use a **min-heap (priority queue)** storing `(distance, node)` pairs. Start with `(0, source)`. Repeatedly pop the smallest, then **relax** every neighbor: if `distance[node] + edgeWeight < distance[neighbor]`, that's a newly discovered shorter path — update `distance[neighbor]` and push `(newDistance, neighbor)` into the heap.

## 🖼️ Visualizing it

```
Source = 0. Edges (undirected, weight): 0-1(4), 0-2(4), 1-2(2), 2-3(3), 2-4(1), 2-5(6), 3-5(2), 4-5(3)
```

```
distance = {0:0, others: inf}. pq = [(0,0)]

pop (0,0): relax 0-1 -> dist[1]=4, push (4,1)
           relax 0-2 -> dist[2]=4, push (4,2)

pop (4,1): relax 1-0 -> 4+4=8, not better, skip
           relax 1-2 -> 4+2=6, not better than dist[2]=4, skip

pop (4,2): relax 2-0 -> not better, skip
           relax 2-1 -> not better, skip
           relax 2-3 -> 4+3=7 < inf -> dist[3]=7, push (7,3)
           relax 2-4 -> 4+1=5 < inf -> dist[4]=5, push (5,4)
           relax 2-5 -> 4+6=10 < inf -> dist[5]=10, push (10,5)

pop (5,4): relax 4-2 -> not better, skip
           relax 4-5 -> 5+3=8 < 10 -> dist[5]=8, push (8,5)

pop (7,3): relax 3-2 -> not better, skip
           relax 3-5 -> 7+2=9, not better than dist[5]=8, skip

pop (8,5): relax 5-2, 5-3, 5-4 -> none improve, all worse

pop (10,5): stale entry -- distance[5] is already 8, nothing changes

queue empty.
```

**Final distances from 0:** `0:0, 1:4, 2:4, 3:7, 4:5, 5:8` — every value locked in the instant its (distance, node) pair was popped as the smallest in the heap.

## 🛠️ Algorithm / Approach

> [!abstract]
> 1. Build an adjacency list storing `(neighbor, weight)` pairs.
> 2. Initialize `distance[]` to infinity for all nodes except `distance[source] = 0`.
> 3. Push `(0, source)` into a min-heap.
> 4. While the heap isn't empty: pop `(d, node)`. For each `(neighbor, weight)` edge from `node`: if `d + weight < distance[neighbor]`, update `distance[neighbor] = d + weight` and push `(distance[neighbor], neighbor)`.
> 5. Return the `distance` array.

## 👨‍💻 Code

### Java

```java
import java.util.*;

public class DijkstraPriorityQueue {

    public int[] dijkstra(int n, int source, List<List<int[]>> adj) {
        PriorityQueue<int[]> pq = new PriorityQueue<>((a, b) -> a[0] - b[0]);
        int[] distance = new int[n];
        Arrays.fill(distance, Integer.MAX_VALUE);
        distance[source] = 0;
        pq.offer(new int[]{0, source});

        while (!pq.isEmpty()) {
            int[] top = pq.poll();
            int d = top[0], node = top[1];

            for (int[] edge : adj.get(node)) {
                int neighbor = edge[0], weight = edge[1];
                if (d + weight < distance[neighbor]) {
                    distance[neighbor] = d + weight;
                    pq.offer(new int[]{distance[neighbor], neighbor});
                }
            }
        }
        return distance;
    }
}
```

### Python

```python
import heapq

class Solution:
    def dijkstra(self, n, source, adj):
        distance = [float('inf')] * n
        distance[source] = 0
        pq = [(0, source)]

        while pq:
            d, node = heapq.heappop(pq)

            for neighbor, weight in adj[node]:
                if d + weight < distance[neighbor]:
                    distance[neighbor] = d + weight
                    heapq.heappush(pq, (distance[neighbor], neighbor))

        return distance
```

## ⏱️ Complexity Analysis

> [!note]
> - **Time:** O(E log V) — see the dedicated [Why Priority Queue & Time Complexity](Dijkstra-Why-Priority-Queue-and-Time-Complexity.md) note for the full derivation.
> - **Space:** O(V) for `distance`, plus up to O(E) entries in the heap in the worst case.

## ⚠️ Common Pitfalls

> [!warning]
> - **Using this on a graph with negative edge weights.** Dijkstra's assumes a finalized distance never improves — a negative edge can always find a "better" path by looping back through it again, causing an **infinite loop** (demonstrated directly in the source video with a 2-node graph and a single `-2` edge).
> - **Not skipping stale heap entries.** A node can be pushed multiple times with different distances before its true shortest distance is popped; processing a stale (larger) entry is harmless for correctness (the relaxation check will simply fail to improve anything) but wastes work — adding `if (d > distance[node]) continue;` right after popping is a common optimization.
> - **Forgetting to initialize `distance[]` to infinity** for all non-source nodes before starting.
> - **Assuming a plain FIFO queue works just as well.** It produces the same correct answer, but far less efficiently — see [Why Priority Queue & Time Complexity](Dijkstra-Why-Priority-Queue-and-Time-Complexity.md) for exactly why.

## 🔗 Related Problems / Next Up

> [!success]
> - **Alternative implementation:** [Dijkstra's Algorithm (Using a Set)](Dijkstra-Algorithm-Using-Set.md)
> - **Deep dive:** [Why Priority Queue & Time Complexity Derivation](Dijkstra-Why-Priority-Queue-and-Time-Complexity.md)
> - **Contrast with:** [Shortest Path in a DAG](Shortest-Path-DAG.md) — no heap needed there, since topological order substitutes for greedy ordering
> - [↑ Back to Shortest Path Algorithms Index](00-Shortest-Path-Index.md)
