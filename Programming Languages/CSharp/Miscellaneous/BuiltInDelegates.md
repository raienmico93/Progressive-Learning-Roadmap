# C# Built-in Delegates — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
**Built-in delegates** are pre-defined delegate types provided by .NET that cover the most common callback scenarios—taking actions (`Action`), returning values (`Func`), testing conditions (`Predicate`), comparing values (`Comparison`), converting types (`Converter`), and handling events (`EventHandler`). They eliminate the need to declare custom delegate types for routine patterns, standardise the vocabulary of callbacks across .NET libraries, and integrate seamlessly with lambdas, method groups, and LINQ.

**Technical Definition**
The .NET class library ships a family of delegate types under `System`: `Action` and `Action<T>` (up to 16 parameters, returning `void`), `Func<TResult>` and `Func<T, TResult>` (up to 16 parameters plus a return type), `Predicate<T>` (returns `bool`), `Comparison<T>` (returns `int` for ordering), `Converter<TInput, TOutput>` (returns `TOutput`), and the event-handling delegates `EventHandler` and `EventHandler<TEventArgs>` (return `void` and take `(object? sender, TEventArgs e)`). Generic variants support variance (`Action<in T>`, `Func<in T, out TResult>`). All built-in delegates are sealed classes deriving from `System.MulticastDelegate`, and all can be created from lambdas, anonymous methods, and method groups via target typing.

**Beginner-Friendly Explanation**
Built-in delegates are **ready-made function types** the .NET team already wrote for you. Instead of declaring your own delegate to represent “a method that takes an int and returns a bool,” you just use `Func<int, bool>`. They are the standard vocabulary of callbacks: `Action` for “do this,” `Func` for “give me that,” `Predicate` for “is this true?”, `Comparison` for “which comes first?”, `Converter` for “turn this into that,” and `EventHandler` for “something happened.”

---

### Key Characteristics

- **Standardised signatures** — Each built-in delegate family has a fixed, well-known shape.
- **Up to 16 parameters** — `Action` and `Func` support up to 16 generic parameters.
- **Variance support** — `Action<in T>` and `Func<in T, out TResult>` enable covariant/contravariant assignments.
- **Lambda and method-group friendly** — Any compatible lambda or method converts to the delegate automatically.
- **Interoperable with LINQ and events** — `Func`, `Action`, and `EventHandler` are the lingua franca of LINQ and the event model.
- **Specialised vs. generic** — `Predicate`, `Comparison`, and `Converter` are specialised legacy shapes; `Func` and `Action` are the general-purpose modern shapes.

---

### Prerequisites

- Basic understanding of C# methods, parameters, and return types.
- Familiarity with delegates (`delegate` keyword) and delegate instantiation.
- Knowledge of lambdas, method groups, and closures.
- Awareness of LINQ operators that accept `Func`/`Action`.
- Basic understanding of events (for `EventHandler`).

---

### Related Programming Areas

- **Delegates** — Built-in delegates are pre-defined delegate types; custom delegates are still supported.
- **LINQ** — `Where`, `Select`, `OrderBy`, and other operators rely on `Func` and `Comparison`.
- **Events** — `EventHandler` and `EventHandler<TEventArgs>` define the standard event pattern.
- **Asynchronous programming** — `Func<Task>` and `Func<T, Task>` model async callbacks.
- **Functional programming** — `Func` and `Action` are foundational to higher-order functions.

---

### Core Concepts / Features

1. `Action`
2. `Action<T>`
3. `Func<TResult>`
4. `Func<T, TResult>`
5. `Predicate<T>`
6. `Comparison<T>`
7. `Converter<TInput, TOutput>`
8. `EventHandler`
9. `EventHandler<TEventArgs>`
10. Built-in Delegate Best Practices

Each concept follows a uniform structure: definitions, purposes, syntax rules and structure, annotated code examples with expected outputs, real-world cases, and references.

---

## 1. `Action`

### Definitions

**Core Definition**
`Action` is a parameterless delegate that returns `void`. It represents “do this action, and do not give anything back.”

**Technical Definition**
`System.Action` is a delegate for a method that takes **no parameters** and returns **no value**. It is the non-generic root of the `Action` family (`Action<T>`, `Action<T1, T2>`, ..., `Action<T1, ..., T16>`). It derives from `MulticastDelegate` and supports lambdas, method groups, and anonymous methods.

**Beginner-Friendly Explanation**
`Action` is “**a callback that does something and returns nothing**.”

---

### Purposes

- To represent **parameterless side-effecting operations**.
- To decouple producers from actions (e.g., triggering work).
- To pass “do this” callbacks into APIs.
- To compose sequences of side effects.

---

### Syntax Rules and Structure

```
public delegate void Action();
```

**Component breakdown**

- `void` — Return type; no value.
- `()` — No parameters.

#### Syntax Rules

- Compatible with methods, lambdas, and anonymous methods taking no parameters and returning `void`.
- Can be multicast; all handlers run in order.
- Invoked with `action()` or `action.Invoke()`; `action?.Invoke()` is the null-safe form.

#### Constraints and Limitations

- Cannot return a value; use `Func<TResult>` if a result is needed.
- Cannot take parameters; use `Action<T>` for parameterised actions.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Parameterless Action

```csharp
using System;

class Program
{
    static void Main()
    {
        Action printHello = () => Console.WriteLine("Hello!");
        printHello();             // Hello!
        printHello.Invoke();      // Hello!
    }
}
```

**Expected Output**

```
Hello!
Hello!
```

**Why this output occurs**

- `printHello()` and `printHello.Invoke()` both execute the lambda; the lambda takes no parameters and returns `void`.

---

#### Example 2: Multicast Action

```csharp
using System;

class Program
{
    static void Main()
    {
        Action pipeline = () => Console.WriteLine("Step 1");
        pipeline += () => Console.WriteLine("Step 2");
        pipeline += () => Console.WriteLine("Step 3");

        pipeline();
    }
}
```

**Expected Output**

```
Step 1
Step 2
Step 3
```

**Why this output occurs**

- Multicast `Action` invokes every handler in insertion order; `void` return means no truncation concerns.

---

### Real-World Cases

- **Deferred execution** — `Action` to delay work (`Task.Run(Action)`).
- **UI invocations** — Dispatching parameterless updates.
- **Shutdown hooks** — Registering cleanup actions.

---

### References

- Action Delegate — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.action

---

## 2. `Action<T>`

### Definitions

**Core Definition**
`Action<T>` is a generic delegate family representing methods that take **one or more parameters** and return **`void`**.

**Technical Definition**
`System.Action<T>` and its variants (`Action<T1, T2>`, ..., `Action<T1, ..., T16>`) encapsulate methods with up to 16 parameters and no return value. All generic parameters are **contravariant** (`in T`), enabling a wider input type to be substituted where a narrower one is expected.

**Beginner-Friendly Explanation**
`Action<T>` is “**a callback that takes inputs and returns nothing**.”

---

### Purposes

- To represent **parameterised side effects**.
- To pass “consume this value” callbacks.
- To enable **`List<T>.ForEach`**, `Parallel.ForEach`, and similar iteration APIs.
- To decouple producers from parameterised consumers.

---

### Syntax Rules and Structure

```
public delegate void Action<in T>(T obj);
public delegate void Action<in T1, in T2>(T1 arg1, T2 arg2);
// ... up to 16 parameters
```

**Component breakdown**

- `in T` — Contravariant input parameter(s).
- `void` — No return.

#### Syntax Rules

- Up to **16 parameters** supported.
- Contravariant in each parameter.
- Compatible with lambdas/method groups matching the shape.
- Multicast supported.

#### Constraints and Limitations

- Cannot return a value.
- Cannot exceed 16 parameters.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: `Action<T>` with `List<T>.ForEach`

```csharp
using System;
using System.Collections.Generic;

class Program
{
    static void Main()
    {
        var names = new List<string> { "Alice", "Bob", "Carol" };
        Action<string> print = name => Console.WriteLine($"Hello, {name}!");
        names.ForEach(print);
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

- `List<T>.ForEach` accepts an `Action<T>` and invokes it for each element.

---

#### Example 2: Multi-Parameter `Action<T1, T2>`

```csharp
using System;

class Program
{
    static void Main()
    {
        Action<string, int> report = (name, score) =>
            Console.WriteLine($"{name}: {score}");

        report("Alice", 95);   // Alice: 95
    }
}
```

**Expected Output**

```
Alice: 95
```

**Why this output occurs**

- `Action<string, int>` accepts two parameters and returns `void`.

---

### Real-World Cases

- **Logging** — `Action<string>` for configurable logging.
- **Iteration** — `ForEach` and `Parallel.ForEach`.
- **Event dispatch** — `Action<EventArgs>` for lightweight event handling.

---

### References

- Action<T> Delegate — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.action-1

---

## 3. `Func<TResult>`

### Definitions

**Core Definition**
`Func<TResult>` is a parameterless delegate that **returns a value** of type `TResult`. It represents “give me a result.”

**Technical Definition**
`System.Func<TResult>` encapsulates a method that takes **no parameters** and returns `TResult`. It is the non-generic-input root of the `Func` family and is **covariant in `TResult`** (`out TResult`). It is ideal for factories, lazy values, and value-producing callbacks.

**Beginner-Friendly Explanation**
`Func<TResult>` is “**a function that gives you something without needing input**.”

---

### Purposes

- To represent **factories** and value providers.
- To implement **lazy value generation** (`Lazy<T>` accepts `Func<T>`).
- To pass **result-producing callbacks**.
- To enable **deferred computation**.

---

### Syntax Rules and Structure

```
public delegate TResult Func<out TResult>();
```

**Component breakdown**

- `out TResult` — Covariant return type.
- `()` — No parameters.

#### Syntax Rules

- Can be created from lambdas, method groups, or anonymous methods.
- Covariance allows `Func<Derived>` → `Func<Base>`.
- Invoked via `func()` or `func.Invoke()`.

#### Constraints and Limitations

- Cannot take parameters; use `Func<T, TResult>` for parameterised returns.
- Returns `TResult`; if `void`, use `Action`.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: `Func<TResult>` as Factory

```csharp
using System;

class Program
{
    static void Main()
    {
        Func<Guid> newId = () => Guid.NewGuid();
        Func<DateTime> now = () => DateTime.UtcNow;

        Console.WriteLine(newId() is Guid);    // True
        Console.WriteLine(now() is DateTime);  // True
    }
}
```

**Expected Output**

```
True
True
```

**Why this output occurs**

- `Func<Guid>` and `Func<DateTime>` are parameterless but return values.

---

#### Example 2: Covariance with `Func<TResult>`

```csharp
using System;

class Animal { }
class Dog : Animal { }

class Program
{
    static Dog CreateDog() => new Dog();

    static void Main()
    {
        Func<Dog> dogFactory = CreateDog;
        Func<Animal> animalFactory = dogFactory;   // covariance

        Console.WriteLine(animalFactory() is Dog);   // True
    }
}
```

**Expected Output**

```
True
```

**Why this output occurs**

- `Func<out TResult>` is covariant; `Func<Dog>` can be assigned to `Func<Animal>`.

---

### Real-World Cases

- **Dependency injection** — `Func<TService>` factories.
- **Lazy caching** — `Lazy<T>` constructor.
- **Test fakes** — `Func<DateTime>` for time.

---

### References

- Func<TResult> Delegate — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.func-1

---

## 4. `Func<T, TResult>`

### Definitions

**Core Definition**
`Func<T, TResult>` is the generic delegate family representing methods that take **one or more parameters** and **return a value**. The last type parameter is always the return type.

**Technical Definition**
`System.Func<T, TResult>` and its multi-parameter variants (`Func<T1, T2, TResult>`, ..., `Func<T1, ..., T16, TResult>`) encapsulate a method with up to **16 input parameters** and a return value of the **last** type parameter. Input parameters are **contravariant** (`in`), and the return type is **covariant** (`out`). This family is the backbone of LINQ, method chaining, and functional programming in C#.

**Beginner-Friendly Explanation**
`Func<T, TResult>` is “**a function that takes inputs and returns a result**.” The last type argument is what it gives back.

---

### Purposes

- To represent **transformations, projections, and computations**.
- To power **LINQ** (`Select`, `Where`, `Aggregate`).
- To pass **parameterised callbacks** returning values.
- To enable **function composition**.

---

### Syntax Rules and Structure

```
public delegate TResult Func<in T, out TResult>(T arg);
public delegate TResult Func<in T1, in T2, out TResult>(T1 arg1, T2 arg2);
// ... up to 16 input parameters
```

**Component breakdown**

- `in T…` — Contravariant input parameter(s).
- `out TResult` — Covariant return type (always last).

#### Syntax Rules

- **Return type is always the last type argument.**
- **Up to 16 input parameters.**
- Full variance support (contravariant inputs, covariant output).

#### Constraints and Limitations

- Cannot return `void`; use `Action`.
- Cannot exceed 16 input parameters.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: `Func<T, TResult>` in LINQ

```csharp
using System;
using System.Linq;

class Program
{
    static void Main()
    {
        int[] numbers = { 1, 2, 3, 4, 5 };

        Func<int, bool> isEven = n => n % 2 == 0;
        Func<int, int> square = n => n * n;

        var squares = numbers.Where(isEven).Select(square);
        Console.WriteLine(string.Join(", ", squares));   // 4, 16
    }
}
```

**Expected Output**

```
4, 16
```

**Why this output occurs**

- `Where` takes `Func<int, bool>`; `Select` takes `Func<int, int>`. The functions are passed as data.

---

#### Example 2: Multi-Parameter `Func<T1, T2, TResult>`

```csharp
using System;

class Program
{
    static void Main()
    {
        Func<int, int, int> add = (a, b) => a + b;
        Func<int, int, bool> greater = (a, b) => a > b;

        Console.WriteLine(add(3, 4));       // 7
        Console.WriteLine(greater(5, 3));   // True
    }
}
```

**Expected Output**

```
7
True
```

**Why this output occurs**

- `Func<int, int, int>` and `Func<int, int, bool>` accept two inputs and return a value.

---

### Real-World Cases

- **LINQ** — `Select`, `Where`, `GroupBy`, `OrderBy`.
- **ASP.NET Core** — `Func<HttpContext, Task>` middleware.
- **Mapping** — `Func<TEntity, TDto>` projections.

---

### References

- Func<T,TResult> Delegate — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.func-2

---

## 5. `Predicate<T>`

### Definitions

**Core Definition**
`Predicate<T>` is a delegate representing a **test** of an item, returning `bool`. It is functionally identical to `Func<T, bool>` but is the signature used by older .NET APIs such as `List<T>.Find`, `FindAll`, `Exists`, and `RemoveAll`.

**Technical Definition**
`System.Predicate<T>` is declared as `delegate bool Predicate<in T>(T obj)` and is **contravariant** in `T`. It is not the same delegate type as `Func<T, bool>`, though lambdas and method groups convert to either via target typing. Newer APIs prefer `Func<T, bool>`, but `Predicate<T>` remains for backward compatibility.

**Beginner-Friendly Explanation**
`Predicate<T>` is “**a yes/no question about a value**.” Use it when an API specifically expects it—like `List<T>.FindAll`.

---

### Purposes

- To represent **filtering** and **matching** logic.
- To satisfy legacy APIs (`List<T>.Find`, `RemoveAll`).
- To express **validation checks** on individual elements.
- To enable **composition of test conditions**.

---

### Syntax Rules and Structure

```
public delegate bool Predicate<in T>(T obj);
```

**Component breakdown**

- `in T` — Contravariant input.
- `bool` — Return type.

#### Syntax Rules

- Contravariant in `T`.
- Same shape as `Func<T, bool>` but a **different delegate type**.
- Lambdas and method groups convert via target typing.

#### Constraints and Limitations

- Not interchangeable with `Func<T, bool>` without conversion.
- Rarely needed in new APIs; prefer `Func<T, bool>`.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Filtering with `List<T>.FindAll`

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
    }
}
```

**Expected Output**

```
2, 4, 6
```

**Why this output occurs**

- `FindAll` accepts `Predicate<int>` and applies it to each element.

---

#### Example 2: Removing with `RemoveAll`

```csharp
using System;
using System.Collections.Generic;

class Program
{
    static void Main()
    {
        var words = new List<string> { "apple", "fig", "banana", "kiwi" };

        Predicate<string> isShort = s => s.Length <= 3;
        words.RemoveAll(isShort);
        Console.WriteLine(string.Join(", ", words));   // apple, banana
    }
}
```

**Expected Output**

```
apple, banana
```

**Why this output occurs**

- `RemoveAll` removes elements matching the predicate.

---

### Real-World Cases

- **Legacy filtering** — `List<T>.Find`/`FindAll`/`RemoveAll`.
- **Validation** — `Exists(predicate)` for input checks.
- **Search** — `List<T>.FindIndex`.

---

### References

- Predicate<T> Delegate — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.predicate-1

---

## 6. `Comparison<T>`

### Definitions

**Core Definition**
`Comparison<T>` is a delegate representing a **comparison of two `T` values**, returning an `int` that indicates their relative order. It is functionally identical to `Func<T, T, int>` and is used by `List<T>.Sort` and `Array.Sort`.

**Technical Definition**
`System.Comparison<T>` is declared as `delegate int Comparison<in T>(T x, T y)` and is **contravariant** in `T`. The return value follows the standard sign convention: negative if `x < y`, zero if equal, positive if `x > y`. It is a delegate-based alternative to `IComparer<T>`.

**Beginner-Friendly Explanation**
`Comparison<T>` is “**which of these two comes first?**” Pass a lambda to `Sort` to define the order.

---

### Purposes

- To **sort** collections with a custom order.
- To provide **delegate-based comparers** (versus interface-based `IComparer<T>`).
- To enable **multi-key sorting** via lambda composition.
- To avoid implementing `IComparer<T>` for one-off comparisons.

---

### Syntax Rules and Structure

```
public delegate int Comparison<in T>(T x, T y);
```

**Component breakdown**

- `in T` — Contravariant input parameters.
- `int` — Return value: negative, zero, or positive.

#### Syntax Rules

- **Contravariant** in `T`.
- Sign convention: negative (less), zero (equal), positive (greater).
- Accepted by `List<T>.Sort(Comparison<T>)` and `Array.Sort(T[], Comparison<T>)`.

#### Constraints and Limitations

- Unstable sort when the comparison returns 0 for distinct elements.
- Cannot be `null` when passed to sorting APIs (throws `ArgumentNullException`).

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Sorting Descending

```csharp
using System;
using System.Collections.Generic;

class Program
{
    static void Main()
    {
        var numbers = new List<int> { 3, 1, 4, 1, 5 };
        Comparison<int> descending = (a, b) => b.CompareTo(a);
        numbers.Sort(descending);
        Console.WriteLine(string.Join(", ", numbers));   // 5, 4, 3, 1, 1
    }
}
```

**Expected Output**

```
5, 4, 3, 1, 1
```

**Why this output occurs**

- Reversing the sign of `CompareTo` reverses the sort order.

---

#### Example 2: Multi-Key Sorting

```csharp
using System;
using System.Collections.Generic;

class Program
{
    static void Main()
    {
        var employees = new List<(string Name, int Age)>
        {
            ("Alice", 30), ("Bob", 25), ("Carol", 30)
        };

        Comparison<(string Name, int Age)> byAgeThenName = (a, b) =>
        {
            int cmp = a.Age.CompareTo(b.Age);
            return cmp != 0 ? cmp : string.CompareOrdinal(a.Name, b.Name);
        };

        employees.Sort(byAgeThenName);
        foreach (var e in employees)
            Console.WriteLine($"{e.Name} ({e.Age})");
    }
}
```

**Expected Output**

```
Bob (25)
Alice (30)
Carol (30)
```

**Why this output occurs**

- The lambda orders by `Age`, then by `Name` when ages tie.

---

### Real-World Cases

- **Custom ordering** — Sort by computed fields or multiple criteria.
- **`Array.Sort`/`List.Sort`** — In-place sorting with a lambda comparer.
- **Search algorithms** — Binary search requires an ordered collection.

---

### References

- Comparison<T> Delegate — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.comparison-1

---

## 7. `Converter<TInput, TOutput>`

### Definitions

**Core Definition**
`Converter<TInput, TOutput>` is a delegate representing a **conversion** from one type to another. It is functionally identical to `Func<TInput, TOutput>` and is used by `Array.ConvertAll` and `List<T>.ConvertAll<TOutput>`.

**Technical Definition**
`System.Converter<TInput, TOutput>` is declared as `delegate TOutput Converter<in TInput, out TOutput>(TInput input)`. It is **contravariant** in `TInput` and **covariant** in `TOutput`. It is a legacy shape introduced in .NET 2.0 before `Func<T, TResult>` existed; modern APIs prefer `Func<T, TResult>`.

**Beginner-Friendly Explanation**
`Converter<TInput, TOutput>` is “**transform each item into a new type**.” It is the function behind `ConvertAll`.

---

### Purposes

- To represent **element-wise conversions**.
- To enable **`Array.ConvertAll`** and **`List<T>.ConvertAll`**.
- To preserve backwards compatibility with pre-`Func` APIs.
- To express transformation semantics in intent-revealing APIs.

---

### Syntax Rules and Structure

```
public delegate TOutput Converter<in TInput, out TOutput>(TInput input);
```

**Component breakdown**

- `in TInput` — Contravariant input.
- `out TOutput` — Covariant output.
- Returns `TOutput`.

#### Syntax Rules

- **Contravariant** in `TInput`, **covariant** in `TOutput`.
- Accepted by `ConvertAll` APIs.
- Lambdas and method groups convert via target typing.

#### Constraints and Limitations

- Not the same delegate type as `Func<TInput, TOutput>`.
- Newer APIs prefer `Func<T, TResult>`.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: `List<T>.ConvertAll`

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
    }
}
```

**Expected Output**

```
1, 2, 3
```

**Why this output occurs**

- `List<T>.ConvertAll` accepts `Converter<string, int>` and produces a new list of `int`.

---

#### Example 2: `Array.ConvertAll`

```csharp
using System;

class Program
{
    static void Main()
    {
        string[] parts = { "10", "20", "30" };
        var values = Array.ConvertAll(parts, s => int.Parse(s));
        Console.WriteLine(string.Join("+", values) + " = " + (values[0] + values[1] + values[2]));
        // 10+20+30 = 60
    }
}
```

**Expected Output**

```
10+20+30 = 60
```

**Why this output occurs**

- `Array.ConvertAll` returns a new `int[]` array.

---

### Real-World Cases

- **Bulk parsing** — Converting arrays of strings to numbers.
- **DTO mapping** — `Converter<Entity, Dto>`.
- **Interop** — Converting legacy types.

---

### References

- Converter<TInput,TOutput> Delegate — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.converter-2

---

## 8. `EventHandler`

### Definitions

**Core Definition**
`EventHandler` is the **standard delegate** for **events without custom event data**. It is declared as `delegate void EventHandler(object? sender, EventArgs e)` and is used to signal that “something happened.”

**Technical Definition**
`System.EventHandler` is the non-generic delegate used in the standard .NET event pattern. It takes two parameters: the `sender` (usually `object?`) and the `e` (`EventArgs`). It returns `void` and supports multiple subscribers (multicast). It is typically used for events that do not carry payload—`EventArgs.Empty` is passed instead. The `?.Invoke` pattern is used for safe raising.

**Beginner-Friendly Explanation**
`EventHandler` is “**the standard “something happened” notification**.”

---

### Purposes

- To define **parameterless-data events**.
- To follow the **.NET event pattern**.
- To allow multiple subscribers via multicast.
- To integrate cleanly with UI frameworks and event systems.

---

### Syntax Rules and Structure

```
public delegate void EventHandler(object? sender, EventArgs e);
```

**Component breakdown**

- `object? sender` — The event source.
- `EventArgs e` — Event arguments (often `EventArgs.Empty`).
- `void` — No return.

#### Syntax Rules

- Raise via `handler?.Invoke(this, EventArgs.Empty)`.
- Declared with `public event EventHandler? Click;`.
- Subscribers use `+=` / `-=`.
- Invocations run in subscription order.

#### Constraints and Limitations

- No custom payload; use `EventHandler<TEventArgs>` for event-specific data.
- Exceptions in one handler skip the rest.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Simple Event

```csharp
using System;

public class Button
{
    public event EventHandler? Click;

    public void SimulateClick() => Click?.Invoke(this, EventArgs.Empty);
}

class Program
{
    static void Main()
    {
        var button = new Button();
        button.Click += (sender, e) => Console.WriteLine("Clicked!");
        button.SimulateClick();   // Clicked!
    }
}
```

**Expected Output**

```
Clicked!
```

**Why this output occurs**

- `Click?.Invoke(this, EventArgs.Empty)` safely raises the event; the subscriber receives `(sender, EventArgs.Empty)` and prints a message.

---

#### Example 2: Multiple Subscribers

```csharp
using System;

public class Timer
{
    public event EventHandler? Tick;

    public void Fire() => Tick?.Invoke(this, EventArgs.Empty);
}

class Program
{
    static void Main()
    {
        var timer = new Timer();
        timer.Tick += (s, e) => Console.WriteLine("Handler A");
        timer.Tick += (s, e) => Console.WriteLine("Handler B");
        timer.Fire();
    }
}
```

**Expected Output**

```
Handler A
Handler B
```

**Why this output occurs**

- Multicast invocation runs subscribers in subscription order.

---

### Real-World Cases

- **UI buttons** — `event EventHandler Click`.
- **Timers** — `Elapsed` events.
- **Domain events** — Lightweight notifications with no payload.

---

### References

- EventHandler Delegate — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.eventhandler

---

## 9. `EventHandler<TEventArgs>`

### Definitions

**Core Definition**
`EventHandler<TEventArgs>` is the **generic standard event delegate** for events that carry **custom data** of type `TEventArgs`. It is declared as `delegate void EventHandler<TEventArgs>(object? sender, TEventArgs e)`.

**Technical Definition**
`System.EventHandler<TEventArgs>` is the recommended signature for custom events in .NET. `TEventArgs` is typically a subclass of `EventArgs` (though not required). The delegate is **contravariant** in `TEventArgs`. It follows the standard pattern: two parameters `(sender, e)`, `void` return, and multicast semantics.

**Beginner-Friendly Explanation**
`EventHandler<TEventArgs>` is “**the standard event notification with extra data**.” The `TEventArgs` carries the payload.

---

### Purposes

- To define **custom events with payload**.
- To follow the **.NET event pattern** with typed data.
- To allow subscribers to inspect event details.
- To maintain consistency across the .NET ecosystem.

---

### Syntax Rules and Structure

```
public delegate void EventHandler<TEventArgs>(object? sender, TEventArgs e);
```

**Component breakdown**

- `object? sender` — The event source.
- `TEventArgs e` — The event payload.
- `void` — No return.

#### Syntax Rules

- `TEventArgs` conventionally derives from `EventArgs`.
- Raise via `handler?.Invoke(this, args)`.
- Declare with `public event EventHandler<MyArgs>? Changed;`.
- Multicast semantics apply.

#### Constraints and Limitations

- `TEventArgs` can be a struct, but conventionally it is a reference type deriving from `EventArgs`.
- Exceptions in one handler skip the rest.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Event with Custom Payload

```csharp
using System;

public class TemperatureChangedEventArgs : EventArgs
{
    public double OldValue { get; }
    public double NewValue { get; }
    public TemperatureChangedEventArgs(double oldValue, double newValue)
    {
        OldValue = oldValue;
        NewValue = newValue;
    }
}

public class Thermostat
{
    private double _temperature;
    public event EventHandler<TemperatureChangedEventArgs>? TemperatureChanged;

    public void SetTemperature(double newValue)
    {
        var old = _temperature;
        _temperature = newValue;
        TemperatureChanged?.Invoke(this, new TemperatureChangedEventArgs(old, newValue));
    }
}

class Program
{
    static void Main()
    {
        var thermostat = new Thermostat();
        thermostat.TemperatureChanged += (sender, e) =>
            Console.WriteLine($"{e.OldValue} → {e.NewValue}");

        thermostat.SetTemperature(25);   // 0 → 25
    }
}
```

**Expected Output**

```
0 → 25
```

**Why this output occurs**

- The event carries `OldValue` and `NewValue` via `TEventArgs`.

---

#### Example 2: Contravariance in `EventHandler<TEventArgs>`

```csharp
using System;

public class BaseEventArgs : EventArgs { }
public class DerivedEventArgs : BaseEventArgs { }

class Program
{
    static void Main()
    {
        EventHandler<DerivedEventArgs> derivedHandler = (s, e) => Console.WriteLine("Handled derived");
        EventHandler<BaseEventArgs> baseHandler = derivedHandler;   // contravariance
        baseHandler(null, new BaseEventArgs());
    }
}
```

**Expected Output**

```
Handled derived
```

**Why this output occurs**

- `EventHandler<TEventArgs>` is contravariant; a derived-typed handler can be assigned to a base-typed event.

---

### Real-World Cases

- **Domain events** — `OrderPlacedEventArgs`, `UserCreatedEventArgs`.
- **Progress reporting** — `ProgressChangedEventArgs`.
- **File system watchers** — `FileSystemEventArgs`.

---

### References

- EventHandler<TEventArgs> Delegate — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.eventhandler-1
- Standard .NET event patterns — Microsoft Learn - https://learn.microsoft.com/en-us/dotnet/csharp/event-pattern

---

## 10. Built-in Delegate Best Practices

### Definitions

**Core Definition**
Built-in delegate best practices are the guidelines for choosing and using the correct predefined delegate types, avoiding common pitfalls, and aligning with the .NET event and callback conventions.

**Technical Definition**
Best practices derive from the Framework Design Guidelines and C# Programming Guide. Key recommendations include: **prefer built-in delegates over custom ones** unless a custom delegate adds meaningful semantics; **use `Func<T>` for value-returning callbacks and `Action<T>` for side-effecting callbacks**; **use `Predicate<T>`, `Comparison<T>`, and `Converter<TInput, TOutput>` when the API specifically requires them**; **use `EventHandler<TEventArgs>` for events with payload** and `EventHandler` for events without payload; **subscribe and unsubscribe symmetrically** to prevent memory leaks; **use `?.Invoke` for null-safe invocation**; **avoid `DynamicInvoke` in hot paths**; and **prefer `event` over public delegate fields** for public APIs.

**Beginner-Friendly Explanation**
Best practices for built-in delegates say: **use the ones the library gave you** rather than rolling your own; **match the delegate to the scenario** (`Func` for results, `Action` for effects, `EventHandler` for events); and **always unsubscribe when done**.

---

### Purposes

- To **standardise callback types** across APIs.
- To **avoid custom delegate proliferation**.
- To **prevent memory leaks** from orphaned subscriptions.
- To **align with LINQ and event patterns**.
- To make code **interoperable** with the wider .NET ecosystem.

---

### Guidelines and Structure

#### General Guidelines

```
// Prefer built-in delegates
Func<int, bool> isEven = n => n % 2 == 0;    // not a custom delegate
Action<string> log = Console.WriteLine;

// Match the delegate to the scenario
// Func<T, TResult>   → returns a value
// Action<T>          → side effect
// Predicate<T>       → yes/no question (legacy APIs)
// Comparison<T>      → ordering
// Converter<,>       → element-wise conversion

// Use EventHandler<TEventArgs> for events with payload
public event EventHandler<OrderPlacedEventArgs>? OrderPlaced;

// Safe invocation
handler?.Invoke(this, e);

// Symmetric subscription
button.Click += OnClick;
// later...
button.Click -= OnClick;
```

**Component breakdown**

- **Built-in over custom** — Standard delegates simplify APIs.
- **Match the family** — `Func`, `Action`, `Predicate`, `Comparison`, `Converter`, `EventHandler`.
- **Payload events** — `EventHandler<TEventArgs>`.
- **Safe invocation** — `?.Invoke`.
- **Symmetric subscription** — Avoid leaks.

#### Rules and Constraints

- **Do** prefer `Func<T>` and `Action<T>` in new APIs.
- **Do** use `EventHandler<TEventArgs>` for event payloads.
- **Do** use `?.Invoke` for potentially null delegates.
- **Do** unsubscribe symmetrically.
- **Do not** expose public delegate fields; use `event`.
- **Do not** use custom delegates when a built-in one fits.
- **Do not** use `DynamicInvoke` in performance-sensitive paths.

---

### Annotated Code Examples and Expected Outputs

#### Example: Choosing the Right Built-in Delegate

```csharp
using System;

public class OrderPlacedEventArgs : EventArgs
{
    public int OrderId { get; }
    public decimal Total { get; }
    public OrderPlacedEventArgs(int id, decimal total) => (OrderId, Total) = (id, total);
}

public class OrderService
{
    public event EventHandler<OrderPlacedEventArgs>? OrderPlaced;

    public void PlaceOrder(int id, decimal total)
    {
        // ... business logic ...
        OrderPlaced?.Invoke(this, new OrderPlacedEventArgs(id, total));
    }

    public void ProcessOrders(int[] orderIds, Func<int, decimal> totalCalculator,
                              Action<int, decimal> onOrderProcessed)
    {
        foreach (var id in orderIds)
        {
            decimal total = totalCalculator(id);
            onOrderProcessed(id, total);
            PlaceOrder(id, total);
        }
    }
}

class Program
{
    static void Main()
    {
        var service = new OrderService();

        service.OrderPlaced += (sender, e) =>
            Console.WriteLine($"[EVENT] Order {e.OrderId} placed for {e.Total:C}");

        service.ProcessOrders(
            new[] { 1, 2 },
            totalCalculator: id => id * 10m,
            onOrderProcessed: (id, total) =>
                Console.WriteLine($"[ACTION] Processed order {id} with total {total:C}"));
    }
}
```

**Expected Output**

```
[ACTION] Processed order 1 with total $10.00
[EVENT] Order 1 placed for $10.00
[ACTION] Processed order 2 with total $20.00
[EVENT] Order 2 placed for $20.00
```

**Why this output occurs**

- The code uses `Func<int, decimal>` for computation, `Action<int, decimal>` for side effects, and `EventHandler<TEventArgs>` for events—each delegate matching its semantic role.

---

### Real-World Cases

- **LINQ** — `Func<T, TResult>` for projections.
- **DI containers** — `Func<TService>` factories.
- **UI frameworks** — `EventHandler<TEventArgs>` for events.
- **Pipelines** — `Action<T>` for stage execution.

---

### References

- Delegates — C# Programming Guide - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/delegates/
- Standard .NET event patterns — Microsoft Learn - https://learn.microsoft.com/en-us/dotnet/csharp/event-pattern
- Framework Design Guidelines — Delegates - https://learn.microsoft.com/en-us/dotnet/standard/design-guidelines/delegates
- Events — C# Programming Guide - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/events/
- Predicate<T> Delegate — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.predicate-1

---

## Deprecated, Unsafe, Implementation-Dependent, and Version-Specific Notes

| Feature | Status / Note |
|---|---|
| `Action` / `Action<T>` | Introduced in **.NET 2.0** (Action) and **.NET 3.5** (Action<T>); stable. |
| `Func<T>` / `Func<T, TResult>` | Introduced in **.NET 3.5**; stable and preferred. |
| `Predicate<T>` | Introduced in **.NET 2.0**; legacy but supported. |
| `Comparison<T>` | Introduced in **.NET 2.0**; stable. |
| `Converter<,>` | Introduced in **.NET 2.0**; legacy but supported. |
| `EventHandler` / `EventHandler<T>` | Introduced in **.NET 1.0/2.0**; the standard event pattern. |
| Variance on generic delegates | Introduced in **.NET 4.0 / C# 4.0**; reference types only. |
| `DynamicInvoke` | **Slow**; avoid in hot paths. |
| Custom delegate preference | **Modern APIs prefer built-in delegates**; custom delegates remain valid where a custom shape aids clarity. |

---

## References (Consolidated)

- Action Delegate — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.action
- Action<T> Delegate — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.action-1
- Func<TResult> Delegate — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.func-1
- Func<T,TResult> Delegate — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.func-2
- Predicate<T> Delegate — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.predicate-1
- Comparison<T> Delegate — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.comparison-1
- Converter<TInput,TOutput> Delegate — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.converter-2
- EventHandler Delegate — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.eventhandler
- EventHandler<TEventArgs> Delegate — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.eventhandler-1
- Delegates — C# Programming Guide - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/delegates/
- Introduction to delegates and events in C# — Microsoft Learn - https://learn.microsoft.com/en-us/dotnet/csharp/delegates-overview
- Standard .NET event patterns — Microsoft Learn - https://learn.microsoft.com/en-us/dotnet/csharp/event-pattern
- Framework Design Guidelines — Delegates - https://learn.microsoft.com/en-us/dotnet/standard/design-guidelines/delegates
- Events — C# Programming Guide - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/events/

---