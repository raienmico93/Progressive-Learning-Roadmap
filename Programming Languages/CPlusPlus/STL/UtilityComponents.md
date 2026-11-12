# C++ Utility Components — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
Utility components are a collection of standard library class templates and function templates—defined primarily in `<utility>`, `<tuple>`, `<optional>`, `<variant>`, `<any>`, `<string_view>`, `<span>`, and `<functional>`—that provide reusable building blocks for common programming tasks such as storing heterogeneous values, representing optionality, type erasure, non-owning views, and callable wrappers.

**Technical Definition**
The C++ standard library's utility components are templates that model common abstractions: `std::pair` and `std::tuple` provide fixed-size heterogeneous collections; `std::optional`, `std::variant`, and `std::any` provide type-safe discriminated unions and type-erased storage; `std::string_view` and `std::span` provide non-owning views over contiguous sequences; `std::function` provides a polymorphic function wrapper; and `std::reference_wrapper` (with `std::ref`/`std::cref`) wraps references in copyable objects. These components are designed to compose with each other and with the wider standard library, including containers, algorithms, and iterators.

**Beginner-Friendly Explanation**
Think of utility components as a set of "Lego bricks" that solve recurring problems. `pair` and `tuple` group values together. `optional` says "maybe there's a value, maybe not." `variant` says "exactly one of several types." `any` says "any type at all." `string_view` and `span` let you look at data without copying it. `function` stores any callable. `reference_wrapper` lets you put references in containers. These tools save you from writing boilerplate and help express intent clearly.

### Key Characteristics

- **Composable** – many utility components are designed to work together (e.g., `tuple` of `optional`s, `span` of `string_view`s).
- **Zero-overhead where possible** – `pair`, `tuple`, `string_view`, and `span` are typically trivially copyable and impose little or no runtime cost.
- **Type-safe** – `optional`, `variant`, and `any` provide safe alternatives to raw unions and `void*`.
- **Non-owning views** – `string_view` and `span` do not own their data; they must not outlive the underlying storage.
- **Copyable references** – `reference_wrapper` makes references storable in containers and bindable in `std::bind` / `std::thread`.

### Prerequisites

- Basic C++ syntax (variables, types, templates, references).
- Familiarity with standard containers and iterators.
- Understanding of move semantics and RAII.
- Knowledge of lambda expressions and callables.

### Related Programming Areas

- **Containers** – `pair` and `tuple` are used as container element types; `optional`, `variant`, and `any` can be stored in containers.
- **Algorithms** – `tuple` and `pair` support lexicographical comparison and structured bindings.
- **Type erasure** – `std::function` and `std::any` are type-erasure mechanisms.
- **Views and ranges** – `string_view` and `span` are non-owning views, precursors to the C++20 ranges library.
- **Functional programming** – `std::function` and `reference_wrapper` enable higher-order programming.

### Core Concepts / Features

1. `std::pair`
2. `std::tuple`
3. `std::optional`
4. `std::variant`
5. `std::any`
6. `std::string_view`
7. `std::span`
8. `std::function`
9. `std::reference_wrapper`
10. `std::ref` and `std::cref`
11. Utility best practices

---

## 1. `std::pair`

### Definitions

**Core Definition**
`std::pair` is a class template that provides a way to store two heterogeneous objects as a single unit.

**Technical Definition**
`std::pair<T1, T2>` is defined in `<utility>`. It is a specific case of `std::tuple` with two elements. Its members are `first` (of type `T1`) and `second` (of type `T2`). If neither `T1` nor `T2` is a potentially cv-qualified class type with a non-trivial destructor, or an array thereof, the destructor of `pair` is trivial .

**Beginner-Friendly Explanation**
A `pair` is a simple container that holds exactly two values, which can be of different types. It is useful for returning two values from a function or storing key-value associations.

### Purposes

- To group two related values of possibly different types.
- To return multiple values from a function without defining a custom struct.
- To serve as the element type of associative containers (`std::map`, `std::unordered_map`).
- To support structured bindings and lexicographical comparison.

### Syntax Rules and Structure

**General Syntax**
```cpp
#include <utility>
std::pair<T1, T2> p;
std::pair<T1, T2> p = {value1, value2};
auto p = std::make_pair(value1, value2);
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `first` | The first element of type `T1`. |
| `second` | The second element of type `T2`. |
| `first_type` / `second_type` | The types of the elements. |

**Syntax Rules**
- `pair` is an aggregate-like class with public `first` and `second` members.
- `std::make_pair` deduces the types from its arguments.
- Structured bindings (`auto [a, b] = p;`) work with `pair`.

**Constraints and Limitations**
- `pair` holds exactly two elements; use `tuple` for more.
- No custom constructors beyond the standard ones.

### Annotated Code Examples

**Example 1: Basic pair usage**

```cpp
#include <utility>
#include <iostream>
#include <string>

int main() {
    std::pair<std::string, int> p = {"Alice", 30};
    std::cout << p.first << ": " << p.second << '\n';
}
```

**Expected Output**
```
Alice: 30
```

**Why this output?**  
`first` holds the name and `second` holds the age.

**Example 2: Returning a pair from a function**

```cpp
#include <utility>
#include <iostream>

std::pair<bool, int> divide(int a, int b) {
    if (b == 0) return {false, 0};
    return {true, a / b};
}

int main() {
    auto [ok, result] = divide(10, 3);
    if (ok) std::cout << "Result: " << result << '\n';
}
```

**Expected Output**
```
Result: 3
```

**Why this output?**  
The `pair` communicates success/failure and the quotient via structured bindings.

### Real-World Cases

- **Map elements** – `std::map<K, V>` stores `std::pair<const K, V>`.
- **Returning two values** – e.g., `std::pair<bool, T>` for result-and-value.
- **Coordinate pairs** – `std::pair<int, int>` for 2D points.

---

## 2. `std::tuple`

### Definitions

**Core Definition**
`std::tuple` is a class template that represents a fixed-size collection of heterogeneous values.

**Technical Definition**
`std::tuple<Types...>` is defined in `<tuple>` (since C++11). It is a generalization of `std::pair`. If `std::is_trivially_destructible<Ti>::value` is true for every `Ti` in `Types`, the destructor of `tuple` is trivial . Elements are accessed via `std::get<I>` or `std::get<T>` (if unique), and the tuple supports structured bindings (C++17).

**Beginner-Friendly Explanation**
A `tuple` is like a `pair` but for any number of values of different types. It is useful when you need to group several values together without defining a struct.

### Purposes

- To store a fixed-size heterogeneous collection.
- To return multiple values from a function (generalizing `pair`).
- To support metaprogramming via `std::tuple_size`, `std::tuple_element`, and `std::apply`.
- To serve as a building block for other utilities.

### Syntax Rules and Structure

**General Syntax**
```cpp
#include <tuple>
std::tuple<T1, T2, T3> t = {a, b, c};
auto t = std::make_tuple(a, b, c);
auto value = std::get<0>(t);           // index-based access
auto value = std::get<int>(t);         // type-based access (if unique)
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `std::get<I>` | Accesses the element at index `I`. |
| `std::tuple_size` | Compile-time size of the tuple. |
| `std::tuple_element` | Compile-time type of an element. |
| `std::apply` | Applies a function to the tuple’s elements (C++17). |

**Syntax Rules**
- `tuple` can hold zero or more elements.
- Structured bindings (`auto [a, b, c] = t;`) work since C++17.
- `std::tie` creates a tuple of lvalue references (for unpacking).

**Constraints and Limitations**
- Elements are not named; access is by index or type.
- Compile-time index access is required (`std::get<I>`), not runtime indexing.

### Annotated Code Examples

**Example 1: Basic tuple**

```cpp
#include <tuple>
#include <iostream>
#include <string>

int main() {
    std::tuple<int, std::string, double> t = {1, "hello", 3.14};
    std::cout << std::get<0>(t) << ' '
              << std::get<1>(t) << ' '
              << std::get<2>(t) << '\n';
}
```

**Expected Output**
```
1 hello 3.14
```

**Why this output?**  
`std::get<I>` accesses each element by index.

**Example 2: Structured bindings and `std::apply`**

```cpp
#include <tuple>
#include <iostream>

int main() {
    auto t = std::make_tuple(1, 2.5, "three");
    auto [a, b, c] = t;                    // structured bindings
    std::cout << a << ' ' << b << ' ' << c << '\n';

    std::apply([](auto... args) {
        ((std::cout << args << ' '), ...); // fold expression
    }, t);
}
```

**Expected Output**
```
1 2.5 three
1 2.5 three
```

**Why this output?**  
Structured bindings unpack the tuple; `std::apply` calls a function with all elements as arguments.

### Real-World Cases

- **Multi-value returns** – returning `std::tuple<int, std::string, bool>`.
- **Database rows** – representing rows of heterogeneous columns.
- **Metaprogramming** – `std::apply` for generic invocation.

---

## 3. `std::optional`

### Definitions

**Core Definition**
`std::optional` is a class template that manages an optional contained value—a value that may or may not be present.

**Technical Definition**
`std::optional<T>` (since C++17) is defined in `<optional>`. It either contains a value of type `T` or is empty. When converted to `bool`, it returns `true` if it contains a value and `false` otherwise. The contained value is nested within the `optional` object, so `optional` models an object, not a pointer. There are no optional references, functions, arrays, or `void` .

**Beginner-Friendly Explanation**
`optional` is a box that may or may not contain something. It is a clean way to express "maybe there is a value" instead of using a null pointer or a boolean flag.

### Purposes

- To represent a value that may be absent without using pointers or sentinel values.
- To return a value from a function that may fail (instead of `std::pair<T, bool>`).
- To store optional members in classes.
- To avoid dynamic allocation for optional values.

### Syntax Rules and Structure

**General Syntax**
```cpp
#include <optional>
std::optional<T> opt;                  // empty
std::optional<T> opt = value;          // contains value
opt = std::nullopt;                    // reset to empty
if (opt) { /* has value */ }
T val = opt.value();                   // throws if empty
T val = *opt;                          // unchecked access
```

**Component Breakdown**

| Operation | Description |
|-----------|-------------|
| `has_value()` / `operator bool` | Checks presence. |
| `value()` | Returns the value; throws `std::bad_optional_access` if empty. |
| `operator*` / `operator->` | Unchecked access. |
| `value_or(default)` | Returns the value or a default. |
| `reset()` | Makes the optional empty. |

**Syntax Rules**
- `optional` is not a pointer; the contained object lives inside the optional.
- `std::nullopt` represents the empty state.
- `optional<T>` requires `T` to be `Destructible`.

**Constraints and Limitations**
- No optional references (use `std::optional<std::reference_wrapper<T>>` as a workaround).
- Slight space overhead (typically one byte plus padding for the boolean flag).

### Annotated Code Examples

**Example 1: Returning an optional value**

```cpp
#include <optional>
#include <iostream>

std::optional<int> parse_positive(int x) {
    if (x > 0) return x;
    return std::nullopt;
}

int main() {
    auto result = parse_positive(5);
    if (result) std::cout << "Value: " << *result << '\n';

    auto none = parse_positive(-1);
    std::cout << "Empty: " << none.has_value() << '\n';
}
```

**Expected Output**
```
Value: 5
Empty: 0
```

**Why this output?**  
`parse_positive` returns an optional; `*result` accesses the value, and `has_value()` reports emptiness.

**Example 2: `value_or`**

```cpp
#include <optional>
#include <iostream>

int main() {
    std::optional<int> opt;
    std::cout << opt.value_or(42) << '\n'; // 42
    opt = 10;
    std::cout << opt.value_or(42) << '\n'; // 10
}
```

**Expected Output**
```
42
10
```

**Why this output?**  
`value_or` returns the contained value if present, otherwise the default.

### Real-World Cases

- **Configuration** – optional settings that may be omitted.
- **Parsing** – returning `optional<T>` from parsing functions.
- **Caching** – optionally present cached values.

---

## 4. `std::variant`

### Definitions

**Core Definition**
`std::variant` is a class template that represents a type-safe union—an object that holds exactly one value from a set of alternative types at any given time.

**Technical Definition**
`std::variant<Types...>` (since C++17) is defined in `<variant>`. At any time it either holds a value of one of its alternative types or, in an error state, no value (`valueless_by_exception`). It cannot hold references, arrays, or `void`. A default-constructed `variant` holds the value-initialized value of its first alternative, unless that alternative is not default-constructible. `std::monostate` can be used as a first alternative to make the variant default-constructible .

**Beginner-Friendly Explanation**
A `variant` is like a `union`, but type-safe: it can hold one of several types, and it always knows which type it currently holds. You can visit it to process the contained value.

### Purposes

- To represent a value that can be one of several types.
- To replace unsafe C-style unions.
- To implement state machines and tagged unions.
- To support type-safe visitation via `std::visit`.

### Syntax Rules and Structure

**General Syntax**
```cpp
#include <variant>
std::variant<int, double, std::string> v = 42;
v = "hello";
if (std::holds_alternative<int>(v)) { ... }
int i = std::get<int>(v);              // throws if wrong type
std::visit([](auto&& x){ ... }, v);    // visitation
```

**Component Breakdown**

| Operation | Description |
|-----------|-------------|
| `index()` | Zero-based index of the active alternative. |
| `holds_alternative<T>` | Checks if the variant holds `T`. |
| `get<T>` / `get<I>` | Accesses the value (throws on mismatch). |
| `get_if<T>` | Pointer-based access (nullptr on mismatch). |
| `visit` | Calls a visitor with the active value. |

**Syntax Rules**
- A `variant` must have at least one alternative (use `std::monostate` for empty).
- Alternatives must be `Destructible` and not references, arrays, or `void`.
- `variant` is default-constructible only if the first alternative is default-constructible.

**Constraints and Limitations**
- All alternatives must be complete types.
- `valueless_by_exception` is a rare error state.

### Annotated Code Examples

**Example 1: Basic variant**

```cpp
#include <variant>
#include <iostream>
#include <string>

int main() {
    std::variant<int, std::string> v = 42;
    std::cout << std::get<int>(v) << '\n';

    v = "hello";
    std::cout << std::get<std::string>(v) << '\n';

    std::cout << std::boolalpha << std::holds_alternative<std::string>(v) << '\n';
}
```

**Expected Output**
```
42
hello
true
```

**Why this output?**  
The variant changes its active type from `int` to `std::string`; `holds_alternative` confirms the active type.

**Example 2: Visitation**

```cpp
#include <variant>
#include <iostream>

int main() {
    std::variant<int, double, std::string> v = 3.14;
    std::visit([](auto&& x) {
        std::cout << "Value: " << x << '\n';
    }, v);
}
```

**Expected Output**
```
Value: 3.14
```

**Why this output?**  
`std::visit` calls the generic lambda with the active value, regardless of its type.

### Real-World Cases

- **Parsers** – a token can be an identifier, number, or operator.
- **State machines** – states as variant alternatives.
- **Error handling** – `variant<T, Error>` as a result type.

---

## 5. `std::any`

### Definitions

**Core Definition**
`std::any` is a class that describes a type-safe container for single values of any copy-constructible type.

**Technical Definition**
`std::any` (since C++17) is defined in `<any>`. It stores an instance of any type that satisfies the constructor requirements, or is empty. The stored instance is called the *contained object*. Access is provided by the non-member `any_cast` functions, which throw `std::bad_any_cast` on type mismatch. Implementations are encouraged to avoid dynamic allocations for small objects .

**Beginner-Friendly Explanation**
`any` is a box that can hold a value of literally any type—like a `void*` but type-safe. You can put anything in it and later ask for the value back with the correct type.

### Purposes

- To store values of any type in a type-safe way.
- To implement dynamic typing or heterogeneous collections.
- To pass values through interfaces that cannot be templated.
- To replace `void*` with a safer alternative.

### Syntax Rules and Structure

**General Syntax**
```cpp
#include <any>
std::any a = 42;
a = std::string("hello");
if (a.has_value()) { ... }
int i = std::any_cast<int>(a);         // throws on mismatch
auto* p = std::any_cast<int>(&a);       // pointer form (nullptr on mismatch)
a.reset();                              // empty
```

**Component Breakdown**

| Operation | Description |
|-----------|-------------|
| `has_value()` | Checks if a value is stored. |
| `type()` | Returns `typeid` of the contained value. |
| `any_cast<T>` | Type-safe access; throws `std::bad_any_cast` on failure. |
| `any_cast<T>(&a)` | Pointer form; returns `nullptr` on failure. |
| `reset()` | Makes the `any` empty. |

**Syntax Rules**
- `any` can hold any `CopyConstructible` type.
- `any_cast<T>` requires exact type matching (no implicit conversions).
- `any` is empty by default.

**Constraints and Limitations**
- Requires `CopyConstructible`; move-only types are not supported.
- `any_cast` does not perform conversions.
- Potential dynamic allocation for large types.

### Annotated Code Examples

**Example 1: Storing and retrieving values**

```cpp
#include <any>
#include <iostream>
#include <string>

int main() {
    std::any a = 42;
    std::cout << std::any_cast<int>(a) << '\n';

    a = std::string("hello");
    std::cout << std::any_cast<std::string>(a) << '\n';

    try {
        std::cout << std::any_cast<double>(a) << '\n';
    } catch (const std::bad_any_cast& e) {
        std::cout << "Bad cast: " << e.what() << '\n';
    }
}
```

**Expected Output**
```
42
hello
Bad cast: bad any_cast
```

**Why this output?**  
`any_cast` succeeds when the type matches and throws when it does not.

**Example 2: Checking type**

```cpp
#include <any>
#include <iostream>
#include <typeinfo>

int main() {
    std::any a = 3.14;
    std::cout << a.type().name() << '\n';
    std::cout << std::boolalpha << (a.type() == typeid(double)) << '\n';
}
```

**Expected Output** (type name is implementation-dependent)
```
d
true
```

**Why this output?**  
`type()` returns the `type_info` of the contained value.

### Real-World Cases

- **Property systems** – storing arbitrary property values.
- **Scripting bridges** – passing values between typed and dynamic code.
- **Event systems** – carrying arbitrary event payloads.

---

## 6. `std::string_view`

### Definitions

**Core Definition**
`std::string_view` is a non-owning view over a constant contiguous sequence of characters.

**Technical Definition**
`std::basic_string_view<CharT, Traits>` (since C++17) is defined in `<string_view>`. It describes an object that can refer to a constant contiguous sequence of `CharT` with the first element at position zero. Every specialization is a `TriviallyCopyable` type. `std::string_view` is a typedef for `std::basic_string_view<char>`. It does not own the data it refers to; the underlying character sequence must outlive the view .

**Beginner-Friendly Explanation**
`string_view` is a "window" onto a string. It lets you read the characters without copying them. It is cheap to pass around, but you must ensure the original string stays alive.

### Purposes

- To pass strings to functions without copying.
- To provide a lightweight, read-only view over string data.
- To avoid `const std::string&` when ownership is not needed.
- To serve as a building block for parsing and tokenization.

### Syntax Rules and Structure

**General Syntax**
```cpp
#include <string_view>
std::string_view sv = "hello";
std::string_view sv2 = std::string("hello"); // views the temporary (dangerous)
sv.substr(1, 3);                              // "ell"
sv.data();                                    // pointer to underlying data
sv.size();                                    // length
```

**Component Breakdown**

| Operation | Description |
|-----------|-------------|
| `data()` | Pointer to the first character. |
| `size()` / `length()` | Number of characters. |
| `substr(pos, count)` | Returns a sub-view. |
| `remove_prefix(n)` / `remove_suffix(n)` | Shrinks the view. |
| `starts_with` / `ends_with` (C++20) | Prefix/suffix checks. |

**Syntax Rules**
- `string_view` does not own its data; it must not outlive the underlying string.
- `string_view` is trivially copyable.
- Implicit conversion from `std::string` and string literals.

**Constraints and Limitations**
- Dangling views: a `string_view` to a destroyed string is undefined behavior.
- No null-termination guarantee (`data()` is not necessarily null-terminated).
- Cannot modify the underlying characters.

### Annotated Code Examples

**Example 1: Function parameter**

```cpp
#include <string_view>
#include <iostream>

void print(std::string_view sv) {
    std::cout << sv << '\n';
}

int main() {
    print("literal");
    std::string s = "string";
    print(s);
}
```

**Expected Output**
```
literal
string
```

**Why this output?**  
`string_view` accepts both string literals and `std::string` without copying.

**Example 2: Substrings**

```cpp
#include <string_view>
#include <iostream>

int main() {
    std::string_view sv = "hello world";
    std::cout << sv.substr(0, 5) << '\n';  // hello
    std::cout << sv.substr(6) << '\n';     // world
}
```

**Expected Output**
```
hello
world
```

**Why this output?**  
`substr` returns a new view without copying the characters.

### Real-World Cases

- **Parsing** – tokenizing a string without allocations.
- **API parameters** – accepting `string_view` instead of `const std::string&`.
- **Logging** – formatting without copying.

---

## 7. `std::span`

### Definitions

**Core Definition**
`std::span` is a non-owning view over a contiguous sequence of objects.

**Technical Definition**
`std::span<T, Extent>` (since C++20) is defined in `<span>`. It describes an object that can refer to a contiguous sequence of objects with the first element at position zero. A span can have a static extent (known at compile time, encoded in the type) or a dynamic extent (`std::dynamic_extent`). Every specialization is a `TriviallyCopyable` type. It is the contiguous-sequence counterpart to `string_view` .

**Beginner-Friendly Explanation**
`span` is like `string_view` but for any type: it is a window onto a contiguous block of objects (like an array or a vector) without owning them. It lets you pass arrays and vectors to functions uniformly.

### Purposes

- To pass contiguous sequences to functions without templating on container type.
- To provide a bounds-aware view over arrays and vectors.
- To avoid pointer-length pairs in interfaces.
- To serve as a bridge between containers and low-level APIs.

### Syntax Rules and Structure

**General Syntax**
```cpp
#include <span>
std::span<int> s = arr;               // from C array
std::span<int> s = vec;               // from vector
std::span<int, 3> s = arr;            // static extent
s[0]; s.size(); s.data(); s.subspan(1, 2);
```

**Component Breakdown**

| Operation | Description |
|-----------|-------------|
| `data()` | Pointer to the first element. |
| `size()` | Number of elements. |
| `operator[]` | Unchecked access. |
| `subspan(offset, count)` | Returns a sub-span. |
| `first(n)` / `last(n)` | First/last `n` elements. |

**Syntax Rules**
- `span` does not own its elements.
- A static extent is part of the type; a dynamic extent uses `std::dynamic_extent`.
- `span` is trivially copyable.

**Constraints and Limitations**
- The underlying storage must outlive the span.
- No bounds checking on `operator[]`.
- Cannot be constructed from non-contiguous containers.

### Annotated Code Examples

**Example 1: Function accepting a span**

```cpp
#include <span>
#include <iostream>
#include <vector>

void print(std::span<int> s) {
    for (int x : s) std::cout << x << ' ';
}

int main() {
    int arr[] = {1, 2, 3};
    std::vector<int> v = {4, 5, 6};
    print(arr);
    print(v);
}
```

**Expected Output**
```
1 2 3 4 5 6
```

**Why this output?**  
`span` uniformly accepts both a C array and a vector.

**Example 2: Subspan**

```cpp
#include <span>
#include <iostream>

int main() {
    int arr[] = {10, 20, 30, 40};
    std::span<int> s = arr;
    auto sub = s.subspan(1, 2);
    for (int x : sub) std::cout << x << ' ';
}
```

**Expected Output**
```
20 30
```

**Why this output?**  
`subspan` creates a view over a portion of the original span.

### Real-World Cases

- **Numeric APIs** – accepting arrays/vectors uniformly.
- **Buffer processing** – views over byte buffers.
- **Interfacing with C libraries** – pointer-length pairs.

---

## 8. `std::function`

### Definitions

**Core Definition**
`std::function` is a general-purpose polymorphic function wrapper that can store, copy, and invoke any `CopyConstructible` callable target.

**Technical Definition**
`std::function<R(Args...)>` (since C++11) is defined in `<functional>`. It can store functions (via pointers), lambda expressions, bind expressions, other function objects, and pointers to member functions/data members. If a `std::function` contains no target, it is empty; invoking an empty `std::function` throws `std::bad_function_call`. It satisfies `CopyConstructible` and `CopyAssignable` .

**Beginner-Friendly Explanation**
`std::function` is a "universal function holder." You can put any kind of callable into it—a regular function, a lambda, a functor—and call it later. It is useful for callbacks and storing functions in containers.

### Purposes

- To store callables in variables, containers, or class members.
- To pass callbacks with a uniform type.
- To enable runtime polymorphism for functions.
- To type-erase callable objects.

### Syntax Rules and Structure

**General Syntax**
```cpp
#include <functional>
std::function<int(int, int)> f = [](int a, int b) { return a + b; };
f(2, 3);                              // 5
if (f) { /* has target */ }
```

**Component Breakdown**

| Operation | Description |
|-----------|-------------|
| `operator()` | Invokes the target. |
| `operator bool` | Checks if a target is stored. |
| `target<T>()` | Returns a pointer to the stored target if it is of type `T`. |
| `target_type()` | Returns `typeid` of the stored target. |

**Syntax Rules**
- `std::function` requires `CopyConstructible` callables.
- Empty `std::function` throws `std::bad_function_call` on invocation.
- `std::function` can be constructed from any compatible callable.

**Constraints and Limitations**
- Type erasure introduces overhead (indirect call, possible heap allocation).
- Cannot store move-only callables (use `std::move_only_function` in C++23).

### Annotated Code Examples

**Example 1: Storing a lambda**

```cpp
#include <functional>
#include <iostream>

int main() {
    std::function<int(int, int)> add = [](int a, int b) { return a + b; };
    std::cout << add(3, 4) << '\n';        // 7
}
```

**Expected Output**
```
7
```

**Why this output?**  
`std::function` wraps the lambda and invokes it with the given arguments.

**Example 2: Callback**

```cpp
#include <functional>
#include <iostream>
#include <vector>

void for_each(const std::vector<int>& v, std::function<void(int)> f) {
    for (int x : v) f(x);
}

int main() {
    std::vector<int> v = {1, 2, 3};
    for_each(v, [](int x) { std::cout << x * 2 << ' '; });
}
```

**Expected Output**
```
2 4 6
```

**Why this output?**  
The callback is invoked for each element.

### Real-World Cases

- **Event systems** – storing callbacks.
- **Strategy pattern** – runtime selection of algorithms.
- **Command queues** – storing deferred actions.

---

## 9. `std::reference_wrapper`

### Definitions

**Core Definition**
`std::reference_wrapper` is a class template that wraps a reference in a copyable, assignable object.

**Technical Definition**
`std::reference_wrapper<T>` (since C++11) is defined in `<functional>`. It is frequently used to store references inside standard containers (which cannot normally hold references). Instances are objects (copyable, storable) but are implicitly convertible to `T&`, so they can be used as arguments to functions taking the underlying type by reference. It is also used to pass objects by reference to `std::bind` and `std::thread` .

**Beginner-Friendly Explanation**
`reference_wrapper` is a way to put a reference inside a container or pass a reference to `std::bind` or `std::thread`. It behaves like a reference but is copyable.

### Purposes

- To store references in standard containers.
- To pass references to `std::bind`, `std::thread`, and other helpers.
- To avoid copies when a copyable wrapper is required.
- To enable reference semantics in generic code.

### Syntax Rules and Structure

**General Syntax**
```cpp
#include <functional>
std::reference_wrapper<T> ref = std::ref(obj);
T& r = ref;                           // implicit conversion
```

**Component Breakdown**

| Operation | Description |
|-----------|-------------|
| `get()` | Returns the stored reference. |
| `operator T&()` | Implicit conversion to `T&`. |
| `operator()` | Calls the stored callable (if `T` is callable). |

**Syntax Rules**
- `reference_wrapper` is copyable and assignable.
- It cannot be default-constructed.
- `std::ref` and `std::cref` are the preferred creation functions.

**Constraints and Limitations**
- The referenced object must outlive the wrapper.
- Cannot wrap rvalue references (`std::ref` deletes rvalue overloads).

### Annotated Code Example

```cpp
#include <functional>
#include <vector>
#include <iostream>

int main() {
    int a = 1, b = 2, c = 3;
    std::vector<std::reference_wrapper<int>> refs = {std::ref(a), std::ref(b), std::ref(c)};

    for (auto& r : refs) r.get() *= 10;
    std::cout << a << ' ' << b << ' ' << c << '\n'; // 10 20 30
}
```

**Expected Output**
```
10 20 30
```

**Why this output?**  
The vector stores references; modifying through the wrapper modifies the original variables.

### Real-World Cases

- **Containers of references** – storing references to existing objects.
- **`std::bind`** – passing arguments by reference.
- **`std::thread`** – passing references to thread functions.

---

## 10. `std::ref` and `std::cref`

### Definitions

**Core Definition**
`std::ref` and `std::cref` are helper function templates that generate `std::reference_wrapper` objects, using template argument deduction.

**Technical Definition**
`std::ref(T&)` returns `std::reference_wrapper<T>`; `std::cref(const T&)` returns `std::reference_wrapper<const T>` (since C++11). They are defined in `<functional>`. The rvalue-reference overloads are deleted to prevent dangling references .

**Beginner-Friendly Explanation**
`std::ref` and `std::cref` are convenience functions that create `reference_wrapper` objects. You use them when you need to pass a reference to something that would otherwise copy (like `std::bind` or `std::thread`).

### Purposes

- To create a `reference_wrapper` without specifying the type explicitly.
- To pass references to `std::bind`, `std::thread`, and algorithms.
- To avoid copies in generic code.
- To provide const and non-const reference wrappers.

### Syntax Rules and Structure

**General Syntax**
```cpp
#include <functional>
auto r = std::ref(obj);               // reference_wrapper<T>
auto cr = std::cref(obj);             // reference_wrapper<const T>
```

**Component Breakdown**

| Function | Returns |
|----------|---------|
| `std::ref(T&)` | `reference_wrapper<T>` |
| `std::cref(const T&)` | `reference_wrapper<const T>` |

**Syntax Rules**
- `std::ref` cannot bind to rvalues (deleted overload).
- `std::cref` provides const access.

**Constraints and Limitations**
- The referenced object must outlive the wrapper.

### Annotated Code Example

```cpp
#include <functional>
#include <iostream>

void increment(int& x) { ++x; }

int main() {
    int n = 0;
    auto bound = std::bind(increment, std::ref(n));
    bound();
    std::cout << n << '\n';                // 1
}
```

**Expected Output**
```
1
```

**Why this output?**  
`std::ref(n)` ensures `std::bind` stores a reference, so `increment` modifies `n`.

### Real-World Cases

- **`std::bind`** – passing arguments by reference.
- **`std::thread`** – sharing data between threads.
- **Algorithms** – passing references to functions expecting values.

---

## 11. Utility Best Practices

### Definitions

**Core Definition**
Utility best practices are guidelines for choosing and using the standard utility components correctly, safely, and efficiently, as informed by the C++ Core Guidelines.

**Technical Definition**
These practices derive from the C++ Core Guidelines (e.g., SL.con, ES, F) and community consensus: prefer `optional` over pointer-or-sentinel, use `variant` instead of raw unions, use `string_view` and `span` for non-owning views while being mindful of lifetimes, prefer `std::function` only when type erasure is needed, and use `reference_wrapper` deliberately.

**Beginner-Friendly Explanation**
Best practices for utility components are about picking the right tool and using it safely. Use `optional` when a value might be missing; use `variant` when a value can be one of several types; use `string_view` and `span` to avoid copies but watch out for dangling references.

### Key Guidelines

| Guideline | Summary |
|-----------|---------|
| **Prefer `optional` over sentinel values** | Express optionality explicitly. |
| **Prefer `variant` over raw unions** | Type-safe alternatives. |
| **Use `string_view` for read-only string parameters** | Avoids copies; ensure lifetime. |
| **Use `span` for contiguous sequences** | Uniform interface for arrays and vectors. |
| **Use `std::function` only when type erasure is needed** | Templates or `auto` are often better. |
| **Use `reference_wrapper` in containers** | Containers cannot hold references directly. |
| **Prefer `std::ref`/`std::cref` over manual wrappers** | Simpler and less error-prone. |
| **Be mindful of view lifetimes** | `string_view` and `span` must not outlive their data. |
| **Use structured bindings with `pair` and `tuple`** | Cleaner code (C++17). |
| **Prefer `make_pair` / `make_tuple` for type deduction** | Avoids redundant type names. |

### Annotated Code Example

```cpp
#include <optional>
#include <variant>
#include <string_view>
#include <span>
#include <iostream>

std::optional<int> find_value(std::span<const int> data, int target) {
    for (int x : data)
        if (x == target) return x;
    return std::nullopt;
}

int main() {
    int arr[] = {1, 2, 3};
    auto result = find_value(arr, 2);
    if (result) std::cout << "Found: " << *result << '\n';
}
```

**Expected Output**
```
Found: 2
```

**Why this output?**  
`span` provides a uniform view over the array; `optional` expresses the possibility of absence.

### Real-World Cases

- **API design** – using `string_view` and `span` in function signatures.
- **Error handling** – `optional` and `variant` for result types.
- **Generic libraries** – `std::function` for callbacks.

---

## References

- `std::pair` — cppreference – https://en.cppreference.com/w/cpp/utility/pair
- `std::tuple` — cppreference – https://en.cppreference.com/w/cpp/utility/tuple
- `std::optional` — cppreference – https://en.cppreference.com/w/cpp/utility/optional
- `std::variant` — cppreference – https://en.cppreference.com/w/cpp/utility/variant
- `std::any` — cppreference – https://en.cppreference.com/w/cpp/utility/any
- `std::basic_string_view` — cppreference – https://en.cppreference.com/w/cpp/string/basic_string_view
- `std::span` — cppreference – https://en.cppreference.com/w/cpp/container/span
- `std::function` — cppreference – https://en.cppreference.com/w/cpp/utility/functional/function
- `std::reference_wrapper` — cppreference – https://en.cppreference.com/w/cpp/utility/functional/reference_wrapper
- `std::ref`, `std::cref` — cppreference – https://en.cppreference.com/w/cpp/utility/functional/ref
- C++ Core Guidelines — https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines

---

This cheat sheet provides a comprehensive, structured reference for C++ utility components, covering `std::pair`, `std::tuple`, `std::optional`, `std::variant`, `std::any`, `std::string_view`, `std::span`, `std::function`, `std::reference_wrapper`, `std::ref`/`std::cref`, and best practices, with annotated code examples, expected outputs, real-world cases, and authoritative references.