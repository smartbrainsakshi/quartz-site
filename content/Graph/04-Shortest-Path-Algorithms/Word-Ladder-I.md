---
title: "Word Ladder I"
tags: [graph, shortest-path, bfs, implicit-graph, string-processing]
videoLink: "https://www.youtube.com/playlist?list=PLgUwDviBIf0oE3gA41TKO2H5bHpPd7fzn"
difficulty: Hard
---

# Word Ladder I

## 🎯 What problem are we solving?

> [!question]
> Given a `startWord`, a `targetWord`, and a word list, find the length of the **shortest transformation sequence** from `startWord` to `targetWord`, where each step changes **exactly one letter**, and every intermediate word (including the target) must exist in the word list. Return `0` if no such sequence exists.

## 💡 Intuition

> [!tip]
> This doesn't look like a graph problem at first — there's no explicit adjacency list — but it *is* one: think of every word in the list as a **node**, with an **implicit edge** between two words if they differ by exactly one character. The question "shortest transformation sequence" is then literally "**shortest path** from `startWord` to `targetWord` in this implicit, unweighted graph" — and shortest path in an unweighted graph is exactly what **BFS** computes.
>
> The brute-force way to discover a word's neighbors: for every position in the current word, try replacing that character with every letter from `a` to `z`, and check if the result exists in the word list (using a **set** for O(1) lookup instead of scanning the list). Run standard BFS from `startWord`, where the queue stores `(word, stepsSoFar)` pairs — the first time `targetWord` is popped, `stepsSoFar` is the answer, since BFS explores level by level and guarantees the first arrival is the shortest.
>
> One important detail: instead of a separate `visited` array, this problem **removes a word from the set the moment it's used** — functionally identical to marking visited, but reusing the set that's needed anyway for existence checks.

## 🖼️ Visualizing it

```
startWord = "hit", targetWord = "cog"
wordList = {hot, dot, dog, lot, log, cog}
```

```
queue = [("hit", 1)]

pop ("hit", 1): try changing each letter -> only "hot" exists in the set
  push ("hot", 2), remove "hot" from set

pop ("hot", 2): try changing each letter -> "dot" and "lot" exist
  push ("dot", 3), push ("lot", 3), remove both from set

pop ("dot", 3): try changing each letter -> "dog" exists
  push ("dog", 4), remove "dog" from set

pop ("lot", 3): try changing each letter -> "log" exists
  push ("log", 4), remove "log" from set

pop ("dog", 4): try changing each letter -> "cog" exists
  push ("cog", 5), remove "cog" from set

pop ("log", 4): try changing each letter -> "cog" no longer in set (already used) -> nothing new

pop ("cog", 5): this IS the target word -> answer = 5
```

**Result: 5** — `hit → hot → dot → dog → cog` (or `hit → hot → lot → log → cog`, an equally short alternative path that BFS also would have found had it been reached first).

## 🛠️ Algorithm / Approach

> [!abstract]
> 1. Put every word from the word list into a **set** for O(1) existence checks.
> 2. Push `(startWord, 1)` into a BFS queue. (Do **not** need to add `startWord` to the set removal step if it wasn't originally in the list — but if it was, remove it, mirroring "mark visited.")
> 3. While the queue isn't empty:
>    - Pop `(word, steps)`.
>    - If `word == targetWord`, return `steps`.
>    - For each position `i` in `word`, and each letter `c` from `a` to `z`: form `newWord` by replacing position `i` with `c`. If `newWord` exists in the set: push `(newWord, steps + 1)` and **remove `newWord` from the set** (so it's never re-explored).
> 4. If the queue empties without ever popping `targetWord`, return `0`.

## 👨‍💻 Code

### Java

```java
import java.util.*;

public class WordLadderI {

    public int ladderLength(String startWord, String targetWord, List<String> wordList) {
        Queue<Pair> queue = new LinkedList<>();
        queue.offer(new Pair(startWord, 1));

        Set<String> words = new HashSet<>(wordList);
        words.remove(startWord);

        while (!queue.isEmpty()) {
            Pair current = queue.poll();
            String word = current.word;
            int steps = current.steps;

            if (word.equals(targetWord)) return steps;

            char[] chars = word.toCharArray();
            for (int i = 0; i < chars.length; i++) {
                char original = chars[i];
                for (char c = 'a'; c <= 'z'; c++) {
                    chars[i] = c;
                    String newWord = new String(chars);
                    if (words.contains(newWord)) {
                        words.remove(newWord);
                        queue.offer(new Pair(newWord, steps + 1));
                    }
                }
                chars[i] = original;
            }
        }
        return 0;
    }

    private static class Pair {
        String word;
        int steps;
        Pair(String word, int steps) { this.word = word; this.steps = steps; }
    }
}
```

### Python

```python
from collections import deque

class Solution:
    def ladder_length(self, start_word, target_word, word_list):
        words = set(word_list)
        words.discard(start_word)

        queue = deque([(start_word, 1)])

        while queue:
            word, steps = queue.popleft()

            if word == target_word:
                return steps

            for i in range(len(word)):
                for c in 'abcdefghijklmnopqrstuvwxyz':
                    new_word = word[:i] + c + word[i+1:]
                    if new_word in words:
                        words.discard(new_word)
                        queue.append((new_word, steps + 1))

        return 0
```

## ⏱️ Complexity Analysis

> [!note]
> - **Time:** O(N × L × 26) — where `N` is the number of words and `L` is word length: each word (at most once, since it's removed from the set after use) tries `L` positions × 26 letters.
> - **Space:** O(N) for the set and the queue.

## ⚠️ Common Pitfalls

> [!warning]
> - **Using a linear scan of the word list instead of a set** to check existence — this turns an O(1) lookup into O(N), making the whole algorithm far slower.
> - **Forgetting to remove a word from the set once it's used/queued.** Without this, the same word can be re-discovered and re-queued from multiple paths, causing incorrect (larger) answers or infinite-ish blowup, since nothing marks it as already handled.
> - **Not restoring the original character after trying all 26 replacements at a position** (`chars[i] = original` in the Java version) — forgetting this corrupts the word for the next position's iteration.
> - **Checking `word == targetWord` only after the BFS ends instead of as soon as it's popped.** Since BFS guarantees the first pop of `targetWord` is via the shortest path, returning immediately at that point is both correct and avoids unnecessary further work.

## 🔗 Related Problems / Next Up

> [!success]
> - **Builds on:** [G4. BFS Traversal](../01-Foundations/G4-BFS-Traversal.md), [Shortest Path in an Undirected Graph with Unit Weights](Shortest-Path-Undirected-Unit-Weights.md) — same "BFS = shortest path in unweighted graph" idea, applied to an implicit graph instead of an explicit one
> - **Next up:** [Word Ladder II](Word-Ladder-II.md) — same setup, but return *every* shortest transformation sequence, not just the length
> - [↑ Back to Shortest Path Algorithms Index](00-Shortest-Path-Index.md)
