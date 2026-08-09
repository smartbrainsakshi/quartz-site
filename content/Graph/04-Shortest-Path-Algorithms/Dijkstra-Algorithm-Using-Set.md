---
title: "Dijkstra's Algorithm (Using a Set)"
tags: [graph, shortest-path, dijkstra, set, weighted-graph]
videoLink: "https://www.youtube.com/playlist?list=PLgUwDviBIf0oE3gA41TKO2H5bHpPd7fzn"
difficulty: Medium
---

# Dijkstra's Algorithm (Using a Set)

## 🎯 What problem are we solving?

> [!question]
> Same problem as [Dijkstra's Algorithm (Using a Priority Queue)](Dijkstra-Algorithm-Using-Priority-Queue.md) — shortest distance from a source to every node in a non-negative-weight graph — implemented instead with an **ordered set** (e.g. `std::set` / a `TreeSet`).

## 💡 Intuition

> [!tip]
> A set storing unique `(distance, node)` pairs keeps everything sorted ascending, so its smallest element (the "begin" of the set) always gives the same "smallest pending distance" that a min-heap gives — the core greedy property is identical.
>
> The one genuinely new capability a set offers over a heap: **O(log n) erase**. The instant a strictly better distance is found for some node, the algorithm can actively **delete the old, now-obsolete `(oldDistance, node)` entry** from the set — a heap has no efficient way to remove an arbitrary element, so a heap-based implementation just lets stale entries linger and pops them later as harmless-but-wasted work.
>
> Is this actually faster? **Not necessarily** — every erase costs a `log n` operation, so the algorithm is trading "guaranteed extra pops later" for "guaranteed extra erases now." The video is explicit about this: it depends on the graph's structure, and in an interview it's fine to say both are valid O(E log V) approaches with different constant-factor trade-offs.

## 🖼️ Visualizing it

```
Same graph as the priority-queue note. Reusing the moment node 5 gets improved:
```

```
Earlier: distance[5] was updated to 10 via node 2 (4 + 6), so the set contains (10, 5).

Later: node 4 (distance 5) relaxes edge 4-5 (weight 3): candidate = 5 + 3 = 8, which is
better than the current distance[5] = 10.

With a priority queue: just push (8, 5) and leave the stale (10, 5) sitting in the heap
  -- it'll be popped later and discarded as a no-op.

With a set: ERASE (10, 5) from the set first, THEN insert (8, 5). The stale entry never
  gets a chance to be popped at all.
```

Both approaches reach the same final `distance[5] = 8` — the set version just actively removes dead weight instead of tolerating it.

## 🛠️ Algorithm / Approach

> [!abstract]
> 1. Build the adjacency list as usual.
> 2. Initialize `distance[]` to infinity except `distance[source] = 0`.
> 3. Insert `(0, source)` into an ordered set.
> 4. While the set isn't empty: take the **smallest** element (the set's `begin()`), extract `(d, node)`, and **erase it** from the set.
> 5. For each `(neighbor, weight)` edge from `node`: if `d + weight < distance[neighbor]`:
>    - If `distance[neighbor]` is not infinity (i.e. a stale entry actually exists in the set), **erase** `(distance[neighbor], neighbor)` from the set first.
>    - Update `distance[neighbor]` and **insert** the new `(distance[neighbor], neighbor)` pair.
> 6. Return the `distance` array.

## 👨‍💻 Code

### Java

```java
import java.util.*;

public class DijkstraUsingSet {

    public int[] dijkstra(int n, int source, List<List<int[]>> adj) {
        TreeSet<int[]> set = new TreeSet<>((a, b) ->
            a[0] != b[0] ? a[0] - b[0] : a[1] - b[1]);

        int[] distance = new int[n];
        Arrays.fill(distance, Integer.MAX_VALUE);
        distance[source] = 0;
        set.add(new int[]{0, source});

        while (!set.isEmpty()) {
            int[] top = set.pollFirst();
            int d = top[0], node = top[1];

            for (int[] edge : adj.get(node)) {
                int neighbor = edge[0], weight = edge[1];
                if (d + weight < distance[neighbor]) {
                    if (distance[neighbor] != Integer.MAX_VALUE) {
                        set.remove(new int[]{distance[neighbor], neighbor});
                    }
                    distance[neighbor] = d + weight;
                    set.add(new int[]{distance[neighbor], neighbor});
                }
            }
        }
        return distance;
    }
}
```

### Python

```python
# Python's heapq has no O(log n) erase, so the "set" technique doesn't translate
# directly. The practical Python equivalent is lazy deletion: skip a popped
# entry if it's already stale (its distance no longer matches the current best).
import heapq

class Solution:
    def dijkstra(self, n, source, adj):
        distance = [float('inf')] * n
        distance[source] = 0
        pq = [(0, source)]

        while pq:
            d, node = heapq.heappop(pq)
            if d > distance[node]:
                continue   # stale entry -- lazy-deletion equivalent of set.erase

            for neighbor, weight in adj[node]:
                if d + weight < distance[neighbor]:
                    distance[neighbor] = d + weight
                    heapq.heappush(pq, (distance[neighbor], neighbor))

        return distance
```

## ⏱️ Complexity Analysis

> [!note]
> - **Time:** O(E log V) — same asymptotic bound as the priority-queue version; the erase operation doesn't change the overall complexity class.
> - **Space:** O(V) for `distance`, plus at most O(V) live entries in the set at any time (unlike a heap, stale duplicates don't accumulate).

## ⚠️ Common Pitfalls

> [!warning]
> - **Erasing an entry that was never actually inserted.** Only erase `(distance[neighbor], neighbor)` if `distance[neighbor]` is not still infinity — erasing a sentinel/placeholder value that was never in the set is a logic error.
> - **Assuming the set version is strictly faster than the priority-queue version.** It isn't guaranteed — the erase cost and the saved future-pop cost roughly offset in the worst case; both are legitimate O(E log V) implementations.
> - **Trying to directly port this to Python via `heapq`.** Python's standard heap has no efficient arbitrary-element removal — the honest equivalent is the lazy-deletion pattern (check staleness on pop), not a literal translation of the erase-then-insert logic.
> - **Comparator bugs in the ordered set** (e.g. in Java, comparing `int[]` references by identity instead of by value) — a custom comparator ordering by `(distance, node)` is required for correct sorting and correct removal matching.

## 🔗 Related Problems / Next Up

> [!success]
> - **Alternative implementation:** [Dijkstra's Algorithm (Using a Priority Queue)](Dijkstra-Algorithm-Using-Priority-Queue.md)
> - **Deep dive:** [Why Priority Queue & Time Complexity Derivation](Dijkstra-Why-Priority-Queue-and-Time-Complexity.md)
> - [↑ Back to Shortest Path Algorithms Index](00-Shortest-Path-Index.md)
