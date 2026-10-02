# C++ Compilation Model — Comprehensive Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
The C++ compilation model is the multi-stage process by which human-readable C++ source code is transformed into an executable program. It consists of four distinct stages: preprocessing, compilation, assembly, and linking.

**Technical Definition**
The compilation model defines how a C++ program is translated from source files into machine code. Each source file, together with all headers it includes directly or indirectly, forms a **translation unit** (TU). Each TU is compiled independently into an **object file** containing machine code, a symbol table, and relocation information. The **linker** then combines these object files, resolves external references, and produces a final executable or library. The model is governed by three specifications: the C++ language standard (ISO/IEC 14882), the platform ABI (e.g., System V AMD64 ABI), and the C++ ABI (e.g., Itanium C++ ABI).

**Beginner-Friendly Explanation**
Imagine you're writing a book in a language only you understand. To share it, you need to translate it step by step. First, you clean up your notes (preprocessing). Then you translate each chapter into a common language (compilation). Then you write it in a format the printer can read (assembly). Finally, you bind all the chapters together into a single book (linking). C++ works the same way: your `.cpp` files go through four stages before becoming a program you can run.

---

### Key Characteristics

- **Four-stage pipeline**: Preprocessing → Compilation → Assembly → Linking, always in that order.
- **Translation unit isolation**: Each TU is compiled independently; the compiler sees only one TU at a time.
- **Incremental builds**: Changing one source file requires recompiling only that file, not the entire program.
- **One Definition Rule (ODR)**: A symbol can be declared many times but defined only once across all TUs.
- **Linkage control**: `static` and anonymous namespaces give internal linkage; `extern` gives external linkage.
- **Diagnostic stages**: Different errors occur at different stages — compiler errors during compilation, assembler errors during assembly, and linker errors during linking.

---

### Prerequisites

- Basic C++ syntax: functions, variables, classes, and the preprocessor.
- Familiarity with header files and the `#include` directive.
- Understanding of translation units and separate compilation.
- Basic knowledge of the One Definition Rule (ODR).
- (For linking) Familiarity with command-line tools: `g++`, `clang++`, `ld`, `as`.

---

### Related Programming Areas

- **Header files** (declarations that source files implement).
- **Build systems** (Make, CMake, Bazel) that orchestrate compilation.
- **Linkage and symbol visibility** (`extern`, `static`, anonymous namespaces).
- **Libraries** (static `.a`/`.lib` vs. dynamic `.so`/`.dll`).
- **C++20 modules** (a modern alternative to the header/source model).
- **Name mangling** and the C++ ABI (how symbols are encoded for the linker).

---

### Core Concepts / Features

1. **The 4-Stage Pipeline** — Detailed execution tracing through Preprocessing, Compilation, Assembly, and Linking.
2. **Scope Isolation** — Defining the exact boundaries of a Translation Unit (TU).
3. **The Core Rule** — Enforcing the One Definition Rule (ODR) across variables, functions, and class structures.
4. **Symbol Visibility** — Differentiating between internal linkage (`static`, anonymous namespaces) and external linkage (`extern`).
5. **Diagnostic Tracking** — Recognizing the root causes of compiler errors vs. assembler failures vs. undefined reference linker errors.

---

## 1. The 4-Stage Pipeline

### Definitions

**Core Definition**
The 4-stage pipeline is the sequential process by which a C++ source file is transformed into an executable: **preprocessing**, **compilation**, **assembly**, and **linking**.

**Technical Definition**
- **Preprocessing**: The preprocessor handles directives (`#include`, `#define`, `#if`), removes comments, and expands macros. The output is a **translation unit** — the input that the compiler actually compiles.
- **Compilation**: The compiler translates the preprocessed C++ code into assembly language (or directly into machine code, depending on the implementation). It performs parsing, semantic analysis, optimization, and code generation.
- **Assembly**: The assembler translates the assembly code into machine code, producing a **relocatable object file** (`.o` on Unix, `.obj` on Windows).
- **Linking**: The linker combines object files and libraries, resolves external references, and produces an executable or library.

**Beginner-Friendly Explanation**
Think of a four-step factory assembly line. Step 1 (preprocessing) is like gathering all your ingredients and reading the recipe. Step 2 (compilation) is cooking the meal. Step 3 (assembly) is plating it. Step 4 (linking) is serving it with the side dishes from other kitchens. Each step transforms the "food" into something closer to what the customer (the computer) can consume.

---

### Purposes (all begin with "To")

- **To** transform human-readable C++ source code into machine-executable code.
- **To** enable separate compilation, where each source file is processed independently.
- **To** allow errors to be detected at the earliest possible stage (preprocessing, compilation, assembly, or linking).
- **To** support incremental builds by allowing unchanged files to be reused.
- **To** combine multiple translation units into a single cohesive program.

---

### Syntax Rules and Structure

#### Complete General Syntaxes with Breakdowns

**Stage 1 — Preprocessing**
```bash
g++ -E source.cpp -o source.ii
```
- `-E` — run only the preprocessor.
- `source.cpp` — input source file.
- `source.ii` — preprocessed output (C++ source that should not be preprocessed again).

**Stage 2 — Compilation (to assembly)**
```bash
g++ -S source.ii -o source.s
```
- `-S` — compile to assembly language.
- `source.s` — assembly output file.

**Stage 3 — Assembly**
```bash
g++ -c source.s -o source.o
# Or directly from source:
g++ -c source.cpp -o source.o
```
- `-c` — compile and assemble, but do not link.
- `source.o` — object file.

**Stage 4 — Linking**
```bash
g++ source.o -o program
```
- `source.o` — input object file.
- `-o program` — output executable.

#### Syntax Rules

- The stages must be executed in order: preprocessing → compilation → assembly → linking.
- The first three stages apply to an individual source file; linking combines all object files.
- The compiler driver (`g++`) invokes the appropriate tools automatically unless instructed otherwise.
- Intermediate files can be preserved with `--save-temps` or generated explicitly with `-E`, `-S`, and `-c`.

#### Constraints and Limitations

- Object files are platform-specific; a `.o` compiled on Linux cannot be linked on Windows.
- The compiler must see the entire translation unit to generate correct code; it cannot see symbols defined in other TUs.
- The linker resolves symbols by name (after name mangling); mismatched names cause undefined reference errors.
- Optimization flags (`-O2`, `-O3`) affect compilation and assembly but not preprocessing or linking.

---

### Multiple Annotated Complete Step-by-Step Code Examples and Their Expected Outputs

#### Example 1 — Tracing Through All Four Stages

```cpp
// File: hello.cpp
#include <iostream>  // Preprocessor will expand this

#define GREETING "Hello, World!"

int main() {
    std::cout << GREETING << std::endl;  // Macro expansion
    return 0;
}
```

**Step 1 — Preprocessing:**
```bash
g++ -E hello.cpp -o hello.ii
```
The file `hello.ii` contains the expanded `<iostream>` contents and `GREETING` replaced with `"Hello, World!"`.

**Step 2 — Compilation to assembly:**
```bash
g++ -S hello.ii -o hello.s
```
The file `hello.s` contains x86-64 assembly instructions (or ARM, depending on the target).

**Step 3 — Assembly:**
```bash
g++ -c hello.s -o hello.o
```
The file `hello.o` contains machine code and a symbol table with entries like `_ZSt4cout`, `main`.

**Step 4 — Linking:**
```bash
g++ hello.o -o hello
./hello
```

**Expected Output:**
```
Hello, World!
```

**Why this output:** The preprocessor expanded `#include <iostream>` and the `GREETING` macro. The compiler translated the preprocessed code to assembly. The assembler produced machine code in `hello.o`. The linker combined `hello.o` with the C++ standard library (`libstdc++`) to resolve `std::cout` and `std::endl`. Without linking against `libstdc++`, the linker would report `undefined reference to _ZSt4cout`.

---

#### Example 2 — Inspecting Intermediate Files

```bash
# Save all intermediate files
g++ --save-temps -c hello.cpp -o hello.o

# Files produced:
# hello.ii  — preprocessed output
# hello.s   — assembly output
# hello.o   — object file
```

```bash
# Inspect symbols in the object file
nm hello.o

# Expected output (example):
# 0000000000000000 T main
#                  U _ZSt4cout
#                  U _ZStlsISt11char_traitsIcEERSt13basic_ostreamIcT_ES5_PKc
#                  U _ZSt4endlIcSt11char_traitsIcEERSt13basic_ostreamIT_T0_ES6_
```

**Why this matters:** The `T` indicates a defined symbol (`main`), and `U` indicates an undefined symbol to be resolved by the linker. The mangled names (`_ZSt4cout`, etc.) are how the linker identifies C++ symbols. This illustrates why C++ uses name mangling: to distinguish overloaded functions and namespaced entities.

---

### Real-World Cases with Explanation

**Case 1 — Large-scale projects (Chromium, LLVM):** These projects have tens of thousands of source files. The 4-stage pipeline is applied to each file independently, and a build system orchestrates parallel compilation and incremental linking. The `--save-temps` flag is often used to debug compilation issues.

**Case 2 — Cross-compilation for embedded systems:** A cross-compiler (e.g., `arm-none-eabi-g++`) runs the pipeline for a target architecture. The preprocessing and compilation stages run on the host, while assembly and linking produce code for the target.

**Case 3 — Build caching (ccache, distcc):** Tools like `ccache` cache the output of preprocessing and compilation. If a source file hasn't changed, the cached object file is reused, skipping the first two stages. `distcc` distributes the compilation stage across multiple machines.

**Case 4 — Debugging linker errors:** Understanding the pipeline helps diagnose errors. A "syntax error" occurs during compilation. An "undefined reference" occurs during linking. An "assembler error" (rare in C++) occurs during assembly. Each stage has distinct error messages.

---

### References Links

- Phases of translation — cppreference.com - https://en.cppreference.com/w/cpp/language/translation_phases
- Overall Options (GCC) — https://gcc.gnu.org/onlinedocs/gcc/Overall-Options.html
- C++ Compilation — Caltech CS11 - http://courses.cms.caltech.edu/cs11/material/cpp/donnie/lectures/cs11-cpp-lec2.pdf
- Clang Compiler — https://clang.llvm.org/

---

## 2. Scope Isolation (Translation Units)

### Definitions

**Core Definition**
A translation unit (TU) is the fundamental unit of compilation in C++. It consists of a single source file together with all headers it includes directly or indirectly.

**Technical Definition**
A translation unit is defined by the C++ standard as "a source file together with all the headers and source files included via the preprocessing directive `#include`, minus those source lines skipped by any of the conditional inclusion preprocessing directives". Each TU is compiled independently. The compiler processes one TU at a time and produces one object file. A program consists of one or more TUs linked together.

**Beginner-Friendly Explanation**
A translation unit is everything the compiler sees when it compiles one `.cpp` file. If your `main.cpp` includes `<iostream>`, `"my_class.h"`, and `"utils.h"`, then the translation unit includes all of those files' contents, expanded and merged. The compiler doesn't know about `other.cpp` — it only sees `main.cpp` and its includes. That's why you need headers: to tell the compiler what exists in other TUs.

---

### Purposes (all begin with "To")

- **To** define the exact scope of compilation for a single source file.
- **To** enable separate compilation by isolating each source file's compilation.
- **To** limit the compiler's view to a manageable unit of code.
- **To** allow parallel compilation of multiple TUs.
- **To** provide a clear boundary for the ODR: each entity must be defined in exactly one TU.

---

### Syntax Rules and Structure

#### Complete General Syntax with Breakdown

```cpp
// Source file: main.cpp
#include <iostream>      // Header inclusion
#include "my_class.h"    // Project header inclusion

// Declarations and definitions
int main() {
    std::cout << "Hello" << std::endl;
    return 0;
}
```

**Translation unit boundaries:**
- The TU begins at the start of `main.cpp`.
- `<iostream>` is replaced with the contents of the standard library header.
- `"my_class.h"` is replaced with the contents of the project header.
- The TU ends at the end of `main.cpp`.
- Any header included by `my_class.h` (directly or indirectly) is also part of this TU.

#### Syntax Rules

- Each `.cpp` file produces exactly one translation unit (after preprocessing).
- Headers are not compiled independently; they become part of the TU of each `.cpp` file that includes them.
- A header can be part of multiple TUs (one for each `.cpp` that includes it).
- The compiler compiles each TU independently and produces one object file per TU.
- The linker combines all object files into the final program.

#### Constraints and Limitations

- The compiler cannot see symbols defined in other TUs; it relies on declarations.
- A header included in multiple TUs must be protected against multiple inclusion (include guards or `#pragma once`).
- ODR violations occur when a symbol is defined differently in different TUs.
- Templates are instantiated per TU, which can increase compile times and code size.
- The TU boundary is defined by the source file and its includes, not by namespaces or classes.

---

### Multiple Annotated Complete Step-by-Step Code Examples and Their Expected Outputs

#### Example 1 — Two Translation Units Sharing a Header

```cpp
// File: shared.h
#ifndef SHARED_H
#define SHARED_H

void greet();  // Declaration

#endif
```

```cpp
// File: main.cpp — Translation Unit 1
#include "shared.h"
#include <iostream>

void greet() {
    std::cout << "Hello from TU 1" << std::endl;
}

int main() {
    greet();
    return 0;
}
```

```cpp
// File: other.cpp — Translation Unit 2
#include "shared.h"

// This TU sees the declaration of greet() but not its definition
void call_greet() {
    greet();  // Resolved by the linker to the definition in main.cpp
}
```

**Compile and run:**
```bash
g++ -std=c++17 main.cpp other.cpp -o app
./app
```

**Expected Output:**
```
Hello from TU 1
```

**Why this output:** `main.cpp` and `other.cpp` are two separate translation units. Both include `shared.h`, which declares `greet()`. Only `main.cpp` defines `greet()`. When `other.cpp` calls `greet()`, the compiler generates an external reference. The linker resolves this reference to the definition in `main.cpp`'s object file. If `main.cpp` were omitted, the linker would report `undefined reference to greet()`.

---

#### Example 2 — Object File Per Translation Unit

```bash
# Compile each TU separately
g++ -std=c++17 -c main.cpp -o main.o    # TU 1 → main.o
g++ -std=c++17 -c other.cpp -o other.o  # TU 2 → other.o

# Link the object files
g++ main.o other.o -o app
```

**Why this matters:** Each `.cpp` file produces exactly one object file. The linker combines them. This is the essence of separate compilation. If `main.cpp` changes, only `main.o` is recompiled; `other.o` is reused.

---

### Real-World Cases with Explanation

**Case 1 — Header-only libraries:** Libraries like {fmt}, nlohmann/json, and Catch2 are header-only. They consist entirely of headers that are included into the user's TUs. There is no separate compilation or linking against a library file.

**Case 2 — ODR violations across TUs:** Two TUs might define the same class differently (e.g., one with `struct`, one with `class`). This violates the ODR and causes undefined behavior, often manifesting as subtle runtime bugs or linker errors. The solution is to put the class definition in a shared header.

**Case 3 — Anonymous namespaces for TU isolation:** A helper function defined in an anonymous namespace is unique to its TU. If two TUs both define `helper()` in anonymous namespaces, there is no conflict because the symbols have internal linkage.

**Case 4 — Inline variables and templates:** Inline functions and templates are exempt from the ODR's single-definition rule. They can be defined in headers and included in multiple TUs, provided all definitions are identical.

---

### References Links

- Translation units and linkage (C++) — Microsoft Learn - https://learn.microsoft.com/en-us/cpp/cpp/program-and-linkage-cpp
- Phases of translation — cppreference.com - https://en.cppreference.com/w/cpp/language/translation_phases
- One Definition Rule — cppreference.com - https://en.cppreference.com/w/cpp/language/definition

---

## 3. The Core Rule: One Definition Rule (ODR)

### Definitions

**Core Definition**
The One Definition Rule (ODR) states that a program shall contain exactly one definition of every non-inline function or variable that is odr-used in that program.

**Technical Definition**
The ODR is defined by the C++ standard, [basic.def.odr], paragraph 4 [ISO/IEC 14882-2014]: "Every program shall contain exactly one definition of every non-inline function or variable that is odr-used in that program; no diagnostic required". Additionally, no translation unit shall contain more than one definition of any variable, function, class type, enumeration type, or template. Class definitions, inline functions, and templates are exceptions: they can appear in multiple TUs, provided all definitions are identical.

**Beginner-Friendly Explanation**
The ODR is like a rule in a construction project: "Every part of the building must be defined exactly once." If two workers both build the same wall in different rooms, that's a violation — the building doesn't know which wall to use. In C++, if you define a function in two `.cpp` files, the linker gets confused. The ODR ensures that every function and variable has exactly one definition across the entire program.

---

### Purposes (all begin with "To")

- **To** ensure deterministic linking by preventing duplicate definitions.
- **To** allow separate compilation while maintaining a single source of truth for each entity.
- **To** enable class definitions and templates to be shared across TUs via headers.
- **To** provide a clear rule for what can be defined in headers vs. `.cpp` files.
- **To** prevent undefined behavior caused by conflicting definitions.

---

### Syntax Rules and Structure

#### Complete General Syntaxes with Breakdowns

**ODR-compliant: Class definition in a header**
```cpp
// shape.h
#ifndef SHAPE_H
#define SHAPE_H

class Shape {
public:
    virtual double area() const = 0;
    virtual ~Shape() = default;
};

#endif
```
- The class definition is identical in every TU that includes `shape.h`.
- Multiple definitions are allowed because class definitions are exempt from the single-definition rule.

**ODR-violating: Function definition in a header without `inline`**
```cpp
// bad.h
int add(int a, int b) {  // Non-inline function definition in a header
    return a + b;
}
```
- If `bad.h` is included in two TUs, `add` is defined twice → ODR violation.

**ODR-compliant: Function definition in a `.cpp` file**
```cpp
// math.cpp
#include "math.h"
int add(int a, int b) {  // Definition in one TU only
    return a + b;
}
```

#### Syntax Rules

- Every non-inline function or variable that is odr-used must be defined exactly once in the entire program.
- Class types, enumeration types, inline functions, class templates, and non-static function templates can have multiple definitions if they are identical.
- A definition can appear in the standard library, a user-defined library, or be implicitly defined.
- The compiler is not required to diagnose ODR violations; the behavior is undefined.
- ODR violations are often detected by the linker as "multiple definition" errors.

#### Constraints and Limitations

- Two definitions are considered identical only if they consist of the same sequence of tokens. Functionally equivalent but textually different definitions violate the ODR.
- Inline functions must be defined in every TU where they are odr-used.
- Templates must have identical definitions across all TUs.
- Header files are the primary mechanism for sharing identical definitions across TUs.
- ODR violations that are not caught by the linker result in undefined behavior, which may manifest as subtle runtime bugs.

---

### Multiple Annotated Complete Step-by-Step Code Examples and Their Expected Outputs

#### Example 1 — ODR-Compliant Class Definition in a Header

```cpp
// File: point.h
#ifndef POINT_H
#define POINT_H

// Class definition (allowed in multiple TUs because it's identical)
class Point {
public:
    Point(double x, double y) : x_(x), y_(y) {}
    double x() const { return x_; }
    double y() const { return y_; }

private:
    double x_;
    double y_;
};

#endif
```

```cpp
// File: main.cpp
#include "point.h"
#include <iostream>

int main() {
    Point p(3.0, 4.0);
    std::cout << "Point: (" << p.x() << ", " << p.y() << ")" << std::endl;
    return 0;
}
```

```cpp
// File: distance.cpp
#include "point.h"
#include <cmath>

double distance(const Point& a, const Point& b) {
    double dx = a.x() - b.x();
    double dy = a.y() - b.y();
    return std::sqrt(dx * dx + dy * dy);
}
```

**Compile and run:**
```bash
g++ -std=c++17 main.cpp distance.cpp -o app
./app
```

**Expected Output:**
```
Point: (3, 4)
```

**Why this output:** The class `Point` is defined in `point.h` and included in both `main.cpp` and `distance.cpp`. Both TUs have identical definitions of the class. This is allowed by the ODR exception for class types. If the two definitions were different (e.g., different member names), the ODR would be violated, resulting in undefined behavior.

---

#### Example 2 — ODR Violation (Multiple Definitions)

```cpp
// File: bad.h
#ifndef BAD_H
#define BAD_H

int global_value = 42;  // BAD: non-inline variable definition in a header

#endif
```

```cpp
// File: a.cpp
#include "bad.h"
void func_a() { global_value = 10; }
```

```cpp
// File: b.cpp
#include "bad.h"
void func_b() { global_value = 20; }
```

```cpp
// File: main.cpp
#include "bad.h"
void func_a();
void func_b();
int main() {
    func_a();
    func_b();
    return 0;
}
```

**Compile:**
```bash
g++ -std=c++17 a.cpp b.cpp main.cpp -o app
```

**Error (linker):**
```
multiple definition of `global_value'
```

**Why this error:** `global_value` is defined in `bad.h`, which is included in `a.cpp`, `b.cpp`, and `main.cpp`. Each TU defines `global_value`, violating the ODR. The linker detects multiple definitions and reports an error. The fix is to declare `global_value` as `extern` in the header and define it once in a `.cpp` file.

---

### Real-World Cases with Explanation

**Case 1 — The `struct` vs. `class` ODR violation:** A common ODR violation occurs when one TU defines a class using `struct` (default public) and another using `class` (default private). Although the members may be functionally identical, the token sequences differ, violating the ODR.

**Case 2 — Inline functions in headers:** Inline functions are exempt from the single-definition rule because they must be defined in every TU where they are used. This is essential for header-only libraries and templates.

**Case 3 — ODR and static data members:** A `static` data member of a class must be defined exactly once (typically in a `.cpp` file). Since C++17, `inline static` data members can be defined directly in the class definition, avoiding the need for a separate definition.

**Case 4 — ODR and C++20 modules:** Modules are designed to improve upon the ODR model by providing a single source of truth for declarations, reducing the risk of ODR violations and improving compilation speed.

---

### References Links

- One Definition Rule — cppreference.com - https://en.cppreference.com/w/cpp/language/definition
- DCL60-CPP. Obey the one-definition rule — SEI CERT - https://wiki.sei.cmu.edu/confluence/display/cplusplus/DCL60-CPP.+Obey+the+one-definition+rule
- Translation units and linkage (C++) — Microsoft Learn - https://learn.microsoft.com/en-us/cpp/cpp/program-and-linkage-cpp

---

## 4. Symbol Visibility (Linkage)

### Definitions

**Core Definition**
Linkage describes how a name (variable or function) can be accessed across translation units. **Internal linkage** means the name is visible only within its own TU. **External linkage** means the name is visible across all TUs in the program.

**Technical Definition**
A name with **internal linkage** can be referred to only from within its own translation unit. This is achieved with the `static` keyword at namespace scope or by declaring an entity in an anonymous namespace. A name with **external linkage** can be referred to from other translation units. Global functions and variables have external linkage by default. The `extern` keyword declares a name with external linkage without defining it. Since C++11, names in anonymous namespaces have internal linkage (previously, they had external linkage with a unique generated name).

**Beginner-Friendly Explanation**
Linkage determines who can see a symbol. Think of it like this: a `static` function is like a private diary — only you can read it. A regular function is like a public billboard — anyone in the program can see it. An anonymous namespace is like a private room — everything inside is invisible to the outside world. `extern` is like a signpost saying "this billboard exists somewhere else."

---

### Purposes (all begin with "To")

- **To** control whether a symbol is visible to other translation units.
- **To** prevent name collisions between helper functions in different TUs.
- **To** allow global variables and functions to be shared across TUs.
- **To** enable the ODR to be enforced by limiting definitions to one TU.
- **To** support encapsulation at the TU level.

---

### Syntax Rules and Structure

#### Complete General Syntaxes with Breakdowns

**Internal linkage — `static` at namespace scope**
```cpp
static int counter = 0;       // Variable with internal linkage
static void helper() { }      // Function with internal linkage
```
- `static` — storage-class specifier.
- The symbol is visible only within this TU.

**Internal linkage — anonymous namespace**
```cpp
namespace {
    int counter = 0;          // Internal linkage
    void helper() { }         // Internal linkage
}
```
- The anonymous namespace has a unique name per TU.
- All members have internal linkage (C++11 and later).

**External linkage — global variable/function**
```cpp
int global_counter = 0;       // External linkage (default)
void public_function() { }    // External linkage
```

**External linkage — `extern` declaration**
```cpp
extern int global_counter;    // Declaration with external linkage
```
- `extern` — tells the compiler the definition exists elsewhere.

#### Syntax Rules

- `static` at namespace scope gives internal linkage.
- `static` inside a function gives static storage duration but no linkage.
- `const` objects at namespace scope have internal linkage by default unless declared `extern`.
- Anonymous namespace members have internal linkage since C++11.
- `extern` declares a name with external linkage without defining it.
- A function declared `inline` has external linkage but is exempt from the ODR.

#### Constraints and Limitations

- Internal linkage entities cannot be used as template arguments in certain contexts (e.g., non-type template parameters in some cases).
- `extern "C"` has no effect on names with internal linkage in C++11 and later.
- Two TUs can have internal-linkage entities with the same name without conflict.
- External-linkage entities with the same name in two TUs cause ODR violations.
- The default linkage of a `const` variable can be overridden with `extern`.

---

### Multiple Annotated Complete Step-by-Step Code Examples and Their Expected Outputs

#### Example 1 — Internal vs. External Linkage

```cpp
// File: counter_a.cpp
#include <iostream>

// Internal linkage: only visible in this TU
static int counter = 0;

void increment_a() {
    counter++;
    std::cout << "counter_a: " << counter << std::endl;
}
```

```cpp
// File: counter_b.cpp
#include <iostream>

// Internal linkage: only visible in this TU (no conflict)
static int counter = 100;

void increment_b() {
    counter++;
    std::cout << "counter_b: " << counter << std::endl;
}
```

```cpp
// File: main.cpp
void increment_a();
void increment_b();

int main() {
    increment_a();  // prints 1
    increment_a();  // prints 2
    increment_b();  // prints 101
    increment_b();  // prints 102
    return 0;
}
```

**Compile and run:**
```bash
g++ -std=c++17 counter_a.cpp counter_b.cpp main.cpp -o app
./app
```

**Expected Output:**
```
counter_a: 1
counter_a: 2
counter_b: 101
counter_b: 102
```

**Why this output:** Each TU has its own `counter` variable because `static` gives internal linkage. There is no linker conflict. Without `static`, both files would define `counter` with external linkage, causing a "multiple definition" error.

---

#### Example 2 — Anonymous Namespace for Internal Linkage

```cpp
// File: utils.cpp
#include <iostream>

namespace {
    // Internal linkage — unique to this TU
    void log(const std::string& msg) {
        std::cout << "[LOG] " << msg << std::endl;
    }

    const int MAX_RETRIES = 3;
}

void process() {
    log("Processing...");
    log("Max retries: " + std::to_string(MAX_RETRIES));
}
```

```cpp
// File: main.cpp
void process();
int main() {
    process();
    return 0;
}
```

**Compile and run:**
```bash
g++ -std=c++17 utils.cpp main.cpp -o app
./app
```

**Expected Output:**
```
[LOG] Processing...
[LOG] Max retries: 3
```

**Why this output:** `log` and `MAX_RETRIES` are in an anonymous namespace, so they have internal linkage and are only visible in `utils.cpp`. `process` (which has external linkage) can call them. Attempting to call `log` from `main.cpp` would cause a linker error.

---

### Real-World Cases with Explanation

**Case 1 — Helper functions in libraries:** A library's `.cpp` file often contains helper functions that should not be exposed in the public API. Anonymous namespaces hide these symbols from the linker.

**Case 2 — Avoiding name collisions:** Two third-party libraries might both define a file-scope helper called `log_debug`. If both use anonymous namespaces or `static`, there is no ODR violation.

**Case 3 — Global configuration variables:** A program's configuration is declared `extern` in a header and defined once in a `.cpp` file. All other TUs reference the same variable.

**Case 4 — `const` and linkage:** A `const` variable at namespace scope has internal linkage by default. If you want a `const` variable with external linkage, you must declare it `extern` in the header and define it in a `.cpp` file.

---

### References Links

- Storage class specifiers — cppreference.com - https://en.cppreference.com/w/cpp/language/storage_duration
- Translation units and linkage (C++) — Microsoft Learn - https://learn.microsoft.com/en-us/cpp/cpp/program-and-linkage-cpp
- `extern` storage-class specifier — Microsoft Learn - https://learn.microsoft.com/en-us/cpp/cpp/extern-cpp
- CWG Issue 2772: Linkage effects of unnamed namespaces - https://cplusplus.github.io/CWG/issues/2772.html

---

## 5. Diagnostic Tracking

### Definitions

**Core Definition**
Diagnostic tracking is the practice of identifying which stage of the compilation pipeline produced an error — preprocessing, compilation, assembly, or linking — and understanding the root causes of errors at each stage.

**Technical Definition**
Each stage of the compilation pipeline produces distinct categories of errors:
- **Preprocessor errors**: Missing include files, malformed macros, unterminated conditionals.
- **Compiler errors**: Syntax errors, type mismatches, undeclared identifiers, semantic violations.
- **Assembler errors**: Invalid assembly instructions, relocations (rare in C++).
- **Linker errors**: Undefined references (missing definitions), multiple definitions (ODR violations), incompatible library versions.

**Beginner-Friendly Explanation**
When your code fails to build, the first question is: "Which stage failed?" A syntax error means the compiler couldn't understand your code. An "undefined reference" means the linker couldn't find a function you declared. A "multiple definition" means you defined the same thing twice. Knowing which stage failed tells you where to look for the bug.

---

### Purposes (all begin with "To")

- **To** quickly identify the root cause of a build failure.
- **To** distinguish between errors in your code and errors in the build configuration.
- **To** understand the difference between compilation errors and linking errors.
- **To** debug preprocessor issues caused by missing headers or macro conflicts.
- **To** resolve ODR violations detected by the linker.

---

### Syntax Rules and Structure

#### Diagnostic Stage Identification

| Stage | Error Type | Example Message | Root Cause |
|---|---|---|---|
| Preprocessing | Missing include | `fatal error: iostream: No such file or directory` | Header not found or misspelled |
| Preprocessing | Macro error | `error: 'X' was not declared in this scope` | Macro not defined or expanded incorrectly |
| Compilation | Syntax error | `error: expected ';' before '}' token` | Missing semicolon, bracket mismatch |
| Compilation | Type error | `error: cannot convert 'int' to 'std::string'` | Wrong type used |
| Compilation | Undeclared identifier | `error: 'foo' was not declared in this scope` | Missing declaration or include |
| Assembly | Invalid instruction | `Error: bad instruction` | Corrupted assembly or wrong architecture |
| Linking | Undefined reference | `undefined reference to 'foo()'` | Function declared but not defined |
| Linking | Multiple definition | `multiple definition of 'bar'` | ODR violation |
| Linking | Library not found | `cannot find -lmylib` | Library path or name incorrect |

#### Syntax Rules

- Preprocessor errors are reported before compilation begins.
- Compiler errors are reported during the compilation stage; no object file is produced.
- Assembler errors are reported during assembly; no object file is produced.
- Linker errors are reported after all object files are produced; no executable is produced.
- The compiler driver (`g++`) reports errors from all stages it invokes.

#### Constraints and Limitations

- Preprocessor errors can be cryptic because macros expand to code the compiler sees but you don't.
- Template errors are notoriously verbose and can be difficult to trace.
- Linker errors can be caused by missing libraries, ODR violations, or ABI mismatches.
- Runtime errors (crashes, incorrect behavior) occur after all compilation stages and are not part of the compilation model.

---

### Multiple Annotated Complete Step-by-Step Code Examples and Their Expected Outputs

#### Example 1 — Preprocessor Error (Missing Header)

```cpp
// File: main.cpp
#include <iosteam>  // Typo: should be <iostream>

int main() {
    std::cout << "Hello" << std::endl;
    return 0;
}
```

**Compile:**
```bash
g++ -std=c++17 main.cpp -o app
```

**Error:**
```
main.cpp:1:10: fatal error: iosteam: No such file or directory
    1 | #include <iosteam>
      |          ^~~~~~~~~
compilation terminated.
```

**Why this error:** The preprocessor cannot find the header `iosteam` (typo for `iostream`). This is a preprocessing error — it occurs before compilation begins. The fix is to correct the header name.

---

#### Example 2 — Compiler Error (Syntax)

```cpp
// File: main.cpp
#include <iostream>

int main() {
    std::cout << "Hello" << std::endl  // Missing semicolon
    return 0;
}
```

**Compile:**
```bash
g++ -std=c++17 main.cpp -o app
```

**Error:**
```
main.cpp: In function 'int main()':
main.cpp:4:38: error: expected ';' before 'return'
    4 |     std::cout << "Hello" << std::endl
      |                                      ^
      |                                      ;
```

**Why this error:** The compiler expected a semicolon after `std::endl` but found `return`. This is a compilation error — the compiler parsed the code and found a syntax violation. No object file is produced.

---

#### Example 3 — Linker Error (Undefined Reference)

```cpp
// File: math.h
#ifndef MATH_H
#define MATH_H
int add(int a, int b);  // Declaration
#endif
```

```cpp
// File: main.cpp
#include "math.h"
#include <iostream>

int main() {
    std::cout << add(2, 3) << std::endl;  // Calls add()
    return 0;
}
// Note: add() is declared but never defined in any TU
```

**Compile:**
```bash
g++ -std=c++17 main.cpp -o app
```

**Error:**
```
/usr/bin/ld: /tmp/ccXYZ.o: in function `main':
main.cpp:(.text+0x1a): undefined reference to `add(int, int)'
collect2: error: ld returned 1 exit status
```

**Why this error:** The compiler compiled `main.cpp` successfully because `add` was declared in `math.h`. However, the linker could not find a definition of `add` in any object file. This is a linker error — compilation succeeded, but linking failed. The fix is to provide a definition of `add` in a `.cpp` file and link it.

---

#### Example 4 — Linker Error (Multiple Definition)

```cpp
// File: bad.h
#ifndef BAD_H
#define BAD_H
int global_value = 42;  // BAD: definition in a header
#endif
```

```cpp
// File: a.cpp
#include "bad.h"
void func_a() { global_value = 10; }
```

```cpp
// File: b.cpp
#include "bad.h"
void func_b() { global_value = 20; }
```

```cpp
// File: main.cpp
#include "bad.h"
void func_a();
void func_b();
int main() {
    func_a();
    func_b();
    return 0;
}
```

**Compile:**
```bash
g++ -std=c++17 a.cpp b.cpp main.cpp -o app
```

**Error:**
```
/usr/bin/ld: /tmp/ccABC.o:(.bss+0x0): multiple definition of `global_value';
/tmp/ccDEF.o:(.bss+0x0): first defined here
collect2: error: ld returned 1 exit status
```

**Why this error:** `global_value` is defined in `bad.h`, which is included in `a.cpp`, `b.cpp`, and `main.cpp`. Each TU defines `global_value`, violating the ODR. The linker detects multiple definitions and reports an error. The fix is to declare `global_value` as `extern` in the header and define it once in a `.cpp` file.

---

### Real-World Cases with Explanation

**Case 1 — Missing `main()`:** A common linker error is forgetting to define `main()`. The linker reports `undefined reference to main` or `undefined symbols for architecture x86_64: "main"`.

**Case 2 — Library linking order:** On Unix, the order of libraries on the linker command line matters. If a library that depends on another appears before it, the linker may report undefined references. The fix is to reorder the libraries or use `--start-group` / `--end-group`.

**Case 3 — Template instantiation errors:** Templates are instantiated per TU. If a template is declared but not defined in a TU where it is used, the compiler may not instantiate it, leading to undefined references at link time. The fix is to define templates in headers or use explicit instantiation.

**Case 4 — ABI mismatch:** Mixing object files compiled with different compilers or different standard library versions can cause linker errors or runtime crashes. The C++ ABI (e.g., Itanium C++ ABI) defines name mangling, exception handling, and RTTI. Mismatched ABIs cause symbol resolution failures.

**Case 5 — `--save-temps` for debugging:** The `--save-temps` flag preserves intermediate files, allowing you to inspect the preprocessed output (`.ii`), assembly (`.s`), and object file (`.o`) to pinpoint which stage produced an error.

---

### References Links

- Linker errors — Microsoft Learn - https://learn.microsoft.com/en-us/cpp/error-messages/tool-errors/linker-tools-errors-and-warnings
- Undefined reference — Stack Overflow - https://stackoverflow.com/questions/12573816/what-is-an-undefined-reference-unresolved-external-symbol-error-and-how-do-i-fix
- Multiple definition — Stack Overflow - https://stackoverflow.com/questions/1872639/what-is-a-multiple-definition-error
- The C++ ABI — ELTE - https://gsd.web.elte.hu/lectures/multi/pdf/BertalanD-cpp-abi.pdf
- `--save-temps` — GCC Documentation - https://gcc.gnu.org/onlinedocs/gcc/Overall-Options.html

---

## Summary Table of Core Concepts

| Concept | Key Mechanism | Primary Benefit | Key Risk / Constraint |
|---|---|---|---|
| 4-Stage Pipeline | Preprocessing → Compilation → Assembly → Linking | Transforms source to executable; enables incremental builds | Errors at different stages require different debugging strategies |
| Translation Units | Source file + all `#include`d headers | Independent compilation; parallel builds | ODR violations across TUs; header pollution |
| One Definition Rule | One definition per non-inline symbol | Deterministic linking; single source of truth | Undefined behavior if violated; often detected by linker |
| Symbol Visibility | `static` (internal) vs. `extern` (external) | Encapsulation; collision prevention | Internal linkage limits template usage; extern requires a definition |
| Diagnostics | Stage-specific error identification | Faster debugging; root cause analysis | Linker errors can be cryptic; ABI mismatches are hard to diagnose |

---

## References (Consolidated)

- Phases of translation — cppreference.com - https://en.cppreference.com/w/cpp/language/translation_phases
- Translation units and linkage (C++) — Microsoft Learn - https://learn.microsoft.com/en-us/cpp/cpp/program-and-linkage-cpp
- One Definition Rule — cppreference.com - https://en.cppreference.com/w/cpp/language/definition
- Storage class specifiers — cppreference.com - https://en.cppreference.com/w/cpp/language/storage_duration
- DCL60-CPP. Obey the one-definition rule — SEI CERT - https://wiki.sei.cmu.edu/confluence/display/cplusplus/DCL60-CPP.+Obey+the+one-definition+rule
- Overall Options (GCC) — https://gcc.gnu.org/onlinedocs/gcc/Overall-Options.html
- C++ Compilation — Caltech CS11 - http://courses.cms.caltech.edu/cs11/material/cpp/donnie/lectures/cs11-cpp-lec2.pdf
- The C++ ABI — ELTE - https://gsd.web.elte.hu/lectures/multi/pdf/BertalanD-cpp-abi.pdf
- CWG Issue 2772: Linkage effects of unnamed namespaces - https://cplusplus.github.io/CWG/issues/2772.html
- `extern` storage-class specifier — Microsoft Learn - https://learn.microsoft.com/en-us/cpp/cpp/extern-cpp