# Memory Management — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
Memory management is the discipline of allocating, using, and deallocating memory in a C++ program, encompassing storage duration categories (automatic, dynamic, static, thread-local), allocation/deallocation mechanisms (`new`/`delete`, allocators), and strategies to avoid leaks, fragmentation, and undefined behavior.

**Technical Definition**
C++ defines several **storage durations**: *automatic* (block-scope objects, typically on the stack), *dynamic* (objects created with `new`, managed manually or by smart pointers), *static* (objects with static or thread storage duration, existing for the program's or thread's lifetime), and *thread* (objects whose lifetime is tied to a thread). Memory is acquired through allocation functions (`operator new`, `std::malloc`, allocators) and released through deallocation functions (`operator delete`, `std::free`). The C++ standard specifies alignment requirements, the semantics of placement `new`, the customization points `operator new`/`operator delete`, and the allocator model (`std::allocator`, allocator-aware containers, and since C++17 the **Polymorphic Memory Resource** library `std::pmr`).

**Beginner-Friendly Explanation**
Memory management is about **where your data lives and how long it lives there**. Think of memory as a huge warehouse with different sections:
- **Stack memory** is a small, fast shelf where local variables sit; items are automatically removed when you leave the function.
- **Heap memory** is a large, flexible storage area where you can request space at runtime; you must **explicitly** return it when done, or it leaks.
- **Static memory** is a permanent cabinet for globals and `static` variables.
- **Thread-local memory** is a personal locker for each thread.

C++ gives you tools to request and return heap memory (`new`/`delete`), strategies to manage it efficiently (pools, arenas, custom allocators), and modern facilities (`std::pmr`) to make allocation flexible and fast. Doing this correctly prevents **leaks** (forgotten memory), **fragmentation** (wasted gaps), and **crashes**.

### Key Characteristics

- **Storage duration** determines lifetime and where memory is allocated.
- **Manual vs. automatic management**: stack and static memory are managed automatically; heap memory requires explicit management or RAII.
- **Alignment** ensures objects are placed at addresses suitable for their types; misalignment causes undefined behavior or crashes.
- **Fragmentation** reduces usable memory over time; pools and arenas mitigate it.
- **Allocators** decouple container storage from raw `new`/`delete`, enabling custom strategies.
- **`std::pmr`** (C++17) provides runtime-polymorphic memory resources for flexible allocation.
- **RAII** is the idiomatic C++ solution: tie resource lifetime to object lifetime (smart pointers, containers).

### Prerequisites

- Understanding of variables, object lifetimes, and scope.
- Familiarity with pointers and references.
- Knowledge of constructors and destructors.
- Basic awareness of the call stack and the free store.

### Related Programming Areas

- **RAII and smart pointers** – automatic resource management.
- **Containers** – `std::vector`, `std::string`, and allocator-aware containers.
- **Exception safety** – allocation failures and cleanup during unwinding.
- **Performance optimization** – pools, arenas, and custom allocators.
- **Concurrency** – thread-local storage and thread-safe allocators.
- **Embedded systems** – deterministic memory management without a general-purpose heap.

### Core Concepts / Features

1. Stack Memory
2. Heap Memory
3. Static Memory
4. Thread-Local Memory
5. `new`
6. `delete`
7. `new[]`
8. `delete[]`
9. Placement `new`
10. `operator new`
11. `operator delete`
12. Memory Leaks
13. Memory Fragmentation
14. Memory Alignment
15. Memory Pools
16. Arena Allocators
17. Custom Allocators
18. `std::allocator`
19. Allocator-Aware Containers
20. PMR / Polymorphic Memory Resources / `std::pmr`
21. Memory Management Best Practices

---

## 1. Stack Memory

### Definitions

**Core Definition**
Stack memory is the region of memory used for **automatic storage duration** objects — local variables and function call frames — that is automatically allocated when a function is entered and deallocated when it returns.

**Technical Definition**
The stack (also called the *call stack* or *automatic storage*) is a contiguous, LIFO region of memory managed by the program's runtime. Each function call pushes a **stack frame** containing parameters, local variables, saved registers, and the return address; returning from the function pops the frame. Objects with automatic storage duration are constructed when their declaration is reached and destroyed when control leaves their scope (in reverse order of construction). Stack size is typically limited (e.g., 1–8 MB on desktop systems) and exceeding it causes **stack overflow**.

**Beginner-Friendly Explanation**
Stack memory is like a **stack of trays** in a cafeteria: you put a tray on top when you enter a function, and you take it off when you leave. It is very fast because it is just a pointer that moves up and down. Local variables live here, and they disappear automatically when the function returns.

### Purposes

- To provide fast, automatic allocation for local variables and function state.
- To support recursion and nested function calls.
- To automatically manage lifetime (constructors and destructors run at scope exit).
- To enable efficient parameter passing and return-address management.

### Syntax Rules and Structure

Stack allocation is **implicit** — you declare a local variable, and the compiler places it on the stack.

**General Syntax**
```cpp
void f() {
    int x = 42;            // automatic storage duration (stack)
    std::string s = "hi";  // stack object with heap-backed internal buffer
}   // x and s are destroyed here
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| Local variable | Object with automatic storage duration. |
| Stack frame | Per-call region holding locals, parameters, return address. |

**Syntax Rules**
- Local variables have automatic storage duration by default.
- They are destroyed in reverse order of construction when their scope exits.
- Stack allocation is faster than heap allocation (pointer adjustment vs. allocator call).
- Stack objects must have a size known at compile time (no variable-length arrays in standard C++).

**Constraints and Limitations**
- Stack size is limited; deep recursion or large local arrays cause stack overflow.
- Stack objects cannot outlive their scope (returning a pointer/reference to a local is undefined behavior).
- The stack is not suitable for large or long-lived data.

### Annotated Code Examples

**Example 1: Stack allocation and automatic destruction**

```cpp
#include <iostream>

struct Tracer {
    Tracer()  { std::cout << "constructed\n"; }
    ~Tracer() { std::cout << "destroyed\n"; }
};

void f() {
    Tracer t;        // constructed on the stack
    std::cout << "inside f\n";
}   // t destroyed here

int main() {
    f();
    std::cout << "back in main\n";
}
```

**Expected Output**
```
constructed
inside f
destroyed
back in main
```

**Why this output?**  
`Tracer t` is constructed when `f` is entered and destroyed when `f` returns, before `main` continues.

### Real-World Cases

- **Local variables** – the default for most variables.
- **Function parameters** – passed via the stack or registers.
- **RAII objects** – smart pointers, locks, file handles managed on the stack.
- **Small, short-lived buffers** – temporary arrays for computation.

---

## 2. Heap Memory

### Definitions

**Core Definition**
Heap memory (also called the **free store** in C++) is the region of memory used for **dynamic storage duration** objects, whose lifetime is not tied to a scope and is controlled explicitly by the programmer (or by RAII wrappers).

**Technical Definition**
The free store is managed by the runtime's allocator (typically backed by the operating system's heap). Objects are created with `new` (or allocator functions) and destroyed with `delete` (or deallocated with `operator delete`). Their lifetime extends from construction until explicit destruction or program termination (for leaked objects). Heap allocation is slower than stack allocation and can fragment; it is suitable for large objects, objects whose size is not known at compile time, and objects that must outlive their creating scope.

**Beginner-Friendly Explanation**
Heap memory is like a **large warehouse** where you can rent space for as long as you want. You ask for a storage unit (allocate), use it, and must **remember to return the key** (deallocate) when you're done. If you forget, the unit stays rented forever — that's a **memory leak**.

### Purposes

- To allocate objects whose size is determined at runtime.
- To create objects that must outlive the function that created them.
- To allocate large objects that would overflow the stack.
- To share data between functions and threads.

### Syntax Rules and Structure

**General Syntax**
```cpp
T* p = new T(args...);        // allocate and construct
delete p;                      // destroy and deallocate
T* arr = new T[n];             // allocate array
delete[] arr;                  // destroy array and deallocate
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `new T(args...)` | Allocates memory and constructs a `T`. |
| `delete p` | Destroys the object and deallocates memory. |
| `new T[n]` | Allocates an array and default-constructs elements. |
| `delete[] arr` | Destroys array elements and deallocates memory. |

**Syntax Rules**
- Every `new` must be matched by exactly one `delete` (or `delete[]` for arrays).
- `delete` on a null pointer is a no-op.
- Mixing `new`/`delete` with `new[]`/`delete[]` is undefined behavior.
- Prefer smart pointers (`std::unique_ptr`, `std::shared_ptr`) over raw `new`/`delete`.

**Constraints and Limitations**
- Manual `delete` is error-prone (leaks, double deletion, dangling pointers).
- Heap allocation is slower and can fragment memory.
- Exceptions during construction require careful cleanup (use RAII).

### Annotated Code Examples

**Example 1: Basic heap allocation**

```cpp
#include <iostream>

int main() {
    int* p = new int(42);      // allocate and initialize
    std::cout << *p << '\n';
    delete p;                  // destroy and deallocate
    p = nullptr;               // avoid dangling pointer
}
```

**Expected Output**
```
42
```

**Why this output?**  
`new int(42)` allocates heap memory and initializes it to `42`. `delete p` destroys the object and frees the memory.

### Real-World Cases

- **Dynamic data structures** – linked lists, trees, graphs.
- **Large buffers** – image data, audio buffers.
- **Polymorphic objects** – objects created via factories.
- **Shared data** – objects accessed by multiple components.

---

## 3. Static Memory

### Definitions

**Core Definition**
Static memory is the region used for objects with **static storage duration** — global variables, namespace-scope variables, static class members, and function-local `static` variables — which exist for the entire duration of the program.

**Technical Definition**
Objects with static storage duration are allocated when the program starts and destroyed when it ends. They are either **zero-initialized** (for static storage) or **constant-initialized** (if their initializer is a constant expression), and may undergo **dynamic initialization** before `main` (or on first use for function-local statics). Static storage duration applies to variables declared at namespace scope (including global scope), `static` data members, and local variables declared `static`.

**Beginner-Friendly Explanation**
Static memory is a **permanent cabinet** in the warehouse. Anything stored there stays for the entire life of the program. Global variables and `static` variables live here. Function-local `static` variables are special: they are created the first time the function is called and persist until the program ends.

### Purposes

- To provide global state and configuration.
- To implement singletons and function-local caches.
- To share data across function calls without passing parameters.
- To store constants used throughout the program.

### Syntax Rules and Structure

**General Syntax**
```cpp
int global = 42;                 // static storage duration (namespace scope)
void f() {
    static int counter = 0;      // static storage duration (function-local)
    ++counter;
}
struct S {
    static int value;            // static data member
};
int S::value = 0;                // definition
```

**Component Breakdown**

| Form | Storage Duration |
|------|------------------|
| Namespace-scope variable | Static |
| `static` local variable | Static (initialized once) |
| `static` data member | Static |
| `thread_local` variable | Thread |

**Syntax Rules**
- Namespace-scope variables have static storage duration by default.
- Function-local `static` variables are initialized once, on first execution of their declaration.
- Static data members must be defined outside the class (or `inline` since C++17).
- Dynamic initialization order across translation units is unspecified (the “static initialization order fiasco”).

**Constraints and Limitations**
- Static objects persist until program termination; they can hide dependencies and complicate testing.
- Initialization order across translation units is unspecified; use function-local statics or `constinit` to avoid issues.
- Static objects increase startup and shutdown time.

### Annotated Code Examples

**Example 1: Function-local `static`**

```cpp
#include <iostream>

int next_id() {
    static int id = 0;   // initialized once
    return ++id;
}

int main() {
    std::cout << next_id() << ' ' << next_id() << ' ' << next_id() << '\n';
}
```

**Expected Output**
```
1 2 3
```

**Why this output?**  
`id` is a function-local `static` variable, initialized once and retained across calls. Each call increments and returns it.

### Real-World Cases

- **Global configuration** – application settings, constants.
- **Singletons** – a single global instance.
- **Caches** – function-local static caches.
- **Logging** – global logger instances.

---

## 4. Thread-Local Memory

### Definitions

**Core Definition**
Thread-local memory is memory whose storage duration is tied to a **thread**; each thread gets its own independent instance of a `thread_local` variable.

**Technical Definition**
The `thread_local` specifier (C++11) declares that a variable has **thread storage duration**. Each thread that uses the variable has its own copy, initialized when the thread starts (or on first use for block-scope `thread_local`) and destroyed when the thread ends. `thread_local` can be combined with `static` or `extern` to control linkage. It is commonly used to avoid data races on shared state and to implement per-thread caches and contexts.

**Beginner-Friendly Explanation**
Thread-local memory is like a **personal locker** for each thread. If two threads access the same `thread_local` variable, they each see their **own** copy — no sharing, no data races. It is useful for per-thread state like error codes, caches, and random-number generators.

### Purposes

- To provide each thread with its own copy of a variable.
- To avoid data races on shared state.
- To implement per-thread caches, contexts, and error codes.
- To replace global variables in multithreaded programs.

### Syntax Rules and Structure

**General Syntax**
```cpp
thread_local int counter = 0;       // namespace scope
void f() {
    thread_local int local = 0;     // block scope
}
```

**Component Breakdown**

| Form | Meaning |
|------|---------|
| `thread_local` | Thread storage duration. |
| `thread_local static` | Thread storage + internal linkage. |
| `thread_local extern` | Thread storage + external linkage. |

**Syntax Rules**
- `thread_local` can be applied to namespace-scope variables, block-scope variables, and static data members.
- Block-scope `thread_local` variables are initialized on first use in each thread.
- Namespace-scope `thread_local` variables may be initialized before the thread starts or on first use.
- `thread_local` implies static storage duration within each thread.

**Constraints and Limitations**
- Each thread has its own copy, increasing memory usage proportional to the number of threads.
- Initialization order and destruction order across threads are nuanced.
- Accessing a `thread_local` variable from a different thread is not possible (each thread sees its own copy).
- `thread_local` variables may involve guard variables and runtime overhead.

### Annotated Code Examples

**Example 1: Per-thread counter**

```cpp
#include <iostream>
#include <thread>

thread_local int counter = 0;

void increment() {
    ++counter;
    std::cout << "Thread " << std::this_thread::get_id()
              << " counter = " << counter << '\n';
}

int main() {
    std::thread t1(increment);
    std::thread t2(increment);
    t1.join();
    t2.join();
}
```

**Possible Output** (thread IDs vary)
```
Thread 12345 counter = 1
Thread 67890 counter = 1
```

**Why this output?**  
Each thread has its own `counter`, so both start at `0` and increment to `1` independently.

### Real-World Cases

- **Error codes** – `errno` is typically thread-local.
- **Random number generators** – per-thread RNG state.
- **Caches** – per-thread allocation caches.
- **Contexts** – per-thread logging or transaction contexts.

---

## 5. `new`

### Definitions

**Core Definition**
`new` is a C++ operator that **allocates memory** from the free store and **constructs** an object in that memory, returning a pointer to the object.

**Technical Definition**
The `new` expression first calls an allocation function (`operator new`) to obtain raw memory, then constructs the object in that memory using the appropriate constructor. If construction throws, the allocation is automatically deallocated. The result is a pointer to the constructed object. The `new` expression has several forms: `new T`, `new T(args...)`, `new T[n]`, and placement `new`.

**Beginner-Friendly Explanation**
`new` is how you **rent space in the warehouse and put something in it**. It finds room, builds your object there, and gives you the address. You must later use `delete` to destroy the object and return the space.

### Purposes

- To allocate objects with dynamic storage duration.
- To construct objects whose size or lifetime is determined at runtime.
- To create objects that outlive their creating scope.
- To allocate arrays of objects.

### Syntax Rules and Structure

**General Syntax**
```cpp
T* p = new T;              // default-initialized
T* p = new T(args...);     // value-initialized / constructor args
T* arr = new T[n];         // array of n elements
```

**Component Breakdown**

| Form | Behavior |
|------|----------|
| `new T` | Allocates and default-initializes (fundamental types are uninitialized). |
| `new T()` | Allocates and value-initializes (zero for fundamental types). |
| `new T(args...)` | Allocates and constructs with args. |
| `new T[n]` | Allocates an array and default-constructs elements. |

**Syntax Rules**
- `new` returns a pointer to the constructed object (or array).
- If allocation fails, `new` throws `std::bad_alloc` (unless the `nothrow` form is used).
- `new T` default-initializes; `new T()` value-initializes.
- Arrays allocated with `new[]` must be deallocated with `delete[]`.

**Constraints and Limitations**
- Manual `new`/`delete` is error-prone; prefer `std::make_unique` / `std::make_shared`.
- `new` can be overridden by class-specific `operator new`.
- Placement `new` does not allocate memory; it constructs at a given address.

### Annotated Code Examples

**Example 1: `new` with constructor arguments**

```cpp
#include <iostream>
#include <string>

struct Person {
    std::string name;
    int age;
    Person(std::string n, int a) : name(std::move(n)), age(a) {}
};

int main() {
    Person* p = new Person("Alice", 30);
    std::cout << p->name << " is " << p->age << '\n';
    delete p;
}
```

**Expected Output**
```
Alice is 30
```

**Why this output?**  
`new Person("Alice", 30)` allocates memory and constructs the `Person` with the given arguments.

### Real-World Cases

- **Dynamic object creation** – factories returning raw pointers (legacy).
- **Polymorphic objects** – `new Derived()` assigned to `Base*`.
- **Custom allocation** – class-specific `operator new`.

---

## 6. `delete`

### Definitions

**Core Definition**
`delete` is a C++ operator that **destroys** an object created with `new` and **deallocates** its memory.

**Technical Definition**
The `delete` expression first calls the object's destructor, then calls a deallocation function (`operator delete`) to release the memory. The pointer must point to an object created by `new` (or be null); otherwise, the behavior is undefined. Deleting a pointer to a base class with a non-virtual destructor is undefined behavior if the object is actually of a derived class. Deleting the same pointer twice is undefined behavior.

**Beginner-Friendly Explanation**
`delete` is how you **destroy the object and return the warehouse space**. You must use it exactly once for each `new`, and never on the same pointer twice. After `delete`, the pointer is dangling — set it to `nullptr` to avoid accidents.

### Purposes

- To destroy dynamically allocated objects and release their memory.
- To invoke destructors for cleanup (files, locks, resources).
- To prevent memory leaks.

### Syntax Rules and Structure

**General Syntax**
```cpp
delete p;        // destroy single object
delete[] arr;    // destroy array
```

**Component Breakdown**

| Form | Behavior |
|------|----------|
| `delete p` | Destroys the object and deallocates memory. |
| `delete[] arr` | Destroys each element and deallocates the array. |
| `delete nullptr` | No-op (safe). |

**Syntax Rules**
- `delete` on a null pointer is safe (no-op).
- `delete` must match `new`; `delete[]` must match `new[]`.
- After `delete`, the pointer is dangling; set it to `nullptr`.
- The destructor must be virtual if deleting through a base-class pointer to a derived object.

**Constraints and Limitations**
- Double deletion is undefined behavior.
- Deleting a pointer not obtained from `new` is undefined behavior.
- Mixing `new`/`delete` with `new[]`/`delete[]` is undefined behavior.
- Prefer smart pointers to avoid manual `delete`.

### Annotated Code Examples

**Example 1: `delete` invokes the destructor**

```cpp
#include <iostream>

struct Resource {
    Resource()  { std::cout << "acquired\n"; }
    ~Resource() { std::cout << "released\n"; }
};

int main() {
    Resource* r = new Resource;
    delete r;      // destructor runs, memory freed
}
```

**Expected Output**
```
acquired
released
```

**Why this output?**  
`delete r` calls `~Resource()` (printing "released") and then deallocates the memory.

### Real-World Cases

- **Legacy code** – manual `delete` in pre-RAII codebases.
- **Custom allocators** – paired with custom `operator delete`.
- **Polymorphic cleanup** – deleting through base pointers with virtual destructors.

---

## 7. `new[]`

### Definitions

**Core Definition**
`new[]` is the array form of `new` that allocates memory for an **array of objects** and default-constructs each element.

**Technical Definition**
The `new[]` expression allocates memory for an array of `n` objects of type `T`, default-constructs (or value-initializes) each element, and returns a pointer to the first element. The allocated memory is deallocated with `delete[]`. For class types, the compiler may store the array length (the *array cookie*) in the allocation to support correct destruction.

**Beginner-Friendly Explanation**
`new[]` is how you create an **array on the heap**. You specify how many elements you want, and `new[]` builds them all. You must free the array with `delete[]` (not `delete`), because the runtime needs to know to destroy all the elements.

### Purposes

- To allocate arrays whose size is determined at runtime.
- To create dynamic arrays of objects with constructors.
- To support data structures that grow or shrink.

### Syntax Rules and Structure

**General Syntax**
```cpp
T* arr = new T[n];       // default-construct n elements
delete[] arr;            // destroy and deallocate
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `T[n]` | Array of `n` elements of type `T`. |
| `delete[]` | Must be used for arrays. |

**Syntax Rules**
- `new T[n]` default-initializes each element.
- `new T[n]()` value-initializes each element.
- `delete[]` must be used for arrays; `delete` is undefined behavior.
- For class types, constructors run for each element (in order), and destructors run for each element (in reverse order) on `delete[]`.

**Constraints and Limitations**
- `n` must be a non-negative integer; negative or huge values throw `std::bad_alloc` or `std::bad_array_new_length`.
- Array size information is lost; the programmer must track it separately.
- Prefer `std::vector<T>` for dynamic arrays.

### Annotated Code Examples

**Example 1: Dynamic array with `new[]`**

```cpp
#include <iostream>

int main() {
    int n = 5;
    int* arr = new int[n];        // default-initialized (indeterminate values)
    for (int i = 0; i < n; ++i) arr[i] = i * 10;
    for (int i = 0; i < n; ++i) std::cout << arr[i] << ' ';
    std::cout << '\n';
    delete[] arr;                  // must use delete[]
}
```

**Expected Output**
```
0 10 20 30 40 
```

**Why this output?**  
`new int[5]` allocates five `int`s. The loop assigns values; `delete[]` deallocates the array.

### Real-World Cases

- **Dynamic buffers** – byte arrays for I/O.
- **Legacy arrays** – pre-`std::vector` code.
- **Custom data structures** – dynamic arrays inside containers.

---

## 8. `delete[]`

### Definitions

**Core Definition**
`delete[]` is the operator that **destroys an array of objects** created with `new[]` and **deallocates** its memory.

**Technical Definition**
The `delete[]` expression calls the destructor of each element of the array (in reverse order of construction) and then calls `operator delete[]` to release the memory. The pointer must have been returned by `new[]` (or be null); otherwise, the behavior is undefined. Using `delete` instead of `delete[]` on an array is undefined behavior because the array length is not known to `delete`.

**Beginner-Friendly Explanation**
`delete[]` is the **matching cleanup** for `new[]`. It destroys every element in the array and returns the whole block of memory. You must use `delete[]`, not `delete`, because the runtime needs to know how many elements to destroy.

### Purposes

- To destroy all elements of a dynamically allocated array.
- To deallocate the memory block correctly.
- To ensure destructors run for each element.

### Syntax Rules and Structure

**General Syntax**
```cpp
delete[] arr;
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `delete[]` | Array deallocation operator. |
| `arr` | Pointer to the first element of a `new[]`-allocated array. |

**Syntax Rules**
- Must match `new[]`; using `delete` on an array is undefined behavior.
- `delete[] nullptr` is a safe no-op.
- After `delete[]`, the pointer is dangling; set it to `nullptr`.

**Constraints and Limitations**
- The pointer must point to the first element of the array (or be null).
- Deleting a sub-array pointer (e.g., `arr + 1`) is undefined behavior.

### Annotated Code Examples

**Example 1: `delete[]` destroys all elements**

```cpp
#include <iostream>

struct Tracer {
    int id;
    Tracer(int i) : id(i) { std::cout << "construct " << id << '\n'; }
    ~Tracer() { std::cout << "destroy " << id << '\n'; }
};

int main() {
    Tracer* arr = new Tracer[3]{{0}, {1}, {2}};
    delete[] arr;
}
```

**Expected Output**
```
construct 0
construct 1
construct 2
destroy 2
destroy 1
destroy 0
```

**Why this output?**  
`new[]` constructs elements in order; `delete[]` destroys them in reverse order.

### Real-World Cases

- **Array cleanup** – matching `new[]` in dynamic buffers.
- **Custom containers** – internal array deallocation.
- **Legacy code** – replacing raw arrays with `std::vector`.

---

## 9. Placement `new`

### Definitions

**Core Definition**
Placement `new` is a form of `new` that **constructs an object at a caller-provided address** without allocating memory.

**Technical Definition**
Placement `new` is expressed as `new (ptr) T(args...)`, where `ptr` is a `void*` pointing to already-allocated memory. It invokes the constructor of `T` at that address and returns a pointer to the constructed object. It does not call `operator new` (unless overloaded); the standard placement form is `void* operator new(std::size_t, void*) noexcept`. The corresponding destruction must be performed by explicitly calling the destructor (`p->~T()`); `delete` must not be used on placement-constructed objects (unless the allocation was also from `new`).

**Beginner-Friendly Explanation**
Placement `new` is like saying: “**Don’t find new space — build the object right here** at this address I give you.” It is used when you have already reserved memory (e.g., in a memory pool or buffer) and want to construct an object in it.

### Purposes

- To construct objects in pre-allocated memory (pools, arenas, buffers).
- To implement custom allocators and containers.
- To construct objects at specific addresses (e.g., hardware registers).
- To separate allocation from construction.

### Syntax Rules and Structure

**General Syntax**
```cpp
#include <new>

void* buffer = /* pre-allocated memory */;
T* p = new (buffer) T(args...);   // placement new
p->~T();                          // explicit destructor call
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `new (ptr)` | Placement form; `ptr` is the target address. |
| `T(args...)` | The object to construct at `ptr`. |
| `p->~T()` | Explicit destructor call (required). |

**Syntax Rules**
- The target address must be suitably aligned for `T`.
- Placement `new` does not allocate memory; it only constructs.
- The destructor must be called explicitly when the object's lifetime ends.
- The standard placement form is declared in `<new>`.

**Constraints and Limitations**
- Using `delete` on a placement-constructed object is undefined behavior (unless the memory was also obtained from `new`).
- The buffer must remain alive for the object's lifetime.
- Alignment must be correct; use `std::align` or aligned storage.

### Annotated Code Examples

**Example 1: Placement `new` in a buffer**

```cpp
#include <iostream>
#include <new>

struct Widget {
    int value;
    Widget(int v) : value(v) { std::cout << "constructed\n"; }
    ~Widget() { std::cout << "destroyed\n"; }
};

int main() {
    alignas(Widget) unsigned char buffer[sizeof(Widget)];
    Widget* w = new (buffer) Widget(42);   // construct in buffer
    std::cout << w->value << '\n';
    w->~Widget();                          // explicit destruction
}
```

**Expected Output**
```
constructed
42
destroyed
```

**Why this output?**  
`new (buffer) Widget(42)` constructs the `Widget` in the pre-allocated buffer. The destructor is called explicitly to end the object's lifetime.

### Real-World Cases

- **Memory pools** – constructing objects in pool slots.
- **Custom containers** – `std::vector` uses placement `new` internally.
- **Embedded systems** – constructing objects at fixed addresses.
- **Serialization** – constructing objects in byte buffers.

---

## 10. `operator new`

### Definitions

**Core Definition**
`operator new` is the **allocation function** called by `new` expressions to obtain raw memory; it can be overloaded globally or per-class to customize allocation behavior.

**Technical Definition**
The global allocation function `void* operator new(std::size_t)` is called by `new` expressions to allocate memory. It can be replaced (globally) or overloaded (per-class) to implement custom allocation strategies (pools, logging, tracking). The `new` expression calls `operator new` to get memory, then constructs the object. If allocation fails, `operator new` throws `std::bad_alloc` (unless the `nothrow` form is used). Placement `operator new` (`void* operator new(std::size_t, void*)`) does not allocate; it returns the given pointer.

**Beginner-Friendly Explanation**
`operator new` is the **warehouse manager** that finds and reserves raw memory. The `new` keyword asks this manager for space, then builds your object there. You can **replace the manager** to use a custom allocation strategy — for example, a pool that hands out pre-allocated blocks.

### Purposes

- To customize memory allocation (pools, tracking, logging).
- To implement class-specific allocation strategies.
- To integrate with custom allocators and memory resources.
- To debug memory usage by intercepting allocations.

### Syntax Rules and Structure

**General Syntax**
```cpp
void* operator new(std::size_t size);                    // global replacement
void* operator new(std::size_t size, /* custom args */); // overload
```

**Component Breakdown**

| Form | Purpose |
|------|---------|
| `operator new(size_t)` | Standard allocation. |
| `operator new(size_t, nothrow_t)` | Non-throwing allocation. |
| `operator new(size_t, void*)` | Placement (no allocation). |
| Class-specific `operator new` | Per-class allocation. |

**Syntax Rules**
- Global `operator new` can be replaced; class-specific versions can be overloaded.
- The replacement must return a pointer to at least `size` bytes, suitably aligned.
- It should throw `std::bad_alloc` on failure (or return `nullptr` for `nothrow`).
- Class-specific `operator new` is looked up before the global one.

**Constraints and Limitations**
- Replacement functions must obey the standard's requirements (alignment, size).
- Mixing custom `operator new` with `delete` requires a matching `operator delete`.
- Overloading `operator new` does not change the `new` expression's construction semantics.

### Annotated Code Examples

**Example 1: Class-specific `operator new`**

```cpp
#include <iostream>
#include <cstdlib>
#include <new>

struct Widget {
    static void* operator new(std::size_t size) {
        std::cout << "custom alloc: " << size << " bytes\n";
        if (void* p = std::malloc(size)) return p;
        throw std::bad_alloc();
    }
    static void operator delete(void* p) {
        std::cout << "custom free\n";
        std::free(p);
    }
    int value = 42;
};

int main() {
    Widget* w = new Widget;
    std::cout << w->value << '\n';
    delete w;
}
```

**Expected Output**
```
custom alloc: 4 bytes
42
custom free
```

**Why this output?**  
The class-specific `operator new` and `operator delete` are used instead of the global versions, printing custom messages.

### Real-World Cases

- **Memory pools** – class-specific allocation from a pool.
- **Debugging** – tracking allocations and detecting leaks.
- **Embedded systems** – custom allocators with fixed memory regions.
- **Alignment control** – over-aligned types (C++17 `operator new(size_t, align_val_t)`).

---

## 11. `operator delete`

### Definitions

**Core Definition**
`operator delete` is the **deallocation function** called by `delete` expressions to release raw memory; it can be overloaded globally or per-class to match a custom `operator new`.

**Technical Definition**
The global deallocation function `void operator delete(void*)` is called by `delete` expressions after the object's destructor runs. It can be replaced (globally) or overloaded (per-class) to match custom allocation. There are also array forms (`operator delete[](void*)`), sized forms (`operator delete(void*, std::size_t)`), nothrow forms, and aligned forms (C++17). Placement `delete` (matching placement `new`) is called only if the constructor throws.

**Beginner-Friendly Explanation**
`operator delete` is the **warehouse manager's return desk**. It takes back the raw memory that `operator new` handed out. If you replaced `operator new` with a custom allocator, you must also replace `operator delete` to return memory the same way.

### Purposes

- To release memory allocated by a custom `operator new`.
- To track deallocations for debugging.
- To return memory to a pool or arena.
- To maintain symmetry between allocation and deallocation.

### Syntax Rules and Structure

**General Syntax**
```cpp
void operator delete(void* ptr) noexcept;
void operator delete[](void* ptr) noexcept;
void operator delete(void* ptr, std::size_t size) noexcept;
```

**Component Breakdown**

| Form | Purpose |
|------|---------|
| `operator delete(void*)` | Standard deallocation. |
| `operator delete[](void*)` | Array deallocation. |
| `operator delete(void*, size_t)` | Sized deallocation (C++14). |
| `operator delete(void*, align_val_t)` | Aligned deallocation (C++17). |

**Syntax Rules**
- A replacement `operator delete` must have the correct signature and be `noexcept`.
- Class-specific `operator delete` is looked up before the global one.
- If `operator new` is replaced, the matching `operator delete` should also be replaced.
- Placement `delete` is called only if the constructor throws.

**Constraints and Limitations**
- The pointer passed to `operator delete` must have been returned by the matching `operator new` (or be null).
- Mixing custom `operator new` with the default `operator delete` (or vice versa) can cause undefined behavior.

### Annotated Code Examples

**Example 1: Matching `operator new` and `operator delete`**

```cpp
#include <iostream>
#include <cstdlib>
#include <new>

void* operator new(std::size_t size) {
    std::cout << "global alloc: " << size << " bytes\n";
    if (void* p = std::malloc(size)) return p;
    throw std::bad_alloc();
}

void operator delete(void* p) noexcept {
    std::cout << "global free\n";
    std::free(p);
}

int main() {
    int* p = new int(42);
    std::cout << *p << '\n';
    delete p;
}
```

**Expected Output**
```
global alloc: 4 bytes
42
global free
```

**Why this output?**  
The replaced global `operator new` and `operator delete` are used for all allocations and deallocations, printing custom messages.

### Real-World Cases

- **Memory tracking** – logging allocations and frees.
- **Pool deallocation** – returning memory to a pool.
- **Aligned types** – matching aligned `operator new`.

---

## 12. Memory Leaks

### Definitions

**Core Definition**
A memory leak occurs when dynamically allocated memory is **never deallocated**, causing the program's memory usage to grow until resources are exhausted.

**Technical Definition**
A memory leak happens when an object with dynamic storage duration is no longer reachable by the program but has not been destroyed and deallocated. Common causes include forgetting to call `delete`, losing the last pointer to an allocation, circular references in `shared_ptr`, and exceptions that skip cleanup code. Leaks are a form of resource leak and can eventually cause `std::bad_alloc`, performance degradation, or system instability. Tools like Valgrind, AddressSanitizer, and static analyzers detect leaks.

**Beginner-Friendly Explanation**
A memory leak is like **renting a storage unit and never returning the key**. The unit stays rented forever, and the space is wasted. If you keep renting more units and never return keys, eventually the warehouse is full. In C++, a leak happens when you `new` something and forget to `delete` it.

### Purposes

- (This is a failure mode.) Understanding leaks helps prevent them.
- To motivate RAII and smart pointers.
- To justify leak-detection tools and practices.

### Common Causes and Prevention

| Cause | Prevention |
|-------|------------|
| Forgotten `delete` | Use smart pointers (`unique_ptr`, `shared_ptr`). |
| Lost pointer | Store owning pointers in RAII wrappers. |
| Early return or exception | RAII guarantees cleanup. |
| Circular `shared_ptr` | Use `weak_ptr` to break cycles. |
| `new[]` without `delete[]` | Use `std::vector`. |
| `malloc` without `free` | Use RAII wrappers. |

**Syntax Example (RAII Prevention)**
```cpp
void f() {
    auto p = std::make_unique<int>(42);   // automatically deleted
    // no leak even if an exception is thrown
}
```

### Annotated Code Examples

**Example 1: Leak from forgotten `delete`**

```cpp
// DO NOT WRITE THIS CODE
void leak() {
    int* p = new int(42);
    // forgot delete p
}   // p goes out of scope; memory is leaked
```

**Expected Output**  
(Memory leak; no output.)

**Why this leaks?**  
`p` goes out of scope, but the `int` it points to was never deleted. The memory remains allocated but unreachable.

### Real-World Cases

- **Long-running servers** – leaks accumulate and eventually exhaust memory.
- **Embedded systems** – limited memory makes leaks fatal quickly.
- **Leak detection** – Valgrind, AddressSanitizer (`-fsanitize=address`), and MSVC leak detection.

---

## 13. Memory Fragmentation

### Definitions

**Core Definition**
Memory fragmentation is the condition where free memory exists but is **split into small, non-contiguous blocks**, making it unusable for larger allocations.

**Technical Definition**
Fragmentation occurs in two forms: **external fragmentation** (free blocks are individually large enough for some requests but not contiguous for larger ones) and **internal fragmentation** (allocated blocks are larger than requested due to alignment or bookkeeping, wasting space inside each allocation). Fragmentation reduces effective memory capacity over time, especially in programs with varied allocation sizes and lifetimes. Pools, arenas, and size-class allocators mitigate fragmentation by using fixed-size or segregated blocks.

**Beginner-Friendly Explanation**
Fragmentation is like having **lots of small empty parking spaces** when you need one big space for a truck. The total empty space is enough, but it's broken into pieces. Over time, a program that allocates and frees many different-sized objects can end up with memory that is “full of holes.”

### Purposes

- (This is a problem to mitigate.) Understanding fragmentation helps design better allocators.
- To motivate pools, arenas, and size-class allocators.
- To guide allocation patterns in performance-critical code.

### Mitigation Strategies

| Strategy | Description |
|----------|-------------|
| **Memory pools** | Fixed-size blocks per type/size class. |
| **Arenas** | Bulk allocation with bulk deallocation. |
| **Size-class allocators** | Separate free lists per size. |
| **Object reuse** | Recycle objects instead of frequent alloc/free. |
| **Contiguous containers** | Prefer `std::vector` over node-based containers where possible. |

### Annotated Code Examples

**Example 1: Fragmentation from mixed sizes**

```cpp
#include <iostream>
#include <vector>

int main() {
    std::vector<int*> ptrs;
    for (int i = 0; i < 1000; ++i) {
        ptrs.push_back(new int(i));
    }
    for (int i = 0; i < 1000; i += 2) {
        delete ptrs[i];       // free every other allocation
        ptrs[i] = nullptr;
    }
    // Free memory is interleaved with allocated memory → fragmentation
    for (int i = 1; i < 1000; i += 2) {
        delete ptrs[i];
    }
}
```

**Expected Output**  
(No output; illustrative of interleaved allocation/free patterns.)

**Why this matters?**  
Freeing every other allocation creates small gaps between live objects, which can prevent larger allocations from succeeding even though total free memory is sufficient.

### Real-World Cases

- **Long-running servers** – fragmentation can cause allocation failures.
- **Game engines** – pools and arenas avoid fragmentation.
- **Embedded systems** – limited memory makes fragmentation critical.

---

## 14. Memory Alignment

### Definitions

**Core Definition**
Memory alignment is the requirement that objects be placed at addresses that are **multiples of their alignment** (e.g., 4-byte aligned for `int`, 8-byte aligned for `double`), ensuring efficient and correct access.

**Technical Definition**
Each type has an **alignment requirement** (`alignof(T)`), which is the number of bytes between consecutive valid addresses for objects of that type. Misaligned access can cause crashes, performance penalties, or undefined behavior. The `alignas` specifier (C++11) allows specifying stricter alignment for objects and types. Since C++17, over-aligned types (alignment greater than `__STDCPP_DEFAULT_NEW_ALIGNMENT__`) are supported by aligned `operator new`/`operator delete`. The `std::align` function adjusts pointers within a buffer to a specified alignment.

**Beginner-Friendly Explanation**
Alignment is about **placing objects at “nice” addresses**. Just as books on a shelf look neat when aligned to the edge, data in memory is fastest and safest when placed at addresses divisible by its size. Misaligned data is like reading a book with pages out of order — slow and error-prone.

### Purposes

- To ensure correct and efficient memory access.
- To satisfy hardware requirements (some platforms fault on misaligned access).
- To enable SIMD and cache-line alignment.
- To support over-aligned types (C++17).

### Syntax Rules and Structure

**General Syntax**
```cpp
alignas(16) float data[4];       // 16-byte aligned array
struct alignas(32) Vec { float x, y, z, w; };
static_assert(alignof(Vec) == 32);
```

**Component Breakdown**

| Facility | Purpose |
|----------|---------|
| `alignof(T)` | Query alignment requirement. |
| `alignas(N)` | Specify alignment (C++11). |
| `std::align` | Adjust pointer within buffer to alignment. |
| Aligned `operator new` | Support over-aligned allocations (C++17). |

**Syntax Rules**
- `alignas` can be applied to variables, members, and types.
- `alignas` cannot reduce alignment below the natural alignment.
- `alignof` returns `std::size_t`.
- Over-aligned types require aligned allocation (C++17).

**Constraints and Limitations**
- Over-alignment (greater than `alignof(std::max_align_t)`) may not be supported by all allocators before C++17.
- `std::malloc` returns memory aligned for any fundamental type, but not for over-aligned types.
- Misaligned pointer casts cause undefined behavior.

### Annotated Code Examples

**Example 1: `alignas` and `alignof`**

```cpp
#include <iostream>
#include <cstddef>

struct alignas(16) Vec4 {
    float x, y, z, w;
};

int main() {
    std::cout << alignof(Vec4) << '\n';   // 16
    std::cout << sizeof(Vec4) << '\n';    // 16
}
```

**Expected Output**
```
16
16
```

**Why this output?**  
`alignas(16)` forces `Vec4` to be 16-byte aligned. Its size is a multiple of its alignment, so `sizeof` is `16`.

### Real-World Cases

- **SIMD** – 16/32-byte alignment for `__m128`/`__m256`.
- **Cache lines** – 64-byte alignment to avoid false sharing.
- **Hardware registers** – fixed-address, aligned access.
- **Serialization** – ensuring structs are laid out consistently.

---

## 15. Memory Pools

### Definitions

**Core Definition**
A memory pool is a pre-allocated block of memory from which fixed-size (or size-classed) chunks are handed out and returned, reducing allocation overhead and fragmentation.

**Technical Definition**
A memory pool allocates a large contiguous block upfront and subdivides it into slots of a fixed size (or a small set of size classes). Allocation returns a free slot in O(1); deallocation returns the slot to a free list. Pools are ideal for objects of the same type or size, improving locality and eliminating general-purpose allocator overhead. They are a common implementation of custom `operator new` and `std::pmr::memory_resource`.

**Beginner-Friendly Explanation**
A memory pool is like a **vending machine pre-loaded with identical snacks**. Instead of going to the warehouse (general allocator) every time, you grab a snack from the machine (a free slot) instantly. When you're done, you put it back. It is fast and avoids fragmentation because all slots are the same size.

### Purposes

- To reduce allocation overhead for many small objects.
- To avoid fragmentation by using fixed-size blocks.
- To improve cache locality.
- To provide deterministic allocation for real-time systems.

### Syntax Rules and Structure

Pools are typically implemented as custom allocators or `operator new` overloads.

**Conceptual Syntax**
```cpp
template <typename T, std::size_t N>
class Pool {
    alignas(T) unsigned char storage[N * sizeof(T)];
    std::vector<T*> free_list;
public:
    void* allocate() { /* return a free slot */ }
    void deallocate(void* p) { /* return slot to free list */ }
};
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| Storage block | Contiguous memory for all slots. |
| Free list | Tracks available slots. |
| Allocate/deallocate | O(1) operations on the free list. |

**Syntax Rules**
- Slots must be properly aligned for `T`.
- Free list can be an array, linked list, or bitmap.
- Pool lifetime must exceed the lifetime of all allocated objects.

**Constraints and Limitations**
- Fixed-size pools only serve one size class.
- Multi-size pools require size-class segregation.
- Pool memory is not returned to the system until the pool is destroyed.

### Annotated Code Examples

**Example 1: Simple fixed-size pool via custom `operator new`**

```cpp
#include <iostream>
#include <cstddef>
#include <new>

struct Widget {
    int value;
    static void* operator new(std::size_t size) {
        static alignas(Widget) unsigned char buffer[sizeof(Widget) * 10];
        static bool used[10] = {};
        for (int i = 0; i < 10; ++i) {
            if (!used[i]) {
                used[i] = true;
                std::cout << "pool alloc slot " << i << '\n';
                return buffer + i * sizeof(Widget);
            }
        }
        throw std::bad_alloc();
    }
    static void operator delete(void* p) noexcept {
        std::cout << "pool free\n";
        // simplified: real pool would update the free list
    }
};

int main() {
    Widget* w = new Widget{42};
    std::cout << w->value << '\n';
    delete w;
}
```

**Expected Output**
```
pool alloc slot 0
42
pool free
```

**Why this output?**  
The class-specific `operator new` hands out slots from a pre-allocated buffer; `operator delete` returns the slot (simplified here).

### Real-World Cases

- **Game engines** – pools for bullets, particles, and entities.
- **Real-time systems** – deterministic allocation.
- **Embedded systems** – fixed memory budgets.
- **High-frequency trading** – low-latency allocation.

---

## 16. Arena Allocators

### Definitions

**Core Definition**
An arena allocator (also called a **bump allocator** or **region allocator**) allocates memory by simply advancing a pointer through a large pre-allocated block, and frees all memory at once by resetting the pointer.

**Technical Definition**
An arena allocator manages a contiguous region of memory. Each allocation “bumps” the current offset forward by the requested size (aligned appropriately). Individual deallocations are not supported; instead, the entire arena is reset or destroyed at once, invalidating all allocations. Arenas are extremely fast (O(1) allocation, no free list) and are ideal for phase-based workloads (e.g., per-frame allocations, parsing, compilation).

**Beginner-Friendly Explanation**
An arena allocator is like a **notepad**: you write things one after another, and when you're done with the whole page, you **tear it off and start fresh**. You can't erase individual items, but writing is super fast. It's perfect when you allocate many things and then throw them all away together.

### Purposes

- To provide extremely fast allocation (pointer bump).
- To simplify memory management for phase-based workloads.
- To avoid fragmentation entirely (contiguous allocation).
- To enable bulk deallocation in O(1).

### Syntax Rules and Structure

**Conceptual Syntax**
```cpp
class Arena {
    unsigned char* buffer;
    std::size_t capacity;
    std::size_t offset = 0;
public:
    void* allocate(std::size_t size, std::size_t align) {
        // align offset, check capacity, return buffer + offset, advance offset
    }
    void reset() { offset = 0; }   // free all at once
};
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| Buffer | Pre-allocated contiguous memory. |
| Offset | Current bump pointer. |
| Reset | Frees all allocations at once. |

**Syntax Rules**
- Align each allocation appropriately.
- Check capacity before bumping.
- Do not use individual deallocation; reset the entire arena.
- Object destructors must be called before reset if they manage resources.

**Constraints and Limitations**
- No individual deallocation.
- Memory is not returned to the system until the arena is destroyed.
- Large arenas may waste memory if usage is low.

### Annotated Code Examples

**Example 1: Simple bump allocator**

```cpp
#include <iostream>
#include <cstddef>
#include <new>

class Arena {
    alignas(std::max_align_t) unsigned char buffer[1024];
    std::size_t offset = 0;
public:
    void* allocate(std::size_t size) {
        std::size_t aligned = (offset + alignof(std::max_align_t) - 1)
                              & ~(alignof(std::max_align_t) - 1);
        if (aligned + size > sizeof(buffer)) throw std::bad_alloc();
        void* p = buffer + aligned;
        offset = aligned + size;
        return p;
    }
    void reset() { offset = 0; }
};

int main() {
    Arena arena;
    int* a = new (arena.allocate(sizeof(int))) int(10);
    double* b = new (arena.allocate(sizeof(double))) double(3.14);
    std::cout << *a << ' ' << *b << '\n';
    arena.reset();   // free all at once
}
```

**Expected Output**
```
10 3.14
```

**Why this output?**  
The arena hands out memory by bumping its offset. `reset()` frees everything at once (the objects are not destroyed here; in real code, call destructors first).

### Real-World Cases

- **Compilers** – per-function or per-pass arenas.
- **Game engines** – per-frame arenas.
- **Parsers** – per-parse arenas.
- **Web servers** – per-request arenas.

---

## 17. Custom Allocators

### Definitions

**Core Definition**
A custom allocator is a class that implements the allocator interface (`allocate`, `deallocate`, and associated types) to provide a specific memory allocation strategy for containers and other allocator-aware types.

**Technical Definition**
In C++, an allocator is a class template `Allocator<T>` that satisfies the *Allocator* requirements: it provides `value_type`, `allocate(n)`, `deallocate(p, n)`, and (historically) construct/destroy operations. Since C++11, allocators are stateless or stateful, and containers use `std::allocator_traits` to interact with them. Custom allocators enable pools, arenas, tracking, shared memory, and `std::pmr` integration.

**Beginner-Friendly Explanation**
A custom allocator is a **specialized warehouse manager** that you plug into a container. Instead of using the default `new`/`delete`, the container asks your allocator for memory. This lets you use pools, arenas, or shared memory with standard containers like `std::vector`.

### Purposes

- To customize container memory allocation.
- To integrate pools, arenas, or shared memory with standard containers.
- To track allocations for debugging.
- To support `std::pmr` and polymorphic memory resources.

### Syntax Rules and Structure

**General Syntax**
```cpp
template <typename T>
struct MyAllocator {
    using value_type = T;
    MyAllocator() = default;
    template <typename U> MyAllocator(const MyAllocator<U>&) {}
    T* allocate(std::size_t n);
    void deallocate(T* p, std::size_t n);
    bool operator==(const MyAllocator&) const { return true; }
};
std::vector<int, MyAllocator<int>> v;
```

**Component Breakdown**

| Member | Purpose |
|--------|---------|
| `value_type` | The allocated type. |
| `allocate(n)` | Allocate memory for `n` objects. |
| `deallocate(p, n)` | Deallocate memory. |
| `operator==` | Allocator equality (for propagation). |

**Syntax Rules**
- Must satisfy the *Allocator* requirements (use `std::allocator_traits` for portability).
- Stateful allocators must define `operator==` and `operator!=`.
- Containers obtain allocators via `get_allocator()` and propagate them per `allocator_traits`.
- Since C++17, `std::pmr::polymorphic_allocator` is the standard runtime-polymorphic allocator.

**Constraints and Limitations**
- Custom allocators must be careful with alignment and object lifetime.
- Allocator propagation rules affect copy/move/swap behavior of containers.
- Writing a correct custom allocator is nontrivial; `std::pmr` is often simpler.

### Annotated Code Examples

**Example 1: Tracking allocator**

```cpp
#include <iostream>
#include <vector>
#include <memory>

template <typename T>
struct TrackingAllocator {
    using value_type = T;
    TrackingAllocator() = default;
    template <typename U> TrackingAllocator(const TrackingAllocator<U>&) {}
    T* allocate(std::size_t n) {
        std::cout << "alloc " << n << " objects\n";
        return std::allocator<T>{}.allocate(n);
    }
    void deallocate(T* p, std::size_t n) {
        std::cout << "dealloc " << n << " objects\n";
        std::allocator<T>{}.deallocate(p, n);
    }
    bool operator==(const TrackingAllocator&) const { return true; }
    bool operator!=(const TrackingAllocator&) const { return false; }
};

int main() {
    std::vector<int, TrackingAllocator<int>> v;
    v.push_back(1);
    v.push_back(2);
}
```

**Possible Output**
```
alloc 1 objects
alloc 2 objects
dealloc 1 objects
dealloc 2 objects
```

**Why this output?**  
The tracking allocator logs each allocation and deallocation performed by the vector. (Exact order depends on growth strategy.)

### Real-World Cases

- **Shared memory** – allocators that place objects in shared memory segments.
- **Pools** – allocators backed by memory pools.
- **Tracking** – debugging and profiling allocators.
- **`std::pmr`** – polymorphic memory resources as allocators.

---

## 18. `std::allocator`

### Definitions

**Core Definition**
`std::allocator<T>` is the default allocator class template used by standard containers; it allocates memory using `::operator new` and deallocates using `::operator delete`.

**Technical Definition**
`std::allocator<T>` is the default allocator for all standard library containers. It provides `value_type`, `allocate`, `deallocate`, and (before C++17) `construct`/`destroy`. Since C++17, many members are deprecated or removed in favor of `std::allocator_traits`. It is stateless and all instances compare equal. `std::allocator<T>` is not intended to be specialized by user code (since C++17).

**Beginner-Friendly Explanation**
`std::allocator` is the **default warehouse manager** used by `std::vector`, `std::list`, `std::map`, and other containers. It uses `new`/`delete` under the hood. You usually don't need to mention it; it is the implicit default.

### Purposes

- To provide the default allocation strategy for standard containers.
- To serve as a model for custom allocators.
- To integrate with `std::allocator_traits`.

### Syntax Rules and Structure

**General Syntax**
```cpp
std::vector<int> v;                              // uses std::allocator<int> implicitly
std::vector<int, std::allocator<int>> v2;        // explicit
```

**Component Breakdown**

| Member | Purpose |
|--------|---------|
| `value_type` | `T` |
| `allocate(n)` | Allocate `n` objects via `::operator new` |
| `deallocate(p, n)` | Deallocate via `::operator delete` |

**Syntax Rules**
- `std::allocator` is stateless; all instances compare equal.
- It uses global `operator new`/`operator delete`.
- Since C++17, it is not user-specializable.
- Use `std::allocator_traits<std::allocator<T>>` for portable access.

**Constraints and Limitations**
- `std::allocator` uses the global allocator, which may not suit all needs (pools, arenas, shared memory).
- It does not support over-aligned types before C++17 (aligned `operator new` is used since C++17).

### Annotated Code Examples

**Example 1: Explicit `std::allocator`**

```cpp
#include <iostream>
#include <memory>

int main() {
    std::allocator<int> alloc;
    int* p = alloc.allocate(1);      // raw memory
    std::allocator_traits<std::allocator<int>>::construct(alloc, p, 42);
    std::cout << *p << '\n';
    std::allocator_traits<std::allocator<int>>::destroy(alloc, p);
    alloc.deallocate(p, 1);
}
```

**Expected Output**
```
42
```

**Why this output?**  
`allocate` returns raw memory; `construct` builds the `int`; `destroy` ends its lifetime; `deallocate` frees the memory.

### Real-World Cases

- **Default containers** – every standard container uses it by default.
- **Allocator traits** – serves as the canonical allocator model.
- **Custom allocator base** – often wraps `std::allocator` internally.

---

## 19. Allocator-Aware Containers

### Definitions

**Core Definition**
Allocator-aware containers are standard containers (e.g., `std::vector`, `std::list`, `std::map`) that accept an allocator template parameter and use it for all internal memory management.

**Technical Definition**
Allocator-aware containers obtain memory exclusively through their allocator, which is specified as a template parameter (defaulting to `std::allocator<T>`). They use `std::allocator_traits` to interact with the allocator, support allocator propagation on copy/move/swap (per `propagate_on_container_copy_assignment`, `propagate_on_container_move_assignment`, and `propagate_on_container_swap`), and expose `get_allocator()`. Since C++17, `std::pmr` containers use `std::pmr::polymorphic_allocator` for runtime-polymorphic allocation.

**Beginner-Friendly Explanation**
Allocator-aware containers are **containers that let you plug in a custom warehouse manager**. `std::vector<T, MyAllocator>` uses `MyAllocator` for its internal storage instead of the default `new`/`delete`. This lets you use pools, arenas, or shared memory with standard containers.

### Purposes

- To customize container memory allocation.
- To integrate containers with pools, arenas, and shared memory.
- To support `std::pmr` runtime-polymorphic allocation.
- To track and control container memory usage.

### Syntax Rules and Structure

**General Syntax**
```cpp
std::vector<T, MyAllocator<T>> v;
std::pmr::vector<T> pv{&resource};
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `Allocator` | Template parameter controlling allocation. |
| `get_allocator()` | Returns the container's allocator. |
| Propagation traits | Control copy/move/swap behavior. |

**Syntax Rules**
- The allocator must satisfy the *Allocator* requirements.
- Container operations use `allocator_traits` for portability.
- Propagation traits determine whether the allocator is copied/moved/swapped.
- `std::pmr` containers use `std::pmr::polymorphic_allocator` by default.

**Constraints and Limitations**
- Allocator propagation rules can be subtle; mismatched allocators may require element-wise moves.
- Stateful allocators must compare correctly with `operator==`.
- Swapping containers with unequal allocators is undefined behavior (unless `propagate_on_container_swap` is true).

### Annotated Code Examples

**Example 1: Container with a custom allocator**

```cpp
#include <iostream>
#include <vector>
#include <memory>

template <typename T>
struct LoggingAllocator {
    using value_type = T;
    LoggingAllocator() = default;
    template <typename U> LoggingAllocator(const LoggingAllocator<U>&) {}
    T* allocate(std::size_t n) {
        std::cout << "alloc " << n << '\n';
        return std::allocator<T>{}.allocate(n);
    }
    void deallocate(T* p, std::size_t n) {
        std::cout << "dealloc " << n << '\n';
        std::allocator<T>{}.deallocate(p, n);
    }
    bool operator==(const LoggingAllocator&) const { return true; }
    bool operator!=(const LoggingAllocator&) const { return false; }
};

int main() {
    std::vector<int, LoggingAllocator<int>> v;
    v.push_back(1);
    v.push_back(2);
}
```

**Possible Output**
```
alloc 1
alloc 2
dealloc 1
dealloc 2
```

**Why this output?**  
The vector uses `LoggingAllocator` for all internal allocations, logging each `allocate` and `deallocate`.

### Real-World Cases

- **`std::pmr::vector`** – runtime-polymorphic allocator.
- **Embedded containers** – fixed-size allocators.
- **Shared-memory containers** – allocators backed by shared memory.
- **Tracking** – debugging allocators for container memory.

---

## 20. PMR / Polymorphic Memory Resources / `std::pmr`

### Definitions

**Core Definition**
Polymorphic Memory Resources (`std::pmr`, C++17) provide **runtime-polymorphic memory allocation**: a `memory_resource` abstraction that containers can use without being templated on the allocator type.

**Technical Definition**
`std::pmr` (in `<memory_resource>`) defines the abstract base class `std::pmr::memory_resource`, which provides `do_allocate`, `do_deallocate`, and `do_is_equal` virtual functions. Concrete resources include `std::pmr::new_delete_resource` (uses `new`/`delete`), `std::pmr::monotonic_buffer_resource` (arena/bump allocator), `std::pmr::unsynchronized_pool_resource` (pool, not thread-safe), and `std::pmr::synchronized_pool_resource` (thread-safe pool). `std::pmr::polymorphic_allocator<T>` wraps a `memory_resource*` and satisfies the *Allocator* requirements, allowing `std::pmr::vector<T>`, `std::pmr::string`, etc., to use any resource at runtime without changing their type.

**Beginner-Friendly Explanation**
`std::pmr` solves a key problem: normally, changing a container's allocator changes its **type**, which is inconvenient for APIs and class members. `std::pmr` instead lets you change the allocator **at runtime** without changing the type. You pass a `memory_resource*` to the container, and it uses that resource for all allocations. You can switch between `new`/`delete`, pools, and arenas without recompiling or retemplating.

### Purposes

- To provide runtime-polymorphic allocation without templating on the allocator.
- To switch allocation strategies at runtime (pools, arenas, new/delete).
- To simplify allocator-aware class interfaces.
- To support per-object allocators without template proliferation.
- To reduce code bloat from template instantiations.

### Syntax Rules and Structure

**General Syntax**
```cpp
#include <memory_resource>

std::pmr::monotonic_buffer_resource arena;
std::pmr::vector<int> v(&arena);
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `std::pmr::memory_resource` | Abstract base for resources. |
| `std::pmr::polymorphic_allocator<T>` | Allocator wrapping a `memory_resource*`. |
| `std::pmr::vector<T>` | Alias for `std::vector<T, std::pmr::polymorphic_allocator<T>>`. |
| `std::pmr::monotonic_buffer_resource` | Arena/bump allocator. |
| `std::pmr::unsynchronized_pool_resource` | Pool (non-thread-safe). |
| `std::pmr::synchronized_pool_resource` | Pool (thread-safe). |
| `std::pmr::new_delete_resource` | Default resource using `new`/`delete`. |

**Syntax Rules**
- Include `<memory_resource>`.
- Resources can be passed to `std::pmr` containers and strings.
- `std::pmr::get_default_resource()` returns the current default resource.
- `std::pmr::set_default_resource()` changes the default.
- A `memory_resource*` must outlive the containers that use it.

**Constraints and Limitations**
- `polymorphic_allocator` has a `memory_resource*` member (stateless type, stateful value); size and propagation are per its specification.
- Resources must outlive all allocators that reference them.
- `monotonic_buffer_resource` does not deallocate individual objects (arena semantics).
- `std::pmr` containers are aliases; their types differ from `std::` containers.

### Annotated Code Examples

**Example 1: `std::pmr::monotonic_buffer_resource` (arena)**

```cpp
#include <iostream>
#include <memory_resource>
#include <vector>

int main() {
    std::pmr::monotonic_buffer_resource arena;
    std::pmr::vector<int> v(&arena);
    for (int i = 0; i < 10; ++i) v.push_back(i);
    for (int x : v) std::cout << x << ' ';
    std::cout << '\n';
    // All memory is released when arena is destroyed
}
```

**Expected Output**
```
0 1 2 3 4 5 6 7 8 9 
```

**Why this output?**  
The vector uses the arena for all allocations. When `arena` is destroyed, all memory is released at once.

**Example 2: Switching resources at runtime**

```cpp
#include <iostream>
#include <memory_resource>
#include <vector>

void fill(std::pmr::vector<int>& v) {
    for (int i = 0; i < 5; ++i) v.push_back(i * i);
}

int main() {
    std::pmr::monotonic_buffer_resource arena;
    std::pmr::vector<int> v1(&arena);
    fill(v1);

    std::pmr::vector<int> v2;   // uses default resource (new/delete)
    fill(v2);

    for (int x : v1) std::cout << x << ' ';
    std::cout << '\n';
    for (int x : v2) std::cout << x << ' ';
    std::cout << '\n';
}
```

**Expected Output**
```
0 1 4 9 16 
0 1 4 9 16 
```

**Why this output?**  
`v1` uses the arena; `v2` uses the default resource. Both produce the same values; the allocation strategy differs at runtime.

### Real-World Cases

- **Game engines** – per-frame arenas with `monotonic_buffer_resource`.
- **Web servers** – per-request pools.
- **Compilers** – per-pass arenas.
- **APIs** – passing a `memory_resource*` to avoid template explosion.

---

## 21. Memory Management Best Practices

### Definitions

**Core Definition**
Memory management best practices are guidelines for allocating, using, and deallocating memory safely, efficiently, and idiomatically in C++.

**Technical Definition**
These practices are codified in the C++ Core Guidelines (especially **R.10–R.13**, **I.11**, **C.149**, **F.43**), CERT C++ rules (MEM), and the standard library's RAII philosophy. They emphasize: prefer automatic storage, use RAII for dynamic resources, avoid raw `new`/`delete`, use `std::make_unique`/`std::make_shared`, respect alignment, avoid fragmentation with pools/arenas, use `std::pmr` for polymorphic allocation, and never return pointers/references to locals.

**Beginner-Friendly Explanation**
Best practices for memory management are simple: **let the stack do the work when you can**, **use smart pointers instead of raw `new`/`delete`**, **prefer standard containers over manual arrays**, **use `std::make_unique`/`std::make_shared`**, **respect alignment**, and **use pools or `std::pmr` when performance matters**. These habits prevent leaks, crashes, and fragmentation.

### Key Guidelines (Summary Table)

| Guideline | Summary | Recommendation |
|-----------|---------|----------------|
| **R.10** | Avoid `malloc()` and `free()`. | Use `new`/`delete` or, better, smart pointers. |
| **R.11** | Avoid calling `new` and `delete` explicitly. | Use `make_unique`, `make_shared`, containers. |
| **R.12** | Immediately give the result of an explicit resource allocation to a manager object. | RAII wrappers. |
| **R.13** | Perform at most one explicit resource allocation in a single expression statement. | Avoid leaks on exceptions. |
| **I.11** | Never transfer ownership by a raw pointer or reference. | Use smart pointers. |
| **F.43** | Never return a pointer or reference to a local object. | Return by value or use smart pointers. |
| **C.149** | Use `unique_ptr` or `shared_ptr` instead of `new`/`delete`. | Prefer RAII. |
| **Prefer stack** | Use automatic storage when lifetime is scoped. | Avoid unnecessary heap allocation. |
| **Alignment** | Respect `alignof`; use `alignas` for over-aligned types. | Aligned `operator new` (C++17). |
| **Pools/arenas** | Use pools for many small objects; arenas for phase-based allocation. | Mitigate fragmentation. |
| **`std::pmr`** | Use `std::pmr` for runtime-polymorphic allocation. | Avoid template explosion. |

### Annotated Code Examples

**Example 1: RAII instead of raw `new`/`delete`**

```cpp
#include <iostream>
#include <memory>

struct Widget {
    int value = 42;
    ~Widget() { std::cout << "Widget destroyed\n"; }
};

int main() {
    auto w = std::make_unique<Widget>();   // RAII: no manual delete
    std::cout << w->value << '\n';
}   // Widget automatically destroyed
```

**Expected Output**
```
42
Widget destroyed
```

**Why this is best practice?**  
`make_unique` guarantees cleanup even if an exception is thrown (R.11, R.12, C.149).

**Example 2: Arena for phase-based allocation**

```cpp
#include <memory_resource>
#include <vector>
#include <iostream>

int main() {
    std::pmr::monotonic_buffer_resource arena;
    {
        std::pmr::vector<int> v(&arena);
        for (int i = 0; i < 100; ++i) v.push_back(i);
        std::cout << v.size() << '\n';
    }   // vector destroyed; arena still holds memory
    arena.release();   // free all arena memory at once
}
```

**Expected Output**
```
100
```

**Why this is best practice?**  
The arena amortizes allocation cost and frees everything at once, avoiding fragmentation for phase-based workloads.

### Real-World Cases

- **Standard library** – containers and smart pointers follow these rules.
- **Code review** – Core Guidelines R.10–R.13, I.11, F.43, C.149 are common checkpoints.
- **Static analysis** – clang-tidy and MSVC Code Analysis flag raw `new`/`delete` and ownership issues.
- **Safety-critical systems** – pools and arenas provide deterministic memory behavior.

---

## References

- Storage duration (cppreference) – https://en.cppreference.com/w/cpp/language/storage_duration
- `new` expression (cppreference) – https://en.cppreference.com/w/cpp/language/new
- `delete` expression (cppreference) – https://en.cppreference.com/w/cpp/language/delete
- Placement `new` (cppreference) – https://en.cppreference.com/w/cpp/language/new#Placement_new
- `operator new` (cppreference) – https://en.cppreference.com/w/cpp/memory/new/operator_new
- `operator delete` (cppreference) – https://en.cppreference.com/w/cpp/memory/new/operator_delete
- `alignas` specifier (cppreference) – https://en.cppreference.com/w/cpp/language/alignas
- `alignof` operator (cppreference) – https://en.cppreference.com/w/cpp/language/alignof
- `std::allocator` (cppreference) – https://en.cppreference.com/w/cpp/memory/allocator
- Allocator-aware containers (cppreference) – https://en.cppreference.com/w/cpp/named_req/AllocatorAwareContainer
- `<memory_resource>` (cppreference) – https://en.cppreference.com/w/cpp/header/memory_resource
- `std::pmr::memory_resource` (cppreference) – https://en.cppreference.com/w/cpp/memory/memory_resource
- `std::pmr::monotonic_buffer_resource` (cppreference) – https://en.cppreference.com/w/cpp/memory/monotonic_buffer_resource
- `std::pmr::polymorphic_allocator` (cppreference) – https://en.cppreference.com/w/cpp/memory/polymorphic_allocator
- C++ Core Guidelines — R.10: Avoid `malloc()` and `free()` – https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rr-mallocfree
- C++ Core Guidelines — R.11: Avoid calling `new` and `delete` explicitly – https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rr-newdelete
- C++ Core Guidelines — R.12: Immediately give the result of an explicit resource allocation to a manager object – https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rr-immediate-alloc
- C++ Core Guidelines — R.13: Perform at most one explicit resource allocation in a single expression statement – https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rr-single-alloc
- C++ Core Guidelines — I.11: Never transfer ownership by a raw pointer or reference – https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Ri-raw
- C++ Core Guidelines — F.43: Never (directly or indirectly) return a pointer or a reference to a local object – https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rf-dangle
- C++ Core Guidelines — C.149: Use `unique_ptr` or `shared_ptr` instead of `new`/`delete` – https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rh-smart
- CERT C++ MEM30-C: Do not access freed memory – https://wiki.sei.cmu.edu/confluence/display/cplusplus/MEM30-C.+Do+not+access+freed+memory
- Microsoft Learn — `thread_local` – https://learn.microsoft.com/en-us/cpp/cpp/threadlocal
- Microsoft Learn — `alignas` and `alignof` – https://learn.microsoft.com/en-us/cpp/cpp/alignment-cpp-declarations
- cppreference — `std::pmr::vector` – https://en.cppreference.com/w/cpp/memory/vector
- cppreference — `std::pmr::string` – https://en.cppreference.com/w/cpp/string/basic_string
- ISO C++ Standard — `<memory_resource>` (C++17) – https://eel.is/c++draft/mem.res

---

This cheat sheet provides a complete, structured reference for C++ memory management, covering stack, heap, static, and thread-local memory; `new`/`delete`/`new[]`/`delete[]`; placement `new`; `operator new`/`operator delete`; leaks, fragmentation, alignment; pools, arenas, custom allocators; `std::allocator`, allocator-aware containers, and `std::pmr`; and best practices. Each section follows a uniform template with definitions, purposes, syntax rules, annotated examples, expected outputs, and authoritative references.