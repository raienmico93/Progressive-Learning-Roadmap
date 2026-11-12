# C++ `noexcept` — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
`noexcept` is a C++ keyword with two related roles: as a **specifier**, it declares whether a function is allowed to propagate exceptions to its caller; as an **operator**, it performs a compile-time check that determines whether a given expression is declared not to throw.

**Technical Definition**
The `noexcept` specifier (since C++11) is a function exception specification that declares the function as *non-throwing* (if the specification is satisfied) or *potentially throwing*. Since C++17, the `noexcept` specification is part of the function's type and participates in the type system. The `noexcept` operator (since C++11) is a compile-time expression that yields `true` if the operand expression is specified to be non-throwing, and `false` otherwise; it does not evaluate its operand.

**Beginner-Friendly Explanation**
`noexcept` is a promise about exceptions. When you mark a function `noexcept`, you are telling the compiler and everyone who calls it: “This function will not let exceptions escape.” In return, the compiler can optimise more aggressively, and certain standard-library operations (like moving elements during `vector` reallocation) will use faster code paths. The `noexcept` operator is a way to ask the compiler, “Is this expression guaranteed not to throw?”—useful for writing generic code that adapts to the types it is given.

### Key Characteristics

- **Two distinct roles** – the specifier declares exception behaviour; the operator queries it.
- **Part of the function type since C++17** – two functions differing only in `noexcept` have different types and cannot overload each other.
- **Violation terminates** – if an exception escapes a `noexcept` function, `std::terminate` is called (whether stack unwinding occurs is implementation-defined).
- **Enables performance optimisations** – `noexcept` move constructors let `std::vector` move rather than copy during reallocation, which can be dramatically faster.
- **Conditional form** – `noexcept(expression)` allows the specification to depend on compile-time properties of template parameters.
- **Implicit `noexcept`** – destructors, and implicitly-declared or defaulted copy/move constructors, are often `noexcept` by default unless their members are potentially throwing.

### Prerequisites

- Basic C++ syntax (functions, templates, lambdas).
- Understanding of exceptions, `throw`, and `std::terminate`.
- Familiarity with move semantics and the standard containers.
- Knowledge of compile-time expressions (`constexpr`, `decltype`, `std::declval`).
- Awareness of the function type system (especially since C++17).

### Related Programming Areas

- **Exception safety** – `noexcept` is the mechanism that expresses the nothrow guarantee.
- **Move semantics** – `noexcept` move constructors and move assignment operators are critical for container performance.
- **Generic programming** – `noexcept(...)` enables conditionally `noexcept` templates.
- **Standard library** – containers, `std::vector`, `std::optional`, and `std::variant` all depend on `noexcept` properties.
- **API design** – documenting and enforcing nothrow contracts.

### Core Concepts / Features

1. `noexcept` (the keyword and its dual role)
2. `noexcept` specifier
3. `noexcept` operator
4. Conditional `noexcept`
5. `noexcept` best practices
6. `noexcept` pitfalls

---

## 1. `noexcept` (The Keyword)

### Definitions

**Core Definition**
`noexcept` is a C++ keyword introduced in C++11 that appears both as a function exception specification and as a compile-time operator.

**Technical Definition**
The keyword serves two grammatical purposes. As a *specifier*, it forms part of a function declaration (`noexcept` or `noexcept(expression)`) and declares the function non-throwing when the expression evaluates to `true` (the bare form is equivalent to `noexcept(true)`). As an *operator*, it forms an expression (`noexcept(expression)`) that yields a `bool` prvalue without evaluating the operand. Before C++17, the specifier was not part of the function type; since C++17, it is, and a `noexcept` function is implicitly convertible to a pointer to a potentially-throwing function but not vice versa.

**Beginner-Friendly Explanation**
`noexcept` is a single keyword that does two jobs: it makes promises (as a specifier) and it asks questions (as an operator). The promise says “I won’t throw.” The question asks “Will this throw?”

### Purposes

- To declare that a function will not propagate exceptions.
- To query at compile time whether an expression is non-throwing.
- To enable compiler optimisations and standard-library fast paths.
- To enforce nothrow contracts in interfaces.

### Syntax Rules and Structure

**General Syntax**
```cpp
// Specifier form
void f() noexcept;                    // equivalent to noexcept(true)
void g() noexcept(expression);        // conditional

// Operator form
constexpr bool b = noexcept(expression);
```

**Component Breakdown**

| Form | Meaning |
|------|---------|
| `noexcept` | Same as `noexcept(true)`. |
| `noexcept(expr)` (specifier) | Non-throwing if `expr` is `true`. |
| `noexcept(expr)` (operator) | `true` if `expr` is non-throwing. |

**Syntax Rules**
- The specifier may appear on function declarations, definitions, lambdas, and pointers to functions (since C++17, it is part of the type).
- The operator does not evaluate its operand; it inspects the expression's exception specification.
- A `noexcept` function that throws calls `std::terminate`.

### Annotated Code Example

```cpp
#include <iostream>

void may_throw() {}
void no_throw() noexcept {}

int main() {
    std::cout << std::boolalpha;
    std::cout << noexcept(may_throw()) << '\n';  // false
    std::cout << noexcept(no_throw()) << '\n';   // true
}
```

**Expected Output**
```
false
true
```

**Why this output?**  
The `noexcept` operator inspects the declaration of each function: `no_throw` is declared `noexcept`, so the operator yields `true`; `may_throw` is potentially throwing, so it yields `false`.

### Real-World Cases

- **Move constructors** – marking them `noexcept` enables `vector` to move instead of copy.
- **`swap` functions** – `noexcept` swap is required for copy-and-swap strong exception safety.
- **Generic wrappers** – conditionally `noexcept` forwarding functions.

---

## 2. `noexcept` Specifier

### Definitions

**Core Definition**
The `noexcept` specifier is the part of a function declaration that states whether the function is non-throwing or potentially throwing.

**Technical Definition**
The specifier takes two forms: `noexcept` (equivalent to `noexcept(true)`) and `noexcept(expression)`. When the expression is `true` (or the bare form is used), the function is *non-throwing*; when it is `false`, the function is *potentially throwing*. Since C++17, the specifier is part of the function type, and it may appear in any function declarator. The deprecated `throw()` form was removed in C++20 and is equivalent to `noexcept(true)`.

**Beginner-Friendly Explanation**
The specifier is the part that makes the promise. You put `noexcept` after a function’s parameter list to tell the compiler and callers that the function will not throw.

### Purposes

- To declare a nothrow guarantee.
- To allow the compiler to omit exception-handling machinery.
- To enable standard-library optimisations that depend on `noexcept`.
- To document and enforce API contracts.

### Syntax Rules and Structure

**General Syntax**
```cpp
return_type function_name(params) noexcept;              // unconditional
return_type function_name(params) noexcept(expression);  // conditional
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `noexcept` | Unconditional non-throwing. |
| `noexcept(expr)` | Non-throwing if `expr` is `true`. |
| Position | After the parameter list; part of the function type since C++17. |

**Syntax Rules**
- The specifier may appear on function declarations, definitions, and lambdas.
- It cannot appear in a `typedef` or type alias before C++17; since C++17, it is part of the type and may appear in function declarators.
- If a `noexcept` function throws, `std::terminate` is called.

**Constraints and Limitations**
- `noexcept` is a promise, not a compile-time check of the body; the compiler does not verify that the body cannot throw.
- Violating the promise terminates the program (unwinding behaviour is implementation-defined).

### Annotated Code Examples

**Example 1: Unconditional `noexcept`**

```cpp
#include <iostream>

void safe() noexcept {
    std::cout << "safe\n";
}

int main() {
    safe();
}
```

**Expected Output**
```
safe
```

**Why this output?**  
`safe` is declared `noexcept` and does not throw.

**Example 2: Violation leads to termination**

```cpp
#include <iostream>
#include <stdexcept>

void bad() noexcept {
    throw std::runtime_error("boom");      // violates noexcept → std::terminate
}

int main() {
    try {
        bad();
    } catch (...) {
        std::cout << "unreachable\n";
    }
}
```

**Expected Output**
```
terminate called after throwing an instance of 'std::runtime_error'
  what():  boom
```

**Why this output?**  
The exception escapes a `noexcept` function, so `std::terminate` is called; the `catch` in `main` is never reached.

### Real-World Cases

- **Destructors** – implicitly `noexcept` to ensure safe unwinding.
- **Move constructors** – marked `noexcept` to enable efficient container growth.
- **`swap`** – required to be `noexcept` for strong exception safety.

---

## 3. `noexcept` Operator

### Definitions

**Core Definition**
The `noexcept` operator is a unary compile-time operator that returns `true` if its operand expression is specified to be non-throwing, and `false` otherwise.

**Technical Definition**
`noexcept(expression)` yields a `bool` prvalue. It does not evaluate the expression; it inspects the exception specification of the expression. Before C++17, it returned `true` if the set of potential exceptions of the expression was empty; since C++17, it returns `true` if the expression is specified to be non-throwing. It is commonly used inside a function template's `noexcept` specifier to propagate the exception specification of the types it operates on.

**Beginner-Friendly Explanation**
The operator is the question: “If I evaluate this expression, is it guaranteed not to throw?” It answers at compile time, so you can use it to write conditionally `noexcept` code.

### Purposes

- To query whether an expression is non-throwing.
- To propagate exception specifications in templates.
- To conditionally mark functions `noexcept` based on their operations.
- To select optimised code paths at compile time.

### Syntax Rules and Structure

**General Syntax**
```cpp
constexpr bool result = noexcept(expression);
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `expression` | The expression to inspect (not evaluated). |
| Result | `bool` prvalue. |

**Syntax Rules**
- The operand is not evaluated (it is an unevaluated operand).
- The result is `true` if the expression is declared non-throwing, `false` otherwise.
- Available since C++11.

**Constraints and Limitations**
- It inspects only the *declaration*; it does not prove that the body cannot throw.
- The result can depend on the exception specifications of called functions.

### Annotated Code Example

```cpp
#include <iostream>
#include <utility>
#include <vector>

void may_throw();
void no_throw() noexcept;

int main() {
    std::cout << std::boolalpha;
    std::cout << noexcept(may_throw()) << '\n';       // false
    std::cout << noexcept(no_throw()) << '\n';        // true
    std::cout << noexcept(std::declval<int>() + 1) << '\n'; // true
}
```

**Expected Output**
```
false
true
true
```

**Why this output?**  
`no_throw` is declared `noexcept`; `may_throw` is not; built-in arithmetic is non-throwing.

### Real-World Cases

- **Generic wrappers** – `template<class F> void call(F f) noexcept(noexcept(f()))`.
- **Container adaptors** – propagating `noexcept` from contained operations.
- **Compile-time dispatch** – selecting an implementation based on exception guarantees.

---

## 4. Conditional `noexcept`

### Definitions

**Core Definition**
Conditional `noexcept` is the use of `noexcept(expression)` as a function specifier, where the expression determines whether the function is non-throwing based on compile-time properties (often of template parameters).

**Technical Definition**
A conditional `noexcept` specification uses the `noexcept` operator (or another `constexpr bool` expression) to compute the specifier. This allows a function template to be `noexcept` for some instantiations and potentially throwing for others. For example, a wrapper function can propagate the exception specification of the operation it forwards to. The Google C++ Style Guide recommends using conditional `noexcept` specifiers with simple conditions when exceptions are enabled, rather than unconditional `noexcept`.

**Beginner-Friendly Explanation**
Conditional `noexcept` means the promise depends on the situation. A template function can say: “I am `noexcept` if the function I call is `noexcept`; otherwise I might throw.” It lets generic code be as efficient as possible without lying.

### Purposes

- To propagate exception specifications in generic code.
- To make wrappers `noexcept` exactly when their underlying operations are.
- To enable optimisations for types with `noexcept` move operations.
- To avoid unnecessary termination in templates.

### Syntax Rules and Structure

**General Syntax**
```cpp
template<class F, class... Args>
auto invoke(F&& f, Args&&... args)
    noexcept(noexcept(std::forward<F>(f)(std::forward<Args>(args)...)))
    -> decltype(std::forward<F>(f)(std::forward<Args>(args)...));
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `noexcept(...)` specifier | The function's exception specification. |
| Inner `noexcept(...)` operator | Queries whether the forwarded call is non-throwing. |

**Syntax Rules**
- The inner `noexcept` operator inspects the call expression.
- The outer `noexcept` specifier uses that result.
- Requires the call expression to be valid (often via `decltype` or `std::declval`).

**Constraints and Limitations**
- The condition should be simple and correct; an incorrect condition reintroduces the pitfalls of unconditional `noexcept`.
- The inner expression must be well-formed.

### Annotated Code Example

```cpp
#include <iostream>
#include <utility>

void may_throw() {}
void no_throw() noexcept {}

template<class F>
void call(F f) noexcept(noexcept(f())) {
    f();
}

int main() {
    std::cout << std::boolalpha;
    std::cout << noexcept(call(may_throw)) << '\n'; // false
    std::cout << noexcept(call(no_throw)) << '\n';  // true
}
```

**Expected Output**
```
false
true
```

**Why this output?**  
`call` propagates the exception specification of `f`: for `may_throw`, `call` is potentially throwing; for `no_throw`, `call` is `noexcept`.

### Real-World Cases

- **`std::vector` and move constructors** – `vector` checks `noexcept(T(T&&))` to decide between move and copy during reallocation.
- **Perfect-forwarding wrappers** – `std::invoke`, `std::apply`.
- **Custom containers** – conditionally `noexcept` move operations.

---

## 5. `noexcept` Best Practices

### Definitions

**Core Definition**
`noexcept` best practices are guidelines for when and how to apply the `noexcept` specifier so that it provides benefits without introducing termination risks.

**Technical Definition**
The C++ Core Guidelines (F.6) state: “If your function must not throw, declare it `noexcept`.” This means `noexcept` should be used when the function is *designed* to never throw, not merely when it happens not to throw today. Move constructors, move assignment operators, `swap` functions, and destructors are the canonical candidates. The Google C++ Style Guide recommends conditional `noexcept` for templates and warns that `noexcept` is difficult to remove once callers rely on it.

**Beginner-Friendly Explanation**
Best practices are simple: mark `noexcept` when you genuinely promise never to throw—especially for moves and swaps. For templates, use conditional `noexcept` so the promise is only as strong as the operations it depends on. Do not sprinkle `noexcept` everywhere just because you can.

### Key Guidelines

| Guideline | Summary |
|-----------|---------|
| **Core Guidelines F.6** | Declare a function `noexcept` if it must not throw. |
| **Mark move operations `noexcept`** | Enables `vector` and other containers to use moves. |
| **Mark `swap` `noexcept`** | Required for copy-and-swap strong exception safety. |
| **Destructors are implicitly `noexcept`** | Do not throw from them. |
| **Use conditional `noexcept` in templates** | Propagate the exception specification of dependencies. |
| **Use unconditional `noexcept` only with confidence** | Removing `noexcept` later is hard and risky. |
| **Do not mark allocating functions `noexcept`** | Allocation can throw `std::bad_alloc`. |
| **Prefer `noexcept` on simple accessors** | Getters and trivial operations. |

### Annotated Code Example

```cpp
#include <iostream>
#include <utility>
#include <vector>

class Buffer {
    int* data_;
public:
    Buffer() : data_(new int(0)) {}
    ~Buffer() { delete data_; }              // implicitly noexcept

    Buffer(Buffer&& other) noexcept          // best practice: noexcept move
        : data_(std::exchange(other.data_, nullptr)) {}

    Buffer& operator=(Buffer&& other) noexcept {
        if (this != &other) {
            delete data_;
            data_ = std::exchange(other.data_, nullptr);
        }
        return *this;
    }

    friend void swap(Buffer& a, Buffer& b) noexcept {
        std::swap(a.data_, b.data_);
    }
};

int main() {
    std::vector<Buffer> v;
    v.reserve(10);
    v.emplace_back();
    std::cout << v.size() << '\n';
}
```

**Expected Output**
```
1
```

**Why this output?**  
The `noexcept` move constructor and move assignment allow `vector` to use moves during reallocation, satisfying the strong exception guarantee efficiently. The destructor is implicitly `noexcept`, and `swap` is `noexcept` for copy-and-swap.

### Real-World Cases

- **Standard containers** – rely on `noexcept` move constructors for performance.
- **`std::vector` reallocation** – uses moves if `T`’s move constructor is `noexcept`; otherwise copies.
- **`std::optional`, `std::variant`** – propagate `noexcept` from contained types.

---

## 6. `noexcept` Pitfalls

### Definitions

**Core Definition**
`noexcept` pitfalls are the ways in which incorrect or overzealous use of `noexcept` can cause unexpected program termination, performance regressions, or inflexible APIs.

**Technical Definition**
The primary pitfall is **accidental termination**: if an exception escapes a `noexcept` function, `std::terminate` is called, and the stack may or may not be unwound (the behaviour is implementation-defined). A second pitfall is **difficulty of removal**: once a function is `noexcept`, callers may depend on it, and removing the specifier breaks their assumptions. A third is **conditional `noexcept` that is too permissive**, which can make a template `noexcept` for some instantiations and terminate unexpectedly for others. The more `noexcept` specifiers a codebase has, the greater the chance of termination.

**Beginner-Friendly Explanation**
The main danger is that if you promise “I won’t throw” and then throw anyway, your program crashes immediately with no recovery. A second danger is that adding `noexcept` today can be easy, but removing it later can break code that depends on the promise. Be conservative: only promise what you can guarantee.

### Key Pitfalls

| Pitfall | Description |
|---------|-------------|
| **Accidental termination** | An exception escaping a `noexcept` function calls `std::terminate`. |
| **Unwinding uncertainty** | Whether the stack is unwound before termination is implementation-defined. |
| **Irreversible promise** | Removing `noexcept` can break callers that rely on it. |
| **Overuse increases termination risk** | More `noexcept` specifiers mean more chances of accidental termination. |
| **Incorrect conditional conditions** | A wrong condition makes a template `noexcept` when it should not be. |
| **Marking allocating functions `noexcept`** | `new` can throw `std::bad_alloc`; marking such functions `noexcept` leads to termination. |
| **Assuming the compiler checks the body** | `noexcept` is a promise; the compiler does not verify that the body cannot throw. |

### Annotated Code Example

```cpp
#include <iostream>
#include <stdexcept>
#include <vector>

void risky() noexcept {                    // PITFALL: may throw
    std::vector<int> v;
    v.at(100);                             // throws std::out_of_range
}

int main() {
    try {
        risky();
    } catch (const std::exception& e) {
        std::cout << "caught: " << e.what() << '\n'; // never reached
    }
}
```

**Expected Output**
```
terminate called after throwing an instance of 'std::out_of_range'
  what():  vector::_M_range_check: __n (which is 100) >= this->size() (which is 0)
```

**Why this output?**  
`risky` is marked `noexcept` but calls `v.at(100)`, which throws. The exception escapes the `noexcept` function, so `std::terminate` is called; the `catch` in `main` is never reached.

### Real-World Cases

- **Third-party libraries** – exceptions from library code escaping a `noexcept` wrapper.
- **Templates** – conditional `noexcept` with an incorrect condition.
- **Code reviews** – auditing for `noexcept` on functions that call throwing operations.

---

## References

- `noexcept` specifier — cppreference – https://en.cppreference.com/w/cpp/language/noexcept_spec
- `noexcept` operator — cppreference – https://en.cppreference.com/w/cpp/language/noexcept
- C++ Core Guidelines — F.6: If your function must not throw, declare it `noexcept` – https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rf-noexcept
- Google C++ Style Guide — Exception specifications – https://google.github.io/styleguide/cppguide.html
- `std::terminate` (noexcept violation) — cppreference – https://en.cppreference.com/w/cpp/error/terminate
- Ruminations on (node-based) containers and `noexcept` — Ville Voutilainen, Open Standards – http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2014/n4049.pdf
- Friendly reminder to mark your move constructors `noexcept` — Andy G, ISO C++ – https://isocpp.org/blog/2020/09/friendly-reminder-to-mark-your-move-constructors-noexcept

---

This cheat sheet provides a comprehensive, structured reference for C++ `noexcept`, covering the keyword, the specifier, the operator, conditional `noexcept`, best practices, and pitfalls, with annotated code examples, expected outputs, real-world cases, and authoritative references.