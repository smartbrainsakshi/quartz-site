---
title: "Course Schedule II"
tags: [graph, topological-sort, bfs, kahns-algorithm]
videoLink: "https://www.youtube.com/playlist?list=PLgUwDviBIf0oE3gA41TKO2H5bHpPd7fzn"
difficulty: Medium
---

# Course Schedule II

## 🎯 What problem are we solving?

> [!question]
> Same setup as [Course Schedule I](Course-Schedule-I.md): `n` tasks and prerequisite pairs. This time, instead of just answering "is it possible," **return an actual valid ordering** of all tasks that satisfies every prerequisite. If it's impossible (a cyclic dependency exists), return an empty array.
>
> One subtlety: the prerequisite pairs here are given as `(a, b)` meaning "to do `a`, you must first do `b`" — same direction as Course Schedule I, no need to re-derive the edge direction, just build the same graph.

## 💡 Intuition

> [!tip]
> This is **literally Kahn's algorithm with no shortcuts** — Course Schedule I only needed the *count* of processed nodes, but this version needs the *actual sequence* Kahn's algorithm produces. Every time Kahn's algorithm pops a node from the queue, that node is a legitimate "safe to do next" task — append it to the result as you go, instead of just incrementing a counter.
>
> At the end: if the result contains all `n` tasks, it's a valid topological order — return it directly. If it's short (fewer than `n` elements were ever pushed/popped), there's a cyclic dependency blocking some tasks — return an empty array instead.

## 🖼️ Visualizing it

```
n = 4, prerequisites: (1,0), (2,0), (3,1), (3,2)
Meaning: 0 before 1, 0 before 2, 1 before 3, 2 before 3
Edges:   0→1, 0→2, 1→3, 2→3
```

```
in-degree: 0:0  1:1  2:1  3:2

queue = [0]
pop 0 → result=[0], neighbor 1: in-degree 0 → push 1
                      neighbor 2: in-degree 0 → push 2
pop 1 → result=[0,1], neighbor 3: in-degree 1 (not 0 yet)
pop 2 → result=[0,1,2], neighbor 3: in-degree 0 → push 3
pop 3 → result=[0,1,2,3], no neighbors

result has all 4 nodes → return [0, 1, 2, 3]
```

If instead there had been a cycle (e.g. an extra edge `3→0`), node `0` would never reach in-degree `0`, the queue would start empty, and the result would stay shorter than `n` — signaling "return `[]`."

## 🛠️ Algorithm / Approach

> [!abstract]
> 1. Build the adjacency list from prerequisite pairs (same direction as Course Schedule I: `(a, b)` → edge `b → a`).
> 2. Run Kahn's algorithm, **appending each popped node to a result list** instead of just counting.
> 3. After the queue empties: if `result.size() == n`, return `result`. Otherwise, return an empty list (a cycle blocked some tasks).

## 👨‍💻 Code

### Java

```java
import java.util.*;

public class CourseScheduleII {

    public int[] findOrder(int n, int[][] prerequisites) {
        List<List<Integer>> adj = new ArrayList<>();
        for (int i = 0; i < n; i++) adj.add(new ArrayList<>());

        int[] inDegree = new int[n];
        for (int[] pair : prerequisites) {
            int a = pair[0], b = pair[1];   // b must come before a
            adj.get(b).add(a);
            inDegree[a]++;
        }

        Queue<Integer> queue = new LinkedList<>();
        for (int i = 0; i < n; i++) {
            if (inDegree[i] == 0) queue.offer(i);
        }

        List<Integer> topo = new ArrayList<>();
        while (!queue.isEmpty()) {
            int node = queue.poll();
            topo.add(node);

            for (int neighbor : adj.get(node)) {
                inDegree[neighbor]--;
                if (inDegree[neighbor] == 0) {
                    queue.offer(neighbor);
                }
            }
        }

        if (topo.size() != n) return new int[0];   // cycle -> impossible

        int[] result = new int[n];
        for (int i = 0; i < n; i++) result[i] = topo.get(i);
        return result;
    }
}
```

### Python

```python
from collections import deque

class Solution:
    def find_order(self, n, prerequisites):
        adj = [[] for _ in range(n)]
        in_degree = [0] * n

        for a, b in prerequisites:   # b must come before a
            adj[b].append(a)
            in_degree[a] += 1

        queue = deque([i for i in range(n) if in_degree[i] == 0])
        topo = []

        while queue:
            node = queue.popleft()
            topo.append(node)

            for neighbor in adj[node]:
                in_degree[neighbor] -= 1
                if in_degree[neighbor] == 0:
                    queue.append(neighbor)

        return topo if len(topo) == n else []   # cycle -> impossible
```

## ⏱️ Complexity Analysis

> [!note]
> - **Time:** O(V + E) — identical cost profile to plain Kahn's algorithm.
> - **Space:** O(V + E) for the adjacency list, O(V) for in-degree, queue, and the result list.

## ⚠️ Common Pitfalls

> [!warning]
> - **Returning `topo` even when it's short.** The size check (`topo.size() == n`) is mandatory — an incomplete topological order is not a valid schedule and must become an empty array, not a partial list.
> - **Reversing the edge direction** — same trap as Course Schedule I: `(a, b)` means `b → a`, not `a → b`.
> - **Recomputing everything with DFS-based topo sort unnecessarily.** Either DFS or BFS topo sort works here; BFS (Kahn's) is used in this note purely because it naturally produces the ordering incrementally while also doubling as the cycle check.
> - **Forgetting `n = 0` or empty prerequisites is a valid (trivial) input** — with no dependencies, any ordering of all `n` tasks (e.g. `0, 1, ..., n-1`) is valid, and the in-degree-0 queue correctly starts with everything.

## 🔗 Related Problems / Next Up

> [!success]
> - **Builds on:** [Topological Sort (BFS / Kahn's Algorithm)](Topological-Sort-BFS-Kahns-Algorithm.md)
> - **Builds on:** [Course Schedule I](Course-Schedule-I.md) — same graph construction and cycle logic, extended to return the order itself
> - [↑ Back to Topological Sort Index](00-Topological-Sort-Index.md)
