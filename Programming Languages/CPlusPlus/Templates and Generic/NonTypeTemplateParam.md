# C++ Non-Type Template Parameters (NTTPs) — Comprehensive Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
A non-type template parameter (NTTP) is a template parameter that is a constant value rather than a type. It allows templates to be parameterized on compile-time values such as integers, enums, pointers, and (since C++20) floating-point values and certain class types.

**Technical Definition**
A non-type template parameter is declared in a template parameter list with an optional type name followed by a parameter name, e.g., `template <int N>`. Since C++20, the type of a non-type template parameter must be a structural type, which is one of: an lvalue reference type, an integral type, a pointer type, a pointer-to-member type, an enumeration type, a floating-point type (since C++20), a captureless lambda closure type (since C++20), or a non-closure literal class type whose base classes and non-static data members are all public, non-mutable, and themselves structural types or arrays thereof (since C++20). Array and function types are adjusted to pointer types.

**Beginner-Friendly Explanation**
A non-type template parameter lets you pass a value to a template, not just a type. For example, `template <int N> struct Array { int data[N]; };` allows you to write `Array<10>` to create an array of 10 integers. The value `10` is known at compile time, so the compiler can allocate the exact amount of memory. Before C++20, only integers, enums, pointers, and references could be NTTPs. C++20 expanded this to include floating-point numbers and certain class types, enabling powerful compile-time programming.

---

### Key Characteristics

- **Compile-time constants**: NTTP arguments must be constant expressions.
- **Integral parameters**: Integers and enums are the most common NTTPs, used for sizes, bounds, and flags.
- **`auto` deduction (C++17)**: The type of an NTTP can be deduced using `auto`, allowing type-flexible constants.
- **Floating-point NTTPs (C++20)**: `double`, `float`, and `long double` can be used as NTTPs.
- **Class-type NTTPs (C++20)**: Structural types (literal classes with all public, non-mutable members) can be used as NTTPs.
- **Partial ordering**: NTTPs participate in partial ordering of class and function templates.
- **Parameter packs**: Non-type template parameter packs allow a variable number of constant arguments.

---

### Prerequisites

- Solid understanding of templates (function and class templates).
- Familiarity with template parameter lists and instantiation.
- Knowledge of constant expressions and `constexpr`.
- (For `auto` NTTPs) C++17 or later.
- (For floating-point and class-type NTTPs) C++20 or later.

---

### Related Programming Areas

- **Template metaprogramming** (compile-time computation).
- **Compile-time data structures** (fixed-size arrays, matrices).
- **Policy-based design** (selecting policies via integer or enum NTTPs).
- **Concepts and constraints** (C++20, constraining NTTPs).
- **Structural types** (C++20, class types as NTTPs).

---

### Core Concepts / Features

1. **Fixed Bounds** — Using integral parameters (integers, enums) for compile-time buffer and array sizing.
2. **Type Deduction** — Leveraging `auto` inside template parameter lists for type-flexible NTTP constants.
3. **Modern Object Arguments** — Expanding parameters to include floating-point values, literal classes, and C++20 structural types (e.g., compile-time fixed strings).

---

## 1. Fixed Bounds (Integral and Enum NTTPs)

### Definitions

**Core Definition**
Fixed bounds use integral or enumeration NTTPs to specify compile-time sizes, limits, or flags that control the structure and behavior of a template.

**Technical Definition**
An integral non-type template parameter is declared with an integral or enumeration type, e.g., `template <int N>` or `template <std::size_t N>`. The argument must be a converted constant expression of the parameter's type. The parameter name can be used in constant expressions within the template body, such as array bounds, loop limits, and `static_assert` conditions.

**Beginner-Friendly Explanation**
An integral NTTP is like a compile-time number that a template can use to size arrays or set limits. For example, `template <int Size> struct Buffer { int data[Size]; };` lets you write `Buffer<100>` to create a buffer of exactly 100 integers. The compiler knows the size at compile time, so no dynamic allocation is needed.

---

### Purposes (all begin with "To")

- **To** specify compile-time array sizes and buffer capacities without dynamic allocation.
- **To** parameterize templates on loop bounds, dimensions, or iteration counts.
- **To** select template behavior based on integer or enum flags.
- **To** enable zero-overhead abstractions where size information is known at compile time.
- **To** support compile-time validation of bounds using `static_assert`.

---

### Syntax Rules and Structure

#### Complete General Syntax with Breakdown

```cpp
template <int N>
class Array {
    int data[N];
};
```

**Component breakdown:**
- `template <int N>` — template parameter list with an integral NTTP `N`.
- `class Array` — class template using `N` as an array bound.
- `int data[N]` — array whose size is the NTTP value.

**Enum NTTP:**
```cpp
enum class Mode { Fast, Safe };
template <Mode M>
void process() { /* ... */ }
```

#### Syntax Rules

- The NTTP type must be an integral or enumeration type (until C++20) or a structural type (C++20).
- The argument must be a converted constant expression of the parameter's type.
- The NTTP name is an unmodifiable prvalue within the template body (unless it is an lvalue reference or a class type since C++20).
- Default template arguments can be provided: `template <int N = 10>`.
- Non-type template parameter packs are allowed: `template <int... Ns>`.

#### Constraints and Limitations

- Integral NTTPs cannot be floating-point (until C++20).
- The NTTP value must be known at compile time; runtime values cannot be used.
- The NTTP type must be complete if it is a class type.
- Array and function types are adjusted to pointer types.

---

### Multiple Annotated Complete Step-by-Step Code Examples and Their Expected Outputs

#### Example 1 — Fixed-Size Array with Integral NTTP

```cpp
#include <iostream>
#include <array>

// Step 1: Integral NTTP for size
template <std::size_t N>
struct FixedBuffer {
    std::array<int, N> data{};

    void fill_sequential() {
        for (std::size_t i = 0; i < N; ++i) data[i] = static_cast<int>(i + 1);
    }

    void print(const char* label) const {
        std::cout << label << " [";
        for (std::size_t i = 0; i < N; ++i) {
            if (i > 0) std::cout << ", ";
            std::cout << data[i];
        }
        std::cout << "] (size=" << N << ")\n";
    }

    constexpr std::size_t size() const { return N; }
};

int main() {
    // Step 2: Instantiate with size 5
    FixedBuffer<5> buf;
    buf.fill_sequential();
    buf.print("buf5");

    // Step 3: Instantiate with size 3
    FixedBuffer<3> buf3;
    buf3.fill_sequential();
    buf3.print("buf3");

    // Step 4: Compile-time size check
    static_assert(buf.size() == 5);
    static_assert(buf3.size() == 3);

    return 0;
}
```

**Expected Output:**
```
buf5 [1, 2, 3, 4, 5] (size=5)
buf3 [1, 2, 3] (size=3)
```

**Why this output:** The integral NTTP `N` is used as the array size. `FixedBuffer<5>` and `FixedBuffer<3>` are distinct instantiations with different `N` values. The `fill_sequential` function uses `N` as the loop bound, and `print` displays the data and size. The `static_assert` verifies the compile-time size.

---

#### Example 2 — Enum NTTP for Mode Selection

```cpp
#include <iostream>

// Step 1: Enum for mode selection
enum class Mode { Fast, Safe };

// Step 2: Enum NTTP
template <Mode M>
void process(int value) {
    if constexpr (M == Mode::Fast) {
        std::cout << "Fast processing: " << value * 2 << std::endl;
    } else {
        std::cout << "Safe processing: " << value + 10 << std::endl;
    }
}

int main() {
    // Step 3: Instantiate with different modes
    process<Mode::Fast>(5);
    process<Mode::Safe>(5);
    return 0;
}
```

**Expected Output:**
```
Fast processing: 10
Safe processing: 15
```

**Why this output:** The enum NTTP `M` selects the code path at compile time via `if constexpr`. `process<Mode::Fast>(5)` uses the fast path, while `process<Mode::Safe>(5)` uses the safe path. The compiler generates two distinct functions, and the branch not taken is discarded.

---

### Real-World Cases with Explanation

**Case 1 — `std::array<T, N>`:** The standard library's `std::array` uses an integral NTTP `N` to specify the number of elements. This enables stack allocation and compile-time size checking.

**Case 2 — `std::integer_sequence`:** `std::integer_sequence<T, Ints...>` uses a non-type template parameter pack of integers to represent a compile-time sequence, used in index-sequence expansions for tuple and function-call unpacking.

**Case 3 — Fixed-size matrix libraries:** Linear algebra libraries use two integral NTTPs (rows and columns) to define matrix dimensions at compile time, enabling stack allocation and optimized loops.

**Case 4 — Policy selection:** A library might use an enum NTTP to select a threading policy (e.g., `SingleThreaded` vs. `MultiThreaded`), generating different code paths without runtime branching.

---

### References

- Template parameters and template arguments — cppreference.com - https://en.cppreference.com/w/cpp/language/template_parameters
- Non-type template parameter — Microsoft Learn (C2993) - https://learn.microsoft.com/en-us/cpp/error-messages/compiler-errors-2/compiler-error-c2993
- `std::integer_sequence` — cppreference.com - https://en.cppreference.com/w/cpp/utility/integer_sequence

---

## 2. Type Deduction (`auto` NTTPs)

### Definitions

**Core Definition**
`auto` non-type template parameters (C++17) allow the type of an NTTP to be deduced from the template argument, eliminating the need to specify the type explicitly.

**Technical Definition**
A non-type template parameter can be declared with `auto` or `decltype(auto)` as its type. When the template is instantiated, the compiler deduces the type of the argument and uses it as the parameter's type. This is particularly useful for passing constants of different types to the same template without specifying the type each time. The deduction follows the same rules as `auto` in variable declarations.

**Beginner-Friendly Explanation**
Before C++17, if you wanted a template to accept a constant of any integral type, you had to write separate templates for `int`, `long`, `unsigned`, etc., or use a type template parameter alongside a non-type parameter. With `auto`, you write `template <auto N>` and the compiler figures out the type of `N` from the argument. For example, `template <auto N> struct Constant { static constexpr auto value = N; };` can be instantiated with `Constant<42>` (int), `Constant<42u>` (unsigned), or `Constant<'a'>` (char).

---

### Purposes (all begin with "To")

- **To** write a single template that accepts constants of any integral or enumeration type.
- **To** deduce the type of a non-type template argument automatically.
- **To** avoid duplicating templates for different integer types.
- **To** enable generic constants that adapt their type to the argument.
- **To** support `decltype(auto)` for deducing the exact type of an expression.

---

### Syntax Rules and Structure

#### Complete General Syntax with Breakdown

```cpp
template <auto N>
struct Constant {
    static constexpr auto value = N;
};
```

**Component breakdown:**
- `template <auto N>` — NTTP with deduced type `auto`.
- `static constexpr auto value = N;` — stores the value with the deduced type.

**`decltype(auto)` NTTP:**
```cpp
template <decltype(auto) N>
struct X { /* ... */ };
```

#### Syntax Rules

- `auto` NTTPs are available since C++17.
- The deduced type must be one of the allowed NTTP types (integral, enumeration, pointer, reference, etc.).
- `decltype(auto)` can be used to deduce reference types.
- The `auto` can be qualified (e.g., `const auto`, `auto*`).
- `auto` NTTPs participate in partial ordering and overload resolution.

#### Constraints and Limitations

- `auto` NTTPs cannot be used with floating-point types until C++20.
- Deduction follows `auto` rules; array and function types are adjusted to pointers.
- The deduced type must be consistent across all uses of the template.
- `auto` NTTPs cannot be used in C++14 or earlier.

---

### Multiple Annotated Complete Step-by-Step Code Examples and Their Expected Outputs

#### Example 1 — `auto` NTTP for Type-Flexible Constants

```cpp
#include <iostream>
#include <typeinfo>

// Step 1: auto NTTP
template <auto N>
struct Constant {
    static constexpr auto value = N;

    void print() const {
        std::cout << "Value: " << value
                  << " (type: " << typeid(N).name() << ")" << std::endl;
    }
};

int main() {
    // Step 2: Deduce int
    Constant<42> c1;
    c1.print();

    // Step 3: Deduce unsigned
    Constant<42u> c2;
    c2.print();

    // Step 4: Deduce char
    Constant<'a'> c3;
    c3.print();

    // Step 5: Deduce long
    Constant<42L> c4;
    c4.print();

    return 0;
}
```

**Expected Output (implementation-defined type names):**
```
Value: 42 (type: i)
Value: 42 (type: j)
Value: 97 (type: c)
Value: 42 (type: l)
```

**Why this output:** The compiler deduces the type of `N` from the template argument. `42` is `int` (`i`), `42u` is `unsigned int` (`j`), `'a'` is `char` (`c`, printed as its integer value 97), and `42L` is `long` (`l`). The `auto` NTTP allows a single template definition to work with all these types.

---

#### Example 2 — `auto` NTTP with Pointer and Member Pointer

```cpp
#include <iostream>

// Step 1: auto NTTP for pointer types
template <auto Ptr>
void print_pointer() {
    std::cout << "Pointer value: " << *Ptr << std::endl;
}

int global_value = 42;

int main() {
    // Step 2: Pass a pointer to an object
    print_pointer<&global_value>();
    return 0;
}
```

**Expected Output:**
```
Pointer value: 42
```

**Why this output:** The `auto` NTTP `Ptr` deduces the type of the pointer `&global_value`, which is `int*`. The template can dereference the pointer at compile time (since the pointer is a constant expression) and print the value.

---

### Real-World Cases with Explanation

**Case 1 — Compile-time constants:** A `Constant<auto N>` template can store a compile-time value with its exact type, useful for metaprogramming and type-safe constant propagation.

**Case 2 — Function pointer NTTPs:** `template <auto F> void call() { F(); }` can take a function pointer as an NTTP, enabling compile-time dispatch to different functions.

**Case 3 — Member pointer NTTPs:** `template <auto Member> struct Accessor { /* ... */ };` can take a pointer-to-member and use it to access members of objects at compile time.

**Case 4 — Generic wrappers:** A wrapper template `template <auto Value> struct Tag {};` can be used to create unique types based on compile-time values, a technique used in type-level programming.

---

### References

- Declaring non-type template parameters with auto (P0127R2) — WG21 - https://isocpp.open-std.org/JTC1/SC22/WG21/docs/papers/2016/p0127r2.html
- C++17 `auto` non-type template parameters — GCC Patch - https://gcc.gnu.org/pipermail/gcc-patches/2016-November/461798.html
- Template parameters and template arguments — cppreference.com - https://en.cppreference.com/w/cpp/language/template_parameters

---

## 3. Modern Object Arguments (Floating-Point and Class-Type NTTPs)

### Definitions

**Core Definition**
Modern object arguments refer to NTTPs of floating-point type and structural class type (C++20), enabling compile-time parameters that are not limited to integers and pointers.

**Technical Definition**
Since C++20, the type of a non-type template parameter must be a structural type, which includes floating-point types and non-closure literal class types. A class type is structural if all base classes and non-static data members are public and non-mutable, and their types are structural types or arrays thereof. This enables the use of floating-point constants, compile-time strings, and other class objects as template arguments. The argument must satisfy strong structural equality: for class types, the comparison uses member-wise comparison without user-defined `operator==`.

**Beginner-Friendly Explanation**
Before C++20, you couldn't use a `double` or a custom class object as a template argument. C++20 changed that. Now you can write `template <double D> struct Constant {};` and instantiate it with `Constant<3.14>`. You can also create a `FixedString` class that holds a compile-time string and use it as an NTTP: `template <FixedString S> struct Tag {};` and write `Tag<"hello">`. This enables powerful compile-time programming with strings and floating-point values.

---

### Purposes (all begin with "To")

- **To** use floating-point constants as template arguments for numerical algorithms.
- **To** use compile-time string literals as template arguments for type tags and metadata.
- **To** parameterize templates on class objects that have strong structural equality.
- **To** enable compile-time string manipulation and pattern matching.
- **To** provide a type-safe alternative to macro-based string constants.

---

### Syntax Rules and Structure

#### Complete General Syntaxes with Breakdowns

**Floating-point NTTP:**
```cpp
template <double D>
struct Constant {
    static constexpr double value = D;
};
```

**Class-type NTTP (structural type):**
```cpp
struct FixedString {
    char data[32]{};
    std::size_t len{};
    constexpr FixedString(const char* s) : len{0} {
        while (s[len]) { data[len] = s[len]; ++len; }
    }
};

template <FixedString Tag>
struct LogTag {
    static constexpr auto name = Tag;
};
```

#### Syntax Rules

- Floating-point NTTPs require C++20.
- Class-type NTTPs require C++20 and must be structural types.
- The class must have all public, non-mutable base classes and non-static data members.
- The class must not have user-provided `operator==` for any subobject.
- The class must have a `constexpr` constructor or be an aggregate.
- The argument must satisfy strong structural equality.

#### Constraints and Limitations

- Floating-point NTTPs cannot be used in C++17 or earlier.
- Class-type NTTPs cannot have private members or user-defined `operator==`.
- The class must be a literal type (constexpr constructor, trivial destructor).
- `std::string` cannot be used directly as an NTTP; a fixed-size string wrapper is required.
- The class must be comparable at compile time.

---

### Multiple Annotated Complete Step-by-Step Code Examples and Their Expected Outputs

#### Example 1 — Floating-Point NTTP

```cpp
#include <iostream>

// Step 1: Floating-point NTTP (C++20)
template <double D>
struct Constant {
    static constexpr double value = D;
    void print() const {
        std::cout << "Value: " << value << std::endl;
    }
};

int main() {
    // Step 2: Instantiate with double
    Constant<3.14> c1;
    c1.print();

    // Step 3: Instantiate with another double
    Constant<2.71> c2;
    c2.print();

    // Step 4: Compile-time comparison
    static_assert(Constant<3.14>::value > Constant<2.71>::value);

    return 0;
}
```

**Expected Output:**
```
Value: 3.14
Value: 2.71
```

**Why this output:** The floating-point NTTP `D` is used as a compile-time constant. `Constant<3.14>` and `Constant<2.71>` are distinct instantiations. The `static_assert` verifies the compile-time comparison. This was not possible before C++20.

---

#### Example 2 — Compile-Time Fixed String as NTTP

```cpp
#include <iostream>
#include <string_view>

// Step 1: Structural type for fixed strings
struct FixedString {
    char data[32]{};
    std::size_t len{};

    constexpr FixedString(const char* s) : len{0} {
        while (s[len]) { data[len] = s[len]; ++len; }
    }

    constexpr std::string_view view() const {
        return {data, len};
    }
};

// Step 2: Class-type NTTP
template <FixedString Tag>
struct LogTag {
    static constexpr auto name = Tag;
    void print() const {
        std::cout << "Tag: " << Tag.view() << std::endl;
    }
};

// Step 3: Helper for string literal to FixedString
template <std::size_t N>
LogTag(FixedString) -> LogTag<FixedString("")>;  // simplified

int main() {
    // Step 4: Instantiate with string literal
    LogTag<"INFO"> infoTag;
    infoTag.print();

    LogTag<"ERROR"> errorTag;
    errorTag.print();

    return 0;
}
```

**Expected Output:**
```
Tag: INFO
Tag: ERROR
```

**Why this output:** The `FixedString` class is a structural type: all members are public and non-mutable, and it has a `constexpr` constructor. The `LogTag` template takes a `FixedString` NTTP. `LogTag<"INFO">` creates a `FixedString` from the string literal and uses it as a template argument. The `view()` method returns a `std::string_view` for printing. This demonstrates compile-time string handling without macros.

---

### Real-World Cases with Explanation

**Case 1 — Compile-time string tags:** Libraries use fixed-string NTTPs to create type-safe tags for logging, serialization, and configuration. For example, a `Tag<"database">` type can be used to select a connection at compile time.

**Case 2 — Floating-point constants in numerical libraries:** A numerical library can use a floating-point NTTP to specify a compile-time tolerance or constant, e.g., `template <double Epsilon> struct ApproxEqual { /* ... */ };`.

**Case 3 — Pattern matching in parsers:** A parser can use fixed-string NTTPs to match keywords at compile time, e.g., `template <FixedString Keyword> struct KeywordParser { /* ... */ };`.

**Case 4 — `std::format` compile-time checks:** C++20's `std::format` uses compile-time format string checking. While the standard implementation uses `consteval`, the underlying technique involves compile-time string processing that could be implemented with structural-type NTTPs.

---

### References

- P1907R1: Inconsistencies with non-type template parameters — WG21 - https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2019/p1907r1.html
- Template parameters and template arguments — cppreference.com - https://en.cppreference.com/w/cpp/language/template_parameters
- Use Non-Type Template Parameters (NTTPs) Including Class Types (C++20) — Dr-Sergey/learn_cpp (GitHub) - https://github.com/Dr-Sergey/learn_cpp
- Compiler Error C2993 — Microsoft Learn - https://learn.microsoft.com/en-us/cpp/error-messages/compiler-errors-2/compiler-error-c2993

---

## Summary Table of Core Concepts

| Concept | Key Mechanism | Primary Benefit | Key Constraint |
|---|---|---|---|
| Fixed Bounds | `template <int N>` | Compile-time sizing, zero-overhead arrays | Integral/enum only (until C++20) |
| Type Deduction | `template <auto N>` | Type-flexible constants, less duplication | C++17; no floating-point until C++20 |
| Modern Object Arguments | `template <double D>` / `template <FixedString S>` | Floating-point and string constants | C++20; structural types only |

---

## References (Consolidated)

- Template parameters and template arguments — cppreference.com - https://en.cppreference.com/w/cpp/language/template_parameters
- Compiler Error C2993 — Microsoft Learn - https://learn.microsoft.com/en-us/cpp/error-messages/compiler-errors-2/compiler-error-c2993
- C++17 `auto` non-type template parameters — GCC Patch - https://gcc.gnu.org/pipermail/gcc-patches/2016-November/461798.html
- Declaring non-type template parameters with auto (P0127R2) — WG21 - https://isocpp.open-std.org/JTC1/SC22/WG21/docs/papers/2016/p0127r2.html
- P1907R1: Inconsistencies with non-type template parameters — WG21 - https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2019/p1907r1.html
- Use Non-Type Template Parameters (NTTPs) Including Class Types (C++20) — Dr-Sergey/learn_cpp (GitHub) - https://github.com/Dr-Sergey/learn_cpp
- `std::integer_sequence` — cppreference.com - https://en.cppreference.com/w/cpp/utility/integer_sequence