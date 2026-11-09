# Python Decorators: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**

A decorator is a function that takes another function (or class) as an argument, extends or modifies its behavior without explicitly changing its source code, and returns a new function (or class). Decorators provide a clean, readable way to wrap functionality around existing code.

**Technical Definition**

A decorator is a callable that accepts a callable as its sole argument and returns a callable (or another object) as its result. PEP 318 introduced the `@decorator` syntax in Python 2.4, making decorators a first-class language feature. The syntax `@decorator` above a function definition is equivalent to `func = decorator(func)`. Multiple decorators are applied in nested fashion: `@f1(arg)` above `@f2` above `def func()` is equivalent to `func = f1(arg)(f2(func))`. The `functools` module provides utilities for writing well-behaved decorators, including `wraps`, `lru_cache`, `singledispatch`, and `cached_property`.

**Beginner-Friendly Explanation**

Think of a decorator as a gift wrapper. You have a present (a function), and you want to add a bow, a card, or a layer of bubble wrap (extra behavior) without changing the present itself. A decorator wraps your function with extra functionality. For example, you can add logging, timing, or caching to a function without touching its original code. You write `@decorator` above your function, and Python automatically wraps it for you.

### Key Characteristics

- **Higher-Order Functions**: Decorators take functions as arguments and return functions, making them higher-order functions.
- **Syntactic Sugar**: The `@decorator` syntax is a convenient shorthand for `func = decorator(func)`.
- **Non-Invasive**: Decorators modify behavior without changing the decorated function's source code.
- **Composable**: Multiple decorators can be stacked, applying in nested fashion.
- **Versatile**: Decorators can be applied to functions, methods, and classes.
- **Metadata Preservation**: `functools.wraps` preserves the original function's name, docstring, and other metadata.
- **Decorator Factories**: Decorators that accept arguments require an extra layer of nesting.
- **Built-in Support**: Python provides several built-in decorators (`@property`, `@staticmethod`, `@classmethod`, `@abstractmethod`, `@dataclass`, `@contextmanager`, `@atexit.register`).

### Prerequisites

- Basic understanding of Python functions and the `def` keyword.
- Familiarity with first-class functions (functions as objects).
- Knowledge of nested functions and closures.
- Understanding of function arguments (`*args`, `**kwargs`) and return values.
- Awareness of the `functools` module.

### Related Programming Areas

- **Functional Programming**: Decorators are higher-order functions that enable function composition and transformation.
- **Aspect-Oriented Programming**: Decorators implement cross-cutting concerns like logging, timing, and authentication.
- **Object-Oriented Programming**: Class decorators (`@dataclass`, `@total_ordering`) and method decorators (`@property`, `@staticmethod`) are integral to OOP.
- **Web Development**: Flask and Django use decorators for routing, authentication, and middleware.
- **Caching and Performance**: `@lru_cache` and `@cached_property` optimize performance.
- **Testing**: Decorators like `@pytest.mark` configure test behavior.

---

## Core Concepts / Features

---

### 1. Decorators and Decorator Syntax

#### Definitions

**Core Definition**

A decorator is a function that wraps another function, adding behavior before or after the wrapped function runs. The `@decorator` syntax is placed above a function definition to apply the decorator.

**Technical Definition**

The `@decorator` syntax is syntactic sugar: `@my_decorator` above `def my_func()` is equivalent to `my_func = my_decorator(my_func)`. A decorator must be a callable that accepts a callable as its argument and returns a callable. Decorator expressions are evaluated when the function is defined, in the scope that contains the function definition. The result must be callable and is invoked with the function object as the only argument. The returned value is bound to the function name.

**Beginner-Friendly Explanation**

A decorator is like a label you stick on a function. The label says, "Before running this function, do this; after it finishes, do that." You write `@decorator` above the function, and Python applies the decoration automatically.

#### Purposes

- To add behavior to functions without modifying their source code.
- To implement cross-cutting concerns like logging, timing, and authentication.
- To create reusable function wrappers.
- To keep code clean and readable.

#### Syntax Rules and Structure

**Complete General Syntax**

```
# Basic decorator
def decorator(func):
    def wrapper(*args, **kwargs):
        # Before
        result = func(*args, **kwargs)
        # After
        return result
    return wrapper

@decorator
def my_function():
    pass
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `@decorator` | The decorator syntax; equivalent to `func = decorator(func)`. |
| `decorator(func)` | The decorator function; takes the original function as an argument. |
| `wrapper` | The inner function that wraps the original. |
| `*args, **kwargs` | Accepts any arguments passed to the original function. |
| `return wrapper` | Returns the wrapper function, which replaces the original. |

**Syntax Rules**

- The decorator must be a callable that accepts a callable and returns a callable.
- The `@decorator` line must appear immediately above the function definition.
- Multiple decorators are applied bottom-up (the one closest to the function is applied first).
- Decorators are applied at function definition time, not at call time.
- The decorator's return value replaces the original function name.

**Constraints and Limitations**

- Without `functools.wraps`, the decorated function loses its original name and docstring.
- Decorators that take arguments require an extra level of nesting (decorator factory).
- Debugging decorated functions can be harder because tracebacks show the wrapper.

#### Annotated Code Examples

**Example 1: Basic Decorator**

```python
def my_decorator(func):
    def wrapper():
        print("Something is happening before the function is called.")
        func()
        print("Something is happening after the function is called.")
    return wrapper

@my_decorator
def say_hello():
    print("Hello!")

say_hello()
```

**Expected Output:**

```
Something is happening before the function is called.
Hello!
Something is happening after the function is called.
```

**Step-by-Step Breakdown:**

1. `my_decorator` takes `func` (the original function) as an argument.
2. `wrapper` is defined inside `my_decorator` and calls `func()` between two print statements.
3. `@my_decorator` is equivalent to `say_hello = my_decorator(say_hello)`.
4. When `say_hello()` is called, `wrapper` runs instead.

**Why the Output is Produced:**

The decorator replaces `say_hello` with `wrapper`. When `say_hello()` is invoked, it calls `wrapper()`, which prints the "before" message, calls the original `say_hello`, and prints the "after" message.

---

**Example 2: Decorator with Arguments and `functools.wraps`**

```python
from functools import wraps

def logged(func):
    @wraps(func)
    def wrapper(*args, **kwargs):
        print(f"Calling {func.__name__} with {args} and {kwargs}")
        result = func(*args, **kwargs)
        print(f"{func.__name__} returned {result}")
        return result
    return wrapper

@logged
def add(a, b):
    """Add two numbers."""
    return a + b

print(add(3, 5))
print(add.__name__)
print(add.__doc__)
```

**Expected Output:**

```
Calling add with (3, 5) and {}
add returned 8
8
add
Add two numbers.
```

**Step-by-Step Breakdown:**

1. `@wraps(func)` copies `__name__`, `__doc__`, and other metadata from `func` to `wrapper`.
2. `wrapper` accepts `*args` and `**kwargs`, making it flexible.
3. `add(3, 5)` triggers the wrapper, which logs the call and result.

**Why the Output is Produced:**

`functools.wraps` preserves the original function's metadata, so `add.__name__` is `"add"` and `add.__doc__` is `"Add two numbers."` Without `@wraps`, these would be `"wrapper"` and `None`.

#### Real-World Cases

- **Logging**: Logging function calls, arguments, and return values.
- **Timing**: Measuring how long a function takes to execute.
- **Authentication**: Checking user permissions before allowing a function to run.
- **Caching**: Storing results of expensive function calls.

---

### 2. Function Decorators

#### Definitions

**Core Definition**

A function decorator is a decorator applied to a function. It wraps the function with additional behavior, returning a new function that replaces the original.

**Technical Definition**

Function decorators are the most common type of decorator. They accept a function as an argument and return a new function. The `@decorator` syntax is placed above a `def` statement. The decorator is evaluated at function definition time, and the returned callable is bound to the function name. Multiple function decorators can be stacked, applying in nested fashion.

**Beginner-Friendly Explanation**

A function decorator is like adding a filter to a photo. You have a function that does something, and the decorator adds extra effects before or after it runs.

#### Purposes

- To add reusable behavior to functions.
- To implement cross-cutting concerns (logging, timing, caching).
- To enforce pre-conditions and post-conditions.
- To modify function return values.

#### Syntax Rules and Structure

```
@decorator1
@decorator2
def my_function():
    pass
```

This is equivalent to:

```
def my_function():
    pass
my_function = decorator1(decorator2(my_function))
```

**Syntax Rules**

- Decorators are applied bottom-up: `decorator2` is applied first, then `decorator1`.
- Each decorator must accept a callable and return a callable.
- The final result is bound to the function name.

**Constraints and Limitations**

- Stacking many decorators can make debugging difficult.
- Each decorator adds a layer of function call overhead.

#### Annotated Code Examples

**Example 1: Multiple Decorators**

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

**Step-by-Step Breakdown:**

1. `@italic` is applied first: `greet = italic(greet)`.
2. `@bold` is applied second: `greet = bold(greet)`.
3. The final result is `<b><i>Hello</i></b>`.

**Why the Output is Produced:**

Decorators are applied in nested fashion. The function closest to the decorated function is applied first.

#### Real-World Cases

- **Flask Routes**: `@app.route("/")` decorators map URLs to view functions.
- **Django**: `@login_required` decorators enforce authentication.
- **Pytest**: `@pytest.mark.skip` decorators control test execution.

---

### 3. Class Decorators

#### Definitions

**Core Definition**

A class decorator is a decorator applied to a class. It takes a class as an argument and returns a modified class (or a new class). Class decorators were introduced in Python 2.6 via PEP 3129.

**Technical Definition**

Class decorators are applied using the `@decorator` syntax above a class definition. They are evaluated at class definition time and the returned value replaces the class name. Common built-in class decorators include `@dataclass` (from the `dataclasses` module) and `@total_ordering` (from `functools`).

**Beginner-Friendly Explanation**

A class decorator is like a stamp you put on a class. It can add methods, modify attributes, or change how the class works.

#### Purposes

- To add or modify class attributes and methods.
- To automatically generate boilerplate code (e.g., `@dataclass`).
- To implement design patterns like singletons or registries.
- To enforce class-level constraints.

#### Annotated Code Examples

**Example 1: Simple Class Decorator**

```python
def add_greeting(cls):
    cls.greet = lambda self: "Hello from class decorator!"
    return cls

@add_greeting
class Person:
    def __init__(self, name):
        self.name = name

p = Person("Alice")
print(p.greet())
```

**Expected Output:**

```
Hello from class decorator!
```

**Step-by-Step Breakdown:**

1. `add_greeting` takes the class, adds a `greet` method, and returns the class.
2. `@add_greeting` applies the decorator to `Person`.
3. `p.greet()` calls the newly added method.

**Why the Output is Produced:**

The class decorator modifies the class by adding a new method before the class is bound to its name.

#### Real-World Cases

- **@dataclass**: Automatically generates `__init__`, `__repr__`, and `__eq__` methods.
- **@total_ordering**: Fills in missing comparison methods.
- **ORMs**: Some ORMs use class decorators to register models.

---

### 4. Decorator Arguments and Decorator Factories

#### Definitions

**Core Definition**

A decorator factory is a function that returns a decorator. It allows you to pass arguments to a decorator, which are used to configure the decorator's behavior.

**Technical Definition**

A decorator factory is a function that takes configuration arguments and returns a decorator. The returned decorator then takes the function to be decorated. This requires three levels of nesting: the factory, the decorator, and the wrapper. The syntax `@factory(arg)` is equivalent to `func = factory(arg)(func)`.

**Beginner-Friendly Explanation**

A decorator factory is like a factory that makes decorators. You give it some settings (arguments), and it gives you a decorator customized with those settings.

#### Purposes

- To pass arguments to a decorator for configuration.
- To create reusable decorators with customizable behavior.
- To implement decorators that need to know extra information (e.g., log level, retry count).

#### Syntax Rules and Structure

```
def decorator_factory(arg1, arg2):
    def decorator(func):
        def wrapper(*args, **kwargs):
            # Use arg1, arg2
            return func(*args, **kwargs)
        return wrapper
    return decorator

@decorator_factory("value1", "value2")
def my_function():
    pass
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `decorator_factory` | The outermost function that takes configuration arguments. |
| `decorator` | The middle function that takes the function to be decorated. |
| `wrapper` | The innermost function that wraps the original. |

**Syntax Rules**

- The factory must return a decorator.
- The decorator must return a wrapper.
- The wrapper calls the original function.

**Constraints and Limitations**

- Requires an extra layer of nesting compared to simple decorators.
- Can be confusing to write and debug.

#### Annotated Code Examples

**Example 1: Decorator with Arguments**

```python
def repeat(times):
    def decorator(func):
        def wrapper(*args, **kwargs):
            for _ in range(times):
                result = func(*args, **kwargs)
            return result
        return wrapper
    return decorator

@repeat(3)
def say_hello():
    print("Hello!")

say_hello()
```

**Expected Output:**

```
Hello!
Hello!
Hello!
```

**Step-by-Step Breakdown:**

1. `repeat(3)` returns the `decorator` function.
2. `decorator(say_hello)` returns the `wrapper` function.
3. `wrapper` calls `say_hello` three times.

**Why the Output is Produced:**

The decorator factory captures `times = 3` in its closure. The wrapper uses this value to repeat the function call.

#### Real-World Cases

- **Retry Logic**: `@retry(max_attempts=3)` retries a function on failure.
- **Rate Limiting**: `@rate_limit(calls_per_second=10)` limits function calls.
- **Logging**: `@log(level="DEBUG")` logs at a specified level.

---

### 5. `functools.wraps()`

#### Definitions

**Core Definition**

`functools.wraps()` is a decorator that copies metadata (name, docstring, annotations, etc.) from the original function to the wrapper function.

**Technical Definition**

`functools.wraps()` is a convenience function that applies `functools.update_wrapper()` to the wrapper function. It copies `__module__`, `__name__`, `__qualname__`, `__annotations__`, and `__doc__` from the wrapped function to the wrapper, and updates the wrapper's `__dict__` with the wrapped function's `__dict__`. It also sets a `__wrapped__` attribute pointing to the original function.

**Beginner-Friendly Explanation**

When you wrap a function with a decorator, the wrapper replaces the original function. This means the function's name and docstring are lost. `functools.wraps()` fixes this by copying the original function's identity to the wrapper.

#### Purposes

- To preserve the original function's name, docstring, and metadata.
- To enable proper introspection and documentation.
- To support tools like `help()` and `inspect` that rely on metadata.

#### Syntax Rules and Structure

```
from functools import wraps

def decorator(func):
    @wraps(func)
    def wrapper(*args, **kwargs):
        return func(*args, **kwargs)
    return wrapper
```

**Syntax Rules**

- `@wraps(func)` must be applied to the wrapper function.
- It copies `__name__`, `__doc__`, `__module__`, `__qualname__`, `__annotations__`, and `__dict__`.
- It sets `__wrapped__` to the original function.

**Constraints and Limitations**

- `wraps` does not copy `__defaults__`, `__kwdefaults__`, or `__closure__`.
- It only works if the wrapped function has these attributes.

#### Annotated Code Examples

**Example 1: Without and With `wraps`**

```python
from functools import wraps

def decorator_without_wraps(func):
    def wrapper(*args, **kwargs):
        return func(*args, **kwargs)
    return wrapper

def decorator_with_wraps(func):
    @wraps(func)
    def wrapper(*args, **kwargs):
        return func(*args, **kwargs)
    return wrapper

@decorator_without_wraps
def add(a, b):
    """Add two numbers."""
    return a + b

@decorator_with_wraps
def multiply(a, b):
    """Multiply two numbers."""
    return a * b

print("Without wraps:")
print("Name:", add.__name__)
print("Doc:", add.__doc__)
print()
print("With wraps:")
print("Name:", multiply.__name__)
print("Doc:", multiply.__doc__)
```

**Expected Output:**

```
Without wraps:
Name: wrapper
Doc: None

With wraps:
Name: multiply
Doc: Multiply two numbers.
```

**Step-by-Step Breakdown:**

1. Without `@wraps`, `add.__name__` is `"wrapper"` and `add.__doc__` is `None`.
2. With `@wraps`, `multiply.__name__` is `"multiply"` and `multiply.__doc__` is `"Multiply two numbers."`

**Why the Output is Produced:**

`functools.wraps` copies the metadata from the original function to the wrapper, preserving the function's identity.

#### Real-World Cases

- **All decorators**: `@wraps` should be used in every decorator to preserve metadata.
- **Documentation**: Tools like Sphinx use `__doc__` to generate documentation.
- **Debugging**: Tracebacks show the correct function name.

---

### 6. `functools.lru_cache()`

#### Definitions

**Core Definition**

`functools.lru_cache()` is a decorator that caches the results of a function, storing up to `maxsize` most recent calls. When the same arguments are used again, the cached result is returned instead of recomputing.

**Technical Definition**

`lru_cache` implements a Least Recently Used (LRU) cache. It wraps a function with a memoizing callable that saves up to `maxsize` most recent calls. If `maxsize` is `None`, the LRU features are disabled and the cache can grow without bound. The wrapped function is thread-safe and provides `cache_info()` and `cache_clear()` methods.

**Beginner-Friendly Explanation**

`lru_cache` is like a memory for your function. If you ask it the same question twice, it remembers the answer from the first time and gives it to you instantly without recalculating.

#### Purposes

- To speed up expensive or I/O-bound functions.
- To avoid redundant computations.
- To implement memoization easily.

#### Syntax Rules and Structure

```
from functools import lru_cache

@lru_cache(maxsize=128, typed=False)
def expensive_function(arg):
    # Expensive computation
    return result
```

**Component Breakdown**

| Parameter | Description |
|-----------|-------------|
| `maxsize` | Maximum number of cached calls. `None` means unlimited. |
| `typed` | If `True`, arguments of different types are cached separately. |

**Syntax Rules**

- Arguments must be hashable.
- The cache is shared across all calls to the function.
- `cache_info()` returns hits, misses, maxsize, and currsize.
- `cache_clear()` clears the cache.

**Constraints and Limitations**

- Arguments must be hashable.
- The cache persists for the lifetime of the function.
- Memory usage grows with the number of unique calls.

#### Annotated Code Examples

**Example 1: Fibonacci with `lru_cache`**

```python
from functools import lru_cache

@lru_cache(maxsize=None)
def fibonacci(n):
    if n < 2:
        return n
    return fibonacci(n - 1) + fibonacci(n - 2)

print(fibonacci(50))
print(fibonacci.cache_info())
```

**Expected Output:**

```
12586269025
CacheInfo(hits=48, misses=51, maxsize=None, currsize=51)
```

**Step-by-Step Breakdown:**

1. Without caching, `fibonacci(50)` would make billions of recursive calls.
2. With `lru_cache`, each unique `n` is computed only once.
3. `cache_info()` shows 48 hits and 51 misses.

**Why the Output is Produced:**

The cache stores results for each unique `n`. Subsequent calls with the same `n` return the cached result.

#### Real-World Cases

- **Dynamic Programming**: Computing Fibonacci, factorials, or combinations.
- **Web APIs**: Caching API responses.
- **Database Queries**: Caching expensive query results.

---

### 7. `functools.cache()`

#### Definitions

**Core Definition**

`functools.cache()` is a simple, lightweight unbounded cache decorator. It is equivalent to `lru_cache(maxsize=None)` but smaller and faster because it never evicts old values.

**Technical Definition**

Added in Python 3.9, `@cache` is a thin wrapper around a dictionary lookup for function arguments. It is thread-safe and never evicts values. It is smaller and faster than `lru_cache(maxsize=None)` because it does not need to track recency.

**Beginner-Friendly Explanation**

`cache` is like `lru_cache` but simpler: it remembers everything and never forgets. Use it when you have enough memory and want maximum speed.

#### Purposes

- To implement unbounded memoization.
- To speed up functions with repeated calls.
- To simplify code compared to `lru_cache(maxsize=None)`.

#### Syntax Rules and Structure

```
from functools import cache

@cache
def function(arg):
    return result
```

**Syntax Rules**

- Same as `lru_cache(maxsize=None)`.
- No `maxsize` or `typed` parameters.
- Thread-safe.

**Constraints and Limitations**

- Unbounded memory growth.
- Arguments must be hashable.

#### Annotated Code Examples

**Example 1: Factorial with `@cache`**

```python
from functools import cache

@cache
def factorial(n):
    return n * factorial(n - 1) if n else 1

print(factorial(10))
print(factorial(5))
print(factorial(12))
```

**Expected Output:**

```
3628800
120
479001600
```

**Step-by-Step Breakdown:**

1. `factorial(10)` computes 11 recursive calls and caches all results.
2. `factorial(5)` returns the cached result immediately.
3. `factorial(12)` makes only two new recursive calls because `factorial(10)` is cached.

**Why the Output is Produced:**

`@cache` stores all results, so repeated calls with the same or smaller arguments return instantly.

#### Real-World Cases

- **Recursive Algorithms**: Any function with overlapping subproblems.
- **Configuration Loading**: Caching parsed configuration files.
- **Data Processing**: Caching expensive transformations.

---

### 8. `functools.singledispatch()`

#### Definitions

**Core Definition**

`functools.singledispatch()` is a decorator that converts a function into a generic function, allowing different implementations based on the type of the first argument.

**Technical Definition**

`@singledispatch` transforms a function into a generic function whose behavior depends on the type of its first argument. The original function serves as the default implementation (registered for `object`). Additional implementations are registered using the `@function.register` decorator. Dispatch happens on the type of the first argument, using the method resolution order to find the most specific implementation.

**Beginner-Friendly Explanation**

`singledispatch` lets you write one function that behaves differently depending on what type of data you give it. Give it an integer, it does one thing; give it a list, it does another.

#### Purposes

- To implement function overloading based on type.
- To provide specialized implementations for different types.
- To extend functions without modifying existing code.

#### Syntax Rules and Structure

```
from functools import singledispatch

@singledispatch
def fun(arg, verbose=False):
    if verbose:
        print("Let me just say,", end=" ")
    print(arg)

@fun.register
def _(arg: int, verbose=False):
    if verbose:
        print("Strength in numbers, eh?", end=" ")
    print(arg)

@fun.register
def _(arg: list, verbose=False):
    if verbose:
        print("Enumerate this:")
    for i, elem in enumerate(arg):
        print(i, elem)
```

**Syntax Rules**

- The base function is decorated with `@singledispatch`.
- Additional implementations are registered with `@fun.register`.
- Dispatch is on the type of the first argument.
- Type annotations are used to infer the type, or the type can be passed explicitly.

**Constraints and Limitations**

- Dispatch is on the first argument only.
- The base function must be defined first.

#### Annotated Code Examples

**Example 1: Type-Based Dispatch**

```python
from functools import singledispatch

@singledispatch
def describe(arg):
    return f"Object: {arg}"

@describe.register
def _(arg: int):
    return f"Integer: {arg}"

@describe.register
def _(arg: list):
    return f"List with {len(arg)} items"

print(describe(42))
print(describe([1, 2, 3]))
print(describe("hello"))
```

**Expected Output:**

```
Integer: 42
List with 3 items
Object: hello
```

**Step-by-Step Breakdown:**

1. `describe(42)` dispatches to the `int` implementation.
2. `describe([1, 2, 3])` dispatches to the `list` implementation.
3. `describe("hello")` dispatches to the default `object` implementation.

**Why the Output is Produced:**

`singledispatch` selects the most specific registered implementation based on the type of the first argument.

#### Real-World Cases

- **Serialization**: Different serialization logic for different types.
- **Data Validation**: Type-specific validation rules.
- **API Handling**: Different handlers for different input types.

---

### 9. `functools.partial()`

#### Definitions

**Core Definition**

`functools.partial()` is a function that creates a new function with some arguments pre-filled, allowing you to "freeze" some arguments of a function.

**Technical Definition**

`functools.partial(func, /, *args, **keywords)` returns a new `partial` object that behaves like `func` called with the given arguments. Additional arguments can be provided when the partial object is called. If more arguments are supplied, they are appended to the pre-filled arguments. If additional keyword arguments are supplied, they extend and override the pre-filled keywords.

**Beginner-Friendly Explanation**

`partial` lets you create a specialized version of a function. If you have a function `add(a, b)` and you always add 10, you can create `add_ten = partial(add, 10)`. Then `add_ten(5)` gives 15.

#### Purposes

- To create specialized functions from general ones.
- To fix some arguments of a function.
- To simplify callbacks and event handlers.

#### Syntax Rules and Structure

```
from functools import partial

def add(a, b):
    return a + b

add_ten = partial(add, 10)
print(add_ten(5))  # 15
```

**Syntax Rules**

- The first argument is the function to partially apply.
- Additional positional and keyword arguments are pre-filled.
- When called, extra arguments are appended to positional arguments and merged with keyword arguments.

**Constraints and Limitations**

- `partial` does not preserve metadata (use `functools.update_wrapper` if needed).
- Pre-filled arguments cannot be overridden by positional arguments.

#### Annotated Code Examples

**Example 1: Partial for Specialized Functions**

```python
from functools import partial

def power(base, exponent):
    return base ** exponent

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

**Step-by-Step Breakdown:**

1. `partial(power, exponent=2)` creates a function that squares its argument.
2. `partial(power, exponent=3)` creates a function that cubes its argument.

**Why the Output is Produced:**

`partial` pre-fills the `exponent` argument, so `square(5)` calls `power(5, exponent=2)`.

#### Real-World Cases

- **Callbacks**: Creating callbacks with pre-configured behavior.
- **Event Handlers**: Binding event handlers with specific data.
- **Function Factories**: Creating specialized functions from templates.

---

### 10. `functools.reduce()`

#### Definitions

**Core Definition**

`functools.reduce()` applies a binary function cumulatively to the elements of an iterable, reducing it to a single value.

**Technical Definition**

`functools.reduce(function, iterable, initial)` applies `function` to the first two elements, then to the result and the next element, and so on. If `initial` is provided, it is placed before the first element. The function must accept two arguments.

**Beginner-Friendly Explanation**

`reduce` is like folding a list into a single value. For example, adding all numbers in a list by repeatedly adding the next number to the running total.

#### Purposes

- To compute aggregates (sum, product, maximum, minimum).
- To combine elements of a sequence.
- To implement functional programming patterns.

#### Syntax Rules and Structure

```
from functools import reduce

result = reduce(function, iterable, initial)
```

**Syntax Rules**

- `function` must accept two arguments.
- If `initial` is provided, it is used as the starting value.
- If the iterable is empty and no initial value is given, a `TypeError` is raised.

**Constraints and Limitations**

- Often less readable than explicit loops or built-in functions like `sum()`.
- Not available as a built-in in Python 3 (must import from `functools`).

#### Annotated Code Examples

**Example 1: Sum and Product with `reduce`**

```python
from functools import reduce

numbers = [1, 2, 3, 4, 5]

sum_all = reduce(lambda x, y: x + y, numbers)
product_all = reduce(lambda x, y: x * y, numbers)

print("Sum:", sum_all)
print("Product:", product_all)
```

**Expected Output:**

```
Sum: 15
Product: 120
```

**Step-by-Step Breakdown:**

1. `reduce(lambda x, y: x + y, [1, 2, 3, 4, 5])` computes `((((1+2)+3)+4)+5) = 15`.
2. `reduce(lambda x, y: x * y, [1, 2, 3, 4, 5])` computes `1*2*3*4*5 = 120`.

**Why the Output is Produced:**

`reduce` applies the function cumulatively, reducing the iterable to a single value.

#### Real-World Cases

- **Aggregation**: Computing sums, products, and other aggregates.
- **Functional Pipelines**: Combining transformations.
- **String Concatenation**: Joining strings with a separator.

---

### 11. `functools.total_ordering()`

#### Definitions

**Core Definition**

`functools.total_ordering()` is a class decorator that fills in missing rich comparison methods based on the ones you define.

**Technical Definition**

`@total_ordering` is a class decorator that, given a class defining one or more rich comparison ordering methods (`__lt__`, `__le__`, `__gt__`, `__ge__`), supplies the rest. The class must define `__eq__` and one of the ordering methods. The decorator fills in the remaining methods automatically.

**Beginner-Friendly Explanation**

If you define `__eq__` and `__lt__`, `total_ordering` automatically gives you `__le__`, `__gt__`, and `__ge__`. You don't have to write them all.

#### Purposes

- To reduce boilerplate when implementing comparison methods.
- To ensure consistent ordering behavior.
- To simplify class design.

#### Syntax Rules and Structure

```
from functools import total_ordering

@total_ordering
class MyClass:
    def __eq__(self, other):
        ...
    def __lt__(self, other):
        ...
```

**Syntax Rules**

- The class must define `__eq__` and at least one of `__lt__`, `__le__`, `__gt__`, `__ge__`.
- The decorator adds the remaining comparison methods.

**Constraints and Limitations**

- Adds some performance overhead due to additional method calls.
- The derived methods are not as efficient as hand-written ones.

#### Annotated Code Examples

**Example 1: Total Ordering**

```python
from functools import total_ordering

@total_ordering
class Student:
    def __init__(self, name, grade):
        self.name = name
        self.grade = grade

    def __eq__(self, other):
        return self.grade == other.grade

    def __lt__(self, other):
        return self.grade < other.grade

s1 = Student("Alice", 85)
s2 = Student("Bob", 92)

print(s1 < s2)   # True
print(s1 <= s2)  # True
print(s1 > s2)   # False
print(s1 >= s2)  # False
print(s1 == s2)  # False
```

**Expected Output:**

```
True
True
False
False
False
```

**Step-by-Step Breakdown:**

1. Only `__eq__` and `__lt__` are defined.
2. `@total_ordering` fills in `__le__`, `__gt__`, and `__ge__`.

**Why the Output is Produced:**

The decorator uses the defined methods to derive the missing ones.

#### Real-World Cases

- **Sorting Objects**: Enabling sorting and comparison of custom objects.
- **Data Classes**: Adding ordering to data classes.
- **Domain Models**: Comparing entities by specific fields.

---

### 12. `functools.cached_property()`

#### Definitions

**Core Definition**

`functools.cached_property()` is a decorator that transforms a method into a property whose value is computed once and then cached as an instance attribute.

**Technical Definition**

`@cached_property` is similar to `@property` but adds caching. The value is computed on first access and then stored in the instance's `__dict__`. Subsequent accesses return the cached value. It is useful for expensive computed properties of instances that are otherwise effectively immutable.

**Beginner-Friendly Explanation**

`cached_property` is like a property that remembers its value. The first time you access it, it computes the value and saves it. The next time, it just returns the saved value without recomputing.

#### Purposes

- To cache expensive computed properties.
- To avoid recomputing values that don't change.
- To improve performance for immutable instances.

#### Syntax Rules and Structure

```
from functools import cached_property

class DataSet:
    def __init__(self, data):
        self._data = data

    @cached_property
    def stdev(self):
        return statistics.stdev(self._data)
```

**Syntax Rules**

- Can only be used on methods that take no arguments (other than `self`).
- The cached value is stored in the instance's `__dict__`.
- The cache can be cleared by deleting the attribute.

**Constraints and Limitations**

- Requires a mutable `__dict__` on the instance.
- Not suitable for classes with `__slots__` that don't include `__dict__`.
- Not thread-safe by default.

#### Annotated Code Examples

**Example 1: Cached Property**

```python
from functools import cached_property
import statistics

class DataSet:
    def __init__(self, numbers):
        self._data = tuple(numbers)

    @cached_property
    def stdev(self):
        print("Computing stdev...")
        return statistics.stdev(self._data)

ds = DataSet([1, 2, 3, 4, 5])
print(ds.stdev)  # Computes
print(ds.stdev)  # Returns cached value
```

**Expected Output:**

```
Computing stdev...
1.5811388300841898
1.5811388300841898
```

**Step-by-Step Breakdown:**

1. First access to `ds.stdev` computes and caches the value.
2. Second access returns the cached value without recomputing.

**Why the Output is Produced:**

The first access triggers the computation and stores the result. Subsequent accesses find the cached value in `__dict__`.

#### Real-World Cases

- **Data Analysis**: Computing statistics on datasets.
- **Web Applications**: Caching computed user profiles or recommendations.
- **Scientific Computing**: Caching expensive simulations.

---

### 13. Common Decorators

#### Definitions

**Core Definition**

Common decorators are built-in or standard-library decorators that are widely used in Python programming. These include `@property`, `@staticmethod`, `@classmethod`, `@abstractmethod`, `@dataclass`, `@contextmanager`, and `@atexit.register`.

**Technical Definition**

These decorators are provided by Python's standard library and serve specific purposes. `@property` creates managed attributes. `@staticmethod` and `@classmethod` define methods that don't require an instance. `@abstractmethod` declares abstract methods in ABCs. `@dataclass` generates boilerplate methods for classes. `@contextmanager` creates context managers from generator functions. `@atexit.register` registers cleanup functions.

**Beginner-Friendly Explanation**

These are the decorators that come with Python. They solve common problems: making attributes read-only, creating methods that don't need an instance, defining abstract classes, reducing boilerplate, and managing resources.

#### 13.1 `@property`

**Core Definition**

`@property` is a decorator that transforms a method into a read-only attribute. It allows you to define getter, setter, and deleter methods for managed attributes.

**Technical Definition**

`property` is a built-in class that implements managed attributes. When used as a decorator, `@property` creates a property object whose `fget` is the decorated method. Additional methods can be attached using `@x.setter` and `@x.deleter`.

**Purposes**

- To create read-only attributes.
- To add validation logic to attribute access.
- To compute attribute values dynamically.
- To maintain a clean public API while controlling internal access.

**Syntax Rules and Structure**

```
class MyClass:
    def __init__(self):
        self._x = 0

    @property
    def x(self):
        return self._x

    @x.setter
    def x(self, value):
        if value < 0:
            raise ValueError("x must be non-negative")
        self._x = value
```

**Annotated Code Examples**

```python
class Temperature:
    def __init__(self, celsius):
        self._celsius = celsius

    @property
    def celsius(self):
        return self._celsius

    @celsius.setter
    def celsius(self, value):
        if value < -273.15:
            raise ValueError("Temperature below absolute zero!")
        self._celsius = value

    @property
    def fahrenheit(self):
        return self._celsius * 9/5 + 32

t = Temperature(25)
print(t.celsius)    # 25
print(t.fahrenheit) # 77.0

t.celsius = 30
print(t.fahrenheit) # 86.0

try:
    t.celsius = -300
except ValueError as e:
    print(f"Error: {e}")
```

**Expected Output:**

```
25
77.0
86.0
Error: Temperature below absolute zero!
```

**Real-World Cases**

- **Data Validation**: Validating attribute values before assignment.
- **Computed Properties**: Deriving values from other attributes.
- **API Design**: Providing read-only access to internal data.

#### 13.2 `@staticmethod`

**Core Definition**

`@staticmethod` is a decorator that defines a method that belongs to a class but does not receive an implicit first argument (`self` or `cls`).

**Technical Definition**

`@staticmethod` converts a method into a static method. Static methods are functions that are part of the class but do not operate on instances or the class itself. They can be called on the class or on an instance, but the instance is ignored.

**Purposes**

- To define utility functions that logically belong to a class.
- To avoid requiring an instance for functions that don't use `self`.
- To organize related functions within a class namespace.

**Syntax Rules and Structure**

```
class MyClass:
    @staticmethod
    def utility_function(arg):
        return arg * 2
```

**Annotated Code Examples**

```python
class MathUtils:
    @staticmethod
    def add(a, b):
        return a + b

    @staticmethod
    def multiply(a, b):
        return a * b

print(MathUtils.add(3, 5))
print(MathUtils.multiply(3, 5))

# Can also be called on an instance
m = MathUtils()
print(m.add(10, 20))
```

**Expected Output:**

```
8
15
30
```

**Real-World Cases**

- **Utility Functions**: Grouping related helper functions in a class.
- **Alternative Constructors**: Static methods that create instances.
- **Pure Functions**: Functions that don't depend on class or instance state.

#### 13.3 `@classmethod`

**Core Definition**

`@classmethod` is a decorator that defines a method that receives the class (`cls`) as its first argument instead of the instance (`self`).

**Technical Definition**

`@classmethod` converts a method into a class method. Class methods receive the class as the implicit first argument. They can be called on the class or on an instance. They are commonly used for alternative constructors.

**Purposes**

- To create alternative constructors.
- To access or modify class-level state.
- To implement factory methods.

**Syntax Rules and Structure**

```
class MyClass:
    @classmethod
    def from_string(cls, string):
        # Alternative constructor
        return cls(...)
```

**Annotated Code Examples**

```python
class Person:
    def __init__(self, name, age):
        self.name = name
        self.age = age

    @classmethod
    def from_birth_year(cls, name, birth_year):
        age = 2026 - birth_year
        return cls(name, age)

    def __repr__(self):
        return f"Person({self.name}, {self.age})"

p1 = Person("Alice", 30)
p2 = Person.from_birth_year("Bob", 1996)
print(p1)
print(p2)
```

**Expected Output:**

```
Person(Alice, 30)
Person(Bob, 30)
```

**Real-World Cases**

- **Alternative Constructors**: `dict.fromkeys()`, `datetime.fromtimestamp()`.
- **Factory Methods**: Creating instances from different data formats.
- **Class-Level Configuration**: Accessing class variables.

#### 13.4 `@abstractmethod`

**Core Definition**

`@abstractmethod` is a decorator used to declare abstract methods in abstract base classes (ABCs). Abstract methods must be overridden by subclasses.

**Technical Definition**

`@abstractmethod` is provided by the `abc` module. It marks a method as abstract, meaning it has no implementation in the base class and must be implemented by any concrete subclass. A class containing abstract methods cannot be instantiated unless all abstract methods are overridden. The metaclass must be `ABCMeta` or derived from it.

**Purposes**

- To define interfaces that subclasses must implement.
- To enforce contracts in class hierarchies.
- To create abstract base classes for polymorphic designs.

**Syntax Rules and Structure**

```
from abc import ABC, abstractmethod

class MyABC(ABC):
    @abstractmethod
    def my_method(self):
        pass
```

**Annotated Code Examples**

```python
from abc import ABC, abstractmethod

class Shape(ABC):
    @abstractmethod
    def area(self):
        pass

    @abstractmethod
    def perimeter(self):
        pass

class Rectangle(Shape):
    def __init__(self, width, height):
        self.width = width
        self.height = height

    def area(self):
        return self.width * self.height

    def perimeter(self):
        return 2 * (self.width + self.height)

class Circle(Shape):
    def __init__(self, radius):
        self.radius = radius

    def area(self):
        return 3.14159 * self.radius ** 2

    def perimeter(self):
        return 2 * 3.14159 * self.radius

# shape = Shape()  # TypeError: Can't instantiate abstract class
rect = Rectangle(5, 3)
circle = Circle(4)
print(rect.area(), rect.perimeter())
print(f"{circle.area():.2f}", f"{circle.perimeter():.2f}")
```

**Expected Output:**

```
15 16
50.27 25.13
```

**Real-World Cases**

- **Plugin Systems**: Defining interfaces for plugins.
- **Framework Design**: Enforcing method implementation in subclasses.
- **Design Patterns**: Implementing the Template Method pattern.

#### 13.5 `@dataclass`

**Core Definition**

`@dataclass` is a decorator that automatically generates special methods like `__init__`, `__repr__`, and `__eq__` for classes that primarily store data.

**Technical Definition**

Introduced in Python 3.7 (PEP 557), `@dataclass` inspects the class for fields defined as class variables with type annotations. It then generates `__init__`, `__repr__`, and `__eq__` methods automatically. Parameters control which methods are generated and their behavior.

**Purposes**

- To reduce boilerplate code for data-holding classes.
- To ensure consistent `__init__`, `__repr__`, and `__eq__` implementations.
- To provide a clean, declarative syntax for data structures.

**Syntax Rules and Structure**

```
from dataclasses import dataclass

@dataclass
class Person:
    name: str
    age: int
    city: str = "Unknown"
```

**Annotated Code Examples**

```python
from dataclasses import dataclass, field

@dataclass
class InventoryItem:
    name: str
    unit_price: float
    quantity_on_hand: int = 0

    def total_cost(self) -> float:
        return self.unit_price * self.quantity_on_hand

item = InventoryItem("Widget", 9.99, 10)
print(item)
print(item.total_cost())

item2 = InventoryItem("Gadget", 19.99)
print(item2)
```

**Expected Output:**

```
InventoryItem(name='Widget', unit_price=9.99, quantity_on_hand=10)
99.9
InventoryItem(name='Gadget', unit_price=19.99, quantity_on_hand=0)
```

**Real-World Cases**

- **Data Transfer Objects (DTOs)**: Simple classes for moving data.
- **Configuration Objects**: Structured configuration.
- **API Models**: Representing API request/response data.

#### 13.6 `@contextmanager`

**Core Definition**

`@contextmanager` is a decorator from the `contextlib` module that creates a context manager from a generator function, allowing the use of the `with` statement.

**Technical Definition**

`@contextmanager` transforms a generator function into a context manager factory. The generator must yield exactly once. Code before the `yield` is the `__enter__` part; code after is the `__exit__` part. If an exception occurs in the `with` block, it is raised at the `yield` point.

**Purposes**

- To create context managers without writing a class.
- To manage resources (files, connections, locks) cleanly.
- To ensure cleanup code runs even if exceptions occur.

**Syntax Rules and Structure**

```
from contextlib import contextmanager

@contextmanager
def managed_resource(*args, **kwargs):
    resource = acquire(*args, **kwargs)
    try:
        yield resource
    finally:
        release(resource)
```

**Annotated Code Examples**

```python
from contextlib import contextmanager
import time

@contextmanager
def timer(label):
    start = time.perf_counter()
    try:
        yield
    finally:
        end = time.perf_counter()
        print(f"{label}: {end - start:.4f} seconds")

with timer("Sleeping"):
    time.sleep(0.5)

with timer("Computing"):
    total = sum(range(1000000))
```

**Expected Output (approximate):**

```
Sleeping: 0.5002 seconds
Computing: 0.0150 seconds
```

**Real-World Cases**

- **File Management**: Opening and closing files safely.
- **Database Connections**: Ensuring connections are closed.
- **Locking**: Acquiring and releasing locks.
- **Timing**: Measuring code execution time.

#### 13.7 `@atexit.register`

**Core Definition**

`@atexit.register` is a decorator from the `atexit` module that registers a function to be called when the program terminates normally.

**Technical Definition**

`atexit.register(func)` registers `func` to be executed at interpreter termination. The function returns `func`, allowing it to be used as a decorator. Functions are called in reverse order of registration (LIFO). Registered functions are not called if the program is killed by a signal or calls `os._exit()`.

**Purposes**

- To perform cleanup operations on program exit.
- To save state or flush buffers.
- To close resources that weren't managed with context managers.

**Syntax Rules and Structure**

```
import atexit

@atexit.register
def cleanup():
    print("Cleaning up...")
```

**Annotated Code Examples**

```python
import atexit

@atexit.register
def goodbye():
    print("Goodbye, world!")

@atexit.register
def save_state():
    print("Saving state...")

print("Program is running...")
```

**Expected Output:**

```
Program is running...
Saving state...
Goodbye, world!
```

**Step-by-Step Breakdown:**

1. `goodbye` is registered first.
2. `save_state` is registered second.
3. At exit, functions run in reverse order: `save_state` then `goodbye`.

**Why the Output is Produced:**

`atexit` runs handlers in LIFO order (last registered, first executed).

**Real-World Cases**

- **Resource Cleanup**: Closing files, sockets, or database connections.
- **State Persistence**: Saving application state before exit.
- **Logging**: Writing final log entries.

---

### 14. Decorator Use Cases

#### Definitions

**Core Definition**

Decorator use cases are the practical scenarios where decorators provide elegant solutions. These include logging, timing, caching, authentication, validation, retry logic, and more.

**Technical Definition**

Decorators are widely used in Python to implement cross-cutting concerns—functionality that spans multiple parts of a program. They are idiomatic in web frameworks (Flask, Django), testing frameworks (pytest), and standard library modules (`functools`, `contextlib`, `dataclasses`).

**Beginner-Friendly Explanation**

Decorators are useful whenever you want to add the same behavior to multiple functions without repeating code. For example, if you want to log every function call, you write one decorator and apply it to all functions.

#### Common Use Cases

| Use Case | Description | Example |
|----------|-------------|---------|
| **Logging** | Log function calls, arguments, and results. | `@logged` |
| **Timing** | Measure execution time. | `@timer` |
| **Caching** | Store results of expensive computations. | `@lru_cache` |
| **Authentication** | Check user permissions. | `@login_required` |
| **Validation** | Validate input arguments. | `@validate` |
| **Retry** | Retry failed operations. | `@retry(max_attempts=3)` |
| **Rate Limiting** | Limit call frequency. | `@rate_limit` |
| **Deprecation** | Warn about deprecated functions. | `@deprecated` |
| **Debugging** | Print function arguments and return values. | `@debug` |
| **Memoization** | Cache function results. | `@cache` |

#### Annotated Code Examples

**Example 1: Timing Decorator**

```python
import time
from functools import wraps

def timer(func):
    @wraps(func)
    def wrapper(*args, **kwargs):
        start = time.perf_counter()
        result = func(*args, **kwargs)
        end = time.perf_counter()
        print(f"{func.__name__} took {end - start:.4f} seconds")
        return result
    return wrapper

@timer
def slow_function():
    time.sleep(0.1)
    return "Done"

print(slow_function())
```

**Expected Output (approximate):**

```
slow_function took 0.1002 seconds
Done
```

**Step-by-Step Breakdown:**

1. `timer` wraps `slow_function`.
2. `wrapper` records the start time, calls the function, records the end time, and prints the duration.
3. `@wraps` preserves the original function's metadata.

**Why the Output is Produced:**

The decorator adds timing behavior around the original function without modifying its code.

---

**Example 2: Retry Decorator**

```python
import time
from functools import wraps

def retry(max_attempts=3, delay=1):
    def decorator(func):
        @wraps(func)
        def wrapper(*args, **kwargs):
            for attempt in range(1, max_attempts + 1):
                try:
                    return func(*args, **kwargs)
                except Exception as e:
                    print(f"Attempt {attempt} failed: {e}")
                    if attempt < max_attempts:
                        time.sleep(delay)
            raise Exception(f"All {max_attempts} attempts failed")
        return wrapper
    return decorator

@retry(max_attempts=3, delay=0.1)
def unstable_function():
    import random
    if random.random() < 0.7:
        raise ValueError("Random failure")
    return "Success"

try:
    print(unstable_function())
except Exception as e:
    print(f"Final error: {e}")
```

**Expected Output (varies):**

```
Attempt 1 failed: Random failure
Attempt 2 failed: Random failure
Success
```

**Real-World Cases**

- **Web Scraping**: Retrying failed HTTP requests.
- **Database Operations**: Retrying transient failures.
- **API Calls**: Handling rate limits and timeouts.

---

### 15. Decorator Best Practices

#### Definitions

**Core Definition**

Decorator best practices are guidelines for writing clean, maintainable, and correct decorators.

**Technical Definition**

Best practices include always using `functools.wraps`, keeping decorators simple and focused, using `*args` and `**kwargs` for flexibility, documenting decorator behavior, and being aware of decorator stacking order. Decorators should be used for cross-cutting concerns, not for business logic.

**Beginner-Friendly Explanation**

Best practices are good habits that make your decorators easy to use and understand. They include preserving function metadata, keeping decorators simple, and documenting what they do.

#### General Guidelines

```python
from functools import wraps

def good_decorator(func):
    @wraps(func)  # Always preserve metadata
    def wrapper(*args, **kwargs):  # Flexible arguments
        # Before logic
        result = func(*args, **kwargs)
        # After logic
        return result
    return wrapper
```

#### Best Practices Table

| Practice | Description |
|----------|-------------|
| Always use `@functools.wraps` | Preserve the original function's name, docstring, and metadata. |
| Use `*args` and `**kwargs` | Make decorators work with any function signature. |
| Keep decorators simple | One concern per decorator; avoid complex logic. |
| Document decorators | Explain what the decorator does and any side effects. |
| Be aware of stacking order | Decorators apply bottom-up; order matters. |
| Don't overuse decorators | Too many decorators make code hard to debug. |
| Test decorators | Write unit tests for decorators. |
| Use decorator factories for arguments | Pass configuration via factory functions. |
| Preserve type annotations | Consider using `typing` to annotate decorators. |
| Handle exceptions properly | Don't suppress exceptions unless intentional. |

#### Annotated Code Examples

**Example 1: Well-Written Decorator**

```python
from functools import wraps
import logging

logging.basicConfig(level=logging.INFO)

def log_calls(func):
    """Log function calls and their return values."""
    @wraps(func)
    def wrapper(*args, **kwargs):
        logging.info(f"Calling {func.__name__}")
        result = func(*args, **kwargs)
        logging.info(f"{func.__name__} returned {result}")
        return result
    return wrapper

@log_calls
def add(a, b):
    """Add two numbers."""
    return a + b

print(add(3, 5))
print(add.__name__)
print(add.__doc__)
```

**Expected Output:**

```
INFO:root:Calling add
INFO:root:add returned 8
8
add
Add two numbers.
```

**Step-by-Step Breakdown:**

1. `@wraps(func)` preserves `__name__` and `__doc__`.
2. `*args, **kwargs` makes the decorator flexible.
3. The decorator has a docstring explaining its purpose.
4. Logging records the call and return value.

**Why the Output is Produced:**

The decorator adds logging behavior while preserving the original function's identity.

---

**Example 2: Common Pitfall — Forgetting `wraps`**

```python
def bad_decorator(func):
    def wrapper(*args, **kwargs):
        return func(*args, **kwargs)
    return wrapper

@bad_decorator
def my_function():
    """My docstring."""
    pass

print(my_function.__name__)  # wrapper
print(my_function.__doc__)   # None
```

**Expected Output:**

```
wrapper
None
```

**Why the Output is Produced:**

Without `@wraps`, the wrapper function replaces the original, losing its metadata.

---

## References

- PEP 318 – Decorators for Functions and Methods - https://peps.python.org/pep-0318/
- PEP 3129 – Class Decorators - https://peps.python.org/pep-3129/
- Python Official Documentation – Function Definitions (Decorators) - https://docs.python.org/3/reference/compound_stmts.html#function-definitions
- Python Official Documentation – `functools` — Higher-order functions and operations on callable objects - https://docs.python.org/3/library/functools.html
- Python Official Documentation – `functools.wraps` - https://docs.python.org/3/library/functools.html#functools.wraps
- Python Official Documentation – `functools.lru_cache` - https://docs.python.org/3/library/functools.html#functools.lru_cache
- Python Official Documentation – `functools.cache` - https://docs.python.org/3/library/functools.html#functools.cache
- Python Official Documentation – `functools.singledispatch` - https://docs.python.org/3/library/functools.html#functools.singledispatch
- Python Official Documentation – `functools.partial` - https://docs.python.org/3/library/functools.html#functools.partial
- Python Official Documentation – `functools.reduce` - https://docs.python.org/3/library/functools.html#functools.reduce
- Python Official Documentation – `functools.total_ordering` - https://docs.python.org/3/library/functools.html#functools.total_ordering
- Python Official Documentation – `functools.cached_property` - https://docs.python.org/3/library/functools.html#functools.cached_property
- Python Official Documentation – `property` - https://docs.python.org/3/library/functions.html#property
- Python Official Documentation – `staticmethod` - https://docs.python.org/3/library/functions.html#staticmethod
- Python Official Documentation – `classmethod` - https://docs.python.org/3/library/functions.html#classmethod
- Python Official Documentation – `abc.abstractmethod` - https://docs.python.org/3/library/abc.html#abc.abstractmethod
- Python Official Documentation – `dataclasses` — Data Classes - https://docs.python.org/3/library/dataclasses.html
- Python Official Documentation – `contextlib.contextmanager` - https://docs.python.org/3/library/contextlib.html#contextlib.contextmanager
- Python Official Documentation – `atexit` — Exit handlers - https://docs.python.org/3/library/atexit.html
- Real Python – Python Decorators: The Complete Guide - https://realpython.com/primer-on-python-decorators/
- Real Python – Python Decorators Cheat Sheet - https://realpython.com/python-decorators-cheat-sheet/
- Real Python – `functools` — The Complete Guide - https://realpython.com/python-functools/
- Real Python – `functools.lru_cache()` - https://realpython.com/lru-cache-python/
- Real Python – `functools.singledispatch()` - https://realpython.com/python-singledispatch/
- Real Python – Python's `property()` - https://realpython.com/python-property/
- Real Python – Python's `@staticmethod` and `@classmethod` - https://realpython.com/instance-class-and-static-methods-demystified/
- Real Python – Python's `@dataclass` - https://realpython.com/python-data-classes/
- Real Python – Context Managers and Python's `with` Statement - https://realpython.com/python-with-statement/
- Python Wiki – TimeComplexity - https://wiki.python.org/moin/TimeComplexity
- Microsoft Learn – Use Decorators in Python - https://learn.microsoft.com/en-us/training/modules/python-decorators/
- GeeksforGeeks – Decorators in Python - https://www.geeksforgeeks.org/decorators-in-python/
- GeeksforGeeks – `functools` Module in Python - https://www.geeksforgeeks.org/python-functools-module/