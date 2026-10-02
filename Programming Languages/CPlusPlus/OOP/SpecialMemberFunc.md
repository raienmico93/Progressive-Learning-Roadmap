# C++ Special Member Functions — Comprehensive Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
Special member functions are member functions that the compiler can implicitly declare and define for a class. They control how objects are created, copied, moved, and destroyed. There are six special member functions: default constructor, destructor, copy constructor, copy assignment operator, move constructor, and move assignment operator.

**Technical Definition**
The special member functions are: (1) the default constructor, (2) the copy constructor, (3) the copy assignment operator, (4) the move constructor, (5) the move assignment operator, and (6) the destructor. The compiler implicitly declares these functions when they are used (odr-used) and not explicitly declared by the programmer. Implicitly declared special member functions are `public` and `inline`. A special member function is *user-provided* if it is user-declared and not explicitly defaulted or deleted on its first declaration. The set of special member functions and their interactions are governed by the Rule of Zero, Rule of Three, and Rule of Five.

**Beginner-Friendly Explanation**
When you create a class, the compiler quietly writes several functions for you: one to create objects, one to destroy them, one to copy them, and one to move them. These are the "special member functions." For simple classes, the compiler's versions are perfect. But if your class owns a resource (like dynamically allocated memory or a file handle), you need to write your own to avoid shallow copies, double frees, and leaks. The rules of thumb — Rule of Zero, Rule of Three, Rule of Five — tell you when and how many of these functions you should write yourself.

---

### Key Characteristics

- **Six special member functions**: default constructor, destructor, copy constructor, copy assignment, move constructor, move assignment.
- **Implicit generation**: The compiler generates them when needed, but suppresses some when others are user-declared.
- **`= default`**: Explicitly requests the compiler-generated version.
- **`= delete`**: Explicitly removes the function, preventing its use.
- **Rule of Zero/Three/Five**: Guidelines for when to define special member functions.
- **Copy-and-swap**: An idiom for strong exception safety in assignment.
- **Ref-qualifiers**: Restrict member function calls based on lvalue/rvalue state.

---

### Prerequisites

- Solid understanding of classes, constructors, and destructors.
- Familiarity with references, `const` correctness, and rvalue references.
- Knowledge of dynamic memory allocation and resource management.
- Understanding of exception safety guarantees.

---

### Related Programming Areas

- **Resource management** (RAII — Resource Acquisition Is Initialization).
- **Move semantics** (rvalue references, `std::move`).
- **Smart pointers** (`std::unique_ptr`, `std::shared_ptr`).
- **Rule of Zero/Three/Five** (design guidelines).
- **Exception safety** (basic, strong, nothrow guarantees).
- **Polymorphism** (virtual destructors).

---

### Core Concepts / Features

1. **Default Constructor** — Implicit generation vs. `= default` vs. `= delete`.
2. **Destructor** — Virtual destructors for polymorphic base classes.
3. **Copy Constructor and Copy Assignment** — Deep vs. shallow copy.
4. **Move Constructor and Move Assignment** — Rvalue references and `std::move`.
5. **The Rule of Zero, Three, and Five** — When and how many to define.
6. **The Copy-and-Swap Idiom** — Strong exception safety.
7. **Ref-Qualifiers** — `&` and `&&` to restrict calls based on lvalue/rvalue state.

---

## 1. Default Constructor

### Definitions

**Core Definition**
A default constructor is a constructor that can be called with no arguments. It initializes objects that are not given explicit initial values.

**Technical Definition**
A default constructor is a constructor that either has no parameters, or has parameters where all have default arguments. If no user-declared constructors of any kind are provided for a class type, the compiler will always declare a default constructor as an inline public member. If some user-declared constructors are present, the user may still force the automatic generation of a default constructor by declaring it as `= default`. The implicitly declared default constructor is `constexpr` if the constructor selected for every non-static member is `constexpr`.

**Beginner-Friendly Explanation**
A default constructor is the one that runs when you write `MyClass obj;` with no arguments. If you don't write any constructor, the compiler writes one for you. But if you write any other constructor (like one that takes arguments), the compiler stops generating the default one — you have to ask for it explicitly with `= default` or write your own. The default constructor is also what runs for members that aren't explicitly initialized in a constructor's initializer list.

---

### Purposes (all begin with "To")

- **To** allow objects to be created without explicit initial arguments.
- **To** initialize members to their default values.
- **To** enable arrays of objects and containers that default-construct elements.
- **To** provide a valid state for objects that are default-initialized.
- **To** control whether a class can be default-constructed (via `= delete`).

---

### Syntax Rules and Structure

#### Complete General Syntaxes with Breakdowns

**Implicit default constructor (no constructors declared):**
```cpp
class T {
    int x_;
    std::string s_;
};
// Compiler generates: T() : x_(), s_() {}
```

**Explicitly defaulted default constructor:**
```cpp
class T {
public:
    T() = default;
private:
    int x_;
};
```

**User-provided default constructor:**
```cpp
class T {
public:
    T() : x_(0) {}
private:
    int x_;
};
```

**Deleted default constructor:**
```cpp
class T {
public:
    T() = delete;
};
```

#### Syntax Rules

- The default constructor is implicitly declared if no user-declared constructors exist.
- Declaring any constructor suppresses the implicit default constructor.
- `= default` requests the compiler-generated version.
- `= delete` removes the default constructor, preventing default construction.
- The implicitly declared default constructor is `public` and `inline`.
- A member without a default member initializer and without being initialized in the constructor's initializer list is left uninitialized (for fundamental types) or default-constructed (for class types).

#### Constraints and Limitations

- If a class has a member that is not default-constructible, the default constructor is implicitly deleted.
- If a class has a reference member or a `const` member without a default initializer, the default constructor is implicitly deleted.
- A default constructor cannot be a template.
- The default constructor is not generated if the class has a user-declared destructor? (No — the destructor does not suppress the default constructor; only other constructors do.)

---

### Multiple Annotated Complete Step-by-Step Code Examples and Their Expected Outputs

#### Example 1 — Implicit vs. Explicit Default Constructor

```cpp
#include <iostream>
#include <string>

// Step 1: No constructors — implicit default constructor generated
class Implicit {
public:
    int x;
    std::string s;
};

// Step 2: User-declared constructor suppresses implicit default
class UserProvided {
public:
    UserProvided(int value) : x_(value) {}
    // No default constructor available
private:
    int x_;
};

// Step 3: Explicitly defaulted
class Defaulted {
public:
    Defaulted() = default;
    int x = 42;
    std::string s;
};

// Step 4: Deleted default constructor
class NoDefault {
public:
    NoDefault() = delete;
    NoDefault(int value) : x_(value) {}
private:
    int x_;
};

int main() {
    // Step 5: Implicit default constructor works
    Implicit i;
    std::cout << "Implicit: x = " << i.x << ", s = \"" << i.s << "\"" << std::endl;

    // Step 6: UserProvided has no default constructor
    // UserProvided u;  // Error: no default constructor

    // Step 7: Defaulted works
    Defaulted d;
    std::cout << "Defaulted: x = " << d.x << ", s = \"" << d.s << "\"" << std::endl;

    // Step 8: NoDefault cannot be default-constructed
    // NoDefault n;  // Error: call to deleted constructor

    NoDefault n(10);
    std::cout << "NoDefault(10) constructed" << std::endl;

    return 0;
}
```

**Expected Output:**
```
Implicit: x = 0, s = ""
Defaulted: x = 42, s = ""
NoDefault(10) constructed
```

**Why this output:** `Implicit i;` default-initializes `x` (0 for `int`, empty for `std::string`). `Defaulted d;` uses the defaulted constructor, which respects the default member initializer `x = 42`. `NoDefault n(10);` uses the explicit constructor. `NoDefault n;` would fail because the default constructor is deleted.

---

### Real-World Cases with Explanation

**Case 1 — `std::vector`:** `std::vector<int> v;` uses the default constructor, creating an empty vector with no allocated storage.

**Case 2 — `std::string`:** `std::string s;` creates an empty string using the default constructor.

**Case 3 — Smart pointers:** `std::unique_ptr<T> p;` default-constructs a null pointer. `std::unique_ptr<T> p;` is the idiomatic way to create an empty smart pointer.

**Case 4 — Deleted default constructors:** A class that requires an explicit initial value (e.g., a database connection requiring a connection string) deletes its default constructor to force users to provide the required information.

---

### References Links

- Default constructors — cppreference.com - https://en.cppreference.com/w/cpp/language/default_constructor
- Default constructors — Microsoft Learn - https://learn.microsoft.com/en-us/cpp/cpp/default-constructors
- C++ Core Guidelines C.80: Use `= default` if you have to be explicit about using the default semantics - https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rc-default

---

## 2. Destructor (Virtual Destructors)

### Definitions

**Core Definition**
A destructor is a special member function that is called when an object's lifetime ends. It releases resources owned by the object. A virtual destructor ensures that the correct destructor is called when deleting a derived object through a base class pointer.

**Technical Definition**
A destructor is a member function called when the object's lifetime ends. It is invoked automatically when an object goes out of scope, when it is deleted, or when its lifetime ends for any other reason. The destructor has the name `~T` where `T` is the class name; it takes no parameters and has no return type. A destructor can be declared `virtual`; if a class has a virtual destructor, deleting a pointer to the base class will call the derived class's destructor first, then the base's. Without a virtual destructor, deleting a base pointer to a derived object results in undefined behavior.

**Beginner-Friendly Explanation**
A destructor is the cleanup function. When an object dies, its destructor runs to free resources (memory, files, sockets). If you have a base class pointer pointing to a derived object, and you `delete` the pointer, you want both the derived and base destructors to run. For that to happen, the base class destructor must be `virtual`. Without it, only the base destructor runs, and the derived object's resources leak — a dangerous and common bug in polymorphic code.

---

### Purposes (all begin with "To")

- **To** release resources (memory, file handles, network sockets) when an object is destroyed.
- **To** ensure that derived class destructors are called when deleting through a base pointer (virtual destructor).
- **To** implement RAII (Resource Acquisition Is Initialization) — resources are tied to object lifetime.
- **To** allow the compiler to generate an implicit destructor for classes with no resource ownership.
- **To** support exception safety by cleaning up resources during stack unwinding.

---

### Syntax Rules and Structure

#### Complete General Syntaxes with Breakdowns

**Non-virtual destructor:**
```cpp
class T {
public:
    ~T() { /* cleanup */ }
};
```

**Virtual destructor:**
```cpp
class Base {
public:
    virtual ~Base() = default;
};
```

**Pure virtual destructor:**
```cpp
class Base {
public:
    virtual ~Base() = 0;  // must still be defined
};

Base::~Base() = default;
```

#### Syntax Rules

- The destructor has the name `~T`, takes no parameters, and has no return type.
- It can be declared `virtual` for polymorphic base classes.
- It can be declared `= default` to request the compiler-generated version.
- It can be declared `= delete` to prevent destruction (rare).
- A class can have only one destructor.
- The destructor is called automatically; it cannot be called directly (though it can be invoked via placement delete).

#### Constraints and Limitations

- A destructor cannot be a template.
- A destructor cannot be `static`, `const`, or `volatile`.
- If a base class has a virtual destructor, the derived class destructor is automatically virtual.
- A pure virtual destructor must still be defined (unlike other pure virtual functions).
- Declaring a destructor suppresses the implicit move constructor and move assignment operator (but not the copy constructor and copy assignment).

---

### Multiple Annotated Complete Step-by-Step Code Examples and Their Expected Outputs

#### Example 1 — Virtual vs. Non-Virtual Destructor

```cpp
#include <iostream>

// Step 1: Base without virtual destructor
class BaseBad {
public:
    ~BaseBad() { std::cout << "~BaseBad" << std::endl; }
};

class DerivedBad : public BaseBad {
public:
    ~DerivedBad() { std::cout << "~DerivedBad" << std::endl; }
};

// Step 2: Base with virtual destructor
class BaseGood {
public:
    virtual ~BaseGood() { std::cout << "~BaseGood" << std::endl; }
};

class DerivedGood : public BaseGood {
public:
    ~DerivedGood() { std::cout << "~DerivedGood" << std::endl; }
};

int main() {
    std::cout << "--- Bad (no virtual destructor) ---" << std::endl;
    BaseBad* bad = new DerivedBad();
    delete bad;  // UNDEFINED BEHAVIOR: only ~BaseBad runs

    std::cout << "--- Good (virtual destructor) ---" << std::endl;
    BaseGood* good = new DerivedGood();
    delete good;  // Both destructors run

    return 0;
}
```

**Expected Output (implementation-dependent for the bad case):**
```
--- Bad (no virtual destructor) ---
~BaseBad
--- Good (virtual destructor) ---
~DerivedGood
~BaseGood
```

**Why this output:** In the bad case, `delete bad` calls only `~BaseBad` because the destructor is not virtual. `~DerivedBad` is never called, leaking any resources owned by `DerivedBad`. In the good case, `delete good` calls `~DerivedGood` first, then `~BaseGood`, correctly cleaning up both parts of the object.

---

### Real-World Cases with Explanation

**Case 1 — Polymorphic base classes:** Any class designed to be used polymorphically (with virtual functions) should have a virtual destructor. This is enforced by C++ Core Guidelines C.35.

**Case 2 — `std::unique_ptr` and `std::shared_ptr`:** When a `unique_ptr<Base>` owns a `Derived` object, the virtual destructor ensures the correct cleanup. `shared_ptr` captures the deleter at construction, so it works correctly even without a virtual destructor (but it's still recommended).

**Case 3 — RAII classes:** File handles, mutex locks, and network connections use destructors to release resources automatically.

**Case 4 — Pure virtual destructors:** Abstract base classes can declare a pure virtual destructor to make the class abstract while still requiring derived classes to have a destructor. The pure virtual destructor must be defined.

---

### References Links

- Destructors — cppreference.com - https://en.cppreference.com/w/cpp/language/destructor
- Destructors (C++) — Microsoft Learn - https://learn.microsoft.com/en-us/cpp/cpp/destructors-cpp
- C++ Core Guidelines C.35: A base class destructor should be either public and virtual, or protected and non-virtual - https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rc-dtor-virtual

---

## 3. Copy Constructor and Copy Assignment Operator

### Definitions

**Core Definition**
The copy constructor creates a new object as a copy of an existing object. The copy assignment operator replaces the contents of an existing object with a copy of another object.

**Technical Definition**
A copy constructor is a constructor that takes a single parameter of type `T&`, `const T&`, `volatile T&`, or `const volatile T&`. A copy assignment operator is a non-template non-static member function with exactly one parameter of type `T&`, `const T&`, `volatile T&`, or `const volatile T&`. A *trivial* copy constructor/assignment copies the object representation (like `std::memmove`). A *non-trivial* copy constructor/assignment performs a member-wise copy, calling the copy constructor/assignment of each member and base. A *deep copy* duplicates the owned resource; a *shallow copy* duplicates only the pointer to the resource, leading to shared ownership and potential double-free.

**Beginner-Friendly Explanation**
Copying an object means creating a new object with the same value. For simple classes, the compiler does a member-wise copy — that's fine. But if your class owns a resource (like a dynamically allocated array), a member-wise copy copies the pointer, so both objects point to the same memory. When one is destroyed, the other's pointer dangles. A deep copy allocates new memory and copies the contents, so each object owns its own resource. The copy constructor does this at construction time; the copy assignment operator does it when assigning to an existing object.

---

### Purposes (all begin with "To")

- **To** create a new object as a copy of an existing object.
- **To** assign the value of one object to another existing object.
- **To** implement deep copying for classes that own resources.
- **To** ensure that copies are independent (no shared ownership unless intended).
- **To** support value semantics (objects behave like values, not like references).

---

### Syntax Rules and Structure

#### Complete General Syntaxes with Breakdowns

**Copy constructor:**
```cpp
class T {
public:
    T(const T& other) : member_(other.member_) {
        // deep copy if needed
    }
};
```

**Copy assignment operator:**
```cpp
class T {
public:
    T& operator=(const T& other) {
        if (this != &other) {
            // deep copy
        }
        return *this;
    }
};
```

**Defaulted copy operations:**
```cpp
class T {
public:
    T(const T&) = default;
    T& operator=(const T&) = default;
};
```

**Deleted copy operations:**
```cpp
class T {
public:
    T(const T&) = delete;
    T& operator=(const T&) = delete;
};
```

#### Syntax Rules

- The copy constructor takes a single parameter of type `const T&` (typically).
- The copy assignment operator takes a single parameter of type `const T&` and returns `T&`.
- The implicit copy constructor/assignment performs member-wise copy.
- User-declared copy operations suppress the implicit move operations.
- The copy assignment must handle self-assignment.

#### Constraints and Limitations

- The copy constructor cannot be a template (a template is not a copy constructor).
- The copy assignment cannot be a template.
- If a class has a move-only member, the copy operations are implicitly deleted.
- Shallow copies of raw pointers lead to double-free and dangling pointer bugs.
- Copy assignment must be exception-safe (use copy-and-swap for strong safety).

---

### Multiple Annotated Complete Step-by-Step Code Examples and Their Expected Outputs

#### Example 1 — Deep vs. Shallow Copy

```cpp
#include <iostream>
#include <cstring>

// Step 1: Shallow copy (BUGGY)
class ShallowString {
public:
    ShallowString(const char* s) : data_(new char[std::strlen(s) + 1]) {
        std::strcpy(data_, s);
    }

    // Implicit copy constructor does shallow copy (copies pointer)
    ~ShallowString() { delete[] data_; }

    const char* data() const { return data_; }

private:
    char* data_;
};

// Step 2: Deep copy (CORRECT)
class DeepString {
public:
    DeepString(const char* s) : data_(new char[std::strlen(s) + 1]) {
        std::strcpy(data_, s);
    }

    // Copy constructor: deep copy
    DeepString(const DeepString& other)
        : data_(new char[std::strlen(other.data_) + 1]) {
        std::strcpy(data_, other.data_);
    }

    // Copy assignment: deep copy
    DeepString& operator=(const DeepString& other) {
        if (this != &other) {
            delete[] data_;
            data_ = new char[std::strlen(other.data_) + 1];
            std::strcpy(data_, other.data_);
        }
        return *this;
    }

    ~DeepString() { delete[] data_; }

    const char* data() const { return data_; }

private:
    char* data_;
};

int main() {
    // Step 3: Deep copy works correctly
    DeepString a("hello");
    DeepString b = a;  // deep copy
    std::cout << "a: " << a.data() << ", b: " << b.data() << std::endl;

    DeepString c("world");
    c = a;  // deep copy assignment
    std::cout << "c: " << c.data() << std::endl;

    // Step 4: ShallowString would double-free on destruction
    // ShallowString x("hello");
    // ShallowString y = x;  // shallow copy — both point to same memory
    // Destructors both delete[] the same pointer -> UNDEFINED BEHAVIOR

    return 0;
}
```

**Expected Output:**
```
a: hello, b: hello
c: hello
```

**Why this output:** `DeepString` has a user-defined copy constructor and copy assignment that allocate new memory and copy the contents. `b = a` creates an independent copy. `c = a` replaces `c`'s contents with a copy of `a`'s. If `ShallowString` were used, `y = x` would copy the pointer, and both destructors would call `delete[]` on the same memory, causing a double-free.

---

### Real-World Cases with Explanation

**Case 1 — `std::vector`:** `std::vector` implements deep copy: copying a vector allocates new memory and copies all elements.

**Case 2 — `std::string`:** `std::string` implements deep copy (or copy-on-write in some older implementations, but C++11 prohibits COW for `std::string`).

**Case 3 — `std::shared_ptr`:** Copies share ownership (shallow copy of the control block, deep copy of the pointer), incrementing the reference count.

**Case 4 — `std::unique_ptr`:** Copy operations are deleted; only move is allowed.

**Case 5 — File handles:** A class owning a file descriptor should define copy operations that either duplicate the handle (using `dup`) or delete them (move-only).

---

### References Links

- Copy constructors — cppreference.com - https://en.cppreference.com/w/cpp/language/copy_constructor
- Copy assignment operator — cppreference.com - https://en.cppreference.com/w/cpp/language/copy_assignment
- Copy constructors and copy assignment operators (C++) — Microsoft Learn - https://learn.microsoft.com/en-us/cpp/cpp/copy-constructors-and-copy-assignment-operators-cpp
- C++ Core Guidelines C.60: Make copy assignment non-virtual, take the parameter by const&, and return by non-const& - https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rc-copy-assignment

---

## 4. Move Constructor and Move Assignment Operator

### Definitions

**Core Definition**
The move constructor creates a new object by transferring resources from an existing (rvalue) object. The move assignment operator transfers resources from an rvalue object to an existing object. Both leave the source in a valid but unspecified state.

**Technical Definition**
A move constructor is a constructor that takes a single parameter of type `T&&`. A move assignment operator is a non-template non-static member function with exactly one parameter of type `T&&`. Move operations are typically `noexcept` because they do not allocate or throw. They are used when the source is an rvalue (a temporary or an object explicitly cast with `std::move`). The compiler implicitly declares move operations when no user-declared copy operations, move operations, or destructor are present. Declaring a move operation suppresses the implicit copy operations.

**Beginner-Friendly Explanation**
Moving an object is like transferring ownership of a house: the new owner gets the house, and the old owner has nothing. Copying is like building a second identical house. Moving is much cheaper when the object owns expensive resources (like a large array). The move constructor and move assignment operator "steal" the resources from the source and leave it empty (or in a valid but empty state). They are called automatically when the source is a temporary (e.g., the result of a function) or when you explicitly write `std::move(obj)`.

---

### Purposes (all begin with "To")

- **To** transfer resources from a temporary object without copying.
- **To** improve performance by avoiding expensive deep copies.
- **To** enable move-only types (e.g., `std::unique_ptr`) that cannot be copied.
- **To** support efficient container operations (e.g., `std::vector` reallocation).
- **To** implement the Rule of Five for resource-managing classes.

---

### Syntax Rules and Structure

#### Complete General Syntaxes with Breakdowns

**Move constructor:**
```cpp
class T {
public:
    T(T&& other) noexcept
        : data_(other.data_), size_(other.size_) {
        other.data_ = nullptr;
        other.size_ = 0;
    }
};
```

**Move assignment operator:**
```cpp
class T {
public:
    T& operator=(T&& other) noexcept {
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
```

**Defaulted move operations:**
```cpp
class T {
public:
    T(T&&) = default;
    T& operator=(T&&) = default;
};
```

#### Syntax Rules

- The move constructor takes a single parameter of type `T&&`.
- The move assignment operator takes a single parameter of type `T&&` and returns `T&`.
- Move operations should be marked `noexcept` when possible.
- The move constructor should leave the source in a valid but empty state.
- Move operations are implicitly declared if no user-declared copy operations, move operations, or destructor are present.

#### Constraints and Limitations

- Declaring a move operation suppresses the implicit copy operations.
- Declaring a copy operation or destructor suppresses the implicit move operations.
- Move operations do not throw if the resource transfer is `noexcept` (e.g., pointer swap).
- `std::move` does not move anything; it just casts to an rvalue reference.
- The source of a move is left in a valid but unspecified state; do not rely on its value.

---

### Multiple Annotated Complete Step-by-Step Code Examples and Their Expected Outputs

#### Example 1 — Move Constructor and Move Assignment

```cpp
#include <iostream>
#include <utility>
#include <cstring>

class String {
public:
    String(const char* s = "") : data_(new char[std::strlen(s) + 1]) {
        std::strcpy(data_, s);
        std::cout << "Constructor: " << data_ << std::endl;
    }

    // Copy constructor (deep copy)
    String(const String& other)
        : data_(new char[std::strlen(other.data_) + 1]) {
        std::strcpy(data_, other.data_);
        std::cout << "Copy constructor: " << data_ << std::endl;
    }

    // Move constructor (steal resources)
    String(String&& other) noexcept : data_(other.data_) {
        other.data_ = nullptr;
        std::cout << "Move constructor: " << (data_ ? data_ : "null") << std::endl;
    }

    // Move assignment (steal resources)
    String& operator=(String&& other) noexcept {
        if (this != &other) {
            delete[] data_;
            data_ = other.data_;
            other.data_ = nullptr;
            std::cout << "Move assignment: " << (data_ ? data_ : "null") << std::endl;
        }
        return *this;
    }

    ~String() {
        if (data_) std::cout << "Destructor: " << data_ << std::endl;
        delete[] data_;
    }

    const char* data() const { return data_ ? data_ : "(moved)"; }

private:
    char* data_;
};

int main() {
    String a("hello");
    String b = std::move(a);  // move constructor
    std::cout << "a after move: " << a.data() << std::endl;
    std::cout << "b: " << b.data() << std::endl;

    String c("world");
    c = std::move(b);  // move assignment
    std::cout << "b after move: " << b.data() << std::endl;
    std::cout << "c: " << c.data() << std::endl;

    return 0;
}
```

**Expected Output:**
```
Constructor: hello
Move constructor: hello
a after move: (moved)
b: hello
Constructor: world
Move assignment: hello
b after move: (moved)
c: hello
Destructor: hello
Destructor: world
```

**Why this output:** `String b = std::move(a)` calls the move constructor, transferring `a`'s data to `b` and setting `a.data_` to `nullptr`. `c = std::move(b)` calls the move assignment, deleting `c`'s old data, transferring `b`'s data to `c`, and setting `b.data_` to `nullptr`. The destructors run at the end of `main`: `c` (now "hello") and the original `a` (now null, so no message). The source of a move is left in a valid but unspecified state.

---

### Real-World Cases with Explanation

**Case 1 — `std::vector` reallocation:** When a `std::vector` grows, it moves elements from the old buffer to the new one (if the element type has a `noexcept` move constructor); otherwise, it copies.

**Case 2 — `std::unique_ptr`:** Move-only; the move constructor transfers ownership of the pointer.

**Case 3 — Factory functions:** `std::make_unique` and `std::make_shared` return by value, relying on move (or copy elision) to transfer the result efficiently.

**Case 4 — Returning containers:** A function that returns a `std::vector` by value uses move semantics (or NRVO) to avoid copying the entire container.

**Case 5 — Sorting algorithms:** `std::sort` uses move assignment to rearrange elements, avoiding deep copies.

---

### References Links

- Move constructors — cppreference.com - https://en.cppreference.com/w/cpp/language/move_constructor
- Move assignment operator — cppreference.com - https://en.cppreference.com/w/cpp/language/move_assignment
- Move constructors and move assignment operators (C++) — Microsoft Learn - https://learn.microsoft.com/en-us/cpp/cpp/move-constructors-and-move-assignment-operators-cpp
- C++ Core Guidelines C.64: A move operation should move and leave its source in a valid state - https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rc-move-semantic

---

## 5. The Rule of Zero, Three, and Five

### Definitions

**Core Definition**
The Rule of Zero, Three, and Five are design guidelines for deciding when to declare special member functions. The Rule of Zero says prefer classes that need none; the Rule of Three says if you need one of destructor/copy constructor/copy assignment, you need all three; the Rule of Five extends this to include move constructor and move assignment.

**Technical Definition**
- **Rule of Zero**: Classes that have no custom destructor, copy/move constructor, or copy/move assignment operator are preferred. Resource management is delegated to member objects (e.g., smart pointers, containers) that handle it correctly.
- **Rule of Three**: If a class requires a user-defined destructor, copy constructor, or copy assignment operator, it almost certainly requires all three.
- **Rule of Five**: If a class requires a user-defined destructor, copy constructor, or copy assignment operator, it should also define move constructor and move assignment operator for performance (or delete them to prevent expensive copies).

**Beginner-Friendly Explanation**
The Rule of Zero says: don't manage resources yourself. Use `std::vector`, `std::string`, `std::unique_ptr`, and other RAII types, and let them handle the cleanup. Then your class needs no special member functions at all. The Rule of Three says: if you must manage a resource manually, you need to handle destruction, copying, and assignment — all three, because they're intimately connected. The Rule of Five adds move operations for efficiency. Modern C++ strongly prefers the Rule of Zero.

---

### Purposes (all begin with "To")

- **To** guide the decision of which special member functions to declare.
- **To** avoid resource leaks, double-frees, and shallow-copy bugs.
- **To** write classes that are correct by default (Rule of Zero).
- **To** ensure consistency between destruction, copying, and assignment (Rule of Three).
- **To** optimize resource transfer while maintaining correctness (Rule of Five).

---

### Syntax Rules and Structure

#### Complete General Syntaxes with Breakdowns

**Rule of Zero (preferred):**
```cpp
class Widget {
    std::string name_;
    std::vector<int> data_;
    std::unique_ptr<Impl> impl_;
    // No special member functions needed
};
```

**Rule of Three:**
```cpp
class T {
public:
    ~T();                          // destructor
    T(const T&);                   // copy constructor
    T& operator=(const T&);        // copy assignment
    // No move operations (suppressed)
};
```

**Rule of Five:**
```cpp
class T {
public:
    ~T();                          // destructor
    T(const T&);                   // copy constructor
    T& operator=(const T&);        // copy assignment
    T(T&&) noexcept;               // move constructor
    T& operator=(T&&) noexcept;    // move assignment
};
```

#### Syntax Rules

- If you declare any of destructor, copy constructor, or copy assignment, the implicit move operations are suppressed.
- If you declare a move operation, the implicit copy operations are deleted.
- `= default` can be used to restore the implicit behavior.
- `= delete` can be used to prevent copying or moving.

#### Constraints and Limitations

- The Rule of Zero is not always possible (e.g., when interfacing with C APIs that return raw resources).
- The Rule of Three/Five requires careful implementation to avoid bugs.
- Declaring a destructor (even `= default`) suppresses move operations, which can hurt performance.
- Mixing `= default` with user-defined operations requires understanding the suppression rules.

---

### Multiple Annotated Complete Step-by-Step Code Examples and Their Outputs

#### Example 1 — Rule of Zero

```cpp
#include <iostream>
#include <string>
#include <vector>
#include <memory>

// Step 1: Rule of Zero — no special member functions
class Widget {
public:
    Widget(std::string name, int value)
        : name_(std::move(name)), data_(value, 0) {}

    void print() const {
        std::cout << "Widget: " << name_ << ", size: " << data_.size() << std::endl;
    }

private:
    std::string name_;
    std::vector<int> data_;
};

int main() {
    // Step 2: Compiler-generated operations work correctly
    Widget w1("first", 5);
    Widget w2 = w1;             // copy works
    Widget w3 = std::move(w1);  // move works

    w2.print();
    w3.print();

    // w1 is in a valid but unspecified state after move
    return 0;
}
```

**Expected Output:**
```
Widget: first, size: 5
Widget: first, size: 5
```

**Why this output:** `Widget` follows the Rule of Zero. It uses `std::string` and `std::vector` (both RAII types), so the compiler-generated copy and move operations work correctly. `w2 = w1` deep-copies the string and vector. `w3 = std::move(w1)` moves the string and vector from `w1` to `w3`. No manual resource management is needed.

---

#### Example 2 — Rule of Five for a Resource-Managing Class

```cpp
#include <iostream>
#include <utility>
#include <cstring>

// Step 1: Rule of Five
class Buffer {
public:
    explicit Buffer(std::size_t size)
        : size_(size), data_(new int[size]{}) {
        std::cout << "Buffer(" << size << ")" << std::endl;
    }

    ~Buffer() {
        std::cout << "~Buffer(" << size_ << ")" << std::endl;
        delete[] data_;
    }

    Buffer(const Buffer& other)
        : size_(other.size_), data_(new int[other.size_]) {
        std::copy(other.data_, other.data_ + size_, data_);
        std::cout << "Copy Buffer(" << size_ << ")" << std::endl;
    }

    Buffer& operator=(const Buffer& other) {
        if (this != &other) {
            Buffer temp(other);
            swap(temp);
        }
        std::cout << "Copy assign Buffer(" << size_ << ")" << std::endl;
        return *this;
    }

    Buffer(Buffer&& other) noexcept
        : size_(other.size_), data_(other.data_) {
        other.size_ = 0;
        other.data_ = nullptr;
        std::cout << "Move Buffer(" << size_ << ")" << std::endl;
    }

    Buffer& operator=(Buffer&& other) noexcept {
        if (this != &other) {
            delete[] data_;
            size_ = other.size_;
            data_ = other.data_;
            other.size_ = 0;
            other.data_ = nullptr;
        }
        std::cout << "Move assign Buffer(" << size_ << ")" << std::endl;
        return *this;
    }

    void swap(Buffer& other) noexcept {
        std::swap(size_, other.size_);
        std::swap(data_, other.data_);
    }

    std::size_t size() const { return size_; }

private:
    std::size_t size_;
    int* data_;
};

int main() {
    Buffer b1(3);
    Buffer b2 = b1;              // copy
    Buffer b3 = std::move(b1);   // move
    b2 = b3;                     // copy assign
    b3 = std::move(b2);          // move assign

    std::cout << "b3 size: " << b3.size() << std::endl;
    return 0;
}
```

**Expected Output:**
```
Buffer(3)
Copy Buffer(3)
Move Buffer(3)
Copy assign Buffer(3)
Move assign Buffer(3)
b3 size: 3
~Buffer(3)
~Buffer(0)
~Buffer(0)
~Buffer(0)
```

**Why this output:** The Rule of Five class defines all five special member functions. `b2 = b1` uses the copy constructor (deep copy). `b3 = std::move(b1)` uses the move constructor (transfers ownership). `b2 = b3` uses copy assignment (copy-and-swap for strong safety). `b3 = std::move(b2)` uses move assignment. The destructors run in reverse order of construction. The moved-from objects have size 0.

---

### Real-World Cases with Explanation

**Case 1 — Standard library types:** `std::vector`, `std::string`, and `std::map` follow the Rule of Five internally, providing correct copy and move semantics for their resource-managing implementations.

**Case 2 — Smart pointers:** `std::unique_ptr` follows the Rule of Five (move-only). `std::shared_ptr` follows the Rule of Five with reference counting.

**Case 3 — RAII wrappers:** Classes wrapping C APIs (e.g., `FILE*`, `sqlite3*`, `HANDLE`) should follow the Rule of Five or, better, use a smart pointer with a custom deleter to achieve the Rule of Zero.

**Case 4 — Move-only types:** Types like `std::thread`, `std::mutex`, and `std::unique_lock` are move-only (copy operations deleted), following a modified Rule of Five.

**Case 5 — C++ Core Guidelines:** The C++ Core Guidelines strongly recommend the Rule of Zero (C.20: "If you can avoid defining default operations, do").

---

### References Links

- C++ Core Guidelines C.20: If you can avoid defining default operations, do (Rule of Zero) - https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rc-zero
- C++ Core Guidelines C.21: If you define or delete any copy, move, or destructor function, define or delete them all (Rule of Five) - https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rc-five
- Rule of three — cppreference.com - https://en.cppreference.com/w/cpp/language/rule_of_three
- The Rule of Three, Five, And Zero — Sandor Dargo's Blog - https://www.sandordargo.com/blog/2019/04/17/rule-of-3-and-5

---

## 6. The Copy-and-Swap Idiom

### Definitions

**Core Definition**
The copy-and-swap idiom is a technique for implementing the copy assignment operator that provides strong exception safety and reduces code duplication. It works by creating a copy of the source, swapping the copy with the current object, and letting the copy's destructor clean up the old data.

**Technical Definition**
The copy-and-swap idiom implements `operator=` by taking the argument by value (which performs a copy), swapping the value with `*this`, and returning `*this`. The old data is now owned by the temporary (the by-value parameter), which is destroyed at the end of the function. This provides strong exception safety: if the copy constructor throws, the original object is unchanged. It also handles self-assignment naturally (since the copy is made before the swap). The idiom requires a non-throwing `swap` function, typically implemented with `std::swap` on each member.

**Beginner-Friendly Explanation**
The copy-and-swap idiom is a clever trick for writing a safe assignment operator. Instead of trying to delete old data and copy new data in-place (which can leave the object in a broken state if something throws), you make a copy of the source first. If the copy succeeds, you swap the copy with your current data. Now your object has the new data, and the copy has the old data. When the copy is destroyed, it cleans up the old data. If the copy fails, nothing has changed. It's like moving to a new house: you pack everything carefully, then swap the keys, and the old house is cleaned up automatically.

---

### Purposes (all begin with "To")

- **To** achieve strong exception safety in copy assignment.
- **To** handle self-assignment correctly without explicit checks.
- **To** reduce code duplication between copy constructor and copy assignment.
- **To** simplify resource management in the Rule of Five.
- **To** provide a clean, correct implementation of assignment for resource-owning classes.

---

### Syntax Rules and Structure

#### Complete General Syntax with Breakdown

```cpp
class T {
public:
    // Copy-and-swap assignment
    T& operator=(T other) {  // take by value (copy made)
        swap(other);         // swap with current object
        return *this;
    }                        // 'other' destroyed, old data cleaned up

    void swap(T& other) noexcept {
        using std::swap;
        swap(member1_, other.member1_);
        swap(member2_, other.member2_);
    }
};
```

**Component breakdown:**
- `T other` — the parameter is taken by value; the copy is made at the call site.
- `swap(other)` — exchanges the contents of `*this` and `other`.
- `return *this` — returns the modified object.
- The destructor of `other` runs at the end, cleaning up the old data.

#### Syntax Rules

- The assignment operator takes its parameter by value (not by `const&`).
- The `swap` function must be `noexcept` and efficient (usually just pointer swaps).
- The `swap` function should be a member function (or a non-member friend) for ADL.
- The copy-and-swap idiom handles self-assignment naturally.

#### Constraints and Limitations

- Taking the parameter by value means an extra copy compared to the traditional approach (but this is often optimized away or acceptable).
- The `swap` function must be correct and `noexcept`; if it throws, the strong guarantee is lost.
- The copy-and-swap idiom does not work well for types that are not copyable (move-only types need a move-and-swap variant).
- For move assignment, a similar "move-and-swap" idiom can be used.

---

### Multiple Annotated Complete Step-by-Step Code Examples and Their Outputs

#### Example 1 — Copy-and-Swap for a Resource-Managing Class

```cpp
#include <iostream>
#include <utility>
#include <cstring>

class String {
public:
    String(const char* s = "")
        : data_(new char[std::strlen(s) + 1]) {
        std::strcpy(data_, s);
    }

    // Copy constructor
    String(const String& other)
        : data_(new char[std::strlen(other.data_) + 1]) {
        std::strcpy(data_, other.data_);
    }

    // Copy assignment via copy-and-swap
    String& operator=(String other) {  // by value
        swap(other);
        return *this;
    }

    // Move constructor
    String(String&& other) noexcept : data_(other.data_) {
        other.data_ = nullptr;
    }

    // Destructor
    ~String() { delete[] data_; }

    // Swap (noexcept)
    void swap(String& other) noexcept {
        std::swap(data_, other.data_);
    }

    const char* data() const { return data_ ? data_ : "(null)"; }

private:
    char* data_;
};

int main() {
    String a("hello");
    String b("world");

    b = a;  // copy assignment via copy-and-swap
    std::cout << "b after assignment: " << b.data() << std::endl;

    a = a;  // self-assignment
    std::cout << "a after self-assignment: " << a.data() << std::endl;

    return 0;
}
```

**Expected Output:**
```
b after assignment: hello
a after self-assignment: hello
```

**Why this output:** `b = a` calls `operator=(String other)` with `other` a copy of `a`. The `swap` exchanges `b`'s data with `other`'s. `other` now holds `b`'s old data ("world") and is destroyed, freeing it. `b` now holds "hello". For `a = a`, a copy of `a` is made, swapped with `a` (no-op since they're the same), and the copy is destroyed. Self-assignment works correctly without an explicit check.

---

### Real-World Cases with Explanation

**Case 1 — Standard library implementations:** Many standard library implementations use copy-and-swap (or a variant) for `std::vector`, `std::string`, and other containers.

**Case 2 — Exception-safe assignment:** The copy-and-swap idiom is the recommended way to achieve strong exception safety in assignment operators.

**Case 3 — Rule of Five classes:** The copy-and-swap idiom simplifies the Rule of Five by delegating the cleanup to the destructor.

**Case 4 — `std::swap` specialization:** Classes that use copy-and-swap often provide a `swap` member function and a non-member `swap` for ADL.

---

### References Links

- Copy-and-swap — cppreference.com - https://en.cppreference.com/w/cpp/language/operators#Assignment_operator
- Copy-and-swap idiom — Stack Overflow - https://stackoverflow.com/questions/3279543/what-is-the-copy-and-swap-idiom
- C++ Core Guidelines C.83: For value-like types, consider providing a noexcept swap function - https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rc-swap

---

## 7. Ref-Qualifiers for Member Functions (`&` and `&&`)

### Definitions

**Core Definition**
Ref-qualifiers allow member functions to be restricted to being called only on lvalue objects (`&`) or only on rvalue objects (`&&`). They are part of the function's signature and participate in overload resolution.

**Technical Definition**
Ref-qualifiers are part of the function's signature and participate in overload resolution. A member function declared with `&` can only be called on an lvalue; a member function declared with `&&` can only be called on an rvalue. This enables different behaviors for lvalue and rvalue objects, particularly useful for returning references to internal data (lvalue) versus moving data out (rvalue). Ref-qualifiers can be combined with `const` and `volatile` qualifiers.

**Beginner-Friendly Explanation**
Normally, a member function can be called on both lvalue and rvalue objects. Ref-qualifiers let you say: "This function is only for lvalues" (`&`) or "only for rvalues" (`&&`). For example, a `get()` method might return a reference to internal data when called on an lvalue, but move the data out when called on an rvalue (since the object is about to be destroyed). This is an advanced but powerful optimization technique.

---

### Purposes (all begin with "To")

- **To** restrict member function calls based on whether the object is an lvalue or rvalue.
- **To** optimize data extraction from temporary objects (move instead of copy).
- **To** prevent misuse of functions on temporaries (e.g., returning a reference to a temporary's member).
- **To** provide different behavior for lvalue and rvalue objects.
- **To** enable efficient "consume" operations on rvalues.

---

### Syntax Rules and Structure

#### Complete General Syntaxes with Breakdowns

**Lvalue-qualified:**
```cpp
class T {
public:
    T& get() & { return *this; }  // only on lvalues
};
```

**Rvalue-qualified:**
```cpp
class T {
public:
    T&& get() && { return std::move(*this); }  // only on rvalues
};
```

**Const and ref-qualified:**
```cpp
class T {
public:
    const T& get() const & { return *this; }
    T&& get() && { return std::move(*this); }
};
```

#### Syntax Rules

- Ref-qualifiers appear after the parameter list and any cv-qualifiers.
- `&` restricts the function to lvalue objects.
- `&&` restricts the function to rvalue objects.
- A function without a ref-qualifier can be called on both lvalues and rvalues.
- Ref-qualifiers participate in overload resolution.

#### Constraints and Limitations

- Ref-qualifiers cannot be applied to static member functions.
- Ref-qualifiers cannot be applied to constructors or destructors.
- Ref-qualifiers are part of the function's signature; two functions differing only in ref-qualifier are distinct overloads.
- Ref-qualifiers require C++11 or later.

---

### Multiple Annotated Complete Step-by-Step Code Examples and Their Outputs

#### Example 1 — Ref-Qualified `get()` for Lvalue and Rvalue

```cpp
#include <iostream>
#include <utility>
#include <string>

class Data {
public:
    Data(std::string value) : value_(std::move(value)) {}

    // Lvalue: return a reference to the internal data
    const std::string& get() const & {
        std::cout << "Lvalue get" << std::endl;
        return value_;
    }

    // Rvalue: move the internal data out
    std::string get() && {
        std::cout << "Rvalue get" << std::endl;
        return std::move(value_);
    }

private:
    std::string value_;
};

Data make_data() {
    return Data("temporary");
}

int main() {
    Data d("persistent");

    // Step 1: Lvalue call
    const std::string& ref = d.get();
    std::cout << "ref: " << ref << std::endl;

    // Step 2: Rvalue call
    std::string moved = make_data().get();
    std::cout << "moved: " << moved << std::endl;

    return 0;
}
```

**Expected Output:**
```
Lvalue get
ref: persistent
Rvalue get
moved: temporary
```

**Why this output:** `d.get()` is called on an lvalue `d`, so the `const &` overload is selected, returning a reference to the internal string. `make_data().get()` is called on an rvalue (the temporary returned by `make_data`), so the `&&` overload is selected, moving the string out of the temporary. This avoids an unnecessary copy.

---

#### Example 2 — Ref-Qualified Assignment

```cpp
#include <iostream>
#include <utility>

class Widget {
public:
    Widget(int id) : id_(id) {}

    // Lvalue assignment: copy
    Widget& operator=(const Widget& other) & {
        std::cout << "Copy assignment" << std::endl;
        id_ = other.id_;
        return *this;
    }

    // Rvalue assignment: move
    Widget& operator=(Widget&& other) && {
        std::cout << "Move assignment (rvalue)" << std::endl;
        id_ = other.id_;
        other.id_ = -1;
        return *this;
    }

    int id() const { return id_; }

private:
    int id_;
};

int main() {
    Widget a(1), b(2);

    a = b;  // lvalue assignment (copy)
    std::cout << "a.id = " << a.id() << std::endl;

    Widget(3) = Widget(4);  // rvalue assignment (move)
    std::cout << "Temporary assignment done" << std::endl;

    return 0;
}
```

**Expected Output:**
```
Copy assignment
a.id = 2
Move assignment (rvalue)
Temporary assignment done
```

**Why this output:** `a = b` calls the lvalue-qualified `operator=` (copy). `Widget(3) = Widget(4)` calls the rvalue-qualified `operator=` (move), since both operands are temporaries. This prevents accidental assignment to temporaries while allowing intentional moves.

---

### Real-World Cases with Explanation

**Case 1 — `std::optional::value()`:** `std::optional` provides `value() &` returning `T&` and `value() &&` returning `T&&`, allowing efficient extraction from temporaries.

**Case 2 — `std::string::substr()`:** Similar patterns are used in string manipulation to avoid copies when the source is a temporary.

**Case 3 — Proxy objects:** Ref-qualifiers on `operator[]` can return a proxy that behaves differently based on whether the object is an lvalue or rvalue.

**Case 4 — `std::ranges`:** Views use ref-qualifiers to ensure that views do not outlive the ranges they reference.

**Case 5 — Smart pointers:** Custom smart pointers can use ref-qualifiers to prevent `*ptr = value` on temporaries.

---

### References Links

- Member functions: ref-qualifiers — cppreference.com - https://en.cppreference.com/w/cpp/language/member_functions#ref-qualified_member_functions
- Ref-qualifiers — Microsoft Learn - https://learn.microsoft.com/en-us/cpp/cpp/ref-qualifiers
- Ref-qualifiers — C++ Stories - https://www.cppstories.com/2020/04/ref-qualifiers.html

---

## Summary Table of Special Member Functions

| Function | Signature | Purpose | Rule of Zero/Three/Five |
|---|---|---|---|
| Default Constructor | `T()` | Creates object with default state | Generated if no constructors declared |
| Destructor | `~T()` | Releases resources | If declared, suppresses move operations |
| Copy Constructor | `T(const T&)` | Creates independent copy | If declared, suppresses move operations |
| Copy Assignment | `T& operator=(const T&)` | Replaces contents with copy | If declared, suppresses move operations |
| Move Constructor | `T(T&&) noexcept` | Transfers resources from rvalue | If declared, deletes copy operations |
| Move Assignment | `T& operator=(T&&) noexcept` | Transfers resources from rvalue | If declared, deletes copy operations |

---

## References (Consolidated)

- Default constructors — cppreference.com - https://en.cppreference.com/w/cpp/language/default_constructor
- Destructors — cppreference.com - https://en.cppreference.com/w/cpp/language/destructor
- Copy constructors — cppreference.com - https://learn.microsoft.com/en-us/cpp/cpp/copy-constructors-and-copy-assignment-operators-cpp
- Move constructors — cppreference.com - https://learn.microsoft.com/en-us/cpp/cpp/move-constructors-and-move-assignment-operators-cpp
- Rule of three — cppreference.com - https://en.cppreference.com/w/cpp/language/rule_of_three
- C++ Core Guidelines C.20: If you can avoid defining default operations, do (Rule of Zero) - https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rc-zero
- C++ Core Guidelines C.21: If you define or delete any copy, move, or destructor function, define or delete them all (Rule of Five) - https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rc-five
- C++ Core Guidelines C.35: A base class destructor should be either public and virtual, or protected and non-virtual - https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rc-dtor-virtual
- C++ Core Guidelines C.60: Make copy assignment non-virtual, take the parameter by const&, and return by non-const& - https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rc-copy-assignment
- C++ Core Guidelines C.64: A move operation should move and leave its source in a valid state - https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rc-move-semantic
- C++ Core Guidelines C.83: For value-like types, consider providing a noexcept swap function - https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rc-swap
- Default constructors — Microsoft Learn - https://learn.microsoft.com/en-us/cpp/cpp/default-constructors
- Destructors (C++) — Microsoft Learn - https://learn.microsoft.com/en-us/cpp/cpp/destructors-cpp
- Copy-and-swap idiom — Stack Overflow - https://stackoverflow.com/questions/3279543/what-is-the-copy-and-swap-idiom
- The Rule of Three, Five, And Zero — Sandor Dargo's Blog - https://www.sandordargo.com/blog/2019/04/17/rule-of-3-and-5
- Ref-qualifiers — C++ Stories - https://www.cppstories.com/2020/04/ref-qualifiers.html