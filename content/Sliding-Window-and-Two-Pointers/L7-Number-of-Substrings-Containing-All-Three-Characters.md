---
title: "L7. Number of Substrings Containing All Three Characters"
tags: [sliding-window, two-pointers, count-subarrays]
videoLink: "https://www.youtube.com/playlist?list=PLgUwDviBIf0q7vrFA_HEWcqRqMpCXzYAL"
difficulty: Medium
---

# L7. Number of Substrings Containing All Three Characters

> 🎯 **TL;DR:** Count substrings containing all of `a`, `b`, `c`. This is Pattern 3, but solved with a **different technique** than the `atMost(K) − atMost(K−1)` trick from L1 — instead, count valid substrings *ending at each index* using a "last seen position" array.

## 🧩 Problem
A string made up only of characters `a`, `b`, `c`. Count the number of substrings that contain **all three** characters at least once.

## 🐢 Brute Force
Standard `i`/`j` double loop. Use a size-3 hash array (one slot per character) to track which characters are present in the current substring; if all three are present, count it.

```text
count = 0
for i = 0 to n-1:
    hasSeen = [0, 0, 0]
    for j = i to n-1:
        hasSeen[s[j] - 'a'] = 1
        if hasSeen[0] + hasSeen[1] + hasSeen[2] == 3:
            count++
```

**Minor optimization:** the moment a substring starting at `i` becomes valid, **every** longer substring from that same `i` is automatically valid too — so you can add `(n - j)` directly to the count and `break`. This helps on many inputs but **doesn't change the worst case** (e.g. a string of all the same character never triggers the early break).

**Complexity:** Time O(N²) · Space O(1) (fixed size-3 array)

## 🥇 Optimal — "Last Seen Index" Per Character (not the generic atMost trick)
Different idea from L1's `atMost(K) − atMost(K−1)` pattern. Instead, **for every ending index, count how many valid substrings end exactly there.**

**Key insight:** track the most recent index at which each character (`a`, `b`, `c`) was last seen. At any position `i`, if all three have been seen at least once, the **earliest valid starting point** for a substring ending at `i` is right after the *earliest* of the three "last seen" positions — that's the bottleneck character whose presence is the most recently established.

Every starting index from `0` up to that minimum last-seen position also produces a valid substring ending at `i`. So the count of valid substrings ending at `i` is:

```text
1 + min(lastSeen[a], lastSeen[b], lastSeen[c])
```

```text
lastSeen = {a: -1, b: -1, c: -1}
count = 0

for i = 0 to n-1:
    lastSeen[s[i]] = i
    if lastSeen[a] != -1 and lastSeen[b] != -1 and lastSeen[c] != -1:
        count += 1 + min(lastSeen[a], lastSeen[b], lastSeen[c])

return count
```

> 💡 You can even skip the explicit "all three seen" check — if any character hasn't been seen yet, its `lastSeen` is still `-1`, and `1 + min(...)` naturally evaluates to `0` in that case.

**Complexity:** Time O(N) — single pass, no inner loop at all · Space O(1) — just 3 fixed variables/array slots.

## 📊 Complexity Summary

| Approach | Time | Space |
|---|---|---|
| Brute Force | O(N²) | O(1) |
| Optimal (last-seen, per-ending-index count) | O(N) | O(1) |

## 💡 Key Takeaways
- Pattern 3 doesn't *only* mean `atMost(K) − atMost(K−1)`. This shows a second core Pattern 3 technique: **count valid windows ending at each index**, using last-seen positions as the bottleneck.
- The general trick — "if a substring is valid, every longer substring sharing the same start is also valid" — is what made the brute-force optimization possible.
- This kind of "last seen index" tracking also showed up in L3 (longest substring without repeats).

## 🔗 Related
- **Previous:** [L6 — Longest Substring With At Most K Distinct Characters](L6-Longest-Substring-With-At-Most-K-Distinct-Characters.md)
- [↑ Back to Sliding Window Index](00-Sliding-Window-Index.md)
