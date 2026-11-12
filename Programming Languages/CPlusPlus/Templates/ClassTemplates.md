# Class Templates — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
A class template is a C++ construct that defines a **family of classes** parameterized by one or more template parameters, from which the compiler generates concrete classes when template arguments are supplied.

**Technical Definition**
A class template is a template that defines a family of classes. It is introduced by the `template` keyword followed by a template parameter list and a class definition. Each combination of template arguments produces a distinct **class specialization**. Member functions of a class template are instantiated only when used. Class templates may have **template member functions**, **static template member variables**, **nested templates**, and may be **fully or partially specialized**. Since C++17, class template argument deduction (CTAD) allows template arguments to be deduced from constructor arguments in some contexts.

**Beginner-Friendly Explanation**
A class template is a **generic class blueprint**. Instead of writing a `IntVector`, `DoubleVector`, and `StringVector`, you write **one** `Vector<T>` template. When you use `Vector<int>`, the compiler generates an `int` version; when you use `Vector<double>`, it generates a `double` version. Class templates are how C++ implements type-safe containers (`std::vector<T>`, `std::map<K,V>`) and generic data structures. They are the class-level counterpart of function templates: the same “write once, work with many types” idea, applied to entire classes.

### Key Characteristics

- **Generic classes**: One definition works with many types.
- **Distinct specializations**: `Vector<int>` and `Vector<double>` are unrelated types.
- **Lazy instantiation**: Member functions are instantiated only when used.
- **Template members**: Member functions and static members can themselves be templates.
- **Specialization**: Full and partial specializations customize behavior.
- **Nested templates**: Classes can contain nested templates.
- **CTAD (C++17)**: Template arguments can sometimes be deduced from constructors.
- **Header definitions**: Class template definitions must typically be visible at instantiation.

### Prerequisites

- Understanding of classes, member functions, and access control.
- Familiarity with function templates and template parameters.
- Knowledge of specialization and instantiation.
- Basic awareness of the standard library containers.

### Related Programming Areas

- **Standard library containers** – `std::vector<T>`, `std::map<K,V>`, `std::set<T>`.
- **Smart pointers** – `std::unique_ptr<T>`, `std::shared_ptr<T>`.
- **Type traits** – class templates specialized for type properties.
- **Policy-based design** – class templates parameterized by policies.
- **Metaprogramming** – class templates for compile-time computation.
- **Concepts (C++20)** – constrained class templates.

### Core Concepts / Features

1. Class Templates
2. Template Member Functions
3. Template Member Variables
4. Template Specialization
5. Partial Specialization
6. Nested Templates
7. Class Template Best Practices

---

## 1. Class Templates

### Definitions

**Core Definition**
A class template defines a family of classes parameterized by template parameters, from which concrete classes are generated when arguments are provided.

**Technical Definition**
A class template is declared with `template < parameter-list >` followed by a class definition. Each combination of template arguments produces a distinct class specialization. Member functions are instantiated only when used. The template definition must be visible at the point of instantiation. Since C++17, CTAD allows deduction of template arguments from constructor arguments.

**Beginner-Friendly Explanation**
A class template is a **generic class**. `Box<int>` and `Box<std::string>` are different classes generated from the same `Box<T>` template.

### Purposes

- To create type-safe generic containers.
- To implement generic data structures.
- To support policy-based design.
- To reduce code duplication across types.

### Syntax Rules and Structure

**General Syntax**
```cpp
template <typename T>
class ClassName {
    T member_;
public:
    ClassName(T value) : member_(value) {}
    T get() const { return member_; }
};
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `template <typename T>` | Template parameter list. |
| `class ClassName { ... }` | Class definition using `T`. |

**Syntax Rules**
- Template arguments must be provided (or deduced since C++17) when creating objects.
- Member functions defined outside the class need the template header and qualification.
- Each instantiation is a distinct type.
- Member functions are instantiated only when used.

**Constraints and Limitations**
- Different instantiations are unrelated types (no implicit conversion).
- Template definitions must typically be in headers.
- Member functions are instantiated lazily.

### Annotated Code Examples

**Example 1: Basic class template**

```cpp
#include <iostream>
#include <string>

template <typename T>
class Box {
    T value_;
public:
    explicit Box(T value) : value_(value) {}
    T get() const { return value_; }
};

int main() {
    Box<int> b1(42);
    Box<std::string> b2("hello");
    std::cout << b1.get() << ' ' << b2.get() << '\n';
}
```

**Expected Output**
```
42 hello
```

**Why this output?**  
`Box<int>` and `Box<std::string>` are distinct specializations; each stores and returns its respective type.

### Real-World Cases

- **`std::vector<T>`** – dynamic array.
- **`std::map<K, V>`** – associative container.
- **`std::unique_ptr<T>`** – smart pointer.

---

## 2. Template Member Functions

### Definitions

**Core Definition**
A template member function is a member function of a class (template or non-template) that is itself a function template.

**Technical Definition**
A member function can be a template: `template <typename U> void f(U value);`. If the enclosing class is also a template, the member template has its own template parameter list. Member templates are instantiated only when used. They can be defined inside or outside the class.

**Beginner-Friendly Explanation**
A template member function is a **generic function inside a class**. It can accept different types, even if the class itself is not a template.

### Purposes

- To provide generic operations within a class.
- To support conversion constructors and assignment operators.
- To enable generic accessors and mutators.

### Syntax Rules and Structure

**General Syntax**
```cpp
class X {
public:
    template <typename U>
    void f(U value);
};
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `template <typename U>` | Member template parameter list. |
| `void f(U)` | Member function using `U`. |

**Syntax Rules**
- Member templates can appear in class templates and non-template classes.
- They are instantiated only when used.
- Definitions outside the class need both template headers.

**Constraints and Limitations**
- Member templates cannot be virtual.
- They cannot be partially specialized (use overloading).

### Annotated Code Examples

**Example 1: Template member function**

```cpp
#include <iostream>

class Printer {
public:
    template <typename T>
    void print(const T& value) {
        std::cout << value << '\n';
    }
};

int main() {
    Printer p;
    p.print(42);
    p.print("hello");
}
```

**Expected Output**
```
42
hello
```

**Why this output?**  
`print` is a member template; `T` is deduced from the argument.

### Real-World Cases

- **Conversion constructors** – `template <typename U> X(const U&)`.
- **Generic accessors** – `template <typename F> void visit(F f)`.
- **`std::function`** – templated constructors.

---

## 3. Template Member Variables

### Definitions

**Core Definition**
A template member variable is a static data member of a class template whose type depends on the template parameters, or a variable template member (C++14).

**Technical Definition**
In a class template, static data members have types that depend on the class template parameters. They are defined outside the class with the template header. Variable templates (C++14) can also be members. Static data members of class templates are instantiated when used.

**Beginner-Friendly Explanation**
A template member variable is a **static member whose type depends on the class template**. For `Box<T>`, a static `T default_value;` has type `T`.

### Purposes

- To provide type-dependent static state.
- To define constants that depend on template parameters.
- To support compile-time configuration.

### Syntax Rules and Structure

**General Syntax**
```cpp
template <typename T>
class X {
public:
    static T value;   // static member of type T
};

template <typename T>
T X<T>::value = T{};
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `static T value;` | Static member depending on `T`. |
| Out-of-class definition | Required (or `inline` since C++17). |

**Syntax Rules**
- Static data members of class templates must be defined outside the class (or `inline`).
- Variable templates (C++14) can be used for type-dependent constants.

**Constraints and Limitations**
- Static data members of class templates are instantiated when used.
- `inline` (C++17) simplifies definitions.

### Annotated Code Examples

**Example 1: Static member of a class template**

```cpp
#include <iostream>

template <typename T>
class Counter {
public:
    static int count;
};

template <typename T>
int Counter<T>::count = 0;

int main() {
    Counter<int>::count = 5;
    Counter<double>::count = 10;
    std::cout << Counter<int>::count << ' ' << Counter<double>::count << '\n';
}
```

**Expected Output**
```
5 10
```

**Why this output?**  
Each specialization has its own static `count`.

### Real-World Cases

- **Type-dependent constants** – `static constexpr T zero = T{};`.
- **Instance counters** – per-specialization counters.
- **Type traits** – `std::is_integral<T>::value` (class template static member).

---

## 4. Template Specialization

### Definitions

**Core Definition**
Template specialization is the provision of a custom implementation for a specific set of template arguments, overriding the primary class template.

**Technical Definition**
Class templates support **full specialization** (`template <> class X<int> { ... };`) and **partial specialization** (`template <typename T> class X<T*> { ... };`). Full specialization provides a complete implementation for a specific argument list. Partial specialization provides a pattern for a subset of arguments.

**Beginner-Friendly Explanation**
Specialization is **customizing a class template for a specific type** (full) or a family of types (partial).

### Purposes

- To optimize for specific types.
- To provide correct behavior for types with special semantics.
- To customize library templates.

### Syntax Rules and Structure

**Full Specialization**
```cpp
template <>
class X<int> { /* custom */ };
```

**Partial Specialization**
```cpp
template <typename T>
class X<T*> { /* custom for pointers */ };
```

**Component Breakdown**

| Kind | Syntax |
|------|--------|
| Full | `template <> class X<int> { ... };` |
| Partial | `template <typename T> class X<T*> { ... };` |

**Syntax Rules**
- Full specialization uses `template <>`.
- Partial specialization uses a template parameter list with a pattern.
- Specializations must be visible before first use.

**Constraints and Limitations**
- Partial specialization must be more specialized than the primary template.
- Specializations must match the primary template's interface.

### Annotated Code Examples

**Example 1: Full specialization**

```cpp
#include <iostream>

template <typename T>
struct Printer {
    static void print(T value) { std::cout << value << '\n'; }
};

template <>
struct Printer<bool> {
    static void print(bool value) { std::cout << (value ? "true" : "false") << '\n'; }
};

int main() {
    Printer<int>::print(42);
    Printer<bool>::print(true);
}
```

**Expected Output**
```
42
true
```

**Why this output?**  
The full specialization for `bool` prints `"true"`.

### Real-World Cases

- **`std::hash<T>`** – specializations for different types.
- **`std::numeric_limits<T>`** – customized for each type.
- **Type traits** – specializations for categories.

---

## 5. Partial Specialization

### Definitions

**Core Definition**
Partial specialization provides a class template implementation for a **subset of template arguments** that match a pattern.

**Technical Definition**
Partial specialization is allowed for class templates (and variable templates since C++14). It uses a template parameter list with a pattern (e.g., `T*`, `T&`, `std::vector<T>`) and provides a customized implementation when the arguments match that pattern.

**Beginner-Friendly Explanation**
Partial specialization is **customizing for a family of types**, like “all pointers” or “all `std::vector`s.”

### Purposes

- To customize behavior for categories of types.
- To optimize for pointer types, references, or template instantiations.
- To implement type traits.

### Syntax Rules and Structure

**General Syntax**
```cpp
template <typename T>
class X<T*> { ... };   // specialization for all pointer types
```

**Component Breakdown**

| Pattern | Meaning |
|---------|---------|
| `T*` | All pointers. |
| `T&` | All references. |
| `std::vector<T>` | All vectors. |

**Syntax Rules**
- Only for class and variable templates.
- The pattern must be more specialized than the primary template.
- Template parameters are deduced from the pattern.

**Constraints and Limitations**
- Ambiguity can arise with multiple matching partial specializations.
- Must be visible before first use.

### Annotated Code Examples

**Example 1: Partial specialization for pointers**

```cpp
#include <iostream>

template <typename T>
struct TypeInfo {
    static void print() { std::cout << "general\n"; }
};

template <typename T>
struct TypeInfo<T*> {
    static void print() { std::cout << "pointer\n"; }
};

int main() {
    TypeInfo<int>::print();
    TypeInfo<int*>::print();
}
```

**Expected Output**
```
general
pointer
```

**Why this output?**  
The partial specialization `TypeInfo<T*>` matches pointer types.

### Real-World Cases

- **`std::is_pointer<T>`** – partial specialization for `T*`.
- **`std::remove_reference<T>`** – partial specialization for `T&`.
- **Smart pointer traits** – customizing for pointer categories.

---

## 6. Nested Templates

### Definitions

**Core Definition**
A nested template is a template (class or function) declared inside another class or class template.

**Technical Definition**
Class templates can contain nested class templates, nested member templates, and nested alias templates. Nested templates have access to the enclosing template's parameters (since C++11, nested classes have access to enclosing class members). They are instantiated when used.

**Beginner-Friendly Explanation**
A nested template is a **template inside a class**. For example, a container might have a nested `iterator` template.

### Purposes

- To organize related templates.
- To provide nested generic helpers.
- To implement iterators and proxies.

### Syntax Rules and Structure

**General Syntax**
```cpp
template <typename T>
class Outer {
public:
    template <typename U>
    class Inner {
        // uses T and U
    };
};
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `Outer<T>` | Enclosing class template. |
| `Inner<U>` | Nested class template. |

**Syntax Rules**
- Nested templates can use the enclosing template's parameters.
- They are instantiated when used.
- They can be defined outside the enclosing class with proper qualification.

**Constraints and Limitations**
- Nested templates cannot be partially specialized independently.
- Access to enclosing members follows normal rules.

### Annotated Code Examples

**Example 1: Nested class template**

```cpp
#include <iostream>

template <typename T>
class Outer {
public:
    template <typename U>
    class Inner {
    public:
        void show() { std::cout << "inner\n"; }
    };
};

int main() {
    Outer<int>::Inner<double> inner;
    inner.show();
}
```

**Expected Output**
```
inner
```

**Why this output?**  
`Inner<double>` is a nested template inside `Outer<int>`.

### Real-World Cases

- **Container iterators** – nested iterator templates.
- **`std::map`** – nested value types.
- **Proxy classes** – nested templates for proxies.

---

## 7. Class Template Best Practices

### Definitions

**Core Definition**
Class template best practices are guidelines for writing class templates that are correct, readable, efficient, and maintainable.

**Technical Definition**
The C++ Core Guidelines provide template rules (T.1–T.13, T.40–T.83). Key principles: use class templates for generic data structures; constrain templates with concepts (C++20); avoid over-templating; document specializations; use CTAD where helpful (C++17); and keep template definitions in headers.

**Beginner-Friendly Explanation**
Best practices for class templates are simple: **use templates for truly generic classes**, **constrain them with concepts**, **avoid over-templating**, **document specializations**, and **keep definitions in headers**.

### Key Guidelines (Summary Table)

| Guideline | Summary | Recommendation |
|-----------|---------|----------------|
| **Use for generic classes** | Templates are for families of types. | Don't template one-off classes. |
| **Constrain with concepts** | Concepts (C++20) express requirements. | `template <std::integral T>`. |
| **Avoid over-templating** | Over-templating hurts readability and errors. | Use concrete types when possible. |
| **Header definitions** | Templates typically belong in headers. | Header-only libraries. |
| **Document specializations** | Specializations are commitments. | Comment customizations. |
| **Use CTAD** | Deduce template arguments (C++17). | `std::pair p{1, 2.0};`. |

### Annotated Code Examples

**Example 1: Constrained class template (C++20)**

```cpp
#include <iostream>
#include <concepts>

template <std::integral T>
class Counter {
    T value_ = 0;
public:
    void increment() { ++value_; }
    T value() const { return value_; }
};

int main() {
    Counter<int> c;
    c.increment();
    std::cout << c.value() << '\n';
    // Counter<double> d;   // error: double is not integral
}
```

**Expected Output**
```
1
```

**Why this is best practice?**  
The `std::integral` concept expresses the requirement and produces a readable error.

### Real-World Cases

- **Standard library** – containers constrained with concepts in C++20.
- **Code review** – concepts and clear constraints are preferred.
- **Static analysis** – tools check template usage.

---

## References

- Class templates (cppreference) – https://en.cppreference.com/w/cpp/language/class_template
- Templates (cppreference) – https://en.cppreference.com/w/cpp/language/templates
- Member templates (cppreference) – https://en.cppreference.com/w/cpp/language/member_template
- Partial specialization (cppreference) – https://en.cppreference.com/w/cpp/language/partial_specialization
- Explicit specialization (cppreference) – https://en.cppreference.com/w/cpp/language/template_specialization
- Class template argument deduction (cppreference) – https://en.cppreference.com/w/cpp/language/class_template_argument_deduction
- C++ Core Guidelines (templates section) – https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines
- Microsoft Learn — Class templates (C++) – https://learn.microsoft.com/en-us/cpp/cpp/class-templates
- Microsoft Learn — Member templates – https://learn.microsoft.com/en-us/cpp/cpp/member-templates

---

This cheat sheet provides a complete, structured reference for C++ class templates, covering class templates, template member functions, template member variables, full and partial specialization, nested templates, and best practices. Each section follows a uniform template with definitions, purposes, syntax rules, annotated examples, expected outputs, and authoritative references.