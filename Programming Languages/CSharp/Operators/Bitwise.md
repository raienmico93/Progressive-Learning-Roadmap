# C# Bitwise Operators — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
Bitwise operators in C# are symbols that perform operations on the individual binary digits (bits) of integral data types. Instead of treating a number as a single value, bitwise operators let you inspect, set, clear, toggle, and shift the individual `0` and `1` bits that make up that number. They are the primary tool for low-level programming, flag manipulation, and performance-critical optimizations in C#.

**Technical Definition**
The bitwise and shift operators include the unary bitwise complement (`~`), the binary left shift (`<<`), right shift (`>>`), unsigned right shift (`>>>`), and the binary logical AND (`&`), OR (`|`), and exclusive OR (`^`) operators. These operators take operands of the integral numeric types (`int`, `uint`, `long`, `ulong`, `nint`, `nuint`) or the `char` type. When both operands are of smaller integral types (`sbyte`, `byte`, `short`, `ushort`, `char`), their values are converted to `int`, which is also the result type; when operands are of different integral types, they are converted to the closest containing integral type. Bitwise and shift operations **never cause overflow** and produce the same results in `checked` and `unchecked` contexts. The `&`, `|`, and `^` operators are also defined for `bool` operands, where they perform non-short-circuiting logical operations.

**Beginner-Friendly Explanation**
Bitwise operators let you work at the **smallest possible level of your data**: individual bits. Imagine a number as a row of light switches, each either on (`1`) or off (`0`). Bitwise operators let you turn specific switches on or off, check whether a switch is on, flip switches, or slide the whole row left or right. This is useful when you need to pack multiple true/false flags into a single number, squeeze every bit of performance out of a calculation, or talk to hardware that expects raw binary data.

---

### Key Characteristics

- **Operate on individual bits** — Bitwise operators view integral values as sequences of bits, not as whole numbers.
- **Seven operators** — Complement (`~`), left shift (`<<`), right shift (`>>`), unsigned right shift (`>>>`), AND (`&`), OR (`|`), and XOR (`^`).
- **No overflow** — Bitwise and shift operations never cause overflow and behave identically in `checked` and `unchecked` contexts.
- **Numeric promotion** — Smaller integral types are promoted to `int` for the operation; mixed types are promoted to the closest containing type.
- **Dual behaviour** — `&`, `|`, and `^` act as bitwise operators on integral types and as non-short-circuiting logical operators on `bool` operands.
- **Defined precedence** — Complement has the highest precedence, then shifts, then AND, then XOR, then OR; all binary bitwise operators are left-associative.
- **Shift count masking** — For `int`/`uint`, only the low 5 bits of the shift count are used; for `long`/`ulong`, the low 6 bits are used.

---

### Prerequisites

- Understanding of C#’s integral types (`int`, `uint`, `long`, `ulong`, `byte`, etc.) and their ranges.
- Familiarity with binary representation and two’s complement for signed numbers.
- Knowledge of operator precedence and associativity.
- Awareness of `checked` and `unchecked` contexts (though bitwise operations are unaffected by them).
- Basic understanding of `enum` types and the `[Flags]` attribute for flag manipulation.

---

### Related Programming Areas

- **Flag enums** — `[Flags]` enums use bitwise operations to combine and test options.
- **Low-level programming** — Hardware registers, memory-mapped I/O, and binary protocols.
- **Performance optimization** — Replacing multiplication/division by powers of two with shifts.
- **Cryptography and checksums** — XOR and bit manipulation are fundamental to hash functions and ciphers.
- **Binary serialization** — `BitConverter` and `BinaryPrimitives` convert between primitives and byte arrays using endianness.

---

### Core Concepts / Features

1. Bitwise Complement (`~`)
2. Left Shift (`<<`)
3. Right Shift (`>>`)
4. Unsigned Right Shift (`>>>`, C# 11)
5. Bitwise AND (`&`)
6. Bitwise XOR (`^`)
7. Bitwise OR (`|`)
8. Compound Assignment Operators
9. Bit Manipulation (Setting, Clearing, Toggling, Testing Bits)
10. Bit Masks
11. Bitwise Applications
12. `BitConverter`
13. `BinaryPrimitives`
14. Operator Precedence and Associativity
15. Best Practices

Each concept follows a uniform structure: definitions, purposes, syntax rules and structure, annotated examples with expected outputs, real-world cases, and references.

---

## 1. Bitwise Complement (`~`)

### Definitions

**Core Definition**
The bitwise complement operator (`~`) is a unary operator that reverses every bit in its operand: every `0` becomes `1`, and every `1` becomes `0`.

**Technical Definition**
The `~` operator produces a bitwise complement of its operand by reversing each bit. For signed types, this is equivalent to `-(x + 1)` because of two’s complement representation. The operator is defined for `int`, `uint`, `long`, and `ulong`; smaller types are promoted to `int`. The `~` symbol is also used to declare finalizers, but that is a distinct syntactic role.

**Beginner-Friendly Explanation**
`~` flips every switch. If a bit is on, it turns off; if it is off, it turns on. It is the “invert everything” operator.

---

### Purposes

- To invert all bits in a value.
- To compute the bitwise complement for masks (e.g., clearing bits with `value & ~mask`).
- To implement one’s complement arithmetic (though rarely used directly).
- To produce negative values from positive ones via two’s complement (`~x == -x - 1`).

---

### Syntax Rules and Structure

```
~operand
```

**Component breakdown**

- `~` — The bitwise complement operator (prefix unary).
- `operand` — An integral value.

#### Syntax Rules

- The operand must be an integral type or `char`.
- The result type is `int` for smaller types, otherwise the operand’s type.
- Never causes overflow.

#### Constraints and Limitations

- For signed types, the result may be negative; this is language-defined (two’s complement).

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Complement of an Integer

```csharp
using System;

class Program
{
    static void Main()
    {
        int x = 12;   // binary: 0000 1100
        int result = ~x;
        Console.WriteLine(result);        // -13
        Console.WriteLine(Convert.ToString(result, 2)); // 11111111111111111111111111110011
    }
}
```

**Expected Output**

```
-13
11111111111111111111111111110011
```

**Why this output occurs**

- `12` in binary is `0000 1100`. Flipping every bit gives `1111 0011` in the low byte, but as a 32-bit `int`, all high bits become `1`, producing the two’s complement representation of `-13`.

---

#### Example 2: Clearing Bits with `~`

```csharp
using System;

class Program
{
    static void Main()
    {
        byte value = 0b_1111_1111;   // 255
        byte mask  = 0b_0000_1111;   // 15
        byte result = (byte)(value & ~mask);
        Console.WriteLine(Convert.ToString(result, 2)); // 11110000
    }
}
```

**Expected Output**

```
11110000
```

**Why this output occurs**

- `~mask` flips the mask bits, producing a mask that keeps only the bits **not** set in `mask`. ANDing clears those bits in `value`.

---

### Real-World Cases

- **Clearing flag bits** — `permissions &= ~Permissions.Write`.
- **Creating inverse masks** — Keeping all bits except specific ones.
- **Two’s complement arithmetic** — Understanding negative number representation.

---

### References

- Bitwise complement operator `~` — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/bitwise-and-shift-operators#bitwise-complement-operator-

---

## 2. Left Shift (`<<`)

### Definitions

**Core Definition**
The left shift operator (`<<`) shifts the bits of its left operand to the left by the number of positions specified by its right operand, filling the vacated low-order bits with zeros.

**Technical Definition**
The `<<` operator shifts its left-hand operand to the left by the number of bits specified by its right-hand operand. The left-shift operation discards the high-order bits that fall outside the range of the result type and sets the low-order empty bit positions to zero. Because shift operators are defined only for `int`, `uint`, `long`, and `ulong`, the result always contains at least 32 bits; smaller types are promoted to `int`.

**Beginner-Friendly Explanation**
`<<` slides the bits to the left. Each shift left is like multiplying by `2` (for positive numbers without overflow). Bits that fall off the left end are lost, and zeros fill the right end.

---

### Purposes

- To multiply an integer by a power of two efficiently.
- To pack bits into specific positions (e.g., building bit masks).
- To combine flag values by shifting bits into place.
- To implement low-level encoding and decoding.

---

### Syntax Rules and Structure

```
value   <<   count
```

**Component breakdown**

- `value` — The integral value to shift.
- `<<` — The left-shift operator.
- `count` — The number of bit positions to shift.

#### Syntax Rules

- The left operand must be an integral type.
- The shift count is masked: for `int`/`uint`, the low 5 bits are used (`count & 0x1F`); for `long`/`ulong`, the low 6 bits are used (`count & 0x3F`).
- Bits shifted out of the high end are discarded; zeros fill the low end.

#### Constraints and Limitations

- Shifting by a count larger than the bit width wraps the count (e.g., shifting an `int` by 32 is equivalent to shifting by 0).
- Left shift can change the sign bit for signed types, potentially producing negative results.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Multiplying by Powers of Two

```csharp
using System;

class Program
{
    static void Main()
    {
        int x = 3;
        Console.WriteLine(x << 1);   // 6  (3 * 2)
        Console.WriteLine(x << 2);   // 12 (3 * 4)
        Console.WriteLine(x << 3);   // 24 (3 * 8)
    }
}
```

**Expected Output**

```
6
12
24
```

**Why this output occurs**

- Each left shift by `n` multiplies by `2^n` (assuming no overflow).

---

#### Example 2: Building a Bit Mask

```csharp
using System;

class Program
{
    static void Main()
    {
        int bit3 = 1 << 3;   // 8 (binary 1000)
        int bit5 = 1 << 5;   // 32 (binary 100000)
        Console.WriteLine(bit3);   // 8
        Console.WriteLine(bit5);   // 32
    }
}
```

**Expected Output**

```
8
32
```

**Why this output occurs**

- `1 << n` creates a value with only bit `n` set.

---

### Real-World Cases

- **Flag definitions** — `Read = 1 << 0`, `Write = 1 << 1`, `Execute = 1 << 2`.
- **Packing fields** — Combining values into a single integer.
- **Fast multiplication** — Replacing `x * 8` with `x << 3` in performance-critical code.

---

### References

- Left-shift operator `<<` — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/bitwise-and-shift-operators#left-shift-operator-

---

## 3. Right Shift (`>>`)

### Definitions

**Core Definition**
The right shift operator (`>>`) shifts the bits of its left operand to the right by the number of positions specified by its right operand. The vacated high-order bits are filled differently depending on whether the type is signed or unsigned.

**Technical Definition**
The `>>` operator shifts its left-hand operand right by the number of bits defined by its right-hand operand. The low-order bits are discarded. For `int` or `long` (signed types), the operator performs an **arithmetic shift**: the sign bit is propagated to the high-order empty positions (zeros for non-negative, ones for negative). For `uint` or `ulong` (unsigned types), it performs a **logical shift**: high-order empty bits are always set to zero.

**Beginner-Friendly Explanation**
`>>` slides bits to the right. For positive numbers, it is like dividing by `2` (discarding the remainder). For negative signed numbers, the sign is preserved by filling the left side with `1`s. For unsigned numbers, it always fills with `0`s.

---

### Purposes

- To divide an integer by a power of two efficiently.
- To extract specific bits or fields from a packed value.
- To implement arithmetic on packed data.
- To shift signed values while preserving the sign (arithmetic shift).

---

### Syntax Rules and Structure

```
value   >>   count
```

**Component breakdown**

- `value` — The integral value to shift.
- `>>` — The right-shift operator.
- `count` — The number of bit positions to shift.

#### Syntax Rules

- Signed types (`int`, `long`) perform an arithmetic shift (sign-extending).
- Unsigned types (`uint`, `ulong`) perform a logical shift (zero-filling).
- The shift count is masked like the left shift (`& 0x1F` for 32-bit, `& 0x3F` for 64-bit).

#### Constraints and Limitations

- Arithmetic right shift of negative numbers rounds toward negative infinity (e.g., `-1 >> 1 == -1`).
- Use `>>>` for a logical right shift on signed types.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Arithmetic vs. Logical Right Shift

```csharp
using System;

class Program
{
    static void Main()
    {
        int signed = -8;          // binary: ...11111000
        uint unsigned = 0xFFFFFFF8;

        Console.WriteLine(signed >> 1);    // -4  (arithmetic: sign preserved)
        Console.WriteLine(unsigned >> 1);  // 2147483644 (logical: zero-filled)
    }
}
```

**Expected Output**

```
-4
2147483644
```

**Why this output occurs**

- For `signed`, the sign bit is propagated, dividing by `2` while preserving the negative sign.
- For `unsigned`, zeros are shifted in from the left, producing a large positive value.

---

#### Example 2: Extracting a Bit Field

```csharp
using System;

class Program
{
    static void Main()
    {
        int packed = 0b_1101_0110;   // 214
        int highNibble = (packed >> 4) & 0xF;
        Console.WriteLine(Convert.ToString(highNibble, 2)); // 1101
    }
}
```

**Expected Output**

```
1101
```

**Why this output occurs**

- Shifting right by `4` moves the high nibble to the low position; masking with `0xF` isolates it.

---

### Real-World Cases

- **Dividing by powers of two** — `x >> 1` is `x / 2` for positive integers.
- **Extracting packed fields** — Pulling out colour channels, flags, or protocol fields.
- **Sign extension** — Preserving the sign of negative numbers during shifts.

---

### References

- Right-shift operator `>>` — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/bitwise-and-shift-operators#right-shift-operator-

---

## 4. Unsigned Right Shift (`>>>`, C# 11)

### Definitions

**Core Definition**
The unsigned right shift operator (`>>>`, introduced in C# 11) always performs a **logical** right shift, filling the high-order vacated bits with zeros regardless of the operand’s signedness.

**Technical Definition**
The `>>>` operator shifts its left-hand operand right by the number of bits defined by its right-hand operand. Unlike `>>`, which performs an arithmetic shift (sign-propagating) for signed types, `>>>` always performs a **logical** shift: the high-order empty bit positions are always set to zero, regardless of the type of the left-hand operand. This provides a clean way to perform a logical right shift on signed types without casting to an unsigned type and back.

**Beginner-Friendly Explanation**
`>>>` is the “always fill with zeros” right shift. Unlike `>>`, which preserves the sign of negative numbers by filling with `1`s, `>>>` treats the number as if it were unsigned, shifting zeros in from the left.

---

### Purposes

- To perform a logical right shift on signed types without unsafe casts.
- To treat a signed value as a collection of bits rather than a signed quantity.
- To implement cryptographic and hashing algorithms that require logical shifts.
- To avoid the pitfalls of casting to unsigned and back.

---

### Syntax Rules and Structure

```
value   >>>   count
```

**Component breakdown**

- `value` — The integral value to shift.
- `>>>` — The unsigned right-shift operator.
- `count` — The number of bit positions to shift.

#### Syntax Rules

- Available in **C# 11** and later.
- Always performs a logical shift (zero-filling) regardless of the operand’s type.
- The shift count is masked like `>>` and `<<`.

#### Constraints and Limitations

- For unsigned types, `>>>` and `>>` produce the same result; `>>>` is primarily useful for signed types.
- Requires C# 11 or later.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: `>>` vs. `>>>` on a Negative Signed Value

```csharp
using System;

class Program
{
    static void Main()
    {
        int value = -8;   // binary: ...11111000

        Console.WriteLine(value >> 1);    // -4  (arithmetic, sign-preserving)
        Console.WriteLine(value >>> 1);   // 2147483644 (logical, zero-filling)
    }
}
```

**Expected Output**

```
-4
2147483644
```

**Why this output occurs**

- `>>` propagates the sign bit, resulting in `-4`.
- `>>>` treats the bit pattern as unsigned, shifting in zeros and producing a large positive value.

---

#### Example 2: Logical Shift Without Casting

```csharp
using System;

class Program
{
    static void Main()
    {
        int signed = 0x80000000;   // int.MinValue
        // int logicalOld = (int)((uint)signed >> 1); // old workaround
        int logicalNew = signed >>> 1;
        Console.WriteLine(logicalNew); // 1073741824
    }
}
```

**Expected Output**

```
1073741824
```

**Why this output occurs**

- `>>>` shifts the bits logically, producing the same result as casting to `uint`, shifting, and casting back, but without the casts.

---

### Real-World Cases

- **Hash functions** — Logical shifts are common in mixing functions.
- **Cryptographic algorithms** — Bit-level operations on signed words.
- **Protocol implementations** — Treating signed fields as raw bit patterns.

---

### References

- Unsigned right-shift operator `>>>` — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/bitwise-and-shift-operators#unsigned-right-shift-operator-
- C# 11 unsigned right shift operator proposal - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/proposals/csharp-11.0/unsigned-right-shift-operator

---

## 5. Bitwise AND (`&`)

### Definitions

**Core Definition**
The bitwise AND operator (`&`) compares each bit of two integral operands and produces a result bit that is `1` only when **both** corresponding bits are `1`.

**Technical Definition**
The `&` operator computes the bitwise logical AND of its integral operands. For `bool` operands, it computes the logical AND (non-short-circuiting). The unary `&` operator is also the address-of operator in unsafe contexts. The operator is defined for the standard integral types and enums.

**Beginner-Friendly Explanation**
`&` keeps a bit only if **both** numbers have that bit turned on. It is the “both must agree” operator. It is commonly used to check whether a specific bit is set or to mask out unwanted bits.

---

### Purposes

- To test whether a specific bit is set (`value & mask != 0`).
- To clear (mask out) unwanted bits (`value & mask`).
- To extract specific bits from a packed value.
- To combine flags in a non-short-circuiting logical context.

---

### Syntax Rules and Structure

```
leftOperand   &   rightOperand
```

**Component breakdown**

- `leftOperand` — The first integral (or `bool`) operand.
- `&` — The bitwise AND operator.
- `rightOperand` — The second integral (or `bool`) operand.

#### Syntax Rules

- Both operands must be integral types or `bool`.
- The result has the promoted type of the operands.
- For `bool`, both operands are always evaluated (non-short-circuiting).

#### Constraints and Limitations

- Do not confuse with `&&` (conditional AND), which is short-circuiting for `bool`.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Testing a Bit

```csharp
using System;

class Program
{
    static void Main()
    {
        int flags = 0b_1010;   // 10
        int mask  = 0b_0010;   // 2

        bool isSet = (flags & mask) != 0;
        Console.WriteLine(isSet);   // True
    }
}
```

**Expected Output**

```
True
```

**Why this output occurs**

- `flags & mask` isolates bit 1; since it is `1`, the result is non-zero.

---

#### Example 2: Masking Out Unwanted Bits

```csharp
using System;

class Program
{
    static void Main()
    {
        byte value = 0b_1111_1111;
        byte mask  = 0b_0000_1111;
        byte result = (byte)(value & mask);
        Console.WriteLine(Convert.ToString(result, 2)); // 1111
    }
}
```

**Expected Output**

```
1111
```

**Why this output occurs**

- ANDing with `0000 1111` keeps only the low four bits, clearing the high four.

---

### Real-World Cases

- **Permission checks** — `if ((permissions & Permissions.Read) != 0)`.
- **Colour extraction** — Isolating red, green, or blue channels from a packed pixel.
- **Hash table bucketing** — `hash & (bucketCount - 1)` when bucket count is a power of two.

---

### References

- Logical AND operator `&` — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/bitwise-and-shift-operators#logical-and-operator-

---

## 6. Bitwise XOR (`^`)

### Definitions

**Core Definition**
The bitwise exclusive OR operator (`^`) compares each bit of two integral operands and produces a result bit that is `1` when the corresponding bits **differ**.

**Technical Definition**
The `^` operator computes the bitwise logical exclusive OR (XOR) of its integral operands. For `bool` operands, it computes the logical exclusive OR. For `bool`, `^` produces the same result as `!=`.

**Beginner-Friendly Explanation**
`^` turns a bit on only when the two input bits are **different**. It is the “exactly one” operator. It is useful for toggling bits and for checksums.

---

### Purposes

- To toggle specific bits (`value ^ mask`).
- To compare two values and find the bits that differ.
- To implement simple encryption and checksums.
- To swap two variables without a temporary variable (though this is rarely necessary).

---

### Syntax Rules and Structure

```
leftOperand   ^   rightOperand
```

**Component breakdown**

- `leftOperand` — The first integral (or `bool`) operand.
- `^` — The bitwise XOR operator.
- `rightOperand` — The second integral (or `bool`) operand.

#### Syntax Rules

- Both operands must be integral or `bool`.
- The result has the promoted type.
- XOR is associative and commutative.

#### Constraints and Limitations

- For `bool`, `^` is equivalent to `!=`.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Toggling a Bit

```csharp
using System;

class Program
{
    static void Main()
    {
        int value = 0b_1010;
        int toggleMask = 0b_0010;
        int result = value ^ toggleMask;
        Console.WriteLine(Convert.ToString(result, 2)); // 1000
    }
}
```

**Expected Output**

```
1000
```

**Why this output occurs**

- XOR with `0010` flips bit 1 from `1` to `0`, leaving other bits unchanged.

---

#### Example 2: Finding Differing Bits

```csharp
using System;

class Program
{
    static void Main()
    {
        int a = 0b_1100;
        int b = 0b_1010;
        Console.WriteLine(Convert.ToString(a ^ b, 2)); // 0110
    }
}
```

**Expected Output**

```
0110
```

**Why this output occurs**

- Bits that differ between `1100` and `1010` are positions 1 and 2, producing `0110`.

---

### Real-World Cases

- **Toggle flags** — `state ^= ToggleMask`.
- **Checksums** — XOR all bytes to detect corruption.
- **Simple encryption** — XOR with a key (though not secure for real cryptography).
- **Finding differences** — Comparing two bit patterns.

---

### References

- Logical exclusive OR operator `^` — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/bitwise-and-shift-operators#logical-exclusive-or-operator-

---

## 7. Bitwise OR (`|`)

### Definitions

**Core Definition**
The bitwise OR operator (`|`) compares each bit of two integral operands and produces a result bit that is `1` when **at least one** corresponding bit is `1`.

**Technical Definition**
The `|` operator computes the bitwise logical OR of its integral operands. For `bool` operands, it computes the logical OR (non-short-circuiting).

**Beginner-Friendly Explanation**
`|` turns a bit on if **either** number has that bit on. It is the “at least one” operator. It is used to combine flags and set bits.

---

### Purposes

- To combine flag values (`Read | Write`).
- To set specific bits (`value | mask`).
- To merge bit patterns.
- To perform non-short-circuiting logical OR on `bool` operands.

---

### Syntax Rules and Structure

```
leftOperand   |   rightOperand
```

**Component breakdown**

- `leftOperand` — The first integral (or `bool`) operand.
- `|` — The bitwise OR operator.
- `rightOperand` — The second integral (or `bool`) operand.

#### Syntax Rules

- Both operands must be integral or `bool`.
- The result has the promoted type.
- For `bool`, both operands are always evaluated.

#### Constraints and Limitations

- Do not confuse with `||` (conditional OR), which is short-circuiting for `bool`.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Combining Flags

```csharp
using System;

[Flags]
enum Permissions { None = 0, Read = 1, Write = 2, Execute = 4 }

class Program
{
    static void Main()
    {
        Permissions p = Permissions.Read | Permissions.Write;
        Console.WriteLine(p);   // Read, Write
    }
}
```

**Expected Output**

```
Read, Write
```

**Why this output occurs**

- OR combines the bits of `Read` (1) and `Write` (2) into `3`.

---

#### Example 2: Setting Bits

```csharp
using System;

class Program
{
    static void Main()
    {
        byte value = 0b_0000_0000;
        byte mask  = 0b_0000_1010;
        byte result = (byte)(value | mask);
        Console.WriteLine(Convert.ToString(result, 2)); // 1010
    }
}
```

**Expected Output**

```
1010
```

**Why this output occurs**

- OR sets the bits specified by the mask.

---

### Real-World Cases

- **Permission systems** — Combining access flags.
- **Configuration** — Merging option flags.
- **Protocol headers** — Setting multiple header bits simultaneously.

---

### References

- Logical OR operator `|` — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/bitwise-and-shift-operators#logical-or-operator-

---

## 8. Compound Assignment Operators

### Definitions

**Core Definition**
Compound assignment operators (`&=`, `|=`, `^=`, `<<=`, `>>=`, `>>>=`) combine a bitwise operation with assignment, storing the result back into the left-hand operand.

**Technical Definition**
For a binary operator `op`, `x op= y` is equivalent to `x = x op y`, except that `x` is evaluated only once. When the result of `op` is not implicitly convertible to the type of `x`, the compound assignment is evaluated as `x = (T)(x op y)`, where `T` is the type of `x`, again with single evaluation of `x`.

**Beginner-Friendly Explanation**
These are shortcuts that save you from writing the variable name twice. `flags |= mask` sets bits; `flags &= ~mask` clears bits; `value <<= 3` shifts and stores.

---

### Purposes

- To update a variable with a bitwise operation concisely.
- To set, clear, and toggle bits in place.
- To apply shifts and store the result.
- To evaluate the left-hand side only once.

---

### Syntax Rules and Structure

```
leftHandSide   op=   rightHandSide   ;
```

**Component breakdown**

- `leftHandSide` — The target variable.
- `op=` — One of `&=`, `|=`, `^=`, `<<=`, `>>=`, `>>>=`.
- `rightHandSide` — The operand for the operation.

#### Syntax Rules

- The left-hand side is evaluated only once.
- The result is implicitly cast back to the left-hand type if necessary.
- `>>>=` requires C# 11+.

#### Constraints and Limitations

- The left-hand side must be an assignable variable.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Setting and Clearing Bits

```csharp
using System;

class Program
{
    static void Main()
    {
        int flags = 0;
        flags |= 0b_1010;   // set bits
        Console.WriteLine(Convert.ToString(flags, 2)); // 1010

        flags &= ~0b_0010;  // clear bit 1
        Console.WriteLine(Convert.ToString(flags, 2)); // 1000
    }
}
```

**Expected Output**

```
1010
1000
```

**Why this output occurs**

- `|=` sets bits; `&= ~` clears specific bits.

---

#### Example 2: Shift Assignment

```csharp
using System;

class Program
{
    static void Main()
    {
        int x = 2;
        x <<= 3;   // x = x << 3
        Console.WriteLine(x); // 16
    }
}
```

**Expected Output**

```
16
```

**Why this output occurs**

- `2 << 3` is `16`.

---

### Real-World Cases

- **Permission updates** — `permissions |= Permissions.Execute`.
- **Hash mixing** — `hash ^= value; hash <<= 5;`.
- **Packing data** — Shifting fields into place.

---

### References

- Compound assignment — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/bitwise-and-shift-operators#compound-assignment

---

## 9. Bit Manipulation (Setting, Clearing, Toggling, Testing Bits)

### Definitions

**Core Definition**
Bit manipulation is the practice of using bitwise operators to set, clear, toggle, or test individual bits within an integral value.

**Technical Definition**
The four fundamental bit operations are: **set** (`value |= mask`), **clear** (`value &= ~mask`), **toggle** (`value ^= mask`), and **test** (`(value & mask) != 0`). Each operation uses a **mask**—a value with `1`s in the bit positions to be manipulated.

**Beginner-Friendly Explanation**
Think of a bit mask as a stencil. You place it over a number to affect only the bits you care about, leaving the others unchanged.

---

### Purposes

- To manage individual flags packed into a single integer.
- To implement permission systems, feature toggles, and state machines.
- To pack and unpack data fields.
- To optimize memory usage by storing multiple Boolean values in one integer.

---

### Syntax Rules and Structure (Operations)

```
// Set a bit
value   |=   mask   ;

// Clear a bit
value   &=   ~mask   ;

// Toggle a bit
value   ^=   mask   ;

// Test a bit
bool   isSet   =   (value   &   mask)   !=   0   ;
```

**Component breakdown**

- `mask` — A value with `1` in the target bit position(s).
- `|=` — Sets the bits.
- `&= ~` — Clears the bits.
- `^=` — Toggles the bits.
- `&` with comparison — Tests the bits.

#### Syntax Rules

- The mask must be of a compatible type.
- The operations modify the target value in place (or produce a new value).

#### Constraints and Limitations

- Be careful with sign bits when working with signed types.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: The Four Operations

```csharp
using System;

class Program
{
    static void Main()
    {
        int value = 0b_1010;
        int mask  = 0b_0010;

        value |= mask;   // set bit 1
        Console.WriteLine(Convert.ToString(value, 2)); // 1010 (already set)

        value &= ~mask;  // clear bit 1
        Console.WriteLine(Convert.ToString(value, 2)); // 1000

        value ^= mask;   // toggle bit 1
        Console.WriteLine(Convert.ToString(value, 2)); // 1010

        bool isSet = (value & mask) != 0;
        Console.WriteLine(isSet); // True
    }
}
```

**Expected Output**

```
1010
1000
1010
True
```

**Why this output occurs**

- Each operation performs its specific bit manipulation on bit 1.

---

### Real-World Cases

- **File attributes** — `FileAttributes.ReadOnly | FileAttributes.Hidden`.
- **Game state** — Tracking multiple status effects in one integer.
- **Network protocols** — Packing flags into header fields.

---

### References

- Bitwise and shift operators — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/bitwise-and-shift-operators

---

## 10. Bit Masks

### Definitions

**Core Definition**
A bit mask is a value used with bitwise operators to selectively manipulate or extract specific bits from another value.

**Technical Definition**
A bit mask is a binary value with `1`s in the bit positions to be affected and `0`s elsewhere. Masks are used with AND (to extract or clear bits), OR (to set bits), XOR (to toggle bits), and complement (to invert masks). The term is also used for the mask itself as a constant or variable.

**Beginner-Friendly Explanation**
A mask is like a **stencil**. When you place it over a value and apply a bitwise operator, only the bits allowed by the stencil are affected.

---

### Purposes

- To isolate specific bits from a packed value.
- To protect other bits while modifying only the target bits.
- To define flag values in enums and constants.
- To create reusable patterns for bit manipulation.

---

### Syntax Rules and Structure

```
int   mask   =   1   <<   bitPosition   ;
int   extracted   =   value   &   mask   ;
```

**Component breakdown**

- `1 << bitPosition` — Creates a mask with only that bit set.
- `value & mask` — Extracts the bit.

#### Syntax Rules

- Masks are ordinary integral values.
- Use hexadecimal or binary literals for readability (`0xFF`, `0b_1010`).
- Combine masks with `|` for multi-bit operations.

#### Constraints and Limitations

- Masks must fit within the target type’s bit width.
- Sign bits require careful handling with signed types.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Creating and Using a Mask

```csharp
using System;

class Program
{
    static void Main()
    {
        const int ReadMask = 1 << 0;   // 1
        const int WriteMask = 1 << 1;  // 2

        int permissions = ReadMask | WriteMask;
        Console.WriteLine(Convert.ToString(permissions, 2)); // 11

        bool canRead = (permissions & ReadMask) != 0;
        Console.WriteLine(canRead); // True
    }
}
```

**Expected Output**

```
11
True
```

**Why this output occurs**

- The masks define individual bits; OR combines them; AND tests them.

---

### Real-World Cases

- **Flag enums** — Defining `[Flags]` members as masks.
- **Pixel manipulation** — Extracting colour channels with masks.
- **Hardware registers** — Reading and writing specific register bits.

---

### References

- Enumeration types as bit flags — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/enum#enumeration-types-as-bit-flags

---

## 11. Bitwise Applications

### Definitions

**Core Definition**
Bitwise applications are the practical uses of bitwise operators in real-world software, including flag management, performance optimization, hardware interfacing, and binary protocol handling.

**Technical Definition**
Bitwise operators are applied wherever data is naturally represented as bits or where bit-level control is required. Common application areas include: permission and flag systems, hash functions, compression and encryption algorithms, hardware register manipulation, binary serialization, and performance optimizations such as replacing multiplication/division by powers of two with shifts.

**Beginner-Friendly Explanation**
Bitwise operators are not just academic curiosities—they are used every day in operating systems, game engines, networking stacks, and databases. They let you pack flags, process binary data, and compute things quickly.

---

### Purposes

- To manage sets of Boolean flags efficiently.
- To implement low-level protocols and hardware interfaces.
- To optimize arithmetic operations in hot paths.
- To support cryptographic and hashing algorithms.
- To enable compact storage of multiple Boolean values.

---

### Syntax Rules and Structure (Application Patterns)

```
// Flag management
[Flags] enum Permissions { Read = 1, Write = 2, Execute = 4 }
Permissions p = Permissions.Read | Permissions.Write;

// Performance: multiply by 8
int result = value << 3;

// Hardware register access
register |= (1 << 5);   // set bit 5
register &= ~(1 << 3);  // clear bit 3

// Hash mixing
hash ^= value;
hash = (hash << 5) | (hash >> 27);
```

**Component breakdown**

- **Flag management** — Using enums with powers of two.
- **Performance** — Replacing `*` and `/` with shifts.
- **Hardware** — Setting and clearing register bits.
- **Hashing** — Mixing bits with XOR and shifts.

#### Syntax Rules

- Ensure masks and shifts use the correct bit widths.
- Understand the difference between arithmetic and logical shifts.

#### Constraints and Limitations

- Bitwise optimizations can reduce readability; document them clearly.
- Endianness matters when interpreting byte order for multi-byte values.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Flag Management

```csharp
using System;

[Flags]
enum FileAccess { None = 0, Read = 1, Write = 2, Execute = 4 }

class Program
{
    static void Main()
    {
        FileAccess access = FileAccess.Read | FileAccess.Write;
        Console.WriteLine(access);                         // Read, Write
        Console.WriteLine((access & FileAccess.Read) != 0); // True
    }
}
```

**Expected Output**

```
Read, Write
True
```

**Why this output occurs**

- Flags are combined with OR and tested with AND.

---

#### Example 2: Fast Multiplication

```csharp
using System;

class Program
{
    static void Main()
    {
        int x = 7;
        Console.WriteLine(x * 8);   // 56
        Console.WriteLine(x << 3);  // 56 (same result, faster)
    }
}
```

**Expected Output**

```
56
56
```

**Why this output occurs**

- Left-shifting by `3` multiplies by `2^3 = 8`.

---

### Real-World Cases

- **Operating systems** — File permissions, process flags.
- **Game engines** — Entity state, collision layers.
- **Networking** — TCP/IP header flags (SYN, ACK, FIN).
- **Compression** — Bit-packing data for smaller storage.

---

### References

- Bitwise and shift operators — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/bitwise-and-shift-operators

---

## 12. `BitConverter`

### Definitions

**Core Definition**
`BitConverter` is a static class in the `System` namespace that converts base data types to arrays of bytes and vice versa.

**Technical Definition**
`System.BitConverter` converts base data types to an array of bytes, and an array of bytes to base data types. The `GetBytes` method converts a primitive value to a byte array; `ToInt32`, `ToDouble`, etc., convert byte arrays back to primitives. `BitConverter.ToString` produces a human-readable hexadecimal string. The byte order depends on the platform’s endianness (`BitConverter.IsLittleEndian`).

**Beginner-Friendly Explanation**
`BitConverter` is a **translator between numbers and raw bytes**. It lets you take an `int` or `double` and turn it into the bytes that represent it, or turn a sequence of bytes back into a number.

---

### Purposes

- To convert primitive values to byte arrays for storage or transmission.
- To reconstruct primitive values from byte arrays.
- To inspect the raw byte representation of values.
- To support binary serialization and file formats.
- To convert between numeric types and hexadecimal strings.

---

### Syntax Rules and Structure

```
byte[]   bytes   =   BitConverter.GetBytes(value)   ;
int      value   =   BitConverter.ToInt32(bytes,   startIndex)   ;
string   hex     =   BitConverter.ToString(bytes)   ;
```

**Component breakdown**

- `GetBytes` — Converts a value to a byte array.
- `ToInt32`, `ToDouble`, etc. — Converts bytes to a primitive.
- `ToString` — Formats bytes as a hyphen-separated hex string.

#### Syntax Rules

- Byte order is platform-dependent (`BitConverter.IsLittleEndian`).
- Conversion methods require sufficient bytes starting at the given index.

#### Constraints and Limitations

- Allocates a new byte array for each `GetBytes` call.
- Not endian-specific; use `BinaryPrimitives` for explicit endianness.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Converting an Integer to Bytes

```csharp
using System;

class Program
{
    static void Main()
    {
        int value = 123456789;
        byte[] bytes = BitConverter.GetBytes(value);

        Console.WriteLine(BitConverter.ToString(bytes)); // 15-CD-5B-07 (little-endian)
        Console.WriteLine(BitConverter.ToInt32(bytes, 0)); // 123456789
    }
}
```

**Expected Output**

```
15-CD-5B-07
123456789
```

**Why this output occurs**

- The integer is stored as four bytes in little-endian order on most platforms; converting back yields the original value.

---

### Real-World Cases

- **File formats** — Writing binary headers and records.
- **Network protocols** — Converting numeric fields to bytes.
- **Debugging** — Inspecting the raw representation of values.

---

### References

- BitConverter Class (System) — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.bitconverter

---

## 13. `BinaryPrimitives`

### Definitions

**Core Definition**
`BinaryPrimitives` is a static class in `System.Buffers.Binary` that reads and writes primitive values from byte spans with explicit endianness.

**Technical Definition**
`System.Buffers.Binary.BinaryPrimitives` reads bytes as primitives with specific endianness. It provides methods such as `ReadInt32LittleEndian`, `ReadInt32BigEndian`, `WriteInt32LittleEndian`, and `ReverseEndianness`. These methods operate on `ReadOnlySpan<byte>` and `Span<byte>`, avoiding allocations and giving precise control over byte order.

**Beginner-Friendly Explanation**
`BinaryPrimitives` is like `BitConverter`, but it lets you **choose the byte order** (little-endian or big-endian) explicitly, and it is more efficient because it works directly with spans instead of allocating arrays.

---

### Purposes

- To read and write primitives with explicit endianness.
- To parse binary protocols that specify byte order.
- To avoid allocations by working with spans.
- To reverse the endianness of values.
- To support high-performance binary serialization.

---

### Syntax Rules and Structure

```
int   value   =   BinaryPrimitives.ReadInt32LittleEndian(span)   ;
BinaryPrimitives.WriteInt32LittleEndian(span,   value)   ;
```

**Component breakdown**

- `ReadInt32LittleEndian` — Reads a 32-bit integer from a span in little-endian order.
- `WriteInt32LittleEndian` — Writes a 32-bit integer to a span.
- Similar methods exist for `Int16`, `Int64`, `Double`, etc., and for big-endian variants.

#### Syntax Rules

- Operates on `ReadOnlySpan<byte>` (reads) or `Span<byte>` (writes).
- Throws `ArgumentOutOfRangeException` if the span is too short.

#### Constraints and Limitations

- Requires .NET Core 2.1 / .NET Standard 2.1 or later.
- Does not allocate; efficient for high-performance scenarios.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Reading Little-Endian

```csharp
using System;
using System.Buffers.Binary;

class Program
{
    static void Main()
    {
        byte[] buffer = { 0x15, 0xCD, 0x5B, 0x07 };
        int value = BinaryPrimitives.ReadInt32LittleEndian(buffer);
        Console.WriteLine(value);   // 123456789
    }
}
```

**Expected Output**

```
123456789
```

**Why this output occurs**

- The bytes are interpreted as a little-endian 32-bit integer.

---

#### Example 2: Writing and Reversing Endianness

```csharp
using System;
using System.Buffers.Binary;

class Program
{
    static void Main()
    {
        byte[] buffer = new byte[4];
        BinaryPrimitives.WriteInt32BigEndian(buffer, 123456789);
        Console.WriteLine(BitConverter.ToString(buffer)); // 07-5B-CD-15

        uint reversed = BinaryPrimitives.ReverseEndianness(0x12345678);
        Console.WriteLine($"0x{reversed:X8}"); // 0x78563412
    }
}
```

**Expected Output**

```
07-5B-CD-15
0x78563412
```

**Why this output occurs**

- `WriteInt32BigEndian` writes the most significant byte first.
- `ReverseEndianness` swaps the byte order.

---

### Real-World Cases

- **Network protocols** — Reading big-endian headers (many protocols use network byte order).
- **Binary file formats** — Parsing formats with specified endianness.
- **High-performance serialization** — Avoiding allocations with spans.

---

### References

- BinaryPrimitives Class (System.Buffers.Binary) — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.buffers.binary.binaryprimitives

---

## 14. Operator Precedence and Associativity

### Definitions

**Core Definition**
Precedence determines which bitwise operator is evaluated first in an expression with multiple operators; associativity determines the order when operators have the same precedence.

**Technical Definition**
The bitwise and shift operators have the following precedence order from highest to lowest: unary bitwise complement (`~`), then shift operators (`<<`, `>>`, `>>>`), then bitwise AND (`&`), then bitwise XOR (`^`), then bitwise OR (`|`). All binary bitwise operators are **left-associative**. Parentheses can override both precedence and associativity.

**Beginner-Friendly Explanation**
Precedence is the “who goes first” rule. `~` binds tightest, then shifts, then `&`, then `^`, then `|`. So `a | b & c` means `a | (b & c)`. Use parentheses if you want a different order.

---

### Purposes

- To define a consistent evaluation order.
- To allow concise expressions without excessive parentheses.
- To avoid ambiguity in compound conditions.
- To guide the use of parentheses for readability.

---

### Syntax Rules and Structure (Precedence Order)

```
Highest:  ~
          << >>
          &
          ^
Lowest:   |
```

**Component breakdown**

- `~` — Highest precedence.
- `<<`, `>>`, `>>>` — Next.
- `&` — Higher than `^` and `|`.
- `^` — Higher than `|`.
- `|` — Lowest among bitwise operators.

#### Syntax Rules

- Higher-precedence operators bind tighter.
- Binary operators are left-associative.
- Parentheses override precedence.

#### Constraints and Limitations

- Relying on precedence without parentheses can reduce readability.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Precedence of `&` vs. `|`

```csharp
using System;

class Program
{
    static void Main()
    {
        int a = 0b_1100;
        int b = 0b_1010;
        int c = 0b_0110;

        Console.WriteLine(Convert.ToString(a | b & c, 2));       // 1110
        Console.WriteLine(Convert.ToString((a | b) & c, 2));     // 0110
    }
}
```

**Expected Output**

```
1110
0110
```

**Why this output occurs**

- `&` has higher precedence than `|`, so `b & c` is evaluated first; parentheses change the grouping.

---

### References

- Operator precedence — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/#operator-precedence

---

## 15. Best Practices

### Definitions

**Core Definition**
Bitwise best practices are the guidelines that ensure bitwise code is correct, readable, safe, and maintainable.

**Technical Definition**
Best practices include using named constants or `[Flags]` enums instead of magic numbers, choosing binary or hexadecimal literals for clarity, understanding arithmetic vs. logical right shifts, preferring `>>>` over casting for logical shifts on signed types, using `BinaryPrimitives` for endian-specific work, and documenting non-obvious bit tricks.

**Beginner-Friendly Explanation**
Best practices are the habits that keep bitwise code from becoming cryptic: give your masks names, use readable literals, and choose the right shift operator for the job.

---

### Purposes

- To prevent off-by-one and sign-related bugs.
- To make bitwise code understandable to future maintainers.
- To choose the most efficient and correct operator.
- To avoid platform-dependent endianness surprises.
- To leverage modern, safe APIs.

---

### Guidelines and Structure

#### General Guidelines

```
// Use named constants or [Flags] enums
const int ReadMask = 1 << 0;
[Flags] enum Permissions { Read = 1, Write = 2 }

// Use binary literals for clarity
int mask = 0b_0000_1111;

// Prefer >>> over casting for logical shifts on signed types
int logical = signed >>> 1;   // not (int)((uint)signed >> 1)

// Use BinaryPrimitives for endian-specific work
int value = BinaryPrimitives.ReadInt32BigEndian(span);

// Document non-obvious bit tricks
// Extract high nibble: (packed >> 4) & 0xF
```

**Component breakdown**

- **Named constants** — Replace magic numbers with meaningful names.
- **Binary literals** — Show the bit pattern directly.
- **`>>>` for logical shifts** — Avoids unsafe casts.
- **`BinaryPrimitives`** — Explicit endianness, allocation-free.
- **Documentation** — Explain non-obvious operations.

#### Rules and Constraints

- Do not hard-code bit positions without explanation.
- Do not use `>>` for logical shifts on negative signed numbers; use `>>>`.
- Do not assume endianness; use `BinaryPrimitives` or check `BitConverter.IsLittleEndian`.
- Avoid obscure bit hacks when readable code suffices.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Readable Flag Code

```csharp
using System;

[Flags]
enum Permissions { None = 0, Read = 1, Write = 2, Execute = 4 }

class Program
{
    static void Main()
    {
        Permissions p = Permissions.Read | Permissions.Write;
        p |= Permissions.Execute;
        p &= ~Permissions.Write;

        Console.WriteLine(p); // Read, Execute
    }
}
```

**Expected Output**

```
Read, Execute
```

**Why this output occurs**

- The `[Flags]` enum makes the bit meanings clear, and the operations are readable.

---

### Real-World Cases

- **Code reviews** — Bitwise code should be clear enough for review.
- **Library APIs** — Use `[Flags]` enums for public option sets.
- **Performance-critical code** — Document why a bit trick is used.

---

### References

- Bitwise and shift operators — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/bitwise-and-shift-operators
- Enumeration types as bit flags — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/enum#enumeration-types-as-bit-flags

---

## Deprecated, Unsafe, Implementation-Dependent, and Version-Specific Notes

| Feature | Status / Note |
|---------|---------------|
| `>>>` (unsigned right shift) | Introduced in **C# 11**; version-specific. |
| `>>` on signed types | **Arithmetic** shift (sign-propagating); language-defined. |
| `&` / `|` on `bool` | **Non-short-circuiting**; both operands always evaluated. |
| Endianness of `BitConverter` | **Platform-dependent**; check `BitConverter.IsLittleEndian`. |
| `BinaryPrimitives` | Available since **.NET Core 2.1** / .NET Standard 2.1. |
| Shift count masking | **Language-defined**: low 5 bits for 32-bit, low 6 bits for 64-bit. |
| Bitwise operations and overflow | **Never cause overflow**; behave identically in `checked`/`unchecked`. |
| User-defined compound assignment | **C# 14** allows explicit overloads; earlier versions use implicit overloads. |

---

## References (Consolidated)

- Bitwise and shift operators — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/bitwise-and-shift-operators
- BitConverter Class (System) — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.bitconverter
- BinaryPrimitives Class (System.Buffers.Binary) — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.buffers.binary.binaryprimitives
- Operator precedence — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/#operator-precedence
- Enumeration types as bit flags — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/enum#enumeration-types-as-bit-flags
- C# 11 unsigned right shift operator proposal - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/proposals/csharp-11.0/unsigned-right-shift-operator
- Bitwise and shift operators — C# language specification - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/language-specification/expressions#129-bitwise-operators

---