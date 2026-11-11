# Constructors and Destructors — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
Constructors are special member functions that **initialize objects** when they are created, establishing the class invariant. Destructors are special member functions that **clean up objects** when their lifetime ends, releasing resources.

**Technical Definition**
A constructor has the same name as its class, no return type, and is invoked automatically during object creation. It can be default, parameterized, copy, move, delegating, or inherited. Constructors run member initializer lists before their bodies, and members are initialized in declaration order (not initializer-list order). A destructor has the name `~ClassName`, no parameters, and no return type; it is invoked automatically when an object's lifetime ends (scope exit, `delete`, or destruction of a containing object). Destructors should not throw and must be virtual in polymorphic base classes to ensure correct deletion through base pointers.

**Beginner-Friendly Explanation**
A constructor is the **setup crew** that prepares an object when it's built — setting initial values, opening files, acquiring resources. A destructor is the **cleanup crew** that runs when the object is destroyed — closing files, freeing memory, releasing locks. C++ calls both automatically, which is what makes RAII and safe resource management possible. Constructors come in many flavors: some take no arguments (default), some take parameters, some copy or move from another object, and some delegate to other constructors to avoid duplication.

### Key Characteristics

- **Automatic invocation**: Constructors run at object creation; destructors run at object destruction.
- **No return type**: Constructors and destructors return nothing.
- **Member initializer list**: Constructors initialize members before the body runs.
- **Initialization order**: Members are initialized in **declaration order**, regardless of initializer-list order.
- **Multiple constructor forms**: default, parameterized, copy, move, delegating, inherited.
- **`explicit`**: Prevents implicit conversions from single-argument constructors.
- **Virtual destructors**: Required for polymorphic base classes.
- **Pure virtual destructors**: Make a class abstract while still providing a destructor definition.
- **Exception safety**: Constructors that throw leave no object; destructors must not throw.

### Prerequisites

- Understanding of classes, objects, and member functions.
- Familiarity with references, pointers, and value categories.
- Knowledge of RAII and resource management.
- Basic awareness of inheritance and polymorphism.

### Related Programming Areas

- **RAII** – constructors acquire, destructors release.
- **Move semantics** – move constructors transfer resources.
- **Rule of Three/Five/Zero** – governs special member functions.
- **Inheritance and polymorphism** – virtual destructors, inheriting constructors.
- **Exception safety** – constructor failure and destructor no-throw guarantees.
- **Object lifetime** – storage duration and destruction order.

### Core Concepts / Features

1. Constructors
2. Default Constructors
3. Parameterized Constructors
4. Copy Constructors
5. Move Constructors
6. Delegating Constructors
7. Inheriting Constructors
8. Member Initializer List
9. Constructor Initialization Order
10. Explicit Constructors / `explicit` Keyword
11. Destructors
12. Virtual Destructors
13. Pure Virtual Destructors
14. Constructor/Destructor Best Practices

---

## 1. Constructors

### Definitions

**Core Definition**
A constructor is a special member function that initializes an object when it is created.

**Technical Definition**
A constructor has the same name as its class, no return type, and may take parameters. It is invoked automatically during object creation (direct-initialization, copy-initialization, or value-initialization). Constructors can be overloaded, can be `explicit`, `constexpr`, `consteval`, or deleted, and can delegate to other constructors. If no constructor is declared, the compiler generates a default constructor (subject to rules). Constructors establish the class invariant; a constructor that throws means the object is not fully constructed.

**Beginner-Friendly Explanation**
A constructor is the **birth function** of an object. It runs automatically when the object is created and sets up its initial state — filling in data members, opening resources, and making sure the object is ready to use.

### Purposes

- To initialize data members to valid values.
- To establish the class invariant.
- To acquire resources (RAII).
- To provide multiple ways to create objects.
- To support copy and move semantics.

### Syntax Rules and Structure

**General Syntax**
```cpp
class X {
public:
    X();                          // default constructor
    X(int a);                     // parameterized constructor
    X(const X& other);            // copy constructor
    X(X&& other) noexcept;        // move constructor
    X(int a) : member_(a) {}      // member initializer list
};
```

**Component Breakdown**

| Form | Meaning |
|------|---------|
| `X()` | Default constructor. |
| `X(int)` | Parameterized constructor. |
| `X(const X&)` | Copy constructor. |
| `X(X&&)` | Move constructor. |
| `: member_(value)` | Member initializer list. |

**Syntax Rules**
- Constructors have no return type.
- The member initializer list runs before the constructor body.
- Members are initialized in declaration order, not initializer-list order.
- Constructors can be overloaded.
- A constructor can delegate to another constructor of the same class.
- Constructors can be `explicit` to prevent implicit conversions.

**Constraints and Limitations**
- A constructor that throws means the object is not constructed; its destructor is not called for the partially constructed object (but fully constructed members are destroyed).
- Virtual constructors are not allowed (use factory functions or `clone`).
- `const` and reference members must be initialized in the initializer list.

### Annotated Code Examples

**Example 1: Basic constructor**

```cpp
#include <iostream>

class Point {
    int x_, y_;
public:
    Point(int x, int y) : x_(x), y_(y) {   // constructor
        std::cout << "Point(" << x_ << ", " << y_ << ")\n";
    }
};

int main() {
    Point p(3, 4);
}
```

**Expected Output**
```
Point(3, 4)
```

**Why this output?**  
The constructor initializes `x_` and `y_` via the member initializer list and prints their values.

### Real-World Cases

- **Resource acquisition** – opening files, allocating memory.
- **Validation** – throwing on invalid parameters.
- **Default initialization** – setting sensible defaults.
- **Conversion** – `explicit` constructors for controlled conversions.

---

## 2. Default Constructors

### Definitions

**Core Definition**
A default constructor is a constructor that can be called with **no arguments**, initializing an object with default values.

**Technical Definition**
A default constructor is a constructor that can be invoked with no arguments (`X()`, `X(int a = 0)`, or `X(std::initializer_list<int> = {})`). The compiler generates a default constructor if no user-declared constructors exist. A defaulted default constructor is `constexpr` if possible. A default constructor is deleted if a member has no default constructor (e.g., a reference) or if the class has user-declared destructor/copy/move operations that suppress it.

**Beginner-Friendly Explanation**
A default constructor is the **no-arguments constructor**. It creates an object with default values. If you don't write any constructor, the compiler provides one (unless other rules apply).

### Purposes

- To create objects without arguments.
- To support arrays and containers of objects.
- To provide default initialization.
- To satisfy generic code requirements.

### Syntax Rules and Structure

**General Syntax**
```cpp
class X {
public:
    X();                 // user-declared default constructor
    X() = default;       // explicitly defaulted
};
```

**Component Breakdown**

| Form | Meaning |
|------|---------|
| `X()` | Default constructor. |
| `X() = default` | Compiler-generated default constructor. |
| `X() = delete` | No default constructor. |

**Syntax Rules**
- The default constructor can be `= default` or `= delete`.
- If no constructor is declared, the compiler generates a default constructor.
- A default constructor is `constexpr` if all members are initialized by constant expressions.

**Constraints and Limitations**
- The default constructor is deleted if a member has no default constructor.
- User-declared constructors suppress the implicit default constructor.

### Annotated Code Examples

**Example 1: Default constructor**

```cpp
#include <iostream>

class Point {
    int x_ = 0, y_ = 0;   // default member initializers
public:
    Point() = default;    // default constructor
    int x() const { return x_; }
    int y() const { return y_; }
};

int main() {
    Point p;
    std::cout << p.x() << ',' << p.y() << '\n';
}
```

**Expected Output**
```
0,0
```

**Why this output?**  
The defaulted default constructor uses the default member initializers, setting `x_` and `y_` to `0`.

### Real-World Cases

- **Containers** – `std::vector<Point> v(10);` requires a default constructor.
- **Arrays** – `Point arr[5];` requires a default constructor.
- **Generic code** – many templates assume default constructibility.

---

## 3. Parameterized Constructors

### Definitions

**Core Definition**
A parameterized constructor is a constructor that takes **one or more arguments**, allowing objects to be initialized with specific values.

**Technical Definition**
A parameterized constructor has parameters and initializes data members from those parameters (typically via the member initializer list). It can be overloaded with other constructors. A single-argument parameterized constructor can be `explicit` to prevent implicit conversions. Parameterized constructors enable flexible object creation.

**Beginner-Friendly Explanation**
A parameterized constructor is a **constructor with inputs**. It lets you create an object with specific values, like `Point p(3, 4);` — you're giving the object its initial coordinates.

### Purposes

- To initialize objects with specific values.
- To provide multiple ways to create objects.
- To enforce invariants through validation.
- To support flexible APIs.

### Syntax Rules and Structure

**General Syntax**
```cpp
class X {
public:
    X(int a, double b);                      // parameterized constructor
    X(int a, double b) : a_(a), b_(b) {}     // with initializer list
};
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| Parameters | Inputs to the constructor. |
| Initializer list | Initializes members from parameters. |
| Body | Optional additional logic. |

**Syntax Rules**
- Parameterized constructors can have default arguments.
- Single-argument constructors should be `explicit` by default (C.46).
- Constructors can be overloaded.
- The initializer list is preferred over assignment in the body.

**Constraints and Limitations**
- Ambiguous overloads can occur with default arguments.
- The initializer list order is declaration order, not parameter order.

### Annotated Code Examples

**Example 1: Parameterized constructor**

```cpp
#include <iostream>

class Rectangle {
    double width_, height_;
public:
    Rectangle(double w, double h) : width_(w), height_(h) {}   // parameterized
    double area() const { return width_ * height_; }
};

int main() {
    Rectangle r(3.0, 4.0);
    std::cout << r.area() << '\n';
}
```

**Expected Output**
```
12
```

**Why this output?**  
The parameterized constructor initializes `width_` and `height_` from the arguments; `area()` computes `3.0 * 4.0 = 12`.

### Real-World Cases

- **Domain objects** – `Person("Alice", 30)`.
- **Configuration** – `Config(800, 600)`.
- **Containers** – `std::vector<int> v(10, 0)` (size, value).

---

## 4. Copy Constructors

### Definitions

**Core Definition**
A copy constructor is a constructor that **initializes a new object as a copy** of an existing object of the same type.

**Technical Definition**
A copy constructor has the signature `X(const X& other)` (or `X(X& other)` in rare cases). It is invoked when an object is copy-initialized from another object of the same type: `X b = a;`, `X b(a);`, passing by value, returning by value (if elision doesn't apply), and throwing/catching exceptions. If not user-declared, the compiler generates one that member-wise copies (unless deleted by move operations or other rules).

**Beginner-Friendly Explanation**
A copy constructor is the **“duplicate me”** constructor. It creates a new object by copying the contents of an existing one. For simple classes, the compiler writes it for you. For classes that own resources, you often need to write it yourself (deep copy).

### Purposes

- To create an independent copy of an object.
- To support pass-by-value and return-by-value semantics.
- To implement deep copies for resource-owning classes.
- To support value semantics.

### Syntax Rules and Structure

**General Syntax**
```cpp
class X {
public:
    X(const X& other);   // copy constructor
};
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `const X&` | Parameter type (const lvalue reference). |
| Member-wise copy | Default behavior. |
| Deep copy | User-defined behavior for owned resources. |

**Syntax Rules**
- The parameter is typically `const X&`.
- The compiler-generated copy constructor performs member-wise copy.
- A user-declared copy constructor suppresses the implicit move operations.
- The copy constructor should handle self-copy correctly (usually not an issue).

**Constraints and Limitations**
- The compiler-generated copy constructor performs shallow copies (pointers are copied, not pointees).
- If the class owns resources, you need a user-defined copy constructor (Rule of Three/Five).
- Copy constructors are not called when copy elision applies.

### Annotated Code Examples

**Example 1: Copy constructor**

```cpp
#include <iostream>

class Point {
    int x_, y_;
public:
    Point(int x, int y) : x_(x), y_(y) {}
    Point(const Point& other) : x_(other.x_), y_(other.y_) {   // copy ctor
        std::cout << "copied\n";
    }
    int x() const { return x_; }
    int y() const { return y_; }
};

int main() {
    Point a(1, 2);
    Point b = a;   // copy constructor
    std::cout << b.x() << ',' << b.y() << '\n';
}
```

**Expected Output**
```
copied
1,2
```

**Why this output?**  
`Point b = a;` invokes the copy constructor, which copies `x_` and `y_` and prints "copied".

### Real-World Cases

- **Value semantics** – copying objects stored in containers.
- **Deep copies** – classes owning raw pointers.
- **Pass-by-value** – copying arguments (when elision doesn't apply).

---

## 5. Move Constructors

### Definitions

**Core Definition**
A move constructor is a constructor that **transfers resources** from an rvalue of the same type, leaving the source in a valid but unspecified state.

**Technical Definition**
A move constructor has the signature `X(X&& other)` (often `noexcept`). It is selected when the argument is an rvalue of type `X` (or convertible). It “steals” the source's resources (e.g., moves member pointers and nulls the source's pointers) rather than copying them. It is implicitly generated if the class has no user-declared copy operations, move operations, or destructor.

**Beginner-Friendly Explanation**
A move constructor is the **“steal from the temporary”** constructor. When the source is about to be destroyed anyway, the move constructor takes its resources instead of copying them. It's much faster for resource-owning classes.

### Purposes

- To efficiently construct objects from temporaries.
- To transfer ownership of resources.
- To support move-only types.
- To enable efficient container operations.

### Syntax Rules and Structure

**General Syntax**
```cpp
class X {
public:
    X(X&& other) noexcept;   // move constructor
};
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `X&&` | Rvalue reference parameter. |
| `noexcept` | Recommended for container optimizations. |
| Resource transfer | Move member pointers; null the source. |

**Syntax Rules**
- The parameter is a non-`const` rvalue reference.
- Mark `noexcept` when possible (C.66).
- Leave the source in a valid but unspecified state.
- If a move constructor is declared, the copy constructor is deleted unless explicitly declared.

**Constraints and Limitations**
- `const` rvalues do not bind to `X&&` (they bind to `const X&`).
- Moving from the source means it should not be used afterward (except to reassign/destroy).

### Annotated Code Examples

**Example 1: Move constructor**

```cpp
#include <iostream>
#include <utility>

class Buffer {
    int* data_;
public:
    explicit Buffer(int value) : data_(new int(value)) {}
    ~Buffer() { delete data_; }
    Buffer(Buffer&& other) noexcept : data_(other.data_) {
        other.data_ = nullptr;
        std::cout << "moved\n";
    }
    int value() const { return *data_; }
};

int main() {
    Buffer a(42);
    Buffer b(std::move(a));   // move constructor
    std::cout << b.value() << '\n';
}
```

**Expected Output**
```
moved
42
```

**Why this output?**  
`Buffer b(std::move(a))` selects the move constructor, which steals `a`'s pointer and sets it to `nullptr`.

### Real-World Cases

- **`unique_ptr`** – move-only ownership.
- **`std::vector` reallocation** – moves elements.
- **Returning large objects** – move construction avoids copies.

---

## 6. Delegating Constructors

### Definitions

**Core Definition**
A delegating constructor is a constructor that **calls another constructor of the same class** to perform the actual initialization, avoiding duplication of initialization logic.

**Technical Definition**
A delegating constructor (C++11) uses a member initializer list containing a single entry that is a call to another constructor of the same class: `X(args) : X(other_args) { ... }`. The target constructor (the *delegated-to* constructor) performs the full initialization; the delegating constructor's body then runs. This allows common initialization logic to be factored into a single constructor.

**Beginner-Friendly Explanation**
A delegating constructor is a constructor that **passes the buck** to another constructor of the same class. Instead of duplicating initialization code, it says: “Let that other constructor do the real work; I'll just add a little extra afterward.”

### Purposes

- To avoid duplication of initialization logic.
- To centralize the class invariant in one constructor.
- To simplify maintenance of multiple constructors.
- To make constructor relationships explicit.

### Syntax Rules and Structure

**General Syntax**
```cpp
class X {
public:
    X(int a, int b) : x_(a), y_(b) {}   // target constructor
    X(int a) : X(a, 0) {}                // delegating constructor
};
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `: X(args)` | Delegating initializer (must be the only entry). |
| Body | Runs after the delegated constructor completes. |

**Syntax Rules**
- A delegating constructor's initializer list must contain exactly one entry: the call to another constructor.
- The delegated-to constructor must be a constructor of the same class.
- The delegating constructor's body runs after the target constructor completes.
- Delegation cannot be circular (directly or indirectly).

**Constraints and Limitations**
- A constructor cannot delegate to itself (directly or indirectly).
- The target constructor must be accessible.
- Delegation does not allow partial initialization before the delegated call.

### Annotated Code Examples

**Example 1: Delegating constructor**

```cpp
#include <iostream>

class Point {
    int x_, y_;
public:
    Point(int x, int y) : x_(x), y_(y) {          // target
        std::cout << "Point(" << x_ << ", " << y_ << ")\n";
    }
    Point() : Point(0, 0) {}                       // delegating
    Point(int x) : Point(x, 0) {}                  // delegating
};

int main() {
    Point a;         // delegates to Point(0, 0)
    Point b(5);      // delegates to Point(5, 0)
    Point c(1, 2);   // direct
}
```

**Expected Output**
```
Point(0, 0)
Point(5, 0)
Point(1, 2)
```

**Why this output?**  
`Point()` delegates to `Point(0, 0)`, and `Point(int)` delegates to `Point(x, 0)`. The target constructor performs the actual initialization and prints the message.

### Real-World Cases

- **Multiple constructors with shared logic** – validation in one place.
- **Default arguments as delegation** – `X() : X(default_args) {}`.
- **Simplifying overload sets** – one target, several delegating constructors.

---

## 7. Inheriting Constructors

### Definitions

**Core Definition**
An inheriting constructor is a constructor that is **brought into a derived class** from its base class using `using Base::Base;`, allowing the derived class to be constructed with the base class's constructor signatures.

**Technical Definition**
Inheriting constructors (C++11) allow a derived class to inherit the constructors of its base class via a `using` declaration: `using Base::Base;`. The inherited constructors initialize the base subobject and then initialize the derived class's members using their default member initializers. Inherited constructors do not include default, copy, or move constructors of the base (those are handled separately).

**Beginner-Friendly Explanation**
Inheriting constructors let a derived class **reuse the base class's constructors** without rewriting them. If `Base` has `Base(int)`, then `using Base::Base;` in `Derived` gives `Derived` a `Derived(int)` that calls `Base(int)`.

### Purposes

- To avoid rewriting base-class constructors in derived classes.
- To simplify derived-class definitions.
- To make constructor inheritance explicit.
- To support generic and template-based designs.

### Syntax Rules and Structure

**General Syntax**
```cpp
class Derived : public Base {
public:
    using Base::Base;   // inherit base constructors
};
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `using Base::Base;` | Inherits all base constructors (except default/copy/move). |
| Derived members | Initialized with default member initializers. |

**Syntax Rules**
- `using Base::Base;` brings in base constructors.
- Inherited constructors initialize the base subobject, then derived members use default member initializers.
- Default, copy, and move constructors are not inherited (they are generated for the derived class).
- If a derived constructor has the same signature as an inherited one, the derived one hides the inherited one.

**Constraints and Limitations**
- Inherited constructors cannot be `explicit`-changed independently.
- The base constructor must be accessible.
- Members of the derived class without default initializers may be left uninitialized.

### Annotated Code Examples

**Example 1: Inheriting constructors**

```cpp
#include <iostream>

class Base {
public:
    Base(int x) { std::cout << "Base(" << x << ")\n"; }
};

class Derived : public Base {
    int extra_ = 100;
public:
    using Base::Base;   // inherit Base(int)
};

int main() {
    Derived d(42);   // calls Base(42), extra_ = 100
    std::cout << "done\n";
}
```

**Expected Output**
```
Base(42)
done
```

**Why this output?**  
`using Base::Base;` gives `Derived` a constructor `Derived(int)` that calls `Base(int)`. The derived member `extra_` uses its default initializer.

### Real-World Cases

- **Wrapper classes** – inheriting constructors from a base.
- **Template-based designs** – generic derived classes.
- **Reducing boilerplate** – avoiding duplicate constructors.

---

## 8. Member Initializer List

### Definitions

**Core Definition**
A member initializer list is the part of a constructor's definition that **initializes data members before the constructor body runs**.

**Technical Definition**
A member initializer list appears after the constructor's parameter list and before its body, introduced by a colon and containing comma-separated entries of the form `member(initializer)`. Members are initialized in **declaration order**, not initializer-list order. Members not listed are default-initialized (or left uninitialized for fundamental types). The initializer list is the only way to initialize `const` members, reference members, and members without default constructors.

**Beginner-Friendly Explanation**
The member initializer list is the **proper way to set up members** in a constructor. Instead of assigning in the body, you initialize members directly in the list: `X(int a) : member_(a) {}`. It's faster and required for `const` and reference members.

### Purposes

- To initialize members directly (rather than assigning in the body).
- To initialize `const`, reference, and non-default-constructible members.
- To improve performance (avoids default construction + assignment).
- To make initialization order explicit.

### Syntax Rules and Structure

**General Syntax**
```cpp
class X {
public:
    X(int a, double b) : a_(a), b_(b) {   // member initializer list
        // body
    }
};
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `:` | Introduces the initializer list. |
| `member(init)` | Initializes a member. |
| Body | Runs after all members are initialized. |

**Syntax Rules**
- Members are initialized in **declaration order**, not list order.
- Members not listed are default-initialized.
- `const` and reference members must be initialized in the list.
- The list can include base class constructors.

**Constraints and Limitations**
- Initializing in the body is less efficient and impossible for `const`/reference members.
- List order does not affect initialization order; mismatches can cause warnings.

### Annotated Code Examples

**Example 1: Member initializer list**

```cpp
#include <iostream>

class Point {
    int x_, y_;
public:
    Point(int x, int y) : x_(x), y_(y) {   // initializer list
        std::cout << "Point(" << x_ << ", " << y_ << ")\n";
    }
};

int main() {
    Point p(3, 4);
}
```

**Expected Output**
```
Point(3, 4)
```

**Why this output?**  
The initializer list sets `x_` and `y_` before the body runs, then the body prints them.

### Real-World Cases

- **Resource-owning classes** – initializing pointers, handles.
- **`const` and reference members** – must use the list.
- **Base class initialization** – `Derived() : Base(args) {}`.

---

## 9. Constructor Initialization Order

### Definitions

**Core Definition**
Constructor initialization order is the **fixed sequence** in which members and base classes are initialized: base classes first (in declaration order), then non-static data members (in declaration order), then the constructor body.

**Technical Definition**
The order of initialization is:
1. Base class subobjects (in declaration order of the base-specifier-list).
2. Non-static data members (in declaration order within the class).
3. The constructor body.

This order is **independent** of the member initializer list's order. If the initializer list order differs from the declaration order, the compiler may warn (e.g., GCC's `-Wreorder`).

**Beginner-Friendly Explanation**
Constructor initialization order is **fixed by declaration order**, not by how you write the initializer list. Base classes are set up first, then members in the order they're declared in the class, then the constructor body runs. This matters because members can depend on other members' values.

### Purposes

- To understand the actual order of initialization.
- To avoid bugs from order-dependent member initialization.
- To interpret compiler warnings about reorder.
- To design classes with safe initialization dependencies.

### Syntax Rules and Structure

**Initialization Order**
```
1. Base classes (declaration order)
2. Non-static data members (declaration order)
3. Constructor body
```

**Component Breakdown**

| Phase | Order |
|-------|-------|
| Base classes | In base-specifier-list order. |
| Data members | In class declaration order. |
| Body | Last. |

**Syntax Rules**
- The initializer list order does not affect initialization order.
- Members should be initialized in the same order they are declared (to avoid warnings).
- A member's initializer should not depend on another member that is declared later.

**Constraints and Limitations**
- Using a member before it is initialized is undefined behavior.
- The compiler may warn about reordered initializers (`-Wreorder`).

### Annotated Code Examples

**Example 1: Initialization order**

```cpp
#include <iostream>

class X {
    int a_;
    int b_;
public:
    X() : b_(2), a_(1) {   // list order: b, a; actual order: a, b
        std::cout << a_ << ' ' << b_ << '\n';
    }
};
// GCC may warn: 'b_' will be initialized after 'a_'

int main() {
    X x;
}
```

**Expected Output**
```
1 2
```

**Why this output?**  
`a_` is declared first, so it is initialized first (to `1`), then `b_` (to `2`), regardless of the initializer list order.

### Real-World Cases

- **Dependent members** – a member initialized from another member must be declared after it.
- **Compiler warnings** – `-Wreorder` catches mismatched order.
- **Debugging** – understanding why a member has an unexpected value.

---

## 10. Explicit Constructors / `explicit` Keyword

### Definitions

**Core Definition**
The `explicit` keyword prevents a constructor from being used for **implicit conversions** and copy-initialization, requiring direct initialization instead.

**Technical Definition**
A constructor declared `explicit` cannot be used for implicit conversions or copy-initialization. It can only be used for direct-initialization (`X x(args);`) or explicit casts (`static_cast<X>(args)`). Since C++11, `explicit` can also be applied to conversion functions. Since C++20, `explicit(bool)` allows conditional explicitness. The Core Guidelines recommend `explicit` for single-argument constructors by default (C.46).

**Beginner-Friendly Explanation**
`explicit` stops a constructor from **silently converting** one type into another. Without `explicit`, a single-argument constructor can be used implicitly (e.g., passing an `int` where an `X` is expected). With `explicit`, you must construct the object explicitly, making conversions visible and intentional.

### Purposes

- To prevent unintended implicit conversions.
- To make conversions explicit and readable.
- To avoid ambiguous overloads.
- To enforce type safety.

### Syntax Rules and Structure

**General Syntax**
```cpp
class X {
public:
    explicit X(int a);   // explicit constructor
};
```

**Component Breakdown**

| Form | Meaning |
|------|---------|
| `explicit X(args)` | Prevents implicit conversion. |
| `X x(args);` | Direct-initialization (allowed). |
| `X x = args;` | Copy-initialization (forbidden for explicit). |

**Syntax Rules**
- `explicit` applies to constructors and conversion functions.
- `explicit` constructors can be used in direct-initialization and explicit casts.
- `explicit` cannot be used in copy-initialization.
- `explicit(bool)` (C++20) allows conditional explicitness.

**Constraints and Limitations**
- `explicit` does not prevent direct-initialization.
- `explicit` on multi-argument constructors is allowed but usually unnecessary (since C++11, it affects list-initialization).

### Annotated Code Examples

**Example 1: `explicit` preventing implicit conversion**

```cpp
#include <iostream>

class Widget {
public:
    explicit Widget(int value) {   // explicit
        std::cout << "Widget(" << value << ")\n";
    }
};

void use(Widget w) {}

int main() {
    Widget a(42);        // OK: direct-initialization
    // Widget b = 42;    // error: copy-initialization with explicit
    // use(42);          // error: implicit conversion
    use(Widget(42));     // OK: explicit construction
}
```

**Expected Output**
```
Widget(42)
Widget(42)
```

**Why this output?**  
`Widget a(42)` and `use(Widget(42))` use explicit construction. The commented lines would be errors because `explicit` prevents implicit conversion.

### Real-World Cases

- **Single-argument constructors** – `explicit std::vector(size_t)`.
- **Smart pointers** – `explicit unique_ptr(pointer)`.
- **Type wrappers** – `explicit Meters(double)`.

---

## 11. Destructors

### Definitions

**Core Definition**
A destructor is a special member function that **cleans up an object** when its lifetime ends, releasing resources and performing cleanup.

**Technical Definition**
A destructor has the name `~ClassName`, no parameters, and no return type. It is invoked automatically when an object goes out of scope, is deleted, or is destroyed as part of another object. The destructor should release all resources the object owns and must not throw (implicitly `noexcept(true)`). It should be virtual in polymorphic base classes.

**Beginner-Friendly Explanation**
A destructor is the **cleanup function** that runs when an object dies. It closes files, frees memory, and releases resources. C++ calls it automatically, which is what makes RAII work.

### Purposes

- To release resources (memory, files, sockets, locks).
- To perform cleanup (flush buffers, close connections).
- To restore external state.
- To support RAII and exception safety.

### Syntax Rules and Structure

**General Syntax**
```cpp
class X {
public:
    ~X();                // destructor declaration
};
X::~X() { /* ... */ }    // definition
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `~X()` | Destructor name. |
| No parameters | Destructors take no arguments. |
| No return type | Destructors return nothing. |

**Syntax Rules**
- A class has exactly one destructor.
- Destructors are called automatically in reverse order of construction.
- Destructors should not throw.
- Base class destructors should be public and virtual, or protected and non-virtual (C.35).
- Destructors can be `= default` or `= delete`.

**Constraints and Limitations**
- Throwing from a destructor during stack unwinding calls `std::terminate`.
- A non-virtual destructor in a polymorphic base class can cause undefined behavior on `delete`.
- Static objects are destroyed at program exit; order across translation units is unspecified.

### Annotated Code Examples

**Example 1: Destructor cleanup**

```cpp
#include <iostream>

class Tracer {
public:
    Tracer()  { std::cout << "acquired\n"; }
    ~Tracer() { std::cout << "released\n"; }
};

int main() {
    Tracer t;
    std::cout << "inside main\n";
}
```

**Expected Output**
```
acquired
inside main
released
```

**Why this output?**  
The destructor runs when `t` goes out of scope at the end of `main`.

### Real-World Cases

- **RAII wrappers** – `unique_ptr`, `lock_guard`, `fstream`.
- **Resource handles** – sockets, database connections.
- **Containers** – destroy elements and free storage.

---

## 12. Virtual Destructors

### Definitions

**Core Definition**
A virtual destructor is a destructor declared `virtual` in a base class, ensuring that deleting a derived object through a base-class pointer calls the **correct derived destructor**.

**Technical Definition**
A destructor declared `virtual` in a base class is dynamically dispatched: when `delete` is called on a base-class pointer to a derived object, the derived destructor runs first, then the base destructor. Without a virtual destructor, deleting through a base pointer is undefined behavior (the derived destructor is not called). The Core Guidelines state: a base class destructor should be either public and virtual, or protected and non-virtual (C.35).

**Beginner-Friendly Explanation**
A virtual destructor ensures that when you delete an object through a base-class pointer, the **right destructor runs**. Without it, only the base part is destroyed, and the derived part leaks. If a class is meant to be used polymorphically, make its destructor virtual.

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
| `~Base() = default` | Defaulted virtual destructor. |
| `protected: ~Base()` | Non-virtual protected destructor. |

**Syntax Rules**
- A base class with virtual functions should have a virtual destructor if deletion through base pointers is possible.
- Alternatively, make the destructor protected and non-virtual to prevent deletion through base pointers.
- Virtual destructors incur a vtable entry cost.

**Constraints and Limitations**
- Virtual destructors do not prevent slicing (copying a derived object into a base object).
- A class with a virtual destructor is polymorphic and has a vtable.

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

## 13. Pure Virtual Destructors

### Definitions

**Core Definition**
A pure virtual destructor is a destructor declared with `= 0` that makes the class **abstract**, while still requiring a definition (since base-class destructors are always called).

**Technical Definition**
A pure virtual destructor is declared as `virtual ~X() = 0;`. It makes `X` an abstract class (cannot be instantiated) but **must still be defined** because derived-class destructors call it. Pure virtual destructors are used to make a class abstract when it has no other pure virtual functions, often in combination with other pure virtual functions for interface classes.

**Beginner-Friendly Explanation**
A pure virtual destructor is a **destructor that makes the class abstract** — you can't create objects of the class directly, but you can derive from it. Unlike other pure virtual functions, it still needs a definition, because it is always called when derived objects are destroyed.

### Purposes

- To make a class abstract without adding another pure virtual function.
- To provide a base class that can only be used polymorphically.
- To enforce a polymorphic contract.
- To support interface-style class hierarchies.

### Syntax Rules and Structure

**General Syntax**
```cpp
class Base {
public:
    virtual ~Base() = 0;   // pure virtual destructor declaration
};
Base::~Base() {}           // definition required
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `virtual ~Base() = 0;` | Pure virtual destructor declaration. |
| `Base::~Base() {}` | Definition (required). |

**Syntax Rules**
- A pure virtual destructor makes the class abstract.
- The pure virtual destructor must be defined (outside the class).
- Derived classes must override the destructor (or inherit the pure virtual status).
- The definition is called when derived objects are destroyed.

**Constraints and Limitations**
- Forgetting to define the pure virtual destructor causes a linker error.
- The pure virtual destructor cannot be `= delete`.
- It does not prevent the class from having other pure virtual functions.

### Annotated Code Examples

**Example 1: Pure virtual destructor**

```cpp
#include <iostream>

class Interface {
public:
    virtual ~Interface() = 0;   // pure virtual
    virtual void f() = 0;
};
Interface::~Interface() { std::cout << "Interface destroyed\n"; }   // required

class Impl : public Interface {
public:
    ~Impl() override { std::cout << "Impl destroyed\n"; }
    void f() override { std::cout << "f()\n"; }
};

int main() {
    Interface* p = new Impl;
    p->f();
    delete p;
}
```

**Expected Output**
```
f()
Impl destroyed
Interface destroyed
```

**Why this output?**  
`Interface` is abstract due to the pure virtual destructor. `Impl` overrides it. Deleting through `Interface*` calls `Impl::~Impl` then `Interface::~Interface`.

### Real-World Cases

- **Interface classes** – abstract bases with no other pure virtuals.
- **Polymorphic hierarchies** – enforcing abstractness.
- **API design** – preventing direct instantiation.

---

## 14. Constructor/Destructor Best Practices

### Definitions

**Core Definition**
Constructor/destructor best practices are guidelines for writing constructors and destructors that are correct, safe, efficient, and maintainable.

**Technical Definition**
The C++ Core Guidelines provide specific rules: **C.40** (define a constructor if a class has an invariant), **C.41** (a constructor should create a fully initialized object), **C.42** (if a constructor cannot construct a valid object, throw an exception), **C.44** (prefer default constructors to be simple and non-throwing), **C.45** (don't define a default constructor that only initializes data members; use default member initializers), **C.46** (declare single-argument constructors `explicit` by default), **C.47** (define and initialize member variables in the order of declaration), **C.48** (prefer in-class initializers to member initializers in constructors for constant initializers), **C.30–C.35** (destructor rules), and **C.35** (base class destructor: public virtual or protected non-virtual).

**Beginner-Friendly Explanation**
Best practices for constructors and destructors are simple: **make constructors fully initialize objects**, **throw on failure**, **use member initializer lists**, **declare single-argument constructors `explicit`**, **initialize members in declaration order**, **make destructors not throw**, and **make base destructors virtual when needed**.

### Key Guidelines (Summary Table)

| Guideline | Summary | Recommendation |
|-----------|---------|----------------|
| **C.40** | Define a constructor if a class has an invariant. | Enforce invariants at construction. |
| **C.41** | A constructor should create a fully initialized object. | No half-constructed objects. |
| **C.42** | If a constructor cannot construct a valid object, throw an exception. | Fail loudly. |
| **C.44** | Prefer default constructors to be simple and non-throwing. | Default constructors should be trivial. |
| **C.45** | Don't define a default constructor that only initializes data members. | Use default member initializers. |
| **C.46** | Declare single-argument constructors `explicit` by default. | Prevent implicit conversions. |
| **C.47** | Define and initialize member variables in declaration order. | Avoid reorder bugs. |
| **C.48** | Prefer in-class initializers for constant initializers. | Simpler and clearer. |
| **C.35** | Base destructor: public virtual or protected non-virtual. | Correct polymorphic deletion. |
| **No throwing destructors** | Destructors must not throw. | Use `noexcept` (default). |
| **Rule of Zero/Five** | Follow the appropriate special-member rule. | Prefer the Rule of Zero. |

### Annotated Code Examples

**Example 1: Fully initialized object (C.41)**

```cpp
#include <iostream>
#include <stdexcept>

class Person {
    std::string name_;
    int age_;
public:
    Person(std::string name, int age)
        : name_(std::move(name)), age_(age) {
        if (age_ < 0) throw std::invalid_argument("age must be non-negative");
    }
    const std::string& name() const { return name_; }
    int age() const { return age_; }
};

int main() {
    try {
        Person p("Alice", 30);
        std::cout << p.name() << '\n';
        Person q("Bob", -1);   // throws
    } catch (const std::exception& e) {
        std::cout << "Error: " << e.what() << '\n';
    }
}
```

**Expected Output**
```
Alice
Error: age must be non-negative
```

**Why this is best practice?**  
The constructor fully initializes the object and throws on invalid input (C.41, C.42). The object is never in a half-constructed state.

**Example 2: `explicit` constructor (C.46)**

```cpp
#include <iostream>

class Widget {
public:
    explicit Widget(int value) { std::cout << value << '\n'; }
};

void use(Widget w) {}

int main() {
    use(Widget(42));   // OK: explicit construction
    // use(42);        // error: explicit prevents implicit conversion
}
```

**Expected Output**
```
42
```

**Why this is best practice?**  
`explicit` prevents unintended implicit conversions, making the code safer and clearer (C.46).

**Example 3: Virtual destructor (C.35)**

```cpp
#include <iostream>

struct Base {
    virtual ~Base() = default;   // public virtual destructor
};

struct Derived : Base {
    ~Derived() override { std::cout << "Derived destroyed\n"; }
};

int main() {
    Base* p = new Derived;
    delete p;   // correct destruction
}
```

**Expected Output**
```
Derived destroyed
```

**Why this is best practice?**  
The virtual destructor ensures correct polymorphic deletion (C.35).

### Real-World Cases

- **Standard library** – `std::vector`, `std::string`, `std::unique_ptr` follow these rules.
- **Code review** – Core Guidelines C.40–C.48, C.35 are common checkpoints.
- **Static analysis** – clang-tidy enforces many constructor/destructor rules.

---

## References

- Constructors (cppreference) – https://en.cppreference.com/w/cpp/language/constructor
- Default constructor (cppreference) – https://en.cppreference.com/w/cpp/language/default_constructor
- Copy constructor (cppreference) – https://en.cppreference.com/w/cpp/language/copy_constructor
- Move constructor (cppreference) – https://en.cppreference.com/w/cpp/language/move_constructor
- Delegating constructor (cppreference) – https://en.cppreference.com/w/cpp/language/constructor#Delegating_constructor
- Inheriting constructor (cppreference) – https://en.cppreference.com/w/cpp/language/using_declaration#Inheriting_constructors
- Member initializer list (cppreference) – https://en.cppreference.com/w/cpp/language/constructor#Member_initializer_list
- `explicit` specifier (cppreference) – https://en.cppreference.com/w/cpp/language/explicit
- Destructor (cppreference) – https://en.cppreference.com/w/cpp/language/destructor
- Virtual destructor (cppreference) – https://en.cppreference.com/w/cpp/language/destructor#Virtual_destructors
- Pure virtual destructor (cppreference) – https://en.cppreference.com/w/cpp/language/abstract_class
- C++ Core Guidelines — C.35: A base class destructor should be either public and virtual, or protected and non-virtual – https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rc-dtor-virtual
- C++ Core Guidelines — C.40: Define a constructor if a class has an invariant – https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rc-ctor
- C++ Core Guidelines — C.41: A constructor should create a fully initialized object – https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rc-complete
- C++ Core Guidelines — C.42: If a constructor cannot construct a valid object, throw an exception – https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rc-throw
- C++ Core Guidelines — C.44: Prefer default constructors to be simple and non-throwing – https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rc-default
- C++ Core Guidelines — C.45: Don't define a default constructor that only initializes data members – https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rc-default0
- C++ Core Guidelines — C.46: By default, declare single-argument constructors `explicit` – https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rc-explicit
- C++ Core Guidelines — C.47: Define and initialize member variables in the order of declaration – https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rc-order
- C++ Core Guidelines — C.48: Prefer in-class initializers to member initializers in constructors for constant initializers – https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rc-in-class-initializer
- Microsoft Learn — Constructors (C++) – https://learn.microsoft.com/en-us/cpp/cpp/constructors-cpp
- Microsoft Learn — Destructors (C++) – https://learn.microsoft.com/en-us/cpp/cpp/destructors-cpp
- Microsoft Learn — Delegating constructors – https://learn.microsoft.com/en-us/cpp/cpp/delegating-constructors
- Microsoft Learn — Inheriting constructors – https://learn.microsoft.com/en-us/cpp/cpp/inheriting-constructors
- Microsoft Learn — Explicit constructors – https://learn.microsoft.com/en-us/cpp/cpp/explicit-constructors

---

This cheat sheet provides a complete, structured reference for C++ constructors and destructors, covering default, parameterized, copy, move, delegating, and inheriting constructors; member initializer lists and initialization order; `explicit`; destructors, virtual destructors, and pure virtual destructors; and best practices. Each section follows a uniform template with definitions, purposes, syntax rules, annotated examples, expected outputs, and authoritative references.