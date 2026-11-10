# C# Advanced Generics — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
**Advanced generics** is the collection of C# techniques, constraints, built-in delegate families, and design patterns that elevate generics from simple type-safe containers to **powerful abstraction mechanisms**. It covers the full constraint vocabulary (`class`, `struct`, `notnull`, `unmanaged`, `new()`, base class, interfaces, type-parameter relationships), the standard generic delegate families (`Func<T>`, `Action<T>`, `Predicate<T>`, `Comparison<T>`, `Converter<TInput, TOutput>`), and the reusable patterns (repository, factory, strategy, specification, pipeline, options, etc.) that professional .NET codebases rely on.

**Technical Definition**
Advanced generics builds on the C# generic type system to provide **compile-time type safety, code reuse, and performance** without sacrificing flexibility. It encompasses: (1) **constraint composition**—combining primary, base-class, interface, and constructor constraints in the correct order; (2) **variance**—covariant (`out`) and contravariant (`in`) type parameters for interfaces and delegates; (3) **generic delegates**—the `System.Func<>`, `System.Action<>`, `System.Predicate<>`, `System.Comparison<>`, and `System.Converter<,>` families that eliminate custom delegate declarations; (4) **static abstract interface members (C# 11)**—enabling generic math (`INumber<T>`); and (5) **generic design patterns**—repository, unit of work, strategy, factory, specification, and caching patterns. Advanced generic code must respect constraints, avoid unnecessary type parameters (CA1005), and ensure that constraints represent genuine API contracts rather than accidental complexity.

**Beginner-Friendly Explanation**
Advanced generics is where generics stop being “a list that can hold anything” and become a **design superpower**. You learn how to tell the compiler exactly what `T` must be able to do, how to plug functions into other functions using built-in generic delegates, and how to write patterns that work for any entity, any service, any result. The payoff is enormous: less code, fewer bugs, better performance—but only if you use the tools **deliberately**.

---

### Key Characteristics

- **Rich constraint vocabulary** — Primary (`class`, `struct`, `notnull`, `unmanaged`), secondary (base class, interfaces), and `new()` constraints can be composed.
- **Variance support** — `out` and `in` enable flexible assignment of variant interfaces and delegates.
- **Standardised delegate families** — `Func<>`, `Action<>`, `Predicate<>`, `Comparison<>`, and `Converter<,>` replace custom delegates in most scenarios.
- **Generic math (C# 11)** — Static abstract interface members enable arithmetic on generic types.
- **Design-pattern friendly** — Repository, factory, strategy, specification, pipeline, and options patterns shine with generics.
- **Analyzer-enforced discipline** — CA1000 (static members on generic types), CA1005 (excessive type parameters), and CA1007 (use generics) guide design.
- **Performance-critical** — Generic code avoids boxing and enables JIT specialisation.

---

### Prerequisites

- Solid understanding of `List<T>`, `Dictionary<TKey,TValue>`, and generic methods.
- Familiarity with constraints (`where T : ...`) and the `new()` constraint.
- Knowledge of delegates, lambdas, and method groups.
- Awareness of interfaces, abstract classes, and inheritance.
- Basic understanding of design patterns (repository, strategy, factory).

---

### Related Programming Areas

- **LINQ** — Built almost entirely on generic delegates and generic extension methods.
- **Dependency injection** — Generic interfaces (`IRepository<T>`, `IOptions<T>`) are the backbone of DI.
- **Entity Framework Core** — `DbSet<T>`, `IRepository<T>` patterns.
- **Generic math** — `INumber<T>` and static abstract interface members (C# 11).
- **Async programming** — `Task<T>`, `ValueTask<T>`.
- **Testing** — Generic `Func`/`Action` mocks and generic test data builders.

---

### Core Concepts / Features

1. Generic Type Constraints (Advanced Composition)
2. Generic Delegates (Overview)
3. `Func<T>`
4. `Action<T>`
5. `Predicate<T>`
6. `Comparison<T>`
7. `Converter<TInput, TOutput>`
8. Generic Patterns
9. Generic Best Practices

---

## 1. Generic Type Constraints (Advanced Composition)

### Definitions

**Core Definition**
Advanced constraint composition is the art of combining multiple constraints on generic type parameters in the correct order to precisely specify capabilities required by the generic implementation.

**Technical Definition**
A type parameter can have at most one **primary constraint** (`class`, `class?`, `struct`, `unmanaged`, `notnull`, or `default`), followed by zero or more **base class and interface constraints**, and finally the **`new()` constructor constraint**. All constraints for a type parameter must appear in a single `where` clause. Invalid constraint orders cause compiler errors CS0449, CS0450, CS0451, CS0401, CS0406, or CS0409. Constraint composition also interacts with **variance**: covariant type parameters cannot be used as generic constraints for interface methods, while contravariant parameters can.

**Beginner-Friendly Explanation**
Advanced constraint composition is about **layering rules in the right order**: first the primary rule (`class` or `struct`), then the base class, then any interfaces, then `new()` last. Getting the order wrong breaks the compilation.

---

### Purposes

- To **precisely express capabilities** required of type arguments.
- To **unlock the full set of members** available on `T`.
- To **enforce compile-time contracts** for complex generic APIs.
- To **avoid runtime casts and reflection**.
- To **document API intent** through constraints.

---

### Syntax Rules and Structure

```
public class Sample<T, U>
    where T : class, IComparable<T>, new()
    where U : struct, IEquatable<U>
{
    // T is a reference type, comparable, and has a parameterless ctor
    // U is a value type and equatable
}
```

**Component breakdown**

- `where T : ...` — One constraint clause per type parameter.
- **Order** — primary → base class → interfaces → `new()`.
- Multiple `where` clauses — For different type parameters.
- **Primary constraints** — `class`, `class?`, `struct`, `unmanaged`, `notnull`, `default`.
- **Secondary constraints** — base class, interface(s).
- **Final constraint** — `new()`.

#### Syntax Rules

- `struct`, `unmanaged`, and `new()` cannot be combined with each other (struct/unmanaged imply new()).
- `class` and `struct` are mutually exclusive.
- `class` and `notnull` are mutually exclusive.
- Base class constraint must precede interface constraints.
- `new()` must be last.
- Multiple `where` clauses cannot refer to the **same** type parameter.

#### Constraints and Limitations

- Covariant (`out`) type parameters cannot be constraints for interface methods.
- Abstract classes cannot be used with `new()`.
- `System.Object`, `System.Array`, `System.ValueType` are not valid base class constraints.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Advanced Composition

```csharp
using System;

public interface IEntity { int Id { get; } }

public class Repository<T>
    where T : class, IEntity, new()
{
    public T Create() => new T();
    public void PrintId(T entity) => Console.WriteLine(entity.Id);
}

public class Customer : IEntity
{
    public int Id { get; set; } = 42;
}

class Program
{
    static void Main()
    {
        var repo = new Repository<Customer>();
        var customer = repo.Create();
        repo.PrintId(customer);   // 42
    }
}
```

**Expected Output**

```
42
```

**Why this output occurs**

- `Customer` satisfies all three constraints: it is a class, implements `IEntity`, and has a parameterless constructor.

---

#### Example 2: Multiple Type Parameters with Different Constraints

```csharp
using System;

public class Transformer<TIn, TOut>
    where TIn : struct
    where TOut : class, new()
{
    public TOut Transform(TIn input) => new TOut();
}

public class Result { public string Message = "OK"; }

class Program
{
    static void Main()
    {
        var t = new Transformer<int, Result>();
        Console.WriteLine(t.Transform(5).Message);   // OK
    }
}
```

**Expected Output**

```
OK
```

**Why this output occurs**

- `int` satisfies `struct`; `Result` satisfies `class, new()`.

---

### Real-World Cases

- **Repository pattern** — `where T : class, IEntity, new()`.
- **Generic math** — `where T : INumber<T>` (C# 11).
- **DI containers** — `where T : class`.

---

### References

- Constraints on type parameters — C# Programming Guide - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/generics/constraints-on-type-parameters
- Resolve errors related to generic type parameters — Constraint ordering - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/compiler-messages/generic-type-parameters-errors
- Generic math — .NET - https://learn.microsoft.com/en-us/dotnet/standard/generics/math

---

## 2. Generic Delegates (Overview)

### Definitions

**Core Definition**
A generic delegate is a delegate type declared with one or more type parameters, allowing the same delegate definition to represent methods with different type signatures.

**Technical Definition**
Strongly typed delegates simplify working with delegates by allowing you to use generic types instead of defining custom delegate types for each method signature. .NET provides two main families—`Func` and `Action`—covering most scenarios. Additional specialised generic delegates include `Predicate<T>`, `Comparison<T>`, and `Converter<TInput, TOutput>`. Generic delegates can be **variant** (`out` for return types, `in` for parameters). Variance in delegate binding works with **both generic and non-generic** delegates. Custom delegates can still be defined where the built-in ones do not fit.

**Beginner-Friendly Explanation**
Generic delegates are **ready-made delegate types** that cover most callback scenarios. `Func<T>` for “returns a value,” `Action<T>` for “returns nothing,” and specialist variants like `Predicate<T>` and `Comparison<T>`.

---

### Purposes

- To **avoid custom delegate declarations** for common signatures.
- To enable **type-safe callbacks** with full variance support.
- To standardise delegate usage across APIs.
- To support **functional programming** patterns in C#.
- To enable **LINQ** and other generic libraries.

---

### Syntax Rules and Structure

| Delegate | Signature | Return | Use case |
|---|---|---|---|
| `Func<TResult>` | `TResult Function()` | Value | Factories, callbacks with result |
| `Func<T, TResult>` | `TResult Function(T arg)` | Value | Transformations |
| `Func<T1, T2, TResult>` | `TResult Function(T1, T2)` | Value | Binary operations |
| `Action` | `void Action()` | `void` | Side effects |
| `Action<T>` | `void Action(T arg)` | `void` | One-arg side effects |
| `Predicate<T>` | `bool Predicate(T arg)` | `bool` | Filtering |
| `Comparison<T>` | `int Comparison(T a, T b)` | `int` | Sorting |
| `Converter<TInput, TOutput>` | `TOutput Converter(TInput input)` | `TOutput` | Type conversion |

**Component breakdown**

- `Func<>` — Up to 16 parameters + return type as the last type argument.
- `Action<>` — Up to 16 parameters, no return.
- `Predicate<T>` — Specialised `Func<T, bool>` (legacy API shape).
- `Comparison<T>` — Specialised `Func<T, T, int>` for sorting.
- `Converter<TIn, TOut>` — Specialised `Func<TIn, TOut>` for `Array.ConvertAll`.

#### Syntax Rules

- `Func` and `Action` support variance in their parameter/return positions.
- `Predicate`, `Comparison`, and `Converter` are not variant-annotated (though functionally variance flows through use).
- Custom delegates can be declared if the built-in ones do not fit.

#### Constraints and Limitations

- Maximum 16 parameters on built-in `Func`/`Action`.
- `Predicate<T>` is largely superseded by `Func<T, bool>` but retained for compatibility.
- `Converter<,>` is specialised for `Array.ConvertAll`; prefer `Func<,>` for new APIs.

---

### Annotated Code Examples and Expected Outputs

#### Example: Using All Built-In Generic Delegates

```csharp
using System;
using System.Collections.Generic;
using System.Linq;

class Program
{
    static void Main()
    {
        Func<int, int> square = x => x * x;
        Action<string> log = msg => Console.WriteLine($"[LOG] {msg}");
        Predicate<int> isEven = x => x % 2 == 0;
        Comparison<int> descending = (a, b) => b.CompareTo(a);
        Converter<string, int> toInt = s => int.Parse(s);

        log($"Square of 5: {square(5)}");           // [LOG] Square of 5: 25
        log($"Is 4 even? {isEven(4)}");             // [LOG] Is 4 even? True

        var numbers = new List<int> { 3, 1, 4, 1, 5 };
        numbers.Sort(descending);
        log(string.Join(",", numbers));             // [LOG] 5,4,3,1,1

        int[] strAsInts = Array.ConvertAll(new[] { "1", "2", "3" }, toInt);
        log(string.Join(",", strAsInts));           // [LOG] 1,2,3
    }
}
```

**Expected Output**

```
[LOG] Square of 5: 25
[LOG] Is 4 even? True
[LOG] 5,4,3,1,1
[LOG] 1,2,3
```

**Why this output occurs**

- Each generic delegate type encapsulates a lambda with a specific signature, and the APIs (`Sort`, `ConvertAll`) accept that delegate type.

---

### Real-World Cases

- **LINQ** — `Where` accepts `Func<T, bool>`; `Select` accepts `Func<T, TResult>`.
- **Sorting** — `List<T>.Sort(Comparison<T>)`.
- **Converting** — `Array.ConvertAll(Converter<TIn, TOut>)`.

---

### References

- Generic delegates — C# Programming Guide - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/generics/generic-delegates
- Strongly Typed Delegates — .NET (GitHub) - https://github.com/dotnet/docs/blob/main/docs/csharp/delegates-strongly-typed.md

---

## 3. `Func<T>`

### Definitions

**Core Definition**
`Func<T>` is a generic delegate family that represents a method returning a value. The last type parameter is the **return type**; preceding type parameters are **input parameters**.

**Technical Definition**
`System.Func<TResult>` and its multi-parameter variants (`Func<T1, TResult>` up to `Func<T1, ..., T16, TResult>`) encapsulate a method that takes zero or more input parameters and returns `TResult`. `Func<out TResult>` is **covariant** in `TResult`; input parameters are **contravariant**. `Func` is widely used for callbacks, LINQ projections, factory methods, and lazy evaluation.

**Beginner-Friendly Explanation**
`Func<T>` is “**a function that gives you something back**.” The last type argument is the return type.

---

### Purposes

- To represent **callbacks that return a value**.
- To enable **LINQ projections** (`Select`, `Any`, `All`).
- To support **factory methods** and **lazy evaluation**.
- To pass **transformations** as data.
- To enable **functional composition**.

---

### Syntax Rules and Structure

```
Func<TResult>                     // no params, returns TResult
Func<T, TResult>                  // one param, returns TResult
Func<T1, T2, TResult>             // two params, returns TResult
Func<T1, ..., T16, TResult>       // up to 16 params
```

**Component breakdown**

- **Return type** — Always the **last** type argument.
- **Parameters** — Preceding type arguments, up to 16.

#### Syntax Rules

- `Func` is **covariant** in its return type (`out TResult`).
- Lambda expressions, anonymous methods, and method groups convert to `Func` via target typing.
- `Func` can be used as a method parameter, field, or local variable.

#### Constraints and Limitations

- Maximum 16 parameters.
- `void` cannot be a return type (use `Action` instead).

---

### Annotated Code Examples and Expected Outputs

#### Example: `Func<T>` for Factory and Transformation

```csharp
using System;

class Program
{
    static T Create<T>(Func<T> factory) => factory();

    static TResult Map<TInput, TResult>(TInput input, Func<TInput, TResult> map) => map(input);

    static void Main()
    {
        Func<Guid> newGuid = () => Guid.NewGuid();
        Console.WriteLine(Create(newGuid) is Guid);           // True

        Func<string, int> length = s => s.Length;
        Console.WriteLine(Map("Hello", length));              // 5
    }
}
```

**Expected Output**

```
True
5
```

**Why this output occurs**

- `Create<Guid>` uses `Func<Guid>` to lazily produce a `Guid`; `Map` uses `Func<string,int>` to project a string to its length.

---

### Real-World Cases

- **LINQ** — `Enumerable.Select<TSource, TResult>` accepts `Func<TSource, TResult>`.
- **Caching** — `Func<T>` for lazy value generation.
- **Dependency injection** — `Func<TService>` factory registrations.

---

### References

- Func<TResult> Delegate — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.func-1
- Examine the Action and Func delegates — Training - https://learn.microsoft.com/en-us/training/modules/get-started-delegates/04-examine-strongly-typed-delegates

---

## 4. `Action<T>`

### Definitions

**Core Definition**
`Action<T>` is a generic delegate family that represents a method that **does not return a value** (`void`). All type parameters are input parameters.

**Technical Definition**
`System.Action` and `System.Action<T>` (up to 16 parameters) encapsulate methods that perform side effects without returning a value. `Action<in T>` is **contravariant** in its parameters. Used widely for event handling, side-effecting callbacks, and iteration.

**Beginner-Friendly Explanation**
`Action<T>` is “**do this and do not give me anything back**.”

---

### Purposes

- To represent **side-effecting callbacks**.
- To enable **event handling**.
- To model **fire-and-forget** operations.
- To pass **mutation logic** into generic APIs.
- To support **iteration helpers** (`ForEach`-style APIs).

---

### Syntax Rules and Structure

```
Action                              // no params, no return
Action<T>                           // one param, no return
Action<T1, T2>                      // two params, no return
Action<T1, ..., T16>                // up to 16 params
```

**Component breakdown**

- All type arguments are **input parameters**.
- Return type is always `void`.

#### Syntax Rules

- `Action<in T>` is **contravariant** in each parameter.
- Methods, lambdas, and anonymous methods convert to `Action` via target typing.

#### Constraints and Limitations

- Cannot return a value; use `Func` if you need a result.
- Maximum 16 parameters.

---

### Annotated Code Examples and Expected Outputs

#### Example: `Action<T>` for Side Effects

```csharp
using System;
using System.Collections.Generic;

class Program
{
    static void ForEach<T>(IEnumerable<T> items, Action<T> action)
    {
        foreach (var item in items) action(item);
    }

    static void Main()
    {
        var names = new[] { "Alice", "Bob", "Carol" };
        ForEach(names, name => Console.WriteLine($"Hello, {name}!"));
        // Hello, Alice!
        // Hello, Bob!
        // Hello, Carol!
    }
}
```

**Expected Output**

```
Hello, Alice!
Hello, Bob!
Hello, Carol!
```

**Why this output occurs**

- `ForEach` accepts an `Action<string>` and invokes it for each element.

---

### Real-World Cases

- **Event handling** — `EventHandler<TEventArgs>` is a specialised delegate.
- **Logging** — `Action<string> log` for configurable loggers.
- **UI updates** — `Action` callbacks on dispatcher threads.

---

### References

- Action<T> Delegate — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.action-1
- Examine the Action and Func delegates — Training - https://learn.microsoft.com/en-us/training/modules/get-started-delegates/04-examine-strongly-typed-delegates

---

## 5. `Predicate<T>`

### Definitions

**Core Definition**
`Predicate<T>` is a generic delegate that represents a method taking one `T` and returning `bool`, used for filtering and matching.

**Technical Definition**
`System.Predicate<T>` is functionally equivalent to `Func<T, bool>`. It was introduced with `List<T>.Find`, `FindAll`, `Exists`, and `RemoveAll` in .NET 2.0, before `Func` was introduced in .NET 3.5. Newer APIs typically use `Func<T, bool>`; `Predicate<T>` is retained for backward compatibility and is still the delegate type used by many `List<T>` methods.

**Beginner-Friendly Explanation**
`Predicate<T>` is “**a yes/no question about a value**.” It is functionally a `Func<T, bool>` with a legacy name.

---

### Purposes

- To represent **predicate functions** in APIs that predate `Func`.
- To filter collections with `List<T>.Find`, `FindAll`, `RemoveAll`.
- To test elements for membership or matches.
- To maintain compatibility with older .NET APIs.

---

### Syntax Rules and Structure

```
public delegate bool Predicate<in T>(T obj);
```

**Component breakdown**

- `in T` — Contravariant type parameter.
- `bool` — Return type.

#### Syntax Rules

- `Predicate<T>` is contravariant in `T`.
- Method groups and lambdas convert to `Predicate<T>` via target typing.
- `List<T>.Find(Predicate<T>)`, `FindAll`, `Exists`, `RemoveAll`, `FindIndex`, `FindLast`, `TrueForAll` accept `Predicate<T>`.

#### Constraints and Limitations

- Newer APIs usually accept `Func<T, bool>` instead.
- Lambdas convert to either delegate type, but they are **different delegate types** at runtime.

---

### Annotated Code Examples and Expected Outputs

#### Example: `Predicate<T>` with `List<T>`

```csharp
using System;
using System.Collections.Generic;

class Program
{
    static void Main()
    {
        var numbers = new List<int> { 1, 2, 3, 4, 5, 6 };

        Predicate<int> isEven = n => n % 2 == 0;
        var evens = numbers.FindAll(isEven);
        Console.WriteLine(string.Join(", ", evens));   // 2, 4, 6

        numbers.RemoveAll(isEven);
        Console.WriteLine(string.Join(", ", numbers)); // 1, 3, 5
    }
}
```

**Expected Output**

```
2, 4, 6
1, 3, 5
```

**Why this output occurs**

- `FindAll` and `RemoveAll` accept `Predicate<int>` and use it to filter/mutate the list.

---

### Real-World Cases

- **`List<T>` filtering** — `Find`, `FindAll`, `RemoveAll`.
- **Legacy APIs** — Before `Func<T, bool>`.
- **Validation** — `Exists(predicate)`.

---

### References

- Predicate<T> Delegate — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.predicate-1

---

## 6. `Comparison<T>`

### Definitions

**Core Definition**
`Comparison<T>` is a generic delegate that represents a method comparing two `T` values and returning an `int` indicating their relative order.

**Technical Definition**
`System.Comparison<T>` is functionally equivalent to `Func<T, T, int>`. It is used by `List<T>.Sort(Comparison<T>)` and `Array.Sort` (with overloads). Returns a negative number if the first is less than the second, zero if equal, and a positive number if greater. Unlike `IComparer<T>`, `Comparison<T>` is a delegate rather than an interface, so it can be supplied as a lambda.

**Beginner-Friendly Explanation**
`Comparison<T>` is “**which of these two comes first?**” Pass a lambda to `Sort` and you control the order.

---

### Purposes

- To supply **sorting comparators** as delegates.
- To enable **custom sort orders** without implementing `IComparer<T>`.
- To compare elements in generic algorithms.
- To support **sorting LINQ-like pipelines**.

---

### Syntax Rules and Structure

```
public delegate int Comparison<in T>(T x, T y);
```

**Component breakdown**

- `in T` — Contravariant type parameter.
- Return `int` — Negative, zero, or positive.

#### Syntax Rules

- Contravariant in `T`.
- Return sign convention: negative (`x < y`), zero (`x == y`), positive (`x > y`).

#### Constraints and Limitations

- `Comparison<T>` cannot be null (throws `ArgumentNullException` when passed to `Sort`).
- If `Comparison<T>` returns 0 for distinct objects, sort order for those is unstable.

---

### Annotated Code Examples and Expected Outputs

#### Example: Custom Sort with `Comparison<T>`

```csharp
using System;
using System.Collections.Generic;

class Program
{
    static void Main()
    {
        var words = new List<string> { "banana", "apple", "cherry", "date" };

        Comparison<string> byLengthThenAlpha = (a, b) =>
        {
            int lengthCompare = a.Length.CompareTo(b.Length);
            return lengthCompare != 0 ? lengthCompare : string.CompareOrdinal(a, b);
        };

        words.Sort(byLengthThenAlpha);
        Console.WriteLine(string.Join(", ", words));   // date, apple, banana, cherry
    }
}
```

**Expected Output**

```
date, apple, banana, cherry
```

**Why this output occurs**

- `date` (4 chars) sorts first by length, then alphabetical among equal-length words.

---

### Real-World Cases

- **Custom sorting** — Sort by multiple fields.
- **LINQ alternative** — Where `List<T>.Sort` is preferred over `OrderBy`.
- **Numeric or date ordering**.

---

### References

- Comparison<T> Delegate — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.comparison-1

---

## 7. `Converter<TInput, TOutput>`

### Definitions

**Core Definition**
`Converter<TInput, TOutput>` is a generic delegate representing a method that converts a `TInput` to a `TOutput`.

**Technical Definition**
`System.Converter<TInput, TOutput>` is functionally equivalent to `Func<TInput, TOutput>`. It is used by `Array.ConvertAll<TInput, TOutput>` and `List<T>.ConvertAll<TOutput>`. Unlike `Func`, `Converter` is not variant-annotated; its signature has no `in`/`out`.

**Beginner-Friendly Explanation**
`Converter<TInput, TOutput>` is “**take one type, produce another**.” It is a legacy way of saying “converter function.”

---

### Purposes

- To represent **type transformations**.
- To enable **bulk conversions** with `Array.ConvertAll` and `List<T>.ConvertAll`.
- To support **legacy** pre-`Func` APIs.

---

### Syntax Rules and Structure

```
public delegate TOutput Converter<in TInput, out TOutput>(TInput input);
```

**Component breakdown**

- `in TInput` — Contravariant input.
- `out TOutput` — Covariant output.
- Return `TOutput`.

#### Syntax Rules

- Use with `Array.ConvertAll<TInput, TOutput>` and `List<T>.ConvertAll<TOutput>`.
- Lambdas and method groups convert via target typing.

#### Constraints and Limitations

- Newer APIs prefer `Func<TInput, TOutput>`.
- Both `Converter<,>` and `Func<,>` exist; they are **different delegate types**.

---

### Annotated Code Examples and Expected Outputs

#### Example: `Converter<TInput, TOutput>`

```csharp
using System;
using System.Collections.Generic;

class Program
{
    static void Main()
    {
        var strings = new List<string> { "1", "2", "3" };
        Converter<string, int> toInt = s => int.Parse(s);
        var ints = strings.ConvertAll(toInt);
        Console.WriteLine(string.Join(", ", ints));   // 1, 2, 3

        var doubles = Array.ConvertAll(new[] { "1.5", "2.5" }, s => double.Parse(s));
        Console.WriteLine(string.Join(", ", doubles));   // 1.5, 2.5
    }
}
```

**Expected Output**

```
1, 2, 3
1.5, 2.5
```

**Why this output occurs**

- `List<T>.ConvertAll` and `Array.ConvertAll` accept `Converter` and apply it to each element.

---

### Real-World Cases

- **Bulk transformations** — Parsing strings into typed values.
- **DTO mapping** — Converting entities to DTOs.
- **Interop** — Converting between legacy and modern types.

---

### References

- Converter<TInput,TOutput> Delegate — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.converter-2

---

## 8. Generic Patterns

### Definitions

**Core Definition**
Generic patterns are **reusable design structures** that rely on type parameters to work across many types without duplication. They are the practical application of advanced generics to real design problems.

**Technical Definition**
Generic patterns leverage constraints, variance, and generic delegates to deliver reusable, type-safe designs. Common patterns include: **Repository<T>** (data access abstraction), **Factory<T>** (creation with `where T : new()`), **Strategy<T>** (algorithm selection via delegates or interfaces), **Specification<T>** (composable business rules), **Pipeline<T>** (staged transformations), **Options<T>** (strongly typed settings), and **Result<T>** (explicit success/failure outcomes). Each pattern relies on generic type safety while remaining flexible for different `T`.

**Beginner-Friendly Explanation**
Generic patterns are **design recipes that work with any type**. You write the pattern once and apply it to `Customer`, `Order`, `Invoice`, and anything else.

---

### Purposes

- To **reuse design structures** across many entity types.
- To **enforce constraints** required by the pattern.
- To **decouple** implementation from concrete types.
- To **increase testability** through generic interfaces.
- To **reduce duplication** across similar classes.

---

### Syntax Rules and Structure (Pattern Catalogue)

| Pattern | Signature Sketch | Purpose |
|---|---|---|
| **Repository** | `interface IRepository<T> where T : class, IEntity` | CRUD abstraction |
| **Factory** | `class Factory<T> where T : new()` | Generic object creation |
| **Strategy** | `interface IStrategy<T> { T Execute(T input); }` | Interchangeable algorithms |
| **Specification** | `interface ISpecification<T> { bool IsSatisfiedBy(T item); }` | Composable business rules |
| **Pipeline** | `interface IPipeline<T> { T Process(T input); }` | Staged transformations |
| **Options** | `class Options<T>` | Strongly typed configuration |
| **Result** | `class Result<T>` | Explicit success/failure |

**Component breakdown**

- **Repository** — `where T : class, IEntity, new()`.
- **Factory** — `where T : new()`.
- **Strategy** — Often uses `Func<T, T>` or `Action<T>`.
- **Specification** — Composed via `And`, `Or`, `Not` combinators.

#### Syntax Rules

- Each pattern uses generic constraints to lock down valid `T`.
- Patterns often use `Func`/`Action` for flexibility where inheritance is overkill.

#### Constraints and Limitations

- Overly generic patterns can obscure intent; prefer specific types when the pattern is used only once.
- Repository patterns can be over-abstracted; do not create a repository for every entity if a direct ORM call is clearer.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Repository Pattern with Constraints

```csharp
using System;
using System.Collections.Generic;
using System.Linq;

public interface IEntity { int Id { get; } }

public interface IRepository<T> where T : class, IEntity
{
    T? GetById(int id);
    IEnumerable<T> GetAll();
    void Add(T entity);
}

public class InMemoryRepository<T> : IRepository<T> where T : class, IEntity
{
    private readonly List<T> _store = new();
    public T? GetById(int id) => _store.FirstOrDefault(e => e.Id == id);
    public IEnumerable<T> GetAll() => _store;
    public void Add(T entity) => _store.Add(entity);
}

public class Customer : IEntity
{
    public int Id { get; set; }
    public string Name { get; set; } = "";
}

class Program
{
    static void Main()
    {
        IRepository<Customer> repo = new InMemoryRepository<Customer>();
        repo.Add(new Customer { Id = 1, Name = "Alice" });
        Console.WriteLine(repo.GetById(1)!.Name);   // Alice
    }
}
```

**Expected Output**

```
Alice
```

**Why this output occurs**

- The generic `IRepository<T>` works for any `T : class, IEntity`; the concrete implementation is type-safe for `Customer`.

---

#### Example 2: Strategy Pattern with `Func<T, T>`

```csharp
using System;

public class PriceCalculator<T>
{
    private readonly Func<T, T> _strategy;

    public PriceCalculator(Func<T, T> strategy) => _strategy = strategy;

    public T Calculate(T input) => _strategy(input);
}

class Program
{
    static void Main()
    {
        var tenPercent = new PriceCalculator<decimal>(p => p * 0.9m);
        Console.WriteLine(tenPercent.Calculate(100m));   // 90.0

        var twentyPercent = new PriceCalculator<decimal>(p => p * 0.8m);
        Console.WriteLine(twentyPercent.Calculate(100m)); // 80.0
    }
}
```

**Expected Output**

```
90.0
80.0
```

**Why this output occurs**

- The strategy is injected via `Func<decimal, decimal>` and can be swapped at runtime.

---

#### Example 3: Specification Pattern

```csharp
using System;

public interface ISpecification<T>
{
    bool IsSatisfiedBy(T item);
}

public class AndSpecification<T> : ISpecification<T>
{
    private readonly ISpecification<T> _left;
    private readonly ISpecification<T> _right;
    public AndSpecification(ISpecification<T> left, ISpecification<T> right)
    {
        _left = left;
        _right = right;
    }
    public bool IsSatisfiedBy(T item) => _left.IsSatisfiedBy(item) && _right.IsSatisfiedBy(item);
}

public class LambdaSpec<T> : ISpecification<T>
{
    private readonly Func<T, bool> _predicate;
    public LambdaSpec(Func<T, bool> predicate) => _predicate = predicate;
    public bool IsSatisfiedBy(T item) => _predicate(item);
}

class Program
{
    static void Main()
    {
        var over18 = new LambdaSpec<int>(age => age >= 18);
        var under65 = new LambdaSpec<int>(age => age < 65);
        var workingAge = new AndSpecification<int>(over18, under65);

        Console.WriteLine(workingAge.IsSatisfiedBy(30));   // True
        Console.WriteLine(workingAge.IsSatisfiedBy(70));   // False
    }
}
```

**Expected Output**

```
True
False
```

**Why this output occurs**

- The specification composes two predicates via `AndSpecification<T>`, demonstrating composable generic rules.

---

### Real-World Cases

- **DDD repositories** — `IRepository<T>`.
- **ASP.NET Core options** — `IOptions<T>`, `IOptionsSnapshot<T>`.
- **MediatR** — `IRequestHandler<TRequest, TResponse>`.
- **Result/Either** — `Result<T>` for functional error handling.

---

### References

- Generic classes — C# Programming Guide - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/generics/generic-classes
- Generic interfaces — C# Programming Guide - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/generics/generic-interfaces
- ASP.NET Core options pattern — Microsoft Learn - https://learn.microsoft.com/en-us/dotnet/core/extensions/options

---

## 9. Generic Best Practices

### Definitions

**Core Definition**
Generic best practices are the guidelines for designing, composing, and evolving generic types and methods so they remain **readable, safe, evolvable, and performant**.

**Technical Definition**
Best practices for advanced generics derive from the Framework Design Guidelines and code analysis rules. Key recommendations include: **avoid excessive type parameters** (CA1005 — no more than two for public APIs); **do not declare static members on generic types** (CA1000); **apply the least restrictive constraints necessary** to enable the required operations; **prefer interfaces over base classes** for constraints when the relationship is contract-based; **use variant interfaces and delegates (`out`/`in`) to increase flexibility**; **prefer generic delegates (`Func`, `Action`) over custom delegates**; **avoid overriding base-class members on generic types that may not have the correct signature** (CA1003); **document constraints** in XML comments; and **use `nameof(T)`** where possible for diagnostic messages. Advanced patterns (repository, factory, strategy) should be applied only when they reduce duplication meaningfully—not for their own sake.

**Beginner-Friendly Explanation**
Generic best practices say: **use generics when they save real duplication**, **keep type parameters minimal**, **state constraints precisely**, and **prefer the standard delegates** over custom ones.

---

### Purposes

- To **maximise code reuse** without harming readability.
- To **prevent design smells** flagged by analyzers.
- To **choose the right tool** (generic vs. non-generic, interface vs. base class, delegate family).
- To keep APIs **evolvable** without breaking consumers.
- To align with **.NET conventions**.

---

### Guidelines and Structure

#### General Guidelines

```
// Prefer standard generic delegates
Func<int, bool> isEven = n => n % 2 == 0;   // not a custom delegate
Action<string> log = Console.WriteLine;

// Apply the least restrictive constraints needed
static T Max<T>(T a, T b) where T : IComparable<T> => ...;

// Avoid excessive type parameters (CA1005, ≤ 2 for public APIs)
// Avoid static members on generic types (CA1000)
// Prefer interfaces over base classes for contract-based constraints
// Use variant interfaces/delegates to maximise flexibility
```

**Component breakdown**

- **Standard delegates** — `Func`, `Action`, `Predicate`, `Comparison`, `Converter`.
- **Least restrictive constraints** — Do not over-constrain `T`.
- **Minimal type parameters** — CA1005.
- **No static members on generics** — CA1000.
- **Variance** — Enable flexibility with `out`/`in`.

#### Rules and Constraints

- **Do** use standard generic delegates.
- **Do** apply only the constraints your implementation needs.
- **Do** prefer `IEnumerable<T>`/`IReadOnlyList<T>` in public APIs.
- **Do not** add static members to generic types (CA1000).
- **Do not** use more than two type parameters on public generic types (CA1005).
- **Do not** write generic code “just in case”; justify every type parameter.

---

### Annotated Code Examples and Expected Outputs

#### Example: Fully-Realised Advanced Generic API

```csharp
using System;
using System.Collections.Generic;

public interface IEntity { int Id { get; } }

public interface IRepository<T> where T : class, IEntity
{
    T? GetById(int id);
    IEnumerable<T> GetAll();
    void Add(T entity);
    void Remove(T entity);
}

public class InMemoryRepository<T> : IRepository<T> where T : class, IEntity
{
    private readonly Dictionary<int, T> _store = new();
    public T? GetById(int id) => _store.TryGetValue(id, out var e) ? e : null;
    public IEnumerable<T> GetAll() => _store.Values;
    public void Add(T entity) => _store[entity.Id] = entity;
    public void Remove(T entity) => _store.Remove(entity.Id);
}

public class Customer : IEntity
{
    public int Id { get; set; }
    public string Name { get; set; } = "";
}

class Program
{
    static void Main()
    {
        IRepository<Customer> repo = new InMemoryRepository<Customer>();
        repo.Add(new Customer { Id = 1, Name = "Alice" });
        repo.Add(new Customer { Id = 2, Name = "Bob" });

        Func<Customer, bool> startsWithA = c => c.Name.StartsWith("A");
        foreach (var c in repo.GetAll())
            if (startsWithA(c))
                Console.WriteLine(c.Name);   // Alice
    }
}
```

**Expected Output**

```
Alice
```

**Why this output occurs**

- The code uses a **generic interface with a meaningful constraint** (`class, IEntity`), a **dictionary-backed repository**, and a **standard `Func<Customer, bool>`** instead of a custom predicate delegate—demonstrating the recommended combination of constraints, patterns, and delegate families.

---

### Real-World Cases

- **Enterprise APIs** — Constrained generic repositories and services.
- **Middleware pipelines** — `Func<T, Task<T>>` strategies.
- **DI registration** — `services.AddSingleton<IRepository<Customer>, InMemoryRepository<Customer>>()`.

---

### References

- Constraints on type parameters — C# Programming Guide - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/generics/constraints-on-type-parameters
- CA1000: Do not declare static members on generic types — .NET code analysis - https://learn.microsoft.com/en-us/dotnet/fundamentals/code-analysis/quality-rules/ca1000
- CA1005: Avoid excessive parameters on generic types — .NET code analysis - https://learn.microsoft.com/en-us/dotnet/fundamentals/code-analysis/quality-rules/ca1005
- CA1003: Use generic event handler instances — .NET code analysis - https://learn.microsoft.com/en-us/dotnet/fundamentals/code-analysis/quality-rules/ca1003
- Framework Design Guidelines — Microsoft Learn - https://learn.microsoft.com/en-us/dotnet/standard/design-guidelines/

---

## Deprecated, Unsafe, Implementation-Dependent, and Version-Specific Notes

| Feature | Status / Note |
|---|---|
| `Func<T>` and `Action<T>` | Introduced in **.NET 3.5**; stable, preferred. |
| `Predicate<T>` | Introduced in **.NET 2.0**; functionally superseded by `Func<T, bool>` but retained. |
| `Comparison<T>` | Introduced in **.NET 2.0**; stable. |
| `Converter<,>` | Introduced in **.NET 2.0**; specialised; prefer `Func<,>` in new APIs. |
| Generic variance | Introduced in **.NET 4.0 / C# 4.0**; reference types only. |
| Static abstract interface members | Introduced in **C# 11**; enables generic math. |
| CA1000 / CA1005 | **Design rules**; not compile errors, but flagged by analyzers. |
| `ref` / `in` / `out` parameters | **Not variant**; cannot be used in variant interfaces/delegates. |

---

## References (Consolidated)

- Generics (C# Programming Guide) — Microsoft Learn - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/generics/
- Generic classes — C# Programming Guide - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/generics/generic-classes
- Generic interfaces — C# Programming Guide - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/generics/generic-interfaces
- Generic methods — C# Programming Guide - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/generics/generic-methods
- Generic delegates — C# Programming Guide - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/generics/generic-delegates
- Constraints on type parameters — C# Programming Guide - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/generics/constraints-on-type-parameters
- where (generic type constraint) — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/where-generic-type-constraint
- Variance in Delegates (C#) — Microsoft Learn - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/concepts/covariance-contravariance/variance-in-delegates
- Examine the Action and Func delegates — Training - https://learn.microsoft.com/en-us/training/modules/get-started-delegates/04-examine-strongly-typed-delegates
- Func<TResult> Delegate — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.func-1
- Action<T> Delegate — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.action-1
- Predicate<T> Delegate — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.predicate-1
- Comparison<T> Delegate — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.comparison-1
- Converter<TInput,TOutput> Delegate — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.converter-2
- CA1000: Do not declare static members on generic types — .NET code analysis - https://learn.microsoft.com/en-us/dotnet/fundamentals/code-analysis/quality-rules/ca1000
- CA1005: Avoid excessive parameters on generic types — .NET code analysis - https://learn.microsoft.com/en-us/dotnet/fundamentals/code-analysis/quality-rules/ca1005
- CA1003: Use generic event handler instances — .NET code analysis - https://learn.microsoft.com/en-us/dotnet/fundamentals/code-analysis/quality-rules/ca1003
- Framework Design Guidelines — Microsoft Learn - https://learn.microsoft.com/en-us/dotnet/standard/design-guidelines/
- ASP.NET Core options pattern — Microsoft Learn - https://learn.microsoft.com/en-us/dotnet/core/extensions/options

---