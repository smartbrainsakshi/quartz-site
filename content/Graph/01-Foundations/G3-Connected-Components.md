---
title: "G-3. Connected Components & the Visited Array Pattern"
tags: [graph, foundations, connected-components, visited-array]
videoLink: "https://www.youtube.com/playlist?list=PLgUwDviBIf0oE3gA41TKO2H5bHpPd7fzn"
---

# G-3. Connected Components & the Visited Array Pattern

## 🎯 What problem are we solving?

> [!question]
> So far every graph we've drawn has been a single, fully connected blob. But a graph doesn't have to be "one piece" — it can be several disconnected pieces bundled together under one `n`/`m` input. This video explains what that means, and more importantly, establishes the **one pattern every traversal algorithm (BFS, DFS, and beyond) will use for the rest of the series**: the visited-array loop.

## 💡 Intuition

> [!tip]
> Picture four separate little graphs sitting on a page, not touching each other:
>
> ```
> Piece A: 1 -- 2        Piece B: 5 -- 6
>          |    |                 |
>          3 -- 4                 7
>
> Piece C: 8 -- 9        Piece D: 10 (alone)
> ```
>
> Each piece, on its own, is a valid graph (a single node is even a valid graph — remember from G-1, the only requirement is nodes + edges, no "must be one connected blob" rule).
>
> Now imagine someone hands you a single problem: *"Here's an undirected graph with 10 nodes and 8 edges: (1,2) (1,3) (2,4) (3,4) (5,6) (6,7) (5,7) (8,9)."* Even though the edges only describe these 4 separate pieces, the problem still treats it as **one graph — split into 4 connected components.**
>
> > A **connected component** is a maximal group of nodes where every node can reach every other node in the group via some path, and no node outside the group can be reached.

## 🖼️ Visualizing it

```
Component 1: {1, 2, 3, 4}
Component 2: {5, 6, 7}
Component 3: {8, 9}
Component 4: {10}
```

### Why this breaks a naive traversal
Imagine a future traversal algorithm (BFS or DFS — not named yet in this video) that starts at node 1 and visits everything reachable from it: `1 → 2 → 4 → 3`. It would mark `{1,2,3,4}` as visited and then... stop. Nodes `5` through `10` are never touched, because there's no edge connecting component 1 to any of the others. **A single traversal call from one starting node only ever covers that node's own component.**

### The fix: loop over every node + a visited array
To make sure *every* node in *every* component gets processed, every traversal algorithm in this series follows this exact shape:

```
visited = array of size (n+1), all initialized to false

for node in 1 to n:
    if visited[node] == false:
        traverse(node)      # BFS or DFS — marks everyone in this component visited
```

Walking through our 10-node example:
1. `node = 1` → not visited → `traverse(1)` runs, visits `{1,2,3,4}`, marks all four visited.
2. `node = 2` → already visited → skip.
3. `node = 3` → already visited → skip.
4. `node = 4` → already visited → skip.
5. `node = 5` → not visited → `traverse(5)` runs, visits `{5,6,7}`, marks all three visited.
6. `node = 6, 7` → already visited → skip.
7. `node = 8` → not visited → `traverse(8)` runs, visits `{8,9}`.
8. `node = 9` → already visited → skip.
9. `node = 10` → not visited → `traverse(10)` runs, visits just `{10}` (a component of one node).

Result: every node gets visited exactly once, and the number of times `traverse(...)` actually *runs* (not skipped) tells you **the number of connected components** — a pattern you'll reuse directly in problems like "Number of Provinces."

## 🛠️ Algorithm / Approach

> [!abstract]
> 1. Build the adjacency list (from G-2).
> 2. Create a `visited` array of size `n+1`, all `false`.
> 3. Loop `node` from `1` to `n`:
>    - If `visited[node]` is `false`, call the traversal function (BFS/DFS) starting at `node`. This call will mark every node in that node's component as visited.
> 4. (Optional, depending on the problem) Count how many times step 3's traversal actually fires — that count is the number of connected components.

## 👨‍💻 Code

This video doesn't teach BFS/DFS itself yet — just the outer driver pattern. Here's that pattern as a reusable skeleton (the `traverse(...)` body gets filled in with BFS or DFS in the next notes):

### Java

```java
import java.util.*;

public class ConnectedComponents {

    public static int countComponents(int n, List<List<Integer>> adj) {
        boolean[] visited = new boolean[n + 1];
        int components = 0;

        for (int node = 1; node <= n; node++) {
            if (!visited[node]) {
                traverse(node, adj, visited);   // BFS or DFS body goes here
                components++;
            }
        }
        return components;
    }

    // Placeholder — replaced by real BFS/DFS in the next notes
    private static void traverse(int start, List<List<Integer>> adj, boolean[] visited) {
        visited[start] = true;
        for (int neighbor : adj.get(start)) {
            if (!visited[neighbor]) {
                traverse(neighbor, adj, visited);
            }
        }
    }
}
```

### Python

```python
def count_components(n, adj):
    visited = [False] * (n + 1)
    components = 0

    for node in range(1, n + 1):
        if not visited[node]:
            traverse(node, adj, visited)   # BFS or DFS body goes here
            components += 1

    return components

# Placeholder — replaced by real BFS/DFS in the next notes
def traverse(start, adj, visited):
    visited[start] = True
    for neighbor in adj[start]:
        if not visited[neighbor]:
            traverse(neighbor, adj, visited)
```

## ⏱️ Complexity Analysis

> [!note]
> - **Time:** O(V + E) — the outer loop is O(V), and across *all* `traverse(...)` calls combined, every node is visited once and every edge is examined at most twice (once from each endpoint) — same bound as a single full traversal, just possibly split across multiple components.
> - **Space:** O(V) for the `visited` array (plus O(V) recursion stack / queue space depending on whether the traversal is DFS or BFS).

## ⚠️ Common Pitfalls

> [!warning]
> - **Calling the traversal only once, from node 1.** This is the single most common bug when a graph isn't fully connected — it silently ignores every other component and gives wrong answers (e.g., undercounting reachable nodes, missing cycles in other components).
> - **Forgetting to initialize/size the `visited` array as `n+1`** to match 1-indexed nodes (same off-by-one trap as G-2).
> - **Re-traversing an already-visited node's component.** Always check `visited[node]` *before* calling traverse — without this check you'd redo work and could infinite-loop in cyclic graphs.
> - **Assuming "connected component" only applies to weird/contrived inputs.** In real interview problems (friend circles, islands, provinces), disconnected components are the norm, not the exception — always assume the graph might be split.

## 🔗 Related Problems / Next Up

> [!success]
> - **Previous:** [G-2 — Graph Representation](G2-Graph-Representation.md)
> - **Next:** BFS Traversal (fills in the `traverse(...)` placeholder above with a real algorithm)
> - Used directly in: *Number of Provinces*, *Number of Islands*, *Connected Components in an Undirected Graph*
> - [↑ Back to Foundations Index](00-Foundations-Index.md)

