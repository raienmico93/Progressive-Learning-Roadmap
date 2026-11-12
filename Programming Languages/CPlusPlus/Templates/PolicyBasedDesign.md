# Policy-Based Design — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
Policy-based design (PBD) is a C++ template metaprogramming idiom in which a class's behavior is **parameterized by one or more policy classes** — small, focused types that encapsulate a single behavioral aspect — allowing the host class to be configured at compile time with zero runtime overhead.

**Technical Definition**
Policy-based design, popularized by Andrei Alexandrescu in *Modern C++ Design* (2001), is the practice of decomposing a class's behavior into **orthogonal policy axes** and supplying each axis as a template parameter. Each policy is a class (or class template) that implements a particular implicit interface (a **policy**) encapsulating some orthogonal or mostly orthogonal aspect of the host class's behavior. The host class composes its behavior from these policies, typically through **private inheritance** (the "implemented-in-terms-of" idiom) or composition, enabling the compiler to inline policy calls and eliminate virtual dispatch entirely. The resulting type `Host<PolicyA, PolicyB>` is a distinct type for each policy combination, and all policy behavior is resolved statically.

**Beginner-Friendly Explanation**
Policy-based design is a **“build-your-own-class” mechanism**. Instead of writing separate subclasses for every combination of behaviors (e.g., a logging class that can output to console or file, with full or short timestamps), you write small, independent **policy classes** — one for each behavior — and pass them as template arguments to a single host class. The host class “mixes in” the policies, producing a customized type at compile time. It is like ordering a custom sandwich: you pick the bread (output policy), the filling (format policy), and the sauce (threading policy), and the deli (compiler) assembles it instantly with no runtime overhead. The catch: you must choose your policies at compile time — you cannot swap them at runtime.

### Key Characteristics

- **Compile-time configuration**: Policies are selected via template arguments, fixed at compilation.
- **Zero runtime overhead**: Policy calls are statically resolved and can be inlined; no vtable or indirect calls.
- **Orthogonal decomposition**: Policies are designed to be independent and non-interfering, each handling a single aspect of behavior.
- **Class explosion avoidance**: Instead of $M \times N$ subclasses, PBD needs only $M + N$ policy classes and one host template.
- **Distinct types per combination**: `Host<A, B>` and `Host<C, D>` are unrelated types, enabling type-based optimization and safety checks.
- **Private inheritance for EBO**: Policies are often inherited privately to enable the **empty base optimization** (EBO) for stateless policies.
- **Compile-time Strategy pattern**: PBD is essentially the Strategy pattern resolved at compile time rather than runtime.

### Prerequisites

- Solid understanding of class templates and template parameters.
- Familiarity with multiple inheritance and private inheritance.
- Knowledge of the Empty Base Optimization (EBO).
- Basic awareness of the Strategy pattern and runtime polymorphism (for comparison).
- Understanding of template template parameters (often used for policies).

### Related Programming Areas

- **Template metaprogramming (TMP)** – PBD is a prominent TMP idiom.
- **Static polymorphism** – PBD achieves polymorphism without virtual functions.
- **Mixins** – PBD is closely related to (but not identical with) mixin-based composition.
- **CRTP** – often combined with PBD for static interfaces and policy composition.
- **Strategy pattern** – PBD is the compile-time counterpart of the runtime Strategy pattern.
- **Allocator model** – the standard library's allocator template parameter is a policy-like mechanism.

### Core Concepts / Features

1. Policy-Based Design
2. Policy Classes
3. Policy Composition
4. Policy-Based Design Best Practices

---

## 1. Policy-Based Design

### Definitions

**Core Definition**
Policy-based design is a class design paradigm that parameterizes behavior through template arguments (policies) selected at compile time.

**Technical Definition**
Policy-based design (Alexandrescu, *Modern C++ Design*) parameterizes class behavior through template arguments instead of inheritance. A host class template accepts one or more policy class templates (or types) as parameters, each representing an independent behavioral axis. The host class uses the policies via private inheritance or composition, and all policy operations are resolved statically, achieving zero-overhead customization. The design contrasts sharply with virtual inheritance, which requires runtime dispatch and leads to class explosion when behaviors multiply.

**Beginner-Friendly Explanation**
Policy-based design is **making a class's behavior a template parameter**. You write one template class with “slots” for behaviors (output, timing, allocation, threading), and the caller fills those slots with small policy classes. The compiler assembles the perfect class for that combination.

### Purposes

- To achieve **compile-time customization** of behavior without subclassing.
- To avoid **class explosion** (the combinatorial multiplication of subclasses).
- To provide **zero-overhead abstraction** by enabling inlining and eliminating vtables.
- To make behavioral choices **explicit and type-safe** in the type system.
- To encode design trade-offs as independent, reusable policy classes.
- To serve as a **compile-time Strategy pattern**, where the “strategy” is baked into the type.

### Syntax Rules and Structure

**General Syntax (Host Class with Policies)**
```cpp
template <
    typename OutputPolicy,
    typename TimestampPolicy,
    typename ThreadingPolicy = SingleThreaded
>
class Logger : private OutputPolicy, private TimestampPolicy, private ThreadingPolicy {
public:
    void log(const std::string& msg) {
        this->lock();
        this->write(this->timestamp() + msg);
        this->unlock();
    }
};
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `template <...>` | Template parameter list containing policy parameters. |
| `typename OutputPolicy` | A policy class (or template) parameter. |
| `class Logger : private OutputPolicy, ...` | Host class inheriting policies privately. |
| `this->write(...)` | Calling policy member functions statically. |

**Syntax Rules**
- Policies are passed as **template type parameters** (often template template parameters for policy templates).
- The host class typically inherits **privately** from each policy (the “implemented-in-terms-of” idiom) to enable EBO and avoid exposing policy members.
- Policy member functions are called via `this->policy_member()` (or qualified names if ambiguity exists).
- Policies should be **orthogonal** — independent and non-interfering.
- Default policies can be provided for convenience.

**Constraints and Limitations**
- **Compile-time selection only**: Policies cannot be swapped at runtime.
- **Distinct types**: Each policy combination produces a different type, preventing storage in a homogeneous container.
- **Long compile times**: Heavy template instantiation can slow compilation.
- **Binary size**: Multiple policy instantiations can increase code size.

### Annotated Code Examples

**Example 1: Policy-based Logger**

```cpp
#include <iostream>
#include <fstream>
#include <string>
#include <chrono>
#include <ctime>
#include <sstream>

// === Output Policies ===
struct ConsoleOutput {
    void write(const std::string& msg) { std::cout << msg << "\n"; }
};

struct FileOutput {
    std::ofstream file;
    FileOutput() : file("app.log", std::ios::app) {}
    void write(const std::string& msg) { file << msg << "\n"; }
};

// === Timestamp Policies ===
struct FullTimestamp {
    std::string timestamp() {
        auto now = std::chrono::system_clock::now();
        auto t = std::chrono::system_clock::to_time_t(now);
        char buf[64];
        std::strftime(buf, sizeof(buf), "%Y-%m-%d %H:%M:%S", std::localtime(&t));
        return std::string(buf) + " ";
    }
};

struct NoTimestamp {
    std::string timestamp() { return ""; }
};

// === Host Class ===
template <typename OutputPolicy, typename TimestampPolicy>
class Logger : private OutputPolicy, private TimestampPolicy {
public:
    void log(const std::string& msg) {
        this->write(this->timestamp() + msg);
    }
};

int main() {
    Logger<ConsoleOutput, FullTimestamp> console_logger;
    console_logger.log("Hello, policy world!");   // prints timestamped message

    Logger<FileOutput, NoTimestamp> file_logger;
    file_logger.log("Plain file message");         // writes to app.log

    // Logger<ConsoleOutput, FullTimestamp> and Logger<FileOutput, NoTimestamp>
    // are completely different types, each with zero runtime dispatch overhead.
}
```

**Expected Output** (console, timestamp varies)
```
2025-01-15 10:30:45 Hello, policy world!
```

**Why this output?**  
`Logger<ConsoleOutput, FullTimestamp>` inherits privately from both policies. `log()` calls `this->write()` and `this->timestamp()`, which resolve statically to the policies' implementations and are inlined. No virtual functions are involved. The file logger uses a different policy combination and is a distinct type.

**Example 2: Policy-based Smart Pointer**

```cpp
#include <iostream>
#include <mutex>

// Ownership policies
template <typename T>
struct UniqueOwnership {
    void acquire(T*) {}
    bool release(T* p) { delete p; return true; }
};

template <typename T>
struct RefCounted {
    int* count_ = nullptr;
    void acquire(T*) { if (!count_) count_ = new int(1); else ++(*count_); }
    bool release(T*) {
        if (count_ && --(*count_) == 0) { delete count_; count_ = nullptr; return true; }
        return false;
    }
};

// Checking policies
template <typename T>
struct NoCheck { void check(T*) const {} };

template <typename T>
struct AssertCheck {
    void check(T* p) const {
        if (!p) { std::cerr << "Null pointer dereference!\n"; std::abort(); }
    }
};

// Threading policies
struct SingleThreaded { void lock() {} void unlock() {} };
struct MultiThreaded {
    std::mutex mtx_;
    void lock() { mtx_.lock(); }
    void unlock() { mtx_.unlock(); }
};

// Policy-based smart pointer
template <
    typename T,
    template <typename> class OwnershipPolicy = UniqueOwnership,
    template <typename> class CheckPolicy = NoCheck,
    typename ThreadPolicy = SingleThreaded
>
class SmartPtr
    : private OwnershipPolicy<T>
    , private CheckPolicy<T>
    , private ThreadPolicy
{
    T* ptr_ = nullptr;
public:
    explicit SmartPtr(T* p = nullptr) : ptr_(p) {
        if (ptr_) OwnershipPolicy<T>::acquire(ptr_);
    }
    ~SmartPtr() { if (ptr_) OwnershipPolicy<T>::release(ptr_); }

    T& operator*() const {
        CheckPolicy<T>::check(ptr_);
        return *ptr_;
    }
    T* operator->() const {
        CheckPolicy<T>::check(ptr_);
        return ptr_;
    }
};

// Mix-and-match policies
struct Widget { int x = 42; };

using FastPtr        = SmartPtr<Widget, UniqueOwnership, NoCheck, SingleThreaded>;
using SafePtr        = SmartPtr<Widget, UniqueOwnership, AssertCheck, SingleThreaded>;
using SharedSafePtr  = SmartPtr<Widget, RefCounted, AssertCheck, MultiThreaded>;

int main() {
    FastPtr p1(new Widget());      // zero overhead
    SafePtr p2(new Widget());      // null-checked
    std::cout << p1->x << "\n";    // 42
    std::cout << (*p2).x << "\n";  // 42
}
```

**Expected Output**
```
42
42
```

**Why this output?**  
Each `SmartPtr` instantiation composes three policies. `FastPtr` uses `NoCheck` and `SingleThreaded` (zero overhead), while `SafePtr` uses `AssertCheck` (null checking) and `SharedSafePtr` uses `RefCounted` (reference counting) with `MultiThreaded`. All policy calls are statically resolved.

### Real-World Cases

- **Configurable logging** – output, formatting, and threading policies.
- **Smart pointers** – ownership, checking, and threading policies.
- **Allocators** – `std::vector<T, Allocator>` is a policy-like design (the allocator is a policy).
- **Character traits** – `std::char_traits` is a policy for character operations.
- **Policy-based containers** – customizing storage, comparison, and allocation.

---

## 2. Policy Classes

### Definitions

**Core Definition**
A policy class is a small, focused class (or class template) that encapsulates a single behavioral aspect of a host class, implementing an implicit interface (the policy) that the host class relies on.

**Technical Definition**
A policy class represents a **policy** — a set of related functions (and optionally state) that the host class uses to implement one aspect of its behavior. Policies differ from traits: traits describe a type via typedefs and values, while policies represent behavior via member functions. Policies are often class templates (e.g., `template<typename T> struct OwnershipPolicy`) or template template parameters, allowing them to be parameterized by the host's type.

**Beginner-Friendly Explanation**
A policy class is a **small behavior module**. It provides functions (like `write()` or `lock()`) that the host class calls. It has no knowledge of the host class; it just implements its small piece of behavior. You can write many policy classes and mix them freely.

### Purposes

- To encapsulate a single, orthogonal behavioral aspect.
- To provide a reusable building block for host classes.
- To enforce a clear, statically checked interface (the policy).
- To enable compile-time behavior selection without virtual functions.
- To allow independent evolution of each behavioral axis.

### Syntax Rules and Structure

**General Syntax (Policy Class)**
```cpp
template <typename T>
struct OwnershipPolicy {
    void acquire(T* p);
    bool release(T* p);
};
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `template <typename T>` | Policy is often a class template parameterized by the host's type. |
| `struct OwnershipPolicy` | The policy class name. |
| `void acquire(T*);` | A required operation (part of the policy interface). |
| `bool release(T*);` | Another required operation. |

**Syntax Rules**
- A policy must implement the **implicit interface** expected by the host class (e.g., `write()`, `timestamp()`, `lock()`).
- Policies should be **stateless whenever possible** to enable EBO and avoid size overhead.
- Policies can have state (e.g., a `std::mutex` in a threading policy) but must manage it carefully.
- Policies are often passed as **template template parameters** or as type parameters.

**Constraints and Limitations**
- A policy's interface is **implicit** — it is not enforced by the type system until the host class is instantiated.
- Policies should not depend on each other (orthogonality).
- Policy classes with state may increase the size of the host object.

### Annotated Code Examples

**Example 1: Independent Policy Classes**

```cpp
// Output policy
struct ConsoleOutput {
    void write(const std::string& msg) { std::cout << msg << "\n"; }
};

// Format policy
struct JsonFormat {
    std::string format(const std::string& msg) {
        return R"({"message": ")" + msg + R"("})";
    }
};

// The host class composes them
template <typename Output, typename Format>
class Formatter : private Output, private Format {
public:
    void emit(const std::string& msg) {
        this->write(this->format(msg));
    }
};

int main() {
    Formatter<ConsoleOutput, JsonFormat> f;
    f.emit("hello");   // prints {"message": "hello"}
}
```

**Expected Output**
```
{"message": "hello"}
```

**Why this output?**  
`Formatter` privately inherits from `ConsoleOutput` and `JsonFormat`. `emit()` calls `this->format()` then `this->write()`, both resolved statically.

### Real-World Cases

- **`std::char_traits`** – a policy class for character operations.
- **Allocator policies** – `std::allocator` is a policy-like class.
- **Comparison policies** – `std::less`, `std::greater` as policy classes.
- **Threading policies** – `SingleThreaded`, `MultiThreaded`.

---

## 3. Policy Composition

### Definitions

**Core Definition**
Policy composition is the technique of combining multiple policy classes in a single host class, typically through **multiple (private) inheritance**, to produce a class whose behavior is the union of the selected policies.

**Technical Definition**
Policy composition assembles a host class from several orthogonal policies, each controlling a distinct behavioral axis. The host class inherits privately from each policy (or contains them as members), enabling the empty base optimization for stateless policies and avoiding the overhead of virtual inheritance. Because each policy is independent, the number of combinations grows multiplicatively while the number of policy classes grows additively — solving the “class explosion” problem. The key design requirement is **orthogonal decomposition**: policies must be independent and non-interfering.

**Beginner-Friendly Explanation**
Policy composition is **mixing and matching behavior modules**. You have a box of output policies, a box of format policies, and a box of threading policies. You pick one from each box, and the host class combines them. Three output policies and three format policies give you nine combinations with only six policy classes and one host template.

### Purposes

- To combine independent behavioral aspects into a single class.
- To avoid the combinatorial explosion of subclasses.
- To enable fine-grained customization of host behavior.
- To leverage EBO for zero-size overhead of stateless policies.
- To keep each policy focused and reusable.

### Syntax Rules and Structure

**General Syntax (Multiple Policy Composition)**
```cpp
template <
    typename OutputPolicy,
    typename FormatPolicy,
    typename ThreadingPolicy = SingleThreaded
>
class Host
    : private OutputPolicy
    , private FormatPolicy
    , private ThreadingPolicy
{
public:
    void run(const std::string& msg) {
        this->lock();
        this->write(this->format(msg));
        this->unlock();
    }
};
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `template <...>` | Policy parameters for each axis. |
| `private OutputPolicy` | Private inheritance from the output policy. |
| `private FormatPolicy` | Private inheritance from the format policy. |
| `private ThreadingPolicy` | Private inheritance from the threading policy. |
| `this->lock()` | Calling a policy operation. |

**Syntax Rules**
- Use **private inheritance** to compose stateless policies (enables EBO).
- Use **composition** (member objects) when policies have state or when inheritance is inappropriate.
- Policies should be **orthogonal** — they must not depend on each other's behavior.
- If a policy operation is ambiguous (two policies provide the same name), use qualified names (`OutputPolicy::write(...)`).
- Provide **default policy arguments** for convenience.

**Constraints and Limitations**
- Orthogonality is an ideal; some policies may have subtle interactions.
- Ambiguity can arise when multiple policies define the same member name.
- Private inheritance does not allow the host to be used as a policy type externally.

### Annotated Code Examples

**Example 1: Composing Three Policies**

```cpp
#include <iostream>
#include <mutex>

// Policies
struct ConsoleOutput { void write(const std::string& m) { std::cout << m << "\n"; } };
struct FileOutput    { void write(const std::string& m) { /* write to file */ } };

struct PlainFormat   { std::string format(const std::string& m) { return m; } };
struct JsonFormat    { std::string format(const std::string& m) { return R"({"msg":")" + m + R"("})"; } };

struct SingleThreaded { void lock() {} void unlock() {} };
struct MultiThreaded  { std::mutex m; void lock() { m.lock(); } void unlock() { m.unlock(); } };

// Host class composing three policies
template <typename Output, typename Format, typename Threading>
class Engine : private Output, private Format, private Threading {
public:
    void emit(const std::string& msg) {
        this->lock();
        this->write(this->format(msg));
        this->unlock();
    }
};

int main() {
    Engine<ConsoleOutput, JsonFormat, SingleThreaded> e;
    e.emit("hello");   // prints {"msg":"hello"}
}
```

**Expected Output**
```
{"msg":"hello"}
```

**Why this output?**  
`Engine` composes three policies. `emit()` locks, formats, and writes through statically dispatched policy calls.

### Real-World Cases

- **Logger with output + format + threading policies**.
- **Smart pointer with ownership + checking + threading policies**.
- **Configurable containers** – storage, comparison, and allocation policies.

---

## 4. Policy-Based Design Best Practices

### Definitions

**Core Definition**
Policy-based design best practices are guidelines for applying PBD effectively: decomposing behavior orthogonally, designing clean policy interfaces, choosing between inheritance and composition, and avoiding common pitfalls.

**Technical Definition**
Best practices include: identify **orthogonal behavioral axes** and separate them into distinct policies; use **private inheritance** for stateless policies (EBO) and **composition** for stateful policies; define **minimal, focused policy interfaces**; provide **default policies**; document the implicit policy interface; prefer **compile-time policies** for performance-critical customization and **runtime strategies** for dynamic switching; and guard against class explosion by combining PBD with **type erasure** when runtime flexibility is also needed.

**Beginner-Friendly Explanation**
Best practices for PBD are simple: **separate behaviors cleanly**, **keep policies small and independent**, **use private inheritance for zero-size policies**, **provide sensible defaults**, and **know when to use runtime polymorphism instead**.

### Key Guidelines (Summary Table)

| Guideline | Summary | Recommendation |
|-----------|---------|----------------|
| **Orthogonal decomposition** | Separate behaviors into independent policies. | Each policy handles one axis. |
| **Minimal policy interface** | Policies should expose only what the host needs. | `write()`, `timestamp()`, `lock()`. |
| **Private inheritance for stateless policies** | Enable empty base optimization. | `class Host : private Policy`. |
| **Composition for stateful policies** | Avoid inheritance when policies hold state. | `Policy policy_;` as a member. |
| **Provide defaults** | Make policies optional with sensible defaults. | `typename P = DefaultPolicy`. |
| **Document policy contracts** | The implicit interface is not enforced by the compiler. | Comments and static assertions. |
| **Avoid over-decomposition** | Too many policies increase complexity. | Group closely related behaviors. |
| **Know when to use runtime polymorphism** | PBD cannot switch at runtime. | Use Strategy pattern for dynamic needs. |
| **Combine with type erasure** | For runtime flexibility in compile-time-designed components. | `std::function`, custom type erasure. |

### Annotated Code Examples

**Example 1: Private Inheritance for EBO**

```cpp
#include <iostream>

struct EmptyPolicy {
    void do_something() const { /* no state */ }
};

template <typename Policy>
class Host : private Policy {   // private inheritance enables EBO
    int data_;
public:
    Host() : data_(0) {}
    void run() { this->do_something(); }
};

int main() {
    std::cout << sizeof(Host<EmptyPolicy>) << '\n';   // likely 4 (only int), not 8
}
```

**Expected Output**
```
4
```

**Why this is best practice?**  
Private inheritance from an empty policy allows the **empty base optimization**, so the policy occupies no extra bytes. Composition would add at least one byte.

### Real-World Cases

- **Standard library** – `std::vector<T, Allocator>` follows policy-like design principles.
- **Loki library** – Alexandrescu's library of policy-based components.
- **Modern libraries** – use PBD for configurable allocators, loggers, and smart pointers.

---

## 5. Policy-Based Design vs. Related Techniques

### Definitions

**Core Definition**
Policy-based design is often compared with virtual inheritance (runtime polymorphism), the Strategy pattern, mixins, and CRTP. Each has distinct trade-offs in flexibility, performance, and complexity.

**Technical Definition**
PBD selects behavior at **compile time** via template parameters; virtual inheritance and the Strategy pattern select behavior at **runtime** via vtables or type-erased callables. Mixins are a broader concept of composing class functionality; PBD is a specific C++ idiom for achieving mixin-like composition with orthogonal policies. CRTP is a static-polymorphism mechanism often used to implement policy-based static interfaces.

**Beginner-Friendly Explanation**
PBD is the **compile-time version of the Strategy pattern**. It is faster (no virtual calls) but less flexible (no runtime switching). Mixins are a general idea; PBD is a C++ way to implement them. CRTP is a tool often used alongside PBD.

### Comparison Table

| Aspect | Policy-Based Design | Virtual Inheritance / Strategy | Mixins (general) | CRTP |
|--------|---------------------|-------------------------------|------------------|------|
| **Selection time** | Compile time | Runtime | Compile time | Compile time |
| **Mechanism** | Template parameters | Virtual functions / `std::function` | Class templates | Self-inheritance |
| **Overhead** | Zero (inlined) | Vtable + indirection | Zero | Zero |
| **Flexibility** | Fixed at compile time | Switchable at runtime | Fixed at compile time | Fixed at compile time |
| **Type identity** | Each combination is a distinct type | Single type | Distinct types | Distinct types |
| **Class explosion** | Composable (M+N) | M×N subclasses | Composable | Composable |
| **Primary use** | Configurable components | Dynamic behavior | Reusable functionality | Static interfaces |

### Annotated Code Examples

**Example 1: PBD vs. Runtime Strategy**

```cpp
// PBD: compile-time strategy
template <typename Strategy>
void execute_pbd() { Strategy::run(); }

// Runtime strategy
struct IStrategy { virtual void run() = 0; virtual ~IStrategy() = default; };
void execute_runtime(IStrategy& s) { s.run(); }
```

**Expected Output**  
(No output; illustrates the two approaches.)

**Why this matters?**  
`execute_pbd` resolves `Strategy::run()` at compile time (zero overhead); `execute_runtime` uses a virtual call (runtime dispatch).

### Real-World Cases

- **Embedded systems** – prefer PBD for zero overhead.
- **Plugin architectures** – require runtime strategy (virtual functions).
- **Standard library** – uses both (allocator policies and virtual destructors).

---

## 6. Policy-Based Design Pitfalls

### Common Pitfalls (Summary)

| Pitfall | Description | Prevention |
|---------|-------------|------------|
| **Non-orthogonal policies** | Policies depend on each other. | Careful decomposition; document dependencies. |
| **Interface mismatch** | A policy lacks a required member. | Document the policy contract; use `static_assert`. |
| **Ambiguous member names** | Multiple policies define the same name. | Use qualified names or rename. |
| **Object slicing** | Passing PBD objects by value slices policies. | Pass by reference or pointer. |
| **Code bloat** | Many policy combinations instantiate much code. | Accept trade-off or use type erasure. |
| **Compile-time explosion** | Heavy template instantiation slows builds. | Limit policy combinations; use precompiled headers. |
| **Forgetting default policies** | Host requires all policies explicitly. | Provide sensible defaults. |

### Annotated Code Examples

**Example 1: Non-Orthogonal Policies (Pitfall)**

```cpp
// BAD: FormatPolicy depends on OutputPolicy's behavior
struct JsonFormat {
    std::string format(const std::string& msg) {
        // assumes output will add a newline — violates orthogonality
        return R"({"msg":")" + msg + R"("})";
    }
};
```

**Expected Output**  
(No output; illustrates a design smell.)

**Why this is a pitfall?**  
`JsonFormat` implicitly assumes the output policy's behavior, violating orthogonality. Policies should be independent.

### Real-World Cases

- **Ambiguous member names** – two policies defining `write()`.
- **Slicing** – passing a PBD object by value in an API.
- **Code bloat** – many policy instantiations in large projects.

---

## References

- Apply the policy-based design pattern with templates (Dr-Sergey/learn_cpp) — https://github.com/Dr-Sergey/learn_cpp/blob/6642b53a22d13b9812d2ca5cf39a63b34cc70e2c/12_Modern_OOP_Patterns/Apply_the_policy-based_design_pattern_with_templates.md
- Design policy-based classes using template parameters (Dr-Sergey/learn_cpp) — https://github.com/Dr-Sergey/learn_cpp/blob/main/29_OOP_Design/Design_policy-based_classes_using_template_parameters.md
- Design policy-based classes using template parameters (raw) — https://raw.githubusercontent.com/Dr-Sergey/learn_cpp/a285937786f8ea542f74de40134f69efeaa45174/29_OOP_Design/Design_policy-based_classes_using_template_parameters.md
- Are Mixins in C++ simply Policy-Oriented Design? (Stack Overflow) — https://stackoverflow.com/questions/79582964/are-mixins-in-c-simply-policy-oriented-design-or-is-there-more-to-them
- CopperSpice: Template Design With Policy Classes (isocpp.org) — https://isocpp.org/blog/2024/09/copperspice-template-design-with-policy-classes
- Overload Journal #57 — Policy-Based Design (ACCU) — https://www.accu.org/journals/overload/11/57/overload57.pdf
- Policy-Based Design — Compile-Time Strategy Pattern (nimbustan/cpp_mastery) — https://raw.githubusercontent.com/nimbustan/cpp_mastery/refs/heads/master/EN/module_10_design_patterns/05_testable_design.cpp
- 基於原則設計 (Wikipedia zh) — https://browse.library.kiwix.org/content/wikipedia_zh_all_maxi/基於原則設計
- Unlocking flexibility and reusability in C++ with policy-based design (Kickmaker) — https://www.kickmaker.fr/2024/06/11/policy-based-design/
- Modern C++ Design — Andrei Alexandrescu (book reference)
- Policy based design and best practices (C++) (KiwiX) — https://browse.library.kiwix.org/content/stackoverflow.com_en_all_2023-11/questions/1184399/policy-based-design-and-best-practices-c

---

This cheat sheet provides a complete, structured reference for Policy-Based Design in C++, covering its definition, policy classes, policy composition, comparisons with related techniques, best practices, and pitfalls. Each section follows a uniform template with definitions, purposes, syntax rules, annotated examples, expected outputs, and authoritative references.