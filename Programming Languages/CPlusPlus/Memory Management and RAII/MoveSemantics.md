# C++ Move Semantics — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
Move semantics is a C++11 feature that allows resources (such as dynamically allocated memory) to be transferred from one object to another instead of being copied, typically from a temporary object that is about to be destroyed.

**Technical Definition**
Move semantics is based on **rvalue references** (`T&&`), which bind to temporaries and to objects explicitly marked as movable via `std::move`. A type participates in move semantics by providing a **move constructor** and a **move assignment operator**, which transfer ownership of resources from the source (leaving it in a valid but unspecified state) rather than duplicating them. The C++11 standard library was updated to use move semantics throughout—return values, container reallocation, and algorithm implementations all benefit.

**Beginner-Friendly Explanation**
Copying a large object duplicates all its data, which is slow. Moving an object is like handing someone the keys to your car instead of building them a copy of the car: the new owner gets the resource, and you are left with an empty garage. Move semantics makes this safe and automatic in C++.

### Key Characteristics

- **Resource transfer, not duplication** – moves are typically O(1) for heap-owning types.
- **Rvalue references** – `T&&` binds to temporaries and `std::move`-cast lvalues.
- **Moved-from state** – the source of a move is left valid but unspecified; it must remain destructible and assignable.
- **`noexcept` matters** – move constructors and move assignment operators should be `noexcept` to enable standard library optimisations (e.g., `std::vector` reallocation).
- **Rule of Five / Rule of Zero** – defining any of the five special members (destructor, copy/move constructors, copy/move assignment) typically requires defining or deleting all of them.
- **Copy elision** – since C++17, certain copies/moves are guaranteed to be elided entirely.
- **Perfect forwarding** – `std::forward` preserves value categories in template code.

### Prerequisites

- Solid understanding of C++11 basics (`auto`, references, `const`).
- Familiarity with constructors, destructors, and copy semantics.
- Knowledge of the Rule of Three (pre-C++11 copy control).
- Understanding of exception safety and `noexcept`.
- Awareness of the standard containers and their reallocation behaviour.

### Related Programming Areas

- **RAII** – move semantics complements RAII by allowing efficient transfer of resource-owning objects.
- **Smart pointers** – `std::unique_ptr` is move-only; `shared_ptr` is movable and copyable.
- **Standard containers** – `std::vector` uses moves during reallocation when the element type's move constructor is `noexcept`.
- **Perfect forwarding** – `std::forward` is built on the same rvalue-reference machinery.
- **Copy elision** – eliminates copies/moves entirely in certain contexts.

### Core Concepts / Features

1. Move semantics (the overarching concept)
2. Rvalue references
3. `std::move`
4. Move constructors
5. Move assignment operators
6. Rule of Five
7. Rule of Zero
8. Copy elision
9. Perfect forwarding
10. `std::forward`
11. Move semantics best practices

---

## 1. Move Semantics (The Overarching Concept)

### Definitions

**Core Definition**
Move semantics is the set of language rules and library facilities that allow an object's resources to be transferred to another object, rather than copied.

**Technical Definition**
Move semantics (since C++11) enables the transfer of resources—heap memory, file handles, ownership—from one object to another. It is implemented via rvalue references, move constructors, and move assignment operators. A move operation leaves the source object in a *valid but unspecified* state, allowing the destination to take ownership of the source's resources without duplication. Move semantics is used automatically by the standard library (return values, container growth, algorithm swaps) and can be invoked explicitly with `std::move`.

**Beginner-Friendly Explanation**
Move semantics means “steal resources instead of copying them.” If you have a big vector and you're about to throw it away, you can give its internal array to a new vector instead of copying every element. This is much faster.

### Purposes

- To improve performance by avoiding deep copies.
- To enable transfer of non-copyable resources (e.g., `unique_ptr`).
- To make containers reallocate efficiently.
- To support return-by-value of large objects.
- To reduce memory usage and allocations.

### Syntax Rules and Structure

**General Pattern**
```cpp
class MyType {
public:
    MyType(MyType&& other) noexcept;             // move constructor
    MyType& operator=(MyType&& other) noexcept;  // move assignment
};
```

**Component Breakdown**

| Member | Signature | Role |
|--------|-----------|------|
| Move constructor | `T(T&&) noexcept` | Transfers resources from an rvalue. |
| Move assignment | `T& operator=(T&&) noexcept` | Transfers resources from an rvalue. |
| `std::move` | `std::move(x)` | Casts an lvalue to an rvalue. |

**Syntax Rules**
- Move constructors and move assignment should be `noexcept`.
- The moved-from object must be left in a valid state (destructible, assignable).
- `std::move` does not move anything; it is a cast.

**Constraints and Limitations**
- If a type has no move constructor, the compiler falls back to copying.
- Move semantics only applies to rvalues; lvalues must be explicitly cast with `std::move`.

### Annotated Code Example

```cpp
#include <iostream>
#include <utility>
#include <vector>

class Buffer {
    int* data_;
public:
    explicit Buffer(int v) : data_(new int(v)) {}
    ~Buffer() { delete data_; }

    Buffer(const Buffer& other) : data_(new int(*other.data_)) {
        std::cout << "Copy\n";
    }

    Buffer(Buffer&& other) noexcept
        : data_(std::exchange(other.data_, nullptr)) {
        std::cout << "Move\n";
    }

    int value() const { return data_ ? *data_ : -1; }
};

int main() {
    Buffer a(10);
    Buffer b = a;                      // copy
    Buffer c = std::move(a);           // move
    std::cout << b.value() << ' ' << c.value() << '\n';
}
```

**Expected Output**
```
Copy
Move
10 10
```

**Why this output?**  
`Buffer b = a` copies `a`; `Buffer c = std::move(a)` moves from `a` (leaving `a` empty). `b` and `c` both hold `10`.

### Real-World Cases

- **Standard containers** – `std::vector` reallocation uses moves.
- **Return by value** – returning large objects efficiently.
- **`unique_ptr`** – move-only ownership transfer.
- **String and buffer classes** – avoiding deep copies.

---

## 2. Rvalue References

### Definitions

**Core Definition**
An rvalue reference (`T&&`) is a reference that binds only to rvalues (temporaries or objects explicitly marked as movable), enabling move semantics and perfect forwarding.

**Technical Definition**
An rvalue reference to type `T` is created with the syntax `T&&`. It binds to rvalues and extends the lifetime of a temporary. When used as a function parameter, it can distinguish rvalue arguments from lvalue arguments, enabling overload resolution to select move-aware implementations. In a template context, `T&&` is a *forwarding reference* (also called a universal reference) and deduces to `T&` for lvalues.

**Beginner-Friendly Explanation**
An rvalue reference is a special kind of reference that can “claim” a temporary object. Because the temporary is about to be destroyed anyway, you can safely steal its resources instead of copying them.

### Purposes

- To enable move semantics (efficient resource transfer).
- To support perfect forwarding in templates.
- To distinguish lvalues from rvalues in overload resolution.
- To implement move constructors and move assignment operators.

### Syntax Rules and Structure

**General Syntax**
```cpp
T&& rref = temporary;                  // binds to rvalue
void f(T&& x);                         // rvalue reference parameter
template<class T> void g(T&& x);       // forwarding reference
```

**Component Breakdown**

| Form | Binds To |
|------|----------|
| `T&` | Lvalues only. |
| `const T&` | Lvalues and rvalues (read-only). |
| `T&&` | Rvalues only. |
| `T&&` (template) | Forwarding reference (binds to anything). |

**Syntax Rules**
- `T&&` binds only to rvalues; `T&&` in a template context is a forwarding reference.
- Use `std::move` to cast an lvalue to an rvalue.
- Move constructors take `T&&` parameters.

**Constraints and Limitations**
- An rvalue reference is itself an lvalue; use `std::move` to pass it along as an rvalue.
- Overloading on `T&&` and `const T&` requires careful design.

### Annotated Code Example

```cpp
#include <iostream>
#include <utility>

void process(int& x)  { std::cout << "lvalue: " << x << '\n'; }
void process(int&& x) { std::cout << "rvalue: " << x << '\n'; }

int main() {
    int a = 10;
    process(a);                        // lvalue
    process(20);                       // rvalue
    process(std::move(a));             // rvalue (cast)
}
```

**Expected Output**
```
lvalue: 10
rvalue: 20
rvalue: 10
```

**Why this output?**  
`a` is an lvalue; `20` is a temporary rvalue; `std::move(a)` casts `a` to an rvalue, selecting the rvalue overload.

### Real-World Cases

- **Move constructors** – `MyType(MyType&& other) noexcept;`.
- **Perfect forwarding** – `template<class T> void wrapper(T&& arg) { f(std::forward<T>(arg)); }`.
- **Optimised string/vector operations** – avoiding deep copies.

---

## 3. `std::move`

### Definitions

**Core Definition**
`std::move` is a function template that casts its argument to an rvalue reference, signalling that the argument may be moved from.

**Technical Definition**
`std::move(x)` (since C++11, in `<utility>`) is equivalent to `static_cast<std::remove_reference_t<T>&&>(x)`. It does **not** perform any move itself; it is a cast that enables move semantics by making an lvalue eligible for rvalue-reference overloads. After `std::move(x)`, `x` must not be used except to assign a new value or destroy it.

**Beginner-Friendly Explanation**
`std::move` is a “permission slip” that says “you may steal from this object.” It does not move anything by itself; it just changes how the object is treated.

### Purposes

- To cast an lvalue to an rvalue, enabling move semantics.
- To signal intent to transfer ownership.
- To select move overloads in function calls and constructors.

### Syntax Rules and Structure

**General Syntax**
```cpp
auto y = std::move(x);                 // cast x to rvalue
f(std::move(x));                       // pass x as rvalue
```

**Component Breakdown**

| Aspect | Description |
|--------|-------------|
| Header | `<utility>`. |
| Effect | Cast to rvalue reference. |
| Postcondition | `x` is valid but unspecified; must not be used except for assignment/destruction. |

**Syntax Rules**
- `std::move` is a cast, not an operation.
- After moving, the source object is in a valid but unspecified state.
- `std::move` on a `const` object yields a `const T&&`, which typically does not enable moves (copies may still occur).

**Constraints and Limitations**
- Using a moved-from object (except to assign or destroy) is a logic error, not undefined behaviour.
- `std::move` on `const` objects often results in copies.

### Annotated Code Example

```cpp
#include <iostream>
#include <string>
#include <utility>

int main() {
    std::string a = "hello";
    std::string b = std::move(a);      // move a into b

    std::cout << "b: " << b << '\n';   // hello
    std::cout << "a: '" << a << "'\n"; // a is valid but empty (typically)
}
```

**Expected Output**
```
b: hello
a: ''
```

**Why this output?**  
`std::move(a)` casts `a` to an rvalue, enabling the move constructor of `std::string`; `a` is left in a valid but unspecified state (typically empty).

### Real-World Cases

- **Transferring ownership** – `auto p = std::move(unique_ptr);`.
- **Moving into containers** – `v.push_back(std::move(str));`.
- **Returning local objects** – moves are applied automatically (NRVO), but `std::move` can be used explicitly when needed (carefully).

---

## 4. Move Constructors

### Definitions

**Core Definition**
A move constructor is a constructor that transfers resources from an rvalue of the same type, leaving the source in a valid but unspecified state.

**Technical Definition**
A move constructor has the signature `T(T&& other)` (since C++11). It is typically `noexcept` and is selected by overload resolution when the argument is an rvalue. The compiler generates a move constructor automatically if no user-declared copy operations, destructor, or move operations are present; the default move constructor performs member-wise moves.

**Beginner-Friendly Explanation**
A move constructor is the “constructor that steals.” When you create a new object from a temporary, the move constructor transfers the temporary's resources to the new object.

### Purposes

- To efficiently construct an object from a temporary.
- To transfer ownership of resources.
- To enable standard library optimisations (e.g., `vector` reallocation).

### Syntax Rules and Structure

**General Syntax**
```cpp
T(T&& other) noexcept;
```

**Component Breakdown**

| Aspect | Description |
|--------|-------------|
| Signature | `T(T&& other)` |
| Exception specification | Should be `noexcept`. |
| Effect | Transfers resources from `other`; leaves `other` valid but unspecified. |

**Syntax Rules**
- The move constructor must not throw if it is to be used by `std::vector` reallocation.
- The moved-from object must remain destructible and assignable.
- If a move constructor is user-declared, the copy constructor is implicitly deleted (unless also user-declared).

**Constraints and Limitations**
- Default move constructors are generated only if no user-declared copy operations or destructor exist.
- Move constructors of `const` objects are not usable for moves.

### Annotated Code Example

```cpp
#include <iostream>
#include <utility>

class Buffer {
    int* data_;
public:
    explicit Buffer(int v) : data_(new int(v)) {}
    ~Buffer() { delete data_; }

    Buffer(const Buffer& other) : data_(new int(*other.data_)) {}
    Buffer(Buffer&& other) noexcept
        : data_(std::exchange(other.data_, nullptr)) {}

    int value() const { return data_ ? *data_ : -1; }
};

int main() {
    Buffer a(42);
    Buffer b = std::move(a);           // move constructor
    std::cout << b.value() << ' ' << a.value() << '\n';
}
```

**Expected Output**
```
42 -1
```

**Why this output?**  
The move constructor transfers the pointer from `a` to `b` and sets `a`'s pointer to `nullptr`; `a` is left empty.

### Real-World Cases

- **Standard containers** – `vector` uses move constructors during reallocation.
- **Return by value** – move constructors are used when returning local objects (unless elided).
- **`unique_ptr`** – move-only ownership transfer.

---

## 5. Move Assignment Operators

### Definitions

**Core Definition**
A move assignment operator transfers resources from an rvalue of the same type to `*this`, releasing the current resources of `*this`.

**Technical Definition**
A move assignment operator has the signature `T& operator=(T&& other)` (since C++11). It is typically `noexcept`. It must handle self-assignment safely (or document that self-assignment is not required), release the current resources of `*this`, transfer resources from `other`, and leave `other` in a valid but unspecified state.

**Beginner-Friendly Explanation**
A move assignment operator is the “steal and replace” operation: it throws away what the object currently owns and takes the rvalue's resources instead.

### Purposes

- To efficiently transfer resources into an existing object.
- To replace an object's contents without copying.
- To support move-aware assignment in the standard library.

### Syntax Rules and Structure

**General Syntax**
```cpp
T& operator=(T&& other) noexcept;
```

**Component Breakdown**

| Aspect | Description |
|--------|-------------|
| Signature | `T& operator=(T&& other)` |
| Exception specification | Should be `noexcept`. |
| Self-assignment | Must be handled safely or documented as unsupported. |

**Syntax Rules**
- Release current resources before taking new ones.
- Transfer resources from `other`.
- Leave `other` in a valid but unspecified state.

**Constraints and Limitations**
- Default move assignment is generated only if no user-declared copy operations or destructor exist.

### Annotated Code Example

```cpp
#include <iostream>
#include <utility>

class Buffer {
    int* data_;
public:
    explicit Buffer(int v) : data_(new int(v)) {}
    ~Buffer() { delete data_; }

    Buffer(const Buffer& other) : data_(new int(*other.data_)) {}
    Buffer(Buffer&& other) noexcept
        : data_(std::exchange(other.data_, nullptr)) {}

    Buffer& operator=(Buffer&& other) noexcept {
        if (this != &other) {
            delete data_;
            data_ = std::exchange(other.data_, nullptr);
        }
        return *this;
    }

    int value() const { return data_ ? *data_ : -1; }
};

int main() {
    Buffer a(10), b(20);
    b = std::move(a);                  // move assignment
    std::cout << b.value() << ' ' << a.value() << '\n';
}
```

**Expected Output**
```
10 -1
```

**Why this output?**  
The move assignment releases `b`'s old resource (`20`), transfers `a`'s resource to `b`, and leaves `a` empty.

### Real-World Cases

- **Reassigning containers** – `v = std::move(other);`.
- **Returning objects** – move assignment is used in return-value optimisations.
- **Swapping** – `std::swap` uses move assignment.

---

## 6. Rule of Five

### Definitions

**Core Definition**
The Rule of Five states that if a class defines (or deletes) any of the five special member functions—destructor, copy constructor, copy assignment, move constructor, move assignment—it should explicitly define or delete all five.

**Technical Definition**
The five special member functions are: destructor, copy constructor, copy assignment operator, move constructor, and move assignment operator. In C++11, defining any one of these suppresses the implicit generation of some or all of the others, leading to subtle bugs. The Rule of Five says: if you need to customise one, customise or delete all five to make the ownership semantics explicit.

**Beginner-Friendly Explanation**
If you write one of the five special functions (like a destructor), the compiler stops generating the others automatically. So you should either write all five or write none (Rule of Zero).

### Purposes

- To ensure correct resource management.
- To avoid subtle bugs from partially generated special members.
- To make ownership semantics explicit.

### Syntax Rules and Structure

**The Five Special Members**

| Member | Signature |
|--------|-----------|
| Destructor | `~T()` |
| Copy constructor | `T(const T&)` |
| Copy assignment | `T& operator=(const T&)` |
| Move constructor | `T(T&&) noexcept` |
| Move assignment | `T& operator=(T&&) noexcept` |

**Syntax Rules**
- If you define any of the five, define or delete all five.
- Use `= default` for members that should be defaulted.
- Use `= delete` for members that should not exist.

**Constraints and Limitations**
- Defining a destructor suppresses the implicit move operations.
- Defining a copy operation suppresses the implicit move operations.

### Annotated Code Example

```cpp
#include <iostream>

class Buffer {
    int* data_;
public:
    explicit Buffer(int v) : data_(new int(v)) {}
    ~Buffer() { delete data_; }                        // 1
    Buffer(const Buffer& other)                        // 2
        : data_(new int(*other.data_)) {}
    Buffer& operator=(const Buffer& other) {           // 3
        if (this != &other) { *data_ = *other.data_; }
        return *this;
    }
    Buffer(Buffer&& other) noexcept                    // 4
        : data_(std::exchange(other.data_, nullptr)) {}
    Buffer& operator=(Buffer&& other) noexcept {       // 5
        if (this != &other) {
            delete data_;
            data_ = std::exchange(other.data_, nullptr);
        }
        return *this;
    }
};
```

**Expected Output**  
(No output; the class defines all five special members.)

### Real-World Cases

- **Resource-owning classes** – buffers, file handles, sockets.
- **RAII wrappers** – smart pointers and lock guards.

---

## 7. Rule of Zero

### Definitions

**Core Definition**
The Rule of Zero states that a class should not define any of the five special member functions; instead, it should rely on the compiler-generated defaults and use RAII members (smart pointers, containers) to manage resources.

**Technical Definition**
The Rule of Zero (popularised by R. Martinho Fernandes) says that classes should either define all five special members (Rule of Five) or none of them (Rule of Zero). A class that follows the Rule of Zero has its resource management delegated to member objects (e.g., `std::unique_ptr`, `std::vector`), and the compiler-generated special members are correct by default.

**Beginner-Friendly Explanation**
The Rule of Zero says: don't write destructors or copy/move operations at all. Use smart pointers and containers instead, and let the compiler generate everything correctly.

### Purposes

- To eliminate boilerplate and potential bugs.
- To make resource management automatic via RAII members.
- To leverage the compiler's correct default implementations.

### Syntax Rules and Structure

**General Pattern**
```cpp
class Widget {
    std::unique_ptr<Impl> impl_;
    std::vector<int> data_;
    // No user-defined destructor, copy, or move operations
};
```

**Component Breakdown**

| Component | Role |
|-----------|------|
| RAII members | Manage resources automatically. |
| No user-defined special members | Compiler-generated defaults are correct. |

**Syntax Rules**
- Use RAII members for all resources.
- Do not define destructor, copy, or move operations.

**Constraints and Limitations**
- Requires all members to be RAII-aware or trivially copyable.
- Some scenarios (e.g., legacy resources) still need the Rule of Five.

### Annotated Code Example

```cpp
#include <memory>
#include <vector>
#include <iostream>

class Widget {
    std::unique_ptr<int> value_;
    std::vector<int> data_;
public:
    explicit Widget(int v) : value_(std::make_unique<int>(v)), data_{1, 2, 3} {}
    int value() const { return *value_; }
    std::size_t size() const { return data_.size(); }
};

int main() {
    Widget w(42);
    Widget w2 = std::move(w);          // move is generated automatically
    std::cout << w2.value() << ' ' << w2.size() << '\n';
}
```

**Expected Output**
```
42 3
```

**Why this output?**  
`Widget` has no user-defined special members; the compiler-generated move constructor correctly moves the `unique_ptr` and the `vector`.

### Real-World Cases

- **Modern C++ classes** – using smart pointers and containers.
- **Library design** – minimising special member declarations.

---

## 8. Copy Elision

### Definitions

**Core Definition**
Copy elision is the compiler optimisation that eliminates a copy or move operation by constructing the object directly in its target location.

**Technical Definition**
Copy elision (including **Return Value Optimisation**, RVO, and **Named Return Value Optimisation**, NRVO) eliminates copy/move constructor calls. Since C++17, copy elision is **guaranteed** when a prvalue of the same type initialises an object (e.g., `T obj = T()` or `return T();`). For NRVO (returning a named local), elision is permitted but not guaranteed.

**Beginner-Friendly Explanation**
Copy elision means the compiler skips the copy or move entirely and builds the object where it needs to be in the first place. In C++17, some of these elisions are guaranteed by the language.

### Purposes

- To eliminate unnecessary copies and moves.
- To allow non-copyable, non-movable types to be returned by value.
- To improve performance.

### Syntax Rules and Structure

**Guaranteed Elision Contexts (C++17)**

| Context | Example |
|---------|---------|
| Prvalue initialisation | `T obj = T();` |
| Return of prvalue | `return T();` |
| Function argument | `f(T());` |

**Permitted but Not Guaranteed**

| Context | Example |
|---------|---------|
| NRVO | `T local; return local;` |
| Exception object | `throw local;` |

**Syntax Rules**
- In guaranteed elision contexts, no copy/move constructor is required to be accessible.
- For NRVO, the compiler may elide but is not required to.

**Constraints and Limitations**
- NRVO is not guaranteed; do not rely on it for correctness.

### Annotated Code Example

```cpp
#include <iostream>

struct NonCopyable {
    NonCopyable() { std::cout << "Constructed\n"; }
    NonCopyable(const NonCopyable&) = delete;
    NonCopyable(NonCopyable&&) = delete;
};

NonCopyable create() {
    return NonCopyable();              // guaranteed elision (C++17)
}

int main() {
    NonCopyable obj = create();        // no copy/move
    std::cout << "Done\n";
}
```

**Expected Output**
```
Constructed
Done
```

**Why this output?**  
C++17 guarantees that the prvalue from `create()` is constructed directly in `obj`, even though `NonCopyable` has deleted copy and move constructors.

### Real-World Cases

- **Factory functions** – returning non-copyable types by value.
- **`std::lock_guard`** – returning lock guards from helper functions.
- **Immutable types** – types that intentionally disable copying.

---

## 9. Perfect Forwarding

### Definitions

**Core Definition**
Perfect forwarding is the technique of passing arguments to another function while preserving their value category (lvalue/rvalue) and `const`-ness, so that the receiving function sees exactly the same types and value categories as the caller.

**Technical Definition**
Perfect forwarding relies on **forwarding references** (`T&&` in a template context) and `std::forward<T>`. A forwarding reference deduces `T` as `U&` for lvalue arguments and `U` for rvalue arguments. `std::forward<T>(arg)` casts `arg` back to its original value category, enabling move semantics to be preserved through wrapper functions.

**Beginner-Friendly Explanation**
Perfect forwarding means “pass the argument along exactly as you received it.” If the caller passed an lvalue, the wrapper passes an lvalue; if the caller passed an rvalue, the wrapper passes an rvalue. This allows wrappers (like `std::make_unique`, `std::invoke`) to preserve move semantics.

### Purposes

- To write wrapper functions that preserve argument value categories.
- To enable move semantics through forwarding layers.
- To support generic factory functions and adaptors.
- To avoid unnecessary copies in template code.

### Syntax Rules and Structure

**General Syntax**
```cpp
template<class T>
void wrapper(T&& arg) {
    f(std::forward<T>(arg));           // preserve value category
}
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `T&&` | Forwarding reference (deduces `T` based on argument). |
| `std::forward<T>(arg)` | Casts `arg` to `T&&`, preserving value category. |

**Syntax Rules**
- `T&&` in a template is a forwarding reference only if `T` is a deduced template parameter.
- `std::forward` must be given the deduced `T`.
- Use `std::forward` only for forwarding references.

**Constraints and Limitations**
- `std::forward` on a non-forwarding reference is usually a bug.
- Overuse of perfect forwarding can complicate overload resolution and diagnostics.

### Annotated Code Example

```cpp
#include <iostream>
#include <string>
#include <utility>

void process(std::string& s)  { std::cout << "lvalue: " << s << '\n'; }
void process(std::string&& s) { std::cout << "rvalue: " << s << '\n'; }

template<class T>
void wrapper(T&& arg) {
    process(std::forward<T>(arg));
}

int main() {
    std::string a = "hello";
    wrapper(a);                        // lvalue
    wrapper(std::string("world"));     // rvalue
}
```

**Expected Output**
```
lvalue: hello
rvalue: world
```

**Why this output?**  
`wrapper(a)` deduces `T = std::string&` and forwards as an lvalue; `wrapper(std::string("world"))` deduces `T = std::string` and forwards as an rvalue.

### Real-World Cases

- **`std::make_unique` / `std::make_shared`** – forward constructor arguments.
- **`std::invoke`** – forwards callable arguments.
- **Generic wrappers** – logging, timing, or decorating functions.

---

## 10. `std::forward`

### Definitions

**Core Definition**
`std::forward<T>` is a function template that casts its argument to the value category it had when originally passed, given the deduced type `T`.

**Technical Definition**
`std::forward<T>(arg)` (since C++11, in `<utility>`) performs a conditional cast: if `T` is an lvalue reference type, it returns an lvalue; otherwise, it returns an rvalue. It is used exclusively with forwarding references to preserve the original value category.

**Beginner-Friendly Explanation**
`std::forward` is the “pass it along unchanged” tool. It preserves whether the original argument was an lvalue or an rvalue.

### Purposes

- To preserve value categories in forwarding functions.
- To enable perfect forwarding.
- To avoid unnecessary copies in template code.

### Syntax Rules and Structure

**General Syntax**
```cpp
std::forward<T>(arg);
```

**Component Breakdown**

| Aspect | Description |
|--------|-------------|
| Header | `<utility>`. |
| Effect | Conditional cast to `T&&`. |
| Usage | Only with forwarding references. |

**Syntax Rules**
- `T` must be the deduced template parameter.
- Do not use `std::forward` with non-forwarding references.

**Constraints and Limitations**
- Using `std::forward` on a non-forwarding reference can cause dangling references or unexpected moves.

### Annotated Code Example

```cpp
#include <iostream>
#include <utility>

template<class T>
void wrapper(T&& arg) {
    std::cout << "forwarding: " << std::forward<T>(arg) << '\n';
}

int main() {
    std::string a = "hello";
    wrapper(a);                        // lvalue
    wrapper(std::string("world"));     // rvalue
}
```

**Expected Output**
```
forwarding: hello
forwarding: world
```

**Why this output?**  
`std::forward<T>` preserves the value category: `a` is forwarded as an lvalue, and the temporary is forwarded as an rvalue.

### Real-World Cases

- **Factory functions** – forwarding constructor arguments.
- **Generic adaptors** – preserving value categories through wrappers.
- **`std::invoke`** – forwarding callable arguments.

---

## 11. Move Semantics Best Practices

### Definitions

**Core Definition**
Move semantics best practices are guidelines for implementing and using move operations correctly, safely, and efficiently.

**Technical Definition**
These practices include: mark move constructors and move assignment operators `noexcept` (Core Guidelines F.6); follow the Rule of Five or Rule of Zero; avoid using moved-from objects except for assignment/destruction; use `std::move` deliberately; and rely on copy elision where guaranteed. The Core Guidelines emphasise that move operations should be `noexcept` to enable standard library optimisations (e.g., `vector` reallocation).

**Beginner-Friendly Explanation**
Best practices for move semantics are simple: make moves `noexcept`, follow the Rule of Five or Zero, and don't use objects after you've moved from them.

### Key Guidelines

| Guideline | Summary |
|-----------|---------|
| **F.6** | If your function must not throw, declare it `noexcept`. |
| **Rule of Five** | If you define one special member, define or delete all five. |
| **Rule of Zero** | Prefer no user-defined special members; rely on RAII members. |
| **Mark moves `noexcept`** | Enables `vector` and other containers to use moves. |
| **Do not use moved-from objects** | They are valid but unspecified. |
| **Use `std::move` deliberately** | It is a cast, not an operation. |
| **Prefer copy elision** | Rely on guaranteed elision; don't pessimise with `std::move` in returns. |
| **Use `std::exchange`** | For clean move constructors and assignment. |
| **Avoid `const` rvalues** | `const T&&` typically does not enable moves. |

### Annotated Code Example

```cpp
#include <iostream>
#include <utility>
#include <vector>

class Buffer {
    int* data_;
public:
    explicit Buffer(int v) : data_(new int(v)) {}
    ~Buffer() { delete data_; }

    Buffer(const Buffer& other) : data_(new int(*other.data_)) {}
    Buffer(Buffer&& other) noexcept
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

    int value() const { return data_ ? *data_ : -1; }
};

int main() {
    std::vector<Buffer> v;
    v.reserve(2);
    v.emplace_back(10);
    v.emplace_back(20);
    std::cout << v[0].value() << ' ' << v[1].value() << '\n';
}
```

**Expected Output**
```
10 20
```

**Why this output?**  
The `noexcept` move constructor and move assignment enable `vector` to use moves during reallocation (if it occurs); the elements are correctly stored.

### Real-World Cases

- **Standard containers** – rely on `noexcept` moves for efficiency.
- **Smart pointers** – `unique_ptr` is move-only.
- **Factory functions** – returning large objects efficiently.

---

## References

- Move semantics — cppreference – https://en.cppreference.com/w/cpp/language/move
- Rvalue reference declarator: `&&` — Microsoft Learn – https://learn.microsoft.com/en-us/cpp/cpp/rvalue-reference-declarator-amp-amp
- `std::move` — cppreference – https://en.cppreference.com/w/cpp/utility/move
- `std::forward` — cppreference – https://en.cppreference.com/w/cpp/utility/forward
- Copy elision — cppreference – https://en.cppreference.com/w/cpp/language/copy_elision
- C++ Core Guidelines — F.6: If your function must not throw, declare it `noexcept` – https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rf-noexcept