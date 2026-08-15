---
title: "Single Number II"
tags: [bit-manipulation, xor, and, or, modular-counting]
difficulty: Medium
---

# Single Number II

## 🎯 What problem are we solving?

> [!question]
> Given an array where every number appears **exactly three times** except for one number, which appears exactly once, find that single number.

## 💡 Intuition

> [!tip]
> Plain XOR-everything (the [Single Number I](12-Single-Number-I.md) trick) relies on pairs canceling to `0` — but three identical copies XOR down to the number itself, not `0` (`x^x^x = x`), so that trick doesn't directly work here.
>
> Instead, think **per bit position**, across all 32 bits of an int. For any given bit position, sum up how many numbers in the array have that bit set. If every number *except* the answer appears three times, then every bit contributed by a duplicated number gets counted in multiples of `3` — only the answer's own bit contributes a "+1" that breaks the multiple-of-3 pattern. So: **for each bit position, if the count of set bits across the whole array is *not* a multiple of 3, that bit must be set in the answer.**

## 🖼️ Visualizing it

```
nums = [2, 2, 3, 2]   (2 appears three times, 3 appears once -- answer is 3)

Bit 0: 2=010(0), 2=010(0), 3=011(1), 2=010(0)  -> 1 one  -> 1 % 3 = 1 -> answer's bit 0 is SET
Bit 1: 2=010(1), 2=010(1), 3=011(1), 2=010(1)  -> 4 ones -> 4 % 3 = 1 -> answer's bit 1 is SET
Bit 2 and above: all zero -> 0 % 3 = 0 -> unset

Answer bits: bit0=1, bit1=1, rest=0 -> binary 011 -> 3
```

**Result: 3.**

## 🛠️ Algorithm / Approach

> [!abstract]
> 1. For each bit position `i` from `0` to `31`:
>    - Count how many numbers in the array have bit `i` set (using `num & (1 << i)`).
>    - If that count `% 3 != 0`, set bit `i` in the answer (via `answer |= (1 << i)`).
> 2. Return the answer.

## 👨‍💻 Code

### Java

```java
public class SingleNumberII {

    public int singleNumber(int[] nums) {
        int answer = 0;

        for (int i = 0; i < 32; i++) {
            int count = 0;
            for (int num : nums) {
                if ((num & (1 << i)) != 0) {
                    count++;
                }
            }
            if (count % 3 != 0) {
                answer |= (1 << i);
            }
        }
        return answer;
    }
}
```

### Python

```python
def single_number(nums: list[int]) -> int:
    answer = 0

    for i in range(32):
        count = sum(1 for num in nums if num & (1 << i))
        if count % 3 != 0:
            answer |= (1 << i)

    return answer
```

## ⏱️ Complexity Analysis

> [!note]
> - **Time:** O(32 × N) — for each of the 32 bit positions, scan the whole array once.
> - **Space:** O(1).
> - Two other valid approaches exist: (1) a hash map of value → frequency, O(N) time but O(N) space; (2) **sort the array**, then check consecutive triplets — since every triplet of identical values stays adjacent after sorting, a single scan (stepping 3 at a time) finds the one value that breaks the pattern. That's O(N log N) time but only O(1) extra space (aside from mutating the input via sort), often preferred over the bit-counting approach when the array isn't huge, since `N log N` frequently beats `32 × N` in practice for small-to-medium `N`.

## ⚠️ Common Pitfalls

> [!warning]
> - **Assuming plain XOR-everything still works.** It doesn't — three copies of the same number XOR down to that number itself, not `0`, so the "cancel duplicates" trick from Single Number I breaks entirely here.
> - **Off-by-one in the modulo check.** The condition is `count % 3 != 0` (not `== 1`) — while for this problem's guarantee it happens to always land on exactly `1`, phrasing it as `!= 0` is the conceptually correct and more general check.
> - **Trying to invent the O(1)-space, single-pass "bucket" technique from scratch in an interview.** A more advanced variant (tracking two accumulator variables `ones`/`twos` updated via a small set of AND/OR/XOR/NOT rules) achieves this in a single O(N) pass with O(1) space, but it's a genuinely non-obvious trick that's impractical to derive live — know that it exists, but the bit-counting approach above (or the sort-based approach) is what's realistically expected to be produced on the spot.

## 🔗 Related Problems / Next Up

> [!success]
> - **Builds on:** [Single Number I](12-Single-Number-I.md)
> - **Next up:** [Single Number III](14-Single-Number-III.md) — two unique numbers instead of one
> - [↑ Back to Bit Manipulation Index](index.md)
