---
title: "Dijkstra's Algorithm — Why a Priority Queue (Not a Queue), and Time Complexity"
tags: [graph, shortest-path, dijkstra, complexity-analysis]
videoLink: "https://www.youtube.com/playlist?list=PLgUwDviBIf0oE3gA41TKO2H5bHpPd7fzn"
difficulty: Medium
---

# Dijkstra's Algorithm — Why a Priority Queue (Not a Queue), and Time Complexity

## 🎯 What problem are we solving?

> [!question]
> Two follow-up questions naturally arise after learning Dijkstra's algorithm: (1) a plain FIFO queue instead of a priority queue still produces the *correct* answer — so why bother with the heap at all? (2) where does the commonly-quoted **O(E log V)** time complexity actually come from?

## 💡 Intuition

> [!tip]
> **Why a priority queue, not a plain queue:** swapping the min-heap for a plain FIFO queue and re-running the exact same relaxation logic still eventually computes the correct shortest distances — nothing about correctness depends on the ordering. What changes is *efficiency*. With a plain queue, a node might get discovered first via a long, suboptimal path and get fully expanded (relaxing all of *its* neighbors) before the actual short path to that same node is even found — meaning some of that expansion work has to be redone once a better distance arrives later. A priority queue prevents this entirely: because it always pops the globally smallest pending distance next, **a node's distance is guaranteed final at the moment it's popped** — there's no way a cheaper path to it could still be sitting unexplored elsewhere in the queue, since anything currently pending is provably ≥ what was just popped. Not needing to worry about "did I explore this too early" is precisely what saves the redundant work a plain queue leaves on the table.
>
> **Time complexity — O(E log V):** the pseudocode shape is: `while pq not empty { pop; for each neighbor { relax; maybe push } }`. The outer `while` loop runs, at worst, once per "settle" of a node — bounded by `V`. For each pop, the inner loop iterates over that node's edges, and — worst case — a **dense graph** (every node connected to every other) gives each node up to `V - 1` outgoing edges. So the raw shape is `V × (V - 1) × log(heap size)`.
>
> What's the heap size, worst case? Every time a relaxation improves a distance, a new entry gets pushed — even for the *same* node, repeatedly, as better and better distances are found through different neighbors during the traversal. In the most extreme hypothetical (every node pushes up to `V - 1` entries), the heap can hold on the order of `V²` entries, so `log(heap size) ≈ log(V²) = 2 log V`.
>
> Multiplying out: `V × (V - 1) × 2 log V`. Since a dense graph has `E = V × (V - 1)` total edges, this simplifies to `E × 2 log V`, which is `O(E log V)` — the constant factor drops out in Big-O notation.

## 🖼️ Visualizing it

```
Plain-queue inefficiency, illustrated:

Graph: 0-1(3), 0-2(1), 1-3(4), 2-3(2), 3-1(4)

Using a QUEUE (FIFO, not priority):
  pop (0,0): push (3,1), (1,2)
  pop (3,1): push (7,3) [via 1->3, distance 3+4=7]
  pop (1,2): push (3,3) [via 2->3, distance 1+2=3 -- BETTER than 7, but 7 was already queued and will still be processed]
  pop (7,3): relax 3's neighbors using the WORSE distance 7 -- wasted work
  pop (3,3): relax 3's neighbors again using the better distance 3 -- the correct work

Using a PRIORITY QUEUE:
  pop (0,0): push (3,1), (1,2)
  pop (1,2): [smallest first] push (3,3)   <- distance 3 to node 3 discovered immediately
  pop (3,1): relax normally
  pop (3,3): relax using the already-optimal distance -- no wasted expansion ever happens
```

Both approaches land on the same final distances — the queue version just does avoidable extra work processing node 3 with a stale, worse distance first.

## 🛠️ Algorithm / Approach

> [!abstract]
> No new algorithm here — this note explains *why* the priority queue in [Dijkstra's Algorithm (Using a Priority Queue)](Dijkstra-Algorithm-Using-Priority-Queue.md) matters, and derives its complexity:
> 1. **Correctness** holds regardless of queue type, because relaxation is checked with an explicit `<` comparison every time — a queue never silently accepts a worse distance.
> 2. **Efficiency** depends entirely on processing order: priority-first processing guarantees no node is ever expanded using a non-final distance.
> 3. **Complexity derivation**: `V` pops × up to `V-1` edges per pop × `O(log(heap size))` per push, where heap size is bounded by `O(V²)` in the densest case, collapsing to `O(E log V)`.

## 👨‍💻 Code

> [!info]
> This note is conceptual — no new implementation. The only "code change" being discussed is swapping one data structure for another inside the same loop shape:

### Java

```java
// Priority-queue version (correct AND efficient):
PriorityQueue<int[]> pq = new PriorityQueue<>((a, b) -> a[0] - b[0]);

// Plain-queue version (still correct, but does redundant re-expansion):
Queue<int[]> queue = new LinkedList<>();
```

### Python

```python
# Priority-queue version (correct AND efficient):
import heapq
pq = []
heapq.heappush(pq, (0, source))

# Plain-queue version (still correct, but does redundant re-expansion):
from collections import deque
queue = deque([(0, source)])
```

## ⏱️ Complexity Analysis

> [!note]
> - **With a priority queue:** O(E log V) — derived above from `V × (V-1) × 2 log V` collapsing via `E = V × (V-1)` in the dense-graph worst case.
> - **With a plain queue:** still polynomial and still terminates correctly, but does provably more redundant relaxation work in graphs where a node is first reached via a suboptimal path — no clean closed-form improvement over the heap version is gained by using it.

## ⚠️ Common Pitfalls

> [!warning]
> - **Concluding a plain queue is "wrong."** It isn't — it's *correct but inefficient*. This distinction matters in an interview: dismissing the queue approach as broken is itself a wrong answer.
> - **Misremembering the complexity as just "E log E" or "V log V" without being able to justify it.** Being able to reproduce the `V × (V-1) × log(heap size)` → `E log V` derivation is what separates memorized trivia from real understanding.
> - **Forgetting the heap-size bound is a worst-case hypothetical**, not something that happens on every graph — sparse graphs will have far fewer redundant heap entries in practice.
> - **Confusing "settling a node" with "popping a node."** A node can be popped multiple times (once per stale entry) before its truly final distance is popped — the `V` bound in the derivation refers to the number of *distinct* nodes, not the number of pops.

## 🔗 Related Problems / Next Up

> [!success]
> - **Builds on:** [Dijkstra's Algorithm (Using a Priority Queue)](Dijkstra-Algorithm-Using-Priority-Queue.md)
> - [↑ Back to Shortest Path Algorithms Index](00-Shortest-Path-Index.md)
