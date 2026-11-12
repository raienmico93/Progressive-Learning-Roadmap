# Template Metaprogramming — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
Template metaprogramming (TMP) is a family of C++ techniques that use templates to **create new types and compute values at compile time**, effectively running programs during compilation rather than at runtime.

**Technical Definition**
TMP is a family of techniques to create new types and compute values at compile time. C++ templates are Turing complete if there are no limits to the amount of recursive instantiations and the number of allowed state variables. The standard recommends an implementation support at least 1024 levels of recursive instantiation, and infinite recursion in template instantiations is undefined behavior. TMP encompasses **type traits** (compile-time type properties), **compile-time arithmetic**, **SFINAE** (Substitution Failure Is Not An Error), **tag dispatch**, **`constexpr if`**, and **concepts** (C++20). These techniques enable generic libraries, optimization, and compile-time validation.

**Beginner-Friendly Explanation**
Template metaprogramming is **writing programs that run while your code is being compiled**. Instead of computing values at runtime, you compute them at compile time. For example, you can ask the compiler “is this type an integer?” or “what is the common type of these two types?” and get the answer before your program ever runs. TMP is what powers the standard library’s type traits, `std::enable_if`, and much of modern generic code. It can make programs faster (no runtime cost), catch errors earlier, and enable powerful abstractions—at the cost of complex syntax and potentially long compile times.

### Key Characteristics

- **Compile-time computation**: Values and types are computed during translation.
- **Turing complete**: Templates can simulate arbitrary computation (with recursion limits).
- **Type traits**: Compile-time queries about type properties.
- **SFINAE**: Substitution failure removes candidates instead of causing errors.
- **Tag dispatch**: Selecting overloads via tag types (e.g., iterator categories).
- **`constexpr if` (C++17)**: Compile-time branching in templates.
- **Concepts (C++20)**: First-class constraints on template parameters.
- **Zero runtime overhead**: Results are baked into the compiled code.

### Prerequisites

- Solid understanding of templates (function and class templates).
- Familiarity with template specialization and instantiation.
- Knowledge of overload resolution and type deduction.
- Basic awareness of the standard type traits header `<type_traits>`.

### Related Programming Areas

- **Generic programming** – TMP enables type-generic libraries.
- **Type traits** – the `<type_traits>` library.
- **SFINAE** – conditional overload participation.
- **Concepts (C++20)** – modern replacement for many SFINAE patterns.
- **Compile-time computation** – `constexpr`, `consteval`, and template recursion.
- **Library design** – `std::enable_if`, `std::conditional`, tag dispatch.

### Core Concepts / Features

1. Template Metaprogramming (TMP)
2. Compile-Time Computation
3. Type Traits
4. `std::enable_if`
5. `std::conditional`
6. `std::integral_constant` / `std::bool_constant`
7. `std::is_same`
8. `std::is_base_of`
9. `std::is_convertible`
10. SFINAE
11. `decltype` / `std::declval`
12. Tag Dispatch
13. `constexpr if`
14. Concepts (C++20)
15. Template Metaprogramming Best Practices

---

## 1. Template Metaprogramming (TMP)

### Definitions

**Core Definition**
Template metaprogramming is the use of C++ templates to perform computation at compile time, generating types and values as part of the compilation process.

**Technical Definition**
TMP is a family of techniques to create new types and compute values at compile time. C++ templates are Turing complete if there are no limits to the amount of recursive instantiations and the number of allowed state variables. TMP relies on template specialization, recursive instantiation, and SFINAE to implement compile-time logic. The standard recommends support for at least 1024 levels of recursive instantiation; infinite recursion is undefined behavior.

**Beginner-Friendly Explanation**
TMP is **programming the compiler**. You write templates that the compiler evaluates to produce types and constants. It’s like having a second program that runs before your real program starts.

### Purposes

- To perform compile-time computation and type generation.
- To enable zero-overhead generic libraries.
- To validate types and expressions at compile time.
- To implement type traits and metafunctions.
- To optimize code by moving work from runtime to compile time.

### Syntax Rules and Structure

TMP is expressed through:
- Template specialization (primary, full, partial).
- Recursive template instantiation.
- `using` aliases and `typedef`s.
- SFINAE and `std::enable_if`.
- `constexpr` functions and variables.
- `if constexpr` (C++17).
- Concepts and `requires` clauses (C++20).

**Component Breakdown**

| Technique | Role |
|-----------|------|
| Specialization | Selects implementations per type. |
| Recursion | Iterates at compile time. |
| SFINAE | Conditionally removes overloads. |
| `constexpr` | Computes values at compile time. |
| Concepts | Constrains templates. |

**Syntax Rules**
- TMP is typically header-only.
- Recursive instantiation must terminate.
- Specializations must be visible before use.
- C++17 `if constexpr` simplifies branching.

**Constraints and Limitations**
- Complex TMP can produce long error messages.
- Compile times can increase.
- Recursion depth is limited (implementation-defined).

### Annotated Code Examples

**Example 1: Compile-time factorial via TMP**

```cpp
#include <iostream>

template <unsigned N>
struct Factorial {
    static constexpr unsigned value = N * Factorial<N - 1>::value;
};

template <>
struct Factorial<0> {
    static constexpr unsigned value = 1;
};

int main() {
    std::cout << Factorial<5>::value << '\n';   // compile-time
}
```

**Expected Output**
```
120
```

**Why this output?**  
`Factorial<5>` recursively instantiates `Factorial<4>` … `Factorial<0>`, computing `120` at compile time.

### Real-World Cases

- **`std::tuple`** – implemented with TMP and variadic templates.
- **Type traits** – compile-time type queries.
- **Expression templates** – optimizing numerical libraries.
- **Policy-based design** – selecting policies at compile time.

---

## 2. Compile-Time Computation

### Definitions

**Core Definition**
Compile-time computation is the evaluation of expressions, types, or functions during translation rather than at runtime.

**Technical Definition**
C++ provides several mechanisms for compile-time computation: `constexpr` (C++11), `consteval` (C++20), `constinit` (C++20), template metaprogramming, and `if constexpr`. A `constexpr` function can be evaluated at compile time when called in a constant expression context. TMP uses recursive template instantiation and specialization to compute values and types.

**Beginner-Friendly Explanation**
Compile-time computation means **doing the work before the program runs**. The compiler computes the result and bakes it into the binary, so there’s no runtime cost.

### Purposes

- To eliminate runtime computation.
- To enable compile-time validation.
- To generate optimized code.
- To compute constants for templates and array sizes.

### Syntax Rules and Structure

**`constexpr` Function**
```cpp
constexpr int square(int x) { return x * x; }
constexpr int r = square(5);   // compile-time
```

**Template Recursion**
```cpp
template <int N> struct Sum { static constexpr int value = N + Sum<N-1>::value; };
template <> struct Sum<0> { static constexpr int value = 0; };
```

**`if constexpr` (C++17)**
```cpp
if constexpr (condition) { /* compile-time branch */ }
```

**Component Breakdown**

| Mechanism | Standard |
|-----------|----------|
| `constexpr` functions | C++11 (relaxed C++14) |
| Template recursion | C++98 |
| `if constexpr` | C++17 |
| `consteval` | C++20 |
| Concepts | C++20 |

**Syntax Rules**
- `constexpr` functions must be evaluable at compile time for constant expressions.
- Template recursion requires a base case.
- `if constexpr` discards the untaken branch.

**Constraints and Limitations**
- Compile-time recursion depth is limited.
- `constexpr` functions cannot perform all runtime operations.

### Annotated Code Examples

**Example 1: `constexpr` square**

```cpp
#include <iostream>

constexpr int square(int x) { return x * x; }

int main() {
    constexpr int r = square(5);   // compile-time
    std::cout << r << '\n';
}
```

**Expected Output**
```
25
```

**Why this output?**  
`square(5)` is evaluated at compile time because `r` is `constexpr`.

### Real-World Cases

- **Array bounds** – `constexpr` sizes.
- **Template arguments** – compile-time constants.
- **`static_assert`** – compile-time checks.
- **Lookup tables** – generated at compile time.

---

## 3. Type Traits

### Definitions

**Core Definition**
Type traits are class templates that provide **compile-time information about types**, such as whether a type is an integer, a pointer, or convertible to another type.

**Technical Definition**
Type traits define compile-time template-based interfaces to query the properties of types. Most non-transforming type traits derive from `std::integral_constant` (or `std::true_type`/`std::false_type`). The `<type_traits>` header provides a large set of traits (e.g., `is_integral`, `is_pointer`, `is_same`, `is_base_of`, `is_convertible`). Type traits are used with SFINAE, `if constexpr`, and concepts.

**Beginner-Friendly Explanation**
Type traits are **compile-time questions about types**. `std::is_integral<T>::value` asks: “Is `T` an integer type?” The answer is a compile-time `bool`.

### Purposes

- To query type properties at compile time.
- To enable conditional compilation based on types.
- To support generic algorithms and containers.
- To validate template arguments.

### Syntax Rules and Structure

**General Syntax**
```cpp
#include <type_traits>

std::is_integral<T>::value        // bool
std::is_integral_v<T>             // C++17 variable template
```

**Component Breakdown**

| Trait | Query |
|-------|-------|
| `is_integral` | Integer type? |
| `is_pointer` | Pointer type? |
| `is_same` | Same type? |
| `is_base_of` | Base/derived relationship? |
| `is_convertible` | Implicit conversion? |

**Syntax Rules**
- Most traits provide a `value` member (bool).
- `_v` variable templates (C++17) simplify access.
- Traits are used in SFINAE and `if constexpr`.

**Constraints and Limitations**
- Some traits require complete types.
- User specializations of standard traits are undefined behavior (except specific ones).

### Annotated Code Examples

**Example 1: `is_integral`**

```cpp
#include <iostream>
#include <type_traits>

int main() {
    std::cout << std::is_integral<int>::value << '\n';    // 1
    std::cout << std::is_integral<double>::value << '\n'; // 0
}
```

**Expected Output**
```
1
0
```

**Why this output?**  
`is_integral<int>` is `true`; `is_integral<double>` is `false`.

### Real-World Cases

- **SFINAE constraints** – `std::enable_if_t<std::is_integral_v<T>>`.
- **`if constexpr`** – select code paths per type.
- **Concepts** – standard concepts are built on type traits.
- **Optimization** – specialize for trivially copyable types.

---

## 4. `std::enable_if`

### Definitions

**Core Definition**
`std::enable_if` is a type trait that **conditionally defines a member `type`** based on a compile-time boolean, enabling SFINAE-based overload selection.

**Technical Definition**
`template<bool B, class T = void> struct enable_if;` — if `B` is true, `enable_if` has a public member typedef `type` equal to `T`; otherwise, there is no member typedef. This metafunction leverages SFINAE to conditionally remove functions from overload resolution based on type traits. It can be used as an additional function argument, a return type, or a template parameter. Since C++14, `enable_if_t` is a helper alias.

**Beginner-Friendly Explanation**
`std::enable_if` is a **compile-time gate**. You attach it to a function template; if the condition is true, the function exists; if false, the function is removed from consideration (SFINAE).

### Purposes

- To conditionally enable function templates based on type properties.
- To create overloads that participate only for certain types.
- To implement SFINAE-based constraints before concepts.

### Syntax Rules and Structure

**Return-Type Form**
```cpp
template <typename T>
typename std::enable_if<std::is_integral<T>::value, T>::type
f(T value) { return value; }
```

**Template-Parameter Form**
```cpp
template <typename T, std::enable_if_t<std::is_integral_v<T>, bool> = true>
void f(T value) {}
```

**Component Breakdown**

| Form | Usage |
|------|-------|
| Return type | Common for functions. |
| Extra template parameter | Works for constructors and operators. |
| Extra function argument | Works for operators like `operator[]`. |

**Syntax Rules**
- `enable_if<true, T>::type` is `T`; `enable_if<false, T>::type` does not exist.
- `enable_if_t` (C++14) simplifies syntax.
- Using `enable_if` in a non-type template parameter can have ABI issues.

**Constraints and Limitations**
- Common mistake: two function templates that differ only in default template arguments are redeclarations.
- Can produce hard-to-read errors.

### Annotated Code Examples

**Example 1: `enable_if` return type**

```cpp
#include <iostream>
#include <type_traits>

template <typename T>
typename std::enable_if<std::is_integral<T>::value, T>::type
double_it(T value) {
    return value * 2;
}

int main() {
    std::cout << double_it(5) << '\n';    // OK
    // double_it(3.14);                   // error: no matching function
}
```

**Expected Output**
```
10
```

**Why this output?**  
`double_it` is enabled only for integral types; `int` satisfies the condition.

### Real-World Cases

- **`std::make_pair`** – uses `enable_if` for SFINAE.
- **Conditional overloads** – different implementations per type category.
- **Legacy code** – pre-C++20 SFINAE.

---

## 5. `std::conditional`

### Definitions

**Core Definition**
`std::conditional` is a type trait that selects one of two types at compile time based on a boolean condition.

**Technical Definition**
`template<bool B, class T, class F> struct conditional;` provides a member typedef `type` defined as `T` if `B` is true, or `F` if `B` is false. Since C++14, `conditional_t` is a helper alias.

**Beginner-Friendly Explanation**
`std::conditional` is a **compile-time if-else for types**. `conditional_t<true, int, double>` is `int`; `conditional_t<false, int, double>` is `double`.

### Purposes

- To select a type based on a compile-time condition.
- To implement type-level branching.
- To choose between implementations in metafunctions.

### Syntax Rules and Structure

**General Syntax**
```cpp
typename std::conditional<B, T, F>::type
std::conditional_t<B, T, F>   // C++14
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `B` | Boolean condition. |
| `T` | Type if `B` is true. |
| `F` | Type if `B` is false. |

**Syntax Rules**
- `B` must be a compile-time constant.
- `type` is `T` or `F` accordingly.

**Constraints and Limitations**
- Both `T` and `F` must be valid types (they are not lazily evaluated).

### Annotated Code Examples

**Example 1: Selecting a type**

```cpp
#include <iostream>
#include <type_traits>

int main() {
    using T1 = std::conditional_t<true, int, double>;
    using T2 = std::conditional_t<false, int, double>;
    std::cout << std::is_same_v<T1, int> << ' '
              << std::is_same_v<T2, double> << '\n';
}
```

**Expected Output**
```
1 1
```

**Why this output?**  
`conditional_t<true, int, double>` is `int`; `conditional_t<false, int, double>` is `double`.

### Real-World Cases

- **Selecting storage types** – e.g., small-buffer optimization.
- **Type-level branching** – in metafunctions.
- **Policy selection** – choosing a policy type.

---

## 6. `std::integral_constant` / `std::bool_constant`

### Definitions

**Core Definition**
`std::integral_constant` wraps a static constant of a specified type; `std::bool_constant` is an alias for `integral_constant<bool, B>` (C++17).

**Technical Definition**
`template<class T, T v> struct integral_constant;` wraps a static constant of specified type. It is the base class for the C++ type traits. `bool_constant` is a helper alias for `integral_constant<bool, B>`. `true_type` and `false_type` are typedefs for `integral_constant<bool, true>` and `integral_constant<bool, false>`.

**Beginner-Friendly Explanation**
`integral_constant` is a **compile-time constant wrapper**. `integral_constant<int, 5>::value` is `5` at compile time. `bool_constant<true>` is `true_type`.

### Purposes

- To represent compile-time constants as types.
- To serve as the base for type traits.
- To enable tag dispatch and metafunctions.

### Syntax Rules and Structure

**General Syntax**
```cpp
std::integral_constant<int, 5>::value   // 5
std::bool_constant<true>::value         // true
std::true_type                          // integral_constant<bool, true>
std::false_type                         // integral_constant<bool, false>
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `T` | Type of the constant. |
| `v` | Value of the constant. |
| `value` | Static member constant. |

**Syntax Rules**
- `value` is a `static constexpr T`.
- `integral_constant` is convertible to `T` and callable (C++14).
- `bool_constant` requires C++17.

**Constraints and Limitations**
- Specializing `integral_constant` is undefined behavior.

### Annotated Code Examples

**Example 1: `integral_constant` and `bool_constant`**

```cpp
#include <iostream>
#include <type_traits>

int main() {
    using five = std::integral_constant<int, 5>;
    std::cout << five::value << '\n';          // 5
    std::cout << std::bool_constant<true>::value << '\n'; // 1
}
```

**Expected Output**
```
5
1
```

**Why this output?**  
`integral_constant<int, 5>::value` is `5`; `bool_constant<true>::value` is `true`.

### Real-World Cases

- **Type traits base** – `is_same`, `is_integral`, etc. derive from `integral_constant`.
- **Tag dispatch** – `true_type`/`false_type` overloads.
- **Compile-time constants** – as types.

---

## 7. `std::is_same`

### Definitions

**Core Definition**
`std::is_same<T, U>` checks whether `T` and `U` are the **same type** (including cv-qualification).

**Technical Definition**
`template<class T, class U> struct is_same;` — if `T` and `U` name the same type (including const/volatile qualifications), provides `value` equal to `true`; otherwise `false`. Commutativity is satisfied.

**Beginner-Friendly Explanation**
`is_same<int, int>` is `true`; `is_same<int, double>` is `false`.

### Purposes

- To compare types at compile time.
- To enable type-based branching.
- To implement metafunctions.

### Syntax Rules and Structure

**General Syntax**
```cpp
std::is_same<T, U>::value
std::is_same_v<T, U>   // C++17
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `T`, `U` | Types to compare. |
| `value` | `true` if same. |

**Syntax Rules**
- `const`/`volatile` qualifications matter.
- `is_same_v` is a C++17 variable template.

**Constraints and Limitations**
- Does not consider implicit conversions.

### Annotated Code Examples

**Example 1: `is_same`**

```cpp
#include <iostream>
#include <type_traits>

int main() {
    std::cout << std::is_same_v<int, int> << ' '
              << std::is_same_v<int, double> << '\n';
}
```

**Expected Output**
```
1 0
```

**Why this output?**  
`int` and `int` are the same type; `int` and `double` are not.

### Real-World Cases

- **Template specialization** – `enable_if_t<is_same_v<T, int>>`.
- **Type traits** – building blocks for other traits.
- **Concepts** – `std::same_as`.

---

## 8. `std::is_base_of`

### Definitions

**Core Definition**
`std::is_base_of<Base, Derived>` checks whether `Derived` is derived from `Base` (or is the same class), ignoring cv-qualification.

**Technical Definition**
`std::is_base_of` is a `BinaryTypeTrait`. If `Derived` is derived from `Base` or if both are the same non-union class (ignoring cv-qualification), provides `value` equal to `true`. `is_base_of<A, B>::value` is `true` even if `A` is a private, protected, or ambiguous base of `B`. `is_base_of<T, T>::value` is `true` (models “is-a”).

**Beginner-Friendly Explanation**
`is_base_of<Animal, Dog>` is `true` if `Dog` inherits from `Animal`.

### Purposes

- To detect inheritance relationships at compile time.
- To constrain templates to derived types.
- To implement polymorphic utilities.

### Syntax Rules and Structure

**General Syntax**
```cpp
std::is_base_of<Base, Derived>::value
std::is_base_of_v<Base, Derived>   // C++17
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `Base` | Base class. |
| `Derived` | Derived class. |

**Syntax Rules**
- Both must be class types (non-union).
- Ignores cv-qualification.
- Works with private/protected bases.

**Constraints and Limitations**
- Requires complete `Derived` type.
- `is_base_of<int, int>` is `false` (only classes).

### Annotated Code Examples

**Example 1: `is_base_of`**

```cpp
#include <iostream>
#include <type_traits>

struct A {};
struct B : A {};

int main() {
    std::cout << std::is_base_of_v<A, B> << ' '
              << std::is_base_of_v<B, A> << '\n';
}
```

**Expected Output**
```
1 0
```

**Why this output?**  
`B` derives from `A`; `A` does not derive from `B`.

### Real-World Cases

- **Constraining templates** – `enable_if_t<is_base_of_v<Base, T>>`.
- **Polymorphic utilities** – checking “is-a”.
- **Concepts** – `std::derived_from`.

---

## 9. `std::is_convertible`

### Definitions

**Core Definition**
`std::is_convertible<From, To>` checks whether `From` is **implicitly convertible** to `To`.

**Technical Definition**
`template<class From, class To> struct is_convertible;` — if the imaginary function `To test() { return std::declval<From>(); }` is well-formed, provides `value` equal to `true`; otherwise `false`. Access checks are performed as if from an unrelated context.

**Beginner-Friendly Explanation**
`is_convertible<int, double>` is `true` because `int` converts to `double`; `is_convertible<double, int>` is also `true`.

### Purposes

- To check implicit convertibility at compile time.
- To constrain templates to compatible types.
- To implement generic conversions.

### Syntax Rules and Structure

**General Syntax**
```cpp
std::is_convertible<From, To>::value
std::is_convertible_v<From, To>   // C++17
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `From` | Source type. |
| `To` | Target type. |

**Syntax Rules**
- Uses `std::declval` in unevaluated context.
- Works with reference types, `void`, arrays, functions.

**Constraints and Limitations**
- Some conversions (e.g., explicit) are not considered.
- Incomplete types cause undefined behavior.

### Annotated Code Examples

**Example 1: `is_convertible`**

```cpp
#include <iostream>
#include <type_traits>

int main() {
    std::cout << std::is_convertible_v<int, double> << ' '
              << std::is_convertible_v<double, int> << '\n';
}
```

**Expected Output**
```
1 1
```

**Why this output?**  
`int` implicitly converts to `double`; `double` implicitly converts to `int`.

### Real-World Cases

- **SFINAE constraints** – `enable_if_t<is_convertible_v<T, Base>>`.
- **Concepts** – `std::convertible_to`.
- **Generic APIs** – accepting compatible types.

---

## 10. SFINAE

### Definitions

**Core Definition**
SFINAE stands for **Substitution Failure Is Not An Error**: when substituting template arguments fails, the specialization is discarded from the overload set instead of causing a compile error.

**Technical Definition**
This rule applies during overload resolution of function templates: when substituting the explicitly specified or deduced type for the template parameter fails, the specialization is discarded from the overload set instead of causing a compile error. Substitution failure is any situation where the type or expression would be ill-formed with the substituted arguments. Only failures in the immediate context are SFINAE errors; errors in side effects are hard errors. SFINAE is used in template metaprogramming to manipulate overload sets.

**Beginner-Friendly Explanation**
SFINAE means **“if it doesn’t fit, just ignore it instead of erroring.”** When the compiler tries a template and something doesn’t work for a particular type, it quietly removes that template from consideration rather than failing compilation.

### Purposes

- To conditionally enable or disable function templates.
- To create overload sets that adapt to type properties.
- To implement compile-time introspection.
- To replace some forms of specialization.

### Syntax Rules and Structure

SFINAE is not written directly; it emerges from template substitution. It is typically triggered by `std::enable_if`, trailing return types, or expressions in the immediate context.

**Common Patterns**

| Pattern | Example |
|---------|---------|
| Return type | `enable_if_t<cond, T>` |
| Template parameter | `enable_if_t<cond, bool> = true` |
| Trailing return | `auto f(T) -> decltype(expr)` |

**Syntax Rules**
- Substitution failure must occur in the immediate context.
- Errors in instantiated templates are hard errors.
- `decltype`, `enable_if`, and `void_t` are common tools.

**Constraints and Limitations**
- SFINAE errors can be obscure.
- Not all failures are SFINAE-friendly.
- Concepts (C++20) are often clearer.

### Annotated Code Examples

**Example 1: SFINAE with `enable_if`**

```cpp
#include <iostream>
#include <type_traits>

template <typename T>
std::enable_if_t<std::is_integral_v<T>, T>
twice(T value) { return value * 2; }

int main() {
    std::cout << twice(5) << '\n';    // OK
    // twice(3.14);                   // error: no matching function
}
```

**Expected Output**
```
10
```

**Why this output?**  
`twice` is enabled only for integral types; for `double`, substitution fails and the overload is removed (SFINAE).

### Real-World Cases

- **`std::enable_if`** – the classic SFINAE tool.
- **Detection idiom** – detecting well-formed expressions.
- **Iterator category dispatch** – selecting algorithms.

---

## 11. `decltype` / `std::declval`

### Definitions

**Core Definition**
`decltype` yields the type of an expression without evaluating it; `std::declval<T>()` produces a value of type `T` in unevaluated contexts without requiring a constructor.

**Technical Definition**
`decltype` (C++11) inspects the declared type of an entity or the type and value category of an expression. `std::declval<T>()` is a helper template for writing expressions in unevaluated contexts (typically the operand of `decltype`). It converts any type `T` to a reference type, making it possible to use member functions without specifying constructors. `declval` can only be used in unevaluated contexts and is not required to be defined.

**Beginner-Friendly Explanation**
`decltype` asks: **“What is the type of this expression?”** without actually running it. `declval<T>()` creates a fake value of type `T` so you can write expressions like `decltype(std::declval<T>().foo())` without needing a real object.

### Purposes

- To deduce types in templates.
- To inspect expression types without evaluating them.
- To enable SFINAE with `decltype`.
- To write forwarding and result-type utilities.

### Syntax Rules and Structure

**`decltype`**
```cpp
decltype(expression)   // type of expression
```

**`std::declval`**
```cpp
decltype(std::declval<T>().member())
```

**Component Breakdown**

| Tool | Role |
|------|------|
| `decltype` | Deduces the type of an expression. |
| `declval` | Creates an unevaluated value of a type. |

**Syntax Rules**
- `decltype` does not evaluate its operand.
- `declval` may only be used in unevaluated contexts.
- `declval<T>()` returns `T&&` (reference collapsing).

**Constraints and Limitations**
- `declval` cannot be odr-used.
- `decltype` rules for parenthesized expressions differ from unparenthesized ones.

### Annotated Code Examples

**Example 1: `decltype` and `declval`**

```cpp
#include <iostream>
#include <type_traits>
#include <utility>

struct Widget {
    int value() const { return 42; }
};

int main() {
    using R = decltype(std::declval<Widget>().value());
    std::cout << std::is_same_v<R, int> << '\n';
}
```

**Expected Output**
```
1
```

**Why this output?**  
`declval<Widget>()` creates an unevaluated `Widget`; `.value()` yields `int`; `decltype` deduces `int`.

### Real-World Cases

- **SFINAE** – `decltype` in trailing return types.
- **Result-type deduction** – `decltype(auto)`.
- **Detection idiom** – detecting member functions.

---

## 12. Tag Dispatch

### Definitions

**Core Definition**
Tag dispatch is a technique that selects a function overload based on a **tag type** (e.g., `std::input_iterator_tag`), typically derived from type traits.

**Technical Definition**
Tag dispatch uses tag types to select the most appropriate overload at compile time. A common example is iterator category dispatch: algorithms call a dispatcher with `std::iterator_traits<It>::iterator_category()`, which selects the overload for that category. `std::true_type` and `std::false_type` are commonly used as tags.

**Beginner-Friendly Explanation**
Tag dispatch is **choosing an overload by passing a “label” type**. For example, you pass `true_type` or `false_type` to select one implementation or another.

### Purposes

- To select implementations based on type properties.
- To avoid SFINAE complexity.
- To implement iterator category dispatch.
- To enable compile-time branching via overloads.

### Syntax Rules and Structure

**General Pattern**
```cpp
void impl(std::true_type)  { /* ... */ }
void impl(std::false_type) { /* ... */ }

template <typename T>
void f(T value) {
    impl(std::is_integral<T>{});
}
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| Tag type | `true_type`, `false_type`, `input_iterator_tag`, etc. |
| Dispatcher | Overloads selected by tag. |

**Syntax Rules**
- Tags are usually empty types.
- The dispatcher calls the appropriate overload.
- Tags can be created from type traits.

**Constraints and Limitations**
- Requires overloads for each tag.
- Can be less flexible than SFINAE or concepts.

### Annotated Code Examples

**Example 1: `true_type` / `false_type` dispatch**

```cpp
#include <iostream>
#include <type_traits>

void impl(std::true_type)  { std::cout << "integral\n"; }
void impl(std::false_type) { std::cout << "not integral\n"; }

template <typename T>
void classify(T) {
    impl(std::is_integral<T>{});
}

int main() {
    classify(42);
    classify(3.14);
}
```

**Expected Output**
```
integral
not integral
```

**Why this output?**  
`is_integral<int>` is `true_type`; `is_integral<double>` is `false_type`.

### Real-World Cases

- **STL iterator dispatch** – `std::distance` uses tag dispatch.
- **`std::true_type`/`false_type`** – common tags.
- **Optimization** – selecting fast paths.

---

## 13. `constexpr if`

### Definitions

**Core Definition**
`if constexpr` (C++17) is a compile-time conditional that **discards the untaken branch**, enabling type-dependent branching in templates.

**Technical Definition**
In a `constexpr if` statement, the condition must be a contextually converted constant expression of type `bool`. If the value is `true`, the `statement-false` is discarded (if present); otherwise, `statement-true` is discarded. Discarded statements are not instantiated. `constexpr if` is commonly used to replace SFINAE and tag dispatch for simple cases.

**Beginner-Friendly Explanation**
`if constexpr` is an **if that runs at compile time**. The compiler chooses one branch and completely ignores the other, so you can write code that only makes sense for certain types.

### Purposes

- To branch on type properties at compile time.
- To avoid SFINAE and tag dispatch in simple cases.
- To write type-dependent code in templates.
- To improve error messages and readability.

### Syntax Rules and Structure

**General Syntax**
```cpp
if constexpr (condition) {
    // instantiated if true
} else {
    // instantiated if false
}
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `condition` | Compile-time boolean. |
| True branch | Instantiated if true. |
| False branch | Instantiated if false. |

**Syntax Rules**
- The condition must be a constant expression.
- The untaken branch is discarded (not instantiated).
- Can be chained with `else if constexpr`.

**Constraints and Limitations**
- Requires C++17.
- The discarded branch must still be syntactically valid.

### Annotated Code Examples

**Example 1: `if constexpr`**

```cpp
#include <iostream>
#include <type_traits>

template <typename T>
void describe(T) {
    if constexpr (std::is_integral_v<T>) {
        std::cout << "integral\n";
    } else {
        std::cout << "not integral\n";
    }
}

int main() {
    describe(42);
    describe(3.14);
}
```

**Expected Output**
```
integral
not integral
```

**Why this output?**  
The compiler selects the branch based on `is_integral_v<T>` and discards the other.

### Real-World Cases

- **Generic algorithms** – different implementations per type.
- **Serialization** – type-dependent encoding.
- **Compile-time dispatch** – replacing SFINAE.

---

## 14. Concepts (C++20)

### Definitions

**Core Definition**
Concepts are C++20 **first-class constraints** on template parameters, expressing requirements on types and enabling better error messages and overload selection.

**Technical Definition**
The concepts library provides definitions of fundamental library concepts that can be used to perform compile-time validation of template arguments and perform function dispatch based on properties of types. Concepts impose syntactic and semantic requirements. They replace many SFINAE and `enable_if` patterns with cleaner syntax and better errors.

**Beginner-Friendly Explanation**
Concepts are **named requirements for template parameters**. `template <std::integral T>` says “`T` must be an integer type.” If it isn’t, the compiler gives a clear error instead of a wall of template errors.

### Purposes

- To constrain templates clearly and readably.
- To improve error messages.
- To enable concept-based overloading.
- To replace much SFINAE and `enable_if`.

### Syntax Rules and Structure

**General Syntax**
```cpp
template <std::integral T>
T add(T a, T b) { return a + b; }

template <typename T>
requires std::integral<T>
void f(T value);
```

**Component Breakdown**

| Form | Meaning |
|------|---------|
| `std::integral T` | Constrained template parameter. |
| `requires` clause | Constraint on a template. |

**Syntax Rules**
- Concepts are boolean predicates on types.
- Standard concepts include `std::integral`, `std::floating_point`, `std::same_as`, `std::derived_from`, `std::convertible_to`.
- `requires` clauses can be used with `if constexpr` or on templates.

**Constraints and Limitations**
- Requires C++20.
- Concepts can be more complex to define than traits.

### Annotated Code Examples

**Example 1: Concept-constrained function**

```cpp
#include <iostream>
#include <concepts>

template <std::integral T>
T add(T a, T b) { return a + b; }

int main() {
    std::cout << add(1, 2) << '\n';
    // add(1.5, 2.5);   // error: double does not satisfy std::integral
}
```

**Expected Output**
```
3
```

**Why this output?**  
The `std::integral` concept enables `add` only for integral types.

### Real-World Cases

- **Standard library** – C++20 ranges and algorithms use concepts.
- **Generic libraries** – clear constraints.
- **Code review** – preferred over SFINAE.

---

## 15. Template Metaprogramming Best Practices

### Definitions

**Core Definition**
Template metaprogramming best practices are guidelines for writing TMP code that is correct, readable, and maintainable.

**Technical Definition**
Key practices include: prefer concepts (C++20) over SFINAE where possible; use `if constexpr` (C++17) for compile-time branching; use `enable_if` only when concepts are unavailable; keep metafunctions small and focused; document specializations; prefer `std::integral_constant`/`bool_constant` for compile-time constants; and use tag dispatch for category-based overloads.

**Beginner-Friendly Explanation**
Best practices for TMP are simple: **use concepts when you can**, **use `if constexpr` for branching**, **avoid overly complex SFINAE**, **keep metafunctions simple**, and **document your templates**.

### Key Guidelines (Summary Table)

| Guideline | Summary | Recommendation |
|-----------|---------|----------------|
| **Prefer concepts** | C++20 concepts are clearer and give better errors. | `template <std::integral T>`. |
| **Use `if constexpr`** | Compile-time branching without SFINAE. | `if constexpr (cond)`. |
| **Use `enable_if` when needed** | Pre-C++20 SFINAE. | `enable_if_t<cond, T>`. |
| **Keep metafunctions small** | Focused, reusable traits. | One responsibility per trait. |
| **Document specializations** | Specializations are commitments. | Comments and docs. |
| **Prefer `integral_constant`** | Compile-time constants as types. | `bool_constant`, `true_type`. |
| **Use tag dispatch** | Category-based overloads. | `true_type`/`false_type`. |

### Annotated Code Examples

**Example 1: Concept instead of SFINAE**

```cpp
#include <iostream>
#include <concepts>

// GOOD: concept (C++20)
template <std::integral T>
T twice(T value) { return value * 2; }

int main() {
    std::cout << twice(5) << '\n';
    // twice(3.14);   // clear error
}
```

**Expected Output**
```
10
```

**Why this is best practice?**  
Concepts are clearer and produce better error messages than SFINAE.

### Real-World Cases

- **Modern C++ libraries** – concepts are the default.
- **Code review** – concepts and `if constexpr` are preferred.
- **Static analysis** – tools check template constraints.

---

## References

- Template metaprogramming (cppreference) – https://en.cppreference.com/w/cpp/language/template_metaprogramming
- SFINAE (cppreference) – https://en.cppreference.com/w/cpp/language/sfinae
- Metaprogramming library (cppreference) – https://en.cppreference.com/w/cpp/meta
- `std::enable_if` (cppreference) – https://en.cppreference.com/w/cpp/types/enable_if
- `std::conditional` (cppreference) – https://en.cppreference.com/w/cpp/types/conditional
- `std::integral_constant` (cppreference) – https://en.cppreference.com/w/cpp/types/integral_constant
- `std::is_same` (cppreference) – https://en.cppreference.com/w/cpp/types/is_same
- `std::is_base_of` (cppreference) – https://en.cppreference.com/w/cpp/types/is_base_of
- `std::is_convertible` (cppreference) – https://en.cppreference.com/w/cpp/types/is_convertible
- `std::declval` (cppreference) – https://en.cppreference.com/w/cpp/utility/declval
- Tag dispatch (cppreference, iterator tags) – https://en.cppreference.com/w/cpp/iterator/iterator_tags
- `constexpr if` (cppreference) – https://en.cppreference.com/w/cpp/language/if#Constexpr_if
- Concepts library (cppreference) – https://en.cppreference.com/w/cpp/concepts
- C++ Core Guidelines (templates section) – https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines

---

This cheat sheet provides a complete, structured reference for C++ template metaprogramming, covering TMP, compile-time computation, type traits, `enable_if`, `conditional`, `integral_constant`/`bool_constant`, `is_same`, `is_base_of`, `is_convertible`, SFINAE, `decltype`/`declval`, tag dispatch, `constexpr if`, concepts, and best practices. Each section follows a uniform template with definitions, purposes, syntax rules, annotated examples, expected outputs, and authoritative references.