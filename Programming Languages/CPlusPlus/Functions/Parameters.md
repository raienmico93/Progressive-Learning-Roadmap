# Parameter Passing — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
Parameter passing is the mechanism by which argument values or objects at a call site are used to initialize the parameters of a function, determining whether the function receives a copy, an alias (reference), or the identity of the original object.

**Technical Definition**
Parameter passing in C++ is governed by the initialization of function parameters from the corresponding arguments. The parameter’s declared type determines the initialization semantics: by-value parameters are copy-initialized from the argument (which may invoke a copy or move constructor); reference parameters bind directly to the argument; pointer parameters receive a copy of the pointer value. The value category of the argument (lvalue vs. rvalue) and the cv-qualification of the referenced type further constrain binding.

**Beginner-Friendly Explanation**
Parameter passing is how you hand data to a function. You can give the function a **copy** of the data (by value), give it a **direct alias** to the original data (by reference), give it the **memory address** of the data (by pointer), or give it special permissions to **move** the data (rvalue reference). The choice affects performance, safety, and what the function can do with the data.

### Key Characteristics

- **Copy vs. alias**: By-value passing copies the argument; reference passing binds an alias to the original object.
- **Value category awareness (C++11+)**: Rvalue references and forwarding references distinguish lvalues from rvalues, enabling move semantics and perfect forwarding.
- **cv-qualification**: `const` references and `const` pointers prevent modification through the parameter.
- **Efficiency**: Large objects are expensive to copy; references avoid copies but introduce aliasing.
- **Safety**: Passing by value is the safest (no aliasing, no lifetime issues); references and pointers require attention to lifetime and modification.

### Prerequisites

- Understanding of lvalues, rvalues, and value categories (C++11).
- Familiarity with references (`&`, `&&`) and pointers.
- Basic knowledge of copy/move constructors and move semantics.
- Awareness of the One Definition Rule and linkage (for function declarations).

### Related Programming Areas

- **Move semantics** – rvalue references and `std::move`.
- **Perfect forwarding** – forwarding references and `std::forward`.
- **API design** – choosing parameter types for library interfaces.
- **Performance optimization** – avoiding unnecessary copies.
- **Exception safety** – the interaction of parameter passing with exception guarantees.

### Core Concepts / Features

1. Pass by Value
2. Pass by Reference
3. Pass by Pointer
4. Pass by Const Reference
5. Pass by Rvalue Reference
6. Perfect Forwarding
7. `std::forward`
8. Parameter Passing Best Practices

---

## 1. Pass by Value

### Definitions

**Core Definition**
Passing by value means the function parameter is a **copy** of the argument; modifications to the parameter do not affect the original argument.

**Technical Definition**
A by-value parameter is initialized via copy-initialization from the argument. For class types, this invokes the copy constructor (or the move constructor if the argument is an rvalue and the parameter type is movable). The parameter has automatic storage duration within the function and is destroyed when the function returns.

**Beginner-Friendly Explanation**
When you pass by value, the function gets its own **personal copy** of the data. Whatever the function does to that copy stays inside the function — the original data outside is untouched.

### Purposes

- To ensure the function operates on an independent copy, preventing accidental modification of the caller’s data.
- To simplify reasoning about function behavior (no aliasing).
- To enable the use of the parameter as a local variable that can be freely modified.
- To support move semantics for sink parameters (taking ownership by value and then moving).

### Syntax Rules and Structure

**General Syntax**
```cpp
return_type function_name(type parameter_name);
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `type` | The type of the parameter (no `&` or `*`). |
| `parameter_name` | The local name used inside the function. |

**Syntax Rules**
- The argument is copy-initialized into the parameter.
- For class types, a copy constructor (or move constructor if the argument is an rvalue) must be accessible.
- The parameter is an lvalue inside the function body, regardless of the argument’s value category.

**Constraints and Limitations**
- Copying large objects can be expensive (time and memory).
- Abstract class types cannot be passed by value (slicing would occur; abstract classes cannot be instantiated).
- Passing polymorphic objects by value causes **object slicing** — only the base-class portion is copied.

### Annotated Code Examples

**Example 1: Pass by value with a fundamental type**

```cpp
#include <iostream>

void increment(int x) {
    ++x;                         // modifies the local copy
    std::cout << "Inside: " << x << '\n';
}

int main() {
    int a = 5;
    increment(a);                // a is copied into x
    std::cout << "Outside: " << a << '\n';
}
```

**Expected Output**
```
Inside: 6
Outside: 5
```

**Why this output?**  
`x` is a copy of `a`. The `++x` modifies only the copy; the original `a` remains `5`.

**Example 2: Pass by value with a class type**

```cpp
#include <iostream>
#include <string>

struct Person {
    std::string name;
    int age;
};

void birthday(Person p) {        // copies the Person
    p.age += 1;
    std::cout << p.name << " is now " << p.age << '\n';
}

int main() {
    Person alice{"Alice", 30};
    birthday(alice);
    std::cout << "Original age: " << alice.age << '\n';
}
```

**Expected Output**
```
Alice is now 31
Original age: 30
```

**Why this output?**  
The `Person` object is copied into `p`; modifying `p.age` does not affect `alice.age`.

**Example 3: Sink parameter (by value + move)**

```cpp
#include <iostream>
#include <string>
#include <utility>

class Widget {
    std::string data;
public:
    explicit Widget(std::string d) : data(std::move(d)) {   // move from by-value parameter
        std::cout << "Widget constructed with: " << data << '\n';
    }
};

int main() {
    std::string s = "expensive resource";
    Widget w(std::move(s));      // s is moved into the parameter, then moved into data
}
```

**Expected Output**
```
Widget constructed with: expensive resource
```

**Why this output?**  
The by-value parameter `d` is move-constructed from `s` (since `s` is cast to an rvalue via `std::move`), and then `data` is move-constructed from `d`. This is the **sink parameter** idiom: take by value and move from the parameter to avoid extra copies.

### Real-World Cases

- **Small types** – `int`, `double`, `char`, pointers, enums are cheap to copy and naturally passed by value.
- **Sink parameters** – constructors and setters that take ownership of a resource (e.g., `std::vector`, `std::string`) often take by value and move.
- **Functional programming** – small immutable values are passed by value to avoid aliasing.

---

## 2. Pass by Reference

### Definitions

**Core Definition**
Passing by reference means the function parameter is an **alias** for the argument; modifications to the parameter affect the original argument.

**Technical Definition**
A reference parameter (`T&`) binds directly to the argument without making a copy. The reference is an lvalue inside the function and refers to the original object. No copy constructor is invoked. The argument must be an lvalue (or an rvalue if the reference is to `const`).

**Beginner-Friendly Explanation**
Passing by reference is like giving the function a **nickname** for your variable. Whatever the function does through that nickname happens to your actual variable.

### Purposes

- To allow functions to modify the caller’s object (in-out parameters).
- To avoid the cost of copying large objects while still allowing modification.
- To enable functions to return multiple values through parameters.
- To support operator overloading (e.g., `operator<<` returns `std::ostream&`).

### Syntax Rules and Structure

**General Syntax**
```cpp
return_type function_name(type& parameter_name);
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `type&` | An lvalue reference to `type`. |
| `parameter_name` | The alias used inside the function. |

**Syntax Rules**
- A non-`const` lvalue reference cannot bind to an rvalue (temporary).
- The argument must be an lvalue of a type compatible with `type`.
- No copy is made; the reference directly aliases the argument.
- The parameter is an lvalue expression inside the function.

**Constraints and Limitations**
- Cannot bind a non-`const` lvalue reference to a temporary or literal.
- Returning a reference to a local variable creates a dangling reference (undefined behavior).
- Aliasing can lead to subtle bugs if the function modifies the referent while other aliases exist.

### Annotated Code Examples

**Example 1: Modifying through a reference**

```cpp
#include <iostream>

void increment(int& x) {         // x aliases the argument
    ++x;
}

int main() {
    int a = 5;
    increment(a);
    std::cout << a << '\n';
}
```

**Expected Output**
```
6
```

**Why this output?**  
`x` is a reference to `a`; `++x` directly increments `a`.

**Example 2: Swapping two values**

```cpp
#include <iostream>

void swap(int& a, int& b) {
    int tmp = a;
    a = b;
    b = tmp;
}

int main() {
    int x = 1, y = 2;
    swap(x, y);
    std::cout << x << ' ' << y << '\n';
}
```

**Expected Output**
```
2 1
```

**Why this output?**  
Both `a` and `b` are references; the swap modifies the original `x` and `y`.

**Example 3: Returning multiple values via references**

```cpp
#include <iostream>

void min_max(const int* arr, int n, int& min_out, int& max_out) {
    min_out = max_out = arr[0];
    for (int i = 1; i < n; ++i) {
        if (arr[i] < min_out) min_out = arr[i];
        if (arr[i] > max_out) max_out = arr[i];
    }
}

int main() {
    int data[] = {3, 1, 4, 1, 5, 9, 2, 6};
    int mn, mx;
    min_max(data, 8, mn, mx);
    std::cout << "Min: " << mn << ", Max: " << mx << '\n';
}
```

**Expected Output**
```
Min: 1, Max: 9
```

**Why this output?**  
`min_out` and `max_out` are non-`const` references; the function writes the computed minimum and maximum into the caller’s variables.

### Real-World Cases

- **Swap functions** – `std::swap` uses references.
- **Stream operators** – `operator<<(std::ostream&, const T&)` returns a reference to the stream.
- **In-out parameters** – functions that update a data structure in place.
- **Large objects that need modification** – avoid copying while allowing writes.

---

## 3. Pass by Pointer

### Definitions

**Core Definition**
Passing by pointer means the function receives a **copy of the pointer value**, which points to the original object (or to `nullptr`).

**Technical Definition**
A pointer parameter (`T*`) is a by-value parameter whose type is “pointer to `T`.” The argument is a pointer expression; the pointer itself is copied, but it still points to the same object. The pointed-to object can be modified through the pointer (unless the pointer is to `const`). A null pointer is a valid argument and must be checked before dereferencing.

**Beginner-Friendly Explanation**
Passing by pointer is like giving the function a **map** to your data. The function can follow the map to your data and change it, but the function gets its own copy of the map (the pointer). The function can also receive a “blank map” (`nullptr`), meaning “no data.”

### Purposes

- To allow optional parameters (a null pointer indicates “no object”).
- To allow the function to modify the pointed-to object.
- To pass arrays and strings (which decay to pointers).
- To support C-style APIs and interoperability with C libraries.

### Syntax Rules and Structure

**General Syntax**
```cpp
return_type function_name(type* parameter_name);
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `type*` | A pointer to `type`. |
| `parameter_name` | The pointer variable used inside the function. |

**Syntax Rules**
- The argument must be convertible to `type*`.
- The pointer itself is passed by value; reassigning the pointer inside the function does not affect the caller’s pointer.
- The pointed-to object can be modified through the pointer (unless `type` is `const`).
- Null pointers are valid arguments and should be checked.

**Constraints and Limitations**
- Dereferencing a null or dangling pointer results in undefined behavior.
- Pointer syntax is more verbose than reference syntax.
- Raw pointers do not express ownership; use smart pointers for ownership semantics.
- Array parameters decay to pointers, losing size information.

### Annotated Code Examples

**Example 1: Modifying through a pointer**

```cpp
#include <iostream>

void increment(int* p) {
    if (p) {                    // null check
        ++(*p);                 // dereference and increment
    }
}

int main() {
    int a = 5;
    increment(&a);
    std::cout << a << '\n';
    increment(nullptr);         // safe: no dereference
}
```

**Expected Output**
```
6
```

**Why this output?**  
`p` points to `a`; `++(*p)` increments `a`. The second call passes `nullptr` and is safely ignored.

**Example 2: Optional output parameter**

```cpp
#include <iostream>

bool divide(int numerator, int denominator, int* result) {
    if (denominator == 0) return false;
    if (result) {               // optional output
        *result = numerator / denominator;
    }
    return true;
}

int main() {
    int quotient;
    if (divide(10, 3, &quotient)) {
        std::cout << "Quotient: " << quotient << '\n';
    }
    // Pass nullptr to ignore the result
    if (divide(10, 0, nullptr)) {
        std::cout << "unreachable\n";
    } else {
        std::cout << "Division by zero\n";
    }
}
```

**Expected Output**
```
Quotient: 3
Division by zero
```

**Why this output?**  
The pointer parameter allows the caller to optionally receive the result. Passing `nullptr` means “don’t compute the output.”

### Real-World Cases

- **C APIs** – many C functions take pointers for output parameters (e.g., `gettimeofday`).
- **Optional parameters** – null pointers indicate absence.
- **Array processing** – arrays decay to pointers when passed to functions.
- **Low-level memory manipulation** – buffer pointers, byte-level operations.

---

## 4. Pass by Const Reference

### Definitions

**Core Definition**
Passing by `const` reference means the function receives a read-only alias to the argument, avoiding a copy while preventing modification.

**Technical Definition**
A `const T&` parameter binds to lvalues and rvalues of type `T` (or convertible to `T`) without copying. The referent cannot be modified through this reference. Temporary objects created from conversions have their lifetime extended to the end of the full expression containing the call.

**Beginner-Friendly Explanation**
Passing by `const` reference is like giving the function a **read-only window** into your data. The function can look at the data without copying it, but it cannot change it.

### Purposes

- To avoid copying large objects while guaranteeing the function will not modify them.
- To accept both lvalues and rvalues (temporaries) without extra copies in many cases.
- To provide a safe and efficient default for “input” parameters of class types.
- To enable functions to accept arguments that require implicit conversion (a temporary is created and bound to the `const` reference).

### Syntax Rules and Structure

**General Syntax**
```cpp
return_type function_name(const type& parameter_name);
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `const type&` | A reference to `const type`. |
| `parameter_name` | The read-only alias. |

**Syntax Rules**
- A `const` lvalue reference can bind to rvalues (temporaries).
- No copy of the argument is made (unless a conversion creates a temporary).
- The function cannot modify the referent through this reference.
- `const` can be applied to the referred-to type, not the reference itself (references are inherently immutable).

**Constraints and Limitations**
- Cannot be used when the function needs to modify the argument.
- Binding a temporary to a `const` reference extends the temporary’s lifetime to the end of the full expression, but **not** beyond the function call in all contexts (e.g., returning a reference to the temporary is still dangerous).
- For small, cheap-to-copy types, by-value may be more efficient than by-`const`-reference due to indirection.

### Annotated Code Examples

**Example 1: Avoiding a copy of a large object**

```cpp
#include <iostream>
#include <string>

void print_length(const std::string& s) {   // no copy
    std::cout << "Length: " << s.size() << '\n';
}

int main() {
    std::string text = "Hello, world!";
    print_length(text);                      // binds to lvalue
    print_length("temporary");               // binds to temporary (rvalue)
}
```

**Expected Output**
```
Length: 13
Length: 9
```

**Why this output?**  
The `std::string` is not copied; the function reads its size through the `const` reference. A temporary `std::string` is created for the string literal and bound to the `const` reference.

**Example 2: Accepting a temporary via conversion**

```cpp
#include <iostream>
#include <string>

void greet(const std::string& name) {
    std::cout << "Hello, " << name << "!\n";
}

int main() {
    greet("Alice");    // const char* converts to std::string temporary
}
```

**Expected Output**
```
Hello, Alice!
```

**Why this output?**  
A temporary `std::string` is constructed from `"Alice"` and bound to the `const` reference parameter. The function does not modify it.

### Real-World Cases

- **Printing and logging** – functions that inspect objects without modifying them.
- **Comparison functions** – `bool operator==(const T&, const T&)`.
- **Hashing** – `std::hash<T>::operator()(const T&)`.
- **Standard library algorithms** – predicates and comparators often take `const&`.

---

## 5. Pass by Rvalue Reference

### Definitions

**Core Definition**
Passing by rvalue reference (`T&&`) means the function can bind to rvalue arguments and is allowed to **move from** them, treating them as resources that can be “stolen.”

**Technical Definition**
An rvalue reference parameter (`T&&` where `T` is a concrete type, not a deduced template parameter) binds only to rvalues (prvalues and xvalues). The parameter is an lvalue inside the function, but its type signals that the caller has relinquished the resource and the function may move from it. `std::move` is typically used to cast the parameter to an rvalue when passing it onward to a move constructor or move assignment operator.

**Beginner-Friendly Explanation**
An rvalue reference parameter says: “This argument is a **temporary** or an object the caller doesn’t need anymore. I’m allowed to **take its resources** (like its internal buffer) instead of copying them.”

### Purposes

- To enable move semantics: efficient transfer of resources from temporaries.
- To implement move constructors and move assignment operators.
- To overload functions for rvalues (e.g., “fast path” for temporaries).
- To support perfect forwarding (when `T&&` is a forwarding reference).

### Syntax Rules and Structure

**General Syntax**
```cpp
return_type function_name(type&& parameter_name);
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `type&&` | An rvalue reference to `type` (where `type` is a concrete type). |
| `parameter_name` | The parameter name (an lvalue inside the function). |

**Syntax Rules**
- An rvalue reference to a **concrete type** binds only to rvalues.
- The parameter is an lvalue expression inside the function; use `std::move(parameter)` to treat it as an rvalue when forwarding.
- Rvalue references enable move constructors and move assignment operators.
- `T&&` where `T` is a **deduced template parameter** is a **forwarding reference** (see Perfect Forwarding), not a plain rvalue reference.

**Constraints and Limitations**
- Cannot bind to lvalues (unless `std::move` is used to cast an lvalue to an xvalue).
- Moving from a `const` rvalue reference usually results in a copy (move constructors typically take non-`const` rvalue references).
- After moving from an object, it is in a valid but unspecified state; it should not be used except to be reassigned or destroyed.

### Annotated Code Examples

**Example 1: Move constructor**

```cpp
#include <iostream>
#include <string>
#include <utility>

class Buffer {
    std::string data;
public:
    // Move constructor: takes rvalue reference
    Buffer(Buffer&& other) noexcept : data(std::move(other.data)) {
        std::cout << "Move constructed\n";
        // other is left in a valid but unspecified state
    }

    explicit Buffer(std::string d) : data(std::move(d)) {}
};

int main() {
    Buffer b1("hello");
    Buffer b2(std::move(b1));   // calls move constructor
}
```

**Expected Output**
```
Move constructed
```

**Why this output?**  
`std::move(b1)` casts `b1` to an rvalue, allowing the move constructor to bind to it. The `data` string is moved, not copied.

**Example 2: Overloading on rvalue reference**

```cpp
#include <iostream>
#include <string>

void process(const std::string& s) {
    std::cout << "lvalue version: " << s << '\n';
}

void process(std::string&& s) {
    std::cout << "rvalue version: " << s << '\n';
    // could move from s here
}

int main() {
    std::string a = "persistent";
    process(a);                 // lvalue → const& overload
    process("temporary");       // rvalue → && overload
}
```

**Expected Output**
```
lvalue version: persistent
rvalue version: temporary
```

**Why this output?**  
The lvalue `a` selects the `const std::string&` overload; the temporary string literal selects the `std::string&&` overload. This pattern is sometimes used to provide optimized handling of temporaries (e.g., reusing their storage).

### Real-World Cases

- **Move constructors and move assignment** – every movable class type.
- **`std::vector::push_back(T&&)`** – moves from temporary elements.
- **`std::make_unique<T>(Args&&...)`** – forwards arguments to the constructor.
- **Optimized overloads** – functions that can reuse the storage of temporaries.

---

## 6. Perfect Forwarding

### Definitions

**Core Definition**
Perfect forwarding is the practice of passing arguments through a wrapper function to another function while **preserving their original value category** (lvalue/rvalue) and cv-qualification.

**Technical Definition**
Perfect forwarding is achieved by declaring the wrapper’s parameters as **forwarding references** (`T&&` where `T` is a deduced template parameter) and passing them to the target function via `std::forward<T>(arg)`. Reference collapsing rules ensure that lvalues are forwarded as lvalues and rvalues as rvalues.

**Beginner-Friendly Explanation**
Imagine a receptionist (the wrapper function) who receives packages (arguments) and passes them to the right department (the target function). Perfect forwarding means the receptionist **doesn’t repackage** the items — a fragile item (rvalue) stays fragile, and a heavy item (lvalue) stays heavy, so the target department knows exactly how to handle them.

### Purposes

- To write wrapper functions (factories, delegating constructors, `std::make_unique`, `std::make_shared`) that forward arguments to constructors or other functions without extra copies.
- To preserve move semantics through multiple layers of function calls.
- To support generic code that should not impose unnecessary copying.
- To implement emplace-style operations (`emplace_back`, `emplace`).

### Syntax Rules and Structure

**General Syntax**
```cpp
template <typename T>
return_type wrapper(T&& arg) {
    target(std::forward<T>(arg));
}
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `T&&` | A forwarding reference (only when `T` is a deduced template parameter). |
| `std::forward<T>(arg)` | Casts `arg` back to its original value category. |

**Syntax Rules**
- `T&&` is a forwarding reference **only** when `T` is a deduced template parameter. `Widget&&` (concrete type) is a plain rvalue reference.
- Reference collapsing rules: `& + & = &`, `& + && = &`, `&& + & = &`, `&& + && = &&`. Only `&& + &&` yields `&&`.
- The pattern `template<typename T> void f(T&& x)` is the canonical forwarding reference.
- Use `std::forward<T>(x)` to forward; do **not** use `std::move` (which unconditionally casts to rvalue).

**Constraints and Limitations**
- Overloading on forwarding references can be tricky (they are greedy).
- `std::forward` must be given the correct template argument; using the wrong `T` can cause errors or incorrect behavior.
- Forwarding references do not work for out-parameters (you cannot forward an lvalue as an lvalue and also bind it to a non-`const` lvalue reference in the same call if the original argument was an rvalue).

### Annotated Code Examples

**Example 1: A forwarding wrapper**

```cpp
#include <iostream>
#include <string>
#include <utility>

void target(const std::string& s) { std::cout << "lvalue: " << s << '\n'; }
void target(std::string&& s)     { std::cout << "rvalue: " << s << '\n'; }

template <typename T>
void wrapper(T&& arg) {
    target(std::forward<T>(arg));   // preserve value category
}

int main() {
    std::string a = "hello";
    wrapper(a);              // T = std::string& → forwards as lvalue
    wrapper("temporary");    // T = const char(&)[10]? → adjusts; forwards as rvalue
}
```

**Expected Output**
```
lvalue: hello
rvalue: temporary
```

**Why this output?**  
`wrapper(a)` deduces `T = std::string&`; `std::forward<std::string&>(a)` returns `a` as an lvalue. `wrapper("temporary")` deduces `T = const char(&)[10]` (array reference); after decay to `const char*`, a temporary `std::string` is created and forwarded as an rvalue.

**Example 2: Perfect forwarding factory**

```cpp
#include <iostream>
#include <memory>
#include <string>
#include <utility>

struct Widget {
    Widget(std::string name, int value) {
        std::cout << "Widget(" << name << ", " << value << ")\n";
    }
};

template <typename T, typename... Args>
std::unique_ptr<T> make_unique_custom(Args&&... args) {
    return std::unique_ptr<T>(new T(std::forward<Args>(args)...));
}

int main() {
    auto w = make_unique_custom<Widget>("gadget", 42);
}
```

**Expected Output**
```
Widget(gadget, 42)
```

**Why this output?**  
`Args&&...` is a forwarding reference parameter pack. `std::forward<Args>(args)...` forwards each argument with its original value category to the `Widget` constructor, avoiding unnecessary copies.

### Real-World Cases

- **`std::make_unique<T>(args...)`** – forwards arguments to `T`’s constructor.
- **`std::make_shared<T>(args...)`** – same, with shared ownership.
- **`std::vector::emplace_back(args...)`** – constructs an element in place from forwarded arguments.
- **Delegating constructors** – forwarding constructor arguments to another constructor.
- **Generic wrapper functions** – logging, timing, or instrumentation wrappers that should not alter argument semantics.

---

## 7. `std::forward`

### Definitions

**Core Definition**
`std::forward<T>(arg)` is a utility that casts `arg` to the value category encoded in `T`, enabling perfect forwarding.

**Technical Definition**
`std::forward` is defined in `<utility>`. It has two overloads: one for lvalue references and one for rvalue references. When `T` is an lvalue reference type, `std::forward<T>(arg)` returns an lvalue; when `T` is a non-reference type (or an rvalue reference type), it returns an rvalue reference. It is the mechanism that makes perfect forwarding possible.

**Beginner-Friendly Explanation**
`std::forward` is like a **label maker**. If the original argument was an lvalue (a named object), `std::forward` labels it as an lvalue again. If it was an rvalue (a temporary), it labels it as an rvalue. This tells the next function whether it can safely move from the argument or must treat it as a persistent object.

### Purposes

- To preserve the value category of arguments during forwarding.
- To implement perfect forwarding in wrapper functions.
- To allow move semantics to propagate through layers of function calls.
- To avoid unnecessary copies in generic code.

### Syntax Rules and Structure

**General Syntax**
```cpp
std::forward<T>(arg)
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `T` | The template parameter deduced by the wrapper. |
| `arg` | The forwarding reference parameter. |

**Syntax Rules**
- `std::forward` must be used with a **forwarding reference** (`T&&` where `T` is deduced).
- `T` must be the exact deduced type (often `T` from `template<typename T> void f(T&& x)`).
- `std::forward` is a conditional cast: it casts to `T&&` (which, after reference collapsing, may be an lvalue or rvalue reference).
- It is `noexcept`.

**Constraints and Limitations**
- Do **not** use `std::forward` on a non-forwarding reference (it will still compile but may not do what you expect).
- Using `std::forward` outside the forwarding-reference pattern can cause bugs.
- Overloading `std::forward` is not allowed.

### Annotated Code Examples

**Example 1: `std::forward` preserving value category**

```cpp
#include <iostream>
#include <string>
#include <utility>

void inspect(const std::string& s) { std::cout << "lvalue ref\n"; }
void inspect(std::string&& s)     { std::cout << "rvalue ref\n"; }

template <typename T>
void relay(T&& arg) {
    inspect(std::forward<T>(arg));
}

int main() {
    std::string a = "hello";
    relay(a);              // T = std::string& → forwards lvalue
    relay(std::move(a));   // T = std::string  → forwards rvalue
    relay("temp");         // T = const char(&)[5] → temporary std::string forwarded as rvalue
}
```

**Expected Output**
```
lvalue ref
rvalue ref
rvalue ref
```

**Why this output?**  
`relay(a)` deduces `T = std::string&`; `std::forward<std::string&>(a)` yields an lvalue. `relay(std::move(a))` deduces `T = std::string`; `std::forward<std::string>` yields an rvalue. The literal creates a temporary that is forwarded as an rvalue.

**Example 2: `std::forward` vs. `std::move`**

```cpp
#include <iostream>
#include <string>
#include <utility>

template <typename T>
void wrong_forward(T&& arg) {
    // BAD: unconditionally casts to rvalue, even if arg was an lvalue
    std::string s = std::move(arg);   // moves from lvalue arguments too!
}

template <typename T>
void correct_forward(T&& arg) {
    // GOOD: preserves value category
    std::string s = std::forward<T>(arg);
}

int main() {
    std::string a = "important";
    // correct_forward(a);   // copies a (lvalue forwarded as lvalue)
    // wrong_forward(a);     // would move from a, leaving it empty
}
```

**Expected Output**  
(No output; illustrative.)

**Why this matters?**  
`std::move` unconditionally produces an rvalue, which can silently move from lvalues that the caller did not intend to give up. `std::forward<T>` preserves the original value category, so lvalues remain lvalues.

### Real-World Cases

- **Standard library factories** – `std::make_unique`, `std::make_shared`, `std::vector::emplace_back`.
- **Perfect-forwarding constructors** – forwarding constructors in wrapper classes (e.g., `std::pair`, `std::tuple`).
- **Generic logging wrappers** – `template<class F, class... Args> auto log_call(F&& f, Args&&... args)`.

---

## 8. Parameter Passing Best Practices

### Definitions

**Core Definition**
Parameter passing best practices are guidelines for choosing the most appropriate parameter types to balance correctness, safety, and performance.

**Technical Definition**
These practices are codified in the C++ Core Guidelines, particularly rules F.15–F.21, which categorize parameters as “in,” “in-out,” “out,” and “consume” parameters and recommend specific passing mechanisms for each category.

**Beginner-Friendly Explanation**
Best practices are simple rules of thumb: use **by value** for small things, use **`const` reference** for big things you only want to read, use **reference** for things you want to change, use **rvalue reference** or **forwarding** for things you want to move or perfectly forward, and avoid raw pointers unless you need optional/nullable parameters.

### Purposes

- To avoid unnecessary copies and thereby improve performance.
- To express intent clearly through the choice of parameter type.
- To prevent accidental modification of caller data.
- To enable move semantics where appropriate.
- To write safe, maintainable, and idiomatic C++ interfaces.

### Key Guidelines (Summary Table)

| Guideline | Summary | Recommended Passing |
|-----------|---------|---------------------|
| **F.15** | Prefer simple and conventional ways of passing information. | Start with the simple rules below. |
| **F.16** | For “in” parameters, pass cheaply-copied types by value and others by `const&`. | `int`, `double`, pointers, enums → by value; class types → `const&`. |
| **F.17** | For “in-out” parameters, pass by reference to non-`const`. | `T&`. |
| **F.18** | For “consume” parameters, take by value and `std::move` if the type is movable and cheap to move. | `T` by value + `std::move`. |
| **F.19** | Use a forwarding reference for forwarding parameters. | `T&&` + `std::forward<T>`. |
| **F.20** | For “out” output values, prefer return values to output parameters. | Return by value (RVO/NRVO). |
| **F.21** | To return multiple values, prefer returning a struct or tuple. | `struct` or `std::tuple`. |

**Additional Rules of Thumb**

- **Do not pass by `const&` when you need to modify** the argument; use `T&`.
- **Do not pass by `const&` small types** (`int`, `double`, `char`, pointers); by value is likely faster and simpler.
- **Do not pass raw pointers** unless you need to express “optional” (nullptr) or you are interfacing with C APIs.
- **Use `noexcept`** on move constructors and move assignment operators to enable efficient container operations.
- **Avoid `const T&&`** parameters; they rarely make sense (moving from a `const` rvalue is usually a copy).
- **Prefer `std::string_view`** (C++17) for read-only string parameters when you do not need a null-terminated string.
- **Prefer `std::span`** (C++20) for contiguous sequences when you need a view over an array.

### Annotated Code Examples

**Example 1: Choosing between by-value and `const&`**

```cpp
#include <iostream>
#include <string>

// Cheap to copy: by value
void print_int(int x) { std::cout << x << '\n'; }

// Expensive to copy: by const reference
void print_string(const std::string& s) { std::cout << s << '\n'; }

// In-out: by non-const reference
void append_world(std::string& s) { s += " world"; }

int main() {
    print_int(42);
    std::string msg = "hello";
    print_string(msg);
    append_world(msg);
    print_string(msg);
}
```

**Expected Output**
```
42
hello
hello world
```

**Why this output?**  
`print_int` takes by value (cheap). `print_string` takes by `const&` (no copy, read-only). `append_world` takes by `T&` (modifies the caller’s string).

**Example 2: Sink parameter (by-value + move)**

```cpp
#include <iostream>
#include <string>
#include <utility>

class Database {
    std::string connection_string;
public:
    // Sink parameter: take by value and move
    explicit Database(std::string conn) : connection_string(std::move(conn)) {
        std::cout << "Connected to: " << connection_string << '\n';
    }
};

int main() {
    std::string cs = "server=localhost;db=test";
    Database db(std::move(cs));      // move into parameter, then move into member
}
```

**Expected Output**
```
Connected to: server=localhost;db=test
```

**Why this output?**  
The by-value parameter `conn` is move-constructed from the temporary (or from `std::move(cs)`), and then the member is move-constructed from `conn`. No deep copy of the connection string occurs.

### Real-World Cases

- **Standard library interfaces** – `std::vector::push_back(const T&)` and `push_back(T&&)` follow these guidelines.
- **Factory functions** – `std::make_unique` uses perfect forwarding.
- **API design** – library authors use these rules to design intuitive and efficient interfaces.

---

## References

- Value categories – https://en.cppreference.com/w/cpp/language/value_category
- Reference declaration – https://en.cppreference.com/w/cpp/language/reference
- Pointer declaration – https://en.cppreference.com/w/cpp/language/pointer
- `std::forward` – https://en.cppreference.com/w/cpp/utility/forward
- Forwarding references – https://en.cppreference.com/w/cpp/language/reference#Forwarding_references
- `std::move` – https://en.cppreference.com/w/cpp/utility/move
- C++ Core Guidelines — F.15: Prefer simple and conventional ways of passing information – https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rf-conventional
- C++ Core Guidelines — F.16: For “in” parameters, pass cheaply-copied types by value and others by reference to `const` – https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rf-in
- C++ Core Guidelines — F.17: For “in-out” parameters, pass by reference to non-`const` – https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rf-inout
- C++ Core Guidelines — F.18: For “consume” parameters, pass by `&&` and `std::move` the parameter – https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rf-consume
- C++ Core Guidelines — F.19: Use a forwarding reference for forwarding parameters – https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rf-forward
- C++ Core Guidelines — F.20: For “out” output values, prefer return values to output parameters – https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rf-out
- C++ Core Guidelines — F.21: To return multiple “out” values, prefer returning a struct – https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rf-out-multi
- Technical Report on C++ Performance (N1336) — parameter passing discussion – https://www.rap.no/JTC1/SC22/WG21/docs/papers/2001/n1336.pdf
- Effective Modern C++ (Scott Meyers) — Item 24: Distinguish universal references from rvalue references; Item 25: Use `std::move` on rvalue references, `std::forward` on universal references. (Book reference; no URL.)

---

This cheat sheet provides a complete, structured reference for C++ parameter passing, covering every requested mechanism with definitions, syntax, annotated examples, and authoritative references. Each section follows the same uniform template to facilitate study and review.