---
title: Sliding Window & Two Pointers
description: take U forward — Strivers A2Z DSA Course, Step 10. All 4 core patterns and 12 lectures, ported over from a previous Notion notes project.
---

# 07 · Sliding Window & Two Pointers

Notes built from [take U forward's Sliding Window & Two Pointers playlist](https://www.youtube.com/playlist?list=PLgUwDviBIf0q7vrFA_HEWcqRqMpCXzYAL) (Strivers A2Z DSA Course, Step 10 — 12 videos). Originally written up in a Notion workspace; ported here so everything lives in one place.

This isn't part of the Graph Series — it's a separate algorithmic topic — but it follows the same note structure and lives in the same repo for convenience.

## Notes in this section — 12/12

- [x] [L1. Introduction to Sliding Window & 2 Pointers](L1-Introduction-to-Sliding-Window-and-2-Pointers.md) — *Foundations, all 4 patterns*
- [x] [L2. Maximum Points You Can Obtain from Cards](L2-Maximum-Points-You-Can-Obtain-from-Cards.md) — *Pattern 1, Constant Window*
- [x] [L3. Longest Substring Without Repeating Characters](L3-Longest-Substring-Without-Repeating-Characters.md) — *Pattern 2, Longest with Condition*
- [x] [L4. Max Consecutive Ones III](L4-Max-Consecutive-Ones-III.md) — *Pattern 2, Longest with Condition*
- [x] [L5. Fruit Into Baskets](L5-Fruit-Into-Baskets.md) — *Pattern 2, Longest with Condition*
- [x] [L6. Longest Substring With At Most K Distinct Characters](L6-Longest-Substring-With-At-Most-K-Distinct-Characters.md) — *Pattern 2, Longest with Condition*
- [x] [L7. Number of Substrings Containing All Three Characters](L7-Number-of-Substrings-Containing-All-Three-Characters.md) — *Pattern 3, Count Subarrays*
- [x] [L8. Longest Repeating Character Replacement](L8-Longest-Repeating-Character-Replacement.md) — *Pattern 2, Longest with Condition*
- [x] [L9. Binary Subarrays With Sum](L9-Binary-Subarrays-With-Sum.md) — *Pattern 3, Count Subarrays*
- [x] [L10. Count Number of Nice Subarrays](L10-Count-Number-of-Nice-Subarrays.md) — *Pattern 3, Count Subarrays*
- [x] [L11. Subarray with K Different Integers](L11-Subarray-with-K-Different-Integers.md) — *Pattern 3, Count Subarrays*
- [x] [L12. Minimum Window Substring](L12-Minimum-Window-Substring.md) — *Pattern 4, Shortest Window*

✅ **Section complete (12/12)**

## The 4 patterns, at a glance

| Pattern | Goal | Core Mechanic |
|---|---|---|
| 1. Constant window | Fixed-size window stats | Slide by 1 |
| 2. Longest w/ condition | Maximize length | Expand always; shrink only when invalid |
| 3. Count subarrays | Count windows | Two Pattern-2 "at most" calls, subtract |
| 4. Shortest w/ condition | Minimize length | Expand until valid, shrink while valid |

**Recurring threads across the whole series:**
- The `while → if` "stall" optimization (L1, L4, L5, L6, L8)
- The `atMost(K) − atMost(K−1)` trick (L9, L10, L11)
- "Last seen index" tracking (L3, L7)

[← Back to All Topics](../index.md)
