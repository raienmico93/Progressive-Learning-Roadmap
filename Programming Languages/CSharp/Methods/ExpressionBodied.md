# C# Expression-Bodied Members — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
Expression-bodied members are a C# language feature that lets you implement a member (method, property, constructor, destructor, operator, or accessor) with a **single expression** following the `=>` token, instead of a traditional brace-enclosed block. Introduced in **C# 6.0** for methods and getter-only properties, and expanded in **C# 7.0** to constructors, destructors, property accessors, indexers, and operators, expression-bodied members reduce boilerplate while preserving type safety and readability. The `=>` symbol in this context is a **directive to the compiler** to generate the equivalent block body behind the scenes—it is not a lambda expression and does not create a delegate.

**Technical Definition**
An expression body definition provides a member's implementation in a concise form and has the general syntax `member => expression;`, where `expression` is a valid C# expression. For methods, the expression must return a value whose type matches the method's return type, or, for `void` methods, perform an operation. For constructors, the expression is typically a single assignment expression or a method call that initializes instance state. For finalizers (historically called destructors), the expression typically contains cleanup statements such as releasing unmanaged resources. For operators, the expression follows the operator declaration and provides the operation's result. Expression-bodied members are **compiled into the same IL** as their block-bodied equivalents; they are syntactic sugar that the compiler expands into a getter, setter, or method body.

**Beginner-Friendly Explanation**
An expression-bodied member is a **shorthand way to write a member that does only one thing**. Instead of writing a method with `{ return x * 2; }`, you write `=> x * 2;`. It is like the difference between writing a full sentence (“The area is calculated by multiplying width by height”) and a formula (“Area => width * height”). The `=>` symbol means “this member is defined as this expression.” It makes simple members **shorter and easier to read** without changing what the code does.

---

### Key Characteristics

- **Single-expression requirement** — The body must be a **single expression**; multiple statements require a block body.
- **Introduced incrementally** — C# 6.0 added expression bodies for methods and getter-only properties; C# 7.0 expanded to constructors, destructors, property accessors, indexers, and operators.
- **Compiler-generated equivalence** — The compiler converts the expression body into the equivalent block body (a getter, a method body, etc.).
- **Not a lambda** — The `=>` in an expression-bodied member is **not** a lambda operator; it is a member-implementation directive.
- **IDE-enforceable** — Style rules such as **IDE0021–IDE0026** allow teams to prefer expression bodies for constructors, methods, properties, indexers, and operators.
- **Works with `void`** — Even `void` methods can use expression bodies when the expression performs an operation (e.g., a method call).

---

### Prerequisites

- Understanding of C# methods, properties, constructors, and operators.
- Familiarity with lambda expressions and the `=>` token in lambdas.
- Knowledge of the difference between expressions and statements.
- Awareness of `void` return and `return` statements.
- Basic understanding of operator overloading and finalizers.

---

### Related Programming Areas

- **Code style and tooling** — Analyzers and `.editorconfig` rules enforce expression-bodied preferences.
- **Functional programming** — Expression-oriented style aligns with functional idioms.
- **API design** — Concise members improve readability of small, focused types.
- **Performance** — Expression-bodied members compile to the same IL as block bodies; no runtime penalty.
- **Records and primary constructors** — Modern C# codebases frequently combine expression bodies with records and primary constructors.

---

### Core Concepts / Features

1. Expression-Bodied Methods
2. Expression-Bodied Properties (Read-Only and Accessors)
3. Expression-Bodied Constructors
4. Expression-Bodied Destructors (Finalizers)
5. Expression-Bodied Operators
6. The `=>` Syntax
7. Expression-Bodied Best Practices

Each concept follows a uniform structure: definitions, purposes, syntax rules and structure, annotated code examples with expected outputs, real-world cases, and references.

---

## 1. Expression-Bodied Methods

### Definitions

**Core Definition**
An expression-bodied method is a method whose entire implementation is a **single expression** following the `=>` token, without a `return` keyword for non-void methods.

**Technical Definition**
An expression-bodied method consists of a single expression that returns a value whose type matches the method's return type, or, for methods that return `void`, performs some operation. The `return` keyword is **not used** in the expression body; the expression's value is implicitly returned. For `void` methods, the expression must be a statement expression (e.g., a method call or assignment).

**Beginner-Friendly Explanation**
Instead of writing `{ return a + b; }`, you write `=> a + b;`. The expression is the return value. If the method returns `void`, the expression is an action (like `Console.WriteLine(...)`).

---

### Purposes

- To reduce boilerplate for simple, single-expression methods.
- To improve readability of small helper methods.
- To align method syntax with lambda expressions.
- To enable concise `ToString` overrides and computed helpers.
- To support IDE-enforced style rules (IDE0022).

---

### Syntax Rules and Structure

```
[accessModifier] [modifiers] returnType MethodName(parameters) => expression;
```

**Component breakdown**

- `accessModifier` — Optional; default `private`.
- `modifiers` — `static`, `virtual`, `override`, `async`, etc.
- `returnType` — The method's return type, or `void`.
- `MethodName` — The method's identifier.
- `parameters` — The parameter list.
- `=>` — The expression-body separator.
- `expression` — The single expression providing the implementation.

#### Syntax Rules

- The expression must be implicitly convertible to the return type (for non-`void` methods).
- For `void` methods, the expression must be a statement expression.
- The `return` keyword is **not** used in the expression body.
- The body must be a **single expression**; multiple statements require a block body.

#### Constraints and Limitations

- Cannot contain multiple statements, local variable declarations, or control-flow statements (other than expressions like ternary).
- `async` methods can use expression bodies if the expression is `await`-able.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Basic Expression-Bodied Method

```csharp
using System;

class Calculator
{
    // Block body
    public int AddBlock(int a, int b)
    {
        return a + b;
    }

    // Expression body (equivalent)
    public int AddExpression(int a, int b) => a + b;
}

class Program
{
    static void Main()
    {
        var calc = new Calculator();
        Console.WriteLine(calc.AddBlock(2, 3));       // 5
        Console.WriteLine(calc.AddExpression(2, 3));  // 5
    }
}
```

**Expected Output**

```
5
5
```

**Why this output occurs**

- `AddExpression` uses `=>` to return `a + b` directly; the compiler generates the same IL as `AddBlock`.

---

#### Example 2: `void` Expression-Bodied Method

```csharp
using System;

class Program
{
    static void Log(string message) => Console.WriteLine($"[LOG] {message}");

    static void Main()
    {
        Log("Started");   // [LOG] Started
    }
}
```

**Expected Output**

```
[LOG] Started
```

**Why this output occurs**

- The `void` method uses an expression body to perform a method call; no `return` is needed.

---

#### Example 3: `ToString` Override

```csharp
using System;

public class Person
{
    public string FirstName { get; }
    public string LastName { get; }

    public Person(string firstName, string lastName)
    {
        FirstName = firstName;
        LastName = lastName;
    }

    public override string ToString() => $"{FirstName} {LastName}".Trim();
}

class Program
{
    static void Main()
    {
        var p = new Person("Mandy", "Dejesus");
        Console.WriteLine(p);   // Mandy Dejesus
    }
}
```

**Expected Output**

```
Mandy Dejesus
```

**Why this output occurs**

- The `ToString` override uses an expression body to return the formatted name.

---

### Real-World Cases

- **`ToString` overrides** — Concise string representations.
- **Computed helpers** — `Area()`, `Distance()`, `IsValid()`.
- **Fluent APIs** — Methods that return `this` or a computed value.
- **Extension methods** — Simple transformations and predicates.

---

### References

- Expression-bodied members — C# Programming Guide - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/statements-expressions-operators/expression-bodied-members
- IDE0022: Use expression body for methods - .NET - https://learn.microsoft.com/en-us/dotnet/fundamentals/code-analysis/style-rules/ide0022

---

## 2. Expression-Bodied Properties (Read-Only and Accessors)

### Definitions

**Core Definition**
An expression-bodied property implements a **getter-only property** or an individual **`get` or `set` accessor** using a single expression following `=>`.

**Technical Definition**
A **read-only property** can be implemented with the syntax `PropertyType PropertyName => expression;`, which is equivalent to a getter that returns the expression's value. Property **`get` and `set` accessors** can also be expression-bodied: `public string Name { get => locationName; set => locationName = value; }`. Expression-bodied accessors are supported in C# 6.0 for getters and C# 7.0 for setters and indexers.

**Beginner-Friendly Explanation**
A read-only property `public int Value => 42;` means “when you ask for `Value`, you get `42`.” An expression-bodied getter or setter is a shorthand for a single-statement accessor.

---

### Purposes

- To implement computed read-only properties concisely.
- To simplify property accessors that perform a single operation.
- To reduce boilerplate in value objects and DTOs.
- To align property syntax with expression-oriented style.

---

### Syntax Rules and Structure

```
// Read-only property
PropertyType PropertyName => expression;

// Expression-bodied accessors
public PropertyType PropertyName
{
    get => expression;
    set => statementExpression;
}
```

**Component breakdown**

- `PropertyType` — The property's type.
- `PropertyName` — The property's identifier.
- `=>` — The expression-body separator.
- `expression` — The getter's return expression.
- `statementExpression` — The setter's statement expression (e.g., assignment).

#### Syntax Rules

- Read-only expression-bodied properties are implicitly getter-only.
- Getter expression bodies must return a value convertible to the property type.
- Setter expression bodies must be statement expressions (e.g., assignments).
- Accessors can be mixed: one expression-bodied, one block-bodied.

#### Constraints and Limitations

- A read-only expression-bodied property cannot have a setter.
- Computed properties are evaluated on each access (no caching).

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Read-Only Computed Property

```csharp
using System;

public class Rectangle
{
    public double Width { get; }
    public double Height { get; }

    public Rectangle(double width, double height)
    {
        Width = width;
        Height = height;
    }

    public double Area => Width * Height;
}

class Program
{
    static void Main()
    {
        var rect = new Rectangle(3, 4);
        Console.WriteLine(rect.Area);   // 12
    }
}
```

**Expected Output**

```
12
```

**Why this output occurs**

- `Area` is an expression-bodied read-only property that computes `Width * Height` on each access.

---

#### Example 2: Expression-Bodied Getter and Setter

```csharp
using System;

public class Location
{
    private string _name;

    public Location(string name) => _name = name;

    public string Name
    {
        get => _name;
        set => _name = value;
    }
}

class Program
{
    static void Main()
    {
        var loc = new Location("London");
        Console.WriteLine(loc.Name);   // London
        loc.Name = "Paris";
        Console.WriteLine(loc.Name);   // Paris
    }
}
```

**Expected Output**

```
London
Paris
```

**Why this output occurs**

- Both the `get` and `set` accessors use expression bodies to read and write `_name`.

---

### Real-World Cases

- **Computed properties** — `FullName => $"{First} {Last}"`.
- **Value objects** — `IsEmpty => Count == 0`.
- **DTOs** — Simple properties backed by fields.

---

### References

- Read-only properties — Expression-bodied members - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/statements-expressions-operators/expression-bodied-members#read-only-properties
- Properties (accessors) — Expression-bodied members - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/statements-expressions-operators/expression-bodied-members#properties
- IDE0025: Use expression body for properties - .NET - https://learn.microsoft.com/en-us/dotnet/fundamentals/code-analysis/style-rules/ide0025

---

## 3. Expression-Bodied Constructors

### Definitions

**Core Definition**
An expression-bodied constructor implements a constructor using a **single assignment expression** or a **method call** following `=>`, typically initializing a field or property.

**Technical Definition**
An expression body definition for a constructor typically consists of a **single assignment expression** or a method call that handles the constructor's arguments or initializes instance state. Expression-bodied constructors were introduced in **C# 7.0**. For constructors that need to assign **multiple values**, a tuple deconstruction can be used: `public Person(string name, int age) => (Name, Age) = (name, age);`.

**Beginner-Friendly Explanation**
Instead of writing `{ _name = name; }`, you write `=> _name = name;`. If you need to set two fields, you can use a tuple: `=> (Name, Age) = (name, age);`.

---

### Purposes

- To reduce boilerplate for simple constructors that only assign arguments to fields.
- To align constructor syntax with expression-oriented style.
- To support concise initialization in records and small classes.
- To enable IDE-enforced style rules (IDE0021).

---

### Syntax Rules and Structure

```
public ClassName(parameters) => assignmentExpression;
public ClassName(parameters) => (Field1, Field2) = (arg1, arg2);
```

**Component breakdown**

- `ClassName` — The class name.
- `parameters` — The constructor's parameters.
- `=>` — The expression-body separator.
- `assignmentExpression` — A single assignment expression.
- Tuple form — Multiple assignments via deconstruction.

#### Syntax Rules

- The expression must be a statement expression (assignment or method call).
- A single assignment initializes one member.
- Multiple assignments require a tuple deconstruction or a helper method call.
- The constructor body cannot contain multiple statements directly.

#### Constraints and Limitations

- Complex initialization logic requires a traditional block body.
- Tuple deconstruction syntax requires C# 7.0+.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Single Assignment Constructor

```csharp
using System;

public class Server
{
    public string Name;

    public Server(string name) => Name = name;
}

class Program
{
    static void Main()
    {
        var server = new Server("Domain Controller");
        Console.WriteLine(server.Name);   // Domain Controller
    }
}
```

**Expected Output**

```
Domain Controller
```

**Why this output occurs**

- The constructor uses an expression body to assign `name` to `Name`.

---

#### Example 2: Multiple Assignments via Tuple Deconstruction

```csharp
using System;

public class Person
{
    public string Name { get; }
    public int Age { get; }

    public Person(string name, int age) => (Name, Age) = (name, age);
}

class Program
{
    static void Main()
    {
        var p = new Person("Alice", 30);
        Console.WriteLine($"{p.Name}, {p.Age}");   // Alice, 30
    }
}
```

**Expected Output**

```
Alice, 30
```

**Why this output occurs**

- The tuple deconstruction assigns both `Name` and `Age` in a single expression.

---

### Real-World Cases

- **DTOs and records** — Simple constructors that copy parameters to properties.
- **Value objects** — Constructors that validate and assign.
- **Immutable types** — Constructors that initialize `readonly` fields.

---

### References

- Constructors — Expression-bodied members (C# Programming Guide) - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/statements-expressions-operators/expression-bodied-members
- IDE0021: Use expression body for constructors - .NET - https://learn.microsoft.com/en-us/dotnet/fundamentals/code-analysis/style-rules/ide0021
- C# 7 Expression Bodied Constructors — Stack Overflow - https://stackoverflow.com/questions/41996465/c-sharp-7-expression-bodied-constructors

---

## 4. Expression-Bodied Destructors (Finalizers)

### Definitions

**Core Definition**
An expression-bodied destructor (finalizer) implements the finalizer using a **single expression** following `=>`, typically a cleanup statement or method call.

**Technical Definition**
A finalizer (historically called a **destructor**) can be implemented as an expression body definition. The expression body typically contains cleanup statements, such as releasing unmanaged resources or logging. Expression-bodied finalizers were introduced in **C# 7.0**.

**Beginner-Friendly Explanation**
Instead of writing `~Server() { Console.WriteLine("Cleaning up"); }`, you write `~Server() => Console.WriteLine("Cleaning up");`. The expression is the finalizer's action.

---

### Purposes

- To reduce boilerplate in finalizers that perform a single cleanup action.
- To align finalizer syntax with other expression-bodied members.
- To support concise logging or resource-release calls in finalizers.
- To enable IDE-enforced style rules.

---

### Syntax Rules and Structure

```
~ClassName() => expression;
```

**Component breakdown**

- `~ClassName` — The finalizer syntax.
- `()` — The finalizer takes no parameters.
- `=>` — The expression-body separator.
- `expression` — A statement expression (method call or assignment).

#### Syntax Rules

- The expression must be a statement expression (e.g., method call).
- The finalizer cannot return a value.
- The expression body is expanded to the equivalent block body by the compiler.

#### Constraints and Limitations

- Finalizers are non-deterministic; cleanup timing is controlled by the garbage collector.
- Complex cleanup logic should use `IDisposable` and `Dispose` instead of relying solely on finalizers.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Basic Expression-Bodied Finalizer

```csharp
using System;

public class Server
{
    public string Name;

    public Server(string name) => Name = name;

    ~Server() => Console.WriteLine("The end is near...");
}

class Program
{
    static void Main()
    {
        var server = new Server("Domain Controller");
        Console.WriteLine($"The name of the server is: {server.Name}");
    }
}
```

**Expected Output (eventually)**

```
The name of the server is: Domain Controller
The end is near...
```

**Why this output occurs**

- The finalizer uses an expression body to print a cleanup message when the object is garbage-collected.

---

#### Example 2: Expression-Bodied Finalizer Calling `Dispose`

```csharp
using System;

public class ResourceHolder : IDisposable
{
    private bool _disposed;

    ~ResourceHolder() => Dispose(false);

    public void Dispose()
    {
        Dispose(true);
        GC.SuppressFinalize(this);
    }

    protected virtual void Dispose(bool disposing)
    {
        if (!_disposed)
        {
            if (disposing)
            {
                // Free managed resources
            }
            // Free unmanaged resources
            _disposed = true;
        }
    }
}
```

**Expected Output (conceptual)**

```
// Finalizer calls Dispose(false) as a safety net
```

**Why this output occurs**

- The expression-bodied finalizer delegates to the standard `Dispose(bool)` pattern.

---

### Real-World Cases

- **Resource wrappers** — Finalizers that release unmanaged handles.
- **Logging** — Finalizers that log object lifetime for diagnostics.
- **Safety nets** — Finalizers that call `Dispose(false)` when callers forget to dispose.

---

### References

- Finalizers — Expression-bodied members - C# Programming Guide - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/statements-expressions-operators/expression-bodied-members
- Finalizers — C# Programming Guide - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/finalizers
- IDE0021: Use expression body for constructors (covers finalizers) - .NET - https://learn.microsoft.com/en-us/dotnet/fundamentals/code-analysis/style-rules/ide0021

---

## 5. Expression-Bodied Operators

### Definitions

**Core Definition**
An expression-bodied operator implements an overloaded operator using a **single expression** following `=>`, instead of a block body with a `return` statement.

**Technical Definition**
Operators, including conversion operators, can be implemented with an expression body: `public static T operator +(T left, T right) => expression;`. Expression-bodied operators were introduced in **C# 7.0**. The expression must produce a value compatible with the operator's return type.

**Beginner-Friendly Explanation**
Instead of writing `{ return new ComplexNumber(...); }`, you write `=> new ComplexNumber(...);`. The expression is the operator's result.

---

### Purposes

- To reduce boilerplate in operator overloads that perform a single computation.
- To improve readability of mathematical and conversion operators.
- To align operator syntax with expression-oriented style.
- To enable IDE-enforced style rules (IDE0023/IDE0024).

---

### Syntax Rules and Structure

```
public static ReturnType operator OperatorSymbol(parameters) => expression;
public static implicit operator TargetType(SourceType value) => expression;
public static explicit operator TargetType(SourceType value) => expression;
```

**Component breakdown**

- `operator` — The operator keyword.
- `OperatorSymbol` — The operator being overloaded (e.g., `+`, `==`).
- `parameters` — The operator's parameters.
- `=>` — The expression-body separator.
- `expression` — The operator's result.

#### Syntax Rules

- The expression must be convertible to the operator's return type.
- Conversion operators use `implicit` or `explicit` with `operator`.
- The expression body is expanded to a block body by the compiler.

#### Constraints and Limitations

- Complex operator logic requires a block body.
- Operator declarations must follow C# operator overloading rules.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Expression-Bodied Arithmetic Operator

```csharp
using System;

public readonly struct ComplexNumber
{
    public double Real { get; }
    public double Imaginary { get; }

    public ComplexNumber(double real, double imaginary)
    {
        Real = real;
        Imaginary = imaginary;
    }

    public static ComplexNumber operator +(ComplexNumber c1, ComplexNumber c2) =>
        new ComplexNumber(c1.Real + c2.Real, c1.Imaginary + c2.Imaginary);

    public override string ToString() => $"{Real} + {Imaginary}i";
}

class Program
{
    static void Main()
    {
        var a = new ComplexNumber(1, 2);
        var b = new ComplexNumber(3, 4);
        Console.WriteLine(a + b);   // 4 + 6i
    }
}
```

**Expected Output**

```
4 + 6i
```

**Why this output occurs**

- The `+` operator uses an expression body to construct a new `ComplexNumber` with summed components.

---

#### Example 2: Expression-Bodied Conversion Operator

```csharp
using System;

public readonly struct Digit
{
    private readonly byte _value;

    public Digit(byte value)
    {
        if (value > 9) throw new ArgumentOutOfRangeException();
        _value = value;
    }

    public static implicit operator byte(Digit d) => d._value;
    public static explicit operator Digit(byte b) => new Digit(b);
}

class Program
{
    static void Main()
    {
        var d = new Digit(7);
        byte number = d;          // implicit
        Console.WriteLine(number); // 7

        var digit = (Digit)number; // explicit
        Console.WriteLine(digit);  // 7
    }
}
```

**Expected Output**

```
7
7
```

**Why this output occurs**

- Both conversion operators use expression bodies to convert between `Digit` and `byte`.

---

### Real-World Cases

- **Mathematical types** — `Complex`, `Vector`, `Matrix`.
- **Value wrappers** — `Money`, `Temperature`, `Distance`.
- **Conversion operators** — Wrapping and unwrapping primitive values.

---

### References

- Operators — Expression-bodied members - C# Programming Guide - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/statements-expressions-operators/expression-bodied-members
- IDE0023 and IDE0024: Use expression body for operators - .NET - https://learn.microsoft.com/en-us/dotnet/fundamentals/code-analysis/style-rules/ide0023-ide0024
- Operator overloading — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/operator-overloading

---

## 6. The `=>` Syntax (General Rules)

### Definitions

**Core Definition**
The `=>` token in expression-bodied members is a **member-implementation separator** that tells the compiler “this member is implemented by the following expression.” It is syntactically similar to the lambda operator but semantically distinct.

**Technical Definition**
An expression body definition has the general syntax `member => expression;`, where `expression` is a valid expression. The `=>` token is **not overloadable** and cannot be used as a lambda operator in contexts where an expression-bodied member is expected. The compiler expands the expression body into the equivalent block body, a getter, or an operator implementation.

**Beginner-Friendly Explanation**
The `=>` symbol means **“is defined as.”** In `public int Area => Width * Height;`, it means “Area is defined as Width times Height.” In a lambda, `x => x * 2` means “x goes to x times 2.” The context tells you which meaning applies.

---

### Purposes

- To provide a concise, readable syntax for single-expression members.
- To unify the syntax of methods, properties, constructors, and operators.
- To enable IDE style rules and code fixes.
- To reduce boilerplate without sacrificing type safety.

---

### Syntax Rules and Structure

```
member => expression;
```

**Component breakdown**

- `member` — The member declaration (method, property, constructor, etc.).
- `=>` — The expression-body separator.
- `expression` — The single expression implementing the member.

#### Syntax Rules

- The expression must be valid for the member's context.
- The `return` keyword is omitted in expression bodies.
- The `=>` token cannot be overloaded.

#### Constraints and Limitations

- Only one expression; multiple statements require a block body.
- The expression must be a valid C# expression (no statements, local declarations, or control-flow statements).

---

### Annotated Code Examples and Expected Outputs

#### Example 1: The Same `=>` in Different Contexts

```csharp
using System;

class Program
{
    // Expression-bodied property
    public static int Answer => 42;

    // Expression-bodied method
    public static int Double(int x) => x * 2;

    // Expression-bodied constructor
    public Program() => Console.WriteLine("Constructed");

    static void Main()
    {
        Console.WriteLine(Answer);       // 42
        Console.WriteLine(Double(5));    // 10
        var p = new Program();            // Constructed
    }
}
```

**Expected Output**

```
42
10
Constructed
```

**Why this output occurs**

- The same `=>` token implements a property, a method, and a constructor, each with a single expression.

---

### Real-World Cases

- **All expression-bodied members** — Methods, properties, constructors, finalizers, operators, accessors.
- **Records and primary constructors** — Concise initialization and computed members.

---

### References

- Expression body definition syntax — Expression-bodied members - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/statements-expressions-operators/expression-bodied-members
- `=>` operator — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/lambda-operator

---

## 7. Expression-Bodied Best Practices

### Definitions

**Core Definition**
Expression-bodied member best practices are the guidelines that ensure expression bodies are used **appropriately, readably, and consistently** across a codebase.

**Technical Definition**
Best practices for expression-bodied members derive from .NET code style rules (IDE0021–IDE0026) and community conventions. Key recommendations include: **use expression bodies for simple members** whose logic is a single expression; **prefer block bodies for complex logic** that requires multiple statements, local variables, or control flow; **keep expression bodies short and readable**; **use expression bodies consistently** for getter-only properties and simple methods; and **enable analyzer rules** to enforce team preferences.

**Beginner-Friendly Explanation**
Use expression bodies when the member does **one simple thing**. If the logic needs multiple steps, use a normal block body. Be consistent—if one simple method uses `=>`, make other simple methods use it too.

---

### Purposes

- To improve readability by choosing the right body style.
- To maintain consistency across a codebase.
- To avoid cramming complex logic into a single expression.
- To leverage tooling for automatic style enforcement.
- To align with modern C# conventions.

---

### Guidelines and Structure

#### General Guidelines

```
// Use expression bodies for simple members
public double Area => Width * Height;
public string FullName => $"{First} {Last}";
public void Log(string message) => Console.WriteLine(message);

// Prefer block bodies for complex logic
public int Compute(int x)
{
    int y = x * 2;
    return y + 1;
}

// Configure analyzer rules
// .editorconfig
// csharp_style_expression_bodied_methods = true
// csharp_style_expression_bodied_properties = true
// csharp_style_expression_bodied_constructors = true
// csharp_style_expression_bodied_operators = true
```

**Component breakdown**

- **Simple members** — Single-expression logic is ideal for `=>`.
- **Complex logic** — Multiple statements, local variables, or control flow require `{ }`.
- **Analyzer configuration** — `.editorconfig` enforces team preferences.

#### Rules and Constraints

- **Do** use expression bodies for getter-only properties and simple methods.
- **Do** keep expression bodies short (ideally a single line).
- **Do** configure IDE rules (IDE0021–IDE0026) to enforce consistency.
- **Do not** use expression bodies for members with multiple statements or complex control flow.
- **Do not** sacrifice readability for brevity.

---

### Annotated Code Examples and Expected Outputs

#### Example: Appropriate Use of Expression Bodies

```csharp
using System;

public class Circle
{
    public double Radius { get; }

    public Circle(double radius) => Radius = radius;

    public double Area => Math.PI * Radius * Radius;

    public double Circumference => 2 * Math.PI * Radius;

    public override string ToString() => $"Circle(r={Radius})";
}

class Program
{
    static void Main()
    {
        var c = new Circle(2);
        Console.WriteLine(c.Area.ToString("F2"));    // 12.57
        Console.WriteLine(c);                         // Circle(r=2)
    }
}
```

**Expected Output**

```
12.57
Circle(r=2)
```

**Why this output occurs**

- All members have simple single-expression logic, so expression bodies are appropriate and readable.

---

### Real-World Cases

- **Records and DTOs** — Computed properties and `ToString` overrides.
- **Value objects** — Simple constructors and conversion operators.
- **Fluent APIs** — Simple methods that return `this` or a computed value.

---

### References

- C# code style — Expression-bodied members - https://github.com/cratis/applicationmodel/blob/0eac455579644b25fac8967e7cfc505c58d3aa1b/.cratis/ai/skills/cratis-engineering-csharp-conventions/references/code-style.md
- Code Style — Prefer expression-bodied members - https://github.com/brightercommand/brighter/blob/eab8e9de3efe4762c7fb5a11172d4ac40cda3900/.agent_instructions/code_style.md
- IDE0021: Use expression body for constructors - .NET - https://learn.microsoft.com/en-us/dotnet/fundamentals/code-analysis/style-rules/ide0021
- IDE0022: Use expression body for methods - .NET - https://learn.microsoft.com/en-us/dotnet/fundamentals/code-analysis/style-rules/ide0022
- IDE0025: Use expression body for properties - .NET - https://learn.microsoft.com/en-us/dotnet/fundamentals/code-analysis/style-rules/ide0025
- IDE0023 and IDE0024: Use expression body for operators - .NET - https://learn.microsoft.com/en-us/dotnet/fundamentals/code-analysis/style-rules/ide0023-ide0024

---

## Deprecated, Unsafe, Implementation-Dependent, and Version-Specific Notes

| Feature | Status / Note |
|---------|---------------|
| Expression-bodied methods and getter-only properties | Introduced in **C# 6.0**. |
| Expression-bodied constructors, destructors, setters, indexers, operators | Introduced in **C# 7.0**. |
| Expression-bodied finalizers | Supported since **C# 7.0**; use `IDisposable` for deterministic cleanup. |
| Expression-bodied operators | Supported since **C# 7.0**; IDE0023/IDE0024 enforce style. |
| Tuple deconstruction in constructors | Requires **C# 7.0+**. |
| Complex logic in expression bodies | **Not allowed**; use block bodies for multiple statements. |
| `=>` operator overloading | **Not allowed**; the `=>` token cannot be overloaded. |

---

## References (Consolidated)

- Expression-bodied members — C# Programming Guide - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/statements-expressions-operators/expression-bodied-members
- IDE0021: Use expression body for constructors - .NET - https://learn.microsoft.com/en-us/dotnet/fundamentals/code-analysis/style-rules/ide0021
- IDE0022: Use expression body for methods - .NET - https://learn.microsoft.com/en-us/dotnet/fundamentals/code-analysis/style-rules/ide0022
- IDE0025: Use expression body for properties - .NET - https://learn.microsoft.com/en-us/dotnet/fundamentals/code-analysis/style-rules/ide0025
- IDE0023 and IDE0024: Use expression body for operators - .NET - https://learn.microsoft.com/en-us/dotnet/fundamentals/code-analysis/style-rules/ide0023-ide0024
- Expression-bodied Function Syntax in C# — Pluralsight - https://www.pluralsight.com/resources/blog/guides/expression-bodied-function-syntax-in-c
- Expression-bodied members — Stack Overflow - https://stackoverflow.com/questions/28526628/expression-bodied-members
- C# 7 Expression Bodied Constructors — Stack Overflow - https://stackoverflow.com/questions/41996465/c-sharp-7-expression-bodied-constructors
- Finalizers — C# Programming Guide - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/finalizers
- C# code style — Expression-bodied members - https://github.com/cratis/applicationmodel/blob/0eac455579644b25fac8967e7cfc505c58d3aa1b/.cratis/ai/skills/cratis-engineering-csharp-conventions/references/code-style.md
- Code Style — Prefer expression-bodied members - https://github.com/brightercommand/brighter/blob/eab8e9de3efe4762c7fb5a11172d4ac40cda3900/.agent_instructions/code_style.md
- `=>` operator — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/lambda-operator

---