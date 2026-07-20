---
title: "Topological Sort (DFS)"
tags: [graph, topological-sort, dfs, dag]
videoLink: "https://www.youtube.com/playlist?list=PLgUwDviBIf0oE3gA41TKO2H5bHpPd7fzn"
difficulty: Medium
---

# Topological Sort (DFS)

## 🎯 What problem are we solving?

> [!question]
> A **topological sort** is a linear ordering of a graph's vertices such that for every directed edge `u → v`, `u` appears **before** `v` in the ordering. Given a graph, produce any valid topological ordering — using **DFS**.
>
> Topological sort only exists for a **DAG** (Directed **A**cyclic Graph): it must be directed (an undirected edge `u—v` would force both `u` before `v` *and* `v` before `u`, which is impossible), and it must have no cycle (a cycle like `1→2→3→1` would similarly demand `1` before `2` before `3` before `1` — a contradiction). Multiple valid orderings can exist for the same DAG; any one of them is an acceptable answer.

## 💡 Intuition

> [!tip]
> Run a normal DFS. The key trick: **whenever a DFS call finishes exploring a node completely** (no more unvisited neighbors left to recurse into), **push that node onto a stack** before returning. A node only finishes after *everything reachable from it* has already finished (and been pushed) — so by construction, everything that `node` depends on (i.e. everything it points to) ends up **lower** in the stack than `node` itself.
>
> Once the DFS across all components is done, **popping the stack from top to bottom** gives a valid topological order: whichever node finished last (was "least dependent," i.e. had the most still to explore beneath it, or simply happened to be visited first and had a long chain hanging off it) comes out first, and terminal/leaf nodes — which finish immediately — end up at the bottom of the stack and come out last.

## 🖼️ Visualizing it

```
Edges: 5→0, 4→0, 5→2, 2→3, 3→1, 4→1
```

```
dfs(0): visited. No adjacent nodes. → push 0.        stack: [0]
dfs(1): visited. No adjacent nodes. → push 1.        stack: [0, 1]
dfs(2): visited.
  → dfs(3): visited.
    → neighbor 1 already visited, skip. No more neighbors → push 3.   stack: [0, 1, 3]
  no more neighbors for 2 → push 2.                   stack: [0, 1, 3, 2]
dfs(4): visited. neighbors 0, 1 both already visited → push 4.        stack: [0, 1, 3, 2, 4]
dfs(5): visited. neighbors 0, 2 both already visited → push 5.        stack: [0, 1, 3, 2, 4, 5]
```

Popping the stack top-to-bottom: `5, 4, 2, 3, 1, 0` — a valid topological order. Check: `5→0` (5 before 0 ✓), `4→0` (✓), `5→2` (✓), `2→3` (✓), `3→1` (✓), `4→1` (✓). All edges satisfied.

## 🛠️ Algorithm / Approach

> [!abstract]
> 1. Initialize a `visited` array (all `false`) and an empty `stack`.
> 2. For every node `1..n` not yet visited (handles disconnected components), call `dfs(node, adj, visited, stack)`.
> 3. **`dfs(node, ...)`:**
>    - Mark `node` visited.
>    - For each unvisited `neighbor`: recursively call `dfs(neighbor, ...)`.
>    - After the loop (all neighbors handled), **push `node` onto the stack**.
> 4. Once all components are processed, pop the stack repeatedly into a result list — that list is the topological order.

## 👨‍💻 Code

### Java

```java
import java.util.*;

public class TopologicalSortDFS {

    private void dfs(int node, List<List<Integer>> adj, boolean[] visited, Deque<Integer> stack) {
        visited[node] = true;

        for (int neighbor : adj.get(node)) {
            if (!visited[neighbor]) {
                dfs(neighbor, adj, visited, stack);
            }
        }
        stack.push(node);   // this node is fully explored -> everything below it is done
    }

    public int[] topoSort(int n, List<List<Integer>> adj) {
        boolean[] visited = new boolean[n];
        Deque<Integer> stack = new ArrayDeque<>();

        for (int i = 0; i < n; i++) {
            if (!visited[i]) {
                dfs(i, adj, visited, stack);
            }
        }

        int[] result = new int[n];
        for (int i = 0; i < n; i++) {
            result[i] = stack.pop();
        }
        return result;
    }
}
```

### Python

```python
class Solution:
    def topo_sort(self, n, adj):
        visited = [False] * n
        stack = []

        def dfs(node):
            visited[node] = True
            for neighbor in adj[node]:
                if not visited[neighbor]:
                    dfs(neighbor)
            stack.append(node)   # this node is fully explored -> everything below it is done

        for i in range(n):
            if not visited[i]:
                dfs(i)

        return stack[::-1]   # pop from the top = reverse of append order
```

## ⏱️ Complexity Analysis

> [!note]
> - **Time:** O(V + E) — a single DFS traversal over the whole graph.
> - **Space:** O(N) for `visited` and the `stack`, plus O(N) worst-case recursion depth.

## ⚠️ Common Pitfalls

> [!warning]
> - **Pushing a node onto the stack *before* recursing into its neighbors** instead of after. This breaks the ordering entirely — the whole trick depends on pushing only once a node's *entire* subtree of dependencies has finished.
> - **Trying to run this on a graph with a cycle.** Topological sort is only well-defined on a DAG — running this DFS on a cyclic graph will still terminate (thanks to the `visited` check) but produces a meaningless/invalid ordering, since it silently ignores the back-edge that created the cycle.
> - **Forgetting the multi-component loop.** A DAG can have multiple disconnected pieces — each unvisited node needs its own `dfs` call to be included in the final ordering.
> - **Reading the stack in the wrong direction.** The answer is obtained by popping (or reversing the append order) — reading the stack front-to-back instead gives the *reverse* of a valid topological order.

## 🔗 Related Problems / Next Up

> [!success]
> - **Builds on:** [G5. DFS Traversal](../01-Foundations/G5-DFS-Traversal.md)
> - **Companion:** [Topological Sort (BFS / Kahn's Algorithm)](Topological-Sort-BFS-Kahns-Algorithm.md) — an alternative, iterative approach using in-degrees
> - [↑ Back to Topological Sort Index](00-Topological-Sort-Index.md)
