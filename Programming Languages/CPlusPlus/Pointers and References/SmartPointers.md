# Smart Pointers — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
Smart pointers are RAII (Resource Acquisition Is Initialization) class templates in the C++ standard library that own and manage dynamically allocated objects, automatically destroying the managed object when the smart pointer goes out of scope, thereby preventing memory leaks and simplifying resource management.

**Technical Definition**
Smart pointers are class templates defined in `<memory>` that overload `operator*` and `operator->` to behave like pointers while encoding **ownership semantics** through their type. The three primary smart pointers are `std::unique_ptr<T>` (exclusive ownership), `std::shared_ptr<T>` (shared ownership via reference counting), and `std::weak_ptr<T>` (non-owning observer of a `shared_ptr`-managed object). They implement the RAII idiom: the destructor releases the owned resource, guaranteeing cleanup on both normal and exceptional exit. The C++ Core Guidelines mandate: “Use `unique_ptr` or `shared_ptr` to represent ownership” (R.20), “Prefer `unique_ptr` over `shared_ptr` unless you need to share ownership” (R.21), and “Never transfer ownership by a raw pointer (`T*`) or reference (`T&`)” (I.11).

**Beginner-Friendly Explanation**
A smart pointer is a **“self-cleaning pointer.”** A raw pointer is like a borrowed library book — you must remember to return it, or it's lost forever. A smart pointer is like a book with a built-in due-date alarm: when you're done with it (it goes out of scope), it automatically returns itself. There are three kinds: **`unique_ptr`** (one owner, like a personal diary), **`shared_ptr`** (multiple owners, like a shared Google Doc with a reference count), and **`weak_ptr`** (a bystander that can peek at the document but doesn't keep it alive). Smart pointers eliminate the most common C++ bug: forgetting to `delete`.

### Key Characteristics

- **Automatic cleanup**: The managed object is destroyed when the smart pointer's lifetime ends (RAII).
- **Ownership semantics**: The smart pointer type documents who is responsible for the object.
- **Exception safety**: Cleanup occurs even if an exception is thrown.
- **No manual `delete`**: The smart pointer's destructor handles deallocation.
- **Move semantics**: `unique_ptr` is movable but not copyable; `shared_ptr` is both copyable and movable.
- **Reference counting**: `shared_ptr` tracks how many owners exist; the object dies when the count reaches zero.
- **Cycle prevention**: `weak_ptr` breaks circular references between `shared_ptr`s.

### Prerequisites

- Understanding of raw pointers and dynamic allocation (`new`/`delete`).
- Familiarity with RAII and object lifetimes.
- Knowledge of move semantics and rvalue references (for `unique_ptr`).
- Basic awareness of reference counting.

### Related Programming Areas

- **RAII and resource management** – smart pointers are the canonical RAII example.
- **Exception safety** – smart pointers guarantee cleanup during stack unwinding.
- **Move semantics** – `unique_ptr` relies on move construction and move assignment.
- **Concurrency** – `shared_ptr`'s control block is thread-safe for reference count operations (but not for the managed object).
- **Interfaces** – Core Guidelines R.30–R.34 govern smart pointer parameter passing.

### Core Concepts / Features

1. Smart Pointers
2. `std::unique_ptr`
3. `std::shared_ptr`
4. `std::weak_ptr`
5. `std::auto_ptr` (Deprecated)
6. Ownership Semantics
7. Reference Counting
8. Custom Deleters
9. `std::make_unique`
10. `std::make_shared`
11. `std::enable_shared_from_this`
12. Smart Pointer Best Practices
13. Smart Pointer Pitfalls
14. Circular References

---

## 1. Smart Pointers

### Definitions

**Core Definition**
A smart pointer is a class template that wraps a raw pointer and provides automatic lifetime management through RAII, expressing ownership semantics in its type.

**Technical Definition**
Smart pointers are class templates in `<memory>` that behave like pointers (via `operator*` and `operator->`) while owning the pointed-to object. The three standard smart pointers are `std::unique_ptr` (exclusive ownership), `std::shared_ptr` (shared ownership via reference counting), and `std::weak_ptr` (non-owning observer). Smart pointers replace raw owning pointers, which the Core Guidelines describe as error-prone because “a raw pointer (`T*`) does not convey ownership” (R.3).

**Beginner-Friendly Explanation**
A smart pointer is a **wrapper around a raw pointer** that knows how to clean up after itself. It overloads `*` and `->` so you can use it almost like a raw pointer, but when it goes out of scope, it automatically deletes the object it owns. It is the modern C++ way to manage dynamic memory safely.

### Purposes

- To eliminate memory leaks by automating `delete`.
- To make ownership explicit in the type system.
- To provide exception-safe resource management.
- To replace error-prone raw owning pointers.
- To enable safe sharing of dynamically allocated objects.

### Syntax Rules and Structure

**General Syntax**
```cpp
#include <memory>

std::unique_ptr<T> up = std::make_unique<T>(args...);
std::shared_ptr<T> sp = std::make_shared<T>(args...);
std::weak_ptr<T> wp = sp;
```

**Component Breakdown**

| Smart Pointer | Ownership | Copyable? | Movable? |
|---------------|-----------|-----------|----------|
| `unique_ptr<T>` | Exclusive | No | Yes |
| `shared_ptr<T>` | Shared | Yes | Yes |
| `weak_ptr<T>` | None (observer) | Yes | Yes |

**Syntax Rules**
- Include `<memory>`.
- Prefer `std::make_unique` (C++14) and `std::make_shared` (C++11) over raw `new`.
- Smart pointers automatically delete the managed object when they go out of scope.
- `unique_ptr` must be moved to transfer ownership; `shared_ptr` can be copied.
- `weak_ptr` must be converted to `shared_ptr` (via `lock()`) to access the object.

**Constraints and Limitations**
- Smart pointers add a small overhead compared to raw pointers.
- `shared_ptr`'s reference counting has atomic overhead (even in single-threaded code).
- Smart pointers are not a substitute for understanding ownership; misuse can still cause leaks (e.g., circular references).

### Annotated Code Examples

**Example 1: Smart pointer overview**

```cpp
#include <iostream>
#include <memory>

int main() {
    auto up = std::make_unique<int>(10);   // exclusive ownership
    std::cout << *up << '\n';

    auto sp = std::make_shared<int>(20);   // shared ownership
    std::cout << *sp << '\n';

    std::weak_ptr<int> wp = sp;            // non-owning observer
    if (auto locked = wp.lock()) {         // temporary shared_ptr
        std::cout << *locked << '\n';
    }
}
```

**Expected Output**
```
10
20
20
```

**Why this output?**  
`up` owns `10` and prints it; `sp` owns `20`; `wp` observes `sp`'s object and locks it successfully to print `20`.

### Real-World Cases

- **Factory functions** – returning `unique_ptr` or `shared_ptr` from `make_*` functions.
- **Polymorphic containers** – `std::vector<std::unique_ptr<Base>>`.
- **Shared resources** – caches, connection pools, and configuration objects managed by `shared_ptr`.
- **Observer patterns** – `weak_ptr` avoids keeping observers alive.

---

## 2. `std::unique_ptr`

### Definitions

**Core Definition**
`std::unique_ptr` is a smart pointer that retains **sole ownership** of a dynamically allocated object and destroys it when the `unique_ptr` goes out of scope.

**Technical Definition**
`std::unique_ptr<T, Deleter>` is a class template that owns and manages another object through a pointer and disposes of that object when the `unique_ptr` goes out of scope. It is movable but **not copyable** (the copy constructor is deleted), enforcing exclusive ownership. It supports custom deleters (specified as the second template parameter) and has a partial specialization `unique_ptr<T[]>` for arrays. The Core Guidelines recommend `unique_ptr` as the default smart pointer for exclusive ownership (R.21).

**Beginner-Friendly Explanation**
`unique_ptr` is a **“one-owner” pointer**. Only one `unique_ptr` can own a given object at a time. You can **move** ownership from one `unique_ptr` to another, but you cannot copy it — that would create two owners, which is exactly what `unique_ptr` prevents. When the owner dies, the object is automatically deleted.

### Purposes

- To express exclusive ownership of a dynamically allocated object.
- To guarantee deletion when the owner goes out of scope.
- To transfer ownership safely via move semantics.
- To replace raw owning pointers in factories and containers.
- To provide zero-overhead abstraction compared to raw pointers (with the default deleter).

### Syntax Rules and Structure

**General Syntax**
```cpp
std::unique_ptr<T> p = std::make_unique<T>(args...);   // preferred (C++14)
std::unique_ptr<T> q = std::move(p);                   // transfer ownership
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `T` | The type of the managed object. |
| `Deleter` | Optional custom deleter (default: `std::default_delete<T>`). |
| `std::make_unique<T>(args...)` | Creates a `unique_ptr` owning a `T` constructed with `args`. |

**Syntax Rules**
- `unique_ptr` is movable but not copyable.
- Ownership is transferred via `std::move`.
- `operator*` and `operator->` provide access to the managed object.
- `get()` returns the raw pointer without releasing ownership.
- `release()` releases ownership and returns the raw pointer (caller must delete).
- `reset()` destroys the current object and optionally takes ownership of a new one.

**Constraints and Limitations**
- `unique_ptr` cannot be copied; passing by value requires moving (or using a reference/pointer to the `unique_ptr`).
- The default deleter uses `delete`, so it must not be used for memory allocated with `malloc` (use a custom deleter).
- `unique_ptr<T[]>` must be used for arrays allocated with `new[]`.

### Annotated Code Examples

**Example 1: Basic `unique_ptr` usage**

```cpp
#include <iostream>
#include <memory>

struct Widget {
    int value;
    Widget(int v) : value(v) { std::cout << "Widget(" << v << ")\n"; }
    ~Widget() { std::cout << "~Widget(" << value << ")\n"; }
};

int main() {
    auto p = std::make_unique<Widget>(42);   // unique ownership
    std::cout << p->value << '\n';
    // p is destroyed here, and Widget is deleted automatically
}
```

**Expected Output**
```
Widget(42)
42
~Widget(42)
```

**Why this output?**  
`make_unique` constructs the `Widget`; `p->value` accesses it; when `main` ends, `p`'s destructor deletes the `Widget`.

**Example 2: Transferring ownership**

```cpp
#include <iostream>
#include <memory>

void take(std::unique_ptr<int> p) {
    std::cout << *p << '\n';
}   // p is destroyed here

int main() {
    auto p = std::make_unique<int>(99);
    take(std::move(p));   // ownership transferred; p is now null
    // *p would be undefined behavior
}
```

**Expected Output**
```
99
```

**Why this output?**  
`std::move(p)` transfers ownership to the parameter of `take`. After the call, `p` is empty (null).

### Real-World Cases

- **Factory functions** – `std::make_unique<T>(args...)` returns a `unique_ptr`.
- **Polymorphic containers** – `std::vector<std::unique_ptr<Shape>>`.
- **Pimpl idiom** – `class X { std::unique_ptr<Impl> impl; };`.
- **Resource handles** – file handles, sockets, and mutexes.

---

## 3. `std::shared_ptr`

### Definitions

**Core Definition**
`std::shared_ptr` is a smart pointer that retains **shared ownership** of an object through a pointer; multiple `shared_ptr` objects can own the same object, and the object is destroyed when the last owner is destroyed or reset.

**Technical Definition**
`std::shared_ptr<T>` is a class template that manages shared ownership via a **control block** containing a reference count (strong count) and a weak count. The managed object is destroyed when the last `shared_ptr` owning it is destroyed, assigned another pointer, or reset. All member functions (including copy construction and copy assignment) can be called concurrently on different `shared_ptr` instances without additional synchronization, even if they share ownership of the same object. It supports custom deleters (specified at construction, not as a template parameter) and can be created with `std::make_shared`.

**Beginner-Friendly Explanation**
`shared_ptr` is a **“shared-ownership” pointer**. Imagine a shared document with a sign-out sheet: every person who opens the document signs in, and when they're done, they sign out. The document is deleted only when **everyone** has signed out. `shared_ptr` does exactly this with a **reference count**: it counts how many `shared_ptr`s point to the object, and deletes the object when the count reaches zero.

### Purposes

- To share ownership of a dynamically allocated object among multiple owners.
- To automate destruction when the last owner releases the object.
- To pass shared resources across threads safely (for the reference count).
- To implement caches, registries, and shared configuration.
- To provide a safe alternative to raw shared pointers.

### Syntax Rules and Structure

**General Syntax**
```cpp
std::shared_ptr<T> sp = std::make_shared<T>(args...);   // preferred
std::shared_ptr<T> sp2 = sp;                            // shared ownership
sp.reset();                                             // release ownership
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `T` | The type of the managed object. |
| Control block | Stores reference count, weak count, and deleter. |
| `std::make_shared<T>(args...)` | Creates a `shared_ptr` with a single allocation for object + control block. |

**Syntax Rules**
- `shared_ptr` is copyable and movable.
- The managed object is destroyed when the **strong reference count** reaches zero.
- `use_count()` returns the current strong count.
- `reset()` releases ownership and optionally takes a new object.
- Custom deleters are passed as constructor arguments, not template parameters.

**Constraints and Limitations**
- Reference counting has atomic overhead (even single-threaded).
- `shared_ptr` cannot manage arrays directly before C++17 (use `shared_ptr<T[]>` or a custom deleter).
- **Circular references** can cause memory leaks (see §14).
- `make_shared` allocates object and control block together, which can delay memory release if weak references persist.

### Annotated Code Examples

**Example 1: Shared ownership**

```cpp
#include <iostream>
#include <memory>

int main() {
    auto sp1 = std::make_shared<int>(42);
    std::cout << sp1.use_count() << '\n';   // 1

    {
        auto sp2 = sp1;                     // shared ownership
        std::cout << sp1.use_count() << '\n'; // 2
        std::cout << sp2.use_count() << '\n'; // 2
    }   // sp2 destroyed

    std::cout << sp1.use_count() << '\n';   // 1
}   // sp1 destroyed: int is deleted
```

**Expected Output**
```
1
2
2
1
```

**Why this output?**  
The reference count increases when `sp2` is created and decreases when it is destroyed. The object is deleted when `sp1` is destroyed.

### Real-World Cases

- **Shared configuration** – a single configuration object used across components.
- **Caches** – shared entries with automatic eviction when all users are done.
- **Thread pools** – shared task queues and worker contexts.
- **Graph structures** – shared nodes with multiple owners.

---

## 4. `std::weak_ptr`

### Definitions

**Core Definition**
`std::weak_ptr` is a smart pointer that holds a **non-owning (“weak”) reference** to an object managed by `std::shared_ptr`; it does not affect the object's lifetime and must be converted to `shared_ptr` (via `lock()`) to access the object.

**Technical Definition**
`std::weak_ptr<T>` is a smart pointer that tracks an object managed by `shared_ptr` without incrementing its strong reference count. It models **temporary ownership**: the object may be accessed only if it still exists. `weak_ptr` is used to **break circular references** of `shared_ptr` and to observe objects that may be deleted at any time by other owners. To access the object, `lock()` returns a `shared_ptr` that either refers to the object (extending its lifetime) or is empty.

**Beginner-Friendly Explanation**
`weak_ptr` is a **“bystander” pointer**. It can look at an object managed by `shared_ptr`, but it doesn't keep the object alive. If all the `shared_ptr` owners are gone, the object is destroyed, and the `weak_ptr` becomes **expired**. To use the object, you must **lock** the `weak_ptr` to get a temporary `shared_ptr`; if the object is already gone, you get an empty `shared_ptr`.

### Purposes

- To observe objects managed by `shared_ptr` without affecting their lifetime.
- To break circular references that would otherwise cause memory leaks.
- To implement caches with expiration (checking if an object still exists).
- To avoid dangling pointers to `shared_ptr`-managed objects.

### Syntax Rules and Structure

**General Syntax**
```cpp
std::weak_ptr<T> wp = sp;                 // observe sp's object
if (auto locked = wp.lock()) {            // temporary shared_ptr
    // use *locked or locked->
} else {
    // object has been destroyed
}
wp.expired();                             // true if object is gone
```

**Component Breakdown**

| Operation | Description |
|-----------|-------------|
| `wp.lock()` | Returns a `shared_ptr` to the object, or empty if expired. |
| `wp.expired()` | Returns `true` if the object has been destroyed. |
| `wp.use_count()` | Returns the strong count of the observed object. |
| `wp.reset()` | Releases the weak reference. |

**Syntax Rules**
- `weak_ptr` is constructed from a `shared_ptr` (or another `weak_ptr`).
- `weak_ptr` does not increment the strong reference count.
- `lock()` is the only safe way to access the object.
- `weak_ptr` can be copied and moved.

**Constraints and Limitations**
- `weak_ptr` cannot be dereferenced directly (`*wp` and `wp->` are not available).
- The observed object must be managed by `shared_ptr`.
- `weak_ptr` does not prevent the object from being destroyed.

### Annotated Code Examples

**Example 1: Observing with `weak_ptr`**

```cpp
#include <iostream>
#include <memory>

int main() {
    auto sp = std::make_shared<int>(42);
    std::weak_ptr<int> wp = sp;

    if (auto locked = wp.lock()) {
        std::cout << "Alive: " << *locked << '\n';
    }

    sp.reset();   // destroy the object

    if (auto locked = wp.lock()) {
        std::cout << "Still alive\n";
    } else {
        std::cout << "Expired\n";
    }
}
```

**Expected Output**
```
Alive: 42
Expired
```

**Why this output?**  
`lock()` succeeds while `sp` owns the object. After `sp.reset()`, the object is destroyed and `wp` is expired; `lock()` returns an empty `shared_ptr`.

### Real-World Cases

- **Circular references** – breaking `shared_ptr` cycles in graph structures.
- **Caches** – weak references to cache entries that can be evicted.
- **Observer patterns** – observers hold weak references to subjects.
- **Parent-child relationships** – children hold `weak_ptr` to parents to avoid cycles.

---

## 5. `std::auto_ptr` (Deprecated)

### Definitions

**Core Definition**
`std::auto_ptr` is a **deprecated** smart pointer from C++98 that attempted to provide exclusive ownership but had broken copy semantics (copying transferred ownership, leaving the source null), leading to its deprecation in C++11 and removal in C++17.

**Technical Definition**
`std::auto_ptr<T>` was the original standard smart pointer. It had a non-const copy constructor that **transferred ownership** from the source to the destination, leaving the source in a null state. This violated the usual copy semantics and made `auto_ptr` incompatible with standard containers (which require copyable elements). It was deprecated in C++11 (where `std::unique_ptr` was introduced as a safe replacement) and removed from the standard in C++17. Modern code should **never** use `auto_ptr`.

**Beginner-Friendly Explanation**
`auto_ptr` was the **“prototype” smart pointer** — the first attempt at automatic ownership. But it had a fatal flaw: **copying it silently stole the object from the original**. This made it dangerous and incompatible with containers. It has been completely replaced by `unique_ptr`, which uses explicit moves instead of sneaky ownership transfer.

### Purposes

- (Historical) To provide automatic cleanup of dynamically allocated objects.
- (Historical) To express exclusive ownership before move semantics existed.
- (Modern) To serve as a cautionary example of why move semantics were needed.

### Syntax Rules and Structure

**General Syntax (Deprecated)**
```cpp
std::auto_ptr<T> p(new T);   // deprecated: do not use
```

**Component Breakdown**

| Property | Value |
|----------|-------|
| Standard | C++98 (deprecated C++11, removed C++17) |
| Copy semantics | **Transfers ownership** (source becomes null) |
| Replacement | `std::unique_ptr` |

**Syntax Rules (Historical)**
- `auto_ptr` could be copied, but copying transferred ownership.
- Passing `auto_ptr` by value left the caller's pointer null.
- It could not be stored in standard containers.

**Constraints and Limitations**
- **Deprecated**: Do not use in new code.
- **Removed**: Not available in C++17 and later.
- **Broken semantics**: Copy operations mutate the source.

### Annotated Code Examples

**Example 1: `auto_ptr` ownership transfer (deprecated)**

```cpp
#include <memory>
#include <iostream>

int main() {
    std::auto_ptr<int> p1(new int(42));   // deprecated
    std::auto_ptr<int> p2 = p1;           // ownership transferred; p1 becomes null
    std::cout << *p2 << '\n';
    // std::cout << *p1 << '\n';          // undefined behavior: p1 is null
}
```

**Expected Output**
```
42
```

**Why this is problematic?**  
`p2 = p1` **steals** the object from `p1`, leaving `p1` null. This silent mutation is why `auto_ptr` was replaced by `unique_ptr`, which requires `std::move` for ownership transfer.

### Real-World Cases

- **Legacy code migration** – replacing `auto_ptr` with `unique_ptr` in old codebases.
- **Historical education** – understanding why move semantics were introduced.
- **Code review** – flagging any remaining `auto_ptr` usage.

---

## 6. Ownership Semantics

### Definitions

**Core Definition**
Ownership semantics describe **who is responsible for destroying a dynamically allocated object** and are encoded in the type of the smart pointer used.

**Technical Definition**
Ownership is a design concept: the owner of an object is responsible for its lifetime, including its destruction. In C++, ownership is expressed through smart pointers: `unique_ptr` means **exclusive ownership** (one owner), `shared_ptr` means **shared ownership** (multiple owners, object destroyed when the last owner releases it), and `weak_ptr` means **no ownership** (observation only). Raw pointers and references are **non-owning** by convention (R.3, R.4). The Core Guidelines state: “Never transfer ownership by a raw pointer (`T*`) or reference (`T&`)” (I.11).

**Beginner-Friendly Explanation**
Ownership is about **who cleans up**. If you own a toy, you're responsible for putting it away. In C++: `unique_ptr` means “this is **my** toy, I'll put it away.” `shared_ptr` means “we **share** this toy; the last person to leave puts it away.” `weak_ptr` means “I can **look** at the toy, but I'm not responsible for it.” Raw pointers mean “I'm just **borrowing** it; someone else owns it.”

### Purposes

- To make lifetime responsibility explicit in the type system.
- To prevent double deletion and memory leaks.
- To guide API design (who deletes what).
- To enable safe sharing and observation.

### Comparison Table

| Type | Ownership | Responsibility | Core Guideline |
|------|-----------|-----------------|----------------|
| `unique_ptr<T>` | Exclusive | Single owner deletes | R.20, R.21 |
| `shared_ptr<T>` | Shared | Last owner deletes | R.20 |
| `weak_ptr<T>` | None | Observer only | R.20 (breaks cycles) |
| `T*` | Non-owning | Caller/other owner deletes | R.3, I.11 |
| `T&` | Non-owning | Caller/other owner deletes | R.4, I.11 |

**Syntax Rules**
- Use `unique_ptr` for exclusive ownership (default choice).
- Use `shared_ptr` only when ownership is genuinely shared.
- Use `weak_ptr` to observe `shared_ptr`-managed objects without owning.
- Use raw pointers and references only for **non-owning** access.

**Constraints and Limitations**
- Ownership must be clear at every interface boundary.
- Transferring ownership through raw pointers or references is forbidden (I.11).
- Smart pointers should be passed as parameters only when ownership is being transferred or shared (R.30, R.33, R.34).

### Annotated Code Examples

**Example 1: Expressing ownership in function signatures**

```cpp
#include <iostream>
#include <memory>

// Takes ownership (sink)
void consume(std::unique_ptr<int> p) {
    std::cout << "Consumed: " << *p << '\n';
}

// Shares ownership
void share(std::shared_ptr<int> p) {
    std::cout << "Shared count: " << p.use_count() << '\n';
}

// Non-owning observation
void observe(const int* p) {
    if (p) std::cout << "Observed: " << *p << '\n';
}

int main() {
    auto up = std::make_unique<int>(10);
    consume(std::move(up));               // ownership transferred

    auto sp = std::make_shared<int>(20);
    share(sp);                             // shared ownership

    observe(sp.get());                     // non-owning raw pointer
}
```

**Expected Output**
```
Consumed: 10
Shared count: 2
Observed: 20
```

**Why this matters?**  
The signatures clearly document ownership: `consume` takes ownership, `share` shares it, and `observe` merely borrows.

### Real-World Cases

- **API design** – factory functions return `unique_ptr`; registration functions take `shared_ptr`; observers take `const T*` or `const T&`.
- **Core Guidelines** – R.20, R.21, R.30–R.34, I.11 guide ownership decisions.
- **Code review** – ownership ambiguity is a common review finding.

---

## 7. Reference Counting

### Definitions

**Core Definition**
Reference counting is the technique used by `std::shared_ptr` to track how many owners exist for an object; the object is destroyed when the count reaches zero.

**Technical Definition**
Each `shared_ptr` shares a **control block** containing a **strong reference count** (number of `shared_ptr` owners) and a **weak reference count** (number of `weak_ptr` observers). Copying a `shared_ptr` increments the strong count; destroying or resetting it decrements the count. When the strong count reaches zero, the managed object is destroyed (and its memory is deallocated depending on how it was created). The weak count tracks `weak_ptr`s separately; the control block itself is destroyed when both counts reach zero.

**Beginner-Friendly Explanation**
Reference counting is like a **sign-out sheet** for a shared object. Every time someone takes a copy of the `shared_ptr`, they sign in (count goes up). Every time a copy is destroyed, they sign out (count goes down). When the count hits zero, the object is deleted. `weak_ptr`s have their own separate count and don't keep the object alive.

### Purposes

- To determine when a shared object can be safely destroyed.
- To enable automatic cleanup of shared resources.
- To track ownership in multi-owner scenarios.
- To provide the foundation for `weak_ptr` observation.

### Syntax Rules and Structure

**Reference Count Operations**

| Operation | Effect on Strong Count |
|-----------|------------------------|
| `auto sp2 = sp1;` | +1 |
| `sp2 = sp3;` | -1 for sp2's old object, +1 for sp3's |
| `sp.reset();` | -1 |
| `sp.~shared_ptr();` | -1 |
| `sp.use_count()` | Returns current strong count |

**Syntax Rules**
- Copying a `shared_ptr` increments the count.
- Destroying or reassigning a `shared_ptr` decrements the count.
- `use_count()` is approximate in multithreaded contexts (it may be stale).
- The strong count reaching zero triggers object destruction.
- The weak count reaching zero triggers control block destruction.

**Constraints and Limitations**
- Reference counting cannot detect **cycles** (circular references); `weak_ptr` must break cycles.
- Atomic reference counting has performance overhead.
- The managed object is destroyed when the strong count reaches zero, but memory may not be freed immediately if `make_shared` was used and weak references remain.

### Annotated Code Examples

**Example 1: Observing reference counts**

```cpp
#include <iostream>
#include <memory>

int main() {
    auto sp1 = std::make_shared<int>(42);
    std::cout << sp1.use_count() << '\n';   // 1

    auto sp2 = sp1;
    std::cout << sp1.use_count() << '\n';   // 2

    sp2.reset();
    std::cout << sp1.use_count() << '\n';   // 1

    sp1.reset();
    std::cout << "Object destroyed\n";      // count reached 0
}
```

**Expected Output**
```
1
2
1
Object destroyed
```

**Why this output?**  
`use_count()` reflects the number of `shared_ptr` owners. When `sp1.reset()` brings the count to zero, the object is destroyed.

### Real-World Cases

- **Shared caches** – entries are destroyed when no component needs them.
- **Thread-safe sharing** – reference counts are updated atomically.
- **Cycle detection** – reference counting alone cannot detect cycles; `weak_ptr` is needed.

---

## 8. Custom Deleters

### Definitions

**Core Definition**
A custom deleter is a callable object that a smart pointer invokes instead of `delete` when it needs to destroy the managed object, allowing non-`new`/`delete` resources (e.g., `malloc`/`free`, file handles, C API objects) to be managed safely.

**Technical Definition**
Smart pointers support custom deleters to control how the managed object is destroyed. For `std::unique_ptr<T, Deleter>`, the deleter is a **template parameter** and is part of the type; for `std::shared_ptr<T>`, the deleter is passed as a **constructor argument** and is not part of the type (it is stored in the control block). Custom deleters enable smart pointers to manage resources acquired by `malloc`, `fopen`, `dlopen`, or any API that does not use `new`/`delete`.

**Beginner-Friendly Explanation**
A custom deleter tells the smart pointer **how to clean up** when the object isn't a simple `new`-allocated object. For example, if you opened a file with `fopen`, you need to close it with `fclose`, not `delete`. A custom deleter is a function or lambda that the smart pointer calls instead of `delete`.

### Purposes

- To manage resources that are not allocated with `new` (e.g., `malloc`, `fopen`, `socket`).
- To use smart pointers with C APIs.
- To provide specialized cleanup (e.g., logging, pooling).
- To support `unique_ptr` with function pointers or lambdas as deleters.

### Syntax Rules and Structure

**General Syntax**
```cpp
// unique_ptr: deleter is a template parameter
std::unique_ptr<T, Deleter> p(new T, deleter);

// shared_ptr: deleter is a constructor argument
std::shared_ptr<T> p(new T, deleter);
```

**Component Breakdown**

| Smart Pointer | Deleter Location | Part of Type? |
|---------------|------------------|---------------|
| `unique_ptr<T, D>` | Template parameter `D` | Yes |
| `shared_ptr<T>` | Constructor argument | No (stored in control block) |

**Syntax Rules**
- The deleter must be callable with a pointer to `T`.
- `unique_ptr`'s deleter is part of the type, so different deleters produce different types.
- `shared_ptr`'s deleter is type-erased; it does not affect the `shared_ptr` type.
- `make_unique` does not support custom deleters; `make_shared` does (via factory).
- The deleter must not throw exceptions.

**Constraints and Limitations**
- `unique_ptr` with a custom deleter cannot be created with `make_unique` (C++14); use the constructor or a factory function.
- `shared_ptr` with a custom deleter allocates a control block separately (unless `make_shared` is used, which doesn't support custom deleters before C++20).
- The deleter must be copy-constructible for `shared_ptr` (until C++20).

### Annotated Code Examples

**Example 1: `unique_ptr` with a custom deleter (file handle)**

```cpp
#include <cstdio>
#include <memory>
#include <iostream>

struct FileDeleter {
    void operator()(std::FILE* f) const {
        if (f) {
            std::fclose(f);
            std::cout << "File closed\n";
        }
    }
};

int main() {
    std::unique_ptr<std::FILE, FileDeleter> file(
        std::fopen("example.txt", "w"));
    if (file) {
        std::fputs("hello", file.get());
    }
    // File automatically closed when unique_ptr goes out of scope
}
```

**Expected Output**
```
File closed
```

**Why this output?**  
`FileDeleter` calls `fclose` instead of `delete`. When the `unique_ptr` is destroyed, the file is properly closed.

**Example 2: `shared_ptr` with a lambda deleter**

```cpp
#include <memory>
#include <iostream>

int main() {
    auto deleter = [](int* p) {
        std::cout << "Custom delete: " << *p << '\n';
        delete p;
    };
    std::shared_ptr<int> sp(new int(42), deleter);
    // When sp is destroyed, the lambda is called
}
```

**Expected Output**
```
Custom delete: 42
```

**Why this output?**  
The lambda is stored in the control block and invoked when the last `shared_ptr` releases the object.

### Real-World Cases

- **File handles** – `fclose` deleter for `FILE*`.
- **Sockets** – `close` deleter for socket descriptors.
- **C API objects** – custom destructors for `malloc`-allocated structures.
- **Memory pools** – deleters that return memory to a pool.

---

## 9. `std::make_unique`

### Definitions

**Core Definition**
`std::make_unique` (C++14) is a function template that creates a `unique_ptr` owning a newly constructed object, forwarding its arguments to the constructor.

**Technical Definition**
`template<class T, class... Args> unique_ptr<T> make_unique(Args&&... args);` constructs an object of type `T` using `new T(std::forward<Args>(args)...)` and returns a `unique_ptr<T>` owning it. It provides exception safety and eliminates the need to write `new` explicitly. It is the recommended way to create `unique_ptr`s (Core Guideline R.23).

**Beginner-Friendly Explanation**
`make_unique` is a **factory function** that creates a `unique_ptr` for you. Instead of writing `std::unique_ptr<T> p(new T(args))`, you write `auto p = std::make_unique<T>(args)`. It is safer (no chance of leaking if an exception occurs) and shorter.

### Purposes

- To create `unique_ptr`s without writing `new`.
- To avoid resource leaks if an exception occurs during construction.
- To deduce the `unique_ptr` type automatically (with `auto`).
- To provide a consistent factory style with `make_shared`.

### Syntax Rules and Structure

**General Syntax**
```cpp
auto p = std::make_unique<T>(args...);   // C++14
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `T` | The type of object to create. |
| `args...` | Arguments forwarded to `T`'s constructor. |

**Syntax Rules**
- Requires C++14 or later.
- Does not support custom deleters.
- For arrays, `make_unique<T[]>(n)` creates an array (C++14).
- The returned `unique_ptr` owns the newly created object.

**Constraints and Limitations**
- No custom deleter support; use the `unique_ptr` constructor if you need one.
- `make_unique` does not exist for `shared_ptr` (use `make_shared`).
- Cannot create a `unique_ptr` to an existing object (it always allocates).

### Annotated Code Examples

**Example 1: `make_unique` vs. raw `new`**

```cpp
#include <memory>
#include <iostream>

struct Widget {
    int value;
    Widget(int v) : value(v) {}
};

int main() {
    // Preferred
    auto p1 = std::make_unique<Widget>(42);

    // Avoid
    std::unique_ptr<Widget> p2(new Widget(43));

    std::cout << p1->value << ' ' << p2->value << '\n';
}
```

**Expected Output**
```
42 43
```

**Why this is preferred?**  
`make_unique` is exception-safe and avoids explicit `new`. If `Widget`'s constructor throws, no memory is leaked.

### Real-World Cases

- **Factory functions** – returning `make_unique<T>(args...)`.
- **Containers** – `push_back(std::make_unique<T>(...))`.
- **Pimpl** – `impl(std::make_unique<Impl>())`.

---

## 10. `std::make_shared`

### Definitions

**Core Definition**
`std::make_shared` (C++11) is a function template that creates a `shared_ptr` with a single allocation for both the object and the control block, forwarding its arguments to the constructor.

**Technical Definition**
`template<class T, class... Args> shared_ptr<T> make_shared(Args&&... args);` allocates memory for the object and the control block in a **single allocation**, constructs the object with the forwarded arguments, and returns a `shared_ptr` owning it. This is more efficient than `shared_ptr<T>(new T(args))`, which performs two allocations (one for the object, one for the control block). `make_shared` is the recommended way to create `shared_ptr`s (R.22).

**Beginner-Friendly Explanation**
`make_shared` is like `make_unique` but for **shared ownership**. Its special trick is that it packs the object and the reference-counting machinery into **one memory block**, making it faster than creating a `shared_ptr` from a raw `new`.

### Purposes

- To create `shared_ptr`s with a single allocation (better performance).
- To provide exception safety (no leak if construction throws).
- To deduce the `shared_ptr` type automatically.
- To replace `shared_ptr<T>(new T(...))`.

### Syntax Rules and Structure

**General Syntax**
```cpp
auto sp = std::make_shared<T>(args...);   // C++11
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `T` | The type of object to create. |
| `args...` | Arguments forwarded to `T`'s constructor. |
| Single allocation | Object + control block in one block. |

**Syntax Rules**
- Requires C++11 or later.
- Supports custom deleters only via a separate factory (until C++20; C++20 adds allocator support).
- For arrays, `make_shared<T[]>(n)` is available in C++20.
- The returned `shared_ptr` shares ownership via the combined control block.

**Constraints and Limitations**
- Single allocation means the object's memory is not freed until **all weak references** are also gone (the control block cannot be freed separately).
- Custom deleters are not directly supported (until C++20).
- Cannot create a `shared_ptr` to an existing object.

### Annotated Code Examples

**Example 1: `make_shared` efficiency**

```cpp
#include <memory>
#include <iostream>

struct Data {
    int a, b;
    Data(int x, int y) : a(x), b(y) {}
};

int main() {
    auto sp = std::make_shared<Data>(1, 2);   // single allocation
    std::cout << sp->a << ' ' << sp->b << '\n';
}
```

**Expected Output**
```
1 2
```

**Why this is efficient?**  
`make_shared` allocates the `Data` object and the control block together, halving the number of allocations compared to `shared_ptr<Data>(new Data(1, 2))`.

### Real-World Cases

- **Shared configuration** – `auto config = std::make_shared<Config>();`.
- **Caches** – creating shared cache entries.
- **Factory functions** – returning `make_shared<T>(args...)`.

---

## 11. `std::enable_shared_from_this`

### Definitions

**Core Definition**
`std::enable_shared_from_this<T>` is a base class that allows an object already managed by `std::shared_ptr` to safely obtain a `shared_ptr` to itself via the `shared_from_this()` member function.

**Technical Definition**
When a class derives from `std::enable_shared_from_this<T>`, it gains a `shared_from_this()` member that returns a `shared_ptr<T>` sharing ownership with the existing `shared_ptr` that manages the object. This avoids creating a second, independent `shared_ptr` (which would lead to double deletion). The base class stores a `weak_ptr` internally, which is initialized when the object is first managed by a `shared_ptr`. `shared_from_this()` is only valid on objects that are already owned by a `shared_ptr`; calling it on an object not managed by `shared_ptr` throws `std::bad_weak_ptr`.

**Beginner-Friendly Explanation**
Sometimes a class needs to **return a `shared_ptr` to itself** (for example, to register itself as an observer or pass itself to another component). If you just create a new `shared_ptr(this)`, you end up with **two independent owners** — and a double deletion when both try to delete the object. `enable_shared_from_this` solves this by giving the object a safe way to get a `shared_ptr` that **shares ownership** with the existing one.

### Purposes

- To safely obtain a `shared_ptr` to `*this` from within a member function.
- To avoid double deletion from creating a second `shared_ptr` from `this`.
- To share ownership with an existing `shared_ptr`.
- To enable objects to register themselves in shared-ownership systems.

### Syntax Rules and Structure

**General Syntax**
```cpp
class T : public std::enable_shared_from_this<T> {
public:
    std::shared_ptr<T> get_shared() {
        return shared_from_this();   // safe: shares existing ownership
    }
};
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `enable_shared_from_this<T>` | Base class providing `shared_from_this()`. |
| `shared_from_this()` | Returns `shared_ptr<T>` sharing ownership. |
| Internal `weak_ptr` | Initialized when the object is first managed by `shared_ptr`. |

**Syntax Rules**
- Derive publicly from `std::enable_shared_from_this<T>`.
- The object must be managed by a `shared_ptr` before `shared_from_this()` is called.
- `shared_from_this()` can be called in member functions.
- Since C++17, `weak_from_this()` is also available.

**Constraints and Limitations**
- Calling `shared_from_this()` on an object **not** managed by `shared_ptr` throws `std::bad_weak_ptr`.
- It is undefined behavior to call `shared_from_this()` in a constructor or destructor (the weak reference may not be initialized or may be expired).
- The object must be owned by a `shared_ptr` created **after** the base class is initialized.

### Annotated Code Examples

**Example 1: Safe self-registration**

```cpp
#include <iostream>
#include <memory>
#include <vector>

class Observer : public std::enable_shared_from_this<Observer> {
public:
    void register_self(std::vector<std::shared_ptr<Observer>>& registry) {
        registry.push_back(shared_from_this());   // share existing ownership
        std::cout << "Registered\n";
    }
};

int main() {
    auto obs = std::make_shared<Observer>();
    std::vector<std::shared_ptr<Observer>> registry;
    obs->register_self(registry);
    std::cout << "Count: " << obs.use_count() << '\n';   // 2
}
```

**Expected Output**
```
Registered
Count: 2
```

**Why this output?**  
`shared_from_this()` returns a `shared_ptr` that shares ownership with `obs`. The reference count becomes `2`. Without `enable_shared_from_this`, creating `shared_ptr<Observer>(this)` would create a second independent owner, causing a double deletion.

### Real-World Cases

- **Observer patterns** – subjects return `shared_ptr` to themselves when notifying.
- **Component registration** – objects register themselves in a shared registry.
- **Asynchronous callbacks** – capturing a `shared_ptr` to `*this` in a lambda.
- **Parent-child relationships** – children obtain shared ownership of themselves.

---

## 12. Smart Pointer Best Practices

### Definitions

**Core Definition**
Smart pointer best practices are guidelines for choosing, passing, and managing smart pointers correctly, based on the C++ Core Guidelines.

**Technical Definition**
The Core Guidelines provide specific rules for smart pointers: **R.20** (use `unique_ptr` or `shared_ptr` to represent ownership), **R.21** (prefer `unique_ptr` over `shared_ptr` unless you need to share ownership), **R.22** (use `make_shared()` to make `shared_ptr`s), **R.23** (use `make_unique()` to make `unique_ptr`s), **R.30** (take smart pointers as parameters only to explicitly express lifetime semantics), **R.33** (take a `unique_ptr<T>&` parameter to express that a function reseats the widget), **R.34** (take a `shared_ptr<T>` parameter to express shared ownership), and **R.35** (take a `shared_ptr<T>&` parameter to express that a function might reseat the shared pointer).

**Beginner-Friendly Explanation**
Best practices for smart pointers are simple: **use `unique_ptr` by default**, **only use `shared_ptr` when you truly need sharing**, **always use `make_unique` or `make_shared`**, **don’t pass smart pointers unless ownership is involved**, and **never transfer ownership through a raw pointer**. These rules keep your code safe and clear.

### Key Guidelines (Summary Table)

| Guideline | Summary | Recommendation |
|-----------|---------|----------------|
| **R.20** | Use `unique_ptr` or `shared_ptr` to represent ownership. | Raw pointers should not own. |
| **R.21** | Prefer `unique_ptr` over `shared_ptr`. | Use `shared_ptr` only when ownership is genuinely shared. |
| **R.22** | Use `make_shared()` to make `shared_ptr`s. | Single allocation, exception-safe. |
| **R.23** | Use `make_unique()` to make `unique_ptr`s. | C++14; avoids explicit `new`. |
| **R.30** | Take smart pointers as parameters only to express lifetime semantics. | Otherwise use `T*` or `T&`. |
| **R.33** | Take `unique_ptr<T>&` to express reseating. | For functions that replace the owned object. |
| **R.34** | Take `shared_ptr<T>` to express shared ownership. | For functions that share ownership. |
| **R.35** | Take `shared_ptr<T>&` to express possible reseating. | For functions that may reseat the shared pointer. |
| **I.11** | Never transfer ownership by a raw pointer or reference. | Use smart pointers. |
| **ES.21** | Don’t introduce a variable before you need it. | Use `auto` with `make_*`. |

### Annotated Code Examples

**Example 1: R.21 — prefer `unique_ptr`**

```cpp
#include <memory>
#include <iostream>

struct Widget { int id; };

int main() {
    // GOOD: unique_ptr when ownership is not shared
    auto w = std::make_unique<Widget>(Widget{1});
    std::cout << w->id << '\n';

    // AVOID: shared_ptr when unique would suffice
    auto ws = std::make_shared<Widget>(Widget{2});
    std::cout << ws->id << '\n';
}
```

**Expected Output**
```
1
2
```

**Why this is best practice?**  
`unique_ptr` has less overhead (no reference counting) and clearer ownership. Use `shared_ptr` only when sharing is genuinely needed (R.21).

**Example 2: R.30 — don’t pass smart pointers unless ownership is involved**

```cpp
#include <memory>
#include <iostream>

void print_widget(const Widget* w) {   // non-owning: raw pointer
    if (w) std::cout << w->id << '\n';
}

void take_widget(std::unique_ptr<Widget> w) {   // ownership transfer
    std::cout << "Took: " << w->id << '\n';
}

int main() {
    auto w = std::make_unique<Widget>(Widget{3});
    print_widget(w.get());          // observe without ownership
    take_widget(std::move(w));      // transfer ownership
}
```

**Expected Output**
```
3
Took: 3
```

**Why this is best practice?**  
`print_widget` doesn’t need ownership, so it takes a raw pointer. `take_widget` needs ownership, so it takes a `unique_ptr` by value (R.30).

### Real-World Cases

- **Standard library** – `make_unique` and `make_shared` are the primary factory functions.
- **Code review** – Core Guidelines R.20–R.35 are common checkpoints.
- **Static analysis** – clang-tidy enforces these rules via `cppcoreguidelines-owning-memory` and related checks.

---

## 13. Smart Pointer Pitfalls

### Definitions

**Core Definition**
Smart pointer pitfalls are common mistakes and dangerous patterns in smart pointer usage that can lead to memory leaks, double deletion, undefined behavior, or performance degradation.

**Technical Definition**
Pitfalls arise from misunderstanding ownership semantics, reference counting, and the interaction between smart pointers and raw pointers. Common pitfalls include creating multiple `shared_ptr`s from the same raw pointer (double deletion), circular references (memory leaks), passing smart pointers unnecessarily (overhead), returning `shared_ptr` by value from functions that should use `unique_ptr`, and mixing `new` with smart pointers incorrectly.

**Beginner-Friendly Explanation**
Smart pointers are safer than raw pointers, but they are **not foolproof**. If you create two `shared_ptr`s from the same raw pointer, both will try to delete it — **double deletion**! If two `shared_ptr`s point to each other in a cycle, neither can be deleted — **memory leak**! Knowing these pitfalls helps you avoid them.

### Common Pitfalls (Summary)

| Pitfall | Description | Prevention |
|---------|-------------|------------|
| **Double deletion** | Two `shared_ptr`s created from the same raw pointer. | Use `make_shared`; never pass a raw pointer to two `shared_ptr`s. |
| **Circular references** | `shared_ptr` cycles keep objects alive forever. | Use `weak_ptr` to break cycles. |
| **Unnecessary `shared_ptr`** | Using `shared_ptr` when `unique_ptr` suffices. | Prefer `unique_ptr` (R.21). |
| **Passing smart pointers by value** | Copies `shared_ptr` (increments count) unnecessarily. | Pass by `const&` for observation, by value for ownership transfer. |
| **Returning `shared_ptr` from factories** | Overkill if the caller doesn’t need to share. | Return `unique_ptr` by default. |
| **`shared_ptr(this)`** | Creates a second owner from `this`. | Use `enable_shared_from_this`. |
| **Mixing `new` and `make_shared`** | Different allocation strategies. | Prefer `make_shared` consistently. |

### Annotated Code Examples

**Example 1: Double deletion from two `shared_ptr`s**

```cpp
// DO NOT WRITE THIS CODE
#include <memory>

int main() {
    int* raw = new int(42);
    std::shared_ptr<int> sp1(raw);
    std::shared_ptr<int> sp2(raw);   // separate control blocks!
    // sp1 and sp2 will both delete raw → double deletion
}
```

**Expected Output**  
(Undefined behavior: double deletion.)

**Why this is wrong?**  
Each `shared_ptr` constructor creates its **own control block** with its own reference count. Both think they are the sole owner and both delete the same pointer. Use `make_shared` or copy an existing `shared_ptr` to share the control block.

**Example 2: Circular reference memory leak**

```cpp
#include <memory>
#include <iostream>

struct Node {
    std::shared_ptr<Node> next;
    ~Node() { std::cout << "Node destroyed\n"; }
};

int main() {
    auto a = std::make_shared<Node>();
    auto b = std::make_shared<Node>();
    a->next = b;
    b->next = a;   // cycle: neither is destroyed
    // Memory leak: no "Node destroyed" output
}
```

**Expected Output**  
(No output; memory leak.)

**Why this is a leak?**  
`a` and `b` reference each other. Their reference counts never reach zero, so neither is destroyed. Use `weak_ptr` for one direction to break the cycle (see §14).

### Real-World Cases

- **Accidental double deletion** – passing the same raw pointer to two `shared_ptr`s.
- **Memory leaks from cycles** – graph structures, parent-child relationships, observer patterns.
- **Performance regressions** – unnecessary `shared_ptr` copies and atomic reference counting.

---

## 14. Circular References

### Definitions

**Core Definition**
A circular reference occurs when two or more objects hold `shared_ptr`s to each other, creating a cycle that prevents the reference counts from ever reaching zero, resulting in a memory leak.

**Technical Definition**
Reference counting cannot detect cycles. If object `A` owns a `shared_ptr` to `B` and `B` owns a `shared_ptr` to `A`, both objects keep each other's strong reference count at least `1`. Neither is destroyed when external owners release them, because the cycle maintains the count. The solution is to make one direction a **`weak_ptr`** , which does not increment the strong count and allows the object to be destroyed when external owners release it. `std::weak_ptr` is specifically designed to “break circular references of `std::shared_ptr`”.

**Beginner-Friendly Explanation**
A circular reference is like two people holding hands so tightly that neither can leave the room. Even if everyone else leaves, they keep each other from being deleted. To fix it, you make one of them a **weak_ptr** — a “loose handshake” that doesn't keep the other alive. When the strong owners are gone, the objects can be deleted.

### Purposes

- To understand why reference counting alone cannot handle cycles.
- To identify memory leaks caused by circular `shared_ptr` structures.
- To apply `weak_ptr` as the standard solution.
- To design data structures (graphs, trees, observers) without leaks.

### Syntax Rules and Structure

**Problem Pattern**
```cpp
struct Node {
    std::shared_ptr<Node> other;   // cycle!
};
```

**Solution Pattern**
```cpp
struct Node {
    std::weak_ptr<Node> other;     // breaks the cycle
};
```

**Component Breakdown**

| Pointer Type | Effect on Strong Count | Breaks Cycle? |
|--------------|------------------------|---------------|
| `shared_ptr` | Increments | No |
| `weak_ptr` | Does not increment | Yes |

**Syntax Rules**
- Identify one direction in the cycle to be the “weak” direction.
- Replace `shared_ptr` with `weak_ptr` for that direction.
- Use `lock()` to access the object through the `weak_ptr`.
- Check for expiration before accessing.

**Constraints and Limitations**
- `weak_ptr` cannot be dereferenced directly; `lock()` is required.
- The weak direction must be carefully chosen based on ownership semantics (usually the “back-pointer” or “parent-pointer” direction).
- `weak_ptr` adds a small overhead (weak count management).

### Annotated Code Examples

**Example 1: Breaking a cycle with `weak_ptr`**

```cpp
#include <iostream>
#include <memory>

struct Node {
    int value;
    std::shared_ptr<Node> next;
    std::weak_ptr<Node> prev;   // weak back-pointer breaks the cycle
    ~Node() { std::cout << "Node " << value << " destroyed\n"; }
};

int main() {
    auto a = std::make_shared<Node>(Node{1});
    auto b = std::make_shared<Node>(Node{2});
    a->next = b;
    b->prev = a;                // weak reference: does not keep a alive
    // When a and b go out of scope, both are destroyed
}
```

**Expected Output**
```
Node 1 destroyed
Node 2 destroyed
```

**Why this output?**  
The `prev` pointer is `weak_ptr`, so it does not increment `a`'s strong count. When `a` and `b` are destroyed, their counts reach zero and both nodes are destroyed. Without the `weak_ptr`, neither would be destroyed.

### Real-World Cases

- **Doubly-linked lists** – back-pointers should be `weak_ptr`.
- **Parent-child trees** – children hold `weak_ptr` to parents.
- **Observer patterns** – observers hold `weak_ptr` to subjects.
- **Graph structures** – back-edges should be `weak_ptr`.

---

## References

- `std::unique_ptr` (cppreference) – https://en.cppreference.com/w/cpp/memory/unique_ptr
- `std::shared_ptr` (cppreference) – https://en.cppreference.com/w/cpp/memory/shared_ptr
- `std::weak_ptr` (cppreference) – https://en.cppreference.com/w/cpp/memory/weak_ptr
- `std::make_unique` (cppreference) – https://en.cppreference.com/w/cpp/memory/unique_ptr/make_unique
- `std::make_shared` (cppreference) – https://en.cppreference.com/w/cpp/memory/shared_ptr/make_shared
- `std::enable_shared_from_this` (cppreference) – https://en.cppreference.com/w/cpp/memory/enable_shared_from_this
- `std::auto_ptr` (cppreference) – https://en.cppreference.com/w/cpp/memory/auto_ptr
- C++ Core Guidelines — R.20: Use `unique_ptr` or `shared_ptr` to represent ownership – https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rr-owner
- C++ Core Guidelines — R.21: Prefer `unique_ptr` over `shared_ptr` unless you need to share ownership – https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rr-unique
- C++ Core Guidelines — R.22: Use `make_shared()` to make `shared_ptr`s – https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rr-make_shared
- C++ Core Guidelines — R.23: Use `make_unique()` to make `unique_ptr`s – https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rr-make_unique
- C++ Core Guidelines — R.30: Take smart pointers as parameters only to explicitly express lifetime semantics – https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rr-smartptrparam
- C++ Core Guidelines — R.33: Take a `unique_ptr<T>&` parameter to express that a function reseats the widget – https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rr-uniqueptrparam
- C++ Core Guidelines — R.34: Take a `shared_ptr<T>` parameter to express shared ownership – https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rr-sharedptrparam-owner
- C++ Core Guidelines — R.35: Take a `shared_ptr<T>&` parameter to express that a function might reseat the shared pointer – https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rr-sharedptrparam
- C++ Core Guidelines — I.11: Never transfer ownership by a raw pointer (`T*`) or reference (`T&`) – https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Ri-raw
- C++ Core Guidelines — R.3: A raw pointer (`T*`) is non-owning – https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rr-ptr
- C++ Core Guidelines — R.4: A raw reference (`T&`) is non-owning – https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rr-ref
- Microsoft Learn — `auto_ptr` Class (deprecated) – https://learn.microsoft.com/en-us/cpp/standard-library/auto-ptr-class
- AUTOSAR C++14 Rule A18-1-3 — Avoid using `std::auto_ptr` – https://www.mathworks.com/help/bugfinder/ref/autosarcpp14rulea18-1-3.html
- Stack Overflow — Pros and cons of `make_unique` vs direct constructor call – https://stackoverflow.com/questions/79700634/pros-and-cons-of-make-unique-vs-direct-constructor-call-in-c17
- Stack Overflow — Custom deleter for `unique_ptr` vs `shared_ptr` – https://stackoverflow.com/questions/12334241/custom-deleter-for-unique-ptr-and-shared-ptr
- Visual Studio Magazine — Weak Pointers and Circular References in C++11 – https://visualstudiomagazine.com/articles/2012/10/19/circular-references.aspx
- IIT Bombay — Pointer Cycles and `weak_ptr` – https://www.cse.iitb.ac.in/~akg/courses/2024-ds/lec-02-containers.pdf

---

This cheat sheet provides a complete, structured reference for C++ smart pointers, covering `unique_ptr`, `shared_ptr`, `weak_ptr`, deprecated `auto_ptr`, ownership semantics, reference counting, custom deleters, `make_unique`, `make_shared`, `enable_shared_from_this`, best practices, pitfalls, and circular references. Each section follows a uniform template with definitions, purposes, syntax rules, annotated examples, expected outputs, and authoritative references.