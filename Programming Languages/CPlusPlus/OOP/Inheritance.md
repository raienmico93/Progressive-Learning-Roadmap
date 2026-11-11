# Inheritance — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
Inheritance is a C++ mechanism that allows a **derived class** to acquire the members (data and functions) of a **base class**, enabling code reuse, hierarchical classification, and runtime polymorphism.

**Technical Definition**
Inheritance establishes an *is-a* relationship between a base class and a derived class. The derived class inherits the base class's members and may add new members, override virtual functions, and access `protected` members. Access control for inheritance (`public`, `protected`, `private`) determines how base-class members are accessible through the derived class. C++ supports single inheritance, multiple inheritance, and virtual inheritance (to resolve the diamond problem). Virtual functions and abstract classes enable runtime polymorphism; `override` and `final` (C++11) make overriding explicit and prevent further derivation.

**Beginner-Friendly Explanation**
Inheritance is a way to build a **new class from an existing one**. The new class (derived) gets everything the old class (base) has, and can add its own features or change some behaviors. It models “is-a” relationships: a `Dog` **is an** `Animal`, so `Dog` can inherit from `Animal`. Inheritance lets you avoid rewriting common code and enables **polymorphism** — treating different derived objects through a common base interface.

### Key Characteristics

- **Is-a relationship**: Derived is a kind of base.
- **Member inheritance**: Derived classes inherit data members and member functions (subject to access).
- **Access control**: `public`, `protected`, and `private` inheritance control base-member accessibility.
- **Virtual functions**: Enable runtime polymorphism through base-class pointers/references.
- **Abstract classes**: Classes with pure virtual functions cannot be instantiated.
- **Multiple inheritance**: A class can inherit from multiple bases (with complications).
- **Virtual inheritance**: Solves the diamond problem by sharing a single base subobject.
- **`override`/`final`**: Make overriding explicit and prevent further overriding/derivation.

### Prerequisites

- Understanding of classes, objects, and member functions.
- Familiarity with constructors, destructors, and access specifiers.
- Knowledge of pointers and references (for polymorphic use).
- Basic awareness of virtual functions and vtables.

### Related Programming Areas

- **Polymorphism** – virtual functions and dynamic dispatch.
- **Abstract classes and interfaces** – pure virtual functions.
- **Composition** – an alternative to inheritance for “has-a” relationships.
- **Multiple inheritance** – mixins, interface implementation.
- **Virtual inheritance** – solving the diamond problem.
- **Design patterns** – inheritance underpins many patterns (Template Method, Strategy, etc.).

### Core Concepts / Features

1. Inheritance
2. Base Classes
3. Derived Classes
4. `public` Inheritance
5. `protected` Inheritance
6. `private` Inheritance
7. Multiple Inheritance
8. Virtual Inheritance
9. Diamond Problem
10. `override`
11. `final`
12. Abstract Classes
13. Pure Virtual Functions
14. Interface Classes
15. Inheritance Best Practices
16. Composition over Inheritance

---

## 1. Inheritance

### Definitions

**Core Definition**
Inheritance is the mechanism by which a derived class acquires the properties and behaviors of a base class, establishing an is-a relationship.

**Technical Definition**
Inheritance is specified in the base-clause of a class definition: `class Derived : access-specifier Base { ... };`. The derived class inherits all members of the base class (except constructors, destructors, and assignment operators, which have special rules). The derived class can access `public` and `protected` members of the base, override virtual functions, and add new members. The base class subobject is constructed before the derived class members and destroyed after them.

**Beginner-Friendly Explanation**
Inheritance is **building on top of an existing class**. You create a new class that “inherits” everything from an old one, then add or customize. It's like starting with a basic recipe and adding your own twists.

### Purposes

- To reuse code from existing classes.
- To model hierarchical “is-a” relationships.
- To enable runtime polymorphism through base-class interfaces.
- To support extensibility and open/closed design.
- To organize related types into a coherent hierarchy.

### Syntax Rules and Structure

**General Syntax**
```cpp
class Derived : public Base {
    // additional members
};
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `class Derived` | The derived class. |
| `: public Base` | Base-clause with access specifier and base class. |
| `{ ... }` | Additional members of the derived class. |

**Syntax Rules**
- The base-clause specifies base classes and inheritance access.
- Constructors and destructors are not inherited (constructors can be inherited via `using`).
- The base class subobject is constructed before derived members.
- The derived class must have access to the base class's constructors.

**Constraints and Limitations**
- Inheritance creates tight coupling between base and derived.
- The base class must be complete (defined) when used as a base.
- Private base members are not accessible in derived classes.

### Annotated Code Examples

**Example 1: Basic inheritance**

```cpp
#include <iostream>

class Animal {
public:
    void eat() const { std::cout << "eating\n"; }
};

class Dog : public Animal {
public:
    void bark() const { std::cout << "woof\n"; }
};

int main() {
    Dog d;
    d.eat();    // inherited from Animal
    d.bark();   // Dog's own method
}
```

**Expected Output**
```
eating
woof
```

**Why this output?**  
`Dog` inherits `eat()` from `Animal` and adds `bark()`. Both are accessible on a `Dog` object.

### Real-World Cases

- **GUI frameworks** – `Button` derives from `Widget`.
- **Game entities** – `Player` and `Enemy` derive from `Entity`.
- **Shapes** – `Circle` and `Square` derive from `Shape`.
- **Streams** – `ifstream` derives from `istream`.

---

## 2. Base Classes

### Definitions

**Core Definition**
A base class (or parent class) is a class from which other classes derive, providing common members and interface.

**Technical Definition**
A base class is specified in the base-clause of a derived class. It can be `public`, `protected`, or `private` inherited. The base class's subobject is part of every derived object. Virtual functions in the base enable polymorphism; virtual destructors ensure correct cleanup. A base class can be abstract (with pure virtual functions) or concrete.

**Beginner-Friendly Explanation**
A base class is the **parent** in the inheritance relationship. It provides common features that derived classes share. `Animal` is a base class for `Dog` and `Cat`.

### Purposes

- To provide common data and behavior for derived classes.
- To define an interface for polymorphic use.
- To enable code reuse across related types.
- To establish a hierarchy of related concepts.

### Syntax Rules and Structure

**General Syntax**
```cpp
class Base {
public:
    virtual void f();
    virtual ~Base() = default;
};
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| Members | Common data and functions. |
| Virtual functions | Overridable behavior. |
| Virtual destructor | Correct polymorphic deletion. |

**Syntax Rules**
- A base class can be abstract or concrete.
- Virtual functions are overridden in derived classes.
- The destructor should be virtual if polymorphic deletion is possible.
- Base class constructors are called before derived constructors.

**Constraints and Limitations**
- Base classes should not have non-virtual destructors if used polymorphically.
- Base classes with data members can cause slicing if copied by value.
- Deep hierarchies can become fragile.

### Annotated Code Examples

**Example 1: Base class with virtual function**

```cpp
#include <iostream>

class Shape {
public:
    virtual double area() const = 0;   // pure virtual
    virtual ~Shape() = default;        // virtual destructor
};

class Square : public Shape {
    double side_;
public:
    explicit Square(double side) : side_(side) {}
    double area() const override { return side_ * side_; }
};

int main() {
    Square s(3.0);
    std::cout << s.area() << '\n';
}
```

**Expected Output**
```
9
```

**Why this output?**  
`Shape` is an abstract base class. `Square` overrides `area()` to compute `3.0 * 3.0 = 9`.

### Real-World Cases

- **Abstract interfaces** – `Shape`, `Animal`, `Widget`.
- **Framework base classes** – `QObject`, `std::exception`.
- **Mixins** – small base classes providing specific functionality.

---

## 3. Derived Classes

### Definitions

**Core Definition**
A derived class (or child class) is a class that inherits from one or more base classes, acquiring their members and adding or overriding behavior.

**Technical Definition**
A derived class is declared with a base-clause: `class Derived : access Base { ... };`. It inherits all members of the base (subject to access), can override virtual functions, and can add new members. The derived class's constructor calls the base class constructor (explicitly or implicitly) before initializing its own members. The derived class's destructor runs before the base class destructor.

**Beginner-Friendly Explanation**
A derived class is the **child** in the inheritance relationship. It gets everything from the parent and can add its own features or change some behaviors. `Dog` is a derived class of `Animal`.

### Purposes

- To reuse base-class code.
- To extend or specialize base-class behavior.
- To implement abstract interfaces.
- To add type-specific data and operations.

### Syntax Rules and Structure

**General Syntax**
```cpp
class Derived : public Base {
public:
    Derived(args) : Base(base_args), member_(value) {}
    void f() override;   // override virtual function
};
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `: public Base` | Base-clause. |
| Constructor | Calls base constructor. |
| `override` | Marks overridden virtual function. |
| New members | Additional data and functions. |

**Syntax Rules**
- The derived class constructor must initialize the base class (explicitly or implicitly).
- The derived class can override virtual functions (with `override` for safety).
- The derived class can access `public` and `protected` base members.
- The derived class destructor runs before the base destructor.

**Constraints and Limitations**
- The base class must be accessible and complete.
- The derived class cannot access private base members.
- Slicing occurs if a derived object is copied into a base object.

### Annotated Code Examples

**Example 1: Derived class overriding a virtual function**

```cpp
#include <iostream>

class Animal {
public:
    virtual void speak() const { std::cout << "...\n"; }
    virtual ~Animal() = default;
};

class Dog : public Animal {
public:
    void speak() const override { std::cout << "Woof!\n"; }
};

int main() {
    Animal* a = new Dog;
    a->speak();   // dynamic dispatch
    delete a;
}
```

**Expected Output**
```
Woof!
```

**Why this output?**  
`Dog` overrides `speak()`. The call through `Animal*` dispatches to `Dog::speak()` at runtime.

### Real-World Cases

- **Specialized types** – `SavingsAccount` derives from `Account`.
- **Concrete implementations** – `ArrayList` derives from `List`.
- **Framework subclasses** – user-defined widgets derive from `Widget`.

---

## 4. `public` Inheritance

### Definitions

**Core Definition**
`public` inheritance models an **is-a** relationship: the derived class is a kind of base class, and public members of the base remain public in the derived class.

**Technical Definition**
With `public` inheritance, `public` and `protected` members of the base retain their access in the derived class: `public` stays `public`, `protected` stays `protected`. The derived class can be used wherever the base class is expected (upcasting). This is the most common form of inheritance and the only one that supports the is-a relationship and polymorphic use.

**Beginner-Friendly Explanation**
`public` inheritance means **“is a.”** A `Dog` publicly inherits from `Animal` because a `Dog` **is an** `Animal`. Anyone who can use an `Animal` can also use a `Dog`.

### Purposes

- To model is-a relationships.
- To enable polymorphic use through base-class pointers/references.
- To allow derived objects to be used as base objects.
- To support interface implementation.

### Syntax Rules and Structure

**General Syntax**
```cpp
class Derived : public Base { ... };
```

**Component Breakdown**

| Base Member Access | Access in Derived |
|--------------------|-------------------|
| `public` | `public` |
| `protected` | `protected` |
| `private` | inaccessible |

**Syntax Rules**
- Public inheritance preserves the base's public interface.
- Derived objects can bind to base references/pointers.
- Public inheritance is the only form that supports polymorphism for external users.

**Constraints and Limitations**
- Slicing occurs when copying a derived object into a base object.
- The base class must have a virtual destructor for polymorphic deletion.

### Annotated Code Examples

**Example 1: `public` inheritance**

```cpp
#include <iostream>

class Animal {
public:
    void breathe() const { std::cout << "breathing\n"; }
};

class Dog : public Animal {
public:
    void bark() const { std::cout << "woof\n"; }
};

int main() {
    Dog d;
    d.breathe();   // accessible: public in base
    d.bark();

    Animal& a = d;   // upcast: Dog is-a Animal
    a.breathe();
}
```

**Expected Output**
```
breathing
woof
breathing
```

**Why this output?**  
`public` inheritance keeps `breathe()` public in `Dog`, and allows a `Dog` to bind to an `Animal&`.

### Real-World Cases

- **Polymorphic hierarchies** – `Derived` used as `Base`.
- **Interface implementation** – classes implementing abstract interfaces.
- **Standard library** – `std::ifstream` publicly inherits from `std::istream`.

---

## 5. `protected` Inheritance

### Definitions

**Core Definition**
`protected` inheritance makes `public` and `protected` members of the base **protected** in the derived class, preventing external code from treating the derived class as a base.

**Technical Definition**
With `protected` inheritance, `public` and `protected` members of the base become `protected` in the derived class; `private` members remain inaccessible. External code cannot convert a derived object to a base reference/pointer, but further derived classes can access the inherited members. This models an “implemented-in-terms-of” relationship rather than an is-a relationship.

**Beginner-Friendly Explanation**
`protected` inheritance is like saying: “My derived class uses the base class internally, but outsiders shouldn't treat it as a base class.” It's rarely used; it's a middle ground between `public` and `private` inheritance.

### Purposes

- To express “implemented-in-terms-of” while allowing further derivation.
- To share base functionality with derived classes without exposing it publicly.
- To control upcasting to the base.

### Syntax Rules and Structure

**General Syntax**
```cpp
class Derived : protected Base { ... };
```

**Component Breakdown**

| Base Member Access | Access in Derived |
|--------------------|-------------------|
| `public` | `protected` |
| `protected` | `protected` |
| `private` | inaccessible |

**Syntax Rules**
- External code cannot upcast to the base.
- Further derived classes can access the inherited members.
- The base's public interface is not part of the derived class's public interface.

**Constraints and Limitations**
- Rarely the right choice; often `private` inheritance or composition is clearer.
- The derived class cannot be used polymorphically through the base externally.

### Annotated Code Examples

**Example 1: `protected` inheritance**

```cpp
#include <iostream>

class Base {
public:
    void public_fn() const { std::cout << "public\n"; }
protected:
    void protected_fn() const { std::cout << "protected\n"; }
};

class Derived : protected Base {
public:
    void use() const {
        public_fn();      // accessible inside Derived
        protected_fn();
    }
};

int main() {
    Derived d;
    d.use();
    // d.public_fn();     // error: not accessible externally
    // Base& b = d;       // error: not accessible externally
}
```

**Expected Output**
```
public
protected
```

**Why this output?**  
`protected` inheritance makes `public_fn` protected in `Derived`, so it's accessible inside `Derived` but not externally.

### Real-World Cases

- **Rare** – mostly used in specialized designs.
- **Implementation inheritance** – sharing code with further derivations.
- **Framework internals** – controlled access.

---

## 6. `private` Inheritance

### Definitions

**Core Definition**
`private` inheritance makes all inherited `public` and `protected` members **private** in the derived class, modeling an “implemented-in-terms-of” relationship.

**Technical Definition**
With `private` inheritance, `public` and `protected` members of the base become `private` in the derived class; `private` members remain inaccessible. External code cannot convert a derived object to a base reference/pointer, and further derived classes cannot access the inherited members. Private inheritance is essentially a form of composition with some extra features (access to protected members, virtual function overriding).

**Beginner-Friendly Explanation**
`private` inheritance means “my derived class is **built using** the base class, but it's not a kind of base class.” It's like composition, but with access to the base's protected members. It is rarely used; composition is usually clearer.

### Purposes

- To model “implemented-in-terms-of” with access to protected members.
- To override virtual functions without exposing the base interface.
- To reuse implementation details privately.

### Syntax Rules and Structure

**General Syntax**
```cpp
class Derived : private Base { ... };
```

**Component Breakdown**

| Base Member Access | Access in Derived |
|--------------------|-------------------|
| `public` | `private` |
| `protected` | `private` |
| `private` | inaccessible |

**Syntax Rules**
- External code cannot upcast to the base.
- Further derived classes cannot access inherited members.
- The derived class can access the base's `public` and `protected` members internally.
- The derived class can override virtual functions.

**Constraints and Limitations**
- Prefer composition when possible (Core Guidelines C.120, C.121).
- Private inheritance is rarely the best design choice.

### Annotated Code Examples

**Example 1: `private` inheritance**

```cpp
#include <iostream>

class Base {
public:
    void public_fn() const { std::cout << "public\n"; }
protected:
    void protected_fn() const { std::cout << "protected\n"; }
};

class Derived : private Base {
public:
    void use() const {
        public_fn();      // accessible inside Derived
        protected_fn();
    }
};

int main() {
    Derived d;
    d.use();
    // d.public_fn();     // error: private in Derived
    // Base& b = d;       // error: not accessible
}
```

**Expected Output**
```
public
protected
```

**Why this output?**  
`private` inheritance makes the base's members private in `Derived`. They're accessible inside `Derived` but not externally.

### Real-World Cases

- **Rare** – mostly superseded by composition.
- **Legacy designs** – pre-composition idioms.
- **Virtual function overriding** – privately inheriting to override virtuals.

---

## 7. Multiple Inheritance

### Definitions

**Core Definition**
Multiple inheritance allows a class to inherit from **more than one base class**, combining their members and interfaces.

**Technical Definition**
A class can have multiple base classes in its base-clause: `class Derived : public Base1, public Base2 { ... };`. The derived class inherits members from all bases. Ambiguities arise when multiple bases have members with the same name; the derived class must disambiguate with qualified names. Multiple inheritance is used for combining interfaces (mixins) and, less commonly, implementations. The diamond problem occurs when two bases share a common base; virtual inheritance solves it.

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

**Example 1: Multiple inheritance with interfaces**

```cpp
#include <iostream>

class Serializable {
public:
    virtual void serialize() const = 0;
    virtual ~Serializable() = default;
};

class Comparable {
public:
    virtual bool operator<(const Comparable&) const = 0;
    virtual ~Comparable() = default;
};

class Record : public Serializable, public Comparable {
    int id_;
public:
    explicit Record(int id) : id_(id) {}
    void serialize() const override { std::cout << "Record(" << id_ << ")\n"; }
    bool operator<(const Comparable&) const override { return true; }
};

int main() {
    Record r(42);
    r.serialize();
}
```

**Expected Output**
```
Record(42)
```

**Why this output?**  
`Record` inherits from two interfaces and implements both. The multiple inheritance combines the interfaces.

### Real-World Cases

- **Interface composition** – combining multiple abstract interfaces.
- **Mixins** – adding functionality from multiple bases.
- **Framework classes** – inheriting from both a base and an interface.

---

## 8. Virtual Inheritance

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

## 9. Diamond Problem

### Definitions

**Core Definition**
The diamond problem is the ambiguity that arises when a class inherits from two classes that share a common base, resulting in **two copies** of the base subobject.

**Technical Definition**
In a diamond hierarchy `D : B, C` where `B : A` and `C : A`, `D` contains two `A` subobjects (one via `B`, one via `C`). Accessing an `A` member through `D` is ambiguous; the programmer must qualify (`d.B::member` or `d.C::member`) or use virtual inheritance to share a single `A` subobject. Virtual inheritance eliminates the duplication but adds complexity.

**Beginner-Friendly Explanation**
The diamond problem is the **“two copies of grandpa”** problem. If `D` inherits from `B` and `C`, and both inherit from `A`, then `D` has two copies of `A`. When you say `d.value`, the compiler doesn't know which copy you mean. Virtual inheritance fixes this by making `D` have just one shared copy.

### Purposes

- (This is a problem to solve.) Understanding it motivates virtual inheritance.
- To recognize ambiguity in multiple inheritance hierarchies.
- To design hierarchies that avoid duplication.

### Syntax Rules and Structure

**Problem Pattern**
```cpp
class A { public: int value; };
class B : public A {};
class C : public A {};
class D : public B, public C {};   // two A subobjects
```

**Solution Pattern**
```cpp
class B : virtual public A {};
class C : virtual public A {};
class D : public B, public C {};   // one shared A
```

**Component Breakdown**

| Pattern | A Subobjects in D |
|---------|-------------------|
| Non-virtual | Two (ambiguous) |
| Virtual | One (shared) |

**Syntax Rules**
- Without virtual inheritance, access to `A` members through `D` is ambiguous.
- Disambiguate with `d.B::value` or `d.C::value`.
- Virtual inheritance shares a single `A`.

**Constraints and Limitations**
- Virtual inheritance adds overhead.
- The most-derived class must initialize the virtual base.

### Annotated Code Examples

**Example 1: Diamond problem without virtual inheritance**

```cpp
#include <iostream>

class A { public: int value = 1; };
class B : public A {};
class C : public A {};
class D : public B, public C {};

int main() {
    D d;
    // d.value = 42;   // error: ambiguous
    d.B::value = 10;   // disambiguate
    d.C::value = 20;
    std::cout << d.B::value << ' ' << d.C::value << '\n';
}
```

**Expected Output**
```
10 20
```

**Why this output?**  
`D` has two `A` subobjects. `d.value` is ambiguous; `d.B::value` and `d.C::value` access the separate copies.

**Example 2: Diamond problem solved with virtual inheritance**

```cpp
#include <iostream>

class A { public: int value = 1; };
class B : virtual public A {};
class C : virtual public A {};
class D : public B, public C {};

int main() {
    D d;
    d.value = 42;    // unambiguous: one shared A
    std::cout << d.value << '\n';
}
```

**Expected Output**
```
42
```

**Why this output?**  
Virtual inheritance gives `D` a single shared `A`, so `d.value` is unambiguous.

### Real-World Cases

- **I/O stream hierarchy** – `std::iostream` inherits from `std::istream` and `std::ostream`, both virtually inheriting from `std::basic_ios`.
- **Framework hierarchies** – shared base state across multiple paths.
- **Multiple interface implementation** – avoiding duplicate base state.

---

## 10. `override`

### Definitions

**Core Definition**
`override` is a C++11 specifier that **explicitly marks a function as overriding a virtual function** from a base class, causing a compile-time error if no such virtual function exists.

**Technical Definition**
The `override` specifier is placed after the parameter list of a member function (and after `const`/`volatile`/ref-qualifiers). It tells the compiler that the function is intended to override a virtual function from a base class. If no matching virtual function exists (e.g., due to a signature mismatch), the compiler issues an error. It does not change the function's behavior; it is a safety and documentation tool.

**Beginner-Friendly Explanation**
`override` is a **safety check** for virtual function overriding. It tells the compiler: “I intend to override a base-class virtual function.” If you made a typo or signature mistake, the compiler catches it instead of silently creating a new function.

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
- It implies `virtual` (the function is virtual even without the keyword).

**Constraints and Limitations**
- `override` does not work if the base function is not virtual.
- It is a compile-time check only; it does not affect runtime behavior.

### Annotated Code Examples

**Example 1: `override` catching a signature mistake**

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
`Derived::f(int) override` correctly overrides `Base::f(int)`. The commented `f(double)` would be an error because it doesn't override the base's `f(int)`.

### Real-World Cases

- **Polymorphic hierarchies** – ensuring correct overrides.
- **Framework subclassing** – overriding virtual hooks.
- **Code review** – `override` is required by many style guides.

---

## 11. `final`

### Definitions

**Core Definition**
`final` is a C++11 specifier that **prevents further overriding** of a virtual function or **prevents further derivation** from a class.

**Technical Definition**
The `final` specifier can be applied to:
- A virtual function: `void f() final;` — no derived class may override `f`.
- A class: `class X final { ... };` — no class may derive from `X`.

Attempting to override a `final` function or derive from a `final` class is a compile-time error. `final` is used to seal interfaces or implementations and can enable compiler optimizations (devirtualization).

**Beginner-Friendly Explanation**
`final` means **“no more changes.”** On a function, it says “no derived class can override this.” On a class, it says “no one can inherit from this.” It's useful for sealing designs and helping the compiler optimize.

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
    void f() final { std::cout << "Derived::f\n"; }   // no further override
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

## 12. Abstract Classes

### Definitions

**Core Definition**
An abstract class is a class that **cannot be instantiated** because it has at least one **pure virtual function**; it serves as an interface or base for derived classes.

**Technical Definition**
A class is abstract if it has at least one pure virtual function (declared with `= 0`). Abstract classes cannot be instantiated directly, but pointers and references to them can be used polymorphically. Derived classes must override all pure virtual functions to become concrete. Abstract classes often provide partial implementations and define interfaces.

**Beginner-Friendly Explanation**
An abstract class is a **blueprint that can't be built directly** — it's incomplete. You must derive from it and fill in the missing parts (pure virtual functions) before you can create objects. It defines an interface that derived classes must implement.

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
    virtual void f() = 0;   // pure virtual
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
- A pure virtual function can have a definition (callable via qualified name).

### Annotated Code Examples

**Example 1: Abstract class**

```cpp
#include <iostream>

class Shape {
public:
    virtual double area() const = 0;   // pure virtual
    virtual ~Shape() = default;
};

class Circle : public Shape {
    double radius_;
public:
    explicit Circle(double r) : radius_(r) {}
    double area() const override { return 3.14159 * radius_ * radius_; }
};

int main() {
    // Shape s;                  // error: abstract
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
`Shape` is abstract (pure virtual `area`). `Circle` implements `area`, so it is concrete. The call through `Shape&` dispatches to `Circle::area`.

### Real-World Cases

- **Interfaces** – `Shape`, `Animal`, `Comparable`.
- **Framework hooks** – abstract base classes with virtual methods.
- **Plug-in systems** – abstract interfaces for extensions.

---

## 13. Pure Virtual Functions

### Definitions

**Core Definition**
A pure virtual function is a virtual function declared with `= 0` that **must be overridden** by derived classes for them to be concrete.

**Technical Definition**
A pure virtual function is declared as `virtual return_type f(params) = 0;`. It makes the class abstract. The function may still have a definition (outside the class), which can be called via qualified name. Derived classes must override all pure virtual functions to become concrete. Pure virtual functions define the interface contract.

**Beginner-Friendly Explanation**
A pure virtual function is a **“you must implement this”** declaration. It says: “Every derived class must provide its own version of this function, or it can't be instantiated.” It's how interfaces are defined in C++.

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
| Definition | Optional (callable via `Interface::f`). |
| Override required | Derived classes must implement. |

**Syntax Rules**
- A pure virtual function makes the class abstract.
- It can have a definition (outside the class).
- Derived classes must override all pure virtual functions to be concrete.
- Pure virtual functions can be `const`, `noexcept`, etc.

**Constraints and Limitations**
- A pure virtual function with a definition is still pure virtual.
- Abstract classes cannot be instantiated.
- Pure virtual functions cannot be `final` (they must be overridable).

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
- **Template method pattern** – base class calls pure virtual hooks.
- **Abstract base classes** – contracts for derived implementations.

---

## 14. Interface Classes

### Definitions

**Core Definition**
An interface class is an abstract class with **only pure virtual functions** and a virtual destructor, defining a contract that derived classes must fulfill.

**Technical Definition**
An interface class (or protocol) is an abstract class with no data members, no non-virtual functions (except maybe a virtual destructor), and only pure virtual functions. It defines a pure interface. Derived classes implement the interface. Interface classes are used for multiple inheritance of interfaces, dependency inversion, and testability.

**Beginner-Friendly Explanation**
An interface class is a **pure contract** — it says what operations exist but provides no implementation. It's like a job description: anyone who takes the job (derived class) must fulfill all the duties (pure virtual functions).

### Purposes

- To define pure interfaces for polymorphism.
- To enable dependency inversion and testability.
- To support multiple interface implementation.
- To decouple clients from implementations.

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
- All member functions should be pure virtual (except destructor).
- Derived classes implement the interface.

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

## 15. Inheritance Best Practices

### Definitions

**Core Definition**
Inheritance best practices are guidelines for using inheritance correctly, safely, and idiomatically, based on the C++ Core Guidelines.

**Technical Definition**
The C++ Core Guidelines provide inheritance rules: **C.120** (use class hierarchies to represent concepts with inherent hierarchical structure), **C.121** (if a base class is used polymorphically, make it abstract), **C.122** (use abstract classes as interfaces), **C.126** (an abstract class typically doesn't need a user-written constructor), **C.127** (a class with a virtual function should have a virtual or protected destructor), **C.128** (virtual functions should specify exactly one of `virtual`, `override`, or `final`), **C.129** (when designing a class hierarchy, distinguish between implementation inheritance and interface inheritance), **C.130** (use multiple inheritance only for interface composition), **C.131** (avoid trivial getters and setters), **C.132** (don't make functions virtual without reason), **C.133** (avoid `protected` data), **C.134** (ensure all non-`const` data members have the same access level), **C.135** (use multiple inheritance to represent multiple distinct interfaces), **C.136** (use multiple inheritance to represent implementation inheritance only if the base classes are pure interfaces), **C.137** (use `virtual` base classes only for interface inheritance), **C.138** (create an overload set for a derived class and its base), **C.139** (use `final` sparingly), and **C.140** (avoid adding virtual functions to base classes with existing derived classes).

**Beginner-Friendly Explanation**
Best practices for inheritance are simple: **use inheritance for is-a relationships**, **make base classes abstract when possible**, **use virtual destructors**, **use `override` and `final`**, **avoid deep hierarchies**, **prefer composition when the relationship isn't is-a**, and **use multiple inheritance only for interfaces**.

### Key Guidelines (Summary Table)

| Guideline | Summary | Recommendation |
|-----------|---------|----------------|
| **C.120** | Use class hierarchies for inherently hierarchical concepts. | Inheritance for is-a. |
| **C.121** | If a base class is used polymorphically, make it abstract. | Abstract base classes. |
| **C.127** | A class with a virtual function should have a virtual or protected destructor. | Virtual destructors. |
| **C.128** | Virtual functions should specify exactly one of `virtual`, `override`, or `final`. | Use `override` in derived classes. |
| **C.129** | Distinguish implementation inheritance from interface inheritance. | Interfaces for contracts; composition for reuse. |
| **C.130** | Use multiple inheritance only for interface composition. | Multiple interfaces, single implementation. |
| **C.133** | Avoid `protected` data. | Private data with protected accessors. |
| **C.135** | Use multiple inheritance to represent multiple distinct interfaces. | Combine interfaces. |
| **C.139** | Use `final` sparingly. | Seal only when justified. |
| **C.140** | Avoid adding virtual functions to base classes with existing derived classes. | ABI compatibility. |

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

### Real-World Cases

- **Standard library** – `std::exception` has a virtual destructor; streams use virtual inheritance.
- **Code review** – Core Guidelines C.120–C.140 are common checkpoints.
- **Static analysis** – clang-tidy enforces many inheritance rules.

---

## 16. Composition over Inheritance

### Definitions

**Core Definition**
Composition over inheritance is the principle that, when possible, classes should **contain** other classes as members (has-a) rather than inherit from them (is-a), promoting flexibility and reducing coupling.

**Technical Definition**
Composition is a design technique where a class achieves functionality by containing instances of other classes as data members, rather than deriving from them. The C++ Core Guidelines state: **C.120** (use class hierarchies for is-a), **C.129** (distinguish implementation inheritance from interface inheritance), **C.130** (use multiple inheritance only for interface composition), and **C.121** (abstract base classes for polymorphism). The principle is often summarized as “favor composition over inheritance” (Gamma et al., *Design Patterns*).

**Beginner-Friendly Explanation**
Composition over inheritance means: **“has-a” is often better than “is-a.”** Instead of making `Car` inherit from `Engine` (which is wrong — a car isn't an engine), make `Car` **contain** an `Engine` as a member. Composition is more flexible and avoids the tight coupling of inheritance.

### Purposes

- To reduce coupling between classes.
- To increase flexibility (swap components at runtime).
- To avoid fragile base class problems.
- To model has-a relationships correctly.
- To enable easier testing and maintenance.

### Comparison Table

| Aspect | Inheritance | Composition |
|--------|-------------|-------------|
| Relationship | is-a | has-a |
| Coupling | Tight | Loose |
| Flexibility | Limited (fixed at compile time) | High (swap components) |
| Reuse | White-box (exposes internals) | Black-box (uses public interface) |
| Fragility | Fragile base class problem | Robust |
| Polymorphism | Virtual functions | Delegation or interfaces |

### Annotated Code Examples

**Example 1: Composition instead of inheritance**

```cpp
#include <iostream>

class Engine {
public:
    void start() const { std::cout << "Engine started\n"; }
};

// BAD: Car is not an Engine
// class Car : public Engine { ... };

// GOOD: Car has an Engine
class Car {
    Engine engine_;   // composition
public:
    void start() const {
        engine_.start();
        std::cout << "Car started\n";
    }
};

int main() {
    Car c;
    c.start();
}
```

**Expected Output**
```
Engine started
Car started
```

**Why this is better?**  
A `Car` **has an** `Engine`, not **is an** `Engine`. Composition models the relationship correctly and avoids inheriting irrelevant behavior.

**Example 2: Composition with interfaces for flexibility**

```cpp
#include <iostream>
#include <memory>

class Logger {
public:
    virtual void log(const std::string& msg) = 0;
    virtual ~Logger() = default;
};

class ConsoleLogger : public Logger {
public:
    void log(const std::string& msg) override {
        std::cout << "[console] " << msg << '\n';
    }
};

class Service {
    std::unique_ptr<Logger> logger_;   // composition
public:
    explicit Service(std::unique_ptr<Logger> logger) : logger_(std::move(logger)) {}
    void run() { logger_->log("running"); }
};

int main() {
    Service s(std::make_unique<ConsoleLogger>());
    s.run();
}
```

**Expected Output**
```
[console] running
```

**Why this is better?**  
`Service` **has a** `Logger` (composition), not **is a** `Logger`. The logger can be swapped for testing (e.g., a mock logger) without changing `Service`.

### Real-World Cases

- **Dependency injection** – services contain interface pointers.
- **Strategy pattern** – composition of algorithms.
- **Game engines** – entities compose components (ECS).
- **UI frameworks** – widgets contain layout managers.

---

## References

- Inheritance (cppreference) – https://en.cppreference.com/w/cpp/language/derived_class
- Access specifiers (cppreference) – https://en.cppreference.com/w/cpp/language/access
- Virtual inheritance (cppreference) – https://en.cppreference.com/w/cpp/language/derived_class#Virtual_base_classes
- Multiple inheritance (cppreference) – https://en.cppreference.com/w/cpp/language/derived_class#Multiple_inheritance
- `override` specifier (cppreference) – https://en.cppreference.com/w/cpp/language/override
- `final` specifier (cppreference) – https://en.cppreference.com/w/cpp/language/final
- Abstract class (cppreference) – https://en.cppreference.com/w/cpp/language/abstract_class
- Pure virtual function (cppreference) – https://en.cppreference.com/w/cpp/language/abstract_class
- C++ Core Guidelines — C.120: Use class hierarchies to represent concepts with inherent hierarchical structure – https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rh-domain
- C++ Core Guidelines — C.121: If a base class is used polymorphically, make it abstract – https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rh-abstract
- C++ Core Guidelines — C.126: An abstract class typically doesn't need a user-written constructor – https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rh-abstract-ctor
- C++ Core Guidelines — C.127: A class with a virtual function should have a virtual or protected destructor – https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rh-virtual-dtor
- C++ Core Guidelines — C.128: Virtual functions should specify exactly one of `virtual`, `override`, or `final` – https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rh-override
- C++ Core Guidelines — C.129: When designing a class hierarchy, distinguish between implementation inheritance and interface inheritance – https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rh-kind
- C++ Core Guidelines — C.130: For making deep copies of polymorphic classes prefer a virtual `clone` function – https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rh-copy
- C++ Core Guidelines — C.133: Avoid `protected` data – https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rh-protected
- C++ Core Guidelines — C.135: Use multiple inheritance to represent multiple distinct interfaces – https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rh-mi-interface
- C++ Core Guidelines — C.139: Use `final` sparingly – https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rh-final
- C++ Core Guidelines — C.140: Do not provide different default arguments for a virtual function and an overrider – https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rh-virtual-default-arg
- Microsoft Learn — Inheritance (C++) – https://learn.microsoft.com/en-us/cpp/cpp/inheritance-cpp
- Microsoft Learn — Virtual functions – https://learn.microsoft.com/en-us/cpp/cpp/virtual-functions
- Microsoft Learn — Multiple base classes – https://learn.microsoft.com/en-us/cpp/cpp/multiple-base-classes
- Design Patterns: Elements of Reusable Object-Oriented Software (Gamma et al.) — “Favor composition over inheritance” (book reference)

---

This cheat sheet provides a complete, structured reference for C++ inheritance, covering base and derived classes, `public`/`protected`/`private` inheritance, multiple inheritance, virtual inheritance, the diamond problem, `override`, `final`, abstract classes, pure virtual functions, interface classes, best practices, and composition over inheritance. Each section follows a uniform template with definitions, purposes, syntax rules, annotated examples, expected outputs, and authoritative references.