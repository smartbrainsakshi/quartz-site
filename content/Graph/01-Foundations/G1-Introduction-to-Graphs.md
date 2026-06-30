---
title: "G-1. Introduction to Graphs — Types & Conventions"
tags: [graph, foundations, terminology]
videoLink: "https://www.youtube.com/watch?v=M3_pLsDdeuU&list=PLgUwDviBIf0oE3gA41TKO2H5bHpPd7fzn"
---

# G-1. Introduction to Graphs — Types & Conventions

## 🎯 What problem are we solving?

> [!question]
> Before writing a single line of graph code, we need a shared vocabulary. This video doesn't solve a problem — it defines every term (node, edge, degree, path, cycle, DAG...) that the rest of the series will use without re-explaining. Get this note solid and every future video becomes much easier to follow.

## 💡 Intuition

> [!tip]
> A **graph** is the most general way to represent "things and the relationships between them." A tree, a linked list, even a single isolated dot — all of these technically qualify as graphs, because the *only* requirement is:
>
> > a graph = a set of **nodes** + a set of **edges** connecting them.
>
> It does **not** need to be a closed loop, it does **not** need to look circular, and it does **not** need to be "fully connected." That flexibility is exactly why graphs show up everywhere — maps, social networks, dependency chains, etc.

## 🖼️ Visualizing it

### Nodes & Vertices
The circles in a graph are called **nodes** (or **vertices**). They're labeled with numbers, but the numbering order is arbitrary — node 1 doesn't need to connect to node 2 first.

```
   (1)   (2)   (3)   (4)   (5)
```
5 nodes → we write **V = 5** (V = number of vertices).

### Undirected Graph
An **edge** is the line connecting two nodes. In an undirected graph, an edge is bidirectional — "edge between u and v" means you can go u→v **and** v→u.

```
    1 ---- 2
    |      |
    5 ---- 3
     \    /
      \  /
       4
```
Edge (1, 2) is the same thing as edge (2, 1).

### Directed Graph
Only difference: edges now have an arrow, meaning travel is one-way.

```
    1 ----> 4
```
Here you can go 1 → 4, but **not** 4 → 1 — unless a *separate* edge 4 → 1 also exists (graphs can have edges in both directions between the same pair of nodes, but each direction is its own edge).

### Cycles
A **cycle** = start at a node, travel along edges, and return to that **same** node.

```
    1 ---- 2
    |      |
    5 ---- 3
     \    /
      \  /
       4
```
Path `1 → 2 → 3 → 4 → 5 → 1` returns to the start → this graph has a cycle → it's called an **undirected cyclic graph**.

For directed graphs, the same idea applies but you must be able to follow the *arrows* back to the start:

```
    1 ---> 2 ---> 3        (no cycle here — can't get back to 1)
```
This is a **Directed Acyclic Graph (DAG)** — "acyclic" = no cycles. You'll hear "DAG" constantly in upcoming videos (especially Topological Sort).

If you add one more edge so a path leads back to the start, it becomes a **directed cyclic graph** instead.

### Path
A path is a sequence of nodes where:
1. **Every pair of adjacent nodes in the sequence has a direct edge between them.**
2. **No node repeats.**

Using the 5-node graph above:
- `1 → 2 → 3 → 5` — wait, check edges: this is only valid *if* an edge exists between every consecutive pair. ✅ Valid path
- `1 → 2 → 3 → 2 → 1` — ❌ **not** a path (node 2 and node 1 repeat)
- `1 → 3 → 5` — ❌ **not** a path *if* there's no direct edge between 1 and 3 (you can't skip nodes just because they're both "reachable eventually" — adjacency in the sequence must mean adjacency in the graph)

### Degree (Undirected Graphs)
**Degree of a node** = number of edges touching it.

```
    1 ---- 2
    |      |
    5 ---- 3
     \    /
      \  /
       4
```
Edges: (1,2), (2,3), (3,4), (4,5), (5,1), (3,5) → **6 edges total**

| Node | Edges touching it | Degree |
|---|---|---|
| 1 | (1,2), (5,1) | 2 |
| 2 | (1,2), (2,3) | 2 |
| 3 | (2,3), (3,4), (3,5) | **3** |
| 4 | (3,4), (4,5) | 2 |
| 5 | (4,5), (5,1), (3,5) | **3** |

**Sum of all degrees = 2+2+3+2+3 = 12**

> #### 📐 Handshake Property
> **Total degree of a graph = 2 × (number of edges)**
> Here: 2 × 6 = 12 ✅
>
> **Why:** every single edge touches exactly two nodes, so it contributes +1 to *two* different nodes' degree counts — hence the factor of 2. This property is a quick sanity check you'll use to verify graph inputs in problems.

### In-Degree & Out-Degree (Directed Graphs)
For directed graphs, "degree" splits into two:
- **In-degree** = number of edges **coming into** the node
- **Out-degree** = number of edges **going out of** the node

```
    1 ---> 3
    2 ---> 3
    3 ---> 4
```
Node 3 has **in-degree = 2** (from 1 and 2) and **out-degree = 1** (to 4).

### Edge Weight
An edge can optionally carry a **weight** (a number representing cost/distance/time):
```
    1 --(3)--> 2 --(5)--> 3
```
- If a problem doesn't mention weights, **assume every edge has weight 1** (unit weight). This matters a lot later — unweighted shortest paths use BFS, weighted ones need Dijkstra/Bellman-Ford.

## 🛠️ Key Terms — Quick Reference

| Term | Definition |
|---|---|
| Node / Vertex | A single point in the graph |
| Edge | A connection between two nodes |
| Undirected edge | Bidirectional — edge(u,v) ⇒ edge(v,u) too |
| Directed edge | One-way — edge(u,v) does **not** imply edge(v,u) |
| Undirected graph | All edges are undirected |
| Directed graph | All edges are directed |
| Cycle | A path that starts and ends at the same node |
| Acyclic | No cycles exist in the graph |
| DAG | Directed **A**cyclic **G**raph |
| Path | Sequence of nodes, consecutive nodes have an edge, no node repeats |
| Degree (undirected) | Number of edges attached to a node |
| In-degree (directed) | Number of incoming edges to a node |
| Out-degree (directed) | Number of outgoing edges from a node |
| Edge weight | Cost assigned to an edge; defaults to 1 if unspecified |
| V | Number of vertices/nodes |
| E | Number of edges |

*(No code in this video — it's pure terminology. Graph **representation** in code — adjacency list vs adjacency matrix — is covered next in G-2.)*

## ⚠️ Common Pitfalls

> [!warning]
> - **Confusing "graph" with "must be circular."** Trees, single chains, even a lone disconnected node are all valid graphs.
> - **Thinking an edge between u and v in an undirected graph is "one edge stored once."** Conceptually it's one edge, but when you *code* it (next video), you'll store it on **both** node u's and node v's adjacency lists.
> - **Mixing up undirected degree with directed in/out-degree.** Undirected graphs have one degree number; directed graphs split it into two.
> - **Forgetting the path rule that a node can't repeat.** This trips people up later when path-finding problems ask for "simple paths."
> - **Assuming all edges have weight 1 by default in a weighted-graph problem.** Always read the problem statement — if weights are given, use them; only fall back to unit weight when none are specified.

## 🔗 Related Problems / Next Up

> [!success]
> - **Next:** G-2 — Graph Representation (Adjacency List & Adjacency Matrix)
> - [↑ Back to Foundations Index](00-Foundations-Index.md)

