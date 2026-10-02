# C++ Function Templates — Comprehensive Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
A function template is a blueprint that defines a family of functions. It is not a function itself; no code is generated from the template until it is instantiated with specific template arguments.

**Technical Definition**
A function template is a C++ entity parameterized by one or more template parameters (type, non-type, template, or parameter pack). It is declared using the syntax `template <parameter-list> function-declaration`. When a function template specialization is referenced, all template arguments must have values; these values can be explicitly specified, deduced from the call arguments, or obtained from default template arguments. Template argument deduction takes place after the function template name lookup and before overload resolution.

**Beginner-Friendly Explanation**
Think of a function template as a cookie cutter. The cookie cutter itself is not a cookie—it's a shape that can produce many cookies, each with a different flavor (type). For example, a function template for `add` can work with `int`, `double`, or even custom types. You write the recipe once, and the compiler generates the appropriate version for each type you use.

---

### Key Characteristics

- **Blueprint generation**: Defines a family of functions, not a single function.
- **Type parameterization**: Template parameters can be types (`typename T` / `class T`), non-types (`int N`), templates, or parameter packs.
- **Automatic deduction**: The compiler can infer template arguments from function call arguments.
- **Code generation on demand**: No code is generated until the template is instantiated.
- **Overload participation**: Function templates participate in overload resolution alongside non-template functions.
- **Abbreviated syntax (C++20)**: `void foo(auto x)` is shorthand for a template with an invented type parameter.

---

### Prerequisites

- Basic C++ syntax: functions, parameters, return types, and scope.
- Understanding of types, references, and `const` qualifiers.
- Familiarity with the compilation pipeline (templates are compiled at instantiation time).
- (For C++20 features) A compiler supporting C++20 abbreviated function templates and concepts.

---

### Related Programming Areas

- **Class templates** (templates for classes).
- **Template argument deduction** (how the compiler infers types).
- **Overload resolution** (how the compiler chooses between candidates).
- **Concepts and constraints** (C++20, restricting template parameters).
- **SFINAE** (Substitution Failure Is Not An Error) and `std::enable_if`.

---

### Core Concepts / Features

1. **Blueprint Generation** — Declaring function templates with type parameters (`typename` vs. `class`).
2. **Deduction Engine** — Automatic inference of template arguments and C++20 abbreviated function templates (`void foo(auto x)`).
3. **Parameter Handling** — Managing multiple parameters, trailing return types, and default template arguments.
4. **Overloading Behavior** — Priority hierarchies between template functions, specialized templates, and non-template functions.

---

## 1. Blueprint Generation

### Definitions

**Core Definition**
Blueprint generation is the declaration of a function template using type parameters, which defines a family of functions rather than a single concrete function.

**Technical Definition**
A function template is declared with `template <parameter-list> function-declaration`. The parameter list is a non-empty comma-separated list of template parameters, each of which is either a constant parameter (non-type), a type parameter, a template parameter, or a parameter pack of any of those (C++11). Type parameters are introduced with either the `typename` or `class` keyword; there is no semantic difference between them in a template parameter declaration.

**Beginner-Friendly Explanation**
When you write `template <typename T> T add(T a, T b)`, you're creating a template—a blueprint. The `T` is a placeholder for a type. Later, when you call `add(3, 4)`, the compiler replaces `T` with `int` and generates a concrete function. The `typename` keyword (or `class`, which is interchangeable) tells the compiler that `T` is a type parameter.

---

### Purposes (all begin with "To")

- **To** define a single algorithm that works with many different types.
- **To** avoid code duplication when the same logic applies to multiple types.
- **To** enable generic programming, where algorithms are decoupled from specific data types.
- **To** support type-safe code generation with compile-time type checking.
- **To** provide a foundation for template metaprogramming and library design.

---

### Syntax Rules and Structure

#### Complete General Syntaxes with Breakdowns

**Syntax 1 — Basic function template**
```cpp
template <typename T>
T add(T a, T b) {
    return a + b;
}
```
**Component breakdown:**
- `template` — keyword introducing a template declaration.
- `<typename T>` — template parameter list; `T` is a type parameter.
- `T add(T a, T b)` — function signature using the type parameter.
- `{ return a + b; }` — function body.

**Syntax 2 — Using `class` instead of `typename`**
```cpp
template <class T>
T add(T a, T b) { return a + b; }
```
- `class` and `typename` are semantically equivalent in this context.

**Syntax 3 — Non-type template parameter**
```cpp
template <typename T, int N>
T sum_array(T (&arr)[N]) {
    T result{};
    for (int i = 0; i < N; ++i) result += arr[i];
    return result;
}
```
- `int N` — a non-type parameter; the value is provided at compile time.

**Syntax 4 — Template parameter pack (C++11)**
```cpp
template <typename... Ts>
void print_all(Ts... args) { /* ... */ }
```
- `Ts...` — a parameter pack that can accept zero or more type arguments.

#### Syntax Rules

- The `template` keyword must precede the function declaration.
- Template parameters are declared inside `< >`.
- `typename` and `class` are interchangeable for type parameters.
- The function name declared becomes a template name.
- Function templates cannot have default template arguments in C++03; C++11 and later allow them.

#### Constraints and Limitations

- Function templates cannot be partially specialized (only class templates can).
- Function template definitions must be visible at the point of instantiation (typically in headers).
- Template parameters cannot be deduced from the return type alone.
- The `export` keyword for templates was removed in C++11.

---

### Multiple Annotated Complete Step-by-Step Code Examples and Their Expected Outputs

#### Example 1 — Basic Function Template

```cpp
#include <iostream>
#include <string>

// Step 1: Declare a function template
template <typename T>
T max_value(T a, T b) {
    return (a > b) ? a : b;
}

int main() {
    // Step 2: Instantiate with int
    std::cout << "max(3, 7) = " << max_value(3, 7) << std::endl;

    // Step 3: Instantiate with double
    std::cout << "max(3.14, 2.71) = " << max_value(3.14, 2.71) << std::endl;

    // Step 4: Instantiate with std::string
    std::cout << "max(\"apple\", \"banana\") = "
              << max_value(std::string("apple"), std::string("banana"))
              << std::endl;

    // Step 5: Explicit template argument
    std::cout << "max<double>(5, 3.5) = "
              << max_value<double>(5, 3.5) << std::endl;

    return 0;
}
```

**Expected Output:**
```
max(3, 7) = 7
max(3.14, 2.71) = 3.14
max("apple", "banana") = banana
max<double>(5, 3.5) = 5
```

**Why this output:** The template `max_value` is instantiated with `T = int`, `T = double`, and `T = std::string` based on the call arguments. The explicit instantiation `max_value<double>(5, 3.5)` forces `T = double`, so `5` is converted to `5.0` and compared with `3.5`. In all cases, the compiler generates a concrete function from the blueprint.

---

#### Example 2 — `typename` vs. `class` (Equivalence)

```cpp
#include <iostream>

// Step 1: Using typename
template <typename T>
T square_typename(T x) { return x * x; }

// Step 2: Using class (semantically identical)
template <class T>
T square_class(T x) { return x * x; }

int main() {
    std::cout << "square_typename(5) = " << square_typename(5) << std::endl;
    std::cout << "square_class(5) = " << square_class(5) << std::endl;
    return 0;
}
```

**Expected Output:**
```
square_typename(5) = 25
square_class(5) = 25
```

**Why this output:** Both templates produce identical results because `typename` and `class` are interchangeable in template parameter declarations. Modern style guides prefer `typename` because it more clearly indicates that the parameter is a type, not necessarily a class.

---

### Real-World Cases with Explanation

**Case 1 — Standard library algorithms:** `std::sort`, `std::find`, and `std::transform` are function templates that work with any iterator type and any comparison function, enabling generic algorithms across containers.

**Case 2 — Container adaptors:** `std::make_pair`, `std::make_shared`, and `std::make_unique` are function templates that deduce the types of the objects they create, providing a safer alternative to manual type specification.

**Case 3 — Mathematical libraries:** A `clamp` function template that restricts a value between a minimum and maximum can be instantiated for `int`, `float`, `double`, or any comparable type.

**Case 4 — Serialization frameworks:** A `serialize` function template can generate serialization code for any type that provides the required operations, eliminating per-type boilerplate.

---

### References

- Function template — cppreference.com - https://en.cppreference.com/w/cpp/language/function_template
- Template parameters — cppreference.com - https://en.cppreference.com/w/cpp/language/template_parameters
- CWG Issue 1874: `typename` vs. `class` — https://cplusplus.github.io/CWG/issues/1874.html

---

## 2. Deduction Engine

### Definitions

**Core Definition**
The deduction engine is the compiler mechanism that automatically infers template arguments from the types of function call arguments, allowing callers to omit explicit template arguments.

**Technical Definition**
Template argument deduction takes place after the function template name lookup (which may involve argument-dependent lookup) and before overload resolution. For each function parameter that uses a template parameter, the compiler compares the parameter type with the argument type and deduces the template argument. If deduction succeeds for all parameters, the template is instantiated with the deduced arguments. If explicit template arguments are provided, deduction is not performed for those parameters.

**Beginner-Friendly Explanation**
When you call `add(3, 4)`, the compiler looks at the arguments (`3` and `4`, both `int`) and figures out that `T` should be `int`. This is deduction: the compiler "deduces" the template argument from the function call. You don't have to write `add<int>(3, 4)` unless you want to override the deduction or resolve ambiguity. C++20 makes this even simpler with abbreviated function templates: `void foo(auto x)` is shorthand for a template with a deduced type parameter.

---

### Purposes (all begin with "To")

- **To** allow callers to use templates without explicitly specifying template arguments.
- **To** reduce boilerplate and improve code readability.
- **To** enable generic algorithms to work seamlessly with different argument types.
- **To** support perfect forwarding and type deduction in generic contexts.
- **To** simplify template syntax with C++20 abbreviated function templates.

---

### Syntax Rules and Structure

#### Complete General Syntaxes with Breakdowns

**Syntax 1 — Deduction from function call**
```cpp
template <typename T>
void print(T value);

print(42);        // T deduced as int
print(3.14);      // T deduced as double
print("hello");   // T deduced as const char*
```

**Syntax 2 — Abbreviated function template (C++20)**
```cpp
void print(auto value) {
    std::cout << value << std::endl;
}
```
- `auto` in a parameter list creates an invented template parameter.
- Equivalent to `template <typename T> void print(T value)`.

**Syntax 3 — Constrained abbreviated template (C++20)**
```cpp
void print(std::integral auto value) {
    std::cout << value << std::endl;
}
```
- `std::integral auto` — the parameter is constrained to integral types.
- Equivalent to `template <std::integral T> void print(T value)`.

#### Syntax Rules

- Deduction occurs for each template parameter that is not explicitly specified.
- The deduced type must be consistent across all parameters for the same template parameter.
- Deduction considers implicit conversions (e.g., `int` to `long`).
- Deduction does not consider the return type.
- Abbreviated function templates are themselves function templates and can be specialized.

#### Constraints and Limitations

- Deduction fails if the argument type cannot be matched to the parameter type.
- Ambiguous deductions (e.g., `template <typename T> void f(T, T)` called with `f(1, 2.0)`) cause compilation errors.
- Deduction cannot deduce template parameters from braced-init-lists without additional context.
- Abbreviated function templates require C++20; not all compilers support them equally.

---

### Multiple Annotated Complete Step-by-Step Code Examples and Their Expected Outputs

#### Example 1 — Deduction from Function Arguments

```cpp
#include <iostream>
#include <typeinfo>

template <typename T>
void show_type(T value) {
    std::cout << "Type: " << typeid(T).name() << std::endl;
}

int main() {
    // Step 1: Deduce int
    show_type(42);

    // Step 2: Deduce double
    show_type(3.14);

    // Step 3: Deduce const char*
    show_type("hello");

    // Step 4: Explicit template argument overrides deduction
    show_type<double>(42);  // T is double, 42 converted to 42.0

    return 0;
}
```

**Expected Output (implementation-defined type names):**
```
Type: i
Type: d
Type: PKc
Type: d
```

**Why this output:** The compiler deduces `T` from each call argument. `42` is `int` (mangled as `i`), `3.14` is `double` (`d`), and `"hello"` is `const char*` (`PKc`). When `show_type<double>(42)` is called, the explicit template argument forces `T = double`, and `42` is converted to `42.0`.

---

#### Example 2 — C++20 Abbreviated Function Template

```cpp
#include <iostream>
#include <concepts>

// Step 1: Abbreviated function template with auto
void print_auto(auto value) {
    std::cout << "auto: " << value << std::endl;
}

// Step 2: Abbreviated template with concept constraint
void print_integral(std::integral auto value) {
    std::cout << "integral: " << value << std::endl;
}

int main() {
    // Step 3: auto accepts any type
    print_auto(42);
    print_auto(3.14);
    print_auto("hello");

    // Step 4: std::integral auto only accepts integral types
    print_integral(42);
    // print_integral(3.14);  // ERROR: double does not satisfy std::integral

    return 0;
}
```

**Expected Output:**
```
auto: 42
auto: 3.14
auto: hello
integral: 42
```

**Why this output:** `print_auto(auto value)` is an abbreviated function template; the compiler creates an invented template parameter for `auto`. `print_integral(std::integral auto value)` constrains that parameter to types satisfying `std::integral`. Calling `print_integral(3.14)` would produce a compile error because `double` does not satisfy the constraint.

---

### Real-World Cases with Explanation

**Case 1 — `std::make_pair`:** Deduces the types of the pair from its arguments: `std::make_pair(1, 2.0)` deduces `std::pair<int, double>`.

**Case 2 — `std::make_shared`:** Deduces the type of the shared pointer from the constructor arguments, eliminating the need for explicit type specification.

**Case 3 — Range-based algorithms:** C++20 ranges algorithms use abbreviated function templates and concepts to accept any range type without explicit template arguments.

**Case 4 — Generic lambdas:** Lambda expressions with `auto` parameters are effectively abbreviated function templates, enabling generic lambdas without explicit template syntax.

---

### References

- Template argument deduction — cppreference.com - https://en.cppreference.com/w/cpp/language/template_argument_deduction
- Abbreviated function templates — cppreference.com - https://en.cppreference.com/w/cpp/language/function_template#Abbreviated_function_template
- CWG Issue 2447: Abbreviated function templates — https://cplusplus.github.io/CWG/issues/2447.html

---

## 3. Parameter Handling

### Definitions

**Core Definition**
Parameter handling refers to the management of multiple template parameters, the specification of trailing return types (for return types that depend on template parameters), and the use of default template arguments.

**Technical Definition**
Function templates can have multiple template parameters of different kinds (type, non-type, template, parameter pack). When the return type depends on the template parameters and cannot be deduced (e.g., in `decltype` expressions), a trailing return type is used: `auto function_name(parameters) -> return-type`. Default template arguments for function templates were introduced in C++11; they allow template parameters to have default values that are used when the argument is not explicitly specified or deduced.

**Beginner-Friendly Explanation**
Sometimes a function template needs more than one type parameter, or its return type depends on the types of its arguments. For example, `add(T a, U b)` might return whatever type `a + b` produces. The trailing return type syntax `auto add(T a, U b) -> decltype(a + b)` lets you say "the return type is whatever `a + b` gives." Default template arguments let you provide a fallback type, like `template <typename T = int>`, so callers can omit the type if they want the default.

---

### Purposes (all begin with "To")

- **To** handle multiple template parameters of different kinds in a single function template.
- **To** specify a return type that depends on template parameters using trailing return types.
- **To** provide default template arguments for parameters that are not deduced or explicitly specified.
- **To** enable perfect forwarding and type-dependent computations in generic code.
- **To** support non-type template parameters for compile-time constants.

---

### Syntax Rules and Structure

#### Complete General Syntaxes with Breakdowns

**Syntax 1 — Multiple type parameters**
```cpp
template <typename T, typename U>
auto add(T a, U b) -> decltype(a + b) {
    return a + b;
}
```
- `typename T, typename U` — two type parameters.
- `-> decltype(a + b)` — trailing return type; the return type is deduced from `a + b`.

**Syntax 2 — Trailing return type (C++11)**
```cpp
template <typename T, typename U>
auto multiply(T a, U b) -> decltype(a * b) {
    return a * b;
}
```
- The return type appears after the parameter list, using `->`.
- Required when the return type depends on the parameters and cannot be deduced by `auto` alone.

**Syntax 3 — Default template argument (C++11)**
```cpp
template <typename T = int>
T get_default() {
    return T{};
}
```
- `= int` — default template argument; used when `T` is not specified or deduced.

**Syntax 4 — Non-type parameter with default**
```cpp
template <typename T, int N = 10>
T sum_array(T (&arr)[N]) {
    T result{};
    for (int i = 0; i < N; ++i) result += arr[i];
    return result;
}
```
- `int N = 10` — non-type parameter with a default value.

#### Syntax Rules

- Template parameters can be mixed: type, non-type, template, and parameter packs.
- Default template arguments must appear after parameters that have no default.
- Trailing return types are required when the return type depends on parameters and cannot be deduced by `auto`.
- C++14 allows `auto` return type deduction without a trailing return type for simple cases.
- Non-type template arguments must be constant expressions.

#### Constraints and Limitations

- Default template arguments for function templates are only used when the parameter is not deduced (C++11 and later).
- Trailing return types cannot be used with `auto` return type deduction simultaneously.
- Non-type template parameters cannot be floating-point or class types (until C++20 for some cases).
- The return type in a trailing return type must be valid for all possible template arguments.

---

### Multiple Annotated Complete Step-by-Step Code Examples and Their Expected Outputs

#### Example 1 — Multiple Parameters with Trailing Return Type

```cpp
#include <iostream>
#include <string>

// Step 1: Multiple type parameters with trailing return type
template <typename T, typename U>
auto add(T a, U b) -> decltype(a + b) {
    return a + b;
}

int main() {
    // Step 2: int + int = int
    std::cout << "add(3, 4) = " << add(3, 4) << std::endl;

    // Step 3: int + double = double
    std::cout << "add(3, 4.5) = " << add(3, 4.5) << std::endl;

    // Step 4: std::string + std::string = std::string
    std::cout << "add(\"Hello, \", \"World!\") = "
              << add(std::string("Hello, "), std::string("World!"))
              << std::endl;

    return 0;
}
```

**Expected Output:**
```
add(3, 4) = 7
add(3, 4.5) = 7.5
add("Hello, ", "World!") = Hello, World!
```

**Why this output:** The trailing return type `decltype(a + b)` computes the type of `a + b` for each instantiation. For `int` and `int`, it is `int`. For `int` and `double`, it is `double` (the result of the usual arithmetic conversions). For `std::string` and `std::string`, it is `std::string` (the result of `operator+`). Without the trailing return type, the compiler could not know the return type at the point of declaration.

---

#### Example 2 — Default Template Arguments

```cpp
#include <iostream>
#include <string>

// Step 1: Default template argument for T
template <typename T = int>
T zero() {
    return T{};
}

// Step 2: Default non-type parameter for N
template <typename T, int N = 3>
T sum_first() {
    T result{};
    for (int i = 1; i <= N; ++i) result += static_cast<T>(i);
    return result;
}

int main() {
    // Step 3: Use default T = int
    std::cout << "zero() = " << zero() << std::endl;

    // Step 4: Override default T
    std::cout << "zero<double>() = " << zero<double>() << std::endl;

    // Step 5: Use default N = 3
    std::cout << "sum_first<int>() = " << sum_first<int>() << std::endl;

    // Step 6: Override default N
    std::cout << "sum_first<int, 5>() = " << sum_first<int, 5>() << std::endl;

    return 0;
}
```

**Expected Output:**
```
zero() = 0
zero<double>() = 0
sum_first<int>() = 6
sum_first<int, 5>() = 15
```

**Why this output:** `zero()` uses the default `T = int`, returning `0`. `zero<double>()` overrides the default, returning `0.0` (printed as `0`). `sum_first<int>()` uses the default `N = 3`, summing `1 + 2 + 3 = 6`. `sum_first<int, 5>()` overrides `N` to 5, summing `1 + 2 + 3 + 4 + 5 = 15`.

---

### Real-World Cases with Explanation

**Case 1 — Perfect forwarding:** `std::forward<T>(arg)` uses a type parameter `T` deduced from the argument to preserve value categories during forwarding.

**Case 2 — Type traits:** `std::enable_if<B, T = void>` uses a default template argument to conditionally enable or disable function templates via SFINAE.

**Case 3 — Variable templates:** `std::is_same_v<T, U>` is a variable template that uses two type parameters and a default (the `_v` suffix convention).

**Case 4 — Matrix dimensions:** A matrix template `template <typename T, int Rows, int Cols>` uses non-type parameters for dimensions, enabling compile-time size checking and stack allocation.

---

### References

- Trailing return type — cppreference.com - https://en.cppreference.com/w/cpp/language/function#Return_type
- Default template arguments — cppreference.com - https://en.cppreference.com/w/cpp/language/template_parameters#Default_template_arguments
- CWG Issue 226: Default template arguments for function templates — https://cplusplus.github.io/CWG/issues/226.html

---

## 4. Overloading Behavior

### Definitions

**Core Definition**
Overloading behavior describes how the compiler chooses between a non-template function, a function template specialization, and an explicit specialization when multiple candidates with the same name are visible.

**Technical Definition**
When a function template and a non-template function have the same name, the compiler first attempts to resolve the call by using template argument deduction to instantiate the function template with a unique specialization. If deduction succeeds, the generated function is compared with the non-template function and other candidates using the standard overload resolution rules. If a non-template function is an equally good match to a function template, the non-template function is chosen (unless template arguments were explicitly specified). If the non-template function requires a conversion that the template does not, the template is preferred.

**Beginner-Friendly Explanation**
Imagine you have two tools for the same job: a general-purpose tool (the template) and a specialized tool (the non-template function). When you call the function, the compiler tries both. If the non-template function is just as good as the template, the non-template wins—it's more specialized. But if the non-template requires extra work (like converting an argument), the template might win because it's a better fit.

---

### Purposes (all begin with "To")

- **To** allow non-template functions to coexist with templates of the same name.
- **To** provide optimized or specialized implementations for specific types.
- **To** enable explicit specialization of function templates for particular types.
- **To** control which function is chosen when multiple candidates are viable.
- **To** support the "prefer non-template over template" rule for overload resolution.

---

### Syntax Rules and Structure

#### Complete General Syntaxes with Breakdowns

**Syntax 1 — Non-template function overloading a template**
```cpp
template <typename T>
void f(T x) { std::cout << "template" << std::endl; }

void f(int x) { std::cout << "non-template" << std::endl; }
```
- `f(42)` calls the non-template `f(int)` because it is an equally good match.
- `f(3.14)` calls the template with `T = double`.

**Syntax 2 — Explicit specialization**
```cpp
template <>
void f<int>(int x) { std::cout << "explicit specialization" << std::endl; }
```
- Explicit specialization for `T = int`.
- Called when `f<int>(x)` is invoked or when `f(42)` is called and the non-template is absent.

**Syntax 3 — Overload resolution priority**
```cpp
// 1. Exact non-template match (highest priority)
// 2. Exact template match
// 3. Non-template requiring conversion
// 4. Template requiring conversion (lowest priority)
```

#### Syntax Rules

- Overload resolution considers both template and non-template candidates.
- A non-template function is preferred over a template if both are equally good matches.
- A template that produces an exact match is preferred over a non-template that requires a conversion.
- Explicit template arguments (`f<int>(x)`) bypass the "prefer non-template" rule.
- Explicit specializations are not considered during overload resolution; they are selected after a template is chosen.

#### Constraints and Limitations

- Function templates cannot be partially specialized; only fully specialized.
- Overloading with templates can lead to ambiguity if the template and non-template are equally good matches and the non-template is not selected by the preference rule.
- The presence of a non-template function can hide a template from a different namespace unless using-declarations are employed.
- Explicit specializations must be declared before their first use.

---

### Multiple Annotated Complete Step-by-Step Code Examples and Their Expected Outputs

#### Example 1 — Non-Template vs. Template Overload

```cpp
#include <iostream>

// Step 1: Function template
template <typename T>
void f(T x) {
    std::cout << "template: " << x << std::endl;
}

// Step 2: Non-template function with the same name
void f(int x) {
    std::cout << "non-template: " << x << std::endl;
}

int main() {
    // Step 3: Exact match — non-template is preferred
    f(42);        // Calls non-template f(int)

    // Step 4: Template is the only viable match for double
    f(3.14);      // Calls template with T = double

    // Step 5: Explicit template argument forces the template
    f<int>(42);   // Calls template with T = int

    return 0;
}
```

**Expected Output:**
```
non-template: 42
template: 3.14
template: 42
```

**Why this output:** `f(42)` is an exact match for the non-template `f(int)`, so the non-template is chosen. `f(3.14)` cannot match `f(int)` without a conversion, so the template is chosen with `T = double`. `f<int>(42)` explicitly specifies the template argument, which bypasses the non-template preference.

---

#### Example 2 — Template Preferred When Non-Template Requires Conversion

```cpp
#include <iostream>

// Step 1: Non-template function
void g(int x) {
    std::cout << "non-template: " << x << std::endl;
}

// Step 2: Function template
template <typename T>
void g(T x) {
    std::cout << "template: " << x << std::endl;
}

int main() {
    long value = 42;

    // Step 3: Non-template requires long-to-int conversion
    // Template provides an exact match for T = long
    g(value);  // Calls template

    return 0;
}
```

**Expected Output:**
```
template: 42
```

**Why this output:** `value` is `long`. The non-template `g(int)` requires a narrowing conversion from `long` to `int`. The template `g(T)` with `T = long` is an exact match. According to overload resolution rules, the exact match template is preferred over the non-template that requires a conversion.

---

### Real-World Cases with Explanation

**Case 1 — `std::swap`:** The standard library provides a generic `std::swap` template, and users can provide non-template overloads for their own types (found via ADL) that are more efficient. The non-template overload is preferred when it is an equally good match.

**Case 2 — `std::begin` and `std::end`:** The standard library provides both template and non-template overloads for `begin` and `end` to handle arrays, containers, and custom types.

**Case 3 — `operator<<` for output:** A user-defined `operator<<` for a specific type overloads the generic template, allowing custom formatting while falling back to the template for other types.

**Case 4 — `std::to_string`:** The standard library provides overloads for `int`, `long`, `float`, etc., and a template for other types. The overloads are preferred for their specific types.

---

### References

- Overload resolution of function template calls — Microsoft Learn - https://learn.microsoft.com/en-us/cpp/cpp/overload-resolution-of-function-template-calls
- Function template overloading — cppreference.com - https://en.cppreference.com/w/cpp/language/function_template#Function_template_overloading
- CWG Issue 495: Ranking of template vs. non-template — https://cplusplus.github.io/CWG/issues/495.html

---

## Summary Table of Core Concepts

| Concept | Key Mechanism | Primary Benefit | Key Constraint |
|---|---|---|---|
| Blueprint Generation | `template <typename T> function` | Generic algorithms, code reuse | Cannot partially specialize function templates |
| Deduction Engine | Argument type deduction from call | Reduces boilerplate, improves readability | Ambiguous deductions cause errors |
| Parameter Handling | Multiple parameters, trailing return types, defaults | Flexible, type-dependent returns | Default template arguments only used when not deduced |
| Overloading Behavior | Non-template preferred over template | Optimized specializations, ADL customization | Explicit specializations not considered during overload resolution |

---

## References (Consolidated)

- Function template — cppreference.com - https://en.cppreference.com/w/cpp/language/function_template
- Template parameters — cppreference.com - https://en.cppreference.com/w/cpp/language/template_parameters
- Template argument deduction — cppreference.com - https://en.cppreference.com/w/cpp/language/template_argument_deduction
- Overload resolution of function template calls — Microsoft Learn - https://learn.microsoft.com/en-us/cpp/cpp/overload-resolution-of-function-template-calls
- Abbreviated function templates — cppreference.com - https://en.cppreference.com/w/cpp/language/function_template#Abbreviated_function_template
- Trailing return type — cppreference.com - https://en.cppreference.com/w/cpp/language/function#Return_type
- Default template arguments — cppreference.com - https://en.cppreference.com/w/cpp/language/template_parameters#Default_template_arguments
- CWG Issue 1874: `typename` vs. `class` — https://cplusplus.github.io/CWG/issues/1874.html
- CWG Issue 2447: Abbreviated function templates — https://cplusplus.github.io/CWG/issues/2447.html
- CWG Issue 226: Default template arguments for function templates — https://cplusplus.github.io/CWG/issues/226.html
- CWG Issue 495: Ranking of template vs. non-template — https://cplusplus.github.io/CWG/issues/495.html
- Function template overloading — cppreference.com - https://en.cppreference.com/w/cpp/language/function_template#Function_template_overloading