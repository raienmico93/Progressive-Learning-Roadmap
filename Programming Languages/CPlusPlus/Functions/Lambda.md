# Lambda Expressions — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
A lambda expression is a C++11 feature that constructs a **closure** — an unnamed function object capable of capturing variables from its enclosing scope — directly at the point where it is needed.

**Technical Definition**
A lambda expression is a prvalue expression of a unique, unnamed, non-union, non-aggregate class type known as the *closure type*. The closure type has a function call operator (`operator()`) whose parameters, body, and specifiers are determined by the lambda expression. Variables from the enclosing scope may be captured by copy or by reference, and since C++14, captures may be initialized with arbitrary expressions (init-captures), enabling move capture and renaming. Since C++20, lambdas may declare explicit template parameter lists, making them *templated lambdas*.

**Beginner-Friendly Explanation**
A lambda is a **throwaway function** you write right where you need it, without giving it a name. It can “capture” nearby variables so it can use them later. Think of it as a sticky note with a small piece of code on it: you write it, hand it to an algorithm or store it in a variable, and it remembers the context you gave it. Lambdas make C++ code shorter and more expressive, especially when working with the STL.

### Key Characteristics

- **Unnamed**: Every lambda introduces a unique closure type; no name is declared for it.
- **Capturing**: Lambdas can capture variables from the enclosing scope by copy, by reference, by move (via init-capture), or with a capture-default.
- **Value category**: A lambda expression is a prvalue; the resulting closure object is a temporary until bound to a variable.
- **`const` by default**: The closure type’s `operator()` is `const` unless the lambda is declared `mutable`.
- **Generic since C++14**: Parameters may be declared with `auto`, making `operator()` a template.
- **Templated since C++20**: Explicit template parameter lists may be declared between `[]` and `()`.
- **Return type deducible**: The return type is deduced from `return` statements unless a trailing return type is specified.

### Prerequisites

- Understanding of function objects (functors) and `operator()`.
- Familiarity with value categories (lvalues, rvalues) and references.
- Knowledge of capture semantics and object lifetime.
- Basic awareness of templates (for generic and templated lambdas).

### Related Programming Areas

- **STL algorithms** – lambdas are the primary way to supply predicates and operations.
- **Function objects** – lambdas are syntactic sugar for compiler-generated functors.
- **`std::function`** – lambdas can be stored in `std::function` (with type erasure).
- **Move semantics** – init-capture enables move-only types to be captured.
- **Templates and generic programming** – generic and templated lambdas extend this paradigm.
- **Concurrency** – lambdas are commonly passed to `std::thread`, `std::async`, etc.

### Core Concepts / Features

1. Lambda Expressions
2. Lambda Syntax
3. Capture List
4. Capture by Value
5. Capture by Reference
6. Capture by Move
7. Init Capture
8. `mutable` Lambdas
9. Generic Lambdas
10. Templated Lambdas
11. Lambda Return Type
12. Lambdas in STL Algorithms
13. Lambda Best Practices

---

## 1. Lambda Expressions

### Definitions

**Core Definition**
A lambda expression is a C++ construct that creates an unnamed function object (a closure) capable of capturing variables from its enclosing scope.

**Technical Definition**
A lambda expression is a prvalue expression of unique unnamed non-union non-aggregate class type, known as the *closure type*, which is declared in the smallest block scope, class scope, or namespace scope that contains the lambda expression. The closure type has a public `operator()` whose parameters and body are those of the lambda. Captured entities become non-static data members of the closure type (for by-copy captures) or are accessed through references (for by-reference captures).

**Beginner-Friendly Explanation**
A lambda is a **mini-function you write inline**. You don’t need to declare it elsewhere or give it a name. You write `[](){ ... }` and you have a function object you can pass around or call immediately. It can “remember” variables from around it, which is what makes it so useful for algorithms and callbacks.

### Purposes

- To define short, localized operations without writing a separate named function or functor class.
- To pass custom behavior to STL algorithms (`std::sort`, `std::transform`, `std::for_each`, etc.).
- To create callbacks and event handlers concisely.
- To capture context for deferred execution (e.g., in threads or asynchronous tasks).
- To simplify code that would otherwise require verbose function objects.

### Syntax Rules and Structure

**General Syntax (C++11/14/17)**
```cpp
[captures](params) specs -> ret { body }
```

**General Syntax (C++20+, templated lambda)**
```cpp
[captures]<tparams>(params) specs -> ret { body }
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `[captures]` | Capture list: specifies which outer variables are accessible inside the body and how. |
| `<tparams>` | (C++20) Optional explicit template parameter list for templated lambdas. |
| `(params)` | Parameter list, as in a named function. May be omitted if there are no parameters. |
| `specs` | Optional specifiers: `mutable`, `constexpr`, `consteval` (C++20), `static` (C++23), `noexcept`, attributes. |
| `-> ret` | Optional trailing return type. If omitted, the return type is deduced. |
| `{ body }` | The function body. |

**Syntax Rules**
- A lambda expression is a prvalue; the closure object is a temporary until stored.
- The capture list may be empty (`[]`), a capture-default (`[=]` or `[&]`), or an explicit list (`[x, &y]`).
- If the parameter list is omitted, the lambda takes no parameters (`[]{}` is equivalent to `[](){}`).
- The `mutable` specifier allows modification of by-copy captures inside the body.
- `constexpr` (C++17) makes `operator()` a `constexpr` function; since C++17, lambdas are implicitly `constexpr` if possible.
- `consteval` (C++20) makes the lambda an immediate function.

**Constraints and Limitations**
- Lambdas cannot be overloaded directly (the closure type has a single `operator()`).
- The closure type is not default-constructible, assignable, or swappable unless the captures allow it.
- Capturing `this` captures the pointer, not the object; the object’s lifetime must be managed carefully.

### Annotated Code Examples

**Example 1: A minimal lambda**

```cpp
#include <iostream>

int main() {
    auto greet = [] { std::cout << "Hello from a lambda!\n"; };
    greet();                       // call the closure
}
```

**Expected Output**
```
Hello from a lambda!
```

**Why this output?**  
The lambda `[] { ... }` creates a closure with an `operator()` that prints the message. `greet()` invokes that operator.

**Example 2: Lambda with parameters**

```cpp
#include <iostream>

int main() {
    auto add = [](int a, int b) { return a + b; };
    std::cout << add(3, 4) << '\n';
}
```

**Expected Output**
```
7
```

**Why this output?**  
The lambda takes two `int` parameters and returns their sum. The return type is deduced as `int`.

### Real-World Cases

- **STL algorithms** – `std::sort(v.begin(), v.end(), [](int a, int b){ return a > b; });`.
- **Event callbacks** – `button.on_click([&]{ save(); });`.
- **Thread creation** – `std::thread t([&]{ worker(); });`.
- **Immediate invocation** – `[&]{ /* setup */ }();` (IIFE-style initialization).

---

## 2. Lambda Syntax

### Definitions

**Core Definition**
Lambda syntax is the specific grammatical form used to write a lambda expression, including the capture list, parameter list, specifiers, return type, and body.

**Technical Definition**
The full lambda grammar (simplified from the standard) is:

```cpp
[captures] <tparams>(optional) (params) specs except attr -> ret requires { body }
```

where `captures` is a comma-separated list of captures (optionally beginning with a capture-default), `tparams` is an optional template parameter list (C++20), `params` is an optional parameter list, `specs` includes `mutable`, `constexpr`, `consteval`, `static` (C++23), `except` is an optional exception specification, `attr` is optional attributes, `ret` is an optional trailing return type, `requires` is an optional requires-clause (C++20), and `body` is the compound statement.

**Beginner-Friendly Explanation**
The syntax is like a recipe with several optional parts. At minimum you need `[]` (capture list) and `{}` (body). You can add `()` for parameters, `mutable` if you want to modify copies, and `-> type` if you want to be explicit about the return type. In C++20 you can even add template parameters between `[]` and `()`.

### Purposes

- To provide a consistent, readable way to write inline function objects.
- To allow fine-grained control over captures, parameters, and return types.
- To support evolution from basic lambdas (C++11) to generic/templated lambdas (C++14/20).

### Syntax Rules and Structure

**Simplified Grammar**

| Form | Meaning | Since |
|------|---------|-------|
| `[captures] { body }` | No parameters (equivalent to `[](){}`) | C++11 |
| `[captures](params) { body }` | Full form | C++11 |
| `[captures](params) mutable { body }` | Mutable lambda | C++11 |
| `[captures](params) -> ret { body }` | Explicit return type | C++11 |
| `[captures](auto params) { body }` | Generic lambda | C++14 |
| `[captures]<tparams>(params) { body }` | Templated lambda | C++20 |
| `[captures]<tparams>(params) requires C { body }` | Constrained lambda | C++20 |

**Component Breakdown**

| Component | Required? | Description |
|-----------|-----------|-------------|
| `[captures]` | Yes | Capture list (may be empty). |
| `<tparams>` | No (C++20) | Template parameter list. |
| `(params)` | No | Parameter list (omitted means no parameters). |
| `specs` | No | `mutable`, `constexpr`, `consteval`, `static`, `noexcept`. |
| `-> ret` | No | Trailing return type. |
| `{ body }` | Yes | Function body. |

**Constraints and Limitations**
- The capture list `[]` is always required, even if empty.
- You cannot have both a capture-default and an explicit init-capture of the same name in some ambiguous cases.
- Specifiers must appear in a specific order (e.g., `mutable` before `noexcept`).

### Annotated Code Examples

**Example 1: Various syntax forms**

```cpp
#include <iostream>
#include <vector>

int main() {
    auto no_params = []{ return 42; };                 // no parameter list
    auto with_params = [](int x){ return x * 2; };     // parameter list
    auto mutable_l = [n = 0]() mutable { return ++n; }; // init-capture + mutable
    auto explicit_ret = [](int x) -> long { return x * 1L; }; // trailing return type

    std::cout << no_params() << '\n';
    std::cout << with_params(5) << '\n';
    std::cout << mutable_l() << '\n';
    std::cout << mutable_l() << '\n';
    std::cout << explicit_ret(3) << '\n';
}
```

**Expected Output**
```
42
10
1
2
3
```

**Why this output?**  
Each lambda uses a different syntactic form. `mutable_l` retains its state between calls because the init-capture `n = 0` creates a member, and `mutable` allows modification.

### Real-World Cases

- **Passing to algorithms** – `std::find_if(v.begin(), v.end(), [](int x){ return x > 0; });`.
- **Thread lambdas** – `std::thread t([&]{ run(); });`.
- **Immediate execution** – `auto x = [&]{ /* init */ return value; }();`.

---

## 3. Capture List

### Definitions

**Core Definition**
The capture list is the part of a lambda expression between `[` and `]` that specifies which variables from the enclosing scope are accessible inside the lambda body and how they are captured (by copy, by reference, or via init-capture).

**Technical Definition**
A capture list is a comma-separated list of zero or more captures, optionally beginning with a capture-default (`=` or `&`). Each capture specifies an entity from the reaching scope: by copy (`x`), by reference (`&x`), by move/init-capture (`x = expr`), or `this`. A capture-default (`[=]` or `[&]`) implicitly captures any odr-used entity not explicitly captured. Since C++20, a capture-default cannot be combined with an explicit capture of `this` in certain forms, and `*this` may be captured by copy (C++17).

**Beginner-Friendly Explanation**
The capture list is the lambda’s **“guest list.”** It tells the lambda which outside variables it is allowed to use and whether it gets its own copy of them or just a reference. You can list variables individually (`[x, &y]`), use a default (`[=]` or `[&]`), or create new variables with initializers (`[z = x + y]`).

### Purposes

- To control which outer variables the lambda can access.
- To choose between copy (independent snapshot) and reference (shared, live view).
- To capture move-only types via init-capture.
- To rename captured variables for clarity.
- To capture `this` for member access.

### Syntax Rules and Structure

**General Syntax**
```cpp
[ capture1, capture2, ... ]
```

**Capture Forms**

| Form | Meaning | Since |
|------|---------|-------|
| `[x]` | Capture `x` by copy. | C++11 |
| `[&x]` | Capture `x` by reference. | C++11 |
| `[=]` | Capture any odr-used variable by copy (capture-default). | C++11 |
| `[&]` | Capture any odr-used variable by reference (capture-default). | C++11 |
| `[this]` | Capture `this` pointer. | C++11 |
| `[*this]` | Capture the object by copy. | C++17 |
| `[x = expr]` | Init-capture: create a new variable `x` initialized with `expr`. | C++14 |
| `[&x = expr]` | Init-capture by reference. | C++14 |
| `[x = std::move(y)]` | Move capture. | C++14 |

**Syntax Rules**
- A capture-default must appear first if present.
- A capture-default cannot be combined with an explicit `this` capture in some contexts (C++20 deprecates `[=]` capturing `this` implicitly).
- The same entity cannot be captured twice (e.g., `[x, x]` is an error).
- `[*this]` captures a copy of the object; `[this]` captures the pointer.
- Init-captures introduce new names visible only inside the lambda.

**Constraints and Limitations**
- Capturing by reference creates a reference member; the lambda does not extend the referent’s lifetime. Dangling references are possible if the lambda outlives the captured object.
- Capture-defaults can accidentally capture more than intended; explicit captures are safer for non-local lambdas.
- `[*this]` requires the class to be copy-constructible.

### Annotated Code Examples

**Example 1: Explicit captures**

```cpp
#include <iostream>

int main() {
    int a = 10, b = 20;
    auto f = [a, &b] { return a + b; };   // a by copy, b by reference
    b = 30;
    std::cout << f() << '\n';              // a=10, b=30
}
```

**Expected Output**
```
40
```

**Why this output?**  
`a` is copied (stays `10`), `b` is referenced (reflects the updated value `30`). The lambda computes `10 + 30 = 40`.

**Example 2: Capture-default**

```cpp
#include <iostream>

int main() {
    int x = 5, y = 10;
    auto sum = [=] { return x + y; };      // capture all by copy
    x = 100;                                // does not affect the lambda
    std::cout << sum() << '\n';
}
```

**Expected Output**
```
15
```

**Why this output?**  
`[=]` captures `x` and `y` by copy at the point of lambda creation. Later modification of `x` does not affect the captured copy.

**Example 3: Capture `this`**

```cpp
#include <iostream>

struct Button {
    int clicks = 0;
    auto get_clicker() {
        return [this] { return ++clicks; };   // captures this pointer
    }
};

int main() {
    Button b;
    auto click = b.get_clicker();
    std::cout << click() << ' ' << click() << '\n';
}
```

**Expected Output**
```
1 2
```

**Why this output?**  
`[this]` captures the pointer to `b`; each call increments `b.clicks` through that pointer.

### Real-World Cases

- **STL predicates** – capturing a threshold by value: `[threshold](int x){ return x > threshold; }`.
- **Mutating shared state** – capturing an accumulator by reference: `[&total](int x){ total += x; }`.
- **Member callbacks** – capturing `this` to call member functions.
- **Move-only resources** – init-capture with `std::move` (see §6).

---

## 4. Capture by Value

### Definitions

**Core Definition**
Capturing by value means the lambda receives its own **copy** of the captured variable, independent of the original.

**Technical Definition**
When a variable is captured by copy (`[x]` or via a capture-default `[=]`), the closure type contains an unnamed non-static data member of the variable’s type, initialized with the variable’s value at the point of lambda creation. The member is `const` inside the lambda body unless the lambda is declared `mutable`. The lifetime of the copy is tied to the lifetime of the closure object.

**Beginner-Friendly Explanation**
Capture by value is like taking a **photograph** of a variable. The lambda keeps the snapshot; if the original variable changes later, the lambda still sees the old value. This is safe and avoids dangling references, but copying large objects can be expensive.

### Purposes

- To create an independent snapshot of a variable, immune to later changes.
- To avoid dangling references when the lambda outlives the scope of the captured variable.
- To safely pass lambdas to asynchronous tasks or store them beyond the current scope.
- To capture small, cheap-to-copy types without reference overhead.

### Syntax Rules and Structure

**General Syntax**
```cpp
[x]        // capture x by copy
[=]        // capture all odr-used variables by copy
```

**Component Breakdown**

| Form | Description |
|------|-------------|
| `[x]` | Copy `x` into a member of the closure. |
| `[=]` | Copy every odr-used variable not explicitly captured. |

**Syntax Rules**
- The copy is made at the point of lambda creation, not at the point of call.
- Inside a non-`mutable` lambda, the copy is `const`; it cannot be modified.
- Inside a `mutable` lambda, the copy can be modified, but the original is unaffected.
- The copy must be copy-constructible.

**Constraints and Limitations**
- Copying large objects (e.g., `std::vector`, `std::string`) can be expensive.
- Move-only types (e.g., `std::unique_ptr`) cannot be captured by copy; use init-capture with `std::move` instead.
- A capture-default `[=]` may capture more than intended; explicit captures are clearer.

### Annotated Code Examples

**Example 1: Snapshot semantics**

```cpp
#include <iostream>

int main() {
    int value = 10;
    auto f = [value] { return value; };   // copy
    value = 99;
    std::cout << f() << '\n';             // still 10
}
```

**Expected Output**
```
10
```

**Why this output?**  
The lambda captured `value` by copy when it was `10`. The later assignment to `value` does not affect the copy.

**Example 2: `mutable` allows modifying the copy**

```cpp
#include <iostream>

int main() {
    int counter = 0;
    auto increment = [counter]() mutable { return ++counter; };
    std::cout << increment() << ' ' << increment() << '\n';
    std::cout << counter << '\n';   // original unchanged
}
```

**Expected Output**
```
1 2
0
```

**Why this output?**  
The lambda holds its own copy of `counter`. `mutable` allows the copy to be incremented. The original `counter` remains `0`.

### Real-World Cases

- **Asynchronous callbacks** – capturing a snapshot of configuration values by copy so the lambda can safely run on another thread.
- **Delayed evaluation** – storing a lambda with copied inputs for later execution.
- **Small predicates** – capturing an `int` threshold by value.

---

## 5. Capture by Reference

### Definitions

**Core Definition**
Capturing by reference means the lambda receives an **alias** to the original variable; changes through the lambda affect the original, and changes to the original are visible through the lambda.

**Technical Definition**
When a variable is captured by reference (`[&x]` or via `[&]`), the closure type stores a reference to the original entity (or an equivalent mechanism). The lifetime of the reference is not tied to the lifetime of the closure; if the lambda outlives the referent, the reference dangles, and accessing it is undefined behavior.

**Beginner-Friendly Explanation**
Capture by reference is like giving the lambda a **live video feed** of your variable. Whatever happens to the variable is immediately visible to the lambda, and the lambda can change the variable directly. This is efficient (no copying), but dangerous if the lambda survives longer than the variable.

### Purposes

- To avoid copying large objects while allowing the lambda to read and modify them.
- To share mutable state between the lambda and the enclosing scope.
- To implement accumulators, counters, and in-place transformations.
- To be efficient in local, short-lived algorithm calls.

### Syntax Rules and Structure

**General Syntax**
```cpp
[&x]       // capture x by reference
[&]        // capture all odr-used variables by reference
```

**Component Breakdown**

| Form | Description |
|------|-------------|
| `[&x]` | Reference to `x`. |
| `[&]` | Reference to every odr-used variable not explicitly captured. |

**Syntax Rules**
- The lambda can modify the referent even without `mutable` (because the reference itself is `const`, but the referent is not).
- The referent must outlive every call to the lambda.
- `[&]` should only be used when the lambda is obviously short-lived (e.g., passed to an STL algorithm).

**Constraints and Limitations**
- **Dangling references**: If the lambda outlives the captured variable, using the lambda is undefined behavior.
- Returning a lambda that captures locals by reference is almost always a bug (CERT EXP61-CPP).
- Storing a reference-capturing lambda in a member or global is dangerous unless lifetime is carefully managed.

### Annotated Code Examples

**Example 1: Modifying through a reference**

```cpp
#include <iostream>

int main() {
    int total = 0;
    auto add_to_total = [&total](int x) { total += x; };
    add_to_total(10);
    add_to_total(20);
    std::cout << total << '\n';
}
```

**Expected Output**
```
30
```

**Why this output?**  
`total` is captured by reference; `+=` inside the lambda modifies the original variable.

**Example 2: Dangling reference (undefined behavior — illustrative)**

```cpp
// DO NOT WRITE THIS CODE
#include <functional>

std::function<int()> make_bad_lambda() {
    int local = 42;
    return [&local] { return local; };   // local dies when function returns
}

int main() {
    auto f = make_bad_lambda();
    // int x = f();   // undefined behavior: dangling reference
}
```

**Expected Output**  
(Undefined behavior if the returned lambda is called.)

**Why this is wrong?**  
`local` is destroyed when `make_bad_lambda` returns, but the lambda still holds a reference to it. Calling the lambda dereferences a dangling reference.

### Real-World Cases

- **STL algorithm predicates** – `std::find_if(v.begin(), v.end(), [&](int x){ return x == target; });`.
- **Accumulators** – `std::for_each(v.begin(), v.end(), [&sum](int x){ sum += x; });`.
- **In-place transforms** – `std::transform(in.begin(), in.end(), out.begin(), [&scale](int x){ return x * scale; });`.
- **Short-lived callbacks** – event handlers within a single scope.

---

## 6. Capture by Move

### Definitions

**Core Definition**
Capture by move is a form of init-capture (C++14) that **moves** an object into the lambda’s closure, allowing move-only types (such as `std::unique_ptr`) to be captured.

**Technical Definition**
Capture by move is written as `[x = std::move(y)]`. It creates a new member `x` of the closure type, initialized by move-constructing from `y`. The original `y` is left in a valid but unspecified state. This is the only way to capture move-only types, because the copy-capture mechanism requires copy-constructibility.

**Beginner-Friendly Explanation**
Some objects (like `std::unique_ptr`) cannot be copied — they can only be **moved**. Normal capture by value would fail to compile. Capture by move says: “Take this object and put it inside the lambda; the original doesn’t need it anymore.” This transfers ownership cleanly.

### Purposes

- To capture move-only types (e.g., `std::unique_ptr`, `std::thread`, `std::fstream`).
- To avoid copying large objects that the caller does not need to retain.
- To transfer ownership of a resource into a closure.
- To enable move-semantics in callbacks and asynchronous tasks.

### Syntax Rules and Structure

**General Syntax**
```cpp
[x = std::move(y)]       // move y into new member x
[x = std::move(y), ...]  // multiple move captures
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `x` | The new member name inside the lambda. |
| `= std::move(y)` | Move-initializes `x` from `y`. |

**Syntax Rules**
- `std::move` casts `y` to an rvalue; the move constructor of the member’s type is invoked.
- The original `y` must not be used after the capture (except to be reassigned or destroyed).
- Capture by move requires the type to be move-constructible.
- The lambda’s `operator()` is `const` by default; calling non-`const` methods on the moved-to member requires `mutable`.

**Constraints and Limitations**
- After moving, `y` is in a valid but unspecified state; do not rely on its value.
- If the lambda itself is copied, the moved-to member is copied (not moved), which may fail for move-only types.
- Move capture does not automatically make the lambda `mutable`.

### Annotated Code Examples

**Example 1: Capturing `std::unique_ptr`**

```cpp
#include <iostream>
#include <memory>

int main() {
    auto ptr = std::make_unique<int>(42);
    auto f = [p = std::move(ptr)] { return *p; };   // move capture
    std::cout << f() << '\n';
    // ptr is now empty
}
```

**Expected Output**
```
42
```

**Why this output?**  
`ptr` is moved into the closure member `p`. The lambda dereferences `p` and returns `42`. `ptr` is left null.

**Example 2: Move capture with `mutable`**

```cpp
#include <iostream>
#include <memory>

int main() {
    auto ptr = std::make_unique<int>(10);
    auto f = [p = std::move(ptr)]() mutable {
        ++(*p);
        return *p;
    };
    std::cout << f() << ' ' << f() << '\n';
}
```

**Expected Output**
```
11 12
```

**Why this output?**  
`p` is a move-captured `unique_ptr`. `mutable` allows `*p` to be modified through the pointer (the pointer itself is not `const` inside a `mutable` lambda).

### Real-World Cases

- **Asynchronous ownership transfer** – moving a `unique_ptr` into a lambda passed to `std::thread`.
- **Deferred cleanup** – capturing a resource by move and releasing it later.
- **Pipeline transformations** – moving large objects through lambda-based pipelines.

---

## 7. Init Capture

### Definitions

**Core Definition**
Init capture (C++14) is a capture form that allows a **new variable** to be introduced in the capture list and initialized with an arbitrary expression, enabling move capture, renaming, and computed captures.

**Technical Definition**
An init-capture has the form `[identifier = expression]` or `[&identifier = expression]`. It declares a new variable (the *init-capture*) whose type is deduced from the expression (as if by `auto`), and whose initializer is the expression. The init-capture inhabits the scope of the lambda body. This is the mechanism that enables capture by move (`[p = std::move(ptr)]`) and capture with a computed value (`[x = a + b]`).

**Beginner-Friendly Explanation**
Init capture lets you **create new variables on the fly** inside the capture list. You can rename an existing variable, compute a value, or move an object in. It is the most flexible capture form.

### Purposes

- To capture by move (`[p = std::move(ptr)]`).
- To rename captured variables for clarity inside the lambda.
- To compute and capture derived values (`[sum = a + b]`).
- To capture a single member without capturing the whole `this` (`[x = member]`).
- To capture references with new names (`[&r = x]`).

### Syntax Rules and Structure

**General Syntax**
```cpp
[x = expr]        // new member x initialized from expr
[&x = expr]       // new reference x bound to expr
[x = std::move(y)] // move capture
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `x` | The name of the new member inside the lambda. |
| `= expr` | The initializer; type is deduced as `auto`. |
| `&x = expr` | A reference member bound to `expr`. |

**Syntax Rules**
- The initializer is evaluated at the point of lambda creation.
- The type of the init-capture is deduced using the rules of `auto`.
- An init-capture cannot have the same name as an explicit capture in the same capture list.
- `[&x = expr]` creates a reference member; the referent must outlive the lambda.
- Init-captures do not require the original variable to be named; they can capture arbitrary expressions.

**Constraints and Limitations**
- The initializer must be a valid expression in the enclosing scope.
- Reference init-captures (`[&x = expr]`) can dangle if the referent dies.
- Init-capture of a bit-field is not allowed.

### Annotated Code Examples

**Example 1: Renaming and computing**

```cpp
#include <iostream>

int main() {
    int a = 3, b = 4;
    auto f = [sum = a + b] { return sum; };   // computed capture
    std::cout << f() << '\n';
}
```

**Expected Output**
```
7
```

**Why this output?**  
`sum` is a new member initialized with `a + b = 7`. The lambda returns `sum`.

**Example 2: Reference init-capture**

```cpp
#include <iostream>

int main() {
    int x = 10;
    auto f = [&r = x] { r += 5; };   // reference to x, named r
    f();
    std::cout << x << '\n';
}
```

**Expected Output**
```
15
```

**Why this output?**  
`r` is a reference to `x`; `r += 5` modifies `x`.

### Real-World Cases

- **Move capture** – `[p = std::move(ptr)]` (see §6).
- **Capturing a class member by value** – `[value = this->member]` instead of `[this]`.
- **Precomputing values** – `[threshold = compute_threshold()]` for expensive computations done once.

---

## 8. `mutable` Lambdas

### Definitions

**Core Definition**
A `mutable` lambda is a lambda whose function call operator is **not `const`** , allowing it to modify the objects captured by copy inside the lambda body.

**Technical Definition**
By default, the closure type’s `operator()` is `const`. This means that members corresponding to by-copy captures are `const` inside the lambda body, and non-`const` member functions cannot be called on them. The `mutable` specifier removes this `const` qualification from `operator()`, allowing modification of by-copy captures. It does **not** affect by-reference captures (the referent can always be modified through a reference).

**Beginner-Friendly Explanation**
Normally, a lambda is **read-only** with respect to its copied variables. If you want to change those copies, you add `mutable` after the parameter list. The original variables outside the lambda are never affected; only the lambda’s internal copies change.

### Purposes

- To allow stateful lambdas that maintain internal counters or accumulators.
- To modify by-copy captures without affecting the original.
- To call non-`const` member functions on captured objects.
- To implement generator-like behavior with retained state.

### Syntax Rules and Structure

**General Syntax**
```cpp
[captures](params) mutable { body }
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `mutable` | Specifier placed after the parameter list (and before `noexcept`/return type). |

**Syntax Rules**
- `mutable` applies to the closure type’s `operator()`.
- By-copy captures become non-`const` members and can be modified.
- By-reference captures are unaffected; the referent can always be modified.
- `mutable` does not change the lambda’s type or signature.
- A `mutable` lambda cannot be called on a `const` closure object.

**Constraints and Limitations**
- The original captured variables are never modified by `mutable`.
- `mutable` lambdas cannot be used where a `const` callable is required (e.g., some algorithms may copy the lambda and call `operator()` on a `const` copy).

### Annotated Code Examples

**Example 1: Stateful counter**

```cpp
#include <iostream>

int main() {
    auto counter = [n = 0]() mutable { return ++n; };
    std::cout << counter() << ' ' << counter() << ' ' << counter() << '\n';
}
```

**Expected Output**
```
1 2 3
```

**Why this output?**  
`n` is an init-capture. `mutable` allows `++n` to modify the closure’s copy of `n`. Each call increments the stored value.

**Example 2: Modifying a by-copy capture**

```cpp
#include <iostream>

int main() {
    int x = 5;
    auto f = [x]() mutable { x *= 2; return x; };
    std::cout << f() << ' ' << f() << '\n';
    std::cout << x << '\n';   // original unchanged
}
```

**Expected Output**
```
10 20
5
```

**Why this output?**  
The lambda holds a copy of `x`. `mutable` allows the copy to be modified. The original `x` stays `5`.

### Real-World Cases

- **Stateful predicates** – counting how many times a predicate was called.
- **Generators** – lambdas that produce a sequence of values.
- **Accumulators in algorithms** – modifying a copy-captured accumulator (though reference capture is usually preferred).

---

## 9. Generic Lambdas

### Definitions

**Core Definition**
A generic lambda (C++14) is a lambda whose parameters are declared with `auto`, making the closure’s `operator()` a **function template**.

**Technical Definition**
When one or more parameters of a lambda are declared with `auto`, the lambda is a *generic lambda*, and its `operator()` is a template. Each `auto` parameter corresponds to a template parameter; the type is deduced at the call site according to the rules of template argument deduction. This is equivalent to an abbreviated function template.

**Beginner-Friendly Explanation**
A normal lambda has fixed parameter types (e.g., `int`). A generic lambda uses `auto`, so it can accept **any type** — `int`, `double`, `std::string`, etc. The compiler generates a template behind the scenes.

### Purposes

- To write a single lambda that works with multiple argument types.
- To avoid duplicating lambdas for each type.
- To support generic algorithms and type-agnostic operations.
- To enable perfect forwarding with `auto&&` parameters.

### Syntax Rules and Structure

**General Syntax**
```cpp
[](auto params) { body }
[](auto&& params) { body }   // forwarding reference
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `auto` | Placeholder for a deduced template parameter. |
| `auto&&` | Forwarding reference (preserves value category). |

**Syntax Rules**
- Each `auto` parameter introduces a separate template parameter.
- The lambda’s `operator()` becomes a template member function.
- Return type deduction applies as usual.
- Generic lambdas can be combined with `mutable`.

**Constraints and Limitations**
- Generic lambdas cannot have explicit template arguments at the call site (until C++20 templated lambdas provide named template parameters).
- The `auto` parameters follow template argument deduction rules; `auto&&` is a forwarding reference.

### Annotated Code Examples

**Example 1: Generic addition**

```cpp
#include <iostream>
#include <string>

int main() {
    auto add = [](auto a, auto b) { return a + b; };
    std::cout << add(2, 3) << '\n';          // int + int
    std::cout << add(1.5, 2.5) << '\n';      // double + double
    std::cout << add<std::string>("a", "b") << '\n'; // explicit template args (C++20)
}
```

**Expected Output**
```
5
4
ab
```

**Why this output?**  
`add` is a generic lambda; its `operator()` is a template that deduces `a` and `b` types. The same lambda works for `int`, `double`, and `std::string`.

**Example 2: Forwarding reference in a generic lambda**

```cpp
#include <iostream>
#include <string>
#include <utility>

int main() {
    auto forward_to = [](auto&& x) { return std::forward<decltype(x)>(x); };
    std::string s = "hello";
    std::cout << forward_to(s) << '\n';          // lvalue
    std::cout << forward_to(std::string("world")) << '\n'; // rvalue
}
```

**Expected Output**
```
hello
world
```

**Why this output?**  
`auto&&` is a forwarding reference; `std::forward` preserves the value category. The lambda accepts both lvalues and rvalues efficiently.

### Real-World Cases

- **STL algorithms** – `std::transform(v.begin(), v.end(), out.begin(), [](auto x){ return x * 2; });`.
- **Logging utilities** – `auto log = [](auto&& msg){ std::cout << msg << '\n'; };`.
- **Generic visitors** – combining with `std::variant` visitation.

---

## 10. Templated Lambdas

### Definitions

**Core Definition**
A templated lambda (C++20) is a lambda that declares an **explicit template parameter list** between the capture list and the parameter list, giving names to its template parameters.

**Technical Definition**
Since C++20, a lambda expression may have a template parameter list introduced by `<>` after the capture list. Such a lambda is always a generic lambda, and the template parameters are used by the closure type’s `operator()`. This allows explicit type parameters (`typename T`), non-type parameters (`int N`), and constrained parameters (`requires`), enabling more expressive and constrained generic lambdas.

**Beginner-Friendly Explanation**
A generic lambda uses `auto`, but you cannot name the type. A templated lambda lets you **give the type a name** (`<typename T>`) so you can use it explicitly in the body, or impose constraints. It is the lambda equivalent of a function template.

### Purposes

- To name template parameters for use inside the lambda body.
- To apply constraints (`requires`) to lambda parameters.
- To use non-type template parameters (e.g., `int N`).
- To provide more precise interfaces than `auto` alone.

### Syntax Rules and Structure

**General Syntax**
```cpp
[captures]<tparams>(params) { body }
[captures]<tparams>(params) requires constraint { body }
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `<tparams>` | Template parameter list (e.g., `<typename T>`, `<int N>`). |
| `requires constraint` | Optional constraint on the template parameters. |

**Syntax Rules**
- The template parameter list appears after `[]` and before `()`.
- The lambda is always generic.
- Template parameters can be used in the parameter list and body.
- Constraints can be applied with `requires`.

**Constraints and Limitations**
- Explicit template arguments cannot be provided at the call site directly (unlike function templates); they are deduced.
- Templated lambdas require C++20.

### Annotated Code Examples

**Example 1: Named type parameter**

```cpp
#include <iostream>
#include <vector>

int main() {
    auto print_size = []<typename T>(const std::vector<T>& v) {
        std::cout << v.size() << '\n';
    };
    std::vector<int> vi = {1, 2, 3};
    std::vector<double> vd = {1.0, 2.0};
    print_size(vi);   // 3
    print_size(vd);   // 2
}
```

**Expected Output**
```
3
2
```

**Why this output?**  
`T` is named explicitly and used in the parameter type `const std::vector<T>&`. The lambda works for any `T`.

**Example 2: Constrained templated lambda**

```cpp
#include <iostream>
#include <concepts>

int main() {
    auto only_integral = []<std::integral T>(T x) { return x * 2; };
    std::cout << only_integral(5) << '\n';      // OK: int is integral
    // std::cout << only_integral(3.14);        // error: double is not integral
}
```

**Expected Output**
```
10
```

**Why this output?**  
The `requires`-style constraint `std::integral T` restricts `T` to integral types. The `double` call would be rejected at compile time.

### Real-World Cases

- **Constrained generic algorithms** – `[]<std::integral T>(T x){ ... }`.
- **Compile-time constants** – `[]<int N>(){ return N; }`.
- **Type-aware logging** – `[]<typename T>(T x){ std::cout << typeid(T).name(); }`.

---

## 11. Lambda Return Type

### Definitions

**Core Definition**
The return type of a lambda is the type of the value produced by its `operator()`; it is either **deduced** from the `return` statements or **explicitly specified** with a trailing return type.

**Technical Definition**
If a lambda has no trailing return type, its return type is deduced from its `return` statements, as if the function were declared with `auto`. All `return` statements must yield the same type (after deduction); otherwise, the program is ill-formed. If a trailing return type is provided (`-> ret`), the return type is exactly `ret`, and all `return` statements must be convertible to `ret`.

**Beginner-Friendly Explanation**
Most of the time you don’t need to write the return type — the compiler figures it out from what you `return`. If you need a specific type (e.g., a reference or a `long`), you can write `-> type` after the parameter list.

### Purposes

- To let the compiler deduce the return type automatically (concise code).
- To specify an exact return type when deduction would be wrong or ambiguous.
- To return references or `void` explicitly.
- To make the lambda’s interface self-documenting.

### Syntax Rules and Structure

**General Syntax**
```cpp
[captures](params) { return expr; }             // deduced
[captures](params) -> ret { return expr; }      // explicit
```

**Component Breakdown**

| Form | Meaning |
|------|---------|
| No `->` | Deduce from `return` statements. |
| `-> ret` | Explicit trailing return type. |

**Syntax Rules**
- Without an explicit return type, the lambda’s return type is deduced as if by `auto`.
- All `return` statements must be compatible under deduction rules.
- A lambda with no `return` statement has return type `void`.
- `-> auto&` can be used to deduce a reference return type.

**Constraints and Limitations**
- Return type deduction for lambdas follows the same rules as `auto` return type deduction for functions.
- If multiple `return` statements yield different types, deduction fails.
- `-> void` can be used to explicitly discard a value.

### Annotated Code Examples

**Example 1: Deduced return type**

```cpp
#include <iostream>

int main() {
    auto square = [](int x) { return x * x; };   // deduced as int
    std::cout << square(5) << '\n';
}
```

**Expected Output**
```
25
```

**Why this output?**  
`x * x` is an `int`, so the lambda’s return type is deduced as `int`.

**Example 2: Explicit return type**

```cpp
#include <iostream>

int main() {
    auto divide = [](double a, double b) -> double { return a / b; };
    std::cout << divide(7.0, 2.0) << '\n';
}
```

**Expected Output**
```
3.5
```

**Why this output?**  
The trailing `-> double` explicitly sets the return type, preventing any deduction surprises.

**Example 3: Deduced reference return**

```cpp
#include <iostream>
#include <vector>

int main() {
    std::vector<int> v = {1, 2, 3};
    auto front_ref = [&v]() -> auto& { return v.front(); };
    front_ref() = 99;
    std::cout << v[0] << '\n';
}
```

**Expected Output**
```
99
```

**Why this output?**  
`-> auto&` deduces a reference return type, so assigning to `front_ref()` modifies `v[0]`.

### Real-World Cases

- **STL algorithm predicates** – return `bool` (often deduced).
- **Transformations** – return a computed value (deduced).
- **Getter lambdas** – return references explicitly with `-> auto&`.

---

## 12. Lambdas in STL Algorithms

### Definitions

**Core Definition**
Lambdas are commonly passed to STL algorithms as **predicates**, **comparators**, and **operations**, providing inline, customized behavior without writing separate function objects.

**Technical Definition**
STL algorithms in `<algorithm>` and `<numeric>` accept callable objects (function pointers, function objects, or lambdas) for customization. A lambda passed to an algorithm is constructed at the call site and stored (by value) inside the algorithm’s implementation. The algorithm may copy the lambda multiple times; therefore, mutable state in the lambda may not behave as expected unless captured by reference. The C++ Core Guidelines recommend capturing by reference for lambdas used locally (F.52) and avoiding by-reference captures for lambdas used non-locally (F.53).

**Beginner-Friendly Explanation**
Lambdas and STL algorithms are best friends. Instead of writing a loop with an `if` inside, you write `std::find_if(v.begin(), v.end(), [](int x){ return x > 10; });`. The lambda tells the algorithm **what to do**, and the algorithm handles the iteration. It makes code shorter and less error-prone.

### Purposes

- To customize algorithm behavior inline (`std::sort`, `std::find_if`, `std::transform`, `std::for_each`).
- To replace hand-written loops with expressive, declarative code.
- To capture context (thresholds, accumulators) directly in the predicate.
- To improve readability and reduce boilerplate.

### Common Algorithm Patterns

| Algorithm | Lambda Role | Example |
|-----------|-------------|---------|
| `std::sort` | Comparator | `[](int a, int b){ return a > b; }` |
| `std::find_if` | Predicate | `[threshold](int x){ return x > threshold; }` |
| `std::transform` | Operation | `[](int x){ return x * 2; }` |
| `std::for_each` | Operation | `[&sum](int x){ sum += x; }` |
| `std::count_if` | Predicate | `[](int x){ return x % 2 == 0; }` |
| `std::remove_if` | Predicate | `[&](const auto& x){ return x.is_dead(); }` |
| `std::accumulate` | Binary operation | `[](int acc, int x){ return acc + x; }` |

**Syntax Rules**
- The lambda must satisfy the algorithm’s requirements (e.g., predicate must return `bool`).
- The algorithm may copy the lambda; avoid relying on mutable state unless captured by reference and the algorithm is known to use a single copy.
- For local use (algorithm call within the same scope), capture by reference is efficient and safe (`[&]`).
- For non-local use (storing the lambda, passing to another thread), capture by value (`[=]` or explicit) to avoid dangling references.

### Annotated Code Examples

**Example 1: `std::sort` with a lambda comparator**

```cpp
#include <algorithm>
#include <iostream>
#include <vector>

int main() {
    std::vector<int> v = {3, 1, 4, 1, 5};
    std::sort(v.begin(), v.end(), [](int a, int b) { return a > b; });
    for (int x : v) std::cout << x << ' ';
    std::cout << '\n';
}
```

**Expected Output**
```
5 4 3 1 1
```

**Why this output?**  
The lambda comparator `a > b` sorts in descending order. `std::sort` uses the lambda to compare elements.

**Example 2: `std::transform` with a capturing lambda**

```cpp
#include <algorithm>
#include <iostream>
#include <vector>

int main() {
    std::vector<int> in = {1, 2, 3, 4};
    std::vector<int> out(in.size());
    int factor = 10;
    std::transform(in.begin(), in.end(), out.begin(),
                   [factor](int x) { return x * factor; });
    for (int x : out) std::cout << x << ' ';
    std::cout << '\n';
}
```

**Expected Output**
```
10 20 30 40
```

**Why this output?**  
The lambda captures `factor` by value and multiplies each element by it. `std::transform` applies the lambda to each element.

**Example 3: `std::for_each` with a reference capture**

```cpp
#include <algorithm>
#include <iostream>
#include <vector>

int main() {
    std::vector<int> v = {1, 2, 3, 4};
    int sum = 0;
    std::for_each(v.begin(), v.end(), [&sum](int x) { sum += x; });
    std::cout << sum << '\n';
}
```

**Expected Output**
```
10
```

**Why this output?**  
`sum` is captured by reference; the lambda modifies it for each element. `std::for_each` calls the lambda once per element.

### Real-World Cases

- **Filtering** – `std::copy_if(in.begin(), in.end(), out.begin(), [](int x){ return x > 0; });`.
- **Custom ordering** – sorting records by a field with a capturing comparator.
- **Aggregation** – `std::accumulate` with a lambda that applies a custom formula.
- **Parallel algorithms** – `std::for_each(std::execution::par, ...)` with lambdas (capture by value for thread safety).

---

## 13. Lambda Best Practices

### Definitions

**Core Definition**
Lambda best practices are guidelines for writing lambdas that are correct, readable, safe (especially regarding lifetime), and efficient.

**Technical Definition**
These practices are anchored in the C++ Core Guidelines, particularly **F.52** (“Prefer capturing by reference in lambdas that will be used locally, including passed to algorithms”), **F.53** (“Avoid capturing by reference in lambdas that will be used non-locally, including returned, stored on the heap, or passed to another thread”), and **F.54** (“If you capture `this`, capture all variables explicitly (no default capture)”), along with CERT EXP61-CPP (lambda must not outlive its reference-captured objects).

**Beginner-Friendly Explanation**
Lambda best practices are simple rules: **capture by reference only when the lambda is short-lived**, **capture by value when the lambda might outlive the current scope**, **be explicit about what you capture** (avoid `[=]` and `[&]` for non-local lambdas), **don’t let a lambda outlive the variables it references**, and **keep lambdas short and focused**.

### Key Guidelines (Summary Table)

| Guideline | Summary | Recommendation |
|-----------|---------|----------------|
| **F.52** | Prefer capturing by reference in lambdas used locally, including passed to algorithms. | Use `[&]` or explicit `&` captures for algorithm calls. |
| **F.53** | Avoid capturing by reference in lambdas used non-locally (returned, stored, passed to another thread). | Use value captures or init-captures. |
| **F.54** | If you capture `this`, capture all variables explicitly. | Avoid `[=]` when `this` is involved; use `[this, x, y]`. |
| **EXP61-CPP** | A lambda must not outlive any of its reference-captured objects. | Capture by copy if the lambda escapes the scope. |
| **Prefer explicit captures** | Implicit capture-defaults can capture more than intended. | List captures explicitly for non-trivial lambdas. |
| **Keep lambdas short** | Long lambdas hurt readability. | If a lambda exceeds ~5 lines, consider a named function. |
| **Prefer `auto` parameters for genericity** | Generic lambdas reduce duplication. | Use `auto`/`auto&&` when types vary. |
| **Be careful with `mutable` and algorithms** | Algorithms may copy the lambda; mutable state may not persist. | Prefer reference capture for accumulators, or use `std::ref`. |

### Annotated Code Examples

**Example 1: F.52 — capture by reference for local algorithm use**

```cpp
#include <algorithm>
#include <iostream>
#include <vector>

int main() {
    std::vector<int> v = {1, 2, 3, 4, 5};
    int threshold = 3;
    // GOOD: lambda used locally in the same scope → capture by reference
    auto it = std::find_if(v.begin(), v.end(),
                           [&threshold](int x) { return x > threshold; });
    std::cout << *it << '\n';
}
```

**Expected Output**
```
4
```

**Why this is best practice?**  
The lambda is used immediately within `std::find_if`, so the reference to `threshold` cannot dangle. Capturing by reference avoids copying.

**Example 2: F.53 — capture by value for a returned lambda**

```cpp
#include <iostream>
#include <functional>

std::function<int()> make_counter(int start) {
    // GOOD: capture by value; the lambda may outlive `start`
    return [start]() mutable { return ++start; };
}

int main() {
    auto counter = make_counter(10);
    std::cout << counter() << ' ' << counter() << '\n';
}
```

**Expected Output**
```
11 12
```

**Why this is best practice?**  
The lambda is returned from the function, so capturing `start` by reference would dangle. Capturing by value (with `mutable`) safely transfers the state into the closure.

**Example 3: F.54 — capture `this` and variables explicitly**

```cpp
#include <iostream>

struct Widget {
    int value = 10;
    auto get_lambda() {
        int local = 5;
        // GOOD: explicit captures, including this
        return [this, local] { return value + local; };
    }
};

int main() {
    Widget w;
    auto f = w.get_lambda();
    std::cout << f() << '\n';
}
```

**Expected Output**
```
15
```

**Why this is best practice?**  
When `this` is captured, using a default capture like `[=]` can silently capture `this` (and since C++20 this is deprecated). Explicitly listing `this` and `local` makes the captures clear and avoids surprises.

**Example 4: Avoiding dangling references (EXP61-CPP)**

```cpp
#include <iostream>
#include <functional>

// BAD: returning a lambda that captures a local by reference
// std::function<int()> bad() {
//     int x = 42;
//     return [&x] { return x; };   // dangling reference
// }

// GOOD: capture by value
std::function<int()> good() {
    int x = 42;
    return [x] { return x; };        // safe copy
}
```

**Expected Output**  
(The `good` version compiles and works; the commented `bad` version would have undefined behavior.)

**Why this matters?**  
Returning a reference-capturing lambda that outlives the local is a classic bug. Capturing by value ensures the closure owns its data.

### Real-World Cases

- **STL algorithm calls** – follow F.52 for local predicates and comparators.
- **Asynchronous tasks** – follow F.53 for lambdas passed to `std::thread`, `std::async`, or stored in `std::function`.
- **Member callbacks** – follow F.54 to avoid implicit `this` capture.
- **Code review** – CERT EXP61-CPP and Core Guidelines F.52–F.54 are common review checkpoints.

---

## References

- Lambda expressions (cppreference) – https://en.cppreference.com/w/cpp/language/lambda
- C++ Core Guidelines — F.52: Prefer capturing by reference in lambdas that will be used locally, including passed to algorithms – https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rf-reference-capture
- C++ Core Guidelines — F.53: Avoid capturing by reference in lambdas that will be used non-locally – https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rf-value-capture
- C++ Core Guidelines — F.54: If you capture `this`, capture all variables explicitly – https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rf-this-capture
- CERT C++ EXP61-CPP: A lambda object must not outlive any of its reference captured objects – https://wiki.sei.cmu.edu/confluence/display/cplusplus/EXP61-CPP.+A+lambda+object+must+not+outlive+any+of+its+reference+captured+objects
- SonarSource rule RSPEC-5495: Capture by reference in lambdas used locally – https://rules.sonarsource.com/cpp/rspec-5495/
- Lambda expressions (Stack Overflow canonical answer on syntax and captures) – https://stackoverflow.com/revisions/7627330/4
- Generic lambdas (cppreference) – https://en.cppreference.com/w/cpp/language/lambda#Generic_lambdas
- Templated lambdas (cppreference, C++20 explicit template parameter list) – https://en.cppreference.com/w/cpp/language/lambda#Lambda_expressions_with_an_explicit_template_parameter_list
- `std::function` (cppreference) – https://en.cppreference.com/w/cpp/utility/functional/function

---

This cheat sheet provides a complete, structured reference for C++ lambda expressions, covering syntax, capture lists, capture by value/reference/move, init-capture, `mutable` lambdas, generic and templated lambdas, return type deduction, STL algorithm usage, and best practices. Each section follows a uniform template with definitions, purposes, syntax rules, annotated examples, expected outputs, and authoritative references.