# Python Documentation: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Core Definition

Python documentation is the practice of embedding structured, human-readable descriptions directly within source code (as docstrings) and generating external reference materials from those descriptions using automated tooling. It encompasses conventions for docstring structure, documentation generation tools, and best practices for writing documentation that serves both developers and end users.

### Technical Definition

Python documentation is implemented through docstrings — string literals that appear as the first statement in a module, function, class, or method definition, becoming the `__doc__` special attribute of that object. PEP 257 defines the semantics and conventions for docstrings, while the `typing` module's type hints (PEP 484) complement docstrings by providing machine-checkable type information. Documentation generation tools such as Sphinx (with the `autodoc` and `napoleon` extensions) and MkDocs (with the `mkdocstrings` plugin) parse docstrings to produce HTML, PDF, and other output formats. Modern Python documentation practice integrates docstring style conventions (Google, NumPy, Sphinx reStructuredText) with static type annotations for comprehensive API references.

### Beginner-Friendly Explanation

Documentation in Python means writing descriptions of what your code does, right inside the code itself. These descriptions are called docstrings. When you write a docstring at the top of a function, anyone who uses that function can read what it does. Tools can also collect all these docstrings and turn them into a website — that's how most Python libraries' documentation is made. Writing good documentation is like leaving clear instructions for the next person who reads your code, even if that person is future you.

### Key Characteristics

- **Embedded**: Documentation lives alongside the code it describes.
- **Structured**: Docstrings follow conventions (Google, NumPy, reStructuredText) for consistency.
- **Machine-readable**: Tools can parse docstrings to generate external documentation.
- **Runtime-accessible**: Docstrings are available at runtime via the `__doc__` attribute.
- **Complementary to type hints**: Type annotations provide machine-checkable information; docstrings provide human-readable context.
- **Tool-supported**: Sphinx, MkDocs, and MkDocStrings automate documentation generation.

### Prerequisites

- Python 3.x installed (some tools require Python 3.6+).
- Basic understanding of Python functions, classes, modules, and string literals.
- Familiarity with a text editor or IDE.
- (Optional) Knowledge of Markdown or reStructuredText for writing documentation.

### Related Programming Areas

- **Static type checking**: Type hints complement docstrings.
- **API design**: Public interfaces require clear documentation.
- **Software maintenance**: Documentation reduces onboarding time.
- **Open-source development**: Documentation is essential for library adoption.

### Core Concepts / Features

The following sections cover each core concept using a uniform structure.

---

## 1. Docstrings

### Definitions

**Core Definition**: A docstring is a string literal that occurs as the first statement in a module, function, class, or method definition, serving as the documentation for that object.

**Technical Definition**: Per PEP 257, a docstring is a string literal that becomes the `__doc__` special attribute of the enclosing module, class, function, or method. Docstrings are delimited by triple double quotes (`"""..."""`) and follow conventions for one-line and multi-line structures. PEP 257 explicitly states it does not touch on any markup syntax within docstrings; the choice of markup (Google, NumPy, reStructuredText) is a separate concern handled by tools like Sphinx Napoleon.

**Beginner-Friendly Explanation**: A docstring is just a string you put right after a function or class definition to explain what it does. Python automatically attaches it to the function so you can see it later with `help()` or `.__doc__`.

### Purposes

- To provide human-readable documentation for modules, classes, functions, and methods.
- To make documentation available at runtime via the `__doc__` attribute.
- To enable automatic documentation generation from source code.
- To standardize documentation structure across a project.
- To complement type hints with descriptive information.

### Syntax Rules and Structure

#### Complete General Syntax

```python
# One-line docstring
def kos_root():
    """Return the pathname of the KOS root directory."""
    ...

# Multi-line docstring
def complex(real=0.0, imag=0.0):
    """Form a complex number.

    Keyword arguments:
    real -- the real part (default 0.0)
    imag -- the imaginary part (default 0.0)
    """
    ...
```

**Component Breakdown**:
- `"""` — triple double quotes delimit the docstring.
- Summary line — one-line description ending in a period.
- Blank line — separates summary from detailed description.
- Detailed description — optional elaboration.
- `"""` — closing triple double quotes.

#### Docstring Style Comparison

| Aspect | Google Style | NumPy Style | Sphinx reST |
|--------|-------------|-------------|-------------|
| Section separator | Indentation | Underlines | Directives (`:param:`) |
| Readability (short) | Easier to read | Requires more vertical space | Dense, harder to read |
| Readability (long) | Uses more horizontal space | Easier to read | Dense |
| Tool support | Napoleon | Napoleon, numpydoc | Native Sphinx |

The main difference between Google and NumPy styles is that Google uses indentation to separate sections, whereas NumPy uses underlines. Google style tends to be easier to read for short and simple docstrings, whereas NumPy style tends to be easier to read for long and in-depth docstrings.

#### Syntax Rules

1. **Triple double quotes**: Always use `"""triple double quotes"""` around docstrings.
2. **One-line docstrings**: Closing quotes on the same line; no blank line before or after; phrase as a command.
3. **Multi-line docstrings**: Summary line, blank line, detailed description; closing quotes on a line by themselves.
4. **No trailing whitespace**: Do not rely on significant trailing whitespace; it is visually indistinguishable and easily stripped.
5. **`__doc__` attribute**: The docstring becomes the object's `__doc__` attribute.
6. **Consistency**: Choose one style (Google, NumPy, or reST) and use it consistently throughout the project.
7. **PEP 257 compliance**: Follow PEP 257's high-level structure conventions.

#### Constraints and Limitations

- PEP 257 contains conventions, not laws; violating them causes "dirty looks" but not syntax errors.
- Docstring markup (Google, NumPy, reST) is outside PEP 257's scope.
- Trailing whitespace conventions are fragile and easily broken by editors.
- The two styles (Google and NumPy) should not be mixed within a project.

### Multiple Annotated Complete Step by Step Code Examples and Their Expected Outputs

#### Example 1: One-Line and Multi-Line Docstrings

```python
# step1: One-line docstring
def add(a: int, b: int) -> int:
    """Return the sum of a and b."""
    return a + b

# step2: Multi-line docstring
def divide(dividend: float, divisor: float) -> float:
    """Divide dividend by divisor.

    Args:
        dividend: The number to be divided.
        divisor: The number to divide by.

    Returns:
        The quotient of dividend divided by divisor.

    Raises:
        ZeroDivisionError: If divisor is zero.
    """
    if divisor == 0:
        raise ZeroDivisionError("Cannot divide by zero")
    return dividend / divisor

# step3: Access docstrings
print(add.__doc__)                    # Return the sum of a and b.
print(divide.__doc__)
# Divide dividend by divisor.
#
#     Args:
#         dividend: The number to be divided.
#         divisor: The number to divide by.
#
#     Returns:
#         The quotient of dividend divided by divisor.
#
#     Raises:
#         ZeroDivisionError: If divisor is zero.

print(add(3, 4))                      # 7
print(divide(10, 2))                  # 5.0
```

**Expected Output**:
```
Return the sum of a and b.
Divide dividend by divisor.

    Args:
        dividend: The number to be divided.
        divisor: The number to divide by.

    Returns:
        The quotient of dividend divided by divisor.

    Raises:
        ZeroDivisionError: If divisor is zero.

7
5.0
```

**Why**: The one-line docstring provides a concise summary; the multi-line docstring includes sections for arguments, return value, and exceptions, following Google style.

#### Example 2: Google vs. NumPy Style

```python
# step1: Google style
def connect_google(host: str, port: int = 5432) -> Connection:
    """Open a connection to the database server.

    Args:
        host: The database host address.
        port: The port to connect to. Defaults to 5432.

    Returns:
        A connection object to the database.

    Raises:
        ConnectionError: If the connection cannot be established.
    """
    ...

# step2: NumPy style
def connect_numpy(host: str, port: int = 5432) -> Connection:
    """Open a connection to the database server.

    Parameters
    ----------
    host : str
        The database host address.
    port : int, optional
        The port to connect to (default is 5432).

    Returns
    -------
    Connection
        A connection object to the database.

    Raises
    ------
    ConnectionError
        If the connection cannot be established.
    """
    ...
```

**Expected Output**: No output (illustrative docstrings).

**Why**: Google style uses indentation-based sections; NumPy style uses underlined section headers. NumPy style requires more vertical space but is often easier to read for detailed descriptions.

#### Example 3: Sphinx reStructuredText Style

```python
def connect_rst(host: str, port: int = 5432) -> Connection:
    """Open a connection to the database server.

    :param host: The database host address.
    :type host: str
    :param port: The port to connect to. Defaults to 5432.
    :type port: int
    :returns: A connection object to the database.
    :rtype: Connection
    :raises ConnectionError: If the connection cannot be established.
    """
    ...
```

**Expected Output**: No output (illustrative docstring).

**Why**: Sphinx reStructuredText style uses explicit directives (`:param:`, `:type:`, `:returns:`, `:raises:`) which are native to Sphinx but dense to read in source code.

### Real-World Cases

- **Library development**: Docstrings are the primary source for API documentation.
- **IDE tooltips**: Docstrings appear as tooltips in IDEs.
- **Interactive help**: `help()` and `__doc__` display docstrings.
- **Automatic documentation**: Sphinx and MkDocs generate documentation from docstrings.

### References

- PEP 257 – Docstring Conventions - https://peps.python.org/pep-0257/
- sphinx.ext.napoleon — Support for NumPy and Google style docstrings - https://www.sphinx-doc.org/en/master/usage/extensions/napoleon.html
- Google Python Style Guide (Docstrings) - https://google.github.io/styleguide/pyguide.html#38-comments-and-docstrings
- NumPy Docstring Standard - https://numpydoc.readthedocs.io/en/latest/format.html
- Example Google Style Python Docstrings - https://www.sphinx-doc.org/en/master/usage/extensions/example_google.html
- Example NumPy Style Python Docstrings - https://www.sphinx-doc.org/en/master/usage/extensions/example_numpy.html

---

## 2. Module Documentation

### Definitions

**Core Definition**: Module documentation is a docstring placed at the absolute top of a Python module file (after any `from __future__` imports) that describes the module's purpose, contents, and usage.

**Technical Definition**: A module docstring is the first statement in a module file, stored in the module's `__doc__` attribute. It should outline the module's core purpose, its public API, and any dependencies or usage notes. Per PEP 257, the docstring for a module should generally list the classes, exceptions, and functions (and any other objects) that are exported by the module, with a one-line summary of each.

**Beginner-Friendly Explanation**: A module docstring is the "front page" of a Python file. It tells readers what the file is for and what's inside it. You put it at the very top of the file, before any imports or code.

### Purposes

- To describe the module's overall purpose and scope.
- To list the public API (classes, functions, exceptions) the module exports.
- To document dependencies and usage notes.
- To provide a high-level overview before diving into details.
- To be included in generated documentation as the module's main page.

### Syntax Rules and Structure

#### Complete General Syntax

```python
"""Module docstring.

This module provides utilities for processing text data.

Exported:
    - TextProcessor: Class for processing text.
    - tokenize: Function to tokenize text.
    - TokenError: Exception raised on tokenization failure.

Dependencies:
    - re: For regular expression operations.
    - collections: For specialized container types.
"""

import re
from collections import Counter
```

**Component Breakdown**:
- First statement in the file (after `from __future__` imports).
- Summary line: one-line description.
- Extended description: purpose, features, usage.
- Exported items: classes, functions, exceptions.
- Dependencies: third-party or standard library modules.

#### Syntax Rules

1. **First statement**: The module docstring must be the first statement in the file (after any `from __future__` imports).
2. **Triple double quotes**: Use `"""..."""`.
3. **Summary line**: One-line description ending in a period.
4. **Extended description**: Purpose, features, usage examples.
5. **Exported items**: List public classes, functions, and exceptions.
6. **Dependencies**: Note any significant dependencies.
7. **`__doc__` attribute**: The docstring becomes the module's `__doc__`.

#### Constraints and Limitations

- Must be the first statement; comments and `from __future__` imports may precede it.
- Cannot contain code; it is a string literal.
- Its content is a convention, not enforced by Python.

### Multiple Annotated Complete Step by Step Code Examples and Their Expected Outputs

#### Example 1: Basic Module Docstring

```python
"""Text processing utilities.

This module provides functions for tokenizing and analyzing text.

Exported:
    - tokenize(text): Split text into words.
    - word_count(text): Count occurrences of each word.

Usage:
    >>> from text_utils import tokenize, word_count
    >>> tokenize("Hello, World!")
    ['hello', 'world']
"""

import re


def tokenize(text: str) -> list[str]:
    """Split text into lowercase words."""
    return re.findall(r'\w+', text.lower())


def word_count(text: str) -> dict[str, int]:
    """Count occurrences of each word in text."""
    from collections import Counter
    return dict(Counter(tokenize(text)))


if __name__ == "__main__":
    print(tokenize("Hello, World!"))
    print(word_count("Hello, World!"))
```

**Expected Output**:
```
['hello', 'world']
{'hello': 1, 'world': 1}
```

**Why**: The module docstring describes the module's purpose, exported functions, and usage example.

#### Example 2: Accessing Module Docstring

```python
"""Example module for demonstrating module documentation."""

# step1: Access module docstring
print(__doc__)  # Example module for demonstrating module documentation.

# step2: Import and access another module's docstring
import os
print(os.__doc__[:50])  # OS routines for NT or Posix depending on what syst
```

**Expected Output**:
```
Example module for demonstrating module documentation.
OS routines for NT or Posix depending on what system we're on.
```

**Why**: Module docstrings are accessible via the module's `__doc__` attribute.

#### Example 3: Comprehensive Module Documentation

```python
"""Advanced text analysis module.

This module provides tools for tokenizing, counting, and analyzing
text data. It is designed for processing large text corpora efficiently.

Public API:
    - TextAnalyzer: Main class for text analysis.
    - TokenError: Exception raised on tokenization failure.
    - DEFAULT_ENCODING: Default encoding for file reading.

Dependencies:
    - re: Standard library regular expressions.
    - collections.Counter: For counting word frequencies.

Example:
    >>> analyzer = TextAnalyzer()
    >>> analyzer.analyze("Hello world hello")
    {'hello': 2, 'world': 1}
"""

DEFAULT_ENCODING = "utf-8"


class TokenError(Exception):
    """Raised when tokenization fails."""
    pass


class TextAnalyzer:
    """Analyze text data for word frequencies."""

    def analyze(self, text: str) -> dict[str, int]:
        """Return word frequency counts."""
        import re
        from collections import Counter
        words = re.findall(r'\w+', text.lower())
        return dict(Counter(words))
```

**Expected Output**: No output (illustrative module).

**Why**: The module docstring comprehensively documents the public API, dependencies, and usage.

### Real-World Cases

- **Library packages**: Module docstrings serve as the main documentation page.
- **Utility modules**: Describing the purpose and usage of utility functions.
- **Configuration modules**: Documenting settings and their effects.

### References

- PEP 257 – Docstring Conventions (Module Docstrings) - https://peps.python.org/pep-0257/#multi-line-docstrings
- Python Modules — Python Tutorial - https://docs.python.org/3/tutorial/modules.html

---

## 3. Function Documentation

### Definitions

**Core Definition**: Function documentation is a docstring that describes a function's behaviour, including its parameters, return value, side effects, and exceptions raised.

**Technical Definition**: A function docstring appears immediately after the `def` statement and before the function body. Per PEP 257, the docstring for a function or method should summarize its behaviour, document its arguments, return values, side effects, exceptions raised, and restrictions on when it can be called. Optional arguments should be indicated, and it should be documented whether keyword arguments are part of the interface.

**Beginner-Friendly Explanation**: A function docstring tells you how to use the function: what arguments it takes, what it returns, and what errors it might raise. It's the instruction manual for the function.

### Purposes

- To document the function's purpose and behaviour.
- To describe each parameter's type and meaning.
- To specify the return value's type and meaning.
- To document exceptions that may be raised.
- To note side effects (e.g., file writes, state changes).
- To provide usage examples.

### Syntax Rules and Structure

#### Complete General Syntax (Google Style)

```python
def function_name(param1: type, param2: type = default) -> return_type:
    """One-line summary.

    Extended description of the function's behaviour.

    Args:
        param1: Description of param1.
        param2: Description of param2. Defaults to default.

    Returns:
        Description of the return value.

    Raises:
        ExceptionType: Description of when this exception is raised.

    Examples:
        >>> function_name(1, 2)
        3
    """
    ...
```

**Component Breakdown**:
- `"""` — docstring delimiter.
- Summary line — one-line description ending in a period.
- Extended description — optional elaboration.
- `Args:` — parameter descriptions (Google style).
- `Returns:` — return value description.
- `Raises:` — exception documentation.
- `Examples:` — usage examples.

#### Syntax Rules

1. **Summary line**: One-line description of the function's behaviour.
2. **Args section**: Document each parameter, its type, and meaning.
3. **Returns section**: Document the return value's type and meaning.
4. **Raises section**: Document exceptions that may be raised.
5. **Examples section**: Provide doctest-compatible usage examples.
6. **No `self`**: Do not include `self` in the `Args` section for methods.
7. **Consistency**: Use the same docstring style throughout the project.

#### Constraints and Limitations

- Docstrings are for external documentation; internal comments should explain implementation details.
- Function docstrings should be sufficient for using the function as a black box.
- Over-documenting trivial functions can be noise.

### Multiple Annotated Complete Step by Step Code Examples and Their Expected Outputs

#### Example 1: Basic Function Docstring

```python
def calculate_area(length: float, width: float) -> float:
    """Calculate the area of a rectangle.

    Args:
        length: The length of the rectangle.
        width: The width of the rectangle.

    Returns:
        The area of the rectangle (length * width).

    Raises:
        ValueError: If length or width is negative.
    """
    if length < 0 or width < 0:
        raise ValueError("Dimensions must be non-negative")
    return length * width

print(calculate_area(5, 3))   # 15.0
print(calculate_area.__doc__)
# Calculate the area of a rectangle.
#
#     Args:
#         length: The length of the rectangle.
#         width: The width of the rectangle.
#
#     Returns:
#         The area of the rectangle (length * width).
#
#     Raises:
#         ValueError: If length or width is negative.
```

**Expected Output**:
```
15.0
Calculate the area of a rectangle.

    Args:
        length: The length of the rectangle.
        width: The width of the rectangle.

    Returns:
        The area of the rectangle (length * width).

    Raises:
        ValueError: If length or width is negative.
```

**Why**: The docstring documents the function's purpose, parameters, return value, and exceptions.

#### Example 2: Function with Examples (Doctest)

```python
def factorial(n: int) -> int:
    """Return the factorial of n.

    Args:
        n: A non-negative integer.

    Returns:
        The factorial of n (n!).

    Raises:
        ValueError: If n is negative.

    Examples:
        >>> factorial(5)
        120
        >>> factorial(0)
        1
    """
    if n < 0:
        raise ValueError("n must be non-negative")
    return 1 if n == 0 else n * factorial(n - 1)

print(factorial(5))   # 120
print(factorial(0))   # 1
```

**Expected Output**:
```
120
1
```

**Why**: The `Examples` section provides doctest-compatible examples that can be automatically tested.

#### Example 3: NumPy Style Function Docstring

```python
def linear_regression(x: list[float], y: list[float]) -> tuple[float, float]:
    """Fit a linear regression to the data.

    Parameters
    ----------
    x : list of float
        The independent variable values.
    y : list of float
        The dependent variable values.

    Returns
    -------
    slope : float
        The slope of the regression line.
    intercept : float
        The intercept of the regression line.

    Raises
    ------
    ValueError
        If x and y have different lengths.

    Examples
    --------
    >>> linear_regression([1, 2, 3], [2, 4, 6])
    (2.0, 0.0)
    """
    if len(x) != len(y):
        raise ValueError("x and y must have the same length")
    n = len(x)
    mean_x = sum(x) / n
    mean_y = sum(y) / n
    slope = sum((x[i] - mean_x) * (y[i] - mean_y) for i in range(n)) / \
            sum((x[i] - mean_x) ** 2 for i in range(n))
    intercept = mean_y - slope * mean_x
    return slope, intercept

print(linear_regression([1, 2, 3], [2, 4, 6]))  # (2.0, 0.0)
```

**Expected Output**:
```
(2.0, 0.0)
```

**Why**: NumPy style uses underlined section headers (`Parameters`, `Returns`, `Raises`, `Examples`) and is well-suited for detailed function documentation.

### Real-World Cases

- **API endpoints**: Documenting request/response behaviour.
- **Utility functions**: Documenting parameters and return values.
- **Scientific computing**: NumPy style is standard in scientific Python libraries.
- **Library development**: Function docstrings are the primary API reference.

### References

- PEP 257 – Docstring Conventions (Function Docstrings) - https://peps.python.org/pep-0257/#multi-line-docstrings
- Google Python Style Guide (Function Docstrings) - https://google.github.io/styleguide/pyguide.html#38-comments-and-docstrings
- NumPy Docstring Standard (Functions) - https://numpydoc.readthedocs.io/en/latest/format.html

---

## 4. Class Documentation

### Definitions

**Core Definition**: Class documentation is a docstring that describes a class's purpose, its constructor parameters, its public attributes, and its methods.

**Technical Definition**: A class docstring appears immediately after the `class` statement and before the class body. It should describe the class's overall purpose, its initialization contract (constructor parameters), instance attributes, and public methods. Per PEP 257, the docstring for a class should summarize its behaviour and document its public methods and instance variables.

**Beginner-Friendly Explanation**: A class docstring tells you what the class is for, what you need to pass to create an instance, and what attributes and methods instances will have.

### Purposes

- To describe the class's purpose and responsibilities.
- To document constructor parameters and initialization contracts.
- To describe instance attributes and their types.
- To list and briefly describe public methods.
- To provide usage examples.
- To document class-level constants and class variables.

### Syntax Rules and Structure

#### Complete General Syntax (Google Style)

```python
class ClassName:
    """One-line summary of the class.

    Extended description of the class's purpose and responsibilities.

    Attributes:
        attr1: Description of attr1.
        attr2: Description of attr2.

    Args:
        param1: Description of constructor parameter.
        param2: Description of constructor parameter.
    """

    def __init__(self, param1: type, param2: type) -> None:
        """Initialize the class.

        Args:
            param1: Description of param1.
            param2: Description of param2.
        """
        self.attr1 = param1
        self.attr2 = param2
```

**Component Breakdown**:
- Class docstring — describes the class.
- `Attributes:` — documents instance attributes (Google style).
- `Args:` — documents constructor parameters.
- `__init__` docstring — documents initialization.

#### Syntax Rules

1. **Class docstring**: Summary line, extended description, attributes, and constructor parameters.
2. **`__init__` docstring**: Documents constructor parameters (may duplicate class docstring).
3. **Attributes section**: Document public instance attributes.
4. **Methods**: Each method has its own docstring.
5. **Class variables**: Document with `ClassVar` in type hints.
6. **Examples section**: Provide usage examples.

#### Constraints and Limitations

- Class docstrings should focus on external behaviour, not implementation details.
- Attribute documentation may become stale if not maintained.
- Some projects prefer documenting `__init__` separately from the class docstring.

### Multiple Annotated Complete Step by Step Code Examples and Their Expected Outputs

#### Example 1: Basic Class Documentation

```python
class BankAccount:
    """A simple bank account with deposit and withdrawal operations.

    Attributes:
        owner: The name of the account owner.
        balance: The current account balance.
        transaction_count: Number of transactions performed.
    """

    def __init__(self, owner: str, initial_balance: float = 0.0) -> None:
        """Initialize a bank account.

        Args:
            owner: The name of the account owner.
            initial_balance: The starting balance. Defaults to 0.0.
        """
        self.owner = owner
        self.balance = initial_balance
        self.transaction_count = 0

    def deposit(self, amount: float) -> None:
        """Deposit money into the account.

        Args:
            amount: The amount to deposit. Must be positive.

        Raises:
            ValueError: If amount is not positive.
        """
        if amount <= 0:
            raise ValueError("Deposit amount must be positive")
        self.balance += amount
        self.transaction_count += 1

    def withdraw(self, amount: float) -> None:
        """Withdraw money from the account.

        Args:
            amount: The amount to withdraw. Must be positive and
                not exceed the current balance.

        Raises:
            ValueError: If amount is not positive or exceeds balance.
        """
        if amount <= 0:
            raise ValueError("Withdrawal amount must be positive")
        if amount > self.balance:
            raise ValueError("Insufficient funds")
        self.balance -= amount
        self.transaction_count += 1

# step1: Create and use account
account = BankAccount("Alice", 100.0)
account.deposit(50)
account.withdraw(30)
print(account.balance)          # 120.0
print(account.transaction_count) # 2
print(BankAccount.__doc__)
# A simple bank account with deposit and withdrawal operations.
#
#     Attributes:
#         owner: The name of the account owner.
#         balance: The current account balance.
#         transaction_count: Number of transactions performed.
```

**Expected Output**:
```
120.0
2
A simple bank account with deposit and withdrawal operations.

    Attributes:
        owner: The name of the account owner.
        balance: The current account balance.
        transaction_count: Number of transactions performed.
```

**Why**: The class docstring documents the class's purpose, attributes, and constructor parameters; methods have their own docstrings.

#### Example 2: NumPy Style Class Documentation

```python
class Matrix:
    """A simple 2D matrix with basic operations.

    Parameters
    ----------
    rows : int
        Number of rows.
    cols : int
        Number of columns.
    fill : float, optional
        Initial fill value (default 0.0).

    Attributes
    ----------
    data : list of list of float
        The matrix data as a nested list.
    shape : tuple of int
        The matrix dimensions (rows, cols).

    Examples
    --------
    >>> m = Matrix(2, 3)
    >>> m.shape
    (2, 3)
    """

    def __init__(self, rows: int, cols: int, fill: float = 0.0) -> None:
        """Initialize the matrix.

        Parameters
        ----------
        rows : int
            Number of rows.
        cols : int
            Number of columns.
        fill : float, optional
            Initial fill value (default 0.0).
        """
        self.data = [[fill for _ in range(cols)] for _ in range(rows)]
        self.shape = (rows, cols)

m = Matrix(2, 3)
print(m.shape)  # (2, 3)
```

**Expected Output**:
```
(2, 3)
```

**Why**: NumPy style class documentation separates parameters, attributes, and examples with underlined headers.

#### Example 3: Dataclass Documentation

```python
from dataclasses import dataclass

@dataclass
class Point:
    """A 2D point with x and y coordinates.

    Attributes:
        x: The x-coordinate.
        y: The y-coordinate.
    """
    x: float
    y: float

    def distance_to(self, other: "Point") -> float:
        """Calculate Euclidean distance to another point.

        Args:
            other: The other point.

        Returns:
            The Euclidean distance between the two points.
        """
        return ((self.x - other.x) ** 2 + (self.y - other.y) ** 2) ** 0.5

p1 = Point(0, 0)
p2 = Point(3, 4)
print(p1.distance_to(p2))  # 5.0
print(Point.__doc__)
# A 2D point with x and y coordinates.
#
#     Attributes:
#         x: The x-coordinate.
#         y: The y-coordinate.
```

**Expected Output**:
```
5.0
A 2D point with x and y coordinates.

    Attributes:
        x: The x-coordinate.
        y: The y-coordinate.
```

**Why**: Dataclasses use class docstrings to document their attributes; the `@dataclass` decorator generates `__init__` from type annotations.

### Real-World Cases

- **Domain models**: Documenting business entities.
- **Data structures**: Documenting custom containers.
- **API clients**: Documenting service client classes.
- **Framework components**: Documenting base classes and mixins.

### References

- PEP 257 – Docstring Conventions (Class Docstrings) - https://peps.python.org/pep-0257/#multi-line-docstrings
- Google Python Style Guide (Class Docstrings) - https://google.github.io/styleguide/pyguide.html#38-comments-and-docstrings
- NumPy Docstring Standard (Classes) - https://numpydoc.readthedocs.io/en/latest/format.html

---

## 5. API Documentation

### Definitions

**Core Definition**: API documentation is external-facing documentation that describes the public interface of a software library or service, including its entry points, parameters, return values, exceptions, versioning, and usage examples.

**Technical Definition**: API documentation encompasses the reference material generated from docstrings (via Sphinx, MkDocs, etc.) plus supplementary documentation such as tutorials, migration guides, and architectural overviews. It documents the public API — the subset of modules, classes, functions, and constants intended for external consumption — and is maintained across versions.

**Beginner-Friendly Explanation**: API documentation is the manual for your library or service. It tells users how to use your code: what functions to call, what arguments to pass, what to expect back, and what can go wrong. It also explains how the API changes between versions.

### Purposes

- To provide a complete reference for the public API.
- To guide users in getting started (tutorials, quickstart).
- To document versioning and migration paths.
- To provide architectural context and design rationale.
- To reduce support burden by answering common questions.
- To enable self-service adoption of the library or service.

### Syntax Rules and Structure

#### API Documentation Components

| Component | Purpose | Example |
|-----------|---------|---------|
| Reference | Complete API listing | Sphinx autodoc output |
| Tutorial | Step-by-step guide | "Getting Started" page |
| How-to guides | Task-oriented instructions | "How to authenticate" |
| Explanation | Conceptual background | "Architecture overview" |
| Changelog | Version history | "What's new in v2.0" |
| Migration guide | Upgrade instructions | "Migrating from v1 to v2" |

#### Versioning Strategies

| Strategy | Description | Example |
|----------|-------------|---------|
| URL path | Version in URL path | `/api/v1/users` |
| Header | Version in HTTP header | `X-API-Version: 2` |
| Query parameter | Version as query param | `/api/users?version=2` |
| Media type | Version in Accept header | `Accept: application/vnd.api.v2+json` |

#### Syntax Rules

1. **Document the public API**: Use `__all__` to define what is public.
2. **Include examples**: Provide runnable examples for each major feature.
3. **Document versioning**: Explain the versioning scheme and deprecation policy.
4. **Maintain multiple versions**: Keep documentation for supported versions.
5. **Use consistent terminology**: Define terms in a glossary.
6. **Provide migration guides**: Help users upgrade between versions.
7. **Include architectural context**: Explain how components fit together.

#### Constraints and Limitations

- API documentation requires ongoing maintenance.
- Versioning multiplies documentation effort.
- Examples must be tested to remain accurate.
- Documentation can become stale if not integrated into the development workflow.

### Multiple Annotated Complete Step by Step Code Examples and Their Expected Outputs

#### Example 1: API Reference with Sphinx Autodoc

```python
# myapi/__init__.py
"""My API Library.

A simple library for demonstrating API documentation.

Example:
    >>> from myapi import greet
    >>> greet("World")
    'Hello, World!'
"""

__all__ = ["greet", "User"]

def greet(name: str) -> str:
    """Return a greeting for the given name.

    Args:
        name: The name to greet.

    Returns:
        A greeting string.

    Example:
        >>> greet("Alice")
        'Hello, Alice!'
    """
    return f"Hello, {name}!"

class User:
    """Represents a user in the system.

    Attributes:
        name: The user's name.
        email: The user's email address.
    """

    def __init__(self, name: str, email: str) -> None:
        """Initialize a user.

        Args:
            name: The user's name.
            email: The user's email address.
        """
        self.name = name
        self.email = email

print(greet("Alice"))  # Hello, Alice!
print(User.__doc__)
# Represents a user in the system.
#
#     Attributes:
#         name: The user's name.
#         email: The user's email address.
```

**Expected Output**:
```
Hello, Alice!
Represents a user in the system.

    Attributes:
        name: The user's name.
        email: The user's email address.
```

**Why**: The public API is defined by `__all__`; docstrings document each public object; Sphinx autodoc can generate a reference from this code.

#### Example 2: Versioning Documentation

```python
# api/v1/users.py
"""API v1 — User endpoints.

.. deprecated:: 2.0
    Use API v2 (``/api/v2/users``) instead.
"""

def get_user(user_id: int) -> dict:
    """Get a user by ID.

    Args:
        user_id: The user's ID.

    Returns:
        A dictionary with user data.

    .. deprecated:: 2.0
        Use ``GET /api/v2/users/{id}`` instead.
    """
    return {"id": user_id, "name": "Alice"}

# api/v2/users.py
"""API v2 — User endpoints.

.. versionadded:: 2.0
    Added pagination support and cursor-based navigation.
"""

def get_user(user_id: int, include: str = "profile") -> dict:
    """Get a user by ID with optional includes.

    Args:
        user_id: The user's ID.
        include: Comma-separated list of related resources.

    Returns:
        A dictionary with user data.

    .. versionadded:: 2.0
        Added the ``include`` parameter.
    """
    return {"id": user_id, "name": "Alice", "profile": {}}
```

**Expected Output**: No output (illustrative versioning).

**Why**: Sphinx directives like `.. deprecated::` and `.. versionadded::` document version-specific changes in the API reference.

#### Example 3: Migration Guide

```python
"""Migration Guide: v1 to v2.

This guide helps users migrate from API v1 to v2.

Breaking Changes
----------------

1. **Authentication**: v1 used API keys; v2 uses OAuth 2.0 tokens.

   v1: ``headers={"X-API-Key": "your-key"}``
   v2: ``headers={"Authorization": "Bearer your-token"}``

2. **Pagination**: v1 used offset-based pagination; v2 uses cursor-based.

   v1: ``GET /users?offset=0&limit=10``
   v2: ``GET /users?cursor=abc123&limit=10``

Deprecation Timeline
--------------------

- v1 deprecated: 2026-01-01
- v1 sunset: 2026-12-31
- v2 available: 2026-01-01

New Features in v2
------------------

- Cursor-based pagination
- Field selection via ``?fields=``
- Batch operations
"""
```

**Expected Output**: No output (illustrative migration guide).

**Why**: Migration guides help users upgrade between API versions by documenting breaking changes and new features.

### Real-World Cases

- **Library documentation**: Sphinx-generated reference for PyPI packages.
- **REST APIs**: OpenAPI/Swagger documentation for web services.
- **SDK documentation**: Generated references for client libraries.
- **Internal APIs**: Documentation for microservices and internal tools.

### References

- Sphinx autodoc — Including documentation from docstrings - https://www.sphinx-doc.org/en/master/usage/extensions/autodoc.html
- sphinx.ext.apidoc — Generate API documentation - https://www.sphinx-doc.org/en/master/usage/extensions/apidoc.html
- API Versioning Best Practices - https://github.com/Compile-N-Run/docs-framework
- OpenAPI Specification - https://spec.openapis.org/oas/latest.html

---

## 6. Documentation Generation

### Definitions

**Core Definition**: Documentation generation is the automated process of extracting docstrings and source code structure to produce external reference documentation in formats such as HTML, PDF, and ePub.

**Technical Definition**: Documentation generators parse Python source code, extract docstrings and type annotations, and render them into structured documentation using templates and themes. The two principal tools in the Python ecosystem are Sphinx (with `autodoc`, `napoleon`, and `apidoc` extensions) and MkDocs (with the `mkdocstrings` plugin). Sphinx uses reStructuredText as its source format and produces HTML, LaTeX, ePub, and PDF. MkDocs uses Markdown and produces HTML.

**Beginner-Friendly Explanation**: Documentation generators take the docstrings you've written in your code and turn them into a polished website. You don't have to copy-paste anything — the tool reads your code and generates the documentation automatically.

### Purposes

- To automate the creation of API reference documentation from source code.
- To keep documentation synchronized with code changes.
- To produce professional, searchable documentation websites.
- To support multiple output formats (HTML, PDF, ePub).
- To integrate documentation builds into CI/CD pipelines.

### Syntax Rules and Structure

#### Sphinx Configuration (conf.py)

```python
# conf.py
import os
import sys
sys.path.insert(0, os.path.abspath('..'))

project = 'My Project'
extensions = [
    'sphinx.ext.autodoc',
    'sphinx.ext.napoleon',
    'sphinx.ext.viewcode',
]
html_theme = 'sphinx_rtd_theme'
```

#### MkDocs Configuration (mkdocs.yml)

```yaml
# mkdocs.yml
site_name: My Project
theme:
  name: material

plugins:
  - search
  - mkdocstrings:
      handlers:
        python:
          options:
            show_source: true
            docstring_style: google
            members_order: source
```

#### Syntax Rules

1. **Sphinx**: Uses reStructuredText source files; `autodoc` pulls in docstrings; `napoleon` converts Google/NumPy styles to reST; `apidoc` generates stub files.
2. **MkDocs**: Uses Markdown source files; `mkdocstrings` injects API documentation from docstrings.
3. **Configuration**: Sphinx uses `conf.py`; MkDocs uses `mkdocs.yml`.
4. **Extensions/plugins**: Sphinx extensions and MkDocs plugins extend functionality.
5. **Build command**: `sphinx-build` for Sphinx; `mkdocs build` for MkDocs.
6. **Theme**: Both support custom themes (e.g., `sphinx_rtd_theme`, `material`).

#### Sphinx vs. MkDocs Comparison

| Aspect | Sphinx | MkDocs |
|--------|--------|--------|
| Source format | reStructuredText | Markdown |
| Configuration | `conf.py` | `mkdocs.yml` |
| API extraction | `autodoc` extension | `mkdocstrings` plugin |
| Google/NumPy support | `napoleon` extension | `mkdocstrings` (native) |
| Output formats | HTML, LaTeX, ePub, PDF | HTML |
| Learning curve | Steeper | Gentler |
| Ecosystem | Mature, extensive | Growing, modern |

#### Constraints and Limitations

- Sphinx requires reStructuredText knowledge (or Napoleon for Google/NumPy styles).
- MkDocs requires Markdown knowledge.
- Both require configuration and maintenance.
- Docstrings must be well-formed for good output.

### Multiple Annotated Complete Step by Step Code Examples and Their Expected Outputs

#### Example 1: Sphinx Documentation Setup

```python
# docs/conf.py
"""Sphinx configuration file."""

import os
import sys

# Add the project root to the path
sys.path.insert(0, os.path.abspath('..'))

project = 'My Library'
copyright = '2026, Author'
author = 'Author'
release = '1.0'

extensions = [
    'sphinx.ext.autodoc',
    'sphinx.ext.napoleon',
    'sphinx.ext.viewcode',
    'sphinx.ext.autosummary',
]

autosummary_generate = True
napoleon_google_docstring = True
napoleon_numpy_docstring = False

html_theme = 'sphinx_rtd_theme'
```

```rst
# docs/api.rst
API Reference
=============

.. automodule:: mylibrary
   :members:
   :undoc-members:
   :show-inheritance:
```

```bash
# Build command
sphinx-build -b html docs/ docs/_build/
```

**Expected Output**:
```
Running Sphinx v7.2.6
...
build succeeded, 0 warnings.
The HTML pages are in docs/_build.
```

**Why**: Sphinx reads `conf.py`, processes `api.rst`, imports the module, extracts docstrings, and generates HTML documentation.

#### Example 2: MkDocs with MkDocStrings

```yaml
# mkdocs.yml
site_name: My Library
theme:
  name: material
  features:
    - navigation.tabs
    - search.suggest

plugins:
  - search
  - mkdocstrings:
      handlers:
        python:
          options:
            show_source: true
            show_root_heading: true
            docstring_style: google
            members_order: source
            group_by_category: true
            filters:
              - "!^_"
```

```markdown
# docs/api.md
# API Reference

## Core Module

::: mylibrary.core
    options:
      show_source: true
      members_order: source
```

```bash
# Build command
mkdocs build
```

**Expected Output**:
```
INFO    -  Building documentation...
INFO    -  Cleaning site directory
INFO    -  Documentation built in 1.23 seconds
```

**Why**: MkDocs reads `mkdocs.yml`, processes Markdown files, and MkDocStrings injects API documentation from docstrings.

#### Example 3: CI/CD Integration

```yaml
# .github/workflows/docs.yml
name: Documentation
on:
  push:
    branches: [main]

jobs:
  docs:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v5
        with:
          python-version: "3.11"
      - run: pip install sphinx sphinx-rtd-theme
      - run: sphinx-build -b html docs/ docs/_build/
      - uses: peaceiris/actions-gh-pages@v3
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}
          publish_dir: docs/_build
```

**Expected CI Output**:
```
Running Sphinx v7.2.6
build succeeded, 0 warnings.
The HTML pages are in docs/_build.
Pushed to gh-pages branch.
```

**Why**: The CI workflow builds documentation on every push to `main` and deploys it to GitHub Pages.

### Real-World Cases

- **Open-source libraries**: Sphinx is used by Django, NumPy, SciPy, and Python itself.
- **Modern Python projects**: MkDocs with Material theme is popular for newer projects.
- **Internal documentation**: Both tools support private documentation servers.
- **API references**: Sphinx autodoc and MkDocStrings generate reference pages.

### References

- Sphinx Documentation - https://www.sphinx-doc.org/
- sphinx.ext.autodoc - https://www.sphinx-doc.org/en/master/usage/extensions/autodoc.html
- sphinx.ext.napoleon - https://www.sphinx-doc.org/en/master/usage/extensions/napoleon.html
- sphinx.ext.apidoc - https://www.sphinx-doc.org/en/master/usage/extensions/apidoc.html
- MkDocs Documentation - https://www.mkdocs.org/
- MkDocStrings Documentation - https://mkdocstrings.github.io/
- mkdocstrings Python handler - https://mkdocstrings.github.io/python/

---

## References

- PEP 257 – Docstring Conventions - https://peps.python.org/pep-0257/
- PEP 256 – Docstring Processing System Framework - https://peps.python.org/pep-0256/
- PEP 258 – Docutils Design Specification - https://peps.python.org/pep-0258/
- PEP 484 – Type Hints - https://peps.python.org/pep-0484/
- sphinx.ext.napoleon — Support for NumPy and Google style docstrings - https://www.sphinx-doc.org/en/master/usage/extensions/napoleon.html
- Sphinx autodoc — Including documentation from docstrings - https://www.sphinx-doc.org/en/master/usage/extensions/autodoc.html
- sphinx.ext.apidoc — Generate API documentation - https://www.sphinx-doc.org/en/master/usage/extensions/apidoc.html
- MkDocs Documentation - https://www.mkdocs.org/
- MkDocStrings Documentation - https://mkdocstrings.github.io/
- mkdocstrings Python handler - https://mkdocstrings.github.io/python/
- Google Python Style Guide (Docstrings) - https://google.github.io/styleguide/pyguide.html#38-comments-and-docstrings
- NumPy Docstring Standard - https://numpydoc.readthedocs.io/en/latest/format.html
- Example Google Style Python Docstrings - https://www.sphinx-doc.org/en/master/usage/extensions/example_google.html
- Example NumPy Style Python Docstrings - https://www.sphinx-doc.org/en/master/usage/extensions/example_numpy.html
- Real Python: Documenting Python Code - https://realpython.com/documenting-python-code/