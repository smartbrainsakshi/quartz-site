---
title: "L10. Count Number of Nice Subarrays"
tags: [sliding-window, two-pointers, count-subarrays]
videoLink: "https://www.youtube.com/playlist?list=PLgUwDviBIf0q7vrFA_HEWcqRqMpCXzYAL"
difficulty: Medium
---

# L10. Count Number of Nice Subarrays

> 🎯 **TL;DR:** A "nice subarray" = exactly K odd numbers. This is L9 (Binary Subarrays With Sum) in disguise — map odd→1, even→0, and reuse the exact same `atMost(K) − atMost(K−1)` solution unchanged.

## 🧩 Problem
Given an integer array and a value `K`, count the number of subarrays containing **exactly K odd numbers**.

## 🔁 Reframing the Problem — Recognizing It's L9 Again
The whole exercise here is **pattern recognition**: treat every odd number as `1` and every even number as `0`. Counting subarrays with exactly `K` odd numbers is then *identical* to L9's problem — "count binary-array subarrays with sum equal to `goal`" — with `goal = K`.

## ⚡ Solution — Reuse L9's `atMost(K) − atMost(K−1)` directly
No new algorithm needed. Reuse L9's `atMost(X)` two-pointer helper, with one tiny implementation tweak: instead of adding/removing the raw array value when expanding/shrinking the window, add/remove `value % 2` (which evaluates to `1` for odd numbers and `0` for even numbers).

```text
function atMost(nums, X):
    if X < 0:
        return 0

    L = 0, R = 0, oddCount = 0, count = 0
    while R < nums.length:
        oddCount += nums[R] % 2       // 1 if odd, 0 if even

        while oddCount > X:
            oddCount -= nums[L] % 2  // 1 if odd, 0 if even
            L++

        count += R - L + 1
        R++

    return count

answer = atMost(nums, K) - atMost(nums, K - 1)
```

**Complexity:** Time O(N) (two `atMost` calls, each O(2N)) · Space O(1) — identical to L9.

## 💡 Key Takeaways
- The most important skill this lecture teaches isn't a new algorithm — it's **recognizing a disguised version of a problem you've already solved.** Interviewers (and problem setters) frequently rephrase known problems with different wording.
- A useful habit: when a problem mentions counting subarrays/substrings with an exact count of some *category* of element (odd/even, vowels/consonants, etc.), try mapping that category to a binary 0/1 array — it may reduce directly to a binary-array sum problem like L9.
- This is a good test of whether the **general template** (`atMost(K) − atMost(K−1)`) was actually understood in L9, versus just memorized for that specific problem's surface details.

## 🔗 Related
- **Previous:** [L9 — Binary Subarrays With Sum](L9-Binary-Subarrays-With-Sum.md)
- [↑ Back to Sliding Window Index](00-Sliding-Window-Index.md)
