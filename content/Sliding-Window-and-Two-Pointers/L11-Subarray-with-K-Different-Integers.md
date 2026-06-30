---
title: "L11. Subarray with K Different Integers"
tags: [sliding-window, two-pointers, count-subarrays]
videoLink: "https://www.youtube.com/playlist?list=PLgUwDviBIf0q7vrFA_HEWcqRqMpCXzYAL"
difficulty: Hard
---

# L11. Subarray with K Different Integers

> 🎯 **TL;DR:** Count subarrays with **exactly** K distinct integers. Another non-monotonic "exactly" condition — direct expand/shrink loses track of valid subarrays. Solved with `atMost(K) − atMost(K−1)`, generalizing L9's binary-specific trick to arbitrary integers using a frequency map.

## 🧩 Problem
Given an array of integers and a value `K`, count the number of subarrays containing **exactly K distinct integers**.

## 🐢 Brute Force
Standard `i`/`j` double loop with a frequency map tracking distinct count. If `map.size == K`, count it. If `map.size > K`, `break`.

**Complexity:** Time O(N²) (or O(N² log N) if the map adds a log factor) · Space O(N) at worst (if every element is unique)

## 🤔 Why direct two-pointer expand/shrink fails here
Try expanding `R`, and once `map.size` exceeds `K`, shrinking from `L`. The problem surfaces once you find a valid window (`size == K`) and then expand `R` again to look for the next one — you can end up **skipping over valid smaller subarrays** that existed within the previous valid window. There's no clean monotonic rule for whether the next move should be expanding or shrinking. Same fundamental issue as L9's "exactly equal" non-monotonicity, manifesting differently here.

## 🥇 The Fix — Reframe to "at most K", then subtract
Same trick as L9: solve **"at most K distinct integers"**, which *is* cleanly two-pointer solvable (the distinct count only grows as the window expands, shrinks as you remove from the left — fully monotonic). Then:

```text
count(distinct == K) = atMost(K) − atMost(K − 1)
```

**`atMost(X)` — count subarrays with at most `X` distinct integers:**

```text
function atMost(nums, X):
    L = 0, R = 0, count = 0
    freq = empty map

    while R < n:
        freq[nums[R]]++

        while freq.size > X:           // shrink until valid
            freq[nums[L]]--
            if freq[nums[L]] == 0:
                remove nums[L] from freq
            L++

        count += R - L + 1             // every subarray ending at R, starting from L..R, is valid
        R++

    return count
```

**Why `count += R - L + 1` is safe here:** once `[L, R]` is the largest valid window ending at `R` (distinct count ≤ X), every smaller window `[start, R]` for `start` from `L` to `R` is *also* valid — removing elements from the front of a valid window can only keep the distinct count the same or reduce it, never increase it.

**Final answer:**
```text
answer = atMost(nums, K) - atMost(nums, K - 1)
```

## 📊 Complexity

| Step | Time | Space |
|---|---|---|
| Each `atMost` call | O(2N) ≈ O(N) | O(N) worst case (map of distinct values) |
| Calling `atMost` twice | O(N) overall | O(N) |

## 💡 Key Takeaways
- This is the **general-array version** of L9's trick: L9's binary-specific `atMost` (tracking a sum) generalizes here to tracking distinct-count via a frequency map — same `atMost(K) − atMost(K−1)` structure either way.
- The recurring failure mode for "exactly K" conditions: direct two-pointer expand/shrink risks silently skipping valid subarrays once you've already counted one. Reframing to "at most K" sidesteps this entirely because "at most" is monotonic.
- `count += R - L + 1` is the standard counting move once you have the largest valid window ending at `R` — same pattern seen in L9.

## 🔗 Related
- **Previous:** [L10 — Count Number of Nice Subarrays](L10-Count-Number-of-Nice-Subarrays.md)
- **Builds on:** [L9 — Binary Subarrays With Sum](L9-Binary-Subarrays-With-Sum.md)
- [↑ Back to Sliding Window Index](00-Sliding-Window-Index.md)
