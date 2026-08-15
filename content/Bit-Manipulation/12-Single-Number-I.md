---
title: "Single Number I"
tags: [bit-manipulation, xor]
difficulty: Easy
---

# Single Number I

## 🎯 What problem are we solving?

> [!question]
> Given an array where every number appears **exactly twice** except for one number, which appears **exactly once**, find that single number.

## 💡 Intuition

> [!tip]
> XOR-ing a value with itself gives `0`, and order doesn't matter for XOR (it's commutative and associative). So XOR-ing the **entire array together** causes every pair of duplicates to cancel out to `0` — no matter where they sit in the array — leaving only the one number that never had a partner to cancel against.

## 🖼️ Visualizing it

```
nums = [4, 1, 2, 1, 2]

xor = 4 ^ 1 ^ 2 ^ 1 ^ 2
    = 4 ^ (1^1) ^ (2^2)       -- regroup, since XOR is commutative/associative
    = 4 ^ 0 ^ 0
    = 4
```

**Result: 4** — the only number without a duplicate to cancel it out.

## 🛠️ Algorithm / Approach

> [!abstract]
> 1. Initialize `xorResult = 0`.
> 2. XOR every element of the array into `xorResult`.
> 3. Return `xorResult` — it's the single number.

## 👨‍💻 Code

### Java

```java
public class SingleNumber {

    public int singleNumber(int[] nums) {
        int xorResult = 0;
        for (int num : nums) {
            xorResult ^= num;
        }
        return xorResult;
    }
}
```

### Python

```python
def single_number(nums: list[int]) -> int:
    xor_result = 0
    for num in nums:
        xor_result ^= num
    return xor_result
```

## ⏱️ Complexity Analysis

> [!note]
> - **Time:** O(N) — a single pass over the array.
> - **Space:** O(1) — no auxiliary data structure needed at all (contrast with the O(N)-space hash-map brute force).

## ⚠️ Common Pitfalls

> [!warning]
> - **Reaching for a hash map by default.** It works (count frequencies, return the key with count `1`), but costs O(N) extra space and generally slower constant factors than the O(1)-space XOR trick — the XOR approach should be the go-to once recognized.
> - **Assuming order or position of the duplicates matters.** It never does — XOR's commutativity guarantees all pairs cancel regardless of where they appear in the array.
> - **Confusing this with [Single Number II](13-Single-Number-II.md) or [Single Number III](14-Single-Number-III.md).** This variant assumes exactly one unique element and all others appearing exactly twice — the moment either of those assumptions changes (three occurrences, or two unique elements), plain XOR-everything no longer works and a different technique is needed.

## 🔗 Related Problems / Next Up

> [!success]
> - **Builds on:** [Introduction to Bit Manipulation](01-Introduction-to-Bit-Manipulation.md) — specifically the XOR operator
> - **Next up:** [Single Number II](13-Single-Number-II.md) — same theme, harder constraint (triples instead of pairs)
> - [↑ Back to Bit Manipulation Index](index.md)
