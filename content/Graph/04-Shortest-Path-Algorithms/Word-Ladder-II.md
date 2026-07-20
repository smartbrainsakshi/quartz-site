---
title: "Word Ladder II"
tags: [graph, shortest-path, bfs, backtracking, implicit-graph, string-processing]
videoLink: "https://www.youtube.com/playlist?list=PLgUwDviBIf0oE3gA41TKO2H5bHpPd7fzn"
difficulty: Hard
---

# Word Ladder II

## 🎯 What problem are we solving?

> [!question]
> Same setup as [Word Ladder I](Word-Ladder-I.md): a `startWord`, `targetWord`, and word list, transforming one letter at a time through words in the list. This time, return **every** shortest transformation sequence (not just the length) as a list of word-sequences.

## 💡 Intuition

> [!tip]
> The key structural fact from BFS still holds: **level 1, level 2, level 3, ...** — BFS always explores strictly in order of increasing distance from the source. So the queue this time stores the **entire sequence so far** (not just the current word), and whenever a sequence's last word matches `targetWord`, its length is guaranteed to be the shortest possible — but there might be *multiple* such sequences of that same shortest length, and all of them need collecting.
>
> The critical subtlety versus Word Ladder I: **don't delete a word from the set the instant it's used** — because the same word might be reachable via a *different* sequence at the *same* BFS level (e.g. two different level-2 words both transforming into the same level-3 word). Deleting too early would block that second, equally-valid path. Instead: **process one entire level at a time**, collecting all new words discovered during that level without touching the set yet — only **after the whole level is finished**, delete every word that was used during that level from the set. This guarantees a word is only reachable at its *true* shortest distance, while still allowing multiple parents at the same level to all reach it.

## 🖼️ Visualizing it

```
startWord = "bat", targetWord = "coz"
wordList = {pat, pot, poz, coz, oat, dat, bot, boz}   (simplified sketch)
```

```
Level 1: queue has [["bat"]]

Process "bat": generates "pat" (-> ["bat","pat"]) and "bot" (-> ["bat","bot"])
  do NOT delete pat/bot from the set yet -- level 1 not finished until all level-1
  sequences are processed

-- level boundary reached, level 1 done --
delete "pat" and "bot" from the set now (their level-1 processing is complete)

Level 2: process ["bat","pat"] and ["bat","bot"]
  "pat" -> "pot" (-> ["bat","pat","pot"])
  "bot" -> "pot" (-> ["bat","bot","pot"])   <- SAME word "pot" reached two different ways!

  Because "pot" wasn't deleted mid-level, both parents could reach it.

-- level boundary reached, level 2 done --
delete "pot" from the set now

Level 3: process both ["bat","pat","pot"] and ["bat","bot","pot"]
  "pot" -> "poz" for both -> two sequences: ["bat","pat","pot","poz"], ["bat","bot","pot","poz"]

-- level boundary reached, level 3 done --
delete "poz"

Level 4: "poz" -> "coz" for both sequences -> both reach the target at the same length

Result: [["bat","pat","pot","poz","coz"], ["bat","bot","pot","poz","coz"]]
```

Both sequences are equally short (length 5) and both get returned, because the set deletion was deferred until each full level finished.

## 🛠️ Algorithm / Approach

> [!abstract]
> 1. Put every word from the word list into a set; remove `startWord` from it.
> 2. Push a queue entry containing the single-element sequence `[startWord]`.
> 3. Track a "used this level" list, cleared at the start of each level.
> 4. While the queue isn't empty (processing one BFS level at a time — track the level boundary via the queue's size at the start of the level):
>    - Pop a sequence, look at its **last word**.
>    - Try every one-letter change at every position; if the resulting word is still in the set, append it to a **copy** of the current sequence and push that new sequence. Record the new word as "used this level" (but don't remove it from the set yet).
>    - After the entire level's queue entries have been processed, remove every "used this level" word from the set.
> 5. Whenever a popped sequence's last word equals `targetWord`: if this is the *first* time a sequence has reached the target, record its length as the answer length and add it to the results; if a *later* sequence reaches the target with the **same** length, also add it (BFS levels guarantee nothing longer than the first-found length will ever need to be added).
> 6. Return the collected results.

## 👨‍💻 Code

### Java

```java
import java.util.*;

public class WordLadderII {

    public List<List<String>> findLadders(String startWord, String targetWord, List<String> wordList) {
        Set<String> words = new HashSet<>(wordList);
        List<List<String>> result = new ArrayList<>();

        Queue<List<String>> queue = new LinkedList<>();
        queue.offer(new ArrayList<>(List.of(startWord)));
        words.remove(startWord);

        boolean found = false;

        while (!queue.isEmpty() && !found) {
            Set<String> usedThisLevel = new HashSet<>();
            int levelSize = queue.size();

            for (int i = 0; i < levelSize; i++) {
                List<String> sequence = queue.poll();
                String lastWord = sequence.get(sequence.size() - 1);

                char[] chars = lastWord.toCharArray();
                for (int pos = 0; pos < chars.length; pos++) {
                    char original = chars[pos];
                    for (char c = 'a'; c <= 'z'; c++) {
                        chars[pos] = c;
                        String newWord = new String(chars);

                        if (words.contains(newWord)) {
                            List<String> newSequence = new ArrayList<>(sequence);
                            newSequence.add(newWord);

                            if (newWord.equals(targetWord)) {
                                result.add(newSequence);
                                found = true;
                            }

                            usedThisLevel.add(newWord);
                            queue.offer(newSequence);
                        }
                    }
                    chars[pos] = original;
                }
            }

            words.removeAll(usedThisLevel);   // only delete after the whole level is done
        }
        return result;
    }
}
```

### Python

```python
from collections import deque

class Solution:
    def find_ladders(self, start_word, target_word, word_list):
        words = set(word_list)
        words.discard(start_word)

        queue = deque([[start_word]])
        result = []
        found = False

        while queue and not found:
            used_this_level = set()
            level_size = len(queue)

            for _ in range(level_size):
                sequence = queue.popleft()
                last_word = sequence[-1]

                for i in range(len(last_word)):
                    for c in 'abcdefghijklmnopqrstuvwxyz':
                        new_word = last_word[:i] + c + last_word[i+1:]

                        if new_word in words:
                            new_sequence = sequence + [new_word]

                            if new_word == target_word:
                                result.append(new_sequence)
                                found = True

                            used_this_level.add(new_word)
                            queue.append(new_sequence)

            words -= used_this_level   # only delete after the whole level is done

        return result
```

## ⏱️ Complexity Analysis

> [!note]
> - **Time:** Hard to bound tightly — depends heavily on how many shortest sequences exist for a given input, which varies per test case. In the worst case it's substantially more than Word Ladder I, since multiple full sequences (not just word counts) are tracked and copied.
> - **Space:** O(N × L) for the set and the sequences stored in the queue and result, where multiple full-length sequences may coexist at once.

## ⚠️ Common Pitfalls

> [!warning]
> - **Deleting a word from the set the moment it's used** (as in Word Ladder I) — this is the single most important difference from Word Ladder I. Doing so here would block a second, equally valid parent at the same level from ever reaching that word, silently dropping valid shortest sequences from the result.
> - **Not tracking level boundaries explicitly.** Without processing one full level (via `queue.size()` snapshotted at the top of the loop) before deleting used words, the algorithm degenerates into the naive per-word deletion bug above.
> - **Copying the sequence incorrectly (aliasing instead of cloning).** Each new sequence must be an independent copy — appending to a shared list reference across multiple branches will corrupt unrelated sequences.
> - **Continuing to search deeper levels after the target has been found at the current level.** Once any sequence reaches `targetWord`, no sequence from a *later* level could possibly be shorter — stop expanding further levels (though same-level sequences reaching the target should still all be collected).

## 🔗 Related Problems / Next Up

> [!success]
> - **Builds on:** [Word Ladder I](Word-Ladder-I.md) — same implicit BFS graph, extended to track and return full sequences instead of just a count
> - **Bonus (not required for interviews):** [Word Ladder II — Optimized Two-Pass Approach](Word-Ladder-II-Optimized.md) — a competitive-programming-style optimization using BFS distances + backward DFS reconstruction, needed only to pass stricter online judge time limits
> - [↑ Back to Shortest Path Algorithms Index](00-Shortest-Path-Index.md)
