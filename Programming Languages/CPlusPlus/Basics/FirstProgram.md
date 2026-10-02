# First C++ Program

Writing your first C++ program introduces the fundamental building blocks of the language: program structure, headers, the `main()` function, statements, blocks, output with `std::cout`, namespaces, and return values. This guide also traces what happens from source code to a running executable.

---

## 1. The Classic First Program

```cpp
#include <iostream>

int main() {
    std::cout << "Hello, World!" << std::endl;
    return 0;
}
```

This is the canonical "Hello, World!" program in C++. Every part of it has a purpose.

---

## 2. Program Structure

A C++ program is organized into several structural components:

```
┌─────────────────────────────────────┐
│  Preprocessor directives (#include) │
├─────────────────────────────────────┤
│  Declarations (functions, classes)  │
├─────────────────────────────────────┤
│  main() function (entry point)      │
├─────────────────────────────────────┤
│  Other function definitions         │
└─────────────────────────────────────┘
```

### General Structure

```cpp
// 1. Preprocessor directives
#include <iostream>
#include <vector>

// 2. Declarations (optional)
int add(int a, int b);

// 3. Main function (required)
int main() {
    // statements
    return 0;
}

// 4. Other function definitions (optional)
int add(int a, int b) {
    return a + b;
}
```

| Component | Purpose | Required? |
|---|---|---|
| **Preprocessor directives** | Include headers, define macros | Optional |
| **Declarations** | Announce functions, classes, variables | Optional |
| **`main()`** | Program entry point | **Required** |
| **Other definitions** | Implement functionality | Optional |

### Key Rules

- **Exactly one `main()`** per program
- **Case-sensitive** — `Main`, `MAIN`, and `main` are different
- **Statements end with `;`**
- **Whitespace is insignificant** (except inside string literals)
- **C++ is free-form** — indentation is for humans, not the compiler

---

## 3. Header Inclusion

### 3.1 The `#include` Directive

`#include` is a **preprocessor directive** that inserts the contents of another file into the current file **before compilation**.

```cpp
#include <iostream>
```

### 3.2 Angle Brackets vs. Quotes

| Syntax | Search Path | Use For |
|---|---|---|
| `#include <header>` | System/standard directories | Standard library headers |
| `#include "header"` | Current directory first, then system | Your own headers |

**Examples:**

```cpp
#include <iostream>      // standard library
#include <vector>        // standard library
#include "my_header.h"   // your project header
#include "../utils/helper.h"  // relative path
```

### 3.3 What `<iostream>` Provides

`<iostream>` declares the **standard input/output stream objects**:

| Object | Purpose | Default Device |
|---|---|---|
| `std::cin` | Standard input | Keyboard |
| `std::cout` | Standard output | Console |
| `std::cerr` | Standard error (unbuffered) | Console |
| `std::clog` | Standard log (buffered) | Console |

> "The `<iostream>` library allows for input and output in C++ programs".

### 3.4 Common Standard Headers

| Header | Purpose |
|---|---|
| `<iostream>` | Input/output streams |
| `<iomanip>` | I/O manipulators (formatting) |
| `<string>` | `std::string` |
| `<vector>` | Dynamic array |
| `<array>` | Fixed-size array |
| `<map>` | Associative container |
| `<algorithm>` | Algorithms (sort, find, etc.) |
| `<cmath>` | Math functions |
| `<cstdlib>` | General utilities |
| `<fstream>` | File streams |

### 3.5 Modern Alternative: Modules (C++20)

C++20 introduced **modules** as a replacement for `#include`:

```cpp
import <iostream>;    // module-based (C++20+)
```

Modules are faster to compile and avoid macro pollution, but compiler support is still maturing.

### 3.6 Include Guards

For your own headers, use **include guards** or `#pragma once`:

```cpp
// my_header.h
#ifndef MY_HEADER_H
#define MY_HEADER_H

// declarations here

#endif
```

Or:

```cpp
// my_header.h
#pragma once

// declarations here
```

---

## 4. `main()`

### 4.1 The Entry Point

`main()` is the **entry point** of every C++ program. When the operating system launches your executable, it calls `main()`.

> "The `main` function is the entry point of the program".

### 4.2 Signatures

C++ defines two standard signatures for `main()`:

```cpp
int main() {
    // ...
    return 0;
}
```

```cpp
int main(int argc, char* argv[]) {
    // argc = argument count
    // argv = argument vector (array of C-strings)
    return 0;
}
```

| Signature | Purpose |
|---|---|
| `int main()` | No command-line arguments |
| `int main(int argc, char* argv[])` | Accepts command-line arguments |

### 4.3 Command-Line Arguments

```cpp
#include <iostream>

int main(int argc, char* argv[]) {
    std::cout << "Argument count: " << argc << "\n";
    for (int i = 0; i < argc; ++i) {
        std::cout << "argv[" << i << "] = " << argv[i] << "\n";
    }
    return 0;
}
```

**Running:**

```bash
./app hello world
```

**Output:**

```
Argument count: 3
argv[0] = ./app
argv[1] = hello
argv[2] = world
```

- `argv[0]` is the program name
- `argc` is always at least 1

### 4.4 Rules for `main()`

| Rule | Description |
|---|---|
| **Exactly one** | Only one `main()` per program |
| **Return type is `int`** | Cannot be `void` (in standard C++) |
| **Not callable recursively** | The standard prohibits calling `main()` |
| **Implicit `return 0`** | If omitted, `main()` returns 0 implicitly |
| **Not overloadable** | Cannot have multiple versions |

### 4.5 Implicit Return

If `main()` reaches the closing brace without a `return`, it implicitly returns `0`:

```cpp
int main() {
    std::cout << "Hello\n";
    // implicit return 0
}
```

This is a special exception — **only `main()`** has this behavior.

---

## 5. Statements

A **statement** is a complete instruction that performs an action.

### 5.1 Statement Types

| Type | Example |
|---|---|
| **Declaration** | `int x = 5;` |
| **Expression** | `x = x + 1;` |
| **Control flow** | `if (x > 0) { ... }` |
| **Loop** | `for (int i = 0; i < 10; ++i) { ... }` |
| **Return** | `return 0;` |
| **Compound** | `{ int a = 1; int b = 2; }` |
| **Null** | `;` (does nothing) |

### 5.2 Statement Termination

Every statement ends with a **semicolon** (`;`):

```cpp
int x = 5;                    // declaration
std::cout << "Hello";         // expression
return 0;                     // return
```

### 5.3 Common Statement Errors

**Missing semicolon:**

```cpp
int x = 5    // ✗ error: expected ';'
int y = 10;
```

**Extra semicolon:**

```cpp
int x = 5;;  // ✓ legal but unnecessary
```

**Semicolon after a block:**

```cpp
if (x > 0) {
    std::cout << "positive";
};  // ✗ unnecessary semicolon — creates an empty statement
```

---

## 6. Blocks

A **block** (or **compound statement**) is a sequence of statements enclosed in **curly braces** `{ }`.

### 6.1 Syntax

```cpp
{
    statement;
    statement;
    // ...
}
```

### 6.2 Example

```cpp
int main() {
    // start of main's block

    int x = 5;

    {
        // nested block
        int y = 10;
        std::cout << x + y << "\n";
    }
    // y is out of scope here

    return 0;
}
```

### 6.3 Scope

A block defines a **scope** — variables declared inside are visible only within that block.

```cpp
int main() {
    int x = 5;

    {
        int y = 10;      // y is local to this block
        std::cout << x;  // ✓ x is visible
    }

    // std::cout << y;   // ✗ error: y not declared
    return 0;
}
```

### 6.4 Block Rules

| Rule | Description |
|---|---|
| Enclosed in `{ }` | Required |
| No semicolon after `}` | `}` does not need `;` (except after class/struct definitions) |
| Can be nested | Blocks within blocks |
| Defines scope | Variables are local to the block |
| Can appear anywhere a statement can | In functions, loops, conditionals |

### 6.5 Empty Block

```cpp
{
    // valid — does nothing
}
```

---

## 7. `std::cout`

### 7.1 Standard Output Stream

`std::cout` is the **standard output stream** — typically the console or terminal.

> "The `cout` object is used to print text to the standard output (the console). The `<<` operator is used to send data to `cout`".

### 7.2 The `<<` Operator

`<<` is the **stream insertion operator**. It sends data to the stream.

```cpp
std::cout << "Hello, World!";
```

### 7.3 Chaining

`<<` returns the stream, so it can be **chained**:

```cpp
std::cout << "Hello, " << "World" << "!" << std::endl;
```

This is equivalent to:

```cpp
((((std::cout << "Hello, ") << "World") << "!") << std::endl);
```

### 7.4 Common Output Operations

```cpp
#include <iostream>

int main() {
    int age = 30;
    double pi = 3.14159;
    std::string name = "Alice";

    // Multiple values
    std::cout << "Name: " << name << ", Age: " << age << "\n";

    // Newline options
    std::cout << "Line 1\n";           // C-style escape
    std::cout << "Line 2" << std::endl; // C++ manipulator (flushes)

    // Formatting numbers
    std::cout << "Pi: " << pi << "\n";

    return 0;
}
```

### 7.5 `\n` vs. `std::endl`

| Aspect | `\n` | `std::endl` |
|---|---|---|
| **Meaning** | Newline character | Newline + flush |
| **Flush** | No | Yes |
| **Performance** | Faster | Slower (flush overhead) |
| **Header** | Any | `<iostream>` |
| **Recommended** | Yes, for most cases | Only when flushing is needed |

> "`std::endl` flushes the stream — use `\n` unless you specifically need a flush".

### 7.6 Multiple Insertions

```cpp
std::cout << 1 << 2 << 3 << "\n";
// Output: 123 (not 6)
```

Each `<<` inserts its operand as text.

### 7.7 Output of Different Types

```cpp
std::cout << 42 << "\n";           // int
std::cout << 3.14 << "\n";         // double
std::cout << 'A' << "\n";          // char
std::cout << "text" << "\n";       // string literal
std::cout << true << "\n";         // bool (prints 1)
std::cout << std::boolalpha << true << "\n";  // prints "true"
```

---

## 8. Namespaces

### 8.1 What Is a Namespace?

A **namespace** is a **named scope** that groups related identifiers to prevent **name collisions**.

> "A namespace is a declarative region that provides a scope to the identifiers (names of types, function, variables, etc.) inside it".

### 8.2 The `std` Namespace

Everything in the C++ standard library lives in the **`std`** namespace:

- `std::cout`, `std::cin`, `std::cerr`
- `std::string`, `std::vector`, `std::map`
- `std::sort`, `std::find`

### 8.3 Accessing Namespaced Identifiers

**Option 1 — Fully qualified (recommended):**

```cpp
std::cout << "Hello\n";
std::string name = "Alice";
std::vector<int> v = {1, 2, 3};
```

**Option 2 — Using declaration:**

```cpp
using std::cout;
using std::string;

cout << "Hello\n";
string name = "Alice";
```

**Option 3 — Using directive (discouraged):**

```cpp
using namespace std;

cout << "Hello\n";  // works, but risky
string name = "Alice";
```

### 8.4 Why `using namespace std;` Is Discouraged

> "When you use `using namespace std;`, you're pulling **every** name from `std` into the global namespace. This can cause conflicts".

```cpp
#include <iostream>
using namespace std;

int count = 10;   // your variable

int main() {
    // std::count algorithm from <algorithm> could conflict
    // with your "count" variable
    return 0;
}
```

| Risk | Explanation |
|---|---|
| **Name collisions** | Your names may clash with `std` names |
| **Ambiguity** | The compiler may not know which name you mean |
| **Readability** | Readers can't tell what's from `std` |
| **Header pollution** | If used in a header, it affects every file that includes it |

**Recommendation:** Prefer **fully qualified** names (`std::cout`) or **specific `using` declarations** (`using std::cout;`).

### 8.5 Defining Your Own Namespace

```cpp
namespace math {
    int add(int a, int b) {
        return a + b;
    }

    const double PI = 3.14159;
}

int main() {
    std::cout << math::add(2, 3) << "\n";      // 5
    std::cout << math::PI << "\n";             // 3.14159
    return 0;
}
```

### 8.6 Nested Namespaces (C++17)

```cpp
namespace company::project::module {
    void doWork() { }
}

// Usage
company::project::module::doWork();
```

### 8.7 Namespace Aliases

```cpp
namespace fs = std::filesystem;
fs::path p = "/tmp/file.txt";
```

### 8.8 The Global Namespace

Identifiers not inside any namespace belong to the **global namespace**:

```cpp
int globalVar = 42;          // global namespace

namespace app {
    int localVar = 10;       // app namespace
}

// Accessing global from a namespace:
namespace app {
    int getGlobal() {
        return ::globalVar;   // :: refers to global namespace
    }
}
```

---

## 9. Return Values

### 9.1 `main()` Return Value

`main()` returns an `int` to the **operating system** — the **exit status**.

```cpp
int main() {
    return 0;    // success
}
```

### 9.2 Exit Codes

| Code | Meaning |
|---|---|
| `0` | Success (conventional) |
| Non-zero | Failure (varies by program) |
| `EXIT_SUCCESS` | Success (`<cstdlib>`, typically 0) |
| `EXIT_FAILURE` | Failure (`<cstdlib>`, typically 1) |

```cpp
#include <cstdlib>

int main() {
    if (/* error */) {
        return EXIT_FAILURE;
    }
    return EXIT_SUCCESS;
}
```

### 9.3 Checking Exit Codes

**Linux/macOS:**

```bash
./app
echo $?      # prints exit code
```

**Windows:**

```cmd
app.exe
echo %ERRORLEVEL%
```

### 9.4 Using Exit Codes in Scripts

```bash
#!/bin/bash
./app
if [ $? -eq 0 ]; then
    echo "Success"
else
    echo "Failure"
fi
```

### 9.5 Returning from Other Functions

Any function with a non-`void` return type must return a value:

```cpp
int add(int a, int b) {
    return a + b;
}

double square(double x) {
    return x * x;
}

void printMessage() {
    std::cout << "Hello\n";
    // no return needed
}
```

### 9.6 `return` in `void` Functions

```cpp
void process(int x) {
    if (x < 0) {
        return;   // early exit
    }
    std::cout << x << "\n";
}
```

---

## 10. Compilation

### 10.1 The Compilation Process

Compilation translates C++ source code into an executable:

```
source.cpp → [Preprocessor] → [Compiler] → [Assembler] → source.o
                                                              ↓
                                                          [Linker] → executable
```

### 10.2 Step-by-Step

| Stage | Tool | Input | Output |
|---|---|---|---|
| **Preprocessing** | `cpp` | `.cpp` | Expanded source |
| **Compilation** | `g++` / `clang++` | Expanded source | Assembly (`.s`) |
| **Assembly** | `as` | Assembly | Object code (`.o`) |
| **Linking** | `ld` | Object files + libraries | Executable |

### 10.3 Basic Compilation

**GCC/G++:**

```bash
g++ -o hello hello.cpp
```

**Clang:**

```bash
clang++ -o hello hello.cpp
```

**MSVC:**

```cmd
cl hello.cpp /Fe:hello.exe
```

### 10.4 With Common Flags

```bash
g++ -std=c++20 -Wall -Wextra -O2 -o hello hello.cpp
```

| Flag | Purpose |
|---|---|
| `-std=c++20` | Use C++20 standard |
| `-Wall` | Enable common warnings |
| `-Wextra` | Enable additional warnings |
| `-O2` | Optimize for speed |
| `-o hello` | Output file name |

### 10.5 Compile Only (No Link)

```bash
g++ -c hello.cpp -o hello.o
```

Produces an object file without linking.

### 10.6 Preprocess Only

```bash
g++ -E hello.cpp > hello.i
```

Produces the fully preprocessed source.

### 10.7 Generate Assembly

```bash
g++ -S hello.cpp -o hello.s
```

Produces assembly code.

### 10.8 Verbose Output

```bash
g++ -v -o hello hello.cpp
```

Shows all stages and commands executed.

---

## 11. Linking

### 11.1 What Linking Does

**Linking** combines object files and libraries into a single executable, resolving **symbol references**.

> "The linker takes the object files generated by the compiler, and combines them together ... Linker makes sure that every required function is defined in some object file".

### 11.2 Linking in Action

For a simple program:

```bash
g++ -o hello hello.cpp
```

The compiler:

1. Compiles `hello.cpp` → `hello.o`
2. Links `hello.o` with the **C++ standard library** (which provides `std::cout`, etc.)
3. Produces the executable `hello`

### 11.3 Manual Linking

```bash
# Compile
g++ -c hello.cpp -o hello.o

# Link
g++ hello.o -o hello
```

### 11.4 Linking Multiple Files

```bash
g++ -c main.cpp -o main.o
g++ -c utils.cpp -o utils.o
g++ main.o utils.o -o app
```

### 11.5 Linking with Libraries

```bash
g++ main.cpp -lm -o app           # link math library
g++ main.cpp -L./lib -lmyLib -o app  # link custom library
```

| Flag | Purpose |
|---|---|
| `-lm` | Link `libm` (math library) |
| `-L/path` | Add library search directory |
| `-lname` | Link `libname.a` or `libname.so` |

### 11.6 The Standard Library

The C++ standard library is linked **automatically**:

- **libstdc++** (GCC)
- **libc++** (Clang)
- **MSVC runtime** (MSVC)

You rarely need to link it explicitly.

### 11.7 Static vs. Dynamic Linking

| Aspect | Static | Dynamic |
|---|---|---|
| **Extension** | `.a`, `.lib` | `.so`, `.dll`, `.dylib` |
| **When linked** | Build time | Runtime |
| **Executable size** | Larger | Smaller |
| **Portability** | High (self-contained) | Requires libraries present |
| **Updates** | Requires recompilation | Library can be updated independently |

**Static linking:**

```bash
g++ main.cpp -static -o app
```

**Dynamic linking:**

```bash
g++ main.cpp -o app              # dynamic by default
```

### 11.8 Common Linker Errors

| Error | Cause | Fix |
|---|---|---|
| **undefined reference** | Function declared but not defined | Link correct object/library |
| **duplicate symbol** | Same symbol defined twice | Remove duplicate; use `inline` or `static` |
| **cannot find -lfoo** | Library not in search path | Add `-L/path` |
| **undefined symbol at runtime** | Dynamic library missing | Install library; check `LD_LIBRARY_PATH` |

### 11.9 Complete Build Example

**Files:**

```cpp
// main.cpp
#include <iostream>
#include "math_utils.h"

int main() {
    std::cout << "2 + 3 = " << add(2, 3) << "\n";
    return 0;
}
```

```cpp
// math_utils.h
#pragma once
int add(int a, int b);
```

```cpp
// math_utils.cpp
#include "math_utils.h"
int add(int a, int b) { return a + b; }
```

**Compile and link:**

```bash
g++ -c main.cpp -o main.o
g++ -c math_utils.cpp -o math_utils.o
g++ main.o math_utils.o -o app
./app
```

**One-liner:**

```bash
g++ -std=c++20 -Wall -Wextra main.cpp math_utils.cpp -o app
```

---

## 12. Execution

### 12.1 Running the Program

**Linux/macOS:**

```bash
./hello
```

**Windows:**

```cmd
hello.exe
```

Or if the current directory is not in `PATH`:

```cmd
.\hello.exe
```

### 12.2 What Happens at Runtime

```
1. OS loads the executable into memory
2. OS sets up the runtime environment (stack, heap, argv, envp)
3. C++ runtime initializes (static constructors, iostream)
4. OS calls main(argc, argv)
5. main() executes statements
6. main() returns an int (exit code)
7. C++ runtime cleans up (static destructors, flush streams)
8. OS reclaims resources
9. Exit code is available to the shell
```

### 12.3 Program Startup Details

Before `main()` runs:

- **Static storage** is initialized
- **Global objects** are constructed
- **`std::ios_base::Init`** initializes `std::cout`, `std::cin`, etc.
- **Command-line arguments** are parsed into `argc`/`argv`

After `main()` returns:

- **Static destructors** run
- **Streams are flushed**
- **Resources are released**

### 12.4 Checking the Exit Code

```bash
./hello
echo $?      # Linux/macOS
```

```cmd
hello.exe
echo %ERRORLEVEL%    # Windows
```

### 12.5 Full Workflow Example

**Source (`hello.cpp`):**

```cpp
#include <iostream>

int main() {
    std::cout << "Hello, World!" << std::endl;
    return 0;
}
```

**Compile:**

```bash
g++ -std=c++20 -Wall -Wextra -o hello hello.cpp
```

**Execute:**

```bash
./hello
```

**Output:**

```
Hello, World!
```

**Check exit code:**

```bash
echo $?
# 0
```

---

## 13. Complete Annotated Example

```cpp
// ─── Preprocessor directive ─────────────────────────────
#include <iostream>   // include the I/O stream library

// ─── Main function — entry point ────────────────────────
int main(int argc, char* argv[]) {

    // ─── Statement: variable declaration ────────────────
    int count = 3;

    // ─── Statement: output ──────────────────────────────
    std::cout << "Hello from C++!\n";        // \n newline
    std::cout << "Count: " << count << "\n"; // stream insertion

    // ─── Block: nested scope ────────────────────────────
    {
        int inner = 42;
        std::cout << "Inner: " << inner << "\n";
    }
    // inner is out of scope here

    // ─── Return value: exit code ────────────────────────
    return 0;   // 0 = success
}
```

**Compile and run:**

```bash
g++ -std=c++20 -Wall -Wextra -o demo demo.cpp
./demo
```

**Output:**

```
Hello from C++!
Count: 3
Inner: 42
```

**Check exit code:**

```bash
echo $?
# 0
```

---

## Summary Table

| Concept | Definition | Example |
|---|---|---|
| **Program structure** | Directives + declarations + `main()` + definitions | `#include`, `int main()` |
| **Header inclusion** | `#include` inserts file contents | `#include <iostream>` |
| **`main()`** | Program entry point | `int main() { ... }` |
| **Statements** | Complete instructions | `int x = 5;` |
| **Blocks** | `{ }` grouping with scope | `{ int y = 10; }` |
| **`std::cout`** | Standard output stream | `std::cout << "Hi";` |
| **Namespaces** | Named scope for identifiers | `std::`, `namespace app { }` |
| **Return values** | Exit status from `main()` | `return 0;` |
| **Compilation** | Source → object code | `g++ -c main.cpp` |
| **Linking** | Object files → executable | `g++ main.o -o app` |
| **Execution** | OS runs the executable | `./app` |

---

## Key Takeaways

1. A C++ program has a **structure**: preprocessor directives, declarations, `main()`, and other definitions.
2. **`#include <iostream>`** brings in the standard I/O library; use angle brackets for standard headers, quotes for your own.
3. **`main()`** is the required entry point — it returns an `int` exit code to the OS.
4. **Statements** end with `;`; **blocks** use `{ }` to group statements and define scope.
5. **`std::cout`** is the standard output stream; `<<` is the stream insertion operator.
6. Use **`\n`** instead of **`std::endl`** unless you need to flush the stream.
7. **Namespaces** prevent name collisions — `std` is the standard library's namespace.
8. Prefer **fully qualified names** (`std::cout`) over `using namespace std;`.
9. **`return 0;`** signals success; non-zero signals failure.
10. **Compilation** translates source → object code; **linking** combines object files + libraries → executable.
11. **Execution** loads the binary, initializes the runtime, calls `main()`, and returns the exit code.
12. **One `main()` per program** — the standard prohibits overloading or calling it recursively.

---

Would you like me to continue with the next topic — **C++ Data Types**, **C++ Variables and Constants**, or **C++ Operators**? I can format the next section in the same style.