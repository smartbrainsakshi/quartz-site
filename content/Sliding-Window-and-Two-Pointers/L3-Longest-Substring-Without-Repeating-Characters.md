---
title: "L3. Longest Substring Without Repeating Characters"
tags: [sliding-window, two-pointers, longest-window]
videoLink: "https://www.youtube.com/playlist?list=PLgUwDviBIf0q7vrFA_HEWcqRqMpCXzYAL"
difficulty: Medium
---

# L3. Longest Substring Without Repeating Characters

> 🎯 **TL;DR:** Find the length of the longest substring with no repeating characters. Two pointer + "last seen index" hash array gets it to O(N).

## 🧩 Problem
String with any of 256 ASCII chars (any case). Return the **length** of the longest substring where all characters are unique.

## 🐢 Brute Force
Double loop (`i` = start, `j` = extend), with a 256-size hash array reset per `i`. Break the moment a repeat is found — every longer substring from that `i` would also repeat.

```text
for i = 0 to n-1:
    hash = array of 256, all 0
    for j = i to n-1:
        if hash[s[j]] == 1:
            break
        hash[s[j]] = 1
        maxLength = max(maxLength, j - i + 1)
```

**Complexity:** Time O(N²) · Space O(256)

## ⚡ Optimal — Two Pointer with Direct Jump
A hash array of size 256 stores the **last index each character was seen at** (initialized to `-1`).

1. If `hash[s[R]] ≥ L` → that character's last sighting is *inside* the current window → jump `L = hash[s[R]] + 1`.
2. Compute `length = R - L + 1`; update `maxLength`.
3. Update `hash[s[R]] = R`; move `R++`.

```text
hash = array of 256, all -1
L = 0, R = 0, maxLength = 0

while R < n:
    if hash[s[R]] >= L:
        L = hash[s[R]] + 1
    maxLength = max(maxLength, R - L + 1)
    hash[s[R]] = R
    R = R + 1
```

> 💡 **Why this isn't the generic Pattern 2 template:** instead of shrinking `L` one step at a time, `L` **jumps directly** to `lastSeenIndex + 1` — because the exact violating position is already known via the hash array, there's no need to incrementally re-check every position in between.

**Complexity:** Time O(N) · Space O(256) ≈ O(1)

## 📊 Complexity Summary

| Approach | Time | Space |
|---|---|---|
| Brute Force | O(N²) | O(256) |
| Optimal (jump) | O(N) | O(256) |

## 💡 Key Takeaways
- "Longest/maximum substring under a constraint" → think two pointer / sliding window immediately.
- Two pointer/sliding window is a **constructive technique**, reshaped per problem — not a fixed algorithm.
- When the violating position is directly knowable (e.g. via a "last seen" map), you can **jump** `L` instead of incrementally shrinking — more optimal than the generic `while`/`if` shrink from L1.

## 🔗 Related
- **Previous:** [L2 — Maximum Points You Can Obtain from Cards](L2-Maximum-Points-You-Can-Obtain-from-Cards.md)
- [↑ Back to Sliding Window Index](00-Sliding-Window-Index.md)
