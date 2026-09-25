# Function Fundamentals: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
A function is a named, reusable block of code that performs a specific task, optionally accepts input values, and optionally returns a result.

**Technical Definition**
In Python, a function is a first-class object created by the `def` statement (or `lambda` expression). The `def` statement binds a function name in the enclosing namespace to a function object whose `__code__` attribute contains the compiled bytecode of the body. Function objects support `__call__`, allowing them to be invoked with arguments; they also carry metadata such as `__name__`, `__doc__`, `__defaults__`, and `__annotations__`. Arguments are bound to parameters according to Python's calling conventions (positional, keyword, `*args`, `**kwargs`) before the body executes.

**Beginner-Friendly Explanation**
Think of a function as a recipe with a name. You write the recipe once, and whenever you need to cook that dish, you "call" the recipe by name instead of rewriting all the steps. Some recipes need ingredients (parameters/arguments), and most recipes produce a finished dish (return value). If a recipe doesn't explicitly say what to produce, it produces `None` by default.

### Key Characteristics

| Characteristic | Description |
|---|---|
| **Reusability** | Define once, call many times from different parts of a program |
| **First-Class Objects** | Functions can be stored in variables, passed as arguments, and returned from other functions  |
| **Encapsulation** | A function's internal variables (local scope) are not accessible outside unless returned |
| **Abstraction** | Callers only need to know the function's interface, not its implementation |
| **Optional Return** | Functions return `None` implicitly if no `return` statement is executed  |
| **Introspectable** | Function metadata (signature, docstring, annotations) can be inspected at runtime via the `inspect` module  |

### Prerequisites

- **Basic Python syntax**: Indentation, variables, expressions
- **The `def` statement and function call syntax**
- **Understanding of Python objects**: Everything is an object, including functions 
- **Basic understanding of scope**: Local vs. global variables 

### Related Programming Areas

- **Object-Oriented Programming**: Methods are functions bound to class instances
- **Functional Programming**: Functions as first-class citizens, higher-order functions, closures
- **Decorators**: Functions that modify other functions
- **Metaprogramming**: Inspecting and manipulating function signatures at runtime
- **API Design**: Function signatures and docstrings form the public interface of libraries

### Core Concepts / Features

1. **Defining Functions** (`def` statement, function body, naming rules)
2. **Calling Functions** (invocation, argument passing, `func` vs `func()`)
3. **Parameters vs. Arguments** (formal vs. actual, terminology)
4. **Return Values** (explicit `return`, implicit `None`, multiple values via tuples)
5. **Function Documentation** (Docstrings, Google/NumPy styles, automated generation)
6. **Function Signature Inspection** (`inspect` module, `Signature`, `Parameter` objects)

---

## Core Concept 1: Defining Functions

### Definitions

**Core Definition**
Defining a function creates a named, reusable block of code that can be executed later by calling its name.

**Technical Definition**
The `def` statement is an executable statement that compiles the function body into a code object, creates a function object from that code object, and binds the function name in the current namespace to that object . The function object stores default values, annotations, and a reference to the enclosing global namespace. The body is not executed at definition time; execution occurs only when the function is called .

**Beginner-Friendly Explanation**
Writing `def greet():` tells Python "here comes a recipe named `greet`." Everything indented under it is the recipe's steps. Python remembers the recipe but doesn't cook it yet. Only when you write `greet()` does Python actually run the steps.

### Purposes

- **To encapsulate reusable logic** under a single name
- **To break large programs into smaller, manageable pieces**
- **To avoid code duplication** by defining an algorithm once and calling it many times
- **To create abstractions** that hide implementation details from callers
- **To organize code** into logical, testable units

### Syntax Rules and Structure

**Complete General Syntax**

```python
def function_name(parameter1, parameter2=default_value, *args, **kwargs):
    """Optional docstring."""
    # Function body
    statement1
    statement2
    return value  # Optional
```

**Breakdown:**
- `def`: Keyword that starts a function definition.
- `function_name`: A valid identifier (letters, digits, underscores; cannot start with a digit; cannot be a keyword) .
- `parameter1, parameter2`: Names that will receive argument values.
- `default_value`: Optional default if no argument is provided .
- `*args`: Collects extra positional arguments into a tuple.
- `**kwargs`: Collects extra keyword arguments into a dictionary.
- `: `: Ends the function header; the indented block is the body.

**Syntax Rules**

| Rule | Description |
|---|---|
| `def` keyword required | Function definitions always begin with `def` |
| Parentheses required | Even for functions with no parameters: `def f():` |
| Colon required | The header ends with a colon |
| Indented body | Body must be indented consistently (4 spaces per PEP 8) |
| Return is optional | If absent, the function returns `None`  |
| Naming rules apply | Same rules as variables: letters, digits, underscores, not starting with digit, not a keyword  |

**Constraints and Limitations**

- **No execution at definition**: The function body does not run when the `def` statement executes; only the function object is created .
- **Default arguments are evaluated once**: Default values are evaluated at definition time, not call time. Mutable defaults (e.g., `def f(x=[])`) accumulate changes across calls .
- **Default parameters must follow non-default parameters**: `def f(a=1, b)` is a `SyntaxError` .

### Multiple Annotated Complete Code Examples and Expected Outputs

**Example 1: Minimal Function Definition**

```python
# Step 1: Define the function
def greet():
    """Print a simple greeting."""
    print("Hello, World!")

# At this point, the function is defined but not executed.
# The name 'greet' now refers to a function object.

# Step 2: Call the function to execute it
greet()
```

**Expected Output:**
```
Hello, World!
```

**Breakdown:**
1. `def greet():` creates a function object and binds the name `greet` to it.
2. The docstring `"""Print a simple greeting."""` becomes the function's `__doc__` attribute.
3. `greet()` invokes the function, executing the `print` statement.

**Example 2: Function with Parameters**

```python
# Step 1: Define a function that accepts two parameters
def add(x, y):
    """Return the sum of x and y."""
    result = x + y
    return result

# Step 2: Call with arguments
sum1 = add(3, 4)
sum2 = add(10, 20)

print(sum1)
print(sum2)
```

**Expected Output:**
```
7
30
```

**Breakdown:**
1. `x` and `y` are parameters (placeholders for values).
2. `add(3, 4)` passes arguments `3` and `4`; the function returns `7`.
3. The return value is stored in `sum1`.
4. The second call returns `30`, stored in `sum2`.

**Example 3: Function with Default Parameters and `*args`**

```python
# Function with default parameter
def power(base, exponent=2):
    """Raise base to exponent (default is square)."""
    return base ** exponent

print(power(5))        # Uses default exponent=2
print(power(2, 3))     # Explicit exponent=3

# Function with *args
def sum_all(*numbers):
    """Return the sum of all provided numbers."""
    total = 0
    for n in numbers:
        total += n
    return total

print(sum_all(1, 2, 3))       # Three arguments
print(sum_all(10, 20, 30, 40)) # Four arguments
```

**Expected Output:**
```
25
8
6
100
```

**Breakdown:**
1. `power(5)` uses the default `exponent=2`, returning 5² = 25.
2. `power(2, 3)` overrides the default, returning 2³ = 8.
3. `*numbers` collects all positional arguments into a tuple: `(1, 2, 3)`.
4. The loop sums the tuple elements.

### Real-World Cases with Explanation

**Case 1: Mathematical Utilities**

```python
def celsius_to_fahrenheit(celsius):
    """Convert Celsius to Fahrenheit."""
    return (celsius * 9/5) + 32

def fahrenheit_to_celsius(fahrenheit):
    """Convert Fahrenheit to Celsius."""
    return (fahrenheit - 32) * 5/9
```

**Why it matters:** Temperature conversion is a common operation. Defining these as functions makes the conversion logic reusable and testable, and the descriptive names document intent.

**Case 2: Input Validation**

```python
def is_valid_email(email):
    """Check if a string looks like a valid email address."""
    return "@" in email and "." in email.split("@")[-1]
```

**Why it matters:** Validation logic is often repeated. Encapsulating it in a function ensures consistency and makes it easy to update the validation rules in one place.

---

## Core Concept 2: Calling Functions

### Definitions

**Core Definition**
Calling a function means executing it by writing its name followed by parentheses containing any arguments.

**Technical Definition**
The call expression `func(arg1, arg2)` evaluates `func` to obtain a callable object, evaluates the arguments, binds the arguments to the function's parameters according to its signature, and then executes the function body. The result of the call is the value returned by the `return` statement, or `None` if no `return` is executed. The special syntax `func` without parentheses refers to the function object itself, not its result .

**Beginner-Friendly Explanation**
Defining a function is like writing a recipe. Calling it is like actually following the recipe to make the dish. If you just write the recipe's name without saying "make it" (the parentheses), nothing gets cooked—you just have a reference to the recipe card.

### Purposes

- **To execute the function's logic** and produce its result
- **To pass input values** (arguments) into the function for processing
- **To trigger side effects** such as printing, writing files, or modifying state
- **To obtain return values** for use in further computation
- **To reuse code** without duplicating the algorithm

### Syntax Rules and Structure

**Complete General Syntax**

```
function_name(argument1, argument2, keyword=value, *positional, **keyword_args)
```

**Breakdown:**
- `function_name`: The name bound to a function object.
- `( )`: Required for invocation; without them, you refer to the function object, not call it .
- `argument1, argument2`: Positional arguments matched to parameters by order.
- `keyword=value`: Keyword argument matched to a parameter by name.
- `*positional`: Unpacks an iterable into positional arguments.
- `**keyword_args`: Unpacks a dictionary into keyword arguments.

**Syntax Rules**

| Rule | Description |
|---|---|
| Parentheses required | `func` is the function object; `func()` is the call  |
| Arguments separated by commas | `func(a, b, c)` |
| Keyword arguments after positional | `func(1, 2, key=3)` is valid; `func(key=3, 1, 2)` is a `SyntaxError` |
| Argument count must match | Unless defaults or `*args`/`**kwargs` are defined |
| Function must be defined before call | The `def` statement must execute before the call |

**Constraints and Limitations**

- **`TypeError` on mismatch**: Calling with wrong number of arguments raises `TypeError`.
- **Mutable default arguments**: As noted in Core Concept 1, default mutable values are shared across calls.
- **Recursion depth**: Python has a default recursion limit (typically 1000); deep recursion raises `RecursionError`.
- **Call stack overhead**: Each call adds a frame to the call stack; excessive call depth or frequency can impact performance.

### Multiple Annotated Complete Code Examples and Expected Outputs

**Example 1: Positional vs. Keyword Arguments**

```python
def describe_pet(name, species):
    """Print a description of a pet."""
    print(f"{name} is a {species}.")

# Positional arguments (order matters)
describe_pet("Rex", "dog")
# name="Rex", species="dog"

# Keyword arguments (order does not matter)
describe_pet(species="cat", name="Whiskers")
# name="Whiskers", species="cat"
```

**Expected Output:**
```
Rex is a dog.
Whiskers is a cat.
```

**Breakdown:**
1. In the first call, arguments are matched by position: `"Rex"` → `name`, `"dog"` → `species`.
2. In the second call, arguments are matched by keyword name, so order is irrelevant.
3. Both calls produce valid output.

**Example 2: The `func` vs. `func()` Distinction**

```python
def say_hello():
    print("Hello!")

# Reference the function object (no call)
ref = say_hello
print(ref)
# Output: <function say_hello at 0x...>

# Call the function
ref()
# Output: Hello!

# This is a common bug:
result = say_hello  # Assigns the function, not the result
print(result)       # Prints the function object
# Output: <function say_hello at 0x...>
```

**Expected Output:**
```
<function say_hello at 0x...>
Hello!
<function say_hello at 0x...>
```

**Breakdown:**
1. `ref = say_hello` stores the function object in `ref`; nothing is executed.
2. `ref()` calls the function, printing "Hello!".
3. The bug: `result = say_hello` (without parentheses) assigns the function object, not the return value. Printing `result` shows the function's representation, not `None` .

**Example 3: Nested Function Calls**

```python
def add(x, y):
    return x + y

def square(x):
    return x * x

# The inner call executes first
result = square(add(3, 4))
# Step 1: add(3, 4) returns 7
# Step 2: square(7) returns 49
print(result)
```

**Expected Output:**
```
49
```

**Breakdown:**
1. Python evaluates `add(3, 4)` first because it is an argument to `square`.
2. `add` returns `7`.
3. `square(7)` returns `49`.
4. The result is assigned to `result` .

### Real-World Cases with Explanation

**Case 1: Event Handlers**

```python
def on_click(event):
    print(f"Clicked at {event.x}, {event.y}")

# The handler is passed as a reference, not called
button.on_click(on_click)  # Correct: passes the function
# button.on_click(on_click())  # Wrong: calls it immediately
```

**Why it matters:** In GUI programming and event-driven systems, functions are often passed as references to be called later. Confusing `func` with `func()` is a common source of bugs .

**Case 2: Pipeline Composition**

```python
def clean(text):
    return text.strip().lower()

def tokenize(text):
    return text.split()

result = tokenize(clean("  Hello World  "))
print(result)
# Output: ['hello', 'world']
```

**Why it matters:** Nested function calls create data pipelines. The inner function produces output that the outer function consumes, enabling clean, readable composition.

---

## Core Concept 3: Parameters vs. Arguments

### Definitions

**Core Definition**
A parameter is a variable in the function definition; an argument is the actual value passed when calling the function.

**Technical Definition**
Parameters (sometimes called "formal parameters") are the names listed in the function's `def` statement. Arguments (sometimes called "actual parameters") are the values supplied in the function call. At call time, Python binds each argument to its corresponding parameter according to the function's signature. The distinction is terminological, but precise usage aids clarity in documentation and discussion .

**Beginner-Friendly Explanation**
Think of a function as a form with blank fields. The blanks in the form (before anyone fills them) are **parameters**. When someone fills in the blanks with actual information, those filled-in values are **arguments**. So `def greet(name):` has a parameter `name`; `greet("Alice")` passes the argument `"Alice"`.

### Purposes

- **To distinguish definition-time placeholders** from call-time values
- **To enable precise technical communication** about function interfaces
- **To support documentation** that clearly describes what values a function expects
- **To understand binding behavior** (how arguments map to parameters)
- **To clarify error messages** that mention parameters or arguments

### Syntax Rules and Structure

**Definition Site (Parameters):**

```python
def function(param1, param2, default_param=10):
    # param1, param2, default_param are parameters
    pass
```

**Call Site (Arguments):**

```python
function(1, 2, 3)
# 1, 2, 3 are arguments
```

**Terminology Table:**

| Term | Location | Example | Meaning |
|---|---|---|---|
| Parameter | Function definition | `def f(x):` — `x` | Placeholder variable |
| Argument | Function call | `f(5)` — `5` | Actual value passed |
| Formal parameter | Function definition | Synonym for parameter | Older terminology  |
| Actual parameter | Function call | Synonym for argument | Older terminology  |

**Syntax Rules**

| Rule | Description |
|---|---|
| Parameters are local variables | Inside the function, parameters behave like variables  |
| Arguments are values or references | Arguments are evaluated before binding |
| Binding happens at call time | Parameters take on argument values at invocation  |
| Keyword arguments use parameter names | `f(x=5)` requires `x` to be a parameter name |

**Constraints and Limitations**

- **Terminology is often blurred**: Many texts and developers use "parameter" and "argument" interchangeably . Strict usage distinguishes them, but practical communication often tolerates the overlap.
- **Parameter names matter for keyword arguments**: Callers using keyword arguments must use the exact parameter names.
- **Parameter names are not accessible outside the function**: They are local to the function's scope.

### Multiple Annotated Complete Code Examples and Expected Outputs

**Example 1: Identifying Parameters and Arguments**

```python
# 'a' and 'b' are PARAMETERS
def multiply(a, b):
    return a * b

# 3 and 4 are ARGUMENTS
result = multiply(3, 4)

print(result)
```

**Expected Output:**
```
12
```

**Breakdown:**
- The `def` statement defines parameters `a` and `b`.
- The call `multiply(3, 4)` supplies arguments `3` and `4`.
- At call time, `a` is bound to `3` and `b` is bound to `4`.

**Example 2: Keyword Arguments Use Parameter Names**

```python
def create_user(username, email, role="user"):
    """Create a user with the given details."""
    return {"username": username, "email": email, "role": role}

# Positional arguments
user1 = create_user("alice", "alice@example.com")

# Keyword arguments (must match parameter names)
user2 = create_user(username="bob", email="bob@example.com", role="admin")

print(user1)
print(user2)
```

**Expected Output:**
```
{'username': 'alice', 'email': 'alice@example.com', 'role': 'user'}
{'username': 'bob', 'email': 'bob@example.com', 'role': 'admin'}
```

**Breakdown:**
- `username`, `email`, and `role` are the parameter names.
- In the second call, keyword arguments must use those exact names.
- `role="admin"` overrides the default `"user"`.

**Example 3: Parameter Defaults and Argument Omission**

```python
def greet(name, greeting="Hello"):
    """Greet someone with an optional custom greeting."""
    print(f"{greeting}, {name}!")

# Only one argument supplied; 'greeting' uses its default
greet("Alice")
# Output: Hello, Alice!

# Both arguments supplied
greet("Bob", "Hi")
# Output: Hi, Bob!
```

**Expected Output:**
```
Hello, Alice!
Hi, Bob!
```

**Breakdown:**
- `name` is a required parameter; `greeting` has a default value.
- Omitting the second argument causes `greeting` to use `"Hello"`.
- Providing it overrides the default.

### Real-World Cases with Explanation

**Case 1: API Documentation**

```python
def send_email(to, subject, body, cc=None, bcc=None):
    """Send an email.

    Parameters:
        to (str): Recipient email address.
        subject (str): Email subject line.
        body (str): Email body text.
        cc (str, optional): Carbon copy address.
        bcc (str, optional): Blind carbon copy address.
    """
    pass
```

**Why it matters:** Clear distinction between parameters and their expected argument types helps users of the function know what to pass. Documentation typically uses "parameters" for the definition-site names.

**Case 2: Debugging TypeErrors**

```python
def divide(a, b):
    return a / b

try:
    divide(10)  # Missing 'b'
except TypeError as e:
    print(e)
    # Output: divide() missing 1 required positional argument: 'b'
```

**Why it matters:** Python's error message uses the word "argument" for the call-site problem and "argument" for the missing value. Understanding the terminology helps interpret error messages.

---

## Core Concept 4: Return Values

### Definitions

**Core Definition**
A return value is the result that a function sends back to the caller, specified by the `return` statement or implicitly `None` if no `return` is executed.

**Technical Definition**
The `return` statement terminates the function's execution immediately and passes the value of its expression to the caller. If the expression is omitted, `None` is returned. If no `return` statement is executed (e.g., the function body ends without one), the function also returns `None` . A function can return multiple values by returning a tuple, which the caller can unpack. The `return` statement ends the function; any code after it is unreachable and ignored .

**Beginner-Friendly Explanation**
When a function finishes its work, it can hand back a result to whoever called it. If you don't explicitly say what to hand back, Python hands back `None`—a special value meaning "nothing." You can hand back one value, or you can hand back several values by putting them in a tuple (like handing someone a shopping bag with multiple items).

### Purposes

- **To send computed results** back to the caller for further use
- **To signal success or failure** (e.g., returning `True`/`False` or an error code)
- **To terminate a function early** when a condition is met
- **To enable function composition** by using return values as arguments to other functions
- **To provide multiple outputs** through tuple returns and unpacking

### Syntax Rules and Structure

**Complete General Syntaxes**

```python
# Explicit return of a single value
return expression

# Explicit return of multiple values (as a tuple)
return value1, value2, value3

# Implicit return of None (no return statement)
# Function ends, returns None

# Early return (exits function immediately)
if condition:
    return value
```

**Breakdown:**
- `return expression`: Evaluates `expression`, terminates the function, and passes the result to the caller.
- `return value1, value2`: Creates a tuple `(value1, value2)` and returns it.
- If no `return` executes, the function returns `None` .

**Syntax Rules**

| Rule | Description |
|---|---|
| `return` ends the function | Code after `return` in the same block is unreachable  |
| `return` without expression | Returns `None` |
| Multiple values | Returned as a tuple; caller can unpack |
| Bare `return` | `return` alone returns `None` |
| No `return` | Function returns `None` implicitly  |

**Constraints and Limitations**

- **Single return value concept**: Python functions technically return exactly one object. "Multiple return values" are syntactic sugar for returning a tuple .
- **Unreachable code**: Statements after `return` within the same block are never executed and may produce warnings.
- **`return` only inside functions**: Using `return` outside a function is a `SyntaxError`.

### Multiple Annotated Complete Code Examples and Expected Outputs

**Example 1: Explicit Return vs. Implicit None**

```python
def add(a, b):
    """Return the sum."""
    return a + b

def print_sum(a, b):
    """Print the sum (no return)."""
    print(a + b)

# Explicit return
result = add(3, 4)
print(result)
# Output: 7

# Implicit None
result = print_sum(3, 4)
print(result)
# Output: 7
#         None
```

**Expected Output:**
```
7
7
None
```

**Breakdown:**
1. `add(3, 4)` returns `7`, which is stored in `result`.
2. `print_sum(3, 4)` prints `7` but returns `None`.
3. Printing `result` after `print_sum` shows `None` because the function has no `return` .

**Example 2: Multiple Return Values via Tuple**

```python
def min_max(numbers):
    """Return both the minimum and maximum of a list."""
    return min(numbers), max(numbers)

# The returned tuple is unpacked into two variables
lowest, highest = min_max([3, 1, 4, 1, 5, 9, 2, 6])

print(f"Lowest: {lowest}")
print(f"Highest: {highest}")

# The function actually returns a single tuple
result = min_max([3, 1, 4, 1, 5, 9, 2, 6])
print(result)
print(type(result))
```

**Expected Output:**
```
Lowest: 1
Highest: 9
(1, 9)
<class 'tuple'>
```

**Breakdown:**
1. `return min(numbers), max(numbers)` creates a tuple `(1, 9)`.
2. The caller unpacks it into `lowest` and `highest`.
3. Printing `result` shows the tuple; `type(result)` confirms it is a `tuple` .

**Example 3: Early Return**

```python
def safe_divide(a, b):
    """Return a/b, or None if b is zero."""
    if b == 0:
        return None  # Early return: function ends here
    return a / b

print(safe_divide(10, 2))
print(safe_divide(10, 0))
```

**Expected Output:**
```
5.0
None
```

**Breakdown:**
1. When `b == 0`, the early `return None` terminates the function immediately.
2. The second `return a / b` is not reached in that case.
3. When `b != 0`, the division executes and the result is returned.

### Real-World Cases with Explanation

**Case 1: Data Processing Pipeline**

```python
def parse_line(line):
    """Parse a CSV line into fields, or return None if empty."""
    line = line.strip()
    if not line:
        return None
    return line.split(",")

# Usage
for line in ["a,b,c", "", "x,y"]:
    fields = parse_line(line)
    if fields is not None:
        print(fields)
```

**Why it matters:** Returning `None` to signal "no result" is a common pattern. The caller checks for `None` before using the result.

**Case 2: Multiple Outputs from Computation**

```python
def analyze(numbers):
    """Return count, sum, and average."""
    count = len(numbers)
    total = sum(numbers)
    average = total / count if count > 0 else 0
    return count, total, average

n, s, avg = analyze([10, 20, 30])
print(f"Count: {n}, Sum: {s}, Average: {avg}")
```

**Why it matters:** Returning multiple related values as a tuple allows a single function call to provide all necessary outputs, and tuple unpacking makes the call site clean.

---

## Core Concept 5: Function Documentation (Docstrings)

### Definitions

**Core Definition**
A docstring is a string literal that appears as the first statement in a function, providing documentation for the function's purpose, parameters, and return values.

**Technical Definition**
Per PEP 257, a docstring is a string literal that occurs as the first statement in a module, function, class, or method definition. It becomes the object's `__doc__` attribute and is used by tools like `help()`, `pydoc`, and Sphinx. Two common formatting conventions are Google style and NumPy style, each with structured sections for arguments, returns, and exceptions .

**Beginner-Friendly Explanation**
A docstring is a special comment placed right inside the function (usually at the top) that explains what the function does. It's written with triple quotes. Python stores it as the function's documentation, and you can view it with `help(my_function)`. Unlike regular comments, docstrings are accessible at runtime and are used by documentation generators.

### Purposes

- **To explain the function's purpose** to users and other developers
- **To document parameters, return values, and exceptions** in a structured way
- **To enable automated documentation generation** via tools like Sphinx and pdoc
- **To make `help()` output useful** in interactive sessions
- **To serve as a contract** between the function author and its users

### Syntax Rules and Structure

**Complete General Syntax (One-line)**

```python
def function():
    """Brief description."""
    ...
```

**Complete General Syntax (Multi-line, Google Style)**

```python
def function(param1, param2):
    """Brief description.

    Longer description if needed.

    Args:
        param1 (type): Description of param1.
        param2 (type): Description of param2.

    Returns:
        type: Description of return value.

    Raises:
        ExceptionType: Description of when raised.
    """
    ...
```

**Complete General Syntax (Multi-line, NumPy Style)**

```python
def function(param1, param2):
    """Brief description.

    Parameters
    ----------
    param1 : type
        Description of param1.
    param2 : type
        Description of param2.

    Returns
    -------
    type
        Description of return value.
    """
    ...
```

**Syntax Rules**

| Rule | Description |
|---|---|
| First statement | Must be the first statement in the function body  |
| Triple quotes | Use `"""` (or `'''`) |
| One-line: ends with period | Brief, imperative mood  |
| Multi-line: blank line after summary | Followed by detailed description  |
| Do not repeat signature | Do not restate the function's parameters in prose  |

**Google Style Sections:**

| Section | Purpose |
|---|---|
| `Args:` | Describe parameters, types, defaults |
| `Returns:` | Describe return value and type |
| `Raises:` | Describe exceptions raised |
| `Example:` | Usage examples (with `>>>` for doctest)  |

**NumPy Style Sections:**

| Section | Purpose |
|---|---|
| `Parameters` | Describe parameters (underlined with `----------`) |
| `Returns` | Describe return values |
| `Raises` | Describe exceptions |
| `Examples` | Usage examples  |

**Constraints and Limitations**

- **Only first string literal counts**: A string later in the function is not a docstring.
- **Not enforced by Python**: Docstrings are conventions; Python does not validate their presence or format.
- **Style consistency**: Mixing Google and NumPy styles in one project is discouraged .
- **Doctests must be valid**: Examples prefixed with `>>>` are executed by `doctest`; broken examples cause test failures .

### Multiple Annotated Complete Code Examples and Expected Outputs

**Example 1: One-line Docstring**

```python
def square(x):
    """Return the square of x."""
    return x * x

# Access the docstring
print(square.__doc__)
# Output: Return the square of x.

# Or via help()
help(square)
```

**Expected Output:**
```
Return the square of x.
Help on function square in module __main__:

square(x)
    Return the square of x.
```

**Breakdown:**
1. The string `"""Return the square of x."""` is the first statement, so it becomes `square.__doc__`.
2. `print(square.__doc__)` outputs the docstring.
3. `help(square)` shows the docstring along with the function signature.

**Example 2: Google Style Docstring**

```python
def divide(dividend, divisor):
    """Divide two numbers.

    Args:
        dividend (int or float): The number to be divided.
        divisor (int or float): The number to divide by.

    Returns:
        float: The result of the division.

    Raises:
        ZeroDivisionError: If divisor is zero.
    """
    return dividend / divisor

print(divide.__doc__)
```

**Expected Output:**
```
Divide two numbers.

    Args:
        dividend (int or float): The number to be divided.
        divisor (int or float): The number to divide by.

    Returns:
        float: The result of the division.

    Raises:
        ZeroDivisionError: If divisor is zero.
```

**Breakdown:**
1. The first line is a brief summary.
2. `Args:` documents each parameter with its type and description.
3. `Returns:` documents the return value.
4. `Raises:` documents exceptions .

**Example 3: NumPy Style Docstring**

```python
def divide(dividend, divisor):
    """Divide two numbers.

    Parameters
    ----------
    dividend : int or float
        The number to be divided.
    divisor : int or float
        The number to divide by.

    Returns
    -------
    float
        The result of the division.

    Raises
    ------
    ZeroDivisionError
        If divisor is zero.
    """
    return dividend / divisor

print(divide.__doc__)
```

**Expected Output:**
```
Divide two numbers.

    Parameters
    ----------
    dividend : int or float
        The number to be divided.
    divisor : int or float
        The number to divide by.

    Returns
    -------
    float
        The result of the division.

    Raises
    ------
    ZeroDivisionError
        If divisor is zero.
```

**Breakdown:**
1. `Parameters` section uses underline `----------` for emphasis.
2. Each parameter is listed with `name : type` followed by an indented description.
3. `Returns` and `Raises` follow the same underline pattern .

### Real-World Cases with Explanation

**Case 1: Library API Documentation**

```python
def fetch_data(url, timeout=30, headers=None):
    """Fetch data from a URL.

    Args:
        url (str): The URL to fetch.
        timeout (int, optional): Request timeout in seconds. Defaults to 30.
        headers (dict, optional): HTTP headers. Defaults to None.

    Returns:
        bytes: The response content.

    Raises:
        requests.RequestException: If the request fails.
    """
    pass
```

**Why it matters:** Well-documented functions make libraries usable. Sphinx and other tools parse docstrings to generate HTML documentation automatically .

**Case 2: Interactive Help**

```python
def calculate_bmi(weight_kg, height_m):
    """Calculate Body Mass Index (BMI).

    Args:
        weight_kg (float): Weight in kilograms.
        height_m (float): Height in meters.

    Returns:
        float: The BMI value (weight / height²).
    """
    return weight_kg / (height_m ** 2)

# In interactive Python:
help(calculate_bmi)
```

**Why it matters:** The `help()` function displays docstrings, making it easy for users to understand functions without leaving the interpreter.

---

## Core Concept 6: Function Signature Inspection

### Definitions

**Core Definition**
Function signature inspection is the process of examining a function's parameters, defaults, annotations, and return annotation at runtime.

**Technical Definition**
The `inspect` module provides the `signature()` function, which returns a `Signature` object representing a callable's parameters and return annotation. The `Signature` object contains `Parameter` objects, each with `name`, `kind`, `default`, and `annotation` attributes. `Parameter.kind` is an enum describing how the argument is bound (POSITIONAL_ONLY, POSITIONAL_OR_KEYWORD, VAR_POSITIONAL, KEYWORD_ONLY, VAR_KEYWORD) .

**Beginner-Friendly Explanation**
Python functions carry information about themselves: their parameter names, which ones have defaults, their types (if annotated), and what they return. The `inspect` module lets you read this information programmatically. It's useful for writing tools that validate function calls, generate documentation, or adapt behavior based on a function's interface.

### Purposes

- **To validate function calls** before invoking them
- **To generate documentation** automatically from function metadata
- **To write decorators** that need to know the wrapped function's parameters
- **To build frameworks** that adapt to user-defined functions (e.g., web frameworks mapping URL parameters to function arguments)
- **To support introspection** in interactive environments and IDEs

### Syntax Rules and Structure

**Complete General Syntax**

```python
import inspect

# Get a function's signature
sig = inspect.signature(function_name)

# Iterate over parameters
for name, param in sig.parameters.items():
    print(name, param.kind, param.default, param.annotation)

# Get the return annotation
print(sig.return_annotation)
```

**Breakdown:**
- `inspect.signature(obj)`: Returns a `Signature` object for `obj` .
- `sig.parameters`: An ordered mapping of parameter names to `Parameter` objects.
- `param.kind`: One of `Parameter.POSITIONAL_ONLY`, `POSITIONAL_OR_KEYWORD`, `VAR_POSITIONAL`, `KEYWORD_ONLY`, `VAR_KEYWORD` .
- `param.default`: The default value, or `Parameter.empty` if none.
- `param.annotation`: The type annotation, or `Parameter.empty` if none.
- `sig.return_annotation`: The return annotation, or `Signature.empty` if none.

**Syntax Rules**

| Rule | Description |
|---|---|
| `signature()` accepts any callable | Functions, methods, classes, `functools.partial` objects |
| `follow_wrapped=True` default | Unwraps decorated functions to inspect the original |
| Parameters are ordered | Positional-only, then positional-or-keyword, then var-positional, then keyword-only, then var-keyword  |
| `Parameter.empty` | Sentinel for missing default or annotation |
| `Signature` is immutable | Use `sig.replace()` to create modified copies  |

**Constraints and Limitations**

- **Built-in functions**: Some built-in functions (e.g., `print`) may not have inspectable signatures in older Python versions.
- **Decorators without `functools.wraps`**: If a decorator does not preserve `__wrapped__`, `signature()` returns the wrapper's signature, not the original's.
- **Performance**: `inspect.signature()` is relatively expensive; avoid calling it in tight loops.
- **Annotations are not enforced**: Python does not validate arguments against annotations at runtime; annotations are metadata only.

### Multiple Annotated Complete Code Examples and Expected Outputs

**Example 1: Basic Signature Inspection**

```python
import inspect

def greet(name, greeting="Hello", *args, **kwargs):
    """Greet someone."""
    pass

sig = inspect.signature(greet)
print(f"Signature: {sig}")

for name, param in sig.parameters.items():
    print(f"  {name}: kind={param.kind}, default={param.default}")
```

**Expected Output:**
```
Signature: (name, greeting='Hello', *args, **kwargs)
  name: kind=POSITIONAL_OR_KEYWORD, default=<class 'inspect._empty'>
  greeting: kind=POSITIONAL_OR_KEYWORD, default=Hello
  args: kind=VAR_POSITIONAL, default=<class 'inspect._empty'>
  kwargs: kind=VAR_KEYWORD, default=<class 'inspect._empty'>
```

**Breakdown:**
1. `inspect.signature(greet)` returns a `Signature` object.
2. `str(sig)` displays the signature in a readable format.
3. Each parameter's `kind` and `default` are printed.
4. `Parameter.empty` (displayed as `<class 'inspect._empty'>`) indicates no default .

**Example 2: Validating Arguments with `bind()`**

```python
import inspect

def divide(a, b):
    """Divide a by b."""
    return a / b

sig = inspect.signature(divide)

# Valid call
try:
    bound = sig.bind(10, 2)
    print(f"Valid: {bound.arguments}")
except TypeError as e:
    print(f"Invalid: {e}")

# Invalid call (missing argument)
try:
    bound = sig.bind(10)
except TypeError as e:
    print(f"Invalid: {e}")

# Invalid call (unexpected keyword)
try:
    bound = sig.bind(10, 2, c=3)
except TypeError as e:
    print(f"Invalid: {e}")
```

**Expected Output:**
```
Valid: {'a': 10, 'b': 2}
Invalid: missing a required argument: 'b'
Invalid: got an unexpected keyword argument 'c'
```

**Breakdown:**
1. `sig.bind(10, 2)` successfully maps arguments to parameters.
2. `sig.bind(10)` raises `TypeError` because `b` is missing.
3. `sig.bind(10, 2, c=3)` raises `TypeError` because `c` is not a parameter .

**Example 3: Extracting Parameter Information for Documentation**

```python
import inspect

def process_data(data, format="json", validate=True):
    """Process data with options.

    Args:
        data: Input data.
        format: Output format.
        validate: Whether to validate.
    """
    pass

sig = inspect.signature(process_data)

print("Parameters:")
for name, param in sig.parameters.items():
    type_info = param.annotation if param.annotation is not inspect.Parameter.empty else "no annotation"
    default_info = f"default={param.default}" if param.default is not inspect.Parameter.empty else "required"
    print(f"  {name}: {type_info}, {default_info}")

print(f"\nDocstring: {process_data.__doc__}")
```

**Expected Output:**
```
Parameters:
  data: no annotation, required
  format: no annotation, default=json
  validate: no annotation, default=True

Docstring: Process data with options.

    Args:
        data: Input data.
        format: Output format.
        validate: Whether to validate.
```

**Breakdown:**
1. The loop iterates over all parameters.
2. For each parameter, it prints the annotation (or "no annotation") and default (or "required").
3. The docstring is accessible via `__doc__` and can be combined with signature info for automated documentation .

### Real-World Cases with Explanation

**Case 1: Web Framework Routing**

```python
import inspect

def route_handler(request, user_id, action="view"):
    """Handle a route."""
    pass

# A framework might inspect the signature to know which URL parameters to extract
sig = inspect.signature(route_handler)
param_names = [name for name in sig.parameters if name != "request"]
print(param_names)
# Output: ['user_id', 'action']
```

**Why it matters:** Frameworks like Flask and Django inspect view functions to determine how to map URL components and query parameters to function arguments.

**Case 2: Decorator That Adapts to Signatures**

```python
import inspect
from functools import wraps

def log_calls(func):
    @wraps(func)
    def wrapper(*args, **kwargs):
        sig = inspect.signature(func)
        bound = sig.bind(*args, **kwargs)
        print(f"Calling {func.__name__} with {bound.arguments}")
        return func(*args, **kwargs)
    return wrapper

@log_calls
def add(a, b):
    return a + b

add(3, 4)
# Output: Calling add with {'a': 3, 'b': 4}
```

**Why it matters:** Decorators often need to know the wrapped function's parameters to log, validate, or transform arguments. `inspect.signature()` provides this information reliably.

---

## References

- Python Software Foundation. *PEP 257 – Docstring Conventions*. https://peps.python.org/pep-0257/ 
- Python Software Foundation. *inspect — Inspect live objects*. https://docs.python.org/3/library/inspect.html 
- Python Software Foundation. *PEP 362 – Function Signature Object*. https://peps.python.org/pep-0362/
- Python.org Tutor Mailing List. *parameters vs arguments*. https://mail.python.org/archives/list/tutor@python.org/thread/5FVSVIIMJRBQ2SHMTIPDCHOLNVJAJYXP/ 
- Python.org Tutor Mailing List. *Re: parameters vs arguments*. https://mail.python.org/pipermail/tutor/2009-November/072688.html 
- ReddyBytes. *Python-DSA-API-Mastery: Functions*. https://github.com/ReddyBytes/Python-DSA-API-Mastery/blob/main/01_Python_Mastery/04_functions/theory.md 
- Consiliency. *Docstring Standards Cookbook*. https://github.com/Consiliency/treesitter-chunker/blob/main/.ai-dev-kit/skills/post-impl-docs/cookbook/docstring-standards.md 
- NumPy Documentation. *numpydoc format*. https://numpydoc.readthedocs.io/en/latest/format.html 
- Python Software Foundation. *6. Modules — Python Tutorial*. https://docs.python.org/3/tutorial/modules.html 