# Advanced Functions & Metaprogramming: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
Advanced functions and metaprogramming in Python refer to the language's treatment of functions as first-class objects and the techniques that leverage this property—including higher-order functions, closures, decorators, and callable objects—to write flexible, reusable, and self-modifying code.

**Technical Definition**
Python implements functions as first-class objects: function objects can be created at runtime, assigned to variables, stored in data structures, passed as arguments, and returned from other functions. This enables higher-order functions (functions that accept or return other functions), closures (functions that capture free variables from enclosing scopes), and metaprogramming patterns like decorators that wrap callables to modify their behavior. The `functools` module provides utilities such as `wraps` (to preserve metadata) and `lru_cache` (for memoization) that support these patterns.

**Beginner-Friendly Explanation**
In Python, a function is not just a block of code—it is a "thing" you can hold, pass around, and modify. You can store a function in a variable, give it to another function as input, or even write a function that creates new functions on the fly. This flexibility lets you write code that writes code, which is the essence of metaprogramming. Decorators are the most common metaprogramming tool: they wrap a function to add behavior (like timing or caching) without changing the function's source code.

### Key Characteristics

| Characteristic | Description |
|---|---|
| **First-Class Functions** | Functions are objects: can be assigned, stored, passed, and returned |
| **Lexical Scoping** | Nested functions capture variables from enclosing scopes via closures |
| **Metaprogramming** | Functions can inspect and modify other functions at runtime |
| **Decorator Syntax** | The `@` syntax provides a clean way to apply wrappers |
| **Utility Support** | `functools` provides `wraps`, `lru_cache`, `partial`, and more |
| **Recursion Limits** | Python limits call stack depth (default 1000) to prevent crashes |

### Prerequisites

- **Basic function definitions and calls**
- **Scope and namespace concepts** (LEGB rule, closures)
- **Basic understanding of objects and classes**

### Related Programming Areas

- **Functional Programming**: Higher-order functions, `map`, `filter`, `reduce`
- **Object-Oriented Programming**: Callable objects and `__call__`
- **Decorator Design Patterns**: Authentication, logging, caching, rate limiting
- **Memoization**: Caching expensive function results

### Core Concepts / Features

1. **First-Class Functions** (functions as objects)
2. **Higher-Order Functions** (functions that accept/return functions)
3. **Nested Functions** (functions defined inside functions)
4. **Closures and `__closure__`** (captured free variables)
5. **Function Factories** (functions that create specialized functions)
6. **Lambda Expressions** (anonymous inline functions)
7. **Recursion and Limits** (execution frames, `sys.getrecursionlimit()`)
8. **Callable Objects** (`__call__` method)
9. **Decorators** (function decorators, stacked decorators, parameterized decorators)
10. **Functools Utilities** (`wraps`, `lru_cache`)

---

## Core Concept 1: First-Class Functions

### Definitions

**Core Definition**
First-class functions are functions that can be treated like any other object: assigned to variables, stored in data structures, passed as arguments, and returned as values.

**Technical Definition**
In Python, functions are first-class objects created by the `def` statement or `lambda` expression. They are instances of `types.FunctionType` and support the same operations as other objects: assignment, storage, parameter passing, and return. A "first-class" function means functions themselves are values in the type system, not merely relationships between values.

**Beginner-Friendly Explanation**
In some programming languages, functions are special—you can only call them, not treat them as data. In Python, functions are just objects, like integers or strings. You can put a function in a list, pass it to another function, or return it from a function. This is what "first-class" means: functions have the same privileges as any other value.

### Purposes

- **To enable functional programming patterns** like map, filter, and reduce
- **To allow functions to be stored and retrieved dynamically** based on runtime conditions
- **To support higher-order functions and decorators**
- **To enable callbacks and event handlers**
- **To facilitate dynamic dispatch and plugin architectures**

### Syntax Rules and Structure

**Complete General Syntax**

```python
def my_function():
    return "Hello"

# Assign to variable
ref = my_function

# Store in data structure
functions = [my_function, len, str.upper]

# Pass as argument
def call_func(f):
    return f()

# Return from function
def get_function():
    return my_function
```

**Syntax Rules**

| Rule | Description |
|---|---|
| Functions are objects | Can be assigned, stored, passed, returned |
| Call requires parentheses | `my_function` is the object; `my_function()` calls it |
| No special syntax needed | Functions behave like any other object |

### Multiple Annotated Complete Code Examples and Expected Outputs

**Example 1: Assigning Functions to Variables**

```python
def greet(name):
    return f"Hello, {name}!"

# Assign the function object to a variable
say_hello = greet

# Call through the new name
print(say_hello("Alice"))
print(greet("Bob"))
```

**Expected Output:**
```
Hello, Alice!
Hello, Bob!
```

**Breakdown:** `say_hello` and `greet` refer to the same function object. Calling either produces the same result.

**Example 2: Storing Functions in Data Structures**

```python
def add(a, b):
    return a + b

def subtract(a, b):
    return a - b

def multiply(a, b):
    return a * b

# Store functions in a dictionary
operations = {
    "add": add,
    "subtract": subtract,
    "multiply": multiply,
}

# Look up and call dynamically
result = operations["add"](10, 5)
print(result)

result = operations["multiply"](4, 3)
print(result)
```

**Expected Output:**
```
15
12
```

**Breakdown:** Functions are stored as dictionary values and retrieved by key. This is the basis for dispatch tables and plugin systems.

**Example 3: Passing Functions as Arguments**

```python
def apply_operation(func, x, y):
    """Apply a binary function to x and y."""
    return func(x, y)

def power(base, exponent):
    return base ** exponent

# Pass the function as an argument
result = apply_operation(power, 2, 3)
print(result)

# Use a built-in function
result = apply_operation(max, 10, 20)
print(result)
```

**Expected Output:**
```
8
20
```

**Breakdown:** `apply_operation` is a higher-order function because it accepts a function as a parameter. This pattern enables generic algorithms.

### Real-World Cases with Explanation

**Case 1: Command Dispatch**

```python
def save_file():
    print("Saving...")

def load_file():
    print("Loading...")

def quit_app():
    print("Quitting...")

commands = {"save": save_file, "load": load_file, "quit": quit_app}

user_input = "save"
commands[user_input]()
# Output: Saving...
```

**Why it matters:** Dispatch tables using first-class functions eliminate long `if/elif` chains and make the code extensible.

---

## Core Concept 2: Higher-Order Functions

### Definitions

**Core Definition**
A higher-order function is a function that takes one or more functions as arguments or returns a function as its result.

**Technical Definition**
Higher-order functions are possible because functions are first-class objects. Built-in examples include `map()`, `filter()`, and `sorted()` (with a `key` function). User-defined higher-order functions are common in decorators, callbacks, and functional pipelines.

**Beginner-Friendly Explanation**
A higher-order function is a function that works on other functions. It might take a function as input (like "apply this operation to each item") or produce a function as output (like "make me a function that adds 5"). This is a powerful abstraction tool.

### Purposes

- **To abstract over operations** rather than just values
- **To create generic algorithms** that can be customized with different functions
- **To implement decorators and callbacks**
- **To support functional programming patterns**

### Syntax Rules and Structure

```python
def higher_order(func):
    """Takes a function, returns a function or calls it."""
    def wrapper(*args, **kwargs):
        return func(*args, **kwargs)
    return wrapper
```

### Multiple Annotated Complete Code Examples and Expected Outputs

**Example 1: `map` as a Higher-Order Function**

```python
numbers = [1, 2, 3, 4]

# map takes a function and an iterable
squares = list(map(lambda x: x ** 2, numbers))
print(squares)
```

**Expected Output:**
```
[1, 4, 9, 16]
```

**Breakdown:** `map` applies the lambda to each element. The lambda is passed as an argument.

**Example 2: Custom Higher-Order Function**

```python
def repeat(n):
    """Return a function that repeats another function n times."""
    def repeater(func):
        def wrapper(*args, **kwargs):
            for _ in range(n):
                result = func(*args, **kwargs)
            return result
        return wrapper
    return repeater

@repeat(3)
def greet(name):
    print(f"Hello, {name}!")

greet("Alice")
```

**Expected Output:**
```
Hello, Alice!
Hello, Alice!
Hello, Alice!
```

**Breakdown:** `repeat(3)` returns `repeater`, which returns `wrapper`. This is a parameterized decorator (see Core Concept 9).

### Real-World Cases with Explanation

**Case 1: Data Pipeline**

```python
def pipeline(*functions):
    def run(value):
        for func in functions:
            value = func(value)
        return value
    return run

process = pipeline(str.strip, str.lower, lambda s: s.replace(" ", "_"))
result = process("  Hello World  ")
print(result)
# Output: hello_world
```

**Why it matters:** Pipelines compose simple functions into complex transformations, making data processing declarative.

---

## Core Concept 3: Nested Functions

### Definitions

**Core Definition**
A nested function (also called an inner function) is a function defined inside another function.

**Technical Definition**
Nested functions are created each time the outer function is called. They can access variables from the enclosing function's scope (free variables) and are the foundation of closures and decorators.

**Beginner-Friendly Explanation**
You can define a function inside another function. The inner function can see the outer function's variables. This is useful for creating helper functions that are only needed inside one place.

### Purposes

- **To encapsulate helper logic** that is only relevant to the outer function
- **To create closures** that capture state from the enclosing scope
- **To implement decorators** (the wrapper is a nested function)
- **To avoid polluting the global namespace**

### Syntax Rules and Structure

```python
def outer(x):
    def inner(y):
        return y + 5
    return inner(x)

result = outer(10)
print(result)
```

### Multiple Annotated Complete Code Examples and Expected Outputs

**Example 1: Basic Nested Function**

```python
def outer(x):
    def inner(y):
        return y + 5
    return inner(x)

print(outer(10))
```

**Expected Output:**
```
15
```

**Breakdown:** `inner` is defined inside `outer` and called immediately. It is not accessible outside.

**Example 2: Nested Function Accessing Outer Variable**

```python
def make_adder(n):
    def adder(x):
        return x + n
    return adder

add_5 = make_adder(5)
print(add_5(10))
```

**Expected Output:**
```
15
```

**Breakdown:** `adder` captures `n` from `make_adder`'s scope. This is a closure.

### Real-World Cases with Explanation

**Case 1: Validation Helpers**

```python
def validate_user(user):
    def check_name(name):
        return len(name) > 0

    def check_age(age):
        return age >= 0

    return check_name(user["name"]) and check_age(user["age"])
```

**Why it matters:** Nested functions keep validation logic scoped to the function that uses it, improving organization.

---

## Core Concept 4: Closures and `__closure__`

### Definitions

**Core Definition**
A closure is a nested function that captures and remembers variables from its enclosing scope, even after the outer function has returned.

**Technical Definition**
When a nested function references a variable from an enclosing function, Python stores that variable in a cell object. The function's `__closure__` attribute is a tuple of these cells, accessible via `cell_contents`. The captured variables are called "free variables" because they are not bound in the nested function's local scope.

**Beginner-Friendly Explanation**
A closure "closes over" variables from its outer function. Even after the outer function finishes, the inner function still remembers those variables. You can inspect them with `__closure__`, though you rarely need to do so directly.

### Purposes

- **To preserve state** across function calls without using global variables
- **To create function factories** that produce specialized functions
- **To implement decorators** that remember configuration
- **To support callbacks** that need to remember context

### Syntax Rules and Structure

```python
def outer():
    x = 10
    def inner():
        return x  # x is a free variable
    return inner

closure_func = outer()
print(closure_func())  # 10

# Inspect the closure
print(closure_func.__closure__)  # (<cell at ...>,)
print(closure_func.__closure__[0].cell_contents)  # 10
```

### Multiple Annotated Complete Code Examples and Expected Outputs

**Example 1: Closure Preserving State**

```python
def counter():
    count = 0
    def increment():
        nonlocal count
        count += 1
        return count
    return increment

c = counter()
print(c())
print(c())
print(c())
```

**Expected Output:**
```
1
2
3
```

**Breakdown:** `count` is captured in the closure. Each call to `c` increments and returns the same `count` variable.

**Example 2: Inspecting `__closure__`**

```python
def make_multiplier(factor):
    def multiply(x):
        return x * factor
    return multiply

double = make_multiplier(2)
print(double(5))

# Inspect the captured variable
print(double.__closure__[0].cell_contents)
```

**Expected Output:**
```
10
2
```

**Breakdown:** `factor` is stored in a cell object. `__closure__[0].cell_contents` retrieves the value `2`.

### Real-World Cases with Explanation

**Case 1: Configuration-Based Functions**

```python
def create_formatter(prefix, suffix):
    def format_text(text):
        return f"{prefix}{text}{suffix}"
    return format_text

html_bold = create_formatter("<b>", "</b>")
print(html_bold("Hello"))
# Output: <b>Hello</b>
```

**Why it matters:** Closures enable creating specialized functions without classes, which is useful for simple configuration-based behavior.

---

## Core Concept 5: Function Factories

### Definitions

**Core Definition**
A function factory is a function that creates and returns new functions, often specialized with captured parameters.

**Technical Definition**
Function factories use closures to produce functions with customized behavior. They are a practical application of higher-order functions and nested functions, commonly used in decorators, partial application, and configuration-driven design.

**Beginner-Friendly Explanation**
A function factory is a "function that makes functions." You give it some configuration, and it gives you back a new function tailored to that configuration. It's like a cookie cutter: you choose the shape, and it produces cookies of that shape.

### Purposes

- **To create specialized functions** from a general template
- **To avoid repetitive function definitions**
- **To support configuration-driven behavior**
- **To implement partial application** (pre-filling some arguments)

### Syntax Rules and Structure

```python
def factory(config):
    def specialized(x):
        return x + config
    return specialized
```

### Multiple Annotated Complete Code Examples and Expected Outputs

**Example 1: Multiplier Factory**

```python
def make_multiplier(factor):
    def multiply(x):
        return x * factor
    return multiply

double = make_multiplier(2)
triple = make_multiplier(3)

print(double(10))
print(triple(10))
```

**Expected Output:**
```
20
30
```

**Breakdown:** Each call to `make_multiplier` creates a new closure with a different `factor` value.

**Example 2: Using `functools.partial`**

```python
from functools import partial

def power(base, exponent):
    return base ** exponent

# Create a specialized function
square = partial(power, exponent=2)
cube = partial(power, exponent=3)

print(square(5))
print(cube(5))
```

**Expected Output:**
```
25
125
```

**Breakdown:** `partial` is a built-in function factory that pre-fills arguments. It is a simpler alternative to writing custom closures for simple cases.

### Real-World Cases with Explanation

**Case 1: API Client Configuration**

```python
def create_api_client(base_url):
    def client(endpoint):
        return f"{base_url}/{endpoint}"
    return client

github = create_api_client("https://api.github.com")
print(github("users"))
# Output: https://api.github.com/users
```

**Why it matters:** Function factories allow creating configured clients without classes or global state.

---

## Core Concept 6: Lambda Expressions

### Definitions

**Core Definition**
A lambda expression creates an anonymous, single-expression function inline.

**Technical Definition**
Lambda expressions are syntactically restricted to a single expression. They are semantically equivalent to a normal function definition with that expression as the body. Like nested functions, lambdas can reference variables from the containing scope.

**Beginner-Friendly Explanation**
A lambda is a tiny function you write inline without a `def` statement. It's useful for short, throwaway functions where naming them would be overkill. For example, `lambda x: x + 1` is a function that adds 1 to its argument.

### Purposes

- **To create short, inline functions** for callbacks and keys
- **To avoid naming trivial functions** that are used only once
- **To pass functions to higher-order functions** like `map`, `filter`, and `sorted`

### Syntax Rules and Structure

```python
lambda arguments: expression
```

**Syntax Rules**

| Rule | Description |
|---|---|
| Single expression only | No statements, assignments, or multiple lines |
| Returns the expression | No `return` keyword needed |
| Can be assigned | `f = lambda x: x + 1` is valid but discouraged by PEP 8 |
| Can capture enclosing variables | Works like nested functions |

### Multiple Annotated Complete Code Examples and Expected Outputs

**Example 1: Lambda with `sorted`**

```python
students = [("Alice", 85), ("Bob", 92), ("Charlie", 78)]

# Sort by score (second element)
sorted_students = sorted(students, key=lambda s: s[1])
print(sorted_students)
```

**Expected Output:**
```
[('Charlie', 78), ('Alice', 85), ('Bob', 92)]
```

**Breakdown:** The lambda extracts the second element for comparison.

**Example 2: Lambda in `map`**

```python
numbers = [1, 2, 3, 4]
doubled = list(map(lambda x: x * 2, numbers))
print(doubled)
```

**Expected Output:**
```
[2, 4, 6, 8]
```

**Breakdown:** The lambda defines the transformation applied by `map`.

**Example 3: Lambda Capturing Scope**

```python
def make_incrementor(n):
    return lambda x: x + n

f = make_incrementor(42)
print(f(0))
print(f(1))
```

**Expected Output:**
```
42
43
```

**Breakdown:** The lambda captures `n` from `make_incrementor`'s scope.

### Real-World Cases with Explanation

**Case 1: Sorting by Multiple Criteria**

```python
data = [("Alice", 30, 85), ("Bob", 25, 92), ("Charlie", 30, 78)]

# Sort by age, then by score descending
sorted_data = sorted(data, key=lambda x: (x[1], -x[2]))
print(sorted_data)
```

**Why it matters:** Lambdas with tuple keys enable complex sorting without writing named functions.

---

## Core Concept 7: Recursion and Execution Frame Limitations

### Definitions

**Core Definition**
Recursion is a technique where a function calls itself to solve a problem by breaking it into smaller subproblems.

**Technical Definition**
Each recursive call creates a new execution frame on the call stack. Python limits the maximum call stack depth to prevent stack overflow crashes. The default limit is 1000 frames, accessible and modifiable via `sys.getrecursionlimit()` and `sys.setrecursionlimit()`.

**Beginner-Friendly Explanation**
Recursion is when a function calls itself. It's useful for problems that can be divided into smaller versions of the same problem (like calculating factorials or traversing trees). But if recursion goes too deep, Python raises a `RecursionError` to protect against crashes.

### Purposes

- **To solve problems with recursive structure** (trees, fractals, divide-and-conquer)
- **To implement algorithms elegantly** (factorial, Fibonacci, quicksort)
- **To traverse nested data structures**

### Syntax Rules and Structure

```python
def recursive_function(n):
    if n <= 1:          # Base case
        return 1
    return n * recursive_function(n - 1)  # Recursive case
```

### Multiple Annotated Complete Code Examples and Expected Outputs

**Example 1: Factorial**

```python
def factorial(n):
    if n <= 1:
        return 1
    return n * factorial(n - 1)

print(factorial(5))
```

**Expected Output:**
```
120
```

**Breakdown:** Each call multiplies `n` by the factorial of `n-1`, terminating at `n=1`.

**Example 2: Recursion Limit Error**

```python
import sys

def infinite_recursion(n):
    return infinite_recursion(n + 1)

try:
    infinite_recursion(0)
except RecursionError as e:
    print(f"Error: {e}")
    print(f"Limit: {sys.getrecursionlimit()}")
```

**Expected Output:**
```
Error: maximum recursion depth exceeded
Limit: 1000
```

**Breakdown:** Python raises `RecursionError` when the call stack exceeds the limit. The default is 1000.

### Real-World Cases with Explanation

**Case 1: Tree Traversal**

```python
def traverse(node):
    if node is None:
        return
    print(node.value)
    traverse(node.left)
    traverse(node.right)
```

**Why it matters:** Recursion is natural for tree structures where each node has children of the same type.

---

## Core Concept 8: Callable Objects (`__call__`)

### Definitions

**Core Definition**
A callable object is an instance of a class that defines the `__call__` method, allowing instances to be called like functions.

**Technical Definition**
When a class implements `__call__`, instances of that class become callable. Calling an instance invokes `__call__` with the provided arguments. This allows objects to maintain state while offering a function-like interface.

**Beginner-Friendly Explanation**
Normally, you call functions with parentheses. But you can also make objects callable by defining `__call__`. Then you can use the object as if it were a function, but it can remember state between calls.

### Purposes

- **To create stateful callables** that remember data between calls
- **To implement function-like objects** with methods and attributes
- **To replace closures** when more complex state management is needed
- **To create decorators as classes**

### Syntax Rules and Structure

```python
class CallableClass:
    def __init__(self, value):
        self.value = value

    def __call__(self, x):
        return x + self.value

obj = CallableClass(10)
result = obj(5)  # Calls __call__
```

### Multiple Annotated Complete Code Examples and Expected Outputs

**Example 1: Basic Callable Object**

```python
class Adder:
    def __init__(self, n):
        self.n = n

    def __call__(self, x):
        return x + self.n

add_5 = Adder(5)
print(add_5(10))
print(add_5(20))
```

**Expected Output:**
```
15
25
```

**Breakdown:** `add_5` is an instance, not a function, but calling it invokes `__call__`.

**Example 2: Stateful Counter**

```python
class Counter:
    def __init__(self):
        self.count = 0

    def __call__(self):
        self.count += 1
        return self.count

c = Counter()
print(c())
print(c())
print(c())
```

**Expected Output:**
```
1
2
3
```

**Breakdown:** The instance maintains state (`count`) across calls, unlike a plain function closure.

### Real-World Cases with Explanation

**Case 1: Middleware in Web Frameworks**

```python
class TimingMiddleware:
    def __init__(self, app):
        self.app = app

    def __call__(self, request):
        start = time.time()
        response = self.app(request)
        print(f"Request took {time.time() - start:.3f}s")
        return response
```

**Why it matters:** Callable objects are common in middleware patterns where state (like configuration) is set once and the object is called per request.

---

## Core Concept 9: Decorators

### Definitions

**Core Definition**
A decorator is a callable that takes a function and returns a modified version of it, applied using the `@` syntax.

**Technical Definition**
Decorators use closures to wrap functions. The `@decorator` syntax is equivalent to `func = decorator(func)`. Stacked decorators apply from bottom to top. Parameterized decorators require three levels of nesting: a factory that returns a decorator that returns a wrapper. Since PEP 614, decorators can be any valid expression.

**Beginner-Friendly Explanation**
A decorator is like a gift wrapper for functions. You put your function inside, and the decorator adds something extra—like logging, timing, or caching—without changing the function itself. You apply it with `@` above the function definition.

### Purposes

- **To add behavior to functions** without modifying their source code
- **To implement cross-cutting concerns** (logging, authentication, caching)
- **To compose multiple behaviors** by stacking decorators
- **To create reusable function enhancements**

### Syntax Rules and Structure

**Basic Decorator:**

```python
def my_decorator(func):
    def wrapper(*args, **kwargs):
        print("Before call")
        result = func(*args, **kwargs)
        print("After call")
        return result
    return wrapper

@my_decorator
def say_hello():
    print("Hello!")

say_hello()
```

**Parameterized Decorator:**

```python
def repeat(times):
    def decorator(func):
        def wrapper(*args, **kwargs):
            for _ in range(times):
                result = func(*args, **kwargs)
            return result
        return wrapper
    return decorator

@repeat(times=3)
def greet(name):
    print(f"Hello, {name}!")

greet("Alice")
```

**Stacked Decorators:**

```python
@decorator1
@decorator2
def func():
    pass
# Equivalent to: func = decorator1(decorator2(func))
```

### Multiple Annotated Complete Code Examples and Expected Outputs

**Example 1: Simple Decorator**

```python
def log_calls(func):
    def wrapper(*args, **kwargs):
        print(f"Calling {func.__name__}")
        return func(*args, **kwargs)
    return wrapper

@log_calls
def add(a, b):
    return a + b

print(add(3, 4))
```

**Expected Output:**
```
Calling add
7
```

**Breakdown:** `log_calls` wraps `add`. The wrapper prints before calling the original.

**Example 2: Parameterized Decorator**

```python
def repeat(times):
    def decorator(func):
        def wrapper(*args, **kwargs):
            for _ in range(times):
                result = func(*args, **kwargs)
            return result
        return wrapper
    return decorator

@repeat(times=3)
def greet(name):
    print(f"Hello, {name}!")

greet("Alice")
```

**Expected Output:**
```
Hello, Alice!
Hello, Alice!
Hello, Alice!
```

**Breakdown:** `repeat(3)` returns `decorator`, which returns `wrapper`. The `@` syntax applies the result.

**Example 3: Stacked Decorators**

```python
def bold(func):
    def wrapper():
        return f"<b>{func()}</b>"
    return wrapper

def italic(func):
    def wrapper():
        return f"<i>{func()}</i>"
    return wrapper

@bold
@italic
def greet():
    return "Hello"

print(greet())
```

**Expected Output:**
```
<b><i>Hello</i></b>
```

**Breakdown:** Decorators apply bottom-up: `italic` wraps `greet` first, then `bold` wraps the result.

### Real-World Cases with Explanation

**Case 1: Authentication Decorator**

```python
def require_auth(func):
    def wrapper(user, *args, **kwargs):
        if not user.get("authenticated"):
            raise PermissionError("Authentication required")
        return func(user, *args, **kwargs)
    return wrapper
```

**Why it matters:** Decorators enforce cross-cutting concerns like authentication without cluttering business logic.

---

## Core Concept 10: Functools Utilities

### Definitions

**Core Definition**
The `functools` module provides higher-order functions and utilities for working with callables, including `wraps` (metadata preservation) and `lru_cache` (memoization).

**Technical Definition**
`functools.wraps` is a decorator that copies `__name__`, `__doc__`, `__module__`, `__annotations__`, and `__dict__` from the wrapped function to the wrapper, preserving introspection. `functools.lru_cache` implements a least-recently-used cache for memoization, storing results of function calls and returning cached values for repeated inputs.

**Beginner-Friendly Explanation**
`wraps` makes sure that when you wrap a function with a decorator, the wrapped function still "looks like" the original (same name, same docstring). `lru_cache` remembers the results of function calls so that if you call the function again with the same arguments, it returns the cached result instantly instead of recomputing.

### Purposes

- **To preserve function metadata** when writing decorators (`wraps`)
- **To speed up expensive functions** by caching results (`lru_cache`)
- **To provide partial function application** (`partial`)
- **To support functional programming patterns**

### Syntax Rules and Structure

```python
from functools import wraps, lru_cache

@lru_cache(maxsize=128)
def expensive_function(n):
    # Expensive computation
    return n * n

def my_decorator(func):
    @wraps(func)
    def wrapper(*args, **kwargs):
        return func(*args, **kwargs)
    return wrapper
```

### Multiple Annotated Complete Code Examples and Expected Outputs

**Example 1: `wraps` Preserving Metadata**

```python
from functools import wraps

def my_decorator(func):
    @wraps(func)
    def wrapper(*args, **kwargs):
        return func(*args, **kwargs)
    return wrapper

@my_decorator
def greet(name):
    """Greet someone by name."""
    return f"Hello, {name}!"

print(greet.__name__)
print(greet.__doc__)
```

**Expected Output:**
```
greet
Greet someone by name.
```

**Breakdown:** Without `@wraps`, `greet.__name__` would be `"wrapper"` and the docstring would be lost.

**Example 2: `lru_cache` Memoization**

```python
from functools import lru_cache
import time

@lru_cache(maxsize=None)
def slow_square(n):
    time.sleep(1)  # Simulate expensive computation
    return n * n

start = time.time()
print(slow_square(5))  # Takes ~1 second
print(slow_square(5))  # Instant (cached)
print(f"Time: {time.time() - start:.2f}s")
```

**Expected Output:**
```
25
25
Time: 1.00s
```

**Breakdown:** The second call returns the cached result without sleeping. `cache_info()` can inspect hits and misses.

**Example 3: `lru_cache` Statistics**

```python
from functools import lru_cache

@lru_cache(maxsize=128)
def fibonacci(n):
    if n < 2:
        return n
    return fibonacci(n - 1) + fibonacci(n - 2)

fibonacci(100)
print(fibonacci.cache_info())
```

**Expected Output:**
```
CacheInfo(hits=98, misses=101, maxsize=128, currsize=101)
```

**Breakdown:** `cache_info()` shows cache performance. `cache_clear()` resets the cache.

### Real-World Cases with Explanation

**Case 1: API Response Caching**

```python
from functools import lru_cache

@lru_cache(maxsize=100)
def fetch_user(user_id):
    # Expensive API call
    return api.get(f"/users/{user_id}")
```

**Why it matters:** Caching API responses reduces latency and rate-limit issues for repeated lookups.

---

## References

- Python.org Tutor Mailing List. *Decorators, __call__ (able) objects*. https://mail.python.org/archives/list/tutor@python.org/message/2G25E3AWUTBF2QGMXEEJV2I4Y3RWJNT2/ 
- Python.org Python-list. *Recursion limit*. https://mail.python.org/pipermail/python-list/2003-May/225408.html 
- Python.org Python-list. *Recursion limit (2)*. https://mail.python.org/pipermail/python-list/2005-November/304643.html 
- Python.org Python-list. *Apply() [function calling depends on end parentheses]*. https://mail.python.org/pipermail/tutor/2003-February/020546.html 
- Python.org Tutor Mailing List. *Lambda Forms*. https://mail.python.org/pipermail/tutor/attachments/20120831/2dd31246/attachment-0001.html 
- Python.org Tutor Mailing List. *Lambda?? Whaaaaat?*. https://mail.python.org/pipermail/tutor/2012-August/091159.html 
- Python.org Python-list. *Function factory?*. https://mail.python.org/archives/list/python-list@python.org/message/GQP3IN5FMKNJG7YY7VRDFFN5QA4FDURT/ 
- Python.org Python-Dev. *PEP309 re-written*. https://mail.python.org/pipermail/python-dev/2004-April/044055.html 
- Python.org Python-ideas. *Unify global and nonlocal*. https://mail.python.org/pipermail/python-ideas/2014