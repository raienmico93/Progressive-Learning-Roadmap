# C++ Modules (C++20 and Beyond) — Comprehensive Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
C++ Modules are a language feature introduced in C++20 that replaces the traditional textual inclusion model (`#include`) with a structured, component-based architecture for sharing declarations and definitions across translation units. A module is a named collection of translation units whose exported declarations are made available to importers through a binary interface, eliminating the preprocessor's textual substitution and its associated fragility.

**Technical Definition**
A module consists of one or more **module units** — translation units that contain a module declaration. The module declaration has the form `[export] module module_name[:partition_name];`. A module unit with the `export` keyword is a **module interface unit**; without it, it is a **module implementation unit**. For every named module, there must be exactly one primary module interface unit (with no partition), whose exported content is available when importing the module. Modules are orthogonal to namespaces: a module can contain multiple namespaces, and a namespace can span multiple modules.

**Beginner-Friendly Explanation**
Think of traditional `#include` as photocopying pages from a book and pasting them into your own document. Every time you need something, you make a copy — and if the book changes, your copies are outdated. Modules are like checking out a book from a library: you get a reference to the original, and everyone reads the same, up-to-date version. The compiler reads a binary description of the module's exports once, and every translation unit that imports it reuses that description. No more copying, no more macro leaks, no more include-order surprises.

---

### Key Characteristics

- **Component architecture**: Modules replace textual inclusion with named, importable components.
- **Binary interface**: Module interface units compile to a Compiled Module Interface (CMI) that the compiler reads when importing.
- **Macro isolation**: Macros, preprocessor directives, and non-exported names are invisible to importers.
- **Module linkage**: A new linkage category — between external and internal — for non-exported declarations with external linkage in a module's purview.
- **Strong ownership**: The linker attaches exported names to the module that exports them, preventing cross-module name clashes.
- **Header units**: A transitional mechanism allowing `import <header>;` for "well-behaved" legacy headers.
- **Order independence**: Modules can be imported in any order without changing semantics.

---

### Prerequisites

- C++17 or later familiarity (for building the mental model of translation units and linkage).
- A compiler with C++20 modules support: GCC 14+, Clang 16+, or MSVC 2022 17.4+.
- A build system with modules support: CMake 3.28+ (with `cmake-cxxmodules`), MSBuild, or manual compiler invocations.
- Understanding of translation units, the One Definition Rule, and linkage concepts.

---

### Related Programming Areas

- **Header files and preprocessor mechanics** (the model modules replace).
- **Build systems** (CMake, MSBuild, Bazel) with modules dependency scanning.
- **Linkage and symbol visibility** (external, module, internal linkage).
- **C++20 standard library modules** (`import std;`, `std.core`, `std.memory`, etc.).
- **Package management** (vcpkg, Conan) with module-aware distribution.

---

### Core Concepts / Features

1. **Modern Translation Units** — Transitioning from textual inclusion to component architecture via Module Interfaces (`.ixx`, `.cppm`).
2. **Structural Partitioning** — Splitting large logical systems into Module Implementation Units and Module Partitions (`export module A:B`).
3. **Symbol Exporting** — Fine-grained encapsulation using the `export` keyword vs. keeping module-internal logic hidden.
4. **Consumption Mechanics** — Utilizing `import` directives and transforming legacy headers into Header Units.
5. **Performance Profiling** — Evaluating build speed advantages, macro-isolation benefits, and structural differences vs. traditional text headers.

---

## 1. Modern Translation Units (Module Interfaces)

### Definitions

**Core Definition**
A module interface unit is a translation unit whose module declaration includes the `export` keyword. It declares what the module provides to importers. Module interface units are typically stored in files with `.ixx` (MSVC convention) or `.cppm` (Clang/libc++ convention) extensions.

**Technical Definition**
A primary module interface unit is a module unit whose module declaration is `export module module_name;` with no module partition specified. A named module shall contain exactly one primary module interface unit. The interface unit may contain both exported declarations (prefixed with `export` or inside an `export { }` block) and non-exported declarations. Only exported declarations are visible to importers; non-exported declarations with external linkage receive module linkage instead.

**Beginner-Friendly Explanation**
A module interface is like the table of contents of a book. It tells you what chapters (functions, classes, templates) are available, but it doesn't show you the full text of every chapter. Some chapters might be "internal notes" that the author uses while writing but doesn't include in the published book. Only what's in the table of contents gets shared with readers.

---

### Purposes (all begin with "To")

- **To** define a clear, binary interface for a logical component that other translation units can import.
- **To** replace fragile header files with a structured, order-independent interface.
- **To** enable the compiler to read a precompiled binary description instead of reprocessing text.
- **To** provide a single source of truth for a module's exported entities.
- **To** separate the public API from implementation details at the language level.

---

### Syntax Rules and Structure

#### Complete General Syntaxes with Breakdowns

**Syntax 1 — Primary module interface unit**
```cpp
export module mylib;  // Module declaration

// Exported declarations
export int add(int a, int b);
export class Widget { /* ... */ };

// Non-exported (module-internal) declarations
int helper();  // has module linkage
```
**Component breakdown:**
- `export module mylib;` — declares this as the primary module interface unit for module `mylib`.
- `export int add(...)` — exports the function declaration.
- `export class Widget { ... }` — exports the class.
- `int helper();` — NOT exported; receives module linkage.

**Syntax 2 — Export block (C++20)**
```cpp
export module mylib;

export {
    int add(int a, int b);
    int subtract(int a, int b);
    namespace math {
        double pi();
    }
}
```
**Component breakdown:**
- `export { ... }` — exports all declarations inside the block.
- Namespaces can be exported: `export namespace math { ... }` exports all names in the namespace.

**Syntax 3 — Module implementation unit**
```cpp
module mylib;  // No 'export' keyword

// Implementation of exported declarations
int add(int a, int b) { return a + b; }
int subtract(int a, int b) { return a - b; }
```
**Component breakdown:**
- `module mylib;` — declares this as an implementation unit for module `mylib`.
- Implementation units implicitly import the primary module interface unit.
- The `export` keyword may not appear in a module implementation file.

#### Syntax Rules

- The module declaration, if present, must be the first declaration in the translation unit (except for the global module fragment).
- Module names consist of identifiers separated by dots (e.g., `mylib`, `mylib.utils`); dots have no semantic meaning.
- A module can have at most one primary module interface unit.
- Import declarations in a module unit must be grouped after the module declaration and before any other declarations.
- The `export` keyword can be applied to declarations, definitions, and namespaces, but not to entities with internal linkage or no linkage.

#### Constraints and Limitations

- GCC requires the `-fmodules` flag in addition to `-std=c++20`; modular compilation is not enabled by `-std=c++20` alone.
- GCC's modules support is not complete: Private Module Fragments are recognized but error out, and partition definition visibility rules are not fully implemented.
- Module interface units must be compiled before any importer; the import graph is a Directed Acyclic Graph (DAG).
- Module names beginning with `std` followed by digits, or containing reserved identifiers, are reserved for the implementation.
- Macros cannot be exported from named modules; this is by design (macro isolation).

---

### Multiple Annotated Complete Step-by-Step Code Examples and Their Expected Outputs

#### Example 1 — Basic Module Interface and Implementation

```cpp
// File: math.ixx
// Step 1: Module declaration with export
export module math;

// Step 2: Export the public API
export int add(int a, int b);
export int multiply(int a, int b);

// Step 3: Non-exported helper (module linkage)
int internal_helper(int x) {
    return x * 2;
}
```

```cpp
// File: math.cpp
// Step 4: Module implementation unit (no export)
module math;

// Step 5: Implement the exported declarations
int add(int a, int b) {
    return a + b;
}

int multiply(int a, int b) {
    return internal_helper(a) * b;  // Can call module-internal helper
}
```

```cpp
// File: main.cpp
// Step 6: Import the module
import math;
#include <iostream>

int main() {
    std::cout << "add(2, 3) = " << add(2, 3) << std::endl;
    std::cout << "multiply(4, 5) = " << multiply(4, 5) << std::endl;
    // internal_helper is NOT visible:
    // internal_helper(5);  // ERROR: not exported
    return 0;
}
```

**Compile and run (MSVC example):**
```bash
cl /std:c++20 /c /interface math.ixx
cl /std:c++20 /c math.cpp
cl /std:c++20 /reference math.ifc math.obj main.cpp /Fe:app.exe
./app.exe
```

**Expected Output:**
```
add(2, 3) = 5
multiply(4, 5) = 20
```

**Why this output:** The interface unit `math.ixx` exports `add` and `multiply` but keeps `internal_helper` unexported. The implementation unit `math.cpp` defines the exported functions and can call `internal_helper` because both are in the same module (module linkage). `main.cpp` imports `math` and can call `add` and `multiply`, but `internal_helper` is invisible — attempting to call it produces a compile error. This demonstrates the fundamental encapsulation guarantee of modules.

---

#### Example 2 — Module with Template and Class Exports

```cpp
// File: container.cppm
export module container;

#include <vector>  // Global module fragment handles headers

export template <typename T>
class Stack {
public:
    void push(const T& value) { data_.push_back(value); }
    T pop() {
        T value = data_.back();
        data_.pop_back();
        return value;
    }
    bool empty() const { return data_.empty(); }
    size_t size() const { return data_.size(); }

private:
    std::vector<T> data_;
};

export template <typename T>
T sum(const std::vector<T>& values) {
    T result{};
    for (const auto& v : values) result += v;
    return result;
}
```

```cpp
// File: main.cpp
import container;
#include <iostream>

int main() {
    Stack<int> stack;
    stack.push(10);
    stack.push(20);
    stack.push(30);

    std::cout << "Stack size: " << stack.size() << std::endl;
    std::cout << "Popped: " << stack.pop() << std::endl;
    std::cout << "Popped: " << stack.pop() << std::endl;

    std::vector<int> nums = {1, 2, 3, 4, 5};
    std::cout << "Sum: " << sum(nums) << std::endl;

    return 0;
}
```

**Expected Output:**
```
Stack size: 3
Popped: 30
Popped: 20
Sum: 15
```

**Why this output:** The module exports the class template `Stack<T>` and the function template `sum<T>`. Templates are fully supported in modules and are instantiated at the point of use in `main.cpp`. The `#include <vector>` appears in the module's global module fragment, which means the `std::vector` macros (if any) are quarantined within the module and do not leak to importers. This demonstrates that modules work seamlessly with templates, the primary use case for header-only libraries.

---

### Real-World Cases with Explanation

**Case 1 — Microsoft's STL modularization:** MSVC provides standard library modules such as `std.core` (everything else in the C++ Standard Library), `std.memory` (contents of `<memory>`), and `std.threading` (contents of `<atomic>`, `<condition_variable>`, `<future>`, `<mutex>`, `<shared_mutex>`, `<thread>`). These allow programs to `import std.core;` instead of including dozens of headers.

**Case 2 — libc++ modularization:** libc++ uses `.cppm` files for its module interface units, matching its existing `.cpp` source file convention. This enables Clang users to import standard library modules.

**Case 3 — Large-scale library conversion:** A mathematical software package converted to C++20 modules showed 30–40% less CPU time compared to the header-based version. The conversion replaced `#include` directives with `import` declarations, eliminating redundant parsing of the same headers across translation units.

---

### References Links

- Modules (since C++20) — cppreference.com - https://en.cppreference.com/w/cpp/language/modules
- Standard C++ Modules — Clang Documentation - https://llvm.github.io/www-releases/19.1.0/tools/clang/docs/StandardCPlusPlusModules.html
- Named modules tutorial in C++ — Microsoft Learn - https://learn.microsoft.com/en-us/cpp/cpp/tutorial-named-modules-cpp
- C++ Modules — GCC Documentation - https://gcc.gnu.org/onlinedocs/gcc/C_002b_002b-Modules.html
- Working Draft, Standard for Programming Language C++ — Module section - https://eel.is/c++draft/module

---

## 2. Structural Partitioning (Module Partitions)

### Definitions

**Core Definition**
A module partition is a module unit whose module declaration contains a module partition name (e.g., `export module A:B;`). Partitions allow a single logical module to be split across multiple files while maintaining the module's encapsulation boundary.

**Technical Definition**
A module partition is a module unit whose module declaration contains a module partition. There are two kinds: **module partition interface units** (`export module A:B;`) and **module partition implementation units** (`module A:B;`, also called internal partitions). All module partitions of a module that are module interface units shall be directly or indirectly exported by the primary module interface unit. Partitions can import sibling partitions using `import :B;` syntax. Module partitions are an implementation detail of their containing module and are not observable to code outside the module.

**Beginner-Friendly Explanation**
Imagine a large book called "Graphics." Instead of putting everything in one massive file, you split it into chapters: "Types," "Renderer," "Shaders," and "Window." Each chapter is a partition. Readers outside only see the finished book (the primary interface), but inside, the chapters can reference each other. You can reorganize the chapters without changing how readers use the book.

---

### Purposes (all begin with "To")

- **To** split large modules into manageable, logically cohesive units.
- **To** allow parallel compilation of different parts of the same module.
- **To** share module-internal declarations across partitions without exposing them to importers.
- **To** reduce recompilation scope: changing one partition does not require recompiling the entire module.
- **To** provide a structured alternative to the "everything in one file" approach for large libraries.

---

### Syntax Rules and Structure

#### Complete General Syntaxes with Breakdowns

**Syntax 1 — Module partition interface unit**
```cpp
export module graphics:types;

export struct Color { float r, g, b, a; };
export struct Rect { int x, y, w, h; };
```
**Component breakdown:**
- `export module graphics:types;` — declares a module partition interface unit for module `graphics`, partition `:types`.
- Exported declarations in this partition become visible when the primary interface exports this partition.

**Syntax 2 — Module partition implementation unit**
```cpp
module graphics:renderer;

// Implementation details for the renderer partition
void internal_render_setup() { /* ... */ }
```
**Component breakdown:**
- `module graphics:renderer;` — declares an implementation partition (no `export`).
- These units are imported only by other units within the same module.

**Syntax 3 — Primary module interface importing partitions**
```cpp
export module graphics;

export import :types;     // Export the types partition
export import :renderer;  // Export the renderer partition
```
**Component breakdown:**
- `export import :types;` — imports and re-exports the `:types` partition.
- Importers of `graphics` will see everything exported by `:types` and `:renderer`.

**Syntax 4 — Partition importing a sibling partition**
```cpp
export module graphics:renderer;
import :types;  // Import sibling partition

export class Renderer {
    // Can use Color and Rect from :types
};
```
**Component breakdown:**
- `import :types;` — imports the sibling partition `:types` within the same module.

#### Syntax Rules

- A module partition's name is only accessible to modules that have access to its parent module.
- An internal partition unit (without `export`) is not necessarily reachable by all other partitions; it must be explicitly imported.
- All partition interface units must be directly or indirectly exported by the primary module interface unit.
- Partition names are local to the module; two different modules can have partitions with the same name.
- A module partition implementation unit does not implicitly import the primary module interface unit (unlike regular implementation units).

#### Constraints and Limitations

- GCC's partition definition visibility rules are not fully implemented; definitions in implementation partitions may be available to extra-module use, violating encapsulation.
- The build system must understand partition dependencies; partition units must be compiled in the correct order.
- Partition names cannot be used in `import` declarations from outside the module.
- A partition implementation unit cannot export entities; only interface partitions can export.

---

### Multiple Annotated Complete Step-by-Step Code Examples and Their Expected Outputs

#### Example 1 — Graphics Module with Three Partitions

```cpp
// File: graphics_types.cppm
export module graphics:types;

export struct Color {
    float r, g, b, a;
};

export struct Rect {
    int x, y, w, h;
};
```

```cpp
// File: graphics_shapes.cppm
export module graphics:shapes;
import :types;  // Import sibling partition

export class Circle {
public:
    Circle(Color c, float radius) : color_(c), radius_(radius) {}
    float area() const { return 3.14159f * radius_ * radius_; }
    Color color() const { return color_; }

private:
    Color color_;
    float radius_;
};
```

```cpp
// File: graphics.cppm
export module graphics;

export import :types;
export import :shapes;
```

```cpp
// File: main.cpp
import graphics;
#include <iostream>

int main() {
    Color red{1.0f, 0.0f, 0.0f, 1.0f};
    Circle c(red, 5.0f);

    std::cout << "Circle area: " << c.area() << std::endl;
    std::cout << "Color R: " << c.color().r << std::endl;

    return 0;
}
```

**Expected Output:**
```
Circle area: 78.5398
Color R: 1
```

**Why this output:** The `graphics` module is split into two partitions: `:types` (defining `Color` and `Rect`) and `:shapes` (defining `Circle`, which uses `Color`). The `:shapes` partition imports `:types` as a sibling. The primary interface `graphics.cppm` export-imports both partitions, making them visible to `main.cpp`. The partition structure is invisible to `main.cpp` — it only sees `graphics`, not `graphics:types` or `graphics:shapes`. This demonstrates encapsulation: the internal partition structure can be reorganized without affecting importers.

---

#### Example 2 — Internal Partition (Implementation Partition)

```cpp
// File: engine_impl.cppm
module engine:impl;  // Internal partition (no export)

// Shared internal helpers — visible only within the engine module
int internal_counter = 0;

int next_id() {
    return ++internal_counter;
}
```

```cpp
// File: engine.cppm
export module engine;
import :impl;  // Import the internal partition

export class Engine {
public:
    Engine() : id_(next_id()) {}
    int id() const { return id_; }

private:
    int id_;
};
```

```cpp
// File: main.cpp
import engine;
#include <iostream>

int main() {
    Engine e1, e2, e3;
    std::cout << "Engine 1 ID: " << e1.id() << std::endl;
    std::cout << "Engine 2 ID: " << e2.id() << std::endl;
    std::cout << "Engine 3 ID: " << e3.id() << std::endl;
    // next_id() is NOT visible: // next_id();  // ERROR
    return 0;
}
```

**Expected Output:**
```
Engine 1 ID: 1
Engine 2 ID: 2
Engine 3 ID: 3
```

**Why this output:** The internal partition `engine:impl` defines `next_id()` and `internal_counter`, which have module linkage (visible to all units in the `engine` module but not to importers). The primary interface imports `:impl` and uses `next_id()` in the `Engine` constructor. `main.cpp` imports `engine` and can create `Engine` objects, but cannot call `next_id()` directly. This is the Goldilocks linkage: shared within the module, hidden from everyone else.

---

### Real-World Cases with Explanation

**Case 1 — Large game engine module:** A rendering engine module (`renderer`) can be split into partitions: `renderer:device` (GPU device management), `renderer:pipeline` (pipeline state objects), `renderer:shaders` (shader compilation), and `renderer:resources` (texture/buffer management). Each partition can be developed and compiled independently, with internal dependencies managed via `import :partition`.

**Case 2 — Mathematical library:** A linear algebra library (`linalg`) might have partitions for `:vector`, `:matrix`, `:operations`, and `:decomposition`. The `:operations` partition imports `:vector` and `:matrix`, while the primary interface export-imports all partitions.

**Case 3 — Standard library implementation:** libc++ uses partitions internally for its standard library modules, separating concerns like `std:vector`, `std:string`, and `std:algorithm` while presenting a unified `std` module to users.

---

### References Links

- Modules (since C++20): Module partitions — cppreference.com - https://en.cppreference.com/w/cpp/language/modules
- Standard C++ Modules: Module partitions — Clang Documentation - https://llvm.github.io/www-releases/19.1.0/tools/clang/docs/StandardCPlusPlusModules.html
- Named modules tutorial: Module partitions — Microsoft Learn - https://learn.microsoft.com/en-us/cpp/cpp/tutorial-named-modules-cpp
- Module linkage vs external linkage — Dr-Sergey/learn_cpp (GitHub) - https://github.com/Dr-Sergey/learn_cpp

---

## 3. Symbol Exporting (`export` Keyword)

### Definitions

**Core Definition**
The `export` keyword controls which declarations in a module interface unit are visible to importers. Exported declarations form the module's public API; non-exported declarations remain module-internal (module linkage).

**Technical Definition**
An export declaration has the form `export declaration` or `export { declaration-seq }`. Only entities with external linkage can be exported; entities with internal linkage (static, anonymous namespace) or no linkage cannot be exported. When `export` is applied to a namespace name, all names in the namespace are exported. The `export` keyword may not appear in a module implementation unit. An entity may be declared as exported, and later defined without the `export` keyword (e.g., in an implementation unit).

**Beginner-Friendly Explanation**
The `export` keyword is like a "public" label on a function or class. Everything without it is "private" to the module. This is much more precise than the header model, where anything in a header is visible to anyone who includes it. With modules, you can have helper functions, internal data structures, and implementation details that are genuinely hidden from consumers — not just by convention (like `detail` namespaces) but enforced by the compiler.

---

### Purposes (all begin with "To")

- **To** define a precise, compiler-enforced public API for a module.
- **To** hide implementation details that are not part of the module's contract.
- **To** reduce the symbol table size and prevent name collisions with other modules.
- **To** enable module linkage for non-exported helpers shared across module units.
- **To** support selective export of namespaces, classes, functions, and templates.

---

### Syntax Rules and Structure

#### Complete General Syntaxes with Breakdowns

**Syntax 1 — Export a single declaration**
```cpp
export int add(int a, int b);
export class Widget { /* ... */ };
export template <typename T> T max(T a, T b);
```

**Syntax 2 — Export a block of declarations**
```cpp
export {
    int add(int a, int b);
    int subtract(int a, int b);
    class Widget { /* ... */ };
}
```

**Syntax 3 — Export a namespace**
```cpp
export namespace math {
    double pi();
    double e();
}
```
All names in `math` are exported.

**Syntax 4 — Export an import (re-export)**
```cpp
export import other_module;  // Re-exports other_module's exports
```

#### Syntax Rules

- Only entities with external linkage can be exported.
- `export` may be applied to declarations, definitions, and namespaces.
- The `export` keyword cannot appear in a module implementation unit.
- An entity may be declared exported in one unit and defined without `export` in another.
- A module's exported entities are visible to any translation unit that imports the module.
- Non-exported declarations in a module interface unit receive module linkage (visible to all units of the same module).

#### Constraints and Limitations

- Macros cannot be exported from named modules.
- Entities with internal linkage (static, anonymous namespace) cannot be exported.
- If an exported function uses a non-exported type in its signature, the type must be made visible through some other mechanism (e.g., a header unit or a separate module).
- Exporting a namespace exports all names declared in that namespace in the module unit, including names that may not be intended for export.
- Overloading across exports requires all overloads to be exported or made visible.

---

### Multiple Annotated Complete Step-by-Step Code Examples and Their Expected Outputs

#### Example 1 — Selective Export vs. Module-Linkage Helpers

```cpp
// File: logger.cppm
export module logger;

#include <string>
#include <iostream>

// Step 1: Non-exported helper — module linkage
namespace detail {
    std::string timestamp() {
        return "[LOG] ";
    }
}

// Step 2: Exported public API
export void log_message(const std::string& msg) {
    std::cout << detail::timestamp() << msg << std::endl;
}

export void log_error(const std::string& msg) {
    std::cout << detail::timestamp() << "ERROR: " << msg << std::endl;
}
```

```cpp
// File: main.cpp
import logger;

int main() {
    log_message("Application started");
    log_error("Something went wrong");
    // detail::timestamp() is NOT visible:
    // detail::timestamp();  // ERROR: not exported
    return 0;
}
```

**Expected Output:**
```
[LOG] Application started
[LOG] ERROR: Something went wrong
```

**Why this output:** The `detail` namespace and its `timestamp()` function are not exported, so they receive module linkage. They are visible to `log_message` and `log_error` (which are in the same module) but invisible to `main.cpp`. This is compiler-enforced encapsulation: no `#define` tricks, no `detail` namespace convention — the symbol is genuinely inaccessible.

---

#### Example 2 — Exporting a Namespace

```cpp
// File: geometry.cppm
export module geometry;

export namespace shapes {
    struct Point {
        double x, y;
    };

    struct Circle {
        Point center;
        double radius;
    };

    double area(const Circle& c) {
        return 3.14159 * c.radius * c.radius;
    }
}

// Non-exported namespace — module linkage
namespace internal {
    void validate(const shapes::Circle& c) {
        // internal validation logic
    }
}
```

```cpp
// File: main.cpp
import geometry;
#include <iostream>

int main() {
    shapes::Circle c{{0, 0}, 5.0};
    std::cout << "Area: " << shapes::area(c) << std::endl;
    // internal::validate(c);  // ERROR: not exported
    return 0;
}
```

**Expected Output:**
```
Area: 78.5397
```

**Why this output:** The entire `shapes` namespace is exported, so `Point`, `Circle`, and `area` are all visible to `main.cpp`. The `internal` namespace is not exported and receives module linkage, making it invisible to importers. This demonstrates that a single module can have both a public API (exported namespace) and private implementation (non-exported namespace).

---

### Real-World Cases with Explanation

**Case 1 — Library API design:** A library module exports only the functions and classes intended for users (e.g., `export class Database`), while keeping connection pooling, query parsing, and error handling helpers non-exported. This prevents users from accidentally depending on internal implementation details.

**Case 2 — Template libraries:** A template-heavy library exports the templates themselves, but keeps the internal traits classes and helper functions non-exported. This ensures that only the intended customization points are visible.

**Case 3 — Standard library modules:** MSVC's `std.core` module exports everything from the standard library except what's covered by more specific modules like `std.memory` and `std.threading`. This gives users a choice between a single monolithic import (`import std.core;`) and fine-grained imports (`import std.memory; import std.threading;`).

---

### References Links

- module, import, export — Microsoft Learn - https://learn.microsoft.com/en-us/cpp/cpp/import-export-module
- Modules (since C++20): Exporting declarations — cppreference.com - https://en.cppreference.com/w/cpp/language/modules
- Understand module linkage vs external linkage — Dr-Sergey/learn_cpp (GitHub) - https://github.com/Dr-Sergey/learn_cpp

---

## 4. Consumption Mechanics (Import Directives and Header Units)

### Definitions

**Core Definition**
Consumption mechanics refers to how translation units use modules: through `import` declarations for named modules and through **header units** (`import <header>;`) for legacy headers that have been compiled into a module-like binary format.

**Technical Definition**
An import declaration has the form `import module-name;` or `export import module-name;`. Import declarations make the exported declarations of the named module available in the current translation unit. **Header units** are a transitional mechanism: a "well-behaved" header can be compiled into a header unit, which is then imported with `import <header-name>;` syntax (replacing `#include <header-name>`). Header units are not affected by macros from the importer, and they do not expose macros to the importer (unlike `#include`). The **global module fragment** (`module;`) allows a module to consume traditional headers while keeping their macros quarantined.

**Beginner-Friendly Explanation**
Think of `import` as a modern replacement for `#include`. For your own code, you write `import mylib;`. For old headers that you can't convert to modules yet, you write `import <vector>;` — this compiles the header into a header unit and imports it. The header unit is immune to whatever macros you've defined in your own file, and it doesn't leak its own macros to you. It's like the header is in a clean room.

---

### Purposes (all begin with "To")

- **To** consume module exports with a clean, order-independent syntax.
- **To** provide a migration path from `#include` to modules via header units.
- **To** isolate legacy headers from macro pollution in the importing translation unit.
- **To** reduce the cost of consuming standard library and third-party headers.
- **To** allow modules to use traditional headers (via the global module fragment) without macro leakage.

---

### Syntax Rules and Structure

#### Complete General Syntaxes with Breakdowns

**Syntax 1 — Import a named module**
```cpp
import mylib;  // Import all exports from mylib
```

**Syntax 2 — Import a header unit**
```cpp
import <vector>;      // Import standard library header as header unit
import "my_header.h"; // Import user header as header unit
```

**Syntax 3 — Export an import (re-export)**
```cpp
export import other_module;  // Import and re-export other_module
```

**Syntax 4 — Global module fragment (for consuming headers in a module)**
```cpp
module;                   // Start global module fragment
#include <iostream>       // Headers here are in the global module
#include <string>

export module mylib;      // Start the actual module

export void print(const std::string& msg) {
    std::cout << msg << std::endl;
}
```
**Component breakdown:**
- `module;` — begins the global module fragment.
- `#include` directives — headers included here have their macros quarantined.
- `export module mylib;` — the module declaration proper.

#### Syntax Rules

- Import declarations in a module unit must be grouped after the module declaration and before any other declarations.
- Importing a module is not transitive by default, except for exported imports.
- Header units are compiled with `-fmodule-header` (GCC), `-fmodules-ts` (Clang), or `/headerUnit` (MSVC).
- Header units may be affected by macros defined before the import, but they do not leak their own macros to the importer.
- A translation unit can either `#include` a header or `import` it as a header unit, not both (in the same TU).
- The global module fragment must appear before the module declaration and may only contain preprocessing directives and declarations with external linkage (no definitions).

#### Constraints and Limitations

- Header units are a transitional mechanism and may be slower than named modules.
- Not all headers can be header units; "well-behaved" headers (no conditional compilation based on macros that change between TUs) work best.
- GCC requires `-fmodules` and `-fmodule-header` for header units; these are not enabled by `-std=c++20` alone.
- The global module fragment cannot contain definitions of entities with external linkage (only declarations).
- Header units are not a complete replacement for named modules; they don't provide the same encapsulation guarantees.

---

### Multiple Annotated Complete Step-by-Step Code Examples and Their Expected Outputs

#### Example 1 — Importing a Named Module and a Header Unit

```cpp
// File: math.cppm
export module math;

export int add(int a, int b) { return a + b; }
export int multiply(int a, int b) { return a * b; }
```

```cpp
// File: main.cpp
// Step 1: Import a header unit (standard library)
import <iostream>;
import <vector>;

// Step 2: Import a named module
import math;

int main() {
    std::vector<int> values = {1, 2, 3, 4, 5};
    int total = 0;

    for (int v : values) {
        total = add(total, v);  // Use math module
    }

    std::cout << "Total: " << total << std::endl;
    std::cout << "Product: " << multiply(3, 7) << std::endl;

    return 0;
}
```

**Compile and run (GCC example):**
```bash
# Compile header units
g++ -std=c++20 -fmodules -fmodule-header -x c++-system-header iostream
g++ -std=c++20 -fmodules -fmodule-header -x c++-system-header vector

# Compile the math module
g++ -std=c++20 -fmodules -c math.cppm -o math.o

# Compile main
g++ -std=c++20 -fmodules -c main.cpp -o main.o

# Link
g++ math.o main.o -o app
./app
```

**Expected Output:**
```
Total: 15
Product: 21
```

**Why this output:** `main.cpp` imports `<iostream>` and `<vector>` as header units (replacing `#include`), and imports the `math` module for `add` and `multiply`. The header units are precompiled binary representations of the headers, immune to any macros defined in `main.cpp`. The `math` module provides its exports through the binary CMI. Both mechanisms work together seamlessly.

---

#### Example 2 — Global Module Fragment for Legacy Headers

```cpp
// File: legacy_wrapper.cppm
// Step 1: Global module fragment for C headers and macros
module;
#include <cstdio>   // stdio.h macros like NULL, EOF are quarantined
#include <cstring>

// Step 2: Module declaration
export module legacy_wrapper;

// Step 3: Exported API wrapping legacy C functions
export const char* get_error_string() {
    return std::strerror(0);
}

export int safe_print(const char* msg) {
    if (msg == nullptr) return EOF;  // EOF macro used internally only
    return std::printf("%s\n", msg);
}
```

```cpp
// File: main.cpp
import legacy_wrapper;

int main() {
    const char* err = get_error_string();
    safe_print("Hello from legacy wrapper");
    safe_print(err);
    // EOF and NULL are NOT visible here:
    // int x = EOF;  // ERROR: not defined
    return 0;
}
```

**Expected Output:**
```
Hello from legacy wrapper
Success
```

**Why this output:** The `#include <cstdio>` and `#include <cstring>` appear in the global module fragment. Macros like `EOF` and `NULL` are confined to the global module fragment and do not leak to importers. The exported functions `get_error_string` and `safe_print` provide a clean C++ API. `main.cpp` can use these functions but cannot see `EOF` or `NULL`, demonstrating macro isolation.

---

### Real-World Cases with Explanation

**Case 1 — Migrating a codebase gradually:** A project can convert one library at a time to modules. Headers that haven't been converted are consumed as header units, while converted libraries use named modules. This allows incremental adoption without a "big bang" rewrite.

**Case 2 — Standard library consumption:** MSVC supports `import std.core;` and `import std.memory;` as named modules, but also allows `import <vector>;` as a header unit for transitional use. This gives developers a choice between the fastest option (named modules) and the most compatible option (header units).

**Case 3 — Third-party libraries:** A library like nlohmann/json can be consumed as `import "json.hpp";` (header unit) without modification, or the library vendor can provide a `.cppm` module interface for named module consumption.

**Case 4 — C API wrapping:** C libraries with macros (e.g., `errno`, `ENOENT`, `EACCES`) can be wrapped in a module that includes the C headers in the global module fragment and exports type-safe C++ abstractions. Importers never see the raw macros.

---

### References Links

- Compare header units, modules, and precompiled headers — Microsoft Learn - https://learn.microsoft.com/en-us/cpp/build/compare-inclusion-methods
- Modules (since C++20): Importing modules and headers — cppreference.com - https://en.cppreference.com/w/cpp/language/modules
- Understand how to mix modules with header-based code — Dr-Sergey/learn_cpp (GitHub) - https://github.com/Dr-Sergey/learn_cpp
- C++ Modules: Header units — GCC Documentation - https://gcc.gnu.org/onlinedocs/gcc/C_002b_002b-Modules.html

---

## 5. Performance Profiling (Modules vs. Headers)

### Definitions

**Core Definition**
Performance profiling in the context of modules refers to the empirical comparison of build times, macro isolation, and structural differences between module-based and header-based compilation.

**Technical Definition**
Modules are compiled once into a binary Compiled Module Interface (CMI). Importing a module reads the CMI instead of reprocessing text. This eliminates redundant parsing of the same declarations across translation units. Header units provide a similar benefit for legacy headers. The key performance metrics are: (1) initial build time, (2) incremental build time after a change, (3) macro isolation (no macro leakage), and (4) ODR protection through module ownership.

**Beginner-Friendly Explanation**
With headers, every `.cpp` file that includes `<vector>` re-parses the entire `<vector>` header from scratch. With modules, `<vector>` is compiled once into a binary file, and every file that imports it reads that binary. It's like the difference between reading a recipe from a cookbook every time you cook (headers) versus memorizing the recipe once and never looking it up again (modules).

---

### Purposes (all begin with "To")

- **To** quantify the build time savings of modules over headers.
- **To** demonstrate the macro isolation benefit of named modules.
- **To** compare module-based and header-based approaches for specific project sizes.
- **To** identify when modules provide the greatest benefit (many TUs consuming the same library).
- **To** evaluate the trade-offs of header units vs. named modules vs. precompiled headers.

---

### Syntax Rules and Structure

#### Benchmark Methodology (General Approach)

1. Create a header-based version of a project.
2. Convert the project to modules (named modules or header units).
3. Measure: (a) clean build time, (b) incremental build time after modifying one file, (c) total CPU time across all cores.
4. Compare results across compilers (GCC, Clang, MSVC).

#### Typical Compiler Flags

| Compiler | Modules Flag | Header Units Flag |
|---|---|---|
| GCC | `-fmodules -std=c++20` | `-fmodule-header` |
| Clang | `-fmodules-ts -std=c++20` | `-fmodule-header` |
| MSVC | `/std:c++20` (automatic) | `/headerUnit` |

#### Constraints and Limitations

- Build time improvements depend heavily on project structure: projects with many TUs consuming the same large libraries benefit most.
- Modules with large interfaces can be slower to load than smaller, focused modules.
- Header units are faster than `#include` but slower than named modules.
- Precompiled headers (PCH) are faster than `#include` but slower than import and harder to maintain.
- Compiler support for modules varies; GCC's support is less mature than MSVC's or Clang's.

---

### Multiple Annotated Complete Step-by-Step Code Examples and Their Expected Outputs

#### Example 1 — Benchmarking Standard Library Import vs. Include

```cpp
// File: header_version.cpp
#include <vector>
#include <string>
#include <algorithm>
#include <iostream>
#include <map>
#include <set>

int main() {
    std::vector<int> v = {3, 1, 4, 1, 5};
    std::sort(v.begin(), v.end());
    return v[0];
}
```

```cpp
// File: module_version.cpp
import <vector>;
import <string>;
import <algorithm>;
import <iostream>;
import <map>;
import <set>;

int main() {
    std::vector<int> v = {3, 1, 4, 1, 5};
    std::sort(v.begin(), v.end());
    return v[0];
}
```

**Benchmark results (from ISOCPP std-proposals list, 2026):**

| Compiler | Header Unit Build | Module Import Build | `#include` Build |
|---|---|---|---|
| GCC 16.1.0 | 3345 ms | 249 ms | 1579 ms |
| Clang 22.1.6 | 2014 ms | 52 ms | 1545 ms |
| MSVC 19.38.33145 | 2058 ms | 81 ms | 1295 ms |

**Why this matters:** Importing all standard library headers as header units (about 0.10 seconds) is roughly a 10x decrease in time over including them. The "Module Build" column represents the one-time cost of building the header units; the "Module Import" column represents the cost per translation unit after the header units are built. The 52–249 ms import time is 6–30x faster than the 877–1579 ms include time.

---

#### Example 2 — Boost.Mp11 Test Suite Benchmark

A real-world benchmark of the Boost.Mp11 test suite (around 200 translation units):

| Approach | Build Time |
|---|---|
| Headers | 2 min 10 s |
| Modules | 39 s |

**Why this is significant:** The modules build includes the time required to build the `std` and Boost modules. Even with this overhead, the total time is less than one-third of the header-based build. The savings come from two sources: (1) the standard library is compiled once into modules instead of being reprocessed in each of the 200 TUs, and (2) the Boost.Mp11 templates are compiled once instead of being re-instantiated in each TU.

---

#### Example 3 — Macro Isolation Demonstration

```cpp
// File: logging.cppm
export module logging;

#define INTERNAL_DEBUG 1

export void log(const char* msg) {
    // INTERNAL_DEBUG is visible here
}

// INTERNAL_DEBUG is NOT exported
```

```cpp
// File: main.cpp
import logging;

#ifdef INTERNAL_DEBUG
#error "Macro leaked from module!"
#else
// This branch is taken — no macro leakage
#endif

int main() {
    log("Hello");
    return 0;
}
```

**Expected behavior:** Compilation succeeds. The `#ifdef INTERNAL_DEBUG` check fails (the macro is not defined in `main.cpp`), confirming that the macro defined in `logging.cppm` did not leak to the importer. This is the firewall behavior of modules: macros defined in a module are quarantined and invisible to importers.

**Why this matters:** In the header model, any `#define` in a header leaks to every file that includes it (directly or indirectly). This causes "macro pollution," where a macro defined for one purpose (e.g., `min` or `max` in Windows headers) breaks unrelated code. Modules eliminate this class of bugs entirely.

---

### Real-World Cases with Explanation

**Case 1 — Mathematical software conversion:** A large mathematical software package (published in ACM TOMS) converted from headers to modules showed 30–40% less CPU time on average. At lower core counts (`-j16`), the wall time was 2m33s for modules vs. 2m52s–3m12s for headers, a 10% improvement.

**Case 2 — Clang module cache:** Including `<ccomplex>` without a module cache took 22.4 seconds; with a module cache, it took 1.6 seconds — a 14x improvement. This demonstrates the dramatic speedup when the same header is consumed repeatedly across a large project.

**Case 3 — Precompiled headers vs. modules vs. header units:** Microsoft's comparison ranks `#include` as the slowest and least robust, PCH as faster but hard to maintain, header units as faster than both, and named modules as the fastest and most robust. A 250 MB PCH file might be represented by an 80 MB header unit file.

**Case 4 — Large-scale project migration:** A project with hundreds of translation units importing the same standard library and third-party libraries benefits most from modules. The one-time cost of building the module interface is amortized across all TUs. For small projects with few TUs, the benefit is smaller, and the build system complexity may not be worth it.

---

### References Links

- Benchmarking including the C++ standard library — ISOCPP std-proposals list - https://lists.isocpp.org
- Experience converting a large mathematical software package to C++20 modules — ACM TOMS - https://arxiv.org
- Compare header units, modules, and precompiled headers — Microsoft Learn - https://learn.microsoft.com/en-us/cpp/build/compare-inclusion-methods
- What do we want from a modularized Standard Library? — WG21 P2172R0 - https://rap.no/JTC1/SC22/WG21/docs/papers/2020/p2172r0.pdf

---

## Summary Table of Core Concepts

| Concept | Key Mechanism | Primary Benefit | Key Constraint |
|---|---|---|---|
| Modern Translation Units | `export module M;` + `.ixx`/`.cppm` | Binary interface, order independence | GCC requires `-fmodules`; incomplete support |
| Structural Partitioning | `export module A:B;` | Split large modules, parallel compilation | Build order dependencies; partition names module-local |
| Symbol Exporting | `export` keyword | Compiler-enforced encapsulation | Cannot export macros or internal-linkage entities |
| Consumption Mechanics | `import M;`, `import <header>;` | Clean consumption, header migration path | Header units slower than named modules |
| Performance Profiling | CMI reuse vs. text reprocessing | 30–40% CPU time reduction, 6–30x faster imports | Benefits scale with project size |

---

## References (Consolidated)

- Modules (since C++20) — cppreference.com - https://en.cppreference.com/w/cpp/language/modules
- Standard C++ Modules — Clang Documentation - https://llvm.github.io/www-releases/19.1.0/tools/clang/docs/StandardCPlusPlusModules.html
- Named modules tutorial in C++ — Microsoft Learn - https://learn.microsoft.com/en-us/cpp/cpp/tutorial-named-modules-cpp
- C++ Modules — GCC Documentation - https://gcc.gnu.org/onlinedocs/gcc/C_002b_002b-Modules.html
- Working Draft, Standard for Programming Language C++ — Module section - https://eel.is/c++draft/module
- Compare header units, modules, and precompiled headers — Microsoft Learn - https://learn.microsoft.com/en-us/cpp/build/compare-inclusion-methods
- module, import, export — Microsoft Learn - https://learn.microsoft.com/en-us/cpp/cpp/import-export-module
- Understanding module linkage vs external linkage — Dr-Sergey/learn_cpp (GitHub) - https://github.com/Dr-Sergey/learn_cpp
- Understanding global module fragment for macros — Dr-Sergey/learn_cpp (GitHub) - https://github.com/Dr-Sergey/learn_cpp
- Benchmarking including the C++ standard library — ISOCPP std-proposals list - https://lists.isocpp.org
- Experience converting a large mathematical software package to C++20 modules — ACM TOMS - https://arxiv.org
- What do we want from a modularized Standard Library? — WG21 P2172R0 - https://rap.no/JTC1/SC22/WG21/docs/papers/2020/p2172r0.pdf
- Merging Modules — WG21 P1103 - https://wg21.link/p1103