# Function Parameters: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
Function parameters are the named placeholders in a function definition that specify what inputs the function accepts. Arguments are the actual values supplied when the function is called.

**Technical Definition**
Python's function parameter system defines how arguments are bound to parameters at call time. The binding order follows a strict grammar: positional-only parameters (marked by `/`) must come first, followed by positional-or-keyword parameters, then `*args`, then keyword-only parameters (marked by `*`), and finally `**kwargs`. Default values are evaluated once at function definition time and stored on the function object . Parameter kinds are formally classified by the `inspect` module as `POSITIONAL_ONLY`, `POSITIONAL_OR_KEYWORD`, `VAR_POSITIONAL`, `KEYWORD_ONLY`, and `VAR_KEYWORD` .

**Beginner-Friendly Explanation**
Think of a function as a form with blanks to fill in. The blanks are **parameters**. When you fill in the blanks, the values you write are **arguments**. Python gives you many ways to fill these blanks: by position (the first blank gets the first value), by name (you write the blank's name and value), or in bulk (you hand over a bag of values and Python distributes them).

### Key Characteristics

| Characteristic | Description |
|---|---|
| **Binding Flexibility** | Arguments can be passed positionally or by keyword, unless restricted by `/` or `*` |
| **Default Values** | Parameters can have defaults, making them optional; defaults are evaluated once at definition time |
| **Variadic Support** | `*args` collects extra positional arguments; `**kwargs` collects extra keyword arguments |
| **Explicit Restrictions** | `/` enforces positional-only; `*` enforces keyword-only |
| **Unpacking Operators** | `*` and `**` unpack iterables and mappings at the call site |

### Prerequisites

- **Basic function definitions**: Understanding `def`, function bodies, and calls
- **The distinction between parameters and arguments**
- **Basic data structures**: Lists, tuples, dictionaries

### Related Programming Areas

- **Function Signatures and Introspection**: `inspect.signature()` for programmatic analysis 
- **Decorators**: Often need to preserve or manipulate parameter signatures
- **API Design**: Choosing parameter kinds affects usability and backward compatibility
- **Type Hints**: Annotations work with all parameter kinds 

### Core Concepts / Features

1. **Positional Arguments** (values matched by order)
2. **Keyword Arguments** (values matched by parameter name)
3. **Default Parameters** (optional arguments and the mutable default trap)
4. **Variable-Length Arguments** (`*args`, `**kwargs`)
5. **Positional-Only Parameters** (`/`)
6. **Keyword-Only Parameters** (`*`)
7. **Parameter Unpacking** (`*` and `**` at call site)

---

## Core Concept 1: Positional Arguments

### Definitions

**Core Definition**
Positional arguments are values passed to a function in order, matched to parameters by their position in the function definition.

**Technical Definition**
When a function is called with positional arguments, Python binds each argument to the corresponding parameter based on position: the first argument binds to the first parameter, the second to the second, and so on. Parameters marked as `POSITIONAL_ONLY` (before a `/`) accept only positional arguments. `VAR_POSITIONAL` parameters (`*args`) collect any excess positional arguments into a tuple .

**Beginner-Friendly Explanation**
When you call `divide(10, 2)`, Python matches `10` to the first parameter and `2` to the second. Order matters: `divide(2, 10)` would give a different result. This is the most natural way to pass arguments.

### Purposes

- **To pass values in a predictable order** when the meaning of each argument is clear from the function signature
- **To avoid verbose keyword names** for simple, commonly understood parameters
- **To support positional-only parameters** in built-in functions and APIs where parameter names are implementation details

### Syntax Rules and Structure

**Complete General Syntax**

```python
def function(param1, param2, param3):
    ...

function(value1, value2, value3)
```

**Breakdown:**
- `param1`, `param2`, `param3`: Parameters in the definition.
- `value1`, `value2`, `value3`: Arguments in the call, matched by position.

**Syntax Rules**

| Rule | Description |
|---|---|
| Order matters | First argument binds to first parameter |
| Count must match | Unless defaults or `*args` are present |
| Positional-only | Parameters before `/` cannot be passed by keyword  |

**Constraints and Limitations**

- **Too many arguments**: Raises `TypeError` unless a `*args` parameter exists.
- **Too few arguments**: Raises `TypeError` unless defaults exist.
- **Positional-only restriction**: Parameters before `/` cannot be passed as keywords .

### Multiple Annotated Complete Code Examples and Expected Outputs

**Example 1: Basic Positional Arguments**

```python
def divide(dividend, divisor):
    """Divide dividend by divisor."""
    return dividend / divisor

# Positional: 10 -> dividend, 2 -> divisor
result = divide(10, 2)
print(result)
```

**Expected Output:**
```
5.0
```

**Breakdown:** `10` binds to `dividend` and `2` binds to `divisor` because of their positions.

**Example 2: Order Matters**

```python
def describe_pet(name, species):
    print(f"{name} is a {species}.")

# Correct order
describe_pet("Rex", "dog")
# Output: Rex is a dog.

# Swapped order (wrong)
describe_pet("dog", "Rex")
# Output: dog is a Rex.
```

**Expected Output:**
```
Rex is a dog.
dog is a Rex.
```

**Breakdown:** Swapping arguments produces incorrect output because the values are matched by position, not by meaning.

### Real-World Cases with Explanation

**Case 1: Mathematical Functions**

```python
def power(base, exponent):
    return base ** exponent

power(2, 3)  # 2³ = 8
```

**Why it matters:** Mathematical functions naturally use positional arguments because the order is intuitive and consistent.

---

## Core Concept 2: Keyword Arguments

### Definitions

**Core Definition**
Keyword arguments are values passed to a function by explicitly naming the parameter they correspond to.

**Technical Definition**
A keyword argument uses the syntax `parameter_name=value` in the call. Python binds the value to the named parameter, regardless of position. Parameters marked as `KEYWORD_ONLY` (after a `*`) must be passed by keyword . Keyword arguments must follow positional arguments in the call .

**Beginner-Friendly Explanation**
Instead of relying on order, you say exactly which parameter gets which value: `divide(divisor=2, dividend=10)`. This is clearer and prevents mistakes when a function has many parameters.

### Purposes

- **To make function calls self-documenting** by naming each argument
- **To pass arguments in any order** without worrying about position
- **To set only specific optional parameters** while using defaults for others
- **To enforce keyword-only parameters** for clarity in APIs

### Syntax Rules and Structure

**Complete General Syntax**

```python
function(param1=value1, param2=value2)
```

**Breakdown:**
- `param1=value1`: Binds `value1` to parameter `param1`.
- Keyword arguments can appear in any order.
- All keyword arguments must come after positional arguments.

**Syntax Rules**

| Rule | Description |
|---|---|
| Name must match parameter | Using an unknown name raises `TypeError` |
| Order is free | Keyword arguments can be in any order |
| After positional | `f(1, key=2)` is valid; `f(key=2, 1)` is a `SyntaxError`  |
| Keyword-only | Parameters after `*` must be passed by keyword  |

### Multiple Annotated Complete Code Examples and Expected Outputs

**Example 1: Basic Keyword Arguments**

```python
def create_user(username, email, role="user"):
    return {"username": username, "email": email, "role": role}

# Keyword arguments in any order
user = create_user(email="alice@example.com", username="alice")
print(user)
```

**Expected Output:**
```
{'username': 'alice', 'email': 'alice@example.com', 'role': 'user'}
```

**Breakdown:** The arguments are matched by name, so order is irrelevant. `role` uses its default.

**Example 2: Mixing Positional and Keyword**

```python
def send_email(to, subject, body, cc=None):
    print(f"To: {to}, Subject: {subject}, CC: {cc}")

# Positional first, then keyword
send_email("bob@example.com", "Hello", body="Hi Bob", cc="cc@example.com")
```

**Expected Output:**
```
To: bob@example.com, Subject: Hello, CC: cc@example.com
```

**Breakdown:** `to` and `subject` are positional; `body` and `cc` are keyword. Keyword arguments must follow positional ones.

### Real-World Cases with Explanation

**Case 1: Configuration Functions**

```python
def connect(host, port=5432, user="admin", password=None):
    ...

connect("localhost", user="alice", password="secret")
```

**Why it matters:** Keyword arguments make it clear which values correspond to which settings, especially when many parameters have defaults.

---

## Core Concept 3: Default Parameters

### Definitions

**Core Definition**
A default parameter is a parameter that has a pre-assigned value, used when the caller does not supply an argument for it.

**Technical Definition**
Default values are evaluated once, at function definition time, and stored on the function object in `__defaults__`. If the caller omits the argument, the stored default is used. This means mutable defaults (e.g., `[]`, `{}`) are shared across all calls that rely on the default .

**Beginner-Friendly Explanation**
You can give a parameter a "fallback" value. If the caller doesn't provide a value, Python uses the fallback. But be careful: if the fallback is a list or dictionary, Python creates it only once, so every call shares the same object—and if the function modifies it, those changes persist.

### Purposes

- **To make parameters optional** with sensible defaults
- **To simplify common calls** by allowing callers to omit parameters they don't need to customize
- **To support backward compatibility** when adding new parameters

### Syntax Rules and Structure

**Complete General Syntax**

```python
def function(param=default_value):
    ...
```

**Breakdown:**
- `param`: The parameter name.
- `default_value`: The value used if no argument is supplied.

**Syntax Rules**

| Rule | Description |
|---|---|
| Defaults after non-defaults | `def f(a, b=1)` is valid; `def f(a=1, b)` is a `SyntaxError` |
| Evaluated once | Defaults are computed at definition time  |
| Mutable defaults shared | The same object is reused across calls  |

**Constraints and Limitations**

- **Mutable default trap**: Using `[]` or `{}` as a default causes all calls to share the same mutable object .
- **Default evaluation timing**: Defaults are evaluated once, not per-call, which can surprise developers from languages like Ruby .

### Multiple Annotated Complete Code Examples and Expected Outputs

**Example 1: Basic Default Parameter**

```python
def greet(name, greeting="Hello"):
    print(f"{greeting}, {name}!")

greet("Alice")
greet("Bob", "Hi")
```

**Expected Output:**
```
Hello, Alice!
Hi, Bob!
```

**Breakdown:** `greeting` defaults to `"Hello"` when omitted. Providing a value overrides it.

**Example 2: The Mutable Default Trap**

```python
def append_to(element, to=[]):
    to.append(element)
    return to

print(append_to(12))
print(append_to(42))
```

**Expected Output:**
```
[12]
[12, 42]
```

**Breakdown:** The default `[]` is created once. The first call appends `12`, the second call appends `42` to the *same* list, producing `[12, 42]` .

**Example 3: Correct Pattern (Using `None`)**

```python
def append_to(element, to=None):
    if to is None:
        to = []
    to.append(element)
    return to

print(append_to(12))
print(append_to(42))
```

**Expected Output:**
```
[12]
[42]
```

**Breakdown:** `None` is immutable. A new list is created inside the function on each call, avoiding the shared-state trap .

### Real-World Cases with Explanation

**Case 1: Accumulator Functions**

```python
# WRONG: shared mutable default
def collect(item, result=[]):
    result.append(item)
    return result

# CORRECT: new list each call
def collect(item, result=None):
    if result is None:
        result = []
    result.append(item)
    return result
```

**Why it matters:** The correct pattern ensures each call gets a fresh list unless the caller explicitly provides one .

---

## Core Concept 4: Variable-Length Arguments (`*args`, `**kwargs`)

### Definitions

**Core Definition**
`*args` collects extra positional arguments into a tuple; `**kwargs` collects extra keyword arguments into a dictionary.

**Technical Definition**
A parameter prefixed with `*` (conventionally named `args`) is classified as `VAR_POSITIONAL` by `inspect` and receives a tuple of all unmatched positional arguments . A parameter prefixed with `**` (conventionally `kwargs`) is classified as `VAR_KEYWORD` and receives a dictionary of all unmatched keyword arguments . `*args` must appear before `**kwargs` in the definition .

**Beginner-Friendly Explanation**
Sometimes you don't know how many arguments you'll receive. `*args` says "put all extra positional values in a tuple." `**kwargs` says "put all extra keyword values in a dictionary." This is useful for wrappers and flexible APIs.

### Purposes

- **To accept an arbitrary number of positional arguments**
- **To accept an arbitrary number of keyword arguments**
- **To write wrapper functions and decorators** that pass through unknown arguments
- **To create flexible APIs** that forward arguments to other functions

### Syntax Rules and Structure

**Complete General Syntax**

```python
def function(*args, **kwargs):
    ...
```

**Breakdown:**
- `*args`: A tuple of extra positional arguments.
- `**kwargs`: A dictionary of extra keyword arguments.

**Syntax Rules**

| Rule | Description |
|---|---|
| `*args` before `**kwargs` | `def f(*args, **kwargs)` is valid; reverse is a `SyntaxError` |
| Names are conventional | `args` and `kwargs` are conventions, not requirements |
| Binding order | `*args` collects after named positional parameters; `**kwargs` collects after named keyword parameters  |

### Multiple Annotated Complete Code Examples and Expected Outputs

**Example 1: `*args` for Summing**

```python
def sum_all(*numbers):
    total = 0
    for n in numbers:
        total += n
    return total

print(sum_all(1, 2, 3))        # 6
print(sum_all(10, 20, 30, 40)) # 100
```

**Expected Output:**
```
6
100
```

**Breakdown:** `numbers` is a tuple containing all positional arguments: `(1, 2, 3)` and `(10, 20, 30, 40)`.

**Example 2: `**kwargs` for Options**

```python
def configure(**options):
    for key, value in options.items():
        print(f"{key} = {value}")

configure(debug=True, verbose=False, level=3)
```

**Expected Output:**
```
debug = True
verbose = False
level = 3
```

**Breakdown:** `options` is a dictionary: `{'debug': True, 'verbose': False, 'level': 3}`.

**Example 3: Combined `*args` and `**kwargs`**

```python
def wrapper(*args, **kwargs):
    print(f"Positional: {args}")
    print(f"Keyword: {kwargs}")

wrapper(1, 2, 3, name="Alice", age=30)
```

**Expected Output:**
```
Positional: (1, 2, 3)
Keyword: {'name': 'Alice', 'age': 30}
```

**Breakdown:** `args` captures all positional arguments as a tuple; `kwargs` captures all keyword arguments as a dictionary .

### Real-World Cases with Explanation

**Case 1: Decorator Wrappers**

```python
def log_calls(func):
    def wrapper(*args, **kwargs):
        print(f"Calling {func.__name__}")
        return func(*args, **kwargs)
    return wrapper
```

**Why it matters:** Decorators need to accept any arguments the wrapped function takes, so `*args` and `**kwargs` are essential.

---

## Core Concept 5: Positional-Only Parameters (`/`)

### Definitions

**Core Definition**
Positional-only parameters are parameters that can only be passed by position, never by keyword. They are marked by placing a `/` after them in the function definition.

**Technical Definition**
Introduced in PEP 570, the `/` separator marks all parameters before it as `POSITIONAL_ONLY` in the `inspect.Parameter` sense . Their names are not usable as keywords in calls. This is useful for parameters whose names are implementation details or may change without breaking callers .

**Beginner-Friendly Explanation**
If you see `def f(a, b, /, c):`, then `a` and `b` can only be passed by position. `f(1, 2, 3)` works; `f(a=1, b=2, c=3)` fails. The `/` says "everything before me is position-only."

### Purposes

- **To prevent callers from relying on parameter names** that are implementation details
- **To allow parameter names to change** without breaking callers
- **To resolve ambiguity** when a parameter name conflicts with a `**kwargs` key 
- **To improve performance** slightly in CPython (positional-only parameters avoid dictionary lookups)

### Syntax Rules and Structure

**Complete General Syntax**

```python
def function(pos_only, /, standard, *, kw_only):
    ...
```

**Breakdown:**
- Parameters before `/`: positional-only.
- Parameters between `/` and `*`: positional-or-keyword.
- Parameters after `*`: keyword-only.

**Syntax Rules**

| Rule | Description |
|---|---|
| `/` marks the boundary | Everything before is positional-only  |
| `/` can appear with or without `*` | `def f(a, /, b)` is valid |
| No keyword calls for positional-only | `f(a=1)` raises `TypeError` if `a` is positional-only |

### Multiple Annotated Complete Code Examples and Expected Outputs

**Example 1: Basic Positional-Only**

```python
def greet(name, /):
    print(f"Hello, {name}!")

greet("Alice")       # Works
# greet(name="Bob")  # TypeError: greet() got some positional-only arguments passed as keyword arguments: 'name'
```

**Expected Output:**
```
Hello, Alice!
```

**Breakdown:** `name` is before `/`, so it must be passed positionally. Keyword call fails.

**Example 2: Mixed Parameter Kinds**

```python
def combined(pos_only, /, standard, *, kw_only):
    print(f"pos_only={pos_only}, standard={standard}, kw_only={kw_only}")

combined(1, 2, kw_only=3)
combined(1, standard=2, kw_only=3)
```

**Expected Output:**
```
pos_only=1, standard=2, kw_only=3
pos_only=1, standard=2, kw_only=3
```

**Breakdown:** `pos_only` must be positional. `standard` can be positional or keyword. `kw_only` must be keyword .

### Real-World Cases with Explanation

**Case 1: Built-in Function Signatures**

```python
# The inspect module reveals positional-only markers
import inspect
print(inspect.signature(divmod))
```

**Expected Output:**
```
(x, y, /)
```

**Why it matters:** Built-in functions like `divmod` use positional-only parameters because their parameter names are not part of the public API .

---

## Core Concept 6: Keyword-Only Parameters (`*`)

### Definitions

**Core Definition**
Keyword-only parameters are parameters that must be passed by keyword, never by position. They are marked by placing a bare `*` before them in the definition.

**Technical Definition**
A bare `*` in the parameter list signals that all following parameters are `KEYWORD_ONLY` . These parameters cannot be bound positionally. If a `*args` parameter is present, it serves the same role: parameters after `*args` are keyword-only .

**Beginner-Friendly Explanation**
Writing `def f(a, *, b):` means `a` can be positional, but `b` must be passed as `b=value`. This forces callers to be explicit about certain arguments, which improves readability.

### Purposes

- **To force explicit naming** of parameters whose meaning is not obvious from position
- **To prevent accidental argument mix-ups** in functions with many optional parameters
- **To make function calls more readable** by requiring keyword names for less obvious arguments
- **To allow adding new parameters** without breaking positional calls

### Syntax Rules and Structure

**Complete General Syntax**

```python
def function(*, kw_only1, kw_only2=default):
    ...
```

**Breakdown:**
- Bare `*`: Signals that following parameters are keyword-only.
- `kw_only1`: Required keyword argument.
- `kw_only2`: Optional keyword argument with default.

**Syntax Rules**

| Rule | Description |
|---|---|
| Bare `*` alone | Marks all following parameters as keyword-only  |
| `*args` also works | Parameters after `*args` are keyword-only |
| Must use keyword syntax | `f(kw_only1=1)` works; `f(1)` fails |

### Multiple Annotated Complete Code Examples and Expected Outputs

**Example 1: Basic Keyword-Only**

```python
def compare(a, b, *, key=None):
    print(f"a={a}, b={b}, key={key}")

compare(1, 2)
compare(1, 2, key="value")
# compare(1, 2, "value")  # TypeError
```

**Expected Output:**
```
a=1, b=2, key=None
a=1, b=2, key=value
```

**Breakdown:** `key` is after `*`, so it must be passed as `key="value"`. Positional passing fails .

**Example 2: Using `*args` to Enforce Keyword-Only**

```python
def process(first, *args, mode="normal"):
    print(f"first={first}, args={args}, mode={mode}")

process("a", "b", "c", mode="fast")
```

**Expected Output:**
```
first=a, args=('b', 'c'), mode=fast
```

**Breakdown:** `mode` is after `*args`, making it keyword-only. `args` captures `('b', 'c')` .

### Real-World Cases with Explanation

**Case 1: Configuration APIs**

```python
def request(url, *, timeout=30, retries=3):
    ...
```

**Why it matters:** `timeout` and `retries` are keyword-only because their meaning is not obvious from position. Forcing keywords prevents mistakes .

---

## Core Concept 7: Parameter Unpacking (`*` and `**` Operators)

### Definitions

**Core Definition**
The `*` and `**` operators at the call site unpack iterables and mappings into positional and keyword arguments, respectively.

**Technical Definition**
`*iterable` unpacks the iterable's elements as positional arguments: `f(*[1, 2, 3])` is equivalent to `f(1, 2, 3)`. `**mapping` unpacks the dictionary's key-value pairs as keyword arguments: `f(**{"a": 1, "b": 2})` is equivalent to `f(a=1, b=2)` . Unpacking can be combined with regular arguments.

**Beginner-Friendly Explanation**
If you have a list of values you want to pass as separate arguments, use `*` to "spread" them. If you have a dictionary of keyword arguments, use `**` to spread them. This is the reverse of `*args` and `**kwargs` in definitions.

### Purposes

- **To pass a list or tuple as separate positional arguments**
- **To pass a dictionary as keyword arguments**
- **To forward arguments** from one function to another
- **To dynamically construct calls** when arguments come from data structures

### Syntax Rules and Structure

**Complete General Syntax**

```python
function(*iterable)
function(**mapping)
function(*iterable, **mapping)
```

**Breakdown:**
- `*iterable`: Each element becomes a positional argument.
- `**mapping`: Each key-value pair becomes a keyword argument.
- Both can be combined with regular arguments.

**Syntax Rules**

| Rule | Description |
|---|---|
| `*` on any iterable | Works with lists, tuples, strings, etc.  |
| `**` on any mapping | Works with dictionaries |
| Order | Positional unpacking before keyword unpacking |
| Multiple unpackings | Can use multiple `*` and `**` in one call  |

### Multiple Annotated Complete Code Examples and Expected Outputs

**Example 1: Unpacking a List**

```python
def add(a, b, c):
    return a + b + c

numbers = [1, 2, 3]
result = add(*numbers)
print(result)
```

**Expected Output:**
```
6
```

**Breakdown:** `*numbers` unpacks `[1, 2, 3]` into `add(1, 2, 3)` .

**Example 2: Unpacking a Dictionary**

```python
def greet(name, greeting):
    print(f"{greeting}, {name}!")

params = {"name": "Alice", "greeting": "Hello"}
greet(**params)
```

**Expected Output:**
```
Hello, Alice!
```

**Breakdown:** `**params` unpacks the dictionary into `greet(name="Alice", greeting="Hello")` .

**Example 3: Combining Unpacking with Regular Arguments**

```python
def mixed(a, b, c, d):
    print(a, b, c, d)

args = (2, 3)
kwargs = {"d": 4}
mixed(1, *args, **kwargs)
```

**Expected Output:**
```
1 2 3 4
```

**Breakdown:** `1` is positional, `*args` supplies `2` and `3`, `**kwargs` supplies `d=4` .

### Real-World Cases with Explanation

**Case 1: Forwarding Arguments**

```python
def log_and_call(func, *args, **kwargs):
    print(f"Calling {func.__name__} with {args}, {kwargs}")
    return func(*args, **kwargs)
```

**Why it matters:** Wrappers receive `*args` and `**kwargs`, then forward them using `*` and `**` to the wrapped function .

---

## References

- Python Software Foundation. *inspect — Inspect live objects*. https://docs.python.org/3/library/inspect.html 
- Python Software Foundation. *PEP 570 – Python Positional-Only Parameters*. https://peps.python.org/pep-0570/ 
- Python.org Tutor Mailing List. *parameters vs arguments*. https://mail.python.org/archives/list/tutor@python.org/thread/5FVSVIIMJRBQ2SHMTIPDCHOLNVJAJYXP/ 
- Python.org Tutor Mailing List. *Playing with generators*. https://mail.python.org/pipermail/tutor/2022-August/120037.html 
- Python.org Python-list. *Mutable Default Arguments*. https://mail.python.org/pipermail/chennaipy/attachments/20221126/052f2653/attachment-0001.pdf 
- Python.org Tutor Mailing List. **args, **kwargs*. https://mail.python.org/archives/list/tutor@python.org/message/JTZLTORJD2T4Y5D7RJPWSRBII2OG6KMT/ 