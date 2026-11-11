# Operator Overloading in Classes — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
Operator overloading is a C++ feature that allows programmers to define the behavior of standard operators (such as `+`, `==`, `<<`, `[]`, `()`) for user-defined types, making class objects behave like built-in types.

**Technical Definition**
Operator overloading is implemented by defining functions with special names (`operator+`, `operator==`, `operator<<`, etc.). These functions can be **member functions**, **non-member functions**, or **friend functions**. The operator's meaning is resolved through overload resolution, which considers both the operator's built-in meanings and user-defined overloads. Certain operators cannot be overloaded (`::`, `.`, `.*`, `?:`, `sizeof`, `typeid`, `alignof`, `noexcept`, `static_cast`, `dynamic_cast`, `reinterpret_cast`, `const_cast`, `co_await`); others must be members (`=`, `[]`, `()`, `->`, `->*`, conversion functions). The C++ Core Guidelines provide extensive rules on operator overloading: **C.160** (define operators primarily to mimic conventional usage), **C.161** (use nonmember functions for symmetric operators), **C.162** (overload operations that are roughly equivalent), **C.163** (overload only for operations that are roughly equivalent), **C.164** (avoid implicit conversions), **C.165** (use `==` when appropriate), **C.166** (overload `==` to compare equivalent values), **C.167** (overload `==` and `!=` consistently), **C.168** (define `==` and `!=` together), **C.170** (use `operator<<` for stream output), and **C.171** (avoid `operator` abuse).

**Beginner-Friendly Explanation**
Operator overloading lets you use familiar symbols like `+`, `==`, and `<<` with your own classes. For example, you can define `operator+` for a `Complex` class so that `a + b` adds two complex numbers, or `operator<<` for a `Point` class so that `std::cout << p` prints it. It makes your types feel natural and readable. But with great power comes responsibility: only overload operators when the meaning is **obvious and intuitive**, and follow conventional patterns so other programmers aren't surprised.

### Key Characteristics

- **Natural syntax**: Overloaded operators let user-defined types use familiar notation.
- **Multiple forms**: Member, non-member, and friend functions can implement operators.
- **Restrictions**: Some operators cannot be overloaded; some must be members.
- **Conventional semantics**: Operators should mimic their built-in meanings.
- **Symmetry**: Non-member functions are often better for symmetric binary operators.
- **Implicit conversions**: Member operators may cause asymmetric conversions; non-members can avoid this.
- **Consistency**: Related operators (`==`/`!=`, `+`/`+=`, `<`/`>`) should be consistent.

### Prerequisites

- Understanding of classes, member functions, and access control.
- Familiarity with function overloading and overload resolution.
- Knowledge of references, const-correctness, and value semantics.
- Basic awareness of friend functions and templates.

### Related Programming Areas

- **Classes and objects** – operators are functions that operate on objects.
- **Overload resolution** – the compiler selects the best operator function.
- **Templates** – operators can be templated (e.g., `operator<<` for any type).
- **Standard library** – stream operators, comparison operators, and arithmetic operators are heavily overloaded.
- **C++20 spaceship operator** – `<=>` provides a unified comparison mechanism.
- **Design patterns** – operators are used in many library interfaces (iterators, smart pointers).

### Core Concepts / Features

1. Member Operator Overloading
2. Non-Member Operator Overloading
3. Friend Operator Overloading
4. Comparison Operators
5. Arithmetic Operators
6. Assignment Operators
7. Stream Operators
8. Subscript Operators
9. Function Call Operators
10. Conversion Operators
11. Increment/Decrement Operators
12. Operator Overloading Best Practices

---

## 1. Member Operator Overloading

### Definitions

**Core Definition**
Member operator overloading defines an operator as a **non-static member function** of a class, with the left-hand operand as the implicit `*this` object.

**Technical Definition**
A member operator function is declared inside the class as `return_type operator@(parameters)`. The left-hand operand is the object on which the operator is called (the `*this` object); the right-hand operand(s) are the function parameters. Certain operators **must** be members: `=`, `[]`, `()`, `->`, `->*`, and conversion functions. Member operators are often used for mutating operators (`+=`, `-=`, `++`) and operators that require direct access to the object's state.

**Beginner-Friendly Explanation**
A member operator is an operator defined **inside the class**. The object on the left of the operator is the object itself, and the object on the right is passed as a parameter. For example, `a += b` calls `a.operator+=(b)`.

### Purposes

- To implement operators that modify the object (`+=`, `++`, `=`, `[]`).
- To provide direct access to private members.
- To satisfy the requirement that certain operators be members.
- To implement operators that logically belong to the class.

### Syntax Rules and Structure

**General Syntax**
```cpp
class X {
public:
    X& operator+=(const X& rhs);       // binary member
    X& operator++();                    // prefix increment
    X operator++(int);                  // postfix increment
    int& operator[](std::size_t i);     // subscript (must be member)
    int operator()(int arg);            // function call (must be member)
};
```

**Component Breakdown**

| Form | Meaning |
|------|---------|
| `operator+=(const X&)` | Binary member operator (`a += b`). |
| `operator++()` | Prefix increment. |
| `operator++(int)` | Postfix increment (dummy `int` parameter). |
| `operator[]` | Subscript (must be member). |
| `operator()` | Function call (must be member). |

**Syntax Rules**
- The left-hand operand is the implicit `*this` object.
- The right-hand operand is the parameter.
- Unary member operators take no parameters (except postfix `++`/`--` with a dummy `int`).
- Certain operators must be members: `=`, `[]`, `()`, `->`, `->*`, conversion functions.
- Member operators are often declared `const` when they don't modify the object.

**Constraints and Limitations**
- Member operators may cause asymmetric implicit conversions (the left operand must be an object of the class).
- Binary symmetric operators (e.g., `==`, `+`) are often better as non-members.
- Member operators cannot be used if the left operand is not of the class type.

### Annotated Code Examples

**Example 1: Member arithmetic and assignment operators**

```cpp
#include <iostream>

class Complex {
    double real_, imag_;
public:
    Complex(double r = 0, double i = 0) : real_(r), imag_(i) {}

    Complex& operator+=(const Complex& rhs) {   // member +=
        real_ += rhs.real_;
        imag_ += rhs.imag_;
        return *this;
    }

    double real() const { return real_; }
    double imag() const { return imag_; }
};

int main() {
    Complex a(1, 2), b(3, 4);
    a += b;
    std::cout << a.real() << '+' << a.imag() << "i\n";
}
```

**Expected Output**
```
4+6i
```

**Why this output?**  
`a += b` calls `a.operator+=(b)`, which adds `b`'s components to `a` and returns `a`.

### Real-World Cases

- **Compound assignment** – `+=`, `-=`, `*=`, `/=`.
- **Increment/decrement** – `++`, `--`.
- **Subscript** – `operator[]` (must be member).
- **Function call** – `operator()` (functors).

---

## 2. Non-Member Operator Overloading

### Definitions

**Core Definition**
Non-member operator overloading defines an operator as a **free function** (not a member), with all operands passed as parameters.

**Technical Definition**
A non-member operator function is declared at namespace scope as `return_type operator@(parameters)`. Both operands are parameters; the function may be a friend of the class to access private members. Non-member operators are preferred for **symmetric binary operators** (`==`, `!=`, `+`, `-`, `*`, `/`, `<`, `>`, etc.) because they allow implicit conversions on both operands. The C++ Core Guidelines recommend non-member functions for symmetric operators (C.161).

**Beginner-Friendly Explanation**
A non-member operator is an operator defined **outside the class** as a free function. Both operands are passed as parameters, so implicit conversions can apply to both sides. This makes expressions like `2 + complex` work as naturally as `complex + 2`.

### Purposes

- To implement symmetric binary operators.
- To allow implicit conversions on both operands.
- To support operators where the left operand is not of the class type (e.g., `std::ostream << object`).
- To keep the class interface minimal.

### Syntax Rules and Structure

**General Syntax**
```cpp
class X {
    int value_;
public:
    int value() const { return value_; }
};

X operator+(const X& lhs, const X& rhs);   // non-member
bool operator==(const X& lhs, const X& rhs);
```

**Component Breakdown**

| Form | Meaning |
|------|---------|
| `operator+(const X&, const X&)` | Binary non-member operator. |
| `operator==(const X&, const X&)` | Comparison operator. |
| `friend` | Optional, for access to private members. |

**Syntax Rules**
- Both operands are parameters.
- The function is declared at namespace scope.
- It may be a friend to access private members.
- It can be a template (e.g., `operator<<` for streams).

**Constraints and Limitations**
- Non-member operators cannot be used for operators that must be members (`=`, `[]`, `()`, `->`).
- They must be found via argument-dependent lookup (ADL) or be in scope.
- They cannot access private members without friendship or public accessors.

### Annotated Code Examples

**Example 1: Non-member symmetric operator**

```cpp
#include <iostream>

class Complex {
    double real_, imag_;
public:
    Complex(double r = 0, double i = 0) : real_(r), imag_(i) {}
    double real() const { return real_; }
    double imag() const { return imag_; }
};

Complex operator+(const Complex& lhs, const Complex& rhs) {
    return Complex(lhs.real() + rhs.real(), lhs.imag() + rhs.imag());
}

int main() {
    Complex a(1, 2), b(3, 4);
    Complex c = a + b;
    std::cout << c.real() << '+' << c.imag() << "i\n";
}
```

**Expected Output**
```
4+6i
```

**Why this output?**  
`a + b` calls `operator+(a, b)`, which returns a new `Complex` with the summed components.

### Real-World Cases

- **Symmetric arithmetic** – `operator+(const T&, const T&)`.
- **Comparison** – `operator==`, `operator!=`, `operator<`.
- **Stream insertion** – `operator<<(std::ostream&, const T&)`.
- **Iterator operators** – `operator+(iterator, difference_type)`.

---

## 3. Friend Operator Overloading

### Definitions

**Core Definition**
Friend operator overloading defines an operator as a **non-member function declared as a friend** of the class, giving it access to private and protected members.

**Technical Definition**
A friend operator function is declared inside the class with the `friend` keyword but is not a member. It is a non-member function with access to the class's private and protected members. Friend operators are commonly used for stream operators (`operator<<`, `operator>>`) and symmetric binary operators that need private access. Friendship is granted by the class, not taken by the function.

**Beginner-Friendly Explanation**
A friend operator is an **outside function that the class trusts**. It's not a member, but the class gives it special permission to access private members. This is useful for operators like `<<` that need to read private data but must be non-members (so the left operand can be `std::ostream`).

### Purposes

- To give non-member operators access to private members.
- To implement stream operators (`<<`, `>>`).
- To implement symmetric binary operators without public accessors.
- To keep the class interface minimal while allowing operator access.

### Syntax Rules and Structure

**General Syntax**
```cpp
class X {
    int value_;
public:
    friend X operator+(const X& lhs, const X& rhs);   // friend declaration
    friend std::ostream& operator<<(std::ostream& os, const X& x);
};
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `friend` | Grants access to private members. |
| Non-member function | The operator is not a member. |
| Access | Can access private/protected members. |

**Syntax Rules**
- The `friend` declaration appears inside the class.
- The function is defined at namespace scope (or inline inside the class).
- Friendship is not transitive, inherited, or mutual.
- Friend functions should be declared in the class and defined outside.

**Constraints and Limitations**
- Friendship breaks encapsulation; use sparingly.
- Friend functions cannot be virtual.
- Friendship is not inherited by derived classes.

### Annotated Code Examples

**Example 1: Friend stream operator**

```cpp
#include <iostream>

class Point {
    int x_, y_;
public:
    Point(int x, int y) : x_(x), y_(y) {}
    friend std::ostream& operator<<(std::ostream& os, const Point& p);
};

std::ostream& operator<<(std::ostream& os, const Point& p) {
    return os << '(' << p.x_ << ", " << p.y_ << ')';
}

int main() {
    Point p(3, 4);
    std::cout << p << '\n';
}
```

**Expected Output**
```
(3, 4)
```

**Why this output?**  
`operator<<` is a friend, so it can access `p.x_` and `p.y_` to print the point.

### Real-World Cases

- **Stream operators** – `operator<<`, `operator>>`.
- **Symmetric arithmetic** – `operator+`, `operator-`.
- **Comparison** – `operator==`, `operator<`.
- **Debugging helpers** – friend functions for printing.

---

## 4. Comparison Operators

### Definitions

**Core Definition**
Comparison operators (`==`, `!=`, `<`, `<=`, `>`, `>=`, and since C++20 `<=>`) define how objects of a class are compared for equality and ordering.

**Technical Definition**
Comparison operators can be member or non-member functions. `operator==` and `operator!=` define equality; `<`, `<=`, `>`, `>=` define ordering. Since C++20, the three-way comparison operator `operator<=>` (spaceship operator) can generate all six comparison operators automatically. The C++ Core Guidelines state: define `==` and `!=` together (C.168), make `==` symmetric (C.161), and use `==` when appropriate (C.165). In C++20, `operator==` is automatically generated as `!=` when defined.

**Beginner-Friendly Explanation**
Comparison operators let you compare your objects with `==`, `<`, etc. For example, `a == b` checks if two `Complex` numbers are equal, and `a < b` checks ordering. Since C++20, defining `<=>` gives you all six comparisons for free.

### Purposes

- To define equality and ordering for user-defined types.
- To allow objects to be used in containers (`std::map`, `std::set`).
- To support algorithms (`std::sort`, `std::find`).
- To provide natural syntax for comparisons.

### Syntax Rules and Structure

**General Syntax**
```cpp
class X {
public:
    bool operator==(const X& rhs) const;   // member equality
    bool operator<(const X& rhs) const;    // member ordering
};

// Or non-member:
bool operator==(const X& lhs, const X& rhs);
bool operator<(const X& lhs, const X& rhs);

// C++20 spaceship:
auto operator<=>(const X& rhs) const = default;
```

**Component Breakdown**

| Operator | Meaning |
|----------|---------|
| `==`, `!=` | Equality. |
| `<`, `<=`, `>`, `>=` | Ordering. |
| `<=>` | Three-way comparison (C++20). |

**Syntax Rules**
- `==` and `!=` should be consistent (C.167, C.168).
- Comparison operators should be `const` and not modify the object.
- In C++20, `operator==` generates `!=`; `operator<=>` generates all six.
- `operator<=>` can be `= default` for member-wise comparison.
- Comparison operators should be symmetric (non-member for symmetry).

**Constraints and Limitations**
- `operator==` and `operator<=>` must be consistent.
- Ordering must be a strict weak ordering for `std::sort` and ordered containers.
- Floating-point comparisons require care (NaN, precision).

### Annotated Code Examples

**Example 1: Comparison operators (pre-C++20)**

```cpp
#include <iostream>

class Point {
    int x_, y_;
public:
    Point(int x, int y) : x_(x), y_(y) {}

    bool operator==(const Point& rhs) const {
        return x_ == rhs.x_ && y_ == rhs.y_;
    }
    bool operator!=(const Point& rhs) const {
        return !(*this == rhs);
    }
    bool operator<(const Point& rhs) const {
        return x_ < rhs.x_ || (x_ == rhs.x_ && y_ < rhs.y_);
    }
};

int main() {
    Point a(1, 2), b(1, 3);
    std::cout << (a == b) << ' ' << (a != b) << ' ' << (a < b) << '\n';
}
```

**Expected Output**
```
0 1 1
```

**Why this output?**  
`a == b` is false (`0`), `a != b` is true (`1`), and `a < b` is true (`1`) because `(1,2) < (1,3)`.

**Example 2: C++20 spaceship operator**

```cpp
#include <iostream>
#include <compare>

class Point {
    int x_, y_;
public:
    Point(int x, int y) : x_(x), y_(y) {}
    auto operator<=>(const Point&) const = default;   // generates all six
};

int main() {
    Point a(1, 2), b(1, 3);
    std::cout << (a == b) << ' ' << (a != b) << ' ' << (a < b) << '\n';
}
```

**Expected Output**
```
0 1 1
```

**Why this output?**  
`operator<=>` with `= default` generates all comparison operators via member-wise comparison.

### Real-World Cases

- **Containers** – `std::map` and `std::set` require `operator<` or `<=>`.
- **Algorithms** – `std::sort`, `std::find`.
- **Value types** – `Point`, `Complex`, `Date`.

---

## 5. Arithmetic Operators

### Definitions

**Core Definition**
Arithmetic operators (`+`, `-`, `*`, `/`, `%`) define arithmetic operations for user-defined types.

**Technical Definition**
Arithmetic operators are typically implemented as **non-member functions** for symmetry, with compound assignment operators (`+=`, `-=`, etc.) as members. The standard pattern is to implement the compound assignment as a member and the binary arithmetic operator as a non-member in terms of the compound assignment. This ensures consistency and avoids duplication. The C++ Core Guidelines state: define binary operators as non-members (C.161), and implement `a + b` as `a += b` then return `a` (copy).

**Beginner-Friendly Explanation**
Arithmetic operators let you use `+`, `-`, `*`, `/` with your objects. For example, `a + b` adds two vectors. The common pattern is to implement `+=` as a member, then define `+` as a non-member that copies the left operand, applies `+=`, and returns the result.

### Purposes

- To provide natural arithmetic syntax for numeric types.
- To support expression templates and chaining.
- To implement vector/matrix arithmetic.
- To maintain consistency between `+` and `+=`.

### Syntax Rules and Structure

**General Syntax**
```cpp
class X {
public:
    X& operator+=(const X& rhs);   // member compound assignment
};

X operator+(X lhs, const X& rhs) {   // non-member binary, by value
    lhs += rhs;
    return lhs;
}
```

**Component Breakdown**

| Operator | Typical Form |
|----------|--------------|
| `+=`, `-=`, `*=`, `/=` | Member, mutating. |
| `+`, `-`, `*`, `/` | Non-member, returns new object. |
| Unary `-`, `+` | Member or non-member. |

**Syntax Rules**
- Compound assignment should be members and return `*this`.
- Binary arithmetic should be non-members and take the left operand by value (or `const&`).
- Binary operators can be implemented in terms of compound assignment.
- Unary operators are often members.

**Constraints and Limitations**
- Arithmetic operators should not have surprising semantics.
- Overloading `&&`, `||`, and `,` is discouraged (they lose short-circuiting/sequencing).
- Arithmetic on user-defined types should follow the usual algebraic laws where possible.

### Annotated Code Examples

**Example 1: Arithmetic operators**

```cpp
#include <iostream>

class Vec2 {
    double x_, y_;
public:
    Vec2(double x = 0, double y = 0) : x_(x), y_(y) {}

    Vec2& operator+=(const Vec2& rhs) {
        x_ += rhs.x_;
        y_ += rhs.y_;
        return *this;
    }

    friend Vec2 operator+(Vec2 lhs, const Vec2& rhs) {
        lhs += rhs;
        return lhs;
    }

    void print() const { std::cout << '(' << x_ << ", " << y_ << ")\n"; }
};

int main() {
    Vec2 a(1, 2), b(3, 4);
    Vec2 c = a + b;
    c.print();
}
```

**Expected Output**
```
(4, 6)
```

**Why this output?**  
`a + b` calls `operator+`, which copies `a`, applies `+=` with `b`, and returns the result `(4, 6)`.

### Real-World Cases

- **Vector/matrix math** – `Vec2`, `Matrix`.
- **Complex numbers** – `Complex`.
- **Big integers** – `BigInt`.
- **Date arithmetic** – `Date + days`.

---

## 6. Assignment Operators

### Definitions

**Core Definition**
Assignment operators (`=`, `+=`, `-=`, `*=`, `/=`, `%=`, etc.) assign a new value to an existing object, replacing its previous value.

**Technical Definition**
The **copy assignment operator** (`operator=(const X&)`) and **move assignment operator** (`operator=(X&&)`) are special members with specific rules (Rule of Three/Five/Zero). Compound assignment operators (`+=`, etc.) are typically members that modify the object and return `*this`. The simple assignment operator must be a non-static member function and cannot be a friend. Assignment operators should handle self-assignment and provide the strong exception guarantee where possible.

**Beginner-Friendly Explanation**
Assignment operators change an object's value. `a = b` copies `b` into `a`; `a += b` adds `b` to `a`. The copy and move assignment operators are special because they're automatically generated or deleted by the compiler under certain rules.

### Purposes

- To assign new values to objects.
- To support compound assignment (`+=`, `-=`).
- To implement deep copies or move semantics.
- To maintain consistency with constructors.

### Syntax Rules and Structure

**General Syntax**
```cpp
class X {
public:
    X& operator=(const X& rhs);       // copy assignment (member)
    X& operator=(X&& rhs) noexcept;   // move assignment (member)
    X& operator+=(const X& rhs);      // compound assignment (member)
};
```

**Component Breakdown**

| Operator | Form | Notes |
|----------|------|-------|
| `=` | Member, non-virtual | Special member function. |
| `+=` etc. | Member | Returns `*this`. |
| Copy assignment | `operator=(const X&)` | Rule of Three/Five. |
| Move assignment | `operator=(X&&)` | `noexcept` recommended. |

**Syntax Rules**
- `operator=` must be a non-static member function.
- It cannot be a friend or a static function.
- It should return `*this` for chaining.
- It should handle self-assignment.
- Move assignment should be `noexcept`.

**Constraints and Limitations**
- The simple assignment operator cannot be a non-member.
- The compiler generates copy/move assignment unless suppressed.
- Assignment operators should be exception-safe.

### Annotated Code Examples

**Example 1: Compound assignment**

```cpp
#include <iostream>

class Counter {
    int value_ = 0;
public:
    Counter& operator+=(int n) {
        value_ += n;
        return *this;
    }
    int value() const { return value_; }
};

int main() {
    Counter c;
    c += 5;
    c += 10;
    std::cout << c.value() << '\n';
}
```

**Expected Output**
```
15
```

**Why this output?**  
`c += 5` and `c += 10` call `operator+=`, which adds to `value_` and returns `*this`.

### Real-World Cases

- **Numeric types** – `Complex`, `BigInt`, `Matrix`.
- **String builders** – `operator+=` for strings.
- **Container assignment** – `std::vector::operator=`.

---

## 7. Stream Operators

### Definitions

**Core Definition**
Stream operators (`<<` for output, `>>` for input) allow objects of user-defined types to be written to or read from standard library streams.

**Technical Definition**
`operator<<(std::ostream& os, const T& value)` writes a value to an output stream and returns `os` for chaining. `operator>>(std::istream& is, T& value)` reads a value and returns `is`. Stream operators are **non-member functions** (usually friends) because the left operand is a stream, not the user-defined type. They are the idiomatic way to support formatted I/O for user-defined types.

**Beginner-Friendly Explanation**
Stream operators let you write `std::cout << myObject` or `std::cin >> myObject`. They're non-member functions because the stream is on the left. They return the stream so you can chain: `std::cout << a << b << c`.

### Purposes

- To provide formatted output and input for user-defined types.
- To integrate with `std::cout`, `std::cin`, `std::ostringstream`, etc.
- To support chaining.
- To define textual representations of objects.

### Syntax Rules and Structure

**General Syntax**
```cpp
class X {
    int value_;
public:
    friend std::ostream& operator<<(std::ostream& os, const X& x);
    friend std::istream& operator>>(std::istream& is, X& x);
};

std::ostream& operator<<(std::ostream& os, const X& x) {
    return os << x.value_;
}
```

**Component Breakdown**

| Operator | Signature |
|----------|-----------|
| Output | `std::ostream& operator<<(std::ostream&, const T&)` |
| Input | `std::istream& operator>>(std::istream&, T&)` |

**Syntax Rules**
- The first parameter is the stream (by reference).
- The second parameter is the object (`const&` for output, `&` for input).
- The operator returns the stream reference for chaining.
- Usually non-member, often friend for private access.
- Include `<ostream>` or `<istream>`.

**Constraints and Limitations**
- The stream operator cannot be a member of the user-defined type (the left operand would be the object).
- Input operators should handle malformed input gracefully.
- Stream operators should not throw unless the stream is configured to.

### Annotated Code Examples

**Example 1: Stream output and input**

```cpp
#include <iostream>

class Point {
    int x_, y_;
public:
    Point(int x = 0, int y = 0) : x_(x), y_(y) {}

    friend std::ostream& operator<<(std::ostream& os, const Point& p) {
        return os << '(' << p.x_ << ", " << p.y_ << ')';
    }
    friend std::istream& operator>>(std::istream& is, Point& p) {
        return is >> p.x_ >> p.y_;
    }
};

int main() {
    Point p(3, 4);
    std::cout << p << '\n';
    Point q;
    std::cin >> q;
    std::cout << q << '\n';
}
```

**Expected Output** (for input `5 6`)
```
(3, 4)
(5, 6)
```

**Why this output?**  
`operator<<` prints `(3, 4)`. `operator>>` reads two integers into `q`, and the second `operator<<` prints `(5, 6)`.

### Real-World Cases

- **Debugging output** – printing objects.
- **Serialization** – writing to string streams.
- **Parsing** – reading from input streams.
- **Logging** – formatted log messages.

---

## 8. Subscript Operators

### Definitions

**Core Definition**
The subscript operator (`operator[]`) provides array-like element access for user-defined types.

**Technical Definition**
`operator[]` must be a **non-static member function**. It can be overloaded for `const` and non-`const` objects to provide mutable and read-only access. The parameter is typically an index type (`std::size_t`, or any type for associative containers). The return type is usually a reference to the element, allowing assignment. Since C++23, `operator[]` can take multiple parameters for multidimensional subscripting.

**Beginner-Friendly Explanation**
`operator[]` lets your class be indexed like an array: `obj[i]`. For example, a custom vector class can define `operator[]` so `v[2]` returns the third element.

### Purposes

- To provide array-like access to elements.
- To support both mutable and read-only access.
- To implement custom containers.
- To support associative lookup (e.g., `map[key]`).

### Syntax Rules and Structure

**General Syntax**
```cpp
class Container {
public:
    T& operator[](std::size_t index);              // mutable access
    const T& operator[](std::size_t index) const;  // read-only access
};
```

**Component Breakdown**

| Form | Meaning |
|------|---------|
| `operator[]` | Must be a non-static member. |
| Return reference | Allows assignment (`c[i] = x`). |
| `const` overload | Read-only access on `const` objects. |

**Syntax Rules**
- `operator[]` must be a non-static member.
- It is typically overloaded for `const` and non-`const` objects.
- It should return a reference to the element.
- The index type can be any type (often `std::size_t`).
- Since C++23, `operator[]` can take multiple parameters.

**Constraints and Limitations**
- Bounds checking is not automatic; `std::vector::at()` provides it.
- Returning by value prevents assignment; return by reference for mutable access.
- `operator[]` cannot be a non-member.

### Annotated Code Examples

**Example 1: Subscript operator**

```cpp
#include <iostream>
#include <stdexcept>

class IntArray {
    int data_[3] = {10, 20, 30};
public:
    int& operator[](std::size_t i) {
        if (i >= 3) throw std::out_of_range("index");
        return data_[i];
    }
    const int& operator[](std::size_t i) const {
        if (i >= 3) throw std::out_of_range("index");
        return data_[i];
    }
};

int main() {
    IntArray a;
    a[1] = 99;
    std::cout << a[1] << '\n';
}
```

**Expected Output**
```
99
```

**Why this output?**  
`a[1] = 99` calls the non-`const` `operator[]`, which returns a reference to `data_[1]`, allowing assignment.

### Real-World Cases

- **Custom containers** – `IntArray`, `Matrix`.
- **Associative containers** – `std::map::operator[]`.
- **Multidimensional access** – `Matrix[row][col]`.

---

## 9. Function Call Operators

### Definitions

**Core Definition**
The function call operator (`operator()`) allows an object to be **called like a function**, creating a *function object* (functor).

**Technical Definition**
`operator()` must be a **non-static member function**. It can take any number and type of parameters, be overloaded, and be templated. Objects with `operator()` are called **function objects** or **functors**. They can be used wherever a callable is expected (e.g., `std::function`, algorithms). Since C++11, lambdas are syntactic sugar for function objects.

**Beginner-Friendly Explanation**
`operator()` lets your object behave like a function. For example, a `Multiplier` class with `operator()(int)` can be called as `mult(5)`. Functors are useful because they can hold state.

### Purposes

- To create function objects (functors) that carry state.
- To pass callable objects to algorithms.
- To implement strategy patterns.
- To support lambdas (which are functors under the hood).

### Syntax Rules and Structure

**General Syntax**
```cpp
class Functor {
public:
    ReturnType operator()(Parameters) const;   // function call operator
};
```

**Component Breakdown**

| Form | Meaning |
|------|---------|
| `operator()` | Must be a non-static member. |
| Parameters | Any number and types. |
| `const` | Recommended for stateless functors. |

**Syntax Rules**
- `operator()` must be a non-static member.
- It can be overloaded and templated.
- It can be `const`, `noexcept`, etc.
- It can take any number of parameters.

**Constraints and Limitations**
- `operator()` cannot be a non-member.
- Overloading `operator()` can make objects callable with different signatures.

### Annotated Code Examples

**Example 1: Function object**

```cpp
#include <iostream>

class Multiplier {
    int factor_;
public:
    explicit Multiplier(int factor) : factor_(factor) {}
    int operator()(int x) const { return x * factor_; }
};

int main() {
    Multiplier times3(3);
    std::cout << times3(5) << '\n';
}
```

**Expected Output**
```
15
```

**Why this output?**  
`times3(5)` calls `times3.operator()(5)`, which returns `5 * 3 = 15`.

### Real-World Cases

- **STL algorithms** – passing functors to `std::sort`, `std::transform`.
- **`std::function`** – storing callable objects.
- **Strategy pattern** – interchangeable algorithms.
- **Lambdas** – compiler-generated functors.

---

## 10. Conversion Operators

### Definitions

**Core Definition**
A conversion operator is a member function that defines how an object of a class is **converted to another type**.

**Technical Definition**
A conversion operator is declared as `operator TargetType() const;` (or without `const`). It must be a **non-static member function**. It can be `explicit` (since C++11) to prevent implicit conversions. Conversion operators enable objects to be used where the target type is expected. They can be templated (e.g., `operator T()`), but this can lead to ambiguity. The C++ Core Guidelines recommend making conversion operators `explicit` by default (C.164).

**Beginner-Friendly Explanation**
A conversion operator lets your object be **converted to another type**. For example, a `Fraction` class might define `operator double()` so it can be used where a `double` is expected.

### Purposes

- To allow objects to be used as other types.
- To support implicit or explicit conversions.
- To provide alternative representations.
- To integrate with APIs expecting other types.

### Syntax Rules and Structure

**General Syntax**
```cpp
class X {
public:
    operator TargetType() const;           // implicit conversion
    explicit operator TargetType() const;  // explicit conversion (C++11)
};
```

**Component Breakdown**

| Form | Meaning |
|------|---------|
| `operator TargetType()` | Conversion to `TargetType`. |
| `explicit` | Prevents implicit conversion. |
| `const` | Recommended. |

**Syntax Rules**
- Conversion operators must be non-static members.
- They take no parameters.
- They can be `explicit` to prevent implicit conversions.
- They can be templated, but this can cause ambiguity.

**Constraints and Limitations**
- Multiple conversion operators can cause ambiguous conversions.
- Implicit conversions can lead to surprising behavior.
- Conversion to `bool` should be `explicit` (to avoid accidental arithmetic).

### Annotated Code Examples

**Example 1: Conversion operator**

```cpp
#include <iostream>

class Fraction {
    int num_, den_;
public:
    Fraction(int num, int den) : num_(num), den_(den) {}
    explicit operator double() const {
        return static_cast<double>(num_) / den_;
    }
};

int main() {
    Fraction f(1, 4);
    std::cout << static_cast<double>(f) << '\n';
    // double d = f;   // error: explicit conversion
}
```

**Expected Output**
```
0.25
```

**Why this output?**  
`static_cast<double>(f)` calls `operator double()`, returning `1.0 / 4 = 0.25`. The commented line would be an error because the conversion is `explicit`.

### Real-World Cases

- **Numeric types** – `Complex::operator double()`.
- **Smart pointers** – `operator bool()` for null checks.
- **String types** – `operator const char*()`.
- **Ranges** – conversion to iterators.

---

## 11. Increment/Decrement Operators

### Definitions

**Core Definition**
Increment (`++`) and decrement (`--`) operators define how objects are incremented or decremented, with both **prefix** and **postfix** forms.

**Technical Definition**
Prefix increment (`++x`) is a member function taking no parameters: `X& operator++();`. Postfix increment (`x++`) is a member function taking a dummy `int` parameter: `X operator++(int);`. Prefix returns a reference to the modified object; postfix returns a copy of the original object. The postfix form is typically implemented in terms of the prefix form. The same pattern applies to `--`.

**Beginner-Friendly Explanation**
Prefix increment (`++x`) increments and returns the new value; postfix increment (`x++`) returns the old value and then increments. The dummy `int` parameter distinguishes the two forms.

### Purposes

- To support iteration and counting.
- To increment/decrement counters and iterators.
- To provide natural syntax for state machines.
- To support loop constructs.

### Syntax Rules and Structure

**General Syntax**
```cpp
class Counter {
public:
    Counter& operator++();       // prefix ++
    Counter operator++(int);     // postfix ++ (dummy int)
    Counter& operator--();       // prefix --
    Counter operator--(int);     // postfix --
};
```

**Component Breakdown**

| Form | Signature | Returns |
|------|-----------|---------|
| Prefix `++` | `operator++()` | Reference to `*this` |
| Postfix `++` | `operator++(int)` | Copy of original |
| Prefix `--` | `operator--()` | Reference to `*this` |
| Postfix `--` | `operator--(int)` | Copy of original |

**Syntax Rules**
- Prefix operators return `*this` (by reference).
- Postfix operators return a copy of the original value.
- The dummy `int` parameter distinguishes postfix from prefix.
- Postfix is typically implemented in terms of prefix.

**Constraints and Limitations**
- Postfix operators are less efficient (they create a copy).
- Increment/decrement should preserve invariants.
- Overloading both forms is recommended for consistency.

### Annotated Code Examples

**Example 1: Prefix and postfix increment**

```cpp
#include <iostream>

class Counter {
    int value_ = 0;
public:
    Counter& operator++() {          // prefix
        ++value_;
        return *this;
    }
    Counter operator++(int) {        // postfix
        Counter tmp = *this;
        ++(*this);
        return tmp;
    }
    int value() const { return value_; }
};

int main() {
    Counter c;
    Counter a = ++c;
    Counter b = c++;
    std::cout << a.value() << ' ' << b.value() << ' ' << c.value() << '\n';
}
```

**Expected Output**
```
1 1 2
```

**Why this output?**  
`++c` increments `c` to `1` and returns `a` (value `1`). `c++` returns `b` (value `1`), then increments `c` to `2`. Final `c` is `2`.

### Real-World Cases

- **Iterators** – `++it`, `it++`.
- **Counters** – `++count`.
- **State machines** – advancing states.
- **Loops** – `for (auto it = v.begin(); it != v.end(); ++it)`.

---

## 12. Operator Overloading Best Practices

### Definitions

**Core Definition**
Operator overloading best practices are guidelines for overloading operators in ways that are intuitive, consistent, safe, and maintainable, based on the C++ Core Guidelines.

**Technical Definition**
The C++ Core Guidelines provide operator overloading rules: **C.160** (define operators primarily to mimic conventional usage), **C.161** (use nonmember functions for symmetric operators), **C.162** (overload operations that are roughly equivalent), **C.163** (overload only for operations that are roughly equivalent), **C.164** (avoid implicit conversions), **C.165** (use `==` when appropriate), **C.166** (overload `==` to compare equivalent values), **C.167** (overload `==` and `!=` consistently), **C.168** (define `==` and `!=` together), **C.170** (use `operator<<` for stream output), and **C.171** (avoid `operator` abuse). Additional guidance: prefer `explicit` conversion operators, implement compound assignment as members, implement binary arithmetic as non-members in terms of compound assignment, and use the C++20 spaceship operator for comparisons.

**Beginner-Friendly Explanation**
Best practices for operator overloading are simple: **only overload operators when the meaning is obvious**, **follow the built-in semantics**, **use non-members for symmetric operators**, **keep related operators consistent**, **make conversion operators `explicit`**, and **don't abuse operators**.

### Key Guidelines (Summary Table)

| Guideline | Summary | Recommendation |
|-----------|---------|----------------|
| **C.160** | Define operators primarily to mimic conventional usage. | `+` should add, not subtract. |
| **C.161** | Use nonmember functions for symmetric operators. | `==`, `+`, `<` as non-members. |
| **C.162** | Overload operations that are roughly equivalent. | `+` and `+=` should be consistent. |
| **C.163** | Overload only for operations that are roughly equivalent. | Don't give `+` unrelated meanings. |
| **C.164** | Avoid implicit conversions. | Make conversion operators `explicit`. |
| **C.165** | Use `==` when appropriate. | Prefer `==` over `!=` alone. |
| **C.166** | Overload `==` to compare equivalent values. | `==` should mean “equal value.” |
| **C.167** | Overload `==` and `!=` consistently. | `a != b` ≡ `!(a == b)`. |
| **C.168** | Define `==` and `!=` together. | Or use C++20 `<=>`. |
| **C.170** | Use `operator<<` for stream output. | Don't use `operator<<` for other purposes. |
| **C.171** | Avoid `operator` abuse. | Don't overload `&&`, `||`, `,` without strong reason. |

### Annotated Code Examples

**Example 1: Non-member symmetric comparison (C.161, C.168)**

```cpp
#include <iostream>

class Point {
    int x_, y_;
public:
    Point(int x, int y) : x_(x), y_(y) {}

    friend bool operator==(const Point& a, const Point& b) {
        return a.x_ == b.x_ && a.y_ == b.y_;
    }
    friend bool operator!=(const Point& a, const Point& b) {
        return !(a == b);
    }
};

int main() {
    Point a(1, 2), b(1, 2);
    std::cout << (a == b) << ' ' << (a != b) << '\n';
}
```

**Expected Output**
```
1 0
```

**Why this is best practice?**  
`==` and `!=` are non-member functions for symmetry (C.161) and are defined together consistently (C.168).

**Example 2: Consistent arithmetic (C.162)**

```cpp
#include <iostream>

class Vec2 {
    double x_, y_;
public:
    Vec2(double x = 0, double y = 0) : x_(x), y_(y) {}

    Vec2& operator+=(const Vec2& rhs) {
        x_ += rhs.x_;
        y_ += rhs.y_;
        return *this;
    }

    friend Vec2 operator+(Vec2 lhs, const Vec2& rhs) {
        lhs += rhs;
        return lhs;
    }

    void print() const { std::cout << '(' << x_ << ", " << y_ << ")\n"; }
};

int main() {
    Vec2 a(1, 2), b(3, 4);
    Vec2 c = a + b;
    c.print();
}
```

**Expected Output**
```
(4, 6)
```

**Why this is best practice?**  
`+=` is a member (mutating); `+` is a non-member implemented in terms of `+=` (C.162). The semantics are consistent.

### Real-World Cases

- **Standard library** – `std::complex`, `std::string`, `std::vector`.
- **Code review** – Core Guidelines C.160–C.171 are common checkpoints.
- **Static analysis** – clang-tidy enforces many operator overloading rules.

---

## References

- Operator overloading (cppreference) – https://en.cppreference.com/w/cpp/language/operators
- Operator overloading (Microsoft Learn) – https://learn.microsoft.com/en-us/cpp/cpp/operator-overloading
- Member operator overloading (Microsoft Learn) – https://learn.microsoft.com/en-us/cpp/cpp/operator-overloading
- Non-member operator overloading (Microsoft Learn) – https://learn.microsoft.com/en-us/cpp/cpp/operator-overloading
- Friend functions (cppreference) – https://en.cppreference.com/w/cpp/language/friend
- Comparison operators (cppreference) – https://en.cppreference.com/w/cpp/language/operator_comparison
- Three-way comparison (`<=>`) (cppreference) – https://en.cppreference.com/w/cpp/language/operator_comparison#Three-way_comparison
- Stream extraction and insertion (cppreference) – https://en.cppreference.com/w/cpp/io/basic_ostream/operator_ltlt
- Subscript operator (cppreference) – https://en.cppreference.com/w/cpp/language/operator_member_access#Built-in_subscript_operator
- Function call operator (cppreference) – https://en.cppreference.com/w/cpp/language/operators#Function_call_operator
- Conversion operators (cppreference) – https://en.cppreference.com/w/cpp/language/cast_operator
- Increment/decrement operators (cppreference) – https://en.cppreference.com/w/cpp/language/operator_incdec
- C++ Core Guidelines — C.160: Define operators primarily to mimic conventional usage – https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Ro-conventional
- C++ Core Guidelines — C.161: Use nonmember functions for symmetric operators – https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Ro-symmetric
- C++ Core Guidelines — C.162: Overload operations that are roughly equivalent – https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Ro-equivalent
- C++ Core Guidelines — C.163: Overload only for operations that are roughly equivalent – https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Ro-equivalent-2
- C++ Core Guidelines — C.164: Avoid implicit conversion operators – https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Ro-conversion
- C++ Core Guidelines — C.165: Use `==` when appropriate – https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Ro-eq
- C++ Core Guidelines — C.166: Overload `==` to compare equivalent values – https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Ro-eq-2
- C++ Core Guidelines — C.167: Overload `==` and `!=` consistently – https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Ro-eq-3
- C++ Core Guidelines — C.168: Define `==` and `!=` together – https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Ro-eq-4
- C++ Core Guidelines — C.170: Use `operator<<` for stream output – https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Ro-stream
- C++ Core Guidelines — C.171: Avoid `operator` abuse – https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Ro-abuse

---

This cheat sheet provides a complete, structured reference for C++ operator overloading in classes, covering member, non-member, and friend operators; comparison, arithmetic, assignment, stream, subscript, function call, conversion, and increment/decrement operators; and best practices. Each section follows a uniform template with definitions, purposes, syntax rules, annotated examples, expected outputs, and authoritative references.