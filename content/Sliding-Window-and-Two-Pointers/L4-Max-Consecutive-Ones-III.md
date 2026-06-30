---
title: "L4. Max Consecutive Ones III"
tags: [sliding-window, two-pointers, longest-window]
videoLink: "https://www.youtube.com/playlist?list=PLgUwDviBIf0q7vrFA_HEWcqRqMpCXzYAL"
difficulty: Medium
---

# L4. Max Consecutive Ones III

> 🎯 **TL;DR:** Find the max length of consecutive 1s after flipping at most K zeros. Same as: longest subarray with at most K zeros. Classic Pattern 2 — Brute → Better (`while`) → Optimal (`if`, no inner loop at all).

## 🧩 Problem
Binary array (0s and 1s) + integer `K`. You may flip at most `K` zeros to ones. Return the max length of consecutive ones achievable. (Length only, not the segment.)

## 🔁 Reframing the Problem
"Max consecutive ones after flipping ≤ K zeros" is equivalent to: **find the longest subarray containing at most K zeros.** Same problem, cleaner framing for the sliding-window template.

## 🐢 Brute Force
Standard `i`/`j` double loop generating every subarray starting at `i`. Track a running zero-count as `j` extends; the moment the zero-count exceeds `K`, `break` (every longer subarray from this `i` will also exceed K).

```text
maxLength = 0
for i = 0 to n-1:
    zeros = 0
    for j = i to n-1:
        if nums[j] == 0:
            zeros++
        if zeros <= K:
            maxLength = max(maxLength, j - i + 1)
        else:
            break
```

**Complexity:** Time O(N²) (roughly) · Space O(1)

## 🥈 Better — Two Pointer with `while`
`R` always expands, tracking `zeros`. Once `zeros > K`, shrink from `L` with a `while` loop until `zeros` is back within K.

```text
L = 0, R = 0, zeros = 0, maxLength = 0
while R < n:
    if nums[R] == 0:
        zeros++
    while zeros > K:
        if nums[L] == 0:
            zeros--
        L++
    maxLength = max(maxLength, R - L + 1)
    R++
```

> 💡 **Why O(2N):** `R` drives the outer loop, O(N) total. The inner `while` doesn't run O(N) on *every* iteration of `R` — across the whole run, `L` moves forward at most N times total. So total work is O(N) + O(N) = **O(2N) ≈ O(N)**, not O(N²), despite the nested-loop look.

**Complexity:** Time O(2N) ≈ O(N) · Space O(1)

## 🥇 Optimal — Two Pointer with `if` (no inner loop at all)
Same insight as L1's Pattern 2 optimal: once you've achieved a window of length `maxLength`, don't bother shrinking back below it — a smaller window can never beat the best you already have. Replace the inner `while` with a single-step `if`.

```text
L = 0, R = 0, zeros = 0, maxLength = 0
while R < n:
    if nums[R] == 0:
        zeros++
    if zeros > K:
        if nums[L] == 0:
            zeros--
        L++
    maxLength = max(maxLength, R - L + 1)
    R++
```

**How this differs in behavior:** instead of always shrinking back down until `zeros == K` again, `L` only moves **one step** per invalid `R`. The window size effectively "stalls" at the best length found so far until a new zero gets trimmed off the left, at which point it can grow again. This means `maxLength` never decreases — it can only stay the same or grow — so there's no need to fully re-validate after each shrink.

**Complexity:** Time O(N) · Space O(1) — only `R` drives the loop; no inner loop, so no extra `N` factor.

## 📊 Complexity Summary

| Approach | Time | Space |
|---|---|---|
| Brute Force | O(N²) | O(1) |
| Better (`while` shrink) | O(2N) ≈ O(N) | O(1) |
| Optimal (`if`, no inner loop) | O(N) | O(1) |

## 💡 Key Takeaways
- "Max consecutive elements after K flips" reframes cleanly into "longest subarray with at most K [condition]" — recognizing this reframing is often the hardest part.
- The `if`-instead-of-`while` optimization works here for the same reason as L3: since you only care about the **maximum length**, you never need to shrink below your current best — a window can stall at its best size while waiting to become valid again, rather than fully re-shrinking each time.
- Confirms the L1 Pattern 2 complexity argument in practice: even with an inner `while`, total combined pointer movement is bounded at O(2N), not O(N²).

## 🔗 Related
- **Previous:** [L3 — Longest Substring Without Repeating Characters](L3-Longest-Substring-Without-Repeating-Characters.md)
- [↑ Back to Sliding Window Index](00-Sliding-Window-Index.md)
