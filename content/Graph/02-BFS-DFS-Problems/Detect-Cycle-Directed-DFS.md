---
title: "Detect Cycle in a Directed Graph (DFS)"
tags: [graph, bfs-dfs-problems, cycle-detection, dfs, directed-graph]
videoLink: "https://www.youtube.com/playlist?list=PLgUwDviBIf0oE3gA41TKO2H5bHpPd7fzn"
difficulty: Medium
---

# Detect Cycle in a Directed Graph (DFS)

## 🎯 What problem are we solving?

> [!question]
> Given a **directed** graph, determine whether it contains a cycle. This looks like the undirected cycle-detection problem, but the undirected trick (mark `visited`, flag a clash with any non-parent visited neighbor) **does not work here** — it needs a genuinely new idea.

## 💡 Intuition

> [!tip]
> Why does the undirected approach break? Because in a directed graph, reaching an already-`visited` node doesn't mean you looped back — it might just mean two *different* paths happen to converge on the same node, which is completely legal and not a cycle.
>
> The real rule: **a cycle exists only if you revisit a node that is on your *current* path** — i.e., a node you're still "inside of" recursively, not one you finished exploring earlier via some unrelated branch. So DFS needs **two** tracking arrays instead of one:
> - `visited[node]` — has this node ever been explored, in any path, ever (so we never redo work).
> - `pathVisited[node]` — is this node part of the **current** recursion stack, right now (so we can detect "I came back around to something still open above me").
>
> Landing on a node that is `visited` but **not** `pathVisited` is totally fine — it was finished via a different path and can be skipped. Landing on a node that is both `visited` **and** `pathVisited` means the current path curled back onto itself: a genuine cycle. Critically, when a DFS call for a node finishes *without* finding a cycle, that node must be **unmarked from `pathVisited`** (set back to `false`) before returning — it's leaving the current path, even though it stays `visited` forever.

## 🖼️ Visualizing it

```
Directed edges: 1→2, 2→3, 3→4, 4→5, 5→6, 3→7, 7→5
                8→9, 9→10, 10→8, 8→1, 9→11
```

```
dfs(1): visited[1]=1, pathVisited[1]=1
  → dfs(2): visited[2]=1, pathVisited[2]=1
    → dfs(3): visited[3]=1, pathVisited[3]=1
      → dfs(4): visited[4]=1, pathVisited[4]=1
        → dfs(5): visited[5]=1, pathVisited[5]=1
          → dfs(6): visited[6]=1, pathVisited[6]=1
            no outgoing edges → return false, pathVisited[6]=0
          return false, pathVisited[5]=0
        return false, pathVisited[4]=0
      (back at 3, still has edge to 7)
      → dfs(7): visited[7]=1, pathVisited[7]=1
        neighbor 5: visited=1 but pathVisited=0 (5 already finished, different path) → skip, no cycle
        return false, pathVisited[7]=0
      return false, pathVisited[3]=0
    return false, pathVisited[2]=0
  return false, pathVisited[1]=0
```

No cycle found starting from `1` — every "already visited" hit (like `7 → 5`) was on a node that had **already exited** the current path (`pathVisited = 0`), so it's safe.

Now the second component:

```
dfs(8): visited[8]=1, pathVisited[8]=1
  → dfs(9): visited[9]=1, pathVisited[9]=1
    → dfs(10): visited[10]=1, pathVisited[10]=1
      neighbor 8: visited=1 AND pathVisited=1 → CYCLE → return true
    dfs(10) returned true → dfs(9) returns true immediately
  dfs(9) returned true → dfs(8) returns true immediately
```

`10 → 8` hits a node that is still **on the current path** (`pathVisited[8] = 1`) — that's the real cycle `8 → 9 → 10 → 8`. The result `true` propagates straight back up the call chain.

## 🛠️ Algorithm / Approach

> [!abstract]
> **`dfs(node, adj, visited, pathVisited) → boolean`:**
> 1. Set `visited[node] = true` and `pathVisited[node] = true`.
> 2. For each `neighbor` of `node`:
>    - If **not visited**: recursively call `dfs(neighbor, ...)`. **If it returns `true`, immediately return `true`.**
>    - Else if `pathVisited[neighbor] == true`: this neighbor is on the current path → **return `true`** (cycle).
>    - (If visited but not path-visited → it's finished business from another path, skip it.)
> 3. Before returning `false`, set `pathVisited[node] = false` — this node is leaving the current path.
> 4. Return `false`.
>
> **Multi-component handling:** loop over all nodes `1..n`; if unvisited, call `dfs(i, ...)`. If any call returns `true`, the graph has a cycle.

## 👨‍💻 Code

### Java

```java
import java.util.*;

public class DetectCycleDirectedDFS {

    private boolean dfsCheck(int node, List<List<Integer>> adj, boolean[] visited, boolean[] pathVisited) {
        visited[node] = true;
        pathVisited[node] = true;

        for (int neighbor : adj.get(node)) {
            if (!visited[neighbor]) {
                if (dfsCheck(neighbor, adj, visited, pathVisited)) {
                    return true;   // propagate the cycle finding upward immediately
                }
            } else if (pathVisited[neighbor]) {
                return true;       // visited AND still on the current path -> cycle
            }
        }

        pathVisited[node] = false;   // leaving the current path
        return false;
    }

    public boolean isCyclic(int n, List<List<Integer>> adj) {
        boolean[] visited = new boolean[n];
        boolean[] pathVisited = new boolean[n];

        for (int i = 0; i < n; i++) {
            if (!visited[i]) {
                if (dfsCheck(i, adj, visited, pathVisited)) return true;
            }
        }
        return false;
    }
}
```

### Python

```python
class Solution:
    def is_cyclic(self, n, adj):
        visited = [False] * n
        path_visited = [False] * n

        def dfs_check(node):
            visited[node] = True
            path_visited[node] = True

            for neighbor in adj[node]:
                if not visited[neighbor]:
                    if dfs_check(neighbor):
                        return True   # propagate the cycle finding upward immediately
                elif path_visited[neighbor]:
                    return True       # visited AND still on the current path -> cycle

            path_visited[node] = False   # leaving the current path
            return False

        for i in range(n):
            if not visited[i]:
                if dfs_check(i):
                    return True
        return False
```

## ⏱️ Complexity Analysis

> [!note]
> - **Time:** O(V + E) — each node is visited once and each directed edge examined once (unlike undirected graphs, there's no "×2" factor since edges aren't traversed from both ends).
> - **Space:** O(2N) — `visited` and `pathVisited` arrays — plus O(N) worst-case recursion stack. (Can be optimized to a single array using values `0`/`1`/`2` instead of two booleans, but two clearly-named arrays are far more readable — prefer that in interviews.)

## ⚠️ Common Pitfalls

> [!warning]
> - **Forgetting to unmark `pathVisited[node] = false` before returning `false`.** Without this, nodes stay marked "on the path" forever, causing false-positive cycle detections for completely unrelated later paths that happen to revisit them.
> - **Treating "already visited" as automatically a cycle**, like in the undirected version. In a directed graph, a visited-but-not-path-visited node is a legitimate reconvergence of two separate paths — not a cycle.
> - **Not short-circuiting the moment a `true` is found.** Every calling frame must immediately return `true` without continuing to check other neighbors, or without accidentally still resetting `pathVisited` on the way out (once a cycle is confirmed, there's no more "leaving the path" cleanup to do for that call).
> - **Not handling disconnected graphs** — always loop over every node and start a fresh DFS from each unvisited one.

## 🔗 Related Problems / Next Up

> [!success]
> - **Contrast with:** [Detect Cycle in an Undirected Graph (DFS)](Detect-Cycle-Undirected-DFS.md) — same recursive shape, but directed graphs need the extra `pathVisited` array since the simple parent-exclusion trick doesn't generalize.
> - **Used by:** Eventual Safe States, and later, cycle detection via Kahn's algorithm / topological sort (BFS-based alternative for directed graphs).
> - [↑ Back to BFS/DFS Problems Index](00-BFS-DFS-Index.md)
