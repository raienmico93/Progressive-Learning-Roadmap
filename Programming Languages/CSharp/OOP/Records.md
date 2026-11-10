# C# Records — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
A **record** in C# is a data-centric type (introduced in C# 9) that the compiler equips with **value-based equality**, a concise **positional syntax** for defining immutable properties, **non-destructive mutation** via `with` expressions, and a built-in human-readable `ToString`. Records can be either **reference types** (`record class`, or simply `record`) or, since C# 10, **value types** (`record struct`). They are designed for scenarios where a type’s primary role is to **store data** and two instances with the same values should be considered equal.

**Technical Definition**
The `record` modifier provides built-in functionality for encapsulating data. The `record class` and `record` syntax define **reference types**; the `record struct` syntax defines a **value type**. When a primary constructor is declared on a record, the compiler generates public **positional properties** that mirror the constructor parameters (init-only for `record class` and `readonly record struct`; read-write for `record struct`). The compiler synthesizes `Equals`, `GetHashCode`, `operator ==`, `operator !=`, `ToString`, a `Deconstruct` method (for positional records), and, for `record class`, a copy constructor and a virtual `Clone` method that power `with` expressions. Records support inheritance only from other records (`record class`); `record struct` types are implicitly sealed and cannot participate in inheritance. `with` expressions perform **shallow copies** and are the mechanism for non-destructive mutation.

**Beginner-Friendly Explanation**
A record is a **smart data container**. Instead of writing dozens of lines of boilerplate for a class that just holds data, you write `public record Person(string FirstName, string LastName);` and C# generates everything you need: a constructor, properties, equality checks, a nice `ToString`, and the ability to create a modified copy with `with`. Records are ideal when **two objects with the same data should be considered equal**—like a point in geometry, a DTO from an API, or a configuration snapshot. The `record struct` variant gives you the same conveniences but with value-type semantics (stack allocation, copy on assignment), making it perfect for small, self-contained values.

---

### Key Characteristics

- **Value-based equality** — Two records of the same type are equal if their fields and auto-implemented properties are equal.
- **Concise positional syntax** — `public record Point(int X, int Y);` generates a constructor, properties, and a deconstructor.
- **Immutable by default (record class)** — Positional properties are `init`-only; mutation requires a `with` expression.
- **`with` expressions** — Create a modified shallow copy without mutating the original.
- **Built-in `ToString`** — Produces a readable `TypeName { Property = Value, ... }` string.
- **Record structs (C# 10)** — Value-type records with the same equality and `with` semantics; properties are read-write unless `readonly record struct` is used.
- **Inheritance (record class only)** — Records can inherit from other records; they cannot inherit from classes, and classes cannot inherit from records.
- **Shallow immutability** — `init`-only properties prevent reassignment, but the contents of reference-type properties remain mutable.
- **Compiler-generated `Deconstruct`** — Positional records support deconstruction into their positional parameters.

---

### Prerequisites

- Understanding of C# classes, structs, properties, and constructors.
- Familiarity with value types vs. reference types and equality semantics.
- Knowledge of `init` accessors (C# 9) and object initializers.
- Awareness of `with` expressions and non-destructive mutation.
- Basic understanding of inheritance (for `record class` hierarchies).

---

### Related Programming Areas

- **DTOs and API models** — Records are the idiomatic choice for data transfer objects.
- **Value objects** — Domain concepts like `Money`, `Point`, `DateRange` benefit from value equality.
- **Immutable data models** — Configuration snapshots, event-sourcing payloads, and functional pipelines.
- **Pattern matching** — Records integrate naturally with `switch` expressions and positional patterns.
- **Serialization** — Records work seamlessly with JSON serializers and ORMs (with caveats for EF Core).

---

### Core Concepts / Features

1. Records (C# 9)
2. Record Declaration
3. Record Structs (C# 10)
4. Positional Records
5. `with` Expression
6. Record Equality
7. Record Immutability
8. Record Inheritance
9. Record Best Practices

Each concept follows a uniform structure: definitions, purposes, syntax rules and structure, annotated code examples with expected outputs, real-world cases, and references.

---

## 1. Records (C# 9) — Overarching Concept

### Definitions

**Core Definition**
A record is a C# type (reference or value) whose compiler-generated members provide value-based equality, concise data definition, and non-destructive mutation.

**Technical Definition**
The `record` modifier provides built-in functionality for encapsulating data. The `record class` and `record` syntax define reference types; the `record struct` syntax defines a value type. Records are primarily intended for supporting immutable data models, though they can be mutable. The record modifier causes the compiler to synthesize: an override of `Object.Equals(Object)`, a strongly typed `Equals(R? other)`, an override of `GetHashCode`, `operator ==` and `operator !=`, a `ToString` override, and, for `record class`, a copy constructor and a virtual `Clone` method used by `with` expressions.

**Beginner-Friendly Explanation**
A record is a **type that behaves like a value for equality purposes**. Two `Person` records with the same name and age are equal, even if they are different objects. The compiler writes all the boring equality code for you.

---

### Purposes

- To model data-centric types with minimal boilerplate.
- To get **value equality** without manual `Equals`/`GetHashCode` overrides.
- To support **immutable data models** with concise syntax.
- To enable **non-destructive mutation** via `with` expressions.
- To provide readable `ToString` output automatically.

---

### Syntax Rules and Structure

```
// Record class (reference type)
public record Person(string FirstName, string LastName);

// Record struct (value type, C# 10+)
public record struct Point(int X, int Y);

// Record with explicit members
public record Employee(string FirstName, string LastName)
{
    public int Id { get; init; }
}
```

**Component breakdown**

- `record` — The modifier (equivalent to `record class`).
- `record struct` — Declares a value-type record.
- `(parameters)` — Positional parameters that generate properties and a primary constructor.
- `{ ... }` — Optional body for additional members.

#### Syntax Rules

- `record` alone declares a `record class` (reference type).
- `record struct` declares a value-type record.
- Positional parameters generate public `init`-only properties for `record class` and `readonly record struct`; read-write properties for `record struct`.
- The compiler synthesizes equality, `ToString`, and `Deconstruct` (for positional records).

#### Constraints and Limitations

- Records cannot inherit from classes (except `object`), and classes cannot inherit from records.
- `record struct` types cannot be abstract and do not support inheritance.
- Records are not suitable as Entity Framework Core entity types because EF Core relies on reference equality.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Basic Record Class

```csharp
using System;

public record Person(string FirstName, string LastName);

class Program
{
    static void Main()
    {
        var p1 = new Person("Nancy", "Davolio");
        var p2 = new Person("Nancy", "Davolio");

        Console.WriteLine(p1 == p2);          // True (value equality)
        Console.WriteLine(p1);                 // Person { FirstName = Nancy, LastName = Davolio }
    }
}
```

**Expected Output**

```
True
Person { FirstName = Nancy, LastName = Davolio }
```

**Why this output occurs**

- The compiler-generated `==` operator and `Equals` compare all properties; both records hold the same values, so they are equal. `ToString` is also compiler-generated.

---

#### Example 2: Record Struct

```csharp
using System;

public record struct Coordinate(double Latitude, double Longitude);

class Program
{
    static void Main()
    {
        var c1 = new Coordinate(51.5, -0.1);
        var c2 = new Coordinate(51.5, -0.1);

        Console.WriteLine(c1 == c2);   // True (value equality)
        Console.WriteLine(c1);         // Coordinate { Latitude = 51.5, Longitude = -0.1 }
    }
}
```

**Expected Output**

```
True
Coordinate { Latitude = 51.5, Longitude = -0.1 }
```

**Why this output occurs**

- `record struct` provides value equality just like `record class`, but instances are value types.

---

### Real-World Cases

- **API DTOs** — `public record CustomerDto(int Id, string Name);`
- **Domain value objects** — `public record Money(decimal Amount, string Currency);`
- **Configuration snapshots** — `public record AppSettings(string ConnectionString, int Timeout);`

---

### References

- Records — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/record
- C# record types — Microsoft Learn - https://learn.microsoft.com/en-au/Dotnet/csharp/fundamentals/types/records
- Apply records for immutable data models in C# — Training - https://learn.microsoft.com/sk-sk/training/modules/implement-enum-struct-record-types/4-apply-records-immutable-data-models-csharp

---

## 2. Record Declaration

### Definitions

**Core Definition**
Record declaration is the syntax that introduces a record type, specifying its kind (`record class` or `record struct`), name, positional parameters (if any), and additional members.

**Technical Definition**
The syntax for a record declaration follows the general form: `attributes? class_modifier* 'partial'? ('record' | 'record struct') identifier type_parameter_list? record_base? type_parameter_constraints_clause* record_body`. The `record` keyword may be applied to a class or struct declaration. The positional parameter list, when present, defines a primary constructor and generates positional properties. The record body may contain additional fields, properties, methods, and other members.

**Beginner-Friendly Explanation**
Declaring a record is **writing a concise data type**. You can declare it in one line with positional parameters, or add a body with extra members.

---

### Purposes

- To introduce a data-centric type with minimal ceremony.
- To specify whether the record is a reference type (`record class`) or value type (`record struct`).
- To define positional properties and a primary constructor in one line.
- To add custom members (methods, additional properties, validation).

---

### Syntax Rules and Structure

```
[attributes] [modifiers] [partial] record [class] Name(parameters) [: baseRecord]
{
    // additional members
}
```

**Component breakdown**

- `record` — The modifier.
- `[class]` — Optional; `record` alone means `record class`.
- `Name` — The record identifier.
- `(parameters)` — Optional positional parameters.
- `: baseRecord` — Optional base record (for `record class` inheritance).
- `{ ... }` — Optional body for additional members.

#### Syntax Rules

- `record` alone is equivalent to `record class` (reference type).
- `record struct` declares a value-type record.
- Positional parameters generate properties and a primary constructor.
- Records can be `sealed`, `abstract`, or `partial`.

#### Constraints and Limitations

- `record struct` cannot be `abstract` and cannot inherit from other types.
- Records cannot inherit from non-record classes.

---

### Annotated Code Examples and Expected Outputs

#### Example: Record Declaration with Additional Members

```csharp
using System;

public record Person(string FirstName, string LastName)
{
    public string FullName => $"{FirstName} {LastName}";
    public int Age { get; init; }
}

class Program
{
    static void Main()
    {
        var p = new Person("Nancy", "Davolio") { Age = 30 };
        Console.WriteLine(p.FullName);   // Nancy Davolio
        Console.WriteLine(p);             // Person { FirstName = Nancy, LastName = Davolio, Age = 30 }
    }
}
```

**Expected Output**

```
Nancy Davolio
Person { FirstName = Nancy, LastName = Davolio, Age = 30 }
```

**Why this output occurs**

- The positional parameters generate `FirstName` and `LastName`; the body adds `FullName` and `Age`.

---

### Real-World Cases

- **All record definitions** — The declaration syntax is universal.
- **Adding validation** — Custom constructors or property initializers inside the record body.

---

### References

- Records — C# feature specifications - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/record
- Records (C# reference) — Declaration syntax - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/record

---

## 3. Record Structs (C# 10)

### Definitions

**Core Definition**
A `record struct` is a value-type record (introduced in C# 10) that provides the same value equality, `with` expression support, and readable `ToString` as a `record class`, but with struct semantics (stack allocation, copy on assignment).

**Technical Definition**
The `record struct` syntax defines a value type. Positional properties in a `record struct` are **read-write by default**; adding the `readonly` modifier (`readonly record struct`) makes them `init`-only. A `record struct` has a compiler-generated parameterless constructor that sets all fields to their default values. Unlike `record class`, `record struct` does not support inheritance (it is implicitly sealed) and does not have a synthesized copy constructor (values are copied on assignment).

**Beginner-Friendly Explanation**
A `record struct` is a **lightweight record** that lives on the stack. It is perfect for small values like coordinates or measurements, where copying is cheap and you do not need inheritance.

---

### Purposes

- To provide value-type records for small, self-contained data.
- To avoid heap allocations for frequently copied values.
- To combine struct performance with record conveniences (equality, `ToString`, `with`).
- To offer mutable value records (default) or immutable ones (`readonly record struct`).

---

### Syntax Rules and Structure

```
public record struct Point(int X, int Y);
public readonly record struct Temperature(double Celsius);
```

**Component breakdown**

- `record struct` — Declares a value-type record.
- `readonly` — Makes positional properties `init`-only.

#### Syntax Rules

- `record struct` properties are read-write by default.
- `readonly record struct` properties are `init`-only.
- `record struct` cannot inherit and cannot be `abstract`.
- A parameterless constructor is synthesized for `record struct`.

#### Constraints and Limitations

- No inheritance support.
- No copy constructor synthesized (assignment copies the value).
- `with` expressions still work but perform value copies.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Mutable Record Struct

```csharp
using System;

public record struct Point(int X, int Y);

class Program
{
    static void Main()
    {
        var p = new Point(3, 4);
        p.X = 10;   // mutable by default
        Console.WriteLine(p);   // Point { X = 10, Y = 4 }
    }
}
```

**Expected Output**

```
Point { X = 10, Y = 4 }
```

**Why this output occurs**

- `record struct` positional properties are read-write; `X` can be reassigned.

---

#### Example 2: Readonly Record Struct

```csharp
using System;

public readonly record struct Temperature(double Celsius)
{
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

- `readonly record struct` makes properties `init`-only; `Fahrenheit` is computed.

---

### Real-World Cases

- **Geometric primitives** — `record struct Point(double X, double Y)`.
- **Measurements** — `readonly record struct Temperature(double Celsius)`.
- **Money** — `readonly record struct Money(decimal Amount, string Currency)`.

---

### References

- `record struct` — Records (C# reference) - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/record
- C# record structs — Microsoft Learn (search result) - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/record

---

## 4. Positional Records

### Definitions

**Core Definition**
A positional record is a record declared with parameters on the type declaration itself, which the compiler converts into a primary constructor, public positional properties, and a `Deconstruct` method.

**Technical Definition**
When you declare a primary constructor on a record, the compiler generates public automatically implemented properties for each positional parameter. For `record class` and `readonly record struct`, these properties are `init`-only; for `record struct`, they are read-write. The compiler also generates a primary constructor whose parameters match the positional parameters, and a `Deconstruct` method with an `out` parameter for each positional parameter. Positional records can be deconstructed like tuples.

**Beginner-Friendly Explanation**
Positional records let you **define data in one line**: `public record Person(string FirstName, string LastName);` creates a constructor, two properties, and the ability to deconstruct the record into its parts.

---

### Purposes

- To reduce boilerplate for data-centric types.
- To provide a primary constructor and properties in a single declaration.
- To enable deconstruction of records into their positional values.
- To support positional pattern matching in `switch` expressions.

---

### Syntax Rules and Structure

```
public record Person(string FirstName, string LastName);
// Deconstruction:
var (first, last) = person;
```

**Component breakdown**

- `(FirstName, LastName)` — Positional parameters.
- Generated members: primary constructor, properties, `Deconstruct`.

#### Syntax Rules

- Positional parameters generate public properties.
- A `Deconstruct` method is synthesized for positional records.
- Deconstruction can be used to extract values.

#### Constraints and Limitations

- Deconstruction requires the record to have a primary constructor (positional parameters).

---

### Annotated Code Examples and Expected Outputs

#### Example: Deconstructing a Positional Record

```csharp
using System;

public record Point(int X, int Y);

class Program
{
    static void Main()
    {
        var point = new Point(3, 4);
        var (x, y) = point;
        Console.WriteLine($"X = {x}, Y = {y}");   // X = 3, Y = 4
    }
}
```

**Expected Output**

```
X = 3, Y = 4
```

**Why this output occurs**

- The compiler-generated `Deconstruct` method extracts `X` and `Y`.

---

### Real-World Cases

- **DTOs** — `public record CustomerDto(int Id, string Name);`
- **Value objects** — `public record Money(decimal Amount, string Currency);`
- **Pattern matching** — Using positional patterns in `switch` expressions.

---

### References

- Positional syntax for property and field definition — Records (C# reference) - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/record#positional-syntax-for-property-and-field-definition
- Deconstructing tuples and other types — C# - https://learn.microsoft.com/en-us/dotnet/csharp/fundamentals/functional/deconstruct

---

## 5. `with` Expression

### Definitions

**Core Definition**
A `with` expression creates a **new record instance** that is a copy of an existing instance, with specified properties or fields modified. It performs **non-destructive mutation**.

**Technical Definition**
A `with` expression first copies the existing record instance, then modifies any properties or fields specified in the expression using object initializer syntax. For `record class` types, the compiler synthesizes a virtual `Clone` method and a copy constructor; the `with` expression calls the clone method and then sets the modified properties. The result is a **shallow copy**: reference-type properties are copied by reference, so the original and the copy share the same referenced objects. For `record struct`, the values are copied on assignment. Computed properties should be calculated on access, not during initialization, to avoid incorrect values in the copy.

**Beginner-Friendly Explanation**
`with` is **“give me a copy with this change.”** The original record stays untouched; you get a new record with the modification.

---

### Purposes

- To perform non-destructive mutation on records.
- To create modified copies without mutating the original.
- To support immutable data workflows.
- To simplify “update one property” scenarios.

---

### Syntax Rules and Structure

```
var modified = original with { Property = newValue };
```

**Component breakdown**

- `original` — The record instance to copy.
- `with` — The non-destructive mutation keyword.
- `{ Property = newValue }` — The properties to change.

#### Syntax Rules

- The left operand must be a record (class or struct) or a struct/anonymous type (C# 10+).
- The specified properties must have `init` or `set` accessors.
- The result is a shallow copy.

#### Constraints and Limitations

- Reference-type properties are shared between the original and the copy.
- Computed properties should be computed on access, not cached during initialization.

---

### Annotated Code Examples and Expected Outputs

#### Example: Using `with` to Create a Modified Copy

```csharp
using System;

public record Person(string FirstName, string LastName);

class Program
{
    static void Main()
    {
        var p1 = new Person("Nancy", "Davolio");
        var p2 = p1 with { LastName = "Smith" };

        Console.WriteLine(p1);   // Person { FirstName = Nancy, LastName = Davolio }
        Console.WriteLine(p2);   // Person { FirstName = Nancy, LastName = Smith }
    }
}
```

**Expected Output**

```
Person { FirstName = Nancy, LastName = Davolio }
Person { FirstName = Nancy, LastName = Smith }
```

**Why this output occurs**

- `with` copies `p1` and changes `LastName`; `p1` remains unchanged.

---

### Real-World Cases

- **State management** — Updating a record in a Redux-style store.
- **DTO updates** — Creating a modified DTO for an API response.
- **Configuration** — Deriving new settings from a base configuration.

---

### References

- The `with` expression — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/with-expression
- Nondestructive mutation — Records (C# reference) - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/record#nondestructive-mutation

---

## 6. Record Equality

### Definitions

**Core Definition**
Record equality is **value-based**: two record instances are equal if they are of the **same runtime type** and store the **same values** in all their fields and auto-implemented properties.

**Technical Definition**
For types with the `record` modifier (`record class`, `record struct`, and `readonly record struct`), two objects are equal if they are of the same type and store the same values. The compiler synthesizes: an override of `Object.Equals(Object)`, a strongly typed `Equals(R? other)`, an override of `GetHashCode`, and `operator ==`/`operator !=`. In an inheritance hierarchy (`record class`), equality requires the **runtime types** to match, enforced by a synthesized `EqualityContract` property. The synthesized `GetHashCode` combines the hash codes of all data members.

**Beginner-Friendly Explanation**
Two records are equal **if they have the same type and the same data**. It is like comparing two identical forms: if every field matches, the forms are equal.

---

### Purposes

- To provide intuitive value equality without manual overrides.
- To enable records to be used as dictionary keys and in hash sets.
- To support equality in inheritance hierarchies (runtime type must match).
- To reduce bugs from reference-equality confusion.

---

### Syntax Rules and Structure

```
// Equality is provided automatically
bool areEqual = record1 == record2;
bool alsoEqual = record1.Equals(record2);
```

**Component breakdown**

- `==` — Value equality operator.
- `.Equals` — Strongly typed equality method.

#### Syntax Rules

- Equality requires the runtime types to match.
- All fields and auto-implemented properties are compared.
- `GetHashCode` is consistent with `Equals`.

#### Constraints and Limitations

- Reference-type properties are compared by reference (shallow equality).
- Collections are compared by reference unless you override `Equals`.
- EF Core requires reference equality and is not compatible with record entity types.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Value Equality of Records

```csharp
using System;

public record Person(string FirstName, string LastName);

class Program
{
    static void Main()
    {
        var p1 = new Person("Nancy", "Davolio");
        var p2 = new Person("Nancy", "Davolio");
        var p3 = new Person("Nancy", "Smith");

        Console.WriteLine(p1 == p2);   // True
        Console.WriteLine(p1 == p3);   // False
    }
}
```

**Expected Output**

```
True
False
```

**Why this output occurs**

- `p1` and `p2` have the same values, so they are equal; `p3` differs.

---

#### Example 2: Equality in an Inheritance Hierarchy

```csharp
using System;

public abstract record Person(string FirstName, string LastName);
public record Teacher(string FirstName, string LastName, int Grade) : Person(FirstName, LastName);
public record Student(string FirstName, string LastName, int Grade) : Person(FirstName, LastName);

class Program
{
    static void Main()
    {
        Person teacher = new Teacher("Nancy", "Davolio", 3);
        Person student = new Student("Nancy", "Davolio", 3);

        Console.WriteLine(teacher == student);   // False (different runtime types)
    }
}
```

**Expected Output**

```
False
```

**Why this output occurs**

- Although both are `Person` variables, their runtime types (`Teacher` and `Student`) differ, so they are not equal.

---

### Real-World Cases

- **DTO comparison** — Checking whether two API responses carry the same data.
- **Caching** — Using records as cache keys.
- **Testing** — Asserting that two records have the same values.

---

### References

- Value equality — Records (C# reference) - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/record#value-equality
- Equality in inheritance hierarchies — Records (C# reference) - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/record#equality-in-inheritance-hierarchies

---

## 7. Record Immutability

### Definitions

**Core Definition**
Record immutability means that a record’s positional properties (for `record class` and `readonly record struct`) are **`init`-only**, so they can only be assigned during object construction or via a `with` expression.

**Technical Definition**
A positional `record class` and a positional `readonly record struct` declare **init-only properties**. A positional `record struct` declares **read-write properties**. You can override these defaults by declaring properties explicitly. Init-only properties provide **shallow immutability**: after initialization, you cannot change the value of value-type properties or the reference of reference-type properties, but the data that a reference-type property refers to remains mutable. Records are primarily intended for immutable data models, though they can be mutable if required.

**Beginner-Friendly Explanation**
Records are **immutable by default** (for `record class` and `readonly record struct`). Once you create a record, you cannot change its properties—you must create a modified copy with `with`. However, if a property holds a reference to a mutable object (like an array), the object’s contents can still be changed.

---

### Purposes

- To enforce immutability and thread safety.
- To ensure hash codes remain stable for dictionary keys.
- To support functional-style non-destructive updates.
- To prevent accidental state mutation.

---

### Syntax Rules and Structure

```
// Immutable (init-only) positional properties
public record Person(string FirstName, string LastName);

// Mutable positional record struct
public record struct Point(int X, int Y);   // X, Y are read-write
```

**Component breakdown**

- `record class` — Positional properties are `init`-only.
- `readonly record struct` — Positional properties are `init`-only.
- `record struct` — Positional properties are read-write.

#### Syntax Rules

- Init-only properties can be assigned in the constructor or object initializer.
- `with` expressions create modified copies.
- Shallow immutability: reference-type contents remain mutable.

#### Constraints and Limitations

- Immutability is shallow; deep immutability requires immutable member types.
- `record struct` is mutable by default; use `readonly record struct` for immutability.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Immutable Record Class

```csharp
using System;

public record Person(string FirstName, string LastName);

class Program
{
    static void Main()
    {
        var p = new Person("Nancy", "Davolio");
        // p.FirstName = "Bob"; // Compile error: init-only
        var p2 = p with { FirstName = "Bob" };
        Console.WriteLine(p2);   // Person { FirstName = Bob, LastName = Davolio }
    }
}
```

**Expected Output**

```
Person { FirstName = Bob, LastName = Davolio }
```

**Why this output occurs**

- `FirstName` is `init`-only; it cannot be changed after construction, but `with` creates a modified copy.

---

#### Example 2: Shallow Immutability

```csharp
using System;

public record Container(int[] Data);

class Program
{
    static void Main()
    {
        var c1 = new Container(new[] { 1, 2, 3 });
        var c2 = c1 with { };
        c2.Data[0] = 99;   // allowed: reference content is mutable
        Console.WriteLine(c1.Data[0]);   // 99
    }
}
```

**Expected Output**

```
99
```

**Why this output occurs**

- `Data` is a reference-type property; the reference is init-only, but the array contents are mutable.

---

### Real-World Cases

- **Immutable configuration** — `public record AppSettings(string ConnectionString);`
- **Event sourcing** — Immutable event payloads.
- **Thread-safe data** — Records shared across threads.

---

### References

- Immutability — Records (C# reference) - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/record#immutability
- `init` accessor — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/init

---

## 8. Record Inheritance

### Definitions

**Core Definition**
Record inheritance allows a `record class` to inherit from another `record class`, enabling hierarchies of related data types while preserving value equality, `with` expression support, and readable formatting. `record struct` types **cannot** participate in inheritance.

**Technical Definition**
A `record class` can inherit from another `record class`. However, a record cannot inherit from a class, and a class cannot inherit from a record. The derived record declares positional parameters for all parameters in the base record’s primary constructor; the base record declares and initializes those properties. Equality in a record hierarchy requires the **runtime types** to match, enforced by a synthesized `EqualityContract` property. `with` expressions on derived records return the same runtime type as the operand, and properties of the compile-time type can be set. Starting with C# 15, a `record class` can use the `closed` modifier to restrict direct derivation to its declaring assembly.

**Beginner-Friendly Explanation**
Record inheritance lets you **build data hierarchies**. A `Teacher` record can inherit from a `Person` record and add a `Grade` property. Equality checks the full runtime type, so a `Teacher` is never equal to a `Student`, even if their base data matches.

---

### Purposes

- To model related data types in a hierarchy.
- To share common data members while adding specialized ones.
- To maintain value equality across the hierarchy.
- To support `with` expressions on derived records.

---

### Syntax Rules and Structure

```
public abstract record Person(string FirstName, string LastName);
public record Teacher(string FirstName, string LastName, int Grade)
    : Person(FirstName, LastName);
```

**Component breakdown**

- `: Person(FirstName, LastName)` — Calls the base record’s primary constructor.
- Derived record adds `Grade`.

#### Syntax Rules

- Derived records must declare all base primary constructor parameters.
- Equality requires the runtime types to match.
- `with` on a derived record returns the derived type.

#### Constraints and Limitations

- Only `record class` supports inheritance.
- `record struct` is implicitly sealed.
- `record class` cannot inherit from non-record classes.

---

### Annotated Code Examples and Expected Outputs

#### Example: Record Inheritance with Positional Parameters

```csharp
using System;

public abstract record Person(string FirstName, string LastName);
public record Teacher(string FirstName, string LastName, int Grade)
    : Person(FirstName, LastName);

class Program
{
    static void Main()
    {
        var teacher = new Teacher("Nancy", "Davolio", 3);
        Console.WriteLine(teacher);   // Teacher { FirstName = Nancy, LastName = Davolio, Grade = 3 }
    }
}
```

**Expected Output**

```
Teacher { FirstName = Nancy, LastName = Davolio, Grade = 3 }
```

**Why this output occurs**

- `Teacher` inherits `FirstName` and `LastName` from `Person` and adds `Grade`.

---

### Real-World Cases

- **Domain hierarchies** — `Person` → `Teacher`, `Student`.
- **Event hierarchies** — `Event` → `UserCreated`, `OrderPlaced`.
- **API response types** — `Response` → `SuccessResponse`, `ErrorResponse`.

---

### References

- Inheritance — Records (C# reference) - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/record#inheritance
- Positional parameters in derived record types — Records (C# reference) - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/record#positional-parameters-in-derived-record-types

---

## 9. Record Best Practices

### Definitions

**Core Definition**
Record best practices are the design guidelines that ensure records are used **appropriately, safely, and consistently**, especially when choosing between `record class`, `record struct`, and ordinary classes or structs.

**Technical Definition**
Best practices for records derive from Microsoft’s guidance on choosing between tuples, records, structs, and classes. Key recommendations include: **use records when the type’s primary role is storing data and two instances with the same values should be equal**; **use `record class` when you need inheritance or when copying on assignment would be expensive**; **use `record struct` for small, self-contained values where copy semantics and stack allocation are beneficial**; **avoid mutable `record struct` when possible**; **do not use records as Entity Framework Core entity types** (EF Core relies on reference equality); and **compute derived properties on access, not during initialization**, to ensure correctness with `with` expressions.

**Beginner-Friendly Explanation**
Best practices help you **pick the right record flavour**. Use `record class` for DTOs and value objects that need inheritance; use `record struct` for tiny values like coordinates; avoid records for mutable entities with identity; and be careful with computed properties in `with` expressions.

---

### Purposes

- To choose the correct record kind for the scenario.
- To avoid misusing records where classes or structs are better.
- To ensure correct behaviour with `with` expressions and equality.
- To maintain performance and memory efficiency.
- To align with .NET design conventions.

---

### Guidelines and Structure

#### General Guidelines

```
// Use record class for DTOs and value objects that need inheritance
public record CustomerDto(int Id, string Name);

// Use record struct for small, copy-friendly values
public readonly record struct Point(double X, double Y);

// Avoid records for EF Core entities (reference equality required)
// Compute derived properties on access, not at initialization
public record Point(int X, int Y)
{
    public double Distance => Math.Sqrt(X * X + Y * Y);
}
```

**Component breakdown**

- **`record class`** — Data-centric reference types with inheritance.
- **`record struct`** — Small value types with copy semantics.
- **Avoid EF Core entities** — Use classes for entities with identity.
- **Computed properties on access** — Ensures correctness with `with`.

#### Rules and Constraints

- **Do** use records when data is the primary concern and value equality is desired.
- **Do** use `record class` when inheritance or large data is involved.
- **Do** use `record struct` for small, self-contained values.
- **Do** compute derived properties on access.
- **Do not** use records as EF Core entity types.
- **Do not** use mutable `record struct` unless mutation is intentional.
- **Do not** rely on records for behaviour-heavy types; use classes.

---

### Annotated Code Examples and Expected Outputs

#### Example: Choosing Between `record class` and `record struct`

```csharp
using System;

// Record class for a DTO with inheritance potential
public record CustomerDto(int Id, string Name);

// Record struct for a small geometric value
public readonly record struct Point(double X, double Y);

class Program
{
    static void Main()
    {
        var customer = new CustomerDto(1, "Acme");
        var point = new Point(3, 4);
        Console.WriteLine(customer);   // CustomerDto { Id = 1, Name = Acme }
        Console.WriteLine(point);      // Point { X = 3, Y = 4 }
    }
}
```

**Expected Output**

```
CustomerDto { Id = 1, Name = Acme }
Point { X = 3, Y = 4 }
```

**Why this output occurs**

- `record class` is used for the DTO; `record struct` is used for the small value.

---

### Real-World Cases

- **API layer** — DTOs as `record class`.
- **Domain primitives** — `record struct` for coordinates, money, temperature.
- **Configuration** — Immutable `record class` for settings.

---

### References

- When to use records — C# record types - https://learn.microsoft.com/en-au/Dotnet/csharp/fundamentals/types/records#when-to-use-records
- Choose between tuples, records, structs, and classes — C# tutorial - https://learn.microsoft.com/en-us/dotnet/csharp/fundamentals/tutorials/choosing-types
- Records — C# reference (best practices) - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/record

---

## Deprecated, Unsafe, Implementation-Dependent, and Version-Specific Notes

| Feature | Status / Note |
|---------|---------------|
| Records (C# 9) | Introduced in **C# 9.0**; `record class` is the default. |
| Record structs (C# 10) | Introduced in **C# 10**; value-type records. |
| `with` expressions | Introduced with records in C# 9; extended to structs and anonymous types in C# 10. |
| Positional records | Available for both `record class` and `record struct`. |
| Record inheritance | Only `record class` supports inheritance; `record struct` is sealed. |
| `closed` modifier (C# 15) | Restricts derivation to the declaring assembly; version-specific. |
| EF Core entity types | **Not recommended**; EF Core relies on reference equality. |
| Shallow immutability | Reference-type contents remain mutable; deep immutability requires immutable member types. |

---

## References (Consolidated)

- Records — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/record
- C# record types — Microsoft Learn - https://learn.microsoft.com/en-au/Dotnet/csharp/fundamentals/types/records
- Apply records for immutable data models in C# — Training - https://learn.microsoft.com/sk-sk/training/modules/implement-enum-struct-record-types/4-apply-records-immutable-data-models-csharp
- Use record types tutorial — C# - https://learn.microsoft.com/en-us/dotnet/csharp/tutorials/records
- The `with` expression — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/with-expression
- Value equality — Records (C# reference) - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/record#value-equality
- Immutability — Records (C# reference) - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/record#immutability
- Inheritance — Records (C# reference) - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/record#inheritance
- Positional syntax for property and field definition — Records (C# reference) - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/record#positional-syntax-for-property-and-field-definition
- Deconstructing tuples and other types — C# - https://learn.microsoft.com/en-us/dotnet/csharp/fundamentals/functional/deconstruct
- Choose between tuples, records, structs, and classes — C# tutorial - https://learn.microsoft.com/en-us/dotnet/csharp/fundamentals/tutorials/choosing-types
- Records — C# feature specifications - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/record

---