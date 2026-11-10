# C# Local Functions — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
A local function is a **named method nested inside another member** (a method, constructor, property accessor, or another local function). It is visible only to its containing member and can be called only from within that member. Local functions let you keep helper logic **close to where it is used**, avoid polluting the class with private methods that serve a single call site, and enable recursion without the awkward ceremony required by lambda expressions.

**Technical Definition**
Local functions are methods of a type that are nested in another member. They can only be called from their containing member. Local functions can be declared in and called from methods, constructors, property accessors, event accessors, anonymous methods, lambda expressions, finalizers, and other local functions. Beginning with **C# 7.0**, local functions are supported; **C# 8.0** added the `static` modifier, which prevents a local function from capturing local variables or instance state. A local function declaration cannot include an access modifier (including `private`); doing so produces compiler error **CS0106**. Local functions can be `async`, `unsafe`, `static`, or `extern` (an `extern` local function must be `static`). Unlike lambda expressions, local functions are converted to delegates **only when used as a delegate**; if a local function is only ever called directly, the compiler can avoid the heap allocations associated with delegate and closure creation.

**Beginner-Friendly Explanation**
A local function is a **helper method that lives inside another method**. Instead of scattering small utility methods across your class, you define them right next to the code that uses them. They are **private by nature**—only the containing method can see or call them. Think of them as **nested recipes**: a large recipe (the containing method) can contain smaller recipes (local functions) that are only used within that larger recipe. This keeps related logic together, makes the code easier to read, and signals clearly that the helper is not meant to be used anywhere else. Since C# 8, you can mark a local function `static` to explicitly state that it does not depend on any surrounding variables.

---

### Key Characteristics

- **Nested, named helpers** — Local functions have names and live inside another member, unlike anonymous lambdas.
- **Containing-member-only access** — They are implicitly private and cannot be called from outside their containing member.
- **No access modifiers** — Declaring a local function as `private` (or any other access modifier) causes CS0106; they are private by definition.
- **Optional capture** — Non-static local functions can capture local variables and parameters of the enclosing member.
- **`static` local functions** — Since C# 8.0, a local function can be marked `static` to **prevent capture** of enclosing state, enabling allocation-free execution.
- **Delegate conversion only when needed** — Local functions become delegates only when used as delegates; direct calls avoid heap allocations.
- **Recursion-friendly** — Local functions can call themselves recursively without the “declare-and-assign-null-first” ceremony required by lambdas.
- **Iterator support** — Local functions can use `yield return` to produce sequences; lambda expressions cannot.

---

### Prerequisites

- Understanding of C# methods and method declarations.
- Familiarity with local variables and scope.
- Awareness of lambda expressions and delegates (for comparison).
- Knowledge of `async` and `yield return` (for advanced scenarios).
- Basic understanding of heap allocations and closures.

---

### Related Programming Areas

- **Method organisation** — Local functions reduce the number of private class-level helpers.
- **Functional programming** — Local functions and lambdas both support functional-style decomposition, but with different trade-offs.
- **Performance engineering** — Avoiding delegate and closure allocations in hot paths.
- **Recursive algorithms** — Local functions make recursion cleaner than lambdas.
- **Iterator and async patterns** — Local functions can be iterators and async methods, which lambdas cannot.

---

### Core Concepts / Features

1. Local Functions (Overarching Concept)
2. Local Function Declaration
3. Local Function Access
4. `static` Local Functions
5. Local Function vs. Lambda
6. Local Function Best Practices

Each concept follows a uniform structure: definitions, purposes, syntax rules and structure, annotated examples with expected outputs, real-world cases, and references.

---

## 1. Local Functions (Overarching Concept)

### Definitions

**Core Definition**
A local function is a named method nested inside another member, callable only from that containing member.

**Technical Definition**
Local functions are methods of a type that are nested in another member. They can only be called from their containing member. Local functions can be declared in and called from methods, constructors, property accessors, event accessors, anonymous methods, lambda expressions, finalizers, and other local functions. They make the intent of the code clear: anyone reading the code can see that the method is not callable except by the containing method. Local functions were introduced in **C# 7.0**.

**Beginner-Friendly Explanation**
A local function is a **helper that belongs to one method**. It is declared inside the method, used inside the method, and invisible to everything else.

---

### Purposes

- To keep helper logic **close to its caller**, improving locality and readability.
- To make it **impossible for other members** to call a helper that is only relevant to one method.
- To enable **recursion** without delegate ceremony.
- To support **iterator** and **async** helpers that lambdas cannot express.
- To allow the compiler to **avoid heap allocations** when the helper is not used as a delegate.

---

### Syntax Rules and Structure

```
<containing member>
{
    // local function declaration
    <modifiers> <return-type> <method-name> <parameter-list>
    {
        // body
    }
}
```

**Component breakdown**

- `modifiers` — Optional: `async`, `unsafe`, `static`, `extern`.
- `return-type` — The result type, or `void`.
- `method-name` — The local function’s identifier.
- `parameter-list` — The parameters.

#### Syntax Rules

- Local functions must be declared inside another member.
- No access modifiers (`private`, `public`, etc.) are allowed (CS0106).
- `extern` local functions must be `static`.
- Local functions cannot be declared inside an **expression-bodied member**.
- Parameters cannot be named `value` (causes ambiguity with compiler-generated capture variable).

#### Constraints and Limitations

- Cannot be called from outside the containing member.
- Cannot be `abstract`, `virtual`, or `override`.
- A local function that is never used produces warning CS8321.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: A Simple Local Function

```csharp
using System;

class Program
{
    static void Main()
    {
        string text = "Hello, World!";

        string AppendSeparator(string s) => s + "!";

        Console.WriteLine(AppendSeparator(text));   // Hello, World!!
    }
}
```

**Expected Output**

```
Hello, World!!
```

**Why this output occurs**

- `AppendSeparator` is a local function inside `Main`; it appends `!` and is called only from `Main`.

---

#### Example 2: Recursive Local Function

```csharp
using System;

class Program
{
    static void Main()
    {
        int Factorial(int n) => n <= 1 ? 1 : n * Factorial(n - 1);

        Console.WriteLine(Factorial(5));   // 120
    }
}
```

**Expected Output**

```
120
```

**Why this output occurs**

- `Factorial` calls itself recursively; no delegate variable or null-assignment ceremony is needed.

---

### Real-World Cases

- **Validation helpers** — Local functions that validate arguments before the main logic.
- **Recursive algorithms** — Tree traversal, factorial, Fibonacci.
- **Iterator helpers** — Local functions that produce `IEnumerable<T>` sequences.
- **Async helpers** — Local functions that perform asynchronous work.

---

### References

- Local functions — C# Programming Guide - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/local-functions
- Local functions — C# language specification - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/language-specification/statements#1354-local-functions

---

## 2. Local Function Declaration

### Definitions

**Core Definition**
Local function declaration is the syntax that introduces a named helper method inside another member.

**Technical Definition**
A local function is defined as a nested method inside a containing member. Its definition has the syntax `<modifiers> <return-type> <method-name> <parameter-list>`. The available modifiers are `async`, `unsafe`, `static`, and `extern`. A local function declaration cannot include an access modifier. The parameter list should not contain a parameter named `value`, because the compiler creates a temporary variable named `value` for captured outer variables, which can cause ambiguity.

**Beginner-Friendly Explanation**
Declaring a local function is **writing a mini-method inside a method**. You give it a name, a return type, and parameters, just like a normal method—but you cannot put `public` or `private` in front of it.

---

### Purposes

- To introduce a named helper inside a containing member.
- To specify the helper’s signature and body.
- To allow `async`, `unsafe`, `static`, or `extern` behaviour.
- To keep the helper’s scope limited to the containing member.

---

### Syntax Rules and Structure

```
<modifiers> <return-type> <method-name> <parameter-list>
{
    // body
}
```

**Component breakdown**

- `modifiers` — `async`, `unsafe`, `static`, `extern` (optional).
- `return-type` — The result type or `void`.
- `method-name` — The helper’s name.
- `parameter-list` — Parameters; avoid naming a parameter `value`.

#### Syntax Rules

- Must be inside a containing member.
- No access modifiers (CS0106).
- Cannot be inside an expression-bodied member.
- `extern` local functions must be `static`.

#### Constraints and Limitations

- Local function declarations are not visible outside the containing member.
- Unused local functions produce warning CS8321.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Declaration with `async`

```csharp
using System;
using System.Threading.Tasks;

class Program
{
    static async Task Main()
    {
        async Task<int> GetValueAsync()
        {
            await Task.Delay(100);
            return 42;
        }

        Console.WriteLine(await GetValueAsync());   // 42
    }
}
```

**Expected Output**

```
42
```

**Why this output occurs**

- The local function `GetValueAsync` is declared with `async` and called with `await`.

---

#### Example 2: Declaration with `static`

```csharp
using System;

class Program
{
    static void Main()
    {
        static int Square(int x) => x * x;

        Console.WriteLine(Square(5));   // 25
    }
}
```

**Expected Output**

```
25
```

**Why this output occurs**

- The `static` local function cannot capture enclosing state, so it is independent.

---

### Real-World Cases

- **Async helpers** — Local async functions for I/O-bound subtasks.
- **Unsafe helpers** — Local functions performing pointer operations.
- **Iterator helpers** — Local functions using `yield return`.

---

### References

- Local function syntax — Local functions — C# Programming Guide - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/local-functions#local-function-syntax
- Local function bodies — Resolve local function errors — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/compiler-messages/local-function-errors

---

## 3. Local Function Access

### Definitions

**Core Definition**
Local function access refers to the rules governing **who can call** a local function and **what a local function can access**.

**Technical Definition**
Local functions can only be called from their containing member. They are **implicitly private**; including an access modifier such as `private` generates compiler error CS0106. All local variables defined in the containing member, including its method parameters, are accessible in a non-static local function. Local functions can capture enclosing state unless marked `static`. A local function declaration can be referenced from any code location where it is in scope; unlike lambda expressions, it does not need to be assigned to a delegate before being called. This difference in definite assignment rules makes recursion easier with local functions. The compiler can also perform static analysis to determine that a local function definitely assigns captured variables.

**Beginner-Friendly Explanation**
Local functions are **private to their containing method**. Only that method can call them. They can see and use the method’s local variables, but they cannot be seen or called from anywhere else in the class. You do not need to declare them before calling them—you can call a local function that appears later in the method.

---

### Purposes

- To restrict helper visibility to a single member.
- To allow helpers to capture and use enclosing local state.
- To enable recursion without delegate initialization.
- To let the compiler reason about definite assignment of captured variables.
- To prevent accidental calls from other members.

---

### Syntax Rules and Structure (Access Rules)

| Aspect | Rule |
|---|---|
| **Callable from** | Only the containing member |
| **Access modifiers** | Not allowed (CS0106); implicitly private |
| **Captures enclosing locals** | Yes (non-static local functions) |
| **Callable before declaration** | Yes; scope is the whole containing block |
| **Delegate conversion** | Only when used as a delegate |

**Component breakdown**

- **Visibility** — Containing member only.
- **Capture** — Enclosing locals and parameters (unless `static`).
- **Declaration order** — Callable regardless of textual order.
- **Delegate conversion** — Lazy; avoids allocations when not used as delegate.

#### Syntax Rules

- No access modifiers.
- Non-static local functions can capture enclosing state.
- Local functions can be called before their declaration point.
- Capture may prevent garbage collection of captured variables.

#### Constraints and Limitations

- Cannot be called from outside the containing member.
- Capturing can extend the lifetime of local variables.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Capturing Enclosing State

```csharp
using System;

class Program
{
    static void Main()
    {
        int factor = 3;

        int Multiply(int x) => x * factor;   // captures 'factor'

        Console.WriteLine(Multiply(5));   // 15
    }
}
```

**Expected Output**

```
15
```

**Why this output occurs**

- `Multiply` is a non-static local function that captures `factor` from the enclosing method.

---

#### Example 2: Calling Before Declaration

```csharp
using System;

class Program
{
    static void Main()
    {
        Console.WriteLine(GetMessage());   // Hello

        string GetMessage() => "Hello";
    }
}
```

**Expected Output**

```
Hello
```

**Why this output occurs**

- The local function is in scope throughout the containing block; it can be called before its textual declaration.

---

### Real-World Cases

- **Closure-style helpers** — Local functions capturing configuration values.
- **Recursive descent parsers** — Local functions calling each other before declaration.
- **Definite-assignment patterns** — Using local functions to initialize out variables.

---

### References

- Local function access — Local functions — C# Programming Guide - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/local-functions#local-function-syntax
- CS0106: The modifier is not valid for this item — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/compiler-messages/cs0106

---

## 4. `static` Local Functions

### Definitions

**Core Definition**
A `static` local function is a local function marked with the `static` modifier (C# 8.0+). It **cannot capture** local variables or instance state from its enclosing member.

**Technical Definition**
A local function declared with the `static` modifier (available in C# 8.0 and later) cannot capture local variables or instance state. A static local function cannot contain a reference to an enclosing local variable (CS8421) or to `this` or `base` (CS8422). This restriction guarantees that the local function does not depend on the enclosing scope, which can allow the compiler to avoid allocating a closure. A static local function can still be `async` or `unsafe`. An `extern` local function must be `static`.

**Beginner-Friendly Explanation**
A `static` local function is a **self-contained helper**. It does not borrow anything from the surrounding method—no variables, no `this`. It is like a standalone mini-method that happens to be written inside another method. Because it does not capture anything, it is cheaper and clearer.

---

### Purposes

- To prevent unintended capture of enclosing state.
- To make a helper’s independence **explicit**.
- To enable allocation-free execution when no delegate conversion occurs.
- To avoid bugs caused by accidental closure over mutable variables.
- To satisfy analyzer rules that flag unnecessary captures.

---

### Syntax Rules and Structure

```
static   <return-type>   <method-name>(<parameters>)
{
    // cannot reference enclosing locals, this, or base
}
```

**Component breakdown**

- `static` — The modifier (C# 8.0+).
- No references to enclosing locals, `this`, or `base`.

#### Syntax Rules

- Must be marked `static`.
- Cannot reference enclosing local variables (CS8421).
- Cannot reference `this` or `base` (CS8422).
- Can be `async` or `unsafe`.
- `extern` local functions must be `static`.

#### Constraints and Limitations

- Cannot capture any enclosing state.
- Cannot access instance members of the containing type unless passed as parameters.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Static Local Function Without Capture

```csharp
using System;

class Program
{
    static void Main()
    {
        static int Square(int x) => x * x;

        Console.WriteLine(Square(6));   // 36
    }
}
```

**Expected Output**

```
36
```

**Why this output occurs**

- `Square` is `static`; it depends only on its parameter and captures nothing.

---

#### Example 2: Capture Error in Static Local Function

```csharp
using System;

class Program
{
    static void Main()
    {
        int factor = 2;

        // COMPILE ERROR CS8421: A static local function cannot contain a reference to 'factor'
        // static int Multiply(int x) => x * factor;
    }
}
```

**Expected Output (conceptual)**

```
Compile-time error CS8421
```

**Why this output occurs**

- The `static` local function attempts to reference `factor`, which is an enclosing local variable.

---

### Real-World Cases

- **Pure helpers** — Math or formatting helpers that do not need surrounding state.
- **Analyzer compliance** — Making independence explicit when a helper does not capture.
- **Performance-sensitive code** — Avoiding closure allocation when no capture is needed.

---

### References

- static local functions — Local functions — C# Programming Guide - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/local-functions#static-local-functions
- CS8421 and CS8422 — Resolve local function errors — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/compiler-messages/local-function-errors

---

## 5. Local Function vs. Lambda

### Definitions

**Core Definition**
Local functions and lambda expressions are both ways to define inline methods, but they differ in **naming**, **definite assignment**, **capture allocation**, **recursion**, **iterator support**, and **delegate conversion semantics**.

**Technical Definition**
At first glance, local functions and lambda expressions are very similar. In many cases, the choice between them is a matter of style and personal preference. However, there are real differences. Local functions have names; lambdas are anonymous methods assigned to `Func` or `Action` variables. Local functions have different rules for **definite assignment**: a local function declaration can be referenced from any code location where it is in scope, whereas a lambda expression must be assigned to a delegate variable before it can be accessed or called. Local functions are converted to delegates **only when used as a delegate**; if only called directly, they avoid the delegate allocation. For capturing variables, lambdas always use a class for the closure, while local functions **may use a struct** (passed by reference), which can avoid allocations. Local functions can be **recursive** without the “declare null, then assign” ceremony required by lambdas. Local functions can be **iterators** (`yield return`); lambdas cannot.

**Beginner-Friendly Explanation**
Both local functions and lambdas are **small functions written inside another method**. The difference is that a local function is a **named mini-method** you call directly, while a lambda is a **function value** you store in a variable. Local functions are better when you just want a helper; lambdas are better when you need to pass a function around as data. Local functions are also more efficient when you call them directly, because they do not necessarily create a delegate or closure object.

---

### Purposes

- To choose the right tool for inline helper logic.
- To understand the performance implications of capture and delegate allocation.
- To use recursion and iterators where lambdas cannot.
- To write clearer code with named helpers.
- To avoid unnecessary allocations in performance-sensitive code.

---

### Syntax Rules and Structure (Key Differences)

| Aspect | Local Function | Lambda Expression |
|---|---|---|
| **Name** | Has a name | Anonymous |
| **Definite assignment** | Callable anywhere in containing block | Must be assigned to a delegate before calling |
| **Delegate conversion** | Only when used as a delegate | Always converted to a delegate |
| **Capture allocation** | May use a `struct` (passed by ref) | Always uses a `class` |
| **Recursion** | Directly recursive | Requires declare-and-assign-null-first |
| **Iterator support** | Supports `yield return` | Does not support `yield return` |
| **Direct call allocation** | None (if not converted to delegate) | Delegate allocation (though may be cached) |

**Component breakdown**

- **Name** — Local functions are named; lambdas are anonymous.
- **Definite assignment** — Local functions can be called before their declaration; lambdas cannot.
- **Delegate conversion** — Local functions are lazy; lambdas are eager.
- **Capture** — Local functions can use struct closures; lambdas use class closures.
- **Recursion** — Local functions are naturally recursive; lambdas require initialization first.
- **Iterators** — Only local functions support `yield return`.

#### Syntax Rules

- Use a local function when the helper is **called directly** and needs a name.
- Use a lambda when the helper must be **passed as a delegate** or expression tree.
- Use a local function for **recursive** helpers.
- Use a local function for **iterator** helpers.

#### Constraints and Limitations

- Lambdas cannot use `yield return`.
- Lambdas always allocate a delegate when created (though the compiler may cache).
- Local functions may avoid allocations entirely when not converted to delegates.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Recursion — Local Function vs. Lambda

```csharp
using System;

class Program
{
    static void Main()
    {
        // Local function: direct recursion
        int FactorialLocal(int n) => n <= 1 ? 1 : n * FactorialLocal(n - 1);
        Console.WriteLine(FactorialLocal(5));   // 120

        // Lambda: requires declare-and-assign-null-first
        Func<int, int>? factorialLambda = null;
        factorialLambda = n => n <= 1 ? 1 : n * factorialLambda(n - 1);
        Console.WriteLine(factorialLambda(5));   // 120
    }
}
```

**Expected Output**

```
120
120
```

**Why this output occurs**

- The local function is directly recursive. The lambda must be assigned to `null` first and then reassigned to a body that references the delegate variable.

---

#### Example 2: Iterator — Only Local Functions Can Use `yield return`

```csharp
using System;
using System.Collections.Generic;

class Program
{
    static void Main()
    {
        IEnumerable<int> GetNumbers()
        {
            yield return 1;
            yield return 2;
            yield return 3;
        }

        foreach (var n in GetNumbers())
            Console.WriteLine(n);   // 1 2 3
    }
}
```

**Expected Output**

```
1
2
3
```

**Why this output occurs**

- The local function uses `yield return`; a lambda cannot express this.

---

### Real-World Cases

- **Recursive algorithms** — Use local functions for cleaner recursion.
- **Iterators** — Use local functions to produce sequences.
- **Delegates and LINQ** — Use lambdas when passing predicates or selectors.
- **Hot paths** — Use local functions to avoid delegate and closure allocations.

---

### References

- Local functions compared to lambda expressions — .NET docs (GitHub) - https://raw.githubusercontent.com/dotnet/docs/597cc4158103b3ad2bbddaf2da8bdb7abb92dccc/docs/csharp/local-functions-vs-lambdas.md
- Local functions vs. lambda expressions — C# Programming Guide - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/local-functions#local-functions-vs-lambda-expressions

---

## 6. Local Function Best Practices

### Definitions

**Core Definition**
Local function best practices are the guidelines that ensure local functions are used **appropriately, readably, and efficiently**.

**Technical Definition**
Best practices for local functions derive from Microsoft’s C# documentation and community guidance. Key recommendations include: **keep helper logic near its caller**; **use parameters or capture local state** appropriately; **use recursion when the problem repeats itself**; **choose between local functions and lambdas based on whether the helper is called directly or passed as a delegate**; **use a regular method instead of a local function when several members need the helper**; **mark local functions `static` when they do not capture enclosing state**; and **avoid declaring local functions that are never used** (warning CS8321).

**Beginner-Friendly Explanation**
Best practices are the habits that keep local functions clean: put them **near the code that uses them**, do not make them capture things they do not need, use `static` when possible, and move them to a regular method if more than one member needs them.

---

### Purposes

- To keep helpers local and focused.
- To avoid unnecessary captures and allocations.
- To choose the right construct (local function vs. lambda vs. method).
- To keep the containing method readable.
- To align with compiler warnings and analyzers.

---

### Guidelines and Structure

#### General Guidelines

```
// Keep helper logic near its caller
static void Process()
{
    // main logic
    ValidateInput();
    // ...

    // local function at the end
    void ValidateInput() { ... }
}

// Use parameters or capture local state appropriately
// Use static when no capture is needed
static int Square(int x) => x * x;

// Use a regular method when several members need the helper
// Avoid unused local functions (CS8321)
```

**Component breakdown**

- **Near the caller** — Declare local functions close to their usage, often at the end of the method.
- **Capture intentionally** — Capture only what is needed; use `static` to prevent accidental capture.
- **Choose the right tool** — Local function for direct calls and recursion; lambda for delegates.
- **Promote to method** — If multiple members need the helper, use a private method.

#### Rules and Constraints

- **Do** keep local functions near their call sites.
- **Do** mark local functions `static` when they do not capture enclosing state.
- **Do** use local functions for recursive and iterator helpers.
- **Do** use lambdas when the helper must be passed as a delegate.
- **Do not** declare unused local functions.
- **Do not** use local functions when a regular private method is more appropriate (multiple callers).

---

### Annotated Code Examples and Expected Outputs

#### Example: Well-Placed Static Local Function

```csharp
using System;

class Program
{
    static void Main()
    {
        string input = "  42  ";

        int value = ParseOrDefault(input, 0);
        Console.WriteLine(value);   // 42

        static int ParseOrDefault(string s, int fallback)
        {
            return int.TryParse(s.Trim(), out int result) ? result : fallback;
        }
    }
}
```

**Expected Output**

```
42
```

**Why this output occurs**

- The local function is `static` because it depends only on its parameters; it is placed at the end of `Main` near its usage.

---

### Real-World Cases

- **Validation helpers** — Local functions that check arguments.
- **Recursive descent** — Local functions for tree traversal.
- **Iterator helpers** — Local functions producing sequences.
- **Performance-sensitive code** — Static local functions avoiding capture.

---

### References

- Local functions — C# Programming Guide (best practices) - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/local-functions
- CS8321: The local function is declared but never used — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/compiler-messages/local-function-errors

---

## Deprecated, Unsafe, Implementation-Dependent, and Version-Specific Notes

| Feature | Status / Note |
|---------|---------------|
| Local functions | Introduced in **C# 7.0**; version-specific. |
| `static` local functions | Introduced in **C# 8.0**; version-specific. |
| Local functions inside expression-bodied members | **Not allowed**; compile-time error. |
| Access modifiers on local functions | **Not allowed**; CS0106. |
| Parameter named `value` | **Discouraged**; may cause ambiguity with compiler-generated capture variable. |
| Unused local functions | **Warning CS8321**; remove or call the function. |
| Capture allocation | **Implementation-dependent**; local functions may use a `struct` closure, lambdas always use a `class`. |
| Delegate conversion | **Lazy** for local functions; **eager** for lambdas. |
| Iterator support | **Local functions only**; lambdas cannot use `yield return`. |

---

## References (Consolidated)

- Local functions — C# Programming Guide - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/local-functions
- Local functions — C# language specification - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/language-specification/statements#1354-local-functions
- Local function syntax — Local functions — C# Programming Guide - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/local-functions#local-function-syntax
- static local functions — Local functions — C# Programming Guide - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/local-functions#static-local-functions
- Local functions vs. lambda expressions — C# Programming Guide - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/local-functions#local-functions-vs-lambda-expressions
- Local functions compared to lambda expressions — .NET docs (GitHub) - https://raw.githubusercontent.com/dotnet/docs/597cc4158103b3ad2bbddaf2da8bdb7abb92dccc/docs/csharp/local-functions-vs-lambdas.md
- Resolve errors and warnings related to local functions — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/compiler-messages/local-function-errors
- CS0106: The modifier is not valid for this item — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/compiler-messages/cs0106
- CS8321: The local function is declared but never used — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/compiler-messages/local-function-errors
- Essential C#: Local Functions - https://essentialcsharp.com/local-functions

---