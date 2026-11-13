# C++ Formatting (C++20/23) — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
The C++ Formatting library (introduced in C++20) is a component of the C++ Standard Library, declared in `<format>`, that provides a type-safe, extensible, and high-performance mechanism for formatting values into strings and output streams.

**Technical Definition**
The text formatting library offers a safe and extensible alternative to the `printf` family of functions, intended to complement the existing C++ I/O streams library. It centres on the function template `std::format`, which performs compile-time format string checking (via `std::format_string` since P2216R3) and returns a `std::string`. Supporting functions `std::format_to`, `std::format_to_n`, and `std::formatted_size` allow writing to output iterators or computing the required buffer size without allocation. Extensibility is achieved through specialisations of `std::formatter<T, CharT>`, which define parsing and formatting rules for a given type. C++23 adds `std::print` and `std::println` (header `<print>`) for direct formatted output to streams without an intermediate string.

**Beginner-Friendly Explanation**
Think of the C++20 Formatting library as a modern replacement for `printf` and messy `std::stringstream` code. You write a format string with `{}` placeholders and pass the values to be inserted. The compiler checks at compile time that the types match, so you get clear errors instead of runtime crashes. The result is a `std::string` (or you can write directly to an output iterator). In C++23, `std::print` and `std::println` let you send formatted text straight to the console, just like `printf` but type-safe.

### Key Characteristics

- **Type-safe** — the compiler verifies that argument types match the format string at compile time (since P2216R3).
- **Extensible** — user-defined types can be formatted by specialising `std::formatter`.
- **Performance-oriented** — generally faster than iostreams and often faster than `printf` due to compile-time parsing.
- **Python-inspired syntax** — format specifications are based on Python’s `str.format` syntax.
- **Locale-aware** — an optional `std::locale` overload supports localised formatting.
- **No intermediate buffer (C++23)** — `std::print` writes directly to the output stream.
- **Unicode-friendly** — `std::print` handles UTF-8 correctly on supported platforms.

### Prerequisites

- C++20 (or later) compiler and standard library.
- Basic understanding of templates and string manipulation.
- Familiarity with `std::string` and output iterators.
- Awareness of format strings and placeholders (e.g., `{}`, `{:d}`).

### Related Programming Areas

- **String conversion** — converting values to textual form.
- **Logging** — building formatted log messages.
- **User output** — printing formatted results to the console.
- **Internationalisation** — locale-aware number and date formatting.
- **Serialization** — producing textual representations of objects.

### Core Concepts / Features

1. `<format>`
2. `std::format`
3. `std::format_to`
4. `std::format_to_n`
5. `std::formatted_size`
6. `std::formatter`
7. Format specifications
8. `std::print` (C++23)
9. `std::println` (C++23)
10. Formatting best practices

---

## 1. `<format>`

### Definitions

**Core Definition**
`<format>` is the standard library header that declares the C++20 text formatting facilities, including `std::format` and related functions, as well as `std::formatter`.

**Technical Definition**
The header defines the function templates `format`, `format_to`, `format_to_n`, and `formatted_size`, the class template `basic_format_string` (and its aliases `format_string`/`wformat_string`), the extensibility class template `formatter`, and supporting types such as `basic_format_parse_context`, `basic_format_context`, `basic_format_arg`, and `format_error`.

**Beginner-Friendly Explanation**
`<format>` is the header you include to use the new formatting library. It gives you `std::format` (to create a string) and `std::formatter` (to teach the library how to format your own types).

### Purposes

- To provide the declarations for all C++20 formatting functions and types.
- To expose the `std::format` family of functions.
- To enable compile-time format string checking.
- To support extensible, user-defined formatting.

### Syntax Rules and Structure

**General Syntax**
```cpp
#include <format>
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `<format>` | Standard header for the C++20 formatting library. |

**Syntax Rules**
- Must be included before using `std::format` or any related type.
- The header is self-contained; no additional headers are required for basic usage.

**Constraints and Limitations**
- Requires C++20 (or later).
- Some early implementations may have incomplete support; check compiler documentation.

### Annotated Code Example

```cpp
#include <format>    // enables std::format
#include <iostream>

int main() {
    std::string s = std::format("Hello, {}!", "world");
    std::cout << s << '\n';
}
```

**Expected Output**
```
Hello, world!
```

**Why this output?**  
`<format>` declares `std::format`, which replaces `{}` with the argument `"world"`.

### Real-World Cases

- Any modern C++20 project needing type-safe string formatting.
- Libraries that want to provide `std::formatter` specialisations.
- Applications replacing `printf` or `stringstream` with a safer alternative.

### References

- `<format>` header — https://en.cppreference.com/w/cpp/header/format

---

## 2. `std::format`

### Definitions

**Core Definition**
`std::format` is a function template that formats its arguments according to a format string and returns the result as a `std::string`.

**Technical Definition**
`std::format(fmt, args...)` is equivalent to `std::vformat(fmt.get(), std::make_format_args(args...))`. Since P2216R3, the format string is checked at compile time via `std::format_string`; a mismatch between the format string and argument types is a compile-time error. An overload accepting a `std::locale` is also provided for locale-specific formatting.

**Beginner-Friendly Explanation**
`std::format` is the main tool. You give it a format string like `"x = {}"` and the value to insert, and it returns a `std::string` with the value filled in. It is type-safe and fast.

### Purposes

- To create a formatted `std::string` from a format string and arguments.
- To replace unsafe `printf`-style formatting.
- To avoid the verbosity and performance overhead of `std::stringstream`.
- To provide compile-time validation of format strings.

### Syntax Rules and Structure

**General Syntax**
```cpp
std::string s = std::format("format string", arg1, arg2, ...);
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `"format string"` | A string literal containing `{}` placeholders. |
| `arg1, arg2, ...` | Values to be formatted. |

**Syntax Rules**
- Placeholders can be `{}` (automatic indexing) or `{index}` (manual indexing).
- Format specifications follow a colon: `{index:spec}`.
- The format string must be a constant expression for compile-time checking (use `std::vformat` for runtime strings).
- All `std::formatter<T, CharT>` requirements must be satisfied for each argument type.

**Constraints and Limitations**
- Cannot format types that lack a `std::formatter` specialisation.
- Compile-time checking means format strings must be known at compile time (unless using `std::vformat`).

### Annotated Code Examples

**Example 1: Basic formatting**

```cpp
#include <format>
#include <iostream>

int main() {
    int x = 42;
    double pi = 3.14159;
    std::string s = std::format("x = {}, pi = {:.2f}", x, pi);
    std::cout << s << '\n';
}
```

**Expected Output**
```
x = 42, pi = 3.14
```

**Why this output?**  
`{}` is replaced by `42`; `{:.2f}` formats `3.14159` with two decimal places.

**Example 2: Indexed arguments**

```cpp
#include <format>
#include <iostream>

int main() {
    std::string s = std::format("{1} {0}", "world", "hello");
    std::cout << s << '\n';
}
```

**Expected Output**
```
hello world
```

**Why this output?**  
`{1}` refers to the second argument (`"hello"`), `{0}` to the first (`"world"`).

### Real-World Cases

- **Logging** — building log messages with values.
- **User output** — formatting results for display.
- **Serialization** — converting objects to strings.

### References

- `std::format` — https://en.cppreference.com/w/cpp/utility/format/format

---

## 3. `std::format_to`

### Definitions

**Core Definition**
`std::format_to` is a function template that formats its arguments and writes the result through an output iterator.

**Technical Definition**
`std::format_to(OutputIt out, fmt, args...)` writes the formatted representation to the output iterator `out` and returns an iterator past the last character written. It is the iterator-based counterpart of `std::format`.

**Beginner-Friendly Explanation**
Instead of creating a new `std::string`, `std::format_to` lets you write the formatted text directly into an existing buffer (via an iterator), such as a `std::string`’s `back_inserter` or a raw character buffer.

### Purposes

- To format directly into an output iterator, avoiding an intermediate string.
- To append formatted text to an existing string.
- To support high-performance formatting scenarios.
- To enable formatting into custom output devices.

### Syntax Rules and Structure

**General Syntax**
```cpp
std::format_to(out_iterator, "format string", args...);
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `out_iterator` | Output iterator to write to. |
| `"format string"` | Format string with placeholders. |
| `args...` | Values to format. |

**Syntax Rules**
- The output iterator must satisfy the output iterator requirements.
- Returns an iterator one past the last character written.
- Compile-time format string checking applies (unless using `std::vformat_to`).

**Constraints and Limitations**
- The caller is responsible for ensuring the output range is large enough (when using raw pointers).
- Does not perform bounds checking on the output iterator.

### Annotated Code Examples

**Example 1: Appending to a string**

```cpp
#include <format>
#include <iostream>
#include <string>
#include <iterator>

int main() {
    std::string s = "Values: ";
    std::format_to(std::back_inserter(s), "{}, {}", 1, 2);
    std::cout << s << '\n';
}
```

**Expected Output**
```
Values: 1, 2
```

**Why this output?**  
`back_inserter` appends the formatted text to `s`.

**Example 2: Writing to a character array**

```cpp
#include <format>
#include <iostream>

int main() {
    char buf[64];
    auto end = std::format_to(buf, "x = {}", 42);
    *end = '\0';                       // null-terminate
    std::cout << buf << '\n';
}
```

**Expected Output**
```
x = 42
```

**Why this output?**  
`format_to` writes into the array and returns an iterator to the end.

### Real-World Cases

- **High-performance logging** — writing formatted messages into a pre-allocated buffer.
- **String building** — appending formatted values to an existing string.
- **Embedded systems** — avoiding dynamic allocation.

### References

- `std::format_to` — https://en.cppreference.com/w/cpp/utility/format/format_to

---

## 4. `std::format_to_n`

### Definitions

**Core Definition**
`std::format_to_n` is a function template that formats its arguments and writes at most a specified number of characters through an output iterator.

**Technical Definition**
`std::format_to_n(OutputIt out, std::size_t n, fmt, args...)` writes at most `n` characters of the formatted output to `out` and returns a `std::format_to_n_result` containing the output iterator past the last character written and the total (untruncated) size of the formatted output.

**Beginner-Friendly Explanation**
`std::format_to_n` is like `std::format_to`, but it limits how many characters it writes. It is useful when you have a fixed-size buffer and want to avoid overflow.

### Purposes

- To format into a bounded output range safely.
- To determine the full formatted size even when truncating.
- To write into fixed-size buffers without overflow.
- To support embedded and real-time systems with limited memory.

### Syntax Rules and Structure

**General Syntax**
```cpp
auto result = std::format_to_n(out, n, "format string", args...);
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `out` | Output iterator. |
| `n` | Maximum number of characters to write. |
| `"format string"` | Format string. |
| `args...` | Values to format. |

**Syntax Rules**
- Writes at most `n` characters.
- Returns a `format_to_n_result` with `out` (past-the-last-written) and `size` (full formatted size).
- Format string checking is performed at compile time.

**Constraints and Limitations**
- The output iterator must be writable.
- Truncation may occur if the formatted output exceeds `n` characters.

### Annotated Code Examples

**Example 1: Truncating to a buffer**

```cpp
#include <format>
#include <iostream>

int main() {
    char buf[8];
    auto result = std::format_to_n(buf, sizeof(buf) - 1, "Hello, {}!", "world");
    *result.out = '\0';                // null-terminate
    std::cout << buf << '\n';
    std::cout << "Full size: " << result.size << '\n';
}
```

**Expected Output**
```
Hello, 
Full size: 13
```

**Why this output?**  
Only 7 characters (plus null) fit; `result.size` reports the full length `13`.

**Example 2: Safe formatting**

```cpp
#include <format>
#include <iostream>

int main() {
    char buf[4];
    auto result = std::format_to_n(buf, sizeof(buf) - 1, "{}", 123456);
    *result.out = '\0';
    std::cout << buf << '\n';          // "123"
}
```

**Expected Output**
```
123
```

**Why this output?**  
The output is truncated to fit the buffer.

### Real-World Cases

- **Embedded systems** — writing to fixed-size buffers.
- **Network protocols** — formatting into bounded packets.
- **Safety-critical code** — preventing buffer overflows.

### References

- `std::format_to_n` — https://en.cppreference.com/w/cpp/utility/format/format_to_n

---

## 5. `std::formatted_size`

### Definitions

**Core Definition**
`std::formatted_size` is a function template that returns the number of characters that `std::format` would produce for the given arguments.

**Technical Definition**
`std::formatted_size(fmt, args...)` returns the total number of characters in the formatted string without actually producing the string. It is useful for pre-allocating buffers or computing sizes.

**Beginner-Friendly Explanation**
`std::formatted_size` tells you how long the formatted result will be, so you can allocate exactly the right amount of memory before formatting.

### Purposes

- To determine the required buffer size before formatting.
- To pre-allocate strings efficiently.
- To avoid reallocations in performance-sensitive code.
- To support two-pass formatting strategies.

### Syntax Rules and Structure

**General Syntax**
```cpp
std::size_t n = std::formatted_size("format string", args...);
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `"format string"` | Format string. |
| `args...` | Values to format. |

**Syntax Rules**
- Returns the full length of the formatted output as `std::size_t`.
- Format string checking is performed at compile time.

**Constraints and Limitations**
- Does not produce the actual formatted string.
- Requires the same arguments that would be passed to `std::format`.

### Annotated Code Examples

**Example 1: Computing size**

```cpp
#include <format>
#include <iostream>

int main() {
    std::size_t n = std::formatted_size("x = {}, pi = {:.2f}", 42, 3.14159);
    std::cout << "Needed: " << n << " chars\n";
}
```

**Expected Output**
```
Needed: 16 chars
```

**Why this output?**  
The string `"x = 42, pi = 3.14"` has 16 characters.

**Example 2: Pre-allocating a buffer**

```cpp
#include <format>
#include <iostream>
#include <string>

int main() {
    auto n = std::formatted_size("{}", 12345);
    std::string s;
    s.reserve(n);                      // exact allocation
    std::format_to(std::back_inserter(s), "{}", 12345);
    std::cout << s << '\n';
}
```

**Expected Output**
```
12345
```

**Why this output?**  
`formatted_size` enables exact reservation, avoiding reallocations.

### Real-World Cases

- **Memory-constrained systems** — exact allocation.
- **Performance-critical logging** — avoiding reallocation.
- **Buffer management** — sizing before `format_to_n`.

### References

- `std::formatted_size` — https://en.cppreference.com/w/cpp/utility/format/formatted_size

---

## 6. `std::formatter`

### Definitions

**Core Definition**
`std::formatter` is a class template that defines how a given type is formatted by the C++20 formatting library.

**Technical Definition**
The primary template `template<class T, class CharT = char> struct formatter;` is disabled by default. Enabled specialisations meet the `Formatter` named requirements and provide `parse` and `format` member functions. The standard library provides specialisations for basic types, string types, pointers, and (C++23) range types.

**Beginner-Friendly Explanation**
`std::formatter` is how you teach `std::format` to handle your own classes. You write a specialisation that tells the library how to parse a format specification and how to produce the output characters.

### Purposes

- To make user-defined types formattable with `std::format`.
- To control the textual representation of custom types.
- To support custom format specifications (e.g., `{:custom}`).
- To integrate with the standard formatting ecosystem.

### Syntax Rules and Structure

**General Syntax**
```cpp
template <>
struct std::formatter<MyType> {
    constexpr auto parse(std::format_parse_context& ctx) {
        // parse format spec
        return ctx.begin();
    }
    auto format(const MyType& obj, std::format_context& ctx) const {
        // write to ctx.out()
        return std::format_to(ctx.out(), "...");
    }
};
```

**Component Breakdown**

| Member | Description |
|--------|-------------|
| `parse(ctx)` | Parses the format specification; must be `constexpr` (since P2216R3). |
| `format(obj, ctx)` | Produces the output; returns an output iterator. |

**Syntax Rules**
- `parse` must return an iterator past the parsed specification.
- `format` must write via `ctx.out()` and return the resulting iterator.
- Specialisations must meet `BasicFormatter` (and usually `Formatter`) requirements.

**Constraints and Limitations**
- Without an enabled specialisation, a type cannot be formatted.
- `parse` must not consume the closing `}`.
- Writing incorrect `format` logic can lead to buffer issues.

### Annotated Code Examples

**Example 1: Formatting a custom struct**

```cpp
#include <format>
#include <iostream>
#include <string>

struct Point {
    int x, y;
};

template <>
struct std::formatter<Point> {
    constexpr auto parse(std::format_parse_context& ctx) {
        return ctx.begin();            // no custom spec
    }
    auto format(const Point& p, std::format_context& ctx) const {
        return std::format_to(ctx.out(), "({}, {})", p.x, p.y);
    }
};

int main() {
    Point p{1, 2};
    std::cout << std::format("Point: {}", p) << '\n';
}
```

**Expected Output**
```
Point: (1, 2)
```

**Why this output?**  
The `formatter<Point>` specialisation defines how to produce `(x, y)`.

**Example 2: Custom format specification**

```cpp
#include <format>
#include <iostream>

struct RGB {
    int r, g, b;
};

template <>
struct std::formatter<RGB> {
    bool hex = false;
    constexpr auto parse(std::format_parse_context& ctx) {
        auto it = ctx.begin();
        if (it != ctx.end() && *it == 'x') hex = true;
        return it;
    }
    auto format(const RGB& c, std::format_context& ctx) const {
        if (hex)
            return std::format_to(ctx.out(), "#{:02x}{:02x}{:02x}", c.r, c.g, c.b);
        return std::format_to(ctx.out(), "rgb({}, {}, {})", c.r, c.g, c.b);
    }
};

int main() {
    RGB c{255, 128, 0};
    std::cout << std::format("{:x}", c) << '\n';   // hex format
}
```

**Expected Output**
```
#ff8000
```

**Why this output?**  
The `parse` function recognises the `'x'` specifier and sets `hex = true`.

### Real-World Cases

- **Domain types** — formatting dates, coordinates, colours.
- **Logging** — consistent representations of objects.
- **Library APIs** — making types work with `std::format`.

### References

- `std::formatter` — https://en.cppreference.com/w/cpp/utility/format/formatter

---

## 7. Format Specifications

### Definitions

**Core Definition**
Format specifications are the mini-language embedded in a format string that controls how each argument is formatted (alignment, width, precision, base, etc.).

**Technical Definition**
For basic types and string types, the format specification is based on Python’s format specification mini-language. The syntax is: `fill-and-align(optional) sign(optional) #(optional) 0(optional) width(optional) precision(optional) L(optional) type(optional)`. The `sign`, `#`, and `0` options are only valid when an integer or floating-point presentation type is used.

**Beginner-Friendly Explanation**
After the colon in `{:...}`, you can write a compact specification that says how to align, pad, and present the value. For example, `{:>10}` means right-align in a width of 10, and `{:.2f}` means fixed-point with two decimals.

### Purposes

- To control alignment, width, and padding.
- To choose numeric base (decimal, hex, octal, binary).
- To set floating-point notation (fixed, scientific, general).
- To control precision and sign display.
- To enable locale-specific formatting.

### Syntax Rules and Structure

**General Syntax**
```cpp
"{:spec}"
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `fill-and-align` | Optional fill character followed by `<`, `>`, or `^`. |
| `sign` | `+`, `-`, or space. |
| `#` | Alternate form (e.g., `0x` prefix). |
| `0` | Zero-padding. |
| `width` | Minimum field width. |
| `precision` | Digits after decimal (for floats) or significant digits. |
| `L` | Locale-specific formatting. |
| `type` | Presentation type (`d`, `x`, `o`, `b`, `f`, `e`, `g`, `s`, etc.). |

**Syntax Rules**
- The fill character can be any character except `{` or `}`.
- `<` aligns left (default for non-numeric), `>` right (default for numeric), `^` centres.
- The `0` option enables zero-padding.
- The `#` option adds base prefixes (`0x`, `0b`, etc.).

**Constraints and Limitations**
- Not all combinations are valid (e.g., alignment with `0` padding).
- Custom formatters may interpret the specification differently.

### Annotated Code Examples

**Example 1: Alignment and width**

```cpp
#include <format>
#include <iostream>

int main() {
    std::cout << std::format("|{:<10}|{:>10}|{:^10}|\n", "left", "right", "center");
}
```

**Expected Output**
```
|left      |     right|  center  |
```

**Why this output?**  
`<` left-aligns, `>` right-aligns, `^` centres within width 10.

**Example 2: Numeric formats**

```cpp
#include <format>
#include <iostream>

int main() {
    int x = 255;
    std::cout << std::format("dec={0} hex={0:#x} oct={0:#o} bin={0:#b}\n", x);
}
```

**Expected Output**
```
dec=255 hex=0xff oct=0377 bin=0b11111111
```

**Why this output?**  
The `#` option adds base prefixes for hex, octal, and binary.

### Real-World Cases

- **Tabular output** — aligning columns.
- **Debugging** — displaying values in hexadecimal.
- **Financial reports** — fixed decimal precision.

### References

- Standard format specification — https://en.cppreference.com/w/cpp/utility/format/spec

---

## 8. `std::print` (C++23)

### Definitions

**Core Definition**
`std::print` is a C++23 function template, declared in `<print>`, that formats its arguments and writes the result directly to an output stream (by default `stdout`).

**Technical Definition**
`std::print(fmt, args...)` formats `args` according to `fmt` and writes the result to `stdout` without allocating an intermediate string. Overloads accept a `std::FILE*` or a `std::ostream&` as the first argument. The format syntax is identical to `std::format`.

**Beginner-Friendly Explanation**
`std::print` is like `std::format` but it prints the result immediately instead of giving you a string. It is the modern, type-safe replacement for `printf` and `std::cout << std::format(...)`.

### Purposes

- To print formatted output directly to a stream.
- To avoid the overhead of creating an intermediate string.
- To provide a type-safe alternative to `printf`.
- To handle UTF-8 output correctly on supported platforms.

### Syntax Rules and Structure

**General Syntax**
```cpp
#include <print>
std::print("format string", args...);
```

**Component Breakdown**

| Overload | Description |
|----------|-------------|
| `print(fmt, args...)` | Writes to `stdout`. |
| `print(stream, fmt, args...)` | Writes to a `FILE*` or `ostream&`. |

**Syntax Rules**
- Format string checking is performed at compile time.
- No trailing newline is added automatically.
- Flushing behaviour depends on the target stream (buffered by default).

**Constraints and Limitations**
- Requires C++23 and a compatible standard library.
- Not all compilers provide `<print>` yet (as of early 2025).

### Annotated Code Examples

**Example 1: Basic printing**

```cpp
#include <print>

int main() {
    std::print("Hello, {}!\n", "world");
}
```

**Expected Output**
```
Hello, world!
```

**Why this output?**  
`std::print` formats and writes directly to `stdout`.

**Example 2: Printing to `stderr`**

```cpp
#include <print>

int main() {
    std::print(stderr, "Error: {}\n", 42);
}
```

**Expected Output**
```
Error: 42
```

**Why this output?**  
The first argument selects the `stderr` stream.

### Real-World Cases

- **Console applications** — fast, type-safe output.
- **Logging** — writing to `stderr` or `stdout`.
- **CLI tools** — formatted progress and status messages.

### References

- `<print>` header — https://en.cppreference.com/w/cpp/header/print
- `std::print` — https://en.cppreference.com/w/cpp/io/print

---

## 9. `std::println` (C++23)

### Definitions

**Core Definition**
`std::println` is a C++23 function template, declared in `<print>`, that formats its arguments and writes the result followed by a newline to an output stream.

**Technical Definition**
`std::println(fmt, args...)` behaves like `std::print` but automatically appends a newline character to each output. An overload with no arguments prints just a newline (C++26).

**Beginner-Friendly Explanation**
`std::println` is `std::print` plus a newline. It is the easiest way to print a formatted line of text.

### Purposes

- To print a formatted line with an automatic newline.
- To simplify console output loops.
- To provide a convenient alternative to `std::cout << ... << '\n'`.
- To ensure consistent line termination.

### Syntax Rules and Structure

**General Syntax**
```cpp
std::println("format string", args...);
```

**Component Breakdown**

| Overload | Description |
|----------|-------------|
| `println(fmt, args...)` | Formats and prints with a trailing newline. |
| `println()` | Prints a bare newline (C++26). |

**Syntax Rules**
- Same format string checking as `std::format`.
- Adds `'\n'` after the formatted output.
- Works with `stdout` by default; overloads accept a stream.

**Constraints and Limitations**
- Requires C++23 (the no-argument overload is C++26).

### Annotated Code Examples

**Example 1: Basic println**

```cpp
#include <print>

int main() {
    std::println("x = {}, pi = {:.2f}", 42, 3.14159);
}
```

**Expected Output**
```
x = 42, pi = 3.14
```

**Why this output?**  
The formatted text is printed followed by a newline.

**Example 2: Loop with println**

```cpp
#include <print>
#include <vector>

int main() {
    std::vector<int> v = {1, 2, 3};
    for (int x : v) std::print("{} ", x);
    std::println();                    // newline after loop
}
```

**Expected Output**
```
1 2 3 
```

**Why this output?**  
`std::print` outputs each value without newline; `std::println()` adds the final newline.

### Real-World Cases

- **Console output** — everyday formatted printing.
- **Logging** — one-line log entries.
- **Test output** — readable formatted results.

### References

- `std::println` — https://en.cppreference.com/w/cpp/io/println

---

## 10. Formatting Best Practices

### Definitions

**Core Definition**
C++ formatting best practices are guidelines for using the C++20/23 formatting library safely, efficiently, and idiomatically.

**Technical Definition**
These practices include preferring `std::format` over `printf` and `stringstream` for type safety and performance, using `std::format_to`/`std::format_to_n` to avoid intermediate allocations, leveraging `std::formatted_size` for exact pre-allocation, providing `std::formatter` specialisations for user-defined types, and using `std::print`/`std::println` for direct output in C++23. Performance considerations include avoiding unnecessary formatting in hot loops and pre-reserving buffers.

**Beginner-Friendly Explanation**
Best practices are good habits: use `std::format` instead of `printf` (it is safer), use `format_to` when you do not need a separate string, and teach the library how to format your own types. In C++23, prefer `print`/`println` for console output.

### Key Guidelines

| Guideline | Summary |
|-----------|---------|
| **Prefer `std::format` over `printf`** | Type-safe and extensible. |
| **Prefer `std::format` over `stringstream`** | Faster and more concise. |
| **Use `format_to` for direct output** | Avoid intermediate strings. |
| **Use `formatted_size` for pre-allocation** | Exact buffer sizing. |
| **Specialise `std::formatter` for user types** | Enable seamless formatting. |
| **Prefer `print`/`println` in C++23** | No intermediate string, direct output. |
| **Avoid formatting in hot loops** | Reduce overhead where possible. |
| **Use `std::format_to_n` for bounded buffers** | Prevent overflow. |

### Annotated Code Example

```cpp
#include <format>
#include <iostream>
#include <string>

struct Point { int x, y; };

template <>
struct std::formatter<Point> {
    constexpr auto parse(std::format_parse_context& ctx) { return ctx.begin(); }
    auto format(const Point& p, std::format_context& ctx) const {
        return std::format_to(ctx.out(), "({}, {})", p.x, p.y);
    }
};

int main() {
    Point p{1, 2};
    // Prefer format for type-safe string creation
    std::string s = std::format("Point: {}", p);
    std::cout << s << '\n';

    // Use formatted_size for exact allocation
    auto n = std::formatted_size("Point: {}", p);
    std::string s2;
    s2.reserve(n);
    std::format_to(std::back_inserter(s2), "Point: {}", p);
    std::cout << s2 << '\n';
}
```

**Expected Output**
```
Point: (1, 2)
Point: (1, 2)
```

**Why this output?**  
The custom `formatter<Point>` enables formatting of `Point` with `std::format` and `std::format_to`.

### Real-World Cases

- **Modern C++ codebases** — replacing legacy `printf` and `stringstream`.
- **High-performance applications** — using `format_to` and `formatted_size`.
- **Library development** — providing `formatter` specialisations.

### References

- Formatting library (cppreference) — https://en.cppreference.com/w/cpp/utility/format
- `std::format` — https://en.cppreference.com/w/cpp/utility/format/format
- `std::print` / `std::println` — https://en.cppreference.com/w/cpp/io/print
- C++ Core Guidelines — SL: The Standard Library — https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#S-stdlib

---

## References

- Formatting library (since C++20) — https://en.cppreference.com/w/cpp/utility/format
- `<format>` header — https://en.cppreference.com/w/cpp/header/format
- `std::format` — https://en.cppreference.com/w/cpp/utility/format/format
- `std::format_to` — https://en.cppreference.com/w/cpp/utility/format/format_to
- `std::format_to_n` — https://en.cppreference.com/w/cpp/utility/format/format_to_n
- `std::formatted_size` — https://en.cppreference.com/w/cpp/utility/format/formatted_size
- `std::formatter` — https://en.cppreference.com/w/cpp/utility/format/formatter
- Standard format specification — https://en.cppreference.com/w/cpp/utility/format/spec
- `<print>` header (C++23) — https://en.cppreference.com/w/cpp/header/print
- `std::print` — https://en.cppreference.com/w/cpp/io/print
- `std::println` — https://en.cppreference.com/w/cpp/io/println
- C++ Core Guidelines — SL: The Standard Library — https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#S-stdlib

---

This cheat sheet provides a comprehensive, structured reference for the C++20/C++23 formatting library, covering every requested component individually, with definitions, purposes, syntax rules, annotated code examples, real-world cases, and authoritative references.