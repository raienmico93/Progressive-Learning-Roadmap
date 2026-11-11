# Value Categories — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
A value category is a property of every C++ expression that describes whether the expression identifies a **persistent object** (has an identity and can be modified or addressed) or a **temporary value** (has no persistent identity and can be safely moved from).

**Technical Definition**
The C++ standard (since C++11) classifies every expression into one of three primary value categories — **lvalue**, **prvalue** (pure rvalue), and **xvalue** (expiring value) — and two composite categories — **glvalue** (generalized lvalue, the union of lvalue and xvalue) and **rvalue** (the union of prvalue and xvalue). These categories determine reference binding, overload resolution, move semantics, and template argument deduction. The classification is exhaustive, mutually exclusive at the primary level, and recursive (a parenthesized expression retains its operand's category).

**Beginner-Friendly Explanation**
Every expression in C++ is either an **lvalue** (something with a name and a home in memory) or an **rvalue** (a temporary value that doesn't have a lasting home). Since C++11, rvalues are further split into **prvalues** (pure temporaries like `42` or `x + y`) and **xvalues** (objects that are about to expire, like the result of `std::move(x)`). The labels “lvalue,” “prvalue,” and “xvalue” tell the compiler whether it should **copy** data or **move** it. Understanding value categories is the key to understanding move semantics, perfect forwarding, and reference binding.

### Key Characteristics

- **Identity vs. value**: glvalues (lvalues and xvalues) have an identity (an address); prvalues do not.
- **Movability**: rvalues (prvalues and xvalues) can be moved from; lvalues cannot (unless cast).
- **Reference binding**: `T&` binds to lvalues; `T&&` binds to rvalues; `const T&` binds to both.
- **Parenthesized expressions** retain the category of their operand.
- **Function calls**: a call returning `T` is a prvalue; returning `T&` is an lvalue; returning `T&&` is an xvalue.
- **Member access**: `a.m` where `a` is an lvalue and `m` is a non-static data member is an lvalue; if `a` is an rvalue, `a.m` is an xvalue.
- **Composite categories**: glvalue = lvalue ∪ xvalue; rvalue = prvalue ∪ xvalue; xvalue is the overlap.

### Prerequisites

- Understanding of expressions and statements.
- Familiarity with references (lvalue and rvalue).
- Knowledge of move semantics and `std::move`.
- Basic awareness of templates and type deduction.

### Related Programming Areas

- **Move semantics** – `std::move` produces xvalues; move constructors bind to rvalues.
- **Perfect forwarding** – `std::forward` preserves value categories.
- **Reference binding** – the compiler selects reference types based on value categories.
- **Overload resolution** – value categories participate in selecting overloads.
- **Template argument deduction** – forwarding references deduce based on value categories.
- **Copy elision** – C++17 redefined prvalues as initialization recipes.

### Core Concepts / Features

1. Lvalues
2. Rvalues
3. Xvalues
4. Prvalues
5. Glvalues
6. Value Categories (taxonomy and rules)
7. `std::move`
8. `std::forward`
9. Value Category Best Practices

---

## 1. Lvalues

### Definitions

**Core Definition**
An lvalue is an expression that identifies a **persistent object** with a name and a stable memory location; it can appear on the left-hand side of an assignment.

**Technical Definition**
An lvalue is a glvalue that is **not an xvalue**. It designates a function or an object with an identity (an address) that persists beyond the expression. Lvalues cannot be implicitly moved from; they must be explicitly cast to rvalues (via `std::move`) to enable move semantics. Examples include named variables, dereferenced pointers, array elements, members of lvalue objects, and functions returning lvalue references.

**Beginner-Friendly Explanation**
An lvalue is **something with a name and a home**. It's a persistent object that lives at a specific memory address. If you can take its address with `&` and it sticks around after the expression, it's an lvalue. You can assign to it, read from it, and refer to it by name.

### Purposes

- To identify persistent objects that can be modified and referred to.
- To serve as the left-hand operand of assignment.
- To bind to lvalue references (`T&`).
- To support operations that require an address (e.g., taking `&x`).

### Syntax Rules and Structure

**Common Lvalue Expressions**

| Expression | Category | Why |
|------------|----------|-----|
| `x` (named variable) | lvalue | Has identity. |
| `*p` (dereference) | lvalue | Refers to an object. |
| `arr[i]` (array element) | lvalue | Refers to an object. |
| `a.m` (member of lvalue) | lvalue | Refers to a member. |
| `f()` returning `T&` | lvalue | Returns a reference to an object. |
| `"literal"` (string literal) | lvalue | An array object. |
| `++x` (pre-increment) | lvalue | Returns the incremented object. |

**Syntax Rules**
- Lvalues bind to `T&` and `const T&`.
- Lvalues do **not** bind to `T&&` (unless cast with `std::move`).
- The address-of operator `&` requires an lvalue operand.
- Assignment requires a modifiable lvalue on the left.

**Constraints and Limitations**
- A `const` lvalue cannot be assigned to.
- Lvalues cannot be implicitly moved from; use `std::move` to enable moving.
- Some lvalues are not modifiable (e.g., `const` objects, string literals).

### Annotated Code Examples

**Example 1: Lvalues in action**

```cpp
#include <iostream>

int main() {
    int x = 10;           // x is an lvalue
    int& r = x;           // binds to lvalue
    *(&x) = 20;           // *(&x) is an lvalue
    std::cout << x << '\n';

    int arr[3] = {1, 2, 3};
    arr[1] = 99;          // arr[1] is an lvalue
    std::cout << arr[1] << '\n';
}
```

**Expected Output**
```
20
99
```

**Why this output?**  
`x`, `*(&x)`, and `arr[1]` are all lvalues referring to persistent objects. Assigning to them modifies those objects.

### Real-World Cases

- **Named variables** – `int x;` is an lvalue.
- **Array elements** – `arr[i]` is an lvalue.
- **Dereferenced pointers** – `*p` is an lvalue.
- **Members of objects** – `obj.member` is an lvalue if `obj` is an lvalue.
- **Functions returning references** – `T& f()` returns an lvalue.

---

## 2. Rvalues

### Definitions

**Core Definition**
An rvalue is an expression that is **not an lvalue** — it represents a temporary value or an object that is about to expire, and it can be safely moved from.

**Technical Definition**
An rvalue is either a **prvalue** (pure rvalue, no identity) or an **xvalue** (expiring value, has identity but can be moved from). Rvalues bind to rvalue references (`T&&`) and to `const` lvalue references (`const T&`). They are the primary source of move semantics: move constructors and move assignment operators accept rvalues.

**Beginner-Friendly Explanation**
An rvalue is **a temporary value** — something that doesn't have a lasting home. It's the right-hand side of an assignment, like `5`, `x + y`, or the result of a function returning by value. Since it's temporary, the compiler can safely **move** from it instead of copying.

### Purposes

- To represent temporary values that can be moved from.
- To bind to rvalue references, enabling move semantics.
- To participate in overload resolution (selecting move vs. copy).
- To support perfect forwarding.

### Syntax Rules and Structure

**Common Rvalue Expressions**

| Expression | Category | Why |
|------------|----------|-----|
| `42` (literal) | prvalue | No identity. |
| `x + y` | prvalue | Computed value. |
| `f()` returning `T` | prvalue | Temporary result. |
| `std::move(x)` | xvalue | Cast to expiring value. |
| `f()` returning `T&&` | xvalue | Returns expiring reference. |
| `static_cast<T&&>(x)` | xvalue | Cast to rvalue reference. |

**Syntax Rules**
- Rvalues bind to `T&&` and `const T&`.
- Rvalues do **not** bind to non-`const` `T&`.
- Move constructors and move assignment accept rvalues.
- `std::move` produces an rvalue (specifically, an xvalue).

**Constraints and Limitations**
- After moving from an rvalue, the source (if it has identity) is in a valid but unspecified state.
- A `const` rvalue binds to `const T&&`, which usually results in a copy (move constructors take non-`const` `T&&`).
- Rvalues cannot be the target of assignment (except via special member functions).

### Annotated Code Examples

**Example 1: Rvalues in action**

```cpp
#include <iostream>
#include <string>
#include <utility>

int main() {
    int a = 10, b = 20;
    int c = a + b;              // a + b is a prvalue (rvalue)
    std::cout << c << '\n';

    std::string s = "hello";
    std::string t = std::move(s);   // std::move(s) is an xvalue (rvalue)
    std::cout << "t: " << t << '\n';
}
```

**Expected Output**
```
30
t: hello
```

**Why this output?**  
`a + b` is a prvalue, copied into `c`. `std::move(s)` is an xvalue, allowing the move constructor to transfer the string's buffer to `t`.

### Real-World Cases

- **Literals** – `42`, `'a'`, `true` are prvalues.
- **Arithmetic results** – `x + y` is a prvalue.
- **Function return values** – `f()` returning `T` is a prvalue.
- **`std::move` results** – `std::move(x)` is an xvalue.
- **Temporary objects** – `T()` is a prvalue.

---

## 3. Xvalues

### Definitions

**Core Definition**
An xvalue (expiring value) is an expression that identifies an object with an identity that is **about to expire** and can be safely moved from.

**Technical Definition**
An xvalue is a glvalue that is **not an lvalue** — it has an identity but represents an object whose resources can be reused. Xvalues arise from `std::move`, casts to rvalue references (`static_cast<T&&>`), functions returning `T&&`, and certain member accesses. They bind to `T&&` and `const T&`, but not to `T&`. Xvalues are the bridge between glvalues (identity) and rvalues (movability).

**Beginner-Friendly Explanation**
An xvalue is an **object that still exists but is about to die**. It has a name (identity) but is marked as “expendable” — the compiler can steal its resources. `std::move(x)` turns the lvalue `x` into an xvalue, saying: “I'm done with `x`; you can move from it.”

### Purposes

- To mark objects as movable even though they have identity.
- To enable move semantics for named objects via `std::move`.
- To support functions returning rvalue references.
- To participate in perfect forwarding.

### Syntax Rules and Structure

**Common Xvalue Expressions**

| Expression | Category | Why |
|------------|----------|-----|
| `std::move(x)` | xvalue | Casts lvalue to rvalue reference. |
| `static_cast<T&&>(x)` | xvalue | Explicit cast to rvalue reference. |
| `f()` returning `T&&` | xvalue | Returns expiring reference. |
| `a.m` where `a` is an rvalue | xvalue | Member of expiring object. |

**Syntax Rules**
- Xvalues bind to `T&&` and `const T&`.
- Xvalues do **not** bind to non-`const` `T&`.
- `std::move` produces an xvalue.
- A function returning `T&&` produces an xvalue.

**Constraints and Limitations**
- Using an object after it has been moved from is a logic error (valid but unspecified state).
- `std::move` on a `const` object produces `const T&&` (usually copies).
- Xvalues have identity, so they can be addressed (unlike prvalues).

### Annotated Code Examples

**Example 1: Xvalue via `std::move`**

```cpp
#include <iostream>
#include <string>
#include <utility>

int main() {
    std::string a = "hello";
    std::string b = std::move(a);   // std::move(a) is an xvalue
    std::cout << "b: " << b << '\n';
    std::cout << "a size: " << a.size() << '\n';   // likely 0
}
```

**Expected Output**
```
b: hello
a size: 0
```

**Why this output?**  
`std::move(a)` casts the lvalue `a` to an xvalue, allowing the move constructor to steal its buffer. `a` is left in a valid but unspecified (likely empty) state.

**Example 2: Xvalue from a function returning `T&&`**

```cpp
#include <iostream>
#include <string>
#include <utility>

std::string&& get_rvalue(std::string&& s) {
    return std::move(s);   // returns an xvalue
}

int main() {
    std::string s = "world";
    std::string t = get_rvalue(std::move(s));   // xvalue
    std::cout << t << '\n';
}
```

**Expected Output**
```
world
```

**Why this output?**  
`get_rvalue` returns `std::string&&` (an xvalue), which binds to the move constructor of `t`.

### Real-World Cases

- **`std::move`** – the primary producer of xvalues.
- **`std::forward`** – conditionally produces xvalues.
- **Move constructors** – accept xvalues (and prvalues).
- **`&&`-qualified member functions** – can return xvalues.

---

## 4. Prvalues

### Definitions

**Core Definition**
A prvalue (pure rvalue) is an expression that **does not have an identity** and represents a computed value or a temporary object that can be used to initialize another object.

**Technical Definition**
A prvalue is an expression whose evaluation produces a value with no result object (until C++17) or an initialization recipe (since C++17). It is not a glvalue. Prvalues include literals, arithmetic results, function calls returning non-reference types, and `new` expressions. Since C++17, a prvalue is not an object but an instruction for initializing an object; this redefinition enables **guaranteed copy elision**.

**Beginner-Friendly Explanation**
A prvalue is a **pure temporary** — a value that doesn't live anywhere in particular until it's assigned or used. `42`, `x + y`, and the result of a function returning by value are prvalues. They don't have an address you can take, because they're just values, not objects with a home.

### Purposes

- To represent computed values and temporaries.
- To initialize objects (via copy/move elision in C++17).
- To participate in move semantics (move constructors accept prvalues).
- To support value semantics and expression evaluation.

### Syntax Rules and Structure

**Common Prvalue Expressions**

| Expression | Category | Why |
|------------|----------|-----|
| `42` | prvalue | Literal value. |
| `x + y` | prvalue | Computed result. |
| `f()` returning `T` | prvalue | Temporary result. |
| `T()` | prvalue | Temporary object. |
| `new T` | prvalue | Pointer value (until C++17). |
| `true`, `nullptr` | prvalue | Literal values. |

**Syntax Rules**
- Prvalues bind to `T&&` and `const T&`.
- Prvalues do **not** bind to non-`const` `T&`.
- You cannot take the address of a prvalue (until C++17, when temporary materialization occurs in some contexts).
- Since C++17, prvalues are initialization recipes; they are materialized into temporaries only when needed.

**Constraints and Limitations**
- Prvalues have no identity; they cannot be assigned to or modified in place.
- Before C++17, prvalues could be copied or moved into their destination; since C++17, they initialize directly (guaranteed elision).

### Annotated Code Examples

**Example 1: Prvalue literal and computed value**

```cpp
#include <iostream>

int main() {
    int a = 10, b = 20;
    int c = a + b;        // a + b is a prvalue
    std::cout << c << '\n';

    int d = 42;           // 42 is a prvalue
    std::cout << d << '\n';
}
```

**Expected Output**
```
30
42
```

**Why this output?**  
`a + b` and `42` are prvalues — computed values used to initialize `c` and `d`.

### Real-World Cases

- **Literals** – `42`, `'a'`, `true`, `nullptr`.
- **Arithmetic/logical results** – `x + y`, `a && b`.
- **Function return values** – `f()` returning `T`.
- **Temporary objects** – `T()`, `std::string("hello")`.

---

## 5. Glvalues

### Definitions

**Core Definition**
A glvalue (generalized lvalue) is any expression that has an **identity** — it is the union of lvalues and xvalues.

**Technical Definition**
A glvalue is an expression whose evaluation determines the identity of an object, bit-field, or function. Glvalues include both lvalues (which persist) and xvalues (which are about to expire). A glvalue can be converted to a prvalue through lvalue-to-rvalue conversion (for reads). Glvalues are the “addressable” expressions.

**Beginner-Friendly Explanation**
A glvalue is **anything with a home** — whether it's a permanent home (lvalue) or a home that's about to be abandoned (xvalue). It's the umbrella category for expressions that have an identity, meaning you can point to them or take their address.

### Purposes

- To unify lvalues and xvalues under a single category for rules that apply to identity-bearing expressions.
- To describe expressions that can be converted to prvalues via lvalue-to-rvalue conversion.
- To support reference binding rules (glvalues bind to `T&` or `T&&` depending on subcategory).
- To clarify the value category taxonomy.

### Syntax Rules and Structure

**Glvalue Composition**

| Category | Description |
|----------|-------------|
| glvalue | lvalue or xvalue |
| lvalue | glvalue that is not xvalue |
| xvalue | glvalue that is movable |

**Syntax Rules**
- Glvalues have identity (you can take their address, except for bit-fields).
- Glvalues can be converted to prvalues (lvalue-to-rvalue conversion).
- Glvalues bind to `T&` (if lvalue) or `T&&` (if xvalue).
- The `&` operator requires a glvalue operand.

**Constraints and Limitations**
- Glvalues are not a single type; they are a category. The compiler distinguishes lvalues from xvalues for binding purposes.
- Bit-fields are glvalues but cannot be bound to non-`const` references.

### Annotated Code Examples

**Example 1: Glvalue identity**

```cpp
#include <iostream>

int main() {
    int x = 10;           // x is an lvalue (glvalue)
    int& r = x;           // binds to glvalue
    int* p = &x;          // address of glvalue
    std::cout << *p << '\n';

    int y = std::move(x); // std::move(x) is an xvalue (glvalue)
    std::cout << y << '\n';
}
```

**Expected Output**
```
10
10
```

**Why this output?**  
`x` is an lvalue glvalue; `std::move(x)` is an xvalue glvalue. Both have identity.

### Real-World Cases

- **Reference binding** – glvalues bind to references based on subcategory.
- **Address-of operator** – requires a glvalue.
- **Lvalue-to-rvalue conversion** – reads a glvalue's value.

---

## 6. Value Categories (Taxonomy and Rules)

### Definitions

**Core Definition**
The value category taxonomy is the complete classification of C++ expressions into lvalues, prvalues, xvalues, glvalues, and rvalues, with rules governing conversion and binding.

**Technical Definition**
The C++11 taxonomy is:
- **lvalue**: has identity, not movable.
- **prvalue**: no identity, movable.
- **xvalue**: has identity, movable.
- **glvalue** = lvalue ∪ xvalue.
- **rvalue** = prvalue ∪ xvalue.

The categories are mutually exclusive at the primary level. They determine reference binding (`T&` → lvalue; `T&&` → rvalue; `const T&` → anything), overload resolution (move vs. copy), and template argument deduction.

**Beginner-Friendly Explanation**
Think of value categories as **three boxes** with two **overlapping labels**:
- **Box 1: lvalue** — has a name, can't be moved.
- **Box 2: prvalue** — no name, can be moved.
- **Box 3: xvalue** — has a name, can be moved.
- **Glvalue** = Box 1 + Box 3 (has a name).
- **Rvalue** = Box 2 + Box 3 (can be moved).

### Purposes

- To provide a complete, precise classification of expressions.
- To determine reference binding and overload resolution.
- To enable move semantics and perfect forwarding.
- To support generic programming (type deduction).

### Syntax Rules and Structure

**The Taxonomy**

| Category | Identity? | Movable? | Binds to `T&`? | Binds to `T&&`? | Binds to `const T&`? |
|----------|-----------|----------|----------------|-----------------|----------------------|
| lvalue | Yes | No | Yes | No | Yes |
| prvalue | No | Yes | No | Yes | Yes |
| xvalue | Yes | Yes | No | Yes | Yes |
| glvalue | Yes | Maybe | — | — | — |
| rvalue | Maybe | Yes | — | — | — |

**Common Expression Categories**

| Expression | Category |
|------------|----------|
| `x` (variable) | lvalue |
| `*p` | lvalue |
| `arr[i]` | lvalue |
| `a.m` (a is lvalue) | lvalue |
| `f()` returning `T` | prvalue |
| `42` | prvalue |
| `x + y` | prvalue |
| `std::move(x)` | xvalue |
| `static_cast<T&&>(x)` | xvalue |
| `f()` returning `T&&` | xvalue |

**Syntax Rules**
- Parenthesized expressions retain their operand's category.
- Function calls: return type determines category (`T` → prvalue; `T&` → lvalue; `T&&` → xvalue).
- Member access: `a.m` is lvalue if `a` is lvalue and `m` is a non-static data member; xvalue if `a` is rvalue.
- Conditional operator: category depends on operands.

**Constraints and Limitations**
- The categories are exhaustive; every expression has exactly one primary category.
- Bit-fields are glvalues but cannot bind to non-`const` references.
- C++17 redefined prvalues as initialization recipes, affecting copy elision.

### Annotated Code Examples

**Example 1: Category identification**

```cpp
#include <iostream>
#include <string>
#include <utility>

template <typename T>
constexpr const char* category() {
    if constexpr (std::is_lvalue_reference_v<T>) return "lvalue";
    else if constexpr (std::is_rvalue_reference_v<T>) return "xvalue";
    else return "prvalue";
}

void f(int&)  { std::cout << "int&\n"; }
void f(int&&) { std::cout << "int&&\n"; }

int main() {
    int x = 10;
    f(x);                 // lvalue → int&
    f(42);                // prvalue → int&&
    f(std::move(x));      // xvalue → int&&
}
```

**Expected Output**
```
int&
int&&
int&&
```

**Why this output?**  
`x` is an lvalue, `42` is a prvalue, and `std::move(x)` is an xvalue. The overloads are selected based on the argument's value category.

### Real-World Cases

- **Reference binding** – `T&` for lvalues, `T&&` for rvalues.
- **Overload resolution** – copy vs. move constructors.
- **Perfect forwarding** – `std::forward` preserves categories.
- **Type deduction** – `auto` and templates deduce based on categories.

---

## 7. `std::move`

### Definitions

**Core Definition**
`std::move` is a utility that **unconditionally casts its argument to an rvalue reference** (specifically, an xvalue), enabling move semantics.

**Technical Definition**
`std::move(x)` is equivalent to `static_cast<std::remove_reference_t<decltype(x)>&&>(x)`. It produces an xvalue. It does **not** move anything by itself; it merely makes the expression eligible for move constructors and move assignment. The actual move occurs when a move operation is selected.

**Beginner-Friendly Explanation**
`std::move` is a **label** that says: “I'm done with this object; you can treat it as a temporary and move from it.” It doesn't move anything — it just changes the value category from lvalue to xvalue, so the compiler picks the move constructor.

### Purposes

- To cast an lvalue to an xvalue, enabling move semantics.
- To indicate that an object can be moved from.
- To transfer ownership in move constructors and move assignment.
- To support move-only types.

### Syntax Rules and Structure

**General Syntax**
```cpp
#include <utility>
T&& moved = std::move(lvalue);   // produces an xvalue
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `std::move(x)` | Returns an xvalue (rvalue reference). |
| Result | Binds to `T&&`. |

**Syntax Rules**
- `std::move` is a cast, not an operation.
- Use it on objects you no longer need.
- Do not use it on `const` objects (produces `const T&&`, usually copies).
- Do not use it on objects you need afterward.

**Constraints and Limitations**
- `std::move` on a `const` object yields `const T&&`, which typically binds to copy constructors.
- After a move, the source is valid but unspecified.
- Using `std::move` in return statements (`return std::move(local);`) inhibits NRVO (F.48).

### Annotated Code Examples

**Example 1: `std::move` enabling move construction**

```cpp
#include <iostream>
#include <string>
#include <utility>

int main() {
    std::string a = "hello";
    std::string b = std::move(a);   // a is cast to xvalue
    std::cout << "b: " << b << '\n';
    std::cout << "a size: " << a.size() << '\n';   // likely 0
}
```

**Expected Output**
```
b: hello
a size: 0
```

**Why this output?**  
`std::move(a)` casts `a` to an xvalue, so the move constructor transfers the buffer to `b`, leaving `a` empty.

### Real-World Cases

- **Ownership transfer** – `std::unique_ptr<T> p2 = std::move(p1);`.
- **Container insertion** – `v.push_back(std::move(obj));`.
- **Swap** – `std::swap` uses `std::move`.

---

## 8. `std::forward`

### Definitions

**Core Definition**
`std::forward<T>(arg)` is a utility that **conditionally casts** its argument to an rvalue if `T` is not an lvalue reference type, preserving the original value category during perfect forwarding.

**Technical Definition**
`std::forward` is defined in `<utility>`. When `T` is an lvalue reference type, it returns an lvalue; otherwise, it returns an rvalue reference. It is used with forwarding references (`T&&` where `T` is deduced) to pass arguments to another function while preserving their value category. It is the mechanism that makes perfect forwarding possible.

**Beginner-Friendly Explanation**
`std::forward` is a **value-category restorer**. If the original argument was an lvalue, `std::forward` keeps it an lvalue; if it was an rvalue, it makes it an rvalue again. This lets wrapper functions pass arguments through without accidentally copying or moving them.

### Purposes

- To preserve value categories in perfect forwarding.
- To enable move semantics through wrapper functions.
- To avoid unnecessary copies in generic code.
- To implement factories and emplace operations.

### Syntax Rules and Structure

**General Syntax**
```cpp
#include <utility>
target(std::forward<T>(arg));
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `T` | The deduced template parameter. |
| `arg` | The forwarding reference parameter. |

**Syntax Rules**
- Use only with forwarding references (`T&&` where `T` is deduced).
- `T` must be the exact deduced type.
- `std::forward` is `noexcept`.
- Do not use `std::move` instead of `std::forward` in forwarding contexts.

**Constraints and Limitations**
- Using `std::forward` outside the forwarding-reference pattern may not behave as expected.
- Passing the wrong `T` can cause errors or incorrect behavior.

### Annotated Code Examples

**Example 1: `std::forward` preserving value category**

```cpp
#include <iostream>
#include <string>
#include <utility>

void inspect(const std::string&) { std::cout << "lvalue\n"; }
void inspect(std::string&&) { std::cout << "rvalue\n"; }

template <typename T>
void wrapper(T&& arg) {
    inspect(std::forward<T>(arg));   // preserve category
}

int main() {
    std::string s = "hello";
    wrapper(s);              // lvalue → lvalue overload
    wrapper("temporary");    // rvalue → rvalue overload
}
```

**Expected Output**
```
lvalue
rvalue
```

**Why this output?**  
`wrapper(s)` deduces `T = std::string&`, so `std::forward<std::string&>(s)` returns an lvalue. `wrapper("temporary")` deduces a non-reference type, so `std::forward` returns an rvalue.

### Real-World Cases

- **`std::make_unique` / `std::make_shared`** – forward constructor arguments.
- **`emplace_back`** – construct elements in place.
- **Generic wrappers** – logging, timing, and instrumentation.

---

## 9. Value Category Best Practices

### Definitions

**Core Definition**
Value category best practices are guidelines for correctly reasoning about and using lvalues, rvalues, xvalues, and prvalues, and for applying `std::move` and `std::forward` appropriately.

**Technical Definition**
These practices are anchored in the C++ Core Guidelines (especially **ES.56**, **F.48**, **F.15–F.21**), the standard's value category taxonomy, and community wisdom (Scott Meyers' universal references, Thomas Becker's value category explanations). Key principles: **do not use `std::move` on objects you still need**, **do not return `std::move(local)`**, **use `std::forward` for forwarding**, **understand reference binding**, and **prefer value semantics with move where appropriate**.

**Beginner-Friendly Explanation**
Best practices for value categories are simple: **know whether something is an lvalue or rvalue**, **use `std::move` only when you're done with an object**, **use `std::forward` in wrapper functions**, **don't move from `const` objects**, and **let the compiler elide copies**. These habits prevent subtle bugs and performance issues.

### Key Guidelines (Summary Table)

| Guideline | Summary | Recommendation |
|-----------|---------|----------------|
| **ES.56** | Write `std::move` only when you need to explicitly move to another scope. | Don't overuse `std::move`. |
| **F.48** | Don't `return std::move(local)`. | Prevents NRVO; pessimization. |
| **F.15–F.21** | Parameter passing depends on value category and ownership. | Use `const T&` for input, `T&&` for forwarding. |
| **Do not move from `const`** | `std::move` on `const` produces `const T&&` (usually copies). | Use non-`const` objects for moving. |
| **Do not use moved-from objects** | Valid but unspecified state. | Reassign or destroy only. |
| **Use `std::forward` in templates** | Preserves value categories. | `T&&` + `std::forward<T>`. |
| **Understand reference binding** | `T&` ← lvalue; `T&&` ← rvalue; `const T&` ← both. | Choose the right parameter type. |
| **Prefer value semantics** | Return by value; move/elision makes it efficient. | Don't return raw pointers/references unnecessarily. |

### Annotated Code Examples

**Example 1: Do not use `std::move` on objects you still need**

```cpp
#include <iostream>
#include <string>
#include <utility>

int main() {
    std::string a = "hello";
    std::string b = std::move(a);
    // std::cout << a;   // BAD: a is moved-from (unspecified)
    std::cout << "b: " << b << '\n';
}
```

**Expected Output**
```
b: hello
```

**Why this is best practice?**  
After `std::move(a)`, `a` is in a valid but unspecified state. Using it (other than to reassign or destroy) is a logic error.

**Example 2: Use `std::forward` for forwarding**

```cpp
#include <iostream>
#include <string>
#include <utility>

void target(const std::string& s) { std::cout << "lvalue\n"; }
void target(std::string&& s) { std::cout << "rvalue\n"; }

template <typename T>
void wrapper(T&& arg) {
    target(std::forward<T>(arg));   // correct: preserves category
    // target(std::move(arg));      // wrong: always moves
}

int main() {
    std::string s = "hello";
    wrapper(s);            // lvalue
    wrapper("temp");       // rvalue
}
```

**Expected Output**
```
lvalue
rvalue
```

**Why this is best practice?**  
`std::forward<T>` preserves the argument's value category, so lvalues stay lvalues and rvalues stay rvalues. Using `std::move` would incorrectly treat lvalues as rvalues.

### Real-World Cases

- **Standard library** – `make_unique`, `emplace_back`, and `push_back` follow these rules.
- **Code review** – ES.56, F.48, and F.15–F.21 are common checkpoints.
- **Static analysis** – clang-tidy flags pessimizing moves and unnecessary `std::move`.

---

## References

- Value categories (cppreference) – https://en.cppreference.com/w/cpp/language/value_category
- Value categories (C++ standard draft, [basic.lval]) – https://eel.is/c++draft/basic.lval
- `std::move` (cppreference) – https://en.cppreference.com/w/cpp/utility/move
- `std::forward` (cppreference) – https://en.cppreference.com/w/cpp/utility/forward
- Reference declaration (cppreference) – https://en.cppreference.com/w/cpp/language/reference
- C++ Core Guidelines — ES.56: Write `std::move()` only when you need to explicitly move an object to another scope – https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Res-move
- C++ Core Guidelines — F.48: Don't `return std::move(local)` – https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rf-return-move-local
- C++ Core Guidelines — F.15: Prefer simple and conventional ways of passing information – https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rf-conventional
- C++ Core Guidelines — F.16: For “in” parameters, pass cheaply-copied types by value and others by reference to `const` – https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rf-in
- C++ Core Guidelines — F.18: For “consume” parameters, pass by `&&` and `std::move` the parameter – https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rf-consume
- C++ Core Guidelines — F.19: Use a forwarding reference for forwarding parameters – https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rf-forward
- “Universal References in C++11” (Scott Meyers, isocpp.org) – https://isocpp.org/blog/2012/11/universal-references-in-c11-scott-meyers
- “On the Tension Between Object-Oriented and Generic Programming in C++” (Thomas Becker) – http://thbecker.net/articles/rvalue_references/section_01.html
- Stack Overflow — “What are rvalues, lvalues, xvalues, glvalues, and prvalues?” – https://stackoverflow.com/questions/3601602/what-are-rvalues-lvalues-xvalues-glvalues-and-prvalues
- Microsoft Learn — Lvalues and rvalues (C++) – https://learn.microsoft.com/en-us/cpp/cpp/lvalues-and-rvalues-visual-cpp
- cppreference — Value categories (detailed examples) – https://en.cppreference.com/w/cpp/language/value_category

---

This cheat sheet provides a complete, structured reference for C++ value categories, covering lvalues, rvalues, xvalues, prvalues, glvalues, the full taxonomy, `std::move`, `std::forward`, and best practices. Each section follows a uniform template with definitions, purposes, syntax rules, annotated examples, expected outputs, and authoritative references.