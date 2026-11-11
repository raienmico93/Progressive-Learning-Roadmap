# Abstract Classes and Interfaces — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
An **abstract class** is a class that cannot be instantiated because it contains at least one **pure virtual function**; it defines a contract that derived classes must fulfill. An **interface class** is an abstract class with only pure virtual functions (and a virtual destructor), representing a pure contract with no implementation.

**Technical Definition**
A class is abstract if it has at least one *pure virtual function* (`virtual f() = 0;`). Abstract classes cannot be instantiated directly, but pointers and references to them can be used polymorphically. An **interface class** is an abstract class with no non-static data members, no non-virtual member functions (except the destructor), and only pure virtual functions; it defines a pure interface. Interfaces are commonly combined through **multiple inheritance**; **virtual inheritance** is used when a shared base must appear only once in a hierarchy (the diamond problem). The C++ Core Guidelines recommend using abstract classes as interfaces (C.122) and using class hierarchies for inherent hierarchical concepts (C.120).

**Beginner-Friendly Explanation**
An abstract class is a **blueprint that is intentionally incomplete** — it says “every derived class must fill in these missing pieces.” You cannot create an object of an abstract class directly; you must derive from it and implement the missing functions. An **interface** is the purest form of an abstract class: it has **only** missing pieces (pure virtual functions) and no data — it is a pure contract. Think of an interface as a job description: it lists the duties (functions) but provides no worker (implementation). Any class that “takes the job” must implement every duty. Multiple inheritance lets a class take on several job descriptions at once, and virtual inheritance ensures that if two job descriptions share a common requirement, it only counts once.

### Key Characteristics

- **Cannot be instantiated**: Abstract classes have no objects of their own type.
- **Pure virtual functions**: `= 0` declares a function that derived classes must override.
- **Polymorphic use**: Pointers and references to abstract classes support dynamic dispatch.
- **Interfaces are pure contracts**: No data, only pure virtual functions and a virtual destructor.
- **Multiple inheritance**: A class can implement multiple interfaces.
- **Virtual inheritance**: Shares a single base subobject across multiple inheritance paths.
- **Virtual destructor required**: Interface classes must have a virtual destructor for safe polymorphic deletion.
- **Concrete derived classes**: Must override all pure virtual functions to be instantiable.

### Prerequisites

- Understanding of classes, inheritance, and virtual functions.
- Familiarity with pointers and references (for polymorphic use).
- Knowledge of constructors, destructors, and access control.
- Basic awareness of multiple inheritance and the diamond problem.

### Related Programming Areas

- **Polymorphism** – abstract classes are the primary vehicle for runtime polymorphism.
- **Design patterns** – Strategy, Template Method, Factory, Visitor, Observer.
- **Dependency inversion** – interfaces decouple high-level and low-level modules.
- **Multiple inheritance** – combining interfaces.
- **Virtual inheritance** – solving diamond hierarchies.
- **Testing** – interfaces enable mock implementations.

### Core Concepts / Features

1. Abstract Classes
2. Pure Virtual Functions
3. Interface Classes
4. Multiple Inheritance
5. Virtual Inheritance
6. Abstract Class Best Practices
7. Interface Best Practices

---

## 1. Abstract Classes

### Definitions

**Core Definition**
An abstract class is a class that cannot be instantiated because it contains at least one pure virtual function; it serves as a base class defining a contract for derived classes.

**Technical Definition**
A class is abstract if it has at least one pure virtual function (declared with `= 0`). Abstract classes cannot be instantiated directly, but pointers and references to them can be used polymorphically. Derived classes must override all pure virtual functions to become concrete. Abstract classes may contain data members, non-virtual functions, constructors, and destructors, and may provide partial implementations. The destructor of an abstract class should be virtual if polymorphic deletion is possible.

**Beginner-Friendly Explanation**
An abstract class is a **partially finished blueprint**. It defines some common parts and leaves other parts (pure virtual functions) for derived classes to complete. You cannot build the blueprint directly; you must finish it first by deriving and implementing the missing parts.

### Purposes

- To define a common interface for a family of related types.
- To prevent instantiation of incomplete types.
- To provide partial implementations shared by derived classes.
- To enable runtime polymorphism through base-class pointers/references.
- To express design contracts and invariants.

### Syntax Rules and Structure

**General Syntax**
```cpp
class Abstract {
public:
    virtual void f() = 0;              // pure virtual
    virtual int g() const = 0;         // pure virtual const
    virtual ~Abstract() = default;     // virtual destructor
protected:
    int shared_data_;                  // optional shared data
};
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `= 0` | Pure virtual function (makes the class abstract). |
| Virtual destructor | Required for polymorphic deletion. |
| Data members | Optional shared state for derived classes. |
| Non-virtual functions | Optional shared implementation. |

**Syntax Rules**
- A class with at least one pure virtual function is abstract.
- Abstract classes cannot be instantiated.
- Derived classes must override all pure virtual functions to be concrete.
- Abstract classes can have constructors and destructors (called by derived classes).
- Pure virtual functions can have definitions (callable via qualified name).

**Constraints and Limitations**
- Cannot create objects of abstract classes.
- Cannot pass abstract objects by value (only pointers/references).
- Cannot use abstract classes as function return types by value.
- A pure virtual function cannot be `final`.

### Annotated Code Examples

**Example 1: Abstract class with partial implementation**

```cpp
#include <iostream>
#include <string>

class Shape {
public:
    virtual double area() const = 0;          // pure virtual
    virtual std::string name() const = 0;      // pure virtual
    virtual ~Shape() = default;                 // virtual destructor

    void describe() const {                     // shared implementation
        std::cout << name() << " has area " << area() << '\n';
    }
};

class Circle : public Shape {
    double radius_;
public:
    explicit Circle(double r) : radius_(r) {}
    double area() const override { return 3.14159 * radius_ * radius_; }
    std::string name() const override { return "Circle"; }
};

int main() {
    // Shape s;   // error: abstract
    Circle c(2.0);
    Shape& s = c;
    s.describe();
}
```

**Expected Output**
```
Circle has area 12.5664
```

**Why this output?**  
`Shape` is abstract (pure virtual `area` and `name`). `Circle` implements both, becoming concrete. `describe()` uses the virtual functions polymorphically.

### Real-World Cases

- **Shape hierarchies** – `Shape` with `area()` and `perimeter()`.
- **Document elements** – `Element` with `render()`.
- **State machines** – `State` with `handle()`.
- **Framework base classes** – abstract hooks for customization.

---

## 2. Pure Virtual Functions

### Definitions

**Core Definition**
A pure virtual function is a virtual function declared with `= 0` that **must be overridden** by derived classes for them to be concrete; it makes the class abstract.

**Technical Definition**
A pure virtual function is declared as `virtual return_type f(params) = 0;`. It makes the class abstract (cannot be instantiated). The function may still have a definition (outside the class), which can be called via qualified name. Derived classes must override all pure virtual functions to become concrete. Pure virtual functions define the interface contract.

**Beginner-Friendly Explanation**
A pure virtual function is a **“you must implement this”** declaration. It says: “Every derived class must provide its own version of this function, or it can't be instantiated.”

### Purposes

- To define interface contracts.
- To make a class abstract.
- To require derived classes to provide implementations.
- To support polymorphism through a common interface.
- To specify required operations without providing an implementation.

### Syntax Rules and Structure

**General Syntax**
```cpp
class Interface {
public:
    virtual void f() = 0;           // pure virtual
    virtual int g() const = 0;      // pure virtual const
    virtual ~Interface() = default; // virtual destructor
};
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `= 0` | Pure virtual specifier. |
| Definition | Optional (callable via `Interface::f`). |
| Override required | Derived classes must implement. |

**Syntax Rules**
- A pure virtual function makes the class abstract.
- It can have a definition (outside the class).
- Derived classes must override all pure virtual functions to be concrete.
- Pure virtual functions can be `const`, `noexcept`, `&&`-qualified, etc.
- Pure virtual functions cannot be `final`.

**Constraints and Limitations**
- Abstract classes cannot be instantiated.
- Pure virtual functions cannot be `final`.
- A pure virtual function with a definition is still pure virtual (the class remains abstract).

### Annotated Code Examples

**Example 1: Pure virtual function with definition**

```cpp
#include <iostream>

class Base {
public:
    virtual void f() = 0;   // pure virtual
};
void Base::f() { std::cout << "Base::f definition\n"; }

class Derived : public Base {
public:
    void f() override {
        Base::f();          // call the pure virtual definition
        std::cout << "Derived::f\n";
    }
};

int main() {
    Derived d;
    d.f();
}
```

**Expected Output**
```
Base::f definition
Derived::f
```

**Why this output?**  
`Base::f` is pure virtual but has a definition. `Derived::f` overrides it and explicitly calls `Base::f()`.

### Real-World Cases

- **Interfaces** – defining required operations.
- **Template Method pattern** – base class calls pure virtual hooks.
- **Abstract base classes** – contracts for derived implementations.

---

## 3. Interface Classes

### Definitions

**Core Definition**
An interface class is an abstract class with **only pure virtual functions** (and a virtual destructor), defining a pure contract with no data members and no implementation.

**Technical Definition**
An interface class is an abstract class that has:
- No non-static data members.
- No non-virtual member functions (except the destructor).
- Only pure virtual functions.
- A virtual destructor.

Interface classes represent pure contracts. They are commonly used for dependency inversion, testing (mock implementations), and multiple interface implementation. The C++ Core Guidelines state: “Use abstract classes as interfaces” (C.122) and “Use multiple inheritance to represent multiple distinct interfaces” (C.135).

**Beginner-Friendly Explanation**
An interface class is a **pure contract** — it says what operations exist but provides no implementation and no data. It's like a job description: anyone who takes the job (derived class) must fulfill all the duties (pure virtual functions).

### Purposes

- To define pure interfaces for polymorphism.
- To enable dependency inversion and testability.
- To support multiple interface implementation.
- To decouple clients from implementations.
- To specify contracts without implementation details.

### Syntax Rules and Structure

**General Syntax**
```cpp
class Interface {
public:
    virtual void f() = 0;
    virtual int g() const = 0;
    virtual ~Interface() = default;
};
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| Pure virtual functions | Define the contract. |
| Virtual destructor | Correct polymorphic deletion. |
| No data members | Pure interface. |

**Syntax Rules**
- Interface classes should have a virtual destructor.
- They should have no data members.
- All member functions should be pure virtual (except the destructor).
- Derived classes implement the interface.
- Interfaces can be combined via multiple inheritance.

**Constraints and Limitations**
- Interface classes add a vtable pointer overhead.
- Multiple interface inheritance can complicate layout.
- Interface classes cannot be instantiated.

### Annotated Code Examples

**Example 1: Interface class**

```cpp
#include <iostream>

class Printable {
public:
    virtual void print() const = 0;
    virtual ~Printable() = default;
};

class Document : public Printable {
public:
    void print() const override { std::cout << "Document\n"; }
};

int main() {
    Document d;
    Printable& p = d;
    p.print();
}
```

**Expected Output**
```
Document
```

**Why this output?**  
`Printable` is an interface class. `Document` implements `print()`. The call through `Printable&` dispatches to `Document::print`.

### Real-World Cases

- **Dependency injection** – interfaces for services.
- **Testing** – mock implementations of interfaces.
- **Plugin systems** – interfaces for extensions.
- **Multiple interfaces** – combining `Serializable`, `Comparable`, etc.

---

## 4. Multiple Inheritance

### Definitions

**Core Definition**
Multiple inheritance allows a class to inherit from **more than one base class**, combining their members and interfaces.

**Technical Definition**
A class can have multiple base classes in its base-clause: `class Derived : public Base1, public Base2 { ... };`. The derived class inherits members from all bases. Ambiguities arise when multiple bases have members with the same name; the derived class must disambiguate with qualified names. Multiple inheritance is used for combining interfaces (mixins) and, less commonly, implementations. The C++ Core Guidelines recommend using multiple inheritance primarily for interface composition (C.135, C.136, C.137).

**Beginner-Friendly Explanation**
Multiple inheritance lets a class have **more than one parent**. It can combine features from several sources. It's powerful but can be complicated — especially when parents share a grandparent (the diamond problem).

### Purposes

- To combine multiple interfaces (e.g., `Serializable`, `Comparable`).
- To mix in functionality from multiple bases.
- To model complex relationships that single inheritance cannot capture.
- To implement multiple abstract interfaces.

### Syntax Rules and Structure

**General Syntax**
```cpp
class Derived : public Base1, public Base2, ... { ... };
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| Multiple base classes | Comma-separated base-clause. |
| Ambiguity resolution | `Base1::member` to disambiguate. |
| Construction order | Bases constructed in declaration order. |

**Syntax Rules**
- Bases are constructed in declaration order, destroyed in reverse.
- Ambiguous members must be disambiguated with qualified names.
- The derived class constructor initializes all bases (explicitly or implicitly).
- Multiple inheritance can combine abstract interfaces.

**Constraints and Limitations**
- Ambiguity can arise with same-named members.
- The diamond problem requires virtual inheritance.
- Layout and pointer adjustment can be complex.

### Annotated Code Examples

**Example 1: Multiple interface implementation**

```cpp
#include <iostream>

class Serializable {
public:
    virtual void serialize() const = 0;
    virtual ~Serializable() = default;
};

class Comparable {
public:
    virtual bool less_than(const Comparable&) const = 0;
    virtual ~Comparable() = default;
};

class Record : public Serializable, public Comparable {
    int id_;
public:
    explicit Record(int id) : id_(id) {}
    void serialize() const override { std::cout << "Record(" << id_ << ")\n"; }
    bool less_than(const Comparable&) const override { return true; }
};

int main() {
    Record r(42);
    r.serialize();
    Serializable& s = r;
    s.serialize();
}
```

**Expected Output**
```
Record(42)
Record(42)
```

**Why this output?**  
`Record` implements two interfaces. Both direct and polymorphic calls to `serialize()` work.

### Real-World Cases

- **Interface composition** – combining multiple abstract interfaces.
- **Mixins** – adding functionality from multiple bases.
- **Framework classes** – inheriting from both a base and an interface.

---

## 5. Virtual Inheritance

### Definitions

**Core Definition**
Virtual inheritance ensures that a **common base class** is shared as a **single subobject** when inherited through multiple paths, solving the diamond problem.

**Technical Definition**
Virtual inheritance is specified with the `virtual` keyword in the base-clause: `class Derived : virtual public Base { ... };`. When multiple inheritance paths lead to the same virtual base, only **one** shared base subobject is created. The most-derived class is responsible for initializing the virtual base. Virtual inheritance is used to solve the diamond problem and to share common state across a hierarchy.

**Beginner-Friendly Explanation**
Virtual inheritance is like **sharing one grandparent** instead of having two copies. In the diamond problem, `D` inherits from `B` and `C`, both of which inherit from `A`. Without virtual inheritance, `D` gets two copies of `A`; with virtual inheritance, it gets one shared copy.

### Purposes

- To solve the diamond problem.
- To share a common base subobject across multiple inheritance paths.
- To avoid ambiguity and duplicate state.
- To model hierarchies where a base should appear only once.

### Syntax Rules and Structure

**General Syntax**
```cpp
class B : virtual public A { ... };
class C : virtual public A { ... };
class D : public B, public C { ... };   // one shared A
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `virtual public A` | Virtual inheritance from `A`. |
| Shared subobject | Only one `A` in `D`. |
| Most-derived init | `D` initializes `A`. |

**Syntax Rules**
- The `virtual` keyword appears in the base-clause.
- The most-derived class initializes the virtual base.
- Virtual inheritance affects layout and construction order.
- Virtual bases are constructed before non-virtual bases.

**Constraints and Limitations**
- Adds overhead (virtual base pointer).
- Complicates construction and layout.
- Should be used sparingly; often composition is simpler.

### Annotated Code Examples

**Example 1: Virtual inheritance solving the diamond**

```cpp
#include <iostream>

class A {
public:
    int value = 1;
};

class B : virtual public A {};
class C : virtual public A {};
class D : public B, public C {};

int main() {
    D d;
    d.value = 42;           // one shared A
    std::cout << d.value << '\n';
    std::cout << d.B::value << '\n';   // same A
    std::cout << d.C::value << '\n';   // same A
}
```

**Expected Output**
```
42
42
42
```

**Why this output?**  
Virtual inheritance makes `B` and `C` share a single `A` subobject in `D`, so all accesses refer to the same `value`.

### Real-World Cases

- **Diamond hierarchies** – shared base across multiple paths.
- **I/O streams** – `std::iostream` uses virtual inheritance from `std::istream` and `std::ostream`.
- **Framework hierarchies** – shared base for common state.

---

## 6. Abstract Class Best Practices

### Definitions

**Core Definition**
Abstract class best practices are guidelines for designing abstract classes that are correct, safe, and maintainable, based on the C++ Core Guidelines.

**Technical Definition**
The C++ Core Guidelines provide rules: **C.120** (use class hierarchies for inherent hierarchical concepts), **C.121** (if a base class is used polymorphically, make it abstract), **C.126** (an abstract class typically doesn't need a user-written constructor), **C.127** (a class with a virtual function should have a virtual or protected destructor), **C.128** (virtual functions should specify exactly one of `virtual`, `override`, or `final`), **C.129** (distinguish implementation inheritance from interface inheritance), **C.132** (don't make functions virtual without reason), **C.133** (avoid `protected` data), and **C.134** (ensure all non-`const` data members have the same access level).

**Beginner-Friendly Explanation**
Best practices for abstract classes are simple: **make base classes abstract when used polymorphically**, **give them virtual destructors**, **avoid `protected` data**, **use `override` in derived classes**, and **don't add unnecessary virtual functions**.

### Key Guidelines (Summary Table)

| Guideline | Summary | Recommendation |
|-----------|---------|----------------|
| **C.121** | If a base class is used polymorphically, make it abstract. | Abstract base classes. |
| **C.126** | An abstract class typically doesn't need a user-written constructor. | Let derived classes handle construction. |
| **C.127** | A class with a virtual function should have a virtual or protected destructor. | Virtual destructors. |
| **C.128** | Virtual functions should specify exactly one of `virtual`, `override`, or `final`. | Use `override` in derived classes. |
| **C.129** | Distinguish implementation inheritance from interface inheritance. | Interfaces for contracts. |
| **C.132** | Don't make functions virtual without reason. | Virtual functions have overhead. |
| **C.133** | Avoid `protected` data. | Private data with protected accessors. |
| **C.134** | Ensure all non-`const` data members have the same access level. | Consistent access. |

### Annotated Code Examples

**Example 1: Abstract base with virtual destructor**

```cpp
#include <iostream>

class Shape {
public:
    virtual double area() const = 0;
    virtual ~Shape() = default;   // virtual destructor
};

class Square : public Shape {
    double side_;
public:
    explicit Square(double side) : side_(side) {}
    double area() const override { return side_ * side_; }
};

int main() {
    Square s(3.0);
    Shape* p = &s;
    std::cout << p->area() << '\n';
}
```

**Expected Output**
```
9
```

**Why this is best practice?**  
`Shape` is abstract (C.121) and has a virtual destructor (C.127). `Square` uses `override` (C.128).

### Real-World Cases

- **Standard library** – `std::exception` has a virtual destructor.
- **Code review** – Core Guidelines C.120–C.134 are common checkpoints.
- **Static analysis** – clang-tidy enforces many abstract class rules.

---

## 7. Interface Best Practices

### Definitions

**Core Definition**
Interface best practices are guidelines for designing and using interface classes that are pure, minimal, and safe, based on the C++ Core Guidelines and community standards.

**Technical Definition**
The C++ Core Guidelines provide interface-related rules: **C.122** (use abstract classes as interfaces), **C.135** (use multiple inheritance to represent multiple distinct interfaces), **C.136** (use multiple inheritance to represent implementation inheritance only if the base classes are pure interfaces), **C.137** (use `virtual` base classes only for interface inheritance), **C.129** (distinguish implementation inheritance from interface inheritance), and **I.25** (prefer abstract classes as interfaces to class hierarchies). Additional guidance: interfaces should have virtual destructors, no data members, and only pure virtual functions; keep interfaces minimal and cohesive.

**Beginner-Friendly Explanation**
Best practices for interfaces are simple: **keep them pure** (no data, only pure virtual functions), **give them virtual destructors**, **keep them small and focused**, **use multiple inheritance only for interfaces**, and **avoid mixing implementation and interface inheritance**.

### Key Guidelines (Summary Table)

| Guideline | Summary | Recommendation |
|-----------|---------|----------------|
| **C.122** | Use abstract classes as interfaces. | Pure interfaces for polymorphism. |
| **C.135** | Use multiple inheritance to represent multiple distinct interfaces. | Combine interfaces, not implementations. |
| **C.136** | Use multiple inheritance to represent implementation inheritance only if the base classes are pure interfaces. | Avoid implementation diamonds. |
| **C.137** | Use `virtual` base classes only for interface inheritance. | Virtual inheritance for interfaces. |
| **I.25** | Prefer abstract classes as interfaces to class hierarchies. | Interfaces for decoupling. |
| **Virtual destructor** | Interfaces must have a virtual destructor. | Safe polymorphic deletion. |
| **No data members** | Interfaces should be pure contracts. | No state in interfaces. |
| **Minimal and cohesive** | Keep interfaces small and focused. | Follow the interface segregation principle. |

### Annotated Code Examples

**Example 1: Pure interface with virtual destructor**

```cpp
#include <iostream>

class Drawable {
public:
    virtual void draw() const = 0;
    virtual ~Drawable() = default;   // virtual destructor
};

class Resizable {
public:
    virtual void resize(double factor) = 0;
    virtual ~Resizable() = default;
};

class Widget : public Drawable, public Resizable {
public:
    void draw() const override { std::cout << "drawing\n"; }
    void resize(double) override { std::cout << "resizing\n"; }
};

int main() {
    Widget w;
    Drawable& d = w;
    Resizable& r = w;
    d.draw();
    r.resize(2.0);
}
```

**Expected Output**
```
drawing
resizing
```

**Why this is best practice?**  
`Drawable` and `Resizable` are pure interfaces with virtual destructors. `Widget` implements both via multiple inheritance (C.135).

### Real-World Cases

- **Dependency injection** – interfaces for services.
- **Testing** – mock implementations of interfaces.
- **Plugin systems** – interfaces for extensions.
- **Framework design** – pure interfaces for decoupling.

---

## References

- Abstract class (cppreference) – https://en.cppreference.com/w/cpp/language/abstract_class
- Pure virtual function (cppreference) – https://en.cppreference.com/w/cpp/language/abstract_class
- Multiple inheritance (cppreference) – https://en.cppreference.com/w/cpp/language/derived_class#Multiple_inheritance
- Virtual inheritance (cppreference) – https://en.cppreference.com/w/cpp/language/derived_class#Virtual_base_classes
- Virtual destructors (cppreference) – https://en.cppreference.com/w/cpp/language/destructor#Virtual_destructors
- C++ Core Guidelines — C.120: Use class hierarchies to represent concepts with inherent hierarchical structure – https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rh-domain
- C++ Core Guidelines — C.121: If a base class is used polymorphically, make it abstract – https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rh-abstract
- C++ Core Guidelines — C.122: Use abstract classes as interfaces when complete separation of interface and implementation is needed – https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rh-abstract-interface
- C++ Core Guidelines — C.126: An abstract class typically doesn't need a user-written constructor – https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rh-abstract-ctor
- C++ Core Guidelines — C.127: A class with a virtual function should have a virtual or protected destructor – https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rh-virtual-dtor
- C++ Core Guidelines — C.128: Virtual functions should specify exactly one of `virtual`, `override`, or `final` – https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rh-override
- C++ Core Guidelines — C.129: When designing a class hierarchy, distinguish between implementation inheritance and interface inheritance – https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rh-kind
- C++ Core Guidelines — C.135: Use multiple inheritance to represent multiple distinct interfaces – https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rh-mi-interface
- C++ Core Guidelines — C.136: Use multiple inheritance to represent implementation inheritance only if the base classes are pure interfaces – https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rh-mi-implementation
- C++ Core Guidelines — C.137: Use `virtual` base classes only for interface inheritance – https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rh-vbase
- C++ Core Guidelines — I.25: Prefer abstract classes as interfaces to class hierarchies – https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Ri-abstract
- Microsoft Learn — Abstract classes (C++) – https://learn.microsoft.com/en-us/cpp/cpp/abstract-classes-cpp
- Microsoft Learn — Multiple base classes – https://learn.microsoft.com/en-us/cpp/cpp/multiple-base-classes
- Microsoft Learn — Virtual base classes – https://learn.microsoft.com/en-us/cpp/cpp/virtual-base-classes

---

This cheat sheet provides a complete, structured reference for C++ abstract classes and interfaces, covering abstract classes, pure virtual functions, interface classes, multiple inheritance, virtual inheritance, and best practices. Each section follows a uniform template with definitions, purposes, syntax rules, annotated examples, expected outputs, and authoritative references.