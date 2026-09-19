# Python Bitwise Operators: A Comprehensive Programming Cheat Sheet

## Topic Overview

### Definitions

**Core Definition**
Bitwise operators are symbols that perform operations on the individual bits (binary digits) of integer values, manipulating data at the most granular level.

**Technical Definition**
Python's bitwise operators (`&`, `|`, `^`, `~`, `<<`, `>>`) are part of the language's expression grammar and are defined for integer types. They operate on the two's complement representation of integers, treating negative numbers as if they had an infinite number of leading sign bits. The operators are implemented via the special methods `__and__()`, `__or__()`, `__xor__()`, `__invert__()`, `__lshift__()`, and `__rshift__()`. Python integers have arbitrary precision, so bitwise operations never overflow; instead, they produce results with as many bits as needed.

**Beginner-Friendly Explanation**
Bitwise operators work on the binary (base-2) representation of numbers. Instead of adding or subtracting whole numbers, they manipulate individual bits — the 0s and 1s that computers use internally. For example, `5 & 3` compares the binary forms of 5 (`101`) and 3 (`011`) bit by bit, producing `001` (which is 1). These operators are useful for low-level programming, working with flags, and optimising certain algorithms.

### Key Characteristics

| Characteristic | Description |
|---|---|
| **Bit-Level Operation** | Each operator works on individual bits of integers |
| **Two's Complement** | Negative numbers are treated as their two's complement representation |
| **Arbitrary Precision** | Python integers have unlimited bits; no overflow occurs |
| **Bitwise AND (`&`)** | Sets a bit to 1 only if both corresponding bits are 1 |
| **Bitwise OR (`\|`)** | Sets a bit to 1 if at least one corresponding bit is 1 |
| **Bitwise XOR (`^`)** | Sets a bit to 1 if exactly one corresponding bit is 1 |
| **Bitwise NOT (`~`)** | Inverts all bits; `~x` is equivalent to `-(x+1)` |
| **Left Shift (`<<`)** | Shifts bits left, filling with zeros |
| **Right Shift (`>>`)** | Shifts bits right, preserving the sign bit |

### Prerequisites

- **Basic Python syntax**: variables, expressions, and `print()`
- **Integer types**: understanding of `int` and how numbers are represented
- **Binary number system**: familiarity with base-2 representation
- **Two's complement**: how negative integers are stored
- **Logical operators**: `and`, `or`, `not` for comparison (though bitwise operators are different)

### Related Programming Areas

- **Bit Manipulation**: Setting, clearing, and toggling individual bits
- **Bitmasks**: Using masks to extract or modify specific bits
- **Flags**: Encoding multiple Boolean options in a single integer
- **Low-Level Programming**: Working with hardware, protocols, and binary formats
- **Cryptography**: XOR-based encryption and hash functions
- **Compression**: Bit-level data packing and encoding
- **Error Detection**: Checksums and parity bits
- **Computer Graphics**: Colour manipulation and pixel operations

### Core Concepts / Features

1. **`&` (Bitwise AND)**
2. **`|` (Bitwise OR)**
3. **`^` (Bitwise XOR)**
4. **`~` (Bitwise NOT)**
5. **`<<` (Bitwise Zero Fill Left Shift)**
6. **`>>` (Bitwise Signed Right Shift)**
7. **Bitmasks and Common Patterns**
8. **Two's Complement and Negative Numbers**
9. **Operator Precedence**
10. **Common Pitfalls and Edge Cases**


## Core Concept 1: `&` (Bitwise AND)

### Definitions

**Core Definition**
The `&` operator performs a bitwise AND, setting each result bit to 1 only if both corresponding bits in the operands are 1.

**Technical Definition**
The expression `a & b` evaluates each bit position: if both `a` and `b` have a 1 at that position, the result has a 1; otherwise, the result has a 0. For negative numbers, the two's complement representation is used, with an infinite number of sign bits extended to the left. The operator is implemented by `__and__()` and `__rand__()`.

**Beginner-Friendly Explanation**
`&` is like asking "are BOTH bits 1?" For example, `5 & 3` compares `101` and `011` bit by bit: the first bit (1 and 0) gives 0, the second (0 and 1) gives 0, the third (1 and 1) gives 1. Result: `001` = 1.

### Purposes

- **To mask bits** — extract specific bits from a number
- **To check if a bit is set** (e.g., `if flags & FLAG:`)
- **To clear bits** (e.g., `flags &= ~FLAG`)
- **To test multiple conditions** represented as bit flags
- **To implement low-level hardware control**

### Syntax Rules and Structure

**Complete General Syntax**

```
operand1 & operand2
```

**Breakdown:**
- `operand1`, `operand2`: Integer expressions.
- Returns an integer with bits set where both operands have 1s.

**Syntax Rules**

| Rule | Description |
|---|---|
| Integer operands | Both operands must be integers |
| Bit-by-bit | Each bit position evaluated independently |
| Two's complement | Negative numbers use two's complement |
| Augmented form | `x &= y` is equivalent to `x = x & y` |
| Precedence | Lower than comparison operators but higher than `\|` |

**Constraints and Limitations**

- **Cannot mix with floats**: `5.0 & 3` raises `TypeError`.
- **Bitwise vs. logical**: `&` is not the same as `and` (which operates on truthiness).
- **Infinite sign extension**: Negative numbers behave as if they have infinite leading 1s.

### Multiple Annotated Complete Code Examples and Their Expected Outputs

**Example 1: Basic Bitwise AND**

```python
# Step 1: 5 = 0101, 3 = 0011
result = 5 & 3
# 0101 & 0011 = 0001 = 1
print(f"5 & 3 = {result}")

# Step 2: Another example
result2 = 12 & 10
# 12 = 1100, 10 = 1010, 1100 & 1010 = 1000 = 8
print(f"12 & 10 = {result2}")
```

**Expected Output:**
```
5 & 3 = 1
12 & 10 = 8
```

**Breakdown:** Each bit is compared: 1 only if both bits are 1.

**Example 2: Bit Masking (Extracting Bits)**

```python
# Step 1: Extract the lower 4 bits
value = 0b11011010  # 218 in decimal
mask = 0b00001111   # 15 in decimal
lower = value & mask
print(f"Lower 4 bits: {lower:04b} (decimal {lower})")

# Step 2: Check if a specific bit is set
flags = 0b1010
BIT_2 = 0b0100  # 4
if flags & BIT_2:
    print("Bit 2 is set")
else:
    print("Bit 2 is not set")
```

**Expected Output:**
```
Lower 4 bits: 1010 (decimal 10)
Bit 2 is not set
```

**Breakdown:** `&` with a mask extracts specific bits. Checking `flags & BIT_2` tests whether that bit is set.

**Example 3: Clearing Bits**

```python
# Step 1: Clear bit 1 (second bit)
flags = 0b1111  # 15
BIT_1 = 0b0010   # 2
flags &= ~BIT_1  # Clear bit 1
print(f"After clearing bit 1: {flags:04b} (decimal {flags})")
```

**Expected Output:**
```
After clearing bit 1: 1101 (decimal 13)
```

**Breakdown:** `~BIT_1` inverts the mask, and `&=` clears the specified bit.

### Real-World Cases with Explanation

**Case 1: Permission Flags**

```python
READ = 0b001
WRITE = 0b010
EXECUTE = 0b100

permissions = READ | WRITE  # 0b011

# Check if WRITE is set
if permissions & WRITE:
    print("Write access granted")
```

**Why it matters:** Bit flags efficiently pack multiple Boolean permissions into a single integer.

**Case 2: Network Subnet Masking**

```python
ip = 0xC0A80101       # 192.168.1.1
subnet_mask = 0xFFFFFF00  # 255.255.255.0
network = ip & subnet_mask
print(f"Network: {network:#010x}")  # 0xc0a80100
```

**Why it matters:** Bitwise AND is fundamental for IP address masking in networking.


## Core Concept 2: `|` (Bitwise OR)

### Definitions

**Core Definition**
The `|` operator performs a bitwise OR, setting each result bit to 1 if at least one corresponding bit in the operands is 1.

**Technical Definition**
The expression `a | b` evaluates each bit position: if either `a` or `b` has a 1 at that position, the result has a 1. The operator is implemented by `__or__()` and `__ror__()`.

**Beginner-Friendly Explanation**
`|` is like asking "is EITHER bit 1?" For example, `5 | 3` compares `101` and `011`: `101 | 011 = 111` = 7.

### Purposes

- **To set bits** — turn specific bits on
- **To combine flags** (e.g., `READ | WRITE`)
- **To ensure a bit is 1** regardless of its previous state
- **To implement bitwise addition** (without carry)

### Syntax Rules and Structure

**Complete General Syntax**

```
operand1 | operand2
```

**Syntax Rules**

| Rule | Description |
|---|---|
| Integer operands | Both operands must be integers |
| Bit-by-bit | 1 if either bit is 1 |
| Augmented form | `x \|= y` |
| Precedence | Lower than `^` and `&`, higher than `and` |

**Constraints and Limitations**

- **Cannot mix with floats**: Raises `TypeError`.
- **Not the same as `or`**: `|` is bitwise; `or` is logical.

### Multiple Annotated Complete Code Examples and Their Expected Outputs

**Example 1: Basic Bitwise OR**

```python
# Step 1: 5 = 0101, 3 = 0011
print(f"5 | 3 = {5 | 3}")  # 0111 = 7

# Step 2: 12 = 1100, 10 = 1010
print(f"12 | 10 = {12 | 10}")  # 1110 = 14
```

**Expected Output:**
```
5 | 3 = 7
12 | 10 = 14
```

**Breakdown:** Each bit is 1 if either operand has a 1 at that position.

**Example 2: Setting Flags**

```python
# Step 1: Combine permissions
READ = 0b001
WRITE = 0b010
EXECUTE = 0b100

permissions = READ | WRITE  # 0b011
print(f"Permissions: {permissions:03b}")

# Step 2: Add execute permission
permissions |= EXECUTE
print(f"With execute: {permissions:03b}")
```

**Expected Output:**
```
Permissions: 011
With execute: 111
```

**Breakdown:** `|` combines flags, and `|=` adds a flag without affecting others.

### Real-World Cases with Explanation

**Case 1: File Permissions**

```python
OWNER_READ = 0o400
OWNER_WRITE = 0o200
OWNER_EXEC = 0o100

permissions = OWNER_READ | OWNER_WRITE | OWNER_EXEC
print(f"Permissions: {permissions:o}")  # 700
```

**Why it matters:** Unix file permissions are commonly represented as bit flags.

**Case 2: Colour Channels**

```python
RED   = 0xFF0000
GREEN = 0x00FF00
BLUE  = 0x0000FF

colour = RED | BLUE  # Purple
print(f"Colour: {colour:#08x}")
```

**Why it matters:** Bitwise OR combines colour channels in RGB representations.


## Core Concept 3: `^` (Bitwise XOR)

### Definitions

**Core Definition**
The `^` operator performs a bitwise XOR (exclusive OR), setting each result bit to 1 if exactly one of the corresponding bits in the operands is 1.

**Technical Definition**
The expression `a ^ b` evaluates each bit position: if exactly one of `a` or `b` has a 1 at that position, the result has a 1. The operator is implemented by `__xor__()` and `__rxor__()`.

**Beginner-Friendly Explanation**
`^` is like asking "are the bits DIFFERENT?" If they're different (one is 1, the other 0), the result is 1. If they're the same (both 0 or both 1), the result is 0. For example, `5 ^ 3` compares `101` and `011`: `101 ^ 011 = 110` = 6.

### Purposes

- **To toggle bits** — flip bits from 0 to 1 and vice versa
- **To swap values without a temporary variable**
- **To implement simple encryption** (XOR cipher)
- **To detect differences** between two values
- **To compute checksums** and parity bits

### Syntax Rules and Structure

**Complete General Syntax**

```
operand1 ^ operand2
```

**Syntax Rules**

| Rule | Description |
|---|---|
| Integer operands | Both operands must be integers |
| Exactly one | 1 if exactly one bit is 1 |
| Augmented form | `x ^= y` |
| Precedence | Lower than `&`, higher than `\|` |

**Constraints and Limitations**

- **Cannot mix with floats**: Raises `TypeError`.
- **Not the same as `!=`**: `^` is bitwise; `!=` is value inequality.

### Multiple Annotated Complete Code Examples and Their Expected Outputs

**Example 1: Basic XOR**

```python
print(f"5 ^ 3 = {5 ^ 3}")    # 101 ^ 011 = 110 = 6
print(f"12 ^ 10 = {12 ^ 10}") # 1100 ^ 1010 = 0110 = 6
```

**Expected Output:**
```
5 ^ 3 = 6
12 ^ 10 = 6
```

**Breakdown:** Each bit is 1 if exactly one operand has a 1 at that position.

**Example 2: Toggling Bits**

```python
# Step 1: Toggle a specific bit
value = 0b1010
TOGGLE = 0b0100
result = value ^ TOGGLE
print(f"Toggled: {result:04b}")  # 1110

# Step 2: Toggle back
result = result ^ TOGGLE
print(f"Toggled back: {result:04b}")  # 1010
```

**Expected Output:**
```
Toggled: 1110
Toggled back: 1010
```

**Breakdown:** XOR with a mask toggles the bits where the mask has 1s. Applying XOR twice returns the original value.

**Example 3: Swapping Without Temporary Variable**

```python
a, b = 5, 3

a = a ^ b  # a = 5 ^ 3 = 6
b = a ^ b  # b = 6 ^ 3 = 5
a = a ^ b  # a = 6 ^ 5 = 3

print(f"a = {a}, b = {b}")  # a = 3, b = 5
```

**Expected Output:**
```
a = 3, b = 5
```

**Breakdown:** The XOR swap algorithm exchanges two values without a temporary variable.

### Real-World Cases with Explanation

**Case 1: XOR Cipher**

```python
def xor_cipher(text, key):
    return ''.join(chr(ord(c) ^ key) for c in text)

encrypted = xor_cipher("hello", 42)
decrypted = xor_cipher(encrypted, 42)
print(f"Encrypted: {encrypted}")
print(f"Decrypted: {decrypted}")
```

**Why it matters:** XOR is a fundamental operation in many encryption algorithms.

**Case 2: Finding the Missing Number**

```python
def find_missing(nums):
    result = 0
    for i in range(1, len(nums) + 2):
        result ^= i
    for num in nums:
        result ^= num
    return result

print(find_missing([1, 2, 4, 5]))  # 3
```

**Why it matters:** XOR properties enable finding a missing number in O(n) time without extra memory.


## Core Concept 4: `~` (Bitwise NOT)

### Definitions

**Core Definition**
The `~` operator performs a bitwise NOT (inversion), flipping all bits of its operand; in Python, `~x` is mathematically equivalent to `-(x + 1)`.

**Technical Definition**
The bitwise inversion of `x` is defined as `-(x+1)`. This is simply how the bitwise inversion of the two's complement representation of an integer works. Python represents negative integers in unbounded two's-complement form, maintaining the illusion of infinite sign bits. The `~` operator is implemented by `__invert__()`.

**Beginner-Friendly Explanation**
`~` flips every bit: 0 becomes 1, and 1 becomes 0. For example, `~5` is `-6`. Why? Because Python uses two's complement, and inverting the bits of a positive number gives a negative number. The formula `~x = -(x+1)` makes it easy to remember: `~5 = -(5+1) = -6`.

### Purposes

- **To invert all bits** of an integer
- **To create a mask** for clearing bits (e.g., `flags &= ~MASK`)
- **To compute the two's complement** of a number
- **To implement bitwise negation** in algorithms

### Syntax Rules and Structure

**Complete General Syntax**

```
~operand
```

**Syntax Rules**

| Rule | Description |
|---|---|
| Unary | Takes one operand |
| Integer only | Operand must be an integer |
| Arithmetic result | `~x` equals `-(x+1)` |
| No augmented form | `~=` is not a valid operator |

**Constraints and Limitations**

- **Unbounded two's complement**: The result depends on Python's infinite-precision representation.
- **Not the same as logical `not`**: `~` is bitwise; `not` is logical.

### Multiple Annotated Complete Code Examples and Their Expected Outputs

**Example 1: Basic NOT**

```python
print(f"~5 = {~5}")      # -6
print(f"~0 = {~0}")      # -1
print(f"~-1 = {~-1}")    # 0
print(f"~-6 = {~-6}")    # 5
```

**Expected Output:**
```
~5 = -6
~0 = -1
~-1 = 0
~-6 = 5
```

**Breakdown:** `~x` equals `-(x+1)`. This is the arithmetic form of bitwise inversion in two's complement.

**Example 2: Clearing Bits with NOT**

```python
# Step 1: Clear bit 2 using NOT and AND
flags = 0b1111  # 15
BIT_2 = 0b0100   # 4
flags &= ~BIT_2
print(f"After clearing bit 2: {flags:04b} (decimal {flags})")
```

**Expected Output:**
```
After clearing bit 2: 1011 (decimal 11)
```

**Breakdown:** `~BIT_2` creates a mask with all bits set except bit 2, and `&=` clears that bit.

**Example 3: Two's Complement**

```python
# Step 1: Two's complement of 18
value = 18
complement = ~value + 1
print(f"Two's complement of 18: {complement}")  # -18
```

**Expected Output:**
```
Two's complement of 18: -18
```

**Breakdown:** The two's complement of a number is its negation. `~x + 1` computes `-x`.

### Real-World Cases with Explanation

**Case 1: Creating Masks**

```python
# Create a mask for the lower 8 bits
ALL_BITS = 0xFFFFFFFF
LOWER_8 = ALL_BITS & ~(0xFFFFFF00)
# Or simply: LOWER_8 = 0xFF
```

**Why it matters:** `~` is used to create masks that clear specific bits.

**Case 2: Bitwise Negation in Algorithms**

```python
def bitwise_negate(x):
    return ~x + 1  # Equivalent to -x
```

**Why it matters:** Understanding `~` and two's complement is essential for low-level bit manipulation.


## Core Concept 5: `<<` (Bitwise Zero Fill Left Shift)

### Definitions

**Core Definition**
The `<<` operator shifts the bits of its left operand to the left by the number of positions specified by its right operand, filling the vacated rightmost bits with zeros.

**Technical Definition**
The expression `a << n` shifts the bits of `a` left by `n` positions. Each left shift effectively multiplies the number by 2. For positive integers, this is equivalent to `a * (2 ** n)`. The leftmost bits that are shifted beyond the representation's capacity are discarded (though Python's arbitrary precision means there is no fixed capacity). The operator is implemented by `__lshift__()`.

**Beginner-Friendly Explanation**
`<<` moves all the bits to the left. It's like adding zeros on the right side. `5 << 1` takes `101` and makes it `1010` (which is 10). Each shift left doubles the number.

### Purposes

- **To multiply by powers of 2** efficiently
- **To pack data** into a single integer
- **To create bitmasks** at specific positions
- **To implement binary protocols** and data formats
- **To perform fast arithmetic** in performance-critical code

### Syntax Rules and Structure

**Complete General Syntax**

```
operand << shift_amount
```

**Syntax Rules**

| Rule | Description |
|---|---|
| Integer operands | Both must be integers |
| Zeros filled | Right side filled with zeros |
| Arithmetic effect | `a << n` is equivalent to `a * (2 ** n)` |
| Augmented form | `x <<= n` |
| Shift by 0 | Returns the original value |

**Constraints and Limitations**

- **Negative shift counts**: Raise `ValueError`.
- **Large shifts**: Can produce very large integers (memory-intensive).
- **No overflow**: Python integers grow as needed.

### Multiple Annotated Complete Code Examples and Their Expected Outputs

**Example 1: Basic Left Shift**

```python
print(f"5 << 1 = {5 << 1}")   # 101 -> 1010 = 10
print(f"5 << 2 = {5 << 2}")   # 101 -> 10100 = 20
print(f"3 << 3 = {3 << 3}")   # 11 -> 11000 = 24
```

**Expected Output:**
```
5 << 1 = 10
5 << 2 = 20
3 << 3 = 24
```

**Breakdown:** Each left shift multiplies by 2.

**Example 2: Packing Data**

```python
# Step 1: Pack RGB values into a single integer
r, g, b = 255, 128, 64
packed = (r << 16) | (g << 8) | b
print(f"Packed: {packed:#08x}")  # 0xff8040

# Step 2: Unpack
r2 = (packed >> 16) & 0xFF
g2 = (packed >> 8) & 0xFF
b2 = packed & 0xFF
print(f"Unpacked: r={r2}, g={g2}, b={b2}")
```

**Expected Output:**
```
Packed: 0xff8040
Unpacked: r=255, g=128, b=64
```

**Breakdown:** Shifting and OR-ing packs multiple values into one integer; shifting and AND-ing unpacks them.

### Real-World Cases with Explanation

**Case 1: Fast Multiplication**

```python
x = 7
result = x << 3  # x * 8
print(result)  # 56
```

**Why it matters:** Left shift is faster than multiplication for powers of 2 in performance-critical code.

**Case 2: Bitmask Creation**

```python
# Create a bitmask with bit 5 set
mask = 1 << 5  # 0b100000 = 32
```

**Why it matters:** `1 << n` creates a mask with only bit `n` set.


## Core Concept 6: `>>` (Bitwise Signed Right Shift)

### Definitions

**Core Definition**
The `>>` operator shifts the bits of its left operand to the right by the number of positions specified by its right operand, preserving the sign bit (sign-extending).

**Technical Definition**
The expression `a >> n` shifts the bits of `a` right by `n` positions. For non-negative integers, this is equivalent to `a // (2 ** n)` (integer division by a power of 2). For negative integers, the sign bit is extended: the vacated leftmost bits are filled with copies of the sign bit. This preserves the sign of the number. The operator is implemented by `__rshift__()`.

**Beginner-Friendly Explanation**
`>>` moves all the bits to the right. `5 >> 1` takes `101` and makes it `10` (which is 2). Each shift right roughly halves the number, discarding the remainder. Negative numbers stay negative because the sign bit is copied into the new leftmost positions.

### Purposes

- **To divide by powers of 2** efficiently
- **To extract packed data** from a single integer
- **To implement binary protocols** and data formats
- **To perform fast arithmetic** in performance-critical code
- **To unpack bit fields** from encoded values

### Syntax Rules and Structure

**Complete General Syntax**

```
operand >> shift_amount
```

**Syntax Rules**

| Rule | Description |
|---|---|
| Integer operands | Both must be integers |
| Sign-extending | For negative numbers, sign bit is copied left |
| Arithmetic effect | `a >> n` is equivalent to `a // (2 ** n)` for non-negative `a` |
| Augmented form | `x >>= n` |
| Shift by 0 | Returns the original value |

**Constraints and Limitations**

- **Negative shift counts**: Raise `ValueError`.
- **Sign extension**: Negative numbers behave differently than unsigned right shift in other languages.
- **No unsigned right shift**: Python does not have a `>>>` operator.

### Multiple Annotated Complete Code Examples and Their Expected Outputs

**Example 1: Basic Right Shift**

```python
print(f"10 >> 1 = {10 >> 1}")   # 1010 -> 101 = 5
print(f"10 >> 2 = {10 >> 2}")   # 1010 -> 10 = 2
print(f"20 >> 2 = {20 >> 2}")   # 10100 -> 101 = 5
```

**Expected Output:**
```
10 >> 1 = 5
10 >> 2 = 2
20 >> 2 = 5
```

**Breakdown:** Each right shift divides by 2, discarding the remainder.

**Example 2: Sign Extension with Negative Numbers**

```python
print(f"-10 >> 1 = {-10 >> 1}")  # -5
print(f"-10 >> 2 = {-10 >> 2}")  # -3
print(f"-1 >> 1 = {-1 >> 1}")    # -1 (infinite sign extension)
```

**Expected Output:**
```
-10 >> 1 = -5
-10 >> 2 = -3
-1 >> 1 = -1
```

**Breakdown:** Negative numbers preserve their sign through right shift because the sign bit is extended.

**Example 3: Unpacking Data**

```python
# Step 1: Unpack RGB from a packed integer
packed = 0xFF8040
r = (packed >> 16) & 0xFF
g = (packed >> 8) & 0xFF
b = packed & 0xFF
print(f"r={r}, g={g}, b={b}")
```

**Expected Output:**
```
r=255, g=128, b=64
```

**Breakdown:** Right shift moves the desired bits into the lower positions, and `& 0xFF` masks them.

### Real-World Cases with Explanation

**Case 1: Fast Division**

```python
x = 100
result = x >> 3  # x // 8
print(result)  # 12
```

**Why it matters:** Right shift is faster than division for powers of 2 in performance-critical code.

**Case 2: Extracting Bit Fields**

```python
# Extract bits 4-7 from a value
value = 0b10110100
field = (value >> 4) & 0b1111
print(f"Field: {field:04b}")  # 1011
```

**Why it matters:** Right shift and masking extract specific bit fields from packed data.


## Core Concept 7: Bitmasks and Common Patterns

### Definitions

**Core Definition**
A bitmask is an integer used to selectively manipulate specific bits of another integer through bitwise operations.

**Technical Definition**
Bitmasks exploit the fact that `1 & x` preserves the bit `x`, while `0 & x` clears it. Common operations include: checking if a bit is set (`x & mask`), setting a bit (`x | mask`), clearing a bit (`x & ~mask`), and toggling a bit (`x ^ mask`). Bitmasks are fundamental to low-level programming, flag management, and data packing.

**Beginner-Friendly Explanation**
A bitmask is like a stencil: you place it over a number, and only the bits you care about show through. You can use masks to check, set, clear, or flip specific bits.

### Purposes

- **To pack multiple Boolean flags** into a single integer
- **To extract specific bits** from a value
- **To modify individual bits** without affecting others
- **To implement efficient data structures** like bit arrays
- **To control hardware registers** in embedded systems

### Common Bitmask Patterns

| Operation | Pattern | Example |
|---|---|---|
| Check if bit `n` is set | `x & (1 << n)` | `if x & (1 << 3):` |
| Set bit `n` | `x \|= (1 << n)` | `x \|= (1 << 3)` |
| Clear bit `n` | `x &= ~(1 << n)` | `x &= ~(1 << 3)` |
| Toggle bit `n` | `x ^= (1 << n)` | `x ^= (1 << 3)` |
| Check if even | `x & 1 == 0` | `if not (x & 1):` |
| Check if odd | `x & 1 == 1` | `if x & 1:` |
| Multiply by 2^n | `x << n` | `x << 3` = `x * 8` |
| Divide by 2^n | `x >> n` | `x >> 3` = `x // 8` |

### Multiple Annotated Complete Code Examples and Their Expected Outputs

**Example 1: Flag Management**

```python
# Step 1: Define flags
READ = 1 << 0   # 0b001
WRITE = 1 << 1  # 0b010
EXECUTE = 1 << 2  # 0b100

# Step 2: Combine flags
permissions = READ | WRITE
print(f"Permissions: {permissions:03b}")

# Step 3: Check a flag
if permissions & READ:
    print("Read permission granted")

# Step 4: Add a flag
permissions |= EXECUTE
print(f"With execute: {permissions:03b}")

# Step 5: Remove a flag
permissions &= ~WRITE
print(f"Without write: {permissions:03b}")
```

**Expected Output:**
```
Permissions: 011
Read permission granted
With execute: 111
Without write: 101
```

**Breakdown:** Bit flags pack multiple options into one integer, with efficient check/set/clear operations.

**Example 2: Even/Odd Check**

```python
for num in range(5):
    if num & 1:
        print(f"{num} is odd")
    else:
        print(f"{num} is even")
```

**Expected Output:**
```
0 is even
1 is odd
2 is even
3 is odd
4 is even
```

**Breakdown:** `x & 1` checks the least significant bit, which determines parity.

**Example 3: Power of 2 Check**

```python
def is_power_of_two(n):
    return n > 0 and (n & (n - 1)) == 0

for num in [1, 2, 3, 4, 8, 12, 16]:
    print(f"{num}: {is_power_of_two(num)}")
```

**Expected Output:**
```
1: True
2: True
3: False
4: True
8: True
12: False
16: True
```

**Breakdown:** A power of 2 has exactly one bit set; `n & (n-1)` clears it.

### Real-World Cases with Explanation

**Case 1: File Permission Systems**

```python
READ = 0o4
WRITE = 0o2
EXECUTE = 0o1

user_perms = READ | WRITE  # 0o6
if user_perms & READ:
    print("Can read")
```

**Why it matters:** Unix file permissions use bitmasks for owner, group, and others.

**Case 2: Network Protocol Flags**

```python
SYN = 0x02
ACK = 0x10
FIN = 0x01

flags = SYN | ACK
if flags & SYN:
    print("SYN flag set")
```

**Why it matters:** Network protocols (like TCP) use bit flags in packet headers.


## Core Concept 8: Two's Complement and Negative Numbers

### Definitions

**Core Definition**
Two's complement is the standard method for representing signed integers in binary, where the most significant bit represents the sign and negative numbers are formed by inverting all bits and adding 1.

**Technical Definition**
Python represents negative integers in unbounded two's-complement form. The 2's-complement of an integer is equal to 1 plus its 1's-complement form (bitwise inversion). So `-n = 1 + (~n)`, which rearranges to `~n = -(n+1)`. This means `~5 = -6`, `~0 = -1`, and `~-1 = 0`. The illusion of infinite sign bits means that `-1` is represented as `...11111111` with an unbounded number of 1s to the left.

**Beginner-Friendly Explanation**
Two's complement is how computers store negative numbers. The key idea: to make a number negative, flip all its bits and add 1. Python extends this infinitely, so negative numbers behave as if they have an endless row of 1s on the left. That's why `~5` is `-6`, not `-5`.

### Key Formulas

| Operation | Formula |
|---|---|
| Bitwise NOT | `~x = -(x + 1)` |
| Two's complement | `-x = ~x + 1` |
| Verification | `~x + 1 + x = 0` |

### Multiple Annotated Complete Code Examples and Their Expected Outputs

**Example 1: Understanding `~` with Two's Complement**

```python
# Step 1: Bitwise NOT of positive numbers
print(f"~5 = {~5}")    # -6
print(f"~0 = {~0}")    # -1
print(f"~1 = {~1}")    # -2

# Step 2: Bitwise NOT of negative numbers
print(f"~-1 = {~-1}")  # 0
print(f"~-6 = {~-6}")  # 5
```

**Expected Output:**
```
~5 = -6
~0 = -1
~1 = -2
~-1 = 0
~-6 = 5
```

**Breakdown:** `~x = -(x+1)` holds for all integers.

**Example 2: Two's Complement Calculation**

```python
def twos_complement(x):
    """Compute the two's complement (negation) of x."""
    return ~x + 1

print(twos_complement(5))   # -5
print(twos_complement(-5))  # 5
print(twos_complement(0))   # 0
```

**Expected Output:**
```
-5
5
0
```

**Breakdown:** `~x + 1` computes `-x`, the two's complement.

### Real-World Cases with Explanation

**Case 1: Sign Extension in Right Shift**

```python
# Negative numbers preserve sign through right shift
x = -8
print(x >> 1)   # -4
print(x >> 2)   # -2
print(x >> 3)   # -1
```

**Why it matters:** Sign extension ensures negative numbers stay negative after right shift.

**Case 2: Bitwise Operations on Negative Numbers**

```python
# Bitwise AND with negative numbers
print(-1 & 0xFF)   # 255 (all bits in the lower byte)
print(-2 & 0xFF)   # 254
```

**Why it matters:** Masking negative numbers with `&` yields their two's complement bits in the masked range.


## Core Concept 9: Operator Precedence

### Definitions

**Core Definition**
Operator precedence determines the order in which bitwise operators are evaluated relative to each other and to other operators.

**Technical Definition**
The bitwise operators have the following precedence (highest to lowest): `~` (unary), `<<` and `>>` (shifts), `&` (AND), `^` (XOR), `|` (OR). All three binary bitwise operations have different priority levels. Comparisons (`==`, `<`, etc.) have higher precedence than bitwise operators, which in turn have higher precedence than logical `and`/`or`. This differs from C, where `&`, `^`, and `|` have lower precedence than comparisons.

**Beginner-Friendly Explanation**
Precedence is the "order of operations" for bitwise operators. `<<` and `>>` bind tightest, then `&`, then `^`, then `|`. Comparisons bind even tighter. This means `a & 3 == 1` is parsed as `a & (3 == 1)`, which is usually not what you want. Use parentheses to be safe.

### Precedence Table (Bitwise Operators)

| Level | Operators | Associativity |
|---|---|---|
| 1 (highest) | `~` (unary) | Right |
| 2 | `<<`, `>>` | Left |
| 3 | `&` | Left |
| 4 | `^` | Left |
| 5 (lowest) | `\|` | Left |

### Multiple Annotated Complete Code Examples and Their Expected Outputs

**Example 1: Precedence in Action**

```python
# Step 1: Shift binds tighter than AND
result1 = 1 << 2 & 3
# 1 << 2 = 4, then 4 & 3 = 0
print(f"1 << 2 & 3 = {result1}")

# Step 2: AND binds tighter than OR
result2 = 1 | 2 & 3
# 2 & 3 = 2, then 1 | 2 = 3
print(f"1 | 2 & 3 = {result2}")

# Step 3: Comparison binds tighter than bitwise
a = 5
result3 = a & 3 == 1
# 3 == 1 is False (0), then a & 0 = 0
print(f"a & 3 == 1 = {result3}")
```

**Expected Output:**
```
1 << 2 & 3 = 0
1 | 2 & 3 = 3
a & 3 == 1 = 0
```

**Breakdown:** Shifts bind tightest, then `&`, then `^`, then `|`. Comparisons bind even tighter than bitwise operators in Python.

**Example 2: Using Parentheses for Clarity**

```python
a = 5

# Without parentheses (confusing)
result1 = a & 3 == 1
print(f"a & 3 == 1 = {result1}")  # 0 (False)

# With parentheses (clear)
result2 = (a & 3) == 1
print(f"(a & 3) == 1 = {result2}")  # True
```

**Expected Output:**
```
a & 3 == 1 = 0
(a & 3) == 1 = True
```

**Breakdown:** Parentheses make the intent clear and override precedence.

### Real-World Cases with Explanation

**Case 1: Flag Checking**

```python
# Always use parentheses when combining with comparisons
if (flags & MASK) == EXPECTED:
    print("Match")
```

**Why it matters:** Without parentheses, the comparison binds tighter than `&`, producing incorrect results.

**Case 2: Complex Bitwise Expressions**

```python
# Clear and set bits in one expression
result = (value & ~CLEAR_MASK) | SET_MASK
```

**Why it matters:** Parentheses clarify the order of operations in complex bitwise manipulations.


## Core Concept 10: Common Pitfalls and Edge Cases

### Definitions

**Core Definition**
Common pitfalls are frequent mistakes or misunderstandings when using Python's bitwise operators, especially regarding negative numbers, precedence, and the distinction from logical operators.

**Technical Definition**
Pitfalls arise from Python's unbounded two's complement representation, the difference between bitwise and logical operators, precedence quirks, and the behaviour of shifts with negative numbers. The most common mistakes include confusing `&` with `and`, expecting `~x` to be `-x`, and misunderstanding sign extension in right shifts.

### Key Pitfalls Table

| Pitfall | Example | Result | Explanation |
|---|---|---|---|
| `&` vs. `and` | `5 & 3` vs. `5 and 3` | `1` vs. `3` | Bitwise vs. logical |
| `~x` is not `-x` | `~5` | `-6` | `~x = -(x+1)` |
| Precedence | `a & 3 == 1` | `a & (3 == 1)` | Comparison binds tighter |
| Negative right shift | `-10 >> 1` | `-5` | Sign extension |
| Shift by negative | `5 << -1` | `ValueError` | Negative shift counts invalid |
| Large shifts | `1 << 1000000` | Huge int | Memory-intensive |

### Multiple Annotated Complete Code Examples and Their Expected Outputs

**Example 1: Bitwise vs. Logical**

```python
# Step 1: Bitwise AND
print(f"5 & 3 = {5 & 3}")   # 1

# Step 2: Logical AND
print(f"5 and 3 = {5 and 3}")  # 3 (returns second operand)

# Step 3: Different results
print(f"5 | 3 = {5 | 3}")   # 7 (bitwise OR)
print(f"5 or 3 = {5 or 3}")  # 5 (logical OR)
```

**Expected Output:**
```
5 & 3 = 1
5 and 3 = 3
5 | 3 = 7
5 or 3 = 5
```

**Breakdown:** Bitwise operators work on bits; logical operators work on truthiness and return operands.

**Example 2: `~x` is not `-x`**

```python
x = 5
print(f"~x = {~x}")      # -6, not -5
print(f"~x + 1 = {~x + 1}")  # -5, which is -x
```

**Expected Output:**
```
~x = -6
~x + 1 = -5
```

**Breakdown:** `~x` is bitwise inversion; `-x` is arithmetic negation. They differ by 1.

**Example 3: Negative Shift Counts**

```python
try:
    result = 5 << -1
except ValueError as e:
    print(f"Error: {e}")

try:
    result = 5 >> -1
except ValueError as e:
    print(f"Error: {e}")
```

**Expected Output:**
```
Error: negative shift count
Error: negative shift count
```

**Breakdown:** Shift counts must be non-negative integers.

### Real-World Cases with Explanation

**Case 1: Permission Bugs**

```python
# BUG: Using 'and' instead of '&' for flags
if permissions and READ:  # Wrong! Checks truthiness
    ...

# FIX: Use '&' for bitwise flag check
if permissions & READ:    # Correct
    ...
```

**Why it matters:** Using `and` instead of `&` for flag checks produces incorrect results.

**Case 2: Masking Negative Numbers**

```python
# Negative numbers have infinite leading 1s
x = -1
print(f"x & 0xFF = {x & 0xFF}")  # 255
print(f"x >> 4 = {x >> 4}")       # -1 (sign extended)
```

**Why it matters:** Understanding two's complement is essential for correct bitwise operations on negative numbers.


## References

- Python Software Foundation. *Expressions — Python Language Reference (Bitwise Operations)*. https://docs.python.org/3/reference/expressions.html#unary-arithmetic-and-bitwise-operations
- Python Software Foundation. *Operator precedence*. https://docs.python.org/3/reference/expressions.html#operator-precedence
- Python Software Foundation. *BitwiseOperators — Python Wiki*. https://wiki.python.org/moin/BitwiseOperators
- Real Python. *Bitwise Operators in Python*. https://realpython.com/python-bitwise-operators/
- W3Schools. *Python Bitwise Operators*. https://www.w3schools.com/python/gloss_python_bitwise_operators.asp
- Python Mailing List. *bitwise not - not what I expected*. https://mail.python.org/pipermail/python-list/2003-August/197767.html
- Stack Overflow. *~ Binary Ones Complement in Python 3*. https://stackoverflow.com/questions/55145028/binary-ones-complement-in-python-3
- Python Software Foundation. *Operator precedence problem*. https://mail.python.org/pipermail/python-list/2016-June/708059.html
- KodeKloud. *Bitwise Operators*. https://notes.kodekloud.com/docs/Python/Bitwise-Operators
- Python Software Foundation. *manipulating bits in python*. https://mail.python.org/pipermail/tutor/2004-November/034103.html
- Real Python. *Bitwise in Practice: Flipping Bits*. https://realpython.com/lessons/bitwise-practice-flipping-bits/