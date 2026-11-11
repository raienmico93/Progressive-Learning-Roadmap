# Default Arguments — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
A default argument is a value specified in a function declaration that is automatically supplied by the compiler when the corresponding argument is omitted at the call site, allowing functions to be called with fewer arguments than they have parameters.

**Technical Definition**
Default arguments are specified using the syntax `parameter = initializer` in the parameter list of a function declaration or lambda expression (since C++11). They are used in place of missing trailing arguments during a function call. Default arguments are **not** part of the function type, are evaluated each time the function is called without the corresponding argument, and follow strict ordering and redeclaration rules that govern where they may appear and how they combine across declarations.

**Beginner-Friendly Explanation**
A default argument is a “backup value” for a function parameter. If you don’t provide a value for that parameter when you call the function, the compiler fills in the backup automatically. This lets you write one function that works with different levels of detail, without writing several separate functions.

### Key Characteristics

- **Trailing-only**: After a parameter with a default argument, every subsequent parameter must also have a default (or be a parameter pack, or an ellipsis).
- **Not part of the function type**: The presence or absence of default arguments does not affect the function’s type or signature; you cannot overload on default arguments alone.
- **Evaluated at call time**: The default expression is evaluated each time the function is called without the corresponding argument.
- **Scope-sensitive**: Default arguments are looked up and bound at the point of declaration, but the expressions are executed at the point of the call.
- **Cumulative across redeclarations**: In the same scope, default arguments from multiple visible declarations are combined, but the same parameter cannot receive a default twice.
- **Not inherited by virtual overrides**: A virtual function’s default arguments are determined by the **static type** of the object, not the dynamic type.

### Prerequisites

- Understanding of function declarations, definitions, and calls.
- Familiarity with the difference between parameters and arguments.
- Knowledge of scope and name lookup in C++.
- Basic awareness of overloading and overload resolution (default arguments interact with it).

### Related Programming Areas

- **Function overloading** – default arguments are often an alternative to writing multiple overloads.
- **API design** – default arguments simplify interfaces and reduce overload sets.
- **Overload resolution** – default arguments affect which functions are viable during a call.
- **Virtual functions** – default arguments are resolved statically, which can cause surprises.
- **Lambda expressions** – default arguments are permitted in lambdas (since C++11).

### Core Concepts / Features

1. Default Arguments
2. Default Argument Rules
3. Default Argument Pitfalls
4. Default Arguments vs. Overloads
5. Default Argument Best Practices

---

## 1. Default Arguments

### Definitions

**Core Definition**
A default argument is an initializer attached to a function parameter that supplies a value when the caller omits the corresponding argument.

**Technical Definition**
Default arguments are declared in the parameter list of a function declaration using the form `decl-specifier-seq declarator = initializer` or `decl-specifier-seq abstract-declarator(optional) = initializer`. They are used only in function declarations and lambda expressions; they are not permitted in pointer-to-function declarations, references to functions, or `typedef` declarations.

**Beginner-Friendly Explanation**
A default argument is a pre-filled value for a parameter. If you don’t supply that argument, the function uses the pre-filled value instead.

### Purposes

- To allow functions to be called with fewer arguments when certain parameters have sensible defaults.
- To simplify API interfaces by reducing the number of overloads needed.
- To provide backward-compatible extensions to existing functions (by adding a defaulted parameter).
- To improve readability by making common call patterns shorter.

### Syntax Rules and Structure

**General Syntax**
```cpp
return_type function_name(type1 param1 = default1, type2 param2 = default2, ...);
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `type` | The parameter type. |
| `param` | The parameter name (optional in declarations). |
| `= default` | The default argument expression. |

**Syntax Rules**
- After a parameter with a default argument, all subsequent parameters must also have defaults (or be a parameter pack, or an ellipsis).
- Default arguments may appear only in function declarations and lambda expressions (since C++11).
- Default arguments are **not** part of the function type; `void f(int = 0)` and `void f(int)` have the same type.
- Default arguments are evaluated at the point of the call, not at the point of declaration (though names are looked up at declaration).

**Constraints and Limitations**
- Local variables cannot be used in default arguments (unless in an unevaluated context).
- Non-static data members cannot be used in default arguments (except in member access expressions).
- Function parameters cannot be used in default arguments (except in unevaluated contexts).
- Operators other than `operator()` and `operator[]` (since C++23) cannot have default arguments.

### Annotated Code Examples

**Example 1: Basic default argument usage**

```cpp
#include <iostream>

void point(int x = 3, int y = 4) {
    std::cout << '(' << x << ", " << y << ")\n";
}

int main() {
    point();        // uses both defaults: (3, 4)
    point(1);       // x=1, y=4 (default)
    point(1, 2);    // both provided: (1, 2)
}
```

**Expected Output**
```
(3, 4)
(1, 4)
(1, 2)
```

**Why this output?**  
The first call omits both arguments, so both defaults are used. The second call provides `x` but omits `y`, so the default for `y` is used. The third call provides both arguments.

**Example 2: Default argument in a class member function**

```cpp
#include <iostream>

class Greeter {
public:
    void greet(const std::string& name = "Guest") {
        std::cout << "Hello, " << name << "!\n";
    }
};

int main() {
    Greeter g;
    g.greet();          // uses default "Guest"
    g.greet("Alice");   // explicit argument
}
```

**Expected Output**
```
Hello, Guest!
Hello, Alice!
```

**Why this output?**  
The member function declares a default argument for `name`; when the argument is omitted, the default `"Guest"` is used.

**Example 3: Default argument evaluated at call time**

```cpp
#include <iostream>

int counter = 0;

int next(int increment = ++counter) {   // default expression has side effect
    return increment;
}

int main() {
    std::cout << next() << '\n';   // counter becomes 1
    std::cout << next() << '\n';   // counter becomes 2
    std::cout << next(10) << '\n'; // counter unchanged; uses 10
}
```

**Expected Output**
```
1
2
10
```

**Why this output?**  
The default expression `++counter` is evaluated **each time** the function is called without an argument. The third call provides an explicit argument, so the default expression is not evaluated.

### Real-World Cases

- **Standard library** – `std::string` constructors, `std::vector` constructors, and many algorithms use default arguments for allocators, comparators, and policies.
- **GUI and graphics APIs** – functions with default colors, window flags, or sizes.
- **Configuration functions** – default timeouts, retry counts, and buffer sizes.
- **Mathematical utilities** – functions with default precision or rounding modes.

---

## 2. Default Argument Rules

### Definitions

**Core Definition**
Default argument rules are the constraints that govern where default arguments may appear, how they combine across declarations, and how they interact with scope, redeclaration, and other language features.

**Technical Definition**
The rules for default arguments are specified in [dcl.fct.default] of the C++ standard and summarized on cppreference. They cover trailing-only ordering, permitted declaration contexts, redeclaration and scope behavior, evaluation semantics, and interactions with virtual functions, templates, and friend declarations.

**Beginner-Friendly Explanation**
Default arguments follow a specific set of “house rules.” They must be at the **end** of the parameter list, they can be **added** in later declarations in the same scope, they are **not inherited** across scopes, and they are **resolved statically** for virtual functions. Knowing these rules prevents compile-time errors and subtle runtime surprises.

### Purposes

- To define exactly when and where default arguments are legal.
- To prevent ambiguous or ill-formed declarations.
- To guide the design of APIs that evolve over time.
- To explain the behavior of default arguments in inheritance and templates.

### Key Rules (Summary Table)

| Rule | Summary | Consequence |
|------|---------|-------------|
| **Trailing-only** | After a defaulted parameter, all following parameters must be defaulted, a parameter pack, or an ellipsis. | `void f(int = 1, int);` is an error. |
| **Declaration contexts** | Default arguments may appear only in function declarations and lambdas (since C++11). | Not allowed in pointer-to-function, reference-to-function, or `typedef` declarations. |
| **Redeclaration in same scope** | Defaults may be added in a redeclaration in the same scope; the visible set is the union of all visible defaults. | `void f(int, int); void f(int, int = 7);` is OK. |
| **No duplicate default** | A redeclaration cannot introduce a default for a parameter that already has a visible default (even with the same value). | `void f(int, int = 6); void f(int, int = 6);` is an error. |
| **Inner scope isolation** | Defaults from outer scopes are **not** acquired by declarations in inner scopes. | A block-scope declaration must supply its own defaults. |
| **Member functions** | Out-of-class definitions may supply defaults that combine with in-class declarations; defaults cannot turn a function into a special member. | `void C::f(int i = 3) {}` after `void f(int i = 3);` is an error. |
| **Virtual functions** | Overriders do **not** inherit defaults; defaults are chosen by the **static type** of the object. | A call through a base reference uses the base’s defaults. |
| **Evaluation** | Default expressions are looked up at declaration but **evaluated at each call** when the argument is missing. | Side effects in default expressions occur per call. |

### Annotated Code Examples

**Example 1: Trailing-only rule**

```cpp
// Error: only trailing parameters can have default arguments
// void bad(int x = 1, int y);

// OK: all parameters after the first defaulted one are also defaulted
void good(int x, int y = 2, int z = 3);

// OK: ellipsis can follow a defaulted parameter
void with_ellipsis(int x = 1, ...);
```

**Expected Output**  
(Compilation error for `bad`; `good` and `with_ellipsis` are valid.)

**Why this matters?**  
The trailing-only rule ensures that omitted arguments can be unambiguously matched to the remaining parameters.

**Example 2: Redeclaration in the same scope**

```cpp
#include <iostream>

void f(int, int);          // #1: no defaults
void f(int, int = 7);      // #2: adds a default for the second parameter

int main() {
    f(3);                  // OK: calls f(3, 7)
    f(3, 4);               // OK: calls f(3, 4)
}
```

**Expected Output**
```
(No output; illustrative.)
```

**Why this is valid?**  
Default arguments from all visible declarations in the same scope are combined. The second declaration adds a default that was not visible before.

**Example 3: Inner scope does not inherit defaults**

```cpp
void f(int, int = 1);      // outer scope: default for second parameter

void g() {
    void f(int, int);      // inner scope: no default
    // f(3);               // Error: not enough arguments
    f(3, 4);               // OK
}

void h() {
    void f(int, int = 6);  // inner scope: new default
    f(3);                  // OK: calls f(3, 6)
}
```

**Expected Output**  
(Compilation error for the commented-out call; the valid calls compile.)

**Why this matters?**  
A declaration in an inner scope does not acquire defaults from an outer scope; it must supply its own defaults if it wants them.

**Example 4: Virtual function default arguments are static**

```cpp
#include <iostream>

struct Base {
    virtual void f(int a = 7) { std::cout << "Base::f(" << a << ")\n"; }
};

struct Derived : Base {
    void f(int a) override { std::cout << "Derived::f(" << a << ")\n"; }
};

int main() {
    Derived d;
    Base& b = d;
    b.f();       // OK: uses Base's default 7, calls Derived::f(7)
    // d.f();    // Error: Derived::f has no default argument
}
```

**Expected Output**
```
Derived::f(7)
```

**Why this output?**  
Default arguments are resolved based on the **static type** of the object (`Base&`), even though the **dynamic type** is `Derived`. The override does not inherit the default argument.

### Real-World Cases

- **Library evolution** – adding a defaulted parameter to a function without breaking existing callers (but redeclaration rules must be respected).
- **Header/source split** – placing defaults in headers and ensuring out-of-class definitions follow the rules.
- **Template design** – understanding that member functions of class templates must provide all defaults in the initial declaration.

---

## 3. Default Argument Pitfalls

### Definitions

**Core Definition**
Default argument pitfalls are common mistakes and surprising behaviors that arise from the interaction of default arguments with overload resolution, scope, evaluation, and API design.

**Technical Definition**
Pitfalls stem from the fact that default arguments are not part of the function type, are invisible to function pointers, affect overload resolution by expanding the viable candidate set, and can silently change behavior when combined with implicit conversions or virtual dispatch.

**Beginner-Friendly Explanation**
Default arguments can be tricky. A call that looks correct might silently do the wrong thing because a default value was used when you didn’t expect it, or because the compiler picked a different overload. Knowing the pitfalls helps you avoid subtle bugs.

### Common Pitfalls (Summary)

| Pitfall | Description | Example / Symptom |
|---------|-------------|-------------------|
| **Ambiguity with overloads** | Default arguments can make multiple overloads viable for the same call, causing ambiguity. | `void f(int); void f(int, int = 0);` called with `f(5)` is ambiguous. |
| **Invisible to function pointers** | A function pointer cannot be called with fewer arguments; defaults are not part of the type. | `int (*p)(int) = &f; p();` is an error even if `f` has a default. |
| **Implicit conversion surprises** | A default argument can trigger unintended implicit conversions, especially with pointer-to-bool or similar conversions. | `void f(int a, bool d = false);` called with a pointer may silently convert to `bool`. |
| **Evaluation side effects** | Default expressions with side effects are evaluated on every call that omits the argument. | A default of `++counter` increments on each call. |
| **Static binding for virtuals** | Default arguments for virtual functions are chosen by static type, leading to unexpected values. | A base reference uses the base’s defaults even when calling a derived override. |
| **Redeclaration confusion** | Adding a default in an inner scope does not inherit outer defaults, causing unexpected errors. | `void f(int, int = 1); void g() { void f(int, int); f(3); }` is an error. |
| **Turning functions into special members** | Out-of-class defaults can accidentally turn a constructor into a default constructor, making the program ill-formed. | `C::C(int arg = 1) {}` is an error if `C` already has a default constructor. |

### Annotated Code Examples

**Example 1: Ambiguity from default arguments**

```cpp
#include <iostream>

void f(int)           { std::cout << "f(int)\n"; }
void f(int, int = 0)  { std::cout << "f(int, int)\n"; }

int main() {
    f(5);   // error: ambiguous — both overloads are viable
}
```

**Expected Output**  
(Compilation error: call of overloaded `f(int)` is ambiguous.)

**Why this is a pitfall?**  
Default arguments are applied during overload resolution, making both `f(int)` and `f(int, int=0)` viable for a single-`int` call. Neither is better, so the call is ambiguous.

**Example 2: Implicit conversion through defaulted parameter**

```cpp
#include <iostream>

void set_flag(int value, bool enabled = false) {
    std::cout << "value=" << value << " enabled=" << enabled << '\n';
}

int main() {
    int x = 42;
    set_flag(1, &x);   // pointer implicitly converts to bool! (true)
}
```

**Expected Output**
```
value=1 enabled=1
```

**Why this is a pitfall?**  
The pointer `&x` is implicitly converted to `bool` (`true`), and the function accepts it because the second parameter has a default that makes the call “look” like it should work. This is a classic source of silent bugs.

**Example 3: Default expression evaluated on every call**

```cpp
#include <iostream>

int calls = 0;

void log(const std::string& msg = "default #" + std::to_string(++calls)) {
    std::cout << msg << '\n';
}

int main() {
    log();    // "default #1"
    log();    // "default #2"
    log("explicit");   // "explicit"
}
```

**Expected Output**
```
default #1
default #2
explicit
```

**Why this is a pitfall?**  
The default expression has a side effect (`++calls`) that occurs on every call that omits the argument. If you expected the default to be computed once, this behavior can be surprising.

### Real-World Cases

- **C++ standard library proposals** – discussions on lists.isocpp.org highlight real bugs caused by default arguments combined with implicit conversions.
- **API evolution** – adding a defaulted parameter can silently change overload resolution and break existing code.
- **Legacy codebases** – default arguments in constructors can accidentally create default constructors, leading to ambiguity.

---

## 4. Default Arguments vs. Overloads

### Definitions

**Core Definition**
Default arguments and function overloading are two alternative mechanisms for allowing a function to be called with different numbers or types of arguments; they differ in explicitness, overload-resolution behavior, and interface design.

**Technical Definition**
Default arguments provide a **single function** with multiple call signatures (by omitting trailing arguments), while overloading provides **multiple distinct functions** with the same name but different parameter lists. The C++ Core Guidelines recommend preferring default arguments over overloading when the alternatives are of the same type and the semantics are identical (F.51).

**Beginner-Friendly Explanation**
Both features let you call a function in different ways. Default arguments say: “Use one function, and fill in missing values automatically.” Overloading says: “Write several functions with the same name, and the compiler picks the right one.” Default arguments are simpler when the only difference is a trailing value; overloading is more flexible when the parameter **types** differ.

### Comparison Table

| Aspect | Default Arguments | Overloading |
|--------|-------------------|-------------|
| **Number of functions** | One function with multiple call signatures. | Multiple distinct functions. |
| **Interface clarity** | Simpler interface; fewer names to document. | More explicit; each overload can have distinct semantics. |
| **Overload resolution** | Default arguments expand the viable set; can cause ambiguity. | Resolution is based on parameter types; no default-expansion. |
| **Function pointers** | Defaults are invisible to function pointers. | Each overload has its own pointer type. |
| **Semantic guarantees** | One implementation ensures consistent semantics. | Different overloads may accidentally diverge in behavior. |
| **Core Guidelines** | **Preferred** when the choice is between default arguments and overloading for the same types (F.51). | Use when parameter **types** or semantics genuinely differ. |

### Annotated Code Examples

**Example 1: Default arguments preferred (F.51)**

```cpp
#include <iostream>

// GOOD: one function with a default argument
void print(int x, int base = 10) {
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

**Why this is preferred?**  
Default arguments reduce the number of functions in the overload set, making the interface simpler and ensuring both call forms use the same implementation.

**Example 2: Overloading when types differ**

```cpp
#include <iostream>
#include <string>

// Overloading is appropriate here: parameter types differ
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

**Why overloading here?**  
Default arguments cannot distinguish between parameter **types**; overloading is necessary when the operation applies to genuinely different types with potentially different implementations.

### Real-World Cases

- **Standard library** – `std::vector` constructors use overloading because the parameter types differ (size, initial value, allocator, etc.).
- **API design** – Core Guideline F.51 is applied when extending functions with optional trailing parameters of the same type.
- **Code review** – reviewers often ask to replace trivial overload sets with default arguments to reduce duplication.

---

## 5. Default Argument Best Practices

### Definitions

**Core Definition**
Default argument best practices are guidelines for using default arguments safely, clearly, and consistently, while avoiding ambiguity and hidden behavior.

**Technical Definition**
These practices are anchored in the C++ Core Guidelines, particularly **F.51** (“Where there is a choice, prefer default arguments over overloading”), and in community wisdom about API design, evaluation side effects, and interaction with overload resolution and virtual functions.

**Beginner-Friendly Explanation**
Best practices are simple rules: **use default arguments when the only difference is a trailing value of the same type**, **avoid default arguments that trigger surprising implicit conversions**, **don’t rely on side effects in defaults**, and **be careful with virtual functions**. Following these rules keeps your code predictable.

### Key Guidelines (Summary Table)

| Guideline | Summary | Recommendation |
|-----------|---------|----------------|
| **F.51** | Prefer default arguments over overloading when the choice exists. | Use a single function with a defaulted trailing parameter instead of trivial overloads. |
| **Prefer same-type alternatives** | Default arguments work best when the omitted value is of the same type as the parameter. | Avoid mixing default arguments with implicit conversions that can silently change meaning. |
| **Avoid side effects** | Default expressions are evaluated on every call that omits the argument. | Keep default expressions simple and side-effect-free. |
| **Be explicit with virtuals** | Virtual overrides do not inherit defaults; defaults are chosen by static type. | Consider the non-virtual interface (NVI) pattern or avoid defaults on virtuals. |
| **Respect redeclaration rules** | Add defaults only in the same scope, and never duplicate a visible default. | Follow the rules summarized in Section 2. |
| **Document defaults** | Default values are part of the interface contract. | Document them in comments or API documentation. |
| **Consider `std::optional`** | If “no value” is a meaningful state, consider `std::optional` instead of a default. | Default arguments represent a **default value**, not an **absent** value. |

### Annotated Code Examples

**Example 1: Prefer default arguments over trivial overloads (F.51)**

```cpp
#include <iostream>
#include <string>

// GOOD: one function, one implementation, default for the optional part
void configure(std::string name, int timeout = 30, bool verbose = false) {
    std::cout << "name=" << name
              << " timeout=" << timeout
              << " verbose=" << verbose << '\n';
}

// AVOID: multiple overloads that differ only by trailing same-type arguments
// void configure(std::string name);
// void configure(std::string name, int timeout);
// void configure(std::string name, int timeout, bool verbose);

int main() {
    configure("server");
    configure("server", 60);
    configure("server", 60, true);
}
```

**Expected Output**
```
name=server timeout=30 verbose=0
name=server timeout=60 verbose=0
name=server timeout=60 verbose=1
```

**Why this is best practice?**  
The default-argument version guarantees that all call forms use the same implementation, reducing the risk of semantic divergence and making the interface easier to understand.

**Example 2: Keeping defaults side-effect-free**

```cpp
#include <iostream>
#include <string>

// GOOD: default is a simple, side-effect-free value
void log_message(const std::string& msg, const std::string& prefix = "[INFO] ") {
    std::cout << prefix << msg << '\n';
}

int main() {
    log_message("Started");                  // uses default prefix
    log_message("Failed", "[ERROR] ");       // explicit prefix
}
```

**Expected Output**
```
[INFO] Started
[ERROR] Failed
```

**Why this is best practice?**  
The default `"[INFO] "` is a constant expression with no side effects, so its evaluation on each call is predictable and inexpensive.

**Example 3: Avoiding defaults on virtual functions**

```cpp
#include <iostream>

struct Base {
    // AVOID: default on a virtual function
    virtual void process(int x = 10) { std::cout << "Base: " << x << '\n'; }
    virtual ~Base() = default;
};

struct Derived : Base {
    void process(int x) override { std::cout << "Derived: " << x << '\n'; }
};

int main() {
    Derived d;
    Base& b = d;
    b.process();   // uses Base's default 10 → Derived::process(10)
    // d.process(); // error: Derived::process has no default
}
```

**Expected Output**
```
Derived: 10
```

**Why this requires care?**  
Defaults on virtual functions are resolved statically, which can confuse readers. The Core Guidelines suggest avoiding this pattern in favor of explicit arguments or the NVI pattern.

### Real-World Cases

- **Standard library** – `std::string` constructors, `std::vector` constructors, and many algorithms use default arguments judiciously.
- **API evolution** – adding a defaulted trailing parameter is a common backward-compatible extension technique (but must follow redeclaration rules).
- **Code review** – Core Guideline F.51 is frequently cited when simplifying overload sets.
- **Static analysis** – tools can warn about side effects in default arguments and about ambiguous overload sets caused by defaults.

---

## References

- Default arguments (cppreference) – https://en.cppreference.com/w/cpp/language/default_arguments
- C++ Core Guidelines — F.51: Where there is a choice, prefer default arguments over overloading – https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rf-default-args
- C++ Core Guidelines (main page) – https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines
- Default arguments — cppreference revision history – https://en.cppreference.com/w/cpp/language/default_arguments (see lines on evaluation and type)
- std-proposals discussion on default argument pitfalls – https://lists.isocpp.org/std-proposals/att-18130/attachment
- Microsoft Learn — Compiler error C2548 (default argument missing) – https://learn.microsoft.com/en-us/cpp/error-messages/compiler-errors-1/compiler-error-c2548
- GCC Bug 8285 — default argument in `vector` – https://gcc.gnu.org/bugzilla/show_bug.cgi?id=8285

---

This cheat sheet provides a complete, structured reference for C++ default arguments, covering every requested topic with definitions, rules, annotated examples, expected outputs, pitfalls, comparisons, and best practices. Each section follows a uniform template to facilitate study and review.