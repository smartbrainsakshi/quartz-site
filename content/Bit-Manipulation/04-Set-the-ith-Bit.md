---
title: "Set the ith Bit"
tags: [bit-manipulation, or, shift]
difficulty: Easy
---

# Set the ith Bit

## 🎯 What problem are we solving?

> [!question]
> Given a number `n` and an index `i`, turn the `i`th bit **on** (to `1`), leaving every other bit unchanged — regardless of whether that bit was already `1` or `0`.

## 💡 Intuition

> [!tip]
> Build a marker `1 << i` — a number with only bit `i` set. **OR** it with `n`: OR-ing anything with `0` leaves it unchanged, so every bit outside position `i` passes through untouched, while position `i` itself becomes `1` regardless of what it was before (`0 | 1 = 1` and `1 | 1 = 1` both give `1`).

## 🖼️ Visualizing it

```
n = 9 (10001), i = 2  (bit 2 is currently 0)

1 << 2 = 00100
10001
|00100
------
10101  = 13

Bit 2 is now set; every other bit is untouched.
```

```
n = 13 (1101), i = 2  (bit 2 is already 1)

1 << 2 = 0100
1101
|0100
-----
1101  = 13   (unchanged, since it was already set)
```

## 🛠️ Algorithm / Approach

> [!abstract]
> 1. Build the marker `1 << i`.
> 2. Compute `n | (1 << i)` and return it — this is `n` with bit `i` forced to `1`, everything else preserved.

## 👨‍💻 Code

### Java

```java
public class SetIthBit {

    public int setBit(int n, int i) {
        return n | (1 << i);
    }
}
```

### Python

```python
def set_bit(n: int, i: int) -> int:
    return n | (1 << i)
```

## ⏱️ Complexity Analysis

> [!note]
> - **Time:** O(1).
> - **Space:** O(1).

## ⚠️ Common Pitfalls

> [!warning]
> - **Using AND or XOR instead of OR.** AND can only ever turn bits off (or leave them), never on — and XOR would *toggle* the bit instead of unconditionally setting it, silently clearing it if it was already `1`.
> - **Forgetting this is idempotent by design.** Calling it again on an already-set bit is safe and expected — no need to guard against "already set" beforehand.

## 🔗 Related Problems / Next Up

> [!success]
> - **Contrast with:** [Clear the ith Bit](05-Clear-the-ith-Bit.md) — the AND-based mirror image of this operation
> - **Builds on:** [Check if the ith Bit is Set](03-Check-if-the-ith-Bit-is-Set.md)
> - [↑ Back to Bit Manipulation Index](index.md)
