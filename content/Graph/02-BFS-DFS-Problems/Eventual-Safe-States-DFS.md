---
title: "Find Eventual Safe States (DFS)"
tags: [graph, bfs-dfs-problems, cycle-detection, dfs, directed-graph]
videoLink: "https://www.youtube.com/playlist?list=PLgUwDviBIf0oE3gA41TKO2H5bHpPd7fzn"
difficulty: Medium
---

# Find Eventual Safe States (DFS)

## 🎯 What problem are we solving?

> [!question]
> Given a directed graph with `V` vertices, a node is a **terminal node** if it has **no outgoing edges** (out-degree `0`). A node is a **safe node** if **every possible path** starting from it eventually leads to a terminal node. Return all safe nodes, sorted in ascending order.
>
> This is solved here using the **cycle detection technique** from the previous note. (A BFS/topological-sort based alternative exists too, covered separately.)

## 💡 Intuition

> [!tip]
> The key structural insight: **any node that is part of a cycle, or that has a path leading into a cycle, can never be safe** — because that path never terminates, it just loops forever instead of reaching a terminal node. Conversely, if a node's every path avoids cycles entirely, it's guaranteed to eventually run out of edges and hit a terminal node — making it safe.
>
> So the whole problem reduces to: **run cycle detection (DFS with `visited` + `pathVisited`, same as before), and any node that is never part of a cycle and never leads into one is safe.**
>
> The mechanism: a `dfs(node)` call only finishes cleanly (returns `false` — no cycle) if **every one of its outgoing paths** also finished cleanly. That's exactly the same requirement as being "safe" — every path from this node eventually bottoms out. So: **whenever a `dfs(node)` call returns `false` (no cycle found through it), mark that node as safe.** If it returns `true` (a cycle was found somewhere down that path), the node is not safe — and neither is any node on the way there.

## 🖼️ Visualizing it

```
Directed edges: 0→1, 1→3, 3→0, 1→2, 2→5, 3→4, 4→6, 6→7, 5→6, 8→1, 8→9, 9→10, 9→11, 10→8
Terminal nodes (no outgoing edges): 6? no — 6→7. Let's use: 5 and 6 are terminal in the reduced example below.
```

Using the example from the video (nodes 0–7, plus 8–11 as a second component):

```
dfs(0): visited, pathVisited
  → dfs(1): visited, pathVisited
    → dfs(2): visited, pathVisited
      → dfs(3): visited, pathVisited
        → dfs(4): visited, pathVisited
          → dfs(6): visited, pathVisited
            → dfs(7): visited, pathVisited
              no outgoing edges → SAFE (terminal), return false, unmark pathVisited
            no more edges → SAFE, return false, unmark pathVisited
          no more edges → SAFE, return false, unmark pathVisited
        → dfs(5): visited, pathVisited
          → 6 already visited, NOT pathVisited (finished, different path) → no cycle here
          no more edges → SAFE, return false, unmark pathVisited
        no more edges → SAFE, return false, unmark pathVisited
      no more edges → SAFE, return false, unmark pathVisited
    no more edges → SAFE, return false, unmark pathVisited
  no more edges → SAFE, return false, unmark pathVisited

Second component:
dfs(8): visited, pathVisited
  → dfs(9): visited, pathVisited
    → dfs(10): visited, pathVisited
      neighbor 8: visited AND pathVisited → CYCLE → return true
    dfs(10) returned true → dfs(9) returns true, NOT marked safe
  dfs(9) returned true → dfs(8) returns true, NOT marked safe
  (9's other neighbor, 11, is never reached because the true short-circuits — but 11 itself
   gets its own dfs call later since it's still unvisited, and being terminal, is marked safe)
```

Every node reachable from `0` safely bottoms out at a terminal node (`7`), so all of `0,1,2,3,4,5,6,7` are safe. But `8,9,10` form/feed a cycle (`8→9→10→8`), so none of them are safe — even though `9` also points at `11`, that doesn't save `9`, because **every** path from `9` must terminate, and the `9→10→8→9` path doesn't.

## 🛠️ Algorithm / Approach

> [!abstract]
> This directly reuses cycle-detection DFS, with one twist: **don't stop at the first cycle found** — every unvisited node still needs its own DFS call so every node gets correctly classified.
>
> 1. Initialize `visited[]`, `pathVisited[]`, and a `check[]` (or `safe[]`) array, all `false`.
> 2. For each node `1..n` not yet visited, call `dfs(node, ...)`.
> 3. **`dfs(node) → boolean`** (returns `true` if a cycle is found through `node`):
>    - Set `visited[node] = true`, `pathVisited[node] = true`.
>    - For each `neighbor`:
>      - If unvisited: recursively call `dfs(neighbor)`. If it returns `true`, this call also returns `true` (do **not** mark `node` safe, but keep checking other nodes at the outer level — don't `break` out of the whole algorithm).
>      - Else if `pathVisited[neighbor]` is `true`: return `true` (cycle).
>    - If the loop completes with no cycle found: **mark `check[node] = true`** (safe), unmark `pathVisited[node] = false`, return `false`.
> 4. After processing all nodes, collect every node where `check[node] == true`, sorted ascending — that's the answer.

## 👨‍💻 Code

### Java

```java
import java.util.*;

public class EventualSafeStates {

    public List<Integer> eventualSafeNodes(int n, List<List<Integer>> adj) {
        boolean[] visited = new boolean[n];
        boolean[] pathVisited = new boolean[n];
        boolean[] check = new boolean[n];   // true = safe node

        for (int i = 0; i < n; i++) {
            if (!visited[i]) {
                dfsCheck(i, adj, visited, pathVisited, check);
            }
        }

        List<Integer> safeNodes = new ArrayList<>();
        for (int i = 0; i < n; i++) {
            if (check[i]) safeNodes.add(i);
        }
        return safeNodes;   // already ascending since we iterate 0..n-1
    }

    private boolean dfsCheck(int node, List<List<Integer>> adj,
                              boolean[] visited, boolean[] pathVisited, boolean[] check) {
        visited[node] = true;
        pathVisited[node] = true;
        check[node] = false;

        for (int neighbor : adj.get(node)) {
            if (!visited[neighbor]) {
                if (dfsCheck(neighbor, adj, visited, pathVisited, check)) {
                    return true;   // cycle found through this path
                }
            } else if (pathVisited[neighbor]) {
                return true;       // visited AND still on the current path -> cycle
            }
        }

        pathVisited[node] = false;   // leaving the current path
        check[node] = true;          // every path from here terminated cleanly -> safe
        return false;
    }
}
```

### Python

```python
class Solution:
    def eventual_safe_nodes(self, n, adj):
        visited = [False] * n
        path_visited = [False] * n
        check = [False] * n   # True = safe node

        def dfs_check(node):
            visited[node] = True
            path_visited[node] = True
            check[node] = False

            for neighbor in adj[node]:
                if not visited[neighbor]:
                    if dfs_check(neighbor):
                        return True   # cycle found through this path
                elif path_visited[neighbor]:
                    return True       # visited AND still on the current path -> cycle

            path_visited[node] = False   # leaving the current path
            check[node] = True            # every path from here terminated cleanly -> safe
            return False

        for i in range(n):
            if not visited[i]:
                dfs_check(i)

        return [i for i in range(n) if check[i]]   # already ascending
```

## ⏱️ Complexity Analysis

> [!note]
> - **Time:** O(V + E) — one DFS pass over the whole graph, same as directed cycle detection.
> - **Space:** O(3N) for `visited`, `pathVisited`, `check` arrays, plus O(N) worst-case recursion stack. This can be compressed into a single array of counters (`0` = unvisited, `1` = in-progress/path-visited, `2` = safe), but three clearly-named arrays are preferred for interview code quality/readability.

## ⚠️ Common Pitfalls

> [!warning]
> - **Breaking out of the entire algorithm on the first cycle found.** Unlike plain cycle detection (where finding *any* cycle is enough to answer "yes/no"), here you must keep calling `dfs` on every remaining unvisited node — a cycle in one component doesn't tell you anything about nodes in a different, cycle-free component.
> - **Marking a node safe *before* checking all its neighbors.** `check[node]` must only be set to `true` after the full neighbor loop completes without returning `true` — setting it early (or defaulting arrays to `true`) will incorrectly mark nodes safe that actually lead into a cycle later in the loop.
> - **Forgetting to reset `pathVisited[node] = false`** when a node finishes safely — this is exactly the same undo-on-the-way-back-up step from directed cycle detection, and skipping it causes false cycle detections in later, unrelated DFS calls.
> - **Returning the result unsorted.** Iterating `0..n-1` when collecting `check[i] == true` naturally produces ascending order — don't shuffle the order some other way (e.g. via a `Set` without sorting) and forget to sort before returning.

## 🔗 Related Problems / Next Up

> [!success]
> - **Builds directly on:** [Detect Cycle in a Directed Graph (DFS)](Detect-Cycle-Directed-DFS.md) — same `visited`/`pathVisited` machinery, repurposed to classify every node instead of just answering yes/no.
> - **Alternative approach (not covered in this note):** reverse the graph's edges and run Kahn's algorithm (BFS-based topological sort) — terminal nodes become sources, and nodes fully processed by Kahn's are exactly the safe nodes.
> - [↑ Back to BFS/DFS Problems Index](00-BFS-DFS-Index.md)
