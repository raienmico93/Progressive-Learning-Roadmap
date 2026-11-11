# Constexpr — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
`constexpr` is a C++ specifier (introduced in C++11) that declares that the value of a variable or function **can be evaluated at compile time**, allowing that entity to appear in constant expressions where only compile-time constants are permitted.

**Technical Definition**
The `constexpr` specifier, when applied to a variable, requires the variable to be of a literal type, be initialized, and have its initialization be a constant expression; a `constexpr` variable is implicitly `const`. When applied to a function, it declares that the function is *constexpr-suitable* — its parameters and return type must be literal types, its body must satisfy the syntactic restrictions of the standard, and at least one set of arguments must produce a constant expression for the function to be usable in constant contexts. A `constexpr` specifier used in the first declaration of a function or static data member (since C++17) implies `inline`. Since C++20, `consteval` enforces compile-time evaluation, and `constinit` asserts static initialization without implying `const`.

**Beginner-Friendly Explanation**
`constexpr` is a promise to the compiler: “This value, or this function's result, can be computed **while the program is being compiled**, not while it is running.” For variables, this means the value must be known at compile time (unlike `const`, which only means “cannot be changed”). For functions, it means the function is **capable** of running at compile time when given compile-time-known inputs, but it can also run normally at runtime. The benefit is performance: work is moved from runtime to compile time, and the results can be used in contexts that demand compile-time constants (array sizes, template arguments, `static_assert`).

### Key Characteristics

- **Two contexts**: Variables (`constexpr int x = 5;`) and functions (`constexpr int f(int);`).
- **Implicit `const`** : A `constexpr` variable is implicitly `const` (and must be initialized at its declaration).
- **Implicit `inline`** : A `constexpr` function or static data member (since C++17) is implicitly `inline`, so it can be defined in headers.
- **Relaxed rules over time**: C++11 `constexpr` functions were restricted to a single `return` statement; C++14 allowed loops, local variables, and multiple statements; C++20/23 relaxed further.
- **Compile-time *and* runtime**: A `constexpr` function is not forced to run at compile time; it can be called with runtime arguments.
- **Enforced compile-time**: `consteval` (C++20) *forces* every call to be a constant expression.
- **Static-init assertion**: `constinit` (C++20) guarantees a variable is initialized at compile time (static initialization) but does **not** make it `const`.

### Prerequisites

- Understanding of `const`, `inline`, and the One Definition Rule.
- Familiarity with literal types (types that can be used in `constexpr` contexts: scalars, references, and certain class types with `constexpr` constructors/destructors).
- Basic knowledge of `static_assert` and template metaprogramming.
- Awareness of the difference between static initialization and dynamic initialization.

### Related Programming Areas

- **Compile-time computation** – template metaprogramming, `if constexpr`, variable templates.
- **`static_assert`** – compile-time assertions using `constexpr` conditions.
- **Literal types** – the type category required for `constexpr` variables and parameters/returns.
- **`consteval` and `constinit`** – C++20 refinements of compile-time evaluation.
- **`inline`** – `constexpr` functions and variables are implicitly inline.
- **ABI and performance** – compile-time evaluation can eliminate runtime cost.

### Core Concepts / Features

1. `constexpr`
2. Compile-Time Evaluation
3. Constant Expressions
4. `constexpr` Functions
5. `constexpr` Variables
6. `constexpr` Constructors
7. `consteval` (C++20)
8. `constinit` (C++20)
9. Compile-Time Computation
10. `static_assert`
11. Constexpr Best Practices

---

## 1. `constexpr`

### Definitions

**Core Definition**
`constexpr` is a specifier that indicates an entity (variable or function) **can be evaluated at compile time** and therefore can appear in constant expressions.

**Technical Definition**
When applied to a variable, `constexpr` requires the variable to be a definition, be of a literal type, be initialized, and have its full-expression be a constant expression; the variable is implicitly `const` (until C++14 for non-static member functions, the same implication applies). When applied to a function, `constexpr` declares the function to be *constexpr-suitable*: its return type and parameter types must be literal types (until C++23), its body must not contain `goto`, labels (other than `case`/`default`), `try` blocks, inline assembly, or definitions of variables of non-literal type or static/thread storage duration (until C++20/23), and the function must not be virtual (until C++20) or a coroutine (since C++20).

**Beginner-Friendly Explanation**
`constexpr` is the keyword that unlocks **compile-time programming**. Write it before a variable to say “this value is computed during compilation.” Write it before a function to say “this function can run during compilation if the inputs are known at compile time.” It is stricter than `const` for variables (must be compile-time initialized) and more permissive than `consteval` for functions (compile-time evaluation is optional, not forced).

### Purposes

- To allow values to be computed at compile time, eliminating runtime cost.
- To enable the use of those values in contexts that require compile-time constants (array sizes, template arguments, `static_assert`).
- To make functions usable in both compile-time and runtime contexts.
- To enable `inline` definitions in headers (since `constexpr` implies `inline` for functions).

### Syntax Rules and Structure

**General Syntax**
```cpp
constexpr type variable_name = constant_expression;   // variable
constexpr return_type function_name(parameters) { ... } // function (C++11 style)
constexpr auto function_name(parameters) { ... }       // deduced return type (C++14)
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `constexpr` | Specifier. |
| `type` | Literal type (for variables). |
| `= constant_expression` | Required initializer for variables. |
| Function body | Subject to syntactic restrictions (relaxed across standards). |

**Syntax Rules**
- A `constexpr` variable must be a definition, be of a literal type, be initialized, and have a constant-expression initializer.
- A `constexpr` function must not be virtual (until C++20), must have literal parameter and return types (until C++23), and must satisfy body restrictions.
- A `constexpr` function is implicitly `inline` (since C++17 for functions; implied for all `constexpr` functions).
- Every declaration of a `constexpr` function must contain `constexpr`.
- Since C++23, `constexpr` functions may contain non-literal local variables and `goto`/labels under certain conditions.

**Constraints and Limitations**
- `constexpr` variables cannot be of non-literal types (e.g., `std::vector`, `std::string` at namespace scope) unless in limited C++20/23 contexts.
- `constexpr` functions cannot unconditionally call non-`constexpr` functions; if no argument set yields a constant expression, the program is ill-formed, no diagnostic required (until C++23).
- `constexpr` does not guarantee compile-time evaluation for functions; the compiler may choose runtime if arguments are not constant.

### Annotated Code Examples

**Example 1: `constexpr` variable**

```cpp
#include <iostream>

constexpr int answer = 42;          // must be initialized with a constant expression

int main() {
    int arr[answer];                // OK: answer is a constant expression
    std::cout << sizeof(arr) / sizeof(arr[0]) << '\n';
}
```

**Expected Output**
```
42
```

**Why this output?**  
`answer` is a `constexpr` variable, so it can be used as an array bound. `const` alone would not suffice if `answer` were initialized at runtime.

**Example 2: `constexpr` function usable at compile time and runtime**

```cpp
#include <iostream>

constexpr int square(int x) {
    return x * x;
}

int main() {
    constexpr int compile_time = square(5);   // evaluated at compile time
    int runtime_input = 6;
    int runtime_result = square(runtime_input); // may run at runtime
    std::cout << compile_time << ' ' << runtime_result << '\n';
}
```

**Expected Output**
```
25 36
```

**Why this output?**  
`square(5)` is a constant expression, so `compile_time` is initialized at compile time. `square(runtime_input)` is a runtime call. The same function serves both purposes.

### Real-World Cases

- **Array sizes and template arguments** – `constexpr` variables and functions provide compile-time constants.
- **Mathematical utilities** – `square`, `factorial`, `fibonacci` implemented as `constexpr`.
- **Configuration constants** – `constexpr double pi = 3.14159...;`.
- **Header-only libraries** – `constexpr` functions are implicitly `inline`.

---

## 2. Compile-Time Evaluation

### Definitions

**Core Definition**
Compile-time evaluation is the process by which the compiler computes the value of a constant expression during translation, rather than deferring the computation to runtime.

**Technical Definition**
Compile-time evaluation occurs when a *constant expression* is required (array bound, template argument, `static_assert`, `constexpr` variable initializer) or when a `consteval` function is called. The compiler performs the computation following the rules of constant expressions; if the evaluation would invoke undefined behavior (or violate other constraints), the program is ill-formed (a diagnostic is required for `constexpr`/`consteval` contexts).

**Beginner-Friendly Explanation**
Compile-time evaluation means **doing the math before the program runs**. If you write `constexpr int x = 5 * 5;`, the compiler computes `25` and bakes it into the program. No runtime multiplication occurs. This saves time and enables the result to be used where only compile-time constants are allowed.

### Purposes

- To eliminate runtime computation for values known at compile time.
- To enable the use of results in compile-time-only contexts.
- To catch errors (e.g., overflow, invalid operations) at compile time.
- To reduce binary size and startup time by precomputing constants.

### Syntax Rules and Structure

Compile-time evaluation is triggered by **contexts that require a constant expression**:

| Context | Example |
|---------|---------|
| `constexpr` variable initializer | `constexpr int x = f(5);` |
| Array bound | `int arr[f(5)];` |
| Template argument | `std::array<int, f(5)> a;` |
| `static_assert` condition | `static_assert(f(5) == 25);` |
| `consteval` function call | `consteval int g() { return 1; }` |
| `case` label in `switch` | `case f(3):` |

**Syntax Rules**
- The expression must be a *constant expression* according to the standard’s rules.
- `constexpr` functions are evaluated at compile time **only when required** by context; otherwise they may run at runtime.
- `consteval` functions **must** be evaluated at compile time; a runtime call is ill-formed.

**Constraints and Limitations**
- Not all operations are permitted in constant expressions (e.g., `reinterpret_cast`, calls to non-`constexpr` functions, undefined behavior).
- Compile-time evaluation can hit compiler limits (e.g., recursion depth, step count).
- Large computations may slow compilation.

### Annotated Code Examples

**Example 1: Compile-time vs. runtime evaluation**

```cpp
#include <iostream>

constexpr int add(int a, int b) { return a + b; }

int main() {
    constexpr int ct = add(2, 3);       // compile-time evaluation required
    int x = 4, y = 5;
    int rt = add(x, y);                 // runtime evaluation (may or may not be constant-folded)
    std::cout << ct << ' ' << rt << '\n';
}
```

**Expected Output**
```
5 9
```

**Why this output?**  
The initializer of `ct` requires a constant expression, so `add(2, 3)` is evaluated at compile time. `add(x, y)` uses runtime variables, so the call is a runtime evaluation (though the optimizer may constant-fold it if it can prove the values).

**Example 2: `consteval` forces compile-time evaluation**

```cpp
consteval int square(int n) { return n * n; }

constexpr int a = square(4);   // OK: compile-time
int main() {
    int x = 4;
    // int b = square(x);        // error: x is not a constant expression
}
```

**Expected Output**  
(Compiles; the commented call would be an error.)

**Why this works?**  
`consteval` requires every call to produce a constant expression. `square(4)` is valid; `square(x)` with a runtime variable is rejected at compile time.

### Real-World Cases

- **Metaprogramming** – computing type traits and sizes at compile time.
- **Embedded systems** – precomputing lookup tables.
- **High-performance computing** – eliminating runtime overhead for constants.

---

## 3. Constant Expressions

### Definitions

**Core Definition**
A constant expression is an expression that can be **evaluated at compile time** by the compiler and whose value is a compile-time constant.

**Technical Definition**
A constant expression is either a *glvalue core constant expression* that refers to a permitted entity, or a *prvalue core constant expression* whose value satisfies additional constraints (e.g., each reference member refers to a permitted entity, no indeterminate values, no pointer to a non-`constexpr` object). A *core constant expression* is an expression whose evaluation does not invoke certain disallowed operations (calls to non-`constexpr` functions, undefined behavior, `reinterpret_cast`, etc.).

**Beginner-Friendly Explanation**
A constant expression is any expression the compiler can fully compute **right now**, during compilation. Examples include `42`, `2 + 3`, `sizeof(int)`, and calls to `constexpr` functions with constant arguments. You cannot use runtime variables, call non-`constexpr` functions, or do things that are not allowed in compile-time computation.

### Purposes

- To define what can be used in compile-time contexts.
- To enable `constexpr` variables, template arguments, and `static_assert`.
- To provide a formal basis for compile-time evaluation.
- To allow the compiler to validate computations at compile time.

### Syntax Rules and Structure

Constant expressions are not written with special syntax; they are recognized by context.

**Examples of Constant Expressions**
```cpp
42                      // literal
2 + 3                   // arithmetic on literals
sizeof(int)             // sizeof expression
constexpr int x = 5;
x * 2                   // arithmetic on constexpr variable
square(5)               // call to constexpr function with constant args
```

**Examples of Non-Constant Expressions**
```cpp
int y = 10;
y * 2                   // y is not constexpr
std::rand()             // non-constexpr function
reinterpret_cast<int*>(0x1000)  // disallowed in constant expressions
```

**Syntax Rules**
- Literals of integral and enumeration types are constant expressions.
- `constexpr` variables are constant expressions.
- Calls to `constexpr` functions with constant arguments are constant expressions.
- `sizeof`, `alignof`, and `noexcept` expressions are constant expressions.
- Operations that would cause undefined behavior in the evaluation make the expression non-constant (and ill-formed in constant contexts).

**Constraints and Limitations**
- Floating-point literals are not allowed in *integral constant expressions* unless explicitly cast (pre-C++23 rules).
- `reinterpret_cast`, `new`/`delete` (before C++20, and with restrictions after), and calls to non-`constexpr` functions are disallowed.
- The evaluation must not read an indeterminate value.

### Annotated Code Examples

**Example 1: Constant expression in `static_assert`**

```cpp
#include <iostream>

constexpr int factorial(int n) {
    return (n <= 1) ? 1 : n * factorial(n - 1);
}

static_assert(factorial(5) == 120, "factorial(5) must be 120");

int main() {
    std::cout << factorial(5) << '\n';
}
```

**Expected Output**
```
120
```

**Why this works?**  
`factorial(5)` is a constant expression because `factorial` is `constexpr` and `5` is a constant. `static_assert` requires a constant expression; the condition is verified at compile time.

**Example 2: Non-constant expression rejected**

```cpp
int get_value() { return 42; }   // not constexpr

// constexpr int x = get_value();   // error: get_value() is not a constant expression
```

**Expected Output**  
(Compilation error.)

**Why this error?**  
`get_value()` is not a `constexpr` function, so its result is not a constant expression and cannot initialize a `constexpr` variable.

### Real-World Cases

- **Template arguments** – non-type template parameters must be constant expressions.
- **Array bounds** – the size must be a constant expression.
- **`static_assert`** – the condition must be a constant expression.
- **`case` labels** – must be constant expressions.

---

## 4. `constexpr` Functions

### Definitions

**Core Definition**
A `constexpr` function is a function that is **capable of being evaluated at compile time** when called with constant arguments, while also being callable at runtime.

**Technical Definition**
A function or function template declared `constexpr` is *constexpr-suitable* if it is not virtual (until C++20), its return type and parameter types are literal types (until C++23), it is not a coroutine (since C++20), and its body satisfies the syntactic restrictions of the applicable standard (which have been progressively relaxed from C++11 to C++23). A `constexpr` function is implicitly `inline`. If no argument values exist such that an invocation could be a constant expression, the program is ill-formed, no diagnostic required (until C++23).

**Beginner-Friendly Explanation**
A `constexpr` function is a **dual-mode** function: if you give it compile-time-known inputs and use it where a compile-time result is required, it runs at compile time. If you give it runtime inputs, it runs at runtime like a normal function. It is not *forced* to be compile-time (that is `consteval`), but it *can* be.

### Purposes

- To enable the same function to be used in compile-time and runtime contexts.
- To allow compile-time computation of results for `constexpr` variables, template arguments, and `static_assert`.
- To make header-defined functions implicitly `inline`.
- To support generic programming and metaprogramming.

### Syntax Rules and Structure

**General Syntax**
```cpp
constexpr return_type function_name(parameters) {
    // body satisfying constexpr restrictions
}
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `constexpr` | Specifier. |
| `return_type` | Literal type (until C++23). |
| `parameters` | Literal types (until C++23). |
| Body | No `goto`/labels (until C++20), no `try` (until C++20), no static/thread locals (until C++23), etc. |

**Syntax Rules**
- Every declaration must include `constexpr` if any does.
- Implicitly `inline` (so safe in headers).
- C++14 relaxed the “single return statement” rule, allowing loops, local variables, and multiple statements.
- C++20 relaxed further (virtual functions allowed, `try` blocks allowed, etc.).
- C++23 relaxed most remaining restrictions.

**Constraints and Limitations**
- Before C++23, at least one set of arguments must produce a constant expression, or the program is ill-formed (NDR).
- `constexpr` functions cannot unconditionally call non-`constexpr` functions in constant evaluation.
- Recursion depth and step count are compiler-implementation limits.

### Annotated Code Examples

**Example 1: C++11-style `constexpr` function (single return)**

```cpp
constexpr int factorial(int n) {
    return (n <= 1) ? 1 : n * factorial(n - 1);
}
```

**Expected Output**  
(Compile-time result usable in constant expressions.)

**Why this style?**  
C++11 required the body to contain exactly one `return` statement (plus `static_assert`, `typedef`, `using`). Recursion was the only way to loop.

**Example 2: C++14-style `constexpr` function (loops and locals)**

```cpp
#include <iostream>

constexpr int factorial(int n) {
    int result = 1;
    for (int i = 2; i <= n; ++i) {
        result *= i;
    }
    return result;
}

int main() {
    constexpr int f5 = factorial(5);
    std::cout << f5 << '\n';
}
```

**Expected Output**
```
120
```

**Why this works?**  
C++14 allows loops, local variables, and multiple statements in `constexpr` functions. `factorial(5)` is evaluated at compile time because it initializes a `constexpr` variable.

### Real-World Cases

- **Standard library** – `std::array`, `std::span`, and many algorithms are `constexpr` in modern C++.
- **Mathematical utilities** – `gcd`, `lcm`, `power` implemented as `constexpr`.
- **Compile-time strings** – fixed-size string types with `constexpr` operations.
- **Type traits** – many traits are implemented with `constexpr` functions.

---

## 5. `constexpr` Variables

### Definitions

**Core Definition**
A `constexpr` variable is a variable whose value is **fixed at compile time** and can be used in constant expressions.

**Technical Definition**
A variable is `constexpr` if its declaration is a definition, it is of a literal type, it is initialized, and the full-expression of its initialization is a constant expression; it is implicitly `const`. Since C++20, `constexpr` variables must also have constant destruction (a `constexpr` destructor or no class type).

**Beginner-Friendly Explanation**
A `constexpr` variable is a **compile-time constant**. Unlike `const`, which can be initialized at runtime, `constexpr` requires the value to be known during compilation. This means it can be used as an array size, a template argument, or in `static_assert`.

### Purposes

- To provide compile-time constants for use in constant expressions.
- To replace `#define` macros for constants with type-safe, scoped alternatives.
- To enable compile-time configuration and lookup tables.
- To guarantee that a value is available at compile time (unlike `const`).

### Syntax Rules and Structure

**General Syntax**
```cpp
constexpr type variable_name = constant_expression;
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `constexpr` | Specifier. |
| `type` | Literal type. |
| `= constant_expression` | Required; must be a constant expression. |

**Syntax Rules**
- Must be a definition (not just a declaration).
- Must be of a literal type.
- Must be initialized with a constant expression.
- Implicitly `const`.
- Since C++20, must have constant destruction (for class types).
- At namespace scope, `constexpr` variables are implicitly `inline` (since C++17) if they are `static` data members; otherwise they have internal linkage by default.

**Constraints and Limitations**
- Non-literal types (e.g., `std::vector`, `std::string` in C++17) cannot be `constexpr` at namespace scope (C++20 relaxed this for some contexts).
- The initializer must be a constant expression; runtime values are not allowed.

### Annotated Code Examples

**Example 1: `constexpr` variable vs. `const` variable**

```cpp
#include <iostream>

int runtime_value() { return 10; }

const int a = runtime_value();     // OK: initialized at runtime
// constexpr int b = runtime_value(); // error: not a constant expression

constexpr int c = 20;              // OK: compile-time constant

int main() {
    int arr1[a];   // error: a is not a constant expression (VLA not standard C++)
    int arr2[c];   // OK: c is a constant expression
    std::cout << sizeof(arr2) / sizeof(arr2[0]) << '\n';
}
```

**Expected Output**  
(Compilation error for `arr1`; `arr2` is valid.)

**Why this matters?**  
`const` only guarantees immutability, not compile-time initialization. `constexpr` guarantees the value is a constant expression, so it can be used as an array bound.

**Example 2: `constexpr` static data member**

```cpp
struct Constants {
    static constexpr int max_size = 1024;   // implicitly inline (C++17)
};

int main() {
    int buffer[Constants::max_size];        // OK
}
```

**Expected Output**  
(Compiles; `buffer` has 1024 elements.)

**Why this works?**  
`max_size` is a `constexpr` static data member, implicitly `inline` since C++17, and usable as a constant expression.

### Real-World Cases

- **Array sizes** – `constexpr std::size_t buffer_size = 1024;`.
- **Template arguments** – `std::array<int, constexpr_value>`.
- **Mathematical constants** – `constexpr double pi = 3.141592653589793;`.
- **Bit masks and flags** – `constexpr unsigned flag = 1u << 3;`.

---

## 6. `constexpr` Constructors

### Definitions

**Core Definition**
A `constexpr` constructor is a constructor that can be used to initialize objects in constant expressions, making the object's construction a compile-time operation.

**Technical Definition**
On top of the requirements for `constexpr` functions, a constructor must satisfy additional conditions to be *constexpr-suitable*: every non-variant non-static data member and base class subobject must be initialized (until C++20); if the constructor is delegating, the target constructor must be `constexpr`; every constructor selected to initialize non-static data members and base classes must be `constexpr`; and the class must not have virtual base classes (until C++26). A class with a `constexpr` constructor becomes a literal type (if other conditions are met), enabling `constexpr` instances.

**Beginner-Friendly Explanation**
A `constexpr` constructor lets you create objects **during compilation**. For example, `constexpr Point p{1, 2};` constructs `p` at compile time if `Point` has a `constexpr` constructor. This is essential for making user-defined types usable in constant expressions.

### Purposes

- To enable user-defined types to be used in constant expressions.
- To allow compile-time construction of objects (e.g., lookup tables, configuration objects).
- To make classes literal types.
- To support `constexpr` variables of class type.

### Syntax Rules and Structure

**General Syntax**
```cpp
struct S {
    constexpr S(parameters) : member_initializers { /* body */ }
};
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `constexpr` | Specifier on the constructor. |
| Member initializer list | Required to initialize all non-variant non-static members (until C++20). |
| Body | Must satisfy `constexpr` function body restrictions. |

**Syntax Rules**
- Every non-variant non-static data member and base class subobject must be initialized (until C++20).
- Delegating constructors must target a `constexpr` constructor.
- Selected constructors for members/bases must be `constexpr`.
- Class must not have virtual base classes (until C++26).
- The constructor must satisfy the general `constexpr` function requirements.

**Constraints and Limitations**
- A `constexpr` constructor cannot leave members uninitialized (until C++20).
- Virtual base classes prevent `constexpr` construction (until C++26).
- The class must be a literal type for `constexpr` instances to be valid.

### Annotated Code Examples

**Example 1: `constexpr` constructor for a literal type**

```cpp
#include <iostream>

struct Point {
    int x, y;
    constexpr Point(int x_, int y_) : x(x_), y(y_) {}
};

int main() {
    constexpr Point p{3, 4};   // compile-time construction
    static_assert(p.x == 3 && p.y == 4, "coordinates must match");
    std::cout << p.x << ',' << p.y << '\n';
}
```

**Expected Output**
```
3,4
```

**Why this works?**  
`Point` has a `constexpr` constructor that initializes both members. `p` is constructed at compile time, and `static_assert` verifies its members at compile time.

**Example 2: `constexpr` constructor with defaulted body**

```cpp
struct Widget {
    int value;
    constexpr Widget() = default;   // constexpr default constructor
};

constexpr Widget w{};   // OK: w.value is zero-initialized
```

**Expected Output**  
(Compiles; `w.value == 0` is a constant expression.)

**Why this works?**  
A defaulted constructor can be `constexpr` if the implicit initialization satisfies the requirements. `w` is constructed at compile time with `value` zero-initialized.

### Real-World Cases

- **`std::array`** – `constexpr` aggregate initialization.
- **`std::pair` and `std::tuple`** – `constexpr` constructors.
- **Compile-time geometry** – `Point`, `Vector`, `Matrix` types with `constexpr` constructors.
- **Configuration objects** – compile-time settings used in templates.

---

## 7. `consteval` (C++20)

### Definitions

**Core Definition**
`consteval` is a C++20 specifier that declares a function to be an **immediate function**: every call to it must produce a compile-time constant.

**Technical Definition**
The `consteval` specifier declares a function or function template to be an immediate function. Every potentially-evaluated call to an immediate function must (directly or indirectly) produce a constant expression. An immediate function is a `constexpr` function subject to its requirements; `consteval` implies `inline`. It may not be applied to destructors, allocation functions, or deallocation functions. A declaration specifying `consteval` may not also specify `constexpr`, and all redeclarations must specify `consteval`.

**Beginner-Friendly Explanation**
`consteval` is **strict `constexpr`** . A `constexpr` function *can* run at compile time; a `consteval` function **must** run at compile time. You cannot call it with runtime arguments. It is for computations that are meaningless or unsafe at runtime, and it guarantees zero runtime cost.

### Purposes

- To enforce compile-time evaluation, guaranteeing no runtime call.
- To express that a computation is only meaningful at compile time.
- To catch errors (e.g., invalid inputs) at compile time.
- To support compile-time-only APIs (e.g., compile-time parsers, validators).

### Syntax Rules and Structure

**General Syntax**
```cpp
consteval return_type function_name(parameters) { ... }
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `consteval` | Specifier for immediate functions. |
| Function | Must satisfy `constexpr` requirements. |

**Syntax Rules**
- Every call must produce a constant expression.
- Implies `inline`; cannot be combined with `constexpr`.
- Cannot be applied to destructors, `operator new`, or `operator delete`.
- All redeclarations must use `consteval`.
- Pointers/references to immediate functions cannot escape constant evaluation.

**Constraints and Limitations**
- Runtime arguments are rejected at compile time.
- `consteval` cannot propagate through a non-`consteval` function (until C++23 defect resolution; some propagation rules apply).
- Cannot be used for virtual functions.

### Annotated Code Examples

**Example 1: `consteval` function**

```cpp
#include <iostream>

consteval int square(int n) { return n * n; }

int main() {
    constexpr int a = square(5);   // OK: compile-time
    std::cout << a << '\n';
    int x = 5;
    // int b = square(x);         // error: x is not a constant expression
}
```

**Expected Output**
```
25
```

**Why this works?**  
`square(5)` is a constant expression. The commented call with a runtime variable would be a compile-time error because `consteval` requires every call to be constant.

**Example 2: `consteval` with `static_assert`**

```cpp
#include <iostream>

consteval int factorial(int n) {
    int result = 1;
    for (int i = 2; i <= n; ++i) result *= i;
    return result;
}

static_assert(factorial(5) == 120);

int main() {
    std::cout << factorial(5) << '\n';
}
```

**Expected Output**
```
120
```

**Why this works?**  
`factorial` is `consteval`, so every call — including the one in `static_assert` and the one in `main` — must be a constant expression. The compiler evaluates both at compile time.

### Real-World Cases

- **Compile-time validation** – functions that reject invalid inputs at compile time.
- **Compile-time parsers** – parsing string literals during compilation.
- **Metaprogramming utilities** – ensuring computations are always compile-time.
- **Embedded systems** – guaranteeing no runtime cost.

---

## 8. `constinit` (C++20)

### Definitions

**Core Definition**
`constinit` is a C++20 specifier that asserts a variable has **static initialization** (zero initialization and constant initialization), making the program ill-formed otherwise.

**Technical Definition**
The `constinit` specifier declares a variable with static or thread storage duration. If the variable has dynamic initialization (even if performed as static initialization), the program is ill-formed. `constinit` cannot be used with `constexpr`. When applied to a reference, `constinit` is equivalent to `constexpr`. When applied to an object, `constexpr` mandates static initialization and constant destruction and makes the object `const`-qualified; `constinit` does **not** mandate constant destruction or `const`-qualification, so a mutable object with a non-`constexpr` destructor can be `constinit` but not `constexpr`.

**Beginner-Friendly Explanation**
`constinit` guarantees that a global or thread-local variable is **initialized at compile time**, avoiding the “static initialization order fiasco.” Unlike `constexpr`, it does **not** make the variable `const` — you can still modify it at runtime. It is a way to say “this variable must be ready before any dynamic initialization runs.”

### Purposes

- To guarantee static initialization for global/thread-local variables.
- To avoid dynamic initialization and the static initialization order problem.
- To allow mutable variables with compile-time initialization.
- To reduce startup overhead and guard-variable overhead for `thread_local`.

### Syntax Rules and Structure

**General Syntax**
```cpp
constinit type variable_name = constant_expression;
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `constinit` | Specifier. |
| Variable | Must have static or thread storage duration. |
| Initializer | Must be a constant expression (static initialization). |

**Syntax Rules**
- Only variables with static or thread storage duration.
- The initializing declaration must be reachable and use `constinit`.
- Dynamic initialization (even early dynamic initialization) makes the program ill-formed.
- Cannot be combined with `constexpr`.
- For references, `constinit` is equivalent to `constexpr`.
- Does not imply `const`; the variable can be mutated at runtime.

**Constraints and Limitations**
- Types without `constexpr` constructors cannot be `constinit` (unless they have constant initialization via zero-initialization or aggregate initialization).
- `constinit` does not guarantee constant destruction.

### Annotated Code Examples

**Example 1: `constinit` global variable**

```cpp
#include <iostream>

constexpr int square(int n) { return n * n; }

constinit int global_square = square(5);   // static initialization

int main() {
    std::cout << global_square << '\n';
    global_square = 100;                   // OK: constinit does not imply const
    std::cout << global_square << '\n';
}
```

**Expected Output**
```
25
100
```

**Why this works?**  
`global_square` is initialized at compile time via `square(5)` (static initialization), satisfying `constinit`. It remains mutable because `constinit` does not imply `const`.

**Example 2: `constinit` rejects dynamic initialization**

```cpp
int f() { return 42; }   // not constexpr

// constinit int x = f();   // error: f() is not a constant expression
```

**Expected Output**  
(Compilation error.)

**Why this error?**  
`f()` is not `constexpr`, so its result is not a constant expression, and `x` would require dynamic initialization, violating `constinit`.

### Real-World Cases

- **Global configuration** – constants initialized at compile time but mutable.
- **`thread_local` variables** – avoiding guard-variable checks.
- **Embedded systems** – deterministic startup without dynamic initialization.
- **Singletons** – compile-time-initialized global instances.

---

## 9. Compile-Time Computation

### Definitions

**Core Definition**
Compile-time computation is the broader practice of performing calculations during translation using `constexpr`, `consteval`, templates, and `static_assert`, so that results are available before the program runs.

**Technical Definition**
Compile-time computation encompasses constant expression evaluation, template instantiation, `if constexpr`, variable templates, and metaprogramming techniques. It is enabled by `constexpr` (C++11), relaxed `constexpr` function rules (C++14), `if constexpr` (C++17), and `consteval`/`constinit` (C++20).

**Beginner-Friendly Explanation**
Compile-time computation means **solving problems before the program starts**. Examples include computing factorials, generating lookup tables, validating configurations, and selecting code paths with `if constexpr`. It makes programs faster and catches errors earlier.

### Purposes

- To eliminate runtime computation for known values.
- To generate efficient code (e.g., unrolled loops, specialized paths).
- To validate code at compile time (`static_assert`).
- To build compile-time data structures (arrays, strings, maps).

### Techniques (Summary Table)

| Technique | Standard | Use Case |
|-----------|----------|----------|
| `constexpr` variables | C++11 | Compile-time constants. |
| `constexpr` functions | C++11 (relaxed C++14) | Compile-time calculations. |
| `static_assert` | C++11 (message optional C++17) | Compile-time validation. |
| `if constexpr` | C++17 | Compile-time branching in templates. |
| `consteval` | C++20 | Forced compile-time evaluation. |
| `constinit` | C++20 | Guaranteed static initialization. |
| Variable templates | C++14 | Compile-time computed values as templates. |

### Annotated Code Examples

**Example 1: Compile-time lookup table with `constexpr`**

```cpp
#include <iostream>

constexpr int make_table_entry(int i) { return i * i; }

template <std::size_t N>
struct Table {
    int data[N];
    constexpr Table() : data{} {
        for (std::size_t i = 0; i < N; ++i) data[i] = make_table_entry(static_cast<int>(i));
    }
};

constexpr Table<5> table{};

int main() {
    for (int v : table.data) std::cout << v << ' ';
    std::cout << '\n';
}
```

**Expected Output**
```
0 1 4 9 16
```

**Why this works?**  
The `constexpr` constructor of `Table` fills the array at compile time. `table` is a `constexpr` object, so its data is available without runtime initialization.

**Example 2: `if constexpr` with `constexpr` functions**

```cpp
#include <iostream>
#include <type_traits>

template <typename T>
constexpr auto describe(T value) {
    if constexpr (std::is_integral_v<T>) {
        return value + 1;       // integer path
    } else {
        return value;           // non-integer path
    }
}

int main() {
    std::cout << describe(10) << '\n';      // 11
    std::cout << describe(3.14) << '\n';    // 3.14
}
```

**Expected Output**
```
11
3.14
```

**Why this works?**  
`if constexpr` discards the untaken branch at compile time. The `constexpr` function `describe` is instantiated differently for `int` and `double`.

### Real-World Cases

- **Lookup tables** – trigonometric tables, CRC tables, hash tables.
- **Compile-time parsing** – parsing format strings or configuration strings.
- **Type-safe units** – dimensional analysis at compile time.
- **Game development** – precomputed level data and constants.

---

## 10. `static_assert`

### Definitions

**Core Definition**
`static_assert` is a declaration that performs **compile-time assertion checking**; if the condition is false, the compiler issues a diagnostic and the program is ill-formed.

**Technical Definition**
The `static_assert` declaration has the form `static_assert(bool-constexpr, message)` (C++11) or `static_assert(bool-constexpr)` (C++17, message optional). It may appear at namespace scope, block scope, or inside a class body. The condition must be a constant expression contextually converted to `bool`. If the condition is well-formed and evaluates to `true` (or the template is uninstantiated), the declaration has no effect; otherwise, a compile-time error is issued with the message.

**Beginner-Friendly Explanation**
`static_assert` is a **compile-time sanity check**. You give it a condition that must be true at compile time and a message to show if it is not. It is used to catch errors early, document assumptions, and enforce constraints on template parameters.

### Purposes

- To validate compile-time conditions.
- To enforce template parameter requirements.
- To document assumptions about types, sizes, and constants.
- To catch errors at compile time rather than runtime.

### Syntax Rules and Structure

**General Syntax**
```cpp
static_assert(condition, "message");   // C++11
static_assert(condition);              // C++17: message optional
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `condition` | A constant expression convertible to `bool`. |
| `message` | String literal (required until C++17; optional since). |

**Syntax Rules**
- The condition must be a constant expression contextually converted to `bool`.
- May appear at namespace scope, block scope, or in a class body.
- Inside a template, the assertion is checked only upon instantiation (if the condition depends on template parameters).
- Since C++17, the message may be omitted.
- Non-narrowing integral conversions to `bool` are allowed (until C++23).

**Constraints and Limitations**
- The message must be a string literal; it cannot contain runtime information or template type names.
- The condition must be evaluable at compile time; runtime values are not allowed.

### Annotated Code Examples

**Example 1: Basic `static_assert`**

```cpp
#include <iostream>

static_assert(sizeof(int) == 4, "This code requires 32-bit int");

int main() {
    std::cout << "int is 4 bytes\n";
}
```

**Expected Output**
```
int is 4 bytes
```

**Why this works?**  
`sizeof(int) == 4` is a constant expression; if true, the assertion passes and compilation continues. If false, a compile-time error with the message is issued.

**Example 2: `static_assert` in a template**

```cpp
#include <type_traits>

template <typename T>
void process(T value) {
    static_assert(std::is_integral_v<T>, "process requires an integral type");
    // ...
}

int main() {
    process(42);       // OK
    // process(3.14);  // error: static_assert fails
}
```

**Expected Output**  
(Compiles for `int`; the `double` call would trigger the assertion.)

**Why this works?**  
The assertion is checked when the template is instantiated with a specific type. `int` satisfies `is_integral_v`; `double` does not, so the second call would produce a compile-time error with the message.

### Real-World Cases

- **Standard library** – `std::array` uses `static_assert` for size constraints.
- **Type traits** – assertions about type properties in templates.
- **Platform assumptions** – checking `sizeof` or `alignof` values.
- **API contracts** – ensuring template parameters meet requirements.

---

## 11. Constexpr Best Practices

### Definitions

**Core Definition**
Constexpr best practices are guidelines for using `constexpr` effectively: when to apply it, how to avoid pitfalls, and how to balance compile-time computation with readability and compile time.

**Technical Definition**
These practices are informed by the C++ Core Guidelines, particularly **F.4** (“If a function may have to be evaluated at compile time, declare it `constexpr`”), and by community guidance on compile-time programming. Key considerations include the trade-off between compile-time cost and runtime savings, the limitations of literal types, and the interaction between `constexpr` and API evolution.

**Beginner-Friendly Explanation**
Best practices for `constexpr` are simple: **use it for things that genuinely benefit from compile-time evaluation**, **don’t mark everything `constexpr` just because you can**, **keep `constexpr` functions simple**, **prefer `constexpr` variables over macros**, and **be aware that making a function `constexpr` is a long-term commitment** (it must be evaluable at compile time for all future implementations if used in constant contexts).

### Key Guidelines (Summary Table)

| Guideline | Summary | Recommendation |
|-----------|---------|----------------|
| **F.4** | If a function may have to be evaluated at compile time, declare it `constexpr`. | Mark functions `constexpr` when they are used in constant contexts. |
| **F.4 (caution)** | Don’t try to make **all** functions `constexpr`. | Most computation is best done at runtime; over-`constexpr`-ing increases compile time and constrains implementations. |
| **Prefer `constexpr` over macros** | `constexpr` is type-safe and scoped. | Replace `#define` constants with `constexpr` variables. |
| **Keep `constexpr` functions simple** | Complex `constexpr` functions slow compilation and may hit limits. | Use `constexpr` for small, pure computations; leave complex logic at runtime. |
| **Use `static_assert` for validation** | Compile-time checks catch errors early. | Pair `constexpr` with `static_assert` to document and enforce assumptions. |
| **Consider `consteval` when runtime calls are meaningless** | `consteval` enforces compile-time evaluation. | Use `consteval` for compile-time-only computations (parsers, validators). |
| **Use `constinit` for static initialization** | `constinit` avoids dynamic initialization without `const`. | Use for mutable globals/thread-locals that must be statically initialized. |
| **Be mindful of literal type restrictions** | Not all types can be `constexpr`. | Use `std::array` instead of `std::vector` for compile-time sequences. |
| **API evolution risk** | Adding `constexpr` to an existing function is generally safe, but removing it is an ABI break. | Consider `constexpr` as part of the interface contract. |

### Annotated Code Examples

**Example 1: Applying F.4 — declare `constexpr` when compile-time evaluation is needed**

```cpp
#include <iostream>

// GOOD: function used in a constant expression (array size)
constexpr int buffer_size(int n) { return n * 2; }

int main() {
    int data[buffer_size(16)];   // requires constexpr
    std::cout << sizeof(data) / sizeof(data[0]) << '\n';
}
```

**Expected Output**
```
32
```

**Why this is best practice?**  
`buffer_size` is used in a context that requires a constant expression, so declaring it `constexpr` is necessary and appropriate (F.4).

**Example 2: Not everything should be `constexpr`**

```cpp
#include <iostream>
#include <vector>

// AVOID: making a complex, runtime-oriented function constexpr
// constexpr std::vector<int> process(const std::vector<int>& v) { ... } // not possible before C++20, and questionable after

// GOOD: simple, pure function as constexpr
constexpr int square(int x) { return x * x; }

// GOOD: leave I/O-bound or complex logic as runtime functions
void print_vector(const std::vector<int>& v) {
    for (int x : v) std::cout << x << ' ';
    std::cout << '\n';
}

int main() {
    std::vector<int> data = {1, 2, 3};
    print_vector(data);
}
```

**Expected Output**
```
1 2 3
```

**Why this is best practice?**  
`square` is a small, pure computation that benefits from `constexpr`. `print_vector` performs I/O and operates on runtime data; marking it `constexpr` would be meaningless and would constrain its implementation. The Core Guidelines caution against making all functions `constexpr`.

**Example 3: `constexpr` with `static_assert` for compile-time validation**

```cpp
#include <iostream>

constexpr int factorial(int n) {
    return (n <= 1) ? 1 : n * factorial(n - 1);
}

static_assert(factorial(5) == 120, "factorial(5) must be 120");
static_assert(factorial(0) == 1, "factorial(0) must be 1");

int main() {
    std::cout << factorial(5) << '\n';
}
```

**Expected Output**
```
120
```

**Why this is best practice?**  
The `static_assert` declarations validate the `constexpr` function at compile time, documenting expected behavior and catching regressions during compilation.

### Real-World Cases

- **Standard library** – `std::array`, `std::span`, and many algorithms are `constexpr` where meaningful.
- **Embedded systems** – `constexpr` lookup tables and configuration constants.
- **Game development** – compile-time asset hashes and level data.
- **API design** – marking functions `constexpr` when they are pure and usable in constant contexts, following F.4.

---

## References

- `constexpr` specifier (cppreference) – https://en.cppreference.com/w/cpp/language/constexpr
- `consteval` specifier (cppreference) – https://en.cppreference.com/w/cpp/language/consteval
- `constinit` specifier (cppreference) – https://en.cppreference.com/w/cpp/language/constinit
- Constant expressions (cppreference) – https://en.cppreference.com/w/cpp/language/constant_expression
- `static_assert` declaration (cppreference) – https://en.cppreference.com/w/cpp/language/static_assert
- C++ Core Guidelines — F.4: If a function may have to be evaluated at compile time, declare it `constexpr` – https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rf-constexpr
- C++ Core Guidelines — F.5: If a function is very small and time-critical, declare it `inline` – https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rf-inline
- `constexpr` vs `const` (Microsoft Learn) – https://learn.microsoft.com/en-us/cpp/cpp/constexpr-cpp
- Literal type (cppreference) – https://en.cppreference.com/w/cpp/named_req/LiteralType
- `if constexpr` (cppreference) – https://en.cppreference.com/w/cpp/language/if#Constexpr_if

---

This cheat sheet provides a complete, structured reference for C++ `constexpr` and related compile-time features, covering `constexpr` variables, functions, constructors, `consteval`, `constinit`, constant expressions, compile-time computation, `static_assert`, and best practices. Each section follows a uniform template with definitions, purposes, syntax rules, annotated examples, expected outputs, and authoritative references.