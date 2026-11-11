# Function Overloading — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
Function overloading is a C++ feature that allows multiple functions in the same scope to share the same name, provided their parameter lists differ, so that the compiler can select the most appropriate function based on the arguments supplied at the call site.

**Technical Definition**
When a name refers to more than one entity, it is said to be *overloaded*, and the compiler must determine which overload to call through a process called **overload resolution**. Overloading is permitted when the functions differ in the number or types of their parameters (and, for member functions, in cv-/ref-qualifiers); functions that differ only in return type cannot be overloaded. The compiler’s name-mangling scheme encodes parameter types into the symbol name, making overloading possible at the linker level.

**Beginner-Friendly Explanation**
Function overloading means you can give several functions the **same name** as long as they take **different inputs**. For example, you can have a `print` function that prints an `int` and another `print` that prints a `std::string`. When you call `print(...)`, the compiler looks at what you passed and automatically picks the right version. It is like having one verb (“print”) that works naturally with many kinds of objects.

### Key Characteristics

- **Same name, different parameters**: Overloads must differ in their parameter-type-list (or, for member functions, cv-/ref-qualifiers).
- **Return type alone is insufficient**: Functions that differ only in return type cannot be overloaded.
- **Compile-time resolution**: The compiler selects the best overload at compile time based on the arguments’ types and value categories.
- **Enabled by name mangling**: The compiler encodes parameter types into symbol names to distinguish overloads at the linker level.
- **Interaction with templates**: Non-template functions are preferred over template specializations when all else is equal.
- **Potential for ambiguity**: If the compiler cannot determine a unique best match, the call is ill-formed.

### Prerequisites

- Understanding of function declarations and definitions.
- Familiarity with C++ types and implicit conversions (promotions, standard conversions, user-defined conversions).
- Knowledge of value categories (lvalues, rvalues) and reference binding.
- Basic awareness of namespaces and scope.

### Related Programming Areas

- **Overload resolution** – the algorithm that selects the best function.
- **Name mangling and linkage** – how overloads are represented in object files.
- **Function templates** – template specialization and overloading interact.
- **Operator overloading** – a special case of function overloading.
- **Default arguments** – an alternative to some overload sets.

### Core Concepts / Features

1. Function Overloading
2. Overloading Rules
3. Overload Resolution
4. Ambiguity
5. Name Mangling
6. `extern "C"`
7. Overloading Best Practices

---

## 1. Function Overloading

### Definitions

**Core Definition**
Function overloading is the declaration of multiple functions with the same name but different parameter lists within the same scope.

**Technical Definition**
Two functions are considered overloads if they are declared in the same scope, have the same name, and their parameter-type-lists differ (after array-to-pointer and function-to-pointer adjustments, and considering cv-qualification for reference parameters). The return type is **not** part of the function signature for overloading purposes, so functions that differ only in return type are redeclarations and cause a compilation error.

**Beginner-Friendly Explanation**
Overloading lets you reuse a familiar name for operations that are conceptually similar but operate on different kinds of data. The compiler figures out which version you mean by looking at the arguments you provide.

### Purposes

- To provide a natural, uniform interface for operations on different types.
- To reduce the number of distinct function names a programmer must remember.
- To support generic-style programming without templates in simple cases.
- To improve code readability and expressiveness.

### Syntax Rules and Structure

**General Syntax**
```cpp
return_type function_name(type1);
return_type function_name(type2);
// ... additional overloads
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `function_name` | The shared identifier. |
| `type1`, `type2`, … | Distinct parameter-type-lists. |

**Syntax Rules**
- Overloads must be declared in the **same scope**.
- The parameter lists must differ in the **number** or **types** of parameters.
- For member functions, cv-qualifiers (`const`, `volatile`) and ref-qualifiers (`&`, `&&`) can distinguish overloads.
- Return type is **not** part of the signature and cannot distinguish overloads.
- Default arguments do **not** create distinct overloads.

**Constraints and Limitations**
- Functions that differ only in return type cannot be overloaded.
- Functions that differ only in `noexcept` cannot be overloaded (since C++17).
- Overloading is resolved at compile time; there is no runtime dispatch (for that, use `virtual` functions).

### Annotated Code Examples

**Example 1: Overloading on parameter type**

```cpp
#include <iostream>

void print(int x)    { std::cout << "int: " << x << '\n'; }
void print(double x) { std::cout << "double: " << x << '\n'; }
void print(const char* s) { std::cout << "string: " << s << '\n'; }

int main() {
    print(42);          // selects print(int)
    print(3.14);        // selects print(double)
    print("hello");     // selects print(const char*)
}
```

**Expected Output**
```
int: 42
double: 3.14
string: hello
```

**Why this output?**  
The argument type at each call site determines which overload is selected. `42` is an `int`, `3.14` is a `double`, and `"hello"` decays to `const char*`.

**Example 2: Overloading on number of parameters**

```cpp
#include <iostream>

int sum(int a)             { return a; }
int sum(int a, int b)      { return a + b; }
int sum(int a, int b, int c) { return a + b + c; }

int main() {
    std::cout << sum(1)       << '\n';
    std::cout << sum(1, 2)    << '\n';
    std::cout << sum(1, 2, 3) << '\n';
}
```

**Expected Output**
```
1
3
6
```

**Why this output?**  
Different numbers of arguments select different overloads. The compiler matches the argument count to the parameter count.

**Example 3: Member function overloading on `const`**

```cpp
#include <iostream>

class Container {
    int value = 42;
public:
    int get()       { std::cout << "non-const\n"; return value; }
    int get() const { std::cout << "const\n";     return value; }
};

int main() {
    Container c;
    const Container cc;
    c.get();    // non-const overload
    cc.get();   // const overload
}
```

**Expected Output**
```
non-const
const
```

**Why this output?**  
The cv-qualification of the implicit object argument selects between the `const` and non-`const` member function overloads.

### Real-World Cases

- **Standard library** – `std::abs` overloads for `int`, `long`, `float`, `double`.
- **String conversion** – `std::to_string` overloads for numeric types.
- **Stream insertion** – `operator<<` overloads for many types.
- **Container access** – `operator[]` overloads for `const` and non-`const`.

---

## 2. Overloading Rules

### Definitions

**Core Definition**
Overloading rules define the conditions under which two functions with the same name can coexist as overloads and which declarations are considered distinct.

**Technical Definition**
Two function declarations in the same scope are overloads if they have the same name and their parameter-type-lists differ. The parameter-type-list is the list of parameter types after array-to-pointer and function-to-pointer adjustments. For non-static member functions, the cv-qualification and ref-qualification of the implicit object parameter also participate. Return type, `noexcept` specification (since C++17), and default arguments do **not** participate in distinguishing overloads.

**Beginner-Friendly Explanation**
The compiler needs a clear way to tell overloaded functions apart. The rule is simple: **the inputs must be different**. Whether a function is `const`, whether it can throw, or what it returns does not matter for overloading (with the exception that `const` member functions can be overloaded against non-`const` ones).

### Purposes

- To define precisely when overloading is permitted.
- To prevent ambiguous or ill-formed declarations.
- To guide the design of overload sets.
- To explain why certain combinations fail to compile.

### Syntax Rules and Structure

**Permitted Distinguishers**

| Distinguisher | Example | Allowed? |
|---------------|---------|----------|
| Parameter type | `f(int)` vs. `f(double)` | Yes |
| Number of parameters | `f(int)` vs. `f(int, int)` | Yes |
| cv-qualifier (member) | `T::f()` vs. `T::f() const` | Yes |
| ref-qualifier (member) | `T::f() &` vs. `T::f() &&` | Yes |
| Return type | `int f()` vs. `double f()` | **No** |
| `noexcept` (since C++17) | `f() noexcept` vs. `f()` | **No** |
| Default arguments | `f(int = 0)` vs. `f(int)` | **No** (redeclaration) |

**Syntax Rules**
- Overloads must be in the same scope.
- Parameter names are irrelevant.
- `typedef`s that alias the same type do not create distinct overloads.
- Array and function parameters decay to pointers, so `f(int[])` and `f(int*)` are the **same** overload.

**Constraints and Limitations**
- Overloading on return type alone is ill-formed.
- Overloading on `noexcept` alone is ill-formed (since C++17).
- Overloading on `const` is only possible for member functions (the implicit object parameter).

### Annotated Code Examples

**Example 1: Allowed vs. disallowed overloading**

```cpp
#include <iostream>

// Allowed: different parameter types
void f(int)    { std::cout << "f(int)\n"; }
void f(double) { std::cout << "f(double)\n"; }

// Disallowed: different return types only
// int g() { return 1; }
// double g() { return 2.0; }   // error: redefinition

// Disallowed: different noexcept only
// void h() noexcept {}
// void h() {}                   // error: redefinition

int main() {
    f(1);      // f(int)
    f(1.5);    // f(double)
}
```

**Expected Output**
```
f(int)
f(double)
```

**Why this output?**  
`f` overloads differ in parameter type, so they are valid overloads. The calls select the appropriate version. The commented-out declarations would be errors because return type and `noexcept` alone do not distinguish overloads.

**Example 2: Array-to-pointer decay**

```cpp
void process(int* p) { /* ... */ }
void process(int arr[]) { /* ... */ }   // redeclaration of the same function!

// The two declarations above are the same function: int[] decays to int*.
```

**Expected Output**  
(Compilation error: redefinition of `process`.)

**Why this error?**  
Array parameters undergo array-to-pointer adjustment, so `int[]` and `int*` produce the same parameter-type-list.

### Real-World Cases

- **API design** – ensuring overload sets are unambiguous.
- **Code review** – spotting accidental redeclarations.
- **Template metaprogramming** – understanding when templates and non-templates compete.

---

## 3. Overload Resolution

### Definitions

**Core Definition**
Overload resolution is the compiler process that selects the single best-matching function from a set of overloaded candidates for a given function call.

**Technical Definition**
Overload resolution proceeds in three main steps: (1) building the **candidate function set** from name lookup and template argument deduction; (2) trimming the set to **viable functions** whose parameters can be initialized from the arguments; and (3) analyzing the viable set to determine the **single best viable function** by ranking the implicit conversion sequences for each argument. If no unique best match exists, the call is ambiguous.

**Beginner-Friendly Explanation**
When you call an overloaded function, the compiler acts like a matchmaker. First it gathers all functions with that name. Then it discards the ones that cannot accept your arguments. Finally, it picks the one that requires the “least work” to convert your arguments — the closest match. If two functions are equally good, the compiler gives up and reports an ambiguity error.

### Purposes

- To select the most appropriate function for each call.
- To ensure type-safe and predictable behavior.
- To support generic and intuitive APIs.
- To detect ambiguous or ill-formed calls at compile time.

### Syntax Rules and Structure

**The Overload Resolution Algorithm (Simplified)**

| Step | Description |
|------|-------------|
| 1. Candidate set | All functions with the name found by lookup (including ADL and template deduction). |
| 2. Viable set | Candidates whose parameters can be initialized from the arguments (number and types compatible). |
| 3. Best viable | Ranking of implicit conversion sequences; the candidate that is “not worse” for all arguments and “better” for at least one is selected. |

**Ranking of Implicit Conversion Sequences (Best to Worst)**

| Rank | Category | Examples |
|------|----------|----------|
| 1 | Exact match | No conversion; lvalue-to-rvalue, qualification conversion |
| 2 | Promotion | `short` → `int`, `float` → `double` |
| 3 | Standard conversion | `int` → `double`, `double` → `int`, pointer conversions |
| 4 | User-defined conversion | Converting constructors, conversion operators |

**Tie-Breakers (when conversion ranks are equal)**
- Non-template function is preferred over template specialization.
- More specialized template is preferred over less specialized.
- For reference binding, rvalue references are preferred over lvalue references for rvalues (and vice versa for lvalues).

**Constraints and Limitations**
- If two candidates require indistinguishable conversion sequences, the call is **ambiguous**.
- User-defined conversions are not considered if a standard conversion suffices (for non-class types).
- The ranking is per-argument; a candidate that is better for one argument but worse for another may still win if it is “not worse” overall and better for at least one.

### Annotated Code Examples

**Example 1: Exact match vs. conversion**

```cpp
#include <iostream>

void f(int x)    { std::cout << "f(int)\n"; }
void f(double x) { std::cout << "f(double)\n"; }

int main() {
    f(42);      // exact match: f(int)
    f(3.14);    // exact match: f(double)
    f(3.14f);   // float → double (promotion? standard conversion) → f(double)
}
```

**Expected Output**
```
f(int)
f(double)
f(double)
```

**Why this output?**  
`42` exactly matches `f(int)`. `3.14` exactly matches `f(double)`. `3.14f` is a `float`, which undergoes a floating-point promotion to `double`, selecting `f(double)`.

**Example 2: Promotion vs. standard conversion**

```cpp
#include <iostream>

void g(int x)    { std::cout << "g(int)\n"; }
void g(long x)   { std::cout << "g(long)\n"; }

int main() {
    short s = 10;
    g(s);       // short → int (promotion) beats short → long (conversion)
}
```

**Expected Output**
```
g(int)
```

**Why this output?**  
`short` → `int` is a **promotion**, which ranks higher than `short` → `long` (a standard conversion). The promotion candidate wins.

**Example 3: Non-template preferred over template**

```cpp
#include <iostream>

void h(int x) { std::cout << "non-template\n"; }

template <typename T>
void h(T x) { std::cout << "template\n"; }

int main() {
    h(42);      // non-template preferred
}
```

**Expected Output**
```
non-template
```

**Why this output?**  
Both candidates are viable and require an exact match, but the non-template function is preferred over the template specialization as a tie-breaker.

### Real-World Cases

- **Standard library algorithms** – overload resolution selects between `std::sort` overloads (with and without comparators).
- **Stream operators** – `operator<<` for many types relies on overload resolution.
- **Constructor selection** – `std::vector` constructors are chosen by overload resolution.

---

## 4. Ambiguity

### Definitions

**Core Definition**
Ambiguity occurs when overload resolution cannot determine a single best viable function because two or more candidates are equally good matches for the arguments.

**Technical Definition**
A function call is ambiguous if, after building the viable set and ranking implicit conversion sequences, there is no candidate that is “not worse” for all arguments and “better” for at least one. In that case, the program is ill-formed and the compiler must issue a diagnostic.

**Beginner-Friendly Explanation**
Ambiguity is when the compiler is “torn” between two overloads — neither is clearly better than the other. It cannot decide, so it reports an error and refuses to compile the call.

### Purposes

- To ensure that every call has a unique, well-defined meaning.
- To alert the programmer to ill-formed overload sets.
- To motivate clearer interface design (e.g., avoiding too many implicit conversions).

### Common Causes of Ambiguity

| Cause | Example |
|-------|---------|
| Equal-rank conversions | `f(long)` vs. `f(float)` called with `0` (ambiguous) |
| Multiple user-defined conversions | Two classes with conversion operators to `int` |
| Default arguments creating multiple viable candidates | `f(int, int = 0)` vs. `f(int)` called with one argument |
| Templates vs. non-templates with indistinguishable signatures | Rare, but possible with certain specializations |

### Syntax Rules and Structure (Diagnosis and Resolution)

**Resolution Strategies**

1. **Make one overload clearly better** by adding an exact-match overload.
2. **Use explicit casts** at the call site to select a specific overload.
3. **Avoid relying on multiple implicit conversions** in API design.
4. **Prefer default arguments** when the alternatives are of the same types (Core Guideline F.51).

**Constraints and Limitations**
- Ambiguity is a compile-time error; there is no runtime fallback.
- Some ambiguities are caused by implementation-defined conversion rankings (e.g., `long` vs. `long long` on some platforms).

### Annotated Code Examples

**Example 1: Ambiguous call**

```cpp
#include <iostream>

void f(long x)  { std::cout << "f(long)\n"; }
void f(float x) { std::cout << "f(float)\n"; }

int main() {
    f(0);       // error: ambiguous — int → long and int → float are both standard conversions
}
```

**Expected Output**  
(Compilation error: call of overloaded `f(int)` is ambiguous.)

**Why this error?**  
`0` is an `int`. Both `int` → `long` and `int` → `float` are standard conversions of equal rank (neither is a promotion). The compiler cannot choose.

**Example 2: Resolving ambiguity with a cast**

```cpp
#include <iostream>

void f(long x)  { std::cout << "f(long)\n"; }
void f(float x) { std::cout << "f(float)\n"; }

int main() {
    f(static_cast<long>(0));    // unambiguous: selects f(long)
}
```

**Expected Output**
```
f(long)
```

**Why this output?**  
The explicit cast makes the argument type exactly `long`, so `f(long)` is an exact match and no ambiguity arises.

**Example 3: Ambiguity from default arguments**

```cpp
void g(int a)          { /* ... */ }
void g(int a, int b = 0) { /* ... */ }

int main() {
    g(5);   // error: ambiguous — both g(int) and g(int, int=0) are viable
}
```

**Expected Output**  
(Compilation error: ambiguous.)

**Why this error?**  
Default arguments are applied during overload resolution, making both overloads viable for a single-argument call. Since neither is better, the call is ambiguous.

### Real-World Cases

- **API pitfalls** – libraries that provide too many implicit-conversion-friendly overloads can cause ambiguous calls.
- **Numeric types** – mixing `int`, `long`, `float`, `double` overloads often leads to ambiguity.
- **User-defined conversions** – types with multiple conversion operators can create ambiguity.

---

## 5. Name Mangling

### Definitions

**Core Definition**
Name mangling (also called name decoration) is the compiler’s process of encoding a function’s name and parameter types into a unique symbol name used by the linker.

**Technical Definition**
C++ compilers mangle function names to encode parameter types, namespaces, templates, and other information into the symbol name. This is what makes function overloading possible: two overloads of `f(int)` and `f(double)` produce different mangled symbols (e.g., `_Z1fi` and `_Z1fd` under the Itanium ABI). Name mangling is not standardized; different compilers and platforms use different schemes (the Itanium C++ ABI is the most widely used).

**Beginner-Friendly Explanation**
When you write `void f(int)` and `void f(double)`, the linker needs to tell them apart. Since the linker only sees plain names, the compiler **mangles** the names — it encodes the parameter types into a longer, unique symbol like `_Z1fi` for `f(int)` and `_Z1fd` for `f(double)`. This is how overloading works under the hood.

### Purposes

- To enable function overloading at the linker level.
- To encode namespace and class scope into symbol names.
- To support template instantiation symbols.
- To prevent clashes between functions with the same name in different scopes.

### Syntax Rules and Structure

Name mangling is an **implementation detail**; there is no standard syntax. The **Itanium C++ ABI** is the de facto standard for GCC, Clang, and many other compilers.

**Example (Itanium ABI)**

| Source declaration | Mangled name (example) |
|-------------------|------------------------|
| `void f(int)` | `_Z1fi` |
| `void f(double)` | `_Z1fd` |
| `int ns::g(char)` | `_ZN2ns1gEc` |
| `template<typename T> void h(T)` (T = int) | `_Z1hIiEvT_` |

**Component Breakdown (Itanium ABI, simplified)**

| Prefix | Meaning |
|--------|---------|
| `_Z` | Marks a mangled name. |
| `1f` | Function name `f` (length-prefixed). |
| `i` | Parameter type `int`. |
| `d` | Parameter type `double`. |
| `N ... E` | Nested name (namespace/class). |

**Syntax Rules (for programmers)**
- Name mangling is **not** something you write manually; it is generated by the compiler.
- You can inspect mangled names using tools like `nm`, `objdump`, or `c++filt`.
- Mangled names differ between compilers and platforms; they are **not portable**.

**Constraints and Limitations**
- Name mangling is implementation-defined; relying on specific mangled names is non-portable.
- `extern "C"` disables mangling (see next section).
- Some platforms (e.g., Windows/MSVC) use different mangling schemes.

### Annotated Code Examples

**Example 1: Observing mangled names (conceptual)**

```cpp
// math.cpp
int add(int a, int b)      { return a + b; }
double add(double a, double b) { return a + b; }
```

When compiled with GCC/Clang (Itanium ABI), the symbols in the object file might look like:
```
_Z3addii    // add(int, int)
_Z3adddd    // add(double, double)
```

**Expected Output (symbol table excerpt)**
```
0000000000000000 T _Z3addii
0000000000000014 T _Z3adddd
```

**Why this output?**  
The compiler mangles `add(int, int)` as `_Z3addii` and `add(double, double)` as `_Z3adddd`, encoding the parameter types so the linker can distinguish them.

**Example 2: Inspecting with `c++filt`**

```bash
$ echo "_Z3addii" | c++filt
add(int, int)
$ echo "_Z3adddd" | c++filt
add(double, double)
```

**Expected Output**  
(Demangled names as shown.)

**Why this output?**  
`c++filt` reverses the mangling, showing the original source-level signatures.

### Real-World Cases

- **Debugging linker errors** – “undefined reference to `_Z3addii`” means the definition for `add(int, int)` is missing.
- **ABI compatibility** – libraries compiled with different compilers may have incompatible mangled names.
- **Symbol versioning** – shared libraries use mangled names for versioned symbols.

---

## 6. `extern "C"`

### Definitions

**Core Definition**
`extern "C"` is a language-linkage specification that instructs the C++ compiler to use C linkage for the declared functions, which **disables name mangling** and uses the C calling convention.

**Technical Definition**
The `extern "C"` linkage specification applies C language linkage to function types and function names with external linkage. This means the function’s symbol name is **not mangled**, making it compatible with C code (or any code expecting unmangled symbols). Because names are not mangled, `extern "C"` functions **cannot be overloaded** (all overloads would produce the same symbol).

**Beginner-Friendly Explanation**
C++ normally mangles function names to support overloading. But C does not mangle names. `extern "C"` tells the C++ compiler: “Treat this function like a C function — do not mangle its name, and use the C calling convention.” This is how you make C++ functions callable from C, or call C functions from C++.

### Purposes

- To enable interoperability between C++ and C code.
- To provide unmangled symbols for dynamic linking (e.g., `dlsym`).
- To call C library functions from C++.
- To define C-callable callbacks in C++.

### Syntax Rules and Structure

**General Syntax**
```cpp
extern "C" return_type function_name(parameters);
extern "C" {
    // multiple declarations/definitions
}
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `extern "C"` | Linkage specification for C language linkage. |
| Function declaration/definition | The function to receive C linkage. |

**Syntax Rules**
- `extern "C"` can be applied to a single declaration or to a block of declarations.
- It affects both the function **type** (calling convention) and the function **name** (mangling).
- `extern "C"` functions cannot be overloaded; all declarations with the same name must refer to the same function.
- The linkage specification can be nested.

**Constraints and Limitations**
- Overloading is not possible for `extern "C"` functions.
- `extern "C"` does not change the C++ type system; it only affects linkage.
- On some platforms, the C and C++ calling conventions may differ (e.g., name decoration on Windows).
- `extern "C"` is not a type; it is a linkage specification.

### Annotated Code Examples

**Example 1: Calling a C function from C++**

```cpp
// C library header (e.g., math.h)
extern "C" {
    double sqrt(double x);   // C linkage: no mangling
}

#include <iostream>

int main() {
    std::cout << sqrt(16.0) << '\n';   // calls the C sqrt
}
```

**Expected Output**
```
4
```

**Why this output?**  
The `extern "C"` declaration tells the C++ compiler to look for the unmangled symbol `sqrt`, which the C library provides.

**Example 2: Defining a C-callable function in C++**

```cpp
// callback.cpp
#include <iostream>

extern "C" void handler(int code) {   // C linkage: callable from C
    std::cout << "Handler called with " << code << '\n';
}
```

```c
/* main.c */
extern void handler(int code);   /* C declaration */

int main(void) {
    handler(42);                  /* calls the C++ function */
    return 0;
}
```

**Expected Output**
```
Handler called with 42
```

**Why this output?**  
The C++ function is defined with `extern "C"`, so its symbol is unmangled (`handler`), allowing the C `main` to call it.

**Example 3: `extern "C"` prevents overloading**

```cpp
extern "C" void f(int);
// extern "C" void f(double);   // error: conflicting declaration (same unmangled name)
```

**Expected Output**  
(Compilation error.)

**Why this error?**  
Both declarations would produce the same unmangled symbol `f`, so the compiler cannot distinguish them. `extern "C"` disables the name mangling that makes overloading possible.

### Real-World Cases

- **C standard library** – `<cstdlib>`, `<cstdio>`, `<cmath>` declare C functions with `extern "C"`.
- **Dynamic loading** – `dlsym` requires unmangled symbol names.
- **Plugin systems** – C ABI is used for cross-language plugin interfaces.
- **Embedded systems** – C linkage is common in firmware.

---

## 7. Overloading Best Practices

### Definitions

**Core Definition**
Overloading best practices are guidelines for designing overload sets that are intuitive, unambiguous, and maintainable.

**Technical Definition**
These practices are codified in the C++ Core Guidelines, particularly **C.162** (“Overload operations that are roughly equivalent”), **C.163** (“Overload only for operations that are roughly equivalent”), and **F.51** (“Where there is a choice, prefer default arguments over overloading”). They also include advice on avoiding ambiguity, using `const` correctly, and preferring non-member functions for symmetric operators.

**Beginner-Friendly Explanation**
Best practices for overloading are simple rules: **only overload functions that do the same kind of thing**, **avoid confusing overload sets**, **use default arguments when the alternatives are the same type**, and **make sure the compiler can always tell which one you mean**.

### Key Guidelines (Summary Table)

| Guideline | Summary | Recommendation |
|-----------|---------|----------------|
| **C.162** | Overload operations that are roughly equivalent. | Overloads should have the same semantics. |
| **C.163** | Overload only for operations that are roughly equivalent. | Avoid overloading common names like `open`, `move`, `+`, `==` with unrelated meanings. |
| **F.51** | Where there is a choice, prefer default arguments over overloading. | If overloads differ only by trailing arguments of the same type, use a default argument. |
| **Avoid ambiguity** | Ensure every call has a unique best match. | Provide exact-match overloads or use explicit casts. |
| **Prefer non-member functions** for symmetric operators | `operator==(const T&, const T&)` as a non-member. | Avoid asymmetric behavior. |
| **Use `const` correctly** | Overload on `const` for member functions when both read-only and mutating versions are needed. | `T& operator[](size_t)` vs. `const T& operator[](size_t) const`. |
| **Do not overload on return type** | Impossible and misleading. | Use different function names instead. |
| **Do not overload on `noexcept`** | Impossible since C++17. | Document exception guarantees instead. |

### Annotated Code Examples

**Example 1: Prefer default arguments over overloading (F.51)**

```cpp
#include <iostream>

// GOOD: one function with a default argument
void print(int x, int base = 10) {
    // print x in the given base
    std::cout << "value=" << x << " base=" << base << '\n';
}

// AVOID: two overloads that differ only by a trailing argument of the same type
// void print(int x) { print(x, 10); }
// void print(int x, int base) { /* ... */ }

int main() {
    print(42);        // uses default base 10
    print(42, 16);    // explicit base 16
}
```

**Expected Output**
```
value=42 base=10
value=42 base=16
```

**Why this is good?**  
The default argument keeps the interface simple and avoids two nearly identical overloads. Core Guideline F.51 recommends this when the alternatives are from the same set of types.

**Example 2: Overloading for roughly equivalent operations**

```cpp
#include <iostream>
#include <string>

// GOOD: all overloads print a value — same conceptual operation
void display(int x)         { std::cout << "int: " << x << '\n'; }
void display(double x)      { std::cout << "double: " << x << '\n'; }
void display(const std::string& s) { std::cout << "string: " << s << '\n'; }

int main() {
    display(42);
    display(3.14);
    display("hello");
}
```

**Expected Output**
```
int: 42
double: 3.14
string: hello
```

**Why this is good?**  
All overloads perform the same logical operation (displaying a value). They differ only in the type of data, which is exactly the kind of overloading the guidelines encourage.

**Example 3: Avoiding ambiguous overload sets**

```cpp
#include <iostream>

// RISKY: int → long and int → float are both standard conversions
// void f(long);
// void f(float);

// BETTER: provide an exact match for int
void f(int x)   { std::cout << "f(int)\n"; }
void f(long x)  { std::cout << "f(long)\n"; }
void f(float x) { std::cout << "f(float)\n"; }

int main() {
    f(0);   // unambiguous: f(int) is an exact match
}
```

**Expected Output**
```
f(int)
```

**Why this is better?**  
Adding an exact-match `int` overload eliminates the ambiguity that would arise between `long` and `float` for an `int` argument.

### Real-World Cases

- **Standard library** – `std::vector` constructors, `std::string` overloads, and `std::abs` follow these guidelines.
- **API design** – library authors use Core Guidelines C.162/C.163/F.51 as design checklists.
- **Code review** – ambiguous overload sets are a common review finding.
- **Static analysis** – tools like clang-tidy warn about ambiguous calls and suggest default arguments.

---

## References

- Overload resolution – https://en.cppreference.com/w/cpp/language/overload_resolution
- Function declaration – https://en.cppreference.com/w/cpp/language/function
- Language linkage (`extern "C"`) – https://en.cppreference.com/w/cpp/language/language_linkage
- Name mangling (Itanium ABI) – https://itanium-cxx-abi.github.io/cxx-abi/abi.html#mangling
- Name mangling (IBM documentation) – https://www.ibm.com/docs/en/zos/2.4.0?topic=only-name-mangling-c
- C++ Core Guidelines — C.162: Overload operations that are roughly equivalent – https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rc-overload
- C++ Core Guidelines — C.163: Overload only for operations that are roughly equivalent – https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rc-overload
- C++ Core Guidelines — F.51: Where there is a choice, prefer default arguments over overloading – https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rf-default-args
- C++ Core Guidelines — C.160: Define operators primarily to mimic conventional usage – https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Ro-conventional
- C++ Core Guidelines — C.161: Use nonmember functions for symmetric operators – https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Ro-symmetric
- Itanium C++ ABI specification – https://itanium-cxx-abi.github.io/cxx-abi/
- GCC `-Wpessimizing-move` warning documentation – https://gcc.gnu.org/onlinedocs/gcc/C_002b_002b-Dialect-Options.html

---

This cheat sheet provides a complete, structured reference for C++ function overloading, covering every requested topic with definitions, syntax rules, annotated examples, expected outputs, and authoritative references. Each section follows the same uniform template to facilitate study and review.