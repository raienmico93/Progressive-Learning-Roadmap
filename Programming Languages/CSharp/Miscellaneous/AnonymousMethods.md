# C# Anonymous Methods — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
An **anonymous method** is an **inline, unnamed block of code** declared with the `delegate` keyword that can be assigned to a delegate variable or passed as a delegate argument. Introduced in **C# 2.0**, anonymous methods allow you to write method bodies directly at the point where a delegate is needed—without declaring a separate named method. They were the **direct precursor to lambda expressions** (C# 3.0) and remain valid, fully supported C# syntax today, though lambdas are the preferred modern form.

**Technical Definition**
An anonymous method is an **anonymous function** expressed using the `delegate` operator: `delegate (parameters) { statements }`. The compiler generates a hidden method for the anonymous body and creates a delegate instance pointing to it, capturing any referenced outer variables into a **closure class**. Anonymous methods support **parameter lists** (or can omit them entirely, unlike lambdas) and **statement blocks** (not single expressions). They are **convertible to any delegate type** whose signature matches, and they participate in **multicast** delegation. Anonymous methods **cannot** contain `yield return`, `break`, `continue`, or `goto` statements that jump outside the anonymous method body, cannot access `ref`/`out` parameters of the enclosing method, and cannot be used in **expression trees** (only statement trees via `Expression<...>`? Actually, anonymous methods are **not** convertible to expression trees—only lambdas are). They also **cannot be marked with attributes**. Since C# 3.0, lambdas are generally preferred because they are more concise, but anonymous methods remain fully supported for backward compatibility and for cases where an empty parameter list is desirable.

**Beginner-Friendly Explanation**
An anonymous method is a **method without a name, written right where you need it**. Before C# 2.0, if you wanted to pass a callback to a method, you had to declare a full named method somewhere else in your class and then reference it. Anonymous methods let you write the callback **inline**: `button.Click += delegate { Console.WriteLine("Clicked!"); };`. They are like a **sticky note** attached to the delegate—the code is right there, no separate declaration needed. Anonymous methods are the **older sibling** of lambdas: lambdas are shorter and more expressive, but anonymous methods still work perfectly and can do a couple of things lambdas cannot (like omitting the parameter list entirely). Modern C# code almost always uses lambdas instead, but understanding anonymous methods is essential for reading older C# codebases, for understanding how closures and compiler-generated code work, and for the occasional scenario where the `delegate { }` form is clearer.

---

### Key Characteristics

- **Inline, unnamed code blocks** — Anonymous methods are declared where they are used, with no separate method name.
- **`delegate` keyword form** — Syntax is `delegate (parameters) { body }`; the parameter list may be **omitted entirely**, which lambdas cannot do.
- **Statement bodies** — Anonymous methods use **block bodies** (`{ ... }`), not expression bodies.
- **Full closure support** — They can capture local variables and parameters of the enclosing method, creating closures.
- **Convertible to any compatible delegate** — No specific delegate type is required; target typing applies.
- **Multicast-compatible** — Can be combined with `+`/`+=` and removed with `-`/`-=`.
- **Not usable with expression trees** — Only lambdas convert to `Expression<T>`; anonymous methods do not.
- **No attribute support** — Anonymous methods cannot be decorated with attributes.
- **No `ref`/`out` enclosing-parameter access** — Cannot use the enclosing method's `ref`/`out` parameters.

---

### Prerequisites

- Basic understanding of C# delegates (declaration, instantiation, invocation).
- Familiarity with delegates as method references and multicast delegates.
- Knowledge of local variables and scope.
- Awareness of lambdas (for comparison and modern alternatives).
- Basic understanding of closure semantics.

---

### Related Programming Areas

- **Delegates** — Anonymous methods are a way to create delegate instances inline.
- **Lambdas** — The modern replacement for anonymous methods; more concise and expression-tree-capable.
- **Events** — Anonymous methods are often used to subscribe to events without declaring named handlers.
- **LINQ** — Anonymous methods predate LINQ but are rarely used with it because lambdas are preferred.
- **Functional programming** — Anonymous methods introduced inline functions to C#.

---

### Core Concepts / Features

1. Anonymous Methods (Overarching Concept)
2. `delegate` Keyword
3. Anonymous Method Syntax
4. Anonymous Method Limitations
5. Anonymous Method Best Practices

Each concept follows a uniform structure: definitions, purposes, syntax rules and structure, annotated code examples with expected outputs, real-world cases, and references.

---

## 1. Anonymous Methods (Overarching Concept)

### Definitions

**Core Definition**
An anonymous method is a **nameless block of code** declared inline with the `delegate` keyword, which the compiler turns into a delegate instance.

**Technical Definition**
Anonymous methods (C# 2.0) provide a way to write an **unnamed block of code** as a delegate. The syntax is `delegate (optional-params) { statements }`. The compiler generates a **hidden method** for the body and a **closure class** if outer variables are captured. Anonymous methods are convertible to any delegate type with a compatible signature. They were superseded by lambdas in C# 3.0 but remain fully supported.

**Beginner-Friendly Explanation**
An anonymous method is a **method without a name, written right where you need it**.

---

### Purposes

- To **avoid declaring separate named methods** for short one-off callbacks.
- To keep **callback logic close to its usage**, improving locality.
- To support **inline event handlers** without boilerplate.
- To provide a **pre-lambda** mechanism for inline delegates (backward compatibility).
- To enable **closures** that capture surrounding state.

---

### Syntax Rules and Structure

```
// Anonymous method with explicit parameters
SomeDelegate d = delegate (int x, int y) { return x + y; };

// Anonymous method with omitted parameter list
SomeDelegate d2 = delegate { Console.WriteLine("No params"); };
```

**Component breakdown**

- `delegate` — The keyword (used as an operator here, not as a declaration).
- `(optional-params)` — Optional parameter list; can be **omitted entirely**.
- `{ statements }` — The statement block.

#### Syntax Rules

- The parameter list can be **omitted**, making it compatible with any delegate type.
- If present, the parameter list must match the delegate's signature.
- The body is a **statement block** (`{ ... }`), not a single expression.
- Anonymous methods participate in closure capture like lambdas.
- Anonymous methods are **not** convertible to expression trees.

#### Constraints and Limitations

- Cannot be used with expression trees (`Expression<Func<int,int>>`).
- Cannot use `ref`/`out` parameters of the enclosing method.
- Cannot contain `goto`, `break`, or `continue` that would jump **outside** the body.
- Cannot be decorated with attributes.
- Cannot contain `yield return` or `yield break`.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Basic Anonymous Method

```csharp
using System;

public class Program
{
    public static void Main()
    {
        Func<int, int, int> add = delegate (int a, int b) { return a + b; };
        Console.WriteLine(add(3, 4));   // 7
    }
}
```

**Expected Output**

```
7
```

**Why this output occurs**

- The compiler generates a hidden method for the body `{ return a + b; }` and creates a delegate instance. `add(3, 4)` invokes that method.

---

#### Example 2: Anonymous Method with Omitted Parameter List

```csharp
using System;

public class Program
{
    public static void Main()
    {
        EventHandler handler = delegate { Console.WriteLine("Event raised"); };
        handler(null, EventArgs.Empty);   // Event raised
    }
}
```

**Expected Output**

```
Event raised
```

**Why this output occurs**

- The parameter list is omitted, making the anonymous method compatible with `EventHandler` (which has `(object?, EventArgs)` parameters). The method simply ignores the parameters.

---

#### Example 3: Closure Capture

```csharp
using System;

public class Program
{
    public static void Main()
    {
        int capturedValue = 100;
        Func<int> getValue = delegate { return capturedValue; };
        Console.WriteLine(getValue());   // 100
    }
}
```

**Expected Output**

```
100
```

**Why this output occurs**

- The anonymous method captures `capturedValue` from the enclosing method's scope. The compiler generates a closure class to hold the captured variable.

---

### Real-World Cases

- **Event handlers** — `button.Click += delegate { ... };`.
- **Legacy codebases** — Code written before C# 3.0 lambdas.
- **Callback registration** — Inline action handlers.
- **Learning closures** — The canonical closure example in C# 2.0.

---

### References

- Anonymous methods — C# Programming Guide - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/statements-expressions-operators/anonymous-methods
- Delegates — C# Programming Guide - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/delegates/

---

## 2. `delegate` Keyword

### Definitions

**Core Definition**
In the context of anonymous methods, the `delegate` keyword functions as an **operator** that introduces an inline unnamed method, distinct from its role as a **type declaration** keyword used to declare delegate types.

**Technical Definition**
The `delegate` keyword has **two distinct roles** in C#: (1) as a **declaration keyword** to introduce a delegate type (e.g., `public delegate int Op(int a, int b);`), and (2) as an **operator** to introduce an anonymous method (e.g., `Op add = delegate (int a, int b) { return a + b; };`). The compiler distinguishes the two based on context. In the anonymous-method form, `delegate` is followed by an optional parameter list and a required statement block; in the declaration form, it is followed by a return type and an identifier.

**Beginner-Friendly Explanation**
`delegate` does **two jobs**: “**declare a delegate type**” and “**write an unnamed method inline**.” The context tells you which.

---

### Purposes

- To **introduce an anonymous method** inline.
- To **distinguish anonymous methods** from lambdas (which use `=>`).
- To provide the only syntax for **omitting the parameter list** entirely (lambdas require `()` or at least one parameter).

---

### Syntax Rules and Structure

```
// Role 1: delegate type declaration
public delegate int Operation(int a, int b);

// Role 2: anonymous method operator
Operation add = delegate (int a, int b) { return a + b; };

// Role 2 with omitted parameters
Operation noOp = delegate { return 0; };
```

**Component breakdown**

- **Declaration form** — `delegate ReturnType Name(parameters);`
- **Anonymous method form** — `delegate (parameters) { body }` or `delegate { body }`.

#### Syntax Rules

- In the anonymous form, `delegate` is followed by an **optional** parameter list and a **required** block.
- The block must be `{ ... }` — a single expression is not allowed.
- The `delegate` keyword in the anonymous form is **not** followed by an identifier.
- The compiler resolves ambiguity based on whether an identifier and return type follow.

#### Constraints and Limitations

- Anonymous methods cannot be declared at namespace or class scope (only as expressions inside methods).
- Anonymous methods cannot be generic.
- Anonymous methods cannot have attributes.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Both Roles Side by Side

```csharp
using System;

public class Program
{
    // Role 1: delegate type declaration
    public delegate int Operation(int a, int b);

    public static void Main()
    {
        // Role 2: anonymous method operator
        Operation add = delegate (int a, int b) { return a + b; };
        Operation multiply = delegate (int a, int b) { return a * b; };

        Console.WriteLine(add(3, 4));       // 7
        Console.WriteLine(multiply(3, 4));  // 12
    }
}
```

**Expected Output**

```
7
12
```

**Why this output occurs**

- The `delegate` keyword declares `Operation`, then creates two anonymous methods bound to it.

---

#### Example 2: Omitted Parameter List

```csharp
using System;

public class Program
{
    public static void Main()
    {
        Action greet = delegate { Console.WriteLine("Hello"); };
        greet();   // Hello
    }
}
```

**Expected Output**

```
Hello
```

**Why this output occurs**

- The parameter list is omitted, making the method compatible with `Action`. Lambdas would require `() => ...`.

---

### Real-World Cases

- **Pre-lambda code** — Reading C# 2.0 codebases.
- **Parameter-less event handlers** — Where `delegate { }` is shorter than `() => { }`.
- **Teaching C# history** — Understanding the evolution from delegates to lambdas.

---

### References

- delegate keyword — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/reference-types
- Anonymous methods — `delegate` operator - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/statements-expressions-operators/anonymous-methods

---

## 3. Anonymous Method Syntax

### Definitions

**Core Definition**
Anonymous method syntax is the specific form `delegate (parameters) { statements }` used to write an inline anonymous method, with the parameter list being **optional**.

**Technical Definition**
Anonymous method expressions have the form `delegate (parameter-list) { block }`, where the parameter list is optional. When the parameter list is omitted, the anonymous method is compatible with any delegate type whose signature's parameters are all ignored. The block is a statement block (`{ ... }`) containing any valid statements (except `yield`, `goto` out of scope, `break`/`continue` out of scope, and access to enclosing `ref`/`out` parameters).

**Beginner-Friendly Explanation**
Anonymous method syntax is **`delegate { ... }`** or **`delegate (args) { ... }`**. That is the whole shape.

---

### Purposes

- To provide the **explicit syntax** for inline anonymous methods.
- To allow the **parameter list to be omitted** for maximum brevity.
- To express **multi-statement bodies** inline.
- To enable **closures** with explicit parameter handling.

---

### Syntax Rules and Structure

```
// Full syntax
delegate (Type1 param1, Type2 param2) { statements }

// Omitted parameter list
delegate { statements }
```

**Component breakdown**

- `delegate` — Required operator keyword.
- `(parameter-list)` — Optional; if present, must match the delegate's signature.
- `{ statements }` — Required block body.

#### Syntax Rules

- The block body must contain zero or more statements.
- The parameter list, when present, must match the delegate type's signature in count and types.
- Parameter types and names may be explicit; type inference does not apply (unlike lambda).
- The body must not contain `yield return`, `yield break`, or `goto` out of the body.
- `ref`/`out` parameters of the enclosing method cannot be captured or referenced.

#### Constraints and Limitations

- No expression bodies (must use `{ }`).
- No attributes on the method.
- No generic type parameters on the anonymous method itself.
- Cannot access enclosing `ref`/`out` parameters.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Full Syntax with Explicit Types

```csharp
using System;

public class Program
{
    public static void Main()
    {
        Func<int, int, int> add = delegate (int x, int y) { return x + y; };
        Console.WriteLine(add(5, 6));   // 11
    }
}
```

**Expected Output**

```
11
```

**Why this output occurs**

- Explicit parameter types and names match `Func<int, int, int>`; the body returns the sum.

---

#### Example 2: Multi-Statement Body

```csharp
using System;

public class Program
{
    public static void Main()
    {
        Action<string> log = delegate (string msg)
        {
            var timestamp = DateTime.UtcNow.ToString("O");
            Console.WriteLine($"[{timestamp}] {msg}");
        };
        log("Started");
    }
}
```

**Expected Output (varies by time)**

```
[2024-XX-XXTXX:XX:XX.XXXXXXXZ] Started
```

**Why this output occurs**

- The anonymous method body contains multiple statements (local variable, log output) executed on each invocation.

---

#### Example 3: Nested and Capturing

```csharp
using System;

public class Program
{
    public static void Main()
    {
        int total = 0;
        Action<int> addToTotal = delegate (int amount)
        {
            total += amount;
            Console.WriteLine($"Total: {total}");
        };

        addToTotal(10);   // Total: 10
        addToTotal(5);    // Total: 15
    }
}
```

**Expected Output**

```
Total: 10
Total: 15
```

**Why this output occurs**

- The anonymous method captures `total` and mutates it across invocations, demonstrating closure semantics.

---

### Real-World Cases

- **Event subscriptions** — `button.Click += delegate(object s, EventArgs e) { ... };`.
- **Short callbacks** — Passing actions to APIs.
- **Legacy LINQ-like code** — Before lambdas.

---

### References

- Anonymous methods — Syntax - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/statements-expressions-operators/anonymous-methods

---

## 4. Anonymous Method Limitations

### Definitions

**Core Definition**
Anonymous method limitations are the **restrictions** that prevent anonymous methods from being used in certain contexts and from containing certain constructs.

**Technical Definition**
Anonymous methods have several syntactic and semantic restrictions. They **cannot**:
- Be converted to **expression trees** (`Expression<DelegateType>`) — only lambdas can.
- Access the enclosing method's **`ref` or `out` parameters**.
- Contain `goto`, `break`, or `continue` statements that jump **outside** the anonymous method body.
- Contain `yield return` or `yield break` (iterators must use lambdas or named methods; even lambdas cannot use `yield`, only named methods can).
- Have **attributes** applied.
- Have **generic type parameters** of their own.
- Be assigned to a variable of `dynamic` for dynamic invocation? (Actually they can be assigned to `dynamic` type since `dynamic` is `object` at runtime; the delegate instance can be invoked dynamically.)
- Be used as the **target** of a `using` or `lock` statement.

**Beginner-Friendly Explanation**
Anonymous methods are **great for simple inline callbacks**, but they cannot do everything lambdas can. In particular, they **cannot be used with expression trees** (used by ORMs and remote LINQ providers), and they have **scope restrictions** on `goto`/`break`/`continue`.

---

### Purposes

- To **define the boundaries** of anonymous method usability.
- To guide developers toward **lambdas** when expression trees are needed.
- To prevent confusing `goto`/`break`/`continue` semantics.
- To document **compiler restrictions** clearly.

---

### Syntax Rules and Structure (Limitation Matrix)

| Context | Allowed? | Reason |
|---|---|---|
| **Expression tree conversion** (`Expression<TDelegate>`) | ❌ | Anonymous methods cannot be expression-tree bodies. |
| **Enclosing `ref`/`out` parameter access** | ❌ | Closure capture of ref/out parameters is prohibited. |
| **`goto` out of the body** | ❌ | Cannot jump to a label outside the anonymous method. |
| **`break`/`continue` out of the body** | ❌ | Loop control cannot cross anonymous method boundaries. |
| **`yield return`/`yield break`** | ❌ | Iterators must be named methods (not lambdas either). |
| **Attributes** | ❌ | Anonymous methods have no declaration site for attributes. |
| **Generic type parameters** | ❌ | Anonymous methods cannot declare their own type parameters. |
| **`using`/`lock` as target** | ❌ | The anonymous method is an expression, not a variable. |

**Component breakdown**

- **Expression trees** — Only lambdas convert to `Expression<T>`.
- **Ref/out capture** — Prohibited to preserve reference semantics.
- **Out-of-body jumps** — Prevent confusing control flow.
- **Iterators** — Require named methods.
- **Attributes** — No declaration site.
- **Generics** — No syntax for type parameters.

#### Syntax Rules

- Anonymous methods can only contain statements that complete within the block.
- They cannot cross reference the enclosing method's `ref`/`out` parameters.

#### Constraints and Limitations

- Modern C# **prefers lambdas** for most scenarios, particularly where expression trees are needed (e.g., `IQueryable`, ORMs, remote LINQ).

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Compile Error — Expression Tree

```csharp
using System;
using System.Linq.Expressions;

public class Program
{
    public static void Main()
    {
        // COMPILE ERROR: An anonymous method cannot be converted to an expression tree
        // Expression<Func<int, int>> square = delegate (int x) { return x * x; };
    }
}
```

**Expected Output (conceptual)**

```
Compile-time error CS0831: An expression tree may not contain an anonymous method expression.
```

**Why this error occurs**

- Only lambdas convert to `Expression<T>`. Anonymous methods are not permitted in expression trees.

---

#### Example 2: Compile Error — `ref` Parameter Capture

```csharp
using System;

public class Program
{
    public static void Run(ref int value)
    {
        // COMPILE ERROR: Cannot capture a ref parameter
        // Action a = delegate { Console.WriteLine(value); };
    }
}
```

**Expected Output (conceptual)**

```
Compile-time error CS1628: Cannot use ref, out, or in parameter 'value' inside an anonymous method, lambda expression, query expression, or local function.
```

**Why this error occurs**

- Capturing `ref`/`out`/`in` parameters in a closure is prohibited because the closure could outlive the method's stack frame.

---

#### Example 3: Compile Error — `goto` Out of Body

```csharp
using System;

public class Program
{
    public static void Main()
    {
    label:
        Action a = delegate
        {
            // COMPILE ERROR: Cannot jump outside the body
            // goto label;
        };
        a();
    }
}
```

**Expected Output (conceptual)**

```
Compile-time error CS0159: No such label 'label' within the scope of the goto statement.
```

**Why this error occurs**

- `goto` cannot cross anonymous method boundaries.

---

### Real-World Cases

- **ORMs** — Expression trees require lambdas, not anonymous methods.
- **Iterators** — Named methods or lambdas (with `yield`) but never anonymous methods.
- **Closure safety** — Capturing `ref`/`out` parameters is disabled to protect stack lifetime.

---

### References

- Anonymous methods — Limitations - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/statements-expressions-operators/anonymous-methods
- Expression trees — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/concepts/expression-trees/

---

## 5. Anonymous Method Best Practices

### Definitions

**Core Definition**
Anonymous method best practices are the guidelines that help developers **use anonymous methods appropriately** while preferring lambdas in modern code where they are more concise, more flexible, and expression-tree-compatible.

**Technical Definition**
Best practices derive from Microsoft's guidance on anonymous functions and the natural migration from C# 2.0 anonymous methods to C# 3.0 lambdas. Key recommendations include: **prefer lambdas in new code** because they are more concise, expression-tree-capable, and idiomatic; **use anonymous methods only where the omitted-parameter-list form is desirable** or for maintaining legacy code; **avoid deeply nested anonymous methods**; **do not use anonymous methods in performance-critical code that constructs the delegate per call** unless the closure captures nothing and the delegate can be cached; **avoid capturing loop variables** in older C# versions (C# 5 fixed `foreach` capture, but `for` capture still requires care); and **document closure semantics** when an anonymous method mutates captured variables.

**Beginner-Friendly Explanation**
Best practices for anonymous methods are simple: **use lambdas in new code**, **use anonymous methods when they are clearer** (e.g., when you truly want to ignore all parameters), **watch out for closures capturing loop variables**, and **do not nest anonymous methods deeply**.

---

### Purposes

- To **choose the right syntax** between anonymous methods and lambdas.
- To **avoid closure pitfalls** in loops and shared state.
- To maintain **modern, idiomatic C# code**.
- To **preserve backward compatibility** while understanding trade-offs.
- To **align with tooling and analyzers**.

---

### Guidelines and Structure

#### General Guidelines

```
// Prefer lambdas in new code
Func<int, int> square = x => x * x;

// Use anonymous methods when you want to ignore all parameters
Action ignoreAll = delegate { /* ... */ };
// Lambda equivalent requires explicit empty parameter list: () => { ... }

// Avoid capturing loop variables (for pre-C# 5 foreach, and always for for)
// Bad: for (int i = 0; i < 3; i++) { callbacks.Add(delegate { Console.WriteLine(i); }); }
// Good: capture a copy: int copy = i; callbacks.Add(delegate { Console.WriteLine(copy); });

// Avoid deeply nested anonymous methods
// Document closure semantics when mutation is intentional
```

**Component breakdown**

- **Lambdas preferred** — More concise, expression-tree-capable.
- **`delegate { }` for ignoring all parameters** — Shorter than `() => { }`.
- **Loop capture awareness** — Capture a copy to avoid surprising results.
- **Avoid excessive nesting** — Readability first.
- **Document closure mutations** — Make intentional side effects clear.

#### Rules and Constraints

- **Do** use lambdas in new code unless anonymous methods are clearer.
- **Do** prefer the `delegate { }` form when ignoring all parameters.
- **Do** capture a copy of loop variables when needed.
- **Do** document closure semantics.
- **Do not** use anonymous methods in expression-tree contexts (use lambdas).
- **Do not** capture `ref`/`out`/`in` parameters (compiler enforces this).
- **Do not** nest anonymous methods deeply.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Lambda Preferred Over Anonymous Method

```csharp
using System;

public class Program
{
    public static void Main()
    {
        // Modern: lambda
        Func<int, int> square = x => x * x;
        Console.WriteLine(square(5));   // 25

        // Equivalent anonymous method (legacy style)
        Func<int, int> squareAnon = delegate (int x) { return x * x; };
        Console.WriteLine(squareAnon(5));   // 25
    }
}
```

**Expected Output**

```
25
25
```

**Why this output occurs**

- Both forms are functionally equivalent; lambdas are preferred for conciseness.

---

#### Example 2: Loop Variable Capture Pitfall

```csharp
using System;
using System.Collections.Generic;

public class Program
{
    public static void Main()
    {
        var callbacks = new List<Action>();

        // BAD: all callbacks capture the SAME variable i
        for (int i = 0; i < 3; i++)
        {
            callbacks.Add(delegate { Console.Write(i + " "); });
        }
        foreach (var cb in callbacks) cb();
        Console.WriteLine();   // 3 3 3

        // GOOD: capture a copy
        callbacks.Clear();
        for (int i = 0; i < 3; i++)
        {
            int copy = i;
            callbacks.Add(delegate { Console.Write(copy + " "); });
        }
        foreach (var cb in callbacks) cb();
        Console.WriteLine();   // 0 1 2
    }
}
```

**Expected Output**

```
3 3 3 
0 1 2 
```

**Why this output occurs**

- In the bad case, all closures share the same `i` variable, which has the final value `3`. In the good case, each iteration creates a fresh `copy` captured independently.

---

#### Example 3: Using `delegate { }` for Ignoring All Parameters

```csharp
using System;

public class Program
{
    public static void Main()
    {
        // Anonymous method: shortest form when ignoring everything
        EventHandler handler = delegate { Console.WriteLine("Fired"); };
        handler(null, EventArgs.Empty);   // Fired

        // Lambda equivalent requires empty parameter list
        EventHandler lambdaHandler = (_, _) => Console.WriteLine("Fired");
        lambdaHandler(null, EventArgs.Empty);   // Fired
    }
}
```

**Expected Output**

```
Fired
Fired
```

**Why this output occurs**

- The `delegate { }` form omits parameters entirely; the lambda form uses `(_, _)` discards. Both are valid; `delegate { }` is slightly shorter when every parameter is ignored.

---

### Real-World Cases

- **Modernising legacy code** — Convert `delegate { }` to lambdas where clearer.
- **Event subscriptions** — Prefer `(s, e) => { ... }` over `delegate (object s, EventArgs e) { ... }`.
- **Test code** — Use lambdas for mocks and stubs.
- **Reading older .NET 2.0 code** — Recognise and understand `delegate` inline blocks.

---

### References

- Anonymous methods — C# Programming Guide - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/statements-expressions-operators/anonymous-methods
- Lambdas vs. anonymous methods — Lambda expressions — https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/lambda-expressions
- Closure capture in loops — C# 5 breaking change - https://learn.microsoft.com/en-us/archive/blogs/ericlippert/closing-over-the-loop-variable-considered-harmful

---

## Deprecated, Unsafe, Implementation-Dependent, and Version-Specific Notes

| Feature | Status / Note |
|---|---|
| Anonymous methods | Introduced in **C# 2.0**; fully supported but **superseded** by lambdas in C# 3.0. |
| Lambda expressions | Introduced in **C# 3.0**; **preferred** in new code. |
| Expression tree conversion | **Not supported** for anonymous methods; lambdas only. |
| Parameter-list omission | **Supported** only for anonymous methods, not lambdas (lambdas require `()`). |
| `yield return` in anonymous methods | **Not allowed**. |
| `ref`/`out`/`in` capture | **Not allowed** (compiler error). |
| `goto` out of body | **Not allowed**. |
| Attributes on anonymous methods | **Not allowed**. |
| Loop variable capture (pre-C# 5) | **Changed** in C# 5 for `foreach`; `for` still requires explicit copy. |

---

## References (Consolidated)

- Anonymous methods — C# Programming Guide - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/statements-expressions-operators/anonymous-methods
- Delegates — C# Programming Guide - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/delegates/
- Lambda expressions — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/lambda-expressions
- Expression trees — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/concepts/expression-trees/
- Closure capture in loops — C# 5 breaking change (Eric Lippert) - https://learn.microsoft.com/en-us/archive/blogs/ericlippert/closing-over-the-loop-variable-considered-harmful
- delegate keyword — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/reference-types
- Introduction to delegates and events in C# — Microsoft Learn - https://learn.microsoft.com/en-us/dotnet/csharp/delegates-overview

---