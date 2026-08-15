---
title: "Remove the Last Set Bit"
tags: [bit-manipulation, and]
difficulty: Easy
---

# Remove the Last Set Bit

## 🎯 What problem are we solving?

> [!question]
> Given a number `n`, clear its **rightmost (least significant) set bit** — turn the lowest `1` into a `0`, leaving every other bit unchanged.

## 💡 Intuition

> [!tip]
> Look at what `n - 1` does to the binary representation: the rightmost set bit turns to `0`, and **every bit to its right** (which were all `0`, since it was the rightmost set bit) turns to `1`. Everything to the *left* of that bit is untouched. So `n` and `n - 1` agree everywhere except at and to the right of the rightmost set bit — and at that specific position, one has `1` while the other has `0`. AND-ing `n & (n - 1)` therefore zeroes out exactly that rightmost set bit (since `1 & 0 = 0` there) while the newly-flipped-to-`1` bits to its right also AND down to `0` (`0 & 1 = 0`), and everything to the left — being identical in both numbers — passes through unchanged.

## 🖼️ Visualizing it

```
n = 12 (1100)
n - 1 = 11 (1011)   -- rightmost set bit (position 2) -> 0, everything right of it -> 1

1100
&1011
-----
1000  = 8   -- the rightmost set bit is gone, left bits preserved
```

```
n = 40 (101000)
n - 1 = 39 (100111)

101000
&100111
-------
100000  = 32   -- rightmost set bit removed
```

## 🛠️ Algorithm / Approach

> [!abstract]
> 1. Compute `n & (n - 1)` and return it.

## 👨‍💻 Code

### Java

```java
public class RemoveLastSetBit {

    public int removeLastSetBit(int n) {
        return n & (n - 1);
    }
}
```

### Python

```python
def remove_last_set_bit(n: int) -> int:
    return n & (n - 1)
```

## ⏱️ Complexity Analysis

> [!note]
> - **Time:** O(1).
> - **Space:** O(1).

## ⚠️ Common Pitfalls

> [!warning]
> - **Calling this on `n = 0`.** There's no set bit to remove — `0 - 1` underflows to a large value in most languages (or a negative number), so this should be guarded against if `0` is a valid input in context.
> - **Assuming this removes the *leftmost* set bit.** It's specifically the rightmost (least significant) one — a common point of confusion when first learning the trick.

## 🔗 Related Problems / Next Up

> [!success]
> - **Directly powers:** [Check if a Number is a Power of Two](08-Check-if-a-Number-is-a-Power-of-Two.md), and one of the two counting techniques in [Count the Number of Set Bits](09-Count-the-Number-of-Set-Bits.md) (Brian Kernighan's algorithm)
> - [↑ Back to Bit Manipulation Index](index.md)
