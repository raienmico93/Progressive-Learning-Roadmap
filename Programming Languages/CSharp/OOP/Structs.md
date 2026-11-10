# C# Structs — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
A **struct** (structure type) in C# is a **value type** that encapsulates data and related functionality into a single lightweight unit. Unlike classes, struct variables directly contain their data—there is no separate heap object and no reference indirection. Structs are the foundation of .NET’s primitive types (`int`, `double`, `bool`, `char`, `DateTime` are all structs) and are the recommended choice for small, data-centric types that behave like single values.

**Technical Definition**
A *structure type* (or *struct type*) is a value type that can encapsulate data and related functionality. Structure types have **value semantics**: a variable of a structure type contains an instance of the type, and by default the system copies variable values on assignment, when passing an argument to a method, and when returning a method result. Structs cannot inherit from other types (except implicitly from `System.ValueType`), cannot be inherited from, and cannot be `null` unless wrapped in `Nullable<T>`. Since C# 10, structs may declare explicit parameterless constructors and field initializers. The `readonly` modifier declares an immutable struct, requiring all fields to be `readonly` and all properties to be read-only or `init`-only. The `ref` modifier declares a **ref struct**, which is guaranteed to be allocated on the stack and cannot escape to the managed heap.

**Beginner-Friendly Explanation**
A struct is a **lightweight container for data**. Think of it as a small box that holds a few related pieces of information—like a `Point` with `X` and `Y` coordinates, or a `Money` value with an amount and a currency. When you copy the box, you get an independent copy of everything inside; changing your copy does not affect the original. This is different from a class, which is more like a **business card** pointing to a shared object on the heap. Structs are perfect for small, simple values because they avoid the overhead of heap allocation and garbage collection. But they come with trade-offs: every copy copies the entire struct, so large structs can be expensive, and mutable structs can cause subtle bugs because modifications to a copy are invisible to the original.

---

### Key Characteristics

- **Value type** — Struct variables directly contain their data; assignment copies the entire value.
- **Stack or inline allocation** — Structs are typically allocated on the stack or inline within a containing object, avoiding heap allocation and garbage collection.
- **No inheritance** — Structs cannot inherit from other structs or classes, and cannot be inherited from; they implicitly derive from `System.ValueType`.
- **No `null` by default** — A struct variable cannot be `null` unless it is a nullable value type (`T?`).
- **Immutability recommended** — Microsoft recommends immutable structs because mutable value types cause subtle copy-related bugs.
- **`readonly` structs** — The `readonly` modifier enforces immutability at the compiler level.
- **`ref` structs** — The `ref` modifier guarantees stack-only allocation and prevents heap escape.
- **Default zero-initialization** — A `default` struct has all fields set to `0`, `false`, or `null`; this state should be valid.
- **`IEquatable<T>` recommended** — Implementing `IEquatable<T>` avoids boxing and reflection in equality checks.

---

### Prerequisites

- Understanding of C# variables, methods, and properties.
- Familiarity with value types vs. reference types.
- Knowledge of constructors and object initialization.
- Awareness of the `readonly` and `ref` modifiers.
- Basic understanding of boxing and heap allocation.

---

### Related Programming Areas

- **Performance engineering** — Structs avoid heap allocations and reduce GC pressure in hot paths.
- **Interoperability** — Structs are used for P/Invoke and native memory layouts.
- **Mathematical and geometric types** — `Point`, `Vector`, `Complex`, `Color` are natural structs.
- **Records** — `record struct` (C# 10) combines struct performance with record equality and `with` expressions.
- **Span<T>** — `ref struct` types like `Span<T>` and `ReadOnlySpan<T>` enable allocation-free memory access.

---

### Core Concepts / Features

1. Structs (Overarching Concept)
2. `struct` Keyword
3. Value Semantics
4. `readonly` Structs
5. `ref` Structs
6. Struct Constructors
7. Struct Methods
8. Struct Best Practices
9. Struct vs. Class

Each concept follows a uniform structure: definitions, purposes, syntax rules and structure, annotated code examples with expected outputs, real-world cases, and references.

---

## 1. Structs (Overarching Concept)

### Definitions

**Core Definition**
A struct is a value type that groups related data and behaviour into a lightweight, copy-on-assignment unit.

**Technical Definition**
A structure type is a value type that can encapsulate data and related functionality. Structs are used to design small data-centric types that provide little or no behaviour; .NET uses structs for numbers, booleans, characters, and time instances. Because structs have value semantics, the system copies an instance on assignment, argument passing, and return. Structs cannot be `null` unless nullable, cannot inherit, and cannot be inherited from.

**Beginner-Friendly Explanation**
A struct is a **compact data package**. It holds related values together and behaves like a single value when copied.

---

### Purposes

- To represent small, data-centric values with value semantics.
- To avoid heap allocation and garbage collection for frequently used small types.
- To provide a lightweight alternative to classes when inheritance and reference identity are unnecessary.
- To serve as the foundation for .NET primitive types.

---

### Syntax Rules and Structure

```
[access modifier] [modifiers] struct StructName
{
    // fields, properties, methods, constructors
}
```

**Component breakdown**

- `struct` — The keyword introducing a value type.
- `StructName` — The struct identifier (PascalCase).
- Body — Fields, properties, methods, and constructors.

#### Syntax Rules

- Structs are value types; assignment copies the entire value.
- Structs cannot inherit from other types (except implicitly from `System.ValueType`).
- Structs cannot be `null` unless nullable (`T?`).
- Structs cannot declare a finalizer.

#### Constraints and Limitations

- Structs are copied on assignment and method calls; large structs are expensive to copy.
- Mutable structs can cause subtle bugs because modifications to a copy are invisible to the original.
- Structs should generally be small (guideline: under 16 bytes) and immutable.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Basic Struct with Value Semantics

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
}

class Program
{
    static void Main()
    {
        Point p1 = new Point(1, 2);
        Point p2 = p1;   // copy the entire struct

        p2.X = 100;

        Console.WriteLine($"p1: ({p1.X}, {p1.Y})");   // p1: (1, 2)
        Console.WriteLine($"p2: ({p2.X}, {p2.Y})");   // p2: (100, 2)
    }
}
```

**Expected Output**

```
p1: (1, 2)
p2: (100, 2)
```

**Why this output occurs**

- `p1` and `p2` are independent values. Assigning `p1` to `p2` copies the entire struct; modifying `p2.X` does not affect `p1.X`.

---

#### Example 2: Struct with Property and Method

```csharp
using System;

public struct Rectangle
{
    public double Width { get; }
    public double Height { get; }

    public Rectangle(double width, double height)
    {
        Width = width;
        Height = height;
    }

    public double Area() => Width * Height;
}

class Program
{
    static void Main()
    {
        var rect = new Rectangle(3, 4);
        Console.WriteLine(rect.Area());   // 12
    }
}
```

**Expected Output**

```
12
```

**Why this output occurs**

- The struct encapsulates dimensions and a computed method; because it is a value type, it is copied on assignment.

---

### Real-World Cases

- **Primitive-like types** — `int`, `double`, `bool`, `char`, `DateTime` are all structs.
- **Geometric primitives** — `Point`, `Size`, `Rectangle`.
- **Financial values** — `decimal` is a struct.

---

### References

- Structure types — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/struct
- struct keyword — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/struct

---

## 2. `struct` Keyword

### Definitions

**Core Definition**
The `struct` keyword is the syntactic marker that introduces a new structure type (value type) in C#.

**Technical Definition**
The `struct` keyword defines a structure type. It is followed by the struct identifier and a body containing fields, properties, methods, and constructors. The `struct` keyword distinguishes struct declarations from class and record declarations, and instructs the compiler to generate a value type that derives from `System.ValueType`.

**Beginner-Friendly Explanation**
The `struct` keyword is **how you start writing a struct**. You write `struct Point { ... }`.

---

### Purposes

- To introduce a new value type.
- To distinguish structs from classes and records.
- To signal value semantics to the compiler and readers.

---

### Syntax Rules and Structure

```
[access modifier] [modifiers] struct Name
{
    // members
}
```

**Component breakdown**

- `struct` — The keyword.
- `Name` — The struct identifier.
- Body — Fields, properties, methods, constructors.

#### Syntax Rules

- The keyword must be followed by a valid identifier.
- Struct names conventionally use PascalCase.
- The body is enclosed in braces.

#### Constraints and Limitations

- Cannot be used to declare classes or interfaces.

---

### Annotated Code Examples and Expected Outputs

#### Example: Using the `struct` Keyword

```csharp
using System;

public struct Coords
{
    public double X { get; }
    public double Y { get; }

    public Coords(double x, double y)
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
        var c = new Coords(1.5, 2.5);
        Console.WriteLine(c);   // (1.5, 2.5)
    }
}
```

**Expected Output**

```
(1.5, 2.5)
```

**Why this output occurs**

- The `struct` keyword declares `Coords` as a value type with properties and a `ToString` override.

---

### Real-World Cases

- **Every struct declaration** — The keyword is mandatory.

---

### References

- struct keyword — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/struct

---

## 3. Value Semantics

### Definitions

**Core Definition**
Value semantics means that a struct variable **contains the data itself**, and assignment or argument passing **copies the entire value**, so the source and destination are independent after the copy.

**Technical Definition**
Structure types have value semantics: a variable of a structure type contains an instance of the type. By default, the system copies variable values on assignment, when passing an argument to a method, and when returning a method result. For structure-type variables, the system copies an instance of the type. This is the opposite of reference semantics, where a variable holds a reference to a shared object on the heap.

**Beginner-Friendly Explanation**
Value semantics means **“copy the contents, not the address.”** If you hand someone a photocopy of a document, they can scribble on it without affecting your original.

---

### Purposes

- To provide predictable, independent copies on assignment.
- To avoid unintended shared mutable state.
- To enable efficient stack allocation for small values.
- To support value-based equality and hashing.

---

### Syntax Rules and Structure (Behaviour)

```
struct Point { public int X, Y; }

Point a = new Point { X = 1, Y = 2 };
Point b = a;          // b is an independent copy
b.X = 100;            // a.X remains 1
```

**Component breakdown**

- **Assignment** — Copies the struct’s fields.
- **Method arguments** — Value parameters copy the struct.
- **Return values** — Returning a struct copies it.

#### Syntax Rules

- Structs are copied on assignment, argument passing, and return.
- `ref`, `in`, and `out` modifiers can pass structs by reference to avoid copying.

#### Constraints and Limitations

- Large structs are expensive to copy.
- Mutable structs can be modified through copies without affecting the original, which can be confusing.

---

### Annotated Code Examples and Expected Outputs

#### Example: Independent Copies

```csharp
using System;

public struct Counter
{
    public int Value;
}

class Program
{
    static void Main()
    {
        var c1 = new Counter { Value = 10 };
        var c2 = c1;
        c2.Value = 20;

        Console.WriteLine(c1.Value);   // 10
        Console.WriteLine(c2.Value);   // 20
    }
}
```

**Expected Output**

```
10
20
```

**Why this output occurs**

- `c2` is an independent copy of `c1`; modifying `c2.Value` does not affect `c1.Value`.

---

### Real-World Cases

- **Mathematical values** — Adding two `Vector` structs produces a new struct without mutating inputs.
- **Configuration snapshots** — Copying a settings struct gives an independent snapshot.

---

### References

- Value types — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/value-types
- Structure types — value semantics - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/struct

---

## 4. `readonly` Structs

### Definitions

**Core Definition**
A `readonly` struct is a struct whose instance state **cannot be modified after construction**, enforced by the compiler through read-only fields and read-only or `init`-only properties.

**Technical Definition**
Use the `readonly` modifier to declare that a structure type is immutable. All data members of a `readonly` struct must be read-only: any field declaration must have the `readonly` modifier, and any property (including auto-implemented ones) must be read-only or `init`-only. This rule guarantees that no member of a `readonly` struct modifies the state of the struct. All other instance members except constructors are implicitly `readonly`. A `readonly` struct can still hold a reference to a mutable object (e.g., a `List<T>`), and that object’s contents can be mutated—only the reference itself is immutable.

**Beginner-Friendly Explanation**
A `readonly` struct is a **frozen value**. Once you create it, you cannot change its fields or properties. It is like a sealed envelope—you can read the contents but cannot replace them.

---

### Purposes

- To enforce immutability at compile time.
- To prevent accidental mutation and copy-related bugs.
- To enable compiler optimizations (defensive copies can be avoided).
- To provide thread-safe, predictable values.

---

### Syntax Rules and Structure

```
public readonly struct Point
{
    public readonly int X { get; }
    public readonly int Y { get; }

    public Point(int x, int y) => (X, Y) = (x, y);
}
```

**Component breakdown**

- `readonly` — The modifier.
- Fields must be `readonly`.
- Properties must be get-only or `init`-only.

#### Syntax Rules

- All instance fields must be `readonly`.
- All properties must be read-only or `init`-only.
- All other instance members (except constructors) are implicitly `readonly`.

#### Constraints and Limitations

- A `readonly` struct can contain a mutable reference type; its contents can still be mutated.
- `readonly` structs cannot declare field-like events.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Readonly Struct with `init` Properties

```csharp
using System;

public readonly struct Temperature
{
    public double Celsius { get; init; }

    public Temperature(double celsius) => Celsius = celsius;

    public double Fahrenheit => Celsius * 9.0 / 5.0 + 32.0;
}

class Program
{
    static void Main()
    {
        var t = new Temperature(25);
        Console.WriteLine(t.Fahrenheit);   // 77
    }
}
```

**Expected Output**

```
77
```

**Why this output occurs**

- `Celsius` is `init`-only, and `Fahrenheit` is a computed property. The struct is immutable after construction.

---

#### Example 2: Shallow Immutability

```csharp
using System;
using System.Collections.Generic;

public readonly struct Container
{
    public List<int> Items { get; }

    public Container(List<int> items) => Items = items;
}

class Program
{
    static void Main()
    {
        var list = new List<int> { 1, 2, 3 };
        var c = new Container(list);
        c.Items.Add(4);   // allowed: the list contents are mutable
        Console.WriteLine(c.Items.Count);   // 4
    }
}
```

**Expected Output**

```
4
```

**Why this output occurs**

- The `readonly` modifier prevents replacing `Items`, but the referenced list itself remains mutable.

---

### Real-World Cases

- **Immutable value objects** — `readonly record struct Money(decimal Amount, string Currency)`.
- **Geometric primitives** — `readonly struct Point(double X, double Y)`.
- **Configuration snapshots** — Immutable settings structs.

---

### References

- readonly struct — Structure types (C# reference) - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/struct#readonly-struct
- readonly instance members — Structure types (C# reference) - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/struct#readonly-instance-members

---

## 5. `ref` Structs

### Definitions

**Core Definition**
A `ref` struct is a struct declared with the `ref` modifier that is **guaranteed to be allocated on the stack** and **cannot escape to the managed heap**.

**Technical Definition**
You use the `ref` modifier when declaring a structure type. Instances of a `ref` struct type are allocated on the stack and cannot escape to the managed heap. To ensure this property, the compiler limits the usage of `ref` struct types: they cannot be boxed, cannot be used as array elements, cannot be type arguments, cannot be captured by lambdas or local functions, and cannot be used in `async` or iterator methods (with limited exceptions in C# 13+). `ref struct` types can also be `readonly ref struct`, combining stack-only allocation with immutability. `Span<T>` and `ReadOnlySpan<T>` are the most common `ref struct` types.

**Beginner-Friendly Explanation**
A `ref` struct is a **stack-only value**. It lives on the stack and cannot be moved to the heap. This makes it extremely efficient for temporary views over memory, like `Span<T>`.

---

### Purposes

- To provide high-performance, stack-only views over memory.
- To avoid heap allocations for temporary buffers and slices.
- To enable safe, allocation-free slicing with `Span<T>` and `ReadOnlySpan<T>`.
- To support low-level, high-performance scenarios without unsafe code.

---

### Syntax Rules and Structure

```
public ref struct SpanWrapper
{
    public Span<int> Span { get; }
    public SpanWrapper(Span<int> span) => Span = span;
}

// readonly ref struct
public readonly ref struct ReadOnlyWrapper
{
    public ReadOnlySpan<char> Text { get; }
    public ReadOnlyWrapper(ReadOnlySpan<char> text) => Text = text;
}
```

**Component breakdown**

- `ref` — The modifier (before `struct`).
- `readonly ref` — Combines immutability with stack-only allocation.
- Members must not allow heap escape.

#### Syntax Rules

- `ref struct` instances are stack-allocated.
- Cannot be boxed, stored in arrays, or used as type arguments.
- Cannot be captured by lambdas or iterators (with C# 13+ exceptions).
- `readonly ref struct` requires all fields to be `readonly ref` or `readonly ref readonly`.

#### Constraints and Limitations

- Cannot escape to the heap.
- Cannot be used in `async` methods (prior to C# 13).
- Cannot implement interfaces (prior to C# 13; limited support since).

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Using `Span<T>` (a `ref struct`)

```csharp
using System;

class Program
{
    static void Main()
    {
        int[] numbers = { 1, 2, 3, 4, 5 };
        Span<int> slice = numbers.AsSpan(1, 3);

        slice[0] = 20;   // mutates the original array

        Console.WriteLine(numbers[1]);   // 20
        Console.WriteLine(slice[0]);      // 20
    }
}
```

**Expected Output**

```
20
20
```

**Why this output occurs**

- `Span<int>` is a `ref struct` that provides a stack-only view over the array; modifications through the span affect the original array.

---

#### Example 2: Custom `ref struct`

```csharp
using System;

public ref struct BufferView
{
    public Span<byte> Data { get; }

    public BufferView(Span<byte> data) => Data = data;
}

class Program
{
    static void Main()
    {
        byte[] buffer = { 10, 20, 30 };
        var view = new BufferView(buffer.AsSpan());
        Console.WriteLine(view.Data[1]);   // 20
    }
}
```

**Expected Output**

```
20
```

**Why this output occurs**

- `BufferView` is a `ref struct` that wraps a `Span<byte>` without heap allocation.

---

### Real-World Cases

- **`Span<T>` and `ReadOnlySpan<T>`** — Allocation-free slicing of arrays, strings, and native memory.
- **Parsing and formatting** — `Utf8Parser` and `Utf8Formatter` use `ref struct` views.
- **High-performance networking** — Zero-copy buffers using `Span<byte>`.

---

### References

- ref struct types — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/ref-struct
- Span<T> — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.span-1

---

## 6. Struct Constructors

### Definitions

**Core Definition**
A struct constructor initializes the struct’s fields and properties. Structs can declare parameterized constructors and, since C# 10, explicit parameterless constructors and field initializers.

**Technical Definition**
A struct may declare instance constructors with zero or more parameters. If a struct does not explicitly declare a parameterless instance constructor, the compiler synthesizes a public parameterless constructor that returns the value resulting from setting all value-type fields to their default values and all reference-type fields to `null`. Since C# 10, structs may declare explicit parameterless constructors and instance field initializers; an explicit parameterless struct constructor must be declared `public`. For `readonly` structs, constructors must initialize all fields. For `ref` structs, constructors follow the same rules.

**Beginner-Friendly Explanation**
A struct constructor is the **setup routine that runs when you create a struct instance**. You can require specific values (parameterized) or provide a custom default (parameterless, C# 10+).

---

### Purposes

- To initialize struct fields to meaningful values.
- To enforce invariants at creation.
- To provide convenient construction paths.
- To support `readonly` and `ref` structs with proper initialization.

---

### Syntax Rules and Structure

```
public struct Point
{
    public int X { get; }
    public int Y { get; }

    public Point(int x, int y) => (X, Y) = (x, y);

    // Parameterless constructor (C# 10+)
    public Point() => (X, Y) = (0, 0);
}
```

**Component breakdown**

- `public Point(int x, int y)` — Parameterized constructor.
- `public Point()` — Explicit parameterless constructor (C# 10+).

#### Syntax Rules

- Parameterized constructors must initialize all fields.
- Explicit parameterless constructors must be `public` (C# 10+).
- `readonly` struct constructors must initialize all fields.
- Structs can have multiple constructors.

#### Constraints and Limitations

- Prior to C# 10, structs could not declare parameterless constructors.
- Constructors cannot be `virtual` or `abstract`.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Parameterized Constructor

```csharp
using System;

public struct Point
{
    public int X { get; }
    public int Y { get; }

    public Point(int x, int y)
    {
        X = x;
        Y = y;
    }
}

class Program
{
    static void Main()
    {
        var p = new Point(3, 4);
        Console.WriteLine($"{p.X}, {p.Y}");   // 3, 4
    }
}
```

**Expected Output**

```
3, 4
```

**Why this output occurs**

- The parameterized constructor initializes `X` and `Y`.

---

#### Example 2: Parameterless Constructor (C# 10+)

```csharp
using System;

public struct Counter
{
    public int Value { get; }

    public Counter() => Value = 42;
}

class Program
{
    static void Main()
    {
        var c = new Counter();
        Console.WriteLine(c.Value);   // 42
    }
}
```

**Expected Output**

```
42
```

**Why this output occurs**

- The explicit parameterless constructor sets `Value` to `42` when `new Counter()` is called.

---

### Real-World Cases

- **Value objects** — Constructors that validate and assign.
- **Geometric types** — `Point(x, y)`, `Rectangle(width, height)`.
- **Default construction** — `new Point()` for zero-initialized structs.

---

### References

- Structure types — constructors - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/struct
- Parameterless struct constructors — C# 10 feature specifications - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/proposals/csharp-10.0/parameterless-struct-constructors

---

## 7. Struct Methods

### Definitions

**Core Definition**
Struct methods are instance or static members that define behaviour for the struct. Instance methods operate on the struct’s data; static methods belong to the type.

**Technical Definition**
Structs can contain methods just like classes. Instance methods operate on a copy of the struct when called through a value parameter, but can modify the original when called through a `ref` parameter. Methods can be marked `readonly` to indicate they do not modify the struct’s state; within a `readonly` method, the compiler creates a defensive copy if the method calls a non-`readonly` member. Structs can also implement interfaces and override `Object` methods such as `ToString`, `Equals`, and `GetHashCode`.

**Beginner-Friendly Explanation**
Struct methods are the **actions the struct can perform**. They can compute values, validate data, or produce formatted output.

---

### Purposes

- To provide behaviour related to the struct’s data.
- To compute derived values without storing them.
- To override `ToString` and `Equals` for better usability.
- To implement interface contracts.

---

### Syntax Rules and Structure

```
public struct Point
{
    public int X { get; }
    public int Y { get; }

    public double Distance() => Math.Sqrt(X * X + Y * Y);

    public readonly override string ToString() => $"({X}, {Y})";
}
```

**Component breakdown**

- Instance methods — Operate on the struct’s data.
- `readonly` methods — Promise not to modify the struct.
- Overrides — `ToString`, `Equals`, `GetHashCode`.

#### Syntax Rules

- Instance methods can modify the struct unless marked `readonly`.
- Static methods belong to the type.
- `readonly` methods cannot assign to fields.

#### Constraints and Limitations

- Mutable struct methods can cause copy-related bugs.
- `readonly` methods may create defensive copies when calling non-`readonly` members.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Instance Method

```csharp
using System;

public struct Point
{
    public int X { get; }
    public int Y { get; }

    public Point(int x, int y) => (X, Y) = (x, y);

    public double Distance() => Math.Sqrt(X * X + Y * Y);
}

class Program
{
    static void Main()
    {
        var p = new Point(3, 4);
        Console.WriteLine(p.Distance());   // 5
    }
}
```

**Expected Output**

```
5
```

**Why this output occurs**

- `Distance` computes the Euclidean distance from the origin.

---

#### Example 2: `readonly` Method and `ToString`

```csharp
using System;

public struct Point
{
    public int X { get; }
    public int Y { get; }

    public Point(int x, int y) => (X, Y) = (x, y);

    public readonly override string ToString() => $"({X}, {Y})";
}

class Program
{
    static void Main()
    {
        var p = new Point(1, 2);
        Console.WriteLine(p);   // (1, 2)
    }
}
```

**Expected Output**

```
(1, 2)
```

**Why this output occurs**

- `ToString` is `readonly`; it does not modify the struct and provides a readable representation.

---

### Real-World Cases

- **Mathematical operations** — `Vector.Normalize()`, `Complex.Magnitude()`.
- **Formatting** — `ToString` overrides for debugging and logging.
- **Validation** — `IsValid()` methods on value objects.

---

### References

- Structure types — methods - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/struct
- readonly instance members — Structure types - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/struct#readonly-instance-members

---

## 8. Struct Best Practices

### Definitions

**Core Definition**
Struct best practices are the design guidelines that ensure structs are **small, immutable, and appropriately chosen** over classes, avoiding the performance and correctness pitfalls of mutable value types.

**Technical Definition**
Microsoft’s Framework Design Guidelines provide specific recommendations for struct design: **do not provide a parameterless constructor** (pre-C# 10; C# 10+ allows it but the default zero-initialized state must remain valid), **do not define mutable value types**, **ensure the zero-initialized state is valid**, **implement `IEquatable<T>`** to avoid boxing and reflection in equality checks, and **do not explicitly extend `ValueType`**. The general rule of thumb is that the majority of types should be classes; consider a struct only when instances are small (under 16 bytes), short-lived, commonly embedded in other objects, logically represent a single value, are immutable, and will not be boxed frequently.

**Beginner-Friendly Explanation**
Best practices for structs are simple: **keep them small, keep them immutable, and make sure the default (all-zero) state makes sense**. Do not use a struct just to save memory; use one only when the type truly behaves like a single value.

---

### Purposes

- To prevent performance degradation from large or boxed structs.
- To avoid subtle bugs caused by mutable value types.
- To ensure that default-initialized structs are valid.
- To align with .NET design conventions.

---

### Guidelines and Structure

#### General Guidelines

```
// Prefer immutable structs
public readonly struct Point
{
    public int X { get; }
    public int Y { get; }
}

// Implement IEquatable<T>
public readonly struct Point : IEquatable<Point>
{
    public bool Equals(Point other) => X == other.X && Y == other.Y;
    public override bool Equals(object? obj) => obj is Point p && Equals(p);
    public override int GetHashCode() => HashCode.Combine(X, Y);
}

// Keep structs small (< 16 bytes)
// Avoid mutable value types
// Ensure zero-initialized state is valid
```

**Component breakdown**

- **Immutable** — Use `readonly` or `readonly` fields.
- **`IEquatable<T>`** — Avoid boxing in equality checks.
- **Small size** — Keep under 16 bytes for efficient copying.
- **Valid default** — All-zero state should be meaningful.

#### Rules and Constraints

- **Do** make structs immutable.
- **Do** implement `IEquatable<T>`.
- **Do** ensure the default state is valid.
- **Do not** define mutable value types.
- **Do not** provide a parameterless constructor unless you have a specific need (and use C# 10+).
- **Do not** box structs frequently.

---

### Annotated Code Examples and Expected Outputs

#### Example: Well-Designed Immutable Struct

```csharp
using System;

public readonly struct Money : IEquatable<Money>
{
    public decimal Amount { get; }
    public string Currency { get; }

    public Money(decimal amount, string currency)
    {
        Amount = amount;
        Currency = currency ?? throw new ArgumentNullException(nameof(currency));
    }

    public bool Equals(Money other) => Amount == other.Amount && Currency == other.Currency;
    public override bool Equals(object? obj) => obj is Money m && Equals(m);
    public override int GetHashCode() => HashCode.Combine(Amount, Currency);
    public override string ToString() => $"{Amount} {Currency}";
}

class Program
{
    static void Main()
    {
        var m1 = new Money(10m, "USD");
        var m2 = new Money(10m, "USD");
        Console.WriteLine(m1.Equals(m2));   // True
        Console.WriteLine(m1);               // 10 USD
    }
}
```

**Expected Output**

```
True
10 USD
```

**Why this output occurs**

- The struct is immutable, implements `IEquatable<T>`, and provides value equality and readable formatting.

---

### Real-World Cases

- **Financial values** — `Money`, `Price`, `Amount`.
- **Geometric primitives** — `Point`, `Size`, `Rectangle`.
- **Identifiers** — `Guid`-like small value types.

---

### References

- Struct Design — Framework Design Guidelines - https://learn.microsoft.com/en-us/dotnet/standard/design-guidelines/struct
- Choosing Between Classes and Structures — Microsoft Learn - https://learn.microsoft.com/en-us/previous-versions/dotnet/netframework-2.0/ms229017(v=vs.80)

---

## 9. Struct vs. Class

### Definitions

**Core Definition**
Structs and classes are the two primary user-defined types in C#. **Structs are value types** (copy semantics, stack/inline allocation), while **classes are reference types** (reference semantics, heap allocation). Choosing between them is a fundamental design decision.

**Technical Definition**
Classes are reference types and structures are value types. Reference types are allocated on the heap and managed by the garbage collector; value types are allocated on the stack or inline and are deallocated when they go out of scope. In general, value types are cheaper to allocate and deallocate, but they perform poorly in scenarios that require significant boxing and unboxing. Consider defining a struct instead of a class if instances are small and commonly short-lived or commonly embedded in other objects. Do not define a struct unless the type logically represents a single value, has an instance size smaller than 16 bytes, is immutable, and will not be boxed frequently.

**Beginner-Friendly Explanation**
Use a **class** for complex objects that have identity, mutable state, or need inheritance. Use a **struct** for small, simple values that behave like numbers—things you copy and compare by value.

---

### Purposes

- To choose the right type for the scenario.
- To optimize memory allocation and performance.
- To avoid boxing and reference-equality confusion.
- To model value-like concepts naturally.

---

### Syntax Rules and Structure (Comparison)

| Aspect | Struct | Class |
|---|---|---|
| **Type kind** | Value type | Reference type |
| **Allocation** | Stack or inline | Heap |
| **Assignment** | Copies the value | Copies the reference |
| **Nullability** | Cannot be `null` unless `T?` | Can be `null` |
| **Inheritance** | No inheritance | Supports inheritance |
| **Size guideline** | < 16 bytes | Any size |
| **Default state** | Zero-initialized; must be valid | `null` reference |
| **Boxing** | Boxed when cast to `object` or interface | Not boxed |
| **Mutability** | Should be immutable | Can be mutable |

**Component breakdown**

- **Value semantics** — Structs.
- **Reference semantics** — Classes.
- **Inheritance** — Classes only.
- **Nullability** — Classes can be `null`.

#### Syntax Rules

- Use structs for small, immutable values.
- Use classes for complex behaviour and identity.
- Implement `IEquatable<T>` on structs to avoid boxing.

#### Constraints and Limitations

- Large structs are expensive to copy.
- Mutable structs cause copy-related bugs.
- Classes require heap allocation and garbage collection.

---

### Annotated Code Examples and Expected Outputs

#### Example: Struct vs. Class Behaviour

```csharp
using System;

public struct PointStruct { public int X, Y; }
public class PointClass { public int X, Y; }

class Program
{
    static void Main()
    {
        // Struct: value semantics
        var s1 = new PointStruct { X = 1, Y = 2 };
        var s2 = s1;
        s2.X = 100;
        Console.WriteLine($"Struct: s1.X = {s1.X}, s2.X = {s2.X}"); // Struct: s1.X = 1, s2.X = 100

        // Class: reference semantics
        var c1 = new PointClass { X = 1, Y = 2 };
        var c2 = c1;
        c2.X = 100;
        Console.WriteLine($"Class: c1.X = {c1.X}, c2.X = {c2.X}"); // Class: c1.X = 100, c2.X = 100
    }
}
```

**Expected Output**

```
Struct: s1.X = 1, s2.X = 100
Class: c1.X = 100, c2.X = 100
```

**Why this output occurs**

- `s2` is a copy of `s1`; modifying `s2.X` does not affect `s1`. `c2` and `c1` reference the same object, so modifying `c2.X` changes `c1.X`.

---

### Real-World Cases

- **Use structs for** — `Point`, `Size`, `Color`, `Money`, `DateTime`, `Guid`.
- **Use classes for** — `Customer`, `Order`, `Service`, `Repository`, `Controller`.

---

### References

- Choosing Between Classes and Structures — Microsoft Learn - https://learn.microsoft.com/en-us/previous-versions/dotnet/netframework-2.0/ms229017(v=vs.80)
- Struct Design — Framework Design Guidelines - https://learn.microsoft.com/en-us/dotnet/standard/design-guidelines/struct
- Classes, structs, and records — C# - https://learn.microsoft.com/en-us/dotnet/csharp/fundamentals/object-oriented/

---

## Deprecated, Unsafe, Implementation-Dependent, and Version-Specific Notes

| Feature | Status / Note |
|---------|---------------|
| Parameterless struct constructors | Introduced in **C# 10**; earlier versions disallowed explicit parameterless constructors. |
| `readonly` structs | Introduced in **C# 7.2**; stable and recommended. |
| `ref` structs | Introduced in **C# 7.2**; stack-only allocation, no heap escape. |
| `record struct` | Introduced in **C# 10**; combines struct performance with record equality. |
| `readonly` instance members | Available since **C# 8.0**; allows marking individual members as non-mutating. |
| Mutable structs | **Discouraged**; can cause subtle copy-related bugs. |
| Boxing of structs | **Performance concern**; implement `IEquatable<T>` to avoid boxing in equality checks. |
| Zero-initialized default state | **Must be valid**; arrays of structs are zero-initialized without running constructors. |

---

## References (Consolidated)

- Structure types — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/struct
- Value types — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/value-types
- readonly struct — Structure types (C# reference) - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/struct#readonly-struct
- readonly instance members — Structure types (C# reference) - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/struct#readonly-instance-members
- ref struct types — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/ref-struct
- Span<T> — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.span-1
- Parameterless struct constructors — C# 10 feature specifications - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/proposals/csharp-10.0/parameterless-struct-constructors
- Struct Design — Framework Design Guidelines - https://learn.microsoft.com/en-us/dotnet/standard/design-guidelines/struct
- Choosing Between Classes and Structures — Microsoft Learn - https://learn.microsoft.com/en-us/previous-versions/dotnet/netframework-2.0/ms229017(v=vs.80)
- Classes, structs, and records — C# - https://learn.microsoft.com/en-us/dotnet/csharp/fundamentals/object-oriented/
- Records — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/record
- IEquatable<T> — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.iequatable-1

---