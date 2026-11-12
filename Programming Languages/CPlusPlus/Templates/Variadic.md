# Variadic Templates — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
A variadic template is a C++11 feature that allows a template (function, class, variable, or alias) to accept a **variable number of template arguments** (zero or more) of potentially different types, via a construct called a **parameter pack**.

**Technical Definition**
A template with at least one parameter pack is called a variadic template. A template parameter pack is a template parameter that accepts zero or more template arguments (non-types, types, or templates); a function parameter pack is a function parameter that accepts zero or more function arguments. Pack expansion, written as `pattern...`, expands a pattern containing one or more packs into a comma-separated (or space-separated) list. Since C++17, fold expressions reduce a parameter pack over a binary operator in a single expression. Before fold expressions, variadic templates were processed recursively with a base case and a recursive case.

**Beginner-Friendly Explanation**
A variadic template is a **template that doesn't know in advance how many arguments it will receive**. You write one template, and the compiler generates versions for any number of arguments of any types. For example, `std::make_unique<T>(args...)` can forward any number of constructor arguments to `T`. Parameter packs are the “bags” that hold the arguments; pack expansion is how you unpack the bag; fold expressions are a concise way to combine all elements of the bag with an operator. Before C++17, you had to write recursive functions to process packs; since C++17, fold expressions make many tasks a one-liner.

### Key Characteristics

- **Zero or more arguments**: Packs may be empty, one element, or many.
- **Heterogeneous types**: Each element of a pack may have a different type.
- **Compile-time expansion**: Pack expansion is performed by the compiler at instantiation.
- **Two kinds of packs**: Template parameter packs (types/values) and function parameter packs (arguments).
- **Expansion contexts**: Function arguments, template arguments, base class lists, member initializer lists, braced-init-lists, and more.
- **Fold expressions (C++17)**: Reduce a pack with a binary operator without recursion.
- **Recursive processing (pre-C++17)**: Use a base case and a recursive case to peel off arguments.

### Prerequisites

- Understanding of templates (function and class templates).
- Familiarity with template parameters and argument deduction.
- Knowledge of recursion and base cases.
- Basic awareness of perfect forwarding and `std::forward`.

### Related Programming Areas

- **Generic programming** – variadic templates are the foundation of generic utilities.
- **Perfect forwarding** – `std::make_unique<T>(args...)` forwards packs.
- **`std::tuple`** – variadic class template.
- **`std::function`** – variadic function wrapper.
- **Fold expressions** – C++17 reduction of packs.
- **Template metaprogramming** – variadic templates enable compile-time recursion.

### Core Concepts / Features

1. Variadic Templates
2. Parameter Packs
3. Pack Expansion
4. Fold Expressions
5. Recursive Variadic Templates
6. Variadic Template Best Practices

---

## 1. Variadic Templates

### Definitions

**Core Definition**
A variadic template is a template that accepts a variable number of template arguments through one or more parameter packs.

**Technical Definition**
A template with at least one parameter pack is called a variadic template. A variadic class template can be instantiated with any number of template arguments: `template<class... Types> struct Tuple {};` — `Tuple<>`, `Tuple<int>`, `Tuple<int, float>`, etc.. A variadic function template can be called with any number of function arguments; template arguments are deduced through template argument deduction.

**Beginner-Friendly Explanation**
A variadic template is a **template that can take any number of arguments**. `std::make_unique<T>(args...)` is variadic: you can pass zero, one, or many constructor arguments, and it forwards them to `T`'s constructor.

### Purposes

- To write functions and classes that accept a variable number of arguments of various types.
- To implement generic factories (`std::make_unique`, `std::make_shared`).
- To build tuples, variants, and other heterogeneous containers.
- To replace unsafe C-style varargs (`printf`-style) with type-safe variadic templates.
- To enable perfect forwarding of argument packs.

### Syntax Rules and Structure

**General Syntax**
```cpp
template <typename... Args>
return_type function_name(Args... args);

template <typename... Types>
class ClassName { ... };
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `typename... Args` | Template parameter pack of types. |
| `Args... args` | Function parameter pack. |
| `Args...` | Pack expansion in a template argument list. |

**Syntax Rules**
- A template parameter pack is introduced by `typename...` or `class...` (and for non-type packs, by a type followed by `...`).
- A function parameter pack is introduced by a type followed by `...` and a declarator.
- At least one parameter pack is required for a template to be variadic.
- Pack expansion requires a pattern containing at least one unexpanded pack.

**Constraints and Limitations**
- The number of elements in a pack is determined at instantiation.
- Packs cannot be indexed directly; they must be expanded or processed recursively.
- Variadic templates can increase compile times and code size.

### Annotated Code Examples

**Example 1: Variadic function template**

```cpp
#include <iostream>

template <typename... Args>
void print_all(Args... args) {
    (std::cout << ... << args) << '\n';   // C++17 fold expression
}

int main() {
    print_all(1, 2.5, "hello");
}
```

**Expected Output**
```
12.5hello
```

**Why this output?**  
`Args...` is a function parameter pack. The fold expression `(std::cout << ... << args)` expands to `std::cout << 1 << 2.5 << "hello"`.

**Example 2: Variadic class template**

```cpp
#include <iostream>
#include <tuple>

template <typename... Types>
struct Tuple {};

int main() {
    Tuple<> t0;                 // empty pack
    Tuple<int> t1;              // one element
    Tuple<int, double> t2;      // two elements
    std::cout << "ok\n";
}
```

**Expected Output**
```
ok
```

**Why this output?**  
`Tuple` is a variadic class template; it accepts zero or more type arguments.

### Real-World Cases

- **`std::make_unique<T>(args...)`** – forwards constructor arguments.
- **`std::tuple<Types...>`** – heterogeneous tuple.
- **`std::function<Sig>`** – variadic function wrapper.
- **`std::thread` constructor** – forwards callable and arguments.

---

## 2. Parameter Packs

### Definitions

**Core Definition**
A parameter pack is a C++ entity that accepts zero or more template arguments (template parameter pack) or zero or more function arguments (function parameter pack).

**Technical Definition**
A template parameter pack is a template parameter that accepts zero or more template arguments (non-types, types, or templates). A function parameter pack is a function parameter that accepts zero or more function arguments. A template with at least one parameter pack is called a variadic template. The number of elements of a pack equals the number of arguments provided.

**Beginner-Friendly Explanation**
A parameter pack is a **bag that holds zero or more arguments**. You put the bag in the parameter list (`Args... args`), and later you unpack it with `args...` where you need the individual elements.

### Purposes

- To accept a variable number of template arguments or function arguments.
- To forward arguments perfectly (`std::forward<Args>(args)...`).
- To expand packs into base class lists, initializer lists, or function call arguments.
- To compute pack sizes with `sizeof...`.

### Syntax Rules and Structure

**Template Parameter Pack**
```cpp
template <typename... Args>            // type pack
template <int... Ns>                   // non-type pack
template <template <typename> class... Ts>  // template template pack
```

**Function Parameter Pack**
```cpp
template <typename... Args>
void f(Args... args);
```

**Pack Expansion**
```cpp
f(args...);                            // expands to f(a1, a2, ..., aN)
std::forward<Args>(args)...            // expands each forward expression
```

**Component Breakdown**

| Form | Meaning |
|------|---------|
| `typename... Args` | Type template parameter pack. |
| `Args... args` | Function parameter pack. |
| `sizeof...(Args)` | Number of elements in the pack. |

**Syntax Rules**
- A pack expansion must contain at least one unexpanded pack.
- In a primary class template, a template parameter pack must be the final parameter.
- In a function template, a pack may appear earlier if following parameters can be deduced or defaulted.
- `sizeof...` returns a `std::size_t`.

**Constraints and Limitations**
- Packs cannot be indexed directly; you must expand them or use helper facilities (e.g., `std::tuple`).
- Before C++17, processing a pack required recursion; fold expressions (C++17) simplify this.

### Annotated Code Examples

**Example 1: `sizeof...` and pack expansion**

```cpp
#include <iostream>

template <typename... Args>
void inspect(Args... args) {
    std::cout << "count = " << sizeof...(args) << '\n';
}

int main() {
    inspect(1, 2, 3);
    inspect("a", 2.5);
}
```

**Expected Output**
```
count = 3
count = 2
```

**Why this output?**  
`sizeof...(args)` yields the number of arguments in each call.

**Example 2: Perfect forwarding a parameter pack**

```cpp
#include <iostream>
#include <memory>
#include <string>

struct Widget {
    Widget(std::string name, int value) {
        std::cout << name << ": " << value << '\n';
    }
};

template <typename... Args>
auto make_widget(Args&&... args) {
    return std::make_unique<Widget>(std::forward<Args>(args)...);
}

int main() {
    auto w = make_widget("gadget", 42);
}
```

**Expected Output**
```
gadget: 42
```

**Why this output?**  
`Args&&... args` is a forwarding reference pack. `std::forward<Args>(args)...` expands to perfectly forward each argument to `Widget`'s constructor.

### Real-World Cases

- **`std::make_unique` / `std::make_shared`** – forward parameter packs to constructors.
- **`std::tuple` construction** – expands a pack to initialize elements.
- **`std::apply`** – expands a tuple’s elements into a function call.

---

## 3. Pack Expansion

### Definitions

**Core Definition**
Pack expansion is the process of expanding a pattern containing one or more parameter packs into a list of zero or more elements, by substituting each element of the pack.

**Technical Definition**
A pattern followed by an ellipsis, in which the name of at least one parameter pack appears at least once, is expanded into zero or more instantiations of the pattern, where the name of the parameter pack is replaced by each of the elements from the pack, in order. Depending on where the expansion takes place, the resulting comma-separated list is a different kind of list: function argument list, template argument list, base specifier list, member initializer list, etc..

**Beginner-Friendly Explanation**
Pack expansion is **unpacking the bag**. You write `pattern...`, and the compiler replaces the pack with each of its elements, separated by commas. For example, `f(args...)` expands to `f(a1, a2, a3)`.

### Purposes

- To unpack a parameter pack into a list of arguments.
- To forward arguments to constructors or functions.
- To expand packs into base class lists or initializer lists.
- To apply operations to each element of a pack.

### Syntax Rules and Structure

**General Syntax**
```cpp
pattern...
```

**Common Expansion Contexts**

| Context | Example |
|---------|---------|
| Function argument list | `f(args...)` |
| Template argument list | `Tuple<Types...>` |
| Base class list | `class D : public Bases...` |
| Member initializer list | `D() : Bases(args)...` |
| Braced-init-list | `{args...}` |

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `pattern` | An expression or type containing a pack. |
| `...` | Expansion operator. |

**Syntax Rules**
- The pattern must contain at least one unexpanded pack.
- Each pack in the pattern is expanded simultaneously.
- The expansion order is the order of elements in the pack.

**Constraints and Limitations**
- A pack expansion that contains no unexpanded pack is ill-formed.
- The expansion must be in an allowed context (function call, template argument, etc.).

### Annotated Code Examples

**Example 1: Function argument expansion**

```cpp
#include <iostream>

void print() { std::cout << '\n'; }

template <typename T, typename... Rest>
void print(T first, Rest... rest) {
    std::cout << first << ' ';
    print(rest...);             // expand rest into arguments
}

int main() {
    print(1, 2.5, "three");
}
```

**Expected Output**
```
1 2.5 three 
```

**Why this output?**  
`print(rest...)` expands the remaining pack into arguments for the recursive call.

**Example 2: Base class expansion**

```cpp
#include <iostream>

struct A { void a() { std::cout << "A\n"; } };
struct B { void b() { std::cout << "B\n"; } };

template <typename... Bases>
struct D : Bases... { };        // expand Bases into base classes

int main() {
    D<A, B> d;
    d.a();
    d.b();
}
```

**Expected Output**
```
A
B
```

**Why this output?**  
`Bases...` expands to `A, B`, making `D` inherit from both.

### Real-World Cases

- **`std::make_unique<T>(args...)`** – expands `args` into the constructor call.
- **`std::tuple<Types...>`** – expands `Types` into template arguments.
- **`std::apply`** – expands tuple elements into a function call.
- **Variadic mixins** – expanding base classes.

---

## 4. Fold Expressions

### Definitions

**Core Definition**
A fold expression (C++17) reduces a parameter pack over a binary operator in a single expression, without recursion.

**Technical Definition**
A fold expression performs a fold of a template parameter pack over a binary operator. There are four forms: unary right fold `(pack op ...)`, unary left fold `(... op pack)`, binary right fold `(pack op ... op init)`, and binary left fold `(init op ... op pack)`. All 32 binary operators are supported, including `+`, `-`, `*`, `/`, `&&`, `||`, `,`, and comparison operators. An empty pack is valid only for `&&` (→ `true`), `||` (→ `false`), and `,` (→ `void()`) in unary folds.

**Beginner-Friendly Explanation**
A fold expression lets you **combine all elements of a pack with an operator in one line**. Instead of writing a recursive function to sum a pack, you write `(args + ...)`.

### Purposes

- To reduce a parameter pack with a binary operator without recursion.
- To simplify variadic template code.
- To express summations, logical conjunctions, and comma-separated operations concisely.
- To improve compile times and readability compared to recursive expansion.

### Syntax Rules and Structure

**General Syntax**
```cpp
(pack op ...)              // unary right fold
(... op pack)              // unary left fold
(pack op ... op init)      // binary right fold
(init op ... op pack)      // binary left fold
```

**Component Breakdown**

| Form | Expansion (conceptual) |
|------|------------------------|
| `(E op ...)` | `E1 op (E2 op (... op EN))` |
| `(... op E)` | `((E1 op E2) op ...) op EN` |
| `(E op ... op I)` | `E1 op (... op (EN op I))` |
| `(I op ... op E)` | `((I op E1) op E2) op ...` |

**Syntax Rules**
- The parentheses are required.
- In a binary fold, both operators must be the same.
- An empty unary fold is allowed only for `&&` (true), `||` (false), and `,` (void()).

**Constraints and Limitations**
- Fold expressions are a C++17 feature.
- The operator must be one of the 32 supported binary operators.
- Overloaded operators are used as-is; side effects and evaluation order follow the operator’s rules.

### Annotated Code Examples

**Example 1: Summing a pack**

```cpp
#include <iostream>

template <typename... Args>
auto sum(Args... args) {
    return (args + ...);        // unary right fold
}

int main() {
    std::cout << sum(1, 2, 3, 4) << '\n';
}
```

**Expected Output**
```
10
```

**Why this output?**  
`(args + ...)` expands to `1 + (2 + (3 + 4))`, which evaluates to `10`.

**Example 2: Logical AND with an empty pack**

```cpp
#include <iostream>

template <typename... Args>
bool all_true(Args... args) {
    return (args && ...);       // empty pack yields true
}

int main() {
    std::cout << all_true(true, true, false) << '\n';
    std::cout << all_true() << '\n';   // empty pack
}
```

**Expected Output**
```
0
1
```

**Why this output?**  
`all_true(true, true, false)` folds to `true && true && false` = `false` (printed as `0`). The empty pack yields `true` (printed as `1`), as specified for `&&`.

**Example 3: Comma fold for side effects**

```cpp
#include <iostream>

template <typename... Args>
void print_all(Args... args) {
    ((std::cout << args << ' '), ...) << '\n';   // comma fold
}

int main() {
    print_all(1, 2.5, "three");
}
```

**Expected Output**
```
1 2.5 three 
```

**Why this output?**  
The comma fold expands to `(std::cout << 1 << ' '), (std::cout << 2.5 << ' '), (std::cout << "three << ' ')`, executing each comma-separated expression in order.

### Real-World Cases

- **`std::apply`** – often implemented with fold expressions.
- **Variadic `sum`, `product`, `all_of`, `any_of`** – compact implementations.
- **Variadic logging** – folding `<<` over a pack.
- **Tuple hashing** – folding a hash-combining operation.

---

## 5. Recursive Variadic Templates

### Definitions

**Core Definition**
Recursive variadic templates process a parameter pack by peeling off one argument at a time and recursively calling the function with the remaining arguments, until a base case is reached.

**Technical Definition**
Before fold expressions (pre-C++17), variadic templates were processed with recursion: a general template processes the first argument and recursively calls itself with the rest; a base-case overload (with no arguments or a single argument) terminates the recursion. This pattern is still useful when the processing logic cannot be expressed as a fold expression.

**Beginner-Friendly Explanation**
A recursive variadic template is a **loop written as recursion**. It takes the first argument, does something with it, then calls itself with the rest of the arguments. When no arguments are left, the base case stops the recursion.

### Purposes

- To process each element of a pack with custom logic.
- To implement variadic utilities before C++17 fold expressions.
- To perform type-dependent processing of each argument.
- To handle cases where fold expressions are insufficient.

### Syntax Rules and Structure

**General Syntax**
```cpp
// Base case
void f() { /* termination */ }

// Recursive case
template <typename T, typename... Rest>
void f(T first, Rest... rest) {
    // process first
    f(rest...);   // recurse with remaining arguments
}
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| Base case | Non-template or template with no pack. |
| Recursive case | Template with `T` and `Rest...`. |

**Syntax Rules**
- The base case must be reachable when the pack is empty.
- The recursive case peels off the first argument and passes the rest.
- The base case is often a non-template function.

**Constraints and Limitations**
- Recursion depth can be high for large packs.
- Compile times may increase.
- Fold expressions (C++17) are preferred for simple reductions.

### Annotated Code Examples

**Example 1: Recursive print**

```cpp
#include <iostream>

void print() { std::cout << '\n'; }   // base case

template <typename T, typename... Rest>
void print(T first, Rest... rest) {
    std::cout << first << ' ';
    print(rest...);                   // recursive case
}

int main() {
    print(1, 2.5, "three");
}
```

**Expected Output**
```
1 2.5 three 
```

**Why this output?**  
The recursive case prints `first` and recurses with `rest...` until the base case `print()` is reached.

**Example 2: Recursive sum (pre-C++17)**

```cpp
#include <iostream>

int sum() { return 0; }               // base case

template <typename T, typename... Rest>
auto sum(T first, Rest... rest) {
    return first + sum(rest...);      // recursive case
}

int main() {
    std::cout << sum(1, 2, 3, 4) << '\n';
}
```

**Expected Output**
```
10
```

**Why this output?**  
`sum(1, 2, 3, 4)` expands to `1 + sum(2, 3, 4)` → `1 + 2 + sum(3, 4)` → … → `1 + 2 + 3 + 4 + 0 = 10`.

### Real-World Cases

- **Pre-C++17 variadic utilities** – printing, summing, applying functions.
- **Type-dependent processing** – handling each argument differently.
- **Tuple element access** – recursive indexing (before `std::apply`).

---

## 6. Variadic Template Best Practices

### Definitions

**Core Definition**
Variadic template best practices are guidelines for using variadic templates correctly, safely, and efficiently.

**Technical Definition**
The C++ Core Guidelines provide variadic template rules: **T.100** (use variadic templates for heterogeneous variable arguments; don’t use C varargs), **T.101** (how to pass arguments: beware of move-only and reference arguments), **T.102** (how to process arguments: forwarding, type checking, references), and **T.103** (don’t use variadic templates for homogeneous argument lists; use `std::initializer_list` instead). Additional guidance: prefer fold expressions (C++17) over recursion for simple reductions; use perfect forwarding (`std::forward`) to preserve value categories; and constrain templates with concepts (C++20) where possible.

**Beginner-Friendly Explanation**
Best practices for variadic templates are simple: **use variadic templates for heterogeneous argument lists**, **don’t use them for homogeneous lists**, **use fold expressions when possible**, **forward arguments perfectly**, and **constrain with concepts**.

### Key Guidelines (Summary Table)

| Guideline | Summary | Recommendation |
|-----------|---------|----------------|
| **T.100** | Use variadic templates for heterogeneous variable arguments. | Avoid C varargs. |
| **T.101** | Pass arguments carefully (move-only, references). | Use `std::forward`. |
| **T.102** | Process arguments with forwarding and type checking. | Prefer fold expressions. |
| **T.103** | Don’t use variadic templates for homogeneous lists. | Use `std::initializer_list`. |
| **Prefer folds** | Fold expressions (C++17) are clearer than recursion. | `(args + ...)`. |
| **Constrain with concepts** | Concepts (C++20) express requirements. | `template <std::integral... Ts>`. |

### Annotated Code Examples

**Example 1: Fold expression instead of recursion (T.102)**

```cpp
#include <iostream>

// GOOD: fold expression (C++17)
template <typename... Args>
auto sum(Args... args) {
    return (args + ...);
}

int main() {
    std::cout << sum(1, 2, 3, 4) << '\n';
}
```

**Expected Output**
```
10
```

**Why this is best practice?**  
Fold expressions are concise and avoid the boilerplate of recursive base cases.

**Example 2: Homogeneous list should use `initializer_list` (T.103)**

```cpp
#include <iostream>
#include <initializer_list>

// GOOD: initializer_list for homogeneous arguments
int sum(std::initializer_list<int> values) {
    int total = 0;
    for (int v : values) total += v;
    return total;
}

int main() {
    std::cout << sum({1, 2, 3, 4}) << '\n';
}
```

**Expected Output**
```
10
```

**Why this is best practice?**  
When all arguments have the same type, `std::initializer_list` is simpler and more precise than a variadic template (T.103).

### Real-World Cases

- **Modern logging libraries** – use variadic templates and fold expressions.
- **Factory functions** – perfect forwarding of parameter packs.
- **Type-safe formatting** – `std::format` (C++20) replaces `printf` with variadic templates.

---

## References

- Parameter pack (cppreference) – https://en.cppreference.com/w/cpp/language/parameter_pack
- Pack (cppreference) – https://en.cppreference.com/w/cpp/language/pack
- Fold expressions (cppreference) – https://en.cppreference.com/w/cpp/language/fold
- Folding expressions (N4295, Sutton & Smith) – https://isocpp.org/files/papers/n4295.html
- C++ Core Guidelines — T.100: Use variadic templates when you need a function that takes a variable number of arguments of a variety of types – https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Tvar
- C++ Core Guidelines — T.103: Don’t use variadic templates for homogeneous argument lists – https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Tvar
- C++ Core Guidelines — T.101, T.102 (variadic template rules) – https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Tvar
- Use Fold Expressions (C++17) to Operate on Parameter Packs (Dr-Sergey/learn_cpp) – https://raw.githubusercontent.com/Dr-Sergey/learn_cpp/08279578ce23b1cbf2d3394357620600db618eca/05_Templates_and_Generic_Programming/Use_fold_expressions_C++17_to_operate_on_parameter_packs.md
- Use Variadic Templates and Parameter Packs Correctly (Dr-Sergey/learn_cpp) – https://raw.githubusercontent.com/Dr-Sergey/learn_cpp/6642b53a22d13b9812d2ca5cf39a63b34cc70e2c/05_Templates_and_Generic_Programming/Use_variadic_templates_and_parameter_packs_correctly.md
- Variadic Templates (OpenTechSchool Zurich) – https://raw.githubusercontent.com/opentechschool-zurich/cpp-co-learning/refs/heads/master/topics/variadic-templates/delf/01_intro/README.md

---

This cheat sheet provides a complete, structured reference for C++ variadic templates, covering variadic templates, parameter packs, pack expansion, fold expressions, recursive variadic templates, and best practices. Each section follows a uniform template with definitions, purposes, syntax rules, annotated examples, expected outputs, and authoritative references.