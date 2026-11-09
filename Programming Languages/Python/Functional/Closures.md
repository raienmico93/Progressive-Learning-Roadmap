# Python Closures: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**

A closure is a nested function that retains access to variables from its enclosing function's scope, even after the enclosing function has finished executing. It "closes over" free variables from its lexical environment, preserving them for later use.

**Technical Definition**

In Python, a closure is a function object that has access to variables in its enclosing (containing) lexical scope, even when the function is called outside that scope. Formally, a closure is created when a nested function references a variable that is not bound in its local scope and not in the global scope — this variable is called a **free variable**. Python implements closures using **cell objects**, which are containers for free variables located in a function's `__closure__` attribute. The `co_freevars` attribute of a function's code object lists the names of free variables, and `__closure__` holds the corresponding cell objects. The feature was introduced by PEP 227 (Statically Nested Scopes), which added lexical scoping to Python 2.1/2.2.

**Beginner-Friendly Explanation**

A closure is like a backpack that a function carries with it. When you define a function inside another function, and the inner function uses a variable from the outer function, the inner function packs that variable into its backpack. Even after the outer function finishes and its local variables would normally disappear, the inner function still has them in its backpack. So when you call the inner function later, it remembers those values.

### Key Characteristics

- **Nested Function**: A closure always involves a function defined inside another function (a nested function).
- **Free Variables**: The inner function references at least one variable from the enclosing function's local scope.
- **Persistent State**: The free variables survive after the enclosing function returns, preserving their last values.
- **Lexical Scoping**: Variable resolution follows the LEGB rule, with the enclosing scope (E) providing the free variables.
- **Late Binding**: Free variables are resolved at call time, not at definition time — this is the source of the classic closure pitfall.
- **Cell Objects**: CPython implements closures using cell objects, accessible via the `__closure__` attribute.
- **`nonlocal` Support**: Python 3 provides the `nonlocal` keyword to rebind free variables from the inner function.

### Prerequisites

- Basic understanding of Python functions and the `def` keyword.
- Familiarity with nested functions and function scope (LEGB rule).
- Knowledge of function parameters, arguments, and return values.
- Awareness of first-class functions and higher-order functions.

### Related Programming Areas

- **Decorators**: Closures are the foundation of Python decorators.
- **Functional Programming**: Closures enable stateful functions without classes.
- **Factory Functions**: Functions that generate specialized functions using closures.
- **Callbacks**: Closures provide private state for callback functions.
- **Metaprogramming**: Closures enable dynamic function generation and code generation.

---

## Core Concepts / Features

---

### 1. Closures

#### Definitions

**Core Definition**

A closure is a function object that has access to variables in its enclosing lexical scope, even when the function is called outside that scope. In simpler terms, a closure "closes over" the free variables from its enclosing scope, preserving access to them.

**Technical Definition**

A closure occurs when a nested function references variables from its enclosing function. The nested function "closes over" the data, maintaining a reference to it. This data is stored in what's called the function's cell, and remains accessible even after the outer function has finished executing. The closure holds a tuple of cells, one for each free variable (named in `co_freevars`), accessible through the function's `__closure__` attribute.

**Beginner-Friendly Explanation**

Imagine you have a function that creates other functions. The inner function uses some variables from the outer function. When the outer function is done and returns the inner function, those variables would normally be gone. But the inner function has "remembered" them — it has closed over them. That's a closure.

#### Purposes

- To create functions with private, persistent state without using classes.
- To enable factory functions that generate specialized functions.
- To support decorators that wrap functions with additional behavior.
- To implement data hiding and encapsulation.
- To create callbacks that remember their creation context.

#### Syntax Rules and Structure

**Complete General Syntax**

```
def outer_function(free_var):
    """Enclose the value of free_var in the closure scope."""
    def inner_function(arg):
        # free_var is a free variable captured from the enclosing scope
        return free_var + arg
    return inner_function

# Create a closure
closure_instance = outer_function(5)
result = closure_instance(3)
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `outer_function` | The enclosing function that defines free variables. |
| `free_var` | A variable in the enclosing scope that will be captured. |
| `inner_function` | The nested function that references `free_var`. |
| `return inner_function` | Returns the nested function, creating the closure. |
| `closure_instance` | The closure object that remembers `free_var`. |

**Syntax Rules**

- A closure requires a nested function (a function defined inside another function).
- The nested function must reference at least one variable from the enclosing function's scope.
- The enclosing function must return the nested function.
- The free variable is captured by reference, not by value.
- The free variable persists as long as the closure exists.
- To rebind a free variable from the inner function, use the `nonlocal` keyword.

**Constraints and Limitations**

- Free variables are resolved at call time (late binding), which can cause unexpected behavior in loops.
- Closures cannot be pickled if they capture unpicklable objects.
- Excessive use of closures can lead to memory leaks if they hold references to large objects.
- Debugging closures can be harder because tracebacks may not show meaningful names.

#### Annotated Code Examples

**Example 1: Basic Closure**

```python
def make_multiplier(factor):
    """Return a function that multiplies its argument by factor."""
    def multiply(x):
        # 'factor' is a free variable captured from the enclosing scope
        return x * factor
    return multiply

# Create two closures with different free variables
double = make_multiplier(2)
triple = make_multiplier(3)

print("Double 5:", double(5))
print("Triple 5:", triple(5))

# Inspect the closure
print("Free variables:", double.__code__.co_freevars)
print("Closure cells:", double.__closure__)
print("Cell contents:", double.__closure__[0].cell_contents)
```

**Expected Output:**

```
Double 5: 10
Triple 5: 15
Free variables: ('factor',)
Closure cells: (<cell at 0x...: int object at 0x...>,)
Cell contents: 2
```

**Step-by-Step Breakdown:**

1. `make_multiplier(2)` creates a local scope with `factor = 2`.
2. `multiply` references `factor` from its enclosing scope, making it a free variable.
3. `return multiply` returns the nested function, which carries the closure.
4. `double(5)` calls the closure, using the captured `factor = 2`.
5. `double.__code__.co_freevars` shows the free variable name.
6. `double.__closure__` holds the cell object containing `2`.

**Why the Output is Produced:**

The inner function `multiply` closes over the `factor` variable. Each call to `make_multiplier` creates a new enclosing scope with its own `factor`, and the returned closure remembers that specific value.

---

**Example 2: Closure with `nonlocal` to Modify State**

```python
def make_counter(start=0):
    """Return a counter function that increments on each call."""
    count = start

    def increment(step=1):
        nonlocal count  # Declare that 'count' is in the enclosing scope
        count += step
        return count

    return increment

# Create a counter starting at 10
counter = make_counter(10)
print(counter())    # 11
print(counter(5))   # 16
print(counter())    # 17

# Each counter has independent state
another = make_counter(0)
print(another())    # 1
print(counter())    # 18
```

**Expected Output:**

```
11
16
17
1
18
```

**Step-by-Step Breakdown:**

1. `make_counter(10)` creates an enclosing scope with `count = 10`.
2. `increment` uses `nonlocal count` to modify the enclosing `count`.
3. `counter()` increments `count` by 1, returning 11.
4. `counter(5)` increments by 5, returning 16.
5. `another = make_counter(0)` creates a separate closure with its own `count`.
6. `another()` returns 1; `counter()` continues from 18.

**Why the Output is Produced:**

`nonlocal` allows the inner function to rebind the enclosing `count`. Each closure instance has its own independent `count` because each call to `make_counter` creates a new enclosing scope.

---

**Example 3: Closure vs Class**

```python
# Closure approach
def make_account(balance):
    def deposit(amount):
        nonlocal balance
        balance += amount
        return balance

    def withdraw(amount):
        nonlocal balance
        balance -= amount
        return balance

    return deposit, withdraw

# Class approach
class Account:
    def __init__(self, balance):
        self.balance = balance

    def deposit(self, amount):
        self.balance += amount
        return self.balance

    def withdraw(self, amount):
        self.balance -= amount
        return self.balance

# Using the closure
deposit, withdraw = make_account(100)
print("Deposit 50:", deposit(50))    # 150
print("Withdraw 30:", withdraw(30))  # 120

# Using the class
acc = Account(100)
print("Class deposit 50:", acc.deposit(50))    # 150
print("Class withdraw 30:", acc.withdraw(30))  # 120
```

**Expected Output:**

```
Deposit 50: 150
Withdraw 30: 120
Class deposit 50: 150
Class withdraw 30: 120
```

**Step-by-Step Breakdown:**

1. The closure approach uses `nonlocal` to share `balance` between `deposit` and `withdraw`.
2. The class approach uses `self.balance` to share state.
3. Both produce the same results.

**Why the Output is Produced:**

Closures can encapsulate state just like classes, but without the overhead of class definition. The closure's free variables act as private instance variables.

#### Real-World Cases

- **Decorators**: `@wraps` and custom decorators use closures to wrap functions.
- **Factory Functions**: Generating specialized functions (e.g., `make_multiplier`, `make_validator`).
- **Stateful Callbacks**: Event handlers that remember their configuration.
- **Data Hiding**: Encapsulating private state without classes.

---

### 2. Nested Functions

#### Definitions

**Core Definition**

A nested function is a function defined inside another function. Nested functions can access variables from their enclosing function's scope, and when they do, they form a closure.

**Technical Definition**

A nested function is a function definition that appears within the body of another function. The nested function has its own local scope, but it can also read (and with `nonlocal`, write) variables from the enclosing function's local scope. If the nested function is returned by the enclosing function, it becomes a closure. PEP 227 added statically nested scoping to Python, allowing nested functions to reference variables defined in surrounding namespaces.

**Beginner-Friendly Explanation**

A nested function is a function inside a function. It's like a helper function that only the outer function knows about. The nested function can use the outer function's variables, which is what makes closures possible.

#### Purposes

- To organize code by keeping helper functions close to where they are used.
- To create closures that capture state from the enclosing scope.
- To implement decorators and factory functions.
- To encapsulate logic that should not be accessible from the outside.

#### Syntax Rules and Structure

**Complete General Syntax**

```
def outer_function():
    # Outer function scope
    outer_var = value

    def inner_function():
        # Inner function can access outer_var
        return outer_var

    return inner_function
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `outer_function` | The enclosing function. |
| `outer_var` | A variable in the enclosing scope. |
| `inner_function` | The nested function defined inside `outer_function`. |
| `return inner_function` | Returns the nested function (creates a closure). |

**Syntax Rules**

- Nested functions are defined with `def` inside another function.
- Nested functions can access variables from the enclosing scope.
- Nested functions cannot be accessed from outside the enclosing function unless returned.
- Nested functions can be arbitrarily deep (functions inside functions inside functions).
- The `nonlocal` keyword allows rebinding enclosing variables.

**Constraints and Limitations**

- Nested functions increase indentation and can reduce readability if overused.
- Nested functions are re-created each time the enclosing function is called.
- Debugging nested functions can be harder due to qualified names (`outer.<locals>.inner`).

#### Annotated Code Examples

**Example 1: Basic Nested Function**

```python
def outer():
    message = "Hello from outer"

    def inner():
        # inner can read message from the enclosing scope
        print(message)

    inner()  # Call the nested function

outer()
```

**Expected Output:**

```
Hello from outer
```

**Step-by-Step Breakdown:**

1. `outer()` creates a local scope with `message = "Hello from outer"`.
2. `inner()` is defined inside `outer()` and can access `message`.
3. `inner()` is called inside `outer()`, printing the message.

**Why the Output is Produced:**

Nested functions have access to their enclosing function's local variables through lexical scoping (the E in LEGB).

---

**Example 2: Returning a Nested Function (Creating a Closure)**

```python
def outer():
    message = "Hello from closure"

    def inner():
        print(message)

    return inner  # Return the nested function

# Get the closure
my_func = outer()
my_func()  # Call the closure
```

**Expected Output:**

```
Hello from closure
```

**Step-by-Step Breakdown:**

1. `outer()` defines `message` and `inner`.
2. `return inner` returns the nested function without calling it.
3. `my_func = outer()` stores the closure.
4. `my_func()` calls the closure, which still has access to `message`.

**Why the Output is Produced:**

The closure retains access to the enclosing scope's variables even after `outer()` has returned.

#### Real-World Cases

- **Decorators**: The wrapper function is a nested function.
- **Factory Functions**: Returning nested functions to create specialized functions.
- **Callback Factories**: Generating callbacks with pre-configured behavior.

---

### 3. Free Variables

#### Definitions

**Core Definition**

A free variable is a variable that is referenced in a nested function but not bound in that function's local scope and not in the global scope. It is "free" because it is not local to the inner function.

**Technical Definition**

In a closure, a free variable is a variable that is used in the inner function but defined in the enclosing function's scope. Python identifies free variables at compile time and stores them in cell objects, accessible via the function's `__closure__` attribute and listed in `co_freevars`. Free variables are resolved at call time (late binding), meaning the inner function sees the current value of the variable when it is called, not the value at definition time.

**Beginner-Friendly Explanation**

A free variable is a variable that the inner function "borrows" from the outer function. It's not defined inside the inner function, but the inner function can still use it because it's in the outer function's scope.

#### Purposes

- To allow inner functions to access data from their enclosing scope.
- To enable closures to carry state.
- To provide a mechanism for data hiding and encapsulation.

#### Syntax Rules and Structure

**How Free Variables Are Determined**

```
def outer():
    x = 10  # This will be a free variable if inner uses it
    y = 20  # This will NOT be a free variable if inner doesn't use it

    def inner():
        print(x)  # x is a free variable for inner
        # y is not referenced, so it's not a free variable

    return inner

f = outer()
print(f.__code__.co_freevars)  # ('x',)
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `x` | A variable in the enclosing scope that is referenced by `inner`. |
| `co_freevars` | Tuple of free variable names in the inner function. |
| `__closure__` | Tuple of cell objects holding the free variable values. |

**Syntax Rules**

- A variable is free if it is used in a nested function but not bound there.
- Free variables are captured by reference, not by value.
- The `nonlocal` keyword allows rebinding of free variables.
- `co_freevars` lists the names of free variables.
- `__closure__` holds the cell objects for free variables.

**Constraints and Limitations**

- Free variables are resolved at call time, not definition time.
- If the enclosing function rebinds the variable before the closure is called, the closure sees the new value.
- Free variables can cause reference cycles if the closure references itself.

#### Annotated Code Examples

**Example 1: Inspecting Free Variables**

```python
def outer():
    a = 1
    b = 2
    c = 3

    def inner():
        return a + b  # a and b are free variables; c is not used

    return inner

f = outer()
print("Free variables:", f.__code__.co_freevars)
print("Closure cells:", f.__closure__)
print("Cell contents:", [cell.cell_contents for cell in f.__closure__])
print("Result:", f())
```

**Expected Output:**

```
Free variables: ('a', 'b')
Closure cells: (<cell at 0x...: int object at 0x...>, <cell at 0x...: int object at 0x...>)
Cell contents: [1, 2]
Result: 3
```

**Step-by-Step Breakdown:**

1. `outer()` defines `a`, `b`, and `c`.
2. `inner()` references `a` and `b` but not `c`.
3. `co_freevars` shows `('a', 'b')`.
4. `__closure__` holds cell objects for `a` and `b`.
5. `f()` returns `1 + 2 = 3`.

**Why the Output is Produced:**

Python identifies free variables at compile time. Only variables referenced in the inner function are captured.

---

**Example 2: Late Binding of Free Variables**

```python
def outer():
    x = 10

    def inner():
        print(x)  # Looks up x at call time

    x = 20  # Rebind x before returning inner
    return inner

f = outer()
f()  # Prints 20, not 10
```

**Expected Output:**

```
20
```

**Step-by-Step Breakdown:**

1. `outer()` initially sets `x = 10`.
2. `inner` references `x` but does not capture its value at definition time.
3. `x` is rebound to `20`.
4. `inner` is returned; when called, it looks up `x` and finds `20`.

**Why the Output is Produced:**

Free variables are resolved at call time (late binding). The closure sees the current value of `x`, which is `20`.

#### Real-World Cases

- **Loop Closures**: The classic late binding pitfall occurs because loop variables are free variables.
- **Configuration Capturing**: Closures capture configuration values for later use.
- **Stateful Functions**: Free variables hold state between calls.

---

### 4. Closure Scope

#### Definitions

**Core Definition**

Closure scope refers to the lexical scope that a closure captures — the scope of the enclosing function at the time the closure is created. This scope persists as long as the closure exists.

**Technical Definition**

Closure scope is the enclosing function's local namespace, which is captured by the closure. The variables in this scope (free variables) are stored in cell objects and remain accessible even after the enclosing function returns. Python's scoping rules (LEGB) ensure that the closure searches its local scope first, then the enclosing scope (the closure scope), then global, then built-in. The `nonlocal` keyword allows the closure to rebind variables in the closure scope.

**Beginner-Friendly Explanation**

The closure scope is like a snapshot of the outer function's variables that the inner function can access. Even after the outer function is done, the inner function still has access to this scope.

#### Purposes

- To preserve state from the enclosing function.
- To allow inner functions to access variables that would otherwise be out of scope.
- To enable data encapsulation and private state.

#### Syntax Rules and Structure

**Closure Scope in the LEGB Rule**

```
# LEGB: Local -> Enclosing -> Global -> Built-in
def outer():
    enclosing_var = "enclosing"  # Closure scope

    def inner():
        local_var = "local"      # Local scope
        print(local_var)         # L
        print(enclosing_var)     # E (closure scope)
        print(global_var)        # G
        print(len)               # B

    return inner
```

**Component Breakdown**

| Scope | Description |
|-------|-------------|
| Local | Variables defined in the inner function. |
| Enclosing (Closure) | Variables from the enclosing function. |
| Global | Module-level variables. |
| Built-in | Python's built-in names. |

**Syntax Rules**

- The closure scope is searched after the local scope and before the global scope.
- Free variables are stored in cell objects.
- The `nonlocal` keyword allows rebinding of closure scope variables.
- The closure scope persists as long as the closure exists.

**Constraints and Limitations**

- The closure scope is read-only unless `nonlocal` is used.
- Deleting a free variable from the closure scope is an error.
- The closure scope is not accessible from outside the closure.

#### Annotated Code Examples

**Example 1: Closure Scope in Action**

```python
global_var = "global"

def outer():
    enclosing_var = "enclosing"

    def inner():
        local_var = "local"
        print("Local:", local_var)
        print("Enclosing:", enclosing_var)
        print("Global:", global_var)
        print("Built-in:", len)

    return inner

f = outer()
f()
```

**Expected Output:**

```
Local: local
Enclosing: enclosing
Global: global
Built-in: <built-in function len>
```

**Step-by-Step Breakdown:**

1. `inner()` defines `local_var` in its local scope.
2. `enclosing_var` is found in the closure scope (enclosing).
3. `global_var` is found in the global scope.
4. `len` is found in the built-in scope.

**Why the Output is Produced:**

Python resolves names using the LEGB rule. The closure scope provides the "E" level.

---

**Example 2: Modifying Closure Scope with `nonlocal`**

```python
def outer():
    count = 0

    def inner():
        nonlocal count
        count += 1
        return count

    return inner

f = outer()
print(f())  # 1
print(f())  # 2
print(f())  # 3
```

**Expected Output:**

```
1
2
3
```

**Step-by-Step Breakdown:**

1. `outer()` creates `count = 0` in its local scope.
2. `inner()` uses `nonlocal count` to rebind the enclosing variable.
3. Each call to `f()` increments and returns `count`.

**Why the Output is Produced:**

`nonlocal` tells Python that `count` refers to the variable in the nearest enclosing scope, not a new local variable.

#### Real-World Cases

- **Counters and Accumulators**: Closures with `nonlocal` maintain state across calls.
- **Memoization**: Caches stored in the closure scope.
- **Configuration**: Closure scope holds configuration parameters.

---

### 5. Closure Use Cases

#### Definitions

**Core Definition**

Closure use cases are the scenarios where closures provide elegant solutions: factory functions, decorators, callbacks, stateful functions, and data hiding.

**Technical Definition**

Closures are idiomatic in Python when you need a function that carries private state, when you want to generate specialized functions from a template, or when you need to wrap a function with additional behavior. They are used extensively in the standard library (`functools.wraps`, `functools.lru_cache`, `functools.partial`) and in frameworks (Flask routes, Django middleware, GUI event handlers).

**Beginner-Friendly Explanation**

Closures are useful whenever you want a function to "remember" something. For example, a function that adds 10 to any number, or a counter that keeps track of how many times it's been called, or a decorator that logs function calls.

#### Purposes

- To create factory functions that generate specialized functions.
- To implement decorators that wrap functions with additional behavior.
- To create stateful functions that remember data between calls.
- To provide callbacks with private configuration.
- To implement data hiding and encapsulation without classes.

#### Annotated Code Examples

**Example 1: Factory Function**

```python
def power_factory(exponent):
    """Return a function that raises its argument to the given exponent."""
    def power(base):
        return base ** exponent
    return power

square = power_factory(2)
cube = power_factory(3)

print("Square 4:", square(4))
print("Cube 4:", cube(4))
print("Square 5:", square(5))
print("Cube 5:", cube(5))
```

**Expected Output:**

```
Square 4: 16
Cube 4: 64
Square 5: 25
Cube 5: 125
```

**Step-by-Step Breakdown:**

1. `power_factory(2)` creates a closure with `exponent = 2`.
2. `power_factory(3)` creates a closure with `exponent = 3`.
3. Each closure remembers its exponent.

**Why the Output is Produced:**

The closures capture the `exponent` free variable, creating specialized functions.

---

**Example 2: Decorator**

```python
import time
from functools import wraps

def timed(func):
    """Decorator that prints the execution time of a function."""
    @wraps(func)
    def wrapper(*args, **kwargs):
        start = time.perf_counter()
        result = func(*args, **kwargs)
        end = time.perf_counter()
        print(f"{func.__name__} took {end - start:.4f} seconds")
        return result
    return wrapper

@timed
def slow_add(a, b):
    time.sleep(0.1)
    return a + b

print("Result:", slow_add(3, 5))
```

**Expected Output (approximate):**

```
slow_add took 0.1002 seconds
Result: 8
```

**Step-by-Step Breakdown:**

1. `timed` is a decorator that wraps `slow_add`.
2. `wrapper` is a closure that captures `func`.
3. `wrapper` times the execution and calls the original function.

**Why the Output is Produced:**

The decorator uses a closure to wrap the original function with additional behavior.

---

**Example 3: Stateful Function**

```python
def make_accumulator():
    """Return a function that accumulates values."""
    total = 0

    def add(amount):
        nonlocal total
        total += amount
        return total

    return add

acc = make_accumulator()
print(acc(10))   # 10
print(acc(20))   # 30
print(acc(5))    # 35

# Each accumulator is independent
acc2 = make_accumulator()
print(acc2(100)) # 100
print(acc(1))    # 36
```

**Expected Output:**

```
10
30
35
100
36
```

**Step-by-Step Breakdown:**

1. `make_accumulator()` creates a closure with `total = 0`.
2. `add` uses `nonlocal total` to accumulate values.
3. Each accumulator instance has its own `total`.

**Why the Output is Produced:**

The closure captures `total` and maintains its state across calls. Separate calls to `make_accumulator` create independent closures.

#### Real-World Cases

- **Web Frameworks**: Flask's `@app.route` decorator uses closures.
- **Caching**: `functools.lru_cache` uses closures to store cached results.
- **GUI Programming**: Tkinter buttons use closures for event handlers.
- **Data Pipelines**: Closures create reusable transformation functions.

---

### 6. Closure Pitfalls

#### Definitions

**Core Definition**

Closure pitfalls are common mistakes and surprising behaviors related to closures, most notably the late binding of free variables in loops.

**Technical Definition**

The most famous closure pitfall is **late binding**: closures capture variables, not values. In a loop, all closures created in the loop share the same variable, and when called later, they all see the final value of that variable. Other pitfalls include memory leaks from long-lived closures holding references to large objects, and reference cycles when closures reference themselves.

**Beginner-Friendly Explanation**

The classic closure pitfall is when you create functions in a loop and expect them to remember different values, but they all end up using the last value. This happens because Python looks up the variable when the function is called, not when it's defined.

#### Purposes

- To recognize and avoid the late binding trap.
- To understand how to capture values correctly in closures.
- To prevent memory leaks and reference cycles.

#### Syntax Rules and Structure

**The Late Binding Problem**

```
# BUG: All functions return 4
funcs = [lambda: i for i in range(5)]
for f in funcs:
    print(f(), end=" ")
print()

# FIX: Capture the current value with a default argument
funcs_fixed = [lambda i=i: i for i in range(5)]
for f in funcs_fixed:
    print(f(), end=" ")
print()
```

**Component Breakdown**

| Approach | Description |
|----------|-------------|
| Buggy | `lambda: i` captures the variable `i`, not its value. |
| Fixed (default arg) | `lambda i=i: i` binds the current value at definition time. |
| Fixed (factory function) | `lambda i=i: i` or `partial(lambda i: i, i)` |

**Syntax Rules**

- Closures capture variables, not values.
- Use default arguments to capture values at definition time.
- Use `functools.partial` to bind arguments.
- Use a factory function to create a new scope for each iteration.

**Constraints and Limitations**

- Late binding is a fundamental consequence of Python's scoping rules.
- The default argument trick works but can be confusing.
- `functools.partial` is a cleaner alternative.

#### Annotated Code Examples

**Example 1: Late Binding Bug and Fix**

```python
# BUG: All closures return 4
funcs_buggy = [lambda: i for i in range(5)]
print("Buggy:", [f() for f in funcs_buggy])

# FIX 1: Default argument
funcs_fixed1 = [lambda i=i: i for i in range(5)]
print("Fixed (default):", [f() for f in funcs_fixed1])

# FIX 2: Factory function
def make_func(i):
    return lambda: i

funcs_fixed2 = [make_func(i) for i in range(5)]
print("Fixed (factory):", [f() for f in funcs_fixed2])

# FIX 3: functools.partial
from functools import partial

funcs_fixed3 = [partial(lambda i: i, i) for i in range(5)]
print("Fixed (partial):", [f() for f in funcs_fixed3])
```

**Expected Output:**

```
Buggy: [4, 4, 4, 4, 4]
Fixed (default): [0, 1, 2, 3, 4]
Fixed (factory): [0, 1, 2, 3, 4]
Fixed (partial): [0, 1, 2, 3, 4]
```

**Step-by-Step Breakdown:**

1. The buggy version creates five lambdas that all reference the same variable `i`.
2. By the time the lambdas are called, `i` is `4`.
3. The default argument fix binds the current value of `i` at definition time.
4. The factory function creates a new scope for each `i`.
5. `partial` binds the argument at definition time.

**Why the Output is Produced:**

Late binding causes all closures to see the final value of the loop variable. Capturing the value at definition time (via default argument, factory, or partial) fixes the problem.

---

**Example 2: Memory Leak from Long-Lived Closures**

```python
def create_big_closure():
    big_data = [0] * 10_000_000  # Large list

    def get_length():
        return len(big_data)

    return get_length

# The closure holds a reference to big_data
f = create_big_closure()
print(f())  # 10000000

# Even though we only need the length, big_data stays in memory
del f  # Now big_data can be garbage collected
```

**Expected Output:**

```
10000000
```

**Step-by-Step Breakdown:**

1. `create_big_closure` creates a large list and a closure that references it.
2. The closure `get_length` captures `big_data` as a free variable.
3. Even though `get_length` only needs the length, the entire list is kept alive.
4. Deleting `f` releases the closure and allows `big_data` to be garbage collected.

**Why the Output is Produced:**

Closures hold references to all free variables they capture. If a closure captures a large object but only uses a small part of it, the entire object stays in memory until the closure is deleted.

#### Real-World Cases

- **Loop Closures**: Event handlers registered in loops are a common source of late binding bugs.
- **GUI Programming**: Button callbacks in loops often suffer from late binding.
- **Memory Management**: Long-lived closures can prevent garbage collection of large objects.

---

## References

- Python Official Documentation – Lambdas and Nested Functions - https://docs.python.org/3/tutorial/controlflow.html#lambda-expressions
- Python Language Reference – Naming and Binding - https://docs.python.org/3/reference/executionmodel.html#naming-and-binding
- Python Language Reference – The `nonlocal` Statement - https://docs.python.org/3/reference/simple_stmts.html#the-nonlocal-statement
- Python Language Reference – Data Model: Cell Objects - https://docs.python.org/3/c-api/cell.html
- PEP 227 – Statically Nested Scopes - https://peps.python.org/pep-0227/
- Real Python – Python Closures: Common Use Cases and Examples - https://realpython.com/python-closure/
- Real Python – Closure | Python Glossary - https://realpython.com/ref/glossary/closure/
- Real Python – Recapping Scopes and Closures - https://realpython.com/lessons/recapping-scopes-closures/
- Python Guide – Common Gotchas: Late Binding Closures - https://docs.python-guide.org/writing/gotchas/#late-binding-closures
- Stas Bekman's Python Cookbook – Functions, Closures, Decorators - https://huggingface.co/stas/python-cookbook
- 廖雪峰 – 返回函数 (Closures) - https://liaoxuefeng.com/books/python/functional/return-function/index.html
- Tencent Cloud – Python之函数编程（闭包） - https://cloud.tencent.com.cn/developer/article/1533566