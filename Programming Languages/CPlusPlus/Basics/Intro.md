# Introduction to C++

C++ is one of the most influential and widely used programming languages in modern software development. It powers operating systems, game engines, browsers, financial systems, embedded devices, and high-performance computing infrastructure. Understanding what C++ is, where it came from, and how it has evolved provides essential context for learning the language.

---

## 1. C++ Definition

### 1.1 General-Purpose Programming Language

C++ is a **general-purpose programming language** with a **bias towards systems programming**. It is defined by an **ISO standard** and offers **stability over decades** with a large and lively user community.

> "C++ is a general-purpose programming language with a bias towards systems programming that — is a better C — supports data abstraction (e.g., classes) — supports object-oriented programming (e.g., inheritance) — supports generic programming (e.g., reusable generic containers and algorithms) — supports functional programming (e.g., template metaprogramming, lambda functions, `constexpr`)"

### 1.2 Multi-Paradigm Programming Language

C++ is a **multi-paradigm language** — it supports multiple programming styles without enforcing any single one:

| Paradigm | Description | C++ Features |
|---|---|---|
| **Procedural** | Step-by-step instructions | Functions, loops, conditionals |
| **Object-Oriented** | Objects encapsulating data and behavior | Classes, inheritance, polymorphism |
| **Generic** | Reusable code parameterized by type | Templates, STL |
| **Functional** | Functions as first-class values | Lambdas, `std::function`, `constexpr` |
| **Data Abstraction** | Hiding implementation details | Classes, access specifiers |

> "C++ supports the object paradigm but does not enforce it; it is a multi-paradigm language. Programmers who have primarily used procedural paradigm decomposition techniques can migrate to the language at a comfortable pace and still achieve many benefits of the language."

### 1.3 Compiled Language

C++ is a **compiled language** — source code is translated by a **compiler** into machine code (a binary executable) before execution.

| Stage | Description |
|---|---|
| **Preprocessing** | Handles `#include`, `#define`, macros |
| **Compilation** | Translates source to object code |
| **Linking** | Combines object files and libraries into an executable |
| **Execution** | The binary runs directly on the CPU |

**Implications:**

- **Performance** — no interpreter or virtual machine overhead at runtime
- **Platform-specific binaries** — must recompile for different architectures
- **Compile-time errors** — many bugs are caught before execution
- **Build systems required** — CMake, Make, Ninja, Bazel manage the build process

### 1.4 Statically Typed Language

C++ is **statically typed** — types are checked at **compile time**, not at runtime.

> "C++ is defined as a statically typed, free-form language that supports procedural programming, data abstraction, and object-orientation."

| Aspect | Static Typing | Dynamic Typing (e.g., Python) |
|---|---|---|
| **Type checking** | Compile time | Runtime |
| **Type declarations** | Required (mostly) | Optional |
| **Performance** | Faster (no runtime checks) | Slower (type checks at runtime) |
| **Safety** | Catches type errors early | Errors appear at runtime |
| **Flexibility** | Less flexible | More flexible |

**Example:**

```cpp
int x = 42;        // x is an int
std::string s = "hello";  // s is a std::string

// x = s;          // Compile error — type mismatch
```

**Type deduction with `auto`:**

```cpp
auto x = 42;           // int
auto y = 3.14;         // double
auto z = std::string("hello");  // std::string
```

### 1.5 The Zero-Overhead Principle

A guiding design principle of C++ is the **zero-overhead principle**:

> "What you don't use, you don't pay for (in time or space) and further: What you do use, you couldn't hand code any better."

This means:

- No feature should make existing code larger or slower
- No feature should generate code worse than hand-written equivalent
- Abstractions compile down to efficient machine code

---

## 2. Historical Development

### 2.1 C++ Origins

C++ was created by **Bjarne Stroustrup**, a Danish computer scientist, at **AT&T Bell Laboratories** in Murray Hill, New Jersey.

> "Bjarne Stroustrup is the creator of C++, the object-oriented extension to the C language. He is a researcher at the AT&T Bell Laboratories Computing Science Research Center where, in 1980, he began the development of the C++ extensions that add data abstraction, class hierarchies, and function and operator overloading to C."

**The problem Stroustrup faced:**

> "I had written some simulations of distributed computer systems and was thinking about doing more of them. At the same time I was thinking about the problem of splitting Unix up to run on many CPUs. In both cases I decided that the problem was building greater modularity to get fire walls in place, and I couldn't do that with C. I had experience with Simula, writing rather complex simulations, so I knew the basic techniques of object-oriented programming and how it applied. To solve the problem I added classes to C that were very much like Simula classes."

**Timeline of C++ origins:**

| Year | Milestone |
|---|---|
| **1979** | Stroustrup begins work on "C with Classes" at AT&T Bell Labs |
| **1980** | Development of C++ extensions begins |
| **1983** | The language is officially named **C++** (the `++` increment operator) |
| **1985** | First commercial release from AT&T; first edition of *The C++ Programming Language* |
| **1989** | ANSI committee X3J16 meets for the first time |
| **1998** | **ISO C++ standard** (ISO/IEC 14882:1998) published |

> "Developed by Bjarne Stroustrup, C++ has become one of the most popular programming languages in the world. Originally ... Stroustrup began with the idea that object oriented programming would be an important addition to C, and created C with Classes. In 1983, Stroustrup's contributions officially became known as C++, its name stemming from C and adding the ++ (increment) operator."

### 2.2 Relationship to C

C++ is **largely a superset of C**:

> "Except for minor details, C++ is a superset of the C programming language."

| Aspect | C | C++ |
|---|---|---|
| **Origin** | Dennis Ritchie, 1972 | Bjarne Stroustrup, 1979+ |
| **Paradigm** | Procedural | Multi-paradigm |
| **Abstraction** | Functions, structs | Classes, templates, inheritance |
| **Memory** | `malloc`/`free` | `new`/`delete`, smart pointers |
| **Standard library** | Minimal (libc) | Rich (STL, iostream, algorithms) |
| **Compatibility** | C code mostly compiles in C++ | C++ code does not compile in C |

**Key differences:**

- C++ adds **classes, inheritance, polymorphism, templates, exceptions, namespaces, references, operator overloading**
- C++ provides **RAII** (Resource Acquisition Is Initialization) — deterministic cleanup
- C++ has a **richer standard library** — containers, algorithms, smart pointers, threading

### 2.3 Evolution of the C++ Standard

> "It is defined by an ISO standard, offers stability over decades."

| Standard | Year | Key Features |
|---|---|---|
| **C++98** | 1998 | First ISO standard; STL; templates; exceptions |
| **C++03** | 2003 | Minor revision; bug fixes |
| **C++11** | 2011 | **Major overhaul**: `auto`, lambdas, smart pointers, move semantics, threading |
| **C++14** | 2014 | Generic lambdas, `std::make_unique`, variable templates |
| **C++17** | 2017 | `std::optional`, `std::variant`, `std::filesystem`, structured bindings, `if constexpr` |
| **C++20** | 2020 | **Concepts, Modules, Coroutines, Ranges** (the "Big Four") |
| **C++23** | 2023 | `std::print`, `std::expected`, `std::mdspan`, modules for standard library |
| **C++26** | 2026 (in progress) | Reflection, Contracts, memory safety improvements |

### 2.4 Major Language Generations

C++ development is often described in **generations**:

| Generation | Era | Characteristics |
|---|---|---|
| **C with Classes** | 1979–1983 | Pre-standard; Simula-inspired classes |
| **Classic C++** | 1985–1998 | ARM (Annotated Reference Manual); pre-standard |
| **Standard C++** | 1998–2011 | ISO standardization; STL |
| **Modern C++** | 2011–2020 | C++11/14/17/20; auto, lambdas, move |
| **Contemporary C++** | 2023+ | C++23/26; modules, ranges, reflection |

---

## 3. Modern C++

Modern C++ refers to the language as it has evolved since **C++11** — a period of rapid, transformative change. The standard is now updated **every three years**, with the committee maintaining an aggressive release cadence.

### 3.1 C++11 — The Game Changer

C++11 was the **largest overhaul since the language's inception**, transforming how C++ is written.

| Feature | Description |
|---|---|
| **`auto`** | Type deduction for variables |
| **Lambda expressions** | Anonymous functions defined inline |
| **Smart pointers** | `unique_ptr`, `shared_ptr`, `weak_ptr` |
| **Move semantics** | Efficient transfer of resources |
| **Rvalue references** | `T&&` for perfect forwarding |
| **Range-based for loops** | `for (auto& x : container)` |
| **`nullptr`** | Type-safe null pointer |
| **Strongly-typed enums** | `enum class` |
| **`constexpr`** | Compile-time evaluation |
| **Threading library** | `std::thread`, `std::mutex`, `std::async` |
| **Uniform initialization** | `{}` syntax for all types |

> "C++11 — Avanços gigantes: Ponteiros Inteligentes (Smart Pointers), lambdas, `std::move`, ..."

**Example — lambda and `auto`:**

```cpp
#include <algorithm>
#include <vector>

std::vector<int> v = {5, 2, 8, 1, 9};

// Lambda with auto
auto is_even = [](int n) { return n % 2 == 0; };

// Range-based for
for (const auto& x : v) {
    std::cout << x << " ";
}

// Sort with lambda
std::sort(v.begin(), v.end(), [](int a, int b) { return a > b; });
```

### 3.2 C++14 — Refinements

| Feature | Description |
|---|---|
| **Generic lambdas** | Lambdas with `auto` parameters |
| **`std::make_unique`** | Factory for `unique_ptr` |
| **Variable templates** | Templates for variables |
| **Relaxed `constexpr`** | More operations allowed in `constexpr` |
| **Return type deduction** | Functions can deduce return type |

**Example — generic lambda:**

```cpp
auto add = [](auto a, auto b) { return a + b; };
auto i = add(1, 2);        // int
auto d = add(1.5, 2.5);    // double
```

### 3.3 C++17 — Practical Improvements

| Feature | Description |
|---|---|
| **`std::optional`** | Optional values without pointers |
| **`std::variant`** | Type-safe union |
| **`std::filesystem`** | File system operations |
| **Structured bindings** | `auto [a, b] = pair;` |
| **`if constexpr`** | Compile-time branching |
| **`std::string_view`** | Non-owning string reference |
| **Fold expressions** | Concise variadic operations |
| **`inline` variables** | Header-only globals |

**Example — structured bindings:**

```cpp
std::map<std::string, int> scores = {{"Alice", 95}, {"Bob", 87}};

for (const auto& [name, score] : scores) {
    std::cout << name << ": " << score << "\n";
}
```

### 3.4 C++20 — The "Big Four"

C++20 introduced **four major features** that reshape how C++ is written:

> "C++20 is a major version after C++17, featuring major features (concepts, modules, coroutines, and ranges)."

| Feature | Description |
|---|---|
| **Concepts** | Precise specification of template requirements |
| **Modules** | Replace `#include` with `import` |
| **Coroutines** | Suspendable/resumable functions |
| **Ranges** | Composable algorithms on ranges |

**Concepts — constraining templates:**

```cpp
#include <concepts>

template <std::integral T>
T add(T a, T b) {
    return a + b;
}

// add(1, 2);        // OK
// add(1.5, 2.5);    // Error: double is not integral
```

**Ranges — composable algorithms:**

```cpp
#include <ranges>
#include <vector>

std::vector<int> v = {1, 2, 3, 4, 5, 6};

auto even = v | std::views::filter([](int n) { return n % 2 == 0; })
              | std::views::transform([](int n) { return n * n; });

for (int n : even) {
    std::cout << n << " ";  // 4 16 36
}
```

**Modules — replacing headers:**

```cpp
// math.cppm
export module math;
export int add(int a, int b) { return a + b; }

// main.cpp
import math;
int main() { return add(1, 2); }
```

### 3.5 C++23 — Standard Library Enhancements

C++23 focused on **library improvements** and **smaller language additions**:

| Feature | Description |
|---|---|
| **`std::print` / `std::println`** | Python-like formatted output |
| **`std::expected`** | Error handling without exceptions |
| **`std::mdspan`** | Multi-dimensional array views |
| **`std::flat_map` / `std::flat_set`** | Cache-friendly containers |
| **`import std;`** | Modules for the entire standard library |
| **`std::stacktrace`** | Stack trace support |
| **Deducing `this`** | Explicit object parameter |

**Example — `std::print`:**

```cpp
#include <print>

int main() {
    std::println("Hello, {}! Score: {}", "World", 42);
}
```

> "C++23 adds `std::print` and `std::println` (header `<print>`) as direct replacements for the `std::cout <<` ..."

> "C++23 (padrão atual) - Biblioteca padrão com CXX Modules: 'import std;' - Impressão com `std::print`"

### 3.6 C++26 — The Next Frontier

C++26 is **feature-complete** and undergoing final review:

> "C++26 is feature-complete, and C++29 is underway."

| Feature | Description |
|---|---|
| **Reflection** | Compile-time introspection and code generation |
| **Contracts** | Preconditions, postconditions, assertions |
| **Memory safety** | Improvements without code rewrites |
| **New async model** | Modern asynchronous programming |

> "Several C++26 features have been implemented: P2996R13, Reflection; P2900R14, Contracts."

> "Reflection — the most eagerly anticipated feature of C++26 — profoundly reshapes how we reason about C++ and what we can accomplish with it."

> "Contract assertions introduce scalable and configurable correctness checks for identifying program defects and making C++ code safer."

**Example — contracts (proposed):**

```cpp
int divide(int a, int b)
    pre(b != 0)           // precondition
    post(result != 0)     // postcondition
{
    return a / b;
}
```

### 3.7 Newer Standard Evolution

| Standard | Status | Focus |
|---|---|---|
| **C++23** | **Current standard** (ISO/IEC 14882:2024) | Library improvements, `std::print` |
| **C++26** | Feature-complete | Reflection, contracts, safety |
| **C++29** | In development | Complete undefined behavior catalog, SIMD, pattern matching |

> "The current ISO C++ standard is **C++23**, formally known as ISO International Standard ISO/IEC 14882:2024(E) – Programming Language C++."

**Adoption reality:**

> "C++17 remains the most common standard (43%), but C++20 (34%) and C++23 (21%) are gaining traction."

---

## 4. C++ Application Domains

C++ is used in **essentially every application area**:

> "C++ is used in essentially every application areas, incl. scientific calculations, compilers, operating systems, device drivers, games, distributed systems infrastructure, animation, telecommunications, embedded systems applications (e.g. mars rover autonomous driving), aeronautics software, CAD/CAM systems, ordinary business applications, graphics, e-commerce sites, and large web applications (such as airline reservation)."

### 4.1 Systems Software

C++ provides **extensive control over system resources and memory**, making it ideal for developing operating systems and low-level system utilities.

| System | Description |
|---|---|
| **Windows** | Large portions written in C++ |
| **Linux** | Kernel in C, user-space tools in C++ |
| **macOS** | Core frameworks in C++ |
| **Device drivers** | Hardware interfacing, USB drivers |
| **Compilers** | GCC, Clang/LLVM, MSVC |

> "System Programming — It means, operating systems, device drivers, hardware interfacing, USB drivers."

### 4.2 Game Development

C++ is a **staple in the game development industry** due to its performance and efficiency.

| Area | Description |
|---|---|
| **Game engines** | Unreal Engine, Unity (core) |
| **AAA games** | Most large-scale titles |
| **3D engines** | Rendering pipelines |
| **AR/VR** | Real-time graphics |
| **Physics engines** | Real-time simulation |

> "3D game engines and AAA (triple-A) games."

> "C++ is a staple in the game development industry due to its performance and efficiency."

### 4.3 Embedded Systems

C++ was designed for **embedded systems programming** and is used in **resource-constrained environments**:

> "C++ was designed to be a systems programming language and has been used for embedded systems programming and other resource-constrained types of programming since the earliest days."

| Area | Examples |
|---|---|
| **Microcontrollers** | Arduino, ESP32, STM32 |
| **DSPs** | Digital Signal Processors |
| **Automotive** | ECU firmware, autonomous driving |
| **Aerospace** | Mars rover autonomous driving |
| **IoT devices** | Smart home, wearables |

> "Embedded Systems — DSP (Digital Signal Processors), Microcontrollers, Embedded Processors and so on."

### 4.4 High-Performance Computing (HPC)

C++ is used extensively in **scientific computing**, **simulation**, and **performance-critical applications**:

| Area | Examples |
|---|---|
| **Scientific simulations** | Physics, chemistry, climate |
| **Numerical computing** | Linear algebra, FFT |
| **High-energy physics** | CERN, particle accelerators |
| **Weather prediction** | Atmospheric models |
| **Computational fluid dynamics** | Aerospace, automotive |

> "High performance numerical simulation (Number crunching), High Performance Computing (HPC), Physics, High Energy Physics - See CERN."

### 4.5 Financial Software

C++ is used in **trading systems**, **risk engines**, and **market data infrastructure**:

| Area | Description |
|---|---|
| **Algorithmic trading** | Low-latency execution |
| **Risk management** | Real-time calculations |
| **Market data** | High-throughput feeds |
| **Banking systems** | Core transaction processing |

> "Low Latency/Games/Embedded/Financial Trading" — SG14 working group

### 4.6 Desktop Applications

| Application | Description |
|---|---|
| **Adobe Photoshop** | Image editing |
| **Adobe Illustrator** | Vector graphics |
| **Microsoft Office** | Word, Excel, PowerPoint |
| **Web browsers** | Chrome (Blink engine), Firefox (Gecko) |
| **CAD software** | AutoCAD, SolidWorks |

### 4.7 Compilers

Many major compilers and toolchains are written in C++:

| Compiler | Description |
|---|---|
| **GCC** | GNU Compiler Collection (C++ core) |
| **Clang/LLVM** | Modern compiler infrastructure |
| **MSVC** | Microsoft Visual C++ |
| **Java HotSpot** | JVM JIT compiler |

### 4.8 Networking

C++ powers **network infrastructure**, **servers**, and **protocol implementations**:

| Area | Description |
|---|---|
| **Web servers** | High-performance HTTP servers |
| **DNS servers** | Domain name resolution |
| **Load balancers** | Traffic distribution |
| **VPNs** | Encryption and tunneling |
| **Telecommunications** | Network equipment |

> "Telecommunications, distributed systems infrastructure."

### 4.9 Robotics

| Area | Description |
|---|---|
| **Robot operating systems** | ROS (Robot Operating System) |
| **Autonomous vehicles** | Perception, planning, control |
| **Industrial robots** | Manufacturing automation |
| **Drones** | UAV flight control |

> "Hard real time: audio, signal processing, radar, some control systems, UAVs - Unmanned Aerial Vehicles, Robots …"

### 4.10 Summary — Application Domains

| Domain | Key Characteristics | Why C++ |
|---|---|---|
| **Systems software** | Low-level control | Direct hardware access |
| **Game development** | Real-time performance | Zero-overhead abstractions |
| **Embedded systems** | Resource-constrained | Minimal runtime, small footprint |
| **HPC** | Numerical computation | Performance, parallelism |
| **Financial software** | Low latency | Deterministic performance |
| **Desktop applications** | Rich UI | Performance + ecosystem |
| **Compilers** | Complex optimization | Performance + abstraction |
| **Networking** | High throughput | Efficiency, control |
| **Robotics** | Real-time control | Predictability, hardware access |

---

## Summary Table

| Topic | Key Points |
|---|---|
| **Definition** | General-purpose, multi-paradigm, compiled, statically typed, ISO-standard |
| **Origin** | Bjarne Stroustrup, AT&T Bell Labs, 1979 |
| **Relationship to C** | Superset; adds classes, templates, exceptions, STL |
| **C++98** | First ISO standard; STL, templates, exceptions |
| **C++11** | `auto`, lambdas, smart pointers, move semantics, threading |
| **C++14** | Generic lambdas, `make_unique`, variable templates |
| **C++17** | `optional`, `variant`, `filesystem`, structured bindings |
| **C++20** | **Concepts, Modules, Coroutines, Ranges** (Big Four) |
| **C++23** | `std::print`, `std::expected`, `std::mdspan`, `import std;` |
| **C++26** | Reflection, Contracts, memory safety (feature-complete) |
| **Application domains** | Systems, games, embedded, HPC, finance, desktop, compilers, networking, robotics |
| **Zero-overhead principle** | What you don't use, you don't pay for |

---

## Key Takeaways

1. **C++** is a **general-purpose, multi-paradigm, compiled, statically typed** programming language with a bias toward systems programming.
2. It was created by **Bjarne Stroustrup** at **AT&T Bell Labs** in **1979** as "C with Classes," driven by the need for modularity and Simula-like classes in C.
3. C++ is **largely a superset of C** — C code mostly compiles as C++, but not vice versa.
4. The language is defined by an **ISO standard**, first published in **1998**, with updates every **three years**.
5. **C++11** was a transformative overhaul — `auto`, lambdas, smart pointers, move semantics, and threading.
6. **C++20** introduced the **"Big Four"**: Concepts, Modules, Coroutines, and Ranges.
7. **C++23** focused on library improvements — `std::print`, `std::expected`, and `import std;`.
8. **C++26** is **feature-complete**, bringing **Reflection** and **Contracts** — two of the most anticipated features in C++ history.
9. The **zero-overhead principle** — "what you don't use, you don't pay for" — guides the language's design.
10. C++ is used in **virtually every domain**: operating systems, games, embedded systems, HPC, finance, compilers, networking, and robotics.

---

Would you like me to continue with the next topic — **C++ Setup and Compilation**, **C++ Basic Syntax**, or **C++ Data Types**? I can format the next section in the same style.