---
title: "Introduction to Bit Manipulation"
tags: [bit-manipulation, foundations, binary, twos-complement, bitwise-operators]
difficulty: Easy
---

# Introduction to Bit Manipulation

## 🎯 What are we learning?

> [!question]
> Before solving any bit-manipulation problems, three foundations are needed: (1) how to convert between decimal and binary by hand and in code, (2) how a computer actually **stores** a number in memory — including negative numbers, via one's and two's complement, and (3) the 5 core **bitwise operators** (AND, OR, XOR, shift, NOT) that every later problem in this playlist builds on.

## 💡 How a computer really stores a number

> [!tip]
> A computer never stores a decimal number like `13` — it only understands `0` and `1`. Writing `int x = 13` causes the computer to convert `13` into its binary form (`1101`) and store *that*. Printing `x` later does the reverse conversion. **The decimal representation only ever exists at the boundary of input/output — internally, everything is binary, always.**
>
> An `int` is **32 bits**. The binary form of `13` (`1101`) only needs 4 bits, so the remaining 28 bits are padded with leading zeros. (`long long` is 64 bits — same concepts, just double the width; this playlist teaches everything in terms of 32-bit `int`, and it extends directly.)

### Decimal → Binary

Repeatedly divide by `2`, recording each remainder, until the quotient reaches `1`. Read the remainders **bottom-to-top** (reverse order of how they were produced).

```
7 / 2 = 3 remainder 1
3 / 2 = 1 remainder 1
(stop at quotient 1)
Read bottom-to-top: 1, 1, 1  →  binary "111"
```

### Binary → Decimal

Index bits from the **right** starting at `0`. Multiply each bit by `2^index` and sum.

```
binary "1101", indices (right to left): 0,1,2,3
1×2^0 + 0×2^1 + 1×2^2 + 1×2^3 = 1 + 0 + 4 + 8 = 13
```

## 🛠️ Algorithm / Approach

> [!abstract]
> **Decimal → Binary (as a function):**
> 1. While `n != 1`: compute `n % 2` (the remainder) and append it to a result string; then `n = n / 2`.
> 2. After the loop, `n` is `1` — append it too.
> 3. Reverse the result string (the digits were collected in the wrong order) and return it.
>
> **Binary → Decimal (as a function):**
> 1. Traverse the binary string **right to left**, tracking a `powerOfTwo` starting at `1` (i.e. `2^0`).
> 2. For each character: if it's `'1'`, add `powerOfTwo` to a running `number`. Then multiply `powerOfTwo` by `2` for the next position (avoids recomputing `2^k` from scratch each time).
> 3. Return `number`.

## 👨‍💻 Code

### Java

```java
public class BinaryDecimalConversion {

    public String convertToBinary(int n) {
        StringBuilder result = new StringBuilder();

        while (n != 1) {
            result.append(n % 2);
            n = n / 2;
        }
        result.append(n);          // append the final 1
        return result.reverse().toString();
    }

    public int convertToDecimal(String binary) {
        int number = 0;
        long powerOfTwo = 1;

        for (int i = binary.length() - 1; i >= 0; i--) {
            if (binary.charAt(i) == '1') {
                number += powerOfTwo;
            }
            powerOfTwo *= 2;
        }
        return number;
    }
}
```

### Python

```python
def convert_to_binary(n: int) -> str:
    result = []
    while n != 1:
        result.append(str(n % 2))
        n //= 2
    result.append(str(n))          # append the final 1
    return ''.join(reversed(result))

def convert_to_decimal(binary: str) -> int:
    number = 0
    power_of_two = 1

    for ch in reversed(binary):
        if ch == '1':
            number += power_of_two
        power_of_two *= 2

    return number
```

## 🖼️ Visualizing negative-number storage (one's & two's complement)

The **31st bit** (the leftmost, in a 32-bit `int`) is reserved as the **sign bit**: `0` for positive, `1` for negative. The remaining bits store the magnitude — but only for **positive** numbers. Negative numbers are stored in their **two's complement** form:

```
One's complement: flip every bit.
  13 = ...0001101  ->  one's complement = ...1110010

Two's complement: one's complement, then add 1.
  ...1110010 + 1 = ...1110011
```

So when the computer needs to store `-3`:
1. Write the plain positive binary for `3`: `...00011`.
2. Flip every bit (one's complement): `...11100`.
3. Add `1` (two's complement): `...11101`.
4. **This is what's actually stored.** The leading `1` (sign bit) tells the computer to interpret it as negative and reverse the process to read it back out.

This is also why the largest and smallest values an `int` can hold aren't symmetric:

```
INT_MAX: sign bit 0, every other bit 1  →  0111...1  =  2^31 - 1
INT_MIN: sign bit 1, every other bit 0  →  1000...0  =  -2^31

(INT_MIN is NOT -(2^31 - 1) -- because the sign bit itself is doing extra
work here: 1000...0's two's complement decodes back to exactly 2^31,
so the negative side of the range extends one further than the positive side.)
```

## 🔧 The 5 core bitwise operators

> [!note]
> **AND (`&`)** — all bits must be `1` to get `1`.
> ```
> 13 & 7:   1101
>         & 0111
>         ------
>           0101  =  5
> ```

> [!note]
> **OR (`|`)** — any bit being `1` gives `1`.
> ```
> 13 | 7:   1101
>         | 0111
>         ------
>           1111  =  15
> ```

> [!note]
> **XOR (`^`)** — an **odd** count of `1`s gives `1`; an **even** count gives `0`.
> ```
> 13 ^ 7:   1101
>         ^ 0111
>         ------
>           1010  =  10
> ```

> [!note]
> **Right Shift (`>>`)** — `x >> k` drops the rightmost `k` bits off the end, equivalent to `x / 2^k` (integer division).
> ```
> 13 >> 1 = 6   (13 / 2 = 6)
> 13 >> 2 = 3   (13 / 4 = 3)
> ```
>
> **Left Shift (`<<`)** — `x << k` appends `k` zero bits on the right, equivalent to `x * 2^k`.
> ```
> 13 << 1 = 26   (13 * 2 = 26)
> ```
> ⚠️ Left shift can **overflow** past `INT_MAX` exactly like ordinary multiplication by 2 can — shifting `INT_MAX` left by even `1` produces garbage.

> [!note]
> **NOT (`~`)** — flips every bit, then, if the result's sign bit ends up `1`, the value is read back out via two's complement (same decode process as storing any negative number).
> ```
> ~5:  0...0101 (5)  -> flip ->  1...1010
>      sign bit is 1 -> negative -> decode via two's complement -> value is -6
> ~(-6): 1...1010 (-6's stored form) -> flip -> 0...0101 -> sign bit 0 -> positive -> 5
> ```

## ⏱️ Complexity Analysis

> [!note]
> - **Time:** the decimal↔binary conversions are O(log N) — the number of bits needed to represent `N` scales with `log₂ N`. Every individual bitwise operator application (AND, OR, XOR, shift, NOT) is O(1) — the CPU performs it directly on a fixed-width word in a single instruction.
> - **Space:** O(log N) for the conversion functions (the result string/number of bits scales with `log₂ N`); O(1) for applying any single bitwise operator.

## ⚠️ Common Pitfalls

> [!warning]
> - **Assuming the computer "understands" decimal digits internally.** It never does — every stored integer is binary, always; decimal only exists at input/output time.
> - **Forgetting the sign bit is reserved.** Treating all 32 bits as pure magnitude leads to wrong reasoning about `INT_MAX`/`INT_MIN` and about how negative numbers are actually laid out in memory.
> - **Confusing one's complement (just flip) with two's complement (flip, then add 1).** Two's complement — not one's complement — is what's actually stored for negative numbers.
> - **Not accounting for left-shift overflow.** `x << k` can silently overflow exactly like `x * 2^k` can — always sanity-check against `INT_MAX` when shifting large values left.

## 🔗 Related Problems / Next Up

> [!success]
> This is the foundation for every problem in the section — next up: [Swap Two Numbers](02-Swap-Two-Numbers.md), which puts the XOR operator to its first real use.
> - [↑ Back to Bit Manipulation Index](index.md)
