---
title: "Alien Dictionary"
tags: [graph, topological-sort, bfs, dag, string-processing]
videoLink: "https://www.youtube.com/playlist?list=PLgUwDviBIf0oE3gA41TKO2H5bHpPd7fzn"
difficulty: Hard
---

# Alien Dictionary

## 🎯 What problem are we solving?

> [!question]
> Given a sorted dictionary of `n` words from an **alien language** using the first `k` letters of the English alphabet, figure out the **order of characters** in that alien language. Multiple valid orders may exist for a given input — return any one. (A common follow-up: detect when the dictionary is **invalid**, i.e. no valid order could produce it.)

## 💡 Intuition

> [!tip]
> Since the dictionary is *sorted* according to the (unknown) alien order, comparing **adjacent words** reveals direct ordering constraints, exactly the same way comparing `"apple"` and `"apply"` in English tells you `e` comes before `y` at the first differing position.
>
> For every consecutive pair of words, walk both strings character by character until the **first mismatch**. That mismatch is the *only* useful information in the pair — it says "this character comes before that character" in the alien order. (Everything after the first mismatch is irrelevant — once two words diverge, their relative order is already decided.) Turn each such mismatch into a **directed edge**: `firstChar → secondChar`.
>
> Once every adjacent pair has been converted into an edge, the result is a directed graph over the `k` alphabet characters — and finding a valid character order is now **exactly topological sort**. Any character with no incoming constraints can safely go first, and so on. (Any of the `k` letters that appears in no comparison at all can be placed anywhere, since it has no constraints — it can be treated as an isolated node in the graph.)

## 🖼️ Visualizing it

```
Dictionary (k=4, alphabet is a,b,c,d encoded as 0,1,2,3): "baa", "abcd", "abca", "cab", "cad"
```

```
Compare "baa" vs "abcd": first mismatch at position 0 -> 'b' before 'a'  => edge b -> a
Compare "abcd" vs "abca": positions 0,1,2 match (a,b,c) -> mismatch at 3: 'd' before 'a' => edge d -> a
Compare "abca" vs "cab":  first mismatch at position 0 -> 'a' before 'c' => edge a -> c
Compare "cab"  vs "cad":  positions 0,1 match (c,a) -> mismatch at 2: 'b' before 'd' => edge b -> d
```

Graph: `b → a`, `d → a`, `a → c`, `b → d`

```
Topological sort: b has no incoming edges -> place first
  b's neighbors a, d have their in-degree reduced
  d now has in-degree 0 (only incoming was from b) -> place next
  d's neighbor a has in-degree reduced -> a now in-degree 0 -> place next
  a's neighbor c has in-degree reduced -> c now in-degree 0 -> place last

Order: b, d, a, c
```

**Result: `b d a c`** — one valid alien alphabet order. Verify: `baa` before `abcd` (b < a ✓), `abcd` before `abca` (d < a ✓), `abca` before `cab` (a < c ✓), `cab` before `cad` (b < d ✓).

## 🛠️ Algorithm / Approach

> [!abstract]
> 1. Build an adjacency list over `k` nodes (one per alphabet character, `0..k-1`).
> 2. For every pair of **adjacent** words `(s1, s2)` in the dictionary (indices `i` and `i+1`):
>    - Walk both strings simultaneously up to `min(len(s1), len(s2))`.
>    - At the **first** index where the characters differ, add a directed edge `s1[idx] → s2[idx]` and **stop comparing this pair** (later characters carry no information).
> 3. Run topological sort (DFS-stack or BFS/Kahn's) on this `k`-node graph.
> 4. Convert the resulting node order back into characters — that's the alien alphabet order.
> 5. **(Invalid-dictionary check, if required):** the input is invalid if either (a) some adjacent pair has the shorter word appearing *after* the longer one despite being a strict prefix of it (e.g. `"abcd"` before `"abc"` — a prefix must always come first), or (b) the constructed graph has a **cycle** (no valid topological order exists).

## 👨‍💻 Code

### Java

```java
import java.util.*;

public class AlienDictionary {

    public String findOrder(String[] dictionary, int k) {
        List<List<Integer>> adj = new ArrayList<>();
        for (int i = 0; i < k; i++) adj.add(new ArrayList<>());

        for (int i = 0; i < dictionary.length - 1; i++) {
            String s1 = dictionary[i], s2 = dictionary[i + 1];
            int minLen = Math.min(s1.length(), s2.length());

            for (int ptr = 0; ptr < minLen; ptr++) {
                if (s1.charAt(ptr) != s2.charAt(ptr)) {
                    adj.get(s1.charAt(ptr) - 'a').add(s2.charAt(ptr) - 'a');
                    break;   // only the first mismatch matters
                }
            }
        }

        int[] topo = topoSort(k, adj);   // BFS/Kahn's or DFS-based topo sort

        StringBuilder answer = new StringBuilder();
        for (int node : topo) {
            answer.append((char) ('a' + node));
        }
        return answer.toString();
    }

    private int[] topoSort(int n, List<List<Integer>> adj) {
        int[] inDegree = new int[n];
        for (int i = 0; i < n; i++) {
            for (int neighbor : adj.get(i)) inDegree[neighbor]++;
        }

        Queue<Integer> queue = new LinkedList<>();
        for (int i = 0; i < n; i++) if (inDegree[i] == 0) queue.offer(i);

        int[] result = new int[n];
        int idx = 0;
        while (!queue.isEmpty()) {
            int node = queue.poll();
            result[idx++] = node;
            for (int neighbor : adj.get(node)) {
                if (--inDegree[neighbor] == 0) queue.offer(neighbor);
            }
        }
        return result;
    }
}
```

### Python

```python
from collections import deque

class Solution:
    def find_order(self, dictionary, k):
        adj = [[] for _ in range(k)]

        for s1, s2 in zip(dictionary, dictionary[1:]):
            min_len = min(len(s1), len(s2))
            for ptr in range(min_len):
                if s1[ptr] != s2[ptr]:
                    adj[ord(s1[ptr]) - ord('a')].append(ord(s2[ptr]) - ord('a'))
                    break   # only the first mismatch matters

        in_degree = [0] * k
        for u in range(k):
            for v in adj[u]:
                in_degree[v] += 1

        queue = deque([i for i in range(k) if in_degree[i] == 0])
        topo = []

        while queue:
            node = queue.popleft()
            topo.append(node)
            for neighbor in adj[node]:
                in_degree[neighbor] -= 1
                if in_degree[neighbor] == 0:
                    queue.append(neighbor)

        return ''.join(chr(ord('a') + node) for node in topo)
```

## ⏱️ Complexity Analysis

> [!note]
> - **Time:** O(N × L + K + E) — comparing all adjacent word pairs costs O(N × L) where `L` is the max word length; topological sort on the `k`-node graph costs O(K + E).
> - **Space:** O(K + E) for the adjacency list, plus O(K) for in-degree/queue during topo sort.

## ⚠️ Common Pitfalls

> [!warning]
> - **Comparing every pair of words instead of only adjacent ones.** Only consecutive words in the sorted dictionary carry a direct ordering constraint — comparing all `O(N²)` pairs is both wasteful and can introduce spurious/incorrect edges.
> - **Not stopping at the first mismatch.** Continuing to compare characters after finding one difference can add wrong or redundant edges — only the first differing character in a pair is meaningful.
> - **Forgetting the invalid-dictionary edge case**: a longer word appearing before a shorter word that is its exact prefix (e.g. `"abcd"` before `"abc"`) is never valid in *any* ordering, regardless of the alphabet — this must be checked separately from the graph/cycle logic.
> - **Sizing the graph as 26 nodes instead of `k` nodes.** The problem explicitly restricts the alphabet to the first `k` letters — using all 26 wastes space and can make an incomplete/cyclic subgraph look larger than it is.

## 🔗 Related Problems / Next Up

> [!success]
> - **Builds on:** [Topological Sort (BFS / Kahn's Algorithm)](Topological-Sort-BFS-Kahns-Algorithm.md)
> - This is the last of the 7 Topological Sort section problems — up next: [04 · Shortest Path Algorithms](../04-Shortest-Path-Algorithms/00-Shortest-Path-Index.md)
> - [↑ Back to Topological Sort Index](00-Topological-Sort-Index.md)
