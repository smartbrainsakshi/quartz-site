---
title: "G-5. DFS Traversal"
tags: [graph, foundations, dfs, traversal, recursion]
videoLink: "https://www.youtube.com/playlist?list=PLgUwDviBIf0oE3gA41TKO2H5bHpPd7fzn"
---

# G-5. DFS Traversal

## 🎯 What problem are we solving?

> [!question]
> BFS explored level by level. **DFS (Depth-First Search)** is the second fundamental traversal — instead of fanning out evenly, it commits to **one path and goes as deep as possible** before backtracking. Both traversals visit every node exactly once, but the *order* and the *problems they're naturally suited for* differ a lot.

## 💡 Intuition

> [!tip]
> "Depth" is the keyword. From the starting node, pick **any one** neighbor and go to it. From there, don't stop — pick **any one** of *its* neighbors and keep going deeper. Only when you hit a node with no unvisited neighbors left do you **backtrack** (return to the previous node) and try its next unexplored branch.
>
> > This "go deep, hit a dead end, backtrack, try the next branch" behavior is exactly what **recursion** does naturally — which is why DFS is implemented recursively (using the call stack) rather than with an explicit queue like BFS.
>
> Just like BFS, **the exact DFS order depends on the starting node and on the order neighbors are stored in the adjacency list** — there's no single "correct" DFS order for a graph, only "a valid DFS order."

## 🖼️ Visualizing it

Take this graph (edges: 1-2, 1-3, 2-5, 2-6, 3-4, 3-7, 4-8, 7-8), starting DFS from node **1**:

```
        1
       / \
      2   3
     / \   \
    5   6   4
             \
              8
             /
            7  (7 also connects back to 3)
```

**Trace, going "first neighbor first" each time:**

```
Visit 1 → neighbors: 2, 3 → go deep into 2 first
  Visit 2 → neighbors: 1(visited), 5, 6 → go deep into 5
    Visit 5 → neighbors: 2(visited) → dead end → backtrack to 2
  Back at 2 → next neighbor: 6
    Visit 6 → neighbors: 2(visited) → dead end → backtrack to 2
  Back at 2 → no more neighbors → backtrack to 1
Back at 1 → next neighbor: 3
  Visit 3 → neighbors: 1(visited), 4, 7 → go deep into 4
    Visit 4 → neighbors: 3(visited), 8 → go deep into 8
      Visit 8 → neighbors: 4(visited), 7 → go deep into 7
        Visit 7 → neighbors: 3(visited), 8(visited) → dead end → backtrack
      Back at 8 → no more neighbors → backtrack to 4
    Back at 4 → no more neighbors → backtrack to 3
  Back at 3 → next neighbor 7 already visited → skip → backtrack to 1
Back at 1 → no more neighbors → done
```

**DFS order:** `1 → 2 → 5 → 6 → 3 → 4 → 8 → 7`

Notice how node `7` got visited as a side effect of diving through `4 → 8`, *before* node `3` ever directly tried to visit it — that's the backtracking nature of DFS in action.

If you start DFS from a *different* node, or the adjacency list stores neighbors in a different order, you'd get a completely different (but equally valid) DFS sequence — same as BFS.

## 🛠️ Algorithm / Approach

> [!abstract]
> **Recursive DFS from a node:**
> 1. Mark the current node as visited, add it to the result list.
> 2. For each neighbor of the current node (from the adjacency list):
>    - If the neighbor is **not** visited, recursively call DFS on it.
> 3. When a node has no unvisited neighbors left, the call returns — this *is* the backtrack.
>
> *(For multiple components — same as BFS — wrap this in the G-3 outer loop: call DFS from every unvisited node.)*

## 👨‍💻 Code

### Java

```java
import java.util.*;

public class DFSTraversal {

    public static void dfs(int node, List<List<Integer>> adj, boolean[] visited, List<Integer> result) {
        visited[node] = true;
        result.add(node);

        for (int neighbor : adj.get(node)) {
            if (!visited[neighbor]) {
                dfs(neighbor, adj, visited, result);
            }
        }
    }

    // Handles disconnected graphs too (uses the G-3 pattern)
    public static List<Integer> dfsAllComponents(int n, List<List<Integer>> adj) {
        boolean[] visited = new boolean[n + 1];   // adjust if 0-indexed
        List<Integer> result = new ArrayList<>();

        for (int node = 1; node <= n; node++) {
            if (!visited[node]) {
                dfs(node, adj, visited, result);
            }
        }
        return result;
    }
}
```

### Python

```python
def dfs(node, adj, visited, result):
    visited[node] = True
    result.append(node)

    for neighbor in adj[node]:
        if not visited[neighbor]:
            dfs(neighbor, adj, visited, result)

# Handles disconnected graphs too (uses the G-3 pattern)
def dfs_all_components(n, adj):
    visited = [False] * (n + 1)   # adjust if 0-indexed
    result = []

    for node in range(1, n + 1):
        if not visited[node]:
            dfs(node, adj, visited, result)

    return result
```

## ⏱️ Complexity Analysis

> [!note]
> - **Time:** O(N + 2E) ≈ **O(N + E)** for undirected graphs (O(N + E) for directed, since each edge is only traversed once instead of twice).
>   - Each node's DFS function is called exactly once → O(N).
>   - Inside each call, we iterate over that node's neighbors — summed across all nodes, this equals the total degree count, which is `2E` for an undirected graph (handshake property from G-1).
> - **Space:** O(N) for the `visited` array and result list, **plus O(N) worst-case recursion stack space**. A "skewed" graph (essentially a long chain: 1→2→3→...→n) forces the recursion N levels deep before any backtracking happens — this stack usage is the main practical difference from BFS's queue-based space.

## ⚠️ Common Pitfalls

> [!warning]
> - **Forgetting the `visited` check before recursing**, leading to infinite recursion on any graph with a cycle (DFS will keep bouncing back and forth between two connected, already-visited nodes forever without it).
> - **Marking visited too late** (e.g., at the start of the *next* recursive call instead of immediately upon entering the current one) — this can cause the same node to be pushed onto the call stack multiple times.
> - **Underestimating stack depth on large/skewed graphs.** A graph with `N` up to `10^5` arranged as a long chain can cause a stack overflow with recursive DFS in languages with limited default stack size (Java/Python especially) — an iterative DFS using an explicit stack is the fix if this becomes a problem.
> - **Expecting BFS and DFS to produce the same node order.** They won't, except in trivial graphs — and most problems only require *a* valid traversal, not a specific one.

## 🔗 Related Problems / Next Up

> [!success]
> - **Previous:** [G-4 — BFS Traversal](G4-BFS-Traversal.md)
> - **Next:** Number of Provinces (first problem applying BFS/DFS + the G-3 component-counting pattern together)
> - Used directly in: cycle detection, topological sort (DFS-based), connected components, backtracking-style graph problems
> - [↑ Back to Foundations Index](00-Foundations-Index.md)

