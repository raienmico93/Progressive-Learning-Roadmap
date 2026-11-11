# C# Lambda Expressions — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
A **lambda expression** is an **anonymous function** written inline using the `=>` operator (the “goes to” arrow) that can be converted to a delegate type or an **expression tree**. Introduced in **C# 3.0**, lambdas are the modern, idiomatic way to write inline callbacks, LINQ predicates, and functional transformations in C#. They supersede anonymous methods (C# 2.0) in nearly every scenario, offering more concise syntax, support for expression trees, and a richer set of parameter forms.

**Technical Definition**
A lambda expression is an expression of the form `(parameters) => expression-or-block`. The left side specifies the lambda’s parameter list (possibly empty, possibly with explicit or inferred types); the right side is either an **expression** (expression lambda) whose value is the lambda’s return value, or a **block** (statement lambda) containing arbitrary statements. Lambdas are **target-typed**: the compiler determines the delegate or expression-tree type from the context. Parameters can be **implicitly typed** (types inferred from the target delegate) or **explicitly typed**. Since **C# 10**, lambdas support **natural types** (`var` inference), **attributes**, and **explicit return types**; since **C# 12**, lambdas can declare **default parameter values** and `params` arrays. Capturing local variables produces a **closure**: the compiler generates a class (or struct) that stores the captured variables and the lambda body as a method. Expression lambdas that are convertible to `Expression<TDelegate>` are parsed into an **expression tree** at compile time, enabling LINQ providers (e.g., Entity Framework) to translate them into other languages.

**Beginner-Friendly Explanation**
A lambda expression is a **short, inline function** written right where you need it, using the `=>` arrow. Instead of declaring a named method and referencing it, you write `x => x * x` and pass it directly to a method like `Select`. The arrow reads “goes to”: `x` goes to `x * x`. Lambdas are the building blocks of LINQ (`numbers.Where(n => n > 5)`), event handlers (`button.Click += (s, e) => { ... }`), and functional programming in C#. They can **capture** variables from the surrounding scope—called a **closure**—which makes them powerful but also introduces subtle pitfalls (like loop-variable capture and memory retention). Modern C# gives lambdas nearly the same power as named methods: attributes, default parameters, explicit return types, and natural types.

---

### Key Characteristics

- **Inline anonymous functions** — No named method needed; lambdas are written where they are used.
- **Arrow syntax** — `(parameters) => expression` or `(parameters) => { statements }`.
- **Target typing** — The compiler infers the delegate or expression-tree type from context.
- **Expression vs. statement lambdas** — Expression lambdas return a value implicitly; statement lambdas use a block with explicit `return` if needed.
- **Implicit or explicit parameter types** — `(int x) => x` or `x => x`, with type inference from the target delegate.
- **Closures** — Lambdas capture variables from the enclosing scope, extending their lifetime.
- **Expression tree convertible** — Expression lambdas can be converted to `Expression<TDelegate>` for LINQ providers.
- **Natural type (C# 10)** — `var` can infer a lambda's delegate type when necessary.
- **Modern features** — Attributes, default parameter values, `params`, and explicit return types are supported in recent C# versions.

---

### Prerequisites

- Basic understanding of C# delegates (`Func<T>`, `Action<T>`).
- Familiarity with method groups and method invocation.
- Knowledge of local variables, scopes, and closures.
- Awareness of LINQ (for the most common lambda use case).
- Basic understanding of expression trees (for the `IQueryable`/ORM distinction).

---

### Related Programming Areas

- **LINQ** — Almost every LINQ operator accepts lambdas.
- **Delegates** — Lambdas are syntactic sugar for delegate creation.
- **Events** — Lambdas are the idiomatic inline event handler form.
- **Expression trees** — Enable ORM and remote query translation.
- **Functional programming** — Lambdas are the primary tool for higher-order functions.
- **Asynchronous programming** — `async` lambdas are commonly used in event handlers and task continuations.

---

### Core Concepts / Features

1. Lambda Expressions (Overarching Concept)
2. Lambda Syntax
3. Expression Lambdas
4. Statement Lambdas
5. Lambda Parameters
6. Lambda Captures
7. Closure
8. Lambda Best Practices
9. Lambda Performance

Each concept follows a uniform structure: definitions, purposes, syntax rules and structure, annotated code examples with expected outputs, real-world cases, and references.

---

## 1. Lambda Expressions (Overarching Concept)

### Definitions

**Core Definition**
A lambda expression is an inline, anonymous function converted to a delegate or expression tree via the `=>` operator.

**Technical Definition**
A lambda expression is an anonymous function that you can use to create delegates or expression tree types. By using lambda expressions, you can write local functions that can be passed as arguments or returned as the value of function calls. Lambda expressions are particularly helpful for writing LINQ query expressions. The `=>` is the lambda operator; the left side is the parameter list, and the right side is the body (expression or block).

**Beginner-Friendly Explanation**
A lambda is a **throwaway function** written right at the call site. It saves you from declaring a full named method for a one-off callback.

---

### Purposes

- To write **inline callbacks** without separate method declarations.
- To power **LINQ queries** with concise predicates and projections.
- To enable **functional programming** patterns in C#.
- To simplify **event subscriptions**.
- To support **expression trees** for remote query translation.

---

### Syntax Rules and Structure

```
(input-parameters) => expression
(input-parameters) => { statements }
```

**Component breakdown**

- `(input-parameters)` — Optional if the lambda has exactly one implicitly typed parameter.
- `=>` — The lambda operator.
- `expression` or `{ statements }` — The body.

#### Syntax Rules

- The lambda’s parameter list can be empty (`() => ...`), single-parameter without parentheses (`x => ...`), or multiple-parameter with parentheses (`(x, y) => ...`).
- Lambdas are **target-typed**: the compiler infers the delegate or expression-tree type from context.
- The body may be an expression or a block.
- Since C# 10, lambdas can have natural types, attributes, explicit return types, and `static` modifier.
- Since C# 12, lambdas can have default parameter values and `params` arrays.

#### Constraints and Limitations

- Cannot use `yield return` (use named iterator methods or local functions).
- Statement lambdas cannot be converted to expression trees.
- Lambdas capture variables by reference (`ref` semantics for the captured variable).

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Lambda as Delegate

```csharp
using System;

class Program
{
    static void Main()
    {
        Func<int, int> square = x => x * x;
        Console.WriteLine(square(5));   // 25
    }
}
```

**Expected Output**

```
25
```

**Why this output occurs**

- `x => x * x` is an expression lambda assigned to `Func<int, int>`, which the compiler converts into a delegate instance.

---

#### Example 2: Lambda in LINQ

```csharp
using System;
using System.Linq;

class Program
{
    static void Main()
    {
        int[] numbers = { 1, 2, 3, 4, 5 };
        var evens = numbers.Where(n => n % 2 == 0);
        Console.WriteLine(string.Join(", ", evens));   // 2, 4
    }
}
```

**Expected Output**

```
2, 4
```

**Why this output occurs**

- `n => n % 2 == 0` is a predicate lambda converted to `Func<int, bool>` and applied by `Where`.

---

### Real-World Cases

- **LINQ queries** — `Where`, `Select`, `OrderBy`.
- **Event handlers** — `button.Click += (s, e) => { ... };`.
- **Task continuations** — `task.ContinueWith(t => ...)`.
- **Configuration** — `services.AddSingleton<IFoo>(_ => new Foo())`.

---

### References

- Lambda expressions — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/lambda-expressions
- Lambda expressions — C# Programming Guide - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/lambda-expressions
- Expression trees — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/concepts/expression-trees/

---

## 2. Lambda Syntax

### Definitions

**Core Definition**
Lambda syntax is the `(parameters) => body` form used to write a lambda expression.

**Technical Definition**
Lambda expressions use the lambda operator `=>`, which separates the parameter list on the left from the body on the right. The parameter list is enclosed in parentheses when there are zero or multiple parameters, or when explicit types are used; a single implicitly typed parameter can omit parentheses. The body is either an expression (expression lambda) or a block of statements (statement lambda). Since C# 10, lambda declarations can include modifiers (`static`, `async`) and attributes; since C# 12, default and `params` parameters are supported.

**Beginner-Friendly Explanation**
Lambda syntax is **`args => body`**. That is the entire shape. The parentheses are optional for a single parameter.

---

### Purposes

- To provide a **uniform, concise** function-literal syntax.
- To support expression bodies and block bodies.
- To allow implicit and explicit parameter types.
- To enable modern modifiers and attributes (C# 10+).

---

### Syntax Rules and Structure

```
// Empty parameter list
() => expression

// Single implicitly typed parameter
x => expression

// Multiple or explicitly typed parameters
(x, y) => x + y
(int x, int y) => x + y

// Statement lambda
(x) => { var y = x * 2; return y + 1; }

// Static lambda (C# 9)
static x => x * x

// Async lambda
async () => await SomeAsyncCall()

// Attributes (C# 10)
[MyAttribute] (int x) => x * 2

// Explicit return type (C# 10)
int (int x) => x * 2
```

**Component breakdown**

- `()` — Required for zero or multiple parameters; optional for a single implicitly typed parameter.
- `(x, y)` — Multiple parameters, implicitly typed.
- `(int x, int y)` — Explicit parameter types.
- `=>` — The lambda operator.
- Expression or block — The body.

#### Syntax Rules

- Parentheses can be omitted only for a **single implicitly typed** parameter.
- Explicit types require parentheses and matching types for all parameters.
- The body may be an expression (implicit return) or a block (explicit return where required).
- Modifiers (`static`, `async`) and attributes precede the parameter list.
- Explicit return types (C# 10) precede the parameter list.

#### Constraints and Limitations

- `yield return` is not allowed in lambdas.
- Expression lambdas convert to expression trees; statement lambdas do not.
- The `ref`/`out`/`in` modifiers are supported in parameters with matching delegate types.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Parameter Forms

```csharp
using System;

class Program
{
    static void Main()
    {
        Func<int> noArgs = () => 42;
        Func<int, int> oneArg = x => x + 1;
        Func<int, int, int> twoArgs = (x, y) => x + y;
        Func<int, int> explicitArg = (int x) => x * 2;

        Console.WriteLine(noArgs());       // 42
        Console.WriteLine(oneArg(5));      // 6
        Console.WriteLine(twoArgs(2, 3));  // 5
        Console.WriteLine(explicitArg(10)); // 20
    }
}
```

**Expected Output**

```
42
6
5
20
```

**Why this output occurs**

- Each syntax form is valid; the compiler infers the delegate type from the `Func<>` target.

---

#### Example 2: Static and Async Lambdas

```csharp
using System;
using System.Threading.Tasks;

class Program
{
    static async Task Main()
    {
        Func<int, int> square = static x => x * x;
        Console.WriteLine(square(4));   // 16

        Func<Task<int>> asyncFunc = async () =>
        {
            await Task.Delay(10);
            return 99;
        };
        Console.WriteLine(await asyncFunc());   // 99
    }
}
```

**Expected Output**

```
16
99
```

**Why this output occurs**

- `static` prevents capture; `async` allows awaiting inside the lambda body.

---

### Real-World Cases

- **LINQ** — `Where(x => x.IsActive)`, `Select(x => new { x.Id, x.Name })`.
- **Events** — `handler += (s, e) => { ... };`.
- **Async** — `Task.Run(async () => await DoWorkAsync())`.

---

### References

- Lambda expressions — C# reference (syntax) - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/lambda-expressions
- Lambda expressions — C# 10 features - https://learn.microsoft.com/en-us/dotnet/csharp/whats-new/csharp-10#lambda-improvements
- Lambda expressions — C# 12 features - https://learn.microsoft.com/en-us/dotnet/csharp/whats-new/csharp-12#lambda-improvements

---

## 3. Expression Lambdas

### Definitions

**Core Definition**
An expression lambda is a lambda whose body is a **single expression**, whose value is implicitly returned (or, for `void` targets, performed).

**Technical Definition**
An expression lambda has the form `(parameters) => expression`. The expression’s value is returned for non-`void` delegates; for `void` delegates, the expression is evaluated for its side effects. Expression lambdas are the form used in LINQ and expression-tree conversion. They are the most concise lambda form.

**Beginner-Friendly Explanation**
An expression lambda is a **one-line lambda**. The value of the expression is the result.

---

### Purposes

- To write **concise, single-expression** inline functions.
- To enable **expression tree** conversion (ORMs, remote LINQ).
- To simplify **predicates, projections, and transformations**.
- To reduce noise in functional-style code.

---

### Syntax Rules and Structure

```
(parameters) => expression
```

**Component breakdown**

- `expression` — Any valid C# expression returning a value (or performing an operation for `void`).

#### Syntax Rules

- The expression’s value becomes the lambda’s return value (implicitly).
- For `void` delegates, the expression must be a statement expression (e.g., a method call).
- Expression lambdas are convertible to `Expression<TDelegate>` when the target is an expression tree.

#### Constraints and Limitations

- Cannot contain multiple statements (use a statement lambda).
- Cannot contain local variable declarations or control-flow statements.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Expression Lambda as Predicate

```csharp
using System;
using System.Linq;

class Program
{
    static void Main()
    {
        int[] numbers = { 1, 2, 3, 4, 5, 6 };
        var evens = numbers.Where(n => n % 2 == 0);
        Console.WriteLine(string.Join(", ", evens));   // 2, 4, 6
    }
}
```

**Expected Output**

```
2, 4, 6
```

**Why this output occurs**

- `n => n % 2 == 0` is an expression lambda whose value is a `bool` used as a filter.

---

#### Example 2: Expression Lambda for `void` Target

```csharp
using System;

class Program
{
    static void Main()
    {
        Action<string> log = message => Console.WriteLine($"[LOG] {message}");
        log("Started");   // [LOG] Started
    }
}
```

**Expected Output**

```
[LOG] Started
```

**Why this output occurs**

- The expression `Console.WriteLine(...)` is a statement expression usable with `void` delegates.

---

### Real-World Cases

- **LINQ predicates** — `Where(x => x.IsActive)`.
- **LINQ projections** — `Select(x => new { x.Id, x.Name })`.
- **`Action`/`Func` callbacks** — `Action<int> log = x => Console.WriteLine(x)`.

---

### References

- Lambda expressions — Expression lambdas - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/lambda-expressions

---

## 4. Statement Lambdas

### Definitions

**Core Definition**
A statement lambda is a lambda whose body is a **block of statements** enclosed in `{ }`, allowing multiple statements, local variables, and explicit `return`.

**Technical Definition**
A statement lambda has the form `(parameters) => { statements }`. The body may contain zero or more statements, including local declarations, control-flow statements, and an explicit `return` for non-`void` delegates. Statement lambdas **cannot be converted to expression trees** because expression trees cannot contain statement bodies.

**Beginner-Friendly Explanation**
A statement lambda is a **multi-line lambda**. You write `{ ... }` just like a method body.

---

### Purposes

- To write **multi-statement inline functions**.
- To use **local variables and control flow** inside lambdas.
- To perform **side-effecting sequences**.
- To express inline logic that does not fit in a single expression.

---

### Syntax Rules and Structure

```
(parameters) => { statements }
```

**Component breakdown**

- `{ statements }` — Any statements; use `return` for non-`void` delegates.

#### Syntax Rules

- Non-`void` statement lambdas must return a value on all paths.
- Can contain local declarations, loops, conditionals.
- Cannot be converted to `Expression<TDelegate>` (expression trees require expression bodies).
- Can be `async` and use `await`.

#### Constraints and Limitations

- Not usable in expression trees (ORMs require expression lambdas).
- Cannot contain `yield return`.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Multi-Statement Predicate

```csharp
using System;
using System.Collections.Generic;
using System.Linq;

class Program
{
    static void Main()
    {
        int[] numbers = { 1, 2, 3, 4, 5, 6 };
        Func<int, bool> isEvenAndGreaterThanTwo = n =>
        {
            var isEven = n % 2 == 0;
            if (!isEven) return false;
            return n > 2;
        };

        var results = numbers.Where(isEvenAndGreaterThanTwo);
        Console.WriteLine(string.Join(", ", results));   // 4, 6
    }
}
```

**Expected Output**

```
4, 6
```

**Why this output occurs**

- The statement lambda uses local variables and explicit return to express a more complex predicate.

---

#### Example 2: Async Statement Lambda

```csharp
using System;
using System.Threading.Tasks;

class Program
{
    static async Task Main()
    {
        Func<Task<int>> compute = async () =>
        {
            await Task.Delay(10);
            return 42;
        };
        Console.WriteLine(await compute());   // 42
    }
}
```

**Expected Output**

```
42
```

**Why this output occurs**

- The statement lambda contains `await` and returns `Task<int>`.

---

### Real-World Cases

- **Event handlers** — `button.Click += (s, e) => { /* multi-statement */ };`.
- **Async continuations** — `task.ContinueWith(t => { ... });`.
- **LINQ** — When a filter requires multiple checks.

---

### References

- Lambda expressions — Statement lambdas - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/lambda-expressions

---

## 5. Lambda Parameters

### Definitions

**Core Definition**
Lambda parameters are the **inputs** declared on the left side of the `=>` operator. They can be **implicitly typed** (types inferred from the target delegate) or **explicitly typed**, and can use modifiers (`ref`, `out`, `in`, `params`, `scoped`, default values).

**Technical Definition**
Lambda parameters are declared similarly to method parameters. Implicitly typed parameters use the delegate’s parameter types; explicitly typed parameters declare types directly. Lambda parameters can be modified with `ref`, `out`, `in`, and (C# 12) `params` and default values. Discards (`_`) are valid parameter names and are used to ignore arguments. Multiple `_` parameters are allowed in C# 9+ (for discards). Parameters can have attributes (C# 10).

**Beginner-Friendly Explanation**
Lambda parameters are **the inputs**. You can leave the types off (the compiler figures them out from the target delegate) or write them explicitly.

---

### Purposes

- To **accept inputs** for the lambda body.
- To support **implicit typing** for conciseness.
- To support **explicit typing** for clarity or disambiguation.
- To use **modifiers** (`ref`, `out`, `in`) matching the delegate.
- To **discard** unused parameters with `_`.

---

### Syntax Rules and Structure

```
// Implicit types
x => x * 2
(x, y) => x + y

// Explicit types
(int x) => x * 2
(int x, string y) => $"{x}: {y}"

// Modifiers
(ref int x) => x++
(out int x) => { x = 42; return true; }

// Discards
_ => Console.WriteLine("ignored")
(_, _) => Console.WriteLine("ignored both")

// Default values (C# 12)
(int x = 5) => x * 2
```

**Component breakdown**

- `x` — Implicitly typed single parameter.
- `(x, y)` — Multiple implicitly typed parameters.
- `(int x)` — Explicit type.
- `_` — Discard.
- `ref`/`out`/`in` — Passing modes.
- `= 5` — Default value (C# 12).

#### Syntax Rules

- Parentheses required when there are zero, multiple, or explicitly typed parameters.
- A single implicitly typed parameter may omit parentheses.
- Modifiers must match the target delegate’s parameter modifiers.
- Discards can only be used when the argument is truly unneeded.

#### Constraints and Limitations

- Explicit types must be specified for all parameters or none (mixing is allowed in C# 10+ but generally avoid).
- Discards with multiple `_` parameters require C# 9+ (previously only one `_` was valid per lambda).

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Implicit vs. Explicit Parameter Types

```csharp
using System;

class Program
{
    static void Main()
    {
        Func<int, int> implicitSquare = x => x * x;
        Func<int, int> explicitSquare = (int x) => x * x;

        Console.WriteLine(implicitSquare(3));   // 9
        Console.WriteLine(explicitSquare(3));   // 9
    }
}
```

**Expected Output**

```
9
9
```

**Why this output occurs**

- Both forms compile to the same behavior; explicit types document intent.

---

#### Example 2: Discards in Event Handler

```csharp
using System;

class Program
{
    static void Main()
    {
        EventHandler handler = (_, _) => Console.WriteLine("Event fired");
        handler(null, EventArgs.Empty);   // Event fired
    }
}
```

**Expected Output**

```
Event fired
```

**Why this output occurs**

- Both parameters are discarded since they are unused.

---

#### Example 3: `ref` Parameter

```csharp
using System;

public delegate void Increment(ref int value);

class Program
{
    static void Main()
    {
        Increment inc = (ref int x) => x++;
        int n = 0;
        inc(ref n);
        Console.WriteLine(n);   // 1
    }
}
```

**Expected Output**

```
1
```

**Why this output occurs**

- The lambda parameter is declared `ref` to match the delegate’s `ref int` parameter.

---

### Real-World Cases

- **LINQ** — `Where(x => ...)` uses single parameter.
- **Event handlers** — `(_, _) => ...` for unused parameters.
- **Callbacks** — `(ref int x) => ...` for by-ref scenarios.

---

### References

- Lambda expressions — Parameters - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/lambda-expressions

---

## 6. Lambda Captures

### Definitions

**Core Definition**
Lambda captures are **outer variables** that a lambda references. The lambda captures them by reference, extending their lifetime and making them accessible inside the lambda body.

**Technical Definition**
When a lambda references a variable from an enclosing scope (a local variable, parameter, or `this`), the compiler captures that variable by reference into a **closure class**. The variable's lifetime is extended to match the delegate's lifetime. Multiple lambdas can capture the same variable, causing them to share state. Loop variables in `foreach` are captured per-iteration since C# 5; `for` loop variables are captured by reference (single shared variable) and require explicit copying to avoid pitfalls. `static` lambdas (C# 9) **cannot capture** anything from the enclosing scope.

**Beginner-Friendly Explanation**
Capturing means **the lambda remembers a variable from the outside**. When a lambda captures a variable, the compiler keeps that variable alive for as long as the lambda exists. If two lambdas capture the same variable, they see each other's changes.

---

### Purposes

- To allow lambdas to **use surrounding state**.
- To enable **callbacks** that depend on context.
- To support **stateful transformations**.
- To enable **dependency injection** through closures.
- To support **event handlers** that reference instance members.

---

### Syntax Rules and Structure

```
int factor = 10;
Func<int, int> multiply = x => x * factor;   // captures factor
Console.WriteLine(multiply(5));   // 50
```

**Component breakdown**

- `factor` — The captured variable.
- `multiply` — The lambda that captures it.

#### Syntax Rules

- Captures are **by reference** to the variable, not its value.
- Multiple lambdas can share the same captured variable.
- `static` lambdas cannot capture any enclosing state (CS8820).
- `this` (instance members) can be captured; this holds a reference to the enclosing object.
- In `foreach` (C# 5+), the iteration variable is captured per-iteration.
- In `for`, the loop variable is captured once (requires copying).

#### Constraints and Limitations

- Captured variables extend the lifetime of the variable and any object it references.
- `ref`/`out`/`in` parameters of the enclosing method **cannot be captured** (CS1628).
- `this` capture prevents garbage collection of the enclosing object.
- Thread safety: shared captures across threads need synchronization.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Basic Capture

```csharp
using System;

class Program
{
    static void Main()
    {
        int factor = 10;
        Func<int, int> multiply = x => x * factor;

        Console.WriteLine(multiply(5));   // 50

        factor = 20;   // change after capture
        Console.WriteLine(multiply(5));   // 100 (captured by reference)
    }
}
```

**Expected Output**

```
50
100
```

**Why this output occurs**

- The lambda captures `factor` by reference; changing `factor` after capture affects future invocations.

---

#### Example 2: Loop Variable Capture

```csharp
using System;
using System.Collections.Generic;

class Program
{
    static void Main()
    {
        var actions = new List<Action>();
        for (int i = 0; i < 3; i++)
        {
            int copy = i;
            actions.Add(() => Console.WriteLine(copy));
        }
        foreach (var a in actions) a();
        // 0
        // 1
        // 2
    }
}
```

**Expected Output**

```
0
1
2
```

**Why this output occurs**

- Copying the loop variable into a local per iteration produces independent captures.

---

#### Example 3: Static Lambda Cannot Capture

```csharp
using System;

class Program
{
    static void Main()
    {
        int outer = 10;

        // Func<int, int> captured = static x => x + outer; // CS8820

        Func<int, int> safe = static x => x * 2;
        Console.WriteLine(safe(5));   // 10
    }
}
```

**Expected Output**

```
10
```

**Why this output occurs**

- The `static` lambda does not capture `outer`; attempting to would cause compile error CS8820.

---

### Real-World Cases

- **LINQ** — Capture configuration values in predicates.
- **Event handlers** — Capture instance state.
- **Closure-based state** — Counters and accumulators.
- **Async state** — Capturing variables across `await` boundaries.

---

### References

- Lambda expressions — Captures - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/lambda-expressions
- C# 5 breaking change in `foreach` capture - https://learn.microsoft.com/en-us/archive/blogs/ericlippert/closing-over-the-loop-variable-considered-harmful
- Static anonymous functions — C# 9 new features - https://learn.microsoft.com/en-us/dotnet/csharp/whats-new/csharp-9#static-anonymous-functions

---

## 7. Closure

### Definitions

**Core Definition**
A closure is the **combination of a lambda (or anonymous function) and its captured environment**. The compiler generates a class (a “closure class”) to store captured variables, allowing the lambda to outlive the scope in which it was created.

**Technical Definition**
A closure is a **first-class function that captures variables from its lexical scope**. In C#, when a lambda captures variables, the compiler generates a private `<>c__DisplayClass` (or similar) class with fields for each captured variable and a method for the lambda body. The captured variables become fields of that class, and the lambda becomes an instance method invoked on the closure object. This enables the lambda to outlive the scope where the captured variables were originally declared. Closures are fundamental to functional programming, but they introduce **lifetime extension**: the captured variables and the enclosing object (for `this` capture) remain alive until the delegate is garbage-collected.

**Beginner-Friendly Explanation**
A closure is **the lambda plus the environment it remembers**. When a lambda captures a variable, the compiler creates a hidden class to hold that variable, so the lambda can keep using it even after the original method returns.

---

### Purposes

- To enable **stateful lambdas** that persist across invocations.
- To support **functional programming** (higher-order functions).
- To implement **callbacks** that depend on context.
- To enable **event handlers** that hold references to instance data.
- To support **asynchronous continuations** that remember state.

---

### Syntax Rules and Structure

Closures are created implicitly whenever a lambda captures an outer variable:

```
int counter = 0;
Func<int> increment = () => ++counter;   // closure captures counter

Console.WriteLine(increment());   // 1
Console.WriteLine(increment());   // 2
```

**Component breakdown**

- `counter` — Captured variable, stored in a closure class.
- `increment` — Lambda whose body accesses `counter` through the closure.

#### Syntax Rules

- Closure is created **implicitly** on first capture.
- Multiple lambdas in the same scope **share** the same closure class for the same variable.
- `static` lambdas cannot create closures for enclosing state.
- Closures live as long as the delegate (and any other delegates sharing the same closure).

#### Constraints and Limitations

- **Memory retention** — Captured variables and objects stay alive.
- **Thread-safety** — Shared closures need synchronization.
- **Performance** — A closure object is allocated per scope (once per method invocation per closure), not per lambda invocation.
- **Loop capture pitfalls** — Capture-by-reference can lead to surprising results.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Counter Closure

```csharp
using System;

class Program
{
    static Func<int> MakeCounter()
    {
        int count = 0;
        return () => ++count;   // closure over 'count'
    }

    static void Main()
    {
        var counter1 = MakeCounter();
        var counter2 = MakeCounter();

        Console.WriteLine(counter1());   // 1
        Console.WriteLine(counter1());   // 2
        Console.WriteLine(counter2());   // 1 (independent closure)
    }
}
```

**Expected Output**

```
1
2
1
```

**Why this output occurs**

- Each call to `MakeCounter` creates a fresh closure object holding its own `count`.

---

#### Example 2: Shared Closure

```csharp
using System;

class Program
{
    static void Main()
    {
        int shared = 0;
        Action increment = () => shared++;
        Action print = () => Console.WriteLine(shared);

        increment();
        increment();
        print();   // 2
    }
}
```

**Expected Output**

```
2
```

**Why this output occurs**

- Both lambdas share the same closure over `shared`, so the increment is visible to `print`.

---

### Real-World Cases

- **Counters and accumulators** — Stateful functions.
- **Dependency injection** — Capturing configuration in factory lambdas.
- **LINQ** — Capturing predicates from outer variables.
- **Async continuations** — Capturing state across `await` boundaries.

---

### References

- Lambda expressions — Closures - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/lambda-expressions
- Anonymous functions — Closures - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/lambda-expressions
- Closure capture in loops — Eric Lippert's blog - https://learn.microsoft.com/en-us/archive/blogs/ericlippert/closing-over-the-loop-variable-considered-harmful

---

## 8. Lambda Best Practices

### Definitions

**Core Definition**
Lambda best practices are the guidelines for writing lambdas that are **concise, readable, correct, and performant**, while avoiding common pitfalls such as loop variable capture, unnecessary closures, and expression-tree mismatches.

**Technical Definition**
Best practices for lambdas derive from Microsoft's C# programming guide and community experience. Key recommendations include: **use lambdas for short, focused logic**; **extract complex lambdas into named methods or local functions**; **prefer expression lambdas for simple transformations and statement lambdas for multi-statement logic**; **use `static` lambdas when no capture is needed** (C# 9) to prevent accidental capture and reduce allocation; **be mindful of loop variable capture** and copy when necessary; **avoid capturing `this` unnecessarily**; **use expression lambdas with `IQueryable`** (expression trees) and statement lambdas only with `IEnumerable`; and **avoid deeply nested lambdas**.

**Beginner-Friendly Explanation**
Best practices for lambdas are: **keep them short**, **make them static when possible**, **watch out for loop captures**, and **do not nest too many arrows**.

---

### Purposes

- To keep lambdas **readable and maintainable**.
- To **prevent closure pitfalls**.
- To **optimize allocations** with `static` lambdas.
- To choose the right lambda form for the target API.
- To align with **.NET conventions**.

---

### Guidelines and Structure

#### General Guidelines

```
// Prefer concise expression lambdas
Func<int, int> square = x => x * x;

// Extract complex logic into named methods or local functions
static bool IsValidOrder(Order o) => o.Total > 0 && o.Items.Count > 0;
var valid = orders.Where(IsValidOrder);

// Use static lambdas when no capture is needed (C# 9)
Func<int, int> doubleValue = static x => x * 2;

// Copy loop variables when capturing in for loops
for (int i = 0; i < n; i++) { int copy = i; callbacks.Add(() => copy); }

// Prefer expression lambdas in expression trees (IQueryable)
// Avoid capturing this unnecessarily
```

**Component breakdown**

- **Concise lambdas** — Ideal for one-line logic.
- **Extract complex logic** — Named method group or local function.
- **Static lambdas** — Prevent capture, reduce allocations.
- **Loop copy** — Capture a per-iteration copy.
- **Expression trees** — Only expression lambdas convert.
- **Avoid `this` capture** — Prevents unintentional object retention.

#### Rules and Constraints

- **Do** keep lambdas short and focused.
- **Do** use `static` lambdas when no capture is needed.
- **Do** copy loop variables when capturing in `for` loops.
- **Do** prefer expression lambdas in expression-tree contexts.
- **Do not** nest lambdas deeply.
- **Do not** capture `this` if not needed.
- **Do not** use statement lambdas in expression-tree contexts (compile error).

---

### Annotated Code Examples and Expected Outputs

#### Example: Well-Designed Lambda Usage

```csharp
using System;
using System.Collections.Generic;

public class Order { public decimal Total; public List<string> Items = new(); }

class Program
{
    static bool IsValid(Order o) => o.Total > 0 && o.Items.Count > 0;

    static void Main()
    {
        var orders = new List<Order>
        {
            new() { Total = 100m, Items = { "Widget" } },
            new() { Total = 0m, Items = { } }
        };

        // Prefer method group over complex lambda
        var valid = orders.FindAll(IsValid);

        // Simple lambdas stay inline
        var totals = orders.ConvertAll(o => o.Total);

        Console.WriteLine(string.Join(", ", valid.Count));   // 1
        Console.WriteLine(string.Join(", ", totals));         // 100, 0
    }
}
```

**Expected Output**

```
1
100, 0
```

**Why this output occurs**

- `IsValid` is extracted as a named method (clearer than a complex inline lambda); `o => o.Total` remains inline because it is a simple projection.

---

### Real-World Cases

- **LINQ queries** — Simple lambdas for filters and projections.
- **Event handlers** — `static` lambdas for handlers that do not capture.
- **Expression trees** — Expression lambdas for `IQueryable`.
- **Factory methods** — Lambdas for DI registrations.

---

### References

- Lambda expressions — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/lambda-expressions
- Static anonymous functions — C# 9 new features - https://learn.microsoft.com/en-us/dotnet/csharp/whats-new/csharp-9#static-anonymous-functions
- Lambda improvements — C# 10 new features - https://learn.microsoft.com/en-us/dotnet/csharp/whats-new/csharp-10#lambda-improvements

---

## 9. Lambda Performance

### Definitions

**Core Definition**
Lambda performance is the **allocation, capture, and dispatch cost** associated with creating and invoking lambdas. Understanding these costs helps you avoid unnecessary allocations in hot paths while keeping code readable.

**Technical Definition**
Lambda performance depends on several factors: (1) **delegate allocation** — each non-cached lambda creates a delegate instance (though C# caches non-capturing static lambdas); (2) **closure allocation** — capturing lambdas allocate a closure object per scope; (3) **invocation overhead** — delegate invocation has a small per-call cost; (4) **`static` lambdas** — no closure allocation; (5) **cached delegates** — non-capturing lambdas can be cached by the compiler since C# 9+ (and by the runtime for some cases); (6) **expression trees** — parsing and compilation are more expensive than direct delegate use, but often amortized by query caching in ORMs. In hot paths (millions of invocations or allocations per second), avoiding closures and caching delegates can significantly reduce GC pressure.

**Beginner-Friendly Explanation**
Lambdas are **fast, but not free**. Creating a lambda that captures variables allocates a small object (a closure). In a tight loop, that can add up. Using `static` lambdas or caching delegates avoids most of the cost. For LINQ over in-memory collections, the cost is usually negligible compared to the loop body.

---

### Purposes

- To **understand the allocation profile** of lambdas.
- To **avoid closure allocations** in hot paths.
- To **choose between lambdas and method groups** based on cost.
- To **optimize LINQ and event handling** code.
- To **benchmark and measure** lambda performance responsibly.

---

### Syntax Rules and Structure (Performance Patterns)

```
// 1. Cache non-capturing lambdas (the compiler often caches them automatically)
static readonly Func<int, int> Square = x => x * x;

// 2. Use static lambdas to prevent closure allocation (C# 9)
Func<int, int> doubleValue = static x => x * 2;

// 3. Avoid capturing loop variables; use method groups when possible
Func<int, int> abs = Math.Abs;   // no allocation beyond delegate creation

// 4. Cache factory lambdas if called in hot paths
private static readonly Func<Service> _factory = static () => new Service();
```

**Component breakdown**

- **Cached delegates** — Static fields hold lambda delegates for reuse.
- **Static lambdas** — Prevent closure allocations.
- **Method groups** — Convert to delegate with less overhead.
- **Reuse** — Avoid re-creating the lambda per call.

#### Syntax Rules

- Non-capturing lambdas are cached by the compiler in many scenarios, but explicit caching is still advisable in hot paths.
- `static` lambdas prevent closure allocation but still create (or cache) a delegate instance.
- Method groups are convertible to delegates with similar overhead to non-capturing lambdas.

#### Constraints and Limitations

- Capturing lambdas allocate a closure **once per scope execution** (not per invocation).
- Invocations of capturing lambdas have slightly higher cost due to closure field access.
- Expression trees are much more expensive to create and compile than delegates; cache expression trees carefully.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Capturing vs. Static Lambda

```csharp
using System;

class Program
{
    static void Main()
    {
        int factor = 10;

        // Capturing: allocates a closure
        Func<int, int> capturing = x => x * factor;

        // Static: no closure allocation
        Func<int, int> nonCapturing = static x => x * 2;

        Console.WriteLine(capturing(5));    // 50
        Console.WriteLine(nonCapturing(5)); // 10
    }
}
```

**Expected Output**

```
50
10
```

**Why this output occurs**

- The capturing lambda allocates a closure object per `Main` execution; the static lambda does not. Semantically, both produce the expected results.

---

#### Example 2: Caching a Non-Capturing Lambda

```csharp
using System;

class Program
{
    static readonly Func<int, int> Square = static x => x * x;

    static void Main()
    {
        for (int i = 0; i < 1_000_000; i++)
        {
            // No closure allocation; the cached delegate is reused.
            int _ = Square(i);
        }
        Console.WriteLine("Done");   // Done
    }
}
```

**Expected Output**

```
Done
```

**Why this output occurs**

- The static field holds a single cached delegate reused across iterations, eliminating per-call allocations.

---

### Real-World Cases

- **High-frequency LINQ** — In-memory collections with hot loops.
- **Event handlers** — `static` handlers avoid capturing `this` unnecessarily.
- **Factories** — Cached `Func<T>` delegates.
- **Expression trees** — Cache query expressions in ORM scenarios.

---

### References

- Lambda expressions — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/lambda-expressions
- Static anonymous functions — C# 9 new features - https://learn.microsoft.com/en-us/dotnet/csharp/whats-new/csharp-9#static-anonymous-functions
- Performance improvements in .NET 5 — Lambdas and closures - https://devblogs.microsoft.com/dotnet/performance-improvements-in-net-5/#lambdas
- BenchmarkDotNet — https://benchmarkdotnet.org/

---

## Deprecated, Unsafe, Implementation-Dependent, and Version-Specific Notes

| Feature | Status / Note |
|---|---|
| Lambda expressions | Introduced in **C# 3.0**; stable and idiomatic. |
| Statement lambdas | Supported since C# 3.0. |
| Expression-tree conversion | Only expression lambdas are convertible. |
| `static` lambdas | Introduced in **C# 9**; prevents capture. |
| Attributes on lambdas | Introduced in **C# 10**. |
| Natural types for lambdas (`var`) | Introduced in **C# 10**. |
| Explicit return types on lambdas | Introduced in **C# 10**. |
| Default values and `params` on lambdas | Introduced in **C# 12**. |
| Anonymous methods (older alternative) | C# 2.0; **superseded** by lambdas. |
| `yield return` in lambdas | **Not allowed**; use named iterators. |
| Loop capture (pre-C# 5 `foreach`) | **Changed** in C# 5; still applies to `for`. |

---

## References (Consolidated)

- Lambda expressions — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/lambda-expressions
- Lambda expressions — C# Programming Guide - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/lambda-expressions
- Expression trees — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/concepts/expression-trees/
- Static anonymous functions — C# 9 new features - https://learn.microsoft.com/en-us/dotnet/csharp/whats-new/csharp-9#static-anonymous-functions
- Lambda improvements — C# 10 new features - https://learn.microsoft.com/en-us/dotnet/csharp/whats-new/csharp-10#lambda-improvements
- Lambda improvements — C# 12 new features - https://learn.microsoft.com/en-us/dotnet/csharp/whats-new/csharp-12#lambda-improvements
- Closure capture in loops — Eric Lippert's blog - https://learn.microsoft.com/en-us/archive/blogs/ericlippert/closing-over-the-loop-variable-considered-harmful
- Performance improvements in .NET 5 — Lambdas and closures - https://devblogs.microsoft.com/dotnet/performance-improvements-in-net-5/#lambdas
- BenchmarkDotNet — https://benchmarkdotnet.org/
- Anonymous methods — C# Programming Guide - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/statements-expressions-operators/anonymous-methods
- Delegates — C# Programming Guide - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/delegates/

---