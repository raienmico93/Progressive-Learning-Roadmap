# C++ Custom Exceptions — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
A custom exception is a user-defined class that derives from `std::exception` (or one of its standard subclasses) and represents a domain-specific error condition, integrating seamlessly with the standard `try`/`catch` mechanism.

**Technical Definition**
A custom exception class is a class that publicly inherits from `std::exception` or a class derived from it (such as `std::runtime_error` or `std::logic_error`). It typically overrides the virtual `what()` member function (inherited from `std::exception`) to provide a human-readable error message, and may carry additional structured data. By deriving from `std::exception`, custom exceptions become catchable by handlers for `const std::exception&`, enabling polymorphic error handling. The C++ Core Guidelines (E.14) recommend using purpose-designed user-defined types as exceptions rather than built-in types.

**Beginner-Friendly Explanation**
A custom exception is your own “error type” that you create to describe a specific problem in your program—like `NetworkError` or `ValidationError`. By making it a subclass of the standard `std::exception`, it works with the same `try`/`catch` blocks you already use, so you can catch it specifically or catch it as part of a broader category of errors.

### Key Characteristics

- **Inheritance from `std::exception`** – ensures compatibility with generic exception handlers and the standard error interface.
- **Overridden `what()`** – provides a descriptive, human-readable message.
- **Polymorphic catching** – catching by `const std::exception&` or an intermediate base catches all derived types.
- **Structured error data** – custom members can carry error codes, context, and other diagnostic information.
- **Lightweight** – exceptions are copied during stack unwinding, so custom exceptions should be cheap to copy.
- **Type-based matching** – handlers are selected by the dynamic type of the exception.

### Prerequisites

- Basic C++ syntax (classes, inheritance, virtual functions).
- Understanding of `try`/`catch`/`throw` and the standard exception hierarchy.
- Familiarity with `std::exception`, `std::runtime_error`, and `std::logic_error`.
- Knowledge of `const` correctness and `noexcept`.
- Awareness of stack unwinding and exception safety.

### Related Programming Areas

- **Exception fundamentals** – `throw`, `try`, `catch`, and the standard exception hierarchy.
- **Exception safety** – custom exceptions must be safe to copy and destroy during unwinding.
- **Error handling design** – choosing between exceptions, error codes, and `std::optional`/`std::expected`.
- **API design** – documenting and exposing custom exception types as part of a library's contract.
- **Standard library** – `std::runtime_error` and `std::logic_error` provide ready-made bases.

### Core Concepts / Features

1. Custom exception classes
2. Exception inheritance
3. `std::exception`
4. `what()`
5. Exception constructors
6. Exception best practices

---

## 1. Custom Exception Classes

### Definitions

**Core Definition**
A custom exception class is a user-defined type that represents a specific error condition and derives from the standard exception hierarchy.

**Technical Definition**
A custom exception class is typically defined as a `class` or `struct` that publicly inherits from `std::exception` or one of its subclasses. It provides a constructor (usually taking a message string) and may override `what()` or rely on the base class's implementation. The class may carry additional data members for structured error reporting. Custom exception hierarchies allow you to define domain-specific error types that integrate seamlessly with the standard `try`/`catch` mechanism.

**Beginner-Friendly Explanation**
A custom exception is your own error class. Instead of throwing a generic `std::runtime_error`, you create a class like `FileNotFoundError` or `ConnectionTimeout` that describes exactly what went wrong. By deriving from `std::exception`, it plugs into the standard error-handling machinery.

### Purposes

- To represent domain-specific errors with meaningful names.
- To carry structured error data (codes, context, paths).
- To enable precise, type-based error handling.
- To integrate with the standard exception hierarchy.
- To improve code readability and maintainability.

### Syntax Rules and Structure

**General Syntax**
```cpp
#include <stdexcept>
#include <string>

class MyError : public std::runtime_error {
public:
    explicit MyError(const std::string& msg)
        : std::runtime_error(msg) {}
};
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| Base class | `std::exception` or a subclass (preferred: `std::runtime_error` or `std::logic_error`). |
| Constructor | Forwards the message to the base class. |
| `what()` | Inherited from `std::runtime_error` (stores the message) or overridden. |
| Additional members | Domain-specific data. |

**Syntax Rules**
- Derive publicly from `std::exception` or a subclass.
- Pass the message to the base class constructor for automatic storage and `what()` support.
- Keep the class lightweight (cheap to copy).
- Optionally mark the class `final` if it is not intended to be further derived.

**Constraints and Limitations**
- Avoid deriving directly from `std::exception` if you need a message; `std::exception` itself has no string constructor (though some implementations like MSVC provide one as a non-standard extension).
- Do not throw from destructors or `what()`.
- Avoid deep hierarchies that are hard to maintain.

### Annotated Code Examples

**Example 1: Simple custom exception**

```cpp
#include <iostream>
#include <stdexcept>
#include <string>

class FileNotFoundError : public std::runtime_error {
public:
    explicit FileNotFoundError(const std::string& path)
        : std::runtime_error("File not found: " + path) {}
};

int main() {
    try {
        throw FileNotFoundError("/etc/config");
    } catch (const FileNotFoundError& e) {
        std::cout << e.what() << '\n';
    }
}
```

**Expected Output**
```
File not found: /etc/config
```

**Why this output?**  
The constructor forwards a composed message to `std::runtime_error`, which stores it and returns it from `what()`.

**Example 2: Custom exception with structured data**

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
        std::cout << "Error " << e.code() << ": " << e.what() << '\n';
    }
}
```

**Expected Output**
```
Error 111: connection refused
```

**Why this output?**  
The `NetworkError` class stores an error code alongside the inherited message, providing structured context.

### Real-World Cases

- **Library error reporting** – custom root exceptions for library-specific failures.
- **Application domains** – database, network, and validation errors.
- **Structured diagnostics** – carrying error codes, file paths, or HTTP status codes.

---

## 2. Exception Inheritance

### Definitions

**Core Definition**
Exception inheritance is the organisation of exception types into a class hierarchy, enabling polymorphic catching and logical grouping of related errors.

**Technical Definition**
The standard exception hierarchy is rooted at `std::exception`, with two main branches: `std::logic_error` (errors detectable before execution, e.g., precondition violations) and `std::runtime_error` (errors detectable only during execution). Custom exception hierarchies extend this tree, allowing callers to catch errors at different levels of granularity. Because `catch` clauses match polymorphically, a handler for a base class catches all derived types.

**Beginner-Friendly Explanation**
Exception inheritance is a family tree of error types. By grouping related errors under a common base class, you can catch an entire family of errors with one handler, or catch a specific type for precise handling.

### Purposes

- To group related error types under a common base.
- To enable polymorphic catching (catch a base to handle derived types).
- To provide layered error handling at different abstraction levels.
- To allow callers to choose the granularity of error handling.

### Syntax Rules and Structure

**Hierarchy Design**
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
└── (custom exceptions)
```

**Component Breakdown**

| Level | Typical Use |
|-------|-------------|
| `std::exception` | Root; catch-all for standard exceptions. |
| `std::runtime_error` | Runtime failures; good base for custom exceptions. |
| `std::logic_error` | Programming errors; often unrecoverable. |
| Custom intermediate base | Domain-specific grouping (e.g., `AppError`). |
| Custom leaf | Specific error type (e.g., `NetworkError`). |

**Syntax Rules**
- Derive from `std::runtime_error` for recoverable runtime failures; from `std::logic_error` for programming errors.
- Order `catch` clauses from most derived to least derived.
- Provide a virtual destructor (inherited from `std::exception`).

**Constraints and Limitations**
- Catching by value slices derived objects; always catch by `const` reference.
- A flat hierarchy (everything derived directly from `std::exception`) provides no granularity.

### Annotated Code Example

```cpp
#include <iostream>
#include <stdexcept>

class AppError : public std::runtime_error {
public:
    explicit AppError(const std::string& msg) : std::runtime_error(msg) {}
};

class NetworkError : public AppError {
public:
    explicit NetworkError(const std::string& msg) : AppError(msg) {}
};

class DatabaseError : public AppError {
public:
    explicit DatabaseError(const std::string& msg) : AppError(msg) {}
};

int main() {
    try {
        throw NetworkError("connection lost");
    } catch (const NetworkError& e) {       // most derived first
        std::cout << "Network: " << e.what() << '\n';
    } catch (const AppError& e) {           // catches other AppErrors
        std::cout << "App: " << e.what() << '\n';
    } catch (const std::exception& e) {     // catches everything else
        std::cout << "Other: " << e.what() << '\n';
    }
}
```

**Expected Output**
```
Network: connection lost
```

**Why this output?**  
The `NetworkError` handler matches first because handlers are tried in order. The `AppError` handler would catch other `AppError` types (like `DatabaseError`), and the `std::exception` handler provides a final fallback.

### Real-World Cases

- **Layered applications** – data layer errors wrapped in domain errors; service layer wraps domain errors in API errors.
- **Library design** – a library root exception allowing users to catch all library errors with one handler.
- **Plugin systems** – plugin-specific exceptions grouped under a common base.

---

## 3. `std::exception`

### Definitions

**Core Definition**
`std::exception` is the base class for all exceptions thrown by the standard library, providing a common interface for error reporting.

**Technical Definition**
Defined in `<exception>`, `std::exception` provides a virtual destructor and a virtual `what()` member function that returns an explanatory string. It is the root of the standard exception hierarchy. The class `exception` does not contain a constructor that takes a message string, although it can be thrown without a message; calling `what()` on a base `std::exception` returns an implementation-defined default message.

**Beginner-Friendly Explanation**
`std::exception` is the “mother of all standard exceptions.” It guarantees that every standard exception has a `what()` message. Catching `const std::exception&` catches all standard exceptions and all custom exceptions that derive from it.

### Purposes

- To provide a common base for all standard exceptions.
- To enable catching any standard exception with one handler.
- To expose a human-readable error message via `what()`.
- To support polymorphic destruction.

### Syntax Rules and Structure

**General Syntax**
```cpp
#include <exception>

try {
    // ...
} catch (const std::exception& e) {
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
- `std::exception` itself has no string constructor in standard C++; use `std::runtime_error` for message storage.

**Constraints and Limitations**
- `what()` returns a pointer to a string whose lifetime is tied to the exception object.
- Throwing from `what()` is forbidden (it is `noexcept`).

### Annotated Code Example

```cpp
#include <exception>
#include <iostream>
#include <stdexcept>

int main() {
    try {
        throw std::runtime_error("something failed");
    } catch (const std::exception& e) {
        std::cout << "Caught: " << e.what() << '\n';
    }
}
```

**Expected Output**
```
Caught: something failed
```

**Why this output?**  
The `std::runtime_error` derives from `std::exception`; catching by `const std::exception&` handles it polymorphically and accesses the stored message via `what()`.

### Real-World Cases

- **Top-level handlers** – catching all standard exceptions in `main`.
- **Logging** – extracting `what()` for error messages.
- **Custom hierarchy roots** – deriving custom exceptions from `std::exception`.

---

## 4. `what()`

### Definitions

**Core Definition**
`what()` is a virtual member function of `std::exception` that returns a null-terminated string describing the exception.

**Technical Definition**
`virtual const char* what() const noexcept;` is declared in `std::exception` and overridden by standard library exception classes derived from it (e.g., `std::runtime_error` stores a message string and returns it from `what()`). The returned pointer is guaranteed to be valid at least until the exception object from which it is obtained is destroyed or until a non-const member function (such as copy assignment) is called on the exception object. In C++11 and later, `what()` is `noexcept` (previously `throw()`).

**Beginner-Friendly Explanation**
`what()` is the function that gives you the error message. When you catch an exception, you call `e.what()` to get a human-readable description of what went wrong.

### Purposes

- To provide a human-readable description of the error.
- To support logging and diagnostics.
- To serve as the primary interface for generic exception handling.
- To carry context without requiring downcasting.

### Syntax Rules and Structure

**General Syntax**
```cpp
const char* message = e.what();
```

**Component Breakdown**

| Aspect | Requirement |
|--------|-------------|
| Signature | `virtual const char* what() const noexcept` |
| Return type | `const char*` (null-terminated string). |
| Lifetime | Valid until the exception object is destroyed or modified. |

**Syntax Rules**
- Overrides must be `const` and `noexcept`.
- Do not throw from `what()`.
- Do not return a pointer to a temporary that will be destroyed.
- The pointer remains valid for the lifetime of the exception object (or until a non-const member is called).

**Constraints and Limitations**
- Returns `const char*`, not `std::string`; converting may allocate.
- Messages are not automatically localized or formatted.
- Some implementations return a static string for `std::exception::what()`.

### Annotated Code Example

```cpp
#include <iostream>
#include <stdexcept>
#include <string>

class ValidationError : public std::runtime_error {
public:
    explicit ValidationError(const std::string& msg)
        : std::runtime_error(msg) {}
};

int main() {
    try {
        throw ValidationError("email address is invalid");
    } catch (const std::exception& e) {
        std::cout << "Error: " << e.what() << '\n';
    }
}
```

**Expected Output**
```
Error: email address is invalid
```

**Why this output?**  
`std::runtime_error` stores the message and returns it from `what()`; the custom class simply forwards the message to the base constructor.

### Real-World Cases

- **Logging** – recording `what()` at error boundaries.
- **User-facing errors** – translating exception messages to friendly UI text.
- **Structured diagnostics** – combining `what()` with custom accessors.

---

## 5. Exception Constructors

### Definitions

**Core Definition**
Exception constructors are the constructors of exception classes that initialise the exception object, typically storing a descriptive message.

**Technical Definition**
`std::exception` provides a default constructor and a copy constructor, but no constructor that takes a message string. Standard library exception classes derived from `std::exception` (such as `std::runtime_error` and `std::logic_error`) provide constructors that accept a `std::string` or `const char*` message and store it for retrieval by `what()`. Custom exception classes typically forward a message to their base class constructor. All standard library exception classes derived from `std::exception` must have a publicly accessible copy constructor and copy assignment operator that do not exit via an exception.

**Beginner-Friendly Explanation**
Exception constructors are how you set up the error message when you create an exception. You pass a description to the constructor, and the exception stores it so `what()` can return it later.

### Purposes

- To initialise the exception object with a descriptive message.
- To store structured error data in custom exceptions.
- To forward messages to base-class constructors.
- To ensure the exception is copyable during unwinding.

### Syntax Rules and Structure

**General Syntax**
```cpp
// Custom exception forwarding to std::runtime_error
class MyError : public std::runtime_error {
public:
    explicit MyError(const std::string& msg)
        : std::runtime_error(msg) {}
};

// Custom exception with additional data
class DetailedError : public std::runtime_error {
    int code_;
public:
    DetailedError(const std::string& msg, int code)
        : std::runtime_error(msg), code_(code) {}
};
```

**Component Breakdown**

| Constructor | Purpose |
|-------------|---------|
| Default | Creates an exception with an implementation-defined `what()` message. |
| Copy | Copies the exception (required for throwing). |
| Message-taking | Stores a description for `what()` (provided by `std::runtime_error`/`std::logic_error`). |

**Syntax Rules**
- Prefer deriving from `std::runtime_error` or `std::logic_error` to get message storage for free.
- Forward the message to the base class constructor.
- Mark single-argument constructors `explicit` to prevent implicit conversions.
- Ensure the copy constructor and copy assignment are `noexcept` and do not throw.

**Constraints and Limitations**
- `std::exception` itself has no message-taking constructor in standard C++; deriving directly from it means you must override `what()` and manage the message yourself.
- Some implementations (e.g., MSVC) provide a non-standard `std::exception(const char*)` constructor; do not rely on it for portability.

### Annotated Code Example

```cpp
#include <iostream>
#include <stdexcept>
#include <string>

class HttpError : public std::runtime_error {
    int status_code_;
public:
    HttpError(int status, const std::string& msg)
        : std::runtime_error(msg), status_code_(status) {}

    int status_code() const noexcept { return status_code_; }
};

int main() {
    try {
        throw HttpError(404, "Not Found");
    } catch (const HttpError& e) {
        std::cout << "HTTP " << e.status_code() << ": " << e.what() << '\n';
    }
}
```

**Expected Output**
```
HTTP 404: Not Found
```

**Why this output?**  
The constructor forwards the message to `std::runtime_error` and stores the status code in the custom member.

### Real-World Cases

- **HTTP clients** – custom exceptions carrying status codes and messages.
- **Database libraries** – exceptions carrying SQL error codes.
- **File handling** – exceptions carrying file paths and system error codes.

---

## 6. Exception Best Practices

### Definitions

**Core Definition**
Exception best practices for custom exceptions are guidelines for designing, implementing, and using custom exception types correctly, safely, and idiomatically.

**Technical Definition**
These practices are grounded in the C++ Core Guidelines (especially E.14: “Use purpose-designed user-defined types as exceptions”), the High Integrity C++ Coding Standard (rule 15.1: every thrown value should be an instance of `std::exception`), and community consensus. Key rules include: derive from `std::exception` or a subclass; override `what()` via constructor string; catch by `const` reference; catch most-derived first; keep exceptions lightweight; and make `what()` `noexcept`.

**Beginner-Friendly Explanation**
Best practices for custom exceptions are simple rules: make them inherit from `std::exception`, pass the error message to the constructor, catch them by `const` reference, and keep them small and cheap to copy.

### Key Guidelines

| Guideline | Summary |
|-----------|---------|
| **Derive from `std::exception` or a subclass** | Ensures compatibility with generic handlers. |
| **Use `std::runtime_error` / `std::logic_error` as bases** | Provides message storage and `what()` for free. |
| **Override `what()` via constructor string** | Provides a human-readable error message. |
| **Catch by `const` reference** | Prevents slicing of derived exception data. |
| **Catch most-derived first** | C++ matches the first matching catch clause in order. |
| **Keep exceptions lightweight** | They are copied during stack unwinding. |
| **Make `what()` `noexcept`** | It is declared `noexcept` in the base class. |
| **Use purpose-designed types (Core Guidelines E.14)** | Avoid throwing built-in types like `int` or `const char*`. |
| **Provide a virtual destructor** | Inherited from `std::exception`. |
| **Document your exception types** | Callers need to know what to catch. |

### Annotated Code Example

```cpp
#include <iostream>
#include <stdexcept>
#include <string>

// Good hierarchy: granular, derived from std::runtime_error
class AppError : public std::runtime_error {
public:
    explicit AppError(const std::string& msg) : std::runtime_error(msg) {}
};

class NetworkError : public AppError {
    int status_;
public:
    NetworkError(const std::string& msg, int status)
        : AppError(msg), status_(status) {}
    int status() const noexcept { return status_; }
};

int main() {
    try {
        throw NetworkError("connection timeout", 408);
    } catch (const NetworkError& e) {       // catch most-derived first
        std::cout << "Network " << e.status() << ": " << e.what() << '\n';
    } catch (const AppError& e) {           // catches other AppErrors
        std::cout << "App: " << e.what() << '\n';
    } catch (const std::exception& e) {     // final fallback
        std::cout << "Other: " << e.what() << '\n';
    }
}
```

**Expected Output**
```
Network 408: connection timeout
```

**Why this output?**  
The hierarchy provides granularity, the exception is caught by `const` reference (no slicing), and handlers are ordered from most-derived to least-derived.

### Real-World Cases

- **Code reviews** – checking for catch-by-value and missing `noexcept` on `what()`.
- **Static analysis** – clang-tidy `hicpp-exception-baseclass` enforces deriving from `std::exception`.
- **Library API design** – documenting custom exception hierarchies for users.

---

## References

- `std::exception` — cppreference – https://en.cppreference.com/w/cpp/error/exception
- `std::exception::what` — cppreference – https://en.cppreference.com/w/cpp/error/exception/what
- Using Exceptions — Apache C++ Standard Library User's Guide – http://svn.apache.org/repos/asf/stdcxx/tags/4.2.0-rc-3/doc/stdlibug/18-3.html
- Use custom exception classes in a hierarchy with `std::exception` base — Dr-Sergey/learn_cpp – https://github.com/Dr-Sergey/learn_cpp/blob/08279578ce23b1cbf2d3394357620600db618eca/11_Error_Handling/Use_custom_exception_classes_in_a_hierarchy_with_stdexception_base.md
- C++ Core Guidelines — E.14: Use purpose-designed user-defined types as exceptions – https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Re-exception-types
- High Integrity C++ Coding Standard — Rule 15.1 (hicpp-exception-baseclass) – https://man.freebsd.org/cgi/man.cgi?query=extraclangtools14&sektion=1&manpath=freebsd-ports#15
- Creating new exceptions derived from existing — Stack Overflow – https://browse.library.kiwix.org/content/stackoverflow.com_en_all_nopic_2022-07/questions/18025755/creating-new-exceptions-derived-from-existing
- `std::exception` is the class whose only purpose is to serve as the base class — Stack Overflow – https://stackoverflow.com/revisions/01633996-3311-4960-868d-b389a55abd80/view-source
- Example custom exceptions (pybind11) — Android Googlesource – https://android.googlesource.com/platform/external/python/pybind11/+/00a300950bd0881a4e275a1646f2ad71abb59e65/example/example-custom-exceptions.cpp
- Custom exception example (StudentException) — PacktPublishing – https://raw.githubusercontent.com/PacktPublishing/Demystified-Object-Oriented-Programming-with-CPP/master/Chapter11/Chp11-Ex5.cpp

---

This cheat sheet provides a comprehensive, structured reference for C++ custom exceptions, covering custom exception classes, exception inheritance, `std::exception`, `what()`, exception constructors, and best practices, with annotated code examples, expected outputs, real-world cases, and authoritative references.