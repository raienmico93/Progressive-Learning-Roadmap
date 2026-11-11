# Classes and Objects — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
A class is a user-defined type that bundles **data** (data members) and **operations** (member functions) into a single unit, providing encapsulation and abstraction. An object is an **instance** of a class — a concrete entity in memory whose type is the class.

**Technical Definition**
A class is a user-defined type whose definition introduces a new type into the program. It declares a set of members: data members, member functions, nested types, enumerators, and member templates. Access to members is controlled by access specifiers (`public`, `private`, `protected`). Objects are instances of class types; they occupy storage, have a lifetime governed by constructors and destructors, and are accessed through member functions or (for `public` members) directly. The C++ object model supports value semantics (copy/move), inheritance, and polymorphism (via virtual functions).

**Beginner-Friendly Explanation**
A class is a **blueprint** for creating objects. Think of a class as the architectural plan for a house, and an object as an actual house built from that plan. The blueprint defines what parts the house has (data members, like rooms and doors) and what you can do with it (member functions, like opening doors or turning on lights). You can build many houses from the same blueprint — each is a separate object. Classes let you model real-world entities with both **state** and **behavior**, and they let you control who can see or change that state.

### Key Characteristics

- **Encapsulation**: Data and operations are bundled; access is controlled by `public`/`private`/`protected`.
- **Abstraction**: Users interact through a public interface without knowing implementation details.
- **Value semantics**: Class objects can be copied and moved (subject to the special member functions).
- **Lifetime management**: Constructors initialize; destructors clean up.
- **Special members**: Default constructor, destructor, copy constructor, copy assignment, move constructor, move assignment.
- **Inheritance and polymorphism**: Classes can derive from base classes and override virtual functions.
- **`this` pointer**: Non-static member functions have access to the implicit object pointer.
- **`= default` and `= delete`**: Control which special members the compiler generates.

### Prerequisites

- Understanding of variables, functions, and scope.
- Familiarity with references and pointers.
- Basic knowledge of memory management and RAII.
- Awareness of move semantics and value categories (for copy/move members).

### Related Programming Areas

- **Object-oriented programming** – encapsulation, inheritance, polymorphism.
- **RAII** – constructors and destructors manage resources.
- **Move semantics** – move constructor and move assignment.
- **Rule of Three/Five/Zero** – governing special member functions.
- **Templates** – class templates generalize classes over types.
- **Design patterns** – classes are the building blocks of most patterns.

### Core Concepts / Features

1. Classes
2. Objects
3. Instances
4. Class Declaration
5. Class Definition
6. Class Members
7. Data Members
8. Member Functions
9. Access Specifiers (`public`, `private`, `protected`)
10. `this` Pointer
11. Constructor
12. Destructor
13. Copy Constructor
14. Move Constructor
15. Copy Assignment
16. Move Assignment
17. Default Constructor
18. Deleted Functions
19. Defaulted Functions (`= default`, `= delete`)
20. Class Best Practices

---

## 1. Classes

### Definitions

**Core Definition**
A class is a user-defined type that groups data and operations into a single entity, providing encapsulation and abstraction.

**Technical Definition**
A class is a user-defined type introduced by a *class-specifier* (`class` or `struct`). It defines a set of members: data members (non-static and static), member functions (non-static and static), nested types, enumerators, and member templates. Members have access control (`public`, `private`, `protected`). A class definition introduces a complete type (after the closing brace); a class declaration (forward declaration) introduces an incomplete type.

**Beginner-Friendly Explanation**
A class is a **recipe** or **blueprint**. It describes what data an object will hold and what operations it supports. You define the class once, and then you can create as many objects (instances) as you need.

### Purposes

- To model real-world entities with state and behavior.
- To encapsulate data and protect it from unauthorized access.
- To provide a clear interface (public member functions) separate from implementation.
- To enable code reuse through inheritance and templates.
- To support value semantics and RAII.

### Syntax Rules and Structure

**General Syntax**
```cpp
class ClassName {
public:
    // public members
private:
    // private members
protected:
    // protected members
};   // semicolon required
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `class ClassName` | Introduces the class type. |
| `public:` | Members accessible to everyone. |
| `private:` | Members accessible only within the class. |
| `protected:` | Members accessible within the class and derived classes. |
| `};` | The closing brace must be followed by a semicolon. |

**Syntax Rules**
- `class` and `struct` differ only in default access (`class` → `private`, `struct` → `public`) and default inheritance.
- The class definition ends with `};` (semicolon required).
- Members are declared inside the class body.
- A class can have a forward declaration: `class ClassName;` (incomplete type).

**Constraints and Limitations**
- A class cannot contain an instance of itself as a non-static data member (infinite size); use a pointer or reference instead.
- Member functions defined inside the class body are implicitly `inline`.

### Annotated Code Examples

**Example 1: Basic class definition**

```cpp
#include <iostream>

class Rectangle {
public:
    double width;
    double height;

    double area() const {
        return width * height;
    }
};

int main() {
    Rectangle r;
    r.width = 3.0;
    r.height = 4.0;
    std::cout << r.area() << '\n';
}
```

**Expected Output**
```
12
```

**Why this output?**  
`Rectangle` is a class with two data members and one member function. `r` is an object; `r.area()` computes `3.0 * 4.0 = 12`.

### Real-World Cases

- **Domain models** – `Customer`, `Order`, `Product`.
- **GUI widgets** – `Button`, `Window`, `TextBox`.
- **Data structures** – `Vector`, `Matrix`, `Complex`.
- **System resources** – `File`, `Socket`, `Thread`.

---

## 2. Objects

### Definitions

**Core Definition**
An object is an **instance** of a class — a concrete entity in memory with a type, a storage location, a lifetime, and a value.

**Technical Definition**
An object is a region of storage with a type. For class types, an object contains its non-static data members (and base class subobjects). Its lifetime begins when its constructor completes and ends when its destructor starts. Objects can have automatic, dynamic, static, or thread storage duration. The C++ object model defines how objects are laid out, how they are accessed, and how they are copied or moved.

**Beginner-Friendly Explanation**
An object is an **actual thing** built from a class blueprint. If `Rectangle` is the blueprint, then `Rectangle r;` creates an actual rectangle in memory. Each object has its own copy of the data members and can call the member functions.

### Purposes

- To represent concrete entities in a program.
- To hold state (data members) and provide behavior (member functions).
- To participate in expressions, function calls, and containers.
- To manage resources through RAII.

### Syntax Rules and Structure

**General Syntax**
```cpp
ClassName objectName;                    // default-initialized
ClassName objectName(args);              // direct-initialized
ClassName objectName = other;            // copy-initialized
ClassName* ptr = new ClassName(args);    // dynamically allocated
```

**Component Breakdown**

| Form | Meaning |
|------|---------|
| `ClassName obj;` | Default-initialization (calls default constructor). |
| `ClassName obj(args);` | Direct-initialization (calls matching constructor). |
| `ClassName obj = other;` | Copy-initialization. |
| `new ClassName(args)` | Dynamic allocation. |

**Syntax Rules**
- Objects can be created on the stack, heap, or with static/thread storage duration.
- The constructor runs at creation; the destructor runs at destruction.
- Objects can be copied (via copy constructor) or moved (via move constructor).

**Constraints and Limitations**
- An object of a class with a deleted default constructor cannot be default-constructed.
- Objects of abstract classes cannot be instantiated (only pointers/references to them).

### Annotated Code Examples

**Example 1: Creating objects in different ways**

```cpp
#include <iostream>

class Point {
public:
    int x, y;
    Point() : x(0), y(0) { std::cout << "default\n"; }
    Point(int x_, int y_) : x(x_), y(y_) { std::cout << "(" << x << "," << y << ")\n"; }
};

int main() {
    Point a;              // default constructor
    Point b(3, 4);        // parameterized constructor
    Point c = b;          // copy constructor (implicit)
    Point* d = new Point(5, 6);   // dynamic
    delete d;
    std::cout << c.x << ',' << c.y << '\n';
}
```

**Expected Output**
```
default
(3,4)
(5,6)
3,4
```

**Why this output?**  
`a` uses the default constructor; `b` uses the parameterized one; `c` copies `b`; `d` is dynamically allocated and deleted.

### Real-World Cases

- **Stack objects** – local variables of class type.
- **Heap objects** – created with `new` or `std::make_unique`.
- **Static objects** – global or function-local `static` objects.
- **Container elements** – objects stored in `std::vector`, `std::map`, etc.

---

## 3. Instances

### Definitions

**Core Definition**
An instance is a specific object created from a class; the terms “instance” and “object” are often used interchangeably.

**Technical Definition**
An instance of a class is an object whose type is that class (or a derived class). Each instance has its own copy of the class's non-static data members and shares the class's member functions and static members. The term emphasizes the relationship between the object and its class — the object is an *instance* of the class.

**Beginner-Friendly Explanation**
“Instance” and “object” mean the same thing. When you create a `Rectangle` object, you've created an **instance** of the `Rectangle` class. It's like saying a particular house is an instance of the architectural plan.

### Purposes

- To emphasize the object-class relationship.
- To describe multiple objects of the same class.
- To clarify that each object has its own state.

### Syntax Rules and Structure

Instances are created the same way as objects:
```cpp
ClassName instance1;
ClassName instance2(args);
```

**Component Breakdown**

| Term | Meaning |
|------|---------|
| Class | The blueprint. |
| Instance | A concrete object built from the blueprint. |

**Syntax Rules**
- Each instance has its own non-static data members.
- All instances share static members and member function code.
- The term “instance” is not a keyword; it's conceptual.

**Constraints and Limitations**
- Instances of the same class are independent (modifying one does not affect another, unless they share resources).
- Instances cannot exist for abstract classes.

### Annotated Code Examples

**Example 1: Multiple instances**

```cpp
#include <iostream>

class Counter {
public:
    int count = 0;
    void increment() { ++count; }
};

int main() {
    Counter a, b;
    a.increment();
    a.increment();
    b.increment();
    std::cout << a.count << ' ' << b.count << '\n';
}
```

**Expected Output**
```
2 1
```

**Why this output?**  
`a` and `b` are separate instances, each with its own `count`. Incrementing `a` does not affect `b`.

### Real-World Cases

- **Multiple customers** – each `Customer` object is an instance with its own data.
- **Multiple game entities** – each `Enemy` instance has its own health and position.
- **Multiple connections** – each `Connection` instance manages its own socket.

---

## 4. Class Declaration

### Definitions

**Core Definition**
A class declaration introduces the class name (and optionally its members) into a scope without necessarily providing a complete definition.

**Technical Definition**
A class declaration can be:
- A **forward declaration**: `class ClassName;` — introduces an incomplete type, sufficient for pointers/references.
- A **complete declaration/definition**: `class ClassName { ... };` — introduces a complete type with all members.

A forward declaration allows the class name to be used before its full definition is available.

**Beginner-Friendly Explanation**
A class declaration is a **promise** that a class with a certain name exists. A forward declaration (`class Foo;`) says “there will be a class `Foo`” without saying what's inside. A full definition says what's inside. Forward declarations are useful when you only need to mention the class (e.g., as a pointer type) before defining it.

### Purposes

- To introduce a class name for use in pointers, references, and function signatures.
- To break circular dependencies between headers.
- To reduce compile-time dependencies.
- To separate interface from implementation.

### Syntax Rules and Structure

**Forward Declaration**
```cpp
class ClassName;   // incomplete type
```

**Full Definition**
```cpp
class ClassName {
    // members
};
```

**Component Breakdown**

| Form | Meaning |
|------|---------|
| `class ClassName;` | Forward declaration (incomplete type). |
| `class ClassName { ... };` | Full definition (complete type). |

**Syntax Rules**
- A forward-declared class is an incomplete type; you can declare pointers/references to it, but not instantiate it or access members.
- A class can be declared multiple times but defined only once (ODR).
- `struct` and `class` forward declarations are interchangeable (both introduce the same type).

**Constraints and Limitations**
- You cannot create objects of an incomplete type.
- You cannot access members of an incomplete type.
- Forward declarations must match the eventual definition's kind (`class`/`struct` can be mixed for the same type).

### Annotated Code Examples

**Example 1: Forward declaration**

```cpp
#include <iostream>

class Widget;   // forward declaration

void use(Widget* w);   // OK: pointer to incomplete type

class Widget {         // full definition
public:
    int value = 42;
};

void use(Widget* w) {
    std::cout << w->value << '\n';
}

int main() {
    Widget w;
    use(&w);
}
```

**Expected Output**
```
42
```

**Why this output?**  
The forward declaration allows `use` to be declared before `Widget` is fully defined. The definition provides the complete type needed for instantiation and member access.

### Real-World Cases

- **Header files** – forward-declare classes to avoid including their headers.
- **Circular dependencies** – break cycles with forward declarations.
- **Pimpl idiom** – forward-declare the implementation class.

---

## 5. Class Definition

### Definitions

**Core Definition**
A class definition provides the complete specification of a class, including all its members, making it a **complete type**.

**Technical Definition**
A class definition consists of the class-key (`class` or `struct`), the class name, an optional base-clause, and a member-specification enclosed in braces, followed by a semicolon. It introduces a complete type, allowing objects to be created, members to be accessed, and the class to be used in all contexts requiring a complete type.

**Beginner-Friendly Explanation**
A class definition is the **full blueprint** — it lists all the parts and behaviors. Once you have the full definition, you can create objects and use them.

### Purposes

- To provide the complete type for instantiation and member access.
- To specify data members, member functions, and access control.
- To establish the class's interface and implementation.
- To enable inheritance and polymorphism.

### Syntax Rules and Structure

**General Syntax**
```cpp
class ClassName : base_classes {
    access_specifier:
        member_declarations
};   // semicolon required
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `class ClassName` | Class key and name. |
| `: base_classes` | Optional base class list. |
| `{ ... }` | Member specification. |
| `;` | Required terminator. |

**Syntax Rules**
- The definition must end with `};`.
- Members can be data members, member functions, nested types, etc.
- Member functions defined in the class body are implicitly `inline`.
- A class can be defined only once per translation unit (ODR).

**Constraints and Limitations**
- A class cannot be defined inside a function (except local classes, which have restrictions).
- A class cannot contain an instance of itself as a non-static data member.

### Annotated Code Examples

**Example 1: Complete class definition**

```cpp
#include <iostream>
#include <string>

class Person {
private:
    std::string name_;
    int age_;
public:
    Person(std::string name, int age) : name_(std::move(name)), age_(age) {}
    const std::string& name() const { return name_; }
    int age() const { return age_; }
};

int main() {
    Person p("Alice", 30);
    std::cout << p.name() << " is " << p.age() << '\n';
}
```

**Expected Output**
```
Alice is 30
```

**Why this output?**  
The complete class definition provides the constructor, data members, and accessor functions needed to create and use `Person`.

### Real-World Cases

- **Header files** – class definitions are typically placed in headers.
- **Implementation files** – member function definitions can be in `.cpp` files.
- **Library interfaces** – public class definitions form the library API.

---

## 6. Class Members

### Definitions

**Core Definition**
Class members are the entities declared inside a class definition: data members, member functions, nested types, enumerators, and member templates.

**Technical Definition**
Members of a class include non-static data members, static data members, non-static member functions, static member functions, nested types (classes, enums, typedefs), enumerators, and member templates. Non-static members are associated with individual objects; static members are associated with the class itself. Access to members is controlled by access specifiers.

**Beginner-Friendly Explanation**
Class members are the **parts and behaviors** listed in the blueprint. Data members are the parts (variables); member functions are the behaviors (methods); nested types and static members are shared or organizational elements.

### Purposes

- To define the state (data members) and behavior (member functions) of objects.
- To organize related types and constants within the class scope.
- To support encapsulation through access control.
- To enable static (class-level) state and behavior.

### Syntax Rules and Structure

**Member Categories**

| Category | Example |
|----------|---------|
| Non-static data member | `int x;` |
| Static data member | `static int count;` |
| Non-static member function | `void f();` |
| Static member function | `static void g();` |
| Nested type | `struct Inner { ... };` |
| Enumerator | `enum Color { Red, Green };` |
| Member template | `template<typename T> void h(T);` |

**Syntax Rules**
- Non-static data members cannot be `auto`, `register`, or `extern`.
- Static data members must be defined outside the class (or `inline` since C++17).
- Member functions defined in the class body are implicitly `inline`.
- Nested types are in the class scope.

**Constraints and Limitations**
- Non-static data members cannot be initialized with `auto` (until C++14 for `static constexpr`).
- Static data members cannot be `mutable`.
- Bit-fields have restrictions.

### Annotated Code Examples

**Example 1: Various member types**

```cpp
#include <iostream>

class Example {
public:
    int data = 10;                       // non-static data member
    static int count;                    // static data member (defined outside)

    void print() const {                 // non-static member function
        std::cout << data << '\n';
    }

    static void show_count() {           // static member function
        std::cout << count << '\n';
    }

    struct Nested { int value; };        // nested type
};

int Example::count = 0;   // definition of static member

int main() {
    Example e;
    e.print();
    Example::count = 5;
    Example::show_count();
    Example::Nested n{42};
    std::cout << n.value << '\n';
}
```

**Expected Output**
```
10
5
42
```

**Why this output?**  
The class has a non-static data member (`data`), a static data member (`count`), member functions, and a nested type. Each is accessed appropriately.

### Real-World Cases

- **Data members** – store object state.
- **Member functions** – implement behavior.
- **Static members** – class-level counters, factories, constants.
- **Nested types** – helper types scoped to the class.

---

## 7. Data Members

### Definitions

**Core Definition**
Data members are variables declared inside a class that store the state of individual objects (non-static) or the class as a whole (static).

**Technical Definition**
Non-static data members are part of each object; each object has its own copy. They are initialized by constructors and destroyed by destructors. Static data members are shared by all objects of the class and have static storage duration. Data members can have access specifiers, default member initializers (since C++11), and `mutable` or `const` qualifiers.

**Beginner-Friendly Explanation**
Data members are the **variables inside an object** — the object's state. Each object has its own set of non-static data members. Static data members are shared across all objects.

### Purposes

- To store the state of objects.
- To define the data part of the class's abstraction.
- To support encapsulation through access control.
- To provide class-level shared state (static members).

### Syntax Rules and Structure

**General Syntax**
```cpp
class X {
    int non_static_;                     // non-static data member
    static int static_;                  // static data member
    mutable int cache_;                  // mutable member
    const int constant_ = 42;            // const member with initializer
};
```

**Component Breakdown**

| Form | Meaning |
|------|---------|
| `int x;` | Non-static data member. |
| `static int x;` | Static data member (shared). |
| `mutable int x;` | Modifiable in `const` member functions. |
| `const int x = 42;` | Constant member (must be initialized). |

**Syntax Rules**
- Non-static data members can have default member initializers (C++11).
- Static data members must be defined outside the class (or `inline` since C++17).
- `const` and reference members must be initialized in the constructor's initializer list.
- `mutable` members can be modified in `const` member functions.

**Constraints and Limitations**
- Non-static data members cannot be `auto` (before C++14 for static).
- Static data members cannot be `mutable`.
- Reference members cannot be default-initialized; they must be bound in the constructor.

### Annotated Code Examples

**Example 1: Data members with initializers**

```cpp
#include <iostream>

class Config {
public:
    int width = 800;             // default member initializer
    int height = 600;
    static inline int version = 2;   // inline static member (C++17)
};

int main() {
    Config c;
    std::cout << c.width << 'x' << c.height << " v" << Config::version << '\n';
}
```

**Expected Output**
```
800x600 v2
```

**Why this output?**  
`width` and `height` use default member initializers; `version` is an inline static member shared by all instances.

### Real-World Cases

- **Object state** – `Point::x`, `Point::y`.
- **Configuration** – `Config::width`, `Config::height`.
- **Shared counters** – `static int instance_count;`.
- **Caches** – `mutable std::string cached_string;`.

---

## 8. Member Functions

### Definitions

**Core Definition**
Member functions are functions declared inside a class that operate on objects of that class (non-static) or on the class itself (static).

**Technical Definition**
Non-static member functions have an implicit `this` pointer to the object they are called on; they can access all members of the object. Static member functions do not have a `this` pointer and can only access static members (unless given an object). Member functions can be defined inside the class body (implicitly `inline`) or outside using the scope resolution operator (`ClassName::function`). They can be `const`, `volatile`, `&&`-qualified, `virtual`, `static`, or `friend`.

**Beginner-Friendly Explanation**
Member functions are the **actions an object can perform**. They are functions that belong to the class and can access the object's data. `void print() const` is a member function that prints the object's state.

### Purposes

- To define the behavior of objects.
- To provide controlled access to data members (encapsulation).
- To implement operations, queries, and mutations.
- To support operator overloading and polymorphism.

### Syntax Rules and Structure

**General Syntax**
```cpp
class X {
public:
    void f();                    // declaration
    void g() const;              // const member function
    static void h();             // static member function
};

void X::f() { /* ... */ }        // out-of-class definition
void X::g() const { /* ... */ }
void X::h() { /* ... */ }
```

**Component Breakdown**

| Form | Meaning |
|------|---------|
| `void f();` | Non-static member function. |
| `void g() const;` | `const` member function (cannot modify non-`mutable` members). |
| `static void h();` | Static member function (no `this`). |
| `void X::f() { ... }` | Out-of-class definition. |

**Syntax Rules**
- Member functions defined in the class body are implicitly `inline`.
- `const` member functions cannot modify non-`mutable` data members.
- Static member functions cannot access non-static members directly.
- Member functions can be overloaded.

**Constraints and Limitations**
- `const` member functions can be called on `const` objects; non-`const` cannot.
- Static member functions cannot be `const` or `virtual`.
- Constructors and destructors are special member functions with their own rules.

### Annotated Code Examples

**Example 1: Non-static, const, and static member functions**

```cpp
#include <iostream>

class Counter {
    int count_ = 0;
    static int total_;
public:
    void increment() { ++count_; ++total_; }   // non-static
    int get() const { return count_; }          // const
    static int total() { return total_; }       // static
};

int Counter::total_ = 0;

int main() {
    Counter a, b;
    a.increment();
    a.increment();
    b.increment();
    std::cout << a.get() << ' ' << b.get() << ' ' << Counter::total() << '\n';
}
```

**Expected Output**
```
2 1 3
```

**Why this output?**  
`increment` modifies each object's `count_` and the shared `total_`. `get` is `const` and reads `count_`. `total` is static and returns the shared total.

### Real-World Cases

- **Getters/setters** – `int value() const`, `void set_value(int)`.
- **Operations** – `void push_back(T)`, `T pop()`.
- **Static factories** – `static T create()`.
- **Operator overloads** – `T& operator=(const T&)`.

---

## 9. Access Specifiers

### Definitions

**Core Definition**
Access specifiers (`public`, `private`, `protected`) control which parts of a class are accessible from which contexts.

**Technical Definition**
Access specifiers determine the accessibility of class members:
- **`public`** : accessible from any context.
- **`private`** : accessible only to members and friends of the class.
- **`protected`** : accessible to members and friends of the class and to members and friends of derived classes.

The default access for `class` is `private`; for `struct` and `union` it is `public`. Access specifiers can appear multiple times and in any order.

**Beginner-Friendly Explanation**
Access specifiers are **doors with different keys**:
- **`public`** — anyone can enter.
- **`private`** : only the class itself can enter.
- **`protected`** : the class and its children can enter.

They let you hide implementation details and expose only the interface you want.

### Purposes

- To encapsulate implementation details.
- To protect invariants by restricting modification.
- To define the public interface of a class.
- To support inheritance through `protected` access.
- To enable friend declarations when needed.

### Syntax Rules and Structure

**General Syntax**
```cpp
class X {
public:
    // accessible everywhere
private:
    // accessible only within X (and friends)
protected:
    // accessible within X, derived classes, and friends
};
```

**Component Breakdown**

| Specifier | Access |
|-----------|--------|
| `public` | Any code. |
| `private` | Members and friends of the class. |
| `protected` | Members and friends of the class and derived classes. |

**Syntax Rules**
- Access specifiers are labels followed by a colon (`public:`).
- They apply to all subsequent members until the next specifier.
- Default access: `class` → `private`; `struct` → `public`.
- Friends can access `private` and `protected` members.

**Constraints and Limitations**
- `private` members are not accessible to derived classes (use `protected` for that).
- Access control is checked at compile time; it does not affect runtime.
- `friend` declarations can bypass access control but should be used sparingly.

### Annotated Code Examples

**Example 1: Access specifiers**

```cpp
#include <iostream>

class Account {
private:
    double balance_ = 0;          // private: hidden
protected:
    int account_id_ = 1;          // protected: for derived
public:
    void deposit(double amount) { balance_ += amount; }   // public
    double balance() const { return balance_; }           // public
};

class SavingsAccount : public Account {
public:
    int id() const { return account_id_; }   // can access protected
    // double b() const { return balance_; } // error: private
};

int main() {
    Account a;
    a.deposit(100);
    std::cout << a.balance() << '\n';
    SavingsAccount s;
    std::cout << s.id() << '\n';
}
```

**Expected Output**
```
100
1
```

**Why this output?**  
`balance_` is private and accessed only through public methods. `account_id_` is protected and accessible in the derived class.

### Real-World Cases

- **Encapsulation** – private data with public getters/setters.
- **Inheritance** – protected members for derived classes.
- **API design** – public interface, private implementation.
- **Friend functions** – selective access for operators or helpers.

---

## 10. `this` Pointer

### Definitions

**Core Definition**
`this` is an implicit pointer available in non-static member functions that points to the object on which the member function was called.

**Technical Definition**
In a non-static member function, the keyword `this` is a prvalue pointer to the object for which the function is invoked. Its type is `X*` (or `const X*` in a `const` member function, `volatile X*` in a `volatile` member function, etc.). It can be used to access members, return the object, or pass the object to other functions. It cannot be modified (it is not an lvalue).

**Beginner-Friendly Explanation**
`this` is a **pointer to the current object**. Inside a member function, `this->member` refers to the member of the object that called the function. It's useful for chaining (`return *this;`) and for disambiguating parameters from members.

### Purposes

- To access members of the current object explicitly.
- To return the current object for chaining.
- To pass the current object to other functions.
- To disambiguate parameter names from member names.
- To check for self-assignment.

### Syntax Rules and Structure

**General Syntax**
```cpp
void X::f() {
    this->member = value;   // explicit member access
    return *this;           // return current object (by reference)
}
```

**Component Breakdown**

| Form | Meaning |
|------|---------|
| `this` | Pointer to the current object. |
| `this->member` | Access member through `this`. |
| `*this` | The current object (lvalue). |

**Syntax Rules**
- `this` is available only in non-static member functions.
- Its type is `X*` (or cv-qualified in cv-qualified member functions).
- `this` is a prvalue; it cannot be assigned to.
- In a `const` member function, `this` has type `const X*`.

**Constraints and Limitations**
- `this` cannot be used in static member functions.
- `delete this;` is allowed only in specific patterns (e.g., self-deleting objects) and is dangerous.
- Returning `*this` by reference enables method chaining.

### Annotated Code Examples

**Example 1: Using `this` for chaining and disambiguation**

```cpp
#include <iostream>

class Builder {
    int value_ = 0;
public:
    Builder& set(int value) {
        this->value_ = value;   // disambiguate
        return *this;           // enable chaining
    }
    int get() const { return value_; }
};

int main() {
    Builder b;
    b.set(10).set(20).set(30);
    std::cout << b.get() << '\n';
}
```

**Expected Output**
```
30
```

**Why this output?**  
`set` returns `*this`, allowing chained calls. The final `set(30)` sets `value_` to `30`.

### Real-World Cases

- **Fluent interfaces** – `builder.set_x(1).set_y(2)`.
- **Assignment operators** – `return *this;`.
- **Self-assignment checks** – `if (this == &other)`.
- **Passing the object** – `register(this);`.

---

## 11. Constructor

### Definitions

**Core Definition**
A constructor is a special member function that **initializes an object** when it is created, establishing the class invariant.

**Technical Definition**
A constructor has the same name as the class, no return type, and is invoked automatically when an object is created. Constructors can be overloaded, take parameters, use member initializer lists, delegate to other constructors, and be `explicit`, `constexpr`, or `consteval`. The default constructor, copy constructor, and move constructor are special constructors with specific roles. If no constructor is declared, the compiler generates a default constructor (unless other constructors are declared).

**Beginner-Friendly Explanation**
A constructor is the **setup function** that runs when an object is created. It initializes the object's data members and ensures the object is ready to use. If you don't write one, the compiler provides a default constructor.

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
    X();                             // default constructor
    X(int a);                        // parameterized constructor
    X(const X& other);               // copy constructor
    X(X&& other) noexcept;           // move constructor
    X(int a, int b) : x_(a), y_(b) {} // member initializer list
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
- Constructors can be `explicit` to prevent implicit conversions.
- Delegating constructors can call another constructor of the same class.

**Constraints and Limitations**
- A constructor that throws means the object is not fully constructed.
- Virtual constructors are not allowed (use factory functions or virtual `clone`).
- A `const` or reference member must be initialized in the initializer list.

### Annotated Code Examples

**Example 1: Constructor with member initializer list**

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
The constructor initializes `x_` and `y_` via the member initializer list, then prints their values.

### Real-World Cases

- **Resource acquisition** – open files, allocate memory.
- **Validation** – throw if parameters are invalid.
- **Default initialization** – set sensible defaults.
- **Conversion** – `explicit` constructors for conversions.

---

## 12. Destructor

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

## 13. Copy Constructor

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
- The parameter is typically `const X&` (allows binding to temporaries).
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

## 14. Move Constructor

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

## 15. Copy Assignment

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
    X& operator=(const X& other);   // copy assignment
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

## 16. Move Assignment

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
    X& operator=(X&& other) noexcept;   // move assignment
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

## 17. Default Constructor

### Definitions

**Core Definition**
A default constructor is a constructor that can be called with **no arguments**, initializing an object with default values.

**Technical Definition**
A default constructor is a constructor that can be invoked with no arguments: `X()`, `X(int a = 0)`, or `X(std::initializer_list<int> = {})`. The compiler generates a default constructor if no user-declared constructors exist. A defaulted default constructor is `constexpr` if possible. A default constructor is deleted if the class has a user-declared destructor, copy/move operations, or reference members without initializers (in some cases).

**Beginner-Friendly Explanation**
A default constructor is the **“no-arguments” constructor**. It creates an object with default values. If you don't write any constructor, the compiler provides one (unless other rules apply).

### Purposes

- To create objects without arguments.
- To support arrays and containers of objects.
- To provide default initialization.
- To satisfy the requirements of generic code.

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
- The default constructor is deleted if a member has no default constructor (e.g., a reference).
- User-declared constructors suppress the implicit default constructor.

### Annotated Code Examples

**Example 1: Default constructor**

```cpp
#include <iostream>

class Point {
    int x_, y_;
public:
    Point() : x_(0), y_(0) {}   // default constructor
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
The default constructor initializes `x_` and `y_` to `0`.

### Real-World Cases

- **Containers** – `std::vector<Point> v(10);` requires a default constructor.
- **Arrays** – `Point arr[5];` requires a default constructor.
- **Generic code** – many templates assume default constructibility.

---

## 18. Deleted Functions

### Definitions

**Core Definition**
A deleted function is a function declared with `= delete`, which **prevents its use** and causes a compile-time error if called.

**Technical Definition**
The `= delete` specifier declares a function as deleted: any use of the function is ill-formed. Deleted functions participate in overload resolution (unlike omitted functions), allowing the programmer to explicitly forbid certain operations (e.g., copying a non-copyable type, calling a function with certain argument types). Deleted functions are implicitly `inline`.

**Beginner-Friendly Explanation**
A deleted function is a **“do not use”** sign. You declare a function with `= delete` to make the compiler reject any call to it. It's useful for banning copying, preventing unwanted implicit conversions, and removing overloads.

### Purposes

- To prevent copying or moving of a class.
- To prevent unwanted implicit conversions.
- To remove specific overloads.
- To document that an operation is not supported.

### Syntax Rules and Structure

**General Syntax**
```cpp
class X {
public:
    X(const X&) = delete;             // no copy construction
    X& operator=(const X&) = delete;  // no copy assignment
};
```

**Component Breakdown**

| Form | Meaning |
|------|---------|
| `= delete` | The function cannot be used. |
| Deleted copy ctor | Class is non-copyable. |
| Deleted overload | Specific overload is forbidden. |

**Syntax Rules**
- `= delete` can be applied to any function, including special members and free functions.
- Deleted functions participate in overload resolution; their selection is an error.
- Deleted functions are implicitly `inline`.
- The first declaration of a function must contain `= delete`.

**Constraints and Limitations**
- Deleted functions cannot be used, even indirectly.
- A deleted function cannot be defined elsewhere.

### Annotated Code Examples

**Example 1: Non-copyable class**

```cpp
#include <iostream>

class NonCopyable {
public:
    NonCopyable() = default;
    NonCopyable(const NonCopyable&) = delete;
    NonCopyable& operator=(const NonCopyable&) = delete;
};

int main() {
    NonCopyable a;
    // NonCopyable b = a;   // error: copy constructor is deleted
    std::cout << "ok\n";
}
```

**Expected Output**
```
ok
```

**Why this output?**  
The copy constructor is deleted, so copying `a` would be a compile-time error. The program only creates one object.

### Real-World Cases

- **Non-copyable types** – `std::unique_ptr`, `std::thread`, `std::mutex`.
- **Preventing slicing** – deleted copy operations.
- **Banning implicit conversions** – deleted overloads.

---

## 19. Defaulted Functions

### Definitions

**Core Definition**
A defaulted function is a function declared with `= default`, which **explicitly requests the compiler-generated implementation**.

**Technical Definition**
The `= default` specifier declares a function as explicitly defaulted: the compiler generates the default implementation (e.g., member-wise copy or move). Defaulted functions can be defined inside the class body or outside. They are used to regain compiler-generated special members that would otherwise be suppressed, and to document that the default behavior is intended.

**Beginner-Friendly Explanation**
A defaulted function is a **“let the compiler do it”** declaration. You write `= default` to tell the compiler to generate the standard implementation (e.g., copy all members). It's useful when you want the default behavior but need to declare the function for some reason.

### Purposes

- To explicitly request the compiler-generated implementation.
- To regain special members suppressed by other declarations.
- To document that the default behavior is intended.
- To combine with `virtual`, `constexpr`, or `noexcept`.

### Syntax Rules and Structure

**General Syntax**
```cpp
class X {
public:
    X() = default;
    X(const X&) = default;
    X& operator=(const X&) = default;
    ~X() = default;
};
```

**Component Breakdown**

| Form | Meaning |
|------|---------|
| `= default` | Compiler generates the implementation. |
| Defaulted copy ctor | Member-wise copy. |
| Defaulted destructor | Default cleanup. |

**Syntax Rules**
- `= default` can be used for special members and comparison operators (C++20).
- The first declaration must contain `= default`.
- Defaulted functions can be defined inside or outside the class.
- A defaulted function that cannot be generated (e.g., due to a non-copyable member) is deleted.

**Constraints and Limitations**
- A defaulted function that cannot be generated is implicitly deleted.
- Defaulted functions are `noexcept` if the generated implementation is `noexcept`.

### Annotated Code Examples

**Example 1: Defaulted special members**

```cpp
#include <iostream>

class Point {
    int x_, y_;
public:
    Point(int x, int y) : x_(x), y_(y) {}
    Point(const Point&) = default;              // member-wise copy
    Point& operator=(const Point&) = default;   // member-wise assign
    ~Point() = default;
    int x() const { return x_; }
    int y() const { return y_; }
};

int main() {
    Point a(1, 2);
    Point b = a;   // uses defaulted copy constructor
    std::cout << b.x() << ',' << b.y() << '\n';
}
```

**Expected Output**
```
1,2
```

**Why this output?**  
`= default` tells the compiler to generate the copy constructor, which copies `x_` and `y_`.

### Real-World Cases

- **Rule of Five** – explicitly defaulting copy/move operations.
- **Virtual destructors** – `virtual ~X() = default;`.
- **Documenting intent** – making default behavior explicit.

---

## 20. Class Best Practices

### Definitions

**Core Definition**
Class best practices are guidelines for designing classes that are correct, maintainable, efficient, and safe, based on the C++ Core Guidelines.

**Technical Definition**
The C++ Core Guidelines provide class design rules: **C.1** (organize related data into structures), **C.2** (use `class` if there is an invariant), **C.3** (represent the distinction between interface and implementation), **C.4** (make a function a member only if it needs direct access), **C.5** (place helper functions in the same namespace), **C.7** (don't define a class and its member function definitions in separate files unless necessary), **C.8** (use `class` rather than `struct` if there is an invariant), **C.9** (minimize the exposure of members), **C.20** (avoid defining default operations), **C.21** (define or delete all copy/move/destructor operations), **C.30–C.35** (destructor rules), **C.40** (define a constructor if a class has an invariant), **C.41** (a constructor should create a fully initialized object), **C.45** (don't define a default constructor that only initializes data members), **C.46** (by default, declare single-argument constructors `explicit`), **C.60–C.66** (copy/move assignment), and **C.80–C.90** (virtual functions and overrides).

**Beginner-Friendly Explanation**
Best practices for classes are simple: **keep classes focused**, **use `class` when you need invariants**, **make members private by default**, **use `explicit` for single-argument constructors**, **follow the Rule of Zero/Five**, **make destructors virtual when needed**, and **document your interface**. These habits make classes safe and easy to use.

### Key Guidelines (Summary Table)

| Guideline | Summary | Recommendation |
|-----------|---------|----------------|
| **C.1** | Organize related data into structures. | Use classes for related data. |
| **C.2** | Use `class` if there is an invariant. | `struct` for plain data, `class` for invariants. |
| **C.4** | Make a function a member only if it needs direct access. | Prefer non-member functions where possible. |
| **C.8** | Use `class` rather than `struct` if there is an invariant. | Encapsulate invariants. |
| **C.9** | Minimize the exposure of members. | Private data, public interface. |
| **C.20** | Avoid defining default operations. | Rule of Zero. |
| **C.21** | Define or delete all copy/move/destructor operations. | Rule of Five. |
| **C.40** | Define a constructor if a class has an invariant. | Enforce invariants at construction. |
| **C.41** | A constructor should create a fully initialized object. | No half-constructed objects. |
| **C.46** | Declare single-argument constructors `explicit` by default. | Prevent implicit conversions. |
| **C.35** | Base class destructor: public virtual or protected non-virtual. | Correct polymorphic deletion. |
| **C.60/C.63** | Copy/move assignment: non-virtual, take by reference, return by reference. | Standard signatures. |

### Annotated Code Examples

**Example 1: Well-designed class**

```cpp
#include <iostream>
#include <string>

class Person {
private:
    std::string name_;   // private data
    int age_;            // private data

public:
    explicit Person(std::string name, int age)   // explicit single-arg? (two args here)
        : name_(std::move(name)), age_(age) {
        if (age_ < 0) throw std::invalid_argument("age must be non-negative");
    }

    const std::string& name() const noexcept { return name_; }   // const getter
    int age() const noexcept { return age_; }                    // const getter

    void set_age(int age) {                                      // mutator
        if (age < 0) throw std::invalid_argument("age must be non-negative");
        age_ = age;
    }
};

int main() {
    Person p("Alice", 30);
    std::cout << p.name() << " is " << p.age() << '\n';
    p.set_age(31);
    std::cout << p.age() << '\n';
}
```

**Expected Output**
```
Alice is 30
31
```

**Why this is good design?**  
- Private data with public interface (C.9).
- Constructor enforces the invariant (C.41).
- Getters are `const` and `noexcept` (C.2, C.4).
- Mutator validates its input.
- No unnecessary special members (Rule of Zero).

### Real-World Cases

- **Standard library** – `std::vector`, `std::string`, `std::map` follow these rules.
- **Code review** – Core Guidelines C.1–C.90 are common checkpoints.
- **Static analysis** – clang-tidy enforces many class design rules.

---

## References

- Classes (cppreference) – https://en.cppreference.com/w/cpp/language/classes
- Class declaration (cppreference) – https://en.cppreference.com/w/cpp/language/class
- Data members (cppreference) – https://en.cppreference.com/w/cpp/language/data_members
- Member functions (cppreference) – https://en.cppreference.com/w/cpp/language/member_functions
- Access specifiers (cppreference) – https://en.cppreference.com/w/cpp/language/access
- `this` pointer (cppreference) – https://en.cppreference.com/w/cpp/language/this
- Constructors (cppreference) – https://en.cppreference.com/w/cpp/language/constructor
- Destructors (cppreference) – https://en.cppreference.com/w/cpp/language/destructor
- Copy constructors (cppreference) – https://en.cppreference.com/w/cpp/language/copy_constructor
- Move constructors (cppreference) – https://en.cppreference.com/w/cpp/language/move_constructor
- Copy assignment operator (cppreference) – https://en.cppreference.com/w/cpp/language/copy_assignment
- Move assignment operator (cppreference) – https://en.cppreference.com/w/cpp/language/move_assignment
- Default constructor (cppreference) – https://en.cppreference.com/w/cpp/language/default_constructor
- Deleted functions (cppreference) – https://en.cppreference.com/w/cpp/language/function#Deleted_functions
- Defaulted functions (cppreference) – https://en.cppreference.com/w/cpp/language/function#Defaulted_functions
- C++ Core Guidelines — C.1: Organize related data into structures (`struct`s or `class`es) – https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rc-org
- C++ Core Guidelines — C.2: Use `class` if the class has an invariant; use `struct` if the data members can vary independently – https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rc-struct
- C++ Core Guidelines — C.9: Minimize the exposure of members – https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rc-private
- C++ Core Guidelines — C.20: If you can avoid defining default operations, do – https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rc-zero
- C++ Core Guidelines — C.21: If you define or `=delete` any copy, move, or destructor function, define or `=delete` them all – https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rc-five
- C++ Core Guidelines — C.35: A base class destructor should be either public and virtual, or protected and non-virtual – https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rc-dtor-virtual
- C++ Core Guidelines — C.40: Define a constructor if a class has an invariant – https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rc-ctor
- C++ Core Guidelines — C.41: A constructor should create a fully initialized object – https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rc-complete
- C++ Core Guidelines — C.46: By default, declare single-argument constructors `explicit` – https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rc-explicit
- Microsoft Learn — Classes and structs (C++) – https://learn.microsoft.com/en-us/cpp/cpp/classes-and-structs-cpp
- Microsoft Learn — Special member functions – https://learn.microsoft.com/en-us/cpp/cpp/special-member-functions

---

This cheat sheet provides a complete, structured reference for C++ classes and objects, covering class declarations and definitions, members, access specifiers, `this`, constructors, destructors, copy/move operations, default/deleted/defaulted functions, and best practices. Each section follows a uniform template with definitions, purposes, syntax rules, annotated examples, expected outputs, and authoritative references.