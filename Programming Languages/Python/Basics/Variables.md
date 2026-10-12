# Python Variables — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition.** A Python variable is a symbolic name bound to an object stored in memory; the name is a reference (or “binding”) to that object, not the object itself. Assignment creates or rebinds that name to a value.

**Technical Definition.** In Python’s data model, every piece of data is represented by an object possessing three invariant properties: an identity (a unique integer, accessible via `id()`), a type (determining supported operations and mutability), and a value. A variable is a name in a namespace that holds a reference to such an object. Assignment stores a reference to the right-hand-side object in the namespace entry for the left-hand-side name; it does not copy the object.

**Beginner-Friendly Explanation.** Think of a variable as a labelled sticky note attached to a box. The box holds the actual data. Writing `x = 10` creates a note labelled `x` and sticks it onto the box containing the number 10. If you later write `x = 20`, you peel the `x` note off the old box and stick it onto a new box containing 20; the old box is unaffected unless nothing else references it.

### Key Characteristics

- **Dynamic typing.** Python infers the type of a variable from the object it currently references. A variable may be rebound to an object of a completely different type during execution.
- **Reference semantics.** Assignment never copies an object; it creates an additional reference to the same object. Multiple names can therefore refer to identical data.
- **First-class objects.** Variables can reference any object, including functions, classes, modules, and other variables’ values.
- **Namespaces and scope.** Variable names live in namespaces (local, enclosing, global, built-in) and are resolved according to the LEGB rule.
- **Mutability matters.** Whether an operation “changes” a variable depends on whether the referenced object is mutable (e.g., `list`, `dict`) or immutable (e.g., `int`, `str`, `tuple`).
- **No explicit declaration.** A variable is created the moment it is first assigned; there is no separate declaration keyword in ordinary code.

### Prerequisites

To follow this cheat sheet effectively, you should already be able to:

- Run a Python script or use the interactive interpreter (`python` or `python3`).
- Understand basic Python syntax: expressions, statements, indentation, and comments.
- Have a preliminary familiarity with built-in types such as `int`, `float`, `str`, `list`, `dict`, and `tuple`.
- Know how to define and call simple functions using `def`.

### Related Programming Areas

- **Object-oriented programming:** Understanding object identity and references is foundational to Python’s OOP model.
- **Functional programming:** Closures and `nonlocal` behaviour are integral to decorators and factory functions.
- **Memory management and performance:** Reference counting and garbage collection influence performance-sensitive code.
- **Concurrency:** Shared mutable state across threads depends on understanding variable scope and reference semantics.
- **Debugging and testing:** Name resolution errors (`UnboundLocalError`, `NameError`) are among the most common Python bugs.

### Core Concepts / Features

The remainder of this cheat sheet treats each major topic from the requested outline as a core concept, following a uniform structure: Definitions, Purposes (each beginning with “To”), Syntax Rules and Structure, Annotated Code Examples with expected outputs and explanations, and Real-World Cases.

---

## 1. Variables

### Definitions

**Core Definition.** A variable is a name that refers to a value stored in memory.

**Technical Definition.** A variable is a binding between an identifier (a name in a namespace) and a Python object. The binding is stored in a namespace dictionary; resolving the name retrieves the object reference.

**Beginner-Friendly Explanation.** A variable is a label you give to a piece of data so you can use it later without retyping the data itself.

### Purposes

- To store intermediate results of computations for later use.
- To give descriptive meaning to otherwise opaque values (e.g., `total_price` instead of `49.99`).
- To enable reuse of the same value at multiple points in a program.
- To support dynamic state: a variable’s referenced object can change over time.

### Syntax Rules and Structure

**Complete General Syntax.**

```python
variable_name = expression
```

**Component Breakdown.**

| Component | Meaning |
|---|---|
| `variable_name` | A valid Python identifier that will be bound to the result. |
| `=` | The assignment operator; not to be confused with mathematical equality (`==`). |
| `expression` | Any Python expression whose evaluated result becomes the referenced object. |

**Syntax Rules.**

- The right-hand side is evaluated completely before the assignment occurs.
- A variable must be assigned before it is used; otherwise a `NameError` is raised.
- Assignment is a statement, not an expression, so `a = (b = 5)` is illegal.
- Reassignment replaces the binding; the old object is unaffected unless it becomes unreachable.

**Constraints and Limitations.**

- A variable name cannot be a Python keyword (`if`, `for`, `class`, etc.).
- The assignment operator binds names, not values; it does not create a copy.
- Rebinding a variable that is referenced elsewhere does not affect those other references.

### Annotated Code Examples

**Example 1: Basic assignment and reassignment**

```python
# Step 1: Create a variable named score and bind it to the integer 100.
score = 100
print(score)          # Expected output: 100

# Step 2: Reassign score to a new integer object.
score = 250
print(score)          # Expected output: 250

# Step 3: Reassign score to a string object.
score = "high"
print(score)          # Expected output: high
```

**Why the output is what it is.** The first `print` shows 100 because `score` was bound to the integer 100. After the second assignment, the name `score` is rebound to the integer 250, so the second `print` shows 250. The third assignment binds `score` to the string `"high"`; Python does not enforce type consistency, demonstrating dynamic typing.

**Example 2: Right-hand side evaluation first**

```python
x = 5
y = x + 3    # x is evaluated to 5, then 3 is added, yielding 8.
print(y)     # Expected output: 8

x = 10       # Rebinding x does not change y.
print(y)     # Expected output: 8
```

**Why the output is what it is.** When `y = x + 3` executes, Python reads the current object referenced by `x` (the integer 5), computes 5 + 3 = 8, and binds `y` to the resulting integer. Later rebinding `x` to 10 has no effect on `y` because `y` already holds its own reference to 8.

### Real-World Cases

- **Configuration values.** `MAX_RETRIES = 3` at module level provides a single point of truth used throughout the codebase.
- **Accumulators.** `total = 0` followed by `total += price` in a loop is a standard pattern for summing values.
- **Temporary state.** `temp = data.copy()` preserves the original while allowing modification of a working copy.
- **Feature flags.** `DEBUG = False` can be toggled to control conditional behaviour without altering logic.

---

## 2. Variable Assignment

### Definitions

**Core Definition.** Variable assignment is the act of binding a name to an object using the `=` operator.

**Technical Definition.** The assignment statement evaluates the expression on the right-hand side, producing an object, and stores a reference to that object in the namespace entry for the target name on the left-hand side.

**Beginner-Friendly Explanation.** Assignment is how you put a label on a box: you choose the label name, and you choose what goes into the box.

### Purposes

- To introduce a new name into a namespace.
- To replace the object referenced by an existing name.
- To capture the result of a computation for subsequent use.
- To establish state that can be read and modified by later statements.

### Syntax Rules and Structure

The general syntax and component breakdown are identical to §1. The key rules are:

- The left-hand side must be a valid assignment target: a name, an attribute (`obj.attr`), an index (`container[key]`), or a slice.
- Only one `=` is used for assignment; `==` is the equality operator.
- Chained assignment (`a = b = c`) assigns the same object to multiple names, evaluated left-to-right at the name-binding level.

**Constraints and Limitations.**

- Assignment targets cannot be expressions such as `x + 1 = 5`.
- The right-hand side is fully evaluated before any binding occurs.
- If the right-hand side raises an exception, no binding takes place.

### Annotated Code Examples

**Example 1: Assignment to different target types**

```python
# Assign to a simple name.
count = 10
print(count)                     # Expected output: 10

# Assign to an attribute of an object.
class Box:
    pass
b = Box()
b.value = 42
print(b.value)                   # Expected output: 42

# Assign to an index in a list.
items = [1, 2, 3]
items[1] = 99
print(items)                     # Expected output: [1, 99, 3]

# Assign to a slice.
items[0:2] = [7, 8]
print(items)                     # Expected output: [7, 8, 3]
```

**Why the output is what it is.** Each assignment targets a different kind of mutable location: a namespace entry, an object attribute, a list element, and a list slice. Python evaluates the right-hand side and stores the reference in the appropriate location. The list slice assignment replaces the first two elements with two new elements.

### Real-World Cases

- **Reading user input.** `name = input("Enter your name: ")` captures the string returned by `input()`.
- **Function return values.** `result = compute(x, y)` stores the returned object.
- **Updating object state.** `self.balance = self.balance - amount` modifies an attribute within a method.
- **Building data structures.** `matrix[0][0] = 1` assigns to a nested container.

---

## 3. Multiple Assignment

### Definitions

**Core Definition.** Multiple assignment (also called tuple unpacking or iterable unpacking) binds several names to the elements of an iterable in a single statement.

**Technical Definition.** The statement `a, b = iterable` evaluates the right-hand iterable, then assigns each element in order to the corresponding target on the left. The number of targets must equal the number of elements unless starred-target syntax is used.

**Beginner-Friendly Explanation.** Multiple assignment lets you give labels to several pieces of data at once, like taking items out of a box and putting each one on its own shelf.

### Purposes

- To unpack a sequence into named variables in one concise statement.
- To swap the values of two variables without a temporary variable.
- To capture multiple return values from a function directly.
- To improve readability when a sequence has a known structure (e.g., coordinates).

### Syntax Rules and Structure

**Complete General Syntax.**

```python
target1, target2, ..., targetN = iterable
```

**Starred form:**

```python
first, *middle, last = iterable
```

**Component Breakdown.**

| Component | Meaning |
|---|---|
| `target1, target2, ...` | Comma-separated assignment targets. |
| `*middle` | A starred target that collects remaining elements into a list. |
| `iterable` | Any iterable object (tuple, list, string, generator, etc.). |

**Syntax Rules.**

- The number of non-starred targets must match the number of iterable elements.
- At most one starred target may appear on the left-hand side.
- The starred target always receives a list, even if empty.
- Nested unpacking is supported: `(a, b), c = (1, 2), 3`.

**Constraints and Limitations.**

- A `ValueError` is raised if the number of targets does not match the iterable length.
- Unpacking a string assigns individual characters.
- Unpacking is not recursive by default; nested structures must be unpacked explicitly.

### Annotated Code Examples

**Example 1: Basic tuple unpacking**

```python
# Unpack a 3-element tuple.
point = (10, 20, 30)
x, y, z = point
print(x, y, z)          # Expected output: 10 20 30

# Swap two variables without a temporary variable.
a, b = 1, 2
a, b = b, a
print(a, b)             # Expected output: 2 1
```

**Why the output is what it is.** The first unpacking assigns the three elements of `point` to `x`, `y`, and `z` in order. The swap works because the right-hand tuple `(b, a)` is fully evaluated before any assignment occurs; then `a` receives the old value of `b`, and `b` receives the old value of `a`.

**Example 2: Starred unpacking**

```python
first, *rest = [1, 2, 3, 4, 5]
print(first)   # Expected output: 1
print(rest)    # Expected output: [2, 3, 4, 5]

*initial, last = (10, 20, 30)
print(initial) # Expected output: [10, 20]
print(last)    # Expected output: 30
```

**Why the output is what it is.** The starred target collects all remaining elements into a list. When the starred target appears first, it collects everything except the final element; when it appears last, it collects everything except the first element.

### Real-World Cases

- **Coordinate pairs.** `x, y = (3.5, -2.1)` for geometric computations.
- **Multiple return values.** `quotient, remainder = divmod(17, 5)` is a classic example.
- **Loop over key–value pairs.** `for key, value in my_dict.items():` is the most common use of multiple assignment.
- **Parsing structured data.** `name, age, city = row` when reading rows from a CSV-like source.

---

## 4. Chained Assignment

### Definitions

**Core Definition.** Chained assignment binds several names to the same object in a single statement.

**Technical Definition.** The statement `a = b = c` evaluates the expression `c` once, then assigns the resulting object to `b` and `a` (binding proceeds right-to-left in terms of the assignment sequence, but the expression is evaluated once).

**Beginner-Friendly Explanation.** Chained assignment is a shortcut for saying “set all of these labels to the same thing.”

### Purposes

- To initialise multiple variables to the same starting value concisely.
- To reduce repetitive assignment statements when the value is genuinely shared.
- To express intent that several names should reference the identical object.

### Syntax Rules and Structure

**Complete General Syntax.**

```python
name1 = name2 = ... = nameN = expression
```

**Component Breakdown.**

| Component | Meaning |
|---|---|
| `name1, name2, ..., nameN` | Two or more assignment targets. |
| `expression` | A single expression evaluated once. |

**Syntax Rules.**

- The expression is evaluated only once.
- All names receive references to the same object.
- If the object is mutable, mutating it through one name affects all names.

**Constraints and Limitations.**

- Chained assignment is dangerous with mutable objects: `a = b = []` creates two names for the same list, so `a.append(1)` also affects `b`.
- It cannot be used to assign different values to different names.

### Annotated Code Examples

**Example 1: Chained assignment with an immutable value**

```python
# All three names refer to the same integer object.
a = b = c = 100
print(a, b, c)        # Expected output: 100 100 100

# Rebinding one name does not affect the others.
a = 200
print(a, b, c)        # Expected output: 200 100 100
```

**Why the output is what it is.** Initially, `a`, `b`, and `c` all reference the integer 100. Because integers are immutable, rebinding `a` to 200 simply changes the namespace entry for `a`; `b` and `c` still reference 100.

**Example 2: The mutable-object gotcha**

```python
# All names refer to the SAME list object.
x = y = []
x.append(1)
print(y)              # Expected output: [1]  ← y is affected!
```

**Why the output is what it is.** `x` and `y` both reference the same list. `x.append(1)` mutates that list in place; because `y` references the same object, `y` “sees” the change. To give each name its own independent list, use separate assignments: `x = []; y = []`.

### Real-World Cases

- **Initialising several counters.** `i = j = k = 0` is safe because integers are immutable.
- **Placeholder values.** `first = second = third = None` is a common pattern.
- **Avoiding the mutable trap.** In configuration code, `DEFAULT_OPTIONS = USER_OPTIONS = {}` is dangerous; explicit separate dictionaries are preferred.

---

## 5. Augmented Assignment

### Definitions

**Core Definition.** Augmented assignment combines a binary operation with assignment in a single statement, using operators such as `+=`, `-=`, `*=`, and `/=`.

**Technical Definition.** The statement `x += y` attempts to call `x.__iadd__(y)` (the in-place addition hook). If that method is unavailable, Python falls back to `x = x + y` using the normal binary addition and then rebinding the name.

**Beginner-Friendly Explanation.** Augmented assignment is a shorthand for “take the current value, do some arithmetic with it, and store the result back in the same variable.”

### Purposes

- To update a variable in place concisely.
- To signal that the left-hand operand should be modified in place when possible.
- To avoid repeating the variable name on both sides of an assignment.
- To enable mutable objects to optimise in-place modifications.

### Syntax Rules and Structure

**Complete General Syntax.**

```python
target <op>= expression
```

**Supported operators:**

| Operator | Equivalent fallback |
|---|---|
| `+=` | `target = target + expression` |
| `-=` | `target = target - expression` |
| `*=` | `target = target * expression` |
| `/=` | `target = target / expression` |
| `//=` | `target = target // expression` |
| `%=` | `target = target % expression` |
| `**=` | `target = target ** expression` |
| `<<=` | `target = target << expression` |
| `>>=` | `target = target >> expression` |
| `&=` | `target = target & expression` |
| `\^=` | `target = target ^ expression` |
| `\|=` | `target = target \| expression` |

**Component Breakdown.**

| Component | Meaning |
|---|---|
| `target` | A valid assignment target. |
| `<op>` | The augmented operator, combining the binary operation with `=`. |
| `expression` | The right-hand operand. |

**Syntax Rules.**

- The left-hand side is evaluated only once, even in the fallback case.
- For immutable objects, augmented assignment is effectively a rebinding.
- For mutable objects that define `__iadd__` (e.g., `list`), the object is modified in place.
- The in-place method receives the right-hand operand and should return the result (often `self`).

**Constraints and Limitations.**

- Augmented assignment is a statement, not an expression; `y = (x += 1)` is illegal.
- Not all objects support all augmented operators; unsupported combinations raise `TypeError`.
- In-place modification of a mutable object shared by multiple names affects all references.

### Annotated Code Examples

**Example 1: Augmented assignment with an integer (immutable)**

```python
count = 10
count += 5          # Equivalent to count = count + 5
print(count)        # Expected output: 15

count *= 2          # Equivalent to count = count * 2
print(count)        # Expected output: 30
```

**Why the output is what it is.** Integers are immutable, so `count += 5` creates a new integer object (15) and rebinds `count` to it. The original integer 10 is unchanged. The second operation creates another new integer (30) and rebinds again.

**Example 2: Augmented assignment with a list (mutable, in-place)**

```python
items = [1, 2]
other = items           # other references the SAME list.
items += [3, 4]         # In-place: extends the list.
print(items)            # Expected output: [1, 2, 3, 4]
print(other)            # Expected output: [1, 2, 3, 4]
```

**Why the output is what it is.** The `list` type implements `__iadd__`, which calls `list.extend()` and returns the same list object. Because `other` references the same list, the mutation is visible through both names. This differs from `items = items + [3, 4]`, which would create a new list and leave `other` pointing to the old one.

### Real-World Cases

- **Accumulating totals.** `total += price` in a loop.
- **Building strings.** `message += f"{name}: {score}\n"` (though `join()` is preferred for many concatenations).
- **Bit manipulation.** `flags |= MASK` to set bits, `flags &= ~MASK` to clear bits.
- **In-place list growth.** `results += new_items` is more efficient than `results = results + new_items` for large lists.

---

## 6. Variable Naming

### Definitions

**Core Definition.** Variable naming is the process of choosing valid, meaningful identifiers for variables according to Python’s lexical rules and community conventions.

**Technical Definition.** A Python identifier must match the grammar production `identifier ::= (letter | "_") (letter | digit | "_")*`, with additional restrictions that keywords are reserved and certain names have special meaning in interactive mode.

**Beginner-Friendly Explanation.** Variable names are the labels you write in your code; Python has strict rules about which labels are allowed, and programmers follow additional style conventions to make code readable.

### Purposes

- To satisfy the interpreter’s lexical requirements so the code can run at all.
- To communicate the purpose of a variable to human readers.
- To avoid collisions with keywords, built-in names, and other variables.
- To establish consistent style across a codebase, aiding maintenance.

### Syntax Rules and Structure

**Complete General Syntax.**

```
identifier ::= (letter | "_") (letter | digit | "_")*
```

where `letter` includes Unicode letters as defined by the language reference.

**Component Breakdown.**

| Component | Rule |
|---|---|
| First character | Must be a letter (A–Z, a–z, or Unicode letter) or underscore `_`. Never a digit. |
| Subsequent characters | Letters, digits, or underscores. |
| Case sensitivity | `name` and `Name` are distinct identifiers. |
| Keywords | Cannot be used as identifiers (`if`, `for`, `class`, `return`, etc.). |

**PEP 8 Conventions (advisory).**

| Category | Convention | Example |
|---|---|---|
| Regular variables | `snake_case` | `user_name` |
| Constants | `UPPER_CASE` | `MAX_RETRIES` |
| “Private” names | Leading underscore | `_internal` |
| Name-mangled (class) | Double leading underscore | `__private` |

**Constraints and Limitations.**

- Names cannot contain spaces or punctuation other than underscores.
- Names cannot start with a digit (`1st_place` is illegal).
- Single-character names like `l`, `O`, and `I` are discouraged because they can be confused with `1` and `0`.
- Although Unicode letters are technically allowed, ASCII-only names are strongly preferred for portability.

### Annotated Code Examples

**Example 1: Valid versus invalid names**

```python
# Valid names
user_name = "Alice"
_total = 100
score2 = 85

# Invalid names (would raise SyntaxError)
# 2nd_place = 10     # starts with a digit
# user-name = "Bob"  # hyphen is not allowed
# class = "Math"     # 'class' is a keyword
```

**Why the output is what it is.** Python’s tokenizer reads the source and rejects any identifier that violates the grammar. The invalid lines are commented out because they would prevent the entire script from being parsed.

**Example 2: Snake case versus other styles**

```python
# Recommended (PEP 8)
student_name = "Alice"
total_score = 95

# Discouraged but legal
StudentName = "Alice"   # PascalCase (reserved for classes)
totalScore = 95         # camelCase (not PEP 8 for variables)
```

**Why the output is what it is.** All four names are lexically valid and will execute without error. The distinction is purely stylistic: `snake_case` aligns with the widely adopted PEP 8 style guide, while `PascalCase` and `camelCase` are conventionally reserved for classes and (in other languages) methods.

### Real-World Cases

- **Web development.** `request_body`, `user_id`, `is_authenticated` follow snake_case conventions in frameworks like Django and Flask.
- **Data science.** `df_train`, `feature_columns`, `target_vector` are common descriptive names.
- **Library APIs.** Public names avoid leading underscores; internal helpers use a single leading underscore to signal non-public status.

---

## 7. Variable Scope

### Definitions

**Core Definition.** Variable scope is the region of a program in which a variable name is accessible and meaningful.

**Technical Definition.** Python resolves names using the **LEGB rule**: Local → Enclosing → Global → Built-in. When a name is referenced, Python searches these namespaces in order and raises `NameError` if the name is not found.

**Beginner-Friendly Explanation.** Scope determines “who can see this variable.” A variable created inside a function is generally invisible outside it, just as a note written inside a room cannot be read from the hallway.

### Purposes

- To prevent accidental interference between different parts of a program.
- To allow local reasoning about a function without knowing the entire module.
- To enable nested functions to share state through enclosing scopes.
- To provide a predictable name-resolution order.

### Syntax Rules and Structure

**LEGB Resolution Order.**

| Level | Scope | Where Defined |
|---|---|---|
| **L**ocal | Innermost function | Parameters and assignments inside the current function. |
| **E**nclosing | Outer function (for nested functions) | Variables in any enclosing function scope, excluding globals. |
| **G**lobal | Module level | Names assigned at the top level of a module or declared `global`. |
| **B**uilt-in | Python built-ins | Names preloaded by the interpreter (`len`, `print`, etc.). |

**Constraints and Limitations.**

- A name assigned anywhere inside a function is considered local to that function unless declared `global` or `nonlocal`.
- Reading a local variable before assignment raises `UnboundLocalError`.
- Class bodies introduce a scope that is **not** searched by methods; methods look through the class scope to the enclosing module scope.

### Annotated Code Examples

**Example 1: Local versus global visibility**

```python
x = "global"          # Module-level (global) name.

def show_global():
    print(x)          # Reads the global x because no local x is assigned.

def shadow_global():
    x = "local"       # Creates a NEW local variable; does not affect global x.
    print(x)

show_global()         # Expected output: global
shadow_global()       # Expected output: local
print(x)              # Expected output: global
```

**Why the output is what it is.** `show_global` does not assign to `x`, so Python resolves `x` through LEGB and finds the global one. `shadow_global` assigns `x = "local"`, which makes `x` local to that function; the global `x` is unaffected. The final `print(x)` at module level sees the global `x`, which is still `"global"`.

**Example 2: `UnboundLocalError`**

```python
counter = 0

def increment():
    print(counter)    # This line raises UnboundLocalError.
    counter += 1      # Because counter is assigned here, it is local throughout.

# increment()          # Uncommenting this line raises the error.
```

**Why the error occurs.** Python scans the function body and sees the assignment `counter += 1`. It therefore treats `counter` as a local variable for the entire function. When `print(counter)` executes, the local `counter` has not yet been assigned, producing `UnboundLocalError`. To fix this, declare `global counter` at the top of the function.

### Real-World Cases

- **Configuration globals.** `DEBUG = True` at module level is read by many functions without being passed explicitly.
- **Closures.** A factory function returns an inner function that “remembers” a variable from its enclosing scope.
- **Recursion.** Each recursive call creates a new local scope with its own parameters and locals.
- **Avoiding global state.** Passing parameters explicitly and returning values is preferred over modifying globals inside functions.

---

## 8. Global Variables

### Definitions

**Core Definition.** A global variable is a name defined at the top level of a module (or declared `global` inside a function) that is accessible throughout the module.

**Technical Definition.** Module-level names are stored in the module’s global namespace dictionary. A function can read a global name directly, but to rebind it the function must declare `global name`.

**Beginner-Friendly Explanation.** A global variable is a note pinned to the wall of the main room; everyone in the building can read it, but to change what it says you must explicitly announce that you are modifying the wall note.

### Purposes

- To provide module-wide configuration or state.
- To allow multiple functions to share data without passing parameters.
- To implement singleton-like patterns in simple scripts.
- To expose public API names from a module.

### Syntax Rules and Structure

**Reading a global inside a function:**

```python
GLOBAL_NAME = value

def func():
    print(GLOBAL_NAME)    # Read access works without declaration.
```

**Rebinding a global inside a function:**

```python
GLOBAL_NAME = value

def func():
    global GLOBAL_NAME
    GLOBAL_NAME = new_value   # Rebinds the module-level name.
```

**Component Breakdown.**

| Component | Meaning |
|---|---|
| `global` | A declaration keyword that tells the parser that the following names refer to module-level bindings. |
| `GLOBAL_NAME` | The name to be rebound. |

**Constraints and Limitations.**

- Using `global` is widely considered a code smell because it introduces hidden dependencies and makes testing harder.
- A `global` declaration must appear before any use or assignment of the name in that function.
- The `global` statement does not create a variable; it only changes the scope of an existing or future binding.

### Annotated Code Examples

**Example 1: Reading a global variable**

```python
CONFIG = {"debug": True, "version": "1.0"}

def show_config():
    print(CONFIG)          # Reads the global dictionary.

show_config()
# Expected output: {'debug': True, 'version': '1.0'}
```

**Why the output is what it is.** `CONFIG` is defined at module level. `show_config` does not assign to `CONFIG`, so Python resolves it through the global namespace and prints the dictionary.

**Example 2: Rebinding a global with `global`**

```python
status = "idle"

def activate():
    global status
    status = "running"

print(status)    # Expected output: idle
activate()
print(status)    # Expected output: running
```

**Why the output is what it is.** Without the `global` declaration, `status = "running"` would create a local variable. The declaration forces the assignment to target the module-level name. The first `print` shows the original value; after `activate()`, the module-level `status` has been rebound.

### Real-World Cases

- **Feature flags in scripts.** `VERBOSE = False` toggled by a command-line argument.
- **Caching.** A module-level cache dictionary shared across functions.
- **Application state in small programs.** `current_user = None` updated by login/logout functions.
- **Library internals.** Some standard-library modules use module-level sentinels and configuration objects.

---

## 9. Local Variables

### Definitions

**Core Definition.** A local variable is a name assigned inside a function (or method) whose scope is limited to that function’s execution.

**Technical Definition.** Local variables live in the function’s local namespace, which is created when the function is called and discarded when it returns. Parameters are also local variables.

**Beginner-Friendly Explanation.** A local variable is a note written inside a room; it is only readable while you are in that room and is thrown away when you leave.

### Purposes

- To hold temporary values that are not needed outside the function.
- To avoid polluting the global namespace.
- To make functions self-contained and easier to test.
- To enable recursion by giving each call its own independent state.

### Syntax Rules and Structure

Any name assigned inside a function body (including parameters and loop variables) is local by default. The only exceptions are names declared `global` or `nonlocal`.

**Constraints and Limitations.**

- Local variables cannot be accessed from outside the function.
- Each function call gets a fresh local namespace; locals do not persist between calls.
- A local variable must be assigned before it is read within the function.

### Annotated Code Examples

**Example 1: Local variable lifetime**

```python
def compute():
    temp = 42          # Local variable.
    return temp

result = compute()
print(result)          # Expected output: 42
# print(temp)          # NameError: temp is not defined outside the function.
```

**Why the output is what it is.** `temp` is created inside `compute`’s local namespace and returned as the function’s result. After the function returns, the local namespace is destroyed, so `temp` is no longer accessible at module level. The caller receives the returned value and binds it to `result`.

**Example 2: Recursion uses independent local scopes**

```python
def factorial(n):
    if n <= 1:
        return 1
    return n * factorial(n - 1)

print(factorial(5))    # Expected output: 120
```

**Why the output is what it is.** Each recursive call to `factorial` has its own local `n`. The call `factorial(5)` multiplies 5 by the result of `factorial(4)`, which in turn multiplies 4 by `factorial(3)`, and so on. Because each call’s `n` is local, there is no interference between recursive levels.

### Real-World Cases

- **Utility functions.** Temporary calculations inside a function that are irrelevant to the caller.
- **Recursive algorithms.** Each recursive call maintains its own local state.
- **Event handlers.** Callback functions use local variables to process event data.
- **Data transformation pipelines.** Local variables hold intermediate stages of a transformation.

---

## 10. Nonlocal Variables

### Definitions

**Core Definition.** A nonlocal variable is a name in an enclosing function scope (not the global scope) that an inner function can rebind using the `nonlocal` keyword.

**Technical Definition.** The `nonlocal` declaration causes the named variable to refer to the nearest enclosing scope that is not global. It allows inner functions to modify variables in outer functions, enabling stateful closures.

**Beginner-Friendly Explanation.** Imagine a nested set of rooms. A `nonlocal` declaration lets someone in the innermost room change a note written in the middle room, without affecting notes in the outer building (globals).

### Purposes

- To allow a nested function to update a counter or accumulator defined in its enclosing function.
- To implement closures that maintain state across calls.
- To avoid using global variables for state that is only shared between an outer function and its inner function.
- To support decorator patterns that track call counts or cache results.

### Syntax Rules and Structure

**Complete General Syntax.**

```python
def outer():
    x = initial_value
    def inner():
        nonlocal x
        x = new_value
    inner()
    return x
```

**Component Breakdown.**

| Component | Meaning |
|---|---|
| `nonlocal x` | Declares that `x` refers to the nearest enclosing non-global scope. |
| `x = new_value` | Rebinds the enclosing `x` rather than creating a local `x`. |

**Syntax Rules.**

- The `nonlocal` declaration must appear before any use or assignment of the name in the inner function.
- The target name must exist in an enclosing function scope; otherwise a `SyntaxError` is raised.
- `nonlocal` cannot refer to a global variable; for that, use `global`.
- A name can be declared both `global` and `nonlocal` only if they are different names.

**Constraints and Limitations.**

- `nonlocal` is only available in Python 3 and later (PEP 3104).
- It only searches enclosing function scopes, skipping the global and built-in scopes.
- Overuse of `nonlocal` can make code harder to follow; explicit state objects or classes may be clearer in complex cases.

### Annotated Code Examples

**Example 1: Stateful counter with `nonlocal`**

```python
def make_counter():
    count = 0
    def increment():
        nonlocal count
        count += 1
        return count
    return increment

counter = make_counter()
print(counter())   # Expected output: 1
print(counter())   # Expected output: 2
print(counter())   # Expected output: 3
```

**Why the output is what it is.** `make_counter` creates a local `count` and an inner function `increment`. The `nonlocal count` declaration tells `increment` to rebind the `count` in `make_counter`’s scope. Each call to `counter()` increments that same persistent `count`, so the output increases by 1 each time.

**Example 2: `nonlocal` without a matching enclosing variable**

```python
def outer():
    def inner():
        nonlocal y    # SyntaxError: no binding for nonlocal 'y' found
        y = 10
    inner()

# outer()  # Uncommenting raises SyntaxError.
```

**Why the error occurs.** The `nonlocal` declaration requires that the name already exists in an enclosing function scope. Since `y` is not defined in `outer` (or any enclosing function), Python raises `SyntaxError` at compile time.

### Real-World Cases

- **Decorators.** A decorator factory uses `nonlocal` to count how many times the decorated function has been called.
- **Closures for callbacks.** Event handlers that maintain state between invocations.
- **Memoisation.** A cache dictionary in an enclosing scope updated by an inner function.
- **State machines.** Enclosing variables represent states, and inner functions transition between them.

---

## 11. `global` Keyword

### Definitions

**Core Definition.** The `global` keyword is a declaration that one or more names inside a function refer to bindings in the module-level (global) namespace.

**Technical Definition.** `global name` instructs the compiler to emit `LOAD_GLOBAL` and `STORE_GLOBAL` instructions for `name` within the function, bypassing the local namespace.

**Beginner-Friendly Explanation.** Writing `global x` inside a function is like announcing, “When I say `x`, I mean the `x` written on the wall outside this room, not a new note inside the room.”

### Purposes

- To rebind a module-level variable from inside a function.
- To modify shared configuration or state without passing it as a parameter.
- To implement module-level counters or flags.
- To interoperate with legacy code that relies on global state.

### Syntax Rules and Structure

**Complete General Syntax.**

```python
def func():
    global name1, name2, ...
    # assignments to name1 and name2 now target module-level bindings.
```

**Component Breakdown.**

| Component | Meaning |
|---|---|
| `global` | The declaration keyword. |
| `name1, name2, ...` | One or more identifiers that should be treated as global. |

**Syntax Rules.**

- The `global` declaration may appear anywhere in the function body, but it must precede any use or assignment of the declared names.
- Multiple names can be declared in a single `global` statement, separated by commas.
- The declaration applies to the entire function scope, not just the line on which it appears.

**Constraints and Limitations.**

- Using `global` is generally discouraged for state management; explicit parameter passing and return values are preferred.
- A `global` declaration at module level has no effect (it is redundant).
- `global` cannot be used to create a new global variable at runtime from inside a function in a way that surprises readers; the name is created if it does not already exist when assigned.

### Annotated Code Examples

**Example 1: Rebinding a global counter**

```python
call_count = 0

def record_call():
    global call_count
    call_count += 1

record_call()
record_call()
print(call_count)    # Expected output: 2
```

**Why the output is what it is.** Without the `global` declaration, `call_count += 1` would create a local variable and raise `UnboundLocalError`. The declaration directs the augmented assignment to the module-level `call_count`, so each call increments the shared counter.

**Example 2: `global` with multiple names**

```python
width = 10
height = 20

def resize():
    global width, height
    width = 100
    height = 200

resize()
print(width, height)   # Expected output: 100 200
```

**Why the output is what it is.** Both `width` and `height` are declared global, so the assignments inside `resize` rebind the module-level names. After the call, the module-level values reflect the new assignments.

### Real-World Cases

- **Simple scripts.** A small utility script uses `global VERBOSE` to toggle output verbosity.
- **Module-level caches.** `global _cache` to store results across calls without a class.
- **Game state in prototypes.** Global variables for score, lives, and level in small games.
- **Legacy code migration.** Understanding `global` helps when refactoring older Python codebases toward cleaner patterns.

---

## 12. `nonlocal` Keyword

### Definitions

**Core Definition.** The `nonlocal` keyword declares that a name in a nested function refers to a binding in the nearest enclosing function scope that is not global.

**Technical Definition.** `nonlocal name` causes the compiler to emit `LOAD_DEREF` and `STORE_DEREF` instructions that access a cell variable shared between the inner function and its enclosing function.

**Beginner-Friendly Explanation.** A `nonlocal` declaration is like telling the inner function, “Don’t create your own copy of this variable; use the one from the enclosing room.”

### Purposes

- To allow inner functions to modify state in enclosing functions.
- To implement closures with mutable state.
- To avoid the use of `global` for state that is only shared between nested functions.
- To support functional programming patterns such as accumulators and generators.

### Syntax Rules and Structure

The syntax is identical to `global` but with the keyword `nonlocal`. The target name must exist in an enclosing function scope.

**Constraints and Limitations.**

- `nonlocal` cannot target a global or built-in name.
- The name must be bound in an enclosing function scope before the `nonlocal` declaration is encountered (at compile time).
- `nonlocal` is Python 3+ only.

### Annotated Code Examples

**Example 1: Accumulator closure**

```python
def make_accumulator():
    total = 0
    def add(amount):
        nonlocal total
        total += amount
        return total
    return add

acc = make_accumulator()
print(acc(10))     # Expected output: 10
print(acc(5))      # Expected output: 15
print(acc(20))     # Expected output: 35
```

**Why the output is what it is.** `total` is a local variable in `make_accumulator`. The inner function `add` declares `nonlocal total`, allowing it to rebind that variable. Because `add` is returned and stored in `acc`, the enclosing scope of `total` persists across calls, so the accumulated total continues to grow.

**Example 2: `nonlocal` versus `global`**

```python
x = "global"

def outer():
    x = "enclosing"
    def inner():
        nonlocal x
        x = "modified"
    inner()
    print(x)        # Expected output: modified

outer()
print(x)            # Expected output: global
```

**Why the output is what it is.** `inner` uses `nonlocal x`, which resolves to the `x` in `outer`, not the global `x`. The inner assignment changes the enclosing `x` to `"modified"`. The module-level `x` remains `"global"` because `nonlocal` never touches the global scope.

### Real-World Cases

- **Memoisation decorators.** The cache dictionary lives in the decorator’s enclosing scope and is updated by the wrapper.
- **Stateful generators.** A generator function uses `nonlocal` to track its internal state.
- **Callback factories.** Creating callbacks that remember configuration values from their creation context.
- **Test fixtures.** A fixture function creates a resource and returns inner functions that use `nonlocal` to manage the resource’s lifecycle.

---

## 13. Constants

### Definitions

**Core Definition.** A constant is a value that is not intended to change during the execution of a program.

**Technical Definition.** Python has no built-in mechanism for enforcing constantness at runtime. Instead, constants are implemented by convention: module-level names written in `UPPER_CASE` with underscores are treated as constants by convention.

**Beginner-Friendly Explanation.** A constant is a value you promise not to change. Python cannot stop you from changing it, but writing the name in ALL_CAPS tells other programmers, “Please don’t modify this.”

### Purposes

- To give a descriptive name to a fixed value (e.g., `MAX_CONNECTIONS = 100`).
- To avoid “magic numbers” scattered throughout code.
- To provide a single point of change for configuration values.
- To signal intent to human readers through naming convention.

### Syntax Rules and Structure

**Convention:**

```python
CONSTANT_NAME = value
```

**Component Breakdown.**

| Component | Meaning |
|---|---|
| `CONSTANT_NAME` | All-uppercase letters with underscores separating words. |
| `value` | The fixed value. |

**Enforcement (optional):**

- Use `typing.Final` to annotate constants for static type checkers: `MAX: Final = 100`.
- Use a custom descriptor or `enum` for stricter enforcement in larger codebases.

**Constraints and Limitations.**

- Python does not prevent reassignment of a constant.
- The convention is purely advisory; it relies on developer discipline and code review.
- `typing.Final` is checked by static analysers (mypy, pyright) but not enforced at runtime.

### Annotated Code Examples

**Example 1: Module-level constants**

```python
# config.py
MAX_RETRIES = 3
DEFAULT_TIMEOUT = 30
BASE_URL = "https://api.example.com"

# main.py
import config
print(config.MAX_RETRIES)      # Expected output: 3
print(config.DEFAULT_TIMEOUT)  # Expected output: 30
```

**Why the output is what it is.** The constants are defined at module level in `config.py` and accessed via the module name. The `UPPER_CASE` naming convention signals that these values should not be modified.

**Example 2: `Final` annotation for static checking**

```python
from typing import Final

MAX_SIZE: Final = 1024
# MAX_SIZE = 2048  # mypy would report an error; Python runtime would allow it.
print(MAX_SIZE)       # Expected output: 1024
```

**Why the output is what it is.** `Final` is a type-checker annotation. At runtime, Python ignores it and `MAX_SIZE` behaves as an ordinary variable. Static analysis tools like mypy will flag reassignment as a type error, providing an additional layer of protection.

### Real-World Cases

- **Mathematical constants.** `PI = 3.141592653589793` at module level.
- **API endpoints.** `BASE_URL = "https://api.example.com/v2"`.
- **Limits and thresholds.** `MAX_UPLOAD_SIZE = 10 * 1024 * 1024`.
- **Configuration defaults.** `DEFAULT_LANGUAGE = "en"`.

---

## 14. `del` Statement

### Definitions

**Core Definition.** The `del` statement removes a binding from a namespace or removes an element from a mutable container.

**Technical Definition.** `del name` unbinds the name from its current object, decrementing the object’s reference count. If the reference count reaches zero, the object becomes eligible for garbage collection. `del container[key]` or `del container[index]` invokes the container’s deletion protocol.

**Beginner-Friendly Explanation.** `del` is like peeling a label off a box. If no other labels remain on the box, the box itself is discarded.

### Purposes

- To explicitly release a name when it is no longer needed.
- To remove an element from a list or dictionary.
- To free memory by eliminating references to large objects.
- To make code clearer by indicating that a variable’s lifetime has ended.

### Syntax Rules and Structure

**Complete General Syntax.**

```python
del target1, target2, ...
```

where each `target` is one of:

- A name: `del x`
- An attribute: `del obj.attr`
- An index: `del container[index]`
- A slice: `del container[start:stop]`

**Component Breakdown.**

| Component | Meaning |
|---|---|
| `del` | The statement keyword. |
| `target` | The name, attribute, or container element to delete. |

**Syntax Rules.**

- `del` on a name removes the binding; subsequent reference raises `NameError`.
- `del` on a container element removes that element.
- Multiple targets can be deleted in one statement: `del a, b, c`.
- `del` is a statement, not a function; `del(x)` deletes the name `x`, not the result of a function call.

**Constraints and Limitations.**

- Deleting a name that does not exist raises `NameError`.
- `del` cannot be used on immutable container elements (e.g., tuple items).
- Deleting a local variable inside a function affects only the local namespace.

### Annotated Code Examples

**Example 1: Deleting names**

```python
x = 10
y = 20
del x
# print(x)          # NameError: name 'x' is not defined
print(y)            # Expected output: 20
```

**Why the output is what it is.** `del x` removes the binding for `x` from the module namespace. The object that `x` referenced may still exist if other references remain, but the name `x` is no longer valid. `y` is unaffected.

**Example 2: Deleting list and dictionary elements**

```python
numbers = [10, 20, 30, 40]
del numbers[1]
print(numbers)       # Expected output: [10, 30, 40]

data = {"a": 1, "b": 2, "c": 3}
del data["b"]
print(data)          # Expected output: {'a': 1, 'c': 3}

del numbers[0:2]     # Delete a slice.
print(numbers)       # Expected output: []
```

**Why the output is what it is.** `del numbers[1]` removes the element at index 1 (value 20). `del data["b"]` removes the key–value pair for `"b"`. Slice deletion removes the specified range of elements. All three operations mutate the container in place.

### Real-World Cases

- **Releasing large objects.** `del large_data` after processing to free memory promptly (though reference counting usually handles this automatically).
- **Removing sensitive data.** `del password` after authentication to reduce the window of exposure.
- **Cleaning up temporary dictionaries.** `del temp_cache[key]` after a cached value is no longer needed.
- **Dynamic attribute removal.** `del obj.attr` in metaprogramming or cleanup code.

---

## 15. Memory Model

### Definitions

**Core Definition.** Python’s memory model describes how objects are created, referenced, and reclaimed in memory.

**Technical Definition.** CPython, the reference implementation, uses a private heap for object allocation and a **reference-counting** scheme as its primary memory-management mechanism, supplemented by a cycle detector for circular references.

**Beginner-Friendly Explanation.** Python keeps a mental note of how many names (and other references) point to each box. When the count drops to zero, the box can be thrown away. For boxes that point to each other in a cycle, Python has a backup cleaner.

### Purposes

- To allocate memory for new objects automatically.
- To reclaim memory when objects are no longer reachable.
- To provide predictable object lifetimes in most cases.
- To abstract away manual `malloc`/`free` management.

### Key Concepts

- **Object identity.** Every object has a unique identity, accessed via `id()`; in CPython this is the object’s memory address.
- **Reference count.** Each object tracks how many references point to it. When the count reaches zero, the object is deallocated.
- **Garbage collection.** CPython’s `gc` module detects and collects reference cycles that reference counting alone cannot reclaim.
- **Interning.** Small integers and some strings are interned (reused) for performance, meaning identical literals may share the same object.
- **Immortal objects.** PEP 683 introduces immortal objects with a fixed refcount, which are never deallocated; these are used for singletons like `None`, `True`, and `False`.

### Annotated Code Examples

**Example 1: Reference counting in action**

```python
import sys

a = [1, 2, 3]
print(sys.getrefcount(a))   # Expected output: 2 (a + the argument to getrefcount)

b = a
print(sys.getrefcount(a))   # Expected output: 3 (a, b, and the argument)

del b
print(sys.getrefcount(a))   # Expected output: 2 (a and the argument)
```

**Why the output is what it is.** `sys.getrefcount` returns the reference count of its argument, including the temporary reference created by the function call itself. When `b = a` adds a second reference, the count increases. Deleting `b` decrements the count. When the count reaches zero (not shown here because `a` still exists), the list is deallocated.

**Example 2: Reference cycle and garbage collection**

```python
import gc

class Node:
    def __init__(self):
        self.ref = None

# Create a reference cycle.
a = Node()
b = Node()
a.ref = b
b.ref = a
del a, b          # The cycle is now unreachable but refcounts are non-zero.
gc.collect()      # The cycle collector reclaims the memory.
print("Cycle collected")
```

**Why the output is what it is.** After `del a, b`, the two `Node` objects reference each other, so their reference counts remain positive even though no external references exist. The `gc.collect()` call runs the cycle detector, which identifies and collects the unreachable cycle.

### Real-World Cases

- **Long-running processes.** Understanding reference counting helps diagnose memory leaks caused by lingering references.
- **Performance optimisation.** Avoiding unnecessary object creation reduces allocation overhead.
- **Resource management.** Ensuring files and sockets are closed explicitly rather than relying on garbage collection.
- **Debugging.** Using `gc` and `sys.getrefcount` to investigate unexpected memory growth.

---

## 16. Reference Counting

### Definitions

**Core Definition.** Reference counting is a memory-management technique in which each object maintains a count of how many references point to it; the object is deallocated when the count reaches zero.

**Technical Definition.** In CPython, every `PyObject` includes an `ob_refcnt` field. Operations that store a reference increment the count; operations that remove a reference decrement it. When `ob_refcnt` reaches zero, the object’s deallocator is called.

**Beginner-Friendly Explanation.** Imagine each box has a tally mark for every person holding a string attached to it. When the last person lets go, the box is thrown away.

### Purposes

- To reclaim memory promptly when objects become unreachable.
- To avoid the overhead of tracing garbage collection in the common case.
- To provide deterministic finalisation for most objects.
- To integrate naturally with Python’s object model.

### Constraints and Limitations.

- Reference counting cannot reclaim reference cycles on its own; a cycle detector is required.
- Incrementing and decrementing refcounts adds a small overhead to every reference operation.
- Objects with `__del__` methods may complicate cycle collection.
- CPython-specific; other Python implementations (PyPy, Jython) may use different strategies.

### Annotated Code Examples

**Example 1: Observing reference count changes**

```python
import sys

obj = object()
print(sys.getrefcount(obj))   # Expected output: 2 (obj + argument)

ref1 = obj
ref2 = obj
print(sys.getrefcount(obj))   # Expected output: 4 (obj, ref1, ref2, argument)

del ref1, ref2
print(sys.getrefcount(obj))   # Expected output: 2 (obj + argument)
```

**Why the output is what it is.** `sys.getrefcount` includes the temporary reference created by the function call. Each additional name bound to the object increases the count by one. Deleting those names decreases the count.

### Real-World Cases

- **Memory profiling.** `sys.getrefcount` and `gc.get_referrers` help identify unexpected references.
- **C extensions.** The Python/C API relies on explicit refcount management (`Py_INCREF`/`Py_DECREF`).
- **Weak references.** The `weakref` module allows referencing objects without increasing their refcount, useful for caches.

---

## 17. Object Identity

### Definitions

**Core Definition.** Object identity is the unique, immutable identifier that distinguishes one Python object from every other object for its entire lifetime.

**Technical Definition.** Every object has an identity, accessible via `id()`. In CPython, the identity is the object’s memory address. The `is` operator compares identities.

**Beginner-Friendly Explanation.** Identity is like a national ID number: two people may have the same name and birthdate (equal values), but their ID numbers are different. `is` checks the ID number; `==` checks the name and birthdate.

### Purposes

- To determine whether two names refer to the exact same object.
- To implement identity-based caching and interning.
- To distinguish mutable objects that happen to have equal values.
- To support the `is` operator and `id()` function semantics.

### Annotated Code Examples

**Example 1: Identity of mutable objects**

```python
a = [1, 2, 3]
b = [1, 2, 3]
c = a

print(a == b)   # Expected output: True  (equal values)
print(a is b)   # Expected output: False (different objects)
print(a is c)   # Expected output: True  (same object)
```

**Why the output is what it is.** `a` and `b` are two distinct list objects with the same contents, so they are equal but not identical. `c = a` binds `c` to the same object as `a`, so `a is c` is `True`.

**Example 2: Identity of interned integers**

```python
x = 256
y = 256
print(x is y)   # Expected output: True (small integers are interned)

a = 1000
b = 1000
print(a is b)   # Expected output: False (large integers are not interned)
```

**Why the output is what it is.** CPython interns integers in the range −5 to 256 for performance, so `256 is 256` is `True`. Integers outside that range are not guaranteed to be interned, so `1000 is 1000` is typically `False` in CPython.

### Real-World Cases

- **Sentinel values.** `if value is None:` is the idiomatic way to check for `None`.
- **Identity-based caching.** `functools.lru_cache` uses identity for hashable keys.
- **Debugging.** `id()` helps determine whether two variables reference the same underlying object.

---

## 18. `id()` Function

### Definitions

**Core Definition.** The `id()` built-in returns the identity of an object as an integer.

**Technical Definition.** `id(object)` returns an integer that is guaranteed to be unique and constant for the object during its lifetime. In CPython, this is the memory address of the object.

**Beginner-Friendly Explanation.** `id()` gives you the “serial number” of an object.

### Purposes

- To obtain a unique identifier for an object.
- To compare object identity programmatically.
- To debug reference and aliasing issues.
- To implement low-level object tracking.

### Syntax Rules and Structure

**Complete General Syntax.**

```python
id(object)
```

**Component Breakdown.**

| Component | Meaning |
|---|---|
| `object` | Any Python object. |

**Constraints and Limitations.**

- The return value is an implementation detail; it may differ across Python implementations and runs.
- Two objects with non-overlapping lifetimes may share the same `id()` value.
- `id()` should not be used as a persistent key across program runs.

### Annotated Code Examples

**Example 1: Comparing identities**

```python
a = [1, 2]
b = [1, 2]
c = a

print(id(a) == id(b))   # Expected output: False
print(id(a) == id(c))   # Expected output: True
```

**Why the output is what it is.** `a` and `b` are separate list objects, so their IDs differ. `c` is bound to the same object as `a`, so their IDs are equal.

**Example 2: `id()` of interned small integers**

```python
print(id(10) == id(10))     # Expected output: True
print(id(1000) == id(1000)) # Expected output: May be True or False (not guaranteed)
```

**Why the output is what it is.** Small integers are interned, so the same object is reused, yielding the same `id`. Large integers may or may not be interned depending on the context; the result is implementation-dependent.

### Real-World Cases

- **Debugging aliasing.** Checking whether two variables point to the same list before mutating.
- **Weak references.** The `weakref` module uses identity to track objects.
- **Custom containers.** Implementing `__hash__` based on identity for objects that should be distinguished by reference.

---

## 19. `is` Operator

### Definitions

**Core Definition.** The `is` operator tests whether two operands refer to the same object in memory.

**Technical Definition.** `a is b` evaluates to `True` if and only if `id(a) == id(b)`. It cannot be overloaded by user-defined classes.

**Beginner-Friendly Explanation.** `is` asks, “Are these two names pointing to the exact same box?”

### Purposes

- To check for `None`, `True`, and `False` reliably.
- To verify that two names reference the same mutable object.
- To implement identity-based logic in caching and memoisation.
- To distinguish between equal-but-distinct objects.

### Syntax Rules and Structure

**Complete General Syntax.**

```python
left is right
left is not right
```

**Component Breakdown.**

| Component | Meaning |
|---|---|
| `left`, `right` | Any expressions yielding objects. |
| `is` | Identity comparison operator. |
| `is not` | Negated identity comparison. |

**Constraints and Limitations.**

- `is` should not be used for value comparison of numbers, strings, or other types where equality is intended.
- Small integers and interned strings may make `is` return `True` for equal values by accident, leading to fragile code.
- CPython may emit a `SyntaxWarning` for `is` comparisons with literals in future versions.

### Annotated Code Examples

**Example 1: Correct use with `None`**

```python
result = None
if result is None:
    print("No result")    # Expected output: No result
```

**Why the output is what it is.** `None` is a singleton; there is only one `None` object in a Python process. `is` is the correct operator to check for it, and it is both faster and more semantically precise than `== None`.

**Example 2: Incorrect use for value comparison**

```python
a = 1000
b = 1000
print(a is b)    # Expected output: False (in most CPython contexts)
print(a == b)    # Expected output: True
```

**Why the output is what it is.** `a` and `b` are equal in value but are not the same object (large integers are not interned). Using `is` here produces an unexpected `False` if the programmer intended value equality. The `==` operator is the correct choice.

### Real-World Cases

- **Singleton checks.** `if x is None`, `if flag is True`.
- **Identity-based dispatch.** Checking whether an argument is a specific sentinel object.
- **Testing object aliasing.** `if a is b:` to determine whether two variables share a reference.

---

## 20. `==` Operator

### Definitions

**Core Definition.** The `==` operator tests whether two objects have equal values.

**Technical Definition.** `a == b` calls `a.__eq__(b)` (or the reflected `b.__eq__(a)`). User-defined classes can override `__eq__` to define custom equality semantics.

**Beginner-Friendly Explanation.** `==` asks, “Do these two boxes contain the same thing?”

### Purposes

- To compare values of numbers, strings, lists, and other types.
- To implement custom equality in user-defined classes.
- To check membership and uniqueness based on value.
- To support sorting and hashing based on value equality.

### Syntax Rules and Structure

**Complete General Syntax.**

```python
left == right
left != right
```

**Component Breakdown.**

| Component | Meaning |
|---|---|
| `left`, `right` | Any expressions yielding objects. |
| `==` | Equality operator. |
| `!=` | Inequality operator. |

**Constraints and Limitations.**

- `==` can raise exceptions if `__eq__` is not implemented consistently.
- Objects that define `__eq__` should also define `__hash__` to be hashable.
- `==` is not guaranteed to be reflexive for floating-point `NaN`: `float('nan') == float('nan')` is `False`.

### Annotated Code Examples

**Example 1: Value equality for lists**

```python
a = [1, 2, 3]
b = [1, 2, 3]
print(a == b)    # Expected output: True
print(a != b)    # Expected output: False
```

**Why the output is what it is.** Lists compare element by element. Since both lists contain the same elements in the same order, they are equal.

**Example 2: Custom equality**

```python
class Point:
    def __init__(self, x, y):
        self.x = x
        self.y = y
    def __eq__(self, other):
        return isinstance(other, Point) and self.x == other.x and self.y == other.y

p1 = Point(1, 2)
p2 = Point(1, 2)
print(p1 == p2)   # Expected output: True
print(p1 is p2)   # Expected output: False
```

**Why the output is what it is.** The custom `__eq__` method defines equality based on coordinates. `p1` and `p2` have the same coordinates, so `==` returns `True`. They are distinct objects, so `is` returns `False`.

### Real-World Cases

- **Data validation.** Checking whether input matches an expected value.
- **Unit testing.** `assert result == expected` is the standard assertion pattern.
- **Deduplication.** Removing duplicates from a list based on value equality.
- **Configuration comparison.** Checking whether two configuration dictionaries are equal.

---

## 21. `is` vs `==`

### Definitions

**Core Definition.** `is` compares object identity; `==` compares object value.

**Technical Definition.** `is` is implemented as a pointer comparison (`id(a) == id(b)`); `==` invokes the `__eq__` protocol, which may be customised by the object’s type.

**Beginner-Friendly Explanation.** `is` asks, “Is this the same box?” `==` asks, “Do these boxes contain the same thing?”

### Purposes

- To use the correct operator for the intended comparison.
- To avoid subtle bugs caused by confusing identity with equality.
- To write idiomatic Python that is clear to other developers.
- To ensure correct behaviour with singletons and value objects.

### Key Differences

| Aspect | `is` | `==` |
|---|---|---|
| Compares | Identity (memory reference) | Value (contents) |
| Overloadable | No | Yes (`__eq__`) |
| Typical use | `None`, `True`, `False`, sentinels | Numbers, strings, lists, custom objects |
| Performance | Faster (pointer comparison) | May be slower (method call) |
| Example | `x is None` | `x == 0` |

### Annotated Code Examples

**Example 1: When `is` and `==` disagree**

```python
a = [1, 2, 3]
b = [1, 2, 3]
print(a is b)   # Expected output: False
print(a == b)   # Expected output: True
```

**Why the output is what it is.** `a` and `b` are distinct list objects, so `is` returns `False`. Their contents are identical, so `==` returns `True`.

**Example 2: When `is` and `==` agree (interning)**

```python
x = 100
y = 100
print(x is y)   # Expected output: True (interned)
print(x == y)   # Expected output: True
```

**Why the output is what it is.** CPython interns small integers, so `x` and `y` reference the same object. Consequently, both `is` and `==` return `True`. This coincidence should not be relied upon for value comparison.

### Real-World Cases

- **Singleton pattern.** Always use `is` for `None`, `True`, `False`.
- **Value objects.** Use `==` for dataclasses, named tuples, and custom value types.
- **Code review.** Flagging `is` comparisons with literals (e.g., `x is 5`) as potential bugs.
- **Performance-critical code.** Using `is` for fast identity checks in hot loops where identity semantics are correct.

---

## References

- Python Software Foundation. “3. Data Model.” *Python 3.13 Reference*. https://docs.python.org/3/reference/datamodel.html 
- Python Software Foundation. “Built-in Functions: id().” *Python 3 Documentation*. https://docs.python.org/3/library/functions.html#id
- Python Software Foundation. “PEP 203 – Augmented Assignments.” *Python Enhancement Proposals*. https://peps.python.org/pep-0203/ 
- Python Software Foundation. “PEP 3104 – Access to Names in Outer Scopes.” *Python Enhancement Proposals*. https://peps.python.org/pep-3104/
- Python Software Foundation. “PEP 8 – Style Guide for Python Code.” *Python Enhancement Proposals*. https://peps.python.org/pep-0008/
- Python Software Foundation. “PEP 683 – Immortal Objects, Using a Fixed Refcount.” *Python Enhancement Proposals*. https://peps.python.org/pep-0683/ 
- Real Python. “The LEGB Rule & Understanding Python Scope.” https://realpython.com/courses/legb-rule-python-scope/ 
- Real Python. “Variables: Python Best Practices.” https://realpython.com/ref/best-practices/variables/ 
- Real Python. “Constants: Python Best Practices.” https://realpython.com/ref/best-practices/constants/
- Real Python. “nonlocal – Python Keywords.” https://realpython.com/ref/keywords/nonlocal/ 
- Real Python. “Python != Is Not is not: Comparing Objects in Python.” https://realpython.com/python-is-identity-vs-equality/ 
- Microsoft Learn. “Understand Variables and Dynamic Typing.” https://learn.microsoft.com/en-us/training/modules/work-with-data-python/2-understand-variables-dynamic-typing 
- New Mexico Tech Computer Center. “The del Statement; The global Statement.” *Python 2.7 Quick Reference*. https://www.cs.nmt.edu/~jeffery/Shipman/www/docs/tcc/help/pubs/python27/python27.pdf 
- Python Software Foundation. “Extending Python with C or C++: Reference Counting.” *Python 3 C API Documentation*. https://docs.python.org/3/extending/extending.html 
- Python Software Foundation. “gc — Garbage Collector Interface.” *Python 3 Library Reference*. https://docs.python.org/3/library/gc.html