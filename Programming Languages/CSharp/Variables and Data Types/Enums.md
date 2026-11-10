# C# Enums — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
An enum (enumeration type) is a distinct value type that declares a set of named constants backed by an underlying integral type. Enums are used when a value must be one of a fixed set of options, such as days of the week, HTTP status codes, log levels, or directions. They make code more readable and less error-prone than raw integer constants because the compiler enforces the named values.

**Technical Definition**
An enum type is a distinct value type that declares a set of named constants. Every enum type has a corresponding integral type called the **underlying type**, which must be able to represent all enumerator values defined in the enumeration. If no underlying type is explicitly declared, the underlying type is `int`. The underlying type must be one of the integral types (`byte`, `sbyte`, `short`, `ushort`, `int`, `uint`, `long`, `ulong`) but cannot be `char`. An enum declaration defines the name, accessibility, underlying type, and members of the enum. Enum members are constants whose values are in the range of the underlying type.

**Beginner-Friendly Explanation**
An enum is a **named list of options**. Instead of remembering that Monday is `1` and Tuesday is `2`, you write `DayOfWeek.Monday` and `DayOfWeek.Tuesday`. The compiler ensures you only use the names that exist in the list, which makes your code safer and easier to read.

---

### Key Characteristics

- **Distinct value type** — Enums are value types (structs) and derive implicitly from `System.Enum`, which derives from `System.ValueType`.
- **Named constants** — Each member is a named constant with an underlying integral value.
- **Fixed set of options** — Enums are ideal for representing a closed set of choices; open sets (like OS versions) are discouraged.
- **Default underlying type** — `int` unless explicitly specified; other integral types are allowed except `char`.
- **Implicit sequential values** — The first member defaults to `0`, and each subsequent member increments by `1` unless explicit values are assigned.
- **Bit flags support** — The `[Flags]` attribute enables enums to represent combinations of choices using powers of two.
- **Compile-time substitution** — All references to enum values are converted to numeric literals at compile time, which can create versioning considerations.

---

### Prerequisites

- Basic understanding of C# program structure (`class`, `Main`, namespaces).
- Familiarity with value types and integral types (`int`, `byte`, etc.).
- Knowledge of `switch` statements and pattern matching (helpful for using enums).
- Awareness of the `System` namespace and `Console` class for examples.
- Understanding of bitwise operators (`|`, `&`, `^`) for flag enums.

---

### Related Programming Areas

- **Type safety** — Enums strongly type parameters, properties, and return values that represent sets of values.
- **Bit manipulation** — Flag enums use bitwise operations to combine and test values.
- **Serialization** — Enums are commonly serialized as names or numeric values in JSON/XML.
- **Switch expressions** — Enums integrate naturally with `switch` expressions and pattern matching.
- **API design** — Enums provide self-documenting, discoverable constants for public APIs.

---

### Core Concepts / Features

1. Enums (overarching concept)
2. Enum declaration
3. Enum values
4. Enum underlying type
5. Enum methods (`Enum.GetValues()`, `Enum.GetNames()`, `Enum.Parse()`, `Enum.TryParse()`, `Enum.IsDefined()`, `Enum.GetUnderlyingType()`)
6. `[Flags]` attribute
7. Flag enums
8. Enum best practices

Each concept follows a uniform structure: definitions, purposes, syntax rules and structure, annotated examples with expected outputs, real-world cases, and references.

---

## 1. Enums (Overarching Concept)

### Definitions

**Core Definition**
An enum is a value type that defines a set of named integral constants, providing a type-safe way to work with fixed sets of options.

**Technical Definition**
`System.Enum` is the abstract base class for all enumeration types. Enum types are distinct value types that declare a set of named constants. Each enum member has an associated constant value of the underlying integral type. Enums support conversions to and from their underlying types, parsing from strings, and enumeration of their members via reflection-based methods.

**Beginner-Friendly Explanation**
An enum is a **predefined list of named values**. You use the names instead of raw numbers, which makes your code clearer and prevents mistakes like passing `5` when only `1`, `2`, or `3` are valid.

---

### Purposes

- To strongly type parameters, properties, and return values that represent sets of values.
- To replace magic numbers and static constants with readable, discoverable names.
- To enable compiler checking of valid values.
- To provide a natural fit for `switch` expressions and pattern matching.
- To support bit flags for representing combinations of choices.

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
- `EnumName` — The type name (PascalCase by convention).
- `: underlyingType` — Optional; defaults to `int`. Must be an integral type except `char`.
- `Member` — Named constant.
- `= value` — Optional explicit value; subsequent members increment by `1`.

#### Syntax Rules

- Enum members must be unique within the enum.
- The first member defaults to `0` and increments by `1`.
- Values must be within the range of the underlying type.
- Enums can be declared at namespace level or nested within a class or struct.
- Enums cannot contain methods, properties, or events (extension methods can be written for them).

#### Constraints and Limitations

- Enums cannot inherit from other types.
- A variable of an enum type can hold any value in the range of the underlying type, even if it is not a named member.
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

- `Spring` is `0`, `Summer` is `1`, etc., following the default sequential assignment.
- `Enum.GetValues` returns all members in order.
- The explicit cast `(int)` reveals the underlying integer value.

---

#### Example 2: Enum in a Switch Expression

```csharp
using System;

public enum LogLevel { Debug, Info, Warning, Error }

class Program
{
    static string Describe(LogLevel level) => level switch
    {
        LogLevel.Debug => "Detailed diagnostic",
        LogLevel.Info => "General information",
        LogLevel.Warning => "Potential issue",
        LogLevel.Error => "Failure",
        _ => "Unknown"
    };

    static void Main()
    {
        Console.WriteLine(Describe(LogLevel.Warning)); // Potential issue
    }
}
```

**Expected Output**

```
Potential issue
```

**Why this output occurs**

- The `switch` expression matches the enum member and returns the corresponding description.
- The discard pattern `_` handles any undefined numeric value.

---

### Real-World Cases

- **HTTP status codes** — `Ok`, `NotFound`, `InternalServerError`.
- **Log levels** — `Debug`, `Info`, `Warning`, `Error`.
- **Days of the week, months, seasons** — Fixed sets of named values.
- **Order status** — `Pending`, `Processing`, `Shipped`, `Delivered`.

---

### References

- C# Enumerations — Microsoft Learn - https://learn.microsoft.com/en-us/dotnet/csharp/fundamentals/types/enums
- Enums — C# Language Specification - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/language-specification/enums

---

## 2. Enum Declaration

### Definitions

**Core Definition**
Enum declaration is the syntax that introduces a new enumeration type, specifying its name, accessibility, underlying type, and members.

**Technical Definition**
An enum declaration begins with the `enum` keyword and defines the name, accessibility, underlying type, and members of the enum. The declaration grammar is: `attributes? enum_modifier* 'enum' identifier enum_base? enum_body`. The `enum_base` is optional and, when present, explicitly declares the underlying type, which must be an integral type other than `char`.

**Beginner-Friendly Explanation**
Declaring an enum is **creating a new named list of options**. You give the list a name, optionally choose what kind of numbers back it, and list the members.

---

### Purposes

- To introduce a new type representing a fixed set of values.
- To specify the underlying storage type for the members.
- To define the exact set of named constants available to consumers.
- To control the accessibility and scope of the enum.
- To group related constants under a single type name.

---

### Syntax Rules and Structure

```
[attributes]   [modifiers]   enum   EnumName   [: integralType]
{
    Member1,
    Member2,
    ...
}
```

**Component breakdown**

- `attributes` — Optional attributes (e.g., `[Flags]`).
- `modifiers` — Optional accessibility modifiers (`public`, `internal`, etc.).
- `enum` — Keyword.
- `EnumName` — The type name.
- `: integralType` — Optional underlying type.
- `{ ... }` — Enumerator list.

#### Syntax Rules

- The enum body can have a trailing comma.
- The underlying type can be specified by keyword or type name.
- Enum declarations cannot include a type parameter list, but nested enums inside generic types are allowed.
- Enums are usually defined directly within a namespace for equal access by all classes.

#### Constraints and Limitations

- `char` cannot be used as an underlying type.
- Members cannot contain whitespace in their names.
- Enums cannot have a parameterless constructor explicitly.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Basic Declaration

```csharp
using System;

public enum Direction
{
    North,
    East,
    South,
    West
}

class Program
{
    static void Main()
    {
        Console.WriteLine((int)Direction.North); // 0
        Console.WriteLine((int)Direction.West);  // 3
    }
}
```

**Expected Output**

```
0
3
```

**Why this output occurs**

- Members are assigned sequential values starting at `0`.
- Casting reveals the underlying `int` values.

---

#### Example 2: Declaration with Explicit Underlying Type

```csharp
using System;

public enum StatusCode : byte
{
    Ok = 200,
    NotFound = 404,
    ServerError = 500
}

class Program
{
    static void Main()
    {
        Console.WriteLine((byte)StatusCode.NotFound); // 404
    }
}
```

**Expected Output**

```
404
```

**Why this output occurs**

- The underlying type is `byte`, and explicit values are assigned.
- Casting to `byte` reveals the stored value.

---

### Real-World Cases

- **Protocol identifiers** — Using explicit values for wire compatibility.
- **Flags** — Declaring members as powers of two for bitwise combinations.
- **Domain models** — Declaring order statuses, user roles, or categories.

---

### References

- enum keyword — C# Reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/enum
- Enums — C# Language Specification — Enum declarations - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/language-specification/enums#192-enum-declarations

---

## 3. Enum Values

### Definitions

**Core Definition**
Enum values are the named constants and their associated integral values that make up an enumeration.

**Technical Definition**
Each enum member is a named constant whose value is of the enum’s underlying type. By default, the first member has the value `0`, and each successive member is increased by `1`. Explicit initializers can override these values. The value of an enum member must be within the range of the underlying type. A variable of an enum type can be assigned any value in the range of the underlying type, not just the named constants.

**Beginner-Friendly Explanation**
Enum values are the **numbers behind the names**. `Spring` might be `0`, `Summer` `1`, etc. You can change these numbers if the numbers have special meaning, like HTTP status codes.

---

### Purposes

- To assign meaningful numeric values when they have external significance.
- To control ordering and comparison of enum members.
- To allow non-contiguous values for protocol compatibility.
- To support bit flags via powers of two.
- To provide a default zero value for uninitialized enums.

---

### Syntax Rules and Structure

```
enum   EnumName
{
    Member1   =   constantExpression1,
    Member2,                    // Member1 + 1
    Member3   =   constantExpression3
}
```

**Component breakdown**

- `Member1 = constantExpression1` — Explicit value.
- `Member2` (no initializer) — Previous member’s value + 1.
- `constantExpression` — A compile-time constant of the underlying type.

#### Syntax Rules

- Values must be within the range of the underlying type.
- Implicit values follow the previous member’s value plus one.
- Duplicate values are allowed but can cause ambiguity in `ToString()` and parsing.
- The default value of an enum `E` is `(E)0`.

#### Constraints and Limitations

- Values are converted to numeric literals at compile time, which can create versioning issues when enums are used across assemblies.
- A variable can hold an undefined numeric value, e.g., `(Season)99`.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Sequential and Explicit Values

```csharp
using System;

enum Priority
{
    Low,          // 0
    Medium,       // 1
    High = 10,    // 10
    Critical      // 11
}

class Program
{
    static void Main()
    {
        Console.WriteLine((int)Priority.Low);      // 0
        Console.WriteLine((int)Priority.High);     // 10
        Console.WriteLine((int)Priority.Critical); // 11
    }
}
```

**Expected Output**

```
0
10
11
```

**Why this output occurs**

- `Low` and `Medium` use default sequential values.
- `High` is explicitly `10`, so `Critical` becomes `11`.

---

#### Example 2: Undefined Values

```csharp
using System;

enum Season { Spring, Summer, Autumn, Winter }

class Program
{
    static void Main()
    {
        Season s = (Season)99;
        Console.WriteLine(s);          // 99
        Console.WriteLine(Enum.IsDefined(typeof(Season), s)); // False
    }
}
```

**Expected Output**

```
99
False
```

**Why this output occurs**

- Casting an arbitrary integer produces an enum value outside the named members.
- `IsDefined` checks whether the value corresponds to a named constant.

---

### Real-World Cases

- **HTTP status codes** — Explicit values like `200`, `404`, `500`.
- **File access flags** — Powers of two for `Read`, `Write`, `Execute`.
- **Protocol versions** — Numeric values that must match external specifications.

---

### References

- enum keyword — C# Reference (values and defaults) - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/enum
- Enums — C# Language Specification — Enum values - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/language-specification/enums

---

## 4. Enum Underlying Type

### Definitions

**Core Definition**
The underlying type of an enum is the integral type that stores the value of each member.

**Technical Definition**
Every enum has a corresponding integral type called the underlying type, which must be able to represent all enumerator values. The default underlying type is `int`. If specified, it must be one of `byte`, `sbyte`, `short`, `ushort`, `int`, `uint`, `long`, or `ulong`. `char` is explicitly disallowed. The underlying type determines the storage size and the range of valid values.

**Beginner-Friendly Explanation**
The underlying type is the **kind of number** used to store each enum value. By default, it is an `int`. You can choose a smaller type like `byte` to save memory, or a larger type like `long` for a wider range.

---

### Purposes

- To control the storage size of enum values.
- To expand the range of representable values.
- To match the size expected by unmanaged code or protocols.
- To save memory in large arrays or frequently instantiated structures.
- To provide interoperability with APIs expecting specific integer types.

---

### Syntax Rules and Structure

```
enum   EnumName   :   integralType
{
    ...
}
```

**Component breakdown**

- `: integralType` — Optional; defaults to `int`.
- `integralType` — One of `byte`, `sbyte`, `short`, `ushort`, `int`, `uint`, `long`, `ulong`.

#### Syntax Rules

- The underlying type must be an integral type.
- `char` is not permitted.
- The underlying type determines the range of valid member values.
- Default is `int` if omitted.

#### Constraints and Limitations

- The underlying type cannot be changed after the enum is compiled.
- Using a non-default underlying type may affect interoperability and serialization.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Default Underlying Type

```csharp
using System;

enum Color { Red, Green, Blue }

class Program
{
    static void Main()
    {
        Type underlying = Enum.GetUnderlyingType(typeof(Color));
        Console.WriteLine(underlying); // System.Int32
    }
}
```

**Expected Output**

```
System.Int32
```

**Why this output occurs**

- No explicit underlying type is specified, so the default `int` is used.

---

#### Example 2: Explicit Underlying Type

```csharp
using System;

enum Small : byte { A = 1, B = 2 }

class Program
{
    static void Main()
    {
        Console.WriteLine(Enum.GetUnderlyingType(typeof(Small))); // System.Byte
        Console.WriteLine(sizeof(Small)); // 1 (requires unsafe context)
    }
}
```

**Expected Output**

```
System.Byte
```

**Why this output occurs**

- The underlying type is explicitly `byte`.
- `GetUnderlyingType` confirms the storage type.

---

### Real-World Cases

- **Memory-constrained systems** — Using `byte` or `short` for small enums in large arrays.
- **Interop** — Matching the size expected by C/C++ APIs.
- **Protocols** — Using `ushort` or `uint` where the specification requires it.

---

### References

- enum keyword — C# Reference (underlying types) - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/enum
- Enum.GetUnderlyingType Method — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.enum.getunderlyingtype
- Enums — C# Language Specification — Underlying type - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/language-specification/enums#192-enum-declarations

---

## 5. Enum Methods

### Definitions

**Core Definition**
Enum methods are the static methods on `System.Enum` that provide reflection-based access to enum metadata and conversion between strings, values, and enum instances.

**Technical Definition**
`System.Enum` exposes static methods for enumerating members (`GetValues`, `GetNames`), parsing strings (`Parse`, `TryParse`), validating values (`IsDefined`), and retrieving the underlying type (`GetUnderlyingType`). These methods operate on `Type` objects and return arrays, booleans, or boxed enum values. Many have generic overloads for type safety.

**Beginner-Friendly Explanation**
Enum methods are **tools for working with enums at runtime**. They let you list all the options, get their names, turn strings into enums, check whether a value is valid, and find out what number type backs the enum.

---

### Purposes

- To enumerate all members and their names.
- To convert strings to enum values safely.
- To validate that a value or name corresponds to a defined member.
- To discover the underlying integral type.
- To support dynamic and reflection-based scenarios.

---

### Syntax Rules and Structure

```
Enum.GetValues(typeof(EnumName))   ;
Enum.GetNames(typeof(EnumName))    ;
Enum.Parse(typeof(EnumName),   stringValue)   ;
Enum.TryParse<TEnum>(stringValue,   out   TEnum   result)   ;
Enum.IsDefined(typeof(EnumName),   value)   ;
Enum.GetUnderlyingType(typeof(EnumName))   ;
```

**Component breakdown**

- `typeof(EnumName)` — The enum type.
- `stringValue` — The name or numeric string to parse.
- `out result` — The parsed enum value.
- `value` — The value or name to check.

#### Syntax Rules

- `GetValues` and `GetNames` return arrays in the order of the underlying values.
- `Parse` throws `ArgumentException` or `FormatException` on failure.
- `TryParse` returns `false` on failure without throwing.
- `IsDefined` returns `bool`; it checks whether the value or name is a named member.
- `GetUnderlyingType` returns a `Type` object representing the integral type.

#### Constraints and Limitations

- `Parse` can succeed for numeric strings that do not correspond to a named member; combine with `IsDefined` to validate.
- `IsDefined` performs a linear search and can be slow for large enums.
- `GetValues` and `GetNames` allocate arrays; avoid in hot paths.

---

### 5.1 `Enum.GetValues()`

**Definitions**

- **Core Definition:** Returns an array containing the values of all members of the specified enum.
- **Technical Definition:** `Enum.GetValues(Type enumType)` returns an `Array` of the enum’s underlying values. The generic overload `Enum.GetValues<TEnum>()` returns `TEnum[]`.
- **Beginner-Friendly:** `GetValues` gives you **all the numeric values** in the enum.

**Purposes**

- To iterate over all defined enum values.
- To build lookup tables or mappings.
- To perform validation across all members.

**Syntax Rules and Structure**

```
Array   values   =   Enum.GetValues(typeof(EnumName))   ;
TEnum[]   values2   =   Enum.GetValues<TEnum>()   ;
```

**Constraints:** Returns a new array; elements are in order of increasing value.

**Annotated Example**

```csharp
using System;

enum Season { Spring, Summer, Autumn, Winter }

class Program
{
    static void Main()
    {
        foreach (Season s in Enum.GetValues(typeof(Season)))
            Console.WriteLine($"{(int)s} = {s}");
    }
}
```

**Expected Output**

```
0 = Spring
1 = Summer
2 = Autumn
3 = Winter
```

**Real-World Cases:** Populating dropdowns with all options; validating that a value is among the defined members.

**References**

- Enum.GetValues Method — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.enum.getvalues

---

### 5.2 `Enum.GetNames()`

**Definitions**

- **Core Definition:** Returns an array of the names of all members of the specified enum.
- **Technical Definition:** `Enum.GetNames(Type enumType)` returns a `string[]` containing the names of the enum’s members, in order of their underlying values.
- **Beginner-Friendly:** `GetNames` gives you **all the names** in the enum.

**Purposes**

- To display the names of all options.
- To search for a member by name.
- To build UI lists from enum members.

**Syntax Rules and Structure**

```
string[]   names   =   Enum.GetNames(typeof(EnumName))   ;
```

**Constraints:** Returns a new array; order matches underlying values.

**Annotated Example**

```csharp
using System;

enum Season { Spring, Summer, Autumn, Winter }

class Program
{
    static void Main()
    {
        foreach (string name in Enum.GetNames(typeof(Season)))
            Console.WriteLine(name);
    }
}
```

**Expected Output**

```
Spring
Summer
Autumn
Winter
```

**Real-World Cases:** Displaying enum options in a console or UI.

**References**

- Enum.GetNames Method — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.enum.getnames

---

### 5.3 `Enum.Parse()`

**Definitions**

- **Core Definition:** Converts the string representation of an enum member name or numeric value to an enum instance.
- **Technical Definition:** `Enum.Parse(Type enumType, string value)` returns an `object` that must be cast to the enum type. It accepts either a name (case-sensitive by default) or a numeric string. Throws `ArgumentException` or `FormatException` on failure. The generic overload `Enum.Parse<TEnum>(string)` returns `TEnum` directly.
- **Beginner-Friendly:** `Parse` turns a **string into an enum value**.

**Purposes**

- To convert user input or configuration strings to enums.
- To deserialize enum names from data sources.
- To implement command-line argument parsing.

**Syntax Rules and Structure**

```
EnumName   value   =   (EnumName)Enum.Parse(typeof(EnumName),   "MemberName")   ;
EnumName   value2   =   Enum.Parse<EnumName>("MemberName")   ;
```

**Constraints:** Throws on failure; numeric strings outside named members succeed unless validated with `IsDefined`.

**Annotated Example**

```csharp
using System;

enum LogLevel { Debug, Info, Warning, Error }

class Program
{
    static void Main()
    {
        LogLevel level = (LogLevel)Enum.Parse(typeof(LogLevel), "Warning");
        Console.WriteLine(level); // Warning
    }
}
```

**Expected Output**

```
Warning
```

**Real-World Cases:** Parsing configuration values, command-line arguments, or JSON string fields.

**References**

- Enum.Parse Method — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.enum.parse
- Enum Class — Parsing (Microsoft Learn) - https://learn.microsoft.com/en-us/dotnet/api/system.enum

---

### 5.4 `Enum.TryParse()`

**Definitions**

- **Core Definition:** Attempts to convert a string to an enum value without throwing on failure.
- **Technical Definition:** `Enum.TryParse<TEnum>(string value, out TEnum result)` returns `true` if parsing succeeds and assigns the result; otherwise returns `false`. It is the safe alternative to `Parse`.
- **Beginner-Friendly:** `TryParse` is like `Parse` but **does not crash** if the string is invalid; it returns `false` instead.

**Purposes**

- To safely parse user input without exceptions.
- To validate strings before using them as enums.
- To simplify error handling in parsing logic.

**Syntax Rules and Structure**

```
bool   success   =   Enum.TryParse<TEnum>(stringValue,   out   TEnum   result)   ;
```

**Constraints:** Returns `false` for invalid strings; numeric strings may succeed even if not a named member (validate with `IsDefined`).

**Annotated Example**

```csharp
using System;

enum LogLevel { Debug, Info, Warning, Error }

class Program
{
    static void Main()
    {
        bool ok = Enum.TryParse<LogLevel>("Info", out LogLevel level);
        Console.WriteLine(ok);    // True
        Console.WriteLine(level); // Info

        ok = Enum.TryParse<LogLevel>("Invalid", out _);
        Console.WriteLine(ok);    // False
    }
}
```

**Expected Output**

```
True
Info
False
```

**Real-World Cases:** Safe parsing of user input, configuration files, or network messages.

**References**

- Enum.TryParse Method — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.enum.tryparse

---

### 5.5 `Enum.IsDefined()`

**Definitions**

- **Core Definition:** Determines whether a value or name corresponds to a defined member of the specified enum.
- **Technical Definition:** `Enum.IsDefined(Type enumType, object value)` returns `true` if the value (or name) is a named constant of the enum. It performs a linear search through the enum’s members.
- **Beginner-Friendly:** `IsDefined` answers: “Is this a **real option** in the list?”

**Purposes**

- To validate parsed numeric values that may not correspond to named members.
- To check user-supplied names or values.
- To guard against arbitrary integers cast to enums.

**Syntax Rules and Structure**

```
bool   defined   =   Enum.IsDefined(typeof(EnumName),   valueOrName)   ;
```

**Constraints:** O(n) search; accepts `string` names or numeric values.

**Annotated Example**

```csharp
using System;

enum Season { Spring, Summer, Autumn, Winter }

class Program
{
    static void Main()
    {
        Console.WriteLine(Enum.IsDefined(typeof(Season), "Summer")); // True
        Console.WriteLine(Enum.IsDefined(typeof(Season), 99));       // False
    }
}
```

**Expected Output**

```
True
False
```

**Real-World Cases:** Validating deserialized enum values, defensive programming against undefined enum values.

**References**

- Enum.IsDefined Method — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.enum.isdefined
- Enum Class — IsDefined and parsing validation - https://learn.microsoft.com/en-us/dotnet/api/system.enum

---

### 5.6 `Enum.GetUnderlyingType()`

**Definitions**

- **Core Definition:** Returns the underlying integral type of the specified enum.
- **Technical Definition:** `Enum.GetUnderlyingType(Type enumType)` returns a `Type` object representing the integral type (`Int32`, `Byte`, etc.) used to store the enum’s values.
- **Beginner-Friendly:** `GetUnderlyingType` tells you **what kind of number** backs the enum.

**Purposes**

- To discover the storage type at runtime.
- To perform conversions or serialization based on the underlying type.
- To validate interop expectations.

**Syntax Rules and Structure**

```
Type   underlying   =   Enum.GetUnderlyingType(typeof(EnumName))   ;
```

**Constraints:** Returns `System.Int32` by default; reflects the explicit underlying type if specified.

**Annotated Example**

```csharp
using System;

enum Small : byte { A = 1 }

class Program
{
    static void Main()
    {
        Console.WriteLine(Enum.GetUnderlyingType(typeof(Small))); // System.Byte
    }
}
```

**Expected Output**

```
System.Byte
```

**Real-World Cases:** Reflection-based serialization, interop code that needs to match native integer sizes.

**References**

- Enum.GetUnderlyingType Method — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.enum.getunderlyingtype
- Type.GetEnumUnderlyingType Method — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.type.getenumunderlyingtype

---

## 6. `[Flags]` Attribute

### Definitions

**Core Definition**
The `[Flags]` attribute is an attribute applied to an enum to indicate that its members can be meaningfully combined using bitwise operations.

**Technical Definition**
`System.FlagsAttribute` is an attribute that changes the behaviour of an enum when combined values are converted to strings and when tools (such as `Console` and the expression evaluator) display the enum. It signals that the enum represents a bit field, where individual members should be powers of two so they can be combined with bitwise OR (`|`) and tested with bitwise AND (`&`). The attribute does not change the runtime representation of the enum; it affects formatting and analysis behaviour.

**Beginner-Friendly Explanation**
`[Flags]` tells C# that an enum is not just a single choice but a **combination of choices**. It allows the enum to display combined values as a list of names (e.g., `Read, Write`) instead of a single number.

---

### Purposes

- To mark an enum as a bit field whose members can be combined.
- To enable readable `ToString()` output for combined values.
- To satisfy code analysis rule CA1027 for enums whose values are powers of two.
- To communicate intent to other developers and tools.
- To support bitwise operations on enum values.

---

### Syntax Rules and Structure

```
[Flags]
[accessModifiers]   enum   EnumName
{
    None   =   0,
    Member1   =   1,      // 2^0
    Member2   =   2,      // 2^1
    Member3   =   4,      // 2^2
    ...
}
```

**Component breakdown**

- `[Flags]` — Attribute indicating combinable members.
- `None = 0` — Recommended zero value representing “no flags set.”
- Members — Powers of two (1, 2, 4, 8, ...).

#### Syntax Rules

- Members should be powers of two so they can be combined unambiguously.
- A `None = 0` member is recommended to represent the empty combination.
- Combined values are formed with the bitwise OR operator (`|`).
- Individual flags are tested with `HasFlag` or bitwise AND.

#### Constraints and Limitations

- The attribute does not enforce powers of two; CA1027 flags violations.
- Without `[Flags]`, `ToString()` on a combined value returns the numeric value rather than a list of names.
- `HasFlag` boxes the argument and is relatively slow; bitwise AND is faster in hot paths.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Flags Enum with `ToString()`

```csharp
using System;

[Flags]
enum Permissions
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
        Console.WriteLine((int)p);             // 3
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

- `Read` (1) and `Write` (2) combine to `3`.
- `[Flags]` makes `ToString()` display the combined names.
- `HasFlag` checks whether the `Read` bit is set.

---

#### Example 2: CA1027 Violation (Without `[Flags]`)

```csharp
using System;

enum Permissions   // CA1027: should have [Flags]
{
    Read = 1,
    Write = 2,
    Execute = 4
}

class Program
{
    static void Main()
    {
        Permissions p = (Permissions)3;
        Console.WriteLine(p);   // 3 (not "Read, Write")
    }
}
```

**Expected Output**

```
3
```

**Why this output occurs**

- Without `[Flags]`, `ToString()` returns the numeric value instead of combining names.
- Code analysis would flag CA1027 because the values are powers of two but the attribute is missing.

---

### Real-World Cases

- **File access permissions** — `Read`, `Write`, `Execute`.
- **Feature toggles** — Combining multiple enabled features.
- **Days of the week availability** — A resource available on `Monday | Wednesday | Friday`.

---

### References

- CA1027: Mark enums with FlagsAttribute — .NET Code Analysis - https://learn.microsoft.com/en-us/dotnet/fundamentals/code-analysis/quality-rules/ca1027
- FlagsAttribute Class — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.flagsattribute
- C# Enumerations — Bit flags - https://learn.microsoft.com/en-us/dotnet/csharp/fundamentals/types/enums#bit-flags

---

## 7. Flag Enums

### Definitions

**Core Definition**
A flag enum is an enum whose members represent individual bits that can be combined to represent multiple simultaneous choices.

**Technical Definition**
A flag enum is an enum annotated with `[Flags]` whose members are assigned powers of two (1, 2, 4, 8, ...). Combinations are formed with bitwise OR (`|`), individual flags are tested with bitwise AND (`&`) or `HasFlag`, and combined values are typically displayed as comma-separated names. The zero value (`None`) represents the empty combination. Flag enums are intended to be combined and examined with bitwise operations.

**Beginner-Friendly Explanation**
A flag enum is a **set of on/off switches packed into one number**. Each switch is a bit; you can turn multiple switches on at once and check which ones are on.

---

### Purposes

- To represent combinations of independent choices in a single value.
- To save storage compared to storing separate boolean fields.
- To enable bitwise set operations (union, intersection, complement).
- To provide readable `ToString()` output for combinations.
- To model permissions, features, and options naturally.

---

### Syntax Rules and Structure

```
[Flags]
enum   EnumName
{
    None   =   0,
    A      =   1 << 0,   // 1
    B      =   1 << 1,   // 2
    C      =   1 << 2,   // 4
    ...
}
```

**Component breakdown**

- `None = 0` — Represents no flags set.
- `1 << n` — Power-of-two expression (common idiom).
- `|` — Combines flags.
- `&` — Tests flags.

#### Syntax Rules

- Each member should be a distinct power of two.
- `None = 0` is conventional and recommended.
- Combined values are formed with `|`.
- Individual flags are tested with `&` or `HasFlag`.
- `ToString()` on a combined value returns a comma-separated list when `[Flags]` is present.

#### Constraints and Limitations

- The compiler does not enforce powers of two; CA1027 analyzes this.
- `HasFlag` boxes its argument and is slower than bitwise AND.
- Undefined bits can be set by casting arbitrary integers; validate with masking if needed.
- The number of distinct flags is limited by the underlying type (32 for `int`, 64 for `long`).

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Combining and Testing Flags

```csharp
using System;

[Flags]
enum FileAccess
{
    None = 0,
    Read = 1,
    Write = 2,
    Execute = 4
}

class Program
{
    static void Main()
    {
        FileAccess access = FileAccess.Read | FileAccess.Write;
        Console.WriteLine(access);                        // Read, Write
        Console.WriteLine((access & FileAccess.Read) != 0); // True (bitwise AND)
        Console.WriteLine(access.HasFlag(FileAccess.Execute)); // False
    }
}
```

**Expected Output**

```
Read, Write
True
False
```

**Why this output occurs**

- `Read | Write` sets both bits.
- Bitwise AND tests whether the `Read` bit is set.
- `HasFlag` returns `false` because `Execute` is not set.

---

#### Example 2: Using `None` and `All`

```csharp
using System;

[Flags]
enum Permissions
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
        Console.WriteLine(Permissions.None);   // None
        Console.WriteLine(Permissions.All);    // Read, Write, Execute
        Console.WriteLine((int)Permissions.All); // 7
    }
}
```

**Expected Output**

```
None
Read, Write, Execute
7
```

**Why this output occurs**

- `None` is `0`, displayed as `None` (or `0` on some runtimes).
- `All` combines all three flags.
- The numeric value is `1 + 2 + 4 = 7`.

---

### Real-World Cases

- **File system permissions** — `Read`, `Write`, `Delete`.
- **Feature flags** — `DarkMode`, `Notifications`, `AutoSave`.
- **Scheduling** — `Monday | Wednesday | Friday`.
- **Network protocols** — TCP flags such as `SYN | ACK`.

---

### References

- C# Enumerations — Bit flags - https://learn.microsoft.com/en-us/dotnet/csharp/fundamentals/types/enums#bit-flags
- Enumeration Design — Flags enumerations - https://learn.microsoft.com/en-us/previous-versions/dotnet/netframework-2.0/ms229058(v=vs.80)
- CA1027: Mark enums with FlagsAttribute - https://learn.microsoft.com/en-us/dotnet/fundamentals/code-analysis/quality-rules/ca1027

---

## 8. Enum Best Practices

### Definitions

**Core Definition**
Enum best practices are the design guidelines that ensure enums are used effectively, safely, and consistently across a codebase.

**Technical Definition**
Best practices encompass using enums for strongly typed sets of values, providing a zero-valued member on simple enums, naming flag enums with plural nouns, avoiding open sets and sentinel values, choosing the default `Int32` underlying type unless specific reasons exist, and validating parsed values with `IsDefined`. These guidelines are documented in Microsoft’s enumeration design recommendations and code analysis rules.

**Beginner-Friendly Explanation**
Best practices are the habits that make enums reliable: always include a `None = 0` option, use `[Flags]` only when values should combine, do not use enums for things that change often, and use `TryParse` with `IsDefined` when converting strings.

---

### Purposes

- To ensure enums represent closed, well-defined sets of options.
- To provide a valid default zero value for uninitialized enums.
- To prevent versioning and compatibility issues.
- To make flag enums intuitive and correct.
- To validate external input safely.

---

### Guidelines and Structure

#### General Guidelines

```
// Do provide a zero value on simple enums
enum LogLevel { None = 0, Debug, Info, Warning, Error }

// Do use [Flags] only for combinable values
[Flags]
enum Permissions { None = 0, Read = 1, Write = 2 }

// Do validate parsed values
if (Enum.TryParse<LogLevel>(input, out var level) &&
    Enum.IsDefined(typeof(LogLevel), level))
{
    // use level
}

// Do use Int32 as the default underlying type
enum Status { Active, Inactive }

// Do name flag enums with plural nouns
[Flags] enum FileAccesses { Read, Write, Execute }
```

**Component breakdown**

- **Zero value** — Ensures the default `(E)0` is valid and meaningful; name it `None` when appropriate.
- **`[Flags]` only when combinable** — Misapplying the attribute causes confusing `ToString()` output and violates CA1027.
- **Validate parsed values** — `Parse` can produce undefined numeric values; `IsDefined` guards against them.
- **Default `Int32`** — Use `int` unless memory savings, interop, or more than 32 flags justify another type.
- **Plural naming for flags** — Flag enums represent combinations, so plural nouns are conventional.

#### Rules and Constraints

- **Do not use enums for open sets** (e.g., OS versions) because adding values can break existing code.
- **Do not define reserved values for future use**; add them when needed.
- **Avoid publicly exposing single-value enums**; they add no value.
- **Do not include sentinel values** like `LastValue` in the enum.
- **Prefer enums over static constants** for sets of values.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Zero Value and Safe Parsing

```csharp
using System;

enum LogLevel
{
    None = 0,
    Debug = 1,
    Info = 2,
    Warning = 3,
    Error = 4
}

class Program
{
    static void Main()
    {
        LogLevel defaultLevel = default;          // None (0)
        Console.WriteLine(defaultLevel);          // None

        string input = "2";
        if (Enum.TryParse<LogLevel>(input, out var level) &&
            Enum.IsDefined(typeof(LogLevel), level))
        {
            Console.WriteLine(level);             // Info
        }
    }
}
```

**Expected Output**

```
None
Info
```

**Why this output occurs**

- The zero value `None` is the default for `LogLevel`.
- `TryParse` succeeds, and `IsDefined` confirms `2` is a named member (`Info`).

---

#### Example 2: Correct vs. Incorrect `[Flags]` Usage

```csharp
using System;

[Flags]
enum CorrectFlags { None = 0, A = 1, B = 2, C = 4 }

// Incorrect: values are not powers of two
[Flags]
enum IncorrectFlags { A = 1, B = 2, C = 3 }   // CA1027 violation

class Program
{
    static void Main()
    {
        Console.WriteLine(CorrectFlags.A | CorrectFlags.B); // A, B
        Console.WriteLine(IncorrectFlags.A | IncorrectFlags.B); // 3
    }
}
```

**Expected Output**

```
A, B
3
```

**Why this output occurs**

- `CorrectFlags` follows the power-of-two convention, producing readable output.
- `IncorrectFlags` violates the convention; combined values are not decomposable into distinct flags, so `ToString()` returns the numeric value.

---

### Real-World Cases

- **API design** — Providing `None = 0` on enums returned by public methods.
- **Configuration** — Using `TryParse` + `IsDefined` for user-supplied settings.
- **Feature flags** — Using `[Flags]` with powers of two for combinable features.

---

### References

- Enumeration Design — Microsoft Learn - https://learn.microsoft.com/en-us/previous-versions/dotnet/netframework-2.0/ms229058(v=vs.80)
- CA1008: Enums should have zero value — .NET Code Analysis - https://learn.microsoft.com/en-us/dotnet/fundamentals/code-analysis/quality-rules/ca1008
- CA1027: Mark enums with FlagsAttribute — .NET Code Analysis - https://learn.microsoft.com/en-us/dotnet/fundamentals/code-analysis/quality-rules/ca1027
- C# Enumerations — Microsoft Learn - https://learn.microsoft.com/en-us/dotnet/csharp/fundamentals/types/enums

---

## Deprecated, Unsafe, Implementation-Dependent, and Version-Specific Notes

| Feature | Status / Note |
|---------|---------------|
| `char` as underlying type | **Not allowed** by the C# specification. |
| Enum values as compile-time constants | References are substituted as numeric literals; changing values can break binary compatibility. |
| `Enum.HasFlag` | Works but **boxes** its argument; slower than bitwise AND in hot paths. |
| `Enum.Parse` with numeric strings | Can produce **undefined enum values**; always validate with `IsDefined`. |
| Undefined enum values | A variable can hold any value in the underlying type’s range, even if not a named member. |
| `[Flags]` without powers of two | **CA1027 violation**; combined values may not decompose correctly. |
| Enumeration classes | For rich behaviour, Microsoft suggests considering **enumeration classes** instead of enums in some scenarios. |

---

## References (Consolidated)

- C# Enumerations — Microsoft Learn - https://learn.microsoft.com/en-us/dotnet/csharp/fundamentals/types/enums
- enum keyword — C# Reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/enum
- Enums — C# Language Specification - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/language-specification/enums
- Enum Class (System) — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.enum
- Enum.GetValues Method — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.enum.getvalues
- Enum.GetNames Method — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.enum.getnames
- Enum.Parse Method — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.enum.parse
- Enum.TryParse Method — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.enum.tryparse
- Enum.IsDefined Method — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.enum.isdefined
- Enum.GetUnderlyingType Method — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.enum.getunderlyingtype
- Type.GetEnumUnderlyingType Method — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.type.getenumunderlyingtype
- FlagsAttribute Class — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.flagsattribute
- CA1027: Mark enums with FlagsAttribute — .NET Code Analysis - https://learn.microsoft.com/en-us/dotnet/fundamentals/code-analysis/quality-rules/ca1027
- CA1008: Enums should have zero value — .NET Code Analysis - https://learn.microsoft.com/en-us/dotnet/fundamentals/code-analysis/quality-rules/ca1008
- Enumeration Design — Microsoft Learn (previous versions) - https://learn.microsoft.com/en-us/previous-versions/dotnet/netframework-2.0/ms229058(v=vs.80)
- Enumeration types — C# Programming Guide - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/enumeration-types

---