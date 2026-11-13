# Python Structural Pattern Matching: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**

Structural pattern matching is a control-flow mechanism introduced in Python 3.10 (via PEP 634) that allows a program to compare a subject value against a series of patterns, destructure its components, bind variables, and execute the first matching branch. It is a declarative alternative to long `if`/`elif` chains for branching on the shape, type, and value of data.

**Technical Definition**

The `match` statement evaluates a subject expression once and then attempts to match the resulting value against each `case` pattern in order. A pattern is a syntactic construct that either succeeds or fails against the subject; on success, it may bind names to subcomponents of the subject. Matching proceeds top-to-bottom, and only the first successful pattern’s block executes—there is no fall-through. Patterns include literal, capture, wildcard, value, group, OR, sequence, mapping, and class patterns. An optional `guard` (`if` clause) can further constrain a pattern. The `match` and `case` keywords are soft keywords, meaning they remain valid identifiers outside match statements. The feature is specified in PEP 634, motivated in PEP 635, and taught in PEP 636.

**Beginner-Friendly Explanation**

Think of structural pattern matching as a smart sorting machine. You drop an item onto a conveyor belt, and the machine checks it against a series of “shapes” or “templates.” If the item fits a template—say, a list with exactly two elements, or a dictionary with a specific key—the machine grabs the relevant parts and runs the matching task. Unlike a traditional `if`/`elif` chain, the machine can look inside the item, pull out pieces, and hand them to you as named variables, all in one step.

### Key Characteristics

- **Declarative branching**: Patterns describe the shape of data rather than the procedural steps to check it.
- **Destructuring**: Patterns bind subcomponents of the subject to names (e.g., `case [x, y]:` binds `x` and `y`).
- **No fall-through**: Only the first matching `case` block executes; there is no implicit continuation to subsequent cases.
- **Soft keywords**: `match` and `case` are not reserved words and remain usable as identifiers elsewhere.
- **Top-to-bottom evaluation**: Patterns are tried in order; specificity and ordering matter.
- **Guards**: An `if` guard can add runtime conditions that the pattern syntax cannot express.
- **Structural inspection**: The statement checks type and shape, not merely value equality.
- **Python 3.10+**: The feature requires Python 3.10 or newer.

### Prerequisites

- Python 3.10 or later installed.
- Familiarity with basic control flow (`if`/`elif`/`else`).
- Understanding of Python data structures: tuples, lists, dictionaries, and classes.
- Knowledge of variable binding and scope in Python.
- Awareness of Python’s soft-keyword mechanism.

### Related Programming Areas

- **Object-oriented programming**: Class patterns integrate with `__match_args__` and dataclasses.
- **Functional programming**: Pattern matching is a staple of ML-family languages and complements Python’s functional tools.
- **Parsing and AST processing**: Structural patterns excel at destructuring syntax trees and protocol messages.
- **State machines**: Match statements are well suited to dispatching on event types and payload shapes.
- **Data validation**: Mapping and sequence patterns can validate the shape of incoming JSON or configuration data.

---

## Core Concepts / Features

---

### 1. `match` and `case` Statements

#### Definitions

**Core Definition**

The `match` statement is a compound statement that takes a subject expression and compares its value against one or more `case` patterns, executing the block of the first pattern that matches.

**Technical Definition**

The syntax is `match subject_expr: case patterns [guard]: block ...`. The subject expression is evaluated once; if it contains commas, a tuple is constructed. Each `case` clause consists of one or more patterns (separated by commas for sequence-like matching) and an optional `if` guard. Matching proceeds top-to-bottom. A pattern may bind names; bindings from a failed pattern are discarded. If no case matches, the entire `match` statement does nothing (there is no implicit error). The `match` statement is a soft compound statement and does not introduce a new scope for bindings outside the case block—names bound in a successful case remain accessible after the `match` statement.

**Beginner-Friendly Explanation**

The `match` statement is the “switchboard” of structural pattern matching. You give it a value, and then list several `case` templates. Python tries each template in order. When one fits, Python runs that case’s code and skips the rest. If none fit, nothing happens—unless you add a wildcard `case _:` as a catch-all.

#### Purposes

- To replace long, repetitive `if`/`elif` chains that inspect type and shape.
- To destructure complex data (nested sequences, mappings, objects) in a single readable construct.
- To dispatch on the structure of protocol messages, AST nodes, or configuration dictionaries.
- To bind subcomponents of a subject to meaningful variable names without manual indexing.
- To combine structural checks with runtime conditions via guards.

#### Syntax Rules and Structure

**Complete General Syntax**

```
match subject_expression:
    case pattern_1 [if guard_1]:
        block_1
    case pattern_2 [if guard_2]:
        block_2
    case _:
        default_block
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `match` | Soft keyword initiating the statement. |
| `subject_expression` | The value to be matched; evaluated once. |
| `case` | Soft keyword introducing a pattern branch. |
| `pattern` | A structural pattern (literal, capture, sequence, mapping, class, etc.). |
| `guard` | Optional `if` expression evaluated only if the pattern matches. |
| `:` | Terminates the case header. |
| `block` | Indented suite executed when the pattern (and guard) succeed. |
| `_` | Wildcard pattern that matches anything; typically the final catch-all. |

**Syntax Rules**

- The subject expression is evaluated exactly once.
- Cases are tried top-to-bottom; the first success wins.
- If no case matches, execution continues after the `match` statement.
- The `match` and `case` keywords are soft keywords, not reserved words.
- Patterns may bind names; bound names are available in the case block and afterward.
- A guard is evaluated only when its pattern has already matched.
- A wildcard `case _:` is the idiomatic default branch.

**Constraints and Limitations**

- Requires Python 3.10+.
- There is no fall-through; each case is independent.
- A bare name in a pattern is a capture pattern, not a value comparison; dotted names or enums are needed for value matching.
- Bindings made in a case that does not execute are not created.
- Deeply nested patterns can become hard to read.

#### Annotated Code Examples

**Example 1: Basic `match`/`case` Dispatch**

```python
# Dispatch on HTTP status codes.
def describe_status(code):
    match code:
        case 200:
            return "OK"
        case 201:
            return "Created"
        case 404:
            return "Not Found"
        case 500:
            return "Internal Server Error"
        case _:
            return f"Unknown status: {code}"

print(describe_status(200))   # OK
print(describe_status(404))   # Not Found
print(describe_status(418))   # Unknown status: 418
```

**Expected Output:**

```
OK
Not Found
Unknown status: 418
```

**Step-by-Step Breakdown:**

1. The subject `code` is evaluated once.
2. Each literal pattern is compared against `code` in order.
3. For `200`, the literal pattern `200` succeeds, and the first block returns `"OK"`.
4. For `418`, no literal matches, so the wildcard `case _:` executes.

**Why the Output is Produced:**

Literal patterns match by equality (except `True`, `False`, and `None`, which match by identity). The wildcard captures everything, providing a default path.

---

**Example 2: `match` with Destructuring**

```python
# Destructure a command into action and object.
def interpret_command(command: str):
    match command.split():
        case [action, obj]:
            return f"Action: {action}, Object: {obj}"
        case [action]:
            return f"Action: {action} (no object)"
        case _:
            return "Unrecognized command format"

print(interpret_command("get sword"))
print(interpret_command("look"))
print(interpret_command("attack dragon with fire"))
```

**Expected Output:**

```
Action: get, Object: sword
Action: look (no object)
Unrecognized command format
```

**Step-by-Step Breakdown:**

1. `command.split()` produces a list of words.
2. The sequence pattern `[action, obj]` matches exactly two elements and binds them.
3. The pattern `[action]` matches exactly one element.
4. The wildcard handles lists of other lengths.

**Why the Output is Produced:**

Sequence patterns match the length and bind elements positionally, avoiding manual index checks.

#### Real-World Cases

- **CLI argument parsing**: Dispatching on the shape of `sys.argv` or parsed command objects.
- **API response handling**: Matching JSON dictionaries by their `"type"` key and destructuring payloads.
- **AST processing**: Matching nodes like `BinOp("+", a, BinOp("*", b, c))` to recognize expression trees.
- **Event systems**: Routing GUI or network events based on event class and attributes.

---

### 2. Literal Patterns

#### Definitions

**Core Definition**

A literal pattern matches a subject value against a literal constant such as a number, string, `True`, `False`, or `None`.

**Technical Definition**

Literal patterns compare the subject to the literal using equality (`==`), with special handling for `True`, `False`, and `None`, which are compared by identity (`is`). Literal patterns can appear alone or as subpatterns inside sequence, mapping, or class patterns. They are the simplest pattern form and are often used in combination with OR patterns for multiple-value matching. Literal patterns cannot bind names.

**Beginner-Friendly Explanation**

A literal pattern is like saying “does this value equal exactly this specific thing?” For example, `case 404:` checks whether the value is exactly the number 404. `case "quit":` checks for the exact string `"quit"`. It is the most straightforward kind of pattern, similar to a traditional `if value == literal:`.

#### Purposes

- To match against exact numeric, string, or boolean constants.
- To handle sentinel values such as `None` or `True` explicitly.
- To combine with OR patterns for multiple literal alternatives (e.g., `401 | 403`).
- To serve as the foundation for more complex patterns that embed literals.
- To provide clear, readable dispatch tables for status codes, command names, or enum-like values.

#### Syntax Rules and Structure

**Complete General Syntax**

```
case literal:
    block
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `literal` | A number, string, `True`, `False`, or `None`. |
| `:` | Ends the case header. |
| `block` | Executed if the subject equals the literal. |

**Syntax Rules**

- Literal patterns support integers, floats, complex numbers, strings, bytes, `True`, `False`, and `None`.
- `True`, `False`, and `None` are matched by identity, not equality.
- `1` does not match `True` even though `1 == True` in Python.
- Literals can be nested inside other patterns (e.g., `case [1, x]:`).
- Negative numbers are represented as unary minus applied to a literal; the grammar treats them as literal patterns.

**Constraints and Limitations**

- Literal patterns cannot bind variables.
- Only immutable literal constants are allowed; expressions and variable references are not literal patterns.
- A bare name is never a literal pattern; it is a capture pattern.
- Dotted names (e.g., `Color.RED`) are value patterns, not literal patterns.

#### Annotated Code Examples

**Example 1: Matching Status Codes**

```python
def status_message(code: int) -> str:
    match code:
        case 200:
            return "OK"
        case 201:
            return "Created"
        case 204:
            return "No Content"
        case 400:
            return "Bad Request"
        case 401 | 403:
            return "Unauthorized or Forbidden"
        case _:
            return "Other"

print(status_message(200))
print(status_message(403))
print(status_message(302))
```

**Expected Output:**

```
OK
Unauthorized or Forbidden
Other
```

**Step-by-Step Breakdown:**

1. `200` matches the first literal.
2. `403` matches the OR pattern `401 | 403`.
3. `302` matches none of the literals, so the wildcard handles it.

**Why the Output is Produced:**

Literal patterns check exact equality. OR patterns allow multiple literals in one case.

---

**Example 2: Identity Matching for `None` and Booleans**

```python
def classify(value):
    match value:
        case None:
            return "None"
        case True:
            return "True"
        case False:
            return "False"
        case 0:
            return "Zero"
        case _:
            return "Other"

print(classify(None))
print(classify(True))
print(classify(0))
print(classify(1))
```

**Expected Output:**

```
None
True
Zero
Other
```

**Step-by-Step Breakdown:**

1. `None` matches the `None` literal by identity.
2. `True` matches the `True` literal; note that `1` would not match `True`.
3. `0` matches the `0` literal.
4. `1` matches none of the literals and falls to the wildcard.

**Why the Output is Produced:**

`None`, `True`, and `False` are identity-matched, while numeric literals are equality-matched.

#### Real-World Cases

- **HTTP status handling**: Mapping status codes to messages or retry logic.
- **Command routers**: Matching exact command strings like `"start"`, `"stop"`, `"restart"`.
- **Configuration flags**: Matching boolean or `None` defaults in configuration dictionaries.
- **Enum-like constants**: Combined with value patterns for symbolic constants.

---

### 3. Capture Patterns

#### Definitions

**Core Definition**

A capture pattern is a bare name (identifier) that matches any subject value and binds that value to the name.

**Technical Definition**

A capture pattern consists of a single identifier that is not a keyword. It always succeeds and binds the subject (or subcomponent) to the name. Capture patterns are irrefutable: they match everything. Because a bare name captures, it is impossible to match against a variable’s value using a bare name; dotted names or literal patterns must be used instead. Capture patterns are essential for destructuring, as they bind the pieces extracted by sequence, mapping, and class patterns.

**Beginner-Friendly Explanation**

A capture pattern is like a sticky note that says “put whatever is here into this box.” If you write `case x:`, Python puts the entire subject into a variable named `x`. If you write `case [first, second]:`, the names `first` and `second` capture the two elements of a list. Capture patterns never fail—they grab whatever is presented.

#### Purposes

- To bind the entire subject to a name for further processing.
- To destructure sequences, mappings, and objects by capturing their components.
- To provide a fallback catch-all that binds the value (though `case _:` is often preferable when the value is not needed).
- To name subpatterns inside complex patterns for use in guards or case blocks.
- To extract values from nested structures without manual indexing.

#### Syntax Rules and Structure

**Complete General Syntax**

```
case name:
    block
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `name` | A valid Python identifier that is not a keyword. |
| `:` | Ends the case header. |
| `block` | Executed with `name` bound to the subject. |

**Syntax Rules**

- A capture pattern is a single bare name.
- It always matches; it is irrefutable.
- The name is bound in the local scope of the case block and remains accessible afterward.
- A bare name cannot be used to compare against a variable’s value; use a dotted name or literal instead.
- Capture patterns can appear as subpatterns inside sequence, mapping, and class patterns.
- The wildcard `_` is a special pattern that does not bind; a regular name captures.

**Constraints and Limitations**

- A capture pattern always succeeds, so a bare capture cannot be followed by meaningful cases (subsequent cases are unreachable).
- Using a capture where a constant comparison was intended is a common bug.
- Captured names are not deleted after the `match` statement; they remain in scope.
- The name `_` is reserved as the wildcard and should not be used as a capture name.

#### Annotated Code Examples

**Example 1: Capturing the Entire Subject**

```python
def describe(value):
    match value:
        case 0:
            return "Zero"
        case x:
            return f"Non-zero value: {x}"

print(describe(0))
print(describe(42))
print(describe("hello"))
```

**Expected Output:**

```
Zero
Non-zero value: 42
Non-zero value: hello
```

**Step-by-Step Breakdown:**

1. `0` matches the literal `0`.
2. Any other value matches the capture pattern `x`, which binds the entire subject.
3. The capture pattern is irrefutable, so it acts as a catch-all that also names the value.

**Why the Output is Produced:**

Capture patterns match everything and bind the value. The literal pattern must come before the capture to handle the zero case.

---

**Example 2: Capturing Subcomponents**

```python
def process_point(point):
    match point:
        case (x, y):
            return f"Point at ({x}, {y})"
        case _:
            return "Not a point"

print(process_point((3, 4)))
print(process_point([10, 20]))
print(process_point("not a point"))
```

**Expected Output:**

```
Point at (3, 4)
Point at (10, 20)
Not a point
```

**Step-by-Step Breakdown:**

1. The sequence pattern `(x, y)` matches any sequence of length 2 and captures its elements.
2. Tuples and lists of length 2 both match.
3. A string of length greater than 2 does not match the sequence pattern, so the wildcard handles it.

**Why the Output is Produced:**

Capture patterns inside sequence patterns bind subcomponents, enabling destructuring without manual indexing.

#### Real-World Cases

- **Data extraction**: Pulling fields out of JSON-like dictionaries or tuples.
- **Parser combinators**: Binding matched tokens for later semantic analysis.
- **Configuration processing**: Capturing values from nested configuration structures.
- **Error handling**: Binding an exception or error object for logging.

---

### 4. Wildcard Patterns

#### Definitions

**Core Definition**

The wildcard pattern, written `_`, matches any subject value without binding it to a name. It is typically used as the final catch-all case.

**Technical Definition**

The wildcard pattern `_` is an irrefutable pattern that always succeeds and performs no binding. It is represented by the single underscore identifier and is distinct from a capture pattern because it does not create a variable. The wildcard is commonly used as the default `case _:` branch to handle values that do not match any preceding pattern. It can also appear as a subpattern inside sequence or mapping patterns to ignore certain components (e.g., `case [first, _, third]:`). Because `_` does not bind, it cannot be referenced in the case block or guard.

**Beginner-Friendly Explanation**

The wildcard `_` is the “anything goes” pattern. It matches whatever is left over and doesn’t bother naming it. Think of it as a garbage bin: anything that didn’t fit the more specific patterns falls into the wildcard case. If you need to ignore part of a structure—say, the middle element of a three-item list—you can use `_` as a placeholder without creating an unnecessary variable.

#### Purposes

- To provide a default branch that catches all unmatched values.
- To ignore specific components of a subject during destructuring.
- To avoid creating unused variable bindings.
- To make the intent explicit that a value is deliberately discarded.
- To serve as the final case in a match statement when exhaustive handling is desired.

#### Syntax Rules and Structure

**Complete General Syntax**

```
case _:
    block
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `_` | Wildcard pattern; matches anything, binds nothing. |
| `:` | Ends the case header. |
| `block` | Executed when no earlier pattern matched. |

**Syntax Rules**

- `_` is a single underscore and is treated as a wildcard in patterns.
- The wildcard always succeeds and does not bind a name.
- It is conventionally placed as the last case.
- It can appear as a subpattern (e.g., `case (_, y):` to ignore the first element).
- Using `_` as a variable name in other contexts is a valid identifier, but in patterns it is the wildcard.

**Constraints and Limitations**

- A wildcard cannot be referenced in the case block because it binds nothing.
- A wildcard case makes subsequent cases unreachable.
- The wildcard is not a capture pattern; if the value is needed, use a named capture instead.
- Overuse of wildcards can hide unexpected input; explicit handling is often preferable.

#### Annotated Code Examples

**Example 1: Default Catch-All**

```python
def handle_command(command: str):
    match command:
        case "start":
            return "Starting..."
        case "stop":
            return "Stopping..."
        case _:
            return f"Unknown command: {command}"

print(handle_command("start"))
print(handle_command("stop"))
print(handle_command("pause"))
```

**Expected Output:**

```
Starting...
Stopping...
Unknown command: pause
```

**Step-by-Step Breakdown:**

1. `"start"` and `"stop"` match their literal patterns.
2. `"pause"` matches none, so the wildcard executes and uses `command` (the original subject) in the message.

**Why the Output is Produced:**

The wildcard provides a default path. Since it binds nothing, the original subject variable is used to construct the message.

---

**Example 2: Ignoring Components**

```python
def extract_edges(sequence):
    match sequence:
        case (first, _, last):
            return f"First: {first}, Last: {last}"
        case _:
            return "Sequence too short"

print(extract_edges([1, 2, 3]))
print(extract_edges([1, 2]))
```

**Expected Output:**

```
First: 1, Last: 3
Sequence too short
```

**Step-by-Step Breakdown:**

1. `[1, 2, 3]` matches the pattern `(first, _, last)`; the middle element is ignored.
2. `[1, 2]` has length 2 and does not match the three-element pattern, so the wildcard handles it.

**Why the Output is Produced:**

The wildcard subpattern `_` ignores the middle element without binding it, making the intent explicit.

#### Real-World Cases

- **Default handlers**: Catching unexpected commands, events, or message types.
- **Tuple destructuring**: Ignoring irrelevant fields when unpacking records.
- **Protocol parsing**: Skipping header or padding fields that are not needed.
- **Configuration validation**: Providing a fallback for unrecognized configuration shapes.

---

### 5. Value Patterns

#### Definitions

**Core Definition**

A value pattern matches a subject against a named constant using dotted notation, such as `Color.RED`, `Status.ACTIVE`, or a module-level constant.

**Technical Definition**

A value pattern is a dotted name (an attribute reference) that is evaluated to a constant and compared to the subject using equality (`==`). Value patterns allow matching against symbolic constants such as enum members, module-level constants, or class attributes without hard-coding their literal values. The dotted name must be a valid attribute access; a bare name is a capture pattern, not a value pattern. Value patterns are irrefutable only if the constant’s equality is irreflexive (which is rare); normally they are refutable.

**Beginner-Friendly Explanation**

A value pattern lets you compare against a named constant instead of a literal. For example, if you have `Color.RED` defined as an enum member, you can write `case Color.RED:` and Python will compare the subject to that constant. This is useful because it keeps your code readable and avoids magic numbers or strings scattered throughout.

#### Purposes

- To match against symbolic constants such as enum members.
- To avoid hard-coding literal values in patterns.
- To keep pattern matching synchronized with constant definitions.
- To leverage the readability and safety of enums in dispatch logic.
- To match against module-level or class-level constants without repeating their values.

#### Syntax Rules and Structure

**Complete General Syntax**

```
case Dotted.Name:
    block
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `Dotted.Name` | An attribute reference that evaluates to a constant. |
| `:` | Ends the case header. |
| `block` | Executed if the subject equals the constant. |

**Syntax Rules**

- The pattern must be a dotted name (e.g., `Color.RED`, `module.CONSTANT`).
- A bare name is always a capture pattern, never a value pattern.
- The dotted name is evaluated at match time (or looked up) and compared using equality.
- Enum members are the most common use case.
- Value patterns can be nested inside other patterns.

**Constraints and Limitations**

- Bare names cannot be used for value matching; they capture instead.
- The dotted name must be resolvable in the current scope.
- Value patterns rely on `==`; custom equality behavior can affect matching.
- There is no built-in “constant pattern” syntax for bare names.

#### Annotated Code Examples

**Example 1: Matching Enum Members**

```python
from enum import Enum, auto

class Color(Enum):
    RED = auto()
    GREEN = auto()
    BLUE = auto()

def describe_color(color):
    match color:
        case Color.RED:
            return "The color is red."
        case Color.GREEN:
            return "The color is green."
        case Color.BLUE:
            return "The color is blue."
        case _:
            return "Unknown color."

print(describe_color(Color.RED))
print(describe_color(Color.GREEN))
print(describe_color("red"))
```

**Expected Output:**

```
The color is red.
The color is green.
Unknown color.
```

**Step-by-Step Breakdown:**

1. `Color.RED` is a dotted value pattern that compares the subject to the enum member.
2. Enum members match by identity/equality.
3. A plain string `"red"` does not match any enum member and falls to the wildcard.

**Why the Output is Produced:**

Value patterns with enum members provide type-safe, readable dispatch that avoids magic literals.

#### Real-World Cases

- **Enum-based state machines**: Matching on state enum members.
- **Configuration constants**: Matching against module-level constants like `DEFAULT_TIMEOUT`.
- **Protocol opcodes**: Matching named opcode constants in network protocols.
- **Error codes**: Matching named error constants from a library.

---

### 6. Group Patterns

#### Definitions

**Core Definition**

A group pattern is a parenthesized pattern used to group subpatterns, control precedence in OR patterns, or apply an `as` binding to a composite pattern.

**Technical Definition**

Group patterns use parentheses to enclose a pattern, which can be useful for readability and for controlling the scope of `as` bindings and OR alternatives. The syntax `(pattern)` groups the pattern without changing its matching behavior. Group patterns are often combined with `as` to bind the entire matched value to a name while also destructuring it, e.g., `case (Point(x, y) as p):`. They can also group OR alternatives: `case (1 | 2 | 3):`.

**Beginner-Friendly Explanation**

Group patterns are like parentheses in arithmetic: they tell Python how to group things. If you want to match a pattern and also give the whole matched value a name, you can write `case (pattern as name):`. If you want an OR pattern to apply to a whole sequence, parentheses help make that clear. They don’t change what matches—they just organize the pattern for clarity and binding.

#### Purposes

- To bind the entire matched value to a name while also destructuring it using `as`.
- To group OR alternatives for clarity or precedence.
- To improve readability of complex nested patterns.
- To apply a guard to a grouped pattern as a unit.
- To nest patterns explicitly when the grammar might otherwise be ambiguous.

#### Syntax Rules and Structure

**Complete General Syntax**

```
case (pattern):
    block

case (pattern as name):
    block
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `(` `)` | Parentheses grouping the pattern. |
| `pattern` | Any valid pattern. |
| `as name` | Optional binding of the entire matched value to `name`. |
| `:` | Ends the case header. |

**Syntax Rules**

- Parentheses group patterns without altering matching semantics.
- The `as` clause binds the entire subject (or subpattern) to a name.
- Group patterns can be nested inside other patterns.
- `as` bindings are available in the case block and guard.
- Parentheses are optional in many cases but improve clarity.

**Constraints and Limitations**

- Grouping does not create a new scope.
- The `as` name must be a valid identifier and cannot be `_` if a binding is needed.
- Over-parenthesizing can reduce readability.
- The grammar allows parentheses in sequence patterns; the grouping interpretation depends on context.

#### Annotated Code Examples

**Example 1: `as` Binding with Group Pattern**

```python
def process_point(point):
    match point:
        case (Point(x, y) as p):
            return f"Point {p} has coordinates ({x}, {y})"
        case _:
            return "Not a point"

class Point:
    __match_args__ = ("x", "y")
    def __init__(self, x, y):
        self.x = x
        self.y = y
    def __repr__(self):
        return f"Point({self.x}, {self.y})"

print(process_point(Point(3, 4)))
print(process_point("hello"))
```

**Expected Output:**

```
Point Point(3, 4) has coordinates (3, 4)
Not a point
```

**Step-by-Step Breakdown:**

1. `Point(x, y)` is a class pattern that matches `Point` instances and binds `x` and `y`.
2. `as p` binds the entire matched `Point` object to `p`.
3. Both the components and the whole object are available in the case block.

**Why the Output is Produced:**

Group patterns with `as` allow simultaneous destructuring and whole-value binding, which is useful when both the parts and the whole are needed.

#### Real-World Cases

- **AST processing**: Binding an entire node while also extracting its children.
- **Data validation**: Capturing a whole record while validating its fields.
- **Event handling**: Naming the full event object while matching its attributes.
- **Parser design**: Binding matched subtrees while destructuring their components.

---

### 7. OR Patterns

#### Definitions

**Core Definition**

An OR pattern matches if any of its alternative subpatterns matches, using the pipe operator `|` to separate alternatives.

**Technical Definition**

An OR pattern consists of two or more alternative patterns separated by `|`. The alternatives are tried left-to-right; the first successful alternative determines the match, and only bindings from that successful alternative are made available. If all alternatives fail, the OR pattern fails. OR patterns are useful for matching multiple literals, multiple class types, or multiple structural shapes in a single case. The `as` binding can be applied to an OR pattern only if all alternatives bind the same set of names.

**Beginner-Friendly Explanation**

An OR pattern is like saying “match this OR that OR that.” For example, `case 401 | 403:` matches either 401 or 403. It’s a compact way to group multiple alternatives that should be handled the same way. Python tries each alternative in order and stops at the first one that fits.

#### Purposes

- To handle multiple literal values in a single case (e.g., `401 | 403`).
- To match against multiple types or class patterns (e.g., `int() | float()`).
- To combine multiple structural shapes that share handling logic.
- To reduce duplication in dispatch tables.
- To group related alternatives for clarity.

#### Syntax Rules and Structure

**Complete General Syntax**

```
case pattern1 | pattern2 | pattern3:
    block
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `pattern1`, `pattern2`, … | Alternative patterns. |
| `|` | OR operator separating alternatives. |
| `:` | Ends the case header. |
| `block` | Executed if any alternative matches. |

**Syntax Rules**

- Alternatives are tried left-to-right; the first success wins.
- Bindings from unsuccessful alternatives are discarded.
- All alternatives in an OR pattern must bind the same set of names if `as` is used.
- OR patterns can be nested inside other patterns.
- OR patterns can combine literals, value patterns, class patterns, and structural patterns.

**Constraints and Limitations**

- Alternatives must be compatible in the names they bind if bindings are used.
- An OR pattern with a capture pattern in one alternative and a literal in another may bind names inconsistently, which is disallowed.
- The pipe `|` is also the bitwise OR operator outside patterns; context disambiguates.
- Deeply nested OR patterns can reduce readability.

#### Annotated Code Examples

**Example 1: Multiple Literal Alternatives**

```python
def classify_status(code: int) -> str:
    match code:
        case 200 | 201 | 204:
            return "Success"
        case 301 | 302:
            return "Redirect"
        case 400 | 401 | 403 | 404:
            return "Client Error"
        case 500 | 502 | 503:
            return "Server Error"
        case _:
            return "Unknown"

print(classify_status(201))
print(classify_status(302))
print(classify_status(404))
print(classify_status(500))
```

**Expected Output:**

```
Success
Redirect
Client Error
Server Error
```

**Step-by-Step Breakdown:**

1. `201` matches the first OR alternative.
2. `302` matches the redirect group.
3. `404` matches the client error group.
4. `500` matches the server error group.

**Why the Output is Produced:**

OR patterns group literals that share handling logic, reducing the number of case blocks.

---

**Example 2: OR with Class Patterns**

```python
class Circle:
    __match_args__ = ("radius",)
    def __init__(self, radius):
        self.radius = radius

class Square:
    __match_args__ = ("side",)
    def __init__(self, side):
        self.side = side

def describe_shape(shape):
    match shape:
        case Circle(radius) | Square(side):
            return f"Shape with dimension {radius if isinstance(shape, Circle) else side}"
        case _:
            return "Unknown shape"

print(describe_shape(Circle(5)))
print(describe_shape(Square(4)))
```

**Expected Output:**

```
Shape with dimension 5
Shape with dimension 4
```

**Step-by-Step Breakdown:**

1. The OR pattern `Circle(radius) | Square(side)` matches either class and binds the corresponding attribute.
2. The bound name is used in the case block.

**Why the Output is Produced:**

OR patterns with class patterns allow unified handling of different types that share a conceptual role.

#### Real-World Cases

- **HTTP status grouping**: Handling 4xx and 5xx errors together.
- **Type dispatch**: Treating `int` and `float` as numeric alternatives.
- **Protocol messages**: Matching multiple message types that share a handler.
- **Shape processing**: Handling different geometric shapes uniformly.

---

### 8. Sequence Patterns

#### Definitions

**Core Definition**

A sequence pattern matches subjects that are sequences (lists, tuples, strings, etc.) with a specific structure, such as a fixed number of elements, optional elements, or starred captures.

**Technical Definition**

Sequence patterns use bracket or parenthesis syntax: `[p1, p2, ...]`, `(p1, p2, ...)`, or bare comma-separated patterns. They match if the subject is a sequence (implementing `__getitem__` and `__len__`) of the correct length and if each subpattern matches the corresponding element. A starred name `*rest` captures a variable-length tail (or middle) as a list. The pattern does not match if the subject is a non-sequence (e.g., a set or dictionary) or if the length constraints are not satisfied.

**Beginner-Friendly Explanation**

A sequence pattern is like a template for a list or tuple. `case [x, y]:` means “a sequence of exactly two items, which I’ll call x and y.” You can use `*rest` to capture any number of extra items: `case [first, *rest]:` matches any non-empty sequence and puts the tail in `rest`. It’s a powerful way to unpack data without writing index-based code.

#### Purposes

- To destructure lists, tuples, and other sequences by position.
- To match sequences of a specific length or with a variable-length tail.
- To validate the shape of incoming data (e.g., command tokens, coordinate pairs).
- To capture sub-sequences for further processing.
- To avoid manual length checks and indexing.

#### Syntax Rules and Structure

**Complete General Syntax**

```
case [p1, p2, ...]:
    block

case [p1, *rest]:
    block

case (p1, p2):
    block
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `[` `]` / `(` `)` | Brackets or parentheses denoting a sequence pattern. |
| `p1`, `p2`, … | Subpatterns matched against elements by position. |
| `*rest` | Starred capture for variable-length remaining elements (bound as a list). |
| `:` | Ends the case header. |

**Syntax Rules**

- Sequence patterns match lists, tuples, and other sequences (excluding strings? Strings are sequences of characters and can match sequence patterns, but this is often not the intended use).
- The number of non-starred subpatterns must match the sequence length, unless a starred pattern is present.
- A starred pattern captures the remaining elements as a list.
- At most one starred name is allowed per sequence pattern.
- Parentheses, brackets, and bare comma-separated patterns are equivalent in many contexts.
- Sequence patterns do not match sets, dictionaries, or other non-sequence iterables.

**Constraints and Limitations**

- Sequence patterns require a sequence-like subject; generators and sets do not match.
- Strings are sequences and can be matched by sequence patterns, but this can be surprising.
- Only one starred capture is allowed.
- Deeply nested sequence patterns can be difficult to read.
- The length check is strict; optional elements require separate patterns or wildcards.

#### Annotated Code Examples

**Example 1: Fixed-Length Sequence**

```python
def parse_coordinates(data):
    match data:
        case [x, y]:
            return f"2D point: ({x}, {y})"
        case [x, y, z]:
            return f"3D point: ({x}, {y}, {z})"
        case _:
            return "Invalid coordinates"

print(parse_coordinates([1, 2]))
print(parse_coordinates((3, 4, 5)))
print(parse_coordinates([1]))
```

**Expected Output:**

```
2D point: (1, 2)
3D point: (3, 4, 5)
Invalid coordinates
```

**Step-by-Step Breakdown:**

1. `[1, 2]` matches the two-element pattern.
2. `(3, 4, 5)` matches the three-element pattern (tuples are sequences).
3. `[1]` matches neither and falls to the wildcard.

**Why the Output is Produced:**

Sequence patterns check length and bind elements positionally.

---

**Example 2: Starred Capture**

```python
def analyze_sequence(seq):
    match seq:
        case [first, *rest]:
            return f"First: {first}, Rest: {rest}"
        case []:
            return "Empty sequence"

print(analyze_sequence([1, 2, 3, 4]))
print(analyze_sequence([]))
```

**Expected Output:**

```
First: 1, Rest: [2, 3, 4]
Empty sequence
```

**Step-by-Step Breakdown:**

1. `[first, *rest]` matches any non-empty sequence; `first` gets the first element, `rest` gets the remaining list.
2. `[]` matches the empty sequence pattern.

**Why the Output is Produced:**

The starred pattern captures a variable number of elements as a list, enabling flexible destructuring.

#### Real-World Cases

- **Command parsing**: Splitting input into action and arguments.
- **Coordinate processing**: Handling 2D and 3D points uniformly.
- **Record destructuring**: Extracting fields from tuples returned by APIs.
- **Queue processing**: Capturing the head and tail of a work queue.

---

### 9. Mapping Patterns

#### Definitions

**Core Definition**

A mapping pattern matches dictionary-like subjects by checking for the presence of specific keys and matching their values against subpatterns.

**Technical Definition**

Mapping patterns use the syntax `{"key": pattern, ...}`. They match if the subject is a mapping (implementing `collections.abc.Mapping`) and contains all specified keys, with each value matching the corresponding subpattern. Additional keys in the subject are ignored. The special pattern `**rest` can capture the remaining key-value pairs as a dictionary. Mapping patterns do not match sequences or other non-mapping objects. They are particularly useful for processing JSON-like data structures.

**Beginner-Friendly Explanation**

A mapping pattern is like a checklist for a dictionary. `case {"name": name, "age": age}:` means “this dictionary must have a `"name"` key and an `"age"` key, and I’ll bind their values to variables.” Extra keys are fine—they’re simply ignored. It’s a convenient way to validate the shape of JSON-like data without writing a series of `in` and indexing checks.

#### Purposes

- To validate that a dictionary contains required keys.
- To destructure dictionary values by key into named variables.
- To match JSON-like data by its shape and key structure.
- To capture additional unknown keys into a rest dictionary.
- To replace verbose key-checking and indexing code.

#### Syntax Rules and Structure

**Complete General Syntax**

```
case {"key1": pattern1, "key2": pattern2}:
    block

case {"key": pattern, **rest}:
    block
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `{` `}` | Braces denoting a mapping pattern. |
| `"key": pattern` | A required key and its value subpattern. |
| `**rest` | Captures remaining key-value pairs as a dictionary. |
| `:` | Ends the case header. |

**Syntax Rules**

- The subject must be a mapping (e.g., `dict`).
- All specified keys must be present.
- Values must match their corresponding subpatterns.
- Extra keys are ignored unless `**rest` is used to capture them.
- `**rest` can appear at most once and captures a new dictionary.
- Mapping patterns do not match sequences or other non-mapping types.

**Constraints and Limitations**

- The subject must implement the mapping protocol; lists and tuples do not match.
- Keys in the pattern must be literals or value patterns; they cannot be capture patterns.
- `**rest` is optional and captures remaining items.
- Mapping patterns are not ordered; they check key presence and value matching.
- Deeply nested mapping patterns can become hard to read.

#### Annotated Code Examples

**Example 1: Validating Required Keys**

```python
def handle_user(user):
    match user:
        case {"name": name, "age": age}:
            return f"User {name}, age {age}"
        case {"name": name}:
            return f"User {name}, age unknown"
        case _:
            return "Invalid user record"

print(handle_user({"name": "Alice", "age": 30}))
print(handle_user({"name": "Bob"}))
print(handle_user({"id": 123}))
```

**Expected Output:**

```
User Alice, age 30
User Bob, age unknown
Invalid user record
```

**Step-by-Step Breakdown:**

1. The first record has both required keys and matches the first pattern.
2. The second has only `"name"`, matching the second pattern.
3. The third lacks `"name"` and falls to the wildcard.

**Why the Output is Produced:**

Mapping patterns check for key presence and match values, allowing progressive validation of dictionary shapes.

---

**Example 2: Capturing Rest Keys**

```python
def summarize_config(config):
    match config:
        case {"host": host, "port": port, **rest}:
            return f"Host: {host}, Port: {port}, Extra: {rest}"
        case _:
            return "Missing required config"

print(summarize_config({"host": "localhost", "port": 8080, "debug": True}))
print(summarize_config({"host": "localhost"}))
```

**Expected Output:**

```
Host: localhost, Port: 8080, Extra: {'debug': True}
Missing required config
```

**Step-by-Step Breakdown:**

1. The first config has both required keys; `**rest` captures `{"debug": True}`.
2. The second config lacks `"port"` and does not match, so the wildcard executes.

**Why the Output is Produced:**

`**rest` captures additional keys as a dictionary, enabling flexible handling of optional or unknown configuration entries.

#### Real-World Cases

- **JSON API parsing**: Matching response dictionaries by their `"type"` or `"status"` fields.
- **Configuration validation**: Ensuring required settings are present before use.
- **Message routing**: Dispatching on message dictionaries with `"cmd"` keys.
- **Data cleaning**: Destructuring records while preserving unknown fields.

---

### 10. Class Patterns

#### Definitions

**Core Definition**

A class pattern matches subjects that are instances of a specified class and optionally destructures their attributes using positional or keyword subpatterns.

**Technical Definition**

Class patterns have the syntax `ClassName(pattern1, pattern2, ...)` or `ClassName(attr1=pattern1, attr2=pattern2, ...)`. They match if the subject is an instance of `ClassName` (or a subclass) and if the specified attributes match the subpatterns. Positional patterns rely on the class defining `__match_args__`, a tuple of attribute names that determines which attributes are matched positionally. Keyword patterns match named attributes directly. Class patterns are the primary mechanism for integrating pattern matching with object-oriented data models and dataclasses.

**Beginner-Friendly Explanation**

A class pattern is like a mold for objects. `case Point(x, y):` means “this object must be a `Point`, and I’ll grab its `x` and `y` attributes.” Python uses the class’s `__match_args__` to know which attributes correspond to positional patterns. You can also use keyword patterns: `case Point(x=0, y=y):` matches only points with `x` equal to 0. It’s a clean way to inspect object types and extract their data simultaneously.

#### Purposes

- To match objects by type and destructure their attributes in one step.
- To dispatch on class hierarchies without manual `isinstance` checks.
- To validate object state by matching attribute values against subpatterns.
- To integrate pattern matching with dataclasses and `__match_args__`.
- To replace visitor-pattern boilerplate for simple dispatch scenarios.

#### Syntax Rules and Structure

**Complete General Syntax**

```
case ClassName(pattern1, pattern2, ...):
    block

case ClassName(attr1=pattern1, attr2=pattern2, ...):
    block
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `ClassName` | The class to match against. |
| `(pattern1, ...)` | Positional subpatterns matched via `__match_args__`. |
| `(attr=pattern, ...)` | Keyword subpatterns matched against named attributes. |
| `:` | Ends the case header. |

**Syntax Rules**

- The subject must be an instance of `ClassName` (or a subclass).
- Positional patterns require `ClassName` to define `__match_args__` as a tuple of attribute names.
- Keyword patterns match attributes by name without needing `__match_args__`.
- Positional and keyword patterns can be mixed, but positional patterns must precede keyword patterns.
- Subpatterns can be any valid pattern (literal, capture, nested class, etc.).

**Constraints and Limitations**

- Classes without `__match_args__` cannot be used with positional patterns.
- `__match_args__` must be a tuple of strings; invalid definitions raise errors.
- Built-in classes like `int` and `str` can be matched by class patterns but do not support positional destructuring without `__match_args__`.
- Deeply nested class patterns can become complex.
- Class patterns only match instances, not the class objects themselves.

#### Annotated Code Examples

**Example 1: Positional Class Pattern with `__match_args__`**

```python
from dataclasses import dataclass

@dataclass
class Point:
    x: int
    y: int

def describe(point):
    match point:
        case Point(0, 0):
            return "Origin"
        case Point(x, 0):
            return f"On x-axis at {x}"
        case Point(0, y):
            return f"On y-axis at {y}"
        case Point(x, y):
            return f"Point at ({x}, {y})"
        case _:
            return "Not a point"

print(describe(Point(0, 0)))
print(describe(Point(5, 0)))
print(describe(Point(3, 4)))
```

**Expected Output:**

```
Origin
On x-axis at 5
Point at (3, 4)
```

**Step-by-Step Breakdown:**

1. `@dataclass` automatically generates `__match_args__ = ("x", "y")`.
2. `Point(0, 0)` matches the literal subpatterns for origin.
3. `Point(5, 0)` matches the second pattern, binding `x = 5`.
4. `Point(3, 4)` matches the general pattern.

**Why the Output is Produced:**

Dataclasses provide `__match_args__` by default, enabling positional destructuring in class patterns.

---

**Example 2: Keyword Class Pattern**

```python
class Config:
    def __init__(self, host, port, debug=False):
        self.host = host
        self.port = port
        self.debug = debug

def check_config(config):
    match config:
        case Config(debug=True):
            return "Debug mode enabled"
        case Config(host="localhost", port=8080):
            return "Local development config"
        case Config():
            return "Other config"
        case _:
            return "Not a config"

print(check_config(Config("localhost", 8080, debug=True)))
print(check_config(Config("localhost", 8080)))
print(check_config(Config("prod.example.com", 443)))
```

**Expected Output:**

```
Debug mode enabled
Local development config
Other config
```

**Step-by-Step Breakdown:**

1. The first config has `debug=True`, matching the first keyword pattern.
2. The second matches the local development keyword pattern.
3. The third matches the bare `Config()` pattern.

**Why the Output is Produced:**

Keyword patterns match named attributes regardless of `__match_args__`, providing flexible state-based dispatch.

#### Real-World Cases

- **Event handling**: Matching event objects by class and attributes.
- **AST traversal**: Matching node classes and destructuring their fields.
- **API response dispatch**: Matching response objects by type and status attributes.
- **Game entities**: Matching player, enemy, and item objects with their properties.

---

### 11. Guard Clauses

#### Definitions

**Core Definition**

A guard clause is an optional `if` condition attached to a `case` pattern that must evaluate to true for the case block to execute.

**Technical Definition**

A guard is introduced by `if` after the pattern(s) in a `case` clause: `case pattern if condition:`. The guard is evaluated only if the pattern has already matched. If the guard is falsy, the match fails and matching continues with the next case. Guards can reference names bound by the pattern, as well as any variables in scope. They are useful for adding runtime conditions that cannot be expressed structurally, such as numeric thresholds, membership tests, or external state checks.

**Beginner-Friendly Explanation**

A guard is an extra condition layered on top of a pattern. First, the pattern must fit; then the guard must be true. For example, `case [x, y] if x > 0:` means “match a two-element sequence, but only if the first element is positive.” If the pattern matches but the guard is false, Python moves on to the next case. Guards let you add logic that the pattern syntax alone cannot express.

#### Purposes

- To add numeric or relational constraints to structural patterns.
- To check membership, thresholds, or external state after a pattern matches.
- To disambiguate between cases that share the same structural pattern.
- To avoid deeply nested `if` statements inside case blocks.
- To combine structural matching with runtime validation.

#### Syntax Rules and Structure

**Complete General Syntax**

```
case pattern if guard_expression:
    block
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `pattern` | Any valid pattern. |
| `if` | Keyword introducing the guard. |
| `guard_expression` | A boolean expression evaluated only if the pattern matches. |
| `:` | Ends the case header. |

**Syntax Rules**

- The guard is evaluated after the pattern succeeds.
- If the guard is false, the match fails and the next case is tried.
- The guard can reference names bound by the pattern.
- The guard can reference any in-scope variables.
- A guard does not affect the pattern’s bindings if the guard fails; those bindings are discarded.

**Constraints and Limitations**

- Guards are expressions, not statements; they cannot contain assignments or multi-line logic.
- Guards are evaluated only when the pattern matches, so they do not add overhead for non-matching patterns.
- Overusing guards can make patterns harder to read; prefer structural patterns when possible.
- Guards cannot introduce new bindings that affect subsequent patterns.

#### Annotated Code Examples

**Example 1: Numeric Guard**

```python
def classify_number(value):
    match value:
        case 0:
            return "Zero"
        case n if n > 0:
            return f"Positive: {n}"
        case n if n < 0:
            return f"Negative: {n}"

print(classify_number(0))
print(classify_number(5))
print(classify_number(-3))
```

**Expected Output:**

```
Zero
Positive: 5
Negative: -3
```

**Step-by-Step Breakdown:**

1. `0` matches the literal pattern.
2. `5` matches the capture pattern `n`, and the guard `n > 0` is true.
3. `-3` matches the capture pattern `n`, but the first guard fails; the second guard `n < 0` is true.

**Why the Output is Produced:**

Guards add runtime conditions to capture patterns, allowing numeric classification without nested `if` statements.

---

**Example 2: Guard with Sequence Pattern**

```python
def parse_range(data):
    match data:
        case [start, end] if start < end:
            return f"Range from {start} to {end}"
        case [start, end]:
            return "Invalid range (start >= end)"
        case _:
            return "Invalid format"

print(parse_range([1, 10]))
print(parse_range([10, 1]))
print(parse_range([5]))
```

**Expected Output:**

```
Range from 1 to 10
Invalid range (start >= end)
Invalid format
```

**Step-by-Step Breakdown:**

1. `[1, 10]` matches the sequence pattern; the guard `1 < 10` is true.
2. `[10, 1]` matches the sequence pattern, but the guard is false; it falls to the next case.
3. `[5]` does not match the two-element pattern, so the wildcard handles it.

**Why the Output is Produced:**

Guards allow a single structural pattern to serve multiple cases distinguished by runtime conditions.

#### Real-World Cases

- **Numeric validation**: Checking ranges, thresholds, or positivity after destructuring.
- **Permission checks**: Guarding on user roles or access levels.
- **Protocol validation**: Ensuring sequence lengths or field values meet constraints.
- **Game logic**: Matching events only when certain game-state conditions hold.

---

### 12. Pattern Matching Design, Use Cases, and Best Practices

#### Definitions

**Core Definition**

Pattern matching design and best practices encompass guidelines for deciding when to use `match`/`case`, how to order patterns, how to combine pattern types, and how to avoid common pitfalls, ensuring that the feature improves readability and correctness rather than adding complexity.

**Technical Definition**

Best practices for structural pattern matching include: ordering cases from most specific to most general; always providing a wildcard or exhaustive coverage for unexpected input; preferring structural patterns over guards when possible; avoiding bare-name captures where a constant comparison is intended; using enums and `__match_args__` deliberately; keeping patterns shallow and readable; and recognizing that `match` is not a switch statement but a destructuring and dispatch tool. Design considerations include choosing `match` for branching that also destructures, and avoiding it for simple value comparisons where `if`/`elif` or dictionary dispatch is clearer.

**Beginner-Friendly Explanation**

Best practices are the “rules of thumb” for using pattern matching well. The biggest mistake is thinking `match` is just a fancy `switch`—it’s much more. Use it when you need to look inside data and pull it apart, not when you’re just comparing a single value to a few constants. Put specific cases first and a catch-all `_` last. Avoid using bare variable names when you mean to compare against a constant, because a bare name captures everything. And don’t nest patterns so deeply that they become a puzzle.

#### Purposes

- To decide when structural pattern matching is the right tool versus `if`/`elif` or dictionary dispatch.
- To order cases for correctness and readability.
- To ensure exhaustive handling of possible inputs.
- To avoid common semantic mistakes (e.g., capture patterns where literals were intended).
- To integrate pattern matching effectively with dataclasses, enums, and APIs.
- To keep patterns maintainable as data structures evolve.

#### Syntax Rules and Structure

**Complete General Syntax**

```
match subject:
    case specific_pattern:
        handle_specific()
    case less_specific_pattern:
        handle_less_specific()
    case _:
        handle_default()
```

**Component Breakdown**

| Practice | Description |
|----------|-------------|
| Specific-first ordering | Put narrower patterns before broader ones. |
| Wildcard last | `case _:` should be the final catch-all. |
| Guard sparingly | Use guards only for conditions patterns cannot express. |
| Prefer literals/dotted names | Avoid bare names for constant comparisons. |
| Shallow patterns | Keep nesting to a manageable depth. |
| Enum usage | Prefer enums for symbolic constants. |
| Exhaustiveness | Ensure all possible shapes are handled or deliberately defaulted. |

**Syntax Rules**

- Place more specific patterns before more general ones.
- Always consider adding `case _:` as a safety net unless the match is truly exhaustive.
- Use dotted names for value matching; bare names capture.
- Keep guard expressions simple and side-effect-free.
- Use `__match_args__` deliberately on classes that will be destructured positionally.
- Test pattern matching with representative and edge-case inputs.

**Constraints and Limitations**

- `match` is not a drop-in replacement for every `if`/`elif` chain; simple comparisons may be clearer with `if`.
- Pattern matching does not enforce exhaustiveness at runtime; forgetting a case silently falls through.
- Deeply nested patterns can obscure logic and are hard to maintain.
- Overuse of guards can turn pattern matching into a convoluted conditional chain.
- Bare-name capture mistakes are a common source of bugs.

#### Annotated Code Examples

**Example 1: Specific-First Ordering and Wildcard**

```python
def route_message(msg):
    match msg:
        case {"type": "click", "target": target, "position": pos}:
            return f"Click on {target} at {pos}"
        case {"type": "click"}:
            return "Incomplete click event"
        case {"type": "keypress", "key": key}:
            return f"Key pressed: {key}"
        case {"type": "keypress"}:
            return "Incomplete keypress event"
        case {"type": "scroll", "delta": delta}:
            return f"Scroll by {delta}"
        case {"type": "scroll"}:
            return "Scroll by 0"
        case _:
            return "Unknown message type"

print(route_message({"type": "click", "target": "btn", "position": (10, 20)}))
print(route_message({"type": "click"}))
print(route_message({"type": "unknown"}))
```

**Expected Output:**

```
Click on btn at (10, 20)
Incomplete click event
Unknown message type
```

**Step-by-Step Breakdown:**

1. The most specific click pattern (with target and position) is first.
2. The less specific click pattern (only type) follows.
3. The wildcard handles unrecognized types.

**Why the Output is Produced:**

Specific-first ordering ensures that richer patterns are tried before broader ones, preventing premature matches.

---

**Example 2: Avoiding Bare-Name Capture Bugs**

```python
# BAD: Bare name captures everything; the "status" comparison never works.
STATUS_OK = 200
def check_bad(value):
    match value:
        case STATUS_OK:
            return "OK"
        case _:
            return "Not OK"

# GOOD: Use a dotted name or literal.
def check_good(value):
    match value:
        case 200:
            return "OK"
        case _:
            return "Not OK"

print(check_bad(404))   # "OK" — unexpected!
print(check_good(404))  # "Not OK"
```

**Expected Output:**

```
OK
Not OK
```

**Step-by-Step Breakdown:**

1. In `check_bad`, `STATUS_OK` is a bare name, so it captures the subject and always matches.
2. In `check_good`, the literal `200` is used, so only `200` matches.

**Why the Output is Produced:**

A bare name in a pattern is always a capture pattern, regardless of the variable’s value. Dotted names or literals are required for value comparisons.

#### Real-World Cases

- **API routing**: Dispatching on JSON payload shape with mapping patterns and specific-first ordering.
- **Parser design**: Matching AST node classes and destructuring their children.
- **State machines**: Dispatching on event classes and attribute values.
- **CLI tools**: Parsing command tokens with sequence patterns and guards.
- **Security tooling**: Routing protocol messages by structure and opcode.

---

## References

- PEP 634 – Structural Pattern Matching: Specification - https://peps.python.org/pep-0634/
- PEP 635 – Structural Pattern Matching: Motivation and Rationale - https://peps.python.org/pep-0635/
- PEP 636 – Structural Pattern Matching: Tutorial - https://peps.python.org/pep-0636/
- Python Documentation – The `match` Statement (canonical reference) - https://docs.python.org/3/reference/compound_stmts.html#the-match-statement
- Python Documentation – What’s New in Python 3.10 - https://docs.python.org/3/whatsnew/3.10.html
- Python Match Case: Structural Pattern Matching Explained (Python 3.10+) - https://docs.kanaries.net/topics/Python/python-match-case
- Python Switch Statement (`match-case`): Syntax, Examples, Best Practices - https://phoenixnap.com/kb/python-switch-statement
- Python-for-Security-Professionals – Match Statement - https://github.com/armourinfosec/Python-for-Security-Professionals/blob/main/Python-Statements-and-Control-Flow/Match-Statement.md
- dot-skills – Use `match` Statement for Structural Pattern Matching - https://github.com/pproenca/dot-skills/blob/master/skills/.curated/python/references/py-match-statement.md
- Python 3.10 模式匹配：告别冗长的 if-elif 链条 - https://developer.aliyun.com/article/1691625