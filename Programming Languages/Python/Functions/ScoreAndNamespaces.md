# Scope and Namespaces: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
Scope is the region of a program where a name is visible and accessible. A namespace is the mapping from names to objects that exists within a scope.

**Technical Definition**
In Python, a **namespace** is a dictionary that maps names (identifiers) to objects. A **scope** is the textual region of a program where a namespace is directly accessible. Python resolves unqualified name references using the **LEGB rule**: Local, Enclosing, Global, Built-in. Each function call creates a new local namespace; module-level code executes in the global namespace; and the built-in namespace contains Python's predefined names. The `global` and `nonlocal` statements allow a function to rebind names in the global and enclosing namespaces, respectively .

**Beginner-Friendly Explanation**
Think of scopes as rooms in a building. When you call a function, you enter a new room (local scope). If you need something that's not in the room, you look in the hallway outside (enclosing scope), then the building's lobby (global scope), and finally the city's public library (built-in scope). You can always *read* things from outer rooms, but to *change* something in the lobby, you need special permission (`global`), and to change something in the hallway, you need a different permission (`nonlocal`).

### Key Characteristics

| Characteristic | Description |
|---|---|
| **Lexical (Static) Scoping** | Scope is determined by the textual position of code, not by the call stack  |
| **LEGB Lookup** | Names are resolved in order: Local → Enclosing → Global → Built-in  |
| **Assignment Creates Locals** | Any assignment to a name inside a function makes that name local to the function  |
| **Namespaces are Dynamic** | Local namespaces are created when a function is called and destroyed when it returns |
| **Explicit Rebinding** | `global` and `nonlocal` are required to rebind names in outer scopes |

### Prerequisites

- **Basic function definitions**: Understanding `def`, parameters, and calls
- **The concept of variables as name bindings**
- **Basic understanding of modules**: Python files execute in their own namespace

### Related Programming Areas

- **Closures**: Nested functions that capture variables from enclosing scopes
- **Decorators**: Often rely on closures and scope rules
- **Class definitions**: Class bodies have their own namespace rules (not part of LEGB for methods) 
- **Comprehensions**: Have their own local scope (in Python 3)

### Core Concepts / Features

1. **Local Scope** (function-level names)
2. **Global Scope** (module-level names)
3. **Enclosing Scope** (nested function scopes)
4. **Built-in Scope** (Python's predefined names)
5. **LEGB Lookup Rule** (the resolution order)
6. **`global` Keyword** (rebinding module-level names)
7. **`nonlocal` Keyword** (rebinding enclosing function names)
8. **Namespace Lifecycles** (creation, modification, destruction)
9. **Variable Shadowing Risks and Best Practices**

---

## Core Concept 1: Local Scope

### Definitions

**Core Definition**
Local scope is the namespace created when a function is called, containing names that are local to that function.

**Technical Definition**
When a function is called, Python creates a new local namespace (a dictionary). Any name assigned within the function body—including parameters, loop variables, and `import` statements—is bound in this local namespace. The local namespace is destroyed when the function returns, unless it is captured by a closure .

**Beginner-Friendly Explanation**
Local scope is the function's private workspace. Variables you create inside a function belong to that function alone. When the function finishes, those variables disappear. Other functions cannot see them unless you pass them out.

### Purposes

- **To isolate function state** from the rest of the program
- **To prevent naming conflicts** between different functions
- **To enable recursion** by giving each call its own set of local variables
- **To support closures** by allowing nested functions to capture local variables

### Syntax Rules and Structure

```python
def my_function():
    local_var = 10  # local_var is in the local namespace
    print(local_var)

my_function()
# print(local_var)  # NameError: local_var is not defined outside
```

**Syntax Rules**

| Rule | Description |
|---|---|
| Assignment creates local | Any assignment to a name inside a function makes it local  |
| Parameters are local | Function parameters are always in the local namespace |
| `import` binds locally | Importing inside a function creates a local name |
| Entire function is scanned | Locality is determined by scanning the entire function for assignments  |

**Constraints and Limitations**

- **`UnboundLocalError`**: If a name is assigned somewhere in the function, all references to it are treated as local, even before the assignment 
- **No block scope**: Unlike C or Java, Python does not have block-level scope (e.g., `if` blocks do not create new scopes) 

### Multiple Annotated Complete Code Examples and Expected Outputs

**Example 1: Basic Local Scope**

```python
def calculate():
    result = 42       # 'result' is local
    print(result)

calculate()
# Output: 42

# print(result)  # NameError: name 'result' is not defined
```

**Expected Output:**
```
42
```

**Breakdown:** `result` is created in the local namespace of `calculate`. After the function returns, the name is gone.

**Example 2: `UnboundLocalError` Trap**

```python
count = 0

def increment():
    count = count + 1  # Error: count is local because of assignment
    print(count)

# increment()  # UnboundLocalError: local variable 'count' referenced before assignment
```

**Expected Output (if run):**
```
UnboundLocalError: local variable 'count' referenced before assignment
```

**Breakdown:** Because `count = ...` appears in the function, Python treats `count` as local throughout. The right-hand side `count + 1` tries to read a local `count` that has not yet been assigned .

### Real-World Cases with Explanation

**Case 1: Recursive Functions**

```python
def factorial(n):
    if n <= 1:
        return 1
    return n * factorial(n - 1)
```

**Why it matters:** Each recursive call gets its own local `n`, allowing the function to track state across calls.

---

## Core Concept 2: Global Scope

### Definitions

**Core Definition**
Global scope is the namespace of the module (file) where the code is executing. Names defined at the top level of a module are global.

**Technical Definition**
When a Python module is executed, its top-level code runs in the module's global namespace (a dictionary accessible via `globals()`). Functions and classes defined at the module level are bound in this namespace. Functions can *read* global names without declaration, but must use the `global` statement to *rebind* them .

**Beginner-Friendly Explanation**
Global scope is the module's shared workspace. Everything you define at the top level of your file (functions, classes, constants) lives here. Functions can look up these names, but if they want to change them, they need to say `global`.

### Purposes

- **To share data across functions** within a module
- **To define module-level constants** and configuration
- **To provide names for imported modules and functions**

### Syntax Rules and Structure

```python
# Global scope
GLOBAL_VAR = 100

def read_global():
    print(GLOBAL_VAR)  # Reading is allowed

def write_global():
    global GLOBAL_VAR
    GLOBAL_VAR = 200   # Rebinding requires 'global'

read_global()   # Output: 100
write_global()
read_global()   # Output: 200
```

**Syntax Rules**

| Rule | Description |
|---|---|
| Top-level names are global | Module-level assignments create global names |
| Reading is automatic | Functions can read globals without declaration |
| Rebinding requires `global` | Assignment to a global inside a function needs `global`  |
| `global` must precede use | The `global` statement must appear before any use of the name in the function  |

### Multiple Annotated Complete Code Examples and Expected Outputs

**Example 1: Reading Globals**

```python
CONFIG = {"debug": True}

def show_config():
    print(CONFIG)  # Reading global is fine

show_config()
```

**Expected Output:**
```
{'debug': True}
```

**Breakdown:** `CONFIG` is found via the global namespace. No declaration needed for reading.

**Example 2: Rebinding Without `global` (Bug)**

```python
counter = 0

def bad_increment():
    counter = 5  # Creates a LOCAL counter, does not modify global
    print(f"Inside: {counter}")

bad_increment()
print(f"Outside: {counter}")
```

**Expected Output:**
```
Inside: 5
Outside: 0
```

**Breakdown:** The assignment creates a local `counter`. The global remains unchanged. This is a common source of confusion .

### Real-World Cases with Explanation

**Case 1: Module Configuration**

```python
# config.py
DEBUG = False
DATABASE_URL = "sqlite:///app.db"
```

**Why it matters:** Global constants provide a single source of truth for configuration across a module.

---

## Core Concept 3: Enclosing Scope

### Definitions

**Core Definition**
Enclosing scope refers to the local scope of an outer function relative to a nested inner function.

**Technical Definition**
When a function is defined inside another function, the inner function can access names from the outer function's local namespace. These names are called "free variables" and are captured by the inner function as part of a closure. Python 3 introduced the `nonlocal` keyword to allow rebinding such names from the inner function .

**Beginner-Friendly Explanation**
If you have a function inside another function, the inner function can see the outer function's variables. This is called a closure. The inner function can read those variables directly. To *change* them, it needs the `nonlocal` keyword.

### Purposes

- **To create closures** that capture state from an enclosing function
- **To implement decorators** that remember configuration
- **To build factory functions** that produce specialized functions

### Syntax Rules and Structure

```python
def outer():
    x = 10

    def inner():
        print(x)  # Reads enclosing x

    inner()

outer()  # Output: 10
```

**With `nonlocal`:**

```python
def outer():
    x = 10

    def inner():
        nonlocal x
        x = 20  # Modifies enclosing x

    inner()
    print(x)

outer()  # Output: 20
```

**Syntax Rules**

| Rule | Description |
|---|---|
| Only function scopes | Enclosing scope applies only to functions nested inside functions  |
| Classes are not enclosing | A class body does not serve as an enclosing scope for its methods  |
| `nonlocal` requires binding | The name must exist in an enclosing function scope at compile time  |

**Constraints and Limitations**

- **No class scope in LEGB**: Class attributes are not accessible as unqualified names in methods 
- **`nonlocal` cannot reach global**: `nonlocal` skips the global scope; use `global` for module-level names 

### Multiple Annotated Complete Code Examples and Expected Outputs

**Example 1: Closure Reading Enclosing Variable**

```python
def make_multiplier(factor):
    def multiply(x):
        return x * factor  # 'factor' is a free variable
    return multiply

double = make_multiplier(2)
print(double(5))
```

**Expected Output:**
```
10
```

**Breakdown:** `multiply` captures `factor` from `make_multiplier`'s scope. This is a closure .

**Example 2: Using `nonlocal`**

```python
def counter():
    count = 0

    def increment():
        nonlocal count
        count += 1
        return count

    return increment

c = counter()
print(c())  # 1
print(c())  # 2
```

**Expected Output:**
```
1
2
```

**Breakdown:** `nonlocal count` allows `increment` to modify the `count` variable in `counter`'s scope, preserving state across calls.

### Real-World Cases with Explanation

**Case 1: Decorators with State**

```python
def retry(max_attempts):
    def decorator(func):
        attempts = 0
        def wrapper(*args, **kwargs):
            nonlocal attempts
            attempts += 1
            print(f"Attempt {attempts}")
            return func(*args, **kwargs)
        return wrapper
    return decorator
```

**Why it matters:** Closures enable decorators to maintain state (like attempt counts) across function calls.

---

## Core Concept 4: Built-in Scope

### Definitions

**Core Definition**
Built-in scope is the namespace containing Python's predefined names, such as `print`, `len`, `list`, and `Exception`.

**Technical Definition**
The built-in namespace is implemented as the `builtins` module (formerly `__builtin__` in Python 2). It contains all of Python's built-in functions, exceptions, and types. This namespace is always available and is the last place Python looks when resolving a name .

**Beginner-Friendly Explanation**
Built-in scope is Python's "standard library of names" that are always available. Things like `print()`, `len()`, and `range()` live here. You don't need to import them.

### Purposes

- **To provide universally available functions and types** without imports
- **To serve as the final fallback** in name resolution

### Syntax Rules and Structure

Built-in names are accessed automatically:

```python
print(len([1, 2, 3]))  # len and print are built-ins
```

**Constraints and Limitations**

- **Do not shadow built-ins**: Assigning to names like `list` or `str` can break code that expects the built-in 
- **Do not modify `__builtins__`**: Adding to the built-in namespace is strongly discouraged 

### Multiple Annotated Complete Code Examples and Expected Outputs

**Example 1: Built-in Lookup**

```python
def my_func():
    return len("hello")  # 'len' is found in built-in scope

print(my_func())
```

**Expected Output:**
```
5
```

**Breakdown:** `len` is not local or global, so Python finds it in the built-in namespace.

**Example 2: Shadowing a Built-in (Bad Practice)**

```python
list = [1, 2, 3]  # Shadows the built-in 'list'

# Now 'list' refers to the local variable, not the built-in type
# print(list("abc"))  # TypeError: 'list' object is not callable

del list  # Remove the shadow
print(list("abc"))
```

**Expected Output:**
```
['a', 'b', 'c']
```

**Breakdown:** Shadowing `list` makes the built-in type inaccessible by that name .

### Real-World Cases with Explanation

**Case 1: Avoiding Built-in Shadowing**

```python
# BAD
str = "hello"
# ... later ...
# result = str(123)  # TypeError: 'str' object is not callable

# GOOD
message = "hello"
result = str(123)  # Built-in still works
```

**Why it matters:** Shadowing built-ins leads to confusing errors and code that is hard to debug .

---

## Core Concept 5: LEGB Lookup Rule

### Definitions

**Core Definition**
LEGB is the order in which Python resolves unqualified name references: Local, Enclosing, Global, Built-in.

**Technical Definition**
When a name is used in a code block, Python searches the nearest enclosing scopes in order: first the local namespace, then any enclosing function namespaces (from innermost to outermost), then the global namespace, and finally the built-in namespace. If the name is not found, `NameError` is raised .

**Beginner-Friendly Explanation**
When Python sees a name, it looks for it in this order: (1) the current function, (2) any outer functions, (3) the module, (4) Python's built-ins. The first match wins.

### Purposes

- **To provide a predictable name resolution order**
- **To enable closures and nested functions**
- **To allow local customization** without affecting outer scopes

### Syntax Rules and Structure

The lookup follows this priority:

1. **L**ocal: Names assigned in the current function
2. **E**nclosing: Names in outer function scopes (for nested functions)
3. **G**lobal: Names at the module level
4. **B**uilt-in: Python's predefined names

**Example:**

```python
x = "global"

def outer():
    x = "enclosing"

    def inner():
        x = "local"
        print(x)  # Finds local first

    inner()

outer()
```

**Expected Output:**
```
local
```

### Multiple Annotated Complete Code Examples and Expected Outputs

**Example 1: Full LEGB Demonstration**

```python
# Built-in: len is in built-in scope

def outer():
    # Enclosing scope for inner
    message = "enclosing"

    def inner():
        # Local scope
        message = "local"
        print(message)  # Local wins

    inner()
    print(message)      # Enclosing message

outer()
print(len([1,2]))      # Built-in
```

**Expected Output:**
```
local
enclosing
2
```

**Breakdown:** Each scope provides its own `message`, and the lookup respects the nesting order.

**Example 2: Falling Through LEGB**

```python
def outer():
    def inner():
        print(len("hello"))  # No local/enclosing 'len', finds built-in
    inner()

outer()
```

**Expected Output:**
```
5
```

**Breakdown:** `len` is not local or enclosing, so Python searches global then built-in.

### Real-World Cases with Explanation

**Case 1: Debugging Name Resolution**

```python
value = 10

def process():
    # print(value)  # Works: finds global
    value = 20      # Makes 'value' local
    print(value)    # 20

process()
```

**Why it matters:** Understanding LEGB explains why adding an assignment changes a name from global to local.

---

## Core Concept 6: `global` Keyword

### Definitions

**Core Definition**
The `global` statement declares that specified names refer to bindings in the global (module) namespace, allowing a function to rebind them.

**Technical Definition**
When `global name` appears in a function, all assignments to `name` within that function modify the module-level binding. The `global` statement must appear before any use of the name in the function. It creates a new global binding if one does not exist .

**Beginner-Friendly Explanation**
`global` is like a permission slip that says "I want to change the module-level variable, not create a new local one."

### Purposes

- **To modify module-level state** from within a function
- **To implement module-level counters or caches**
- **To share mutable configuration** across functions

### Syntax Rules and Structure

```python
count = 0

def increment():
    global count
    count += 1

increment()
print(count)  # 1
```

**Syntax Rules**

| Rule | Description |
|---|---|
| Must precede use | `global` must appear before any reference to the name  |
| Creates binding if absent | `global x` followed by `x = 1` creates a new global |
| No `nonlocal` mixing | A name cannot be both `global` and `nonlocal` |

### Multiple Annotated Complete Code Examples and Expected Outputs

**Example 1: Modifying a Global Counter**

```python
total = 0

def add_to_total(amount):
    global total
    total += amount

add_to_total(10)
add_to_total(20)
print(total)
```

**Expected Output:**
```
30
```

**Breakdown:** Without `global`, `total += amount` would raise `UnboundLocalError` .

**Example 2: Creating a New Global**

```python
def setup():
    global config
    config = {"debug": True}

setup()
print(config)
```

**Expected Output:**
```
{'debug': True}
```

**Breakdown:** `global config` allows `setup` to create a module-level `config`.

### Real-World Cases with Explanation

**Case 1: Module-Level Cache**

```python
_cache = {}

def get_data(key):
    if key in _cache:
        return _cache[key]
    # ... fetch data ...
    _cache[key] = result
    return result
```

**Why it matters:** `global` enables functions to maintain module-level caches without passing the cache around.

---

## Core Concept 7: `nonlocal` Keyword

### Definitions

**Core Definition**
The `nonlocal` statement declares that a name refers to a binding in the nearest enclosing function scope (excluding global and built-in).

**Technical Definition**
Introduced in Python 3 (PEP 3104), `nonlocal name` allows an inner function to rebind a name in an enclosing function's scope. The name must already be bound in an enclosing function scope at compile time; otherwise, a `SyntaxError` is raised .

**Beginner-Friendly Explanation**
`nonlocal` is like `global`, but for the nearest outer function instead of the module. It lets an inner function change a variable in its enclosing function.

### Purposes

- **To implement mutable state in closures**
- **To build counters, accumulators, and stateful decorators**
- **To avoid mutable default argument workarounds**

### Syntax Rules and Structure

```python
def outer():
    count = 0

    def inner():
        nonlocal count
        count += 1
        return count

    return inner

counter = outer()
print(counter())  # 1
print(counter())  # 2
```

**Syntax Rules**

| Rule | Description |
|---|---|
| Must exist in enclosing scope | Compile-time error if no binding found  |
| Skips global | `nonlocal` does not reach the module namespace  |
| Only in nested functions | Requires an enclosing function scope |

### Multiple Annotated Complete Code Examples and Expected Outputs

**Example 1: Stateful Closure**

```python
def make_accumulator():
    total = 0

    def add(amount):
        nonlocal total
        total += amount
        return total

    return add

acc = make_accumulator()
print(acc(5))   # 5
print(acc(10))  # 15
```

**Expected Output:**
```
5
15
```

**Breakdown:** `nonlocal total` allows `add` to persist changes to `total` across calls .

**Example 2: `nonlocal` Error**

```python
def outer():
    def inner():
        nonlocal x  # SyntaxError: no binding for nonlocal 'x' found
        x = 1
    inner()
```

**Expected Output (at compile time):**
```
SyntaxError: no binding for nonlocal 'x' found
```

**Breakdown:** `nonlocal` requires the name to exist in an enclosing function scope .

### Real-World Cases with Explanation

**Case 1: Counting Function Calls**

```python
def count_calls(func):
    calls = 0
    def wrapper(*args, **kwargs):
        nonlocal calls
        calls += 1
        print(f"Called {calls} times")
        return func(*args, **kwargs)
    return wrapper
```

**Why it matters:** `nonlocal` enables decorators to track state without using global variables or mutable defaults.

---

## Core Concept 8: Namespace Lifecycles

### Definitions

**Core Definition**
Namespace lifecycle refers to when a namespace is created, how it is modified, and when it is destroyed.

**Technical Definition**
- **Local namespace**: Created when a function is called; destroyed when the function returns (unless captured by a closure) .
- **Global namespace**: Created when the module is loaded; persists until the program ends or the module is explicitly deleted.
- **Built-in namespace**: Created when the Python interpreter starts; persists for the interpreter's lifetime.
- **Enclosing namespace**: Created when an outer function is called; persists as long as the closure references it .

**Beginner-Friendly Explanation**
Local variables live only as long as the function is running. Global variables live as long as the program. Built-ins live as long as Python is running. Closures keep their enclosing variables alive even after the outer function returns.

### Multiple Annotated Complete Code Examples and Expected Outputs

**Example 1: Local Namespace Destruction**

```python
def create_local():
    temp = "I am local"
    return temp

result = create_local()
print(result)
# print(temp)  # NameError: temp is destroyed
```

**Expected Output:**
```
I am local
```

**Breakdown:** `temp` exists only during the call to `create_local`.

**Example 2: Closure Keeping Enclosing Namespace Alive**

```python
def make_counter():
    count = 0
    def increment():
        nonlocal count
        count += 1
        return count
    return increment

c = make_counter()
print(c())  # 1
print(c())  # 2
# 'count' persists even though make_counter has returned
```

**Expected Output:**
```
1
2
```

**Breakdown:** The closure captures `count`, keeping it alive beyond `make_counter`'s return .

---

## Core Concept 9: Variable Shadowing Risks and Best Practices

### Definitions

**Core Definition**
Shadowing occurs when a name in an inner scope hides a name with the same identifier in an outer scope.

**Technical Definition**
When a local assignment binds a name that already exists in an enclosing or global scope, the inner binding "shadows" the outer one for the duration of the inner scope. This is legal Python but often leads to bugs, especially when shadowing built-ins or global names.

**Beginner-Friendly Explanation**
Shadowing is like putting a new label on a box that already has a label. Inside the room, everyone uses the new label. But outside, the old label is still there. If you don't realize there are two, you can get confused.

### Purposes (of Avoiding Shadowing)

- **To prevent confusing bugs** where the wrong variable is accessed
- **To maintain readability** by keeping names meaningful and unique
- **To preserve access to built-in functions** and types
- **To avoid `UnboundLocalError`** caused by accidental local bindings

### Best Practices

| Practice | Why |
|---|---|
| Do not shadow built-ins (`list`, `str`, `dict`, `id`, etc.) | Breaks access to the built-in, leading to confusing errors  |
| Use descriptive, specific names | Reduces the chance of collision |
| Avoid `global` when possible | Global state makes code harder to reason about  |
| Use `nonlocal` sparingly | Prefer passing state explicitly or using classes |
| Be aware of the entire function | Locality is determined by scanning for any assignment  |
| Use `from module import *` cautiously | Can silently shadow existing names  |

### Multiple Annotated Complete Code Examples and Expected Outputs

**Example 1: Shadowing a Built-in (Bad)**

```python
def process_list(input):
    list = input.split(",")  # Shadows built-in 'list'
    return list

result = process_list("a,b,c")
print(result)

# This fails because 'list' is now a local variable, not the type
# new_list = list("abc")  # TypeError: 'list' object is not callable
```

**Expected Output:**
```
['a', 'b', 'c']
```

**Breakdown:** Inside `process_list`, `list` refers to the local variable. Outside, the built-in is still available, but the shadowing inside the function is a bad habit .

**Example 2: Shadowing a Global (Confusing)**

```python
value = 10

def process():
    value = 20  # Shadows global
    print(f"Inside: {value}")

process()
print(f"Outside: {value}")
```

**Expected Output:**
```
Inside: 20
Outside: 10
```

**Breakdown:** The local `value` shadows the global. This is legal but can confuse readers who expect the global to be modified.

### Real-World Cases with Explanation

**Case 1: Accidental Shadowing in Comprehensions**

```python
# Python 3 comprehensions have their own scope
x = 10
squares = [x * x for x in range(5)]  # 'x' here does not shadow outer x
print(x)  # Still 10
```

**Why it matters:** Python 3 fixed the Python 2 behavior where comprehension variables leaked into the enclosing scope.

---

## References

- Python.org Tutor Mailing List. *parameters vs arguments*. https://mail.python.org/archives/list/tutor@python.org/thread/5FVSVIIMJRBQ2SHMTIPDCHOLNVJAJYXP/ 
- Python.org Tutor Mailing List. *Re: [Tutor] python closures*. https://mail.python.org/archives/list/tutor@python.org/message/2HNYABXFIJ54MIFBT625AUFR2JRUMHOU/ 
- Python.org Python-ideas Mailing List. *Unify global and nonlocal*. https://mail.python.org/pipermail/python-ideas/2014-February/026223.html 
- Python.org Docs Mailing List. *possible bug*. https://mail.python.org/pipermail/docs/2016-April/026190.html 
- Python Software Foundation. *Execution model — Python 2.7 documentation*. https://docs.python.org/ko/2.7/_sources/reference/executionmodel.rst.txt 
- Python.org Python-list. *LEGB rule, totally confused*. https://mail.python.org/pipermail/python-list/2007-August/458499.html 
- Python.org Python-list. *Indirection is using a name to refer to an object*. https://mail.python.org/archives/list/pydotorg-www@python.org/message/5NL2BX7U4RYJAUTVJP5KVXPZGIT4TLDY/ 
- Python.org Python-list. *Problem understanding how closures work*. https://mail.python.org/pipermail/python-list/2006-December/400641.html 
- Python.org Python-list. *Python 2.6 Global Variables*. https://mail.python.org/pipermail/python-list/2009-October/706874.html 
- Marc.info. *[Python-ideas] Unify global and nonlocal*. https://marc.info/?l=python-ideas&m=139152751010856&w=1 
- Python.org Python-list. *shadowing built-in names*. https://mail.python.org/pipermail/python-list/2003-May/227194.html 
- Python.org Python-list. *why is this not working? (nested scope question)*. https://mail.python.org/pipermail/python-list/2006-July/409548.html 
- Open Standards. *ISO/IEC JTC1 SC22 WG23 N1058 Python Vulnerabilities*. https://www.open-std.org/JTC1/sc22/wg23/docs/ISO-IECJTC1-SC22-WG23_N1058-tr24772-4-python-vulnerabilities-20210317.docx 
- Python.org Python-list. *Why do closures do this?*. https://mail.python.org/pipermail/python-list/2011-August/761750.html 