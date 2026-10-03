# Advanced Primitives & Functional Utilities (C++20 / C++23 / C++26) — Comprehensive Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
Advanced primitives and functional utilities are a set of standard library components introduced in C++20, C++23, and C++26 that provide modern, safe, and efficient abstractions for function wrapping, integer comparison, compiler hints, and bit manipulation. They replace older, error-prone idioms with type-safe, expressive alternatives.

**Technical Definition**
This family of utilities spans several headers: `<functional>` (`std::move_only_function`, `std::copyable_function`), `<utility>` (`std::cmp_equal`, `std::cmp_less`, `std::unreachable`), and `<bit>` (`std::bit_cast`, `std::has_single_bit`, `std::popcount`). These components leverage modern C++ features such as concepts, `constexpr`, `noexcept`, and template metaprogramming to provide compile-time safety and runtime efficiency. `std::move_only_function` (C++23) and `std::copyable_function` (C++26) are polymorphic function wrappers that improve upon `std::function` by supporting move-only callables and eliminating unnecessary copies. `std::cmp_*` functions (C++20) provide mathematically correct comparisons between signed and unsigned integers, avoiding the pitfalls of integral promotion. `std::unreachable` (C++23) signals to the compiler that a code path is unreachable, enabling optimisation. The bit manipulation utilities (C++20) provide portable, hardware-accelerated operations for bit-level programming.

**Beginner-Friendly Explanation**
These utilities are like upgraded tools in a toolbox. `std::move_only_function` is a lighter, faster version of `std::function` that can hold move-only objects (like `std::unique_ptr`). The `cmp_*` functions let you compare signed and unsigned numbers correctly — something that trips up even experienced programmers. `std::unreachable()` tells the compiler "this code will never run," so it can optimise better. And the bit utilities let you count bits, check for powers of two, and reinterpret memory safely — tasks that used to require error-prone manual bit twiddling.

---

### Key Characteristics

- **Move-only function wrappers**: `std::move_only_function` supports move-only callables and is itself move-only, making it more efficient than `std::function` for many use cases.
- **Copyable function wrappers**: `std::copyable_function` (C++26) provides a copyable alternative with improved const-correctness and reference qualifier support.
- **Safe integer comparison**: `std::cmp_equal`, `std::cmp_less`, etc., compare signed and unsigned integers without the surprises of built-in operators.
- **Compiler hints**: `std::unreachable` invokes undefined behaviour at a given point, allowing the compiler to eliminate dead branches or trap in debug builds.
- **Bit manipulation**: `std::bit_cast` performs type-safe bit reinterpretation; `std::has_single_bit` checks for powers of two; `std::popcount` counts set bits.
- **`constexpr` and `noexcept`**: All these utilities are `constexpr` and `noexcept` (where applicable), enabling compile-time evaluation and zero-overhead runtime use.

---

### Prerequisites

- Basic C++ syntax: templates, lambdas, `auto`, and move semantics.
- Familiarity with `std::function` and callable objects.
- Understanding of integral types, signed/unsigned arithmetic, and integral promotion.
- Basic knowledge of bitwise operations.
- A compiler supporting C++20 or later (for `std::cmp_*`, bit utilities).
- A compiler supporting C++23 or later (for `std::move_only_function`, `std::unreachable`).
- A compiler supporting C++26 or later (for `std::copyable_function`).

---

### Related Programming Areas

- **Functional programming**: Function wrappers and utilities enable higher-order functions and callbacks.
- **Generic programming**: Perfect forwarding and callable concepts are used throughout the standard library.
- **Systems programming**: Bit manipulation and compiler hints are essential for low-level code.
- **Safety and security**: Safe integer comparison prevents subtle bugs from signed/unsigned mismatches.
- **Performance engineering**: Move-only function wrappers reduce allocations and copies.

---

### Core Concepts / Features

1. **Modern Functional Wrappers** — `std::move_only_function` (C++23) and `std::copyable_function` (C++26).
2. **Safe Integer Math** — `std::cmp_equal`, `std::cmp_less`, and related functions (C++20).
3. **Compiler Optimization Hints** — `std::unreachable()` (C++23).
4. **Bit Manipulation Utilities** — `std::bit_cast`, `std::has_single_bit`, `std::popcount` (C++20).

---

## 1. Modern Functional Wrappers

### Definitions

**Core Definition**
`std::move_only_function` (C++23) is a polymorphic function wrapper that can store and invoke any move-only callable target. `std::copyable_function` (C++26) is a polymorphic function wrapper that can store and invoke any copy-constructible callable target, with improved const-correctness and reference qualifier support compared to `std::function`.

**Technical Definition**
`std::move_only_function<R(Args...)>` is a class template defined in `<functional>`. It satisfies MoveConstructible and MoveAssignable but not CopyConstructible or CopyAssignable. It supports every combination of cv-qualifiers (except `volatile`), ref-qualifiers, and `noexcept` specifiers in its template parameter, which are applied to its `operator()`. Unlike `std::function`, calling an empty `std::move_only_function` results in undefined behaviour rather than throwing `std::bad_function_call`. `std::copyable_function<R(Args...)>` is the copyable counterpart, introduced in C++26, which stores CopyConstructible callables and supports the same qualifier combinations. Both wrappers are designed to replace `std::function` in performance-critical or move-only contexts.

**Beginner-Friendly Explanation**
`std::function` is a general-purpose function wrapper, but it always copies the callable it stores. If your callable is move-only (like a lambda capturing a `std::unique_ptr`), `std::function` won't work. `std::move_only_function` solves this by allowing move-only callables and avoiding copies. `std::copyable_function` is the copyable version, but with better support for `const`, `noexcept`, and reference qualifiers — something `std::function` lacks.

---

### Purposes (all begin with "To")

- **To** store and invoke move-only callables (e.g., lambdas capturing `std::unique_ptr`) that `std::function` cannot handle.
- **To** reduce dynamic allocations and copies compared to `std::function`.
- **To** provide fine-grained control over cv-qualifiers, ref-qualifiers, and `noexcept` in the wrapper's `operator()`.
- **To** enable efficient callback registration in performance-critical systems.
- **To** replace `std::function` in move-only contexts where copying is undesirable or impossible.
- **To** offer a copyable wrapper (`std::copyable_function`) with improved const-correctness for APIs that require copy semantics.

---

### Syntax Rules and Structure

```cpp
// std::move_only_function (C++23)
template<class R, class... Args>
class move_only_function<R(Args...)>;

template<class R, class... Args>
class move_only_function<R(Args...) noexcept>;

// std::copyable_function (C++26)
template<class R, class... Args>
class copyable_function<R(Args...)>;

template<class R, class... Args>
class copyable_function<R(Args...) const>;
```

**Component breakdown:**
- `R` — return type.
- `Args...` — parameter types.
- Qualifiers (`const`, `&`, `&&`, `noexcept`) — applied to the wrapper's `operator()`.

**Common operations:**
- Construction from a callable: `std::move_only_function<int(int)> f = [](int x) { return x * 2; };`
- Invocation: `f(5)`.
- Move assignment: `f = std::move(other);`
- Emptiness check: `if (f) { ... }`

**Constraints and limitations:**
- `std::move_only_function` is not copyable; it must be moved.
- Calling an empty `std::move_only_function` is undefined behaviour (unlike `std::function`, which throws).
- `std::copyable_function` requires the stored callable to be CopyConstructible.
- Both are C++23/C++26 features; compiler support may be incomplete in older toolchains.

---

### Multiple Annotated Complete Step-by-Step Code Examples and Their Expected Outputs

#### Example 1 — `std::move_only_function` with Move-Only Callable

```cpp
// File: move_only_function_example.cpp
// Compile with: g++ -std=c++23 move_only_function_example.cpp -o move_only_function_example

#include <functional>
#include <iostream>
#include <memory>

int main() {
    // Step 1: Create a move-only lambda capturing a unique_ptr
    auto ptr = std::make_unique<int>(42);
    auto lambda = [p = std::move(ptr)](int x) {
        return *p + x;
    };

    // Step 2: Store in std::move_only_function
    std::move_only_function<int(int)> f = std::move(lambda);

    // Step 3: Invoke
    std::cout << "Result: " << f(8) << std::endl;  // 42 + 8 = 50

    // Step 4: Move to another wrapper
    std::move_only_function<int(int)> g = std::move(f);
    std::cout << "Result from g: " << g(10) << std::endl;  // 42 + 10 = 52

    // Step 5: Check emptiness
    std::cout << "f is empty: " << (f == nullptr) << std::endl;

    return 0;
}
```

**Expected Output:**
```
Result: 50
Result from g: 52
f is empty: 1
```

**Why this output:** The lambda captures a `std::unique_ptr`, making it move-only. `std::move_only_function` accepts the move-only callable. Moving `f` to `g` transfers the stored callable, leaving `f` empty. Calling `g` invokes the lambda.

---

#### Example 2 — `std::copyable_function` with Const and Reference Qualifiers (C++26)

```cpp
// File: copyable_function_example.cpp
// Compile with: g++ -std=c++26 copyable_function_example.cpp -o copyable_function_example

#include <functional>
#include <iostream>

int main() {
    // Step 1: Create a copyable function with const qualifier
    std::copyable_function<int(int) const> f = [](int x) {
        return x * 3;
    };

    // Step 2: Invoke (const-qualified operator())
    std::cout << "f(5): " << f(5) << std::endl;  // 15

    // Step 3: Copy the function
    auto g = f;
    std::cout << "g(7): " << g(7) << std::endl;  // 21

    // Step 4: Assign a new target
    f = [](int x) { return x + 100; };
    std::cout << "f(1) after reassign: " << f(1) << std::endl;  // 101

    return 0;
}
```

**Expected Output:**
```
f(5): 15
g(7): 21
f(1) after reassign: 101
```

**Why this output:** `std::copyable_function` is copyable, so `g = f` creates an independent copy. The `const` qualifier on the wrapper means the callable is invoked through a const object. Reassigning `f` replaces the stored callable without affecting `g`.

---

### Real-World Cases with Explanation

**Case 1 — Event systems**: A callback registry stores `std::move_only_function<void(Event)>` for callbacks that capture move-only resources, avoiding copies and enabling efficient dispatch.

**Case 2 — Async task queues**: Tasks captured as move-only lambdas (holding `unique_ptr` to state) are stored in `std::move_only_function` for later execution.

**Case 3 — Plugin systems**: A plugin manager stores `std::copyable_function` for factory functions that need to be copied and distributed across threads.

**Case 4 — Configuration callbacks**: `std::move_only_function` stores validation callbacks that capture `unique_ptr` to validator state, improving performance over `std::function`.

---

### References

- std::move_only_function — cppreference.com - https://en.cppreference.com/w/cpp/utility/functional/move_only_function
- std::copyable_function — cppreference.com - https://en.cppreference.com/w/cpp/utility/functional/copyable_function
- P0288R9: move_only_function — Open-Std - https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2021/p0288r9.html
- P2548R6: copyable_function — Open-Std - https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2022/p2548r6.pdf

---

## 2. Safe Integer Math

### Definitions

**Core Definition**
`std::cmp_equal`, `std::cmp_not_equal`, `std::cmp_less`, `std::cmp_greater`, `std::cmp_less_equal`, and `std::cmp_greater_equal` are C++20 function templates that compare two integers with correct mathematical semantics, regardless of their signedness.

**Technical Definition**
These functions are defined in `<utility>` (C++20). Each takes two integer arguments `t` and `u` and returns a `bool`. Unlike built-in comparison operators, they treat negative signed integers as always less than unsigned integers, avoiding the pitfalls of integral promotion and conversion. The functions are `constexpr` and `noexcept`. They are constrained to integral types (including character types and `bool`); passing a non-integral type is a compile-time error.

**Beginner-Friendly Explanation**
In C++, comparing a negative `int` with an `unsigned int` often gives surprising results because the negative value is converted to a huge unsigned number. For example, `-1 > 0u` is `true` on most systems! The `cmp_*` functions fix this: `std::cmp_greater(-1, 0u)` correctly returns `false`.

---

### Purposes (all begin with "To")

- **To** compare signed and unsigned integers with mathematically correct results.
- **To** avoid the bugs caused by integral promotion and conversion in mixed-sign comparisons.
- **To** provide a safe alternative to built-in operators when comparing values of different signedness.
- **To** enable generic code that works correctly with both signed and unsigned integer types.
- **To** support `constexpr` comparison in compile-time contexts.

---

### Syntax Rules and Structure

```cpp
template<class T, class U>
constexpr bool cmp_equal(T t, U u) noexcept;

template<class T, class U>
constexpr bool cmp_not_equal(T t, U u) noexcept;

template<class T, class U>
constexpr bool cmp_less(T t, U u) noexcept;

template<class T, class U>
constexpr bool cmp_greater(T t, U u) noexcept;

template<class T, class U>
constexpr bool cmp_less_equal(T t, U u) noexcept;

template<class T, class U>
constexpr bool cmp_greater_equal(T t, U u) noexcept;
```

**Constraints and limitations:**
- Both `T` and `U` must be integral types (including `bool` and character types); otherwise, a compile-time error occurs.
- These functions are for value comparison, not for checking whether a value fits in a target type (use `std::in_range` for that).
- Available since C++20; no runtime overhead compared to built-in operators.

---

### Multiple Annotated Complete Step-by-Step Code Examples and Their Expected Outputs

#### Example 1 — The Signed/Unsigned Comparison Bug

```cpp
// File: cmp_bug.cpp
// Compile with: g++ -std=c++20 cmp_bug.cpp -o cmp_bug

#include <iostream>
#include <utility>

int main() {
    // Step 1: Built-in comparison (surprising result)
    int a = -1;
    unsigned int b = 0;

    std::cout << "Built-in: -1 > 0u = " << (a > b) << std::endl;  // true!

    // Step 2: Safe comparison with std::cmp_greater
    std::cout << "cmp_greater(-1, 0u) = "
              << std::cmp_greater(a, b) << std::endl;  // false

    // Step 3: More examples
    std::cout << "cmp_less(-1, 0u) = "
              << std::cmp_less(a, b) << std::endl;  // true

    std::cout << "cmp_equal(0, 0u) = "
              << std::cmp_equal(0, 0u) << std::endl;  // true

    return 0;
}
```

**Expected Output:**
```
Built-in: -1 > 0u = 1
cmp_greater(-1, 0u) = 0
cmp_less(-1, 0u) = 1
cmp_equal(0, 0u) = 1
```

**Why this output:** The built-in operator converts `-1` to a huge unsigned value, making `-1 > 0u` true. `std::cmp_greater(-1, 0u)` correctly returns `false` because `-1` is mathematically less than `0`. `std::cmp_less(-1, 0u)` returns `true`.

---

### Real-World Cases with Explanation

**Case 1 — Container size comparisons**: Comparing an `int` loop counter with a `size_t` container size is a common source of bugs. `std::cmp_less` makes the comparison correct.

**Case 2 — Network protocol parsing**: Parsing length fields (often unsigned) and comparing with signed offsets or indices is made safe with `cmp_*`.

**Case 3 — Generic algorithms**: Templates that accept both signed and unsigned integer types can use `cmp_*` to compare values without special-casing.

**Case 4 — Financial calculations**: Comparing signed account balances with unsigned transaction amounts uses `cmp_*` to avoid incorrect ordering.

---

### References

- std::cmp_equal, std::cmp_not_equal, std::cmp_less, std::cmp_greater, std::cmp_less_equal, std::cmp_greater_equal — cppreference.com - https://en.cppreference.com/w/cpp/utility/intcmp
- P0586R2: Safe integral comparisons — Open-Std - https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2018/p0586r2.html
- Use std::cmp_less and std::in_range (C++20) for safe integer comparisons — Dr-Sergey/learn_cpp - https://github.com/Dr-Sergey/learn_cpp/blob/main/23_Safety_and_Security/Use_stdcmp_less_and_stdin_range_C%2B%2B20_for_safe_integer_comparisons.md

---

## 3. Compiler Optimization Hints — `std::unreachable()`

### Definitions

**Core Definition**
`std::unreachable()` is a C++23 function that invokes undefined behaviour at a given point in the program, signalling to the compiler that the code path is unreachable.

**Technical Definition**
`[[noreturn]] void unreachable();` is defined in `<utility>` (C++23). Calling it invokes undefined behaviour. Implementations may use this to optimise away impossible code branches (typically in optimised builds) or to trap the program to prevent further execution (typically in debug builds). The feature-test macro `__cpp_lib_unreachable` has the value `202202L`.

**Beginner-Friendly Explanation**
Sometimes you know a piece of code can never run — for example, the `default` case of a `switch` where all possible values are handled. `std::unreachable()` tells the compiler "I promise this never happens." The compiler can then remove that branch entirely, making the code faster. But if you're wrong and it *does* run, the program has undefined behaviour — which could mean a crash or worse.

---

### Purposes (all begin with "To")

- **To** inform the compiler that a code path is unreachable, enabling better optimisation.
- **To** eliminate dead branches in exhaustive `switch` statements and similar constructs.
- **To** trap unreachable paths in debug builds, helping catch logic errors.
- **To** provide a standard, portable replacement for compiler-specific intrinsics like `__builtin_unreachable()`.
- **To** document assumptions about program state in a way the compiler can exploit.

---

### Syntax Rules and Structure

```cpp
#include <utility>

[[noreturn]] void unreachable();
```

**Constraints and limitations:**
- Calling `std::unreachable()` when the code path is actually reachable results in undefined behaviour — this is the programmer's responsibility.
- Available since C++23.
- The implementation may use compiler-specific extensions (e.g., `__builtin_unreachable()` for GCC/Clang, `__assume(false)` for MSVC).
- Not a substitute for assertions; use `assert` for runtime checks in debug builds.

---

### Multiple Annotated Complete Step-by-Step Code Examples and Their Expected Outputs

#### Example 1 — Exhaustive Switch with `std::unreachable()`

```cpp
// File: unreachable_switch.cpp
// Compile with: g++ -std=c++23 -O2 unreachable_switch.cpp -o unreachable_switch

#include <iostream>
#include <utility>

enum class Color { Red, Green, Blue };

int color_code(Color c) {
    switch (c) {
        case Color::Red:   return 0xFF0000;
        case Color::Green: return 0x00FF00;
        case Color::Blue:  return 0x0000FF;
    }
    // All enumerators are handled, so this line is unreachable
    std::unreachable();
}

int main() {
    std::cout << std::hex << color_code(Color::Red) << std::endl;
    std::cout << std::hex << color_code(Color::Green) << std::endl;
    std::cout << std::hex << color_code(Color::Blue) << std::endl;

    return 0;
}
```

**Expected Output:**
```
ff0000
ff00
ff
```

**Why this output:** The `switch` handles all enumerators of `Color`. `std::unreachable()` tells the compiler that the code after the switch is unreachable, allowing it to optimise the function without generating a default case.

---

#### Example 2 — Unreachable in a Coroutine or State Machine

```cpp
// File: unreachable_state.cpp
// Compile with: g++ -std=c++23 -O2 unreachable_state.cpp -o unreachable_state

#include <iostream>
#include <utility>

enum class State { Idle, Running, Stopped };

void handle_state(State s) {
    switch (s) {
        case State::Idle:
            std::cout << "Idle" << std::endl;
            break;
        case State::Running:
            std::cout << "Running" << std::endl;
            break;
        case State::Stopped:
            std::cout << "Stopped" << std::endl;
            break;
    }
    std::unreachable();  // All states handled
}

int main() {
    handle_state(State::Idle);
    handle_state(State::Running);
    handle_state(State::Stopped);
    return 0;
}
```

**Expected Output:**
```
Idle
Running
Stopped
```

**Why this output:** The function handles all possible states and marks the end as unreachable. This allows the compiler to omit any implicit return or default handling.

---

### Real-World Cases with Explanation

**Case 1 — Protocol parsers**: A parser that handles all known message types can use `std::unreachable()` in the default case, optimising the hot path.

**Case 2 — Finite state machines**: After handling all states in a `switch`, `std::unreachable()` documents that no other state exists.

**Case 3 — Assertion-heavy code**: Replacing `assert(false)` with `std::unreachable()` in release builds removes the assertion overhead while preserving compiler assumptions.

**Case 4 — Low-level systems code**: Embedded and kernel code uses `std::unreachable()` to mark impossible branches, reducing code size and improving performance.

---

### References

- std::unreachable — cppreference.com - https://en.cppreference.com/w/cpp/utility/unreachable
- P0627R6: Function to mark unreachable code — Open-Std - https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2022/p0627r6.pdf
- Use std::unreachable() (C++23) to indicate unreachable code paths — Dr-Sergey/learn_cpp - https://github.com/Dr-Sergey/learn_cpp/blob/main/26_Standard_Library_New_Cpp23_26/Use_stdunreachable_C%2B%2B23_to_indicate_unreachable_code_paths.md

---

## 4. Bit Manipulation Utilities

### Definitions

**Core Definition**
The bit manipulation utilities are a set of C++20 function templates defined in `<bit>` that provide portable, efficient operations for bit-level programming: `std::bit_cast` reinterprets object representations, `std::has_single_bit` checks for powers of two, and `std::popcount` counts set bits.

**Technical Definition**
- `std::bit_cast<To>(from)` obtains a value of type `To` by reinterpreting the object representation of `from` (C++20). It requires `sizeof(To) == sizeof(From)` and both types to be trivially copyable.
- `std::has_single_bit(x)` returns `true` if `x` is a power of two (i.e., exactly one bit is set).
- `std::popcount(x)` returns the number of bits set to one in the unsigned integer `x`.

All functions are `constexpr` and `noexcept`, and are constrained to unsigned integer types (except `bit_cast`, which works with any trivially copyable types of equal size).

**Beginner-Friendly Explanation**
- `std::bit_cast` is like `reinterpret_cast` but safe: it copies the bits from one type to another without violating strict aliasing rules.
- `std::has_single_bit` checks if a number is a power of two — useful for alignment checks and hash table sizing.
- `std::popcount` counts how many bits are "on" in a number — useful for cryptography, error correction, and bioinformatics.

---

### Purposes (all begin with "To")

- **To** safely reinterpret object representations without undefined behaviour (`std::bit_cast`).
- **To** check whether a value is a power of two efficiently (`std::has_single_bit`).
- **To** count the number of set bits in an integer (`std::popcount`).
- **To** provide portable, hardware-accelerated bit operations that replace compiler intrinsics.
- **To** enable `constexpr` bit manipulation for compile-time computations.
- **To** improve code readability and correctness in low-level programming.

---

### Sub-Feature: `std::bit_cast`

#### Definitions

**Core Definition**
`std::bit_cast<To>(from)` reinterprets the object representation of `from` as a value of type `To`.

**Technical Definition**
`template<class To, class From> constexpr To bit_cast(const From& from) noexcept;` (C++20, `<bit>`). The function requires `sizeof(To) == sizeof(From)` and both types to be trivially copyable. It is the only standard way to perform type punning without undefined behaviour. The resulting value is a copy of the bits of `from` interpreted as `To`. If multiple object representations of `To` correspond to the same value, the result is unspecified.

**Beginner-Friendly Explanation**
`std::bit_cast` lets you copy the raw bits of one type into another. For example, you can convert a `float` to a `uint32_t` to inspect its IEEE 754 representation. Unlike `reinterpret_cast`, this is well-defined and portable.

#### Syntax Rules and Structure

```cpp
template<class To, class From>
constexpr To bit_cast(const From& from) noexcept;
```

**Constraints and limitations:**
- `sizeof(To)` must equal `sizeof(From)`.
- Both `To` and `From` must be trivially copyable.
- `To` must not be a pointer type (C++20 constraint).
- The result may be unspecified if `To` has multiple object representations for the same value (e.g., `bool`).

#### Annotated Code Example

```cpp
// File: bit_cast_example.cpp
// Compile with: g++ -std=c++20 bit_cast_example.cpp -o bit_cast_example

#include <bit>
#include <iostream>
#include <cstdint>

int main() {
    // Step 1: Reinterpret float as uint32_t
    float f = 3.14f;
    uint32_t bits = std::bit_cast<uint32_t>(f);
    std::cout << "Float bits: 0x" << std::hex << bits << std::endl;

    // Step 2: Reinterpret uint32_t as float
    uint32_t recovered_bits = 0x40490FDB;  // 3.14159265f
    float pi = std::bit_cast<float>(recovered_bits);
    std::cout << "Recovered float: " << pi << std::endl;

    // Step 3: Bit manipulation on the raw bits
    uint32_t sign = bits >> 31;
    uint32_t exponent = (bits >> 23) & 0xFF;
    uint32_t mantissa = bits & 0x7FFFFF;
    std::cout << "Sign: " << sign
              << ", Exponent: " << exponent
              << ", Mantissa: 0x" << mantissa << std::endl;

    return 0;
}
```

**Expected Output:**
```
Float bits: 0x40490fdb
Recovered float: 3.14159
Sign: 0, Exponent: 128, Mantissa: 0x490fdb
```

**Why this output:** `std::bit_cast<uint32_t>(f)` gives the IEEE 754 representation of `3.14f`. `std::bit_cast<float>(0x40490FDB)` recovers the float value. The bit fields are extracted manually.

---

### Sub-Feature: `std::has_single_bit` and `std::popcount`

#### Definitions

**Core Definition**
`std::has_single_bit(x)` returns `true` if `x` is a power of two. `std::popcount(x)` returns the number of 1-bits in `x`.

**Technical Definition**
`template<class T> constexpr bool has_single_bit(T x) noexcept;` and `template<class T> constexpr int popcount(T x) noexcept;` (C++20, `<bit>`). Both require `T` to be an unsigned integer type. `has_single_bit(x)` is equivalent to `popcount(x) == 1`. Implementations typically use compiler intrinsics (e.g., `__builtin_popcount`) for performance.

**Beginner-Friendly Explanation**
`has_single_bit` checks if a number is a power of two — useful for checking if a size is a power of two for hash tables or memory alignment. `popcount` counts how many bits are set — useful for algorithms that work with bitmasks.

#### Annotated Code Example

```cpp
// File: bit_ops.cpp
// Compile with: g++ -std=c++20 bit_ops.cpp -o bit_ops

#include <bit>
#include <iostream>

int main() {
    // Step 1: has_single_bit — powers of two
    for (uint32_t x : {1u, 2u, 3u, 4u, 8u, 16u, 17u}) {
        std::cout << x << " is power of two: "
                  << std::boolalpha << std::has_single_bit(x) << std::endl;
    }

    // Step 2: popcount — count set bits
    uint32_t v = 0b10110110;
    std::cout << "\npopcount(0b10110110) = " << std::popcount(v) << std::endl;

    // Step 3: Using popcount for Hamming distance
    uint32_t a = 0b1100;
    uint32_t b = 0b1010;
    std::cout << "Hamming distance: "
              << std::popcount(a ^ b) << std::endl;

    return 0;
}
```

**Expected Output:**
```
1 is power of two: true
2 is power of two: true
3 is power of two: false
4 is power of two: true
8 is power of two: true
16 is power of two: true
17 is power of two: false

popcount(0b10110110) = 5
Hamming distance: 2
```

**Why this output:** `has_single_bit` returns `true` for powers of two (1, 2, 4, 8, 16) and `false` otherwise. `popcount(0b10110110)` counts five 1-bits. The Hamming distance between `0b1100` and `0b1010` is `popcount(0b0110)` = 2.

---

### Real-World Cases with Explanation

**Case 1 — Hash table sizing**: `std::has_single_bit` checks whether a hash table's bucket count is a power of two, enabling fast modulo operations via bit masking.

**Case 2 — IEEE 754 inspection**: `std::bit_cast` extracts sign, exponent, and mantissa fields from floating-point numbers for numerical analysis.

**Case 3 — Bioinformatics**: `std::popcount` counts mutations in DNA sequences encoded as bitmasks.

**Case 4 — Error-correcting codes**: `popcount` computes the Hamming weight of error syndromes in ECC memory.

**Case 5 — Cryptography**: `popcount` and `bit_cast` are used in constant-time implementations of cryptographic primitives.

---

### References

- std::bit_cast — cppreference.com - https://en.cppreference.com/w/cpp/numeric/bit_cast
- std::has_single_bit — cppreference.com - https://en.cppreference.com/w/cpp/numeric/has_single_bit
- std::popcount — cppreference.com - https://en.cppreference.com/w/cpp/numeric/popcount
- Bit manipulation — cppreference.com - https://en.cppreference.com/w/cpp/numeric#Bit_manipulation
- P0476R2: Bit-casting object representations — Open-Std - https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2017/p0476r2.html
- P0553R4: Bit operations — Open-Std - https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2019/p0553r4.html

---

## Summary Table

| Utility | Header | Standard | Purpose | Key Feature |
|---|---|---|---|---|
| `std::move_only_function` | `<functional>` | C++23 | Move-only function wrapper | Stores move-only callables |
| `std::copyable_function` | `<functional>` | C++26 | Copyable function wrapper | Supports const/ref qualifiers |
| `std::cmp_equal` etc. | `<utility>` | C++20 | Safe integer comparison | Correct signed/unsigned semantics |
| `std::unreachable` | `<utility>` | C++23 | Mark unreachable code | Enables compiler optimisation |
| `std::bit_cast` | `<bit>` | C++20 | Bit-level reinterpretation | Safe type punning |
| `std::has_single_bit` | `<bit>` | C++20 | Check for power of two | Equivalent to `popcount(x) == 1` |
| `std::popcount` | `<bit>` | C++20 | Count set bits | Hardware-accelerated |

---

## References (Consolidated)

- std::move_only_function — cppreference.com - https://en.cppreference.com/w/cpp/utility/functional/move_only_function
- std::copyable_function — cppreference.com - https://en.cppreference.com/w/cpp/utility/functional/copyable_function
- std::cmp_equal, std::cmp_less, etc. — cppreference.com - https://en.cppreference.com/w/cpp/utility/intcmp
- std::unreachable — cppreference.com - https://en.cppreference.com/w/cpp/utility/unreachable
- std::bit_cast — cppreference.com - https://en.cppreference.com/w/cpp/numeric/bit_cast
- std::has_single_bit — cppreference.com - https://en.cppreference.com/w/cpp/numeric/has_single_bit
- std::popcount — cppreference.com - https://en.cppreference.com/w/cpp/numeric/popcount
- P0288R9: move_only_function — Open-Std - https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2021/p0288r9.html
- P2548R6: copyable_function — Open-Std - https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2022/p2548r6.pdf
- P0586R2: Safe integral comparisons — Open-Std - https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2018/p0586r2.html
- P0627R6: Function to mark unreachable code — Open-Std - https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2022/p0627r6.pdf
- P0476R2: Bit-casting object representations — Open-Std - https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2017/p0476r2.html
- P0553R4: Bit operations — Open-Std - https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2019/p0553r4.html