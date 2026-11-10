# C# Enums — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
An **enum** (enumeration type) is a distinct **value type** that defines a set of named constants backed by an underlying integral type. Enums are used when a value must be one of a fixed, closed set of options—such as days of the week, HTTP status codes, log levels, or directions. They make code more readable, self-documenting, and resistant to invalid values compared to raw integer or string constants.

**Technical Definition**
An enumeration is a set of named constants whose underlying type is any integral type. If no underlying type is explicitly declared, `System.Int32` is used. The C# `enum` keyword declares a distinct type consisting of a set of named constants called the **enumerator list**. By default, the first enumerator has the value `0`, and each successive enumerator increases by `1`; initializers can override these values. Every enumeration type has an **underlying type**, which can be any integral type except `char` (`byte`, `sbyte`, `short`, `ushort`, `int`, `uint`, `long`, `ulong`). The default value of an enum `E` is the value produced by the expression `(E)0`. Enums inherit implicitly from `System.Enum` (which inherits from `System.ValueType`), but the inheritance relationship is handled implicitly by the compiler.

**Beginner-Friendly Explanation**
An enum is a **named list of options**. Instead of remembering that Monday is `1` and Tuesday is `2`, you write `DayOfWeek.Monday` and `DayOfWeek.Tuesday`. The compiler ensures you only use the names that exist in the list, which makes your code safer and easier to read. Enums are ideal when a value should be one of a small, known set of choices—and they are especially powerful when the `[Flags]` attribute lets you combine multiple options into a single value using bitwise operations.

---

### Key Characteristics

- **Distinct value type** — Enums are value types that implicitly derive from `System.Enum` / `System.ValueType`.
- **Named constants** — Each member is a named constant with an underlying integral value.
- **Closed set** — Enums are intended for small, closed sets of choices; open sets (e.g., OS versions) should use other types.
- **Default values** — The first member defaults to `0`; each subsequent member increments by `1` unless explicitly assigned.
- **Underlying type** — Defaults to `int`; can be any integral type except `char`.
- **Bit flag support** — The `[Flags]` attribute enables members to be combined using bitwise OR (`|`) and tested using bitwise AND (`&`).
- **Compile-time substitution** — All references to enum values are converted to numeric literals at compile time, which can create versioning considerations.

---

### Prerequisites

- Basic understanding of C# program structure (`class`, `Main`, namespaces).
- Familiarity with value types and integral types (`int`, `byte`, etc.).
- Knowledge of `switch` statements and pattern matching (for using enums).
- Awareness of bitwise operators (`|`, `&`, `^`) for flag enums.
- Understanding of the `System` namespace and `Console` class.

---

### Related Programming Areas

- **Type safety** — Enums strongly type parameters, properties, and return values.
- **Bit manipulation** — Flag enums use bitwise operations to combine and test values.
- **Serialization** — Enums are commonly serialized as names or numeric values in JSON/XML.
- **Switch expressions** — Enums integrate naturally with `switch` expressions and pattern matching.
- **API design** — Enums provide self-documenting, discoverable constants for public APIs.

---

### Core Concepts / Features

1. Enums (Overarching Concept)
2. Enum Declaration
3. Enum Values (and Underlying Type)
4. Flag Enums
5. `[Flags]` Attribute
6. Enum Methods
7. Enum Best Practices

Each concept follows a uniform structure: definitions, purposes, syntax rules and structure, annotated code examples with expected outputs, real-world cases, and references.

---

## 1. Enums (Overarching Concept)

### Definitions

**Core Definition**
An enum is a value type that defines a set of named integral constants, providing a type-safe way to work with fixed sets of options.

**Technical Definition**
`System.Enum` is the abstract base class for all enumeration types. Enum types are distinct value types that declare a set of named constants. Each enum member has an associated constant value of the underlying integral type. Enums support conversions to and from their underlying types, parsing from strings, and enumeration of their members via reflection-based methods. Enums can be **simple** (representing a single choice from a closed set) or **flag** enums (representing combinations of choices via bitwise operations).

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
[accessModifiers] enum EnumName [: underlyingType]
{
    Member1 [= value1],
    Member2 [= value2],
    ...
}
```

**Component breakdown**

- `enum` — Keyword introducing an enumeration type.
- `EnumName` — The type name (PascalCase by convention; singular for simple enums, plural for flag enums).
- `: underlyingType` — Optional; defaults to `int`. Must be an integral type except `char`.
- `Member` — Named constant.
- `= value` — Optional explicit value; subsequent members increment by `1`.

#### Syntax Rules

- Enum members must be unique within the enum (though duplicate values are allowed but discouraged).
- The first member defaults to `0`; each subsequent member increments by `1`.
- Values must be within the range of the underlying type.
- Enums can be declared at namespace level or nested within a class or struct.

#### Constraints and Limitations

- Enums cannot inherit from other types (except implicitly from `System.ValueType`).
- A variable of an enum type can hold any value in the range of the underlying type, even if it is not a named member.
- The underlying type cannot be `char`.
- All references to enum values are converted to numeric literals at compile time, creating potential versioning issues.

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

- System.Enum class — .NET - https://learn.microsoft.com/en-us/dotnet/api/system.enum
- enum (C# Reference) - https://learn.microsoft.com/en-us/previous-versions/visualstudio/visual-studio-2010/sbbt4032(v=vs.100)

---

## 2. Enum Declaration

### Definitions

**Core Definition**
Enum declaration is the syntax that introduces a new enumeration type, specifying its name, accessibility, underlying type, and members.

**Technical Definition**
An enum declaration begins with the `enum` keyword and defines the name, accessibility, underlying type, and members of the enum. The declaration may explicitly declare an underlying type from the set `byte`, `sbyte`, `short`, `ushort`, `int`, `uint`, `long`, or `ulong`; `char` is disallowed. If no underlying type is specified, `int` is used. Enums are typically defined directly within a namespace so all classes in the namespace can access them with equal convenience, though they can also be nested within a class or struct.

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
[attributes] [modifiers] enum EnumName [: integralType]
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
- Enum members cannot contain whitespace in their names.

#### Constraints and Limitations

- `char` cannot be used as an underlying type.
- Enums cannot have a parameterless constructor explicitly.
- Enums cannot inherit from other types.

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
        Console.WriteLine(Enum.GetUnderlyingType(typeof(StatusCode))); // System.Byte
    }
}
```

**Expected Output**

```
404
System.Byte
```

**Why this output occurs**

- The underlying type is explicitly `byte`, and explicit values are assigned.
- `GetUnderlyingType` confirms the storage type.

---

### Real-World Cases

- **Protocol identifiers** — Using explicit values for wire compatibility.
- **Flags** — Declaring members as powers of two for bitwise combinations.
- **Domain models** — Declaring order statuses, user roles, or categories.

---

### References

- enum (C# Reference) — Declaration syntax - https://learn.microsoft.com/en-us/previous-versions/visualstudio/visual-studio-2010/sbbt4032(v=vs.100)
- System.Enum class — Create an enumeration type - https://learn.microsoft.com/en-us/dotnet/api/system.enum

---

## 3. Enum Values (and Underlying Type)

### Definitions

**Core Definition**
Enum values are the named constants and their associated integral values. The **underlying type** is the integral type that stores each member’s value.

**Technical Definition**
Each enum member is a named constant whose value is of the enum’s underlying type. By default, the first member has the value `0`, and each successive member is increased by `1`; explicit initializers can override these values. The underlying type determines the range of valid member values and the storage size. A variable of an enum type can be assigned any value in the range of the underlying type, not only the named constants. The default value of an enum `E` is `(E)0`.

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
enum EnumName
{
    Member1 = constantExpression1,
    Member2,                    // Member1 + 1
    Member3 = constantExpression3
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

#### Example 2: Undefined Values and Underlying Type

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
        Console.WriteLine(Enum.GetUnderlyingType(typeof(Season))); // System.Int32
    }
}
```

**Expected Output**

```
99
False
System.Int32
```

**Why this output occurs**

- Casting an arbitrary integer produces an enum value outside the named members.
- `IsDefined` checks whether the value corresponds to a named constant.
- The default underlying type is `Int32`.

---

### Real-World Cases

- **HTTP status codes** — Explicit values like `200`, `404`, `500`.
- **File access flags** — Powers of two for `Read`, `Write`, `Execute`.
- **Protocol versions** — Numeric values that must match external specifications.

---

### References

- enum (C# Reference) — Values and underlying type - https://learn.microsoft.com/en-us/previous-versions/visualstudio/visual-studio-2010/sbbt4032(v=vs.100)
- Type.GetEnumUnderlyingType Method — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.type.getenumunderlyingtype

---

## 4. Flag Enums

### Definitions

**Core Definition**
A **flag enum** is an enum whose members represent individual bits that can be combined to represent multiple simultaneous choices.

**Technical Definition**
Flag enums are designed to support bitwise operations on the enum values. A common example of a flags enum is a list of options. A flag enum is annotated with `[Flags]` and assigns each member a **power of two** so that members can be combined unambiguously with bitwise OR (`|`). Combinations are tested with bitwise AND (`&`). For a bit field to store and retrieve combined values correctly, each defined value must be a power of two (1, 2, 4, 8, …). The zero value member (`None`) represents the empty combination.

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
enum EnumName
{
    None   = 0,
    A      = 1 << 0,   // 1
    B      = 1 << 1,   // 2
    C      = 1 << 2,   // 4
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
- Individual flags are tested with `&`.

#### Constraints and Limitations

- The compiler does not enforce powers of two; CA1027 analyzes this.
- The number of distinct flags is limited by the underlying type (32 for `int`, 64 for `long`).
- Negative flag values should be used with caution because many bits may be set.

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
        Console.WriteLine((access & FileAccess.Read) != 0); // True
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

- `None` is `0`, displayed as `None`.
- `All` combines all three flags.
- The numeric value is `1 + 2 + 4 = 7`.

---

### Real-World Cases

- **File system permissions** — `Read`, `Write`, `Delete`.
- **Feature flags** — `DarkMode`, `Notifications`, `AutoSave`.
- **Scheduling** — `Monday | Wednesday | Friday`.

---

### References

- System.FlagsAttribute class — .NET - https://learn.microsoft.com/en-us/dotnet/api/system.flagsattribute
- CA1027: Mark enums with FlagsAttribute - https://learn.microsoft.com/en-us/dotnet/fundamentals/code-analysis/quality-rules/ca1027

---

## 5. `[Flags]` Attribute

### Definitions

**Core Definition**
The `[Flags]` attribute (`System.FlagsAttribute`) indicates that an enum can be treated as a **bit field**—a set of flags that can be combined using bitwise operations.

**Technical Definition**
The `FlagsAttribute` indicates that an enumeration can be treated as a bit field; that is, a set of flags. When formatting a flag enum, `ToString()` produces a comma-separated list of the combined member names for values that are bitwise OR combinations of multiple fields. The attribute should be applied only when bitwise operations (AND, OR, XOR) are to be performed on the numeric value. For correct behaviour, each defined value must be a power of two.

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
[accessModifiers] enum EnumName
{
    None   = 0,
    Member1 = 1,
    Member2 = 2,
    Member3 = 4,
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
- Individual flags are tested with bitwise AND (`&`).

#### Constraints and Limitations

- The attribute does not enforce powers of two; CA1027 flags violations.
- Without `[Flags]`, `ToString()` on a combined value returns the numeric value rather than a list of names.

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

- System.FlagsAttribute class — .NET - https://learn.microsoft.com/en-us/dotnet/api/system.flagsattribute
- CA1027: Mark enums with FlagsAttribute — .NET code analysis - https://learn.microsoft.com/en-us/dotnet/fundamentals/code-analysis/quality-rules/ca1027

---

## 6. Enum Methods

### Definitions

**Core Definition**
Enum methods are the static methods on `System.Enum` that provide reflection-based access to enum metadata and conversion between strings, values, and enum instances.

**Technical Definition**
`System.Enum` provides methods for comparing instances, converting values to and from their string representations, and creating instances from values. The key static methods include `GetValues` (returns an array of all member values), `GetNames` (returns an array of all member names), `Parse` (converts a string to an enum value, throwing on failure), `TryParse` (safe parsing without exceptions), `IsDefined` (checks whether a value or name is a defined member), and `GetUnderlyingType` (returns the integral type). Many of these methods operate on `Type` objects; generic overloads are available in modern .NET.

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
Enum.GetValues(typeof(EnumName));
Enum.GetNames(typeof(EnumName));
Enum.Parse(typeof(EnumName), stringValue);
Enum.TryParse<TEnum>(stringValue, out TEnum result);
Enum.IsDefined(typeof(EnumName), value);
Enum.GetUnderlyingType(typeof(EnumName));
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

### Annotated Code Examples and Expected Outputs

#### Example 1: `GetValues` and `GetNames`

```csharp
using System;

enum Season { Spring, Summer, Autumn, Winter }

class Program
{
    static void Main()
    {
        foreach (Season s in Enum.GetValues(typeof(Season)))
            Console.WriteLine($"{(int)s} = {s}");

        foreach (string name in Enum.GetNames(typeof(Season)))
            Console.WriteLine(name);
    }
}
```

**Expected Output**

```
0 = Spring
1 = Summer
2 = Autumn
3 = Winter
Spring
Summer
Autumn
Winter
```

**Why this output occurs**

- `GetValues` returns all member values in order.
- `GetNames` returns all member names in the same order.

---

#### Example 2: `TryParse` and `IsDefined`

```csharp
using System;

enum LogLevel { Debug, Info, Warning, Error }

class Program
{
    static void Main()
    {
        if (Enum.TryParse<LogLevel>("Warning", out LogLevel level) &&
            Enum.IsDefined(typeof(LogLevel), level))
        {
            Console.WriteLine(level);   // Warning
        }

        if (!Enum.TryParse<LogLevel>("Invalid", out _))
            Console.WriteLine("Parse failed");   // Parse failed
    }
}
```

**Expected Output**

```
Warning
Parse failed
```

**Why this output occurs**

- `TryParse` succeeds for the valid name `"Warning"` and `IsDefined` confirms it.
- `TryParse` returns `false` for `"Invalid"` without throwing.

---

### Real-World Cases

- **Populating dropdowns** — `GetValues` or `GetNames` to list options.
- **Parsing user input** — `TryParse` + `IsDefined` for safe conversion.
- **Reflection-based serialization** — Discovering enum metadata at runtime.

---

### References

- Enum Methods — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.enum
- System.Enum class — Methods overview - https://learn.microsoft.com/en-us/dotnet/api/system.enum

---

## 7. Enum Best Practices

### Definitions

**Core Definition**
Enum best practices are the design guidelines that ensure enums are used **appropriately, safely, and consistently** across a codebase.

**Technical Definition**
Best practices for enums derive from the Framework Design Guidelines and code analysis rules. Key recommendations include: **use enums to strongly type parameters, properties, and return values that represent sets of values**; **prefer enums over static constants**; **do not use enums for open sets**; **do not provide reserved enum values for future use**; **avoid sentinel values**; **provide a zero value on simple enums** (typically named `None`); **prefer `Int32` as the underlying type unless specific reasons exist**; **name simple enums with singular nouns and flag enums with plural nouns**; and **mark flag enums with `[Flags]`**.

**Beginner-Friendly Explanation**
Best practices make enums reliable: always include a `None = 0` option, use `[Flags]` only when values should combine, use singular names for simple enums and plural names for flag enums, and do not use enums for things that change often.

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

// Do not use enums for open sets
// Do not include sentinel values
// Do not provide reserved values for future use
```

**Component breakdown**

- **Zero value** — Ensures the default `(E)0` is valid and meaningful; name it `None` when appropriate.
- **`[Flags]` only when combinable** — Misapplying the attribute causes confusing `ToString()` output and violates CA1027.
- **Validate parsed values** — `Parse` can produce undefined numeric values; `IsDefined` guards against them.
- **Default `Int32`** — Use `int` unless memory savings, interop, or more than 32 flags justify another type.
- **Plural naming for flags** — Flag enums represent combinations, so plural nouns are conventional.
- **Singular naming for simple enums** — Simple enums represent a single choice.

#### Rules and Constraints

- **Do** use enums for closed sets of values.
- **Do** provide a zero value on simple enums.
- **Do** use `[Flags]` for combinable values with powers of two.
- **Do** validate parsed values with `IsDefined`.
- **Do** prefer `Int32` as the underlying type unless specific needs exist.
- **Do not** use enums for open sets (e.g., OS versions).
- **Do not** provide reserved values for future use.
- **Do not** include sentinel values in enums.

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

- Enum Design — Framework Design Guidelines - https://learn.microsoft.com/en-us/dotnet/standard/design-guidelines/enum
- CA1027: Mark enums with FlagsAttribute — .NET code analysis - https://learn.microsoft.com/en-us/dotnet/fundamentals/code-analysis/quality-rules/ca1027
- CA1717: Only FlagsAttribute enums should have plural names — .NET code analysis - https://learn.microsoft.com/en-us/dotnet/fundamentals/code-analysis/quality-rules/ca1717
- System.FlagsAttribute class — Guidelines for FlagsAttribute and enum - https://learn.microsoft.com/en-us/dotnet/api/system.flagsattribute

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
| Sentinel values | **Discouraged** by Framework Design Guidelines; confusing to users. |
| Reserved values for future use | **Discouraged**; add values later instead. |

---

## References (Consolidated)

- System.Enum class — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.enum
- enum (C# Reference) - https://learn.microsoft.com/en-us/previous-versions/visualstudio/visual-studio-2010/sbbt4032(v=vs.100)
- Enum Design — Framework Design Guidelines - https://learn.microsoft.com/en-us/dotnet/standard/design-guidelines/enum
- System.FlagsAttribute class — .NET - https://learn.microsoft.com/en-us/dotnet/api/system.flagsattribute
- CA1027: Mark enums with FlagsAttribute — .NET code analysis - https://learn.microsoft.com/en-us/dotnet/fundamentals/code-analysis/quality-rules/ca1027
- CA1717: Only FlagsAttribute enums should have plural names — .NET code analysis - https://learn.microsoft.com/en-us/dotnet/fundamentals/code-analysis/quality-rules/ca1717
- Type.GetEnumUnderlyingType Method — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.type.getenumunderlyingtype
- Enum Methods — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.enum

---