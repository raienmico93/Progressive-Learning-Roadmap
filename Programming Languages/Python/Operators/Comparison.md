# Python Comparison (Relational) Operators: A Comprehensive Programming Cheat Sheet

## Topic Overview

### Definitions

**Core Definition**
Comparison operators are symbols that compare two values and return a Boolean result (`True` or `False`).

**Technical Definition**
Python's comparison operators (`==`, `!=`, `>`, `<`, `>=`, `<=`) are defined by the language's expression grammar and implemented through rich comparison methods (`__eq__()`, `__ne__()`, `__lt__()`, `__gt__()`, `__le__()`, `__ge__()`). All eight comparison operations in Python have the same priority, which is higher than that of Boolean operations. Comparisons can be chained arbitrarily, and objects of different types can be compared, though unless stated otherwise, objects of different types never compare equal.

**Beginner-Friendly Explanation**
Comparison operators are like asking questions: "Are these two things equal?" or "Is this bigger than that?" Python answers with `True` or `False`. You use them everywhere — in `if` statements, loops, and anywhere you need to make a decision based on how two values relate.

### Key Characteristics

| Characteristic | Description |
|---|---|
| **Boolean Result** | Every comparison evaluates to `True` or `False` |
| **Same Precedence** | All comparison operators have equal priority |
| **Chaining** | `a < b < c` is valid and means `a < b and b < c` |
| **Type Flexibility** | Equality works across types; ordering may raise `TypeError` |
| **Rich Comparison** | Classes can customise behaviour via `__eq__`, `__lt__`, etc. |
| **Identity vs. Equality** | `is` tests identity; `==` tests value equality |

### Prerequisites

- **Basic Python syntax**: variables, expressions, and `print()`
- **Numeric and string types**: understanding of `int`, `float`, and `str`
- **Boolean logic**: familiarity with `True` and `False`
- **Conditional statements**: `if`/`else` for using comparison results

### Related Programming Areas

- **Boolean Operators**: `and`, `or`, `not` for combining comparisons
- **Identity Operators**: `is` and `is not` for object identity
- **Membership Operators**: `in` and `not in` for containment tests
- **Operator Precedence**: How comparisons interact with arithmetic
- **Rich Comparison Methods**: Customising comparison behaviour in classes

### Core Concepts / Features

1. **`==`** (Equal to)
2. **`!=`** (Not equal to)
3. **`>`** (Greater than)
4. **`<`** (Less than)
5. **`>=`** (Greater than or equal to)
6. **`<=`** (Less than or equal to)
7. **Chained Comparisons** (Multiple comparisons in one expression)
8. **Common Pitfalls** (Identity vs. equality, type errors, float precision)


## Core Concept 1: `==` (Equal to)

### Definitions

**Core Definition**
The `==` operator tests whether two objects have equal values, returning `True` if they are equal and `False` otherwise.

**Technical Definition**
The `==` operator invokes the left operand's `__eq__()` method, falling back to the right operand's `__eq__()` if the left returns `NotImplemented`. The default implementation in `object` compares identity, but built-in types override this to compare values. For numeric types, `0 == 0.0` is `True` because Python compares them mathematically. For sequences, comparison is lexicographic. The operator is always defined; for some object types (like class objects), it may be equivalent to identity comparison.

**Beginner-Friendly Explanation**
The `==` operator asks "are these two things the same?" For example, `2 + 2 == 4` is `True`. But `==` is not the same as `=` — `=` assigns a value, while `==` compares values. A common beginner mistake is writing `if x = 5` instead of `if x == 5`.

### Purposes

- **To test if two values are equal** in conditional statements
- **To compare numeric results** from calculations
- **To check if strings or sequences match** expected values
- **To verify data integrity** in tests and assertions
- **To implement equality logic** in custom classes

### Syntax Rules and Structure

**Complete General Syntax**

```
operand1 == operand2
```

**Breakdown:**
- `operand1`, `operand2`: Any Python objects.
- Returns `True` if equal, `False` otherwise.

**Syntax Rules**

| Rule | Description |
|---|---|
| Any types | Works with numbers, strings, sequences, objects |
| Cross-type | `0 == 0.0` is `True`; `"2" == 2` is `False` |
| Chaining | `a == b == c` means `a == b and b == c` |
| Customisable | Classes can define `__eq__()` |
| Precedence | Same as all comparison operators |

**Constraints and Limitations**

- **Float precision**: `0.1 + 0.2 == 0.3` is `False` due to binary floating-point representation.
- **Identity vs. equality**: `==` checks value, not memory location.
- **`__eq__` and `__hash__`**: If you override `__eq__`, you should also override `__hash__` for hashable objects.

### Multiple Annotated Complete Code Examples and Their Expected Outputs

**Example 1: Basic Equality**

```python
# Step 1: Numeric equality
print(2 + 2 == 4)      # True
print(10 == 10.0)      # True (int and float compare mathematically)

# Step 2: String equality
print("hello" == "hello")   # True
print("Hello" == "hello")   # False (case-sensitive)

# Step 3: List equality
print([1, 2, 3] == [1, 2, 3])   # True
print([1, 2, 3] == [3, 2, 1])   # False (order matters)
```

**Expected Output:**
```
True
True
True
False
True
False
```

**Breakdown:** `==` compares values. `10 == 10.0` is `True` because Python compares them numerically. String comparison is case-sensitive. List comparison checks element-by-element.

**Example 2: Cross-Type Equality**

```python
# Step 1: Different types
print("2" == 2)        # False (string vs. int)
print(True == 1)       # True (bool is a subclass of int)
print(False == 0)      # True

# Step 2: None comparison
x = None
print(x == None)       # True (but PEP 8 recommends `is None`)
print(x is None)       # True (preferred)
```

**Expected Output:**
```
False
True
True
True
True
```

**Breakdown:** Strings and integers never compare equal. Booleans are integers in Python, so `True == 1` is `True`. Use `is None` instead of `== None` per PEP 8.

**Example 3: Custom `__eq__`**

```python
class Point:
    def __init__(self, x, y):
        self.x = x
        self.y = y

    def __eq__(self, other):
        if not isinstance(other, Point):
            return NotImplemented
        return self.x == other.x and self.y == other.y

p1 = Point(1, 2)
p2 = Point(1, 2)
p3 = Point(3, 4)

print(p1 == p2)   # True
print(p1 == p3)   # False
print(p1 == "not a point")  # False
```

**Expected Output:**
```
True
False
False
```

**Breakdown:** The custom `__eq__` method defines what it means for two `Point` objects to be equal. Returning `NotImplemented` for unsupported types lets Python fall back gracefully.

### Real-World Cases with Explanation

**Case 1: Login Validation**

```python
stored_password = "secret123"
entered_password = input("Enter password: ")
if entered_password == stored_password:
    print("Access granted")
else:
    print("Access denied")
```

**Why it matters:** Equality comparison is fundamental for validating user input against stored values.

**Case 2: Data Verification**

```python
expected = [1, 2, 3, 4, 5]
actual = get_data_from_api()
if actual == expected:
    print("Data matches")
else:
    print("Data mismatch")
```

**Why it matters:** Comparing sequences verifies that data from external sources matches expectations.


## Core Concept 2: `!=` (Not equal to)

### Definitions

**Core Definition**
The `!=` operator tests whether two objects have different values, returning `True` if they are not equal and `False` if they are equal.

**Technical Definition**
The `!=` operator invokes `__ne__()`, which by default delegates to the negation of `__eq__()` unless explicitly overridden. The forms `<>` and `!=` were historically equivalent, but `<>` is obsolescent and removed in Python 3; `!=` is preferred for consistency with C. For numeric types, `0 != 0.0` is `False` because they compare equal. For sequences, inequality follows the same lexicographic rules as equality.

**Beginner-Friendly Explanation**
The `!=` operator is the opposite of `==`. It asks "are these two things different?" For example, `5 != 3` is `True`, and `5 != 5` is `False`. It's useful when you want to check that something is not a particular value.

### Purposes

- **To test if two values are different** in conditional statements
- **To filter out unwanted values** in loops and comprehensions
- **To validate that input has changed** from a default
- **To implement inequality logic** in custom classes
- **To check that a value is not `None`** (though `is not` is preferred)

### Syntax Rules and Structure

**Complete General Syntax**

```
operand1 != operand2
```

**Breakdown:**
- Returns `True` if operands are not equal, `False` otherwise.

**Syntax Rules**

| Rule | Description |
|---|---|
| Inverse of `==` | `a != b` is equivalent to `not (a == b)` |
| Any types | Works with all comparable types |
| Cross-type | Usually `True` for different types |
| Chaining | `a != b != c` means `a != b and b != c` |
| Customisable | Classes can define `__ne__()` |

**Constraints and Limitations**

- **`<>` is deprecated**: Use `!=` instead.
- **`!= None` vs. `is not None`**: PEP 8 recommends `is not` for `None` comparisons.
- **Float precision**: Same issues as `==`.

### Multiple Annotated Complete Code Examples and Their Expected Outputs

**Example 1: Basic Inequality**

```python
# Step 1: Numeric inequality
print(5 != 3)      # True
print(5 != 5)      # False
print(10 != 10.0)  # False (they compare equal)

# Step 2: String inequality
print("hello" != "world")   # True
print("hello" != "hello")   # False
```

**Expected Output:**
```
True
False
False
True
False
```

**Breakdown:** `!=` returns `True` when values differ. `10 != 10.0` is `False` because they compare equal numerically.

**Example 2: Filtering with `!=`**

```python
# Step 1: Filter out a specific value
values = [1, 2, 3, 2, 4, 2, 5]
filtered = [v for v in values if v != 2]
print(filtered)
```

**Expected Output:**
```
[1, 3, 4, 5]
```

**Breakdown:** The list comprehension keeps only values not equal to `2`.

**Example 3: Checking Input Changed**

```python
default_name = "Guest"
user_name = input("Enter name: ")
if user_name != default_name:
    print(f"Welcome, {user_name}!")
else:
    print("Using default name.")
```

**Expected Output (if user enters "Alice"):**
```
Welcome, Alice!
```

**Breakdown:** `!=` checks whether the user provided a name different from the default.

### Real-World Cases with Explanation

**Case 1: Avoiding Duplicates**

```python
seen = set()
for item in items:
    if item not in seen:
        process(item)
        seen.add(item)
```

**Why it matters:** While `not in` uses membership, `!=` is often used in similar filtering logic.

**Case 2: Sentinel Value Detection**

```python
value = get_input()
while value != "quit":
    process(value)
    value = get_input()
```

**Why it matters:** `!=` is the standard way to check for a sentinel value in loops.


## Core Concept 3: `>` (Greater than)

### Definitions

**Core Definition**
The `>` operator tests whether the left operand is strictly greater than the right operand.

**Technical Definition**
The `>` operator invokes `__gt__()`. For numeric types, it performs mathematical comparison. For strings, it compares lexicographically by Unicode code points. For sequences, it compares element-by-element. A default order comparison is not provided by `object`, so custom classes must implement `__gt__()` or inherit from a class that does; otherwise, `TypeError` is raised.

**Beginner-Friendly Explanation**
The `>` operator asks "is the left number bigger than the right one?" For example, `10 > 3` is `True`. You can also compare strings alphabetically: `"banana" > "apple"` is `True`.

### Purposes

- **To compare numeric magnitudes** in calculations and conditions
- **To sort data** in ascending or descending order
- **To implement threshold checks** (e.g., "is the temperature above 30?")
- **To compare strings alphabetically** for ordering
- **To drive decision logic** in algorithms

### Syntax Rules and Structure

**Complete General Syntax**

```
operand1 > operand2
```

**Syntax Rules**

| Rule | Description |
|---|---|
| Numeric comparison | Mathematical ordering |
| String comparison | Lexicographic (Unicode code points) |
| Sequence comparison | Element-by-element |
| Chaining | `a > b > c` means `a > b and b > c` |
| No default for custom classes | Raises `TypeError` unless `__gt__` is defined |

**Constraints and Limitations**

- **Cannot compare incompatible types**: `5 > "abc"` raises `TypeError`.
- **Complex numbers**: Do not support ordering.
- **NaN**: Any ordered comparison with `NaN` is `False`.

### Multiple Annotated Complete Code Examples and Their Expected Outputs

**Example 1: Numeric Comparison**

```python
print(10 > 3)      # True
print(3 > 10)      # False
print(5 > 5)       # False (strictly greater)
print(5.5 > 5)     # True
```

**Expected Output:**
```
True
False
False
True
```

**Breakdown:** `>` is strict: equal values return `False`.

**Example 2: String Comparison**

```python
print("banana" > "apple")   # True (b > a)
print("apple" > "banana")   # False
print("abc" > "ab")         # True (longer string with same prefix)
```

**Expected Output:**
```
True
False
True
```

**Breakdown:** Strings compare lexicographically by Unicode code points.

**Example 3: Threshold Check**

```python
temperature = 35
if temperature > 30:
    print("It's hot!")
else:
    print("It's comfortable.")
```

**Expected Output:**
```
It's hot!
```

**Breakdown:** `>` is used to check if a value exceeds a threshold.

### Real-World Cases with Explanation

**Case 1: Grade Assignment**

```python
score = 85
if score > 90:
    grade = "A"
elif score > 80:
    grade = "B"
else:
    grade = "C"
```

**Why it matters:** Threshold comparisons drive classification logic.

**Case 2: Sorting**

```python
numbers = [5, 2, 8, 1]
numbers.sort(reverse=True)  # Sorts using > internally
print(numbers)
```

**Why it matters:** The `>` operator underlies sorting algorithms.


## Core Concept 4: `<` (Less than)

### Definitions

**Core Definition**
The `<` operator tests whether the left operand is strictly less than the right operand.

**Technical Definition**
The `<` operator invokes `__lt__()`. It behaves analogously to `>` but with reversed ordering. For strings, lexicographic comparison applies. Custom classes must implement `__lt__()` for ordering, or `TypeError` is raised.

**Beginner-Friendly Explanation**
The `<` operator asks "is the left number smaller than the right one?" For example, `3 < 10` is `True`. It's the opposite of `>`.

### Purposes

- **To compare numeric magnitudes** (smaller than)
- **To sort data** in ascending order
- **To check lower bounds** (e.g., "is the value below the minimum?")
- **To compare strings alphabetically**
- **To drive loop conditions** (e.g., `while i < n`)

### Syntax Rules and Structure

**Complete General Syntax**

```
operand1 < operand2
```

**Syntax Rules**

| Rule | Description |
|---|---|
| Numeric comparison | Mathematical ordering |
| String comparison | Lexicographic |
| Chaining | `a < b < c` means `a < b and b < c` |
| No default for custom classes | Requires `__lt__()` |

**Constraints and Limitations**

- **Incompatible types**: `5 < "abc"` raises `TypeError`.
- **Complex numbers**: Not orderable.
- **NaN**: Any ordered comparison with `NaN` is `False`.

### Multiple Annotated Complete Code Examples and Their Expected Outputs

**Example 1: Basic Less Than**

```python
print(3 < 10)      # True
print(10 < 3)      # False
print(5 < 5)       # False (strictly less)
print(2.5 < 3)     # True
```

**Expected Output:**
```
True
False
False
True
```

**Breakdown:** `<` is strict: equal values return `False`.

**Example 2: Loop Condition**

```python
i = 0
while i < 5:
    print(i, end=" ")
    i += 1
```

**Expected Output:**
```
0 1 2 3 4 
```

**Breakdown:** `<` is commonly used in loop conditions.

**Example 3: Range Check**

```python
age = 25
if 18 <= age < 65:
    print("Working age")
```

**Expected Output:**
```
Working age
```

**Breakdown:** Chained comparison combines `<=` and `<`.

### Real-World Cases with Explanation

**Case 1: Binary Search**

```python
def binary_search(arr, target):
    low, high = 0, len(arr) - 1
    while low <= high:
        mid = (low + high) // 2
        if arr[mid] < target:
            low = mid + 1
        elif arr[mid] > target:
            high = mid - 1
        else:
            return mid
    return -1
```

**Why it matters:** `<` and `>` are essential for binary search comparisons.

**Case 2: Validating Bounds**

```python
if 0 < x < 100:
    print("x is in range")
```

**Why it matters:** Chained `<` checks both bounds in one expression.


## Core Concept 5: `>=` (Greater than or equal to)

### Definitions

**Core Definition**
The `>=` operator tests whether the left operand is greater than or equal to the right operand.

**Technical Definition**
The `>=` operator invokes `__ge__()`. It is the non-strict version of `>`, returning `True` when operands are equal or the left is greater. Custom classes must implement `__ge__()` for ordering.

**Beginner-Friendly Explanation**
The `>=` operator asks "is the left number bigger than or equal to the right?" For example, `10 >= 10` is `True`, and `10 >= 5` is `True`. It's inclusive.

### Purposes

- **To include equality in greater-than checks** (e.g., "is the score at least 60?")
- **To set minimum thresholds** in validation
- **To implement inclusive bounds** in algorithms
- **To compare versions** or dates inclusively

### Syntax Rules and Structure

**Complete General Syntax**

```
operand1 >= operand2
```

**Syntax Rules**

| Rule | Description |
|---|---|
| Inclusive | Returns `True` when equal |
| Chaining | `a >= b >= c` means `a >= b and b >= c` |
| Customisable | Requires `__ge__()` for custom classes |

**Constraints and Limitations**

- **Same type restrictions** as `>`.
- **NaN comparisons** are `False`.

### Multiple Annotated Complete Code Examples and Their Expected Outputs

**Example 1: Inclusive Comparison**

```python
print(10 >= 10)   # True
print(10 >= 5)    # True
print(5 >= 10)    # False
```

**Expected Output:**
```
True
True
False
```

**Breakdown:** `>=` returns `True` when values are equal or left is greater.

**Example 2: Minimum Threshold**

```python
score = 60
if score >= 60:
    print("Pass")
else:
    print("Fail")
```

**Expected Output:**
```
Pass
```

**Breakdown:** `>=` includes the boundary value.

### Real-World Cases with Explanation

**Case 1: Age Verification**

```python
if age >= 18:
    print("Eligible to vote")
```

**Why it matters:** `>=` includes the boundary age of 18.

**Case 2: Version Comparison**

```python
if current_version >= required_version:
    print("Compatible")
```

**Why it matters:** `>=` ensures compatibility with minimum required versions.


## Core Concept 6: `<=` (Less than or equal to)

### Definitions

**Core Definition**
The `<=` operator tests whether the left operand is less than or equal to the right operand.

**Technical Definition**
The `<=` operator invokes `__le__()`. It is the non-strict version of `<`, returning `True` when operands are equal or the left is less.

**Beginner-Friendly Explanation**
The `<=` operator asks "is the left number smaller than or equal to the right?" For example, `5 <= 10` is `True`, and `10 <= 10` is `True`.

### Purposes

- **To include equality in less-than checks** (e.g., "is the count at most 100?")
- **To set maximum thresholds** in validation
- **To implement inclusive upper bounds**
- **To drive loops with inclusive conditions**

### Syntax Rules and Structure

**Complete General Syntax**

```
operand1 <= operand2
```

**Syntax Rules**

| Rule | Description |
|---|---|
| Inclusive | Returns `True` when equal |
| Chaining | `a <= b <= c` means `a <= b and b <= c` |
| Customisable | Requires `__le__()` for custom classes |

**Constraints and Limitations**

- **Same type restrictions** as `<`.
- **NaN comparisons** are `False`.

### Multiple Annotated Complete Code Examples and Their Expected Outputs

**Example 1: Inclusive Less Than**

```python
print(5 <= 10)    # True
print(10 <= 10)   # True
print(10 <= 5)    # False
```

**Expected Output:**
```
True
True
False
```

**Breakdown:** `<=` returns `True` when values are equal or left is less.

**Example 2: Maximum Capacity**

```python
items = 100
max_capacity = 100
if items <= max_capacity:
    print("Capacity OK")
```

**Expected Output:**
```
Capacity OK
```

**Breakdown:** `<=` includes the maximum capacity.

### Real-World Cases with Explanation

**Case 1: Speed Limit**

```python
if speed <= 65:
    print("Within speed limit")
else:
    print("Speeding")
```

**Why it matters:** `<=` includes the exact speed limit.

**Case 2: Count Validation**

```python
if len(items) <= max_items:
    process(items)
```

**Why it matters:** `<=` ensures the count does not exceed the maximum.


## Core Concept 7: Chained Comparisons

### Definitions

**Core Definition**
Chained comparisons allow multiple comparison operators to be used in a single expression, with the semantics of a conjunction (logical AND) between each pair.

**Technical Definition**
Comparisons can be chained arbitrarily, e.g., `x < y <= z` is equivalent to `x < y and y <= z`, except that `y` is evaluated only once (but in both cases `z` is not evaluated at all when `x < y` is found to be false). Formally, `a op1 b op2 c ... y opN z` is equivalent to `a op1 b and b op2 c and ... y opN z`, with each expression evaluated at most once.

**Beginner-Friendly Explanation**
Chaining lets you write comparisons the way you would in mathematics: `0 < x < 10` checks that `x` is between 0 and 10. Python evaluates it as `0 < x and x < 10`, but `x` is only evaluated once. This is cleaner than writing two separate comparisons.

### Purposes

- **To express range checks concisely** (`0 < x < 100`)
- **To avoid repeating the middle expression**
- **To improve readability** for mathematical-style comparisons
- **To ensure the middle expression is evaluated only once**

### Syntax Rules and Structure

**Complete General Syntax**

```
expr1 op1 expr2 op2 expr3 ...
```

**Breakdown:**
- Each expression is evaluated at most once.
- Equivalent to a chain of `and` operations.
- Short-circuits: if an earlier comparison is false, later expressions are not evaluated.

**Syntax Rules**

| Rule | Description |
|---|---|
| Any comparison operators | `<`, `>`, `==`, `>=`, `<=`, `!=` can be mixed |
| Left-to-right | Evaluated left to right |
| Single evaluation | The middle expression is evaluated once |
| Short-circuit | If `a < b` is false, `b < c` is not evaluated |
| No `and`/`or` | Chain uses implicit `and` |

**Constraints and Limitations**

- **No `or` chaining**: `a < b or b < c` is not the same as `a < b < c`.
- **Readability**: Long chains can be hard to read; use parentheses or separate comparisons when unclear.

### Multiple Annotated Complete Code Examples and Their Expected Outputs

**Example 1: Range Check**

```python
x = 5
print(0 < x < 10)   # True (x is between 0 and 10)
print(0 < x < 3)    # False (x is not less than 3)
```

**Expected Output:**
```
True
False
```

**Breakdown:** `0 < x < 10` is equivalent to `0 < x and x < 10`.

**Example 2: Mixed Operators**

```python
a, b, c = 3, 5, 7

print(a < b < c)    # True
print(a < b > c)    # False (b > c is false)
print(a <= b == 5)  # True (b == 5)
```

**Expected Output:**
```
True
False
True
```

**Breakdown:** Chaining works with any mix of comparison operators.

**Example 3: Short-Circuit Behaviour**

```python
def check():
    print("check called")
    return 5

x = 10
print(x < 0 < check())  # check() is NOT called
```

**Expected Output:**
```
False
```

**Breakdown:** Because `x < 0` is `False`, the rest of the chain is not evaluated, so `check()` is never called.

### Real-World Cases with Explanation

**Case 1: Age Range Validation**

```python
if 18 <= age <= 65:
    print("Working age")
```

**Why it matters:** Chained comparisons cleanly express range checks.

**Case 2: Temperature Monitoring**

```python
if 20 <= temperature <= 25:
    print("Comfortable")
```

**Why it matters:** Range validation with chained comparisons is readable and efficient.


## Core Concept 8: Common Pitfalls

### Definitions

**Core Definition**
Common pitfalls are frequent mistakes or misunderstandings when using comparison operators in Python.

**Technical Definition**
Pitfalls arise from Python's distinction between identity and equality, floating-point precision limitations, cross-type comparison rules, and the behaviour of `NaN`. The most frequent bug is using `is` instead of `==` for value comparison. Float precision issues cause `0.1 + 0.2 == 0.3` to be `False`. `NaN` is never equal to itself.

**Beginner-Friendly Explanation**
Some comparison behaviours surprise beginners. `0.1 + 0.2` doesn't equal `0.3` exactly. `is` and `==` are different. `NaN` is not equal to itself. Knowing these traps helps you write correct code.

### Key Pitfalls Table

| Pitfall | Example | Result | Explanation |
|---|---|---|---|
| `is` vs. `==` | `[1] is [1]` | `False` | Different objects, same value |
| Float precision | `0.1 + 0.2 == 0.3` | `False` | IEEE 754 representation |
| `NaN` inequality | `float('nan') == float('nan')` | `False` | NaN is not equal to itself |
| Cross-type ordering | `5 > "3"` | `TypeError` | Cannot order incompatible types |
| `==` with `None` | `x == None` | Works but discouraged | PEP 8 prefers `is None` |
| Chaining confusion | `x < y > z` | Legal but confusing | No comparison between x and z |

### Multiple Annotated Complete Code Examples and Their Expected Outputs

**Example 1: Identity vs. Equality**

```python
a = [1, 2, 3]
b = [1, 2, 3]

print(a == b)   # True (same value)
print(a is b)   # False (different objects)

c = a
print(a is c)   # True (same object)
```

**Expected Output:**
```
True
False
True
```

**Breakdown:** `==` checks value equality; `is` checks identity.

**Example 2: Float Precision**

```python
import math

print(0.1 + 0.2)                # 0.30000000000000004
print(0.1 + 0.2 == 0.3)         # False
print(math.isclose(0.1 + 0.2, 0.3))  # True
```

**Expected Output:**
```
0.30000000000000004
False
True
```

**Breakdown:** Float arithmetic has precision limits. Use `math.isclose()` for safe comparison.

**Example 3: NaN Behaviour**

```python
nan = float('nan')
print(nan == nan)   # False
print(nan != nan)   # True
print(3 < nan)      # False
print(nan < 3)      # False
```

**Expected Output:**
```
False
True
False
False
```

**Breakdown:** `NaN` is not equal to itself and any ordered comparison with it is `False`.

### Real-World Cases with Explanation

**Case 1: Financial Calculations**

```python
from decimal import Decimal
price = Decimal("0.1") + Decimal("0.2")
print(price == Decimal("0.3"))  # True
```

**Why it matters:** Use `Decimal` for exact decimal arithmetic in financial applications.

**Case 2: Safe `None` Checks**

```python
value = get_value()
if value is None:       # Preferred
    handle_missing()
elif value == 0:        # Correct for numbers
    handle_zero()
```

**Why it matters:** Use `is None` for identity checks and `==` for value checks.


## References

- Python Software Foundation. *Expressions — Python Language Reference (Comparisons)*. https://docs.python.org/3/reference/expressions.html#comparisons
- Python Software Foundation. *Built-in Types — Comparisons*. https://docs.python.org/3/library/stdtypes.html#comparisons
- Python Software Foundation. *Operator precedence — Python Language Reference*. https://docs.python.org/3/reference/expressions.html#operator-precedence
- Real Python. *Comparison and Membership Operators*. https://realpython.com/lessons/comparison-membership/
- Real Python. *Python != Is Not is not: Comparing Objects in Python*. https://realpython.com/python-is-identity-vs-equality/
- Python Software Foundation. *PEP 8 – Style Guide for Python Code (Programming Recommendations)*. https://peps.python.org/pep-0008/#programming-recommendations
- Python Software Foundation. *decimal — Decimal fixed-point and floating-point arithmetic*. https://docs.python.org/3/library/decimal.html
- Python Software Foundation. *math.isclose()*. https://docs.python.org/3/library/math.html#math.isclose
- Python Tutor Mailing List. *Chained Comparisons*. https://mail.python.org/pipermail/tutor/2014-March/100492.html
- Python Bug Tracker. *Documentation should warn that 'is' is not a safe comparison operator*. https://mail.python.org/pipermail/python-bugs-list/2021-February/464425.html