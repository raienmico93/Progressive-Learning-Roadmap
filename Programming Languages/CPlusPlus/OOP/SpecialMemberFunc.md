# Special Member Functions — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
Special member functions are member functions that the C++ compiler may **implicitly declare and define** for a class: the default constructor, destructor, copy constructor, copy assignment operator, move constructor, and move assignment operator (plus, in C++20, comparison operators are related but not part of the "special" six).

**Technical Definition**
The C++ standard defines **special member functions** as the default constructor, copy/move constructors, copy/move assignment operators, and destructor. The compiler implicitly declares and defines these functions when they are odr-used, subject to complex rules governing when they are **defaulted**, **deleted**, or **suppressed** by user declarations. Since C++11, the six special members are: default constructor, destructor, copy constructor, copy assignment, move constructor, and move assignment. The interaction of these functions is governed by the **Rule of Three**, **Rule of Five**, and **Rule of Zero**.

**Beginner-Friendly Explanation**
Special member functions are the **automatic housekeeping functions** every class gets: how to construct (default constructor), how to destroy (destructor), how to copy (copy constructor, copy assignment), and how to move (move constructor, move assignment). If you don't write them, the compiler writes them for you — but the compiler's versions may be wrong if your class owns resources. The Rule of Three/Five/Zero tells you when to write your own and when to let the compiler do it.

### Key Characteristics

- **Six special members** (C++11): default constructor, destructor, copy constructor, copy assignment, move constructor, move assignment.
- **Implicitly generated**: The compiler declares and defines them when needed, unless suppressed.
- **Suppression rules**: User-declaring one special member can suppress or delete others.
- **`= default` / `= delete`**: Explicitly request or forbid the compiler-generated version.
- **Rule of Three**: If you need a destructor, copy constructor, or copy assignment, you likely need all three.
- **Rule of Five**: Extends the Rule of Three to move operations.
- **Rule of Zero**: Prefer composing RAII members and declaring no special members.
- **Exception specifications**: Destructors are implicitly `noexcept`; move operations should be `noexcept`.

### Prerequisites

- Understanding of classes, objects, and constructors.
- Familiarity with copy and move semantics.
- Knowledge of resource management (RAII) and smart pointers.
- Basic awareness of value categories and `std::move`.

### Related Programming Areas

- **RAII** – special members manage resource lifetimes.
- **Move semantics** – move constructor and move assignment.
- **Rule of Three/Five/Zero** – governing special member declarations.
- **Smart pointers** – RAII wrappers that follow the Rule of Zero.
- **Exception safety** – constructors and destructors in exception contexts.
- **Value semantics** – copy/move operations for value types.

### Core Concepts / Features

1. Default Constructor
2. Destructor
3. Copy Constructor
4. Copy Assignment Operator
5. Move Constructor
6. Move Assignment Operator
7. Rule of Three
8. Rule of Five
9. Rule of Zero
10. Special Member Function Best Practices

---

## 1. Default Constructor

### Definitions

**Core Definition**
A default constructor is a constructor that can be called with **no arguments**, initializing an object with default values.

**Technical Definition**
A default constructor is a constructor that can be invoked with no arguments: `X()`, `X(int a = 0)`, or `X(std::initializer_list<int> = {})`. The compiler implicitly declares a default constructor if no user-declared constructors exist. It is **defaulted** (defined as `= default`) if possible; it is **deleted** if a member has no default constructor (e.g., a reference member, or a member of a class with a deleted default constructor). It is **suppressed** by any user-declared constructor.

**Beginner-Friendly Explanation**
The default constructor is the **“no-arguments” constructor** that creates an object with default values. If you don't write any constructor, the compiler provides one (unless other rules apply).

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
    X() = delete;        // explicitly deleted
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
- A defaulted default constructor is `constexpr` if possible.
- Default member initializers (C++11) are used if the constructor doesn't initialize a member.

**Constraints and Limitations**
- The default constructor is deleted if a member has no default constructor (e.g., a reference).
- User-declared constructors suppress the implicit default constructor.
- The default constructor is suppressed by user-declared copy/move operations or destructor (in some cases).

### Annotated Code Examples

**Example 1: Defaulted default constructor**

```cpp
#include <iostream>

class Point {
    int x_ = 0, y_ = 0;   // default member initializers
public:
    Point() = default;    // defaulted default constructor
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

## 2. Destructor

### Definitions

**Core Definition**
A destructor is a special member function that **cleans up an object** when its lifetime ends, releasing resources and performing cleanup.

**Technical Definition**
A destructor has the name `~ClassName`, no parameters, and no return type. It is invoked automatically when an object goes out of scope, is deleted, or is destroyed as part of another object. The destructor should release all resources the object owns and must not throw (implicitly `noexcept(true)`). It should be virtual in polymorphic base classes. The compiler implicitly declares a destructor if none is declared; it is defaulted and `noexcept` unless a member's destructor can throw.

**Beginner-Friendly Explanation**
The destructor is the **cleanup function** that runs when an object dies. It closes files, frees memory, and releases resources. C++ calls it automatically, which is what makes RAII work.

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
    ~X() = default;      // defaulted
    virtual ~X() = default; // virtual destructor
};
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

## 3. Copy Constructor

### Definitions

**Core Definition**
A copy constructor is a constructor that **initializes a new object as a copy** of an existing object of the same type.

**Technical Definition**
A copy constructor has the signature `X(const X& other)` (or `X(X& other)` in rare cases). It is invoked when an object is copy-initialized from another object of the same type: `X b = a;`, `X b(a);`, passing by value, returning by value (if elision doesn't apply), and throwing/catching exceptions. If not user-declared, the compiler generates one that member-wise copies (unless deleted by move operations or other rules).

**Beginner-Friendly Explanation**
The copy constructor is the **“duplicate me”** constructor. It creates a new object by copying the contents of an existing one. For simple classes, the compiler writes it for you. For classes that own resources, you often need to write it yourself (deep copy).

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
    X(const X& other);        // copy constructor
    X(const X& other) = default;  // defaulted
    X(const X& other) = delete;   // deleted
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

## 4. Copy Assignment Operator

### Definitions

**Core Definition**
The copy assignment operator **replaces the contents** of an existing object with a copy of another object of the same type.

**Technical Definition**
The copy assignment operator has the signature `X& operator=(const X& other)`. It is invoked when assigning to an existing object from an lvalue of the same type: `a = b;`. The compiler generates one that member-wise assigns (unless deleted). A user-defined copy assignment must release the target's old resources, copy the source's, handle self-assignment, and return `*this`.

**Beginner-Friendly Explanation**
Copy assignment is **“replace my contents with a copy of yours.”** It's different from the copy constructor, which creates a new object. Copy assignment modifies an existing object.

### Purposes

- To support assignment between existing objects.
- To implement deep copies for resource-owning classes.
- To handle self-assignment correctly.
- To return `*this` for chaining.

### Syntax Rules and Structure

**General Syntax**
```cpp
class X {
public:
    X& operator=(const X& other);        // copy assignment
    X& operator=(const X& other) = default;  // defaulted
    X& operator=(const X& other) = delete;   // deleted
};
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `X&` | Return type (allows chaining). |
| `const X&` | Parameter type. |
| Self-assignment check | `if (this != &other)`. |
| Release + copy | Release old, copy new. |

**Syntax Rules**
- Release the target's old resources before copying.
- Handle self-assignment (either with a check or copy-and-swap).
- Return `*this`.
- A user-declared copy assignment suppresses implicit move operations.

**Constraints and Limitations**
- Self-assignment without a check can cause leaks or undefined behavior.
- Copy assignment cannot be `virtual` (C.60).
- Copy-and-swap provides the strong exception guarantee.

### Annotated Code Examples

**Example 1: Copy assignment**

```cpp
#include <iostream>

class Buffer {
    int* data_;
public:
    explicit Buffer(int v) : data_(new int(v)) {}
    ~Buffer() { delete data_; }

    Buffer(const Buffer& other) : data_(new int(*other.data_)) {}

    Buffer& operator=(const Buffer& other) {
        if (this != &other) {          // self-assignment check
            delete data_;              // release old
            data_ = new int(*other.data_);  // deep copy
        }
        std::cout << "copy assigned\n";
        return *this;
    }
    int value() const { return *data_; }
};

int main() {
    Buffer a(10), b(20);
    b = a;
    std::cout << b.value() << '\n';
}
```

**Expected Output**
```
copy assigned
10
```

**Why this output?**  
`b = a` invokes the copy assignment operator, which releases `b`'s old buffer, deep-copies `a`'s buffer, and prints "copy assigned".

### Real-World Cases

- **Container element assignment** – `v[i] = obj;`.
- **Resource-owning classes** – releasing old, acquiring new.
- **Copy-and-swap** – strong exception guarantee.

---

## 5. Move Constructor

### Definitions

**Core Definition**
A move constructor is a constructor that **transfers resources** from an rvalue of the same type, leaving the source in a valid but unspecified state.

**Technical Definition**
A move constructor has the signature `X(X&& other)` (often `noexcept`). It is selected when the argument is an rvalue of type `X` (or convertible). It “steals” the source's resources (e.g., moves member pointers and nulls the source's pointers) rather than copying them. It is implicitly generated if the class has no user-declared copy operations, move operations, or destructor.

**Beginner-Friendly Explanation**
The move constructor is the **“steal from the temporary”** constructor. When the source is about to be destroyed anyway, the move constructor takes its resources instead of copying them. It's much faster for resource-owning classes.

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
    X(X&& other) noexcept;              // move constructor
    X(X&& other) noexcept = default;    // defaulted
    X(X&& other) = delete;              // deleted
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

## 6. Move Assignment Operator

### Definitions

**Core Definition**
The move assignment operator **transfers resources** from an rvalue of the same type into an existing object, releasing the target's old resources.

**Technical Definition**
The move assignment operator has the signature `X& operator=(X&& other)` (often `noexcept`). It is invoked when assigning from an rvalue: `a = std::move(b);` or `a = temporary;`. It releases the target's current resources, steals the source's resources, leaves the source in a valid but unspecified state, and returns `*this`.

**Beginner-Friendly Explanation**
Move assignment is **“empty my container and take yours.”** It releases what the target had and takes the source's resources. The source is left empty.

### Purposes

- To efficiently reassign from a temporary.
- To transfer ownership without copying.
- To support move-only types.
- To maintain exception safety.

### Syntax Rules and Structure

**General Syntax**
```cpp
class X {
public:
    X& operator=(X&& other) noexcept;              // move assignment
    X& operator=(X&& other) noexcept = default;    // defaulted
    X& operator=(X&& other) = delete;              // deleted
};
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `X&` | Return type. |
| `X&&` | Rvalue reference parameter. |
| `noexcept` | Recommended. |
| Release + steal | Free target, take source's. |

**Syntax Rules**
- Release the target's old resources.
- Handle self-assignment (or rely on the source being a temporary).
- Leave the source valid but unspecified.
- Return `*this`.
- Mark `noexcept` when possible.

**Constraints and Limitations**
- Self-move-assignment must be safe or documented as unsupported.
- A throwing move assignment can violate container guarantees.

### Annotated Code Examples

**Example 1: Move assignment**

```cpp
#include <iostream>
#include <utility>

class Buffer {
    int* data_;
public:
    explicit Buffer(int v) : data_(new int(v)) {}
    ~Buffer() { delete data_; }

    Buffer(Buffer&& other) noexcept : data_(other.data_) {
        other.data_ = nullptr;
    }

    Buffer& operator=(Buffer&& other) noexcept {
        if (this != &other) {
            delete data_;
            data_ = other.data_;
            other.data_ = nullptr;
        }
        std::cout << "move assigned\n";
        return *this;
    }
    int value() const { return *data_; }
};

int main() {
    Buffer a(10), b(20);
    b = std::move(a);
    std::cout << b.value() << '\n';
}
```

**Expected Output**
```
move assigned
10
```

**Why this output?**  
`b = std::move(a)` selects the move assignment operator, which releases `b`'s buffer, steals `a`'s buffer, and leaves `a` empty.

### Real-World Cases

- **Container element assignment** – `v[i] = std::move(obj);`.
- **Swap** – moving into temporaries.
- **Resource handles** – reassigning ownership.

---

## 7. Rule of Three

### Definitions

**Core Definition**
The Rule of Three states that if a class needs a **user-defined destructor**, **copy constructor**, or **copy assignment operator**, it likely needs **all three**.

**Technical Definition**
The Rule of Three (C++98) is a rule of thumb: if a class manages a resource directly (requiring a destructor), it also needs a copy constructor and copy assignment operator to manage that resource correctly during copying. Before C++11, these three special members were the only ones relevant to resource management. With C++11, the Rule of Five extends this to include move operations.

**Beginner-Friendly Explanation**
If you're managing a resource yourself (like a raw pointer), you need to handle **three things**: cleanup (destructor), copying (copy constructor), and copy assignment. If you do one, you probably need all three — otherwise, copying will do the wrong thing (e.g., double-delete).

### Purposes

- To remind developers to define all resource-management special members together.
- To prevent double deletion, leaks, and shallow-copy bugs.
- To provide a baseline for the Rule of Five and Rule of Zero.

### Syntax Rules and Structure

**The Three Special Members**
```cpp
class T {
public:
    ~T();                            // destructor
    T(const T& other);               // copy constructor
    T& operator=(const T& other);    // copy assignment
};
```

**Component Breakdown**

| Member | Purpose |
|--------|---------|
| Destructor | Releases resources. |
| Copy constructor | Deep-copies resources. |
| Copy assignment | Releases old, deep-copies new. |

**Syntax Rules**
- If you define any of the three, define all three (or use the Rule of Zero).
- Copy operations must perform deep copies for owning resources.
- Copy assignment must handle self-assignment.

**Constraints and Limitations**
- The Rule of Three does not cover move semantics; use the Rule of Five for that.
- Modern C++ prefers the Rule of Zero (compose RAII members) over manual resource management.

### Annotated Code Examples

**Example 1: Rule of Three**

```cpp
#include <iostream>
#include <cstring>

class String {
    char* data_;
public:
    explicit String(const char* s) : data_(new char[std::strlen(s) + 1]) {
        std::strcpy(data_, s);
    }
    ~String() { delete[] data_; }                       // destructor
    String(const String& other)                         // copy ctor
        : data_(new char[std::strlen(other.data_) + 1]) {
        std::strcpy(data_, other.data_);
    }
    String& operator=(const String& other) {            // copy assignment
        if (this != &other) {
            delete[] data_;
            data_ = new char[std::strlen(other.data_) + 1];
            std::strcpy(data_, other.data_);
        }
        return *this;
    }
    const char* c_str() const { return data_; }
};

int main() {
    String a("hello");
    String b = a;
    a = b;
    std::cout << b.c_str() << '\n';
}
```

**Expected Output**
```
hello
```

**Why this works?**  
All three special members are defined, ensuring deep copies and correct cleanup.

### Real-World Cases

- **Legacy resource-managing classes** – pre-C++11 code.
- **Understanding Rule of Five** – the Rule of Three is the foundation.
- **Code review** – spotting classes with a destructor but no copy operations.

---

## 8. Rule of Five

### Definitions

**Core Definition**
The Rule of Five states that if a class defines any of the five special members (**destructor, copy constructor, copy assignment, move constructor, move assignment**), it should consider defining all five for correctness and efficiency.

**Technical Definition**
The Rule of Five (C++11) extends the Rule of Three with move constructor and move assignment. If a class directly manages a resource (e.g., a raw owning pointer), it needs all five special members to correctly handle copying, moving, and destruction. If move operations are not defined, copying is used for rvalues, which may be inefficient. If copy operations are not defined, they may be implicitly deleted when move operations are user-declared.

**Beginner-Friendly Explanation**
If you manage a resource yourself, you now have **five things** to consider: cleanup, copying (two forms), and moving (two forms). Define all five, or none (Rule of Zero). If you define only some, the compiler may delete the others, leading to surprising behavior.

### Purposes

- To ensure correct resource management for copy and move.
- To prevent implicit deletion of copy operations when moves are defined.
- To provide efficient move semantics alongside copy semantics.
- To guide the design of resource-owning classes.

### Syntax Rules and Structure

**The Five Special Members**
```cpp
class T {
public:
    ~T();                             // destructor
    T(const T& other);                // copy constructor
    T& operator=(const T& other);     // copy assignment
    T(T&& other) noexcept;            // move constructor
    T& operator=(T&& other) noexcept; // move assignment
};
```

**Component Breakdown**

| Member | Purpose |
|--------|---------|
| Destructor | Releases resources. |
| Copy ctor | Deep-copies from lvalue. |
| Copy assignment | Deep-copies from lvalue. |
| Move ctor | Steals from rvalue. |
| Move assignment | Steals from rvalue. |

**Syntax Rules**
- Define all five, or none (Rule of Zero).
- Move operations should be `noexcept`.
- Copy operations should handle self-assignment.
- If move operations are defined, copy operations must be explicitly defined or defaulted (otherwise they are deleted).

**Constraints and Limitations**
- Defining all five is error-prone; prefer the Rule of Zero.
- The compiler's implicit generation rules are complex; consult the standard.

### Annotated Code Examples

**Example 1: Rule of Five**

```cpp
#include <iostream>
#include <utility>

class Buffer {
    int* data_;
    std::size_t size_;
public:
    explicit Buffer(std::size_t n) : data_(new int[n]), size_(n) {}
    ~Buffer() { delete[] data_; }                              // 1. destructor

    Buffer(const Buffer& other)                                // 2. copy ctor
        : data_(new int[other.size_]), size_(other.size_) {
        std::copy(other.data_, other.data_ + size_, data_);
    }

    Buffer& operator=(const Buffer& other) {                   // 3. copy assignment
        if (this != &other) {
            delete[] data_;
            data_ = new int[other.size_];
            size_ = other.size_;
            std::copy(other.data_, other.data_ + size_, data_);
        }
        return *this;
    }

    Buffer(Buffer&& other) noexcept                            // 4. move ctor
        : data_(other.data_), size_(other.size_) {
        other.data_ = nullptr;
        other.size_ = 0;
    }

    Buffer& operator=(Buffer&& other) noexcept {               // 5. move assignment
        if (this != &other) {
            delete[] data_;
            data_ = other.data_;
            size_ = other.size_;
            other.data_ = nullptr;
            other.size_ = 0;
        }
        return *this;
    }
};

int main() {
    Buffer a(10);
    Buffer b = a;              // copy
    Buffer c = std::move(a);   // move
    std::cout << "done\n";
}
```

**Expected Output**
```
done
```

**Why this works?**  
All five special members are defined, so the class correctly handles copying, moving, and destruction.

### Real-World Cases

- **Resource-owning classes** – classes that directly manage raw resources.
- **Library design** – ensuring correct behavior for all value categories.
- **Code review** – checking for all five when a destructor is present.

---

## 9. Rule of Zero

### Definitions

**Core Definition**
The Rule of Zero states that a class should **not define any special members** (destructor, copy/move constructors, copy/move assignment) unless it directly manages a resource; instead, it should compose RAII members that handle resource management.

**Technical Definition**
The Rule of Zero (popularized by R. Martinho Fernandes and the C++ Core Guidelines) recommends that classes avoid declaring special member functions and instead rely on the compiler-generated ones, which correctly compose the special members of their members and bases. By using RAII types (e.g., `std::string`, `std::vector`, `std::unique_ptr`), a class gets correct copy, move, and destruction behavior for free. The Core Guidelines express this as **C.20** (“If you can avoid defining default operations, do”) and **C.21** (“If you define or `=delete` any copy, move, or destructor function, define or `=delete` them all”).

**Beginner-Friendly Explanation**
The Rule of Zero says: **don't write destructors or copy/move operations yourself**. Instead, use members that already handle resources correctly (like `std::string` and `std::unique_ptr`). The compiler will generate the right operations for you, and your class will be simpler and safer.

### Purposes

- To simplify classes by avoiding manual resource management.
- To reduce bugs from incorrect copy/move/destructor implementations.
- To leverage the correctness of standard RAII types.
- To make classes automatically correct for copy, move, and destruction.
- To follow the Core Guidelines' preference for composition.

### Syntax Rules and Structure

**Rule of Zero Pattern**
```cpp
class Widget {
    std::string name_;              // RAII member
    std::unique_ptr<Impl> impl_;    // RAII member
public:
    Widget(std::string name, std::unique_ptr<Impl> impl)
        : name_(std::move(name)), impl_(std::move(impl)) {}
    // No destructor, copy, or move declarations needed
};
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| RAII members | `std::string`, `std::vector`, `std::unique_ptr`, etc. |
| No special members | Compiler generates them correctly. |
| Constructor | Initializes members with `std::move` for efficiency. |

**Syntax Rules**
- Use RAII members for all resources.
- Do not declare destructor, copy, or move operations unless necessary.
- Initialize members with `std::move` when taking ownership.
- If a member is move-only (e.g., `unique_ptr`), the class becomes move-only automatically.

**Constraints and Limitations**
- The Rule of Zero applies when resource management can be delegated to members.
- If a class directly manages a raw resource (e.g., a C API handle), it may need the Rule of Five.
- Some legacy or performance-critical code may need custom resource management.

### Annotated Code Examples

**Example 1: Rule of Zero**

```cpp
#include <iostream>
#include <memory>
#include <string>

class Widget {
    std::string name_;
    std::unique_ptr<int> data_;
public:
    Widget(std::string name, int value)
        : name_(std::move(name)), data_(std::make_unique<int>(value)) {}
    // No special members needed
    const std::string& name() const { return name_; }
    int value() const { return *data_; }
};

int main() {
    Widget w("gadget", 42);
    // Widget copy = w;         // error: unique_ptr is move-only
    Widget moved = std::move(w); // OK: move
    std::cout << moved.name() << ": " << moved.value() << '\n';
}
```

**Expected Output**
```
gadget: 42
```

**Why this works?**  
`Widget` composes `std::string` and `std::unique_ptr`, both RAII types. The compiler generates the destructor, copy operations (deleted because of `unique_ptr`), and move operations correctly. No manual special members are needed.

### Real-World Cases

- **Modern C++ classes** – most classes should follow the Rule of Zero.
- **Standard library** – `std::vector`, `std::string`, `std::map` all manage resources internally.
- **Code review** – preferring composition over manual resource management.
- **Core Guidelines** – C.20 and C.21 recommend the Rule of Zero.

---

## 10. Special Member Function Best Practices

### Definitions

**Core Definition**
Special member function best practices are guidelines for declaring, defining, and using the six special member functions correctly and efficiently, based on the C++ Core Guidelines.

**Technical Definition**
The C++ Core Guidelines provide rules: **C.20** (avoid defining default operations), **C.21** (define or delete all copy/move/destructor operations together), **C.30** (define a destructor if a class needs an explicit action at object destruction), **C.31** (all resources acquired by a class must be released by the class's destructor), **C.32** (if a class has a raw pointer or reference, consider whether it might be owning), **C.33** (if a class has an owning pointer member, consider if it is an owner), **C.35** (base class destructor: public virtual or protected non-virtual), **C.40** (define a constructor if a class has an invariant), **C.41** (a constructor should create a fully initialized object), **C.42** (if a constructor cannot construct a valid object, throw an exception), **C.43** (ensure a copyable class has a default constructor? no — C.43 is about value types), **C.44** (prefer default constructors to be simple and non-throwing), **C.45** (don't define a default constructor that only initializes data members), **C.46** (declare single-argument constructors `explicit` by default), **C.47** (define and initialize member variables in declaration order), **C.48** (prefer in-class initializers for constant initializers), **C.60** (copy assignment: non-virtual, take `const&`, return reference), **C.63** (move assignment: non-virtual, take `&&`, return reference), **C.64** (move constructor should move and leave source valid), **C.65** (move assignment should be `noexcept`), **C.66** (make move operations `noexcept`), **C.67** (a polymorphic class should suppress public copy/move), and **C.80–C.90** (virtual functions and overrides).

**Beginner-Friendly Explanation**
Best practices for special members are simple: **use the Rule of Zero when possible**, **define or delete all five together**, **mark move operations `noexcept`**, **make destructors virtual in polymorphic bases**, **declare single-argument constructors `explicit`**, and **initialize members in declaration order**.

### Key Guidelines (Summary Table)

| Guideline | Summary | Recommendation |
|-----------|---------|----------------|
| **C.20** | Avoid defining default operations. | Rule of Zero. |
| **C.21** | Define or delete all copy/move/destructor operations together. | Rule of Five. |
| **C.30** | Define a destructor if a class needs explicit destruction. | Destructor releases resources. |
| **C.31** | All resources acquired must be released by the destructor. | No leaks. |
| **C.35** | Base destructor: public virtual or protected non-virtual. | Correct polymorphic deletion. |
| **C.41** | A constructor should create a fully initialized object. | No half-constructed objects. |
| **C.42** | If a constructor cannot construct a valid object, throw. | Fail loudly. |
| **C.46** | Declare single-argument constructors `explicit`. | Prevent implicit conversions. |
| **C.47** | Initialize members in declaration order. | Avoid reorder bugs. |
| **C.60** | Copy assignment: non-virtual, take `const&`, return reference. | Standard signature. |
| **C.63** | Move assignment: non-virtual, take `&&`, return reference. | Standard signature. |
| **C.66** | Make move operations `noexcept`. | Enable container optimizations. |
| **C.67** | A polymorphic class should suppress public copy/move. | Prevent slicing. |

### Annotated Code Examples

**Example 1: Rule of Zero (C.20)**

```cpp
#include <iostream>
#include <memory>
#include <string>

class Widget {
    std::string name_;
    std::unique_ptr<int> data_;
public:
    Widget(std::string name, int value)
        : name_(std::move(name)), data_(std::make_unique<int>(value)) {}
    // No special members: Rule of Zero
    const std::string& name() const { return name_; }
    int value() const { return *data_; }
};

int main() {
    Widget w("gadget", 42);
    Widget moved = std::move(w);
    std::cout << moved.name() << ": " << moved.value() << '\n';
}
```

**Expected Output**
```
gadget: 42
```

**Why this is best practice?**  
`Widget` composes RAII members and declares no special members, letting the compiler generate correct operations (C.20, Rule of Zero).

**Example 2: `noexcept` move operations (C.66)**

```cpp
#include <iostream>
#include <utility>
#include <vector>

class Buffer {
    int* data_;
    std::size_t size_;
public:
    explicit Buffer(std::size_t n) : data_(new int[n]), size_(n) {}
    ~Buffer() { delete[] data_; }

    Buffer(Buffer&& other) noexcept                    // C.66
        : data_(other.data_), size_(other.size_) {
        other.data_ = nullptr;
        other.size_ = 0;
    }

    Buffer& operator=(Buffer&& other) noexcept {       // C.66
        if (this != &other) {
            delete[] data_;
            data_ = other.data_;
            size_ = other.size_;
            other.data_ = nullptr;
            other.size_ = 0;
        }
        return *this;
    }

    Buffer(const Buffer&) = delete;
    Buffer& operator=(const Buffer&) = delete;
};

int main() {
    std::vector<Buffer> v;
    v.push_back(Buffer(10));   // move constructor used
}
```

**Expected Output**  
(No output; vector reallocation uses the `noexcept` move constructor.)

**Why this is best practice?**  
`noexcept` move operations allow `std::vector` to move elements during reallocation, providing the strong exception guarantee (C.66).

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
- **Code review** – Core Guidelines C.20, C.21, C.35, C.41, C.46, C.47, C.60, C.63, C.66, C.67 are common checkpoints.
- **Static analysis** – clang-tidy enforces many special member function rules.

---

## References

- Special member functions (cppreference) – https://en.cppreference.com/w/cpp/language/member_functions#Special_member_functions
- Default constructor (cppreference) – https://en.cppreference.com/w/cpp/language/default_constructor
- Destructor (cppreference) – https://en.cppreference.com/w/cpp/language/destructor
- Copy constructor (cppreference) – https://en.cppreference.com/w/cpp/language/copy_constructor
- Copy assignment operator (cppreference) – https://en.cppreference.com/w/cpp/language/copy_assignment
- Move constructor (cppreference) – https://en.cppreference.com/w/cpp/language/move_constructor
- Move assignment operator (cppreference) – https://en.cppreference.com/w/cpp/language/move_assignment
- The Rule of Three/Five/Zero (cppreference) – https://en.cppreference.com/w/cpp/language/rule_of_three
- C++ Core Guidelines — C.20: If you can avoid defining default operations, do – https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rc-zero
- C++ Core Guidelines — C.21: If you define or `=delete` any copy, move, or destructor function, define or `=delete` them all – https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rc-five
- C++ Core Guidelines — C.30: Define a destructor if a class needs an explicit action at object destruction – https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rc-dtor
- C++ Core Guidelines — C.31: All resources acquired by a class must be released by the class's destructor – https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rc-dtor-release
- C++ Core Guidelines — C.35: A base class destructor should be either public and virtual, or protected and non-virtual – https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rc-dtor-virtual
- C++ Core Guidelines — C.41: A constructor should create a fully initialized object – https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rc-complete
- C++ Core Guidelines — C.42: If a constructor cannot construct a valid object, throw an exception – https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rc-throw
- C++ Core Guidelines — C.46: By default, declare single-argument constructors `explicit` – https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rc-explicit
- C++ Core Guidelines — C.47: Define and initialize member variables in the order of declaration – https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rc-order
- C++ Core Guidelines — C.60: Make copy assignment non-`virtual`, take the parameter by `const&`, and return by non-`const&` – https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rc-copy-assignment
- C++ Core Guidelines — C.63: Make move assignment non-`virtual`, take the parameter by `&&`, and return by non-`const&` – https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rc-move-assignment
- C++ Core Guidelines — C.66: Make move operations `noexcept` – https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rc-move-noexcept
- C++ Core Guidelines — C.67: A polymorphic class should suppress public copy/move – https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rc-copy-virtual
- Microsoft Learn — Special member functions – https://learn.microsoft.com/en-us/cpp/cpp/special-member-functions
- Microsoft Learn — Rule of Three/Five/Zero (Modern C++) – https://learn.microsoft.com/en-us/cpp/cpp/rule-of-three-five-zero

---

This cheat sheet provides a complete, structured reference for C++ special member functions, covering the default constructor, destructor, copy constructor, copy assignment, move constructor, move assignment, Rule of Three, Rule of Five, Rule of Zero, and best practices. Each section follows a uniform template with definitions, purposes, syntax rules, annotated examples, expected outputs, and authoritative references.