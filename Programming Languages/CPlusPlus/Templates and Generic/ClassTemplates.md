# C++ Class Templates — Comprehensive Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
A class template is a blueprint for creating a family of classes. It is not a class itself; it is a parameterized pattern from which concrete classes are generated when template arguments are supplied.

**Technical Definition**
A class template defines a layout and operations parameterized by one or more template parameters (type, non-type, template, or parameter pack). It is declared using the syntax `template <parameter-list> class-declaration`. Each instantiation of a class template produces a distinct class type with its own members and static data. Members of class templates can be defined inside or outside the class body, and when defined outside, they require the `template <...>` prefix and the `ClassName<T>::` qualification. Since C++17, class template argument deduction (CTAD) allows template arguments to be deduced from constructor arguments, eliminating the need for explicit specification in many cases.

**Beginner-Friendly Explanation**
A class template is like a cookie cutter: it defines the shape, but it's not a cookie. When you use the cookie cutter with a specific dough (a type), you get a cookie (a concrete class). For example, `std::vector<int>` and `std::vector<std::string>` are two different classes generated from the same `std::vector` class template. You write the code once, and the compiler generates the appropriate class for each type you use.

---

### Key Characteristics

- **Generic blueprints**: Define type-agnostic classes, structures, and alias templates.
- **Member templates**: Class templates can contain nested class templates and member function templates.
- **Friend templates**: Class templates can declare functions, classes, or other templates as friends.
- **CTAD (C++17)**: Class template arguments can be deduced from constructor arguments.
- **Deduction guides**: Custom rules for CTAD beyond what constructors provide.
- **Explicit instantiation control**: `extern template` suppresses redundant instantiation across translation units.
- **Header-only or split**: Template definitions can be header-only or explicitly instantiated in a `.cpp` file.

---

### Prerequisites

- Basic C++ syntax: classes, functions, constructors, and member access.
- Understanding of function templates (type parameters, deduction).
- Familiarity with the One Definition Rule (ODR) and translation units.
- (For CTAD) C++17 or later; (for concepts) C++20 or later.

---

### Related Programming Areas

- **Function templates** (templates for functions).
- **Template specialization** (full and partial specialization of class templates).
- **Concepts and constraints** (C++20, restricting template parameters).
- **Standard library containers** (`std::vector`, `std::map`, `std::optional`).
- **Metaprogramming** (type traits, policy-based design).

---

### Core Concepts / Features

1. **Generic Blueprints** — Designing type-agnostic classes, structures, and modern template aliases (`using`).
2. **Internal Signatures** — Implementing nested template member functions and friend templates.
3. **CTAD** — Class Template Argument Deduction and custom deduction guides.
4. **Component Grouping** — Structuring header-only template definitions vs. `extern template` for explicit instantiation control.

---

## 1. Generic Blueprints

### Definitions

**Core Definition**
Generic blueprints are class templates and alias templates that define type-agnostic classes and type aliases, parameterized by one or more template parameters.

**Technical Definition**
A class template is declared with `template <typename T> class ClassName { ... };`. The `typename` and `class` keywords are equivalent for type parameters; there is no semantic difference between them. An alias template (C++11) is declared with `template <typename T> using AliasName = ...;`. Unlike typedefs, alias templates can be parameterized and can refer to dependent types. Alias templates are never deduced by template argument deduction and can only be declared at class or namespace scope.

**Beginner-Friendly Explanation**
A class template is a generic class: `template <typename T> class Box { T value; };` defines a `Box` that can hold any type. An alias template is a shortcut for a complex type: `template <typename T> using Vec = std::vector<T, MyAllocator<T>>;` lets you write `Vec<int>` instead of the full type. Both are "blueprints" — they don't exist as concrete types until you supply the template arguments.

---

### Purposes (all begin with "To")

- **To** define a single class implementation that works with many different types.
- **To** avoid code duplication when the same class logic applies to multiple types.
- **To** enable generic programming, where data structures are decoupled from specific element types.
- **To** provide type-safe containers, smart pointers, and utility classes.
- **To** simplify complex type expressions with alias templates.

---

### Syntax Rules and Structure

#### Complete General Syntaxes with Breakdowns

**Syntax 1 — Basic class template**
```cpp
template <typename T>
class Box {
public:
    void set(const T& value) { value_ = value; }
    T get() const { return value_; }
private:
    T value_;
};
```
**Component breakdown:**
- `template <typename T>` — template parameter list.
- `class Box { ... }` — class template definition; `T` is used as a placeholder type.

**Syntax 2 — Alias template (C++11)**
```cpp
template <typename T>
using Vec = std::vector<T, MyAllocator<T>>;
```
- `using Vec = ...` — declares an alias template.
- `Vec<int>` is equivalent to `std::vector<int, MyAllocator<int>>`.

**Syntax 3 — Partial specialization**
```cpp
template <typename T>
class Box<T*> {
    // specialization for pointer types
};
```
- Partial specialization allows customizing the class template for a category of types (e.g., pointers).

#### Syntax Rules

- `typename` and `class` are interchangeable for type parameters.
- Class templates can be partially specialized; function templates cannot.
- Alias templates cannot be specialized; they are simply aliases.
- Member functions defined outside the class template require the `template <...>` prefix and `ClassName<T>::` qualification.
- Since C++20, class templates can be constrained with concepts: `template <std::integral T> class Box { ... };`.

#### Constraints and Limitations

- Class template definitions must be visible at the point of instantiation (typically in headers).
- Partial specializations must be declared after the primary template.
- Alias templates are not deduced when deducing template template parameters.
- Static data members of class templates must be defined outside the class template (or since C++17, can be `inline`).

---

### Multiple Annotated Complete Step-by-Step Code Examples and Their Expected Outputs

#### Example 1 — Basic Class Template with Alias Template

```cpp
#include <iostream>
#include <vector>
#include <string>

// Step 1: Define a class template
template <typename T>
class Box {
public:
    explicit Box(const T& value) : value_(value) {}
    T get() const { return value_; }
    void set(const T& value) { value_ = value; }
private:
    T value_;
};

// Step 2: Define an alias template
template <typename T>
using BoxList = std::vector<Box<T>>;

int main() {
    // Step 3: Instantiate with int
    Box<int> intBox(42);
    std::cout << "intBox: " << intBox.get() << std::endl;

    // Step 4: Instantiate with std::string
    Box<std::string> strBox("hello");
    std::cout << "strBox: " << strBox.get() << std::endl;

    // Step 5: Use the alias template
    BoxList<double> boxes;
    boxes.emplace_back(3.14);
    boxes.emplace_back(2.71);
    std::cout << "boxes[0]: " << boxes[0].get() << std::endl;

    return 0;
}
```

**Expected Output:**
```
intBox: 42
strBox: hello
boxes[0]: 3.14
```

**Why this output:** `Box<int>` and `Box<std::string>` are two distinct classes generated from the same template. The alias template `BoxList<double>` is equivalent to `std::vector<Box<double>>`, allowing a shorter, more readable type name. The class template is instantiated with `int`, `std::string`, and `double`, and each instantiation has its own `get()` and `set()` methods.

---

#### Example 2 — Partial Specialization for Pointers

```cpp
#include <iostream>

// Step 1: Primary class template
template <typename T>
class Printer {
public:
    void print(const T& value) {
        std::cout << "Value: " << value << std::endl;
    }
};

// Step 2: Partial specialization for pointer types
template <typename T>
class Printer<T*> {
public:
    void print(T* value) {
        if (value) {
            std::cout << "Pointer to: " << *value << std::endl;
        } else {
            std::cout << "Null pointer" << std::endl;
        }
    }
};

int main() {
    // Step 3: Use the primary template
    Printer<int> intPrinter;
    intPrinter.print(42);

    // Step 4: Use the partial specialization for pointers
    int x = 100;
    Printer<int*> ptrPrinter;
    ptrPrinter.print(&x);
    ptrPrinter.print(nullptr);

    return 0;
}
```

**Expected Output:**
```
Value: 42
Pointer to: 100
Null pointer
```

**Why this output:** `Printer<int>` uses the primary template, printing the value directly. `Printer<int*>` uses the partial specialization for `T*`, which dereferences the pointer and prints the pointed-to value, with a special case for null pointers. Partial specialization allows the class template to behave differently for pointer types without duplicating the entire class.

---

### Real-World Cases with Explanation

**Case 1 — Standard library containers:** `std::vector`, `std::map`, `std::list`, and `std::unordered_map` are class templates that provide type-safe, generic containers. `std::vector<int>` and `std::vector<std::string>` are distinct classes generated from the same template.

**Case 2 — Smart pointers:** `std::unique_ptr<T>` and `std::shared_ptr<T>` are class templates that manage memory for any type. `std::make_unique<T>` and `std::make_shared<T>` are function templates that deduce the type.

**Case 3 — Alias templates for allocators:** Libraries like the C++ standard library use alias templates to provide convenient names for complex types, e.g., `template <typename T> using Vec = std::vector<T, MyAlloc<T>>;`.

**Case 4 — Policy-based design:** Class templates are used to compose classes from independent policies (e.g., `template <typename CreationPolicy, typename ThreadingPolicy> class Manager`), enabling flexible, reusable components.

---

### References

- Class template — cppreference.com - https://en.cppreference.com/w/cpp/language/class_template
- Type alias, alias template — cppreference.com - https://en.cppreference.com/w/cpp/language/type_alias
- Aliases and typedefs (C++) — Microsoft Learn - https://learn.microsoft.com/en-us/cpp/cpp/aliases-and-typedefs-cpp
- Partial specialization — cppreference.com - https://en.cppreference.com/w/cpp/language/partial_specialization

---

## 2. Internal Signatures

### Definitions

**Core Definition**
Internal signatures refer to the member functions, nested class templates, and friend templates that are defined within or associated with a class template.

**Technical Definition**
Member functions of class templates can be defined inside the class body (where they are implicitly inline) or outside the class body. When defined outside, each member function requires the `template <...>` prefix and the `ClassName<T>::` qualification; the template argument list appears twice (e.g., `MyStack<T, i>::MyStack`). A member template is a template declared inside a class or class template. A friend template is a friend declaration that names a function template or class template; a true template declaration must precede the friend declaration, as the friend declaration constitutes a use of the template, not a declaration.

**Beginner-Friendly Explanation**
Member functions of a class template are just like regular member functions, but when you define them outside the class, you have to repeat the template parameters. A member template is a function or class inside a class template that has its own template parameters — like a template inside a template. A friend template is a way to give a function or class access to the private members of a class template. You have to declare the friend template before you declare it as a friend.

---

### Purposes (all begin with "To")

- **To** define member functions of class templates inside or outside the class body.
- **To** create nested class templates and member function templates within a class template.
- **To** grant friendship to function templates, class templates, or their specializations.
- **To** enable operator overloading and stream insertion for class templates.
- **To** support generic algorithms that need access to private members.

---

### Syntax Rules and Structure

#### Complete General Syntaxes with Breakdowns

**Syntax 1 — Member function defined outside the class template**
```cpp
template <typename T, int N>
class MyStack {
public:
    MyStack();
    void push(const T& item);
    T& pop();
private:
    T buffer[N];
    int top_;
};

template <typename T, int N>
MyStack<T, N>::MyStack() : top_(0) {}

template <typename T, int N>
void MyStack<T, N>::push(const T& item) {
    buffer[top_++] = item;
}
```
- The template parameter list is repeated for each member function.
- The class name is qualified as `MyStack<T, N>::`.

**Syntax 2 — Member function template**
```cpp
template <typename T>
class X {
public:
    template <typename U>
    void mf(const U& u);
};

template <typename T>
template <typename U>
void X<T>::mf(const U& u) { }
```
- The outer template is `T`, the inner (member) template is `U`.
- Two `template` clauses are required.

**Syntax 3 — Friend function template**
```cpp
template <typename T>
class Array {
    T* data;
    int size;
public:
    template <typename U>
    friend Array<U>* combine(Array<U>& a1, Array<U>& a2);
};

template <typename T>
Array<T>* combine(Array<T>& a1, Array<T>& a2) { /* ... */ }
```
- The friend declaration names the function template.
- The friend function is defined outside the class template.

#### Syntax Rules

- Member functions defined inside the class body are implicitly inline.
- Member functions defined outside require `template <...>` and `ClassName<T>::`.
- Member templates can be function templates or nested class templates.
- A friend declaration does not declare a template; the template must be declared before the friend declaration.
- Friend specializations are allowed, but partial specializations cannot be friends.

#### Constraints and Limitations

- Member function templates cannot be virtual.
- Nested class templates cannot be defined inside local classes.
- Friend declarations cannot declare explicit specializations of templates.
- The definition of a friend function template must be visible to the compiler.

---

### Multiple Annotated Complete Step-by-Step Code Examples and Their Expected Outputs

#### Example 1 — Member Function Defined Outside the Class Template

```cpp
#include <iostream>
#include <string>

// Step 1: Class template with member function declarations
template <typename T, int N>
class FixedArray {
public:
    FixedArray();
    void set(int index, const T& value);
    T get(int index) const;
    int size() const { return N; }
private:
    T data_[N];
};

// Step 2: Constructor definition outside the class
template <typename T, int N>
FixedArray<T, N>::FixedArray() : data_{} {}

// Step 3: Member function definition outside the class
template <typename T, int N>
void FixedArray<T, N>::set(int index, const T& value) {
    if (index >= 0 && index < N) data_[index] = value;
}

template <typename T, int N>
T FixedArray<T, N>::get(int index) const {
    return (index >= 0 && index < N) ? data_[index] : T{};
}

int main() {
    // Step 4: Instantiate with int, size 5
    FixedArray<int, 5> arr;
    arr.set(0, 10);
    arr.set(1, 20);

    std::cout << "arr[0] = " << arr.get(0) << std::endl;
    std::cout << "arr[1] = " << arr.get(1) << std::endl;
    std::cout << "Size: " << arr.size() << std::endl;

    // Step 5: Instantiate with std::string, size 3
    FixedArray<std::string, 3> names;
    names.set(0, "Alice");
    std::cout << "names[0] = " << names.get(0) << std::endl;

    return 0;
}
```

**Expected Output:**
```
arr[0] = 10
arr[1] = 20
Size: 5
names[0] = Alice
```

**Why this output:** The class template `FixedArray` is instantiated with `int` and `std::string`. Each member function is defined outside the class with the `template <typename T, int N>` prefix and `FixedArray<T, N>::` qualification. The compiler generates the appropriate member functions for each instantiation. The `size()` function is defined inline inside the class and returns the non-type template parameter `N`.

---

#### Example 2 — Friend Function Template

```cpp
#include <iostream>

// Step 1: Forward declaration of the class template
template <typename T>
class Array;

// Step 2: Forward declaration of the friend function template
template <typename T>
Array<T>* combine(Array<T>& a1, Array<T>& a2);

// Step 3: Class template with friend declaration
template <typename T>
class Array {
    T* data_;
    int size_;
public:
    Array(int size) : size_(size), data_(new T[size]{}) {}
    ~Array() { delete[] data_; }

    T& operator[](int i) { return data_[i]; }
    int size() const { return size_; }

    // Step 4: Friend declaration
    friend Array<T>* combine<T>(Array<T>& a1, Array<T>& a2);
};

// Step 5: Definition of the friend function template
template <typename T>
Array<T>* combine(Array<T>& a1, Array<T>& a2) {
    Array<T>* result = new Array<T>(a1.size_ + a2.size_);
    for (int i = 0; i < a1.size_; ++i) (*result)[i] = a1[i];
    for (int i = 0; i < a2.size_; ++i) (*result)[a1.size_ + i] = a2[i];
    return result;
}

int main() {
    // Step 6: Use the friend function
    Array<int> a1(3), a2(2);
    a1[0] = 1; a1[1] = 2; a1[2] = 3;
    a2[0] = 4; a2[1] = 5;

    Array<int>* combined = combine(a1, a2);
    std::cout << "Combined size: " << combined->size() << std::endl;
    for (int i = 0; i < combined->size(); ++i) {
        std::cout << (*combined)[i] << " ";
    }
    std::cout << std::endl;

    delete combined;
    return 0;
}
```

**Expected Output:**
```
Combined size: 5
1 2 3 4 5 
```

**Why this output:** The friend function template `combine` is declared as a friend of `Array<T>`. Because it is a friend, it can access the private members `data_` and `size_`. The template must be forward-declared before the friend declaration. The `combine` function creates a new `Array<T>` and copies elements from both input arrays, producing a combined array of size 5.

---

### Real-World Cases with Explanation

**Case 1 — `std::vector` member functions:** `std::vector<T>` defines many member functions outside the class body (e.g., `push_back`, `resize`, `operator[]`), each with the `template <typename T>` prefix.

**Case 2 — Nested class templates:** `std::map` has nested class templates for iterators and allocators. These nested templates are declared inside the class template scope.

**Case 3 — Friend `operator<<`:** A common pattern is to declare `operator<<` as a friend of a class template to allow stream insertion, e.g., `template <typename T> std::ostream& operator<<(std::ostream&, const MyClass<T>&);`.

**Case 4 — Policy-based design:** Friend templates are used in policy-based design to allow policies to access private members of the host class template.

---

### References

- Member functions of class templates — Microsoft Learn - https://learn.microsoft.com/en-us/cpp/cpp/member-functions-cpp
- Member templates — cppreference.com - https://en.cppreference.com/w/cpp/language/member_template
- Friend declaration — cppreference.com - https://en.cppreference.com/w/cpp/language/friend
- Template friends — Microsoft Learn - https://learn.microsoft.com/en-us/cpp/cpp/template-friends

---

## 3. CTAD (Class Template Argument Deduction)

### Definitions

**Core Definition**
Class Template Argument Deduction (CTAD) is a C++17 feature that allows the compiler to deduce class template arguments from the types of constructor arguments, eliminating the need to explicitly specify template arguments.

**Technical Definition**
When a variable is declared with a class template name without explicit template arguments, the compiler performs CTAD by constructing a set of fictional function templates (deduction candidates) from the constructors of the class template. These candidates are used to deduce the template arguments. Since C++17, user-defined deduction guides can be provided to customize or supplement the implicit deduction guides. A deduction guide is a pattern that tells the compiler how to deduce class template arguments from constructor arguments.

**Beginner-Friendly Explanation**
Before C++17, you had to write `std::pair<int, double> p(2, 4.5);`. With CTAD, you can write `std::pair p(2, 4.5);` and the compiler figures out that `p` is `std::pair<int, double>`. Deduction guides let you customize this behavior: if you have a class template where the constructor arguments don't directly map to the template parameters, you can write a guide that tells the compiler how to deduce them.

---

### Purposes (all begin with "To")

- **To** eliminate redundant template argument specification in variable declarations.
- **To** reduce boilerplate and improve code readability.
- **To** allow class templates to be used with the same ease as `auto`.
- **To** provide custom deduction rules for constructors that don't directly map to template parameters.
- **To** support factory functions and wrapper types with clean syntax.

---

### Syntax Rules and Structure

#### Complete General Syntaxes with Breakdowns

**Syntax 1 — Implicit CTAD**
```cpp
std::pair p(2, 4.5);       // deduces std::pair<int, double>
std::tuple t(1, 2, 3.5);   // deduces std::tuple<int, int, double>
std::vector v{1, 2, 3};    // deduces std::vector<int>
```
- The compiler deduces template arguments from constructor arguments.

**Syntax 2 — User-defined deduction guide**
```cpp
template <typename T>
struct Wrapper {
    T value;
    Wrapper(T v) : value(v) {}
};

// Deduction guide
template <typename T>
Wrapper(T) -> Wrapper<T>;
```
- The guide tells the compiler that `Wrapper(T)` deduces `Wrapper<T>`.

**Syntax 3 — Deduction guide with `std::string`**
```cpp
template <typename T>
struct Box {
    T value;
    Box(T v) : value(v) {}
};

// Guide to convert const char* to std::string
Box(const char*) -> Box<std::string>;
```
- This guide ensures `Box("hello")` deduces `Box<std::string>`.

#### Syntax Rules

- CTAD applies only in variable declarations and function-style casts, not in function calls or template argument lists.
- Implicit deduction guides are generated from the constructors of the class template.
- User-defined deduction guides must be declared in the same scope as the class template.
- A deduction guide is a fictional function whose return type is the class template specialization.
- Deduction guides are not functions; they are not called, only used for deduction.

#### Constraints and Limitations

- CTAD requires C++17; not available in C++14 or earlier.
- CTAD does not apply to partial specializations directly; implicit guides are generated from the primary template.
- Deduction guides cannot be used to deduce non-type template parameters from constructor arguments unless they appear in the parameter list.
- CTAD can be ambiguous if multiple constructors are viable; explicit guides can resolve ambiguity.

---

### Multiple Annotated Complete Step-by-Step Code Examples and Their Expected Outputs

#### Example 1 — Implicit CTAD

```cpp
#include <iostream>
#include <utility>
#include <vector>
#include <tuple>

int main() {
    // Step 1: std::pair CTAD
    std::pair p(2, 4.5);  // deduces std::pair<int, double>
    std::cout << "pair: " << p.first << ", " << p.second << std::endl;

    // Step 2: std::tuple CTAD
    std::tuple t(1, 2, 3.5);  // deduces std::tuple<int, int, double>
    std::cout << "tuple: " << std::get<0>(t) << ", "
              << std::get<1>(t) << ", " << std::get<2>(t) << std::endl;

    // Step 3: std::vector CTAD from initializer list
    std::vector v{1, 2, 3};  // deduces std::vector<int>
    std::cout << "vector size: " << v.size() << std::endl;

    return 0;
}
```

**Expected Output:**
```
pair: 2, 4.5
tuple: 1, 2, 3.5
vector size: 3
```

**Why this output:** The compiler deduces template arguments from the constructor arguments. `std::pair p(2, 4.5)` deduces `std::pair<int, double>` because `2` is `int` and `4.5` is `double`. `std::tuple t(1, 2, 3.5)` deduces `std::tuple<int, int, double>`. `std::vector v{1, 2, 3}` deduces `std::vector<int>` from the initializer list.

---

#### Example 2 — User-Defined Deduction Guide

```cpp
#include <iostream>
#include <string>

// Step 1: Class template
template <typename T>
struct Box {
    T value;
    Box(T v) : value(v) {}
    void print() const { std::cout << "Box: " << value << std::endl; }
};

// Step 2: Deduction guide for const char*
Box(const char*) -> Box<std::string>;

int main() {
    // Step 3: Without the guide, Box("hello") would deduce Box<const char*>
    // With the guide, it deduces Box<std::string>
    Box b("hello");
    b.print();

    // Step 4: Other types work normally
    Box b2(42);
    b2.print();

    Box b3(3.14);
    b3.print();

    return 0;
}
```

**Expected Output:**
```
Box: hello
Box: 42
Box: 3.14
```

**Why this output:** Without the deduction guide, `Box("hello")` would deduce `T = const char*`, which is not what we want. The guide `Box(const char*) -> Box<std::string>` tells the compiler to deduce `T = std::string` when the argument is a `const char*`. The other instantiations (`int` and `double`) use the implicit guide from the constructor.

---

### Real-World Cases with Explanation

**Case 1 — `std::lock_guard`:** CTAD allows `std::lock_guard lck(mtx);` instead of `std::lock_guard<std::mutex> lck(mtx);`, making the code cleaner and less error-prone.

**Case 2 — `std::pair` and `std::tuple`:** CTAD eliminates the need for `std::make_pair` and `std::make_tuple` in many cases, e.g., `std::pair p(1, 2.5)`.

**Case 3 — Custom wrapper types:** A library that provides a `Wrapper<T>` class can add a deduction guide to handle special cases, such as converting string literals to `std::string`.

**Case 4 — Range constructors:** CTAD for `std::vector` deduces the element type from iterator pairs: `std::vector v(first, last)`.

---

### References

- Class template argument deduction (CTAD) — cppreference.com - https://en.cppreference.com/w/cpp/language/class_template_argument_deduction
- Deduction guides — cppreference.com - https://en.cppreference.com/w/cpp/language/class_template_deduction
- CTAD (C++17) — Microsoft Learn - https://learn.microsoft.com/en-us/cpp/cpp/class-template-argument-deduction

---

## 4. Component Grouping

### Definitions

**Core Definition**
Component grouping refers to the organization of template definitions: either header-only (definitions in headers) or split using explicit instantiation (`extern template`) to control where instantiation occurs.

**Technical Definition**
Template definitions must be visible at the point of instantiation. By default, this means defining templates in headers. However, when a template is used with a known set of types, an explicit instantiation definition (`template class ClassName<int>;`) can be placed in a `.cpp` file, and an explicit instantiation declaration (`extern template class ClassName<int>;`) can be placed in the header to suppress implicit instantiation in other translation units. This reduces compile times and code size by ensuring that the template is instantiated exactly once.

**Beginner-Friendly Explanation**
Normally, template code goes in headers so that every file that uses the template can see the definition. But this means the compiler generates the same template code in every file that uses it, which is wasteful. With `extern template`, you tell the compiler: "Don't generate this template here; it's already been generated in another file." This is like telling your friends, "Don't make another copy of the recipe; I already have one."

---

### Purposes (all begin with "To")

- **To** reduce compile times by preventing redundant template instantiation across translation units.
- **To** reduce binary size by ensuring a template is instantiated exactly once.
- **To** hide template implementation details by moving definitions to a `.cpp` file.
- **To** control which template instantiations are provided by a library.
- **To** enable faster incremental builds when templates change.

---

### Syntax Rules and Structure

#### Complete General Syntaxes with Breakdowns

**Syntax 1 — Header-only template (default)**
```cpp
// my_template.h
template <typename T>
class MyClass {
public:
    void do_something();
};

template <typename T>
void MyClass<T>::do_something() { /* ... */ }
```
- The definition is in the header; every TU that uses `MyClass` instantiates it.

**Syntax 2 — Explicit instantiation definition (`.cpp`)**
```cpp
// my_template.cpp
#include "my_template.h"
template class MyClass<int>;
template class MyClass<double>;
```
- Forces instantiation of `MyClass<int>` and `MyClass<double>` in this TU.

**Syntax 3 — Explicit instantiation declaration (`extern template`, header)**
```cpp
// my_template.h
extern template class MyClass<int>;
extern template class MyClass<double>;
```
- Suppresses implicit instantiation of these specializations in TUs that include this header.

#### Syntax Rules

- The `extern template` declaration must appear before any use of the template in the TU.
- The explicit instantiation definition must appear in exactly one TU.
- `extern template` only prevents instantiation of the class's non-inline, non-template member functions.
- Inline member functions and templates are still instantiated implicitly.
- Since C++11, `extern template` is standard; before C++11, it was a compiler extension.

#### Constraints and Limitations

- `extern template` is an optimization, not a semantic change; the program behaves the same with or without it.
- The explicit instantiation definition must be in a TU that is linked into the final program.
- `extern template` does not work for templates that are defined entirely in headers and have no out-of-line definitions.
- If the explicit instantiation definition is missing, the linker reports undefined references.

---

### Multiple Annotated Complete Step-by-Step Code Examples and Their Expected Outputs

#### Example 1 — Header-Only Template vs. `extern template`

```cpp
// File: heavy.h
#ifndef HEAVY_H
#define HEAVY_H

#include <vector>
#include <string>

template <typename T>
class Heavy {
public:
    void process();
    void analyze();
private:
    std::vector<T> data_;
};

// Step 1: Explicit instantiation declaration (extern template)
extern template class Heavy<int>;
extern template class Heavy<double>;

#endif
```

```cpp
// File: heavy.cpp
#include "heavy.h"
#include <iostream>

// Step 2: Member definitions
template <typename T>
void Heavy<T>::process() {
    std::cout << "Processing " << data_.size() << " items" << std::endl;
}

template <typename T>
void Heavy<T>::analyze() {
    std::cout << "Analyzing " << data_.size() << " items" << std::endl;
}

// Step 3: Explicit instantiation definition
template class Heavy<int>;
template class Heavy<double>;
```

```cpp
// File: main.cpp
#include "heavy.h"
#include <iostream>

int main() {
    // Step 4: No implicit instantiation occurs
    Heavy<int> h1;
    h1.process();
    h1.analyze();

    Heavy<double> h2;
    h2.process();

    return 0;
}
```

**Compile and run:**
```bash
g++ -std=c++17 heavy.cpp main.cpp -o app
./app
```

**Expected Output:**
```
Processing 0 items
Analyzing 0 items
Processing 0 items
```

**Why this output:** The header declares `extern template class Heavy<int>;` and `extern template class Heavy<double>;`, telling the compiler not to instantiate these specializations in `main.cpp`. The actual instantiation happens in `heavy.cpp` via the explicit instantiation definitions. The linker connects the calls in `main.cpp` to the instantiations in `heavy.cpp`. Without `extern template`, `main.cpp` would also instantiate `Heavy<int>` and `Heavy<double>`, wasting compile time.

---

### Real-World Cases with Explanation

**Case 1 — Standard library implementations:** libstdc++ and libc++ use `extern template` extensively for common instantiations like `std::vector<int>`, `std::string`, and `std::map<std::string, int>` to avoid instantiating them in every translation unit.

**Case 2 — Large template libraries:** Libraries like Boost and Eigen use explicit instantiation to control which template specializations are compiled and to reduce build times for users.

**Case 3 — Reducing binary size:** In embedded systems, `extern template` is used to ensure that only the required template instantiations are included in the firmware, reducing flash memory usage.

**Case 4 — Incremental builds:** When a template header changes, all TUs that instantiate it must be recompiled. With `extern template`, only the TU containing the explicit instantiation needs to be recompiled, significantly speeding up incremental builds.

---

### References

- Class template — cppreference.com (Explicit instantiation) - https://en.cppreference.com/w/cpp/language/class_template#Explicit_instantiation
- Extern templates — C++11 Language Extensions - https://isocpp.org/wiki/faq/cpp11-language-templates#extern-templates
- Explicit instantiation — Microsoft Learn - https://learn.microsoft.com/en-us/cpp/cpp/explicit-instantiation

---

## Summary Table of Core Concepts

| Concept | Key Mechanism | Primary Benefit | Key Constraint |
|---|---|---|---|
| Generic Blueprints | `template <typename T> class ClassName` | Type-agnostic classes, code reuse | Definitions must be visible at instantiation |
| Internal Signatures | Member functions outside class, member templates, friend templates | Fine-grained access control, nested generics | Friend templates must be declared before use |
| CTAD | Implicit + user-defined deduction guides | Eliminates redundant template arguments | C++17 required; ambiguity must be resolved |
| Component Grouping | `extern template` + explicit instantiation | Reduces compile time and binary size | Requires manual management of instantiations |

---

## References (Consolidated)

- Class template — cppreference.com - https://en.cppreference.com/w/cpp/language/class_template
- Type alias, alias template — cppreference.com - https://en.cppreference.com/w/cpp/language/type_alias
- Member templates — cppreference.com - https://en.cppreference.com/w/cpp/language/member_template
- Friend declaration — cppreference.com - https://en.cppreference.com/w/cpp/language/friend
- Class template argument deduction (CTAD) — cppreference.com - https://en.cppreference.com/w/cpp/language/class_template_argument_deduction
- Deduction guides — cppreference.com - https://en.cppreference.com/w/cpp/language/class_template_deduction
- Partial specialization — cppreference.com - https://en.cppreference.com/w/cpp/language/partial_specialization
- Aliases and typedefs (C++) — Microsoft Learn - https://learn.microsoft.com/en-us/cpp/cpp/aliases-and-typedefs-cpp
- Member functions of class templates — Microsoft Learn - https://learn.microsoft.com/en-us/cpp/cpp/member-functions-cpp
- Template friends — Microsoft Learn - https://learn.microsoft.com/en-us/cpp/cpp/template-friends
- CTAD (C++17) — Microsoft Learn - https://learn.microsoft.com/en-us/cpp/cpp/class-template-argument-deduction
- Explicit instantiation — Microsoft Learn - https://learn.microsoft.com/en-us/cpp/cpp/explicit-instantiation
- Extern templates — C++11 Language Extensions - https://isocpp.org/wiki/faq/cpp11-language-templates#extern-templates