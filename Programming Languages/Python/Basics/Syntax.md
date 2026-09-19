# Python Syntax Fundamentals

Python's syntax is designed to be readable, consistent, and close to plain English. This section covers the building blocks of Python code: statements, expressions, indentation, comments, identifiers, keywords, literals, and formatting conventions.

---

## Statements

A **statement** is an instruction that tells Python to do something. Statements are the basic units of execution in a program.

Common types of statements:

| Type | Example |
|---|---|
| Assignment | `x = 10` |
| Expression | `print("hi")` |
| Conditional | `if x > 0:` |
| Loop | `for i in range(5):` |
| Import | `import math` |
| Return | `return x` |
| Pass | `pass` |
| Break / Continue | `break`, `continue` |
| Function / Class definition | `def f():`, `class A:` |

Example:

```python
x = 5                # assignment statement
y = x + 3            # assignment with expression
print(y)             # expression statement
if y > 5:            # compound statement
    print("big")
```

A statement is usually written on one line, but compound statements contain a header and an indented body.

---

## Expressions

An **expression** is a combination of values, variables, operators, and function calls that produces a value.

Examples:

```python
2 + 3
x * y
len("hello")
max(1, 2, 3)
a and b or c
[x**2 for x in range(5)]
```

Key differences between statements and expressions:

| Aspect | Statement | Expression |
|---|---|---|
| Purpose | Performs an action | Produces a value |
| Example | `x = 5` | `x + 5` |
| Can be printed | Not usually | Yes |
| Can appear in `if` | No | Yes |

Example:

```python
x = 5           # statement
print(x + 3)    # x + 3 is an expression inside a print statement
```

Python also has the **walrus operator** `:=` that lets you assign inside an expression:

```python
if (n := len("hello")) > 3:
    print(f"Length is {n}")
```

---

## Indentation

Unlike many languages that use braces `{}` to group code, Python uses **indentation** to define blocks.

Rules:

- Use **spaces** (4 spaces per level is standard, per PEP 8).
- Do not mix tabs and spaces.
- All lines in the same block must have the same indentation.

Example:

```python
if True:
    print("Inside block")
    print("Still inside")
print("Outside block")
```

Incorrect indentation causes `IndentationError` or `TabError`.

```python
if True:
print("oops")   # IndentationError
```

---

## Code Blocks

A **code block** is a group of statements that run together. Blocks are introduced by a colon `:` and are defined by indentation.

Constructs that create blocks:

- `if`, `elif`, `else`
- `for`, `while`
- `def`, `class`
- `try`, `except`, `finally`
- `with`
- `match` / `case` (Python 3.10+)

Example:

```python
def greet(name):
    if name:
        print(f"Hello, {name}")
    else:
        print("Hello, stranger")

for i in range(3):
    print(i)
```

Nested blocks increase indentation:

```python
for i in range(3):
    if i % 2 == 0:
        print(f"{i} is even")
```

---

## Whitespace

Python is whitespace-sensitive in two ways:

1. **Indentation** — defines code blocks (must be consistent).
2. **Spacing inside expressions** — optional but recommended for readability.

PEP 8 spacing guidelines:

| Context | Rule | Example |
|---|---|---|
| Around binary operators | One space | `x = a + b` |
| After comma | One space | `f(a, b, c)` |
| Around `=` in keyword args | No spaces | `f(x=1, y=2)` |
| Inside parentheses/brackets | No spaces | `(a + b)`, `[1, 2]` |
| Around `:` in slices | No spaces (or equal on both sides) | `a[1:3]` |
| After `#` in comments | One space | `# comment` |

Blank lines:

- Use **2 blank lines** around top-level functions and classes.
- Use **1 blank line** between methods inside a class.

---

## Comments

Comments describe code and are ignored by the interpreter.

### Single-Line Comments

Start with `#`. Everything after `#` on that line is ignored.

```python
# This is a comment
x = 10  # inline comment
```

### Documentation Comments (Docstrings)

Docstrings are string literals used to document modules, functions, classes, and methods. They are placed immediately after the definition.

```python
def add(a, b):
    """Return the sum of a and b."""
    return a + b
```

Multi-line docstring:

```python
def divide(a, b):
    """
    Divide a by b.

    Args:
        a (float): Numerator.
        b (float): Denominator.

    Returns:
        float: The result of a / b.
    """
    return a / b
```

Access a docstring:

```python
print(add.__doc__)
help(add)
```

Docstring conventions: **PEP 257**. Common styles include Google, NumPy, and reStructuredText.

---

## Identifiers

An **identifier** is a name given to a variable, function, class, module, or other object.

Rules:

- Must start with a letter (A–Z, a–z) or underscore `_`.
- Followed by letters, digits, or underscores.
- Cannot be a keyword.
- Case-sensitive.
- Unicode letters are allowed (PEP 3131), but ASCII is recommended.

Valid identifiers:

```python
name
_user
count1
total_amount
CamelCase
MAX_SIZE
```

Invalid identifiers:

```python
1name      # starts with digit
my-var     # hyphen not allowed
class      # keyword
total$     # invalid character
```

Special identifiers:

| Name | Meaning |
|---|---|
| `_` | Often a throwaway variable or last REPL result |
| `__name__` | Module name (dunder) |
| `__init__` | Constructor method |
| `_private` | Convention for internal use |
| `__mangled` | Name-mangled in classes |

---

## Keywords

**Keywords** are reserved words with special meaning. They cannot be used as identifiers.

Python 3 keywords:

```text
False      None       True       and        as
assert     async      await      break      class
continue   def        del        elif       else
except     finally    for        from       global
if         import     in         is         lambda
nonlocal   not        or         pass       raise
return     try        while      with       yield
```

Check with:

```python
import keyword
print(keyword.kwlist)
print(keyword.iskeyword("for"))   # True
```

Soft keywords (contextual, Python 3.9+):

```text
_    case    match    type
```

These can still be used as identifiers in most contexts.

---

## Literals

A **literal** is a value written directly in source code.

### Numeric Literals

```python
42            # int
3.14          # float
1_000_000     # underscore separator
0b1010        # binary
0o17          # octal
0xFF          # hexadecimal
1e-3          # scientific notation
2 + 3j        # complex
```

### String Literals

```python
"hello"
'hello'
"""multi
line"""
'''also multi
line'''
r"C:\path"         # raw string
b"bytes"           # bytes literal
f"{x} items"       # f-string (formatted)
```

### Boolean and None

```python
True
False
None
```

### Collection Literals

```python
[1, 2, 3]                  # list
(1, 2, 3)                  # tuple
{1, 2, 3}                  # set
{"a": 1, "b": 2}           # dict
```

### Ellipsis

```python
...
```

Used as a placeholder or in type hints and slicing.

---

## Naming Conventions

Follow **PEP 8**:

| Item | Convention | Example |
|---|---|---|
| Variable | snake_case | `total_price` |
| Function | snake_case | `calculate_total` |
| Method | snake_case | `get_value` |
| Class | CapWords (PascalCase) | `HttpClient` |
| Constant | UPPER_SNAKE_CASE | `MAX_RETRIES` |
| Module | short snake_case | `math_utils` |
| Package | short lowercase | `mypackage` |
| Private | leading underscore | `_helper` |
| Name-mangled | double leading underscore | `__internal` |
| Dunder | double leading and trailing | `__init__` |

Avoid:

- Single-character names except for counters (`i`, `j`) or `_`.
- Names that shadow built-ins: `list`, `dict`, `str`, `id`, `type`.
- Ambiguous names like `l`, `O`, `I`.

---

## Case Sensitivity

Python is **case-sensitive**.

```python
name = "Alice"
Name = "Bob"
NAME = "Carol"
print(name)   # Alice
```

The three variables above are distinct.

Same applies to:

- Keywords: `True` is valid, `true` is not.
- Built-ins: `print` works; `Print` raises `NameError`.
- Imports: `import math` ≠ `import Math`.

---

## Line Continuation

Long lines can be broken in two ways.

### Implicit Continuation

Anything inside parentheses, brackets, or braces can span multiple lines.

```python
total = (
    1 + 2 +
    3 + 4
)

items = [
    "apple",
    "banana",
    "cherry",
]
```

This is the preferred method.

### Explicit Continuation

Use a backslash `\` at the end of a line to continue it.

```python
total = 1 + 2 + \
        3 + 4
```

Use explicit continuation only when necessary; implicit is cleaner and safer.

You cannot place a comment after a backslash:

```python
x = 1 + \
    2   # valid
# x = 1 + \  # invalid comment after continuation
```

---

## Multiple Statements

Multiple statements can be placed on one line using a semicolon `;`.

```python
a = 1; b = 2; print(a + b)
```

This is legal but discouraged unless the statements are trivially related. Prefer one statement per line.

Compound statements can also be written on one line:

```python
if x > 0: print("positive")
```

This is allowed but hurts readability. Prefer:

```python
if x > 0:
    print("positive")
```

A block cannot contain multiple statements on one line after the header unless separated by `;`:

```python
if x > 0: y = 1; z = 2   # allowed but not recommended
```

---

## Pythonic Formatting

**Pythonic** code is idiomatic, readable, and consistent with community conventions.

### PEP 8 Highlights

- 4 spaces per indentation level.
- Maximum line length: **79 characters** for code, **72** for docstrings/comments.
- Two blank lines around top-level definitions.
- One blank line between methods.
- Imports at the top, one per line, grouped:
  ```python
  import os
  import sys

  import requests

  from mypackage import mymodule
  ```
- Avoid wildcard imports (`from module import *`).
- Use spaces around operators, not inside brackets.
- No trailing whitespace.
- End files with a single newline.

### Pythonic Code Examples

Loops:

```python
# Non-Pythonic
for i in range(len(items)):
    print(items[i])

# Pythonic
for item in items:
    print(item)
```

Enumerate:

```python
# Non-Pythonic
i = 0
for item in items:
    print(i, item)
    i += 1

# Pythonic
for i, item in enumerate(items):
    print(i, item)
```

Swap:

```python
a, b = b, a
```

List comprehension:

```python
squares = [x**2 for x in range(10)]
```

Truthiness:

```python
# Non-Pythonic
if len(items) != 0:
    ...

# Pythonic
if items:
    ...
```

F-strings:

```python
name = "Alice"
print(f"Hello, {name}!")
```

Context managers:

```python
with open("file.txt") as f:
    data = f.read()
```

### Tools for Pythonic Formatting

| Tool | Purpose |
|---|---|
| `black` | Opinionated code formatter |
| `ruff` | Fast linter and formatter |
| `flake8` | Linter combining pycodestyle, pyflakes, mccabe |
| `isort` | Sort imports |
| `mypy` / `pyright` | Static type checking |
| `pylint` | Comprehensive linter |
| `pre-commit` | Run checks before commits |

Example:

```bash
pip install black ruff
black script.py
ruff check script.py
```

### Example: Well-Formatted Python

```python
"""Module docstring explaining the file."""

import math


MAX_ITEMS = 100


def area_of_circle(radius):
    """Return the area of a circle given its radius."""
    if radius < 0:
        raise ValueError("radius must be non-negative")
    return math.pi * radius ** 2


class Circle:
    """A simple circle."""

    def __init__(self, radius):
        self.radius = radius

    def area(self):
        return area_of_circle(self.radius)


if __name__ == "__main__":
    c = Circle(3)
    print(f"Area: {c.area():.2f}")
```

---

## Summary

| Concept | Key Point |
|---|---|
| Statement | Instruction that performs an action |
| Expression | Produces a value |
| Indentation | Defines code blocks (4 spaces) |
| Code block | Group of statements after `:` |
| Whitespace | Significant for blocks; stylized for readability |
| Comments | `#` for single-line; docstrings for documentation |
| Identifier | Name for variables, functions, classes |
| Keyword | Reserved word; cannot be used as identifier |
| Literal | Direct value in code |
| Naming | Follow PEP 8 conventions |
| Case sensitivity | `name`, `Name`, `NAME` are different |
| Line continuation | Implicit via brackets; explicit via `\` |
| Multiple statements | Semicolons allowed but discouraged |
| Pythonic formatting | PEP 8, clean, idiomatic code |

Mastering these fundamentals builds the foundation for writing clean, readable, and maintainable Python code.