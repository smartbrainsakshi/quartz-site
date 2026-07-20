---
title: "Course Schedule I"
tags: [graph, topological-sort, bfs, cycle-detection, kahns-algorithm]
videoLink: "https://www.youtube.com/playlist?list=PLgUwDviBIf0oE3gA41TKO2H5bHpPd7fzn"
difficulty: Medium
---

# Course Schedule I

## 🎯 What problem are we solving?

> [!question]
> Given a total of `n` tasks (numbered `0` to `n-1`) and a list of prerequisite pairs — where a pair `(a, b)` means "to do task `a`, you must first complete task `b`" — determine whether it's **possible to finish all tasks**.

## 💡 Intuition

> [!tip]
> Reframe each prerequisite pair `(a, b)` ("`b` before `a`") as a **directed edge `b → a`**. Now the question "can all tasks be completed given these dependencies" becomes exactly: **"does a valid linear ordering of all tasks exist such that every dependency edge points forward?"** — which is precisely the definition of a **topological sort**.
>
> A topological sort exists **if and only if the dependency graph is a DAG** — i.e., it has no cycle. So the entire problem reduces to: *build the graph from the prerequisite pairs, then check whether a cycle exists.* If a cycle exists (e.g. task 1 needs task 2, task 2 needs task 4, task 4 needs task 1), it's impossible to satisfy every dependency simultaneously — no valid order can exist. If there's no cycle, the tasks can always be arranged in some valid completion order.
>
> Practically: run Kahn's algorithm (BFS-based topological sort). If it successfully orders all `n` tasks, the answer is `true`. If it stalls early (fewer than `n` tasks processed — some cyclic dependency prevented certain tasks from ever reaching in-degree 0), the answer is `false`.

## 🖼️ Visualizing it

```
n = 4, prerequisites: (1,0), (2,1), (3,2)
Meaning: 0 before 1, 1 before 2, 2 before 3
Edges:   0→1, 1→2, 2→3
```

```
in-degree: 0:0  1:1  2:1  3:1

queue = [0]
pop 0 → count=1, neighbor 1: in-degree 0 → push 1
pop 1 → count=2, neighbor 2: in-degree 0 → push 2
pop 2 → count=3, neighbor 3: in-degree 0 → push 3
pop 3 → count=4, no neighbors

count == n (4 == 4) → all tasks can be completed. Answer: true
```

Now a cyclic version: `n = 4`, prerequisites `(1,0), (2,1), (4,2), (1,4)` → edges `0→1, 1→2, 2→4, 4→1`. Node `1` depends on `4`, `4` depends on `2`, `2` depends on `1` — a cycle. Kahn's algorithm would only ever process node `0` (in-degree 0), then stall — `1`, `2`, `4` never reach in-degree 0 because they're all waiting on each other. `count = 1 != 4` → **false**, impossible.

## 🛠️ Algorithm / Approach

> [!abstract]
> 1. Build a directed adjacency list: for each prerequisite pair `(a, b)`, add edge `b → a` (since `b` must come before `a`).
> 2. Run Kahn's algorithm (BFS topological sort) on this graph, counting how many nodes get processed.
> 3. Return `true` if the count equals `n` (all tasks orderable — no cycle), `false` otherwise.

## 👨‍💻 Code

### Java

```java
import java.util.*;

public class CourseScheduleI {

    public boolean canFinish(int n, int[][] prerequisites) {
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

        int count = 0;
        while (!queue.isEmpty()) {
            int node = queue.poll();
            count++;

            for (int neighbor : adj.get(node)) {
                inDegree[neighbor]--;
                if (inDegree[neighbor] == 0) {
                    queue.offer(neighbor);
                }
            }
        }

        return count == n;
    }
}
```

### Python

```python
from collections import deque

class Solution:
    def can_finish(self, n, prerequisites):
        adj = [[] for _ in range(n)]
        in_degree = [0] * n

        for a, b in prerequisites:   # b must come before a
            adj[b].append(a)
            in_degree[a] += 1

        queue = deque([i for i in range(n) if in_degree[i] == 0])
        count = 0

        while queue:
            node = queue.popleft()
            count += 1

            for neighbor in adj[node]:
                in_degree[neighbor] -= 1
                if in_degree[neighbor] == 0:
                    queue.append(neighbor)

        return count == n
```

## ⏱️ Complexity Analysis

> [!note]
> - **Time:** O(V + E) — building the graph is O(E), Kahn's algorithm is O(V + E).
> - **Space:** O(V + E) for the adjacency list, plus O(V) for in-degree and the queue.

## ⚠️ Common Pitfalls

> [!warning]
> - **Reversing the edge direction.** Given pair `(a, b)` meaning "`b` before `a`," the edge must be `b → a`, not `a → b` — getting this backwards silently inverts the entire dependency graph and produces wrong answers on any input with real dependencies.
> - **Using DFS-based cycle detection instead and forgetting it needs `visited` + `pathVisited`** (not just `visited`) — directed graphs can't reuse the simple undirected cycle check.
> - **Not handling `n = 0` or no prerequisites at all** — these trivially return `true` (nothing blocks anything), and the algorithm handles them correctly as long as the loops aren't skipped by an off-by-one mistake.
> - **Forgetting this is a yes/no question**, not "return the order" — that's [Course Schedule II](Course-Schedule-II.md), a near-identical problem with a different return value.

## 🔗 Related Problems / Next Up

> [!success]
> - **Builds on:** [Topological Sort (BFS / Kahn's Algorithm)](Topological-Sort-BFS-Kahns-Algorithm.md), [Detect Cycle in a Directed Graph (BFS / Kahn's)](Detect-Cycle-Directed-BFS.md)
> - **Next up:** [Course Schedule II](Course-Schedule-II.md) — same graph and cycle check, but returns the actual valid order instead of just true/false
> - [↑ Back to Topological Sort Index](00-Topological-Sort-Index.md)
