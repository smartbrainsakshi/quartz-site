---
title: "L6. Longest Substring With At Most K Distinct Characters"
tags: [sliding-window, two-pointers, longest-window]
videoLink: "https://www.youtube.com/playlist?list=PLgUwDviBIf0q7vrFA_HEWcqRqMpCXzYAL"
difficulty: Medium
---

# L6. Longest Substring With At Most K Distinct Characters

> 🎯 **TL;DR:** The generalized version of L5 — instead of "at most 2 distinct types" (fruit baskets), it's "at most K distinct characters." Identical template: frequency map + Brute → Better (`while`) → Optimal (`if`).

## 🧩 Problem
Given a string and an integer `K`, find the length of the **longest substring with at most K distinct characters**.

## 🐢 Brute Force
Standard `i`/`j` double loop. For each starting index `i`, clear a frequency map and extend `j` to the right, inserting characters into the map. The moment `map.size` exceeds `K`, `break`.

```text
maxLength = 0
for i = 0 to n-1:
    map = empty
    for j = i to n-1:
        map[s[j]]++
        if map.size <= K:
            maxLength = max(maxLength, j - i + 1)
        else:
            break
```

**Complexity:** Time O(N²) · Space O(min(N, charset size)) — up to 256 for full ASCII, fewer if restricted (e.g. lowercase-only → 26).

## 🥈 Better — Two Pointer with `while`
`R` expands, inserting into a frequency map. Whenever `map.size > K`, shrink from `L` with a `while` loop — decrementing frequencies, removing entries that hit zero — until the map size is back within `K`.

```text
L = 0, R = 0, maxLength = 0
freq = empty map

while R < n:
    freq[s[R]]++

    while freq.size > K:
        freq[s[L]]--
        if freq[s[L]] == 0:
            remove s[L] from freq
        L++

    maxLength = max(maxLength, R - L + 1)
    R++
```

> 💡 Same amortized argument as always: `R` moves O(N) total, `L` moves O(N) total across the whole run → **O(2N) ≈ O(N)**, not O(N²).

**Complexity:** Time O(2N) ≈ O(N) · Space O(K) (map never holds more than K+1 entries at a time)

## 🥇 Optimal — Two Pointer with `if` (no inner loop)
Same "stall" trick from L4/L5: don't fully re-shrink back to validity each time — once you've achieved a window of length `maxLength`, let the window size stall there. Trim **one** character off the left per invalid step.

```text
L = 0, R = 0, maxLength = 0
freq = empty map

while R < n:
    freq[s[R]]++

    if freq.size > K:           // shrink by exactly ONE step
        freq[s[L]]--
        if freq[s[L]] == 0:
            remove s[L] from freq
        L++

    maxLength = max(maxLength, R - L + 1)
    R++
```

**Complexity:** Time O(N) · Space O(K)

## 📊 Complexity Summary

| Approach | Time | Space |
|---|---|---|
| Brute Force | O(N²) | O(charset size) |
| Better (`while` shrink) | O(2N) ≈ O(N) | O(K) |
| Optimal (`if`, no inner loop) | O(N) | O(K) |

## 💡 Key Takeaways
- This is a direct generalization of L5 (Fruit Into Baskets): "at most 2 distinct" → "at most K distinct," same frequency-map mechanics.
- The `if`-instead-of-`while` optimization recurs again, same reasoning as L4 and L5.
- Pattern recognition payoff: once you've internalized this template from L4/L5/L6, "at most K [something]" problems become close to mechanical.

## 🔗 Related
- **Previous:** [L5 — Fruit Into Baskets](L5-Fruit-Into-Baskets.md)
- [↑ Back to Sliding Window Index](index.md)
