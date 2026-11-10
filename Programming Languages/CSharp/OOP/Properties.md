# C# Properties — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
A **property** in C# is a member that provides a flexible mechanism to read, write, or compute the value of a data field. To code that uses an object, a property appears to be a field; to the class that implements it, a property is one or two code blocks called **accessors** (`get` and `set`). Properties are the primary tool for encapsulation in C#, letting a class expose data safely while retaining control over how that data is read, written, or computed.

**Technical Definition**
A property is a member that provides a flexible mechanism to read, write, or compute the value of a data field. Properties appear as public data members, but they are implemented as special methods called accessors. A property definition contains declarations for a `get` accessor and a `set` (or `init`) accessor that retrieve and assign the value of the property. The `get` accessor is used to return the property value, and the `set` accessor is used to assign a new value. These accessors can have different access levels. In C# 9 and later, the `init` keyword defines an accessor that assigns a value only during object construction. In C# 11 and later, the `required` modifier indicates that a field or property must be initialized by an object initializer. In C# 14, the `field` keyword allows access to the compiler-synthesized backing field without declaring a separate field. Indexers are a special kind of property whose accessors take parameters, enabling instances of a class or struct to be indexed like arrays.

**Beginner-Friendly Explanation**
A property is a **smart field**. From the outside, it looks and feels like a variable: you can read it (`person.Name`) or assign to it (`person.Age = 30`). Inside the class, however, the property is backed by two small methods—a **getter** and a **setter**—that run whenever the value is read or written. This lets the class validate data, compute values on the fly, or make a value read-only. Properties are how C# keeps data safe without giving up the convenience of field-style syntax. Auto-implemented properties are the short form: the compiler creates the hidden backing field for you. The `init` accessor lets you set a value during object creation but not afterward, and `required` forces callers to set the property. Indexers let your objects be accessed with `[ ]`, just like arrays.

---

### Key Characteristics

- **Field-like syntax, method-like power** — Properties are accessed like fields but execute code on read/write.
- **Two accessors** — `get` (read) and `set` (write); since C# 9, `init` (write-once during construction).
- **Auto-implemented properties** — The compiler generates a hidden backing field; no explicit field needed.
- **Accessor accessibility** — `get` and `set` can have different access levels (e.g., public `get`, private `set`).
- **Read-only and write-only** — A property with only `get` is read-only; only `set` is write-only.
- **Computed properties** — Properties can compute values from other data; expression-bodied syntax (`=>`) makes this concise.
- **`required` members** — Since C# 11, `required` forces callers to initialize the property in an object initializer.
- **`field` keyword** — Since C# 14, access the synthesized backing field directly inside an accessor.
- **Static properties** — Properties can belong to the type rather than to an instance.
- **Indexers** — Properties with parameters, enabling `obj[0]`-style access.

---

### Prerequisites

- Basic understanding of C# classes, fields, and methods.
- Familiarity with access modifiers (`public`, `private`, `protected`, `internal`).
- Knowledge of `readonly` fields and `readonly struct` (for `init`).
- Awareness of object initializers and object creation with `new`.
- Basic understanding of expression-bodied members (`=>`).

---

### Related Programming Areas

- **Encapsulation** — Properties are the primary tool for hiding implementation details and exposing safe data access.
- **Object-oriented programming** — Properties model the state of objects while enforcing invariants.
- **Serialization and ORM** — Properties are the standard surface for JSON serialization and database mapping.
- **Data binding** — UI frameworks bind to properties (and rely on `INotifyPropertyChanged`).
- **API design** — Property design guidelines shape public contracts in .NET libraries.

---

### Core Concepts / Features

1. Properties (Overarching Concept)
2. Auto-Implemented Properties
3. Property Getters
4. Property Setters
5. `init` Accessor (C# 9)
6. `required` Members (C# 11)
7. Read-Only Properties
8. Write-Only Properties
9. Computed Properties
10. Expression-Bodied Properties
11. Static Properties
12. Indexers
13. Field-Backed Properties (C# 14)
14. Property Best Practices

Each concept follows a uniform structure: definitions, purposes, syntax rules and structure, annotated code examples with expected outputs, real-world cases, and references.

---

## 1. Properties (Overarching Concept)

### Definitions

**Core Definition**
A property is a member that provides a flexible mechanism to read, write, or compute the value of a data field, appearing to callers as a public data member while being implemented as accessor methods.

**Technical Definition**
A property is a member that provides a flexible mechanism to read, write, or compute the value of a data field. Properties appear as public data members, but they are implemented as special methods called accessors. The syntax for properties is a natural extension to fields: a field defines a storage location, while a property defines accessors that control how that storage is read and written. A property definition contains declarations for a `get` and `set` accessor that retrieves and assigns the value of that property. Accessors can have different accessibility levels, enabling scenarios such as public read access and private write access.

**Beginner-Friendly Explanation**
A property is a **field with a bodyguard**. You access it like a field, but behind the scenes the getter and setter run code that can validate, compute, or restrict access.

---

### Purposes

- To encapsulate data access behind controlled `get`/`set` logic.
- To provide field-like syntax with method-like flexibility.
- To enable validation, computation, and lazy initialization.
- To support different accessibility for reading and writing.
- To expose data safely in APIs without public fields.

---

### Syntax Rules and Structure

```
[access modifier] [modifiers] type PropertyName
{
    get { return backingField; }
    set { backingField = value; }
}
```

**Component breakdown**

- `access modifier` — `public`, `private`, `protected`, `internal`, etc.
- `modifiers` — `static`, `virtual`, `abstract`, `override`, `sealed`, etc.
- `type` — The property’s type.
- `PropertyName` — The property identifier (PascalCase by convention).
- `get` — Accessor that returns the value.
- `set` — Accessor that assigns the value; `value` is the implicit parameter.
- `init` — Accessor (C# 9+) that assigns only during object construction.

#### Syntax Rules

- A property must have at least one accessor (`get`, `set`, or `init`).
- `set` and `init` accessors have an implicit parameter named `value`.
- Accessors can have different access modifiers (e.g., `public get; private set;`).
- A property cannot be `ref`-returning unless using `ref` properties (advanced).
- Expression-bodied properties (`=>`) are read-only shorthands.

#### Constraints and Limitations

- A property cannot be used as a `ref` or `out` argument.
- A property cannot be `volatile`.
- A property cannot be a `const`.
- An auto-property cannot be declared in an interface (interfaces declare signatures only).

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Basic Property with Backing Field

```csharp
using System;

public class Person
{
    private string _name;

    public string Name
    {
        get { return _name; }
        set { _name = value; }
    }
}

class Program
{
    static void Main()
    {
        var p = new Person();
        p.Name = "Alice";
        Console.WriteLine(p.Name);   // Alice
    }
}
```

**Expected Output**

```
Alice
```

**Why this output occurs**

- `Name` wraps the `_name` field; the getter returns it and the setter assigns it.

---

#### Example 2: Property with Validation

```csharp
using System;

public class Account
{
    private decimal _balance;

    public decimal Balance
    {
        get => _balance;
        set
        {
            if (value < 0)
                throw new ArgumentOutOfRangeException(nameof(value));
            _balance = value;
        }
    }
}

class Program
{
    static void Main()
    {
        var acc = new Account();
        acc.Balance = 100m;
        Console.WriteLine(acc.Balance);   // 100

        try { acc.Balance = -5m; }
        catch (ArgumentOutOfRangeException) { Console.WriteLine("Rejected"); }
    }
}
```

**Expected Output**

```
100
Rejected
```

**Why this output occurs**

- The setter validates the value before assigning it to `_balance`.

---

### Real-World Cases

- **Domain models** — `Order.Total`, `Customer.Name`.
- **DTOs** — Data-carrying properties for serialization.
- **Configuration** — Settings exposed as properties with validation.

---

### References

- Properties (C# Programming Guide) — Microsoft Learn - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/properties
- Property Design — Framework Design Guidelines - https://learn.microsoft.com/en-us/dotnet/standard/design-guidelines/property
- Properties — C# language specification - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/language-specification/classes#157-properties

---

## 2. Auto-Implemented Properties

### Definitions

**Core Definition**
An auto-implemented property (auto-property) is a property whose accessors contain only semicolons; the compiler generates a hidden backing field automatically.

**Technical Definition**
In C# 3.0 and later, auto-implemented properties make property-declaration more concise when no additional logic is required in the property accessors. When a property is specified as an automatically implemented property, a hidden backing field is automatically available for the property, and the accessors are implemented to read from and write to that backing field. The hidden backing field is inaccessible; it can be read and written only through the property accessors, even within the containing type. An auto-property shall have a `get` accessor and may optionally have a `set` or `init` accessor. In C# 6 and later, auto-properties can be initialized inline: `public string Name { get; set; } = "Jane";`.

**Beginner-Friendly Explanation**
An auto-property is a property with **no visible backing field**. The compiler creates the field behind the scenes. You get the field-like syntax without writing the boilerplate.

---

### Purposes

- To declare properties concisely when no custom logic is needed.
- To let the compiler manage the backing field.
- To support inline initialization (C# 6+).
- To enable immutable auto-properties via `get`-only or `get; init;`.
- To reduce boilerplate in DTOs and lightweight classes.

---

### Syntax Rules and Structure

```
public type PropertyName { get; set; }
public type PropertyName { get; set; } = initialValue;
public type PropertyName { get; init; }
public type PropertyName { get; }   // read-only auto-property
```

**Component breakdown**

- `{ get; set; }` — Read-write auto-property.
- `= initialValue` — Inline initializer (C# 6+).
- `{ get; init; }` — Init-only auto-property (C# 9+).
- `{ get; }` — Getter-only auto-property (read-only everywhere except the constructor).

#### Syntax Rules

- Auto-properties must have a `get` accessor; `set` or `init` is optional.
- The backing field is inaccessible and readonly if there is no `set`/`init` accessor.
- Field-targeted attributes can be applied using `[field: ...]`.
- Auto-properties cannot be declared in interfaces.

#### Constraints and Limitations

- No custom logic in accessors (use a manual property for validation).
- Getter-only auto-properties can only be assigned in the constructor.
- Init-only auto-properties can only be assigned during object construction.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Read-Write Auto-Property

```csharp
using System;

public class Person
{
    public string Name { get; set; } = "Unknown";
}

class Program
{
    static void Main()
    {
        var p = new Person();
        Console.WriteLine(p.Name);   // Unknown

        p.Name = "Alice";
        Console.WriteLine(p.Name);   // Alice
    }
}
```

**Expected Output**

```
Unknown
Alice
```

**Why this output occurs**

- The inline initializer sets `Name` to `"Unknown"`; the setter allows later changes.

---

#### Example 2: Getter-Only Auto-Property

```csharp
using System;

public class Point
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
        var p = new Point(1, 2);
        Console.WriteLine($"{p.X}, {p.Y}");   // 1, 2
        // p.X = 5; // Compile error: read-only
    }
}
```

**Expected Output**

```
1, 2
```

**Why this output occurs**

- Getter-only auto-properties can be assigned in the constructor but not afterward.

---

### Real-World Cases

- **DTOs** — Simple data carriers.
- **Configuration objects** — Properties with defaults.
- **Lightweight classes** — Immutable value-like types with getter-only auto-properties.

---

### References

- Auto-Implemented Properties (C# Programming Guide) — Microsoft Learn - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/auto-implemented-properties
- Automatically implemented properties — C# language specification - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/language-specification/classes#1574-automatically-implemented-properties

---

## 3. Property Getters

### Definitions

**Core Definition**
A getter is the `get` accessor of a property. It is a method that returns the property’s value when the property is read.

**Technical Definition**
The `get` accessor is used to return the property value. A `get` accessor corresponds to a method with no parameters and a return type equal to the property type. The `get` keyword can be used with an expression body (`get => expression;`) to provide a concise implementation. If a property has only a `get` accessor and no `set` or `init`, it is read-only.

**Beginner-Friendly Explanation**
The getter is what runs when you **read** the property. It returns the value.

---

### Purposes

- To return the property’s value on read.
- To compute values on demand.
- To enforce read-only semantics.
- To expose data safely.

---

### Syntax Rules and Structure

```
get { return expression; }
get => expression;   // expression-bodied
```

**Component breakdown**

- `get` — The accessor keyword.
- `return expression` — The value returned.
- `get => expression` — Expression-bodied getter.

#### Syntax Rules

- The getter must return a value convertible to the property type.
- A getter can have an expression body.
- A getter cannot accept parameters (except in indexers).

#### Constraints and Limitations

- Getter-only properties are read-only.
- Getters should be simple and should not throw exceptions (Framework Design Guidelines).

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Basic Getter

```csharp
using System;

public class Circle
{
    private double _radius = 2;

    public double Radius
    {
        get { return _radius; }
    }
}

class Program
{
    static void Main()
    {
        var c = new Circle();
        Console.WriteLine(c.Radius);   // 2
    }
}
```

**Expected Output**

```
2
```

**Why this output occurs**

- The getter returns `_radius`.

---

#### Example 2: Expression-Bodied Getter

```csharp
using System;

public class Rectangle
{
    public double Width { get; set; }
    public double Height { get; set; }

    public double Area => Width * Height;
}

class Program
{
    static void Main()
    {
        var r = new Rectangle { Width = 3, Height = 4 };
        Console.WriteLine(r.Area);   // 12
    }
}
```

**Expected Output**

```
12
```

**Why this output occurs**

- `Area` uses an expression-bodied getter to compute `Width * Height`.

---

### Real-World Cases

- **Computed properties** — `FullName => $"{First} {Last}"`.
- **Read-only views** — Exposing internal state safely.

---

### References

- get keyword — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/get
- Expression-bodied members — C# Programming Guide - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/statements-expressions-operators/expression-bodied-members

---

## 4. Property Setters

### Definitions

**Core Definition**
A setter is the `set` accessor of a property. It is a method that assigns the property’s value when the property is written.

**Technical Definition**
The `set` accessor is used to assign a new value. A `set` accessor corresponds to a method with a single `value` parameter of the property type and a `void` return type. The `set` keyword can be used with an expression body (`set => field = value;`) to provide a concise implementation. If a property has only a `set` accessor and no `get`, it is write-only.

**Beginner-Friendly Explanation**
The setter is what runs when you **assign** to the property. It receives the new value in the implicit `value` parameter.

---

### Purposes

- To assign a new value to the property.
- To validate incoming values.
- To enforce invariants.
- To trigger side effects (e.g., change notifications).

---

### Syntax Rules and Structure

```
set { field = value; }
set => field = value;   // expression-bodied
```

**Component breakdown**

- `set` — The accessor keyword.
- `value` — The implicit parameter holding the new value.
- `set => expression` — Expression-bodied setter.

#### Syntax Rules

- The `value` parameter is implicitly typed as the property type.
- The setter can contain validation logic.
- The setter can be expression-bodied.
- The setter can have a different access modifier than the getter.

#### Constraints and Limitations

- Write-only properties are discouraged by design guidelines.
- Setters should preserve the previous value if an exception is thrown.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Setter with Validation

```csharp
using System;

public class Person
{
    private int _age;

    public int Age
    {
        get => _age;
        set
        {
            if (value < 0)
                throw new ArgumentOutOfRangeException(nameof(value));
            _age = value;
        }
    }
}

class Program
{
    static void Main()
    {
        var p = new Person();
        p.Age = 30;
        Console.WriteLine(p.Age);   // 30
    }
}
```

**Expected Output**

```
30
```

**Why this output occurs**

- The setter validates the value before assigning it.

---

#### Example 2: Expression-Bodied Setter

```csharp
using System;

public class Location
{
    private string _name;

    public string Name
    {
        get => _name;
        set => _name = value;
    }
}

class Program
{
    static void Main()
    {
        var loc = new Location { Name = "London" };
        Console.WriteLine(loc.Name);   // London
    }
}
```

**Expected Output**

```
London
```

**Why this output occurs**

- Both accessors are expression-bodied.

---

### Real-World Cases

- **Validation** — Rejecting invalid values.
- **Change notification** — Raising `INotifyPropertyChanged`.
- **Normalization** — Trimming strings or adjusting values.

---

### References

- set keyword — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/set
- Expression-bodied members — C# Programming Guide - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/statements-expressions-operators/expression-bodied-members

---

## 5. `init` Accessor (C# 9)

### Definitions

**Core Definition**
The `init` accessor is a variant of the `set` accessor that assigns a value to the property **only during object construction**, enforcing immutability after initialization.

**Technical Definition**
In C# 9 and later, the `init` keyword defines an accessor method in a property or indexer. An init-only setter assigns a value to the property or the indexer element only during object construction. This enforces immutability, so once the object is initialized, it cannot be changed. An `init` accessor corresponds to a method with a single `value` parameter of the property type and a `void` return type; the implicit parameter is always named `value`. Init accessors can be used on auto-properties and manually implemented properties, and they can be expression-bodied. All overrides of a property shall have `init` if the base has `init`; this rule also applies to interface implementation. Init accessors are permitted on properties of `readonly` structs and on `readonly` properties.

**Beginner-Friendly Explanation**
`init` is a **setter that only works while the object is being built**. Once construction finishes, the value is locked. It gives you immutability without giving up object initializers.

---

### Purposes

- To enforce immutability after construction.
- To allow object initializers to set values during construction.
- To combine the convenience of `set` with the safety of read-only properties.
- To support immutable data models and records.

---

### Syntax Rules and Structure

```
public type PropertyName { get; init; }
public type PropertyName { get => field; init => field = value; }
```

**Component breakdown**

- `get; init;` — Auto-implemented init-only property.
- `init => field = value` — Expression-bodied init accessor.
- `init { field = value; }` — Block-bodied init accessor.

#### Syntax Rules

- `init` can only be used on instance properties.
- The `init` accessor can only be called during object construction or in `with` expressions for records.
- Overrides of `init` properties must also use `init`.
- Init-only auto-properties can be assigned in the constructor.

#### Constraints and Limitations

- `init` does not force callers to set the property; use `required` for that.
- Init accessors cannot be marked `readonly` themselves, even in `readonly` structs.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Init-Only Auto-Property

```csharp
using System;

public class Person
{
    public string Name { get; init; }
}

class Program
{
    static void Main()
    {
        var p = new Person { Name = "Alice" };
        Console.WriteLine(p.Name);   // Alice
        // p.Name = "Bob"; // Compile error: init-only
    }
}
```

**Expected Output**

```
Alice
```

**Why this output occurs**

- `Name` can be set during object initialization but not afterward.

---

#### Example 2: Init Accessor with Validation

```csharp
using System;

public class Person
{
    private int _yearOfBirth;

    public int YearOfBirth
    {
        get => _yearOfBirth;
        init
        {
            if (value < 1900 || value > DateTime.Now.Year)
                throw new ArgumentOutOfRangeException(nameof(value));
            _yearOfBirth = value;
        }
    }
}

class Program
{
    static void Main()
    {
        var p = new Person { YearOfBirth = 1984 };
        Console.WriteLine(p.YearOfBirth);   // 1984
    }
}
```

**Expected Output**

```
1984
```

**Why this output occurs**

- The `init` accessor validates the value during construction.

---

### Real-World Cases

- **Immutable DTOs** — Data that must not change after creation.
- **Records** — Positional and nominal records rely on `init` accessors.
- **Configuration snapshots** — Settings fixed at construction.

---

### References

- init keyword — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/init
- Init accessors — C# language specification - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/language-specification/classes#1573-init-accessors
- Welcome to C# 9.0 — .NET Blog (init accessors) - https://devblogs.microsoft.com/dotnet/welcome-to-csharp-9/

---

## 6. `required` Members (C# 11)

### Definitions

**Core Definition**
The `required` modifier indicates that a field or property **must be initialized by an object initializer** when a new instance of the type is created.

**Technical Definition**
The `required` modifier indicates that the field or property it applies to must be initialized by an object initializer. Any expression that initializes a new instance of the type must initialize all required members. The `required` modifier is available starting with C# 11. Required members must be at least as visible as their containing type. Required properties must have setters (`set` or `init` accessors) that are at least as visible as their containing type. Derived classes cannot hide a `required` member declared in the base class, and derived types that override a required property must include the `required` modifier. The `required` modifier cannot be applied to members of an interface, explicit interface implementations, or positional parameters on a record (though an explicit declaration for a positional property can be required).

**Beginner-Friendly Explanation**
`required` is a **“you must set this”** marker. If a class has a required property, anyone creating a new object must assign it in the object initializer. The compiler enforces this.

---

### Purposes

- To force callers to set essential properties when creating objects.
- To avoid objects in an invalid or incomplete state.
- To combine immutability (`init`) with mandatory initialization.
- To document which properties are essential.

---

### Syntax Rules and Structure

```
public required type PropertyName { get; set; }
public required type PropertyName { get; init; }
```

**Component breakdown**

- `required` — The modifier.
- `type` — The property type.
- `PropertyName` — The property identifier.
- `{ get; set; }` or `{ get; init; }` — Accessors.

#### Syntax Rules

- `required` can be applied to fields and properties in classes, structs, and records.
- The member’s setter (`set` or `init`) must be at least as visible as the containing type.
- Required members must be initialized in the object initializer or by a constructor annotated with `[SetsRequiredMembers]`.
- Required members cannot be hidden by derived classes.
- `required` is not the same as non-nullable; a required member can be set to `null` or `default` (the compiler may warn for non-nullable reference types).

#### Constraints and Limitations

- Cannot be applied to interface members or explicit interface implementations.
- Cannot be applied to positional parameters on records directly.
- Cannot be used with the `new()` generic constraint when the type has required members.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Required Property

```csharp
using System;

public class Person
{
    public required string FirstName { get; init; }
}

class Program
{
    static void Main()
    {
        var p = new Person { FirstName = "Alice" };
        Console.WriteLine(p.FirstName);   // Alice

        // var p2 = new Person(); // Compile error: FirstName is required
    }
}
```

**Expected Output**

```
Alice
```

**Why this output occurs**

- `FirstName` is `required`; the compiler forces the object initializer to set it.

---

#### Example 2: `[SetsRequiredMembers]` Constructor

```csharp
using System;
using System.Diagnostics.CodeAnalysis;

public class Person
{
    public required string FirstName { get; init; }

    [SetsRequiredMembers]
    public Person(string firstName) => FirstName = firstName;
}

class Program
{
    static void Main()
    {
        var p = new Person("Bob");
        Console.WriteLine(p.FirstName);   // Bob
    }
}
```

**Expected Output**

```
Bob
```

**Why this output occurs**

- The constructor annotated with `[SetsRequiredMembers]` satisfies the requirement; callers do not need an object initializer.

---

### Real-World Cases

- **Domain entities** — Ensuring essential fields like `Id` or `Name` are set.
- **DTOs** — Mandating key data for API payloads.
- **Configuration** — Requiring critical settings.

---

### References

- required modifier — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/required
- Required members — C# language specification - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/language-specification/classes#1571-required-members
- Properties — Required properties — Microsoft Learn - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/properties#required-properties

---

## 7. Read-Only Properties

### Definitions

**Core Definition**
A read-only property is a property with only a `get` accessor (and no `set` or `init` accessor). Its value cannot be changed after construction.

**Technical Definition**
A property that has only a `get` accessor is read-only. In the case of an auto-implemented property, the backing field is considered `readonly` and may only be assigned in the constructor of the enclosing class. A read-only property can also be implemented manually with a `get` accessor that returns a backing field or computes a value. Expression-bodied properties (`=>`) are inherently read-only.

**Beginner-Friendly Explanation**
A read-only property is **“look but don’t touch.”** You can read the value, but you cannot assign to it after the object is created.

---

### Purposes

- To expose data without allowing mutation.
- To enforce immutability.
- To protect invariants.
- To provide computed values that should not be set directly.

---

### Syntax Rules and Structure

```
public type PropertyName { get; }
public type PropertyName => expression;
public type PropertyName { get { return field; } }
```

**Component breakdown**

- `{ get; }` — Getter-only auto-property.
- `=> expression` — Expression-bodied read-only property.
- `{ get { ... } }` — Manual read-only property.

#### Syntax Rules

- Getter-only auto-properties can be assigned in the constructor.
- Expression-bodied properties cannot have setters.
- Read-only properties cannot be assigned outside the constructor (for auto-properties) or at all (for computed properties).

#### Constraints and Limitations

- If the property type is a mutable reference type, the referenced object can still be mutated even though the property is read-only.
- Read-only properties do not prevent changes to the object they reference.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Getter-Only Auto-Property

```csharp
using System;

public class Circle
{
    public double Radius { get; }

    public Circle(double radius) => Radius = radius;
}

class Program
{
    static void Main()
    {
        var c = new Circle(2);
        Console.WriteLine(c.Radius);   // 2
        // c.Radius = 3; // Compile error
    }
}
```

**Expected Output**

```
2
```

**Why this output occurs**

- `Radius` is getter-only; it is assigned in the constructor and cannot be changed afterward.

---

#### Example 2: Computed Read-Only Property

```csharp
using System;

public class Person
{
    public string FirstName { get; set; }
    public string LastName { get; set; }

    public string FullName => $"{FirstName} {LastName}";
}

class Program
{
    static void Main()
    {
        var p = new Person { FirstName = "Alice", LastName = "Smith" };
        Console.WriteLine(p.FullName);   // Alice Smith
    }
}
```

**Expected Output**

```
Alice Smith
```

**Why this output occurs**

- `FullName` is computed from the other properties and has no setter.

---

### Real-World Cases

- **Immutable value objects** — `Money`, `Point`, `Temperature`.
- **Computed properties** — `FullName`, `Area`, `IsValid`.
- **Exposed internal state** — Returning collections as `IReadOnlyList<T>`.

---

### References

- Read-only properties — Properties (C# Programming Guide) - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/properties
- Property Design — Framework Design Guidelines (get-only properties) - https://learn.microsoft.com/en-us/dotnet/standard/design-guidelines/property

---

## 8. Write-Only Properties

### Definitions

**Core Definition**
A write-only property is a property with only a `set` accessor and no `get` accessor. Its value can be assigned but not read.

**Technical Definition**
A property that has only a `set` accessor is a write-only property. Except as the target of an assignment, it is a compile-time error to reference a write-only property in an expression. The pre- and postfix `++` and `--` operators and compound assignment operators cannot be applied to write-only properties. Write-only properties are **rare** and are most commonly used to restrict access to sensitive data. The Framework Design Guidelines **prohibit** write-only properties because they are confusing and can be replaced by methods. If a property getter cannot be provided, implement the functionality as a method instead (e.g., `SetCachePath` instead of a write-only `CachePath` property).

**Beginner-Friendly Explanation**
A write-only property is **“set it but don’t read it.”** It is uncommon and generally discouraged. Use a method instead if you need write-only behavior.

---

### Purposes

- To restrict read access to sensitive data (rare).
- Historically used for password-like inputs.
- Generally **not recommended** by design guidelines.

---

### Syntax Rules and Structure

```
public type PropertyName { set { field = value; } }
public type PropertyName { set; }   // not allowed in auto-properties (must have get)
```

**Component breakdown**

- `set` — The only accessor.
- No `get` accessor.

#### Syntax Rules

- A write-only property has only a `set` accessor.
- Auto-implemented properties must have a `get` accessor; write-only auto-properties are not allowed.
- Write-only properties cannot be used in expressions other than assignment.

#### Constraints and Limitations

- **Discouraged** by Framework Design Guidelines; use methods instead.
- Cannot use `++`, `--`, or compound assignment on write-only properties.
- Cannot be read.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Write-Only Property (Not Recommended)

```csharp
using System;

public class SecretHolder
{
    private string _secret;

    public string Secret
    {
        set { _secret = value; }
    }

    public bool IsSet() => _secret != null;
}

class Program
{
    static void Main()
    {
        var h = new SecretHolder();
        h.Secret = "password";
        Console.WriteLine(h.IsSet());   // True
        // Console.WriteLine(h.Secret); // Compile error
    }
}
```

**Expected Output**

```
True
```

**Why this output occurs**

- The property can only be written; reading it is a compile-time error.

---

#### Example 2: Method Instead of Write-Only Property

```csharp
using System;

public class Cache
{
    private string _path;

    public void SetCachePath(string path) => _path = path;
    public string GetCachePath() => _path;
}

class Program
{
    static void Main()
    {
        var c = new Cache();
        c.SetCachePath("/tmp");
        Console.WriteLine(c.GetCachePath());   // /tmp
    }
}
```

**Expected Output**

```
/tmp
```

**Why this output occurs**

- The method-based approach follows the design guideline to avoid write-only properties.

---

### Real-World Cases

- **Legacy code** — Occasionally found in older APIs.
- **Sensitive data** — Rarely, write-only properties are used for secrets.

---

### References

- CA1044: Properties should not be write only — .NET code analysis - https://learn.microsoft.com/en-us/dotnet/fundamentals/code-analysis/quality-rules/ca1044
- Property Design — Framework Design Guidelines (DO NOT provide set-only properties) - https://learn.microsoft.com/en-us/dotnet/standard/design-guidelines/property

---

## 9. Computed Properties

### Definitions

**Core Definition**
A computed property is a property whose value is calculated from other data on each access, rather than stored in a backing field.

**Technical Definition**
A property can be computed from other data. For example, a `FullName` property can combine `FirstName` and `LastName`. Computed properties are typically implemented with a `get` accessor that performs the computation, often using expression-bodied syntax (`=>`). Expression-bodied properties are a concise way to create computed properties introduced in C# 6. Computed properties are read-only (no `set` accessor) unless a setter is explicitly provided.

**Beginner-Friendly Explanation**
A computed property is a **formula**. It does not store a value; it calculates one whenever you ask for it.

---

### Purposes

- To derive values from other properties or fields.
- To avoid storing redundant data.
- To keep objects consistent automatically.
- To provide calculated views of an object’s state.

---

### Syntax Rules and Structure

```
public type PropertyName => expression;
public type PropertyName { get { return expression; } }
```

**Component breakdown**

- `=> expression` — Expression-bodied computed property.
- `{ get { return expression; } }` — Block-bodied computed property.

#### Syntax Rules

- Computed properties are typically read-only (no setter).
- The computation is performed on each access.
- Expression-bodied syntax is concise for single expressions.

#### Constraints and Limitations

- Computation cost is incurred on every access; consider caching for expensive computations.
- Computed properties cannot be used as `ref` or `out` arguments.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Computed `FullName`

```csharp
using System;

public class Person
{
    public string FirstName { get; set; }
    public string LastName { get; set; }

    public string FullName => $"{FirstName} {LastName}";
}

class Program
{
    static void Main()
    {
        var p = new Person { FirstName = "Alice", LastName = "Smith" };
        Console.WriteLine(p.FullName);   // Alice Smith
    }
}
```

**Expected Output**

```
Alice Smith
```

**Why this output occurs**

- `FullName` is computed from `FirstName` and `LastName` on each access.

---

#### Example 2: Computed `IsValid`

```csharp
using System;

public class Order
{
    public decimal Total { get; set; }
    public decimal MinOrder { get; set; } = 10m;

    public bool IsValid => Total >= MinOrder;
}

class Program
{
    static void Main()
    {
        var o = new Order { Total = 15m };
        Console.WriteLine(o.IsValid);   // True
    }
}
```

**Expected Output**

```
True
```

**Why this output occurs**

- `IsValid` computes whether `Total` meets `MinOrder`.

---

### Real-World Cases

- **`FullName`** — Combining name parts.
- **`Area` / `Circumference`** — Geometry calculations.
- **`IsValid` / `IsEmpty`** — Derived state checks.

---

### References

- Computed properties — Properties (C# Programming Guide) - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/properties
- Expression-bodied members — C# Programming Guide - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/statements-expressions-operators/expression-bodied-members

---

## 10. Expression-Bodied Properties

### Definitions

**Core Definition**
An expression-bodied property is a property implemented with a single expression following `=>`, without a block body. It is a concise syntax for read-only properties and accessors.

**Technical Definition**
Expression-bodied members use the lambda expression syntax to define members that contain a single expression. A read-only property can be implemented with `public type Name => expression;`, which is equivalent to a getter that returns the expression. Property `get` and `set` accessors can also be expression-bodied: `public string Name { get => field; set => field = value; }`. Expression-bodied properties were introduced in C# 6 for getter-only properties and extended in C# 7 for accessors.

**Beginner-Friendly Explanation**
An expression-bodied property is a **one-line property**. Instead of writing `{ get { return ...; } }`, you write `=> ...`. It is short and readable.

---

### Purposes

- To reduce boilerplate for simple computed properties.
- To improve readability of small members.
- To align property syntax with lambda expressions.
- To enable concise `get` and `set` accessors.

---

### Syntax Rules and Structure

```
public type PropertyName => expression;
public type PropertyName { get => expression; set => expression; }
```

**Component breakdown**

- `=> expression` — Read-only expression-bodied property.
- `get => expression` — Expression-bodied getter.
- `set => expression` — Expression-bodied setter.

#### Syntax Rules

- A read-only expression-bodied property has no setter.
- Accessors can be individually expression-bodied.
- The expression must be a single expression (no statements).

#### Constraints and Limitations

- Cannot contain multiple statements.
- Cannot declare local variables.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Expression-Bodied Read-Only Property

```csharp
using System;

public class Circle
{
    public double Radius { get; set; }

    public double Area => Math.PI * Radius * Radius;
}

class Program
{
    static void Main()
    {
        var c = new Circle { Radius = 2 };
        Console.WriteLine(c.Area.ToString("F2"));   // 12.57
    }
}
```

**Expected Output**

```
12.57
```

**Why this output occurs**

- `Area` is a single-expression read-only property.

---

#### Example 2: Expression-Bodied Accessors

```csharp
using System;

public class Location
{
    private string _name;

    public string Name
    {
        get => _name;
        set => _name = value;
    }
}

class Program
{
    static void Main()
    {
        var loc = new Location { Name = "London" };
        Console.WriteLine(loc.Name);   // London
    }
}
```

**Expected Output**

```
London
```

**Why this output occurs**

- Both accessors use expression bodies.

---

### Real-World Cases

- **Computed properties** — `FullName`, `Area`.
- **Simple getters/setters** — Wrapping fields with no logic.

---

### References

- Expression-bodied members — C# Programming Guide - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/statements-expressions-operators/expression-bodied-members

---

## 11. Static Properties

### Definitions

**Core Definition**
A static property is a property that belongs to the **type itself** rather than to any instance. It is accessed through the type name.

**Technical Definition**
When a property declaration includes the `static` modifier, the property is said to be a static property. A static property is not associated with a specific instance, and it is a compile-time error to refer to `this` in the accessors of a static property. Static properties are accessed using the type name (`TypeName.PropertyName`). They can be read-only, read-write, or init-only, and they follow the same accessor rules as instance properties.

**Beginner-Friendly Explanation**
A static property is a **property on the class itself**, not on objects. You access it with the class name: `Math.PI` is a static property.

---

### Purposes

- To expose type-level state.
- To provide singleton-like values.
- To share data across all instances.
- To implement utility properties.

---

### Syntax Rules and Structure

```
public static type PropertyName { get; set; }
public static type PropertyName => expression;
```

**Component breakdown**

- `static` — The modifier.
- `type` — The property type.
- `PropertyName` — The identifier.
- Accessors — `get`, `set`, or `init`.

#### Syntax Rules

- Static properties cannot use `this`.
- They are accessed via the type name.
- They can be auto-implemented or manually implemented.

#### Constraints and Limitations

- Static properties are shared across all instances and threads; thread-safety must be considered.
- Cannot be `virtual`, `abstract`, or `override` in the traditional sense (static abstract members in interfaces are an advanced feature).

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Static Property

```csharp
using System;

public class Configuration
{
    public static string AppName { get; set; } = "MyApp";
}

class Program
{
    static void Main()
    {
        Console.WriteLine(Configuration.AppName);   // MyApp
    }
}
```

**Expected Output**

```
MyApp
```

**Why this output occurs**

- `AppName` belongs to `Configuration`; it is accessed via the type name.

---

### Real-World Cases

- **Constants** — `Math.PI`, `string.Empty`.
- **Singleton instances** — `Instance` property.
- **Environment settings** — `Environment.NewLine`.

---

### References

- Static and instance properties — C# language specification - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/language-specification/classes#1572-static-and-instance-properties

---

## 12. Indexers

### Definitions

**Core Definition**
An indexer is a special kind of property that enables instances of a class or struct to be indexed like arrays, using `[ ]` syntax with parameters.

**Technical Definition**
Indexers allow instances of a class or struct to be indexed just like arrays. The indexed value can be set or retrieved without explicitly specifying a type or instance member. Indexers resemble properties except that their accessors take parameters. An indexer is declared using the `this` keyword followed by a parameter list in square brackets: `public T this[int i] { get => arr[i]; set => arr[i] = value; }`. The `this` keyword defines the indexer, and the `value` keyword is the argument to the set accessor. Indexers do not require an integer index; any type of index can be used. Indexers can be overloaded, can have multiple parameters, and can be partial in partial types. Indexers cannot be auto-implemented because the compiler cannot always generate the correct storage for an indexer. The `get` accessor returns a value and the `set` accessor assigns a value.

**Beginner-Friendly Explanation**
An indexer makes your object **look like an array**. Instead of `obj.GetItem(0)`, you write `obj[0]`. It is a property with parameters.

---

### Purposes

- To enable array-like indexing on custom types.
- To model collections and lookup tables.
- To provide a natural API for accessing internal collections.
- To support multiple parameters for multidimensional indexing.

---

### Syntax Rules and Structure

```
public returnType this[parameterList]
{
    get { ... }
    set { ... }
}
```

**Component breakdown**

- `this` — The indexer keyword.
- `[parameterList]` — One or more index parameters.
- `get` — Returns the value at the index.
- `set` — Assigns the value at the index; `value` is the implicit parameter.

#### Syntax Rules

- The `this` keyword identifies the indexer.
- Indexers can have one or more parameters.
- Indexers can be overloaded.
- Indexers can be read-only (getter only) or read-write.
- Indexers can use expression-bodied syntax.

#### Constraints and Limitations

- Indexers **cannot be auto-implemented** (unlike properties).
- Indexers should be used only when the API models a collection.
- Avoid indexers with more than one parameter unless modeling a multidimensional collection.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Read-Write Indexer

```csharp
using System;

public class SampleCollection<T>
{
    private T[] arr = new T[100];

    public T this[int i]
    {
        get => arr[i];
        set => arr[i] = value;
    }
}

class Program
{
    static void Main()
    {
        var c = new SampleCollection<string>();
        c[0] = "Hello";
        Console.WriteLine(c[0]);   // Hello
    }
}
```

**Expected Output**

```
Hello
```

**Why this output occurs**

- The indexer gets and sets `arr[i]`.

---

#### Example 2: Read-Only Expression-Bodied Indexer

```csharp
using System;

public class ReadOnlySampleCollection<T>
{
    private T[] arr = new T[100];

    public T this[int i] => arr[i];
}

class Program
{
    static void Main()
    {
        var c = new ReadOnlySampleCollection<string>();
        c.arr[0] = "Hi";   // not accessible externally; simplified example
        Console.WriteLine(c[0]);   // Hi
    }
}
```

**Expected Output**

```
Hi
```

**Why this output occurs**

- The indexer is getter-only; it returns `arr[i]`.

---

### Real-World Cases

- **Custom collections** — `MyList<T>`, `Matrix`.
- **String-like types** — Accessing characters by index.
- **Configuration** — `Settings["key"]`.

---

### References

- Indexers — C# Programming Guide - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/indexers/
- Indexed Property Design — Framework Design Guidelines - https://learn.microsoft.com/en-us/dotnet/standard/design-guidelines/property#indexed-property-design

---

## 13. Field-Backed Properties (C# 14)

### Definitions

**Core Definition**
In C# 14, the `field` keyword allows a property accessor to directly access the compiler-synthesized backing field without declaring a separate field.

**Technical Definition**
In C# 14, you can add validation or other logic in the accessor for a property using the `field` keyword. The `field` keyword accesses the compiler-synthesized backing field for a property. It enables you to write a property accessor without explicitly declaring a separate backing field. This feature combines the conciseness of auto-properties with the ability to add logic in the accessor.

**Beginner-Friendly Explanation**
`field` is a **magic name for the hidden backing field**. You can use it inside an accessor to read or write the compiler-generated field, so you do not have to declare `_name` yourself.

---

### Purposes

- To add validation or logic without declaring a separate field.
- To reduce boilerplate in properties with simple logic.
- To combine auto-property convenience with custom accessor logic.
- To access the backing field in both `get` and `set`/`init` accessors.

---

### Syntax Rules and Structure

```
public type PropertyName
{
    get => field;
    set => field = value.Trim();   // field accesses the backing field
}
```

**Component breakdown**

- `field` — Contextual keyword referring to the synthesized backing field.
- `get` — Can return `field`.
- `set` / `init` — Can assign to `field`.

#### Syntax Rules

- `field` can be used in `get`, `set`, and `init` accessors.
- The backing field is synthesized by the compiler; no explicit field declaration is needed.
- `field` cannot be used outside an accessor.

#### Constraints and Limitations

- Requires **C# 14** or later.
- Cannot be used in static properties (the backing field is instance-level).

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Property with Validation Using `field`

```csharp
using System;

public class Person
{
    public string Name
    {
        get => field;
        set => field = value?.Trim() ?? "";
    }
}

class Program
{
    static void Main()
    {
        var p = new Person { Name = "  Alice  " };
        Console.WriteLine($"|{p.Name}|");   // |Alice|
    }
}
```

**Expected Output**

```
|Alice|
```

**Why this output occurs**

- The setter trims the value before assigning it to the synthesized backing field accessed via `field`.

---

### Real-World Cases

- **Validation** — Trimming or range-checking values.
- **Normalization** — Standardizing input.
- **Lazy initialization** — Computing values on first access.

---

### References

- Field-backed properties — Properties (C# Programming Guide) - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/properties#field-backed-properties
- field keyword — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/field

---

## 14. Property Best Practices

### Definitions

**Core Definition**
Property best practices are the design guidelines that ensure properties are **safe, predictable, and maintainable**, and that they express design intent clearly.

**Technical Definition**
Best practices for properties derive from the Framework Design Guidelines and the C# programming guide. Key recommendations include: **create get-only properties if callers should not change the value**; **do not provide set-only properties**; **provide sensible default values**; **allow properties to be set in any order** (even if temporary invalid states occur); **preserve the previous value if a setter throws**; **avoid throwing exceptions from getters**; **use indexers only for collection-like APIs**; and **avoid indexed properties with more than one parameter**.

**Beginner-Friendly Explanation**
Best practices are the habits that keep properties clean: **make read-only what should not change**, **do not write write-only properties**, **give sensible defaults**, **do not throw from getters**, and **only use indexers for collections**.

---

### Purposes

- To make properties intuitive and safe.
- To prevent misuse (e.g., write-only properties).
- To ensure consistent behaviour.
- To align with .NET conventions.

---

### Guidelines and Structure

#### General Guidelines

```
// DO create get-only properties if callers should not change the value
public string Id { get; }

// DO NOT provide set-only properties
// Use a method instead

// DO provide sensible default values
public string Name { get; set; } = string.Empty;

// DO allow properties to be set in any order
// DO preserve the previous value if a setter throws
// DO NOT throw exceptions from getters
// AVOID indexers with more than one parameter
```

**Component breakdown**

- **Get-only** — Read-only properties for immutable data.
- **No set-only** — Write-only properties are prohibited.
- **Defaults** — Sensible defaults for all properties.
- **Order-independent** — Allow setting in any order.
- **Setter safety** — Preserve previous value on exception.
- **Getter simplicity** — Avoid exceptions from getters.
- **Indexer usage** — Only for collection APIs; avoid multi-parameter indexers.

#### Rules and Constraints

- **Do** use get-only properties for read-only data.
- **Do** provide sensible defaults.
- **Do** preserve previous values if a setter throws.
- **Do** allow properties to be set in any order.
- **Do not** provide set-only properties; use methods instead.
- **Do not** throw exceptions from getters.
- **Do not** use indexers with more than one parameter.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Get-Only Property with Default

```csharp
using System;

public class Person
{
    public string Name { get; } = "Unknown";
    public int Age { get; set; }
}

class Program
{
    static void Main()
    {
        var p = new Person();
        Console.WriteLine(p.Name);   // Unknown
    }
}
```

**Expected Output**

```
Unknown
```

**Why this output occurs**

- `Name` is get-only with a default; `Age` is read-write.

---

#### Example 2: Safe Setter with Previous Value Preservation

```csharp
using System;

public class Account
{
    private decimal _balance;

    public decimal Balance
    {
        get => _balance;
        set
        {
            if (value < 0)
                throw new ArgumentOutOfRangeException(nameof(value));
            _balance = value;
        }
    }
}

class Program
{
    static void Main()
    {
        var acc = new Account { Balance = 100m };
        try { acc.Balance = -5m; }
        catch { }
        Console.WriteLine(acc.Balance);   // 100 (preserved)
    }
}
```

**Expected Output**

```
100
```

**Why this output occurs**

- The setter throws before assigning, so the previous value is preserved.

---

### Real-World Cases

- **API design** — Public properties follow these guidelines.
- **Domain models** — Get-only properties for identity.
- **Collections** — Indexers only on collection-like types.

---

### References

- Property Design — Framework Design Guidelines - https://learn.microsoft.com/en-us/dotnet/standard/design-guidelines/property
- Properties (C# Programming Guide) — Microsoft Learn - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/properties

---

## Deprecated, Unsafe, Implementation-Dependent, and Version-Specific Notes

| Feature | Status / Note |
|---------|---------------|
| Auto-implemented properties | Introduced in **C# 3.0**; stable. |
| `init` accessor | Introduced in **C# 9**; version-specific. |
| `required` modifier | Introduced in **C# 11**; version-specific. |
| `field` keyword (field-backed properties) | Introduced in **C# 14**; version-specific. |
| Write-only properties | **Discouraged** by Framework Design Guidelines (CA1044). |
| Auto-properties in interfaces | **Not allowed**; interfaces may not declare instance fields. |
| Indexer auto-implementation | **Not allowed**; indexers cannot be auto-implemented. |
| Getters throwing exceptions | **Avoid**; getters should be simple operations. |
| Setter with broader accessibility than getter | **Discouraged**. |

---

## References (Consolidated)

- Properties (C# Programming Guide) — Microsoft Learn - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/properties
- Auto-Implemented Properties (C# Programming Guide) — Microsoft Learn - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/auto-implemented-properties
- Automatically implemented properties — C# language specification - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/language-specification/classes#1574-automatically-implemented-properties
- Init accessors — C# language specification - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/language-specification/classes#1573-init-accessors
- init keyword — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/init
- Welcome to C# 9.0 — .NET Blog (init accessors) - https://devblogs.microsoft.com/dotnet/welcome-to-csharp-9/
- required modifier — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/required
- Required members — C# language specification - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/language-specification/classes#1571-required-members
- Required properties — Properties (C# Programming Guide) - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/properties#required-properties
- Field-backed properties — Properties (C# Programming Guide) - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/properties#field-backed-properties
- field keyword — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/field
- Indexers — C# Programming Guide - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/indexers/
- get keyword — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/get
- set keyword — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/set
- Static and instance properties — C# language specification - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/language-specification/classes#1572-static-and-instance-properties
- Expression-bodied members — C# Programming Guide - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/statements-expressions-operators/expression-bodied-members
- Property Design — Framework Design Guidelines - https://learn.microsoft.com/en-us/dotnet/standard/design-guidelines/property
- CA1044: Properties should not be write only — .NET code analysis - https://learn.microsoft.com/en-us/dotnet/fundamentals/code-analysis/quality-rules/ca1044
- Indexed Property Design — Framework Design Guidelines - https://learn.microsoft.com/en-us/dotnet/standard/design-guidelines/property#indexed-property-design
- Properties — C# language specification - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/language-specification/classes#157-properties

---