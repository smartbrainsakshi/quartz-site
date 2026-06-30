---
title: "G-2. Graph Representation — Adjacency Matrix & Adjacency List"
tags: [graph, foundations, representation, adjacency-matrix, adjacency-list]
videoLink: "https://www.youtube.com/watch?v=M3_pLsDdeuU&list=PLgUwDviBIf0oE3gA41TKO2H5bHpPd7fzn"
---

# G-2. Graph Representation — Adjacency Matrix & Adjacency List

## 🎯 What problem are we solving?

> [!question]
> A graph problem always starts the same way: you're handed a list of edges as input, and you need to store them in a data structure that lets future algorithms (BFS, DFS, Dijkstra...) quickly answer "who are node X's neighbors?" This video covers the two standard ways to do that — **Adjacency Matrix** and **Adjacency List** — and why the list is almost always preferred.

## 💡 Intuition

> [!tip]
> Every graph problem's input follows the same shape:
>
> ```
> n m            ← n = number of nodes, m = number of edges
> u1 v1          ← edge between u1 and v1
> u2 v2          ← edge between u2 and v2
> ...            ← m lines total
> um vm
> ```
>
> `n` is fixed once given, but `m` (number of edges) can be anything — it's not tied to `n` by any formula. Edges can appear in any order in the input.
>
> Once you have these edges, you need to *store* them so that "is there an edge between A and B?" or "what are A's neighbors?" can be answered fast. There are exactly two standard ways to do this — trading off **space** vs **lookup style**.

## 🖼️ Visualizing it

Take this 5-node, 6-edge undirected graph (1-indexed):

```
edges: (1,2) (1,3) (2,4) (3,4) (2,5) (4,5)

    1 ---- 2 ---- 5
    |      |      |
    3 -----4------+
```

### Method 1 — Adjacency Matrix
Build an `(n+1) × (n+1)` 2D array (1-indexed, so index 0 row/col is just unused padding). For every edge `(u, v)`, set `matrix[u][v] = 1` **and** `matrix[v][u] = 1` (undirected ⇒ symmetric).

```
      0  1  2  3  4  5
   0  .  .  .  .  .  .
   1  .  0  1  1  0  0
   2  .  1  0  0  1  1
   3  .  1  0  0  1  0
   4  .  0  1  1  0  1
   5  .  0  1  0  1  0
```
Now `matrix[3][4] == 1` instantly tells you "yes, edge exists between 3 and 4" — O(1) lookup. But notice how much of this grid is wasted on `0`s when the graph is sparse (few edges relative to possible pairs).

- **Space used:** `(n+1) × (n+1)` ≈ **O(n²)** — regardless of how many edges actually exist. This is the matrix's biggest weakness.

### Method 2 — Adjacency List
Instead of a full grid, keep an array of `n+1` **lists** (vectors). For edge `(u, v)`: append `v` to `list[u]`, and append `u` to `list[v]`.

```
index:    0    1      2        3      4        5
list:    []  [2,3]  [1,4,5]  [1,4]  [2,3,5]  [2,4]
```

To find node 4's neighbors, you just read `list[4] = [2, 3, 5]` directly — no scanning through unrelated pairs.

- **Space used:** every edge contributes exactly 2 entries (one per endpoint) → **O(2·E)**, i.e. **O(E)**. Far cheaper than O(n²) when the graph is sparse — which most real graphs are.

> **This is why adjacency list is the default choice for almost every graph problem going forward in this series** — matrix is only worth it when the graph is dense or you need O(1) "does edge (u,v) exist" lookups very frequently.

### Directed Graphs
For a directed edge `u → v`, you only add `v` to `list[u]` — **not** the reverse. So:
- Matrix: only `matrix[u][v] = 1` (not symmetric)
- List: only `list[u].push(v)`
- **Space for directed adjacency list: O(E)** (not O(2E), since each edge is stored once, not twice)

### Weighted Graphs
When edges carry weights, you store the weight alongside the neighbor:
- **Matrix:** `matrix[u][v] = weight` instead of `1`
- **List:** each entry becomes a **pair** `(neighbor, weight)` instead of a plain integer

```
index:    4
list:    [(2, 1), (3, 4), (5, 3)]   ← (neighbor, weight) pairs
```

## 🛠️ Algorithm / Approach

> [!abstract]
> **Building an adjacency list (undirected, unweighted):**
> 1. Read `n`, `m`.
> 2. Create `n+1` empty lists.
> 3. For each of the `m` edges `(u, v)`:
>    - Add `v` to `adj[u]`
>    - Add `u` to `adj[v]` (skip this line for directed graphs)
> 4. Done — `adj[i]` now holds all neighbors of node `i`.

## 👨‍💻 Code

### Java

```java
import java.util.*;

public class GraphRepresentation {

    // ---------- Adjacency List: Undirected, Unweighted ----------
    public static List<List<Integer>> buildAdjList(int n, int[][] edges) {
        List<List<Integer>> adj = new ArrayList<>();
        for (int i = 0; i <= n; i++) adj.add(new ArrayList<>());

        for (int[] edge : edges) {
            int u = edge[0], v = edge[1];
            adj.get(u).add(v);
            adj.get(v).add(u);   // omit this line for a directed graph
        }
        return adj;
    }

    // ---------- Adjacency Matrix: Undirected, Unweighted ----------
    public static int[][] buildAdjMatrix(int n, int[][] edges) {
        int[][] matrix = new int[n + 1][n + 1];
        for (int[] edge : edges) {
            int u = edge[0], v = edge[1];
            matrix[u][v] = 1;
            matrix[v][u] = 1;    // omit this line for a directed graph
        }
        return matrix;
    }

    // ---------- Adjacency List: Weighted ----------
    // Each neighbor stored as int[]{neighbor, weight}
    public static List<List<int[]>> buildWeightedAdjList(int n, int[][] edges) {
        List<List<int[]>> adj = new ArrayList<>();
        for (int i = 0; i <= n; i++) adj.add(new ArrayList<>());

        for (int[] edge : edges) {
            int u = edge[0], v = edge[1], w = edge[2];
            adj.get(u).add(new int[]{v, w});
            adj.get(v).add(new int[]{u, w});   // omit for directed
        }
        return adj;
    }

    public static void main(String[] args) {
        int n = 5, m = 6;
        int[][] edges = {{1,2}, {1,3}, {2,4}, {3,4}, {2,5}, {4,5}};

        List<List<Integer>> adjList = buildAdjList(n, edges);
        for (int i = 1; i <= n; i++) {
            System.out.println(i + " -> " + adjList.get(i));
        }
    }
}
```

### Python

```python
# ---------- Adjacency List: Undirected, Unweighted ----------
def build_adj_list(n, edges):
    adj = [[] for _ in range(n + 1)]
    for u, v in edges:
        adj[u].append(v)
        adj[v].append(u)        # omit this line for a directed graph
    return adj

# ---------- Adjacency Matrix: Undirected, Unweighted ----------
def build_adj_matrix(n, edges):
    matrix = [[0] * (n + 1) for _ in range(n + 1)]
    for u, v in edges:
        matrix[u][v] = 1
        matrix[v][u] = 1         # omit this line for a directed graph
    return matrix

# ---------- Adjacency List: Weighted ----------
# Each neighbor stored as a (neighbor, weight) tuple
def build_weighted_adj_list(n, edges):
    adj = [[] for _ in range(n + 1)]
    for u, v, w in edges:
        adj[u].append((v, w))
        adj[v].append((u, w))    # omit this line for a directed graph
    return adj

if __name__ == "__main__":
    n, m = 5, 6
    edges = [(1, 2), (1, 3), (2, 4), (3, 4), (2, 5), (4, 5)]

    adj_list = build_adj_list(n, edges)
    for i in range(1, n + 1):
        print(i, "->", adj_list[i])
```

## ⏱️ Complexity Analysis

> [!note]
> | Representation | Space | Build Time | "Edge (u,v) exists?" |
> |---|---|---|---|
> | Adjacency Matrix | O(n²) | O(n²) (or O(E) if only filling edges) | O(1) |
> | Adjacency List (undirected) | O(2·E) ≈ O(E) | O(E) | O(degree of u) |
> | Adjacency List (directed) | O(E) | O(E) | O(degree of u) |
>
> **Why O(2E) for undirected lists:** every edge has two endpoints, and you store the edge once in *each* endpoint's list — so total entries = 2 × (number of edges).
>
> **Why directed lists only cost O(E):** an edge `u → v` is only ever stored in `adj[u]`, never duplicated into `adj[v]`.

## ⚠️ Common Pitfalls

> [!warning]
> - **Sizing the matrix/list as `n` instead of `n+1`** when using 1-indexed nodes — off-by-one errors are extremely common here. If nodes are numbered `1..n`, allocate `n+1` slots so index `n` is valid.
> - **Adding the edge to both `adj[u]` and `adj[v]` for a directed graph** — this silently turns your directed graph into an undirected one and will break every algorithm downstream.
> - **Defaulting to adjacency matrix "because it's simpler"** — for sparse graphs (most interview problems), this wastes huge amounts of space and will TLE/MLE on large inputs. Default to adjacency list unless you specifically need O(1) edge-existence checks.
> - **Forgetting to store weights as pairs** when the graph is weighted — storing only the neighbor loses the weight information you'll need for Dijkstra/Prim's later.

## 🔗 Related Problems / Next Up

> [!success]
> - **Previous:** [G-1 — Introduction to Graphs](G1-Introduction-to-Graphs.md)
> - **Next:** G-3 — BFS Traversal
> - [↑ Back to Foundations Index](00-Foundations-Index.md)

