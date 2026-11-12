# Concepts (C++20) — Comprehensive Programming Cheat Sheet


## Topic Overview

### Definitions

**Core Definition**
A concept is a named compile-time predicate that specifies the **syntactic and semantic requirements** that a template argument (or a set of template arguments) must satisfy for a template to be used.

**Technical Definition**
A concept is a named set of **constraints** on template parameters. Concepts are defined using the `concept` keyword and a `requires` expression (or a boolean expression of other concepts). They participate in **constraint normalization** and **partial ordering**, enabling the compiler to select the most constrained viable overload, produce precise error messages naming the unsatisfied concept, and eliminate verbose SFINAE boilerplate. Concepts are first-class citizens of the type system: they can be used in `requires` clauses, as abbreviated function template parameter types (e.g., `void f(std::integral auto x)`), and in `if constexpr` or `static_assert` conditions.

**Beginner-Friendly Explanation**
A concept is a **named contract for template parameters**. Instead of writing cryptic SFINAE (`enable_if`, `void_t`, `decltype`) to say “this function only works for integer types,” you write a concept called `std::integral` and use it directly. If someone passes a `double`, the compiler says “`double` does not satisfy `std::integral`” instead of dumping pages of template substitution errors. Concepts make generic code **readable, self-documenting, and easier to get right**—they are the modern replacement for most SFINAE techniques.

### Key Characteristics

- **Named constraints**: Concepts are named predicates that can be reused and composed.
- **Compile-time evaluation**: A concept evaluates to `true` or `false` at compile time.
- **Readable syntax**: Constraints appear next to the template parameter or function signature, not buried in return types.
- **Better error messages**: The compiler names the unsatisfied concept instead of showing substitution failures.
- **Partial ordering**: More constrained templates are preferred during overload resolution.
- **Composability**: Concepts can be combined with `&&` and `||` to form new concepts.
- **Abbreviated syntax**: `std::integral auto x` is shorthand for a constrained template parameter.
- **Standard library support**: The `<concepts>` header provides many predefined concepts.

### Prerequisites

- Understanding of templates (function and class templates).
- Familiarity with SFINAE and `std::enable_if` (for comparison).
- Knowledge of type traits (`<type_traits>`).
- Basic awareness of `requires` expressions and compile-time boolean logic.

### Related Programming Areas

- **Generic programming** – concepts constrain generic templates.
- **Template metaprogramming** – concepts replace much SFINAE machinery.
- **Ranges library (C++20)** – `std::ranges::range` and related concepts.
- **Standard library** – many C++20 algorithms and containers are concept-constrained.
- **API design** – concepts express interface contracts at API boundaries.
- **Error diagnostics** – concepts dramatically improve template error messages.

### Core Concepts / Features

1. Concepts
2. `concept` Keyword
3. `requires` Clause
4. `requires` Expression
5. Concept Definition
6. Concept Composition
7. Standard Concepts
8. Concept Best Practices
9. Concepts vs. SFINAE


## 1. Concepts

### Definitions

**Core Definition**
A concept is a named compile-time boolean predicate that expresses requirements on template parameters.

**Technical Definition**
A concept is a template entity declared with the `concept` keyword. It is defined by a constraint-expression, which is a constant expression of type `bool`. When a template uses a concept (via a `requires` clause or abbreviated syntax), the compiler checks whether the supplied template arguments satisfy the concept. If they do not, the template is not a viable candidate (for function templates) or the program is ill-formed (for class templates). Concepts participate in constraint normalization and partial ordering, enabling more constrained overloads to be preferred.

**Beginner-Friendly Explanation**
A concept is a **named “requirement”** for template parameters. You define it once, give it a clear name (like `std::integral` or `std::copyable`), and then use it wherever you need that requirement. It is like a type of “label” that says “this template only works for types that satisfy this label.”

### Purposes

- To express template parameter requirements clearly and declaratively.
- To replace verbose SFINAE (`enable_if`, `void_t`) with readable constraints.
- To improve compiler error messages by naming the unsatisfied requirement.
- To enable concept-based overloading (more constrained templates are preferred).
- To document the interface contract of generic code.
- To compose requirements using logical operators (`&&`, `||`).

### Syntax Rules and Structure

**Using a Standard Concept (Requires Clause)**
```cpp
#include <concepts>

template <typename T>
requires std::integral<T>
T add(T a, T b) { return a + b; }
```

**Using an Abbreviated Function Template (C++20)**
```cpp
void process(std::integral auto value) { /* ... */ }
```

**Using a Concept in a Class Template**
```cpp
template <std::integral T>
class Counter { /* ... */ };
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `template <...>` | Template parameter list. |
| `requires std::integral<T>` | `requires` clause constraining `T`. |
| `std::integral auto` | Abbreviated constrained parameter. |
| `template <std::integral T>` | Constrained template parameter. |

**Syntax Rules**
- Concepts can be used in a `requires` clause after the template parameter list.
- Concepts can be used as constrained `auto` placeholders in function parameters.
- Concepts can be used as constrained template parameters (`template <Concept T>`).
- Concepts can be used in `if constexpr` and `static_assert` conditions.

**Constraints and Limitations**
- Concepts are a C++20 feature; they require a C++20-compliant compiler.
- Concepts should be statically verifiable by the compiler; semantic-only requirements should be expressed via comments or assertions.
- Overusing concepts can increase code coupling and complexity.

### Annotated Code Examples

**Example 1: Constraining a function template with `std::integral`**

```cpp
#include <iostream>
#include <concepts>

template <typename T>
requires std::integral<T>
T double_it(T value) {
    return value * 2;
}

int main() {
    std::cout << double_it(5) << '\n';    // OK: int satisfies std::integral
    // double_it(3.14);                   // error: double does not satisfy std::integral
}
```

**Expected Output**
```
10
```

**Why this output?**  
`double_it` is constrained to integral types. `int` satisfies `std::integral`, so the function is viable and returns `10`. If `double` is passed, the compiler reports that `double` does not satisfy `std::integral`.

**Example 2: Abbreviated constrained parameter**

```cpp
#include <iostream>
#include <concepts>

void print_integral(std::integral auto value) {
    std::cout << "integral: " << value << '\n';
}

int main() {
    print_integral(42);       // OK
    // print_integral(3.14);  // error
}
```

**Expected Output**
```
integral: 42
```

**Why this output?**  
`std::integral auto` is shorthand for a constrained template parameter. Only integral types are accepted.

### Real-World Cases

- **`std::ranges::sort`** – constrained on `std::ranges::range` and comparability concepts.
- **`std::make_unique`** – constrained on constructibility concepts in some implementations.
- **Library API design** – concepts document and enforce interface contracts.
- **Generic algorithms** – selecting the most constrained overload via partial ordering.


## 2. `concept` Keyword

### Definitions

**Core Definition**
The `concept` keyword introduces a named concept definition.

**Technical Definition**
The `concept` keyword is used in a concept-definition: `template < parameter-list > concept concept-name = constraint-expression ;`. The constraint-expression is a constant expression of type `bool` that evaluates to `true` if the requirements are satisfied. Concepts are typically defined at namespace scope and may be used in `requires` clauses or as constrained template parameters.

**Beginner-Friendly Explanation**
The `concept` keyword is how you **create your own named requirement**. You write `template <typename T> concept MyConcept = ...;`, and then you can use `MyConcept` wherever you need that requirement.

### Purposes

- To define reusable, named constraints.
- To encapsulate complex requirement logic behind a clear name.
- To enable composition of requirements.
- To improve code readability and maintenance.

### Syntax Rules and Structure

**General Syntax**
```cpp
template <typename T>
concept ConceptName = constraint-expression;
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `template <typename T>` | Template parameter list. |
| `concept ConceptName` | The concept name. |
| `= constraint-expression` | A compile-time boolean expression. |

**Syntax Rules**
- The constraint-expression must be a constant expression of type `bool`.
- Concept definitions are typically placed at namespace scope.
- Concepts can be used immediately after definition.

**Constraints and Limitations**
- Concepts cannot be defined inside function bodies (until C++23 for some cases).
- Concept names must be unique within their scope.

### Annotated Code Examples

**Example 1: Defining a custom concept**

```cpp
#include <iostream>
#include <concepts>

template <typename T>
concept Addable = requires(T a, T b) {
    a + b;   // simple requirement: a + b must be valid
};

template <Addable T>
T add(T a, T b) { return a + b; }

int main() {
    std::cout << add(1, 2) << '\n';
}
```

**Expected Output**
```
3
```

**Why this output?**  
`Addable` requires that `a + b` is valid. `int` satisfies `Addable`, so `add` is viable.

### Real-World Cases

- **Domain-specific concepts** – `Serializable`, `Printable`, `Hashable`.
- **Library constraints** – custom concepts for library-specific requirements.
- **Type categorization** – grouping types by supported operations.


## 3. `requires` Clause

### Definitions

**Core Definition**
A `requires` clause attaches a **constraint** to a template or function, restricting the template parameters to those that satisfy the constraint.

**Technical Definition**
A `requires` clause appears after the template parameter list (or after the function declarator) and before the function body: `template <typename T> requires Constraint<T> void f();`. The constraint is a constant expression of type `bool` (often a concept, a `requires` expression, or a conjunction/disjunction of concepts). If the constraint evaluates to `false`, the template is not considered during overload resolution.

**Beginner-Friendly Explanation**
A `requires` clause is the **“only if” clause** attached to a template. It says: “Only use this template if this requirement is met.” It is different from a `requires` expression: the clause is a constraint on the template, while the expression is the compile-time test itself.

### Purposes

- To constrain templates to types that satisfy specific requirements.
- To enable concept-based overloading (more constrained templates are preferred).
- To improve error messages by rejecting invalid types early.
- To document template requirements at the point of declaration.

### Syntax Rules and Structure

**General Syntax**
```cpp
template <typename T>
requires Constraint<T>
return_type function_name(parameters);
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `template <typename T>` | Template parameter list. |
| `requires Constraint<T>` | The constraint clause. |
| `function_name` | The constrained function. |

**Syntax Rules**
- The `requires` clause follows the template parameter list.
- The constraint must be a constant expression of type `bool`.
- Multiple constraints can be combined with `&&` and `||`.
- A `requires` clause can be used with class templates, function templates, and member functions.

**Constraints and Limitations**
- The constraint must be statically verifiable by the compiler.
- Constraints that are too restrictive can make code inflexible.

### Annotated Code Examples

**Example 1: `requires` clause with a concept**

```cpp
#include <iostream>
#include <concepts>

template <typename T>
requires std::floating_point<T>
T safe_divide(T a, T b) {
    return b != 0 ? a / b : 0;
}

int main() {
    std::cout << safe_divide(10.0, 3.0) << '\n';
    // safe_divide(10, 3);   // error: int does not satisfy std::floating_point
}
```

**Expected Output**
```
3.33333
```

**Why this output?**  
`safe_divide` is constrained to floating-point types. `double` satisfies `std::floating_point`, so the function is viable and returns the division result.

**Example 2: `requires` clause with a `requires` expression**

```cpp
#include <iostream>
#include <concepts>

template <typename T>
requires requires(T a, T b) { a + b; }
T add(T a, T b) { return a + b; }

int main() {
    std::cout << add(1, 2) << '\n';
}
```

**Expected Output**
```
3
```

**Why this output?**  
The `requires` clause contains a `requires` expression that checks whether `a + b` is valid. `int` satisfies it, so the function is enabled.

### Real-World Cases

- **Standard library algorithms** – constrained on iterator and range concepts.
- **Generic factories** – constrained on constructibility.
- **Mathematical utilities** – constrained on numeric concepts.


## 4. `requires` Expression

### Definitions

**Core Definition**
A `requires` expression is a **compile-time boolean expression** that tests whether a set of syntactic and semantic requirements are satisfied by a type or a set of expressions.

**Technical Definition**
A `requires` expression has the form `requires ( optional parameter-list ) { requirement-seq }`. It yields a prvalue of type `bool`. The requirements are checked in lexical order; if substitution fails or a semantic constraint is violated, the expression evaluates to `false` without causing a hard error (provided the failure is in the immediate context). There are four kinds of requirements: **simple** (`expr;`), **type** (`typename T::type;`), **compound** (`{ expr } -> concept;`), and **nested** (`requires predicate;`).

**Beginner-Friendly Explanation**
A `requires` expression is the **actual test** that checks whether a type supports certain operations. It asks questions like: “Is `t.size()` valid?” “Does `T` have a nested `value_type`?” “Does `t.serialize()` return something convertible to `std::string`?” The answer is `true` or `false` at compile time.

### Purposes

- To test whether expressions are valid for a type.
- To check for the existence of nested types.
- To verify that a function returns a specific type.
- To compose multiple requirements into a single predicate.
- To define concepts (a `requires` expression is often the body of a concept).

### Syntax Rules and Structure

**General Syntax**
```cpp
requires ( optional-parameter-list ) {
    requirement-seq
}
```

**Four Requirement Kinds**

| Kind | Syntax | Checks |
|------|--------|--------|
| **Simple** | `expr;` | Expression is valid (compiles) |
| **Type** | `typename T::type;` | Type exists |
| **Compound** | `{ expr } -> concept;` | Expression is valid AND its return type satisfies a concept |
| **Nested** | `requires predicate;` | An additional boolean constraint must hold |

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `optional-parameter-list` | Local parameters used in the requirements. |
| `requirement-seq` | The list of requirements. |

**Syntax Rules**
- Each requirement is one of the four kinds.
- Requirements are checked in lexical order; evaluation stops at the first failure.
- If substitution fails for every possible template argument, the program is ill-formed (no diagnostic required).
- Local parameters have no linkage or lifetime; they are used only as notation.

**Constraints and Limitations**
- A `requires` expression can appear inside a `requires` clause or a concept definition.
- The expression must not contain errors that are not in the immediate context (those are hard errors).

### Annotated Code Examples

**Example 1: All four requirement kinds**

```cpp
#include <iostream>
#include <concepts>
#include <string>

template <typename T>
concept FullyConstrained = requires(T a, T b) {
    a + b;                                          // Simple: a+b compiles
    typename T::value_type;                         // Type: T has value_type
    { a.size() } -> std::convertible_to<std::size_t>; // Compound: size() returns size_t-like
    requires std::copyable<T>;                      // Nested: T must be copyable
};

struct Widget {
    using value_type = int;
    Widget operator+(const Widget&) const;
    std::size_t size() const;
};

int main() {
    std::cout << std::boolalpha
              << FullyConstrained<Widget> << '\n'    // 1
              << FullyConstrained<int> << '\n';      // 0
}
```

**Expected Output**
```
true
false
```

**Why this output?**  
`Widget` satisfies all four requirements: `operator+` exists, `value_type` exists, `size()` returns `std::size_t`, and `Widget` is copyable. `int` lacks `value_type` and `size()`, so it fails.

**Example 2: Compound requirement with a concept**

```cpp
#include <iostream>
#include <concepts>
#include <string>

template <typename T>
concept Serializable = requires(T t) {
    { t.serialize() } -> std::convertible_to<std::string>;
};

struct Document {
    std::string serialize() const { return "doc"; }
};

struct Bad {
    int serialize() const { return 0; }
};

int main() {
    std::cout << std::boolalpha
              << Serializable<Document> << '\n'   // 1
              << Serializable<Bad> << '\n';       // 0
}
```

**Expected Output**
```
true
false
```

**Why this output?**  
`Document::serialize()` returns `std::string`, which satisfies `std::convertible_to<std::string>`. `Bad::serialize()` returns `int`, which does not.

### Real-World Cases

- **`std::ranges::range`** – defined using `requires` expressions checking `begin()`/`end()`.
- **`std::invocable`** – checks that a callable can be invoked with given arguments.
- **`std::predicate`** – checks that a callable returns a boolean-testable value.
- **Custom concepts** – domain-specific requirements.


## 5. Concept Definition

### Definitions

**Core Definition**
A concept definition is a declaration that introduces a named concept with a constraint-expression.

**Technical Definition**
A concept definition has the form `template < parameter-list > concept name = constraint-expression;`. The constraint-expression may be a `requires` expression, a boolean expression involving other concepts, or a combination. The concept is a compile-time boolean predicate on its template parameters. Concepts are typically placed at namespace scope and can be used immediately after definition.

**Beginner-Friendly Explanation**
A concept definition is **creating your own named requirement**. You write `template <typename T> concept MyReq = ...;`, and then `MyReq` becomes a reusable label for that requirement.

### Purposes

- To create reusable, named constraints.
- To encapsulate complex requirements behind a clear name.
- To enable composition and partial ordering.
- To document requirements in a self-describing way.

### Syntax Rules and Structure

**General Syntax**
```cpp
template <typename T, ...>
concept Name = constraint-expression;
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `template <typename T, ...>` | Template parameter list. |
| `concept Name` | The concept name. |
| `= constraint-expression` | Compile-time boolean constraint. |

**Syntax Rules**
- The constraint-expression must be a constant expression of type `bool`.
- Concepts can be defined in namespaces.
- A concept can use other concepts in its definition.

**Constraints and Limitations**
- Concepts should be statically verifiable by the compiler.
- Avoid defining concepts that duplicate requirements obvious from the function body.

### Annotated Code Examples

**Example 1: Defining a concept with a `requires` expression**

```cpp
#include <iostream>
#include <concepts>

template <typename T>
concept HasSize = requires(T t) {
    { t.size() } -> std::convertible_to<std::size_t>;
};

template <HasSize T>
void print_size(const T& container) {
    std::cout << container.size() << '\n';
}

#include <vector>
int main() {
    print_size(std::vector<int>{1, 2, 3});   // OK
}
```

**Expected Output**
```
3
```

**Why this output?**  
`HasSize` checks that `t.size()` is valid and returns something convertible to `std::size_t`. `std::vector<int>` satisfies this concept.

### Real-World Cases

- **Library-specific concepts** – `Serializable`, `Hashable`, `Printable`.
- **Domain concepts** – `Numeric`, `Container`, `Iterator`.
- **Composition of standard concepts** – `std::integral || std::floating_point`.


## 6. Concept Composition

### Definitions

**Core Definition**
Concept composition is the combination of two or more concepts using logical operators (`&&`, `||`, `!`) to form new constraints.

**Technical Definition**
Concepts can be combined in a constraint-expression using `&&` (conjunction), `||` (disjunction), and `!` (negation). Constraint normalization and partial ordering determine which composed constraint is more specialized. Composed concepts can be used in `requires` clauses and concept definitions.

**Beginner-Friendly Explanation**
Concept composition is **combining requirements with “and”, “or”, and “not.”** For example, `std::integral<T> || std::floating_point<T>` means “T is either an integer or a floating-point type.”

### Purposes

- To express complex requirements by combining simpler concepts.
- To create disjunctions (either/or requirements).
- To negate a requirement.
- To enable more precise overload selection via partial ordering.

### Syntax Rules and Structure

**General Syntax**
```cpp
template <typename T>
concept Numeric = std::integral<T> || std::floating_point<T>;

template <typename T>
requires std::integral<T> && std::copyable<T>
void f(T value);
```

**Component Breakdown**

| Operator | Meaning |
|----------|---------|
| `&&` | Conjunction (both must hold). |
| `\|\|` | Disjunction (at least one must hold). |
| `!` | Negation (must not hold). |

**Syntax Rules**
- Composed constraints are evaluated at compile time.
- Partial ordering prefers more constrained templates.
- Parentheses can be used to group constraints.

**Constraints and Limitations**
- Overly complex compositions can be hard to read.
- Partial ordering rules for disjunctions can be subtle.

### Annotated Code Examples

**Example 1: Disjunction of concepts**

```cpp
#include <iostream>
#include <concepts>

template <typename T>
concept Numeric = std::integral<T> || std::floating_point<T>;

template <Numeric T>
T half(T value) { return value / 2; }

int main() {
    std::cout << half(10) << '\n';       // OK: int is integral
    std::cout << half(3.0) << '\n';      // OK: double is floating-point
    // half("hello");                   // error: const char* is not numeric
}
```

**Expected Output**
```
5
1.5
```

**Why this output?**  
`Numeric` accepts both integral and floating-point types. `int` and `double` satisfy the concept; `const char*` does not.

### Real-World Cases

- **`std::ranges::range` composition** – combining `begin()` and `end()` requirements.
- **Standard concepts** – many are composed from simpler concepts.
- **Custom numeric concepts** – `Numeric = integral || floating_point`.


## 7. Standard Concepts

### Definitions

**Core Definition**
Standard concepts are the predefined concepts in the C++20 `<concepts>` header (and the Ranges library), covering common requirements such as type identity, inheritance, convertibility, arithmetic categories, callability, and range properties.

**Technical Definition**
The `<concepts>` header provides a large set of standard concepts, including language-related concepts (`same_as`, `derived_from`, `convertible_to`, `integral`, `floating_point`), comparison concepts (`equality_comparable`, `totally_ordered`), object concepts (`copyable`, `movable`, `regular`), and callable concepts (`invocable`, `predicate`, `relation`, `strict_weak_order`). The Ranges library adds `std::ranges::range` and related concepts.

**Beginner-Friendly Explanation**
Standard concepts are **ready-made requirements** provided by the C++ standard library. Instead of writing your own `is_integral` SFINAE, you use `std::integral`. They cover the most common needs and are the preferred choice over hand-written traits.

### Purposes

- To provide ready-to-use, well-tested constraints.
- To replace hand-written type traits and SFINAE.
- To standardize the vocabulary of generic programming.
- To improve error messages and code readability.

### `std::same_as`

**Core Definition** — `std::same_as<T, U>` is satisfied if `T` and `U` are the same type (including cv-qualification).

**Syntax**
```cpp
std::same_as<T, U>   // concept
std::is_same_v<T, U> // equivalent trait
```

**Example**
```cpp
#include <concepts>
#include <iostream>

template <typename T>
requires std::same_as<T, int>
void only_int(T value) { std::cout << value << '\n'; }

int main() {
    only_int(42);        // OK
    // only_int(3.14);   // error: double is not int
}
```

**Expected Output**
```
42
```

### `std::derived_from`

**Core Definition** — `std::derived_from<Derived, Base>` is satisfied if `Derived` is derived from `Base` (or is the same class, ignoring cv-qualification).

**Syntax**
```cpp
std::derived_from<Derived, Base>
```

**Example**
```cpp
#include <concepts>
#include <iostream>

struct A {};
struct B : A {};

template <typename T>
requires std::derived_from<T, A>
void process(T) { std::cout << "A-derived\n"; }

int main() {
    process(B{});   // OK
}
```

**Expected Output**
```
A-derived
```

### `std::convertible_to`

**Core Definition** — `std::convertible_to<From, To>` is satisfied if `From` is implicitly convertible to `To`.

**Syntax**
```cpp
std::convertible_to<From, To>
```

**Example**
```cpp
#include <concepts>
#include <iostream>

template <typename T>
requires std::convertible_to<T, int>
void to_int(T value) { std::cout << static_cast<int>(value) << '\n'; }

int main() {
    to_int(3.14);   // OK: double -> int
}
```

**Expected Output**
```
3
```

### `std::integral`

**Core Definition** — `std::integral<T>` is satisfied if `T` is an integral type (including `bool`, `char`, and all integer types).

**Syntax**
```cpp
std::integral<T>
```

**Example**
```cpp
#include <concepts>
#include <iostream>

template <std::integral T>
T double_it(T value) { return value * 2; }

int main() {
    std::cout << double_it(5) << '\n';   // 10
}
```

**Expected Output**
```
10
```

### `std::floating_point`

**Core Definition** — `std::floating_point<T>` is satisfied if `T` is a floating-point type (`float`, `double`, `long double`).

**Syntax**
```cpp
std::floating_point<T>
```

**Example**
```cpp
#include <concepts>
#include <iostream>

template <std::floating_point T>
T half(T value) { return value / 2; }

int main() {
    std::cout << half(3.0) << '\n';   // 1.5
}
```

**Expected Output**
```
1.5
```

### `std::invocable`

**Core Definition** — `std::invocable<F, Args...>` is satisfied if the callable `F` can be invoked with arguments of types `Args...`.

**Syntax**
```cpp
std::invocable<F, Args...>
```

**Example**
```cpp
#include <concepts>
#include <iostream>
#include <functional>

template <typename F>
requires std::invocable<F, int>
void call_with_42(F f) { f(42); }

int main() {
    call_with_42([](int x) { std::cout << x << '\n'; });
}
```

**Expected Output**
```
42
```

### `std::predicate`

**Core Definition** — `std::predicate<F, Args...>` is satisfied if `F` is invocable with `Args...` and returns a boolean-testable value.

**Syntax**
```cpp
std::predicate<F, Args...>
```

**Example**
```cpp
#include <concepts>
#include <iostream>

template <typename F>
requires std::predicate<F, int>
void test(F f, int value) { std::cout << std::boolalpha << f(value) << '\n'; }

int main() {
    test([](int x) { return x > 0; }, 42);   // true
}
```

**Expected Output**
```
true
```

### `std::ranges::range`

**Core Definition** — `std::ranges::range<R>` is satisfied if `R` is a range (has `begin()` and `end()` iterators).

**Syntax**
```cpp
std::ranges::range<R>
```

**Example**
```cpp
#include <ranges>
#include <concepts>
#include <iostream>
#include <vector>

template <std::ranges::range R>
void print_first(const R& r) {
    if (!r.empty()) std::cout << *r.begin() << '\n';
}

int main() {
    print_first(std::vector<int>{10, 20, 30});
}
```

**Expected Output**
```
10
```

### Real-World Cases

- **`std::ranges` algorithms** – constrained on `std::ranges::range`.
- **Standard containers** – satisfy `std::ranges::range` and related concepts.
- **Callable utilities** – `std::invocable`, `std::predicate` for generic callbacks.


## 8. Concept Best Practices

### Definitions

**Core Definition**
Concept best practices are guidelines for defining and using concepts effectively, readably, and maintainably, based on the C++ Core Guidelines and the Google C++ Style Guide.

**Technical Definition**
Key practices include: prefer standard library concepts over hand-written traits (e.g., `std::integral` instead of `std::is_integral_v`); prefer modern constraint syntax (`requires(Concept<T>)`) over `enable_if`; avoid manually re-implementing existing concepts; define new concepts rarely and only internally within a library, not at API boundaries; do not define concepts that duplicate the function body or obvious requirements; ensure concepts are statically verifiable by the compiler; and use concepts only where their legacy equivalents would be used.

**Beginner-Friendly Explanation**
Best practices for concepts are simple: **use standard concepts when they exist**, **write your own only when necessary**, **keep concepts small and focused**, **don't duplicate what the compiler already knows**, and **use concepts to improve readability and errors, not to over-constrain**.

### Key Guidelines (Summary Table)

| Guideline | Summary | Recommendation |
|-----------|---------|----------------|
| **Prefer standard concepts** | Use `<concepts>` library concepts over traits. | `std::integral` not `std::is_integral_v`. |
| **Prefer modern syntax** | Use `requires` clauses and abbreviated syntax. | `std::integral auto x` or `requires std::integral<T>`. |
| **No manual re-implementation** | Don't redefine existing concepts. | Use `std::default_initializable`, not `requires { T v; }`. |
| **Define concepts rarely** | New concepts only when necessary and internally. | Avoid API-boundary concepts. |
| **Avoid duplication** | Don't duplicate function body requirements. | Leave code as ordinary template unless concepts add value. |
| **Statically verifiable** | Concepts must be compiler-checkable. | Use comments/assertions for semantic constraints. |
| **Prefer concepts over SFINAE** | In C++20+, use concepts, not `enable_if`. | `template <std::integral T>` not `enable_if_t<...>`. |

### Annotated Code Examples

**Example 1: Prefer standard concepts (Good vs. Bad)**

```cpp
#include <iostream>
#include <concepts>
#include <type_traits>

// BAD: hand-written trait-based constraint (pre-C++20 style)
template <typename T, std::enable_if_t<std::is_integral_v<T>, bool> = true>
void bad(T value) { std::cout << value << '\n'; }

// GOOD: standard concept (C++20)
template <std::integral T>
void good(T value) { std::cout << value << '\n'; }

int main() {
    bad(42);
    good(42);
}
```

**Expected Output**
```
42
42
```

**Why this is best practice?**  
`std::integral` is a standard concept that replaces verbose SFINAE and gives better error messages.

### Real-World Cases

- **Standard library** – uses standard concepts extensively.
- **Google C++ Style Guide** – recommends standard concepts over traits, and rare custom concept definitions.
- **Code review** – concept usage is reviewed for necessity and clarity.


## 9. Concepts vs. SFINAE

### Definitions

**Core Definition**
Concepts (C++20) and SFINAE (C++11) both constrain templates, but concepts provide **clearer syntax, better error messages, and composable constraints**, while SFINAE relies on substitution failure and is harder to read and diagnose.

**Technical Definition**
SFINAE removes invalid template specializations from the overload set when substitution fails. Concepts replace this with named predicates that are checked before substitution, producing errors that name the unsatisfied concept. Concepts support partial ordering, abbreviated syntax, and standard library support. The C++ Core Guidelines and Google Style Guide recommend preferring concepts over SFINAE in C++20 code.

**Beginner-Friendly Explanation**
SFINAE is the **old, cryptic way**; concepts are the **new, readable way**. SFINAE says “if substituting the type fails, silently discard the template.” Concepts say “this template requires `std::integral`; if it’s not integral, here’s the error.” Concepts are clearer, safer, and preferred in C++20.

### Comparison Table

| Aspect | SFINAE (`enable_if`) | Concepts (C++20) |
|--------|----------------------|------------------|
| **Syntax** | Verbose, buried in return type or template params | Clean, declarative, next to the signature |
| **Error messages** | Terrible (substitution noise) | Excellent (names the unsatisfied concept) |
| **Readability** | Poor | Great |
| **Composability** | Limited | `&&`, `\|\|`, `!` |
| **Partial ordering** | Limited | Yes |
| **Standard support** | `<type_traits>` | `<concepts>` |
| **Recommendation** | Only in pre-C++20 code | Preferred in C++20+ |

### Annotated Code Examples

**Example 1: SFINAE vs. Concepts side by side**

```cpp
#include <iostream>
#include <type_traits>
#include <concepts>

// SFINAE: constraint buried in template parameter
template <typename T, std::enable_if_t<std::is_integral_v<T>, bool> = true>
T sfinae_double(T value) { return value * 2; }

// Concepts: constraint clearly visible
template <std::integral T>
T concept_double(T value) { return value * 2; }

int main() {
    std::cout << sfinae_double(5) << '\n';   // 10
    std::cout << concept_double(5) << '\n';  // 10
    // sfinae_double(3.14);  // cryptic error
    // concept_double(3.14); // clear error: double does not satisfy std::integral
}
```

**Expected Output**
```
10
10
```

**Why this matters?**  
Both produce the same result, but the concept version communicates intent instantly and produces a clear error naming the unsatisfied constraint.

### Real-World Cases

- **New C++20 code** – concepts are the default; SFINAE is reserved for legacy code.
- **Code migration** – replacing `enable_if` with `requires` clauses.
- **Library design** – concepts improve API usability and diagnostics.


## References

- Concepts (cppreference) — https://en.cppreference.com/w/cpp/language/constraints
- `requires` expression (cppreference) — https://en.cppreference.com/w/cpp/language/requires
- Standard library header `<concepts>` (cppreference) — https://en.cppreference.com/w/cpp/header/concepts
- Concepts library (cppreference) — https://en.cppreference.com/w/cpp/concepts
- C++ Core Guidelines — T.10: Specify concepts for all template arguments — https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#S-templates
- Google C++ Style Guide — Concepts and Constraints — https://chromium.googlesource.com/external/github.com/google/styleguide/+/HEAD/cppguide.md#concepts
- cppreference — `std::same_as` — https://en.cppreference.com/w/cpp/concepts/same_as
- cppreference — `std::derived_from` — https://en.cppreference.com/w/cpp/concepts/derived_from
- cppreference — `std::convertible_to` — https://en.cppreference.com/w/cpp/concepts/convertible_to
- cppreference — `std::integral` — https://en.cppreference.com/w/cpp/concepts/integral
- cppreference — `std::floating_point` — https://en.cppreference.com/w/cpp/concepts/floating_point
- cppreference — `std::invocable` — https://en.cppreference.com/w/cpp/concepts/invocable
- cppreference — `std::predicate` — https://en.cppreference.com/w/cpp/concepts/predicate
- cppreference — `std::ranges::range` — https://en.cppreference.com/w/cpp/ranges/range
- Understand `requires` Expressions and the `requires` Clause (C++20) — https://raw.githubusercontent.com/Dr-Sergey/learn_cpp/226151340afd0f1ddd9af6d47103a0884bf1712c/06_Compile_Time_Programming/Understand_requires_expressions_and_the_requires_clause_C++20.md
- Understand SFINAE and Know When to Replace It with Concepts — https://github.com/Dr-Sergey/learn_cpp/blob/e2c54b8163b274e4f724168226553ae2ab4d7ddd/05_Templates_and_Generic_Programming/Understand_SFINAE_and_know_when_to_replace_it_with_Concepts.md
- Implement SFINAE-friendly and concept-constrained class interfaces — https://github.com/Dr-Sergey/learn_cpp/blob/main/29_OOP_Design/Implement_SFINAE-friendly_and_concept-constrained_class_interfaces.md