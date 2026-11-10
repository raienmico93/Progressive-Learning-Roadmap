# C# Strings — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
A `string` in C# is an immutable reference type that represents a sequence of text. It is an alias for `System.String` and is the primary type used for textual data throughout .NET applications.

**Technical Definition**
A string is an object of type `System.String` whose value is text. Internally, the text is stored as a sequential read-only collection of `Char` objects. The `Length` property represents the number of `Char` objects (UTF-16 code units), not the number of Unicode characters. Beginning with C# 11, string literals can be annotated with a `u8` suffix to produce UTF-8 string literals of type `ReadOnlySpan<byte>`.

**Beginner-Friendly Explanation**
A string is a **read-only strip of text**. You can create strings, read their characters, search inside them, and combine them—but you cannot change an existing string in place. Every operation that appears to modify a string actually produces a new string. Think of it as writing on a whiteboard that has been sealed under glass: you can read it and create a new board with the changes, but the original board remains untouched.

---

### Key Characteristics

- **Immutable** — String objects cannot be modified after creation; all mutation methods return new strings.
- **Reference type with value-like equality** — Strings are reference types, but `==` and `Equals` compare content, not references.
- **Unicode-based** — Internally uses UTF-16 encoding; each `char` is a UTF-16 code unit.
- **Interning** — Identical string literals in the same assembly typically refer to the same instance.
- **Rich API** — `System.String` provides dozens of methods for searching, replacing, formatting, and comparing text.
- **Literal forms** — Regular, verbatim (`@`), raw (`"""`), interpolated (`$`), and UTF-8 (`u8`) literals are available.
- **Culture-aware** — Comparison and formatting can be culture-sensitive or ordinal, depending on the method and `StringComparison` argument.

---

### Prerequisites

- Basic understanding of C# program structure (`class`, `Main`, namespaces).
- Familiarity with value types and reference types.
- Knowledge of Unicode fundamentals (code points, code units, encodings).
- Awareness of the `System` and `System.Text` namespaces.
- Basic understanding of memory allocation and garbage collection (helpful but not required).

---

### Related Programming Areas

- **Text processing and parsers** — Strings are the primary input and output format for parsers, tokenizers, and compilers.
- **Data serialization** — JSON, XML, CSV, and other formats are represented as strings.
- **Internationalization (i18n)** — Culture-sensitive comparison and formatting are essential for global applications.
- **Memory and performance engineering** — String allocation, `StringBuilder`, `Span<char>`, and UTF-8 literals are key performance tools.
- **Security** — String comparison and encoding choices affect authentication, authorization, and injection prevention.

---

### Core Concepts / Features

1. String (overarching concept)
2. String immutability
3. String literals
4. Verbatim strings
5. Raw string literals (C# 11)
6. Interpolated strings
7. `$"..."` interpolation
8. String interpolation improvements
9. String methods (grouped: length and access, searching, modifying, case and trimming, splitting and joining, padding, formatting, validation, comparison)
10. `StringBuilder`
11. String performance
12. String best practices
13. `StringComparison`
14. Culture-sensitive comparison
15. Unicode
16. UTF-8
17. Encoding

Each concept follows a uniform structure: definitions, purposes, syntax rules and structure, annotated examples with expected outputs, real-world cases, and references.

---

## 1. String (Overarching Concept)

### Definitions

**Core Definition**
`string` is an immutable reference type representing text, alias for `System.String`.

**Technical Definition**
`System.String` is a sealed class that represents text as a sequential collection of UTF-16 code units (`char`). It is immutable: methods such as `Replace`, `Substring`, and `ToUpper` return new `string` instances rather than modifying the original. The `string` keyword is a C# alias for `System.String`, so the two are interchangeable.

**Beginner-Friendly Explanation**
A string is a **read-only sentence**. You can read it, copy it, search inside it, and create a new sentence with changes, but you cannot erase or overwrite characters in the original sentence.

---

### Purposes

- To represent textual data in variables, fields, parameters, and return values.
- To provide a rich, safe API for searching, replacing, formatting, and comparing text.
- To serve as the universal text interchange format for files, networks, and APIs.
- To support Unicode and culture-aware operations for international applications.
- To enable compile-time string literals and constant folding.

---

### Syntax Rules and Structure

```
string   identifier   [ = "literal" | @“verbatim” | $”interpolated” ]   ;
System.String   identifier   ;
```

**Component breakdown**

- `string` — C# alias for `System.String`.
- `identifier` — Variable name.
- Literal forms — Regular, verbatim, interpolated, raw, or UTF-8.

#### Syntax Rules

- `string` and `System.String` are equivalent.
- Strings are immutable; all modification methods return new strings.
- `==` and `!=` compare string **values**, not references.
- String literals are interned within an assembly.
- `string.Empty` is preferred over `""` for empty strings.

#### Constraints and Limitations

- Strings are reference types and can be `null`; use null checks or nullable reference types.
- `Length` counts UTF-16 code units, not Unicode grapheme clusters.
- Repeated concatenation can create many temporary strings; use `StringBuilder` for heavy manipulation.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Declaring and Initializing Strings

```csharp
using System;

class Program
{
    static void Main()
    {
        string message1;                          // uninitialized
        string? message2 = null;                  // nullable
        string message3 = System.String.Empty;    // empty string
        string oldPath = "c:\\Program Files";     // regular literal with escapes
        string newPath = @"c:\Program Files";     // verbatim literal
        var greeting = "Hello World!";            // implicitly typed
        const string ConstantMessage = "Fixed";   // constant

        Console.WriteLine(newPath);               // c:\Program Files
        Console.WriteLine(greeting);              // Hello World!
        Console.WriteLine(ConstantMessage);       // Fixed
    }
}
```

**Expected Output**

```
c:\Program Files
Hello World!
Fixed
```

**Why this output occurs**

- `newPath` uses a verbatim literal, so backslashes are not escape sequences.
- `greeting` is inferred as `string`.
- `ConstantMessage` is a compile-time constant.

---

#### Example 2: Value Equality of Strings

```csharp
using System;

class Program
{
    static void Main()
    {
        string a = "hello";
        string b = "hel" + "lo";   // compile-time concatenation

        Console.WriteLine(a == b);                // True
        Console.WriteLine(ReferenceEquals(a, b)); // True (interned)
    }
}
```

**Expected Output**

```
True
True
```

**Why this output occurs**

- `==` compares content, so `a` and `b` are equal.
- The compiler folds `"hel" + "lo"` into `"hello"`, and literals are interned, so both refer to the same instance.

---

### Real-World Cases

- **User input and output** — Reading names, messages, and commands.
- **Serialization** — JSON, XML, and CSV strings.
- **Configuration** — Connection strings, file paths, and settings.
- **Localization** — Translated UI text.

---

### References

- Strings — C# Programming Guide - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/strings/
- System.String — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.string
- string keyword — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/reference-types

---

## 2. String Immutability

### Definitions

**Core Definition**
String immutability means that a string object’s contents cannot be changed after the object is created.

**Technical Definition**
`System.String` is immutable. All methods and operators that appear to modify a string (e.g., `+=`, `Replace`, `ToUpper`, `Substring`) actually return a **new** `string` object containing the result, leaving the original string unchanged. This design enables safe sharing of string references, thread safety, and efficient interning.

**Beginner-Friendly Explanation**
Once a string is created, it is **frozen**. You cannot poke new characters into it. Any “change” produces a brand-new string and leaves the old one exactly as it was.

---

### Purposes

- To enable safe sharing of strings across methods, threads, and assemblies.
- To support string interning and compile-time constant folding.
- To eliminate a class of concurrency bugs (strings cannot be mutated by another thread).
- To simplify reasoning about program state.
- To allow the runtime to optimize string storage and comparison.

---

### Syntax Rules and Structure

Immutability is a semantic property, not a syntactic constraint. It is observed through the behaviour of operations:

```
string s = "Hello";
s += " World";          // creates a new string; the original "Hello" is unchanged
string upper = s.ToUpper(); // new string; s remains "Hello World"
```

**Component breakdown**

- `+=` — Concatenation operator; returns a new string.
- `ToUpper()` — Returns a new uppercase string.
- `Replace()`, `Substring()`, etc. — All return new strings.

#### Syntax Rules

- No method can modify the contents of an existing string instance.
- `StringBuilder` is the mutable counterpart for heavy manipulation.
- String literals and interned strings are shared and immutable.

#### Constraints and Limitations

- Repeated concatenation in loops creates many temporary strings and increases GC pressure.
- “Modifying” a string always allocates a new object (unless the result is the same reference, as with `Replace` when no match occurs).
- `unsafe` code can technically mutate string memory, but doing so is undefined behaviour and strongly discouraged.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Immutability in Action

```csharp
using System;

class Program
{
    static void Main()
    {
        string original = "Hello";
        string modified = original.Replace("H", "J");

        Console.WriteLine(original);   // Hello (unchanged)
        Console.WriteLine(modified);   // Jello
        Console.WriteLine(ReferenceEquals(original, modified)); // False
    }
}
```

**Expected Output**

```
Hello
Jello
False
```

**Why this output occurs**

- `Replace` returns a new string; `original` is untouched.
- `ReferenceEquals` confirms they are different objects.

---

#### Example 2: Loop Concatenation and Allocation

```csharp
using System;

class Program
{
    static void Main()
    {
        string result = "";
        for (int i = 0; i < 5; i++)
        {
            result += i.ToString();   // each iteration creates a new string
        }
        Console.WriteLine(result);    // 01234
    }
}
```

**Expected Output**

```
01234
```

**Why this output occurs**

- Each `+=` creates a new string because strings are immutable.
- The final string is the concatenation of all intermediate results.
- For large loops, this pattern is inefficient; `StringBuilder` is preferred.

---

### Real-World Cases

- **Caching and interning** — Immutable strings can be safely cached and reused.
- **Concurrent systems** — Strings can be shared across threads without locks.
- **Configuration keys** — Constant strings used as dictionary keys are safe.

---

### References

- String immutability — Strings (C# Programming Guide) - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/strings/#immutability-of-string-objects
- String vs. StringBuilder - https://github.com/skylinecommunications/dataminer-docs/blob/573e18c727d6935f48c1a4d57e34ab039702f997/develop/codingguidelines/Protocol/String_vs_StringBuilder.md

---

## 3. String Literals

### Definitions

**Core Definition**
A string literal is source-code text enclosed in double quotes that represents a fixed string value.

**Technical Definition**
C# supports several forms of string literals: **regular** (`"..."`), **verbatim** (`@"..."`), **raw** (`"""..."""`, C# 11+), **interpolated** (`$"..."`), and **UTF-8** (`"..."u8`, C# 11+). Regular literals support escape sequences such as `\n`, `\t`, `\"`, and `\\`. Verbatim literals treat backslashes literally and allow multi-line content. Raw literals require no escaping and support multi-line text with indentation trimming. UTF-8 literals produce `ReadOnlySpan<byte>` values.

**Beginner-Friendly Explanation**
A string literal is **text you write directly in your code**, surrounded by quotes. Different literal forms let you handle special characters, multi-line text, and embedded quotes more conveniently.

---

### Purposes

- To embed fixed text directly in source code.
- To provide compile-time constants for messages, paths, and keys.
- To support readable multi-line strings (raw and verbatim literals).
- To avoid escaping when working with JSON, XML, SQL, and regular expressions.
- To enable UTF-8 encoded literals for interoperability and performance.

---

### Syntax Rules and Structure

```
// Regular literal
"Hello, World!"

// Verbatim literal
@"C:\Program Files"

// Raw literal (C# 11+)
"""This is "raw" content"""

// Interpolated literal
$"Hello, {name}!"

// UTF-8 literal (C# 11+)
"Hello"u8
```

**Component breakdown**

- `"..."` — Regular literal; supports escape sequences.
- `@"..."` — Verbatim literal; no escape processing, multi-line allowed.
- `"""..."""` — Raw literal; no escaping, multi-line, indentation trimming.
- `$"..."` — Interpolated literal; embeds expressions in `{}`.
- `"..."u8` — UTF-8 literal; type `ReadOnlySpan<byte>`.

#### Syntax Rules

- Regular literals require escapes for `"`, `\`, and control characters.
- Verbatim literals use `""` to represent a double quote inside the literal.
- Raw literals use at least three double quotes; the opening and closing delimiter must have the same number of quotes.
- Interpolated literals can combine with verbatim (`$@"..."`) or raw (`$$"""..."""`) forms.
- UTF-8 literals require the `u8` suffix and are not constants.

#### Constraints and Limitations

- Raw literals require C# 11+.
- UTF-8 literals require C# 11+ and produce `ReadOnlySpan<byte>`, not `string`.
- Verbatim literals cannot contain a `"` followed by a newline without doubling.
- Regular literals cannot span multiple lines.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Regular vs. Verbatim Literals

```csharp
using System;

class Program
{
    static void Main()
    {
        string regular = "C:\\Program Files\\App";
        string verbatim = @"C:\Program Files\App";

        Console.WriteLine(regular);    // C:\Program Files\App
        Console.WriteLine(verbatim);   // C:\Program Files\App
        Console.WriteLine(regular == verbatim); // True
    }
}
```

**Expected Output**

```
C:\Program Files\App
C:\Program Files\App
True
```

**Why this output occurs**

- Both literals represent the same text; the regular literal uses escapes, the verbatim literal does not.
- `==` compares content, so they are equal.

---

#### Example 2: Raw String Literal with Quotes

```csharp
using System;

class Program
{
    static void Main()
    {
        string raw = """He said, "Hello!" and left.""";
        Console.WriteLine(raw);   // He said, "Hello!" and left.
    }
}
```

**Expected Output**

```
He said, "Hello!" and left.
```

**Why this output occurs**

- The raw literal is delimited by three double quotes, so internal double quotes require no escaping.

---

### Real-World Cases

- **SQL queries** — Raw literals avoid escaping quotes.
- **JSON/XML payloads** — Raw literals preserve structure without escapes.
- **Regular expressions** — Verbatim literals avoid double escaping.
- **File paths** — Verbatim literals are convenient on Windows.

---

### References

- String literals — Strings (C# Programming Guide) - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/strings/#string-literals
- Raw string literals — C# 11 features - https://learn.microsoft.com/en-us/dotnet/csharp/whats-new/csharp-11#raw-string-literals
- UTF-8 string literals — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/reference-types#utf-8-string-literals

---

## 4. Verbatim Strings

### Definitions

**Core Definition**
A verbatim string is a string literal prefixed with `@` that treats backslashes literally and allows multi-line content.

**Technical Definition**
A verbatim string literal begins with `@"` and ends with `"`. Within the literal, escape sequences are not processed; backslashes are ordinary characters. A double quote inside a verbatim string is represented by doubling it (`""`). Verbatim strings can span multiple lines, including newline characters in the source file.

**Beginner-Friendly Explanation**
A verbatim string is **exactly what you type**, with no escape processing. Backslashes stay as backslashes, and you can write text across multiple lines. The only rule is that a literal double quote is written as `""`.

---

### Purposes

- To write file paths and regular expressions without doubling backslashes.
- To embed multi-line text (SQL, JSON, HTML) readably.
- To avoid escape-sequence mistakes in strings that contain many backslashes.
- To improve readability of strings that mix quotes and backslashes.
- To simplify localization strings that contain special characters.

---

### Syntax Rules and Structure

```
@"verbatim string content"
```

**Component breakdown**

- `@` — Verbatim prefix.
- `"..."` — Content; backslashes are literal; `""` represents a double quote.

#### Syntax Rules

- No escape sequences are processed.
- `""` inside the literal represents a single `"`.
- The literal can span multiple lines.
- Can be combined with interpolation: `$@"..."` or `@$"..."`.

#### Constraints and Limitations

- Cannot contain a lone `"` without doubling.
- Multi-line verbatim strings include the exact line breaks and indentation from the source file.
- Not suitable for strings that must contain a literal `""` sequence without interpretation (use raw strings instead).

---

### Annotated Code Examples and Expected Outputs

#### Example 1: File Paths

```csharp
using System;

class Program
{
    static void Main()
    {
        string path1 = "C:\\Users\\Alice\\Documents";
        string path2 = @"C:\Users\Alice\Documents";

        Console.WriteLine(path1);   // C:\Users\Alice\Documents
        Console.WriteLine(path2);   // C:\Users\Alice\Documents
    }
}
```

**Expected Output**

```
C:\Users\Alice\Documents
C:\Users\Alice\Documents
```

**Why this output occurs**

- The regular literal requires `\\` for each backslash.
- The verbatim literal treats each `\` literally.

---

#### Example 2: Multi-Line SQL

```csharp
using System;

class Program
{
    static void Main()
    {
        string sql = @"
SELECT Id, Name
FROM Customers
WHERE Country = 'USA'
";
        Console.WriteLine(sql);
    }
}
```

**Expected Output**

```

SELECT Id, Name
FROM Customers
WHERE Country = 'USA'

```

**Why this output occurs**

- The verbatim literal preserves line breaks and indentation from the source.
- No escape sequences are needed for the single quotes.

---

### Real-World Cases

- **Windows file paths** — Avoiding doubled backslashes.
- **Regular expressions** — Patterns like `\d+` remain readable.
- **SQL queries** — Multi-line queries with quotes.
- **Configuration snippets** — JSON or XML fragments.

---

### References

- Verbatim string literals — Strings (C# Programming Guide) - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/strings/#verbatim-string-literals

---

## 5. Raw String Literals (C# 11)

### Definitions

**Core Definition**
A raw string literal is a string literal delimited by at least three double quotes (`"""..."""`) that requires no escaping and supports multi-line content with automatic indentation trimming.

**Technical Definition**
Raw string literals were introduced in C# 11. They are delimited by three or more double quotes; the opening and closing delimiters must have the same number of quotes. Inside the literal, no escape sequences are processed: backslashes, double quotes, and braces are literal content. Multi-line raw literals automatically trim leading whitespace based on the indentation of the closing delimiter. Raw literals can be interpolated by prefixing with `$` (one dollar sign per brace required for interpolation holes).

**Beginner-Friendly Explanation**
A raw string is **“what you see is what you get”** text. You can paste JSON, XML, SQL, or HTML directly between triple quotes without worrying about escaping quotes, backslashes, or braces. The compiler even removes common leading indentation so the string looks clean in code.

---

### Purposes

- To avoid escaping quotes and backslashes in structured text (JSON, XML, SQL, regex).
- To write multi-line strings with readable source formatting.
- To embed text that contains sequences like `"""` or `{}` without conflicts.
- To simplify maintenance of embedded code or data.
- To support interpolation without brace escaping (using `$$"""..."""`).

---

### Syntax Rules and Structure

```
// Single-line raw literal
"""content"""

// Multi-line raw literal
"""
    line1
    line2
    """

// Raw literal with embedded quotes
""""content with """ inside"""" // use more outer quotes

// Interpolated raw literal ($$ means double braces are holes)
$$"""
{ "name": "{{name}}" }
"""
```

**Component breakdown**

- `"""` (or more) — Delimiter; opening and closing counts must match.
- Content — Everything between delimiters, including newlines and quotes.
- `$` prefixes — Interpolation; `$$` requires `{{...}}` for holes, leaving single braces literal.

#### Syntax Rules

- At least three double quotes are required.
- The closing delimiter determines indentation trimming for multi-line literals.
- No escape sequences are processed.
- To include a sequence of `"""` in content, use more than three quotes in the delimiter.
- Interpolated raw literals use `$` (one dollar = single braces holes) or `$$` (double braces holes), etc.

#### Constraints and Limitations

- Requires **C# 11** or later.
- Raw literals are UTF-16 strings by default; add `u8` for UTF-8 bytes.
- Multi-line indentation trimming follows strict rules based on the closing delimiter’s position.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: JSON Without Escaping

```csharp
using System;

class Program
{
    static void Main()
    {
        string json = """
        {
            "name": "Alice",
            "age": 30,
            "city": "New York"
        }
        """;

        Console.WriteLine(json);
    }
}
```

**Expected Output**

```
{
    "name": "Alice",
    "age": 30,
    "city": "New York"
}
```

**Why this output occurs**

- The raw literal preserves the JSON structure without requiring `\"` escapes.
- Leading indentation is trimmed based on the closing delimiter’s position.

---

#### Example 2: Interpolated Raw Literal

```csharp
using System;

class Program
{
    static void Main()
    {
        string name = "Alice";
        int age = 30;

        string json = $$"""
        {
            "name": "{{name}}",
            "age": {{age}}
        }
        """;

        Console.WriteLine(json);
    }
}
```

**Expected Output**

```
{
    "name": "Alice",
    "age": 30
}
```

**Why this output occurs**

- `$$` means interpolation holes use double braces (`{{...}}`).
- Single braces (`{` and `}`) remain literal, so JSON structure is preserved.

---

### Real-World Cases

- **Embedded JSON/XML in tests** — Readable payloads without escaping.
- **SQL queries in code** — Multi-line queries with quotes.
- **Regular expressions** — Patterns that contain backslashes and quotes.
- **Code generation templates** — Raw text that includes braces and quotes.

---

### References

- Raw string literals — C# 11 features - https://learn.microsoft.com/en-us/dotnet/csharp/whats-new/csharp-11#raw-string-literals
- Welcome to C# 11 — Raw string literals - https://devblogs.microsoft.com/dotnet/welcome-to-csharp-11/#raw-string-literals
- C# 11 Preview Updates — Raw string literals - https://devblogs.microsoft.com/dotnet/csharp-11-preview-updates/#raw-string-literals

---

## 6. Interpolated Strings

### Definitions

**Core Definition**
An interpolated string is a string literal prefixed with `$` that embeds expressions inside curly braces.

**Technical Definition**
String interpolation, introduced in C# 6, allows expressions to be placed inside `{}` within a string prefixed with `$`. The compiler replaces each hole with the formatted result of the expression. Holes can include optional alignment and format specifiers: `$"{expr,alignment:format}"`. Interpolated strings are compiled to efficient handler-based code in C# 10+ and .NET 6+, avoiding `object[]` allocations and boxing for many scenarios.

**Beginner-Friendly Explanation**
An interpolated string lets you **mix text and variables** in one readable line: `$"Hello, {name}!"`. You do not need to concatenate with `+` or call `String.Format`.

---

### Purposes

- To build strings from text and expressions readably.
- To avoid error-prone positional format strings.
- To provide compile-time checking of embedded expressions.
- To support formatting, alignment, and culture control inline.
- To improve performance over `String.Format` on modern .NET.

---

### Syntax Rules and Structure

```
$"literal {expression[,alignment][:format]} literal"
```

**Component breakdown**

- `$` — Interpolation prefix.
- `{expression}` — Expression to evaluate.
- `,alignment` — Optional signed minimum width (positive right-aligns, negative left-aligns).
- `:format` — Optional format specifier.

#### Syntax Rules

- The expression can be any C# expression returning a value.
- A `null` expression produces an empty string in the output.
- Literal braces are escaped by doubling: `{{` and `}}`.
- Interpolated strings can be combined with verbatim (`$@"..."`) or raw (`$$"""..."""`) forms.
- Since C# 10, interpolated strings whose holes are all constants can be `const`.
- Since C# 11, line breaks are allowed inside interpolation holes.

#### Constraints and Limitations

- On older frameworks or .NET Standard, interpolation lowers to `String.Format` with the associated overhead.
- Do not use interpolation to build SQL or shell commands directly (injection risk); use parameterized APIs.
- For high-performance logging, use message templates rather than pre-formatted interpolated strings.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Basic Interpolation

```csharp
using System;

class Program
{
    static void Main()
    {
        string name = "Alice";
        int age = 30;
        string message = $"Hello, my name is {name} and I am {age} years old.";
        Console.WriteLine(message);
    }
}
```

**Expected Output**

```
Hello, my name is Alice and I am 30 years old.
```

**Why this output occurs**

- Each hole is replaced by the expression’s value.
- The compiler generates efficient handler calls on modern .NET.

---

#### Example 2: Alignment and Formatting

```csharp
using System;

class Program
{
    static void Main()
    {
        double pi = Math.PI;
        Console.WriteLine($"|{pi,10:F3}|");   // right-aligned, 3 decimals
        Console.WriteLine($"|{pi,-10:F3}|");  // left-aligned
        Console.WriteLine($"{1234567:N0}");   // thousands separator
    }
}
```

**Expected Output**

```
|     3.142|
|3.142     |
1,234,567
```

**Why this output occurs**

- `,10` sets a minimum width of 10, right-aligning.
- `, -10` left-aligns.
- `:F3` formats to 3 decimal places.
- `:N0` adds group separators.

---

### Real-World Cases

- **User-facing messages** — Personalized greetings, confirmations.
- **Logging** — Structured messages (though message templates are preferred for performance).
- **Report generation** — Aligned tabular text.
- **Error messages** — Including variable values for diagnostics.

---

### References

- String interpolation — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/tokens/interpolated
- C# String Interpolation Explained - https://blog.ndepend.com/c-string-interpolation-explained/
- String Interpolation in C# 10 and .NET 6 - https://devblogs.microsoft.com/dotnet/string-interpolation-in-c-10-and-net-6/

---

## 7. `$"..."` Interpolation (Detailed Syntax)

### Definitions

**Core Definition**
`$"..."` is the syntactic form of an interpolated string literal in C#.

**Technical Definition**
The `$` character before a string literal marks it as interpolated. The compiler parses `{...}` sequences as interpolation holes and generates calls to an interpolated string handler (on .NET 6+) or `String.Format` (on older targets). The full hole grammar is `{expression[,alignment][:formatString]}`.

**Beginner-Friendly Explanation**
`$"..."` is the **standard way** to write an interpolated string. Put a dollar sign before the quotes, and anything inside `{}` becomes a computed value.

---

### Purposes

- To provide a concise, readable syntax for string composition.
- To enable inline formatting and alignment.
- To reduce errors compared to positional `String.Format`.
- To allow compile-time verification of embedded expressions.
- To integrate with modern handler-based compilation for performance.

---

### Syntax Rules and Structure

```
$"text {expression,alignment:format} text"
```

**Component breakdown**

- `$` — Required prefix.
- `{expression}` — Mandatory expression.
- `,alignment` — Optional; signed integer.
- `:format` — Optional; standard or custom format string.

#### Syntax Rules

- Literal braces are written `{{` and `}}`.
- `null` results produce empty strings.
- Ternary expressions inside holes must be parenthesized.
- Culture defaults to `CultureInfo.CurrentCulture`; use `string.Create` or `FormattableString` for invariant output.

#### Constraints and Limitations

- Interpolation is not a substitute for parameterized SQL or structured logging.
- On .NET Framework, interpolation lowers to `String.Format` and may box value types.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Ternary Inside Interpolation

```csharp
using System;

class Program
{
    static void Main()
    {
        int score = 1200;
        string message = $"Score: {score}, status: {(score > 1000 ? "high" : "low")}";
        Console.WriteLine(message);
    }
}
```

**Expected Output**

```
Score: 1200, status: high
```

**Why this output occurs**

- The ternary expression is wrapped in parentheses, so the `?` is not interpreted as a format separator.
- The result is inserted into the string.

---

#### Example 2: Escaping Braces

```csharp
using System;

class Program
{
    static void Main()
    {
        int count = 3;
        string message = $"{{This message includes {count} braces}}";
        Console.WriteLine(message);
    }
}
```

**Expected Output**

```
{This message includes 3 braces}
```

**Why this output occurs**

- `{{` and `}}` are literal brace escapes.
- `{count}` is the interpolation hole.

---

### Real-World Cases

- **UI text** — Dynamic labels and messages.
- **Diagnostics** — Including variable values in exceptions.
- **File naming** — Constructing filenames from parts.
- **Report headers** — Combining static text and computed values.

---

### References

- $ — string interpolation — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/tokens/interpolated
- String interpolation — C# Programming Guide - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/strings/#string-interpolation

---

## 8. String Interpolation Improvements (C# 10 and C# 11)

### Definitions

**Core Definition**
String interpolation improvements are language and runtime enhancements that make interpolated strings more capable and more efficient.

**Technical Definition**
C# 10 and .NET 6 introduced **interpolated string handlers** (`DefaultInterpolatedStringHandler`), which compile interpolation into direct append calls instead of `String.Format`, eliminating `object[]` allocation and boxing for many value types. C# 10 also allowed constant interpolated strings when all holes are constants. C# 11 added support for newlines inside interpolation holes and raw string interpolation (`$$"""..."""`).

**Beginner-Friendly Explanation**
Modern C# makes interpolated strings **faster and more flexible**. They no longer secretly create arrays and boxes, and you can write multi-line expressions and raw JSON-style strings more easily.

---

### Purposes

- To reduce allocations and improve throughput in string formatting.
- To allow `const` interpolated strings when all parts are constant.
- To improve readability of complex interpolation expressions.
- To support interpolation inside raw string literals without brace escaping.
- To enable custom interpolated string handlers for specialized formatting.

---

### Syntax Rules and Structure (What Changed)

```
// C# 10: constant interpolated string
const string Product = "Widget";
const string Version = "1.0";
const string Banner = $"{Product} v{Version}";   // valid since C# 10

// C# 11: newline inside interpolation hole
var s = $"{someValue +
           otherValue}";

// C# 11: raw interpolated string
var json = $$"""{"name": "{{name}}"}""";
```

**Component breakdown**

- Constant interpolation — All holes must be compile-time constants.
- Newline in holes — Expressions can span lines inside `{}`.
- Raw interpolation — `$$` changes hole delimiters to `{{...}}`.

#### Syntax Rules

- Constant interpolated strings require all expressions to be constants.
- Newlines inside holes are allowed in C# 11+.
- Raw interpolated strings use `$` count to determine brace escaping.

#### Constraints and Limitations

- Performance benefits depend on targeting .NET 6+; older frameworks still use `String.Format`.
- Custom handlers are advanced APIs; most developers use the default handler.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Constant Interpolated String (C# 10)

```csharp
using System;

class Program
{
    const string AppName = "MyApp";
    const string Version = "2.0";
    const string Banner = $"{AppName} v{Version}";

    static void Main()
    {
        Console.WriteLine(Banner);   // MyApp v2.0
    }
}
```

**Expected Output**

```
MyApp v2.0
```

**Why this output occurs**

- All holes are constants, so the entire interpolated string is a compile-time constant.

---

#### Example 2: Raw Interpolated String (C# 11)

```csharp
using System;

class Program
{
    static void Main()
    {
        string user = "Alice";
        string json = $$"""
        {
            "user": "{{user}}",
            "active": true
        }
        """;
        Console.WriteLine(json);
    }
}
```

**Expected Output**

```
{
    "user": "Alice",
    "active": true
}
```

**Why this output occurs**

- `$$` means double braces are interpolation holes; single braces are literal JSON structure.

---

### Real-World Cases

- **High-performance logging** — Handler-based interpolation reduces allocations.
- **JSON templating** — Raw interpolated strings keep braces readable.
- **Configuration constants** — Compile-time interpolated constants.
- **Complex expressions** — Multi-line holes improve readability.

---

### References

- String interpolation in C# 10 and .NET 6 - https://devblogs.microsoft.com/dotnet/string-interpolation-in-c-10-and-net-6/
- C# 10 — Constant interpolated strings - https://learn.microsoft.com/en-us/dotnet/csharp/whats-new/csharp-10#constant-interpolated-strings
- C# 11 — Newlines in string interpolations - https://learn.microsoft.com/en-us/dotnet/csharp/whats-new/csharp-11#newlines-in-string-interpolations
- C# String Interpolation Explained — NDepend - https://blog.ndepend.com/c-string-interpolation-explained/

---

## 9. String Methods

### Definitions

**Core Definition**
String methods are the instance and static methods on `System.String` that provide searching, extraction, modification, formatting, comparison, and validation operations.

**Technical Definition**
`System.String` exposes a large API surface. Instance methods such as `Substring`, `IndexOf`, `Replace`, and `Trim` operate on the current string and return new strings or values. Static methods such as `string.Join`, `string.Concat`, `string.Format`, `string.IsNullOrEmpty`, and `string.Compare` provide utility operations. Most methods have overloads that accept `StringComparison` to control culture sensitivity and case sensitivity.

**Beginner-Friendly Explanation**
String methods are the **tools in your text-processing toolbox**. They let you answer questions like “Where is this word?”, “What does this part say?”, “How do I clean up spaces?”, and “How do I split this into pieces?”

---

### Purposes

- To search for characters and substrings.
- To extract portions of a string.
- To replace or remove content.
- To change case or trim whitespace.
- To split strings into parts and join parts into strings.
- To pad strings for alignment.
- To format values as text.
- To validate empty or whitespace-only strings.
- To compare strings with controlled rules.

---

### Syntax Rules and Structure (General Pattern)

String methods follow standard instance or static method syntax:

```
result   =   stringInstance.MethodName(arguments)   ;
result   =   string.StaticMethodName(arguments)   ;
```

**Component breakdown**

- `stringInstance` — The string on which an instance method is called.
- `MethodName` — The operation (e.g., `Substring`, `IndexOf`).
- `arguments` — Inputs such as indices, search strings, or `StringComparison`.
- `string` — The type for static methods.

#### Syntax Rules

- Instance methods do not modify the original string; they return new values.
- Methods that search return indices (`int`) or booleans.
- Methods that transform return new strings.
- Overloads accepting `StringComparison` should be preferred for explicit control.
- Many methods throw `ArgumentNullException` or `ArgumentOutOfRangeException` for invalid inputs.

#### Constraints and Limitations

- `Substring` allocates a new string; use `AsSpan`/`Slice` for allocation-free slicing when possible.
- `IndexOf` and `Contains` default to culture-sensitive or ordinal depending on overload; be explicit.
- `Split` allocates arrays and strings; consider `Span`-based alternatives in hot paths.
- `Format` re-parses format strings at runtime; prefer interpolation on modern .NET.

---

Below, methods are grouped by functional category to keep the cheat sheet organized while preserving the uniform explanatory structure.

### 9.1 Length

**Definitions**

- **Core Definition:** `Length` is a property that returns the number of `char` objects in the string.
- **Technical Definition:** `String.Length` returns an `int` representing the number of UTF-16 code units. It does not count Unicode grapheme clusters or code points outside the Basic Multilingual Plane as single units.
- **Beginner-Friendly:** `Length` tells you how many characters the string contains (technically, how many UTF-16 units).

**Purposes**

- To determine the size of a string for loops, validation, and allocation.
- To check whether a string is empty (`Length == 0`).
- To compute indices for `Substring` and other operations.
- To compare string sizes.

**Syntax Rules and Structure**

```
int   length   =   stringVariable.Length   ;
```

**Constraints:** Counts code units, not grapheme clusters; surrogate pairs count as two.

**Annotated Example**

```csharp
using System;

class Program
{
    static void Main()
    {
        string text = "Hello";
        Console.WriteLine(text.Length);   // 5

        string emoji = "😀";
        Console.WriteLine(emoji.Length);  // 2 (surrogate pair)
    }
}
```

**Expected Output**

```
5
2
```

**Real-World Cases:** Input validation, loop bounds, substring calculations.

**References**

- String.Length — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.string.length

---

### 9.2 `Substring()`

**Definitions**

- **Core Definition:** Returns a substring starting at a specified index.
- **Technical Definition:** `Substring(int startIndex)` returns a new string from `startIndex` to the end; `Substring(int startIndex, int length)` returns a substring of the specified length. Throws `ArgumentOutOfRangeException` for invalid indices.
- **Beginner-Friendly:** `Substring` cuts out a piece of the string.

**Purposes**

- To extract a portion of text.
- To remove prefixes or suffixes.
- To parse fixed-width fields.
- To obtain a slice for further processing.

**Syntax Rules and Structure**

```
string   result   =   original.Substring(startIndex)   ;
string   result2   =   original.Substring(startIndex, length)   ;
```

**Constraints:** Allocates a new string; use `AsSpan().Slice()` for allocation-free slicing in performance-sensitive code.

**Annotated Example**

```csharp
using System;

class Program
{
    static void Main()
    {
        string text = "Hello, World!";
        Console.WriteLine(text.Substring(7));      // World!
        Console.WriteLine(text.Substring(0, 5));   // Hello
    }
}
```

**Expected Output**

```
World!
Hello
```

**Real-World Cases:** Parsing, extracting file extensions, removing prefixes.

**References**

- String.Substring — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.string.substring
- CA1846: Prefer AsSpan over Substring - https://learn.microsoft.com/en-us/dotnet/fundamentals/code-analysis/quality-rules/ca1846

---

### 9.3 `IndexOf()`

**Definitions**

- **Core Definition:** Returns the zero-based index of the first occurrence of a character or substring.
- **Technical Definition:** `IndexOf(char)` or `IndexOf(string)` searches from the beginning (or a specified start index) and returns the index, or `-1` if not found. Overloads accept `StringComparison` to control case and culture sensitivity.
- **Beginner-Friendly:** `IndexOf` tells you **where** something first appears.

**Purposes**

- To locate substrings for extraction or replacement.
- To check whether a string contains a value.
- To find delimiters for splitting.
- To search with culture or case rules.

**Syntax Rules and Structure**

```
int   index   =   text.IndexOf(searchValue)   ;
int   index2   =   text.IndexOf(searchValue, startIndex)   ;
int   index3   =   text.IndexOf(searchValue, StringComparison.Ordinal)   ;
```

**Constraints:** Returns `-1` if not found; always check before using the result.

**Annotated Example**

```csharp
using System;

class Program
{
    static void Main()
    {
        string text = "Hello, World!";
        Console.WriteLine(text.IndexOf("World"));   // 7
        Console.WriteLine(text.IndexOf("xyz"));     // -1
        Console.WriteLine(text.IndexOf("world", StringComparison.OrdinalIgnoreCase)); // 7
    }
}
```

**Expected Output**

```
7
-1
7
```

**Real-World Cases:** Parsing, searching, validating input.

**References**

- String.IndexOf — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.string.indexof

---

### 9.4 `LastIndexOf()`

**Definitions**

- **Core Definition:** Returns the index of the last occurrence of a character or substring.
- **Technical Definition:** Searches backward from the end (or a specified start index) and returns the index, or `-1`.
- **Beginner-Friendly:** `LastIndexOf` tells you **where something last appears**.

**Purposes**

- To find the last delimiter in a path or URL.
- To locate the final occurrence for replacement.
- To parse extensions or suffixes.

**Syntax Rules and Structure**

```
int   index   =   text.LastIndexOf(searchValue)   ;
```

**Constraints:** Returns `-1` if not found.

**Annotated Example**

```csharp
using System;

class Program
{
    static void Main()
    {
        string path = @"C:\folder\file.txt";
        Console.WriteLine(path.LastIndexOf('\\'));   // 8
        Console.WriteLine(path.LastIndexOf('.'));    // 14
    }
}
```

**Expected Output**

```
8
14
```

**Real-World Cases:** File path parsing, URL handling, suffix detection.

**References**

- String.LastIndexOf — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.string.lastindexof

---

### 9.5 `Replace()`

**Definitions**

- **Core Definition:** Returns a new string with all occurrences of a specified character or substring replaced.
- **Technical Definition:** `Replace(string oldValue, string newValue)` replaces all occurrences. `Replace(char, char)` replaces character occurrences. The original string is unchanged.
- **Beginner-Friendly:** `Replace` swaps parts of the text.

**Purposes**

- To substitute text.
- To sanitize input by removing or replacing characters.
- To normalize data.

**Syntax Rules and Structure**

```
string   result   =   text.Replace(oldValue, newValue)   ;
```

**Constraints:** Allocates a new string; if `oldValue` is not found, the original reference may be returned.

**Annotated Example**

```csharp
using System;

class Program
{
    static void Main()
    {
        string text = "Hello, World!";
        Console.WriteLine(text.Replace("World", "C#")); // Hello, C#!
        Console.WriteLine(text);                         // Hello, World!
    }
}
```

**Expected Output**

```
Hello, C#!
Hello, World!
```

**Real-World Cases:** Templating, sanitization, data normalization.

**References**

- String.Replace — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.string.replace

---

### 9.6 `ToUpper()` and `ToLower()`

**Definitions**

- **Core Definition:** Return a new string converted to uppercase or lowercase.
- **Technical Definition:** `ToUpper()` and `ToLower()` use the current culture’s casing rules unless an overload with `CultureInfo` is used. `ToUpperInvariant()` and `ToLowerInvariant()` use the invariant culture.
- **Beginner-Friendly:** These methods change the case of the letters.

**Purposes**

- To normalize text for case-insensitive comparison.
- To display text in a consistent case.
- To prepare data for storage or matching.

**Syntax Rules and Structure**

```
string   upper   =   text.ToUpper()   ;
string   lower   =   text.ToLower()   ;
string   invariantUpper   =   text.ToUpperInvariant()   ;
```

**Constraints:** Culture-sensitive; use invariant versions for stable comparisons.

**Annotated Example**

```csharp
using System;

class Program
{
    static void Main()
    {
        string text = "Hello";
        Console.WriteLine(text.ToUpper());   // HELLO
        Console.WriteLine(text.ToLower());   // hello
    }
}
```

**Expected Output**

```
HELLO
hello
```

**Real-World Cases:** Case-insensitive search, display formatting.

**References**

- String.ToUpper — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.string.toupper
- String.ToLower — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.string.tolower

---

### 9.7 `Trim()`, `TrimStart()`, and `TrimEnd()`

**Definitions**

- **Core Definition:** Remove leading and/or trailing whitespace (or specified characters).
- **Technical Definition:** `Trim()` removes whitespace from both ends. `TrimStart()` removes leading whitespace; `TrimEnd()` removes trailing whitespace. Overloads accept a `char[]` of characters to trim.
- **Beginner-Friendly:** These methods clean up spaces at the edges of a string.

**Purposes**

- To clean user input.
- To remove padding added for alignment.
- To normalize text for comparison.

**Syntax Rules and Structure**

```
string   trimmed   =   text.Trim()   ;
string   trimmedStart   =   text.TrimStart()   ;
string   trimmedEnd   =   text.TrimEnd()   ;
```

**Constraints:** Returns a new string; if nothing is trimmed, may return the original reference.

**Annotated Example**

```csharp
using System;

class Program
{
    static void Main()
    {
        string text = "  Hello  ";
        Console.WriteLine($"|{text.Trim()}|");       // |Hello|
        Console.WriteLine($"|{text.TrimStart()}|");  // |Hello  |
        Console.WriteLine($"|{text.TrimEnd()}|");    // |  Hello|
    }
}
```

**Expected Output**

```
|Hello|
|Hello  |
|  Hello|
```

**Real-World Cases:** Input validation, CSV parsing, user-supplied data.

**References**

- String.Trim — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.string.trim
- String.TrimStart — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.string.trimstart
- String.TrimEnd — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.string.trimend

---

### 9.8 `Split()`

**Definitions**

- **Core Definition:** Splits a string into an array of substrings based on delimiters.
- **Technical Definition:** `Split(char)` or `Split(string[])` divides the string at each delimiter. Overloads control the maximum number of substrings and whether empty entries are included.
- **Beginner-Friendly:** `Split` cuts a string into pieces wherever it sees a separator.

**Purposes**

- To parse CSV-like data.
- To separate words or tokens.
- To process lines from text.

**Syntax Rules and Structure**

```
string[]   parts   =   text.Split(delimiter)   ;
string[]   parts2   =   text.Split(new[] { ',', ';' }, StringSplitOptions.RemoveEmptyEntries)   ;
```

**Constraints:** Allocates an array and strings; consider `ReadOnlySpan<char>`-based splitting in hot paths.

**Annotated Example**

```csharp
using System;

class Program
{
    static void Main()
    {
        string csv = "apple,banana,cherry";
        string[] fruits = csv.Split(',');
        foreach (var f in fruits) Console.WriteLine(f);
    }
}
```

**Expected Output**

```
apple
banana
cherry
```

**Real-World Cases:** CSV parsing, command-line argument parsing, tokenization.

**References**

- String.Split — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.string.split

---

### 9.9 `Join()`

**Definitions**

- **Core Definition:** Concatenates elements of a collection into a single string, separated by a delimiter.
- **Technical Definition:** `string.Join(string separator, IEnumerable<string> values)` combines values with the separator between each pair. Overloads accept `string[]` and generic sequences.
- **Beginner-Friendly:** `Join` is the opposite of `Split`: it glues pieces together.

**Purposes**

- To build comma-separated lists.
- To reconstruct strings from arrays.
- To format collections for display.

**Syntax Rules and Structure**

```
string   joined   =   string.Join(separator, collection)   ;
```

**Constraints:** Does not add a trailing separator.

**Annotated Example**

```csharp
using System;

class Program
{
    static void Main()
    {
        string[] words = { "Hello", "World" };
        Console.WriteLine(string.Join(" ", words));   // Hello World
        Console.WriteLine(string.Join(",", words));   // Hello,World
    }
}
```

**Expected Output**

```
Hello World
Hello,World
```

**Real-World Cases:** Building query strings, CSV output, display lists.

**References**

- String.Join — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.string.join

---

### 9.10 `StartsWith()`, `EndsWith()`, `Contains()`

**Definitions**

- **Core Definition:** Boolean methods that test whether a string starts with, ends with, or contains a specified value.
- **Technical Definition:** `StartsWith`, `EndsWith`, and `Contains` return `bool` and have overloads accepting `StringComparison` to control case and culture sensitivity.
- **Beginner-Friendly:** These methods answer yes/no questions about the presence of a substring.

**Purposes**

- To validate prefixes and suffixes.
- To check for keywords or patterns.
- To filter strings based on content.

**Syntax Rules and Structure**

```
bool   result   =   text.StartsWith(prefix, StringComparison.Ordinal)   ;
bool   result2   =   text.EndsWith(suffix)   ;
bool   result3   =   text.Contains(substring)   ;
```

**Constraints:** Default overloads may be culture-sensitive; specify `StringComparison` for predictable behaviour.

**Annotated Example**

```csharp
using System;

class Program
{
    static void Main()
    {
        string url = "https://example.com";
        Console.WriteLine(url.StartsWith("https"));   // True
        Console.WriteLine(url.EndsWith(".com"));      // True
        Console.WriteLine(url.Contains("example"));   // True
    }
}
```

**Expected Output**

```
True
True
True
```

**Real-World Cases:** URL validation, file extension checks, keyword detection.

**References**

- String.StartsWith — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.string.startswith
- String.EndsWith — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.string.endswith
- String.Contains — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.string.contains

---

### 9.11 `PadLeft()` and `PadRight()`

**Definitions**

- **Core Definition:** Return a new string padded with spaces (or a specified character) to reach a total length.
- **Technical Definition:** `PadLeft(int totalWidth)` adds spaces on the left; `PadRight(int totalWidth)` adds spaces on the right. Overloads accept a `char` padding character.
- **Beginner-Friendly:** These methods align text by adding spaces.

**Purposes**

- To align columns in text output.
- To format fixed-width fields.
- To prepare data for parsing.

**Syntax Rules and Structure**

```
string   padded   =   text.PadLeft(totalWidth)   ;
string   padded2   =   text.PadRight(totalWidth, padChar)   ;
```

**Constraints:** If `totalWidth` is less than the string length, the original string is returned.

**Annotated Example**

```csharp
using System;

class Program
{
    static void Main()
    {
        string id = "42";
        Console.WriteLine($"[{id.PadLeft(5)}]");    // [   42]
        Console.WriteLine($"[{id.PadRight(5, '0')}]"); // [42000]
    }
}
```

**Expected Output**

```
[   42]
[42000]
```

**Real-World Cases:** Tabular output, fixed-width files, report generation.

**References**

- String.PadLeft — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.string.padleft
- String.PadRight — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.string.padright

---

### 9.12 `Format()`

**Definitions**

- **Core Definition:** A static method that replaces format placeholders in a composite format string with values.
- **Technical Definition:** `string.Format(format, arg0, arg1, ...)` substitutes `{0}`, `{1}`, etc. with the formatted values. Overloads accept `IFormatProvider` for culture control.
- **Beginner-Friendly:** `Format` is the older way to combine a template with values; interpolation is usually more readable.

**Purposes**

- To format values using composite format strings.
- To support format strings loaded from resources or configuration.
- To provide culture-aware formatting.

**Syntax Rules and Structure**

```
string   result   =   string.Format("Value: {0:F2}", value)   ;
```

**Constraints:** Parses the format string at runtime; prefer interpolation on modern .NET for performance.

**Annotated Example**

```csharp
using System;

class Program
{
    static void Main()
    {
        Console.WriteLine(string.Format("{0} + {1} = {2}", 1, 2, 3));
    }
}
```

**Expected Output**

```
1 + 2 = 3
```

**Real-World Cases:** Localized templates, legacy code, dynamic format strings.

**References**

- String.Format — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.string.format

---

### 9.13 `IsNullOrEmpty()` and `IsNullOrWhiteSpace()`

**Definitions**

- **Core Definition:** Static methods that test whether a string is `null`, empty, or whitespace-only.
- **Technical Definition:** `string.IsNullOrEmpty` returns `true` for `null` or `""`. `string.IsNullOrWhiteSpace` also returns `true` for strings containing only whitespace.
- **Beginner-Friendly:** These methods safely check whether a string has meaningful content.

**Purposes**

- To validate user input.
- To avoid `NullReferenceException`.
- To check for blank fields.

**Syntax Rules and Structure**

```
bool   empty   =   string.IsNullOrEmpty(text)   ;
bool   blank   =   string.IsNullOrWhiteSpace(text)   ;
```

**Constraints:** Neither method trims the string; `IsNullOrWhiteSpace` considers `\t`, `\n`, and other Unicode whitespace.

**Annotated Example**

```csharp
using System;

class Program
{
    static void Main()
    {
        string? a = null;
        string b = "";
        string c = "   ";

        Console.WriteLine(string.IsNullOrEmpty(a));       // True
        Console.WriteLine(string.IsNullOrEmpty(c));       // False
        Console.WriteLine(string.IsNullOrWhiteSpace(c));  // True
    }
}
```

**Expected Output**

```
True
False
True
```

**Real-World Cases:** Form validation, configuration checks, null-safe processing.

**References**

- String.IsNullOrEmpty — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.string.isnullorempty
- String.IsNullOrWhiteSpace — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.string.isnullorwhitespace

---

### 9.14 `Concat()`

**Definitions**

- **Core Definition:** Static method that concatenates one or more strings.
- **Technical Definition:** `string.Concat` combines arguments in order. Overloads accept `object`, `string[]`, `IEnumerable<string>`, etc. `null` arguments are treated as empty strings.
- **Beginner-Friendly:** `Concat` glues strings together.

**Purposes**

- To combine strings without `+` operators.
- To concatenate collections of strings.
- To safely handle `null` values.

**Syntax Rules and Structure**

```
string   result   =   string.Concat(a, b, c)   ;
string   result2   =   string.Concat(collection)   ;
```

**Constraints:** Repeated concatenation in loops can be inefficient; use `StringBuilder`.

**Annotated Example**

```csharp
using System;

class Program
{
    static void Main()
    {
        Console.WriteLine(string.Concat("Hello", " ", "World")); // Hello World
    }
}
```

**Expected Output**

```
Hello World
```

**Real-World Cases:** Building simple messages, combining parts.

**References**

- String.Concat — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.string.concat

---

### 9.15 `Compare()` and `CompareTo()`

**Definitions**

- **Core Definition:** Methods that compare two strings and return an integer indicating their relative order.
- **Technical Definition:** `string.Compare(a, b)` and `a.CompareTo(b)` return a negative number if `a < b`, zero if equal, and a positive number if `a > b`. Overloads accept `StringComparison` and `CultureInfo`.
- **Beginner-Friendly:** These methods tell you how two strings are ordered (for sorting).

**Purposes**

- To sort strings.
- To determine lexical order.
- To implement custom comparisons.

**Syntax Rules and Structure**

```
int   order   =   string.Compare(a, b, StringComparison.Ordinal)   ;
int   order2   =   a.CompareTo(b)   ;
```

**Constraints:** Do not use `CompareTo` to test equality; use `Equals` instead.

**Annotated Example**

```csharp
using System;

class Program
{
    static void Main()
    {
        Console.WriteLine(string.Compare("apple", "banana"));   // -1
        Console.WriteLine(string.Compare("apple", "apple"));    // 0
        Console.WriteLine(string.Compare("banana", "apple"));   // 1
    }
}
```

**Expected Output**

```
-1
0
1
```

**Real-World Cases:** Sorting lists, ordering output.

**References**

- String.Compare — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.string.compare
- String.CompareTo — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.string.comparetoto

---

### 9.16 `Equals()`

**Definitions**

- **Core Definition:** Determines whether two strings have the same value.
- **Technical Definition:** `string.Equals` returns `bool`. Overloads accept `StringComparison` to control case and culture sensitivity. The `==` operator delegates to `Equals` for strings.
- **Beginner-Friendly:** `Equals` asks “are these two strings the same text?”

**Purposes**

- To compare strings for equality.
- To perform case-insensitive or culture-aware equality checks.
- To avoid reference equality confusion.

**Syntax Rules and Structure**

```
bool   equal   =   a.Equals(b)   ;
bool   equal2   =   string.Equals(a, b, StringComparison.OrdinalIgnoreCase)   ;
```

**Constraints:** `Equals` is case-sensitive by default; use `StringComparison` for control.

**Annotated Example**

```csharp
using System;

class Program
{
    static void Main()
    {
        Console.WriteLine("Hello".Equals("Hello"));   // True
        Console.WriteLine("Hello".Equals("hello"));   // False
        Console.WriteLine(string.Equals("Hello", "hello", StringComparison.OrdinalIgnoreCase)); // True
    }
}
```

**Expected Output**

```
True
False
True
```

**Real-World Cases:** Authentication, configuration comparison, validation.

**References**

- String.Equals — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.string.equals

---

## 10. `StringBuilder`

### Definitions

**Core Definition**
`StringBuilder` is a mutable sequence of characters in the `System.Text` namespace, designed for efficient repeated string manipulation.

**Technical Definition**
`System.Text.StringBuilder` maintains an internal buffer that can grow and shrink without creating a new object for every modification. Methods such as `Append`, `Insert`, `Replace`, and `Remove` modify the builder in place and return the same instance, enabling fluent chaining. `ToString()` produces the final immutable string. `StringBuilder` is ideal when the number of modifications is unknown or large.

**Beginner-Friendly Explanation**
`StringBuilder` is like a **notepad** that you can edit freely. Instead of creating a new string every time you add or change text, you write on the same notepad and only convert it to a final string when you are done.

---

### Purposes

- To efficiently build strings through many append/insert/replace operations.
- To avoid the allocation overhead of immutable string concatenation.
- To support loops that accumulate text.
- To provide a mutable buffer for complex formatting.
- To reduce garbage-collection pressure in string-heavy applications.

---

### Syntax Rules and Structure

```
using System.Text;

var   sb   =   new   StringBuilder()   ;
sb.Append(value)   ;
sb.AppendLine(text)   ;
sb.Insert(index, value)   ;
sb.Replace(oldValue, newValue)   ;
sb.Remove(startIndex, length)   ;
string   result   =   sb.ToString()   ;
```

**Component breakdown**

- `StringBuilder()` — Creates an empty builder with a default capacity.
- `Append` — Adds text to the end.
- `AppendLine` — Adds text followed by a newline.
- `Insert` — Inserts text at an index.
- `Replace` — Replaces occurrences.
- `Remove` — Deletes a range.
- `ToString()` — Produces the final string.

#### Syntax Rules

- `StringBuilder` is mutable; methods return the same instance.
- Capacity grows automatically but can be pre-sized for performance.
- `StringBuilder` is not thread-safe; use synchronization or separate builders for concurrent use.
- `ToString()` creates a snapshot; further modifications do not affect the returned string.

#### Constraints and Limitations

- `StringBuilder` lacks search methods such as `IndexOf` and `StartsWith`; converting to a string negates some performance benefits.
- For a small number of concatenations, `StringBuilder` may offer no benefit over `string`.
- `StringBuilder` is a reference type and must be allocated.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Building a String in a Loop

```csharp
using System;
using System.Text;

class Program
{
    static void Main()
    {
        var sb = new StringBuilder();
        for (int i = 1; i <= 5; i++)
        {
            sb.Append(i);
            if (i < 5) sb.Append(", ");
        }
        Console.WriteLine(sb.ToString());   // 1, 2, 3, 4, 5
    }
}
```

**Expected Output**

```
1, 2, 3, 4, 5
```

**Why this output occurs**

- `StringBuilder` mutates its internal buffer instead of creating new strings.
- `ToString()` produces the final string.

---

#### Example 2: `AppendFormat` for Structured Output

```csharp
using System;
using System.Text;

class Program
{
    static void Main()
    {
        var sb = new StringBuilder();
        sb.AppendFormat("Name: {0}, Age: {1}", "Alice", 30);
        sb.AppendLine();
        sb.Append("Done");
        Console.WriteLine(sb.ToString());
    }
}
```

**Expected Output**

```
Name: Alice, Age: 30
Done
```

**Why this output occurs**

- `AppendFormat` formats values into the builder.
- `AppendLine` adds a newline.

---

### Real-World Cases

- **SQL query building** — Accumulating clauses in a loop (with parameterization for safety).
- **HTML/XML generation** — Building markup incrementally.
- **Log message assembly** — Concatenating many parts.
- **Report generation** — Building large text outputs.

---

### References

- StringBuilder — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.text.stringbuilder
- String vs. StringBuilder - https://github.com/skylinecommunications/dataminer-docs/blob/573e18c727d6935f48c1a4d57e34ab039702f997/develop/codingguidelines/Protocol/String_vs_StringBuilder.md

---

## 11. String Performance

### Definitions

**Core Definition**
String performance refers to the CPU time, memory allocations, and garbage-collection impact of string operations.

**Technical Definition**
Because strings are immutable, operations that appear to modify a string allocate new string objects. Repeated concatenation in loops creates many temporary strings, increasing GC pressure. `StringBuilder` reduces allocations for many modifications. On modern .NET, interpolated strings compile to handler-based code that avoids `object[]` allocation and boxing. `Span<char>` and `ReadOnlySpan<char>` enable allocation-free slicing, searching, and parsing. UTF-8 literals (`u8`) avoid transcoding overhead when interoperating with UTF-8 APIs.

**Beginner-Friendly Explanation**
String operations can be fast or slow depending on how many new strings they create. The more temporary strings you create, the harder the garbage collector has to work. Using `StringBuilder`, spans, and modern interpolation keeps things efficient.

---

### Purposes

- To minimize allocations and GC pressure in string-heavy code.
- To choose the right tool (`string`, `StringBuilder`, `Span<char>`) for the job.
- To avoid boxing and intermediate strings in formatting.
- To reduce transcoding overhead with UTF-8 literals.
- To identify and fix string-related performance bottlenecks.

---

### Syntax Rules and Structure (Performance Patterns)

```
// Avoid repeated concatenation in loops
string result = "";
for (...) result += value;   // bad: many allocations

// Prefer StringBuilder for many modifications
var sb = new StringBuilder();
for (...) sb.Append(value);  // good

// Use spans for allocation-free slicing
ReadOnlySpan<char> slice = text.AsSpan(start, length);  // no allocation

// Use UTF-8 literals for UTF-8 APIs
ReadOnlySpan<byte> utf8 = "Hello"u8;   // no transcoding
```

**Component breakdown**

- `+=` — Creates a new string each time.
- `StringBuilder.Append` — Mutates the buffer.
- `AsSpan()` — Returns a view without copying.
- `u8` — Produces UTF-8 bytes at compile time.

#### Syntax Rules

- Use `StringBuilder` when the number of modifications is large or unknown.
- Use `Span<char>` / `ReadOnlySpan<char>` for parsing and slicing without allocation.
- Use interpolation on .NET 6+ for efficient formatting.
- Use `StringComparison` overloads to avoid culture-sensitive overhead when not needed.

#### Constraints and Limitations

- `StringBuilder` has overhead for small numbers of concatenations.
- Spans are `ref struct` types and cannot be stored on the heap or used across `await` boundaries in some contexts.
- UTF-8 literals are `ReadOnlySpan<byte>` and cannot be boxed.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Concatenation vs. `StringBuilder`

```csharp
using System;
using System.Diagnostics;
using System.Text;

class Program
{
    static void Main()
    {
        const int N = 10_000;

        var sw = Stopwatch.StartNew();
        string s = "";
        for (int i = 0; i < N; i++) s += i;
        sw.Stop();
        Console.WriteLine($"Concat: {sw.ElapsedMilliseconds} ms");

        sw.Restart();
        var sb = new StringBuilder();
        for (int i = 0; i < N; i++) sb.Append(i);
        string s2 = sb.ToString();
        sw.Stop();
        Console.WriteLine($"StringBuilder: {sw.ElapsedMilliseconds} ms");
    }
}
```

**Expected Output (approximate)**

```
Concat: 120 ms
StringBuilder: 2 ms
```

**Why this output occurs**

- `+=` creates a new string on each iteration, causing many allocations and copies.
- `StringBuilder` appends to an internal buffer, growing it as needed.
- The performance difference grows with `N`.

---

#### Example 2: Allocation-Free Slicing with `AsSpan`

```csharp
using System;

class Program
{
    static void Main()
    {
        string text = "Hello, World!";
        ReadOnlySpan<char> slice = text.AsSpan(7, 5);
        Console.WriteLine(slice.ToString());   // World
    }
}
```

**Expected Output**

```
World
```

**Why this output occurs**

- `AsSpan` returns a view over the existing string without allocating a new string.
- `ToString()` is only needed for console output.

---

### Real-World Cases

- **High-throughput parsers** — Using spans to avoid substring allocations.
- **Logging systems** — Efficient message formatting.
- **Web servers** — Reducing allocations per request.
- **Game engines** — Minimizing GC pauses.

---

### References

- String vs. StringBuilder — Microsoft Learn (performance insights) - https://learn.microsoft.com/en-us/dotnet/fundamentals/code-analysis/quality-rules/ca1846
- String Interpolation in C# 10 and .NET 6 — Performance - https://devblogs.microsoft.com/dotnet/string-interpolation-in-c-10-and-net-6/
- Write safe and efficient C# code — Performance - https://learn.microsoft.com/en-us/dotnet/csharp/advanced-topics/performance/

---

## 12. String Best Practices

### Definitions

**Core Definition**
String best practices are the guidelines for using strings correctly, safely, and efficiently in C#.

**Technical Definition**
Best practices encompass using explicit `StringComparison`, preferring `StringBuilder` for heavy modification, avoiding unnecessary allocations with spans, using `IsNullOrEmpty`/`IsNullOrWhiteSpace` for validation, preferring `string.Empty` over `""`, using verbatim/raw literals for structured text, and avoiding culture-sensitive defaults when ordinal comparison is intended.

**Beginner-Friendly Explanation**
Best practices are the habits that keep string code fast, correct, and safe: be explicit about comparison rules, avoid creating too many temporary strings, and validate before using a string.

---

### Purposes

- To avoid subtle culture-related bugs.
- To reduce allocations and GC pressure.
- To improve readability and maintainability.
- To prevent null-reference exceptions.
- To ensure predictable, secure comparisons.

---

### Guidelines and Structure

#### General Guidelines

```
// Use explicit StringComparison
if (text.Equals("value", StringComparison.Ordinal)) { ... }

// Use StringBuilder for many appends
var sb = new StringBuilder();

// Use spans for slicing
ReadOnlySpan<char> part = text.AsSpan(start, length);

// Validate before use
if (string.IsNullOrWhiteSpace(input)) return;

// Prefer string.Empty
string empty = string.Empty;

// Use verbatim/raw literals for structured text
string json = """{ "key": "value" }""";
```

**Component breakdown**

- `StringComparison` — Explicit control over case and culture.
- `StringBuilder` — Mutable buffer for many modifications.
- `Span` — Allocation-free views.
- Validation methods — Safe null/empty checks.
- Literal choices — Readability and escaping avoidance.

#### Rules and Constraints

- Always specify `StringComparison` for equality, `StartsWith`, `EndsWith`, `IndexOf`, etc..
- Use `Ordinal` or `OrdinalIgnoreCase` for non-linguistic comparisons (identifiers, paths, keys).
- Use `CurrentCulture` for user-facing text that should follow locale rules.
- Avoid `CompareTo` for equality testing.
- Use `StringBuilder` when the number of modifications is unknown or large.
- Prefer `AsSpan` over `Substring` in hot paths.
- Use `String.IsNullOrWhiteSpace` for user input validation.
- Prefer `string.Empty` over `""`.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Explicit Comparison

```csharp
using System;

class Program
{
    static void Main()
    {
        string input = "HELLO";
        Console.WriteLine(input.Equals("hello", StringComparison.OrdinalIgnoreCase)); // True
    }
}
```

**Expected Output**

```
True
```

**Why this output occurs**

- `OrdinalIgnoreCase` compares characters by ordinal value while ignoring case, independent of culture.

---

#### Example 2: Safe Input Validation

```csharp
using System;

class Program
{
    static void Main()
    {
        string? input = null;
        if (string.IsNullOrWhiteSpace(input))
        {
            Console.WriteLine("Input is missing");
        }
    }
}
```

**Expected Output**

```
Input is missing
```

**Why this output occurs**

- `IsNullOrWhiteSpace` safely handles `null` and whitespace-only strings.

---

### Real-World Cases

- **Security-sensitive comparisons** — Using `Ordinal` to avoid culture tricks.
- **User input validation** — `IsNullOrWhiteSpace` before processing.
- **High-performance APIs** — Spans and `StringBuilder`.

---

### References

- Best Practices for Comparing Strings in .NET - https://learn.microsoft.com/en-us/dotnet/standard/base-types/best-practices-strings
- String vs. StringBuilder - https://github.com/skylinecommunications/dataminer-docs/blob/573e18c727d6935f48c1a4d57e34ab039702f997/develop/codingguidelines/Protocol/String_vs_StringBuilder.md
- CA1846: Prefer AsSpan over Substring - https://learn.microsoft.com/en-us/dotnet/fundamentals/code-analysis/quality-rules/ca1846

---

## 13. `StringComparison`

### Definitions

**Core Definition**
`StringComparison` is an enumeration that specifies the rules for comparing strings.

**Technical Definition**
`System.StringComparison` has six members: `CurrentCulture`, `CurrentCultureIgnoreCase`, `InvariantCulture`, `InvariantCultureIgnoreCase`, `Ordinal`, and `OrdinalIgnoreCase`. `Ordinal` compares the numeric values of UTF-16 code units; culture-sensitive options use linguistic rules of the current or invariant culture. Using explicit `StringComparison` avoids ambiguity and improves performance and security.

**Beginner-Friendly Explanation**
`StringComparison` tells the comparison method **which rules to follow**: case-sensitive or not, and whether to use cultural language rules or simple character-by-character comparison.

---

### Purposes

- To make comparison intent explicit.
- To control case sensitivity.
- To choose between linguistic and ordinal comparison.
- To avoid culture-related bugs.
- To improve performance by selecting the cheapest appropriate comparison.

---

### Syntax Rules and Structure

```
StringComparison.Ordinal
StringComparison.OrdinalIgnoreCase
StringComparison.CurrentCulture
StringComparison.CurrentCultureIgnoreCase
StringComparison.InvariantCulture
StringComparison.InvariantCultureIgnoreCase
```

**Component breakdown**

- `Ordinal` — Byte-by-byte comparison of UTF-16 code units.
- `OrdinalIgnoreCase` — Ordinal with case folding.
- `CurrentCulture` — Linguistic rules of the current culture.
- `InvariantCulture` — Linguistic rules of the invariant culture.

#### Syntax Rules

- Pass `StringComparison` to methods like `Equals`, `Compare`, `StartsWith`, `IndexOf`.
- Defaults vary: `Equals` is ordinal by default for `string`, while `Compare` and `StartsWith` may be culture-sensitive.
- Use `Ordinal` for identifiers, paths, and security-sensitive comparisons.

#### Constraints and Limitations

- Culture-sensitive comparisons are slower and platform-dependent.
- `InvariantCulture` is not a “safe default” for all linguistic data; it is mainly for stable cross-cultural ordering.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Ordinal vs. Culture Comparison

```csharp
using System;
using System.Globalization;

class Program
{
    static void Main()
    {
        CultureInfo.CurrentCulture = new CultureInfo("en-US");
        Console.WriteLine(string.Compare("æ", "ae", StringComparison.Ordinal));       // 1 (different)
        Console.WriteLine(string.Compare("æ", "ae", StringComparison.InvariantCulture)); // -1 (linguistically ordered)
    }
}
```

**Expected Output**

```
1
-1
```

**Why this output occurs**

- Ordinal compares code units; `æ` and `ae` are different.
- Invariant culture applies linguistic rules where `æ` may sort before `ae`.

---

#### Example 2: Case-Insensitive Ordinal

```csharp
using System;

class Program
{
    static void Main()
    {
        Console.WriteLine("Hello".Equals("HELLO", StringComparison.OrdinalIgnoreCase)); // True
    }
}
```

**Expected Output**

```
True
```

**Why this output occurs**

- `OrdinalIgnoreCase` ignores case using ordinal rules.

---

### Real-World Cases

- **File paths** — `Ordinal` or `OrdinalIgnoreCase` on Windows.
- **Configuration keys** — `Ordinal`.
- **User-visible sorting** — `CurrentCulture`.
- **Security tokens** — `Ordinal` to avoid culture tricks.

---

### References

- StringComparison — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.stringcomparison
- Best Practices for Comparing Strings in .NET - https://learn.microsoft.com/en-us/dotnet/standard/base-types/best-practices-strings
- How to compare strings in C# - https://learn.microsoft.com/en-us/dotnet/csharp/how-to/compare-strings

---

## 14. Culture-Sensitive Comparison

### Definitions

**Core Definition**
Culture-sensitive comparison uses the linguistic rules of a specific culture to determine string equality and ordering.

**Technical Definition**
Culture-sensitive comparison considers language-specific rules such as casing, accent handling, and sort order. `StringComparison.CurrentCulture` and `InvariantCulture` enable linguistic comparison. `String.Compare` and `CompareTo` perform culture-sensitive comparisons by default using the current culture. Culture-sensitive comparison can produce different results on different platforms and locales.

**Beginner-Friendly Explanation**
Culture-sensitive comparison asks: “How would a human from this culture order these words?” It understands that `æ` might be treated like `ae` in some languages, and that uppercase/lowercase rules vary.

---

### Purposes

- To sort text according to the user’s language rules.
- To compare user-facing text correctly.
- To support internationalization and localization.
- To handle accented characters and ligatures appropriately.
- To respect cultural casing conventions.

---

### Syntax Rules and Structure

```
string.Compare(a, b, StringComparison.CurrentCulture)
a.Equals(b, StringComparison.CurrentCultureIgnoreCase)
```

**Component breakdown**

- `CurrentCulture` — Uses the current thread’s culture.
- `CurrentCultureIgnoreCase` — Case-insensitive linguistic comparison.
- `InvariantCulture` — Uses a neutral, stable culture.

#### Syntax Rules

- Culture-sensitive methods use `Thread.CurrentThread.CurrentCulture` by default.
- Results can vary by .NET version and operating system (ICU vs. NLS).
- Use `InvariantCulture` for stable cross-cultural ordering, not for user-facing sorting.

#### Constraints and Limitations

- Culture-sensitive comparison is slower than ordinal.
- Behavior can change across .NET versions and platforms.
- Not suitable for identifiers, paths, or security tokens.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Sorting with Current Culture

```csharp
using System;
using System.Globalization;
using System.Linq;

class Program
{
    static void Main()
    {
        CultureInfo.CurrentCulture = new CultureInfo("sv-SE"); // Swedish
        string[] words = { "zebra", "äpple", "apple" };
        var sorted = words.OrderBy(w => w, StringComparer.CurrentCulture);
        Console.WriteLine(string.Join(", ", sorted));
    }
}
```

**Expected Output (Swedish)**

```
apple, zebra, äpple
```

**Why this output occurs**

- In Swedish, `ä` sorts after `z`, so `äpple` comes last.
- Ordinal sorting would place `äpple` before `zebra` because `ä` has a higher code unit.

---

#### Example 2: Culture-Sensitive Equality

```csharp
using System;
using System.Globalization;

class Program
{
    static void Main()
    {
        CultureInfo.CurrentCulture = new CultureInfo("tr-TR"); // Turkish
        Console.WriteLine("FILE".Equals("file", StringComparison.CurrentCultureIgnoreCase)); // False
    }
}
```

**Expected Output**

```
False
```

**Why this output occurs**

- In Turkish, the uppercase of `i` is `İ`, not `I`, so `FILE` and `file` are not case-insensitive equal under Turkish rules.

---

### Real-World Cases

- **UI sorting** — Sorting names and labels by user locale.
- **Localized search** — Matching accented characters.
- **Report generation** — Ordered output for a specific culture.

---

### References

- Comparing strings — C# how-to - https://learn.microsoft.com/en-us/dotnet/csharp/how-to/compare-strings
- Best Practices for Comparing Strings in .NET - https://learn.microsoft.com/en-us/dotnet/standard/base-types/best-practices-strings

---

## 15. Unicode

### Definitions

**Core Definition**
Unicode is a universal character encoding standard that assigns a unique code point to every character in every writing system.

**Technical Definition**
Unicode defines code points (e.g., `U+0041` for `A`) and encoding forms (UTF-8, UTF-16, UTF-32). C# `char` represents a UTF-16 code unit; `string` is a sequence of UTF-16 code units. Characters outside the Basic Multilingual Plane (BMP) are represented by **surrogate pairs** (two `char` values). `System.Text.Rune` and `System.Globalization.StringInfo` provide APIs for working with Unicode code points and grapheme clusters.

**Beginner-Friendly Explanation**
Unicode is the **global alphabet** that includes letters, digits, symbols, and emoji from every language. C# stores text as UTF-16, so some characters (like emoji) take two `char` slots.

---

### Purposes

- To represent text from all writing systems consistently.
- To enable internationalized applications.
- To support emoji and supplementary characters.
- To provide a standard basis for encoding conversions.
- To avoid data corruption across platforms.

---

### Syntax Rules and Structure

Unicode is a conceptual foundation; syntax involves character literals and encoding APIs:

```
char   c   =   'A';               // UTF-16 code unit
string   s   =   "😀";            // surrogate pair (2 chars)
Rune   r   =   Rune.GetRuneAt(s, 0); // Unicode code point
```

**Component breakdown**

- `char` — One UTF-16 code unit.
- `string` — Sequence of UTF-16 code units.
- `Rune` — A Unicode scalar value (code point).

#### Syntax Rules

- `char` cannot represent code points above `U+FFFF` as a single unit.
- `Length` counts code units, not code points or grapheme clusters.
- Use `StringInfo` or `Rune` for code-point-aware operations.

#### Constraints and Limitations

- Grapheme clusters (user-perceived characters) can consist of multiple code points (e.g., emoji with skin tone modifiers).
- `Substring` may split a surrogate pair, producing invalid text.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Surrogate Pairs

```csharp
using System;

class Program
{
    static void Main()
    {
        string emoji = "😀";
        Console.WriteLine(emoji.Length);             // 2
        Console.WriteLine(char.IsSurrogate(emoji[0])); // True

        Rune rune = Rune.GetRuneAt(emoji, 0);
        Console.WriteLine(rune.Value);               // 128512
    }
}
```

**Expected Output**

```
2
True
128512
```

**Why this output occurs**

- The emoji is outside the BMP, so it is stored as a surrogate pair (2 `char` values).
- `Rune` retrieves the Unicode code point (`U+1F600` = 128512).

---

#### Example 2: Grapheme Clusters

```csharp
using System;
using System.Globalization;

class Program
{
    static void Main()
    {
        string text = "e\u0301";   // e + combining acute accent
        Console.WriteLine(text.Length);                    // 2
        var enumerator = StringInfo.GetTextElementEnumerator(text);
        while (enumerator.MoveNext())
            Console.WriteLine(enumerator.GetTextElement()); // é
    }
}
```

**Expected Output**

```
2
é
```

**Why this output occurs**

- The string has two code points (`e` and combining accent) but one grapheme cluster (`é`).
- `StringInfo` enumerates grapheme clusters correctly.

---

### Real-World Cases

- **Internationalized UIs** — Displaying names in multiple scripts.
- **Emoji support** — Storing and processing emoji correctly.
- **Text normalization** — Comparing equivalent Unicode representations.
- **Security** — Preventing homograph attacks (visually similar characters).

---

### References

- Character encoding in .NET — Unicode, UTF-16, UTF-8 - https://learn.microsoft.com/en-us/dotnet/standard/base-types/character-encoding-introduction
- Rune — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.text.rune
- StringInfo — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.globalization.stringinfo

---

## 16. UTF-8

### Definitions

**Core Definition**
UTF-8 is a variable-length encoding for Unicode that uses 1–4 bytes per code point and is the dominant encoding on the web.

**Technical Definition**
UTF-8 encodes Unicode code points using 1 byte for ASCII, 2 bytes for many European characters, 3 bytes for most CJK characters, and 4 bytes for supplementary characters and emoji. C# `string` is UTF-16 internally, but UTF-8 literals (`"..."u8`) produce `ReadOnlySpan<byte>` containing UTF-8 bytes, avoiding transcoding overhead when interoperating with UTF-8 APIs.

**Beginner-Friendly Explanation**
UTF-8 is a **compact way to store text as bytes**, especially for English and web content. C# normally stores strings as UTF-16, but you can create UTF-8 bytes directly with the `u8` suffix.

---

### Purposes

- To interoperate with web protocols, JSON, and UTF-8 APIs.
- To reduce storage size for ASCII-heavy text.
- To avoid transcoding overhead when writing to UTF-8 streams.
- To produce byte spans for low-level processing.
- To support international text with variable-width encoding.

---

### Syntax Rules and Structure

```
ReadOnlySpan<byte>   utf8   =   "Hello"u8   ;
byte[]   bytes   =   Encoding.UTF8.GetBytes("Hello")   ;
string   text   =   Encoding.UTF8.GetString(bytes)   ;
```

**Component breakdown**

- `"..."u8` — UTF-8 string literal (C# 11+).
- `Encoding.UTF8.GetBytes` — Runtime conversion from string to UTF-8 bytes.
- `Encoding.UTF8.GetString` — Runtime conversion from UTF-8 bytes to string.

#### Syntax Rules

- `u8` literals are `ReadOnlySpan<byte>` and cannot be boxed.
- UTF-8 literals require C# 11+.
- `Encoding.UTF8` handles runtime conversions with error handling options.

#### Constraints and Limitations

- `u8` literals are not `string` values; they are byte spans.
- `ReadOnlySpan<byte>` is a `ref struct` and cannot be stored on the heap.
- UTF-8 encoding can throw on invalid sequences unless configured otherwise.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: UTF-8 Literal

```csharp
using System;

class Program
{
    static void Main()
    {
        ReadOnlySpan<byte> utf8 = "Hello"u8;
        Console.WriteLine(utf8.Length);          // 5
        Console.WriteLine(utf8[0]);              // 72 ('H')
    }
}
```

**Expected Output**

```
5
72
```

**Why this output occurs**

- `"Hello"u8` produces the UTF-8 bytes for `Hello`, which are identical to ASCII for these characters.
- Indexing returns the byte value.

---

#### Example 2: Runtime Conversion

```csharp
using System;
using System.Text;

class Program
{
    static void Main()
    {
        string text = "café";
        byte[] utf8Bytes = Encoding.UTF8.GetBytes(text);
        Console.WriteLine(utf8Bytes.Length);      // 5
        string roundTrip = Encoding.UTF8.GetString(utf8Bytes);
        Console.WriteLine(roundTrip);              // café
    }
}
```

**Expected Output**

```
5
café
```

**Why this output occurs**

- `é` requires 2 UTF-8 bytes, so the total is 5 bytes for 4 characters.
- `GetString` decodes the bytes back to the original string.

---

### Real-World Cases

- **JSON serialization** — UTF-8 is the default encoding for JSON.
- **HTTP APIs** — Request and response bodies are often UTF-8.
- **File I/O** — Writing UTF-8 text files.
- **Interop with native libraries** — Passing UTF-8 byte spans.

---

### References

- UTF-8 string literals — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/reference-types#utf-8-string-literals
- Welcome to C# 11 — UTF-8 string literals - https://devblogs.microsoft.com/dotnet/welcome-to-csharp-11/#utf-8-string-literals
- Encoding.UTF8 — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.text.encoding.utf8

---

## 17. Encoding

### Definitions

**Core Definition**
Encoding is the process of converting text (strings) to bytes and bytes back to text using a character encoding scheme.

**Technical Definition**
`System.Text.Encoding` is an abstract class that provides methods for converting between `string` and `byte[]`. Common encodings include `UTF8`, `Unicode` (UTF-16 LE), `UTF32`, `ASCII`, and `Latin1`. `GetBytes` converts a string to bytes; `GetString` converts bytes to a string. Encoding choices affect size, compatibility, and correctness for non-ASCII text.

**Beginner-Friendly Explanation**
Encoding is the **translation layer** between text and bytes. Computers store bytes, but humans read text; encoding defines how to map between the two.

---

### Purposes

- To serialize text to bytes for storage or transmission.
- To deserialize bytes back to text.
- To interoperate with files, networks, and APIs that expect specific encodings.
- To handle international characters correctly.
- To avoid data corruption from mismatched encodings.

---

### Syntax Rules and Structure

```
using System.Text;

byte[]   bytes   =   Encoding.UTF8.GetBytes(text)   ;
string   text2   =   Encoding.UTF8.GetString(bytes)   ;
```

**Component breakdown**

- `Encoding.UTF8` — UTF-8 encoding instance.
- `GetBytes` — String to byte array.
- `GetString` — Byte array to string.

#### Syntax Rules

- Choose the encoding that matches the data source or destination.
- UTF-8 is the recommended default for interoperability.
- `GetString` on invalid bytes may produce replacement characters or throw depending on the decoder fallback.

#### Constraints and Limitations

- Using the wrong encoding produces mojibake (garbled text).
- `Encoding.ASCII` replaces non-ASCII characters with `?`.
- Encoding conversions allocate byte arrays and strings.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: UTF-8 Round Trip

```csharp
using System;
using System.Text;

class Program
{
    static void Main()
    {
        string original = "Hello, 世界!";
        byte[] bytes = Encoding.UTF8.GetBytes(original);
        string restored = Encoding.UTF8.GetString(bytes);

        Console.WriteLine(bytes.Length);        // 13
        Console.WriteLine(restored);            // Hello, 世界!
        Console.WriteLine(original == restored); // True
    }
}
```

**Expected Output**

```
13
Hello, 世界!
True
```

**Why this output occurs**

- UTF-8 encodes ASCII characters as 1 byte each and CJK characters as 3 bytes each.
- The round trip preserves the text exactly.

---

#### Example 2: Encoding Comparison

```csharp
using System;
using System.Text;

class Program
{
    static void Main()
    {
        string text = "é";
        Console.WriteLine(Encoding.UTF8.GetBytes(text).Length);   // 2
        Console.WriteLine(Encoding.Unicode.GetBytes(text).Length); // 2 (UTF-16)
        Console.WriteLine(Encoding.ASCII.GetBytes(text)[0]);       // 63 ('?')
    }
}
```

**Expected Output**

```
2
2
63
```

**Why this output occurs**

- `é` is 2 bytes in UTF-8 and 2 bytes in UTF-16.
- ASCII cannot represent `é` and replaces it with `?` (byte value 63).

---

### Real-World Cases

- **File I/O** — Reading and writing text files with the correct encoding.
- **Network protocols** — HTTP headers and bodies.
- **Database storage** — Storing text in columns with defined encodings.
- **Serialization** — JSON and XML encoding declarations.

---

### References

- Encoding — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.text.encoding
- Character encoding in .NET - https://learn.microsoft.com/en-us/dotnet/standard/base-types/character-encoding-introduction
- UTF-8 strings in C# - https://learn.microsoft.com/en-us/archive/msdn-technet-blogs/utf-8-strings-in-c

---

## Deprecated, Unsafe, Implementation-Dependent, and Version-Specific Notes

| Feature | Status / Note |
|---------|---------------|
| Raw string literals (`"""..."""`) | Introduced in **C# 11**; version-specific. |
| UTF-8 string literals (`"..."u8`) | Introduced in **C# 11**; version-specific. |
| Constant interpolated strings | Introduced in **C# 10**; version-specific. |
| Newlines inside interpolation holes | Introduced in **C# 11**; version-specific. |
| `StringBuilder` vs. `string` performance | **Implementation- and workload-dependent**; benchmark before optimizing. |
| Culture-sensitive comparison results | **Platform- and version-dependent** (ICU vs. NLS on Windows). |
| `unsafe` string mutation | **Unsafe and undefined behaviour**; strongly discouraged. |
| `String.Substring` allocations | **Performance concern** in hot paths; prefer `AsSpan().Slice()` (CA1846). |
| `String.Format` vs. interpolation performance | Interpolation is generally faster on .NET 6+ but may lower to `String.Format` on older targets. |
| String interning | **Implementation detail**; do not rely on it for correctness. |

---

## References (Consolidated)

- Strings — C# Programming Guide - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/strings/
- System.String — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.string
- string keyword — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/reference-types
- String immutability — Strings (C# Programming Guide) - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/strings/#immutability-of-string-objects
- String vs. StringBuilder - https://github.com/skylinecommunications/dataminer-docs/blob/573e18c727d6935f48c1a4d57e34ab039702f997/develop/codingguidelines/Protocol/String_vs_StringBuilder.md
- String literals — Strings (C# Programming Guide) - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/strings/#string-literals
- Raw string literals — C# 11 features - https://learn.microsoft.com/en-us/dotnet/csharp/whats-new/csharp-11#raw-string-literals
- UTF-8 string literals — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/reference-types#utf-8-string-literals
- Welcome to C# 11 — Raw string literals and UTF-8 - https://devblogs.microsoft.com/dotnet/welcome-to-csharp-11/
- C# 11 Preview Updates — Raw string literals - https://devblogs.microsoft.com/dotnet/csharp-11-preview-updates/
- C# String Interpolation Explained - https://blog.ndepend.com/c-string-interpolation-explained/
- String interpolation in C# 10 and .NET 6 - https://devblogs.microsoft.com/dotnet/string-interpolation-in-c-10-and-net-6/
- $ — string interpolation — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/tokens/interpolated
- C# 10 — Constant interpolated strings - https://learn.microsoft.com/en-us/dotnet/csharp/whats-new/csharp-10#constant-interpolated-strings
- C# 11 — Newlines in string interpolations - https://learn.microsoft.com/en-us/dotnet/csharp/whats-new/csharp-11#newlines-in-string-interpolations
- String.Length — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.string.length
- String.Substring — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.string.substring
- CA1846: Prefer AsSpan over Substring - https://learn.microsoft.com/en-us/dotnet/fundamentals/code-analysis/quality-rules/ca1846
- String.IndexOf — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.string.indexof
- String.LastIndexOf — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.string.lastindexof
- String.Replace — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.string.replace
- String.ToUpper — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.string.toupper
- String.ToLower — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.string.tolower
- String.Trim — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.string.trim
- String.TrimStart — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.string.trimstart
- String.TrimEnd — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.string.trimend
- String.Split — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.string.split
- String.Join — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.string.join
- String.StartsWith — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.string.startswith
- String.EndsWith — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.string.endswith
- String.Contains — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.string.contains
- String.PadLeft — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.string.padleft
- String.PadRight — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.string.padright
- String.Format — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.string.format
- String.IsNullOrEmpty — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.string.isnullorempty
- String.IsNullOrWhiteSpace — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.string.isnullorwhitespace
- String.Concat — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.string.concat
- String.Compare — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.string.compare
- String.CompareTo — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.string.comparetoto
- String.Equals — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.string.equals
- StringBuilder — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.text.stringbuilder
- Best Practices for Comparing Strings in .NET - https://learn.microsoft.com/en-us/dotnet/standard/base-types/best-practices-strings
- StringComparison — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.stringcomparison
- How to compare strings in C# - https://learn.microsoft.com/en-us/dotnet/csharp/how-to/compare-strings
- Character encoding in .NET - https://learn.microsoft.com/en-us/dotnet/standard/base-types/character-encoding-introduction
- Rune — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.text.rune
- StringInfo — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.globalization.stringinfo
- Encoding — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.text.encoding
- Encoding.UTF8 — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.text.encoding.utf8
- Write safe and efficient C# code — Performance - https://learn.microsoft.com/en-us/dotnet/csharp/advanced-topics/performance/