---
title: "Number of Provinces"
tags: [graph, bfs-dfs-problems, connected-components, dfs, bfs]
videoLink: "https://www.youtube.com/playlist?list=PLgUwDviBIf0oE3gA41TKO2H5bHpPd7fzn"
difficulty: Medium
---

# Number of Provinces

## 🎯 What problem are we solving?

> [!question]
> Given an undirected graph, two nodes `u` and `v` belong to the same **province** if there's a path between them (directly or indirectly). Find the total number of provinces — i.e., **how many separate connected components does this graph have?**
>
> This is the very first problem in the series, and it's deliberately the simplest possible application of everything learned so far: BFS/DFS (G-4, G-5) combined with the visited-array-driven outer loop pattern (G-3).

## 💡 Intuition

> [!tip]
> A "province" is just another name for a **connected component**. If you can travel from node A to node B through any sequence of edges, they're in the same province.
>
> The key realization: **a single BFS or DFS call, started from any node, will visit every node reachable from it — i.e., its entire province — and nothing outside it.** So if you:
>
> 1. Loop through every node from `1` to `n`.
> 2. Whenever you find an **unvisited** node, that means you've found the *first* node of a province nobody has touched yet — increment your province counter and run a full BFS/DFS from it (marking everyone in that province as visited).
> 3. Keep going until every node has been visited.
>
> ...the number of times you triggered step 2 *is* the number of provinces.
>
> This is **exactly** the G-3 pattern (visited array + loop + traverse), with one line added: a counter that increments every time the `if (!visited[node])` check passes.

## 🖼️ Visualizing it

```
Province A: 1 -- 2          Province B: 4 -- 5          Province C: 7 -- 8
            |  /                        |
            3                           6
```

```
node = 1 → not visited → DFS(1) visits {1, 2, 3} → provinces = 1
node = 2 → visited → skip
node = 3 → visited → skip
node = 4 → not visited → DFS(4) visits {4, 5, 6} → provinces = 2
node = 5 → visited → skip
node = 6 → visited → skip
node = 7 → not visited → DFS(7) visits {7, 8} → provinces = 3
node = 8 → visited → skip
```

**Answer: 3 provinces.**

### ⚠️ Watch the input format
This problem is commonly given as an **adjacency matrix** (`isConnected[i][j] == 1` means an edge exists), not an adjacency list like every previous video. Two ways to handle it:
- **Option A:** convert the matrix to an adjacency list first (clean, reuses your existing DFS/BFS code unchanged).
- **Option B:** run DFS/BFS directly on the matrix, treating row `i` as "scan all `j` for `matrix[i][j] == 1`".

The note below uses Option A since it keeps the traversal code identical to G-4/G-5.

## 🛠️ Algorithm / Approach

> [!abstract]
> 1. If given as a matrix, convert to an adjacency list: for every `i, j` where `matrix[i][j] == 1` and `i != j`, add `j` to `adj[i]` and `i` to `adj[j]`.
> 2. Create a `visited` array, all `false`, and a `provinces` counter at `0`.
> 3. Loop `node` from `0` (or `1`, depending on indexing) to `n - 1`:
>    - If `!visited[node]`: run DFS (or BFS) from `node` to mark its whole component visited, and increment `provinces`.
> 4. Return `provinces`.

## 👨‍💻 Code

### Java

```java
import java.util.*;

public class NumberOfProvinces {

    public int findCircleNum(int[][] isConnected) {
        int n = isConnected.length;

        // Step 1: convert adjacency matrix -> adjacency list
        List<List<Integer>> adj = new ArrayList<>();
        for (int i = 0; i < n; i++) adj.add(new ArrayList<>());

        for (int i = 0; i < n; i++) {
            for (int j = 0; j < n; j++) {
                if (isConnected[i][j] == 1 && i != j) {
                    adj.get(i).add(j);
                    adj.get(j).add(i);
                }
            }
        }

        // Step 2: G-3 pattern — loop + visited + DFS, count triggers
        boolean[] visited = new boolean[n];
        int provinces = 0;

        for (int node = 0; node < n; node++) {
            if (!visited[node]) {
                dfs(node, adj, visited);
                provinces++;
            }
        }
        return provinces;
    }

    private void dfs(int node, List<List<Integer>> adj, boolean[] visited) {
        visited[node] = true;
        for (int neighbor : adj.get(node)) {
            if (!visited[neighbor]) {
                dfs(neighbor, adj, visited);
            }
        }
    }
}
```

### Python

```python
class Solution:
    def findCircleNum(self, isConnected: list[list[int]]) -> int:
        n = len(isConnected)

        # Step 1: convert adjacency matrix -> adjacency list
        adj = [[] for _ in range(n)]
        for i in range(n):
            for j in range(n):
                if isConnected[i][j] == 1 and i != j:
                    adj[i].append(j)
                    adj[j].append(i)

        # Step 2: G-3 pattern — loop + visited + DFS, count triggers
        visited = [False] * n
        provinces = 0

        def dfs(node):
            visited[node] = True
            for neighbor in adj[node]:
                if not visited[neighbor]:
                    dfs(neighbor)

        for node in range(n):
            if not visited[node]:
                dfs(node)
                provinces += 1

        return provinces
```

## ⏱️ Complexity Analysis

> [!note]
> - **Time:** O(N²) for converting the matrix to a list (have to scan the full N×N matrix), **plus O(N + 2E)** for the traversal itself.
>   - The outer `for` loop runs N times total (across the whole algorithm), and DFS is only ever called once per node — so even though DFS gets *triggered* from multiple starting points, the combined work across all those partial DFS calls still sums to exactly one full traversal: **O(N + 2E)**.
>   - If the input is already given as an adjacency list (as most problems do going forward), drop the O(N²) conversion cost entirely.
> - **Space:** O(N) for the `visited` array + O(N) worst-case recursion stack (skewed graph) — the adjacency list/matrix itself isn't counted as extra since it's part of the input.

## ⚠️ Common Pitfalls

> [!warning]
> - **Forgetting to skip self-loops (`i == j`)** when converting the matrix — `isConnected[i][i]` is always `1` by definition (a node is trivially connected to itself) and should **not** be added as an edge.
> - **Re-running DFS from an already-visited node** — wastes time and, if the visited check is missing, breaks the province count (you'd overcount).
> - **Assuming the input is always an adjacency list.** This specific problem (LeetCode 547) gives a matrix — always check the input format before reusing traversal code blindly.
> - **Off-by-one indexing mismatch** between the matrix size and the visited array / loop bounds — this problem is naturally 0-indexed (matrix rows/cols start at 0), unlike some earlier 1-indexed examples in G-1–G-5.

## 🔗 Related Problems / Next Up

> [!success]
> - **Builds on:** [G-3 — Connected Components](../01-Foundations/G3-Connected-Components.md), [G-4 — BFS](../01-Foundations/G4-BFS-Traversal.md), [G-5 — DFS](../01-Foundations/G5-DFS-Traversal.md)
> - **Pattern reused in:** every "count the number of X" problem on graphs (Number of Islands, Friend Circles, etc.)
> - [↑ Back to BFS/DFS Problems Index](00-BFS-DFS-Index.md)

