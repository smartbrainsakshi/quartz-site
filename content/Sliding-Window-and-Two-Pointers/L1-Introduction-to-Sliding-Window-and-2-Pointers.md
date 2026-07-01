---
title: "L1. Introduction to Sliding Window & 2 Pointers"
tags: [sliding-window, two-pointers, foundations]
videoLink: "https://www.youtube.com/playlist?list=PLgUwDviBIf0q7vrFA_HEWcqRqMpCXzYAL"
---

# L1. Introduction to Sliding Window & 2 Pointers

> 🎯 **TL;DR:** Sliding window / two pointer problems fall into 4 recurring patterns. This lecture introduces all 4 and their core templates — everything else in this section is a variation on one of them.

## Pattern 1 — Constant Window

Window size `k` is fixed.

**Example:** max sum of any 4 *consecutive* elements in an array.

**Approach:**
1. Compute the sum of the first window (`L=0` to `R=k-1`).
2. Slide forward: before moving `L`, subtract `arr[L]`; then `L++`, `R++`, add `arr[R]`.
3. Track `maxSum` after each shift.

```text
sum = sum of arr[0..k-1]
maxSum = sum
for R = k to n-1:
    sum = sum - arr[L]
    L++
    sum = sum + arr[R]
    maxSum = max(maxSum, sum)
```

**Complexity:** Time O(N) · Space O(1)

📍 Rare in interviews — only L2 in this playlist uses it directly.

## Pattern 2 — Longest Subarray/Substring With a Condition

**The most common pattern** on this topic. Solved in 3 stages: **Brute → Better → Optimal**.

**Example:** longest subarray with sum ≤ K.

### 🐢 Brute Force
Generate all subarrays (`i`,`j` double loop), track sum, check condition, early-`break` once sum exceeds K.

**Complexity:** O(N²) time · O(1) space

### 🥈 Better — Two Pointer with `while`
`R` always expands; `L` shrinks via a `while` loop whenever the window becomes invalid.

```text
L = 0, R = 0, sum = 0, maxLen = 0
while R < n:
    sum = sum + arr[R]
    while sum > K:
        sum = sum - arr[L]
        L++
    maxLen = max(maxLen, R - L + 1)
    R++
```

> 💡 **Why O(2N), not O(N²):** `R` moves forward at most N times total. `L` *also* moves forward at most N times total across the whole run (it never resets). Total pointer moves ≤ 2N → **O(2N) ≈ O(N)**. Classic amortized-analysis trap — nested loops ≠ automatically quadratic.

**Complexity:** Time O(2N) ≈ O(N) · Space O(1)

### 🥇 Optimal — Two Pointer with `if`
Once a valid window of length `maxLen` is found, never shrink below it — replace the `while` with a single-step `if`.

```text
L = 0, R = 0, sum = 0, maxLen = 0
while R < n:
    sum = sum + arr[R]
    if sum > K:
        sum = sum - arr[L]
        L++
    maxLen = max(maxLen, R - L + 1)
    R++
```

**Complexity:** Time O(N) · Space O(1)

> ⚠️ **Caveat:** the `if` shortcut only works when you need the **length**. If asked to **return the actual subarray**, you must use the `while` (Better) version instead.

## Pattern 3 — Number of Subarrays Satisfying a Condition

**Example:** count subarrays with sum **exactly** K.

Exact-equality conditions are **non-monotonic** — you can't reliably tell whether to expand or shrink. So Pattern 2's template doesn't directly apply.

**The trick:**

```text
count(sum == K) = count(sum ≤ K) − count(sum ≤ K−1)
```

Both right-hand terms are solved with the Pattern 2 "at most" technique, then subtracted. This `atMost(K) − atMost(K−1)` trick reappears across the playlist.

## Pattern 4 — Shortest/Minimum Window Satisfying a Condition

Rare pattern; opposite goal to Pattern 2.

1. Expand `R` until the window becomes valid.
2. Once valid, shrink from `L` **while it remains valid**, updating `minLen` at each valid step.
3. Repeat.

## 📊 Recap Table

| Pattern | Goal | Core Mechanic | Complexity |
|---|---|---|---|
| 1. Constant window | Fixed-size window stats | Slide by 1 | O(N) |
| 2. Longest w/ condition | Maximize length | Expand always; shrink only when invalid | O(N²) → O(2N) → O(N) |
| 3. Count subarrays | Count windows | Two Pattern-2 "at most" calls, subtract | O(N) |
| 4. Shortest w/ condition | Minimize length | Expand until valid, shrink while valid | O(N) |

## 💡 Key Takeaways
- Identify the pattern **before** coding.
- Pattern 2: go Brute → Better (`while`) → Optimal (`if`) — but the `if` shortcut breaks if you must reconstruct the actual subarray.
- "Nested loop ⇒ O(N²)" is a trap — total pointer movement is bounded by 2N, not N×N.
- Pattern 3's `atMost(K) − atMost(K−1)` trick is reused often — memorize it.

## 🔗 Related
- [↑ Back to Sliding Window Index](index.md)
