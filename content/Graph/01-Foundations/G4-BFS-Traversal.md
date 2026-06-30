---
title: "G-4. BFS Traversal"
tags: [graph, foundations, bfs, traversal, queue]
videoLink: "https://www.youtube.com/playlist?list=PLgUwDviBIf0oE3gA41TKO2H5bHpPd7fzn"
---

# G-4. BFS Traversal

## 🎯 What problem are we solving?

> [!question]
> We now have a graph stored (G-2) and know we need to visit every node across every component (G-3). But *in what order* do we actually visit nodes? **BFS (Breadth-First Search)** is the first of the two fundamental traversal orders — and it's the one that explores **level by level**, like ripples spreading out from a starting point.

## 💡 Intuition

> [!tip]
> "Breadth" is the keyword — BFS explores everything at the *current* distance from the start before moving any further out. This is why it's also called **level-order traversal**.
>
> Take a starting node and call it **level 0**. Every node directly connected to it is **level 1**. Every node connected to a level-1 node (that hasn't been visited yet) is **level 2**, and so on. BFS visits all of level 0, then all of level 1, then all of level 2 — never jumping ahead.
>
> > Within a single level, the order you visit nodes in doesn't matter (you could do `2 then 6` or `6 then 2`) — what matters is that **every node from one level is fully processed before any node from the next level**.
>
> Crucially: **the BFS order depends entirely on the starting node.** Change the start, and the whole level structure (and traversal order) changes — even on the exact same graph.

## 🖼️ Visualizing it

Take this graph, starting BFS from node **1**:

```
        1
       / \
      2   6
     /|    |\
    3 4    7 9
      |    |
      5    8
```
(edges: 1-2, 1-6, 2-3, 2-4, 4-5, 6-7, 6-9, 7-8)

**Level 0:** `1`
**Level 1:** `2, 6` (both at distance 1 from node 1)
**Level 2:** `3, 4, 7, 9`
**Level 3:** `5, 8`

**BFS traversal order:** `1 → 2 → 6 → 3 → 4 → 7 → 9 → 5 → 8`

Now restart BFS from node **6** instead — same graph, completely different order:

**Level 0:** `6`
**Level 1:** `1, 7, 9`
**Level 2:** `2, 8`
**Level 3:** `3, 4`

**BFS traversal order:** `6 → 1 → 7 → 9 → 2 → 8 → 3 → 4`

### Mechanics: Queue + Visited Array

BFS is implemented with a **queue** (First-In-First-Out) plus the `visited` array from G-3:

1. **Initial setup:** push the starting node into the queue, mark it visited.
2. **Loop while the queue isn't empty:**
   - Pop the front node, add it to the BFS result.
   - For each of its neighbors (from the adjacency list): if not visited, mark visited **and** push into the queue.

```
Queue: [1]              Visited: {1}             BFS so far: []

Pop 1 → BFS: [1]
  neighbors of 1: 2, 6 → both unvisited → push both, mark visited
  Queue: [2, 6]          Visited: {1,2,6}

Pop 2 → BFS: [1,2]
  neighbors of 2: 1(visited, skip), 3, 4 → push 3, 4
  Queue: [6, 3, 4]       Visited: {1,2,6,3,4}

Pop 6 → BFS: [1,2,6]
  neighbors of 6: 1(skip), 7, 9 → push 7, 9
  Queue: [3, 4, 7, 9]    Visited: {1,2,6,3,4,7,9}

Pop 3 → BFS: [1,2,6,3]   neighbors: 2(skip) → nothing new
Pop 4 → BFS: [1,2,6,3,4] neighbors: 2(skip), 5 → push 5
Pop 7 → BFS: [...,7]     neighbors: 6(skip), 8 → push 8
Pop 9 → BFS: [...,9]     neighbors: 6(skip) → nothing new
Pop 5 → BFS: [...,5]     neighbors: 4(skip) → nothing new
Pop 8 → BFS: [...,8]     neighbors: 7(skip) → nothing new

Queue empty → done.
Final BFS: 1, 2, 6, 3, 4, 7, 9, 5, 8
```

> **Key insight:** a node is marked `visited` the moment it's **pushed** into the queue — not when it's popped. This prevents the same node from being pushed multiple times by different neighbors before it's even processed.

## 🛠️ Algorithm / Approach

> [!abstract]
> 1. Create a `visited` array (size `n` or `n+1` depending on indexing), all `false`.
> 2. Create an empty queue and an empty result list.
> 3. Push the `start` node into the queue, mark `visited[start] = true`.
> 4. While the queue is not empty:
>    - Pop the front node `cur`, append it to the result.
>    - For each `neighbor` of `cur` in the adjacency list:
>      - If `!visited[neighbor]`: mark it visited, push it into the queue.
> 5. Return the result.
>
> *(For multiple components, wrap this in the G-3 outer loop — call BFS from every unvisited node.)*

## 👨‍💻 Code

### Java

```java
import java.util.*;

public class BFSTraversal {

    // Single-component BFS starting from a given node
    public static List<Integer> bfs(int start, List<List<Integer>> adj, boolean[] visited) {
        List<Integer> result = new ArrayList<>();
        Queue<Integer> queue = new LinkedList<>();

        queue.offer(start);
        visited[start] = true;

        while (!queue.isEmpty()) {
            int cur = queue.poll();
            result.add(cur);

            for (int neighbor : adj.get(cur)) {
                if (!visited[neighbor]) {
                    visited[neighbor] = true;   // mark visited when PUSHED, not when popped
                    queue.offer(neighbor);
                }
            }
        }
        return result;
    }

    // Handles disconnected graphs too (uses the G-3 pattern)
    public static List<Integer> bfsAllComponents(int n, List<List<Integer>> adj) {
        boolean[] visited = new boolean[n + 1];   // adjust if 0-indexed
        List<Integer> result = new ArrayList<>();

        for (int node = 1; node <= n; node++) {
            if (!visited[node]) {
                result.addAll(bfs(node, adj, visited));
            }
        }
        return result;
    }
}
```

### Python

```python
from collections import deque

# Single-component BFS starting from a given node
def bfs(start, adj, visited):
    result = []
    queue = deque([start])
    visited[start] = True

    while queue:
        cur = queue.popleft()
        result.append(cur)

        for neighbor in adj[cur]:
            if not visited[neighbor]:
                visited[neighbor] = True   # mark visited when PUSHED, not when popped
                queue.append(neighbor)

    return result

# Handles disconnected graphs too (uses the G-3 pattern)
def bfs_all_components(n, adj):
    visited = [False] * (n + 1)   # adjust if 0-indexed
    result = []

    for node in range(1, n + 1):
        if not visited[node]:
            result.extend(bfs(node, adj, visited))

    return result
```

## ⏱️ Complexity Analysis

> [!note]
> - **Time:** O(N + 2E) ≈ **O(N + E)**
>   - Every node enters the queue exactly once → O(N) total pops.
>   - For each popped node, we scan its adjacency list — across *all* nodes, this sums to the total degree count, which (from G-1's handshake property) is `2E` for an undirected graph.
> - **Space:** O(N) — for the queue, the visited array, and the result list (each holds at most N elements). The adjacency list itself is given/already built, so it's not counted as extra.

## ⚠️ Common Pitfalls

> [!warning]
> - **Marking a node visited when it's *popped* instead of when it's *pushed*.** If you wait until popping, the same node can get pushed into the queue multiple times (once for each neighbor that reaches it before it's processed), wasting time and potentially producing duplicate entries in the result.
> - **Forgetting BFS only covers one component.** Always wrap it in the G-3 loop if the graph might be disconnected.
> - **Assuming BFS order is unique for a graph.** It depends on the starting node *and* the order neighbors appear in the adjacency list — don't hardcode expected output order unless the problem fixes both.
> - **Using BFS when you actually need shortest path with weighted edges.** Plain BFS gives shortest paths only on **unweighted** graphs (because level number = number of edges = distance). For weighted graphs, you'll need Dijkstra (covered later).

## 🔗 Related Problems / Next Up

> [!success]
> - **Previous:** [G-3 — Connected Components & Visited Array](G3-Connected-Components.md)
> - **Next:** DFS Traversal
> - Used directly in: shortest path in unweighted graphs, level-order problems, multi-source BFS (rotting oranges, 01 matrix)
> - [↑ Back to Foundations Index](00-Foundations-Index.md)

