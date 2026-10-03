# C++ Header Files — Comprehensive Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
A header file is a source file, typically with a `.h`, `.hpp`, or `.hxx` extension, that contains declarations of functions, classes, templates, and other entities intended to be shared across multiple translation units (source files) via the `#include` preprocessor directive.

**Technical Definition**
A header file is a file whose contents are textually inserted into a translation unit by the preprocessor at the point of an `#include` directive. Header files serve as the primary mechanism for interface/implementation separation in C++, allowing declarations to be shared across multiple translation units while definitions reside in a single source file. The C++ standard does not mandate a specific file extension or naming convention for headers; however, the Standard Library headers are conventionally identified by angle-bracket includes (e.g., `<vector>`, `<string>`), while project and third-party headers are typically included using double quotes (e.g., `"my_class.h"`).

**Beginner-Friendly Explanation**
Imagine you're building a house. The header file is like a blueprint that shows where the doors, windows, and rooms go. It tells everyone *what* the house looks like from the outside. The `.cpp` file is the actual construction work—the plumbing, wiring, and drywall—that makes it all work. Other builders (other `.cpp` files) only need to see the blueprint (header) to know how to connect their work to yours. They don't need to see the messy construction details. In C++, a header file is that blueprint: it declares *what* exists, while the `.cpp` file defines *how* it works.

---

### Key Characteristics

- **Interface/implementation separation**: Headers declare *what* exists; `.cpp` files define *how* it works.
- **Textual inclusion**: The preprocessor inserts header contents verbatim at each `#include` site.
- **Multiple inclusion risk**: Without protection, a header included twice in the same translation unit causes redefinition errors.
- **Dependency management**: Headers can include other headers, forming a dependency graph that affects compilation order and build times.
- **One Definition Rule (ODR)**: Non-inline functions and variables can be defined only once across all translation units; inline functions and templates are exempt (but must have identical definitions).
- **Categories**: Headers fall into three broad categories — standard library headers, project headers, and third-party headers — each with distinct include syntax and management strategies.
- **C++20 modules**: Modules are a modern alternative to headers, offering faster compilation and better encapsulation.

---

### Prerequisites

- Basic C++ syntax: functions, classes, variables, and the preprocessor.
- Understanding of translation units and separate compilation.
- Familiarity with the compilation pipeline: preprocessing → compilation → assembly → linking.
- Basic knowledge of the One Definition Rule (ODR).
- (For `#pragma once`) A compiler that supports this non-standard extension (most modern compilers do).
- (For C++20 modules) A compiler with C++20 modules support (e.g., MSVC 2019 16.8+, GCC 11+, Clang 16+).

---

### Related Programming Areas

- **Build systems** (CMake, Make, Bazel) and dependency tracking.
- **Linkage and symbol visibility** (external vs. internal linkage, `extern`, `static`).
- **Template metaprogramming** (templates must be defined in headers).
- **Preprocessor directives** (`#include`, `#define`, `#ifndef`, `#pragma`).
- **C++20 modules** (a potential replacement for header-based organization).
- **Compile-time optimization** (precompiled headers, header units, include-what-you-use).

---

### Core Concepts / Features

1. **Structural Isolation** — Defining interface declarations apart from code logic.
2. **Multiple Inclusion Defense** — Macro-based `#ifndef` include guards vs. compiler-supported `#pragma once`.
3. **Dependency Graph Optimization** — Forward declarations to resolve circular references and decrease compile times.
4. **Header Categories** — Managing project headers vs. standard library `<headers>` vs. third-party dependencies.
5. **Implementation Layout** — Correctly placing inline functions and template definitions directly inside headers.

---

## 1. Structural Isolation

### Definitions

**Core Definition**
Structural isolation is the practice of separating the *interface* of a module (what it provides to the outside world) from its *implementation* (how it works internally), with the interface placed in a header file and the implementation placed in one or more source files.

**Technical Definition**
In C++, structural isolation is achieved by declaring entities (functions, classes, templates) in a header file and defining them in a `.cpp` file. The compiler needs only the declarations to compile code that *uses* those entities; the linker resolves the actual definitions at link time. This separation enables independent compilation, faster incremental builds, and encapsulation of implementation details.

**Beginner-Friendly Explanation**
Think of a restaurant. The menu is the "header"—it tells you what dishes are available, their names, and prices. The kitchen is the "implementation"—it does the actual cooking. You don't need to know the recipe to order a dish. In C++, a header file is the menu: it tells other code what functions and classes exist. The `.cpp` file is the kitchen: it contains the actual code that makes those functions work.

---

### Purposes (all begin with "To")

- **To** allow multiple translation units to share a common interface without duplicating code.
- **To** enable separate compilation, where changing one `.cpp` file does not require recompiling unrelated files.
- **To** hide implementation details from consumers of a module, reducing coupling and improving maintainability.
- **To** define a stable public API that can evolve independently of internal implementation.
- **To** reduce compilation dependencies by allowing consumers to include only the declarations they need.

---

### Syntax Rules and Structure

#### Complete General Syntaxes with Breakdowns

**Syntax 1 — Header file declaration (interface)**
```cpp
// my_class.h
#ifndef MY_CLASS_H
#define MY_CLASS_H

class MyClass {
public:
    MyClass();
    ~MyClass();
    int compute(int x) const;
private:
    int value_;
};

#endif // MY_CLASS_H
```
**Component breakdown:**
- `#ifndef MY_CLASS_H` / `#define MY_CLASS_H` — include guard preventing multiple inclusion.
- `class MyClass { ... };` — the class declaration (interface): public methods, private data members (but not their initial values or method bodies).
- `#endif` — closes the include guard.

**Syntax 2 — Source file definition (implementation)**
```cpp
// my_class.cpp
#include "my_class.h"

MyClass::MyClass() : value_(0) {}

MyClass::~MyClass() {}

int MyClass::compute(int x) const {
    return x * value_;
}
```
**Component breakdown:**
- `#include "my_class.h"` — includes the corresponding header.
- `MyClass::MyClass()` — constructor definition.
- `MyClass::~MyClass()` — destructor definition.
- `int MyClass::compute(int x) const` — member function definition.

#### Syntax Rules

- A `.cpp` file that defines members of a class must include the header that declares the class.
- The header should include only what it needs (minimal includes) to reduce compile-time dependencies.
- Forward declarations should be preferred over `#include` when only a pointer or reference to a type is needed.
- The header should be self-contained: it should include all headers it needs to compile independently.
- Each `.cpp` file should include its corresponding header first, to verify self-containment.

#### Constraints and Limitations

- Non-inline functions and variables must be defined exactly once across all translation units (ODR).
- Headers cannot contain non-inline function definitions or non-const variable definitions (unless they are `inline` or `static`).
- Circular dependencies between headers must be resolved using forward declarations.
- Including unnecessary headers increases compile time and creates hidden dependencies.
- Private implementation details should not be exposed in public headers; use the Pimpl idiom or private headers for internal helpers.

---

### Multiple Annotated Complete Step-by-Step Code Examples and Their Expected Outputs

#### Example 1 — Basic Interface/Implementation Separation

```cpp
// File: calculator.h
// Step 1: Include guard to prevent multiple inclusion
#ifndef CALCULATOR_H
#define CALCULATOR_H

// Step 2: The class declaration (interface)
class Calculator {
public:
    // Step 3: Public member function declarations
    int add(int a, int b) const;
    int subtract(int a, int b) const;
    int multiply(int a, int b) const;

private:
    // Step 4: Private data member (no definition here)
    int last_result_;
};

#endif // CALCULATOR_H
```

```cpp
// File: calculator.cpp
// Step 5: Include the corresponding header first
#include "calculator.h"

// Step 6: Define the member functions
int Calculator::add(int a, int b) const {
    return a + b;
}

int Calculator::subtract(int a, int b) const {
    return a - b;
}

int Calculator::multiply(int a, int b) const {
    return a * b;
}
```

```cpp
// File: main.cpp
#include "calculator.h"
#include <iostream>

int main() {
    // Step 7: Use the class through its interface
    Calculator calc;
    std::cout << "5 + 3 = " << calc.add(5, 3)      << std::endl;
    std::cout << "5 - 3 = " << calc.subtract(5, 3) << std::endl;
    std::cout << "5 * 3 = " << calc.multiply(5, 3) << std::endl;
    return 0;
}
```

**Compile and run:**
```bash
g++ -std=c++17 calculator.cpp main.cpp -o calculator_app
./calculator_app
```

**Expected Output:**
```
5 + 3 = 8
5 - 3 = 2
5 * 3 = 15
```

**Why this output:** The header `calculator.h` declares the class and its methods but does not define them. The `.cpp` file defines the methods. The `main.cpp` file includes the header and calls the methods, which are resolved at link time. This separation allows `main.cpp` to be recompiled independently of `calculator.cpp` (as long as the interface does not change).

---

#### Example 2 — Self-Contained Header Verification

```cpp
// File: string_utils.h
// Step 1: Include guard
#ifndef STRING_UTILS_H
#define STRING_UTILS_H

// Step 2: Include all headers this header needs
#include <string>
#include <vector>

// Step 3: Declare the interface
std::vector<std::string> split(const std::string& str, char delimiter);
std::string join(const std::vector<std::string>& parts, const std::string& delim);

#endif // STRING_UTILS_H
```

```cpp
// File: string_utils.cpp
// Step 4: Include the header FIRST to verify self-containment
#include "string_utils.h"

#include <sstream>

std::vector<std::string> split(const std::string& str, char delimiter) {
    std::vector<std::string> result;
    std::stringstream ss(str);
    std::string item;
    while (std::getline(ss, item, delimiter)) {
        result.push_back(item);
    }
    return result;
}

std::string join(const std::vector<std::string>& parts, const std::string& delim) {
    std::string result;
    for (size_t i = 0; i < parts.size(); ++i) {
        if (i > 0) result += delim;
        result += parts[i];
    }
    return result;
}
```

```cpp
// File: main.cpp
#include "string_utils.h"
#include <iostream>

int main() {
    auto parts = split("apple,banana,cherry", ',');
    for (const auto& p : parts) {
        std::cout << p << std::endl;
    }
    std::cout << join(parts, " | ") << std::endl;
    return 0;
}
```

**Compile and run:**
```bash
g++ -std=c++17 string_utils.cpp main.cpp -o string_app
./string_app
```

**Expected Output:**
```
apple
banana
cherry
apple | banana | cherry
```

**Why this output:** The header `string_utils.h` includes `<string>` and `<vector>` itself, making it self-contained. The `.cpp` file includes its own header first, which verifies that the header does not depend on hidden includes. If `string_utils.h` forgot to include `<string>`, the `.cpp` file would fail to compile when it includes the header first.

---

### Real-World Cases with Explanation

**Case 1 — Standard library organization:** The C++ Standard Library is organized as a collection of headers (`<vector>`, `<string>`, `<algorithm>`) that declare the interfaces of library components. Implementations reside in the library's compiled binary. This allows millions of programs to use the same library without duplicating code.

**Case 2 — Pimpl idiom (Pointer to Implementation):** A widely used structural isolation technique where the public header contains only a pointer to an incomplete implementation class. The implementation class is defined entirely in the `.cpp` file, completely hiding private data and reducing compile-time dependencies. This is especially useful for ABI-stable libraries.

**Case 3 — LLVM coding standards:** LLVM's Coding Standards explicitly state that "modules should be completely independent of each other, and their header files should only `#include` the absolute minimum number of headers possible". This principle is applied throughout the LLVM codebase to keep compilation fast and dependencies minimal.

**Case 4 — Private headers for internal communication:** When a module has multiple `.cpp` files that need to share internal helpers, LLVM recommends placing a *private* header in the same directory as the source files (not in the public include directory) and including it locally. This keeps the private interface hidden from external consumers.

---

### References Links

- C++ Core Guidelines SF.2: A header file must not contain object definitions or non-inline function definitions - https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rs-inline
- LLVM Coding Standards: Header Files - https://llvm.org/docs/CodingStandards.html
- C++ FAQ: What's the difference between a declaration and a definition? - https://isocpp.org/wiki/faq/classes-and-objects#declaration-vs-definition

---

## 2. Multiple Inclusion Defense

### Definitions

**Core Definition**
Multiple inclusion defense refers to the mechanisms used to ensure that a header file's contents are compiled only once per translation unit, even if the header is `#include`d multiple times.

**Technical Definition**
The two primary mechanisms are **include guards** (macro-based `#ifndef`/`#define`/`#endif` directives) and **`#pragma once`** (a non-standard compiler directive). Include guards rely on the preprocessor's conditional compilation to check whether a unique macro has been defined; if not, the macro is defined and the header contents are processed. `#pragma once` instructs the compiler to skip the file entirely if it has already been included, using the file's identity (inode, path) rather than a macro.

**Beginner-Friendly Explanation**
Imagine you're handing out flyers at a concert. If someone already has a flyer, you don't give them another one—that would be wasteful and confusing. In C++, if a header file is included twice (e.g., because two other headers both include it), the compiler would see the same declarations twice, which is an error. Include guards and `#pragma once` are like a bouncer at the door: "Have you already seen this file? Yes? Then skip it."

---

### Purposes (all begin with "To")

- **To** prevent redefinition errors caused by including the same header multiple times in one translation unit.
- **To** avoid violations of the One Definition Rule (ODR).
- **To** reduce preprocessing time by skipping already-processed files.
- **To** allow headers to include each other without infinite recursion.
- **To** ensure that a header is self-contained and can be included in any order.

---

### Syntax Rules and Structure

#### Complete General Syntaxes with Breakdowns

**Syntax 1 — Include guard (`#ifndef`)**
```cpp
#ifndef UNIQUE_MACRO_NAME
#define UNIQUE_MACRO_NAME

// header contents

#endif // UNIQUE_MACRO_NAME
```
**Component breakdown:**
- `#ifndef UNIQUE_MACRO_NAME` — checks whether the macro is *not* defined.
- `#define UNIQUE_MACRO_NAME` — defines the macro so subsequent inclusions are skipped.
- `// header contents` — the declarations to be included once.
- `#endif` — closes the conditional block.

**Syntax 2 — `#pragma once`**
```cpp
#pragma once

// header contents
```
**Component breakdown:**
- `#pragma once` — a non-standard directive instructing the compiler to include the file only once per translation unit.

#### Syntax Rules

- The include guard macro name must be unique across the entire project to avoid collisions.
- The macro name should include the package name, class name, and possibly the project name (e.g., `LIBRARY_FOOBAR_H`).
- The macro name must not begin with an underscore followed by an uppercase letter (reserved for the implementation).
- `#pragma once` must appear at the very top of the file (before any other code) to be effective.
- `#pragma once` is not part of the C++ standard but is supported by all major compilers (GCC, Clang, MSVC).

#### Constraints and Limitations

- **Include guards**: If the same header is reachable via different paths (e.g., symbolic links or different include directories), the guard still works because the macro check is path-independent. However, if two different files use the same guard macro, one may be silently skipped.
- **`#pragma once`**: If the same file is reachable via different paths (e.g., through a symlink), some compilers may not recognize it as the same file and include it twice. This can defeat the optimization.
- **Performance**: Historically, `#pragma once` was believed to be faster. However, modern compilers implement the "multiple-include optimization" for include guards as well, making performance effectively equivalent. Some older versions of GCC were actually *slower* with `#pragma once`.
- **Portability**: `#pragma once` is non-standard. If strict portability to all compilers is required, include guards are the safer choice. The C++ Core Guidelines recommend include guards over `#pragma once` for this reason.

---

### Multiple Annotated Complete Step-by-Step Code Examples and Their Expected Outputs

#### Example 1 — Include Guards in Action

```cpp
// File: math_utils.h
// Step 1: Include guard with a unique macro name
#ifndef MATH_UTILS_H
#define MATH_UTILS_H

// Step 2: Header contents
int add(int a, int b);
int subtract(int a, int b);

#endif // MATH_UTILS_H
```

```cpp
// File: physics_utils.h
// Step 1: Include guard
#ifndef PHYSICS_UTILS_H
#define PHYSICS_UTILS_H

// Step 2: This header needs math_utils.h
#include "math_utils.h"

double calculate_velocity(double distance, double time);

#endif // PHYSICS_UTILS_H
```

```cpp
// File: math_utils.cpp
#include "math_utils.h"

int add(int a, int b) { return a + b; }
int subtract(int a, int b) { return a - b; }
```

```cpp
// File: physics_utils.cpp
#include "physics_utils.h"

double calculate_velocity(double distance, double time) {
    return distance / time;
}
```

```cpp
// File: main.cpp
#include "math_utils.h"      // First inclusion — contents processed
#include "physics_utils.h"   // Includes math_utils.h again
#include "math_utils.h"      // Third inclusion — skipped by guard
#include <iostream>

int main() {
    std::cout << "add(2, 3) = " << add(2, 3) << std::endl;
    std::cout << "velocity  = " << calculate_velocity(100.0, 10.0) << std::endl;
    return 0;
}
```

**Compile and run:**
```bash
g++ -std=c++17 math_utils.cpp physics_utils.cpp main.cpp -o app
./app
```

**Expected Output:**
```
add(2, 3) = 5
velocity  = 10
```

**Why this output:** The include guard `MATH_UTILS_H` is defined the first time `math_utils.h` is included. When `physics_utils.h` includes it again, the preprocessor sees that `MATH_UTILS_H` is already defined and skips the contents. The third inclusion in `main.cpp` is also skipped. Without the guard, `add` and `subtract` would be declared multiple times, causing a redefinition error.

---

#### Example 2 — `#pragma once` Equivalent

```cpp
// File: math_utils_pragma.h
// Step 1: Use #pragma once instead of include guards
#pragma once

// Step 2: Header contents
int multiply(int a, int b);
int divide(int a, int b);
```

```cpp
// File: main_pragma.cpp
#include "math_utils_pragma.h"
#include "math_utils_pragma.h"  // Skipped by #pragma once
#include <iostream>

int main() {
    std::cout << "multiply(4, 5) = " << multiply(4, 5) << std::endl;
    return 0;
}
```

```cpp
// File: math_utils_pragma.cpp
#include "math_utils_pragma.h"
int multiply(int a, int b) { return a * b; }
int divide(int a, int b) { return b != 0 ? a / b : 0; }
```

**Compile and run:**
```bash
g++ -std=c++17 math_utils_pragma.cpp main_pragma.cpp -o pragma_app
./pragma_app
```

**Expected Output:**
```
multiply(4, 5) = 20
```

**Why this output:** `#pragma once` instructs the compiler to include the file only once. The second `#include` in `main_pragma.cpp` is skipped entirely. The result is identical to using include guards, but with less code and no risk of macro name collisions.

---

#### Example 3 — Performance Comparison (Conceptual)

```cpp
// File: heavy_header.h
// This header simulates a heavy header with many declarations
#ifndef HEAVY_HEADER_H
#define HEAVY_HEADER_H

// Simulate 1000 lines of declarations
// (in practice, this would be real content)
#define HEAVY_CONTENT_1 1
#define HEAVY_CONTENT_2 2
// ... (repeated 1000 times)

#endif // HEAVY_HEADER_H
```

```cpp
// File: test_guards.cpp
#include "heavy_header.h"
#include "heavy_header.h"  // Skipped by include guard
#include "heavy_header.h"  // Skipped by include guard
// ... (100 more includes)

int main() { return 0; }
```

**Expected behavior:** Modern compilers implement the "multiple-include optimization" for include guards. When the compiler encounters `#ifndef HEAVY_HEADER_H`, it checks whether the macro is defined. If it is, the compiler skips to the matching `#endif` without reading the entire file. This makes include guards effectively as fast as `#pragma once` in modern compilers.

**Why this matters:** The historical performance advantage of `#pragma once` over include guards has largely disappeared. Modern compilers (GCC, Clang, MSVC) all implement the multiple-include optimization for properly formatted include guards. The choice between the two is now primarily a matter of portability and style, not performance.

---

### Real-World Cases with Explanation

**Case 1 — CERN's ATLAS software:** The ATLAS experiment's coding guidelines state: "Header files must begin and end with multiple-inclusion protection." They recommend include guards over `#pragma once` because `#pragma once` is non-standard and "has no compelling advantage" on modern compilers. The guard name should include the package name and class name to ensure uniqueness.

**Case 2 — LLVM and Clang:** Both LLVM and Clang use `#pragma once` in many of their headers, reflecting a pragmatic choice for internal code where all supported compilers implement the directive. However, for public-facing headers that must be portable, include guards are preferred.

**Case 3 — The C++ Core Guidelines:** SF.8 states: "Use `#include` guards for all header files." The reason given is to avoid files being `#include`d several times. The guideline notes that `#pragma once` is not standard and is not portable.

**Case 4 — Google Style Guide:** Google's C++ Style Guide requires the use of include guards with the format `<PROJECT>_<PATH>_<FILE>_H_`. For example, a header `foo/bar/baz.h` in project `foo` would use the guard `FOO_BAR_BAZ_H_`. This convention ensures global uniqueness and prevents guard collisions.

**Case 5 — Microsoft's recommendation:** Microsoft recommends `#pragma once` for new code because "it doesn't pollute the global namespace with a preprocessor symbol. It requires less typing, it's less distracting, and it can't cause symbol collisions". However, they also acknowledge that include guards should be used when portability to compilers that don't support `#pragma once` is required.

---

### References Links

- `#pragma once` — Microsoft Learn - https://learn.microsoft.com/en-us/cpp/preprocessor/once
- C++ Core Guidelines SF.8: Use `#include` guards for all header files - https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rs-guards
- CERN ATLAS Coding Guidelines: Header Guards - https://atlas-software.docs.cern.ch/coding-guidelines/rules-coding/
- Include guard — cppreference.com - https://en.cppreference.com/w/cpp/preprocessor/include

---

## 3. Dependency Graph Optimization (Forward Declarations)

### Definitions

**Core Definition**
A forward declaration is a declaration of an identifier (typically a class, struct, or function) without its full definition. It tells the compiler that a type or function exists, without providing the details needed to use it fully.

**Technical Definition**
A forward declaration for a class has the form `class ClassName;`. It introduces the class name into the current scope and allows the declaration of pointers and references to the class, as well as function parameters and return types that use the class by pointer or reference. It does **not** allow use of the class's members, inheritance from it, or creation of objects of that type, because the compiler lacks the full definition.

**Beginner-Friendly Explanation**
A forward declaration is like a promise: "I don't know the details yet, but I promise there's a class called `Foo`." It's enough to say "I'll take a `Foo*` pointer" without knowing what's inside `Foo`. This is useful because it lets two classes refer to each other without one needing to include the other's full header. It also speeds up compilation because the compiler doesn't have to process a large header file just to know that a class exists.

---

### Purposes (all begin with "To")

- **To** break circular dependencies between headers that would otherwise include each other infinitely.
- **To** reduce compile times by avoiding unnecessary header inclusions.
- **To** minimize the dependency graph, so that a change in one header does not force recompilation of unrelated files.
- **To** allow a header to declare functions that take or return pointers/references to types without including their full definitions.
- **To** enable the use of pointers and references to types that are only partially known.

---

### Syntax Rules and Structure

#### Complete General Syntax with Breakdown

```cpp
// Forward declaration of a class
class ClassName;

// Forward declaration of a struct
struct StructName;

// Forward declaration of a function
return_type function_name(parameter_types);
```

**Component breakdown:**
- `class ClassName;` — declares `ClassName` as a class without defining its members.
- `struct StructName;` — declares `StructName` as a struct.
- `return_type function_name(parameter_types);` — declares a function's signature without its body.

#### Syntax Rules

- A forward declaration must appear before any use of the name.
- The name must be declared consistently (e.g., `class` vs. `struct`; using `struct` for a class declared with `class` is technically allowed but can trigger warnings).
- A forward-declared type can be used as a pointer or reference, but not as a value type (you cannot create an object of an incomplete type).
- A forward-declared function can be called if it is defined elsewhere; the compiler needs only the signature.
- Forward declarations should be placed in the header file that needs them, not in the `.cpp` file, if the header's interface uses the forward-declared type.

#### Constraints and Limitations

- You cannot access members of a forward-declared class.
- You cannot use a forward-declared class as a base class.
- You cannot create an object of a forward-declared type (the size is unknown).
- You cannot use `sizeof` or `alignof` on an incomplete type.
- Forward declarations of typedefs and type aliases are not possible; you must include the header that defines the alias.
- Templates cannot be forward-declared in the same way as classes; you need to declare the template with its parameters.

---

### Multiple Annotated Complete Step-by-Step Code Examples and Their Expected Outputs

#### Example 1 — Breaking Circular Dependencies

```cpp
// File: entity.h
// Step 1: Forward declaration of Component (needed for the pointer)
class Component;  // <-- forward declaration

class Entity {
public:
    Entity();
    ~Entity();
    void set_component(Component* comp);  // pointer — OK with forward declaration
    Component* get_component() const;     // pointer — OK

private:
    Component* component_;  // pointer — OK with incomplete type
};
```

```cpp
// File: component.h
// Step 1: Forward declaration of Entity (needed for the pointer)
class Entity;  // <-- forward declaration

class Component {
public:
    Component();
    ~Component();
    void set_owner(Entity* owner);  // pointer — OK
    Entity* get_owner() const;      // pointer — OK

private:
    Entity* owner_;  // pointer — OK with incomplete type
};
```

```cpp
// File: entity.cpp
#include "entity.h"
#include "component.h"  // full definition needed here for method bodies

Entity::Entity() : component_(nullptr) {}
Entity::~Entity() {}

void Entity::set_component(Component* comp) {
    component_ = comp;
}

Component* Entity::get_component() const {
    return component_;
}
```

```cpp
// File: component.cpp
#include "component.h"
#include "entity.h"  // full definition needed here

Component::Component() : owner_(nullptr) {}
Component::~Component() {}

void Component::set_owner(Entity* owner) {
    owner_ = owner;
}

Entity* Component::get_owner() const {
    return owner_;
}
```

```cpp
// File: main.cpp
#include "entity.h"
#include "component.h"
#include <iostream>

int main() {
    Entity e;
    Component c;
    e.set_component(&c);
    c.set_owner(&e);

    std::cout << "Entity has component: "
              << (e.get_component() != nullptr ? "yes" : "no") << std::endl;
    std::cout << "Component has owner: "
              << (c.get_owner() != nullptr ? "yes" : "no") << std::endl;
    return 0;
}
```

**Compile and run:**
```bash
g++ -std=c++17 entity.cpp component.cpp main.cpp -o entity_app
./entity_app
```

**Expected Output:**
```
Entity has component: yes
Component has owner: yes
```

**Why this output:** The headers `entity.h` and `component.h` use forward declarations to break the circular dependency. Each header needs only a pointer to the other class, not the full definition. The `.cpp` files include both headers to access the full definitions when implementing method bodies. Without forward declarations, `entity.h` would need to include `component.h`, and `component.h` would need to include `entity.h`, creating an infinite inclusion loop.

---

#### Example 2 — Reducing Compile-Time Dependencies

```cpp
// File: database.h
// This header is large and slow to parse
#ifndef DATABASE_H
#define DATABASE_H
#include <string>
#include <vector>
#include <map>

class Database {
public:
    Database();
    void connect(const std::string& conn_str);
    std::vector<std::string> query(const std::string& sql);
private:
    std::map<std::string, std::string> config_;
};
#endif
```

```cpp
// File: user_service.h
// BAD: includes database.h unnecessarily
// #include "database.h"  // <-- This would slow compilation

// GOOD: forward declaration — no need for the full Database definition
class Database;  // <-- forward declaration

class UserService {
public:
    UserService(Database* db);  // pointer — OK with forward declaration
    void create_user(const std::string& name);
private:
    Database* db_;  // pointer — OK
};
```

```cpp
// File: user_service.cpp
#include "user_service.h"
#include "database.h"  // full definition needed here

UserService::UserService(Database* db) : db_(db) {}

void UserService::create_user(const std::string& name) {
    db_->query("INSERT INTO users (name) VALUES ('" + name + "')");
}
```

**Why this is better:** `user_service.h` does not include `database.h`, so any file that includes `user_service.h` does not pay the cost of parsing `database.h`. Only `user_service.cpp` includes the full `database.h` definition. If `database.h` changes, only `user_service.cpp` (and files that directly include `database.h`) need to be recompiled. This can dramatically reduce build times in large projects.

---

### Real-World Cases with Explanation

**Case 1 — CERN's ATLAS software:** The ATLAS coding guidelines explicitly recommend: "Use forward declaration instead of including a header file, if this is sufficient." The example given is a `Point` class that takes a `const Line&` parameter—forward declaring `Line` avoids including the `Line` header entirely.

**Case 2 — LLVM's `#include` philosophy:** LLVM's Coding Standards state: "`#include` hurts compile time performance. Don't do it unless you have to, especially in header files." They note that if you are using a pointer or reference to a class, you don't need the full definition—a forward declaration suffices.

**Case 3 — Large-scale game engines:** Game engines often have thousands of headers with complex interdependencies. Forward declarations are used extensively to break cycles between subsystems (e.g., `Renderer` needs `Scene*`, `Scene` needs `Renderer*`), keeping compile times manageable.

**Case 4 — The Pimpl idiom:** The Pimpl (Pointer to Implementation) idiom uses a forward declaration of the implementation class in the header. The header contains only `class Impl;` and a pointer `Impl* pimpl_;`. The full definition of `Impl` is in the `.cpp` file. This completely hides implementation details and reduces compile-time dependencies.

**Case 5 — Google Style Guide:** Google's C++ Style Guide states: "Use forward declarations to minimize the number of `#include`s in a header file." It recommends using forward declarations when a header only needs a pointer or reference to a class, rather than including the full definition.

---

### References Links

- Forward declaration — cppreference.com - https://en.cppreference.com/w/cpp/language/elaborated_type_specifier
- C++ Core Guidelines SF.3: Use `.h` files for all declarations used in multiple source files - https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rs-declaration-header
- LLVM Coding Standards: `#include` as Little as Possible - https://llvm.org/docs/CodingStandards.html
- CERN ATLAS Coding Guidelines: Forward Declarations - https://atlas-software.docs.cern.ch/coding-guidelines/rules-coding/

---

## 4. Header Categories

### Definitions

**Core Definition**
Header categories refer to the classification of headers into three groups — **standard library headers**, **project headers**, and **third-party headers** — each with distinct include syntax, search path rules, and management strategies.

**Technical Definition**
- **Standard library headers** (e.g., `<vector>`, `<string>`, `<algorithm>`) are provided by the C++ implementation and are included using angle brackets `<...>`. They are searched in implementation-defined system directories.
- **Project headers** (e.g., `"my_class.h"`) are part of the current project and are included using double quotes `"..."`. They are searched first in the directory of the including file, then in the project's include paths.
- **Third-party headers** (e.g., `"nlohmann/json.hpp"`) are external dependencies. They may be included with either syntax depending on the build system configuration, but are typically included with double quotes or angle brackets with custom include paths.

**Beginner-Friendly Explanation**
Think of headers as different types of mail. Standard library headers are like public utilities (water, electricity) — you get them from the city (`<...>`). Project headers are like mail within your own house — you know exactly where they are (`"..."`). Third-party headers are like packages from an online store — they come from outside, and you need to tell your build system where to find them.

---

### Purposes (all begin with "To")

- **To** distinguish between headers provided by the language implementation and those provided by the project.
- **To** control the preprocessor's search order for header files.
- **To** ensure that the correct version of a header is found when multiple versions exist.
- **To** organize the project's build system include paths correctly.
- **To** prevent accidental inclusion of a project header when a standard library header with the same name is intended (or vice versa).

---

### Syntax Rules and Structure

#### Complete General Syntaxes with Breakdowns

**Syntax 1 — Standard library header inclusion**
```cpp
#include <vector>
#include <string>
#include <algorithm>
```
- Uses angle brackets `<...>`.
- The preprocessor searches implementation-defined system include directories.
- The current file's directory is **not** searched.

**Syntax 2 — Project header inclusion**
```cpp
#include "my_class.h"
#include "utils/string_utils.h"
```
- Uses double quotes `"..."`.
- The preprocessor searches the current file's directory first, then the project's include paths.
- If not found, it falls back to the system include path.

**Syntax 3 — Third-party header inclusion**
```cpp
#include <nlohmann/json.hpp>
#include "third_party/foo/foo.h"
```
- May use either syntax depending on how the third-party library is installed and configured.
- Often uses angle brackets if the library is installed in a system-wide location, or double quotes if it is vendored in the project.

#### Syntax Rules

- Use angle brackets for standard library headers and headers installed system-wide.
- Use double quotes for project headers and third-party headers vendored in the project.
- The include order should be: **own header** → **project headers** → **standard library headers** → **system headers** (as recommended by LLVM).
- Group includes logically and separate groups with blank lines (Google Style Guide).
- Each `.cpp` file should include its own header first to verify self-containment.

#### Constraints and Limitations

- The search order for `#include "..."` is implementation-defined after the current directory; some compilers search the system path, others do not.
- Using angle brackets for a project header may fail if the project's include path is not configured correctly.
- Using double quotes for a standard library header is technically allowed but considered poor style and may cause unexpected search behavior.
- The preprocessor does not understand namespaces or modules; it operates purely on file paths.
- In C++20 modules, the distinction between header categories is different: modules use `import` instead of `#include`, and the standard library can be imported as a module.

---

### Multiple Annotated Complete Step-by-Step Code Examples and Their Expected Outputs

#### Example 1 — Correct Include Order and Syntax

```cpp
// File: my_class.cpp
// Step 1: Include own header first (verifies self-containment)
#include "my_class.h"

// Step 2: Project headers (double quotes)
#include "utils/string_utils.h"
#include "logging/logger.h"

// Step 3: Standard library headers (angle brackets)
#include <string>
#include <vector>
#include <iostream>

// Step 4: System headers (angle brackets, platform-specific)
#include <unistd.h>  // POSIX — not available on Windows

// Step 5: Implementation
MyClass::MyClass() {}

void MyClass::process() {
    std::vector<std::string> parts = split("a,b,c", ',');
    for (const auto& p : parts) {
        Logger::log(p);
    }
}
```

**Why this order:** Including your own header first ensures that it is self-contained — if it forgets to include something it needs, the compiler will error here rather than in a consumer's file. Project headers come next, followed by standard library headers. System headers come last because they are platform-specific and may not be portable.

---

#### Example 2 — Third-Party Header Management

```cpp
// File: main.cpp
// Option A: Third-party header vendored in the project (double quotes)
#include "third_party/nlohmann/json.hpp"

// Option B: Third-party header installed system-wide (angle brackets)
// #include <nlohmann/json.hpp>

#include <iostream>
#include <fstream>

int main() {
    // Use the third-party library
    nlohmann::json j;
    j["name"] = "Alice";
    j["age"] = 30;

    std::cout << j.dump(2) << std::endl;
    return 0;
}
```

**Compile and run:**
```bash
# If json.hpp is in ./third_party/nlohmann/
g++ -std=c++17 main.cpp -o json_app

# If json.hpp is installed in /usr/local/include/nlohmann/
g++ -std=c++17 -I/usr/local/include main.cpp -o json_app
```

**Expected Output:**
```json
{
  "age": 30,
  "name": "Alice"
}
```

**Why this output:** The third-party header is included with double quotes because it is vendored in the project directory. The build system knows where to find it because the compiler searches the current directory first. If the library were installed system-wide, angle brackets and an explicit include path (`-I`) would be used instead.

---

### Real-World Cases with Explanation

**Case 1 — Google Style Guide include order:** Google recommends the following include order: (1) related header, (2) C system headers, (3) C++ standard library headers, (4) other libraries' headers, (5) your project's headers. This order ensures that your header is self-contained and that dependencies are resolved in a predictable order.

**Case 2 — CMake's `target_include_directories`:** Modern CMake projects use `target_include_directories` to specify include paths for project and third-party headers. System headers are automatically searched by the compiler. This separates the concerns of "where to find headers" from the code itself.

**Case 3 — Vendored dependencies:** Many projects vendor third-party libraries (copy them into the project's `third_party/` or `external/` directory) to ensure reproducible builds. These are included with double quotes relative to the project root. Examples: `"third_party/googletest/include/gtest/gtest.h"`.

**Case 4 — Package managers (vcpkg, Conan):** These tools install third-party libraries into a known location and configure the build system's include paths. Headers are then included with angle brackets (e.g., `<nlohmann/json.hpp>`) or double quotes depending on the package's convention.

**Case 5 — LLVM's include ordering:** LLVM's Coding Standards state: "System headers should be included after user headers for a translation unit." This prevents system headers from defining macros that accidentally affect user code.

---

### References Links

- C++ standard library header files — Microsoft Learn - https://learn.microsoft.com/en-us/cpp/standard-library/cpp-standard-library-header-files
- Google C++ Style Guide: Include Order - https://google.github.io/styleguide/cppguide.html#Include_Order
- LLVM Coding Standards: Include Order - https://llvm.org/docs/CodingStandards.html
- `#include` directive — cppreference.com - https://en.cppreference.com/w/cpp/preprocessor/include

---

## 5. Implementation Layout (Inline Functions and Templates)

### Definitions

**Core Definition**
Implementation layout refers to the placement of function and template definitions within header files, as opposed to `.cpp` files, to satisfy the One Definition Rule (ODR) and enable instantiation across translation units.

**Technical Definition**
Inline functions and templates are exempt from the ODR's single-definition requirement: they may be defined in multiple translation units, provided all definitions are identical and are present in every translation unit in which they are odr-used. Because the compiler must see the full definition to inline a function or instantiate a template, these definitions are typically placed in header files.

**Beginner-Friendly Explanation**
Normally, a function is defined once in a `.cpp` file. But inline functions and templates are special: they can be defined in a header file and included in many `.cpp` files. Why? Because the compiler needs to see the *full code* of an inline function or template to generate the machine code for it in each translation unit. If the definition were hidden in a `.cpp` file, other `.cpp` files couldn't use it. So inline functions and templates "live" in headers.

---

### Purposes (all begin with "To")

- **To** satisfy the ODR requirement that inline functions and templates have identical definitions in every translation unit where they are used.
- **To** make template definitions visible to the compiler at the point of instantiation.
- **To** enable cross-translation-unit inlining of small functions.
- **To** allow header-only libraries (libraries that consist entirely of headers) to work correctly.
- **To** eliminate linker errors caused by missing template definitions.

---

### Syntax Rules and Structure

#### Complete General Syntaxes with Breakdowns

**Syntax 1 — Inline function definition in a header**
```cpp
// utils.h
inline int square(int x) {
    return x * x;
}
```
**Component breakdown:**
- `inline` — keyword indicating the function is inline (allows multiple definitions across translation units).
- `int square(int x)` — function signature.
- `{ return x * x; }` — function body (definition).

**Syntax 2 — Template function definition in a header**
```cpp
// utils.h
template <typename T>
T max(T a, T b) {
    return (a > b) ? a : b;
}
```
**Component breakdown:**
- `template <typename T>` — template parameter declaration.
- `T max(T a, T b)` — function template signature.
- `{ return (a > b) ? a : b; }` — function body.

**Syntax 3 — Class template definition in a header**
```cpp
// my_vector.h
template <typename T>
class MyVector {
public:
    void push_back(const T& value);
    T& operator[](size_t index);
private:
    std::vector<T> data_;
};

// Member function definitions (must be in header)
template <typename T>
void MyVector<T>::push_back(const T& value) {
    data_.push_back(value);
}

template <typename T>
T& MyVector<T>::operator[](size_t index) {
    return data_[index];
}
```
**Component breakdown:**
- `template <typename T>` — template parameter.
- `class MyVector { ... }` — class template declaration.
- `template <typename T> void MyVector<T>::push_back(...)` — member function definition (must be in header).

#### Syntax Rules

- Inline functions must be defined in every translation unit where they are odr-used.
- Template definitions must be visible at the point of instantiation.
- The `inline` keyword is a hint to the compiler, not a guarantee that the function will be inlined.
- Functions defined inside a class body are implicitly `inline`.
- Member functions of class templates defined outside the class body must be marked with `template <...>` and must appear in a header.
- The `inline` keyword on a function template is redundant but allowed.

#### Constraints and Limitations

- Inline function definitions must be identical across all translation units. Different definitions in different files cause undefined behavior.
- Templates cannot be defined in a `.cpp` file and used in another `.cpp` file unless explicit instantiation is used (for all required types).
- Defining large inline functions in headers can increase compile times and code bloat.
- Google Style Guide: "When a header declares inline functions or templates that clients of the header will instantiate, the inline functions and templates must also have definitions in the header, either directly or in files it includes".
- Separating template definitions into `-inl.h` files was common in the past but is no longer recommended by Google Style Guide.
- Oracle Developer Studio: "Because template definitions often appear in header files, a template definition might be repeated in several compilation units. All definitions must be the same".

---

### Multiple Annotated Complete Step-by-Step Code Examples and Their Expected Outputs

#### Example 1 — Inline Function in a Header

```cpp
// File: math_utils.h
#ifndef MATH_UTILS_H
#define MATH_UTILS_H

// Step 1: Inline function definition in the header
inline int square(int x) {
    return x * x;
}

// Step 2: Another inline function
inline int cube(int x) {
    return x * x * x;
}

#endif // MATH_UTILS_H
```

```cpp
// File: main.cpp
#include "math_utils.h"
#include <iostream>

int main() {
    // Step 3: Use the inline functions
    std::cout << "square(5) = " << square(5) << std::endl;
    std::cout << "cube(3) = " << cube(3) << std::endl;
    return 0;
}
```

```cpp
// File: other.cpp
#include "math_utils.h"

// Step 4: Other translation unit also uses the inline functions
int use_square(int x) {
    return square(x);
}
```

**Compile and run:**
```bash
g++ -std=c++17 main.cpp other.cpp -o inline_app
./inline_app
```

**Expected Output:**
```
square(5) = 25
cube(3) = 27
```

**Why this output:** The inline functions `square` and `cube` are defined in the header, so both `main.cpp` and `other.cpp` have identical definitions. The linker accepts multiple definitions because they are marked `inline`. If the functions were not inline, the linker would report a "multiple definition" error.

---

#### Example 2 — Template Function in a Header

```cpp
// File: generic_utils.h
#ifndef GENERIC_UTILS_H
#define GENERIC_UTILS_H

#include <string>
#include <sstream>

// Step 1: Template function definition in the header
template <typename T>
std::string to_string(const T& value) {
    std::ostringstream oss;
    oss << value;
    return oss.str();
}

#endif // GENERIC_UTILS_H
```

```cpp
// File: main.cpp
#include "generic_utils.h"
#include <iostream>

int main() {
    // Step 2: Instantiate the template with different types
    std::cout << to_string(42) << std::endl;
    std::cout << to_string(3.14) << std::endl;
    std::cout << to_string(std::string("hello")) << std::endl;
    return 0;
}
```

**Compile and run:**
```bash
g++ -std=c++17 main.cpp -o template_app
./template_app
```

**Expected Output:**
```
42
3.14
hello
```

**Why this output:** The template function `to_string` is defined in the header. When `main.cpp` uses `to_string(42)`, the compiler instantiates the template with `T = int`. When it uses `to_string(3.14)`, it instantiates with `T = double`. The definitions are generated in `main.cpp`'s translation unit. If the template were defined in a `.cpp` file, the compiler would not see the definition and would fail to link.

---

#### Example 3 — Class Template with Member Definitions in Header

```cpp
// File: my_stack.h
#ifndef MY_STACK_H
#define MY_STACK_H

#include <vector>
#include <stdexcept>

// Step 1: Class template declaration
template <typename T>
class MyStack {
public:
    void push(const T& value);
    T pop();
    bool empty() const;
    size_t size() const;

private:
    std::vector<T> data_;
};

// Step 2: Member function definitions (must be in header)
template <typename T>
void MyStack<T>::push(const T& value) {
    data_.push_back(value);
}

template <typename T>
T MyStack<T>::pop() {
    if (data_.empty()) {
        throw std::runtime_error("Stack is empty");
    }
    T value = data_.back();
    data_.pop_back();
    return value;
}

template <typename T>
bool MyStack<T>::empty() const {
    return data_.empty();
}

template <typename T>
size_t MyStack<T>::size() const {
    return data_.size();
}

#endif // MY_STACK_H
```

```cpp
// File: main.cpp
#include "my_stack.h"
#include <iostream>

int main() {
    MyStack<int> stack;
    stack.push(1);
    stack.push(2);
    stack.push(3);

    std::cout << "Size: " << stack.size() << std::endl;
    std::cout << "Popped: " << stack.pop() << std::endl;
    std::cout << "Popped: " << stack.pop() << std::endl;
    std::cout << "Size after pops: " << stack.size() << std::endl;

    return 0;
}
```

**Compile and run:**
```bash
g++ -std=c++17 main.cpp -o stack_app
./stack_app
```

**Expected Output:**
```
Size: 3
Popped: 3
Popped: 2
Size after pops: 1
```

**Why this output:** The class template `MyStack` and all its member function definitions are in the header. When `main.cpp` instantiates `MyStack<int>`, the compiler generates the code for `push`, `pop`, `empty`, and `size` for `int`. If the member definitions were in a `.cpp` file, the compiler would not be able to instantiate them for `int`, resulting in linker errors.

---

### Real-World Cases with Explanation

**Case 1 — Standard Library headers:** The C++ Standard Library is almost entirely template-based and header-only. Headers like `<vector>`, `<map>`, and `<algorithm>` contain full template definitions because the compiler needs them to instantiate the templates for the user's types.

**Case 2 — Header-only libraries:** Libraries like {fmt}, nlohmann/json, and Catch2 are header-only. They consist entirely of header files containing inline functions and templates. Users simply `#include` the header and get full functionality without linking a separate library.

**Case 3 — Google Style Guide on inline/template definitions:** Google's C++ Style Guide states: "When a header declares inline functions or templates that clients of the header will instantiate, the inline functions and templates must also have definitions in the header, either directly or in files it includes". The guide explicitly disallows the old practice of placing template definitions in separate `-inl.h` files.

**Case 4 — Explicit instantiation as an alternative:** If a template is used with only a known set of types, it can be explicitly instantiated in a `.cpp` file (e.g., `template class MyStack<int>;`), and the definition can be placed in that `.cpp` file. This reduces compile times for other translation units. This is common in libraries that want to hide template implementations.

**Case 5 — Oracle Developer Studio recommendation:** Oracle's C++ User's Guide states: "The simplest and most common way to make template definitions visible throughout a translation unit, is to put the definitions in the header file itself. Any `.cpp` file that uses the template simply has to `#include` the header. This approach is used in the Standard Library".

---

### References Links

- Inline functions and the ODR — Microsoft Learn - https://learn.microsoft.com/en-us/cpp/cpp/inline-functions-cpp
- Source code organization (C++ Templates) — Microsoft Learn - https://learn.microsoft.com/en-us/cpp/cpp/source-code-organization-cpp-templates
- Google C++ Style Guide: Inline Functions and Templates - https://google.github.io/styleguide/cppguide.html#Inline_Functions
- One Definition Rule — cppreference.com - https://en.cppreference.com/w/cpp/language/definition

---

## Summary Table of Core Concepts

| Concept | Key Mechanism | Primary Benefit | Key Risk / Constraint |
|---|---|---|---|
| Structural Isolation | Header declares, `.cpp` defines | Separate compilation, encapsulation | ODR violations if definitions in headers |
| Multiple Inclusion Defense | `#ifndef` guards or `#pragma once` | Prevents redefinition errors | `#pragma once` is non-standard; include guards need unique macro names |
| Dependency Graph Optimization | Forward declarations | Faster compile times, breaks cycles | Cannot use incomplete types by value |
| Header Categories | `<...>` vs. `"..."` | Correct search path, organization | Wrong syntax may cause file-not-found errors |
| Implementation Layout | Inline/template definitions in headers | Enables instantiation and inlining | Code bloat, longer compile times |

---

## References (Consolidated)

- C++ Core Guidelines: SF.8 (Include Guards) - https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rs-guards
- C++ Core Guidelines: SF.7 (No `using namespace` in headers) - https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rs-using-directive
- `#pragma once` — Microsoft Learn - https://learn.microsoft.com/en-us/cpp/preprocessor/once
- C++ Standard Library Header Files — Microsoft Learn - https://learn.microsoft.com/en-us/cpp/standard-library/cpp-standard-library-header-files
- Inline Functions (C++) — Microsoft Learn - https://learn.microsoft.com/en-us/cpp/cpp/inline-functions-cpp
- Source Code Organization (C++ Templates) — Microsoft Learn - https://learn.microsoft.com/en-us/cpp/cpp/source-code-organization-cpp-templates
- LLVM Coding Standards - https://llvm.org/docs/CodingStandards.html
- Google C++ Style Guide - https://google.github.io/styleguide/cppguide.html
- CERN ATLAS Coding Guidelines - https://atlas-software.docs.cern.ch/coding-guidelines/rules-coding/
- One Definition Rule — cppreference.com - https://en.cppreference.com/w/cpp/language/definition
- Forward Declaration — cppreference.com - https://en.cppreference.com/w/cpp/language/elaborated_type_specifier
- `#include` Directive — cppreference.com - https://en.cppreference.com/w/cpp/preprocessor/include
- Overview of Modules in C++ — Microsoft Learn - https://learn.microsoft.com/en-us/cpp/cpp/modules-cpp