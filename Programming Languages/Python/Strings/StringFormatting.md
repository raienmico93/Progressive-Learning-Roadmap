# Python String Formatting: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Core Definition

String formatting is the process of constructing a new string by interpolating values into a template string. Python provides three principal mechanisms: the `%` operator (printf-style formatting), the `str.format()` method, and f-strings (formatted string literals). A fourth mechanism, `string.Template`, is available for simple substitution but is outside the scope of this cheat sheet.

### Technical Definition

String formatting in Python is implemented through the `__format__()` protocol, which allows objects to define their own formatting behaviour. The `%` operator is modelled on C's `printf()` syntax and supports a limited set of conversion types. The `str.format()` method and f-strings share the Format Specification Mini-Language, which provides fine-grained control over alignment, width, precision, signs, grouping, and numeric conversion types. F-strings, introduced in PEP 498, evaluate expressions at runtime and are the recommended mechanism for Python 3.6+.

### Beginner-Friendly Explanation

String formatting lets you insert values into text. You write a template with placeholders and tell Python what to put in each one. There are three main ways: the old `%` operator (like `"Hello %s" % name`), the `.format()` method (like `"Hello {}".format(name)`), and f-strings (like `f"Hello {name}"`). F-strings are the newest and easiest to read.

### Key Characteristics

- **Three coexisting mechanisms**: `%`, `str.format()`, and f-strings. F-strings are recommended for new code.
- **Shared format specification mini-language**: `str.format()` and f-strings use the same syntax after the `:` separator.
- **Runtime evaluation**: F-strings evaluate expressions at runtime, not at compile time.
- **`__format__()` protocol**: Objects can define their own formatting behaviour.
- **Extensible**: The `string.Formatter` class allows custom formatting behaviours.

### Prerequisites

- Python 3.6+ for f-strings; Python 3.8+ for the `=` debug specifier; Python 3.10+ for improved error messages.
- Basic understanding of variables, strings, and the `print()` function.
- Familiarity with numeric types (int, float) and the `datetime` module for date/time formatting.

### Related Programming Areas

- **Output formatting**: Console reports, logs, and tables.
- **Data serialization**: Generating CSV, JSON, and HTML.
- **Internationalization**: Locale-aware number and date formatting.
- **Debugging**: Self-documenting f-string expressions.

### Core Concepts / Features

The following sections cover each formatting mechanism using a uniform structure.

---

## 1. `%` Formatting (printf-Style)

### Definitions

**Core Definition**: The `%` operator performs string interpolation by treating the string on the left as a format string and the value on the right as the data to be formatted.

**Technical Definition**: String objects have one unique built-in operation: the `%` operator (modulo). Given `format % values` (where `format` is a string), the `%` conversion specifications in `format` are replaced with zero or more elements of `values`. The effect is similar to the `sprintf()` function in C. The formatting operations described here are modelled on C's `printf()` syntax and only support formatting of certain builtin types. There are no current plans to deprecate printf-style formatting, but it exhibits a variety of quirks that lead to common errors; newer alternatives are recommended.

**Beginner-Friendly Explanation**: The `%` operator is the oldest way to format strings in Python. You write placeholders like `%s` (for a string) or `%d` (for an integer) inside a string, then use `%` followed by the values to fill them in. For example, `"Hello %s" % "World"` gives `"Hello World"`.

### Purposes

- To interpolate a single value or a tuple of values into a string template.
- To perform C-style formatted output for numeric values.
- To inject dictionary values using mapping keys like `%(name)s`.
- To maintain compatibility with legacy code that uses printf-style formatting.
- To provide a concise syntax for simple formatting tasks.

### Syntax Rules and Structure

#### Complete General Syntax

```python
"%s" % value                    # Single value
"%s %d" % (str_val, int_val)    # Tuple of values
"%(name)s" % {"name": "Alice"}  # Dictionary mapping
"%f" % 3.14                     # Float formatting
```

**Component Breakdown**:
- `%` — the format specifier introducer.
- `s`, `d`, `f` — conversion types (string, integer, float).
- `(name)` — optional mapping key for dictionary injection.
- `values` — a single value, a tuple, or a mapping.

#### Conversion Types

| Specifier | Meaning |
|-----------|---------|
| `%s` | String (uses `str()`) |
| `%d` | Signed integer decimal |
| `%f` | Floating-point decimal |
| `%x` / `%X` | Hex (lowercase/uppercase) |
| `%o` | Octal |
| `%%` | Literal percent sign |

#### Syntax Rules

1. **Single value or tuple**: A single value can be passed directly; multiple values must be in a tuple.
2. **Tuple trap**: A single-element tuple must be written as `(value,)`; otherwise it is treated as multiple arguments.
3. **Dictionary mapping**: Use `%(key)s` to inject values from a dictionary.
4. **Width and precision**: `%5d` (minimum width 5), `%.2f` (2 decimal places), `%5.2f` (width 5, precision 2).
5. **Sign flags**: `%+d` (always show sign), `%-d` (left-justify).
6. **Padding**: `%05d` (zero-padded to width 5).

#### Constraints and Limitations

- Only supports int, str, and float (and types convertible to these).
- The tuple trap: a single tuple value must be wrapped in another tuple.
- Does not support named arguments in the modern sense; only dictionary mapping.
- No alignment controls beyond `-` for left-justification.
- Considered legacy; f-strings are recommended for new code.

### Multiple Annotated Complete Step by Step Code Examples and Their Expected Outputs

#### Example 1: Basic `%` Formatting

```python
# step1: Single string value
name = "Pete"
print("Hello %s" % name)          # Hello Pete

# step2: Integer value
num = 5
print("I have %d apples" % num)   # I have 5 apples

# step3: Multiple values (tuple)
first, last = "John", "Doe"
print("%s %s" % (first, last))    # John Doe

# step4: Float formatting
price = 3.14159
print("Price: %.2f" % price)      # Price: 3.14

# step5: Width and precision
print("|%10.2f|" % price)         # |      3.14|
print("|%-10.2f|" % price)        # |3.14      |
```

**Expected Output**:
```
Hello Pete
I have 5 apples
John Doe
Price: 3.14
|      3.14|
|3.14      |
```

**Why**: `%s` interpolates strings; `%d` interpolates integers; `%.2f` formats a float to two decimal places; `%10.2f` pads to a minimum width of 10.

#### Example 2: Dictionary Mapping

```python
# step1: Dictionary-based injection
data = {"name": "Alice", "age": 30}
print("Name: %(name)s, Age: %(age)d" % data)
# Name: Alice, Age: 30

# step2: Mixing mapping with tuple is not allowed
try:
    print("%(name)s %s" % {"name": "Alice"})
except KeyError as e:
    print(f"KeyError: {e}")
```

**Expected Output**:
```
Name: Alice, Age: 30
KeyError: 's'
```

**Why**: When the right operand is a dictionary, every format specifier must use a mapping key. Mixing mapping and positional specifiers raises `KeyError`.

#### Example 3: Tuple Trap and Escape

```python
# step1: Single tuple value must be wrapped
msg = "disk failure"
print("error: %s" % msg)          # error: disk failure

# step2: If msg is a tuple, it fails
msg_tuple = ("disk failure", 32)
try:
    print("error: %s" % msg_tuple)
except TypeError as e:
    print(f"TypeError: {e}")

# step3: Correct way — wrap in another tuple
print("error: %s" % (msg_tuple,))  # error: ('disk failure', 32)

# step4: Literal percent sign
print("100%% complete")            # 100% complete
```

**Expected Output**:
```
error: disk failure
TypeError: not all arguments converted during string formatting
error: ('disk failure', 32)
100% complete
```

**Why**: A tuple on the right side is interpreted as multiple arguments. Wrapping it in another tuple passes it as a single value. `%%` produces a literal percent sign.

### Real-World Cases

- **Legacy code maintenance**: Modifying existing Python 2/3 code that uses `%`.
- **Simple logging**: `"ERROR: %s" % message`.
- **C-style formatting**: Porting C code to Python.
- **Quick debugging**: `print("x = %d" % x)`.

### References

- printf-style String Formatting - https://docs.python.org/3/library/stdtypes.html#printf-style-string-formatting
- PEP 3101 – Advanced String Formatting - https://peps.python.org/pep-3101/
- Python String Formatting Cheatsheet - https://raw.githubusercontent.com/Yaya9256/python-cheatsheet/master/docs/cheatsheet/string-formatting.md

---

## 2. `str.format()` Method

### Definitions

**Core Definition**: The `str.format()` method performs variable substitution and value formatting by replacing `{}` placeholders in a string with arguments passed to the method.

**Technical Definition**: The `str.format()` method and the `Formatter` class share the same syntax for format strings. Format strings contain "replacement fields" surrounded by curly braces `{}`. Anything not contained in braces is literal text. The `format()` method takes any number of positional and keyword arguments and substitutes them into the replacement fields. The syntax is extensible through the `__format__()` method on the object being converted.

**Beginner-Friendly Explanation**: `.format()` is a more modern way to format strings. You write `{}` placeholders and pass the values to `.format()`. You can use positional references like `{0}` and `{1}`, named references like `{name}`, and even access attributes and indexes inside the braces.

### Purposes

- To interpolate positional and keyword arguments into a string.
- To support named references for readability.
- To access attributes and indexes inside replacement fields.
- To provide a more regular and extensible syntax than `%` formatting.
- To support dynamic format specifications via nested replacement fields.

### Syntax Rules and Structure

#### Complete General Syntax

```python
"Hello {}".format(name)                 # Automatic numbering
"Hello {0}".format(name)                # Explicit positional
"Hello {name}".format(name="Alice")     # Named argument
"{0.attr}".format(obj)                  # Attribute access
"{0[0]}".format(seq)                    # Index access
"{:{}}".format(value, spec)             # Nested format spec
```

**Component Breakdown**:
- `{}` — replacement field.
- `{0}` — positional reference by index.
- `{name}` — keyword reference by name.
- `{0.attr}` — attribute access on the first positional argument.
- `{0[0]}` — index access on the first positional argument.
- `{:{}}` — nested replacement field for dynamic format specification.

#### Syntax Rules

1. **Automatic numbering**: `"{} {}".format(a, b)` assigns `{}` in order.
2. **Explicit numbering**: `"{1} {0}".format(a, b)` allows reordering.
3. **Named references**: `"{name}".format(name="Alice")`.
4. **Attribute access**: `"{0.attr}"` accesses `attr` on the first argument.
5. **Index access**: `"{0[0]}"` accesses index 0 of the first argument.
6. **Nested fields**: `"{:{}}".format(value, spec)` substitutes `spec` first.
7. **Escaping braces**: `{{` and `}}` produce literal `{` and `}`.

#### Constraints and Limitations

- Attribute and index access do not allow arbitrary expressions (no function calls).
- Nested replacement fields cannot contain further nested fields (only one level).
- Cannot use literal curly braces as fill characters in f-strings or `.format()`.
- Verbose compared to f-strings.

### Multiple Annotated Complete Step by Step Code Examples and Their Expected Outputs

#### Example 1: Basic `.format()` Usage

```python
# step1: Automatic numbering
print("Hello I'm {}, my age is {}".format("John", 20))
# Hello I'm John, my age is 20

# step2: Explicit positional
print("Hello I'm {0}, my age is {1}".format("John", 20))
# Hello I'm John, my age is 20

# step3: Named arguments
print("Hello I'm {name}, my age is {age}".format(name="John", age=20))
# Hello I'm John, my age is 20

# step4: Reordering
print("{1} {0}".format("world", "Hello"))  # Hello world
```

**Expected Output**:
```
Hello I'm John, my age is 20
Hello I'm John, my age is 20
Hello I'm John, my age is 20
Hello world
```

**Why**: Automatic numbering assigns arguments in order; explicit numbering allows reordering; named arguments provide clarity.

#### Example 2: Attribute and Index Access

```python
class Point:
    def __init__(self, x, y):
        self.x = x
        self.y = y

p = Point(3, 4)
print("x = {0.x}, y = {0.y}".format(p))  # x = 3, y = 4

# step2: Index access
items = ["a", "b", "c"]
print("First: {0[0]}, Last: {0[2]}".format(items))
# First: a, Last: c

# step3: Dictionary access
data = {"name": "Alice", "age": 30}
print("{0[name]} is {0[age]}".format(data))
# Alice is 30
```

**Expected Output**:
```
x = 3, y = 4
First: a, Last: c
Alice is 30
```

**Why**: `{0.attr}` accesses attributes; `{0[key]}` accesses indexes or dictionary keys.

#### Example 3: Nested Format Specification

```python
# step1: Dynamic width
value = 42
width = 10
print("|{:{}}|".format(value, width))  # |        42|

# step2: Dynamic precision
pi = 3.14159
precision = 3
print("{:.{}f}".format(pi, precision))  # 3.142

# step3: Escaping braces
print("{{}} is a literal brace pair")  # {} is a literal brace pair
```

**Expected Output**:
```
|        42|
3.142
{} is a literal brace pair
```

**Why**: Nested `{}` fields allow the format specification itself to be dynamic. `{{` and `}}` escape literal braces.

### Real-World Cases

- **Template rendering**: Generating HTML or email templates.
- **Internationalization**: Formatting numbers and dates with locale-specific specifications.
- **Logging**: Structured log messages with named fields.
- **Report generation**: Aligning tabular data.

### References

- `str.format()` - https://docs.python.org/3/library/stdtypes.html#str.format
- Format String Syntax - https://docs.python.org/3/library/string.html#format-string-syntax
- PEP 3101 – Advanced String Formatting - https://peps.python.org/pep-3101/

---

## 3. F-Strings (Formatted String Literals)

### Definitions

**Core Definition**: An f-string is a string literal prefixed with `f` or `F` that contains replacement fields (expressions in curly braces) which are evaluated at runtime.

**Technical Definition**: F-strings, formally called "formatted string literals", were introduced in PEP 498 (Python 3.6). An f-string is a literal string, prefixed with `f` or `F`, which contains expressions inside braces. The expressions are replaced with their values. While other string literals always have a constant value, formatted strings are really expressions evaluated at run time. F-strings reuse much of the `str.format()` syntax and machinery, including the Format Specification Mini-Language.

**Beginner-Friendly Explanation**: F-strings are the modern, most readable way to format strings. You put an `f` before the opening quote, then put variables or expressions directly inside `{}`. For example, `f"Hello {name}"` gives you `"Hello Alice"` if `name` is `"Alice"`. You can even do calculations inside: `f"{a + b}"`.

### Purposes

- To embed expressions directly in string literals for maximum readability.
- To perform inline arithmetic and function calls inside strings.
- To use self-documenting debug expressions with `=`.
- To format dates, times, and numbers with concise syntax.
- To replace `%` and `.format()` with a single, consistent mechanism.

### Syntax Rules and Structure

#### Complete General Syntax

```python
f"Hello {name}"                    # Basic interpolation
f"{a + b}"                         # Inline expression
f"{value:.2f}"                     # Format specification
f"{obj.attr}"                      # Attribute access
f"{func()}"                        # Function call
f"{var=}"                          # Self-documenting (Python 3.8+)
f"{var!r}"                         # Conversion (!r = repr)
```

**Component Breakdown**:
- `f` or `F` — prefix that identifies an f-string.
- `{expression}` — replacement field containing a Python expression.
- `{expression:format_spec}` — format specification after the colon.
- `{expression!conversion}` — conversion (`!s`, `!r`, `!a`).
- `{expression=}` — self-documenting expression (Python 3.8+).

#### Syntax Rules

1. **Prefix required**: The `f` or `F` prefix must precede the opening quote.
2. **Curly braces**: Expressions are delimited by `{}`.
3. **Inline expressions**: Any valid Python expression can be used inside `{}`.
4. **Conversion flags**: `!r` calls `repr()`, `!s` calls `str()`, `!a` calls `ascii()`.
5. **Format specification**: After `:` you can use the full Format Specification Mini-Language.
6. **Self-documenting**: `f"{expr=}"` expands to the text of the expression, an equal sign, and the representation of the evaluated expression (Python 3.8+).
7. **Quote nesting**: In Python 3.12+, f-strings can reuse the same quote character inside expressions (e.g., `f"{"hello"}"`).
8. **No backslashes**: Before Python 3.12, backslashes were not allowed inside f-string expressions.

#### Constraints and Limitations

- F-strings are evaluated at runtime, not compile time.
- Before Python 3.12, the same quote character could not be reused inside expressions.
- Backslashes were not permitted inside f-string expressions before Python 3.12.
- Cannot be used as format strings for `str.format()` directly.
- Comments are not allowed inside f-string expressions (before Python 3.12).

### Multiple Annotated Complete Step by Step Code Examples and Their Expected Outputs

#### Example 1: Basic F-String Usage

```python
# step1: Simple variable interpolation
name = "Elizabeth"
print(f"Hello {name}!")           # Hello Elizabeth!

# step2: Inline arithmetic
a, b = 5, 10
print(f"Five plus ten is {a + b} and not {2 * (a + b)}.")
# Five plus ten is 15 and not 30.

# step3: Function calls
print(f"Length: {len(name)}")     # Length: 9

# step4: Attribute access
class Point:
    def __init__(self, x, y):
        self.x = x
        self.y = y
p = Point(3, 4)
print(f"Point: ({p.x}, {p.y})")   # Point: (3, 4)
```

**Expected Output**:
```
Hello Elizabeth!
Five plus ten is 15 and not 30.
Length: 9
Point: (3, 4)
```

**Why**: F-strings evaluate any expression inside `{}` and substitute the result directly into the string.

#### Example 2: Self-Documenting Expressions

```python
# step1: Basic self-documenting
foo = 42
print(f"{foo=}")                  # foo=42

# step2: With format specification
pi = 3.14159
print(f"{pi=:.2f}")               # pi=3.14

# step3: With expression
x, y = 3, 4
print(f"{x + y = }")              # x + y = 7
```

**Expected Output**:
```
foo=42
pi=3.14
x + y = 7
```

**Why**: The `=` specifier expands to the expression text, an equal sign, and the evaluated result, making debugging easier.

#### Example 3: Date and Format Specifications

```python
from datetime import datetime

now = datetime(2026, 9, 28, 14, 30, 45)

# step1: Date formatting
print(f"Date: {now:%Y-%m-%d}")          # Date: 2026-09-28

# step2: Time formatting
print(f"Time: {now:%H:%M:%S}")          # Time: 14:30:45

# step3: Full format
print(f"Full: {now:%Y-%m-%d %H:%M:%S}") # Full: 2026-09-28 14:30:45

# step4: Numeric formatting
price = 1234567.891
print(f"Price: {price:,.2f}")           # Price: 1,234,567.89
```

**Expected Output**:
```
Date: 2026-09-28
Time: 14:30:45
Full: 2026-09-28 14:30:45
Price: 1,234,567.89
```

**Why**: F-strings support the full Format Specification Mini-Language, including date/time format codes and numeric formatting with thousands separators.

### Real-World Cases

- **Logging and debugging**: Self-documenting expressions for quick inspection.
- **User-facing output**: Formatted messages and reports.
- **Data presentation**: Tables, charts, and dashboards.
- **API responses**: Constructing JSON-like strings.

### References

- PEP 498 – Literal String Interpolation - https://peps.python.org/pep-0498/
- Formatted String Literals - https://docs.python.org/3/reference/lexical_analysis.html#f-strings
- Python 3.8 What's New (Self-Documenting F-Strings) - https://docs.python.org/3/whatsnew/3.8.html#f-strings-support-for-self-documenting-expressions-and-debugging

---

## 4. Format Specifications

### Definitions

**Core Definition**: A format specification is a mini-language placed after the `:` separator inside a replacement field that controls how a value is presented (width, alignment, precision, type, etc.).

**Technical Definition**: The format specification is used within replacement fields contained within a format string to define how individual values are presented. It can also be passed directly to the built-in `format()` function. Most built-in types implement a common formatting mini-language, described by the following general form:

```
format_spec ::= [[fill]align][sign][#][0][width][grouping_option][.precision][type]
fill        ::= <any character>
align       ::= "<" | ">" | "=" | "^"
sign        ::= "+" | "-" | " "
width       ::= digit+
grouping_option ::= "_" | ","
precision   ::= digit+
type        ::= "b" | "c" | "d" | "e" | "E" | "f" | "F" | "g" | "G" | "n" | "o" | "s" | "x" | "X" | "%"
```

**Beginner-Friendly Explanation**: The format specification is everything after the colon in `{value:spec}`. It tells Python how to display the value — how wide the field should be, whether to align left or right, how many decimal places to show, and what kind of number to use (decimal, hex, percentage, etc.).

### Purposes

- To control the width and alignment of the output field.
- To specify the number of decimal places for floating-point numbers.
- To choose the numeric base (binary, octal, hex) for integers.
- To add thousands separators for readability.
- To control the sign display for numbers.
- To format values as percentages.

### Syntax Rules and Structure

#### Complete General Syntax

```
[[fill]align][sign][#][0][width][grouping_option][.precision][type]
```

**Component Breakdown**:
- `fill` — any character used for padding (default space).
- `align` — `<` (left), `>` (right), `=` (pad after sign), `^` (center).
- `sign` — `+` (always show sign), `-` (only negative, default), ` ` (space for positive).
- `#` — alternate form (e.g., `0x` prefix for hex).
- `0` — enable zero-padding.
- `width` — minimum field width (integer).
- `grouping_option` — `,` (comma separator) or `_` (underscore separator).
- `.precision` — number of digits after the decimal point (for floats) or maximum string length.
- `type` — conversion type (`b`, `c`, `d`, `e`, `f`, `g`, `n`, `o`, `s`, `x`, `X`, `%`).

#### Syntax Rules

1. **Fill and align**: If a fill character is specified, it must be followed by an alignment character.
2. **Zero-padding**: The `0` option enables zero-padding and is equivalent to `fill='0'` with `align='='`.
3. **Grouping**: `,` uses commas as thousands separators; `_` uses underscores.
4. **Precision for strings**: For string types, precision indicates the maximum number of characters.
5. **Precision for floats**: For `f` and `F`, precision is the number of digits after the decimal point.
6. **Type defaults**: If no type is specified, the default is `s` for strings and `d` for integers.

#### Constraints and Limitations

- Literal curly braces cannot be used as fill characters in f-strings or `.format()`.
- Not all format codes are supported by all platforms (date/time formatting).
- Precision for integers is not allowed (it raises `ValueError`).
- The `=` alignment is valid only for numeric types.

### Multiple Annotated Complete Step by Step Code Examples and Their Expected Outputs

#### Example 1: Basic Format Specifications

```python
# step1: Width and alignment
print(f"|{42:>10}|")      # |        42|
print(f"|{42:<10}|")      # |42        |
print(f"|{42:^10}|")      # |    42    |
print(f"|{42:=10}|")      # |        42| (pad after sign)

# step2: Zero-padding
print(f"{42:05d}")        # 00042

# step3: Sign display
print(f"{42:+d}")         # +42
print(f"{-42:+d}")        # -42
print(f"{42: d}")         #  42
```

**Expected Output**:
```
|        42|
|42        |
|    42    |
|        42|
00042
+42
-42
 42
```

**Why**: The format specification controls width, alignment, padding, and sign display.

#### Example 2: Numeric Conversion Types

```python
# step1: Decimal (default)
print(f"{255:d}")         # 255

# step2: Binary
print(f"{255:b}")         # 11111111

# step3: Octal
print(f"{255:o}")         # 377

# step4: Hexadecimal (lowercase)
print(f"{255:x}")         # ff

# step5: Hexadecimal (uppercase)
print(f"{255:X}")         # FF

# step6: Percentage
print(f"{0.25:%}")        # 25.000000%
print(f"{0.25:.1%}")      # 25.0%
```

**Expected Output**:
```
255
11111111
377
ff
FF
25.000000%
25.0%
```

**Why**: The type character determines the numeric base or conversion. `%` multiplies by 100 and appends a percent sign.

#### Example 3: Grouping and Precision

```python
# step1: Comma grouping
print(f"{1234567:,}")     # 1,234,567

# step2: Underscore grouping
print(f"{1234567:_}")     # 1_234_567

# step3: Grouping with float
print(f"{1234567.891:,.2f}")  # 1,234,567.89

# step4: Precision for strings
print(f"{'hello world':.5}")   # hello
```

**Expected Output**:
```
1,234,567
1_234_567
1,234,567.89
hello
```

**Why**: Grouping options add separators; precision limits decimal places for floats or character count for strings.

### Real-World Cases

- **Financial reports**: `f"{amount:,.2f}"` for currency.
- **Data tables**: Aligning columns with width and alignment specifications.
- **Scientific notation**: `f"{value:.2e}"` for exponential notation.
- **Percentages**: `f"{ratio:.1%}"` for percentage display.

### References

- Format Specification Mini-Language - https://docs.python.org/3/library/string.html#format-specification-mini-language
- PEP 378 – Format Specifier for Thousands Separator - https://peps.python.org/pep-0378/
- Format examples - https://docs.python.org/3/library/string.html#format-examples

---

## 5. Alignment

### Definitions

**Core Definition**: Alignment controls the horizontal placement of text within a field of a given width, using the anchors `<` (left), `>` (right), `^` (center), and `=` (pad after sign).

**Technical Definition**: When a width is specified in a format specification, the alignment character determines where the value is placed within the field. If no alignment is given, the default depends on the type: numbers are right-aligned and strings are left-aligned. The `=` alignment is valid only for numeric types and places the padding after the sign. Alignment can be preceded by a fill character.

**Beginner-Friendly Explanation**: Alignment lets you line up text in columns. `<` puts the text on the left, `>` on the right, and `^` in the middle. For example, `f"{'hello':>10}"` gives you `"     hello"`.

### Purposes

- To align text in fixed-width columns for tables and reports.
- To right-align numbers for readability.
- To center headings and titles.
- To control the placement of padding characters.
- To create visually consistent output.

### Syntax Rules and Structure

#### Complete General Syntax

```python
"{:>10}".format(value)     # Right-align in width 10
"{:<10}".format(value)     # Left-align in width 10
"{:^10}".format(value)     # Center in width 10
"{:=10}".format(value)     # Pad after sign (numeric)
"{:*>10}".format(value)    # Fill with '*' and right-align
```

**Component Breakdown**:
- `>` — right-align.
- `<` — left-align.
- `^` — center.
- `=` — pad after sign (numeric only).
- `*` — optional fill character.

#### Syntax Rules

1. **Default alignment**: Numbers are right-aligned; strings are left-aligned.
2. **Fill character**: Must precede the alignment character.
3. **Center alignment**: If the padding is odd, the extra character goes to the right.
4. **`=` alignment**: Only valid for numeric types; pads between the sign and the digits.
5. **No width, no alignment**: Alignment has no effect without a specified width.

#### Constraints and Limitations

- `=` alignment raises `ValueError` for non-numeric types.
- Fill character cannot be a literal curly brace in f-strings or `.format()`.
- Center alignment may appear asymmetric when the total padding is odd.

### Multiple Annotated Complete Step by Step Code Examples and Their Expected Outputs

#### Example 1: Basic Alignment

```python
# step1: Right-align
print(f"|{'hello':>10}|")    # |     hello|

# step2: Left-align
print(f"|{'hello':<10}|")    # |hello     |

# step3: Center
print(f"|{'hello':^10}|")    # |  hello   |

# step4: Numbers default to right-align
print(f"|{42:10}|")         # |        42|
```

**Expected Output**:
```
|     hello|
|hello     |
|  hello   |
|        42|
```

**Why**: The alignment character determines placement within the field; numbers default to right alignment.

#### Example 2: Custom Fill Characters

```python
# step1: Fill with asterisks
print(f"{'hello':*>10}")     # *****hello

# step2: Fill with dashes and center
print(f"{'hello':-^10}")     # --hello---

# step3: Fill with zeros (equivalent to 0 flag)
print(f"{42:0>5}")           # 00042
print(f"{42:05d}")           # 00042
```

**Expected Output**:
```
*****hello
--hello---
00042
00042
```

**Why**: Any character can be used as a fill character, placed before the alignment anchor.

#### Example 3: Numeric Alignment with `=`

```python
# step1: Pad after sign
print(f"{-42:=10d}")         # -       42
print(f"{42:=10d}")          #         42

# step2: Compare with right-align
print(f"{-42:>10d}")         #        -42

# step3: Zero-padding with sign
print(f"{-42:010d}")         # -000000042
```

**Expected Output**:
```
-       42
        42
       -42
-000000042
```

**Why**: `=` places padding after the sign, keeping the sign adjacent to the digits; `>` right-aligns the entire value including the sign.

### Real-World Cases

- **Table formatting**: Aligning columns in console reports.
- **Financial statements**: Right-aligning currency values.
- **Headings**: Centering titles in reports.
- **Progress bars**: Formatting labels and percentages.

### References

- Format Specification Mini-Language (align) - https://docs.python.org/3/library/string.html#format-specification-mini-language
- Format examples (align) - https://docs.python.org/3/library/string.html#format-examples

---

## 6. Width and Precision

### Definitions

**Core Definition**: Width is the minimum number of characters in the formatted field; precision is the number of digits after the decimal point for floats (or maximum characters for strings).

**Technical Definition**: The width is a decimal integer defining the minimum total field width, including any prefixes, separators, and other formatting characters. If not specified, the width is determined by the content. The precision is a decimal integer indicating how many digits should be displayed after the decimal point for presentation types `f` and `F`, or before and after the decimal point for `g` and `G`. For string presentation types, precision indicates the maximum field size — in other words, how many characters will be used from the field content.

**Beginner-Friendly Explanation**: Width sets the minimum size of the field. Precision controls how many decimal places a float shows (e.g., `:.2f` shows two decimal places) or how many characters of a string to use.

### Purposes

- To set a minimum field width for alignment and padding.
- To control the number of decimal places in floating-point output.
- To limit the number of characters displayed from a string.
- To ensure consistent output width across values.
- To control the significant digits in scientific notation.

### Syntax Rules and Structure

#### Complete General Syntax

```python
"{:width}".format(value)          # Minimum width
"{:.precisionf}".format(value)    # Float precision
"{:width.precisionf}".format(v)   # Both
"{:.precision}".format(string)    # String max length
```

**Component Breakdown**:
- `width` — minimum field width (integer).
- `.precision` — number of digits after decimal point (float) or max characters (string).
- `f` — fixed-point presentation type.

#### Syntax Rules

1. **Width is a minimum**: If the value is longer than the width, the full value is shown.
2. **Precision for floats**: Number of digits after the decimal point.
3. **Precision for strings**: Maximum number of characters.
4. **Precision for `g`/`G`**: Number of significant digits.
5. **Default float precision**: 6 digits after the decimal point.
6. **Zero-padding**: Use `0` flag or `0` fill character with alignment.

#### Constraints and Limitations

- Precision is not allowed for integer types (raises `ValueError`).
- Very large widths can produce very long strings.
- Precision for floats is limited to 50 for safety in some contexts.

### Multiple Annotated Complete Step by Step Code Examples and Their Expected Outputs

#### Example 1: Width and Precision for Floats

```python
pi = 3.141592653589793

# step1: Default precision
print(f"{pi}")            # 3.141592653589793

# step2: Two decimal places
print(f"{pi:.2f}")        # 3.14

# step3: Width and precision
print(f"|{pi:10.2f}|")    # |      3.14|

# step4: Zero-padded width
print(f"|{pi:08.2f}|")    # |00003.14|
```

**Expected Output**:
```
3.141592653589793
3.14
|      3.14|
|00003.14|
```

**Why**: `.2f` limits to two decimal places; `10.2f` combines width 10 with precision 2; `08.2f` zero-pads to width 8.

#### Example 2: Precision for Strings

```python
text = "hello world"

# step1: Default
print(f"{text}")          # hello world

# step2: Limit to 5 characters
print(f"{text:.5}")       # hello

# step3: Width and precision
print(f"|{text:15.5}|")   # |hello          |
```

**Expected Output**:
```
hello world
hello
|hello          |
```

**Why**: Precision for strings limits the number of characters; width pads the result.

#### Example 3: Width for Integers

```python
# step1: Default
print(f"{42}")            # 42

# step2: Minimum width
print(f"{42:5d}")         #    42

# step3: Zero-padded
print(f"{42:05d}")        # 00042

# step4: Width with sign
print(f"{-42:05d}")       # -0042
```

**Expected Output**:
```
42
   42
00042
-0042
```

**Why**: Width sets the minimum field size; zero-padding fills with zeros; the sign is preserved.

### Real-World Cases

- **Financial reports**: `f"{price:.2f}"` for two decimal places.
- **Scientific data**: `f"{value:.3e}"` for three significant digits.
- **Tabular output**: Combining width and precision for aligned columns.
- **String truncation**: `f"{text:.20}"` to limit displayed characters.

### References

- Format Specification Mini-Language (width, precision) - https://docs.python.org/3/library/string.html#format-specification-mini-language
- Format examples (precision) - https://docs.python.org/3/library/string.html#format-examples

---

## 7. Numeric Formatting

### Definitions

**Core Definition**: Numeric formatting controls the presentation of numbers, including base conversion, thousands separators, sign display, and percentage conversion.

**Technical Definition**: The type character in the format specification determines the presentation of numeric values. Available types include `b` (binary), `c` (character), `d` (decimal), `e`/`E` (scientific), `f`/`F` (fixed-point), `g`/`G` (general), `n` (locale-aware), `o` (octal), `x`/`X` (hexadecimal), and `%` (percentage). The grouping options `,` and `_` add thousands separators. The sign options `+`, `-`, and ` ` control sign display.

**Beginner-Friendly Explanation**: Numeric formatting lets you display numbers in different ways: as binary, octal, hexadecimal, scientific notation, or with commas for thousands. You can also force the sign to always show and format as a percentage.

### Purposes

- To display integers in different bases (binary, octal, hex).
- To format numbers with thousands separators for readability.
- To control the sign display (always positive, negative only, space for positive).
- To format numbers as percentages.
- To use scientific notation for very large or very small numbers.
- To use locale-aware formatting with the `n` type.

### Syntax Rules and Structure

#### Complete General Syntax

```python
"{:b}".format(255)        # Binary
"{:o}".format(255)        # Octal
"{:x}".format(255)        # Hexadecimal (lowercase)
"{:X}".format(255)        # Hexadecimal (uppercase)
"{:d}".format(255)        # Decimal
"{:,}".format(1234567)    # Comma separator
"{:_}".format(1234567)    # Underscore separator
"{:%}".format(0.25)       # Percentage
"{:e}".format(12345.678)  # Scientific notation
"{:+d}".format(42)        # Always show sign
"{: d}".format(42)        # Space for positive
```

#### Syntax Rules

1. **Base conversion**: `b`, `o`, `x`, `X` convert integers to other bases.
2. **Grouping**: `,` and `_` add thousands separators (underscore is valid for integers and floats in Python 3.6+).
3. **Percentage**: `%` multiplies by 100 and appends a percent sign.
4. **Scientific notation**: `e` and `E` use exponential notation.
5. **Sign options**: `+` always shows sign, `-` shows only negative (default), ` ` shows a space for positive numbers.
6. **Alternate form**: `#` adds base prefixes (`0b`, `0o`, `0x`).

#### Constraints and Limitations

- Precision is not allowed for integer types.
- The `n` type is locale-aware and may behave differently on different platforms.
- Grouping with `_` is not supported for all presentation types.

### Multiple Annotated Complete Step by Step Code Examples and Their Expected Outputs

#### Example 1: Base Conversion

```python
# step1: Binary
print(f"{255:b}")         # 11111111

# step2: Octal
print(f"{255:o}")         # 377

# step3: Hexadecimal
print(f"{255:x}")         # ff
print(f"{255:X}")         # FF

# step4: Alternate form with prefixes
print(f"{255:#b}")        # 0b11111111
print(f"{255:#o}")        # 0o377
print(f"{255:#x}")        # 0xff
```

**Expected Output**:
```
11111111
377
ff
FF
0b11111111
0o377
0xff
```

**Why**: The type character selects the base; `#` adds the base prefix.

#### Example 2: Thousands Separators

```python
# step1: Comma separator
print(f"{1234567:,}")     # 1,234,567

# step2: Underscore separator
print(f"{1234567:_}")     # 1_234_567

# step3: With float
print(f"{1234567.891:,.2f}")  # 1,234,567.89

# step4: With scientific notation
print(f"{1234567.891:.2e}")   # 1.23e+06
```

**Expected Output**:
```
1,234,567
1_234_567
1,234,567.89
1.23e+06
```

**Why**: `,` and `_` add grouping separators; `.2f` controls decimal places; `.2e` uses scientific notation.

#### Example 3: Signs and Percentages

```python
# step1: Always show sign
print(f"{42:+d}")         # +42
print(f"{-42:+d}")        # -42

# step2: Space for positive
print(f"{42: d}")         #  42
print(f"{-42: d}")        # -42

# step3: Percentage
print(f"{0.25:%}")        # 25.000000%
print(f"{0.25:.1%}")      # 25.0%
print(f"{0.125:.2%}")     # 12.50%
```

**Expected Output**:
```
+42
-42
 42
-42
25.000000%
25.0%
12.50%
```

**Why**: The sign option controls sign display; `%` multiplies by 100 and appends a percent sign.

### Real-World Cases

- **Financial reports**: `f"{amount:,.2f}"` for currency.
- **Data displays**: Hex colors, binary flags, octal permissions.
- **Scientific computing**: `f"{value:.3e}"` for scientific notation.
- **Statistics**: `f"{ratio:.1%}"` for percentages.

### References

- Format Specification Mini-Language (type) - https://docs.python.org/3/library/string.html#format-specification-mini-language
- PEP 378 – Format Specifier for Thousands Separator - https://peps.python.org/pep-0378/
- Format examples (numeric) - https://docs.python.org/3/library/string.html#format-examples

---

## 8. Date/Time Formatting

### Definitions

**Core Definition**: Date/time formatting uses the `strftime` format codes to interpolate `datetime` objects directly into strings, producing formatted temporal output.

**Technical Definition**: `date`, `datetime`, and `time` objects all support a `strftime(format)` method, to create a string representing the time under the control of an explicit format string. When used with f-strings or `.format()`, the format specification after the colon is passed to the object's `__format__()` method, which in turn uses the same format codes as `strftime()`. The format codes are platform-dependent because Python calls the platform C library's `strftime` function.

**Beginner-Friendly Explanation**: You can format dates and times directly inside f-strings by putting a `:` and then date format codes like `%Y-%m-%d`. For example, `f"{now:%Y-%m-%d}"` gives you `"2026-09-28"`.

### Purposes

- To format dates and times in a human-readable form.
- To produce ISO 8601 date strings.
- To extract specific components (year, month, day, hour, minute, second).
- To format timestamps for logging.
- To combine date and time formatting with other string content.

### Syntax Rules and Structure

#### Complete General Syntax

```python
f"{datetime_obj:%Y-%m-%d}"           # Date only
f"{datetime_obj:%H:%M:%S}"           # Time only
f"{datetime_obj:%Y-%m-%d %H:%M:%S}"  # Full timestamp
f"{datetime_obj:%A, %B %d, %Y}"      # Long format
```

**Component Breakdown**:
- `%Y` — 4-digit year.
- `%m` — 2-digit month (01-12).
- `%d` — 2-digit day (01-31).
- `%H` — hour (00-23).
- `%M` — minute (00-59).
- `%S` — second (00-59).
- `%A` — full weekday name.
- `%B` — full month name.

#### Common Format Codes

| Code | Meaning | Example |
|------|---------|---------|
| `%Y` | Year with century | 2026 |
| `%y` | Year without century | 26 |
| `%m` | Month (zero-padded) | 09 |
| `%B` | Full month name | September |
| `%b` | Abbreviated month name | Sep |
| `%d` | Day of month | 28 |
| `%A` | Full weekday name | Monday |
| `%a` | Abbreviated weekday name | Mon |
| `%H` | Hour (24-hour) | 14 |
| `%I` | Hour (12-hour) | 02 |
| `%M` | Minute | 30 |
| `%S` | Second | 45 |
| `%f` | Microsecond | 123456 |
| `%p` | AM/PM | PM |
| `%z` | UTC offset | +0000 |
| `%Z` | Timezone name | UTC |

#### Syntax Rules

1. **F-string syntax**: `f"{obj:%format}"` where `format` uses `strftime` codes.
2. **`.format()` syntax**: `"{:%format}".format(obj)`.
3. **`strftime()` method**: `obj.strftime("%format")`.
4. **Platform-dependent**: The full set of format codes varies across platforms.
5. **Date vs. time**: Date objects should not use time codes; time objects should not use date codes.
6. **Microseconds**: `%f` displays microseconds (6 digits).

#### Constraints and Limitations

- Platform-dependent format codes; not all codes work on all systems.
- Date and time objects have different valid format codes.
- The `%z` and `%Z` codes may produce empty strings for naive datetime objects.
- Locale-specific output for `%A`, `%B`, etc.

### Multiple Annotated Complete Step by Step Code Examples and Their Expected Outputs

#### Example 1: Basic Date/Time Formatting

```python
from datetime import datetime

now = datetime(2026, 9, 28, 14, 30, 45, 123456)

# step1: Date only
print(f"{now:%Y-%m-%d}")           # 2026-09-28

# step2: Time only
print(f"{now:%H:%M:%S}")           # 14:30:45

# step3: Full timestamp
print(f"{now:%Y-%m-%d %H:%M:%S}")  # 2026-09-28 14:30:45

# step4: Long format
print(f"{now:%A, %B %d, %Y}")      # Monday, September 28, 2026
```

**Expected Output**:
```
2026-09-28
14:30:45
2026-09-28 14:30:45
Monday, September 28, 2026
```

**Why**: F-strings pass the format specification to the datetime object's `__format__()` method, which uses `strftime` codes.

#### Example 2: Using `strftime()` and `.format()`

```python
from datetime import datetime

now = datetime(2026, 9, 28, 14, 30, 45)

# step1: strftime() method
print(now.strftime("%Y-%m-%d"))
# 2026-09-28

# step2: .format() method
print("{:%Y-%m-%d}".format(now))
# 2026-09-28

# step3: All three approaches produce the same result
print(f"{now:%Y-%m-%d}" == now.strftime("%Y-%m-%d") == "{:%Y-%m-%d}".format(now))
# True
```

**Expected Output**:
```
2026-09-28
2026-09-28
True
```

**Why**: All three approaches use the same underlying format codes.

#### Example 3: ISO 8601 and Custom Formats

```python
from datetime import datetime

now = datetime(2026, 9, 28, 14, 30, 45, 123456)

# step1: ISO 8601
print(f"{now:%Y-%m-%dT%H:%M:%S}")  # 2026-09-28T14:30:45

# step2: With microseconds
print(f"{now:%Y-%m-%d %H:%M:%S.%f}")  # 2026-09-28 14:30:45.123456

# step3: 12-hour format
print(f"{now:%I:%M %p}")              # 02:30 PM

# step4: Combined with other text
print(f"Report generated on {now:%B %d, %Y} at {now:%I:%M %p}")
# Report generated on September 28, 2026 at 02:30 PM
```

**Expected Output**:
```
2026-09-28T14:30:45
2026-09-28 14:30:45.123456
02:30 PM
Report generated on September 28, 2026 at 02:30 PM
```

**Why**: Date/time formatting can be combined with other string content and supports a wide range of format codes.

### Real-World Cases

- **Logging**: Timestamped log entries.
- **Reports**: Date-stamped report titles.
- **APIs**: ISO 8601 formatted timestamps.
- **User interfaces**: Displaying dates in user-friendly formats.
- **Data export**: Formatting dates for CSV and JSON.

### References

- `strftime()` and `strptime()` Format Codes - https://docs.python.org/3/library/datetime.html#strftime-and-strptime-format-codes
- `datetime` — Basic date and time types - https://docs.python.org/3/library/datetime.html
- F-String Date Formatting (PEP 498) - https://peps.python.org/pep-0498/

---

## References

- String Formatting — Python Standard Library - https://docs.python.org/3/library/stdtypes.html#printf-style-string-formatting
- Format String Syntax - https://docs.python.org/3/library/string.html#format-string-syntax
- Format Specification Mini-Language - https://docs.python.org/3/library/string.html#format-specification-mini-language
- Format examples - https://docs.python.org/3/library/string.html#format-examples
- PEP 498 – Literal String Interpolation - https://peps.python.org/pep-0498/
- PEP 3101 – Advanced String Formatting - https://peps.python.org/pep-3101/
- PEP 378 – Format Specifier for Thousands Separator - https://peps.python.org/pep-0378/
- Python 3.8 What's New (Self-Documenting F-Strings) - https://docs.python.org/3/whatsnew/3.8.html#f-strings-support-for-self-documenting-expressions-and-debugging
- `str.format()` - https://docs.python.org/3/library/stdtypes.html#str.format
- `datetime` — Basic date and time types - https://docs.python.org/3/library/datetime.html
- `strftime()` and `strptime()` Format Codes - https://docs.python.org/3/library/datetime.html#strftime-and-strptime-format-codes
- Python String Formatting Cheatsheet - https://raw.githubusercontent.com/Yaya9256/python-cheatsheet/master/docs/cheatsheet/string-formatting.md