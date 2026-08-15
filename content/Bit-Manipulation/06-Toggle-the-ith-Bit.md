---
title: "Toggle the ith Bit"
tags: [bit-manipulation, xor, shift]
difficulty: Easy
---

# Toggle the ith Bit

## 🎯 What problem are we solving?

> [!question]
> Given a number `n` and an index `i`, **flip** the `i`th bit — `1` becomes `0`, and `0` becomes `1` — leaving every other bit unchanged.

## 💡 Intuition

> [!tip]
> XOR-ing a bit with `1` always flips it (`0^1=1`, `1^1=0`), while XOR-ing with `0` always leaves it unchanged (`0^0=0`, `1^0=1`). Build the usual marker `1 << i` (a `1` only at position `i`, `0` everywhere else) and XOR it directly with `n` — position `i` flips, every other position passes through untouched.

## 🖼️ Visualizing it

```
n = 13 (1101), i = 2  (bit 2 is currently 1 -> should become 0)

1 << 2 = 0100
1101
^0100
-----
1001  = 9   -- bit 2 toggled off, rest unchanged
```

```
n = 13 (1101), i = 1  (bit 1 is currently 0 -> should become 1)

1 << 1 = 0010
1101
^0010
-----
1111  = 15   -- bit 1 toggled on, rest unchanged
```

## 🛠️ Algorithm / Approach

> [!abstract]
> 1. Build the marker `1 << i`.
> 2. Compute `n ^ (1 << i)` and return it — bit `i` flips, everything else preserved.

## 👨‍💻 Code

### Java

```java
public class ToggleIthBit {

    public int toggleBit(int n, int i) {
        return n ^ (1 << i);
    }
}
```

### Python

```python
def toggle_bit(n: int, i: int) -> int:
    return n ^ (1 << i)
```

## ⏱️ Complexity Analysis

> [!note]
> - **Time:** O(1).
> - **Space:** O(1).

## ⚠️ Common Pitfalls

> [!warning]
> - **Confusing this with "set" or "clear."** Toggle is unconditional flip, not a forced `1` (that's [Set](04-Set-the-ith-Bit.md), via OR) or forced `0` (that's [Clear](05-Clear-the-ith-Bit.md), via AND-with-NOT) — using the wrong operator silently produces the wrong bit-manipulation problem's answer.
> - **Applying it twice by accident.** Toggling the same bit twice returns it to its original value — easy to lose track of in code paths that might call this more than once unintentionally.

## 🔗 Related Problems / Next Up

> [!success]
> - **Companions:** [Set the ith Bit](04-Set-the-ith-Bit.md), [Clear the ith Bit](05-Clear-the-ith-Bit.md)
> - **Next up:** [Remove the Last Set Bit](07-Remove-the-Last-Set-Bit.md)
> - [↑ Back to Bit Manipulation Index](index.md)
