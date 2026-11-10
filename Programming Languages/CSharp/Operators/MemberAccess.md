# C# Member Access Operators — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
Member access operators in C# are the symbols and expressions that let you reach into a type, object, namespace, or pointer to use the members defined within it. They are the syntax of “reaching in”—whether you are calling a method on an object (`.`), following a pointer to a struct (`->`), safely navigating a chain of possibly-null references (`?.`), resolving a name through a namespace alias (`::`), or defining the body of a lambda or expression-bodied member (`=>`). The tilde (`~`) is a special syntactic marker for finalizers, the last cleanup hook a class can define.

**Technical Definition**
The C# language reference groups several operators under the umbrella of **member access**. The primary member access operator is the **dot** (`.`), which accesses a member of a namespace or a type. The **null-conditional operators** (`?.` and `?[]`) perform member or element access only when the left-hand operand is non-null, and they short-circuit otherwise. In **unsafe** contexts, the **pointer member access operator** (`->`) combines pointer dereferencing and member access. The **namespace alias qualifier** (`::`) resolves the left-hand identifier strictly as a namespace alias, extern alias, or the `global` alias, preventing ambiguity with types of the same name. The **lambda operator** (`=>`) separates a lambda’s parameter list from its body, and the same token serves as the separator in **expression-bodied members**. The **tilde** (`~`) introduces a **finalizer** (historically called a destructor), a parameterless, non-inheritable, non-callable method that runs before the garbage collector reclaims an object’s memory.

**Beginner-Friendly Explanation**
Member access operators are the **“dot and arrow” family**—the punctuation that tells C# where to look. The dot (`.`) is the most familiar: `person.Name` means “go into `person` and find `Name`.” The arrow (`->`) does the same thing but through a raw memory pointer, and only appears in unsafe code. The safe-navigation operators (`?.` and `?[]`) are like a cautious dot: “go in only if the thing is not empty.” The double-colon (`::`) is a namespace specialist: “look up this name strictly as a namespace alias.” The fat arrow (`=>`) is the “goes to” symbol of lambdas and concise method bodies. The tilde (`~`) marks the finalizer—the method that runs when an object is about to be collected. Together, these operators control **how you reach members** and **when it is safe to do so**.

---

### Key Characteristics

- **Seven operators, five jobs** — `.` (general member access), `->` (unsafe pointer member access), `?.` / `?[]` (null-safe access), `::` (namespace alias resolution), `=>` (lambda / expression body), and `~` (finalizer marker).
- **`.` is the workhorse** — It accesses members of namespaces, types, objects, and extension methods.
- **`?.` and `?[]` short-circuit** — If the left-hand side is `null`, the entire expression is `null` and no further members are evaluated.
- **`->` requires `unsafe`** — Pointer member access is only available in unsafe contexts and is used for unmanaged memory.
- **`::` is stricter than `.`** — It guarantees that the left-hand identifier is a namespace alias, even if a type with the same name exists.
- **`=>` is dual-purpose** — It defines lambdas and also expression-bodied members (methods, properties, constructors, finalizers).
- **`~` is not callable** — Finalizers are invoked automatically by the garbage collector; they cannot be called explicitly and cannot be inherited or overloaded.

---

### Prerequisites

- Basic understanding of C# program structure (namespaces, classes, methods, properties).
- Familiarity with value types, reference types, and `null`.
- Knowledge of namespaces and `using` directives (for `::`).
- Awareness of `unsafe` code and pointers (for `->`).
- Understanding of delegates and lambdas (for `=>`).
- Basic knowledge of garbage collection and `IDisposable` (for `~`).

---

### Related Programming Areas

- **Object-oriented programming** — `.` is how you interact with objects and their members.
- **Null safety** — `?.` and `?[]` are central to C#’s nullable reference type system.
- **Interoperability** — `->` is used in P/Invoke and low-level memory scenarios.
- **Namespace organisation** — `::` resolves naming conflicts in large codebases.
- **Functional programming** — `=>` is the syntax of lambdas and expression-bodied members.
- **Resource management** — `~` is the finalizer syntax, historically paired with `IDisposable`.

---

### Core Concepts / Features

1. Dot Operator (`.`)
2. Pointer Member Access (`->`)
3. Null-Conditional Member Access (`?.`)
4. Null-Conditional Indexer Access (`?[]`)
5. Namespace Alias Qualifier (`::`)
6. Lambda Operator (`=>`)
7. Finalizer Marker (`~`)
8. Member Access Best Practices

Each concept follows a uniform structure: definitions, purposes, syntax rules and structure, annotated examples with expected outputs, real-world cases, and references.

---

## 1. Dot Operator (`.`)

### Definitions

**Core Definition**
The dot operator (`.`) accesses a member—such as a field, property, method, event, or nested type—of a namespace or a type. It is the most fundamental and frequently used member access operator in C#.

**Technical Definition**
The member access expression `E.M`, where `E` is an expression of a type or a namespace and `M` is a member of that type or namespace, accesses the member `M`. The `.` token is used to form **qualified names** (e.g., `System.Collections.Generic`), to access **static and instance members** of a type (e.g., `Math.PI`, `person.Name`), and to access **extension methods**. When the left-hand operand is a namespace, the right-hand operand must name a type or nested namespace within it.

**Beginner-Friendly Explanation**
The dot is the **“go into”** operator. `person.Name` means “go into `person` and find `Name`.” `Math.PI` means “go into the `Math` type and find `PI`.” It is how you navigate from a container to the thing inside it.

---

### Purposes

- To access static and instance members of a type.
- To form qualified names for namespaces and types.
- To invoke extension methods.
- To chain member accesses into readable expressions.
- To navigate object graphs (`order.Customer.Address.City`).

---

### Syntax Rules and Structure

```
expression   .   memberName
namespaceOrType   .   memberName
```

**Component breakdown**

- `expression` / `namespaceOrType` — The container being accessed.
- `.` — The member access operator.
- `memberName` — The member, nested type, or namespace being reached.

#### Syntax Rules

- The left-hand operand must be a namespace, a type, or an expression that evaluates to a value.
- The right-hand operand must be a member of that namespace or type.
- The dot operator is **left-associative**: `a.b.c` is `(a.b).c`.
- The dot has higher precedence than most binary operators.

#### Constraints and Limitations

- Accessing an instance member through a `null` reference throws `NullReferenceException`.
- The dot cannot be used to access private members from outside the declaring type.
- Extension methods are accessed via `.` but are resolved as static methods.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Accessing Namespaces, Types, and Members

```csharp
using System;

class Person
{
    public string Name { get; set; }
    public static int Count { get; set; }
}

class Program
{
    static void Main()
    {
        Person p = new Person { Name = "Alice" };

        // Instance member access
        Console.WriteLine(p.Name);        // Alice

        // Static member access
        Person.Count = 1;
        Console.WriteLine(Person.Count);  // 1

        // Qualified name access
        Console.WriteLine(System.Math.PI); // 3.14159265358979
    }
}
```

**Expected Output**

```
Alice
1
3.14159265358979
```

**Why this output occurs**

- `p.Name` accesses the instance property through the object reference.
- `Person.Count` accesses the static property through the type name.
- `System.Math.PI` navigates the namespace hierarchy to reach the `PI` field.

---

#### Example 2: Chaining Member Access

```csharp
using System;

class Address { public string City { get; set; } = "London"; }
class Customer { public Address Address { get; set; } = new(); }
class Order { public Customer Customer { get; set; } = new(); }

class Program
{
    static void Main()
    {
        var order = new Order();
        Console.WriteLine(order.Customer.Address.City); // London
    }
}
```

**Expected Output**

```
London
```

**Why this output occurs**

- The dot chains left-to-right: `order.Customer` returns the `Customer`, `.Address` returns the `Address`, and `.City` returns the string.

---

### Real-World Cases

- **Domain models** — Navigating `order.Customer.Name`.
- **Static utilities** — `Math.Sqrt`, `Console.WriteLine`.
- **Namespace organisation** — `System.Collections.Generic.List<T>`.
- **Extension methods** — `myString.IsNullOrEmpty()` (extension).

---

### References

- Member access expression — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/member-access-operators#member-access-expression-
- Member access operators and expressions — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/member-access-operators

---

## 2. Pointer Member Access (`->`)

### Definitions

**Core Definition**
The pointer member access operator (`->`) combines pointer dereferencing with member access. It is used in **unsafe** code to access a member of a struct or class through a pointer.

**Technical Definition**
The `->` operator is available only in an **unsafe** context. `pointer->member` is semantically equivalent to `(*pointer).member`: it dereferences the pointer and then accesses the member. Pointers in C# can point only to **unmanaged types** (structs that contain only unmanaged fields); they cannot point to reference types or to structs that contain references. The `->` operator is part of C#’s unsafe code feature set, which must be explicitly enabled in the project file.

**Beginner-Friendly Explanation**
The arrow is the **pointer’s dot**. When you have a raw memory address (a pointer) to a struct, you cannot use `.` directly—you use `->` to say “go to that address and grab this member.” It only appears in `unsafe` code, which is code that is allowed to work with raw memory.

---

### Purposes

- To access members of unmanaged structs through pointers.
- To write low-level, performance-critical code that avoids managed overhead.
- To interoperate with native libraries and memory-mapped data.
- To implement custom memory management and buffer manipulation.
- To work with `fixed` buffers and stack-allocated data.

---

### Syntax Rules and Structure

```
pointer   ->   memberName
```

**Component breakdown**

- `pointer` — A pointer variable of type `T*`, where `T` is an unmanaged type.
- `->` — The pointer member access operator.
- `memberName` — A field or property of `T`.

#### Syntax Rules

- Must be inside an `unsafe` context (`unsafe` block, method, or type).
- The pointer must point to a valid, allocated instance.
- `pointer->member` is equivalent to `(*pointer).member`.
- The pointed-to type must be an unmanaged type.

#### Constraints and Limitations

- Cannot be used with reference types or structs containing references.
- Requires enabling unsafe code in the project (`<AllowUnsafeBlocks>true</AllowUnsafeBlocks>`).
- Pointers are not managed by the garbage collector; incorrect use can corrupt memory.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Accessing a Struct Through a Pointer

```csharp
using System;

struct Point
{
    public int X;
    public int Y;
}

class Program
{
    static unsafe void Main()
    {
        Point p = new Point { X = 10, Y = 20 };
        Point* ptr = &p;

        Console.WriteLine(ptr->X);   // 10
        Console.WriteLine(ptr->Y);   // 20
    }
}
```

**Expected Output**

```
10
20
```

**Why this output occurs**

- `&p` obtains a pointer to the local struct.
- `ptr->X` dereferences the pointer and accesses the `X` field, equivalent to `(*ptr).X`.

---

#### Example 2: Modifying a Member Through a Pointer

```csharp
using System;

struct Counter { public int Value; }

class Program
{
    static unsafe void Main()
    {
        Counter c = new Counter { Value = 5 };
        Counter* p = &c;

        p->Value = 42;
        Console.WriteLine(c.Value); // 42
    }
}
```

**Expected Output**

```
42
```

**Why this output occurs**

- Writing through `p->Value` modifies the same memory location as `c.Value`.

---

### Real-World Cases

- **Interop with native libraries** — Passing pointers to structs to C functions.
- **High-performance buffers** — Manipulating raw memory for image or audio processing.
- **Embedded systems** — Direct memory-mapped I/O access.
- **Custom allocators** — Managing unmanaged memory pools.

---

### References

- Pointer member access operator `->` — C# reference (unsafe code) - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/member-access-operators#pointer-member-access-operator-
- Unsafe code and pointers — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/unsafe-code/

---

## 3. Null-Conditional Member Access (`?.`)

### Definitions

**Core Definition**
The null-conditional member access operator (`?.`) accesses a member only if the left-hand operand is **not null**. If the operand is null, the entire expression evaluates to `null` instead of throwing an exception.

**Technical Definition**
The `?.` operator is a **null-conditional** operator. `A?.B` evaluates to `null` when `A` is `null`; otherwise, it evaluates to `A.B`. When the member `B` has a value type, the result of `A?.B` is a **nullable value type** (`T?`). The operator short-circuits: if the left-hand side is null, nothing to the right is evaluated. Multiple `?.` operators can be chained, and the chain stops at the first `null`.

**Beginner-Friendly Explanation**
The `?.` is a **cautious dot**. It says: “If the thing on the left is not empty, go ahead and grab the member; if it is empty, just return `null` and don’t crash.” It is the primary tool for writing null-safe code.

---

### Purposes

- To safely access members of objects that might be `null`.
- To avoid `NullReferenceException` without explicit `if` checks.
- To chain member accesses safely (`order.Customer?.Address?.City`).
- To simplify null-handling in complex expressions.
- To integrate with nullable reference types and null-state analysis.

---

### Syntax Rules and Structure

```
expression   ?.   memberName
```

**Component breakdown**

- `expression` — A potentially null value.
- `?.` — The null-conditional member access operator.
- `memberName` — The member to access if the expression is non-null.

#### Syntax Rules

- If the left-hand operand is `null`, the result is `null`.
- If the member is a value type, the result type is the corresponding nullable value type.
- The operator short-circuits: no further evaluation occurs when the left-hand side is `null`.
- Can be chained: `A?.B?.C`.

#### Constraints and Limitations

- The result must be handled as potentially `null` (use `??` or null checks).
- Cannot be used to assign to a member directly (use `??=` or explicit checks).
- Does not work with non-nullable value types on the left (they cannot be `null`).

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Safe Member Access

```csharp
using System;

class Program
{
    static void Main()
    {
        string? name = null;
        int? length = name?.Length;
        Console.WriteLine(length.HasValue);   // False

        name = "C#";
        Console.WriteLine(name?.Length);        // 2
    }
}
```

**Expected Output**

```
False
2
```

**Why this output occurs**

- When `name` is `null`, `name?.Length` returns `null` (a `Nullable<int>` with no value).
- When `name` is `"C#"`, the length is accessed normally.

---

#### Example 2: Chaining `?.`

```csharp
using System;

class Address { public string? City { get; set; } }
class Customer { public Address? Address { get; set; } }
class Order { public Customer? Customer { get; set; } }

class Program
{
    static void Main()
    {
        var order = new Order();
        string? city = order.Customer?.Address?.City;
        Console.WriteLine(city ?? "(no city)"); // (no city)
    }
}
```

**Expected Output**

```
(no city)
```

**Why this output occurs**

- `order.Customer` is `null`, so the entire chain short-circuits and returns `null`; `??` provides the fallback.

---

### Real-World Cases

- **Event invocation** — `handler?.Invoke(this, e)`.
- **DTO navigation** — Safely reading optional nested properties.
- **Logging** — `logger?.Log(message)`.
- **UI frameworks** — Accessing controls that may not yet be initialised.

---

### References

- Null-conditional operators `?.` and `?[]` — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/member-access-operators#null-conditional-operators--and-
- Null operators in C# — C# fundamentals - https://learn.microsoft.com/en-us/dotnet/csharp/fundamentals/null-safety/null-operators

---

## 4. Null-Conditional Indexer Access (`?[]`)

### Definitions

**Core Definition**
The null-conditional indexer access operator (`?[]`) applies the same null-safe semantics as `?.` but for **indexer and array element access**.

**Technical Definition**
`A?[index]` evaluates to `null` when `A` is `null`; otherwise, it evaluates to `A[index]`. Like `?.`, it short-circuits and returns a nullable result when the element type is a value type. It is used when the collection itself might be null.

**Beginner-Friendly Explanation**
`?[]` is the **cautious square bracket**. It says: “If the collection is not empty, grab the element at this index; if the collection is empty, return `null` instead of crashing.”

---

### Purposes

- To safely access array or collection elements when the collection might be `null`.
- To avoid `NullReferenceException` on null arrays or lists.
- To chain with `?.` for complex null-safe navigation.
- To simplify code that would otherwise require explicit null checks.

---

### Syntax Rules and Structure

```
expression   ?[   index   ]
```

**Component breakdown**

- `expression` — A potentially null array or indexable object.
- `?[` — The null-conditional indexer operator.
- `index` — The index expression.
- `]` — Closes the access.

#### Syntax Rules

- If the left-hand operand is `null`, the result is `null`.
- The element type’s nullable form is the result type when the element is a value type.
- Short-circuits like `?.`.

#### Constraints and Limitations

- Does not work with non-nullable value types on the left.
- The result must be handled as potentially `null`.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Safe Array Access

```csharp
using System;

class Program
{
    static void Main()
    {
        string[]? tags = null;
        string? first = tags?[0];
        Console.WriteLine(first ?? "(none)");   // (none)

        tags = ["csharp", "dotnet"];
        Console.WriteLine(tags?[0]);             // csharp
    }
}
```

**Expected Output**

```
(none)
csharp
```

**Why this output occurs**

- When `tags` is `null`, `tags?[0]` returns `null`; `??` provides the fallback.
- When `tags` is non-null, indexing works normally.

---

#### Example 2: Chaining `?.` and `?[]`

```csharp
using System;
using System.Collections.Generic;

class Program
{
    static void Main()
    {
        List<string>? list = null;
        string? item = list?.Count > 0 ? list?[0] : null;
        Console.WriteLine(item ?? "(empty)"); // (empty)
    }
}
```

**Expected Output**

```
(empty)
```

**Why this output occurs**

- The null-conditional checks prevent access on the null list.

---

### Real-World Cases

- **Configuration arrays** — Accessing optional arrays of settings.
- **Parsed data** — Reading elements from potentially null collections.
- **API responses** — Handling optional array fields.

---

### References

- Null-conditional operators `?.` and `?[]` — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/member-access-operators#null-conditional-operators--and-
- Null operators in C# — C# fundamentals - https://learn.microsoft.com/en-us/dotnet/csharp/fundamentals/null-safety/null-operators

---

## 5. Namespace Alias Qualifier (`::`)

### Definitions

**Core Definition**
The namespace alias qualifier (`::`) accesses a member of an aliased namespace, ensuring that the left-hand identifier is resolved strictly as a namespace alias, extern alias, or the `global` alias—never as a type or nested namespace with the same name.

**Technical Definition**
The `::` qualifier can be used only between two identifiers. Its left-hand identifier must be a **namespace alias** (created with a `using` alias directive), an **extern alias**, or the **`global`** alias. The `::` operator differs from `.` in that it guarantees namespace resolution: even if a type or namespace with the same name exists in scope, `::` treats the left-hand identifier as an alias. The `global` alias always references the global namespace when used with `::`.

**Beginner-Friendly Explanation**
The `::` operator is a **namespace specialist**. Sometimes you create an alias like `using forwpf = System.Windows;`. Later, if someone creates a type named `forwpf`, the dot might get confused. The `::` operator says: “No, I mean the namespace alias, not the type.” It prevents naming collisions.

---

### Purposes

- To resolve namespace aliases unambiguously.
- To avoid collisions between namespace aliases and type names.
- To access members of extern aliases.
- To explicitly access the global namespace with `global::`.
- To improve clarity in large codebases with many namespaces.

---

### Syntax Rules and Structure

```
aliasName   ::   memberName
global   ::   fullyQualifiedName
externAlias   ::   memberName
```

**Component breakdown**

- `aliasName` — A namespace alias defined by `using`.
- `::` — The namespace alias qualifier.
- `memberName` — A type or namespace within the alias.
- `global` — The global namespace alias.

#### Syntax Rules

- The left-hand identifier must be a namespace alias, extern alias, or `global`.
- `::` can only appear between two identifiers.
- `global::` always refers to the global namespace, even if a user-defined `global` alias exists.
- `::` has higher precedence than `.`.

#### Constraints and Limitations

- Cannot be used with arbitrary identifiers; the left side must be an alias.
- The `global` keyword is the global alias only when it is the left-hand identifier of `::`.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Using a Namespace Alias

```csharp
using System;
using forwpf = System.Windows;   // hypothetical namespace

class Program
{
    static void Main()
    {
        // Using the alias with :: ensures namespace resolution
        forwpf::Point p = new forwpf::Point(10, 20);
    }
}

namespace System.Windows
{
    public struct Point
    {
        public int X, Y;
        public Point(int x, int y) => (X, Y) = (x, y);
    }
}
```

**Expected Output**

```
(no output; compiles successfully)
```

**Why this output occurs**

- `forwpf::Point` resolves `Point` within the aliased namespace `System.Windows`.

---

#### Example 2: `global::` to Avoid Collisions

```csharp
using System;

namespace MyCompany.MyProduct.System
{
    class Console
    {
        public static void WriteLine(string s) => throw new Exception("Wrong Console!");
    }

    class Program
    {
        static void Main()
        {
            // Without global::, this would resolve to MyCompany.MyProduct.System.Console
            global::System.Console.WriteLine("Using global alias"); // Using global alias
        }
    }
}
```

**Expected Output**

```
Using global alias
```

**Why this output occurs**

- `global::System.Console` bypasses the user-defined `System` namespace and accesses the real .NET `System.Console`.

---

### Real-World Cases

- **Large solutions** — Disambiguating namespace aliases from types.
- **Extern aliases** — Referencing assemblies with conflicting namespaces.
- **Global namespace access** — Ensuring the root namespace is used.

---

### References

- Namespace alias operator `::` — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/namespace-alias-qualifier
- `::` operator — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/namespace-alias-qualifier

---

## 6. Lambda Operator (`=>`)

### Definitions

**Core Definition**
The lambda operator (`=>`) separates the parameter list of a lambda expression from its body. The same token also separates a member declaration from its expression body in **expression-bodied members**.

**Technical Definition**
In a **lambda expression**, `=>` separates input parameters on the left from the body on the right. The body can be an expression (`x => x * 2`) or a statement block (`x => { return x * 2; }`). In an **expression body definition**, `member => expression` provides a concise implementation for methods, properties, constructors, finalizers, and accessors. The `=>` token is right-associative and has the same precedence as assignment.

**Beginner-Friendly Explanation**
The `=>` symbol is pronounced **“goes to.”** In a lambda, it means “these inputs go to this result”: `x => x * 2` means “take `x`, go to `x * 2`.” In an expression-bodied method, it means “this member is defined as this expression”: `public int Double(int x) => x * 2;`.

---

### Purposes

- To define inline anonymous functions (lambdas).
- To write concise expression-bodied members.
- To pass behaviour as data to LINQ, delegates, and higher-order methods.
- To reduce boilerplate for simple methods and properties.
- To enable functional-style programming in C#.

---

### Syntax Rules and Structure

```
// Lambda expression
(parameters)   =>   expression
(parameters)   =>   { statements }

// Expression body definition
member   =>   expression
```

**Component breakdown**

- `parameters` — The lambda’s input parameters (parentheses optional for a single parameter).
- `=>` — The lambda operator / expression-body separator.
- `expression` — The result expression or the member’s implementation.

#### Syntax Rules

- A lambda with one parameter may omit parentheses: `x => x * 2`.
- A lambda with zero parameters uses empty parentheses: `() => Console.WriteLine()`.
- The body can be an expression or a block.
- Expression-bodied members can be used for methods, constructors, finalizers, properties, and accessors.
- `=>` is right-associative.

#### Constraints and Limitations

- Lambda expressions cannot be used without a target type (delegate or expression tree) in some contexts.
- Expression-bodied members must be a single expression (or a statement expression for `void` members).
- Finalizers and constructors with expression bodies must use a statement expression.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Lambda Expression in LINQ

```csharp
using System;
using System.Linq;

class Program
{
    static void Main()
    {
        int[] numbers = { 1, 2, 3, 4, 5 };
        var evens = numbers.Where(n => n % 2 == 0);

        foreach (var n in evens)
            Console.WriteLine(n);   // 2 4
    }
}
```

**Expected Output**

```
2
4
```

**Why this output occurs**

- `n => n % 2 == 0` is a lambda that returns `true` for even numbers; `Where` uses it to filter.

---

#### Example 2: Expression-Bodied Method and Property

```csharp
using System;

class Circle
{
    public double Radius { get; }

    public Circle(double radius) => Radius = radius;

    public double Area => Math.PI * Radius * Radius;

    public override string ToString() => $"Circle(r={Radius})";
}

class Program
{
    static void Main()
    {
        var c = new Circle(2);
        Console.WriteLine(c.Area);     // 12.566370614359172
        Console.WriteLine(c);          // Circle(r=2)
    }
}
```

**Expected Output**

```
12.566370614359172
Circle(r=2)
```

**Why this output occurs**

- `=>` provides concise bodies for the constructor, the `Area` property, and `ToString`.

---

### Real-World Cases

- **LINQ queries** — `Where(x => x.IsActive)`.
- **Event handlers** — `button.Click += (s, e) => { ... };`.
- **Expression-bodied members** — Concise properties and methods.
- **Functional composition** — `Func<int, int> square = x => x * x;`.

---

### References

- Lambda expression (`=>`) operator — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/lambda-operator
- Lambda expressions — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/lambda-expressions

---

## 7. Finalizer Marker (`~`)

### Definitions

**Core Definition**
The tilde (`~`) introduces a **finalizer** (historically called a destructor), a special method that runs before the garbage collector reclaims an object’s memory.

**Technical Definition**
A finalizer is declared with `~` followed by the class name, takes no parameters, has no access modifiers, and cannot be called explicitly. It is invoked automatically by the garbage collector when an object is no longer reachable, before the object’s memory is reclaimed. Finalizers are used to release **unmanaged resources** (file handles, native memory). They are **non-deterministic**: you cannot predict when they run. A class can have only one finalizer, and finalizers cannot be inherited or overloaded. In C#, finalizers are not destructors in the C++ sense; for deterministic cleanup, the `IDisposable` pattern with `Dispose` is recommended.

**Beginner-Friendly Explanation**
The tilde (`~`) marks the **cleanup method** that runs when an object is about to be garbage-collected. It is the last chance to release resources that the garbage collector does not know about, such as open files or raw memory. Because you cannot control when it runs, it is a safety net—not a replacement for `Dispose`.

---

### Purposes

- To release unmanaged resources before an object is collected.
- To provide a safety net when `Dispose` is not called.
- To wrap native handles and ensure they are eventually freed.
- To support interoperability with unmanaged libraries.
- To implement the dispose pattern with a finalizer as a fallback.

---

### Syntax Rules and Structure

```
~ClassName()
{
    // cleanup statements
}
```

**Component breakdown**

- `~` — The finalizer marker.
- `ClassName` — The name of the declaring class (must match exactly).
- `()` — No parameters are allowed.
- Body — Cleanup statements.

#### Syntax Rules

- No access modifiers, parameters, or return type.
- Cannot be called explicitly.
- Cannot be inherited or overloaded.
- Only classes can have finalizers; structs cannot.
- A class can have at most one finalizer.

#### Constraints and Limitations

- Finalization is non-deterministic; the GC decides when to run it.
- Objects with finalizers survive longer and may be promoted to older generations, impacting performance.
- For deterministic cleanup, implement `IDisposable` and call `Dispose`.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Basic Finalizer

```csharp
using System;

class ResourceHolder
{
    private IntPtr _handle;

    public ResourceHolder()
    {
        _handle = /* allocate unmanaged resource */ new IntPtr(42);
        Console.WriteLine("Resource acquired");
    }

    ~ResourceHolder()
    {
        // Release unmanaged resource
        _handle = IntPtr.Zero;
        Console.WriteLine("Finalizer running");
    }
}

class Program
{
    static void Main()
    {
        var r = new ResourceHolder();
        // r becomes unreachable; finalizer will run eventually
    }
}
```

**Expected Output (eventually)**

```
Resource acquired
Finalizer running
```

**Why this output occurs**

- The finalizer runs when the GC determines the object is unreachable; the exact timing is non-deterministic.

---

#### Example 2: Expression-Bodied Finalizer

```csharp
using System;

public class Destroyer
{
    public override string ToString() => GetType().Name;

    ~Destroyer() => Console.WriteLine($"The {ToString()} finalizer is executing.");
}

class Program
{
    static void Main()
    {
        var d = new Destroyer();
    }
}
```

**Expected Output (eventually)**

```
The Destroyer finalizer is executing.
```

**Why this output occurs**

- The finalizer is defined with `=>`, using an expression body.

---

### Real-World Cases

- **File handles** — Closing native file descriptors.
- **Native memory** — Freeing memory allocated with `Marshal.AllocHGlobal`.
- **COM objects** — Releasing COM references.
- **SafeHandle** — Wrapping unmanaged handles; `SafeHandle` is preferred over writing finalizers manually.

---

### References

- Finalizers — C# programming guide - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/finalizers
- Examine class finalizers — Microsoft Learn training - https://learn.microsoft.com/en-sg/training/modules/manage-class-implementations/7-examine-class-finalizers
- `~` (finalizer) — C# language specification - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/language-specification/classes#1513-finalizers

---

## 8. Member Access Best Practices

### Definitions

**Core Definition**
Member access best practices are the guidelines for using member access operators correctly, safely, and readably—especially when dealing with `null`, namespaces, pointers, and lambdas.

**Technical Definition**
Best practices include preferring `?.` and `?[]` over explicit null checks for safe navigation, using `::` to disambiguate namespace aliases, keeping `->` confined to well-reviewed unsafe code, using `=>` for concise expression-bodied members without sacrificing clarity, and avoiding finalizers unless wrapping unmanaged resources. These guidelines align with Microsoft’s null-safety guidance, C# coding conventions, and the .NET runtime coding style.

**Beginner-Friendly Explanation**
Best practices are the habits that keep member access code safe and clear: use the cautious operators (`?.`, `?[]`) when something might be empty, use the namespace specialist (`::`) when names collide, keep the pointer arrow (`->`) in small, well-checked unsafe blocks, use the fat arrow (`=>`) for short and clear bodies, and do not write finalizers unless you really need to clean up unmanaged resources.

---

### Purposes

- To prevent `NullReferenceException` through safe-navigation operators.
- To resolve naming ambiguities with `::`.
- To minimise unsafe code and isolate `->` usage.
- To keep expression-bodied members readable.
- To use finalizers only where necessary and correctly.

---

### Guidelines and Structure

#### General Guidelines

```
// Prefer null-conditional operators for safe navigation
var city = order.Customer?.Address?.City;

// Use ?[] for potentially null collections
var first = tags?[0];

// Use :: to disambiguate namespace aliases
forwpf::Point p = new forwpf::Point(1, 2);

// Keep unsafe code and -> usage minimal and reviewed
unsafe { ptr->Value = 42; }

// Use => for concise members when the body is a single clear expression
public double Area => Math.PI * Radius * Radius;

// Avoid finalizers; prefer IDisposable and SafeHandle
// Only add a finalizer when wrapping unmanaged resources
```

**Component breakdown**

- **`?.` / `?[]`** — Safe navigation; short-circuit on `null`.
- **`::`** — Namespace alias resolution; prevents type-name collisions.
- **`->`** — Unsafe pointer access; isolate and review.
- **`=>`** — Concise bodies; use when readability improves.
- **`~`** — Finalizers; use only for unmanaged resources, preferably via `SafeHandle`.

#### Rules and Constraints

- Do not dereference potentially null references with `.`; use `?.` or explicit checks.
- Do not use `::` with arbitrary identifiers; the left side must be an alias.
- Do not expose pointers or unsafe code in public APIs without strong justification.
- Do not use expression-bodied members for complex multi-step logic.
- Do not rely on finalizers for deterministic cleanup; use `Dispose`.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Safe Navigation vs. Direct Access

```csharp
using System;

class Customer { public string? Name { get; set; } }

class Program
{
    static void Main()
    {
        Customer? c = null;

        // Safe: returns null instead of throwing
        string? name = c?.Name;
        Console.WriteLine(name ?? "(no customer)"); // (no customer)

        // Unsafe: would throw NullReferenceException
        // string bad = c.Name;
    }
}
```

**Expected Output**

```
(no customer)
```

**Why this output occurs**

- The `?.` operator short-circuits on `null`, avoiding the exception.

---

#### Example 2: Expression-Bodied Member for Clarity

```csharp
using System;

class Temperature
{
    public double Celsius { get; }

    public Temperature(double celsius) => Celsius = celsius;

    public double Fahrenheit => Celsius * 9 / 5 + 32;
}

class Program
{
    static void Main()
    {
        var t = new Temperature(100);
        Console.WriteLine(t.Fahrenheit); // 212
    }
}
```

**Expected Output**

```
212
```

**Why this output occurs**

- The expression-bodied property keeps the formula concise and readable.

---

### Real-World Cases

- **API clients** — Using `?.` to safely navigate optional response fields.
- **Large solutions** — Using `::` to resolve namespace alias conflicts.
- **Interop libraries** — Isolating `->` in reviewed unsafe blocks.
- **Domain models** — Using `=>` for computed properties.
- **Resource wrappers** — Using `SafeHandle` instead of manual finalizers.

---

### References

- Null operators in C# — C# fundamentals - https://learn.microsoft.com/en-us/dotnet/csharp/fundamentals/null-safety/null-operators
- Member access operators and expressions — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/member-access-operators
- Namespace alias operator `::` — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/namespace-alias-qualifier
- Lambda expression (`=>`) operator — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/lambda-operator
- Finalizers — C# programming guide - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/finalizers

---

## Deprecated, Unsafe, Implementation-Dependent, and Version-Specific Notes

| Feature | Status / Note |
|---------|---------------|
| `->` (pointer member access) | **Unsafe**; requires `<AllowUnsafeBlocks>true</AllowUnsafeBlocks>`. |
| `::` with `global` | The `global` alias is only a namespace alias when used as the left-hand identifier of `::`. |
| `?.` and `?[]` | Introduced in **C# 6.0**; stable and recommended for null-safe navigation. |
| `=>` in expression-bodied members | Expanded over several C# versions; supported for methods, properties, constructors, finalizers, and accessors. |
| `~` (finalizer) | **Non-deterministic**; performance impact; prefer `IDisposable` and `SafeHandle`. |
| Pointer member access on managed types | **Not allowed**; pointers can only point to unmanaged types. |

---

## References (Consolidated)

- Member access operators and expressions — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/member-access-operators
- Member access expression — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/member-access-operators#member-access-expression-
- Null-conditional operators `?.` and `?[]` — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/member-access-operators#null-conditional-operators--and-
- Pointer member access operator `->` — C# reference (unsafe code) - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/member-access-operators#pointer-member-access-operator-
- Unsafe code and pointers — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/unsafe-code/
- Namespace alias operator `::` — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/namespace-alias-qualifier
- Lambda expression (`=>`) operator — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/lambda-operator
- Lambda expressions — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/lambda-expressions
- Finalizers — C# programming guide - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/finalizers
- Examine class finalizers — Microsoft Learn training - https://learn.microsoft.com/en-sg/training/modules/manage-class-implementations/7-examine-class-finalizers
- Null operators in C# — C# fundamentals - https://learn.microsoft.com/en-us/dotnet/csharp/fundamentals/null-safety/null-operators
- `~` (finalizer) — C# language specification - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/language-specification/classes#1513-finalizers

---