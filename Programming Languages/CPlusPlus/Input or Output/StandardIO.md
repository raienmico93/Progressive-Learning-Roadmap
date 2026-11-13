# C++ Standard I/O — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
C++ Standard I/O is the part of the C++ Standard Library that provides facilities for reading from and writing to external devices (such as the console, files, and strings) using *streams*.

**Technical Definition**
The C++ I/O library is built around a hierarchy of class templates: `std::basic_streambuf` manages the actual character sequences, while `std::basic_istream`, `std::basic_ostream`, and `std::basic_iostream` provide high-level formatted and unformatted input/output operations. The header `<iostream>` declares eight predefined stream objects (`cin`, `cout`, `cerr`, `clog`, and their wide-character counterparts) that are tied to the C standard streams `stdin`, `stdout`, and `stderr`. Formatting is controlled through *format flags* (`fmtflags`) and *manipulators* — helper functions that modify stream state via `operator<<` or `operator>>`.

**Beginner-Friendly Explanation**
Think of a *stream* as a flow of characters between your program and the outside world. `std::cout` is the flow going *out* to your screen; `std::cin` is the flow coming *in* from your keyboard. You can send data into an output stream with `<<` and receive data from an input stream with `>>`. Along the way, you can use small helper tools called *manipulators* to control how numbers look (e.g., how many decimal places, which base) and how text is aligned.

### Key Characteristics

- **Type-safe** — the compiler checks that you are not inserting an incompatible type.
- **Extensible** — you can define `operator<<` and `operator>>` for your own types.
- **Buffered** — most streams buffer data for efficiency; `cerr` and `wcerr` are unbuffered (or unit-buffered).
- **Stateful** — every stream carries an error state (`goodbit`, `failbit`, `badbit`, `eofbit`) that must be checked.
- **Locale-aware** — formatting can be customised through locales (e.g., decimal separators).
- **Wide-character support** — parallel `wchar_t` streams (`wcin`, `wcout`, etc.) are available.

### Prerequisites

- Basic C++ syntax (variables, operators, `#include`).
- Understanding of fundamental types (`int`, `double`, `char`, `bool`, `std::string`).
- Familiarity with the `main` function and console output.

### Related Programming Areas

- **File I/O** — `std::ifstream`, `std::ofstream`, `std::fstream`.
- **String streams** — `std::istringstream`, `std::ostringstream`.
- **Locale and internationalisation** — number formatting, currency, dates.
- **Serialization** — writing/reading object state.
- **Performance optimisation** — buffering, `sync_with_stdio`.

### Core Concepts / Features

1. Standard stream objects (`cin`, `cout`, `cerr`, `clog`, `wcin`, `wcout`, `wcerr`, `wclog`)
2. Stream operators (`<<`, `>>`)
3. Stream state (`goodbit`, `failbit`, `badbit`, `eofbit`)
4. Stream formatting (format flags)
5. I/O manipulators (each listed separately)
6. I/O best practices

---

## 1. `std::cin`

### Definitions

**Core Definition**
`std::cin` is the predefined global object of type `std::istream` that reads formatted input from the standard C input stream, `stdin`.

**Technical Definition**
`std::cin` is declared in `<iostream>` as `extern std::istream cin;`. It controls input from a stream buffer associated with the C standard stream `stdin`. After initialisation, `cin.tie()` returns `&cout`, meaning `cout` is flushed before `cin` performs input.

**Beginner-Friendly Explanation**
`std::cin` is how your program reads typed input from the keyboard. You use `>>` to pull values (numbers, words) out of it.

### Purposes

- To read formatted input (numbers, strings) from the console.
- To interact with the user at runtime.
- To parse simple textual data.
- To serve as the default input stream in console applications.

### Syntax Rules and Structure

**General Syntax**
```cpp
#include <iostream>
std::cin >> variable;
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `std::cin` | The input stream object. |
| `>>` | Extraction operator. |
| `variable` | The object to receive the extracted value. |

**Syntax Rules**
- `>>` skips leading whitespace by default (controlled by the `skipws` flag).
- Reading stops at whitespace for `std::string` and at the first invalid character for numeric types.
- The stream state must be checked after extraction.

**Constraints and Limitations**
- `cin` is synchronised with `stdio` by default, which can be slow; `std::ios_base::sync_with_stdio(false)` can disable synchronisation.
- Mixing `>>` and `getline` requires care because `>>` leaves the newline character in the buffer.

### Annotated Code Examples

**Example 1: Reading an integer and a string**

```cpp
#include <iostream>
#include <string>

int main() {
    int age;
    std::string name;
    std::cout << "Enter name and age: ";
    std::cin >> name >> age;          // extraction
    std::cout << name << " is " << age << " years old.\n";
}
```

**Expected Output** (interactive input: `Alice 30`)
```
Enter name and age: Alice is 30 years old.
```

**Why this output?**  
`>>` extracts `Alice` into `name` (stops at space) and `30` into `age`.

**Example 2: Checking stream state**

```cpp
#include <iostream>

int main() {
    int x;
    if (std::cin >> x) {
        std::cout << "Read: " << x << '\n';
    } else {
        std::cout << "Invalid input\n";
    }
}
```

**Expected Output** (input: `abc`)
```
Invalid input
```

**Why this output?**  
Extraction fails because `abc` is not an `int`; `cin` enters the fail state.

### Real-World Cases

- **Command-line utilities** — reading user commands.
- **Interactive applications** — prompts and responses.
- **Simple data entry** — numeric input in educational programs.

### References

- `std::cin` — https://en.cppreference.com/w/cpp/io/cin
- `<iostream>` — https://en.cppreference.com/w/cpp/header/iostream

---

## 2. `std::cout`

### Definitions

**Core Definition**
`std::cout` is the predefined global object of type `std::ostream` that writes formatted output to the standard C output stream, `stdout`.

**Technical Definition**
`std::cout` is declared as `extern std::ostream cout;`. It controls output to a stream buffer associated with `stdout`. It is buffered for efficiency.

**Beginner-Friendly Explanation**
`std::cout` is how your program prints text and numbers to the screen. You use `<<` to send data to it.

### Purposes

- To display results to the user.
- To produce formatted console output.
- To debug by printing values.
- To serve as the default output stream.

### Syntax Rules and Structure

**General Syntax**
```cpp
std::cout << expression;
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `std::cout` | The output stream object. |
| `<<` | Insertion operator. |
| `expression` | A value to be formatted and written. |

**Syntax Rules**
- `<<` returns a reference to the stream, allowing chaining: `cout << a << b;`.
- The output is buffered; it may not appear immediately unless the buffer is flushed.
- Formatting flags (e.g., `std::hex`, `std::fixed`) affect subsequent output.

**Constraints and Limitations**
- `cout` is not automatically flushed on every insertion (except for `cerr`).
- Using `std::endl` flushes the buffer, which can be inefficient.

### Annotated Code Examples

**Example 1: Basic output**

```cpp
#include <iostream>

int main() {
    std::cout << "Hello, world!\n";
    int x = 42;
    std::cout << "x = " << x << '\n';
}
```

**Expected Output**
```
Hello, world!
x = 42
```

**Why this output?**  
Each `<<` sends the literal or variable to the output stream.

**Example 2: Chaining and formatting**

```cpp
#include <iostream>
#include <iomanip>

int main() {
    std::cout << std::hex << 255 << '\n';          // hexadecimal
    std::cout << std::fixed << std::setprecision(2) << 3.14159 << '\n';
}
```

**Expected Output**
```
ff
3.14
```

**Why this output?**  
`std::hex` switches to hexadecimal; `std::fixed` + `setprecision(2)` forces two decimal places.

### Real-World Cases

- **Console applications** — menus, reports, prompts.
- **Logging** — `std::cout` for informational messages.
- **Data presentation** — tables, formatted numbers.

### References

- `std::cout` — https://en.cppreference.com/w/cpp/io/cout
- `<iostream>` — https://en.cppreference.com/w/cpp/header/iostream

---

## 3. `std::cerr`

### Definitions

**Core Definition**
`std::cerr` is the predefined global object of type `std::ostream` that writes unbuffered output to the standard C error stream, `stderr`.

**Technical Definition**
`std::cerr` is declared as `extern std::ostream cerr;`. It controls output to a stream buffer associated with `stderr`. After initialisation, `cerr.flags() & unitbuf` is nonzero, meaning it is flushed after each output operation. It is intended for error messages.

**Beginner-Friendly Explanation**
`std::cerr` is like `std::cout`, but it is meant for error messages. It prints immediately (unbuffered), so you see errors right away, even if the program crashes.

### Purposes

- To output error and diagnostic messages.
- To ensure messages appear immediately (unbuffered).
- To separate error output from normal output.
- To support redirection of errors independently from standard output.

### Syntax Rules and Structure

**General Syntax**
```cpp
std::cerr << expression;
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `std::cerr` | The unbuffered error stream. |
| `<<` | Insertion operator (works identically to `cout`). |

**Syntax Rules**
- `cerr` is unit-buffered (`unitbuf`), so each insertion flushes.
- It is tied to `cout` by default (`cerr.tie() == &cout`), so `cout` is flushed before `cerr` output.
- Formatting flags apply to `cerr` just as to `cout`.

**Constraints and Limitations**
- Because it is unbuffered, `cerr` can be slower than `cout` for large volumes of output.
- Using `cerr` for non-error messages is considered poor style.

### Annotated Code Examples

**Example 1: Error message**

```cpp
#include <iostream>

int main() {
    std::cerr << "Error: file not found\n";
    std::cout << "This is normal output\n";
}
```

**Expected Output**
```
Error: file not found
This is normal output
```

**Why this output?**  
`cerr` writes to the error stream (unbuffered), `cout` to standard output.

**Example 2: Redirecting `cerr`**

```cpp
// On a Unix-like shell:
// ./program 2> errors.txt
// cerr output goes to errors.txt, cout to the terminal.
```

**Expected Output**  
(No direct console output; `cerr` is redirected to a file.)

**Why this output?**  
`cerr` is associated with `stderr`, which can be redirected independently.

### Real-World Cases

- **Error reporting** — invalid input, file-open failures.
- **Debugging** — printing diagnostic information.
- **CLI tools** — separating errors from normal output.

### References

- `std::cerr` — https://en.cppreference.com/w/cpp/io/cerr
- `<iostream>` — https://en.cppreference.com/w/cpp/header/iostream

---

## 4. `std::clog`

### Definitions

**Core Definition**
`std::clog` is the predefined global object of type `std::ostream` that writes buffered output to the standard C error stream, `stderr`.

**Technical Definition**
`std::clog` is declared as `extern std::ostream clog;`. Unlike `cerr`, `clog` is **fully buffered**; output is written to the external device when the buffer is full or when the stream is flushed. It is intended for logging.

**Beginner-Friendly Explanation**
`std::clog` is like `cerr` but with buffering. It is more efficient for writing many log messages, but messages may not appear instantly.

### Purposes

- To output log messages efficiently.
- To reduce the overhead of frequent flushing.
- To separate logging from error output.
- To support redirection of log messages.

### Syntax Rules and Structure

**General Syntax**
```cpp
std::clog << expression;
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `std::clog` | The buffered error/log stream. |
| `<<` | Insertion operator. |

**Syntax Rules**
- `clog` is fully buffered, unlike `cerr`.
- It is tied to `cout` by default.
- Flushing can be forced with `std::flush` or `std::endl`.

**Constraints and Limitations**
- Messages may not appear immediately, which can be problematic for real-time diagnostics.
- Buffering behaviour may vary across implementations.

### Annotated Code Examples

**Example 1: Logging with `clog`**

```cpp
#include <iostream>

int main() {
    std::clog << "Log entry 1\n";
    std::clog << "Log entry 2\n";
    // buffer may not be flushed yet
    std::clog << std::flush;          // force output
}
```

**Expected Output** (after flush)
```
Log entry 1
Log entry 2
```

**Why this output?**  
`clog` buffers until flushed; `std::flush` forces the buffer to be written.

**Example 2: Performance comparison**

```cpp
#include <iostream>

int main() {
    for (int i = 0; i < 1000; ++i) {
        std::clog << i << '\n';      // buffered, efficient
        // std::cerr << i << '\n';   // unbuffered, slower
    }
}
```

**Expected Output**  
(Numbers appear in chunks as the buffer fills.)

**Why this output?**  
Buffering reduces the number of system calls.

### Real-World Cases

- **Application logging** — writing diagnostic logs to a file.
- **Server software** — high-volume logging.
- **Debug builds** — verbose logging with minimal overhead.

### References

- `std::clog` — https://en.cppreference.com/w/cpp/io/clog
- `<iostream>` — https://en.cppreference.com/w/cpp/header/iostream

---

## 5. `std::wcin`

### Definitions

**Core Definition**
`std::wcin` is the predefined global object of type `std::wistream` that reads wide-character input from the standard input stream.

**Technical Definition**
`std::wcin` is declared as `extern std::wistream wcin;`. It controls input from a stream buffer associated with `stdin`, using `wchar_t` characters. After initialisation, `wcin.tie()` returns `&wcout`.

**Beginner-Friendly Explanation**
`std::wcin` is the wide-character version of `cin`. It reads wide characters, which can represent Unicode characters beyond the basic ASCII range.

### Purposes

- To read wide-character input.
- To support internationalisation and Unicode text.
- To process text that requires characters larger than `char`.
- To provide a consistent interface with narrow streams.

### Syntax Rules and Structure

**General Syntax**
```cpp
#include <iostream>
std::wcin >> variable;   // variable of wide-character or compatible type
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `std::wcin` | The wide input stream. |
| `>>` | Extraction operator for `wchar_t`-based types. |

**Syntax Rules**
- The variable must be of a type compatible with `wchar_t` (e.g., `wchar_t`, `std::wstring`).
- Mixing narrow and wide I/O requires care; typically you should choose one mode.

**Constraints and Limitations**
- Wide-character support depends on the locale and platform.
- Not all terminals handle wide characters correctly.

### Annotated Code Examples

**Example 1: Reading a wide string**

```cpp
#include <iostream>
#include <string>

int main() {
    std::wstring name;
    std::wcout << L"Enter name: ";
    std::wcin >> name;
    std::wcout << L"Hello, " << name << L"!\n";
}
```

**Expected Output** (interactive input: `世界`)
```
Enter name: Hello, 世界!
```

**Why this output?**  
`wcin` reads wide characters into `wstring`, and `wcout` prints them.

**Example 2: Wide integer input**

```cpp
#include <iostream>

int main() {
    int x;
    std::wcin >> x;
    std::wcout << L"You entered: " << x << L'\n';
}
```

**Expected Output** (input: `42`)
```
You entered: 42
```

**Why this output?**  
`wcin` extracts an `int` just like `cin`, but through a wide stream.

### Real-World Cases

- **Internationalised applications** — user interfaces in multiple languages.
- **Text processing** — handling Unicode documents.
- **Games** — wide-character text rendering.

### References

- `std::wcin` — https://en.cppreference.com/w/cpp/io/wcin
- Wide stream objects — https://timsong-cpp.github.io/cppwp/n3337/wide.stream.objects

---

## 6. `std::wcout`

### Definitions

**Core Definition**
`std::wcout` is the predefined global object of type `std::wostream` that writes wide-character output to the standard output stream.

**Technical Definition**
`std::wcout` is declared as `extern std::wostream wcout;`. It controls output to a stream buffer associated with `stdout`, using `wchar_t` characters.

**Beginner-Friendly Explanation**
`std::wcout` is the wide-character version of `cout`. You use it to print wide strings and wide characters.

### Purposes

- To output wide-character text.
- To support Unicode output.
- To provide symmetry with `wcin`.
- To enable locale-aware formatting for wide characters.

### Syntax Rules and Structure

**General Syntax**
```cpp
std::wcout << expression;
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `std::wcout` | The wide output stream. |
| `<<` | Insertion operator for wide-character data. |

**Syntax Rules**
- Use `L"..."` literals for wide strings.
- Wide and narrow output should not be mixed arbitrarily.

**Constraints and Limitations**
- Terminal support for wide characters varies.
- Performance may differ from narrow streams.

### Annotated Code Examples

**Example 1: Wide string output**

```cpp
#include <iostream>

int main() {
    std::wcout << L"Hello, wide world!\n";
}
```

**Expected Output**
```
Hello, wide world!
```

**Why this output?**  
`wcout` writes the wide string literal to standard output.

**Example 2: Wide numeric formatting**

```cpp
#include <iostream>
#include <iomanip>

int main() {
    std::wcout << std::fixed << std::setprecision(3) << 2.71828 << L'\n';
}
```

**Expected Output**
```
2.718
```

**Why this output?**  
Manipulators work with wide streams as well.

### Real-World Cases

- **Unicode console applications** — displaying multilingual text.
- **Wide-character logging** — logs containing non-ASCII characters.
- **Localised software** — user-facing messages in native scripts.

### References

- `std::wcout` — https://en.cppreference.com/w/cpp/io/wcout
- Wide stream objects — https://timsong-cpp.github.io/cppwp/n3337/wide.stream.objects

---

## 7. `std::wcerr`

### Definitions

**Core Definition**
`std::wcerr` is the predefined global object of type `std::wostream` that writes unbuffered wide-character output to the standard error stream.

**Technical Definition**
`std::wcerr` is declared as `extern std::wostream wcerr;`. It is associated with `stderr` and is unit-buffered (`unitbuf` flag set) and tied to `wcout`.

**Beginner-Friendly Explanation**
`std::wcerr` is the wide-character version of `cerr`. It is used for wide-character error messages and flushes immediately.

### Purposes

- To output wide-character error messages.
- To ensure immediate visibility of errors.
- To separate wide-character errors from normal output.
- To support internationalised error reporting.

### Syntax Rules and Structure

**General Syntax**
```cpp
std::wcerr << expression;
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `std::wcerr` | The wide error stream (unbuffered). |

**Syntax Rules**
- Works like `cerr` but with `wchar_t` data.
- Unit-buffered by default.

**Constraints and Limitations**
- Wide-character error output may not be supported on all platforms.

### Annotated Code Examples

**Example 1: Wide error message**

```cpp
#include <iostream>

int main() {
    std::wcerr << L"Error: invalid input\n";
}
```

**Expected Output**
```
Error: invalid input
```

**Why this output?**  
`wcerr` writes the wide error message immediately.

**Example 2: Wide and narrow errors**

```cpp
#include <iostream>

int main() {
    std::cerr << "Narrow error\n";
    std::wcerr << L"Wide error\n";
}
```

**Expected Output**
```
Narrow error
Wide error
```

**Why this output?**  
Both error streams write to `stderr`, but with different character widths.

### Real-World Cases

- **Localised error messages** — errors in the user’s language.
- **Wide-character debugging** — diagnostics involving Unicode data.

### References

- `std::wcerr` — https://en.cppreference.com/w/cpp/io/wcerr
- Wide stream objects — https://timsong-cpp.github.io/cppwp/n3337/wide.stream.objects

---

## 8. `std::wclog`

### Definitions

**Core Definition**
`std::wclog` is the predefined global object of type `std::wostream` that writes buffered wide-character output to the standard error stream.

**Technical Definition**
`std::wclog` is declared as `extern std::wostream wclog;`. It is associated with `stderr` and is fully buffered.

**Beginner-Friendly Explanation**
`std::wclog` is the wide-character version of `clog`. It is used for wide-character logging and is buffered for efficiency.

### Purposes

- To output wide-character log messages.
- To reduce flushing overhead.
- To separate wide logging from errors.
- To support internationalised logging.

### Syntax Rules and Structure

**General Syntax**
```cpp
std::wclog << expression;
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `std::wclog` | The wide log stream (buffered). |

**Syntax Rules**
- Fully buffered; use `std::flush` to force output.
- Works with wide-character data.

**Constraints and Limitations**
- Buffering may delay log visibility.

### Annotated Code Examples

**Example 1: Wide logging**

```cpp
#include <iostream>

int main() {
    std::wclog << L"Log: started\n";
    std::wclog << std::flush;
}
```

**Expected Output**
```
Log: started
```

**Why this output?**  
`wclog` buffers until flushed.

**Example 2: Performance-oriented wide logging**

```cpp
#include <iostream>

int main() {
    for (int i = 0; i < 100; ++i)
        std::wclog << L"Entry " << i << L'\n';
    // buffer flushed at program exit or when full
}
```

**Expected Output**  
(Log entries appear in batches.)

**Why this output?**  
Buffering reduces the number of writes.

### Real-World Cases

- **Wide-character application logs** — logs with Unicode content.
- **High-volume logging** — efficient wide logging.

### References

- `std::wclog` — https://en.cppreference.com/w/cpp/io/wclog
- Wide stream objects — https://timsong-cpp.github.io/cppwp/n3337/wide.stream.objects

---

## 9. Stream Operators

### Definitions

**Core Definition**
Stream operators are the overloaded `<<` (insertion) and `>>` (extraction) operators used to write to and read from stream objects.

**Technical Definition**
`operator<<` for output streams is a member or non-member function that formats an object and inserts the resulting characters into the stream. `operator>>` for input streams extracts characters, parses them according to the target type, and stores the result. Both operators return a reference to the stream to enable chaining.

**Beginner-Friendly Explanation**
`<<` means “put into” for output (like arrows pointing into the stream), and `>>` means “take out of” for input. You can chain them: `cout << a << b;`.

### Purposes

- To provide a uniform syntax for formatted I/O.
- To enable chaining of insertions/extractions.
- To allow user-defined types to participate in I/O.
- To support type-safe conversion between text and objects.

### Syntax Rules and Structure

**General Syntax**
```cpp
stream << value;       // output
stream >> variable;    // input
```

**Component Breakdown**

| Operator | Direction | Example |
|----------|-----------|---------|
| `<<` | Output (insertion) | `std::cout << 42;` |
| `>>` | Input (extraction) | `std::cin >> x;` |

**Syntax Rules**
- Both operators are left-associative and return the stream reference.
- `>>` skips leading whitespace by default.
- User-defined overloads must follow the same signature pattern: `std::ostream& operator<<(std::ostream&, const T&)`.

**Constraints and Limitations**
- Overloading `<<` or `>>` for built-in types is not allowed.
- The operators are not inherently thread-safe.

### Annotated Code Examples

**Example 1: Chaining output**

```cpp
#include <iostream>

int main() {
    int a = 1, b = 2;
    std::cout << "a = " << a << ", b = " << b << '\n';
}
```

**Expected Output**
```
a = 1, b = 2
```

**Why this output?**  
Each `<<` returns `cout`, so the chain continues.

**Example 2: Chaining input**

```cpp
#include <iostream>

int main() {
    int x, y;
    std::cin >> x >> y;
    std::cout << "Sum: " << x + y << '\n';
}
```

**Expected Output** (input: `3 4`)
```
Sum: 7
```

**Why this output?**  
`>>` returns `cin`, allowing successive extractions.

### Real-World Cases

- **All formatted I/O** — every `cout <<` and `cin >>` uses these operators.
- **Custom type I/O** — defining `operator<<` for a `Point` class.

### References

- `operator<<` — https://en.cppreference.com/w/cpp/io/basic_ostream/operator_ltlt
- `operator>>` — https://en.cppreference.com/w/cpp/io/basic_istream/operator_gtgt

---

## 10. Stream State

### Definitions

**Core Definition**
Stream state is the set of error flags that indicate the current condition of a stream.

**Technical Definition**
Each stream maintains an `iostate` bitmask composed of `goodbit`, `badbit`, `failbit`, and `eofbit`. These flags can be queried with `rdstate()`, `good()`, `eof()`, `fail()`, and `bad()`, and cleared with `clear()`.

**Beginner-Friendly Explanation**
A stream is like a pipe that can get clogged. The state flags tell you whether the last operation succeeded or whether something went wrong (e.g., you tried to read a number but got letters).

### Purposes

- To detect and handle I/O errors.
- To control loops that read until end-of-file.
- To recover from failed operations.
- To ensure robust input validation.

### Syntax Rules and Structure

**General Syntax**
```cpp
stream.rdstate();       // returns iostate
stream.good();          // true if no error
stream.eof();           // true if end-of-file reached
stream.fail();          // true if failbit or badbit set
stream.bad();           // true if badbit set
stream.clear();         // reset state to goodbit
```

**Component Breakdown**

| Flag | Meaning |
|------|---------|
| `goodbit` | No error (value 0). |
| `eofbit` | End of input reached. |
| `failbit` | Logical error (e.g., type mismatch). |
| `badbit` | Irrecoverable stream error. |

**Syntax Rules**
- `fail()` returns true if `failbit` **or** `badbit` is set.
- `eof()` returns true if `eofbit` is set.
- `clear()` can reset specific flags or all flags.
- Once `failbit` is set, further input operations fail until cleared.

**Constraints and Limitations**
- `eofbit` alone does not mean failure; it is normal at end of input.
- Checking only `eof()` in a loop can lead to reading an extra time; check the extraction result instead.

### Annotated Code Examples

**Example 1: Detecting end-of-file**

```cpp
#include <iostream>

int main() {
    int x;
    while (std::cin >> x) {          // checks fail state
        std::cout << x << ' ';
    }
    if (std::cin.eof()) std::cout << "\nEnd of input\n";
}
```

**Expected Output** (input: `1 2 3` then EOF)
```
1 2 3 
End of input
```

**Why this output?**  
The loop stops when `>>` fails (at EOF); `eof()` confirms the cause.

**Example 2: Clearing a failed state**

```cpp
#include <iostream>
#include <limits>

int main() {
    int x;
    std::cin >> x;                  // suppose input is "abc"
    if (std::cin.fail()) {
        std::cin.clear();                           // reset state
        std::cin.ignore(std::numeric_limits<std::streamsize>::max(), '\n'); // discard bad input
        std::cout << "Recovered\n";
    }
}
```

**Expected Output**
```
Recovered
```

**Why this output?**  
`clear()` resets the flags; `ignore` skips the invalid input.

### Real-World Cases

- **Robust input handling** — recovering from invalid user input.
- **File processing** — reading until EOF.
- **Network I/O** — detecting connection failures.

### References

- `std::ios_base::iostate` — https://en.cppreference.com/w/cpp/io/ios_base/iostate
- `std::basic_ios::rdstate` — https://en.cppreference.com/w/cpp/io/basic_ios/rdstate

---

## 11. Stream Formatting

### Definitions

**Core Definition**
Stream formatting is the process of controlling how values are converted to or from textual representation using format flags and manipulators.

**Technical Definition**
Formatting is governed by the `fmtflags` bitmask, which includes flags for base (`dec`, `hex`, `oct`), floating-point notation (`fixed`, `scientific`), alignment (`left`, `right`, `internal`), and various presentation options (`boolalpha`, `showbase`, `showpoint`, `showpos`, `uppercase`). Manipulators are helper functions that modify these flags.

**Beginner-Friendly Explanation**
Formatting lets you decide whether numbers are printed in decimal or hex, how many decimal places a `double` has, whether booleans print as `true`/`false` or `1`/`0`, and how text is aligned in a field.

### Purposes

- To produce human-readable output.
- To meet data interchange formats (e.g., hexadecimal).
- To align tabular data.
- To control precision and notation.

### Syntax Rules and Structure

**General Syntax**
```cpp
stream << std::manipulator;   // for output
stream >> std::manipulator;   // for input (some manipulators)
```

**Component Breakdown**

| Category | Manipulators |
|----------|--------------|
| Base | `std::hex`, `std::dec`, `std::oct`, `std::setbase` |
| Float notation | `std::fixed`, `std::scientific` |
| Precision | `std::setprecision` |
| Width/fill | `std::setw`, `std::setfill` |
| Boolean | `std::boolalpha`, `std::noboolalpha` |
| Base prefix | `std::showbase`, `std::noshowbase` |
| Decimal point | `std::showpoint`, `std::noshowpoint` |
| Sign | `std::showpos`, `std::noshowpos` |
| Case | `std::uppercase`, `std::nouppercase` |
| Alignment | `std::left`, `std::right`, `std::internal` |
| Flush | `std::endl`, `std::flush`, `std::ends` |

**Syntax Rules**
- Manipulators without arguments are functions taking `ios_base&`.
- Manipulators with arguments (e.g., `setw`) are function templates returning an unspecified type.
- Some manipulators affect only the **next** output operation (`setw`); others persist (`hex`, `fixed`).

**Constraints and Limitations**
- Format flags are **sticky** — they remain in effect until changed.
- `setw` must be applied before each formatted output if a width is desired.

---

## 12. `std::endl`

### Definitions

**Core Definition**
`std::endl` is a manipulator that inserts a newline character and flushes the output stream.

**Technical Definition**
`std::endl` is a function template that, when inserted into an output stream, calls `os.put(os.widen('\n'))` followed by `os.flush()`.

**Beginner-Friendly Explanation**
`std::endl` means “end line and flush”. It moves to the next line and forces any buffered output to be written immediately.

### Purposes

- To end a line of output.
- To flush the stream buffer.
- To ensure output appears promptly.
- To provide a convenient line terminator.

### Syntax Rules and Structure

**General Syntax**
```cpp
stream << std::endl;
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `std::endl` | Newline + flush manipulator. |

**Syntax Rules**
- `endl` flushes, which can be inefficient in loops.
- It is equivalent to `<< '\n' << std::flush`.

**Constraints and Limitations**
- Frequent use of `endl` can degrade performance.
- Prefer `'\n'` when flushing is not needed.

### Annotated Code Examples

**Example 1: Basic `endl`**

```cpp
#include <iostream>

int main() {
    std::cout << "Line 1" << std::endl;
    std::cout << "Line 2" << std::endl;
}
```

**Expected Output**
```
Line 1
Line 2
```

**Why this output?**  
Each `endl` inserts a newline and flushes.

**Example 2: Performance impact**

```cpp
#include <iostream>

int main() {
    for (int i = 0; i < 1000; ++i)
        std::cout << i << std::endl;   // flushes every iteration — slow
}
```

**Expected Output**  
(Numbers appear one per line, but many flushes occur.)

**Why this output?**  
Flushing after every insertion incurs a system call.

### Real-World Cases

- **Interactive prompts** — ensuring the prompt is visible before input.
- **Debugging** — forcing output to appear before a crash.
- **Simple programs** — readability over performance.

### References

- `std::endl` — https://en.cppreference.com/w/cpp/io/manip/endl

---

## 13. `std::flush`

### Definitions

**Core Definition**
`std::flush` is a manipulator that flushes the output stream’s buffer without inserting any characters.

**Technical Definition**
`std::flush` calls `os.flush()` on the stream, forcing any buffered characters to be written to the underlying device.

**Beginner-Friendly Explanation**
`std::flush` says “write everything now” — it empties the buffer without adding a newline.

### Purposes

- To force buffered output to appear immediately.
- To synchronise output with external events.
- To ensure data is written before reading from the same device.
- To control buffering without adding characters.

### Syntax Rules and Structure

**General Syntax**
```cpp
stream << std::flush;
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `std::flush` | Flush manipulator. |

**Syntax Rules**
- Works on output streams.
- No characters are inserted.
- It is often used after a sequence of insertions.

**Constraints and Limitations**
- Unnecessary flushing can reduce performance.

### Annotated Code Examples

**Example 1: Forcing output**

```cpp
#include <iostream>

int main() {
    std::cout << "Loading";
    std::cout << std::flush;          // force "Loading" to appear
    // ... long computation ...
    std::cout << " done\n";
}
```

**Expected Output**  
`Loading` appears immediately, then `done` later.

**Why this output?**  
`flush` writes the buffer without a newline.

**Example 2: Progress indicator**

```cpp
#include <iostream>

int main() {
    for (int i = 0; i <= 100; i += 10) {
        std::cout << "\rProgress: " << i << '%' << std::flush;
    }
    std::cout << '\n';
}
```

**Expected Output**  
(A progress percentage updates in place.)

**Why this output?**  
`\r` returns the cursor, and `flush` makes each update visible.

### Real-World Cases

- **Progress bars** — updating the same line.
- **Interactive prompts** — showing a prompt before blocking for input.
- **Logging** — forcing log output at critical points.

### References

- `std::flush` — https://en.cppreference.com/w/cpp/io/manip/flush

---

## 14. `std::ends`

### Definitions

**Core Definition**
`std::ends` is a manipulator that inserts a null character (`'\0'`) into an output stream.

**Technical Definition**
`std::ends` outputs a null character. It is typically used when constructing C-style strings in memory (e.g., with `std::ostringstream`) to ensure null termination.

**Beginner-Friendly Explanation**
`std::ends` adds a `'\0'` at the end of output. This is useful when you are building a C string in a string stream.

### Purposes

- To null-terminate a C-style string in a stream.
- To ensure compatibility with C APIs expecting `'\0'`-terminated strings.
- To explicitly mark the end of a string in memory.
- To avoid manual insertion of `'\0'`.

### Syntax Rules and Structure

**General Syntax**
```cpp
stream << std::ends;
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `std::ends` | Null character manipulator. |

**Syntax Rules**
- Inserts `'\0'` (the null character).
- Does **not** flush the stream.
- Works with both narrow and wide streams (inserts `wchar_t()`).

**Constraints and Limitations**
- In normal console output, `'\0'` is invisible.
- Only meaningful when the resulting buffer is used as a C string.

### Annotated Code Examples

**Example 1: Null-terminating a string stream**

```cpp
#include <sstream>
#include <iostream>

int main() {
    std::ostringstream oss;
    oss << "Hello" << std::ends;
    std::string s = oss.str();
    std::cout << s.size() << '\n';    // 6 (including '\0')
}
```

**Expected Output**
```
6
```

**Why this output?**  
`std::ends` appends `'\0'`, increasing the size to 6.

**Example 2: C-style string construction**

```cpp
#include <sstream>
#include <cstring>
#include <iostream>

int main() {
    std::ostringstream oss;
    oss << "data" << std::ends;
    const char* cstr = oss.str().c_str();   // null-terminated
    std::cout << std::strlen(cstr) << '\n'; // 4
}
```

**Expected Output**
```
4
```

**Why this output?**  
`strlen` stops at the null character inserted by `ends`.

### Real-World Cases

- **C interop** — producing null-terminated strings for C functions.
- **String building** — ensuring proper termination in string streams.
- **Legacy APIs** — working with code that expects `'\0'`.

### References

- `std::ends` — https://en.cppreference.com/w/cpp/io/manip/ends

---

## 15. `std::setw`

### Definitions

**Core Definition**
`std::setw` is a manipulator that sets the width of the next formatted output field.

**Technical Definition**
`std::setw(n)` sets the field width to `n` for the next insertion operation. It is defined in `<iomanip>` and returns an object of unspecified type that modifies the stream’s width when inserted.

**Beginner-Friendly Explanation**
`std::setw` tells the stream “make the next thing you print exactly `n` characters wide” (padding with spaces or a fill character if necessary).

### Purposes

- To align output in columns.
- To pad values to a fixed width.
- To format tabular data.
- To control the appearance of numeric fields.

### Syntax Rules and Structure

**General Syntax**
```cpp
stream << std::setw(n) << value;
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `std::setw(n)` | Sets width to `n`. |
| `n` | Number of characters (integer). |

**Syntax Rules**
- `setw` affects **only the next** output operation.
- The default fill character is a space (changeable with `setfill`).
- Alignment is controlled by `left`, `right`, or `internal`.

**Constraints and Limitations**
- If the value is wider than `n`, the width is ignored.
- `setw` is not sticky; it must be reapplied for each field.

### Annotated Code Examples

**Example 1: Column alignment**

```cpp
#include <iostream>
#include <iomanip>

int main() {
    std::cout << std::setw(10) << "Name" << std::setw(5) << "Age" << '\n';
    std::cout << std::setw(10) << "Alice" << std::setw(5) << 30 << '\n';
}
```

**Expected Output**
```
      Name  Age
     Alice   30
```

**Why this output?**  
`setw(10)` pads `"Name"` to 10 characters; `setw(5)` pads `"Age"` to 5.

**Example 2: Right vs left alignment**

```cpp
#include <iostream>
#include <iomanip>

int main() {
    std::cout << std::right << std::setw(8) << 42 << '\n';
    std::cout << std::left << std::setw(8) << 42 << '\n';
}
```

**Expected Output**
```
      42
42      
```

**Why this output?**  
`right` pads on the left; `left` pads on the right.

### Real-World Cases

- **Tables** — aligning columns of data.
- **Reports** — formatting numeric fields.
- **User interfaces** — neat console layout.

### References

- `std::setw` — https://en.cppreference.com/w/cpp/io/manip/setw

---

## 16. `std::setprecision`

### Definitions

**Core Definition**
`std::setprecision` is a manipulator that sets the precision (number of significant digits or decimal places) for floating-point output.

**Technical Definition**
`std::setprecision(n)` sets the `precision` field of the stream. Its exact meaning depends on the float notation: in default (general) format it is the number of significant digits; with `fixed` or `scientific` it is the number of digits after the decimal point.

**Beginner-Friendly Explanation**
`std::setprecision` controls how many digits are shown when printing a `double` or `float`. With `fixed`, it controls decimal places; otherwise it controls significant digits.

### Purposes

- To control the number of decimal places.
- To limit significant digits.
- To format monetary and scientific values.
- To produce consistent numeric output.

### Syntax Rules and Structure

**General Syntax**
```cpp
stream << std::setprecision(n) << value;
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `n` | Precision value. |

**Syntax Rules**
- The effect depends on `floatfield` (`fixed`, `scientific`, or neither).
- `setprecision` is sticky — it persists for subsequent output.
- Include `<iomanip>`.

**Constraints and Limitations**
- With `fixed`, `n` is the number of digits after the decimal point.
- With default notation, `n` is the total number of significant digits.

### Annotated Code Examples

**Example 1: Fixed decimal places**

```cpp
#include <iostream>
#include <iomanip>

int main() {
    double pi = 3.14159265358979;
    std::cout << std::fixed << std::setprecision(3) << pi << '\n';
}
```

**Expected Output**
```
3.142
```

**Why this output?**  
`fixed` + `setprecision(3)` rounds to three decimal places.

**Example 2: Significant digits**

```cpp
#include <iostream>
#include <iomanip>

int main() {
    double x = 12345.6789;
    std::cout << std::setprecision(4) << x << '\n';   // 4 significant digits
}
```

**Expected Output**
```
1.235e+04
```

**Why this output?**  
Default notation uses precision as significant digits; the value is rounded to four significant digits and printed in scientific notation.

### Real-World Cases

- **Financial calculations** — two decimal places.
- **Scientific data** — significant digits.
- **Reports** — consistent numeric formatting.

### References

- `std::setprecision` — https://en.cppreference.com/w/cpp/io/manip/setprecision

---

## 17. `std::setfill`

### Definitions

**Core Definition**
`std::setfill` is a manipulator that sets the fill character used for padding output fields.

**Technical Definition**
`std::setfill(c)` sets the stream’s fill character to `c`. The fill character is used when a value is narrower than the field width set by `setw`.

**Beginner-Friendly Explanation**
`std::setfill` chooses what character to use for padding (default is space). For example, you can pad with `'*'` or `'0'`.

### Purposes

- To customise padding characters.
- To format numeric fields with leading zeros.
- To create decorative output.
- To improve readability of tabular data.

### Syntax Rules and Structure

**General Syntax**
```cpp
stream << std::setfill(c);
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `c` | Fill character (`char` or `wchar_t`). |

**Syntax Rules**
- `setfill` is sticky — it persists.
- Works together with `setw` and alignment manipulators.
- Include `<iomanip>`.

**Constraints and Limitations**
- If no width is set, `setfill` has no effect.

### Annotated Code Examples

**Example 1: Zero-padding**

```cpp
#include <iostream>
#include <iomanip>

int main() {
    std::cout << std::setfill('0') << std::setw(5) << 42 << '\n';
}
```

**Expected Output**
```
00042
```

**Why this output?**  
`setw(5)` requests width 5; `setfill('0')` pads with zeros.

**Example 2: Star padding**

```cpp
#include <iostream>
#include <iomanip>

int main() {
    std::cout << std::setfill('*') << std::setw(10) << "Hi" << '\n';
}
```

**Expected Output**
```
********Hi
```

**Why this output?**  
The field is padded with `*` to width 10.

### Real-World Cases

- **ID formatting** — zero-padded numbers.
- **Decorative headers** — padded titles.
- **Tabular reports** — custom fill for alignment.

### References

- `std::setfill` — https://en.cppreference.com/w/cpp/io/manip/setfill

---

## 18. `std::setbase`

### Definitions

**Core Definition**
`std::setbase` is a manipulator that sets the numeric base for integer input and output.

**Technical Definition**
`std::setbase(base)` sets the `basefield` flag to `dec`, `oct`, or `hex` depending on the argument (8, 10, or 16). If `base` is 0, the base is deduced from the prefix (`0` for octal, `0x`/`0X` for hex).

**Beginner-Friendly Explanation**
`std::setbase` lets you switch between decimal, octal, and hexadecimal with one function, using 8, 10, or 16 as the argument.

### Purposes

- To switch numeric base programmatically.
- To support input/output in octal or hexadecimal.
- To simplify code that toggles bases.
- To allow automatic base detection (`setbase(0)`).

### Syntax Rules and Structure

**General Syntax**
```cpp
stream << std::setbase(base);
```

**Component Breakdown**

| `base` | Effect |
|--------|--------|
| 8 | Octal |
| 10 | Decimal |
| 16 | Hexadecimal |
| 0 | Auto-detect from prefix |

**Syntax Rules**
- `setbase` is sticky.
- For input, `setbase(0)` enables prefix-based detection.
- Include `<iomanip>`.

**Constraints and Limitations**
- Only 8, 10, 16, and 0 are valid; other values set `basefield` to 0 (no base).

### Annotated Code Examples

**Example 1: Switching bases**

```cpp
#include <iostream>
#include <iomanip>

int main() {
    int x = 255;
    std::cout << std::setbase(16) << x << '\n';   // hex
    std::cout << std::setbase(8) << x << '\n';    // octal
    std::cout << std::setbase(10) << x << '\n';   // decimal
}
```

**Expected Output**
```
ff
377
255
```

**Why this output?**  
`setbase` changes the base for integer formatting.

**Example 2: Auto-detection on input**

```cpp
#include <iostream>
#include <iomanip>

int main() {
    int x;
    std::cin >> std::setbase(0) >> x;   // input "0xff" -> 255
    std::cout << x << '\n';
}
```

**Expected Output** (input: `0xff`)
```
255
```

**Why this output?**  
`setbase(0)` enables prefix-based base detection.

### Real-World Cases

- **Configuration files** — hexadecimal values.
- **Debugging** — viewing memory addresses in hex.
- **Low-level programming** — bit masks in hex/octal.

### References

- `std::setbase` — https://en.cppreference.com/w/cpp/io/manip/setbase

---

## 19. `std::fixed`

### Definitions

**Core Definition**
`std::fixed` is a manipulator that sets floating-point output to fixed-point notation.

**Technical Definition**
`std::fixed` sets the `floatfield` to `fixed`, causing floating-point values to be printed in decimal notation with exactly `precision()` digits after the decimal point (default 6).

**Beginner-Friendly Explanation**
`std::fixed` makes floating-point numbers print in normal decimal form (like `3.140000`), not scientific notation.

### Purposes

- To display numbers with a fixed number of decimal places.
- To avoid scientific notation.
- To format monetary values.
- To ensure consistent numeric output.

### Syntax Rules and Structure

**General Syntax**
```cpp
stream << std::fixed;
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `std::fixed` | Fixed notation flag. |

**Syntax Rules**
- Sticky flag.
- Works with `setprecision` to control decimal places.
- Opposite of `scientific`.

**Constraints and Limitations**
- Very large/small numbers may be unwieldy in fixed notation.

### Annotated Code Examples

**Example 1: Fixed notation**

```cpp
#include <iostream>
#include <iomanip>

int main() {
    double x = 123.456789;
    std::cout << std::fixed << std::setprecision(2) << x << '\n';
}
```

**Expected Output**
```
123.46
```

**Why this output?**  
`fixed` forces decimal notation; `setprecision(2)` gives two decimal places.

**Example 2: Default precision**

```cpp
#include <iostream>

int main() {
    std::cout << std::fixed << 3.14 << '\n';
}
```

**Expected Output**
```
3.140000
```

**Why this output?**  
Default precision is 6.

### Real-World Cases

- **Currency** — always two decimal places.
- **Measurements** — fixed decimal precision.
- **Reports** — consistent formatting.

### References

- `std::fixed` — https://en.cppreference.com/w/cpp/io/manip/fixed

---

## 20. `std::scientific`

### Definitions

**Core Definition**
`std::scientific` is a manipulator that sets floating-point output to scientific notation.

**Technical Definition**
`std::scientific` sets `floatfield` to `scientific`, producing output like `1.234568e+02` with `precision()` digits after the decimal point.

**Beginner-Friendly Explanation**
`std::scientific` prints numbers in scientific notation (with an exponent), which is useful for very large or very small values.

### Purposes

- To display very large or very small numbers compactly.
- To show the exponent explicitly.
- To format scientific data.
- To ensure consistent exponent notation.

### Syntax Rules and Structure

**General Syntax**
```cpp
stream << std::scientific;
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `std::scientific` | Scientific notation flag. |

**Syntax Rules**
- Sticky.
- Works with `setprecision`.
- Opposite of `fixed`.

**Constraints and Limitations**
- Less readable for everyday numbers.

### Annotated Code Examples

**Example 1: Scientific notation**

```cpp
#include <iostream>
#include <iomanip>

int main() {
    double x = 12345.6789;
    std::cout << std::scientific << std::setprecision(3) << x << '\n';
}
```

**Expected Output**
```
1.235e+04
```

**Why this output?**  
Scientific notation with three digits after the decimal point.

**Example 2: Small number**

```cpp
#include <iostream>

int main() {
    std::cout << std::scientific << 0.000123 << '\n';
}
```

**Expected Output**
```
1.230000e-04
```

**Why this output?**  
Default precision (6) is used.

### Real-World Cases

- **Physics/engineering** — very small or large quantities.
- **Data logging** — consistent exponent format.
- **Spreadsheets** — scientific display.

### References

- `std::scientific` — https://en.cppreference.com/w/cpp/io/manip/fixed

---

## 21. `std::hex`

### Definitions

**Core Definition**
`std::hex` is a manipulator that sets integer I/O to use hexadecimal (base 16) notation.

**Technical Definition**
`std::hex` sets the `basefield` to `hex`. For output, integers are printed in base 16 using lowercase letters `a`–`f` unless `uppercase` is set.

**Beginner-Friendly Explanation**
`std::hex` makes integers print in hexadecimal (base 16), e.g., `255` becomes `ff`.

### Purposes

- To display integers in hexadecimal.
- To work with bit patterns and memory addresses.
- To read hexadecimal input.
- To format data for low-level programming.

### Syntax Rules and Structure

**General Syntax**
```cpp
stream << std::hex;
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `std::hex` | Hexadecimal flag. |

**Syntax Rules**
- Sticky.
- Affects both input and output for integers.
- `showbase` can add the `0x` prefix.

**Constraints and Limitations**
- Only affects integer types, not floating-point.

### Annotated Code Examples

**Example 1: Hexadecimal output**

```cpp
#include <iostream>

int main() {
    int x = 255;
    std::cout << std::hex << x << '\n';
}
```

**Expected Output**
```
ff
```

**Why this output?**  
`hex` switches the base to 16.

**Example 2: Hexadecimal input**

```cpp
#include <iostream>

int main() {
    int x;
    std::cin >> std::hex >> x;   // input "ff"
    std::cout << std::dec << x << '\n';
}
```

**Expected Output** (input: `ff`)
```
255
```

**Why this output?**  
`hex` interprets `ff` as 255; `dec` switches back for output.

### Real-World Cases

- **Memory addresses** — printing pointers.
- **Bit manipulation** — masks in hex.
- **File formats** — hex dumps.

### References

- `std::hex` — https://en.cppreference.com/w/cpp/io/manip/hex

---

## 22. `std::dec`

### Definitions

**Core Definition**
`std::dec` is a manipulator that sets integer I/O to use decimal (base 10) notation.

**Technical Definition**
`std::dec` sets `basefield` to `dec`, the default base for integers.

**Beginner-Friendly Explanation**
`std::dec` switches back to normal decimal numbers (base 10).

### Purposes

- To restore decimal notation.
- To ensure numbers are printed in base 10.
- To override previous `hex` or `oct` settings.
- To read decimal input.

### Syntax Rules and Structure

**General Syntax**
```cpp
stream << std::dec;
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `std::dec` | Decimal flag. |

**Syntax Rules**
- Sticky.
- Affects integers.

### Annotated Code Examples

**Example 1: Returning to decimal**

```cpp
#include <iostream>

int main() {
    int x = 255;
    std::cout << std::hex << x << std::dec << ' ' << x << '\n';
}
```

**Expected Output**
```
ff 255
```

**Why this output?**  
`hex` then `dec` switches formatting.

**Example 2: Decimal input**

```cpp
#include <iostream>

int main() {
    int x;
    std::cin >> std::dec >> x;   // force decimal interpretation
}
```

**Expected Output**  
(Reads a decimal integer.)

### Real-World Cases

- **General output** — default decimal display.
- **After hexadecimal sections** — restoring normal formatting.

### References

- `std::dec` — https://en.cppreference.com/w/cpp/io/manip/hex

---

## 23. `std::oct`

### Definitions

**Core Definition**
`std::oct` is a manipulator that sets integer I/O to use octal (base 8) notation.

**Technical Definition**
`std::oct` sets `basefield` to `oct`. Integers are printed in base 8.

**Beginner-Friendly Explanation**
`std::oct` makes integers print in octal (base 8), e.g., `8` becomes `10`.

### Purposes

- To display integers in octal.
- To work with Unix file permissions.
- To read octal input.
- To support legacy formats.

### Syntax Rules and Structure

**General Syntax**
```cpp
stream << std::oct;
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `std::oct` | Octal flag. |

**Syntax Rules**
- Sticky.
- Affects integers.

### Annotated Code Examples

**Example 1: Octal output**

```cpp
#include <iostream>

int main() {
    int x = 64;
    std::cout << std::oct << x << '\n';
}
```

**Expected Output**
```
100
```

**Why this output?**  
`64` in base 8 is `100`.

**Example 2: Octal input**

```cpp
#include <iostream>

int main() {
    int x;
    std::cin >> std::oct >> x;   // input "10"
    std::cout << std::dec << x << '\n';
}
```

**Expected Output** (input: `10`)
```
8
```

**Why this output?**  
Octal `10` is decimal `8`.

### Real-World Cases

- **File permissions** — octal notation (e.g., `0755`).
- **Legacy systems** — octal data formats.

### References

- `std::oct` — https://en.cppreference.com/w/cpp/io/manip/hex

---

## 24. `std::boolalpha`

### Definitions

**Core Definition**
`std::boolalpha` is a manipulator that enables textual representation of `bool` values (`true` / `false`).

**Technical Definition**
`std::boolalpha` sets the `boolalpha` flag, causing `bool` values to be inserted/extracted as `"true"` / `"false"` instead of `1` / `0`.

**Beginner-Friendly Explanation**
`std::boolalpha` makes booleans print as the words `true` and `false` instead of `1` and `0`.

### Purposes

- To improve readability of boolean output.
- To allow textual boolean input.
- To avoid ambiguity between `1` and `true`.
- To produce human-friendly logs.

### Syntax Rules and Structure

**General Syntax**
```cpp
stream << std::boolalpha;
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `std::boolalpha` | Boolean text flag. |

**Syntax Rules**
- Sticky.
- Affects both input and output.

### Annotated Code Examples

**Example 1: Boolean output**

```cpp
#include <iostream>

int main() {
    bool b = true;
    std::cout << b << ' ' << std::boolalpha << b << '\n';
}
```

**Expected Output**
```
1 true
```

**Why this output?**  
Without `boolalpha`, `true` prints as `1`; with it, as `true`.

**Example 2: Boolean input**

```cpp
#include <iostream>

int main() {
    bool b;
    std::cin >> std::boolalpha >> b;   // input "true"
    std::cout << b << '\n';
}
```

**Expected Output** (input: `true`)
```
1
```

**Why this output?**  
`boolalpha` allows textual input; output reverts to numeric because `boolalpha` is still set? (Actually `cout` also has `boolalpha` if set on `cin`? No — flags are per-stream. Here `cin` has `boolalpha`; `cout` does not, so `b` prints as `1`.)

### Real-World Cases

- **Configuration flags** — printing `true`/`false`.
- **Debug logs** — readable boolean states.
- **User prompts** — reading `true`/`false` responses.

### References

- `std::boolalpha` — https://en.cppreference.com/w/cpp/io/manip/boolalpha

---

## 25. `std::noboolalpha`

### Definitions

**Core Definition**
`std::noboolalpha` is a manipulator that disables textual boolean representation, reverting to `1`/`0`.

**Technical Definition**
`std::noboolalpha` clears the `boolalpha` flag.

**Beginner-Friendly Explanation**
`std::noboolalpha` switches booleans back to numeric `1` and `0`.

### Purposes

- To restore numeric boolean output.
- To disable textual input.
- To override a previous `boolalpha`.
- To produce compact output.

### Syntax Rules and Structure

**General Syntax**
```cpp
stream << std::noboolalpha;
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `std::noboolalpha` | Clears boolean text flag. |

**Syntax Rules**
- Sticky.
- Default state.

### Annotated Code Examples

**Example 1: Reverting to numeric**

```cpp
#include <iostream>

int main() {
    bool b = true;
    std::cout << std::boolalpha << b << std::noboolalpha << ' ' << b << '\n';
}
```

**Expected Output**
```
true 1
```

**Why this output?**  
`noboolalpha` clears the flag.

**Example 2: Mixed formatting**

```cpp
#include <iostream>

int main() {
    bool x = false;
    std::cout << std::boolalpha << x << std::noboolalpha << ' ' << x << '\n';
}
```

**Expected Output**
```
false 0
```

### Real-World Cases

- **Compact output** — numeric booleans.
- **After textual sections** — restoring default.

### References

- `std::noboolalpha` — https://en.cppreference.com/w/cpp/io/manip/boolalpha

---

## 26. `std::showbase`

### Definitions

**Core Definition**
`std::showbase` is a manipulator that enables the display of a numeric base prefix for integer output.

**Technical Definition**
`std::showbase` sets the `showbase` flag. For hexadecimal, `0x` is prefixed; for octal, `0` is prefixed.

**Beginner-Friendly Explanation**
`std::showbase` adds `0x` before hex numbers and `0` before octal numbers, so you can tell the base at a glance.

### Purposes

- To make the numeric base explicit.
- To produce unambiguous output.
- To support round-trippable text representations.
- To improve readability of hex/octal data.

### Syntax Rules and Structure

**General Syntax**
```cpp
stream << std::showbase;
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `std::showbase` | Base prefix flag. |

**Syntax Rules**
- Sticky.
- Works with `hex`, `oct`, and `dec` (decimal has no prefix).

### Annotated Code Examples

**Example 1: Hex with prefix**

```cpp
#include <iostream>

int main() {
    std::cout << std::showbase << std::hex << 255 << '\n';
}
```

**Expected Output**
```
0xff
```

**Why this output?**  
`showbase` adds `0x`.

**Example 2: Octal with prefix**

```cpp
#include <iostream>

int main() {
    std::cout << std::showbase << std::oct << 64 << '\n';
}
```

**Expected Output**
```
0100
```

**Why this output?**  
Octal prefix `0` is added.

### Real-World Cases

- **Debugging** — clear hex addresses.
- **Data serialization** — self-describing numbers.
- **User output** — explicit bases.

### References

- `std::showbase` — https://en.cppreference.com/w/cpp/io/manip/showbase

---

## 27. `std::noshowbase`

### Definitions

**Core Definition**
`std::noshowbase` is a manipulator that disables numeric base prefixes.

**Technical Definition**
`std::noshowbase` clears the `showbase` flag.

**Beginner-Friendly Explanation**
`std::noshowbase` stops adding `0x` or `0` prefixes.

### Purposes

- To restore plain numeric output.
- To remove prefixes for compactness.
- To override `showbase`.
- To produce clean hexadecimal strings.

### Syntax Rules and Structure

**General Syntax**
```cpp
stream << std::noshowbase;
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `std::noshowbase` | Clears base prefix flag. |

**Syntax Rules**
- Sticky.
- Default state.

### Annotated Code Examples

**Example 1: Removing prefix**

```cpp
#include <iostream>

int main() {
    std::cout << std::showbase << std::hex << 255
              << std::noshowbase << ' ' << 255 << '\n';
}
```

**Expected Output**
```
0xff ff
```

**Why this output?**  
`noshowbase` removes the `0x` for the second value.

**Example 2: Clean hex string**

```cpp
#include <iostream>

int main() {
    std::cout << std::noshowbase << std::hex << 255 << '\n';
}
```

**Expected Output**
```
ff
```

### Real-World Cases

- **Compact output** — hex without prefix.
- **String building** — raw hex digits.

### References

- `std::noshowbase` — https://en.cppreference.com/w/cpp/io/manip/showbase

---

## 28. `std::showpoint`

### Definitions

**Core Definition**
`std::showpoint` is a manipulator that forces the decimal point to always be written for floating-point output, even if there are no fractional digits.

**Technical Definition**
`std::showpoint` sets the `showpoint` flag, causing the decimal point to be included unconditionally.

**Beginner-Friendly Explanation**
`std::showpoint` ensures that a number like `5` is printed as `5.` (or `5.00000` with default precision), making it clear it is a floating-point value.

### Purposes

- To make floating-point nature explicit.
- To ensure a decimal point in all cases.
- To format numeric tables consistently.
- To avoid confusion with integers.

### Syntax Rules and Structure

**General Syntax**
```cpp
stream << std::showpoint;
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `std::showpoint` | Decimal point flag. |

**Syntax Rules**
- Sticky.
- Works with floating-point output.

### Annotated Code Examples

**Example 1: Showing the point**

```cpp
#include <iostream>

int main() {
    double x = 5.0;
    std::cout << x << ' ' << std::showpoint << x << '\n';
}
```

**Expected Output**
```
5 5.00000
```

**Why this output?**  
`showpoint` forces the decimal point; default precision adds zeros.

**Example 2: With precision**

```cpp
#include <iostream>
#include <iomanip>

int main() {
    std::cout << std::showpoint << std::setprecision(2) << 3.0 << '\n';
}
```

**Expected Output**
```
3.0
```

### Real-World Cases

- **Scientific tables** — consistent decimal points.
- **Financial reports** — explicit decimals.
- **User interfaces** — clarity of numeric types.

### References

- `std::showpoint` — https://en.cppreference.com/w/cpp/io/manip/showpoint

---

## 29. `std::noshowpoint`

### Definitions

**Core Definition**
`std::noshowpoint` is a manipulator that allows the decimal point to be omitted when there are no fractional digits.

**Technical Definition**
`std::noshowpoint` clears the `showpoint` flag.

**Beginner-Friendly Explanation**
`std::noshowpoint` lets whole-number floating-point values print without a decimal point.

### Purposes

- To produce compact floating-point output.
- To avoid unnecessary `.0` suffixes.
- To restore default behaviour.
- To clean up numeric display.

### Syntax Rules and Structure

**General Syntax**
```cpp
stream << std::noshowpoint;
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `std::noshowpoint` | Clears decimal point flag. |

**Syntax Rules**
- Sticky.
- Default state.

### Annotated Code Examples

**Example 1: Omitting the point**

```cpp
#include <iostream>

int main() {
    double x = 5.0;
    std::cout << std::showpoint << x
              << std::noshowpoint << ' ' << x << '\n';
}
```

**Expected Output**
```
5.00000 5
```

**Why this output?**  
`noshowpoint` omits the decimal point for the second value.

### Real-World Cases

- **Compact numeric output** — no trailing `.0`.
- **After `showpoint` sections** — restoring default.

### References

- `std::noshowpoint` — https://en.cppreference.com/w/cpp/io/manip/showpoint

---

## 30. `std::showpos`

### Definitions

**Core Definition**
`std::showpos` is a manipulator that forces a plus sign to be written before non-negative numeric output.

**Technical Definition**
`std::showpos` sets the `showpos` flag, causing `+` to be inserted before non-negative numbers.

**Beginner-Friendly Explanation**
`std::showpos` makes positive numbers display an explicit `+` sign, e.g., `+42`.

### Purposes

- To make signs explicit.
- To align signed numbers in tables.
- To emphasise positivity.
- To format mathematical output.

### Syntax Rules and Structure

**General Syntax**
```cpp
stream << std::showpos;
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `std::showpos` | Plus sign flag. |

**Syntax Rules**
- Sticky.
- Affects numeric output.

### Annotated Code Examples

**Example 1: Explicit plus sign**

```cpp
#include <iostream>

int main() {
    int x = 42;
    std::cout << std::showpos << x << '\n';
}
```

**Expected Output**
```
+42
```

**Example 2: Negative numbers**

```cpp
#include <iostream>

int main() {
    int y = -7;
    std::cout << std::showpos << y << '\n';
}
```

**Expected Output**
```
-7
```

**Why this output?**  
`showpos` only adds `+` for non-negative values.

### Real-World Cases

- **Mathematical displays** — explicit signs.
- **Financial data** — positive/negative amounts.
- **Scientific tables** — sign alignment.

### References

- `std::showpos` — https://en.cppreference.com/w/cpp/io/manip/showpos

---

## 31. `std::noshowpos`

### Definitions

**Core Definition**
`std::noshowpos` is a manipulator that disables the explicit plus sign for non-negative numbers.

**Technical Definition**
`std::noshowpos` clears the `showpos` flag.

**Beginner-Friendly Explanation**
`std::noshowpos` stops printing `+` before positive numbers.

### Purposes

- To restore default numeric output.
- To remove plus signs for compactness.
- To override `showpos`.
- To produce standard numeric strings.

### Syntax Rules and Structure

**General Syntax**
```cpp
stream << std::noshowpos;
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `std::noshowpos` | Clears plus sign flag. |

**Syntax Rules**
- Sticky.
- Default state.

### Annotated Code Examples

**Example 1: Removing plus sign**

```cpp
#include <iostream>

int main() {
    int x = 42;
    std::cout << std::showpos << x << std::noshowpos << ' ' << x << '\n';
}
```

**Expected Output**
```
+42 42
```

### Real-World Cases

- **Compact output** — no plus signs.
- **After `showpos` sections** — restoring default.

### References

- `std::noshowpos` — https://en.cppreference.com/w/cpp/io/manip/showpos

---

## 32. `std::uppercase`

### Definitions

**Core Definition**
`std::uppercase` is a manipulator that causes uppercase characters to be used in certain output formats (hexadecimal digits, scientific notation exponent).

**Technical Definition**
`std::uppercase` sets the `uppercase` flag, affecting hex digits (`A`–`F`) and exponent letters (`E` in scientific notation).

**Beginner-Friendly Explanation**
`std::uppercase` makes hex letters and exponent markers uppercase, e.g., `FF` instead of `ff`, `1.0E+2` instead of `1.0e+2`.

### Purposes

- To produce uppercase hexadecimal.
- To format exponents in uppercase.
- To meet style guidelines.
- To distinguish from lowercase data.

### Syntax Rules and Structure

**General Syntax**
```cpp
stream << std::uppercase;
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `std::uppercase` | Uppercase flag. |

**Syntax Rules**
- Sticky.
- Affects `hex` and `scientific` output.

### Annotated Code Examples

**Example 1: Uppercase hex**

```cpp
#include <iostream>

int main() {
    std::cout << std::uppercase << std::hex << 255 << '\n';
}
```

**Expected Output**
```
FF
```

**Example 2: Uppercase scientific**

```cpp
#include <iostream>

int main() {
    std::cout << std::uppercase << std::scientific << 12345.0 << '\n';
}
```

**Expected Output**
```
1.234500E+04
```

### Real-World Cases

- **Hex dumps** — uppercase digits.
- **Scientific reports** — uppercase exponent.

### References

- `std::uppercase` — https://en.cppreference.com/w/cpp/io/manip/uppercase

---

## 33. `std::nouppercase`

### Definitions

**Core Definition**
`std::nouppercase` is a manipulator that reverts to lowercase characters in formatted output.

**Technical Definition**
`std::nouppercase` clears the `uppercase` flag.

**Beginner-Friendly Explanation**
`std::nouppercase` makes hex digits and exponents lowercase again (the default).

### Purposes

- To restore default lowercase formatting.
- To override `uppercase`.
- To produce lowercase hex strings.
- To match specific style requirements.

### Syntax Rules and Structure

**General Syntax**
```cpp
stream << std::nouppercase;
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `std::nouppercase` | Clears uppercase flag. |

**Syntax Rules**
- Sticky.
- Default state.

### Annotated Code Examples

**Example 1: Reverting to lowercase**

```cpp
#include <iostream>

int main() {
    std::cout << std::uppercase << std::hex << 255
              << std::nouppercase << ' ' << 255 << '\n';
}
```

**Expected Output**
```
FF ff
```

### Real-World Cases

- **Lowercase hex** — common in URLs and hashes.
- **After uppercase sections** — restoring default.

### References

- `std::nouppercase` — https://en.cppreference.com/w/cpp/io/manip/uppercase

---

## 34. `std::left`

### Definitions

**Core Definition**
`std::left` is a manipulator that sets left alignment for output fields, padding on the right.

**Technical Definition**
`std::left` sets the `adjustfield` to `left`. When a field width is set, the value is placed at the left and fill characters are added on the right.

**Beginner-Friendly Explanation**
`std::left` makes text line up on the left side of its field, with extra spaces (or fill characters) on the right.

### Purposes

- To left-align text in columns.
- To format tables with left-justified labels.
- To improve readability.
- To control field padding direction.

### Syntax Rules and Structure

**General Syntax**
```cpp
stream << std::left;
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `std::left` | Left adjustment flag. |

**Syntax Rules**
- Sticky.
- Requires a field width (`setw`) to take effect.

### Annotated Code Examples

**Example 1: Left alignment**

```cpp
#include <iostream>
#include <iomanip>

int main() {
    std::cout << std::left << std::setw(10) << "Name" << "Age\n";
    std::cout << std::left << std::setw(10) << "Alice" << 30 << '\n';
}
```

**Expected Output**
```
Name      Age
Alice     30
```

**Why this output?**  
`left` pads on the right.

### Real-World Cases

- **Tables** — left-aligned text columns.
- **Reports** — readable labels.

### References

- `std::left` — https://en.cppreference.com/w/cpp/io/manip/left

---

## 35. `std::right`

### Definitions

**Core Definition**
`std::right` is a manipulator that sets right alignment for output fields, padding on the left.

**Technical Definition**
`std::right` sets `adjustfield` to `right`, placing the value on the right and fill characters on the left.

**Beginner-Friendly Explanation**
`std::right` makes numbers and text line up on the right side of their field.

### Purposes

- To right-align numeric columns.
- To align integers in tables.
- To improve readability of numbers.
- To control padding direction.

### Syntax Rules and Structure

**General Syntax**
```cpp
stream << std::right;
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `std::right` | Right adjustment flag. |

**Syntax Rules**
- Sticky.
- Requires `setw`.

### Annotated Code Examples

**Example 1: Right-aligned numbers**

```cpp
#include <iostream>
#include <iomanip>

int main() {
    std::cout << std::right << std::setw(5) << 1 << '\n'
              << std::setw(5) << 22 << '\n'
              << std::setw(5) << 333 << '\n';
}
```

**Expected Output**
```
    1
   22
  333
```

### Real-World Cases

- **Financial tables** — right-aligned amounts.
- **Numeric reports** — decimal alignment.

### References

- `std::right` — https://en.cppreference.com/w/cpp/io/manip/left

---

## 36. `std::internal`

### Definitions

**Core Definition**
`std::internal` is a manipulator that sets internal alignment, padding between the sign/base prefix and the numeric value.

**Technical Definition**
`std::internal` sets `adjustfield` to `internal`. Fill characters are inserted after any sign or base prefix but before the digits.

**Beginner-Friendly Explanation**
`std::internal` is useful for numbers: it puts padding after the `+`/`-` sign or `0x` prefix, so the digits stay aligned.

### Purposes

- To align signed numbers neatly.
- To pad after base prefixes.
- To improve readability of numeric columns.
- To control padding in formatted output.

### Syntax Rules and Structure

**General Syntax**
```cpp
stream << std::internal;
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `std::internal` | Internal adjustment flag. |

**Syntax Rules**
- Sticky.
- Requires `setw`.

### Annotated Code Examples

**Example 1: Internal padding with sign**

```cpp
#include <iostream>
#include <iomanip>

int main() {
    std::cout << std::internal << std::showpos << std::setw(6) << 42 << '\n'
              << std::setw(6) << -7 << '\n';
}
```

**Expected Output**
```
+   42
-    7
```

**Why this output?**  
Padding is inserted after the sign.

**Example 2: Internal padding with hex prefix**

```cpp
#include <iostream>
#include <iomanip>

int main() {
    std::cout << std::internal << std::showbase << std::hex
              << std::setw(8) << 255 << '\n';
}
```

**Expected Output**
```
0x    ff
```

### Real-World Cases

- **Mathematical tables** — aligned signs.
- **Hexadecimal dumps** — aligned prefixes.

### References

- `std::internal` — https://en.cppreference.com/w/cpp/io/manip/left

---

## 37. I/O Best Practices

### Definitions

**Core Definition**
C++ I/O best practices are guidelines for using iostreams safely, efficiently, and idiomatically.

**Technical Definition**
These practices are codified in the C++ Core Guidelines (SL.io series): prefer iostreams for type safety and extensibility, validate input, avoid `endl` in performance-critical loops, disable `sync_with_stdio` when not using C I/O, and handle stream state errors.

**Beginner-Friendly Explanation**
Best practices are good habits: check for errors, don’t flush unnecessarily, prefer `'\n'` over `endl` in loops, and always validate user input.

### Key Guidelines

| Guideline | Summary |
|-----------|---------|
| **SL.io.1** | Use character-level input only when necessary. |
| **SL.io.2** | Always consider ill-formed input when reading. |
| **SL.io.3** | Prefer iostreams for I/O (type-safe, extensible). |
| **SL.io.10** | Call `ios_base::sync_with_stdio(false)` if not using `printf`. |
| **SL.io.50** | Avoid `std::endl` (use `'\n'` instead). |
| **Check stream state** | Always verify extraction succeeded. |
| **Clear failures** | Use `clear()` and `ignore()` to recover. |
| **Use `const` references** | Pass streams by reference. |
| **Prefer `'\n'`** | Avoid unnecessary flushes. |

### Annotated Code Example

```cpp
#include <iostream>

int main() {
    std::ios_base::sync_with_stdio(false);   // faster I/O
    int x;
    while (std::cin >> x) {                  // check state
        std::cout << x << '\n';              // '\n', not endl
    }
}
```

**Expected Output**  
(Reads integers until EOF, printing each on a new line.)

**Why this output?**  
`sync_with_stdio(false)` disables synchronisation; `'\n'` avoids flushing.

### Real-World Cases

- **Competitive programming** — fast I/O.
- **Robust applications** — input validation.
- **Library design** — safe stream interfaces.

### References

- C++ Core Guidelines — SL.io: Iostream — https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#S-io
- `std::ios_base::sync_with_stdio` — https://en.cppreference.com/w/cpp/io/ios_base/sync_with_stdio

---

## References

- `<iostream>` header — https://en.cppreference.com/w/cpp/header/iostream
- `std::cin` — https://en.cppreference.com/w/cpp/io/cin
- `std::cout` — https://en.cppreference.com/w/cpp/io/cout
- `std::cerr` — https://en.cppreference.com/w/cpp/io/cerr
- `std::clog` — https://en.cppreference.com/w/cpp/io/clog
- `std::wcin` — https://en.cppreference.com/w/cpp/io/wcin
- `std::wcout` — https://en.cppreference.com/w/cpp/io/wcout
- `std::wcerr` — https://en.cppreference.com/w/cpp/io/wcerr
- `std::wclog` — https://en.cppreference.com/w/cpp/io/wclog
- Input/output manipulators — https://en.cppreference.com/w/cpp/io/manip
- `std::ios_base::iostate` — https://en.cppreference.com/w/cpp/io/ios_base/iostate
- `std::ios_base::fmtflags` — https://en.cppreference.com/w/cpp/io/ios_base/fmtflags
- C++ Core Guidelines — SL.io: Iostream — https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#S-io
- C++ Working Draft — Wide stream objects — https://timsong-cpp.github.io/cppwp/n3337/wide.stream.objects

---

This cheat sheet provides a comprehensive, structured reference for C++ Standard I/O, covering every requested stream object, stream operator, stream state, formatting concept, and each listed manipulator individually, with definitions, purposes, syntax rules, annotated examples, real-world cases, and authoritative references.