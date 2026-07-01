---
title: "L2. Maximum Points You Can Obtain from Cards"
tags: [sliding-window, two-pointers, constant-window]
videoLink: "https://www.youtube.com/playlist?list=PLgUwDviBIf0q7vrFA_HEWcqRqMpCXzYAL"
difficulty: Medium
---

# L2. Maximum Points You Can Obtain from Cards

> 🎯 **TL;DR:** Pick exactly K cards from the front and/or back (consecutively) to maximize points. Treat it as a constant window split between two ends.

## 🧩 Problem
Array of `n` cards' point values + integer `K`. Pick exactly `K` cards, only from the front and/or back, consecutively. Maximize total points.

## 💡 Key Insight
Picking `x` from the front + `(K−x)` from the back, for every `x` from `K` down to `0`, covers every valid combination.

> `answer = leftSum + rightSum`, where `leftSum` shrinks one element at a time and `rightSum` grows one element at a time.

## ⚡ Approach
1. `leftSum` = sum of first `K` elements. `rightSum = 0`. `maxSum = leftSum`.
2. Loop `i` from `K-1` down to `0`:
   - Remove `nums[i]` from `leftSum`.
   - Add `nums[rightIndex]` to `rightSum` (`rightIndex` starts at `n-1`, decrements each step).
   - Update `maxSum = max(maxSum, leftSum + rightSum)`.
3. Return `maxSum`.

```text
function maxScore(nums, K):
    leftSum = sum of nums[0..K-1]
    rightSum = 0
    maxSum = leftSum
    rightIndex = n - 1

    for i = K-1 down to 0:
        leftSum = leftSum - nums[i]
        rightSum = rightSum + nums[rightIndex]
        rightIndex = rightIndex - 1
        maxSum = max(maxSum, leftSum + rightSum)

    return maxSum
```

## 📊 Complexity

| Step | Time |
|---|---|
| Initial `leftSum` | O(K) |
| Shifting loop | O(K) |
| **Total** | **O(2K) ≈ O(K)** time · **O(1)** space |

> ⚠️ Prefer this 2-variable approach over prefix-sum-heavy solutions you'll find elsewhere — it's cleaner and equally optimal.

## 🏷️ Why Pattern 1, not Pattern 2
Window size stays fixed at exactly `K` throughout — only the *split point* between front/back picks slides. Same constant-window idea as L1, just applied at both ends simultaneously.

## 🔗 Related
- **Previous:** [L1 — Introduction](L1-Introduction-to-Sliding-Window-and-2-Pointers.md)
- [↑ Back to Sliding Window Index](index.md)
