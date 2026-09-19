# Python Logical Operators: A Comprehensive Programming Cheat Sheet

## Topic Overview

### Definitions

**Core Definition**
Logical operators are keywords (`and`, `or`, `not`) that combine or invert Boolean expressions, returning a value based on the truthiness of their operands.

**Technical Definition**
Python's logical operators are part of the language's expression grammar and are defined in the `operator` module as `operator.and_()`, `operator.or_()`, and `operator.not_()`. Unlike many other languages, Python's `and` and `or` operators do not necessarily return `True` or `False`; they return one of their operands based on short-circuit evaluation rules. The `not` operator always returns a Boolean. All three operators operate on the truthiness of objects, meaning any object can be tested for truth value.

**Beginner-Friendly Explanation**
Logical operators are the "and," "or," and "not" of programming. They let you combine conditions: "if it's sunny **and** warm, go outside." Or check alternatives: "if it's raining **or** snowing, take an umbrella." Or flip a condition: "if it's **not** a weekday, sleep in." Python evaluates these operators in a smart way called "short-circuiting" — it stops as soon as it knows the answer.

### Key Characteristics

| Characteristic | Description |
|---|---|
| **Operand Return** | `and` and `or` return one of their operands, not necessarily a Boolean |
| **Short-Circuit** | `and` stops at first falsy; `or` stops at first truthy |
| **Truthiness-Based** | Any object can be used; its truth value is determined by `__bool__` or `__len__` |
| **Precedence** | `not` > `and` > `or` (ascending priority) |
| **Associativity** | Left-to-right for `and` and `or` |
| **`not` Returns Boolean** | `not` always returns `True` or `False` |

### Prerequisites

- **Basic Python syntax**: variables, expressions, and `print()`
- **Boolean concepts**: understanding of `True` and `False`
- **Comparison operators**: `==`, `!=`, `<`, `>`, etc.
- **Conditional statements**: `if`/`else` for using logical results

### Related Programming Areas

- **Boolean Algebra**: Mathematical foundations of logic
- **Comparison Operators**: Combine with logical operators for compound conditions
- **Truthiness and Falsiness**: How Python evaluates non-Boolean objects
- **Short-Circuit Evaluation**: Optimisation and safety mechanism
- **Operator Precedence**: How logical operators interact with others
- **Conditional Expressions**: Ternary operator as an alternative to `and`/`or`

### Core Concepts / Features

1. **`and`** (Logical conjunction)
2. **`or`** (Logical disjunction)
3. **`not`** (Logical negation)
4. **Truthiness and Falsiness** (Implicit Boolean evaluation)
5. **Short-Circuit Evaluation** (Early termination)
6. **Operator Precedence** (Order of evaluation)
7. **Common Pitfalls** (Surprising behaviours)


## Core Concept 1: `and` (Logical Conjunction)

### Definitions

**Core Definition**
The `and` operator returns the first falsy operand, or the last operand if all are truthy.

**Technical Definition**
The expression `x and y` first evaluates `x`. If `x` is falsy, its value is returned. Otherwise, `y` is evaluated and its value is returned. This is a short-circuit operator: the second operand is evaluated only if the first is truthy. The operator is equivalent to `operator.and_(x, y)`.

**Beginner-Friendly Explanation**
The `and` operator says "both must be true." If the first thing is false, Python knows the whole expression is false and doesn't bother checking the second. If the first is true, it returns the second value. For example, `True and "hello"` returns `"hello"`, while `False and "hello"` returns `False`.

### Purposes

- **To require multiple conditions** to be true simultaneously
- **To guard against errors** by checking a condition before evaluating a risky expression
- **To provide default values** when the first operand is falsy
- **To chain conditions** in `if` statements
- **To return a value** based on the first falsy operand

### Syntax Rules and Structure

**Complete General Syntax**

```
x and y
```

**Breakdown:**
- `x`: First operand (evaluated always).
- `y`: Second operand (evaluated only if `x` is truthy).
- Returns: `x` if `x` is falsy; otherwise `y`.

**Syntax Rules**

| Rule | Description |
|---|---|
| Short-circuit | `y` is evaluated only if `x` is truthy |
| Returns operand | Returns the actual operand value, not a Boolean |
| Left-to-right | Multiple `and`s associate left to right |
| Precedence | Higher than `or`, lower than comparison operators |
| Associativity | Left-to-right |

**Constraints and Limitations**

- **Not a Boolean conversion**: `5 and 3` returns `3`, not `True`.
- **Unexpected types**: The result may be a non-Boolean value.
- **Short-circuit surprises**: Side effects in `y` may not execute.

### Multiple Annotated Complete Code Examples and Their Expected Outputs

**Example 1: Basic `and` Behaviour**

```python
# Step 1: Both truthy — returns the second operand
print(True and True)      # True
print(True and "hello")   # "hello"
print(5 and 3)            # 3

# Step 2: First falsy — returns the first operand
print(False and True)     # False
print(0 and "hello")      # 0
print("" and "world")     # ""

# Step 3: Chained
print(1 and 2 and 3)      # 3
print(1 and 0 and 3)      # 0
```

**Expected Output:**
```
True
hello
3
False
0

3
0
```

**Breakdown:** `and` returns the first falsy operand or the last operand. When all are truthy, the last value is returned. When a falsy value is found, it is returned immediately.

**Example 2: Guarding Against Errors**

```python
# Step 1: Safe division using short-circuit
x = 0
y = 10

# Without short-circuit, this would raise ZeroDivisionError
result = x != 0 and y / x > 1
print(f"Result: {result}")

# Step 2: Safe list access
items = []
first = items and items[0]
print(f"First: {first}")
```

**Expected Output:**
```
Result: False
First: []
```

**Breakdown:** `x != 0` is `False`, so `and` short-circuits and returns `False` without evaluating `y / x`, avoiding `ZeroDivisionError`. Similarly, `items` is an empty list (falsy), so `and` returns the empty list without attempting `items[0]`.

**Example 3: Providing Default Values**

```python
# Step 1: Using 'and' for conditional value
user_input = ""
name = user_input and user_input.strip()
print(f"Name: '{name}'")

user_input = "  Alice  "
name = user_input and user_input.strip()
print(f"Name: '{name}'")
```

**Expected Output:**
```
Name: ''
Name: 'Alice'
```

**Breakdown:** When `user_input` is empty (falsy), `and` returns the empty string. When it's truthy, `and` evaluates and returns the stripped value.

### Real-World Cases with Explanation

**Case 1: Validating Multiple Conditions**

```python
age = 25
has_license = True

if age >= 18 and has_license:
    print("Can drive")
```

**Why it matters:** `and` ensures both conditions must be met for the action to proceed.

**Case 2: Safe Attribute Access**

```python
user = {"name": "Alice", "profile": None}
bio = user and user["profile"] and user["profile"].get("bio")
print(bio)  # None — short-circuits at user["profile"]
```

**Why it matters:** Chaining `and` safely navigates nested structures without risking `TypeError`.


## Core Concept 2: `or` (Logical Disjunction)

### Definitions

**Core Definition**
The `or` operator returns the first truthy operand, or the last operand if all are falsy.

**Technical Definition**
The expression `x or y` first evaluates `x`. If `x` is truthy, its value is returned. Otherwise, `y` is evaluated and its value is returned. This is a short-circuit operator: the second operand is evaluated only if the first is falsy. The operator is equivalent to `operator.or_(x, y)`.

**Beginner-Friendly Explanation**
The `or` operator says "at least one must be true." If the first thing is true, Python returns it and skips the rest. If the first is false, it checks the second and returns that. For example, `False or "hello"` returns `"hello"`, while `True or "hello"` returns `True`.

### Purposes

- **To accept alternatives** (one condition OR another)
- **To provide fallback values** when the first operand is falsy
- **To chain multiple possible values**
- **To simplify conditional assignments**
- **To set defaults** in function arguments

### Syntax Rules and Structure

**Complete General Syntax**

```
x or y
```

**Breakdown:**
- `x`: First operand (evaluated always).
- `y`: Second operand (evaluated only if `x` is falsy).
- Returns: `x` if `x` is truthy; otherwise `y`.

**Syntax Rules**

| Rule | Description |
|---|---|
| Short-circuit | `y` is evaluated only if `x` is falsy |
| Returns operand | Returns the actual operand value |
| Left-to-right | Multiple `or`s associate left to right |
| Precedence | Lower than `and` |
| Associativity | Left-to-right |

**Constraints and Limitations**

- **Returns non-Boolean**: `5 or 3` returns `5`, not `True`.
- **Default value surprise**: `x = value or default` fails if `value` is falsy but valid (e.g., `0`).

### Multiple Annotated Complete Code Examples and Their Expected Outputs

**Example 1: Basic `or` Behaviour**

```python
# Step 1: First truthy — returns the first operand
print(True or False)      # True
print("hello" or "world") # "hello"
print(5 or 3)             # 5

# Step 2: First falsy — returns the second operand
print(False or True)      # True
print(0 or "hello")       # "hello"
print("" or "world")      # "world"

# Step 3: Chained
print(0 or "" or "third") # "third"
```

**Expected Output:**
```
True
hello
5
True
hello
world
third
```

**Breakdown:** `or` returns the first truthy operand. If all are falsy, the last value is returned.

**Example 2: Default Values**

```python
# Step 1: Using 'or' for defaults
user_name = input("Enter name (or press Enter): ") or "Guest"
print(f"Hello, {user_name}")

# Step 2: Numeric default (careful with 0)
count = 0
result = count or 10
print(f"Result: {result}")  # 10 (0 is falsy!)
```

**Expected Output (if user presses Enter):**
```
Hello, Guest
Result: 10
```

**Breakdown:** The `or` operator provides a default when the first value is falsy. Note that `0` is falsy, so `0 or 10` returns `10`, which may be surprising.

**Example 3: Fallback Chain**

```python
# Step 1: Try multiple sources for a value
config_value = None
env_value = ""
default = "fallback"

result = config_value or env_value or default
print(result)  # "fallback"
```

**Expected Output:**
```
fallback
```

**Breakdown:** `or` chains provide a series of fallbacks, returning the first truthy value or the last one.

### Real-World Cases with Explanation

**Case 1: Configuration Defaults**

```python
timeout = config.get("timeout") or 30
```

**Why it matters:** `or` provides a concise default when a configuration value is missing or falsy.

**Case 2: User Input Fallback**

```python
name = input("Name: ") or "Anonymous"
```

**Why it matters:** `or` handles empty input gracefully by providing a default.


## Core Concept 3: `not` (Logical Negation)

### Definitions

**Core Definition**
The `not` operator inverts the truth value of its operand, returning `True` for falsy values and `False` for truthy values.

**Technical Definition**
The expression `not x` returns `True` if `x` is falsy, and `False` if `x` is truthy. Unlike `and` and `or`, `not` always returns a Boolean (`True` or `False`). It is equivalent to `operator.not_(x)`. The `not` operator has lower precedence than non-Boolean operators, so `not a == b` is interpreted as `not (a == b)`.

**Beginner-Friendly Explanation**
The `not` operator flips true to false and false to true. `not True` is `False`, and `not False` is `True`. It's also useful for checking if something is empty: `not []` is `True` because an empty list is falsy.

### Purposes

- **To invert a Boolean condition**
- **To check if a value is falsy** (empty, zero, None)
- **To simplify negative conditions** (`not x` instead of `x == False`)
- **To implement guard clauses**
- **To combine with `in`** (`x not in y`)

### Syntax Rules and Structure

**Complete General Syntax**

```
not x
```

**Breakdown:**
- `x`: Any Python object.
- Returns `True` if `x` is falsy, `False` if `x` is truthy.

**Syntax Rules**

| Rule | Description |
|---|---|
| Always Boolean | Returns `True` or `False` |
| Precedence | Lower than comparison operators; higher than `and`/`or` |
| Unary | Takes one operand |
| No `__not__` | The interpreter core defines the operation, not objects |

**Constraints and Limitations**

- **Double negation**: `not not x` returns the Boolean equivalent of `x`, not `x` itself.
- **Precedence confusion**: `not a == b` is `not (a == b)`, not `(not a) == b`.

### Multiple Annotated Complete Code Examples and Their Expected Outputs

**Example 1: Basic Negation**

```python
print(not True)     # False
print(not False)    # True
print(not 0)        # True (0 is falsy)
print(not 5)        # False (5 is truthy)
print(not "")       # True (empty string is falsy)
print(not "hello")  # False
```

**Expected Output:**
```
False
True
True
False
True
False
```

**Breakdown:** `not` inverts truthiness. Falsy values become `True`; truthy values become `False`.

**Example 2: Guard Clauses**

```python
# Step 1: Check if a list is empty
items = []
if not items:
    print("List is empty")
else:
    print(f"List has {len(items)} items")

# Step 2: Check if a value is None
value = None
if not value:
    print("Value is None or falsy")
```

**Expected Output:**
```
List is empty
Value is None or falsy
```

**Breakdown:** `not` is used to check for emptiness or falsiness in guard clauses.

**Example 3: Precedence with Comparisons**

```python
a, b = 5, 5

# Step 1: 'not' binds lower than '=='
print(not a == b)    # not (a == b) → not True → False

# Step 2: With parentheses
print(not (a == b))  # Same: False

# Step 3: 'not' in membership
print(5 not in [1, 2, 3])  # True
```

**Expected Output:**
```
False
False
True
```

**Breakdown:** `not a == b` is parsed as `not (a == b)` because `==` has higher precedence than `not`. `not in` is a single operator for membership negation.

### Real-World Cases with Explanation

**Case 1: Validating Required Fields**

```python
if not username:
    raise ValueError("Username required")
```

**Why it matters:** `not` concisely checks for empty or missing values.

**Case 2: Toggling Booleans**

```python
is_active = True
is_active = not is_active  # False
```

**Why it matters:** `not` is used to toggle Boolean flags.


## Core Concept 4: Truthiness and Falsiness (Implicit Boolean Evaluation)

### Definitions

**Core Definition**
Truthiness and falsiness describe how Python evaluates non-Boolean objects as `True` or `False` in Boolean contexts.

**Technical Definition**
Any object can be tested for truth value, for use in an `if` or `while` condition or as an operand of the Boolean operations below. By default, an object is considered true unless its class defines either a `__bool__()` method that returns `False` or a `__len__()` method that returns zero. The following are considered falsy: `None`, `False`, zero of any numeric type, and empty sequences/collections (`''`, `()`, `[]`, `{}`, `set()`, `range(0)`).

**Beginner-Friendly Explanation**
Python doesn't require `True` or `False` in conditions. Empty things (empty strings, empty lists, the number 0, `None`) are treated as false. Everything else is true. This makes code shorter: `if my_list:` checks if the list is non-empty.

### Falsy Values Table

| Category | Examples |
|---|---|
| Constants | `None`, `False` |
| Zero numbers | `0`, `0.0`, `0j`, `Decimal(0)`, `Fraction(0, 1)` |
| Empty sequences | `''`, `()`, `[]` |
| Empty collections | `{}`, `set()`, `frozenset()` |
| Empty ranges | `range(0)` |

**Truthy**: Everything else, including non-zero numbers, non-empty strings/collections, and arbitrary objects.

### Multiple Annotated Complete Code Examples and Their Expected Outputs

**Example 1: Truthiness in Conditions**

```python
# Step 1: Falsy values
for value in [None, False, 0, 0.0, "", [], {}, set()]:
    if value:
        print(f"{value!r} is truthy")
    else:
        print(f"{value!r} is falsy")
```

**Expected Output:**
```
None is falsy
False is falsy
0 is falsy
0.0 is falsy
'' is falsy
[] is falsy
{} is falsy
set() is falsy
```

**Breakdown:** All empty or zero-like values are falsy.

**Example 2: Truthy Values**

```python
for value in [1, -1, "hello", [0], {"a": 1}]:
    if value:
        print(f"{value!r} is truthy")
```

**Expected Output:**
```
1 is truthy
-1 is truthy
'hello' is truthy
[0] is truthy
{'a': 1} is truthy
```

**Breakdown:** Non-zero numbers, non-empty strings, and non-empty collections are truthy.

### Real-World Cases with Explanation

**Case 1: Optional Parameters**

```python
def greet(name=None):
    if not name:
        name = "Guest"
    print(f"Hello, {name}")
```

**Why it matters:** Truthiness checks handle optional parameters without explicit `is None` comparisons.

**Case 2: Validating Input**

```python
user_input = input("Enter value: ")
if user_input:
    process(user_input)
else:
    print("No input provided")
```

**Why it matters:** Empty input is falsy, so the check is concise.


## Core Concept 5: Short-Circuit Evaluation

### Definitions

**Core Definition**
Short-circuit evaluation means `and` and `or` stop evaluating operands as soon as the final result is determined.

**Technical Definition**
When evaluating `x and y`, Python evaluates `x` first; if `x` is falsy, it returns `x` without evaluating `y`. When evaluating `x or y`, Python evaluates `x` first; if `x` is truthy, it returns `x` without evaluating `y`. This is guaranteed by the language specification and can be used to control side effects and prevent errors.

**Beginner-Friendly Explanation**
Short-circuiting is Python being lazy in a good way. For `and`, if the first thing is false, the whole thing must be false, so Python skips the second. For `or`, if the first thing is true, the whole thing must be true, so it skips the second. This can prevent errors (like dividing by zero) and save time.

### Truth Table for `and` / `or`

| `A` | `B` | `A and B` | `A or B` |
|---|---|---|---|
| Truthy | Truthy | B | A |
| Truthy | Falsy | B | A |
| Falsy | Truthy | A | B |
| Falsy | Falsy | A | B |

### Multiple Annotated Complete Code Examples and Their Expected Outputs

**Example 1: Preventing Division by Zero**

```python
x = 0
y = 10

# Safe: short-circuits when x is 0
if x != 0 and y / x > 1:
    print("Ratio > 1")
else:
    print("Cannot compute ratio")
```

**Expected Output:**
```
Cannot compute ratio
```

**Breakdown:** `x != 0` is `False`, so `and` short-circuits and does not evaluate `y / x`, avoiding `ZeroDivisionError`.

**Example 2: Demonstrating Side Effects**

```python
def side_effect(label):
    print(f"  Evaluating: {label}")
    return label

print("Test 1: False and side_effect('B')")
result = False and side_effect("B")
print(f"Result: {result}")

print("\nTest 2: True or side_effect('B')")
result = True or side_effect("B")
print(f"Result: {result}")
```

**Expected Output:**
```
Test 1: False and side_effect('B')
Result: False

Test 2: True or side_effect('B')
Result: True
```

**Breakdown:** In both cases, the side-effect function is never called because short-circuiting stops evaluation early.

**Example 3: Safe List Access**

```python
items = []

# Safe: short-circuits when list is empty
first = items and items[0]
print(f"First: {first}")

items = [10, 20, 30]
first = items and items[0]
print(f"First: {first}")
```

**Expected Output:**
```
First: []
First: 10
```

**Breakdown:** When `items` is empty (falsy), `and` returns it without attempting `items[0]`.

### Real-World Cases with Explanation

**Case 1: Guarding Against None**

```python
user = get_user()
if user and user.is_active:
    process(user)
```

**Why it matters:** Short-circuiting prevents `AttributeError` when `user` is `None`.

**Case 2: Default Values with Side Effects**

```python
config = load_config() or load_default_config()
```

**Why it matters:** If `load_config()` returns a truthy value, `load_default_config()` is never called.


## Core Concept 6: Operator Precedence

### Definitions

**Core Definition**
Operator precedence determines the order in which logical operators are evaluated relative to each other and to other operators.

**Technical Definition**
The Boolean operations `or`, `and`, and `not` are ordered by ascending priority: `or` (lowest), `and`, `not` (highest among the three). The `not` operator has lower priority than non-Boolean operators, so `not a == b` is interpreted as `not (a == b)`. Comparisons have higher precedence than Boolean operations.

**Beginner-Friendly Explanation**
Precedence is the "order of operations" for logic. `not` binds tightest, then `and`, then `or`. So `a or b and not c` means `a or (b and (not c))`. Use parentheses to make your intent clear.

### Precedence Table (Highest to Lowest)

| Level | Operators | Associativity |
|---|---|---|
| 1 | `**` | Right |
| 2 | `+x`, `-x`, `~x` | Right |
| 3 | `*`, `/`, `//`, `%` | Left |
| 4 | `+`, `-` | Left |
| 5 | `<<`, `>>` | Left |
| 6 | `&` | Left |
| 7 | `^` | Left |
| 8 | `\|` | Left |
| 9 | Comparisons (`==`, `<`, etc.) | Left |
| 10 | `not x` | Right |
| 11 | `and` | Left |
| 12 | `or` | Left |

### Multiple Annotated Complete Code Examples and Their Expected Outputs

**Example 1: Precedence in Action**

```python
# Step 1: 'and' binds tighter than 'or'
result1 = True or False and False
# True or (False and False) = True or False = True
print(f"True or False and False = {result1}")

# Step 2: 'not' binds tighter than 'and'
result2 = not False and True
# (not False) and True = True and True = True
print(f"not False and True = {result2}")

# Step 3: Comparison binds tighter than 'not'
result3 = not 5 == 5
# not (5 == 5) = not True = False
print(f"not 5 == 5 = {result3}")
```

**Expected Output:**
```
True or False and False = True
not False and True = True
not 5 == 5 = False
```

**Breakdown:** `and` has higher precedence than `or`; `not` has higher precedence than `and`; comparisons have higher precedence than `not`.

**Example 2: Using Parentheses for Clarity**

```python
a, b, c = True, False, True

# Without parentheses (confusing)
result1 = a or b and c
print(f"a or b and c = {result1}")  # a or (b and c) = True

# With parentheses (clear)
result2 = (a or b) and c
print(f"(a or b) and c = {result2}")  # (True or False) and True = True

# Different grouping, different result
result3 = a or (b and c)
print(f"a or (b and c) = {result3}")  # True or (False and True) = True
```

**Expected Output:**
```
a or b and c = True
(a or b) and c = True
a or (b and c) = True
```

**Breakdown:** Parentheses override precedence and clarify intent.

### Real-World Cases with Explanation

**Case 1: Complex Conditions**

```python
if (user.is_admin or user.is_moderator) and user.is_active:
    grant_access()
```

**Why it matters:** Parentheses ensure the intended grouping: either admin or moderator, and must be active.

**Case 2: Avoiding Precedence Bugs**

```python
# BUG: Intended (a or b) and c, but got a or (b and c)
if a or b and c:  # Wrong grouping

# FIX: Use parentheses
if (a or b) and c:  # Correct grouping
```

**Why it matters:** Precedence bugs are subtle and can lead to incorrect logic.


## Core Concept 7: Common Pitfalls

### Definitions

**Core Definition**
Common pitfalls are frequent mistakes or misunderstandings when using Python's logical operators.

**Technical Definition**
Pitfalls arise from Python's operand-returning behaviour, short-circuit evaluation, truthiness rules, and precedence. The most common mistakes include expecting `and`/`or` to return Booleans, misunderstanding short-circuit side effects, and forgetting that `0` and empty containers are falsy.

### Key Pitfalls Table

| Pitfall | Example | Result | Explanation |
|---|---|---|---|
| `and`/`or` return operands | `5 and 3` | `3` | Not `True` |
| `or` default fails for 0 | `0 or 10` | `10` | `0` is falsy |
| Short-circuit skips side effects | `False and func()` | `False` | `func()` not called |
| `not` precedence | `not a == b` | `not (a == b)` | `==` binds tighter |
| `not not x` | `not not 5` | `True` | Not `5` |
| Empty collections falsy | `[] or "default"` | `"default"` | `[]` is falsy |

### Multiple Annotated Complete Code Examples and Their Expected Outputs

**Example 1: Operand Return Confusion**

```python
# Step 1: Unexpected return values
result1 = 5 and 3
print(f"5 and 3 = {result1}")  # 3, not True

result2 = 0 or "default"
print(f"0 or 'default' = {result2}")  # "default"

# Step 2: Safe Boolean conversion
result3 = bool(5 and 3)
print(f"bool(5 and 3) = {result3}")  # True
```

**Expected Output:**
```
5 and 3 = 3
0 or 'default' = default
bool(5 and 3) = True
```

**Breakdown:** `and` and `or` return operands, not Booleans. Wrap in `bool()` if you need a Boolean.

**Example 2: The `0 or default` Trap**

```python
# Step 1: Intended default fails for 0
count = 0
result = count or 10
print(f"Result: {result}")  # 10 (probably not intended!)

# Step 2: Correct way to handle 0
result = count if count is not None else 10
print(f"Correct: {result}")  # 0
```

**Expected Output:**
```
Result: 10
Correct: 0
```

**Breakdown:** `or` treats `0` as falsy, so it returns the default. Use `is not None` for explicit checks.

**Example 3: Short-Circuit Side Effects**

```python
# Step 1: Function with side effect
def log_and_return(value):
    print(f"  Called with {value}")
    return value

# Step 2: 'and' skips second operand
print("Testing False and log_and_return('B'):")
result = False and log_and_return("B")
print(f"Result: {result}")  # No "Called with B" printed
```

**Expected Output:**
```
Testing False and log_and_return('B'):
Result: False
```

**Breakdown:** `False and ...` short-circuits, so `log_and_return` is never called. This is usually desirable but can surprise if side effects are expected.

### Real-World Cases with Explanation

**Case 1: Configuration Values**

```python
# BUG: 0 is a valid timeout but gets replaced
timeout = config.get("timeout") or 30

# FIX: Explicit None check
timeout = config.get("timeout")
if timeout is None:
    timeout = 30
```

**Why it matters:** `or` defaults fail when `0` is a valid value.

**Case 2: Boolean Contexts**

```python
# BUG: Returns "yes" or "" instead of True/False
is_valid = user_input and "yes"
print(bool(is_valid))  # True or False correctly

# FIX: Explicit Boolean
is_valid = bool(user_input)
```

**Why it matters:** `and`/`or` return operands, which may not be Booleans.


## References

- Python Software Foundation. *Built-in Types — Boolean Operations*. https://docs.python.org/3/library/stdtypes.html#boolean-operations-and-or-not
- Python Software Foundation. *Expressions — Boolean Operations*. https://docs.python.org/3/reference/expressions.html#boolean-operations
- Python Software Foundation. *operator — Standard operators as functions*. https://docs.python.org/3/library/operator.html
- Real Python. *Using the "and" Boolean Operator in Python*. https://realpython.com/python-and-operator/
- Real Python. *Using the "or" Boolean Operator in Python*. https://realpython.com/python-or-operator/
- Real Python. *Using the "not" Boolean Operator in Python*. https://realpython.com/python-not-operator/
- Python Mailing List. *Python "and" behavior*. https://mail.python.org/pipermail/python-list/2009-August/698058.html
- Python Mailing List. *Short-circuit evaluation example*. https://mail.python.org/pipermail/tutor/2007-August/056310.html
- Python Software Foundation. *Truth Value Testing*. https://docs.python.org/3/library/stdtypes.html#truth-value-testing
- Python Software Foundation. *Operator Precedence*. https://docs.python.org/3/reference/expressions.html#operator-precedence