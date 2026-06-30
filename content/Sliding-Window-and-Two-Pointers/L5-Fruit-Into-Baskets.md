---
title: "L5. Fruit Into Baskets"
tags: [sliding-window, two-pointers, longest-window]
videoLink: "https://www.youtube.com/playlist?list=PLgUwDviBIf0q7vrFA_HEWcqRqMpCXzYAL"
difficulty: Medium
---

# L5. Fruit Into Baskets

> 🎯 **TL;DR:** Two baskets, each holding only one fruit type → reframes to "longest subarray with at most 2 distinct values." Same Pattern 2 template, this time using a frequency map instead of a simple sum/count. Optimal uses the same "stall, don't fully re-shrink" trick from L4.

## 🧩 Problem
A row of fruit trees, each producing one fruit type. You have 2 baskets, each able to hold only **one** fruit type (but unlimited quantity of that type). Pick a *consecutive* run of trees to maximize total fruit collected, given you can only sort into at most 2 types.

## 🔁 Reframing the Problem
This is exactly: **find the longest subarray containing at most 2 distinct values.** ("At most," not "exactly" — if the whole array has only 1 type, you can still use just one basket and leave the other empty.) Generalizes naturally to "at most K distinct values" for K baskets.

## 🐢 Brute Force
Standard `i`/`j` double loop generating subarrays starting at each `i`. Maintain a `set` of distinct values seen; `break` the inner loop the moment the set size exceeds the allowed count.

```text
maxLength = 0
for i = 0 to n-1:
    seen = empty set
    for j = i to n-1:
        seen.add(arr[j])
        if seen.size <= 2:
            maxLength = max(maxLength, j - i + 1)
        else:
            break
```

- Each `set` operation here is effectively **O(1)**, since the set never holds more than 3 elements at a time (it breaks the moment a 3rd distinct value appears).
- **Complexity:** Time O(N²) (standard double-loop subarray generation) · Space O(1) (set bounded to ≤3 elements)

## 🥈 Better — Two Pointer with `while`, using a frequency map
Maintain a hashmap of `{value: frequency}` for the current window. `R` expands, adding to the map. If the map's size exceeds 2 (or K), shrink from `L` with a `while` loop — decrementing frequencies and removing entries that hit zero — until the map is back within the allowed distinct count.

```text
L = 0, R = 0, maxLength = 0
freq = empty map

while R < n:
    freq[arr[R]] = freq[arr[R]] + 1     // or insert with count 1

    while freq.size > 2:                 // shrink: too many distinct types
        freq[arr[L]] = freq[arr[L]] - 1
        if freq[arr[L]] == 0:
            remove arr[L] from freq
        L++

    maxLength = max(maxLength, R - L + 1)
    R++
```

> 💡 Same complexity argument as L1/L4: `R` moves O(N) total; `L` also moves O(N) total across the whole run (never resets) → **O(2N) ≈ O(N)**, despite the nested loop appearance. Map operations are O(1) since the map is bounded to at most 3 entries at any point.

**Complexity:** Time O(2N) ≈ O(N) · Space O(K) (bounded, tiny — e.g. O(3) here)

## 🥇 Optimal — Two Pointer with `if` (the L4 "stall" trick again)
Same insight as L4 (Max Consecutive Ones III): since you only care about the **maximum length**, don't fully re-shrink the window back to validity every time. Once you've hit a window of length `maxLength`, let the window "stall" at that size — just trim **one** element off the left per invalid step — until a removal happens to bring it back into validity, then let it grow again.

```text
L = 0, R = 0, maxLength = 0
freq = empty map

while R < n:
    freq[arr[R]] = freq[arr[R]] + 1

    if freq.size > 2:                  // shrink by exactly ONE step
        freq[arr[L]] = freq[arr[L]] - 1
        if freq[arr[L]] == 0:
            remove arr[L] from freq
        L++

    maxLength = max(maxLength, R - L + 1)
    R++
```

**Complexity:** Time O(N) · Space O(K) ≈ O(1) for fixed small K (e.g. 2 baskets)

## 📊 Complexity Summary

| Approach | Time | Space |
|---|---|---|
| Brute Force | O(N²) | O(1) |
| Better (`while` shrink) | O(2N) ≈ O(N) | O(K) |
| Optimal (`if`, no full re-shrink) | O(N) | O(K) |

## 💡 Key Takeaways
- "At most K distinct types/values in a subarray" is a frequent Pattern 2 variant — track a frequency map, shrink when `map.size > K`.
- The frequency map never grows large in this family of problems (bounded by K+1 entries at most), so map operations don't add a meaningful log factor — treat them as O(1).
- Same optimization as L4 applies directly here: replacing the shrink `while` with an `if` works because the answer (length) never needs to decrease.

## 🔗 Related
- **Previous:** [L4 — Max Consecutive Ones III](L4-Max-Consecutive-Ones-III.md)
- **Next:** [L6 — Longest Substring With At Most K Distinct Characters](L6-Longest-Substring-With-At-Most-K-Distinct-Characters.md) — generalizes this exact pattern
- [↑ Back to Sliding Window Index](00-Sliding-Window-Index.md)
