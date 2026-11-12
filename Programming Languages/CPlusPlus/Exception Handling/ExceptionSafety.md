# C++ Exception Safety — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
Exception safety is a set of guarantees about the state of a program after an exception is thrown, ensuring that resources are not leaked and that objects remain in a valid, consistent state.

**Technical Definition**
Exception safety is a property of an operation (a function, a class member function, or a whole component) that describes what happens to program state when that operation terminates by throwing an exception. The C++ community recognises three principal levels of guarantee—the **nothrow (nofail)** guarantee, the **strong** guarantee (commit-or-rollback semantics), and the **basic** guarantee—plus a fourth level, **no guarantee**, which is considered defective and unacceptable in production code. The guarantees are cumulative: a function providing the strong guarantee also provides the basic guarantee, and a nothrow function trivially provides all weaker guarantees.

**Beginner-Friendly Explanation**
Exception safety is about answering the question: “If something goes wrong and an exception is thrown, what state is my program left in?” A program with good exception safety will not leak memory, will not leave objects half-broken, and will either complete an operation fully or leave everything exactly as it was before the operation started. Think of it as a safety rating for your code—higher ratings mean fewer surprises when errors occur.

### Key Characteristics

- **Guarantee levels** – nothrow, strong, basic, and no guarantee, ordered from strongest to weakest.
- **Cumulative hierarchy** – strong implies basic; nothrow implies both.
- **RAII as the foundation** – resource management via constructors/destructors is the primary technique for achieving exception safety.
- **Copy-and-swap idiom** – the standard technique for implementing the strong guarantee in assignment operators.
- **`noexcept` enforcement** – the nothrow guarantee is expressed in code via `noexcept` (Core Guidelines F.6).
- **Critical functions must not fail** – destructors, `swap`, move constructors, and deallocation functions are expected to be nothrow.

### Prerequisites

- Basic C++ syntax (classes, constructors, destructors).
- Understanding of `try`/`catch`/`throw` and stack unwinding.
- Familiarity with RAII and smart pointers.
- Knowledge of move semantics and the `noexcept` specifier.
- Awareness of the copy-and-swap idiom.

### Related Programming Areas

- **Exception fundamentals** – `throw`, `try`, `catch`, and the exception hierarchy.
- **RAII** – automatic resource cleanup during stack unwinding.
- **Smart pointers** – `unique_ptr` and `shared_ptr` provide nothrow cleanup.
- **Move semantics** – move constructors and move assignment are expected to be `noexcept`.
- **Standard library containers** – `std::vector::push_back` provides the strong guarantee (with conditions).

### Core Concepts / Features

1. Exception safety (the overarching concept)
2. Basic guarantee
3. Strong guarantee
4. Nothrow guarantee
5. Exception safety levels (including no guarantee)
6. RAII and exception safety
7. Exception safety best practices

---

## 1. Exception Safety (The Concept)

### Definitions

**Core Definition**
Exception safety is the property of code that, when an exception is thrown, it avoids resource leaks, preserves class invariants, and leaves objects in a valid state.

**Technical Definition**
Code is exception safe if it avoids undesirable effects such as failure to release an acquired resource (a leak) or failure to reestablish a class invariant after a partial update. The SEI CERT C++ Coding Standard defines exception safety through three guarantee levels and explicitly states that code providing no exception safety guarantee is unsafe and must be considered defective. cppreference notes that the guarantees describe what happens to the state of the program after an exception is thrown.

**Beginner-Friendly Explanation**
Exception safety is a quality of code: it means your program handles errors gracefully instead of crashing, leaking memory, or corrupting data. It is not about *catching* exceptions—it is about what state your program is in *after* an exception passes through.

### Purposes

- To prevent resource leaks when exceptions are thrown.
- To preserve class invariants and object validity.
- To provide predictable, well-defined behaviour after an exception.
- To enable robust error recovery and program continuation.
- To guide API design by documenting what callers can rely on.

### Syntax Rules and Structure

**General Framework**
Exception safety is not expressed through syntax directly; it is a design property. However, it is enforced and documented through:
- `noexcept` specifications (for the nothrow guarantee).
- Documentation comments stating the guarantee level.
- RAII wrappers for all resources.
- `swap` functions marked `noexcept`.

**Component Breakdown**

| Mechanism | Role in Exception Safety |
|-----------|---------------------------|
| RAII | Ensures resource cleanup during unwinding. |
| `noexcept` | Declares the nothrow guarantee. |
| Copy-and-swap | Implements the strong guarantee for assignment. |
| Destructors | Must not throw (implicitly `noexcept`). |
| `swap` | Must not throw for strong-guarantee idioms. |

**Constraints and Limitations**
- Exception safety cannot be retrofitted easily; it must be designed into the code.
- Raw pointers and manual resource management make exception safety difficult.
- Some operations (e.g., `vector::push_back` without a `noexcept` move constructor) may provide only the basic guarantee.

### Annotated Code Example

```cpp
#include <iostream>
#include <memory>
#include <stdexcept>

void not_exception_safe() {
    int* p = new int(42);                  // raw resource
    throw std::runtime_error("fail");      // p is leaked
    delete p;                              // never reached
}

void exception_safe() {
    auto p = std::make_unique<int>(42);    // RAII resource
    throw std::runtime_error("fail");      // p is destroyed automatically
}                                          // no leak

int main() {
    try { not_exception_safe(); }
    catch (const std::exception&) { std::cout << "Leak occurred\n"; }

    try { exception_safe(); }
    catch (const std::exception&) { std::cout << "No leak\n"; }
}
```

**Expected Output**
```
Leak occurred
No leak
```

**Why this output?**  
In `not_exception_safe`, the raw `new` allocation is leaked when the exception is thrown. In `exception_safe`, `unique_ptr` releases the memory during stack unwinding, preventing a leak.

### Real-World Cases

- **Library design** – documenting exception guarantees for public APIs.
- **Code reviews** – auditing for raw resource leaks in exception paths.
- **Container implementations** – ensuring operations like `push_back` are exception safe.

---

## 2. Basic Guarantee

### Definitions

**Core Definition**
The basic guarantee states that if an operation throws, the program remains in a valid state: no resources are leaked and all invariants are intact, although the state may have changed.

**Technical Definition**
The basic exception safety guarantee is a property of an operation such that, if the operation terminates by raising an exception, it preserves program state invariants and prevents resource leaks. No object is left in an invalid state; all objects are destructible, and the program can continue executing, although the specific values may differ from before the operation.

**Beginner-Friendly Explanation**
The basic guarantee means “no mess.” Even if the operation fails, your program is still in a workable condition—nothing is leaked, nothing is corrupted, and you can keep running. However, the data might not be exactly what it was before the failure.

### Purposes

- To provide a minimum acceptable level of safety for all operations.
- To ensure that no resources are leaked even on failure.
- To preserve invariants so that objects remain usable.
- To allow the program to continue after an exception.

### Syntax Rules and Structure

**General Approach**
The basic guarantee is typically achieved by:
- Using RAII for all resources.
- Ensuring that every operation leaves objects in a valid state (even if the operation fails partway).
- Avoiding raw resource ownership.

**Component Breakdown**

| Technique | Effect |
|-----------|--------|
| RAII | Guarantees cleanup on failure. |
| Invariant-preserving updates | Even partial updates leave the object valid. |
| No raw `new`/`delete` | Eliminates leak-prone code. |

**Constraints and Limitations**
- The basic guarantee does not guarantee that the state is unchanged; it may be a partially completed state.
- It is the minimum acceptable level; providing no guarantee is defective.

### Annotated Code Example

```cpp
#include <iostream>
#include <vector>
#include <stdexcept>

void basic_guarantee() {
    std::vector<int> v = {1, 2, 3};
    try {
        v.push_back(4);                    // may reallocate
        throw std::runtime_error("extra fail");
    } catch (...) {
        // v is still valid (basic guarantee)
        std::cout << "Size: " << v.size() << '\n'; // valid, size is 4
    }
}

int main() {
    basic_guarantee();
}
```

**Expected Output**
```
Size: 4
```

**Why this output?**  
Even though the exception is thrown after `push_back`, the vector is still in a valid state (it contains four elements). No resources are leaked, and invariants hold.

### Real-World Cases

- **Most standard library operations** – many STL operations provide the basic guarantee.
- **Container insertions** – even if insertion fails, the container remains valid.
- **File I/O** – a partially written file remains valid.

---

## 3. Strong Guarantee

### Definitions

**Core Definition**
The strong guarantee (commit-or-rollback semantics) states that if an operation throws, the program state is rolled back to exactly what it was before the operation—as if the operation never started.

**Technical Definition**
The strong exception safety guarantee is a property of an operation such that, in addition to satisfying the basic guarantee, if the operation terminates by raising an exception, it has no observable effects on program state. The state of the program is rolled back to the state just before the function call (for example, `std::vector::push_back` provides this guarantee).

**Beginner-Friendly Explanation**
The strong guarantee is “all or nothing.” If the operation succeeds, you get the new state. If it fails, you get the old state back—exactly as it was, with no changes. It is like a database transaction that rolls back on error.

### Purposes

- To provide transactional semantics for critical operations.
- To ensure that failed operations leave no trace.
- To simplify error recovery for callers (they can retry or fall back).
- To implement reliable, predictable APIs.

### Syntax Rules and Structure

**Key Techniques**
- **Copy-and-swap idiom** – perform all throwing work on a copy, then swap with a `noexcept` swap.
- **Resource acquisition before mutation** – acquire all needed resources before modifying the target object.
- **Noexcept swap** – the swap step must not throw.

**Component Breakdown**

| Technique | Role |
|-----------|------|
| Copy-and-swap | Isolates throwing operations from the target object. |
| `noexcept` swap | Completes the commit phase without risk. |
| Validate-then-modify | Check all preconditions before any mutation. |

**Constraints and Limitations**
- Copy-and-swap requires a `noexcept` swap and a copy or move constructor.
- Not all operations can reasonably provide the strong guarantee (e.g., streaming I/O).
- Performance overhead from copying may be significant.

### Annotated Code Examples

**Example 1: Copy-and-swap for assignment**

```cpp
#include <iostream>
#include <utility>

class Widget {
    int* data_;
public:
    Widget(int v) : data_(new int(v)) {}
    ~Widget() { delete data_; }

    // Copy constructor (may throw)
    Widget(const Widget& other) : data_(new int(*other.data_)) {}

    // Move constructor (noexcept)
    Widget(Widget&& other) noexcept : data_(std::exchange(other.data_, nullptr)) {}

    // Copy-and-swap assignment: strong guarantee
    Widget& operator=(Widget other) noexcept {  // copy made here (may throw)
        swap(*this, other);                     // noexcept
        return *this;
    }

    friend void swap(Widget& a, Widget& b) noexcept {
        using std::swap;
        swap(a.data_, b.data_);
    }

    int value() const { return *data_; }
};

int main() {
    Widget a(10), b(20);
    a = b;                                     // strong guarantee
    std::cout << a.value() << '\n';            // 20
}
```

**Expected Output**
```
20
```

**Why this output?**  
The copy of `b` is made in the parameter `other` before any modification to `a`. If the copy constructor throws, `a` is untouched. If the copy succeeds, `swap` (which is `noexcept`) commits the change. This is the strong guarantee.

**Example 2: Strong guarantee via validate-then-modify**

```cpp
#include <iostream>
#include <vector>
#include <stdexcept>

class Account {
    std::vector<int> transactions_;
public:
    void add_transaction(int amount) {
        if (amount <= 0)
            throw std::invalid_argument("amount must be positive");
        // No mutation before validation — strong guarantee
        transactions_.push_back(amount);
    }
    std::size_t size() const { return transactions_.size(); }
};

int main() {
    Account acct;
    acct.add_transaction(100);
    try {
        acct.add_transaction(-5);              // throws before mutation
    } catch (const std::exception&) {}
    std::cout << acct.size() << '\n';          // 1
}
```

**Expected Output**
```
1
```

**Why this output?**  
The validation happens before any modification; when the exception is thrown, the account is unchanged (strong guarantee).

### Real-World Cases

- **`std::vector::push_back`** – provides the strong guarantee (if the move constructor is `noexcept` or the type is copyable).
- **Assignment operators** – copy-and-swap gives strong guarantee.
- **Configuration updates** – atomic replacement of settings.

---

## 4. Nothrow Guarantee

### Definitions

**Core Definition**
The nothrow (or nofail) guarantee states that an operation will never throw an exception.

**Technical Definition**
The nothrow exception guarantee means the function never throws exceptions. Nothrow (errors are reported by other means or concealed) is expected of destructors and other functions that may be called during stack unwinding. Nofail (the function always succeeds) is expected of swaps, move constructors, and other functions used by those that provide the strong guarantee.

**Beginner-Friendly Explanation**
The nothrow guarantee is the strongest promise: “This operation will always succeed, and it will never throw.” It is essential for destructors, `swap`, and move operations, because if they threw during cleanup, the program would have no safe way to recover.

### Purposes

- To guarantee that cleanup operations (destructors) never fail during unwinding.
- To enable strong-guarantee idioms (copy-and-swap requires `noexcept` swap).
- To support move semantics optimisations (e.g., `vector` reallocation uses move only if `noexcept`).
- To provide reliable, fail-safe primitives.

### Syntax Rules and Structure

**General Syntax**
```cpp
void function() noexcept;                 // declares nothrow guarantee
```

**Component Breakdown**

| Mechanism | Role |
|-----------|------|
| `noexcept` specifier | Declares that the function will not throw. |
| `noexcept(true)` | Explicit form (default when `noexcept` alone is used). |
| Destructors | Implicitly `noexcept` (since C++11). |

**Syntax Rules**
- If a `noexcept` function throws, `std::terminate` is called.
- Core Guidelines F.6: “If your function must not throw, declare it `noexcept`”.
- Destructors are implicitly `noexcept` since C++11; throwing from them terminates the program.

**Constraints and Limitations**
- `noexcept` is a promise, not a compile-time guarantee; violating it terminates the program.
- Some functions cannot realistically be `noexcept` (e.g., allocation functions).
- Overuse of `noexcept` can lead to termination where recovery was possible.

### Annotated Code Example

```cpp
#include <iostream>
#include <utility>
#include <vector>

class Resource {
    int* data_;
public:
    Resource() : data_(new int(0)) {}
    ~Resource() { delete data_; }          // implicitly noexcept

    Resource(Resource&& other) noexcept    // nothrow move
        : data_(std::exchange(other.data_, nullptr)) {}

    Resource& operator=(Resource&& other) noexcept {
        if (this != &other) {
            delete data_;
            data_ = std::exchange(other.data_, nullptr);
        }
        return *this;
    }

    friend void swap(Resource& a, Resource& b) noexcept {
        std::swap(a.data_, b.data_);
    }
};

int main() {
    std::vector<Resource> v;
    v.reserve(10);                         // no reallocation due to noexcept move
    v.emplace_back();
    std::cout << "Size: " << v.size() << '\n';
}
```

**Expected Output**
```
Size: 1
```

**Why this output?**  
The `noexcept` move constructor and `noexcept` swap ensure that vector operations can safely use move semantics, supporting the strong guarantee for `push_back`.

### Real-World Cases

- **Destructors** – must be nothrow to avoid termination during unwinding.
- **`swap` functions** – required for copy-and-swap strong guarantee.
- **Move constructors** – `noexcept` enables efficient reallocation in containers.

---

## 5. Exception Safety Levels (Complete Hierarchy)

### Definitions

**Core Definition**
Exception safety levels form a hierarchy of guarantees from strongest (nothrow) to weakest (no guarantee), with the strong and basic guarantees in between.

**Technical Definition**
The SEI CERT C++ Coding Standard defines three guarantees from most to least desired: **Strong** (no observable effects on program state if the operation throws), **Basic** (preserves invariants and prevents leaks), and **None** (neither strong nor basic; code is not exception safe). The nothrow guarantee sits above strong, as the strongest level. A function providing the strong guarantee also satisfies the basic guarantee.

**Beginner-Friendly Explanation**
Think of exception safety levels as a ladder. At the top is “never fails” (nothrow). Below that is “all or nothing” (strong). Below that is “no mess” (basic). At the bottom is “anything goes” (no guarantee), which is unacceptable.

### Purposes

- To provide a vocabulary for describing and comparing exception safety properties.
- To guide API design and documentation.
- To help developers choose the right level for each operation.
- To establish the minimum acceptable standard (basic guarantee).

### Syntax Rules and Structure (Comparison Table)

| Level | Guarantee | State After Exception | Typical Examples |
|-------|-----------|----------------------|------------------|
| **Nothrow** | Never throws | Unchanged (always succeeds) | Destructors, `swap`, move constructors |
| **Strong** | Commit-or-rollback | Unchanged (rolled back) | `vector::push_back`, copy-and-swap assignment |
| **Basic** | Valid state, no leaks | May differ, but invariants hold | Most STL operations |
| **None** | No guarantee | May be invalid, leak, corrupt | Defective code |

*Source: adapted fromand.*

**Syntax Rules**
- Strong implies basic.
- Nothrow implies strong and basic (trivially).
- No guarantee is never acceptable.

**Constraints and Limitations**
- Not all operations can achieve the strong guarantee (e.g., streaming output).
- The basic guarantee is the minimum acceptable level for all code.

### Annotated Code Example

```cpp
#include <iostream>
#include <vector>
#include <stdexcept>

void demonstrate_levels() {
    std::vector<int> v = {1, 2, 3};

    // Basic guarantee: even if an exception is thrown here,
    // v remains valid (no leaks, invariants intact).
    try {
        v.push_back(4);
        throw std::runtime_error("post-push failure");
    } catch (const std::exception&) {
        std::cout << "Basic: size=" << v.size() << '\n'; // 4
    }

    // Strong guarantee: validate before mutating.
    try {
        int value = -1;
        if (value < 0) throw std::invalid_argument("negative");
        v.push_back(value);
    } catch (const std::exception&) {
        std::cout << "Strong: size=" << v.size() << '\n'; // still 4
    }
}

int main() {
    demonstrate_levels();
}
```

**Expected Output**
```
Basic: size=4
Strong: size=4
```

**Why this output?**  
The basic case shows that the vector remains valid (size 4) after a post-mutation exception. The strong case shows that validation prevents the mutation entirely, so the size remains 4.

### Real-World Cases

- **API documentation** – stating the guarantee level for each function.
- **Code reviews** – checking that every function provides at least the basic guarantee.
- **Container design** – standard containers document their guarantee levels.

---

## 6. RAII and Exception Safety

### Definitions

**Core Definition**
RAII (Resource Acquisition Is Initialization) is a C++ programming idiom in which resource management is tied to object lifetime: resources are acquired in constructors and released in destructors, guaranteeing cleanup during stack unwinding.

**Technical Definition**
RAII ensures that every resource is owned by an object whose destructor releases it. Because destructors run automatically when objects go out of scope—including during exception-driven stack unwinding—RAII provides the foundation for exception safety. The SEI CERT C++ Coding Standard recommends RAII as the preferred approach: “A better approach would be to employ RAII. This forces every object to 'clean up after itself' in the face of abnormal behavior”.

**Beginner-Friendly Explanation**
RAII is the “self-cleaning” principle: whatever you acquire (memory, files, locks), wrap it in an object whose destructor releases it. When an exception unwinds the stack, destructors run automatically, so nothing is leaked.

### Purposes

- To guarantee resource release during stack unwinding.
- To eliminate manual cleanup code that can be bypassed by exceptions.
- To provide the foundation for all exception safety guarantees.
- To simplify code by removing explicit `try`/`catch` for cleanup.

### Syntax Rules and Structure

**General Pattern**
```cpp
class RaiiWrapper {
    Resource resource_;
public:
    RaiiWrapper() : resource_(acquire()) {}
    ~RaiiWrapper() { release(resource_); }
    // non-copyable or move-only to prevent double release
};
```

**Component Breakdown**

| Component | Role |
|-----------|------|
| Constructor | Acquires the resource. |
| Destructor | Releases the resource (must be nothrow). |
| Copy/move control | Prevents double-free or enables transfer. |

**Syntax Rules**
- Destructors must not throw (implicitly `noexcept` since C++11).
- Use smart pointers for heap memory, standard streams for files, and `lock_guard` for mutexes.
- Avoid raw owning pointers.

**Constraints and Limitations**
- RAII requires resources to be wrapped in objects; this may involve some boilerplate.
- Custom resources need custom RAII wrappers.

### Annotated Code Example

```cpp
#include <iostream>
#include <memory>
#include <stdexcept>

void process() {
    auto data = std::make_unique<int[]>(100); // RAII-managed array
    throw std::runtime_error("failure");       // data is freed automatically
}

int main() {
    try {
        process();
    } catch (const std::exception& e) {
        std::cout << "Caught: " << e.what() << '\n';
    }
}
```

**Expected Output**
```
Caught: failure
```

**Why this output?**  
`unique_ptr` owns the array; when the exception unwinds the stack, the `unique_ptr` destructor runs, freeing the memory. No leak occurs.

### Real-World Cases

- **Smart pointers** – `unique_ptr` and `shared_ptr` for heap memory.
- **File streams** – `std::fstream` closes files automatically.
- **Lock guards** – `std::lock_guard` releases mutexes on scope exit.
- **Transaction guards** – custom RAII wrappers for database rollback.

---

## 7. Exception Safety Best Practices

### Definitions

**Core Definition**
Exception safety best practices are guidelines for designing and implementing code that provides appropriate exception safety guarantees, as codified in the C++ Core Guidelines and the SEI CERT C++ Coding Standard.

**Technical Definition**
Best practices include: providing at least the basic guarantee for all operations; using RAII for all resource management; implementing the strong guarantee via copy-and-swap for assignment; marking destructors, `swap`, and move operations `noexcept`; and documenting the guarantee level of public APIs. The Core Guidelines emphasise that destructors, deallocation functions, `swap`, and exception type copy/move construction must never fail.

**Beginner-Friendly Explanation**
Best practices for exception safety are simple rules: always use RAII, make destructors and swaps `noexcept`, use copy-and-swap for assignment, and document what guarantee your functions provide.

### Key Guidelines

| Guideline | Summary |
|-----------|---------|
| **Provide at least the basic guarantee** | No leaks, invariants intact. |
| **Use RAII for every resource** | Guarantees cleanup during unwinding. |
| **Use copy-and-swap for strong assignment** | All-or-nothing semantics. |
| **Mark destructors, `swap`, and moves `noexcept`** | Essential for strong-guarantee idioms. |
| **Validate before mutating** | Enables strong guarantee without copies. |
| **Document the guarantee level** | Callers need to know what to expect. |
| **Avoid raw owning pointers** | Use smart pointers instead. |
| **Use `noexcept` intentionally** | Core Guidelines F.6. |

### Annotated Code Example

```cpp
#include <iostream>
#include <memory>
#include <stdexcept>
#include <utility>

class Buffer {
    std::unique_ptr<int[]> data_;
    std::size_t size_;
public:
    explicit Buffer(std::size_t n) : data_(new int[n]()), size_(n) {}

    // Copy-and-swap: strong guarantee
    Buffer(const Buffer& other) : data_(new int[other.size_]), size_(other.size_) {
        std::copy(other.data_.get(), other.data_.get() + size_, data_.get());
    }

    Buffer(Buffer&& other) noexcept
        : data_(std::move(other.data_)), size_(std::exchange(other.size_, 0)) {}

    Buffer& operator=(Buffer other) noexcept {   // copy made in parameter
        swap(*this, other);
        return *this;
    }

    friend void swap(Buffer& a, Buffer& b) noexcept {
        using std::swap;
        swap(a.data_, b.data_);
        swap(a.size_, b.size_);
    }

    std::size_t size() const noexcept { return size_; }
};

int main() {
    Buffer a(10);
    Buffer b(20);
    a = b;                                      // strong guarantee
    std::cout << a.size() << '\n';              // 20
}
```

**Expected Output**
```
20
```

**Why this output?**  
The copy-and-swap assignment provides the strong guarantee: if the copy constructor throws, `a` is untouched; if it succeeds, `swap` (which is `noexcept`) commits the change. The destructor and `swap` are `noexcept`, satisfying the requirements for strong exception safety.

### Real-World Cases

- **Code reviews** – checking for missing `noexcept` on `swap` and destructors.
- **Static analysis** – tools like clang-tidy enforce Core Guidelines.
- **API design** – documenting guarantee levels for public functions.

---

## References

- Exceptions (exception safety guarantees) — cppreference – https://en.cppreference.com/w/cpp/language/exceptions
- ERR56-CPP. Guarantee exception safety — SEI CERT C++ Coding Standard – https://cmu-sei.github.io/secure-coding-standards/sei-cert-cpp-coding-standard/rules/exceptions-and-error-handling-err/err56-cpp/
- Write correct exception-safe code using RAII and strong/basic guarantees — Dr-Sergey/learn_cpp – https://raw.githubusercontent.com/Dr-Sergey/learn_cpp/08279578ce23b1cbf2d3394357620600db618eca/11_Error_Handling/Write_correct_exception-safe_code_using_RAII_and_strongbasic_guarantees.md
- Apply the copy-and-swap idiom for exception-safe assignment — Dr-Sergey/learn_cpp – https://raw.githubusercontent.com/Dr-Sergey/learn_cpp/575e0775d6aa9df1b80d1343af811215cf9c1745/17_Best_Practices_and_Idioms/Apply_the_copy-and-swap_idiom_for_exception-safe_assignment.md
- C++ Core Guidelines — F.6: If your function must not throw, declare it `noexcept` – https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rf-noexcept
- C++ Core Guidelines — E.16: Destructors, deallocation, `swap`, and exception type copy/move construction must never fail – https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Re-never-fail
- Exceptions (GCC manual, exception safety levels) – https://gcc.gnu.org/onlinedocs/libstdc++/manual/exception_safety.html
- MEM44-CPP. Do not leak resources when handling exceptions — SEI CERT C++ – https://wiki.sei.cmu.edu/confluence/display/cplusplus/MEM44-CPP.+Do+not+leak+resources+when+handling+exceptions
- `std::vector::push_back` — cppreference (strong guarantee example) – https://en.cppreference.com/w/cpp/container/vector/push_back

---

This cheat sheet provides a comprehensive, structured reference for C++ exception safety, covering the concept, the basic, strong, and nothrow guarantees, the complete hierarchy of safety levels, RAII, and best practices, with annotated code examples, expected outputs, real-world cases, and authoritative references.