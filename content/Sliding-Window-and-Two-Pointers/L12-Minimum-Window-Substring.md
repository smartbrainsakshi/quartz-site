---
title: "L12. Minimum Window Substring"
tags: [sliding-window, two-pointers, shortest-window]
videoLink: "https://www.youtube.com/playlist?list=PLgUwDviBIf0q7vrFA_HEWcqRqMpCXzYAL"
difficulty: Hard
---

# L12. Minimum Window Substring

> 🎯 **TL;DR:** Find the smallest substring of `s` containing all characters of `t` (with at least their required multiplicity). Pattern 4: expand until valid, then shrink while still valid — the opposite shrink condition from Pattern 2.

## 🧩 Problem
Given strings `s` and `t`, return the **minimum-length substring of `s`** that contains every character of `t`, including duplicates — e.g., if `t` has two `B`s, the result must contain at least two `B`s too. Return the actual substring (not just its length).

## 🐢 Brute Force
Standard `i`/`j` double loop generating all substrings, with a 256-size frequency array pre-loaded with `t`'s character counts. For each substring, increment a `count` whenever a needed character is encountered. The substring is valid once `count == t.length`. Track the minimum valid length and its starting index.

**Complexity:** Time O(N²) (the inner loop effectively rescans for each starting index) · Space O(256)

## 🥇 Optimal — Two Pointer, Expand Until Valid → Shrink While Valid
Pre-load a 256-size hash array with `t`'s character frequencies. Maintain a running `count` of how many of `t`'s required characters have been matched so far (a character only counts toward `count` if its hash value was still **positive** at the moment it's encountered — meaning it was still "needed").

**Mechanics:**
- **Inserting** a character (moving `R` forward): decrement its hash value. If the value *was* positive before decrementing, increment `count`.
- **Removing** a character (shrinking from `L`): increment its hash value back up. If the value becomes **positive again** after incrementing, it means you've now under-supplied that character — decrement `count` accordingly.
- Whenever `count == t.length`, the current window `[L, R]` is valid. Record it if it's the smallest seen so far, then try to **shrink further from the left** (opposite of Pattern 2 — here you shrink *while* still valid) until it becomes invalid.

```text
hash = array of 256, all 0
for i = 0 to m-1:
    hash[t[i]]--                      // pre-load t's required counts (as negative)

L = 0, R = 0, count = 0
minLength = infinity, startIndex = -1

while R < n:
    if hash[s[R]] > 0:                // this character was still needed
        count++
    hash[s[R]]--
    R++

    while count == m:                 // valid window — try to shrink it smaller
        if (R - L) < minLength:
            minLength = R - L
            startIndex = L

        hash[s[L]]++
        if hash[s[L]] > 0:             // removing this pushed it back into "needed" territory
            count--
        L++

return startIndex == -1 ? "" : s.substring(startIndex, startIndex + minLength)
```

> 💡 **Why this is Pattern 4, not Pattern 2:** in Pattern 2, you shrink *only when invalid* (fixing a broken window). Here, you shrink *while still valid* — actively trying to make a good window even smaller, stopping only once it breaks. The "expand until valid, then shrink while valid" two-phase loop structure is the defining shape of Pattern 4.

## 📊 Complexity

| Approach | Time | Space |
|---|---|---|
| Brute Force | O(N²) | O(256) |
| Optimal (expand → shrink while valid) | O(N) + O(M) (loading `t`) ≈ O(N) | O(256) |

## 💡 Key Takeaways
- Pattern 4's loop shape mirrors Pattern 2 but inverts the shrink condition: **expand until valid → shrink while valid** (vs. Pattern 2's "expand always → shrink only if invalid").
- The "positive value = still needed" trick (decrement on insert, increment on remove, adjust `count` only when crossing the zero boundary) avoids ever needing to rescan the whole hash array.
- This problem needs the **actual substring** as output, not just a length — so (like in L1/L3) you must track the precise `startIndex` and can't take any shortcut that loses position information.

## 🎉 Section Complete
All 12 lectures are now noted, covering all 4 core patterns:
- **Pattern 1** (Constant Window): L2
- **Pattern 2** (Longest with Condition): L3, L4, L5, L6, L8
- **Pattern 3** (Count Subarrays): L7, L9, L10, L11
- **Pattern 4** (Shortest Window): L12

The recurring threads across the whole series: the `while → if` "stall" optimization (L1, L4, L5, L6, L8), the `atMost(K) − atMost(K−1)` trick (L9, L10, L11), and "last seen index" tracking (L3, L7).

## 🔗 Related
- **Previous:** [L11 — Subarray with K Different Integers](L11-Subarray-with-K-Different-Integers.md)
- [↑ Back to Sliding Window Index](00-Sliding-Window-Index.md)
