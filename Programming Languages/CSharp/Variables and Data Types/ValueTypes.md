# C# Value Types — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
A value type in C# is a type whose variable directly contains its data. When a value-type variable is assigned to another variable, the value is copied, and the two variables are thereafter independent. Value types are the foundation of C#'s type system for representing simple, self-contained data such as numbers, characters, booleans, and user-defined structures.

**Technical Definition**
The C# type system divides all types into value types and reference types. Value types include **struct types** (which encompass the simple types and user-defined structs) and **enumeration types**. All value types derive implicitly from `System.ValueType`, which in turn derives from `System.Object`. Unlike reference types, it is not possible to derive a new type from a value type. Value-type variables cannot be `null` by default unless they are instances of a nullable value type. Each value type has an implicit parameterless constructor that initializes the default value of that type. The C# Language Specification classifies value types into two main categories: structs and enumerations. Structs are further divided into simple types (the built-in numeric, boolean, and character types) and user-defined structs. Assignment to a variable of a struct type creates a copy of the value being assigned; the default value of a struct is the value produced by setting all fields to their default values.

**Beginner-Friendly Explanation**
Think of a value type as a **sticky note** rather than a **business card**. When you write “42” on a sticky note and hand it to a friend, your friend gets their own sticky note with “42” written on it. If you later change your note to “43,” your friend’s note still says “42.” Value types work the same way: each variable holds its own copy of the data. This is different from reference types, where variables hold a “business card” pointing to a shared object on the heap. Value types are ideal for small, simple pieces of data that behave like numbers or characters.

---

### Key Characteristics

- **Direct data storage** — A value-type variable contains the value itself, not a reference to an object elsewhere in memory.
- **Copy semantics** — Assignment copies the value; the source and destination are independent after the assignment.
- **Implicit inheritance from `System.ValueType`** — All value types derive from `System.ValueType`, but you cannot create new types by deriving from a value type.
- **Non-nullable by default** — A value-type variable cannot be `null` unless it is a nullable value type (`T?`).
- **Default initialization** — Every value type has an implicit parameterless constructor that produces the default value (e.g., `0` for numeric types, `false` for `bool`, `'\0'` for `char`).
- **Stack or inline allocation** — Value types are typically allocated on the stack or inline within a containing type, though the specification treats stack allocation as an implementation detail.
- **Two main categories** — Structs (including simple types and user-defined structs) and enumerations.

---

### Prerequisites

- Basic understanding of C# program structure (`class`, `Main`, namespaces).
- Familiarity with the distinction between value types and reference types.
- Knowledge of primitive numeric types and their typical use cases.
- Understanding of the stack and heap as memory regions (helpful but not strictly required).
- Awareness of the `System` namespace and the `Console` class for examples.

---

### Related Programming Areas

- **Memory management** — Stack allocation, heap allocation, and garbage collection behaviour are deeply connected to value types.
- **Type system design** — Choosing between `struct` and `class` is a fundamental design decision in C#.
- **Interoperability** — Native-sized integers (`nint`/`nuint`) are important for P/Invoke and low-level libraries.
- **Performance engineering** — Avoiding boxing and choosing `struct` appropriately are key performance techniques.
- **Database and API modelling** — Nullable value types (`T?`) are essential for representing optional data.

---

### Core Concepts / Features

1. Value types (overarching concept)
2. Stack allocation
3. Built-in value types (`bool`, `byte`, `sbyte`, `char`, `decimal`, `double`, `float`, `int`, `uint`, `long`, `ulong`, `short`, `ushort`)
4. `nint` and `nuint`
5. `struct`
6. `enum`
7. Nullable value types
8. `Nullable<T>`
9. `T?`
10. Boxing
11. Unboxing
12. Boxing performance
13. Value type best practices

Each concept below follows the same uniform structure: definitions, purposes, syntax rules and structure, annotated examples with expected outputs, real-world cases, and references.

---

## 1. Value Types (Overarching Concept)

### Definitions

**Core Definition**
A value type is a type whose instances are self-contained values rather than references to objects. The variable directly stores the data, and assignment copies the value.

**Technical Definition**
The C# type system classifies all types as either value types or reference types. Value types include struct types (the simple types and user-defined structs) and enumeration types. All value types implicitly inherit from `System.ValueType`, and no type can derive from a value type. Value-type variables cannot be `null` by default; nullable value types (`T?`) add a null value to the underlying type’s value set. Assignment to a value-type variable copies the value, unlike reference-type assignment, which copies the reference.

**Beginner-Friendly Explanation**
A value type is like a **sticky note** with data written directly on it. If you give a copy of the sticky note to someone else, they get their own copy. Changes to your note do not affect theirs. Reference types are like business cards pointing to a shared object. Value types are simpler and often faster for small data.

---

### Purposes

- To represent small, self-contained data that behaves like a primitive value.
- To provide copy semantics so that assignment creates independent copies.
- To enable stack or inline allocation for predictable, low-overhead memory usage.
- To serve as the building blocks for numeric, boolean, character, and user-defined lightweight types.
- To support efficient value-based equality and hashing for small types.

---

### Syntax Rules and Structure

#### General Syntax

```
// Simple value-type variable declaration
type   identifier   [ = initializer ]   ;

// Struct type declaration
[modifiers]   struct   StructName   { ... }

// Enum type declaration
[modifiers]   enum   EnumName   { ... }

// Nullable value type
Nullable<type>   identifier   ;
type?   identifier   ;
```

**Component breakdown**

- `type` — A value type: a simple type, a struct, or an enum.
- `identifier` — The variable name.
- `= initializer` — Optional; if omitted, the variable is default-initialized (for fields) or unassigned (for locals).
- `struct` — Keyword introducing a user-defined value type.
- `enum` — Keyword introducing an enumeration type.
- `Nullable<type>` / `type?` — The nullable form of a value type.

#### Syntax Rules

- A value-type variable directly contains its data; assignment copies the value.
- Value-type variables cannot be `null` unless they are nullable value types.
- All value types implicitly derive from `System.ValueType`.
- You cannot derive a new type from a value type.
- Local value-type variables must be definitely assigned before use.
- Fields of value types are default-initialized automatically.

#### Constraints and Limitations

- Value types cannot inherit from other types (except implicitly from `System.ValueType`).
- A struct cannot declare a parameterless constructor explicitly in versions prior to C# 10 (C# 10 introduced parameterless struct constructors).
- Value types cannot be `null` by default; use `T?` for nullability.
- Boxing a value type creates a heap-allocated copy, which has performance implications.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Copy Semantics of Value Types

```csharp
using System;

struct Point
{
    public int X;
    public int Y;
}

class Program
{
    static void Main()
    {
        Point p1 = new Point { X = 1, Y = 2 };
        Point p2 = p1;                    // copy of the value

        p2.X = 100;

        Console.WriteLine($"p1: X={p1.X}, Y={p1.Y}");   // unchanged
        Console.WriteLine($"p2: X={p2.X}, Y={p2.Y}");   // modified
    }
}
```

**Expected Output**

```
p1: X=1, Y=2
p2: X=100, Y=2
```

**Why this output occurs**

- `p1` and `p2` are value-type variables. Assigning `p1` to `p2` copies the entire `Point` value.
- Modifying `p2.X` does not affect `p1.X` because the two variables hold independent copies.
- This demonstrates value semantics: each variable owns its data.

---

#### Example 2: Default Values of Value Types

```csharp
using System;

class Program
{
    static int staticInt;          // default 0
    static bool staticBool;        // default false
    static char staticChar;        // default '\0'
    static Point staticPoint;      // default: all fields 0

    static void Main()
    {
        Console.WriteLine(staticInt);        // 0
        Console.WriteLine(staticBool);       // False
        Console.WriteLine((int)staticChar);  // 0
        Console.WriteLine(staticPoint.X);    // 0
        Console.WriteLine(staticPoint.Y);    // 0
    }
}

struct Point
{
    public int X;
    public int Y;
}
```

**Expected Output**

```
0
False
0
0
0
```

**Why this output occurs**

- Static fields of value types are default-initialized.
- The default for numeric types is `0`, for `bool` it is `false`, and for `char` it is `'\0'` (code `0`).
- A struct’s default value is produced by setting all its fields to their default values.

---

### Real-World Cases

- **Mathematical and geometric types** — `Point`, `Vector`, `Complex`, `Color` are naturally value types because they represent self-contained values.
- **Financial and monetary calculations** — `decimal` is a value type designed for precise financial arithmetic.
- **Identifiers and codes** — `Guid`, `DateTime`, and enum types are value types used as identifiers and timestamps.
- **Performance-sensitive data** — Small structs avoid heap allocations in hot paths.

---

### References

- Value types — C# Reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/value-types
- Types — C# Language Specification - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/language-specification/types
- ValueType Class (System) - https://learn.microsoft.com/en-us/dotnet/api/system.valuetype

---

## 2. Stack Allocation

### Definitions

**Core Definition**
Stack allocation refers to the placement of value-type data on the call stack, a region of memory that grows and shrinks in a last-in-first-out (LIFO) manner as methods are called and return.

**Technical Definition**
The C# Language Specification does not mandate that value types always be stack-allocated; stack allocation is an implementation detail. However, in current Microsoft .NET implementations, local value-type variables and method parameters are typically allocated on the thread stack, while value-type fields are allocated inline within their containing type (which may itself be on the stack or the heap). Reference-type objects are always allocated on the managed heap. Stack allocation is deterministic: memory is reclaimed automatically when the method returns or the scope exits, without garbage collection.

**Beginner-Friendly Explanation**
Imagine a stack of plates. When you call a method, you put a new plate on top for that method’s local variables. When the method finishes, you take the plate off. Value-type variables often live on these plates. This is fast because there is no garbage collector involved. Reference types, by contrast, live in a large shared “warehouse” (the heap) that needs periodic cleanup.

---

### Purposes

- To provide fast, deterministic allocation and deallocation for short-lived data.
- To avoid garbage-collection pressure for small, local values.
- To enable efficient parameter passing for small value types.
- To support the LIFO memory model that matches method call structure.
- To reduce heap fragmentation for frequently created small values.

---

### Syntax Rules and Structure

Stack allocation is not expressed through explicit syntax in safe C#; it is a consequence of declaring value-type locals and parameters:

```
void Method()
{
    int local = 42;          // typically stack-allocated
    Point p = new Point();   // typically stack-allocated
}

struct Point { public int X, Y; }
```

**Component breakdown**

- Local variable declaration — A value-type local is typically placed on the stack.
- Method parameter — Value-type parameters are typically passed on the stack.
- Struct field — If a struct is a field of a class, the struct’s data is inline within the heap-allocated object; if the struct is a local, its fields are on the stack.

#### Syntax Rules

- Stack allocation is automatic for value-type locals and parameters in typical .NET implementations.
- The specification guarantees lifetime and assignment semantics, not physical storage location.
- Captured variables (in lambdas) may be moved to the heap even if they are value types.
- `ref struct` types (e.g., `Span<T>`) are guaranteed to be stack-only and cannot be boxed or stored on the heap.

#### Constraints and Limitations

- Stack space is limited (typically 1 MB per thread by default); large value types can cause stack overflow.
- The stack is not shared across threads; each thread has its own stack.
- Stack allocation is an implementation detail; code should not depend on it for correctness.
- Value types captured by closures or stored in reference-type fields are not stack-allocated in the same way.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Local Value Types and the Stack

```csharp
using System;

struct Point
{
    public int X;
    public int Y;
}

class Program
{
    static void CreatePoint()
    {
        Point p = new Point { X = 1, Y = 2 };   // typically stack-allocated
        Console.WriteLine($"{p.X}, {p.Y}");
    }   // stack frame for CreatePoint is popped here

    static void Main()
    {
        CreatePoint();
        CreatePoint();
    }
}
```

**Expected Output**

```
1, 2
1, 2
```

**Why this output occurs**

- Each call to `CreatePoint` creates a fresh stack frame containing `p`.
- When the method returns, the stack frame is discarded, and the memory is reclaimed without garbage collection.
- The two calls produce independent copies of `Point`.

---

#### Example 2: `ref struct` and Stack-Only Guarantees

```csharp
using System;

ref struct StackOnly
{
    public int Value;
}

class Program
{
    static void Main()
    {
        StackOnly s = new StackOnly { Value = 42 };
        Console.WriteLine(s.Value);

        // object o = s;          // Compile error: cannot box a ref struct
        // StackOnly[] arr = new StackOnly[10]; // Compile error: cannot use ref struct as array element
    }
}
```

**Expected Output**

```
42
```

**Why this output occurs**

- `ref struct` types are guaranteed to be stack-only and cannot be boxed or stored in arrays.
- This ensures they never escape to the heap, preserving their stack-only lifetime.

---

### Real-World Cases

- **High-performance parsers** — `Span<T>` and `ReadOnlySpan<T>` are `ref struct` types that operate over stack-allocated or existing memory without allocation.
- **Game loops** — Small structs for positions, vectors, and colours are stack-allocated and avoid GC pressure.
- **Numerical algorithms** — Local accumulators and temporary values are typically stack-allocated for speed.

---

### References

- The Stack Is An Implementation Detail, Part One - https://learn.microsoft.com/en-us/archive/blogs/ericlippert/the-stack-is-an-implementation-detail-part-one
- The Truth About Value Types - https://learn.microsoft.com/en-us/archive/blogs/ericlippert/the-truth-about-value-types
- ref struct — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/ref-struct

---

## 3. Built-in Value Types

### Definitions

**Core Definition**
Built-in value types are the predefined simple types provided by the C# language, including the numeric types, `bool`, and `char`. These are implemented as structs in the `System` namespace and have keyword aliases in C#.

**Technical Definition**
The simple types are a set of predefined struct types provided by C#. They include integral types (signed and unsigned integer types plus `char`), floating-point types (`float`, `double`), `decimal`, and `bool`. Each simple type is an alias for a predefined struct type in the `System` namespace — for example, `int` is an alias for `System.Int32`. Simple types differ from other structs in that they permit literals and can be used to declare constants.

**Beginner-Friendly Explanation**
These are the basic building blocks: whole numbers (`int`, `long`, etc.), decimal numbers (`double`, `decimal`), true/false values (`bool`), and single characters (`char`). You use them every day in C# programs.

---

### Purposes

- To represent numeric data of various sizes and precisions.
- To represent truth values (`bool`) and individual characters (`char`).
- To provide efficient, hardware-friendly arithmetic and comparison operations.
- To serve as the fundamental data types for variables, parameters, and fields.
- To enable compile-time constant expressions and literals.

---

### Syntax Rules and Structure (General for Built-in Types)

```
type   identifier   [ = literal ]   ;
```

**Component breakdown**

- `type` — One of `bool`, `byte`, `sbyte`, `char`, `decimal`, `double`, `float`, `int`, `uint`, `long`, `ulong`, `short`, `ushort`.
- `identifier` — The variable name.
- `= literal` — Optional literal initializer (e.g., `42`, `3.14`, `true`, `'A'`).

#### Syntax Rules

- Built-in type keywords are aliases for `System` struct types.
- Literals are available for all simple types except `decimal` (which uses the `m` suffix) and `float` (which uses the `f` suffix).
- `char` literals use single quotes; `string` (a reference type) uses double quotes.
- Constants of simple types can be declared with `const`.

#### Constraints and Limitations

- Each type has a fixed range and precision; overflow and underflow are possible.
- `decimal` has a smaller range than `double` but higher precision for base-10 fractions.
- `float` and `double` use binary floating-point, which can introduce rounding errors.
- `char` represents a UTF-16 code unit, not a full Unicode code point.

---

### 3.1 `bool`

**Definitions**

- **Core Definition:** `bool` represents a Boolean value: `true` or `false`.
- **Technical Definition:** `bool` is an alias for `System.Boolean`. It occupies 1 byte and is a value type. It supports logical operators `&&`, `||`, `!`, and conditional operators.
- **Beginner-Friendly:** `bool` is a yes/no switch: true or false.

**Purposes**

- To represent truth values in conditions and flags.
- To store the result of comparisons and logical expressions.
- To model binary states (on/off, enabled/disabled, valid/invalid).
- To control program flow through `if`, `while`, and other conditional statements.

**Syntax Rules and Structure**

```
bool   identifier   [ = true | false ]   ;
```

**Constraints:** `bool` cannot be implicitly converted to numeric types; explicit conversions are not allowed (use `Convert.ToInt32` if needed).

**Annotated Example**

```csharp
using System;

class Program
{
    static void Main()
    {
        bool isReady = true;
        bool hasError = false;

        if (isReady && !hasError)
        {
            Console.WriteLine("Ready to proceed");
        }
    }
}
```

**Expected Output**

```
Ready to proceed
```

**Real-World Cases:** Feature flags, validation results, state machines.

**References**

- bool — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/bool

---

### 3.2 `byte`

**Definitions**

- **Core Definition:** `byte` represents an unsigned 8-bit integer with values from 0 to 255.
- **Technical Definition:** `byte` is an alias for `System.Byte`. It is an unsigned integral type occupying 1 byte.
- **Beginner-Friendly:** `byte` is a small positive whole number from 0 to 255.

**Purposes**

- To store small non-negative values efficiently.
- To represent raw binary data (bytes).
- To work with streams, file I/O, and network protocols.
- To save memory when the full `int` range is unnecessary.

**Syntax Rules and Structure**

```
byte   identifier   [ = 0..255 literal ]   ;
```

**Constraints:** Cannot hold negative values; overflow wraps in unchecked contexts.

**Annotated Example**

```csharp
using System;

class Program
{
    static void Main()
    {
        byte b = 200;
        Console.WriteLine(b);          // 200

        // b = 300;                    // Compile-time error: constant out of range
        b = unchecked((byte)300);      // wraps to 44
        Console.WriteLine(b);          // 44
    }
}
```

**Expected Output**

```
200
44
```

**Real-World Cases:** RGB colour components, file byte streams, small counters.

**References**

- byte — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/integral-numeric-types

---

### 3.3 `sbyte`

**Definitions**

- **Core Definition:** `sbyte` represents a signed 8-bit integer with values from −128 to 127.
- **Technical Definition:** `sbyte` is an alias for `System.SByte`. It is a signed integral type occupying 1 byte.
- **Beginner-Friendly:** `sbyte` is a small whole number that can be negative or positive, from −128 to 127.

**Purposes**

- To store small signed values efficiently.
- To interoperate with native APIs that use signed 8-bit integers.
- To represent offsets or deltas in constrained ranges.

**Syntax Rules and Structure**

```
sbyte   identifier   [ = -128..127 literal ]   ;
```

**Constraints:** Range is limited; overflow wraps in unchecked contexts.

**Annotated Example**

```csharp
using System;

class Program
{
    static void Main()
    {
        sbyte temperature = -40;
        Console.WriteLine(temperature);   // -40
    }
}
```

**Expected Output**

```
-40
```

**Real-World Cases:** Sensor readings, small signed offsets, interop with C `int8_t`.

**References**

- sbyte — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/integral-numeric-types

---

### 3.4 `char`

**Definitions**

- **Core Definition:** `char` represents a single UTF-16 code unit.
- **Technical Definition:** `char` is an alias for `System.Char`. It is an integral type occupying 2 bytes, with values from `U+0000` to `U+FFFF`. It is not implicitly convertible to or from other integral types without explicit casts.
- **Beginner-Friendly:** `char` is a single character, like `'A'` or `'7'`, written in single quotes.

**Purposes**

- To represent individual characters in text.
- To process strings character by character.
- To work with character encodings and Unicode code units.
- To perform character classification and conversion.

**Syntax Rules and Structure**

```
char   identifier   [ = 'character literal' ]   ;
```

**Constraints:** Represents a UTF-16 code unit, not a full Unicode scalar value; surrogate pairs represent characters outside the Basic Multilingual Plane.

**Annotated Example**

```csharp
using System;

class Program
{
    static void Main()
    {
        char letter = 'A';
        char digit = '7';
        char newline = '\n';

        Console.WriteLine(letter);          // A
        Console.WriteLine((int)letter);     // 65
        Console.WriteLine(char.IsDigit(digit)); // True
    }
}
```

**Expected Output**

```
A
65
True
```

**Real-World Cases:** Parsers, tokenizers, text formatting, encoding conversions.

**References**

- char — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/char

---

### 3.5 `decimal`

**Definitions**

- **Core Definition:** `decimal` represents a decimal floating-point number with high precision, suitable for financial and monetary calculations.
- **Technical Definition:** `decimal` is an alias for `System.Decimal`. It is a 128-bit value type with a precision of 28–29 significant digits and a range of approximately ±7.9 × 10²⁸. It uses base-10 representation, avoiding binary floating-point rounding errors for decimal fractions.
- **Beginner-Friendly:** `decimal` is a highly precise number for money and exact decimal calculations.

**Purposes**

- To perform financial and monetary calculations without binary rounding errors.
- To represent exact decimal fractions (e.g., 0.1, 0.01).
- To meet regulatory and accounting precision requirements.
- To store values requiring more precision than `double` can reliably provide for base-10 fractions.

**Syntax Rules and Structure**

```
decimal   identifier   [ = literal m ]   ;
```

**Constraints:** Smaller range than `double`; slower arithmetic than `double`; literals require the `m` or `M` suffix.

**Annotated Example**

```csharp
using System;

class Program
{
    static void Main()
    {
        decimal price = 19.99m;
        decimal tax = 0.07m;
        decimal total = price * (1 + tax);

        Console.WriteLine(total);          // 21.3893
        Console.WriteLine(total == 21.3893m); // True
    }
}
```

**Expected Output**

```
21.3893
True
```

**Real-World Cases:** Banking, invoicing, tax calculations, currency conversions.

**References**

- decimal — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/floating-point-numeric-types#the-decimal-type

---

### 3.6 `double`

**Definitions**

- **Core Definition:** `double` represents a double-precision binary floating-point number.
- **Technical Definition:** `double` is an alias for `System.Double`. It is a 64-bit IEEE 754 floating-point type with approximately 15–17 significant decimal digits of precision.
- **Beginner-Friendly:** `double` is a fast decimal number with good precision, widely used in science and graphics.

**Purposes**

- To perform general-purpose floating-point arithmetic.
- To represent scientific measurements and physical quantities.
- To support graphics, simulations, and numerical computations.
- To provide a balance between precision and performance.

**Syntax Rules and Structure**

```
double   identifier   [ = literal d ]   ;
```

**Constraints:** Binary floating-point cannot represent all decimal fractions exactly; use `decimal` for exact base-10 fractions.

**Annotated Example**

```csharp
using System;

class Program
{
    static void Main()
    {
        double radius = 2.5;
        double area = Math.PI * radius * radius;
        Console.WriteLine(area);            // 19.634954084936208
    }
}
```

**Expected Output**

```
19.634954084936208
```

**Real-World Cases:** Physics simulations, graphics, statistical computations, scientific modelling.

**References**

- double — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/floating-point-numeric-types

---

### 3.7 `float`

**Definitions**

- **Core Definition:** `float` represents a single-precision binary floating-point number.
- **Technical Definition:** `float` is an alias for `System.Single`. It is a 32-bit IEEE 754 floating-point type with approximately 7 significant decimal digits of precision.
- **Beginner-Friendly:** `float` is a smaller, faster floating-point number with less precision than `double`.

**Purposes**

- To save memory in large arrays of floating-point values.
- To interoperate with graphics APIs that use 32-bit floats.
- To perform computations where single precision is sufficient.
- To reduce bandwidth in serialization and networking.

**Syntax Rules and Structure**

```
float   identifier   [ = literal f ]   ;
```

**Constraints:** Limited precision; rounding errors accumulate quickly; literals require the `f` or `F` suffix.

**Annotated Example**

```csharp
using System;

class Program
{
    static void Main()
    {
        float x = 1.0f / 3.0f;
        Console.WriteLine(x);               // 0.33333334
        Console.WriteLine(x == 1.0 / 3.0); // False (float vs double)
    }
}
```

**Expected Output**

```
0.33333334
False
```

**Real-World Cases:** Game engines, 3D graphics, embedded systems, signal processing.

**References**

- float — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/floating-point-numeric-types

---

### 3.8 `int`

**Definitions**

- **Core Definition:** `int` represents a signed 32-bit integer with values from −2,147,483,648 to 2,147,483,647.
- **Technical Definition:** `int` is an alias for `System.Int32`. It is the most commonly used integral type in C#.
- **Beginner-Friendly:** `int` is the default whole-number type, from about −2.1 billion to +2.1 billion.

**Purposes**

- To store counts, indices, identifiers, and general-purpose whole numbers.
- To serve as the default integer type for arithmetic and logic.
- To interoperate with most .NET APIs that use 32-bit integers.
- To provide a balance between range and memory usage.

**Syntax Rules and Structure**

```
int   identifier   [ = literal ]   ;
```

**Constraints:** Overflow wraps in unchecked contexts; use `long` for larger ranges.

**Annotated Example**

```csharp
using System;

class Program
{
    static void Main()
    {
        int count = 42;
        int next = count + 1;
        Console.WriteLine(next);            // 43
        Console.WriteLine(int.MaxValue);    // 2147483647
    }
}
```

**Expected Output**

```
43
2147483647
```

**Real-World Cases:** Loop counters, array indices, database IDs, general arithmetic.

**References**

- int — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/integral-numeric-types

---

### 3.9 `uint`

**Definitions**

- **Core Definition:** `uint` represents an unsigned 32-bit integer with values from 0 to 4,294,967,295.
- **Technical Definition:** `uint` is an alias for `System.UInt32`.
- **Beginner-Friendly:** `uint` is a non-negative whole number up to about 4.29 billion.

**Purposes**

- To store values that are inherently non-negative (e.g., counts, sizes).
- To interoperate with native APIs that use unsigned 32-bit integers.
- To double the positive range compared to `int`.
- To represent bit masks and flags.

**Syntax Rules and Structure**

```
uint   identifier   [ = literal u ]   ;
```

**Constraints:** Cannot represent negative values; arithmetic underflow wraps.

**Annotated Example**

```csharp
using System;

class Program
{
    static void Main()
    {
        uint population = 8_000_000_000;   // 8 billion
        Console.WriteLine(population);      // 8000000000
    }
}
```

**Expected Output**

```
8000000000
```

**Real-World Cases:** Population counts, file sizes, interop with unsigned native types.

**References**

- uint — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/integral-numeric-types

---

### 3.10 `long`

**Definitions**

- **Core Definition:** `long` represents a signed 64-bit integer with a range of approximately ±9.22 × 10¹⁸.
- **Technical Definition:** `long` is an alias for `System.Int64`.
- **Beginner-Friendly:** `long` is a very large whole number, for values beyond `int`’s range.

**Purposes**

- To store large counts, timestamps, and identifiers.
- To perform arithmetic that exceeds the `int` range.
- To interoperate with APIs that use 64-bit integers.
- To represent file sizes and memory offsets.

**Syntax Rules and Structure**

```
long   identifier   [ = literal L ]   ;
```

**Constraints:** Larger memory footprint (8 bytes); literals should use the `L` suffix when exceeding `int` range.

**Annotated Example**

```csharp
using System;

class Program
{
    static void Main()
    {
        long ticks = DateTime.UtcNow.Ticks;
        Console.WriteLine(ticks > int.MaxValue);   // True
    }
}
```

**Expected Output**

```
True
```

**Real-World Cases:** Timestamps, large database keys, file sizes, financial amounts.

**References**

- long — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/integral-numeric-types

---

### 3.11 `ulong`

**Definitions**

- **Core Definition:** `ulong` represents an unsigned 64-bit integer with a range of 0 to approximately 1.84 × 10¹⁹.
- **Technical Definition:** `ulong` is an alias for `System.UInt64`.
- **Beginner-Friendly:** `ulong` is a very large non-negative whole number.

**Purposes**

- To store extremely large non-negative values.
- To interoperate with native APIs that use unsigned 64-bit integers.
- To represent hash codes and bit patterns.
- To handle sizes and counts beyond `long`’s positive range.

**Syntax Rules and Structure**

```
ulong   identifier   [ = literal UL ]   ;
```

**Constraints:** Cannot represent negative values; literals require `UL` suffix when exceeding `long` range.

**Annotated Example**

```csharp
using System;

class Program
{
    static void Main()
    {
        ulong max = ulong.MaxValue;
        Console.WriteLine(max);   // 18446744073709551615
    }
}
```

**Expected Output**

```
18446744073709551615
```

**Real-World Cases:** Cryptographic hashes, large bitmasks, interop with unsigned 64-bit native types.

**References**

- ulong — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/integral-numeric-types

---

### 3.12 `short`

**Definitions**

- **Core Definition:** `short` represents a signed 16-bit integer with values from −32,768 to 32,767.
- **Technical Definition:** `short` is an alias for `System.Int16`.
- **Beginner-Friendly:** `short` is a small whole number from about −32 thousand to +32 thousand.

**Purposes**

- To save memory in arrays of small signed values.
- To interoperate with native APIs that use 16-bit integers.
- To represent audio samples and sensor readings.
- To reduce memory footprint when the `int` range is unnecessary.

**Syntax Rules and Structure**

```
short   identifier   [ = literal ]   ;
```

**Constraints:** Limited range; arithmetic may promote to `int`.

**Annotated Example**

```csharp
using System;

class Program
{
    static void Main()
    {
        short temperature = -10;
        Console.WriteLine(temperature);   // -10
    }
}
```

**Expected Output**

```
-10
```

**Real-World Cases:** Audio PCM samples, embedded sensor data, legacy interop.

**References**

- short — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/integral-numeric-types

---

### 3.13 `ushort`

**Definitions**

- **Core Definition:** `ushort` represents an unsigned 16-bit integer with values from 0 to 65,535.
- **Technical Definition:** `ushort` is an alias for `System.UInt16`.
- **Beginner-Friendly:** `ushort` is a small non-negative whole number from 0 to 65,535.

**Purposes**

- To store small non-negative values efficiently.
- To interoperate with native APIs that use unsigned 16-bit integers.
- To represent colour channels, ports, and small counts.
- To work with UTF-16 code units in some contexts.

**Syntax Rules and Structure**

```
ushort   identifier   [ = literal ]   ;
```

**Constraints:** Cannot represent negative values; limited to 65,535.

**Annotated Example**

```csharp
using System;

class Program
{
    static void Main()
    {
        ushort port = 8080;
        Console.WriteLine(port);          // 8080
    }
}
```

**Expected Output**

```
8080
```

**Real-World Cases:** Network ports, colour components, small unsigned counters.

**References**

- ushort — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/integral-numeric-types

---

## 4. `nint` and `nuint` (Native-Sized Integers)

### Definitions

**Core Definition**
`nint` and `nuint` are native-sized signed and unsigned integer types whose size depends on the platform: 32 bits on a 32-bit process and 64 bits on a 64-bit process.

**Technical Definition**
`nint` and `nuint` are contextual keywords introduced in C# 9.0. They are represented by the underlying types `System.IntPtr` and `System.UIntPtr`, with the compiler surfacing additional conversions and operations for those types as native integers. They are intended for interop scenarios and low-level libraries where the integer size must match the platform’s pointer size. The size of `nint` and `nuint` is 32-bit in a 32-bit process and 64-bit in a 64-bit process. `sizeof(nint)` and `sizeof(nuint)` require an unsafe context and are not compile-time constants.

**Beginner-Friendly Explanation**
`nint` and `nuint` are “platform-sized” integers. They automatically match the natural size of the computer’s architecture — 32 bits on older systems, 64 bits on modern ones. They are useful when working with memory addresses or native code.

---

### Purposes

- To interoperate with native code that uses pointer-sized integers.
- To write low-level libraries that work correctly on both 32-bit and 64-bit platforms.
- To perform arithmetic on memory offsets and sizes without platform-specific code.
- To avoid explicit casts between `int` and `IntPtr` in interop scenarios.
- To provide a portable native integer abstraction.

---

### Syntax Rules and Structure

```
nint   identifier   [ = (nint)literal ]   ;
nuint   identifier   [ = (nuint)literal ]   ;
```

**Component breakdown**

- `nint` — Native-sized signed integer.
- `nuint` — Native-sized unsigned integer.
- Cast — Required for constant initialization; there is no direct native-int literal syntax.

#### Syntax Rules

- `nint` and `nuint` are contextual keywords, not reserved words.
- Constant expressions of type `nint`/`nuint` are supported with casts.
- Identity conversions exist between `nint` and `IntPtr`, and between `nuint` and `UIntPtr`.
- `sizeof(nint)` and `sizeof(nuint)` require an unsafe context and are not compile-time constants.

#### Constraints and Limitations

- `nint` and `nuint` are not CLS-compliant; avoid them in public APIs intended for cross-language consumption.
- Constant folding for `nint`/`nuint` is performed using 32-bit operands, which can affect compile-time constant behaviour.
- They are version-specific: introduced in C# 9.0.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Platform-Dependent Size

```csharp
using System;

class Program
{
    static void Main()
    {
        nint signedNative = 42;
        nuint unsignedNative = 42;

        Console.WriteLine($"nint size: {sizeof(nint) * 8} bits");   // requires unsafe
        Console.WriteLine($"nuint size: {sizeof(nuint) * 8} bits"); // requires unsafe
        Console.WriteLine(signedNative);        // 42
        Console.WriteLine(unsignedNative);      // 42
    }
}
```

(To compile, enable unsafe code or omit the `sizeof` lines.)

**Expected Output (on a 64-bit process)**

```
nint size: 64 bits
nuint size: 64 bits
42
42
```

**Why this output occurs**

- `nint` and `nuint` are 64 bits in a 64-bit process.
- The values are simple integers; the native size affects storage and arithmetic, not the value itself.

---

#### Example 2: Interop with `IntPtr`

```csharp
using System;

class Program
{
    static void Main()
    {
        nint native = 123;
        IntPtr ptr = native;          // identity conversion
        nint back = ptr;             // identity conversion

        Console.WriteLine(back);      // 123
    }
}
```

**Expected Output**

```
123
```

**Why this output occurs**

- `nint` and `IntPtr` have an identity conversion, so no cast is required.

---

### Real-World Cases

- **P/Invoke and COM interop** — Passing pointer-sized integers to native APIs.
- **Memory management libraries** — Computing offsets and sizes that must match the platform’s pointer size.
- **Low-level data structures** — Implementing custom collections and buffers that work on both 32-bit and 64-bit systems.

---

### References

- Native-sized integers — C# feature specifications - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/proposals/csharp-9.0/native-integers
- nint and nuint types — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/nint-nuint

---

## 5. `struct`

### Definitions

**Core Definition**
A `struct` is a user-defined value type that can encapsulate data and related functionality.

**Technical Definition**
A structure type (or struct type) is a value type that can encapsulate data and related functionality. You use the `struct` keyword to define a structure type. Structs are value types and have value semantics: assignment copies the value, and each variable has its own copy of the data (except when passed by reference). All struct types implicitly inherit from `System.ValueType`. Structs can implement interfaces but cannot inherit from other types or be inherited from. Structs cannot declare a finalizer and cannot have a parameterless constructor before C# 10 (C# 10 introduced parameterless struct constructors).

**Beginner-Friendly Explanation**
A `struct` is a custom-made value type, like a small package of related data. For example, a `Point` struct might hold `X` and `Y` coordinates. When you copy a struct variable, you get an independent copy of the whole package.

---

### Purposes

- To create small, lightweight types that behave like primitive values.
- To represent a single logical value composed of multiple fields.
- To avoid heap allocations for frequently created small objects.
- To provide value semantics (copy on assignment) for data-centric types.
- To enable inline storage within arrays and other containing types.

---

### Syntax Rules and Structure

```
[accessModifiers]   [readonly]   struct   StructName   [: interfaces]
{
    // fields, properties, methods, constructors
}
```

**Component breakdown**

- `struct` — Keyword introducing a value type.
- `StructName` — The type name (PascalCase by convention).
- `: interfaces` — Optional interface implementations.
- `readonly` — Optional modifier making all instance fields readonly (C# 7.2+).
- Body — May contain fields, properties, methods, constructors, and nested types.

#### Syntax Rules

- Structs are value types; assignment copies the entire value.
- Structs cannot inherit from a class or another struct; they implicitly derive from `System.ValueType`.
- Structs can implement interfaces.
- Structs cannot declare a finalizer.
- Structs cannot have a parameterless constructor before C# 10 (C# 10 allows explicit parameterless constructors).
- All fields of a struct must be definitely assigned before the struct is used.

#### Constraints and Limitations

- Structs are copied on assignment and when passed by value; large structs can be expensive to copy.
- Mutable structs can cause subtle bugs because modifications to a copy do not affect the original.
- Structs should generally be small (guideline: under 16 bytes), immutable, and infrequently boxed.
- Structs cannot be `null` unless nullable (`T?`).

---

### Annotated Code Examples and Expected Outputs

#### Example 1: A Simple `Point` Struct

```csharp
using System;

public struct Point
{
    public int X;
    public int Y;

    public Point(int x, int y)
    {
        X = x;
        Y = y;
    }

    public override string ToString() => $"({X}, {Y})";
}

class Program
{
    static void Main()
    {
        Point p1 = new Point(1, 2);
        Point p2 = p1;              // copy

        p2.X = 100;

        Console.WriteLine(p1);      // (1, 2)
        Console.WriteLine(p2);      // (100, 2)
    }
}
```

**Expected Output**

```
(1, 2)
(100, 2)
```

**Why this output occurs**

- `Point` is a struct, so `p2 = p1` copies the value.
- Modifying `p2.X` does not affect `p1.X`.

---

#### Example 2: `readonly struct`

```csharp
using System;

public readonly struct Vector
{
    public readonly double X;
    public readonly double Y;

    public Vector(double x, double y)
    {
        X = x;
        Y = y;
    }

    public double Length => Math.Sqrt(X * X + Y * Y);
}

class Program
{
    static void Main()
    {
        var v = new Vector(3, 4);
        Console.WriteLine(v.Length);   // 5
    }
}
```

**Expected Output**

```
5
```

**Why this output occurs**

- `readonly struct` makes all instance fields readonly, preventing mutation after construction.
- The `Length` property computes the magnitude of the vector.

---

### Real-World Cases

- **Geometric primitives** — `Point`, `Rectangle`, `Size`, `Color`.
- **Financial values** — `Money` structs combining an amount and currency.
- **Identifiers** — `Guid`, `DateTime`, `TimeSpan` (though these are framework-provided structs).
- **Performance-sensitive data** — Small structs in arrays and buffers to avoid heap allocations.

---

### References

- Structure types — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/struct
- Structs — C# Language Specification - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/language-specification/structs
- Choosing Between Classes and Structures - https://learn.microsoft.com/en-us/dotnet/standard/design-guidelines/choosing-between-class-and-struct

---

## 6. `enum`

### Definitions

**Core Definition**
An `enum` is a distinct value type that consists of a set of named constants called the enumerator list.

**Technical Definition**
An enumeration type (or enum type) is a value type defined by a set of named constants of the underlying integral type. By default, the underlying type is `int`, and the values start at `0` and increase by `1` for each member. The approved underlying types are `byte`, `sbyte`, `short`, `ushort`, `int`, `uint`, `long`, and `ulong` (but not `char`). Enums provide efficient, strongly typed alternatives to numeric constants.

**Beginner-Friendly Explanation**
An `enum` is a list of named options. For example, `DayOfWeek.Monday`, `DayOfWeek.Tuesday`, etc. Instead of remembering that Monday is `1`, you write `DayOfWeek.Monday`, which is clearer and safer.

---

### Purposes

- To define a set of named constants representing a fixed set of options.
- To strongly type parameters, properties, and return types that should accept only specific values.
- To improve code readability by replacing magic numbers with descriptive names.
- To enable compile-time checking of valid values.
- To provide efficient storage (enums are value types backed by integral types).

---

### Syntax Rules and Structure

```
[accessModifiers]   enum   EnumName   [: underlyingType]
{
    Member1   [ = value1 ],
    Member2   [ = value2 ],
    ...
}
```

**Component breakdown**

- `enum` — Keyword introducing an enumeration type.
- `EnumName` — The type name.
- `: underlyingType` — Optional; defaults to `int`. Must be one of the approved integral types.
- `Member` — Named constant.
- `= value` — Optional explicit value; subsequent members increment by `1`.

#### Syntax Rules

- Enum members must be named uniquely within the enum.
- Values must be within the range of the underlying type.
- Members can be assigned explicit values; implicit values follow the previous member’s value plus one.
- Enums can be used as `const` values.
- Enums cannot contain methods, properties, or events (though extension methods can be written for them).

#### Constraints and Limitations

- Enums cannot inherit from other types.
- Enums are not inherently validated at runtime; casting an arbitrary integer to an enum type can produce an undefined value.
- Enums are value types and are copied on assignment.
- The underlying type cannot be `char`.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Basic Enum Usage

```csharp
using System;

public enum Season
{
    Spring,
    Summer,
    Autumn,
    Winter
}

class Program
{
    static void Main()
    {
        Season current = Season.Summer;
        Console.WriteLine(current);            // Summer
        Console.WriteLine((int)current);       // 1

        foreach (Season s in Enum.GetValues(typeof(Season)))
        {
            Console.WriteLine($"{s} = {(int)s}");
        }
    }
}
```

**Expected Output**

```
Summer
1
Spring = 0
Summer = 1
Autumn = 2
Winter = 3
```

**Why this output occurs**

- `Spring` is `0`, `Summer` is `1`, etc.
- `Enum.GetValues` returns all members in order.
- The explicit cast `(int)` reveals the underlying integer value.

---

#### Example 2: Enum with Explicit Values and Underlying Type

```csharp
using System;

[Flags]
public enum Permissions : byte
{
    None = 0,
    Read = 1,
    Write = 2,
    Execute = 4,
    All = Read | Write | Execute
}

class Program
{
    static void Main()
    {
        Permissions p = Permissions.Read | Permissions.Write;
        Console.WriteLine(p);                  // Read, Write
        Console.WriteLine((byte)p);            // 3
        Console.WriteLine(p.HasFlag(Permissions.Read)); // True
    }
}
```

**Expected Output**

```
Read, Write
3
True
```

**Why this output occurs**

- `Read` and `Write` have distinct bit values (`1` and `2`); combining them with `|` yields `3`.
- `[Flags]` changes the `ToString` representation to a comma-separated list.
- `HasFlag` checks whether the bit is set.

---

### Real-World Cases

- **Days of the week, months, seasons** — Fixed sets of named values.
- **HTTP status codes** — `Ok`, `NotFound`, `InternalServerError`.
- **Log levels** — `Debug`, `Info`, `Warning`, `Error`, `Critical`.
- **Flags and permissions** — Bitwise combinations of capabilities.

---

### References

- Enumeration types — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/enum
- enum keyword — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/enum
- Enums — C# Language Specification - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/language-specification/enums

---

## 7. Nullable Value Types

### Definitions

**Core Definition**
A nullable value type `T?` represents all values of its underlying value type `T` plus an additional `null` value.

**Technical Definition**
A nullable value type is written `T?`, where `T` is the underlying type. This syntax is shorthand for `System.Nullable<T>`, and the two forms are interchangeable. The underlying type of a nullable value type must be a non-nullable value type; `int??` is invalid. A nullable value type has two public read-only properties: `HasValue` (bool) and `Value` (T). Reading `Value` when `HasValue` is `false` throws `InvalidOperationException`. Nullable value types are instances of the generic struct `System.Nullable<T>`.

**Beginner-Friendly Explanation**
A nullable value type is a value type that can also be `null` — meaning “no value” or “undefined.” For example, a `bool?` can be `true`, `false`, or `null`. This is useful when a value might be missing, such as an optional field in a database.

---

### Purposes

- To represent the absence of a value for a value type.
- To model optional fields in databases, forms, and APIs.
- To distinguish between `0`/`false` and “not set.”
- To support database null semantics in ORM and data-access code.
- To enable null-conditional and null-coalescing operators with value types.

---

### Syntax Rules and Structure

```
Nullable<T>   identifier   [ = value | null ]   ;
T?   identifier   [ = value | null ]   ;
```

**Component breakdown**

- `T?` / `Nullable<T>` — The nullable form of value type `T`.
- `identifier` — The variable name.
- `= value` — Implicit conversion from `T` to `T?`.
- `= null` — Assigns the null (no-value) state.

#### Syntax Rules

- A non-nullable value type `T` is implicitly convertible to `T?`.
- `null` is implicitly convertible to `T?`.
- `HasValue` indicates whether a value is present.
- `Value` returns the underlying value or throws if `HasValue` is `false`.
- `GetValueOrDefault()` returns the value or the default for `T` if null.
- Nullable value types do not implement interfaces that the underlying type implements (with some exceptions through boxing).

#### Constraints and Limitations

- `T` cannot itself be a nullable value type (`int??` is invalid).
- `T` cannot be a reference type.
- Reading `Value` on a null instance throws `InvalidOperationException`.
- Nullable value types are structs and therefore value types themselves; they are copied on assignment.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: `Nullable<T>` and `T?` Equivalence

```csharp
using System;

class Program
{
    static void Main()
    {
        Nullable<int> a = 5;
        int? b = 10;
        int? c = null;

        Console.WriteLine(a.HasValue);   // True
        Console.WriteLine(b.HasValue);   // True
        Console.WriteLine(c.HasValue);   // False

        Console.WriteLine(a.Value);      // 5
        Console.WriteLine(c.GetValueOrDefault()); // 0
        Console.WriteLine(c.GetValueOrDefault(99)); // 99
    }
}
```

**Expected Output**

```
True
True
False
5
0
99
```

**Why this output occurs**

- `HasValue` distinguishes between a present value and `null`.
- `GetValueOrDefault()` returns `0` (the default for `int`) when null.
- The overload with a fallback returns the supplied fallback (`99`).

---

#### Example 2: Null-Conditional and Null-Coalescing with `T?`

```csharp
using System;

class Program
{
    static void Main()
    {
        int? maybe = null;
        int result = maybe ?? -1;
        Console.WriteLine(result);        // -1

        int? maybe2 = 42;
        Console.WriteLine(maybe2?.ToString() ?? "null"); // 42
    }
}
```

**Expected Output**

```
-1
42
```

**Why this output occurs**

- The null-coalescing operator `??` supplies a fallback when the nullable is null.
- The null-conditional operator `?.` safely accesses members without throwing when null.

---

### Real-World Cases

- **Database ORM** — Mapping nullable database columns to `T?` properties.
- **Form validation** — Distinguishing between “field left blank” (`null`) and “field explicitly set to zero” (`0`).
- **APIs** — Optional query parameters and partial updates.
- **Configuration** — Settings that may or may not be present.

---

### References

- Nullable value types — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/nullable-value-types
- Nullable<T> struct — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.nullable-1
- Types — C# Language Specification (nullable value types) - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/language-specification/types

---

## 8. `Nullable<T>`

### Definitions

**Core Definition**
`Nullable<T>` is the generic struct that provides the implementation for nullable value types.

**Technical Definition**
`System.Nullable<T>` is a generic value type that wraps a non-nullable value type `T` and adds a Boolean `hasValue` flag. It is the underlying implementation of the `T?` syntax. `Nullable<T>` provides the `HasValue` and `Value` properties, as well as methods such as `GetValueOrDefault()`. The type parameter `T` is constrained to be a non-nullable value type.

**Beginner-Friendly Explanation**
`Nullable<T>` is the “box” that holds either a value of type `T` or nothing at all. When you write `int?`, you are using `Nullable<int>`.

---

### Purposes

- To provide the underlying implementation for the `T?` syntax.
- To expose `HasValue` and `Value` for checking and retrieving the value.
- To offer `GetValueOrDefault()` and other helper methods.
- To enable boxing and unboxing of nullable value types.
- To support generic algorithms that operate on nullable values.

---

### Syntax Rules and Structure

```
Nullable<T>   identifier   ;
```

**Component breakdown**

- `T` — The underlying non-nullable value type.
- `identifier` — The variable name.

#### Syntax Rules

- `T` must be a non-nullable value type.
- `Nullable<T>` is a struct and therefore a value type.
- `Nullable<T>` is interchangeable with `T?`.
- `Nullable<T>` is boxed as either `null` (if `HasValue` is `false`) or a boxed `T` (if `HasValue` is `true`).

#### Constraints and Limitations

- `T` cannot be a reference type or a nullable value type.
- `Nullable<T>` itself is a value type; it is copied on assignment.
- Its members are limited; it does not expose the full interface set of `T`.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Using `Nullable<T>` Directly

```csharp
using System;

class Program
{
    static void Main()
    {
        Nullable<int> n = 42;
        Console.WriteLine(n.HasValue);   // True
        Console.WriteLine(n.Value);      // 42

        Nullable<int> empty = null;
        Console.WriteLine(empty.HasValue); // False
        Console.WriteLine(empty.GetValueOrDefault(7)); // 7
    }
}
```

**Expected Output**

```
True
42
False
7
```

**Why this output occurs**

- `Nullable<int>` behaves exactly like `int?`.
- `GetValueOrDefault(7)` returns `7` when the nullable has no value.

---

#### Example 2: Boxing a `Nullable<T>`

```csharp
using System;

class Program
{
    static void Main()
    {
        int? withValue = 42;
        int? withoutValue = null;

        object boxed1 = withValue;   // boxes as int 42
        object boxed2 = withoutValue; // null

        Console.WriteLine(boxed1);   // 42
        Console.WriteLine(boxed2 == null); // True
    }
}
```

**Expected Output**

```
42
True
```

**Why this output occurs**

- Boxing a nullable with a value boxes the underlying value.
- Boxing a null nullable produces a null reference.

---

### Real-World Cases

- **Generic libraries** — APIs that accept `Nullable<T>` to handle optional values.
- **Data access layers** — Mapping nullable database columns.
- **Serialization** — Handling optional fields in JSON/XML.

---

### References

- Nullable<T> struct — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.nullable-1
- Nullable value types — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/nullable-value-types

---

## 9. `T?` (Nullable Shorthand)

### Definitions

**Core Definition**
`T?` is the shorthand syntax for `Nullable<T>`, representing a nullable value type.

**Technical Definition**
The syntax `T?` where `T` is a non-nullable value type is shorthand for `System.Nullable<T>`. The two forms are interchangeable in source code. `T?` is used throughout C# for declaring nullable value types, pattern matching, and null-conditional expressions.

**Beginner-Friendly Explanation**
`T?` is just a shorter way to write `Nullable<T>`. For example, `int?` means “an `int` that can also be `null`.”

---

### Purposes

- To provide a concise, readable syntax for nullable value types.
- To improve code clarity when declaring optional values.
- To integrate with null-conditional and null-coalescing operators.
- To enable pattern matching with `null` checks.
- To align with nullable reference type syntax (`string?`) for consistency.

---

### Syntax Rules and Structure

```
T?   identifier   [ = value | null ]   ;
```

**Component breakdown**

- `T` — A non-nullable value type.
- `?` — Nullable modifier.
- `identifier` — The variable name.

#### Syntax Rules

- `T` must be a non-nullable value type.
- `T?` is interchangeable with `Nullable<T>`.
- Implicit conversions exist from `T` to `T?` and from `null` to `T?`.
- `T?` cannot be nested (`int??` is invalid).

#### Constraints and Limitations

- `T?` is not allowed when `T` is already a nullable value type or a reference type (use nullable reference types for the latter).
- Runtime behaviour is identical to `Nullable<T>`.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: `int?` and `bool?`

```csharp
using System;

class Program
{
    static void Main()
    {
        int? age = null;
        bool? isVerified = true;

        Console.WriteLine(age.HasValue);      // False
        Console.WriteLine(isVerified.Value);  // True

        age = 30;
        Console.WriteLine(age.Value);         // 30
    }
}
```

**Expected Output**

```
False
True
30
```

**Why this output occurs**

- `int?` and `bool?` are nullable value types.
- `HasValue` and `Value` work the same as with `Nullable<T>`.

---

#### Example 2: Pattern Matching with `T?`

```csharp
using System;

class Program
{
    static void Main()
    {
        int? maybe = 42;

        if (maybe is int value)
        {
            Console.WriteLine($"Has value: {value}");
        }
        else
        {
            Console.WriteLine("No value");
        }
    }
}
```

**Expected Output**

```
Has value: 42
```

**Why this output occurs**

- The pattern `is int value` matches when the nullable has a value and extracts it into `value`.

---

### Real-World Cases

- **Optional parameters** — Methods that accept `int?` to distinguish “not supplied” from a default value.
- **DTOs and view models** — Properties that may be null.
- **LINQ queries** — Filtering and projecting nullable values.

---

### References

- Nullable value types — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/nullable-value-types
- Types — C# Language Specification (nullable value types) - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/language-specification/types

---

## 10. Boxing

### Definitions

**Core Definition**
Boxing is the process of converting a value type to a reference type (typically `object` or an interface type) by wrapping the value in a heap-allocated object.

**Technical Definition**
Boxing converts a value type to a reference type by allocating an object on the garbage-collected heap and copying the value into it. The boxed object is an instance of the value type’s runtime type. Boxing occurs implicitly when a value type is assigned to `object`, passed to a parameter of type `object`, or stored in a non-generic collection. The C# Language Specification describes boxing and unboxing as providing a bridge between value types and reference types by permitting any value of a value type to be converted to and from type `object`.

**Beginner-Friendly Explanation**
Boxing is like putting a small item into a shipping box. The item (the value) is placed inside a new box (a heap object) so it can be treated as a reference type. This is convenient but costs time and memory because a new object must be created.

---

### Purposes

- To allow value types to be treated as objects for unified type-system operations.
- To store value types in non-generic collections such as `ArrayList`.
- To pass value types to APIs that accept `object`.
- To call `object` methods such as `ToString()` or `GetHashCode()` on value types (though many are overridden to avoid boxing).
- To enable interface-based polymorphism with value types that implement interfaces.

---

### Syntax Rules and Structure

Boxing occurs implicitly in several contexts:

```
object   o   =   valueTypeVariable   ;      // implicit boxing
IComparable   c   =   valueTypeVariable   ; // implicit boxing to interface
ArrayList   list   =   new ArrayList();
list.Add(42);                                // implicit boxing
```

**Component breakdown**

- `valueTypeVariable` — A variable of a value type.
- `object` / interface target — The reference type to which the value is boxed.
- `ArrayList.Add` — A non-generic API that accepts `object`, triggering boxing.

#### Syntax Rules

- Boxing is implicit when a value type is converted to `object` or an interface type it implements.
- A new heap object is allocated for each boxing operation.
- The boxed object contains a copy of the value.
- Boxing a nullable value type with no value produces a `null` reference; boxing a nullable with a value boxes the underlying value.

#### Constraints and Limitations

- Boxing allocates memory on the heap and creates garbage-collection pressure.
- Each boxing operation creates a separate object, even for the same value.
- Boxed values are immutable in the sense that the box contains a copy; modifying the original value does not affect the boxed copy.
- Boxing can be a significant performance cost in hot loops and non-generic collections.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Implicit Boxing to `object`

```csharp
using System;

class Program
{
    static void Main()
    {
        int value = 42;
        object boxed = value;          // boxing

        value = 100;

        Console.WriteLine(boxed);       // 42
        Console.WriteLine(value);       // 100
        Console.WriteLine(ReferenceEquals(boxed, value)); // False (different types)
    }
}
```

**Expected Output**

```
42
100
False
```

**Why this output occurs**

- `boxed` holds a heap-allocated copy of `42`.
- Changing `value` to `100` does not affect the boxed copy.
- `ReferenceEquals` returns `false` because `boxed` and `value` are not the same object (and `value` is a value type).

---

#### Example 2: Boxing in Non-Generic Collections

```csharp
using System;
using System.Collections;

class Program
{
    static void Main()
    {
        ArrayList list = new ArrayList();
        list.Add(1);          // boxing
        list.Add(2);          // boxing
        list.Add(3);          // boxing

        int sum = 0;
        foreach (int n in list)   // unboxing each element
        {
            sum += n;
        }
        Console.WriteLine(sum);   // 6
    }
}
```

**Expected Output**

```
6
```

**Why this output occurs**

- Each `Add` boxes the integer.
- The `foreach` unboxes each element back to `int` before adding.
- This illustrates the boxing/unboxing overhead of non-generic collections.

---

### Real-World Cases

- **Legacy .NET 1.x collections** — `ArrayList`, `Hashtable`, and other non-generic collections box value types.
- **Reflection and dynamic invocation** — APIs that accept `object` parameters.
- **Logging and formatting** — `string.Format` and similar APIs may box value types.
- **Interface dispatch** — Calling interface methods on structs can box the struct if not constrained properly.

---

### References

- Boxing and unboxing — C# Programming Guide - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/types/boxing-and-unboxing
- Types — C# Language Specification (boxing and unboxing) - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/language-specification/types
- .NET Performance Tips — Boxing and Unboxing - https://learn.microsoft.com/en-us/dotnet/framework/performance/performance-tips

---

## 11. Unboxing

### Definitions

**Core Definition**
Unboxing is the process of converting a boxed reference-type object back to a value type.

**Technical Definition**
Unboxing extracts the value from a boxed object and copies it to a value-type variable. Unboxing requires an explicit cast. The runtime checks that the boxed object’s type matches the target value type; if it does not, an `InvalidCastException` is thrown. Unboxing is the reverse of boxing.

**Beginner-Friendly Explanation**
Unboxing is like taking the item out of the shipping box. You must explicitly ask for the specific type you expect, and if the box contains a different type, you get an error.

---

### Purposes

- To retrieve the original value from a boxed object.
- To convert `object` references back to specific value types.
- To enable value-type operations after boxing.
- To support non-generic collections and APIs that return `object`.
- To allow pattern matching and type-checking on boxed values.

---

### Syntax Rules and Structure

```
valueType   variable   =   (valueType)   boxedObject   ;
```

**Component breakdown**

- `boxedObject` — An `object` or interface reference containing a boxed value.
- `(valueType)` — Explicit cast to the target value type.
- `variable` — The value-type variable that receives the unboxed value.

#### Syntax Rules

- Unboxing requires an explicit cast.
- The boxed object must contain a value of the exact target type (or a compatible type).
- An `InvalidCastException` is thrown if the type does not match.
- Unboxing a `null` reference throws `NullReferenceException`.

#### Constraints and Limitations

- Unboxing performs a runtime type check, which has a performance cost.
- Unboxing copies the value; modifying the unboxed copy does not affect the boxed object.
- Unboxing to a different type than the boxed type fails.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Explicit Unboxing

```csharp
using System;

class Program
{
    static void Main()
    {
        object boxed = 42;              // boxing
        int unboxed = (int)boxed;       // unboxing
        Console.WriteLine(unboxed);     // 42

        try
        {
            long wrong = (long)boxed;   // InvalidCastException
        }
        catch (InvalidCastException)
        {
            Console.WriteLine("Cannot unbox int as long");
        }
    }
}
```

**Expected Output**

```
42
Cannot unbox int as long
```

**Why this output occurs**

- The boxed object contains an `int`.
- Unboxing to `long` fails because the runtime type does not match the target type.

---

#### Example 2: Unboxing in a Non-Generic Collection

```csharp
using System;
using System.Collections;

class Program
{
    static void Main()
    {
        ArrayList list = new ArrayList();
        list.Add(100);                  // boxing
        int value = (int)list[0];       // unboxing
        Console.WriteLine(value);       // 100
    }
}
```

**Expected Output**

```
100
```

**Why this output occurs**

- `list[0]` returns `object`; the explicit cast `(int)` unboxes the value.

---

### Real-World Cases

- **Non-generic collections** — Retrieving values from `ArrayList`, `Hashtable`, etc.
- **Reflection and dynamic APIs** — Unboxing results of method invocations.
- **Interop** — Converting `object` parameters from COM or dynamic languages.

---

### References

- Boxing and unboxing — C# Programming Guide - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/types/boxing-and-unboxing
- Types — C# Language Specification (unboxing) - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/language-specification/types

---

## 12. Boxing Performance

### Definitions

**Core Definition**
Boxing performance refers to the runtime cost—CPU time and memory allocation—incurred when value types are converted to reference types.

**Technical Definition**
Boxing is computationally expensive relative to simple assignments. When a value type is boxed, an entirely new object must be allocated and constructed on the GC heap. According to .NET performance guidance, boxing can take up to **20 times longer** than a simple reference assignment, and unboxing (the cast) can take up to **four times** as long as an assignment. Boxing also increases garbage-collection pressure, which can degrade overall application throughput.

**Beginner-Friendly Explanation**
Boxing is slow because it creates a brand-new box on the heap every time. If you do it in a loop, you generate a lot of garbage, and the garbage collector has to work harder. Avoiding boxing is a key way to make code faster.

---

### Purposes

- To understand the performance implications of boxing in hot paths.
- To identify opportunities to replace non-generic collections with generic collections.
- To guide the design of APIs that avoid unnecessary boxing.
- To motivate the use of generics, `ref struct`, and other allocation-reduction techniques.
- To measure and profile boxing overhead in real applications.

---

### Syntax Rules and Structure (Performance Patterns)

```
// Avoid: non-generic collection boxes every element
ArrayList list = new ArrayList();
list.Add(42);   // boxing

// Prefer: generic collection stores values directly
List<int> list = new List<int>();
list.Add(42);   // no boxing
```

**Component breakdown**

- `ArrayList` — Non-generic collection that stores `object`; every value type is boxed.
- `List<int>` — Generic collection that stores `int` directly without boxing.
- `object` parameters — APIs that accept `object` force boxing for value-type arguments.

#### Guidelines

- Prefer generic collections (`List<T>`, `Dictionary<TKey,TValue>`) over non-generic collections.
- Use generic constraints to avoid boxing when calling interface methods on structs.
- Use `ref struct` types (`Span<T>`, `ReadOnlySpan<T>`) for stack-only operations.
- Cache boxed values when the same box can be reused (though this is rarely worth it unless profiling shows a hot spot).
- Consider `EqualityComparer<T>.Default` instead of non-generic `Equals(object)` to avoid boxing in comparisons.

#### Constraints and Limitations

- Some APIs inherently require `object` (e.g., reflection, `string.Format` with object arguments); boxing may be unavoidable.
- .NET 9 introduced **Object Stack Allocation** via JIT escape analysis, which can eliminate boxing in some cases where the boxed reference does not escape the method scope.
- The performance cost of boxing is relative; for infrequent operations it may be negligible.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Boxing in a Loop vs. Generic Collection

```csharp
using System;
using System.Collections;
using System.Collections.Generic;
using System.Diagnostics;

class Program
{
    static void Main()
    {
        const int N = 1_000_000;

        // Non-generic: boxing each element
        var sw = Stopwatch.StartNew();
        ArrayList arrayList = new ArrayList();
        for (int i = 0; i < N; i++)
            arrayList.Add(i);          // boxing
        sw.Stop();
        Console.WriteLine($"ArrayList: {sw.ElapsedMilliseconds} ms");

        // Generic: no boxing
        sw.Restart();
        List<int> list = new List<int>();
        for (int i = 0; i < N; i++)
            list.Add(i);               // no boxing
        sw.Stop();
        Console.WriteLine($"List<int>: {sw.ElapsedMilliseconds} ms");
    }
}
```

**Expected Output (approximate)**

```
ArrayList: 25 ms
List<int>: 5 ms
```

(Actual timings vary by hardware and runtime version.)

**Why this output occurs**

- `ArrayList.Add` boxes each `int`, allocating a heap object and copying the value.
- `List<int>.Add` stores the `int` directly in the backing array.
- The generic version avoids allocation and GC pressure, resulting in lower elapsed time.

---

#### Example 2: Avoiding Boxing with Generic Constraints

```csharp
using System;

interface IValue { int Get(); }

struct MyStruct : IValue
{
    public int Value;
    public int Get() => Value;
}

class Program
{
    // Boxing version: T is not constrained, interface call boxes
    static int GetViaInterface<T>(T item) where T : IValue
    {
        return item.Get();   // may box if T is a struct
    }

    // No boxing: use generic constraint and EqualityComparer<T>
    static bool AreEqual<T>(T a, T b) where T : IEquatable<T>
    {
        return a.Equals(b);  // no boxing for value types
    }

    static void Main()
    {
        var s = new MyStruct { Value = 42 };
        Console.WriteLine(GetViaInterface(s));   // 42 (may box)
        Console.WriteLine(AreEqual(1, 1));        // True (no boxing)
    }
}
```

**Expected Output**

```
42
True
```

**Why this output occurs**

- `GetViaInterface` may box `s` when calling the interface method, depending on JIT optimization.
- `AreEqual` uses `IEquatable<T>` constraint, allowing `int` to call its `Equals` method without boxing.
- The output is the same, but the second call avoids allocation.

---

### Real-World Cases

- **High-frequency trading systems** — Avoiding boxing in order-processing loops.
- **Game engines** — Using generic collections and structs to avoid GC spikes.
- **Web APIs** — Reducing allocations per request by using generic types and `Span<T>`.
- **Data processing pipelines** — Using `List<T>` instead of `ArrayList` to avoid boxing overhead.

---

### References

- Boxing and unboxing — C# Programming Guide (Performance section) - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/types/boxing-and-unboxing
- .NET Performance Tips — Boxing and Unboxing - https://learn.microsoft.com/en-us/dotnet/framework/performance/performance-tips
- Boxing in C#: What It Costs You and How to Get Rid of It - https://blog.ndepend.com/boxing-in-c-what-it-costs-you-and-how-to-get-rid-of-it/
- Performance Improvements in .NET 9 — Object Stack Allocation - https://devblogs.microsoft.com/dotnet/performance-improvements-in-net-9/#object-stack-allocation

---

## 13. Value Type Best Practices

### Definitions

**Core Definition**
Value type best practices are the guidelines for choosing, designing, and using value types to maximize correctness, performance, and maintainability.

**Technical Definition**
Best practices for value types derive from the .NET Framework Design Guidelines and the C# language specification. They cover when to use `struct` versus `class`, how to design immutable value types, how to avoid boxing, and how to handle nullable value types correctly. The guidelines are not enforced by the compiler but are supported by analyzers, style rules, and community conventions.

**Beginner-Friendly Explanation**
Best practices are the habits that keep value-type code fast and bug-free: keep structs small, make them immutable, avoid boxing in hot loops, and use nullable types when a value might be missing.

---

### Purposes

- To choose the right type (struct vs. class) for the job.
- To design value types that are safe, predictable, and efficient.
- To avoid common pitfalls such as mutable structs and excessive boxing.
- To improve performance by reducing heap allocations.
- To align with .NET ecosystem conventions and tooling.

---

### Guidelines and Structure

#### General Guidelines

```
// Prefer readonly struct for small immutable values
public readonly struct Point
{
    public readonly int X, Y;
    public Point(int x, int y) => (X, Y) = (x, y);
}

// Prefer generic collections over non-generic to avoid boxing
List<int> numbers = new List<int>();

// Use nullable value types for optional values
int? optional = null;

// Avoid mutable structs
// Prefer properties over public fields
```

**Component breakdown**

- **`readonly struct`** — Makes all instance fields readonly; prevents accidental mutation.
- **Generic collections** — Store value types without boxing.
- **Nullable types** — Represent optional values explicitly.
- **Immutable design** — Makes value types safe to copy and share.
- **Small size** — Keeps copies cheap (guideline: under 16 bytes).

#### Rules and Constraints

- **Consider `struct` instead of `class`** if instances are small, short-lived, commonly embedded in other objects, logically represent a single value, have an instance size under 16 bytes, are immutable, and are not frequently boxed.
- **Do not define a mutable struct** — Mutable structs can cause subtle bugs when modified through copies.
- **Prefer `readonly struct`** for new structs when possible (C# 7.2+).
- **Avoid boxing** — Use generics, `ref struct`, and constrained generic methods.
- **Use `T?` for optional value-type data** rather than sentinel values.
- **Do not use `struct` for large data** — Copies become expensive.
- **Do not use `struct` when inheritance or reference semantics are needed**.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Good vs. Poor Struct Design

```csharp
using System;

// GOOD: small, immutable, readonly struct
public readonly struct Temperature
{
    public readonly double Celsius;
    public Temperature(double celsius) => Celsius = celsius;
    public double Fahrenheit => Celsius * 9 / 5 + 32;
}

// POOR: mutable, large struct with reference-like behaviour
public struct MutableBuffer
{
    public byte[] Data;   // reference-type field in a struct
    public int Length;
}

class Program
{
    static void Main()
    {
        var t = new Temperature(100);
        Console.WriteLine(t.Fahrenheit);   // 212

        var b1 = new MutableBuffer { Data = new byte[10], Length = 10 };
        var b2 = b1;
        b2.Length = 20;

        Console.WriteLine(b1.Length);      // 10 (copy semantics)
        Console.WriteLine(b2.Length);      // 20
    }
}
```

**Expected Output**

```
212
10
20
```

**Why this output occurs**

- `Temperature` is a readonly struct with value semantics; copying is safe and cheap.
- `MutableBuffer` contains a reference-type field; copying the struct copies the reference, leading to shared mutable state even though the struct itself is a value type. This is a common design pitfall.

---

#### Example 2: Avoiding Boxing in a Generic Method

```csharp
using System;
using System.Collections.Generic;

class Program
{
    // Good: generic method avoids boxing
    static T Max<T>(T a, T b) where T : IComparable<T>
    {
        return a.CompareTo(b) >= 0 ? a : b;
    }

    static void Main()
    {
        int max = Max(3, 7);
        Console.WriteLine(max);   // 7
    }
}
```

**Expected Output**

```
7
```

**Why this output occurs**

- The `IComparable<T>` constraint allows `int` to call `CompareTo` without boxing.
- If the constraint were non-generic `IComparable`, the `int` would be boxed for the interface call.

---

### Real-World Cases

- **Library design** — Public APIs should prefer immutable structs and generic collections.
- **Performance-critical code** — Avoiding boxing and using `Span<T>` / `readonly struct` reduces GC pressure.
- **Domain modelling** — Small value objects (money, coordinates, identifiers) benefit from value semantics.

---

### References

- Choosing Between Classes and Structures - https://learn.microsoft.com/en-us/dotnet/standard/design-guidelines/choosing-between-class-and-struct
- Value Type Usage Guidelines - https://learn.microsoft.com/en-us/previous-versions/dotnet/netframework-2.0/ms229017(v=vs.80)
- C# Coding Conventions - https://learn.microsoft.com/en-us/dotnet/csharp/fundamentals/coding-style/coding-conventions
- .NET Runtime coding style - https://github.com/dotnet/runtime/blob/main/docs/coding-guidelines/coding-style.md

---

## Deprecated, Unsafe, Implementation-Dependent, and Version-Specific Notes

| Feature | Status / Note |
|---------|---------------|
| `nint` / `nuint` | Introduced in **C# 9.0**; version-specific. Not CLS-compliant; avoid in public cross-language APIs. |
| `readonly struct` | Introduced in **C# 7.2**; version-specific. |
| Parameterless struct constructors | Introduced in **C# 10**; prior versions disallowed explicit parameterless struct constructors. |
| `ref struct` | Stack-only guarantee; cannot be boxed or stored on the heap. |
| Stack allocation of value types | **Implementation-dependent**; the specification guarantees lifetime and assignment semantics, not physical storage location. |
| Boxing cost | **Version- and runtime-dependent**; .NET 9+ can eliminate some boxing via Object Stack Allocation and JIT escape analysis. |
| Mutable structs | **Discouraged**; can lead to subtle bugs due to copy semantics. |
| Public mutable fields | **Discouraged**; prefer properties for encapsulation. |
| `ArrayList` / non-generic collections | **Legacy**; cause boxing of value types. Prefer generic collections. |

---

## References (Consolidated)

- Value types — C# Reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/value-types
- Types — C# Language Specification - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/language-specification/types
- ValueType Class (System) - https://learn.microsoft.com/en-us/dotnet/api/system.valuetype
- Structures — C# Language Specification - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/language-specification/structs
- Structure types — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/struct
- Enumeration types — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/enum
- enum keyword — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/enum
- Enums — C# Language Specification - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/language-specification/enums
- Nullable value types — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/nullable-value-types
- Nullable<T> struct — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.nullable-1
- Native-sized integers — C# feature specifications - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/proposals/csharp-9.0/native-integers
- nint and nuint types — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/nint-nuint
- Integral numeric types — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/integral-numeric-types
- Floating-point numeric types — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/floating-point-numeric-types
- bool — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/bool
- char — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/char
- decimal — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/floating-point-numeric-types#the-decimal-type
- Boxing and unboxing — C# Programming Guide - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/types/boxing-and-unboxing
- .NET Performance Tips — Boxing and Unboxing - https://learn.microsoft.com/en-us/dotnet/framework/performance/performance-tips
- Boxing in C#: What It Costs You and How to Get Rid of It - https://blog.ndepend.com/boxing-in-c-what-it-costs-you-and-how-to-get-rid-of-it/
- Performance Improvements in .NET 9 — Object Stack Allocation - https://devblogs.microsoft.com/dotnet/performance-improvements-in-net-9/#object-stack-allocation
- Choosing Between Classes and Structures - https://learn.microsoft.com/en-us/dotnet/standard/design-guidelines/choosing-between-class-and-struct
- Value Type Usage Guidelines - https://learn.microsoft.com/en-us/previous-versions/dotnet/netframework-2.0/ms229017(v=vs.80)
- The Stack Is An Implementation Detail, Part One - https://learn.microsoft.com/en-us/archive/blogs/ericlippert/the-stack-is-an-implementation-detail-part-one
- The Truth About Value Types - https://learn.microsoft.com/en-us/archive/blogs/ericlippert/the-truth-about-value-types
- ref struct — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/ref-struct
- C# Coding Conventions - https://learn.microsoft.com/en-us/dotnet/csharp/fundamentals/coding-style/coding-conventions
- .NET Runtime coding style - https://github.com/dotnet/runtime/blob/main/docs/coding-guidelines/coding-style.md

---