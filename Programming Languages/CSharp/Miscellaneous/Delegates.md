# C# Delegates — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
A **delegate** is a C# reference type that represents a **type-safe reference to a method**. It is essentially a **function pointer with compile-time type checking**: it encapsulates a method signature (parameter types and return type) and can hold references to any method whose signature matches. Delegates enable methods to be passed as arguments, stored in variables and collections, and invoked indirectly, making them the foundation of **event handling**, **callbacks**, **LINQ**, and **functional programming** in C#.

**Technical Definition**
A delegate is a type that represents references to methods with a particular parameter list and return type. When you instantiate a delegate, you can associate its instance with any method that has a compatible signature and return type. You can call the method through the delegate instance. Delegates are used to pass methods as arguments to other methods. Event handlers are nothing more than methods that are invoked through delegates. You create a custom method, and a class such as a Windows control can accept a delegate as a property; when a particular event occurs, the delegate invokes the associated method. The `delegate` keyword declares a delegate type. Every delegate type derives from `System.Delegate`; a delegate that holds more than one method derives from `System.MulticastDelegate` (which derives from `Delegate`). Delegates are **immutable**—the `Combine` and `Remove` methods return new delegate instances. Since C# 2.0, method groups can be converted implicitly to delegates; since C# 3.0, lambdas are commonly used; since C# 4.0, delegates support **covariance and contravariance**.

**Beginner-Friendly Explanation**
A delegate is a **safe pointer to a method**. Imagine a TV remote control: the remote is not the TV itself—it just sends commands to a specific TV. A delegate is like the remote: it does not contain code, but it points to a method and can invoke it. Delegates let you store a method in a variable, pass it to another method, or combine several methods into one “multicast” delegate. They are the mechanism behind events (`button.Click += Handler`), LINQ (`Where(x => x > 5)`), and callbacks.

---

### Key Characteristics

- **Type-safe function pointers** — Delegates hold references to methods with compatible signatures.
- **Reference types** — Delegates derive from `System.Delegate`; multicast delegates derive from `System.MulticastDelegate`.
- **Immutable** — Combining or removing methods returns a **new** delegate instance; the original is unchanged.
- **Multicast support** — A delegate can hold multiple methods; invoking it calls all of them in order.
- **Covariance and contravariance** — Since C# 4.0, delegates support `out`/`in` variance.
- **Method group conversion** — C# 2.0+ allows a method name to be assigned directly to a compatible delegate.
- **Lambdas and anonymous methods** — C# 3.0+ allows inline delegate bodies (`x => x * 2`).
- **Invocation list** — A multicast delegate exposes `GetInvocationList()` returning the individual methods.
- **Events are delegates** — Events are built on top of delegates and add encapsulation.

---

### Prerequisites

- Basic understanding of C# methods, parameters, and return types.
- Familiarity with value types vs. reference types.
- Knowledge of interfaces (for comparison with functional interfaces).
- Awareness of lambdas and anonymous methods.
- Basic understanding of events (for the events comparison).

---

### Related Programming Areas

- **Events** — Events are built on top of delegates and add encapsulation.
- **LINQ** — `Func<T>` and `Action<T>` are delegates that power LINQ.
- **Asynchronous programming** — `AsyncCallback` and task continuations use delegates.
- **Functional programming** — Delegates enable higher-order functions and composition.
- **Strategy and observer patterns** — Delegates provide lightweight alternatives to dedicated interfaces.

---

### Core Concepts / Features

1. Delegates (Overarching Concept)
2. Delegate Declaration
3. Delegate Instantiation
4. Delegate Invocation
5. Multicast Delegates
6. Delegate Combination
7. Delegate Removal
8. `Delegate` Class
9. `MulticastDelegate` Class
10. Delegate Best Practices

Each concept follows a uniform structure: definitions, purposes, syntax rules and structure, annotated code examples with expected outputs, real-world cases, and references.

---

## 1. Delegates (Overarching Concept)

### Definitions

**Core Definition**
A delegate is a reference type that encapsulates a method with a specific signature, enabling methods to be passed as values, stored in variables, and invoked indirectly.

**Technical Definition**
A delegate is a type that represents references to methods with a particular parameter list and return type. When you instantiate a delegate, you can associate its instance with any method that has a compatible signature and return type. You can call the method through the delegate instance. Delegates are the underlying mechanism for events and callbacks in C#. All delegate types are sealed classes that derive from `System.MulticastDelegate`.

**Beginner-Friendly Explanation**
A delegate is a **variable that holds a method**. You can pass it around, invoke it, or combine several methods in it.

---

### Purposes

- To enable **method passing** as arguments to other methods.
- To implement **callbacks** and **event handlers**.
- To support **functional programming** patterns (higher-order functions).
- To implement **strategy and observer patterns** without dedicated interfaces.
- To decouple producers from consumers of actions.

---

### Syntax Rules and Structure

```
[access modifier] delegate ReturnType DelegateName(ParameterList);
```

**Component breakdown**

- `delegate` — The keyword that declares a delegate type.
- `ReturnType` — The return type of the methods the delegate can reference.
- `DelegateName` — The delegate identifier.
- `ParameterList` — Parameter types and names; delegates cannot have `ref`/`out`? Actually they can have ref/out parameters.

#### Syntax Rules

- A delegate declaration is **syntactically similar to a method declaration** but without a body and with the `delegate` keyword.
- All delegate types are implicitly sealed, deriving from `System.MulticastDelegate`.
- Delegates can have any access modifier (`public`, `internal`, `private`, etc.).
- Delegates can be generic.
- Delegates can have `ref`, `out`, and `in` parameters.

#### Constraints and Limitations

- Delegate types are not inheritable.
- Delegates cannot be `abstract`, `virtual`, or `sealed` (they are implicitly sealed).
- A delegate cannot have a body (unlike methods).

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Basic Delegate Usage

```csharp
using System;

// 1. Declare the delegate type
public delegate int MathOperation(int x, int y);

class Program
{
    static int Add(int a, int b) => a + b;
    static int Multiply(int a, int b) => a * b;

    static void Main()
    {
        // 2. Instantiate with a method
        MathOperation op = Add;
        Console.WriteLine(op(3, 4));   // 7

        // 3. Reassign to a different method with the same signature
        op = Multiply;
        Console.WriteLine(op(3, 4));   // 12
    }
}
```

**Expected Output**

```
7
12
```

**Why this output occurs**

- The delegate `MathOperation` can reference any method with the signature `int (int, int)`. Assigning `Add` or `Multiply` changes the target method; invoking `op` calls whichever method is currently referenced.

---

#### Example 2: Delegates as Method Parameters

```csharp
using System;

public delegate void MessageHandler(string message);

class Program
{
    static void PrintWithBrackets(string s) => Console.WriteLine($"[{s}]");
    static void PrintWithQuotes(string s) => Console.WriteLine($"\"{s}\"");

    static void ShowMessage(string message, MessageHandler handler) => handler(message);

    static void Main()
    {
        ShowMessage("Hello", PrintWithBrackets);   // [Hello]
        ShowMessage("Hello", PrintWithQuotes);     // "Hello"
    }
}
```

**Expected Output**

```
[Hello]
"Hello"
```

**Why this output occurs**

- `ShowMessage` accepts a `MessageHandler` delegate and invokes it with the message; different delegates produce different formatting.

---

### Real-World Cases

- **Event handlers** — `button.Click += OnClick`.
- **LINQ** — `numbers.Where(n => n > 5)` uses `Func<int, bool>`.
- **Callbacks** — Async operations pass completion callbacks.
- **Strategy pattern** — Plug-in algorithms via delegates.

---

### References

- Delegates — C# Programming Guide - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/delegates/
- Introduction to delegates and events in C# — Microsoft Learn - https://learn.microsoft.com/en-us/dotnet/csharp/delegates-overview

---

## 2. Delegate Declaration

### Definitions

**Core Definition**
Delegate declaration is the syntax that introduces a new delegate type, specifying its name and the signature (parameters and return type) that target methods must match.

**Technical Definition**
A delegate declaration defines a new delegate type. The declaration specifies the delegate’s name, its parameter list, and its return type. The resulting type derives from `System.MulticastDelegate`. Delegate types can be declared at the namespace or nested inside another type. The parameter list and return type form the **invocation signature** that all target methods must be compatible with.

**Beginner-Friendly Explanation**
Declaring a delegate is like **defining a contract for method signatures**: “any method matching this shape is compatible.”

---

### Purposes

- To introduce a new type that can hold method references.
- To specify the signature required of target methods.
- To enable type-safe method passing and storage.
- To define custom event handler types.

---

### Syntax Rules and Structure

```
[access modifier] [modifiers] delegate ReturnType DelegateName([parameters]);
```

**Component breakdown**

- `access modifier` — Optional; default is `internal` for top-level delegates.
- `modifiers` — `new`, `unsafe`, or `public` in some contexts.
- `delegate` — The keyword.
- `ReturnType` — The method's return type (or `void`).
- `DelegateName` — The delegate type identifier.
- `parameters` — The parameter list.

#### Syntax Rules

- Delegate declarations cannot have a body (just a semicolon at the end).
- Parameters can have `ref`, `out`, and `in` modifiers.
- Delegate types may be nested in classes, structs, or interfaces.
- Generic delegates can be declared with a type parameter list.

#### Constraints and Limitations

- Delegate types cannot be `abstract`, `sealed`, `virtual`, `override`, or `static`.
- Delegate declarations cannot include method bodies.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Simple Delegate Declaration

```csharp
using System;

// Delegate declaration at namespace level
public delegate bool Predicate<T>(T item);

class Program
{
    static bool IsLong(string s) => s.Length > 5;

    static void Main()
    {
        Predicate<string> p = IsLong;
        Console.WriteLine(p("Hello"));       // False
        Console.WriteLine(p("Hello, world")); // True
    }
}
```

**Expected Output**

```
False
True
```

**Why this output occurs**

- The generic delegate `Predicate<T>` is declared with a return type `bool`; `IsLong` matches the signature for `T = string`.

---

#### Example 2: Nested Delegate Declaration

```csharp
using System;

public class Logger
{
    public delegate void Write(string message);

    public Write? OnLog;

    public void Log(string msg) => OnLog?.Invoke(msg);
}

class Program
{
    static void Main()
    {
        var logger = new Logger();
        logger.OnLog = m => Console.WriteLine($"[LOG] {m}");
        logger.Log("Started");   // [LOG] Started
    }
}
```

**Expected Output**

```
[LOG] Started
```

**Why this output occurs**

- The nested delegate `Logger.Write` is used as the type of `OnLog`; the lambda is converted to it via target typing.

---

### Real-World Cases

- **Event handler types** — `public delegate void ClickHandler(object sender, EventArgs e)`.
- **Library callbacks** — Custom callbacks for async operations.
- **Generic delegate types** — User-defined variants of `Func`/`Action`.

---

### References

- delegate keyword — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/reference-types
- Delegates — C# language specification - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/language-specification/delegates

---

## 3. Delegate Instantiation

### Definitions

**Core Definition**
Delegate instantiation is the process of **creating a delegate instance** and binding it to a target method. Instances can be created with the `new` operator, through implicit method-group conversion, or from lambda expressions.

**Technical Definition**
A delegate instance can be created in three common ways: (1) using the `new` operator with a method name (`new MyDelegate(MethodName)`), (2) via **method-group conversion** (C# 2.0+: `MyDelegate d = MethodName;`), or (3) via a **lambda expression** or **anonymous method** (C# 3.0+: `MyDelegate d = x => x * 2;`). When instantiated with an **instance method**, the delegate captures both the method and the target instance (`Target` property). When instantiated with a **static method**, the delegate's `Target` is `null`.

**Beginner-Friendly Explanation**
Instantiating a delegate is **attaching the delegate to a real method**. You can use `new`, an implicit conversion, or a lambda.

---

### Purposes

- To create a delegate instance that references a specific method.
- To support both static and instance methods.
- To enable concise syntax via method groups and lambdas.
- To capture target instances for later invocation.

---

### Syntax Rules and Structure

```
// 1. Using new
var d1 = new MyDelegate(MethodName);

// 2. Method-group conversion (C# 2.0+)
MyDelegate d2 = MethodName;

// 3. Lambda expression (C# 3.0+)
MyDelegate d3 = (x, y) => x + y;

// 4. Anonymous method (C# 2.0+)
MyDelegate d4 = delegate (int x, int y) { return x + y; };
```

**Component breakdown**

- `new MyDelegate(...)` — Explicit instantiation; required in C# 1.0.
- `MethodName` — Method group; implicitly converted.
- `(args) => body` — Lambda expression.
- `delegate (args) { body }` — Anonymous method.

#### Syntax Rules

- Method-group conversion works when the method’s signature matches the delegate.
- Instance methods bind the delegate to the target object (`Target` property).
- Static methods produce a delegate with `Target == null`.
- Lambdas and anonymous methods produce delegates that may capture variables via a closure.

#### Constraints and Limitations

- Target method must have a **compatible signature** (parameter types and return type).
- Variance (C# 4.0+) allows more flexibility in delegate-to-method binding.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Three Ways to Instantiate

```csharp
using System;

public delegate void Notify(string message);

class Program
{
    static void ConsoleNotify(string msg) => Console.WriteLine($"[STATIC] {msg}");
    void InstanceNotify(string msg) => Console.WriteLine($"[INSTANCE] {msg}");

    static void Main()
    {
        // 1. Using new (C# 1.0 style)
        var d1 = new Notify(ConsoleNotify);
        d1("Via new");   // [STATIC] Via new

        // 2. Method group
        Notify d2 = ConsoleNotify;
        d2("Via method group");   // [STATIC] Via method group

        // 3. Lambda
        Notify d3 = msg => Console.WriteLine($"[LAMBDA] {msg}");
        d3("Via lambda");   // [LAMBDA] Via lambda

        // 4. Instance method (delegate captures Target)
        var program = new Program();
        Notify d4 = program.InstanceNotify;
        Console.WriteLine(d4.Target == program);   // True
        d4("Via instance");   // [INSTANCE] Via instance
    }
}
```

**Expected Output**

```
[STATIC] Via new
[STATIC] Via method group
[LAMBDA] Via lambda
True
[INSTANCE] Via instance
```

**Why this output occurs**

- All four instantiation styles produce delegates pointing to different methods or the same method via different syntactic forms; `Target` is the bound object for instance methods.

---

#### Example 2: Static vs. Instance Target

```csharp
using System;

public delegate int GetValue();

class Program
{
    static int Static() => 42;
    int Instance() => 100;

    static void Main()
    {
        GetValue s = Static;
        Console.WriteLine(s.Target == null);   // True

        var p = new Program();
        GetValue i = p.Instance;
        Console.WriteLine(i.Target == p);      // True
    }
}
```

**Expected Output**

```
True
True
```

**Why this output occurs**

- Static methods produce delegates with `Target == null`; instance methods set `Target` to the object.

---

### Real-World Cases

- **Event subscription** — `button.Click += HandleClick;`.
- **LINQ callbacks** — `Where(x => x.IsActive)`.
- **Factory providers** — `Func<int> generator = () => 42;`.

---

### References

- Delegates — C# Programming Guide — Instantiation - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/delegates/
- How to declare, instantiate, and use a delegate — C# - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/delegates/how-to-declare-instantiate-and-use-a-delegate

---

## 4. Delegate Invocation

### Definitions

**Core Definition**
Delegate invocation is the act of **calling the method(s) that a delegate references**. You invoke a delegate just like a method: `delegateName(args)`.

**Technical Definition**
Delegates are invoked using the method-call syntax `delegateName(args)`, which is equivalent to calling `delegateName.Invoke(args)`. If the delegate is a **multicast delegate**, every method in its invocation list is called in the order they were added. If the delegate has a non-void return type, the return value of the **last method** is returned; earlier return values are lost. If any target method throws an exception, the remaining methods in the invocation list are not invoked. Since C# 6, you can use the null-conditional operator `?.Invoke(...)` for safe invocation.

**Beginner-Friendly Explanation**
Invoking a delegate is **running the method it points to**. If the delegate holds several methods, all of them run in order.

---

### Purposes

- To **execute the referenced method(s)**.
- To trigger **callbacks** and **event handlers**.
- To run **multicast** notification sequences.
- To handle **safe invocation** with `?.Invoke` on nullable delegates.

---

### Syntax Rules and Structure

```
delegateName(args);
delegateName.Invoke(args);
delegateName?.Invoke(args);
```

**Component breakdown**

- `delegateName(args)` — Shorthand invocation.
- `.Invoke(args)` — Explicit invocation method.
- `?.Invoke(args)` — Safe invocation (C# 6+).

#### Syntax Rules

- Arguments must match the delegate’s parameter list.
- For multicast delegates, methods are called in insertion order.
- For non-void multicast delegates, only the **last** method's return value is returned.
- If a target throws, remaining targets are skipped.

#### Constraints and Limitations

- Invoking a null delegate throws `NullReferenceException` (use `?.Invoke`).
- Multicast return values are effectively restricted to the last method.
- Exceptions from targets are **not** aggregated.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Basic Invocation

```csharp
using System;

public delegate int Operation(int x, int y);

class Program
{
    static void Main()
    {
        Operation add = (x, y) => x + y;
        Console.WriteLine(add(3, 4));           // 7
        Console.WriteLine(add.Invoke(3, 4));   // 7
    }
}
```

**Expected Output**

```
7
7
```

**Why this output occurs**

- `add(3, 4)` and `add.Invoke(3, 4)` are equivalent invocations of the delegate.

---

#### Example 2: Safe Invocation

```csharp
using System;

public delegate void Notify(string message);

class Program
{
    static void Main()
    {
        Notify? notify = null;

        // notify("Hello");       // NullReferenceException
        notify?.Invoke("Hello");  // safe; no exception, no output
        Console.WriteLine("Done"); // Done
    }
}
```

**Expected Output**

```
Done
```

**Why this output occurs**

- The null-conditional operator guards against invoking a null delegate.

---

### Real-World Cases

- **Event raising** — `handler?.Invoke(this, e)`.
- **Callbacks** — `onComplete(result)`.
- **LINQ dispatch** — Delegates passed to LINQ operators are invoked internally.

---

### References

- How to combine delegates (Multicast Delegates) — C# - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/delegates/how-to-combine-delegates-multicast-delegates
- Delegates — C# language specification (invocation) - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/language-specification/delegates

---

## 5. Multicast Delegates

### Definitions

**Core Definition**
A multicast delegate is a delegate that holds **references to more than one method**. When invoked, it calls each method in the invocation list in the order they were added.

**Technical Definition**
A useful property of delegate objects is that multiple objects can be assigned to one delegate instance by using the `+` operator. The multicast delegate contains a list of the assigned delegates. When the multicast delegate is called, it invokes the delegates in the list in order. Only delegates of the same type can be combined. Multicast delegates are instances of `System.MulticastDelegate`, which derives from `System.Delegate`. They expose an invocation list via `GetInvocationList()`. If a target method throws, remaining methods in the list are not invoked.

**Beginner-Friendly Explanation**
A multicast delegate is a **list of methods** that all run when the delegate is invoked. You add methods with `+=` and remove them with `-=`.

---

### Purposes

- To notify **multiple subscribers** with one invocation.
- To implement **event-like** behaviour without full events.
- To support **broadcast** or **pub-sub** patterns.
- To chain method calls in order.

---

### Syntax Rules and Structure

```
// Combine with +=
MyDelegate d = Method1;
d += Method2;
d += Method3;

// Invocation calls all methods in order
d(args);
```

**Component breakdown**

- `+=` — Adds a method (creates a new delegate).
- `-=` — Removes a method (creates a new delegate).
- `GetInvocationList()` — Retrieves individual methods.

#### Syntax Rules

- All combined methods must have **the same delegate type**.
- Methods are invoked in the order they were added.
- Only the **last** return value is propagated for non-void delegates.
- Exceptions skip remaining methods.

#### Constraints and Limitations

- Non-void multicast delegates lose earlier return values.
- Thread safety requires explicit synchronization.
- If one handler throws, remaining handlers are not invoked.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Basic Multicast

```csharp
using System;

public delegate void Notify(string msg);

class Program
{
    static void Email(string msg) => Console.WriteLine($"Email: {msg}");
    static void SMS(string msg) => Console.WriteLine($"SMS: {msg}");
    static void Push(string msg) => Console.WriteLine($"Push: {msg}");

    static void Main()
    {
        Notify d = Email;
        d += SMS;
        d += Push;

        d("Hello");   // Email, SMS, Push in order
    }
}
```

**Expected Output**

```
Email: Hello
SMS: Hello
Push: Hello
```

**Why this output occurs**

- The multicast delegate invokes each method in insertion order.

---

#### Example 2: Inspecting the Invocation List

```csharp
using System;

public delegate void Notify(string msg);

class Program
{
    static void Main()
    {
        Notify d = m => Console.WriteLine($"A: {m}");
        d += m => Console.WriteLine($"B: {m}");

        Console.WriteLine($"Invocation list count: {d.GetInvocationList().Length}");  // 2

        foreach (Notify handler in d.GetInvocationList())
            handler("Hi");
    }
}
```

**Expected Output**

```
Invocation list count: 2
A: Hi
B: Hi
```

**Why this output occurs**

- `GetInvocationList()` returns each individual delegate; invoking them explicitly demonstrates the same order.

---

### Real-World Cases

- **Event handlers** — Multiple subscribers to a button click.
- **Logging pipelines** — Multiple log sinks.
- **Broadcast notifications** — Notify all watchers.

---

### References

- How to combine delegates (Multicast Delegates) — C# - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/delegates/how-to-combine-delegates-multicast-delegates

---

## 6. Delegate Combination

### Definitions

**Core Definition**
Delegate combination is the operation of **adding a method to a delegate's invocation list** using the `+` or `+=` operator (or `Delegate.Combine`), producing a new delegate.

**Technical Definition**
Delegates can be combined using the `+` operator (or `+=` compound assignment) and the static `Delegate.Combine` method. Combining two delegates returns a **new** delegate that contains the invocation lists of both delegates in order. Delegates are **immutable**; the original delegates are unchanged. Combining a null delegate with a non-null delegate returns the non-null one.

**Beginner-Friendly Explanation**
Combining delegates is **chaining methods** so that invoking the combined delegate runs all of them in order.

---

### Purposes

- To build **multicast delegates** from individual delegates.
- To implement event subscriptions.
- To aggregate multiple callbacks into one.
- To build **pipeline** patterns.

---

### Syntax Rules and Structure

```
// Using += (recommended)
MyDelegate combined = Method1;
combined += Method2;

// Using + operator
MyDelegate c = d1 + d2;

// Using Delegate.Combine
MyDelegate c2 = (MyDelegate)Delegate.Combine(d1, d2);
```

**Component breakdown**

- `+` — Combines two delegates into a new one.
- `+=` — Compound combination; reassigns to the variable.
- `Delegate.Combine` — Static method equivalent.

#### Syntax Rules

- Both delegates must be of the same type.
- Combining creates a **new** delegate; the original delegates are unchanged.
- Combining `null` with a delegate returns the non-null delegate.

#### Constraints and Limitations

- Non-void multicast delegates return only the last method's result.
- Exceptions in one method skip the rest.
- Removing a method that is not present is a no-op (no exception).

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Combining with `+` and `+=`

```csharp
using System;

public delegate void Notify();

class Program
{
    static void A() => Console.WriteLine("A");
    static void B() => Console.WriteLine("B");
    static void C() => Console.WriteLine("C");

    static void Main()
    {
        Notify d1 = A;
        Notify d2 = B;
        Notify combined = d1 + d2;   // A, B
        combined += C;                // A, B, C

        combined();   // A, B, C
    }
}
```

**Expected Output**

```
A
B
C
```

**Why this output occurs**

- `+` creates a new delegate combining both; `+=` appends another method.

---

#### Example 2: Immutability of Delegates

```csharp
using System;

public delegate void Notify();

class Program
{
    static void A() => Console.WriteLine("A");
    static void B() => Console.WriteLine("B");

    static void Main()
    {
        Notify d1 = A;
        Notify d2 = d1 + B;   // d2 has A, B

        Console.WriteLine("d1:"); d1();   // A
        Console.WriteLine("d2:"); d2();   // A, B
    }
}
```

**Expected Output**

```
d1:
A
d2:
A
B
```

**Why this output occurs**

- `d1` remains unchanged after combining; `d2` is a new delegate.

---

### Real-World Cases

- **Event subscription** — `eventHandler += Handler`.
- **Composite commands** — Command + undo commands combined.
- **Pipeline stages** — Chaining processing functions.

---

### References

- How to combine delegates (Multicast Delegates) — C# - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/delegates/how-to-combine-delegates-multicast-delegates
- Delegate.Combine — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.delegate.combine

---

## 7. Delegate Removal

### Definitions

**Core Definition**
Delegate removal is the operation of **removing the last occurrence of a method from a delegate's invocation list** using the `-` or `-=` operator (or `Delegate.Remove`), producing a new delegate.

**Technical Definition**
`Delegate.Remove` removes the **last occurrence** of the invocation list of the second delegate from the invocation list of the first delegate. If the resulting list is empty, `Remove` returns `null`. `Delegate.RemoveAll` removes **all occurrences**. `-=` and `-` operators use `Delegate.Remove` semantics. The original delegate is not modified—a **new delegate** is returned.

**Beginner-Friendly Explanation**
Removing a delegate is **unsubscribing a method** from the chain.

---

### Purposes

- To **unsubscribe** from events.
- To remove a method from a multicast delegate.
- To manage dynamic handler lists.
- To avoid memory leaks by removing references.

---

### Syntax Rules and Structure

```
// Using -=
combined -= Method1;

// Using - operator
MyDelegate result = d1 - d2;

// Using Delegate.Remove
MyDelegate r = (MyDelegate)Delegate.Remove(d1, d2);

// Remove all occurrences
MyDelegate r2 = (MyDelegate)Delegate.RemoveAll(d1, d2);
```

**Component breakdown**

- `-` / `-=` — Removes the **last occurrence**.
- `Delegate.Remove` — Static equivalent.
- `Delegate.RemoveAll` — Removes all occurrences.

#### Syntax Rules

- Removes the **last** occurrence of the second delegate's method.
- If the invocation list becomes empty, the result is `null`.
- Removing a non-existent method is a **no-op**.
- The original delegate is not modified.

#### Constraints and Limitations

- Only removes one occurrence per call.
- Order matters for `Remove` when duplicates exist.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Removing a Handler

```csharp
using System;

public delegate void Notify();

class Program
{
    static void A() => Console.WriteLine("A");
    static void B() => Console.WriteLine("B");

    static void Main()
    {
        Notify d = A;
        d += B;
        d += A;   // A, B, A

        d();                // A, B, A
        d -= A;             // removes last A → A, B
        Console.WriteLine("After removal:");
        d();                // A, B
    }
}
```

**Expected Output**

```
A
B
A
After removal:
A
B
```

**Why this output occurs**

- `-=` removes the **last occurrence** of the method `A`, leaving one `A` and one `B`.

---

#### Example 2: Removing to Null

```csharp
using System;

public delegate void Notify();

class Program
{
    static void A() => Console.WriteLine("A");

    static void Main()
    {
        Notify d = A;
        d -= A;
        Console.WriteLine(d == null);   // True
    }
}
```

**Expected Output**

```
True
```

**Why this output occurs**

- Removing the only method leaves a null delegate.

---

### Real-World Cases

- **Event unsubscription** — `button.Click -= Handler`.
- **Disposable patterns** — Removing handlers when disposing.
- **Dynamic dispatch** — Managing runtime handler sets.

---

### References

- How to combine delegates (Multicast Delegates) — C# (removal) - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/delegates/how-to-combine-delegates-multicast-delegates
- Delegate.Remove — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.delegate.remove
- Delegate.RemoveAll — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.delegate.removeall

---

## 8. `Delegate` Class

### Definitions

**Core Definition**
`System.Delegate` is the **base class** for all delegate types. It provides static methods and properties for working with delegates, such as `Combine`, `Remove`, `RemoveAll`, `DynamicInvoke`, `GetInvocationList`, `Target`, and `Method`.

**Technical Definition**
All delegate types derive, either directly or indirectly, from `System.Delegate`. `System.Delegate` provides reflection-friendly APIs and static methods to combine and remove delegates, inspect `Target` and `Method`, and invoke dynamically via `DynamicInvoke`. Compiler-generated delegate types never explicitly derive from `Delegate`; they derive from `MulticastDelegate`, which itself derives from `Delegate`. `Delegate` is not inheritable by user code.

**Beginner-Friendly Explanation**
`Delegate` is the **base class** for every delegate. It provides static helpers for working with delegates generically.

---

### Purposes

- To provide **static helpers** (`Combine`, `Remove`, `RemoveAll`).
- To expose **`Target`** (the instance) and **`Method`** (the `MethodInfo`).
- To enable **`DynamicInvoke`** for late-bound invocation.
- To enable reflection-based delegate inspection.

---

### Syntax Rules and Structure

```
// Static methods
Delegate combined = Delegate.Combine(d1, d2);
Delegate removed = Delegate.Remove(d1, d2);
Delegate removedAll = Delegate.RemoveAll(d1, d2);

// Instance properties
object? target = d.Target;
MethodInfo method = d.Method;

// Dynamic invocation
object? result = d.DynamicInvoke(args);
```

**Component breakdown**

- `Combine` — Joins two delegate lists into a new delegate.
- `Remove` — Removes last occurrence.
- `RemoveAll` — Removes all occurrences.
- `Target` — Object bound to the delegate (null for static methods).
- `Method` — The `MethodInfo` describing the target method.
- `DynamicInvoke` — Late-bound invocation accepting `object[]`.

#### Syntax Rules

- `Delegate` is the root of the delegate hierarchy.
- Static methods accept `Delegate` parameters and return `Delegate`.
- `DynamicInvoke` returns `object?`.

#### Constraints and Limitations

- `DynamicInvoke` is slow and not type-safe.
- `Target`/`Method` are useful for reflection but rarely needed in application code.
- `Delegate` cannot be derived from directly.

---

### Annotated Code Examples and Expected Outputs

#### Example: Using `Delegate` Static Methods

```csharp
using System;

public delegate void Notify(string msg);

class Program
{
    static void A(string m) => Console.WriteLine($"A: {m}");
    static void B(string m) => Console.WriteLine($"B: {m}");

    static void Main()
    {
        Notify d1 = A;
        Notify d2 = B;

        var combined = (Notify)Delegate.Combine(d1, d2);
        combined("Hi");   // A: Hi, B: Hi

        Console.WriteLine($"Targets null: {d1.Target == null}");   // True

        var removed = (Notify?)Delegate.Remove(combined, d1);
        removed?.Invoke("After");   // B: After
    }
}
```

**Expected Output**

```
A: Hi
B: Hi
Targets null: True
B: After
```

**Why this output occurs**

- `Delegate.Combine` merges both lists; `Delegate.Remove` removes `d1` from the combination.

---

### Real-World Cases

- **Reflection-based dispatch** — `DynamicInvoke` in serialization frameworks.
- **Generic handler registration** — Using `Delegate.Combine` when the delegate type is not known at compile time.

---

### References

- Delegate class — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.delegate

---

## 9. `MulticastDelegate` Class

### Definitions

**Core Definition**
`System.MulticastDelegate` is the **base class for all compiler-generated delegate types**. It extends `System.Delegate` with support for **invocation lists** (multicast behaviour).

**Technical Definition**
`MulticastDelegate` is the class that all user-defined delegate types implicitly derive from. It provides `GetInvocationList()` and multicast invocation semantics. When a delegate is combined with another, a `MulticastDelegate` is created whose invocation list contains both. Invoking a `MulticastDelegate` invokes every method in its invocation list, in order, and returns the value of the last method for non-void delegates.

**Beginner-Friendly Explanation**
`MulticastDelegate` is the **actual base class of delegates you declare**. It is what enables multiple methods to be chained.

---

### Purposes

- To support **multicast invocation** (multiple methods).
- To expose **`GetInvocationList()`** for inspection.
- To provide the **runtime infrastructure** for `+=`/`-=`.

---

### Syntax Rules and Structure

```
public abstract class MulticastDelegate : Delegate
{
    public sealed override Delegate[] GetInvocationList();
    // ...
}
```

**Component breakdown**

- `MulticastDelegate` — Base class of compiler-generated delegates.
- `GetInvocationList()` — Returns the list of individual delegates.
- Multicast invocation semantics.

#### Syntax Rules

- Every user-declared delegate derives from `MulticastDelegate`.
- `GetInvocationList()` returns a `Delegate[]` of individual targets.
- Multicast invocation calls methods in order.

#### Constraints and Limitations

- `MulticastDelegate` cannot be derived from directly in user code.
- Non-void return values are truncated to the last method's return.
- Exceptions from one target skip remaining targets.

---

### Annotated Code Examples and Expected Outputs

#### Example: `GetInvocationList`

```csharp
using System;

public delegate void Notify();

class Program
{
    static void A() => Console.WriteLine("A");
    static void B() => Console.WriteLine("B");

    static void Main()
    {
        Notify d = A;
        d += B;

        Delegate[] list = d.GetInvocationList();
        Console.WriteLine($"Count: {list.Length}");  // 2

        foreach (var handler in list)
            handler.DynamicInvoke();
    }
}
```

**Expected Output**

```
Count: 2
A
B
```

**Why this output occurs**

- `GetInvocationList()` returns two delegates; `DynamicInvoke` invokes each individually.

---

### Real-World Cases

- **Event inspection** — Enumerate handlers for diagnostics.
- **Debugging** — Inspect which methods are subscribed.

---

### References

- MulticastDelegate class — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.multicastdelegate

---

## 10. Delegate Best Practices

### Definitions

**Core Definition**
Delegate best practices are the design guidelines that ensure delegates are used **safely, clearly, and efficiently**, avoiding common pitfalls such as null invocation, memory leaks, and performance issues.

**Technical Definition**
Best practices for delegates derive from the .NET Framework Design Guidelines and Microsoft's C# programming guide. Key recommendations include: **use the built-in `Func<>` and `Action<>` delegates** unless a custom delegate type adds clarity; **use `?.Invoke()`** to invoke potentially null delegates; **subscribe and unsubscribe symmetrically** to avoid memory leaks; **be aware that exceptions in one handler skip remaining handlers**; **use `Delegate.RemoveAll` to remove all occurrences**; **prefer events over public delegate fields** for subscription management; **avoid `DynamicInvoke` in hot paths**; and **use the `EventHandler<T>` pattern for event handlers**.

**Beginner-Friendly Explanation**
Best practices for delegates are: **use built-in delegates where possible**, **always use `?.Invoke` to avoid null crashes**, **unsubscribe when done**, and **prefer events over public fields**.

---

### Purposes

- To prevent null-reference exceptions.
- To avoid memory leaks from retained subscriptions.
- To improve readability with built-in delegates.
- To choose the right abstraction (delegate vs. event vs. interface).
- To align with .NET conventions.

---

### Guidelines and Structure

#### General Guidelines

```
// 1. Prefer built-in delegates
Func<int, bool> isEven = n => n % 2 == 0;
Action<string> log = Console.WriteLine;

// 2. Use null-safe invocation
handler?.Invoke(sender, args);

// 3. Unsubscribe symmetrically
button.Click += OnClick;
// later...
button.Click -= OnClick;

// 4. Use events rather than public delegate fields
public event EventHandler? Click;

// 5. Avoid DynamicInvoke in hot paths

// 6. Use EventHandler<T> for custom events
public event EventHandler<MyEventArgs>? MyEvent;
```

**Component breakdown**

- **Built-in delegates** — `Func<>` and `Action<>` cover most scenarios.
- **`?.Invoke`** — Prevents null-reference exceptions.
- **Symmetric subscription** — Avoid leaks and duplicate handler calls.
- **`event` keyword** — Encapsulates subscription management.
- **`EventHandler<T>`** — The standard pattern for events.

#### Rules and Constraints

- **Do** prefer `Func<>` and `Action<>` over custom delegate types when possible.
- **Do** use `?.Invoke` for invocation of potentially null delegates.
- **Do** unsubscribe from events you subscribe to.
- **Do** use `event` for public callback surfaces.
- **Do** use `EventHandler<T>` for event declarations.
- **Do not** expose a public delegate field; use `event`.
- **Do not** assume all handlers will execute if one throws.

---

### Annotated Code Examples and Expected Outputs

#### Example: Well-Designed Event Subscription

```csharp
using System;

public class Button
{
    public event EventHandler? Click;

    public void SimulateClick() => Click?.Invoke(this, EventArgs.Empty);
}

class Program
{
    static void OnClick(object? sender, EventArgs e) =>
        Console.WriteLine("Clicked!");

    static void Main()
    {
        var button = new Button();
        button.Click += OnClick;
        button.SimulateClick();   // Clicked!

        button.Click -= OnClick;
        button.SimulateClick();   // (no output)
    }
}
```

**Expected Output**

```
Clicked!
```

**Why this output occurs**

- The event uses the standard `EventHandler` pattern, `?.Invoke` is null-safe, and the handler is unsubscribed symmetrically.

---

### Real-World Cases

- **UI frameworks** — `EventHandler` for button clicks.
- **APIs** — `Func<>` and `Action<>` callbacks.
- **Libraries** — Subscription-based notifications via events.

---

### References

- Delegates — C# Programming Guide (best practices) - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/delegates/
- Framework Design Guidelines — Delegates - https://learn.microsoft.com/en-us/dotnet/standard/design-guidelines/delegates
- Events — C# Programming Guide - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/events/

---

## Deprecated, Unsafe, Implementation-Dependent, and Version-Specific Notes

| Feature | Status / Note |
|---|---|
| `new MyDelegate(Method)` | Core since **C# 1.0**; still works but method groups are preferred. |
| Method-group conversion | Introduced in **C# 2.0**; preferred for clarity. |
| Anonymous methods (`delegate { }`) | Introduced in **C# 2.0**; superseded by lambdas in most code. |
| Lambdas (`x => ...`) | Introduced in **C# 3.0**; preferred for inline delegate bodies. |
| Covariance and contravariance for delegates | Introduced in **C# 4.0**; reference types only. |
| `?.Invoke` | Introduced in **C# 6.0**; preferred for null-safe invocation. |
| `MulticastDelegate` vs. `Delegate` | All user-declared delegates derive from `MulticastDelegate`. |
| `DynamicInvoke` | **Slow**; avoid in hot paths. |
| Public delegate fields | **Discouraged**; use `event` for encapsulation. |
| `Delegate.RemoveAll` | Available since .NET Framework 2.0; removes all occurrences. |

---

## References (Consolidated)

- Delegates — C# Programming Guide - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/delegates/
- Introduction to delegates and events in C# — Microsoft Learn - https://learn.microsoft.com/en-us/dotnet/csharp/delegates-overview
- How to declare, instantiate, and use a delegate — C# - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/delegates/how-to-declare-instantiate-and-use-a-delegate
- How to combine delegates (Multicast Delegates) — C# - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/delegates/how-to-combine-delegates-multicast-delegates
- Delegate class — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.delegate
- MulticastDelegate class — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.multicastdelegate
- Delegate.Combine — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.delegate.combine
- Delegate.Remove — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.delegate.remove
- Delegate.RemoveAll — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.delegate.removeall
- Delegates — C# language specification - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/language-specification/delegates
- delegate keyword — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/reference-types
- Framework Design Guidelines — Delegates - https://learn.microsoft.com/en-us/dotnet/standard/design-guidelines/delegates
- Events — C# Programming Guide - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/events/

---