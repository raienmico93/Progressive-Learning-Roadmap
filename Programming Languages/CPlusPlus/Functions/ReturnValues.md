# Return Values — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
A function’s return value is the result that the function produces and passes back to the caller when its execution completes. The mechanism by which this result is delivered — as a copy, an alias, an address, or a movable resource — is the function’s **return category**.

**Technical Definition**
The return mechanism is determined by the function’s declared return type. For a return type `T`, the function may return by value (`T`), by lvalue reference (`T&`), by rvalue reference (`T&&`), or by pointer (`T*`). The `return` statement initializes a result object; since C++17, the initialization semantics for prvalues have been redefined so that certain copies are not merely optimized away but **never exist conceptually** (“guaranteed copy elision”). For reference and pointer returns, lifetime and ownership are critical concerns; returning a reference or pointer to a local automatic object is always an error.

**Beginner-Friendly Explanation**
Returning a value is how a function “hands back” its result. You can hand back a **copy** (safe but possibly expensive), a **direct alias** (efficient but risky if the original disappears), a **memory address** (powerful but requires careful ownership management), or you can return **nothing at all** (`void`). Modern C++ is designed so that returning by value is usually the simplest and fastest option, because the compiler can often construct the result directly where the caller wants it.

### Key Characteristics

- **By value** is the default and safest mechanism; modern C++ makes it efficient through copy elision and move semantics.
- **By reference** provides an alias; it is appropriate for returning members or persistent objects, but **never** local automatics.
- **By pointer** returns an address; it expresses optionality (`nullptr`) and is common in C-style APIs, but ownership must be unambiguous.
- **By rvalue reference** is almost never used for returning from ordinary functions; its legitimate uses are confined to `std::move`, `std::forward`, and rvalue-ref-qualified member functions.
- **Copy elision** (RVO/NRVO) eliminates copies; **C++17 guaranteed copy elision** makes certain elisions mandatory and redefines prvalue semantics.
- **`return std::move(local)`** is usually an anti-pattern that **disables** NRVO.

### Prerequisites

- Understanding of value categories (lvalues, xvalues, prvalues).
- Familiarity with copy/move constructors and move semantics.
- Knowledge of object lifetime and storage duration.
- Awareness of the difference between the stack (automatic storage) and the heap (dynamic storage).

### Related Programming Areas

- **Move semantics** – rvalue references, move constructors, `std::move`.
- **Copy elision** – RVO, NRVO, guaranteed elision (C++17).
- **API design** – choosing return categories for library interfaces.
- **Exception safety** – return values and the strong exception guarantee.
- **Resource management** – ownership and smart pointers.

### Core Concepts / Features

1. Return by Value
2. Return by Reference
3. Return by Pointer
4. Return by Rvalue Reference
5. Return Value Optimization (RVO)
6. Named Return Value Optimization (NRVO)
7. Copy Elision
8. Guaranteed Copy Elision (C++17)
9. Return Best Practices

---

## 1. Return by Value

### Definitions

**Core Definition**
Returning by value means the function produces a result object that is **copy-initialized** (or move-initialized) from the returned expression, and the caller receives an independent object.

**Technical Definition**
For a function with return type `T` (non-reference, non-pointer), the `return` statement initializes a **result object** of type `T`. Before C++17, this initialization conceptually involved a copy or move from the returned expression to the result object; copy elision could eliminate it. Since C++17, when the operand is a prvalue, the result object is initialized directly from the prvalue, and no temporary exists. For named local variables (NRVO), a move is performed automatically if the type is movable (the return statement treats the local as an rvalue for overload resolution).

**Beginner-Friendly Explanation**
Returning by value gives the caller their own **personal copy** of the result. In modern C++, this is usually not slow, because the compiler constructs the result directly in the caller’s storage (or moves it cheaply). It is the safest and most idiomatic way to return a result.

### Purposes

- To provide the caller with an independent object that owns its resources.
- To avoid lifetime and aliasing hazards associated with references and pointers.
- To enable the compiler to apply copy elision and move semantics for efficiency.
- To support the natural expression of value-producing functions (arithmetic, factories, transformations).

### Syntax Rules and Structure

**General Syntax**
```cpp
T function_name(parameters) {
    // ...
    return expression;   // expression convertible to T
}
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `T` | The return type (no `&` or `*`). |
| `return expression;` | Initializes the result object from `expression`. |

**Syntax Rules**
- The returned expression must be convertible to `T`.
- If `T` is a class type, a copy or move constructor is used (or elided).
- The result object has automatic storage duration in the caller’s context.
- Since C++17, returning a prvalue of type `T` directly initializes the result; no copy/move constructor is required to be accessible.

**Constraints and Limitations**
- Returning by value copies (or moves) the object; for large, expensive-to-copy objects, this can be costly if elision or move is not available.
- Slicing occurs if a derived-class object is returned as a base-class value.
- Abstract class types cannot be returned by value.

### Annotated Code Examples

**Example 1: Returning a fundamental type by value**

```cpp
#include <iostream>

int square(int x) {
    return x * x;          // result object is an int
}

int main() {
    int a = square(5);
    std::cout << a << '\n';
}
```

**Expected Output**
```
25
```

**Why this output?**  
`square(5)` computes `25` and returns it by value; `a` is an independent `int`.

**Example 2: Returning a class type by value**

```cpp
#include <iostream>
#include <string>

std::string make_greeting(const std::string& name) {
    return "Hello, " + name + "!";   // returns a temporary std::string by value
}

int main() {
    std::string g = make_greeting("Alice");
    std::cout << g << '\n';
}
```

**Expected Output**
```
Hello, Alice!
```

**Why this output?**  
The temporary `std::string` is returned by value; since C++17, it is constructed directly into `g` without any copy or move.

**Example 3: Returning a local named variable (NRVO candidate)**

```cpp
#include <iostream>
#include <string>

std::string build_message() {
    std::string msg = "Hello, world!";
    return msg;            // NRVO: msg may be constructed directly in caller's storage
}

int main() {
    std::string s = build_message();
    std::cout << s << '\n';
}
```

**Expected Output**
```
Hello, world!
```

**Why this output?**  
`msg` is a named local; NRVO may elide the copy/move entirely, constructing `msg` directly in `s`. Even if NRVO is not applied, the return statement treats `msg` as an rvalue and moves it into `s` (no deep copy).

### Real-World Cases

- **Factory functions** – `std::make_unique<T>(...)`, `std::make_shared<T>(...)`.
- **Mathematical functions** – `std::sqrt`, `std::pow`, trigonometric functions.
- **String manipulation** – `std::string::substr`, `std::to_string`.
- **Container algorithms** – functions returning transformed sequences by value (e.g., `std::vector` results).

---

## 2. Return by Reference

### Definitions

**Core Definition**
Returning by reference means the function returns an **alias** to an existing object; no copy is made, and the caller can access (and potentially modify) the original object through the returned reference.

**Technical Definition**
A function with return type `T&` returns an lvalue reference. The returned reference must refer to an object that **outlives the function call**; returning a reference to a local automatic object, a temporary, or a function parameter results in a **dangling reference** (undefined behavior). The lifetime of a temporary bound to a returned reference is **not** extended; the temporary is destroyed at the end of the return expression.

**Beginner-Friendly Explanation**
Returning by reference is like giving the caller a **nickname** for an existing object. It is efficient (no copy), and it can be used to allow modification. But it is dangerous if the object the nickname refers to no longer exists — then the nickname points to nothing valid.

### Purposes

- To avoid copying large objects while still allowing the caller to access them.
- To enable functions to return references to class members (e.g., `operator[]`).
- To support operator chaining (e.g., `operator<<` returning `std::ostream&`).
- To allow functions to act as lvalues (e.g., assignment through a returned reference).

### Syntax Rules and Structure

**General Syntax**
```cpp
T& function_name(parameters) {
    // ...
    return lvalue_expression;   // must refer to an object that outlives the call
}
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `T&` | The return type: an lvalue reference to `T`. |
| `return lvalue;` | Must return an lvalue that persists after the function returns. |

**Syntax Rules**
- The returned reference must refer to an object with **static storage duration**, **thread storage duration**, **dynamic storage duration**, or an object that the caller already owns (e.g., a member of an object passed by reference).
- Returning a reference to a **local automatic variable** is always a logical error (dangling reference).
- Returning a reference to a **temporary** is always a dangling reference; lifetime extension does not cross function boundaries.
- `const T&` returns a read-only alias.

**Constraints and Limitations**
- **Never** return a reference to a local variable or a temporary.
- Returning a reference to a member of a by-value parameter is also dangerous (the parameter is destroyed).
- References cannot be null; if absence is possible, use a pointer or `std::optional`.

### Annotated Code Examples

**Example 1: Returning a reference to a persistent object (safe)**

```cpp
#include <iostream>
#include <vector>

int& element_at(std::vector<int>& v, std::size_t index) {
    return v[index];          // safe: v outlives the call
}

int main() {
    std::vector<int> data = {10, 20, 30};
    element_at(data, 1) = 99; // modifies data[1] through the reference
    std::cout << data[1] << '\n';
}
```

**Expected Output**
```
99
```

**Why this output?**  
The returned reference aliases `data[1]`; assigning to it modifies the vector element.

**Example 2: Returning a reference to a static object (safe)**

```cpp
#include <iostream>

int& counter() {
    static int count = 0;
    return count;             // safe: static storage duration
}

int main() {
    counter() = 5;            // modifies the static variable
    std::cout << counter() << '\n';
    ++counter();
    std::cout << counter() << '\n';
}
```

**Expected Output**
```
5
6
```

**Why this output?**  
`count` has static storage duration, so the returned reference remains valid across calls.

**Example 3: Dangling reference (undefined behavior — illustrative only)**

```cpp
// DO NOT WRITE THIS CODE
int& bad() {
    int x = 42;
    return x;                 // x is destroyed when bad() returns
}

int main() {
    int& r = bad();           // r is dangling
    // std::cout << r;        // undefined behavior
}
```

**Expected Output**  
(Undefined behavior; may print garbage or crash.)

**Why this is wrong?**  
`x` is a local automatic variable; its lifetime ends when `bad()` returns, leaving `r` as a dangling reference.

### Real-World Cases

- **Container element access** – `std::vector::operator[]`, `std::map::operator[]` return references to elements.
- **Stream operators** – `operator<<(std::ostream&, const T&)` returns `std::ostream&` for chaining.
- **Class member access** – getters returning `const T&` to avoid copying members.
- **Singleton access** – functions returning references to static instances.

---

## 3. Return by Pointer

### Definitions

**Core Definition**
Returning by pointer means the function returns a **copy of a pointer value** (an address); the caller receives an address that may refer to an existing object, a dynamically allocated object, or `nullptr`.

**Technical Definition**
A function with return type `T*` returns a pointer by value. The pointer itself is an object (the returned value); it may point to an object with static, thread, dynamic, or automatic storage duration. Returning a pointer to a **local automatic variable** produces a **dangling pointer** (undefined behavior if dereferenced). Returning a pointer to **dynamically allocated memory** transfers (at least nominally) the responsibility of deallocation to the caller, creating ownership ambiguity unless documented or wrapped in a smart pointer.

**Beginner-Friendly Explanation**
Returning a pointer is like handing the caller a **map** to a location in memory. The map itself is a small copy, but following it leads to the actual object. The caller must know whether the map is still valid and who is responsible for “cleaning up” the location.

### Purposes

- To express **optionality**: `nullptr` indicates “no object.”
- To return addresses of dynamically allocated objects (though smart pointers are preferred).
- To interface with C APIs that return pointers.
- To return addresses of static or caller-owned objects without using reference syntax.

### Syntax Rules and Structure

**General Syntax**
```cpp
T* function_name(parameters) {
    // ...
    return pointer_expression;   // may be nullptr
}
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `T*` | The return type: a pointer to `T`. |
| `return ptr;` | Returns a copy of the pointer value. |

**Syntax Rules**
- The returned pointer must point to an object that remains valid after the function returns, **or** be `nullptr`.
- Returning `&local` (address of a local automatic) is always a dangling pointer.
- If the pointer is to dynamically allocated memory, ownership must be clearly documented (prefer `std::unique_ptr<T>` or `std::shared_ptr<T>` to raw pointers).
- The caller must check for `nullptr` before dereferencing if absence is possible.

**Constraints and Limitations**
- Raw pointers do not express ownership; the caller must know whether to `delete`.
- Returning a pointer to a local variable is a common beginner mistake leading to undefined behavior.
- Pointer arithmetic and array decay can obscure size information.

### Annotated Code Examples

**Example 1: Returning a pointer to a static object (safe)**

```cpp
#include <iostream>

int* global_counter() {
    static int count = 0;
    return &count;            // safe: static storage duration
}

int main() {
    int* p = global_counter();
    *p = 10;
    std::cout << *global_counter() << '\n';
}
```

**Expected Output**
```
10
```

**Why this output?**  
`count` has static storage duration; the pointer remains valid, and modifying through it modifies the static variable.

**Example 2: Returning `nullptr` to indicate absence**

```cpp
#include <iostream>

int* find_positive(int* arr, int n) {
    for (int i = 0; i < n; ++i) {
        if (arr[i] > 0) return &arr[i];
    }
    return nullptr;           // no positive element found
}

int main() {
    int data[] = {-3, -1, 0, 5, 7};
    int* p = find_positive(data, 5);
    if (p) {
        std::cout << "Found: " << *p << '\n';
    } else {
        std::cout << "Not found\n";
    }
}
```

**Expected Output**
```
Found: 5
```

**Why this output?**  
The function returns a pointer to the first positive element (the `5`), and the caller checks for `nullptr` before dereferencing.

**Example 3: Dangling pointer (undefined behavior — illustrative only)**

```cpp
// DO NOT WRITE THIS CODE
int* bad() {
    int x = 42;
    return &x;                // x is destroyed when bad() returns
}
// The returned pointer is dangling; dereferencing it is undefined behavior.
```

**Expected Output**  
(Undefined behavior.)

**Why this is wrong?**  
The address of a local automatic variable becomes invalid when the function returns.

### Real-World Cases

- **C standard library** – `strchr`, `strstr` return pointers into the input string (or `nullptr`).
- **Optional lookups** – functions returning `T*` where `nullptr` means “not found.”
- **Legacy APIs** – many C++98-era APIs return raw owning pointers (now discouraged).
- **Interfacing with C** – C functions return pointers by convention.

---

## 4. Return by Rvalue Reference

### Definitions

**Core Definition**
Returning by rvalue reference (`T&&`) means the function returns an rvalue reference to an object; the caller can treat the result as a movable value. In ordinary application code, this is **almost never** the correct return category.

**Technical Definition**
A function with return type `T&&` returns an xvalue (an expiring value). The returned reference must refer to an object whose lifetime extends beyond the function call; otherwise, the caller receives a **dangling rvalue reference**. The legitimate use cases are narrowly defined: `std::move` and `std::forward` return rvalue references by design, and rvalue-ref-qualified member functions may return `T&&` to expose a member for moving from a temporary object. Returning `std::move(local)` from an ordinary function is wrong because it returns a reference to a local that is destroyed, **and** it disables NRVO.

**Beginner-Friendly Explanation**
Returning by rvalue reference is a specialized tool for saying “this object is about to expire, so you can take its resources.” It is easy to misuse: if you return a reference to something that disappears, you get a dangling reference. For normal functions, **do not use this**; return by value instead.

### Purposes

- To implement `std::move` and `std::forward` (the primary legitimate uses).
- To allow rvalue-ref-qualified member functions to return a movable member from a temporary object (advanced idiom).
- To support perfect forwarding of return values in generic wrappers (rarely needed directly).

### Syntax Rules and Structure

**General Syntax**
```cpp
T&& function_name(parameters) {
    // ...
    return std::move(expression);   // almost always suspicious in ordinary code
}
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `T&&` | The return type: an rvalue reference to `T`. |
| `return expr;` | Must refer to an object that outlives the call. |

**Syntax Rules**
- The returned reference must refer to an object with a lifetime that extends beyond the function call.
- Returning a reference to a local automatic, a temporary, or a by-value parameter is a dangling reference.
- Returning `std::move(local)` from an ordinary function is a logic error and inhibits NRVO.
- Legitimate returns of `T&&` typically involve `return std::move(member)` inside an rvalue-ref-qualified member function, where `member` belongs to an object that the caller still owns (but has declared as a temporary).

**Constraints and Limitations**
- **Do not** use `T&&` as a return type for ordinary value-producing functions.
- **Do not** return `std::move(local)`; it prevents copy elision and returns a dangling reference.
- The lifetime of a temporary is **not** extended by binding it to a returned rvalue reference.

### Annotated Code Examples

**Example 1: Legitimate use — rvalue-ref-qualified member function (advanced)**

```cpp
#include <iostream>
#include <string>
#include <utility>

class Container {
    std::string data = "payload";
public:
    // Called on an rvalue Container: expose the member for moving
    std::string&& extract() && {
        return std::move(data);
    }

    // Called on an lvalue Container: copy
    std::string extract() & {
        return data;
    }
};

int main() {
    Container c;
    std::string a = c.extract();            // copies (lvalue)
    std::string b = Container{}.extract();  // moves (rvalue temporary)
    std::cout << a << '\n' << b << '\n';
}
```

**Expected Output**
```
payload
payload
```

**Why this output?**  
On the lvalue `c`, `extract() &` returns a copy. On the temporary `Container{}`, `extract() &&` returns `std::string&&`, and `b` is move-constructed. The temporary `Container` lives until the end of the full expression, so the reference is valid during the move.

**Example 2: Wrong — returning `std::move(local)` (dangling + pessimization)**

```cpp
// DO NOT WRITE THIS CODE
#include <string>
#include <utility>

std::string&& bad() {
    std::string s = "hello";
    return std::move(s);      // dangling rvalue reference; also prevents NRVO
}
```

**Expected Output**  
(Undefined behavior if the caller uses the result.)

**Why this is wrong?**  
`s` is destroyed when `bad()` returns. The returned rvalue reference dangles. Additionally, `std::move(s)` prevents the compiler from applying NRVO, forcing a move that could have been elided entirely.

### Real-World Cases

- **`std::move`** – returns `T&&`.
- **`std::forward<T>`** – returns `T&&` (conditionally).
- **Rvalue-ref-qualified member functions** – `std::optional::value() &&` returns `T&&` to allow moving from a temporary optional.
- **`std::get` on rvalue tuples** – returns `T&&` to allow moving elements from temporary tuples.

---

## 5. Return Value Optimization (RVO)

### Definitions

**Core Definition**
Return Value Optimization (RVO) is a copy-elision technique in which a temporary returned from a function is constructed directly in the storage where the caller will use it, eliminating the temporary and the copy/move.

**Technical Definition**
In the pre-C++17 model, RVO refers to the elision of a copy/move when the `return` statement’s operand is a **prvalue** (a nameless temporary) of the same class type as the function’s return type. The compiler is permitted to construct the prvalue directly in the result object’s storage, even if the copy/move constructor has side effects. Since C++17, this form of elision is **mandatory** and is no longer considered an optimization; prvalues are not materialized until needed.

**Beginner-Friendly Explanation**
RVO means the function’s result is built **directly where the caller wants it**, skipping the usual “build a temporary, then copy it” dance. In C++17 and later, this is not just an optimization — the language guarantees it for nameless temporaries.

### Purposes

- To eliminate unnecessary copies and moves for returned temporaries.
- To enable efficient return of large objects without relying on move constructors.
- To allow non-movable types to be returned by value (when returned as prvalues).
- To reduce the cost of factory-style functions.

### Syntax Rules and Structure

RVO is not written explicitly; it applies automatically to `return` statements whose operand is a **prvalue** of the function’s return type (ignoring cv-qualification).

**Pattern**
```cpp
T make() {
    return T(args);      // RVO / guaranteed elision (C++17)
}
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `T(args)` | A prvalue temporary of type `T`. |
| `return ...;` | The prvalue is used to initialize the result object directly. |

**Syntax Rules**
- The operand must be a **prvalue** of the same type as the return type (ignoring cv-qualification).
- The copy/move constructor need **not** be accessible in C++17 for guaranteed elision (pre-C++17, it had to be accessible even if elided).
- RVO applies at every return statement whose operand is a prvalue.

**Constraints and Limitations**
- RVO does **not** apply to named local variables (that is NRVO, which is optional).
- Pre-C++17, RVO was optional; a compiler could choose not to elide.
- Even when elided pre-C++17, the copy/move constructor had to be accessible.

### Annotated Code Examples

**Example 1: RVO with a prvalue temporary (C++17 guaranteed)**

```cpp
#include <iostream>

struct Tracker {
    Tracker()  { std::cout << "default ctor\n"; }
    Tracker(const Tracker&) { std::cout << "copy ctor\n"; }
    Tracker(Tracker&&)      { std::cout << "move ctor\n"; }
    ~Tracker() { std::cout << "dtor\n"; }
};

Tracker make() {
    return Tracker{};        // prvalue: RVO / guaranteed elision
}

int main() {
    Tracker t = make();
    std::cout << "---\n";
}
```

**Expected Output (C++17)**
```
default ctor
---
dtor
```

**Why this output?**  
The `Tracker{}` prvalue is used to initialize `t` directly; no copy or move constructor is called. Only one object (`t`) is constructed and destroyed.

**Example 2: RVO in a chained factory (C++17)**

```cpp
#include <iostream>
#include <string>

std::string inner() { return std::string("hello"); }

std::string outer() { return inner(); }   // prvalue → prvalue → direct init

int main() {
    std::string s = outer();
    std::cout << s << '\n';
}
```

**Expected Output**
```
hello
```

**Why this output?**  
Even through two layers of function calls, the prvalue chain is never materialized as a temporary; the final string is constructed directly into `s`.

### Real-World Cases

- **`std::make_unique<T>(...)`** – returns a prvalue `unique_ptr`; RVO/guaranteed elision applies.
- **`std::make_shared<T>(...)`** – same.
- **Factory functions returning temporaries** – `T create()` with `return T(args);`.
- **Operator overloads returning new objects** – `operator+` returning a temporary.

---

## 6. Named Return Value Optimization (NRVO)

### Definitions

**Core Definition**
Named Return Value Optimization (NRVO) is a copy-elision technique in which a **named local variable** returned from a function is constructed directly in the caller’s result storage, eliminating the copy/move.

**Technical Definition**
NRVO applies when the `return` statement’s operand is the name of a non-volatile local variable with automatic storage duration (not a function parameter), of the same class type (ignoring cv-qualification) as the function’s return type. Unlike RVO, NRVO is **optional** even in C++17; the standard does not mandate it. If NRVO is not applied, the compiler treats the named local as an rvalue in the return statement, allowing a move (if available) rather than a copy.

**Beginner-Friendly Explanation**
NRVO is like RVO, but for **named** local variables. The compiler may build the named variable directly in the caller’s storage. This is a very common optimization, but it is **not guaranteed** — the compiler is allowed to skip it.

### Purposes

- To eliminate copies/moves when returning a named local object.
- To make `return local;` efficient for large objects.
- To allow the natural “build a local, then return it” style without performance penalties.

### Syntax Rules and Structure

NRVO is triggered by the pattern:
```cpp
T function() {
    T local(args);
    // ... use local ...
    return local;           // NRVO candidate
}
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `T local` | A named local variable of type `T`. |
| `return local;` | NRVO may construct `local` directly in the result object’s storage. |

**Syntax Rules**
- The operand must be the **name** of a local automatic variable (not a parameter, not a temporary).
- The variable must be of the same class type as the return type (ignoring cv-qualification).
- If NRVO is not applied, the return statement treats the local as an rvalue (move if possible).
- **Do not** write `return std::move(local);`; it prevents NRVO and is considered a pessimization.

**Constraints and Limitations**
- NRVO is **optional** in all C++ standards, including C++17.
- NRVO is **forbidden** in constant expressions (C++17).
- Different compilers and optimization levels may or may not apply NRVO.

### Annotated Code Examples

**Example 1: NRVO candidate**

```cpp
#include <iostream>

struct Tracker {
    Tracker()  { std::cout << "default ctor\n"; }
    Tracker(const Tracker&) { std::cout << "copy ctor\n"; }
    Tracker(Tracker&&)      { std::cout << "move ctor\n"; }
    ~Tracker() { std::cout << "dtor\n"; }
};

Tracker build() {
    Tracker local;           // named local
    return local;            // NRVO may elide the copy/move
}

int main() {
    Tracker t = build();
    std::cout << "---\n";
}
```

**Expected Output (with NRVO)**
```
default ctor
---
dtor
```

**Expected Output (without NRVO)**
```
default ctor
move ctor
dtor
---
dtor
```

**Why this output?**  
If the compiler applies NRVO, `local` is constructed directly in `t`’s storage, and only one object is created. If NRVO is not applied, `local` is move-constructed into `t`, and both objects exist (briefly). NRVO is optional, so both outputs are conforming.

**Example 2: `return std::move(local)` prevents NRVO**

```cpp
#include <iostream>

struct Tracker {
    Tracker()  { std::cout << "default ctor\n"; }
    Tracker(const Tracker&) { std::cout << "copy ctor\n"; }
    Tracker(Tracker&&)      { std::cout << "move ctor\n"; }
    ~Tracker() { std::cout << "dtor\n"; }
};

Tracker bad() {
    Tracker local;
    return std::move(local);   // inhibits NRVO
}

int main() {
    Tracker t = bad();
    std::cout << "---\n";
}
```

**Expected Output**
```
default ctor
move ctor
dtor
---
dtor
```

**Why this output?**  
`std::move(local)` produces an xvalue, preventing NRVO. A move constructor is called, and `local` is destroyed before `t`. This is strictly worse than `return local;`.

### Real-World Cases

- **String builders** – `std::string build_message() { std::string s; /* ... */ return s; }`.
- **Container construction** – building a `std::vector` locally and returning it.
- **Object initialization** – constructing a complex object step by step and returning it by value.

---

## 7. Copy Elision

### Definitions

**Core Definition**
Copy elision is the compiler’s ability to omit a copy or move of a class object, constructing the object directly in its final destination instead.

**Technical Definition**
Copy elision encompasses several permitted (and, since C++17, certain mandatory) omissions of copy/move constructor calls. The principal forms are RVO (prvalue operands in return statements), NRVO (named locals in return statements), and the initialization of an object from a temporary. Pre-C++17, elision was always optional, but even when elided, the copy/move constructor had to be accessible; post-C++17, guaranteed elision for prvalues requires no constructor at all.

**Beginner-Friendly Explanation**
Copy elision is the compiler’s “magic trick” of **skipping a copy** by building an object in the exact spot where it will ultimately live. It makes returning large objects by value practical and efficient.

### Purposes

- To eliminate unnecessary copies and moves.
- To improve performance and reduce memory usage.
- To allow non-movable types to be returned by value (when elision is guaranteed).
- To simplify code by removing the need for manual optimization tricks.

### Syntax Rules and Structure

Copy elision is not written explicitly; it applies automatically under the standard’s permitted/required circumstances.

**Key Patterns**

| Pattern | Elision Form | Standard |
|---------|--------------|----------|
| `return T(args);` | RVO | C++17: guaranteed |
| `T x = T(args);` | Temporary materialization | C++17: guaranteed |
| `return local;` | NRVO | Optional (all versions) |
| `throw T(args);` | RVO for exceptions | C++17: guaranteed |

**Syntax Rules**
- Pre-C++17: the copy/move constructor must be accessible even if elided.
- C++17: for guaranteed elision (prvalues), the copy/move constructor need not be accessible.
- NRVO remains optional.
- Copy elision can change observable side effects of copy/move constructors (they may not be called).

**Constraints and Limitations**
- NRVO is not guaranteed.
- Elision is not applied if the copy/move constructor has side effects that the program relies on (but since the compiler may elide, relying on those side effects is non-portable).
- `-fno-elide-constructors` can disable elision on some compilers (for testing).

### Annotated Code Examples

**Example 1: Elision of a temporary in initialization**

```cpp
#include <iostream>

struct Tracker {
    Tracker() { std::cout << "default\n"; }
    Tracker(const Tracker&) { std::cout << "copy\n"; }
    Tracker(Tracker&&) { std::cout << "move\n"; }
    ~Tracker() { std::cout << "dtor\n"; }
};

int main() {
    Tracker t = Tracker{};   // elision: Tracker{} directly initializes t
}
```

**Expected Output (C++17)**
```
default
dtor
```

**Why this output?**  
`Tracker{}` is a prvalue; in C++17, it directly initializes `t`. No copy or move constructor is called.

**Example 2: `throw` with elision (C++17)**

```cpp
#include <iostream>

struct E {
    E() { std::cout << "E ctor\n"; }
    E(const E&) { std::cout << "E copy\n"; }
};

void f() {
    throw E{};            // guaranteed elision: E{} initializes the exception object
}

int main() {
    try { f(); } catch (const E&) {}
}
```

**Expected Output (C++17)**
```
E ctor
```

**Why this output?**  
The prvalue `E{}` directly initializes the exception object; no copy constructor is called.

### Real-World Cases

- **Returning large objects** – vectors, strings, matrices from functions.
- **Factory functions** – constructing objects directly in the caller’s storage.
- **Exception objects** – throwing temporaries with guaranteed elision (C++17).
- **Standard library internals** – `std::make_shared` uses elision extensively.

---

## 8. Guaranteed Copy Elision (C++17)

### Definitions

**Core Definition**
Guaranteed copy elision is the C++17 rule that, when an object is initialized from a **prvalue** of the same type, the object is initialized directly from the prvalue’s initializer, and **no temporary exists** and **no copy/move operation is performed** — not even conceptually.

**Technical Definition**
C++17 redefined prvalue semantics so that a prvalue is not an object but an **initialization recipe**. When a prvalue of type `T` is used to initialize an object of type `T` (ignoring cv-qualification), the target object is constructed directly by the prvalue’s initializer. No copy/move constructor is required to exist or be accessible. This applies to return statements, variable initializations, function arguments, and `throw` expressions.

**Beginner-Friendly Explanation**
In C++17, if a function returns a **nameless temporary**, the compiler doesn’t just **optimize away** the copy — the copy **never exists** in the first place. The temporary is built directly where it’s needed. This is stronger than RVO; it’s a language guarantee.

### Purposes

- To make returning temporaries by value **zero-cost by language rule**, not just by compiler optimization.
- To allow **non-movable** and **non-copyable** types to be returned by value (when returned as prvalues).
- To simplify reasoning about performance: `return T(args);` is always efficient.
- To eliminate the need for `std::move` in return statements for temporaries.

### Syntax Rules and Structure

Guaranteed elision applies automatically to the following patterns (C++17+):

**Pattern 1: Return of a prvalue**
```cpp
T f() {
    return T(args);       // guaranteed: no copy/move
}
```

**Pattern 2: Initialization from a prvalue**
```cpp
T x = T(args);            // guaranteed: no copy/move
```

**Pattern 3: Function argument from a prvalue**
```cpp
void g(T);
g(T(args));               // guaranteed: T(args) initializes the parameter directly
```

**Pattern 4: Throw-expression**
```cpp
throw T(args);            // guaranteed: no copy/move into exception object
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `T(args)` | A prvalue (initialization recipe). |
| Target | The object being initialized (result, variable, parameter, exception object). |

**Syntax Rules**
- The prvalue and the target must be of the **same type** (ignoring cv-qualification).
- No copy/move constructor is required to be accessible.
- Guaranteed elision does **not** apply to named locals (NRVO remains optional).
- Guaranteed elision does **not** apply when the return expression is a conditional operator or a cast that yields an lvalue.

**Constraints and Limitations**
- NRVO is still optional; `return local;` does not benefit from guaranteed elision.
- Guaranteed elision does not apply if the return expression is a prvalue of a **different** type (a conversion is needed).
- `std::move` in a return statement **defeats** guaranteed elision (and NRVO).

### Annotated Code Examples

**Example 1: Non-movable type returned by value (C++17)**

```cpp
#include <iostream>

struct Immovable {
    Immovable() { std::cout << "ctor\n"; }
    Immovable(const Immovable&) = delete;
    Immovable(Immovable&&) = delete;
    ~Immovable() { std::cout << "dtor\n"; }
};

Immovable make() {
    return Immovable{};      // guaranteed elision: no copy/move needed
}

int main() {
    Immovable x = make();
}
```

**Expected Output**
```
ctor
dtor
```

**Why this output?**  
`Immovable` is neither copyable nor movable. In C++17, `return Immovable{}` is valid because the prvalue directly initializes `x`; no copy/move constructor is required.

**Example 2: Guaranteed elision through a function call**

```cpp
#include <iostream>

struct Widget {
    Widget() { std::cout << "Widget()\n"; }
    Widget(const Widget&) { std::cout << "copy\n"; }
    Widget(Widget&&) { std::cout << "move\n"; }
};

void consume(Widget) { std::cout << "consumed\n"; }

int main() {
    consume(Widget{});       // Widget{} directly initializes the parameter
}
```

**Expected Output (C++17)**
```
Widget()
consumed
```

**Why this output?**  
The prvalue `Widget{}` initializes the by-value parameter directly; no copy or move constructor is called.

**Example 3: `return std::move(local)` defeats elision (pessimization)**

```cpp
#include <iostream>
#include <string>

std::string pessimistic() {
    std::string s = "data";
    return std::move(s);     // inhibits NRVO; may force a move
}

int main() {
    std::string r = pessimistic();
}
```

**Expected Output**  
(With NRVO normally applied, `return s;` would construct `s` directly in `r`. With `std::move(s)`, a move constructor is forced.) This is why compilers emit `-Wpessimizing-move`.

### Real-World Cases

- **Non-movable resource types** – types with deleted copy/move that are returned as temporaries.
- **Factory functions** – `return T(args);` is guaranteed zero-overhead.
- **`std::make_unique` / `std::make_shared`** – return prvalues that are guaranteed-elided.
- **Exception handling** – throwing temporaries without copies.

---

## 9. Return Best Practices

### Definitions

**Core Definition**
Return best practices are guidelines for choosing the appropriate return mechanism to ensure correctness, performance, and maintainability.

**Technical Definition**
These practices are codified in the C++ Core Guidelines, particularly **F.20** (“For ‘out’ output values, prefer return values to output parameters”) and **F.21** (“To return multiple ‘out’ values, prefer returning a struct or tuple”). Additional rules address dangling references, pessimizing moves, and ownership.

**Beginner-Friendly Explanation**
Best practices for returning values are simple rules: **return by value by default**, **never return a reference or pointer to a local**, **don’t use `std::move` in a return statement**, and **use a struct to return multiple values**. Following these rules makes your code safe, fast, and easy to understand.

### Key Guidelines (Summary Table)

| Guideline | Summary | Preferred Mechanism |
|-----------|---------|---------------------|
| **F.20** | Prefer return values to output parameters. | Return by value. |
| **F.21** | To return multiple values, use a `struct` or `tuple`. | `struct` (named, self-documenting) or `std::tuple`. |
| **Dangling** | Never return a reference or pointer to a local automatic. | Return by value instead. |
| **Pessimizing move** | Never write `return std::move(local);`. | Write `return local;` (NRVO / automatic move). |
| **Ownership** | Do not return raw owning pointers. | Return `std::unique_ptr<T>` or `std::shared_ptr<T>`. |
| **Optional** | Use `std::optional<T>` instead of `T*` for optional values. | `std::optional<T>` (C++17). |
| **Non-movable** | Return non-movable types as prvalues (C++17). | `return T(args);` |

**Additional Rules of Thumb**

- **Return by value** for arithmetic types, small trivially copyable types, and most class types (RVO/NRVO/move make it efficient).
- **Return by `const&`** only for members or persistent objects that the caller must not modify; never for locals.
- **Return by non-`const&`** only when the caller must modify the original (e.g., `operator[]`, stream operators).
- **Return by pointer** only for optional results (`nullptr` means “none”) or C API compatibility; prefer `std::optional` or smart pointers.
- **Return by `T&&`** only in `std::move`, `std::forward`, and rvalue-ref-qualified member functions.
- **Let the compiler elide** — do not try to “help” with `std::move`.

### Annotated Code Examples

**Example 1: Return by value for a computation (F.20)**

```cpp
#include <iostream>
#include <string>

// GOOD: return by value; caller gets an independent object
std::string make_greeting(const std::string& name) {
    return "Hello, " + name + "!";
}

int main() {
    std::string g = make_greeting("Bob");
    std::cout << g << '\n';
}
```

**Expected Output**
```
Hello, Bob!
```

**Why this is good?**  
The function returns a `std::string` by value. Copy elision/move semantics make this efficient, and the caller receives a fully independent object.

**Example 2: Return a `struct` for multiple values (F.21)**

```cpp
#include <iostream>
#include <string>

struct ParseResult {
    bool success;
    std::string token;
    int position;
};

ParseResult parse(const std::string& input) {
    if (input.empty()) return {false, "", 0};
    return {true, input.substr(0, 3), 3};
}

int main() {
    auto r = parse("hello");
    if (r.success) {
        std::cout << "Token: " << r.token << " at " << r.position << '\n';
    }
}
```

**Expected Output**
```
Token: hel at 3
```

**Why this is good?**  
Returning a `struct` with named members is self-documenting and avoids the ambiguity of output parameters. A `tuple` would also work but loses member names.

**Example 3: Avoiding the pessimizing move**

```cpp
#include <iostream>
#include <string>

std::string good() {
    std::string s = "efficient";
    return s;                // NRVO / automatic move
}

std::string bad() {
    std::string s = "inefficient";
    return std::move(s);     // inhibits NRVO; forces a move
}

int main() {
    std::cout << good() << '\n';
    std::cout << bad() << '\n';
}
```

**Expected Output**
```
efficient
inefficient
```

**Why this matters?**  
`return s;` allows NRVO (construct `s` directly in the caller’s storage) or, failing that, an automatic move. `return std::move(s);` prevents NRVO and forces a move, which is strictly worse.

**Example 4: Returning a smart pointer for ownership**

```cpp
#include <iostream>
#include <memory>

std::unique_ptr<int> make_int(int value) {
    return std::make_unique<int>(value);   // prvalue; guaranteed elision
}

int main() {
    auto p = make_int(42);
    std::cout << *p << '\n';
}
```

**Expected Output**
```
42
```

**Why this is good?**  
`std::unique_ptr` expresses exclusive ownership clearly. The prvalue returned by `std::make_unique` is guaranteed-elided into `p`.

### Real-World Cases

- **Standard library** – `std::make_unique`, `std::make_shared`, `std::optional`, `std::variant`, `std::expected` (C++23) all follow these guidelines.
- **Factory patterns** – return by value or smart pointer, never raw owning pointers.
- **API design** – Core Guidelines F.20/F.21 are used as design checklists in code reviews.
- **Static analysis** – compilers and tools warn about `-Wpessimizing-move`, `-Wreturn-local-addr`, and `-Wdangling-reference`.

---

## References

- return statement – https://en.cppreference.com/w/cpp/language/return
- Copy elision – https://en.cppreference.com/w/cpp/language/copy_elision
- Value categories – https://en.cppreference.com/w/cpp/language/value_category
- Reference declaration – https://en.cppreference.com/w/cpp/language/reference
- Pointer declaration – https://en.cppreference.com/w/cpp/language/pointer
- C++ Core Guidelines — F.20: For “out” output values, prefer return values to output parameters – https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rf-out
- C++ Core Guidelines — F.21: To return multiple “out” values, prefer returning a struct or tuple – https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rf-out-multi
- C++ Core Guidelines — F.43: Never (directly or indirectly) return a pointer or a reference to a local object – https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rf-dangle
- GotW #2 Solution: Temporary Objects (Herb Sutter) – https://isocpp.org/blog/2013/05/gotw-2-solution-temporary-objects-herb-sutter
- Quick Q: What does it mean to return a reference? (isocpp.org) – https://isocpp.org/blog/2018/11/quick-q-what-does-it-mean-to-return-a-reference
- Guaranteed copy elision (C++17) — cppreference revision – https://en.cppreference.com/index.php?title=cpp/language/copy_elision&oldid=113462
- `-Wpessimizing-move` (GCC warning documentation) – https://gcc.gnu.org/onlinedocs/gcc/C_002b_002b-Dialect-Options.html (under `-Wpessimizing-move`)
- Return-By-Value / Return-By-Pointer / Return-By-Reference (Weber State University) – https://icarus.cs.weber.edu/~dab/cs1410/textbook/6.Functions/return2.html

---

This cheat sheet provides a complete, structured reference for C++ return values, covering every requested mechanism with definitions, syntax, annotated examples, expected outputs, and authoritative references. Each section follows the same uniform template to facilitate study and review.