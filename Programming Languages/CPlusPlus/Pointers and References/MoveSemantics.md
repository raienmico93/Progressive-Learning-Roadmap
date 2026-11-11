# Move Semantics — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
Move semantics is a C++11 feature that allows resources (such as dynamically allocated memory, file handles, or sockets) to be **transferred** from one object to another instead of being copied, enabling efficient handling of temporary objects and reducing unnecessary deep copies.

**Technical Definition**
Move semantics is enabled by **rvalue references** (`T&&`), which bind to temporary objects (rvalues) and to objects explicitly marked as movable via `std::move`. A type supports move semantics by defining a **move constructor** (`T(T&&)`) and a **move assignment operator** (`T& operator=(T&&)`), which typically “steal” the source object's resources (e.g., by swapping pointers) and leave the source in a valid but unspecified state. The C++ standard library provides `std::move` (an unconditional cast to an rvalue) and `std::forward` (a conditional cast that preserves value categories) to support move semantics and perfect forwarding. The interaction of move semantics with destructors and copy operations is governed by the **Rule of Three**, **Rule of Five**, and **Rule of Zero**.

**Beginner-Friendly Explanation**
Imagine you have a heavy bookshelf and you're moving to a new house. **Copying** means building an identical bookshelf at the new house and carrying every book over one by one — slow and expensive. **Moving** means picking up the entire bookshelf and carrying it to the new house — fast, because you're just transferring the whole thing. Move semantics lets C++ do the “carry the bookshelf” trick: when an object is about to be destroyed (a temporary), the program can **steal its insides** instead of duplicating them. This makes returning large objects from functions, inserting into containers, and swapping values much faster.

### Key Characteristics

- **Rvalue references** (`T&&`) bind to temporaries and to objects cast with `std::move`.
- **Move constructor and move assignment** transfer resources rather than copying them.
- **Source state after move**: valid but unspecified; it must be destructible and assignable.
- **`std::move` is a cast, not a move**: it produces an rvalue; the actual move happens when a move constructor/assignment is invoked.
- **`std::forward` preserves value categories** in templates (perfect forwarding).
- **Copy elision** (RVO/NRVO) can eliminate moves entirely; C++17 guarantees elision for prvalues.
- **Rule of Three/Five/Zero** governs which special members a class should define.
- **Move semantics interacts with exception safety**: move constructors should be `noexcept` to enable optimizations in containers.

### Prerequisites

- Understanding of value categories (lvalues, rvalues, prvalues, xvalues).
- Familiarity with references and constructors/destructors.
- Knowledge of dynamic memory and resource ownership.
- Basic awareness of templates and type deduction (for forwarding references).

### Related Programming Areas

- **RAII and smart pointers** – `std::unique_ptr` relies on move semantics for ownership transfer.
- **Containers** – `std::vector::push_back(T&&)` and `emplace_back` use move semantics.
- **Perfect forwarding** – `std::forward` and forwarding references.
- **Copy elision** – RVO/NRVO eliminate copies and moves.
- **Exception safety** – `noexcept` move constructors enable strong guarantees in containers.
- **Return value optimization** – returning by value is efficient with move semantics and elision.

### Core Concepts / Features

1. Move Semantics
2. Rvalue References
3. `std::move`
4. Move Constructors
5. Move Assignment Operators
6. Rule of Three
7. Rule of Five
8. Rule of Zero
9. Copy Elision
10. Perfect Forwarding
11. `std::forward`
12. Move Semantics Best Practices

---

## 1. Move Semantics

### Definitions

**Core Definition**
Move semantics is the set of C++11 features that allow resources to be transferred from one object to another instead of copied, typically when the source is a temporary or explicitly marked as movable.

**Technical Definition**
Move semantics is enabled by rvalue references and the move constructor/move assignment operator. When an object is initialized or assigned from an rvalue, the compiler prefers the move constructor or move assignment operator (if available) over the copy versions. These operations transfer ownership of the source's resources, leaving the source in a valid but unspecified state. Move semantics is the foundation for efficient value semantics in modern C++.

**Beginner-Friendly Explanation**
Move semantics is the **“steal instead of copy”** feature. When you have a temporary object that's about to disappear, C++ can take its resources (like its internal buffer) and give them to a new object, instead of making a full copy. It's like taking the batteries out of a toy you're about to throw away and putting them in a new toy — no need to buy new batteries.

### Purposes

- To eliminate unnecessary deep copies of large objects.
- To enable efficient return-by-value from functions.
- To support move-only types (e.g., `std::unique_ptr`, `std::thread`).
- To improve performance of containers (e.g., `std::vector` reallocation).
- To enable perfect forwarding of arguments.
- To provide clear ownership transfer semantics.

### Syntax Rules and Structure

**General Pattern**
```cpp
class T {
public:
    T(T&& other) noexcept;              // move constructor
    T& operator=(T&& other) noexcept;   // move assignment
};
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `T&&` | Rvalue reference parameter (move source). |
| `noexcept` | Enables container optimizations. |
| Resource transfer | Usually via `std::move` on members or pointer swap. |
| Source state | Valid but unspecified after the move. |

**Syntax Rules**
- Move operations take an rvalue reference to the same type.
- Move operations should be `noexcept` when possible.
- The source object must be left in a valid (destructible, assignable) state.
- If a class defines a move constructor or move assignment, the copy operations are implicitly deleted (unless explicitly defined).
- Move operations are not automatically generated if the class has a user-declared destructor, copy constructor, or copy assignment operator.

**Constraints and Limitations**
- Moving from an object leaves it in a valid but unspecified state; do not use it except to reassign or destroy.
- Move operations are not selected for `const` rvalues (they bind to copy constructors instead).
- Move semantics does not help if the type has no cheap way to transfer resources (e.g., a class with only `int` members benefits little).

### Annotated Code Examples

**Example 1: Move vs. copy**

```cpp
#include <iostream>
#include <string>
#include <utility>

int main() {
    std::string a = "a very long string that would be expensive to copy";
    std::string b = a;              // copy: a is unchanged
    std::string c = std::move(a);   // move: a is left in a valid but unspecified state
    std::cout << "b: " << b << '\n';
    std::cout << "c: " << c << '\n';
    std::cout << "a size: " << a.size() << '\n';   // likely 0
}
```

**Expected Output** (implementation-dependent for `a.size()`)
```
b: a very long string that would be expensive to copy
c: a very long string that would be expensive to copy
a size: 0
```

**Why this output?**  
`b = a` copies the string. `c = std::move(a)` moves the internal buffer from `a` to `c`, leaving `a` empty (or in some valid but unspecified state).

### Real-World Cases

- **Returning large objects** – `std::vector<T> make()` returns by value; move/elision avoids copies.
- **Container operations** – `push_back(std::move(obj))` moves instead of copies.
- **Smart pointers** – `std::unique_ptr` is move-only.
- **Swap operations** – `std::swap` uses move semantics.

---

## 2. Rvalue References

### Definitions

**Core Definition**
An rvalue reference (`T&&`, C++11) is a reference that binds to **rvalues** (temporaries and objects marked for moving), enabling move semantics.

**Technical Definition**
An rvalue reference binds to rvalues (prvalues and xvalues) and cannot bind to lvalues (unless the lvalue is cast with `std::move`). When used as a function parameter, an rvalue reference signals that the argument is a temporary or can be treated as one, allowing the function to move from it. Rvalue references are distinct from lvalue references (`T&`) and are the foundation of move constructors, move assignment, and perfect forwarding.

**Beginner-Friendly Explanation**
An rvalue reference is a reference to something that's **about to disappear** — a temporary. It tells the compiler: “This object is expendable; you can take its resources.” You create one with `T&&` or by casting an lvalue with `std::move`.

### Purposes

- To enable move constructors and move assignment.
- To bind to temporaries and allow resource transfer.
- To support perfect forwarding via forwarding references.
- To overload functions for rvalues (e.g., “fast path” for temporaries).

### Syntax Rules and Structure

**General Syntax**
```cpp
T&& rref = temporary;        // bind to rvalue
void f(T&& x);               // rvalue reference parameter
T&& rref = std::move(lvalue); // cast lvalue to rvalue
```

**Component Breakdown**

| Form | Binding |
|------|---------|
| `T&&` (concrete `T`) | Binds only to rvalues. |
| `T&&` (deduced `T` in template) | Forwarding reference: binds to lvalues and rvalues. |

**Syntax Rules**
- A concrete `T&&` binds only to rvalues.
- Inside the function, an rvalue reference parameter is an lvalue (use `std::move` to move from it).
- `std::move` casts an lvalue to an xvalue, allowing it to bind to `T&&`.
- Rvalue references are used for move constructors (`T(T&&)`), move assignment (`T& operator=(T&&)`), and `&&`-qualified member functions.

**Constraints and Limitations**
- Cannot bind to lvalues directly (use `std::move`).
- After moving, the source is in a valid but unspecified state.
- `const T&&` is rarely useful; moving from a `const` rvalue typically results in a copy.

### Annotated Code Examples

**Example 1: Rvalue reference parameter**

```cpp
#include <iostream>
#include <string>
#include <utility>

void process(std::string&& s) {
    std::cout << "rvalue: " << s << '\n';
}

int main() {
    process("temporary");            // OK: temporary
    std::string lv = "lvalue";
    process(std::move(lv));          // OK: cast to rvalue
    // process(lv);                  // error: cannot bind lvalue to &&
}
```

**Expected Output**
```
rvalue: temporary
rvalue: lvalue
```

**Why this output?**  
`process(std::string&&)` accepts rvalues. The temporary string is bound directly; `std::move(lv)` casts `lv` to an rvalue so it can bind.

### Real-World Cases

- **Move constructors** – `T(T&& other)`.
- **Move assignment** – `T& operator=(T&& other)`.
- **`std::move`** – casts lvalues to rvalues.
- **`&&`-qualified member functions** – methods that consume `*this`.

---

## 3. `std::move`

### Definitions

**Core Definition**
`std::move` is a standard library function template that **unconditionally casts its argument to an rvalue reference**, enabling move semantics.

**Technical Definition**
`std::move(x)` is defined as `static_cast<std::remove_reference_t<T>&&>(x)`. It performs an unconditional cast to an rvalue reference. It does **not** move anything by itself; it merely produces an rvalue that can bind to move constructors and move assignment operators. The actual move occurs when a move operation is selected and invoked.

**Beginner-Friendly Explanation**
`std::move` is a **label** that says: “I don't need this object anymore; feel free to move from it.” It doesn't actually move anything — it just tells the compiler that the object can be treated as a temporary, so the move constructor or move assignment can be used.

### Purposes

- To cast an lvalue to an rvalue, enabling move semantics.
- To indicate that an object can be moved from.
- To transfer ownership in move constructors and move assignment.
- To support move-only types (e.g., `std::unique_ptr`).

### Syntax Rules and Structure

**General Syntax**
```cpp
#include <utility>

T&& moved = std::move(lvalue);   // cast to rvalue reference
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `std::move(x)` | Returns `static_cast<remove_reference_t<T>&&>(x)`. |
| Result | An rvalue reference (xvalue) to `x`. |

**Syntax Rules**
- `std::move` is a cast, not an operation; it does not move by itself.
- After `std::move(x)`, `x` is still in scope and valid, but its value is unspecified if a move actually occurs.
- Do not use `std::move` on `const` objects (it produces `const T&&`, which usually copies).
- Do not use `std::move` on objects you need to use afterward.

**Constraints and Limitations**
- `std::move` on a `const` object yields `const T&&`, which typically binds to the copy constructor, not the move constructor.
- Using a moved-from object (except to reassign or destroy) is a logic error.
- `std::move` does not guarantee that a move will occur; it only makes it possible.

### Annotated Code Examples

**Example 1: `std::move` enabling move construction**

```cpp
#include <iostream>
#include <string>
#include <utility>

int main() {
    std::string a = "hello";
    std::string b = std::move(a);   // move construction
    std::cout << "b: " << b << '\n';
    std::cout << "a size: " << a.size() << '\n';   // likely 0
}
```

**Expected Output**
```
b: hello
a size: 0
```

**Why this output?**  
`std::move(a)` casts `a` to an rvalue, so `std::string`'s move constructor is selected. The internal buffer is transferred to `b`, leaving `a` empty.

### Real-World Cases

- **Transferring ownership** – `std::unique_ptr<T> p2 = std::move(p1);`.
- **Container insertion** – `v.push_back(std::move(obj));`.
- **Swap implementations** – `std::swap` uses `std::move`.
- **Factory returns** – returning `std::move(local)` is usually unnecessary (NRVO is better).

---

## 4. Move Constructors

### Definitions

**Core Definition**
A move constructor is a constructor that initializes an object by **transferring resources** from an rvalue of the same type, leaving the source in a valid but unspecified state.

**Technical Definition**
A move constructor has the signature `T(T&& other)` (possibly with `noexcept`). It is selected during overload resolution when the argument is an rvalue of type `T` (or a type convertible to `T`). It typically “steals” the source's resources (e.g., by moving member pointers and setting the source's pointers to null) rather than copying them. A move constructor is implicitly declared (as defaulted) if the class has no user-declared copy operations, move operations, or destructor.

**Beginner-Friendly Explanation**
A move constructor is a **special constructor that takes an object's insides and puts them into a new object**. It's used when the source is a temporary or explicitly moved. It's much faster than copying because it just transfers pointers, not the actual data.

### Purposes

- To efficiently construct a new object from a temporary.
- To transfer ownership of resources without copying.
- To support move-only types.
- To enable efficient return-by-value and container operations.
- To provide `noexcept` guarantees for container optimizations.

### Syntax Rules and Structure

**General Syntax**
```cpp
class T {
public:
    T(T&& other) noexcept;   // move constructor
};
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `T&&` | Rvalue reference to the source. |
| `noexcept` | Recommended for container compatibility. |
| Member initialization | Move each member from `other`. |
| Source reset | Leave `other` in a valid empty state. |

**Syntax Rules**
- The move constructor takes a non-`const` rvalue reference to the same type.
- It should be `noexcept` when possible.
- It must leave the source in a valid (destructible, assignable) state.
- If a class defines a move constructor, the copy constructor is implicitly deleted (unless explicitly defined).
- Move constructors are not implicitly generated if the class has a user-declared destructor, copy constructor, or copy assignment operator.

**Constraints and Limitations**
- Moving from a `const` rvalue does not invoke the move constructor (it binds to the copy constructor).
- The source object after the move is valid but unspecified; do not rely on its value.
- A move constructor that throws can violate container guarantees; mark it `noexcept` when possible.

### Annotated Code Examples

**Example 1: Move constructor for a buffer class**

```cpp
#include <iostream>
#include <utility>

class Buffer {
    int* data_;
    std::size_t size_;
public:
    explicit Buffer(std::size_t n) : data_(new int[n]), size_(n) {
        std::cout << "allocated\n";
    }
    ~Buffer() { delete[] data_; }

    // Move constructor: steal resources
    Buffer(Buffer&& other) noexcept
        : data_(other.data_), size_(other.size_) {
        other.data_ = nullptr;
        other.size_ = 0;
        std::cout << "moved\n";
    }

    Buffer(const Buffer&) = delete;
    Buffer& operator=(const Buffer&) = delete;
};

int main() {
    Buffer a(10);
    Buffer b(std::move(a));   // move construction
}
```

**Expected Output**
```
allocated
moved
```

**Why this output?**  
`Buffer b(std::move(a))` selects the move constructor, which steals `a`'s pointer and sets it to `nullptr`. Only one allocation occurs.

### Real-World Cases

- **`std::vector` reallocation** – elements are moved to new storage.
- **`std::unique_ptr`** – move-only ownership transfer.
- **Returning large objects** – move construction avoids deep copies.
- **`std::string`** – moving internal buffers.

---

## 5. Move Assignment Operators

### Definitions

**Core Definition**
A move assignment operator is an assignment operator that **transfers resources** from an rvalue of the same type into an existing object, releasing the target's old resources.

**Technical Definition**
A move assignment operator has the signature `T& operator=(T&& other)` (possibly with `noexcept`). It is selected when assigning from an rvalue. It must release the target's current resources, transfer the source's resources, and leave the source in a valid but unspecified state. It typically handles self-assignment (or is written to avoid it), and it should be `noexcept` when possible.

**Beginner-Friendly Explanation**
A move assignment operator is like **emptying your closet and filling it with someone else's clothes**. It first throws away what you had (releases old resources), then takes the source's resources. The source is left empty.

### Purposes

- To efficiently reassign an object from a temporary.
- To transfer ownership without copying.
- To support move-only types in assignment contexts.
- To maintain exception safety during assignment.

### Syntax Rules and Structure

**General Syntax**
```cpp
class T {
public:
    T& operator=(T&& other) noexcept;   // move assignment
};
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `T&&` | Rvalue reference to the source. |
| Return `T&` | Enables chaining. |
| `noexcept` | Recommended. |
| Release + steal | Free target resources, take source's. |
| Source reset | Leave `other` valid but empty. |

**Syntax Rules**
- Release the target's current resources before taking the source's.
- Handle self-assignment (or rely on the source being a temporary).
- Leave the source in a valid but unspecified state.
- Return `*this` to allow chaining.
- Mark `noexcept` when possible.

**Constraints and Limitations**
- Self-move-assignment (e.g., `x = std::move(x)`) must be safe (or documented as unsupported).
- A move assignment operator that throws can violate container guarantees.
- The source must not be used after the move except to reassign or destroy.

### Annotated Code Examples

**Example 1: Move assignment operator**

```cpp
#include <iostream>
#include <utility>

class Buffer {
    int* data_;
    std::size_t size_;
public:
    explicit Buffer(std::size_t n) : data_(new int[n]), size_(n) {}
    ~Buffer() { delete[] data_; }

    Buffer(Buffer&& other) noexcept
        : data_(other.data_), size_(other.size_) {
        other.data_ = nullptr;
        other.size_ = 0;
    }

    // Move assignment
    Buffer& operator=(Buffer&& other) noexcept {
        if (this != &other) {          // self-assignment check
            delete[] data_;            // release old resources
            data_ = other.data_;       // steal
            size_ = other.size_;
            other.data_ = nullptr;
            other.size_ = 0;
        }
        std::cout << "move assigned\n";
        return *this;
    }

    Buffer(const Buffer&) = delete;
    Buffer& operator=(const Buffer&) = delete;
};

int main() {
    Buffer a(10), b(20);
    b = std::move(a);   // move assignment
}
```

**Expected Output**
```
move assigned
```

**Why this output?**  
`b = std::move(a)` selects the move assignment operator, which releases `b`'s old buffer, steals `a`'s buffer, and leaves `a` empty.

### Real-World Cases

- **Container element assignment** – `v[i] = std::move(obj);`.
- **Swap operations** – moving into temporaries.
- **Resource handles** – reassigning ownership.

---

## 6. Rule of Three

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

**Example 1: Rule of Three in action**

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
    String b = a;    // copy ctor
    a = b;           // copy assignment
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

## 7. Rule of Five

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

## 8. Rule of Zero

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
    Widget copy = w;               // error? unique_ptr is move-only
    // Actually: copy is ill-formed because unique_ptr is move-only
    Widget moved = std::move(w);   // OK: move
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

## 9. Copy Elision

### Definitions

**Core Definition**
Copy elision is a compiler optimization that **eliminates copy or move operations** by constructing objects directly in their final destination.

**Technical Definition**
Copy elision encompasses **RVO** (Return Value Optimization) for prvalue return expressions and **NRVO** (Named Return Value Optimization) for named local variables. Since C++17, copy elision is **guaranteed** for prvalues: when a prvalue initializes an object of the same type, the object is constructed directly, and no copy/move constructor is required. NRVO remains optional. Copy elision interacts with move semantics: even when elision is not applied, the compiler treats returned locals as rvalues (allowing a move).

**Beginner-Friendly Explanation**
Copy elision is the compiler's **“skip the copy”** trick. If you return a local variable or a temporary from a function, the compiler can construct it directly in the caller's storage, avoiding a copy or move entirely. In C++17, this is guaranteed for temporaries, so returning by value is efficient by design.

### Purposes

- To eliminate unnecessary copies and moves.
- To make return-by-value efficient.
- To allow non-movable types to be returned by value (C++17).
- To reduce compile-time overhead of move semantics.
- To simplify code: `return local;` is efficient without `std::move`.

### Syntax Rules and Structure

**Patterns**
```cpp
T f() {
    return T(args);      // RVO: guaranteed elision (C++17)
}

T g() {
    T local(args);
    return local;        // NRVO: optional elision
}
```

**Component Breakdown**

| Pattern | Elision Type | Guarantee |
|---------|--------------|-----------|
| `return T(args);` | RVO | C++17 guaranteed |
| `return local;` | NRVO | Optional |
| `T x = T(args);` | Temporary elision | C++17 guaranteed |

**Syntax Rules**
- For guaranteed elision, the return expression must be a prvalue of the function's return type.
- NRVO is optional; the compiler may or may not apply it.
- Do not write `return std::move(local);` — it **prevents NRVO** and is a pessimization.
- Copy elision can change observable side effects of copy/move constructors.

**Constraints and Limitations**
- NRVO is not guaranteed; the compiler may choose not to elide.
- Guaranteed elision does not apply if the return expression is a named local or a conditional expression.
- Elision may be disabled with compiler flags (e.g., `-fno-elide-constructors`) for testing.

### Annotated Code Examples

**Example 1: RVO (guaranteed in C++17)**

```cpp
#include <iostream>

struct Tracer {
    Tracer() { std::cout << "default\n"; }
    Tracer(const Tracer&) { std::cout << "copy\n"; }
    Tracer(Tracer&&) { std::cout << "move\n"; }
    ~Tracer() { std::cout << "dtor\n"; }
};

Tracer make() {
    return Tracer{};   // RVO: no copy/move
}

int main() {
    Tracer t = make();
}
```

**Expected Output (C++17)**
```
default
dtor
```

**Why this output?**  
`Tracer{}` is a prvalue; in C++17, it directly initializes `t`. No copy or move constructor is called.

**Example 2: NRVO (optional)**

```cpp
#include <iostream>

struct Tracer {
    Tracer() { std::cout << "default\n"; }
    Tracer(const Tracer&) { std::cout << "copy\n"; }
    Tracer(Tracer&&) { std::cout << "move\n"; }
    ~Tracer() { std::cout << "dtor\n"; }
};

Tracer make() {
    Tracer local;
    return local;   // NRVO may elide the move
}

int main() {
    Tracer t = make();
}
```

**Possible Output (with NRVO)**
```
default
dtor
```

**Possible Output (without NRVO)**
```
default
move
dtor
dtor
```

**Why this varies?**  
NRVO is optional; the compiler may construct `local` directly in `t`'s storage (one object) or move it (two objects). Both are conforming.

### Real-World Cases

- **Returning containers** – `std::vector<T> make_vector()` returns by value efficiently.
- **Factory functions** – `std::make_unique<T>(...)` returns a prvalue that is elided.
- **`std::string`** – returning strings by value is cheap with elision.
- **Code review** – flagging `return std::move(local);` as a pessimization.

---

## 10. Perfect Forwarding

### Definitions

**Core Definition**
Perfect forwarding is the technique of passing arguments through a wrapper function to another function while **preserving their original value category** (lvalue/rvalue) and cv-qualification.

**Technical Definition**
Perfect forwarding is implemented using **forwarding references** (`T&&` where `T` is a deduced template parameter) and `std::forward<T>(arg)`. Reference collapsing rules ensure that lvalues are deduced as `T&` and rvalues as `T`, so `std::forward<T>` casts the argument back to its original value category. This allows wrapper functions (e.g., `std::make_unique`, `emplace_back`) to forward arguments to constructors without extra copies.

**Beginner-Friendly Explanation**
Perfect forwarding is like a **receptionist who passes packages to the right department without repackaging them**. If a package is fragile (rvalue), it stays fragile; if it's heavy (lvalue), it stays heavy. The wrapper function forwards arguments exactly as they were given, so the target function can use move semantics when appropriate.

### Purposes

- To write wrapper functions that forward arguments without extra copies.
- To support move semantics through multiple layers of function calls.
- To implement factories (`make_unique`, `make_shared`) and emplace operations.
- To enable generic code that preserves value categories.

### Syntax Rules and Structure

**General Syntax**
```cpp
template <typename T>
void wrapper(T&& arg) {
    target(std::forward<T>(arg));   // preserve value category
}
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `T&&` | Forwarding reference (only when `T` is deduced). |
| `std::forward<T>(arg)` | Casts `arg` to its original value category. |

**Syntax Rules**
- `T&&` is a forwarding reference only when `T` is a deduced template parameter.
- Use `std::forward<T>(arg)`, not `std::move(arg)`, to forward.
- Forwarding references are greedy; they may require SFINAE or concepts to constrain them.
- Reference collapsing rules determine the deduced type.

**Constraints and Limitations**
- Overloading on forwarding references can be tricky (they match almost everything).
- `std::forward` requires the correct template argument (`T`, not `T&&`).
- Forwarding references do not help with out-parameters.

### Annotated Code Examples

**Example 1: Forwarding wrapper**

```cpp
#include <iostream>
#include <string>
#include <utility>

void target(const std::string& s) { std::cout << "lvalue: " << s << '\n'; }
void target(std::string&& s) { std::cout << "rvalue: " << s << '\n'; }

template <typename T>
void wrapper(T&& arg) {
    target(std::forward<T>(arg));   // preserve value category
}

int main() {
    std::string a = "hello";
    wrapper(a);              // lvalue → lvalue overload
    wrapper("temporary");    // rvalue → rvalue overload
}
```

**Expected Output**
```
lvalue: hello
rvalue: temporary
```

**Why this output?**  
`wrapper(a)` deduces `T = std::string&`; `std::forward<std::string&>(a)` returns an lvalue. `wrapper("temporary")` deduces `T = const char(&)[10]`; the temporary `std::string` is forwarded as an rvalue.

### Real-World Cases

- **`std::make_unique<T>(args...)`** – forwards arguments to `T`'s constructor.
- **`std::make_shared<T>(args...)`** – same.
- **`std::vector::emplace_back(args...)`** – constructs in place.
- **Generic wrappers** – logging, timing, and instrumentation.

---

## 11. `std::forward`

### Definitions

**Core Definition**
`std::forward<T>(arg)` is a utility that **conditionally casts** `arg` to an rvalue if `T` is not an lvalue reference type, preserving the original value category during forwarding.

**Technical Definition**
`std::forward` is defined in `<utility>`. When `T` is an lvalue reference type, `std::forward<T>(arg)` returns an lvalue; otherwise, it returns an rvalue reference. This is the mechanism that makes perfect forwarding possible: it casts the forwarding reference parameter back to the value category it had at the call site.

**Beginner-Friendly Explanation**
`std::forward` is a **label maker** that restores the original value category. If the argument was an lvalue, it stays an lvalue; if it was an rvalue, it becomes an rvalue again. This lets the target function choose between copy and move appropriately.

### Purposes

- To preserve value categories in perfect forwarding.
- To enable move semantics through wrapper functions.
- To avoid unnecessary copies in generic code.
- To implement `make_unique`, `emplace_back`, and similar facilities.

### Syntax Rules and Structure

**General Syntax**
```cpp
std::forward<T>(arg);
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `T` | The deduced template parameter (from the forwarding reference). |
| `arg` | The forwarding reference parameter. |

**Syntax Rules**
- Use only with forwarding references (`T&&` where `T` is deduced).
- `T` must be the exact deduced type.
- `std::forward` is `noexcept`.
- Do not use `std::move` instead of `std::forward` in forwarding contexts (it unconditionally casts to rvalue).

**Constraints and Limitations**
- Using `std::forward` outside the forwarding-reference pattern may not behave as expected.
- Passing the wrong `T` can cause errors or incorrect behavior.

### Annotated Code Examples

**Example 1: `std::forward` vs. `std::move`**

```cpp
#include <iostream>
#include <string>
#include <utility>

void inspect(const std::string& s) { std::cout << "lvalue\n"; }
void inspect(std::string&& s) { std::cout << "rvalue\n"; }

template <typename T>
void good_forward(T&& arg) {
    inspect(std::forward<T>(arg));   // preserves value category
}

template <typename T>
void bad_forward(T&& arg) {
    inspect(std::move(arg));         // always casts to rvalue
}

int main() {
    std::string s = "hello";
    good_forward(s);      // lvalue
    bad_forward(s);       // rvalue (moves from s!)
}
```

**Expected Output**
```
lvalue
rvalue
```

**Why this matters?**  
`good_forward` preserves the lvalue category, so `s` is not moved from. `bad_forward` unconditionally casts to rvalue, moving from `s` even though the caller passed an lvalue.

### Real-World Cases

- **`std::make_unique` / `std::make_shared`** – forward constructor arguments.
- **`emplace_back` / `emplace`** – construct elements in place.
- **Generic logging** – forward arguments without copying.

---

## 12. Move Semantics Best Practices

### Definitions

**Core Definition**
Move semantics best practices are guidelines for implementing and using move constructors, move assignment, `std::move`, and `std::forward` correctly and efficiently.

**Technical Definition**
The C++ Core Guidelines provide specific rules: **C.20** (avoid defining default operations), **C.21** (define or delete all copy/move/destructor operations together), **C.60** (make copy assignment non-virtual, take by `const&`, return by reference), **C.63** (make move assignment non-virtual, take by `&&`, return by reference), **C.64** (move constructor should be `noexcept`), **C.65** (move assignment should be `noexcept`), **C.66** (make move operations `noexcept`), **ES.56** (write `std::move` only when you need to explicitly move), **F.48** (do not return `std::move(local)`), and **R.23** (use `std::make_unique`). These practices ensure correctness, efficiency, and compatibility with the standard library.

**Beginner-Friendly Explanation**
Best practices for move semantics are simple: **use the Rule of Zero when possible**, **mark move operations `noexcept`**, **don't use `std::move` in return statements**, **use `std::forward` for forwarding**, **don't use moved-from objects**, and **prefer standard library types that already handle moves correctly**.

### Key Guidelines (Summary Table)

| Guideline | Summary | Recommendation |
|-----------|---------|----------------|
| **C.20** | Avoid defining default operations. | Use the Rule of Zero. |
| **C.21** | Define or delete all copy/move/destructor operations together. | Rule of Five. |
| **C.64** | Move constructor should be `noexcept`. | Enables container optimizations. |
| **C.65** | Move assignment should be `noexcept`. | Same. |
| **C.66** | Make move operations `noexcept`. | Required for strong guarantees. |
| **ES.56** | Write `std::move` only when you need to explicitly move. | Don't overuse `std::move`. |
| **F.48** | Do not return `std::move(local)`. | Prevents NRVO; pessimization. |
| **R.23** | Use `std::make_unique`. | Exception-safe, avoids `new`. |
| **R.22** | Use `std::make_shared`. | Single allocation. |
| **Moved-from state** | Do not use moved-from objects except to reassign or destroy. | Valid but unspecified. |

### Annotated Code Examples

**Example 1: `noexcept` move constructor (C.64)**

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

    Buffer(Buffer&& other) noexcept                    // noexcept move ctor
        : data_(other.data_), size_(other.size_) {
        other.data_ = nullptr;
        other.size_ = 0;
    }

    Buffer& operator=(Buffer&& other) noexcept {       // noexcept move assign
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
    v.push_back(Buffer(10));   // move ctor used: no copy
}
```

**Expected Output**  
(No output; vector reallocation uses the noexcept move constructor.)

**Why this is best practice?**  
The `noexcept` move constructor allows `std::vector` to move elements during reallocation instead of copying them, providing the strong exception guarantee (C.64, C.65, C.66).

**Example 2: Do not return `std::move(local)` (F.48)**

```cpp
#include <string>

std::string good() {
    std::string s = "hello";
    return s;              // NRVO / automatic move
}

std::string bad() {
    std::string s = "hello";
    return std::move(s);   // pessimization: inhibits NRVO
}
```

**Expected Output**  
(Both compile; `bad` may be slower due to the forced move.)

**Why this is best practice?**  
`return s;` allows NRVO and, if not applied, an automatic move. `return std::move(s);` prevents NRVO and forces a move, which is strictly worse (F.48, ES.56).

### Real-World Cases

- **Standard library** – `std::vector`, `std::string`, `std::unique_ptr` follow these rules.
- **Code review** – Core Guidelines C.20, C.21, C.64–C.66, ES.56, F.48 are common checkpoints.
- **Performance-critical code** – `noexcept` move operations enable vector optimizations.
- **Static analysis** – clang-tidy flags pessimizing moves (`-Wpessimizing-move`).

---

## References

- Value categories (cppreference) – https://en.cppreference.com/w/cpp/language/value_category
- Rvalue reference declarator: `&&` (cppreference) – https://en.cppreference.com/w/cpp/language/reference
- `std::move` (cppreference) – https://en.cppreference.com/w/cpp/utility/move
- `std::forward` (cppreference) – https://en.cppreference.com/w/cpp/utility/forward
- Move constructor (cppreference) – https://en.cppreference.com/w/cpp/language/move_constructor
- Move assignment operator (cppreference) – https://en.cppreference.com/w/cpp/language/move_assignment
- Copy elision (cppreference) – https://en.cppreference.com/w/cpp/language/copy_elision
- The Rule of Three/Five/Zero (cppreference) – https://en.cppreference.com/w/cpp/language/rule_of_three
- C++ Core Guidelines — C.20: If you can avoid defining default operations, do – https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rc-zero
- C++ Core Guidelines — C.21: If you define or `=delete` any copy, move, or destructor function, define or `=delete` them all – https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rc-five
- C++ Core Guidelines — C.64: A move constructor should move and leave its source in a valid state – https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rc-move-ctor
- C++ Core Guidelines — C.65: Make move assignment non-`virtual`, take the parameter by `&&`, and return by non-`const` `&` – https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rc-move-assignment
- C++ Core Guidelines — C.66: Make move operations `noexcept` – https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rc-move-noexcept
- C++ Core Guidelines — ES.56: Write `std::move()` only when you need to explicitly move an object to another scope – https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Res-move
- C++ Core Guidelines — F.48: Don't `return std::move(local)` – https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rf-return-move-local
- C++ Core Guidelines — R.23: Use `make_unique()` to make `unique_ptr`s – https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rr-make_unique
- C++ Core Guidelines — R.22: Use `make_shared()` to make `shared_ptr`s – https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rr-make_shared
- Move Semantics (isocpp.org) – https://isocpp.org/blog/2012/11/universal-references-in-c11-scott-meyers
- Microsoft Learn — Move constructors and move assignment operators (C++) – https://learn.microsoft.com/en-us/cpp/cpp/move-constructors-and-move-assignment-operators-cpp

---

This cheat sheet provides a complete, structured reference for C++ move semantics, covering rvalue references, `std::move`, move constructors, move assignment operators, the Rule of Three/Five/Zero, copy elision, perfect forwarding, `std::forward`, and best practices. Each section follows a uniform template with definitions, purposes, syntax rules, annotated examples, expected outputs, and authoritative references.