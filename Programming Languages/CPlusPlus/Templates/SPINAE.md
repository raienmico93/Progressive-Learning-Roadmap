# SFINAE — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
SFINAE (Substitution Failure Is Not An Error) is a C++ template rule stating that if substituting template arguments into a function template fails (i.e., produces an ill-formed type or expression), the compiler **discards that specialization from the overload set** instead of emitting a hard compile error.

**Technical Definition**
SFINAE stands for “Substitution Failure Is Not An Error”. This rule applies during overload resolution of function templates: when substituting the explicitly specified or deduced type for the template parameter fails, the specialization is discarded from the overload set instead of causing a compile error . A substitution failure is any situation when the type or expression above would be ill-formed (with a required diagnostic) . Only failures in the **immediate context** of the function type, its template parameter types, or its explicit specifier (since C++20) are SFINAE errors . Errors inside function bodies or in side effects of instantiations are **hard errors**, not SFINAE. Function template parameters are substituted twice: explicitly specified arguments are substituted before template argument deduction, and deduced arguments are substituted after . SFINAE is the foundation of template metaprogramming techniques such as `std::enable_if`, `std::void_t`, and the detection idiom .

**Beginner-Friendly Explanation**
SFINAE is the compiler’s way of saying: **“If this template doesn’t fit the argument types, just ignore it — don’t abort the whole program.”** Imagine you have a toolbox with several tools. If one tool doesn’t fit the screw you’re working on, you don’t throw away the entire toolbox; you just reach for the next tool. SFINAE lets the compiler try each function template, quietly set aside the ones that don’t match the argument types, and pick the best one that does. This makes generic code flexible and robust. Without SFINAE, a single mismatch in any candidate template would cause a compilation error, even if another candidate would work perfectly.

### Key Characteristics

- **Silent removal**: Invalid specializations are removed from the overload set, not reported as errors.
- **Immediate context only**: Only failures in the function type, template parameter types, or explicit specifier count as SFINAE .
- **Overload resolution**: SFINAE operates during overload resolution of function templates .
- **Two substitution phases**: Explicit arguments are substituted before deduction; deduced arguments after .
- **Metaprogramming foundation**: SFINAE enables `std::enable_if`, `std::void_t`, and the detection idiom .
- **Pre-C++20 mechanism**: Since C++20, concepts are the preferred alternative for constraining templates .

### Prerequisites

- Understanding of function templates and template argument deduction.
- Familiarity with overload resolution.
- Knowledge of the `<type_traits>` header.
- Basic awareness of `decltype` and `std::declval`.

### Related Programming Areas

- **Template metaprogramming** – SFINAE is a core TMP technique.
- **Overload resolution** – SFINAE participates in candidate selection.
- **Type traits** – SFINAE is used with traits like `is_integral`, `is_same`.
- **`std::enable_if`** – the classic SFINAE tool .
- **Detection idiom** – SFINAE with `std::void_t` .
- **Concepts (C++20)** – the modern replacement for most SFINAE uses .

### Core Concepts / Features

1. SFINAE
2. Substitution Failure Is Not An Error
3. `std::enable_if`
4. `std::void_t`
5. Detection Idiom
6. SFINAE Best Practices
7. SFINAE Alternatives

---

## 1. SFINAE

### Definitions

**Core Definition**
SFINAE is the rule that a substitution failure during template argument substitution discards the specialization from the overload set instead of causing a compile error.

**Technical Definition**
During overload resolution of function templates, the compiler substitutes explicitly specified or deduced template arguments into the function type. If substitution fails (the resulting type or expression is ill-formed), the specialization is discarded from the overload set . Only errors in the immediate context — the function type, template parameter types, or explicit specifier — are SFINAE errors . SFINAE is used in template metaprogramming to conditionally enable or disable overloads .

**Beginner-Friendly Explanation**
SFINAE means **“if it doesn’t fit, just skip it.”** The compiler tries each function template; if substituting the types doesn’t work, it quietly moves on to the next candidate instead of failing the whole compilation.

### Purposes

- To silently discard invalid template specializations from overload resolution.
- To enable overload sets that adapt to type properties.
- To support template metaprogramming techniques like `std::enable_if`.
- To allow generic code to work with many types without hard errors.
- To provide the foundation for the detection idiom and type traits .

### Syntax Rules and Structure

SFINAE is not written directly; it emerges from template substitution. It is typically triggered by constructs like `std::enable_if`, trailing return types, or expressions in the immediate context.

**Common Trigger Patterns**

| Pattern | Example |
|---------|---------|
| Return type | `typename T::value_type f(T)` — fails if `T::value_type` doesn’t exist |
| Template parameter | `enable_if_t<cond, T>` |
| Trailing return | `auto f(T) -> decltype(expr)` |
| Expression in type | `decltype(std::declval<T>().member())` |

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| Immediate context | Function type, template parameter types, explicit specifier  |
| Substitution | Replacing template parameters with arguments  |
| Discarding | Removing the specialization from the overload set  |

**Syntax Rules**
- Substitution failure must occur in the **immediate context** of the function template.
- Errors in function bodies or in instantiated templates are **hard errors**.
- The failure must be detectable during substitution (not at instantiation time).
- SFINAE applies to function templates (and partial specializations of class templates, with limits).

**Constraints and Limitations**
- SFINAE errors can produce cryptic error messages.
- Not all failures are SFINAE-friendly (e.g., errors in default arguments, function bodies).
- SFINAE does not work for class template partial specializations in all cases.

### Annotated Code Examples

**Example 1: SFINAE via a missing member type**

```cpp
#include <iostream>
#include <vector>

template <typename T>
typename T::value_type get_first(const T& container) {
    return *container.begin();          // uses T::value_type
}

template <typename T>
T get_first(T value) {                  // fallback: non-container
    return value;
}

int main() {
    std::vector<int> v = {10, 20, 30};
    std::cout << get_first(v) << '\n';  // uses container overload
    std::cout << get_first(42) << '\n'; // uses fallback (int has no ::value_type)
}
```

**Expected Output**
```
10
42
```

**Why this output?**  
For `std::vector<int>`, `T::value_type` exists, so the first overload is viable. For `int`, `T::value_type` does not exist; substitution fails (SFINAE), the overload is discarded, and the fallback is selected .

### Real-World Cases

- **`std::enable_if`** – conditionally enabling overloads .
- **Detection idiom** – detecting member existence .
- **Iterator category dispatch** – selecting algorithms via tag types.
- **Type traits** – many standard traits rely on SFINAE .

---

## 2. Substitution Failure Is Not An Error

### Definitions

**Core Definition**
“Substitution Failure Is Not An Error” is the full name of SFINAE; it expresses the principle that a failed substitution during template argument substitution is not a hard compile error.

**Technical Definition**
The rule applies during overload resolution of function templates: when substituting the explicitly specified or deduced type for the template parameter fails, the specialization is discarded from the overload set instead of causing a compile error . A substitution failure is any situation where the type or expression would be ill-formed . Only failures in the immediate context (function type, template parameter types, explicit specifier) are SFINAE errors .

**Beginner-Friendly Explanation**
The name says it all: **a failed substitution is not treated as an error**. The compiler simply moves on to the next candidate. This is what makes SFINAE possible: the compiler can explore many template candidates without aborting on the first mismatch.

### Purposes

- To allow the compiler to explore multiple template candidates without hard errors.
- To enable overload sets that adapt to type properties.
- To support conditional compilation based on type traits.
- To provide the semantic foundation for SFINAE-based metaprogramming.

### Syntax Rules and Structure

**Substitution Phases**

| Phase | Timing |
|-------|--------|
| Explicit arguments | Substituted before template argument deduction  |
| Deduced arguments | Substituted after template argument deduction  |

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| Substitution failure | Ill-formed type or expression  |
| Immediate context | Function type, template parameter types  |
| Discarding | Removal from overload set  |

**Syntax Rules**
- The failure must be in the immediate context.
- Errors in function bodies or instantiations are hard errors .
- The failure must be detectable during substitution.

**Constraints and Limitations**
- Errors in default arguments are not always SFINAE-friendly.
- Errors in function bodies are always hard errors .

### Annotated Code Examples

**Example 1: Immediate context vs. hard error**

```cpp
// SFINAE: failure in immediate context (return type)
template <typename T>
typename T::value_type first(const T& c) { return *c.begin(); }

// HARD ERROR: failure inside the function body
template <typename T>
void process(T value) {
    typename T::value_type x = value;   // hard error if T has no value_type
}
```

**Expected Output**  
`first` is SFINAE-friendly; `process` produces a hard error.

**Why this difference?**  
The failure in `first` occurs in the return type (immediate context); the failure in `process` occurs in the function body (hard error).

### Real-World Cases

- **`std::enable_if`** – creates substitution failures in the immediate context .
- **Trailing return types** – `decltype` in the return type triggers SFINAE .
- **Expression SFINAE** – expressions in template parameters.

---

## 3. `std::enable_if`

### Definitions

**Core Definition**
`std::enable_if` is a type trait that conditionally defines a member `type` based on a compile-time boolean, enabling SFINAE-based conditional overload selection.

**Technical Definition**
`template<bool B, class T = void> struct enable_if;` — if `B` is true, `std::enable_if` has a public member typedef `type` equal to `T`; otherwise, there is no member typedef . This metafunction is a convenient way to leverage SFINAE prior to C++20’s concepts, in particular for conditionally removing functions from the candidate set based on type traits . Since C++14, `enable_if_t` is a helper alias .

**Beginner-Friendly Explanation**
`std::enable_if` is a **compile-time gate**. You attach it to a function template; if the condition is true, the function exists; if false, the function is removed from consideration (SFINAE) .

### Purposes

- To conditionally enable function templates based on type properties.
- To create overloads that participate only for certain types.
- To implement SFINAE-based constraints before concepts .
- To remove candidates from overload resolution .

### Syntax Rules and Structure

**Return-Type Form**
```cpp
template <typename T>
typename std::enable_if<std::is_integral<T>::value, T>::type
f(T value) { return value; }
```

**Template-Parameter Form (C++14+)**
```cpp
template <typename T, std::enable_if_t<std::is_integral_v<T>, bool> = true>
void f(T value) {}
```

**Extra Function Argument Form**
```cpp
template <typename T>
void f(T value, std::enable_if_t<std::is_integral_v<T>, int> = 0) {}
```

**Component Breakdown**

| Form | Usage |
|------|-------|
| Return type | Common for functions  |
| Template parameter | Works for constructors and operators  |
| Extra function argument | Works for `operator[]` and `operator()`  |

**Syntax Rules**
- `enable_if<true, T>::type` is `T`; `enable_if<false, T>::type` does not exist .
- `enable_if_t` (C++14) simplifies syntax .
- Common mistake: two function templates differing only in default template arguments are treated as redeclarations .

**Constraints and Limitations**
- `enable_if` in non-type template parameters can cause ABI issues .
- Produces hard-to-read error messages .

### Annotated Code Examples

**Example 1: `enable_if` return type**

```cpp
#include <iostream>
#include <type_traits>

template <typename T>
std::enable_if_t<std::is_integral_v<T>, T>
double_it(T value) { return value * 2; }

int main() {
    std::cout << double_it(5) << '\n';    // OK: int is integral
    // double_it(3.14);                   // error: no matching function
}
```

**Expected Output**
```
10
```

**Why this output?**  
`double_it` is enabled only for integral types; `int` satisfies the condition .

**Example 2: `enable_if` as extra template parameter**

```cpp
#include <iostream>
#include <type_traits>

template <typename T, std::enable_if_t<std::is_arithmetic_v<T>, int> = 0>
T add(T a, T b) { return a + b; }

int main() {
    std::cout << add(3, 4) << '\n';        // OK
    std::cout << add(1.5, 2.5) << '\n';    // OK: double is arithmetic
}
```

**Expected Output**
```
7
4
```

**Why this output?**  
`add` is enabled for arithmetic types, including `int` and `double` .

### Real-World Cases

- **`std::make_pair`** – uses `enable_if` for SFINAE.
- **Conditional overloads** – different implementations per type category.
- **Pre-C++20 libraries** – the standard SFINAE tool .

---

## 4. `std::void_t`

### Definitions

**Core Definition**
`std::void_t` (C++17) is a utility metafunction that maps a sequence of any types to `void`, used to detect ill-formed types in SFINAE contexts.

**Technical Definition**
`template<class...> using void_t = void;` — maps a sequence of any types to `void` . This metafunction is a convenient way to leverage SFINAE prior to C++20’s concepts, in particular for conditionally removing functions from the candidate set based on whether an expression is valid in an unevaluated context (such as `decltype`) . It is used in template metaprogramming to detect ill-formed types in SFINAE contexts .

**Beginner-Friendly Explanation**
`std::void_t` is a **detector gadget**. You pass it a type expression; if that expression is valid, `void_t` produces `void` (success). If the expression is ill-formed, `void_t` triggers a substitution failure (SFINAE), and the specialization is discarded .

### Purposes

- To detect whether a type or expression is valid in a SFINAE context .
- To implement the detection idiom .
- To conditionally enable specializations based on supported operations .
- To simplify SFINAE-based type trait implementations .

### Syntax Rules and Structure

**General Syntax**
```cpp
template <typename, typename = void>
struct trait : std::false_type {};

template <typename T>
struct trait<T, std::void_t<expression>> : std::true_type {};
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `void_t` | Maps types to `void`  |
| Primary template | Handles types without the detected member  |
| Specialization | Recognizes types with the member  |

**Syntax Rules**
- `void_t` requires C++17 .
- If the expression inside `void_t` is ill-formed, SFINAE discards the specialization .
- Pre-C++17 compilers may require a more complex definition (e.g., `make_void`) .

**Constraints and Limitations**
- Requires C++17 (or a workaround for older compilers).
- Only detects validity of expressions, not their values.

### Annotated Code Examples

**Example 1: Detecting a nested type**

```cpp
#include <iostream>
#include <type_traits>

// Primary template: no nested ::type member
template <typename, typename = void>
struct has_type_member : std::false_type {};

// Specialization: recognizes types with a nested ::type member
template <typename T>
struct has_type_member<T, std::void_t<typename T::type>> : std::true_type {};

struct WithType { using type = int; };
struct WithoutType {};

int main() {
    std::cout << has_type_member<WithType>::value << '\n';    // 1
    std::cout << has_type_member<WithoutType>::value << '\n'; // 0
}
```

**Expected Output**
```
1
0
```

**Why this output?**  
`WithType::type` exists, so the specialization matches (`true`). `WithoutType::type` does not exist, so SFINAE discards the specialization, and the primary template (`false`) is used .

**Example 2: Detecting an expression**

```cpp
#include <iostream>
#include <type_traits>
#include <utility>

template <typename, typename = void>
struct has_pre_increment : std::false_type {};

template <typename T>
struct has_pre_increment<T, std::void_t<decltype(++std::declval<T&>())>> : std::true_type {};

struct Incrementable { Incrementable& operator++(); };
struct NotIncrementable {};

int main() {
    std::cout << has_pre_increment<Incrementable>::value << '\n';    // 1
    std::cout << has_pre_increment<NotIncrementable>::value << '\n'; // 0
}
```

**Expected Output**
```
1
0
```

**Why this output?**  
`++std::declval<Incrementable&>()` is valid, so the specialization matches (`true`). For `NotIncrementable`, the expression is ill-formed, triggering SFINAE and falling back to `false` .

### Real-World Cases

- **`std::is_iterable`** – detecting `begin()`/`end()` members .
- **Detection idiom** – detecting member functions, operators, and types .
- **Type trait implementations** – SFINAE-friendly traits .

---

## 5. Detection Idiom

### Definitions

**Core Definition**
The detection idiom is a C++ template metaprogramming pattern that uses SFINAE (typically with `std::void_t`) to **detect whether a type supports a particular operation or member**, and to produce a compile-time boolean result.

**Technical Definition**
The detection idiom leverages SFINAE via `std::void_t` in a template to try to provide a compile-time `bool` of whether an expression was detected . It typically consists of a primary template (false case) and a partial specialization using `void_t<expression>` (true case when the expression is valid) . The detection idiom is a dependency of `std::void_t` and is widely used in template metaprogramming .

**Beginner-Friendly Explanation**
The detection idiom is a **yes/no test for type capabilities**. It asks: “Does this type have a `.size()` method?” or “Can this type be incremented?” The answer is a compile-time boolean. You build it with `void_t` and a pair of templates: one for “no” and one for “yes, the expression works.”

### Purposes

- To detect whether a type supports a member, operation, or nested type .
- To produce compile-time booleans about type capabilities .
- To conditionally enable code based on supported operations .
- To implement SFINAE-friendly type traits .

### Syntax Rules and Structure

**General Pattern**
```cpp
// Primary template: false case
template <typename, typename = void>
struct has_member : std::false_type {};

// Specialization: true case if expression is valid
template <typename T>
struct has_member<T, std::void_t<decltype(expression)>> : std::true_type {};
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| Primary template | Defaults to `false`  |
| Specialization with `void_t` | Becomes `true` when the expression is valid  |
| Expression | The operation or member to detect  |

**Syntax Rules**
- The expression must be in an unevaluated context (e.g., `decltype`) .
- `void_t` triggers SFINAE when the expression is ill-formed .
- The result is a `std::true_type` or `std::false_type` .

**Constraints and Limitations**
- Requires C++17 for `std::void_t` (or a workaround).
- Only detects validity, not semantics.
- Complex expressions may require care with `decltype`.

### Annotated Code Examples

**Example 1: Detection idiom for a member function**

```cpp
#include <iostream>
#include <type_traits>
#include <utility>

// Detect whether T has a .size() member function
template <typename, typename = void>
struct has_size : std::false_type {};

template <typename T>
struct has_size<T, std::void_t<decltype(std::declval<T>().size())>> : std::true_type {};

#include <vector>
#include <string>

int main() {
    std::cout << has_size<std::vector<int>>::value << '\n';   // 1
    std::cout << has_size<std::string>::value << '\n';        // 1
    std::cout << has_size<int>::value << '\n';                // 0
}
```

**Expected Output**
```
1
1
0
```

**Why this output?**  
`std::vector<int>` and `std::string` have `.size()`, so the specialization matches (`true`). `int` does not, so SFINAE discards the specialization and the primary template (`false`) is used .

**Example 2: Detection idiom for an operator**

```cpp
#include <iostream>
#include <type_traits>
#include <utility>

template <typename, typename = void>
struct has_plus : std::false_type {};

template <typename T>
struct has_plus<T, std::void_t<decltype(std::declval<T>() + std::declval<T>())>> : std::true_type {};

struct Addable { Addable operator+(const Addable&) const; };
struct NotAddable {};

int main() {
    std::cout << has_plus<Addable>::value << '\n';    // 1
    std::cout << has_plus<NotAddable>::value << '\n'; // 0
}
```

**Expected Output**
```
1
0
```

**Why this output?**  
`Addable + Addable` is valid, so `has_plus<Addable>` is `true`. `NotAddable + NotAddable` is ill-formed, so `has_plus<NotAddable>` is `false` .

### Real-World Cases

- **Standard library traits** – `std::is_iterable`, `has_begin_end` .
- **Generic algorithms** – selecting implementations based on supported operations.
- **Library design** – SFINAE-friendly customization points.

---

## 6. SFINAE Best Practices

### Definitions

**Core Definition**
SFINAE best practices are guidelines for using SFINAE correctly, readably, and maintainably, and for knowing when to prefer alternative techniques.

**Technical Definition**
Key practices include: use SFINAE only in pre-C++20 code; prefer concepts (C++20) for constraints ; prefer `if constexpr` (C++17) for compile-time branching within a single function ; prefer tag dispatch for category-based overloads ; keep SFINAE logic in named traits rather than inline expressions ; and use `static_assert` when only a conditional compile-time error is desired . SFINAE operates only in the immediate context; errors inside function bodies are hard errors .

**Beginner-Friendly Explanation**
Best practices for SFINAE are simple: **use concepts when you can**, **use `if constexpr` for branching inside a function**, **use tag dispatch for category selection**, **keep SFINAE logic in named traits**, and **avoid overly complex inline SFINAE**.

### Key Guidelines (Summary Table)

| Guideline | Summary | Recommendation |
|-----------|---------|----------------|
| **Prefer concepts (C++20)** | Concepts are clearer and give better errors . | `template <std::integral T>`. |
| **Use `if constexpr`** | Compile-time branching within a function . | `if constexpr (cond)`. |
| **Prefer tag dispatch** | Category-based overloads are simpler . | `true_type`/`false_type`. |
| **Name your traits** | Keep SFINAE logic in named traits . | `has_value_member<T>`. |
| **Use `static_assert`** | For conditional compile-time errors . | `static_assert(cond, "msg")`. |
| **Immediate context only** | Errors in bodies are hard errors . | Put `decltype` in return type. |

### Annotated Code Examples

**Example 1: Prefer concepts over SFINAE**

```cpp
#include <iostream>
#include <concepts>

// GOOD (C++20): concept
template <std::integral T>
T double_it(T value) { return value * 2; }

int main() {
    std::cout << double_it(5) << '\n';
    // double_it(3.14);   // clear error: double does not satisfy std::integral
}
```

**Expected Output**
```
10
```

**Why this is best practice?**  
Concepts provide clearer syntax and better error messages than SFINAE .

**Example 2: Prefer `if constexpr` for branching inside a function**

```cpp
#include <iostream>
#include <type_traits>

template <typename T>
void process(T value) {
    if constexpr (std::is_integral_v<T>) {
        std::cout << "integral: " << value << '\n';
    } else {
        std::cout << "non-integral\n";
    }
}

int main() {
    process(42);
    process(3.14);
}
```

**Expected Output**
```
integral: 42
non-integral
```

**Why this is best practice?**  
`if constexpr` eliminates most SFINAE needs for branching within a single function .

### Real-World Cases

- **Modern C++ libraries** – concepts are the default .
- **Code review** – SFINAE is flagged in favor of concepts .
- **Static analysis** – tools check for SFINAE misuse.

---

## 7. SFINAE Alternatives

### Definitions

**Core Definition**
SFINAE alternatives are modern C++ features and techniques that replace SFINAE for constraining templates, branching on types, or selecting overloads, with clearer syntax and better diagnostics.

**Technical Definition**
Where applicable, tag dispatch, `if constexpr` (C++17), and concepts (C++20) are usually preferred over SFINAE . `static_assert` is preferred if only a conditional compile-time error is wanted . Concepts, in particular, provide a first-class mechanism for constraining template parameters, with clearer syntax, composable constraints, and error messages that name the unsatisfied concept . In C++20+, concepts should be preferred; SFINAE should be used only in pre-C++20 code .

**Beginner-Friendly Explanation**
SFINAE has modern replacements. **`if constexpr`** handles branching inside a function. **Tag dispatch** handles category selection. **Concepts** handle template constraints with clear syntax. **`static_assert`** handles conditional compile-time errors. In C++20 and later, **concepts** are the best choice; SFINAE is mainly for legacy code.

### Purposes

- To replace SFINAE with clearer, safer, and more readable techniques.
- To improve error messages .
- To simplify template constraints .
- To reduce reliance on complex template metaprogramming .

### Comparison Table

| Technique | Standard | Error Messages | Readability | Use Case |
|-----------|----------|----------------|-------------|----------|
| **SFINAE (`enable_if`)** | C++11 | Terrible  | Poor  | Pre-C++20 constraints |
| **Tag dispatch** | C++98 | Good | Good | Category-based overloads |
| **`if constexpr`** | C++17 | Better  | Good  | Branching within a function |
| **Concepts** | C++20 | Excellent  | Great  | Template constraints |
| **`static_assert`** | C++11 | Good | Good | Conditional compile-time errors |

### Annotated Code Examples

**Example 1: Concepts vs. SFINAE**

```cpp
#include <iostream>
#include <type_traits>
#include <concepts>

// SFINAE approach
template <typename T>
std::enable_if_t<std::is_integral_v<T>, T>
double_it_sfinae(T value) { return value * 2; }

// Concepts approach (C++20)
template <std::integral T>
T double_it_concept(T value) { return value * 2; }

int main() {
    std::cout << double_it_sfinae(5) << '\n';   // 10
    std::cout << double_it_concept(5) << '\n';  // 10
    // double_it_sfinae(3.14);  // cryptic error
    // double_it_concept(3.14); // clear error: double does not satisfy std::integral
}
```

**Expected Output**
```
10
10
```

**Why this matters?**  
Both produce the same result, but concepts give much clearer error messages .

**Example 2: `if constexpr` instead of SFINAE**

```cpp
#include <iostream>
#include <type_traits>

template <typename T>
void process(T value) {
    if constexpr (std::is_integral_v<T>) {
        std::cout << "integral\n";
    } else {
        std::cout << "not integral\n";
    }
}

int main() {
    process(42);
    process(3.14);
}
```

**Expected Output**
```
integral
not integral
```

**Why this is an alternative?**  
`if constexpr` handles type-dependent branching without SFINAE .

### Real-World Cases

- **Standard library** – C++20 uses concepts extensively.
- **Code migration** – replacing `enable_if` with `requires` clauses .
- **New libraries** – concepts are the default constraint mechanism .

---

## References

- SFINAE (cppreference) – https://en.cppreference.com/w/cpp/language/sfinae 
- `std::enable_if` (cppreference) – https://en.cppreference.com/w/cpp/types/enable_if 
- `std::void_t` (cppreference) – https://en.cppreference.com/w/cpp/types/void_t 
- Detection idiom (cppreference) – https://en.cppreference.com/w/cpp/experimental/is_detected
- C++ Core Guidelines (templates section) – https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines
- Understand SFINAE and Know When to Replace It with Concepts (Dr-Sergey/learn_cpp) – https://github.com/Dr-Sergey/learn_cpp/blob/e2c54b8163b274e4f724168226553ae2ab4d7ddd/05_Templates_and_Generic_Programming/Understand_SFINAE_and_know_when_to_replace_it_with_Concepts.md 
- C++20 Concepts (cppreference) – https://en.cppreference.com/w/cpp/concepts
- Tag dispatch (cppreference, iterator tags) – https://en.cppreference.com/w/cpp/iterator/iterator_tags
- `constexpr if` (cppreference) – https://en.cppreference.com/w/cpp/language/if#Constexpr_if

---

This cheat sheet provides a complete, structured reference for C++ SFINAE, covering the core rule, substitution failure, `std::enable_if`, `std::void_t`, the detection idiom, best practices, and modern alternatives. Each section follows a uniform template with definitions, purposes, syntax rules, annotated examples, expected outputs, and authoritative references.