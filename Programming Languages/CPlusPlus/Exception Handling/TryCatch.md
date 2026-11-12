# C++ Try-Catch — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
Try-catch is the C++ language mechanism for handling exceptions: a `try` block encloses code that may throw, and one or more `catch` blocks provide handlers for exceptions thrown from within that guarded region.

**Technical Definition**
A try-block is a statement that associates one or more exception handlers (catch-clauses) with a compound statement. If an exception is thrown from the guarded compound statement (or from a function called directly or indirectly from it), the exception is matched against the handlers in the handler sequence, in order of appearance. The first matching handler is entered, and the program continues after the handler sequence. If no handler matches, the exception propagates to the next dynamically enclosing try-block; if none exists, `std::terminate` is called.

**Beginner-Friendly Explanation**
Try-catch is C++'s "safety net" for errors. You wrap risky code in a `try` block and attach `catch` blocks that know how to handle specific kinds of problems. When something goes wrong, the program jumps straight to the right `catch` block, cleaning up temporary objects along the way, instead of crashing or returning error codes through every caller.

### Key Characteristics

- **Guarded region** – the `try` block defines the scope whose exceptions are handled by the attached `catch` clauses.
- **Ordered matching** – handlers are tried in order; the first match wins.
- **Reference-based catching** – catching by `const` reference avoids slicing and copies.
- **Catch-all fallback** – `catch (...)` matches any exception and must be last.
- **Nested handling** – try-blocks can be nested, and exceptions can be rethrown or wrapped.
- **Stack unwinding** – automatic objects between the throw site and the handler are destroyed in reverse order of construction.
- **No-throw guarantees** – a `try`/`catch(...)` pair can ensure a function never lets an exception escape.

### Prerequisites

- Basic C++ syntax (functions, classes, constructors/destructors).
- Understanding of inheritance and polymorphism (for catching by base reference).
- Familiarity with `throw` and the exception hierarchy (`std::exception`).
- Knowledge of stack unwinding and RAII.
- Awareness of `noexcept` specifications.

### Related Programming Areas

- **Exception fundamentals** – `throw`, `std::exception`, standard exception types.
- **RAII** – destructors run during unwinding, releasing resources.
- **Error handling design** – exceptions vs. error codes vs. `std::optional`/`std::expected`.
- **Concurrency** – exceptions do not automatically cross thread boundaries; `std::exception_ptr` enables transport.
- **Standard library** – containers, streams, and algorithms throw exceptions on certain errors.

### Core Concepts / Features

1. `try` block
2. `catch` block
3. Multiple catch blocks
4. Catch-all (`catch (...)`)
5. Nested try-catch
6. Re-throwing exceptions
7. Exception chaining
8. Exception messages
9. Exception best practices

---

## 1. `try` Block

### Definitions

**Core Definition**
A `try` block is a compound statement whose exceptions are handled by an associated sequence of `catch` handlers.

**Technical Definition**
A try-block is a statement of the form `try compound-statement handler-seq`. When an exception is thrown from the compound statement (the guarded section), the exception is matched against the handlers in the handler sequence. A try-block may also appear as a function-try-block, wrapping an entire function body including the constructor initializer list.

**Beginner-Friendly Explanation**
The `try` block is the "attempt" part: you put the code that might fail inside it, and the attached `catch` blocks are the "plan" for what to do if it does fail.

### Purposes

- To define the region of code whose exceptions will be caught by the associated handlers.
- To associate exception handlers with a specific compound statement.
- To enable stack unwinding from the throw site to the handler.
- To support function-level exception handling via function-try-blocks.

### Syntax Rules and Structure

**General Syntax**
```cpp
try {
    // guarded compound-statement
} catch (/* handler parameter */) {
    // handler body
}
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `try` | Keyword introducing the try-block. |
| `compound-statement` | The guarded code; exceptions thrown here (or from called functions) are handled. |
| `handler-seq` | A non-empty sequence of `catch` clauses. |

**Syntax Rules**
- A try-block must be followed by at least one `catch` handler.
- A `try` block defines a scope; variables declared inside are not visible in the handlers.
- Jump statements (`goto`, `break`, `return`, `continue`) may transfer control out of a try-block.
- Function-try-blocks wrap an entire function body and can catch exceptions from member initializers (in constructors).

**Constraints and Limitations**
- An exception thrown in a `try` block is **not** handled by the handlers of that same try-block if it is thrown during the execution of a handler (handlers are outside the guarded section).
- A `try` block does not catch exceptions thrown from destructors during unwinding of that same block in a way that would allow recovery (throwing from a destructor during unwinding calls `std::terminate`).

### Annotated Code Examples

**Example 1: Basic try-block**

```cpp
#include <iostream>
#include <stdexcept>

int main() {
    try {
        throw std::runtime_error("something failed");
    } catch (const std::exception& e) {
        std::cout << "Handled: " << e.what() << '\n';
    }
}
```

**Expected Output**
```
Handled: something failed
```

**Why this output?**  
The `try` block throws a `runtime_error`; the `catch` handler for `const std::exception&` matches (base-class rule) and prints the message.

**Example 2: Function-try-block (constructor)**

```cpp
#include <iostream>
#include <stdexcept>

struct Widget {
    int value;
    Widget(int v) try : value(v) {
        if (v < 0) throw std::invalid_argument("negative");
    } catch (const std::exception& e) {
        std::cout << "Constructor caught: " << e.what() << '\n';
        throw;                             // must rethrow from constructor handler
    }
};

int main() {
    try {
        Widget w(-1);
    } catch (const std::exception& e) {
        std::cout << "Main caught: " << e.what() << '\n';
    }
}
```

**Expected Output**
```
Constructor caught: negative
Main caught: negative
```

**Why this output?**  
The function-try-block on the constructor catches the exception from the member initializer/body; because the object is not fully constructed, the handler must rethrow, and `main` catches it again.

### Real-World Cases

- **Top-level `main`** – wrapping the entire program body in a try-block to log and exit gracefully.
- **Library boundaries** – wrapping public API functions to translate exceptions into error codes.
- **Constructors** – using function-try-blocks to handle exceptions from member initializers.

---

## 2. `catch` Block

### Definitions

**Core Definition**
A `catch` block (handler) is a clause that receives and processes an exception whose type matches its parameter.

**Technical Definition**
A handler has the form `catch (attr(optional) type-specifier-seq declarator) compound-statement`. When an exception of type `E` is thrown, a handler with parameter type `T` matches if `E` and `T` are the same type (ignoring top-level cv-qualifiers), or if `T` is an unambiguous public base class of `E`, or (for pointer types) if `E` can be converted to `T` by a standard pointer conversion. The handler's parameter is initialized from the exception object, and the handler's compound statement executes.

**Beginner-Friendly Explanation**
A `catch` block is the "what to do" part: it receives the error object and contains the recovery code. You can have different `catch` blocks for different types of errors.

### Purposes

- To receive and handle a specific type of exception.
- To access the exception object (via the handler parameter) for logging or recovery.
- To translate exceptions from one type to another.
- To provide a fallback handler (`catch (...)`).

### Syntax Rules and Structure

**General Syntax**
```cpp
catch (const Type& e) {
    // handler body
}
```

**Component Breakdown**

| Form | Behaviour |
|------|-----------|
| `catch (T& e)` | Binds to the exception object; can modify it (rarely useful). |
| `catch (const T& e)` | Binds without copying; **preferred**. |
| `catch (T e)` | Copies the exception object (risk of slicing). |
| `catch (...)` | Matches any exception; must be last. |

**Syntax Rules**
- Handlers are tried in order; the first match is selected.
- A handler parameter of array or function type is adjusted to a pointer.
- An incomplete type (other than `void`) in a handler parameter is ill-formed.
- `throw;` inside a handler rethrows the current exception; `throw e;` throws a **copy** (possibly sliced).

**Constraints and Limitations**
- Catching by value slices derived objects when the handler type is a base class.
- A handler that does not rethrow consumes the exception.
- Handlers are outside the guarded section; exceptions thrown inside a handler are not caught by that same handler sequence.

### Annotated Code Examples

**Example 1: Catching by `const` reference**

```cpp
#include <iostream>
#include <stdexcept>

int main() {
    try {
        throw std::out_of_range("index 5 is invalid");
    } catch (const std::out_of_range& e) {
        std::cout << "Out of range: " << e.what() << '\n';
    }
}
```

**Expected Output**
```
Out of range: index 5 is invalid
```

**Why this output?**  
The handler binds to the exception object by `const` reference, avoiding a copy and allowing polymorphic access to `what()`.

**Example 2: Slicing pitfall (catch by value)**

```cpp
#include <iostream>
#include <stdexcept>

struct DerivedError : std::runtime_error {
    int code;
    DerivedError(const std::string& msg, int c)
        : std::runtime_error(msg), code(c) {}
};

int main() {
    try {
        throw DerivedError("failure", 42);
    } catch (std::runtime_error e) {       // BAD: caught by value → sliced
        std::cout << "Caught: " << e.what() << '\n';
        // e.code is inaccessible; the DerivedError part was sliced off
    }
}
```

**Expected Output**
```
Caught: failure
```

**Why this output?**  
Catching by value slices the `DerivedError` object down to `std::runtime_error`; the `code` member is lost. Catching by `const std::runtime_error&` would preserve the dynamic type (though accessing `code` would still require catching `DerivedError` or using `dynamic_cast`).

### Real-World Cases

- **Domain-specific error recovery** – catching `NetworkError` to retry, `DatabaseError` to roll back.
- **Logging** – catching `const std::exception&` to log `what()` at a boundary.
- **Translation** – catching a low-level exception and rethrowing a higher-level type.

---

## 3. Multiple Catch Blocks

### Definitions

**Core Definition**
Multiple catch blocks allow a single try-block to handle several different exception types with separate handlers.

**Technical Definition**
A handler sequence is an ordered list of handlers. When an exception is thrown, the handlers are examined in order of appearance; the first handler whose type matches the exception is selected. Handlers later in the sequence are ignored for that exception.

**Beginner-Friendly Explanation**
You can attach several `catch` blocks to one `try` block, each handling a different kind of error. C++ tries them top to bottom and uses the first one that fits.

### Purposes

- To handle different exception types with type-specific recovery logic.
- To order handlers from most derived to least derived for precise matching.
- To provide a general fallback after specific handlers.
- To avoid duplicating the guarded code for each error type.

### Syntax Rules and Structure

**General Syntax**
```cpp
try {
    // guarded code
} catch (const Type1& e) {
    // handler 1
} catch (const Type2& e) {
    // handler 2
} catch (...) {
    // catch-all
}
```

**Component Breakdown**

| Position | Typical Handler |
|----------|-----------------|
| First | Most-derived / most-specific type. |
| Middle | Intermediate base classes or other specific types. |
| Last | `catch (...)` catch-all (if present). |

**Syntax Rules**
- Handlers are matched in order; the first match wins.
- A handler for a derived class must precede a handler for its base class; otherwise the base-class handler shadows the derived one.
- `catch (...)` must be the last handler in the sequence.

**Constraints and Limitations**
- Unreachable handlers (those after a catch-all or after a base-class handler that covers them) may trigger compiler warnings.
- Only one handler executes per exception.

### Annotated Code Example

```cpp
#include <iostream>
#include <stdexcept>

int main() {
    try {
        throw std::out_of_range("index too large");
    } catch (const std::out_of_range& e) {       // most specific first
        std::cout << "Out of range: " << e.what() << '\n';
    } catch (const std::logic_error& e) {        // base of out_of_range
        std::cout << "Logic error: " << e.what() << '\n';
    } catch (const std::exception& e) {          // base of all standard
        std::cout << "Standard exception: " << e.what() << '\n';
    } catch (...) {
        std::cout << "Unknown exception\n";
    }
}
```

**Expected Output**
```
Out of range: index too large
```

**Why this output?**  
`std::out_of_range` matches the first handler exactly; the later handlers are not considered.

### Real-World Cases

- **Layered APIs** – specific domain exceptions first, then `std::exception`, then `...`.
- **User-facing error messages** – mapping exception types to friendly messages.
- **Retry logic** – different handlers for transient vs. permanent failures.

---

## 4. Catch-All (`catch (...)`)

### Definitions

**Core Definition**
The catch-all handler `catch (...)` matches an exception of any type.

**Technical Definition**
The handler `catch (...)` matches exceptions of any type. If present, it can only be the last handler in a handler sequence. It is commonly used to ensure that no uncaught exceptions escape from a function that offers a nothrow guarantee, or to perform cleanup before rethrowing.

**Beginner-Friendly Explanation**
`catch (...)` is the "catch everything else" handler. It is a safety net that catches any exception not handled by earlier, more specific handlers.

### Purposes

- To ensure no exception escapes a boundary (e.g., a `noexcept` function, a C callback, a thread entry point).
- To perform cleanup or logging before rethrowing.
- To provide a final fallback when the exception type is unknown.
- To enable exception transport (`std::current_exception()`).

### Syntax Rules and Structure

**General Syntax**
```cpp
try {
    // guarded code
} catch (...) {
    // handler body
}
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `...` | Ellipsis matching any exception type. |
| Position | Must be the last handler. |

**Syntax Rules**
- `catch (...)` can only appear as the final handler.
- Inside a catch-all, the exception object is not directly accessible; use `std::current_exception()` to capture it.
- `throw;` inside a catch-all rethrows the original exception.

**Constraints and Limitations**
- You cannot inspect the exception type directly inside `catch (...)` without `std::current_exception()` / `std::rethrow_exception`.
- Catching everything indiscriminately can hide bugs if not carefully managed.

### Annotated Code Example

```cpp
#include <iostream>
#include <exception>
#include <stdexcept>

void risky() {
    throw std::runtime_error("unknown failure");
}

int main() {
    try {
        risky();
    } catch (...) {
        std::cout << "Caught something\n";
        try {
            throw;                         // rethrow the current exception
        } catch (const std::exception& e) {
            std::cout << "It was: " << e.what() << '\n';
        }
    }
}
```

**Expected Output**
```
Caught something
It was: unknown failure
```

**Why this output?**  
`catch (...)` catches the unknown exception; `throw;` rethrows it so it can be inspected by a more specific handler inside the catch-all.

### Real-World Cases

- **C API boundaries** – preventing exceptions from crossing into C code.
- **Thread entry points** – ensuring exceptions in threads are logged and do not terminate the process silently.
- **`noexcept` functions** – wrapping the body in `try { ... } catch (...) { std::terminate(); }` or logging.

---

## 5. Nested Try-Catch

### Definitions

**Core Definition**
Nested try-catch refers to placing a try-block (with its handlers) inside another try-block or inside a catch handler.

**Technical Definition**
Try-blocks can be nested at any depth. An exception thrown in an inner try-block is first matched against the inner handlers; if none matches, it propagates to the enclosing try-block's handlers. A try-block inside a handler handles exceptions thrown by that handler.

**Beginner-Friendly Explanation**
You can put a try-catch inside another try-catch. If the inner one cannot handle the error, the outer one gets a chance.

### Purposes

- To handle exceptions at different levels of granularity.
- To protect cleanup code in a handler from throwing new exceptions.
- To implement retry logic with a fallback handler.
- To translate exceptions at each layer.

### Syntax Rules and Structure

**General Syntax**
```cpp
try {
    try {
        // inner guarded code
    } catch (const InnerError& e) {
        // inner handler
    }
} catch (const OuterError& e) {
    // outer handler
}
```

**Component Breakdown**

| Level | Responsibility |
|-------|----------------|
| Inner try-block | Handles exceptions specific to its guarded code. |
| Outer try-block | Handles exceptions that escape the inner try-block or are thrown by inner handlers. |

**Syntax Rules**
- An exception not matched by inner handlers propagates to the outer handlers.
- Exceptions thrown inside an inner handler are handled by the outer try-block, not by the inner handler sequence.
- Nesting depth is limited only by implementation constraints.

**Constraints and Limitations**
- Deep nesting can hurt readability; prefer functions and RAII where possible.
- A handler that throws a new exception is not caught by the same try-block's handlers.

### Annotated Code Example

```cpp
#include <iostream>
#include <stdexcept>

int main() {
    try {
        try {
            throw std::runtime_error("inner failure");
        } catch (const std::logic_error& e) {   // does not match
            std::cout << "Inner logic: " << e.what() << '\n';
        }
    } catch (const std::exception& e) {         // catches the inner exception
        std::cout << "Outer caught: " << e.what() << '\n';
    }
}
```

**Expected Output**
```
Outer caught: inner failure
```

**Why this output?**  
The inner handler for `logic_error` does not match `runtime_error`; the exception propagates to the outer handler for `std::exception`.

### Real-World Cases

- **Retry with fallback** – inner try retries an operation; outer try handles permanent failure.
- **Cleanup safety** – wrapping cleanup code in a handler to prevent secondary exceptions.
- **Layered protocol handling** – handling frame-level errors inside packet-level errors.

---

## 6. Re-throwing Exceptions

### Definitions

**Core Definition**
Re-throwing means throwing the currently handled exception again, either to let an outer handler process it or to preserve it after partial handling.

**Technical Definition**
The statement `throw;` (with no operand) rethrows the currently handled exception. It is valid only inside a catch handler (or a function called from one). Re-throwing with `throw;` preserves the original exception object's dynamic type. Using `throw e;` instead throws a **new copy** of `e`, sliced to the static type of `e`.

**Beginner-Friendly Explanation**
Sometimes you want to catch an exception, do something (like logging), and then let it continue to an outer handler. `throw;` does exactly that—it re-sends the same exception.

### Purposes

- To add context (logging, cleanup) without consuming the exception.
- To let an outer handler make the final decision.
- To preserve the original exception type (avoiding slicing).
- To implement exception translation at a boundary.

### Syntax Rules and Structure

**General Syntax**
```cpp
catch (const SomeType& e) {
    // partial handling
    throw;                                 // rethrow the original exception
}
```

**Component Breakdown**

| Form | Behaviour |
|------|-----------|
| `throw;` | Rethrows the current exception (no copy, original type preserved). |
| `throw e;` | Throws a new copy of `e` (sliced to static type). |

**Syntax Rules**
- `throw;` must be inside a catch handler or a function called from one.
- `throw;` with no active exception calls `std::terminate`.
- Prefer `throw;` over `throw e;` to avoid slicing.

**Constraints and Limitations**
- `throw;` cannot be used outside a handler.
- If the handler exits normally without rethrowing, the exception is considered handled.

### Annotated Code Example

```cpp
#include <iostream>
#include <stdexcept>

void process() {
    try {
        throw std::runtime_error("original error");
    } catch (const std::exception& e) {
        std::cout << "Logging: " << e.what() << '\n';
        throw;                             // rethrow the same exception
    }
}

int main() {
    try {
        process();
    } catch (const std::runtime_error& e) {
        std::cout << "Main handled: " << e.what() << '\n';
    }
}
```

**Expected Output**
```
Logging: original error
Main handled: original error
```

**Why this output?**  
The inner handler logs and rethrows with `throw;`; `main` catches the original `runtime_error` (its dynamic type is preserved).

### Real-World Cases

- **Logging middleware** – catching, logging, and rethrowing to the caller.
- **Resource cleanup** – releasing resources in a handler, then rethrowing.
- **Exception translation** – catching a low-level exception and rethrowing a domain-specific one (usually `throw NewError(...);` rather than `throw;`).

---

## 7. Exception Chaining

### Definitions

**Core Definition**
Exception chaining (nested exceptions) is a technique for wrapping a caught exception inside a new exception, preserving the original as a nested cause.

**Technical Definition**
C++11 provides `std::nested_exception`, `std::throw_with_nested`, and `std::rethrow_if_nested` in `<exception>`. When `throw_with_nested(e)` is called inside a catch handler, it throws an unspecified type that derives publicly from both `std::nested_exception` and the decayed type of `e`. The `nested_exception` base captures the currently handled exception via `std::current_exception()`. `rethrow_if_nested(e)` rethrows the nested exception (if any), allowing recursive traversal of the chain.

**Beginner-Friendly Explanation**
Exception chaining lets you say "this high-level error was caused by that low-level error." You wrap the original exception in a new one, and you can later peel back the layers to see the root cause.

### Purposes

- To add context at each layer without losing the original cause.
- To preserve the full error history for debugging.
- To translate between abstraction levels (e.g., I/O error → application error) while retaining the root cause.
- To support structured logging of exception chains.

### Syntax Rules and Structure

**General Syntax**
```cpp
#include <exception>

try {
    // operation that may throw
} catch (...) {
    std::throw_with_nested(NewException("high-level context"));
}

// Later, to inspect:
try {
    // something that may throw a nested exception
} catch (const std::exception& e) {
    std::rethrow_if_nested(e);             // rethrows the nested cause
}
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `std::nested_exception` | Mixin base class that stores the nested exception. |
| `std::throw_with_nested(e)` | Throws `e` with the current exception as a nested cause. |
| `std::rethrow_if_nested(e)` | Rethrows the nested exception if `e` derives from `nested_exception`. |

**Syntax Rules**
- `throw_with_nested` must be called inside a catch handler (to have a current exception to capture).
- If the current exception is already a `nested_exception` (or derives from it), `throw_with_nested` does **not** add another level; it throws the current exception unchanged.
- `rethrow_if_nested` is a no-op if `e` does not derive from `nested_exception`.

**Constraints and Limitations**
- The exact type thrown by `throw_with_nested` is unspecified; catch it as `const std::exception&` or the original type.
- Recursive inspection requires a helper that catches `nested_exception` and recurses.
- Not all exception types benefit from chaining; use it when context matters.

### Annotated Code Example

```cpp
#include <exception>
#include <fstream>
#include <iostream>
#include <stdexcept>
#include <string>

void print_exception(const std::exception& e, int level = 0) {
    std::cerr << std::string(level, ' ') << "exception: " << e.what() << '\n';
    try {
        std::rethrow_if_nested(e);
    } catch (const std::exception& nested) {
        print_exception(nested, level + 1);
    } catch (...) {}
}

void open_file(const std::string& path) {
    try {
        std::ifstream file(path);
        file.exceptions(std::ios_base::failbit);
    } catch (...) {
        std::throw_with_nested(
            std::runtime_error("Couldn't open " + path));
    }
}

int main() {
    try {
        open_file("nonexistent.txt");
    } catch (const std::exception& e) {
        print_exception(e);
    }
}
```

**Expected Output** (illustrative)
```
exception: Couldn't open nonexistent.txt
 exception: basic_ios::clear: iostream error
```

**Why this output?**  
`open_file` catches the I/O exception and wraps it in a `runtime_error` via `throw_with_nested`. `print_exception` recursively rethrows and prints the nested cause, showing both levels.

### Real-World Cases

- **Layered applications** – data layer wraps DB errors in domain errors; service layer wraps domain errors in API errors.
- **Plugin systems** – plugin errors wrapped with plugin identity before crossing into the host.
- **Diagnostics** – preserving root-cause information for post-mortem analysis.

---

## 8. Exception Messages

### Definitions

**Core Definition**
Exception messages are human-readable descriptions of an error, accessed via `what()` on `std::exception`-derived types, or via custom accessors on user-defined exception classes.

**Technical Definition**
`std::exception::what()` is a virtual function declared `noexcept` that returns a null-terminated string describing the exception. Standard library exception classes derived from `std::exception` store a message string (e.g., `std::runtime_error`, `std::logic_error`) and return it from `what()`. Custom exception classes typically pass a formatted message to a base class constructor and may expose additional structured data through separate accessors.

**Beginner-Friendly Explanation**
The exception message is the "error text" that tells you what went wrong. You can print it with `e.what()` or store it for logging.

### Purposes

- To provide a human-readable description of the error.
- To support logging and diagnostics.
- To carry context (e.g., the invalid value, the file path, the operation).
- To serve as the primary interface for generic exception handling.

### Syntax Rules and Structure

**General Syntax**
```cpp
try {
    // ...
} catch (const std::exception& e) {
    std::cerr << e.what() << '\n';
}
```

**Component Breakdown**

| Mechanism | Description |
|-----------|-------------|
| `what()` | Virtual, `noexcept`; returns `const char*`. |
| Constructor string | `std::runtime_error(msg)`, `std::logic_error(msg)` store the message. |
| Custom accessor | e.g., `error_code()`, `context()` on user-defined exceptions. |

**Syntax Rules**
- `what()` must not throw; overrides must be `noexcept`.
- The returned pointer is valid for the lifetime of the exception object (or until `what()` is called again, for some implementations).
- Prefer `std::runtime_error` or `std::logic_error` as bases for custom exceptions to get message storage for free.

**Constraints and Limitations**
- `what()` returns `const char*`, not `std::string`; converting to `std::string` may allocate.
- Messages are not automatically localized or formatted; use `std::format` or similar for structured messages.
- Some implementations return a static string for `std::exception::what()`.

### Annotated Code Example

```cpp
#include <iostream>
#include <stdexcept>
#include <string>

class FileError : public std::runtime_error {
    std::string path_;
public:
    FileError(const std::string& msg, std::string path)
        : std::runtime_error(msg), path_(std::move(path)) {}

    const std::string& path() const noexcept { return path_; }
};

int main() {
    try {
        throw FileError("cannot read file", "/etc/config");
    } catch (const FileError& e) {
        std::cout << "Error: " << e.what()
                  << " (path: " << e.path() << ")\n";
    }
}
```

**Expected Output**
```
Error: cannot read file (path: /etc/config)
```

**Why this output?**  
`what()` returns the message passed to the `runtime_error` base; `path()` exposes structured context.

### Real-World Cases

- **Logging** – recording `what()` at error boundaries.
- **User-facing errors** – translating exception messages to friendly UI text.
- **Structured diagnostics** – combining `what()` with custom accessors for error codes and context.

---

## 9. Exception Best Practices

### Definitions

**Core Definition**
Exception best practices are guidelines for using try-catch correctly, safely, and idiomatically, as codified in the C++ Core Guidelines (rules E.12–E.16) and the SEI CERT C++ Coding Standard.

**Technical Definition**
Key rules include: throw by value and catch by `const` reference (E.15); use `noexcept` when a function must not throw (E.12, F.6); minimize the use of explicit `try`/`catch` (E.12 note); ensure destructors, deallocation functions, and `swap` never throw (E.16); and avoid catching by value to prevent slicing (CERT ERR61-CPP). Handlers should be ordered from most specific to most general, with `catch (...)` last.

**Beginner-Friendly Explanation**
Best practices for try-catch are simple: catch by `const` reference, don't use try-catch for normal control flow, don't let destructors throw, and order your handlers from specific to general.

### Key Guidelines

| Guideline | Summary |
|-----------|---------|
| **E.15: Throw by value, catch by reference** | Prevents slicing and unnecessary copies. |
| **CERT ERR61-CPP** | Always catch class-type exceptions by lvalue reference (preferably `const`). |
| **E.12 / F.6: Use `noexcept`** | Mark functions that must not throw. |
| **E.16: Destructors, `delete`, `swap` must not fail** | Throwing from these breaks invariants. |
| **Order handlers specific-to-general** | Derived types before base types. |
| **`catch (...)` last** | Catch-all must be the final handler. |
| **Minimize explicit try/catch** | Prefer RAII and let exceptions propagate to appropriate boundaries. |
| **Use `throw;` to rethrow** | Preserves the original exception type. |
| **Do not catch by value** | Slices derived exceptions. |
| **Use exception chaining for context** | `throw_with_nested` / `rethrow_if_nested`. |
| **Ensure exception copy constructors are `noexcept`** | E.16 note. |

### Annotated Code Example

```cpp
#include <iostream>
#include <stdexcept>
#include <memory>

void process() noexcept {                  // E.12 / F.6: no-throw guarantee
    try {
        auto resource = std::make_unique<int>(42);
        throw std::runtime_error("failure");
    } catch (...) {
        // log and terminate or swallow; no exception escapes
        std::cerr << "process failed\n";
        // In a noexcept function, swallowing is one option;
        // another is std::terminate().
    }
}

int main() {
    try {
        process();
    } catch (const std::exception& e) {    // E.15: catch by const reference
        std::cout << "Main: " << e.what() << '\n';
    }
}
```

**Expected Output**
```
process failed
```

**Why this output?**  
`process` is `noexcept`; it catches all exceptions internally and logs, preventing termination. The `catch (const std::exception&)` in `main` is present for other potentially throwing calls but is not triggered here.

### Real-World Cases

- **Code reviews** – checking for catch-by-value and throwing destructors.
- **Static analysis** – clang-tidy `misc-throw-by-value-catch-by-reference` enforces ERR61-CPP.
- **API design** – documenting `noexcept` guarantees and exception safety levels.

---

## References

- try-block — cppreference – https://en.cppreference.com/w/cpp/language/try
- Handling exceptions (catch) — cppreference – https://en.cppreference.com/w/cpp/language/catch
- C++ Core Guidelines — E.15: Throw by value, catch exceptions from a hierarchy by reference – https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Re-exception-ref
- C++ Core Guidelines — E.12: Use `noexcept` when exiting a function because of a `throw` is impossible or unacceptable – https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Re-noexcept
- C++ Core Guidelines — E.16: Destructors, deallocation, `swap`, and exception type copy/move construction must never fail – https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Re-never-fail
- SEI CERT C++ — ERR61-CPP: Catch exceptions by lvalue reference – https://wiki.sei.cmu.edu/confluence/display/cplusplus/ERR61-CPP.+Catch+exceptions+by+lvalue+reference
- `std::throw_with_nested` — cppreference – https://en.cppreference.com/w/cpp/error/throw_with_nested
- `std::rethrow_if_nested` — cppreference – https://en.cppreference.com/w/cpp/error/rethrow_if_nested
- `std::current_exception` — cppreference – https://en.cppreference.com/w/cpp/error/current_exception
- `std::rethrow_exception` — cppreference – https://en.cppreference.com/w/cpp/error/rethrow_exception
- `std::exception::what` — cppreference – https://en.cppreference.com/w/cpp/error/exception/what
- clang-tidy: `misc-throw-by-value-catch-by-reference` — LLVM – https://clang.llvm.org/extra/clang-tidy/checks/misc/throw-by-value-catch-by-reference.html
- Use custom exception classes in a hierarchy with `std::exception` base — Dr-Sergey/learn_cpp – https://github.com/Dr-Sergey/learn_cpp/blob/08279578ce23b1cbf2d3394357620600db618eca/11_Error_Handling/Use_custom_exception_classes_in_a_hierarchy_with_stdexception_base.md

---

This cheat sheet provides a comprehensive, structured reference for C++ try-catch, covering the `try` block, `catch` block, multiple catch blocks, catch-all, nested try-catch, re-throwing, exception chaining, exception messages, and best practices, with annotated code examples, expected outputs, real-world cases, and authoritative references.