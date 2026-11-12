# C++ RAII — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
RAII (Resource Acquisition Is Initialization) is a C++ programming idiom in which resource management is tied to object lifetime: resources are acquired in a constructor and released in a destructor, guaranteeing cleanup whenever the object goes out of scope.

**Technical Definition**
RAII is the fundamental C++ technique for managing resources such as memory, file handles, sockets, locks, and network connections. A RAII wrapper is a class that acquires a resource in its constructor and releases it in its destructor. Because destructors are automatically invoked when objects go out of scope—including during exception-driven stack unwinding—RAII provides deterministic, leak-free resource management. The C++ Core Guidelines describe RAII as the primary mechanism for avoiding resource leaks and recommend it over manual `try`/`catch` cleanup (R.1, E.6).

**Beginner-Friendly Explanation**
RAII means “let an object own the resource.” When the object is created, it grabs the resource; when the object is destroyed, it lets the resource go. Since C++ guarantees that objects are destroyed when they go out of scope—even when exceptions are thrown—you never have to remember to clean up manually. It is like having a self-returning library book: you borrow it by creating the object, and it returns itself when you are done.

### Key Characteristics

- **Deterministic cleanup** – destructors run at well-defined points (scope exit, stack unwinding).
- **Exception safety** – resources are released during exception propagation without explicit `try`/`catch`.
- **Encapsulation** – the resource and its lifetime are encapsulated in a single object.
- **Composability** – RAII wrappers can be nested and combined; destruction happens in reverse order of construction.
- **Zero-overhead** – RAII wrappers are typically as efficient as manual resource management.
- **Ownership semantics** – the RAII object owns the resource; copying and moving must be carefully controlled (often made non-copyable or move-only).

### Prerequisites

- Basic C++ syntax (classes, constructors, destructors).
- Understanding of stack unwinding and exception handling.
- Familiarity with move semantics and copy control.
- Knowledge of the standard library’s RAII types (smart pointers, `std::fstream`, `std::lock_guard`).

### Related Programming Areas

- **Exception safety** – RAII is the foundation of the basic, strong, and nothrow guarantees.
- **Smart pointers** – `unique_ptr` and `shared_ptr` are RAII wrappers for dynamic memory.
- **Standard library** – `std::fstream`, `std::lock_guard`, `std::unique_lock`, and `std::jthread` are RAII types.
- **Resource management** – files, sockets, mutexes, database connections, GPU buffers.
- **Scope-based cleanup** – RAII is the C++ embodiment of scope-based cleanup.

### Core Concepts / Features

1. RAII (the idiom)
2. Resource Acquisition Is Initialization (the principle)
3. Resource management
4. Destructors
5. Scope-based cleanup
6. Exception safety
7. RAII wrappers
8. RAII examples
9. RAII best practices

---

## 1. RAII (The Idiom)

### Definitions

**Core Definition**
RAII is a C++ programming idiom that ties resource lifetime to object lifetime, ensuring that resources are acquired during object construction and released during object destruction.

**Technical Definition**
RAII is a C++ programming technique that binds the lifecycle of a resource—memory, file handle, socket, mutex, etc.—to the lifetime of an object. The resource is acquired in the constructor and released in the destructor. Because the C++ language guarantees that destructors are called when objects go out of scope (including during stack unwinding caused by exceptions), RAII provides deterministic and exception-safe resource management. The C++ Core Guidelines describe RAII as the primary mechanism for avoiding resource leaks (R.1: “Manage resources automatically using resource handles and RAII”).

**Beginner-Friendly Explanation**
RAII is the C++ way of saying “ownership equals responsibility.” If an object owns a resource, it is responsible for releasing it—and C++ guarantees that the object's destructor runs when it goes out of scope, so the resource is always released.

### Purposes

- To guarantee resource release regardless of how a scope is exited (normal return, `break`, `continue`, `goto`, or exception).
- To eliminate manual cleanup code and the bugs that come with it.
- To provide exception safety without explicit `try`/`catch`.
- To encapsulate resource management in reusable, composable types.
- To simplify code by removing cleanup boilerplate.

### Syntax Rules and Structure

**General Pattern**
```cpp
class RaiiWrapper {
    Resource resource_;                // acquired in constructor
public:
    RaiiWrapper() : resource_(acquire()) {}
    ~RaiiWrapper() { release(resource_); }

    // Copy/move control: often non-copyable, possibly movable
    RaiiWrapper(const RaiiWrapper&) = delete;
    RaiiWrapper& operator=(const RaiiWrapper&) = delete;
    RaiiWrapper(RaiiWrapper&&) noexcept = default;
    RaiiWrapper& operator=(RaiiWrapper&&) noexcept = default;
};
```

**Component Breakdown**

| Component | Role |
|-----------|------|
| Constructor | Acquires the resource. |
| Destructor | Releases the resource (must not throw). |
| Copy/move control | Prevents double release or enables transfer. |
| Member data | Holds the resource handle. |

**Syntax Rules**
- The destructor must not throw (it is implicitly `noexcept` since C++11).
- The wrapper should either be non-copyable or implement proper copy/move semantics.
- The resource handle should be encapsulated; avoid exposing raw handles unless necessary.

**Constraints and Limitations**
- RAII requires resources to be wrapped in objects; some boilerplate is unavoidable for custom resources.
- Destructors cannot report errors by throwing; error reporting must happen elsewhere.

### Annotated Code Example

```cpp
#include <iostream>

class FileHandle {
    std::FILE* file_;
public:
    explicit FileHandle(const char* path)
        : file_(std::fopen(path, "r")) {
        if (!file_) throw std::runtime_error("open failed");
    }
    ~FileHandle() {
        if (file_) std::fclose(file_);
    }
    FileHandle(const FileHandle&) = delete;
    FileHandle& operator=(const FileHandle&) = delete;

    std::FILE* get() const { return file_; }
};

int main() {
    try {
        FileHandle f("data.txt");
        // use f.get()
    } catch (const std::exception& e) {
        std::cout << "Error: " << e.what() << '\n';
    }
}
```

**Expected Output**  
(File opened and closed automatically; error handling depends on whether `data.txt` exists.)

**Why this output?**  
`FileHandle` acquires the file in its constructor and closes it in its destructor. The file is closed automatically when `f` goes out of scope or when an exception unwinds the stack.

### Real-World Cases

- **File management** – `std::fstream` closes files automatically.
- **Mutex locking** – `std::lock_guard` releases mutexes on scope exit.
- **Memory management** – `unique_ptr` and `shared_ptr` release memory.
- **Network connections** – RAII wrappers for sockets and connections.

---

## 2. Resource Acquisition Is Initialization (The Principle)

### Definitions

**Core Definition**
“Resource Acquisition Is Initialization” is the principle that a resource should be acquired during object initialization (construction) so that its lifetime is bound to the object's lifetime.

**Technical Definition**
The RAII principle states that resource acquisition should be the first thing an object does—during construction—so that the resource is available for the entire lifetime of the object and is released when the object is destroyed. Conversely, resource release should be the last thing an object does—during destruction. This ensures that the resource is never in a half-acquired or half-released state, and that it is always released, even if an exception is thrown.

**Beginner-Friendly Explanation**
The principle is simple: “Get the resource when you're born, give it back when you die.” In C++, “born” means the constructor and “die” means the destructor.

### Purposes

- To bind resource lifetime to object lifetime.
- To guarantee resource release on all exit paths.
- To avoid half-initialized or half-destroyed states.
- To provide a clear ownership model.

### Syntax Rules and Structure

**Acquisition in Constructor**
```cpp
class Lock {
    std::mutex& m_;
public:
    explicit Lock(std::mutex& m) : m_(m) { m_.lock(); }   // acquire
    ~Lock() { m_.unlock(); }                               // release
};
```

**Component Breakdown**

| Phase | Action |
|-------|--------|
| Construction | Acquire the resource. |
| Object lifetime | Use the resource. |
| Destruction | Release the resource. |

**Syntax Rules**
- Acquisition must be the first action of the constructor (ideally in the member initializer list).
- If construction fails (throws), the resource must not be leaked.
- Release must be the last action of the destructor.

**Constraints and Limitations**
- If the constructor throws after acquiring some resources, those resources must be released (use member RAII objects to handle this).

### Annotated Code Example

```cpp
#include <mutex>
#include <iostream>

class Lock {
    std::mutex& m_;
public:
    explicit Lock(std::mutex& m) : m_(m) {
        m_.lock();
        std::cout << "Locked\n";
    }
    ~Lock() {
        std::cout << "Unlocked\n";
        m_.unlock();
    }
    Lock(const Lock&) = delete;
    Lock& operator=(const Lock&) = delete;
};

int main() {
    std::mutex m;
    {
        Lock lock(m);
        // critical section
    } // unlocked automatically
}
```

**Expected Output**
```
Locked
Unlocked
```

**Why this output?**  
The `Lock` constructor acquires the mutex; the destructor releases it when `lock` goes out of scope.

### Real-World Cases

- **Mutex locks** – `std::lock_guard`, `std::unique_lock`.
- **File handles** – `std::fstream`.
- **Memory** – `std::unique_ptr`, `std::shared_ptr`.

---

## 3. Resource Management

### Definitions

**Core Definition**
Resource management is the practice of acquiring, using, and releasing resources—memory, file handles, sockets, locks, and other finite system entities—in a safe and deterministic manner.

**Technical Definition**
Resource management in C++ is the discipline of ensuring that every acquired resource is released exactly once, at the right time, regardless of how control flow leaves the scope. RAII is the primary C++ technique for resource management, but it is complemented by smart pointers, containers, and standard library facilities. The C++ Core Guidelines define a *resource* as “something that must be acquired and (explicitly or implicitly) released” (R.1).

**Beginner-Friendly Explanation**
Resource management is about taking care of things you borrow: memory, files, locks, connections. You must give them back when you're done, and you must give them back even if something goes wrong.

### Purposes

- To prevent resource leaks.
- To ensure deterministic release.
- To provide exception safety.
- To simplify code and reduce bugs.

### Syntax Rules and Structure

**Resource Types and Their RAII Wrappers**

| Resource | Manual Management | RAII Wrapper |
|----------|-------------------|--------------|
| Heap memory | `new`/`delete` | `std::unique_ptr`, `std::shared_ptr` |
| File | `fopen`/`fclose` | `std::fstream`, custom `FileHandle` |
| Mutex | `lock`/`unlock` | `std::lock_guard`, `std::unique_lock` |
| Thread | `pthread_create`/`pthread_join` | `std::jthread` |
| Socket | `socket`/`close` | Custom `SocketHandle` |

**Syntax Rules**
- Prefer standard RAII wrappers where available.
- Write custom RAII wrappers for non-standard resources.
- Avoid raw owning pointers.

**Constraints and Limitations**
- Some resources (e.g., C library handles) require custom wrappers.
- Destructors cannot report errors.

### Annotated Code Example

```cpp
#include <fstream>
#include <iostream>

int main() {
    std::ifstream file("data.txt");
    if (!file) {
        std::cout << "Failed to open\n";
        return 1;
    }
    std::string line;
    while (std::getline(file, line))
        std::cout << line << '\n';
} // file closed automatically
```

**Expected Output**  
(Contents of `data.txt`, or “Failed to open”.)

**Why this output?**  
`std::ifstream` is an RAII wrapper: it opens the file in its constructor and closes it in its destructor.

### Real-World Cases

- **Logging** – RAII file handles for log files.
- **Database connections** – RAII wrappers for connection pools.
- **Graphics** – RAII wrappers for GPU buffers and textures.

---

## 4. Destructors

### Definitions

**Core Definition**
A destructor is a special member function that is called when an object's lifetime ends, releasing any resources the object owns.

**Technical Definition**
A destructor is a member function with the name `~ClassName()` that takes no parameters and has no return type. It is called automatically when an object goes out of scope (for automatic objects), when `delete` is called (for dynamic objects), or when a temporary object's lifetime ends. Since C++11, destructors are implicitly `noexcept` unless specified otherwise; throwing from a destructor during stack unwinding calls `std::terminate`.

**Beginner-Friendly Explanation**
A destructor is the “cleanup function” that runs when an object dies. It is where RAII releases the resource.

### Purposes

- To release resources owned by the object.
- To perform cleanup on scope exit.
- To ensure resource release during stack unwinding.
- To maintain invariants during object destruction.

### Syntax Rules and Structure

**General Syntax**
```cpp
class MyClass {
public:
    ~MyClass();                        // declaration
};

MyClass::~MyClass() {                  // definition
    // release resources
}
```

**Component Breakdown**

| Aspect | Requirement |
|--------|-------------|
| Name | `~ClassName` |
| Parameters | None |
| Return type | None |
| Exception specification | Implicitly `noexcept` (C++11+) |

**Syntax Rules**
- Destructors cannot be overloaded.
- A class can have at most one destructor.
- Virtual destructors are required for polymorphic base classes.
- Destructors are implicitly `noexcept` unless a `noexcept(false)` specification is provided.

**Constraints and Limitations**
- Throwing from a destructor during stack unwinding calls `std::terminate`.
- Destructors must not throw in general; error reporting must happen elsewhere.

### Annotated Code Example

```cpp
#include <iostream>

class Widget {
public:
    Widget() { std::cout << "Constructed\n"; }
    ~Widget() { std::cout << "Destroyed\n"; }
};

int main() {
    Widget w;
} // destructor called here
```

**Expected Output**
```
Constructed
Destroyed
```

**Why this output?**  
The destructor is called automatically when `w` goes out of scope.

### Real-World Cases

- **RAII wrappers** – destructors release resources.
- **Polymorphic base classes** – virtual destructors ensure correct cleanup.
- **Container cleanup** – destructors of elements run when containers are destroyed.

---

## 5. Scope-Based Cleanup

### Definitions

**Core Definition**
Scope-based cleanup is the technique of tying resource release to the exit of a scope, so that resources are automatically cleaned up when control leaves the scope.

**Technical Definition**
Scope-based cleanup leverages C++'s guarantee that automatic objects are destroyed when their scope ends—whether by normal control flow (`return`, `break`, `continue`, `goto`) or by exception propagation (stack unwinding). RAII wrappers are the primary mechanism for scope-based cleanup. The C++ Core Guidelines recommend using scope-based cleanup instead of explicit cleanup code (E.6, R.1).

**Beginner-Friendly Explanation**
Scope-based cleanup means “when you leave this block, the cleanup happens automatically.” You don't need to write `try`/`catch` or `finally`—the destructor of the RAII object does the cleanup for you.

### Purposes

- To eliminate manual cleanup code.
- To ensure cleanup on all exit paths.
- To provide exception safety without explicit `try`/`catch`.
- To simplify code and reduce bugs.

### Syntax Rules and Structure

**General Pattern**
```cpp
{
    ResourceGuard guard;               // acquires resource
    // use resource
} // guard destroyed here; resource released
```

**Component Breakdown**

| Scope Exit Path | Cleanup Mechanism |
|-----------------|-------------------|
| Normal flow | Destructor called at scope exit. |
| `return` | Destructor called during return. |
| `break`/`continue` | Destructor called at loop exit. |
| `goto` | Destructor called when scope is exited. |
| Exception | Destructor called during stack unwinding. |

**Syntax Rules**
- RAII objects must be declared in the scope they guard.
- The destructor must not throw.
- The resource must be fully acquired in the constructor (or the object must handle partial acquisition).

**Constraints and Limitations**
- RAII objects declared before a scope exit are destroyed in reverse order of construction.
- Cleanup cannot be conditionally skipped without additional mechanisms (e.g., `release()` methods on smart pointers).

### Annotated Code Example

```cpp
#include <memory>
#include <iostream>
#include <stdexcept>

void process() {
    auto resource = std::make_unique<int>(42);   // RAII
    if (*resource == 42)
        throw std::runtime_error("failure");     // unwinds, resource freed
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
Caught: failure
```

**Why this output?**  
The `unique_ptr` releases the memory during stack unwinding; the exception is caught in `main`.

### Real-World Cases

- **Mutex locking** – `std::lock_guard` releases the mutex on scope exit.
- **File handling** – `std::fstream` closes the file on scope exit.
- **Transaction guards** – rollback on scope exit unless committed.

---

## 6. Exception Safety

### Definitions

**Core Definition**
Exception safety is the property of code that, when an exception is thrown, it avoids resource leaks, preserves class invariants, and leaves objects in a valid state.

**Technical Definition**
Exception safety is described by three guarantees: the **basic guarantee** (no leaks, invariants preserved), the **strong guarantee** (commit-or-rollback semantics), and the **nothrow guarantee** (the operation never throws). RAII is the foundation of all three guarantees: by tying resource release to object destruction, RAII ensures that resources are released during stack unwinding, satisfying the basic guarantee at minimum.

**Beginner-Friendly Explanation**
Exception safety means your program doesn't leak or break when an exception is thrown. RAII is the main tool for achieving this: resources are released automatically during stack unwinding, so you don't need to write cleanup code in every `catch` block.

### Purposes

- To prevent resource leaks when exceptions are thrown.
- To preserve object invariants.
- To provide predictable behaviour after exceptions.
- To enable robust error recovery.

### Syntax Rules and Structure

**Exception Safety Levels**

| Guarantee | Meaning |
|-----------|---------|
| Basic | No leaks; invariants preserved. |
| Strong | Commit-or-rollback; state unchanged on failure. |
| Nothrow | Never throws. |

**RAII's Role**
- RAII provides the basic guarantee by ensuring cleanup during unwinding.
- RAII enables the strong guarantee via copy-and-swap (using `noexcept` swap).
- RAII wrappers (smart pointers, locks) are often nothrow on destruction.

**Constraints and Limitations**
- RAII alone does not guarantee the strong guarantee; additional techniques (copy-and-swap) are needed.

### Annotated Code Example

```cpp
#include <memory>
#include <iostream>
#include <stdexcept>

void basic_guarantee() {
    auto p = std::make_unique<int>(42);
    throw std::runtime_error("failure");   // p released during unwinding
}

int main() {
    try {
        basic_guarantee();
    } catch (const std::exception& e) {
        std::cout << "Caught: " << e.what() << '\n';   // no leak
    }
}
```

**Expected Output**
```
Caught: failure
```

**Why this output?**  
The `unique_ptr` releases the memory during stack unwinding, satisfying the basic guarantee; the exception is caught in `main`.

### Real-World Cases

- **RAII wrappers** – smart pointers, locks, and file handles provide basic exception safety.
- **Copy-and-swap** – strong exception safety for assignment.
- **`noexcept` operations** – nothrow guarantee for critical operations.

---

## 7. RAII Wrappers

### Definitions

**Core Definition**
An RAII wrapper is a class that encapsulates a resource, acquiring it in the constructor and releasing it in the destructor.

**Technical Definition**
An RAII wrapper is a user-defined or standard library type that follows the RAII idiom. Standard library RAII wrappers include `std::unique_ptr`, `std::shared_ptr`, `std::fstream`, `std::lock_guard`, `std::unique_lock`, and `std::jthread`. Custom RAII wrappers are written for resources not covered by the standard library (e.g., C library handles, sockets, database connections).

**Beginner-Friendly Explanation**
An RAII wrapper is a “resource-owning object.” It takes care of the resource so you don't have to. Standard examples include smart pointers (for memory) and lock guards (for mutexes).

### Purposes

- To encapsulate resource management.
- To provide automatic cleanup on scope exit.
- To offer a safe, reusable interface for a resource.
- To simplify code and reduce bugs.

### Syntax Rules and Structure

**General Pattern**
```cpp
class ResourceWrapper {
    Resource resource_;
public:
    ResourceWrapper() : resource_(acquire()) {}
    ~ResourceWrapper() { release(resource_); }

    // Prevent copying (or implement move semantics)
    ResourceWrapper(const ResourceWrapper&) = delete;
    ResourceWrapper& operator=(const ResourceWrapper&) = delete;
};
```

**Component Breakdown**

| Component | Role |
|-----------|------|
| Constructor | Acquires resource. |
| Destructor | Releases resource. |
| Copy/move control | Prevents double release or enables transfer. |
| Accessors | Provide safe access to the resource. |

**Syntax Rules**
- The destructor must not throw.
- Copy operations should be deleted or carefully defined.
- Move operations (if provided) must be `noexcept`.

**Constraints and Limitations**
- Custom wrappers require careful implementation of copy/move semantics.
- The resource handle must be encapsulated to prevent manual release.

### Annotated Code Example

```cpp
#include <cstdio>
#include <iostream>
#include <stdexcept>

class FileWrapper {
    std::FILE* file_;
public:
    explicit FileWrapper(const char* path)
        : file_(std::fopen(path, "r")) {
        if (!file_) throw std::runtime_error("open failed");
    }
    ~FileWrapper() {
        if (file_) std::fclose(file_);
    }
    FileWrapper(const FileWrapper&) = delete;
    FileWrapper& operator=(const FileWrapper&) = delete;

    std::FILE* get() const { return file_; }
};

int main() {
    try {
        FileWrapper f("data.txt");
        std::cout << "Opened\n";
    } catch (const std::exception& e) {
        std::cout << "Error: " << e.what() << '\n';
    }
}
```

**Expected Output**  
(Depends on whether `data.txt` exists.)

**Why this output?**  
`FileWrapper` acquires the file in its constructor and closes it in its destructor; copying is deleted to prevent double close.

### Real-World Cases

- **File handles** – custom wrappers for `FILE*`.
- **Sockets** – wrappers for `socket`/`close`.
- **Database connections** – connection pool handles.
- **GPU resources** – buffer and texture wrappers.

---

## 8. RAII Examples

### Definitions

**Core Definition**
RAII examples are concrete illustrations of the RAII idiom applied to common resources.

**Technical Definition**
RAII examples demonstrate the pattern across different resource types: memory (smart pointers), files (`fstream`), mutexes (`lock_guard`), threads (`jthread`), and custom resources. Each example shows acquisition in the constructor, release in the destructor, and the resulting automatic cleanup on scope exit.

**Beginner-Friendly Explanation**
RAII examples show how the pattern works in practice—for memory, files, locks, and other resources.

### Purposes

- To illustrate the RAII pattern concretely.
- To demonstrate the breadth of RAII applicability.
- To provide templates for custom RAII wrappers.

### Examples

**Example 1: Memory (`std::unique_ptr`)**

```cpp
#include <memory>
#include <iostream>

int main() {
    {
        auto p = std::make_unique<int>(42);
        std::cout << *p << '\n';       // 42
    } // memory released
}
```

**Expected Output**
```
42
```

**Example 2: File (`std::fstream`)**

```cpp
#include <fstream>
#include <iostream>

int main() {
    {
        std::ofstream file("output.txt");
        file << "hello\n";
    } // file closed
    std::cout << "Done\n";
}
```

**Expected Output**
```
Done
```

**Example 3: Mutex (`std::lock_guard`)**

```cpp
#include <mutex>
#include <iostream>

std::mutex mtx;

int main() {
    {
        std::lock_guard<std::mutex> lock(mtx);
        std::cout << "Locked\n";
    } // unlocked
    std::cout << "Unlocked\n";
}
```

**Expected Output**
```
Locked
Unlocked
```

**Example 4: Custom socket wrapper**

```cpp
#include <iostream>

class Socket {
    int fd_;
public:
    Socket() : fd_(-1) { /* open socket */ }
    ~Socket() { if (fd_ >= 0) close(fd_); }
    Socket(const Socket&) = delete;
    Socket& operator=(const Socket&) = delete;
};
```

**Expected Output**  
(No output; socket closed automatically.)

### Real-World Cases

- **Network servers** – RAII socket wrappers.
- **Database clients** – RAII connection handles.
- **Game engines** – RAII GPU resource wrappers.

---

## 9. RAII Best Practices

### Definitions

**Core Definition**
RAII best practices are guidelines for designing and using RAII wrappers correctly, safely, and idiomatically, as codified in the C++ Core Guidelines (R.1–R.5, E.6).

**Technical Definition**
These practices include: manage resources automatically using RAII (R.1); prefer scoped objects over heap allocation (R.5); avoid explicit `new`/`delete` (R.11); ensure destructors never throw (E.16); use `noexcept` for move operations and swap (F.6); and prefer standard RAII wrappers (smart pointers, locks, streams) over custom ones. The guidelines emphasise that RAII should be the default approach to resource management.

**Beginner-Friendly Explanation**
Best practices for RAII are simple: let objects own resources, don't write manual cleanup code, make destructors `noexcept`, and use standard RAII wrappers whenever possible.

### Key Guidelines

| Guideline | Summary |
|-----------|---------|
| **R.1** | Manage resources automatically using resource handles and RAII. |
| **R.5** | Prefer scoped objects; don't heap-allocate unnecessarily. |
| **R.11** | Avoid explicit `new` and `delete`. |
| **E.16** | Destructors, deallocation, and `swap` must never fail. |
| **F.6** | If your function must not throw, declare it `noexcept`. |
| **Prefer standard wrappers** | Use `unique_ptr`, `fstream`, `lock_guard`, etc. |
| **Delete copy operations** | For RAII wrappers that own unique resources. |
| **Provide `noexcept` move** | If the wrapper is movable. |
| **Encapsulate the resource** | Avoid exposing raw handles. |

### Annotated Code Example

```cpp
#include <memory>
#include <iostream>

void process() {
    auto resource = std::make_unique<int>(42);   // RAII
    std::cout << *resource << '\n';
} // resource released automatically

int main() {
    process();
}
```

**Expected Output**
```
42
```

**Why this output?**  
`unique_ptr` manages the memory automatically; no manual `delete` is needed.

### Real-World Cases

- **Code reviews** – flagging manual resource management.
- **Static analysis** – clang-tidy `cppcoreguidelines-owning-memory`.
- **Library design** – exposing RAII-based interfaces.

---

## References

- Resource management — cppreference – https://en.cppreference.com/w/cpp/language/raii
- C++ Core Guidelines — R.1: Manage resources automatically using resource handles and RAII – https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rr-raii
- C++ Core Guidelines — R.5: Prefer scoped objects, don't heap-allocate unnecessarily – https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rr-scoped
- C++ Core Guidelines — R.11: Avoid calling `new` and `delete` explicitly – https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rr-newdelete
- C++ Core Guidelines — E.16: Destructors, deallocation, and `swap` must never fail – https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Re-never-fail
- C++ Core Guidelines — E.6: Use RAII to prevent leaks – https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Re-raii
- RAII — Wikipedia – https://en.wikipedia.org/wiki/Resource_acquisition_is_initialization
- Write correct exception-safe code using RAII and strong/basic guarantees — Dr-Sergey/learn_cpp – https://github.com/Dr-Sergey/learn_cpp/blob/main/11_Error_Handling/Write_correct_exception-safe_code_using_RAII_and_strongbasic_guarantees.md
- Understand and apply the Rule of Five and Rule of Zero — Dr-Sergey/learn_cpp – https://github.com/Dr-Sergey/learn_cpp/blob/main/02_Type_System_and_Deduction/Understand_and_apply_the_Rule_of_Five_and_Rule_of_Zero.md
- RAII — Microsoft Learn – https://learn.microsoft.com/en-us/cpp/cpp/object-lifetime-and-resource-management-modern-cpp
