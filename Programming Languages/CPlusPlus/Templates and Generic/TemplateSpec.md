# C++ Template Specialization — Comprehensive Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
Template specialization is a C++ feature that allows you to provide a custom implementation of a template for a specific set of template arguments, overriding the generic behavior defined by the primary template.

**Technical Definition**
When template arguments are provided (or, for function and class templates, deduced), they are substituted for the template parameters to obtain a specialization of the template — a specific type or a specific function lvalue. Specializations may also be provided explicitly: **full (explicit) specializations** are allowed for class, variable (since C++14), and function templates, while **partial specializations** are only allowed for class templates and variable templates (since C++14). A full specialization is declared with an empty template parameter list (`template <>`), while a partial specialization is declared with a non-empty parameter list that restricts a subset of the primary template's arguments.

**Beginner-Friendly Explanation**
Imagine you have a generic "tool" that works for all materials — wood, metal, plastic. That's the primary template. But for one specific material — say, glass — the generic tool doesn't work well. Template specialization lets you build a special version of the tool just for glass. The generic version still handles everything else. In C++, you write a template once, then provide specialized versions for specific types where the generic version isn't good enough.

---

### Key Characteristics

- **Full specialization**: Overrides the template for an exact set of arguments; available for class, function, and variable templates.
- **Partial specialization**: Restricts the template for a category of types (e.g., all pointers); available only for class and variable templates.
- **Selection priority**: Full specialization beats partial specialization, which beats the primary template.
- **Declaration order**: Specializations must be declared after the primary template and before their first use.
- **Namespace restriction**: Full specializations must be declared in the namespace of which the template is a member.
- **No partial specialization for function templates**: Function templates can only be fully specialized; use overloading instead for partial customization.

---

### Prerequisites

- Solid understanding of function templates and class templates.
- Familiarity with template parameter lists and instantiation.
- Knowledge of the One Definition Rule (ODR) and translation units.
- (For C++14 variable templates) A compiler supporting C++14 or later.

---

### Related Programming Areas

- **Function templates** (full specialization only).
- **Class templates** (both full and partial specialization).
- **Variable templates** (C++14, both full and partial specialization).
- **Template metaprogramming** (type traits, policy-based design).
- **Overload resolution** (interaction between specializations and overloads).

---

### Core Concepts / Features

1. **Absolute Overrides** — Implementing Full (Explicit) Specialization to override behaviors for specific concrete types.
2. **Conditional Modifications** — Applying Partial Specialization to restrict templates by subsets (e.g., pointers `T*`, arrays `T[]`, or specific qualifiers).
3. **Resolution Mechanics** — Understanding matching rules, lookup hierarchies, and avoiding specialization ambiguity errors.

---

## 1. Absolute Overrides: Full (Explicit) Specialization

### Definitions

**Core Definition**
Full (explicit) specialization provides a complete, concrete implementation of a template for a specific set of template arguments, replacing the generic template code entirely for those arguments.

**Technical Definition**
A full specialization is declared with `template <>` followed by the class, function, or variable declaration with all template arguments specified. Any of the following can be fully specialized: function templates, class templates, variable templates (C++14), member functions of class templates, static data members of class templates, member classes of class templates, member enumerations of class templates, member class templates, and member function templates. An explicit specialization shall be declared in the namespace of which the template is a member.

**Beginner-Friendly Explanation**
A full specialization is like saying: "For this one exact type, use this completely different code." For example, you might have a template `Printer<T>` that prints any value. But for `Printer<bool>`, you want it to print "true" or "false" instead of "1" or "0". You write a full specialization for `Printer<bool>` that replaces the generic implementation entirely.

---

### Purposes (all begin with "To")

- **To** provide a completely different implementation for a specific type.
- **To** optimize performance for particular types (e.g., using bitwise operations for `bool`).
- **To** handle types that the generic template cannot support (e.g., C-style strings).
- **To** customize behavior for user-defined types in standard library templates (e.g., `std::hash`).
- **To** override template behavior for a concrete type without affecting other instantiations.

---

### Syntax Rules and Structure

#### Complete General Syntax with Breakdown

```cpp
template <>
class ClassName<SpecificType> {
    // specialized implementation
};
```

**Component breakdown:**
- `template <>` — empty template parameter list; signals a full specialization.
- `class ClassName<SpecificType>` — the class name with the specific argument.
- `{ ... }` — the specialized class body, which may differ entirely from the primary template.

**Function template full specialization:**
```cpp
template <>
ReturnType functionName<SpecificType>(Parameters) {
    // specialized implementation
}
```

#### Syntax Rules

- The `template <>` prefix is mandatory for full specializations.
- The specialization must be declared in the same namespace as the primary template.
- A full specialization of a function template is `inline` only if it is declared with the `inline` specifier; it does not inherit `inline` from the primary template.
- A full specialization can have a completely different set of members and member functions than the primary template.
- Member functions of a specialized class template are defined without the `template <>` syntax.

#### Constraints and Limitations

- Full specialization of a function template does not participate in overload resolution; it is selected only after a function template is chosen by overload resolution.
- A full specialization must be declared before its first use in the translation unit.
- Explicit specializations cannot be declared inside a class scope for member templates (except since CWG 727, which allows it in any scope where the primary template may appear).
- A full specialization of a member function of a class template is implicitly `inline` only if declared `inline`.

---

### Multiple Annotated Complete Step-by-Step Code Examples and Their Expected Outputs

#### Example 1 — Full Specialization of a Class Template

```cpp
#include <iostream>
#include <string>

// Step 1: Primary template
template <typename T>
class Printer {
public:
    void print(const T& value) {
        std::cout << "Generic: " << value << std::endl;
    }
};

// Step 2: Full specialization for bool
template <>
class Printer<bool> {
public:
    void print(bool value) {
        std::cout << "Boolean: " << (value ? "true" : "false") << std::endl;
    }
};

int main() {
    // Step 3: Use the primary template for int
    Printer<int> intPrinter;
    intPrinter.print(42);          // Generic

    // Step 4: Use the full specialization for bool
    Printer<bool> boolPrinter;
    boolPrinter.print(true);       // Boolean: true
    boolPrinter.print(false);      // Boolean: false

    return 0;
}
```

**Expected Output:**
```
Generic: 42
Boolean: true
Boolean: false
```

**Why this output:** The primary template handles all types generically. When `Printer<bool>` is instantiated, the compiler selects the full specialization, which prints "Boolean:" instead of "Generic:" and converts the boolean to "true"/"false". The specialization completely replaces the primary template's implementation for `bool`.

---

#### Example 2 — Full Specialization of a Function Template

```cpp
#include <iostream>
#include <cstring>

// Step 1: Primary function template
template <typename T>
T max_value(T a, T b) {
    std::cout << "Generic max" << std::endl;
    return (a > b) ? a : b;
}

// Step 2: Full specialization for const char*
template <>
const char* max_value<const char*>(const char* a, const char* b) {
    std::cout << "String max" << std::endl;
    return (std::strcmp(a, b) > 0) ? a : b;
}

int main() {
    // Step 3: Generic for int
    std::cout << max_value(3, 7) << std::endl;

    // Step 4: Specialized for const char*
    std::cout << max_value("apple", "banana") << std::endl;

    return 0;
}
```

**Expected Output:**
```
Generic max
7
String max
banana
```

**Why this output:** The primary template uses `operator>` for comparison, which compares `const char*` pointers, not string contents. The full specialization uses `std::strcmp` to compare the strings lexicographically. The compiler selects the specialization when the argument type is `const char*`.

---

### Real-World Cases with Explanation

**Case 1 — `std::hash` specializations:** The standard library provides a primary template `std::hash<T>` and full specializations for `int`, `std::string`, and other types. Users can provide full specializations for their own types to enable their use in `std::unordered_map` and `std::unordered_set`.

**Case 2 — `std::numeric_limits`:** The primary template `std::numeric_limits<T>` is specialized for each fundamental type (`int`, `float`, `double`, etc.) to provide type-specific properties like `min()`, `max()`, and `epsilon()`.

**Case 3 — `std::formatter`:** The `<format>` header uses full specializations of `std::formatter<T>` for each type that can be formatted, allowing users to add custom formatting by specializing the template.

**Case 4 — `Array<bool>` bit-packing:** A common textbook example where `Array<bool>` is fully specialized to pack each boolean into a single bit, reducing memory usage by 8x compared to the generic `Array<T>` that stores one `bool` per element.

---

### References

- Explicit (full) template specialization — cppreference.com - https://en.cppreference.com/w/cpp/language/template_specialization
- Explicit Specialization of Class Templates — Microsoft Learn - https://learn.microsoft.com/en-us/cpp/cpp/explicit-specialization-of-class-templates
- Using template specialization in C++ — Microsoft Learn (archive) - https://learn.microsoft.com/en-us/archive/blogs/the1/using-template-specialization-in-c

---

## 2. Conditional Modifications: Partial Specialization

### Definitions

**Core Definition**
Partial specialization allows you to customize a class template for a category of template arguments (e.g., all pointer types, all array types, or a specific combination of arguments), rather than a single concrete type.

**Technical Definition**
Partial specialization is declared with a non-empty template parameter list and an argument list that restricts the primary template. It allows customizing class and variable (since C++14) templates for a given category of template arguments. Partial specialization may be declared in any scope where its primary template may be defined, and it must appear after the non-specialized template declaration. Partial specializations are only allowed for class templates and variable templates — not for function templates.

**Beginner-Friendly Explanation**
Think of partial specialization as a "family" of specializations. Instead of specializing for one exact type (like `int`), you specialize for a whole category — like "all pointers" (`T*`) or "all arrays" (`T[]`). For example, you might have a generic `Container<T>` and a partial specialization `Container<T*>` that handles pointer types differently, dereferencing before storing.

---

### Purposes (all begin with "To")

- **To** customize behavior for a category of types (e.g., all pointers, all references, all arrays).
- **To** specialize only some of a template's parameters while leaving others generic.
- **To** optimize memory layout or algorithms for specific type categories.
- **To** provide different implementations based on type qualifiers (`const`, `volatile`).
- **To** enable type trait implementations that depend on type properties.

---

### Syntax Rules and Structure

#### Complete General Syntax with Breakdown

```cpp
template <typename T>
class ClassName<T*> {
    // specialization for pointer types
};
```

**Component breakdown:**
- `template <typename T>` — non-empty parameter list; `T` is the deduced parameter.
- `class ClassName<T*>` — the class name with a pattern (`T*`) rather than a concrete type.
- `{ ... }` — the specialized implementation.

**Multiple argument partial specialization:**
```cpp
template <typename T1, typename T2, int I>
class A {};  // primary

template <typename T, int I>
class A<T, T*, I> {};  // partial: T2 is a pointer to T1
```

#### Syntax Rules

- The argument list cannot be identical to the non-specialized argument list (it must specialize something).
- Default arguments cannot appear in the argument list of a partial specialization.
- If any argument is a pack expansion, it must be the last argument in the list.
- Non-type argument expressions can use template parameters as long as the parameter appears at least once outside a non-deduced context.
- A non-type template argument cannot specialize a template parameter whose type depends on a parameter of the specialization.

#### Constraints and Limitations

- Partial specialization is **not allowed for function templates**. Use overloading or `if constexpr` instead.
- A partial specialization must be more specialized than the primary template.
- The `std::unique_ptr` class template has a partial specialization for array types (`T[]`).
- Partial specializations can be declared at class scope or namespace scope for member templates.

---

### Multiple Annotated Complete Step-by-Step Code Examples and Their Expected Outputs

#### Example 1 — Partial Specialization for Pointer Types

```cpp
#include <iostream>

// Step 1: Primary template
template <typename T>
class Wrapper {
public:
    void describe() {
        std::cout << "Generic wrapper" << std::endl;
    }
};

// Step 2: Partial specialization for pointer types
template <typename T>
class Wrapper<T*> {
public:
    void describe() {
        std::cout << "Pointer wrapper" << std::endl;
    }
};

// Step 3: Partial specialization for const pointer types
template <typename T>
class Wrapper<const T*> {
public:
    void describe() {
        std::cout << "Const pointer wrapper" << std::endl;
    }
};

int main() {
    // Step 4: Primary template for int
    Wrapper<int> w1;
    w1.describe();        // Generic wrapper

    // Step 5: Partial for int*
    Wrapper<int*> w2;
    w2.describe();        // Pointer wrapper

    // Step 6: Partial for const int*
    Wrapper<const int*> w3;
    w3.describe();        // Const pointer wrapper

    return 0;
}
```

**Expected Output:**
```
Generic wrapper
Pointer wrapper
Const pointer wrapper
```

**Why this output:** The primary template is used for `Wrapper<int>`. The partial specialization `Wrapper<T*>` matches `Wrapper<int*>`, and `Wrapper<const T*>` matches `Wrapper<const int*>`. The compiler selects the most specialized match, demonstrating the partial ordering of class template specializations.

---

#### Example 2 — Partial Specialization for Array Types

```cpp
#include <iostream>

// Step 1: Primary template
template <typename T>
class Container {
public:
    void info() {
        std::cout << "Generic container" << std::endl;
    }
};

// Step 2: Partial specialization for array types
template <typename T, std::size_t N>
class Container<T[N]> {
public:
    void info() {
        std::cout << "Array container of size " << N << std::endl;
    }
};

int main() {
    // Step 3: Primary for double
    Container<double> c1;
    c1.info();              // Generic container

    // Step 4: Partial for int[5]
    Container<int[5]> c2;
    c2.info();              // Array container of size 5

    return 0;
}
```

**Expected Output:**
```
Generic container
Array container of size 5
```

**Why this output:** The partial specialization `Container<T[N]>` matches any array type, deducing `T` as the element type and `N` as the array size. When `Container<int[5]>` is instantiated, the compiler matches the partial specialization and uses its `info()` implementation.

---

### Real-World Cases with Explanation

**Case 1 — `std::unique_ptr<T[]>`:** The standard library provides a partial specialization of `std::unique_ptr` for array types (`T[]`), which uses `delete[]` instead of `delete` and provides `operator[]` instead of `operator*` and `operator->`.

**Case 2 — Type traits:** Many type traits in `<type_traits>` use partial specialization. For example, `std::is_pointer<T>` is implemented with a primary template inheriting from `false_type` and a partial specialization for `T*` inheriting from `true_type`.

**Case 3 — Policy-based design:** Partial specialization enables policies to be selected based on type properties, e.g., `template <typename T> class Storage<T, true>` for trivially copyable types and `Storage<T, false>` for non-trivial types.

**Case 4 — Iterator category dispatch:** Standard library algorithms use partial specialization (via tag dispatch) to select the most efficient algorithm based on the iterator category (e.g., `std::random_access_iterator_tag` vs. `std::forward_iterator_tag`).

---

### References

- Partial template specialization — cppreference.com - https://en.cppreference.com/w/cpp/language/partial_specialization
- Partial Specialization of Class Templates — Microsoft Learn - https://learn.microsoft.com/en-us/cpp/cpp/partial-specialization-of-class-templates
- Understand partial and full template specialization — Dr-Sergey/learn_cpp (GitHub) - https://github.com/Dr-Sergey/learn_cpp

---

## 3. Resolution Mechanics: Matching Rules and Lookup Hierarchies

### Definitions

**Core Definition**
Resolution mechanics describe how the compiler selects the correct template specialization when multiple candidates (full, partial, and primary) match a given set of template arguments, and how ambiguities are detected.

**Technical Definition**
When a class template specialization is referenced, the compiler first considers full specializations, then partial specializations, then the primary template. If multiple partial specializations match, partial ordering rules determine which is more specialized. A partial specialization A is more specialized than B if A's argument list can be deduced from B's, but not vice versa. If neither is more specialized than the other, the program is ill-formed due to ambiguity. For function templates, overload resolution selects a template, and then an explicit specialization of that template is used if one exists.

**Beginner-Friendly Explanation**
Imagine you have three keys for a lock: a master key (full specialization), a skeleton key (partial specialization), and a generic key (primary template). The compiler tries the most specific key first. If two keys both fit equally well, the compiler can't decide — that's an ambiguity error. The rule is simple: **full beats partial, partial beats primary, and more specialized beats less specialized**.

---

### Purposes (all begin with "To")

- **To** understand which specialization is selected when multiple candidates exist.
- **To** avoid ambiguity errors by writing unambiguous specializations.
- **To** design specialization hierarchies that are predictable and maintainable.
- **To** resolve conflicts between full, partial, and primary templates.
- **To** apply partial ordering rules to determine the most specialized match.

---

### Syntax Rules and Structure

#### Resolution Priority Hierarchy

| Priority | Type | Example |
|---|---|---|
| 1 (highest) | Full specialization | `template <> class A<int, int> {}` |
| 2 | Partial specialization (more specialized) | `template <typename T> class A<T, T> {}` |
| 3 | Partial specialization (less specialized) | `template <typename T> class A<T, int> {}` |
| 4 (lowest) | Primary template | `template <typename T, typename U> class A {}` |

#### Partial Ordering Rules (for Class Templates)

1. If multiple partial specializations match, the compiler applies partial ordering.
2. A partial specialization A is more specialized than B if A's argument list can be deduced from B's, but B's cannot be deduced from A's.
3. If neither is more specialized, the program is ill-formed (ambiguous).

#### Partial Ordering Rules (for Function Templates)

1. Overload resolution selects the best function template.
2. After a template is selected, an explicit specialization of that template is used if one exists.
3. If two function templates are equally specialized, the call is ambiguous.
4. A template taking `T*` is more specialized than one taking `T`, because a hypothetical type `X*` is a valid argument for `T` but `X` is not a valid argument for `T*`.

#### Constraints and Limitations

- Specializations must be declared before their first use.
- A specialization declared after an instantiation that would have used it is ill-formed (no diagnostic required if the specialization is declared after the point of instantiation).
- Full specializations of function templates do not participate in overload resolution; they are selected after a template is chosen.
- Ambiguity errors are detected at instantiation time, not at definition time.

---

### Multiple Annotated Complete Step-by-Step Code Examples and Their Expected Outputs

#### Example 1 — Full vs. Partial vs. Primary Selection

```cpp
#include <iostream>

// Step 1: Primary template
template <typename T, typename U>
class Pair {
public:
    void describe() { std::cout << "Primary<T,U>" << std::endl; }
};

// Step 2: Partial specialization: same types
template <typename T>
class Pair<T, T> {
public:
    void describe() { std::cout << "Partial<T,T>" << std::endl; }
};

// Step 3: Full specialization: int, int
template <>
class Pair<int, int> {
public:
    void describe() { std::cout << "Full<int,int>" << std::endl; }
};

int main() {
    // Step 4: Primary for different types
    Pair<int, double> p1;
    p1.describe();          // Primary<T,U>

    // Step 5: Partial for same types (not int, int)
    Pair<double, double> p2;
    p2.describe();          // Partial<T,T>

    // Step 6: Full for int, int
    Pair<int, int> p3;
    p3.describe();          // Full<int,int>

    return 0;
}
```

**Expected Output:**
```
Primary<T,U>
Partial<T,T>
Full<int,int>
```

**Why this output:** The compiler selects the most specialized match. `Pair<int, double>` matches only the primary template. `Pair<double, double>` matches the partial specialization `Pair<T, T>` (more specialized than the primary). `Pair<int, int>` matches the full specialization (most specialized). The selection priority is clear: Full > Partial > Primary.

---

#### Example 2 — Partial Ordering with Function Templates

```cpp
#include <iostream>

// Step 1: Less specialized template
template <typename T>
void f(T) {
    std::cout << "Less specialized (T)" << std::endl;
}

// Step 2: More specialized template
template <typename T>
void f(T*) {
    std::cout << "More specialized (T*)" << std::endl;
}

// Step 3: Even more specialized template
template <typename T>
void f(const T*) {
    std::cout << "Even more specialized (const T*)" << std::endl;
}

int main() {
    int i = 0;
    const int j = 0;
    int* pi = &i;
    const int* cpi = &j;

    // Step 4: Overload resolution selects the best match
    f(i);       // T = int
    f(pi);      // T = int
    f(cpi);     // T = int

    return 0;
}
```

**Expected Output:**
```
Less specialized (T)
More specialized (T*)
Even more specialized (const T*)
```

**Why this output:** Partial ordering determines that `f(T*)` is more specialized than `f(T)`, and `f(const T*)` is more specialized than `f(T*)`. The compiler selects the most specialized template that matches the argument. For `f(i)`, only `f(T)` matches. For `f(pi)`, both `f(T)` and `f(T*)` match, but `f(T*)` is more specialized. For `f(cpi)`, all three match, but `f(const T*)` is the most specialized.

---

#### Example 3 — Avoiding Ambiguity with Partial Specializations

```cpp
#include <iostream>

// Primary template
template <typename T, typename U>
class X {
public:
    void describe() { std::cout << "Primary" << std::endl; }
};

// Partial specialization #1: T* and U
template <typename T, typename U>
class X<T*, U> {
public:
    void describe() { std::cout << "T*, U" << std::endl; }
};

// Partial specialization #2: T and U*
template <typename T, typename U>
class X<T, U*> {
public:
    void describe() { std::cout << "T, U*" << std::endl; }
};

int main() {
    X<int, double> a;       // Primary
    a.describe();

    X<int*, double> b;      // Partial #1
    b.describe();

    X<int, double*> c;      // Partial #2
    c.describe();

    // X<int*, double*> d;  // AMBIGUOUS! Both partials match equally.
    // d.describe();        // ERROR: ambiguous partial specialization

    return 0;
}
```

**Expected Output:**
```
Primary
T*, U
T, U*
```

**Why this output:** `X<int*, double*>` matches both partial specializations equally well (neither is more specialized than the other). The compiler reports an ambiguity error. This is a classic example of how to accidentally create ambiguous partial specializations. The fix is to add a third specialization that explicitly handles the `T*, U*` case.

---

### Real-World Cases with Explanation

**Case 1 — `std::vector<bool>`:** The standard library provides a full specialization of `std::vector<bool>` that packs bits to save memory. This is the most famous example of full specialization in the standard library.

**Case 2 — Type traits resolution:** `std::is_pointer<T>` uses a primary template inheriting from `false_type` and a partial specialization for `T*` inheriting from `true_type`. The partial ordering ensures that pointer types are correctly identified.

**Case 3 — `std::hash` ambiguity:** If a user provides a full specialization of `std::hash<MyType>` and also a partial specialization that matches `MyType`, the full specialization is selected. If two partial specializations both match, the program is ill-formed.

**Case 4 — Iterator category dispatch:** Standard algorithms use partial ordering of function templates to select the most efficient implementation based on iterator category. For example, `std::advance` uses `std::random_access_iterator_tag` to enable constant-time advancement.

---

### References

- Partial ordering of function templates (C++) — Microsoft Learn - https://learn.microsoft.com/en-us/cpp/cpp/partial-ordering-of-function-templates-cpp
- Partial template specialization — cppreference.com (Partial ordering) - https://en.cppreference.com/w/cpp/language/partial_specialization
- Compiler Error C2752: more than one partial specialization matches — Microsoft Learn - https://learn.microsoft.com/en-us/cpp/error-messages/compiler-errors-2/compiler-error-c2752

---

## Summary Table of Core Concepts

| Concept | Key Mechanism | Primary Benefit | Key Constraint |
|---|---|---|---|
| Full Specialization | `template <> class X<Type>` | Exact override for specific types | Must be declared in same namespace; not considered during overload resolution for functions |
| Partial Specialization | `template <typename T> class X<T*>` | Category-based customization | Not allowed for function templates |
| Resolution Mechanics | Full > Partial > Primary | Predictable selection order | Ambiguity if two partials are equally specialized |

---

## References (Consolidated)

- Explicit (full) template specialization — cppreference.com - https://en.cppreference.com/w/cpp/language/template_specialization
- Partial template specialization — cppreference.com - https://en.cppreference.com/w/cpp/language/partial_specialization
- Templates — cppreference.com - https://en.cppreference.com/w/cpp/language/templates
- Member templates — cppreference.com - https://en.cppreference.com/w/cpp/language/member_template
- Explicit Specialization of Class Templates — Microsoft Learn - https://learn.microsoft.com/en-us/cpp/cpp/explicit-specialization-of-class-templates
- Partial Specialization of Class Templates — Microsoft Learn - https://learn.microsoft.com/en-us/cpp/cpp/partial-specialization-of-class-templates
- Partial ordering of function templates (C++) — Microsoft Learn - https://learn.microsoft.com/en-us/cpp/cpp/partial-ordering-of-function-templates-cpp
- Using template specialization in C++ — Microsoft Learn (archive) - https://learn.microsoft.com/en-us/archive/blogs/the1/using-template-specialization-in-c
- Understand partial and full template specialization — Dr-Sergey/learn_cpp (GitHub) - https://github.com/Dr-Sergey/learn_cpp
- Compiler Error C2752 — Microsoft Learn - https://learn.microsoft.com/en-us/cpp/error-messages/compiler-errors-2/compiler-error-c2752
- CWG Issue 727: Full specializations in class scope - https://cplusplus.github.io/CWG/issues/727.html
- CWG Issue 996: Ambiguous partial specializations of member class templates - https://cplusplus.github.io/CWG/issues/996.html