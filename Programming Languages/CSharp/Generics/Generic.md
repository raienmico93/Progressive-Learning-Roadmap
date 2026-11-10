# C# Generics Fundamentals — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
Generics is a C# language feature that allows you to define **classes, structs, interfaces, methods, and delegates with placeholders for data types**. Instead of committing to a specific type (like `int` or `string`) at design time, you write your code with a **type parameter** (commonly `T`), and the caller supplies the **actual type argument** when the type or method is used. This lets you write a single, reusable implementation that works safely and efficiently with any data type—without casting, boxing, or runtime type errors.

**Technical Definition**
Generics introduce the concept of **type parameters** to .NET. Generics make it possible to design classes and methods that **defer the specification of one or more types** until the class or method is declared and instantiated by client code. A **type parameter** is a placeholder for a specific type that a client specifies when creating an instance of the generic type. Generic types provide **compile-time type safety**, **code reusability**, and **performance benefits** by avoiding unnecessary type conversions and boxing. The C# compiler replaces the type parameter with the actual type during compilation, ensuring type safety and avoiding runtime errors. Generics are used pervasively in .NET: collections (`List<T>`), delegates (`Func<T>`, `Action<T>`), asynchronous return types (`Task<T>`), and LINQ all rely on generic types.

**Beginner-Friendly Explanation**
Generics are like a **box with a label that says “I hold [something]”**. You do not decide whether the box holds toys, books, or clothes until you actually use it. Write the box once, and it works for anything. Instead of writing separate classes for `ListOfInts`, `ListOfStrings`, and `ListOfCats`, you write `List<T>` once and say `List<int>`, `List<string>`, or `List<Cat>` when you need them. The compiler checks that you only put the right kind of thing in the box, so you get **safety** without sacrificing **flexibility** or **speed**.

---

### Key Characteristics

- **Type parameters as placeholders** — `T`, `TKey`, `TValue`, etc. stand in for types until the caller supplies type arguments.
- **Compile-time type safety** — The compiler enforces that only compatible types are used with a generic type or method.
- **Code reusability** — One generic implementation serves many types, eliminating duplication.
- **Performance** — Generics avoid boxing, unboxing, and runtime casts for value types, improving speed.
- **Full runtime type information** — Unlike Java’s type erasure, C# generics retain complete type information at runtime.
- **Constraints refine capabilities** — The `where` clause restricts the types that can be used as type arguments, enabling access to specific members.
- **Generic delegates simplify code** — `Func<T>` and `Action<T>` replace many custom delegate declarations.
- **Variance support** — Covariance (`out`) and contravariance (`in`) enable flexible assignments of generic types (advanced).

---

### Prerequisites

- Basic understanding of C# classes, structs, interfaces, methods, and delegates.
- Familiarity with the type system (value types vs. reference types).
- Knowledge of collections (`List<T>`, `Dictionary<TKey,TValue>`).
- Awareness of casting, boxing, and unboxing (to appreciate the performance benefits).
- Basic understanding of interfaces and inheritance (for constraints).

---

### Related Programming Areas

- **Collections** — The most common use of generics is creating type-safe collection classes.
- **LINQ** — Generic sequences (`IEnumerable<T>`) and generic delegates (`Func<T>`) power LINQ.
- **Asynchronous programming** — `Task<T>` is a generic type representing an async result.
- **Delegates and events** — `Action<T>`, `Func<T>`, and `EventHandler<T>` are generic.
- **Dependency injection** — Generic interfaces and classes are widely used in DI containers.
- **Design patterns** — Repository patterns, factories, and strategies are often implemented with generics.

---

### Core Concepts / Features

1. Generics (Overarching Concept)
2. Generic Classes
3. Generic Interfaces
4. Generic Methods
5. Generic Delegates (`Func<T>`, `Action<T>`)
6. Type Parameters
7. Type Arguments
8. Generic Constraints
9. `where` Clause
10. Generic Best Practices

Each concept follows a uniform structure: definitions, purposes, syntax rules and structure, annotated code examples with expected outputs, real-world cases, and references.

---

## 1. Generics (Overarching Concept)

### Definitions

**Core Definition**
Generics is the language mechanism that lets you parameterise types and methods over the data types they operate on.

**Technical Definition**
Generics were added to C# 2.0 and the CLR. They introduce **type parameters** that make it possible to design classes and methods that defer the specification of one or more types until the class or method is declared and instantiated by client code. Generic types maximize **code reuse**, **type safety**, and **performance**. You can create your own generic interfaces, classes, methods, events, and delegates. Generic classes may be constrained to enable access to methods on particular data types.

**Beginner-Friendly Explanation**
Generics let you write **one piece of code that works with many types**. You write `Box<T>` instead of `BoxInt`, `BoxString`, and `BoxCat`.

---

### Purposes

- To maximize **code reuse** by writing type-agnostic implementations.
- To ensure **compile-time type safety** and eliminate runtime casts and `InvalidCastException`.
- To improve **performance** by avoiding boxing and unboxing of value types.
- To provide a uniform programming model for collections, delegates, and algorithms.
- To enable **type-safe** APIs that adapt to the caller’s data type.

---

### Syntax Rules and Structure

```
// Generic class
public class Box<T> { public T Item { get; set; } }

// Generic method
public T GetFirst<T>(List<T> items) => items[0];

// Generic interface
public interface IRepository<T> { T GetById(int id); }

// Generic delegate
public delegate TResult Transformer<T, TResult>(T input);
```

**Component breakdown**

- `<T>` — Type parameter list; introduces one or more placeholders.
- `Box<T>` — Generic type definition.
- `Box<int>` — Constructed generic type (type argument supplied).
- The compiler replaces `T` with the actual type at compile time.

#### Syntax Rules

- Type parameters are enclosed in angle brackets `< >`.
- Type parameter names conventionally start with `T` (e.g., `T`, `TKey`, `TValue`).
- A generic type or method can have multiple type parameters.
- Type arguments are supplied when the generic type or method is used.

#### Constraints and Limitations

- Generics do not support all operations by default; constraints are required to access members of `T`.
- Generic types cannot be used with `dynamic` in the same way as non-generic types.
- Some runtime scenarios (reflection, serialization) require special handling for generics.

---

### Annotated Code Examples and Expected Outputs

#### Example: A Generic `Box<T>` Class

```csharp
using System;

public class Box<T>
{
    public T Item { get; set; }
    public void AddItem(T item) { Item = item; }
}

class Program
{
    static void Main()
    {
        var intBox = new Box<int> { Item = 42 };
        var stringBox = new Box<string> { Item = "Hello" };

        Console.WriteLine(intBox.Item);      // 42
        Console.WriteLine(stringBox.Item);   // Hello
    }
}
```

**Expected Output**

```
42
Hello
```

**Why this output occurs**

- The same `Box<T>` class is instantiated with `int` and `string`; the compiler generates type-safe code for each closed generic type.

---

### Real-World Cases

- **`List<T>`** — The most common generic collection.
- **`Dictionary<TKey,TValue>`** — Type-safe key-value mapping.
- **`Task<T>`** — Asynchronous result of type `T`.
- **`Func<T, TResult>`** — Generic delegate for functions.

---

### References

- Generics (C# Programming Guide) — Microsoft Learn - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/generics/
- Examine generics — Training - https://learn.microsoft.com/en-us/training/modules/get-started-generic-anonymous-types/2-examine-generics
- Generics in .NET — Microsoft Learn - https://learn.microsoft.com/en-us/dotnet/standard/generics/

---

## 2. Generic Classes

### Definitions

**Core Definition**
A generic class is a class declared with one or more **type parameters**, allowing it to store and manipulate data of any type specified by the caller.

**Technical Definition**
A generic class is a class that is declared with type parameters. The type parameters act as placeholders for the types of fields, properties, method parameters, and return values. When the class is instantiated with specific type arguments (e.g., `Box<int>`), the compiler creates a **closed constructed generic type** with those types substituted. Generic classes can implement generic interfaces and can be constrained to require certain capabilities from their type arguments.

**Beginner-Friendly Explanation**
A generic class is a **reusable blueprint** that works with any type. You write it once with `T`, and use it as `MyClass<int>`, `MyClass<string>`, etc.

---

### Purposes

- To create **type-safe containers** (e.g., `List<T>`, `Stack<T>`).
- To implement **reusable algorithms** and data structures without duplication.
- To avoid boxing and casting for value types.
- To provide **compile-time type checking** for stored data.
- To enable **type-safe** APIs that adapt to the caller’s needs.

---

### Syntax Rules and Structure

```
[access modifier] class ClassName<T1, T2, ...>
{
    // members using T1, T2, ...
}
```

**Component breakdown**

- `ClassName<T>` — Generic class declaration.
- `T` — Type parameter(s).
- Members can use `T` as a field type, property type, parameter type, or return type.

#### Syntax Rules

- A generic class can have one or more type parameters.
- Type parameters can be used anywhere a type is expected within the class.
- The class can implement generic interfaces.
- Constraints can be applied with a `where` clause.

#### Constraints and Limitations

- You cannot use `T` as a base class unless a constraint guarantees it.
- Static members on generic types require the type argument to be specified when called (CA1000).
- Excessive type parameters reduce usability (CA1005 recommends no more than two).

---

### Annotated Code Examples and Expected Outputs

#### Example: Generic `Repository<T>` Class

```csharp
using System;
using System.Collections.Generic;

public class Repository<T>
{
    private readonly List<T> _items = new();

    public void Add(T item) => _items.Add(item);

    public T Get(int index) => _items[index];

    public int Count => _items.Count;
}

class Program
{
    static void Main()
    {
        var names = new Repository<string>();
        names.Add("Alice");
        names.Add("Bob");

        Console.WriteLine(names.Get(0));   // Alice
        Console.WriteLine(names.Count);    // 2
    }
}
```

**Expected Output**

```
Alice
2
```

**Why this output occurs**

- `Repository<T>` is instantiated with `string`; the internal `List<T>` becomes `List<string>`, providing type-safe storage.

---

### Real-World Cases

- **`List<T>`, `Stack<T>`, `Queue<T>`** — Framework generic collections.
- **`Repository<T>`** — Data access abstraction.
- **`Cache<TKey, TValue>`** — Type-safe caching.

---

### References

- Generic classes — C# Programming Guide - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/generics/generic-classes
- Generics (C# Programming Guide) — Overview - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/generics/

---

## 3. Generic Interfaces

### Definitions

**Core Definition**
A generic interface is an interface declared with one or more **type parameters**, defining a **type-safe contract** that implementing classes or structs must fulfil for a specific type.

**Technical Definition**
Generic interfaces provide **type-safe counterparts** to non-generic interfaces for ordering and equality comparisons and for functionality shared by generic collection types. It is often useful to define interfaces either for generic collection classes or for the generic classes that represent items in the collection. Generic classes can implement generic interfaces or closed constructed interfaces as long as the class parameter list supplies all arguments required by the interface. Commonly used generic interfaces in .NET include `IEnumerable<T>`, `IComparer<T>`, and `IEqualityComparer<T>`.

**Beginner-Friendly Explanation**
A generic interface is a **contract that adapts to a type**. `IRepository<T>` says “any repository that works with type `T` must provide these methods.”

---

### Purposes

- To define **type-safe contracts** for generic types.
- To avoid boxing and unboxing operations on value types.
- To enable **generic algorithms** that work across multiple types.
- To provide a uniform abstraction for collections and comparers.
- To support **dependency injection** with generic abstractions.

---

### Syntax Rules and Structure

```
[access modifier] interface IInterfaceName<T>
{
    T Get(int id);
    void Save(T entity);
}
```

**Component breakdown**

- `IInterfaceName<T>` — Generic interface declaration.
- `T` — Type parameter used in member signatures.
- Implementing types supply the type argument (e.g., `class Repo : IRepository<Customer>`).

#### Syntax Rules

- Generic interfaces can have one or more type parameters.
- Implementing classes must supply all type arguments required by the interface.
- Generic interfaces can inherit from other generic interfaces.
- Constraints can be applied to interface type parameters.

#### Constraints and Limitations

- Implementing a generic interface with a closed constructed type is common; open generic implementation is possible in generic classes.
- Interfaces cannot contain instance fields.

---

### Annotated Code Examples and Expected Outputs

#### Example: Generic `IRepository<T>` Interface

```csharp
using System;

public interface IRepository<T>
{
    T GetById(int id);
    void Save(T entity);
}

public class InMemoryRepository<T> : IRepository<T>
{
    private readonly System.Collections.Generic.Dictionary<int, T> _store = new();

    public T GetById(int id) => _store[id];

    public void Save(T entity) =>
        _store[entity!.GetHashCode()] = entity;
}

class Program
{
    static void Main()
    {
        IRepository<string> repo = new InMemoryRepository<string>();
        repo.Save("Hello");
        Console.WriteLine(repo.GetById("Hello".GetHashCode()));   // Hello
    }
}
```

**Expected Output**

```
Hello
```

**Why this output occurs**

- `InMemoryRepository<string>` implements `IRepository<string>`, supplying the type argument `string` for `T`.

---

### Real-World Cases

- **`IEnumerable<T>`** — Type-safe iteration.
- **`IComparer<T>`** — Type-safe comparison.
- **`IEqualityComparer<T>`** — Type-safe equality.
- **`IRepository<T>`** — Data access abstraction.

---

### References

- Generic interfaces — C# Programming Guide - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/generics/generic-interfaces
- Generic interfaces — .NET overview - https://github.com/dotnet/docs/blob/main/docs/standard/generics/interfaces.md
- Explore advanced generics concepts — Training - https://learn.microsoft.com/en-us/training/modules/get-started-generic-anonymous-types/4-explore-advanced-generics-concepts

---

## 4. Generic Methods

### Definitions

**Core Definition**
A generic method is a method declared with its own **type parameters**, allowing it to operate on different types without being part of a generic class.

**Technical Definition**
A generic method is a method that is declared with type parameters. The type parameters are specified in angle brackets after the method name, before the parameter list. Generic methods can be declared in both generic and non-generic classes. Within a generic class, non-generic methods can access the class-level type parameters; generic methods introduce their own type parameters. Generic methods can be overloaded on several type parameters. The compiler can often infer the type arguments from the method arguments, eliminating the need to specify them explicitly.

**Beginner-Friendly Explanation**
A generic method is a **method that works with any type**. You call `GetFirst<int>(list)` or `GetFirst(list)` and the compiler figures out `T`.

---

### Purposes

- To write **type-safe utility methods** that work with any type.
- To avoid duplicating logic for different types.
- To enable **generic algorithms** (sorting, searching, swapping).
- To support **LINQ** and other functional patterns.
- To leverage **type inference** for concise call sites.

---

### Syntax Rules and Structure

```
[access modifier] ReturnType MethodName<T>(parameters)
{
    // body using T
}
```

**Component breakdown**

- `MethodName<T>` — Generic method declaration.
- `T` — Method-level type parameter.
- Type arguments can be supplied explicitly (`Method<int>(...)`) or inferred (`Method(...)`).

#### Syntax Rules

- Type parameters are declared after the method name.
- The type parameter can be used in parameters and return type.
- Type inference works from the method arguments.
- Constraints can be applied with a `where` clause.

#### Constraints and Limitations

- Type inference may fail in ambiguous cases, requiring explicit type arguments.
- Generic methods cannot be `virtual` or `abstract` in the same way as non-generic methods (though they can be overridden if the class is generic).

---

### Annotated Code Examples and Expected Outputs

#### Example: Generic `GetFirstItem<T>` Method

```csharp
using System;
using System.Collections.Generic;

class Program
{
    static T GetFirstItem<T>(List<T> items)
    {
        if (items == null || items.Count == 0)
            throw new ArgumentException("List is empty", nameof(items));
        return items[0];
    }

    static void Main()
    {
        var names = new List<string> { "Hannah", "Mario" };
        string firstName = GetFirstItem(names);
        Console.WriteLine(firstName);   // Hannah

        var numbers = new List<int> { 10, 20 };
        Console.WriteLine(GetFirstItem(numbers));   // 10
    }
}
```

**Expected Output**

```
Hannah
10
```

**Why this output occurs**

- The compiler infers `T` as `string` for the first call and `int` for the second.

---

### Real-World Cases

- **LINQ operators** — `Enumerable.Where<T>`, `Enumerable.Select<T>`.
- **Utility methods** — `Swap<T>`, `Max<T>`, `Min<T>`.
- **Factory methods** — `Activator.CreateInstance<T>()`.

---

### References

- Generic methods — C# Programming Guide - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/generics/generic-methods
- Generic methods — .NET overview (GitHub) - https://github.com/dotnet/docs/blob/main/docs/csharp/programming-guide/generics/generic-methods.md

---

## 5. Generic Delegates (`Func<T>`, `Action<T>`)

### Definitions

**Core Definition**
A generic delegate is a delegate type declared with **type parameters**, enabling a single delegate definition to represent methods with different type signatures. .NET provides two built-in families: `Action<T>` (no return value) and `Func<T, TResult>` (returns a value).

**Technical Definition**
Strongly typed delegates simplify working with delegates by allowing you to use generic types instead of defining custom delegate types for each method signature. The **`Action`** delegate represents a method that takes up to 16 parameters and **does not return a value**; it is used when you want to perform an operation but do not need a result. The **`Func`** delegate represents a method that takes up to 16 parameters and **returns a value**. These generic delegates can be used to define methods with different signatures without the need for custom delegate types. .NET provides two families of generic delegate types, `Func` and `Action`, that cover most scenarios, so you rarely need to use the `delegate` keyword yourself.

**Beginner-Friendly Explanation**
`Action<T>` and `Func<T, TResult>` are **ready-made delegate types** that save you from writing `delegate` declarations. `Action` is for “do something, return nothing”; `Func` is for “do something, return a value.”

---

### Purposes

- To **eliminate custom delegate declarations** for common signatures.
- To provide a **consistent, standard** delegate vocabulary.
- To improve **readability** and **maintainability** of code that uses callbacks and lambdas.
- To enable **functional programming** patterns in C#.
- To support **LINQ** and event handling.

---

### Syntax Rules and Structure

```
// Action: no return value
Action action;                       // no parameters
Action<int> action1;                 // one int parameter
Action<int, string> action2;        // two parameters

// Func: returns a value (last type parameter is the return type)
Func<int> func0;                     // no parameters, returns int
Func<int, string> func1;            // one int parameter, returns string
Func<int, int, bool> func2;        // two int parameters, returns bool
```

**Component breakdown**

- `Action` — Delegate with no return value.
- `Func<TResult>` — Delegate returning `TResult`.
- `Func<T, TResult>` — Delegate taking `T` and returning `TResult`.

#### Syntax Rules

- `Action` variants take 0 to 16 parameters and return `void`.
- `Func` variants take 0 to 16 parameters and return a value (the last type parameter).
- `Predicate<T>` is a specialized generic delegate returning `bool`.
- `Comparison<T>` is a specialized generic delegate for comparing two objects.

#### Constraints and Limitations

- The maximum number of parameters for built-in `Action`/`Func` is 16.
- Custom generic delegates can be declared if needed, but built-in types cover most scenarios.

---

### Annotated Code Examples and Expected Outputs

#### Example: `Action<T>` and `Func<T>` in Practice

```csharp
using System;

class Program
{
    static void Main()
    {
        // Action: performs an operation, returns nothing
        Action<string> log = message => Console.WriteLine($"[LOG] {message}");
        log("Started");   // [LOG] Started

        // Func: takes input, returns a value
        Func<int, int, int> add = (a, b) => a + b;
        Console.WriteLine(add(3, 4));   // 7

        // Func with no parameters
        Func<DateTime> now = () => DateTime.Now;
        Console.WriteLine(now() is DateTime);   // True
    }
}
```

**Expected Output**

```
[LOG] Started
7
True
```

**Why this output occurs**

- `Action<string>` encapsulates a method that takes a `string` and returns nothing; `Func<int,int,int>` encapsulates a method that takes two `int`s and returns an `int`.

---

### Real-World Cases

- **LINQ** — `Func<T, bool>` for `Where`, `Func<T, TResult>` for `Select`.
- **Event handling** — `Action<object, EventArgs>` for event handlers.
- **Callbacks** — Passing `Action` or `Func` to asynchronous APIs.
- **Comparisons** — `Comparison<T>` for sorting.

---

### References

- Examine the Action and Func delegates — Training - https://learn.microsoft.com/en-us/training/modules/get-started-delegates/04-examine-strongly-typed-delegates
- Strongly Typed Delegates — .NET (GitHub) - https://github.com/dotnet/docs/blob/main/docs/csharp/delegates-strongly-typed.md
- Generic delegates — C# Programming Guide - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/generics/generic-delegates

---

## 6. Type Parameters

### Definitions

**Core Definition**
A type parameter is a **placeholder identifier** (such as `T`) that represents a type that will be specified by the caller when the generic type or method is used.

**Technical Definition**
In a generic type or method definition, a type parameter is a placeholder for a specific type that a client specifies when they create an instance of the generic type. Type parameters are declared in angle brackets (`<T>`) and are used throughout the generic definition wherever a type is expected. A generic type parameter is a **compile-time construct**; the compiler replaces it with the actual type argument during compilation. Type parameters can have **constraints** that restrict the types that can be used as arguments.

**Beginner-Friendly Explanation**
`T` is a **blank** that gets filled in with a real type when you use the generic type or method. It is a placeholder.

---

### Purposes

- To **abstract over types** in generic definitions.
- To enable **type-safe** reuse across multiple types.
- To allow **constraints** that refine the capabilities available to `T`.
- To make the generic definition **readable** and **self-documenting** (e.g., `TKey`, `TValue`).

---

### Syntax Rules and Structure

```
// Single type parameter
class Box<T> { }

// Multiple type parameters
class Dictionary<TKey, TValue> { }

// Type parameter with constraint
class Repository<T> where T : class { }
```

**Component breakdown**

- `T`, `TKey`, `TValue` — Type parameter identifiers.
- `<T>` — Type parameter list.
- `where T : constraint` — Optional constraint.

#### Syntax Rules

- Type parameter names conventionally start with `T`.
- Multiple type parameters are separated by commas.
- Type parameters can be used as field types, property types, parameter types, and return types.
- Constraints are specified with the `where` clause.

#### Constraints and Limitations

- Type parameters cannot be used as base classes without constraints.
- Type parameters cannot be `static`.
- The number of type parameters should be limited (CA1005 recommends no more than two).

---

### Annotated Code Examples and Expected Outputs

#### Example: Type Parameters in a Generic Class

```csharp
using System;

public class Pair<TFirst, TSecond>
{
    public TFirst First { get; }
    public TSecond Second { get; }

    public Pair(TFirst first, TSecond second)
    {
        First = first;
        Second = second;
    }

    public override string ToString() => $"({First}, {Second})";
}

class Program
{
    static void Main()
    {
        var pair = new Pair<string, int>("Alice", 30);
        Console.WriteLine(pair);   // (Alice, 30)
    }
}
```

**Expected Output**

```
(Alice, 30)
```

**Why this output occurs**

- `TFirst` is replaced with `string` and `TSecond` with `int` when the `Pair` is instantiated.

---

### Real-World Cases

- **`Dictionary<TKey, TValue>`** — `TKey` and `TValue` are type parameters.
- **`Func<T, TResult>`** — `T` and `TResult` are type parameters.
- **`Repository<T>`** — `T` is the entity type.

---

### References

- Generic type parameters — C# Programming Guide - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/generics/generic-type-parameters
- Generic type parameters — .NET (GitHub) - https://github.com/dotnet/docs/blob/main/docs/csharp/programming-guide/generics/generic-type-parameters.md

---

## 7. Type Arguments

### Definitions

**Core Definition**
A type argument is the **actual type** supplied by the caller for a type parameter when a generic type or method is used.

**Technical Definition**
When a generic type or method is used, the caller provides **type arguments** in angle brackets (e.g., `List<int>`). The compiler substitutes the type arguments for the type parameters, creating a **closed constructed generic type**. If the type argument is omitted for a generic method, the compiler attempts **type inference** from the method arguments. Type arguments must satisfy any constraints declared on the type parameters.

**Beginner-Friendly Explanation**
A type argument is the **real type you put in the blank**. `List<int>` uses `int` as the type argument for `T`.

---

### Purposes

- To **specialize** a generic type or method for a specific type.
- To enable **compile-time type checking** for the constructed type.
- To support **type inference** for concise method calls.
- To satisfy **constraints** declared on type parameters.

---

### Syntax Rules and Structure

```
// Explicit type argument
List<int> numbers = new List<int>();

// Type inference for generic method
var first = GetFirstItem(names);   // T inferred as string

// Multiple type arguments
Dictionary<string, int> ages = new();
```

**Component breakdown**

- `<int>` — Type argument for `T`.
- `<string, int>` — Multiple type arguments.
- Inferred type arguments — Determined by the compiler from method arguments.

#### Syntax Rules

- Type arguments are supplied in angle brackets after the generic type or method name.
- Type arguments must satisfy any constraints on the type parameters.
- For generic methods, type inference may eliminate the need for explicit type arguments.

#### Constraints and Limitations

- Type inference may fail in ambiguous cases.
- Type arguments must be accessible at the call site.
- Value types and reference types are both valid type arguments (subject to constraints).

---

### Annotated Code Examples and Expected Outputs

#### Example: Explicit and Inferred Type Arguments

```csharp
using System;
using System.Collections.Generic;

class Program
{
    static T GetFirstItem<T>(List<T> items) => items[0];

    static void Main()
    {
        // Explicit type argument
        var firstExplicit = GetFirstItem<int>(new List<int> { 10, 20 });
        Console.WriteLine(firstExplicit);   // 10

        // Inferred type argument
        var firstInferred = GetFirstItem(new List<string> { "Alice", "Bob" });
        Console.WriteLine(firstInferred);   // Alice
    }
}
```

**Expected Output**

```
10
Alice
```

**Why this output occurs**

- The explicit call specifies `int`; the inferred call deduces `string` from the list argument.

---

### Real-World Cases

- **All generic type usage** — Type arguments are required for constructed generic types.
- **LINQ method calls** — Type inference is common (`Where(x => x > 5)`).
- **Factory methods** — `Activator.CreateInstance<T>()` uses type arguments.

---

### References

- Generic type parameters — C# Programming Guide (type arguments) - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/generics/generic-type-parameters
- Generics (C# Programming Guide) — Overview - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/generics/

---

## 8. Generic Constraints

### Definitions

**Core Definition**
Generic constraints are **restrictions** placed on type parameters that specify what capabilities a type argument must have, enabling the generic code to use those capabilities safely.

**Technical Definition**
Constraints declare the **capabilities that the type argument must have**. Without constraints, `T` is treated as `object`, and only `object` members are available. Constraints are specified with the `where` clause. The main constraint types are: **`where T : struct`** (non-nullable value type), **`where T : class`** (non-nullable reference type), **`where T : class?`** (nullable or non-nullable reference type), **`where T : notnull`** (non-nullable type), **`where T : unmanaged`** (unmanaged type), **`where T : new()`** (public parameterless constructor), **`where T : base-class`** (derived from or is the base class), **`where T : interface`** (implements the interface), and **`where T : U`** (derived from or is `U`). Multiple constraints can be combined, with the base class constraint appearing first.

**Beginner-Friendly Explanation**
Constraints are **rules about what `T` can be**. `where T : IComparable<T>` says “T must be comparable.” This lets you call `CompareTo` inside your generic code.

---

### Purposes

- To **enable access to members** of `T` (methods, properties, operators).
- To ensure **type safety** by restricting valid type arguments.
- To **document requirements** clearly in the API.
- To support **generic algorithms** (sorting, comparison, arithmetic).
- To avoid runtime casts and reflection.

---

### Syntax Rules and Structure (Constraint Matrix)

| Constraint | Meaning |
|---|---|
| `where T : struct` | `T` must be a non-nullable value type. |
| `where T : class` | `T` must be a non-nullable reference type. |
| `where T : class?` | `T` may be a nullable or non-nullable reference type. |
| `where T : notnull` | `T` must be a non-nullable type. |
| `where T : unmanaged` | `T` must be an unmanaged type. |
| `where T : new()` | `T` must have a public parameterless constructor. |
| `where T : BaseClass` | `T` must derive from or be `BaseClass`. |
| `where T : IInterface` | `T` must implement `IInterface`. |
| `where T : U` | `T` must derive from or be `U`. |

**Component breakdown**

- `where` — The contextual keyword introducing constraints.
- `T` — The type parameter being constrained.
- `:` — Separator.
- Constraint — The restriction (struct, class, interface, base class, new(), etc.).

#### Syntax Rules

- The `where` clause appears after the base class or implemented interfaces in a class declaration.
- Multiple constraints can be applied to a single type parameter.
- The base class constraint must appear before interface constraints.
- The `new()` constraint must appear last.
- Constraints can be applied to classes, methods, delegates, and local functions.

#### Constraints and Limitations

- `System.Object`, `System.Array`, and `System.ValueType` cannot be used as base class constraints.
- `struct` implicitly includes `new()` but not vice versa.
- `unmanaged` implies `struct`.
- Constraint combinations must be valid (e.g., `struct` and `class` are mutually exclusive).

---

### Annotated Code Examples and Expected Outputs

#### Example: `IComparable<T>` Constraint

```csharp
using System;

public static class Comparer
{
    public static T Max<T>(T a, T b) where T : IComparable<T>
    {
        return a.CompareTo(b) >= 0 ? a : b;
    }
}

class Program
{
    static void Main()
    {
        Console.WriteLine(Comparer.Max(3, 7));          // 7
        Console.WriteLine(Comparer.Max("apple", "banana")); // banana
    }
}
```

**Expected Output**

```
7
banana
```

**Why this output occurs**

- The `IComparable<T>` constraint allows `CompareTo` to be called on `T`; `int` and `string` both implement `IComparable<T>`.

---

### Real-World Cases

- **`IComparable<T>`** — Sorting and comparison algorithms.
- **`new()`** — Factory methods that create instances of `T`.
- **`class` / `struct`** — Distinguishing reference and value types.
- **`unmanaged`** — Interop and low-level memory scenarios.

---

### References

- Constraints on type parameters — C# Programming Guide - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/generics/constraints-on-type-parameters
- where (generic type constraint) — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/where-generic-type-constraint

---

## 9. `where` Clause

### Definitions

**Core Definition**
The `where` clause is the **syntax** used to specify constraints on generic type parameters.

**Technical Definition**
In a generic definition, use the `where` clause to specify constraints on the types that you use as arguments for type parameters in a generic type, method, delegate, or local function. The `where` clause declares capabilities that the type argument must have. It can specify interfaces, base classes, or require a generic type to be a reference, value, or unmanaged type. The `where` clause appears after any declared base class or implemented interfaces.

**Beginner-Friendly Explanation**
`where` is the **keyword that starts a constraint rule**. `where T : IComparable<T>` means “T must be comparable.”

---

### Purposes

- To **declare constraints** on type parameters.
- To enable **member access** on `T`.
- To **document API requirements**.
- To **restrict** the set of valid type arguments.
- To support **generic algorithms** that require specific capabilities.

---

### Syntax Rules and Structure

```
where T : constraint1, constraint2, ...
```

**Component breakdown**

- `where` — The contextual keyword.
- `T` — The type parameter.
- `:` — Separator.
- `constraint1, constraint2, ...` — One or more constraints.

#### Syntax Rules

- The `where` clause appears after the base class/interface list in a class declaration.
- Multiple `where` clauses can be used for multiple type parameters.
- Constraints within a single `where` clause are comma-separated.
- The base class constraint must come first; the `new()` constraint must come last.

#### Constraints and Limitations

- The `where` keyword is also used in LINQ query expressions (a different context).
- Invalid constraint combinations cause compile-time errors.

---

### Annotated Code Examples and Expected Outputs

#### Example: Multiple Constraints

```csharp
using System;

public class Factory<T> where T : class, IComparable<T>, new()
{
    public T Create() => new T();
}

public class Widget : IComparable<Widget>
{
    public int CompareTo(Widget? other) => 0;
}

class Program
{
    static void Main()
    {
        var factory = new Factory<Widget>();
        var widget = factory.Create();
        Console.WriteLine(widget is Widget);   // True
    }
}
```

**Expected Output**

```
True
```

**Why this output occurs**

- `Widget` satisfies all three constraints: it is a `class`, implements `IComparable<Widget>`, and has a parameterless constructor.

---

### Real-World Cases

- **Generic repositories** — `where T : class, IEntity`.
- **Math libraries** — `where T : INumber<T>` (C# 11+).
- **Factory methods** — `where T : new()`.

---

### References

- where (generic type constraint) — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/where-generic-type-constraint
- Constraints on type parameters — C# Programming Guide - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/generics/constraints-on-type-parameters

---

## 10. Generic Best Practices

### Definitions

**Core Definition**
Generic best practices are the design guidelines that ensure generic types and methods are **usable, performant, and maintainable**, while avoiding common pitfalls such as excessive type parameters and misplaced static members.

**Technical Definition**
Best practices for generics derive from the Framework Design Guidelines and code analysis rules. Key recommendations include: **use generic types to maximize code reuse, type safety, and performance**; **use generic collections whenever possible** instead of non-generic collections; **avoid declaring static members on generic types** (CA1000) because the type argument must be specified for every call, creating confusing syntax; **avoid excessive type parameters** (CA1005) because more than two type parameters make the type difficult to understand; **apply constraints to enable member access** and document requirements; **prefer generic methods for algorithms** that are independent of the containing type; and **use built-in generic delegates** (`Func<T>`, `Action<T>`) instead of custom delegate types.

**Beginner-Friendly Explanation**
Best practices for generics are simple: **use them for collections and reusable algorithms**, **do not add too many type parameters**, **do not put static members on generic types**, and **use `Func`/`Action` instead of writing your own delegates**.

---

### Purposes

- To maximize the benefits of generics (reuse, safety, performance).
- To avoid confusing or hard-to-use generic APIs.
- To align with .NET design conventions and code analysis rules.
- To ensure generic code is maintainable and discoverable.
- To choose the right generic mechanism for the scenario.

---

### Guidelines and Structure

#### General Guidelines

```
// USE generic types for reusable, type-safe code
public class Repository<T> { ... }

// USE generic collections over non-generic
List<int> numbers = new();   // not ArrayList

// AVOID static members on generic types (CA1000)
// Instead, use a non-generic static class or instance members

// AVOID excessive type parameters (CA1005) — keep to ≤ 2
// Prefer Dictionary<TKey,TValue> style (2 parameters)

// USE built-in generic delegates (Func, Action)
Func<int, bool> isPositive = n => n > 0;

// APPLY constraints to enable member access
public T Max<T>(T a, T b) where T : IComparable<T> => ...;
```

**Component breakdown**

- **Generic collections** — Prefer `List<T>` over `ArrayList`.
- **No static members on generic types** — CA1000 recommends removing or converting to instance members.
- **Limited type parameters** — CA1005 recommends no more than two.
- **Built-in delegates** — `Func` and `Action` cover most scenarios.
- **Constraints** — Enable access to members of `T`.

#### Rules and Constraints

- **Do** use generic types for code reuse, type safety, and performance.
- **Do** use generic collections over non-generic collections.
- **Do** use constraints to enable member access and document requirements.
- **Do** use `Func<T>` and `Action<T>` instead of custom delegate types.
- **Do not** declare static members on generic types (CA1000).
- **Do not** use more than two type parameters on externally visible generic types (CA1005).
- **Do not** use generics when a non-generic solution is simpler and sufficient.

---

### Annotated Code Examples and Expected Outputs

#### Example: Well-Designed Generic API

```csharp
using System;

// Constraint enables member access
public static class CollectionHelper
{
    public static T Max<T>(T a, T b) where T : IComparable<T>
        => a.CompareTo(b) >= 0 ? a : b;
}

class Program
{
    static void Main()
    {
        Console.WriteLine(CollectionHelper.Max(10, 20));    // 20
        Console.WriteLine(CollectionHelper.Max("A", "B"));  // B
    }
}
```

**Expected Output**

```
20
B
```

**Why this output occurs**

- The `IComparable<T>` constraint allows `CompareTo` to be called on any comparable type, making the generic method broadly reusable and type-safe.

---

### Real-World Cases

- **.NET Framework collections** — `List<T>`, `Dictionary<TKey,TValue>`.
- **LINQ** — Generic sequences and delegates.
- **Dependency injection** — Generic repositories and services.
- **Factory patterns** — `where T : new()` constraints.

---

### References

- Generics (C# Programming Guide) — Benefits and guidelines - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/generics/
- CA1000: Do not declare static members on generic types — .NET code analysis - https://learn.microsoft.com/en-us/dotnet/fundamentals/code-analysis/quality-rules/ca1000
- CA1005: Avoid excessive parameters on generic types — .NET code analysis - https://learn.microsoft.com/en-us/dotnet/fundamentals/code-analysis/quality-rules/ca1005
- CA1007: Use generics where appropriate — .NET code analysis - https://learn.microsoft.com/en-us/dotnet/fundamentals/code-analysis/quality-rules/ca1007

---

## Deprecated, Unsafe, Implementation-Dependent, and Version-Specific Notes

| Feature | Status / Note |
|---------|---------------|
| Generics | Introduced in **C# 2.0** and the CLR; stable and foundational. |
| `Func<T>` and `Action<T>` | Introduced in **.NET 3.5**; stable. |
| Generic math (`INumber<T>`) | Introduced in **C# 11 / .NET 7**; version-specific. |
| Static abstract interface members | Introduced in **C# 11**; enables generic math. |
| Variance (`in` / `out`) | Introduced in **C# 4.0**; stable. |
| CA1000 (no static members on generic types) | **Design rule**; not a compile error, but flagged by analyzers. |
| CA1005 (avoid excessive type parameters) | **Design rule**; recommends ≤ 2 type parameters. |
| Type erasure | **Not present** in C#; full type information is preserved at runtime. |
| `default` keyword for `T` | Use `default(T)` or `default` to get the default value of a type parameter. |

---

## References (Consolidated)

- Generics (C# Programming Guide) — Microsoft Learn - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/generics/
- Examine generics — Training - https://learn.microsoft.com/en-us/training/modules/get-started-generic-anonymous-types/2-examine-generics
- Generics in .NET — Microsoft Learn - https://learn.microsoft.com/en-us/dotnet/standard/generics/
- Generic classes — C# Programming Guide - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/generics/generic-classes
- Generic interfaces — C# Programming Guide - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/generics/generic-interfaces
- Generic methods — C# Programming Guide - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/generics/generic-methods
- Generic delegates — C# Programming Guide - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/generics/generic-delegates
- Generic type parameters — C# Programming Guide - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/generics/generic-type-parameters
- Constraints on type parameters — C# Programming Guide - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/generics/constraints-on-type-parameters
- where (generic type constraint) — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/where-generic-type-constraint
- Examine the Action and Func delegates — Training - https://learn.microsoft.com/en-us/training/modules/get-started-delegates/04-examine-strongly-typed-delegates
- CA1000: Do not declare static members on generic types — .NET code analysis - https://learn.microsoft.com/en-us/dotnet/fundamentals/code-analysis/quality-rules/ca1000
- CA1005: Avoid excessive parameters on generic types — .NET code analysis - https://learn.microsoft.com/en-us/dotnet/fundamentals/code-analysis/quality-rules/ca1005
- CA1007: Use generics where appropriate — .NET code analysis - https://learn.microsoft.com/en-us/dotnet/fundamentals/code-analysis/quality-rules/ca1007

---