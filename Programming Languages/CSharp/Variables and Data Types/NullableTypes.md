# C# Nullable Types — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
A nullable type in C# is a type that can represent either a valid value of an underlying type or the special value `null`, meaning “no value.” C# provides two distinct nullable mechanisms: **nullable value types** (since C# 2.0), which wrap a value type such as `int` or `DateTime` so it can also hold `null`, and **nullable reference types** (since C# 8.0), a compile-time annotation feature that lets you declare which reference variables are intended to hold `null` and which are not.

**Technical Definition**
Nullable value types are instances of the generic struct `System.Nullable<T>`, written in C# as `T?`, where `T` is a non-nullable value type. A `Nullable<T>` has two read-only properties: `HasValue` (a `bool` indicating whether a value is present) and `Value` (the underlying value, which throws `InvalidOperationException` if `HasValue` is `false`). Nullable reference types are not a separate runtime type; they are a compile-time annotation system. In a nullable-enabled context, `string` means “non-nullable reference,” and `string?` means “nullable reference.” The compiler performs **null-state analysis** to track whether expressions are *not-null* or *maybe-null* at each point, and it produces warnings when code may dereference a maybe-null value or assign a maybe-null value to a non-nullable variable.

**Beginner-Friendly Explanation**
Think of a nullable type as a **box that is allowed to be empty**. A regular `int` box must always contain a number; a nullable `int?` box can contain a number or be empty (`null`). For reference types, nullable reference types are like **labels on boxes** that say “this box may be empty” (`string?`) or “this box must never be empty” (`string`). The compiler reads the labels and warns you if you try to use an empty box as if it had something inside.

---

### Key Characteristics

- **Two separate features sharing one name** — Nullable value types (C# 2.0) and nullable reference types (C# 8.0) solve related but distinct problems.
- **Runtime vs. compile-time** — `Nullable<T>` is a real runtime struct; nullable reference types are purely compile-time annotations that do not change runtime behaviour.
- **Safe access operators** — `?.`, `?[]`, `??`, and `??=` provide concise, null-safe expressions.
- **Null-state analysis** — The compiler tracks whether expressions are *not-null* or *maybe-null* and issues warnings (CS8602, CS8600, CS8603, etc.).
- **Optional feature** — Nullable reference type checking is off by default and must be enabled via `<Nullable>enable</Nullable>` or `#nullable enable`.
- **Null-forgiving operator** — The postfix `!` suppresses nullable warnings when you know a value is not null.

---

### Prerequisites

- Basic understanding of C# program structure (`class`, `Main`, namespaces).
- Familiarity with value types and reference types.
- Knowledge of `System.Nullable<T>` and generic types.
- Awareness of compiler warnings and project settings (`.csproj`).
- Basic understanding of null and `NullReferenceException`.

---

### Related Programming Areas

- **Null safety** — Preventing `NullReferenceException`, the “billion-dollar mistake.”
- **API design** — Expressing whether parameters and return values may be null.
- **Database ORM** — Mapping nullable database columns to `T?` properties.
- **Configuration and optional data** — Modelling settings that may be absent.
- **Static analysis** — Compiler warnings and nullable attributes for API contracts.

---

### Core Concepts / Features

1. Nullable value types
2. `int?`
3. `Nullable<T>`
4. `HasValue`
5. `Value`
6. `GetValueOrDefault()`
7. Nullable reference types (C# 8)
8. Nullable context
9. `#nullable enable`
10. Null-forgiving operator (`!`)
11. Null-coalescing operator (`??`)
12. Null-coalescing assignment (`??=`)
13. Null-conditional operator (`?.` and `?[]`)
14. Nullable best practices

Each concept follows the same uniform structure: definitions, purposes, syntax rules and structure, annotated code examples with expected outputs, real-world cases, and references.

---

## 1. Nullable Value Types

### Definitions

**Core Definition**
A nullable value type is a value type that can also represent `null`, meaning “no value.”

**Technical Definition**
A nullable value type `T?` is an instance of `System.Nullable<T>`, where `T` is a non-nullable value type (e.g., `int`, `bool`, `DateTime`, or a custom struct). The range of possible values of a nullable type is the range of the underlying type plus `null`. `Nullable<T>` is a lightweight immutable struct with two read-only properties: `HasValue` and `Value`, and two methods: `GetValueOrDefault()` and `ToString()`.

**Beginner-Friendly Explanation**
Normally, a value type like `int` must always have a value—you cannot set it to “nothing.” A nullable value type lets you say “this `int` might have a number, or it might be empty.” This is useful for optional data, such as a database column that allows `NULL`.

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

- `T?` — Shorthand for `Nullable<T>`; `T` must be a non-nullable value type.
- `Nullable<T>` — The full generic form; interchangeable with `T?`.
- `identifier` — The variable name.
- `= value` — Implicit conversion from `T` to `T?`.
- `= null` — Assigns the null (no-value) state.

#### Syntax Rules

- `T` must be a non-nullable value type; `int??` is invalid.
- Implicit conversions exist from `T` to `T?` and from `null` to `T?`.
- `HasValue` indicates whether a value is present.
- `Value` returns the underlying value or throws `InvalidOperationException` if `HasValue` is `false`.
- `GetValueOrDefault()` returns the value or the default for `T` if null.
- Nullable value types are structs and therefore value types themselves; they are copied on assignment.

#### Constraints and Limitations

- `T` cannot be a reference type or a nullable value type.
- Reading `Value` on a null instance throws `InvalidOperationException`.
- Nullable value types do not implement the interfaces that the underlying type implements (except through boxing).

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

#### Example 2: Nullable Value Type in Arithmetic

```csharp
using System;

class Program
{
    static void Main()
    {
        int? x = 10;
        int? y = null;

        int? sum = x + y;   // null propagates
        Console.WriteLine(sum.HasValue);   // False
        Console.WriteLine(sum ?? -1);       // -1
    }
}
```

**Expected Output**

```
False
-1
```

**Why this output occurs**

- Arithmetic with a nullable operand returns `null` if any operand is `null`.
- The null-coalescing operator `??` provides a fallback.

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

## 2. `int?` (Nullable Shorthand)

### Definitions

**Core Definition**
`int?` is the shorthand syntax for `Nullable<int>`, representing an integer that can also be `null`.

**Technical Definition**
`int?` is a nullable value type where the underlying type is `System.Int32`. The `?` suffix is syntactic sugar for `System.Nullable<T>`. The two forms are interchangeable in source code, and the compiler treats them identically.

**Beginner-Friendly Explanation**
`int?` is just a shorter way to write “an `int` that might be empty.” Instead of writing `Nullable<int>`, you write `int?`.

---

### Purposes

- To provide concise, readable syntax for nullable value types.
- To improve code clarity when declaring optional integers.
- To integrate with null-conditional and null-coalescing operators.
- To enable pattern matching with `null` checks.
- To align with nullable reference type syntax (`string?`).

---

### Syntax Rules and Structure

```
int?   identifier   [ = value | null ]   ;
```

**Component breakdown**

- `int` — The underlying value type.
- `?` — Nullable modifier.
- `identifier` — The variable name.

#### Syntax Rules

- `int?` is interchangeable with `Nullable<int>`.
- Implicit conversions exist from `int` to `int?` and from `null` to `int?`.
- `int?` cannot be nested (`int??` is invalid).

#### Constraints and Limitations

- Runtime behaviour is identical to `Nullable<int>`.
- `int?` is a value type and is copied on assignment.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: `int?` in Practice

```csharp
using System;

class Program
{
    static void Main()
    {
        int? age = null;
        Console.WriteLine(age.HasValue);   // False

        age = 30;
        Console.WriteLine(age.Value);       // 30

        int safe = age ?? 0;
        Console.WriteLine(safe);            // 30
    }
}
```

**Expected Output**

```
False
30
30
```

**Why this output occurs**

- `age` starts as `null` (no value).
- Assigning `30` gives it a value.
- `??` provides a fallback when the nullable is null.

---

#### Example 2: `bool?` in a Condition

```csharp
using System;

class Program
{
    static void Main()
    {
        bool? isVerified = null;

        if (isVerified == true)
            Console.WriteLine("Verified");
        else
            Console.WriteLine("Not verified");
    }
}
```

**Expected Output**

```
Not verified
```

**Why this output occurs**

- `isVerified == true` evaluates to `false` when `isVerified` is `null`.
- The `else` branch runs.

---

### Real-World Cases

- **Optional numeric fields** — `int? Age` in a registration form.
- **Database columns** — `decimal? Discount` for nullable discounts.
- **State flags** — `bool? IsActive` where `null` means “unknown.”

---

### References

- Nullable value types — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/nullable-value-types

---

## 3. `Nullable<T>`

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

        object boxed1 = withValue;     // boxes as int 42
        object boxed2 = withoutValue;   // null

        Console.WriteLine(boxed1);           // 42
        Console.WriteLine(boxed2 == null);   // True
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

## 4. `HasValue`

### Definitions

**Core Definition**
`HasValue` is a read-only Boolean property of `Nullable<T>` that indicates whether the nullable instance contains a value.

**Technical Definition**
`Nullable<T>.HasValue` returns `true` if the instance has been assigned a non-null value of the underlying type, and `false` if it is `null`. It is the safe way to test whether a nullable has a value before accessing `Value`.

**Beginner-Friendly Explanation**
`HasValue` answers the question: “Does this box contain something?” If it returns `true`, you can safely open the box (`Value`); if `false`, the box is empty.

---

### Purposes

- To safely check whether a nullable has a value before reading `Value`.
- To avoid `InvalidOperationException` from accessing an empty nullable.
- To branch logic based on the presence or absence of a value.
- To validate optional data before processing.
- To support conditional expressions and pattern matching.

---

### Syntax Rules and Structure

```
bool   hasValue   =   nullableVariable.HasValue   ;
```

**Component breakdown**

- `nullableVariable` — A variable of type `T?` or `Nullable<T>`.
- `HasValue` — Read-only `bool` property.

#### Syntax Rules

- `HasValue` is `true` when a value is present; `false` when `null`.
- `HasValue` is safe to call on any nullable instance.
- `HasValue` is commonly used with `if` statements and conditional expressions.

#### Constraints and Limitations

- `HasValue` does not reveal the value itself; use `Value` for that.
- `HasValue` is redundant when using pattern matching (`is int value`).

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Checking Before Accessing `Value`

```csharp
using System;

class Program
{
    static void Main()
    {
        int? score = 85;

        if (score.HasValue)
        {
            Console.WriteLine($"Score: {score.Value}");
        }
        else
        {
            Console.WriteLine("No score");
        }
    }
}
```

**Expected Output**

```
Score: 85
```

**Why this output occurs**

- `HasValue` is `true`, so the `if` branch runs and `Value` is safe to access.

---

#### Example 2: `HasValue` with Null

```csharp
using System;

class Program
{
    static void Main()
    {
        int? score = null;
        Console.WriteLine(score.HasValue);   // False
    }
}
```

**Expected Output**

```
False
```

**Why this output occurs**

- The nullable is `null`, so `HasValue` is `false`.

---

### Real-World Cases

- **Optional configuration** — Checking whether a setting has been provided.
- **Database results** — Testing whether a column returned a value.
- **Form fields** — Determining whether the user supplied a value.

---

### References

- Nullable<T>.HasValue — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.nullable-1.hasvalue

---

## 5. `Value`

### Definitions

**Core Definition**
`Value` is a read-only property of `Nullable<T>` that returns the underlying value if one is present.

**Technical Definition**
`Nullable<T>.Value` returns the stored value of type `T`. If `HasValue` is `false`, accessing `Value` throws `InvalidOperationException`. It is the primary way to retrieve the actual value from a nullable after confirming that a value exists.

**Beginner-Friendly Explanation**
`Value` opens the box and gives you what is inside. But if the box is empty, it throws an error—so you must check `HasValue` first.

---

### Purposes

- To retrieve the underlying value from a nullable.
- To perform operations that require a non-nullable value.
- To extract values after confirming `HasValue` is `true`.
- To support arithmetic and method calls on nullable contents.
- To avoid implicit conversions and maintain type safety.

---

### Syntax Rules and Structure

```
T   actualValue   =   nullableVariable.Value   ;
```

**Component breakdown**

- `nullableVariable` — A `T?` or `Nullable<T>` instance.
- `Value` — Read-only property returning `T`.

#### Syntax Rules

- `Value` throws `InvalidOperationException` if `HasValue` is `false`.
- Always check `HasValue` or use pattern matching before accessing `Value`.
- `Value` is safe after a `null` check or in a branch where `HasValue` is known to be true.

#### Constraints and Limitations

- Accessing `Value` on a null nullable is a common source of runtime errors.
- `Value` does not perform any conversion; it returns the exact stored value.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Safe Access with `HasValue`

```csharp
using System;

class Program
{
    static void Main()
    {
        int? count = 5;

        if (count.HasValue)
        {
            int actual = count.Value;
            Console.WriteLine(actual * 2);   // 10
        }
    }
}
```

**Expected Output**

```
10
```

**Why this output occurs**

- `HasValue` is `true`, so `Value` returns `5`.
- The value is multiplied by `2`.

---

#### Example 2: `InvalidOperationException` on Null

```csharp
using System;

class Program
{
    static void Main()
    {
        int? count = null;

        try
        {
            Console.WriteLine(count.Value);
        }
        catch (InvalidOperationException)
        {
            Console.WriteLine("No value present");
        }
    }
}
```

**Expected Output**

```
No value present
```

**Why this output occurs**

- Accessing `Value` on a null nullable throws `InvalidOperationException`.

---

### Real-World Cases

- **Data extraction** — Retrieving optional database values after validation.
- **Calculations** — Using nullable inputs in arithmetic after confirming presence.
- **Configuration** — Reading optional settings with known defaults.

---

### References

- Nullable<T>.Value — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.nullable-1.value

---

## 6. `GetValueOrDefault()`

### Definitions

**Core Definition**
`GetValueOrDefault()` is a method on `Nullable<T>` that returns the underlying value if present, or a default value if the nullable is `null`.

**Technical Definition**
`Nullable<T>.GetValueOrDefault()` has two overloads: one with no parameters that returns `default(T)` (e.g., `0` for `int`, `false` for `bool`), and one that accepts a `T defaultValue` and returns that value when `HasValue` is `false`. It never throws.

**Beginner-Friendly Explanation**
`GetValueOrDefault()` is a safe way to say: “Give me the value if there is one; otherwise, give me a fallback.” It never crashes on null.

---

### Purposes

- To safely retrieve a value without risking an exception.
- To provide a fallback for null nullable values.
- To simplify code that would otherwise require `HasValue` checks.
- To supply sensible defaults for optional data.
- To support arithmetic and logic with nullable inputs.

---

### Syntax Rules and Structure

```
T   value   =   nullableVariable.GetValueOrDefault()   ;
T   value2   =   nullableVariable.GetValueOrDefault(defaultValue)   ;
```

**Component breakdown**

- `nullableVariable` — A `T?` or `Nullable<T>` instance.
- `GetValueOrDefault()` — Returns `default(T)` if null.
- `GetValueOrDefault(T defaultValue)` — Returns the specified fallback if null.

#### Syntax Rules

- Never throws for null nullable values.
- The parameterless overload returns `default(T)` (e.g., `0`, `false`, `null` for reference types).
- The overload with a fallback returns the fallback only when `HasValue` is `false`.

#### Constraints and Limitations

- The default value for the underlying type may not be meaningful in all contexts; choose an explicit fallback when it matters.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Parameterless `GetValueOrDefault()`

```csharp
using System;

class Program
{
    static void Main()
    {
        int? a = 42;
        int? b = null;

        Console.WriteLine(a.GetValueOrDefault());   // 42
        Console.WriteLine(b.GetValueOrDefault());   // 0
    }
}
```

**Expected Output**

```
42
0
```

**Why this output occurs**

- `a` has a value, so `GetValueOrDefault()` returns `42`.
- `b` is null, so the method returns the default for `int` (`0`).

---

#### Example 2: `GetValueOrDefault` with Custom Fallback

```csharp
using System;

class Program
{
    static void Main()
    {
        int? stock = null;
        int available = stock.GetValueOrDefault(-1);

        Console.WriteLine(available);   // -1
    }
}
```

**Expected Output**

```
-1
```

**Why this output occurs**

- `stock` is null, so the custom fallback `-1` is returned.

---

### Real-World Cases

- **Inventory systems** — Defaulting missing stock counts to `0` or `-1`.
- **Configuration** — Supplying default timeouts or limits.
- **Aggregations** — Treating null numeric values as zero in sums.

---

### References

- Nullable<T>.GetValueOrDefault — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.nullable-1.getvalueordefault

---

## 7. Nullable Reference Types (C# 8)

### Definitions

**Core Definition**
Nullable reference types are a compile-time annotation feature (introduced in C# 8.0) that lets you declare whether a reference variable is intended to hold `null` or not.

**Technical Definition**
Nullable reference types are not a separate runtime type. In a nullable-enabled context, every reference type variable is **non-nullable by default**: `string` means “intended to never be null,” and `string?` means “may be null.” The compiler performs static **null-state analysis**, tracking whether expressions are *not-null* or *maybe-null*, and issues warnings when the code’s behaviour may contradict its annotations. The feature is entirely compile-time; runtime behaviour is unchanged.

**Beginner-Friendly Explanation**
Nullable reference types are **labels on boxes**. Writing `string` says “this box must never be empty”; writing `string?` says “this box may be empty.” The compiler reads the labels and warns you if you try to use an empty box without checking.

---

### Purposes

- To express design intent about nullability in the type system.
- To reduce `NullReferenceException` by catching potential null dereferences at compile time.
- To make APIs self-documenting about optional and required values.
- To enable static analysis and tooling support.
- To improve code safety without changing runtime behaviour.

---

### Syntax Rules and Structure

```
// Non-nullable reference type (default in enabled context)
string   name   =   "Alice";

// Nullable reference type
string?   middleName   =   null;
```

**Component breakdown**

- `string` — Non-nullable reference type; compiler warns if assigned or returned `null`.
- `string?` — Nullable reference type; `null` is allowed, but dereference requires a null check.

#### Syntax Rules

- Nullable reference types require a nullable-enabled context (`<Nullable>enable</Nullable>` or `#nullable enable`).
- The `?` suffix is meaningful only in an enabled annotation context.
- Non-nullable variables have a default null-state of *not-null*; nullable variables default to *maybe-null*.
- Warnings are produced for possible null dereferences (CS8602) and possible null assignments to non-nullable variables (CS8600, CS8603).

#### Constraints and Limitations

- Nullable reference types do not change runtime types; `string` and `string?` are both `System.String`.
- The feature is opt-in; old projects may not have it enabled.
- Some constructs (base classes, interfaces, object creation) cannot use nullable reference type annotations.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Non-Nullable vs. Nullable Reference

```csharp
#nullable enable
using System;

class Program
{
    static void Main()
    {
        string required = "Hello";        // non-nullable
        string? optional = null;          // nullable

        Console.WriteLine(required.Length);   // 5

        if (optional is not null)
        {
            Console.WriteLine(optional.Length); // 0 (if optional were "")
        }
    }
}
```

**Expected Output**

```
5
```

**Why this output occurs**

- `required` is non-nullable and safe to dereference.
- `optional` is nullable; dereferencing it without a check would produce a warning.

---

#### Example 2: Warning When Dereferencing Nullable Without Check

```csharp
#nullable enable
using System;

class Program
{
    static void Main()
    {
        string? name = null;
        // Console.WriteLine(name.Length);   // Warning CS8602: possible null dereference

        if (name != null)
        {
            Console.WriteLine(name.Length);   // Safe
        }
    }
}
```

**Expected Output**

```
(no output; the safe branch is not executed)
```

**Why this output occurs**

- The commented line would produce a compiler warning because `name` is maybe-null.
- The null check makes the dereference safe.

---

### Real-World Cases

- **API contracts** — Marking parameters and return values as nullable or non-nullable.
- **Domain models** — `FirstName` (required) vs. `MiddleName?` (optional).
- **Configuration** — Optional settings with explicit nullability.
- **Testing** — Reducing null-related bugs in test code.

---

### References

- Nullable reference types — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/nullable-reference-types
- Nullable reference types — C# Fundamentals - https://learn.microsoft.com/en-us/dotnet/csharp/fundamentals/null-safety/nullable-reference-types
- Design with nullable reference types - https://learn.microsoft.com/en-us/dotnet/csharp/whats-new/tutorials/nullable-reference-types

---

## 8. Nullable Context

### Definitions

**Core Definition**
A nullable context is the compiler setting that determines whether nullable reference type annotations and warnings are active.

**Technical Definition**
A nullable context has two components: the **annotation context** (whether the `?` suffix on reference types is meaningful) and the **warning context** (whether the compiler emits nullable warnings). Both are controlled by project-level settings (`<Nullable>enable</Nullable>`) or file/block-level pragmas (`#nullable enable`, `#nullable disable`, `#nullable restore`). The context can be enabled, disabled, or set to warnings-only or annotations-only.

**Beginner-Friendly Explanation**
The nullable context is the **on/off switch** for nullable reference type checking. When it is on, the compiler pays attention to `?` labels and warns about null issues. When it is off, `?` has no special meaning and no warnings are produced.

---

### Purposes

- To control when nullable reference type checking is active.
- To enable gradual adoption of nullable reference types in existing codebases.
- To allow per-file or per-block overrides.
- To separate annotation intent from warning enforcement.
- To support migration strategies.

---

### Syntax Rules and Structure

```
// Project-level (csproj)
<Nullable>enable</Nullable>
<Nullable>disable</Nullable>
<Nullable>warnings</Nullable>
<Nullable>annotations</Nullable>

// File-level pragma
#nullable enable
#nullable disable
#nullable restore
```

**Component breakdown**

- `enable` — Enables both annotation and warning contexts.
- `disable` — Disables both.
- `warnings` — Enables warnings only.
- `annotations` — Enables annotations only.
- `#nullable restore` — Restores the project-level setting.

#### Syntax Rules

- The context can be set at the project level or via pragmas.
- `#nullable enable` enables both annotation and warning contexts.
- `#nullable disable` disables both.
- Pragmas can be placed anywhere in a file and affect subsequent code.

#### Constraints and Limitations

- Mixed contexts within a project can cause confusion; consistency is important.
- Older projects may not have nullable enabled by default.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Enabling Nullable Context in Code

```csharp
#nullable enable
using System;

class Program
{
    static void Main()
    {
        string? name = null;
        if (name != null)
        {
            Console.WriteLine(name.Length);
        }
    }
}
```

**Expected Output**

```
(no output; no warning because of the null check)
```

**Why this output occurs**

- `#nullable enable` activates nullable reference type checking.
- The null check makes the dereference safe.

---

#### Example 2: Disabling Warnings in a Block

```csharp
#nullable enable
using System;

class Program
{
    static void Main()
    {
        string? name = null;

#nullable disable
        Console.WriteLine(name.Length);   // no warning; context disabled
#nullable restore
    }
}
```

**Expected Output**

```
(System.NullReferenceException at runtime)
```

**Why this output occurs**

- `#nullable disable` turns off nullable warnings for the block.
- The code compiles without warnings but throws at runtime.

---

### Real-World Cases

- **Legacy migration** — Enabling nullable context file by file.
- **Third-party code** — Using pragmas to suppress warnings in generated code.
- **Libraries** — Providing nullable annotations while controlling warning levels.

---

### References

- Nullable contexts — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/nullable-references#nullable-context
- Nullable migration strategies - https://learn.microsoft.com/en-us/dotnet/csharp/nullable-migration-strategies

---

## 9. `#nullable enable`

### Definitions

**Core Definition**
`#nullable enable` is a preprocessor directive that turns on the nullable annotation and warning contexts for the code that follows it.

**Technical Definition**
The `#nullable enable` pragma activates both the annotation context (making `?` meaningful on reference types) and the warning context (causing the compiler to emit nullable warnings). It is equivalent to setting `<Nullable>enable</Nullable>` at the project level, but scoped to the file or block in which it appears.

**Beginner-Friendly Explanation**
`#nullable enable` is the **switch** that tells the compiler: “Pay attention to nullable labels and warn me about null problems from this point on.”

---

### Purposes

- To enable nullable reference type checking in a specific file.
- To support gradual migration in large codebases.
- To override project-level settings for specific code.
- To activate warnings for code that was previously unchecked.
- To align with modern C# null-safety practices.

---

### Syntax Rules and Structure

```
#nullable enable
```

**Component breakdown**

- `#nullable` — Preprocessor directive for nullable context.
- `enable` — Turns on both annotation and warning contexts.

#### Syntax Rules

- The directive affects all code that follows it in the file.
- It can be used multiple times to toggle the context.
- It does not affect other files.

#### Constraints and Limitations

- Requires C# 8.0 or later.
- Does not change runtime behaviour; only compiler analysis.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Enabling Nullable in a File

```csharp
#nullable enable
using System;

class Program
{
    static void Main()
    {
        string? name = null;
        // Console.WriteLine(name.Length);   // Warning CS8602

        Console.WriteLine(name?.Length ?? 0); // Safe
    }
}
```

**Expected Output**

```
0
```

**Why this output occurs**

- `#nullable enable` activates warnings.
- The null-conditional operator `?.` safely handles the null value.

---

#### Example 2: Mixing Contexts

```csharp
#nullable disable
string oldStyle = null;   // no warning

#nullable enable
string? newStyle = null;   // warning if dereferenced without check
```

**Expected Output**

```
(no warnings for oldStyle; warnings for newStyle if misused)
```

**Why this output occurs**

- The context toggles with each directive.

---

### Real-World Cases

- **File-by-file migration** — Adding `#nullable enable` as files are cleaned up.
- **Generated code** — Disabling nullable warnings in generated files.
- **Testing** — Enabling nullable checking in test projects.

---

### References

- #nullable directive — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/preprocessor-directives#nullable-context

---

## 10. Null-Forgiving Operator (`!`)

### Definitions

**Core Definition**
The null-forgiving operator (`!`) is a postfix operator that suppresses nullable warnings for an expression, telling the compiler “I know this is not null.”

**Technical Definition**
The null-forgiving, or null-suppression, operator (`!`) can be applied to an expression in a nullable-enabled context to suppress all nullable warnings for that expression. It has no runtime effect; it is purely a compile-time assertion. It is useful when the programmer has knowledge that the compiler’s static analysis cannot prove.

**Beginner-Friendly Explanation**
The `!` operator means “trust me—this is not null.” It tells the compiler to stop warning you, but it does not actually check anything at runtime. If you are wrong, you get a `NullReferenceException`.

---

### Purposes

- To suppress nullable warnings when the programmer knows more than the compiler.
- To satisfy the compiler when initializing non-nullable variables in constructors.
- To handle scenarios where null-state analysis cannot prove safety.
- To reduce warning noise in code with external guarantees.
- To support incremental adoption of nullable reference types.

---

### Syntax Rules and Structure

```
expression!
```

**Component breakdown**

- `expression` — Any expression that may produce a nullable warning.
- `!` — Null-forgiving operator; suppresses warnings for the preceding expression.

#### Syntax Rules

- The `!` operator is only meaningful in a nullable-enabled warning context.
- It does not perform a runtime null check.
- It applies only to the immediately preceding expression.

#### Constraints and Limitations

- Using `!` incorrectly can hide real null bugs.
- It should be used sparingly and with justification.
- It does not change the runtime type or value.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Suppressing a Warning

```csharp
#nullable enable
using System;

class Program
{
    static void Main()
    {
        string? maybeNull = GetValue();
        string notNull = maybeNull!;   // suppress warning
        Console.WriteLine(notNull.Length); // may throw at runtime if null
    }

    static string? GetValue() => "Hello";
}
```

**Expected Output**

```
5
```

**Why this output occurs**

- The `!` suppresses the warning that `maybeNull` might be null.
- At runtime, the value is non-null, so no exception occurs.

---

#### Example 2: Constructor Initialization

```csharp
#nullable enable
public class Person
{
    public string Name { get; }

    public Person(string name)
    {
        Name = name ?? throw new ArgumentNullException(nameof(name));
    }
}
```

**Expected Output**

```
(no warnings; Name is guaranteed non-null)
```

**Why this output occurs**

- The null-coalescing throw ensures `Name` is non-null.
- No `!` is needed because the compiler understands the guard.

---

### Real-World Cases

- **Constructor initialization** — When external guarantees ensure a value is non-null.
- **Interop** — Suppressing warnings for APIs that guarantee non-null returns.
- **Testing** — Temporarily suppressing warnings in tests.

---

### References

- Null-forgiving operator — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/null-forgiving
- ! (null-forgiving) operator — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/null-forgiving

---

## 11. Null-Coalescing Operator (`??`)

### Definitions

**Core Definition**
The null-coalescing operator (`??`) returns the left-hand operand if it is not null; otherwise, it returns the right-hand operand.

**Technical Definition**
The `??` operator is a binary operator that provides a fallback value. `a ?? b` evaluates to `a` if `a` is non-null, and to `b` if `a` is null. It works with both nullable value types and reference types. The right-hand operand is evaluated only if the left-hand operand is null.

**Beginner-Friendly Explanation**
`??` means “use this value, or if it is missing, use this other value.” It is a simple way to provide a default.

---

### Purposes

- To provide a default value for null expressions.
- To simplify null checks.
- To avoid `if (x == null)` branches.
- To support safe chaining of nullable operations.
- To improve code readability.

---

### Syntax Rules and Structure

```
result   =   leftOperand   ??   rightOperand   ;
```

**Component breakdown**

- `leftOperand` — An expression that may be null.
- `??` — Null-coalescing operator.
- `rightOperand` — The fallback value.

#### Syntax Rules

- The left operand can be a nullable value type or a reference type.
- The right operand must be implicitly convertible to the type of the left operand.
- The right operand is evaluated only when the left operand is null.
- `??` is right-associative: `a ?? b ?? c` is evaluated as `a ?? (b ?? c)`.

#### Constraints and Limitations

- The left operand must be a nullable type or a reference type.
- The result type is the underlying type of the left operand (or the common type).

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Default Value for Null

```csharp
using System;

class Program
{
    static void Main()
    {
        int? score = null;
        int finalScore = score ?? 0;
        Console.WriteLine(finalScore);   // 0

        string? name = null;
        string displayName = name ?? "Guest";
        Console.WriteLine(displayName);  // Guest
    }
}
```

**Expected Output**

```
0
Guest
```

**Why this output occurs**

- `score` is null, so `??` returns `0`.
- `name` is null, so `??` returns `"Guest"`.

---

#### Example 2: Chaining `??`

```csharp
using System;

class Program
{
    static void Main()
    {
        string? a = null;
        string? b = null;
        string c = "default";

        string result = a ?? b ?? c;
        Console.WriteLine(result);   // default
    }
}
```

**Expected Output**

```
default
```

**Why this output occurs**

- `a` is null, so the expression evaluates `b`.
- `b` is null, so it evaluates `c`.
- `c` is `"default"`.

---

### Real-World Cases

- **Configuration defaults** — Supplying fallback values for missing settings.
- **Database values** — Defaulting null columns to meaningful values.
- **User input** — Using a placeholder when input is missing.

---

### References

- ?? and ??= operators — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/null-coalescing-operator

---

## 12. Null-Coalescing Assignment (`??=`)

### Definitions

**Core Definition**
The null-coalescing assignment operator (`??=`) assigns the right-hand value to the left-hand variable only if the left-hand variable is null.

**Technical Definition**
`a ??= b` is equivalent to `a ?? (a = b)`: if `a` is null, `b` is evaluated and assigned to `a`; otherwise, `a` is unchanged. Introduced in C# 8.0, it is useful for lazily initializing variables.

**Beginner-Friendly Explanation**
`??=` means “if this is empty, fill it with this value; otherwise, leave it alone.”

---

### Purposes

- To lazily initialize variables with a default value.
- To avoid redundant assignments.
- To simplify null-check-and-assign patterns.
- To support caching and memoization.
- To make code more concise.

---

### Syntax Rules and Structure

```
leftVariable   ??=   rightValue   ;
```

**Component breakdown**

- `leftVariable` — A variable that may be null.
- `??=` — Null-coalescing assignment operator.
- `rightValue` — The value to assign if `leftVariable` is null.

#### Syntax Rules

- The right-hand side is evaluated only when the left-hand side is null.
- The left-hand side must be a variable, property, or indexer.
- Introduced in C# 8.0.

#### Constraints and Limitations

- The left-hand side must be assignable.
- Does not work with readonly variables.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Lazy Initialization

```csharp
using System;
using System.Collections.Generic;

class Program
{
    static void Main()
    {
        List<int>? data = null;
        data ??= new List<int> { 1, 2, 3 };

        Console.WriteLine(data.Count);   // 3
    }
}
```

**Expected Output**

```
3
```

**Why this output occurs**

- `data` is null, so `??=` assigns a new list.
- The list contains three elements.

---

#### Example 2: No Assignment When Non-Null

```csharp
using System;

class Program
{
    static void Main()
    {
        string? name = "Alice";
        name ??= "Guest";

        Console.WriteLine(name);   // Alice
    }
}
```

**Expected Output**

```
Alice
```

**Why this output occurs**

- `name` is not null, so `??=` does not change it.

---

### Real-World Cases

- **Caching** — Initializing a cache only when it is null.
- **Lazy loading** — Deferring object creation until first use.
- **Configuration defaults** — Setting defaults only when values are missing.

---

### References

- ?? and ??= operators — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/null-coalescing-operator

---

## 13. Null-Conditional Operator (`?.` and `?[]`)

### Definitions

**Core Definition**
The null-conditional operators (`?.` and `?[]`) access a member or element only when the target is not null; if the target is null, the expression evaluates to `null` instead of throwing.

**Technical Definition**
The `?.` operator performs member access (`obj?.Member`) and short-circuits when `obj` is null. The `?[]` operator performs indexer access (`obj?[index]`) with the same short-circuit behaviour. Both operators return a nullable result when the accessed member or element is a value type. Chains of `?.` short-circuit at the first null.

**Beginner-Friendly Explanation**
`?.` means “only access this member if the object is not null.” It is a safe way to navigate objects that might be null without writing explicit `if` checks.

---

### Purposes

- To safely access members on potentially null objects.
- To avoid `NullReferenceException`.
- To simplify null-checking code.
- To chain member accesses safely.
- To support safe indexer access on nullable collections.

---

### Syntax Rules and Structure

```
result   =   obj?.Member   ;
result2   =   obj?[index]   ;
```

**Component breakdown**

- `obj` — An expression that may be null.
- `?.` — Null-conditional member access.
- `?[]` — Null-conditional indexer access.

#### Syntax Rules

- If the left-hand side is null, the entire expression evaluates to `null`.
- The result type is nullable if the member’s type is a value type.
- Chains short-circuit at the first null.
- Can be combined with `??` to provide fallbacks.

#### Constraints and Limitations

- Cannot be used to assign to a member directly (use `??=` or explicit checks).
- The result of `?.` on a value-type member is a nullable value type.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Safe Member Access

```csharp
using System;

class Program
{
    static void Main()
    {
        string? name = null;
        int? length = name?.Length;
        Console.WriteLine(length.HasValue);   // False

        name = "C#";
        Console.WriteLine(name?.Length);        // 2
    }
}
```

**Expected Output**

```
False
2
```

**Why this output occurs**

- When `name` is null, `name?.Length` returns null.
- When `name` is `"C#"`, `Length` is `2`.

---

#### Example 2: Chaining `?.`

```csharp
using System;

class Program
{
    static void Main()
    {
        string? input = null;
        string? upper = input?.Trim()?.ToUpperInvariant();
        Console.WriteLine(upper ?? "(none)");   // (none)

        input = " hello ";
        Console.WriteLine(input?.Trim()?.ToUpperInvariant()); // HELLO
    }
}
```

**Expected Output**

```
(none)
HELLO
```

**Why this output occurs**

- The chain short-circuits at the first null (`input`).
- When `input` is non-null, `Trim()` and `ToUpperInvariant()` execute.

---

#### Example 3: `?[]` Indexer Access

```csharp
using System;

class Program
{
    static void Main()
    {
        string[]? tags = null;
        string? first = tags?[0];
        Console.WriteLine(first ?? "(none)");   // (none)

        tags = new[] { "csharp", "dotnet" };
        Console.WriteLine(tags?[0]);             // csharp
    }
}
```

**Expected Output**

```
(none)
csharp
```

**Why this output occurs**

- When `tags` is null, `tags?[0]` returns null.
- When `tags` is non-null, indexing works normally.

---

### Real-World Cases

- **Object graphs** — Safely navigating relationships that may be null.
- **Collections** — Accessing elements in possibly null arrays or lists.
- **Event handlers** — Invoking events with `?.Invoke()`.
- **DTOs** — Accessing optional nested properties.

---

### References

- Null-conditional operators — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/member-access-operators#null-conditional-operators--and-
- Null operators — C# Fundamentals - https://learn.microsoft.com/en-us/dotnet/csharp/fundamentals/null-safety/null-operators

---

## 14. Nullable Best Practices

### Definitions

**Core Definition**
Nullable best practices are the guidelines for using nullable value types, nullable reference types, and null-safe operators correctly and consistently.

**Technical Definition**
Best practices encompass enabling nullable reference types in new projects, expressing nullability intent through annotations, preferring null-safe operators (`?.`, `??`, `??=`) over explicit null checks, using `HasValue`/`Value` correctly for nullable value types, avoiding the null-forgiving operator unless justified, and leveraging nullable attributes for API contracts.

**Beginner-Friendly Explanation**
Best practices are the habits that keep null-related bugs out of your code: label your nullable intentions clearly, use safe operators, and check before you open a box that might be empty.

---

### Purposes

- To prevent `NullReferenceException`.
- To make nullability intent explicit in APIs.
- To leverage compiler analysis for safety.
- To reduce unnecessary null checks.
- To improve code readability and maintainability.

---

### Guidelines and Structure

#### General Guidelines

```
// Enable nullable reference types
<Nullable>enable</Nullable>

// Express intent with annotations
string required = "value";
string? optional = null;

// Prefer null-safe operators
var length = text?.Length ?? 0;
var name = input ?? "Guest";

// Use HasValue before Value
if (count.HasValue) { ... count.Value ... }

// Use GetValueOrDefault for safe retrieval
int safe = count.GetValueOrDefault();

// Use ! only when you know better than the compiler
string definitelyNotNull = maybeNull!;
```

**Component breakdown**

- **Enable nullable context** — Turn on the feature in new projects.
- **Annotate intent** — Use `?` for nullable, no `?` for non-nullable.
- **Safe operators** — Prefer `?.`, `??`, `??=` over nested `if` checks.
- **Nullable value types** — Check `HasValue` or use `GetValueOrDefault()`.
- **Null-forgiving** — Use sparingly and with justification.

#### Rules and Constraints

- Enable nullable reference types in new projects; migrate existing ones gradually.
- Always annotate public APIs to express nullability.
- Prefer null-safe operators over manual null checks.
- Avoid using `!` to silence warnings unless you have a strong guarantee.
- Use `HasValue`/`Value` or pattern matching for nullable value types.
- Use `GetValueOrDefault()` or `??` for safe fallbacks.
- Consider nullable attributes (`[NotNullWhen]`, `[MaybeNull]`) for complex API contracts.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Safe Access with `?.` and `??`

```csharp
#nullable enable
using System;

class Program
{
    static void Main()
    {
        string? name = null;
        int length = name?.Length ?? 0;

        Console.WriteLine(length);   // 0

        name = "Alice";
        Console.WriteLine(name?.Length ?? 0); // 5
    }
}
```

**Expected Output**

```
0
5
```

**Why this output occurs**

- When `name` is null, `?.Length` returns null and `?? 0` supplies `0`.
- When `name` is non-null, `Length` is used.

---

#### Example 2: Nullable Value Type with `GetValueOrDefault`

```csharp
using System;

class Program
{
    static void Main()
    {
        int? stock = null;
        int available = stock.GetValueOrDefault(-1);

        Console.WriteLine(available);   // -1
    }
}
```

**Expected Output**

```
-1
```

**Why this output occurs**

- `GetValueOrDefault(-1)` returns the fallback when the nullable is null.

---

### Real-World Cases

- **API design** — Expressing nullability in public contracts.
- **Data access** — Handling nullable database columns safely.
- **Configuration** — Providing defaults with `??` and `??=`.
- **Testing** — Reducing null-related test failures.

---

### References

- Nullable reference types — Best practices - https://learn.microsoft.com/en-us/dotnet/csharp/fundamentals/null-safety/nullable-reference-types
- Null operators — C# Fundamentals - https://learn.microsoft.com/en-us/dotnet/csharp/fundamentals/null-safety/null-operators
- Design with nullable reference types - https://learn.microsoft.com/en-us/dotnet/csharp/whats-new/tutorials/nullable-reference-types
- C# Nullable Types Explained (2026) - https://blog.ndepend.com/c-nullable-types-explained/

---

## Deprecated, Unsafe, Implementation-Dependent, and Version-Specific Notes

| Feature | Status / Note |
|---------|---------------|
| Nullable value types (`T?`) | Since **C# 2.0**; stable and widely used. |
| Nullable reference types | Introduced in **C# 8.0**; opt-in feature. Requires nullable context. |
| `#nullable enable` | Since **C# 8.0**; version-specific. |
| Null-forgiving operator (`!`) | Since **C# 8.0**; compile-time only, no runtime effect. |
| Null-coalescing assignment (`??=`) | Since **C# 8.0**; version-specific. |
| `GetValueOrDefault` | Available since .NET 2.0; stable. |
| Nullable attributes (`[NotNullWhen]`, etc.) | Since **C# 8.0+**; version-specific. |
| `required` members | Since **C# 11**; related to nullability and initialization. |
| Null-forgiving operator misuse | **Unsafe** if used to suppress warnings without a real guarantee. |
| Null-state analysis results | **Implementation-dependent**; may vary across compiler versions. |

---

## References (Consolidated)

- Nullable value types — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/nullable-value-types
- Nullable<T> struct — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.nullable-1
- Nullable<T>.HasValue — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.nullable-1.hasvalue
- Nullable<T>.Value — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.nullable-1.value
- Nullable<T>.GetValueOrDefault — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.nullable-1.getvalueordefault
- Nullable reference types — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/nullable-reference-types
- Nullable reference types — C# Fundamentals - https://learn.microsoft.com/en-us/dotnet/csharp/fundamentals/null-safety/nullable-reference-types
- Nullable contexts — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/nullable-references#nullable-context
- Nullable migration strategies - https://learn.microsoft.com/en-us/dotnet/csharp/nullable-migration-strategies
- #nullable directive — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/preprocessor-directives#nullable-context
- Null-forgiving operator — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/null-forgiving
- ?? and ??= operators — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/null-coalescing-operator
- Null-conditional operators — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/member-access-operators#null-conditional-operators--and-
- Null operators — C# Fundamentals - https://learn.microsoft.com/en-us/dotnet/csharp/fundamentals/null-safety/null-operators
- Design with nullable reference types - https://learn.microsoft.com/en-us/dotnet/csharp/whats-new/tutorials/nullable-reference-types
- C# Nullable Types Explained (2026) — NDepend Blog - https://blog.ndepend.com/c-nullable-types-explained/
- Taming the Billion Dollar Mistake — JetBrains Blog - https://blog.jetbrains.com/dotnet/2025/11/04/taming-the-billion-dollar-mistake-maarten-balliauw-s-guide-to-c-nullable-reference-types/
- Start Dealing With Nullable Reference Types — Xebia - https://xebia.com/blog/start-dealing-with-nullable-reference-types/

---