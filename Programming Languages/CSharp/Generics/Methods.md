# C# Generic Methods — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
A **generic method** is a method that is declared with its own **type parameters**, allowing it to operate on different data types while maintaining full compile-time type safety. Unlike a non-generic method that is tied to a specific type (like `int` or `string`), a generic method uses a placeholder type (commonly `T`) that the caller supplies—either explicitly or through **type inference**—when the method is invoked. This enables you to write a single, reusable implementation of an algorithm (such as swapping two values, finding the maximum, or filtering a collection) that works correctly and efficiently for any type, without casting, boxing, or duplicating code.

**Technical Definition**
A generic method is a method that is declared with type parameters. Its declaration introduces one or more **type parameters** in angle brackets after the method name, and these type parameters can be used throughout the method signature and body wherever a type is expected. Generic methods can be declared in both generic and non-generic classes; within a generic class, non-generic methods can access the class-level type parameters, while a generic method that declares its own type parameters introduces a **new scope** for those names (and can cause compiler warning CS0693 if it reuses the class-level name). The compiler often **infers the type arguments** from the values passed to the method, eliminating the need for explicit specification. Generic methods support **constraints** via the `where` clause, which restrict the set of valid type arguments and declare the capabilities that `T` must have, enabling the method body to access members such as `IComparable<T>.CompareTo` or `IEnumerable<T>.GetEnumerator`. Generic methods can be **overloaded** on different type parameter lists and can use the type parameter as a return type.

**Beginner-Friendly Explanation**
A generic method is a **method that works with any type**. Imagine you want to write a method that swaps two values. Without generics, you might write `SwapInt(int a, int b)`, `SwapString(string a, string b)`, and `SwapDouble(double a, double b)`—three nearly identical methods. With generics, you write **one** method, `Swap<T>(ref T a, ref T b)`, and use it for `int`, `string`, or any other type. The `<T>` is a **placeholder** that gets replaced with the actual type when you call the method: `Swap<int>(ref x, ref y)` or simply `Swap(ref x, ref y)` because the compiler is smart enough to **figure out** `T` from the arguments you pass. This saves code, prevents mistakes, and makes your programs faster because the compiler generates specialised, type-safe code for each type you use.

---

### Key Characteristics

- **Own type parameters** — A generic method declares its own type parameters, independent of any enclosing generic class.
- **Type inference** — The compiler can often infer the type arguments from the method arguments, so you do not need to write `<int>` explicitly.
- **Compile-time type safety** — The compiler enforces that only compatible types are used, and it generates specialised code for each closed generic method.
- **Performance benefits** — Generics avoid boxing and unboxing for value types, improving speed and reducing heap allocations.
- **Constraints refine capabilities** — The `where` clause restricts valid type arguments and enables access to members of `T`.
- **Overloadable** — Generic methods can be overloaded on different type parameter lists and arities.
- **Type parameter as return type** — A generic method can return `T`, `T[]`, or any type involving `T`.

---

### Prerequisites

- Basic understanding of C# methods, parameters, and return types.
- Familiarity with generics (`<T>`, type parameters, generic classes).
- Knowledge of interfaces and base classes (for constraints).
- Awareness of value types vs. reference types and boxing.
- Basic understanding of method overloading and overload resolution.

---

### Related Programming Areas

- **Generic classes** — Generic methods often complement generic classes, providing reusable algorithms that operate on the class’s type parameters.
- **LINQ** — Many LINQ operators (`Where<T>`, `Select<T>`, `OrderBy<T>`) are generic methods on `IEnumerable<T>`.
- **Collections** — Generic methods like `List<T>.Find<T>` and `Array.Sort<T>` operate on generic collections.
- **Delegates** — Generic methods can be converted to generic delegates (`Func<T, TResult>`, `Action<T>`).
- **Utility libraries** — `Swap<T>`, `Max<T>`, `Min<T>`, and similar helpers are classic generic methods.

---

### Core Concepts / Features

1. Generic Method Declaration
2. Type Inference
3. Generic Method Invocation
4. Generic Method Constraints
5. Generic Method Best Practices

Each concept follows a uniform structure: definitions, purposes, syntax rules and structure, annotated code examples with expected outputs, real-world cases, and references.

---

## 1. Generic Method Declaration

### Definitions

**Core Definition**
Generic method declaration is the syntax that introduces a method with its own **type parameter list** (e.g., `<T>`), enabling the method to operate on any type supplied by the caller.

**Technical Definition**
A generic method is declared by inserting a **type parameter list** in angle brackets between the method name and its parameter list: `ReturnType MethodName<T>(parameters)`. The type parameters can be used throughout the method signature—as parameter types, return types, and within the method body—wherever a type is expected. A generic method can be declared in a non-generic class, a generic class, a struct, an interface, or a record. Within a generic class, a generic method that declares its own type parameter with the same name as the class-level type parameter triggers **compiler warning CS0693** because the inner type parameter **hides** the outer one; the recommended practice is to use a different name (e.g., `U` instead of `T`). Generic methods can be **overloaded** by type parameter count or by parameter types, just like non-generic methods.

**Beginner-Friendly Explanation**
Declaring a generic method is **writing a method with a blank type**. You put `<T>` after the method name: `public T GetFirst<T>(List<T> list)`. Now the method works with any type.

---

### Purposes

- To write **reusable algorithms** that work with any type.
- To **avoid code duplication** across similar types.
- To provide **compile-time type safety** for generic operations.
- To enable **type inference** for concise call sites.
- To support **constraints** that refine the type parameter’s capabilities.

---

### Syntax Rules and Structure

```
[access modifier] [modifiers] ReturnType MethodName<T1, T2, ...>(parameters)
{
    // method body
}
```

**Component breakdown**

- `access modifier` — `public`, `private`, etc.
- `modifiers` — `static`, `virtual`, etc. (generic methods can be `static` or instance methods).
- `ReturnType` — The return type; can be `T`, `T[]`, or any type involving `T`.
- `MethodName<T1, T2, ...>` — The method name followed by the type parameter list.
- `parameters` — Parameters; can use `T1`, `T2`, etc. as types.

#### Syntax Rules

- The type parameter list appears **after the method name** and **before the parameter list**.
- Type parameters are conventionally named `T`, `T1`, `T2`, etc., or with descriptive names like `TKey`, `TValue`.
- A generic method can have one or more type parameters.
- Generic methods can be overloaded by type parameter count or by parameter types.
- Within a generic class, avoid reusing the class-level type parameter name in a generic method (CS0693).

#### Constraints and Limitations

- Type inference does not work with methods that have **no parameters**, because the compiler cannot infer `T` from a constraint or return value alone.
- Generic methods cannot be `abstract` or `virtual` unless the class is generic and the method’s type parameters are compatible (advanced scenario).

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Basic Generic Method

```csharp
using System;

class Program
{
    // Generic method declaration
    static void Swap<T>(ref T lhs, ref T rhs)
    {
        T temp = lhs;
        lhs = rhs;
        rhs = temp;
    }

    static void Main()
    {
        int a = 1, b = 2;
        Swap<int>(ref a, ref b);          // explicit type argument
        Console.WriteLine($"{a} {b}");    // 2 1

        string x = "hello", y = "world";
        Swap(ref x, ref y);                // inferred type argument
        Console.WriteLine($"{x} {y}");    // world hello
    }
}
```

**Expected Output**

```
2 1
world hello
```

**Why this output occurs**

- The `Swap<T>` method is declared with a single type parameter `T` and works for both `int` and `string`. The compiler generates type-safe code for each closed generic method.

---

#### Example 2: Generic Method with Type Parameter as Return Type

```csharp
using System;

class Program
{
    // Returns an array of type T
    static T[] Swap<T>(T a, T b) => new[] { b, a };

    static void Main()
    {
        var ints = Swap(1, 2);
        Console.WriteLine(string.Join(", ", ints));   // 2, 1

        var strings = Swap("A", "B");
        Console.WriteLine(string.Join(", ", strings)); // B, A
    }
}
```

**Expected Output**

```
2, 1
B, A
```

**Why this output occurs**

- The return type `T[]` uses the type parameter, enabling the method to return an array of the same type as its arguments.

---

#### Example 3: Overloading Generic Methods

```csharp
using System;

class Program
{
    static void DoWork() => Console.WriteLine("No type parameters");
    static void DoWork<T>() => Console.WriteLine(typeof(T).Name);
    static void DoWork<T, U>() => Console.WriteLine($"{typeof(T).Name}, {typeof(U).Name}");

    static void Main()
    {
        DoWork();                  // No type parameters
        DoWork<int>();             // Int32
        DoWork<int, string>();     // Int32, String
    }
}
```

**Expected Output**

```
No type parameters
Int32
Int32, String
```

**Why this output occurs**

- Generic methods can be overloaded on different type parameter lists and arities in the same class.

---

### Real-World Cases

- **Swapping values** — `Swap<T>(ref T a, ref T b)` is the canonical generic method.
- **Searching collections** — `List<T>.Find<T>` and `Array.Find<T>`.
- **Sorting** — `Array.Sort<T>` and `List<T>.Sort` (generic comparers).
- **LINQ operators** — `Enumerable.Where<T>`, `Enumerable.Select<T>`.

---

### References

- Generic methods (C# programming guide) — Microsoft Learn - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/generics/generic-methods
- Generic types and methods — C# - Microsoft Learn (GitHub) - https://github.com/dotnet/docs/blob/main/docs/csharp/fundamentals/types/generics.md

---

## 2. Type Inference

### Definitions

**Core Definition**
Type inference is the compiler’s ability to **automatically determine the type arguments** for a generic method from the types of the arguments passed at the call site, eliminating the need for explicit `<T>` specification.

**Technical Definition**
When a generic method is called without specifying type arguments, a **type inference process** attempts to infer type arguments for the call. The presence of type inference allows a more convenient syntax to be used for calling a generic method, and allows the programmer to avoid specifying redundant type information. Type inference occurs as part of the **binding-time processing** of a method invocation and takes place **before** the overload resolution step. The compiler can infer type parameters based on the method arguments you pass in; it **cannot** infer type parameters only from a constraint or return value. Therefore, type inference does **not work** with methods that have no parameters. The inference process applies to all generic methods that share the same name; methods where inference fails do not participate in overload resolution. Type inference is also used when converting a method group containing a generic method to a delegate type.

**Beginner-Friendly Explanation**
Type inference is the compiler **figuring out the blank for you**. You write `Swap(a, b)` instead of `Swap<int>(a, b)`, and the compiler looks at `a` and `b`—if they are `int`s, it infers `T = int`.

---

### Purposes

- To **simplify call sites** by removing redundant type arguments.
- To **improve readability** of generic method calls.
- To enable **fluent APIs** and LINQ-style method chaining.
- To reduce the risk of specifying the wrong type argument.

---

### Syntax Rules and Structure

```
// Explicit type arguments
MethodName<int>(arg1, arg2);

// Inferred type arguments
MethodName(arg1, arg2);
```

**Component breakdown**

- `MethodName<int>` — Explicit type argument.
- `MethodName` — Type inference activated.

#### Syntax Rules

- Type inference occurs **before** overload resolution.
- The compiler infers type arguments from the **method arguments**.
- Type inference does **not** work from constraints or return types alone.
- Type inference fails if the method has no parameters.
- If inference fails for a generic method, that method is excluded from overload resolution.

#### Constraints and Limitations

- Cannot infer `T` from a `null` literal (the compiler cannot determine the type).
- Cannot infer `T` from a method group or anonymous function without additional context.
- Inference may fail for complex nested generic types, requiring explicit type arguments.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Basic Type Inference

```csharp
using System;

class Chooser
{
    static readonly Random rand = new();

    public static T Choose<T>(T first, T second) =>
        rand.Next(2) == 0 ? first : second;
}

class Program
{
    static void Main()
    {
        int i = Chooser.Choose(5, 213);          // inferred as Choose<int>
        string s = Chooser.Choose("apple", "banana"); // inferred as Choose<string>

        Console.WriteLine(i);   // 5 or 213
        Console.WriteLine(s);   // apple or banana
    }
}
```

**Expected Output (varies)**

```
5
apple
```

**Why this output occurs**

- The compiler infers `T = int` for the first call and `T = string` for the second based on the argument types.

---

#### Example 2: Type Inference with Multiple Parameters

```csharp
using System;

class Program
{
    static TResult Convert<TInput, TResult>(TInput input, Func<TInput, TResult> converter)
        => converter(input);

    static void Main()
    {
        var result = Convert(42, x => x.ToString());
        Console.WriteLine(result);   // 42
    }
}
```

**Expected Output**

```
42
```

**Why this output occurs**

- The compiler infers `TInput = int` from `42` and `TResult = string` from the lambda’s return type.

---

### Real-World Cases

- **LINQ** — `Where(x => x > 5)` infers `T` from the source sequence.
- **Utility methods** — `Max(3, 7)` infers `T = int`.
- **Factory methods** — `Activator.CreateInstance<T>()` uses type inference in generic wrappers.

---

### References

- Generic methods (C# programming guide) — Type inference - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/generics/generic-methods
- Expressions — C# language specification (type inference) - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/language-specification/expressions

---

## 3. Generic Method Invocation

### Definitions

**Core Definition**
Generic method invocation is the act of **calling a generic method**, either by supplying explicit type arguments (`Method<int>(...)`) or by relying on type inference (`Method(...)`).

**Technical Definition**
A generic method is invoked like any other method, but the caller may optionally supply type arguments in angle brackets after the method name. If type arguments are omitted, the compiler attempts type inference. The invocation is bound at **compile time**; the compiler generates a **closed constructed generic method** specialised for the inferred or supplied type arguments. Generic methods can be invoked on instances (instance methods) or on the type itself (static methods). The same rules for type inference apply to both static and instance methods.

**Beginner-Friendly Explanation**
Invoking a generic method is just **calling the method**. You can be explicit (`Swap<int>(a, b)`) or let the compiler infer (`Swap(a, b)`).

---

### Purposes

- To execute the generic algorithm with a **specific type**.
- To leverage **type inference** for concise calls.
- To obtain type-safe results without casting.
- To reuse the same method across many types.

---

### Syntax Rules and Structure

```
// Explicit type arguments
MethodName<int>(arg1, arg2);

// Inferred type arguments
MethodName(arg1, arg2);

// Generic method on an instance
instance.MethodName<T>(arg1);

// Generic method on a type
TypeName.MethodName<T>(arg1);
```

**Component breakdown**

- `MethodName<T>` — Invocation with explicit type arguments.
- `MethodName` — Invocation with type inference.
- `instance.MethodName` — Instance generic method.
- `TypeName.MethodName` — Static generic method.

#### Syntax Rules

- Type arguments, when supplied, are enclosed in angle brackets.
- Type inference is attempted when type arguments are omitted.
- The compiler generates specialised code for each closed generic method.

#### Constraints and Limitations

- If type inference fails and no explicit type arguments are supplied, the call is a compile-time error.
- Overload resolution considers only generic methods where type inference succeeded.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Explicit and Inferred Invocation

```csharp
using System;

class Program
{
    static T GetFirst<T>(T[] items) => items[0];

    static void Main()
    {
        int[] numbers = { 10, 20 };
        string[] words = { "Hello", "World" };

        // Explicit type argument
        int firstNumber = GetFirst<int>(numbers);
        Console.WriteLine(firstNumber);   // 10

        // Inferred type argument
        string firstWord = GetFirst(words);
        Console.WriteLine(firstWord);     // Hello
    }
}
```

**Expected Output**

```
10
Hello
```

**Why this output occurs**

- The explicit call specifies `int`; the inferred call deduces `string` from the `string[]` argument.

---

#### Example 2: Static vs. Instance Generic Methods

```csharp
using System;

public class Container
{
    public T Echo<T>(T value) => value;          // instance generic method
    public static T StaticEcho<T>(T value) => value; // static generic method
}

class Program
{
    static void Main()
    {
        var c = new Container();
        Console.WriteLine(c.Echo(42));                 // 42
        Console.WriteLine(Container.StaticEcho("Hi")); // Hi
    }
}
```

**Expected Output**

```
42
Hi
```

**Why this output occurs**

- The same rules for type inference apply to both instance and static generic methods.

---

### Real-World Cases

- **LINQ method syntax** — `numbers.Where(n => n > 5)` invokes a static generic extension method.
- **Collection methods** — `list.Find(item => item.IsActive)`.
- **Utility calls** — `Math.Max(3, 7)` (though `Math.Max` is overloaded, not generic).

---

### References

- Generic methods (C# programming guide) — Invocation - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/generics/generic-methods

---

## 4. Generic Method Constraints

### Definitions

**Core Definition**
Generic method constraints are **restrictions** applied to a method’s type parameters using the `where` clause, specifying the capabilities that a type argument must have.

**Technical Definition**
Constraints on generic methods are specified with the `where` clause, exactly as for generic types. They restrict the set of valid type arguments and declare the capabilities that `T` must have. Without constraints, `T` is treated as `object`, and only `object` members are available. The main constraint types are: `where T : class` (reference type), `where T : struct` (value type), `where T : notnull` (non-nullable), `where T : unmanaged` (unmanaged type), `where T : new()` (parameterless constructor), `where T : BaseClass`, `where T : IInterface`, and `where T : U` (type parameter relationship). Multiple constraints can be combined in a single `where` clause, with a specific ordering: primary constraint, base class, interfaces, and `new()` last. Constraints enable the method body to access members of `T`, such as calling `CompareTo` on an `IComparable<T>` constraint.

**Beginner-Friendly Explanation**
Generic method constraints are **rules about what `T` can be**. `where T : IComparable<T>` says “T must be comparable,” so you can call `CompareTo` inside the method.

---

### Purposes

- To **enable access to members** of `T` (methods, properties, operators).
- To ensure **type safety** by restricting valid type arguments.
- To **document requirements** clearly in the method signature.
- To support **generic algorithms** that require specific capabilities (comparison, construction, etc.).

---

### Syntax Rules and Structure (Constraint Matrix)

| Constraint | Meaning |
|---|---|
| `where T : class` | `T` must be a reference type. |
| `where T : struct` | `T` must be a non-nullable value type. |
| `where T : notnull` | `T` must be a non-nullable type. |
| `where T : unmanaged` | `T` must be an unmanaged type. |
| `where T : new()` | `T` must have a public parameterless constructor. |
| `where T : BaseClass` | `T` must derive from or be `BaseClass`. |
| `where T : IInterface` | `T` must implement `IInterface`. |
| `where T : U` | `T` must derive from or be `U`. |

**Component breakdown**

- `where` — The constraint keyword.
- `T` — The method-level type parameter.
- `:` — Separator.
- Constraint — The restriction.

#### Syntax Rules

- The `where` clause appears **after** the method signature, before the body.
- Multiple constraints for a single type parameter are comma-separated.
- `new()` must appear **last**.
- The base class constraint must appear **before** interface constraints.
- `struct` and `unmanaged` imply `new()` and cannot be combined with it.
- `class`, `struct`, `unmanaged`, `notnull`, and `default` are **mutually exclusive** (primary constraints).

#### Constraints and Limitations

- `System.Object`, `System.Array`, and `System.ValueType` cannot be used as base class constraints.
- Abstract types cannot be used with `new()` (they cannot be instantiated).
- Marker interfaces as the sole constraint are discouraged.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: `IComparable<T>` Constraint

```csharp
using System;

class Program
{
    static void SwapIfGreater<T>(ref T lhs, ref T rhs) where T : IComparable<T>
    {
        if (lhs.CompareTo(rhs) > 0)
        {
            T temp = lhs;
            lhs = rhs;
            rhs = temp;
        }
    }

    static void Main()
    {
        int a = 5, b = 3;
        SwapIfGreater(ref a, ref b);
        Console.WriteLine($"{a} {b}");   // 5 3 (no swap, a > b)

        int c = 3, d = 5;
        SwapIfGreater(ref c, ref d);
        Console.WriteLine($"{c} {d}");   // 5 3 (swapped)
    }
}
```

**Expected Output**

```
5 3
5 3
```

**Why this output occurs**

- The `IComparable<T>` constraint allows `CompareTo` to be called on `T`; the method swaps values only when the first is greater than the second.

---

#### Example 2: `new()` Constraint for Factory

```csharp
using System;

public class Factory<T> where T : new()
{
    public T Create() => new T();
}

public class Widget
{
    public string Name { get; set; } = "Default";
}

class Program
{
    static void Main()
    {
        var factory = new Factory<Widget>();
        var widget = factory.Create();
        Console.WriteLine(widget.Name);   // Default
    }
}
```

**Expected Output**

```
Default
```

**Why this output occurs**

- The `new()` constraint allows `new T()` inside `Create`; `Widget` has a parameterless constructor.

---

### Real-World Cases

- **`IComparable<T>`** — Sorting and comparison algorithms.
- **`new()`** — Generic factories and object creation.
- **`class` / `struct`** — Distinguishing reference and value types.
- **`unmanaged`** — Interop and low-level memory scenarios.

---

### References

- Constraints on type parameters — C# programming guide - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/generics/constraints-on-type-parameters
- where (generic type constraint) — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/where-generic-type-constraint
- Types — C# language specification (constraints) - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/language-specification/types

---

## 5. Generic Method Best Practices

### Definitions

**Core Definition**
Generic method best practices are the design guidelines that ensure generic methods are **usable, safe, performant, and maintainable**, while avoiding common pitfalls such as incorrect type inference, hidden type parameters, and misplaced static members.

**Technical Definition**
Best practices for generic methods derive from Microsoft’s guidance on generics and generic classes, which notes that a good rule is to apply the **maximum constraints possible** that will still let you handle the types you must handle. Key recommendations include: **use generic methods for algorithms that are independent of the containing type**; **apply constraints to enable member access** while keeping the API as flexible as possible; **avoid hiding class-level type parameters** with method-level type parameters (use distinct names like `U` instead of `T` to avoid CS0693); **prefer type inference** where it improves readability; **do not declare static members on generic types** (CA1000) because the type argument must be specified for every call; and **avoid excessive type parameters** (CA1005 recommends no more than two for externally visible types). Generic methods should **document their constraints** and **be tested with multiple type arguments** (value types, reference types, and constrained types).

**Beginner-Friendly Explanation**
Best practices for generic methods are simple: **use them for reusable algorithms**, **add only the constraints you need**, **do not confuse the compiler with hidden type parameters**, and **let type inference do its job**.

---

### Purposes

- To maximize the benefits of generics (reuse, safety, performance).
- To avoid confusing or hard-to-use generic APIs.
- To align with .NET design conventions and code analysis rules.
- To ensure generic methods are maintainable and testable.
- To choose the right generic mechanism for the scenario.

---

### Guidelines and Structure

#### General Guidelines

```
// USE generic methods for reusable algorithms
static T Max<T>(T a, T b) where T : IComparable<T> => ...;

// APPLY constraints to enable member access
static void SwapIfGreater<T>(ref T a, ref T b) where T : IComparable<T> { ... }

// AVOID hiding class-level type parameters (CS0693)
class GenericList<T>
{
    // void SampleMethod<T>() { } // CS0693
    void SampleMethod<U>() { }    // OK: distinct name
}

// PREFER type inference for concise calls
var first = GetFirst(numbers);   // not GetFirst<int>(numbers)

// DO NOT declare static members on generic types (CA1000)
// AVOID excessive type parameters (CA1005) — ≤ 2 for public APIs
```

**Component breakdown**

- **Generic methods for algorithms** — Independent of the containing type.
- **Constraints** — Maximum useful constraints, but no more.
- **No hidden type parameters** — Use distinct names to avoid CS0693.
- **Type inference** — Prefer inferred calls when readable.
- **CA1000 / CA1005** — Avoid static members on generic types and excessive type parameters.

#### Rules and Constraints

- **Do** use generic methods for reusable, type-agnostic algorithms.
- **Do** apply the maximum constraints that still allow the required types.
- **Do** use distinct type parameter names to avoid hiding (CS0693).
- **Do** prefer type inference for concise calls.
- **Do not** declare static members on generic types (CA1000).
- **Do not** use more than two type parameters on externally visible generic methods (CA1005).
- **Do not** rely on type inference for methods with no parameters.

---

### Annotated Code Examples and Expected Outputs

#### Example: Well-Designed Generic Method

```csharp
using System;

public static class CollectionHelper
{
    // Minimal, useful constraint; type inference works
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

- The `IComparable<T>` constraint enables `CompareTo` on `T`; the constraint is minimal, and type inference eliminates the need for explicit type arguments.

---

### Real-World Cases

- **LINQ operators** — `Where<T>`, `Select<T>`, `OrderBy<T>` are generic methods with appropriate constraints.
- **Utility libraries** — `Max<T>`, `Min<T>`, `Swap<T>` with `IComparable<T>` constraints.
- **Factory methods** — `Create<T>() where T : new()`.

---

### References

- Generic methods (C# programming guide) — Best practices - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/generics/generic-methods
- Generic classes (C# programming guide) — Constraints guidelines - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/generics/generic-classes
- CA1000: Do not declare static members on generic types — .NET code analysis - https://learn.microsoft.com/en-us/dotnet/fundamentals/code-analysis/quality-rules/ca1000
- CA1005: Avoid excessive parameters on generic types — .NET code analysis - https://learn.microsoft.com/en-us/dotnet/fundamentals/code-analysis/quality-rules/ca1005

---

## Deprecated, Unsafe, Implementation-Dependent, and Version-Specific Notes

| Feature | Status / Note |
|---|---|
| Generic methods | Introduced in **C# 2.0**; stable and foundational. |
| Type inference | Core since **C# 2.0**; stable. |
| CS0693 (type parameter hiding) | **Compiler warning**; avoid by using distinct names. |
| CA1000 (static members on generic types) | **Design rule**; flagged by analyzers, not a compile error. |
| CA1005 (excessive type parameters) | **Design rule**; recommends ≤ 2 type parameters. |
| Generic math (`INumber<T>`) | Introduced in **C# 11 / .NET 7**; version-specific. |
| Type inference with no parameters | **Not supported**; the compiler cannot infer `T`. |

---

## References (Consolidated)

- Generic methods (C# programming guide) — Microsoft Learn - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/generics/generic-methods
- Expressions — C# language specification (type inference) - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/language-specification/expressions
- Constraints on type parameters — C# programming guide - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/generics/constraints-on-type-parameters
- where (generic type constraint) — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/where-generic-type-constraint
- Generic classes (C# programming guide) — Guidelines - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/generics/generic-classes
- Generic types and methods — C# - Microsoft Learn (GitHub) - https://github.com/dotnet/docs/blob/main/docs/csharp/fundamentals/types/generics.md
- CA1000: Do not declare static members on generic types — .NET code analysis - https://learn.microsoft.com/en-us/dotnet/fundamentals/code-analysis/quality-rules/ca1000
- CA1005: Avoid excessive parameters on generic types — .NET code analysis - https://learn.microsoft.com/en-us/dotnet/fundamentals/code-analysis/quality-rules/ca1005

---