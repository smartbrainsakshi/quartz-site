---
title: "Check if the ith Bit is Set"
tags: [bit-manipulation, and, shift]
difficulty: Easy
---

# Check if the ith Bit is Set

## 🎯 What problem are we solving?

> [!question]
> Given a number `n` and an index `i` (bits counted from the right, starting at `0`), determine whether the `i`th bit of `n` is `1` (set) or `0` (unset).

## 💡 Intuition

> [!tip]
> There are two symmetric ways to isolate a single bit for inspection:
> - **Bring a marker to the bit:** build `1 << i` — a number with only bit `i` set, everything else `0`. AND-ing it with `n` zeroes out every bit except position `i`; the result is non-zero if and only if that bit was set in `n`.
> - **Bring the bit to a marker:** right-shift `n` by `i` places, sliding the bit of interest all the way down to position `0`. Then AND with `1` to isolate just that bottom bit.

## 🖼️ Visualizing it

```
n = 13 (1101), i = 2

Method 1 (shift the marker to the bit):
  1 << 2 = 100
  1101 & 0100 = 0100  -> non-zero -> bit is SET

Method 2 (shift the bit down to the marker):
  13 >> 2 = 11 (binary), i.e. 3
  3 & 1 = 1 -> non-zero -> bit is SET
```

```
n = 13 (1101), i = 1  (checking the unset bit this time)

Method 1: 1 << 1 = 010;  1101 & 0010 = 0000 -> zero -> bit is UNSET
```

## 🛠️ Algorithm / Approach

> [!abstract]
> **Method 1:** compute `n & (1 << i)`. If the result is non-zero, the bit is set; otherwise it's unset.
>
> **Method 2:** compute `(n >> i) & 1`. If the result is `1`, the bit is set; if `0`, it's unset.

## 👨‍💻 Code

### Java

```java
public class CheckIthBit {

    public boolean isSetMethod1(int n, int i) {
        return (n & (1 << i)) != 0;
    }

    public boolean isSetMethod2(int n, int i) {
        return ((n >> i) & 1) == 1;
    }
}
```

### Python

```python
def is_set_method1(n: int, i: int) -> bool:
    return (n & (1 << i)) != 0

def is_set_method2(n: int, i: int) -> bool:
    return ((n >> i) & 1) == 1
```

## ⏱️ Complexity Analysis

> [!note]
> - **Time:** O(1) — a fixed handful of bitwise operations regardless of `n`'s size.
> - **Space:** O(1).

## ⚠️ Common Pitfalls

> [!warning]
> - **Checking `n & (1 << i) == 1`** instead of `!= 0`. The AND result carries the bit at its *original* position (e.g. `0100`, which is `4`, not `1`) — only comparing against `0` is correct for Method 1; only the shifted-down version in Method 2 can be safely compared to exactly `1`.
> - **Miscounting bit indices.** Bits are indexed from the right starting at `0` — off-by-one errors here silently check the wrong bit entirely.
> - **Using this on a negative number without considering two's complement.** The bit pattern of a negative number is its two's complement form, not its "plain" magnitude — see [Introduction to Bit Manipulation](01-Introduction-to-Bit-Manipulation.md).

## 🔗 Related Problems / Next Up

> [!success]
> - **Builds on:** [Introduction to Bit Manipulation](01-Introduction-to-Bit-Manipulation.md)
> - **Next up:** [Set the ith Bit](04-Set-the-ith-Bit.md)
> - [↑ Back to Bit Manipulation Index](index.md)
