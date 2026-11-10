# C# Object Initializers — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
An **object initializer** is a C# syntax that lets you **create an object and assign values to its accessible fields or properties in a single expression**, without having to invoke a constructor and then write separate assignment statements. Collection initializers, index initializers, and anonymous types are closely related features built on the same underlying idea: **declarative, concise initialization**.

**Technical Definition**
Object initializers let you assign values to any accessible fields or properties of an object when you create it. You don't need to invoke a constructor and then use assignment statements. The object initializer syntax enables you to specify arguments for a constructor or omit the arguments and parentheses. **Collection initializers** let you specify one or more element initializers when you initialize a collection type that implements `IEnumerable` and has an `Add` method with the appropriate signature. **Index initializers** allow you to set values through an accessible indexer as part of the initializer. **Anonymous types** are created by using the `new` operator together with an object initializer; the compiler generates a type name that you cannot access in source code, so you must use `var` to declare the variable. The `new` operator is the gateway for all these forms: `new T { ... }`, `new T(args) { ... }`, `new T { [index] = value }`, and `new { Property = value }`.

**Beginner-Friendly Explanation**
Object initializers are a **shorthand for setting up an object at the moment you create it**. Instead of writing three lines to create a `Person` and set its name, age, and city, you write one line with a `{ ... }` block. Collection initializers do the same for lists and dictionaries: instead of calling `Add` repeatedly, you list the items in braces. Index initializers let you write `[key] = value` for dictionary-like types. Anonymous types let you create a lightweight, read-only object on the fly without declaring a class. All of these features make C# code **shorter, clearer, and more expressive**—but they come with some subtle rules and trade-offs that are important to understand.

---

### Key Characteristics

- **Single-statement initialization** — Create an object or collection and assign members in one expression.
- **Constructor optionality** — Object initializers work with both parameterless and parameterized constructors; the argument list can be omitted when an initializer is present.
- **Collection initializer requirement** — The collection type must implement `IEnumerable` and have an accessible `Add` method.
- **Index initializer support** — Any accessible indexer with a setter can be used in an initializer.
- **Anonymous type immutability** — Anonymous types expose read-only properties; they are created only with object initializers and must be declared with `var`.
- **`required` and `init` integration** — Object initializers are the mechanism for setting `required` members and `init`-only properties.
- **Nested initializer distinction** — `Property = { ... }` (without `new`) initializes an existing nested object, while `Property = new() { ... }` replaces it with a new instance.
- **Style-rule enforcement** — IDE0017 (object initializers) and IDE0028 (collection initializers) allow teams to enforce consistent usage.

---

### Prerequisites

- Basic understanding of C# classes, structs, fields, and properties.
- Familiarity with constructors and the `new` operator.
- Knowledge of `IEnumerable` and collection types (`List<T>`, `Dictionary<TKey,TValue>`).
- Awareness of indexers and the `this[...]` syntax.
- Understanding of `var` and implicit typing (for anonymous types).

---

### Related Programming Areas

- **Object-oriented programming** — Initializers complement constructors for object creation.
- **Collections** — Collection initializers are the idiomatic way to populate lists and dictionaries.
- **LINQ** — Anonymous types and object initializers are heavily used in query projections.
- **Immutability** — `init` accessors and `required` members rely on object initializers.
- **API design** — Initializer syntax shapes the ergonomics of object and collection creation.

---

### Core Concepts / Features

1. Object Initializers
2. Collection Initializers
3. Index Initializers
4. Anonymous Types
5. `new` with Initializers
6. Object Initializer Best Practices

Each concept follows a uniform structure: definitions, purposes, syntax rules and structure, annotated code examples with expected outputs, real-world cases, and references.

---

## 1. Object Initializers

### Definitions

**Core Definition**
An object initializer is a `{ ... }` block following `new T` (or `new T(args)`) that assigns values to accessible fields and properties of the newly created object.

**Technical Definition**
Object initializers let you assign values to any accessible fields or properties of an object when you create it. You don't need to invoke a constructor and then use assignment statements. The syntax is `new T { Property1 = value1, Property2 = value2, ... }` or `new T(args) { Property1 = value1 }`. The initializer runs **after** the constructor body completes, so it can override values set by the constructor. Any accessible field or property with a setter (including `init`-only properties) can be assigned.

**Beginner-Friendly Explanation**
An object initializer is a **fill-in-the-blanks block at creation time**. You create the object and immediately fill in its properties in one line.

---

### Purposes

- To create and configure an object in a single, readable expression.
- To avoid temporary variables and repeated assignment statements.
- To support `required` members and `init`-only properties.
- To improve the readability of object creation, especially for DTOs and configuration objects.
- To reduce boilerplate when many properties need initial values.

---

### Syntax Rules and Structure

```
// With a parameterless constructor
T obj = new T { Property1 = value1, Property2 = value2 };

// With a parameterized constructor
T obj = new T(arg1, arg2) { Property3 = value3 };
```

**Component breakdown**

- `new T` — Object creation expression.
- `{ ... }` — The object initializer.
- `Property = value` — A member assignment; the property or field must be accessible and settable (or `init`-only).

#### Syntax Rules

- The object initializer runs **after** the constructor body.
- Any accessible field or property with a setter (including `init`-only) can be assigned.
- The constructor argument list can be omitted when an initializer is present.
- The initializer cannot access `this` or instance members of the newly created object during the initializer (the object is fully constructed by the time the initializer runs).
- Nested initializers can use `Property = { ... }` to initialize an existing nested object, or `Property = new() { ... }` to replace it.

#### Constraints and Limitations

- Read-only properties (without `init`) cannot be set in an initializer unless they are initialized in the constructor.
- The initializer executes after the constructor, so constructor-side effects happen first.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Basic Object Initializer

```csharp
using System;

public class Cat
{
    public int Age { get; set; }
    public string? Name { get; set; }

    public Cat() { }
    public Cat(string name) => Name = name;
}

class Program
{
    static void Main()
    {
        Cat cat = new Cat { Age = 10, Name = "Fluffy" };
        Console.WriteLine($"{cat.Name} is {cat.Age} years old.");
        // Fluffy is 10 years old.
    }
}
```

**Expected Output**

```
Fluffy is 10 years old.
```

**Why this output occurs**

- The object initializer assigns `Age` and `Name` after the parameterless constructor runs. Without the initializer, the same code would require separate assignments.

---

#### Example 2: Combining Constructor Arguments and Initializers

```csharp
using System;

public class Cat
{
    public int Age { get; set; }
    public string? Name { get; set; }

    public Cat(string name) => Name = name;
}

class Program
{
    static void Main()
    {
        Cat sameCat = new Cat("Fluffy") { Age = 10 };
        Console.WriteLine($"{sameCat.Name} is {sameCat.Age} years old.");
        // Fluffy is 10 years old.
    }
}
```

**Expected Output**

```
Fluffy is 10 years old.
```

**Why this output occurs**

- The constructor sets `Name`, and the object initializer sets `Age` afterward. The initializer runs after the constructor body, so it can also override constructor-set values.

---

#### Example 3: Nested Object Initializers (With and Without `new`)

```csharp
using System;

public class Inner
{
    public int Value { get; set; }
    public bool Flag { get; set; }
    public Inner() => (Value, Flag) = (23, false);
}

public class Outer
{
    public Inner Nested { get; } = new();
}

class Program
{
    static void Main()
    {
        // Without new: modifies the existing nested instance
        var a = new Outer { Nested = { Value = 100 } };
        Console.WriteLine($"{a.Nested.Value}, {a.Nested.Flag}"); // 100, False

        // With new: replaces the nested instance
        var b = new Outer { Nested = new() { Value = 200 } };
        Console.WriteLine($"{b.Nested.Value}, {b.Nested.Flag}"); // 200, False
    }
}
```

**Expected Output**

```
100, False
200, False
```

**Why this output occurs**

- `Nested = { Value = 100 }` (without `new`) **modifies the existing instance** created by the `Outer` constructor’s property initializer.
- `Nested = new() { Value = 200 }` **replaces** the nested instance with a new one.
- The `without new` syntax is especially useful for **read-only properties** that cannot be reassigned but whose members can still be initialized.

---

### Real-World Cases

- **DTOs and view models** — Creating data carriers with many properties.
- **Configuration objects** — Setting options at creation.
- **Test data** — Building expected objects concisely.
- **LINQ projections** — Initializing named types in `select` clauses.

---

### References

- Object and collection initializers — C# Programming Guide - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/object-and-collection-initializers
- IDE0017: Use object initializers — .NET code analysis - https://learn.microsoft.com/en-us/dotnet/fundamentals/code-analysis/style-rules/ide0017

---

## 2. Collection Initializers

### Definitions

**Core Definition**
A collection initializer is a `{ ... }` block that adds elements to a collection at creation time. The collection type must implement `IEnumerable` and have an accessible `Add` method.

**Technical Definition**
Collection initializers let you specify one or more element initializers when you initialize a collection type that implements `IEnumerable` and has an `Add` method with the appropriate signature as an instance method or an extension method. The element initializers can be a value, an expression, or an object initializer. By using a collection initializer, you don't have to specify multiple calls; the compiler adds the calls automatically. For types whose `Add` method takes multiple parameters (e.g., `Dictionary<TKey,TValue>`), each element can be enclosed in braces as a parameter list.

**Beginner-Friendly Explanation**
A collection initializer is a **shopping list in braces**. Instead of calling `Add` repeatedly, you list the items and the compiler generates the `Add` calls.

---

### Purposes

- To populate a collection concisely at creation time.
- To avoid repetitive `Add` calls.
- To support nested object initializers inside collection initializers.
- To improve readability of test data and configuration.
- To integrate naturally with LINQ queries.

---

### Syntax Rules and Structure

```
// Single-parameter Add
var list = new List<int> { 1, 2, 3, 4, 5 };

// Multi-parameter Add (e.g., Dictionary)
var dict = new Dictionary<string, int>
{
    { "one", 1 },
    { "two", 2 }
};

// Element initializers can be object initializers
var cats = new List<Cat>
{
    new Cat { Name = "Fluffy", Age = 10 },
    new Cat { Name = "Whiskers", Age = 5 }
};
```

**Component breakdown**

- `{ ... }` — The collection initializer.
- Each element — A value, expression, or object initializer.
- For multi-parameter `Add` — Each element is `{ arg1, arg2 }`.

#### Syntax Rules

- The collection type must implement `IEnumerable`.
- The collection type must have an accessible `Add` method (instance or extension).
- The element initializer can be any expression convertible to the `Add` parameter type(s).
- The compiler generates `Add` calls for each element.

#### Constraints and Limitations

- Does not work with types that lack an `Add` method.
- The `Add` method is called for each element; side effects in `Add` occur in order.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: `List<T>` Collection Initializer

```csharp
using System;
using System.Collections.Generic;

class Program
{
    static void Main()
    {
        List<int> digits = new List<int> { 0, 1, 2, 3, 4, 5, 6, 7, 8, 9 };

        foreach (int d in digits)
            Console.Write(d + " ");
        // 0 1 2 3 4 5 6 7 8 9
    }
}
```

**Expected Output**

```
0 1 2 3 4 5 6 7 8 9 
```

**Why this output occurs**

- The collection initializer calls `Add` for each element in order.

---

#### Example 2: `Dictionary<TKey,TValue>` Collection Initializer

```csharp
using System;
using System.Collections.Generic;

class Program
{
    static void Main()
    {
        var ages = new Dictionary<string, int>
        {
            { "Alice", 30 },
            { "Bob", 25 },
            { "Carol", 35 }
        };

        foreach (var pair in ages)
            Console.WriteLine($"{pair.Key}: {pair.Value}");
    }
}
```

**Expected Output (order may vary)**

```
Alice: 30
Bob: 25
Carol: 35
```

**Why this output occurs**

- Each `{ "Name", Age }` is a parameter list for the dictionary’s two-parameter `Add` method.

---

### Real-World Cases

- **Configuration lists** — Populating allowed values, feature flags, or lookup tables.
- **Test fixtures** — Creating collections of test objects.
- **LINQ source data** — Building sequences for query demonstrations.

---

### References

- Object and collection initializers — Collection initializers - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/object-and-collection-initializers
- IDE0028: Use collection initializers — .NET code analysis - https://learn.microsoft.com/en-us/dotnet/fundamentals/code-analysis/style-rules/ide0028

---

## 3. Index Initializers

### Definitions

**Core Definition**
An index initializer is a form of object initializer that uses **indexer syntax** (`[key] = value`) to assign values to an object through its accessible indexer.

**Technical Definition**
Object initializers can set indexers in addition to assigning fields and properties. Any accessible indexer that contains an accessible setter can be used as one of the expressions in an object initializer, regardless of the number or types of arguments. The index arguments form the left side of the assignment, and the value is the right side. Index initializers are particularly common with `Dictionary<TKey,TValue>` and other keyed collections.

**Beginner-Friendly Explanation**
An index initializer lets you write `[key] = value` inside an object initializer, which is a concise way to populate a dictionary-like type.

---

### Purposes

- To populate indexer-based types concisely.
- To initialize dictionaries with key-value pairs in a familiar `[key] = value` form.
- To set values through multi-parameter indexers.
- To improve readability compared with nested `{ key, value }` collection initializers.

---

### Syntax Rules and Structure

```
T obj = new T
{
    [index1] = value1,
    [index2] = value2,
    Property = value3
};
```

**Component breakdown**

- `[index]` — The indexer arguments.
- `= value` — The value to assign through the indexer.
- The type must have an accessible indexer with a setter.

#### Syntax Rules

- Any accessible indexer with an accessible setter can be used.
- The index arguments form the left side of the assignment.
- The value is the right side of the assignment.
- Multiple indexers with different parameter types can be used in the same initializer.

#### Constraints and Limitations

- The indexer must have a setter.
- Index arguments must match the indexer signature.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Dictionary with Index Initializer

```csharp
using System;
using System.Collections.Generic;

class Program
{
    static void Main()
    {
        var ages = new Dictionary<string, int>
        {
            ["Alice"] = 30,
            ["Bob"] = 25,
            ["Carol"] = 35
        };

        Console.WriteLine(ages["Bob"]);   // 25
    }
}
```

**Expected Output**

```
25
```

**Why this output occurs**

- The index initializer uses the dictionary’s public read/write indexer to set each key-value pair.

---

#### Example 2: Multi-Parameter Indexer

```csharp
using System;

public class Matrix
{
    private double[,] _storage = new double[3, 3];

    public double this[int row, int col]
    {
        get => _storage[row, col];
        set => _storage[row, col] = value;
    }
}

class Program
{
    static void Main()
    {
        var identity = new Matrix
        {
            [0, 0] = 1.0, [0, 1] = 0.0, [0, 2] = 0.0,
            [1, 0] = 0.0, [1, 1] = 1.0, [1, 2] = 0.0,
            [2, 0] = 0.0, [2, 1] = 0.0, [2, 2] = 1.0
        };

        Console.WriteLine(identity[1, 1]);   // 1
    }
}
```

**Expected Output**

```
1
```

**Why this output occurs**

- The indexer takes two parameters, and the index initializer uses `[row, col] = value` syntax.

---

### Real-World Cases

- **Dictionaries and maps** — Key-value lookup tables.
- **Matrices and grids** — Multi-dimensional indexers.
- **Configuration caches** — Keyed settings.

---

### References

- Object and collection initializers — Index initializers - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/object-and-collection-initializers
- How to initialize a dictionary with a collection initializer — C# - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/how-to-initialize-a-dictionary-with-a-collection-initializer

---

## 4. Anonymous Types

### Definitions

**Core Definition**
An anonymous type is a **compiler-generated, read-only reference type** created with `new { ... }` and an object initializer. It encapsulates a set of read-only properties into a single object without defining a named type first.

**Technical Definition**
Anonymous types provide a convenient way to encapsulate a set of read-only properties into a single object without defining a named type first. The compiler generates a type name at compile time that you cannot access in source code. The compiler infers the type of each property. You create anonymous types by using the `new` operator together with an object initializer. Because the compiler generates the type name and you cannot access it in source code, you must use `var` to declare the local variable. Two anonymous type instances with the same property names and types in the same order share the same compiler-generated type; `Equals` and `GetHashCode` are overridden to compare property values.

**Beginner-Friendly Explanation**
An anonymous type is a **throwaway object with read-only properties**. You use it when you need a quick container for a few values and do not want to define a class.

---

### Purposes

- To project a subset of data without defining a named type.
- To create lightweight read-only containers for LINQ projections.
- To simplify temporary data shapes in queries and transformations.
- To avoid boilerplate classes for one-off data carriers.

---

### Syntax Rules and Structure

```
var person = new { Name = "Alice", Age = 30 };

// Inferred property names from variables/members
string productName = "Laptop";
decimal price = 999.99m;
var product = new { productName, price };
```

**Component breakdown**

- `new { ... }` — Anonymous object creation expression.
- `Property = value` — Explicit property name.
- `variable` — Inferred property name from the variable name.

#### Syntax Rules

- Anonymous types must be declared with `var` (the type name is compiler-generated).
- Properties are read-only.
- Property names can be explicit (`Name = value`) or inferred from variables/member access expressions.
- Anonymous types can be nested inside other anonymous types.
- Anonymous types cannot contain methods, events, or `null` initializers for properties (the compiler cannot infer a type from `null`).

#### Constraints and Limitations

- Cannot be used as method return types or field types directly.
- Cannot be cast to a named type (except `object`).
- Cannot have mutable properties.
- Property names must be unique within the anonymous type.

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

- The compiler generates an anonymous type with read-only `Name` and `Age` properties, and `var` captures the compiler-generated type.

---

#### Example 2: Inferred Property Names and LINQ

```csharp
using System;
using System.Linq;

class Program
{
    static void Main()
    {
        string productName = "Laptop";
        decimal price = 999.99m;

        var product = new { productName, price };
        Console.WriteLine($"{product.productName}: {product.price:C}");

        var words = new[] { "apple", "blueberry", "cherry" };
        var results = words.Select(w => new { Word = w, Length = w.Length });

        foreach (var item in results)
            Console.WriteLine($"{item.Word} has {item.Length} letters.");
    }
}
```

**Expected Output**

```
Laptop: $999.99
apple has 5 letters.
blueberry has 9 letters.
cherry has 6 letters.
```

**Why this output occurs**

- The first anonymous type infers property names from the variables `productName` and `price`.
- The LINQ projection creates an anonymous type per element with `Word` and `Length` properties.

---

#### Example 3: Anonymous Type Equality

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
    }
}
```

**Expected Output**

```
True
False
```

**Why this output occurs**

- The compiler overrides `Equals` and `GetHashCode` for anonymous types to compare property values, so `a` and `b` are equal because their properties match; `c` differs.

---

### Real-World Cases

- **LINQ projections** — `select new { p.ProductName, p.UnitPrice }`.
- **Temporary data shapes** — Combining values from multiple sources.
- **Debugging and logging** — Creating ad-hoc containers for diagnostic output.
- **Joins** — Projecting composite keys or results.

---

### References

- Anonymous types — C# Programming Guide - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/anonymous-types
- Anonymous object creation expressions — C# language specification - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/language-specification/expressions

---

## 5. `new` with Initializers

### Definitions

**Core Definition**
The `new` operator is the gateway for all initializer forms: `new T { ... }` (object initializer), `new T(args) { ... }` (constructor + initializer), `new T { [index] = value }` (index initializer), and `new { ... }` (anonymous type).

**Technical Definition**
The `new` operator creates an instance of a type. When combined with an object initializer, the `new` expression creates the instance, runs the constructor (if arguments are supplied), and then applies the initializer. For collection types, `new T { ... }` generates `Add` calls. For anonymous types, `new { ... }` generates a compiler-defined type. The `new` operator is also used with target-typed `new()` in C# 9+ when the type can be inferred from context.

**Beginner-Friendly Explanation**
`new` is the **creation keyword**. The initializer is the **setup block** that follows it.

---

### Purposes

- To create objects and collections with initial values in one expression.
- To combine constructor arguments with member assignments.
- To enable target-typed `new()` for concise initialization.
- To provide a uniform creation syntax across objects, collections, and anonymous types.

---

### Syntax Rules and Structure

```
// Object initializer
new T { Property = value }

// Constructor + object initializer
new T(args) { Property = value }

// Collection initializer
new List<T> { item1, item2 }

// Index initializer
new Dictionary<TKey,TValue> { [key] = value }

// Anonymous type
new { Property = value }

// Target-typed new (C# 9+)
T obj = new() { Property = value };
```

**Component breakdown**

- `new` — The creation operator.
- `T` — The type (optional for anonymous types).
- `(args)` — Optional constructor arguments.
- `{ ... }` — The initializer.

#### Syntax Rules

- The initializer runs after the constructor body.
- Target-typed `new()` requires a known target type.
- Anonymous types use `new { ... }` without a type name.

#### Constraints and Limitations

- Target-typed `new()` cannot be used with anonymous types.
- The `new` operator cannot be omitted entirely; only the type name can be inferred in target-typed `new()`.

---

### Annotated Code Examples and Expected Outputs

#### Example: Combining `new` with Different Initializers

```csharp
using System;
using System.Collections.Generic;

class Program
{
    static void Main()
    {
        // Object initializer
        var person = new Person { Name = "Alice", Age = 30 };

        // Collection initializer
        var numbers = new List<int> { 1, 2, 3 };

        // Index initializer
        var map = new Dictionary<string, int> { ["one"] = 1, ["two"] = 2 };

        // Anonymous type
        var anon = new { Message = "Hello" };

        Console.WriteLine($"{person.Name}, {numbers.Count}, {map["one"]}, {anon.Message}");
        // Alice, 3, 1, Hello
    }
}

public class Person
{
    public string? Name { get; set; }
    public int Age { get; set; }
}
```

**Expected Output**

```
Alice, 3, 1, Hello
```

**Why this output occurs**

- Each `new` expression uses the appropriate initializer form to create and configure the instance.

---

### Real-World Cases

- **All initializer usage** — `new` is the universal creation operator.
- **Target-typed `new()`** — Concise field and property initialization in modern C#.

---

### References

- new operator — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/new-operator
- Target-typed new expressions — C# 9.0 new features - https://learn.microsoft.com/en-us/dotnet/csharp/whats-new/csharp-9#target-typed-new

---

## 6. Object Initializer Best Practices

### Definitions

**Core Definition**
Object initializer best practices are the guidelines that ensure initializers are used **consistently, readably, and correctly**, without creating subtle bugs or violating API contracts.

**Technical Definition**
Best practices for object and collection initializers derive from code style rules (IDE0017, IDE0028) and design guidelines. Key recommendations include: **use object initializers when you can** (IDE0017); **use collection initializers instead of repeated `Add` calls** (IDE0028); **prefer object initializers for `required` and `init`-only members**; **be aware that the initializer runs after the constructor**, so constructor logic still applies; **use `Property = { ... }` for read-only nested objects** and `Property = new() { ... }` to replace them; **avoid side effects in initializers**; and **use anonymous types for projections, not for long-lived data**.

**Beginner-Friendly Explanation**
Best practices are the habits that keep initializer code clean: use them when they make the code shorter and clearer, **do not rely on them for logic that belongs in a constructor**, and be careful with the difference between modifying an existing nested object and replacing it.

---

### Purposes

- To improve readability and consistency.
- To avoid subtle bugs related to constructor ordering.
- To correctly handle `required` and `init`-only members.
- To choose the right initializer form for the scenario.
- To align with .NET style rules and tooling.

---

### Guidelines and Structure

#### General Guidelines

```
// Use object initializers when possible (IDE0017)
var person = new Person { Name = "Alice", Age = 30 };

// Use collection initializers instead of repeated Add calls (IDE0028)
var numbers = new List<int> { 1, 2, 3 };

// Use object initializers for required and init-only members
var pet = new Pet { Age = 10 };   // Age is required

// Nested: modify existing vs. replace
var outer = new Outer { Nested = { Value = 100 } };       // modify
var outer2 = new Outer { Nested = new() { Value = 100 } }; // replace

// Avoid side effects in initializers
// Use anonymous types for LINQ projections, not long-lived data
```

**Component breakdown**

- **IDE0017** — Prefer object initializers over separate assignments.
- **IDE0028** — Prefer collection initializers over repeated `Add` calls.
- **`required`/`init`** — Object initializers are the way to set these members.
- **Nested distinction** — `{ ... }` modifies; `new() { ... }` replaces.
- **Avoid side effects** — Initializers should be simple assignments.

#### Rules and Constraints

- **Do** use object initializers for straightforward object creation.
- **Do** use collection initializers for collection population.
- **Do** use the `without new` syntax for read-only nested objects.
- **Do not** put complex logic or side effects in initializers.
- **Do not** use anonymous types as method return types or long-lived data.
- **Do not** assume the initializer runs before the constructor; it runs after.

---

### Annotated Code Examples and Expected Outputs

#### Example: Consistent and Clear Initializer Usage

```csharp
using System;
using System.Collections.Generic;

public class Order
{
    public required int Id { get; init; }
    public string? Customer { get; init; }
    public List<string> Items { get; } = new();
}

class Program
{
    static void Main()
    {
        var order = new Order
        {
            Id = 1,
            Customer = "Alice",
            Items = { "Widget", "Gadget" }   // modifies existing list
        };

        Console.WriteLine($"{order.Id}: {order.Customer} ordered {order.Items.Count} items.");
        // 1: Alice ordered 2 items.
    }
}
```

**Expected Output**

```
1: Alice ordered 2 items.
```

**Why this output occurs**

- `Id` is `required` and set via the initializer; `Items` uses the `without new` syntax to add items to the existing list created by the property initializer.

---

### Real-World Cases

- **API request/response models** — Setting required and optional properties.
- **Test data builders** — Concise object and collection creation.
- **LINQ projections** — Anonymous types for query results.
- **Configuration** — Setting nested read-only configuration objects.

---

### References

- IDE0017: Use object initializers — .NET code analysis - https://learn.microsoft.com/en-us/dotnet/fundamentals/code-analysis/style-rules/ide0017
- IDE0028: Use collection initializers — .NET code analysis - https://learn.microsoft.com/en-us/dotnet/fundamentals/code-analysis/style-rules/ide0028
- Object and collection initializers — C# Programming Guide - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/object-and-collection-initializers
- Anonymous types — C# Programming Guide - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/anonymous-types

---

## Deprecated, Unsafe, Implementation-Dependent, and Version-Specific Notes

| Feature | Status / Note |
|---------|---------------|
| Object initializers | Core since **C# 3.0**; stable. |
| Collection initializers | Core since **C# 3.0**; stable. |
| Index initializers | Core since **C# 6.0**; stable. |
| Anonymous types | Core since **C# 3.0**; stable. |
| Target-typed `new()` | Introduced in **C# 9.0**; version-specific. |
| `required` members with initializers | Introduced in **C# 11**; version-specific. |
| `init` accessors with initializers | Introduced in **C# 9.0**; version-specific. |
| Collection expressions (`[...]`) | Introduced in **C# 12**; alternate syntax to collection initializers. |
| `Property = { ... }` without `new` | Available for nested object initialization; modifies the existing instance. |
| `Add` method requirement for collection initializers | **Language rule**; must implement `IEnumerable` and have an accessible `Add`. |

---

## References (Consolidated)

- Object and collection initializers — C# Programming Guide - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/object-and-collection-initializers
- Anonymous types — C# Programming Guide - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/anonymous-types
- IDE0017: Use object initializers — .NET code analysis - https://learn.microsoft.com/en-us/dotnet/fundamentals/code-analysis/style-rules/ide0017
- IDE0028: Use collection initializers — .NET code analysis - https://learn.microsoft.com/en-us/dotnet/fundamentals/code-analysis/style-rules/ide0028
- How to initialize a dictionary with a collection initializer — C# - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/how-to-initialize-a-dictionary-with-a-collection-initializer
- new operator — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/new-operator
- Target-typed new expressions — C# 9.0 new features - https://learn.microsoft.com/en-us/dotnet/csharp/whats-new/csharp-9#target-typed-new
- Anonymous object creation expressions — C# language specification - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/language-specification/expressions

---