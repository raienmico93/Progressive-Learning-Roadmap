# Python Structural Pattern Matching: A Comprehensive Programming Cheat Sheet

## Topic Overview

### Definitions

**Core Definition**
Structural pattern matching is a control-flow construct introduced in Python 3.10 that compares a subject value against a series of patterns, executing the code block of the first pattern that matches.

**Technical Definition**
The `match` statement takes a subject expression and compares it against one or more `case` patterns. Each pattern describes the structure of the expected data—its type, shape, and constituent values. Matching succeeds or fails, and on success may bind names to parts of the subject. Patterns are evaluated top-to-bottom, and only the first matching case block executes; there is no fall-through behaviour. The construct is defined by PEP 634 (Specification), PEP 635 (Motivation), and PEP 636 (Tutorial).

**Beginner-Friendly Explanation**
Pattern matching is like a supercharged `if-elif-else` chain. Instead of just comparing a value to constants, you can describe the "shape" of the data you expect. For example, "if this is a list with exactly two items," or "if this is a dictionary containing a 'name' key," or "if this is a Point object with x=0 and y=0." Python checks each pattern in order and runs the code for the first one that fits. It makes complex branching logic much cleaner and more readable.

### Key Characteristics

| Characteristic | Description |
|---|---|
| **Top-to-Bottom Evaluation** | Patterns are tested in order; the first match wins |
| **No Fall-Through** | Only one case block executes per `match` statement |
| **Structural Inspection** | Patterns can inspect the shape, type, and contents of data |
| **Name Binding** | Patterns can capture parts of the subject into variables |
| **Exhaustiveness Optional** | A wildcard (`_`) can provide a default case; without it, unmatched subjects simply skip the match block |
| **Soft Keywords** | `match` and `case` are soft keywords—they remain valid as variable names in other contexts |
| **Python 3.10+ Only** | This feature is not available in earlier Python versions |

### Prerequisites

- **Python 3.10 or later** (check with `python --version`)
- **Basic understanding of conditional statements** (`if`/`elif`/`else`)
- **Familiarity with Python data structures**: lists, tuples, dictionaries, classes
- **Understanding of type checking and comparison operators**
- **Knowledge of variable unpacking and assignment**

### Related Programming Areas

- **Functional Programming**: Pattern matching originated in functional languages like Haskell, Scala, and Rust
- **Abstract Syntax Trees**: The `ast` module uses pattern matching internally for parsing
- **Data Validation**: Validating JSON, API responses, and configuration structures
- **Command Dispatch**: Routing commands, events, or messages to handlers
- **Parser Combinators**: Building parsers that match token sequences

### Core Concepts / Features

1. **`match` and `case` Statements** (The core syntax)
2. **Literal Patterns** (Matching exact values)
3. **OR Patterns** (`|` for alternatives)
4. **Guard Clauses** (`if` conditions within cases)
5. **Sequence Patterns** (Lists and tuples)
6. **Mapping Patterns** (Dictionaries)
7. **Class/Object Patterns** (Matching by type and attributes)
8. **As-Patterns** (`as` keyword for binding)
9. **Wildcard Pattern** (`_` as fallback)
10. **Pattern-Matching Design** (When and how to use it effectively)

---

## Core Concept 1: `match` and `case` Statements

### Definitions

**Core Definition**
The `match` statement evaluates a subject expression and compares it against one or more `case` patterns, executing the first matching case block.

**Technical Definition**
The syntax is `match subject_expr: case pattern [guard]: block ...`. The `match` statement first evaluates the subject expression. If a comma is present, a tuple is constructed. Each `case` block consists of a pattern (optionally followed by a guard condition) and a suite. Patterns are evaluated top-to-bottom until one succeeds; if no pattern matches, execution continues after the `match` statement.

**Beginner-Friendly Explanation**
The `match` statement is the "question" and each `case` is a possible "answer." Python checks each answer in order until it finds one that fits the data. Once it finds a match, it runs that block and skips the rest.

### Purposes

- **To replace long `if-elif` chains** with cleaner, more declarative branching logic
- **To destructure complex data** (nested lists, dictionaries, objects) in a single expression
- **To dispatch on data shape and type** rather than just value equality
- **To bind parts of the subject** to variables for use in the case block
- **To improve code readability** by making the expected structure explicit

### Syntax Rules and Structure

**Complete General Syntax**

```
match subject:
    case pattern1 [if guard]:
        # block for pattern1
    case pattern2:
        # block for pattern2
    case _:
        # default block
```

**Breakdown:**
- `match`: Soft keyword that introduces the statement.
- `subject`: The expression whose value is matched.
- `case`: Soft keyword that introduces each pattern.
- `pattern`: Any valid pattern (literal, sequence, mapping, class, etc.).
- `[if guard]`: Optional guard condition evaluated only if the pattern matches.
- `_`: Wildcard pattern that matches anything (default case).

**Syntax Rules**

| Rule | Description |
|---|---|
| Soft keywords | `match` and `case` can still be used as variable names elsewhere |
| Colon after subject | `match subject:` requires a colon |
| Indented cases | Each `case` must be indented under the `match` |
| Colon after pattern | Each `case pattern:` requires a colon |
| No fall-through | Only one case block executes |
| Wildcard optional | If no wildcard, unmatched subjects skip the entire match block |

**Constraints and Limitations**

- **Python 3.10+ only**: Not available in earlier versions
- **Irrefutable patterns must come last**: A capture pattern (bare name) or wildcard followed by another case raises `SyntaxError`
- **No `else` clause**: The wildcard `_` serves as the default; an explicit `else` was deliberately not added
- **Literal comparison uses `is` for `True`/`False`/`None`**: `match 1` will not match `case True` even though `1 == True`

### Multiple Annotated Complete Code Examples and Their Expected Outputs

**Example 1: Basic `match-case`**

```python
# Step 1: Define a function using match-case
def handle_command(command):
    match command:
        case "quit":
            return "Exiting..."
        case "save":
            return "Saving..."
        case "load":
            return "Loading..."
        case _:
            return "Unknown command"

# Step 2: Test various inputs
print(handle_command("quit"))
print(handle_command("save"))
print(handle_command("help"))
```

**Expected Output:**
```
Exiting...
Saving...
Unknown command
```

**Breakdown:** The `match` statement evaluates `command`. The first matching case executes. `"help"` doesn't match any literal, so the wildcard `_` catches it.

**Example 2: `match-case` vs. `if-elif`**

```python
# Traditional if-elif approach
def classify_if_elif(value):
    if value == 200:
        return "OK"
    elif value == 404:
        return "Not Found"
    elif value == 500:
        return "Server Error"
    else:
        return "Unknown"

# Pattern matching approach
def classify_match(value):
    match value:
        case 200:
            return "OK"
        case 404:
            return "Not Found"
        case 500:
            return "Server Error"
        case _:
            return "Unknown"

# Both produce identical results
for code in [200, 404, 500, 418]:
    print(f"{code}: {classify_match(code)}")
```

**Expected Output:**
```
200: OK
404: Not Found
500: Server Error
418: Unknown
```

**Breakdown:** Both functions behave identically. The `match` version is more declarative and visually separates the pattern (left) from the result (right).

**Example 3: Matching a Tuple Subject**

```python
# Step 1: Match against a tuple subject
def process_point(point):
    match point:
        case (0, 0):
            return "Origin"
        case (x, 0):
            return f"On X-axis at {x}"
        case (0, y):
            return f"On Y-axis at {y}"
        case (x, y):
            return f"Point at ({x}, {y})"

# Step 2: Test various points
print(process_point((0, 0)))
print(process_point((5, 0)))
print(process_point((0, 3)))
print(process_point((2, 7)))
```

**Expected Output:**
```
Origin
On X-axis at 5
On Y-axis at 3
Point at (2, 7)
```

**Breakdown:** The subject is a tuple. Each pattern describes a different shape. The first `case (x, 0)` binds `x` to the first element, and so on.

### Real-World Cases with Explanation

**Case 1: HTTP Status Code Handling**

```python
def get_http_status_message(code):
    match code:
        case 200:
            return "OK"
        case 201:
            return "Created"
        case 301:
            return "Moved Permanently"
        case 400:
            return "Bad Request"
        case 403:
            return "Forbidden"
        case 404:
            return "Not Found"
        case 500:
            return "Internal Server Error"
        case _:
            return f"Unknown status code: {code}"
```

**Why it matters:** HTTP status handling is a classic use case for pattern matching. The declarative syntax maps codes to messages cleanly.

**Case 2: Command-Line Argument Parsing**

```python
def parse_command(args):
    match args:
        case ["--help"]:
            print_help()
        case ["--version"]:
            print_version()
        case ["run", script]:
            run_script(script)
        case ["test", *files]:
            run_tests(files)
        case _:
            print("Unknown command")
```

**Why it matters:** CLI tools often dispatch on argument patterns. Sequence patterns with starred targets (`*files`) handle variable-length arguments elegantly.

---

## Core Concept 2: Literal Patterns

### Definitions

**Core Definition**
Literal patterns match the subject against exact literal values such as numbers, strings, booleans, and `None`.

**Technical Definition**
A literal pattern corresponds to most literals in Python: numbers, strings, bytes, and the singleton constants `True`, `False`, and `None`. The comparison uses `==` for most literals, but `is` for the three singleton constants. A literal pattern succeeds if the subject compares equal to the literal.

**Beginner-Friendly Explanation**
Literal patterns are the simplest kind—they match exact values. If the subject is `200`, the pattern `case 200:` matches. If it's `"quit"`, the pattern `case "quit":` matches. It's exactly like comparing with `==`, except for `True`, `False`, and `None`, which use identity comparison.

### Purposes

- **To match exact values** like status codes, command names, or configuration flags
- **To handle enumerated constants** in a readable, declarative way
- **To replace long `if value == X` chains** with a single match statement
- **To provide a foundation for more complex patterns** (literals inside sequences and mappings)

### Syntax Rules and Structure

**Complete General Syntax**

```
match subject:
    case literal1:
        # ...
    case literal2:
        # ...
```

**Syntax Rules**

| Rule | Description |
|---|---|
| Supported types | Numbers (int, float, complex), strings, bytes, booleans, `None` |
| Comparison | `==` for most literals; `is` for `True`, `False`, `None` |
| Dotted names | Constants from modules use dotted names (e.g., `case http.OK:`) |
| No bare names | A bare name is a capture pattern, not a literal comparison |

**Constraints and Limitations**

- **`True`/`False`/`None` use identity**: `match 1` will not match `case True`
- **Constant values need dotted names**: To match a module-level constant, use `case module.CONSTANT:` or a guard
- **No expressions**: You cannot use arbitrary expressions as patterns (e.g., `case x + 1:` is invalid)

### Multiple Annotated Complete Code Examples and Their Expected Outputs

**Example 1: Literal Pattern for Numbers**

```python
# Step 1: Define a function using literal patterns
def describe_value(value):
    match value:
        case 0:
            return "Zero"
        case 1:
            return "One"
        case 42:
            return "The answer"
        case _:
            return f"Other: {value}"

# Step 2: Test values
print(describe_value(0))
print(describe_value(1))
print(describe_value(42))
print(describe_value(99))
```

**Expected Output:**
```
Zero
One
The answer
Other: 99
```

**Breakdown:** Each literal pattern matches the subject by equality. The wildcard handles everything else.

**Example 2: Matching `True`, `False`, and `None`**

```python
# Step 1: Match singleton values
def classify(value):
    match value:
        case True:
            return "Boolean true"
        case False:
            return "Boolean false"
        case None:
            return "None value"
        case _:
            return f"Other: {value!r}"

# Step 2: Test the function
print(classify(True))
print(classify(False))
print(classify(None))
print(classify(1))    # Note: 1 does NOT match True
```

**Expected Output:**
```
Boolean true
Boolean false
None value
Other: 1
```

**Breakdown:** `True`, `False`, and `None` are matched by identity (`is`), not equality. Therefore, `1` (which `== True`) does **not** match `case True`.

**Example 3: Literal Pattern with Strings**

```python
# Step 1: Define a command dispatcher
def dispatch(command):
    match command:
        case "start":
            return "Starting..."
        case "stop":
            return "Stopping..."
        case "pause":
            return "Pausing..."
        case _:
            return "Unknown command"

# Step 2: Test commands
for cmd in ["start", "stop", "pause", "restart"]:
    print(f"{cmd}: {dispatch(cmd)}")
```

**Expected Output:**
```
start: Starting...
stop: Stopping...
pause: Pausing...
restart: Unknown command
```

**Breakdown:** String literals are compared with `==`. The wildcard catches any unrecognized command.

### Real-World Cases with Explanation

**Case 1: State Machine Transitions**

```python
def next_state(current, event):
    match (current, event):
        case ("idle", "start"):
            return "running"
        case ("running", "pause"):
            return "paused"
        case ("paused", "resume"):
            return "running"
        case ("running", "stop") | ("paused", "stop"):
            return "idle"
        case _:
            return current  # No transition
```

**Why it matters:** State machines map naturally to pattern matching. Literal patterns within tuples express state-transition rules clearly.

**Case 2: Configuration Flag Handling**

```python
match mode:
    case "debug":
        level = logging.DEBUG
    case "info":
        level = logging.INFO
    case "warning":
        level = logging.WARNING
    case _:
        level = logging.ERROR
```

**Why it matters:** Configuration modes are often string constants. Literal patterns provide a clean dispatch mechanism.

---

## Core Concept 3: OR Patterns (`|`)

### Definitions

**Core Definition**
OR patterns use the pipe operator `|` to combine multiple alternative patterns, succeeding if any one of them matches.

**Technical Definition**
An OR pattern matches each of its subpatterns in turn against the subject, until one succeeds. The OR pattern succeeds if any subpattern succeeds; it fails only if all subpatterns fail. Bindings made by subpatterns are only valid if that particular subpattern succeeded.

**Beginner-Friendly Explanation**
OR patterns let you say "match this OR that OR that." Instead of writing three separate `case` blocks with the same body, you can combine them with `|`. It's like saying "if the value is A, B, or C, do this."

### Purposes

- **To combine alternative patterns** that should produce the same result
- **To reduce code duplication** by grouping related values
- **To express multiple acceptable shapes** for the same logical case
- **To improve readability** when several patterns share a common action

### Syntax Rules and Structure

**Complete General Syntax**

```
case pattern1 | pattern2 | pattern3:
    # block for any of the above
```

**Syntax Rules**

| Rule | Description |
|---|---|
| Pipe operator | `|` separates alternatives |
| Left-to-right | Subpatterns tried in order; first success wins |
| Bindings | Only the successful subpattern's bindings are valid |
| No irrefutable left | An irrefutable pattern on the left of `|` raises `SyntaxError` |
| Parentheses | May be needed for clarity when mixing with other patterns |

**Constraints and Limitations**

- **Irrefutable patterns on the left are errors**: `case x | [y]:` is a `SyntaxError` because `x` always succeeds
- **Bindings are conditional**: If the first alternative fails and the second succeeds, only the second's bindings exist
- **No common bindings**: Different alternatives cannot bind the same name with different meanings

### Multiple Annotated Complete Code Examples and Their Expected Outputs

**Example 1: OR Pattern for Weekdays**

```python
# Step 1: Classify days using OR patterns
def classify_day(day):
    match day.lower():
        case "monday" | "tuesday" | "wednesday" | "thursday" | "friday":
            return "Weekday"
        case "saturday" | "sunday":
            return "Weekend"
        case _:
            return "Unknown day"

# Step 2: Test days
for d in ["Monday", "Saturday", "Sunday", "Funday"]:
    print(f"{d}: {classify_day(d)}")
```

**Expected Output:**
```
Monday: Weekday
Saturday: Weekend
Sunday: Weekend
Funday: Unknown day
```

**Breakdown:** The OR pattern combines five weekdays into one case and two weekend days into another. This reduces six separate cases to two.

**Example 2: OR Pattern with Sequence Patterns**

```python
# Step 1: Match multiple sequence shapes
def handle_command(args):
    match args:
        case ["quit"] | ["exit"] | ["q"]:
            return "Quitting..."
        case ["help"] | ["h"] | ["?"]:
            return "Showing help..."
        case _:
            return "Unknown"

# Step 2: Test commands
for cmd in [["quit"], ["exit"], ["q"], ["help"], ["?"], ["run"]]:
    print(f"{cmd}: {handle_command(cmd)}")
```

**Expected Output:**
```
['quit']: Quitting...
['exit']: Quitting...
['q']: Quitting...
['help']: Showing help...
['?']: Showing help...
['run']: Unknown
```

**Breakdown:** OR patterns work with any pattern type, including sequences. Each alternative is a full pattern in its own right.

**Example 3: OR Pattern with Bindings**

```python
# Step 1: Match different shapes and bind a common name
def process(value):
    match value:
        case (1, x) | (x, 1):
            return f"First or second element is 1, other is {x}"
        case (x, y):
            return f"Neither is 1: {x}, {y}"

# Step 2: Test
print(process((1, 5)))
print(process((7, 1)))
print(process((2, 3)))
```

**Expected Output:**
```
First or second element is 1, other is 5
First or second element is 1, other is 7
Neither is 1: 2, 3
```

**Breakdown:** The OR pattern `(1, x) | (x, 1)` binds `x` in both alternatives. Only the binding from the successful alternative is valid.

### Real-World Cases with Explanation

**Case 1: Accepting Multiple Command Aliases**

```python
match command:
    case "start" | "begin" | "go":
        start_game()
    case "stop" | "end" | "quit":
        end_game()
```

**Why it matters:** CLI tools and games often accept multiple aliases for the same command. OR patterns handle this without duplicating the action code.

**Case 2: Matching Multiple Data Shapes**

```python
match response:
    case {"status": "ok", "data": data} | {"status": "success", "data": data}:
        process(data)
    case {"error": msg}:
        handle_error(msg)
```

**Why it matters:** APIs may return slightly different shapes for the same logical outcome. OR patterns unify them under one handler.

---

## Core Concept 4: Guard Clauses (`if` Conditions in Cases)

### Definitions

**Core Definition**
A guard clause is an optional `if` condition attached to a `case` pattern that must evaluate to true for the case to match.

**Technical Definition**
After a pattern successfully matches the subject, the guard expression is evaluated in the context of the bindings made by the pattern. If the guard is true, the case block executes; if false, pattern matching continues with the next case. Guards allow additional, non-structural conditions to be imposed on a match.

**Beginner-Friendly Explanation**
A guard is like an extra filter. The pattern says "this shape fits," and the guard says "and this extra condition must be true." For example, `case Point(x, y) if x == y:` means "match a Point, and only proceed if its x and y are equal."

### Purposes

- **To add value-based conditions** to structural matches
- **To filter matches** based on relationships between bound variables
- **To express complex predicates** that cannot be captured by pattern structure alone
- **To reuse the same pattern** with different guard conditions
- **To check computed properties** of the matched data

### Syntax Rules and Structure

**Complete General Syntax**

```
case pattern if guard_expression:
    # block executed only if pattern matches AND guard is true
```

**Syntax Rules**

| Rule | Description |
|---|---|
| Position | Guard follows the pattern, preceded by `if` |
| Evaluation | Guard evaluated only after pattern succeeds |
| Bindings available | Variables bound by the pattern are accessible in the guard |
| Fall-through | If guard is false, matching continues to the next case |
| Any expression | Guard can be any valid Python expression |

**Constraints and Limitations**

- **Guard is not a pattern**: Guards use regular expressions, not pattern syntax
- **No assignment in guard**: Use the walrus operator (`:=`) if needed, but sparingly
- **Performance**: Guards are evaluated for every successful pattern match, adding overhead

### Multiple Annotated Complete Code Examples and Their Expected Outputs

**Example 1: Guard with Class Pattern**

```python
# Step 1: Define a Point class
class Point:
    def __init__(self, x, y):
        self.x = x
        self.y = y

# Step 2: Use a guard to check diagonal
def classify_point(point):
    match point:
        case Point(x, y) if x == y:
            return f"On diagonal (x==y=={x})"
        case Point(x, y):
            return f"Not on diagonal: ({x}, {y})"

# Step 3: Test points
print(classify_point(Point(3, 3)))
print(classify_point(Point(2, 5)))
```

**Expected Output:**
```
On diagonal (x==y==3)
Not on diagonal: (2, 5)
```

**Breakdown:** The first case matches any Point but only succeeds if `x == y`. If the guard fails, the second case (without a guard) matches.

**Example 2: Guard with Literal Pattern**

```python
# Step 1: Match with a range check
def categorize_score(score):
    match score:
        case n if n >= 90:
            return "A"
        case n if n >= 80:
            return "B"
        case n if n >= 70:
            return "C"
        case n:
            return "F"

# Step 2: Test scores
for s in [95, 85, 75, 65]:
    print(f"{s}: {categorize_score(s)}")
```

**Expected Output:**
```
95: A
85: B
75: C
65: F
```

**Breakdown:** The pattern `n` is a capture pattern that always succeeds. The guard provides the actual condition. Order matters—the first guard that passes wins.

**Example 3: Guard with Sequence Pattern**

```python
# Step 1: Match sequences with guards
def process_pair(pair):
    match pair:
        case (x, y) if x > y:
            return f"Descending: {x} > {y}"
        case (x, y) if x < y:
            return f"Ascending: {x} < {y}"
        case (x, y):
            return f"Equal: {x} == {y}"

# Step 2: Test pairs
print(process_pair((5, 3)))
print(process_pair((2, 8)))
print(process_pair((4, 4)))
```

**Expected Output:**
```
Descending: 5 > 3
Ascending: 2 < 8
Equal: 4 == 4
```

**Breakdown:** The same structural pattern `(x, y)` is used three times with different guards. Only one guard can be true for any given pair.

### Real-World Cases with Explanation

**Case 1: Validation with Business Rules**

```python
match order:
    case {"items": items, "total": total} if total > 0 and len(items) > 0:
        process_order(order)
    case {"items": [], "total": total}:
        raise ValueError("Empty order")
    case _:
        raise ValueError("Invalid order format")
```

**Why it matters:** Guards enforce business rules (e.g., "total must be positive," "items must not be empty") while keeping the structural matching logic clean.

**Case 2: User Permission Checks**

```python
match request:
    case Request(user=user, resource=res) if user.is_admin:
        grant_access(res)
    case Request(user=user, resource=res) if user.has_permission(res):
        grant_access(res)
    case _:
        deny_access()
```

**Why it matters:** Guards allow permission logic to be expressed declaratively alongside the structural pattern, making security rules easy to audit.

---

## Core Concept 5: Sequence Patterns (Lists/Tuples)

### Definitions

**Core Definition**
Sequence patterns match the subject against a sequence (list or tuple) by checking its length, type, and the patterns of its elements.

**Technical Definition**
A sequence pattern consists of a sequence of subpatterns enclosed in `[]` or `()`. It succeeds if the subject is a sequence (not a string, bytes, or bytearray), its length matches the number of subpatterns (unless a starred subpattern is present), and each subpattern matches the corresponding element. A starred subpattern (`*name`) captures remaining elements as a list.

**Beginner-Friendly Explanation**
Sequence patterns describe the shape of a list or tuple. `case [x, y]:` matches a two-element sequence. `case [first, *rest]:` matches any sequence with at least one element, binding the first to `first` and the remainder to `rest`.

### Purposes

- **To destructure lists and tuples** into named components
- **To match sequences of specific length** (exactly two, exactly three, etc.)
- **To capture variable-length sequences** using starred patterns
- **To validate data structures** like coordinates, RGB values, or argument lists
- **To replace manual indexing** (`data[0]`, `data[1]`) with descriptive names

### Syntax Rules and Structure

**Complete General Syntax**

```
case [pattern1, pattern2, ...]:
    # matches a sequence of exact length
case [first, *rest]:
    # matches a sequence with at least one element
case (pattern1, pattern2):
    # tuple syntax (equivalent to list syntax)
```

**Syntax Rules**

| Rule | Description |
|---|---|
| List or tuple syntax | `[]` and `()` are equivalent |
| Exact length | Number of subpatterns must match sequence length (unless starred) |
| Starred subpattern | `*name` captures remaining elements as a list |
| Only one star | At most one starred subpattern per sequence |
| Excludes strings | Strings and bytes are not matched by sequence patterns |

**Constraints and Limitations**

- **Strings are not sequences** for pattern matching: `case [a, b]:` will not match `"ab"`
- **Starred pattern position**: Can be anywhere in the sequence, but only one is allowed
- **Length must match exactly** (without star): `[x, y]` will not match a 3-element sequence

### Multiple Annotated Complete Code Examples and Their Expected Outputs

**Example 1: Exact-Length Sequence Pattern**

```python
# Step 1: Match a 2D coordinate
def describe_coordinate(coord):
    match coord:
        case [x, y]:
            return f"2D point: ({x}, {y})"
        case [x, y, z]:
            return f"3D point: ({x}, {y}, {z})"
        case _:
            return "Unknown coordinate"

# Step 2: Test coordinates
print(describe_coordinate([1, 2]))
print(describe_coordinate([3, 4, 5]))
print(describe_coordinate([1]))
```

**Expected Output:**
```
2D point: (1, 2)
3D point: (3, 4, 5)
Unknown coordinate
```

**Breakdown:** The pattern `[x, y]` matches a 2-element sequence; `[x, y, z]` matches a 3-element sequence. A 1-element sequence falls through to the wildcard.

**Example 2: Starred Sequence Pattern**

```python
# Step 1: Capture first and rest
def process_list(items):
    match items:
        case []:
            return "Empty list"
        case [first, *rest]:
            return f"First: {first}, Rest: {rest}"

# Step 2: Test
print(process_list([]))
print(process_list([1]))
print(process_list([1, 2, 3, 4]))
```

**Expected Output:**
```
Empty list
First: 1, Rest: []
First: 1, Rest: [2, 3, 4]
```

**Breakdown:** `*rest` captures all remaining elements as a list. It can match zero or more elements, so `[first, *rest]` matches any non-empty sequence.

**Example 3: Nested Sequence Patterns**

```python
# Step 1: Match nested structures
def process_matrix_row(row):
    match row:
        case [[a, b], [c, d]]:
            return f"2x2 matrix: {a}, {b}, {c}, {d}"
        case [x, y]:
            return f"Flat pair: {x}, {y}"
        case _:
            return "Unknown"

# Step 2: Test
print(process_matrix_row([[1, 2], [3, 4]]))
print(process_matrix_row([5, 6]))
```

**Expected Output:**
```
2x2 matrix: 1, 2, 3, 4
Flat pair: 5, 6
```

**Breakdown:** Sequence patterns can nest. `[[a, b], [c, d]]` matches a 2-element sequence where each element is itself a 2-element sequence.

### Real-World Cases with Explanation

**Case 1: RGB Colour Parsing**

```python
match colour:
    case [r, g, b]:
        return f"RGB: ({r}, {g}, {b})"
    case [r, g, b, a]:
        return f"RGBA: ({r}, {g}, {b}, {a})"
    case _:
        raise ValueError("Invalid colour format")
```

**Why it matters:** Colour data is naturally expressed as sequences of 3 (RGB) or 4 (RGBA) components. Sequence patterns validate and destructure in one step.

**Case 2: Command Argument Parsing**

```python
match args:
    case ["--verbose", *files]:
        verbose = True
        process(files)
    case files:
        verbose = False
        process(files)
```

**Why it matters:** CLI arguments are lists of strings. Starred patterns handle flags combined with variable-length file lists elegantly.

---

## Core Concept 6: Mapping Patterns (Dictionaries)

### Definitions

**Core Definition**
Mapping patterns match the subject against a dictionary by checking for the presence of specified keys and matching their values against subpatterns.

**Technical Definition**
A mapping pattern consists of key-value pairs enclosed in `{}`. It succeeds if the subject is a mapping (e.g., `dict`), all specified keys are present, and each key's value matches the corresponding subpattern. Additional keys in the subject are allowed and ignored. A `**rest` pattern can capture remaining key-value pairs.

**Beginner-Friendly Explanation**
Mapping patterns describe the shape of a dictionary. `case {"name": name}:` matches any dictionary that has a `"name"` key, binding its value to `name`. Other keys in the dictionary don't matter—mapping patterns only check the keys you specify.

### Purposes

- **To destructure dictionaries** by extracting values for specific keys
- **To validate that required keys are present** in a dictionary
- **To match JSON-like data structures** (API responses, configuration files)
- **To ignore irrelevant keys** and focus only on the ones that matter
- **To capture remaining keys** with `**rest` for further processing

### Syntax Rules and Structure

**Complete General Syntax**

```
case {"key1": pattern1, "key2": pattern2}:
    # matches if subject has key1 and key2 with matching values
case {**rest}:
    # captures all key-value pairs into rest
```

**Syntax Rules**

| Rule | Description |
|---|---|
| Mapping type | Subject must be a mapping (dict, etc.) |
| Extra keys allowed | Keys not mentioned in the pattern are ignored |
| Required keys | All specified keys must be present |
| Value patterns | Values can be any pattern (literal, sequence, nested mapping, etc.) |
| `**rest` | Captures remaining key-value pairs as a dict |

**Constraints and Limitations**

- **Extra keys are allowed** (unlike sequence patterns, which require exact length)
- **Keys must be literals**: Keys in mapping patterns cannot be variables
- **`**rest` must be last**: It must appear after all other key-value pairs

### Multiple Annotated Complete Code Examples and Their Expected Outputs

**Example 1: Basic Mapping Pattern**

```python
# Step 1: Extract values from a dict
def process_user(user):
    match user:
        case {"name": name, "age": age}:
            return f"{name} is {age} years old"
        case {"name": name}:
            return f"{name} (age unknown)"
        case _:
            return "Invalid user"

# Step 2: Test
print(process_user({"name": "Alice", "age": 30}))
print(process_user({"name": "Bob"}))
print(process_user({"id": 123}))
```

**Expected Output:**
```
Alice is 30 years old
Bob (age unknown)
Invalid user
```

**Breakdown:** The first pattern requires both `"name"` and `"age"`. The second only requires `"name"`. Extra keys are ignored.

**Example 2: Mapping Pattern with `**rest`**

```python
# Step 1: Capture remaining keys
def extract_config(config):
    match config:
        case {"host": host, "port": port, **rest}:
            return f"Connecting to {host}:{port}, extras: {rest}"
        case _:
            return "Invalid config"

# Step 2: Test
print(extract_config({"host": "localhost", "port": 8080}))
print(extract_config({"host": "localhost", "port": 8080, "timeout": 30}))
```

**Expected Output:**
```
Connecting to localhost:8080, extras: {}
Connecting to localhost:8080, extras: {'timeout': 30}
```

**Breakdown:** `**rest` captures all keys not explicitly mentioned. If none remain, `rest` is an empty dict.

**Example 3: Nested Mapping and Sequence Patterns**

```python
# Step 1: Match nested JSON-like data
def process_event(event):
    match event:
        case {"type": "click", "position": [x, y]}:
            return f"Click at ({x}, {y})"
        case {"type": "keypress", "key": key}:
            return f"Key pressed: {key}"
        case {"type": "scroll", "direction": direction, "amount": amount}:
            return f"Scrolled {direction} by {amount}"
        case _:
            return "Unknown event"

# Step 2: Test
print(process_event({"type": "click", "position": [100, 200]}))
print(process_event({"type": "keypress", "key": "Enter"}))
print(process_event({"type": "scroll", "direction": "up", "amount": 5}))
```

**Expected Output:**
```
Click at (100, 200)
Key pressed: Enter
Scrolled up by 5
```

**Breakdown:** Mapping patterns nest with sequence patterns. The value for `"position"` is matched against `[x, y]`, extracting coordinates.

### Real-World Cases with Explanation

**Case 1: API Response Handling**

```python
match response:
    case {"status": "success", "data": {"users": [*users]}}:
        process_users(users)
    case {"status": "error", "message": msg}:
        handle_error(msg)
    case _:
        raise ValueError("Unexpected response format")
```

**Why it matters:** API responses are often nested JSON objects. Mapping patterns validate the structure and extract the relevant data in one declarative statement.

**Case 2: Configuration Validation**

```python
match config:
    case {"database": {"host": host, "port": port}}:
        connect_db(host, port)
    case {"database": {"host": host}}:
        connect_db(host, 5432)  # default port
    case _:
        raise ConfigError("Missing database configuration")
```

**Why it matters:** Configuration files must be validated for required keys. Mapping patterns express the expected schema clearly and provide fallbacks.

---

## Core Concept 7: Class/Object Patterns

### Definitions

**Core Definition**
Class patterns match the subject against instances of a specific class, checking the class type and optionally matching attributes against subpatterns.

**Technical Definition**
A class pattern uses the syntax `ClassName(pattern1, pattern2, ..., attr1=pattern3, ...)`. It succeeds if the subject is an instance of the named class (or its subclass) and the positional and keyword subpatterns match the corresponding attributes. Positional subpatterns map to `__match_args__` attributes; keyword subpatterns map to named attributes.

**Beginner-Friendly Explanation**
Class patterns say "if this object is an instance of this class, and its attributes match these patterns, then proceed." They let you destructure objects in a type-safe, declarative way.

### Purposes

- **To dispatch on object type** (e.g., different handling for different classes)
- **To extract attributes** from objects into variables
- **To validate object state** by matching attribute values
- **To implement visitor-like patterns** without explicit method dispatch
- **To work with dataclasses and named tuples** naturally

### Syntax Rules and Structure

**Complete General Syntax**

```
case ClassName(pattern1, pattern2):
    # positional attributes
case ClassName(attr1=pattern3, attr2=pattern4):
    # keyword attributes
case ClassName(pattern1, attr2=pattern3):
    # mixed positional and keyword
```

**Syntax Rules**

| Rule | Description |
|---|---|
| Class name | Must be a dotted name (e.g., `Point`, `module.ClassName`) |
| `__match_args__` | Positional subpatterns map to attributes listed in this class attribute |
| Keyword subpatterns | Match named attributes directly |
| Type check | Subject must be an instance of the class or its subclass |
| No bare class name | `case Point:` matches any Point without destructuring |

**Constraints and Limitations**

- **`__match_args__` required for positional patterns**: Without it, positional subpatterns raise `TypeError`
- **Keyword patterns require attribute access**: The attribute must exist on the class
- **Does not call `__eq__`**: Class patterns check type and attributes, not equality
- **Dataclasses automatically provide `__match_args__`**: Convenient for pattern matching

### Multiple Annotated Complete Code Examples and Their Expected Outputs

**Example 1: Basic Class Pattern**

```python
# Step 1: Define a class with __match_args__
class Point:
    __match_args__ = ("x", "y")

    def __init__(self, x, y):
        self.x = x
        self.y = y

# Step 2: Match Point instances
def describe(point):
    match point:
        case Point(0, 0):
            return "Origin"
        case Point(x, 0):
            return f"X-axis at {x}"
        case Point(0, y):
            return f"Y-axis at {y}"
        case Point(x, y):
            return f"Point ({x}, {y})"

# Step 3: Test
print(describe(Point(0, 0)))
print(describe(Point(5, 0)))
print(describe(Point(0, 3)))
print(describe(Point(2, 7)))
```

**Expected Output:**
```
Origin
X-axis at 5
Y-axis at 3
Point (2, 7)
```

**Breakdown:** `__match_args__` defines that positional subpatterns map to `x` and `y`. The patterns destructure the Point by these attributes.

**Example 2: Class Pattern with Keyword Attributes**

```python
# Step 1: Match by keyword attribute
class Circle:
    def __init__(self, radius, center=None):
        self.radius = radius
        self.center = center

def describe_circle(c):
    match c:
        case Circle(radius=r, center=None):
            return f"Circle at origin with radius {r}"
        case Circle(radius=r, center=(x, y)):
            return f"Circle at ({x}, {y}) with radius {r}"
        case _:
            return "Not a circle"

# Step 2: Test
print(describe_circle(Circle(5)))
print(describe_circle(Circle(3, (1, 2))))
```

**Expected Output:**
```
Circle at origin with radius 5
Circle at (1, 2) with radius 3
```

**Breakdown:** Keyword patterns (`radius=r`) match named attributes directly, without relying on `__match_args__`.

**Example 3: Class Pattern with Dataclasses**

```python
from dataclasses import dataclass

# Step 1: Define a dataclass (automatically provides __match_args__)
@dataclass
class Point3D:
    x: float
    y: float
    z: float

# Step 2: Match dataclass instances
def classify(p):
    match p:
        case Point3D(0, 0, 0):
            return "Origin"
        case Point3D(x, y, 0):
            return f"2D point ({x}, {y})"
        case Point3D(x, 0, z):
            return f"XZ-plane ({x}, {z})"
        case Point3D(x, y, z):
            return f"3D point ({x}, {y}, {z})"

# Step 3: Test
print(classify(Point3D(0, 0, 0)))
print(classify(Point3D(1, 2, 0)))
print(classify(Point3D(3, 4, 5)))
```

**Expected Output:**
```
Origin
2D point (1, 2)
3D point (3, 4, 5)
```

**Breakdown:** Dataclasses automatically generate `__match_args__`, making them ideal for pattern matching without extra boilerplate.

### Real-World Cases with Explanation

**Case 1: AST Node Processing**

```python
import ast

def process_node(node):
    match node:
        case ast.Constant(value=int(v)):
            return f"Integer: {v}"
        case ast.Constant(value=str(s)):
            return f"String: {s!r}"
        case ast.Name(id=name):
            return f"Variable: {name}"
        case ast.BinOp(left=l, op=ast.Add(), right=r):
            return f"Addition: {process_node(l)} + {process_node(r)}"
```

**Why it matters:** Abstract syntax trees are hierarchies of typed nodes. Class patterns dispatch on node type and extract attributes cleanly, which is far more readable than `isinstance` chains.

**Case 2: Geometry Calculations**

```python
@dataclass
class Rectangle:
    width: float
    height: float

@dataclass
class Circle:
    radius: float

def area(shape):
    match shape:
        case Rectangle(w, h):
            return w * h
        case Circle(r):
            return 3.14159 * r ** 2
        case _:
            raise ValueError("Unknown shape")
```

**Why it matters:** Geometry objects naturally map to class patterns. Each shape's area formula is associated with its type, making the dispatch logic transparent.

---

## Core Concept 8: As-Patterns (`as` Keyword for Binding)

### Definitions

**Core Definition**
An as-pattern binds the entire matched value (or a subpattern) to a name using the `as` keyword, allowing both structural matching and whole-value capture simultaneously.

**Technical Definition**
The syntax is `pattern as name`. The subpattern is matched against the subject; if it succeeds, the entire subject value (or the portion matched by the subpattern) is bound to `name`. As-patterns are useful when you need to both destructure a value and keep a reference to the whole. The walrus pattern (`v := p`) was changed to `as` patterns (`p as v`) during PEP development.

**Beginner-Friendly Explanation**
An as-pattern says "match this structure, and also save the whole thing under this name." For example, `case [x, y] as point:` matches a two-element sequence, binds `x` and `y` to the elements, and also binds `point` to the entire list.

### Purposes

- **To bind the entire matched value** while also destructuring it
- **To capture a subpattern's value** for later use in the case block
- **To combine structural matching with whole-value references**
- **To simplify code** that would otherwise need to reconstruct the value from parts
- **To support nested as-patterns** for complex destructuring

### Syntax Rules and Structure

**Complete General Syntax**

```
case pattern as name:
    # name is bound to the entire subject (or subpattern value)
```

**Syntax Rules**

| Rule | Description |
|---|---|
| Position | `as name` follows the pattern |
| Binding | Binds the value matched by the preceding pattern |
| Precedence | As-pattern has low precedence; use parentheses when needed |
| Nested as-patterns | Can appear inside other patterns (sequences, mappings, classes) |
| Name must be valid identifier | The bound name follows Python naming rules |

**Constraints and Limitations**

- **`as` is not a pattern itself**: It modifies the preceding pattern
- **Binding is unconditional on success**: If the subpattern matches, the binding is made
- **Cannot bind to `_`**: The wildcard is not a valid binding target

### Multiple Annotated Complete Code Examples and Their Expected Outputs

**Example 1: Basic As-Pattern**

```python
# Step 1: Capture the whole sequence and its elements
def process(data):
    match data:
        case [x, y] as point:
            return f"Point {point} has coordinates ({x}, {y})"
        case _:
            return "Not a pair"

# Step 2: Test
print(process([3, 4]))
print(process([1, 2, 3]))
```

**Expected Output:**
```
Point [3, 4] has coordinates (3, 4)
Not a pair
```

**Breakdown:** `[x, y] as point` matches a 2-element sequence, binds `x` and `y` to the elements, and binds `point` to the entire list.

**Example 2: As-Pattern with Class**

```python
# Step 1: Define a class
class User:
    __match_args__ = ("name", "role")
    def __init__(self, name, role):
        self.name = name
        self.role = role

# Step 2: Use as-pattern to keep the whole object
def handle(user):
    match user:
        case User(name, "admin") as admin:
            return f"Admin {admin.name} has full access"
        case User(name, role):
            return f"User {name} has role {role}"

# Step 3: Test
print(handle(User("Alice", "admin")))
print(handle(User("Bob", "editor")))
```

**Expected Output:**
```
Admin Alice has full access
User Bob has role editor
```

**Breakdown:** The as-pattern `User(name, "admin") as admin` binds the entire User object to `admin` while also extracting the `name` attribute.

**Example 3: Nested As-Patterns**

```python
# Step 1: Use as-patterns inside a mapping pattern
def process_config(config):
    match config:
        case {"database": {"host": host, "port": port} as db_config}:
            return f"DB config {db_config}: {host}:{port}"
        case _:
            return "No DB config"

# Step 2: Test
print(process_config({"database": {"host": "localhost", "port": 5432}}))
```

**Expected Output:**
```
DB config {'host': 'localhost', 'port': 5432}: localhost:5432
```

**Breakdown:** The as-pattern `as db_config` captures the inner dictionary while also extracting `host` and `port`.

### Real-World Cases with Explanation

**Case 1: Preserving Context While Destructuring**

```python
match event:
    case {"type": "click", "position": [x, y]} as click_event:
        log_event(click_event)
        handle_click(x, y)
```

**Why it matters:** The as-pattern preserves the full event dictionary for logging while extracting coordinates for processing. Without it, you'd need to reconstruct the dictionary.

**Case 2: Pattern Matching in Recursive Functions**

```python
def evaluate(expr):
    match expr:
        case ("add", left, right) as node:
            return evaluate(left) + evaluate(right)
        case ("mul", left, right) as node:
            return evaluate(left) * evaluate(right)
        case int(n):
            return n
```

**Why it matters:** As-patterns allow recursive functions to reference the current node while also destructuring its children.

---

## Core Concept 9: Wildcard Pattern (`_`)

### Definitions

**Core Definition**
The wildcard pattern `_` matches any value without binding it, serving as the default or catch-all case.

**Technical Definition**
The wildcard pattern is written as a single underscore `_`. It always succeeds and does not bind any name. It is typically used as the final case to handle any subject not matched by previous patterns. The use of `_` is optional; without it, unmatched subjects simply skip the match block.

**Beginner-Friendly Explanation**
The wildcard `_` is the "everything else" case. It's like the `else` in an `if-elif-else` chain. If none of the specific patterns match, the wildcard catches the value. Unlike a capture pattern (a bare name), the wildcard doesn't create a variable—it just accepts anything.

### Purposes

- **To provide a default case** when no other pattern matches
- **To avoid binding unnecessary names** when the value is not needed
- **To make the match statement exhaustive** by handling all possible subjects
- **To explicitly document** that a case is intentionally ignored
- **To replace the missing `else` clause** in pattern matching

### Syntax Rules and Structure

**Complete General Syntax**

```
case _:
    # default block
```

**Syntax Rules**

| Rule | Description |
|---|---|
| Always succeeds | `_` matches any subject |
| No binding | `_` does not create a variable |
| Typically last | Should be the final case (an irrefutable pattern before another case raises `SyntaxError`) |
| Optional | Not required; without it, unmatched subjects skip the match |
| Can appear in subpatterns | `case [_, x]:` matches any 2-element sequence, ignoring the first |

**Constraints and Limitations**

- **Irrefutable**: A wildcard case must be last; another case after it raises `SyntaxError`
- **Does not bind**: You cannot reference `_` in the case block (it's not a variable)
- **Optional**: Unlike `else` in `if-else`, the wildcard is not mandatory

### Multiple Annotated Complete Code Examples and Their Expected Outputs

**Example 1: Basic Wildcard as Default**

```python
# Step 1: Handle known commands with a wildcard fallback
def handle_command(cmd):
    match cmd:
        case "start":
            return "Starting..."
        case "stop":
            return "Stopping..."
        case _:
            return "Unknown command"

# Step 2: Test
print(handle_command("start"))
print(handle_command("stop"))
print(handle_command("restart"))
```

**Expected Output:**
```
Starting...
Stopping...
Unknown command
```

**Breakdown:** The wildcard catches `"restart"` because it doesn't match the specific literals.

**Example 2: Wildcard in Subpatterns**

```python
# Step 1: Ignore specific elements
def process_pair(pair):
    match pair:
        case [_, second]:
            return f"Second element: {second}"
        case _:
            return "Not a pair"

# Step 2: Test
print(process_pair([99, 42]))
print(process_pair([1, 2, 3]))
```

**Expected Output:**
```
Second element: 42
Not a pair
```

**Breakdown:** The wildcard `_` in `[_, second]` matches the first element without binding it, while `second` captures the second element.

**Example 3: Wildcard with Class Patterns**

```python
# Step 1: Match any Point but ignore its values
class Point:
    __match_args__ = ("x", "y")
    def __init__(self, x, y):
        self.x = x
        self.y = y

def is_point(value):
    match value:
        case Point(_, _):
            return "This is a Point"
        case _:
            return "Not a Point"

# Step 2: Test
print(is_point(Point(1, 2)))
print(is_point("hello"))
```

**Expected Output:**
```
This is a Point
Not a Point
```

**Breakdown:** `Point(_, _)` matches any Point instance without binding its coordinates. It's a type check with structural confirmation.

### Real-World Cases with Explanation

**Case 1: Safe Default in Event Handling**

```python
match event:
    case {"type": "click", "position": pos}:
        handle_click(pos)
    case {"type": "keypress", "key": key}:
        handle_key(key)
    case _:
        log_unknown_event(event)
```

**Why it matters:** The wildcard ensures no event goes unhandled. Unknown events are logged rather than silently dropped.

**Case 2: Ignoring Unnecessary Data**

```python
match record:
    case {"id": _, "name": name, "email": email}:
        save_user(name, email)
```

**Why it matters:** The wildcard explicitly documents that the `"id"` key is present but intentionally ignored, making the code's intent clear.

---

## Core Concept 10: Pattern-Matching Design

### Definitions

**Core Definition**
Pattern-matching design refers to the principles, practices, and considerations for effectively using structural pattern matching in Python programs.

**Technical Definition**
Effective pattern-matching design involves choosing when to use `match` statements versus alternatives (`if-elif`, dictionaries, polymorphism), how to structure patterns for readability and maintainability, understanding the soft-keyword nature of `match`/`case`, and avoiding common pitfalls such as irrefutable patterns in the wrong position, binding errors, and over-nesting.

**Beginner-Friendly Explanation**
Pattern matching is a powerful tool, but like any tool, it's not always the right choice. This section covers when to use it, when to avoid it, and how to write pattern-matching code that is clean, correct, and maintainable.

### Purposes

- **To choose the right tool** for branching logic (match vs. if-elif vs. polymorphism)
- **To write readable, maintainable** pattern-matching code
- **To avoid common pitfalls** that lead to bugs or confusing behaviour
- **To structure patterns** for clarity and extensibility
- **To understand the performance characteristics** of pattern matching

### Guidelines and Best Practices

**When to Use Pattern Matching**

| Use Case | Example |
|---|---|
| **Dispatching on data shape** | API response handling, event processing |
| **Destructuring nested data** | JSON parsing, configuration validation |
| **Replacing long `if-elif` chains** | Command dispatch, status code handling |
| **Type-based dispatch** | AST processing, geometry calculations |
| **Validating structured input** | Form data, message payloads |

**When to Avoid Pattern Matching**

| Use Case | Better Alternative |
|---|---|
| **Simple value comparison** | `if value == X:` |
| **Range checks** | `if 0 <= x < 100:` |
| **Dynamic conditions** | Regular `if-elif` with computed expressions |
| **Polymorphic dispatch** | Class methods and inheritance |
| **Performance-critical inner loops** | Direct comparison may be faster |

**Common Pitfalls**

| Pitfall | Description | Solution |
|---|---|---|
| **Irrefutable pattern not last** | `case x:` followed by another case raises `SyntaxError` | Put capture patterns and wildcards last |
| **Bare name as literal** | `case CONSTANT:` captures, not compares | Use dotted name (`case module.CONSTANT:`) or guard |
| **Forgetting `__match_args__`** | Positional class patterns fail | Define `__match_args__` or use keyword patterns |
| **Over-nesting** | Deeply nested patterns are hard to read | Extract intermediate variables or functions |
| **Assuming exhaustion** | Without wildcard, unmatched subjects skip | Add `case _:` for exhaustive handling |

### Multiple Annotated Complete Code Examples and Their Expected Outputs

**Example 1: Refactoring `if-elif` to `match-case`**

```python
# BEFORE: if-elif chain
def process_response(response):
    if isinstance(response, dict):
        if response.get("status") == "ok":
            return response["data"]
        elif response.get("status") == "error":
            return f"Error: {response['message']}"
    elif isinstance(response, list):
        return f"List of {len(response)} items"
    return "Unknown"

# AFTER: match-case
def process_response(response):
    match response:
        case {"status": "ok", "data": data}:
            return data
        case {"status": "error", "message": msg}:
            return f"Error: {msg}"
        case [*items]:
            return f"List of {len(items)} items"
        case _:
            return "Unknown"

# Both produce the same results
print(process_response({"status": "ok", "data": [1, 2, 3]}))
print(process_response({"status": "error", "message": "Not found"}))
print(process_response([1, 2, 3]))
print(process_response(42))
```

**Expected Output:**
```
[1, 2, 3]
Error: Not found
List of 3 items
Unknown
```

**Breakdown:** The `match` version is more declarative: the structure of the expected data is immediately visible. The `if-elif` version requires reading through nested conditions.

**Example 2: Avoiding the Irrefutable Pattern Pitfall**

```python
# WRONG: This raises SyntaxError
# match x:
#     case y:        # irrefutable capture pattern
#         print(y)
#     case 1:        # SyntaxError: irrefutable pattern before another case
#         print("one")

# CORRECT: Put specific patterns first, capture last
match x:
    case 1:
        print("one")
    case y:            # capture pattern last
        print(f"Other: {y}")
```

**Expected Output:**
```
Other: 42
```
(Assuming `x = 42`)

**Breakdown:** An irrefutable pattern (capture or wildcard) must be the last case. Placing it earlier prevents subsequent cases from ever being reached, so Python raises `SyntaxError`.

**Example 3: Using Guards for Dynamic Conditions**

```python
# Match with guards for range-based classification
def classify_temperature(temp):
    match temp:
        case t if t < 0:
            return "Freezing"
        case t if t < 20:
            return "Cold"
        case t if t < 30:
            return "Warm"
        case _:
            return "Hot"

# Test
for t in [-5, 15, 25, 35]:
    print(f"{t}°C: {classify_temperature(t)}")
```

**Expected Output:**
```
-5°C: Freezing
15°C: Cold
25°C: Warm
35°C: Hot
```

**Breakdown:** Guards allow range-based classification while keeping the pattern-matching syntax. Each guard is evaluated in order, and the first true guard wins.

### Real-World Cases with Explanation

**Case 1: Refactoring a Parser**

```python
# BEFORE: nested if-elif with indexing
def parse_token(token):
    if token[0] == "NUMBER":
        return int(token[1])
    elif token[0] == "STRING":
        return str(token[1])
    elif token[0] == "BOOL":
        return token[1] == "true"

# AFTER: pattern matching
def parse_token(token):
    match token:
        case ("NUMBER", value):
            return int(value)
        case ("STRING", value):
            return str(value)
        case ("BOOL", value):
            return value == "true"
```

**Why it matters:** The pattern-matching version makes the expected token shapes explicit and eliminates manual indexing.

**Case 2: Designing for Extensibility**

```python
# Use a dispatch dictionary for extensibility
HANDLERS = {
    "click": handle_click,
    "keypress": handle_key,
    "scroll": handle_scroll,
}

def process(event):
    handler = HANDLERS.get(event["type"])
    if handler:
        handler(event)
    else:
        log_unknown(event)
```

**Why it matters:** While `match` is excellent for built-in patterns, a dispatch dictionary is easier to extend at runtime (e.g., plugins registering new handlers).

---

## References

- Python Software Foundation. *PEP 634 – Structural Pattern Matching: Specification*. https://peps.python.org/pep-0634/
- Python Software Foundation. *PEP 635 – Structural Pattern Matching: Motivation and Rationale*. https://peps.python.org/pep-0635/
- Python Software Foundation. *PEP 636 – Structural Pattern Matching: Tutorial*. https://peps.python.org/pep-0636/
- Python Software Foundation. *What's New In Python 3.10 — Structural Pattern Matching*. https://docs.python.org/3/whatsnew/3.10.html
- Real Python. *Structural Pattern Matching in Python*. https://realpython.com/structural-pattern-matching/
- Python Software Foundation. *The Python Tutorial — match Statements*. https://docs.python.org/3/tutorial/controlflow.html#match-statements
- Python Software Foundation. *ast — Abstract Syntax Trees — MatchAs, MatchOr, MatchClass*. https://docs.python.org/3/library/ast.html
- Python-Dev Mailing List. *Pattern matching reborn: PEP 622 is dead, long live PEP 634, 635, 636*. https://mail.python.org/archives/list/python-dev@python.org/message/Q4BU4XMNOVUW6Y2OGXZB6RAQRRQKWAOU/
- Kanaries Docs. *Python Match Case: Structural Pattern Matching Explained (Python 3.10+)*. https://docs.kanaries.net/topics/Python/python-match-case
- Buddy Works. *Structural Pattern Matching In Python*. https://buddy.works/tutorials/structural-pattern-matching-In-python