# C++ Concepts and Constraints (C++20) — Comprehensive Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
Concepts are named sets of requirements on template arguments, evaluated as boolean predicates at compile time, that constrain template parameters and provide dramatically improved diagnostics when requirements are not satisfied.

**Technical Definition**
Concepts and constraints (C++20, ISO/IEC 14882:2020, [temp.concept], [temp.constr]) are a language feature that allows the specification of constraints on template arguments, which can be used to select the most appropriate function overloads and template specializations. Named sets of such requirements are called concepts. Each concept is a predicate, evaluated at compile time, and becomes part of the interface of a template where it is used as a constraint.

A constraint is a sequence of logical operations and operands that specifies requirements on template arguments. Constraints can be introduced through:
- A **requires clause** after the template parameter list
- A **type-constraint** in the template parameter list
- A **requires expression** inside a concept definition or a requires clause

**Beginner-Friendly Explanation**
Before C++20, if you wrote a template function that only worked with numbers, and someone called it with a string, the compiler would produce a wall of cryptic errors deep inside the template implementation. Concepts let you say, right in the function signature, "this function only works with types that support addition." If someone tries to use it with the wrong type, the compiler tells them clearly: "This type does not satisfy the `Addable` concept." Concepts are like type-level contracts: they make the requirements of your templates explicit, readable, and enforceable.

---

### Key Characteristics

- **Boolean predicates**: Each concept is a compile-time predicate that evaluates to `true` or `false`.
- **Named requirements**: Concepts give a name to a set of requirements, improving readability and reusability.
- **Constraint clauses**: The `requires` clause can be used in template parameter lists, after function parameter lists, or inside concept definitions.
- **Multiple syntax forms**: Concepts can be applied through standard syntax, terse syntax (`std::integral auto`), or trailing syntax.
- **Layered composition**: Concepts can be composed using `&&`, `||`, and `!` to build complex constraints from simpler ones.
- **Diagnostic improvement**: Concepts replace SFINAE-based constraints with clear, human-readable error messages.
- **No runtime overhead**: All constraint checking is done at compile time.

---

### Prerequisites

- Solid understanding of function templates and class templates.
- Familiarity with template parameter lists, deduction, and instantiation.
- Basic knowledge of SFINAE and `std::enable_if` (to appreciate the improvement).
- A C++20-compliant compiler: GCC 10+, Clang 10+, or MSVC 2019 16.3+.

---

### Related Programming Areas

- **Template metaprogramming** (compile-time computation with types).
- **SFINAE** (Substitution Failure Is Not An Error) and `std::enable_if` (the pre-C++20 approach).
- **Overload resolution** (concepts participate in overload resolution and partial ordering).
- **Standard library concepts** (`std::integral`, `std::floating_point`, `std::copyable`, `std::invocable`, and the ranges concepts).
- **C++20 ranges** (heavily concept-constrained).

---

### Core Concepts / Features

1. **Boolean Predicates** — Defining compile-time type expectations via the `concept` keyword.
2. **The Constraint Clause** — Writing complex structural evaluations using `requires` clauses (nested, compound, and type requirements).
3. **Grammatical Styling** — Applying constraints through standard, terse, or trailing syntax formats.
4. **Diagnostic Resolution** — Evaluating how Concepts systematically replace unreadable compiler errors with clean constraint-failure diagnostics.

---

## 1. Boolean Predicates: Defining Concepts

### Definitions

**Core Definition**
A concept is a named set of requirements, declared with the `concept` keyword, that is evaluated as a boolean predicate at compile time. It defines what a type must support to be used with a constrained template.

**Technical Definition**
A concept is defined with the syntax `template< template-parameter-list > concept concept-name = constraint-expression;`. The definition must appear at namespace scope. A concept is a compile-time predicate: it evaluates to `true` if the template arguments satisfy all requirements in the constraint-expression, and `false` otherwise. Concepts cannot be explicitly instantiated, explicitly specialized, or partially specialized; the meaning of the original definition of a constraint cannot be changed. A concept can take multiple template parameters and can constrain a contextually deduced type.

**Beginner-Friendly Explanation**
A concept is like a job description for a type. If you write `template <typename T> concept Addable = requires(T a, T b) { a + b; };`, you're saying: "A type is `Addable` if you can add two values of that type together." Then, when you write `template <Addable T> T sum(T a, T b) { return a + b; }`, the compiler checks every time someone calls `sum` whether their type satisfies the `Addable` requirement. If not, you get a clear error message instead of a cryptic failure.

---

### Purposes (all begin with "To")

- **To** define named, reusable requirements for template parameters that can be applied across multiple templates.
- **To** express the semantic intent of a template's requirements in a readable, self-documenting way.
- **To** enable the compiler to produce clear, actionable error messages when requirements are not met.
- **To** participate in overload resolution and partial ordering, allowing more specialized templates to be selected.
- **To** replace complex SFINAE expressions with concise, declarative code.

---

### Syntax Rules and Structure

#### Complete General Syntax with Breakdown

```cpp
template <typename T>
concept ConceptName = constraint-expression;
```

**Component breakdown:**
- `template <typename T>` — template parameter list for the concept.
- `concept` — keyword introducing the concept definition.
- `ConceptName` — identifier for the concept (becomes a concept-name).
- `= constraint-expression` — the boolean expression defining the requirements.

#### Syntax Rules

- The concept definition must appear at namespace scope.
- The constraint-expression must be a constant expression of type `bool`.
- Concepts cannot be explicitly instantiated, explicitly specialized, or partially specialized.
- The concept name can be used in `requires` clauses, type-constraints, and as a boolean predicate in other contexts.
- A concept can be used in an `id-expression` (e.g., `ConceptName<T>::value` is not valid; use `ConceptName<T>` directly).

#### Constraints and Limitations

- Concepts cannot have default template arguments in the concept definition (the template parameters are those of the concept itself).
- A concept cannot be declared inside a class or function scope.
- Concepts cannot be redeclared or redefined.
- The name of a concept is not a type; it cannot be used as a template parameter for a type.

---

### Multiple Annotated Complete Step-by-Step Code Examples and Their Expected Outputs

#### Example 1 — Basic Concept Definition and Usage

```cpp
#include <iostream>
#include <concepts>

// Step 1: Define a concept named Addable
template <typename T>
concept Addable = requires(T a, T b) {
    { a + b } -> std::convertible_to<T>;
};

// Step 2: Apply the concept to a function template
template <Addable T>
T add(T a, T b) {
    return a + b;
}

int main() {
    // Step 3: int satisfies Addable
    std::cout << "add(3, 4) = " << add(3, 4) << std::endl;

    // Step 4: double satisfies Addable
    std::cout << "add(2.5, 3.5) = " << add(2.5, 3.5) << std::endl;

    // Step 5: This would fail with a clear error message:
    // struct NoAdd {};
    // add(NoAdd{}, NoAdd{});  // Error: NoAdd does not satisfy Addable

    return 0;
}
```

**Expected Output:**
```
add(3, 4) = 7
add(2.5, 3.5) = 6
```

**Why this output:** The concept `Addable` checks that `a + b` is a valid expression and that its result is convertible to `T`. Both `int` and `double` satisfy this requirement. The function template `add` is instantiated only for types that satisfy `Addable`. If a type without `operator+` were used, the compiler would produce a clear "constraint not satisfied" error at the call site.

---

#### Example 2 — Composing Concepts

```cpp
#include <iostream>
#include <concepts>
#include <string>

// Step 1: Simple concepts
template <typename T>
concept Integral = std::is_integral_v<T>;

template <typename T>
concept Signed = std::is_signed_v<T>;

// Step 2: Compose concepts with &&
template <typename T>
concept SignedIntegral = Integral<T> && Signed<T>;

// Step 3: Apply the composed concept
template <SignedIntegral T>
T absolute(T x) {
    return (x < 0) ? -x : x;
}

int main() {
    std::cout << "absolute(-5) = " << absolute(-5) << std::endl;
    std::cout << "absolute(3) = " << absolute(3) << std::endl;

    // unsigned int is Integral but not Signed
    // absolute(5u);  // Error: unsigned int does not satisfy SignedIntegral

    return 0;
}
```

**Expected Output:**
```
absolute(-5) = 5
absolute(3) = 3
```

**Why this output:** The concept `SignedIntegral` is composed from `Integral` and `Signed` using the `&&` operator. `int` satisfies both, so `absolute(-5)` and `absolute(3)` compile. `unsigned int` satisfies `Integral` but not `Signed`, so attempting to call `absolute(5u)` produces a compile error with a clear message identifying which sub-requirement failed.

---

### Real-World Cases with Explanation

**Case 1 — Standard library concepts:** The C++20 standard library provides fundamental concepts in `<concepts>` (`std::integral`, `std::floating_point`, `std::copyable`, `std::movable`) and in `<iterator>` (`std::input_iterator`, `std::random_access_iterator`). These are used throughout the ranges library and other standard components.

**Case 2 — Ranges library:** The `<ranges>` header defines concepts like `std::ranges::range`, `std::ranges::view`, and `std::ranges::sized_range`. Algorithms like `std::ranges::sort` are constrained to `std::ranges::random_access_range`, preventing accidental use with non-random-access containers like `std::list`.

**Case 3 — Library API design:** Libraries use concepts to make their interfaces self-documenting. A function template `template <Serializable T> void save(const T& obj, std::ostream& os)` immediately tells the caller that `T` must provide a `serialize` method and a static `type_name` method.

**Case 4 — Compile-time polymorphism:** Concepts enable a form of compile-time polymorphism where different overloads are selected based on type properties, without the boilerplate of SFINAE.

---

### References

- Constraints and concepts (since C++20) — cppreference.com - https://en.cppreference.com/w/cpp/language/constraints
- Concepts library — cppreference.com - https://en.cppreference.com/w/cpp/concepts
- ISO/IEC TS 19217:2015 (Concepts TS) — ISO - https://www.iso.org/standard/64030.html
- Stroustrup P0557r1 Concepts — WG21 - https://rap.no/JTC1/SC22/WG21/docs/papers/2018/p0557r1.pdf
- C++ Core Guidelines T.20: Avoid concepts without meaningful semantics - https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rt-concepts

---

## 2. The Constraint Clause: `requires` Expressions

### Definitions

**Core Definition**
A `requires` expression is an expression of type `bool` that describes constraints on template arguments. It is used inside concept definitions and `requires` clauses to specify what expressions must be valid, what types must exist, and what properties must hold.

**Technical Definition**
A `requires` expression has the form `requires (parameter-list) { requirement-seq }` where `parameter-list` is optional. The result is a prvalue of type `bool`. Requirements can reference template parameters, parameters from the parameter-list, and other declarations visible from the enclosing context. If the substitution of template arguments into a `requires` expression results in an invalid type or expression, the `requires` expression evaluates to `false` rather than causing a hard error (SFINAE-friendly). There are four kinds of requirements: **simple requirements**, **type requirements**, **compound requirements**, and **nested requirements**.

**Beginner-Friendly Explanation**
A `requires` expression is like a checklist that the compiler runs through. You can say "I require that `a + b` is valid" (simple requirement), "I require that `T::value_type` exists" (type requirement), "I require that `a.size()` returns something convertible to `size_t` and doesn't throw" (compound requirement), or "I require that `std::is_integral_v<T>` is true" (nested requirement). The compiler checks each item; if all pass, the `requires` expression is `true`; if any fails, it's `false`.

---

### Purposes (all begin with "To")

- **To** specify simple expressions that must be valid for a type to satisfy a concept.
- **To** require the existence of nested types (type requirements).
- **To** require that an expression returns a specific type or is `noexcept` (compound requirements).
- **To** assert additional boolean conditions on template parameters (nested requirements).
- **To** provide a structured, readable way to express complex constraints.

---

### Syntax Rules and Structure

#### Complete General Syntaxes with Breakdowns

**Syntax 1 — Simple requirement**
```cpp
requires (T a, T b) {
    a + b;  // the expression "a + b" must be valid
}
```
- A simple requirement asserts that the expression is valid.

**Syntax 2 — Type requirement**
```cpp
requires (T t) {
    typename T::value_type;  // T must have a nested type value_type
}
```
- A type requirement asserts that the specified type is valid.

**Syntax 3 — Compound requirement**
```cpp
requires (T a, T b) {
    { a + b } -> std::convertible_to<T>;  // a + b must be valid and convertible to T
    { a.size() } noexcept -> std::same_as<std::size_t>;  // also noexcept
}
```
- A compound requirement combines an expression, an optional `noexcept` specification, and an optional trailing return-type constraint.

**Syntax 4 — Nested requirement**
```cpp
requires (T t) {
    requires std::is_integral_v<T>;  // nested requirement: bool expression must be true
}
```
- A nested requirement asserts that a boolean expression evaluates to `true`.

#### Syntax Rules

- The parameter list in a `requires` expression is optional; if omitted, the requirements can still reference template parameters.
- Local parameters in the parameter list have no linkage, storage, or lifetime; they are only used for notation.
- The `noexcept` specifier in a compound requirement is checked at compile time.
- The trailing return-type constraint in a compound requirement uses `std::convertible_to` or similar concepts.
- Nested requirements use the `requires` keyword followed by a boolean expression.

#### Constraints and Limitations

- A `requires` expression must appear inside a template (or inside a concept definition).
- Substitution failure in a `requires` expression yields `false`; it does not cause a hard error.
- If a `requires` expression contains invalid types or expressions and is not inside a templated entity, the program is ill-formed.
- Compound requirements cannot have both a `noexcept` specifier and a trailing return-type constraint that is not a concept.

---

### Multiple Annotated Complete Step-by-Step Code Examples and Their Expected Outputs

#### Example 1 — Simple, Type, and Compound Requirements

```cpp
#include <iostream>
#include <concepts>
#include <string>
#include <vector>

// Step 1: Concept with simple, type, and compound requirements
template <typename T>
concept Container = requires(T t) {
    // Type requirement: T must have a nested type value_type
    typename T::value_type;

    // Simple requirement: t.size() must be valid
    t.size();

    // Compound requirement: t.begin() must return something convertible to an iterator
    { t.begin() } -> std::input_or_output_iterator;

    // Compound requirement: t.empty() must be noexcept and return bool
    { t.empty() } noexcept -> std::convertible_to<bool>;
};

// Step 2: Apply the concept
template <Container C>
void describe(const C& c) {
    std::cout << "Container with " << c.size() << " elements" << std::endl;
}

int main() {
    // Step 3: std::vector satisfies Container
    std::vector<int> vec = {1, 2, 3};
    describe(vec);

    // Step 4: std::string also satisfies Container
    std::string str = "hello";
    describe(str);

    // Step 5: int does not satisfy Container (no value_type, no begin())
    // describe(42);  // Error: int does not satisfy Container

    return 0;
}
```

**Expected Output:**
```
Container with 3 elements
Container with 5 elements
```

**Why this output:** The concept `Container` requires a nested `value_type`, a `size()` method, a `begin()` method returning an iterator, and an `empty()` method that is `noexcept` and returns a `bool`. Both `std::vector<int>` and `std::string` satisfy all requirements. `int` lacks `value_type` and `begin()`, so it fails the concept with a clear diagnostic indicating which requirement was not satisfied.

---

#### Example 2 — Nested Requirements and Concept Composition

```cpp
#include <iostream>
#include <concepts>

// Step 1: Base concept
template <typename T>
concept Numeric = std::integral<T> || std::floating_point<T>;

// Step 2: Concept with nested requirement
template <typename T>
concept PositiveNumeric = Numeric<T> && requires(T t) {
    requires (t > T{0});  // nested requirement: must be able to compare with zero
};

// Step 3: Apply the concept
template <PositiveNumeric T>
T safe_divide(T numerator, T denominator) {
    if (denominator == T{0}) {
        std::cerr << "Division by zero!" << std::endl;
        return T{0};
    }
    return numerator / denominator;
}

int main() {
    std::cout << "safe_divide(10, 2) = " << safe_divide(10, 2) << std::endl;
    std::cout << "safe_divide(7.5, 2.5) = " << safe_divide(7.5, 2.5) << std::endl;

    return 0;
}
```

**Expected Output:**
```
safe_divide(10, 2) = 5
safe_divide(7.5, 2.5) = 3
```

**Why this output:** The `PositiveNumeric` concept combines `Numeric` with a nested requirement that `t > T{0}` is valid. Both `int` and `double` satisfy this. The nested requirement ensures that the type supports comparison with a default-constructed value, which is necessary for the `safe_divide` function's zero check.

---

### Real-World Cases with Explanation

**Case 1 — Iterator concepts:** The `<iterator>` header defines concepts like `std::input_iterator`, `std::forward_iterator`, and `std::random_access_iterator` using compound requirements that check operations like `++it`, `*it`, and `it + n`. These concepts are used throughout the ranges library.

**Case 2 — Range concepts:** `std::ranges::range` requires that `ranges::begin(t)` and `ranges::end(t)` are valid, `std::ranges::sized_range` additionally requires `ranges::size(t)`, and `std::ranges::contiguous_range` requires that the iterator is contiguous.

**Case 3 — Serialization frameworks:** A `Serializable` concept can use compound requirements to check that `t.serialize(os)` returns `void` and that `T::type_name()` returns something convertible to `std::string`.

**Case 4 — Hashable concept:** `std::hash` specializations can be constrained by a concept that requires `std::hash<T>{}(a)` to be valid and convertible to `std::size_t`.

---

### References

- Requires expression (since C++20) — cppreference.com - https://en.cppreference.com/w/cpp/language/requires
- Constraints and concepts: Requires clauses — cppreference.com - https://en.cppreference.com/w/cpp/language/constraints
- Understand `requires` Expressions and the `requires` Clause (C++20) — Dr-Sergey/learn_cpp (GitHub) - https://github.com/Dr-Sergey/learn_cpp
- C++20 Concepts: Testing Constrained Functions — ACCU - https://accu.org/journals/overload/31/173/patel/

---

## 3. Grammatical Styling: Standard, Terse, and Trailing Syntax

### Definitions

**Core Definition**
Grammatical styling refers to the three syntactic forms for applying constraints to templates: **standard syntax** (requires clause after the template parameter list), **terse syntax** (type-constraint in the parameter list), and **trailing syntax** (requires clause after the function parameter list).

**Technical Definition**
C++20 provides multiple syntactic forms for constraining templates. The **standard form** places a `requires` clause immediately after the template parameter list: `template <typename T> requires Concept<T>`. The **terse form** uses a type-constraint in place of `typename` or `class` in the template parameter list: `template <Concept T>` or in abbreviated function templates: `void f(Concept auto x)`. The **trailing form** places the `requires` clause after the function parameter list: `template <typename T> void f(T x) requires Concept<T>`. All three forms are semantically equivalent.

**Beginner-Friendly Explanation**
C++20 gives you three ways to say the same thing: "this template requires `T` to satisfy `Concept`." The standard way is to write `requires Concept<T>` after the template parameters. The terse way is to write `template <Concept T>` — it's shorter and more readable. The trailing way is to write `requires Concept<T>` after the function's parameter list. You can choose whichever style you find clearest.

---

### Purposes (all begin with "To")

- **To** provide multiple syntactic forms for applying constraints, accommodating different coding styles.
- **To** enable terse, readable constraints in abbreviated function templates.
- **To** allow trailing constraints for functions with complex return types or parameter lists.
- **To** support concept application in class templates, function templates, and member functions.
- **To** improve code readability by placing constraints close to the entities they constrain.

---

### Syntax Rules and Structure

#### Complete General Syntaxes with Breakdowns

**Syntax 1 — Standard syntax (requires clause after template parameter list)**
```cpp
template <typename T>
    requires std::integral<T>
T add(T a, T b) { return a + b; }
```
- The `requires` clause follows the template parameter list.

**Syntax 2 — Terse syntax (type-constraint in template parameter list)**
```cpp
template <std::integral T>
T add(T a, T b) { return a + b; }
```
- The concept replaces `typename` or `class`.

**Syntax 3 — Terse syntax in abbreviated function template**
```cpp
void add(std::integral auto a, std::integral auto b) {
    return a + b;
}
```
- The concept qualifies `auto` in the function parameter list.

**Syntax 4 — Trailing syntax (requires clause after function parameter list)**
```cpp
template <typename T>
T add(T a, T b) requires std::integral<T> {
    return a + b;
}
```
- The `requires` clause follows the function parameter list.

**Syntax 5 — Combined standard and terse**
```cpp
template <typename T>
    requires std::integral<T>
void f(T x) requires std::copyable<T> {
    // both requires clauses apply
}
```
- Multiple `requires` clauses can be combined (they are AND-ed).

#### Syntax Rules

- All three forms are semantically equivalent; the choice is stylistic.
- The terse form cannot be used with concepts that take multiple template arguments in some contexts; use the standard form for clarity.
- The trailing form is useful when the function has a trailing return type.
- In a class template, the terse form can be used for template parameters: `template <std::integral T> class Box`.
- A type-constraint can appear in place of `typename` or `class` in any template parameter list.

#### Constraints and Limitations

- The terse form with `auto` (abbreviated function templates) requires C++20.
- The standard form with `requires` after the template parameter list is the most explicit and is preferred when the constraint is complex.
- Trailing `requires` clauses cannot be used on constructors in some older compilers; check compiler support.
- Mixing multiple constraint syntaxes can reduce readability; choose one style per codebase.

---

### Multiple Annotated Complete Step-by-Step Code Examples and Their Expected Outputs

#### Example 1 — Comparing the Three Syntax Forms

```cpp
#include <iostream>
#include <concepts>

// Step 1: Standard syntax
template <typename T>
    requires std::integral<T>
T add_standard(T a, T b) { return a + b; }

// Step 2: Terse syntax (template parameter)
template <std::integral T>
T add_terse(T a, T b) { return a + b; }

// Step 3: Terse syntax (abbreviated function template)
auto add_abbreviated(std::integral auto a, std::integral auto b) {
    return a + b;
}

// Step 4: Trailing syntax
template <typename T>
T add_trailing(T a, T b) requires std::integral<T> {
    return a + b;
}

int main() {
    // Step 5: All four forms produce the same result
    std::cout << "Standard: " << add_standard(3, 4) << std::endl;
    std::cout << "Terse: " << add_terse(3, 4) << std::endl;
    std::cout << "Abbreviated: " << add_abbreviated(3, 4) << std::endl;
    std::cout << "Trailing: " << add_trailing(3, 4) << std::endl;
    return 0;
}
```

**Expected Output:**
```
Standard: 7
Terse: 7
Abbreviated: 7
Trailing: 7
```

**Why this output:** All four syntax forms apply the `std::integral` constraint to the template parameter `T`. They are semantically identical; the compiler generates the same code for each. The choice among them is purely stylistic, and each has readability trade-offs depending on context.

---

#### Example 2 — Constraints on Class Templates

```cpp
#include <iostream>
#include <concepts>
#include <vector>

// Step 1: Terse syntax on a class template
template <std::integral T>
class SafeCounter {
public:
    void increment() { ++count_; }
    T get() const { return count_; }
private:
    T count_{};
};

// Step 2: Standard syntax on a class template
template <typename T>
    requires std::floating_point<T>
class FloatWrapper {
public:
    FloatWrapper(T value) : value_(value) {}
    T value() const { return value_; }
private:
    T value_;
};

int main() {
    // Step 3: SafeCounter only accepts integral types
    SafeCounter<int> counter;
    counter.increment();
    counter.increment();
    std::cout << "Counter: " << counter.get() << std::endl;

    // Step 4: FloatWrapper only accepts floating-point types
    FloatWrapper<double> fw(3.14);
    std::cout << "Float: " << fw.value() << std::endl;

    // SafeCounter<double> bad;  // Error: double does not satisfy std::integral
    // FloatWrapper<int> bad2;   // Error: int does not satisfy std::floating_point

    return 0;
}
```

**Expected Output:**
```
Counter: 2
Float: 3.14
```

**Why this output:** The `SafeCounter` class template uses the terse syntax `template <std::integral T>`, so only integral types can be used. The `FloatWrapper` class template uses the standard syntax `template <typename T> requires std::floating_point<T>`, so only floating-point types can be used. Both approaches produce clear constraints on the class template parameters.

---

### Real-World Cases with Explanation

**Case 1 — Standard library containers:** The ranges library uses terse syntax extensively, e.g., `template <std::ranges::range R> void process(R&& range)`. This makes the constraints immediately visible in the function signature.

**Case 2 — Abbreviated function templates:** C++20 allows `void print(std::integral auto x)` as a shorthand for a constrained function template. This is particularly useful for generic lambdas and simple utility functions.

**Case 3 — Complex constraints:** For constraints that combine multiple concepts with `&&` and `||`, the standard syntax with a `requires` clause is often clearer than the terse form, especially when the constraint spans multiple lines.

**Case 4 — Trailing constraints for return types:** When a function has a trailing return type (`auto f() -> decltype(...)`), placing the constraint after the parameter list (`requires Concept<T>`) avoids ambiguity.

---

### References

- Constraints and concepts: Constraint syntax — cppreference.com - https://en.cppreference.com/w/cpp/language/constraints
- Abbreviated function templates — cppreference.com - https://en.cppreference.com/w/cpp/language/function_template#Abbreviated_function_template
- C++20 Concepts — Microsoft Learn - https://learn.microsoft.com/en-us/cpp/cpp/concepts-cpp

---

## 4. Diagnostic Resolution: Clean Error Messages

### Definitions

**Core Definition**
Diagnostic resolution is the improvement in compiler error messages that results from using concepts: instead of deep, nested template instantiation errors, the compiler reports exactly which concept requirement was not satisfied, at the call site.

**Technical Definition**
When a constrained template is instantiated with arguments that do not satisfy the constraints, the compiler produces a diagnostic message that identifies the unsatisfied concept and, ideally, the specific atomic constraint that failed. This is in contrast to SFINAE-based constraints, where substitution failure deep inside a template instantiation stack produces "no matching function" errors with many candidate template ignored notes. Concepts make the requirements explicit in the template's interface, allowing the compiler to report constraint failure at the point of use rather than deep in the implementation.

**Beginner-Friendly Explanation**
Before concepts, if you used a template with the wrong type, you would get a page of errors that looked like the compiler was speaking a foreign language. With concepts, the compiler says something like: "Error: `std::list<int>` does not satisfy `RandomAccessIterator`, so `std::sort` cannot be called." This tells you exactly what went wrong and where. Concepts turn a cryptic puzzle into a clear message.

---

### Purposes (all begin with "To")

- **To** replace unreadable SFINAE error messages with clear, human-readable constraint-failure diagnostics.
- **To** identify exactly which requirement a type failed to satisfy.
- **To** report errors at the call site rather than deep inside template implementation details.
- **To** improve the developer experience when working with generic code.
- **To** make template requirements self-documenting through the compiler's own diagnostics.

---

### Syntax Rules and Structure

#### Diagnostic Comparison: SFINAE vs. Concepts

| Aspect | SFINAE (pre-C++20) | Concepts (C++20) |
|---|---|---|
| Error location | Deep inside template instantiation | At the call site |
| Error message | "no matching function for call to 'sort'" + "candidate template ignored: substitution failure..." | "error: cannot call std::sort with std::_List_iterator<int>" + "note: concept RandomAccessIterator<std::_List_iterator<int>> was not satisfied" |
| Readability | Low; requires expertise to decode | High; states the failed requirement directly |
| Diagnostic depth | Many nested notes, often 50+ lines | Concise; one or two notes |
| Actionability | Requires understanding of template internals | Directly identifies what needs to change |

#### Syntax Rules for Good Diagnostics

- Use **named concepts** rather than inline `requires` expressions; named concepts produce better diagnostics.
- **Layer concepts** incrementally: a concept that composes `HasSize`, `HasIterators`, and `HasPushBack` will report which sub-concept failed.
- Avoid deeply nested `requires` expressions; each layer of nesting adds diagnostic noise.
- Use `static_assert` with concepts for additional compile-time validation with custom messages.

#### Constraints and Limitations

- Diagnostic quality varies by compiler; MSVC, GCC, and Clang have different levels of detail.
- Very complex concept hierarchies can still produce verbose diagnostics, though they are generally better than SFINAE.
- The `requires` expression's substitution failure behavior (yielding `false` rather than a hard error) is essential for concept diagnostics to work correctly.

---

### Multiple Annotated Complete Step-by-Step Code Examples and Their Expected Outputs

#### Example 1 — SFINAE vs. Concepts: Diagnostic Comparison

```cpp
// BEFORE (SFINAE): cryptic error messages
#include <type_traits>
#include <iostream>

template <typename T,
          typename = std::enable_if_t<std::is_arithmetic_v<T>>>
T old_add(T a, T b) {
    return a + b;
}

// AFTER (Concepts): clear error messages
#include <concepts>

template <std::integral T>
T new_add(T a, T b) {
    return a + b;
}

int main() {
    // Both work for int
    std::cout << old_add(3, 4) << std::endl;
    std::cout << new_add(3, 4) << std::endl;

    // Attempt with std::string:
    // old_add(std::string("a"), std::string("b"));
    // Error (SFINAE): "no matching function for call to 'old_add'"
    //   "candidate template ignored: requirement 'std::is_arithmetic_v<std::string>' was not satisfied"
    //   "note: candidate function template not viable"

    // new_add(std::string("a"), std::string("b"));
    // Error (Concepts): "error: 'std::string' does not satisfy 'integral'"
    //   "note: because 'std::string' does not satisfy 'is_integral_v'"

    return 0;
}
```

**Expected Output (for valid calls):**
```
7
7
```

**Why this matters:** The SFINAE version produces a "no matching function" error that requires the developer to parse the `enable_if` condition to understand what went wrong. The concept version produces a direct statement: "`std::string` does not satisfy `integral`." The concept diagnostic is shorter, clearer, and actionable.

---

#### Example 2 — Layered Concepts for Clear Diagnostics

```cpp
#include <iostream>
#include <concepts>
#include <ranges>
#include <vector>
#include <list>

// Step 1: Layered concepts
template <typename T>
concept HasSize = requires(const T& t) {
    { t.size() } -> std::convertible_to<std::size_t>;
};

template <typename T>
concept HasBeginEnd = requires(T& t) {
    { t.begin() } -> std::input_or_output_iterator;
    { t.end() } -> std::sentinel_for<decltype(t.begin())>;
};

template <typename T>
concept SimpleContainer = HasSize<T> && HasBeginEnd<T>;

// Step 2: Constrained function
template <SimpleContainer C>
void process(const C& container) {
    std::cout << "Processing container of size "
              << container.size() << std::endl;
}

int main() {
    // Step 3: std::vector satisfies SimpleContainer
    std::vector<int> vec = {1, 2, 3};
    process(vec);

    // Step 4: std::list also satisfies SimpleContainer
    std::list<int> lst = {4, 5};
    process(lst);

    // Step 5: A type without size() fails with a clear message:
    // struct NoSize {};
    // process(NoSize{});  // Error: NoSize does not satisfy HasSize

    return 0;
}
```

**Expected Output:**
```
Processing container of size 3
Processing container of size 2
```

**Why this output:** The `SimpleContainer` concept is built from `HasSize` and `HasBeginEnd`. If a type lacks `size()`, the compiler reports "`NoSize` does not satisfy `HasSize`" rather than the more general "`NoSize` does not satisfy `SimpleContainer`." This layered approach makes diagnostics more precise and actionable.

---

### Real-World Cases with Explanation

**Case 1 — Ranges library diagnostics:** Attempting to call `std::ranges::sort` on a `std::list` produces a diagnostic like "error: cannot call `std::ranges::sort` with `std::_List_iterator<int>`; note: concept `RandomAccessIterator<std::_List_iterator<int>>` was not satisfied." This is dramatically clearer than the pre-C++20 SFINAE error.

**Case 2 — Library API documentation:** Concepts serve as self-documenting constraints. A function signature like `template <Serializable T> void save(const T& obj, std::ostream& os)` tells the caller exactly what is required without needing to read the implementation or documentation.

**Case 3 — Compile-time validation with `static_assert`:** Combining concepts with `static_assert` provides even better diagnostics: `static_assert(Serializable<MyType>, "MyType must be serializable");` produces a clear message when the assertion fails.

**Case 4 — GCC concept diagnostics improvements:** GCC has improved concept diagnostics by setting precise locations for constraint expressions and printing unsatisfied atomic constraint expressions with their template arguments, making it easier to identify exactly which requirement failed.

---

### References

- C++20 Concepts for Nicer Compiler Errors — Daniel Lemire, ISOCPP - https://isocpp.org/blog/2025/06/cpp20-concepts-for-nicer-compiler-errors-daniel-lemire
- Write Concepts-constrained library interfaces with good diagnostics — Dr-Sergey/learn_cpp (GitHub) - https://github.com/Dr-Sergey/learn_cpp
- Constraints and concepts: Diagnostics — cppreference.com - https://en.cppreference.com/w/cpp/language/constraints
- Concepts Error Messages for Humans — WG21 P2429R0 - https://wg21.tartanllama.xyz
- Use `static_assert` with concept-based diagnostics for better error messages — Dr-Sergey/learn_cpp (GitHub) - https://github.com/Dr-Sergey/learn_cpp

---

## Summary Table of Core Concepts

| Concept | Key Mechanism | Primary Benefit | Key Constraint |
|---|---|---|---|
| Boolean Predicates | `template <typename T> concept C = ...` | Named, reusable, self-documenting requirements | Must appear at namespace scope; cannot be specialized |
| Constraint Clause | `requires(...) { ... }` | Structured expression of simple, type, compound, and nested requirements | Requires C++20; must appear inside a template |
| Grammatical Styling | Standard, terse, and trailing syntax | Flexibility in expressing constraints | Terse form with `auto` requires C++20 |
| Diagnostic Resolution | Clear constraint-failure messages | Dramatically improved error messages over SFINAE | Diagnostic quality varies by compiler |

---

## References (Consolidated)

- Constraints and concepts (since C++20) — cppreference.com - https://en.cppreference.com/w/cpp/language/constraints
- Requires expression (since C++20) — cppreference.com - https://en.cppreference.com/w/cpp/language/requires
- Concepts library — cppreference.com - https://en.cppreference.com/w/cpp/concepts
- C++20 Concepts for Nicer Compiler Errors — Daniel Lemire, ISOCPP - https://isocpp.org/blog/2025/06/cpp20-concepts-for-nicer-compiler-errors-daniel-lemire
- Write Concepts-constrained library interfaces with good diagnostics — Dr-Sergey/learn_cpp (GitHub) - https://github.com/Dr-Sergey/learn_cpp
- ISO/IEC TS 19217:2015 (Concepts TS) — ISO - https://www.iso.org/standard/64030.html
- Stroustrup P0557r1 Concepts — WG21 - https://rap.no/JTC1/SC22/WG21/docs/papers/2018/p0557r1.pdf
- C++ Core Guidelines T.20: Avoid concepts without meaningful semantics - https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rt-concepts
- Understand `requires` Expressions and the `requires` Clause (C++20) — Dr-Sergey/learn_cpp (GitHub) - https://github.com/Dr-Sergey/learn_cpp
- C++20 Concepts: Testing Constrained Functions — ACCU - https://accu.org/journals/overload/31/173/patel/
- Use `static_assert` with concept-based diagnostics for better error messages — Dr-Sergey/learn_cpp (GitHub) - https://github.com/Dr-Sergey/learn_cpp
- Concepts Error Messages for Humans — WG21 P2429R0 - https://wg21.tartanllama.xyz