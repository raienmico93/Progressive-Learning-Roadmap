# Function Templates — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
A function template is a C++ construct that defines a **family of functions** parameterized by one or more template parameters, from which the compiler generates concrete functions when template arguments are deduced or explicitly provided.

**Technical Definition**
A function template is a template that defines a family of functions. It is introduced by the `template` keyword followed by a template parameter list and a function declaration/definition. When the template is called, the compiler performs **template argument deduction** to determine the template arguments from the function arguments (or uses explicitly provided arguments), then **instantiates** a concrete function specialization. Function templates participate in **overload resolution** alongside non-template functions and other templates. They can be **specialized** (full specialization only; function templates cannot be partially specialized) and **overloaded** with other function templates and non-template functions.

**Beginner-Friendly Explanation**
A function template is a **generic function recipe**. Instead of writing `max_int`, `max_double`, and `max_string`, you write **one template** `template <typename T> T max(T a, T b)`. When you call `max(3, 7)`, the compiler figures out `T = int` and generates an `int` version. When you call `max(2.5, 1.5)`, it generates a `double` version. The compiler **deduces** the types from your arguments, so you usually don't have to write them explicitly. Function templates are the foundation of generic programming in C++ and power much of the standard library.

### Key Characteristics

- **Generic functions**: One definition works with many types.
- **Template argument deduction**: The compiler deduces template arguments from function arguments.
- **Overload resolution**: Function templates compete with non-templates and other templates.
- **Instantiation**: Concrete functions are generated on demand (implicitly or explicitly).
- **Full specialization**: Customize behavior for specific types.
- **No partial specialization**: Function templates cannot be partially specialized (use overloading instead).
- **Header definitions**: Function template definitions must typically be visible at the point of instantiation.
- **Zero-overhead**: No runtime cost compared to hand-written functions.

### Prerequisites

- Understanding of functions, parameters, and return types.
- Familiarity with templates and template parameters.
- Knowledge of overloading and overload resolution.
- Basic awareness of type deduction and value categories.

### Related Programming Areas

- **Generic programming** – the paradigm function templates enable.
- **Standard library algorithms** – `std::sort`, `std::find`, `std::transform`.
- **Class templates** – member functions of class templates are often function templates.
- **Perfect forwarding** – forwarding references in templates.
- **Concepts (C++20)** – constrained function templates.
- **Template metaprogramming** – compile-time computation.

### Core Concepts / Features

1. Function Templates
2. Template Parameter Deduction
3. Template Argument Deduction
4. Overload Resolution
5. Template Specialization
6. Template Function Overloading
7. Function Template Best Practices

---

## 1. Function Templates

### Definitions

**Core Definition**
A function template defines a family of functions parameterized by template parameters, from which the compiler generates concrete functions.

**Technical Definition**
A function template is a template that defines a family of functions. It is declared with `template < parameter-list >` followed by a function declaration or definition. When called, template arguments are deduced from the function arguments (or explicitly provided), and the compiler instantiates the corresponding function specialization. Function templates can be overloaded with other function templates and with non-template functions.

**Beginner-Friendly Explanation**
A function template is a **generic function**. It works with many types because the compiler generates a version for each type you use.

### Purposes

- To write type-generic functions without duplication.
- To support generic algorithms and utilities.
- To enable zero-overhead abstraction.
- To provide type-safe generic interfaces.

### Syntax Rules and Structure

**General Syntax**
```cpp
template <typename T>
return_type function_name(parameters) {
    // body
}
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `template` | Keyword introducing the template. |
| `<typename T>` | Template parameter list. |
| `return_type function_name(...)` | Function using `T`. |

**Syntax Rules**
- `typename` and `class` are interchangeable for type parameters.
- Template arguments can be deduced from function arguments.
- Explicit arguments can be provided: `max<int>(a, b)`.
- The template definition must typically be visible at the point of use.

**Constraints and Limitations**
- All template parameters must normally appear in the function parameter list (unless deduced elsewhere).
- The type must support the operations used in the function body.
- Function templates cannot be partially specialized.

### Annotated Code Examples

**Example 1: Basic function template**

```cpp
#include <iostream>

template <typename T>
T max(T a, T b) {
    return a > b ? a : b;
}

int main() {
    std::cout << max(3, 7) << '\n';       // T = int
    std::cout << max(2.5, 1.5) << '\n';   // T = double
}
```

**Expected Output**
```
7
2.5
```

**Why this output?**  
The compiler deduces `T = int` for the first call and `T = double` for the second, generating two specializations.

**Example 2: Function template with multiple parameters**

```cpp
#include <iostream>

template <typename T, typename U>
auto add(T a, U b) -> decltype(a + b) {
    return a + b;
}

int main() {
    std::cout << add(2, 3.5) << '\n';   // int + double -> double
}
```

**Expected Output**
```
5.5
```

**Why this output?**  
`T = int`, `U = double`; the return type is deduced via `decltype(a + b)` as `double`.

### Real-World Cases

- **`std::swap`** – swaps any two objects of the same type.
- **`std::min` / `std::max`** – generic comparisons.
- **`std::find`** – searches any container.
- **`std::make_unique<T>`** – factory for any type.

---

## 2. Template Parameter Deduction

### Definitions

**Core Definition**
Template parameter deduction is the process by which the compiler determines the template arguments from the function arguments during a call.

**Technical Definition**
When a function template is called, the compiler performs **template argument deduction**: it matches the function argument types against the parameter types in the template's function parameter list, deducing the template arguments. Deduction follows specific rules based on the form of the parameter (e.g., `T`, `T&`, `const T&`, `T&&`). If deduction fails or is ambiguous, the template is not viable.

**Beginner-Friendly Explanation**
Template parameter deduction is the compiler **figuring out the template arguments** from the types of the arguments you pass. You don't have to write `max<int>(3, 7)` — the compiler deduces `T = int` from the arguments.

### Purposes

- To allow natural, concise calls without explicit template arguments.
- To support generic algorithms that adapt to argument types.
- To enable perfect forwarding via forwarding references.

### Syntax Rules and Structure

**Deduction Forms**

| Parameter Form | Deduction |
|----------------|-----------|
| `T` | By value: deduces the decayed type. |
| `T&` | By lvalue reference: deduces the exact type. |
| `const T&` | By const reference: deduces the type. |
| `T&&` | Forwarding reference: deduces lvalue/rvalue. |
| `T*` | Pointer: deduces the pointed-to type. |

**Syntax Rules**
- Deduction is based on the function argument types.
- `T&&` with deduced `T` is a forwarding reference.
- Arrays and functions decay to pointers unless deduced as references.
- Explicit arguments override deduction for those parameters.

**Constraints and Limitations**
- Deduction can fail for ambiguous or unsupported forms.
- Template parameters not appearing in the function parameter list cannot be deduced (unless defaulted).
- `std::initializer_list` deduction requires special handling.

### Annotated Code Examples

**Example 1: Deduction with references**

```cpp
#include <iostream>

template <typename T>
void show(T& value) {
    std::cout << value << '\n';
}

int main() {
    int x = 42;
    show(x);    // T = int
    // show(42); // error: cannot bind non-const lvalue ref to rvalue
}
```

**Expected Output**
```
42
```

**Why this output?**  
`T&` deduces `T = int` from the lvalue `x`. An rvalue would not bind to `T&`.

**Example 2: Forwarding reference deduction**

```cpp
#include <iostream>
#include <string>
#include <utility>

template <typename T>
void forward_to(T&& value) {
    std::cout << value << '\n';
}

int main() {
    std::string s = "hello";
    forward_to(s);              // T = std::string& (lvalue)
    forward_to(std::string("world")); // T = std::string (rvalue)
}
```

**Expected Output**
```
hello
world
```

**Why this output?**  
`T&&` is a forwarding reference; `T` is deduced as `std::string&` for the lvalue and `std::string` for the rvalue.

### Real-World Cases

- **`std::forward`** – relies on forwarding reference deduction.
- **`std::make_unique<T>(args...)`** – deduces constructor arguments.
- **Generic algorithms** – deduce element types.

---

## 3. Template Argument Deduction

### Definitions

**Core Definition**
Template argument deduction is the broader process of determining all template arguments (not just deduced ones) for a function template call, combining deduction and explicit arguments.

**Technical Definition**
Template argument deduction for function templates determines the template arguments from the function arguments and any explicitly provided arguments. The process considers the template parameter list, the function parameter types, and the argument types. Deduction is performed for each parameter; explicit arguments are used where provided. If deduction fails for any parameter, the template is not viable.

**Beginner-Friendly Explanation**
Template argument deduction is the **full process of figuring out all the template arguments**, whether by deduction from arguments or from explicit specification.

### Purposes

- To determine the template arguments for a call.
- To enable overload resolution to consider templates.
- To support mixed explicit and deduced arguments.

### Syntax Rules and Structure

**General Form**
```cpp
template <typename T, typename U>
void f(T a, U b);
f(1, 2.0);          // T = int, U = double
f<int>(1, 2.0);     // T = int (explicit), U = double (deduced)
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| Deduced arguments | Inferred from function arguments. |
| Explicit arguments | Provided in angle brackets. |

**Syntax Rules**
- Explicit arguments are matched to template parameters left-to-right.
- Remaining parameters are deduced.
- Deduction failure makes the template non-viable.

**Constraints and Limitations**
- You cannot explicitly specify some parameters and leave earlier ones deduced (until C++20 concepts improve this).
- Deduction failure is not an error if other overloads are viable (SFINAE).

### Annotated Code Examples

**Example 1: Mixed explicit and deduced**

```cpp
#include <iostream>

template <typename T, typename U>
void show(T a, U b) {
    std::cout << a << ' ' << b << '\n';
}

int main() {
    show(1, 2.5);          // T = int, U = double
    show<int>(1, 2.5);     // T = int (explicit), U = double
}
```

**Expected Output**
```
1 2.5
1 2.5
```

**Why this output?**  
Explicit `T = int` and deduced `U = double` produce the same call.

### Real-World Cases

- **Partial explicit arguments** – specifying some template arguments.
- **SFINAE** – deduction failure removes candidates.

---

## 4. Overload Resolution

### Definitions

**Core Definition**
Overload resolution is the process by which the compiler selects the best-matching function (template or non-template) for a call.

**Technical Definition**
Overload resolution for function templates builds a candidate set of functions (non-templates, templates with deduced arguments, and explicit specializations), trims it to viable functions, and selects the best match by ranking implicit conversion sequences. Non-template functions are preferred over template specializations when all else is equal. More specialized templates are preferred over less specialized ones.

**Beginner-Friendly Explanation**
Overload resolution is the compiler **picking the best function** when multiple candidates exist. Non-templates beat templates when they're equally good; more specialized templates beat less specialized ones.

### Purposes

- To select the most appropriate function for a call.
- To allow templates and non-templates to coexist.
- To support customization through specialization.

### Syntax Rules and Structure

**Candidate Set**
- Non-template functions with the same name.
- Function templates with deduced arguments.
- Explicit specializations.

**Ranking**
1. Exact match > promotion > conversion > user-defined conversion.
2. Non-template preferred over template.
3. More specialized template preferred over less specialized.

**Syntax Rules**
- Templates participate in overload resolution after deduction.
- Explicit specializations are considered.
- SFINAE removes invalid candidates.

**Constraints and Limitations**
- Ambiguity arises when no unique best match exists.
- Template vs. non-template preference can be surprising.

### Annotated Code Examples

**Example 1: Non-template preferred**

```cpp
#include <iostream>

void f(int) { std::cout << "non-template\n"; }

template <typename T>
void f(T) { std::cout << "template\n"; }

int main() {
    f(42);   // non-template preferred
}
```

**Expected Output**
```
non-template
```

**Why this output?**  
Both candidates are exact matches, but the non-template is preferred.

### Real-World Cases

- **`std::sort`** – overloads with and without comparators.
- **Customization points** – non-template overloads for specific types.

---

## 5. Template Specialization

### Definitions

**Core Definition**
Template specialization is the provision of a custom implementation for a specific set of template arguments, overriding the primary template.

**Technical Definition**
Function templates support **full specialization** (explicit specialization) only: `template <> return_type f<int>(...)`. Partial specialization is not allowed for function templates; use overloading instead. Specializations must be declared before first use.

**Beginner-Friendly Explanation**
Specialization is **customizing a template for a specific type**. For function templates, you can only fully specialize (no partial specialization).

### Purposes

- To customize behavior for specific types.
- To optimize specific cases.
- To handle types with special semantics.

### Syntax Rules and Structure

**Full Specialization**
```cpp
template <>
return_type f<int>(parameters) {
    // custom
}
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `template <>` | Full specialization marker. |
| `f<int>` | Specific arguments. |

**Syntax Rules**
- Only full specialization is allowed for function templates.
- Must be declared before first use.
- Cannot partially specialize (use overloading).

**Constraints and Limitations**
- Partial specialization is ill-formed for function templates.
- Specializations must match the primary template's interface.

### Annotated Code Examples

**Example 1: Full specialization**

```cpp
#include <iostream>
#include <string>

template <typename T>
T add(T a, T b) { return a + b; }

template <>
std::string add<std::string>(std::string a, std::string b) {
    return a + " " + b;
}

int main() {
    std::cout << add(1, 2) << '\n';
    std::cout << add<std::string>("hello", "world") << '\n';
}
```

**Expected Output**
```
3
hello world
```

**Why this output?**  
The full specialization for `std::string` inserts a space.

### Real-World Cases

- **`std::hash`** – specialized for types (class template, but similar principle).
- **Custom serialization** – specialized for specific types.

---

## 6. Template Function Overloading

### Definitions

**Core Definition**
Template function overloading is the practice of defining multiple function templates (or templates and non-templates) with the same name but different parameter lists.

**Technical Definition**
Function templates can be overloaded with other function templates and with non-template functions. Overload resolution selects the best match. More specialized templates are preferred over less specialized ones. Non-templates are preferred over templates when all else is equal.

**Beginner-Friendly Explanation**
Template function overloading is **having multiple templates (or templates and regular functions) with the same name**, and the compiler picks the best one.

### Purposes

- To provide multiple generic interfaces.
- To customize behavior for different parameter counts or types.
- To combine templates with non-template overloads.

### Syntax Rules and Structure

**General Form**
```cpp
template <typename T>
void f(T);

template <typename T>
void f(T, T);

void f(int);   // non-template overload
```

**Component Breakdown**

| Form | Description |
|------|-------------|
| Template overloads | Different parameter lists. |
| Non-template overloads | Compete with templates. |

**Syntax Rules**
- Overloads must differ in parameter lists.
- More specialized templates are preferred.
- Non-templates are preferred over templates when equally good.

**Constraints and Limitations**
- Ambiguity can arise.
- Partial specialization is not allowed; use overloading instead.

### Annotated Code Examples

**Example 1: Template overloading**

```cpp
#include <iostream>

template <typename T>
void print(T value) { std::cout << "one: " << value << '\n'; }

template <typename T, typename U>
void print(T a, U b) { std::cout << "two: " << a << ' ' << b << '\n'; }

int main() {
    print(42);
    print(1, 2.5);
}
```

**Expected Output**
```
one: 42
two: 1 2.5
```

**Why this output?**  
Overload resolution selects the one-argument or two-argument template based on the call.

### Real-World Cases

- **`std::make_unique`** – overloads for arrays and single objects.
- **`std::to_string`** – overloads for numeric types.

---

## 7. Function Template Best Practices

### Definitions

**Core Definition**
Function template best practices are guidelines for writing function templates that are correct, readable, efficient, and maintainable.

**Technical Definition**
The C++ Core Guidelines provide template rules (T.1–T.13, T.40–T.83). Key principles: use templates to express algorithms that work on many argument types (T.2); use function templates to deduce argument types (T.10); specify concepts for template parameters (T.10, C++20); use `auto` and `decltype` carefully; avoid over-templating non-generic code; and constrain templates with concepts where possible.

**Beginner-Friendly Explanation**
Best practices for function templates are simple: **use templates for truly generic code**, **let the compiler deduce arguments**, **constrain templates with concepts**, **avoid over-templating**, and **keep error messages readable**.

### Key Guidelines (Summary Table)

| Guideline | Summary | Recommendation |
|-----------|---------|----------------|
| **Use for generic code** | Templates are for families of types. | Don't template one-off functions. |
| **Constrain with concepts** | Concepts (C++20) express requirements. | `template <std::integral T>`. |
| **Deduce arguments** | Let the compiler deduce. | `max(a, b)` not `max<int>(a, b)`. |
| **Avoid over-templating** | Over-templating hurts readability and errors. | Use concrete types when possible. |
| **Header definitions** | Templates typically belong in headers. | Header-only libraries. |
| **Use `auto` carefully** | `auto` return types can hide types. | Prefer explicit or `decltype(auto)`. |
| **Specialize sparingly** | Full specialization is a commitment. | Document customizations. |

### Annotated Code Examples

**Example 1: Constrained template (C++20)**

```cpp
#include <iostream>
#include <concepts>

template <std::integral T>
T add(T a, T b) {
    return a + b;
}

int main() {
    std::cout << add(1, 2) << '\n';
    // add(1.5, 2.5);   // error: double is not integral
}
```

**Expected Output**
```
3
```

**Why this is best practice?**  
The `std::integral` concept clearly expresses the requirement and produces a readable error at the call site.

### Real-World Cases

- **Standard library** – templates constrained with concepts in C++20.
- **Code review** – concepts and clear constraints are preferred.
- **Static analysis** – tools check template usage.

---

## References

- Function templates (cppreference) – https://en.cppreference.com/w/cpp/language/function_template
- Templates (cppreference) – https://en.cppreference.com/w/cpp/language/templates
- Template argument deduction (cppreference) – https://en.cppreference.com/w/cpp/language/template_argument_deduction
- Overload resolution (cppreference) – https://en.cppreference.com/w/cpp/language/overload_resolution
- C++ Core Guidelines (templates section) – https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines
- Microsoft Learn — Function templates (C++) – https://learn.microsoft.com/en-us/cpp/cpp/function-templates
- Microsoft Learn — Templates (C++) – https://learn.microsoft.com/en-us/cpp/cpp/templates-cpp
- cppreference — Explicit specialization – https://en.cppreference.com/w/cpp/language/template_specialization
- cppreference — Partial specialization – https://en.cppreference.com/w/cpp/language/partial_specialization

---

This cheat sheet provides a complete, structured reference for C++ function templates, covering function templates, template parameter deduction, template argument deduction, overload resolution, template specialization, template function overloading, and best practices. Each section follows a uniform template with definitions, purposes, syntax rules, annotated examples, expected outputs, and authoritative references.