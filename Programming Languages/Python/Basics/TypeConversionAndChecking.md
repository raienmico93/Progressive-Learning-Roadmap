# Python Type Conversion and Type Checking: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Core Definition

Type conversion is the process of changing a value from one data type to another using built-in constructor functions such as `int()`, `float()`, `str()`, and `list()`. Type checking is the process of determining the type of an object at runtime, using functions such as `type()` and `isinstance()`, or verifying type correctness statically using external tools. Together, these mechanisms form the foundation of Python's data handling model.

### Technical Definition

Type conversion in Python is implemented through constructor functions and the `__init__` methods of built-in types. The `int()`, `float()`, `str()`, `bool()`, `list()`, `tuple()`, `set()`, `dict()`, and `bytes()` functions construct new objects of their respective types from a variety of input sources, including numeric literals, strings, and iterables. Type checking is implemented through the `type()` built-in, which returns the exact type object of an instance (or dynamically creates classes when called with three arguments), and `isinstance()`, which returns a Boolean indicating whether an object is an instance of a given class or a subclass thereof. Python's type system is dynamic and strong: implicit conversions are limited to numeric widening (e.g., `int` to `float`), and most conversions must be performed explicitly.

### Beginner-Friendly Explanation

Type conversion means turning one kind of data into another. If you have the string `"42"` but need an integer, you call `int("42")` to get `42`. Type checking means asking "what kind of data is this?" — `type(42)` tells you it's an `int`, while `isinstance(42, int)` tells you `True`. Python is strict about types: it won't silently turn a string into a number, so you must convert explicitly. The only common exception is that Python will automatically promote an integer to a float when needed in arithmetic.

### Key Characteristics

- **Explicit by default**: Python requires explicit conversion for most type changes; implicit conversion occurs only for numeric widening.
- **Constructor-based**: Conversion is performed by calling the type itself as a function (e.g., `int("5")`).
- **Strong typing**: Mixing incompatible types raises `TypeError` rather than silently converting.
- **Runtime inspection**: `type()` and `isinstance()` provide runtime type information.
- **Error-aware**: Conversion failures raise `ValueError` (wrong value for the type) or `TypeError` (wrong type entirely).

### Prerequisites

- Basic understanding of Python data types (`int`, `float`, `str`, `bool`, `list`, `tuple`, `set`, `dict`, `bytes`).
- Familiarity with Python variables and functions.
- A Python interpreter (Python 3.x).

### Related Programming Areas

- **Data validation and parsing**: Converting user input to the correct type.
- **Serialization and deserialization**: Converting between JSON strings and Python objects.
- **Type-oriented design**: Using `type()` and `isinstance()` for runtime type dispatch.
- **Static type checking**: mypy and Pyright verify type correctness before runtime.

---

## 1. `int()`

### Definitions

**Core Definition**: `int()` constructs an integer object from a number, a string, or no argument.

**Technical Definition**: `int(x=0)` and `int(x, base=10)` return an integer object constructed from a number or string `x`, or return `0` if no arguments are given. If `x` is a number, it returns `x.__int__()`. For floating-point numbers, this truncates toward zero. If `x` is a string, it must be a base-`base` integer literal, optionally preceded by `+` or `-`, with no spaces between the sign and the digits.

**Beginner-Friendly Explanation**: `int()` turns things into whole numbers. `int("42")` gives you `42`. `int(3.99)` gives you `3` — it cuts off the decimal part, it doesn't round.

### Purposes

- To convert strings or floats to integers.
- To parse user input (which is always a string) into an integer.
- To convert numbers between different bases (e.g., hexadecimal to decimal).
- To provide a default zero value when called with no arguments.

### Syntax Rules and Structure

#### Complete General Syntax

```python
int(x=0)
int(x, base=10)
```

**Component Breakdown**:
- `x` — optional number or string to convert. If omitted, returns `0`.
- `base` — optional base for string conversion; default is `10`.

#### Syntax Rules

1. **No argument returns `0`**: `int()` → `0`.
2. **Float truncation**: `int(3.99)` → `3`; `int(-3.99)` → `-3` (truncates toward zero).
3. **String must be valid literal**: `int("42")` works; `int("3.5")` raises `ValueError`.
4. **Base parameter only for strings**: `int("ff", 16)` → `255`; `int(255, 16)` raises `TypeError`.
5. **Leading/trailing whitespace allowed**: `int(" 42 ")` → `42`.
6. **Underscores allowed**: `int("1_000")` → `1000`.

#### Constraints and Limitations

- Cannot convert a string containing a decimal point directly: `int("3.5")` raises `ValueError`.
- Cannot convert `None`: `int(None)` raises `TypeError`.
- Base must be between 2 and 36 when specified.

### Multiple Annotated Complete Step by Step Code Examples and Their Expected Outputs

#### Example 1: Basic Conversions

```python
# step1: Convert string to int
print(int("42"))          # 42

# step2: Convert float to int (truncation)
print(int(3.99))          # 3
print(int(-3.99))         # -3

# step3: No argument
print(int())              # 0

# step4: Base conversion
print(int("ff", 16))      # 255
print(int("101", 2))      # 5

# step5: Whitespace and underscores
print(int(" 1_000 "))     # 1000
```

**Expected Output**:
```
42
3
-3
0
255
5
1000
```

**Why**: `int()` parses the string `"42"` as a decimal literal; `3.99` is truncated toward zero; `int("ff", 16)` interprets `"ff"` as a hexadecimal number.

#### Example 2: Error Handling

```python
# step1: ValueError for invalid string
try:
    int("3.5")
except ValueError as e:
    print(f"ValueError: {e}")  # invalid literal for int() with base 10: '3.5'

# step2: TypeError for None
try:
    int(None)
except TypeError as e:
    print(f"TypeError: {e}")   # int() argument must be a string, a bytes-like object or a real number, not 'NoneType'
```

**Expected Output**:
```
ValueError: invalid literal for int() with base 10: '3.5'
TypeError: int() argument must be a string, a bytes-like object or a real number, not 'NoneType'
```

**Why**: `"3.5"` has the right type (string) but an invalid value, so `ValueError` is raised. `None` has the wrong type entirely, so `TypeError` is raised.

#### Example 3: Parsing User Input

```python
# step1: Simulate user input
user_input = "  42  "

# step2: Convert to int
try:
    number = int(user_input)
    print(f"Parsed: {number}")  # Parsed: 42
except ValueError:
    print("Invalid input")
```

**Expected Output**:
```
Parsed: 42
```

**Why**: `int()` ignores surrounding whitespace and successfully parses the integer.

### Real-World Cases

- **User input parsing**: `age = int(input("Enter your age: "))`.
- **Configuration files**: `port = int(config["port"])`.
- **Data processing**: Converting string representations of numbers to integers.
- **Base conversion**: Parsing hexadecimal color codes: `int("ff0000", 16)`.

### References

- `int()` — Built-in Functions - https://docs.python.org/3/library/functions.html#int
- Numeric Types — int, float, complex - https://docs.python.org/3/library/stdtypes.html#typesnumeric

---

## 2. `float()`

### Definitions

**Core Definition**: `float()` constructs a floating-point number from a number or string.

**Technical Definition**: `float(number=0.0)` and `float(string)` return a floating-point number constructed from a number or string. If the argument is a string, it should contain a decimal number, optionally preceded by a sign, and optionally embedded in whitespace. The string may also be `'nan'`, `'inf'`, or `'-inf'` for NaN and positive or negative infinity.

**Beginner-Friendly Explanation**: `float()` turns things into decimal numbers. `float("3.14")` gives you `3.14`. `float(42)` gives you `42.0`.

### Purposes

- To convert strings or integers to floating-point numbers.
- To parse user input into decimal numbers.
- To handle special values like infinity and NaN.
- To provide a default zero value when called with no arguments.

### Syntax Rules and Structure

#### Complete General Syntax

```python
float(number=0.0)
float(string)
```

**Component Breakdown**:
- `number` — optional number (int or float) to convert.
- `string` — optional string containing a decimal representation.

#### Syntax Rules

1. **No argument returns `0.0`**: `float()` → `0.0`.
2. **String must be valid float**: `float("3.14")` works; `float("abc")` raises `ValueError`.
3. **Integer conversion**: `float(42)` → `42.0`.
4. **Special strings**: `float("inf")`, `float("-inf")`, `float("nan")`.
5. **Whitespace allowed**: `float(" 3.14 ")` → `3.14`.
6. **Scientific notation**: `float("1e-3")` → `0.001`.

#### Constraints and Limitations

- Cannot convert `None`: `float(None)` raises `TypeError`.
- String must contain a valid decimal representation; commas are not allowed.

### Multiple Annotated Complete Step by Step Code Examples and Their Expected Outputs

#### Example 1: Basic Conversions

```python
# step1: Convert string to float
print(float("3.14"))      # 3.14

# step2: Convert int to float
print(float(42))          # 42.0

# step3: No argument
print(float())            # 0.0

# step4: Scientific notation
print(float("1e-3"))      # 0.001

# step5: Special values
print(float("inf"))       # inf
print(float("-inf"))      # -inf
print(float("nan"))       # nan
```

**Expected Output**:
```
3.14
42.0
0.0
0.001
inf
-inf
nan
```

**Why**: `float()` parses decimal strings, converts integers, and handles special IEEE 754 values.

#### Example 2: Error Handling

```python
# step1: ValueError for invalid string
try:
    float("abc")
except ValueError as e:
    print(f"ValueError: {e}")  # could not convert string to float: 'abc'

# step2: TypeError for None
try:
    float(None)
except TypeError as e:
    print(f"TypeError: {e}")   # float() argument must be a string or a real number, not 'NoneType'
```

**Expected Output**:
```
ValueError: could not convert string to float: 'abc'
TypeError: float() argument must be a string or a real number, not 'NoneType'
```

**Why**: `"abc"` is a string but not a valid float, so `ValueError` is raised. `None` is the wrong type, so `TypeError` is raised.

#### Example 3: Parsing User Input

```python
user_input = "  3.14  "
try:
    number = float(user_input)
    print(f"Parsed: {number}")  # Parsed: 3.14
except ValueError:
    print("Invalid input")
```

**Expected Output**:
```
Parsed: 3.14
```

**Why**: `float()` ignores surrounding whitespace and parses the decimal number.

### Real-World Cases

- **Scientific computing**: Converting measurement strings to floats.
- **Financial calculations**: Parsing currency values.
- **User input**: `price = float(input("Enter price: "))`.
- **Data processing**: Converting CSV columns to floats.

### References

- `float()` — Built-in Functions - https://docs.python.org/3/library/functions.html#float
- Numeric Types — int, float, complex - https://docs.python.org/3/library/stdtypes.html#typesnumeric

---

## 3. `str()`

### Definitions

**Core Definition**: `str()` returns a string version of an object.

**Technical Definition**: `str(object='')` and `str(object=b'', encoding='utf-8', errors='strict')` return a string version of `object`. If no object is given, returns the empty string `''`. If `encoding` or `errors` is specified, `object` must be a bytes-like object, and the result is the decoded string. Otherwise, `str()` returns the result of `object.__str__()`, or `repr(object)` if `__str__` is not defined.

**Beginner-Friendly Explanation**: `str()` turns anything into text. `str(42)` gives you `"42"`. `str(3.14)` gives you `"3.14"`. `str(True)` gives you `"True"`.

### Purposes

- To convert numbers, booleans, and other objects to strings.
- To format output for printing or concatenation.
- To decode bytes into strings.
- To provide a human-readable representation of an object.

### Syntax Rules and Structure

#### Complete General Syntax

```python
str(object='')
str(object=b'', encoding='utf-8', errors='strict')
```

**Component Breakdown**:
- `object` — optional object to convert; default is `''`.
- `encoding` — optional encoding for bytes decoding.
- `errors` — optional error handling scheme.

#### Syntax Rules

1. **No argument returns `''`**: `str()` → `''`.
2. **Numbers become their decimal representation**: `str(42)` → `'42'`.
3. **Floats use their `repr`**: `str(3.14)` → `'3.14'`.
4. **Booleans become `'True'` or `'False'`**: `str(True)` → `'True'`.
5. **Bytes require encoding**: `str(b'hello', 'utf-8')` → `'hello'`.
6. **Uses `__str__` if defined**: Custom objects can define `__str__` to control conversion.

#### Constraints and Limitations

- `str()` on a bytes object without encoding returns the `repr` of the bytes: `str(b'hello')` → `"b'hello'"`.
- `str()` does not recursively convert collections; it uses their `repr`.

### Multiple Annotated Complete Step by Step Code Examples and Their Expected Outputs

#### Example 1: Basic Conversions

```python
# step1: Convert int to str
print(str(42))            # 42

# step2: Convert float to str
print(str(3.14))          # 3.14

# step3: Convert bool to str
print(str(True))          # True

# step4: No argument
print(repr(str()))        # ''

# step5: Convert list to str
print(str([1, 2, 3]))     # [1, 2, 3]
```

**Expected Output**:
```
42
3.14
True
''
[1, 2, 3]
```

**Why**: `str()` produces the human-readable representation of each object; for collections, it uses their `repr`.

#### Example 2: Bytes Decoding

```python
# step1: Decode bytes to str
data = b"hello"
print(str(data, "utf-8"))  # hello

# step2: Without encoding, bytes use repr
print(str(data))           # b'hello'
```

**Expected Output**:
```
hello
b'hello'
```

**Why**: Specifying `"utf-8"` decodes the bytes into a string; without it, `str()` returns the `repr` of the bytes object.

#### Example 3: Custom `__str__`

```python
class Point:
    def __init__(self, x: int, y: int) -> None:
        self.x = x
        self.y = y

    def __str__(self) -> str:
        return f"Point({self.x}, {self.y})"

p = Point(3, 4)
print(str(p))  # Point(3, 4)
```

**Expected Output**:
```
Point(3, 4)
```

**Why**: The custom `__str__` method defines how the object is converted to a string.

### Real-World Cases

- **Printing and logging**: `print("Value: " + str(42))`.
- **JSON serialization**: Converting values to strings for JSON.
- **File output**: Writing formatted data to text files.
- **User-facing messages**: Building messages with embedded values.

### References

- `str()` — Built-in Functions - https://docs.python.org/3/library/functions.html#str
- Text Sequence Type — str - https://docs.python.org/3/library/stdtypes.html#text-sequence-type-str

---

## 4. `bool()`

### Definitions

**Core Definition**: `bool()` returns the truth value of an object using Python's standard truth testing procedure.

**Technical Definition**: `bool(object)` returns `True` or `False` based on whether the object is considered truthy or falsy. By default, an object is truthy unless its class defines either a `__bool__()` method that returns `False` or a `__len__()` method that returns zero. The following are falsy: `None`, `False`, zero of any numeric type (`0`, `0.0`, `0j`), empty sequences and collections (`''`, `()`, `[]`, `{}`, `set()`).

**Beginner-Friendly Explanation**: `bool()` tells you whether something is "true" or "false" in Python's eyes. `bool(0)` is `False`, `bool(1)` is `True`, `bool("")` is `False`, `bool("hello")` is `True`.

### Purposes

- To determine the truthiness of any object.
- To convert values to explicit booleans for clarity.
- To test whether a container is non-empty.
- To implement custom truthiness via `__bool__` or `__len__`.

### Syntax Rules and Structure

#### Complete General Syntax

```python
bool(object)
```

**Component Breakdown**:
- `object` — the object to evaluate; no default.

#### Syntax Rules

1. **Falsy values**: `None`, `False`, `0`, `0.0`, `0j`, `''`, `()`, `[]`, `{}`, `set()`, `range(0)`.
2. **Truthy values**: Everything else, including non-zero numbers, non-empty strings, and non-empty containers.
3. **String `"False"` is truthy**: `bool("False")` → `True` (non-empty string).
4. **`__bool__` takes precedence**: If a class defines `__bool__`, it is used; otherwise `__len__` is used.
5. **No argument**: `bool()` with no argument raises `TypeError`.

#### Constraints and Limitations

- `bool()` always returns a bool (`True` or `False`), never the original object.
- Custom classes can define truthiness via `__bool__` or `__len__`.

### Multiple Annotated Complete Step by Step Code Examples and Their Expected Outputs

#### Example 1: Basic Truthiness

```python
# step1: Falsy values
print(bool(0))        # False
print(bool(0.0))      # False
print(bool(""))       # False
print(bool([]))       # False
print(bool({}))       # False
print(bool(None))     # False
print(bool(False))    # False

# step2: Truthy values
print(bool(1))        # True
print(bool(-1))       # True
print(bool("hello"))  # True
print(bool([1]))      # True
print(bool("False"))  # True  (non-empty string!)
```

**Expected Output**:
```
False
False
False
False
False
False
False
True
True
True
True
True
```

**Why**: The standard truth testing procedure marks zero, empty, and `None` values as false.

#### Example 2: Custom `__bool__`

```python
class Box:
    def __init__(self, items: list) -> None:
        self.items = items

    def __bool__(self) -> bool:
        return len(self.items) > 0

b1 = Box([])
b2 = Box([1, 2])
print(bool(b1))  # False
print(bool(b2))  # True
```

**Expected Output**:
```
False
True
```

**Why**: The custom `__bool__` method defines truthiness based on the number of items.

#### Example 3: Using `bool()` for Clarity

```python
data = [1, 2, 3]

# step1: Implicit truthiness
if data:
    print("Has data")

# step2: Explicit bool() for clarity
if bool(data):
    print("Has data (explicit)")
```

**Expected Output**:
```
Has data
Has data (explicit)
```

**Why**: `bool()` makes the truthiness test explicit, which can improve readability.

### Real-World Cases

- **Conditional checks**: `if bool(user_input): ...`.
- **Validation**: Checking if a list of errors is empty.
- **Configuration flags**: Converting environment variables to booleans.
- **Custom containers**: Implementing `__bool__` to define emptiness.

### References

- `bool()` — Built-in Functions - https://docs.python.org/3/library/functions.html#bool
- Truth Value Testing - https://docs.python.org/3/library/stdtypes.html#truth-value-testing

---

## 5. `list()`

### Definitions

**Core Definition**: `list()` constructs a list from an iterable or creates an empty list.

**Technical Definition**: `list()` or `list(iterable)` builds a list whose items are the same and in the same order as `iterable`'s items. `iterable` may be a sequence, a container supporting iteration, or an iterator object. If no argument is given, the constructor creates a new empty list, `[]`.

**Beginner-Friendly Explanation**: `list()` turns things into lists. `list("abc")` gives you `['a', 'b', 'c']`. `list((1, 2, 3))` gives you `[1, 2, 3]`. `list()` gives you an empty list.

### Purposes

- To convert any iterable (string, tuple, set, dict, range) into a list.
- To create a mutable copy of an iterable.
- To materialize an iterator into a list.
- To create an empty list when no argument is given.

### Syntax Rules and Structure

#### Complete General Syntax

```python
list()
list(iterable)
```

**Component Breakdown**:
- `iterable` — optional iterable to convert; if omitted, returns `[]`.

#### Syntax Rules

1. **No argument returns `[]`**: `list()` → `[]`.
2. **String to list of characters**: `list("abc")` → `['a', 'b', 'c']`.
3. **Tuple to list**: `list((1, 2, 3))` → `[1, 2, 3]`.
4. **Set to list (order undefined)**: `list({1, 2, 3})` → order varies.
5. **Dict to list of keys**: `list({"a": 1, "b": 2})` → `['a', 'b']`.
6. **Iterator to list**: `list(range(3))` → `[0, 1, 2]`.
7. **Copy semantics**: `list(existing_list)` creates a shallow copy.

#### Constraints and Limitations

- Cannot convert non-iterable objects: `list(42)` raises `TypeError`.
- Converting a dict yields keys, not values or key-value pairs.

### Multiple Annotated Complete Step by Step Code Examples and Their Expected Outputs

#### Example 1: Basic Conversions

```python
# step1: String to list
print(list("abc"))           # ['a', 'b', 'c']

# step2: Tuple to list
print(list((1, 2, 3)))       # [1, 2, 3]

# step3: Range to list
print(list(range(3)))        # [0, 1, 2]

# step4: No argument
print(list())                # []

# step5: Set to list (order may vary)
print(sorted(list({3, 1, 2})))  # [1, 2, 3]
```

**Expected Output**:
```
['a', 'b', 'c']
[1, 2, 3]
[0, 1, 2]
[]
[1, 2, 3]
```

**Why**: `list()` iterates over the input and collects the items in order. Sets are unordered, so sorting is used for a deterministic result.

#### Example 2: Dict to List

```python
data = {"a": 1, "b": 2, "c": 3}
print(list(data))           # ['a', 'b', 'c']  (keys only)
print(list(data.items()))   # [('a', 1), ('b', 2), ('c', 3)]
print(list(data.values()))  # [1, 2, 3]
```

**Expected Output**:
```
['a', 'b', 'c']
[('a', 1), ('b', 2), ('c', 3)]
[1, 2, 3]
```

**Why**: Iterating over a dict yields its keys; use `.items()` or `.values()` for other views.

#### Example 3: Error Handling

```python
try:
    list(42)
except TypeError as e:
    print(f"TypeError: {e}")  # 'int' object is not iterable
```

**Expected Output**:
```
TypeError: 'int' object is not iterable
```

**Why**: `42` is not iterable, so `list()` raises `TypeError`.

### Real-World Cases

- **Data processing**: Converting generator outputs to lists for multiple passes.
- **API responses**: Converting JSON arrays to Python lists.
- **User input**: Converting a comma-separated string to a list.
- **Copying**: Creating a shallow copy of a list.

### References

- `list()` — Built-in Functions - https://docs.python.org/3/library/functions.html#list
- Lists — https://docs.python.org/3/tutorial/datastructures.html#more-on-lists

---

## 6. `tuple()`

### Definitions

**Core Definition**: `tuple()` constructs a tuple from an iterable or creates an empty tuple.

**Technical Definition**: `tuple()` or `tuple(iterable)` builds a tuple whose items are the same and in the same order as `iterable`'s items. If no argument is given, the constructor creates a new empty tuple, `()`.

**Beginner-Friendly Explanation**: `tuple()` turns things into tuples. `tuple([1, 2, 3])` gives you `(1, 2, 3)`. `tuple("abc")` gives you `('a', 'b', 'c')`.

### Purposes

- To convert any iterable into an immutable tuple.
- To create a hashable version of a list (for use as dict keys or set elements).
- To materialize an iterator into a tuple.
- To create an empty tuple when no argument is given.

### Syntax Rules and Structure

#### Complete General Syntax

```python
tuple()
tuple(iterable)
```

**Component Breakdown**:
- `iterable` — optional iterable to convert; if omitted, returns `()`.

#### Syntax Rules

1. **No argument returns `()`**: `tuple()` → `()`.
2. **String to tuple of characters**: `tuple("abc")` → `('a', 'b', 'c')`.
3. **List to tuple**: `tuple([1, 2, 3])` → `(1, 2, 3)`.
4. **Single-element tuple requires comma**: `tuple([1])` → `(1,)`; `(1)` is not a tuple.
5. **Immutable**: Tuples cannot be modified after creation.
6. **Hashable**: Tuples of hashable elements can be used as dict keys.

#### Constraints and Limitations

- Cannot convert non-iterable objects: `tuple(42)` raises `TypeError`.
- Tuples are immutable; no append, remove, or sort methods.

### Multiple Annotated Complete Step by Step Code Examples and Their Expected Outputs

#### Example 1: Basic Conversions

```python
# step1: List to tuple
print(tuple([1, 2, 3]))      # (1, 2, 3)

# step2: String to tuple
print(tuple("abc"))          # ('a', 'b', 'c')

# step3: No argument
print(tuple())               # ()

# step4: Single-element tuple
print(tuple([1]))            # (1,)
print(type(tuple([1])))      # <class 'tuple'>
```

**Expected Output**:
```
(1, 2, 3)
('a', 'b', 'c')
()
(1,)
<class 'tuple'>
```

**Why**: `tuple()` iterates over the input and collects the items. A single-element tuple is denoted by a trailing comma.

#### Example 2: Hashability

```python
# step1: Tuple as dict key
location = (40.7128, -74.0060)
data = {location: "New York"}
print(data[(40.7128, -74.0060)])  # New York

# step2: List cannot be a dict key
try:
    {[1, 2]: "value"}
except TypeError as e:
    print(f"TypeError: {e}")  # unhashable type: 'list'
```

**Expected Output**:
```
New York
TypeError: unhashable type: 'list'
```

**Why**: Tuples are hashable if their elements are hashable; lists are not hashable.

#### Example 3: Error Handling

```python
try:
    tuple(42)
except TypeError as e:
    print(f"TypeError: {e}")  # 'int' object is not iterable
```

**Expected Output**:
```
TypeError: 'int' object is not iterable
```

**Why**: `42` is not iterable, so `tuple()` raises `TypeError`.

### Real-World Cases

- **Coordinates**: `(latitude, longitude)` tuples.
- **Database records**: Immutable row representations.
- **Function returns**: Returning multiple values as a tuple.
- **Dict keys**: Composite keys using tuples.

### References

- `tuple()` — Built-in Functions - https://docs.python.org/3/library/functions.html#tuple
- Tuples and Sequence Types — list, tuple, range - https://docs.python.org/3/library/stdtypes.html#tuples

---

## 7. `set()`

### Definitions

**Core Definition**: `set()` constructs a set from an iterable or creates an empty set.

**Technical Definition**: `set()` or `set(iterable)` returns a new set object whose elements are taken from `iterable`. Sets are unordered collections of unique, hashable elements. If no argument is given, returns an empty set, `set()` (note: `{}` creates an empty dict, not a set).

**Beginner-Friendly Explanation**: `set()` turns things into sets, which are collections of unique items. `set([1, 2, 2, 3])` gives you `{1, 2, 3}` — duplicates are removed. `set()` gives you an empty set.

### Purposes

- To remove duplicates from an iterable.
- To perform mathematical set operations (union, intersection, difference).
- To test membership efficiently.
- To create an empty set when no argument is given.

### Syntax Rules and Structure

#### Complete General Syntax

```python
set()
set(iterable)
```

**Component Breakdown**:
- `iterable` — optional iterable to convert; if omitted, returns `set()`.

#### Syntax Rules

1. **No argument returns `set()`**: `set()` → `set()`.
2. **Duplicates removed**: `set([1, 1, 2])` → `{1, 2}`.
3. **Unordered**: Sets do not preserve insertion order.
4. **Elements must be hashable**: Lists and dicts cannot be set elements.
5. **`{}` is an empty dict, not a set**: Use `set()` for an empty set.

#### Constraints and Limitations

- Cannot contain unhashable elements (lists, dicts, sets).
- Order is not guaranteed; use `sorted()` for deterministic output.
- `set()` is mutable; use `frozenset()` for an immutable version.

### Multiple Annotated Complete Step by Step Code Examples and Their Expected Outputs

#### Example 1: Basic Conversions

```python
# step1: List to set (duplicates removed)
print(set([1, 2, 2, 3, 3, 3]))  # {1, 2, 3}

# step2: String to set of characters
print(set("hello"))              # {'h', 'e', 'l', 'o'}

# step3: No argument
print(set())                     # set()

# step4: Range to set
print(set(range(3)))             # {0, 1, 2}
```

**Expected Output**:
```
{1, 2, 3}
{'h', 'e', 'l', 'o'}
set()
{0, 1, 2}
```

**Why**: `set()` iterates over the input and keeps only unique, hashable elements.

#### Example 2: Set Operations

```python
a = set([1, 2, 3])
b = set([3, 4, 5])

print(a | b)  # {1, 2, 3, 4, 5}  (union)
print(a & b)  # {3}              (intersection)
print(a - b)  # {1, 2}           (difference)
print(a ^ b)  # {1, 2, 4, 5}     (symmetric difference)
```

**Expected Output**:
```
{1, 2, 3, 4, 5}
{3}
{1, 2}
{1, 2, 4, 5}
```

**Why**: Sets support mathematical set operations using operators or methods.

#### Example 3: Error Handling

```python
try:
    set([[1, 2], [3, 4]])
except TypeError as e:
    print(f"TypeError: {e}")  # unhashable type: 'list'
```

**Expected Output**:
```
TypeError: unhashable type: 'list'
```

**Why**: Lists are unhashable and cannot be set elements.

### Real-World Cases

- **Deduplication**: Removing duplicate entries from a list.
- **Membership testing**: Fast `in` checks.
- **Mathematical operations**: Finding common elements between datasets.
- **Tag systems**: Unique tags for objects.

### References

- `set()` — Built-in Functions - https://docs.python.org/3/library/functions.html#set
- Set Types — set, frozenset - https://docs.python.org/3/library/stdtypes.html#set-types-set-frozenset

---

## 8. `dict()`

### Definitions

**Core Definition**: `dict()` constructs a dictionary from keyword arguments, a mapping, or an iterable of key-value pairs.

**Technical Definition**: `dict(**kwargs)`, `dict(mapping, **kwargs)`, and `dict(iterable, **kwargs)` create a new dictionary. The dict object is the dictionary class. Dictionaries map hashable keys to arbitrary values.

**Beginner-Friendly Explanation**: `dict()` turns things into dictionaries. `dict(a=1, b=2)` gives you `{'a': 1, 'b': 2}`. `dict([('a', 1), ('b', 2)])` gives you the same.

### Purposes

- To create dictionaries from keyword arguments.
- To convert a list of key-value pairs into a dictionary.
- To create a copy of an existing dictionary.
- To merge dictionaries.

### Syntax Rules and Structure

#### Complete General Syntax

```python
dict(**kwargs)
dict(mapping, **kwargs)
dict(iterable, **kwargs)
```

**Component Breakdown**:
- `**kwargs` — keyword arguments become key-value pairs.
- `mapping` — a mapping object whose keys and values are copied.
- `iterable` — an iterable of key-value pairs (tuples of length 2).

#### Syntax Rules

1. **Keyword arguments**: `dict(a=1, b=2)` → `{'a': 1, 'b': 2}`.
2. **Mapping**: `dict({"a": 1})` → `{'a': 1}` (shallow copy).
3. **Iterable of pairs**: `dict([("a", 1), ("b", 2)])` → `{'a': 1, 'b': 2}`.
4. **Keys must be hashable**: Lists cannot be keys.
5. **Duplicate keys**: Later values overwrite earlier ones.
6. **Merging**: `dict(d1, **d2)` merges `d1` and `d2`.

#### Constraints and Limitations

- Keys must be hashable.
- Iterable must contain exactly two elements per item.
- Keyword argument keys must be valid identifiers.

### Multiple Annotated Complete Step by Step Code Examples and Their Expected Outputs

#### Example 1: Basic Conversions

```python
# step1: From keyword arguments
print(dict(a=1, b=2))                # {'a': 1, 'b': 2}

# step2: From mapping
print(dict({"x": 10, "y": 20}))      # {'x': 10, 'y': 20}

# step3: From iterable of pairs
print(dict([("a", 1), ("b", 2)]))    # {'a': 1, 'b': 2}

# step4: No argument
print(dict())                        # {}
```

**Expected Output**:
```
{'a': 1, 'b': 2}
{'x': 10, 'y': 20}
{'a': 1, 'b': 2}
{}
```

**Why**: `dict()` accepts keyword arguments, mappings, and iterables of key-value pairs.

#### Example 2: Merging Dictionaries

```python
d1 = {"a": 1, "b": 2}
d2 = {"b": 3, "c": 4}

# step1: Merge with dict()
merged = dict(d1, **d2)
print(merged)  # {'a': 1, 'b': 3, 'c': 4}

# step2: Using the | operator (Python 3.9+)
merged2 = d1 | d2
print(merged2)  # {'a': 1, 'b': 3, 'c': 4}
```

**Expected Output**:
```
{'a': 1, 'b': 3, 'c': 4}
{'a': 1, 'b': 3, 'c': 4}
```

**Why**: `dict(d1, **d2)` merges `d2` into `d1`, with `d2`'s values overwriting conflicting keys.

#### Example 3: Error Handling

```python
try:
    dict([("a", 1, 2)])
except ValueError as e:
    print(f"ValueError: {e}")  # dictionary update sequence element #0 has length 3; 2 is required
```

**Expected Output**:
```
ValueError: dictionary update sequence element #0 has length 3; 2 is required
```

**Why**: Each item in the iterable must have exactly two elements.

### Real-World Cases

- **Configuration**: Building config dicts from keyword arguments.
- **JSON parsing**: Converting JSON objects to Python dicts.
- **Data aggregation**: Counting occurrences with `dict.fromkeys()`.
- **Caching**: Using dicts as key-value stores.

### References

- `dict()` — Built-in Functions - https://docs.python.org/3/library/functions.html#dict
- Mapping Types — dict - https://docs.python.org/3/library/stdtypes.html#typesmapping

---

## 9. `bytes()`

### Definitions

**Core Definition**: `bytes()` constructs an immutable sequence of bytes from an integer, an iterable of integers, or a string with encoding.

**Technical Definition**: `bytes(source=b'')`, `bytes(source, encoding, errors='strict')`, and `bytes(integer)` return a new bytes object, which is an immutable sequence of integers in the range `0 <= x < 256`. The constructor accepts an integer (creating that many null bytes), an iterable of integers, a string with encoding, or a bytes-like object (creating a copy).

**Beginner-Friendly Explanation**: `bytes()` creates a sequence of raw bytes. `bytes(3)` gives you three zero bytes: `b'\x00\x00\x00'`. `bytes("hello", "utf-8")` encodes the string into bytes.

### Purposes

- To create byte sequences for binary data handling.
- To encode strings into bytes.
- To convert iterables of integers into bytes.
- To create immutable binary buffers.

### Syntax Rules and Structure

#### Complete General Syntax

```python
bytes(source=b'')
bytes(source, encoding, errors='strict')
bytes(integer)
```

**Component Breakdown**:
- `source` — optional source: integer, iterable, string, or bytes-like object.
- `encoding` — required when `source` is a string.
- `errors` — optional error handling scheme.

#### Syntax Rules

1. **Integer source**: `bytes(3)` → `b'\x00\x00\x00'` (three null bytes).
2. **String source requires encoding**: `bytes("hello", "utf-8")` → `b'hello'`.
3. **Iterable of integers**: `bytes([65, 66, 67])` → `b'ABC'`.
4. **Bytes-like source creates copy**: `bytes(b'hello')` → `b'hello'`.
5. **No argument returns `b''`**: `bytes()` → `b''`.
6. **Values must be 0–255**: `bytes([256])` raises `ValueError`.

#### Constraints and Limitations

- Bytes are immutable; use `bytearray()` for mutable byte sequences.
- Integer source creates null bytes, not the integer's binary representation.

### Multiple Annotated Complete Step by Step Code Examples and Their Expected Outputs

#### Example 1: Basic Conversions

```python
# step1: Integer source
print(bytes(3))                 # b'\x00\x00\x00'

# step2: String with encoding
print(bytes("hello", "utf-8"))  # b'hello'

# step3: Iterable of integers
print(bytes([65, 66, 67]))      # b'ABC'

# step4: No argument
print(bytes())                  # b''
```

**Expected Output**:
```
b'\x00\x00\x00'
b'hello'
b'ABC'
b''
```

**Why**: `bytes()` creates byte sequences from various sources; integer sources produce null bytes.

#### Example 2: Encoding and Decoding

```python
# step1: Encode string to bytes
text = "héllo"
encoded = bytes(text, "utf-8")
print(encoded)  # b'h\xc3\xa9llo'

# step2: Decode bytes to string
decoded = encoded.decode("utf-8")
print(decoded)  # héllo
```

**Expected Output**:
```
b'h\xc3\xa9llo'
héllo
```

**Why**: UTF-8 encodes the accented character as two bytes; decoding reverses the process.

#### Example 3: Error Handling

```python
try:
    bytes([256])
except ValueError as e:
    print(f"ValueError: {e}")  # bytes must be in range(0, 256)
```

**Expected Output**:
```
ValueError: bytes must be in range(0, 256)
```

**Why**: Byte values must be between 0 and 255 inclusive.

### Real-World Cases

- **File I/O**: Reading and writing binary files.
- **Network protocols**: Constructing packet payloads.
- **Cryptography**: Handling encrypted byte sequences.
- **Encoding**: Converting text to UTF-8 for transmission.

### References

- `bytes()` — Built-in Functions - https://docs.python.org/3/library/functions.html#bytes
- Bytes Objects - https://docs.python.org/3/library/stdtypes.html#bytes

---

## 10. `type()`

### Definitions

**Core Definition**: `type()` returns the type of an object when called with one argument, or dynamically creates a new class when called with three arguments.

**Technical Definition**: `type(object)` returns the type of `object`. `type(name, bases, dict)` returns a new type object, essentially a dynamic form of the `class` statement. The three-argument form is the metaclass constructor.

**Beginner-Friendly Explanation**: `type(42)` tells you that `42` is an `int`. `type("hello")` tells you it's a `str`. With three arguments, `type()` can create new classes on the fly.

### Purposes

- To determine the exact type of an object at runtime.
- To dynamically create classes for metaprogramming.
- To compare types for exact type checking.
- To inspect the class hierarchy.

### Syntax Rules and Structure

#### Complete General Syntax

```python
type(object)
type(name, bases, dict, **kwargs)
```

**Component Breakdown**:
- `object` — the object whose type is returned.
- `name` — the name of the new class.
- `bases` — a tuple of base classes.
- `dict` — a dictionary of class attributes.

#### Syntax Rules

1. **One argument returns type**: `type(42)` → `<class 'int'>`.
2. **Three arguments create class**: `type('MyClass', (), {})` creates a new class.
3. **Exact type only**: `type()` does not consider inheritance.
4. **Comparison**: `type(x) == int` checks exact type.
5. **Metaclass**: The three-argument form is the constructor for all classes.

#### Constraints and Limitations

- `type()` does not respect inheritance; use `isinstance()` for subtype checks.
- Dynamic class creation is rarely needed in everyday code.

### Multiple Annotated Complete Step by Step Code Examples and Their Expected Outputs

#### Example 1: Getting Types

```python
# step1: Get type of various objects
print(type(42))          # <class 'int'>
print(type(3.14))        # <class 'float'>
print(type("hello"))     # <class 'str'>
print(type([1, 2]))      # <class 'list'>
print(type({}))          # <class 'dict'>
print(type(None))        # <class 'NoneType'>
```

**Expected Output**:
```
<class 'int'>
<class 'float'>
<class 'str'>
<class 'list'>
<class 'dict'>
<class 'NoneType'>
```

**Why**: `type()` returns the exact class of the object.

#### Example 2: Exact Type Checking

```python
class Animal:
    pass

class Dog(Animal):
    pass

d = Dog()
print(type(d) == Dog)       # True
print(type(d) == Animal)    # False (exact type only)
print(isinstance(d, Animal))  # True (inheritance considered)
```

**Expected Output**:
```
True
False
True
```

**Why**: `type()` checks the exact class; `isinstance()` considers inheritance.

#### Example 3: Dynamic Class Creation

```python
# step1: Create a class dynamically
MyClass = type("MyClass", (), {"x": 42, "greet": lambda self: "Hello"})

# step2: Instantiate and use
obj = MyClass()
print(obj.x)         # 42
print(obj.greet())   # Hello
print(type(obj))     # <class '__main__.MyClass'>
```

**Expected Output**:
```
42
Hello
<class '__main__.MyClass'>
```

**Why**: The three-argument form of `type()` creates a new class with the given name, bases, and attributes.

### Real-World Cases

- **Debugging**: Inspecting the type of unexpected values.
- **Metaprogramming**: Creating classes dynamically in frameworks.
- **Serialization**: Determining how to serialize objects based on type.
- **Type dispatch**: Using `type()` for exact type matching.

### References

- `type()` — Built-in Functions - https://docs.python.org/3/library/functions.html#type
- Metaclasses - https://docs.python.org/3/reference/datamodel.html#metaclasses

---

## 11. `isinstance()`

### Definitions

**Core Definition**: `isinstance()` returns `True` if an object is an instance of a specified class or a subclass thereof.

**Technical Definition**: `isinstance(object, classinfo)` returns `True` if `object` is an instance of `classinfo`, or of a direct, indirect, or virtual subclass thereof. If `classinfo` is a tuple of type objects, the function returns `True` if `object` is an instance of any of them. If `object` is not an object of the given type, the function always returns `False`.

**Beginner-Friendly Explanation**: `isinstance(42, int)` gives you `True`. `isinstance("hello", int)` gives you `False`. Unlike `type()`, `isinstance()` also considers subclasses, so `isinstance(dog, Animal)` is `True` if `Dog` inherits from `Animal`.

### Purposes

- To check if an object is an instance of a class or its subclasses.
- To support type narrowing in conditional branches.
- To handle polymorphism correctly.
- To validate input types at runtime.

### Syntax Rules and Structure

#### Complete General Syntax

```python
isinstance(object, classinfo)
```

**Component Breakdown**:
- `object` — the object to check.
- `classinfo` — a class, type, tuple of classes/types, or a union type.

#### Syntax Rules

1. **Returns bool**: `isinstance(x, int)` → `True` or `False`.
2. **Considers inheritance**: `isinstance(dog, Animal)` → `True` if `Dog` inherits from `Animal`.
3. **Tuple of types**: `isinstance(x, (int, float))` → `True` if `x` is either.
4. **Union types (Python 3.10+)**: `isinstance(x, int | str)`.
5. **Protocol support**: `isinstance(x, SomeProtocol)` with `@runtime_checkable`.
6. **Virtual subclasses**: Respects `__instancecheck__` and ABC registration.

#### Constraints and Limitations

- Does not work with parameterized generics: `isinstance(x, list[int])` raises `TypeError`.
- Protocol checks only verify method presence, not signatures.

### Multiple Annotated Complete Step by Step Code Examples and Their Expected Outputs

#### Example 1: Basic Checks

```python
# step1: Exact type
print(isinstance(42, int))          # True
print(isinstance(42, str))          # False

# step2: Inheritance
class Animal:
    pass

class Dog(Animal):
    pass

d = Dog()
print(isinstance(d, Dog))           # True
print(isinstance(d, Animal))        # True (inheritance)
print(isinstance(d, str))           # False

# step3: Tuple of types
print(isinstance(42, (int, float)))  # True
print(isinstance(3.14, (int, float))) # True
print(isinstance("hi", (int, float))) # False
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
False
```

**Why**: `isinstance()` considers inheritance and accepts tuples of types.

#### Example 2: Type Narrowing

```python
def process(value: int | str) -> None:
    if isinstance(value, int):
        print(f"Integer: {value + 1}")
    else:
        print(f"String: {value.upper()}")

process(42)        # Integer: 43
process("hello")   # String: HELLO
```

**Expected Output**:
```
Integer: 43
String: HELLO
```

**Why**: The `isinstance` check narrows the type within each branch, enabling type-specific operations.

#### Example 3: Union Types (Python 3.10+)

```python
print(isinstance(42, int | str))    # True
print(isinstance("hi", int | str))  # True
print(isinstance(3.14, int | str))  # False
```

**Expected Output**:
```
True
True
False
```

**Why**: The `|` operator creates a union type that `isinstance` can check against.

### Real-World Cases

- **Input validation**: Checking that arguments are of expected types.
- **Polymorphism**: Dispatching behavior based on object type.
- **Serialization**: Determining how to serialize different object types.
- **Type narrowing**: Enabling safe operations within conditional branches.

### References

- `isinstance()` — Built-in Functions - https://docs.python.org/3/library/functions.html#isinstance
- `isinstance()` vs `type()` - https://realpython.com/python-isinstance/ (search result)
- Union Types - https://peps.python.org/pep-0604/

---

## 12. Explicit versus Implicit Conversion

### Definitions

**Core Definition**: Explicit conversion (casting) is performed by the programmer using constructor functions; implicit conversion (coercion) is performed automatically by Python, primarily for numeric widening.

**Technical Definition**: Python is a strongly typed language: it does not implicitly convert between most types. The primary exception is numeric promotion, where an `int` is automatically converted to a `float` when combined with a `float` in arithmetic operations. All other conversions require explicit calls to constructor functions such as `int()`, `float()`, or `str()`.

**Beginner-Friendly Explanation**: Explicit conversion is when you say "turn this string into a number" by calling `int("42")`. Implicit conversion is when Python does it for you — which only happens with numbers. For example, `1 + 2.5` automatically becomes `3.5` because Python promotes the integer to a float.

### Purposes

- To distinguish between programmer-controlled and automatic type changes.
- To understand when Python will and will not convert types automatically.
- To write code that explicitly converts types for clarity and safety.
- To avoid `TypeError` by knowing when explicit conversion is required.

### Syntax Rules and Structure

#### Complete General Syntax

```python
# Explicit conversion (casting)
new_value = target_type(original_value)

# Implicit conversion (automatic numeric promotion)
result = 1 + 2.5  # int promoted to float
```

**Component Breakdown**:
- `target_type` — the type constructor (`int`, `float`, `str`, etc.).
- `original_value` — the value to convert.
- Implicit promotion occurs in arithmetic operations between `int` and `float`.

#### Syntax Rules

1. **Explicit conversion is required for**: string↔number, list↔tuple↔set, etc.
2. **Implicit conversion occurs for**: `int` → `float` in mixed arithmetic.
3. **Implicit conversion also occurs for**: `bool` → `int` (since `bool` is a subclass of `int`).
4. **No implicit string conversion**: `"5" + 1` raises `TypeError`.
5. **No implicit container conversion**: `list + tuple` raises `TypeError`.

#### Constraints and Limitations

- Python does not implicitly convert between strings and numbers.
- Python does not implicitly convert between different container types.
- Implicit numeric promotion is the only common implicit conversion.

### Multiple Annotated Complete Step by Step Code Examples and Their Expected Outputs

#### Example 1: Explicit vs. Implicit

```python
# step1: Explicit conversion
num_str = "42"
num_int = int(num_str)       # Explicit: str → int
print(num_int + 1)           # 43

# step2: Implicit numeric promotion
result = 1 + 2.5             # Implicit: int → float
print(result)                # 3.5
print(type(result))          # <class 'float'>

# step3: No implicit string conversion
try:
    "5" + 1
except TypeError as e:
    print(f"TypeError: {e}")  # can only concatenate str (not "int") to str
```

**Expected Output**:
```
43
3.5
<class 'float'>
TypeError: can only concatenate str (not "int") to str
```

**Why**: `int("42")` is explicit; `1 + 2.5` is implicit numeric promotion; `"5" + 1` fails because Python does not implicitly convert.

#### Example 2: Bool as Int

```python
# step1: bool is a subclass of int
print(True + 1)    # 2 (implicit bool → int)
print(False + 1)   # 1

# step2: Explicit conversion
print(int(True))   # 1
print(int(False))  # 0
```

**Expected Output**:
```
2
1
1
0
```

**Why**: `bool` is a subclass of `int`, so `True` is implicitly treated as `1` in arithmetic.

#### Example 3: Container Conversion

```python
# step1: Explicit list-to-tuple conversion
lst = [1, 2, 3]
tup = tuple(lst)       # Explicit conversion
print(tup)             # (1, 2, 3)

# step2: No implicit container conversion
try:
    lst + tup
except TypeError as e:
    print(f"TypeError: {e}")  # can only concatenate list (not "tuple") to list
```

**Expected Output**:
```
(1, 2, 3)
TypeError: can only concatenate list (not "tuple") to list
```

**Why**: Converting between container types requires explicit calls; Python does not implicitly convert.

### Real-World Cases

- **User input**: Always a string; must be explicitly converted to numbers.
- **Arithmetic**: Implicit promotion simplifies mixed-type calculations.
- **Data pipelines**: Explicit conversion ensures data is in the expected format.
- **API responses**: JSON numbers may be int or float; explicit conversion ensures consistency.

### References

- Type Casting in Python - https://docs.python.org/3/library/functions.html (search result)
- Implicit vs Explicit Conversion - https://www.educative.io/ (search result)
- Numeric Types - https://docs.python.org/3/library/stdtypes.html#typesnumeric

---

## 13. Conversion Errors

### Definitions

**Core Definition**: Conversion errors are exceptions raised when a conversion function cannot produce a valid result from the given input. The two primary exceptions are `ValueError` and `TypeError`.

**Technical Definition**: `ValueError` is raised when a built-in operation or function receives an argument that has the right type but an inappropriate value (e.g., `int("abc")`). `TypeError` is raised when an operation or function is applied to an object of inappropriate type (e.g., `int(None)`). Additionally, `OverflowError` may be raised for numeric conversions that exceed the representable range.

**Beginner-Friendly Explanation**: When conversion fails, Python tells you why. If you try to turn `"hello"` into a number, you get a `ValueError` — the type is right (it's a string) but the value is wrong. If you try to turn `None` into a number, you get a `TypeError` — `None` is the wrong type entirely.

### Purposes

- To understand and handle conversion failures gracefully.
- To write robust code that validates input before conversion.
- To distinguish between value errors and type errors for appropriate error handling.
- To use `try`/`except` blocks to handle conversion errors.

### Syntax Rules and Structure

#### Complete General Syntax

```python
try:
    result = conversion_function(value)
except ValueError:
    # Handle wrong value for the type
except TypeError:
    # Handle wrong type
except (ValueError, TypeError):
    # Handle both
```

**Component Breakdown**:
- `try` — block containing the conversion.
- `except ValueError` — handles invalid values.
- `except TypeError` — handles invalid types.

#### Syntax Rules

1. **`ValueError`**: Right type, wrong value. Example: `int("abc")`, `float("xyz")`.
2. **`TypeError`**: Wrong type. Example: `int(None)`, `list(42)`.
3. **`OverflowError`**: Numeric value too large. Example: `float("1e1000")` may return `inf` instead.
4. **`ZeroDivisionError`**: Not a conversion error, but may occur during conversion.
5. **Catch specific exceptions**: Prefer `except (ValueError, TypeError)` over bare `except`.

#### Constraints and Limitations

- `ValueError` and `TypeError` are both subclasses of `Exception`.
- Not all conversion functions raise the same exceptions; check the specific function's documentation.

### Multiple Annotated Complete Step by Step Code Examples and Their Expected Outputs

#### Example 1: Distinguishing `ValueError` and `TypeError`

```python
# step1: ValueError — right type, wrong value
try:
    int("abc")
except ValueError as e:
    print(f"ValueError: {e}")  # invalid literal for int() with base 10: 'abc'

# step2: TypeError — wrong type
try:
    int(None)
except TypeError as e:
    print(f"TypeError: {e}")   # int() argument must be a string, a bytes-like object or a real number, not 'NoneType'

# step3: Catching both
try:
    int("abc")
except (ValueError, TypeError) as e:
    print(f"Conversion failed: {type(e).__name__}")
```

**Expected Output**:
```
ValueError: invalid literal for int() with base 10: 'abc'
TypeError: int() argument must be a string, a bytes-like object or a real number, not 'NoneType'
Conversion failed: ValueError
```

**Why**: `"abc"` has the right type but an invalid value; `None` has the wrong type entirely.

#### Example 2: Safe Conversion Function

```python
def safe_int(value, default=0):
    """Convert value to int, returning default on failure."""
    try:
        return int(value)
    except (ValueError, TypeError):
        return default

print(safe_int("42"))      # 42
print(safe_int("abc"))     # 0
print(safe_int(None))      # 0
print(safe_int("3.5"))     # 0
```

**Expected Output**:
```
42
0
0
0
```

**Why**: The function catches both `ValueError` and `TypeError`, returning a safe default.

#### Example 3: Conversion with Validation

```python
def parse_port(value: str) -> int:
    try:
        port = int(value)
    except ValueError:
        raise ValueError(f"Port must be an integer, got: {value}")
    if not (1 <= port <= 65535):
        raise ValueError(f"Port must be between 1 and 65535, got: {port}")
    return port

print(parse_port("8080"))   # 8080

try:
    parse_port("abc")
except ValueError as e:
    print(e)  # Port must be an integer, got: abc

try:
    parse_port("99999")
except ValueError as e:
    print(e)  # Port must be between 1 and 65535, got: 99999
```

**Expected Output**:
```
8080
Port must be an integer, got: abc
Port must be between 1 and 65535, got: 99999
```

**Why**: The function validates both the conversion result and the range, raising `ValueError` with clear messages.

### Real-World Cases

- **User input validation**: Catching conversion errors to show friendly messages.
- **Configuration parsing**: Validating that config values are of the correct type.
- **API data processing**: Handling malformed data gracefully.
- **Data pipelines**: Skipping or defaulting invalid records.

### References

- `ValueError` — Built-in Exceptions - https://docs.python.org/3/library/exceptions.html#ValueError
- `TypeError` — Built-in Exceptions - https://docs.python.org/3/library/exceptions.html#TypeError
- Errors and Exceptions - https://docs.python.org/3/tutorial/errors.html
- Parsing Strings to int and float in Python - https://codegym.cc/ (search result)

---

## References

- Built-in Functions — Python Documentation - https://docs.python.org/3/library/functions.html
- Built-in Types — Python Documentation - https://docs.python.org/3/library/stdtypes.html
- Truth Value Testing - https://docs.python.org/3/library/stdtypes.html#truth-value-testing
- Errors and Exceptions — Python Tutorial - https://docs.python.org/3/tutorial/errors.html
- `int()` - https://docs.python.org/3/library/functions.html#int
- `float()` - https://docs.python.org/3/library/functions.html#float
- `str()` - https://docs.python.org/3/library/functions.html#str
- `bool()` - https://docs.python.org/3/library/functions.html#bool
- `list()` - https://docs.python.org/3/library/functions.html#list
- `tuple()` - https://docs.python.org/3/library/functions.html#tuple
- `set()` - https://docs.python.org/3/library/functions.html#set
- `dict()` - https://docs.python.org/3/library/functions.html#dict
- `bytes()` - https://docs.python.org/3/library/functions.html#bytes
- `type()` - https://docs.python.org/3/library/functions.html#type
- `isinstance()` - https://docs.python.org/3/library/functions.html#isinstance
- `ValueError` - https://docs.python.org/3/library/exceptions.html#ValueError
- `TypeError` - https://docs.python.org/3/library/exceptions.html#TypeError