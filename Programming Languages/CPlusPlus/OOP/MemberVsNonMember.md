# C++ Member vs. Non-Member Operators & Architecture — Comprehensive Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
Operators in C++ can be overloaded as member functions (called on an object) or as non-member functions (called with the object as an argument). The choice between the two affects which operand appears on the left-hand side, how implicit conversions apply, and how overload resolution behaves.

**Technical Definition**
When an operator appears in an expression and at least one operand has a class or enumeration type, overload resolution is used to determine the function to be called among all functions whose signatures match the operator syntax. An operator function must either be a non-static member function or be a non-member function with at least one parameter whose type is a class, a reference to a class, an enumeration, or a reference to an enumeration. Member operators are declared inside the class; non-member operators are declared at namespace scope. Friend functions are non-member functions granted access to the private and protected members of a class.

**Beginner-Friendly Explanation**
When you write `a + b`, the compiler needs to know which function to call. If `operator+` is a member function of `a`'s class, the compiler calls `a.operator+(b)`. If it's a non-member function, the compiler calls `operator+(a, b)`. The difference matters because member operators can only be used when the left operand is of the class type, while non-member operators can handle cases where the left operand is a fundamental type (like `2 + myObject`). Friend operators are non-member functions that have access to private members — useful when you need both symmetry and encapsulation.

---

### Key Characteristics

- **Member operators**: The left operand must be an object of the class (or a derived class); the operator is called as `obj.operator@(arg)`.
- **Non-member operators**: Both operands are parameters; the operator is called as `operator@(lhs, rhs)`.
- **Friend operators**: Non-member functions declared as friends to access private members.
- **Symmetry**: Binary operators that treat both operands equally (arithmetic, comparison) should be non-members to allow implicit conversions on both sides.
- **Overload resolution**: The compiler builds a candidate set from member and non-member operators, then selects the best match.
- **Perfect forwarding**: Advanced operators (like `operator[]`, `operator()`) can use forwarding references to preserve value categories.

---

### Prerequisites

- Solid understanding of classes, member functions, and access specifiers.
- Familiarity with operator overloading syntax and semantics.
- Knowledge of implicit conversions and overload resolution.
- Understanding of friend declarations and ADL (Argument-Dependent Lookup).

---

### Related Programming Areas

- **Operator overloading** (the broader topic).
- **Overload resolution** (how the compiler selects the best match).
- **Implicit conversions** (user-defined conversions and their interaction with operators).
- **ADL** (Argument-Dependent Lookup, which finds non-member operators).
- **Template metaprogramming** (operators in generic code).
- **Perfect forwarding** (preserving value categories in operator implementations).

---

### Core Concepts / Features

1. **Member Overloads** — Left-hand operand must be the class type.
2. **Non-Member Overloads** — Handling fundamental types on the left-hand side.
3. **Friend Overloads** — Granting private access to non-member operators.
4. **Symmetric Operators** — Why binary arithmetic and comparisons should be non-members.
5. **The Overload Resolution Process** — How the compiler matches arguments.
6. **Perfect Forwarding in Operators** — Advanced forwarding references.

---

## 1. Member Overloads (Left-Hand Operand Must Be the Class Type)

### Definitions

**Core Definition**
A member operator is an operator overload declared as a non-static member function of a class. It is invoked with the object on the left-hand side of the operator, and the right-hand operand is passed as a function argument.

**Technical Definition**
A binary operator overloaded as a member function takes exactly one parameter (the right-hand operand); the left-hand operand is the implicit `this` object. A unary operator overloaded as a member takes no parameters. The compiler rewrites `a @ b` (where `@` is a binary operator and `a` is a class object) as `a.operator@(b)`. Member operators cannot be used when the left operand is not of the class type (or a type convertible to it), because there is no object to call the member function on.

**Beginner-Friendly Explanation**
A member operator is like a method of the class. If you write `myObject + 5`, and `operator+` is a member of `myObject`'s class, the compiler calls `myObject.operator+(5)`. But if you write `5 + myObject`, the compiler cannot call `5.operator+(myObject)` because `5` is an `int`, not a class object. Member operators are simple and have access to private members, but they force the class type to be on the left.

---

### Purposes (all begin with "To")

- **To** implement operators that modify the left operand (e.g., `+=`, `-=`, `++`, `--`).
- **To** provide operators that require access to private members without friend declarations.
- **To** implement operators where the left operand is always the class type (e.g., `operator=`, `operator[]`).
- **To** reduce the number of functions in the enclosing namespace.
- **To** enable operators to be virtual (member operators can be virtual; non-member operators cannot).

---

### Syntax Rules and Structure

#### Complete General Syntaxes with Breakdowns

**Member binary operator:**
```cpp
class T {
public:
    T& operator+=(const T& rhs) {
        // modify *this using rhs
        return *this;
    }
};
```
- `operator+=` is called as `a.operator+=(b)`.
- The left operand is `*this`; the right operand is `rhs`.

**Member unary operator:**
```cpp
class T {
public:
    T& operator++() {  // prefix
        ++value_;
        return *this;
    }

    T operator++(int) {  // postfix
        T temp = *this;
        ++(*this);
        return temp;
    }
};
```
- The prefix form takes no parameters; the postfix form takes a dummy `int`.

**Member assignment and subscript:**
```cpp
class T {
public:
    T& operator=(const T& rhs);
    T& operator[](std::size_t index);
};
```
- These operators must be members (`operator=` is implicitly declared as a member; `operator[]` must be a member).

#### Syntax Rules

- A binary member operator takes exactly one parameter.
- A unary member operator takes no parameters (except postfix `++`/`--`, which take an `int`).
- Member operators are called with the object on the left.
- The left operand must be of the class type (or a type derived from it).
- `operator=`, `operator[]`, `operator()`, and `operator->` must be members.

#### Constraints and Limitations

- Member operators cannot be used when the left operand is a fundamental type (e.g., `2 + obj` cannot use a member `operator+`).
- Member operators do not allow implicit conversions on the left operand (the object must already be of the class type).
- A member operator cannot be overloaded for a fundamental type on the left.
- Virtual member operators are possible but rarely useful (they can be overridden in derived classes).

---

### Multiple Annotated Complete Step-by-Step Code Examples and Their Expected Outputs

#### Example 1 — Member Compound Assignment and Increment

```cpp
#include <iostream>

class Counter {
public:
    Counter(int value = 0) : value_(value) {}

    // Member compound assignment
    Counter& operator+=(int n) {
        value_ += n;
        return *this;
    }

    // Member prefix increment
    Counter& operator++() {
        ++value_;
        return *this;
    }

    // Member postfix increment
    Counter operator++(int) {
        Counter temp = *this;
        ++(*this);
        return temp;
    }

    int value() const { return value_; }

private:
    int value_;
};

int main() {
    Counter c(10);

    c += 5;
    std::cout << "After += 5: " << c.value() << std::endl;

    ++c;
    std::cout << "After ++c: " << c.value() << std::endl;

    Counter old = c++;
    std::cout << "After c++: c = " << c.value()
              << ", old = " << old.value() << std::endl;

    return 0;
}
```

**Expected Output:**
```
After += 5: 15
After ++c: 16
After c++: c = 17, old = 16
```

**Why this output:** `c += 5` calls `c.operator+=(5)`, modifying `c` to 15. `++c` calls the prefix member operator, incrementing `c` to 16. `c++` calls the postfix member operator, saving a copy (16), incrementing `c` to 17, and returning the copy. All operators modify or use the left operand (`c`), which is a member of the class.

---

#### Example 2 — Member Assignment and Subscript

```cpp
#include <iostream>
#include <vector>
#include <stdexcept>

class IntVector {
public:
    IntVector(std::size_t size) : data_(size, 0) {}

    // Member assignment operator
    IntVector& operator=(const IntVector& other) {
        if (this != &other) {
            data_ = other.data_;
        }
        return *this;
    }

    // Member subscript operator
    int& operator[](std::size_t index) {
        if (index >= data_.size())
            throw std::out_of_range("Index out of range");
        return data_[index];
    }

    const int& operator[](std::size_t index) const {
        if (index >= data_.size())
            throw std::out_of_range("Index out of range");
        return data_[index];
    }

    std::size_t size() const { return data_.size(); }

private:
    std::vector<int> data_;
};

int main() {
    IntVector v1(3);
    v1[0] = 10;
    v1[1] = 20;
    v1[2] = 30;

    IntVector v2(3);
    v2 = v1;  // member assignment

    std::cout << "v2[0] = " << v2[0] << std::endl;
    std::cout << "v2[1] = " << v2[1] << std::endl;
    std::cout << "v2[2] = " << v2[2] << std::endl;

    return 0;
}
```

**Expected Output:**
```
v2[0] = 10
v2[1] = 20
v2[2] = 30
```

**Why this output:** `v2 = v1` calls the member `operator=`. `v2[0]` calls the member `operator[]`. Both operators must be members because they modify or access the left operand (`v2`). `operator=` and `operator[]` cannot be non-member functions.

---

### Real-World Cases with Explanation

**Case 1 — Standard library containers:** `std::vector::operator[]`, `std::string::operator=`, and `std::array::operator[]` are member operators because the left operand is always the container.

**Case 2 — Smart pointers:** `std::unique_ptr::operator*` and `operator->` are member operators because they access the pointed-to object.

**Case 3 — Iterators:** `operator++`, `operator--`, and `operator*` are member operators on iterator classes.

**Case 4 — Stream buffers:** `std::streambuf::operator=` is a member assignment operator.

---

### References Links

- Operator overloading — cppreference.com - https://en.cppreference.com/w/cpp/language/operators
- Operator Overloading (Chapter 13) — Cambridge University Press - https://www.cambridge.org
- Member vs Non-member Operators — Preshing on Programming - https://preshing.com

---

## 2. Non-Member Overloads (Handling Fundamental Types on the Left)

### Definitions

**Core Definition**
A non-member operator is an operator overload declared as a free function (at namespace scope). Both operands are function parameters, so the operator can be used even when the left operand is a fundamental type or another class type.

**Technical Definition**
A non-member operator function must have at least one parameter whose type is a class, a reference to a class, an enumeration, or a reference to an enumeration. This rule ensures that the operator is at least partially user-defined. Non-member operators are found by unqualified lookup and by argument-dependent lookup (ADL). They allow implicit conversions on both operands, making them ideal for symmetric operators.

**Beginner-Friendly Explanation**
A non-member operator is a free function that takes both operands as parameters. If `operator+` is a non-member function, then `a + b` calls `operator+(a, b)`. Because both operands are function parameters, implicit conversions can apply to both — so `2 + myObject` works if `myObject`'s class has a constructor that accepts an `int`. This is why non-member operators are preferred for arithmetic and comparison operators: they treat both sides equally.

---

### Purposes (all begin with "To")

- **To** allow operators where the left operand is a fundamental type or a different class type.
- **To** enable implicit conversions on both operands (symmetric operators).
- **To** implement binary arithmetic and comparison operators that treat both operands equally.
- **To** allow operators to be found via ADL when the operands are in different namespaces.
- **To** support mixed-type operations (e.g., `2 + rational`, `string + "literal"`).

---

### Syntax Rules and Structure

#### Complete General Syntaxes with Breakdowns

**Non-member binary operator:**
```cpp
T operator+(const T& lhs, const T& rhs) {
    T result = lhs;
    result += rhs;
    return result;
}
```
- Both operands are parameters; implicit conversions apply to both.

**Non-member operator with fundamental left operand:**
```cpp
T operator+(int lhs, const T& rhs) {
    return T(lhs) + rhs;  // convert lhs to T, then use T's operator+
}
```
- Allows `2 + obj` where `2` is an `int`.

**Non-member comparison operator:**
```cpp
bool operator==(const T& lhs, const T& rhs) {
    return lhs.value() == rhs.value();
}
```
- Symmetric; both operands can be implicitly converted.

#### Syntax Rules

- A non-member operator must have at least one parameter of class or enumeration type.
- Non-member operators are declared at namespace scope (or as friends inside a class).
- They are found by unqualified lookup and ADL.
- They cannot be declared `static`, `virtual`, or `const`.
- They can be templates.

#### Constraints and Limitations

- Non-member operators cannot access private members unless declared as friends.
- They cannot be virtual (virtual dispatch requires a member function).
- Some operators must be members: `=`, `[]`, `()`, `->`, and conversion operators.
- Non-member operators must be declared in a namespace where ADL can find them.

---

### Multiple Annotated Complete Step-by-Step Code Examples and Their Expected Outputs

#### Example 1 — Symmetric Arithmetic with Non-Member Operators

```cpp
#include <iostream>

class Rational {
public:
    Rational(int num = 0, int den = 1) : num_(num), den_(den) {}

    // Accessors (public interface)
    int num() const { return num_; }
    int den() const { return den_; }

    // Compound assignment (member)
    Rational& operator+=(const Rational& rhs) {
        num_ = num_ * rhs.den_ + rhs.num_ * den_;
        den_ = den_ * rhs.den_;
        return *this;
    }

private:
    int num_, den_;
};

// Non-member binary arithmetic (symmetric)
Rational operator+(Rational lhs, const Rational& rhs) {
    lhs += rhs;
    return lhs;
}

// Non-member comparison (symmetric)
bool operator==(const Rational& lhs, const Rational& rhs) {
    return lhs.num() * rhs.den() == rhs.num() * lhs.den();
}

int main() {
    Rational a(1, 2), b(1, 3);

    // Both operands are Rational
    Rational c = a + b;
    std::cout << "a + b = " << c.num() << "/" << c.den() << std::endl;

    // Mixed-type: int + Rational (int converts to Rational)
    Rational d = 1 + a;  // 1 converts to Rational(1, 1)
    std::cout << "1 + a = " << d.num() << "/" << d.den() << std::endl;

    // Mixed-type: Rational + int
    Rational e = a + 1;
    std::cout << "a + 1 = " << e.num() << "/" << e.den() << std::endl;

    // Comparison
    std::cout << std::boolalpha;
    std::cout << "a == b: " << (a == b) << std::endl;
    std::cout << "a == Rational(2, 4): "
              << (a == Rational(2, 4)) << std::endl;

    return 0;
}
```

**Expected Output:**
```
a + b = 5/6
1 + a = 3/2
a + 1 = 3/2
a == b: false
a == Rational(2, 4): true
```

**Why this output:** The non-member `operator+` allows implicit conversions on both operands. `1 + a` converts `1` to `Rational(1, 1)`, then adds. `a + 1` also works because `1` converts to `Rational(1, 1)`. If `operator+` were a member, `1 + a` would fail because `1` (an `int`) has no member `operator+`. The non-member comparison operator similarly allows symmetric comparison.

---

#### Example 2 — Mixed-Type Operators with Fundamental Left Operand

```cpp
#include <iostream>

class Meter {
public:
    explicit Meter(double value) : value_(value) {}

    double value() const { return value_; }

    // Member compound assignment
    Meter& operator+=(const Meter& rhs) {
        value_ += rhs.value_;
        return *this;
    }

private:
    double value_;
};

// Non-member operator+ for Meter + Meter
Meter operator+(Meter lhs, const Meter& rhs) {
    lhs += rhs;
    return lhs;
}

// Non-member operator+ for double + Meter
Meter operator+(double lhs, const Meter& rhs) {
    return Meter(lhs) + rhs;
}

// Non-member operator+ for Meter + double
Meter operator+(const Meter& lhs, double rhs) {
    return lhs + Meter(rhs);
}

int main() {
    Meter m(5.0);

    Meter a = m + 2.5;        // Meter + double
    Meter b = 2.5 + m;        // double + Meter
    Meter c = m + Meter(1.5); // Meter + Meter

    std::cout << "m + 2.5 = " << a.value() << std::endl;
    std::cout << "2.5 + m = " << b.value() << std::endl;
    std::cout << "m + Meter(1.5) = " << c.value() << std::endl;

    return 0;
}
```

**Expected Output:**
```
m + 2.5 = 7.5
2.5 + m = 7.5
m + Meter(1.5) = 6.5
```

**Why this output:** Three non-member `operator+` overloads handle `Meter + double`, `double + Meter`, and `Meter + Meter`. The `double + Meter` overload is essential because a member operator could not handle a `double` on the left. Non-member operators enable full symmetry and mixed-type arithmetic.

---

### Real-World Cases with Explanation

**Case 1 — `std::complex`:** Arithmetic operators for `std::complex<T>` are non-members, allowing `2.0 + complex` and `complex + 2.0`.

**Case 2 — `std::string`:** The `operator+` for `std::string` is a non-member, allowing `"literal" + string` and `string + "literal"`.

**Case 3 — Date/time libraries:** Date arithmetic (`date + days`, `days + date`) uses non-member operators to allow both orderings.

**Case 4 — Big integer libraries:** `boost::multiprecision::cpp_int` overloads arithmetic operators as non-members, allowing `2 + bigInt` and `bigInt + 2`.

---

### References Links

- Operator overloading — cppreference.com - https://en.cppreference.com/w/cpp/language/operators
- Non-member operator overloading — Learn C++ - https://www.learncpp.com/cpp-tutorial/overloading-operators-using-non-member-functions/
- C++ Core Guidelines C.161: Use non-member functions for symmetric operators - https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rc-symmetric

---

## 3. Friend Overloads (Granting Private Access to Non-Member Operators)

### Definitions

**Core Definition**
A friend operator is a non-member function declared as a friend of a class. It has the same access to private and protected members as member functions, but it remains a non-member function (so both operands can be parameters).

**Technical Definition**
A friend declaration inside a class grants a non-member function or another class access to the private and protected members of the class. Friend functions are not members; they are not affected by `public`, `private`, or `protected` access specifiers. A friend function can be defined inside the class body (inline) or outside. Friend declarations can be used to grant access to operators that need private data while still allowing implicit conversions on both operands.

**Beginner-Friendly Explanation**
A friend operator is like giving a trusted friend a key to your house. They're not a member of your family (not a member function), but they can access your private rooms (private members). This is useful for operators like `operator<<` and `operator==` that need to read private data but should be symmetric (both operands should be able to convert). The friend declaration says "this function is allowed to see my private members."

---

### Purposes (all begin with "To")

- **To** grant non-member operators access to private members without making them members.
- **To** maintain symmetry while still accessing private data.
- **To** implement stream operators (`<<`, `>>`) that need access to private members.
- **To** allow comparison operators to access private data while remaining symmetric.
- **To** reduce the number of public accessor functions needed.

---

### Syntax Rules and Structure

#### Complete General Syntaxes with Breakdowns

**Friend operator declared and defined inside the class:**
```cpp
class T {
public:
    friend bool operator==(const T& lhs, const T& rhs) {
        return lhs.value_ == rhs.value_;
    }
private:
    int value_;
};
```
- The friend function is defined inline inside the class.

**Friend operator declared inside, defined outside:**
```cpp
class T {
public:
    friend std::ostream& operator<<(std::ostream& os, const T& obj);
private:
    int value_;
};

std::ostream& operator<<(std::ostream& os, const T& obj) {
    return os << obj.value_;
}
```
- The friend declaration grants access; the definition is outside.

**Friend function template:**
```cpp
template <typename T>
class Box {
public:
    friend bool operator==(const Box& lhs, const Box& rhs) {
        return lhs.value_ == rhs.value_;
    }
private:
    T value_;
};
```
- Each instantiation of `Box<T>` has its own friend `operator==`.

#### Syntax Rules

- Friend declarations can appear anywhere in the class body (public, private, or protected).
- A friend function is not a member; it is not called with `this`.
- Friend functions are found by ADL, not by ordinary unqualified lookup (for the version defined inline).
- Friend functions can be defined inside the class body (implicitly inline).
- Friend declarations do not affect access for other functions.

#### Constraints and Limitations

- Friend functions cannot be virtual.
- Friend declarations are not inherited (a friend of a base class is not a friend of a derived class).
- Friend functions defined inside a class template are instantiated per specialization.
- Overusing friends reduces encapsulation; use them only when necessary.

---

### Multiple Annotated Complete Step-by-Step Code Examples and Their Expected Outputs

#### Example 1 — Friend Comparison and Stream Operators

```cpp
#include <iostream>

class Point {
public:
    Point(int x = 0, int y = 0) : x_(x), y_(y) {}

    // Friend comparison operator
    friend bool operator==(const Point& lhs, const Point& rhs) {
        return lhs.x_ == rhs.x_ && lhs.y_ == rhs.y_;
    }

    // Friend stream insertion operator
    friend std::ostream& operator<<(std::ostream& os, const Point& p) {
        os << "(" << p.x_ << ", " << p.y_ << ")";
        return os;
    }

    // Friend arithmetic operator
    friend Point operator+(const Point& lhs, const Point& rhs) {
        return Point(lhs.x_ + rhs.x_, lhs.y_ + rhs.y_);
    }

private:
    int x_, y_;
};

int main() {
    Point a(1, 2), b(3, 4);
    Point c = a + b;

    std::cout << "a = " << a << std::endl;
    std::cout << "b = " << b << std::endl;
    std::cout << "a + b = " << c << std::endl;

    std::cout << std::boolalpha;
    std::cout << "a == b: " << (a == b) << std::endl;
    std::cout << "a == Point(1, 2): " << (a == Point(1, 2)) << std::endl;

    return 0;
}
```

**Expected Output:**
```
a = (1, 2)
b = (3, 4)
a + b = (4, 6)
a == b: false
a == Point(1, 2): true
```

**Why this output:** The friend operators have access to `x_` and `y_` (private members) without needing public accessors. `operator==` compares both coordinates. `operator<<` prints the point. `operator+` creates a new point with the sum of coordinates. Because they are non-members, implicit conversions can apply to both operands (though not exercised in this example).

---

#### Example 2 — Hidden Friend (ADL-Only) Operators

```cpp
#include <iostream>

class Fraction {
public:
    Fraction(int num, int den) : num_(num), den_(den) {}

    // Hidden friend: only found by ADL
    friend bool operator==(const Fraction& lhs, const Fraction& rhs) {
        return lhs.num_ * rhs.den_ == rhs.num_ * lhs.den_;
    }

    friend std::ostream& operator<<(std::ostream& os, const Fraction& f) {
        return os << f.num_ << "/" << f.den_;
    }

private:
    int num_, den_;
};

int main() {
    Fraction a(1, 2), b(2, 4);

    std::cout << "a = " << a << std::endl;
    std::cout << "a == b: " << (a == b) << std::endl;

    // This would not compile because the friend is not visible:
    // operator==(a, b);  // Error: not found by unqualified lookup

    return 0;
}
```

**Expected Output:**
```
a = 1/2
a == b: 1
```

**Why this output:** The friend operators are defined inside the class body but are not visible to unqualified lookup. They are found by ADL when `Fraction` arguments are used. This prevents them from polluting the namespace and reduces overload set sizes, improving compile times. This is the "hidden friend" idiom.

---

### Real-World Cases with Explanation

**Case 1 — Stream operators:** `operator<<` and `operator>>` are almost always friends (or hidden friends) because they need access to private members but must be non-members (the left operand is the stream).

**Case 2 — Comparison operators:** Symmetric comparison operators (`==`, `!=`, `<`, `>`, `<=`, `>=`) are often friends to access private data while allowing conversions.

**Case 3 — Arithmetic operators:** `operator+`, `operator-`, etc., are often friends to access private data and maintain symmetry.

**Case 4 — Boost.Operators:** Boost provides a library for generating friend operators from a minimal set of member operators, reducing boilerplate.

**Case 5 — Hidden friends in libstdc++:** GCC's libstdc++ uses hidden friends for iterator operators to reduce symbol visibility and improve compile times.

---

### References Links

- friend declaration — cppreference.com - https://en.cppreference.com/w/cpp/language/friend
- Hidden friends — SonarSource Rules - https://rules.sonarsource.com/cpp/RSPEC-3471
- Know the hidden friend idiom — Dr-Sergey/learn_cpp (GitHub) - https://github.com/Dr-Sergey/learn_cpp
- Friend functions and classes — Microsoft Learn - https://learn.microsoft.com/en-us/cpp/cpp/friend-cpp

---

## 4. Symmetric Operators (Why Binary Arithmetic and Comparisons Should Be Non-Members)

### Definitions

**Core Definition**
A symmetric operator treats both operands equally, allowing implicit conversions on both the left and right sides. Symmetric operators should be non-member functions (often friends) to ensure that both operands can be converted.

**Technical Definition**
If a binary operator is a member function, the left operand must be an object of the class type; the right operand can be any type that is convertible to the parameter type. This asymmetry means that `obj + 2` works (2 converts to the class type) but `2 + obj` does not (2 is not the class type). Non-member operators take both operands as parameters, so implicit conversions apply to both, making the operator symmetric. C++ Core Guidelines C.161 states: "Use non-member functions for symmetric operators."

**Beginner-Friendly Explanation**
Symmetric means "it works the same way on both sides." If `1 + 2` is the same as `2 + 1` for your type, then your `operator+` should be symmetric. A member `operator+` makes `obj + 2` work but not `2 + obj`. A non-member `operator+` makes both work. This is why arithmetic and comparison operators should be non-members: they treat both operands equally.

---

### Purposes (all begin with "To")

- **To** allow implicit conversions on both operands of a binary operator.
- **To** ensure that `a + b` and `b + a` behave consistently.
- **To** support mixed-type operations in both orders.
- **To** follow the C++ Core Guidelines recommendation for symmetric operators.
- **To** improve code consistency and intuitiveness.

---

### Syntax Rules and Structure

#### Complete General Syntax with Breakdown

```cpp
// Compound assignment (member — modifies left operand)
class T {
public:
    T& operator+=(const T& rhs);
};

// Binary arithmetic (non-member — symmetric)
T operator+(T lhs, const T& rhs) {
    lhs += rhs;
    return lhs;
}
```

#### Syntax Rules

- Binary arithmetic operators (`+`, `-`, `*`, `/`, `%`) should be non-members.
- Binary comparison operators (`==`, `!=`, `<`, `>`, `<=`, `>=`) should be non-members.
- Compound assignment operators (`+=`, `-=`, etc.) should be members.
- The non-member binary operator should be implemented in terms of the member compound assignment.
- Both operands should be able to undergo implicit conversion.

#### Constraints and Limitations

- Non-member binary operators that need private access must be friends.
- The left operand is taken by value for operators that return a modified copy.
- Symmetric operators can lead to ambiguity if multiple conversions are possible.
- ADL must be able to find the non-member operator.

---

### Multiple Annotated Complete Step-by-Step Code Examples and Their Expected Outputs

#### Example 1 — Symmetric vs. Asymmetric Operators

```cpp
#include <iostream>

class Number {
public:
    Number(int value = 0) : value_(value) {}

    // Member operator+ (asymmetric)
    Number operator+(const Number& rhs) const {
        return Number(value_ + rhs.value_);
    }

    int value() const { return value_; }

private:
    int value_;
};

// Non-member operator+ (symmetric)
Number operator+(Number lhs, const Number& rhs) {
    lhs += rhs;
    return lhs;
}

int main() {
    Number n(10);

    // Member: n + 5 works, 5 + n does not (if member only)
    // Non-member: both work
    Number a = n + 5;   // uses non-member
    Number b = 5 + n;   // uses non-member

    std::cout << "n + 5 = " << a.value() << std::endl;
    std::cout << "5 + n = " << b.value() << std::endl;

    return 0;
}
```

**Expected Output:**
```
n + 5 = 15
5 + n = 15
```

**Why this output:** The non-member `operator+` allows implicit conversion of `5` to `Number(5)` on both sides. If `operator+` were only a member, `n + 5` would work (5 converts to Number), but `5 + n` would fail because `5` (an `int`) has no member `operator+`. The non-member version makes the operator symmetric.

---

### Real-World Cases with Explanation

**Case 1 — `std::complex`:** All arithmetic operators are non-members, allowing `2.0 + complex` and `complex + 2.0`.

**Case 2 — `std::string`:** `operator+` is a non-member, allowing `"hello" + string` and `string + "hello"`.

**Case 3 — Date/time libraries:** `date + days` and `days + date` both work because the operators are non-members.

**Case 4 — Matrix libraries:** `scalar * matrix` and `matrix * scalar` both work because the operators are non-members.

**Case 5 — Boost.Operators:** The library generates symmetric operators from member compound assignments, ensuring symmetry without manual duplication.

---

### References Links

- C++ Core Guidelines C.161: Use non-member functions for symmetric operators - https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rc-symmetric
- Operator overloading — cppreference.com - https://en.cppreference.com/w/cpp/language/operators
- Non-member operator overloading — Learn C++ - https://www.learncpp.com/cpp-tutorial/overloading-operators-using-non-member-functions/

---

## 5. The Overload Resolution Process (How the Compiler Matches Arguments)

### Definitions

**Core Definition**
Overload resolution is the process by which the compiler selects the best function to call from a set of candidates (member operators, non-member operators, and built-in operators) based on the types of the operands.

**Technical Definition**
When an operator appears in an expression, overload resolution is used to determine the user-defined function to be called among all functions whose signatures match the operator syntax. The process has three steps: (1) build the candidate set (member operators, non-member operators found by unqualified lookup and ADL, and built-in operators); (2) determine the viable candidates (those for which implicit conversion sequences exist for all arguments); (3) select the best viable candidate by comparing conversion sequences. If no candidate is better than all others, the call is ambiguous.

**Beginner-Friendly Explanation**
When you write `a + b`, the compiler collects all the possible `operator+` functions it can find: member operators of `a`'s class, non-member operators in the same namespace as `a` and `b` (found by ADL), and built-in operators. Then it checks which ones can accept the arguments (possibly with conversions). Finally, it picks the one that requires the least conversion. If two are equally good, it's an ambiguity error.

---

### Purposes (all begin with "To")

- **To** select the most appropriate operator function from multiple candidates.
- **To** prefer exact matches over matches requiring conversions.
- **To** prefer non-template functions over template functions when both are equally good.
- **To** detect and report ambiguous operator calls.
- **To** understand why certain operators are selected over others.

---

### Syntax Rules and Structure

#### Overload Resolution Steps

1. **Build candidate set**: member operators + non-member operators (unqualified lookup + ADL) + built-in operators.
2. **Determine viable candidates**: those for which implicit conversion sequences exist for all arguments.
3. **Rank conversion sequences**: exact match > promotion > standard conversion > user-defined conversion.
4. **Select best viable candidate**: if one candidate is better than all others, it wins; otherwise, the call is ambiguous.
5. **Tie-breakers**: non-template over template; more specialized template over less specialized.

#### Syntax Rules

- Member operators are considered only when the left operand is of the class type (or a derived class).
- Non-member operators are considered when at least one operand has a class or enumeration type.
- Built-in operators are considered for fundamental types.
- Implicit conversions are applied only if needed to make a candidate viable.

#### Constraints and Limitations

- Ambiguity errors occur when two candidates are equally good matches.
- User-defined conversions are ranked lower than standard conversions.
- A member operator and a non-member operator with the same signature can cause ambiguity if both are viable.
- ADL can find operators in namespaces associated with the argument types.

---

### Multiple Annotated Complete Step-by-Step Code Examples and Their Outputs

#### Example 1 — Member vs. Non-Member Ambiguity

```cpp
#include <iostream>

class T {
public:
    // Member operator+
    T operator+(const T& rhs) const {
        std::cout << "Member operator+" << std::endl;
        return T();
    }
};

// Non-member operator+
T operator+(const T& lhs, const T& rhs) {
    std::cout << "Non-member operator+" << std::endl;
    return T();
}

int main() {
    T a, b;
    // T c = a + b;  // AMBIGUOUS: both member and non-member are viable
    // Compiler error: ambiguous overload for 'operator+'
    return 0;
}
```

**Expected Output (compilation error):**
```
error: ambiguous overload for 'operator+' (operand types are 'T' and 'T')
note: candidate: 'T T::operator+(const T&) const'
note: candidate: 'T operator+(const T&, const T&)'
```

**Why this error:** Both the member `operator+` and the non-member `operator+` are viable candidates with identical conversion sequences (exact match for both). The compiler cannot decide which one to call, so it reports an ambiguity error. To fix this, remove one of the operators.

---

#### Example 2 — Overload Resolution with Implicit Conversions

```cpp
#include <iostream>

class T {
public:
    T(int value = 0) : value_(value) {}

    // Member operator+ taking int
    T operator+(int rhs) const {
        std::cout << "Member operator+(int)" << std::endl;
        return T(value_ + rhs);
    }

    int value() const { return value_; }

private:
    int value_;
};

// Non-member operator+ taking two T's
T operator+(const T& lhs, const T& rhs) {
    std::cout << "Non-member operator+(T, T)" << std::endl;
    return T(lhs.value() + rhs.value());
}

int main() {
    T a(10);

    // Exact match for member: int parameter
    T b = a + 5;     // calls member operator+(int)

    // Both viable: member requires int->T conversion, non-member is exact
    T c = a + T(5);  // calls non-member (exact match)

    std::cout << "a + 5 = " << b.value() << std::endl;
    std::cout << "a + T(5) = " << c.value() << std::endl;

    return 0;
}
```

**Expected Output:**
```
Member operator+(int)
Non-member operator+(T, T)
a + 5 = 15
a + T(5) = 15
```

**Why this output:** For `a + 5`, the member `operator+(int)` is an exact match (5 is an `int`), while the non-member would require converting `5` to `T`. The member wins. For `a + T(5)`, both the member (requires `T` to `int`? no, that's not viable) and the non-member are considered. The non-member is an exact match for both arguments, so it wins.

---

### Real-World Cases with Explanation

**Case 1 — Stream operators:** Overload resolution selects the correct `operator<<` based on the type being printed. User-defined types use their own `operator<<`, while built-in types use the standard library's.

**Case 2 — Comparison operators:** When comparing two objects of different types (e.g., `std::string` and `const char*`), overload resolution selects the best match, often converting one operand.

**Case 3 — Smart pointers:** `std::shared_ptr` and `std::unique_ptr` have `operator==` overloads that allow comparison with `nullptr` and with each other. Overload resolution selects the correct one.

**Case 4 — Arithmetic with mixed types:** `2 + 3.5` selects the built-in `operator+(double, double)` after promoting `2` to `double`. For user-defined types, overload resolution selects the user-defined operator.

---

### References Links

- Overload resolution — cppreference.com - https://en.cppreference.com/w/cpp/language/overload_resolution
- Overload resolution — Microsoft Learn - https://learn.microsoft.com/en-us/cpp/cpp/overload-resolution
- C++ Overload Resolution — IBM - https://www.ibm.com/docs

---

## 6. Perfect Forwarding in Operators (Advanced Forwarding References)

### Definitions

**Core Definition**
Perfect forwarding in operators uses forwarding references (`T&&`) and `std::forward` to preserve the value categories of operands when implementing operators, particularly for templated operators like `operator[]`, `operator()`, and comparison operators.

**Technical Definition**
Perfect forwarding is a technique that allows a function template to accept an argument of either lvalue or rvalue reference type and forward it to another function, preserving its lvalue-ness or rvalue-ness. In the context of operators, this is useful for templated operators that need to forward their operands to another function or for operators that construct objects from forwarded arguments. The key components are forwarding references (`Args&&...`), `std::forward<Args>(args)`, and the deduction rules for template arguments.

**Beginner-Friendly Explanation**
Perfect forwarding is about preserving whether an argument is an lvalue (something with a name) or an rvalue (a temporary). When you write a templated operator like `operator[]` that needs to pass its argument to another function, you want the inner function to see the same value category as the original call. Without perfect forwarding, rvalues become lvalues when they get a parameter name, losing their move semantics. `std::forward` fixes this by restoring the original value category.

---

### Purposes (all begin with "To")

- **To** preserve move semantics when forwarding operands to other functions.
- **To** avoid unnecessary copies in templated operators.
- **To** implement generic operators that work with any argument types.
- **To** enable perfect forwarding of constructor arguments in factory-like operators.
- **To** support heterogeneous comparison and arithmetic in generic code.

---

### Syntax Rules and Structure

#### Complete General Syntax with Breakdown

```cpp
template <typename... Args>
auto operator()(Args&&... args) -> decltype(target(std::forward<Args>(args)...)) {
    return target(std::forward<Args>(args)...);
}
```

**Component breakdown:**
- `Args&&... args` — forwarding references (deduce lvalue or rvalue).
- `std::forward<Args>(args)...` — expands the pack, forwarding each argument.
- `target(...)` — the function being forwarded to.

#### Syntax Rules

- Forwarding references are formed by `T&&` where `T` is a template parameter.
- `std::forward<T>(arg)` requires an explicit template argument `T`.
- The pack expansion `std::forward<Args>(args)...` expands to `std::forward<Arg1>(arg1), std::forward<Arg2>(arg2), ...`.
- `std::forward` is declared in `<utility>`.
- For lvalue arguments, `Args` deduces to `T&`; `std::forward<T&>` returns an lvalue.
- For rvalue arguments, `Args` deduces to `T`; `std::forward<T>` returns an rvalue.

#### Constraints and Limitations

- Forwarding references can accidentally bind to copy/move constructors, causing overload ambiguity; use `std::enable_if` or concepts to guard perfect forwarding constructors.
- `std::forward` must be called with the template parameter, not the deduced type.
- Perfect forwarding does not work with braced initializer lists, `0` as a null pointer constant, or overloaded function names.
- Forwarding references are not the same as rvalue references; `T&&` where `T` is a class template parameter is an rvalue reference, not a forwarding reference.

---

### Multiple Annotated Complete Step-by-Step Code Examples and Their Outputs

#### Example 1 — Perfect Forwarding in a Call Operator

```cpp
#include <iostream>
#include <utility>
#include <string>

// Step 1: A callable object that forwards its arguments
class Logger {
public:
    template <typename... Args>
    void operator()(Args&&... args) const {
        std::cout << "Log: ";
        (std::cout << ... << std::forward<Args>(args)) << std::endl;
    }
};

int main() {
    Logger log;

    // Step 2: Forward lvalues
    std::string msg = "hello";
    log(msg);  // forwards lvalue

    // Step 3: Forward rvalues
    log(std::string("world"));  // forwards rvalue

    // Step 4: Forward mixed types
    log("Number: ", 42, " and ", 3.14);

    return 0;
}
```

**Expected Output:**
```
Log: hello
Log: world
Log: Number: 42 and 3.14
```

**Why this output:** The `operator()` is a variadic function template with forwarding references. `std::forward<Args>(args)` preserves the value category of each argument. For `log(msg)`, `msg` is an lvalue, so it is forwarded as an lvalue. For `log(std::string("world"))`, the temporary is an rvalue, so it is forwarded as an rvalue (enabling move semantics if the target function were to move it). The fold expression `(std::cout << ... << ...)` prints all arguments.

---

#### Example 2 — Perfect Forwarding in a Subscript Operator

```cpp
#include <iostream>
#include <utility>
#include <map>
#include <string>

template <typename Key, typename Value>
class LazyMap {
public:
    // Perfect-forwarding subscript operator
    template <typename K>
    Value& operator[](K&& key) {
        auto it = map_.find(key);
        if (it == map_.end()) {
            // Construct the key in place using perfect forwarding
            it = map_.emplace(std::forward<K>(key), Value{}).first;
        }
        return it->second;
    }

    std::size_t size() const { return map_.size(); }

private:
    std::map<Key, Value> map_;
};

int main() {
    LazyMap<std::string, int> m;

    // Step 1: Forward lvalue key
    std::string key = "one";
    m[key] = 1;

    // Step 2: Forward rvalue key
    m[std::string("two")] = 2;

    // Step 3: Forward string literal (converted to std::string)
    m["three"] = 3;

    std::cout << "m[\"one\"] = " << m["one"] << std::endl;
    std::cout << "m[\"two\"] = " << m["two"] << std::endl;
    std::cout << "m[\"three\"] = " << m["three"] << std::endl;
    std::cout << "size = " << m.size() << std::endl;

    return 0;
}
```

**Expected Output:**
```
m["one"] = 1
m["two"] = 2
m["three"] = 3
size = 3
```

**Why this output:** The templated `operator[]` uses a forwarding reference `K&&` to accept the key. `std::forward<K>(key)` forwards the key to `map_.emplace`, preserving its value category. For `m[key]` (lvalue), the key is forwarded as an lvalue, avoiding a copy. For `m[std::string("two")]` (rvalue), the key is forwarded as an rvalue, enabling a move instead of a copy. For `m["three"]` (string literal), a temporary `std::string` is constructed and forwarded as an rvalue.

---

### Real-World Cases with Explanation

**Case 1 — `std::make_shared` and `std::make_unique`:** These factory functions use perfect forwarding to pass their arguments directly to the constructor of the object being created, eliminating intermediate copies.

**Case 2 — `emplace_back` and `emplace`:** Container member functions use perfect forwarding to construct elements in place, avoiding temporary objects and enabling move-only types.

**Case 3 — `std::apply`:** `std::apply` uses perfect forwarding to call a function with a tuple's elements as arguments, preserving the value categories.

**Case 4 — Generic wrapper functions:** A logging or timing wrapper can perfect-forward its arguments to the wrapped function while adding instrumentation.

**Case 5 — `std::function` and `std::bind`:** These use perfect forwarding to store and invoke callable objects with the correct value categories.

---

### References Links

- `std::forward` — cppreference.com - https://en.cppreference.com/w/cpp/utility/forward
- `forward` — Microsoft Learn - https://learn.microsoft.com/en-us/cpp/standard-library/forward-function
- Move semantics and perfect forwarding — ISOCPP - https://isocpp.org/blog/2012/11/universal-references-in-c11-scott-meyers
- Perfect forwarding — cppreference.com - https://en.cppreference.com/w/cpp/language/reference#Forwarding_references
- clang-tidy: bugprone-forwarding-reference-overload — LLVM - https://clang.llvm.org/extra/clang-tidy/checks/bugprone/forwarding-reference-overload.html

---

## Summary Table of Member vs. Non-Member Operators

| Aspect | Member Operator | Non-Member Operator | Friend Operator |
|---|---|---|---|
| Left operand | Must be class type | Can be any type (with conversion) | Can be any type (with conversion) |
| Symmetry | No (left operand must be class) | Yes (both operands are parameters) | Yes (both operands are parameters) |
| Private access | Yes (implicitly) | No (unless friend) | Yes (via friend declaration) |
| Virtual | Yes | No | No |
| Found by | Member lookup | Unqualified lookup + ADL | ADL (hidden friend) |
| Must be member | `=`, `[]`, `()`, `->`, conversion | Never | Never |
| Recommended for | Compound assignment, increment/decrement | Binary arithmetic, comparison | Stream operators, symmetric operators needing private access |

---

## References (Consolidated)

- Operator overloading — cppreference.com - https://en.cppreference.com/w/cpp/language/operators
- C++ Core Guidelines C.161: Use non-member functions for symmetric operators - https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rc-symmetric
- C++ Core Guidelines C.160: Define operators primarily to mimic conventional usage - https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rc-operators
- friend declaration — cppreference.com - https://learn.microsoft.com/en-us/cpp/cpp/friend-cpp
- Hidden friends — SonarSource Rules - https://rules.sonarsource.com/cpp/RSPEC-3471
- Overload resolution — cppreference.com - https://en.cppreference.com/w/cpp/language/overload_resolution
- `std::forward` — cppreference.com - https://en.cppreference.com/w/cpp/utility/forward
- Move semantics and perfect forwarding — ISOCPP - https://isocpp.org/blog/2012/11/universal-references-in-c11-scott-meyers
- Non-member operator overloading — Learn C++ - https://www.learncpp.com/cpp-tutorial/overloading-operators-using-non-member-functions/
- Operator Overloading (Chapter 13) — Cambridge University Press - https://www.cambridge.org
- Know the hidden friend idiom — Dr-Sergey/learn_cpp (GitHub) - https://github.com/Dr-Sergey/learn_cpp
- clang-tidy: bugprone-forwarding-reference-overload — LLVM - https://clang.llvm.org/extra/clang-tidy/checks/bugprone/forwarding-reference-overload.html
- Operator Overloading — IBM - https://www.ibm.com/docs/en/zos/2.4.0?topic=only-operator-overloading-c