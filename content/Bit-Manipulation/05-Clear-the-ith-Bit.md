---
title: "Clear the ith Bit"
tags: [bit-manipulation, and, not, shift]
difficulty: Easy
---

# Clear the ith Bit

## 🎯 What problem are we solving?

> [!question]
> Given a number `n` and an index `i`, turn the `i`th bit **off** (to `0`), leaving every other bit unchanged — regardless of whether that bit was already `1` or `0`.

## 💡 Intuition

> [!tip]
> The mirror image of [Set the ith Bit](04-Set-the-ith-Bit.md), using AND instead of OR. AND-ing with `1` leaves a bit unchanged; AND-ing with `0` forces it to `0`. So build a mask that's `0` at position `i` and `1` everywhere else: start with the usual marker `1 << i` (which is `1` *only* at position `i`), then **NOT** it — flipping every bit turns position `i` into `0` and every other position into `1`, exactly the mask needed.

## 🖼️ Visualizing it

```
n = 13 (1101), i = 2  (bit 2 is currently 1, target: turn it off)

1 << 2        = 0100
NOT(1 << 2)   = 1011   (all bits flipped: position 2 -> 0, everywhere else -> 1)

1101
&1011
-----
1001  = 9

Bit 2 is now cleared; every other bit is untouched.
```

## 🛠️ Algorithm / Approach

> [!abstract]
> 1. Build the marker `1 << i`.
> 2. Negate it: `~(1 << i)` — this is `0` at position `i`, `1` everywhere else.
> 3. Compute `n & ~(1 << i)` and return it — bit `i` is forced to `0`, everything else preserved (AND with `1` passes through unchanged).

## 👨‍💻 Code

### Java

```java
public class ClearIthBit {

    public int clearBit(int n, int i) {
        return n & ~(1 << i);
    }
}
```

### Python

```python
def clear_bit(n: int, i: int) -> int:
    return n & ~(1 << i)
```

## ⏱️ Complexity Analysis

> [!note]
> - **Time:** O(1).
> - **Space:** O(1).

## ⚠️ Common Pitfalls

> [!warning]
> - **Forgetting the NOT step and AND-ing with the plain marker instead of its negation.** `n & (1 << i)` *isolates* bit `i` rather than clearing it — the negation is what turns the marker into a mask that clears exactly one position while preserving the rest.
> - **Python's `~` on arbitrary-width integers.** Unlike a fixed 32-bit language, Python integers aren't width-limited, so `~x` behaves as `-x - 1` conceptually — the bit pattern reasoning still holds correctly for masking purposes here, but it's worth knowing this isn't identical to a 32-bit two's-complement flip if inspected directly.

## 🔗 Related Problems / Next Up

> [!success]
> - **Contrast with:** [Set the ith Bit](04-Set-the-ith-Bit.md)
> - **Next up:** [Toggle the ith Bit](06-Toggle-the-ith-Bit.md)
> - [↑ Back to Bit Manipulation Index](index.md)
