# Polymorphism — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
Polymorphism is the ability of objects of different types to be treated through a **common interface**, with the specific behavior determined either at **compile time** (overloading/templates) or at **runtime** (virtual functions).

**Technical Definition**
In C++, polymorphism manifests in two primary forms. **Compile-time polymorphism** (static polymorphism) is resolved during translation: function overloading, operator overloading, and templates. **Runtime polymorphism** (dynamic polymorphism) is resolved at program execution via **virtual functions**: a base-class pointer or reference can refer to a derived-class object, and the call is dispatched to the most-derived override through a **virtual table** (vtable) and **virtual function pointers**. Runtime polymorphism requires virtual functions, inheritance, and either pointers or references; it is governed by rules for overriding (`override`), sealing (`final`), abstract classes (pure virtual functions), and virtual destructors.

**Beginner-Friendly Explanation**
Polymorphism means “**many forms**.” It lets you write code that works with a **general type** but behaves correctly for **specific types**. For example, you can have a `Shape` pointer that actually points to a `Circle` or `Square`, and calling `area()` runs the right version for each. C++ offers two flavors: **compile-time** (the compiler picks the right function based on types, as with overloading and templates) and **runtime** (the program picks the right function at execution time using virtual functions). Runtime polymorphism is what most people mean by “polymorphism” in object-oriented programming.

### Key Characteristics

- **Common interface, multiple implementations**: Different types respond to the same operation differently.
- **Compile-time resolution**: Overloading and templates are resolved by the compiler.
- **Runtime resolution**: Virtual functions use dynamic dispatch through the vtable.
- **Requires indirection**: Runtime polymorphism needs pointers or references to base classes.
- **Virtual destructors**: Essential for correct polymorphic deletion.
- **Abstract classes**: Define interfaces with pure virtual functions.
- **`override`/`final`**: Make overriding explicit and seal implementations.
- **Slicing hazard**: Copying a derived object into a base object loses the derived part.

### Prerequisites

- Understanding of classes, inheritance, and member functions.
- Familiarity with pointers and references.
- Knowledge of constructors and destructors.
- Basic awareness of function overloading and templates.

### Related Programming Areas

- **Inheritance** – the foundation of runtime polymorphism.
- **Abstract classes and interfaces** – define polymorphic contracts.
- **Templates** – compile-time polymorphism.
- **Design patterns** – Strategy, Template Method, Visitor, Factory.
- **Dynamic dispatch** – vtable-based call resolution.
- **Object slicing** – a common pitfall of polymorphic value semantics.

### Core Concepts / Features

1. Polymorphism
2. Compile-Time Polymorphism
3. Runtime Polymorphism
4. Virtual Functions
5. Virtual Tables (Vtables)
6. Virtual Function Pointers
7. Dynamic Dispatch
8. Virtual Destructors
9. Pure Virtual Functions
10. Abstract Classes
11. `override`
12. `final`
13. Polymorphism Best Practices

---

## 1. Polymorphism

### Definitions

**Core Definition**
Polymorphism is the ability to treat objects of different types through a common interface, with behavior selected by the actual type.

**Technical Definition**
Polymorphism in C++ is achieved through:
- **Ad-hoc polymorphism**: function/operator overloading.
- **Parametric polymorphism**: templates (generic programming).
- **Subtype polymorphism**: virtual functions and inheritance (runtime polymorphism).

The term most commonly refers to subtype polymorphism, where a base-class pointer/reference refers to a derived object and virtual calls dispatch to the derived override.

**Beginner-Friendly Explanation**
Polymorphism means **one interface, many behaviors**. You can call `speak()` on an `Animal*` that points to a `Dog` or a `Cat`, and each animal responds in its own way. The code doesn't need to know the exact type — just that it's an `Animal`.

### Purposes

- To write code that works with general types but behaves correctly for specific types.
- To enable extensibility without modifying existing code.
- To decouple clients from concrete implementations.
- To support plug-in architectures and frameworks.
- To model real-world “many forms” relationships.

### Syntax Rules and Structure

**General Pattern (Runtime Polymorphism)**
```cpp
class Base {
public:
    virtual void f() = 0;   // interface
    virtual ~Base() = default;
};

class Derived : public Base {
public:
    void f() override { /* specific behavior */ }
};

void use(Base& b) { b.f(); }   // polymorphic call
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| Base class | Defines the common interface. |
| Virtual function | Enables dynamic dispatch. |
| Derived class | Overrides the virtual function. |
| Pointer/reference | Required for polymorphic calls. |

**Syntax Rules**
- Runtime polymorphism requires virtual functions and inheritance.
- Calls through base pointers/references dispatch dynamically.
- Calls through base objects (by value) do not dispatch (slicing).

**Constraints and Limitations**
- Runtime polymorphism has overhead (vtable, indirection).
- Object slicing occurs when copying derived objects by value into base objects.
- Virtual calls cannot be inlined as easily as static calls.

### Annotated Code Examples

**Example 1: Runtime polymorphism**

```cpp
#include <iostream>
#include <memory>
#include <vector>

class Animal {
public:
    virtual void speak() const = 0;
    virtual ~Animal() = default;
};

class Dog : public Animal {
public:
    void speak() const override { std::cout << "Woof!\n"; }
};

class Cat : public Animal {
public:
    void speak() const override { std::cout << "Meow!\n"; }
};

int main() {
    std::vector<std::unique_ptr<Animal>> animals;
    animals.push_back(std::make_unique<Dog>());
    animals.push_back(std::make_unique<Cat>());
    for (const auto& a : animals) a->speak();
}
```

**Expected Output**
```
Woof!
Meow!
```

**Why this output?**  
Each `Animal*` points to a different derived object. The virtual `speak()` call dispatches to the correct override at runtime.

### Real-World Cases

- **GUI frameworks** – widgets respond to events polymorphically.
- **Game engines** – entities/behaviors through base interfaces.
- **Plugin systems** – plugins implement a common interface.
- **Shapes** – `Circle`, `Square` implement `Shape::area()`.

---

## 2. Compile-Time Polymorphism

### Definitions

**Core Definition**
Compile-time polymorphism (static polymorphism) is polymorphism resolved during compilation, achieved through function overloading, operator overloading, and templates.

**Technical Definition**
Compile-time polymorphism includes:
- **Overloading**: multiple functions with the same name but different parameter lists; the compiler selects the best match.
- **Operator overloading**: user-defined operators for class types.
- **Templates**: function and class templates instantiated with specific types; the compiler generates specialized code.

All decisions are made at compile time, with no runtime dispatch overhead.

**Beginner-Friendly Explanation**
Compile-time polymorphism is when the compiler **already knows** which function to call based on the types you use. Overloading (`print(int)` vs. `print(string)`) and templates (`max<int>` vs. `max<double>`) are resolved before the program runs.

### Purposes

- To provide type-specific behavior without runtime overhead.
- To write generic code that works with many types.
- To enable optimizations (inlining, specialization).
- To support operator overloading for natural syntax.

### Syntax Rules and Structure

**Overloading**
```cpp
void print(int);
void print(double);
void print(const std::string&);
```

**Templates**
```cpp
template <typename T>
T max(T a, T b) { return a > b ? a : b; }
```

**Component Breakdown**

| Mechanism | Resolution |
|-----------|------------|
| Overloading | Compiler selects based on argument types. |
| Templates | Compiler instantiates for specific types. |
| Operator overloading | Compiler maps operators to functions. |

**Syntax Rules**
- Overloads must differ in parameter lists.
- Templates are instantiated at compile time.
- The compiler performs overload resolution and template argument deduction.

**Constraints and Limitations**
- Compile-time polymorphism cannot dispatch based on runtime type.
- Templates can increase code size and compile time.
- Error messages for templates can be verbose.

### Annotated Code Examples

**Example 1: Overloading and templates**

```cpp
#include <iostream>
#include <string>

void print(int x)    { std::cout << "int: " << x << '\n'; }
void print(double x) { std::cout << "double: " << x << '\n'; }

template <typename T>
T max(T a, T b) { return a > b ? a : b; }

int main() {
    print(42);
    print(3.14);
    std::cout << max(3, 7) << '\n';
    std::cout << max(2.5, 1.5) << '\n';
}
```

**Expected Output**
```
int: 42
double: 3.14
7
2.5
```

**Why this output?**  
Overload resolution selects `print(int)` and `print(double)`. Template instantiation generates `max<int>` and `max<double>`.

### Real-World Cases

- **Standard library algorithms** – `std::sort` with comparators.
- **Generic containers** – `std::vector<T>`.
- **Operator overloading** – `operator+` for complex numbers.
- **Type traits** – compile-time type selection.

---

## 3. Runtime Polymorphism

### Definitions

**Core Definition**
Runtime polymorphism (dynamic polymorphism) is polymorphism resolved at program execution through virtual functions and inheritance, allowing a base-class pointer/reference to dispatch to the derived-class override.

**Technical Definition**
Runtime polymorphism requires:
1. A base class with at least one virtual function.
2. A derived class that overrides the virtual function.
3. A pointer or reference to the base class that refers to a derived object.

The call is resolved through the object's **vtable** and **virtual function pointer**, selecting the most-derived override. This is **dynamic dispatch**.

**Beginner-Friendly Explanation**
Runtime polymorphism is when the program **decides at runtime** which function to call, based on the actual object type. If you have an `Animal*` that points to a `Dog`, calling `speak()` runs `Dog::speak()`, even though the pointer type is `Animal*`.

### Purposes

- To write code that works with objects whose exact type is determined at runtime.
- To support extensible architectures (plug-ins, frameworks).
- To implement design patterns (Strategy, Template Method, Visitor).
- To decouple clients from concrete implementations.

### Syntax Rules and Structure

**General Syntax**
```cpp
class Base {
public:
    virtual void f();        // virtual function
    virtual ~Base() = default;
};

class Derived : public Base {
public:
    void f() override;       // override
};

void use(Base& b) { b.f(); }   // dynamic dispatch
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `virtual` | Enables dynamic dispatch. |
| `override` | Ensures correct overriding. |
| Pointer/reference | Required for polymorphic calls. |
| Vtable | Runtime dispatch mechanism. |

**Syntax Rules**
- Virtual functions enable runtime dispatch.
- The call must be through a pointer or reference to the base class.
- The derived class must override the virtual function.
- The base class must have a virtual destructor for safe deletion.

**Constraints and Limitations**
- Overhead: vtable lookup and indirection.
- Cannot dispatch through base objects by value (slicing).
- Virtual calls may inhibit inlining.

### Annotated Code Examples

**Example 1: Dynamic dispatch**

```cpp
#include <iostream>

class Base {
public:
    virtual void f() const { std::cout << "Base::f\n"; }
    virtual ~Base() = default;
};

class Derived : public Base {
public:
    void f() const override { std::cout << "Derived::f\n"; }
};

void call(const Base& b) { b.f(); }   // dispatch based on dynamic type

int main() {
    Base b;
    Derived d;
    call(b);    // Base::f
    call(d);    // Derived::f
}
```

**Expected Output**
```
Base::f
Derived::f
```

**Why this output?**  
`call` takes a `const Base&`. When passed a `Derived`, the virtual call dispatches to `Derived::f` at runtime.

### Real-World Cases

- **Event handling** – handlers dispatched by event type.
- **Game entities** – `update()` called polymorphically.
- **Document elements** – `render()` for different element types.
- **Plugin interfaces** – dynamic loading and dispatch.

---

## 4. Virtual Functions

### Definitions

**Core Definition**
A virtual function is a member function declared with the `virtual` keyword that can be **overridden** by derived classes and dispatched dynamically through a base-class pointer or reference.

**Technical Definition**
A virtual function is a non-static member function declared with `virtual` in a base class. It is overridden by a function with the same name, parameter types, cv-qualifiers, and ref-qualifiers in a derived class. Calls through a base-class pointer/reference are resolved at runtime using the object's vtable. Virtual functions enable subtype polymorphism. Overriding requires matching signature; `override` (C++11) makes the relationship explicit.

**Beginner-Friendly Explanation**
A virtual function is a function that says: “**Derived classes can replace me**.” When you call it through a base pointer, C++ looks at the actual object type and runs the most-derived version.

### Purposes

- To enable runtime polymorphism.
- To allow derived classes to customize behavior.
- To define interfaces that derived classes implement.
- To support dynamic dispatch in hierarchies.

### Syntax Rules and Structure

**General Syntax**
```cpp
class Base {
public:
    virtual void f();          // declaration
};
void Base::f() { ... }         // definition

class Derived : public Base {
public:
    void f() override;         // override
};
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `virtual` | Declares the function virtual. |
| `override` | Marks an override (C++11). |
| Matching signature | Required for overriding. |

**Syntax Rules**
- Virtual functions are declared with `virtual` in the base class.
- Derived overrides may omit `virtual` (they are implicitly virtual).
- Use `override` in derived classes for safety.
- The function signature must match exactly (name, parameters, cv-qualifiers).
- Virtual functions cannot be `static` or `friend`.

**Constraints and Limitations**
- Virtual functions incur a vtable lookup overhead.
- Constructors and destructors cannot be virtual (destructors can be virtual).
- Calling virtual functions in constructors/destructors dispatches to the current class, not the derived class.

### Annotated Code Examples

**Example 1: Virtual function**

```cpp
#include <iostream>

class Base {
public:
    virtual void f() const { std::cout << "Base::f\n"; }
    virtual ~Base() = default;
};

class Derived : public Base {
public:
    void f() const override { std::cout << "Derived::f\n"; }
};

int main() {
    Base* p = new Derived;
    p->f();       // Derived::f
    delete p;
}
```

**Expected Output**
```
Derived::f
```

**Why this output?**  
`f` is virtual in `Base` and overridden in `Derived`. The call through `Base*` dispatches to `Derived::f`.

### Real-World Cases

- **Interfaces** – pure virtual functions.
- **Template Method pattern** – base class calls virtual hooks.
- **Polymorphic containers** – `vector<unique_ptr<Base>>`.

---

## 5. Virtual Tables (Vtables)

### Definitions

**Core Definition**
A virtual table (vtable) is a compiler-generated table of function pointers used to implement dynamic dispatch for virtual functions.

**Technical Definition**
For each class with virtual functions, the compiler generates a **vtable**: an array of pointers to the virtual functions of that class. Each object of such a class contains a hidden **vptr** (virtual table pointer) pointing to its class's vtable. When a virtual function is called through a base pointer, the compiler loads the vptr, indexes the vtable, and calls the function pointer. This is **dynamic dispatch**. The vtable is typically shared by all objects of the same class.

**Beginner-Friendly Explanation**
A vtable is like a **phone book** for virtual functions. Each class has its own phone book listing which function to call. Each object has a hidden pointer to its class's phone book. When you call a virtual function, the program looks up the correct function in the phone book.

### Purposes

- To implement runtime polymorphism.
- To enable dynamic dispatch with constant-time lookup.
- To share dispatch information across objects of the same class.
- To support virtual inheritance and multiple inheritance.

### Syntax Rules and Structure

**Conceptual Layout**
```
Object: [vptr] [data members...]
Vtable: [&Derived::f] [&Derived::g] ...
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| vptr | Hidden pointer in each object. |
| vtable | Array of function pointers per class. |
| Dispatch | Load vptr, index vtable, call function. |

**Syntax Rules**
- The vtable is an implementation detail; the C++ standard does not specify its layout.
- Each class with virtual functions has its own vtable.
- The vptr is initialized by the constructor.
- Virtual calls use the vptr and vtable.

**Constraints and Limitations**
- The vtable layout is compiler-dependent (not portable).
- Multiple inheritance may require multiple vtables or adjusted pointers.
- Virtual inheritance adds more indirection.

### Annotated Code Examples

**Example 1: Conceptual vtable**

```cpp
#include <iostream>

class Base {
public:
    virtual void f() { std::cout << "Base::f\n"; }
    virtual void g() { std::cout << "Base::g\n"; }
    virtual ~Base() = default;
};

class Derived : public Base {
public:
    void f() override { std::cout << "Derived::f\n"; }
    void g() override { std::cout << "Derived::g\n"; }
};

int main() {
    Base* p = new Derived;
    p->f();   // vtable lookup → Derived::f
    p->g();   // vtable lookup → Derived::g
    delete p;
}
```

**Expected Output**
```
Derived::f
Derived::g
```

**Why this output?**  
`Derived`'s vtable contains pointers to `Derived::f` and `Derived::g`. The calls through `Base*` dispatch to these entries.

### Real-World Cases

- **All runtime polymorphism** – vtables are the standard implementation.
- **ABI compatibility** – vtable layout affects binary compatibility.
- **Performance analysis** – vtable indirection is a common optimization target.

---

## 6. Virtual Function Pointers

### Definitions

**Core Definition**
A virtual function pointer is a pointer to a virtual function, typically stored in the vtable, used for dynamic dispatch.

**Technical Definition**
In vtable-based implementations, each virtual function is represented by a pointer to the function (or a thunk that adjusts `this`). The vtable is an array of such pointers. Calling a virtual function through a base pointer involves loading the vptr, indexing the vtable, and calling the function pointer. In C++, you cannot directly obtain a pointer to a virtual function in a way that bypasses dispatch; taking `&Base::f` yields a pointer-to-member, and calling it through a pointer/reference uses dynamic dispatch.

**Beginner-Friendly Explanation**
Virtual function pointers are the **entries in the phone book**. Each virtual function has an address stored in the vtable. When you call a virtual function, the program follows the pointer to the actual function.

### Purposes

- To implement dynamic dispatch.
- To allow runtime selection of implementations.
- To support vtables and polymorphism.
- To enable low-level function dispatch (e.g., in interpreters).

### Syntax Rules and Structure

**Pointer-to-Member Function**
```cpp
void (Base::*pmf)() = &Base::f;
(b->*pmf)();   // uses dynamic dispatch if f is virtual
```

**Component Breakdown**

| Form | Meaning |
|------|---------|
| `&Base::f` | Pointer to member function. |
| `(obj.*pmf)()` | Call through object. |
| `(ptr->*pmf)()` | Call through pointer. |

**Syntax Rules**
- `&Base::f` yields a pointer-to-member function, not a raw function pointer.
- Calling through a pointer-to-member on a virtual function uses dynamic dispatch.
- Raw virtual function pointers (vtable entries) are implementation details.

**Constraints and Limitations**
- Pointers to virtual functions cannot be converted to `void*`.
- The vtable layout is implementation-defined.
- Direct manipulation of vtables is non-portable.

### Annotated Code Examples

**Example 1: Pointer-to-member with virtual dispatch**

```cpp
#include <iostream>

class Base {
public:
    virtual void f() { std::cout << "Base::f\n"; }
    virtual ~Base() = default;
};

class Derived : public Base {
public:
    void f() override { std::cout << "Derived::f\n"; }
};

int main() {
    void (Base::*pmf)() = &Base::f;
    Derived d;
    Base& b = d;
    (b.*pmf)();   // dynamic dispatch → Derived::f
}
```

**Expected Output**
```
Derived::f
```

**Why this output?**  
`pmf` points to `Base::f`, but since `f` is virtual, calling through `b` (which refers to a `Derived`) dispatches to `Derived::f`.

### Real-World Cases

- **Frameworks** – storing member function pointers for callbacks.
- **Interpreter dispatch** – tables of function pointers.
- **Event systems** – handler pointers.

---

## 7. Dynamic Dispatch

### Definitions

**Core Definition**
Dynamic dispatch is the runtime mechanism that selects which implementation of a virtual function to call based on the object's dynamic type.

**Technical Definition**
Dynamic dispatch is implemented via the vtable and vptr. When a virtual function is called through a base-class pointer/reference, the compiler emits code that loads the vptr, indexes the vtable at the function's slot, and calls the function pointer. This selects the most-derived override for the object's dynamic type. Dynamic dispatch is the essence of runtime polymorphism.

**Beginner-Friendly Explanation**
Dynamic dispatch is the program **deciding at runtime** which function to run. It looks at the actual object (not the pointer type) and picks the right version of the virtual function.

### Purposes

- To implement runtime polymorphism.
- To select behavior based on dynamic type.
- To support extensible and pluggable architectures.
- To decouple callers from implementations.

### Syntax Rules and Structure

**Dispatch Conditions**
```cpp
Base* p = new Derived;
p->f();           // dynamic dispatch: calls Derived::f
Base b = Derived(); // slicing: b is Base, no dispatch
b.f();            // calls Base::f
```

**Component Breakdown**

| Condition | Dispatch? |
|-----------|-----------|
| Call through pointer/reference to base | Yes |
| Call through base object (by value) | No (static) |
| Call on `final` function | May be devirtualized |
| Call in constructor/destructor | Dispatches to current class |

**Syntax Rules**
- Dispatch requires a pointer or reference to a base class.
- The function must be virtual.
- The object's dynamic type determines the override.
- Calls through objects (by value) are statically bound (slicing).

**Constraints and Limitations**
- Dynamic dispatch has overhead.
- Calls in constructors/destructors do not dispatch to derived classes.
- Devirtualization can occur when the dynamic type is known.

### Annotated Code Examples

**Example 1: Dynamic vs. static dispatch**

```cpp
#include <iostream>

class Base {
public:
    virtual void f() const { std::cout << "Base::f\n"; }
    void g() const { std::cout << "Base::g\n"; }   // non-virtual
    virtual ~Base() = default;
};

class Derived : public Base {
public:
    void f() const override { std::cout << "Derived::f\n"; }
    void g() const { std::cout << "Derived::g\n"; }   // hides Base::g
};

int main() {
    Derived d;
    Base& b = d;
    b.f();   // dynamic dispatch → Derived::f
    b.g();   // static dispatch → Base::g
}
```

**Expected Output**
```
Derived::f
Base::g
```

**Why this output?**  
`f` is virtual, so `b.f()` dispatches to `Derived::f`. `g` is not virtual, so `b.g()` calls `Base::g` (static binding).

### Real-World Cases

- **Event handling** – dispatching to the correct handler.
- **Game loops** – updating entities polymorphically.
- **UI frameworks** – rendering widgets polymorphically.

---

## 8. Virtual Destructors

### Definitions

**Core Definition**
A virtual destructor is a destructor declared `virtual` in a base class, ensuring that deleting a derived object through a base pointer calls the **correct derived destructor**.

**Technical Definition**
A virtual destructor is dynamically dispatched like any virtual function. When `delete` is called on a base-class pointer to a derived object, the virtual destructor ensures the derived destructor runs first, then the base destructor. Without a virtual destructor, deleting through a base pointer is undefined behavior (the derived destructor is not called). The C++ Core Guidelines state: a base class destructor should be either public and virtual, or protected and non-virtual (C.35).

**Beginner-Friendly Explanation**
A virtual destructor makes sure that when you **delete through a base pointer**, the **right destructor runs**. Without it, only the base part is destroyed, and the derived part leaks.

### Purposes

- To ensure correct destruction of polymorphic objects.
- To prevent resource leaks when deleting through base pointers.
- To support safe polymorphic deletion.
- To define the class's polymorphic contract.

### Syntax Rules and Structure

**General Syntax**
```cpp
class Base {
public:
    virtual ~Base() = default;   // public virtual destructor
};
```

**Component Breakdown**

| Form | Meaning |
|------|---------|
| `virtual ~Base()` | Virtual destructor. |
| `protected: ~Base()` | Non-virtual protected destructor (alternative). |

**Syntax Rules**
- A base class with virtual functions should have a virtual destructor if deletion through base pointers is possible.
- Alternatively, make the destructor protected and non-virtual to prevent deletion through base pointers.
- Virtual destructors incur a vtable entry cost.

**Constraints and Limitations**
- Virtual destructors do not prevent slicing.
- A class with a virtual destructor has a vtable.

### Annotated Code Examples

**Example 1: Virtual destructor**

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
The virtual destructor dispatches to `Derived::~Derived`, which then calls `Base::~Base`.

### Real-World Cases

- **Polymorphic base classes** – interfaces, abstract classes.
- **Smart pointers** – `unique_ptr<Base>` deleting a `Derived`.
- **Containers of base pointers** – polymorphic storage.

---

## 9. Pure Virtual Functions

### Definitions

**Core Definition**
A pure virtual function is a virtual function declared with `= 0` that **must be overridden** by derived classes for them to be concrete.

**Technical Definition**
A pure virtual function is declared as `virtual return_type f(params) = 0;`. It makes the class abstract. The function may still have a definition (outside the class), which can be called via qualified name. Derived classes must override all pure virtual functions to become concrete. Pure virtual functions define the interface contract.

**Beginner-Friendly Explanation**
A pure virtual function is a **“you must implement this”** declaration. It says: “Every derived class must provide its own version of this function, or it can't be instantiated.”

### Purposes

- To define interface contracts.
- To make a class abstract.
- To require derived classes to provide implementations.
- To support polymorphism through a common interface.

### Syntax Rules and Structure

**General Syntax**
```cpp
class Interface {
public:
    virtual void f() = 0;           // pure virtual
    virtual int g() const = 0;      // pure virtual const
    virtual ~Interface() = default;
};
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `= 0` | Pure virtual specifier. |
| Definition | Optional (callable via qualified name). |
| Override required | Derived classes must implement. |

**Syntax Rules**
- A pure virtual function makes the class abstract.
- It can have a definition.
- Derived classes must override all pure virtual functions to be concrete.
- Pure virtual functions can be `const`, `noexcept`, etc.

**Constraints and Limitations**
- Abstract classes cannot be instantiated.
- Pure virtual functions cannot be `final`.

### Annotated Code Examples

**Example 1: Pure virtual function**

```cpp
#include <iostream>

class Shape {
public:
    virtual double area() const = 0;   // pure virtual
    virtual ~Shape() = default;
};

class Square : public Shape {
    double side_;
public:
    explicit Square(double s) : side_(s) {}
    double area() const override { return side_ * side_; }
};

int main() {
    Square s(3.0);
    Shape& shape = s;
    std::cout << shape.area() << '\n';
}
```

**Expected Output**
```
9
```

**Why this output?**  
`Shape` is abstract. `Square` implements `area()`. The call through `Shape&` dispatches to `Square::area`.

### Real-World Cases

- **Interfaces** – defining required operations.
- **Template Method pattern** – base class calls pure virtual hooks.
- **Abstract base classes** – contracts for derived implementations.

---

## 10. Abstract Classes

### Definitions

**Core Definition**
An abstract class is a class with at least one **pure virtual function** that cannot be instantiated directly; it serves as an interface or base for derived classes.

**Technical Definition**
A class is abstract if it has at least one pure virtual function. Abstract classes cannot be instantiated, but pointers and references to them can be used polymorphically. Derived classes must override all pure virtual functions to become concrete. Abstract classes often provide partial implementations and define interfaces.

**Beginner-Friendly Explanation**
An abstract class is a **blueprint that can't be built directly** — it's incomplete. You must derive from it and fill in the missing parts before you can create objects.

### Purposes

- To define interfaces that derived classes must implement.
- To prevent instantiation of incomplete types.
- To provide polymorphic base classes.
- To express design contracts.

### Syntax Rules and Structure

**General Syntax**
```cpp
class Abstract {
public:
    virtual void f() = 0;
    virtual ~Abstract() = default;
};
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `= 0` | Pure virtual function. |
| Abstract class | Cannot be instantiated. |
| Derived concrete | Must override all pure virtuals. |

**Syntax Rules**
- A class with at least one pure virtual function is abstract.
- Abstract classes cannot be instantiated.
- Derived classes must override all pure virtual functions to be concrete.
- Abstract classes can have constructors, destructors, and non-virtual members.

**Constraints and Limitations**
- Cannot create objects of abstract classes.
- Cannot pass abstract objects by value (only pointers/references).
- A pure virtual function can have a definition.

### Annotated Code Examples

**Example 1: Abstract class**

```cpp
#include <iostream>

class Shape {
public:
    virtual double area() const = 0;
    virtual ~Shape() = default;
};

class Circle : public Shape {
    double radius_;
public:
    explicit Circle(double r) : radius_(r) {}
    double area() const override { return 3.14159 * radius_ * radius_; }
};

int main() {
    // Shape s;   // error: abstract
    Circle c(2.0);
    Shape& s = c;
    std::cout << s.area() << '\n';
}
```

**Expected Output**
```
12.5664
```

**Why this output?**  
`Shape` is abstract. `Circle` implements `area()`. The call through `Shape&` dispatches to `Circle::area`.

### Real-World Cases

- **Interfaces** – `Shape`, `Animal`, `Comparable`.
- **Framework hooks** – abstract base classes with virtual methods.
- **Plug-in systems** – abstract interfaces for extensions.

---

## 11. `override`

### Definitions

**Core Definition**
`override` is a C++11 specifier that **explicitly marks a function as overriding a virtual function** from a base class, causing a compile-time error if no such virtual function exists.

**Technical Definition**
The `override` specifier is placed after the parameter list of a member function. It tells the compiler that the function is intended to override a virtual function from a base class. If no matching virtual function exists, the compiler issues an error. It does not change the function's behavior; it is a safety and documentation tool.

**Beginner-Friendly Explanation**
`override` is a **safety check** for virtual function overriding. It tells the compiler: “I intend to override a base-class virtual function.” If you made a typo or signature mistake, the compiler catches it.

### Purposes

- To catch signature mismatches when overriding.
- To document overriding intent.
- To prevent accidental hiding of virtual functions.
- To improve code safety and readability.

### Syntax Rules and Structure

**General Syntax**
```cpp
class Derived : public Base {
public:
    void f() override;              // override virtual function
    void g() const override;        // const virtual function
};
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `override` | Specifier after the parameter list. |
| Matching signature | Must match a base virtual function. |

**Syntax Rules**
- `override` requires a matching virtual function in a base class.
- It is placed after the parameter list and cv/ref qualifiers.
- It cannot be used on non-virtual functions or functions that don't override.
- It implies `virtual`.

**Constraints and Limitations**
- `override` does not work if the base function is not virtual.
- It is a compile-time check only.

### Annotated Code Examples

**Example 1: `override` catching a mistake**

```cpp
#include <iostream>

class Base {
public:
    virtual void f(int) { std::cout << "Base::f(int)\n"; }
    virtual ~Base() = default;
};

class Derived : public Base {
public:
    // void f(double) override;   // error: does not override Base::f(int)
    void f(int) override { std::cout << "Derived::f(int)\n"; }
};

int main() {
    Derived d;
    Base& b = d;
    b.f(42);
}
```

**Expected Output**
```
Derived::f(int)
```

**Why this output?**  
`Derived::f(int) override` correctly overrides `Base::f(int)`. The commented `f(double)` would be an error.

### Real-World Cases

- **Polymorphic hierarchies** – ensuring correct overrides.
- **Framework subclassing** – overriding virtual hooks.
- **Code review** – `override` is required by many style guides.

---

## 12. `final`

### Definitions

**Core Definition**
`final` is a C++11 specifier that **prevents further overriding** of a virtual function or **prevents further derivation** from a class.

**Technical Definition**
`final` can be applied to a virtual function (`void f() final;`) or a class (`class X final { ... };`). Attempting to override a `final` function or derive from a `final` class is a compile-time error. `final` can enable compiler optimizations (devirtualization).

**Beginner-Friendly Explanation**
`final` means **“no more changes.”** On a function, it says “no derived class can override this.” On a class, it says “no one can inherit from this.”

### Purposes

- To prevent further overriding of a virtual function.
- To prevent derivation from a class.
- To document design decisions.
- To enable compiler optimizations (devirtualization).

### Syntax Rules and Structure

**General Syntax**
```cpp
class Derived : public Base {
public:
    void f() final;        // no further override
};

class Sealed final { ... };   // no further derivation
```

**Component Breakdown**

| Form | Meaning |
|------|---------|
| `void f() final;` | Function cannot be overridden. |
| `class X final { ... };` | Class cannot be derived from. |

**Syntax Rules**
- `final` on a function requires the function to be virtual.
- `final` on a class prevents derivation.
- `final` can be combined with `override`.
- `final` is a contextual keyword.

**Constraints and Limitations**
- `final` on a non-virtual function is an error.
- `final` classes cannot be used as base classes.

### Annotated Code Examples

**Example 1: `final` function and class**

```cpp
#include <iostream>

class Base {
public:
    virtual void f() { std::cout << "Base::f\n"; }
    virtual ~Base() = default;
};

class Derived : public Base {
public:
    void f() final { std::cout << "Derived::f\n"; }
};

class Sealed final {};

// class Further : public Derived { void f() override {} };  // error
// class More : public Sealed {};                             // error

int main() {
    Derived d;
    Base& b = d;
    b.f();
}
```

**Expected Output**
```
Derived::f
```

**Why this output?**  
`Derived::f` is `final`, so no further overriding is allowed. The call through `Base&` dispatches to `Derived::f`.

### Real-World Cases

- **Sealed interfaces** – preventing unintended derivation.
- **Performance** – `final` can enable devirtualization.
- **Design documentation** – marking stable implementations.

---

## 13. Polymorphism Best Practices

### Definitions

**Core Definition**
Polymorphism best practices are guidelines for using compile-time and runtime polymorphism correctly, safely, and efficiently, based on the C++ Core Guidelines.

**Technical Definition**
The C++ Core Guidelines provide polymorphic design rules: **C.120** (use class hierarchies for inherent hierarchical concepts), **C.121** (if a base class is used polymorphically, make it abstract), **C.127** (a class with a virtual function should have a virtual or protected destructor), **C.128** (virtual functions should specify exactly one of `virtual`, `override`, or `final`), **C.129** (distinguish implementation from interface inheritance), **C.130** (use multiple inheritance only for interface composition), **C.132** (don't make functions virtual without reason), **C.133** (avoid `protected` data), **C.134** (ensure all non-`const` data members have the same access level), **C.135** (use multiple inheritance to represent multiple distinct interfaces), **C.137** (use `virtual` base classes only for interface inheritance), **C.138** (create an overload set for a derived class and its base), **C.139** (use `final` sparingly), and **C.140** (avoid adding virtual functions to base classes with existing derived classes).

**Beginner-Friendly Explanation**
Best practices for polymorphism are simple: **make base classes abstract**, **use virtual destructors**, **use `override` and `final`**, **don't make functions virtual without reason**, **prefer composition when inheritance isn't is-a**, and **avoid slicing**. These habits make polymorphic code safe and efficient.

### Key Guidelines (Summary Table)

| Guideline | Summary | Recommendation |
|-----------|---------|----------------|
| **C.121** | If a base class is used polymorphically, make it abstract. | Abstract base classes. |
| **C.127** | A class with a virtual function should have a virtual or protected destructor. | Virtual destructors. |
| **C.128** | Virtual functions should specify exactly one of `virtual`, `override`, or `final`. | Use `override` in derived classes. |
| **C.129** | Distinguish implementation inheritance from interface inheritance. | Interfaces for contracts. |
| **C.132** | Don't make functions virtual without reason. | Virtual functions have overhead. |
| **C.138** | Create an overload set for a derived class and its base. | Avoid hiding base overloads. |
| **C.139** | Use `final` sparingly. | Seal only when justified. |
| **Avoid slicing** | Copying derived objects into base objects loses derived parts. | Use pointers/references or `clone()`. |
| **Prefer composition** | Use inheritance only for is-a. | Composition for has-a. |

### Annotated Code Examples

**Example 1: Abstract base with virtual destructor (C.121, C.127)**

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
`Shape` is abstract (C.121) and has a virtual destructor (C.127). `Square` uses `override` for safety (C.128).

**Example 2: Avoiding slicing**

```cpp
#include <iostream>
#include <memory>
#include <vector>

class Base {
public:
    virtual void f() const { std::cout << "Base\n"; }
    virtual ~Base() = default;
};

class Derived : public Base {
public:
    void f() const override { std::cout << "Derived\n"; }
};

int main() {
    std::vector<std::unique_ptr<Base>> v;
    v.push_back(std::make_unique<Derived>());
    for (const auto& p : v) p->f();   // Derived
}
```

**Expected Output**
```
Derived
```

**Why this is best practice?**  
Using pointers to base classes preserves polymorphism; copying `Derived` into a `Base` object would slice away the derived part.

### Real-World Cases

- **Standard library** – `std::exception` has a virtual destructor; streams use virtual inheritance.
- **Code review** – Core Guidelines C.120–C.140 are common checkpoints.
- **Static analysis** – clang-tidy enforces many polymorphism rules.

---

## References

- Polymorphism (cppreference) – https://en.cppreference.com/w/cpp/language/derived_class
- Virtual functions (cppreference) – https://en.cppreference.com/w/cpp/language/virtual
- Virtual tables (cppreference, ABI) – https://itanium-cxx-abi.github.io/cxx-abi/abi.html#vtable
- `override` specifier (cppreference) – https://en.cppreference.com/w/cpp/language/override
- `final` specifier (cppreference) – https://en.cppreference.com/w/cpp/language/final
- Abstract class (cppreference) – https://en.cppreference.com/w/cpp/language/abstract_class
- Pure virtual function (cppreference) – https://en.cppreference.com/w/cpp/language/abstract_class
- Virtual destructors (cppreference) – https://en.cppreference.com/w/cpp/language/destructor#Virtual_destructors
- C++ Core Guidelines — C.120: Use class hierarchies to represent concepts with inherent hierarchical structure – https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rh-domain
- C++ Core Guidelines — C.121: If a base class is used polymorphically, make it abstract – https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rh-abstract
- C++ Core Guidelines — C.127: A class with a virtual function should have a virtual or protected destructor – https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rh-virtual-dtor
- C++ Core Guidelines — C.128: Virtual functions should specify exactly one of `virtual`, `override`, or `final` – https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rh-override
- C++ Core Guidelines — C.129: When designing a class hierarchy, distinguish between implementation inheritance and interface inheritance – https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rh-kind
- C++ Core Guidelines — C.132: Don't make functions virtual without reason – https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rh-virtual
- C++ Core Guidelines — C.138: Create an overload set for a derived class and its base – https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rh-using
- C++ Core Guidelines — C.139: Use `final` sparingly – https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rh-final
- C++ Core Guidelines — C.140: Do not provide different default arguments for a virtual function and an overrider – https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rh-virtual-default-arg
- Microsoft Learn — Virtual functions – https://learn.microsoft.com/en-us/cpp/cpp/virtual-functions
- Microsoft Learn — Polymorphism (C++) – https://learn.microsoft.com/en-us/cpp/cpp/polymorphism-cpp
- Microsoft Learn — Abstract classes – https://learn.microsoft.com/en-us/cpp/cpp/abstract-classes-cpp
- Itanium C++ ABI — Virtual tables – https://itanium-cxx-abi.github.io/cxx-abi/abi.html#vtable

---

This cheat sheet provides a complete, structured reference for C++ polymorphism, covering compile-time and runtime polymorphism, virtual functions, vtables, virtual function pointers, dynamic dispatch, virtual destructors, pure virtual functions, abstract classes, `override`, `final`, and best practices. Each section follows a uniform template with definitions, purposes, syntax rules, annotated examples, expected outputs, and authoritative references.