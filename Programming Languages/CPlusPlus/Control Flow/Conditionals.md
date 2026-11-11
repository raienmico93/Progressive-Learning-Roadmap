# Conditional Statements — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
A conditional statement is a control-flow construct that executes different blocks of code depending on whether a condition evaluates to `true` or `false`, enabling a program to make decisions.

**Technical Definition**
C++ provides several conditional constructs: the `if` statement, `if`–`else`, `else if` chains, nested conditionals, the conditional (ternary) operator (`?:`), `if` with an initializer (C++17), `if constexpr` (C++17) for compile-time branching in templates, and `if consteval` (C++23) for branching on whether evaluation is happening in a constant expression. Conditions are contextually converted to `bool`; the statement controlled by a condition may be a single statement or a compound statement (`{}`). Conditional statements are the primary mechanism for decision-making and control flow in C++.

**Beginner-Friendly Explanation**
Conditional statements let your program **make choices**. They're the “if this, then that” of programming. If it's raining, take an umbrella; otherwise, wear sunglasses. In C++, you write `if (condition) { ... } else { ... }`. The condition is checked, and the appropriate block runs. You can chain conditions with `else if`, nest them inside each other, use the compact ternary operator for simple choices, and even make choices at **compile time** with `if constexpr`.

### Key Characteristics

- **Decision-making**: Conditional statements select which code to execute based on runtime or compile-time conditions.
- **Contextual conversion to `bool`**: Conditions are converted to `bool` (e.g., `if (ptr)` checks for null).
- **Single vs. compound statements**: The body may be a single statement or a `{}` block.
- **Dangling `else`**: An `else` binds to the nearest unmatched `if`.
- **Ternary operator**: A compact expression-level conditional.
- **`if` with initializer (C++17)**: Declares a variable scoped to the `if`/`else` statement.
- **`if constexpr` (C++17)**: Compile-time branching in templates; the untaken branch is discarded.
- **`if consteval` (C++23)**: Branches on whether evaluation is a constant expression.

### Prerequisites

- Understanding of expressions, operators, and boolean logic.
- Familiarity with statements and blocks.
- Knowledge of scope and lifetime.
- Basic awareness of templates (for `if constexpr`) and constant evaluation (for `if consteval`).

### Related Programming Areas

- **Control flow** – `if`, `switch`, loops.
- **Boolean logic** – `&&`, `||`, `!`, comparison operators.
- **Templates** – `if constexpr` for compile-time branching.
- **Constant evaluation** – `if consteval` for constant-expression contexts.
- **Error handling** – conditional checks for error states.
- **Design patterns** – conditionals underlie strategy selection and state machines.

### Core Concepts / Features

1. `if`
2. `else if`
3. `else`
4. Nested Conditionals
5. Ternary Operator
6. `if` with Initializer (C++17)
7. `if constexpr` (C++17)
8. `if consteval` (C++23)
9. Conditional Best Practices

---

## 1. `if`

### Definitions

**Core Definition**
The `if` statement executes a statement (or block) only if a condition evaluates to `true`.

**Technical Definition**
The `if` statement has the form `if ( condition ) statement`. The condition is contextually converted to `bool`; if it evaluates to `true`, the statement is executed; otherwise, it is skipped. The statement may be a single statement or a compound statement (`{}`). The condition may be an expression or (since C++17) a declaration with an initializer.

**Beginner-Friendly Explanation**
`if` is the **“do this only when…”** statement. If the condition is true, run the code inside; otherwise, skip it.

### Purposes

- To execute code conditionally.
- To guard against invalid states (e.g., null pointers).
- To implement decision logic.
- To control program flow based on runtime values.

### Syntax Rules and Structure

**General Syntax**
```cpp
if (condition) {
    // executed if condition is true
}
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `condition` | Expression contextually converted to `bool`. |
| `statement` | Single statement or compound statement. |

**Syntax Rules**
- The condition must be contextually convertible to `bool`.
- The body may be a single statement or a `{}` block.
- An `if` without braces controls only the next statement (a common source of bugs).
- The condition may declare a variable (C++17).

**Constraints and Limitations**
- Conditions with side effects can be confusing.
- The dangling `else` problem arises without braces.
- Empty statements (`if (x);`) are often mistakes.

### Annotated Code Examples

**Example 1: Basic `if`**

```cpp
#include <iostream>

int main() {
    int x = 10;
    if (x > 5) {
        std::cout << "x is greater than 5\n";
    }
}
```

**Expected Output**
```
x is greater than 5
```

**Why this output?**  
`x > 5` is `true`, so the block executes.

### Real-World Cases

- **Input validation** – `if (input.empty()) return;`.
- **Null checks** – `if (ptr != nullptr) ptr->use();`.
- **Bounds checks** – `if (index < size) ...`.
- **Feature toggles** – `if (feature_enabled) ...`.

---

## 2. `else if`

### Definitions

**Core Definition**
`else if` chains multiple conditions, testing each in order until one is true.

**Technical Definition**
`else if` is not a separate statement but a combination of `else` and `if`: `if (c1) s1 else if (c2) s2 else s3`. The conditions are evaluated in order; the first true condition's statement is executed, and the rest are skipped. If none are true, the final `else` (if present) executes.

**Beginner-Friendly Explanation**
`else if` lets you check **multiple conditions in sequence**. The first one that's true wins; the rest are skipped.

### Purposes

- To handle multiple mutually exclusive cases.
- To implement classification logic.
- To avoid deeply nested `if` statements.
- To express ordered decision rules.

### Syntax Rules and Structure

**General Syntax**
```cpp
if (condition1) {
    // ...
} else if (condition2) {
    // ...
} else if (condition3) {
    // ...
} else {
    // ...
}
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `if (c1)` | First condition. |
| `else if (c2)` | Subsequent conditions. |
| `else` | Fallback (optional). |

**Syntax Rules**
- Conditions are evaluated in order.
- The first true condition's block executes.
- The final `else` is optional.
- Each branch can have its own scope.

**Constraints and Limitations**
- Long `else if` chains can be hard to read; consider `switch` or polymorphism.
- Order matters: earlier conditions take precedence.

### Annotated Code Examples

**Example 1: `else if` chain**

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

**Why this output?**  
`score >= 90` is false, but `score >= 80` is true, so `"B"` is printed.

### Real-World Cases

- **Grade classification** – A/B/C/F.
- **HTTP status handling** – different responses per status code range.
- **State machines** – transitions based on conditions.
- **Configuration parsing** – different formats.

---

## 3. `else`

### Definitions

**Core Definition**
The `else` clause specifies the code to execute when the `if` condition is false.

**Technical Definition**
The `else` clause follows an `if` statement: `if (condition) statement1 else statement2`. If the condition is true, `statement1` executes; otherwise, `statement2` executes. The `else` is optional. An `else` binds to the nearest unmatched `if` (the dangling `else` problem).

**Beginner-Friendly Explanation**
`else` is the **“otherwise”** branch. If the `if` condition is false, the `else` block runs.

### Purposes

- To provide an alternative path when the condition is false.
- To handle both branches of a decision.
- To ensure all cases are covered.

### Syntax Rules and Structure

**General Syntax**
```cpp
if (condition) {
    // true branch
} else {
    // false branch
}
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `if (condition)` | The condition. |
| `else` | The alternative branch. |

**Syntax Rules**
- The `else` is optional.
- It binds to the nearest unmatched `if`.
- Braces are recommended to avoid dangling `else` bugs.

**Constraints and Limitations**
- Without braces, the `else` may bind to the wrong `if`.
- An `else` cannot exist without a preceding `if`.

### Annotated Code Examples

**Example 1: `if`–`else`**

```cpp
#include <iostream>

int main() {
    int x = 3;
    if (x % 2 == 0) {
        std::cout << "even\n";
    } else {
        std::cout << "odd\n";
    }
}
```

**Expected Output**
```
odd
```

**Why this output?**  
`3 % 2 != 0`, so the `else` branch executes.

### Real-World Cases

- **Binary decisions** – yes/no, valid/invalid.
- **Error handling** – success/failure branches.
- **Default cases** – fallback behavior.

---

## 4. Nested Conditionals

### Definitions

**Core Definition**
Nested conditionals are `if`/`else` statements placed inside the branches of other `if`/`else` statements.

**Technical Definition**
A nested conditional is an `if` statement whose controlled statement is itself an `if` statement (or contains one). Nesting allows multi-level decision logic. The dangling `else` problem is especially relevant: an `else` binds to the nearest unmatched `if`, which may not be the intended one. Braces should be used to make the binding explicit.

**Beginner-Friendly Explanation**
Nested conditionals are **if statements inside if statements**. They let you check a second condition only if the first one passes.

### Purposes

- To implement multi-level decision logic.
- To refine conditions progressively.
- To express hierarchical rules.

### Syntax Rules and Structure

**General Syntax**
```cpp
if (condition1) {
    if (condition2) {
        // both true
    } else {
        // condition1 true, condition2 false
    }
} else {
    // condition1 false
}
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| Outer `if` | First-level condition. |
| Inner `if` | Second-level condition. |
| Braces | Clarify binding. |

**Syntax Rules**
- Inner `if` statements are controlled by outer branches.
- The `else` binds to the nearest unmatched `if`.
- Braces are strongly recommended.

**Constraints and Limitations**
- Deep nesting hurts readability.
- The dangling `else` problem is a common bug.
- Consider `else if` or early returns to flatten nesting.

### Annotated Code Examples

**Example 1: Nested conditionals**

```cpp
#include <iostream>

int main() {
    int x = 10, y = 20;
    if (x > 0) {
        if (y > 0) {
            std::cout << "both positive\n";
        } else {
            std::cout << "x positive, y non-positive\n";
        }
    } else {
        std::cout << "x non-positive\n";
    }
}
```

**Expected Output**
```
both positive
```

**Why this output?**  
Both `x > 0` and `y > 0` are true, so the innermost block executes.

### Real-World Cases

- **Access control** – check user, then check permission.
- **Parsing** – check token type, then check value.
- **Game logic** – check state, then check sub-state.

---

## 5. Ternary Operator

### Definitions

**Core Definition**
The ternary operator (`?:`) is an expression-level conditional that evaluates one of two expressions based on a condition.

**Technical Definition**
The conditional operator has the form `condition ? expr1 : expr2`. The condition is contextually converted to `bool`. If true, `expr1` is evaluated; otherwise, `expr2`. The result is a value (or reference) whose type is determined by the common type of `expr1` and `expr2`. It is the only ternary operator in C++ and has right-to-left associativity.

**Beginner-Friendly Explanation**
The ternary operator is a **compact if-else** that produces a value: `x > 0 ? "positive" : "non-positive"`. It's useful for simple choices.

### Purposes

- To choose between two values inline.
- To simplify simple `if`-`else` assignments.
- To initialize variables conditionally.
- To write concise expressions.

### Syntax Rules and Structure

**General Syntax**
```cpp
condition ? expression1 : expression2
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `condition` | Contextually converted to `bool`. |
| `expression1` | Evaluated if true. |
| `expression2` | Evaluated if false. |

**Syntax Rules**
- Only one of the two expressions is evaluated.
- The result type is the common type of both expressions.
- It has right-to-left associativity (can be chained).
- It can be used as an lvalue if both branches are lvalues of the same type (C++11).

**Constraints and Limitations**
- Deep nesting hurts readability.
- Side effects in branches can be confusing.
- The result type must be compatible.

### Annotated Code Examples

**Example 1: Ternary operator**

```cpp
#include <iostream>

int main() {
    int x = 7;
    std::cout << (x % 2 == 0 ? "even" : "odd") << '\n';
}
```

**Expected Output**
```
odd
```

**Why this output?**  
`7 % 2 != 0`, so the second expression (`"odd"`) is evaluated.

### Real-World Cases

- **Inline selection** – `max = a > b ? a : b;`.
- **Default values** – `name = input.empty() ? "Guest" : input;`.
- **Formatting** – `count == 1 ? "item" : "items"`.
- **Error messages** – `ok ? "success" : "failure"`.

---

## 6. `if` with Initializer (C++17)

### Definitions

**Core Definition**
`if` with an initializer allows a variable to be declared and initialized in the `if` condition, scoped to the `if`/`else` statement.

**Technical Definition**
Since C++17, the `if` statement can include an initializer before the condition: `if ( init-statement condition ) statement`. The initializer is a declaration or expression statement; any variable declared is in scope for the condition and both branches. This is useful for checking the result of a function call while keeping the result scoped to the `if`.

**Beginner-Friendly Explanation**
`if` with an initializer lets you **declare a variable inside the `if` condition**. It's useful for `if (auto result = f(); result)`, where `result` is scoped to the `if`.

### Purposes

- To scope a variable to the `if`/`else` statement.
- To check the result of a function call without polluting the enclosing scope.
- To combine declaration and condition.
- To improve code clarity.

### Syntax Rules and Structure

**General Syntax**
```cpp
if (init-statement condition) {
    // ...
}
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `init-statement` | Declaration or expression statement. |
| `condition` | The condition to test. |

**Syntax Rules**
- The initializer runs before the condition is evaluated.
- Variables declared are in scope for the condition and both branches.
- The initializer can be a declaration or an expression.
- The semicolon separates the initializer from the condition.

**Constraints and Limitations**
- Requires C++17.
- The initializer must be a declaration or expression statement.
- The variable is not accessible after the `if`.

### Annotated Code Examples

**Example 1: `if` with initializer**

```cpp
#include <iostream>
#include <map>

int main() {
    std::map<int, int> m = {{1, 10}, {2, 20}};
    if (auto it = m.find(2); it != m.end()) {
        std::cout << "found: " << it->second << '\n';
    } else {
        std::cout << "not found\n";
    }
}
```

**Expected Output**
```
found: 20
```

**Why this output?**  
`m.find(2)` returns an iterator to `{2, 20}`; the condition is true, and `it->second` is `20`.

### Real-World Cases

- **Map/container lookups** – `if (auto it = m.find(k); it != m.end())`.
- **Error checking** – `if (auto err = f(); err) ...`.
- **Locks** – `if (auto lock = acquire(); lock) ...`.
- **Parsing** – `if (auto result = parse(input); result) ...`.

---

## 7. `if constexpr` (C++17)

### Definitions

**Core Definition**
`if constexpr` is a compile-time conditional that evaluates its condition at compile time and **discards** the untaken branch, enabling template metaprogramming and type-dependent branching.

**Technical Definition**
`if constexpr ( condition ) statement` evaluates `condition` as a constant expression. If true, the first branch is instantiated; if false, the second branch (if any) is instantiated. The untaken branch is **discarded** and not instantiated, which allows code that would be ill-formed for certain template arguments. It can only appear in templates (until C++23, where it can appear anywhere, but the condition must still be a constant expression in some contexts).

**Beginner-Friendly Explanation**
`if constexpr` is a **compile-time if**. The compiler chooses one branch based on a compile-time condition and **ignores the other branch entirely**, so you can write code that only makes sense for certain types.

### Purposes

- To write type-dependent code in templates.
- To avoid instantiating invalid branches.
- To simplify template metaprogramming.
- To replace SFINAE-based branching in many cases.

### Syntax Rules and Structure

**General Syntax**
```cpp
if constexpr (condition) {
    // instantiated if condition is true
} else {
    // instantiated if condition is false
}
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `condition` | Constant expression. |
| True branch | Instantiated if true. |
| False branch | Instantiated if false. |

**Syntax Rules**
- The condition must be a constant expression.
- The untaken branch is discarded (not instantiated).
- Can be used in templates to select code based on types.
- Can be used with `else if constexpr` chains.

**Constraints and Limitations**
- Requires C++17.
- The condition must be a constant expression.
- The discarded branch must still be syntactically valid (but not instantiated).

### Annotated Code Examples

**Example 1: `if constexpr` in a template**

```cpp
#include <iostream>
#include <type_traits>

template <typename T>
void describe(T value) {
    if constexpr (std::is_integral_v<T>) {
        std::cout << "integer: " << value << '\n';
    } else if constexpr (std::is_floating_point_v<T>) {
        std::cout << "floating: " << value << '\n';
    } else {
        std::cout << "other\n";
    }
}

int main() {
    describe(42);
    describe(3.14);
    describe("hello");
}
```

**Expected Output**
```
integer: 42
floating: 3.14
other
```

**Why this output?**  
`if constexpr` selects the branch based on the type, discarding the others.

### Real-World Cases

- **Type traits** – `if constexpr (std::is_integral_v<T>)`.
- **Generic algorithms** – selecting implementations per type.
- **Serialization** – different handling for different types.
- **Compile-time branching** – replacing SFINAE.

---

## 8. `if consteval` (C++23)

### Definitions

**Core Definition**
`if consteval` is a C++23 conditional that branches on whether the current evaluation is happening in a **constant expression** (compile-time) context.

**Technical Definition**
`if consteval` (also usable as `if !consteval`) evaluates to true when the enclosing function is being evaluated in a constant expression context, and false otherwise. It is used inside `constexpr` functions to provide different implementations for compile-time and runtime evaluation. It requires the enclosing function to be `constexpr` (or `consteval`).

**Beginner-Friendly Explanation**
`if consteval` lets a `constexpr` function **check whether it's running at compile time or runtime** and behave differently. It's useful for optimizing compile-time evaluation or avoiding runtime-only operations.

### Purposes

- To distinguish compile-time from runtime evaluation in `constexpr` functions.
- To optimize compile-time evaluation.
- To avoid runtime-only operations in constant evaluation.
- To implement dual-mode `constexpr` functions.

### Syntax Rules and Structure

**General Syntax**
```cpp
constexpr T f() {
    if consteval {
        // compile-time path
    } else {
        // runtime path
    }
}
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `if consteval` | True in constant evaluation. |
| `else` | Runtime path. |

**Syntax Rules**
- Requires C++23.
- The enclosing function must be `constexpr` or `consteval`.
- `if !consteval` is the negation form.
- It does not require a condition.

**Constraints and Limitations**
- Requires C++23.
- The function must be `constexpr`/`consteval`.
- The branches must be syntactically valid.

### Annotated Code Examples

**Example 1: `if consteval`**

```cpp
#include <iostream>

constexpr int compute(int n) {
    if consteval {
        return n * 2;   // compile-time path
    } else {
        return n * 3;   // runtime path
    }
}

int main() {
    constexpr int ct = compute(10);   // compile-time: 20
    int rt = compute(10);             // runtime: 30
    std::cout << ct << ' ' << rt << '\n';
}
```

**Expected Output**
```
20 30
```

**Why this output?**  
`compute(10)` in a constant expression returns `20`; at runtime, it returns `30`.

### Real-World Cases

- **Compile-time optimization** – different algorithms for compile-time.
- **Avoiding runtime-only operations** – no I/O in constant evaluation.
- **Dual-mode functions** – `constexpr` functions with optimized paths.

---

## 9. Conditional Best Practices

### Definitions

**Core Definition**
Conditional best practices are guidelines for writing clear, correct, and maintainable conditional statements.

**Technical Definition**
These practices are based on the C++ Core Guidelines (especially **ES.6** on scoping, **ES.7** on naming, **ES.8** on avoiding confusing conditions, **ES.70** on `switch` vs. `if`, and **C.168** on comparison consistency) and community standards. Key principles: use braces for all branches; avoid dangling `else`; prefer early returns to reduce nesting; use `if constexpr` for compile-time branching; avoid side effects in conditions; keep conditions simple; and choose `switch` for multi-way integer comparisons.

**Beginner-Friendly Explanation**
Best practices for conditionals are simple: **use braces**, **avoid deep nesting**, **use early returns**, **keep conditions simple**, **don't put side effects in conditions**, and **use `if constexpr` for compile-time choices**.

### Key Guidelines (Summary Table)

| Guideline | Summary | Recommendation |
|-----------|---------|----------------|
| **Use braces** | Always use braces for `if`/`else` bodies. | Avoid dangling `else`. |
| **Early returns** | Return early to reduce nesting. | Flatten logic. |
| **Simple conditions** | Keep conditions readable. | Use named booleans. |
| **No side effects** | Avoid side effects in conditions. | Evaluate first, then test. |
| **`if constexpr`** | Use for compile-time branching. | Templates and type-dependent code. |
| **`switch` for integers** | Use `switch` for multi-way integer comparisons. | Clearer than `else if` chains. |
| **`[[likely]]`/`[[unlikely]]`** | (C++20) Hint branch prediction. | Performance-critical code. |
| **Consistent comparisons** | Use `==` consistently. | Avoid mixing `==` and `!=` confusingly. |

### Annotated Code Examples

**Example 1: Braces and early returns**

```cpp
#include <iostream>

void process(int x) {
    if (x < 0) {
        std::cout << "negative\n";
        return;   // early return
    }
    if (x == 0) {
        std::cout << "zero\n";
        return;
    }
    std::cout << "positive\n";
}

int main() {
    process(-5);
    process(0);
    process(5);
}
```

**Expected Output**
```
negative
zero
positive
```

**Why this is best practice?**  
Early returns flatten the nesting, making the code easier to read. Braces ensure clarity.

### Real-World Cases

- **Code review** – conditionals are a common review focus.
- **Static analysis** – clang-tidy checks for dangling `else`, missing braces, and side effects.
- **Performance** – `[[likely]]`/`[[unlikely]]` hints.

---

## References

- `if` statement (cppreference) – https://en.cppreference.com/w/cpp/language/if
- `if constexpr` (cppreference) – https://en.cppreference.com/w/cpp/language/if#Constexpr_if
- `if consteval` (cppreference) – https://en.cppreference.com/w/cpp/language/if#Consteval_if
- Conditional operator (cppreference) – https://en.cppreference.com/w/cpp/language/operator_other#Conditional_operator
- `switch` statement (cppreference) – https://en.cppreference.com/w/cpp/language/switch
- C++ Core Guidelines — ES.6: Declare names in for-statement initializers and conditions to limit scope – https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Res-scope
- C++ Core Guidelines — ES.7: Keep common and local names short, and keep uncommon and nonlocal names longer – https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Res-name-length
- C++ Core Guidelines — ES.8: Avoid confusingly similar names – https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Res-name-similar
- C++ Core Guidelines — ES.70: Prefer a `switch`-statement to an `if`-statement when there is a choice – https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Res-switch-if
- Microsoft Learn — `if` statement (C++) – https://learn.microsoft.com/en-us/cpp/cpp/if-else-statement-cpp
- Microsoft Learn — `if constexpr` – https://learn.microsoft.com/en-us/cpp/cpp/if-else-statement-cpp#constexpr-if-statements
- Microsoft Learn — Conditional operator – https://learn.microsoft.com/en-us/cpp/cpp/conditional-operator-q

---

This cheat sheet provides a complete, structured reference for C++ conditional statements, covering `if`, `else if`, `else`, nested conditionals, the ternary operator, `if` with initializer (C++17), `if constexpr` (C++17), `if consteval` (C++23), and best practices. Each section follows a uniform template with definitions, purposes, syntax rules, annotated examples, expected outputs, and authoritative references.