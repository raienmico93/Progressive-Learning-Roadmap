# Python Arithmetic Operators: A Comprehensive Programming Cheat Sheet

## Topic Overview

### Definitions

**Core Definition**
Arithmetic operators are symbols that perform mathematical operations on numeric values (operands), producing a result.

**Technical Definition**
Python's arithmetic operators are part of the language's expression grammar and are defined for the built-in numeric types `int`, `float`, and `complex`. The operators `+`, `-`, `*`, `/`, `//`, `%`, and `**` implement addition, subtraction, multiplication, true division, floor division, modulus, and exponentiation respectively. Python's numeric model follows the principle that division (`/`) always returns a `float`, while floor division (`//`) and modulus (`%`) operate together such that `(a // b) * b + (a % b) == a`. The `**` operator is right-associative, while all other arithmetic operators are left-associative . Python also supports mixed-type arithmetic, where operands of different numeric types are coerced according to a well-defined hierarchy (`int` → `float` → `complex`).

**Beginner-Friendly Explanation**
Arithmetic operators are the basic math symbols you already know from school: `+` for addition, `-` for subtraction, `*` for multiplication, and `/` for division. Python adds a few more: `//` for "how many whole times does one number fit into another," `%` for "what's left over," and `**` for "raise to a power." You use them exactly like a calculator: `2 + 3` gives `5`, `10 / 3` gives `3.333...`, and `2 ** 3` gives `8`.

### Key Characteristics

| Characteristic | Description |
|---|---|
| **Binary Operators** | Most arithmetic operators take two operands (left and right) |
| **Unary Operators** | `+` and `-` can also be unary (e.g., `-5`) |
| **Type Coercion** | Mixed-type operations promote to the "wider" type |
| **Division Semantics** | `/` always returns `float`; `//` returns floor (int or float) |
| **Right-Associative `**`** | `2 ** 3 ** 2` is `2 ** (3 ** 2)`, not `(2 ** 3) ** 2` |
| **Augmented Forms** | Each operator has a shorthand assignment form (`+=`, `-=`, etc.) |

### Prerequisites

- **Basic Python syntax**: variables, expressions, and statements
- **Numeric types**: understanding of `int` and `float`
- **Operator precedence**: familiarity with order of operations
- **The `print()` function**: for displaying results

### Related Programming Areas

- **Comparison Operators**: `==`, `!=`, `<`, `>`, etc., for comparing values
- **Logical Operators**: `and`, `or`, `not`, for boolean logic
- **Bitwise Operators**: `&`, `|`, `^`, `<<`, `>>`, for integer bit manipulation
- **Math Module**: `math.sqrt()`, `math.floor()`, `math.ceil()`, etc.
- **Augmented Assignment**: `+=`, `-=`, etc., for in-place updates

### Core Concepts / Features

1. **`+` (Addition)**
2. **`-` (Subtraction)**
3. **`*` (Multiplication)**
4. **`/` (Division)**
5. **`//` (Floor Division)**
6. **`%` (Modulus/Remainder)**
7. **`**` (Exponentiation)**
8. **Operator Precedence and Associativity**
9. **Augmented Assignment Operators**
10. **Common Pitfalls and Edge Cases**


## Core Concept 1: `+` (Addition)

### Definitions

**Core Definition**
The `+` operator performs arithmetic addition, combining two numeric values into their sum.

**Technical Definition**
The `+` operator is defined by the `__add__()` and `__radd__()` special methods. For numeric types, it computes the mathematical sum. For sequences (strings, lists, tuples), it performs concatenation, which is a distinct behaviour that shares the same symbol. Python's numeric addition follows the standard rules of arithmetic, with automatic type promotion when operands are of different numeric types (e.g., `int + float` yields `float`).

**Beginner-Friendly Explanation**
The `+` operator adds two numbers together. `2 + 3` gives `5`. It can also add a number and a variable, or two variables. When you add an integer and a float, the result is a float (e.g., `2 + 3.5` gives `5.5`).

### Purposes

- **To compute the sum of two numbers**
- **To increment a value** (often with `+=`)
- **To combine numeric quantities** in calculations
- **To concatenate sequences** (strings, lists, tuples)
- **To accumulate totals** in loops and aggregations

### Syntax Rules and Structure

**Complete General Syntax**

```
operand1 + operand2
```

**Breakdown:**
- `operand1`, `operand2`: Numeric expressions (or sequences for concatenation).
- Returns the sum (or concatenated sequence).

**Syntax Rules**

| Rule | Description |
|---|---|
| Operand types | Numbers (`int`, `float`, `complex`) or sequences |
| Type promotion | `int + float` → `float`; `int + complex` → `complex` |
| Associativity | Left-to-right: `a + b + c` = `(a + b) + c` |
| Augmented form | `x += y` is equivalent to `x = x + y` |
| Unary form | `+x` returns `x` unchanged (identity) |

**Constraints and Limitations**

- **Cannot mix numbers and sequences**: `3 + "abc"` raises `TypeError`.
- **Float precision**: Floating-point addition may produce rounding errors (e.g., `0.1 + 0.2 == 0.30000000000000004`).
- **Overflow**: Python integers have arbitrary precision, so no overflow for `int`.

### Multiple Annotated Complete Code Examples and Their Expected Outputs

**Example 1: Basic Addition**

```python
# Step 1: Integer addition
result1 = 2 + 3
print(f"2 + 3 = {result1}")

# Step 2: Float addition
result2 = 2.5 + 3.7
print(f"2.5 + 3.7 = {result2}")

# Step 3: Mixed-type addition
result3 = 2 + 3.5
print(f"2 + 3.5 = {result3}")
print(f"Type: {type(result3)}")
```

**Expected Output:**
```
2 + 3 = 5
2.5 + 3.7 = 6.2
2 + 3.5 = 5.5
Type: <class 'float'>
```

**Breakdown:** Integer addition produces an `int`. Mixed-type addition promotes the `int` to `float`, producing a `float`.

**Example 2: Accumulation in a Loop**

```python
# Step 1: Initialize total
total = 0

# Step 2: Add each number
for num in [10, 20, 30, 40]:
    total += num  # Equivalent to total = total + num
    print(f"After adding {num}: total = {total}")

print(f"Final total: {total}")
```

**Expected Output:**
```
After adding 10: total = 10
After adding 20: total = 30
After adding 30: total = 60
After adding 40: total = 100
Final total: 100
```

**Breakdown:** The `+=` operator adds to the running total in each iteration.

**Example 3: Sequence Concatenation**

```python
# Step 1: String concatenation
greeting = "Hello, " + "World!"
print(greeting)

# Step 2: List concatenation
list1 = [1, 2, 3]
list2 = [4, 5, 6]
combined = list1 + list2
print(combined)
```

**Expected Output:**
```
Hello, World!
[1, 2, 3, 4, 5, 6]
```

**Breakdown:** The `+` operator concatenates sequences as well as adding numbers. This is a form of operator overloading.

### Real-World Cases with Explanation

**Case 1: Shopping Cart Total**

```python
prices = [19.99, 5.49, 12.00, 3.99]
total = 0
for price in prices:
    total += price
print(f"Total: ${total:.2f}")
```

**Why it matters:** Addition and `+=` are fundamental for accumulating totals in financial calculations.

**Case 2: Building Strings**

```python
first_name = "Alice"
last_name = "Smith"
full_name = first_name + " " + last_name
print(full_name)  # Alice Smith
```

**Why it matters:** String concatenation with `+` is a common operation for building output messages.


## Core Concept 2: `-` (Subtraction)

### Definitions

**Core Definition**
The `-` operator performs arithmetic subtraction, finding the difference between two numeric values.

**Technical Definition**
The `-` operator is defined by the `__sub__()` and `__rsub__()` special methods. For numeric types, it computes the mathematical difference. It also serves as the unary negation operator, which is implemented by `__neg__()`. Binary subtraction is left-associative: `a - b - c` evaluates as `(a - b) - c`. Unlike addition, subtraction is not commutative, so operand order matters.

**Beginner-Friendly Explanation**
The `-` operator subtracts the second number from the first. `10 - 3` gives `7`. It can also make a number negative: `-5` is "negative five."

### Purposes

- **To compute the difference between two numbers**
- **To decrement a value** (often with `-=`)
- **To negate a number** (unary `-`)
- **To calculate remaining quantities** (e.g., inventory after a sale)
- **To find differences** in comparisons and analytics

### Syntax Rules and Structure

**Complete General Syntax**

```
operand1 - operand2    # binary subtraction
-operand               # unary negation
```

**Breakdown:**
- Binary `-`: Subtracts `operand2` from `operand1`.
- Unary `-`: Returns the negation of `operand`.

**Syntax Rules**

| Rule | Description |
|---|---|
| Binary subtraction | `a - b` computes `a` minus `b` |
| Unary negation | `-a` returns the negative of `a` |
| Associativity | Left-to-right: `a - b - c` = `(a - b) - c` |
| Augmented form | `x -= y` is equivalent to `x = x - y` |
| Type promotion | `int - float` → `float` |

**Constraints and Limitations**

- **Cannot subtract sequences**: `"abc" - "a"` raises `TypeError`.
- **Floating-point precision**: Subtracting nearly equal floats can cause precision loss.
- **Not commutative**: `a - b` is not the same as `b - a`.

### Multiple Annotated Complete Code Examples and Their Expected Outputs

**Example 1: Basic Subtraction**

```python
# Step 1: Integer subtraction
print(10 - 3)

# Step 2: Float subtraction
print(10.5 - 3.2)

# Step 3: Unary negation
x = 5
print(-x)
print(-(-x))
```

**Expected Output:**
```
7
7.3
-5
5
```

**Breakdown:** Binary subtraction finds the difference. Unary `-` negates a value; applying it twice returns the original.

**Example 2: Left-Associativity**

```python
# Step 1: Without parentheses
result1 = 10 - 5 - 2
print(f"10 - 5 - 2 = {result1}")

# Step 2: With explicit parentheses (same result)
result2 = (10 - 5) - 2
print(f"(10 - 5) - 2 = {result2}")

# Step 3: Different grouping changes result
result3 = 10 - (5 - 2)
print(f"10 - (5 - 2) = {result3}")
```

**Expected Output:**
```
10 - 5 - 2 = 3
(10 - 5) - 2 = 3
10 - (5 - 2) = 7
```

**Breakdown:** Subtraction is left-associative, so `10 - 5 - 2` is `(10 - 5) - 2 = 3`. Parentheses can override this.

**Example 3: Decrementing**

```python
# Step 1: Countdown
count = 5
while count > 0:
    print(count)
    count -= 1  # Equivalent to count = count - 1
print("Liftoff!")
```

**Expected Output:**
```
5
4
3
2
1
Liftoff!
```

**Breakdown:** `-=` decrements the variable in each iteration until the condition fails.

### Real-World Cases with Explanation

**Case 1: Inventory Tracking**

```python
stock = 100
sold = 37
remaining = stock - sold
print(f"Remaining: {remaining}")
```

**Why it matters:** Subtraction is essential for calculating remaining quantities after consumption or sales.

**Case 2: Time Difference**

```python
start_time = 10.5  # seconds
end_time = 15.75
elapsed = end_time - start_time
print(f"Elapsed: {elapsed} seconds")
```

**Why it matters:** Calculating elapsed time is a common operation in profiling and benchmarking.


## Core Concept 3: `*` (Multiplication)

### Definitions

**Core Definition**
The `*` operator performs arithmetic multiplication, computing the product of two numeric values.

**Technical Definition**
The `*` operator is defined by the `__mul__()` and `__rmul__()` special methods. For numeric types, it computes the mathematical product. For sequences, it performs repetition: `"abc" * 3` yields `"abcabcabc"`, and `[1, 2] * 2` yields `[1, 2, 1, 2]`. Multiplication is left-associative and commutative for numbers, and it supports mixed-type promotion (`int * float` → `float`).

**Beginner-Friendly Explanation**
The `*` operator multiplies two numbers. `3 * 4` gives `12`. It can also repeat sequences: `"ha" * 3` gives `"hahaha"`.

### Purposes

- **To compute the product of two numbers**
- **To scale values** (multiply by a factor)
- **To repeat sequences** (strings, lists, tuples)
- **To calculate areas, volumes, and rates**
- **To implement powers of 2 and 10** (though `**` is more general)

### Syntax Rules and Structure

**Complete General Syntax**

```
operand1 * operand2
```

**Breakdown:**
- For numbers: returns the product.
- For sequences: returns the sequence repeated `operand2` times.

**Syntax Rules**

| Rule | Description |
|---|---|
| Numeric multiplication | `a * b` computes the product |
| Sequence repetition | `seq * n` or `n * seq` repeats the sequence |
| Associativity | Left-to-right: `a * b * c` = `(a * b) * c` |
| Augmented form | `x *= y` is equivalent to `x = x * y` |
| Type promotion | `int * float` → `float` |

**Constraints and Limitations**

- **Cannot multiply two sequences**: `[1] * [2]` raises `TypeError`.
- **Sequence repetition must use an integer**: `"abc" * 2.5` raises `TypeError`.
- **Float precision**: Repeated multiplication can accumulate rounding errors.

### Multiple Annotated Complete Code Examples and Their Expected Outputs

**Example 1: Basic Multiplication**

```python
# Step 1: Integer multiplication
print(3 * 4)

# Step 2: Float multiplication
print(2.5 * 4.0)

# Step 3: Mixed-type
print(3 * 2.5)
```

**Expected Output:**
```
12
10.0
7.5
```

**Breakdown:** Multiplication follows standard arithmetic rules. Mixed types promote to `float`.

**Example 2: Sequence Repetition**

```python
# Step 1: String repetition
print("ab" * 3)

# Step 2: List repetition
print([0] * 5)

# Step 3: Tuple repetition
print((1, 2) * 2)
```

**Expected Output:**
```
ababab
[0, 0, 0, 0, 0]
(1, 2, 1, 2)
```

**Breakdown:** The `*` operator repeats sequences when one operand is an integer.

**Example 3: Area Calculation**

```python
# Step 1: Rectangle area
width = 5.5
height = 3.0
area = width * height
print(f"Area: {area} square units")
```

**Expected Output:**
```
Area: 16.5 square units
```

**Breakdown:** Multiplication is used for geometric calculations like area.

### Real-World Cases with Explanation

**Case 1: Scaling Values**

```python
base_price = 100
tax_rate = 1.08
final_price = base_price * tax_rate
print(f"Final price: ${final_price:.2f}")
```

**Why it matters:** Multiplying by a rate is common in financial calculations (tax, discount, interest).

**Case 2: Creating Repeated Patterns**

```python
separator = "-" * 40
print(separator)
print("Title")
print(separator)
```

**Why it matters:** String repetition is useful for creating visual separators and formatting.


## Core Concept 4: `/` (Division)

### Definitions

**Core Definition**
The `/` operator performs true division, returning the quotient as a floating-point number regardless of operand types.

**Technical Definition**
The `/` operator is defined by the `__truediv__()` and `__rtruediv__()` special methods. In Python 3, division (`/`) always returns a `float`, even when both operands are integers and the result is a whole number. This is a deliberate language design choice (PEP 238) that distinguishes true division from floor division (`//`). The result follows IEEE 754 floating-point semantics. Division by zero raises `ZeroDivisionError`.

**Beginner-Friendly Explanation**
The `/` operator divides one number by another and always gives you a decimal result. `10 / 3` gives `3.3333333333333335`, and `10 / 2` gives `5.0` (note the `.0` — it's still a float). Even `4 / 2` gives `2.0`, not `2`.

### Purposes

- **To compute the quotient of two numbers**
- **To calculate ratios and rates**
- **To convert between units** (e.g., kilometres to miles)
- **To compute averages**
- **To perform any division where a fractional result is expected**

### Syntax Rules and Structure

**Complete General Syntax**

```
operand1 / operand2
```

**Breakdown:**
- Returns the quotient as a `float`.
- Raises `ZeroDivisionError` if `operand2` is zero.

**Syntax Rules**

| Rule | Description |
|---|---|
| Always returns `float` | Even `4 / 2` returns `2.0` |
| Division by zero | Raises `ZeroDivisionError` |
| Associativity | Left-to-right: `a / b / c` = `(a / b) / c` |
| Augmented form | `x /= y` is equivalent to `x = x / y` |
| Operand types | Numbers (`int`, `float`, `complex`) |

**Constraints and Limitations**

- **Division by zero**: Must be guarded against.
- **Float precision**: Results may have rounding errors.
- **Always returns float**: If you need an integer result, use `//`.

### Multiple Annotated Complete Code Examples and Their Expected Outputs

**Example 1: True Division**

```python
# Step 1: Integer division that produces a fraction
print(10 / 3)

# Step 2: Integer division that produces a whole number
print(10 / 2)
print(type(10 / 2))

# Step 3: Float division
print(7.5 / 2.5)
```

**Expected Output:**
```
3.3333333333333335
5.0
<class 'float'>
7.5 / 2.5 = 3.0
```

**Breakdown:** `/` always returns a `float`, even when the result is a whole number.

**Example 2: Division by Zero**

```python
# Step 1: Attempt division by zero
try:
    result = 10 / 0
except ZeroDivisionError as e:
    print(f"Error: {e}")

# Step 2: Guard against zero
denominator = 0
if denominator != 0:
    result = 10 / denominator
else:
    print("Cannot divide by zero")
```

**Expected Output:**
```
Error: division by zero
Cannot divide by zero
```

**Breakdown:** Division by zero raises `ZeroDivisionError`. Always check the denominator when it could be zero.

**Example 3: Calculating an Average**

```python
# Step 1: Compute the average of a list
numbers = [10, 20, 30, 40, 50]
average = sum(numbers) / len(numbers)
print(f"Average: {average}")
```

**Expected Output:**
```
Average: 30.0
```

**Breakdown:** Division is essential for computing averages.

### Real-World Cases with Explanation

**Case 1: Unit Conversion**

```python
kilometres = 42.195
miles = kilometres / 1.60934
print(f"{kilometres} km = {miles:.2f} miles")
```

**Why it matters:** Division is used for unit conversions between different measurement systems.

**Case 2: Percentage Calculation**

```python
score = 85
total = 100
percentage = (score / total) * 100
print(f"Percentage: {percentage}%")
```

**Why it matters:** Division computes the ratio, which is then scaled to a percentage.


## Core Concept 5: `//` (Floor Division)

### Definitions

**Core Definition**
The `//` operator performs floor division, dividing two numbers and rounding the result down to the nearest integer (toward negative infinity).

**Technical Definition**
Floor division uses the double-slash (`//`) operator, discards any remainders, and returns only an `int` (when both operands are integers) . The result is the largest integer less than or equal to the mathematical quotient. Floor division rounds toward negative infinity, not toward zero: `-1 // 3` is `-1`, not `0` . Floor division and modulus are complementary: `(a // b) * b + (a % b) == a` for any non-zero `b`. Floor division by zero raises `ZeroDivisionError`.

**Beginner-Friendly Explanation**
The `//` operator divides and then rounds down to the nearest whole number. `10 // 3` gives `3` (because 3.33 rounded down is 3). It's useful when you want to know "how many whole times does this fit?" without caring about the remainder. Note that for negative numbers, it rounds down (toward negative infinity), so `-1 // 3` gives `-1`, not `0`.

### Purposes

- **To compute how many whole times** one number fits into another
- **To get an integer result from division** when the remainder is not needed
- **To implement pagination** (calculating the number of pages)
- **To bucket values** into groups
- **To complement modulus** for exact division verification

### Syntax Rules and Structure

**Complete General Syntax**

```
operand1 // operand2
```

**Breakdown:**
- Returns the floor of the quotient.
- Returns `int` if both operands are `int`; returns `float` if either is `float`.

**Syntax Rules**

| Rule | Description |
|---|---|
| Rounds toward negative infinity | `-1 // 3` → `-1`, not `0` |
| Returns int or float | `int // int` → `int`; `float // int` → `float` |
| Division by zero | Raises `ZeroDivisionError` |
| Complementary to `%` | `(a // b) * b + (a % b) == a` |
| Augmented form | `x //= y` |

**Constraints and Limitations**

- **Rounds down, not toward zero**: For negative numbers, the result is more negative than expected.
- **Division by zero**: Must be guarded against.
- **Result type depends on operands**: `7 // 2` is `3`, but `7.0 // 2` is `3.0`.

### Multiple Annotated Complete Code Examples and Their Expected Outputs

**Example 1: Basic Floor Division**

```python
# Step 1: Positive numbers
print(10 // 3)    # 3
print(10 // 2)    # 5
print(10 // 4)    # 2

# Step 2: Negative numbers
print(-10 // 3)   # -4 (rounds down)
print(10 // -3)   # -4
print(-10 // -3)  # 3
```

**Expected Output:**
```
3
5
2
-4
-4
3
```

**Breakdown:** Floor division rounds toward negative infinity. `-10 // 3` is `-4` because `-3.33` rounds down to `-4`.

**Example 2: Complementarity with Modulus**

```python
# Step 1: Verify the relationship
a, b = 17, 5
quotient = a // b
remainder = a % b
print(f"{a} // {b} = {quotient}")
print(f"{a} % {b} = {remainder}")
print(f"Check: {quotient} * {b} + {remainder} = {quotient * b + remainder}")
```

**Expected Output:**
```
17 // 5 = 3
17 % 5 = 2
Check: 3 * 5 + 2 = 17
```

**Breakdown:** Floor division and modulus satisfy `(a // b) * b + (a % b) == a`.

**Example 3: Pagination**

```python
# Step 1: Calculate number of pages needed
total_items = 47
items_per_page = 10
pages = total_items // items_per_page
if total_items % items_per_page != 0:
    pages += 1  # Add one for the partial page
print(f"Pages needed: {pages}")
```

**Expected Output:**
```
Pages needed: 5
```

**Breakdown:** Floor division gives the number of full pages; modulus checks if an extra page is needed.

### Real-World Cases with Explanation

**Case 1: Time Conversion**

```python
total_seconds = 3672
hours = total_seconds // 3600
minutes = (total_seconds % 3600) // 60
seconds = total_seconds % 60
print(f"{hours}h {minutes}m {seconds}s")
```

**Why it matters:** Floor division extracts whole units when converting between time units.

**Case 2: Splitting Into Batches**

```python
items = list(range(23))
batch_size = 5
for i in range(0, len(items), batch_size):
    batch = items[i:i + batch_size]
    print(f"Batch: {batch}")
```

**Why it matters:** Floor division determines how many complete batches can be formed.


## Core Concept 6: `%` (Modulus/Remainder)

### Definitions

**Core Definition**
The `%` operator returns the remainder after dividing one number by another.

**Technical Definition**
The modulo operator (`%`) returns the remainder of dividing the first operand by the second . In Python, the result of `a % b` has the same sign as the divisor `b` (unlike C and Java, where the result has the sign of the dividend). The relationship `(a // b) * b + (a % b) == a` always holds. Modulus by zero raises `ZeroDivisionError`. The modulo operator is essential for cycle detection, checking divisibility, and wrapping indices.

**Beginner-Friendly Explanation**
The `%` operator gives you the remainder after division. `10 % 3` gives `1` (because 10 divided by 3 is 3 with 1 left over). It's useful for checking if a number is even (`x % 2 == 0`) or for wrapping around a list of items.

### Purposes

- **To determine if a number is divisible** by another (`x % n == 0`)
- **To check if a number is even or odd** (`x % 2`)
- **To cycle through indices** in a circular manner
- **To extract the last digit** of a number (`x % 10`)
- **To implement hash functions** and bucketing

### Syntax Rules and Structure

**Complete General Syntax**

```
operand1 % operand2
```

**Breakdown:**
- Returns the remainder of `operand1` divided by `operand2`.
- Result has the same sign as `operand2`.

**Syntax Rules**

| Rule | Description |
|---|---|
| Sign follows divisor | `-4 % 3` → `2`; `4 % -3` → `-2` |
| Division by zero | Raises `ZeroDivisionError` |
| Complementarity | `(a // b) * b + (a % b) == a` |
| Augmented form | `x %= y` |
| Float support | Works with floats (e.g., `7.5 % 2` → `1.5`) |

**Constraints and Limitations**

- **Sign behavior differs from C/Java**: Python's `%` follows the divisor's sign.
- **Division by zero**: Raises `ZeroDivisionError`.
- **Float precision**: Modulus with floats may produce rounding artefacts.

### Multiple Annotated Complete Code Examples and Their Expected Outputs

**Example 1: Basic Modulus**

```python
# Step 1: Positive numbers
print(10 % 3)    # 1
print(10 % 5)    # 0
print(10 % 7)    # 3

# Step 2: Negative numbers
print(-4 % 3)    # 2 (sign follows divisor)
print(4 % -3)    # -2
print(-4 % -3)   # -1
```

**Expected Output:**
```
1
0
3
2
-2
-1
```

**Breakdown:** Modulus returns the remainder. The sign of the result follows the divisor, not the dividend.

**Example 2: Even/Odd Check**

```python
# Step 1: Check even/odd
for num in range(1, 6):
    if num % 2 == 0:
        print(f"{num} is even")
    else:
        print(f"{num} is odd")
```

**Expected Output:**
```
1 is odd
2 is even
3 is odd
4 is even
5 is odd
```

**Breakdown:** `x % 2 == 0` is the standard way to check for even numbers.

**Example 3: Circular Indexing**

```python
# Step 1: Cycle through a list repeatedly
items = ["a", "b", "c"]
for i in range(7):
    index = i % len(items)
    print(f"i={i}, item={items[index]}")
```

**Expected Output:**
```
i=0, item=a
i=1, item=b
i=2, item=c
i=3, item=a
i=4, item=b
i=5, item=c
i=6, item=a
```

**Breakdown:** `%` wraps the index back to 0 after reaching the end of the list.

### Real-World Cases with Explanation

**Case 1: Checking Divisibility**

```python
def is_leap_year(year):
    return year % 4 == 0 and (year % 100 != 0 or year % 400 == 0)

print(is_leap_year(2024))  # True
print(is_leap_year(1900))  # False
```

**Why it matters:** Modulus checks divisibility, essential for calendar and date calculations.

**Case 2: Extracting Digits**

```python
number = 12345
last_digit = number % 10
print(f"Last digit: {last_digit}")  # 5
```

**Why it matters:** `% 10` extracts the last digit of a number, useful in digit manipulation problems.


## Core Concept 7: `**` (Exponentiation)

### Definitions

**Core Definition**
The `**` operator raises a number to a power, computing exponentiation.

**Technical Definition**
The `**` operator is defined by the `__pow__()` and `__rpow__()` special methods. It computes `base ** exponent`. Unlike other arithmetic operators, `**` is right-associative: `2 ** 3 ** 2` is `2 ** (3 ** 2) = 2 ** 9 = 512`, not `(2 ** 3) ** 2 = 64` . It also has higher precedence than unary negation, so `-5 ** 2` is `-(5 ** 2) = -25`, not `(-5) ** 2 = 25` . The operator supports integer, float, and complex exponents.

**Beginner-Friendly Explanation**
The `**` operator raises a number to a power. `2 ** 3` means "2 to the power of 3," which is `8`. It's like multiplying 2 by itself 3 times. You can also use it for square roots: `9 ** 0.5` gives `3.0`.

### Purposes

- **To compute powers** (e.g., `x ** 2` for squaring)
- **To compute roots** (e.g., `x ** 0.5` for square root)
- **To implement exponential growth** calculations
- **To raise numbers to arbitrary powers** in mathematical formulas
- **To compute compound interest** and scientific notation

### Syntax Rules and Structure

**Complete General Syntax**

```
base ** exponent
```

**Breakdown:**
- `base`: The number being raised to a power.
- `exponent`: The power to which the base is raised.
- Returns `base` raised to `exponent`.

**Syntax Rules**

| Rule | Description |
|---|---|
| Right-associative | `a ** b ** c` = `a ** (b ** c)` |
| Higher than unary `-` | `-a ** b` = `-(a ** b)` |
| Higher than `*`, `/`, `//`, `%` | `a * b ** c` = `a * (b ** c)` |
| Augmented form | `x **= y` |
| Negative exponent | Returns a fraction (e.g., `2 ** -1` = `0.5`) |

**Constraints and Limitations**

- **Right-associativity**: `2 ** 3 ** 2` is `512`, not `64`.
- **Precedence with unary minus**: `-5 ** 2` is `-25`, not `25`.
- **Large results**: Python integers can grow arbitrarily large; `2 ** 1000` is fine, but memory-intensive.

### Multiple Annotated Complete Code Examples and Their Expected Outputs

**Example 1: Basic Exponentiation**

```python
# Step 1: Integer powers
print(2 ** 3)     # 8
print(5 ** 2)     # 25
print(10 ** 0)    # 1

# Step 2: Float exponent
print(2 ** 0.5)   # 1.4142135623730951 (square root of 2)
print(9 ** 0.5)   # 3.0

# Step 3: Negative exponent
print(2 ** -1)    # 0.5
print(2 ** -2)    # 0.25
```

**Expected Output:**
```
8
25
1
1.4142135623730951
3.0
0.5
0.25
```

**Breakdown:** `**` handles integer powers, roots (fractional exponents), and negative exponents.

**Example 2: Right-Associativity**

```python
# Step 1: Right-associative evaluation
result1 = 2 ** 3 ** 2
print(f"2 ** 3 ** 2 = {result1}")  # 2 ** (3 ** 2) = 2 ** 9 = 512

# Step 2: Compare with explicit grouping
result2 = (2 ** 3) ** 2
print(f"(2 ** 3) ** 2 = {result2}")  # 8 ** 2 = 64
```

**Expected Output:**
```
2 ** 3 ** 2 = 512
(2 ** 3) ** 2 = 64
```

**Breakdown:** `**` is right-associative: it groups from the right. Parentheses are needed to change the grouping.

**Example 3: Precedence with Unary Minus**

```python
# Step 1: Without parentheses
result1 = -5 ** 2
print(f"-5 ** 2 = {result1}")  # -(5 ** 2) = -25

# Step 2: With parentheses
result2 = (-5) ** 2
print(f"(-5) ** 2 = {result2}")  # (-5) ** 2 = 25
```

**Expected Output:**
```
-5 ** 2 = -25
(-5) ** 2 = 25
```

**Breakdown:** `**` binds tighter than unary minus, so `-5 ** 2` is `-(5 ** 2)`. Parentheses are needed for `(-5) ** 2`.

### Real-World Cases with Explanation

**Case 1: Compound Interest**

```python
principal = 1000
rate = 0.05
years = 10
amount = principal * (1 + rate) ** years
print(f"Final amount: ${amount:.2f}")
```

**Why it matters:** Exponentiation is used in financial formulas for compound interest.

**Case 2: Distance Calculation**

```python
x1, y1 = 0, 0
x2, y2 = 3, 4
distance = ((x2 - x1) ** 2 + (y2 - y1) ** 2) ** 0.5
print(f"Distance: {distance}")
```

**Why it matters:** Exponentiation computes squares and square roots in geometric formulas.


## Core Concept 8: Operator Precedence and Associativity

### Definitions

**Core Definition**
Operator precedence determines the order in which operators are evaluated; associativity determines the grouping when operators have the same precedence.

**Technical Definition**
Python's operator precedence table defines a strict hierarchy. For arithmetic operators, from highest to lowest precedence: `**` > unary `+`/`-` > `*`, `/`, `//`, `%` > binary `+`, `-` . All arithmetic operators are left-associative except `**`, which is right-associative . Parentheses can override precedence and are evaluated first. Understanding precedence is essential for writing correct expressions without excessive parentheses.

**Beginner-Friendly Explanation**
Precedence is like the "order of operations" you learned in math: multiplication before addition. In Python, `**` comes first, then `*`, `/`, `//`, `%`, and finally `+` and `-`. Associativity determines what happens when operators have the same precedence: `10 - 5 - 2` is `(10 - 5) - 2 = 3`, because `-` is left-associative.

### Purposes

- **To write expressions correctly** without excessive parentheses
- **To understand evaluation order** in complex formulas
- **To avoid bugs** caused by precedence surprises
- **To use parentheses effectively** for clarity

### Syntax Rules and Structure

**Precedence Table (Arithmetic, Highest to Lowest)**

| Level | Operators | Associativity |
|---|---|---|
| 1 (highest) | `**` | Right |
| 2 | unary `+`, `-` | Right |
| 3 | `*`, `/`, `//`, `%` | Left |
| 4 (lowest) | binary `+`, `-` | Left |

**Examples**

```python
# Precedence: ** before *
result1 = 2 + 3 * 4      # 2 + (3 * 4) = 14

# Precedence: * before +
result2 = 5 + 4 % 3      # 5 + (4 % 3) = 6

# Precedence: ** before *
result3 = 2 ** 3 * 4     # (2 ** 3) * 4 = 32

# Associativity: left-to-right for -
result4 = 5 - 4 - 3      # (5 - 4) - 3 = -2

# Associativity: right-to-left for **
result5 = 4 ** 3 ** 2    # 4 ** (3 ** 2) = 262144
```

**Expected Output:**
```
14
6
32
-2
262144
```

**Breakdown:** `**` binds tightest, followed by unary minus, then `*`/`/`/`//`/`%`, then `+`/`-`. Parentheses can override any precedence.

### Real-World Cases with Explanation

**Case 1: Financial Formula**

```python
# Without parentheses (correct due to precedence)
total = principal * (1 + rate) ** years

# With excessive parentheses (works but verbose)
total = principal * ((1 + rate) ** years)
```

**Why it matters:** Understanding precedence allows writing clean expressions without unnecessary parentheses.

**Case 2: Debugging Precedence Bugs**

```python
# BUG: Intended (a + b) / 2, but got a + (b / 2)
a, b = 10, 20
wrong = a + b / 2   # 10 + 10.0 = 20.0
right = (a + b) / 2  # 30 / 2 = 15.0
print(f"Wrong: {wrong}, Right: {right}")
```

**Why it matters:** Precedence bugs are subtle and can lead to incorrect results. Parentheses clarify intent.


## Core Concept 9: Augmented Assignment Operators

### Definitions

**Core Definition**
Augmented assignment operators combine an arithmetic operation with assignment, updating a variable in place.

**Technical Definition**
Python provides augmented assignment operators that combine binary operations with assignment. The operators are `+=`, `-=`, `*=`, `/=`, `//=`, `%=`, and `**=`. The statement `x += y` is equivalent to `x = x + y`, but it evaluates `x` only once and can perform the operation in place for mutable objects. These operators are syntactically distinct and cannot be used in expressions (they are statements, not expressions).

**Beginner-Friendly Explanation**
Augmented assignment is a shorthand: instead of writing `x = x + 5`, you can write `x += 5`. It does the same thing but is shorter and clearer. You can use it with any arithmetic operator.

### Purposes

- **To update a variable concisely** (`x += 1` instead of `x = x + 1`)
- **To accumulate values** in loops
- **To modify mutable objects in place** (e.g., `list += [1, 2]`)
- **To improve readability** in mathematical code
- **To avoid repeating the variable name** on both sides

### Syntax Rules and Structure

**Complete General Syntax**

```
variable operator= expression
```

**Operators**

| Operator | Equivalent |
|---|---|
| `+=` | `x = x + y` |
| `-=` | `x = x - y` |
| `*=` | `x = x * y` |
| `/=` | `x = x / y` |
| `//=` | `x = x // y` |
| `%=` | `x = x % y` |
| `**=` | `x = x ** y` |

**Syntax Rules**

| Rule | Description |
|---|---|
| Statement, not expression | Cannot be used in expressions (e.g., `y = (x += 1)` is invalid) |
| In-place for mutables | `list += [1]` modifies the list in place |
| Single evaluation | The left-hand operand is evaluated only once |
| All arithmetic operators | Each binary operator has an augmented form |

**Constraints and Limitations**

- **Not an expression**: Cannot be used in `print(x += 1)`.
- **Type restrictions**: The operation must be valid for the variable's type.
- **Rebinding vs. mutation**: For immutable objects, `+=` rebinds; for mutable objects, it may mutate.

### Multiple Annotated Complete Code Examples and Their Expected Outputs

**Example 1: Basic Augmented Assignment**

```python
# Step 1: Initialize
x = 10

# Step 2: Apply augmented operations
x += 5
print(f"After +=: {x}")
x -= 3
print(f"After -=: {x}")
x *= 2
print(f"After *=: {x}")
x //= 4
print(f"After //=: {x}")
```

**Expected Output:**
```
After +=: 15
After -=: 12
After *=: 24
After //=: 6
```

**Breakdown:** Each augmented operator applies the operation and updates the variable.

**Example 2: Accumulation in a Loop**

```python
# Step 1: Sum using +=
total = 0
for i in range(1, 6):
    total += i
print(f"Sum: {total}")

# Step 3: Product using *=
product = 1
for i in range(1, 6):
    product *= i
print(f"Product: {product}")
```

**Expected Output:**
```
Sum: 15
Product: 120
```

**Breakdown:** `+=` and `*=` are commonly used for accumulation.

**Example 3: Mutable vs. Immutable**

```python
# Step 1: Immutable (int) — rebinding
x = 10
print(f"Before: id={id(x)}")
x += 5
print(f"After:  id={id(x)}")  # Different id

# Step 2: Mutable (list) — in-place mutation
lst = [1, 2, 3]
print(f"Before: id={id(lst)}")
lst += [4, 5]
print(f"After:  id={id(lst)}")  # Same id (in-place)
```

**Expected Output:**
```
Before: id=140234...
After:  id=140235...  (different)
Before: id=140236...
After:  id=140236...  (same)
```

**Breakdown:** For immutables, `+=` rebinds to a new object. For mutables, `+=` modifies in place (same `id`).

### Real-World Cases with Explanation

**Case 1: Counters**

```python
word_count = 0
for word in text.split():
    word_count += 1
```

**Why it matters:** `+=` is the standard idiom for incrementing counters.

**Case 2: Building a List**

```python
results = []
for item in data:
    results += [transform(item)]
```

**Why it matters:** `+=` on lists extends them in place, which is more efficient than creating a new list each time.


## Core Concept 10: Common Pitfalls and Edge Cases

### Definitions

**Core Definition**
Common pitfalls and edge cases are behaviours of arithmetic operators that frequently surprise beginners or cause subtle bugs.

**Technical Definition**
Python's arithmetic operators have several behaviours that differ from other languages or from mathematical intuition: `/` always returns a float; `//` rounds toward negative infinity; `%` follows the divisor's sign; float arithmetic has precision limitations; and `**` has right-associativity and unusual precedence with unary minus. Understanding these edge cases is essential for writing correct code.

**Beginner-Friendly Explanation**
Some arithmetic behaviours in Python surprise people. For example, `0.1 + 0.2` doesn't equal `0.3` exactly (it's `0.30000000000000004`). `-1 // 3` is `-1`, not `0`. `-5 ** 2` is `-25`, not `25`. Knowing these quirks helps you avoid bugs.

### Key Pitfalls

| Pitfall | Example | Result | Explanation |
|---|---|---|---|
| Float precision | `0.1 + 0.2` | `0.30000000000000004` | IEEE 754 binary representation |
| Floor division negative | `-1 // 3` | `-1` | Rounds toward negative infinity |
| Modulus sign | `-4 % 3` | `2` | Sign follows divisor |
| Exponentiation precedence | `-5 ** 2` | `-25` | `**` binds tighter than unary `-` |
| Right-associative `**` | `2 ** 3 ** 2` | `512` | Groups from the right |
| Division always float | `4 / 2` | `2.0` | `/` always returns float |

### Multiple Annotated Complete Code Examples and Their Expected Outputs

**Example 1: Float Precision**

```python
# Step 1: Unexpected float result
result = 0.1 + 0.2
print(f"0.1 + 0.2 = {result}")
print(f"Equal to 0.3? {result == 0.3}")

# Step 2: Using math.isclose for safe comparison
import math
print(f"Close to 0.3? {math.isclose(result, 0.3)}")
```

**Expected Output:**
```
0.1 + 0.2 = 0.30000000000000004
Equal to 0.3? False
Close to 0.3? True
```

**Breakdown:** Floating-point arithmetic has precision limits. Use `math.isclose()` for comparisons.

**Example 2: Negative Floor Division and Modulus**

```python
# Step 1: Floor division with negatives
print(f"-1 // 3 = {-1 // 3}")   # -1
print(f"-4 // 3 = {-4 // 3}")   # -2

# Step 2: Modulus with negatives
print(f"-4 % 3 = {-4 % 3}")     # 2
print(f"4 % -3 = {4 % -3}")     # -2
```

**Expected Output:**
```
-1 // 3 = -1
-4 // 3 = -2
-4 % 3 = 2
4 % -3 = -2
```

**Breakdown:** Floor division rounds toward negative infinity. Modulus follows the divisor's sign.

**Example 3: Exponentiation Precedence**

```python
# Step 1: Unary minus and **
print(f"-3 ** 2 = {-3 ** 2}")    # -9, not 9
print(f"(-3) ** 2 = {(-3) ** 2}")  # 9

# Step 2: Right-associativity
print(f"2 ** 3 ** 2 = {2 ** 3 ** 2}")  # 512
print(f"(2 ** 3) ** 2 = {(2 ** 3) ** 2}")  # 64
```

**Expected Output:**
```
-3 ** 2 = -9
(-3) ** 2 = 9
2 ** 3 ** 2 = 512
(2 ** 3) ** 2 = 64
```

**Breakdown:** `**` binds tighter than unary `-` and is right-associative.

### Real-World Cases with Explanation

**Case 1: Financial Calculations**

```python
# Use Decimal for precise financial calculations
from decimal import Decimal
price = Decimal("0.1") + Decimal("0.2")
print(price)  # 0.3 (exact)
```

**Why it matters:** Float precision issues are unacceptable in financial applications. Use `Decimal` for exact decimal arithmetic.

**Case 2: Index Wrapping**

```python
# Safe circular indexing
index = (current_index + 1) % len(items)
```

**Why it matters:** Modulus is the standard way to wrap indices in a circular buffer or carousel.


## References

- Python Software Foundation. *Expressions — Python Language Reference*. https://docs.python.org/3/reference/expressions.html
- Python Software Foundation. *Operator precedence*. https://docs.python.org/3/reference/expressions.html#operator-precedence
- Python Software Foundation. *Numeric Types — int, float, complex*. https://docs.python.org/3/library/stdtypes.html#numeric-types-int-float-complex
- Python Software Foundation. *PEP 238 – Changing the Division Operator*. https://peps.python.org/pep-0238/
- Microsoft Learn. *Arithmetic and numeric types in Python*. https://learn.microsoft.com/en-us/training/modules/python-data-science/2-types-arithmetic-numeric
- Microsoft Learn. *Perform operations with numbers*. https://learn.microsoft.com/en-us/training/modules/work-with-data-python/3-perform-operations-with-numbers
- Carnegie Mellon University. *Class Notes: Data Types and Operators*. https://www.cs.cmu.edu/~112-f22/notes/notes-data-and-operations.html
- Python Software Foundation. *decimal — Decimal fixed-point and floating-point arithmetic*. https://docs.python.org/3/library/decimal.html
- Python Software Foundation. *math — Mathematical functions*. https://docs.python.org/3/library/math.html
- Real Python. *Python Operators: Arithmetic, Comparison, and More*. https://realpython.com/python-operators/