# C++ Function Objects — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
A function object (also called a *functor*) is any object for which the function-call operator `operator()` is defined, allowing the object to be invoked with the same syntax as a regular function.

**Technical Definition**
A function object is an instance of a class or struct type that overloads `operator()`. C++ provides many built-in function objects in `<functional>`, as well as support for creating and manipulating new ones. Function objects can carry state (unlike plain function pointers), can be inlined by the compiler (unlike function pointers passed indirectly), and serve as the primary mechanism for customising the behaviour of standard algorithms.

**Beginner-Friendly Explanation**
A function object is like a "smart function" — an object that can be called as if it were a function. Because it is an object, it can store data (state) and carry configuration information with it. Lambdas are a convenient shorthand for writing function objects.

### Key Characteristics

- **Callable via `operator()`** – the defining feature of any function object.
- **Can carry state** – member variables allow the operation to depend on stored data.
- **Cheap to copy** – the C++ Core Guidelines recommend that function objects should be cheap to copy and therefore passed by value.
- **Inlinable** – because the type is known at compile time (when templates are used), the compiler can inline the call, often outperforming function pointers.
- **Composable** – function objects can be stored, copied, and passed to algorithms and higher-order functions.
- **Lambdas are function objects** – a lambda expression generates an anonymous function object.

### Prerequisites

- Basic C++ syntax (classes, structs, operators).
- Understanding of function overloading.
- Familiarity with templates (for generic function objects).
- Knowledge of lambda expressions (as a shorthand for function objects).
- Awareness of STL algorithms that accept callables.

### Related Programming Areas

- **STL algorithms** – algorithms such as `std::sort`, `std::find_if`, and `std::transform` accept function objects to customise behaviour.
- **Lambda expressions** – a syntactic shortcut for creating function objects.
- **`std::function`** – a type-erased wrapper for storing arbitrary callables.
- **Functional programming** – higher-order functions and function composition.
- **Policy-based design** – using function objects as compile-time policies.

### Core Concepts / Features

1. Function objects / Functors
2. `operator()`
3. Predefined function objects
   - Arithmetic operations (`std::plus`, `std::minus`, `std::multiplies`, `std::divides`, `std::modulus`, `std::negate`)
   - Comparison operations (`std::equal_to`, `std::not_equal_to`, `std::greater`, `std::less`, `std::greater_equal`, `std::less_equal`)
   - Logical operations (`std::logical_and`, `std::logical_or`, `std::logical_not`)
   - Bitwise operations (`std::bit_and`, `std::bit_or`, `std::bit_xor`, `std::bit_not`)
4. Function object best practices

---

## 1. Function Objects / Functors

### Definitions

**Core Definition**
A function object (functor) is an object that overloads `operator()` and can therefore be used where a function is expected.

**Technical Definition**
A type `F` is a function object if `f(args...)` is a valid expression for an instance `f` of type `F`. The standard library defines the `FunctionObject` named requirement. Function objects are used in place of pointers to functions as arguments to templated algorithms, because the interface for each algorithmic template can accept either an object with `operator()` defined, or a pointer to a function. The C++ Core Guidelines state: "Use function objects to pass operations to algorithms" (T.40), because function objects can carry more information through an interface than a plain pointer to function, and generally give better performance.

**Beginner-Friendly Explanation**
A functor is a class or struct that you can "call" like a function because it has `operator()` defined. It behaves like a function but can also hold data, which makes it more powerful and flexible than a plain function pointer.

### Purposes

- To pass custom operations to algorithms (sorting, searching, transforming).
- To carry state alongside the operation (e.g., a threshold value).
- To enable inlining and performance optimisation compared with function pointers.
- To serve as the foundation for lambdas and `std::function`.
- To support generic, policy-based programming.

### Syntax Rules and Structure

**General Syntax**
```cpp
struct FunctorName {
    ReturnType operator()(ParameterList) const {
        // operation
    }
};
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `struct` / `class` | The functor type. |
| `operator()` | The function-call operator; defines the operation. |
| `const` | Usually declared `const` so the functor can be called on const objects. |
| Member variables | Optional state that the operation can use. |

**Syntax Rules**
- `operator()` may have any signature and may be overloaded.
- Function objects are typically passed by value to algorithms (Core Guidelines F.16 and T.40).
- A function object is cheap to copy if it has no expensive members.
- Lambdas are automatically generated function objects.

**Constraints and Limitations**
- A function object with mutable state must be used carefully in algorithms that copy the functor (each copy has its own state).
- Function objects that are expensive to copy should generally not be passed by value (prefer `std::ref` or capture by reference in a lambda).

### Annotated Code Examples

**Example 1: A simple functor**

```cpp
#include <iostream>

struct Square {
    int operator()(int x) const {
        return x * x;
    }
};

int main() {
    Square sq;
    std::cout << sq(5) << '\n';            // 25
}
```

**Expected Output**
```
25
```

**Why this output?**  
`sq(5)` invokes `Square::operator()(5)`, which returns `5 * 5`.

**Example 2: A stateful functor**

```cpp
#include <iostream>

struct GreaterThan {
    int threshold;
    explicit GreaterThan(int t) : threshold(t) {}
    bool operator()(int x) const {
        return x > threshold;
    }
};

int main() {
    GreaterThan gt(7);
    std::cout << std::boolalpha << gt(10) << '\n'; // true
    std::cout << gt(3) << '\n';                     // false
}
```

**Expected Output**
```
true
false
```

**Why this output?**  
The functor stores the threshold `7` and compares each argument against it.

### Real-World Cases

- **Custom sorting** – `std::sort(v.begin(), v.end(), MyComparator{})`.
- **Filtering** – `std::find_if` with a predicate that carries state.
- **Event handling** – callback objects that store context.
- **Policy classes** – compile-time strategy selection.

---

## 2. `operator()`

### Definitions

**Core Definition**
`operator()` is the function-call operator, the member function that makes a class or struct callable like a function.

**Technical Definition**
The function-call operator is a member function named `operator()` that can be overloaded with any number of parameters and any return type. When an object `obj` is used in the expression `obj(args...)`, the compiler translates this to `obj.operator()(args...)`.

**Beginner-Friendly Explanation**
`operator()` is the special function that lets an object behave like a function. When you write `myObject(5)`, C++ actually calls `myObject.operator()(5)`.

### Purposes

- To make a class or struct callable.
- To overload the call syntax for different argument lists.
- To enable function-object semantics in user-defined types.
- To support generic programming with callables.

### Syntax Rules and Structure

**General Syntax**
```cpp
ReturnType operator()(ParameterList) cv-qualifiers {
    // body
}
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `ReturnType` | The result type of the call. |
| `ParameterList` | Zero or more parameters. |
| `cv-qualifiers` | Typically `const`; may also be `&`, `&&`, `volatile`. |

**Syntax Rules**
- `operator()` may be overloaded; overload resolution applies.
- It may be a non-static member function (cannot be `static`).
- It may be `const`, `constexpr`, `noexcept`, or `virtual`.

**Constraints and Limitations**
- An object without `operator()` is not callable.
- The operator cannot be declared as a free function; it must be a member of a class or struct.

### Annotated Code Example

```cpp
#include <iostream>

struct Adder {
    int base;
    explicit Adder(int b) : base(b) {}

    int operator()(int x) const { return base + x; }
    int operator()(int x, int y) const { return base + x + y; }
};

int main() {
    Adder add10(10);
    std::cout << add10(5) << '\n';         // 15
    std::cout << add10(5, 3) << '\n';      // 18
}
```

**Expected Output**
```
15
18
```

**Why this output?**  
`operator()` is overloaded for one and two arguments; both calls add the stored `base` to the arguments.

### Real-World Cases

- **Callable objects** – any type that must be used as a callback.
- **Operator overloading** – `operator()` is one of the many overloadable operators.
- **Lambdas** – the compiler-generated closure type overloads `operator()`.

---

## 3. Predefined Function Objects

### Definitions

**Core Definition**
Predefined function objects are standard library class templates in `<functional>` that implement common operations (arithmetic, comparison, logical, bitwise) as callable objects.

**Technical Definition**
The standard library provides function objects representing arithmetic operations (`plus`, `minus`, `multiplies`, `divides`, `modulus`, `negate`), comparisons (`equal_to`, `not_equal_to`, `greater`, `less`, `greater_equal`, `less_equal`), logical operations (`logical_and`, `logical_or`, `logical_not`), and bitwise operations (`bit_and`, `bit_or`, `bit_xor`, `bit_not`). Since C++14, these templates have a `void` specialisation that deduces parameter and return types (transparent function objects), enabling heterogeneous comparisons and arithmetic.

**Beginner-Friendly Explanation**
The standard library gives you ready-made function objects for common operations like addition, comparison, and logic. You can pass them directly to algorithms instead of writing your own functor.

### Purposes

- To provide ready-made callables for standard algorithms.
- To avoid writing boilerplate functors.
- To enable transparent (heterogeneous) operations via the `void` specialisation.
- To serve as building blocks for generic code.

---

### 3.1 Arithmetic Operations

#### Definitions

**Core Definition**
Arithmetic function objects perform addition, subtraction, multiplication, division, modulus, and negation.

**Technical Definition**
Each is a class template `std::X<T>` whose `operator()` applies the corresponding arithmetic operator to two instances of `T` (binary) or one instance (unary). The `void` specialisation deduces argument and return types.

| Functor | Operation | Signature (C++14 `void` spec.) |
|---------|-----------|-------------------------------|
| `std::plus` | `x + y` | `template<> struct plus<void>` |
| `std::minus` | `x - y` | `template<> struct minus<void>` |
| `std::multiplies` | `x * y` | `template<> struct multiplies<void>` |
| `std::divides` | `x / y` | `template<> struct divides<void>` |
| `std::modulus` | `x % y` | `template<> struct modulus<void>` |
| `std::negate` | `-x` | `template<> struct negate<void>` |

*Source: cppreference function objects overview.*

#### Annotated Code Example

```cpp
#include <functional>
#include <iostream>

int main() {
    std::cout << std::plus<>{}(3, 4) << '\n';       // 7
    std::cout << std::minus<>{}(10, 3) << '\n';     // 7
    std::cout << std::multiplies<>{}(3, 4) << '\n'; // 12
    std::cout << std::negate<>{}(5) << '\n';        // -5
}
```

**Expected Output**
```
7
7
12
-5
```

**Why this output?**  
Each functor forwards to the corresponding built-in operator.

---

### 3.2 Comparison Operations

#### Definitions

**Core Definition**
Comparison function objects perform equality, inequality, and ordering comparisons.

**Technical Definition**
Each is a class template `std::X<T>` returning `bool`. The `void` specialisation deduces argument types. For pointer types, `std::greater`, `std::less`, etc., produce an implementation-defined strict total order, even when the built-in operator would be undefined for unrelated pointers.

| Functor | Operation |
|---------|-----------|
| `std::equal_to` | `x == y` |
| `std::not_equal_to` | `x != y` |
| `std::greater` | `x > y` |
| `std::less` | `x < y` |
| `std::greater_equal` | `x >= y` |
| `std::less_equal` | `x <= y` |

*Source: cppreference function objects overview.*

#### Annotated Code Example

```cpp
#include <functional>
#include <iostream>

int main() {
    std::cout << std::boolalpha;
    std::cout << std::equal_to<>{}(3, 3) << '\n';    // true
    std::cout << std::not_equal_to<>{}(3, 4) << '\n'; // true
    std::cout << std::greater<>{}(5, 3) << '\n';     // true
    std::cout << std::less<>{}(5, 3) << '\n';        // false
}
```

**Expected Output**
```
true
true
true
false
```

---

### 3.3 Logical Operations

#### Definitions

**Core Definition**
Logical function objects perform `&&`, `||`, and `!` operations.

| Functor | Operation |
|---------|-----------|
| `std::logical_and` | `x && y` |
| `std::logical_or` | `x \|\| y` |
| `std::logical_not` | `!x` |

*Source: cppreference function objects overview.*

#### Annotated Code Example

```cpp
#include <functional>
#include <iostream>

int main() {
    std::cout << std::boolalpha;
    std::cout << std::logical_and<>{}(true, false) << '\n'; // false
    std::cout << std::logical_or<>{}(true, false) << '\n';  // true
    std::cout << std::logical_not<>{}(true) << '\n';        // false
}
```

**Expected Output**
```
false
true
false
```

---

### 3.4 Bitwise Operations

#### Definitions

**Core Definition**
Bitwise function objects perform `&`, `|`, `^`, and `~` operations.

| Functor | Operation | Signature (C++14 `void` spec.) |
|---------|-----------|-------------------------------|
| `std::bit_and` | `x & y` | `template<> struct bit_and<void>` |
| `std::bit_or` | `x \| y` | `template<> struct bit_or<void>` |
| `std::bit_xor` | `x ^ y` | `template<> struct bit_xor<void>` |
| `std::bit_not` | `~x` | `template<> struct bit_not<void>` |

*Source: cppreference function objects overview, std::bit_and documentation.*

#### Annotated Code Example

```cpp
#include <functional>
#include <iostream>
#include <bitset>

int main() {
    std::cout << std::bit_and<>{}(0b1100, 0b1010) << '\n'; // 8 (1000)
    std::cout << std::bit_or<>{}(0b1100, 0b1010) << '\n';  // 14 (1110)
    std::cout << std::bit_xor<>{}(0b1100, 0b1010) << '\n'; // 6 (0110)
    std::cout << std::bit_not<>{}(0b1111) << '\n';          // -16 (for int)
}
```

**Expected Output** (assuming 32-bit `int`)
```
8
14
6
-16
```

**Why this output?**  
Each functor applies the corresponding bitwise operator.

---

## 4. Function Object Best Practices

### Definitions

**Core Definition**
Function object best practices are guidelines for writing and using functors correctly, efficiently, and idiomatically.

**Technical Definition**
These practices are grounded in the C++ Core Guidelines, particularly T.40 ("Use function objects to pass operations to algorithms") and F.16 ("For 'in' parameters, pass cheaply-copied types by value"), and community consensus on functor design.

**Beginner-Friendly Explanation**
Best practices for function objects are simple: make them cheap to copy, pass them by value, prefer lambdas for short operations, and use standard functors when they fit. Avoid function pointers when a function object would be more expressive or faster.

### Key Guidelines

| Guideline | Summary |
|-----------|---------|
| **T.40: Use function objects** | Prefer function objects over function pointers for operations passed to algorithms. |
| **Keep functors cheap to copy** | Pass them by value; avoid expensive state or resources in functors. |
| **Prefer lambdas for short operations** | Lambdas are function objects and are more concise for inline use. |
| **Use standard functors when they fit** | `std::plus`, `std::greater`, etc., are well-tested and transparent. |
| **Use `void` specialisations for genericity** | `std::plus<>` deduces types and supports heterogeneous operations. |
| **Avoid `std::function` in hot paths** | It introduces type erasure and virtual-like dispatch overhead. |
| **Beware of mutable state in copied functors** | Algorithms copy functors; each copy has its own state. |
| **Mark `operator()` `const`** | Ensures the functor can be called on const objects. |

### Annotated Code Example

```cpp
#include <algorithm>
#include <functional>
#include <vector>
#include <iostream>

int main() {
    std::vector<int> v = {3, 1, 4, 1, 5};

    // Prefer standard functor with transparent specialisation
    std::sort(v.begin(), v.end(), std::greater<>{});

    // Use lambda for a short custom operation
    std::for_each(v.begin(), v.end(), [](int x) { std::cout << x << ' '; });
}
```

**Expected Output**
```
5 4 3 1 1
```

**Why this output?**  
`std::greater<>` sorts in descending order; the lambda prints each element.

### Real-World Cases

- **Algorithm customisation** – passing comparators and predicates.
- **Callback registration** – storing function objects for later invocation.
- **Policy classes** – compile-time behaviour selection.
- **Performance-critical code** – preferring functors over `std::function` when inlining matters.

---

## References

- Function objects — cppreference – https://en.cppreference.com/w/cpp/utility/functional
- `std::plus` — cppreference – https://en.cppreference.com/w/cpp/utility/functional/plus
- `std::greater` — cppreference – https://en.cppreference.com/w/cpp/utility/functional/greater
- `std::bit_and` — cppreference – https://en.cppreference.com/w/cpp/utility/functional/bit_and
- C++ Core Guidelines — T.40: Use function objects to pass operations to algorithms – https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rt-fo
- C++ Core Guidelines — F.16: For "in" parameters, pass cheaply-copied types by value – https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rf-in
- Function Objects (ICTP PGI documentation) – https://icts.ictp.it/manuals/pgi/pgC_lib/ios_8948.htm/Fun_4316.htm
- When should I use `std::function` vs. a template? — Stack Overflow / ISO C++ – https://isocpp.org/blog/2013/02/quick-q-when-should-i-use-stdfunction-vs-make-my-function-a-template
- Why functors are preferred over function pointers — learn_cpp (Dr-Sergey) – https://raw.githubusercontent.com/Dr-Sergey/learn_cpp/master/why-functors.md

---

This cheat sheet provides a comprehensive, structured reference for C++ function objects, covering functors, `operator()`, all predefined function objects across arithmetic, comparison, logical, and bitwise categories, and best practices, with annotated code examples, expected outputs, real-world cases, and authoritative references.