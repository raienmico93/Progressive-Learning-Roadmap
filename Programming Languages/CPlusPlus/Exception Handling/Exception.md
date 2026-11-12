# C++ Exception Fundamentals — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
C++ exception handling is a language mechanism that transfers control and information from the point where an error is detected (the throw site) to a handler (the catch site) associated with a previously entered scope, transferring control up the call stack.

**Technical Definition**
Exception handling in C++ provides a way of transferring control and information from some point in the execution of a program to a handler associated with a point previously passed by the execution (in other words, exception handling transfers control up the call stack). An exception can be thrown by a `throw`-expression, `dynamic_cast`, `typeid`, `new`-expression, allocation functions, and any standard library function specified to throw exceptions. For an exception to be caught, the throw-expression must be inside a `try`-block or inside a function called from a `try`-block, and there must be a `catch` clause that matches the type of the exception object.

**Beginner-Friendly Explanation**
Exception handling is like a safety net for your program. When something goes wrong deep inside a function, instead of crashing or returning error codes through every caller, the program “throws” an exception. The runtime then searches for a “catch” block that knows how to handle that kind of problem, jumping directly to it while cleaning up any temporary objects along the way.

### Key Characteristics

- **Separation of error detection and handling** – the code that detects an error does not need to know how it will be handled.
- **Stack unwinding** – automatic destruction of local objects during propagation.
- **Type-based matching** – handlers are selected by the type of the thrown object.
- **Exception safety guarantees** – functions can promise specific levels of safety (nothrow, strong, basic).
- **Zero-cost when not thrown** – modern implementations typically impose no runtime overhead when no exception is thrown.
- **Hierarchical handling** – exceptions are organised in a class hierarchy rooted at `std::exception`.

### Prerequisites

- Basic C++ syntax (functions, classes, constructors/destructors).
- Understanding of stack frames and automatic variables.
- Familiarity with inheritance and polymorphism (for catching by base reference).
- Knowledge of RAII and destructor semantics.
- Awareness of `noexcept` and exception specifications.

### Related Programming Areas

- **RAII** – destructors run during stack unwinding, releasing resources.
- **Smart pointers** – automatic cleanup during exception propagation.
- **Standard library** – containers and algorithms throw exceptions on certain errors.
- **Concurrency** – exceptions in threads must be caught within the thread or stored/transferred.
- **Error handling design** – choosing between exceptions, error codes, and `std::optional`/`std::expected`.

### Core Concepts / Features

1. Exceptions
2. `throw`
3. `try`
4. `catch`
5. Exception hierarchy
6. `std::exception`
7. Standard exceptions
8. Custom exceptions
9. Exception propagation
10. Stack unwinding
11. Exception best practices

---

## 1. Exceptions

### Definitions

**Core Definition**
An exception is an object that represents an error or unusual condition, created and propagated by the runtime when a `throw` expression is executed.

**Technical Definition**
When a `throw` expression is evaluated, an exception object is created from the operand (involving a copy constructor), and the runtime begins a search for a matching handler in dynamically enclosing `try` blocks. If no matching handler is found, `std::terminate` is called.

**Beginner-Friendly Explanation**
An exception is a special “error message” object that gets created when something goes wrong. The program then looks for a piece of code that is prepared to deal with that kind of message.

### Purposes

- To signal failures to establish invariants, meet postconditions, or meet caller preconditions.
- To separate error detection from error handling.
- To propagate errors across multiple function call levels without error-code plumbing.
- To enable resource cleanup via stack unwinding.

### Syntax Rules and Structure

**General Syntax**
```cpp
throw expression;   // creates an exception object
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `expression` | The operand; its type determines the exception type. A copy is made. |

**Syntax Rules**
- `throw;` (without operand) rethrows the currently handled exception.
- Throwing `0` does not match pointer handlers; use `nullptr` for null pointer exceptions.
- Exceptions can also be thrown implicitly by `dynamic_cast`, `typeid`, `new`, and standard library functions.

**Constraints and Limitations**
- Destructors should generally not throw (they are implicitly `noexcept`).
- Throwing from a `noexcept` function calls `std::terminate`.
- Exception objects are copied; they should be cheap to copy.

### Annotated Code Example

```cpp
#include <iostream>
#include <stdexcept>

void might_fail(int x) {
    if (x < 0)
        throw std::invalid_argument("negative value");
}

int main() {
    try {
        might_fail(-1);
    } catch (const std::exception& e) {
        std::cout << "Caught: " << e.what() << '\n';
    }
}
```

**Expected Output**
```
Caught: negative value
```

**Why this output?**  
`might_fail` throws a `std::invalid_argument`; the `catch` clause matching `const std::exception&` handles it and prints the message.

### Real-World Cases

- **Library APIs** – signaling invalid arguments or resource exhaustion.
- **Parsers** – reporting syntax errors.
- **Network code** – signaling connection failures.

---

## 2. `throw`

### Definitions

**Core Definition**
`throw` is a keyword that initiates exception propagation by creating an exception object from its operand.

**Technical Definition**
A throw-expression creates a temporary exception object from its operand (copy-initialization), transfers control to the matching handler, and unwinds the stack. The operand can be any complete type; the thrown object’s type is the “exception type” used for handler matching.

**Beginner-Friendly Explanation**
`throw` is the command that “sends” an exception. You give it a value, and the program starts looking for a `catch` block that can handle that type.

### Purposes

- To create and raise an exception object.
- To signal an error condition explicitly.
- To rethrow the current exception (`throw;`).
- To propagate exceptions from lower-level code.

### Syntax Rules and Structure

**General Syntax**
```cpp
throw expression;    // throw a new exception
throw;               // rethrow the current exception
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `expression` | Any expression; its type becomes the exception type. |
| `throw;` | Only valid inside a catch handler or a function called from one. |

**Syntax Rules**
- The operand is copied into the exception object; throwing by value is recommended, catching by reference.
- Rethrowing with `throw;` preserves the original exception type.
- `throw` with no active exception calls `std::terminate`.

**Constraints and Limitations**
- Do not throw from destructors during stack unwinding.
- Avoid throwing raw pointers; throw objects by value.
- Throwing in a `noexcept` function terminates.

### Annotated Code Example

```cpp
#include <iostream>
#include <stdexcept>

void check(int x) {
    if (x == 0)
        throw std::runtime_error("zero not allowed");
}

int main() {
    try {
        check(0);
    } catch (const std::exception& e) {
        std::cout << "Error: " << e.what() << '\n';
        // rethrow to outer handler
        try {
            throw;                     // rethrow current exception
        } catch (const std::runtime_error& e2) {
            std::cout << "Rethrown: " << e2.what() << '\n';
        }
    }
}
```

**Expected Output**
```
Error: zero not allowed
Rethrown: zero not allowed
```

**Why this output?**  
`throw;` rethrows the active exception, which is then caught by the inner `runtime_error` handler.

### Real-World Cases

- **Validation** – throwing on invalid input.
- **Resource allocation** – throwing `std::bad_alloc` on failure.
- **Rethrowing** – adding context while preserving the original type.

---

## 3. `try`

### Definitions

**Core Definition**
`try` begins a block of code (the *guarded section*) whose exceptions are handled by the associated `catch` clauses.

**Technical Definition**
A try-block is a statement that associates a compound statement (the guarded section) with a handler sequence. If an exception is thrown inside the guarded section or in any function called directly or indirectly from it, the handlers are examined in order of appearance.

**Beginner-Friendly Explanation**
`try` is like saying “attempt this code, and if anything goes wrong, I have a plan.” It wraps the risky code.

### Purposes

- To define a region where exceptions will be caught.
- To associate handlers with potentially throwing code.
- To enable stack unwinding from the throw site to the handler.

### Syntax Rules and Structure

**General Syntax**
```cpp
try {
    // guarded code
} catch (Type1 e1) {
    // handler 1
} catch (Type2 e2) {
    // handler 2
} catch (...) {
    // catch-all
}
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `try { ... }` | The guarded section. |
| `catch (Type e)` | A handler for exceptions of type `Type`. |
| `catch (...)` | A catch-all handler. |

**Syntax Rules**
- At least one `catch` clause must follow a `try` block.
- Handlers are tried in order; the first match wins.
- A function-try-block can wrap a constructor/destructor body.

**Constraints and Limitations**
- `try` blocks do not catch exceptions thrown during stack unwinding of the same try block (destructor exceptions).
- Overuse of `try`/`catch` can obscure control flow.

### Annotated Code Example

```cpp
#include <iostream>
#include <stdexcept>

int main() {
    try {
        throw std::out_of_range("index too large");
    } catch (const std::logic_error& e) {
        std::cout << "Logic error: " << e.what() << '\n';
    } catch (const std::exception& e) {
        std::cout << "Other: " << e.what() << '\n';
    }
}
```

**Expected Output**
```
Logic error: index too large
```

**Why this output?**  
`std::out_of_range` derives from `std::logic_error`, so the more derived handler matches first.

### Real-World Cases

- **Top-level main** – catching all exceptions to log and exit gracefully.
- **API boundaries** – translating exceptions to error codes.
- **Resource management** – ensuring cleanup on failure.

---

## 4. `catch`

### Definitions

**Core Definition**
A `catch` clause is a handler that receives an exception whose type matches its parameter.

**Technical Definition**
Each `catch` clause specifies a parameter type. The handler matches if the exception type is the same as, or derived from, the parameter type (with adjustments for references and cv-qualification). Handlers are examined in order; the first matching handler is selected.

**Beginner-Friendly Explanation**
`catch` is the “plan” for what to do when a particular type of exception is thrown. You can have different plans for different error types.

### Purposes

- To handle specific exception types.
- To provide a catch-all fallback (`catch (...)`).
- To catch by reference and avoid slicing.
- To rethrow or translate exceptions.

### Syntax Rules and Structure

**General Syntax**
```cpp
catch (Type& e) { ... }
catch (const Type& e) { ... }
catch (Type e) { ... }          // by value (avoid)
catch (...) { ... }              // catch-all
```

**Component Breakdown**

| Form | Behaviour |
|------|-----------|
| `catch (T& e)` | Binds to the exception object; can modify it. |
| `catch (const T& e)` | Binds to the exception object without copying; preferred. |
| `catch (T e)` | Copies the exception object (risk of slicing). |
| `catch (...)` | Catches any exception. |

**Syntax Rules**
- Catch by reference (usually `const`) to avoid unnecessary copying and object slicing.
- Order handlers from most derived to least derived.
- `catch (...)` should be last.

**Constraints and Limitations**
- Catching by value slices derived objects.
- A handler that does not rethrow consumes the exception.

### Annotated Code Example

```cpp
#include <iostream>
#include <stdexcept>

int main() {
    try {
        throw std::runtime_error("failure");
    } catch (const std::runtime_error& e) {
        std::cout << "Runtime error: " << e.what() << '\n';
    } catch (...) {
        std::cout << "Unknown error\n";
    }
}
```

**Expected Output**
```
Runtime error: failure
```

**Why this output?**  
The `runtime_error` handler matches first; the catch-all would only be used for other types.

### Real-World Cases

- **Layered error handling** – catching specific domain errors first.
- **Fallback** – catching `...` at the top level to log and exit.
- **Translation** – catching a library exception and rethrowing a custom type.

---

## 5. Exception Hierarchy

### Definitions

**Core Definition**
The exception hierarchy is the inheritance tree of standard and user-defined exception classes, rooted at `std::exception`.

**Technical Definition**
All exceptions generated by the standard library inherit from `std::exception`. The hierarchy is organised into two main branches: `std::logic_error` (errors detectable before execution, e.g., precondition violations) and `std::runtime_error` (errors detectable only during execution). Other direct derivatives include `std::bad_alloc`, `std::bad_cast`, `std::bad_typeid`, `std::bad_exception`, and `std::ios_base::failure`.

**Beginner-Friendly Explanation**
The exception hierarchy is a family tree of error types. `std::exception` is the great-grandparent; `logic_error` and `runtime_error` are the two main branches. By catching a base class, you can handle a whole family of related errors with one handler.

### Purposes

- To organise error types into meaningful categories.
- To enable polymorphic catching (catch a base class to handle derived types).
- To provide a consistent interface (`what()`).
- To support extensibility via custom derived exceptions.

### Hierarchy Overview

```
std::exception
├── std::logic_error
│   ├── std::invalid_argument
│   ├── std::domain_error
│   ├── std::length_error
│   └── std::out_of_range
├── std::runtime_error
│   ├── std::range_error
│   ├── std::overflow_error
│   └── std::underflow_error
├── std::bad_alloc
├── std::bad_cast
├── std::bad_typeid
├── std::bad_exception
└── std::ios_base::failure
```

*Source: standard exception hierarchy tables.*

### Annotated Code Example

```cpp
#include <iostream>
#include <stdexcept>

int main() {
    try {
        throw std::out_of_range("out of range");
    } catch (const std::logic_error& e) {
        std::cout << "Logic: " << e.what() << '\n';
    }
}
```

**Expected Output**
```
Logic: out of range
```

**Why this output?**  
`std::out_of_range` derives from `std::logic_error`; catching the base class handles it polymorphically.

### Real-World Cases

- **Library design** – defining a root exception for a library’s errors.
- **Application layers** – catching `logic_error` vs `runtime_error` for different recovery strategies.

---

## 6. `std::exception`

### Definitions

**Core Definition**
`std::exception` is the base class for all exceptions thrown by the standard library, providing a consistent interface.

**Technical Definition**
Defined in `<exception>`, `std::exception` provides a virtual destructor and a virtual `what()` member function returning an explanatory string. All member functions are `constexpr` since C++26. Every standard library exception class `T` derived from `std::exception` must have public copy constructor and copy assignment that preserve the `what()` message.

**Beginner-Friendly Explanation**
`std::exception` is the “mother of all standard exceptions.” It guarantees that every standard exception has a `what()` message. Catching `const std::exception&` catches all standard exceptions.

### Purposes

- To provide a common base for all standard exceptions.
- To enable catching any standard exception with one handler.
- To expose a human-readable error message via `what()`.
- To support polymorphic destruction.

### Syntax Rules and Structure

**General Syntax**
```cpp
#include <exception>
try { ... }
catch (const std::exception& e) {
    std::cerr << e.what() << '\n';
}
```

**Component Breakdown**

| Member | Description |
|--------|-------------|
| `virtual ~exception()` | Virtual destructor. |
| `virtual const char* what() const noexcept` | Returns an explanatory string. |

**Syntax Rules**
- `what()` is `virtual` and `noexcept`; overrides must be `noexcept`.
- Always catch by `const` reference to avoid slicing.

**Constraints and Limitations**
- `what()` returns a pointer to a string with lifetime tied to the exception object.
- Throwing from `what()` is forbidden.

### Annotated Code Example

```cpp
#include <exception>
#include <iostream>

int main() {
    try {
        throw std::exception();
    } catch (const std::exception& e) {
        std::cout << "Caught: " << e.what() << '\n';
    }
}
```

**Expected Output** (message is implementation-defined)
```
Caught: std::exception
```

**Why this output?**  
`what()` returns an implementation-defined description of the exception.

### Real-World Cases

- **Top-level handlers** – catching all standard exceptions.
- **Logging** – extracting `what()` for error messages.
- **Custom hierarchy roots** – deriving custom exceptions from `std::exception`.

---

## 7. Standard Exceptions

### Definitions

**Core Definition**
Standard exceptions are the predefined exception classes in the C++ Standard Library, primarily declared in `<stdexcept>`, `<new>`, `<typeinfo>`, and `<exception>`.

**Technical Definition**
The standard library provides exception classes for common error conditions: `logic_error` (and its derivatives `invalid_argument`, `domain_error`, `length_error`, `out_of_range`) and `runtime_error` (and its derivatives `range_error`, `overflow_error`, `underflow_error`). Additional standard exceptions include `bad_alloc`, `bad_cast`, `bad_typeid`, `bad_exception`, and `ios_base::failure`.

**Beginner-Friendly Explanation**
Standard exceptions are ready-made error types for common problems: out-of-range access, invalid arguments, memory allocation failure, bad casts, and I/O errors.

### Key Standard Exceptions

| Exception | Header | Meaning |
|-----------|--------|---------|
| `std::logic_error` | `<stdexcept>` | Precondition/invariant violations |
| `std::invalid_argument` | `<stdexcept>` | Invalid function argument |
| `std::domain_error` | `<stdexcept>` | Domain error (math) |
| `std::length_error` | `<stdexcept>` | Exceeds maximum size |
| `std::out_of_range` | `<stdexcept>` | Index out of range |
| `std::runtime_error` | `<stdexcept>` | Errors detectable only at runtime |
| `std::range_error` | `<stdexcept>` | Result out of range |
| `std::overflow_error` | `<stdexcept>` | Arithmetic overflow |
| `std::underflow_error` | `<stdexcept>` | Arithmetic underflow |
| `std::bad_alloc` | `<new>` | Allocation failure |
| `std::bad_cast` | `<typeinfo>` | Failed `dynamic_cast` |
| `std::bad_typeid` | `<typeinfo>` | `typeid` on null polymorphic pointer |
| `std::bad_exception` | `<exception>` | Unexpected exception in old dynamic exception spec |

*Source: standard exception hierarchy tables.*

### Annotated Code Example

```cpp
#include <vector>
#include <iostream>
#include <stdexcept>

int main() {
    std::vector<int> v = {1, 2, 3};
    try {
        v.at(5);                           // throws std::out_of_range
    } catch (const std::out_of_range& e) {
        std::cout << "Out of range: " << e.what() << '\n';
    }
}
```

**Expected Output**
```
Out of range: vector::_M_range_check: __n (which is 5) >= this->size() (which is 3)
```

**Why this output?**  
`vector::at` performs bounds checking and throws `std::out_of_range` on invalid access.

### Real-World Cases

- **Container access** – `at()` throws `out_of_range`.
- **Memory allocation** – `new` throws `bad_alloc`.
- **Type-safe casts** – `dynamic_cast` throws `bad_cast`.

---

## 8. Custom Exceptions

### Definitions

**Core Definition**
A custom exception is a user-defined class that derives from `std::exception` or one of its subclasses, representing a domain-specific error type.

**Technical Definition**
Custom exception hierarchies allow you to define domain-specific error types that integrate seamlessly with the standard `try`/`catch` mechanism. By deriving from `std::exception` (or its subclasses), you get polymorphic catching, meaningful `what()` messages, and the ability to carry structured error data.

**Beginner-Friendly Explanation**
Custom exceptions are your own error types. By deriving from `std::exception`, they work with standard `catch` blocks, so you can catch them either specifically or as a general `std::exception`.

### Purposes

- To represent domain-specific errors.
- To carry structured error data (error codes, context).
- To integrate with the standard exception hierarchy.
- To enable precise, type-based error handling.

### Syntax Rules and Structure

**General Syntax**
```cpp
class MyError : public std::runtime_error {
public:
    explicit MyError(const std::string& msg)
        : std::runtime_error(msg) {}
};
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| Base class | `std::exception` or a subclass (e.g., `std::runtime_error`). |
| Constructor | Forwards message to the base class. |
| `what()` | Inherited from `std::runtime_error` (stores the message). |
| Additional members | Domain-specific data. |

**Syntax Rules**
- Derive from `std::exception` or a subclass for compatibility.
- Store the message via `std::runtime_error` or `std::logic_error` (they manage string lifetime).
- Catch by `const` reference to avoid slicing.
- Make `what()` `noexcept` if overriding.

**Constraints and Limitations**
- Keep exceptions lightweight (they are copied during unwinding).
- Avoid deep hierarchies that are hard to maintain.
- Do not throw from destructors.

### Annotated Code Example

```cpp
#include <iostream>
#include <stdexcept>
#include <string>

class NetworkError : public std::runtime_error {
    int code_;
public:
    NetworkError(const std::string& msg, int code)
        : std::runtime_error(msg), code_(code) {}

    int code() const noexcept { return code_; }
};

int main() {
    try {
        throw NetworkError("connection refused", 111);
    } catch (const NetworkError& e) {
        std::cout << "Network error " << e.code()
                  << ": " << e.what() << '\n';
    } catch (const std::exception& e) {
        std::cout << "Other: " << e.what() << '\n';
    }
}
```

**Expected Output**
```
Network error 111: connection refused
```

**Why this output?**  
The `NetworkError` handler matches first, providing access to the custom `code()` member and the inherited `what()` message.

### Real-World Cases

- **Library error reporting** – custom root exceptions for library-specific failures.
- **Application domains** – database, network, and validation errors.
- **Structured error data** – carrying error codes and context.

---

## 9. Exception Propagation

### Definitions

**Core Definition**
Exception propagation is the process by which a thrown exception travels up the call stack, searching for a matching handler.

**Technical Definition**
When an exception is thrown, the runtime examines catch handlers in the current `try` block, then in dynamically enclosing `try` blocks, continuing up the call stack. If no matching handler is found, `std::terminate` is called. Exceptions do not propagate across threads unless explicitly caught, stored, and rethrown.

**Beginner-Friendly Explanation**
Propagation is the exception “bubbling up” through the functions that called the throwing function. It stops when it finds a `catch` block that matches its type. If it reaches the top without being caught, the program terminates.

### Purposes

- To route errors to the appropriate handler.
- To avoid error-code plumbing through every function.
- To enable centralized error handling at appropriate boundaries.
- To allow translation between error types at layer boundaries.

### Syntax Rules and Structure

**Propagation Model**
```
throw → current function → caller → caller's caller → ... → matching catch
```

**Component Breakdown**

| Stage | Description |
|-------|-------------|
| Throw | Exception object created. |
| Search | Handlers examined in dynamically enclosing try blocks. |
| Unwind | Automatic objects destroyed along the path. |
| Handle | Matching catch executes. |

**Syntax Rules**
- Handlers are examined in reverse order of function calls (most recent first).
- Propagation stops at the first matching handler.
- `throw;` inside a handler resumes propagation after the handler exits.

**Constraints and Limitations**
- No matching handler → `std::terminate`.
- Exceptions do not automatically cross thread boundaries.

### Annotated Code Example

```cpp
#include <iostream>
#include <stdexcept>

void level3() { throw std::runtime_error("deep error"); }
void level2() { level3(); }               // no handler here
void level1() { level2(); }               // no handler here

int main() {
    try {
        level1();
    } catch (const std::exception& e) {
        std::cout << "Caught in main: " << e.what() << '\n';
    }
}
```

**Expected Output**
```
Caught in main: deep error
```

**Why this output?**  
The exception propagates from `level3` through `level2` and `level1` to the handler in `main`.

### Real-World Cases

- **Layered architecture** – exceptions propagate from data layer to UI layer.
- **Plugin systems** – catching at plugin boundaries.
- **Thread pools** – capturing exceptions for later inspection.

---

## 10. Stack Unwinding

### Definitions

**Core Definition**
Stack unwinding is the process of calling destructors for automatic objects constructed on the path from a `try` block to the point where an exception is thrown.

**Technical Definition**
In the C++ exception mechanism, control moves from the `throw` statement to the first `catch` statement that can handle the thrown type. When the `catch` statement is reached, all automatic variables that are in scope between the `throw` and `catch` statements are destroyed in a process known as stack unwinding. Destruction occurs in reverse order of construction.

**Beginner-Friendly Explanation**
Stack unwinding is the runtime “cleaning up” local objects as the exception travels up the stack. Every local variable in the functions between the `throw` and the `catch` gets its destructor called, in reverse order of creation. This is how resources are released safely.

### Purposes

- To guarantee destruction of local objects during exception propagation.
- To release resources (memory, files, locks) via RAII.
- To maintain program invariants during unwinding.
- To ensure deterministic cleanup.

### Syntax Rules and Structure

**Unwinding Sequence**
1. Exception object created.
2. Handler search begins.
3. Once a handler is selected, automatic objects between the `try` block and the throw site are destroyed.
4. Destruction proceeds in reverse order of construction.
5. The handler executes.

**Constraints and Limitations**
- If a destructor throws during unwinding, `std::terminate` is called (destructors are implicitly `noexcept`).
- Objects with incomplete construction are not destroyed (only fully constructed objects).
- Raw pointers are not destroyed; only objects with destructors are cleaned up.

### Annotated Code Example

```cpp
#include <iostream>
#include <stdexcept>

struct Dummy {
    std::string name;
    Dummy(std::string n) : name(std::move(n)) {
        std::cout << "Construct " << name << '\n';
    }
    ~Dummy() {
        std::cout << "Destroy " << name << '\n';
    }
};

void f() {
    Dummy d1("d1");
    Dummy d2("d2");
    throw std::runtime_error("boom");
}

int main() {
    try {
        f();
    } catch (const std::exception& e) {
        std::cout << "Caught: " << e.what() << '\n';
    }
}
```

**Expected Output**
```
Construct d1
Construct d2
Destroy d2
Destroy d1
Caught: boom
```

**Why this output?**  
`d1` and `d2` are constructed in order; during unwinding, they are destroyed in reverse order (`d2` then `d1`) before the handler runs.

### Real-World Cases

- **RAII** – smart pointers and lock guards release resources during unwinding.
- **File handling** – `std::fstream` closes files automatically.
- **Transaction rollback** – destructors undo partial changes.

---

## 11. Exception Best Practices

### Definitions

**Core Definition**
Exception best practices are guidelines for using exceptions correctly, safely, and efficiently, as codified in the C++ Core Guidelines and community consensus.

**Technical Definition**
These practices include throwing by value and catching by reference (E.15), ensuring exception safety guarantees (E.2–E.6), avoiding exceptions in destructors, using `noexcept` appropriately (F.6), and designing exception-safe interfaces. The four levels of exception guarantee recognised by the standard are nothrow, strong, basic, and no guarantee.

**Beginner-Friendly Explanation**
Best practices for exceptions are simple: throw by value, catch by reference, don’t throw from destructors, and make sure your code is “exception safe” (doesn’t leak or break invariants when an exception occurs).

### Key Guidelines

| Guideline | Summary |
|-----------|---------|
| **Throw by value, catch by reference** | Avoid slicing and unnecessary copies (E.15). |
| **Catch most-derived first** | Order handlers from specific to general. |
| **Do not throw from destructors** | Destructors are implicitly `noexcept`; throwing terminates. |
| **Use `noexcept` for non-throwing functions** | Enables optimisations and documents intent (F.6). |
| **Provide at least the basic guarantee** | No leaks, invariants preserved. |
| **Use the strong guarantee where possible** | Commit-or-rollback semantics. |
| **Use RAII for resource management** | Ensures cleanup during unwinding. |
| **Avoid exception specifications (dynamic)** | Use `noexcept` instead. |
| **Do not use exceptions for control flow** | Reserve them for errors. |
| **Keep exception types lightweight** | They are copied during unwinding. |

### Exception Safety Guarantees

| Guarantee | Meaning |
|-----------|---------|
| **Nothrow / nofail** | Function never throws; marked `noexcept`. |
| **Strong** | If an exception is thrown, program state is rolled back to before the call (commit-or-rollback). |
| **Basic** | If an exception is thrown, program is in a valid state; no leaks; invariants intact. |
| **No guarantee** | Undefined state; unacceptable for production code. |

*Source: cppreference exception safety levels, standard guarantee definitions.*

### Annotated Code Example

```cpp
#include <iostream>
#include <stdexcept>
#include <memory>

void process() {
    auto resource = std::make_unique<int>(42);   // RAII
    throw std::runtime_error("failure");
    // resource released automatically during unwinding
}

int main() {
    try {
        process();
    } catch (const std::exception& e) {          // catch by const reference
        std::cout << "Handled: " << e.what() << '\n';
    }
}
```

**Expected Output**
```
Handled: failure
```

**Why this output?**  
`unique_ptr` releases the resource during stack unwinding; the exception is caught by `const std::exception&`, avoiding slicing and copies.

### Real-World Cases

- **Code reviews** – checking for throw-by-value/catch-by-reference.
- **Static analysis** – clang-tidy enforces best practices.
- **Library design** – documenting exception guarantees.

---

## References

- Exceptions — cppreference – https://en.cppreference.com/w/cpp/language/exceptions
- `std::exception` — cppreference – https://en.cppreference.com/w/cpp/error/exception
- Exceptions and Stack Unwinding in C++ — Microsoft Learn – https://learn.microsoft.com/en-us/cpp/cpp/exceptions-and-stack-unwinding-in-cpp
- Standard Exception Hierarchy — Apache C++ Standard Library – https://svn.apache.org/repos/asf/stdcxx/trunk/doc/stdlibref/exception-hierarchy.html
- C++ Core Guidelines — E.15: Throw by value, catch exceptions from a hierarchy by reference – https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Re-throw
- C++ Core Guidelines — F.6: If your function must not throw, declare it `noexcept` – https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rf-noexcept
- SEI CERT C++ — ERR61-CPP: Catch exceptions by lvalue reference – https://wiki.sei.cmu.edu/confluence/display/cplusplus/ERR61-CPP.+Catch+exceptions+by+lvalue+reference
- Use custom exception classes in a hierarchy with `std::exception` base — Dr-Sergey/learn_cpp – https://github.com/Dr-Sergey/learn_cpp/blob/08279578ce23b1cbf2d3394357620600db618eca/11_Error_Handling/Use_custom_exception_classes_in_a_hierarchy_with_stdexception_base.md
- Exception safety — cppreference – https://en.cppreference.com/w/cpp/language/exceptions#Exception_safety
- C++ Working Draft (N4950) — Exception handling – https://eel.is/c++draft/except

---

This cheat sheet provides a comprehensive, structured reference for C++ exception fundamentals, covering exceptions, `throw`, `try`, `catch`, the exception hierarchy, `std::exception`, standard exceptions, custom exceptions, propagation, stack unwinding, and best practices, with annotated code examples, expected outputs, real-world cases, and authoritative references.