# C# Expression Trees — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
An **expression tree** is an in-memory **tree-shaped data structure** that represents code. Every node in the tree is an **expression** — a constant, a variable, a method call, an operator, a conditional, a member access, or a lambda itself. Introduced in **C# 3.0** and the .NET Framework 3.5, expression trees allow code to be **inspected, transformed, and compiled at runtime**. They are the foundation of **LINQ providers** (such as Entity Framework Core) and enable scenarios like dynamic query building, rule engines, and serialization of logic.

**Technical Definition**
Expression trees represent code in a tree-like data structure, where each node is an expression. The `System.Linq.Expressions` namespace provides the abstract `Expression` class and its many subclasses (`BinaryExpression`, `ConstantExpression`, `ParameterExpression`, `LambdaExpression`, `MethodCallExpression`, `MemberExpression`, `NewExpression`, `UnaryExpression`, `ConditionalExpression`, `BlockExpression`, etc.). The generic `Expression<TDelegate>` class (derived from `LambdaExpression`) is the strongly typed entry point. An expression lambda (`x => x + 1`) assigned to `Expression<Func<int, int>>` is **parsed into a tree** at compile time. Trees are **immutable** — every modification requires building a new tree. You can **inspect** a tree via the node properties or the **`ExpressionVisitor`** pattern, **compile** a tree into an executable delegate via `Expression<TDelegate>.Compile()`, and **construct** trees programmatically using the `Expression` factory methods (`Expression.Parameter`, `Expression.Constant`, `Expression.Add`, `Expression.Call`, etc.). Since **.NET Core 3.0**, `Compile()` also offers a `preferInterpretation: true` option that uses an interpreter (faster to start, slower to run), while `CompileToMethod` is no longer supported.

**Beginner-Friendly Explanation**
An expression tree is **code represented as data**. Imagine taking the lambda `x => x * 2` and breaking it into a little family tree: a “multiply” node at the top, with two children — a “parameter” node (x) and a “constant” node (2). Once you have this tree, you can **read it, change it, analyse it, or run it**. This matters because it lets programs reason about code the way they would reason about a list or a dictionary. ORMs like Entity Framework use expression trees to turn C# lambdas into SQL — they walk the tree, figure out what you asked for, and generate the equivalent database query. Expression trees are the bridge between **statically typed C# code** and **runtime inspection and translation**.

---

### Key Characteristics

- **Code as data** — Lambdas become tree structures that can be inspected and manipulated.
- **Immutable trees** — Every node is immutable; modifications create new trees.
- **Compile-time conversion** — Only **expression lambdas** (not statement lambdas) convert to `Expression<T>`.
- **Rich node vocabulary** — Constants, parameters, binaries, calls, members, conditionals, blocks, loops, and more.
- **Visitor pattern** — `ExpressionVisitor` enables traversal and transformation.
- **Runtime compilation** — `Compile()` produces a delegate; `Compile(preferInterpretation: true)` uses an interpreter.
- **LINQ provider integration** — `IQueryable<T>` queries carry an `Expression` that providers translate.
- **Debugging challenges** — Trees are harder to debug than direct delegates; `ToString()` shows a readable representation.
- **Version-specific restrictions** — Some APIs (e.g., `CompileToMethod`) are unsupported on .NET Core / .NET 5+.

---

### Prerequisites

- Solid understanding of C# **delegates** (`Func<T>`, `Action<T>`).
- Familiarity with **lambda expressions** (expression lambdas and statement lambdas).
- Knowledge of LINQ (`IEnumerable<T>`, `IQueryable<T>`, method syntax).
- Awareness of reflection and metadata (types, `MethodInfo`, `PropertyInfo`).
- Basic understanding of the **visitor** design pattern.
- Familiarity with ORMs or remote query engines (helpful for the LINQ provider section).

---

### Related Programming Areas

- **LINQ providers** — `IQueryable<T>` queries are expression trees; providers translate them to SQL, OData, etc.
- **ORM frameworks** — Entity Framework Core, Dapper’s `SqlBuilder`, NHibernate.
- **Dynamic query building** — Programmatically composing filters and projections.
- **Rule engines** — Business rules expressed as expression trees and compiled on the fly.
- **Serialization of logic** — Sending or storing code as data.
- **Reflection** — Expression trees can call methods and access properties without direct invocation.

---

### Core Concepts / Features

1. Expression Trees (Overarching Concept)
2. `Expression<TDelegate>`
3. Expression Tree Construction
4. Expression Tree Inspection
5. Expression Tree Compilation
6. LINQ Providers
7. Expression Tree Best Practices

Each concept follows a uniform structure: definitions, purposes, syntax rules and structure, annotated code examples with expected outputs, real-world cases, and references.

---

## 1. Expression Trees (Overarching Concept)

### Definitions

**Core Definition**
An expression tree is a tree-shaped representation of code that can be inspected, transformed, and compiled at runtime.

**Technical Definition**
The `System.Linq.Expressions` namespace provides a set of classes that represent **expression trees** — data structures that describe code. The base class is `Expression`; derived classes include `LambdaExpression`, `BinaryExpression`, `ConstantExpression`, `ParameterExpression`, `MemberExpression`, `MethodCallExpression`, `NewExpression`, `UnaryExpression`, `BlockExpression`, `ConditionalExpression`, and many others. Expression trees are **immutable**; each node exposes properties describing its kind and children. The **`NodeType`** property returns an `ExpressionType` enum identifying the kind of node. Expression trees are used primarily by **LINQ providers** to translate C# lambdas into other languages.

**Beginner-Friendly Explanation**
An expression tree is **a little family tree of code**. Each node describes one operation (e.g., “add two things”), and its children describe the inputs. You can walk the tree, print it, modify it, or compile it back into executable code.

---

### Purposes

- To **represent C# code as data** for inspection and transformation.
- To enable **LINQ providers** to translate queries into other languages (SQL, OData, etc.).
- To support **dynamic query building** where queries are constructed at runtime.
- To implement **rule engines** and **DSLs** based on C# syntax.
- To allow **runtime compilation** of user-supplied logic.

---

### Syntax Rules and Structure

```
// Converting a lambda to an expression tree
Expression<Func<int, int>> expr = x => x + 1;

// Programmatic construction
var param = Expression.Parameter(typeof(int), "x");
var body = Expression.Add(param, Expression.Constant(1));
var lambda = Expression.Lambda<Func<int, int>>(body, param);
```

**Component breakdown**

- `Expression<Func<int, int>>` — The strongly typed expression tree.
- `x => x + 1` — Expression lambda parsed into the tree.
- `Expression.Parameter` — Creates a `ParameterExpression` node.
- `Expression.Add` — Creates a `BinaryExpression` node.
- `Expression.Lambda<...>` — Creates the root `Expression<TDelegate>`.

#### Syntax Rules

- Only **expression lambdas** convert to expression trees; statement lambdas (with `{ }`) do **not**.
- Expression trees are **immutable** — every modification yields a new tree.
- The root is typically `Expression<TDelegate>` (a `LambdaExpression`).
- The `Body` property returns the root expression; `Parameters` returns the parameter list.
- `NodeType` identifies the expression kind.

#### Constraints and Limitations

- Statement lambdas cannot be converted to expression trees.
- Not all C# constructs are representable (e.g., `yield return`, `await`, pointer operations).
- Some `Expression` APIs are restricted on .NET Core / .NET 5+ (e.g., `CompileToMethod`).

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Converting a Lambda to an Expression Tree

```csharp
using System;
using System.Linq.Expressions;

class Program
{
    static void Main()
    {
        Expression<Func<int, int>> expr = x => x + 1;

        Console.WriteLine($"NodeType: {expr.NodeType}");        // Lambda
        Console.WriteLine($"Body NodeType: {expr.Body.NodeType}"); // Add
        Console.WriteLine($"Parameter: {expr.Parameters[0].Name}"); // x
        Console.WriteLine($"Expression: {expr}");                // x => (x + 1)
    }
}
```

**Expected Output**

```
NodeType: Lambda
Body NodeType: Add
Parameter: x
Expression: x => (x + 1)
```

**Why this output occurs**

- The lambda `x => x + 1` is parsed into a tree with a `LambdaExpression` root, an `Add` body, and a parameter `x`. `ToString()` produces a readable representation.

---

#### Example 2: Same Lambda as Delegate vs. Expression Tree

```csharp
using System;
using System.Linq.Expressions;

class Program
{
    static void Main()
    {
        Func<int, int> del = x => x + 1;
        Expression<Func<int, int>> expr = x => x + 1;

        Console.WriteLine($"Delegate type: {del.GetType().Name}");
        Console.WriteLine($"Expression type: {expr.GetType().Name}");
    }
}
```

**Expected Output (approximate)**

```
Delegate type: Func`2
Expression type: Expression`1
```

**Why this output occurs**

- The **same lambda** is converted to a delegate (executable) when the target is `Func<int,int>`, and to an expression tree (data) when the target is `Expression<Func<int,int>>`.

---

### Real-World Cases

- **Entity Framework Core** — Translates lambdas to SQL.
- **LINQ to XML** — Queries XML using expression trees.
- **Dynamic LINQ** — Library for parsing string queries into expression trees.
- **Rule engines** — Business rules stored as expressions.

---

### References

- Expression Trees (C#) — Microsoft Learn - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/concepts/expression-trees/
- System.Linq.Expressions — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.linq.expressions
- Expression class — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.linq.expressions.expression

---

## 2. `Expression<TDelegate>`

### Definitions

**Core Definition**
`Expression<TDelegate>` is the **strongly typed** subclass of `LambdaExpression` that represents a lambda expression as a tree, where `TDelegate` is the delegate type the lambda would compile to.

**Technical Definition**
`System.Linq.Expressions.Expression<TDelegate>` is a sealed class derived from `LambdaExpression` and `Expression`. It represents a lambda expression as an expression tree. The `Body` property returns the root expression; `Parameters` returns the parameter list; `ReturnType` returns the delegate's return type; `Name` returns the lambda's name (if assigned); `TailCall` indicates whether tail-call optimization was requested. `Compile()` produces a `TDelegate` instance; `Compile(bool preferInterpretation)` chooses between compilation and interpretation.

**Beginner-Friendly Explanation**
`Expression<TDelegate>` is the **strongly typed root of an expression tree**. It says: “This tree corresponds to a lambda with this delegate signature.”

---

### Purposes

- To provide a **strongly typed** entry point for expression trees.
- To enable `Compile()` to return a `TDelegate`.
- To expose `Body` and `Parameters` for inspection.
- To bridge between lambdas and LINQ providers.

---

### Syntax Rules and Structure

```
Expression<TDelegate> expr = (parameters) => expression;
```

**Component breakdown**

- `TDelegate` — The delegate type, e.g., `Func<int, int>`.
- `Body` — The body expression (`Expression`).
- `Parameters` — A `ReadOnlyCollection<ParameterExpression>`.
- `ReturnType` — The delegate's return type.
- `Compile()` — Returns a `TDelegate`.
- `Compile(bool preferInterpretation)` — Interpreter option (fast startup, slower execution).

#### Syntax Rules

- Only **expression lambdas** convert to `Expression<TDelegate>`.
- `TDelegate` must be a delegate type.
- The lambda must have the same signature as `TDelegate`.
- The tree is **immutable**.

#### Constraints and Limitations

- Statement lambdas cannot convert.
- Not all C# constructs are expressible (e.g., `await`, `yield`).
- `CompileToMethod` is not supported on .NET Core / .NET 5+.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Inspecting `Expression<TDelegate>`

```csharp
using System;
using System.Linq.Expressions;

class Program
{
    static void Main()
    {
        Expression<Func<int, int, int>> add = (a, b) => a + b;

        Console.WriteLine($"Body: {add.Body}");                  // (a + b)
        Console.WriteLine($"Body NodeType: {add.Body.NodeType}"); // Add
        Console.WriteLine($"Parameter count: {add.Parameters.Count}"); // 2
        Console.WriteLine($"Return type: {add.ReturnType}");     // System.Int32
    }
}
```

**Expected Output**

```
Body: (a + b)
Body NodeType: Add
Parameter count: 2
Return type: System.Int32
```

**Why this output occurs**

- The root exposes its body and parameter list via the `LambdaExpression` base properties.

---

#### Example 2: Compile and Invoke

```csharp
using System;
using System.Linq.Expressions;

class Program
{
    static void Main()
    {
        Expression<Func<int, int, int>> add = (a, b) => a + b;
        Func<int, int, int> compiled = add.Compile();
        Console.WriteLine(compiled(3, 4));   // 7
    }
}
```

**Expected Output**

```
7
```

**Why this output occurs**

- `Compile()` produces a delegate that can be invoked like any other `Func<int,int,int>`.

---

### Real-World Cases

- **LINQ provider entry point** — `IQueryable<T>.Expression` is an `Expression`.
- **Compilation caching** — Store compiled delegates for repeated use.
- **Dynamic query composition** — Combine `Expression<Func<T, bool>>` predicates with `And`/`Or` helpers.

---

### References

- Expression<TDelegate> class — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.linq.expressions.expression-1
- LambdaExpression class — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.linq.expressions.lambdaexpression
- How to use expression trees to build dynamic queries — C# - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/concepts/expression-trees/how-to-use-expression-trees-to-build-dynamic-queries

---

## 3. Expression Tree Construction

### Definitions

**Core Definition**
Expression tree construction is the **programmatic building** of expression trees using the static factory methods on the `Expression` class (e.g., `Expression.Parameter`, `Expression.Add`, `Expression.Call`, `Expression.Lambda`).

**Technical Definition**
The `Expression` class exposes **factory methods** for every expression node type: `Expression.Parameter(Type, string)` for parameters, `Expression.Constant(object, Type)` for constants, `Expression.Add`, `Expression.Subtract`, `Expression.Multiply`, `Expression.Divide` for binaries, `Expression.Call` for method calls, `Expression.Property`, `Expression.Field`, `Expression.PropertyOrField` for member access, `Expression.New` for object creation, `Expression.Convert` for conversions, `Expression.Condition` for ternaries, `Expression.Block` for statement blocks, `Expression.Lambda`/`Expression.Lambda<T>` for lambdas. Each factory validates types and throws `ArgumentException` (or `InvalidOperationException`) on mismatches.

**Beginner-Friendly Explanation**
Constructing an expression tree is **building the tree by hand** using LEGO-like factory methods: `Expression.Parameter` for variables, `Expression.Constant` for literals, `Expression.Add` for addition, and `Expression.Lambda` for the root.

---

### Purposes

- To build queries **dynamically** at runtime.
- To implement **rule engines** and **DSLs**.
- To **compose** expressions programmatically (e.g., combining filters).
- To **generate** code from data (schema-driven logic).
- To **translate** external formats (JSON, XML) into executable code.

---

### Syntax Rules and Structure

```
// 1. Create parameters
var x = Expression.Parameter(typeof(int), "x");
var y = Expression.Parameter(typeof(int), "y");

// 2. Build the body
var body = Expression.Add(x, y);

// 3. Wrap in a lambda
var lambda = Expression.Lambda<Func<int, int, int>>(body, x, y);
```

**Component breakdown**

- `Expression.Parameter(type, name)` — `ParameterExpression`.
- `Expression.Constant(value, type?)` — `ConstantExpression`.
- `Expression.Add(left, right)` — `BinaryExpression`.
- `Expression.Call(method, args)` — `MethodCallExpression`.
- `Expression.Property(instance, propertyName)` — `MemberExpression`.
- `Expression.New(type, args)` — `NewExpression`.
- `Expression.Convert(expr, type)` — `UnaryExpression`.
- `Expression.Condition(test, ifTrue, ifFalse)` — `ConditionalExpression`.
- `Expression.Lambda<T>(body, params)` — `Expression<T>`.

#### Syntax Rules

- Factory methods validate operand types and throw on mismatch.
- Parameters must be declared before use.
- The lambda's delegate type must match the parameter and return types.
- Nested lambdas require `Expression.Quote` (for `IQueryable` providers).
- Trees are immutable; modifying requires re-creating the affected nodes.

#### Constraints and Limitations

- Not all C# constructs are constructible (e.g., `await`, `yield`, `lock`).
- Type mismatches produce `ArgumentException` at construction time.
- Building trees is **slower** than writing lambdas directly; cache when possible.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Programmatic Addition

```csharp
using System;
using System.Linq.Expressions;

class Program
{
    static void Main()
    {
        var x = Expression.Parameter(typeof(int), "x");
        var y = Expression.Parameter(typeof(int), "y");
        var body = Expression.Add(x, y);
        var lambda = Expression.Lambda<Func<int, int, int>>(body, x, y);

        var compiled = lambda.Compile();
        Console.WriteLine(compiled(3, 4));   // 7
        Console.WriteLine(lambda);             // (x, y) => (x + y)
    }
}
```

**Expected Output**

```
7
(x, y) => (x + y)
```

**Why this output occurs**

- The tree is built node by node and compiled; `ToString()` prints the reconstructed lambda.

---

#### Example 2: Building a Conditional

```csharp
using System;
using System.Linq.Expressions;

class Program
{
    static void Main()
    {
        var n = Expression.Parameter(typeof(int), "n");
        var condition = Expression.GreaterThan(n, Expression.Constant(0));
        var body = Expression.Condition(
            condition,
            Expression.Constant("positive"),
            Expression.Constant("non-positive"));

        var lambda = Expression.Lambda<Func<int, string>>(body, n);
        Console.WriteLine(lambda.Compile()(5));    // positive
        Console.WriteLine(lambda.Compile()(-1));   // non-positive
    }
}
```

**Expected Output**

```
positive
non-positive
```

**Why this output occurs**

- `Expression.Condition` builds a ternary node; compilation produces a delegate that evaluates it.

---

#### Example 3: Method Call

```csharp
using System;
using System.Linq.Expressions;

class Program
{
    static void Main()
    {
        var s = Expression.Parameter(typeof(string), "s");
        var toUpper = typeof(string).GetMethod(nameof(string.ToUpper), Type.EmptyTypes)!;
        var body = Expression.Call(s, toUpper);
        var lambda = Expression.Lambda<Func<string, string>>(body, s);

        Console.WriteLine(lambda.Compile()("hello"));   // HELLO
    }
}
```

**Expected Output**

```
HELLO
```

**Why this output occurs**

- `Expression.Call` builds a `MethodCallExpression` invoking `string.ToUpper`.

---

### Real-World Cases

- **Dynamic LINQ** — Building `Where` predicates from strings.
- **Specification composition** — Combining `Expression<Func<T, bool>>` filters.
- **Rule engines** — Evaluating user-defined rules.
- **Serialization** — Rebuilding expressions from stored representations.

---

### References

- How to use expression trees to build dynamic queries — C# - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/concepts/expression-trees/how-to-use-expression-trees-to-build-dynamic-queries
- How to execute expression trees — C# - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/concepts/expression-trees/how-to-execute-expression-trees
- Expression class — factory methods - https://learn.microsoft.com/en-us/dotnet/api/system.linq.expressions.expression

---

## 4. Expression Tree Inspection

### Definitions

**Core Definition**
Expression tree inspection is the process of **walking an expression tree** to examine its structure, node types, and children — typically using the `ExpressionVisitor` pattern or direct property access.

**Technical Definition**
An expression tree is traversed by reading its properties (`NodeType`, `Body`, `Parameters`, `Left`, `Right`, `Method`, `Member`, etc.) or by using `ExpressionVisitor`, an abstract class with a `Visit` method and per-node virtual methods (`VisitBinary`, `VisitConstant`, `VisitLambda`, `VisitMember`, `VisitMethodCall`, etc.). Visitors are used to **rewrite** trees (by returning replacement nodes) or to **analyse** them (by collecting information). `ToString()` provides a human-readable representation. `NodeType` returns the `ExpressionType` enum value.

**Beginner-Friendly Explanation**
Inspection is **reading the tree**. You can print it, ask “what kind of node is this?”, or write a visitor that walks every node and does something.

---

### Purposes

- To **analyse** what a query or lambda does.
- To **translate** expression trees to other languages (SQL, OData).
- To **rewrite** trees (optimisation, logging, customisation).
- To **validate** user-supplied expressions.
- To **explain** expressions to end users.

---

### Syntax Rules and Structure

```
// Direct inspection
var node = expr.Body;
Console.WriteLine(node.NodeType);

// Visitor pattern
public class MyVisitor : ExpressionVisitor
{
    protected override Expression VisitBinary(BinaryExpression node)
    {
        // inspect / rewrite
        return base.VisitBinary(node);
    }
}
```

**Component breakdown**

- `NodeType` — The kind of expression (`Add`, `Constant`, `Call`, etc.).
- `ExpressionVisitor.Visit(node)` — Dispatches to the right `Visit…` method.
- `VisitBinary`, `VisitConstant`, `VisitLambda`, `VisitMember`, `VisitMethodCall`, `VisitParameter`, `VisitNew`, `VisitUnary`, `VisitConditional`, `VisitBlock`, etc.

#### Syntax Rules

- `ExpressionVisitor` calls `Visit(node)` which dispatches based on runtime type.
- Subclasses override the relevant `Visit…` methods.
- Returning `node` (or a rewritten node) continues traversal; returning `null` short-circuits.
- `ExpressionVisitor` visits children automatically via `base.Visit…`.

#### Constraints and Limitations

- Custom expression types require custom visitor methods.
- Rewriting must respect type contracts; mismatched types throw.
- Visitors do not modify the original tree; they produce new nodes.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Direct Inspection

```csharp
using System;
using System.Linq.Expressions;

class Program
{
    static void Main()
    {
        Expression<Func<int, int>> expr = x => x * 2 + 1;

        Console.WriteLine($"Root: {expr.NodeType}");              // Lambda
        Console.WriteLine($"Body: {expr.Body.NodeType}");          // Add
        var add = (BinaryExpression)expr.Body;
        Console.WriteLine($"Left: {add.Left.NodeType}");           // Multiply
        Console.WriteLine($"Right: {add.Right.NodeType}");         // Constant
    }
}
```

**Expected Output**

```
Root: Lambda
Body: Add
Left: Multiply
Right: Constant
```

**Why this output occurs**

- The expression `x * 2 + 1` has an `Add` body, with a `Multiply` left child and a `Constant` right child.

---

#### Example 2: `ExpressionVisitor`

```csharp
using System;
using System.Linq.Expressions;

public class PrintVisitor : ExpressionVisitor
{
    protected override Expression VisitBinary(BinaryExpression node)
    {
        Console.WriteLine($"Binary: {node.NodeType}");
        return base.VisitBinary(node);
    }

    protected override Expression VisitConstant(ConstantExpression node)
    {
        Console.WriteLine($"Constant: {node.Value}");
        return base.VisitConstant(node);
    }
}

class Program
{
    static void Main()
    {
        Expression<Func<int, int>> expr = x => x * 2 + 1;
        new PrintVisitor().Visit(expr);
    }
}
```

**Expected Output**

```
Binary: Add
Binary: Multiply
Constant: 2
Constant: 1
```

**Why this output occurs**

- The visitor recursively walks the tree, calling `VisitBinary` for each binary node and `VisitConstant` for constants.

---

### Real-World Cases

- **Query translation** — ORMs walk expression trees to emit SQL.
- **Optimisation** — Visitors can simplify or rewrite expressions.
- **Logging** — Record query shapes for diagnostics.
- **Validation** — Reject expressions containing unsupported constructs.

---

### References

- ExpressionVisitor class — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.linq.expressions.expressionvisitor
- How to modify expression trees — C# - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/concepts/expression-trees/how-to-modify-expression-trees

---

## 5. Expression Tree Compilation

### Definitions

**Core Definition**
Expression tree compilation is the process of converting an expression tree into an **executable delegate** via `Expression<TDelegate>.Compile()`.

**Technical Definition**
`Expression<TDelegate>.Compile()` returns a `TDelegate` that, when invoked, executes the tree's logic. Since **.NET Core 3.0**, `Compile(bool preferInterpretation)` allows selecting an **interpreter** (faster to start, slower to run) or **JIT compilation** (slower to start, faster to run). The compiled delegate is a normal delegate; it can be stored, invoked, combined, and garbage-collected. Compilation is relatively expensive; cached compiled delegates are recommended for repeated use. `CompileToMethod` was removed in .NET Core / .NET 5+.

**Beginner-Friendly Explanation**
Compilation is **turning the tree back into runnable code**. `Compile()` gives you a delegate you can call as if it were a normal method.

---

### Purposes

- To **execute** expression trees.
- To **cache** compiled logic for repeated invocation.
- To choose between **interpreter** (fast startup) and **JIT** (fast execution).
- To **dynamically create** delegates from data.
- To implement **rule engines** and **dynamic dispatch**.

---

### Syntax Rules and Structure

```
TDelegate compiled = expr.Compile();
TDelegate interpreted = expr.Compile(preferInterpretation: true);

// Invocation
var result = compiled(arg1, arg2);
```

**Component breakdown**

- `Compile()` — JIT-compiles the tree to a delegate.
- `Compile(true)` — Uses the interpreter (faster startup, slower per-call).
- `Compile(false)` — JIT-compiles (default).
- The returned delegate matches `TDelegate`.

#### Syntax Rules

- `Compile()` is available on `Expression<TDelegate>` and `LambdaExpression`.
- The interpreter overload is available on .NET Core 3.0+.
- Compiled delegates are thread-safe once compiled.
- Compilation cost is paid once; cache the delegate for reuse.

#### Constraints and Limitations

- Compilation is expensive; not suitable for one-off invocations.
- `CompileToMethod` is not supported on .NET Core / .NET 5+.
- Some expressions (e.g., `await`) cannot be compiled.
- Interpreter mode has lower per-call performance but faster startup.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Compilation and Invocation

```csharp
using System;
using System.Linq.Expressions;

class Program
{
    static void Main()
    {
        Expression<Func<int, int, int>> add = (a, b) => a + b;
        Func<int, int, int> compiled = add.Compile();
        Console.WriteLine(compiled(3, 4));   // 7
    }
}
```

**Expected Output**

```
7
```

**Why this output occurs**

- `Compile()` produces a `Func<int,int,int>`; invoking it runs the tree logic.

---

#### Example 2: Interpreter vs. Compiled

```csharp
using System;
using System.Diagnostics;
using System.Linq.Expressions;

class Program
{
    static void Main()
    {
        Expression<Func<int, int>> square = x => x * x;

        var interpreted = square.Compile(preferInterpretation: true);
        var compiled = square.Compile();

        Console.WriteLine(interpreted(5));   // 25
        Console.WriteLine(compiled(5));      // 25

        const int Iterations = 1_000_000;
        var sw = Stopwatch.StartNew();
        for (int i = 0; i < Iterations; i++) interpreted(i);
        sw.Stop();
        Console.WriteLine($"Interpreted: {sw.ElapsedMilliseconds} ms");

        sw.Restart();
        for (int i = 0; i < Iterations; i++) compiled(i);
        sw.Stop();
        Console.WriteLine($"Compiled: {sw.ElapsedMilliseconds} ms");
    }
}
```

**Expected Output (approximate)**

```
25
25
Interpreted: 30 ms
Compiled: 5 ms
```

**Why this output occurs**

- JIT-compiled delegates run faster per invocation; interpreted trees start faster but are slower per call.

---

#### Example 3: Caching Compiled Delegates

```csharp
using System;
using System.Collections.Concurrent;
using System.Linq.Expressions;

public static class CompiledCache
{
    private static readonly ConcurrentDictionary<string, Delegate> _cache = new();

    public static TDelegate Get<TDelegate>(string key, Expression<TDelegate> expr)
        where TDelegate : Delegate
        => (TDelegate)_cache.GetOrAdd(key, _ => expr.Compile());
}

class Program
{
    static void Main()
    {
        var add = Expression.Lambda<Func<int, int, int>>(
            Expression.Add(
                Expression.Parameter(typeof(int), "a"),
                Expression.Parameter(typeof(int), "b")),
            Expression.Parameter(typeof(int), "a"),
            Expression.Parameter(typeof(int), "b"));

        var compiled = CompiledCache.Get("add", add);
        Console.WriteLine(compiled(3, 4));   // 7
    }
}
```

**Expected Output**

```
7
```

**Why this output occurs**

- The compiled delegate is cached by key; repeated calls reuse the same delegate.

---

### Real-World Cases

- **Rule engines** — Compile rules once, evaluate many times.
- **Dynamic LINQ** — Compile generated expressions to delegates.
- **Serialization of logic** — Load expressions from storage and compile.
- **DSLs** — Build and compile domain-specific expressions.

---

### References

- How to execute expression trees — C# - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/concepts/expression-trees/how-to-execute-expression-trees
- Expression<TDelegate>.Compile — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.linq.expressions.expression-1.compile

---

## 6. LINQ Providers

### Definitions

**Core Definition**
A **LINQ provider** is a library that implements the `IQueryable<T>` and `IQueryProvider` interfaces to translate expression trees into a target language (e.g., SQL, OData, Elasticsearch DSL), execute the translated query, and return results.

**Technical Definition**
`IQueryable<T>` extends `IEnumerable<T>` and exposes an `Expression` property (the query's expression tree) and a `Provider` property (the `IQueryProvider`). The `Queryable` extension methods (`Where`, `Select`, `OrderBy`, etc.) **compose expression trees** rather than executing them, returning a new `IQueryable<T>`. When the query is enumerated (e.g., by `foreach` or `ToList`), the provider's `Execute` method translates the expression tree into the target language and executes it. Examples include Entity Framework Core (SQL), LINQ to XML, LINQ to Objects (which uses `Enumerable` and delegates, not expression trees), and MongoDB LINQ provider.

**Beginner-Friendly Explanation**
A LINQ provider is the **translator** that turns your C# lambda into something else. Entity Framework turns `db.Customers.Where(c => c.Age > 30)` into `SELECT * FROM Customers WHERE Age > 30`. The provider receives the query as an **expression tree**, inspects it, and writes the target-language code.

---

### Purposes

- To **translate C# queries** into other languages (SQL, OData, etc.).
- To **execute queries remotely** (databases, web services).
- To **expose data sources** as queryable sequences.
- To **optimise queries** at translation time.
- To provide **type-safe query APIs** over external systems.

---

### Syntax Rules and Structure

```
// IQueryable<T> carries an expression tree
IQueryable<Customer> query = dbContext.Customers
    .Where(c => c.Age > 30)
    .OrderBy(c => c.Name);

Expression expression = query.Expression;
IQueryProvider provider = query.Provider;

// Execution triggers translation
List<Customer> results = query.ToList();
```

**Component breakdown**

- `IQueryable<T>` — A query that is not yet executed.
- `Expression` — The query's expression tree.
- `Provider` — The `IQueryProvider` that translates and executes.
- `ToList()`, `foreach`, `Count()` — Trigger execution.
- `Queryable` — The static class providing expression-tree-composing operators.

#### Syntax Rules

- `IQueryable<T>` queries **compose**; they do not execute until enumerated.
- The provider receives the entire expression tree at execution time.
- Translation must handle all nodes the query uses; unsupported nodes throw.
- Providers often cache translated queries (e.g., EF Core's query cache).

#### Constraints and Limitations

- Not all C# constructs are translatable; providers throw `NotSupportedException`.
- Client-side evaluation (mixed execution) should be used cautiously.
- Expression trees captured by `IQueryable` may retain references (lifetime considerations).
- Performance depends on translation quality and query execution.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: `IQueryable<T>` vs. `IEnumerable<T>`

```csharp
using System;
using System.Collections.Generic;
using System.Linq;

class Program
{
    static void Main()
    {
        var data = new List<int> { 1, 2, 3, 4, 5 };

        // IEnumerable<T> — delegates, executed in memory
        var enumerableQuery = data.Where(n => n > 2);
        Console.WriteLine($"Enumerable: {enumerableQuery.GetType().Name}");

        // IQueryable<T> — expression trees (simulated with AsQueryable)
        var queryable = data.AsQueryable();
        var queryableQuery = queryable.Where(n => n > 2);
        Console.WriteLine($"Queryable: {queryableQuery.GetType().Name}");
        Console.WriteLine($"Expression: {queryableQuery.Expression}");
    }
}
```

**Expected Output (approximate)**

```
Enumerable: WhereListIterator`1
Queryable: EnumerableQuery`1
Expression: value(System.Collections.Generic.List`1[System.Int32]).Where(n => (n > 2))
```

**Why this output occurs**

- `Where` on `IEnumerable<T>` returns an iterator (delegate-based); `Where` on `IQueryable<T>` returns an expression-tree-carrying query.

---

#### Example 2: Building a Queryable Query

```csharp
using System;
using System.Collections.Generic;
using System.Linq;

class Program
{
    static void Main()
    {
        var data = new List<string> { "apple", "banana", "cherry" }.AsQueryable();

        var query = data
            .Where(s => s.Length > 5)
            .OrderBy(s => s);

        // Query not yet executed; expression tree is composed
        Console.WriteLine($"Expression: {query.Expression}");

        // Execution triggers enumeration
        foreach (var s in query)
            Console.WriteLine(s);
    }
}
```

**Expected Output (approximate)**

```
Expression: value(System.Collections.Generic.List`1[System.String]).Where(s => (s.Length > 5)).OrderBy(s => s)
cherry
banana
```

**Why this output occurs**

- The query is composed as an expression tree; enumeration triggers execution.

---

#### Example 3: Custom `IQueryProvider` (Skeleton)

```csharp
using System;
using System.Collections;
using System.Linq;
using System.Linq.Expressions;

public class PrintProvider : IQueryProvider
{
    public IQueryable CreateQuery(Expression expression)
        => new PrintQueryable(this, expression);
    public IQueryable<T> CreateQuery<T>(Expression expression)
        => new PrintQueryable<T>(this, expression);
    public object? Execute(Expression expression) => null;
    public TResult Execute<TResult>(Expression expression) => default!;
}

public class PrintQueryable<T> : IQueryable<T>
{
    private readonly IQueryProvider _provider;
    public PrintQueryable(IQueryProvider provider, Expression expression)
    {
        _provider = provider;
        Expression = expression;
    }
    public Type ElementType => typeof(T);
    public Expression Expression { get; }
    public IQueryProvider Provider => _provider;
    public IEnumerator<T> GetEnumerator()
    {
        Console.WriteLine($"Executing: {Expression}");
        return Enumerable.Empty<T>().GetEnumerator();
    }
    IEnumerator IEnumerable.GetEnumerator() => GetEnumerator();
}
```

**Expected Output (conceptual)**

```
// A custom provider can translate and execute expression trees however it chooses.
```

**Why this output occurs**

- A custom `IQueryProvider` receives the composed expression tree and decides how to execute it.

---

### Real-World Cases

- **Entity Framework Core** — SQL translation for relational databases.
- **MongoDB LINQ** — Translates to MongoDB query documents.
- **LINQ to XML** — Queries XML documents via expression trees.
- **Cosmos DB LINQ** — Translates to Cosmos SQL.
- **Dynamic LINQ** — Parses string queries into `IQueryable` expression trees.

---

### References

- IQueryable<T> — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.linq.iqueryable-1
- IQueryProvider — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.linq.iqueryprovider
- LINQ Providers — Microsoft Learn - https://learn.microsoft.com/en-us/dotnet/standard/linq/
- Walkthrough: Creating an IQueryable LINQ Provider — Microsoft Learn - https://learn.microsoft.com/en-us/dotnet/standard/linq/walkthrough-creating-iqueryable-linq-provider

---

## 7. Expression Tree Best Practices

### Definitions

**Core Definition**
Expression tree best practices are the guidelines that ensure expression trees are **correct, performant, maintainable, and safe**, especially in provider-driven contexts.

**Technical Definition**
Best practices derive from the .NET expression tree documentation and provider design. Key recommendations include: **prefer expression lambdas over statement lambdas** when the target requires an expression tree; **cache compiled delegates** rather than recompiling; **use `preferInterpretation: true`** for one-shot or fast-startup scenarios; **prefer `ExpressionVisitor` over manual traversal**; **avoid mutating captured state inside visitors** unless intentional; **do not over-compile** (compilation is expensive); **understand provider limitations** when composing queries; **keep expressions translatable** (avoid methods providers cannot translate); **document unsupported constructs**; and **choose between `IEnumerable` and `IQueryable`** based on whether translation is needed.

**Beginner-Friendly Explanation**
Best practices for expression trees are: **use them where they are needed**, **cache compiled delegates**, **use the visitor pattern for inspection**, and **keep queries translatable**.

---

### Purposes

- To **maximise performance** (cache, choose interpreter vs. JIT).
- To **ensure provider compatibility** (translatable expressions).
- To **prevent memory leaks** from captured references.
- To **maintain readability** in dynamic query code.
- To **choose between delegates and expression trees** correctly.

---

### Guidelines and Structure

#### General Guidelines

```
// 1. Prefer expression lambdas when the target is Expression<T>
Expression<Func<int, int>> expr = x => x * 2;   // OK
// Expression<Func<int, int>> bad = x => { return x * 2; }; // CS0834

// 2. Cache compiled delegates
private static readonly Func<int, int> _square =
    ((Expression<Func<int, int>>)(x => x * x)).Compile();

// 3. Use preferInterpretation for one-shot scenarios
var fastStart = expr.Compile(preferInterpretation: true);

// 4. Use ExpressionVisitor for traversal and rewriting

// 5. Keep queries translatable (avoid methods providers cannot translate)
var query = db.Customers.Where(c => c.Age > 30);   // translatable
// var bad = db.Customers.Where(c => MyLocalMethod(c)); // often not translatable

// 6. Choose IEnumerable vs. IQueryable deliberately
IEnumerable<int> inMemory = list.Where(n => n > 0);       // delegates
IQueryable<int> remote = queryable.Where(n => n > 0);      // expression trees
```

**Component breakdown**

- **Expression lambdas** — Required for `Expression<T>` targets.
- **Caching** — Compile once, reuse many times.
- **Interpreter mode** — For one-shot or fast-startup scenarios.
- **Visitor pattern** — Standard traversal mechanism.
- **Translatability** — Keep provider-translatable operations in `IQueryable` queries.
- **IEnumerable vs. IQueryable** — Match the execution model to the data source.

#### Rules and Constraints

- **Do** use expression lambdas for `Expression<T>` targets.
- **Do** cache compiled delegates.
- **Do** use `ExpressionVisitor` for traversal.
- **Do** keep `IQueryable` queries translatable.
- **Do** prefer `preferInterpretation: true` for one-off compilations.
- **Do not** compile the same expression repeatedly in loops.
- **Do not** mix client-side and server-side evaluation carelessly in `IQueryable`.
- **Do not** capture large objects unnecessarily in expression trees.

---

### Annotated Code Examples and Expected Outputs

#### Example: Cached Compiled Delegate

```csharp
using System;
using System.Linq.Expressions;

public static class Predicates
{
    // Cached compiled predicate
    public static readonly Func<int, bool> IsPositive =
        ((Expression<Func<int, bool>>)(x => x > 0)).Compile();
}

class Program
{
    static void Main()
    {
        int[] values = { -2, -1, 0, 1, 2 };
        foreach (var v in values)
            Console.WriteLine($"{v}: {Predicates.IsPositive(v)}");
    }
}
```

**Expected Output**

```
-2: False
-1: False
0: False
1: True
2: True
```

**Why this output occurs**

- The expression is compiled **once** into a static delegate; the loop reuses it with no repeated compilation cost.

---

### Real-World Cases

- **ORM queries** — Keep queries translatable.
- **Rule engines** — Cache compiled rules.
- **Dynamic queries** — Use visitors to compose filters.
- **Performance-critical code** — Prefer compiled delegates over interpreted trees.

---

### References

- Expression Trees (C#) — Microsoft Learn - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/concepts/expression-trees/
- How to execute expression trees — C# - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/concepts/expression-trees/how-to-execute-expression-trees
- Walkthrough: Creating an IQueryable LINQ Provider — Microsoft Learn - https://learn.microsoft.com/en-us/dotnet/standard/linq/walkthrough-creating-iqueryable-linq-provider

---

## Deprecated, Unsafe, Implementation-Dependent, and Version-Specific Notes

| Feature | Status / Note |
|---|---|
| Expression trees | Introduced in **C# 3.0 / .NET Framework 3.5**; stable. |
| `Expression<TDelegate>.Compile()` | Core API; JIT-compiles the tree to a delegate. |
| `Compile(preferInterpretation: true)` | Introduced in **.NET Core 3.0**; uses an interpreter. |
| `CompileToMethod` | **Removed** in .NET Core / .NET 5+ (throws `PlatformNotSupportedException`). |
| Statement lambda → expression tree | **Not allowed** (CS0834). |
| `await` inside expression trees | **Not supported**. |
| `yield return` inside expression trees | **Not supported**. |
| Provider-specific nodes | **Implementation-dependent**; providers may not translate all nodes. |
| Debugging expression trees | **Challenging**; use `ToString()` and visitors. |
| Memory retention via captured variables | **Possible**; captured objects remain alive as long as the tree. |

---

## References (Consolidated)

- Expression Trees (C#) — Microsoft Learn - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/concepts/expression-trees/
- System.Linq.Expressions — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.linq.expressions
- Expression class — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.linq.expressions.expression
- Expression<TDelegate> class — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.linq.expressions.expression-1
- LambdaExpression class — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.linq.expressions.lambdaexpression
- ExpressionVisitor class — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.linq.expressions.expressionvisitor
- Expression<TDelegate>.Compile — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.linq.expressions.expression-1.compile
- How to use expression trees to build dynamic queries — C# - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/concepts/expression-trees/how-to-use-expression-trees-to-build-dynamic-queries
- How to execute expression trees — C# - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/concepts/expression-trees/how-to-execute-expression-trees
- How to modify expression trees — C# - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/concepts/expression-trees/how-to-modify-expression-trees
- IQueryable<T> — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.linq.iqueryable-1
- IQueryProvider — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.linq.iqueryprovider
- Walkthrough: Creating an IQueryable LINQ Provider — Microsoft Learn - https://learn.microsoft.com/en-us/dotnet/standard/linq/walkthrough-creating-iqueryable-linq-provider
- LINQ Providers — Microsoft Learn - https://learn.microsoft.com/en-us/dotnet/standard/linq/

---