# C++ Function Fundamentals — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
A C++ function is a named entity that associates a sequence of statements (the *function body*) with a list of zero or more parameters, enabling code reuse, abstraction, and modular program design.

**Technical Definition**
In the C++ language, a function is a construct whose *type* is composed of its return type, the types of all parameters (after array-to-pointer and function-to-pointer adjustments), whether the function is `noexcept` (since C++17), and, for non-static member functions, cv-qualification and ref-qualification (since C++11). A function declaration introduces the function name and its type; a function definition associates the name/type with a function body.

**Beginner-Friendly Explanation**
Think of a function as a small, reusable machine: you give it some input (arguments), it performs a specific job (the body), and it gives you back a result (the return value) — or it may simply do something without returning anything. Functions let you avoid repeating the same code and make programs easier to read and maintain.

### Key Characteristics

- **Named entity** – every function has an identifier (except lambdas).
- **Strongly typed** – parameters and return types are part of the function’s type.
- **Not an object** – functions cannot be stored in arrays or passed by value, but pointers/references to functions are allowed.
- **Can be overloaded** – multiple functions may share the same name if their parameter lists differ.
- **Can be templated** – functions can operate on generic types.
- **Can participate in compile-time evaluation** – via `constexpr`, `consteval`, and `constinit`.
- **Can carry a `noexcept` specification** – indicating whether the function may throw exceptions.
- **Addressable** – you can take the address of most functions (except `main` and some standard-library functions).

### Prerequisites

To understand this material, the reader should be familiar with:

- Basic C++ syntax (variables, types, operators, control flow).
- The concept of scope and lifetime.
- Basic compilation and linking (translation units, headers, source files).
- Elementary notions of object-oriented programming (classes, member functions) — helpful but not strictly required for all sections.

### Related Programming Areas

- **Object-oriented programming** – member functions, virtual functions, constructors/destructors.
- **Generic programming** – function templates, concepts (C++20).
- **Compile-time computation** – `constexpr`, `consteval`, `constinit`, template metaprogramming.
- **Exception safety** – `noexcept`, exception specifications.
- **Performance optimization** – `inline`, parameter passing conventions, return-value optimization.
- **API design** – function signatures, default arguments, overloading.

### Core Concepts / Features

1. Functions
2. Function Declaration
3. Function Definition
4. Function Prototype
5. Function Signature
6. Function Parameters
7. Function Arguments
8. Return Types
9. `void` Return
10. Function Body
11. Function Call
12. Function Overloading
13. Function Templates
14. Default Arguments
15. Inline Functions
16. `constexpr` Functions
17. `consteval` Functions
18. `constinit`
19. `noexcept`
20. Function Best Practices

---

## 1. Functions

### Definitions

**Core Definition**
A function is a C++ entity that associates a sequence of statements (a function body) with a name and a list of zero or more function parameters.

**Technical Definition**
A function has a *type* consisting of its return type, the types of all parameters (after array-to-pointer and function-to-pointer adjustments), its `noexcept` specification (since C++17), and, for non-static member functions, cv- and ref-qualifications (since C++11). Functions are not objects; they cannot be passed by value or stored in arrays, but pointers and references to functions are permitted.

**Beginner-Friendly Explanation**
A function is a reusable block of code that performs a task. You “call” it by name, optionally give it inputs, and it may return a result.

### Purposes

- To avoid code duplication by packaging reusable logic into named units.
- To break complex problems into smaller, manageable pieces.
- To provide abstraction and encapsulation.
- To enable code organization and readability.
- To facilitate testing and maintenance.

### Syntax Rules and Structure

**General Syntax**
```cpp
return_type function_name(parameter_list) function_body
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `return_type` | The type of the value returned by the function (or `void`). |
| `function_name` | A valid C++ identifier. |
| `parameter_list` | Comma-separated list of parameters (may be empty). |
| `function_body` | A compound statement `{ ... }` containing the executable statements. |

**Syntax Rules**
- A function declaration may appear in any scope; a function definition may appear only in namespace scope or, for member/friend functions, in class scope.
- A function that is declared in a class body without a `friend` specifier is a class member function.
- Functions cannot be returned by value or passed by value; only pointers/references to functions are allowed.

**Constraints and Limitations**
- The `main` function is not addressable in the same way as ordinary functions.
- Some standard-library functions may not be addressable (since C++20).

### Annotated Code Examples

**Example 1: A simple function**

```cpp
#include <iostream>

// Function definition: returns bool, takes one int parameter
bool isodd(int n) {
    return n % 2;          // returns true if n is odd
}

int main() {
    std::cout << isodd(3) << '\n';   // prints 1 (true)
    std::cout << isodd(4) << '\n';   // prints 0 (false)
}
```

**Expected Output**
```
1
0
```

**Why this output?**  
`isodd(3)` computes `3 % 2`, which is `1` (true). `isodd(4)` computes `4 % 2`, which is `0` (false). The function body is executed each time the function is called.

**Example 2: Function with no parameters**

```cpp
#include <iostream>

void greet() {
    std::cout << "Hello, world!\n";
}

int main() {
    greet();   // function call
}
```

**Expected Output**
```
Hello, world!
```

**Why this output?**  
`greet` takes no parameters and simply prints the string when invoked.

### Real-World Cases

- **Mathematical utilities** – `sqrt`, `pow`, `abs` are implemented as functions.
- **String processing** – functions like `std::toupper`, `std::strlen`.
- **Event handlers** – functions invoked in response to user actions.
- **Business logic** – functions encapsulating validation, calculations, or database operations.

---

## 2. Function Declaration

### Definitions

**Core Definition**
A function declaration introduces the function name and its type into a scope, informing the compiler about the function’s existence.

**Technical Definition**
A function declaration specifies the return type, the function name, the parameter list, and optionally attributes, cv-qualifiers, ref-qualifiers, and a `noexcept` specification. It does not provide the function body.

**Beginner-Friendly Explanation**
A function declaration is like a promise: “There will be a function called X that takes these inputs and returns this type.” It tells the compiler what to expect before the actual implementation appears.

### Purposes

- To make a function known to the compiler before its definition.
- To allow functions to be called from other translation units.
- To separate interface from implementation.
- To enable mutual recursion.
- To support library headers that declare functions without defining them.

### Syntax Rules and Structure

**General Syntax**
```cpp
return_type function_name(parameter_list);
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `return_type` | The type of the returned value. |
| `function_name` | The identifier of the function. |
| `parameter_list` | A comma-separated list of parameter declarations (types, optionally names). |

**Syntax Rules**
- A function declaration may appear in any scope.
- Parameter names in declarations are optional and serve documentation purposes only.
- A declaration may include `noexcept`, `constexpr`, `inline`, `virtual`, `explicit`, `friend`, and other specifiers.

**Constraints and Limitations**
- A function declaration alone does not satisfy the linker; a definition must exist somewhere if the function is used.
- Declarations that differ only in return type cannot overload each other.

### Annotated Code Examples

**Example 1: Forward declaration**

```cpp
#include <iostream>

// Declaration (prototype)
int add(int a, int b);

int main() {
    std::cout << add(2, 3) << '\n';   // OK: declaration seen before use
}

// Definition
int add(int a, int b) {
    return a + b;
}
```

**Expected Output**
```
5
```

**Why this output?**  
The declaration tells the compiler that `add` exists and takes two `int`s, returning an `int`. The definition is later linked, and the call succeeds.

**Example 2: Declaration in a header**

```cpp
// math_utils.h
#pragma once
int multiply(int x, int y);   // declaration only
```

```cpp
// main.cpp
#include "math_utils.h"
#include <iostream>

int main() {
    std::cout << multiply(4, 5) << '\n';
}

// math_utils.cpp
int multiply(int x, int y) { return x * y; }
```

**Expected Output**
```
20
```

**Why this output?**  
The header provides the declaration; the definition is in a separate translation unit. The linker resolves the call.

### Real-World Cases

- **Header files** – virtually every C++ header contains function declarations.
- **Library APIs** – public headers declare functions whose implementations are in compiled libraries.
- **Mutual recursion** – two functions that call each other need forward declarations.

---

## 3. Function Definition

### Definitions

**Core Definition**
A function definition associates the function name and type with the function body, providing the actual executable statements.

**Technical Definition**
A function definition consists of a function header (return type, name, parameter list, and optional specifiers) followed by a compound statement `{ ... }` that constitutes the function body.

**Beginner-Friendly Explanation**
The definition is the “real” function — it contains the code that actually runs when the function is called.

### Purposes

- To provide the implementation of a function.
- To make the function usable by the linker.
- To define the behavior that the declaration promises.
- To allow inlining and compile-time evaluation (when combined with `inline` or `constexpr`).

### Syntax Rules and Structure

**General Syntax**
```cpp
return_type function_name(parameter_list) {
    // statements
}
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `return_type` | The type of the returned value. |
| `function_name` | The function identifier. |
| `parameter_list` | Parameter declarations (names are required in the definition if the parameters are used). |
| `function_body` | A compound statement containing the executable code. |

**Syntax Rules**
- A function definition may appear only in namespace scope or, for member/friend functions, in class scope.
- Every declared function that is odr-used must have exactly one definition (One Definition Rule).
- Inline functions and templates may have multiple definitions across translation units, provided they are identical.

**Constraints and Limitations**
- A function definition cannot appear inside another function (except lambdas).
- The return type cannot be deduced in C++11/14 for ordinary functions (C++14 allows `auto` return type deduction, but constraints apply).

### Annotated Code Examples

**Example 1: Basic definition**

```cpp
#include <iostream>

int square(int x) {      // definition
    return x * x;
}

int main() {
    std::cout << square(5) << '\n';
}
```

**Expected Output**
```
25
```

**Why this output?**  
`square(5)` computes `5 * 5` and returns `25`, which is printed.

**Example 2: Definition with multiple statements**

```cpp
#include <iostream>

int max_of_three(int a, int b, int c) {
    int max = a;
    if (b > max) max = b;
    if (c > max) max = c;
    return max;
}

int main() {
    std::cout << max_of_three(3, 7, 5) << '\n';
}
```

**Expected Output**
```
7
```

**Why this output?**  
The function body executes sequentially, updating `max` to the largest value, then returns `7`.

### Real-World Cases

- **Algorithm implementations** – sorting, searching, numerical methods.
- **Business logic** – fee calculation, tax computation.
- **Infrastructure code** – logging, memory allocation wrappers.

---

## 4. Function Prototype

### Definitions

**Core Definition**
A function prototype is a declaration that specifies the function’s return type, name, and parameter types, without providing the body.

**Technical Definition**
In C++, a function prototype is essentially a function declaration that includes the parameter type list; it may omit parameter names. It enables the compiler to perform type checking on calls.

**Beginner-Friendly Explanation**
A prototype is a “preview” of a function — it tells the compiler the function’s name, what it returns, and what kinds of arguments it expects, so that calls can be checked before the full definition is seen.

### Purposes

- To enable early type checking of function calls.
- To allow functions to be used before their definition.
- To support separate compilation.
- To document the interface of a function.

### Syntax Rules and Structure

**General Syntax**
```cpp
return_type function_name(type1, type2, ...);
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `return_type` | The type of the returned value. |
| `function_name` | The function identifier. |
| `type1, type2, ...` | The types of the parameters (names optional). |

**Syntax Rules**
- A prototype must match the definition in return type and parameter types.
- Parameter names in a prototype are ignored by the compiler.
- A prototype may include `noexcept`, `constexpr`, etc.

**Constraints and Limitations**
- A prototype that differs from the definition in parameter types or return type causes a compilation error.
- In C++, `void` in the parameter list means “no parameters”; an empty parameter list also means “no parameters” (unlike C, where it means unspecified).

### Annotated Code Examples

**Example 1: Prototype before definition**

```cpp
#include <iostream>

int multiply(int, int);   // prototype with unnamed parameters

int main() {
    std::cout << multiply(3, 4) << '\n';
}

int multiply(int a, int b) {
    return a * b;
}
```

**Expected Output**
```
12
```

**Why this output?**  
The prototype tells the compiler that `multiply` takes two `int`s and returns `int`. The definition matches, and the call succeeds.

**Example 2: Prototype in a header**

```cpp
// operations.h
#ifndef OPERATIONS_H
#define OPERATIONS_H
int subtract(int x, int y);
#endif
```

```cpp
// main.cpp
#include "operations.h"
#include <iostream>

int main() {
    std::cout << subtract(10, 3) << '\n';
}

// operations.cpp
int subtract(int x, int y) { return x - y; }
```

**Expected Output**
```
7
```

**Why this output?**  
The header provides the prototype; the implementation is linked from another translation unit.

### Real-World Cases

- **Header files** – prototypes are the primary mechanism for exposing functions across translation units.
- **Third-party libraries** – installed headers provide prototypes.
- **Code documentation** – prototypes serve as concise interface summaries.

---

## 5. Function Signature

### Definitions

**Core Definition**
The signature of a function consists of its name and its parameter-type-list.

**Technical Definition**
In C++, the function signature comprises the function name, the parameter-type-list (after adjustments), and, for member functions, the class of which the function is a member and its cv-/ref-qualifiers. The return type is **not** part of the signature, nor is the `noexcept` specification (since C++17) for overloading purposes.

**Beginner-Friendly Explanation**
The signature is the “identity card” of a function for overload resolution: it includes the function’s name and the types of its parameters, but not what it returns.

### Purposes

- To determine which function overload is selected during overload resolution.
- To distinguish between different overloads.
- To define the interface of a function in a type-safe manner.
- To enable name mangling (implementation detail).

### Syntax Rules and Structure

**General Syntax**
The signature is not written explicitly; it is an implicit property of a function declaration/definition.

**Component Breakdown**

| Component | Included in signature? |
|-----------|------------------------|
| Function name | Yes |
| Parameter types | Yes |
| Return type | No |
| `noexcept` (since C++17) | No (for overloading) |
| cv-/ref-qualifiers (member functions) | Yes |
| Enclosing namespace/class | Yes (for linkage) |

**Syntax Rules**
- Two functions in the same scope can be overloaded if their signatures differ.
- Functions that differ only in return type (and `noexcept` since C++17) cannot be overloaded.
- The signature is used during overload resolution to select the best match.

**Constraints and Limitations**
- Default arguments are **not** part of the signature.
- Template parameters are part of the signature of a function template specialization.

### Annotated Code Examples

**Example 1: Overloading based on signature**

```cpp
#include <iostream>

void print(int x)   { std::cout << "int: " << x << '\n'; }
void print(double x) { std::cout << "double: " << x << '\n'; }

int main() {
    print(42);      // selects print(int)
    print(3.14);    // selects print(double)
}
```

**Expected Output**
```
int: 42
double: 3.14
```

**Why this output?**  
The signatures differ in the parameter type (`int` vs `double`), so overloading is allowed. The argument type determines which overload is called.

**Example 2: Return type alone cannot overload**

```cpp
int f() { return 1; }
// double f() { return 2.0; }   // error: cannot overload solely on return type
```

**Expected Output**  
(Compilation error)

**Why this error?**  
The signatures are identical (name `f`, empty parameter list), so the compiler cannot distinguish them.

### Real-World Cases

- **Overload sets** – e.g., `std::sqrt(float)`, `std::sqrt(double)`, `std::sqrt(long double)`.
- **Operator overloading** – operators are functions with special signatures.
- **Template specialization** – signatures guide template argument deduction.

---

## 6. Function Parameters

### Definitions

**Core Definition**
Function parameters are the variables declared in the function’s parameter list that receive the values of the arguments passed at the call site.

**Technical Definition**
Parameters are local variables with automatic storage duration within the function body. Their types (after array-to-pointer and function-to-pointer adjustments) contribute to the function’s type and signature.

**Beginner-Friendly Explanation**
Parameters are the “input slots” of a function. When you call a function, you fill these slots with values (arguments).

### Purposes

- To allow functions to accept input data.
- To enable parameterization of behavior.
- To support overloading and templates.
- To define the function’s interface.

### Syntax Rules and Structure

**General Syntax**
```cpp
return_type function_name(type1 name1, type2 name2, ...);
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `type` | The type of the parameter. |
| `name` | The identifier used inside the function body. |

**Syntax Rules**
- Parameters are separated by commas.
- A parameter list may be empty (`()` or `(void)`).
- Default arguments may be specified for trailing parameters.
- Parameters may be passed by value, by reference (`&`), by pointer (`*`), by rvalue reference (`&&`), or with `const` qualification.

**Constraints and Limitations**
- A parameter cannot have a type that is an array of unknown bound or a function type (these decay to pointers).
- Parameter names are optional in declarations but required in definitions if the parameter is used.
- The `volatile` qualifier on parameters is deprecated.

### Annotated Code Examples

**Example 1: Pass by value**

```cpp
#include <iostream>

void increment(int x) {
    ++x;                 // modifies local copy
}

int main() {
    int a = 5;
    increment(a);
    std::cout << a << '\n';   // a is unchanged
}
```

**Expected Output**
```
5
```

**Why this output?**  
The parameter `x` is a copy of `a`; modifying `x` does not affect `a`.

**Example 2: Pass by reference**

```cpp
#include <iostream>

void increment(int& x) {
    ++x;                 // modifies original
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
`x` is a reference to `a`, so `++x` modifies `a` directly.

**Example 3: Default argument**

```cpp
#include <iostream>

void greet(const std::string& name = "Guest") {
    std::cout << "Hello, " << name << "!\n";
}

int main() {
    greet();            // uses default
    greet("Alice");     // uses provided argument
}
```

**Expected Output**
```
Hello, Guest!
Hello, Alice!
```

**Why this output?**  
The default argument `"Guest"` is used when no argument is supplied.

### Real-World Cases

- **Configuration** – parameters allow functions to be customized.
- **Callbacks** – function parameters accept function pointers or `std::function`.
- **Generic algorithms** – parameters of template types.

---

## 7. Function Arguments

### Definitions

**Core Definition**
Function arguments are the actual values or objects passed to a function at the call site, which initialize the function’s parameters.

**Technical Definition**
Arguments are expressions provided in a function-call expression. They are used to initialize the corresponding parameters according to the parameter passing mechanism (copy-initialization for by-value, reference binding for references).

**Beginner-Friendly Explanation**
Arguments are the “actual data” you hand to a function when you call it. Parameters are the “slots” that receive this data.

### Purposes

- To supply input data to functions.
- To control function behavior at runtime.
- To enable reuse of the same function with different data.
- To facilitate communication between caller and callee.

### Syntax Rules and Structure

**General Syntax**
```cpp
function_name(argument1, argument2, ...)
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `argument` | An expression whose value is used to initialize a parameter. |

**Syntax Rules**
- The number and types of arguments must be compatible with the parameter list (after default arguments are applied).
- Arguments are evaluated before the function is entered (order of evaluation may be unspecified).
- Arguments may be lvalues, rvalues, or prvalues.

**Constraints and Limitations**
- Passing an argument of incompatible type may require an implicit conversion, which can fail.
- Too many or too few arguments result in a compilation error (unless variadic or defaulted).

### Annotated Code Examples

**Example 1: Multiple arguments**

```cpp
#include <iostream>

int add(int a, int b) {
    return a + b;
}

int main() {
    int x = 3, y = 4;
    std::cout << add(x, y) << '\n';   // x and y are arguments
    std::cout << add(10, 20) << '\n'; // literals are arguments
}
```

**Expected Output**
```
7
30
```

**Why this output?**  
The arguments `x` and `y` initialize `a` and `b`; the function returns their sum. Literals `10` and `20` are also valid arguments.

**Example 2: Argument evaluation order**

```cpp
#include <iostream>

void show(int a, int b) {
    std::cout << a << ' ' << b << '\n';
}

int main() {
    int i = 0;
    show(++i, ++i);   // order of evaluation is unspecified
}
```

**Possible Output** (implementation-dependent)
```
1 2
```
or
```
2 1
```

**Why this output?**  
The order in which function arguments are evaluated is unspecified in C++; different compilers may produce different results. This is a classic pitfall.

### Real-World Cases

- **User input** – passing user-provided values to processing functions.
- **Mathematical computations** – arguments are the operands.
- **Event handling** – arguments carry event data.

---

## 8. Return Types

### Definitions

**Core Definition**
The return type of a function specifies the type of the value that the function produces and returns to the caller.

**Technical Definition**
The return type is part of the function’s type but **not** part of its signature for overloading purposes. It may be `void`, a fundamental type, a class type, a reference type, a pointer type, or `auto` (with deduction).

**Beginner-Friendly Explanation**
The return type tells you what kind of value the function gives back after it finishes its work. If it gives nothing back, the return type is `void`.

### Purposes

- To communicate the result of a computation.
- To enable chaining of function calls.
- To support type-safe programming.
- To allow generic algorithms to deduce result types (`auto`, `decltype(auto)`).

### Syntax Rules and Structure

**General Syntax**
```cpp
return_type function_name(parameters) { ... }
```

**Component Breakdown**

| Return Type Form | Meaning |
|------------------|---------|
| `void` | No value returned. |
| `T` | A value of type `T` is returned (by value). |
| `T&` | A reference to `T` is returned. |
| `T*` | A pointer to `T` is returned. |
| `auto` | Return type deduced from `return` statements (C++14). |
| `decltype(auto)` | Deduces reference/value category (C++14). |

**Syntax Rules**
- A function with a non-`void` return type must return a value on all control paths.
- `return` statement exits the function and initializes the return value.
- The return type is not part of the function signature for overloading.

**Constraints and Limitations**
- Returning a reference to a local variable results in a dangling reference (undefined behavior).
- `auto` return type deduction requires all `return` statements to yield the same type.

### Annotated Code Examples

**Example 1: Returning by value**

```cpp
#include <iostream>

int square(int x) {
    return x * x;
}

int main() {
    std::cout << square(6) << '\n';
}
```

**Expected Output**
```
36
```

**Why this output?**  
The `return` statement computes `x * x` and returns the result; `main` prints it.

**Example 2: Returning a reference**

```cpp
#include <iostream>
#include <vector>

int& at(std::vector<int>& v, int index) {
    return v[index];   // returns reference to element
}

int main() {
    std::vector<int> nums = {10, 20, 30};
    at(nums, 1) = 99;   // modifies the element
    std::cout << nums[1] << '\n';
}
```

**Expected Output**
```
99
```

**Why this output?**  
The function returns a reference to `v[1]`, so assigning to it modifies the vector element.

**Example 3: `auto` return type (C++14)**

```cpp
#include <iostream>

auto add(int a, int b) {
    return a + b;   // deduced as int
}

int main() {
    std::cout << add(2, 3) << '\n';
}
```

**Expected Output**
```
5
```

**Why this output?**  
The compiler deduces the return type as `int` from the expression `a + b`.

### Real-World Cases

- **Getter functions** – returning member values (often by reference or `const` reference).
- **Factory functions** – returning newly created objects.
- **Mathematical functions** – returning computed results.

---

## 9. `void` Return

### Definitions

**Core Definition**
`void` is a special return type indicating that the function does not return a value.

**Technical Definition**
A function declared with return type `void` has no return value; a `return` statement without an expression may be used to exit early. The `void` return type is part of the function’s type but not its signature.

**Beginner-Friendly Explanation**
A `void` function does something (e.g., prints, modifies data) but does not hand back a result.

### Purposes

- To indicate that a function performs an action without producing a value.
- To allow functions to have side effects only.
- To simplify interfaces where no result is needed.
- To enable uniform syntax for callbacks.

### Syntax Rules and Structure

**General Syntax**
```cpp
void function_name(parameters) {
    // statements
    // optional: return; (no value)
}
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `void` | Return type keyword. |
| `return;` | Optional early exit (no expression). |

**Syntax Rules**
- A `void` function may use `return;` without a value.
- You cannot use the result of a `void` function in an expression.
- `void` functions can be used in contexts requiring a function with no return value (e.g., `std::function<void()>`).

**Constraints and Limitations**
- Dereferencing a `void*` is illegal; `void` as a return type is distinct from `void*`.

### Annotated Code Examples

**Example 1: `void` function with side effect**

```cpp
#include <iostream>

void print_hello() {
    std::cout << "Hello!\n";
    // no return statement needed
}

int main() {
    print_hello();
}
```

**Expected Output**
```
Hello!
```

**Why this output?**  
The function performs an action (printing) but returns no value.

**Example 2: Early return in a `void` function**

```cpp
#include <iostream>

void check_positive(int x) {
    if (x <= 0) {
        std::cout << "Non-positive\n";
        return;          // early exit
    }
    std::cout << "Positive\n";
}

int main() {
    check_positive(-5);
    check_positive(3);
}
```

**Expected Output**
```
Non-positive
Positive
```

**Why this output?**  
The `return;` exits the function early when `x <= 0`; otherwise execution continues.

### Real-World Cases

- **Logging functions** – `void log(const std::string& message)`.
- **Mutators** – `void set_name(const std::string&)`.
- **Event handlers** – `void on_click()`.

---

## 10. Function Body

### Definitions

**Core Definition**
The function body is the compound statement (enclosed in braces) that contains the executable statements of a function.

**Technical Definition**
The function body is a *compound-statement* that defines the scope, lifetime of local variables, and the sequence of operations performed when the function is called.

**Beginner-Friendly Explanation**
The body is the “inside” of the function — all the instructions that run when you call it.

### Purposes

- To contain the actual logic of the function.
- To define local variables and their scope.
- To control the flow of execution.
- To encapsulate the implementation details.

### Syntax Rules and Structure

**General Syntax**
```cpp
return_type function_name(parameters) {
    // statements
}
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `{ ... }` | A compound statement delimiting the body. |
| Statements | Declarations, expressions, control-flow statements, etc. |

**Syntax Rules**
- The body is a single compound statement.
- Local variables declared in the body have automatic storage duration.
- The body defines a new scope.
- Every function definition must have exactly one body.

**Constraints and Limitations**
- The body cannot be omitted in a definition (except for `= delete` or `= default`).
- Unreachable code may trigger warnings.

### Annotated Code Examples

**Example 1: Body with local variables**

```cpp
#include <iostream>

int compute(int a, int b) {
    int sum = a + b;        // local variable
    int product = a * b;    // local variable
    return sum + product;
}

int main() {
    std::cout << compute(2, 3) << '\n';
}
```

**Expected Output**
```
11
```

**Why this output?**  
`sum = 5`, `product = 6`, result `11`.

**Example 2: Body with control flow**

```cpp
#include <iostream>

void print_parity(int n) {
    if (n % 2 == 0) {
        std::cout << n << " is even\n";
    } else {
        std::cout << n << " is odd\n";
    }
}

int main() {
    print_parity(7);
    print_parity(10);
}
```

**Expected Output**
```
7 is odd
10 is even
```

**Why this output?**  
The body contains an `if-else` that selects the appropriate message.

### Real-World Cases

- **Algorithm implementations** – sorting, searching, numerical computation.
- **Business rules** – validation logic, pricing calculations.
- **State machines** – functions that update object state.

---

## 11. Function Call

### Definitions

**Core Definition**
A function call is an expression that invokes a function, passing arguments and transferring control to the function body.

**Technical Definition**
A function-call expression consists of a postfix expression denoting a function, followed by a parenthesized list of arguments. The arguments initialize the parameters, and the function body executes.

**Beginner-Friendly Explanation**
Calling a function means “running” it by writing its name followed by parentheses containing any inputs.

### Purposes

- To execute the function’s code.
- To obtain the function’s return value.
- To trigger side effects.
- To compose larger computations from smaller functions.

### Syntax Rules and Structure

**General Syntax**
```cpp
function_name(argument_list)
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `function_name` | The function to call (may be qualified). |
| `argument_list` | Comma-separated expressions passed as arguments. |

**Syntax Rules**
- The number and types of arguments must match the parameters (after default arguments).
- For member functions, the object is specified using `.` or `->`.
- Function pointers can be called using `(*ptr)(args)` or `ptr(args)`.
- Overload resolution selects the best-matching function.

**Constraints and Limitations**
- Calling a function with the wrong number or types of arguments is a compile-time error.
- Calling a function through a null pointer results in undefined behavior.

### Annotated Code Examples

**Example 1: Basic call**

```cpp
#include <iostream>

int add(int a, int b) { return a + b; }

int main() {
    int result = add(3, 4);   // function call
    std::cout << result << '\n';
}
```

**Expected Output**
```
7
```

**Why this output?**  
The call `add(3, 4)` executes the function and returns `7`.

**Example 2: Call with default argument**

```cpp
#include <iostream>

void greet(const std::string& name = "World") {
    std::cout << "Hello, " << name << "!\n";
}

int main() {
    greet();            // uses default
    greet("C++");       // explicit argument
}
```

**Expected Output**
```
Hello, World!
Hello, C++!
```

**Why this output?**  
The first call omits the argument, so the default is used; the second provides an explicit argument.

### Real-World Cases

- **Event-driven programming** – callback invocations.
- **Functional composition** – `f(g(x))`.
- **Recursive algorithms** – functions calling themselves.

---

## 12. Function Overloading

### Definitions

**Core Definition**
Function overloading allows multiple functions in the same scope to share the same name, provided their parameter lists differ.

**Technical Definition**
Overloading is resolved by the compiler through *overload resolution*, which selects the function whose parameters best match the arguments. Functions that differ only in return type (or `noexcept` since C++17) cannot be overloaded.

**Beginner-Friendly Explanation**
You can have several functions with the same name as long as they take different inputs (different number or types of parameters). The compiler figures out which one to call based on what you pass.

### Purposes

- To provide a natural interface for operations on different types.
- To reduce the number of distinct function names.
- To support generic-style programming without templates in simple cases.
- To improve code readability.

### Syntax Rules and Structure

**General Syntax**
```cpp
return_type function_name(type1);
return_type function_name(type2);
// ...
```

**Component Breakdown**

| Overload Distinguisher | Allowed? |
|------------------------|----------|
| Different parameter types | Yes |
| Different number of parameters | Yes |
| Different cv-/ref-qualifiers (member functions) | Yes |
| Different return type only | No |
| Different `noexcept` only | No (since C++17) |

**Syntax Rules**
- Overloads must be in the same scope.
- Default arguments are not part of the signature and do not create distinct overloads.
- `const` and non-`const` member functions can overload each other.

**Constraints and Limitations**
- Ambiguous calls can occur if implicit conversions make multiple overloads viable.
- Overloading on pointer vs. array types can be surprising due to array-to-pointer decay.

### Annotated Code Examples

**Example 1: Overloading on parameter type**

```cpp
#include <iostream>

void print(int x)    { std::cout << "int: " << x << '\n'; }
void print(double x) { std::cout << "double: " << x << '\n'; }
void print(const char* s) { std::cout << "string: " << s << '\n'; }

int main() {
    print(42);
    print(3.14);
    print("hello");
}
```

**Expected Output**
```
int: 42
double: 3.14
string: hello
```

**Why this output?**  
Each call matches a different overload based on the argument type.

**Example 2: Overloading on number of parameters**

```cpp
#include <iostream>

int sum(int a) { return a; }
int sum(int a, int b) { return a + b; }
int sum(int a, int b, int c) { return a + b + c; }

int main() {
    std::cout << sum(1) << '\n';
    std::cout << sum(1, 2) << '\n';
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
Different numbers of arguments select different overloads.

### Real-World Cases

- **Standard library** – `std::abs` overloads for `int`, `long`, `float`, `double`.
- **String handling** – `std::to_string` overloads for numeric types.
- **Container access** – `operator[]` overloads for `const` and non-`const`.

---

## 13. Function Templates

### Definitions

**Core Definition**
A function template is a blueprint for generating functions that can operate on different types.

**Technical Definition**
A function template is declared with the `template` keyword followed by a template parameter list and a function declaration/definition. The compiler generates concrete functions (specializations) when the template is instantiated with specific types.

**Beginner-Friendly Explanation**
A function template lets you write a function once and use it with many different types, without rewriting the code for each type.

### Purposes

- To enable generic programming.
- To avoid code duplication across types.
- To provide type-safe, reusable algorithms.
- To support compile-time polymorphism.

### Syntax Rules and Structure

**General Syntax**
```cpp
template <typename T>
return_type function_name(T parameter) {
    // body
}
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `template <typename T>` | Template parameter list (type parameter). |
| `T` | A placeholder for a type. |
| Function | The templated function. |

**Syntax Rules**
- Template parameters can be type parameters (`typename T`, `class T`) or non-type parameters (`int N`).
- Templates can be overloaded.
- Template argument deduction allows calling without explicit template arguments.
- Explicit specialization and partial specialization are allowed for function templates (full specialization only for function templates).

**Constraints and Limitations**
- Template definitions usually must be visible at the point of instantiation (often placed in headers).
- Error messages can be verbose.
- Templates can increase compile times and code size.

### Annotated Code Examples

**Example 1: Simple function template**

```cpp
#include <iostream>

template <typename T>
T max(T a, T b) {
    return (a > b) ? a : b;
}

int main() {
    std::cout << max(3, 7) << '\n';       // T = int
    std::cout << max(3.14, 2.71) << '\n'; // T = double
}
```

**Expected Output**
```
7
3.14
```

**Why this output?**  
The compiler instantiates `max<int>` and `max<double>` based on the argument types.

**Example 2: Template with multiple parameters**

```cpp
#include <iostream>

template <typename T, typename U>
auto add(T a, U b) -> decltype(a + b) {
    return a + b;
}

int main() {
    std::cout << add(2, 3.5) << '\n';   // int + double -> double
}
```

**Expected Output**
```
5.5
```

**Why this output?**  
The return type is deduced via `decltype(a + b)`, and the result is `5.5`.

### Real-World Cases

- **STL algorithms** – `std::sort`, `std::find`, `std::transform`.
- **Mathematical libraries** – generic numeric operations.
- **Container adaptors** – `std::vector<T>`, `std::map<K,V>`.

---

## 14. Default Arguments

### Definitions

**Core Definition**
Default arguments allow a function to be called without providing one or more trailing arguments; the omitted arguments are filled with predefined default values.

**Technical Definition**
Default arguments are specified in the parameter list using `= initializer` syntax. They are used in place of missing trailing arguments during a function call. After a parameter with a default argument, all subsequent parameters must also have defaults (or be a parameter pack).

**Beginner-Friendly Explanation**
You can give a function “backup values” for some of its parameters. If you don’t supply those arguments, the function uses the backups.

### Purposes

- To simplify function calls by omitting commonly used values.
- To provide backward-compatible interfaces.
- To reduce the need for multiple overloads.
- To improve code readability.

### Syntax Rules and Structure

**General Syntax**
```cpp
return_type function_name(type1 param1 = value1, type2 param2 = value2);
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `param = value` | Default argument specification. |

**Syntax Rules**
- Default arguments must be trailing.
- A parameter with a default cannot be followed by a parameter without a default (except a parameter pack).
- Default arguments are not part of the function signature.
- Default arguments can be added in later declarations in the same scope.

**Constraints and Limitations**
- Default arguments are not allowed in pointer-to-function declarations, references to functions, or `typedef`s.
- Default arguments are evaluated at the call site each time they are used.
- Non-static class members cannot be used in default arguments (except in limited contexts).

### Annotated Code Examples

**Example 1: Basic default argument**

```cpp
#include <iostream>

void point(int x = 3, int y = 4) {
    std::cout << '(' << x << ", " << y << ")\n";
}

int main() {
    point();        // uses both defaults
    point(1);       // x=1, y=4
    point(1, 2);    // both provided
}
```

**Expected Output**
```
(3, 4)
(1, 4)
(1, 2)
```

**Why this output?**  
Omitted trailing arguments are replaced by defaults.

**Example 2: Default argument in a class**

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
    g.greet();
    g.greet("Alice");
}
```

**Expected Output**
```
Hello, Guest!
Hello, Alice!
```

**Why this output?**  
The member function uses the default when no argument is passed.

### Real-World Cases

- **Standard library** – `std::string` constructors with default allocators.
- **GUI APIs** – functions with default window flags or colors.
- **Configuration functions** – default timeouts, retry counts.

---

## 15. Inline Functions

### Definitions

**Core Definition**
An inline function is a function whose definition is suggested to be substituted at the call site to avoid function-call overhead.

**Technical Definition**
The `inline` specifier, when applied to a function, allows the function to be defined in multiple translation units (provided all definitions are identical) and serves as a hint to the compiler to perform inline expansion. Functions defined inside a class definition are implicitly inline.

**Beginner-Friendly Explanation**
An inline function is a function that the compiler may “paste” directly into the places where it is called, instead of making a separate function call. This can make the program faster but may increase code size.

### Purposes

- To reduce function-call overhead for small, frequently called functions.
- To allow functions to be defined in headers without violating the One Definition Rule.
- To enable compile-time optimization.
- To provide type-safe alternatives to macros.

### Syntax Rules and Structure

**General Syntax**
```cpp
inline return_type function_name(parameters) {
    // body
}
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `inline` | Specifier indicating inline function. |

**Syntax Rules**
- The `inline` specifier must be present in every declaration/definition in every translation unit.
- An inline function can be defined in multiple translation units if all definitions are identical.
- Functions defined inside a class body are implicitly inline.
- `constexpr` and `consteval` functions are implicitly inline.

**Constraints and Limitations**
- Inline substitution is a hint, not a guarantee; compilers may ignore it.
- Taking the address of an inline function requires the function to have a unique address across translation units (guaranteed by the standard).
- The `inline` specifier cannot be used on block-scope function declarations.

### Annotated Code Examples

**Example 1: Inline function in a header**

```cpp
// math_utils.h
#pragma once
inline int square(int x) {
    return x * x;
}
```

```cpp
// main.cpp
#include "math_utils.h"
#include <iostream>

int main() {
    std::cout << square(7) << '\n';
}
```

**Expected Output**
```
49
```

**Why this output?**  
The inline function is defined in the header and can be included in multiple translation units without linker errors.

**Example 2: Implicitly inline member function**

```cpp
#include <iostream>

class Calculator {
public:
    int add(int a, int b) {   // implicitly inline
        return a + b;
    }
};

int main() {
    Calculator c;
    std::cout << c.add(3, 4) << '\n';
}
```

**Expected Output**
```
7
```

**Why this output?**  
The member function defined inside the class body is implicitly inline.

### Real-World Cases

- **Header-only libraries** – inline functions allow definitions in headers.
- **Small utility functions** – getters, setters, trivial arithmetic.
- **Template functions** – often defined in headers and implicitly inline.

---

## 16. `constexpr` Functions

### Definitions

**Core Definition**
A `constexpr` function is a function that can be evaluated at compile time when used in a constant expression, while also being callable at runtime.

**Technical Definition**
The `constexpr` specifier declares that it is possible to evaluate the function at compile time. A `constexpr` function is implicitly inline. If any declaration of a function has `constexpr`, every declaration must contain it.

**Beginner-Friendly Explanation**
A `constexpr` function can be computed by the compiler during compilation if you give it constant inputs, but it can also run normally at runtime.

### Purposes

- To enable compile-time computation.
- To improve performance by moving work to compile time.
- To allow functions to be used in constant expressions (e.g., array sizes, template arguments).
- To enforce immutability and purity where possible.

### Syntax Rules and Structure

**General Syntax**
```cpp
constexpr return_type function_name(parameters) {
    // body
}
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `constexpr` | Specifier enabling compile-time evaluation. |

**Syntax Rules**
- The function body must be suitable for constant evaluation (C++14 relaxed many restrictions).
- At least one set of arguments must produce a constant expression for the function to be usable in constant contexts.
- `constexpr` functions are implicitly inline.
- `constexpr` can be applied to member functions, constructors, and destructors (since C++20).

**Constraints and Limitations**
- Before C++14, `constexpr` functions were restricted to a single `return` statement (C++11).
- C++23 relaxes restrictions further (e.g., non-literal variables allowed).
- A `constexpr` function that cannot be evaluated at compile time for any arguments may still compile but is not usable in constant expressions.

### Annotated Code Examples

**Example 1: Compile-time square**

```cpp
#include <iostream>

constexpr int square(int x) {
    return x * x;
}

int main() {
    constexpr int result = square(5);   // computed at compile time
    std::cout << result << '\n';
    std::cout << square(6) << '\n';     // may be runtime or compile time
}
```

**Expected Output**
```
25
36
```

**Why this output?**  
`square(5)` is a constant expression, so `result` is initialized at compile time. `square(6)` is used at runtime.

**Example 2: `constexpr` with `if` (C++14)**

```cpp
#include <iostream>

constexpr int factorial(int n) {
    int result = 1;
    for (int i = 2; i <= n; ++i) {
        result *= i;
    }
    return result;
}

int main() {
    constexpr int fact5 = factorial(5);   // compile-time
    std::cout << fact5 << '\n';
}
```

**Expected Output**
```
120
```

**Why this output?**  
The loop is allowed in C++14 `constexpr` functions, so `factorial(5)` is evaluated at compile time.

### Real-World Cases

- **Array bounds** – `constexpr` functions compute sizes.
- **Template arguments** – constant expressions as template parameters.
- **Compile-time validation** – `static_assert` with `constexpr` functions.

---

## 17. `consteval` Functions

### Definitions

**Core Definition**
A `consteval` function is an *immediate function*: every call to it must produce a compile-time constant.

**Technical Definition**
The `consteval` specifier (C++20) declares a function or function template to be an immediate function. Every potentially-evaluated call must directly or indirectly produce a constant expression. `consteval` implies `inline` and is a subset of `constexpr`.

**Beginner-Friendly Explanation**
A `consteval` function **must** be evaluated at compile time — you cannot call it at runtime. It is stricter than `constexpr`.

### Purposes

- To enforce compile-time evaluation.
- To guarantee that certain computations are always done at compile time.
- To prevent runtime overhead.
- To catch errors at compile time.

### Syntax Rules and Structure

**General Syntax**
```cpp
consteval return_type function_name(parameters) {
    // body
}
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `consteval` | Specifier for immediate functions. |

**Syntax Rules**
- Every call must produce a constant expression.
- `consteval` cannot be combined with `constexpr`.
- All redeclarations must also be `consteval`.
- Cannot be applied to destructors, allocation functions, or deallocation functions.

**Constraints and Limitations**
- Calling a `consteval` function with non-constant arguments is a compile-time error.
- Pointers/references to immediate functions cannot escape constant expression evaluation.
- Since C++23, `if consteval` allows branching based on evaluation context.

### Annotated Code Examples

**Example 1: Compile-time-only function**

```cpp
#include <iostream>

consteval int square(int n) {
    return n * n;
}

int main() {
    constexpr int r = square(5);   // OK: compile-time
    std::cout << r << '\n';
    // int x = 5;
    // int y = square(x);          // error: x is not a constant
}
```

**Expected Output**
```
25
```

**Why this output?**  
`square(5)` is a constant expression, so it is valid. A call with a runtime variable would be rejected.

**Example 2: Nested `consteval` calls**

```cpp
consteval int sqr(int n) { return n * n; }

consteval int sqrsqr(int n) {
    return sqr(sqr(n));
}

constexpr int r = sqrsqr(2);   // OK: 16
```

**Expected Output**  
(Compile-time result `16`, usable as a constant expression.)

**Why this output?**  
`sqrsqr(2)` expands to `sqr(sqr(2))`, all evaluated at compile time.

### Real-World Cases

- **Compile-time validation** – ensuring constants are computed at compile time.
- **Metaprogramming** – generating compile-time constants.
- **Embedded systems** – eliminating runtime computation.

---

## 18. `constinit`

### Definitions

**Core Definition**
`constinit` (C++20) asserts that a variable has *static initialization* (zero initialization and constant initialization); otherwise the program is ill-formed.

**Technical Definition**
The `constinit` specifier declares a variable with static or thread storage duration and requires that its initializing declaration be a constant initialization. It does **not** imply `const`; the variable can be modified at runtime.

**Beginner-Friendly Explanation**
`constinit` ensures that a variable is initialized at compile time, avoiding runtime initialization order problems, but the variable can still change later.

### Purposes

- To guarantee compile-time initialization of static/thread-local variables.
- To avoid static initialization order fiasco.
- To ensure no dynamic initialization overhead.
- To allow mutable variables with constant initialization.

### Syntax Rules and Structure

**General Syntax**
```cpp
constinit type variable = constant_expression;
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `constinit` | Specifier for static initialization. |
| `type` | Type of the variable. |
| `= constant_expression` | Constant initializer required. |

**Syntax Rules**
- `constinit` can only be applied to variables with static or thread storage duration.
- The initializing declaration must have `constinit`.
- `constinit` cannot be combined with `constexpr`.
- For references, `constinit` is equivalent to `constexpr`.

**Constraints and Limitations**
- Dynamic initialization (even if performed as static initialization) makes the program ill-formed.
- Not all types can be `constinit` if they lack `constexpr` constructors.

### Annotated Code Examples

**Example 1: `constinit` with constant initialization**

```cpp
#include <cassert>

constexpr int square(int i) { return i * i; }

constinit int sq = square(2);   // OK: constant initialization

int main() {
    assert(sq == 4);
    sq = 10;                    // allowed: constinit does not imply const
    assert(sq == 10);
}
```

**Expected Output**  
(No output; assertions pass.)

**Why this output?**  
`sq` is initialized at compile time via `square(2)`, but it remains mutable.

**Example 2: Error with dynamic initialization**

```cpp
int f() { return 42; }

// constinit int x = f();   // error: f() is not a constant expression
```

**Expected Output**  
(Compilation error)

**Why this error?**  
`f()` is not `constexpr`, so its result is not a constant expression, violating `constinit`.

### Real-World Cases

- **Global configuration** – constants initialized at compile time but mutable during runtime.
- **Thread-local storage** – avoiding guard variables for thread-local initialization.
- **Embedded systems** – deterministic startup.

---

## 19. `noexcept`

### Definitions

**Core Definition**
`noexcept` is a function specifier that indicates whether a function may throw exceptions.

**Technical Definition**
The `noexcept` specification declares that a function is non-throwing if its operand evaluates to `true` (or if written as `noexcept` without an operand). It is not part of the function type (until C++17) but is part of the function type since C++17.

**Beginner-Friendly Explanation**
`noexcept` tells the compiler (and programmers) that a function promises not to throw exceptions. This can enable optimizations and is important for exception safety.

### Purposes

- To indicate that a function does not throw exceptions.
- To enable compiler optimizations (e.g., move semantics).
- To enforce exception safety contracts.
- To improve performance by avoiding exception-handling overhead.

### Syntax Rules and Structure

**General Syntax**
```cpp
return_type function_name(parameters) noexcept;
return_type function_name(parameters) noexcept(expression);
```

**Component Breakdown**

| Form | Meaning |
|------|---------|
| `noexcept` | Same as `noexcept(true)`. |
| `noexcept(expr)` | Non-throwing if `expr` is `true`. |

**Syntax Rules**
- `noexcept` is part of the function type since C++17.
- It can appear in function declarations, definitions, and lambda declarators.
- It cannot appear in `typedef` or type alias declarations.
- `throw()` is deprecated (C++17) and removed (C++20).

**Constraints and Limitations**
- If a `noexcept` function throws, `std::terminate` is called.
- Overloading cannot be based solely on `noexcept` (since C++17).
- Destructors are implicitly `noexcept(true)` unless specified otherwise.

### Annotated Code Examples

**Example 1: `noexcept` function**

```cpp
#include <iostream>

void safe_function() noexcept {
    std::cout << "No exceptions here\n";
}

int main() {
    safe_function();
}
```

**Expected Output**
```
No exceptions here
```

**Why this output?**  
The function is declared `noexcept`, promising not to throw.

**Example 2: Conditional `noexcept`**

```cpp
#include <iostream>

void may_throw() noexcept(false) {
    throw std::runtime_error("error");
}

int main() {
    try {
        may_throw();
    } catch (const std::exception& e) {
        std::cout << "Caught: " << e.what() << '\n';
    }
}
```

**Expected Output**
```
Caught: error
```

**Why this output?**  
`noexcept(false)` allows the function to throw, and the exception is caught in `main`.

### Real-World Cases

- **Move constructors** – marked `noexcept` to enable `std::vector` reallocation optimizations.
- **Destructors** – implicitly `noexcept` to ensure safe stack unwinding.
- **Swap functions** – `noexcept` swap is required for many algorithms.

---

## 20. Function Best Practices

### Definitions

**Core Definition**
Function best practices are guidelines for writing functions that are correct, readable, maintainable, efficient, and safe.

**Technical Definition**
These practices are codified in resources such as the C++ Core Guidelines, which provide rules like “Package meaningful operations as carefully named functions” (F.1), “A function should perform a single logical operation” (F.2), “Keep functions short and simple” (F.3), and “If a function may have to be evaluated at compile time, declare it `constexpr`” (F.4).

**Beginner-Friendly Explanation**
Best practices are like “good habits” for writing functions: give them clear names, keep them focused, use the right features, and avoid common pitfalls.

### Purposes

- To improve code readability and maintainability.
- To reduce bugs and undefined behavior.
- To enhance performance where appropriate.
- To make interfaces intuitive and safe.
- To facilitate testing and debugging.

### Key Guidelines

| Guideline | Summary |
|-----------|---------|
| **F.1** | Package meaningful operations as carefully named functions. |
| **F.2** | A function should perform a single logical operation. |
| **F.3** | Keep functions short and simple. |
| **F.4** | If a function may have to be evaluated at compile time, declare it `constexpr`. |
| **F.6** | If your function may not throw, declare it `noexcept`. |
| **F.8** | Prefer pure functions. |
| **Parameter passing** | Pass cheap-to-copy types by value, others by `const&`; pass in-out parameters by non-const reference. |
| **Avoid output parameters** | Prefer returning values over using output parameters. |
| **Use `const`** | By default, make member functions `const`. |
| **Use `constexpr`** | Declare functions `constexpr` when compile-time evaluation is possible. |

### Annotated Code Examples

**Example 1: Single logical operation**

```cpp
#include <iostream>

// Good: single logical operation
double calculate_area(double radius) {
    constexpr double pi = 3.141592653589793;
    return pi * radius * radius;
}

int main() {
    std::cout << calculate_area(2.0) << '\n';
}
```

**Expected Output**
```
12.5664
```

**Why this output?**  
The function performs one clear task: computing the area of a circle.

**Example 2: `constexpr` and `noexcept`**

```cpp
#include <iostream>

constexpr int square(int x) noexcept {
    return x * x;
}

int main() {
    constexpr int r = square(5);
    std::cout << r << '\n';
}
```

**Expected Output**
```
25
```

**Why this output?**  
`square` is both `constexpr` (usable at compile time) and `noexcept` (promises not to throw).

### Real-World Cases

- **Code reviews** – best practices are used as checklists.
- **Static analysis** – tools like clang-tidy enforce these guidelines.
- **Library design** – interfaces are designed with these rules in mind.

---

## References

- Functions – https://en.cppreference.com/cpp/language/functions
- Function declaration – https://en.cppreference.com/w/cpp/language/function
- Default arguments – https://en.cppreference.com/w/cpp/language/default_arguments
- Function overloading – https://en.cppreference.com/w/cpp/language/function#Function_overloading
- Inline specifier – https://en.cppreference.com/cpp/language/inline
- `constexpr` specifier – https://en.cppreference.com/cpp/language/constexpr
- `consteval` specifier – https://en.cppreference.com/cpp/language/consteval
- `constinit` specifier – https://en.cppreference.com/cpp/language/constinit
- `noexcept` specifier – https://en.cppreference.com/cpp/language/noexcept_spec
- Function templates – https://en.cppreference.com/cpp/language/function_template (see also https://en.cppreference.com/cpp/language/templates)
- C++ Core Guidelines – Functions – https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#S-functions
- C++ Working Draft (N4950) – https://eel.is/c++draft/ (for standard references)
- C++ Core Guidelines – Parameter passing – https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rf-conventional

---

This cheat sheet provides a comprehensive, structured reference for C++ function fundamentals, covering definitions, syntax, examples, best practices, and authoritative references. Each section follows a uniform template to facilitate learning and review.