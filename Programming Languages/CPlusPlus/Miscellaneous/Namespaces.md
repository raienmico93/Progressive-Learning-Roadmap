# C++ Namespaces — Comprehensive Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
A namespace is a declarative region that provides a scope to the identifiers (names of types, functions, variables) inside it. Namespaces are used to organize code into logical groups and to prevent name collisions that can occur especially when your code base includes multiple libraries.

**Technical Definition**
A namespace is a named scope introduced by the `namespace` keyword, containing a sequence of declarations. Entities declared within a namespace block belong to that namespace's scope and must be referred to using qualified name lookup (via the scope resolution operator `::`) or made visible through using-declarations, using-directives, or namespace aliases. Multiple namespace blocks with the same name are permitted; all declarations within those blocks are declared in the same namespace scope.

**Beginner-Friendly Explanation**
Imagine you have two friends named "Alex" — one from school and one from work. When you talk about "Alex," it's confusing. In C++, if two libraries both define a function called `print()`, the compiler doesn't know which one you mean. A namespace is like a last name: `School::Alex` and `Work::Alex` are now unambiguous. Namespaces let you group related code under a "family name" so that identical first names don't clash.

---

### Key Characteristics

- **Name collision prevention**: Namespaces isolate identifiers so that identical names in different namespaces do not conflict.
- **Open for extension**: A namespace can be reopened and extended across multiple files and translation units; all declarations are merged into the same scope.
- **Nesting**: Namespaces can be nested arbitrarily deep, with C++17 introducing a concise nested syntax.
- **Inline namespaces** (C++11): Members of an inline namespace are transparently visible in the enclosing namespace, enabling versioning without breaking client code.
- **Unnamed namespaces**: Introduce internal linkage, restricting symbols to the current translation unit.
- **Aliases**: A namespace can have multiple names via `namespace alias = original;`.
- **ADL interaction**: Argument-Dependent Lookup (ADL) allows unqualified function calls to find functions in the namespaces associated with the argument types, which can be both powerful and dangerous.

---

### Prerequisites

- Basic C++ syntax: declarations, definitions, scope, and the `::` operator.
- Understanding of translation units and separate compilation.
- Familiarity with header files (`.h`/`.hpp`) and source files (`.cpp`).
- Basic knowledge of linkage (external vs. internal).
- (For C++17 nested syntax) A compiler that supports C++17 or later.
- (For inline namespaces) A compiler that supports C++11 or later.

---

### Related Programming Areas

- **Library design and API/ABI versioning**
- **Linkage and symbol visibility** (static, extern, internal linkage)
- **Name lookup and overload resolution**
- **Template metaprogramming** (ADL is critical for customization points like `swap`)
- **Module systems** (C++20 modules are an alternative to header-based namespace organization)
- **Build systems and symbol management** (namespace-aware linking, symbol mangling)

---

### Core Concepts / Features

1. **Explicit Grouping** — Named namespaces and nested syntax (including C++17 `namespace A::B::C`).
2. **API Versioning** — Inline namespaces for backward compatibility and library versioning.
3. **Scope Resolution** — Namespace aliases, using declarations, and using namespace directives.
4. **Internal Linkage** — Anonymous (unnamed) namespaces for translation-unit-local symbols.
5. **Lookup Mechanics** — Argument-Dependent Lookup (ADL / Koenig Lookup) and its security implications.
6. **Pollution Mitigation** — Strict rules for keeping namespaces clean (e.g., forbidding `using namespace` in header files).

---

## 1. Explicit Grouping

### Definitions

**Core Definition**
Explicit grouping refers to declaring named namespaces to logically group related declarations under a single name, and using nested namespace syntax (including the C++17 shorthand) to create hierarchical namespace structures.

**Technical Definition**
A named namespace is defined by the syntax `namespace ns_name { declarations }`. A nested namespace definition `namespace A::B::C { ... }` (C++17) is exactly equivalent to `namespace A { namespace B { namespace C { ... } } }`. Namespace names (along with class names) can appear on the left-hand side of the scope resolution operator as part of qualified name lookup.

**Beginner-Friendly Explanation**
Just as folders on your computer organize files into `Documents/Work/Reports`, namespaces organize code into `Company::Project::Module`. Instead of writing three nested `namespace` blocks, C++17 lets you write `namespace Company::Project::Module { ... }` in one line.

---

### Purposes (all begin with "To")

- **To** prevent name collisions between independently developed libraries and modules.
- **To** provide a logical, hierarchical organization of code that mirrors project structure.
- **To** allow multiple source files to contribute to the same namespace seamlessly.
- **To** reduce boilerplate in deeply nested namespace hierarchies using C++17 syntax.
- **To** make qualified name lookup explicit and unambiguous in large codebases.

---

### Syntax Rules and Structure

#### Complete General Syntaxes with Breakdowns

**Syntax 1 — Named namespace definition (C++98)**
```cpp
namespace ns_name {
    declarations
}
```
**Component breakdown:**
- `namespace` — keyword introducing the namespace definition.
- `ns_name` — identifier for the namespace.
- `{ declarations }` — the namespace body containing any declarations (types, functions, variables, other namespaces).

**Syntax 2 — Nested namespace definition (C++17)**
```cpp
namespace A::B::C {
    declarations
}
```
**Component breakdown:**
- `A::B::C` — a sequence of enclosing namespace specifiers separated by `::`.
- Equivalent to `namespace A { namespace B { namespace C { ... } } }`.

**Syntax 3 — Nested inline namespace definition (C++20)**
```cpp
namespace A::B::inline C {
    declarations
}
```
**Component breakdown:**
- `inline` may appear in front of every namespace name except the first.
- `namespace A::inline B::C {}` is equivalent to `namespace A { inline namespace B { namespace C {} } }`.

#### Syntax Rules

- Namespaces can be reopened: multiple `namespace ns_name { ... }` blocks merge into one scope.
- Namespace names must be unique within their enclosing scope unless reopened.
- `inline` can only appear on a namespace definition, not on a namespace alias.
- Nested namespace definitions require C++17 or later.
- Nested inline namespace definitions (with `inline` after `::`) require C++20 or later.

#### Constraints and Limitations

- A namespace cannot be declared inside a function or class.
- You cannot partially specialize or explicitly instantiate a template in a namespace that has not been declared.
- The global namespace has no declaration and is not an unnamed namespace; it can be referred to explicitly with a leading `::`.
- C++17 nested namespace syntax is not supported by older compilers (e.g., MSVC before 2017 15.3 required `/std:c++17`).

---

### Multiple Annotated Complete Step-by-Step Code Examples and Expected Outputs

#### Example 1 — Basic Named Namespace

```cpp
// File: basic_namespace.cpp
#include <iostream>

// Step 1: Define a namespace called "Math"
namespace Math {
    // Step 2: Declare a constant inside the namespace
    const double PI = 3.141592653589793;

    // Step 3: Declare a function inside the namespace
    double square(double x) {
        return x * x;
    }
}

// Step 4: Define another namespace with the same name (reopened)
namespace Math {
    // Step 5: Add another function to the same namespace
    double cube(double x) {
        return x * x * x;
    }
}

int main() {
    // Step 6: Access members using the scope resolution operator ::
    std::cout << "PI = " << Math::PI << std::endl;
    std::cout << "square(5) = " << Math::square(5) << std::endl;
    std::cout << "cube(3) = " << Math::cube(3) << std::endl;

    // Step 7: Bring a specific name into scope with a using-declaration
    using Math::square;
    std::cout << "square(7) = " << square(7) << std::endl;

    return 0;
}
```

**Expected Output:**
```
PI = 3.14159
square(5) = 25
cube(3) = 27
square(7) = 49
```

**Why this output:** The namespace `Math` is reopened, so `PI`, `square`, and `cube` all belong to the same namespace scope. Qualified lookup with `Math::` resolves each name. The using-declaration `using Math::square;` makes `square` accessible unqualified in `main`, so `square(7)` calls `Math::square(7)`.

---

#### Example 2 — C++17 Nested Namespace Definition

```cpp
// File: nested_cpp17.cpp
// Compile with: g++ -std=c++17 nested_cpp17.cpp -o nested_cpp17

#include <iostream>

// Step 1: C++17 nested namespace definition
namespace Company::Project::Module {
    void greet() {
        std::cout << "Hello from Company::Project::Module!" << std::endl;
    }

    struct Config {
        int version = 17;
    };
}

// Step 2: Reopen the nested namespace using the same C++17 syntax
namespace Company::Project::Module {
    void farewell() {
        std::cout << "Goodbye from Company::Project::Module!" << std::endl;
    }
}

int main() {
    // Step 3: Call functions using fully qualified names
    Company::Project::Module::greet();
    Company::Project::Module::farewell();

    // Step 4: Create an object of the nested struct
    Company::Project::Module::Config cfg;
    std::cout << "Config version: " << cfg.version << std::endl;

    // Step 5: Use a namespace alias to shorten access
    namespace CPM = Company::Project::Module;
    CPM::greet();

    return 0;
}
```

**Expected Output:**
```
Hello from Company::Project::Module!
Goodbye from Company::Project::Module!
Config version: 17
Hello from Company::Project::Module!
```

**Why this output:** The C++17 nested syntax `namespace Company::Project::Module` creates the same hierarchy as three nested `namespace` blocks. Reopening the namespace merges `greet` and `farewell` into the same scope. The namespace alias `CPM` provides a shorter synonym, and `CPM::greet()` resolves to the same function.

---

### Real-World Cases with Explanation

**Case 1 — Standard Library organization:** The entire C++ standard library is placed in `namespace std`. Sub-namespaces like `std::chrono`, `std::filesystem`, and `std::ranges` (C++20) provide further grouping. This prevents user code from accidentally colliding with standard names.

**Case 2 — Boost libraries:** Boost uses deeply nested namespaces such as `boost::asio::ip::tcp` to organize networking components, mirroring the conceptual hierarchy of the library.

**Case 3 — Game engines:** A game engine might organize code as `Engine::Graphics::Rendering`, `Engine::Physics::Collision`, and `Engine::Audio::Mixing`, allowing independent teams to work without name clashes.

**Case 4 — Third-party integration:** When integrating two libraries that both define a `Logger` class, wrapping each in its own namespace (e.g., `LibA::Logger` and `LibB::Logger`) resolves the conflict without renaming either class.

---

### References

- Namespaces — cppreference.com - https://en.cppreference.com/w/cpp/language/namespace
- Nested namespace definition (C++17) — cppreference.com - https://en.cppreference.com/w/cpp/language/namespace#Syntax
- C++17 nested namespace definitions — Microsoft Learn - https://learn.microsoft.com/en-us/cpp/build/reference/std-specify-language-standard-version

---

## 2. API Versioning (Inline Namespaces)

### Definitions

**Core Definition**
An inline namespace is a namespace whose members are also visible in the enclosing namespace as if they were declared directly in the enclosing namespace, without requiring qualification.

**Technical Definition**
An inline namespace is introduced by placing the `inline` keyword before the namespace name: `inline namespace ns_name { declarations }` (since C++11). Declarations inside an inline namespace are visible in its enclosing namespace for unqualified name lookup. This feature is primarily used for ABI versioning: multiple versions of a library can coexist, with the newest version marked `inline` so that client code using unqualified names automatically binds to the current version.

**Beginner-Friendly Explanation**
Imagine a library that has a version 1 and a version 2 of a widget. You want old code that says `Widget` to keep working (using v1), but new code that says `Widget` to get v2. Inline namespaces let you do this: you put both versions in named namespaces `v1` and `v2`, and mark `v2` as `inline`. Then, by default, `Widget` refers to `v2::Widget`, but old code that explicitly says `v1::Widget` still works.

---

### Purposes (all begin with "To")

- **To** evolve a library's ABI without breaking existing client code that uses unqualified names.
- **To** allow multiple versions of the same symbol to coexist in a single binary.
- **To** provide a transparent "current version" that client code uses automatically.
- **To** enable library authors to switch the default version by changing which namespace is marked `inline`.
- **To** support the standard library's own versioning schemes (e.g., `std::__cxx11` in libstdc++, `std::__1` in libc++).

---

### Syntax Rules and Structure

#### Complete General Syntax with Breakdown

```cpp
inline namespace ns_name {
    declarations
}
```

**Component breakdown:**
- `inline` — keyword indicating this is an inline namespace (C++11 and later).
- `namespace` — keyword introducing the namespace definition.
- `ns_name` — identifier for the inline namespace.
- `{ declarations }` — the namespace body.

**Nested inline namespace (C++20):**
```cpp
namespace A::inline B::C {
    declarations
}
```
- `inline` may appear in front of every namespace name except the first.
- Equivalent to `namespace A { inline namespace B { namespace C {} } }`.

#### Syntax Rules

- An inline namespace must be declared with the `inline` keyword.
- The `inline` keyword cannot appear on a namespace extension definition if the original namespace definition was not inline.
- At most one inline namespace per enclosing namespace can be active at a time (though multiple inline namespaces can be declared, name lookup is ambiguous if more than one contains the same name).
- Inline namespaces are transitive: if `A` is inline in `B` and `B` is inline in `C`, then members of `A` are visible in `C`.

#### Constraints and Limitations

- Inline namespaces are a compile-time feature; they do not affect dynamic linking or symbol visibility directly.
- ADL (Argument-Dependent Lookup) considers inline namespaces of associated classes, which can have subtle effects.
- If multiple inline namespaces in the same enclosing namespace declare the same name, unqualified lookup is ambiguous (ill-formed).
- Inline namespaces do not solve source-level incompatibilities; they solve ABI-level versioning.
- The `inline` keyword on namespaces is distinct from the `inline` keyword on functions.

---

### Multiple Annotated Complete Step-by-Step Code Examples and Expected Outputs

#### Example 1 — Basic Inline Namespace Versioning

```cpp
// File: inline_versioning.cpp
// Compile with: g++ -std=c++11 inline_versioning.cpp -o inline_versioning

#include <iostream>

// Step 1: Define version 1 namespace
namespace MyLib {
    namespace v1 {
        void print() {
            std::cout << "MyLib v1::print()" << std::endl;
        }
    }

    // Step 2: Define version 2 namespace and mark it inline
    inline namespace v2 {
        void print() {
            std::cout << "MyLib v2::print()" << std::endl;
        }
    }
}

int main() {
    // Step 3: Unqualified call resolves to the inline namespace (v2)
    MyLib::print();

    // Step 4: Qualified call to v1 still works
    MyLib::v1::print();

    // Step 5: Qualified call to v2 also works explicitly
    MyLib::v2::print();

    return 0;
}
```

**Expected Output:**
```
MyLib v2::print()
MyLib v1::print()
MyLib v2::print()
```

**Why this output:** `v2` is inline, so `MyLib::print()` finds `v2::print` via unqualified lookup. `MyLib::v1::print()` explicitly qualifies to the non-inline v1 namespace. `MyLib::v2::print()` explicitly qualifies to v2.

---

#### Example 2 — Library Versioning with ABI Compatibility

```cpp
// File: abi_versioning.cpp
// Compile with: g++ -std=c++11 abi_versioning.cpp -o abi_versioning

#include <iostream>
#include <string>

// Step 1: Forward declaration of the library's top-level namespace
namespace Library {

    // Step 2: Version 1 namespace (non-inline, kept for old ABI)
    namespace v1 {
        struct Widget {
            int size;
            Widget() : size(10) {}
            void describe() const {
                std::cout << "v1 Widget, size = " << size << std::endl;
            }
        };
    }

    // Step 3: Version 2 namespace (inline, becomes the default)
    inline namespace v2 {
        struct Widget {
            int size;
            int color;  // new field, changing ABI
            Widget() : size(20), color(255) {}
            void describe() const {
                std::cout << "v2 Widget, size = " << size
                          << ", color = " << color << std::endl;
            }
        };
    }
}

// Step 4: Old code compiled against v1 still works
void old_client_code() {
    Library::v1::Widget w;
    w.describe();
}

// Step 5: New code uses unqualified name, gets v2
void new_client_code() {
    Library::Widget w;  // resolves to v2::Widget
    w.describe();
}

int main() {
    old_client_code();
    new_client_code();

    // Step 6: Explicit access to both versions
    Library::v1::Widget old_w;
    Library::v2::Widget new_w;
    old_w.describe();
    new_w.describe();

    return 0;
}
```

**Expected Output:**
```
v1 Widget, size = 10
v2 Widget, size = 20, color = 255
v1 Widget, size = 10
v2 Widget, size = 20, color = 255
```

**Why this output:** `Library::Widget` (unqualified within the namespace) resolves to the inline `v2::Widget`. Old client code that explicitly uses `Library::v1::Widget` continues to work. The ABI of v1 is preserved because `v1::Widget` is a distinct type with a different layout.

---

### Real-World Cases with Explanation

**Case 1 — libstdc++ `std::string` transition:** GCC 5 changed `std::string` from Copy-On-Write (COW) to Small-String Optimization (SSO), which is an ABI break. libstdc++ used inline namespaces (`std::__cxx11`) so that old binaries linked against the COW string could continue to work while new code used the SSO string. The inline namespace technique is used throughout the standard library: libc++ puts everything in `std::__1`, libstdc++ uses `std::__cxx11`, and MSVC STL uses versioned implementation namespaces.

**Case 2 — nlohmann/json library:** The library uses an inline namespace with a version suffix (`_v` followed by version numbers). For example, the namespace for version 3.11.2 might be `nlohmann::json_abi_v3_11_2`, with an inline namespace making it accessible as `nlohmann::json`.

**Case 3 — Library ABI evolution:** A library can define `v1`, `v2`, `v3`, etc., and mark only the newest as `inline`. Client code that uses unqualified names automatically tracks the newest version, while code that explicitly qualifies to an older version continues to link against the old ABI.

---

### References

- Inline namespaces — cppreference.com - https://en.cppreference.com/w/cpp/language/namespace#Inline_namespaces
- Use inline namespaces for ABI versioning — Dr-Sergey/learn_cpp (GitHub) - https://github.com/Dr-Sergey/learn_cpp
- libstdc++ inline namespace versioning — GCC sourceware - https://gcc.gnu.org/onlinedocs/libstdc++/manual/using_dual_abi.html

---

## 3. Scope Resolution

### Definitions

**Core Definition**
Scope resolution is the mechanism by which a name is qualified with its namespace (or class) using the `::` operator to specify exactly which entity is meant. Namespace aliases, using-declarations, and using-directives are tools that modify or simplify scope resolution.

**Technical Definition**
The scope resolution operator `::` performs qualified name lookup, searching the specified namespace or class scope for the declared name. A namespace alias (`namespace alias = qualified-namespace;`) creates a synonym for a namespace. A using-declaration (`using ns_name::member_name;`) introduces a specific name into the current scope. A using-directive (`using namespace ns_name;`) makes all names from the namespace visible for unqualified lookup in the nearest enclosing namespace that contains both the directive and the namespace.

**Beginner-Friendly Explanation**
`::` is like a GPS coordinate for names: instead of just saying "print," you say `Library::print` to specify exactly which `print` you mean. A namespace alias is a nickname (e.g., `namespace fs = std::filesystem;`). A using-declaration brings one specific name into scope (e.g., `using std::cout;`). A using-directive brings *all* names from a namespace into scope — convenient but dangerous in headers.

---

### Purposes (all begin with "To")

- **To** unambiguously identify which namespace a name belongs to.
- **To** create short synonyms for long namespace names via aliases.
- **To** bring specific names into the current scope without full qualification using using-declarations.
- **To** make all names from a namespace available in a limited scope using using-directives.
- **To** control name lookup precisely in large codebases and avoid ambiguities.

---

### Syntax Rules and Structure

#### Complete General Syntaxes with Breakdowns

**Syntax 1 — Qualified name lookup**
```cpp
ns_name::member_name
```
- `ns_name` — the namespace (or class) name.
- `::` — scope resolution operator.
- `member_name` — the entity to look up in `ns_name`.

**Syntax 2 — Namespace alias**
```cpp
namespace alias_name = qualified_namespace;
```
- `alias_name` — the new short name.
- `qualified_namespace` — the fully qualified existing namespace.

**Syntax 3 — Using-declaration**
```cpp
using ns_name::member_name;
```
- Introduces `member_name` into the current scope as if declared there.
- Only that specific name is introduced.

**Syntax 4 — Using-directive**
```cpp
using namespace ns_name;
```
- Makes all names from `ns_name` visible for unqualified lookup in the nearest enclosing namespace containing both the directive and `ns_name`.

#### Syntax Rules

- A namespace alias must refer to an already-declared namespace.
- A using-declaration can only introduce a name that exists in the target namespace.
- A using-declaration in a class scope can only introduce members of a base class.
- A using-directive affects unqualified lookup only; qualified lookup (`ns::name`) is unaffected.
- A using-directive is transitive: if `A` uses `B` and `B` uses `C`, names from `C` may be visible in `A`.
- Using-directives do not add names to a namespace; they only affect lookup.

#### Constraints and Limitations

- **Using-directives in headers are dangerous**: they pollute the namespace of every translation unit that includes the header.
- A using-directive cannot be used to introduce a namespace name into another namespace for the purpose of reopening it.
- Namespace aliases cannot be used in a `using` declaration before the alias is declared.
- Using-declarations that conflict with existing names in the same scope may cause ambiguity errors.
- The `using` keyword has different meanings depending on context: using-declaration, using-directive, type alias, and (C++11) alias template.

---

### Multiple Annotated Complete Step-by-Step Code Examples and Expected Outputs

#### Example 1 — Namespace Alias and Using-Declaration

```cpp
// File: scope_resolution.cpp
// Compile with: g++ -std=c++11 scope_resolution.cpp -o scope_resolution

#include <iostream>
#include <vector>

// Step 1: A deeply nested namespace
namespace VeryLongCompanyName {
    namespace Project {
        namespace Utilities {
            void log(const std::string& msg) {
                std::cout << "[LOG] " << msg << std::endl;
            }
        }
    }
}

int main() {
    // Step 2: Create a namespace alias for convenience
    namespace Util = VeryLongCompanyName::Project::Utilities;
    Util::log("Using alias");

    // Step 3: Using-declaration brings only 'log' into scope
    using VeryLongCompanyName::Project::Utilities::log;
    log("Using declaration");

    // Step 4: Using-directive brings all names from a namespace into scope
    // (acceptable here because it is inside a function, not a header)
    using namespace VeryLongCompanyName::Project::Utilities;
    log("Using directive");

    return 0;
}
```

**Expected Output:**
```
[LOG] Using alias
[LOG] Using declaration
[LOG] Using directive
```

**Why this output:** The alias `Util` provides a short synonym. The using-declaration introduces only `log` into `main`'s scope. The using-directive makes all names from `Utilities` visible, but since only `log` exists, the result is the same. All three calls resolve to `VeryLongCompanyName::Project::Utilities::log`.

---

#### Example 2 — Ambiguity with Using-Directives

```cpp
// File: ambiguity.cpp
// Compile with: g++ -std=c++11 ambiguity.cpp -o ambiguity

#include <iostream>

namespace A {
    void foo() { std::cout << "A::foo" << std::endl; }
}

namespace B {
    void foo() { std::cout << "B::foo" << std::endl; }
}

int main() {
    using namespace A;
    using namespace B;

    // Uncommenting the following line causes a compile error:
    // foo();  // ambiguous: both A::foo and B::foo are visible

    A::foo();  // OK: qualified
    B::foo();  // OK: qualified

    return 0;
}
```

**Expected Output:**
```
A::foo
B::foo
```

**Why this output:** The two using-directives make both `A::foo` and `B::foo` visible for unqualified lookup. Calling `foo()` unqualified would be ambiguous and cause a compile error. Qualified calls `A::foo()` and `B::foo()` work because they bypass the ambiguity.

---

### Real-World Cases with Explanation

**Case 1 — `std::filesystem` alias:** `namespace fs = std::filesystem;` is a common alias that shortens verbose qualification in code that heavily uses filesystem operations.

**Case 2 — Using-declarations in class scope:** A derived class can use `using Base::method;` to bring a base class method into the derived class's scope, which is especially useful when overriding overloaded virtual functions.

**Case 3 — Using-directives in `.cpp` files:** It is common and acceptable to write `using namespace std;` inside a `.cpp` file (not a header) to reduce verbosity, because the pollution is confined to that translation unit.

**Case 4 — Avoiding ambiguity in large projects:** Libraries often provide namespace aliases (e.g., `namespace fs = boost::filesystem;`) to give users a short, stable name while the full namespace hierarchy evolves.

---

### References

- Scope resolution operator — cppreference.com - https://en.cppreference.com/w/cpp/language/identifiers#Qualified_identifiers
- Namespace alias — cppreference.com - https://en.cppreference.com/w/cpp/language/namespace_alias
- Using-declaration — cppreference.com - https://en.cppreference.com/w/cpp/language/using_declaration
- Using-directive — cppreference.com - https://en.cppreference.com/w/cpp/language/namespace#Using-directives

---

## 4. Internal Linkage (Anonymous Namespaces)

### Definitions

**Core Definition**
An anonymous (unnamed) namespace is a namespace without a name whose members have internal linkage, meaning they are visible only within the translation unit in which they are declared.

**Technical Definition**
An unnamed namespace definition has the form `namespace { declarations }`. Its members have potential scope from their point of declaration to the end of the translation unit and have internal linkage. In C++11 and later, members of an unnamed namespace have internal linkage (previously, in C++03, they had external linkage with a unique generated name).

**Beginner-Friendly Explanation**
An anonymous namespace is like a "private room" in a single `.cpp` file. Anything declared inside it can only be seen and used in that file. Other `.cpp` files can't access it, even if they declare the same name. This is useful for helper functions and variables that shouldn't be visible outside the file.

---

### Purposes (all begin with "To")

- **To** restrict helper functions and variables to a single translation unit, avoiding linker collisions.
- **To** replace the older C-style `static` keyword for file-scope functions and variables.
- **To** hide implementation details from other translation units.
- **To** avoid polluting the global namespace with names that are only needed locally.
- **To** ensure that two `.cpp` files can define the same helper name without conflict.

---

### Syntax Rules and Structure

#### Complete General Syntax with Breakdown

```cpp
namespace {
    declarations
}
```

**Component breakdown:**
- `namespace` — keyword introducing an unnamed namespace.
- `{ declarations }` — the body containing declarations that receive internal linkage.

#### Syntax Rules

- An unnamed namespace can be reopened, though this is rarely useful.
- Members of an unnamed namespace are implicitly accessible within the enclosing namespace without qualification.
- The unnamed namespace is unique to each translation unit; each `.cpp` file gets its own distinct unnamed namespace.
- In C++11 and later, names in an unnamed namespace have internal linkage.
- Before C++11, names in unnamed namespaces had external linkage but were effectively unique due to a generated namespace name.

#### Constraints and Limitations

- Entities with internal linkage cannot be used as template arguments in certain contexts (e.g., as non-type template parameters in some cases).
- `extern "C"` has no effect on names with internal linkage in C++11 and later.
- Two-phase name lookup ignores functions with internal linkage during the second phase (ADL).
- Unnamed namespaces inside inline namespaces can lead to ODR violations if not carefully managed.
- The behavior changed between C++03 and C++11; code that relied on external linkage of unnamed namespace members may break.

---

### Multiple Annotated Complete Step-by-Step Code Examples and Expected Outputs

#### Example 1 — Anonymous Namespace for Helper Functions

```cpp
// File: anonymous_namespace.cpp
// Compile with: g++ -std=c++11 anonymous_namespace.cpp -o anonymous_namespace

#include <iostream>
#include <string>

// Step 1: Anonymous namespace for file-local helpers
namespace {
    // This function is only visible in this translation unit
    std::string format_name(const std::string& first,
                            const std::string& last) {
        return last + ", " + first;
    }

    // This constant is also file-local
    const int MAX_RETRIES = 3;
}

// Step 2: Public function uses the private helper
void print_formatted(const std::string& first,
                     const std::string& last) {
    std::cout << format_name(first, last)
              << " (max retries: " << MAX_RETRIES << ")"
              << std::endl;
}

int main() {
    print_formatted("Ada", "Lovelace");
    print_formatted("Alan", "Turing");

    // Uncommenting the following line would cause a compile error
    // because format_name is not visible outside the anonymous namespace:
    // std::cout << format_name("A", "B") << std::endl;

    return 0;
}
```

**Expected Output:**
```
Lovelace, Ada (max retries: 3)
Turing, Alan (max retries: 3)
```

**Why this output:** `format_name` and `MAX_RETRIES` are declared in an anonymous namespace, so they have internal linkage and are only visible in this `.cpp` file. `print_formatted` (which has external linkage) can call them because it is in the same translation unit. Attempting to call `format_name` from another `.cpp` file would result in a linker error.

---

#### Example 2 — Two Translation Units with the Same Helper Name

```cpp
// File: tu1.cpp
#include <iostream>

namespace {
    void helper() {
        std::cout << "tu1: helper" << std::endl;
    }
}

void func1() {
    helper();  // calls tu1's anonymous helper
}
```

```cpp
// File: tu2.cpp
#include <iostream>

namespace {
    void helper() {  // same name, different function
        std::cout << "tu2: helper" << std::endl;
    }
}

void func2() {
    helper();  // calls tu2's anonymous helper
}
```

```cpp
// File: main.cpp
void func1();
void func2();

int main() {
    func1();
    func2();
    return 0;
}
```

**Expected Output:**
```
tu1: helper
tu2: helper
```

**Why this output:** Each translation unit has its own distinct anonymous namespace. The `helper` in `tu1.cpp` and the `helper` in `tu2.cpp` are different functions with internal linkage, so there is no linker conflict. Each `func` calls its own local `helper`.

---

### Real-World Cases with Explanation

**Case 1 — Implementation details in libraries:** A library's `.cpp` file often contains helper functions (e.g., string parsing, memory alignment) that should not be exposed in the public API. Anonymous namespaces hide these symbols from the linker.

**Case 2 — Avoiding ODR violations:** Two third-party libraries might both define a file-scope helper called `log_debug`. If both use anonymous namespaces, there is no ODR violation because the symbols have internal linkage.

**Case 3 — Replacing `static`:** Modern C++ code uses anonymous namespaces instead of `static` for file-scope functions and variables. The `static` keyword at namespace scope is deprecated in C++11 and later.

**Case 4 — GCC 15 linkage change:** Starting with GCC 15, the C++11 rule that anonymous namespaces have internal linkage is fully enforced. Code that previously relied on the old behavior (external linkage with a unique name) may encounter unexpected linker issues due to changes in name mangling.

---

### References

- Unnamed namespaces — cppreference.com - https://en.cppreference.com/w/cpp/language/namespace#Unnamed_namespaces
- Internal linkage — cppreference.com - https://en.cppreference.com/w/cpp/language/storage_duration
- CWG Issue 1603: Linkage of unnamed namespaces - https://cplusplus.github.io/CWG/issues/1603.html
- CWG Issue 2772: Linkage effects of unnamed namespaces - https://cplusplus.github.io/CWG/issues/2772.html

---

## 5. Lookup Mechanics — Argument-Dependent Lookup (ADL / Koenig Lookup)

### Definitions

**Core Definition**
Argument-Dependent Lookup (ADL), also known as Koenig Lookup, is a set of rules for unqualified function name lookup that considers functions declared in the namespaces associated with the types of the function arguments.

**Technical Definition**
When an unqualified function name is used in a function call, ADL causes the compiler to search not only the usual scopes (block, class, namespace) but also the namespaces associated with each argument's type. For a class type, the associated namespaces include the namespace in which the class is defined, the namespaces of its base classes, and the namespaces of its template arguments (if it is a class template specialization). ADL is used to find friend functions and functions in the same namespace as a type, which is essential for operator overloading and customization points like `swap` and `begin`/`end`.

**Beginner-Friendly Explanation**
Normally, when you call a function, the compiler looks in the current scope and enclosing scopes. ADL adds a twist: if the function takes an argument of type `MyNamespace::MyClass`, the compiler also looks inside `MyNamespace` for a matching function. This is why you can write `swap(a, b)` without `std::` when `a` and `b` are `std::string` — ADL finds `std::swap` because `std::string` is in `std`. But this power comes with risks.

---

### Purposes (all begin with "To")

- **To** enable unqualified calls to functions that are logically associated with a type's namespace.
- **To** support operator overloading in namespaces (e.g., `operator<<` for a type defined in a user namespace).
- **To** allow customization points like `swap`, `begin`, and `end` to work without explicit qualification.
- **To** make generic code (templates) more flexible by finding the right overload based on argument types.
- **To** reduce verbosity in code that heavily uses a type's associated functions.

---

### Syntax Rules and Structure

#### How ADL Works (General Rules)

For an unqualified function call `f(args...)`:

1. **Normal unqualified lookup** proceeds from the current scope outward through enclosing scopes.
2. **ADL** additionally searches the namespaces associated with the types of the arguments.
3. The union of the candidates found by both processes forms the overload set.
4. Overload resolution then selects the best match.

**Associated namespaces for a type:**
- If the type is a class, its associated namespace is the innermost enclosing namespace of the class.
- If the class has base classes, the associated namespaces of the bases are included.
- If the class is a class template specialization, the associated namespaces of the template arguments are included.
- For fundamental types, there are no associated namespaces (ADL adds nothing).
- For pointers and arrays, the associated namespaces are those of the pointed-to or element type.

#### Syntax Rules

- ADL applies only to unqualified function calls, not to qualified calls (`ns::f()`).
- ADL does not consider variables, types, or concepts — only functions.
- ADL considers friend functions declared inside a class, even if they are not visible through normal lookup.
- ADL is suppressed in certain contexts: when the function name is followed by `<` (as in `f<int>(x)`), ADL is not performed unless the name is not a template.
- The set of associated namespaces can be surprisingly large, especially with template arguments and base classes.

#### Constraints and Limitations

- **ADL can cause surprising overloads**: calling `move(UserType)` might not call `std::move`; it could call a user-defined `move` in the same namespace as `UserType`. The libc++ coding guidelines state: "Unqualified function calls are susceptible to ADL. Therefore, function calls must use qualified names to avoid ADL."
- **ADL can break encapsulation**: a function in an unrelated namespace can be found if a type from that namespace is used as an argument.
- **ADL and operators**: `&user_object` may call a user-defined `operator&`. Use `std::addressof` instead. Similarly, `operator,` can be hijacked; cast to `void` or avoid it.
- **ADL in templates**: ADL is performed at the point of instantiation for dependent names, which can lead to different overloads being selected in different translation units.
- **Security implications**: ADL can be exploited to inject function calls into generic code. An attacker-controlled type can cause a generic algorithm to call an unexpected function if that function exists in the type's namespace.

---

### Multiple Annotated Complete Step-by-Step Code Examples and Expected Outputs

#### Example 1 — ADL in Action (Finding a Friend Function)

```cpp
// File: adl_basic.cpp
// Compile with: g++ -std=c++11 adl_basic.cpp -o adl_basic

#include <iostream>

namespace Geometry {
    struct Point {
        double x, y;
    };

    // Step 1: Friend function defined inside the class (or namespace)
    // ADL will find this because Point is in Geometry
    void print(const Point& p) {
        std::cout << "(" << p.x << ", " << p.y << ")" << std::endl;
    }
}

int main() {
    Geometry::Point p{3.0, 4.0};

    // Step 2: Unqualified call — ADL finds Geometry::print
    // because the argument p is of type Geometry::Point
    print(p);  // calls Geometry::print

    // Step 3: This also works with operators
    // For example, if Geometry defined operator<<, it would be found

    return 0;
}
```

**Expected Output:**
```
(3, 4)
```

**Why this output:** `print(p)` is an unqualified call. Normal lookup in `main` finds no `print`. ADL then searches the namespace associated with `Geometry::Point`, which is `Geometry`, and finds `Geometry::print`. The function is called with `p`.

---

#### Example 2 — ADL Hijacking and Qualified Calls

```cpp
// File: adl_hijack.cpp
// Compile with: g++ -std=c++11 adl_hijack.cpp -o adl_hijack

#include <iostream>
#include <utility>

namespace Evil {
    struct Malicious {};

    // Step 1: A function named "move" in the Evil namespace
    void move(const Malicious&) {
        std::cout << "Evil::move called! (ADL hijack)" << std::endl;
    }
}

int main() {
    Evil::Malicious m;

    // Step 2: Unqualified call to move with a Malicious argument
    // ADL finds Evil::move, NOT std::move
    move(m);  // Evil::move is called

    // Step 3: The safe way — qualified call
    // std::move(m);  // This would call std::move (which is not what
                      // we want for a non-movable type, but at least
                      // it's the intended function)

    // Step 4: In generic code, always qualify:
    // std::move(obj) instead of move(obj)

    return 0;
}
```

**Expected Output:**
```
Evil::move called! (ADL hijack)
```

**Why this output:** The unqualified call `move(m)` triggers ADL because `m` is of type `Evil::Malicious`. ADL searches the `Evil` namespace and finds `Evil::move`, which is a better match (exact match) than any `std::move` template. The `std::move` function is not even considered if the user intended it. This is why libc++ guidelines mandate qualified calls to standard library functions.

---

#### Example 3 — ADL and `swap` (The Classic Customization Point)

```cpp
// File: adl_swap.cpp
// Compile with: g++ -std=c++11 adl_swap.cpp -o adl_swap

#include <iostream>
#include <utility>

namespace MyContainer {
    struct Vec {
        int* data;
        int size;

        Vec(int n) : data(new int[n]), size(n) {
            for (int i = 0; i < n; ++i) data[i] = i;
        }

        ~Vec() { delete[] data; }

        // Disable copy for simplicity
        Vec(const Vec&) = delete;
        Vec& operator=(const Vec&) = delete;
    };

    // Step 1: Custom swap for Vec (efficient, no allocation)
    void swap(Vec& a, Vec& b) noexcept {
        std::swap(a.data, b.data);
        std::swap(a.size, b.size);
        std::cout << "MyContainer::swap called" << std::endl;
    }
}

int main() {
    MyContainer::Vec a(5), b(3);

    // Step 2: Unqualified swap — ADL finds MyContainer::swap
    swap(a, b);  // calls MyContainer::swap

    // Step 3: In generic code, the standard idiom is:
    // using std::swap;
    // swap(a, b);  // ADL finds the best swap

    return 0;
}
```

**Expected Output:**
```
MyContainer::swap called
```

**Why this output:** `swap(a, b)` is unqualified. ADL searches the namespace of `a` and `b`, which is `MyContainer`, and finds `MyContainer::swap`. This is the intended customization point. The `using std::swap; swap(a, b);` idiom ensures that `std::swap` is available as a fallback while ADL can find a better specialization.

---

### Real-World Cases with Explanation

**Case 1 — Standard library `swap`:** The standard library relies on ADL to find user-defined `swap` functions in generic algorithms. The idiom `using std::swap; swap(a, b);` is ubiquitous in modern C++.

**Case 2 — `operator<<` for user-defined types:** When you write `std::cout << my_object`, ADL finds `operator<<` in the namespace of `my_object` (if defined there), allowing you to print custom types without modifying `std::ostream`.

**Case 3 — Range-based for loops:** The `begin` and `end` functions are found via ADL for user-defined types that don't have member `begin`/`end`, enabling range-based for loops.

**Case 4 — ADL hijacking attacks:** In security-sensitive code, an attacker can define a type in a namespace with a function named like a common algorithm (e.g., `move`, `copy`, `swap`). If generic code uses unqualified calls, the attacker's function may be invoked. This is why coding guidelines like libc++'s require qualified calls to standard library functions.

**Case 5 — `std::addressof`:** Because `&user_object` can invoke a user-defined `operator&`, libraries use `std::addressof` to reliably obtain the address of an object, bypassing ADL.

---

### References

- Argument-dependent lookup — cppreference.com - https://en.cppreference.com/w/cpp/language/adl
- libc++ Coding Guidelines: Don't use ADL unless required — https://libcxx.llvm.org/CodingGuidelines.html
- ADL and the `swap` idiom — C++ Core Guidelines - https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Res-swap

---

## 6. Pollution Mitigation

### Definitions

**Core Definition**
Pollution mitigation refers to the set of rules and best practices that prevent namespace pollution — the unintended introduction of names into scopes where they are not wanted, which can cause name collisions and ambiguity.

**Technical Definition**
Namespace pollution occurs when a using-directive or using-declaration introduces names into a scope, or when a header file contains a `using namespace` directive at global scope, causing all names from the directive's namespace to become visible in every translation unit that includes the header. The primary mitigation rule is: **do not write `using namespace` at global scope in a header file.**

**Beginner-Friendly Explanation**
If a header file says `using namespace std;`, then every `.cpp` file that includes that header suddenly has all of `std`'s names dumped into the global namespace. This can cause collisions with your own functions, macros, or other libraries. The fix is simple: never put `using namespace` in a header file. Use fully qualified names or using-declarations (which introduce only specific names) instead.

---

### Purposes (all begin with "To")

- **To** prevent name collisions caused by unintended visibility of names from other namespaces.
- **To** keep header files clean and predictable for all consumers.
- **To** avoid ODR violations and ambiguity errors in large codebases.
- **To** ensure that including a header does not change the meaning of code in the including file.
- **To** make namespace dependencies explicit and local.

---

### Syntax Rules and Structure

#### The Golden Rule (C++ Core Guidelines SF.7)

> **Don't write `using namespace` at global scope in a header file.**

#### What to Do Instead

| Polluting (avoid in headers) | Safe alternative |
|---|---|
| `using namespace std;` at global scope | Fully qualify: `std::string`, `std::vector` |
| `using namespace boost;` at global scope | Use a namespace alias: `namespace fs = boost::filesystem;` |
| `using namespace MyLib;` at global scope | Use using-declarations for specific names: `using MyLib::Widget;` |
| `using namespace std;` in a header | Place `using namespace std;` only inside `.cpp` files or inside a function body (local scope). |

#### Syntax Rules

- A using-directive at namespace scope in a header file is considered bad practice by the C++ Core Guidelines (SF.7) and is flagged by static analysis tools like SonarQube and clang-tidy.
- A using-directive inside a function body (local scope) is acceptable because it only affects that function.
- A using-declaration (introducing a specific name) is acceptable in a header, but it can still cause conflicts if the name is already used. Prefer fully qualified names in headers.
- A namespace alias is safe in a header because it does not introduce any new names — it only creates a synonym.
- The C++ standard itself does not forbid using-directives in headers, but the collective experience of the C++ community and explicit guidelines recommend against it.

#### Constraints and Limitations

- **MISRA C++:2008, 7-3-6** forbids using-directives and using-declarations (excluding class scope or function scope using-declarations) in header files.
- **C++ Core Guidelines SF.7** explicitly flags `using namespace` at global scope in a header file.
- Using-directives in headers can cause **ODR violations** if two headers introduce conflicting names that are then used in inline functions or templates.
- The effect of a using-directive is **transitive**: if header A uses namespace B, and header C includes A, then names from B are visible in C.
- **Never** put `using namespace std;` in a header. This is the single most common namespace pollution mistake in C++.

---

### Multiple Annotated Complete Step-by-Step Code Examples and Expected Outputs

#### Example 1 — The Bad Header (Pollution)

```cpp
// File: bad_header.h
#pragma once
#include <string>
#include <vector>

// BAD: This pollutes the global namespace of every file that includes
// this header. All of std::string, std::vector, etc. become visible
// unqualified.
using namespace std;  // <-- DON'T DO THIS IN A HEADER

struct Config {
    string name;        // unqualified — works only because of the using-directive
    vector<int> values;
};

void process(const string& name);
```

```cpp
// File: main.cpp
#include "bad_header.h"
#include <iostream>

// Problem: 'count' could now be ambiguous with std::count
int count = 42;  // OK, but risky

int main() {
    Config c;
    c.name = "test";
    c.values.push_back(1);
    c.values.push_back(2);

    // 'cout' and 'endl' are visible because of the using-directive
    cout << "Config: " << c.name << endl;

    // Unqualified 'string' works because of the using-directive
    string s = "hello";
    cout << s << endl;

    return 0;
}
```

**Problem:** The header `bad_header.h` pollutes the global namespace. Any file that includes it gets all of `std` visible unqualified. If the including file defines its own `count`, `string`, or `vector`, conflicts arise.

---

#### Example 2 — The Good Header (Pollution-Free)

```cpp
// File: good_header.h
#pragma once
#include <string>
#include <vector>

// GOOD: Fully qualified names in the header.
// No using-directive, no pollution.

struct Config {
    std::string name;
    std::vector<int> values;
};

void process(const std::string& name);
```

```cpp
// File: good_main.cpp
#include "good_header.h"
#include <iostream>

// Optional: a namespace alias for convenience within this .cpp file
namespace fs = std::filesystem;  // if needed

int main() {
    Config c;  // uses the good header
    c.name = "test";
    c.values.push_back(1);
    c.values.push_back(2);

    // Fully qualified std:: names
    std::cout << "Config: " << c.name << std::endl;

    // Using-declarations are fine in .cpp files
    using std::string;
    string s = "hello";
    std::cout << s << std::endl;

    return 0;
}
```

**Expected Output:**
```
Config: test
hello
```

**Why this is better:** The header `good_header.h` uses fully qualified `std::` names, so it does not pollute the global namespace. The `.cpp` file can use a using-declaration or a local using-directive without affecting other translation units.

---

#### Example 3 — Safe Use of Using-Directives in `.cpp` Files

```cpp
// File: safe_cpp.cpp
// Compile with: g++ -std=c++17 safe_cpp.cpp -o safe_cpp

#include <iostream>
#include <vector>
#include <algorithm>
#include <string>

// It is acceptable to use a using-directive in a .cpp file
// (not in a header) because the pollution is confined to this
// translation unit.
using namespace std;

int main() {
    vector<string> names = {"Charlie", "Alice", "Bob"};

    // Unqualified sort works because of the using-directive
    sort(names.begin(), names.end());

    for (const auto& name : names) {
        cout << name << endl;
    }

    return 0;
}
```

**Expected Output:**
```
Alice
Bob
Charlie
```

**Why this is acceptable:** The `using namespace std;` is in a `.cpp` file, not a header. The pollution is confined to this translation unit. No other file is affected. However, even in `.cpp` files, some guidelines recommend limiting the scope of using-directives (e.g., inside `main` or inside a local block) to minimize risk.

---

### Real-World Cases with Explanation

**Case 1 — The `using namespace std;` in a competitive programming header:** Many competitive programmers write `using namespace std;` in a header that they include in every solution. While this works for small, self-contained programs, it is a bad habit that causes problems in larger projects. In a professional codebase, it can lead to subtle bugs and ambiguous overloads.

**Case 2 — Library header pollution:** A library that provides a header with `using namespace boost;` at global scope forces all of Boost's names into every consumer's namespace. This is considered a serious design flaw. Well-designed libraries (like Boost itself) never do this.

**Case 3 — The C++ Core Guidelines enforcement:** Tools like clang-tidy have checks (e.g., `cppcoreguidelines-avoid-using-directive`) that flag `using namespace` at global scope in headers. SonarQube has a rule: "`using namespace` directives should not be used in header files."

**Case 4 — MISRA C++ compliance:** In safety-critical systems (automotive, aerospace, medical), MISRA C++:2008 Rule 7-3-6 explicitly forbids using-directives in header files. This is enforced to prevent unpredictable name resolution in safety-critical code.

**Case 5 — Namespace pollution in test frameworks:** Some test frameworks (e.g., older versions of Catch2) used `using namespace` in their headers, causing conflicts with user code. Modern versions avoid this by using nested namespaces and fully qualified names.

---

### References

- C++ Core Guidelines SF.7: Don't write `using namespace` at global scope in a header file - https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rs-using-directive
- MISRA C++:2008 Rule 7-3-6 - https://www.misra.org.uk
- SonarQube Rule S1006: "using namespace" directives should not be used in header files - https://rules.sonarsource.com/cpp/RSPEC-1006
- LLVM Coding Standards: Don't use `using namespace` in headers - https://llvm.org/docs/CodingStandards.html

---

## Summary Table of Core Concepts

| Concept | Key Feature | Primary Use | Key Risk / Constraint |
|---|---|---|---|
| Explicit Grouping | `namespace A::B::C` (C++17) | Code organization, collision prevention | C++17 required for nested syntax |
| API Versioning | `inline namespace v2` | ABI evolution without breaking clients | Multiple inline namespaces with same name = ambiguity |
| Scope Resolution | `::`, aliases, using-declarations | Precise name lookup, reduced verbosity | Using-directives in headers cause pollution |
| Internal Linkage | `namespace { ... }` | File-local helpers | Cannot be used as template arguments |
| Lookup Mechanics (ADL) | Finds functions in argument's namespace | Operator overloading, `swap`, `begin`/`end` | Can be hijacked; security risk in generic code |
| Pollution Mitigation | No `using namespace` in headers | Clean, predictable headers | Violates Core Guidelines SF.7, MISRA C++ 7-3-6 |

---

## References (Consolidated)

- Namespaces — cppreference.com - https://en.cppreference.com/w/cpp/language/namespace
- Argument-dependent lookup — cppreference.com - https://en.cppreference.com/w/cpp/language/adl
- Namespace alias — cppreference.com - https://en.cppreference.com/w/cpp/language/namespace_alias
- Using-declaration — cppreference.com - https://en.cppreference.com/w/cpp/language/using_declaration
- Storage duration and linkage — cppreference.com - https://en.cppreference.com/w/cpp/language/storage_duration
- C++ Core Guidelines SF.7 - https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rs-using-directive
- libc++ Coding Guidelines (ADL) - https://libcxx.llvm.org/CodingGuidelines.html
- MISRA C++:2008 Rule 7-3-6 - https://www.misra.org.uk
- SonarQube Rule S1006 - https://rules.sonarsource.com/cpp/RSPEC-1006
- LLVM Coding Standards - https://llvm.org/docs/CodingStandards.html
- CWG Issue 1603 (Unnamed namespace linkage) - https://cplusplus.github.io/CWG/issues/1603.html
- CWG Issue 2772 (Linkage effects of unnamed namespaces) - https://cplusplus.github.io/CWG/issues/2772.html
- GCC 15 Anonymous Namespace Linkage Change - https://gcc.gnu.org/pipermail/gcc-patches/2026-April/712619.html
- libstdc++ Dual ABI - https://gcc.gnu.org/onlinedocs/libstdc++/manual/using_dual_abi.html