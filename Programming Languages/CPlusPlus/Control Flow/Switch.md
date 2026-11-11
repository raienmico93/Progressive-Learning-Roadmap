# Switch Statements — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
A `switch` statement is a control-flow construct that selects one of several **labeled branches** (`case` labels) based on the value of an integral or enumeration expression.

**Technical Definition**
The `switch` statement transfers control to a `case` label whose constant expression equals the value of the switch condition (after integral promotion). If no case matches, control transfers to the `default` label (if present). Cases are labels within the switch body; control falls through from one case to the next unless explicitly terminated by `break`, `return`, `continue`, `goto`, or `throw`. Since C++17, `switch` may include an initializer. Since C++17, `[[fallthrough]]` documents intentional fall-through. The compiler may implement `switch` as a **jump table** (constant-time dispatch) or a series of comparisons, depending on the case values and target platform.

**Beginner-Friendly Explanation**
A `switch` statement is like a **multi-way train switch**: the value you're switching on determines which track (case) the program takes. You list the possible values as `case` labels, and the program jumps to the matching one. If none match, the `default` branch runs. Without a `break`, the program “falls through” to the next case — which is sometimes intentional but often a bug. `switch` is often faster and clearer than long `if`-`else if` chains for integer comparisons.

### Key Characteristics

- **Integral/enumeration condition**: The switch condition must be an integral or enumeration type (or a class type with a single conversion to integral/enumeration).
- **Constant case labels**: Each `case` label must be a constant expression, unique within the switch.
- **Fall-through by default**: Control falls through to the next case unless explicitly terminated.
- **`break` exits**: `break` exits the switch statement.
- **`default` fallback**: The `default` label handles unmatched values.
- **Jump tables**: Compilers may use jump tables for efficient dispatch.
- **C++17 features**: `switch` with initializer; `[[fallthrough]]` attribute.
- **No `break` at the end**: The last case doesn't need `break` (but it's often added for safety).

### Prerequisites

- Understanding of integral types, enumerations, and constant expressions.
- Familiarity with `break`, `continue`, and control flow.
- Knowledge of scope and labels.
- Basic awareness of compiler optimizations (jump tables).

### Related Programming Areas

- **Control flow** – `if`/`else`, loops, `goto`.
- **Enumerations** – `switch` is commonly used with `enum` types.
- **State machines** – switch on state values.
- **Parsing** – switch on token types.
- **Compiler optimization** – jump tables.
- **Error handling** – switch on error codes.

### Core Concepts / Features

1. `switch`
2. `case`
3. `break`
4. `default`
5. Fall-through
6. `[[fallthrough]]`
7. `switch` with Initializer (C++17)
8. Jump Tables
9. Switch Best Practices
10. Switch vs. `if`-`else`

---

## 1. `switch`

### Definitions

**Core Definition**
The `switch` statement selects one of several branches based on the value of an integral or enumeration expression.

**Technical Definition**
The `switch` statement has the form `switch ( condition ) statement`, where `condition` is an integral or enumeration expression (or a class type contextually convertible to one). The statement is typically a compound statement containing `case` and `default` labels. Control transfers to the matching `case` label (after integral promotion of the condition); if no case matches, to the `default` label; if no default, the switch does nothing. The `break` statement exits the switch.

**Beginner-Friendly Explanation**
`switch` is a **multi-way branch** on an integer or enum value. You write `switch (x) { case 1: ...; break; case 2: ...; break; }`. The program jumps to the matching case.

### Purposes

- To select among several discrete alternatives.
- To handle enumeration values exhaustively.
- To implement state machines and parsers.
- To replace long `else if` chains for integer comparisons.
- To enable efficient jump-table dispatch.

### Syntax Rules and Structure

**General Syntax**
```cpp
switch (condition) {
    case constant1:
        // ...
        break;
    case constant2:
        // ...
        break;
    default:
        // ...
        break;
}
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `condition` | Integral or enumeration expression. |
| `case constant:` | A label for a constant value. |
| `default:` | Fallback label. |
| `break;` | Exits the switch. |

**Syntax Rules**
- The condition must be integral or enumeration (or a class with a single conversion).
- Case labels must be constant expressions.
- Case labels must be unique within the switch.
- `default` is optional but recommended.
- `break` exits the switch; without it, control falls through.
- Cases can be grouped (multiple labels for the same code).

**Constraints and Limitations**
- Cannot switch on floating-point, pointer, or class types (without conversion).
- Case labels must be constant expressions.
- Fall-through can be a bug if unintended.
- Variables with initialization cannot be declared directly in a case without braces (to avoid jumping over initialization).

### Annotated Code Examples

**Example 1: Basic `switch`**

```cpp
#include <iostream>

int main() {
    int x = 2;
    switch (x) {
        case 1:
            std::cout << "one\n";
            break;
        case 2:
            std::cout << "two\n";
            break;
        case 3:
            std::cout << "three\n";
            break;
        default:
            std::cout << "other\n";
            break;
    }
}
```

**Expected Output**
```
two
```

**Why this output?**  
`x` is `2`, so control jumps to `case 2`, prints `"two"`, and `break` exits the switch.

### Real-World Cases

- **State machines** – switch on state enum.
- **Parsers** – switch on token type.
- **Command handlers** – switch on command code.
- **Grade classification** – switch on letter grade.

---

## 2. `case`

### Definitions

**Core Definition**
A `case` label marks a branch of a `switch` statement that is selected when the switch condition equals the label's constant value.

**Technical Definition**
A `case` label has the form `case constant-expression :`. The constant expression must be an integral or enumeration constant that is implicitly convertible to the switch condition's type (after integral promotion). Multiple `case` labels may precede the same statement (grouping). Case labels are unique within a switch. Control transfers to the matching label; without `break`, execution continues into subsequent cases.

**Beginner-Friendly Explanation**
A `case` is a **possible value** the switch checks for. If the switch value equals the case value, execution jumps to that case.

### Purposes

- To define the discrete alternatives of a switch.
- To group cases that share code.
- To handle enumeration values.
- To implement exhaustive matching.

### Syntax Rules and Structure

**General Syntax**
```cpp
case constant-expression :
    // statements
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `constant-expression` | A constant value. |
| `:` | Separates the label from statements. |

**Syntax Rules**
- The constant must be an integral or enumeration constant.
- Case values must be unique.
- Multiple cases can share code.
- Cases can appear in any order.
- A case cannot be empty unless it falls through.

**Constraints and Limitations**
- Case values must be compile-time constants.
- Case values must be unique.
- Case values must be convertible to the switch type.

### Annotated Code Examples

**Example 1: Grouped cases**

```cpp
#include <iostream>

int main() {
    char c = 'a';
    switch (c) {
        case 'a':
        case 'e':
        case 'i':
        case 'o':
        case 'u':
            std::cout << "vowel\n";
            break;
        default:
            std::cout << "consonant\n";
            break;
    }
}
```

**Expected Output**
```
vowel
```

**Why this output?**  
`'a'` matches the grouped vowel cases, so `"vowel"` is printed.

### Real-World Cases

- **Enum handling** – one case per enum value.
- **Character classification** – vowels, digits, etc.
- **Command codes** – one case per command.
- **Error codes** – one case per error.

---

## 3. `break`

### Definitions

**Core Definition**
The `break` statement **exits the nearest enclosing `switch` or loop**, transferring control to the statement after it.

**Technical Definition**
In a `switch` statement, `break` terminates the switch and transfers control to the statement following the switch. Without `break`, control falls through to the next case. In loops, `break` exits the loop. `break` can only appear in a `switch` or loop.

**Beginner-Friendly Explanation**
`break` is the **“stop here and leave”** statement. In a switch, it prevents fall-through to the next case.

### Purposes

- To prevent fall-through between cases.
- To exit a switch after handling a case.
- To exit loops early.

### Syntax Rules and Structure

**General Syntax**
```cpp
break;
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `break;` | Exits the nearest switch or loop. |

**Syntax Rules**
- Can appear only inside a `switch` or loop.
- Exits the nearest enclosing construct.
- Often omitted at the end of a switch (but recommended for safety).

**Constraints and Limitations**
- Cannot break out of multiple levels (use `goto` or restructure).
- Not needed in the last case if no fall-through is desired.

### Annotated Code Examples

**Example 1: `break` preventing fall-through**

```cpp
#include <iostream>

int main() {
    int x = 1;
    switch (x) {
        case 1:
            std::cout << "one\n";
            break;   // exits switch
        case 2:
            std::cout << "two\n";
            break;
    }
}
```

**Expected Output**
```
one
```

**Why this output?**  
`case 1` prints `"one"`, and `break` exits the switch, preventing fall-through to `case 2`.

### Real-World Cases

- **Case termination** – preventing fall-through.
- **Loop exit** – breaking out of loops early.
- **Search** – stopping when a match is found.

---

## 4. `default`

### Definitions

**Core Definition**
The `default` label marks the branch of a `switch` that executes when no `case` matches the switch value.

**Technical Definition**
The `default` label has the form `default :`. It is optional and may appear anywhere in the switch body (but conventionally last). If no case matches and a `default` exists, control transfers to the `default` label. Only one `default` is allowed per switch. Like cases, `default` falls through without `break`.

**Beginner-Friendly Explanation**
`default` is the **“none of the above”** branch. If no case matches, the `default` code runs.

### Purposes

- To handle unmatched values.
- To provide a fallback behavior.
- To catch unexpected values (e.g., invalid enum).
- To improve robustness.

### Syntax Rules and Structure

**General Syntax**
```cpp
default:
    // statements
    break;
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `default:` | The fallback label. |
| Statements | The fallback code. |

**Syntax Rules**
- At most one `default` per switch.
- May appear anywhere but conventionally last.
- Falls through without `break`.
- Optional but recommended.

**Constraints and Limitations**
- Only one `default` is allowed.
- Without `default`, unmatched values do nothing.

### Annotated Code Examples

**Example 1: `default` handling unmatched values**

```cpp
#include <iostream>

int main() {
    int x = 5;
    switch (x) {
        case 1:
            std::cout << "one\n";
            break;
        case 2:
            std::cout << "two\n";
            break;
        default:
            std::cout << "unknown\n";
            break;
    }
}
```

**Expected Output**
```
unknown
```

**Why this output?**  
`x` is `5`, which matches no case, so the `default` branch runs.

### Real-World Cases

- **Error handling** – catch unexpected values.
- **Enum completeness** – handle invalid enum values.
- **Robustness** – provide fallback behavior.

---

## 5. Fall-through

### Definitions

**Core Definition**
Fall-through is the behavior where, after executing a case's statements, control **continues into the next case** because there is no `break`.

**Technical Definition**
In a `switch` statement, control falls through from one case to the next unless explicitly terminated by `break`, `return`, `continue`, `goto`, or `throw`. Fall-through can be intentional (e.g., grouped cases) or accidental (a bug). Since C++17, `[[fallthrough]]` documents intentional fall-through.

**Beginner-Friendly Explanation**
Fall-through means **slipping into the next case**. If you forget `break`, the program runs the next case's code too. It's sometimes intentional, but usually a bug.

### Purposes

- To share code across multiple cases (intentional fall-through).
- To implement cascading logic.
- To avoid duplicating code.

### Syntax Rules and Structure

**Intentional Fall-through**
```cpp
switch (x) {
    case 1:
    case 2:
    case 3:
        // shared code
        break;
    case 4:
        do_something();
        [[fallthrough]];   // C++17: intentional fall-through
    case 5:
        do_more();
        break;
}
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| No `break` | Causes fall-through. |
| `[[fallthrough]]` | Documents intentional fall-through (C++17). |

**Syntax Rules**
- Fall-through occurs when no terminating statement is reached.
- `[[fallthrough]]` documents intent and suppresses warnings.
- Grouped cases (empty cases) are a common intentional fall-through.

**Constraints and Limitations**
- Accidental fall-through is a common bug.
- Compilers warn about implicit fall-through (`-Wimplicit-fallthrough`).
- `[[fallthrough]]` must appear immediately before the next case.

### Annotated Code Examples

**Example 1: Intentional fall-through**

```cpp
#include <iostream>

int main() {
    int x = 1;
    switch (x) {
        case 1:
            std::cout << "one ";
            [[fallthrough]];
        case 2:
            std::cout << "two ";
            [[fallthrough]];
        case 3:
            std::cout << "three\n";
            break;
    }
}
```

**Expected Output**
```
one two three
```

**Why this output?**  
`case 1` falls through to `case 2` and `case 3`, printing all three labels.

### Real-World Cases

- **Cascading logic** – cumulative effects.
- **Grouped cases** – shared code.
- **State transitions** – sequential handling.

---

## 6. `[[fallthrough]]`

### Definitions

**Core Definition**
`[[fallthrough]]` is a C++17 attribute that **documents intentional fall-through** in a `switch` statement, suppressing compiler warnings.

**Technical Definition**
The `[[fallthrough]]` attribute appears as a statement immediately before a `case` or `default` label. It indicates that execution is intentionally falling through from the previous case. Compilers may warn about implicit fall-through (`-Wimplicit-fallthrough`); `[[fallthrough]]` documents intent and suppresses such warnings.

**Beginner-Friendly Explanation**
`[[fallthrough]]` is a **note to the compiler and readers**: “I meant to fall through here.” It prevents warnings and makes intent clear.

### Purposes

- To document intentional fall-through.
- To suppress compiler warnings.
- To make code intent explicit.
- To improve maintainability.

### Syntax Rules and Structure

**General Syntax**
```cpp
case 1:
    do_something();
    [[fallthrough]];
case 2:
    // ...
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `[[fallthrough]];` | Attribute statement. |
| Placement | Immediately before the next case/default. |

**Syntax Rules**
- Requires C++17.
- Must appear as a statement before a case label.
- Must be reachable.
- Suppresses fall-through warnings.

**Constraints and Limitations**
- Must be immediately before a case/default label.
- Cannot be used in a loop (only in switch).
- Does not change behavior; only documents intent.

### Annotated Code Examples

**Example 1: Documenting fall-through**

```cpp
#include <iostream>

int main() {
    int x = 1;
    switch (x) {
        case 1:
            std::cout << "one ";
            [[fallthrough]];
        case 2:
            std::cout << "two\n";
            break;
    }
}
```

**Expected Output**
```
one two
```

**Why this output?**  
`case 1` intentionally falls through to `case 2`, printing both.

### Real-World Cases

- **Cascading logic** – documenting intentional fall-through.
- **Code review** – making intent explicit.
- **Compiler warnings** – suppressing `-Wimplicit-fallthrough`.

---

## 7. `switch` with Initializer (C++17)

### Definitions

**Core Definition**
`switch` with initializer allows a variable to be declared and initialized in the switch condition, scoped to the switch statement.

**Technical Definition**
Since C++17, the `switch` statement can include an initializer: `switch ( init-statement condition ) statement`. The initializer declares a variable scoped to the switch. This is useful for switching on the result of a function call while keeping the result scoped.

**Beginner-Friendly Explanation**
`switch` with initializer lets you **declare a variable inside the switch condition**, scoped to the switch. It's useful for `switch (auto x = f(); x)`.

### Purposes

- To scope a variable to the switch statement.
- To switch on the result of a function call.
- To combine declaration and condition.
- To improve code clarity.

### Syntax Rules and Structure

**General Syntax**
```cpp
switch (init-statement condition) {
    // cases
}
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `init-statement` | Declaration or expression statement. |
| `condition` | The switch condition. |

**Syntax Rules**
- Requires C++17.
- The initializer runs before the condition.
- Variables declared are scoped to the switch.
- The semicolon separates the initializer from the condition.

**Constraints and Limitations**
- Requires C++17.
- The initializer must be a declaration or expression statement.

### Annotated Code Examples

**Example 1: `switch` with initializer**

```cpp
#include <iostream>

int compute() { return 2; }

int main() {
    switch (int x = compute(); x) {
        case 1:
            std::cout << "one\n";
            break;
        case 2:
            std::cout << "two\n";
            break;
        default:
            std::cout << "other\n";
            break;
    }
}
```

**Expected Output**
```
two
```

**Why this output?**  
`compute()` returns `2`, which matches `case 2`.

### Real-World Cases

- **Function result switching** – `switch (auto x = f(); x)`.
- **Enum parsing** – switch on parsed enum.
- **Error code handling** – switch on error result.

---

## 8. Jump Tables

### Definitions

**Core Definition**
A jump table is a compiler-generated array of code addresses used to implement `switch` statements with **constant-time dispatch**.

**Technical Definition**
When a `switch` statement has many case labels with a dense range of values, the compiler may generate a **jump table**: an array indexed by the switch value, containing the addresses of the case handlers. This allows O(1) dispatch instead of O(n) comparisons. If the case values are sparse, the compiler may use a series of comparisons or a binary search tree instead. Jump tables are an implementation detail; the standard does not require them.

**Beginner-Friendly Explanation**
A jump table is a **lookup table of code addresses**. For dense switch values, the compiler builds a table so the program can jump directly to the right case in constant time, without checking each case.

### Purposes

- To provide efficient switch dispatch.
- To avoid O(n) comparisons.
- To optimize dense switch ranges.
- To improve performance in hot code paths.

### Syntax Rules and Structure

Jump tables are an implementation detail; no explicit syntax.

**Conceptual Jump Table**
```
switch (x):
    jump_table[x] → case handler
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| Jump table | Array of code addresses. |
| Index | Switch value (offset by minimum). |
| Dispatch | O(1) jump. |

**Syntax Rules**
- Generated automatically by the compiler.
- Used for dense case values.
- Not used for sparse cases (comparisons instead).

**Constraints and Limitations**
- Implementation-defined.
- Sparse cases may not use jump tables.
- Jump tables increase code size.

### Annotated Code Examples

**Example 1: Dense switch (jump table candidate)**

```cpp
#include <iostream>

int main() {
    int x = 3;
    switch (x) {
        case 0: std::cout << "zero\n"; break;
        case 1: std::cout << "one\n"; break;
        case 2: std::cout << "two\n"; break;
        case 3: std::cout << "three\n"; break;
        case 4: std::cout << "four\n"; break;
        default: std::cout << "other\n"; break;
    }
}
```

**Expected Output**
```
three
```

**Why this is a candidate?**  
The case values are dense (`0`–`4`), so the compiler may use a jump table for O(1) dispatch.

### Real-World Cases

- **Parsers** – dense token types.
- **State machines** – dense state values.
- **Interpreter dispatch** – opcode tables.

---

## 9. Switch Best Practices

### Definitions

**Core Definition**
Switch best practices are guidelines for writing clear, correct, and maintainable `switch` statements.

**Technical Definition**
These practices are based on the C++ Core Guidelines (especially **ES.70** on preferring `switch` to `if` when there is a choice) and community standards. Key principles: always handle `default`; use `break` consistently; document intentional fall-through with `[[fallthrough]]`; group cases that share code; avoid declaring variables directly in cases without braces; use `switch` for integral/enumeration dispatch; and consider `switch` vs. `if`-`else` based on clarity and performance.

**Beginner-Friendly Explanation**
Best practices for `switch` are simple: **always include `default`**, **use `break` consistently**, **document intentional fall-through**, **group shared cases**, **avoid variable declarations in cases without braces**, and **use `switch` for discrete integer/enum values**.

### Key Guidelines (Summary Table)

| Guideline | Summary | Recommendation |
|-----------|---------|----------------|
| **Always `default`** | Handle unmatched values. | Add a `default` branch. |
| **Consistent `break`** | Prevent accidental fall-through. | Use `break` unless intentional. |
| **Document fall-through** | Use `[[fallthrough]]`. | Make intent explicit. |
| **Group cases** | Share code across cases. | Empty cases fall through to shared code. |
| **Braces for declarations** | Avoid jumping over initialization. | Use `{}` around case bodies with declarations. |
| **Use for integers/enums** | `switch` is for discrete values. | Prefer `if`-`else` for ranges. |
| **`ES.70`** | Prefer `switch` to `if` when there is a choice. | Clearer and often faster. |

### Annotated Code Examples

**Example 1: `default`, `break`, and grouping**

```cpp
#include <iostream>

int main() {
    int x = 2;
    switch (x) {
        case 1:
        case 2:
        case 3:
            std::cout << "small\n";
            break;
        case 4:
            std::cout << "four\n";
            break;
        default:
            std::cout << "other\n";
            break;
    }
}
```

**Expected Output**
```
small
```

**Why this is best practice?**  
Grouped cases share code; `break` prevents fall-through; `default` handles unmatched values.

### Real-World Cases

- **Code review** – `switch` statements are reviewed for `default` and `break`.
- **Static analysis** – clang-tidy checks for missing `default` and fall-through.
- **Performance** – `switch` can be faster than `if`-`else` for dense integers.

---

## 10. Switch vs. `if`-`else`

### Definitions

**Core Definition**
`switch` and `if`-`else` are both branching constructs; `switch` is suited for **discrete integer/enumeration values**, while `if`-`else` is suited for **ranges, complex conditions, and non-integral types**.

**Technical Definition**
`switch` dispatches on an integral or enumeration value using constant case labels; it can be implemented with jump tables for O(1) dispatch. `if`-`else` chains evaluate arbitrary boolean expressions in order; they support ranges, floating-point, pointers, and complex conditions. The C++ Core Guidelines recommend preferring `switch` to `if`-`else` when there is a choice (ES.70), because `switch` is clearer and often more efficient for discrete values.

**Beginner-Friendly Explanation**
Use `switch` when you have a **list of discrete integer or enum values** to handle. Use `if`-`else` when you have **ranges, complex conditions, or non-integer types**.

### Purposes

- To choose the right branching construct.
- To improve readability and performance.
- To handle enumerations exhaustively with `switch`.
- To handle ranges and complex conditions with `if`-`else`.

### Comparison Table

| Aspect | `switch` | `if`-`else` |
|--------|----------|-------------|
| **Condition type** | Integral/enumeration | Any boolean expression |
| **Case values** | Constants | Arbitrary expressions |
| **Ranges** | Not directly | Yes |
| **Dispatch** | Jump table possible | Sequential |
| **Fall-through** | Yes (default) | No |
| **Clarity** | Good for discrete values | Good for complex conditions |
| **`default`** | Recommended | `else` fallback |
| **C++ Core Guidelines** | **ES.70**: prefer `switch` when there is a choice | Use for ranges/complex conditions |

### Annotated Code Examples

**Example 1: `switch` for discrete values**

```cpp
#include <iostream>

int main() {
    int day = 3;
    switch (day) {
        case 1: std::cout << "Mon\n"; break;
        case 2: std::cout << "Tue\n"; break;
        case 3: std::cout << "Wed\n"; break;
        default: std::cout << "Other\n"; break;
    }
}
```

**Expected Output**
```
Wed
```

**Why `switch` here?**  
The values are discrete integers, making `switch` clear and efficient.

**Example 2: `if`-`else` for ranges**

```cpp
#include <iostream>

int main() {
    int score = 85;
    if (score >= 90) {
        std::cout << "A\n";
    } else if (score >= 80) {
        std::cout << "B\n";
    } else if (score >= 70) {
        std::cout << "C\n";
    } else {
        std::cout << "F\n";
    }
}
```

**Expected Output**
```
B
```

**Why `if`-`else` here?**  
The conditions are ranges, which `switch` cannot express directly.

### Real-World Cases

- **State machines** – `switch` on state enum.
- **Grade classification** – `if`-`else` for ranges.
- **Command dispatch** – `switch` on command code.
- **Configuration** – `if`-`else` for complex conditions.

---

## References

- `switch` statement (cppreference) – https://en.cppreference.com/w/cpp/language/switch
- `break` statement (cppreference) – https://en.cppreference.com/w/cpp/language/break
- `[[fallthrough]]` attribute (cppreference) – https://en.cppreference.com/w/cpp/language/attributes/fallthrough
- C++ Core Guidelines — ES.70: Prefer a `switch`-statement to an `if`-statement when there is a choice – https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Res-switch-if
- C++ Core Guidelines — ES.78: Don't rely on implicit fallthrough in `switch` statements – https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Res-break
- C++ Core Guidelines — ES.79: Use `default` to handle common cases (only) – https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Res-default
- Microsoft Learn — `switch` statement (C++) – https://learn.microsoft.com/en-us/cpp/cpp/switch-statement-cpp
- Microsoft Learn — `[[fallthrough]]` – https://learn.microsoft.com/en-us/cpp/cpp/attributes
- Jump table (Wikipedia) – https://en.wikipedia.org/wiki/Branch_table
- GCC `-Wimplicit-fallthrough` – https://gcc.gnu.org/onlinedocs/gcc/Warning-Options.html

---

This cheat sheet provides a complete, structured reference for C++ switch statements, covering `switch`, `case`, `break`, `default`, fall-through, `[[fallthrough]]`, `switch` with initializer (C++17), jump tables, best practices, and `switch` vs. `if`-`else`. Each section follows a uniform template with definitions, purposes, syntax rules, annotated examples, expected outputs, and authoritative references.