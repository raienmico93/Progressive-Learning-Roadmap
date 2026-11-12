# CRTP (Curiously Recurring Template Pattern) — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
The Curiously Recurring Template Pattern (CRTP) is a C++ idiom in which a class `X` derives from a class template `Y`, taking a template parameter `Z`, where `Y` is instantiated with `Z = X`.

**Technical Definition**
CRTP is a technique in which a class `Derived` inherits from a class template `Base<Derived>`, passing itself as the template argument to its own base. This allows the base class to `static_cast` `this` to `Derived*` and call derived-class methods at compile time, providing **static polymorphism** without virtual dispatch overhead. The base class "knows" the exact derived type at compile time because it is instantiated with that type as a template parameter. CRTP is a C++98 idiom; since C++23, an alternative syntax using deducing-this (`this auto&& self`) is available.

**Beginner-Friendly Explanation**
CRTP is a **C++ trick where a class inherits from a template that is itself**. It sounds like a tongue-twister, but the idea is simple: by passing yourself as a template argument to your base class, you let the base class know **exactly what type you are** at compile time. This lets the base class call your methods directly—no virtual functions, no vtable lookups, no runtime overhead. It is like giving your base class a **direct phone line** to your specific implementation instead of a general switchboard. CRTP is used for static polymorphism, mixins (adding features like comparison operators or logging to any class), and per-type instance counting.

### Key Characteristics

- **Self-inheritance pattern**: `class Derived : public Base<Derived>` — the derived class is the template argument to its base.
- **Compile-time type knowledge**: The base class knows the derived type at instantiation.
- **Static polymorphism**: Derived methods are called via `static_cast<Derived*>(this)`, resolved at compile time.
- **Zero runtime overhead**: No vtable, no indirect calls, enabling full inlining.
- **Mixin mechanism**: CRTP base classes inject reusable functionality (operators, logging, counting) into derived classes.
- **No runtime polymorphism**: CRTP cannot be used to store different derived types in a single container of base pointers; each `Base<Derived>` is a distinct type.
- **C++23 alternative**: Deducing-this (`this auto&& self`) provides a cleaner syntax for the same pattern.

### Prerequisites

- Solid understanding of templates (class templates and template parameters).
- Familiarity with inheritance and `static_cast`.
- Knowledge of virtual functions (for comparison with static polymorphism).
- Basic awareness of mixins and the Curiously Recurring Template Pattern.

### Related Programming Areas

- **Static polymorphism** – CRTP is the primary C++ mechanism for compile-time polymorphism.
- **Mixins** – CRTP base classes compose behavior without virtual dispatch.
- **Expression templates** – CRTP enables compile-time expression trees.
- **Operator generation** – CRTP can derive `!=`, `>`, `<=`, `>=` from `==` and `<`.
- **Instance counting** – CRTP gives each derived class its own static counter.
- **Clone pattern** – CRTP automates `clone()` in polymorphic hierarchies.

### Core Concepts / Features

1. CRTP (Curiously Recurring Template Pattern)
2. Static Polymorphism
3. Mixins
4. CRTP Best Practices
5. CRTP Examples (and Pitfalls)

---

## 1. CRTP (Curiously Recurring Template Pattern)

### Definitions

**Core Definition**
CRTP is an idiom where a class derives from a template instantiated with itself.

**Technical Definition**
The pattern is defined as `template<class Z> class Y {}; class X : public Y<X> {};`. The base class template `Y` is instantiated with `X` as its template argument `Z`, so `Y<X>` has access to `X`'s type at compile time. This enables `Y` to call `X`'s methods through `static_cast<X*>(this)`, providing **static dispatch**.

**Beginner-Friendly Explanation**
CRTP is **“inheriting from a template that uses you as a parameter.”** The base class gets a compile-time guarantee about the derived type, so it can call derived methods directly.

### Purposes

- To implement **static polymorphism** (compile-time dispatch without virtual functions).
- To create **mixin classes** that inject reusable functionality into derived classes.
- To enable **per-type static counters** (instance counting).
- To automate operator generation (e.g., deriving `!=` from `==`).
- To implement the **clone pattern** in polymorphic hierarchies.
- To build **expression templates** for compile-time optimization.

### Syntax Rules and Structure

**General Syntax (Traditional CRTP)**
```cpp
template <typename Derived>
class Base {
public:
    void interface() {
        static_cast<Derived*>(this)->implementation();   // static dispatch
    }
};

class Derived : public Base<Derived> {
public:
    void implementation() { /* ... */ }
};
```

**C++23 Alternative (Deducing-This)**
```cpp
struct Base {
    void interface(this auto&& self) {
        self.implementation();   // deducing-this: no CRTP template needed
    }
};

struct Derived : Base {
    void implementation() { /* ... */ }
};
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `template <typename Derived>` | Base class template parameterized by the derived type. |
| `class Base` | The CRTP base providing shared functionality. |
| `class Derived : public Base<Derived>` | Derived class passing itself to the base. |
| `static_cast<Derived*>(this)` | Downcast to the derived type (safe because `Base<Derived>` is instantiated with `Derived`). |

**Syntax Rules**
- The derived class must pass **itself** as the template argument: `public Base<Derived>`.
- The base class must use `static_cast<Derived*>(this)` to access derived members.
- The base class's constructor should be **private** with `friend Derived` (best practice) to prevent accidental instantiation of `Base` with the wrong type.
- CRTP does **not** support runtime polymorphism; `Base<Derived>` and `Base<Other>` are unrelated types.

**Constraints and Limitations**
- Cannot store different `Derived` types in a single container of `Base*` (no inclusion polymorphism).
- Object slicing occurs if a `Derived` is passed by value as a `Base<Derived>` (the base subobject is copied without the derived part).
- The base class is instantiated once per derived type; code bloat is possible.
- CRTP base constructors must be protected or private (with friendship) to enforce correct usage.

### Annotated Code Examples

**Example 1: Basic CRTP static polymorphism**

```cpp
#include <iostream>

template <typename Derived>
class Shape {
public:
    double area() const {
        return static_cast<const Derived*>(this)->area_impl();
    }
    void describe() const {
        std::cout << "Area: " << area() << '\n';
    }
};

class Circle : public Shape<Circle> {
    double radius_;
public:
    explicit Circle(double r) : radius_(r) {}
    double area_impl() const { return 3.14159 * radius_ * radius_; }
};

class Square : public Shape<Square> {
    double side_;
public:
    explicit Square(double s) : side_(s) {}
    double area_impl() const { return side_ * side_; }
};

int main() {
    Circle c(2.0);
    Square s(3.0);
    c.describe();   // uses Circle::area_impl via static dispatch
    s.describe();   // uses Square::area_impl via static dispatch
}
```

**Expected Output**
```
Area: 12.5664
Area: 9
```

**Why this output?**  
`Shape<Circle>::area()` casts `this` to `Circle*` and calls `area_impl()`. The dispatch is resolved at compile time with zero virtual call overhead.

**Example 2: CRTP instance counter**

```cpp
#include <iostream>

template <typename Derived>
class InstanceCounter {
    static inline int count_ = 0;   // per-derived-type counter
public:
    InstanceCounter()  { ++count_; }
    ~InstanceCounter() { --count_; }
    static int count() { return count_; }
};

class Widget : public InstanceCounter<Widget> {};
class Gadget : public InstanceCounter<Gadget> {};

int main() {
    std::cout << Widget::count() << ' ' << Gadget::count() << '\n';   // 0 0
    {
        Widget w1, w2;
        Gadget g1;
        std::cout << Widget::count() << ' ' << Gadget::count() << '\n'; // 2 1
    }
    std::cout << Widget::count() << ' ' << Gadget::count() << '\n';   // 0 0
}
```

**Expected Output**
```
0 0
2 1
0 0
```

**Why this output?**  
Each CRTP instantiation `InstanceCounter<Widget>` and `InstanceCounter<Gadget>` has its own static `count_`, so each derived class gets an independent counter.

### Real-World Cases

- **Static polymorphism in shape hierarchies** – zero-overhead area/perimeter calculations.
- **Instance counting** – tracking object counts per type.
- **Clone pattern** – automating `clone()` in polymorphic hierarchies.
- **Expression templates** – compile-time expression trees in numerical libraries.
- **Singleton base** – templated singleton pattern.

---

## 2. Static Polymorphism

### Definitions

**Core Definition**
Static polymorphism is polymorphism resolved at compile time, typically through templates, CRTP, or overload resolution, with no runtime dispatch overhead.

**Technical Definition**
Static polymorphism via CRTP uses `static_cast<Derived*>(this)` in a CRTP base to call derived-class methods directly. The compiler resolves the call at compile time because the base is instantiated with the exact derived type. This avoids the vtable and indirect call overhead of virtual functions, enabling inlining and optimization. CRTP cannot, however, provide runtime polymorphism: `Base<Derived>` and `Base<Other>` are distinct types with no common base.

**Beginner-Friendly Explanation**
Static polymorphism means **the compiler already knows which function to call** — no runtime guessing. CRTP achieves this by giving the base class a compile-time reference to the derived type.

### Purposes

- To eliminate virtual dispatch overhead (no vtable, no indirect calls).
- To enable inlining and aggressive optimization.
- To provide polymorphic-like interfaces without runtime cost.
- To support performance-critical code (numerical computing, game engines, embedded systems).

### Syntax Rules and Structure

**CRTP Dispatch Mechanism**
```cpp
template <typename Derived>
class Base {
public:
    void interface() {
        static_cast<Derived*>(this)->implementation();   // compile-time dispatch
    }
};

class Derived : public Base<Derived> {
public:
    void implementation() { /* ... */ }
};
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `Base<Derived>` | Base instantiated with the exact derived type. |
| `static_cast<Derived*>(this)` | Downcast to derived (safe because of instantiation). |
| `implementation()` | Derived method called statically. |

**Syntax Rules**
- The cast is **safe** because `Base<Derived>` is instantiated with `Derived` at compile time.
- The derived method must be accessible (typically `public` or `protected`).
- CRTP static polymorphism **does not** work through base-class pointers or references to a common base.

**Constraints and Limitations**
- **No inclusion polymorphism**: Cannot store `Derived1` and `Derived2` in a `std::vector<Base*>`.
- **Object slicing**: Passing a `Derived` by value as a `Base<Derived>` slices off the derived part.
- **Code bloat**: Each `Base<Derived>` instantiation generates separate code.

### Annotated Code Examples

**Example 1: Zero-overhead static dispatch**

```cpp
#include <iostream>
#include <chrono>

template <typename Derived>
class Task {
public:
    void run() {
        auto start = std::chrono::high_resolution_clock::now();
        static_cast<Derived*>(this)->execute();
        auto end = std::chrono::high_resolution_clock::now();
        std::cout << "Elapsed: "
                  << std::chrono::duration_cast<std::chrono::microseconds>(end - start).count()
                  << "us\n";
    }
};

class FastTask : public Task<FastTask> {
public:
    void execute() { volatile int sum = 0; for (int i = 0; i < 1000; ++i) sum += i; }
};

int main() {
    FastTask t;
    t.run();   // execute() inlined; no virtual call
}
```

**Expected Output** (timing varies)
```
Elapsed: 0us
```

**Why this output?**  
`run()` casts `this` to `FastTask*` and calls `execute()` statically. The compiler can inline `execute()` into `run()`, eliminating call overhead.

### Real-World Cases

- **Numerical libraries** – expression templates for vector/matrix operations.
- **Game engines** – component systems with compile-time dispatch.
- **Embedded systems** – no vtable overhead.

---

## 3. Mixins

### Definitions

**Core Definition**
A mixin is a class that provides a set of operations to a derived class, composing behavior without the overhead of virtual dispatch. CRTP is the classic C++ mixin mechanism.

**Technical Definition**
CRTP mixins work by templating a base class on the derived type, so the base can call derived methods statically. This allows injecting reusable functionality (operators, logging, serialization, counting) into any derived class that inherits from the mixin. Multiple mixins can be composed via multiple inheritance without the diamond problem, because each mixin is templated on the derived type.

**Beginner-Friendly Explanation**
A mixin is a **feature you can add to a class**. Want comparison operators? Inherit from a `Comparable` mixin. Want logging? Inherit from a `Loggable` mixin. CRTP makes these mixins work **without virtual functions**, so they are fast and composable.

### Purposes

- To inject reusable functionality into many classes without duplication.
- To compose behavior from multiple independent mixins.
- To avoid virtual dispatch overhead for common operations.
- To automate operator generation (e.g., deriving `!=` from `==`).
- To add logging, serialization, counting, or comparison to any class.

### Syntax Rules and Structure

**General Syntax (Comparable Mixin)**
```cpp
template <typename Derived>
struct Comparable {
    friend bool operator>(const Derived& a, const Derived& b) { return b < a; }
    friend bool operator<=(const Derived& a, const Derived& b) { return !(b < a); }
    friend bool operator>=(const Derived& a, const Derived& b) { return !(a < b); }
    friend bool operator!=(const Derived& a, const Derived& b) { return a < b || b < a; }
    friend bool operator==(const Derived& a, const Derived& b) { return !(a < b) && !(b < a); }
};

struct Temperature : Comparable<Temperature> {
    double celsius;
    friend bool operator<(const Temperature& a, const Temperature& b) {
        return a.celsius < b.celsius;
    }
};
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `template <typename Derived>` | Mixin template parameterized by derived type. |
| `Comparable<Derived>` | The mixin providing operators. |
| `friend` operators | Free functions generated by the mixin. |
| `Derived : Comparable<Derived>` | Derived class inherits the mixin. |

**Syntax Rules**
- The mixin is a class template parameterized by `Derived`.
- Operators are typically defined as `friend` functions for ADL.
- The derived class provides only the minimal required operation (e.g., `operator<`).
- Multiple mixins can be composed via multiple inheritance.

**Constraints and Limitations**
- Mixins increase code size (one instantiation per derived type).
- The derived class must satisfy the mixin's expectations (e.g., provide `equals()` for `EqualityComparable`).
- Mixin constructors should be protected or private to enforce correct usage.

### Annotated Code Examples

**Example 1: Comparable mixin**

```cpp
#include <iostream>

template <typename Derived>
struct Comparable {
    friend bool operator>(const Derived& a, const Derived& b) { return b < a; }
    friend bool operator<=(const Derived& a, const Derived& b) { return !(b < a); }
    friend bool operator>=(const Derived& a, const Derived& b) { return !(a < b); }
    friend bool operator!=(const Derived& a, const Derived& b) { return a < b || b < a; }
    friend bool operator==(const Derived& a, const Derived& b) { return !(a < b) && !(b < a); }
};

struct Temperature : Comparable<Temperature> {
    double celsius;
    friend bool operator<(const Temperature& a, const Temperature& b) {
        return a.celsius < b.celsius;
    }
};

int main() {
    Temperature boiling{100.0}, freezing{0.0}, body{36.6};
    std::cout << std::boolalpha
              << (boiling > freezing) << '\n'   // true
              << (freezing >= body) << '\n'     // false
              << (body == body) << '\n'          // true
              << (body != boiling) << '\n';      // true
}
```

**Expected Output**
```
true
false
true
true
```

**Why this output?**  
`Temperature` defines only `operator<`. The `Comparable` mixin generates `>`, `<=`, `>=`, `==`, and `!=` from it, all resolved at compile time.

**Example 2: EqualityComparable mixin**

```cpp
#include <iostream>
#include <cmath>

template <typename Derived>
class EqualityComparable {
public:
    friend bool operator==(const Derived& lhs, const Derived& rhs) {
        return lhs.equals(rhs);
    }
    friend bool operator!=(const Derived& lhs, const Derived& rhs) {
        return !lhs.equals(rhs);
    }
};

class Point : public EqualityComparable<Point> {
    double x_, y_;
public:
    Point(double x, double y) : x_(x), y_(y) {}
    bool equals(const Point& other) const {
        constexpr double eps = 1e-9;
        return std::abs(x_ - other.x_) < eps && std::abs(y_ - other.y_) < eps;
    }
};

int main() {
    Point a(1.0, 2.0), b(1.0, 2.0), c(3.0, 4.0);
    std::cout << std::boolalpha
              << (a == b) << '\n'   // true
              << (a != c) << '\n';   // true
}
```

**Expected Output**
```
true
true
```

**Why this output?**  
`Point` implements only `equals()`. The `EqualityComparable` mixin generates `==` and `!=` by calling `equals()` through the CRTP base.

### Real-World Cases

- **Operator generation** – `Comparable`, `EqualityComparable`.
- **Logging mixins** – injecting logging into any class.
- **Instance counting** – per-type counters via CRTP.
- **Serialization mixins** – adding `serialize()` to any class.

---

## 4. CRTP Best Practices

### Definitions

**Core Definition**
CRTP best practices are guidelines for using CRTP safely, correctly, and maintainably.

**Technical Definition**
Key practices include: make the CRTP base constructor **private** and declare the derived class as a `friend` to prevent accidental instantiation of `Base` with the wrong type; use `static_cast<Derived*>(this)` only when the base is instantiated with the exact derived type; avoid object slicing by passing CRTP objects by reference or pointer; prefer CRTP for static polymorphism and mixins, and virtual functions for runtime polymorphism; and use C++23 deducing-this when available for cleaner syntax.

**Beginner-Friendly Explanation**
Best practices for CRTP are simple: **lock down the base constructor**, **be careful with object slicing**, **use CRTP for compile-time polymorphism**, **use virtual functions for runtime polymorphism**, and **prefer the C++23 syntax if you can**.

### Key Guidelines (Summary Table)

| Guideline | Summary | Recommendation |
|-----------|---------|----------------|
| **Private constructor + friend** | Prevent accidental `Base<Wrong>` instantiation. | `CRTP() = default; friend T;`. |
| **Safe `static_cast`** | The cast is safe only if `Base<Derived>` is instantiated with the correct `Derived`. | Ensure derived passes itself. |
| **Avoid object slicing** | Passing CRTP objects by value slices off the derived part. | Pass by reference or pointer. |
| **No runtime polymorphism** | CRTP cannot store different derived types in one container. | Use virtual functions for runtime polymorphism. |
| **Prefer C++23 deducing-this** | Cleaner syntax, no CRTP boilerplate. | Use when C++23 is available. |
| **Document mixin requirements** | Derived classes must provide certain methods (e.g., `equals()`). | Document in the mixin. |

### Annotated Code Examples

**Example 1: Private CRTP constructor with friendship**

```cpp
template <typename Derived>
class Base {
private:
    Base() = default;                 // prevent accidental instantiation
    friend Derived;                   // only Derived can construct Base
public:
    void interface() {
        static_cast<Derived*>(this)->implementation();
    }
};

class Correct : public Base<Correct> {
public:
    void implementation() { /* ... */ }
};

// class Wrong : public Base<Correct> {};   // error: Base<Correct> is private
// Base<int> b;                             // error: constructor is private
```

**Expected Output**  
Compiles only for `Correct`; `Wrong` and `Base<int>` are rejected at compile time.

**Why this is best practice?**  
Making the constructor private with `friend Derived` prevents accidental instantiation of `Base` with the wrong type, which would cause undefined behavior when `static_cast` is used.

### Real-World Cases

- **Library design** – CRTP bases should enforce correct usage.
- **Code review** – CRTP constructor accessibility is a common check (clang-tidy `bugprone-crtp-constructor-accessibility`).
- **Static analysis** – tools detect unsafe CRTP usage.

---

## 5. CRTP Examples (and Pitfalls)

### Definitions

**Core Definition**
CRTP examples demonstrate the pattern in action; CRTP pitfalls are the common mistakes and dangers associated with the idiom.

**Technical Definition**
CRTP examples include static polymorphism, mixins, instance counting, clone automation, and operator generation. Pitfalls include **object slicing** (passing a `Derived` by value as `Base<Derived>`), **unsafe instantiation** (constructing `Base<Wrong>` outside the intended derived class), **inability to use runtime polymorphism** (no common base for different derived types), and **code bloat** (each derived type instantiates the base separately).

**Beginner-Friendly Explanation**
CRTP examples show what the pattern can do; CRTP pitfalls show what can go wrong. The biggest dangers are **slicing** and **accidental misuse** of the base template.

### Common Pitfalls (Summary)

| Pitfall | Description | Prevention |
|---------|-------------|------------|
| **Object slicing** | Passing by value slices derived part. | Pass by reference/pointer. |
| **Unsafe instantiation** | `Base<Wrong>` constructed accidentally. | Private constructor + `friend Derived`. |
| **No runtime polymorphism** | Cannot store different derived types together. | Use virtual functions for runtime polymorphism. |
| **Code bloat** | One base instantiation per derived type. | Accept trade-off for performance. |
| **Wrong `static_cast`** | Casting to the wrong derived type is UB. | Ensure `Base<Derived>` is instantiated correctly. |

### Annotated Code Examples

**Example 1: Object slicing pitfall**

```cpp
// DO NOT WRITE THIS CODE
template <typename Derived>
struct Base { void interface() { static_cast<Derived*>(this)->impl(); } };

struct Derived : Base<Derived> { void impl() { /* ... */ } };

void process(Base<Derived> b) { b.interface(); }   // SLICING: b is a Base<Derived>, not Derived

int main() {
    Derived d;
    process(d);   // slices d; static_cast in interface() is undefined behavior
}
```

**Expected Output**  
Undefined behavior; may crash or misbehave.

**Why this is a pitfall?**  
`process` takes `Base<Derived>` by value, copying only the base subobject. Inside `interface()`, `static_cast<Derived*>(this)` casts a `Base<Derived>*` that does not actually point to a `Derived` object, causing undefined behavior.

**Example 2: Unsafe instantiation pitfall**

```cpp
// DO NOT WRITE THIS CODE
template <typename T>
class CRTP {
public:
    CRTP() = default;   // public constructor: unsafe
};

class Good : CRTP<Good> {};
CRTP<int> BadInstance;   // compiles, but wrong
```

**Expected Output**  
Compiles; `CRTP<int>` is instantiated with a non-derived type, which may cause undefined behavior if `static_cast` is used.

**Why this is a pitfall?**  
With a public constructor, anyone can instantiate `CRTP<Wrong>`, breaking the idiom. The best practice is to make the constructor private and `friend` the derived class.

### Real-World Cases

- **Slicing in APIs** – passing CRTP objects by value in function parameters.
- **Accidental instantiation** – `CRTP<int>` created in generic code.
- **Container limitations** – trying to store mixed CRTP types in one container.

---

## References

- Curiously Recurring Template Pattern (cppreference) — https://en.cppreference.com/w/cpp/language/crtp
- Know the Curiously Recurring Template Pattern (CRTP) and Its Uses (Dr-Sergey/learn_cpp) — https://github.com/Dr-Sergey/learn_cpp/blob/main/05_Templates_and_Generic_Programming/Know_the_Curiously_Recurring_Template_Pattern_CRTP_and_its_uses.md
- Use mixins through CRTP or inheritance to compose behavior (Dr-Sergey/learn_cpp) — https://github.com/Dr-Sergey/learn_cpp/blob/main/12_Modern_OOP_Patterns/Use_mixins_through_CRTP_or_inheritance_to_compose_behavior.md
- Implement the CRTP for static polymorphism and mixin classes (Dr-Sergey/learn_cpp) — https://github.com/Dr-Sergey/learn_cpp/blob/main/29_OOP_Design/Implement_the_CRTP_for_static_polymorphism_and_mixin_classes.md
- clang-tidy `bugprone-crtp-constructor-accessibility` — https://clang.llvm.org/extra/clang-tidy/checks/bugprone/crtp-constructor-accessibility.html
- Stack Overflow — Confusion about CRTP static polymorphism (object slicing) — https://stackoverflow.com/questions/43821542/confusion-about-crtp-static-polymorphism
- Stack Overflow — Object slicing when using CRTP — https://stackoverflow.com/questions/36319928/object-slicing-when-using-crtp
- Herb Sutter's Mill — `clonable` pattern and CRTP — https://herbsutter.com/2019/10/03/gotw-ish-solution-the-clonable-pattern/
- C++ Core Guidelines (templates and OOP sections) — https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines

---

This cheat sheet provides a complete, structured reference for CRTP in C++, covering the idiom itself, static polymorphism, mixins, best practices, examples, and pitfalls. Each section follows a uniform template with definitions, purposes, syntax rules, annotated examples, expected outputs, and authoritative references.