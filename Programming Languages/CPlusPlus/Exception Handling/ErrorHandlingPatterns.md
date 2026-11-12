# C++ Error Handling Patterns — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
Error handling patterns are the structured techniques a C++ program uses to detect, report, and recover from failures, ranging from lightweight return-value codes to exception propagation and the modern `std::expected` vocabulary type.

**Technical Definition**
C++ provides three principal error-handling mechanisms: **exceptions** (`throw`/`try`/`catch`, zero-cost on the happy path via table-based unwinding), **error codes** (`std::error_code`, a lightweight `{int value, error_category*}` pair), and **`std::expected<T, E>`** (C++23, a value-or-error sum type with monadic chaining). `std::error_code` represents a platform-dependent error value tied to a category, while `std::error_condition` represents a platform-independent error condition that `error_code` values can be tested against. Each mechanism has distinct trade-offs in performance, expressiveness, and composability.

**Beginner-Friendly Explanation**
Imagine three ways to tell someone “that didn’t work.” You can hand them a small numbered ticket (`error_code`), you can shout and have everyone nearby immediately stop and listen (exceptions), or you can hand them a box that contains either the result or the reason it failed (`expected`). Each is useful in different situations: tickets are fast but easy to ignore, shouting is impossible to ignore but slow, and the box is both fast and hard to ignore—if you open it properly.

### Key Characteristics

- **Three mechanisms, one goal** – exceptions, error codes, and `std::expected` all report failures, but with different performance and composability profiles.
- **Zero-overhead exceptions** – modern compilers use table-based unwinding; the happy path pays no runtime cost for the existence of `try`/`catch`.
- **Error codes are non-throwing and ABI-stable** – `std::error_code` is a value type (an integer plus a category pointer), making it suitable for C ABI boundaries and `noexcept` APIs.
- **`std::expected` is never valueless** – it always holds either the expected value or the unexpected error, making it safer than pointer-or-null patterns.
- **`error_condition` enables portable testing** – platform-specific `error_code` values from different categories can compare equal to a generic `error_condition` representing the same underlying failure.
- **Consistency matters** – mixing mechanisms freely within a subsystem leads to unhandled errors; choose one strategy per boundary.

### Prerequisites

- Basic C++ syntax (functions, classes, templates, references).
- Understanding of `try`/`catch`/`throw` and the exception hierarchy (`std::exception`).
- Familiarity with `noexcept` and exception safety guarantees.
- Knowledge of C++23 features (if using `std::expected`).
- Awareness of C ABI boundaries and `noexcept` API design.

### Related Programming Areas

- **Exception fundamentals** – `throw`, `try`, `catch`, custom exceptions, and stack unwinding.
- **Exception safety** – basic, strong, and nothrow guarantees.
- **`noexcept`** – the specifier that enforces nothrow APIs.
- **Error categories** – `std::error_category`, `std::system_category`, `std::generic_category`.
- **Monadic programming** – `std::expected`’s `and_then`, `transform`, and `or_else` operations.

### Core Concepts / Features

1. Error codes (the general pattern)
2. `std::error_code`
3. `std::error_condition`
4. `std::expected` (C++23)
5. Exceptions vs. error codes
6. Error handling best practices

---

## 1. Error Codes (The General Pattern)

### Definitions

**Core Definition**
An error code is a return value (typically an integer or enumeration) that communicates whether an operation succeeded or failed, and if it failed, what kind of failure occurred.

**Technical Definition**
Error-code-based error handling returns a status value from a function; the caller must explicitly check this value at the call site and propagate it manually up the call stack if it cannot handle it locally. In C++11 and later, the standard library provides `std::error_code` as a structured, category-aware error-code type, but plain `int` or `enum` codes remain common in C-style APIs and embedded systems.

**Beginner-Friendly Explanation**
An error code is like a receipt from a vending machine: it prints “OK” if the snack dropped, or “OUT OF STOCK” if it didn’t. You have to look at the receipt yourself; nobody forces you to.

### Purposes

- To report errors without the overhead of throwing exceptions.
- To remain compatible with C ABI boundaries and `-fno-exceptions` builds.
- To allow callers to ignore errors deliberately (though this is risky).
- To provide near-zero error-path cost (a simple return value).

### Syntax Rules and Structure

**General Syntax**
```cpp
enum class Status { ok, not_found, permission_denied };

Status read_file(const char* path) {
    if (!exists(path)) return Status::not_found;
    // ...
    return Status::ok;
}

// Call site
Status s = read_file("data.txt");
if (s != Status::ok) { /* handle */ }
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| Return type | `int`, `enum`, or `enum class` representing the status. |
| Success value | Typically `0` or `Status::ok`. |
| Call-site check | `if (s != ok) { ... }` — must be written manually. |
| Propagation | Manual: `return s;` at each layer if unhandled. |

**Syntax Rules**
- The success value should be documented and checked consistently.
- Errors must be propagated explicitly through every intermediate function.
- A function should not return a valid-looking value on failure (e.g., `0` for a file read).

**Constraints and Limitations**
- Errors are easy to ignore accidentally; the compiler does not force a check.
- Manual propagation is verbose and error-prone across many layers.
- No automatic stack unwinding or resource cleanup tied to the error path.

### Annotated Code Example

```cpp
#include <iostream>

enum class Status { ok, not_found, io_error };

Status open_file(const char* path) {
    if (path == nullptr) return Status::not_found;
    // simulate a failure
    return Status::io_error;
}

int main() {
    Status s = open_file("data.txt");
    if (s != Status::ok) {
        std::cout << "Failed with status " << static_cast<int>(s) << '\n';
        return 1;
    }
    std::cout << "Success\n";
}
```

**Expected Output**
```
Failed with status 2
```

**Why this output?**  
`open_file` returns `Status::io_error`; the caller explicitly checks the status and reports the numeric value of the failure.

### Real-World Cases

- **POSIX and Win32 APIs** – return `int` error codes or set `errno` / `GetLastError()`.
- **Embedded and real-time systems** – exceptions may be disabled (`-fno-exceptions`) or have non-deterministic latency.
- **C ABI boundaries** – exceptions cannot cross `extern "C"` boundaries safely.

---

## 2. `std::error_code`

### Definitions

**Core Definition**
`std::error_code` is a lightweight, non-throwing value type that holds a platform-dependent error code value together with a pointer to an `std::error_category` that gives the value meaning.

**Technical Definition**
Defined in `<system_error>` since C++11, `std::error_code` represents a platform-dependent error code value. Each object holds an error code value originating from the operating system or some low-level interface and a pointer to an `std::error_category` corresponding to that interface. Error code values are not required to be unique across different categories. It is a value type storing an integer error number and a category that gives meaning to the error number.

**Beginner-Friendly Explanation**
`std::error_code` is like a labelled error ticket. The number alone is meaningless without the label (the category). For example, the number `2` might mean “file not found” in the generic category, but something else in the Windows system category.

### Purposes

- To provide a structured, category-aware error code that is ABI-stable (`int` + pointer).
- To interoperate with OS and low-level APIs without throwing exceptions.
- To serve as the standard error-reporting format across C++ libraries.
- To enable comparison against portable `std::error_condition` values.

### Syntax Rules and Structure

**General Syntax**
```cpp
#include <system_error>

std::error_code ec;                                   // default: value 0, system_category
std::error_code ec(value, category);                  // explicit value + category
std::error_code ec = std::make_error_code(std::errc::not_supported); // from errc
```

**Component Breakdown**

| Operation | Description |
|-----------|-------------|
| `ec.value()` | The integer error value. |
| `ec.category()` | The `error_category` that interprets the value. |
| `ec.message()` | An explanatory string. |
| `operator bool` | `true` if `value() != 0`. |
| `ec == condition` | Compares against an `error_condition` or `errc`. |

**Syntax Rules**
- A default-constructed `error_code` has value `0` in `system_category` and represents success.
- `error_code` is `noexcept` for construction, assignment, and comparison.
- `std::make_error_code(std::errc)` creates an `error_code` from a portable `errc` value.

**Constraints and Limitations**
- The meaning of a value depends entirely on its category; two different categories may use the same integer for different errors.
- `error_code` is not a replacement for exceptions in all scenarios; it requires manual checking.

### Annotated Code Example

```cpp
#include <iostream>
#include <system_error>

int main() {
    std::error_code ec = std::make_error_code(std::errc::file_exists);
    std::cout << "Value: " << ec.value() << '\n';
    std::cout << "Category: " << ec.category().name() << '\n';
    std::cout << "Message: " << ec.message() << '\n';
    std::cout << std::boolalpha << "Has error: " << (bool)ec << '\n';
}
```

**Expected Output** (message is implementation-defined)
```
Value: 17
Category: generic
Message: File exists
Has error: true
```

**Why this output?**  
`make_error_code(std::errc::file_exists)` creates an `error_code` in the generic category; `value()`, `category().name()`, and `message()` expose its components.

### Real-World Cases

- **Filesystem library** – `std::filesystem::exists(path, ec)` uses `error_code` overloads for non-throwing operation.
- **Networking (Asio, etc.)** – completion handlers receive `std::error_code`.
- **Multi-library projects** – `error_code` is the standard interop format for errors between C++ libraries.

---

## 3. `std::error_condition`

### Definitions

**Core Definition**
`std::error_condition` is a platform-independent value that identifies an error condition, used to test whether an `std::error_code` represents a particular kind of failure.

**Technical Definition**
Defined in `<system_error>` since C++11, `std::error_condition` holds a platform-independent value identifying an error condition. Like `std::error_code`, it is uniquely identified by an integer value and an `std::error_category`, but unlike `error_code`, the value is not platform-dependent. A typical implementation holds one integer data member and a pointer to an `error_category`. The comparison between an `error_code` and an `error_condition` is defined by their error categories: a generic-category condition may compare equal to a system-category code if they represent the same kind of error.

**Beginner-Friendly Explanation**
`std::error_condition` is the “universal translation” of an error. Different platforms use different error codes, but they all map to the same portable condition. You use `error_condition` to ask “is this error a ‘file not found’ error?” without caring which platform-specific code was used.

### Purposes

- To test an `error_code` against a portable condition (e.g., `file_exists`, `permission_denied`).
- To abstract over platform-specific error codes.
- To allow libraries to define their own portable error conditions.
- To enable comparisons like `ec == std::errc::file_exists`.

### Syntax Rules and Structure

**General Syntax**
```cpp
#include <system_error>

std::error_condition cond = std::make_error_condition(std::errc::file_exists);
if (ec == cond) { /* ... */ }

// Idiomatic:
if (ec == std::errc::file_exists) { /* ... */ }
```

**Component Breakdown**

| Operation | Description |
|-----------|-------------|
| `cond.value()` | The integer value. |
| `cond.category()` | The category. |
| `cond.message()` | Explanatory string. |
| `ec == cond` | `true` if the code and condition represent the same error. |

**Syntax Rules**
- An `std::errc` value implicitly converts to an `error_condition` for comparison with an `error_code`.
- `error_condition` values are platform-independent; `error_code` values are platform-dependent.

**Constraints and Limitations**
- `error_condition` is for testing, not for storing the original error; use `error_code` to preserve the exact platform error.
- Comparisons rely on the category’s `equivalent()` logic; custom categories must implement this correctly.

### Annotated Code Example

```cpp
#include <iostream>
#include <system_error>

int main() {
    // Simulate a platform error code (e.g., Windows ERROR_FILE_EXISTS = 80)
    std::error_code ec(80, std::system_category());

    // Test against a portable condition
    if (ec == std::errc::file_exists) {
        std::cout << "The file already exists (portable test)\n";
    }

    std::error_condition cond = std::make_error_condition(std::errc::file_exists);
    std::cout << std::boolalpha << (ec == cond) << '\n';
}
```

**Expected Output** (implementation-dependent)
```
The file already exists (portable test)
true
```

**Why this output?**  
The system-category code `80` represents “file exists” on Windows; the generic-category condition `std::errc::file_exists` maps to the same logical error, so the comparison succeeds.

### Real-World Cases

- **Cross-platform libraries** – testing OS errors portably.
- **`std::filesystem`** – error codes from filesystem operations are compared against `std::errc` conditions.
- **Custom error domains** – defining portable conditions that map to library-specific codes.

---

## 4. `std::expected` (C++23)

### Definitions

**Core Definition**
`std::expected<T, E>` is a class template that represents either an expected value of type `T` or an unexpected error of type `E`; it is never valueless.

**Technical Definition**
Defined in `<expected>` since C++23, `std::expected` provides a way to represent either of two values: an expected value of type `T`, or an unexpected value of type `E`. The contained value or error is nested within the `expected` object (no dynamic allocation). It supports monadic operations (`and_then`, `transform`, `or_else`, `transform_error`) that allow chaining without explicit error checks at every step.

**Beginner-Friendly Explanation**
`std::expected` is a box that contains either the answer or the reason it couldn’t produce the answer. You open the box, check if it has a value, and either use the value or handle the error. Crucially, you can chain operations on the box, and if an error occurs at any step, the remaining steps are skipped automatically.

### Purposes

- To return a value or an error from a function in a single, type-safe return type.
- To avoid the overhead and control-flow disruption of exceptions for common, expected failures (e.g., “file not found”).
- To provide monadic chaining for clean, composable error handling.
- To serve as a vocabulary type for functions that commonly fail.

### Syntax Rules and Structure

**General Syntax**
```cpp
#include <expected>

std::expected<int, std::string> parse_int(const std::string& s) {
    if (/* failure */) return std::unexpected("invalid input");
    return 42;
}

// Usage
auto result = parse_int("42");
if (result) {
    int value = *result;
} else {
    std::string error = result.error();
}
```

**Component Breakdown**

| Operation | Description |
|-----------|-------------|
| `has_value()` | `true` if the expected value is present. |
| `operator bool` | Same as `has_value()`. |
| `operator*` | Unchecked access to the value. |
| `value()` | Checked access; throws `std::bad_expected_access` if no value. |
| `error()` | Access to the unexpected value. |
| `value_or(default)` | Returns the value or a default. |
| `and_then(f)` | Chains a function returning another `expected`. |
| `transform(f)` | Transforms the value if present. |
| `or_else(f)` | Handles the error if present. |

**Syntax Rules**
- `T` must be `void` or `Destructible`; `E` must be `Destructible`.
- `expected` is never valueless; it always contains either `T` or `E`.
- Use `std::unexpected(error)` to construct the error state.
- Monadic operations short-circuit on error: if the `expected` contains an error, subsequent `transform`/`and_then` calls are skipped.

**Constraints and Limitations**
- Requires C++23 support in the compiler and standard library.
- `T` and `E` cannot be reference types, function types, or certain special types.
- `operator*` on an error state is undefined behaviour; use `value()` or check `has_value()` first.

### Annotated Code Examples

**Example 1: Basic value-or-error return**

```cpp
#include <expected>
#include <iostream>
#include <string>

std::expected<int, std::string> parse_int(const std::string& s) {
    try {
        std::size_t pos;
        int value = std::stoi(s, &pos);
        if (pos != s.size())
            return std::unexpected("trailing characters");
        return value;
    } catch (const std::exception& e) {
        return std::unexpected(e.what());
    }
}

int main() {
    auto r1 = parse_int("42");
    if (r1) std::cout << "Value: " << *r1 << '\n';

    auto r2 = parse_int("42abc");
    if (!r2) std::cout << "Error: " << r2.error() << '\n';
}
```

**Expected Output**
```
Value: 42
Error: trailing characters
```

**Why this output?**  
The first call returns the parsed integer; the second returns an unexpected error string explaining that trailing characters were found.

**Example 2: Monadic chaining**

```cpp
#include <expected>
#include <iostream>
#include <string>

std::expected<int, std::string> parse_int(const std::string& s) {
    try {
        std::size_t pos;
        int value = std::stoi(s, &pos);
        if (pos != s.size()) return std::unexpected("trailing characters");
        return value;
    } catch (const std::exception& e) {
        return std::unexpected(e.what());
    }
}

int main() {
    auto result = parse_int("21")
        .transform([](int v) { return v * 2; })       // 42
        .transform([](int v) { return std::to_string(v); }); // "42"

    if (result) std::cout << "Chained: " << *result << '\n';
    else std::cout << "Error: " << result.error() << '\n';
}
```

**Expected Output**
```
Chained: 42
```

**Why this output?**  
`transform` applies each function to the expected value; if at any point the `expected` held an error, the remaining `transform` calls would be skipped and the error would propagate.

### Real-World Cases

- **Parsing** – returning either a parsed value or a parse error.
- **Configuration loading** – `expected<Config, Error>` from a file.
- **Network requests** – `expected<Response, NetworkError>`.
- **Any function with common, expected failure modes** – use `expected` instead of exceptions.

---

## 5. Exceptions vs. Error Codes

### Definitions

**Core Definition**
The choice between exceptions and error codes is a trade-off between automatic propagation with runtime overhead on the error path (exceptions) and manual propagation with near-zero overhead (error codes).

**Technical Definition**
Exceptions use `throw` and `try`/`catch`; on the happy path they cost nothing (table-based unwinding), but throwing an exception costs approximately 1–10 µs due to stack unwinding and RTTI matching. Error codes return `int`/`enum` values and require manual checks at each call site; the error path is near-zero, but the happy path incurs a small branch-per-call cost, and errors must be propagated manually through every layer. `std::expected` combines the type safety of a return value with monadic composability, avoiding both the overhead of exceptions and the boilerplate of manual error-code propagation.

**Beginner-Friendly Explanation**
Exceptions are like fire alarms: they are silent when everything is fine (zero cost), but when they go off, everyone stops and evacuates (slow). Error codes are like sticky notes: you have to read and pass them along yourself, but they are very cheap. `expected` is a sealed envelope that contains either the result or the bad news; you can pass it along without opening it, and only the final recipient needs to check.

### Comparison Table

| Mechanism | API Style | Error Path Cost | Happy Path Cost | Composability |
|-----------|-----------|-----------------|-----------------|---------------|
| **Exceptions** | `throw` / `try`-`catch` | High (~1–10 µs per throw) | Zero (table-based unwinding) | Automatic propagation |
| **Error codes** | Return `int`/`enum`, check at call site | Near-zero | Small (branch per call) | Manual propagation |
| **`std::expected`** | Return value-or-error | Near-zero | Small (slightly larger return) | Monadic chaining (C++23) |

*Source: Dr-Sergey/learn_cpp error-handling comparison table*.

### Purposes

- To choose the right mechanism for each boundary and failure frequency.
- To avoid the performance cliff of exceptions on hot error paths.
- To avoid the maintenance burden of manual error-code propagation.
- To ensure errors are not silently ignored.

### Decision Flow

```
Is this an EXCEPTIONAL condition (rare, unexpected)?
├─ YES → Is the codebase exceptions-enabled?
│        ├─ YES → throw an exception
│        └─ NO  → error code or std::expected
└─ NO (common/expected failure, e.g., "file not found")
         ├─ C++23 available? → std::expected<T, E>
         └─ Pre-C++23?       → error code / std::optional / out-param

Additional constraints:
├─ Real-time / embedded / -fno-exceptions → error codes or expected
├─ noexcept API boundary                  → error codes or expected
└─ Cross-language boundary (C ABI)        → error codes only
```

*Source: adapted from the decision flow in Dr-Sergey/learn_cpp*.

### Annotated Code Example

```cpp
#include <iostream>
#include <system_error>
#include <stdexcept>

// Exception style: for truly exceptional failure
int divide_exception(int a, int b) {
    if (b == 0) throw std::invalid_argument("division by zero");
    return a / b;
}

// Error-code style: for C ABI or noexcept boundaries
std::error_code divide_error_code(int a, int b, int& result) noexcept {
    if (b == 0) return std::make_error_code(std::errc::invalid_argument);
    result = a / b;
    return {};
}

int main() {
    try {
        std::cout << divide_exception(10, 2) << '\n'; // 5
    } catch (const std::exception& e) {
        std::cout << "Exception: " << e.what() << '\n';
    }

    int r = 0;
    std::error_code ec = divide_error_code(10, 0, r);
    if (ec) std::cout << "Error code: " << ec.message() << '\n';
}
```

**Expected Output**
```
5
Error code: Invalid argument
```

**Why this output?**  
The exception version returns `5` on success and throws on failure; the error-code version returns `0` on success and an `error_code` on failure, which the caller checks explicitly.

### Real-World Cases

- **Exceptions** – constructors, deep call stacks, truly exceptional conditions.
- **Error codes** – C ABI libraries, real-time systems, `noexcept` API boundaries.
- **`std::expected`** – modern C++ APIs with common, expected failure modes (parsing, I/O).

---

## 6. Error Handling Best Practices

### Definitions

**Core Definition**
Error handling best practices are guidelines for choosing, implementing, and composing the three mechanisms so that errors are reported consistently, resources are not leaked, and performance targets are met.

**Technical Definition**
These practices are informed by the C++ Core Guidelines, the Microsoft C++ guidelines, and community consensus: prefer exceptions for truly exceptional conditions; use error codes or `std::expected` for common, expected failures and `noexcept` boundaries; never return a valid-looking value on failure; be consistent within a subsystem; and document the chosen mechanism at API boundaries.

**Beginner-Friendly Explanation**
Best practices are simple: use exceptions for rare disasters, error codes for C boundaries and real-time code, and `expected` for everyday “it might fail” situations. Whatever you pick, stick with it in that part of the program, and never pretend a failure didn’t happen.

### Key Guidelines

| Guideline | Summary |
|-----------|---------|
| **Use exceptions for exceptional code** | Exceptions are for logic errors and runtime errors that are truly rare or unrecoverable locally. |
| **Use error codes or `expected` for expected failures** | “File not found” or “parse error” are routine; do not throw for them. |
| **Never return a valid-looking value on failure** | A failed read must not return `0` or a default that looks like real data. |
| **Mark non-throwing functions `noexcept`** | Enables better codegen and move semantics in containers. |
| **Respect `noexcept` boundaries** | Never let an exception escape a `noexcept` function; catch internally or use error codes. |
| **Keep error categories extensible** | Add new error codes at the end; never remove or renumber existing ones. |
| **Be consistent within a subsystem** | Mixing mechanisms freely leads to unhandled errors. |
| **Use `std::expected` as a vocabulary type** | Return it from functions that can commonly fail. |
| **Design for exception safety** | Even when using error codes, RAII ensures no leaks on early returns. |

### Annotated Code Example

```cpp
#include <iostream>
#include <expected>
#include <string>
#include <system_error>

// Good: expected for a common, expected failure
std::expected<int, std::string> parse_positive(const std::string& s) {
    try {
        int v = std::stoi(s);
        if (v <= 0) return std::unexpected("not positive");
        return v;
    } catch (const std::exception&) {
        return std::unexpected("invalid number");
    }
}

// Good: noexcept boundary with error code
std::error_code safe_parse(const std::string& s, int& out) noexcept {
    auto r = parse_positive(s);
    if (!r) return std::make_error_code(std::errc::invalid_argument);
    out = *r;
    return {};
}

int main() {
    auto r1 = parse_positive("42");
    if (r1) std::cout << "Parsed: " << *r1 << '\n';

    auto r2 = parse_positive("-5");
    if (!r2) std::cout << "Error: " << r2.error() << '\n';

    int value = 0;
    std::error_code ec = safe_parse("100", value);
    if (!ec) std::cout << "Safe parsed: " << value << '\n';
}
```

**Expected Output**
```
Parsed: 42
Error: not positive
Safe parsed: 100
```

**Why this output?**  
`parse_positive` returns `expected` for its common failure modes; `safe_parse` wraps it in a `noexcept` function that translates any failure into an `error_code`. This demonstrates the recommended layered approach: `expected` for the rich interface, error codes at the `noexcept` boundary.

### Real-World Cases

- **Library design** – exposing `expected` in modern C++ APIs; providing `error_code` overloads for `noexcept` callers.
- **Code reviews** – checking that every function has a documented error strategy and that errors cannot be silently ignored.
- **Cross-platform code** – using `error_condition` for portable error tests and `expected` for the public interface.

---

## References

- `std::error_code` — cppreference – https://en.cppreference.com/w/cpp/error/error_code
- `std::error_condition` — cppreference – https://en.cppreference.com/w/cpp/error/error_condition
- `std::expected` — cppreference – https://en.cppreference.com/w/cpp/utility/expected
- Know when to use exceptions vs error codes vs `std::expected` — Dr-Sergey/learn_cpp – https://github.com/Dr-Sergey/learn_cpp/blob/main/11_Error_Handling/Know_when_to_use_exceptions_vs_error_codes_vs_stdexpected.md
- Modern C++ best practices for exceptions and error handling — Microsoft Learn – https://learn.microsoft.com/en-us/cpp/cpp/errors-and-exception-handling-modern-cpp
- C++ Core Guidelines — Error handling (E rules) – https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#S-errors
- `std::error_code` — libstdc++ manual – https://gcc.gnu.org/onlinedocs/libstdc++/manual/system_error.html
- `std::error_condition` — libstdc++ class reference – https://gcc.gnu.org/onlinedocs/libstdc++/latest-doxygen/a01858.html

---

This cheat sheet provides a comprehensive, structured reference for C++ error handling patterns, covering error codes, `std::error_code`, `std::error_condition`, `std::expected` (C++23), the trade-offs between exceptions and error codes, and best practices, with annotated code examples, expected outputs, real-world cases, and authoritative references.