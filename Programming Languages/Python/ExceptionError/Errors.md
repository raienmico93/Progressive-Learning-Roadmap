# Python Errors: A Comprehensive Programming Cheat Sheet

## Topic Overview

### Definitions

**Core Definition**
Errors in Python are problems that prevent a program from running correctly or producing the expected result. They fall into three fundamental categories: syntax errors, runtime errors (exceptions), and logical errors.

**Technical Definition**
Python distinguishes between errors detected at compile time (syntax errors) and errors that occur during execution (exceptions/runtime errors). A syntax error prevents the parser from generating bytecode and halts execution before the program starts. A runtime error raises an exception object at the point of failure, which propagates up the call stack unless caught by an `except` clause. Logical errors produce incorrect results without raising any exception, requiring manual debugging to identify.

**Beginner-Friendly Explanation**
Think of writing a recipe. A **syntax error** is like writing in a language the chef doesn't understand—the recipe is rejected before cooking starts. A **runtime error** is like following the recipe but running out of an ingredient halfway through—the cooking stops with a specific complaint. A **logical error** is like following the recipe perfectly but using salt instead of sugar—the cake is baked, but it tastes wrong.

### Key Characteristics

| Characteristic | Description |
|---|---|
| **Syntax Errors** | Detected at compile time; prevent the program from running at all |
| **Runtime Errors** | Detected during execution; raise exception objects that can be caught |
| **Logical Errors** | No exception raised; program runs but produces incorrect results |
| **Exception Hierarchy** | All built-in exceptions inherit from `BaseException`, with `Exception` as the base for most user-facing errors |
| **Traceback** | Python prints a traceback for unhandled exceptions showing the call stack and error location |

### Prerequisites

- Basic Python syntax and program structure
- Understanding of the `try`/`except` statement
- Familiarity with reading error messages and tracebacks
- Basic understanding of data types and operations

### Related Programming Areas

- **Exception Handling**: Catching and recovering from errors using `try`/`except`/`finally`
- **Debugging**: Identifying and fixing errors using print statements, assertions, logging, and debuggers
- **Defensive Programming**: Anticipating and preventing errors before they occur
- **Testing**: Unit tests that verify code behaves correctly under various conditions

### Core Concepts / Features

1. **Syntax Errors** (compile-time failures)
2. **Runtime Errors** (exceptions raised during execution)
3. **Logical Errors** (silent incorrect behavior)
4. **Type Errors** (`TypeError`)
5. **Value Errors** (`ValueError`)
6. **Name Errors** (`NameError`)
7. **Index Errors** (`IndexError`)
8. **Key Errors** (`KeyError`)
9. **Attribute Errors** (`AttributeError`)
10. **Import Errors** (`ImportError`, `ModuleNotFoundError`)

---

## Core Concept 1: Syntax Errors

### Definitions

**Core Definition**
A syntax error is a violation of Python's grammar rules that prevents the code from being compiled into bytecode.

**Technical Definition**
Syntax errors are detected by the parser during compilation. The parser reads the source code, attempts to build an abstract syntax tree (AST), and fails if the tokens do not conform to Python's grammar. The error is reported with a line number and an arrow pointing to the problematic token. The program never begins execution because bytecode cannot be generated.

**Beginner-Friendly Explanation**
A syntax error is like a grammatical mistake in a sentence. If you write "I is happy," a reader knows what you mean but the grammar is wrong. Python is stricter—it refuses to run code with grammatical mistakes.

### Purposes (of Understanding Syntax Errors)

- **To quickly identify and fix typos** in code
- **To understand Python's grammar rules** and avoid common mistakes
- **To distinguish compile-time failures** from runtime failures

### Syntax Rules and Structure

Syntax errors are not "used"—they occur when code violates Python's grammar. Common causes include:

| Cause | Example |
|---|---|
| Missing colon | `if x > 0` instead of `if x > 0:` |
| Unmatched parentheses | `print("hello"` |
| Invalid assignment target | `1 = x` |
| Wrong indentation | Inconsistent spaces/tabs |
| Missing quote | `print("hello)` |

### Multiple Annotated Complete Code Examples and Expected Outputs

**Example 1: Missing Colon**

```python
# Syntax error: missing colon after if condition
if x > 0
    print("positive")
```

**Expected Output:**
```
  File "<stdin>", line 1
    if x > 0
            ^
SyntaxError: expected ':'
```

**Breakdown:** The parser expects a colon to end the `if` statement. Without it, compilation fails before any code runs.

**Example 2: Invalid Assignment Target**

```python
# Syntax error: cannot assign to a literal
1 = x
```

**Expected Output:**
```
  File "<stdin>", line 1
    1 = x
    ^
SyntaxError: cannot assign to literal
```

**Breakdown:** Python's grammar requires the left side of `=` to be an assignable target (variable, attribute, subscript). A number literal is not assignable.

**Example 3: Unclosed String**

```python
# Syntax error: missing closing quote
print("Hello, World!)
```

**Expected Output:**
```
  File "<stdin>", line 1
    print("Hello, World!)
          ^
SyntaxError: unterminated string literal (detected at line 1)
```

**Breakdown:** The string literal is not terminated, so the parser cannot determine where it ends.

### Real-World Cases with Explanation

**Case 1: Typo in Function Definition**

```python
# BAD: missing colon
def greet(name)
    print(f"Hello, {name}!")

# GOOD: colon present
def greet(name):
    print(f"Hello, {name}!")
```

**Why it matters:** Syntax errors are usually the easiest to fix because Python tells you exactly where the problem is. The arrow (`^`) points to the offending token.

---

## Core Concept 2: Runtime Errors

### Definitions

**Core Definition**
A runtime error (exception) is an error that occurs during program execution, after the code has been successfully compiled.

**Technical Definition**
Runtime errors raise exception objects at the point of failure. If not caught by an `except` clause, the exception propagates up the call stack, and Python prints a traceback showing the sequence of function calls that led to the error. Exceptions are not necessarily fatal—they can be caught and handled programmatically.

**Beginner-Friendly Explanation**
A runtime error is like a recipe that looks correct but fails while cooking—maybe an ingredient is missing, or the oven temperature is wrong. The recipe was fine on paper, but something went wrong during execution.

### Purposes (of Runtime Errors)

- **To signal that an operation cannot be completed** as requested
- **To allow programs to recover gracefully** from unexpected conditions
- **To provide diagnostic information** about what went wrong and where

### Syntax Rules and Structure

Runtime errors are raised using the `raise` statement and caught using `try`/`except`:

```python
try:
    result = 10 / 0
except ZeroDivisionError as e:
    print(f"Error: {e}")
```

### Multiple Annotated Complete Code Examples and Expected Outputs

**Example 1: Uncaught Runtime Error**

```python
def divide(a, b):
    return a / b

print(divide(10, 0))
```

**Expected Output:**
```
Traceback (most recent call last):
  File "<stdin>", line 4, in <module>
    print(divide(10, 0))
  File "<stdin>", line 2, in divide
    return a / b
ZeroDivisionError: division by zero
```

**Breakdown:** The traceback shows the call stack. The last line identifies the exception type (`ZeroDivisionError`) and message. Reading bottom-to-top reveals: what error, where it occurred, and how the program got there.

**Example 2: Caught Runtime Error**

```python
try:
    result = 10 / 0
except ZeroDivisionError as e:
    print(f"Caught: {e}")
    result = None

print(f"Result: {result}")
```

**Expected Output:**
```
Caught: division by zero
Result: None
```

**Breakdown:** The `except` clause catches the exception, preventing the program from crashing. Execution continues after the `try`/`except` block.

### Real-World Cases with Explanation

**Case 1: Reading a File That Doesn't Exist**

```python
try:
    with open("config.json") as f:
        config = json.load(f)
except FileNotFoundError:
    config = {"debug": False}  # Default configuration
```

**Why it matters:** Programs often need to handle missing files gracefully rather than crashing.

---

## Core Concept 3: Logical Errors

### Definitions

**Core Definition**
A logical error is a mistake in the program's logic that causes incorrect behavior without raising any exception.

**Technical Definition**
Logical errors occur when the code runs successfully (no syntax or runtime errors) but produces wrong results. The program executes exactly as written, but the written logic does not match the intended behavior. These are the most difficult errors to detect because Python provides no error message.

**Beginner-Friendly Explanation**
A logical error is like following a recipe perfectly but using salt instead of sugar. Everything worked—the cake baked—but the result is wrong. Python won't tell you because Python doesn't know what you intended.

### Purposes (of Understanding Logical Errors)

- **To recognize that not all bugs produce error messages**
- **To develop testing and debugging strategies** for silent failures
- **To use assertions and unit tests** to catch logic errors early

### Syntax Rules and Structure

Logical errors have no specific syntax—they are semantic mistakes. Common causes include:

| Cause | Example |
|---|---|
| Off-by-one errors | `for i in range(len(items) - 1)` skips last item |
| Wrong operator | Using `=` instead of `==` (Python catches this as syntax) |
| Incorrect order of operations | `a + b * c` when `(a + b) * c` was intended |
| Wrong variable | Using `x` when `y` was meant |

### Multiple Annotated Complete Code Examples and Expected Outputs

**Example 1: Off-by-One Error**

```python
def sum_list(items):
    total = 0
    for i in range(len(items) - 1):  # BUG: skips last element
        total += items[i]
    return total

numbers = [1, 2, 3, 4, 5]
print(sum_list(numbers))
```

**Expected Output:**
```
10
```

**Breakdown:** The correct sum is 15, but the loop stops at index 3 (value 4), missing the final element (5). No error is raised—the code runs silently but incorrectly.

**Example 2: Wrong Variable**

```python
def calculate_area(width, height):
    area = width * width  # BUG: should be width * height
    return area

print(calculate_area(5, 10))
```

**Expected Output:**
```
25
```

**Breakdown:** The correct area is 50, but the function uses `width` twice. The code runs without error but produces the wrong result.

### Real-World Cases with Explanation

**Case 1: Incorrect Discount Calculation**

```python
def apply_discount(price, discount_percent):
    # BUG: adds discount instead of subtracting
    return price + (price * discount_percent / 100)

print(apply_discount(100, 20))  # Should be 80, returns 120
```

**Why it matters:** Logical errors in financial calculations can have serious consequences. Unit tests with known expected values catch these errors.

---

## Core Concept 4: Type Errors

### Definitions

**Core Definition**
A `TypeError` is raised when an operation or function is applied to an object of an inappropriate type.

**Technical Definition**
`TypeError` occurs when an operation receives an argument of the correct type category but the wrong specific type for that operation—for example, adding a string to an integer, or calling a non-callable object. It is a subclass of `Exception`.

**Beginner-Friendly Explanation**
A type error is like trying to fit a square peg in a round hole. The peg is a valid object, but it's the wrong kind for this operation.

### Purposes (of Understanding Type Errors)

- **To identify type mismatches** in operations
- **To understand Python's dynamic typing** and when types conflict
- **To use type checking and conversion** to prevent errors

### Syntax Rules and Structure

```python
# Raises TypeError
"hello" + 5          # Cannot concatenate str and int
len(42)              # int has no len()
None()               # None is not callable
```

### Multiple Annotated Complete Code Examples and Expected Outputs

**Example 1: Unsupported Operand Types**

```python
result = "Age: " + 30
```

**Expected Output:**
```
TypeError: can only concatenate str (not "int") to str
```

**Breakdown:** The `+` operator works differently for strings (concatenation) and numbers (addition). Mixing types without conversion raises `TypeError`.

**Example 2: Calling a Non-Callable**

```python
x = 42
x()
```

**Expected Output:**
```
TypeError: 'int' object is not callable
```

**Breakdown:** Only functions, methods, and objects with `__call__` can be called with parentheses. An integer cannot.

### Real-World Cases with Explanation

**Case 1: Function Argument Type Mismatch**

```python
def greet(name):
    return "Hello, " + name

greet(42)  # TypeError: can only concatenate str (not "int") to str
```

**Why it matters:** Functions expecting strings but receiving integers are a common source of `TypeError`. Type hints and validation help prevent this.

---

## Core Concept 5: Value Errors

### Definitions

**Core Definition**
A `ValueError` is raised when an operation or function receives an argument of the correct type but with an inappropriate value.

**Technical Definition**
`ValueError` signals that the type is correct but the specific value cannot be used—for example, `int("hello")` receives a string (correct type for conversion) but the string does not represent a valid integer. It is a subclass of `Exception`.

**Beginner-Friendly Explanation**
A value error is like trying to convert "banana" to a number. The input is a string (the right type for conversion), but "banana" isn't a number, so the conversion fails.

### Purposes (of Understanding Value Errors)

- **To distinguish between type problems and value problems**
- **To handle invalid data** gracefully (e.g., user input)
- **To validate values** before using them

### Syntax Rules and Structure

```python
# Raises ValueError
int("hello")         # String is correct type, but not a valid number
float("abc")         # Same
chr(-1)              # Invalid Unicode code point
```

### Multiple Annotated Complete Code Examples and Expected Outputs

**Example 1: Invalid Conversion**

```python
number = int("hello")
```

**Expected Output:**
```
ValueError: invalid literal for int() with base 10: 'hello'
```

**Breakdown:** `int()` expects a string that represents a number. The string `"hello"` is the correct type (string) but an invalid value.

**Example 2: Unpacking Mismatch**

```python
a, b = [1, 2, 3]
```

**Expected Output:**
```
ValueError: too many values to unpack (expected 2)
```

**Breakdown:** The right side has three elements, but only two targets are provided. The number of values doesn't match.

### Real-World Cases with Explanation

**Case 1: Validating User Input**

```python
def get_age():
    age_str = input("Enter age: ")
    try:
        return int(age_str)
    except ValueError:
        print("Invalid age. Please enter a number.")
        return None
```

**Why it matters:** User input is never guaranteed to be valid. Catching `ValueError` prevents crashes when users enter non-numeric data.

---

## Core Concept 6: Name Errors

### Definitions

**Core Definition**
A `NameError` is raised when a name (variable, function, or module) is referenced but not defined in the current scope.

**Technical Definition**
`NameError` occurs during runtime when Python's name resolution (LEGB rule) fails to find the requested name in local, enclosing, global, or built-in scopes. It is a subclass of `Exception`.

**Beginner-Friendly Explanation**
A name error is like calling someone by a name that doesn't exist. You said "Hey, Bob!" but there's no Bob here. Python looked everywhere it knows but couldn't find the name.

### Purposes (of Understanding Name Errors)

- **To identify typos** in variable or function names
- **To understand scope** and why names aren't accessible in certain contexts
- **To debug `UnboundLocalError`** (a subclass of `NameError`)

### Syntax Rules and Structure

```python
# Raises NameError
print(undefined_variable)
undefined_function()
```

### Multiple Annotated Complete Code Examples and Expected Outputs

**Example 1: Undefined Variable**

```python
print(user_name)
```

**Expected Output:**
```
NameError: name 'user_name' is not defined
```

**Breakdown:** The variable `user_name` was never assigned, so Python cannot find it in any scope.

**Example 2: Typo in Variable Name**

```python
message = "Hello"
print(mesage)
```

**Expected Output:**
```
NameError: name 'mesage' is not defined
```

**Breakdown:** The variable is named `message` (with two s's), but the print statement references `mesage` (one s).

### Real-World Cases with Explanation

**Case 1: Scope Confusion**

```python
def process():
    result = compute()
    return result

def compute():
    return x + 1  # NameError if x is not defined

process()
```

**Why it matters:** Variables defined in one function are not accessible in another. `x` must be defined or passed as a parameter.

---

## Core Concept 7: Index Errors

### Definitions

**Core Definition**
An `IndexError` is raised when a sequence (list, tuple, string) is indexed with a number that is out of range.

**Technical Definition**
`IndexError` occurs when a subscript is outside the valid range for the sequence. For a sequence of length `n`, valid indices are `0` to `n-1` (or `-1` to `-n` for negative indexing). It is a subclass of `LookupError`.

**Beginner-Friendly Explanation**
An index error is like reaching for shelf 5 in a bookcase that only has 3 shelves. You're asking for something that doesn't exist at that position.

### Purposes (of Understanding Index Errors)

- **To identify off-by-one errors** in loops and indexing
- **To safely access sequence elements** with bounds checking
- **To understand negative indexing** and its limits

### Syntax Rules and Structure

```python
# Raises IndexError
items = [1, 2, 3]
items[5]           # Index 5 doesn't exist
items[-4]          # Negative index beyond length
```

### Multiple Annotated Complete Code Examples and Expected Outputs

**Example 1: Index Out of Range**

```python
numbers = [10, 20, 30]
print(numbers[3])
```

**Expected Output:**
```
IndexError: list index out of range
```

**Breakdown:** The list has indices 0, 1, 2. Index 3 is beyond the end.

**Example 2: Off-by-One in Loop**

```python
items = [1, 2, 3, 4, 5]
for i in range(len(items) + 1):  # BUG: +1 goes one too far
    print(items[i])
```

**Expected Output:**
```
1
2
3
4
5
IndexError: list index out of range
```

**Breakdown:** The loop runs from 0 to 5, but valid indices are 0 to 4. The final iteration raises `IndexError`.

### Real-World Cases with Explanation

**Case 1: Safe Access with Bounds Check**

```python
def get_item(items, index):
    if 0 <= index < len(items):
        return items[index]
    return None
```

**Why it matters:** Bounds checking prevents `IndexError` when accessing sequences with user-provided indices.

---

## Core Concept 8: Key Errors

### Definitions

**Core Definition**
A `KeyError` is raised when a dictionary is accessed with a key that does not exist.

**Technical Definition**
`KeyError` occurs when using subscript notation (`d[key]`) on a dictionary where `key` is not present. The exception's argument is the missing key. It is a subclass of `LookupError`.

**Beginner-Friendly Explanation**
A key error is like looking up a word in a dictionary that isn't there. The dictionary exists, but that specific word doesn't.

### Purposes (of Understanding Key Errors)

- **To identify missing keys** in dictionary access
- **To use safe access methods** (`.get()`, `in` checks)
- **To handle configuration or data** that may lack expected keys

### Syntax Rules and Structure

```python
# Raises KeyError
person = {"name": "Alice"}
person["age"]       # Key 'age' doesn't exist
```

### Multiple Annotated Complete Code Examples and Expected Outputs

**Example 1: Missing Key**

```python
config = {"host": "localhost", "port": 5432}
print(config["database"])
```

**Expected Output:**
```
KeyError: 'database'
```

**Breakdown:** The dictionary has keys `"host"` and `"port"`, but not `"database"`.

**Example 2: Safe Access with `.get()`**

```python
config = {"host": "localhost", "port": 5432}
database = config.get("database", "default_db")
print(database)
```

**Expected Output:**
```
default_db
```

**Breakdown:** `.get()` returns the default value instead of raising `KeyError` when the key is missing.

### Real-World Cases with Explanation

**Case 1: API Response Handling**

```python
def get_user_name(response):
    # Safe: returns None if key is missing
    return response.get("user", {}).get("name")
```

**Why it matters:** API responses may omit fields. Using `.get()` with chaining prevents `KeyError` while safely navigating nested dictionaries.

---

## Core Concept 9: Attribute Errors

### Definitions

**Core Definition**
An `AttributeError` is raised when an attribute reference or assignment fails—the object does not have that attribute.

**Technical Definition**
`AttributeError` occurs when using dot notation (`obj.attribute`) to access an attribute that does not exist on the object. This includes typos in method names, accessing instance attributes that haven't been set, or using methods not supported by the object's type. It is a subclass of `Exception`.

**Beginner-Friendly Explanation**
An attribute error is like trying to use a feature on your phone that doesn't exist. You pressed the "teleport" button, but your phone has no such button.

### Purposes (of Understanding Attribute Errors)

- **To identify typos** in method or attribute names
- **To understand what attributes an object supports**
- **To use `hasattr()` and `getattr()`** for safe attribute access

### Syntax Rules and Structure

```python
# Raises AttributeError
"hello".append("!")     # str has no append method
[1, 2].upper()          # list has no upper method
```

### Multiple Annotated Complete Code Examples and Expected Outputs

**Example 1: Typo in Method Name**

```python
items = [1, 2, 3]
items.appedn(4)
```

**Expected Output:**
```
AttributeError: 'list' object has no attribute 'appedn'
```

**Breakdown:** The method is `append`, not `appedn`. The typo causes `AttributeError`.

**Example 2: Wrong Type for Method**

```python
name = "Alice"
name.push("Bob")
```

**Expected Output:**
```
AttributeError: 'str' object has no attribute 'push'
```

**Breakdown:** Strings don't have a `push` method. `push` is associated with lists (and even then, it's `append`).

### Real-World Cases with Explanation

**Case 1: Optional Methods**

```python
def process(data):
    if hasattr(data, "read"):
        return data.read()
    return str(data)
```

**Why it matters:** `hasattr()` checks if an attribute exists before accessing it, preventing `AttributeError` when working with objects of unknown type.

---

## Core Concept 10: Import Errors

### Definitions

**Core Definition**
An `ImportError` is raised when an import statement fails to find a module or a name within a module. `ModuleNotFoundError` is a subclass for the specific case of a missing module.

**Technical Definition**
`ImportError` occurs when `import module` or `from module import name` fails. Causes include: module not installed, module not on `sys.path`, circular imports, or name not defined in the module. `ModuleNotFoundError` (Python 3.6+) specifically indicates the module itself was not found. Both are subclasses of `Exception`.

**Beginner-Friendly Explanation**
An import error is like asking for a tool from a toolbox that doesn't have it. Either the toolbox is missing entirely (`ModuleNotFoundError`), or the toolbox exists but doesn't contain the specific tool you asked for (`ImportError`).

### Purposes (of Understanding Import Errors)

- **To diagnose missing dependencies** (packages not installed)
- **To fix typos** in module or name imports
- **To resolve circular import** issues
- **To handle optional dependencies** gracefully

### Syntax Rules and Structure

```python
# Raises ModuleNotFoundError
import nonexistent_module

# Raises ImportError
from math import nonexistent_function
```

### Multiple Annotated Complete Code Examples and Expected Outputs

**Example 1: Module Not Found**

```python
import banana
```

**Expected Output:**
```
ModuleNotFoundError: No module named 'banana'
```

**Breakdown:** The module `banana` is not installed or not on the Python path.

**Example 2: Name Not Found in Module**

```python
from math import sqrt, nonexistent
```

**Expected Output:**
```
ImportError: cannot import name 'nonexistent' from 'math'
```

**Breakdown:** The module `math` exists, but it does not define a name called `nonexistent`.

### Real-World Cases with Explanation

**Case 1: Optional Dependency**

```python
try:
    import ujson as json
except ImportError:
    import json  # Fallback to standard library
```

**Why it matters:** Libraries often support optional accelerated backends. Catching `ImportError` allows graceful fallback.

---

## References

- Python Software Foundation. *8. Errors and Exceptions*. https://docs.python.org/3.10/tutorial/errors.html 
- Python Software Foundation. *Built-in Exceptions*. https://docs.python.org/3.11/library/exceptions.html 
- Python.org Tutor Mailing List. *[Tutor] python errors*. https://mail.python.org/archives/list/tutor@python.org/thread/Q3EGBN36QXS6LPMG2CLOUH3L2VNIATP6/ 
- Stanford University. *Introduction to Exceptions*. http://stanford.edu/~kach/exceptions.pdf 
- Cornell University. *CS1110: When things go wrong (in Python)*. https://www.cs.cornell.edu/courses/cs1110/2013sp/lectures/04-18-13/presentation-24.pdf 
- Python Numerical Methods (UC Berkeley). *Error Types*. https://pythonnumericalmethods.studentorg.berkeley.edu/notebooks/chapter10.01-Error-Types.html 
- Python.org Python-list. *variable declaration*. https://mail.python.org/pipermail/python-list/2005-February/325631.html 
- PyLadiesCon 2025. *So It Broke… Now What? Debugging Python 101*. https://pretalx.com/pyladiescon-2025/talk/YL3LZC/ 
- Python Documentation (Korean). *6. 내장 예외*. https://docs.python.org/ko/2/library/exceptions.html 
- igadc.cn. *Python 异常处理*. https://cpjrc.igadc.cn/lesson/jt8ed2/ 