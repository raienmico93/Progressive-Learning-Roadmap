# C++ Preprocessor Directives — Comprehensive Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
Preprocessor directives are instructions to the C++ preprocessor — the program that runs before the compiler — that control file inclusion, macro substitution, conditional compilation, and diagnostic generation. Each directive occupies one line and begins with the `#` character.

**Technical Definition**
The preprocessor is executed at translation phase 4, before the actual compilation. The result of preprocessing is a single file which is then passed to the compiler. Preprocessing directives control the behavior of the preprocessor and must not come from macro expansion. The standard-defined directives include `#define`, `#undef`, `#include`, `#if`, `#ifdef`, `#ifndef`, `#else`, `#elif`, `#elifdef`, `#elifndef`, `#endif`, `#line`, `#error`, `#warning`, `#pragma`, `#embed`, and module/import directives.

**Beginner-Friendly Explanation**
Think of the preprocessor as a text-editing assistant that runs before the compiler. When you write `#include <iostream>`, the assistant copies the contents of the `iostream` file into your source. When you write `#define PI 3.14`, the assistant replaces every `PI` with `3.14`. When you write `#if`, the assistant decides whether to keep or throw away code before the compiler sees it. The `#` at the start of a line is the signal: "This is an instruction for the assistant, not for the compiler."

---

### Key Characteristics

- **Line-oriented**: Each directive occupies exactly one logical line; the `#` must be the first non-whitespace character.
- **Textual substitution**: The preprocessor operates on preprocessing tokens, not on the program's semantic structure.
- **Conditional compilation**: Code can be included or excluded based on preprocessor-time conditions.
- **Macro expansion**: Identifiers can be replaced with arbitrary token sequences, with operators for stringification (`#`) and concatenation (`##`).
- **Not macro-expandable**: The `#` that begins a directive cannot come from macro expansion; if `foo` is defined as `define`, `#foo` is not a valid directive.
- **Implementation-defined extensions**: Non-standard directives like `#warning` (standardized in C++23) and `#pragma once` are widely supported.

---

### Prerequisites

- Basic C++ syntax and familiarity with the compilation pipeline (preprocessing → compilation → assembly → linking).
- Understanding of translation units and the `#include` mechanism.
- A compiler with C++17 or later for `__has_include`; C++20 or later for modules; C++23 or later for `#warning`; C++26 or later for `#embed`.

---

### Related Programming Areas

- **Header files and the include model** (what the preprocessor manages).
- **C++20 modules** (a modern alternative to `#include`).
- **Build systems** (CMake, Make) that define macros via command-line flags.
- **Platform abstraction** (using conditional compilation for cross-platform code).
- **Feature detection** (`__has_include`, `__has_cpp_attribute`, feature-test macros).

---

### Core Concepts / Features

1. **File Inclusion Mechanics** — Textual substitution via `#include` (angle brackets vs. quotes).
2. **String Token Replacement** — Constant definitions and stringification utilizing `#define` and `#undef`.
3. **Logical Flow Evaluation** — Directing code filtering with `#if`, `#elif`, `#else`, and `#endif`.
4. **Existence Evaluations** — Macro presence tracking with `#ifdef`, `#ifndef`, and the unified `defined()` operator.
5. **Modern Assets & Introspection** — Conditional asset testing via `__has_include` and `__has_cpp_attribute`.
6. **Resource Embedding** — Direct binary asset injection into source code using the C++26 `#embed` directive.
7. **Diagnostic Routing** — Enforcing custom build aborts and warnings via `#error` and `#warning`.

---

## 1. File Inclusion Mechanics (`#include`)

### Definitions

**Core Definition**
The `#include` directive causes the preprocessor to replace the directive line with the entire contents of a specified file, effectively inserting that file's text into the current translation unit.

**Technical Definition**
A `#include` directive shall identify a header or source file that can be processed by the implementation. The form `#include <h-char-sequence>` searches a sequence of implementation-defined places for a header identified uniquely by the sequence between the `<` and `>` delimiters. The form `#include "q-char-sequence"` causes the replacement of the directive by the entire contents of the source file identified by the sequence between the `"` delimiters. If the quoted search fails, the directive is reprocessed as if it read `#include <h-char-sequence>`.

**Beginner-Friendly Explanation**
`#include` is like saying "paste the contents of this file here." Angle brackets `<...>` tell the preprocessor to look in the system's standard header directories (for standard library headers). Double quotes `"..."` tell it to look first in the current directory (for your own headers), then fall back to the system directories.

---

### Purposes (all begin with "To")

- **To** share declarations and definitions across multiple translation units without manual duplication.
- **To** organize code into logical modules (header files) that can be included where needed.
- **To** provide access to the standard library and third-party libraries.
- **To** enable separate compilation by giving each translation unit access to shared interfaces.
- **To** support conditional inclusion of headers based on platform or configuration.

---

### Syntax Rules and Structure

#### Complete General Syntaxes with Breakdowns

**Syntax 1 — Angle-bracket form**
```cpp
#include <iostream>
```
- The preprocessor searches implementation-defined places (system include directories).
- Used for standard library headers and system headers.

**Syntax 2 — Quoted form**
```cpp
#include "my_header.h"
```
- The preprocessor searches the current file's directory first, then falls back to the angle-bracket search path.
- Used for project headers and local files.

**Syntax 3 — Computed include (macro-expanded)**
```cpp
#include HEADER_NAME
```
- If the tokens after `include` do not match either form, they are macro-expanded and the result is re-examined.
- If the result still does not match, the behavior is undefined.

#### Syntax Rules

- The `#` must be the first non-whitespace character on the line.
- The directive extends to the end of the line; the line may be continued with a backslash-newline.
- The file name in the angle-bracket form is not subject to macro expansion.
- Include files can be nested; an `#include` directive can appear in a file included by another `#include`.
- The preprocessor stops searching as soon as it finds a file with the given name.

#### Constraints and Limitations

- The search order for quoted includes after the current directory is implementation-defined.
- Computed includes are permitted but undefined if the result doesn't match one of the two standard forms.
- Include guards or `#pragma once` are essential to prevent multiple inclusion.
- Including the same file twice without guards causes redefinition errors.

---

### Multiple Annotated Complete Step-by-Step Code Examples and Their Expected Outputs

#### Example 1 — Basic Include with Both Forms

```cpp
// File: my_utils.h
#ifndef MY_UTILS_H
#define MY_UTILS_H
int square(int x);
#endif
```

```cpp
// File: main.cpp
// Step 1: Angle-bracket include for standard library
#include <iostream>

// Step 2: Quoted include for project header
#include "my_utils.h"

int square(int x) { return x * x; }

int main() {
    std::cout << "square(5) = " << square(5) << std::endl;
    return 0;
}
```

**Compile and run:**
```bash
g++ -std=c++17 main.cpp -o app
./app
```

**Expected Output:**
```
square(5) = 25
```

**Why this output:** The angle-bracket include brings in `std::cout` and `std::endl` from the standard library. The quoted include brings in the declaration of `square` from the local header. The quoted form searches the current directory first, finds `my_utils.h`, and inserts its contents.

---

#### Example 2 — Nested Includes

```cpp
// File: config.h
#ifndef CONFIG_H
#define CONFIG_H
#define APP_VERSION "1.0"
#endif
```

```cpp
// File: logger.h
#ifndef LOGGER_H
#define LOGGER_H
#include "config.h"   // Nested include
void log(const char* msg);
#endif
```

```cpp
// File: main.cpp
#include "logger.h"
#include <iostream>

void log(const char* msg) {
    std::cout << "[v" << APP_VERSION << "] " << msg << std::endl;
}

int main() {
    log("Application started");
    return 0;
}
```

**Expected Output:**
```
[v1.0] Application started
```

**Why this output:** `main.cpp` includes `logger.h`, which includes `config.h`. The macro `APP_VERSION` is defined in `config.h` and becomes visible in `main.cpp` after the nested include. The include guard in `config.h` prevents it from being included twice if other headers also include it.

---

### Real-World Cases with Explanation

**Case 1 — Standard library headers:** Every C++ program uses `#include <iostream>`, `#include <vector>`, etc. The angle-bracket form tells the preprocessor to search the compiler's system include directories, where the standard library headers reside.

**Case 2 — Project headers:** In a project with `src/` and `include/` directories, `#include "my_lib/core.h"` searches the current directory first, then the project's include path (configured with `-I` flags).

**Case 3 — Third-party libraries:** Libraries like nlohmann/json are included as `#include "nlohmann/json.hpp"` (vendored) or `#include <nlohmann/json.hpp>` (system-installed), depending on how the build system is configured.

**Case 4 — Platform abstraction:** A project might have `#include "platform_win32.h"` on Windows and `#include "platform_posix.h"` on Unix, selected via conditional compilation.

---

### References Links

- Source file inclusion — cppreference.com - https://en.cppreference.com/w/cpp/preprocessor/include
- `#include` directive (C/C++) — Microsoft Learn - https://learn.microsoft.com/en-us/cpp/preprocessor/hash-include-directive-c-cpp
- Preprocessor — cppreference.com - https://en.cppreference.com/w/cpp/preprocessor

---

## 2. String Token Replacement (`#define` and `#undef`)

### Definitions

**Core Definition**
`#define` creates a macro — a name that the preprocessor replaces with a specified token sequence. `#undef` removes a macro definition, causing the name to no longer be replaced.

**Technical Definition**
The `#define` directive defines an object-like macro (`#define NAME replacement`) or a function-like macro (`#define NAME(params) replacement`). The replacement list is a sequence of preprocessing tokens. The `#` operator (stringification) converts a macro parameter into a string literal. The `##` operator (concatenation) joins two adjacent tokens. The `#undef` directive removes the definition of an identifier, after which the identifier is no longer a macro.

**Beginner-Friendly Explanation**
`#define` is like creating a shortcut. If you write `#define PI 3.14159`, then every time the preprocessor sees `PI`, it replaces it with `3.14159`. Function-like macros take arguments: `#define SQUARE(x) ((x) * (x))` replaces `SQUARE(5)` with `((5) * (5))`. `#undef` is the undo button — it removes the shortcut so `PI` is just a regular identifier again.

---

### Purposes (all begin with "To")

- **To** define symbolic constants that are substituted at preprocessing time.
- **To** create shorthand notations for frequently used code fragments.
- **To** implement conditional compilation flags (e.g., `#define DEBUG`).
- **To** generate string literals from macro arguments using stringification (`#`).
- **To** concatenate tokens to form new identifiers using `##`.
- **To** remove macro definitions to avoid name pollution using `#undef`.

---

### Syntax Rules and Structure

#### Complete General Syntaxes with Breakdowns

**Syntax 1 — Object-like macro**
```cpp
#define PI 3.14159
```
- `PI` is replaced by `3.14159` wherever it appears.

**Syntax 2 — Function-like macro**
```cpp
#define SQUARE(x) ((x) * (x))
```
- `SQUARE(5)` expands to `((5) * (5))`.
- Parameters must be enclosed in parentheses in the replacement to avoid precedence issues.

**Syntax 3 — Stringification (`#`)**
```cpp
#define STRINGIFY(x) #x
STRINGIFY(hello)  // expands to "hello"
```

**Syntax 4 — Token concatenation (`##`)**
```cpp
#define CONCAT(a, b) a##b
CONCAT(foo, bar)  // expands to foobar
```

**Syntax 5 — Undefining a macro**
```cpp
#undef PI
```

#### Syntax Rules

- A macro definition lasts until the end of the translation unit or until `#undef` is encountered.
- Function-like macros require `(` immediately after the name (no whitespace).
- The `#` and `##` operators can only be used in macro replacement lists.
- `#undef` on a name that is not a macro has no effect (it is not an error).
- Redefining a macro with a different replacement list is an error unless the new definition is identical.

#### Constraints and Limitations

- Macros are text-based; they do not respect scope, type, or namespace.
- Function-like macros can evaluate arguments multiple times (e.g., `SQUARE(x++)` increments twice).
- Macros can be accidentally redefined or leak into headers.
- `#define` and `#undef` cannot be used inside macro expansion.
- The replacement list cannot contain a directive (e.g., `#include`).

---

### Multiple Annotated Complete Step-by-Step Code Examples and Their Expected Outputs

#### Example 1 — Object-Like and Function-Like Macros

```cpp
#include <iostream>

// Step 1: Object-like macro
#define PI 3.14159

// Step 2: Function-like macro with parentheses
#define SQUARE(x) ((x) * (x))

// Step 3: Stringification macro
#define TO_STRING(x) #x

// Step 4: Concatenation macro
#define MAKE_FUNC(name) void func_##name() { \
    std::cout << "Called func_" << #name << std::endl; }

MAKE_FUNC(hello)

int main() {
    std::cout << "PI = " << PI << std::endl;
    std::cout << "SQUARE(5) = " << SQUARE(5) << std::endl;
    std::cout << "String: " << TO_STRING(hello world) << std::endl;

    func_hello();
    return 0;
}
```

**Expected Output:**
```
PI = 3.14159
SQUARE(5) = 25
String: hello world
Called func_hello
```

**Why this output:** `PI` is replaced with `3.14159`. `SQUARE(5)` expands to `((5) * (5))` = 25. `TO_STRING(hello world)` expands to `"hello world"`. `MAKE_FUNC(hello)` uses `##` to create `func_hello` and `#` to create the string `"hello"`.

---

#### Example 2 — `#undef` to Avoid Pollution

```cpp
#include <iostream>

#define DEBUG 1

void debug_log(const char* msg) {
#ifdef DEBUG
    std::cout << "[DEBUG] " << msg << std::endl;
#endif
}

// Step 1: Remove the DEBUG macro
#undef DEBUG

void production_log(const char* msg) {
#ifdef DEBUG
    std::cout << "[DEBUG] " << msg << std::endl;
#else
    std::cout << "[INFO] " << msg << std::endl;
#endif
}

int main() {
    debug_log("debug message");
    production_log("production message");
    return 0;
}
```

**Expected Output:**
```
[DEBUG] debug message
[INFO] production message
```

**Why this output:** Before `#undef DEBUG`, `debug_log` compiles the `#ifdef DEBUG` branch. After `#undef DEBUG`, the macro is no longer defined, so `production_log` takes the `#else` branch. This demonstrates how `#undef` can selectively disable macros in different parts of a file.

---

### Real-World Cases with Explanation

**Case 1 — Platform detection:** `#define WINDOWS` or `#define LINUX` is set based on the compiler's predefined macros, then used for conditional compilation of platform-specific code.

**Case 2 — Debug vs. release builds:** `#define DEBUG` is set via the build system (`-DDEBUG`), and `#ifdef DEBUG` controls whether assertion and logging code is compiled.

**Case 3 — Header guards:** `#define MY_HEADER_H` and `#ifndef MY_HEADER_H` work together to prevent multiple inclusion.

**Case 4 — Token pasting for code generation:** Macro libraries (e.g., Boost.Preprocessor) use `##` to generate function names, class names, and variable names programmatically.

---

### References Links

- `#define` directive — Microsoft Learn - https://learn.microsoft.com/en-us/cpp/preprocessor/hash-define-directive-c-cpp
- `#undef` directive — Microsoft Learn - https://learn.microsoft.com/en-us/cpp/preprocessor/hash-undef-directive-c-cpp
- Replacement macros — cppreference.com - https://en.cppreference.com/w/cpp/preprocessor/replace

---

## 3. Logical Flow Evaluation (`#if`, `#elif`, `#else`, `#endif`)

### Definitions

**Core Definition**
Conditional compilation directives control which portions of a source file are compiled based on the evaluation of constant expressions at preprocessing time.

**Technical Definition**
The `#if` directive, with the `#elif`, `#else`, and `#endif` directives, controls compilation of portions of a source file. If the constant expression after `#if` evaluates to a nonzero value, the group of lines immediately following it is retained in the translation unit. Each `#if` must be matched by an `#endif`. Any number of `#elif` directives can appear between `#if` and `#endif`, and at most one `#else` directive is permitted.

**Beginner-Friendly Explanation**
Conditional compilation is like a "choose your own adventure" book. The preprocessor reads the condition after `#if`. If it's true, it keeps the code that follows. If it's false, it skips to the next `#elif` or `#else`. The `#endif` marks the end of the choice. This is how the same source file can compile differently on Windows vs. Linux, or in debug vs. release mode.

---

### Purposes (all begin with "To")

- **To** compile different code for different platforms (Windows, Linux, macOS).
- **To** enable or disable debugging code based on build configuration.
- **To** support multiple language standard versions in a single codebase.
- **To** handle compiler-specific features that may or may not be available.
- **To** provide fallback implementations when optional libraries are missing.

---

### Syntax Rules and Structure

#### Complete General Syntax with Breakdown

```cpp
#if constant-expression
    // code if true
#elif constant-expression
    // code if the first was false and this is true
#else
    // code if all above were false
#endif
```

**Component breakdown:**
- `#if` — starts the conditional block; the constant expression must be an integer constant expression.
- `#elif` — optional; provides additional conditions.
- `#else` — optional; provides the fallback block.
- `#endif` — required; closes the conditional block.

#### Syntax Rules

- The constant expression must have integer type and may contain integer constants, character constants, and the `defined` operator.
- Macro replacement is performed on the expression before evaluation.
- `#if` directives can be nested; each nested `#else`, `#elif`, or `#endif` belongs to the nearest preceding `#if`.
- All conditional directives must be matched before the end of the file.
- `#elifdef` and `#elifndef` (C++23) provide shorthand for `#elif defined(...)` and `#elif !defined(...)`.

#### Constraints and Limitations

- The expression cannot use `sizeof` or type-cast operators.
- The expression must not query the environment or depend on implementation details of the target machine.
- Character constants may be converted to a different code value set than the target environment's.
- Unmatched `#if`/`#endif` pairs cause compilation errors.

---

### Multiple Annotated Complete Step-by-Step Code Examples and Their Expected Outputs

#### Example 1 — Platform-Specific Compilation

```cpp
#include <iostream>

int main() {
#if defined(_WIN32)
    std::cout << "Running on Windows" << std::endl;
#elif defined(__linux__)
    std::cout << "Running on Linux" << std::endl;
#elif defined(__APPLE__)
    std::cout << "Running on macOS" << std::endl;
#else
    std::cout << "Unknown platform" << std::endl;
#endif
    return 0;
}
```

**Expected Output (on Linux):**
```
Running on Linux
```

**Why this output:** The preprocessor checks `_WIN32` first. If not defined, it checks `__linux__`. On a Linux system, `__linux__` is predefined by the compiler, so the Linux branch is compiled. The other branches are discarded before the compiler sees them.

---

#### Example 2 — Feature Detection with Nested Conditionals

```cpp
#include <iostream>

int main() {
#if __cplusplus >= 202002L
    std::cout << "C++20 or later" << std::endl;
    #if __has_include(<concepts>)
        std::cout << "Concepts header available" << std::endl;
    #else
        std::cout << "Concepts header not available" << std::endl;
    #endif
#elif __cplusplus >= 201703L
    std::cout << "C++17" << std::endl;
#else
    std::cout << "Pre-C++17" << std::endl;
#endif
    return 0;
}
```

**Expected Output (with C++20 and Concepts):**
```
C++20 or later
Concepts header available
```

**Why this output:** `__cplusplus` is a predefined macro that expands to the C++ standard version (e.g., `202002L` for C++20). The nested `#if __has_include(<concepts>)` checks whether the `<concepts>` header exists. Both conditions are evaluated at preprocessing time, and only the selected branches are compiled.

---

### Real-World Cases with Explanation

**Case 1 — Windows vs. POSIX API:** Code that uses `CreateFile` on Windows and `open` on Linux is wrapped in `#if defined(_WIN32)` / `#else` blocks, allowing the same source file to compile on both platforms.

**Case 2 — Standard version compatibility:** `#if __cplusplus >= 201703L` enables C++17 features, while `#else` provides a fallback implementation for older standards.

**Case 3 — Compiler-specific features:** `#if defined(__GNUC__) && __GNUC__ >= 10` enables GCC-specific optimizations or attributes.

**Case 4 — Library feature detection:** `#if __has_include(<optional>)` checks for the `<optional>` header and uses `std::optional` if available, otherwise falls back to a custom implementation.

---

### References Links

- Conditional inclusion — cppreference.com - https://en.cppreference.com/w/cpp/preprocessor/conditional
- `#if`, `#elif`, `#else`, and `#endif` directives — Microsoft Learn - https://learn.microsoft.com/en-us/cpp/preprocessor/hash-if-hash-elif-hash-else-and-hash-endif-directives-c-cpp
- Preprocessor directives — Microsoft Learn - https://learn.microsoft.com/en-us/cpp/preprocessor/preprocessor-directives

---

## 4. Existence Evaluations (`#ifdef`, `#ifndef`, `defined()`)

### Definitions

**Core Definition**
`#ifdef` and `#ifndef` test whether a macro name is currently defined. The `defined` operator provides the same functionality within `#if` and `#elif` expressions.

**Technical Definition**
The `#ifdef identifier` directive is equivalent to `#if defined(identifier)`. The `#ifndef identifier` directive is equivalent to `#if !defined(identifier)`. The `defined` operator can be used in `#if` and `#elif` constant expressions as `defined(identifier)` or `defined identifier`. It evaluates to 1 if the identifier is currently defined as a macro, and 0 otherwise. These directives check only for the presence or absence of identifiers defined with `#define`, not for identifiers declared in the C++ source code.

**Beginner-Friendly Explanation**
`#ifdef` asks: "Is this macro defined?" If yes, the code that follows is compiled. `#ifndef` asks the opposite: "Is this macro not defined?" These are the workhorses of include guards — `#ifndef MY_HEADER_H` ensures a header is only included once. The `defined()` operator does the same thing but can be combined with other conditions in a `#if` expression.

---

### Purposes (all begin with "To")

- **To** implement include guards that prevent multiple inclusion of headers.
- **To** conditionally compile code based on whether a feature macro is set.
- **To** provide a portable way to check for the existence of a macro without evaluating its value.
- **To** combine macro existence checks with other conditions using the `defined` operator.
- **To** support command-line macro definitions (`-DDEBUG`) that control compilation.

---

### Syntax Rules and Structure

#### Complete General Syntaxes with Breakdowns

**Syntax 1 — `#ifdef`**
```cpp
#ifdef MY_MACRO
    // compiled if MY_MACRO is defined
#endif
```

**Syntax 2 — `#ifndef`**
```cpp
#ifndef MY_HEADER_H
#define MY_HEADER_H
    // header contents
#endif
```

**Syntax 3 — `defined()` in `#if`**
```cpp
#if defined(MY_MACRO)
    // compiled if MY_MACRO is defined
#endif

#if defined(MY_MACRO) && !defined(OTHER_MACRO)
    // combined conditions
#endif
```

#### Syntax Rules

- `#ifdef identifier` is exactly equivalent to `#if defined(identifier)`.
- `#ifndef identifier` is exactly equivalent to `#if !defined(identifier)`.
- The `defined` operator can appear only in `#if` and `#elif` directives, not in other contexts.
- `defined` can be written as `defined(identifier)` or `defined identifier` (parentheses optional).
- An identifier defined as empty text is considered defined.

#### Constraints and Limitations

- `#ifdef` and `#ifndef` check only for preprocessor macros, not for variables, functions, or types.
- These directives cannot evaluate the value of a macro; they only check existence.
- The `defined` operator cannot be used in regular C++ code, only in preprocessor conditions.
- MSVC allows up to 30 macros via `/D` command-line options.

---

### Multiple Annotated Complete Step-by-Step Code Examples and Their Expected Outputs

#### Example 1 — Include Guard (The Classic Use Case)

```cpp
// File: my_header.h
#ifndef MY_HEADER_H
#define MY_HEADER_H

int add(int a, int b);

#endif // MY_HEADER_H
```

```cpp
// File: main.cpp
#include "my_header.h"
#include "my_header.h"  // Second inclusion — skipped by guard
#include <iostream>

int add(int a, int b) { return a + b; }

int main() {
    std::cout << "add(2, 3) = " << add(2, 3) << std::endl;
    return 0;
}
```

**Expected Output:**
```
add(2, 3) = 5
```

**Why this output:** The first inclusion defines `MY_HEADER_H`. The second inclusion checks `#ifndef MY_HEADER_H`, finds it defined, and skips the contents. Without the guard, `int add(int, int);` would be declared twice, causing a redefinition error.

---

#### Example 2 — Command-Line Macro Control

```cpp
#include <iostream>

int main() {
#ifdef DEBUG
    std::cout << "Debug mode enabled" << std::endl;
#else
    std::cout << "Release mode" << std::endl;
#endif

#if defined(VERBOSE) && defined(DEBUG)
    std::cout << "Verbose debug mode" << std::endl;
#endif

    return 0;
}
```

**Compile with debug flag:**
```bash
g++ -std=c++17 -DDEBUG -DVERBOSE main.cpp -o app
./app
```

**Expected Output:**
```
Debug mode enabled
Verbose debug mode
```

**Compile without flags:**
```bash
g++ -std=c++17 main.cpp -o app
./app
```

**Expected Output:**
```
Release mode
```

**Why this output:** The `-D` flag defines the macro as if `#define DEBUG` appeared at the top of the file. When both `DEBUG` and `VERBOSE` are defined, the combined condition `defined(VERBOSE) && defined(DEBUG)` is true, so the extra message is printed.

---

### Real-World Cases with Explanation

**Case 1 — Header guards everywhere:** Every header file uses `#ifndef`/`#define`/`#endif` to prevent multiple inclusion. This is the most common use of `#ifdef` and `#ifndef`.

**Case 2 — Build configuration:** `#ifdef NDEBUG` controls whether assertions are compiled (the standard library defines `NDEBUG` in release builds to disable `assert`).

**Case 3 — Optional features:** A library might provide `#ifdef HAS_OPENSSL` to enable SSL support only when OpenSSL is available.

**Case 4 — Compiler detection:** `#ifdef __GNUC__` or `#ifdef _MSC_VER` detects the compiler and enables compiler-specific workarounds.

---

### References Links

- `#ifdef` and `#ifndef` directives — Microsoft Learn - https://learn.microsoft.com/en-us/cpp/preprocessor/hash-ifdef-and-hash-ifndef-directives-c-cpp
- Conditional inclusion — cppreference.com - https://en.cppreference.com/w/cpp/preprocessor/conditional
- `defined` operator — Microsoft Learn - https://learn.microsoft.com/en-us/cpp/preprocessor/hash-if-hash-elif-hash-else-and-hash-endif-directives-c-cpp

---

## 5. Modern Assets & Introspection (`__has_include`, `__has_cpp_attribute`)

### Definitions

**Core Definition**
`__has_include` is a preprocessor operator that checks whether a header file exists and can be included. `__has_cpp_attribute` checks whether a specific attribute is supported by the compiler.

**Technical Definition**
`__has_include(header-name)` performs a check on the header file name in the same way a `#include` directive would interpret it. It expands to 1 if the header is found, 0 otherwise. `__has_cpp_attribute(attribute-token)` checks for the presence of an attribute; for standard attributes, it expands to the year and month in which the attribute was added to the working draft. Both are part of C++17 and are treated as defined macros by `#ifdef`, `#ifndef`, and `defined`.

**Beginner-Friendly Explanation**
`__has_include` lets you ask the preprocessor: "Does this header exist on this system?" This is useful for optionally using a library without breaking compilation on systems where it's not available. `__has_cpp_attribute` asks: "Does this compiler support this attribute?" Both are tools for writing portable code that adapts to its environment.

---

### Purposes (all begin with "To")

- **To** conditionally include a header only if it exists, enabling optional library support.
- **To** provide fallback implementations when a header or attribute is unavailable.
- **To** write portable code that adapts to different compilers and platforms.
- **To** avoid compilation errors from missing headers or unsupported attributes.
- **To** enable feature detection without relying on compiler version macros.

---

### Syntax Rules and Structure

#### Complete General Syntaxes with Breakdowns

**Syntax 1 — `__has_include`**
```cpp
#if __has_include(<optional>)
    #include <optional>
    #define HAVE_OPTIONAL 1
#else
    #define HAVE_OPTIONAL 0
#endif
```
- `__has_include(<optional>)` evaluates to 1 if `<optional>` can be included.
- Can also be used with quoted form: `__has_include("my_header.h")`.

**Syntax 2 — `__has_cpp_attribute`**
```cpp
#if __has_cpp_attribute(nodiscard)
    #define NODISCARD [[nodiscard]]
#else
    #define NODISCARD
#endif
```
- `__has_cpp_attribute(nodiscard)` evaluates to a nonzero value if `[[nodiscard]]` is supported.

#### Syntax Rules

- `__has_include` can only appear in the controlling expression of `#if` or `#elif`.
- `__has_cpp_attribute` can only appear in `#if` or `#elif`.
- The identifier `__has_include` shall not appear in any other context.
- These operators are treated as defined macros by `#ifdef`, `#ifndef`, and `defined`.
- The header name in `__has_include` is interpreted as if it were in a `#include` directive.

#### Constraints and Limitations

- `__has_include` is part of C++17; older compilers may not support it (GCC 5.1+, Clang, MSVC 2017+).
- `__has_cpp_attribute` is part of C++17; GCC supports it since 5.1.
- A true result from `__has_include` does not guarantee the header is usable — it only guarantees it exists.
- `__has_cpp_attribute` for vendor-specific attributes returns a nonzero value determined by the implementation.

---

### Multiple Annotated Complete Step-by-Step Code Examples and Their Expected Outputs

#### Example 1 — Optional Header with `__has_include`

```cpp
#include <iostream>

#if __has_include(<filesystem>)
    #include <filesystem>
    namespace fs = std::filesystem;
    #define HAS_FILESYSTEM 1
#elif __has_include(<experimental/filesystem>)
    #include <experimental/filesystem>
    namespace fs = std::experimental::filesystem;
    #define HAS_FILESYSTEM 1
#else
    #define HAS_FILESYSTEM 0
#endif

int main() {
#if HAS_FILESYSTEM
    std::cout << "Filesystem support available" << std::endl;
    fs::path p = "/tmp/test";
    std::cout << "Path: " << p << std::endl;
#else
    std::cout << "Filesystem not available" << std::endl;
#endif
    return 0;
}
```

**Expected Output (with C++17 filesystem):**
```
Filesystem support available
Path: "/tmp/test"
```

**Why this output:** `__has_include(<filesystem>)` evaluates to 1 on a C++17 compiler with filesystem support. The `#if` branch is compiled, making `std::filesystem` available. If the compiler only has the experimental version, the `#elif` branch is used. If neither exists, the fallback is compiled.

---

#### Example 2 — Attribute Detection with `__has_cpp_attribute`

```cpp
#include <iostream>

#if __has_cpp_attribute(nodiscard)
    #define NODISCARD [[nodiscard]]
#else
    #define NODISCARD
#endif

#if __has_cpp_attribute(deprecated)
    #define DEPRECATED(msg) [[deprecated(msg)]]
#else
    #define DEPRECATED(msg)
#endif

NODISCARD int compute() { return 42; }

DEPRECATED("Use compute() instead")
int old_compute() { return 0; }

int main() {
    std::cout << "compute() = " << compute() << std::endl;
    return 0;
}
```

**Expected Output:**
```
compute() = 42
```

**Why this output:** On a compiler that supports `[[nodiscard]]` and `[[deprecated]]`, the macros expand to the attributes. On older compilers, they expand to nothing, and the code still compiles. This makes the code portable across compilers with varying attribute support.

---

### Real-World Cases with Explanation

**Case 1 — Standard library evolution:** Libraries use `__has_include(<optional>)` to use `std::optional` where available, falling back to a custom implementation otherwise. This is the standard migration pattern for C++17 features.

**Case 2 — Attribute portability:** A library defines `NODISCARD` as `[[nodiscard]]` only if the compiler supports it, otherwise as nothing. This allows the library to provide better diagnostics on modern compilers without breaking older ones.

**Case 3 — Feature-test macros:** Combined with `__cpp_*` macros, `__has_include` provides a complete feature-detection system. The WG21 feature-testing recommendations document (SD-6) specifies the exact macro names and values.

**Case 4 — Build system integration:** CMake and other build systems use `__has_include` in configure checks to determine which headers are available and set up the build accordingly.

---

### References Links

- Feature-test recommendations — cppreference.com - https://en.cppreference.com/w/cpp/feature_test
- Feature-testing preprocessor predicates for C++17 — N4535 - https://isocpp.open-std.org/JTC1/SC22/WG21/docs/papers/2015/n4535.html
- `__has_include` — Microsoft Learn - https://learn.microsoft.com/en-us/cpp/preprocessor/hash-if-hash-elif-hash-else-and-hash-endif-directives-c-cpp

---

## 6. Resource Embedding (`#embed`, C++26)

### Definitions

**Core Definition**
`#embed` is a preprocessor directive (introduced in C++26) that includes binary or textual resource files directly into the source code as a comma-separated list of integers.

**Technical Definition**
`#embed` is a preprocessor directive to include resources, where a resource is defined as a source of data accessible from the translation environment. The directive searches for a uniquely identified resource (using `<resource>`) or a named resource (using `"resource"`) and replaces itself with the contents of the resource as a comma-separated list of integers. Parameters such as `limit`, `prefix`, `suffix`, and `if_empty` can modify the embedding behavior. `__has_embed` checks whether a resource exists.

**Beginner-Friendly Explanation**
`#embed` lets you put a binary file — an image, a sound, a configuration file — directly into your program's source code without converting it to a C array manually. For example, `#embed "icon.png"` becomes a list of numbers representing the file's bytes. The compiler compiles those numbers into the program, so the file is available at runtime without needing to read it from disk.

---

### Purposes (all begin with "To")

- **To** embed binary resources (images, fonts, audio, certificates) directly into executables.
- **To** eliminate the need for external resource files at runtime.
- **To** replace error-prone manual byte-array conversion with a standard directive.
- **To** enable compile-time access to resource data for constexpr contexts.
- **To** support embedded systems where the filesystem may not be available.

---

### Syntax Rules and Structure

#### Complete General Syntaxes with Breakdowns

**Syntax 1 — Angle-bracket form**
```cpp
#embed <resource>
```
- Searches for a uniquely identified resource in implementation-defined locations.

**Syntax 2 — Quoted form**
```cpp
#embed "resource"
```
- Searches for a named resource in an implementation-defined manner.

**Syntax 3 — With parameters**
```cpp
#embed "word.txt" prefix(0x2C, 0x20) suffix(0x21) limit(5)
```
- `prefix(...)` — bytes to prepend.
- `suffix(...)` — bytes to append.
- `limit(N)` — maximum number of bytes to embed.
- `if_empty(...)` — bytes to use if the resource is empty.

**Syntax 4 — `__has_embed` check**
```cpp
#if __has_embed(<data.bin>)
    #embed <data.bin>
#else
    // fallback
#endif
```

#### Syntax Rules

- The resource name in the angle-bracket form is searched in implementation-defined places.
- The quoted form uses an implementation-defined search.
- If neither form matches after macro expansion, the behavior is undefined.
- Parameters are specified as preprocessing tokens after the resource name.
- `__has_embed` is part of the conditional inclusion subclause and can be used in `#if`.

#### Constraints and Limitations

- `#embed` is C++26; support is currently limited (GCC 15+ has support).
- The embedded data is always a comma-separated list of integers; it cannot be used directly as a string without additional processing.
- The resource search path is implementation-defined.
- `__has_include`, `__has_embed`, and `__has_cpp_attribute` shall not appear in `#embed` clauses (ill-formed).

---

### Multiple Annotated Complete Step-by-Step Code Examples and Their Expected Outputs

#### Example 1 — Embedding a Text File as a String

```cpp
// File: word.txt contains: world

#include <string>
#include <print>

int main() {
    // Step 1: Embed the file with prefix, suffix, and limit
    std::string s {
        #embed "word.txt" prefix(0x2C, 0x20) suffix(0x21) limit(5)
    };
    std::println("Hello{}", s);
    return 0;
}
```

**Expected Output:**
```
Hello, world!
```

**Why this output:** The file `word.txt` contains `world`. The `prefix(0x2C, 0x20)` adds `, ` (comma and space) before the content. The `suffix(0x21)` adds `!` after. The `limit(5)` limits the embed to 5 bytes. The result is `, world!`, which when concatenated with `"Hello"` produces `"Hello, world!"`. This example is from cppreference's `#embed` documentation.

---

#### Example 2 — Conditional Embedding with Fallback

```cpp
#include <iostream>

int main() {
#if __has_embed("config.json")
    const unsigned char config[] = {
        #embed "config.json"
    };
    std::cout << "Config loaded, " << sizeof(config) << " bytes" << std::endl;
#else
    std::cout << "Config not found, using defaults" << std::endl;
#endif
    return 0;
}
```

**Expected Output (with config.json):**
```
Config loaded, 256 bytes
```

**Expected Output (without config.json):**
```
Config not found, using defaults
```

**Why this output:** `__has_embed` checks whether the resource exists. If it does, `#embed` inserts the bytes as an initializer list for the `config` array. The `sizeof` operator then reports the number of bytes. If the resource is missing, the fallback code is compiled instead.

---

### Real-World Cases with Explanation

**Case 1 — Embedded GUI resources:** A GUI application embeds icons, fonts, and images directly into the executable, eliminating the need for separate resource files during deployment.

**Case 2 — Firmware with configuration data:** Embedded systems embed configuration files, calibration data, or device trees directly into firmware, where no filesystem is available at runtime.

**Case 3 — Cryptographic keys and certificates:** Applications embed public keys, root certificates, or trusted CA lists directly into the binary, avoiding the risk of runtime file tampering.

**Case 4 — WebAssembly and browser applications:** WebAssembly modules embed HTML templates, shaders, or asset manifests directly into the `.wasm` binary, reducing network requests.

---

### References Links

- Resource inclusion (`#embed`) — cppreference.com - https://en.cppreference.com/w/cpp/preprocessor/embed
- P1967R14 `#embed` — WG21 paper - https://wg21.link/P1967R14
- GCC `#embed` support — GCC Patch - https://gcc.gnu.org/pipermail/gcc-patches/2025-February/676352.html

---

## 7. Diagnostic Routing (`#error` and `#warning`)

### Definitions

**Core Definition**
`#error` causes the implementation to produce a diagnostic message and renders the program ill-formed. `#warning` produces a diagnostic message without affecting the validity of the program.

**Technical Definition**
A preprocessing directive of the form `#error pp-tokens` causes the implementation to produce a diagnostic message that includes the specified sequence of preprocessing tokens, and the `#error` directive renders the program ill-formed. The `#warning` directive (standardized in C++23) does the same but does not render the program ill-formed. Before C++23, `#warning` was provided by many compilers as a conforming extension.

**Beginner-Friendly Explanation**
`#error` is a way to stop compilation with a custom message. If you write `#error "This platform is not supported"`, the compiler will refuse to compile and show your message. `#warning` is like a sticky note: it shows a message but lets compilation continue. Both are useful for communicating build-time conditions to the developer.

---

### Purposes (all begin with "To")

- **To** stop compilation with a clear message when an unsupported configuration is detected.
- **To** warn developers about deprecated features, missing dependencies, or non-fatal issues.
- **To** enforce build-time contracts (e.g., required macros, minimum compiler versions).
- **To** provide actionable diagnostics when preprocessor conditions fail.
- **To** document assumptions about the build environment.

---

### Syntax Rules and Structure

#### Complete General Syntaxes with Breakdowns

**Syntax 1 — `#error`**
```cpp
#error "This platform is not supported"
```
- Produces an error diagnostic with the message and stops compilation.

**Syntax 2 — `#warning` (C++23)**
```cpp
#warning "This feature is deprecated"
```
- Produces a warning diagnostic with the message; compilation continues.

**Syntax 3 — Conditional diagnostics**
```cpp
#if !defined(REQUIRED_MACRO)
    #error "REQUIRED_MACRO must be defined"
#endif
```

#### Syntax Rules

- The `#error` and `#warning` directives take an optional sequence of preprocessing tokens.
- The implementation must produce a diagnostic message that includes the tokens.
- `#error` renders the program ill-formed; `#warning` does not.
- The identifiers following the directive must be valid preprocessing tokens.
- These directives can be used inside conditional blocks to provide context-specific diagnostics.

#### Constraints and Limitations

- `#warning` is C++23; before that, it was a widely supported extension but not standard.
- The exact format of the diagnostic message is implementation-defined.
- `#error` stops compilation immediately after producing the diagnostic.
- Multiple `#error` directives may produce multiple diagnostics (the implementation is not required to stop at the first one).

---

### Multiple Annotated Complete Step-by-Step Code Examples and Their Expected Outputs

#### Example 1 — Enforcing a Minimum C++ Standard

```cpp
#include <iostream>

#if __cplusplus < 201703L
    #error "This program requires C++17 or later"
#endif

int main() {
    std::cout << "C++17 or later detected" << std::endl;
    return 0;
}
```

**Compile with C++14:**
```bash
g++ -std=c++14 main.cpp -o app
```

**Expected Error:**
```
main.cpp:4:6: error: #error "This program requires C++17 or later"
    4 |     #error "This program requires C++17 or later"
      |      ^~~~~
```

**Compile with C++17:**
```bash
g++ -std=c++17 main.cpp -o app
./app
```

**Expected Output:**
```
C++17 or later detected
```

**Why this output:** The `__cplusplus` macro expands to `201402L` under C++14, which is less than `201703L`, so the `#error` is triggered. Under C++17, `__cplusplus` is `201703L`, so the error is skipped and compilation succeeds.

---

#### Example 2 — Warning About Deprecated Configuration

```cpp
#include <iostream>

#if defined(USE_OLD_API)
    #warning "USE_OLD_API is deprecated; migrate to USE_NEW_API"
#endif

int main() {
#ifdef USE_OLD_API
    std::cout << "Using old API" << std::endl;
#else
    std::cout << "Using new API" << std::endl;
#endif
    return 0;
}
```

**Compile with old API flag:**
```bash
g++ -std=c++23 -DUSE_OLD_API main.cpp -o app
./app
```

**Expected Output (with warning):**
```
main.cpp:4:6: warning: USE_OLD_API is deprecated; migrate to USE_NEW_API
    4 |     #warning "USE_OLD_API is deprecated; migrate to USE_NEW_API"
      |      ^~~~~~~
Using old API
```

**Why this output:** The `#warning` directive produces a diagnostic message but does not stop compilation. The program continues and prints "Using old API" because the `#ifdef USE_OLD_API` branch is compiled. This gives developers a gentle nudge without breaking their build.

---

### Real-World Cases with Explanation

**Case 1 — Compiler version checks:** A library requires a minimum compiler version and uses `#error` if the detected version is too old, with a message telling the user to upgrade.

**Case 2 — Platform support:** A platform-specific module uses `#error "This module requires Windows"` if compiled on a non-Windows platform, providing a clear message instead of a cascade of syntax errors.

**Case 3 — Deprecation warnings:** A library that has changed its API uses `#warning` to inform users that an old macro is deprecated and will be removed in a future version.

**Case 4 — Build configuration validation:** A build system checks that required macros (e.g., `BUILD_VERSION`) are defined, using `#error` if they are missing, preventing cryptic downstream errors.

---

### References Links

- Diagnostic directives — cppreference.com - https://en.cppreference.com/w/cpp/preprocessor/error
- Support for `#warning` — P2437R1 - https://rap.no/JTC1/SC22/WG21/docs/papers/2022/p2437r1.pdf
- `#error` directive — Microsoft Learn - https://learn.microsoft.com/en-us/cpp/preprocessor/hash-error-directive-c-cpp

---

## Summary Table of Core Concepts

| Concept | Key Directive(s) | Primary Use | Key Constraint |
|---|---|---|---|
| File Inclusion | `#include <...>` / `#include "..."` | Share declarations across TUs | Search order implementation-defined |
| String Token Replacement | `#define`, `#undef`, `#`, `##` | Symbolic constants, code generation | Text-based; no scope or type safety |
| Logical Flow | `#if`, `#elif`, `#else`, `#endif` | Platform/version-specific compilation | Expression must be integer constant |
| Existence Evaluation | `#ifdef`, `#ifndef`, `defined()` | Include guards, feature flags | Only checks macro existence, not value |
| Modern Introspection | `__has_include`, `__has_cpp_attribute` | Portable feature detection | C++17 required; implementation-dependent |
| Resource Embedding | `#embed` | Binary asset injection | C++26 required; limited compiler support |
| Diagnostic Routing | `#error`, `#warning` | Build-time validation, deprecation notices | `#warning` is C++23; `#error` stops compilation |

---

## References (Consolidated)

- Preprocessor — cppreference.com - https://en.cppreference.com/w/cpp/preprocessor
- Conditional inclusion — cppreference.com - https://en.cppreference.com/w/cpp/preprocessor/conditional
- Source file inclusion — cppreference.com - https://en.cppreference.com/w/cpp/preprocessor/include
- Resource inclusion (`#embed`) — cppreference.com - https://en.cppreference.com/w/cpp/preprocessor/embed
- Feature-test recommendations — cppreference.com - https://en.cppreference.com/w/cpp/feature_test
- `#include` directive — Microsoft Learn - https://learn.microsoft.com/en-us/cpp/preprocessor/hash-include-directive-c-cpp
- `#define` directive — Microsoft Learn - https://learn.microsoft.com/en-us/cpp/preprocessor/hash-define-directive-c-cpp
- `#if`, `#elif`, `#else`, `#endif` directives — Microsoft Learn - https://learn.microsoft.com/en-us/cpp/preprocessor/hash-if-hash-elif-hash-else-and-hash-endif-directives-c-cpp
- `#ifdef` and `#ifndef` directives — Microsoft Learn - https://learn.microsoft.com/en-us/cpp/preprocessor/hash-ifdef-and-hash-ifndef-directives-c-cpp
- Preprocessor directives (C/C++) — Microsoft Learn - https://learn.microsoft.com/en-us/cpp/preprocessor/preprocessor-directives
- The C Preprocessor — GCC Documentation - https://gcc.gnu.org/onlinedocs/gcc-8.2.0/cpp/The-preprocessing-language.html
- Feature-testing preprocessor predicates for C++17 — N4535 - https://isocpp.open-std.org/JTC1/SC22/WG21/docs/papers/2015/n4535.html
- Support for `#warning` — P2437R1 - https://rap.no/JTC1/SC22/WG21/docs/papers/2022/p2437r1.pdf
- `#embed` synchronization — N3696 - https://rap.no/JTC1/SC22/WG21/docs/papers/2022/p2348r3.pdf