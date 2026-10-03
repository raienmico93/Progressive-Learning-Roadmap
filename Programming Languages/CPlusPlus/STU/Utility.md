# Modern Utility & Vocabulary Types — Comprehensive Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
Utility and vocabulary types are standard library class templates that provide fundamental abstractions for grouping, representing absence, managing alternative types, handling errors, erasing types, and wrapping references. They are not containers in the classical sense but serve as the building blocks for expressive, type-safe, and modern C++ interfaces.

**Technical Definition**
The C++ standard library provides a family of vocabulary types in headers `<utility>`, `<tuple>`, `<optional>`, `<variant>`, `<expected>`, `<any>`, and `<functional>`. These types are designed to be used in function signatures, return types, and class members to convey intent with minimal overhead. They support C++ features such as structured bindings (C++17), monadic operations (C++23), and type-safe visitation (C++17). Unlike containers, they do not manage sequences of elements; instead, they model specific value semantics: heterogeneous grouping (`pair`, `tuple`), optional presence (`optional`), discriminated unions (`variant`), error-or-value results (`expected`), type erasure (`any`), and copyable reference semantics (`reference_wrapper`).

**Beginner-Friendly Explanation**
Think of these types as specialised boxes in a toolbox. `std::pair` is a two-slot box (e.g., a key and a value). `std::tuple` is a box with any number of slots. `std::optional` is a box that may or may not contain something. `std::variant` is a box that holds exactly one of several possible things. `std::expected` is a box that holds either a result or an error. `std::any` is a box that can hold anything. `std::reference_wrapper` is a box that holds a reference instead of a copy. Each box communicates exactly what the code intends.

---

### Key Characteristics

- **Heterogeneous grouping**: `std::pair` and `std::tuple` store elements of different types in a fixed-size collection.
- **Optional presence**: `std::optional` models a value that may or may not be present, without using sentinel values or exceptions.
- **Type-safe unions**: `std::variant` replaces `union` with a type-safe discriminated union that tracks which alternative is active.
- **Error-or-value returns**: `std::expected` (C++23) represents either a valid value or an error value, enabling exception-free error handling.
- **Type erasure**: `std::any` stores a value of any copy-constructible type, preserving type safety through runtime checks.
- **Copyable references**: `std::reference_wrapper` wraps a reference in a copyable, assignable object.
- **Structured bindings**: C++17 allows decomposing `pair`, `tuple`, and other tuple-like types directly into named variables.
- **Monadic operations**: C++23 adds `transform`, `and_then`, and `or_else` to `std::optional` and `std::expected`, enabling chainable pipelines.

---

### Prerequisites

- Basic C++ syntax: templates, auto, lambdas, and move semantics.
- Familiarity with C++11/14/17 features such as `auto`, range-based for, and structured bindings.
- Understanding of exceptions and error handling.
- (For `std::optional` monadic ops) A compiler supporting C++23 or later.
- (For `std::expected`) A compiler supporting C++23 or later.

---

### Related Programming Areas

- **API design**: Vocabulary types communicate intent clearly in function signatures.
- **Functional programming**: Monadic operations bring composable pipelines to C++.
- **Error handling**: `std::expected` and `std::optional` provide alternatives to exceptions and error codes.
- **Type erasure and metaprogramming**: `std::any` and `std::variant` enable runtime polymorphism without inheritance.
- **Generic programming**: `std::pair` and `std::tuple` are used throughout the standard library (maps, algorithms, structured bindings).

---

### Core Concepts / Features

1. **Heterogeneous Groupings** — `std::pair` and `std::tuple` with structured bindings.
2. **Nullable Values** — `std::optional` and its monadic operations.
3. **Type-Safe Unions** — `std::variant` and `std::visit`.
4. **Functional Error Handling** — `std::expected` (C++23).
5. **Type Erasure & References** — `std::any` and `std::reference_wrapper`.

---

## 1. Heterogeneous Groupings — `std::pair` and `std::tuple`

### Definitions

**Core Definition**
`std::pair` is a class template that stores exactly two heterogeneous objects as a single unit. `std::tuple` is a generalization that stores a fixed-size collection of heterogeneous values of any arity.

**Technical Definition**
`template<class T1, class T2> struct pair;` is defined in `<utility>`. A pair is a special case of `std::tuple` with two elements. `template<class... Types> class tuple;` is defined in `<tuple>` (since C++11). If all element types are trivially destructible, the destructor of `tuple` and `pair` is trivial. Structured bindings (C++17) allow decomposing these types into named variables without `std::get` or `std::tie`. Both types support element access via `std::get<N>` or `.first`/`.second`, comparison, swapping, and use with `std::make_pair` / `std::make_tuple`. Since C++20, `pair` and `tuple` support formatting via `std::format`.

**Beginner-Friendly Explanation**
A `pair` is like a two-slot pillbox: one slot for a key, one for a value. A `tuple` is like a pillbox with any number of slots, each holding a different type of pill. You can take the pills out individually using structured bindings: `auto [key, value] = my_pair;`.

---

### Purposes (all begin with "To")

- **To** group two (pair) or more (tuple) heterogeneous values into a single object.
- **To** return multiple values from a function without defining a custom struct.
- **To** enable structured bindings for clean, readable decomposition.
- **To** serve as the element type of `std::map` and `std::unordered_map`.
- **To** support generic programming with tuple-like protocols (`std::get`, `tuple_size`, `tuple_element`).

---

### Syntax Rules and Structure

```cpp
// std::pair
template<class T1, class T2>
struct pair {
    T1 first;
    T2 second;
};

// std::tuple
template<class... Types>
class tuple;
```

**Common operations:**
- `p.first`, `p.second` — direct member access for `pair`.
- `std::get<N>(t)` — access the Nth element of a tuple (compile-time index).
- `std::make_pair(a, b)` / `std::make_tuple(a, b, c)` — create pairs/tuples with deduced types.
- `std::tie(a, b) = t;` — unpack a tuple into existing variables (C++11).
- `auto [a, b] = t;` — structured binding (C++17).
- `std::tuple_size<T>::value` — number of elements.
- `std::tuple_element<N, T>::type` — type of the Nth element.

**Constraints and limitations:**
- `pair` always has exactly two elements; `tuple` can have zero or more.
- Element access by index requires a compile-time constant.
- Structured bindings cannot be declared `constexpr` in all contexts prior to C++20.
- `std::tie` cannot be used with `const` tuples for assignment (it binds to lvalue references).
- `std::get` by type requires the type to be unique within the tuple.

---

### Multiple Annotated Complete Step-by-Step Code Examples and Their Expected Outputs

#### Example 1 — Pair and Structured Bindings

```cpp
// File: pair_example.cpp
// Compile with: g++ -std=c++17 pair_example.cpp -o pair_example

#include <iostream>
#include <utility>
#include <string>

int main() {
    // Step 1: Create a pair
    std::pair<int, std::string> p = {42, "answer"};

    // Step 2: Access members
    std::cout << "first: " << p.first
              << ", second: " << p.second << std::endl;

    // Step 3: Structured binding (C++17)
    auto [id, label] = p;
    std::cout << "id: " << id << ", label: " << label << std::endl;

    // Step 4: make_pair deduces types
    auto p2 = std::make_pair(3.14, 'pi');
    std::cout << "p2: " << p2.first << ", " << p2.second << std::endl;

    // Step 5: Pair comparison (lexicographic)
    std::pair<int, int> a = {1, 2};
    std::pair<int, int> b = {1, 3};
    std::cout << "a < b: " << (a < b) << std::endl;

    return 0;
}
```

**Expected Output:**
```
first: 42, second: answer
id: 42, label: answer
p2: 3.14, pi
a < b: 1
```

**Why this output:** `p.first` and `p.second` access the two elements. The structured binding `auto [id, label]` decomposes the pair. `make_pair` deduces `double` and `char`. Lexicographic comparison compares `first` first, then `second`.

---

#### Example 2 — Tuple and Structured Bindings

```cpp
// File: tuple_example.cpp
// Compile with: g++ -std=c++17 tuple_example.cpp -o tuple_example

#include <iostream>
#include <tuple>
#include <string>

int main() {
    // Step 1: Create a tuple with three heterogeneous elements
    std::tuple<int, std::string, double> t = {1, "hello", 3.14};

    // Step 2: Access by compile-time index
    std::cout << "get<0>: " << std::get<0>(t) << std::endl;
    std::cout << "get<1>: " << std::get<1>(t) << std::endl;
    std::cout << "get<2>: " << std::get<2>(t) << std::endl;

    // Step 3: Structured binding
    auto [id, msg, value] = t;
    std::cout << "Structured: " << id << ", " << msg << ", "
              << value << std::endl;

    // Step 4: make_tuple deduces types
    auto t2 = std::make_tuple(10, 'x', 2.5f);
    std::cout << "t2 size: " << std::tuple_size<decltype(t2)>::value
              << std::endl;

    // Step 5: tie for unpacking into existing variables
    int i; std::string s; double d;
    std::tie(i, s, d) = t;
    std::cout << "tie: " << i << ", " << s << ", " << d << std::endl;

    return 0;
}
```

**Expected Output:**
```
get<0>: 1
get<1>: hello
get<2>: 3.14
Structured: 1, hello, 3.14
t2 size: 3
tie: 1, hello, 3.14
```

**Why this output:** `std::get<N>` accesses elements by compile-time index. Structured bindings decompose the tuple into named variables. `std::tuple_size` reports the number of elements. `std::tie` unpacks the tuple into pre-declared variables.

---

### Real-World Cases with Explanation

**Case 1 — Map elements**: `std::map` stores `std::pair<const Key, T>` as its element type. Iterating a map yields pairs, and structured bindings make the key and value directly accessible.

**Case 2 — Multi-return functions**: A function returning `std::tuple<int, int, bool>` can return multiple values without defining a struct.

**Case 3 — Zipping ranges**: `std::views::zip` produces tuples when combining multiple ranges (C++23).

**Case 4 — Configuration entries**: A pair of `{name, value}` is a natural representation for configuration key-value entries.

---

### References

- std::pair — cppreference.com - https://en.cppreference.com/w/cpp/utility/pair
- std::tuple — cppreference.com - https://en.cppreference.com/w/cpp/utility/tuple
- Structured binding declaration — cppreference.com - https://en.cppreference.com/w/cpp/language/structured_binding
- std::make_pair — cppreference.com - https://en.cppreference.com/w/cpp/utility/pair/make_pair
- std::make_tuple — cppreference.com - https://en.cppreference.com/w/cpp/utility/tuple/make_tuple
- std::tie — cppreference.com - https://en.cppreference.com/w/cpp/utility/tuple/tie
- std::get (tuple) — cppreference.com - https://en.cppreference.com/w/cpp/utility/tuple/get

---

## 2. Nullable Values — `std::optional`

### Definitions

**Core Definition**
`std::optional<T>` is a class template that manages an optional contained value — a value that may or may not be present.

**Technical Definition**
`template<class T> class optional;` (C++17, `<optional>`). An `optional<T>` either contains a value of type `T` or is empty (`std::nullopt`). It never allocates dynamic memory; the contained value is stored inline within the `optional` object. It provides `has_value()`, `value()`, `value_or()`, `operator*`, and `operator->`. C++23 adds three monadic operations: `transform`, `and_then`, and `or_else`, enabling chainable pipelines without nested `if` checks.

**Beginner-Friendly Explanation**
`std::optional` is like a box that may or may not contain an item. You can ask "do you have something?" and if so, take it out. If not, you get a default or an error. It replaces sentinel values like `-1` or `nullptr` with a type-safe alternative.

---

### Sub-Feature: Monadic Operations (C++23)

#### Definitions

**Core Definition**
Monadic operations (`transform`, `and_then`, `or_else`) are member functions that allow chaining operations on `std::optional` without explicit checks for emptiness.

**Technical Definition**
- `transform(F&& f)` — if engaged, applies `f` to the contained value and returns `optional<U>` where `U` is the result of `f`; if empty, returns empty `optional<U>`.
- `and_then(F&& f)` — if engaged, applies `f` (which must return an `optional`) and returns its result; if empty, returns empty. This is flat-mapping and avoids `optional<optional<U>>`.
- `or_else(F&& f)` — if engaged, returns `*this`; if empty, calls `f()` and returns its result (which must be an `optional<T>`).

**Beginner-Friendly Explanation**
Monadic operations let you write a pipeline: "if I have a value, do this; if not, do that." Instead of writing nested `if (opt.has_value())` checks, you chain `.transform(...).and_then(...).or_else(...)`.

#### Purposes (all begin with "To")

- **To** chain operations on optional values without explicit emptiness checks.
- **To** eliminate deep nesting of `if` statements for optional values.
- **To** provide a fallback value or computation when the optional is empty.
- **To** transform the contained value while preserving optionality.
- **To** flat-map operations that themselves return optionals.

#### Syntax Rules and Structure

```cpp
// transform: optional<T> -> (T -> U) -> optional<U>
template<class F> constexpr auto transform(F&& f) &;

// and_then: optional<T> -> (T -> optional<U>) -> optional<U>
template<class F> constexpr auto and_then(F&& f) &;

// or_else: optional<T> -> (() -> optional<T>) -> optional<T>
template<class F> constexpr optional or_else(F&& f) const&;
```

**Constraints and limitations:**
- Requires C++23 or later.
- `transform` and `and_then` do not participate in overload resolution if the callable's return type is not appropriate.
- `or_else` must return an `optional` of the same type (or a type convertible to it).
- All three operations are `constexpr` and can be used in constant evaluation.

#### Annotated Code Example

```cpp
// File: optional_monadic.cpp
// Compile with: g++ -std=c++23 optional_monadic.cpp -o optional_monadic

#include <iostream>
#include <optional>
#include <string>

// Step 1: Helper functions
std::optional<int> parse_int(const std::string& s) {
    try {
        return std::stoi(s);
    } catch (...) {
        return std::nullopt;
    }
}

std::optional<double> safe_sqrt(int n) {
    if (n < 0) return std::nullopt;
    return std::sqrt(static_cast<double>(n));
}

int main() {
    // Step 2: Monadic pipeline
    std::optional<std::string> input = "49";

    auto result = input
        .and_then(parse_int)           // optional<string> -> optional<int>
        .and_then(safe_sqrt)           // optional<int> -> optional<double>
        .transform([](double d) {      // optional<double> -> optional<string>
            return std::to_string(d);
        })
        .or_else([] {                  // fallback if any step failed
            return std::optional<std::string>{"default"};
        });

    std::cout << "Result: " << *result << std::endl;

    // Step 3: Pipeline with failure
    std::optional<std::string> bad = "not_a_number";
    auto result2 = bad
        .and_then(parse_int)
        .and_then(safe_sqrt)
        .transform([](double d) { return std::to_string(d); })
        .or_else([] { return std::optional<std::string>{"default"}; });

    std::cout << "Result2: " << *result2 << std::endl;

    return 0;
}
```

**Expected Output:**
```
Result: 7.000000
Result2: default
```

**Why this output:** The first pipeline parses "49", takes the square root (7.0), converts to string, and prints it. The second pipeline fails to parse "not_a_number", so `or_else` provides the fallback "default".

---

### Real-World Cases with Explanation

**Case 1 — Configuration lookup**: A function returns `std::optional<std::string>` for a config key that may not exist; monadic operations chain parsing and validation.

**Case 2 — Database queries**: A query returns `std::optional<Row>`; `.transform(extract_name)` extracts a field if the row exists.

**Case 3 — Parser combinators**: A parser returns `std::optional<AST>`; `.and_then(next_parser)` chains parsing steps.

**Case 4 — Cache lookups**: A cache returns `std::optional<Value>`; `.or_else(compute_from_source)` provides a fallback.

---

### References

- std::optional — cppreference.com - https://en.cppreference.com/w/cpp/utility/optional
- std::optional::transform — cppreference.com - https://en.cppreference.com/w/cpp/utility/optional/transform
- std::optional::and_then — cppreference.com - https://en.cppreference.com/w/cpp/utility/optional/and_then
- std::optional::or_else — cppreference.com - https://en.cppreference.com/w/cpp/utility/optional/or_else
- Use std::optional monadic operations (C++23) — Dr-Sergey/learn_cpp - https://github.com/Dr-Sergey/learn_cpp/blob/main/09_Standard_Library_Utilities/Use_stdoptional_monadic_operations_transform_and_then_or_else_C%2B%2B23.md

---

## 3. Type-Safe Unions — `std::variant`

### Definitions

**Core Definition**
`std::variant<Types...>` is a class template that represents a type-safe union: an instance holds a value of exactly one of its alternative types (or no value in rare exception cases).

**Technical Definition**
`template<class... Types> class variant;` (C++17, `<variant>`). Unlike a C `union`, a `variant` knows which type it currently holds and destroys the correct object. Access is via `std::get<N>`, `std::get<T>`, `std::get_if`, or `std::visit`. `std::visit` applies a visitor callable to the currently held alternative. The variant may become `valueless_by_exception()` if an exception occurs during a type-changing assignment, but this state is rare and can be checked.

**Beginner-Friendly Explanation**
A `variant` is like a box that can hold exactly one of several predefined types. It remembers what type it currently holds, so you can't accidentally read it as the wrong type. To do something with the box's contents, you use a "visitor" — a function that handles every possible type.

---

### Sub-Feature: `std::visit`

#### Definitions

**Core Definition**
`std::visit` is a function template that applies a visitor callable to the value held by a `std::variant`, with the correct alternative selected at runtime.

**Technical Definition**
`template<class Visitor, class... Variants> constexpr decltype(auto) visit(Visitor&& vis, Variants&&... vars);`. The visitor must be callable with every possible alternative combination of the variants. The return type is the common type of all invocations. `std::visit` guarantees that the visitor is called exactly once. Since C++26, `variant` also has a member `visit` function.

**Beginner-Friendly Explanation**
`std::visit` is like a universal remote. You give it a function that can handle every possible type in the variant, and it calls the right one automatically.

#### Purposes (all begin with "To")

- **To** apply a callable to the currently held alternative of a variant.
- **To** eliminate manual type checking and `get_if` chains.
- **To** enable type-safe pattern matching over variant alternatives.
- **To** support multiple variants in a single visit (multi-visit).
- **To** return a value from a visitor and propagate it to the caller.

#### Syntax Rules and Structure

```cpp
// Basic visit
std::visit(visitor, variant);

// Multi-variant visit
std::visit(visitor, variant1, variant2);

// Visitor with overloads
template<class... Ts> struct overloads : Ts... { using Ts::operator()...; };
auto vis = overloads{
    [](int i) { /* ... */ },
    [](const std::string& s) { /* ... */ }
};
std::visit(vis, v);
```

**Constraints and limitations:**
- The visitor must handle every possible alternative; otherwise, the program is ill-formed.
- `std::visit` requires all variants to be valid (not `valueless_by_exception`); violating this throws `std::bad_variant_access`.
- Multi-visit requires the visitor to be callable with all combinations of alternatives.
- The return type of `std::visit` is the common type of all visitor return types.

#### Annotated Code Example

```cpp
// File: variant_visit.cpp
// Compile with: g++ -std=c++17 variant_visit.cpp -o variant_visit

#include <iostream>
#include <variant>
#include <string>
#include <vector>

// Step 1: Overload helper for visitors
template<class... Ts> struct overloads : Ts... {
    using Ts::operator()...;
};
template<class... Ts> overloads(Ts...) -> overloads<Ts...>;

int main() {
    // Step 2: Variant holding int, string, or bool
    using Var = std::variant<int, std::string, bool>;

    std::vector<Var> values = {42, "hello", true, 3.14};

    // Step 3: Visitor with overloads
    auto printer = overloads{
        [](int i) { std::cout << "int: " << i << std::endl; },
        [](const std::string& s) { std::cout << "string: " << s << std::endl; },
        [](bool b) { std::cout << "bool: " << b << std::endl; },
        [](double d) { std::cout << "double: " << d << std::endl; }
    };

    // Step 4: Visit each variant
    for (const auto& v : values) {
        std::visit(printer, v);
    }

    // Step 5: get and get_if
    Var v = 100;
    std::cout << "get<int>: " << std::get<int>(v) << std::endl;

    if (auto* p = std::get_if<std::string>(&v)) {
        std::cout << "string: " << *p << std::endl;
    } else {
        std::cout << "v is not a string" << std::endl;
    }

    return 0;
}
```

**Expected Output:**
```
int: 42
string: hello
bool: 1
double: 3.14
get<int>: 100
v is not a string
```

**Why this output:** `std::visit` calls the appropriate overload for each variant's current alternative. `std::get<int>` accesses the value directly (throws if the type is wrong). `std::get_if` returns a pointer or `nullptr`, allowing safe type checking.

---

### Real-World Cases with Explanation

**Case 1 — JSON parsing**: A JSON value is a `variant` of null, bool, number, string, array, or object; `std::visit` processes each type.

**Case 2 — State machines**: A state is a `variant` of different state structs; `std::visit` handles transitions.

**Case 3 — Command dispatch**: A command is a `variant` of different command types; `std::visit` dispatches to the appropriate handler.

**Case 4 — Error handling**: A result is a `variant<Value, Error>`; `std::visit` handles both cases.

---

### References

- std::variant — cppreference.com - https://en.cppreference.com/w/cpp/utility/variant
- std::visit — cppreference.com - https://en.cppreference.com/w/cpp/utility/variant/visit
- std::get (variant) — cppreference.com - https://en.cppreference.com/w/cpp/utility/variant/get
- std::get_if — cppreference.com - https://en.cppreference.com/w/cpp/utility/variant/get_if
- std::variant<Types...>::visit (C++26) — cppreference.com - https://en.cppreference.com/w/cpp/utility/variant/visit

---

## 4. Functional Error Handling — `std::expected` (C++23)

### Definitions

**Core Definition**
`std::expected<T, E>` is a class template that represents either an expected value of type `T` or an unexpected error value of type `E`. It is never valueless.

**Technical Definition**
`template<class T, class E> class expected;` (C++23, `<expected>`). The primary template stores either a `T` or an `E` in a nested variant-like storage. The `void` partial specialization represents either a `void` expected value or an `E`. `T` must be destructible and not a reference, function, or `std::unexpected` specialization. `E` must be destructible and a valid template argument for `std::unexpected`. Access is via `has_value()`, `value()`, `operator*`, `operator->`, and `error()`. Monadic operations (`transform`, `and_then`, `or_else`, `transform_error`) are provided.

**Beginner-Friendly Explanation**
`std::expected` is like a parcel that either contains what you ordered (the expected value) or contains a note explaining what went wrong (the unexpected error). Unlike exceptions, the error is part of the function's return type, so you can't ignore it by accident.

---

### Sub-Feature: Monadic Operations for `std::expected`

#### Definitions

**Core Definition**
Monadic operations for `std::expected` allow chaining transformations and error handling without explicit checks.

**Technical Definition**
- `transform(F&& f)` — if `has_value()`, applies `f` to the expected value and returns `expected<U, E>`; if not, returns the unexpected value.
- `and_then(F&& f)` — if `has_value()`, applies `f` (which returns an `expected`) and returns its result; if not, returns the unexpected value.
- `or_else(F&& f)` — if `has_value()`, returns `*this`; if not, calls `f` with the error value and returns its result.
- `transform_error(F&& f)` — if `has_value()`, returns `*this`; if not, applies `f` to the error value and returns a new `expected`.

**Beginner-Friendly Explanation**
Monadic operations let you write pipelines like `result.and_then(step1).and_then(step2).transform(finalize)`. If any step fails, the error propagates automatically.

#### Purposes (all begin with "To")

- **To** return either a valid value or a detailed error without exceptions.
- **To** chain operations that may fail without nested checks.
- **To** propagate errors automatically through a pipeline.
- **To** transform both success and error values.
- **To** provide a type-safe alternative to error codes and output parameters.

#### Syntax Rules and Structure

```cpp
// Access
if (result.has_value()) { auto v = result.value(); }
else { auto e = result.error(); }

// Monadic operations
result.transform(f);       // expected<T,E> -> expected<U,E>
result.and_then(f);        // expected<T,E> -> expected<U,E>
result.or_else(f);         // expected<T,E> -> expected<T,E>
result.transform_error(f); // expected<T,E> -> expected<T,F>
```

**Constraints and limitations:**
- Requires C++23 or later.
- `T` cannot be a reference, function, or array type.
- `E` must be a valid template argument for `std::unexpected`.
- `std::expected` does not support `valueless_by_exception`.
- Monadic operations are `constexpr`.

#### Annotated Code Example

```cpp
// File: expected_example.cpp
// Compile with: g++ -std=c++23 expected_example.cpp -o expected_example

#include <iostream>
#include <expected>
#include <string>
#include <charconv>

// Step 1: Parse an integer, returning expected<int, string>
std::expected<int, std::string> parse_int(const std::string& s) {
    int result{};
    auto [ptr, ec] = std::from_chars(s.data(), s.data() + s.size(), result);
    if (ec == std::errc{} && ptr == s.data() + s.size())
        return result;
    return std::unexpected("Failed to parse '" + s + "'");
}

// Step 2: Safe division
std::expected<double, std::string> safe_divide(int a, int b) {
    if (b == 0) return std::unexpected("Division by zero");
    return static_cast<double>(a) / b;
}

int main() {
    // Step 3: Successful pipeline
    auto result = parse_int("42")
        .and_then([](int n) { return safe_divide(100, n); })
        .transform([](double d) { return d * 2; });

    if (result) {
        std::cout << "Success: " << *result << std::endl;
    } else {
        std::cout << "Error: " << result.error() << std::endl;
    }

    // Step 4: Failed pipeline
    auto fail = parse_int("abc")
        .and_then([](int n) { return safe_divide(100, n); })
        .transform([](double d) { return d * 2; });

    if (fail) {
        std::cout << "Success: " << *fail << std::endl;
    } else {
        std::cout << "Error: " << fail.error() << std::endl;
    }

    // Step 5: or_else for recovery
    auto recovered = parse_int("0")
        .and_then([](int n) { return safe_divide(100, n); })
        .or_else([](const std::string& err) -> std::expected<double, std::string> {
            std::cout << "Recovering from: " << err << std::endl;
            return 0.0;
        });

    std::cout << "Recovered: " << *recovered << std::endl;

    return 0;
}
```

**Expected Output:**
```
Success: 4.7619
Error: Failed to parse 'abc'
Recovering from: Division by zero
Recovered: 0
```

**Why this output:** The first pipeline parses "42", divides 100 by 42, and doubles the result. The second pipeline fails at parsing and propagates the error. The third pipeline parses "0", fails at division by zero, and `or_else` recovers with 0.0.

---

### Real-World Cases with Explanation

**Case 1 — File I/O**: A function returns `expected<File, ErrorCode>` instead of throwing exceptions, allowing callers to handle errors without try-catch.

**Case 2 — Network requests**: A request returns `expected<Response, HttpError>`, providing detailed error information without exceptions.

**Case 3 — Configuration parsing**: A parser returns `expected<Config, ParseError>`, accumulating errors without throwing.

**Case 4 — Database operations**: A query returns `expected<Row, QueryError>`, allowing callers to handle both success and failure explicitly.

---

### References

- std::expected — cppreference.com - https://en.cppreference.com/w/cpp/utility/expected
- std::expected<T,E>::and_then — cppreference.com - https://en.cppreference.com/w/cpp/utility/expected/and_then
- std::expected<T,E>::transform — cppreference.com - https://en.cppreference.com/w/cpp/utility/expected/transform
- std::expected<T,E>::or_else — cppreference.com - https://en.cppreference.com/w/cpp/utility/expected/or_else
- std::expected<T,E>::transform_error — cppreference.com - https://en.cppreference.com/w/cpp/utility/expected/transform_error
- std::unexpected — cppreference.com - https://en.cppreference.com/w/cpp/utility/expected/unexpected

---

## 5. Type Erasure & References — `std::any` and `std::reference_wrapper`

### Definitions

**Core Definition**
`std::any` is a type-safe container for single values of any copy-constructible type. `std::reference_wrapper<T>` is a copyable, assignable wrapper around a reference to an object or function.

**Technical Definition**
`class any;` (C++17, `<any>`) stores an instance of any type satisfying the constructor requirements, or is empty. `any_cast<T>` provides type-safe access. `template<class T> class reference_wrapper;` (C++11, `<functional>`) is a CopyConstructible and CopyAssignable wrapper around a reference. It is implicitly convertible to `T&` and is trivially copyable (since C++17). Helper functions `std::ref` and `std::cref` create reference wrappers. `reference_wrapper` is used with `std::bind`, `std::thread`, and standard containers to store references.

**Beginner-Friendly Explanation**
`std::any` is like a magic box that can hold anything — an int, a string, a custom object. You can put something in, take it out later, and the box remembers what type it was (throwing if you ask for the wrong type). `std::reference_wrapper` is like a sticky note that says "look over there" instead of copying the thing itself. It lets you store references in containers that normally can't hold them.

---

### Sub-Feature: `std::any`

#### Definitions

**Core Definition**
`std::any` is a type-safe container for a single value of any copy-constructible type.

**Technical Definition**
`class any;` (C++17). An `any` object either stores an instance of some type or is empty. `any_cast<T>(a)` returns the stored value if the type matches, otherwise throws `std::bad_any_cast`. `a.has_value()` checks if the object is non-empty. `a.type()` returns the `std::type_info` of the stored value. Implementations may use small-object optimization to avoid heap allocation for small types.

**Beginner-Friendly Explanation**
`std::any` is a box that can hold any type. You put something in, and later you can ask "what type are you?" and "give me the value." If you ask for the wrong type, it throws an exception.

#### Purposes (all begin with "To")

- **To** store values of any type in a uniform container.
- **To** implement type-erased interfaces without inheritance.
- **To** pass arbitrary values through APIs that cannot be templated.
- **To** defer type decisions until runtime.
- **To** hold heterogeneous values in a single variable.

#### Syntax Rules and Structure

```cpp
std::any a = 42;                    // stores int
a = std::string("hello");           // stores string
a = 3.14;                           // stores double
a.reset();                          // empty

int i = std::any_cast<int>(a);      // throws if wrong type
int* p = std::any_cast<int>(&a);    // returns nullptr if wrong type
```

**Constraints and limitations:**
- `any_cast<T>` requires `T` to be exactly the stored type (no implicit conversions).
- Throws `std::bad_any_cast` on type mismatch (unless using pointer overload).
- Not all types are copy-constructible; `any` requires copy-constructibility.
- Performance overhead: may allocate dynamically for large types.
- No support for move-only types (e.g., `std::unique_ptr`) without workarounds.

#### Annotated Code Example

```cpp
// File: any_example.cpp
// Compile with: g++ -std=c++17 any_example.cpp -o any_example

#include <any>
#include <iostream>
#include <string>

int main() {
    // Step 1: Store different types
    std::any a = 42;
    std::cout << "int: " << std::any_cast<int>(a) << std::endl;

    a = std::string("hello");
    std::cout << "string: " << std::any_cast<std::string>(a) << std::endl;

    a = 3.14;
    std::cout << "double: " << std::any_cast<double>(a) << std::endl;

    // Step 2: Check type and empty
    std::cout << "has_value: " << a.has_value() << std::endl;
    std::cout << "type: " << a.type().name() << std::endl;

    // Step 3: Safe access with pointer
    if (auto* p = std::any_cast<double>(&a)) {
        std::cout << "Pointer cast: " << *p << std::endl;
    }

    // Step 4: Wrong type throws
    try {
        std::cout << std::any_cast<int>(a) << std::endl;
    } catch (const std::bad_any_cast& e) {
        std::cout << "Caught: " << e.what() << std::endl;
    }

    // Step 5: Reset
    a.reset();
    std::cout << "After reset, has_value: " << a.has_value() << std::endl;

    return 0;
}
```

**Expected Output:**
```
int: 42
string: hello
double: 3.14
has_value: 1
type: d
Pointer cast: 3.14
Caught: std::bad_any_cast
After reset, has_value: 0
```

**Why this output:** `any_cast<int>` retrieves the stored integer. `any_cast<std::string>` retrieves the string. `type().name()` returns an implementation-defined name (e.g., "d" for double in GCC). The pointer overload returns `nullptr` on mismatch. The value overload throws `std::bad_any_cast`.

---

### Sub-Feature: `std::reference_wrapper`

#### Definitions

**Core Definition**
`std::reference_wrapper<T>` is a copyable, assignable wrapper around a reference to an object or function of type `T`.

**Technical Definition**
`template<class T> class reference_wrapper;` (C++11, `<functional>`). It is CopyConstructible and CopyAssignable and is guaranteed to be TriviallyCopyable since C++17. It is implicitly convertible to `T&`. When the stored reference is callable, `reference_wrapper` is also callable with the same arguments. Helper functions `std::ref(t)` and `std::cref(t)` create reference wrappers with deduced types. It is used to pass objects by reference to `std::bind`, `std::thread`, and standard containers (which normally cannot hold references).

**Beginner-Friendly Explanation**
`std::reference_wrapper` is a container that holds a reference instead of a copy. It lets you put references into `std::vector` or pass them to `std::thread` without the thread making a copy. It is like a pointer that behaves like a reference.

#### Purposes (all begin with "To")

- **To** store references in standard containers that cannot hold plain references.
- **To** pass objects by reference to `std::bind` and `std::thread`.
- **To** provide a copyable, assignable reference wrapper.
- **To** allow functions to accept references through templates that normally decay to values.
- **To** enable reference semantics in generic code.

#### Syntax Rules and Structure

```cpp
// Direct construction
std::reference_wrapper<int> r = std::ref(x);

// Helper functions
auto r = std::ref(x);   // reference_wrapper<T>
auto cr = std::cref(x); // reference_wrapper<const T>

// Usage
r.get() = 42;           // modifies x
int& ref = r;           // implicit conversion to T&
```

**Constraints and limitations:**
- Cannot be default-constructed (must always refer to something).
- Cannot be rebound to a different object (assignment copies the reference, not the object).
- `reference_wrapper` is not a reference itself; it is an object that holds a pointer.
- Copying a `reference_wrapper` copies the reference, not the underlying object.
- A `reference_wrapper` may dangle if the referenced object is destroyed.

#### Annotated Code Example

```cpp
// File: ref_wrapper_example.cpp
// Compile with: g++ -std=c++17 ref_wrapper_example.cpp -o ref_wrapper_example

#include <functional>
#include <iostream>
#include <vector>

int main() {
    // Step 1: Store references in a vector
    int a = 1, b = 2, c = 3;
    std::vector<std::reference_wrapper<int>> refs = {std::ref(a),
                                                      std::ref(b),
                                                      std::ref(c)};

    // Step 2: Modify through references
    for (auto& r : refs) {
        r.get() *= 10;
    }

    std::cout << "a=" << a << " b=" << b << " c=" << c << std::endl;

    // Step 3: Implicit conversion to T&
    int& ref_a = refs[0];
    ref_a = 100;
    std::cout << "a after ref_a=100: " << a << std::endl;

    // Step 4: Use with std::bind
    auto add = [](int x, int y) { return x + y; };
    auto bound = std::bind(add, std::placeholders::_1, std::ref(a));
    std::cout << "bound(5): " << bound(5) << std::endl;

    // Step 5: Callable reference_wrapper
    auto lambda = [](int x) { return x * 2; };
    std::reference_wrapper<decltype(lambda)> ref_lambda = std::ref(lambda);
    std::cout << "ref_lambda(21): " << ref_lambda(21) << std::endl;

    return 0;
}
```

**Expected Output:**
```
a=100 b=20 c=30
a after ref_a=100: 100
bound(5): 105
ref_lambda(21): 42
```

**Why this output:** The vector stores references to `a`, `b`, and `c`. Modifying through `r.get()` modifies the original variables. `ref_a` is an implicit conversion to `int&`, so assigning 100 changes `a`. `std::ref(a)` in `std::bind` ensures the bound argument is a reference. `reference_wrapper` is callable if the underlying object is callable.

---

### Real-World Cases with Explanation

**Case 1 — Observer pattern**: A vector of `reference_wrapper<Observer>` stores observers without owning them.

**Case 2 — Thread arguments**: `std::thread` copies its arguments by default; wrapping with `std::ref` passes by reference.

**Case 3 — Generic containers**: A container of `reference_wrapper<T>` allows storing non-owning references to objects.

**Case 4 — Plugin systems**: `std::any` stores plugin data of any type, and `any_cast` retrieves it with type safety.

**Case 5 — Command pattern**: `std::any` holds command parameters of varying types, and `any_cast` retrieves them in the command handler.

---

### References

- std::any — cppreference.com - https://en.cppreference.com/w/cpp/utility/any
- std::any_cast — cppreference.com - https://en.cppreference.com/w/cpp/utility/any/any_cast
- std::reference_wrapper — cppreference.com - https://en.cppreference.com/w/cpp/utility/functional/reference_wrapper
- std::ref, std::cref — cppreference.com - https://en.cppreference.com/w/cpp/utility/functional/ref
- std::reference_wrapper::get — cppreference.com - https://en.cppreference.com/w/cpp/utility/functional/reference_wrapper/get
- std::reference_wrapper::operator() — cppreference.com - https://en.cppreference.com/w/cpp/utility/functional/reference_wrapper/operator()

---

## Summary Table

| Type | Header | Standard | Purpose | Key Feature |
|---|---|---|---|---|
| `std::pair` | `<utility>` | C++98 | Two heterogeneous values | `.first`, `.second`, structured bindings |
| `std::tuple` | `<tuple>` | C++11 | N heterogeneous values | `std::get<N>`, structured bindings |
| `std::optional` | `<optional>` | C++17 | Value may be absent | Monadic ops in C++23 |
| `std::variant` | `<variant>` | C++17 | Type-safe union | `std::visit`, `std::get` |
| `std::expected` | `<expected>` | C++23 | Value or error | Monadic ops, no exceptions |
| `std::any` | `<any>` | C++17 | Any copyable type | `any_cast`, `has_value()` |
| `std::reference_wrapper` | `<functional>` | C++11 | Copyable reference | `std::ref`, implicit `T&` conversion |

---

## References (Consolidated)

- std::pair — cppreference.com - https://en.cppreference.com/w/cpp/utility/pair
- std::tuple — cppreference.com - https://en.cppreference.com/w/cpp/utility/tuple
- std::optional — cppreference.com - https://en.cppreference.com/w/cpp/utility/optional
- std::variant — cppreference.com - https://en.cppreference.com/w/cpp/utility/variant
- std::expected — cppreference.com - https://en.cppreference.com/w/cpp/utility/expected
- std::any — cppreference.com - https://en.cppreference.com/w/cpp/utility/any
- std::reference_wrapper — cppreference.com - https://en.cppreference.com/w/cpp/utility/functional/reference_wrapper
- Structured binding declaration — cppreference.com - https://en.cppreference.com/w/cpp/language/structured_binding
- std::visit — cppreference.com - https://en.cppreference.com/w/cpp/utility/variant/visit
- std::optional monadic operations (C++23) — Dr-Sergey/learn_cpp - https://github.com/Dr-Sergey/learn_cpp/blob/main/09_Standard_Library_Utilities/Use_stdoptional_monadic_operations_transform_and_then_or_else_C%2B%2B23.md
- P0323R12: std::expected — Open-Std - https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2022/p0323r12.html
- P0798R8: Monadic operations for std::optional — Open-Std - https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2021/p0798r8.html
- P1957R2: Converting from T* to bool should be considered narrowing — Open-Std - https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2020/p1957r2.html