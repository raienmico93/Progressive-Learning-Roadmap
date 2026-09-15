# Variables & Assignment Mechanics: A Comprehensive Programming Cheat Sheet

## Topic Overview

### Definitions

**Core Definition**
Variables and assignment mechanics describe how programming languages bind names (identifiers) to values or objects in memory, enabling programs to store, reference, and manipulate data throughout execution.

**Technical Definition**
In Python, a variable is not a storage container but a **name binding**—a mapping between an identifier and an object residing in memory. The assignment statement `x = value` creates or updates an entry in a namespace dictionary, binding the name `x` to the object produced by evaluating `value`. Python's assignment model is **reference-based**: multiple names may refer to the same object, and assignment operations copy references, not objects.

**Beginner-Friendly Explanation**
Think of a variable not as a box that holds data, but as a **label** or **sticky note** that you attach to a value. When you write `x = 5`, you are not putting 5 into a container named `x`; you are sticking the label `x` onto the object `5`. If you later write `y = x`, you stick another label (`y`) onto that same object. This distinction matters most when objects can change (like lists), because changing the object through one label affects all other labels pointing to it.

### Key Characteristics

| Characteristic | Description |
|---|---|
| **Reference Semantics** | Variables hold references (pointers) to objects, not the objects themselves |
| **Dynamic Typing** | A name can be rebound to an object of any type at any time; the name itself has no inherent type |
| **Garbage Collection** | When no names reference an object, Python automatically reclaims its memory |
| **Namespace Binding** | Assignment creates entries in namespace dictionaries (local, global, or built-in) |
| **Left-to-Right Assignment Order** | In chained assignments, the rightmost expression is evaluated first, then names are assigned left to right |

### Prerequisites

- **Basic understanding of programming concepts**: sequential execution, expressions, and statements
- **Familiarity with Python syntax**: how to run a script or use the interactive interpreter
- **Concept of memory and objects**: a general idea that data lives somewhere in the computer's memory
- **Python installation** (any version 3.x for modern examples; version-specific notes provided where relevant)

### Related Programming Areas

- **Scope and Namespaces**: How variable visibility is determined by local, enclosing, global, and built-in scopes
- **Object-Oriented Programming**: Understanding that everything in Python is an object, with types and methods
- **Memory Management**: Reference counting and garbage collection of unreferenced objects
- **Mutability and Immutability**: How shared references behave differently for mutable vs. immutable objects
- **Type Systems**: Dynamic typing, type hints, and their interaction with assignment

### Core Concepts / Features

1. **Variable Assignment** (How variables act as labels to objects, not boxes)
2. **Naming Variables** (Rules and PEP 8 conventions)
3. **Multiple Assignment** (Unpacking iterables into multiple variables)
4. **Chained Assignment** (Assigning the same object reference to multiple labels)

---

## Core Concept 1: Variable Assignment

### Definitions

**Core Definition**
Variable assignment is the act of binding an identifier (name) to an object in memory using the `=` operator.

**Technical Definition**
The assignment statement `target = expression` evaluates the right-hand `expression` to produce an object reference, then binds the left-hand `target` (a name, attribute, or subscription) to that reference in the current namespace. The target does not "contain" the object; it is a key in a dictionary (the namespace) whose value is a pointer to the object.

**Beginner-Friendly Explanation**
When you write `x = "hello"`, Python creates the string object `"hello"` in memory and then makes the name `x` point to it. If you later write `x = "world"`, Python creates a new string and makes `x` point to that instead. The old string is not "overwritten"; if no other name points to it, it gets cleaned up automatically.

### Purposes

- **To store data for later use** within a program
- **To give meaningful names to values**, improving code readability and maintainability
- **To allow data to be manipulated** through named references rather than raw memory addresses
- **To support dynamic program state**, where values change over time as the program executes
- **To enable abstraction**, letting programmers work with concepts (like `user_age`) rather than concrete memory locations

### Syntax Rules and Structure

**Complete General Syntax**

```
target = expression
```

**Breakdown:**
- `target`: The name (or attribute/subscript) being bound. Must be a valid identifier unless using attribute access (`obj.attr`) or subscription (`container[key]`).
- `=`: The assignment operator. Note this is **not** an equality test (that is `==`).
- `expression`: Any valid Python expression that evaluates to an object (literals, function calls, arithmetic, etc.).

**Syntax Rules**

| Rule | Description |
|---|---|
| Target validity | Target must be assignable: a name, attribute (`obj.attr`), or subscript (`container[index]`) |
| Right-hand evaluation | The expression is always evaluated before any binding occurs |
| Single `=` | Assignment uses one equals sign; two (`==`) is comparison |
| No expression value | Assignment is a statement, not an expression—it cannot be used inside another expression |
| Type agnosticism | The target does not need a declared type; it can be rebound to any object type |

**Constraints and Limitations**

- **No constants**: Python has no built-in mechanism to prevent rebinding a name (though type checkers may flag it).
- **Assignment is not an expression**: You cannot write `if (x = 5):` as you can in C.
- **Names must be valid identifiers**: Cannot use keywords (`class`, `def`, `if`, etc.) or names starting with digits.
- **Local scope by default**: Inside functions, assigning to a name creates a local variable unless declared `global` or `nonlocal`.

### Multiple Annotated Complete Code Examples and Expected Outputs

**Example 1: Basic Assignment and Reference Behavior**

```python
# Step 1: Assign a string object to the name 'message'
message = "Hello, Python"
# At this point, 'message' is a label pointing to the string object "Hello, Python"

# Step 2: Print the value by referencing the name
print(message)
# Python looks up 'message' in the namespace, finds the reference,
# and follows it to the string object, which prints: Hello, Python

# Step 3: Reassign the name to a new object
message = "Goodbye"
# A new string object "Goodbye" is created; 'message' now points to it.
# The original "Hello, Python" object remains in memory only if
# another name still references it (it doesn't here, so it's garbage collected)

print(message)
# Output: Goodbye
```

**Expected Output:**
```
Hello, Python
Goodbye
```

**Breakdown:** The code demonstrates that reassignment does not modify the original object—it simply rebinds the name. The name `message` acts as a movable label.

**Example 2: Shared References with Immutable Objects**

```python
# Step 1: Bind 'a' to an integer object
a = 42

# Step 2: Bind 'b' to the SAME object (not a copy)
b = a
# Both 'a' and 'b' now reference the integer object 42

print(f"a = {a}, b = {b}")
# Output: a = 42, b = 42

# Step 3: Rebind 'a' to a new object
a = 100
# The name 'a' now points to a new integer 100.
# 'b' still points to 42 because integers are immutable—there was no way
# to "change" 42 in place, so 'a' had to be rebound.

print(f"a = {a}, b = {b}")
# Output: a = 100, b = 42
```

**Expected Output:**
```
a = 42, b = 42
a = 100, b = 42
```

**Breakdown:** Since integers are immutable, rebinding `a` does not affect `b`. Both initially referenced the same object, but the rebinding operation simply reassigned the label `a` to a different object.

**Example 3: Shared References with Mutable Objects**

```python
# Step 1: Create a list (mutable object) and bind it to 'list_a'
list_a = [10, 20, 30]

# Step 2: Bind 'list_b' to the same list object
list_b = list_a
# Both names reference the exact same list object in memory

# Step 3: Modify the list IN PLACE through 'list_b'
list_b[1] = 99
# The list object itself changes: its second element becomes 99.
# Since 'list_a' references the same object, it sees the change too.

print(f"list_a: {list_a}")
print(f"list_b: {list_b}")
# Both print the same updated list
```

**Expected Output:**
```
list_a: [10, 99, 30]
list_b: [10, 99, 30]
```

**Breakdown:** This is the critical distinction. Because lists are mutable, changing the object through one reference is visible through all references. If you wanted `list_b` to be independent, you would need `list_b = list_a.copy()` or `list_b = list(list_a)`.

### Real-World Cases with Explanation

**Case 1: Function Arguments as Shared References**

When you pass a mutable object (like a list) to a function, the function receives a reference to the same object. Modifications inside the function affect the caller's object:

```python
def add_item(items, new_item):
    items.append(new_item)  # Modifies the caller's list

my_list = [1, 2, 3]
add_item(my_list, 4)
print(my_list)  # Output: [1, 2, 3, 4]
```

**Why it matters:** This behavior can be either a feature (in-place updates) or a bug source (unexpected side effects). Understanding reference semantics prevents accidental mutations.

**Case 2: Avoiding Unintended Aliasing**

```python
# Problematic: both variables reference the same list
matrix = [[0] * 3] * 3  # Creates three references to the SAME inner list
matrix[0][0] = 1
print(matrix)  # Output: [[1, 0, 0], [1, 0, 0], [1, 0, 0]]

# Correct: create independent lists
matrix = [[0] * 3 for _ in range(3)]
matrix[0][0] = 1
print(matrix)  # Output: [[1, 0, 0], [0, 0, 0], [0, 0, 0]]
```

**Why it matters:** The first approach appears to create a 3×3 matrix, but actually creates three references to the same inner list. Modifying one row modifies all rows.

---

## Core Concept 2: Naming Variables

### Definitions

**Core Definition**
Variable naming is the process of choosing valid, descriptive identifiers that follow language rules and community conventions.

**Technical Definition**
An identifier in Python must match the lexical rule `[a-zA-Z_][a-zA-Z0-9_]*` and must not be a reserved keyword. By convention (PEP 8), variable names use `snake_case` for regular variables, `UPPER_CASE` for constants, and `_single_leading_underscore` for internal-use names.

**Beginner-Friendly Explanation**
Names are how you and other programmers talk about your data. A good name like `total_price` tells everyone what the variable holds. A bad name like `tp` forces readers to guess. Python has rules about what characters you can use, and the community has agreed on a style (all lowercase with underscores between words) so code looks consistent across projects.

### Purposes

- **To make code readable** and self-documenting
- **To reduce cognitive load** by using descriptive identifiers that convey intent
- **To avoid errors** by preventing collisions with keywords or built-in names
- **To follow community standards** (PEP 8), ensuring consistency across projects
- **To communicate variable scope** through naming conventions (leading underscores)

### Syntax Rules and Structure

**Complete General Syntax**

```
identifier = value
```

**Breakdown:**
- `identifier`: Must start with a letter (a–z, A–Z) or underscore (`_`), followed by letters, digits, or underscores.
- No special characters (`@`, `$`, `-`, etc.) or spaces are allowed.
- Cannot be a Python reserved keyword.

**Syntax Rules**

| Rule | Description | Valid | Invalid |
|---|---|---|---|
| Start with letter or underscore | First character cannot be a digit | `_name`, `name1` | `1name` |
| Subsequent characters | Letters, digits, underscores only | `my_var2` | `my-var` |
| Case sensitivity | `Name` and `name` are different | — | — |
| No keywords | Cannot use `if`, `for`, `class`, etc. | `class_` | `class` |
| No special chars | `@`, `#`, `$`, `%`, spaces prohibited | `user_age` | `user age` |

**Constraints and Limitations**

- **Reserved keywords cannot be used**: Using `class = "Math"` raises `SyntaxError`; use `class_ = "Math"` instead.
- **Built-in shadowing**: Assigning to `sum = 0` overwrites the built-in `sum()` function; prefer `total_sum`.
- **Single leading underscore**: Conventionally indicates "internal use" (not enforced).
- **Double leading underscore**: Triggers name mangling in class contexts (e.g., `__private` becomes `_ClassName__private`).

### Multiple Annotated Complete Code Examples and Expected Outputs

**Example 1: Valid vs. Invalid Names**

```python
# Valid names (no errors)
user_name = "Alice"        # snake_case, descriptive
_total = 100               # leading underscore (internal convention)
item2 = "second"           # digit allowed after first character
CONSTANT_VALUE = 3.14      # UPPER_CASE for constants (convention)

print(user_name, _total, item2, CONSTANT_VALUE)
# Output: Alice 100 second 3.14

# Invalid names would cause SyntaxError:
# 2nd_item = "apple"       # starts with digit
# user-name = "Bob"        # hyphen not allowed
# class = "Biology"        # reserved keyword
```

**Expected Output:**
```
Alice 100 second 3.14
```

**Breakdown:** All valid names follow the identifier rule. Comments show what would fail and why.

**Example 2: Avoiding Built-in Shadowing**

```python
# PROBLEMATIC: shadowing the built-in 'sum'
sum = 0                    # This replaces the built-in sum() function
values = [1, 2, 3]
# sum(values) would now raise TypeError because 'sum' is an int, not a function

# FIX: use a distinct, descriptive name
total_sum = 0
for value in values:
    total_sum += value

print(f"Total: {total_sum}")
# Output: Total: 6
```

**Expected Output:**
```
Total: 6
```

**Breakdown:** By naming the accumulator `total_sum` instead of `sum`, the built-in function remains available for use.

**Example 3: PEP 8 snake_case Convention**

```python
# Favor this (PEP 8 compliant)
first_name = "John"
last_name = "Doe"
total_score = 95

# Avoid this (non-standard casing)
firstName = "John"         # camelCase (not PEP 8 for variables)
TotalScore = 95            # PascalCase (reserved for classes)

# Print combined name
full_name = f"{first_name} {last_name}"
print(full_name, total_score)
# Output: John Doe 95
```

**Expected Output:**
```
John Doe 95
```

**Breakdown:** PEP 8 recommends `snake_case` for variable names. This makes Python code visually consistent and immediately recognizable.

### Real-World Cases with Explanation

**Case 1: Descriptive Names in Data Processing**

```python
# Poor: cryptic names force readers to trace values
d = 0
for r in rows:
    d += r['amt']
print(d)

# Better: names communicate intent
total_sales = 0
for row in rows:
    total_sales += row['amount']
print(total_sales)
```

**Why it matters:** In production code, readability is critical. Descriptive names like `total_sales` and `row` make the logic self-evident without needing comments.

**Case 2: Constants and Configuration**

```python
# Module-level constants use UPPER_CASE
MAX_RETRIES = 3
DEFAULT_TIMEOUT = 30.0

def connect(host, port):
    # Local variables use snake_case
    connection_attempts = 0
    # ...
```

**Why it matters:** This convention immediately signals to readers that `MAX_RETRIES` and `DEFAULT_TIMEOUT` are configuration values that should not be reassigned during normal program flow.

---

## Core Concept 3: Multiple Assignment (Unpacking)

### Definitions

**Core Definition**
Multiple assignment allows a single statement to bind multiple names to multiple values extracted from an iterable.

**Technical Definition**
The statement `x, y = iterable` triggers **iterable unpacking**: the right-hand expression is evaluated to produce an iterable, then its elements are assigned to the left-hand targets in left-to-right order. The number of targets must match the number of elements unless the starred-target syntax (`*rest`) is used.

**Beginner-Friendly Explanation**
Instead of writing three separate lines to assign three variables, you can do it in one line: `x, y, z = 1, 2, 3`. Python automatically takes the values on the right and matches them to the names on the left. This is especially useful for swapping variables (`a, b = b, a`) or extracting values from a tuple or list.

### Purposes

- **To concisely assign multiple variables** from a single iterable
- **To swap values** between variables without a temporary variable
- **To unpack function return values** that return tuples
- **To extract specific elements** from sequences into named variables
- **To improve code readability** by grouping related assignments

### Syntax Rules and Structure

**Complete General Syntax**

```
target1, target2, ..., targetN = iterable
```

**Breakdown:**
- `target1 ... targetN`: Comma-separated names (or attribute/subscript targets).
- `=`: Assignment operator.
- `iterable`: Any iterable object (tuple, list, string, generator, etc.) whose length matches the number of targets.

**Extended Syntax (Starred Target):**

```
first, *rest = iterable
*init, last = iterable
```

The starred target (`*rest`) collects any number of elements into a list.

**Syntax Rules**

| Rule | Description |
|---|---|
| Length matching | Number of targets must equal number of elements (unless starred target used) |
| Right-hand evaluation | The entire right-hand iterable is evaluated before any assignment |
| Left-to-right assignment | Targets are assigned in order from left to right |
| Starred target | Only one starred target allowed per assignment; collects remaining elements as a list |
| Nested unpacking | Targets may themselves be tuples/lists for nested unpacking |

**Constraints and Limitations**

- **ValueError on mismatch**: If the iterable length doesn't match the number of targets (and no starred target is present), `ValueError` is raised.
- **Exactly one starred target**: You cannot have `*a, *b = [1, 2, 3]`; only one `*` is allowed.
- **Iterables only**: The right-hand side must be iterable; integers and non-iterable objects raise `TypeError`.

### Multiple Annotated Complete Code Examples and Expected Outputs

**Example 1: Basic Multiple Assignment**

```python
# Step 1: Assign three names from a tuple literal
x, y, z = 1, 2, 3
# The right side is a tuple (1, 2, 3); unpacking assigns:
# x -> 1, y -> 2, z -> 3

print(f"x={x}, y={y}, z={z}")
# Output: x=1, y=2, z=3

# Step 2: Unpack from a list
a, b = [10, 20]
print(f"a={a}, b={b}")
# Output: a=10, b=20
```

**Expected Output:**
```
x=1, y=2, z=3
a=10, b=20
```

**Breakdown:** The tuple on the right is unpacked element-by-element into the targets. The list on the right works the same way because Python iterates over it.

**Example 2: Swapping Variables**

```python
# Traditional swap (needs temporary variable)
a = 5
b = 10
temp = a
a = b
b = temp
print(f"Traditional: a={a}, b={b}")

# Pythonic swap using multiple assignment
a = 5
b = 10
a, b = b, a
# Right side evaluates to (10, 5) BEFORE any assignment
# Then a -> 10, b -> 5

print(f"Pythonic: a={a}, b={b}")
```

**Expected Output:**
```
Traditional: a=10, b=5
Pythonic: a=10, b=5
```

**Breakdown:** The right-hand side `(b, a)` is evaluated first, creating a tuple `(10, 5)`. Then the unpacking assigns `10` to `a` and `5` to `b`. This avoids the need for a temporary variable.

**Example 3: Starred Target (Extended Unpacking)**

```python
# Collect remaining elements into a list
first, *middle, last = [1, 2, 3, 4, 5]
# 'first' gets 1, 'last' gets 5, 'middle' gets [2, 3, 4]

print(f"first={first}")
print(f"middle={middle}")
print(f"last={last}")
# Output: first=1, middle=[2, 3, 4], last=5
```

**Expected Output:**
```
first=1
middle=[2, 3, 4]
last=5
```

**Breakdown:** The starred target `*middle` absorbs all elements that don't match other targets, collecting them into a list. Only one starred target is permitted.

### Real-World Cases with Explanation

**Case 1: Unpacking Function Return Values**

```python
def get_user_info():
    return "Alice", 30, "alice@example.com"

name, age, email = get_user_info()
print(f"Name: {name}, Age: {age}, Email: {email}")
# Output: Name: Alice, Age: 30, Email: alice@example.com
```

**Why it matters:** Functions commonly return multiple values as tuples. Multiple assignment lets you unpack them cleanly without indexing.

**Case 2: Iterating with Index**

```python
colors = ["red", "green", "blue"]
for index, color in enumerate(colors):
    print(f"{index}: {color}")
# Output:
# 0: red
# 1: green
# 2: blue
```

**Why it matters:** `enumerate()` returns an iterator of `(index, value)` tuples. Multiple assignment in the `for` loop unpacks each tuple automatically.

---

## Core Concept 4: Chained Assignment

### Definitions

**Core Definition**
Chained assignment binds the same object reference to multiple names in a single statement.

**Technical Definition**
The statement `a = b = c = expression` evaluates `expression` exactly once, producing a single object reference, then binds each target (from left to right) to that same reference. The targets are assigned in textual order after the rightmost expression is evaluated.

**Beginner-Friendly Explanation**
When you write `x = y = z = 10`, all three names end up pointing to the exact same object. It's a shortcut for `x = 10; y = 10; z = 10`, except the value is only computed once. This is fine for immutable values like integers and strings, but can cause surprises with mutable objects like lists, because all names share the same object.

### Purposes

- **To assign a single value to multiple names** concisely
- **To initialize several variables** to the same starting value
- **To avoid redundant evaluation** of the right-hand expression
- **To express that multiple names refer to the same initial object**

### Syntax Rules and Structure

**Complete General Syntax**

```
target1 = target2 = ... = targetN = expression
```

**Breakdown:**
- `target1 ... targetN`: Names (or attribute/subscript targets) to be bound.
- `=`: Assignment operators.
- `expression`: Evaluated **once**, producing a single object reference that is bound to every target.

**Syntax Rules**

| Rule | Description |
|---|---|
| Single evaluation | The rightmost expression is evaluated exactly once |
| Left-to-right assignment | Targets are assigned from left to right after expression evaluation |
| Same object | All targets reference the identical object (not copies) |
| No return value | Chained assignment is a statement, not an expression |

**Constraints and Limitations**

- **Mutability hazard**: If the shared object is mutable, modifying it through one name affects all names.
- **Evaluation order subtlety**: In rare cases involving generators or iterators, the left-to-right assignment order can produce surprising results (see example below).
- **Not equivalent to multiple statements with side effects**: `a = b = f()` calls `f()` only once, unlike `a = f(); b = f()`.

### Multiple Annotated Complete Code Examples and Expected Outputs

**Example 1: Chained Assignment with Immutable Values**

```python
# Assign the same integer to three names
x = y = z = 10
# Evaluation: the integer 10 is created (or reused), then x, y, z all point to it

print(f"x={x}, y={y}, z={z}")
# Output: x=10, y=10, z=10

# Verify they reference the same object (for small integers, Python caches them)
print(x is y, y is z)
# Output: True True
# 'is' checks identity (same object), and all three point to the same int object
```

**Expected Output:**
```
x=10, y=10, z=10
True True
```

**Breakdown:** Since `10` is immutable and evaluated once, all three names reference the same object. Rebinding one name (`x = 20`) would not affect the others.

**Example 2: Chained Assignment with Mutable Objects (Shared Reference Hazard)**

```python
# Assign the same list to three names
list1 = list2 = list3 = [1, 2, 3]
# All three names reference the SAME list object

# Modify the list through one name
list1.append(4)
# The list object itself changes: [1, 2, 3, 4]

print(f"list1: {list1}")
print(f"list2: {list2}")
print(f"list3: {list3}")
# All print the same modified list
```

**Expected Output:**
```
list1: [1, 2, 3, 4]
list2: [1, 2, 3, 4]
list3: [1, 2, 3, 4]
```

**Breakdown:** This is the classic pitfall. Because all three names reference the same mutable list, modifying through any name affects all. To create independent lists, use separate assignments: `list1 = [1,2,3]; list2 = [1,2,3]; list3 = [1,2,3]`.

**Example 3: Chained Assignment with Iterator (Evaluation Order Subtlety)**

```python
# Advanced: chained assignment with an iterator
a = b = iter(range(3))
# The iterator object is created once and bound to both a and b.
# They reference the SAME iterator, not separate iterators.

print(f"a: {list(a)}")
# Output: a: [0, 1, 2]
# The first list() consumes the entire iterator

print(f"b: {list(b)}")
# Output: b: []
# The iterator is exhausted, so b produces an empty list
```

**Expected Output:**
```
a: [0, 1, 2]
b: []
```

**Breakdown:** Since `a` and `b` reference the same iterator object, consuming it through `a` exhausts it for `b`. This demonstrates that chained assignment shares objects, including stateful iterators.

### Real-World Cases with Explanation

**Case 1: Initializing Multiple Counters**

```python
# Initialize three counters to zero
player1_score = player2_score = player3_score = 0

# Rebinding one score does NOT affect the others (integers are immutable)
player1_score = 50
print(player1_score, player2_score, player3_score)
# Output: 50 0 0
```

**Why it matters:** For immutable values, chained assignment is safe and convenient for setting a common starting state.

**Case 2: Avoiding the Mutable Shared Reference Bug**

```python
# PROBLEMATIC: all rows share the same list
grid = [[""] * 3] * 3
grid[0][0] = "X"
print(grid)
# Output: [['X', '', ''], ['X', '', ''], ['X', '', '']]

# CORRECT: independent lists
grid = [["" for _ in range(3)] for _ in range(3)]
grid[0][0] = "X"
print(grid)
# Output: [['X', '', ''], ['', '', ''], ['', '', '']]
```

**Why it matters:** This is one of the most common Python bugs. Understanding that chained assignment (and list multiplication) shares references prevents hours of debugging.

---

## References

- Python Software Foundation. *7. Simple statements — Python 3.12.0 documentation*. https://docs.python.org/3/reference/simple_stmts.html
- Python Software Foundation. *PEP 8 – Style Guide for Python Code*. https://peps.python.org/pep-0008/
- Purdue University. *Variables and Objects*. https://www.purdue.edu/hla/sites/varalalab/wp-content/uploads/sites/20/2018/03/Lecture_9.pdf
- Real Python. *Variables – Python Best Practices*. https://realpython.com/ref/best-practices/variables/
- Python.org. *How to Teach Python "Variables" (Python-list mailing list)*. https://mail.python.org/pipermail/python-list/2007-November/418490.html
- Python.org. *[Edu-sig] what is a "variable" in Python?* https://mail.python.org/pipermail/edu-sig/2016-February/011372.html
- Python.org. *Multiple Assignment a = b = c (Python-list mailing list)*. https://mail.python.org/archives/list/python-list@python.org/thread/36R2ML6LMTWMT4R5LQ2YSBBIROQPJRRE/
- Marc.info. *[Python-Dev] chained assignment weirdity*. https://marc.info/?l=python-dev&m=135221640625695&q=mbox
- LabEx. *What are Python variable naming rules?* https://labex.io/questions/what-are-python-variable-naming-rules-685383
- LabEx. *How do I name variables?* https://labex.io/questions/how-do-i-name-variables-598879
- Replit. *How to name variables in Python*. https://replit.com/discover/how-to-name-variables-in-python