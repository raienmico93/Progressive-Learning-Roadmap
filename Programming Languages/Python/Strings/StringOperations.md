# Python String Operations: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Core Definition

String operations are the fundamental sequence operations that Python strings support as immutable sequences of Unicode code points. These operations include indexing, slicing, concatenation, repetition, membership testing, and comparison, and they form the basis for all text manipulation in Python.

### Technical Definition

Python strings are immutable sequences of Unicode code points that support the full set of common sequence operations defined for all sequence types (list, tuple, range, str, bytes, bytearray). These operations are implemented through the `__getitem__` and `__len__` methods for indexing and slicing, the `__add__` method for concatenation, the `__mul__` method for repetition, the `__contains__` method for membership testing, and the rich comparison methods (`__eq__`, `__lt__`, etc.) for lexicographic comparison. The slicing operation additionally uses slice objects, which are created automatically when the `s[i:j:k]` syntax is used.

### Beginner-Friendly Explanation

Strings in Python come with a set of built-in operations that let you work with text. You can grab individual characters (indexing), extract chunks of text (slicing), join strings together (concatenation), repeat strings (repetition), check if one string is inside another (membership), and compare strings alphabetically (comparison). These operations are the basic building blocks for everything from simple text processing to complex data pipelines.

### Key Characteristics

- **Immutable**: All string operations return new strings; the original string is never modified.
- **Sequence-based**: Strings support the same operations as other sequences (lists, tuples).
- **Zero-indexed**: The first character is at index 0; negative indices count from the end.
- **Unicode-native**: Comparison and indexing operate on Unicode code points.
- **Efficient**: Indexing is O(1); slicing is O(k); membership is O(n) in the worst case.

### Prerequisites

- Python 3.x installed.
- Basic understanding of string literals and variables.
- A text editor or IDE.

### Related Programming Areas

- **Text processing**: Parsing, cleaning, and transforming text.
- **Data validation**: Checking input formats and contents.
- **Regular expressions**: Pattern matching uses slicing and membership concepts.
- **Algorithms**: String manipulation is central to many algorithms.

### Core Concepts / Features

The following sections cover each string operation using a uniform structure.

---

## 1. Indexing

### Definitions

**Core Definition**: Indexing is the operation of extracting a single character from a string using its position, specified as an integer index.

**Technical Definition**: Indexing a string `s` with an integer `i` returns the character at position `i`. Python uses zero-based indexing, so `s[0]` is the first character. Negative indices count from the end, with `s[-1]` being the last character. The index must be an integer; if it is out of range, an `IndexError` is raised. If the index is not an integer, a `TypeError` is raised.

**Beginner-Friendly Explanation**: Indexing lets you grab a single character from a string by its position. The first character is at position 0, the second at position 1, and so on. You can also count from the end using negative numbers: `-1` is the last character, `-2` is the second-to-last, etc.

### Purposes

- To extract a specific character from a string by its position.
- To access the first or last character of a string.
- To iterate over a string character by character using indices.
- To validate that a string has a certain character at a certain position.
- To implement algorithms that require positional access to characters.

### Syntax Rules and Structure

#### Complete General Syntax

```python
s[i]      # Character at index i
s[-i]     # Character at index len(s) - i
```

**Component Breakdown**:
- `s` — the string being indexed.
- `[` and `]` — the indexing operators.
- `i` — an integer index; positive counts from the start, negative from the end.

#### Syntax Rules

1. **Zero-based indexing**: The first character is at index 0.
2. **Negative indexing**: `s[-1]` is the last character; `s[-len(s)]` is the first character.
3. **Index must be an integer**: `s[1.0]` raises `TypeError`.
4. **Out-of-range indices raise `IndexError`**: `s[100]` on a short string raises `IndexError`.
5. **No separate character type**: Indexing a string returns a string of length 1; `s[0] == s[0:1]`.
6. **`-0` is `0`**: There is no distinct `-0` index; it refers to the first character.

#### Constraints and Limitations

- Strings are immutable; indexed assignment raises `TypeError`.
- Index must be an integer, not a float or string.
- Out-of-range indices raise `IndexError`.

### Multiple Annotated Complete Step by Step Code Examples and Their Expected Outputs

#### Example 1: Basic Indexing

```python
# step1: Create a string
s = "Python"

# step2: Positive indexing (zero-based)
print(s[0])   # P
print(s[1])   # y
print(s[5])   # n

# step3: Negative indexing
print(s[-1])  # n
print(s[-2])  # o
print(s[-6])  # P

# step4: Length check
print(len(s))  # 6
```

**Expected Output**:
```
P
y
n
n
o
P
6
```

**Why**: Positive indices count from 0; negative indices count from -1 (the last character). `s[-6]` is equivalent to `s[0]`.

#### Example 2: Handling `IndexError`

```python
s = "abc"

# step1: Valid index
print(s[2])  # c

# step2: Out-of-range index
try:
    print(s[5])
except IndexError as e:
    print(f"IndexError: {e}")  # string index out of range

# step3: Negative out-of-range index
try:
    print(s[-5])
except IndexError as e:
    print(f"IndexError: {e}")  # string index out of range
```

**Expected Output**:
```
c
IndexError: string index out of range
IndexError: string index out of range
```

**Why**: Accessing an index beyond the string's length (positive or negative) raises `IndexError`.

#### Example 3: `TypeError` for Non-Integer Index

```python
s = "hello"

# step1: Attempt to use a float index
try:
    print(s[1.5])
except TypeError as e:
    print(f"TypeError: {e}")  # string indices must be integers

# step2: Attempt to use a string index
try:
    print(s["a"])
except TypeError as e:
    print(f"TypeError: {e}")  # string indices must be integers
```

**Expected Output**:
```
TypeError: string indices must be integers
TypeError: string indices must be integers
```

**Why**: Python requires integer indices; floats and strings raise `TypeError`.

### Real-World Cases

- **First/last character checks**: Validating that a string starts with a specific character.
- **Character-by-character processing**: Iterating with `for i in range(len(s))`.
- **Tokenization**: Extracting the first character for classification.
- **Checksum algorithms**: Accessing characters at specific positions.

### References

- Sequence Types — list, tuple, range - https://docs.python.org/3/library/stdtypes.html#sequence-types-list-tuple-range
- Indexing - https://docs.python.org/3/reference/expressions.html#subscriptions

---

## 2. Slicing

### Definitions

**Core Definition**: Slicing is the operation of extracting a sub-sequence (substring) from a string using the syntax `s[start:stop:step]`.

**Technical Definition**: The slice of string `s` from `i` to `j` with step `k` is defined as the sequence of items with index `k` such that `i <= k < j`, where indices are normalized to the range `[0, len(s))`. If `i` is omitted or `None`, it defaults to 0 (or `len(s) - 1` for negative steps). If `j` is omitted or `None`, it defaults to `len(s)` (or `-len(s) - 1` for negative steps). If `i` or `j` is negative, the index is relative to the end of the string: `len(s) + i` or `len(s) + j`. If the slice is out of range, it is silently truncated. The result is always a new string.

**Beginner-Friendly Explanation**: Slicing lets you grab a chunk of a string. `s[2:5]` gives you the characters from index 2 up to (but not including) index 5. You can use negative indices, leave out the start or stop, and even add a step to skip characters. `s[::-1]` reverses the string.

### Purposes

- To extract a substring from a string.
- To reverse a string using `[::-1]`.
- To extract every nth character using a step.
- To split a string into parts programmatically.
- To create a copy of a string.

### Syntax Rules and Structure

#### Complete General Syntax

```python
s[i:j]        # From i to j (exclusive)
s[i:j:k]      # From i to j with step k
s[:j]         # From start to j
s[i:]         # From i to end
s[:]          # Full copy
s[::k]        # Every k-th character
s[::-1]       # Reversed string
```

**Component Breakdown**:
- `i` — start index (inclusive); default 0.
- `j` — stop index (exclusive); default `len(s)`.
- `k` — step (stride); default 1; cannot be 0.

#### Syntax Rules

1. **Stop is exclusive**: `s[2:5]` includes indices 2, 3, 4, not 5.
2. **Defaults**: `i` defaults to 0, `j` defaults to `len(s)`, `k` defaults to 1.
3. **Negative indices**: `s[-3:]` gives the last three characters.
4. **Negative step**: `s[::-1]` reverses the string; `s[5:2:-1]` goes backward.
5. **Out-of-range is safe**: Slices are truncated, not raised as errors.
6. **Empty slice**: `s[3:3]` returns `''`.
7. **`slice()` objects**: `s[slice(2, 5)]` is equivalent to `s[2:5]`.
8. **New object**: Slicing always returns a new string.

#### Constraints and Limitations

- Step cannot be 0: `s[::0]` raises `ValueError`.
- Slicing does not modify the original string.
- Negative step requires the start to be greater than the stop for a non-empty result.

### Multiple Annotated Complete Step by Step Code Examples and Their Expected Outputs

#### Example 1: Basic Slicing

```python
s = "Python Programming"

# step1: Basic slice (i inclusive, j exclusive)
print(s[0:6])      # Python

# step2: Omit start
print(s[:6])       # Python

# step3: Omit stop
print(s[7:])       # Programming

# step4: Negative indices
print(s[-4:])      # ming
print(s[-11:-8])   # Pro

# step5: Full copy
print(s[:])        # Python Programming
```

**Expected Output**:
```
Python
Python
Programming
ming
Pro
Python Programming
```

**Why**: Slicing extracts substrings using start (inclusive) and stop (exclusive) indices.

#### Example 2: Step and Reverse Slicing

```python
s = "abcdefgh"

# step1: Every second character
print(s[::2])      # aceg

# step2: Every third character
print(s[1::3])     # b e h  (indices 1, 4, 7)
print(s[1::3])     # beh

# step3: Reverse the string
print(s[::-1])     # hgfedcba

# step4: Reverse with start and stop
print(s[5:2:-1])   # fed (indices 5, 4, 3)
```

**Expected Output**:
```
aceg
beh
hgfedcba
fed
```

**Why**: The step parameter controls the stride; a negative step reverses direction.

#### Example 3: Slice Objects

```python
s = "Hello, World!"

# step1: Create a slice object
sl = slice(7, 12)
print(s[sl])       # World

# step2: Slice with step
sl2 = slice(0, 12, 2)
print(s[sl2])      # Hlo ol!

# step3: Slice attributes
print(sl.start)    # 7
print(sl.stop)     # 12
print(sl.step)     # None
```

**Expected Output**:
```
World
Hlo ol!
7
12
None
```

**Why**: Slice objects encapsulate start, stop, and step values and can be reused.

#### Example 4: Out-of-Range Slicing

```python
s = "abc"

# step1: Stop beyond length
print(s[0:100])    # abc

# step2: Start beyond length
print(s[100:])     # (empty string)
print(repr(s[100:]))  # ''

# step3: Negative start beyond length
print(s[-100:])    # abc
```

**Expected Output**:
```
abc

''
abc
```

**Why**: Slices silently truncate to the string's bounds; they never raise `IndexError`.

### Real-World Cases

- **File extensions**: `filename[-3:]` to get the extension.
- **String reversal**: `s[::-1]` for palindromes or encoding.
- **Data parsing**: Extracting fixed-width fields from records.
- **URL parsing**: `url[8:]` to strip `"https://"`.
- **Sampling**: `s[::2]` to take every other character.

### References

- Sequence Types — list, tuple, range (Slicings) - https://docs.python.org/3/library/stdtypes.html#slicings
- Slice Objects - https://docs.python.org/3/library/functions.html#slice
- Slicings - https://docs.python.org/3/reference/expressions.html#slicings

---

## 3. Concatenation

### Definitions

**Core Definition**: Concatenation is the operation of joining two or more strings end-to-end using the `+` operator, producing a new string.

**Technical Definition**: The `+` operator, when applied to two strings, returns a new string that is the concatenation of its operands. The `+=` operator (augmented assignment) performs concatenation and rebinds the name to the result. Because strings are immutable, each concatenation creates a new string object and copies the contents of both operands; iterative concatenation in a loop therefore has O(n²) time complexity in the worst case, which can be mitigated by using `str.join()` or `io.StringIO`.

**Beginner-Friendly Explanation**: Concatenation means gluing strings together. `"hello" + " " + "world"` gives you `"hello world"`. The `+` operator creates a brand-new string; it doesn't change the originals. If you need to join many strings in a loop, it's faster to collect them in a list and use `"".join(list)`.

### Purposes

- To combine multiple strings into a single string.
- To build messages from parts.
- To construct file paths or URLs.
- To append text to an existing string (via `+=`).
- To implement string builders in a functional style.

### Syntax Rules and Structure

#### Complete General Syntax

```python
s1 + s2          # Concatenation
s1 += s2         # Augmented concatenation
```

**Component Breakdown**:
- `s1`, `s2` — strings to concatenate.
- `+` — concatenation operator.
- `+=` — augmented assignment operator.

#### Syntax Rules

1. **Both operands must be strings**: `"a" + 1` raises `TypeError`.
2. **Returns a new string**: The original strings are unchanged.
3. **`+=` rebinds**: `s += "x"` is equivalent to `s = s + "x"`.
4. **Implicit concatenation of literals**: Adjacent string literals are concatenated at compile time.
5. **Performance**: Repeated `+=` in a loop is O(n²); use `str.join()` for O(n).
6. **`str.join()`**: The recommended way to concatenate many strings.

#### Constraints and Limitations

- Cannot concatenate strings with non-strings without explicit conversion.
- Repeated concatenation in loops is inefficient.
- `+` always creates a new string object.

### Multiple Annotated Complete Step by Step Code Examples and Their Expected Outputs

#### Example 1: Basic Concatenation

```python
# step1: Concatenate two strings
s1 = "Hello"
s2 = "World"
result = s1 + ", " + s2 + "!"
print(result)  # Hello, World!

# step2: Augmented concatenation
s = "Hello"
s += " World"
print(s)  # Hello World

# step3: Original strings unchanged
print(s1)  # Hello
print(s2)  # World
```

**Expected Output**:
```
Hello, World!
Hello World
Hello
World
```

**Why**: `+` creates a new string; the originals remain unchanged.

#### Example 2: Type Safety

```python
# step1: Concatenating string with non-string raises TypeError
try:
    "Count: " + 42
except TypeError as e:
    print(f"TypeError: {e}")  # can only concatenate str (not "int") to str

# step2: Correct way with str()
print("Count: " + str(42))  # Count: 42
```

**Expected Output**:
```
TypeError: can only concatenate str (not "int") to str
Count: 42
```

**Why**: `+` requires both operands to be strings; non-strings must be converted first.

#### Example 3: Efficient Concatenation with `join()`

```python
# step1: Inefficient concatenation in a loop
parts = []
for i in range(1000):
    parts.append(str(i))
result = "".join(parts)
print(result[:20] + "...")  # 01234567891011121314...

# step2: Compare with naive concatenation (conceptually)
naive = ""
for i in range(10):
    naive += str(i)
print(naive)  # 0123456789
```

**Expected Output**:
```
01234567891011121314...
0123456789
```

**Why**: `str.join()` is O(n) because it computes the total length once and allocates a single result string.

### Real-World Cases

- **Building messages**: `"Error: " + error_message`.
- **File paths**: `os.path.join()` (preferred) or `dir + "/" + filename`.
- **URL construction**: `base_url + "/api/" + endpoint`.
- **Logging**: Combining timestamps and messages.

### References

- Sequence Types — list, tuple, range (Concatenation) - https://docs.python.org/3/library/stdtypes.html#sequence-types-list-tuple-range
- Time Complexity of Operations - https://docs.python.org/3.16/builtins/time-complexity.html
- `str.join()` - https://docs.python.org/3/library/stdtypes.html#str.join

---

## 4. Repetition

### Definitions

**Core Definition**: Repetition is the operation of creating a new string by repeating an existing string a specified number of times using the `*` operator.

**Technical Definition**: The `*` operator, when applied to a string `s` and an integer `n`, returns a new string consisting of `n` copies of `s` concatenated together. If `n <= 0`, the result is the empty string. The augmented assignment `*=` performs repetition and rebinds the name. For strings, repetition is well-defined and safe; for mutable sequences like lists, repetition creates shallow copies of the elements, which can lead to unexpected aliasing.

**Beginner-Friendly Explanation**: Repetition lets you repeat a string multiple times. `"ab" * 3` gives you `"ababab"`. `"x" * 0` gives you an empty string. It's a quick way to create separators, padding, or repeated patterns.

### Purposes

- To create repeated patterns (e.g., separators, borders).
- To pad strings to a fixed width.
- To generate test data quickly.
- To create repeated headers or footers in output.
- To implement simple loops without explicit iteration.

### Syntax Rules and Structure

#### Complete General Syntax

```python
n * s          # Repeat s n times
s * n          # Repeat s n times (commutative)
s *= n         # Augmented repetition
```

**Component Breakdown**:
- `s` — the string to repeat.
- `n` — an integer specifying the number of repetitions.
- `*` — repetition operator.

#### Syntax Rules

1. **Integer required**: `n` must be an integer; `"a" * 1.5` raises `TypeError`.
2. **`n <= 0` yields empty string**: `"abc" * 0` → `''`; `"abc" * -1` → `''`.
3. **Commutative**: `n * s` and `s * n` are equivalent.
4. **Order of evaluation**: In `n * s`, `n` is evaluated first, then `s`.
5. **Shallow copies for mutable elements**: For strings (immutable), this is not a concern.
6. **Performance**: Repetition is O(n·k) where n is the length and k is the repeat count.

#### Constraints and Limitations

- `n` must be an integer; floats raise `TypeError`.
- Large repeat counts can consume significant memory.
- For mutable sequences, repetition creates shared references (not an issue for strings).

### Multiple Annotated Complete Step by Step Code Examples and Their Expected Outputs

#### Example 1: Basic Repetition

```python
# step1: Repeat a string
print("ab" * 3)     # ababab
print(3 * "ab")     # ababab

# step2: Repeat a single character
print("-" * 20)     # --------------------

# step3: Zero and negative repetitions
print(repr("abc" * 0))   # ''
print(repr("abc" * -1))  # ''

# step4: Augmented repetition
s = "x"
s *= 5
print(s)            # xxxxx
```

**Expected Output**:
```
ababab
ababab
--------------------
''
''
xxxxx
```

**Why**: `*` repeats the string; zero or negative counts produce an empty string.

#### Example 2: Creating Patterns

```python
# step1: Header line
header = "=" * 30
print(header)  # ==============================

# step2: Padded output
name = "Alice"
padded = name + " " * (20 - len(name)) + "|"
print(repr(padded))  # 'Alice               |'

# step3: Separator between items
items = ["a", "b", "c"]
print(" | ".join(items))  # a | b | c
```

**Expected Output**:
```
==============================
'Alice               |'
a | b | c
```

**Why**: Repetition creates fixed-width patterns and padding efficiently.

#### Example 3: Type Errors

```python
# step1: Float repetition raises TypeError
try:
    "a" * 1.5
except TypeError as e:
    print(f"TypeError: {e}")  # can't multiply sequence by non-int of type 'float'

# step2: String repetition raises TypeError
try:
    "a" * "b"
except TypeError as e:
    print(f"TypeError: {e}")  # can't multiply sequence by non-int of type 'str'
```

**Expected Output**:
```
TypeError: can't multiply sequence by non-int of type 'float'
TypeError: can't multiply sequence by non-int of type 'str'
```

**Why**: The repeat count must be an integer.

### Real-World Cases

- **Separators**: `"-" * 40` for console output.
- **Padding**: Aligning text in fixed-width columns.
- **Test data**: `"abc" * 100` for stress tests.
- **Banner creation**: Repeated characters for headers.

### References

- Sequence Types — list, tuple, range (Repetition) - https://docs.python.org/3/library/stdtypes.html#sequence-types-list-tuple-range
- Time Complexity of Operations - https://docs.python.org/3.16/builtins/time-complexity.html
- Common Sequence Operations - https://docs.python.org/3/library/stdtypes.html#common-sequence-operations

---

## 5. Membership

### Definitions

**Core Definition**: Membership testing is the operation of determining whether a substring or character exists within a string using the `in` and `not in` operators.

**Technical Definition**: The operators `in` and `not in` test for collection membership. For strings, `x in y` is true if and only if `x` is a substring of `y`; an equivalent test is `y.find(x) != -1`. Empty strings are always considered to be a substring of any other string. The operators evaluate to `True` or `False` and are implemented via the `__contains__` method. The `not in` operator returns the negation of `in`.

**Beginner-Friendly Explanation**: Membership testing answers the question "is this substring inside that string?" You write `"cat" in "concatenate"` and get `True` or `False`. The `not in` operator is the opposite: `"dog" not in "concatenate"` is `True`.

### Purposes

- To check if a string contains a specific substring.
- To validate input (e.g., checking for forbidden characters).
- To search for keywords in text.
- To filter strings based on content.
- To implement simple pattern matching without regular expressions.

### Syntax Rules and Structure

#### Complete General Syntax

```python
substring in string
substring not in string
```

**Component Breakdown**:
- `substring` — the string (or character) to search for.
- `string` — the string to search within.
- `in` / `not in` — membership operators.

#### Syntax Rules

1. **Returns bool**: `in` and `not in` always return `True` or `False`.
2. **Substring test**: For strings, `x in y` tests whether `x` is a substring of `y`.
3. **Empty string**: `"" in "abc"` is `True` (empty string is a substring of any string).
4. **Case-sensitive**: `"A" in "abc"` is `False`.
5. **`not in` is negation**: `x not in y` is equivalent to `not (x in y)`.
6. **Efficiency**: Membership testing is O(n) in the worst case.
7. **`str.find()` equivalent**: `x in y` is equivalent to `y.find(x) != -1`.

#### Constraints and Limitations

- Both operands must be strings; `"a" in 123` raises `TypeError`.
- Membership testing is case-sensitive.
- Cannot test for multiple substrings in one operation (use `any()`).

### Multiple Annotated Complete Step by Step Code Examples and Their Expected Outputs

#### Example 1: Basic Membership Testing

```python
# step1: Substring check
print("cat" in "concatenate")     # True
print("dog" in "concatenate")     # False

# step2: Character check
print("a" in "cat")               # True
print("z" in "cat")               # False

# step3: not in
print("dog" not in "concatenate") # True
print("cat" not in "concatenate") # False

# step4: Empty string
print("" in "abc")                # True
```

**Expected Output**:
```
True
False
True
False
True
False
True
```

**Why**: `in` tests for substring presence; `not in` is the negation; empty strings are always contained.

#### Example 2: Case Sensitivity and Multiple Checks

```python
text = "Hello, World!"

# step1: Case-sensitive
print("hello" in text)     # False
print("Hello" in text)     # True

# step2: Multiple checks with any()
forbidden = ["<", ">", "&"]
user_input = "safe text"
print(any(char in user_input for char in forbidden))  # False

user_input2 = "unsafe <script>"
print(any(char in user_input2 for char in forbidden))  # True
```

**Expected Output**:
```
False
True
False
True
```

**Why**: Membership is case-sensitive; `any()` combines multiple membership tests.

#### Example 3: Type Errors

```python
# step1: Non-string operand raises TypeError
try:
    "a" in 123
except TypeError as e:
    print(f"TypeError: {e}")  # argument of type 'int' is not iterable
```

**Expected Output**:
```
TypeError: argument of type 'int' is not iterable
```

**Why**: The right operand of `in` must be iterable; integers are not.

### Real-World Cases

- **Input validation**: Checking for forbidden characters.
- **Search functionality**: Finding keywords in text.
- **Template processing**: Detecting placeholders.
- **Log analysis**: Searching for error patterns.

### References

- Membership Test Operations - https://docs.python.org/3/reference/expressions.html#membership-test-operations
- `str.find()` - https://docs.python.org/3/library/stdtypes.html#str.find
- Common Sequence Operations - https://docs.python.org/3/library/stdtypes.html#common-sequence-operations

---

## 6. Comparison

### Definitions

**Core Definition**: Comparison is the operation of evaluating the equality or lexicographic ordering of two strings using operators such as `==`, `!=`, `<`, `>`, `<=`, and `>=`.

**Technical Definition**: Strings are compared lexicographically using the numeric equivalents (the result of the built-in function `ord()`) of their characters. For Python 3, comparison uses Unicode code points. Two strings compare equal if they have the same length and all corresponding characters are equal. For ordering, the first differing character determines the result; if one string is a prefix of the other, the shorter string is considered smaller. All comparison operators are supported: `==`, `!=`, `<`, `>`, `<=`, `>=`.

**Beginner-Friendly Explanation**: String comparison lets you check if two strings are the same or which one comes first alphabetically. `"apple" < "banana"` is `True` because `a` comes before `b`. `"abc" == "abc"` is `True`. Uppercase letters come before lowercase letters because their Unicode code points are smaller.

### Purposes

- To check if two strings are equal.
- To sort strings alphabetically.
- To implement search and ordering algorithms.
- To validate that strings match expected values.
- To compare strings in a case-insensitive manner (using `.lower()` or `.casefold()`).

### Syntax Rules and Structure

#### Complete General Syntax

```python
s1 == s2      # Equal
s1 != s2      # Not equal
s1 < s2       # Less than
s1 <= s2      # Less than or equal
s1 > s2       # Greater than
s1 >= s2      # Greater than or equal
```

**Component Breakdown**:
- `s1`, `s2` — strings to compare.
- `==`, `!=` — equality operators.
- `<`, `<=`, `>`, `>=` — ordering operators.

#### Syntax Rules

1. **Lexicographic order**: Comparison is character by character using Unicode code points.
2. **Equality**: `==` returns `True` if strings have the same length and identical characters.
3. **Prefix rule**: If one string is a prefix of the other, the shorter string is smaller: `"abc" < "abcd"` → `True`.
4. **Case sensitivity**: Uppercase letters have smaller code points than lowercase: `"A" < "a"` → `True`.
5. **Empty string**: `""` is smaller than any non-empty string.
6. **Unicode-aware**: Comparison works across all Unicode characters.
7. **`ord()`**: The numeric equivalent of a character is given by `ord()`.

#### Constraints and Limitations

- Comparison is case-sensitive by default.
- Locale-specific collation is not supported natively; use `locale.strcoll()` for locale-aware sorting.
- Comparison of very long strings may be costly in the worst case.

### Multiple Annotated Complete Step by Step Code Examples and Their Expected Outputs

#### Example 1: Basic Comparison

```python
# step1: Equality
print("abc" == "abc")    # True
print("abc" == "abd")    # False
print("abc" != "abd")    # True

# step2: Ordering
print("apple" < "banana")  # True
print("banana" < "apple")  # False
print("abc" < "abcd")      # True (prefix rule)

# step3: Case sensitivity
print("A" < "a")           # True
print("Z" < "a")           # True
```

**Expected Output**:
```
True
False
True
True
False
True
True
True
```

**Why**: Comparison is lexicographic by Unicode code point; uppercase letters have smaller code points.

#### Example 2: Using `ord()` to Understand Comparison

```python
# step1: Code points
print(ord("a"))  # 97
print(ord("A"))  # 65
print(ord("b"))  # 98

# step2: Comparison follows code points
print("A" < "a")    # True (65 < 97)
print("a" < "b")    # True (97 < 98)

# step3: Multi-character comparison
print("ab" < "ac")  # True ('b' < 'c')
print("ab" < "ba")  # True ('a' < 'b')
```

**Expected Output**:
```
97
65
98
True
True
True
True
```

**Why**: Comparison uses the numeric code points of characters from left to right.

#### Example 3: Case-Insensitive Comparison

```python
# step1: Direct comparison is case-sensitive
print("Hello" == "hello")  # False

# step2: Case-insensitive with lower()
print("Hello".lower() == "hello".lower())  # True

# step3: Case-insensitive with casefold()
print("Straße".casefold() == "strasse".casefold())  # True
```

**Expected Output**:
```
False
True
True
```

**Why**: Normalizing case before comparison allows case-insensitive equality; `casefold()` is more aggressive than `lower()` for Unicode.

### Real-World Cases

- **Sorting**: `sorted(words)` uses lexicographic comparison.
- **Validation**: Checking if input matches expected values.
- **Search**: Implementing binary search on sorted strings.
- **Unicode processing**: Comparing strings in multiple languages.

### References

- Comparison Operators - https://docs.python.org/3/reference/expressions.html#comparisons
- `ord()` - https://docs.python.org/3/library/functions.html#ord
- `str.casefold()` - https://docs.python.org/3/library/stdtypes.html#str.casefold
- Sorting Techniques - https://docs.python.org/3/howto/sorting.html

---

## References

- Sequence Types — list, tuple, range - https://docs.python.org/3/library/stdtypes.html#sequence-types-list-tuple-range
- Common Sequence Operations - https://docs.python.org/3/library/stdtypes.html#common-sequence-operations
- Slicings - https://docs.python.org/3/reference/expressions.html#slicings
- Slice Objects - https://docs.python.org/3/library/functions.html#slice
- Membership Test Operations - https://docs.python.org/3/reference/expressions.html#membership-test-operations
- Comparison Operators - https://docs.python.org/3/reference/expressions.html#comparisons
- Time Complexity of Operations - https://docs.python.org/3.16/builtins/time-complexity.html
- `str.join()` - https://docs.python.org/3/library/stdtypes.html#str.join
- `str.find()` - https://docs.python.org/3/library/stdtypes.html#str.find
- `ord()` - https://docs.python.org/3/library/functions.html#ord
- `str.casefold()` - https://docs.python.org/3/library/stdtypes.html#str.casefold
- Sorting Techniques - https://docs.python.org/3/howto/sorting.html