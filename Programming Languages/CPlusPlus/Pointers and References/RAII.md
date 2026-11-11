# RAII (Resource Acquisition Is Initialization) — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
RAII (Resource Acquisition Is Initialization) is a C++ programming idiom in which a resource is **acquired in an object's constructor** and **released in its destructor**, tying the resource's lifetime to the lifetime of a scoped object so that cleanup is automatic and exception-safe.

**Technical Definition**
RAII is a C++ technique for managing resources — memory, file handles, sockets, mutexes, database connections — by binding them to the lifetime of an object with **automatic storage duration**. The resource is acquired during construction (the constructor establishes the class invariant), and released during destruction (the destructor tears it down). Because C++ guarantees that destructors run when objects leave scope — including during **stack unwinding** triggered by exceptions — RAII provides **exception safety** and eliminates resource leaks. The idiom is codified in the C++ Core Guidelines: **R.1** (“Manage resources automatically using resource handles and RAII”), **C.30** (“Define a destructor if a class needs an explicit action at object destruction”), **C.31** (“All resources acquired by a class must be released by the class's destructor”), **C.32** (“If a class has a raw pointer (`T*`) or reference (`T&`), consider whether it might be owning”), and **E.6** (“Use RAII to prevent leaks”).

**Beginner-Friendly Explanation**
RAII is the idea that **acquiring a resource is the same act as initializing an object**, and **releasing it is automatic when the object goes away**. Imagine you rent a hotel room: when you check in (construct), you get the key; when you check out (destruct), you give it back — automatically, even if you leave in a hurry. In C++, you create an object that “owns” the resource. When the object goes out of scope — whether normally, or because an exception was thrown — its destructor runs and cleans up. This is why modern C++ rarely needs `finally` blocks or manual cleanup code: the language does it for you.

### Key Characteristics

- **Automatic cleanup**: Destructors run when objects leave scope, regardless of how the scope is exited (normal return, `break`, `goto`, or exception).
- **Exception safety**: Resources are released during stack unwinding, preventing leaks on exceptional paths.
- **Deterministic lifetime**: Unlike garbage collection, destruction happens at a known point (end of scope).
- **Encapsulation**: Resource management logic is encapsulated in a class, not scattered through user code.
- **Composability**: RAII objects can own other RAII objects, forming a hierarchy of resource management.
- **No manual cleanup**: Users of RAII classes do not write `delete`, `close`, or `release` calls.
- **Separation of acquisition and use**: The resource is acquired once and used freely within the object's lifetime.

### Prerequisites

- Understanding of constructors, destructors, and scope.
- Familiarity with object lifetimes and storage duration.
- Knowledge of exceptions and stack unwinding.
- Basic awareness of copy/move semantics (for rule of five).

### Related Programming Areas

- **Smart pointers** – `std::unique_ptr` and `std::shared_ptr` are RAII wrappers for heap memory.
- **Exception safety** – RAII is the foundation of the strong exception guarantee.
- **Locks** – `std::lock_guard`, `std::unique_lock`, and `std::scoped_lock` are RAII mutex wrappers.
- **File I/O** – `std::fstream` releases file handles in its destructor.
- **Containers** – `std::vector`, `std::string` manage their internal storage with RAII.
- **Resource management** – sockets, database connections, GPU resources, and more.

### Core Concepts / Features

1. RAII
2. Resource Acquisition Is Initialization
3. Resource Management
4. Destructors
5. Scope-Based Cleanup
6. Exception Safety
7. RAII Best Practices
8. RAII Examples

---

## 1. RAII

### Definitions

**Core Definition**
RAII is a C++ idiom that binds resource management to object lifetime: acquisition in the constructor, release in the destructor.

**Technical Definition**
RAII is a C++ technique for managing resources by associating them with objects that have automatic storage duration. The constructor acquires the resource and establishes the class invariant; the destructor releases it. Because the C++ language guarantees that destructors are invoked when an object's lifetime ends — including during exception propagation (stack unwinding) — RAII provides deterministic, exception-safe resource management. The Core Guidelines describe RAII as the primary tool for resource safety: “Manage resources automatically using resource handles and RAII” (R.1).

**Beginner-Friendly Explanation**
RAII is a fancy name for a simple idea: **let an object own your resource, and let its destructor clean up**. You don't have to remember to close the file or free the memory — the object does it for you when it goes out of scope. It's like having a personal assistant who handles all the cleanup automatically.

### Purposes

- To automate resource release, eliminating manual cleanup code.
- To guarantee cleanup on exceptional paths (exception safety).
- To prevent resource leaks (memory, files, sockets, locks).
- To make resource ownership explicit through types.
- To simplify code by removing `finally`-style cleanup blocks.
- To enable composable, reusable resource-handling components.

### Syntax Rules and Structure

**General Pattern**
```cpp
class ResourceHandle {
    Resource resource_;   // the owned resource
public:
    ResourceHandle(Args args) : resource_(acquire(args)) {}  // acquire
    ~ResourceHandle() { release(resource_); }                 // release
    ResourceHandle(const ResourceHandle&) = delete;           // usually non-copyable
    ResourceHandle& operator=(const ResourceHandle&) = delete;
};
```

**Component Breakdown**

| Component | Role |
|-----------|------|
| Constructor | Acquires the resource; throws on failure. |
| Destructor | Releases the resource; must not throw. |
| Copy/move control | Often non-copyable; movable for transfer of ownership. |
| Member functions | Provide access to the resource. |

**Syntax Rules**
- The constructor must acquire the resource or throw (no half-constructed objects).
- The destructor must release the resource and must not throw exceptions.
- Copy operations are often deleted (unique ownership) or carefully defined (shared ownership).
- Move operations transfer ownership, leaving the source in a valid empty state.
- The class should satisfy the **Rule of Zero** where possible (no manual resource management) or the **Rule of Five** when it owns a resource directly.

**Constraints and Limitations**
- Destructors must not throw (if they do during unwinding, `std::terminate` is called).
- RAII relies on deterministic destruction, which C++ guarantees for automatic objects.
- Objects with static storage duration are destroyed at program exit; order across translation units is unspecified.

### Annotated Code Examples

**Example 1: Basic RAII file handle**

```cpp
#include <cstdio>
#include <iostream>
#include <stdexcept>

class FileHandle {
    std::FILE* file_;
public:
    explicit FileHandle(const char* path, const char* mode)
        : file_(std::fopen(path, mode)) {
        if (!file_) throw std::runtime_error("failed to open file");
    }
    ~FileHandle() {
        if (file_) std::fclose(file_);   // automatic cleanup
    }
    FileHandle(const FileHandle&) = delete;
    FileHandle& operator=(const FileHandle&) = delete;

    std::FILE* get() const { return file_; }
};

int main() {
    try {
        FileHandle f("example.txt", "w");
        std::fputs("hello", f.get());
    } catch (const std::exception& e) {
        std::cout << e.what() << '\n';
    }
    // File automatically closed when f goes out of scope
}
```

**Expected Output**
```
(No output; file "example.txt" is created and closed automatically.)
```

**Why this works?**  
`FileHandle` acquires the file in its constructor and releases it in its destructor. The user never calls `fclose`; the destructor does it automatically, even if an exception is thrown.

### Real-World Cases

- **Smart pointers** – `std::unique_ptr`, `std::shared_ptr`.
- **Locks** – `std::lock_guard`, `std::unique_lock`, `std::scoped_lock`.
- **File streams** – `std::ifstream`, `std::ofstream`, `std::fstream`.
- **Database connections** – connection pool handles.
- **GPU resources** – texture and buffer wrappers.

---

## 2. Resource Acquisition Is Initialization

### Definitions

**Core Definition**
“Resource Acquisition Is Initialization” is the full name of RAII: acquiring a resource happens **at initialization time** (construction), and releasing it happens **at destruction time**.

**Technical Definition**
The phrase captures the idiom's core mechanism: a resource is acquired during the construction of an object, so that the object's lifetime directly manages the resource's lifetime. The object's constructor establishes the class invariant (the resource is valid); the destructor restores the external state (the resource is released). This ties resource management to the well-defined, compiler-guaranteed lifetime of C++ objects.

**Beginner-Friendly Explanation**
The name says it all: **acquiring a resource is initialization**. When you create the object, you get the resource. When the object dies, the resource goes away. It's a package deal.

### Purposes

- To ensure resources are acquired only when an object is successfully constructed.
- To tie resource lifetime to object lifetime.
- To make ownership explicit in the type system.
- To prevent “half-acquired” resources (constructor throws on failure).
- To simplify resource management by making it automatic.

### Syntax Rules and Structure

**Acquisition in Constructor**
```cpp
class Handle {
    Resource r_;
public:
    Handle() : r_(acquire()) {   // acquisition at initialization
        if (!r_.valid()) throw std::runtime_error("acquire failed");
    }
    ~Handle() { release(r_); }   // release at destruction
};
```

**Component Breakdown**

| Phase | Action |
|-------|--------|
| Initialization | Acquire resource; validate; throw on failure. |
| Use | Access resource through the handle. |
| Destruction | Release resource; no throwing. |

**Syntax Rules**
- Acquisition must happen in the constructor's member initializer list or body.
- If acquisition fails, the constructor must throw (not leave a partially constructed object).
- The destructor must release exactly what the constructor acquired.
- Copy/move must be designed to avoid double release.

**Constraints and Limitations**
- If the constructor throws after partially acquiring resources, those resources must be released (use member RAII objects to handle this automatically).
- The destructor must not throw.
- Ownership must be clear: copying an owning handle usually requires deletion or shared semantics.

### Annotated Code Examples

**Example 1: Acquisition in the initializer list**

```cpp
#include <iostream>
#include <memory>

class Connection {
    std::unique_ptr<int> socket_;   // RAII member
public:
    Connection() : socket_(std::make_unique<int>(42)) {  // acquire
        std::cout << "connected\n";
    }
    ~Connection() {
        std::cout << "disconnected\n";
    }
};

int main() {
    Connection c;   // acquisition at construction
}   // destruction: "disconnected"
```

**Expected Output**
```
connected
disconnected
```

**Why this output?**  
The `Connection` acquires its socket in the constructor and releases it in the destructor.

### Real-World Cases

- **Connection pools** – acquiring a connection at construction and returning it at destruction.
- **Lock guards** – acquiring a mutex in the constructor and unlocking in the destructor.
- **File handles** – opening in the constructor and closing in the destructor.
- **Memory buffers** – allocating in the constructor and freeing in the destructor.

---

## 3. Resource Management

### Definitions

**Core Definition**
Resource management is the discipline of acquiring, using, and releasing finite resources (memory, files, sockets, locks, etc.) correctly and safely.

**Technical Definition**
Resources are entities with limited availability that must be released after use. In C++, resources include memory (`new`/`delete`), file descriptors (`fopen`/`fclose`), sockets, mutexes, database connections, GPU handles, and thread handles. RAII is the primary C++ technique for resource management: each resource is owned by an object whose destructor releases it. The Core Guidelines state: “All resources acquired by a class must be released by the class's destructor” (C.31).

**Beginner-Friendly Explanation**
Resource management is about **not running out of things**. A file handle, a network connection, or a lock is a finite resource — if you forget to release it, it's gone forever. RAII is the C++ way to make sure every resource is released, no matter what happens.

### Purposes

- To prevent resource leaks (memory, files, sockets, locks).
- To ensure resources are released even when exceptions occur.
- To make ownership and lifetime explicit.
- To support composable, reusable resource-handling components.
- To enable deterministic resource release (unlike garbage collection).

### Syntax Rules and Structure

**Resource Categories and RAII Wrappers**

| Resource | Acquisition | Release | RAII Wrapper |
|----------|-------------|---------|--------------|
| Heap memory | `new` | `delete` | `std::unique_ptr`, `std::shared_ptr` |
| File | `fopen` | `fclose` | `std::fstream`, custom `FileHandle` |
| Mutex | `lock` | `unlock` | `std::lock_guard`, `std::unique_lock` |
| Socket | `socket` | `close` | Custom `SocketHandle` |
| Thread | `std::thread` | `join`/`detach` | `std::jthread` (C++20) |

**Syntax Rules**
- Each resource should be owned by exactly one RAII object (or shared via `shared_ptr`).
- The RAII object's constructor acquires; its destructor releases.
- Copy/move semantics must prevent double release.
- The destructor must not throw.

**Constraints and Limitations**
- Some resources require non-trivial release logic (e.g., flushing before closing); the destructor must handle this carefully.
- Shared ownership requires reference counting (`shared_ptr`) and can create cycles.
- Resources with global lifetime may outlive `main`; destruction order is implementation-defined.

### Annotated Code Examples

**Example 1: Mutex lock guard**

```cpp
#include <iostream>
#include <mutex>
#include <thread>

std::mutex mtx;
int counter = 0;

void increment() {
    std::lock_guard<std::mutex> lock(mtx);   // acquire lock
    ++counter;
}   // lock released automatically

int main() {
    std::thread t1(increment);
    std::thread t2(increment);
    t1.join();
    t2.join();
    std::cout << counter << '\n';
}
```

**Expected Output**
```
2
```

**Why this output?**  
`std::lock_guard` acquires the mutex in its constructor and releases it in its destructor. Even if an exception were thrown, the mutex would be released.

### Real-World Cases

- **Thread safety** – `std::lock_guard`, `std::unique_lock`, `std::scoped_lock`.
- **File I/O** – `std::fstream` releases file handles automatically.
- **Memory** – `std::unique_ptr` and `std::shared_ptr`.
- **Networking** – socket wrappers that close on destruction.
- **Database** – connection handles that return connections to the pool.

---

## 4. Destructors

### Definitions

**Core Definition**
A destructor is a special member function that is called automatically when an object's lifetime ends, releasing the object's resources and performing cleanup.

**Technical Definition**
A destructor is declared with `~ClassName()` and has no parameters and no return type. It is invoked when an object goes out of scope (automatic storage), is explicitly deleted (dynamic storage), or is destroyed as part of another object's destruction (members, bases). The destructor should release all resources the object owns and must not throw exceptions (since C++11, destructors are implicitly `noexcept(true)` unless specified otherwise).

**Beginner-Friendly Explanation**
A destructor is the **cleanup function** that runs when an object dies. It's where you put the code that releases resources — closing files, freeing memory, unlocking mutexes. C++ calls it automatically, which is what makes RAII work.

### Purposes

- To release resources owned by the object.
- To perform cleanup actions (flush buffers, close connections).
- To restore external state changed by the constructor.
- To ensure proper destruction of member objects and base classes.
- To support RAII and exception safety.

### Syntax Rules and Structure

**General Syntax**
```cpp
class X {
public:
    ~X();                    // destructor declaration
};

X::~X() {                    // destructor definition
    // release resources
}
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `~X()` | Destructor name (tilde + class name). |
| No parameters | Destructors take no arguments. |
| No return type | Destructors return nothing. |
| `noexcept` | Implicitly `noexcept(true)` unless specified otherwise. |

**Syntax Rules**
- A class can have only one destructor.
- Destructors are called automatically in reverse order of construction.
- For virtual classes, the destructor should be virtual if deletion through a base pointer is possible (C.35).
- The destructor must release all resources acquired by the class (C.31).
- The destructor should not throw exceptions.

**Constraints and Limitations**
- Throwing from a destructor during stack unwinding calls `std::terminate`.
- A non-virtual destructor in a base class with virtual functions can cause undefined behavior when deleting through a base pointer.
- Destructors of static objects run at program exit; order across translation units is unspecified.

### Annotated Code Examples

**Example 1: Destructor releasing a resource**

```cpp
#include <iostream>

class Tracer {
    int id_;
public:
    Tracer(int id) : id_(id) { std::cout << "acquire " << id_ << '\n'; }
    ~Tracer() { std::cout << "release " << id_ << '\n'; }
};

int main() {
    Tracer a(1);
    {
        Tracer b(2);
    }   // b destroyed here
    std::cout << "back in main\n";
}   // a destroyed here
```

**Expected Output**
```
acquire 1
acquire 2
release 2
back in main
release 1
```

**Why this output?**  
Destructors run in reverse order of construction: `b` is destroyed at the end of its block, then `a` at the end of `main`.

**Example 2: Virtual destructor**

```cpp
#include <iostream>

struct Base {
    virtual ~Base() { std::cout << "Base destroyed\n"; }
};

struct Derived : Base {
    ~Derived() override { std::cout << "Derived destroyed\n"; }
};

int main() {
    Base* p = new Derived;
    delete p;   // virtual destructor ensures Derived::~Derived runs
}
```

**Expected Output**
```
Derived destroyed
Base destroyed
```

**Why this output?**  
The virtual destructor ensures the derived destructor runs before the base destructor when deleting through a base pointer.

### Real-World Cases

- **Smart pointers** – `unique_ptr`'s destructor deletes the managed object.
- **Containers** – `std::vector`'s destructor destroys all elements and frees storage.
- **File streams** – `std::fstream`'s destructor closes the file.
- **Locks** – `std::lock_guard`'s destructor unlocks the mutex.

---

## 5. Scope-Based Cleanup

### Definitions

**Core Definition**
Scope-based cleanup is the mechanism by which RAII objects are destroyed automatically when they leave their **scope**, triggering their destructors and releasing resources.

**Technical Definition**
In C++, objects with automatic storage duration are destroyed when control leaves their scope, whether by normal flow (`return`, `break`, `goto`, end of block) or by exception propagation (stack unwinding). This is guaranteed by the language. RAII leverages this guarantee: the resource's lifetime is exactly the object's scope, so cleanup is tied to scope exit.

**Beginner-Friendly Explanation**
Scope-based cleanup means **the resource lives exactly as long as the object is in scope**. When you leave the `{}` block, the object dies, and its destructor cleans up. You don't have to do anything — the language handles it.

### Purposes

- To tie resource lifetime precisely to a lexical scope.
- To guarantee cleanup on all exit paths.
- To simplify code by removing explicit cleanup calls.
- To enable exception-safe resource management.
- To make resource lifetime visually apparent in code structure.

### Syntax Rules and Structure

**General Pattern**
```cpp
void f() {
    ResourceHandle h(...);   // acquire
    // use h
}   // h destroyed here: resource released
```

**Component Breakdown**

| Exit Path | Cleanup Behavior |
|-----------|------------------|
| End of block | Destructors run in reverse order. |
| `return` | Destructors run before returning. |
| `break`/`continue` | Destructors of inner scopes run. |
| `goto` | Destructors of exited scopes run. |
| Exception | Stack unwinding runs destructors. |

**Syntax Rules**
- Automatic objects are destroyed at scope exit, in reverse order of construction.
- Destructors run even if the scope is exited by exception.
- The scope determines the resource's lifetime; nested scopes create nested lifetimes.

**Constraints and Limitations**
- Only automatic storage duration objects are scope-bound; dynamically allocated objects are not (unless wrapped in RAII).
- Static objects have program lifetime, not scope lifetime.
- The order of destruction across translation units is unspecified.

### Annotated Code Examples

**Example 1: Scope-based cleanup with exception**

```cpp
#include <iostream>
#include <stdexcept>

struct Guard {
    ~Guard() { std::cout << "Guard destroyed\n"; }
};

void f() {
    Guard g;
    throw std::runtime_error("oops");   // g destroyed during unwinding
}

int main() {
    try {
        f();
    } catch (const std::exception& e) {
        std::cout << "Caught: " << e.what() << '\n';
    }
}
```

**Expected Output**
```
Guard destroyed
Caught: oops
```

**Why this output?**  
When the exception is thrown, stack unwinding destroys `g`, running its destructor before the exception is caught in `main`.

### Real-World Cases

- **Locks** – released when the lock guard leaves scope.
- **Files** – closed when the file stream leaves scope.
- **Memory** – freed when the smart pointer leaves scope.
- **Transaction guards** – commit or roll back when the guard leaves scope.

---

## 6. Exception Safety

### Definitions

**Core Definition**
Exception safety is the guarantee that a program remains in a valid state and does not leak resources when exceptions are thrown, achieved through RAII and careful design.

**Technical Definition**
The C++ standard defines several levels of exception safety:
- **No-throw (nothrow) guarantee**: The operation never throws; it always succeeds.
- **Strong guarantee**: If an operation throws, the program state is unchanged (commit-or-rollback semantics).
- **Basic guarantee**: If an operation throws, the program remains in a valid state with no resource leaks, but the state may be modified.
- **No guarantee**: If an operation throws, the program may be in an invalid state.

RAII provides the basic guarantee automatically: resources are released during stack unwinding. The strong guarantee requires transactional design (copy-and-swap).

**Beginner-Friendly Explanation**
Exception safety is about **what happens when something goes wrong**. If your program throws an exception, does it leave a mess? RAII ensures that resources are cleaned up automatically, so you never leak memory or leave files open. The strongest form — the “strong guarantee” — means that if an operation fails, it's as if it never happened.

### Purposes

- To prevent resource leaks when exceptions are thrown.
- To maintain valid program state during error handling.
- To provide predictable behavior on failure.
- To enable robust, production-quality code.
- To support the strong guarantee for critical operations.

### Syntax Rules and Structure

**Exception Safety Levels**

| Level | Guarantee | Example |
|-------|-----------|---------|
| Nothrow | Never throws | `noexcept` functions, destructors |
| Strong | Commit-or-rollback | Copy-and-swap assignment |
| Basic | Valid state, no leaks | Most operations with RAII |
| None | No guarantee | Poorly designed code |

**Syntax Rules**
- Destructors must be `noexcept` (implicitly) and must not throw.
- Use RAII to ensure resource cleanup on all paths.
- For the strong guarantee, perform operations on copies and swap on success.
- Document the exception safety guarantee of each function.

**Constraints and Limitations**
- Not all operations can provide the strong guarantee cheaply (e.g., in-place modifications).
- Some operations can only provide the basic guarantee.
- Throwing from a destructor is forbidden (during unwinding, it calls `std::terminate`).

### Annotated Code Examples

**Example 1: RAII provides basic guarantee**

```cpp
#include <iostream>
#include <stdexcept>
#include <memory>

void process() {
    auto data = std::make_unique<int[]>(100);   // RAII
    if (/* something fails */ true) {
        throw std::runtime_error("failed");      // data automatically freed
    }
    // ...
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
Caught: failed
```

**Why this is exception-safe?**  
The `unique_ptr` is destroyed during stack unwinding, freeing the memory even though an exception was thrown.

**Example 2: Strong guarantee with copy-and-swap**

```cpp
#include <iostream>
#include <string>
#include <utility>

class Widget {
    std::string data_;
public:
    Widget(std::string d) : data_(std::move(d)) {}

    // Copy-and-swap: strong guarantee
    Widget& operator=(Widget other) {
        swap(*this, other);
        return *this;
    }

    friend void swap(Widget& a, Widget& b) noexcept {
        using std::swap;
        swap(a.data_, b.data_);
    }

    const std::string& data() const { return data_; }
};

int main() {
    Widget a("hello");
    Widget b("world");
    a = b;   // strong guarantee: either succeeds or a is unchanged
    std::cout << a.data() << '\n';
}
```

**Expected Output**
```
world
```

**Why this is strong?**  
`operator=` takes `other` by value (a copy is made), then swaps. If the copy throws, the original `a` is unchanged; if the swap succeeds, the assignment completes.

### Real-World Cases

- **Standard library** – containers provide the strong guarantee for many operations.
- **Smart pointers** – `unique_ptr` and `shared_ptr` provide basic-guarantee resource management.
- **Locks** – `std::lock_guard` provides nothrow cleanup.
- **Transactional systems** – commit-or-rollback with RAII guards.

---

## 7. RAII Best Practices

### Definitions

**Core Definition**
RAII best practices are guidelines for implementing and using RAII classes correctly, based on the C++ Core Guidelines.

**Technical Definition**
The C++ Core Guidelines provide specific rules for RAII: **R.1** (manage resources automatically using resource handles and RAII), **C.30** (define a destructor if a class needs an explicit action at destruction), **C.31** (all resources acquired by a class must be released by the class's destructor), **C.32** (if a class has a raw pointer or reference, consider whether it might be owning), **C.33** (if a class has an owning pointer member, consider if it is an owner), **C.35** (a base class destructor should be either public and virtual, or protected and non-virtual), **E.6** (use RAII to prevent leaks), and **F.43** (never return a pointer or reference to a local object).

**Beginner-Friendly Explanation**
Best practices for RAII are simple: **wrap every resource in an RAII object**, **write destructors that never throw**, **use the Rule of Zero when possible**, **make base destructors virtual if needed**, and **prefer smart pointers and standard RAII wrappers over writing your own**. These habits make your code safe and clean.

### Key Guidelines (Summary Table)

| Guideline | Summary | Recommendation |
|-----------|---------|----------------|
| **R.1** | Manage resources automatically using RAII. | Wrap resources in RAII objects. |
| **C.30** | Define a destructor if a class needs explicit destruction. | Destructor releases all resources. |
| **C.31** | All resources acquired by a class must be released by its destructor. | No leaks. |
| **C.32** | Consider whether a raw pointer/reference member might be owning. | Prefer smart pointers. |
| **C.33** | Consider whether an owning pointer member is an owner. | Use `unique_ptr`/`shared_ptr`. |
| **C.35** | Base destructor should be public and virtual, or protected and non-virtual. | Avoid undefined behavior on polymorphic deletion. |
| **E.6** | Use RAII to prevent leaks. | Never rely on manual cleanup. |
| **F.43** | Never return a pointer or reference to a local object. | Return by value or use smart pointers. |
| **Rule of Zero** | If a class manages no resources directly, define no special members. | Compose RAII members. |
| **Rule of Five** | If a class manages a resource directly, define all five special members. | Or better, wrap the resource. |
| **No throwing destructors** | Destructors must not throw. | Use `noexcept` (default). |

### Annotated Code Examples

**Example 1: Rule of Zero with RAII members**

```cpp
#include <iostream>
#include <memory>
#include <string>

class Widget {
    std::string name_;              // RAII member
    std::unique_ptr<int> data_;     // RAII member
public:
    Widget(std::string name, int value)
        : name_(std::move(name)), data_(std::make_unique<int>(value)) {}
    // No destructor, copy, or move needed: the Rule of Zero
    const std::string& name() const { return name_; }
    int value() const { return *data_; }
};

int main() {
    Widget w("gadget", 42);
    std::cout << w.name() << ": " << w.value() << '\n';
}
```

**Expected Output**
```
gadget: 42
```

**Why this is best practice?**  
`Widget` composes RAII members (`std::string`, `std::unique_ptr`) and defines no special members. The compiler-generated destructor, copy, and move operations are correct (Rule of Zero). This is safer and simpler than manually managing resources.

**Example 2: Virtual destructor for polymorphic base**

```cpp
#include <iostream>
#include <memory>

struct Base {
    virtual ~Base() = default;   // public virtual destructor
};

struct Derived : Base {
    ~Derived() override { std::cout << "Derived destroyed\n"; }
};

int main() {
    std::unique_ptr<Base> p = std::make_unique<Derived>();
}   // Derived::~Derived runs, then Base::~Base
```

**Expected Output**
```
Derived destroyed
```

**Why this is best practice?**  
The virtual destructor ensures correct destruction when deleting through a base pointer (C.35). Using `unique_ptr` provides RAII cleanup.

### Real-World Cases

- **Standard library** – `std::vector`, `std::string`, `std::unique_ptr`, `std::lock_guard` all follow these rules.
- **Code review** – Core Guidelines R.1, C.30–C.35, E.6, F.43 are common checkpoints.
- **Static analysis** – clang-tidy checks for non-virtual destructors and resource leaks.

---

## 8. RAII Examples

### Definitions

**Core Definition**
RAII examples demonstrate the idiom applied to common resources, showing how constructors acquire and destructors release, and how the resulting classes provide automatic, exception-safe cleanup.

**Technical Definition**
RAII examples illustrate the pattern across resource categories: memory (`unique_ptr`, `shared_ptr`), files (`fstream`, custom `FileHandle`), locks (`lock_guard`, `unique_lock`), sockets, database connections, and custom resources. Each example follows the same structure: acquire in the constructor, release in the destructor, delete copy operations (or implement shared ownership), and ensure the destructor does not throw.

**Beginner-Friendly Explanation**
RAII examples show you **what RAII looks like in practice**. They are templates you can adapt: file handles, locks, memory, and custom resources. Seeing the pattern in action makes it easier to write your own RAII classes.

### Common RAII Examples

| Resource | RAII Wrapper | Acquire | Release |
|----------|-------------|---------|---------|
| Heap memory | `std::unique_ptr` | `new` / `make_unique` | `delete` |
| Shared memory | `std::shared_ptr` | `new` / `make_shared` | `delete` (last owner) |
| File | `std::fstream` | `open` | `close` |
| Mutex | `std::lock_guard` | `lock` | `unlock` |
| Mutex (deferrable) | `std::unique_lock` | `lock` / `try_lock` | `unlock` |
| Multiple mutexes | `std::scoped_lock` | `lock` (all) | `unlock` (all) |
| Thread | `std::jthread` (C++20) | `std::thread` | `join` |
| Custom handle | User-defined class | Custom acquire | Custom release |

### Annotated Code Examples

**Example 1: Custom socket handle**

```cpp
#include <iostream>
#include <stdexcept>

// Simplified socket handle (platform-specific details omitted)
class SocketHandle {
    int fd_;
public:
    explicit SocketHandle(int fd) : fd_(fd) {
        if (fd_ < 0) throw std::runtime_error("invalid socket");
        std::cout << "socket " << fd_ << " acquired\n";
    }
    ~SocketHandle() {
        std::cout << "socket " << fd_ << " released\n";
    }
    SocketHandle(const SocketHandle&) = delete;
    SocketHandle& operator=(const SocketHandle&) = delete;
    SocketHandle(SocketHandle&& other) noexcept : fd_(other.fd_) {
        other.fd_ = -1;
    }
    int get() const { return fd_; }
};

int main() {
    SocketHandle s(3);
    // Use s.get() ...
}   // socket released automatically
```

**Expected Output**
```
socket 3 acquired
socket 3 released
```

**Why this works?**  
`SocketHandle` acquires the socket in its constructor and releases it in its destructor. Move semantics transfer ownership safely.

**Example 2: Scoped transaction guard**

```cpp
#include <iostream>
#include <stdexcept>

class Transaction {
    bool committed_ = false;
public:
    Transaction() { std::cout << "BEGIN\n"; }
    ~Transaction() {
        if (!committed_) std::cout << "ROLLBACK\n";
    }
    void commit() { committed_ = true; std::cout << "COMMIT\n"; }
};

void do_work(bool fail) {
    Transaction tx;
    // ... work ...
    if (fail) throw std::runtime_error("failure");
    tx.commit();
}

int main() {
    try {
        do_work(true);
    } catch (const std::exception& e) {
        std::cout << "Caught: " << e.what() << '\n';
    }
}
```

**Expected Output**
```
BEGIN
ROLLBACK
Caught: failure
```

**Why this works?**  
The `Transaction` guard prints `BEGIN` at construction and `ROLLBACK` at destruction if `commit()` was not called. If an exception is thrown, the guard rolls back automatically.

### Real-World Cases

- **Database transactions** – commit-or-rollback guards.
- **Network connections** – socket handles that close on destruction.
- **File handles** – custom wrappers for `fopen`/`fclose`.
- **GPU resources** – texture and buffer wrappers.
- **Logging scopes** – timers that log elapsed time on destruction.

---

## References

- C++ Core Guidelines — R.1: Manage resources automatically using resource handles and RAII – https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rr-raii
- C++ Core Guidelines — C.30: Define a destructor if a class needs an explicit action at object destruction – https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rc-dtor
- C++ Core Guidelines — C.31: All resources acquired by a class must be released by the class's destructor – https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rc-dtor-release
- C++ Core Guidelines — C.32: If a class has a raw pointer (`T*`) or reference (`T&`), consider whether it might be owning – https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rc-ptr
- C++ Core Guidelines — C.33: If a class has an owning pointer member, consider if it is an owner – https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rc-owning
- C++ Core Guidelines — C.35: A base class destructor should be either public and virtual, or protected and non-virtual – https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rc-dtor-virtual
- C++ Core Guidelines — E.6: Use RAII to prevent leaks – https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Re-raii
- C++ Core Guidelines — F.43: Never (directly or indirectly) return a pointer or a reference to a local object – https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rf-dangle
- Destructor (cppreference) – https://en.cppreference.com/w/cpp/language/destructor
- RAII (cppreference) – https://en.cppreference.com/w/cpp/language/raii
- Microsoft Learn — Object lifetime and resource management (RAII) – https://learn.microsoft.com/en-us/cpp/cpp/object-lifetime-and-resource-management-modern-cpp
- Microsoft Learn — Modern C++: RAII – https://learn.microsoft.com/en-us/cpp/cpp/raii
- isocpp.org FAQ — What is RAII? – https://isocpp.org/wiki/faq/dtors
- isocpp.org FAQ — What is exception safety? – https://isocpp.org/wiki/faq/exceptions
- cppreference — `std::lock_guard` – https://en.cppreference.com/w/cpp/thread/lock_guard
- cppreference — `std::unique_ptr` – https://en.cppreference.com/w/cpp/memory/unique_ptr

---

This cheat sheet provides a complete, structured reference for C++ RAII, covering the idiom, resource acquisition, resource management, destructors, scope-based cleanup, exception safety, best practices, and practical examples. Each section follows a uniform template with definitions, purposes, syntax rules, annotated examples, expected outputs, and authoritative references.