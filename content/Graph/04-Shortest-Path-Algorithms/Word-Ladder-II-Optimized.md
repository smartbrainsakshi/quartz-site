---
title: "Word Ladder II — Optimized Two-Pass Approach (Bonus)"
tags: [graph, shortest-path, bfs, dfs, backtracking, implicit-graph, competitive-programming]
videoLink: "https://www.youtube.com/playlist?list=PLgUwDviBIf0oE3gA41TKO2H5bHpPd7fzn"
difficulty: Hard
---

# Word Ladder II — Optimized Two-Pass Approach (Bonus)

> [!info]
> **This note is a bonus / competitive-programming technique, not required for interviews.** The straightforward [Word Ladder II](Word-Ladder-II.md) approach (BFS storing full sequences) is what should be presented in an interview. This optimized version exists purely because some online judges tightened time limits enough that the straightforward approach times out — it's documented here for the problem-solving technique itself, not as interview-recommended practice.

## 🎯 What problem are we solving?

> [!question]
> Same problem as Word Ladder II — return every shortest transformation sequence from `startWord` to `targetWord`. The straightforward BFS-with-full-sequences approach can time out on stricter judges because it carries and copies **entire sequences** through the queue, including many that turn out to be dead ends.

## 💡 Intuition

> [!tip]
> The wasted work in the straightforward approach comes from exploring **forward** from the start — many branches explored this way never lead anywhere useful, but their sequences still get carried around at real cost.
>
> Split the problem into two independent steps:
> 1. **Step 1 — BFS for distances only.** Run a plain BFS (like Word Ladder I) from `startWord`, but instead of tracking sequences, just record **the level (distance) at which each word was first reached**, in a map. This is cheap: no sequence copying, just a word → distance map.
> 2. **Step 2 — reconstruct paths by walking *backward* from the target.** Starting at `targetWord`, do a DFS trying to move to a neighboring word (one letter different) whose recorded distance is **exactly one less** than the current word's distance. Every time this backward walk reaches `startWord`, that's one complete valid shortest sequence — record it (reversed, since it was built backward).
>
> Why walking backward from the target is so much cheaper: going forward from the start, *many* words at each level might be dead ends that never reach the target — all of those branches still get explored and paid for. Going backward from the target, **only the words that are actually part of some shortest path can ever be reached at all**, because the DFS only follows edges into strictly decreasing recorded distances that trace back toward the source — there's no way to wander into an irrelevant branch, so the exploration is confined to exactly the part of the graph that matters.

## 🖼️ Visualizing it

```
startWord = "hit", targetWord = "cog", distances computed via Step 1's BFS:
  hit:0, hot:1, dot:2, lot:2, dog:3, log:3, cog:4
```

```
Step 2 - DFS backward from "cog" (distance 4):
  try neighbors of "cog" with distance 3: "log" (yes, dist 3) and "dog" (yes, dist 3)

  branch A: dfs("log", sequence=["cog"])
    neighbors of "log" with distance 2: "lot" (yes)
      dfs("lot", sequence=["cog","log"])
        neighbors of "lot" with distance 1: "hot" (yes)
          dfs("hot", sequence=["cog","log","lot"])
            neighbors of "hot" with distance 0: "hit" (yes) -- this IS startWord!
              record reversed sequence: "hit","hot","lot","log","cog"

  branch B: dfs("dog", sequence=["cog"])
    neighbors of "dog" with distance 2: "dot" (yes)
      dfs("dot", ...) -> ... -> reaches "hit" via "hot"
      record reversed sequence: "hit","hot","dot","dog","cog"
```

Both sequences are found, but the backward DFS never had to explore any word that *wasn't* exactly one step closer to the source at each stage — no wasted branches.

## 🛠️ Algorithm / Approach

> [!abstract]
> **Step 1 (BFS — distances only):**
> 1. Put the word list into a set; run BFS from `startWord`, storing `distance[word] = level` for every word the first time it's reached (delete from the set as usual, exactly like Word Ladder I, since only the *first* — shortest — distance matters here).
>
> **Step 2 (backward DFS — reconstruct sequences):**
> 2. `dfs(word, path)`: if `word == startWord`, the reversed `path + [word]` is a complete valid sequence — record it, and return.
> 3. Otherwise, for every one-letter variation `prev` of `word`: if `prev` exists in the distance map **and** `distance[prev] == distance[word] - 1`, recursively call `dfs(prev, path + [word])`.
> 4. Start the backward DFS from `targetWord` with an empty path, collect all sequences found, reverse each one before returning it (since they were built backward).

## 👨‍💻 Code

### Java

```java
import java.util.*;

public class WordLadderIIOptimized {

    public List<List<String>> findLadders(String startWord, String targetWord, List<String> wordList) {
        Set<String> words = new HashSet<>(wordList);
        Map<String, Integer> distance = new HashMap<>();

        // Step 1: BFS for distances only
        Queue<String> queue = new LinkedList<>();
        queue.offer(startWord);
        distance.put(startWord, 0);
        words.remove(startWord);

        while (!queue.isEmpty()) {
            String word = queue.poll();
            char[] chars = word.toCharArray();

            for (int i = 0; i < chars.length; i++) {
                char original = chars[i];
                for (char c = 'a'; c <= 'z'; c++) {
                    chars[i] = c;
                    String newWord = new String(chars);
                    if (words.contains(newWord)) {
                        words.remove(newWord);
                        distance.put(newWord, distance.get(word) + 1);
                        queue.offer(newWord);
                    }
                }
                chars[i] = original;
            }
        }

        // Step 2: backward DFS from targetWord
        List<List<String>> result = new ArrayList<>();
        if (!distance.containsKey(targetWord)) return result;   // unreachable

        dfsBackward(targetWord, startWord, new ArrayList<>(), distance, result);
        for (List<String> sequence : result) Collections.reverse(sequence);
        return result;
    }

    private void dfsBackward(String word, String startWord, List<String> path,
                              Map<String, Integer> distance, List<List<String>> result) {
        path.add(word);

        if (word.equals(startWord)) {
            result.add(new ArrayList<>(path));
        } else {
            char[] chars = word.toCharArray();
            for (int i = 0; i < chars.length; i++) {
                char original = chars[i];
                for (char c = 'a'; c <= 'z'; c++) {
                    chars[i] = c;
                    String prevWord = new String(chars);
                    if (distance.containsKey(prevWord) && distance.get(prevWord) == distance.get(word) - 1) {
                        dfsBackward(prevWord, startWord, path, distance, result);
                    }
                }
                chars[i] = original;
            }
        }
        path.remove(path.size() - 1);   // backtrack
    }
}
```

### Python

```python
from collections import deque

class Solution:
    def find_ladders(self, start_word, target_word, word_list):
        words = set(word_list)
        distance = {start_word: 0}
        words.discard(start_word)

        # Step 1: BFS for distances only
        queue = deque([start_word])
        while queue:
            word = queue.popleft()
            for i in range(len(word)):
                for c in 'abcdefghijklmnopqrstuvwxyz':
                    new_word = word[:i] + c + word[i+1:]
                    if new_word in words:
                        words.discard(new_word)
                        distance[new_word] = distance[word] + 1
                        queue.append(new_word)

        result = []
        if target_word not in distance:
            return result   # unreachable

        # Step 2: backward DFS from target_word
        def dfs_backward(word, path):
            path.append(word)
            if word == start_word:
                result.append(list(reversed(path)))
            else:
                for i in range(len(word)):
                    for c in 'abcdefghijklmnopqrstuvwxyz':
                        prev_word = word[:i] + c + word[i+1:]
                        if distance.get(prev_word) == distance[word] - 1:
                            dfs_backward(prev_word, path)
            path.pop()   # backtrack

        dfs_backward(target_word, [])
        return result
```

## ⏱️ Complexity Analysis

> [!note]
> - **Time:** Step 1 is O(N × L × 26), identical to Word Ladder I. Step 2's cost depends on how many nodes actually lie on some shortest path — strictly fewer wasted branches than the forward approach, though still hard to bound tightly in the worst case.
> - **Space:** O(N) for the distance map, plus O(N × L) for the collected result sequences.

## ⚠️ Common Pitfalls

> [!warning]
> - **Presenting this in an interview instead of the straightforward Word Ladder II approach.** This technique trades simplicity for constant-factor speed — appropriate for competitive programming judges, but the two-pass/backward-DFS structure is harder to explain and reason about live, so the straightforward version is what's expected in an interview setting.
> - **Forgetting to reverse each collected sequence** at the end — since the DFS walks backward from `targetWord` to `startWord`, every recorded path is in reverse order until explicitly flipped.
> - **Checking `distance[prev] == distance[word] - 1` incorrectly (e.g. using `<=` instead of exact equality).** Only neighbors exactly one level closer to the source are valid steps in a shortest path — anything else would either revisit the same level or skip past it.
> - **Forgetting to backtrack (`path.pop()` / `path.remove(...)`) after each DFS branch** — without this, the shared `path` list leaks entries between sibling branches, corrupting unrelated recorded sequences.

## 🔗 Related Problems / Next Up

> [!success]
> - **Builds on:** [Word Ladder II](Word-Ladder-II.md) — same problem, restructured into BFS-for-distances + backward-DFS-for-paths to reduce wasted exploration
> - [↑ Back to Shortest Path Algorithms Index](00-Shortest-Path-Index.md)
