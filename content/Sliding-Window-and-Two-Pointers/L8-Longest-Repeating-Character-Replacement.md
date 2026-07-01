---
title: "L8. Longest Repeating Character Replacement"
tags: [sliding-window, two-pointers, longest-window]
videoLink: "https://www.youtube.com/playlist?list=PLgUwDviBIf0q7vrFA_HEWcqRqMpCXzYAL"
difficulty: Medium
---

# L8. Longest Repeating Character Replacement

> 🎯 **TL;DR:** Longest substring where, after converting at most K characters, everything becomes equal. Reframes to "longest window where `length − maxFrequency ≤ K`." Closes out Pattern 2 with two stacked optimizations: drop the frequency-rescan, then drop the `while` shrink.

## 🧩 Problem
Given a string of uppercase letters and integer `K`: you may change **at most K** characters in a chosen substring to any other character. Find the length of the **longest substring** that can be made all-equal this way.

## 🔁 Reframing the Problem
For any window, the minimum number of conversions needed to make all characters equal is:

```text
conversions needed = windowLength − maxFrequency
```

(Leave the most frequent character untouched; convert everything else.) So the problem becomes: **find the longest window where `length − maxFrequency ≤ K`.**

## 🐢 Brute Force
Standard `i`/`j` double loop. Maintain a 26-size frequency array, tracking `maxFrequency` as you extend `j`. Check `length - maxFrequency <= K`; if it holds, update `maxLength`; otherwise `break`.

```text
for i = 0 to n-1:
    freq = array of 26, all 0
    maxFreq = 0
    for j = i to n-1:
        freq[s[j]]++
        maxFreq = max(maxFreq, freq[s[j]])
        if (j - i + 1) - maxFreq <= K:
            maxLength = max(maxLength, j - i + 1)
        else:
            break
```

**Complexity:** Time O(N²) · Space O(26)

## 🥈 Better — Two Pointer with `while` + full rescan
`R` expands, updating frequency and `maxFreq`. When `(R-L+1) - maxFreq > K` (invalid), shrink from `L` — but when you remove a character, `maxFreq` might need to **decrease**, so you rescan the entire 26-slot frequency array each time to recompute it.

```text
L = 0, R = 0, maxFreq = 0, maxLength = 0
freq = array of 26, all 0

while R < n:
    freq[s[R]]++
    maxFreq = max(maxFreq, freq[s[R]])

    while (R - L + 1) - maxFreq > K:
        freq[s[L]]--
        maxFreq = 0
        for c = 0 to 25:                // rescan to recompute maxFreq
            maxFreq = max(maxFreq, freq[c])
        L++

    maxLength = max(maxLength, R - L + 1)
    R++
```

**Complexity:** Time O(N) + O(N × 26) ≈ O(26N) (the rescan is the bottleneck) · Space O(26)

## 🚀 Optimization 1 — Stop Rescanning: `maxFreq` Never Needs to Decrease
Key insight: once you've found a valid window of length `L_best`, the only way to beat it is to find a valid window of length `L_best + 1`. For that *new* length to be valid, you need an **equal or higher** `maxFreq` than before — a lower `maxFreq` can never help. So there's no benefit in ever decreasing `maxFreq` when shrinking; just leave it as the highest value ever seen, even if it's momentarily "stale."

```text
L = 0, R = 0, maxFreq = 0, maxLength = 0
freq = array of 26, all 0

while R < n:
    freq[s[R]]++
    maxFreq = max(maxFreq, freq[s[R]])      // only ever increases now

    while (R - L + 1) - maxFreq > K:
        freq[s[L]]--                        // no rescan — maxFreq stays as-is
        L++

    maxLength = max(maxLength, R - L + 1)
    R++
```

This removes the `× 26` rescan factor → **O(2N)** (still has the `while`).

## 🥇 Optimization 2 — Replace `while` with `if` (final, most optimal)
Same "stall" trick used in L4/L5/L6: replace the shrink `while` with a single-step `if`.

```text
L = 0, R = 0, maxFreq = 0, maxLength = 0
freq = array of 26, all 0

while R < n:
    freq[s[R]]++
    maxFreq = max(maxFreq, freq[s[R]])

    if (R - L + 1) - maxFreq > K:    // shrink by exactly ONE step
        freq[s[L]]--
        L++

    maxLength = max(maxLength, R - L + 1)
    R++
```

**Complexity:** Time O(N) · Space O(26)

## 📊 Complexity Summary

| Approach | Time | Space |
|---|---|---|
| Brute Force | O(N²) | O(26) |
| Better (`while` + rescan) | O(26N) | O(26) |
| Opt. 1 (`while`, no rescan) | O(2N) ≈ O(N) | O(26) |
| Opt. 2 / Final (`if`, no rescan) | O(N) | O(26) |

## 💡 Key Takeaways
- This problem stacks **two separate optimizations**: (1) a tracked maximum doesn't need to be decreased on shrink if decreasing it can never improve the answer, and (2) the familiar `while → if` stall trick from L4/L5/L6.
- General principle: **if a tracked value monotonically helps the answer, don't waste time recomputing it downward** — let it become "stale-but-safe" instead of re-scanning.
- This closes out **Pattern 2** for this playlist — L3 through L8 all reduce to the same expand/shrink skeleton, with only the specific *condition* and *what's tracked* changing each time.

## 🔗 Related
- **Previous:** [L7 — Number of Substrings Containing All Three Characters](L7-Number-of-Substrings-Containing-All-Three-Characters.md)
- [↑ Back to Sliding Window Index](index.md)
