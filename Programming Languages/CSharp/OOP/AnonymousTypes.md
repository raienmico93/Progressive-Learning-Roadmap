# C# Anonymous Types — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
An **anonymous type** is a **compiler-generated, read-only reference type** that encapsulates a set of properties into a single object **without requiring a named class declaration**. Introduced in **C# 3.0** alongside LINQ, anonymous types allow developers to project data into lightweight, temporary shapes directly within a method scope. The compiler infers property names and types from the initializer, generates an `internal sealed class` behind the scenes, and provides **value-based equality**, **`GetHashCode`**, and **`ToString`** overrides. Because the type name is not accessible in source code, anonymous types must be declared with `var` and are confined to the **local scope** of the declaring method.

**Technical Definition**
Anonymous types provide a convenient way to encapsulate a set of read-only properties into a single object without defining a named type first. The compiler generates a type name at compile time that cannot be accessed in source code, and it infers the type of each property. Anonymous types are created using the `new` operator together with an **object initializer** (`new { ... }`). Each property is **public** and **read-only** (immutable). The compiler emits an `internal sealed class` that derives from `System.Object` and overrides `Equals`, `GetHashCode`, and `ToString` with **value-based semantics**. Two anonymous type instances that have the **same property names, types, and order** within the same assembly share the **same compiler-generated type**. Anonymous types support **`with` expressions** for nondestructive mutation (C# 10+). They are **not denotable**—the type name cannot be written—so they cannot be used as method return types, parameters, fields, or properties.

**Beginner-Friendly Explanation**
An anonymous type is a **throwaway object you create on the fly**. Instead of writing a whole class definition just to hold two or three related values, you write `new { Name = "Alice", Age = 30 }` and C# creates a hidden, read-only object for you. The compiler names the type (something you cannot type yourself) and gives it **value equality**—two anonymous objects with the same properties and values are considered equal. Because you cannot write the type name, you must use `var` and you can only use the object **inside the method where it was created**. Anonymous types are the Swiss Army knife of **LINQ projections**: they let you select just the columns you need from a query result without defining a DTO class for every shape. They are perfect for **temporary, method-local data**—but they should not be used for long-lived data, public APIs, or anything that needs to cross method boundaries.

---

### Key Characteristics

- **Compiler-generated type** — The C# compiler emits an `internal sealed class` that derives from `System.Object`; the type name is inaccessible in source code.
- **Read-only properties** — All properties are `public` and **read-only** (getter-only); the object is immutable after construction.
- **`var` required** — Because the type name is inaccessible, variables must be declared with `var`.
- **Value-based equality** — The compiler overrides `Equals` and `GetHashCode` to compare **property values**, not references.
- **Structural identity** — Two anonymous type initializers with the **same property names, types, and order** in the same assembly produce the **same compiler-generated type**.
- **`with` expression support** — Anonymous types support `with` expressions for nondestructive mutation (C# 10+).
- **Expression tree support** — Anonymous types can be used in expression trees; tuples cannot.
- **Method-scoped** — Cannot be used as return types, parameters, fields, or properties because the type cannot be named.
- **Inferred property names** — Property names can be inferred from variables or member access expressions.

---

### Prerequisites

- Basic understanding of C# classes, objects, and properties.
- Familiarity with the `new` operator and object initializers.
- Knowledge of `var` and implicit typing.
- Awareness of LINQ and query projections (for the primary use case).
- Understanding of `Equals`, `GetHashCode`, and `ToString`.

---

### Related Programming Areas

- **LINQ** — Anonymous types are the primary projection mechanism in query expressions.
- **Object initializers** — Anonymous types are created using object initializer syntax.
- **Tuples** — An alternative for lightweight data shapes; tuples have value semantics and are denotable, unlike anonymous types.
- **DTOs and view models** — Named types are preferred for public APIs and cross-method data.
- **Expression trees** — Anonymous types support expression trees; tuples do not.
- **Method-local scope** — Anonymous types are confined to the method in which they are declared.

---

### Core Concepts / Features

1. Anonymous Types (Overarching Concept)
2. `new { }` Syntax
3. Anonymous Type Properties (Explicit and Inferred)
4. Anonymous Type Limitations
5. Anonymous Type Best Practices

Each concept follows a uniform structure: definitions, purposes, syntax rules and structure, annotated code examples with expected outputs, real-world cases, and references.

---

## 1. Anonymous Types (Overarching Concept)

### Definitions

**Core Definition**
An anonymous type is a compiler-generated, immutable reference type that groups a set of read-only properties without a named class declaration.

**Technical Definition**
Anonymous types provide a convenient way to encapsulate a set of read-only properties into a single object without defining a named type first. The compiler generates a type name at compile time that cannot be accessed in source code, and it infers the type of each property. The compiler emits an `internal sealed class` that derives from `Object`; all properties are `public` and read-only; and the compiler generates value-based `Equals`, `GetHashCode`, and `ToString` overrides. Anonymous types are created with the `new` operator and an object initializer.

**Beginner-Friendly Explanation**
An anonymous type is a **nameless, read-only object** you create on the spot. You do not write a class; the compiler does it for you.

---

### Purposes

- To **project a subset of data** without defining a named type.
- To create **lightweight, method-local containers** for temporary data.
- To simplify **LINQ query projections** by selecting only the needed properties.
- To avoid boilerplate DTO classes for one-off data shapes.
- To enable **expression tree** scenarios where tuples are not supported.

---

### Syntax Rules and Structure

```
var variableName = new { Property1 = value1, Property2 = value2, ... };
```

**Component breakdown**

- `var` — Required; the compiler-generated type name cannot be written.
- `new` — The creation operator.
- `{ ... }` — The object initializer containing property assignments.
- `Property = value` — Explicit property name assignment.
- `variableName` — Inferred property name from a variable or member access expression.

#### Syntax Rules

- Variables **must** be declared with `var`.
- Properties are **read-only** (getter-only).
- Property names can be explicit (`Name = value`) or inferred from variables/member access expressions.
- Anonymous types can be **nested** (anonymous types as property values).
- Anonymous types support **`with` expressions** (C# 10+).

#### Constraints and Limitations

- Cannot be used as **method return types, parameters, fields, or properties** because the type name cannot be written.
- Cannot be cast to a named type (except `object`).
- Properties are **immutable**; there are no setters.
- Two anonymous types with the **same property names, types, and order** in the same assembly share the same compiler-generated type.
- **Property order matters** for type identity.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Basic Anonymous Type

```csharp
using System;

class Program
{
    static void Main()
    {
        var person = new { Name = "Alice", Age = 30 };
        Console.WriteLine($"{person.Name} is {person.Age} years old.");
        // Alice is 30 years old.
    }
}
```

**Expected Output**

```
Alice is 30 years old.
```

**Why this output occurs**

- The compiler generates an anonymous type with read-only `Name` and `Age` properties. `var` captures the compiler-generated type, and the properties are accessed as if they were declared on a named class.

---

#### Example 2: Structural Identity and Equality

```csharp
using System;

class Program
{
    static void Main()
    {
        var a = new { Name = "Alice", Age = 30 };
        var b = new { Name = "Alice", Age = 30 };
        var c = new { Name = "Bob", Age = 30 };

        Console.WriteLine(a.Equals(b));   // True
        Console.WriteLine(a.Equals(c));   // False
        Console.WriteLine(a == b);        // False (== is reference equality for anonymous types)
    }
}
```

**Expected Output**

```
True
False
False
```

**Why this output occurs**

- `Equals` is overridden to compare **property values**, so `a` and `b` are equal. The `==` operator is **not** overloaded for anonymous types, so it compares references.

---

#### Example 3: Nested Anonymous Types

```csharp
using System;

class Program
{
    static void Main()
    {
        var order = new
        {
            OrderId = 1,
            Customer = new { Name = "Alice", City = "Seattle" },
            Total = 150.00m
        };

        Console.WriteLine($"Order {order.OrderId} for {order.Customer.Name} in {order.Customer.City}");
        // Order 1 for Alice in Seattle
    }
}
```

**Expected Output**

```
Order 1 for Alice in Seattle
```

**Why this output occurs**

- The `Customer` property is itself an anonymous type with `Name` and `City` properties.

---

### Real-World Cases

- **LINQ projections** — Selecting a subset of columns from a query result.
- **Temporary data shapes** — Combining values from multiple sources within a method.
- **Ad-hoc debugging** — Creating quick containers for diagnostic output.
- **Joins** — Projecting composite keys or results.

---

### References

- Anonymous types — C# Programming Guide - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/anonymous-types
- Choose between anonymous types and tuples — .NET - https://learn.microsoft.com/en-us/dotnet/standard/base-types/choosing-between-anonymous-and-tuple

---

## 2. `new { }` Syntax

### Definitions

**Core Definition**
The `new { }` syntax is the **creation expression** for anonymous types, combining the `new` operator with an object initializer block.

**Technical Definition**
Anonymous types are created by using the `new` operator together with an **object initializer**. The initializer block (`{ ... }`) contains property assignments. The compiler infers the types of the properties from the assigned expressions. The `new` expression returns an instance of the compiler-generated anonymous type.

**Beginner-Friendly Explanation**
`new { }` is the **factory syntax** for anonymous types. You write `new` followed by a brace-enclosed list of property assignments.

---

### Purposes

- To create anonymous type instances with inferred property types.
- To support the primary projection mechanism for LINQ.
- To provide a concise alternative to named type definitions for temporary data.

---

### Syntax Rules and Structure

```
new { Property1 = expression1, Property2 = expression2, ... }
```

**Component breakdown**

- `new` — The creation operator.
- `{ ... }` — The object initializer.
- `Property = expression` — Explicit property assignment.
- `expression` — The value; its type is inferred as the property type.

#### Syntax Rules

- The initializer must contain at least one property.
- Property names must be unique within the anonymous type.
- Property names can be explicit or inferred.
- The result must be assigned to a `var` variable.

#### Constraints and Limitations

- Cannot be used with `null` literals for property values (the compiler cannot infer the property type).
- Cannot be used to create mutable properties.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Basic `new { }` Creation

```csharp
using System;

class Program
{
    static void Main()
    {
        var point = new { X = 10, Y = 20 };
        Console.WriteLine($"({point.X}, {point.Y})");   // (10, 20)
    }
}
```

**Expected Output**

```
(10, 20)
```

**Why this output occurs**

- The `new { }` expression creates an anonymous type with `X` and `Y` properties inferred as `int`.

---

#### Example 2: Inferred Property Names

```csharp
using System;

class Program
{
    static void Main()
    {
        string productName = "Laptop";
        decimal price = 999.99m;

        var product = new { productName, price };
        Console.WriteLine($"{product.productName}: {product.price:C}");
        // Laptop: $999.99
    }
}
```

**Expected Output**

```
Laptop: $999.99
```

**Why this output occurs**

- The compiler infers the property names `productName` and `price` from the variable names used in the initializer.

---

### Real-World Cases

- **All anonymous type creation** — The `new { }` syntax is the only way to create anonymous types.
- **LINQ projections** — `select new { ... }` is the most common usage.

---

### References

- Anonymous types — C# Programming Guide (creation) - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/anonymous-types
- new operator — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/new-operator

---

## 3. Anonymous Type Properties

### Definitions

**Core Definition**
Anonymous type properties are the **read-only, public properties** generated by the compiler from the assignments in the object initializer. They can be **explicitly named** or **implicitly inferred** from variables and member access expressions.

**Technical Definition**
Anonymous types contain a set of read-only properties that are public and immutable. Property names can be specified explicitly using the `Name = value` syntax, or inferred from a variable or member access expression when the initializer uses the shorthand form. The compiler infers the **type** of each property from the assigned expression. The order of properties affects type identity: two anonymous types with the same property names, types, and order in the same assembly share the same compiler-generated type.

**Beginner-Friendly Explanation**
Anonymous type properties are the **data fields** of your anonymous object. You can name them yourself, or let the compiler infer the name from a variable.

---

### Purposes

- To define the data shape of the anonymous type.
- To enable **projection** of selected properties from source objects.
- To provide readable, self-documenting property names.
- To support inferred naming for concise code.

---

### Syntax Rules and Structure

```
// Explicit property name
new { PropertyName = expression }

// Inferred property name from variable
new { variableName }

// Inferred property name from member access
new { obj.Property }
```

**Component breakdown**

- `PropertyName = expression` — Explicit name and value.
- `variableName` — Inferred name from the variable.
- `obj.Property` — Inferred name from the member access expression.

#### Syntax Rules

- Properties are **read-only** (getter-only).
- Properties are **public** and **immutable**.
- Property names must be unique within the anonymous type.
- Property order matters for structural identity.
- Inferred names use the last identifier in the expression (e.g., `obj.Property` infers `Property`).

#### Constraints and Limitations

- Cannot have setters; the object is immutable after construction.
- Property names cannot conflict with generated methods (`Equals`, `GetHashCode`, `ToString`) in a way that causes ambiguity.
- Cannot use `null` as a value without a type annotation (the compiler cannot infer the type).

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Explicit and Inferred Property Names

```csharp
using System;

class Program
{
    static void Main()
    {
        string productName = "Laptop";
        decimal price = 999.99m;

        var explicitProduct = new { Name = productName, Price = price };
        var inferredProduct = new { productName, price };

        Console.WriteLine(explicitProduct.Name);       // Laptop
        Console.WriteLine(inferredProduct.productName); // Laptop
    }
}
```

**Expected Output**

```
Laptop
Laptop
```

**Why this output occurs**

- Explicit naming uses `Name = productName`; inferred naming uses the variable name `productName` directly.

---

#### Example 2: Property Order and Type Identity

```csharp
using System;

class Program
{
    static void Main()
    {
        var a = new { Name = "Alice", Age = 30 };
        var b = new { Age = 30, Name = "Alice" };

        Console.WriteLine(a.GetType() == b.GetType());   // False (different order)
    }
}
```

**Expected Output**

```
False
```

**Why this output occurs**

- Although the property names and values are the same, the **order** differs, so the compiler generates **different anonymous types**.

---

### Real-World Cases

- **LINQ projections** — Selecting a subset of properties from a source object.
- **Joins** — Creating composite keys or result shapes.
- **Method-local DTOs** — Temporary data carriers.

---

### References

- Anonymous types — C# Programming Guide (inferred property names) - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/anonymous-types
- Anonymous types — equality and structural identity - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/anonymous-types

---

## 4. Anonymous Type Limitations

### Definitions

**Core Definition**
Anonymous type limitations are the **compile-time and scope restrictions** that prevent anonymous types from being used in many scenarios where named types are required, particularly across method boundaries and in public APIs.

**Technical Definition**
Because the compiler generates the type name and it cannot be accessed in source code, anonymous types have several limitations. You cannot declare a **field, property, event, or method return type** as an anonymous type. Similarly, you cannot declare a **formal parameter** of a method, property, constructor, or indexer as having an anonymous type. Anonymous types are confined to the **local scope** of the declaring method. To pass an anonymous type to another method, the parameter must be declared as `object`, which loses static typing and requires casting or reflection to access its properties. Anonymous types also cannot be used in **serialization** frameworks that require a parameterless constructor or known type information.

**Beginner-Friendly Explanation**
Anonymous types are **trapped inside the method where they are created**. You cannot return them from a method, pass them as parameters, or store them in fields. If you try, you must use `object`, which means you lose all the nice property access and type safety.

---

### Purposes

- To prevent misuse of anonymous types in scenarios where named types are required.
- To enforce the intended **method-local** usage of anonymous types.
- To guide developers toward named types (classes, records, tuples) for cross-method data.

---

### Syntax Rules and Structure (Limitation Matrix)

| Context | Allowed? | Reason |
|---|---|---|
| **Local variable** | ✔️ | `var` captures the compiler-generated type |
| **Method return type** | ❌ | Cannot name the type |
| **Method parameter** | ❌ | Cannot name the type |
| **Field / Property** | ❌ | Cannot name the type |
| **Event** | ❌ | Cannot name the type |
| **Cast target** | ❌ | Cannot name the type (except `object`) |
| **Serialization** | ❌ | No known type, no parameterless constructor |
| **Cross-assembly** | ❌ | Anonymous types are `internal`; not accessible outside the assembly |

**Component breakdown**

- **Local variable** — The primary valid context.
- **Return type / parameter / field** — Not allowed.
- **Serialization** — Not supported.

#### Syntax Rules

- Anonymous types are **`internal`** (cannot be accessed outside the assembly).
- The type name cannot be written in source code.
- To pass an anonymous type to another method, the parameter must be `object`.
- Reflection or `dynamic` is required to access properties when the static type is `object`.

#### Constraints and Limitations

- Cannot be used in **public API signatures**.
- Cannot be used for **long-lived data** or cross-method communication.
- Cannot be used with **serializers** that require known types.
- Property access through `object` requires reflection or `dynamic`.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Attempting to Return an Anonymous Type (Compile Error)

```csharp
using System;

class Program
{
    // COMPILE ERROR: Cannot use anonymous type as return type
    // static var GetPerson() => new { Name = "Alice", Age = 30 };

    static object GetPersonAsObject() => new { Name = "Alice", Age = 30 };

    static void Main()
    {
        object person = GetPersonAsObject();
        // person.Name // Compile error: object does not have Name
        Console.WriteLine(person.GetType().Name);   // <>f__AnonymousType0`2
    }
}
```

**Expected Output**

```
<>f__AnonymousType0`2
```

**Why this output occurs**

- The method returns `object` because the anonymous type cannot be named as a return type. Accessing properties through `object` requires reflection or `dynamic`.

---

#### Example 2: Passing an Anonymous Type as `object`

```csharp
using System;

class Program
{
    static void PrintObject(object obj)
    {
        // obj.Name // Compile error
        Console.WriteLine(obj);   // { Name = Alice, Age = 30 }
    }

    static void Main()
    {
        var person = new { Name = "Alice", Age = 30 };
        PrintObject(person);
    }
}
```

**Expected Output**

```
{ Name = Alice, Age = 30 }
```

**Why this output occurs**

- The anonymous type is passed as `object`; the compiler-generated `ToString` is called.

---

### Real-World Cases

- **Method boundaries** — Anonymous types cannot cross method boundaries; use named types or tuples instead.
- **Public APIs** — Anonymous types cannot appear in public signatures; use DTOs.
- **Serialization** — Anonymous types cannot be serialized reliably; use named types.

---

### References

- Anonymous types — C# Programming Guide (limitations) - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/anonymous-types
- Why anonymous types are not denotable — Stack Overflow - https://stackoverflow.com/questions/686114/what-are-the-fundamental-differences-between-java-and-c-in-terms-of-inner-local-a

---

## 5. Anonymous Type Best Practices

### Definitions

**Core Definition**
Anonymous type best practices are the design guidelines that ensure anonymous types are used **appropriately, safely, and within their intended scope**, while choosing named types or tuples when the scenario demands it.

**Technical Definition**
Best practices for anonymous types derive from Microsoft’s C# documentation and design guidance. Key recommendations include: **use anonymous types for LINQ projections and method-local temporary data**; **prefer named types (classes, records) for public APIs, return types, and cross-method data**; **consider tuples as an alternative when you need value semantics and denotable types**; **be aware of the structural identity rules** (property names, types, and order must match for type sharing); and **do not rely on anonymous types for serialization or long-lived storage**. The choice between anonymous types and tuples depends on the scenario: anonymous types provide named properties and support expression trees; tuples provide denotable types, value semantics, and are lighter weight.

**Beginner-Friendly Explanation**
Best practices for anonymous types are simple: **use them for quick, local projections** (especially in LINQ), **do not try to pass them around**, and **switch to named types or tuples** when the data needs to live longer or cross boundaries.

---

### Purposes

- To choose the right tool (anonymous type vs. tuple vs. named type) for each scenario.
- To avoid the limitations of anonymous types in inappropriate contexts.
- To maintain type safety and API clarity.
- To align with .NET design conventions and LINQ idioms.

---

### Guidelines and Structure

#### General Guidelines

```
// USE anonymous types for LINQ projections and method-local data
var results = customers.Select(c => new { c.Name, c.City });

// PREFER named types (records) for cross-method data and public APIs
public record CustomerDto(string Name, string City);

// CONSIDER tuples when value semantics and denotable types are needed
var (name, city) = GetCustomerTuple();

// DO NOT use anonymous types as return types, parameters, or fields
// DO NOT rely on anonymous types for serialization
// BE AWARE of structural identity (names, types, order)
```

**Component breakdown**

- **LINQ projections** — The primary and idiomatic use of anonymous types.
- **Named types** — Required for cross-method data and public APIs.
- **Tuples** — An alternative with value semantics and denotable types.
- **Structural identity** — Property names, types, and order must match for type sharing.
- **No serialization** — Anonymous types lack known type information and parameterless constructors.

#### Rules and Constraints

- **Do** use anonymous types for LINQ projections and method-local temporary data.
- **Do** use `var` to declare anonymous type variables.
- **Do** be aware of structural identity rules.
- **Do not** use anonymous types as method return types, parameters, fields, or properties.
- **Do not** rely on anonymous types for serialization or long-lived data.
- **Do not** use anonymous types in public API signatures.
- **Consider** tuples as an alternative when value semantics and denotable types are needed.

---

### Annotated Code Examples and Expected Outputs

#### Example: Appropriate Use in LINQ

```csharp
using System;
using System.Linq;

class Program
{
    static void Main()
    {
        var words = new[] { "apple", "blueberry", "cherry" };

        var results = words.Select(w => new { Word = w, Length = w.Length });

        foreach (var item in results)
        {
            Console.WriteLine($"{item.Word} has {item.Length} letters.");
        }
        // apple has 5 letters.
        // blueberry has 9 letters.
        // cherry has 6 letters.
    }
}
```

**Expected Output**

```
apple has 5 letters.
blueberry has 9 letters.
cherry has 6 letters.
```

**Why this output occurs**

- The anonymous type is used **within the method scope** for a LINQ projection, which is the intended use case.

---

#### Example: Choosing a Named Type for Cross-Method Data

```csharp
using System;

// PREFERRED: Named record for cross-method data
public record CustomerDto(string Name, string City);

class Program
{
    static CustomerDto GetCustomer() => new CustomerDto("Alice", "Seattle");

    static void Main()
    {
        var customer = GetCustomer();
        Console.WriteLine($"{customer.Name} in {customer.City}");
        // Alice in Seattle
    }
}
```

**Expected Output**

```
Alice in Seattle
```

**Why this output occurs**

- A named type (`CustomerDto`) is used because the data crosses a method boundary, which anonymous types cannot do.

---

### Real-World Cases

- **LINQ queries** — The primary use case for anonymous types.
- **API design** — Use named types (records or classes) for public APIs.
- **Tuple alternatives** — Use `ValueTuple` when you need value semantics and denotable types.
- **Serialization** — Use named types with parameterless constructors for serializers.

---

### References

- Anonymous types — C# Programming Guide - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/anonymous-types
- Choose between anonymous types and tuples — .NET - https://learn.microsoft.com/en-us/dotnet/standard/base-types/choosing-between-anonymous-and-tuple

---

## Deprecated, Unsafe, Implementation-Dependent, and Version-Specific Notes

| Feature | Status / Note |
|---------|---------------|
| Anonymous types | Core since **C# 3.0**; stable. |
| `with` expression support | Introduced in **C# 10**; version-specific. |
| Expression tree support | Anonymous types support expression trees; tuples do not. |
| Structural identity | Property names, types, and **order** must match for type sharing. |
| Serialization | **Not supported**; anonymous types lack known type information. |
| Cross-method usage | **Not supported**; cannot name the type. |
| `internal` accessibility | Compiler generates `internal sealed class`; not accessible outside the assembly. |

---

## References (Consolidated)

- Anonymous types — C# Programming Guide - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/anonymous-types
- Choose between anonymous types and tuples — .NET - https://learn.microsoft.com/en-us/dotnet/standard/base-types/choosing-between-anonymous-and-tuple
- Anonymous Type Definition — Microsoft Learn (previous versions) - https://learn.microsoft.com/en-us/previous-versions/visualstudio/visual-studio-2008/bb531354(v=vs.90)
- Tuples, Anonymous Types, and Concrete Types — Microsoft Download Center - https://download.microsoft.com
- Why anonymous types are not denotable — Stack Overflow - https://stackoverflow.com/questions/686114/what-are-the-fundamental-differences-between-java-and-c-in-terms-of-inner-local-a
- LINQ Tip of the week: LINQ and Anonymous types — Telerik - https://www.telerik.com
- Understand the concept of anonymous types — GitHub - https://github.com
- Projecting Data Sequences into New Types with LINQ — Educative - https://www.educative.io

---