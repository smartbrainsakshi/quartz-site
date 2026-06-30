---
title: "L9. Binary Subarrays With Sum"
tags: [sliding-window, two-pointers, count-subarrays]
videoLink: "https://www.youtube.com/playlist?list=PLgUwDviBIf0q7vrFA_HEWcqRqMpCXzYAL"
difficulty: Medium
---

# L9. Binary Subarrays With Sum

> 🎯 **TL;DR:** Count subarrays of a **binary** array whose sum equals `goal`. The first Pattern 3 problem to directly use the `atMost(K) − atMost(K−1)` trick from L1 — and it shows *why* you need the trick (you can't tell whether to expand or shrink when sum must be exactly equal).

## 🧩 Problem
A binary array (only 0s and 1s) + an integer `goal`. Count the number of subarrays whose sum equals exactly `goal`.

## 🔗 Connection to a Prior Problem
This is the same problem as "count subarrays with sum = K" for general (positive/negative) arrays, normally solved with prefix-sum + hashmap (O(N) time, O(N) space). That solution still works here. But since this array is specifically **binary**, the extra O(N) space for the hashmap can be eliminated entirely using two pointers.

## 🤔 Why a direct two-pointer "expand/shrink" doesn't work for exact sum
Try expanding `R`, and shrinking `L` if the sum exceeds `goal`. The problem: removing a `0` from the window **doesn't change the sum at all**. So when the window's sum equals `goal`, you genuinely can't tell whether moving `L` forward or moving `R` forward is the "right" next move. There's no clear monotonic signal. Same non-monotonicity problem flagged in L1 for "exactly K" conditions.

## 🥇 The Fix — `atMost(goal) − atMost(goal−1)`
Solve **"sum is at most X"** instead — solvable cleanly with two pointers, since the binary array's sum only ever increases or stays flat as the window grows (never decreases).

```text
count(sum == goal) = atMost(goal) − atMost(goal − 1)
```

**`atMost(X)` — count subarrays with sum ≤ X:**

```text
function atMost(nums, X):
    if X < 0:
        return 0          // sum can never be negative in a binary array

    L = 0, R = 0, sum = 0, count = 0
    while R < nums.length:
        sum += nums[R]

        while sum > X:
            sum -= nums[L]
            L++

        count += R - L + 1    // every subarray ending at R, starting anywhere from L to R, is valid
        R++

    return count
```

**Why `count += R - L + 1` works:** once the window `[L, R]` is the *largest valid* window ending at `R`, every subarray `[start, R]` for `start` from `L` to `R` is also valid (a subset of a valid-sum window of non-negative numbers has sum ≤ the whole window's sum). That's exactly `R - L + 1` subarrays, all newly counted at this step.

**Final answer:**
```text
answer = atMost(nums, goal) - atMost(nums, goal - 1)
```

**Edge case:** if `goal == 0`, then `goal - 1 == -1`, and `atMost` must return `0` immediately for any negative `X`.

## 📊 Complexity

| Step | Time | Space |
|---|---|---|
| Each `atMost` call | O(2N) ≈ O(N) | O(1) |
| Calling `atMost` twice | O(2 × 2N) ≈ O(N) | O(1) |

Compared to the general hashmap solution (O(N) time, O(N) space), this trades a small constant-factor time increase for **O(1) space** — only possible because the array is binary.

## 💡 Key Takeaways
- This is the canonical example of *why* `atMost(K) − atMost(K−1)` is needed: direct two-pointer expand/shrink breaks down for "exactly equal" conditions because removing certain elements (like a `0` here) doesn't change the tracked value.
- The `atMost(X)` helper's `count += R - L + 1` line is the standard way to count once you have a valid window ending at `R`.
- Watch for problems where general techniques (hashmap) work, but a **special property of the input** (here: binary-only values) unlocks a more space-efficient two-pointer approach.

## 🔗 Related
- **Previous:** [L8 — Longest Repeating Character Replacement](L8-Longest-Repeating-Character-Replacement.md)
- **Next:** [L10 — Count Number of Nice Subarrays](L10-Count-Number-of-Nice-Subarrays.md) — this exact problem in disguise
- [↑ Back to Sliding Window Index](00-Sliding-Window-Index.md)
