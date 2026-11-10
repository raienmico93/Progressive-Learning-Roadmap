# C# Method Overloading — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
Method overloading is the practice of declaring **multiple methods with the same name** within the same type, provided each method has a **unique signature** — a distinct combination of parameter types, parameter count, or parameter-passing modes. Overloading lets a single conceptual operation be expressed under one recognisable name while accommodating different input shapes. The C# compiler **selects the best overload** at each call site through a process called **overload resolution**; when no single best match exists, the call is **ambiguous** and a compile-time error occurs.

**Technical Definition**
The C# language specification states that the signature of a method consists of the **name of the method**, the **number of type parameters**, and the **type and parameter-passing mode of each of its parameters**, considered in order from left to right. The signature specifically **does not include** the return type, parameter names, type parameter names, type parameter constraints, the `params` or `this` parameter modifiers, or whether parameters are required or optional. Two signatures are considered the same if they have the same name, number of type parameters, number of parameters, and parameter-passing modes, and an identity conversion exists between the types of their corresponding parameters. Overloading permits a class, struct, or interface to declare multiple methods with the same name, provided their signatures are unique within that type. Although `in`, `out`, and `ref` modifiers are part of a signature, members declared in a single type **cannot differ in signature solely by `in`, `out`, and `ref`**; a compile-time error occurs if two members would have the same signature after all `out` and `in` modifiers were changed to `ref`. The types `object` and `dynamic` are **not distinguished** when comparing signatures.

**Beginner-Friendly Explanation**
Method overloading is like having **several recipes with the same name** — say, “Make a sandwich.” One recipe uses white bread, another uses rye, and a third uses a baguette. When you say “Make a sandwich,” the cook (the compiler) looks at what you handed them (the arguments) and picks the recipe that fits best. If two recipes fit **equally well**, the cook refuses to guess — that is an **ambiguous call**. The key rule is that the recipes must be **distinguishable by their ingredients** (parameter types and count), not by what they produce (return type) or by optional extras.

---

### Key Characteristics

- **Same name, different signature** — Overloads share a method name but differ in parameter types, parameter count, or parameter-passing modes.
- **Return type is excluded** — You cannot overload solely by return type; two methods differing only in return type have the same signature and cause a compile-time error.
- **`in`/`out`/`ref` restriction** — Although these modifiers are part of a signature, a single type cannot contain overloads that differ **only** in these modifiers.
- **`params` is excluded** — The `params` modifier is **not** part of the signature; two overloads that differ only by `params` are not valid overloads.
- **`object` and `dynamic` collide** — Signatures that differ only by replacing `object` with `dynamic` (or vice versa) are considered the same signature.
- **Optional parameters affect resolution** — Overloads that differ only by optional parameters can lead to subtle resolution behaviour; omitted arguments are ignored during resolution, and if two candidates are equally good, preference goes to the candidate **without** omitted optional parameters.

---

### Prerequisites

- Understanding of C# method declarations, parameters, and return types.
- Familiarity with the distinction between value types and reference types.
- Knowledge of implicit conversions (widening, reference, boxing) and their role in resolution.
- Awareness of generics, `params`, optional parameters, and named arguments.
- Basic understanding of compile-time errors and compiler diagnostics (CS0121 for ambiguity).

---

### Related Programming Areas

- **API design** — Overloading shapes the ergonomics and discoverability of public APIs.
- **Optional parameters and named arguments** — These features interact closely with overload resolution and can introduce ambiguity if misused.
- **Generic methods** — Type inference and generic arity are part of the signature and affect overload resolution.
- **Versioning** — Overloads and default values have different binary-compatibility profiles; changing a default value or replacing an overload can break callers.
- **C# 14 span overloads** — New span conversions change which overloads are applicable in certain scenarios.

---

### Core Concepts / Features

1. Method Overloading (Overarching Concept)
2. Overloading Rules (Signatures and Constraints)
3. Overload Resolution (How the Compiler Chooses)
4. Ambiguity (When No Best Match Exists)
5. Overloading Best Practices

Each concept follows a uniform structure: definitions, purposes, syntax rules and structure, annotated examples with expected outputs, real-world cases, and references.

---

## 1. Method Overloading (Overarching Concept)

### Definitions

**Core Definition**
Method overloading is the declaration of multiple methods with the same name but distinct signatures within a single type, enabling a unified API surface for related operations.

**Technical Definition**
Overloading of methods permits a class, struct, or interface to declare multiple methods with the same name, provided their signatures are unique within that class, struct, or interface. The signature is the enabling mechanism for overloading; it comprises the method name, type parameter count, and the types and parameter-passing modes of the parameters. Overloading is resolved at **binding time**: at each invocation, the compiler applies overload resolution to the set of candidate methods to select a single best method; if no single best method can be identified, the invocation is ambiguous and a binding-time error occurs.

**Beginner-Friendly Explanation**
Overloading lets you use **one name for a family of related actions**. `Console.WriteLine` is the classic example: it has dozens of overloads for `int`, `string`, `double`, `object`, and many other types. You always write `Console.WriteLine(...)`, and the compiler picks the right version based on what you pass.

---

### Purposes

- To provide a **cohesive API** for semantically similar operations on different input types.
- To avoid inventing arbitrary name variations (`PrintInt`, `PrintString`, `PrintDouble`) for the same concept.
- To support **convenience overloads** that call a more detailed overload with sensible defaults.
- To enable **type-specific optimisations** while keeping a single call-site syntax.
- To interoperate cleanly with features like `params`, optional parameters, and generics.

---

### Syntax Rules and Structure

```
// Overloads share a name but differ in signature
returnType MethodName(type1 param1) { ... }
returnType MethodName(type2 param1) { ... }
returnType MethodName(type1 param1, type2 param2) { ... }
```

**Component breakdown**

- **Method name** — Identical across all overloads.
- **Parameter types** — Must differ between overloads (or parameter count, or passing mode).
- **Return type** — May differ but does **not** distinguish overloads.

#### Syntax Rules

- Overloads must have **unique signatures** within the same type.
- The signature includes parameter types and passing modes, but **excludes** return type, parameter names, `params`, and optionality.
- `ref`, `out`, and `in` are part of the signature, but overloads cannot differ **solely** by these modifiers.
- `object` and `dynamic` are indistinguishable for signature purposes.

#### Constraints and Limitations

- Cannot overload solely by return type.
- Cannot overload solely by `params`.
- Cannot overload solely by optionality (required vs. optional).
- Cannot overload solely by parameter names.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: A Family of `Add` Overloads

```csharp
using System;

class Calculator
{
    public int Add(int a, int b) => a + b;
    public double Add(double a, double b) => a + b;
    public int Add(int a, int b, int c) => a + b + c;
}

class Program
{
    static void Main()
    {
        var calc = new Calculator();
        Console.WriteLine(calc.Add(1, 2));          // 3 (int overload)
        Console.WriteLine(calc.Add(1.5, 2.5));      // 4 (double overload)
        Console.WriteLine(calc.Add(1, 2, 3));       // 6 (three-parameter overload)
    }
}
```

**Expected Output**

```
3
4
6
```

**Why this output occurs**

- The compiler selects the overload whose parameter types best match the argument types. The `int` literal pair selects the `int` overload; the `double` pair selects the `double` overload; three arguments select the three-parameter overload.

---

#### Example 2: Return Type Alone Cannot Overload

```csharp
using System;

class Program
{
    // COMPILE ERROR CS0111: Type 'Program' already defines a member called 'Get' with the same parameter types
    // static int Get() => 1;
    // static string Get() => "one";

    static void Main() { }
}
```

**Expected Output (conceptual)**

```
Compile-time error: duplicate member 'Get'
```

**Why this output occurs**

- The return type is **not part of the signature**. Both methods have the same signature (`Get()`), so the compiler rejects the second declaration.

---

### Real-World Cases

- **`Console.WriteLine`** — Overloads for `bool`, `char`, `int`, `double`, `string`, `object`, and more.
- **`Math.Max` / `Math.Min`** — Overloads for `int`, `long`, `double`, `decimal`, etc.
- **`string.IndexOf`** — Overloads for `char`, `string`, start index, `StringComparison`, etc..
- **LINQ extension methods** — Overloads that accept different selector or comparer delegates.

---

### References

- Basic concepts — C# language specification (signatures and overloading) - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/language-specification/basic-concepts#76-signatures-and-overloading
- Method Usage Guidelines — Microsoft Learn (legacy) - https://learn.microsoft.com/en-us/previous-versions/dotnet/netframework-1.1/xxfyae0c(v=vs.71)
- Overview of methods — C# — Microsoft Learn - https://learn.microsoft.com/en-us/dotnet/csharp/methods

---

## 2. Overloading Rules (Signatures and Constraints)

### Definitions

**Core Definition**
Overloading rules are the syntactic and semantic constraints that determine whether two method declarations can coexist as overloads within the same type.

**Technical Definition**
The signature of a method is the enabling mechanism for overloading. It comprises the method name, the number of type parameters, and the type and parameter-passing mode of each parameter, in order. The signature excludes the return type, parameter names, type parameter names, type parameter constraints, the `params` or `this` modifiers, and optionality. Two signatures are identical if they have the same name, type parameter count, parameter count, and parameter-passing modes, and an identity conversion exists between corresponding parameter types. Although `in`, `out`, and `ref` are part of the signature, members cannot differ **solely** by these modifiers. `object` and `dynamic` are treated as the same type for signature purposes.

**Beginner-Friendly Explanation**
The **signature** is the method’s “identity card” for overloading. It says: “I am `Add(int, int)`.” Two methods with the same name and the same signature **cannot coexist** — the compiler sees them as duplicates. The return type, parameter names, and `params` keyword **do not** appear on the identity card, so they cannot be used to distinguish overloads.

---

### Purposes

- To prevent duplicate method declarations within a type.
- To define which method characteristics matter for overloading.
- To ensure that the compiler can unambiguously identify a method from a call site.
- To support interoperability across languages and runtimes.

---

### Syntax Rules and Structure (What Counts)

| Component | Part of signature? |
|---|---|
| Method name | ✔️ Yes |
| Number of type parameters | ✔️ Yes |
| Parameter types | ✔️ Yes |
| `ref` / `out` / `in` modifiers | ✔️ Yes (but cannot be the sole difference) |
| Return type | ❌ No |
| Parameter names | ❌ No |
| `params` modifier | ❌ No |
| Optionality (required vs. optional) | ❌ No |
| `object` vs. `dynamic` | ❌ No — treated as identical |

**Component breakdown**

- **Name** — The method’s identifier.
- **Type parameters** — Generic arity.
- **Parameters** — Types and passing modes (`ref`, `out`, `in`).
- **Excluded** — Return type, names, `params`, optionality.

#### Syntax Rules

- Overloads must have **unique signatures** within the same type.
- Two methods differing only in return type are **invalid**.
- Two methods differing only in `params` are **invalid**.
- Two methods differing only by `ref` vs. `out` vs. `in` are **invalid** in the same type.

#### Constraints and Limitations

- You cannot overload a method by return type alone.
- You cannot overload solely by parameter names.
- You cannot overload solely by optionality or `params`.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Valid Overloads by Parameter Type and Count

```csharp
using System;

class Formatter
{
    public void Print(int value) => Console.WriteLine($"int: {value}");
    public void Print(string value) => Console.WriteLine($"string: {value}");
    public void Print(int a, int b) => Console.WriteLine($"two ints: {a}, {b}");
}

class Program
{
    static void Main()
    {
        var f = new Formatter();
        f.Print(42);          // int: 42
        f.Print("hello");    // string: hello
        f.Print(1, 2);       // two ints: 1, 2
    }
}
```

**Expected Output**

```
int: 42
string: hello
two ints: 1, 2
```

**Why this output occurs**

- The parameter types and counts differ, so the signatures are unique and overloading is valid.

---

#### Example 2: Invalid Overload by Return Type Only

```csharp
using System;

class Program
{
    // COMPILE ERROR CS0111: duplicate member 'Get' with same parameter types
    // static int Get() => 1;
    // static string Get() => "one";

    static void Main() { }
}
```

**Expected Output (conceptual)**

```
Compile-time error: the type already defines a member 'Get' with the same parameter types
```

**Why this output occurs**

- The return type is excluded from the signature, so both methods share the same signature.

---

### Real-World Cases

- **Framework APIs** — Designers use distinct parameter types to create overload families.
- **Versioning** — Adding an overload with a new parameter type is a common non-breaking change.
- **Interop** — Signature rules ensure methods can be consumed from other CLS-compliant languages.

---

### References

- Basic concepts — C# language specification (signatures) - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/language-specification/basic-concepts#76-signatures-and-overloading
- Function overloading based on different return types — Microsoft Learn Q&A - https://learn.microsoft.com/en-us/answers/questions/1251249/function-overloading-based-on-different-return-typ

---

## 3. Overload Resolution (How the Compiler Chooses)

### Definitions

**Core Definition**
Overload resolution is the compile-time process by which the C# compiler selects the **single best method** from a set of candidate overloads for a given invocation.

**Technical Definition**
Overload resolution identifies the **best method** from the set of candidate methods using the rules of the C# language specification. In practice, the rules are intended to find the overload that is **“closest”** to the actual arguments supplied. If there is a method whose parameter types exactly match the argument types, that method is obviously the closest. Barring that, one method is closer than another if all of its parameter types are **narrower than (or the same as)** the parameter types of the other method; if neither is narrower, there is no way to determine which is closer. Overload resolution **does not take into account the expected return type** of the method. The resolution process eliminates candidates that are inaccessible or inapplicable, applies type inference for generic methods, and considers the delegate relaxation levels of lambda expressions. If a single best method cannot be identified, the invocation is **ambiguous**, and a binding-time error occurs.

**Beginner-Friendly Explanation**
Overload resolution is the compiler’s **matchmaking process**. It looks at the arguments you supplied and asks: “Which overload is the **best fit**?” An **exact match** always wins. If no exact match exists, it prefers the method whose parameter types are **more specific** (narrower) than the others. If two methods are equally good, the compiler refuses to guess — you get an **ambiguity error** and must disambiguate with a cast, named argument, or different call.

---

### Purposes

- To select a single method from an overload set at each call site.
- To prefer exact matches over conversions.
- To prefer more specific parameter types when multiple candidates are applicable.
- To handle generic type inference and lambda conversions consistently.
- To produce a compile-time error when no unique best method exists.

---

### Syntax Rules and Structure (Resolution Process)

The C# specification describes a **multi-step elimination process**:

1. **Type inference** — For generic methods without explicit type arguments, infer type arguments; eliminate methods where inference fails.
2. **Applicability** — Eliminate members that are inaccessible or inapplicable to the argument list.
3. **Delegate relaxation** — For lambda expressions and method groups, compare delegate relaxation levels and eliminate candidates with worse levels.
4. **Better conversion** — Among remaining candidates, apply rules for better conversions (e.g., `int` to `long` vs. `int` to `object`).
5. **Optional parameters** — Omitted arguments for optional parameters are ignored during resolution; if two candidates are equally good, preference goes to the candidate **without** omitted optional parameters.
6. **Single best method** — If one method remains, it wins; otherwise, the invocation is **ambiguous**.

**Component breakdown**

- **Exact match** — Parameter types exactly match argument types → wins.
- **Narrower parameter types** — A method whose parameter types are all narrower (more specific) than another’s is preferred.
- **Return type ignored** — The expected return type does not influence resolution.
- **Optional parameters** — Candidates without omitted optional parameters are preferred when otherwise tied.

#### Syntax Rules

- The compiler **never** selects an overload based on return type.
- Named arguments can change which parameters are considered, but do not change the fundamental resolution rules.
- Generic type inference is performed before applicability checks.

#### Constraints and Limitations

- Overload resolution cannot resolve calls where no unique best method exists.
- Implicit conversions (e.g., `int` to `long`) may make multiple overloads applicable, but exact matches are preferred.
- `object` and `dynamic` are treated identically for signature purposes, so they do not distinguish overloads.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Exact Match Wins

```csharp
using System;

class Printer
{
    public void Print(object value) => Console.WriteLine("object overload");
    public void Print(int value) => Console.WriteLine("int overload");
}

class Program
{
    static void Main()
    {
        var p = new Printer();
        p.Print(42);          // int overload (exact match)
        p.Print((object)42);  // object overload (explicit cast)
    }
}
```

**Expected Output**

```
int overload
object overload
```

**Why this output occurs**

- `42` is an `int`, so the `Print(int)` overload is an **exact match** and wins.
- Casting to `object` forces the `Print(object)` overload.

---

#### Example 2: Narrower Parameter Type Preferred

```csharp
using System;

class Selector
{
    public void Choose(long value) => Console.WriteLine("long overload");
    public void Choose(int value) => Console.WriteLine("int overload");
}

class Program
{
    static void Main()
    {
        var s = new Selector();
        s.Choose(42);   // int overload (int is narrower than long)
    }
}
```

**Expected Output**

```
int overload
```

**Why this output occurs**

- Both overloads are applicable (`int` implicitly converts to `long`), but `int` is **narrower** than `long`, so the `int` overload is closer.

---

#### Example 3: Optional Parameters and Tie-Breaking

```csharp
using System;

class Greeter
{
    public void Greet(string name, string greeting = "Hello")
        => Console.WriteLine($"{greeting}, {name}!");

    public void Greet(string name)
        => Console.WriteLine($"Hi, {name}!");
}

class Program
{
    static void Main()
    {
        var g = new Greeter();
        g.Greet("Alice");   // Hi, Alice! (overload without optional parameter wins)
    }
}
```

**Expected Output**

```
Hi, Alice!
```

**Why this output occurs**

- Both overloads are applicable for a single argument. The `Greet(string)` overload has **no omitted optional parameters**, so it is preferred over the overload that omits `greeting`.

---

### Real-World Cases

- **Framework evolution** — New overloads are added with more specific parameter types; the compiler automatically selects the newer overload for matching calls.
- **LINQ** — Overload resolution selects the correct `Where`, `Select`, or `OrderBy` overload based on the delegate type and arguments.
- **C# 14 span overloads** — New span conversions make span-based overloads applicable in more scenarios; non-span overloads may need explicit casts to remain bound.

---

### References

- Expressions — C# language specification (overload resolution) - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/language-specification/expressions#1264-overload-resolution
- Overloaded Method Resolution — Microsoft Learn (legacy) - https://learn.microsoft.com/en-us/dotnet/visual-basic/reference/language-specification/overload-resolution
- Overview of methods — Optional parameters and overload resolution — Microsoft Learn - https://learn.microsoft.com/en-us/dotnet/csharp/methods#optional-parameters-and-arguments
- Breaking change: C# 14 overload resolution with span parameters — .NET - https://learn.microsoft.com/en-us/dotnet/core/compatibility/core-libraries/10.0/csharp-overload-resolution

---

## 4. Ambiguity (When No Best Match Exists)

### Definitions

**Core Definition**
An ambiguous invocation occurs when the compiler cannot identify a **single best method** among the candidates; the call is then rejected with a compile-time error (CS0121).

**Technical Definition**
If a single best method cannot be identified, the method invocation is **ambiguous**, and a binding-time error occurs. Ambiguity arises when multiple overloads are equally applicable and the better-conversion rules cannot establish a unique winner. Common causes include: overloads distinguished only by optional parameters, implicit conversions that make two overloads equally viable, and generic overloads whose type inference produces equally specific candidates. Since C# 13 / .NET 9, the `OverloadResolutionPriorityAttribute` allows library authors to set an overload’s priority, guiding the compiler when calls would otherwise be ambiguous.

**Beginner-Friendly Explanation**
Ambiguity is the compiler saying: **“I cannot decide which method you mean — please be more specific.”** It happens when two overloads fit your arguments **equally well**. You can fix it by casting an argument to the exact type you want, using a named argument, or (for library authors) applying `OverloadResolutionPriority`.

---

### Purposes

- To prevent the compiler from silently choosing an unintended overload.
- To force callers to disambiguate when overloads are too similar.
- To protect against subtle bugs caused by optional-parameter overloading.
- To allow library authors to guide resolution with `OverloadResolutionPriority`.

---

### Syntax Rules and Structure (Common Ambiguity Scenarios)

| Scenario | Example | Resolution |
|---|---|---|
| **Optional-parameter ambiguity** | `M(int x = 0)` and `M()` | Exact match wins if one exists; otherwise ambiguous. |
| **Implicit conversion tie** | `M(long)` and `M(double)` called with `int` | Both conversions are equally good → ambiguous. |
| **`params` vs. array** | `M(params int[])` and `M(int[])` | Normal form of `params` is preferred. |
| **Generic inference tie** | `M<T>(T)` and `M(int)` called with `int` | Non-generic exact match wins. |
| **`object` vs. `dynamic`** | `M(object)` and `M(dynamic)` | Treated as identical → duplicate signature error. |

**Component breakdown**

- **CS0121** — The compiler diagnostic for ambiguous calls.
- **Disambiguation techniques** — Explicit casts, named arguments, or `OverloadResolutionPriority`.

#### Syntax Rules

- An ambiguous call is a **compile-time error**; the compiler does not choose arbitrarily.
- `OverloadResolutionPriority` can set a priority from `int.MinValue` to `int.MaxValue`; higher values are preferred.
- The attribute is intended for **library authors** to guide binding, not for general application code.

#### Constraints and Limitations

- Ambiguity cannot be resolved by return type.
- Optional parameters can create ambiguity if overloads are not carefully designed.
- `OverloadResolutionPriority` requires **C# 13 / .NET 9** or later.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Optional-Parameter Ambiguity

```csharp
using System;

class Ambiguous
{
    public void Test(string id, string id2 = "") { }
    public void Test(string id2 = "") { }
}

class Program
{
    static void Main()
    {
        // var a = new Ambiguous();
        // a.Test(""); // COMPILE ERROR CS0121: ambiguous call
    }
}
```

**Expected Output (conceptual)**

```
Compile-time error CS0121: The call is ambiguous between 'Ambiguous.Test(string, string)' and 'Ambiguous.Test(string)'
```

**Why this output occurs**

- Both overloads are applicable for a single string argument; neither is an exact match for the parameter list **including** omitted optional parameters, so the compiler cannot choose.

---

#### Example 2: Disambiguating with a Cast

```csharp
using System;

class Overloads
{
    public void M(long value) => Console.WriteLine("long");
    public void M(double value) => Console.WriteLine("double");
}

class Program
{
    static void Main()
    {
        var o = new Overloads();
        // o.M(42); // ambiguous: int converts equally to long and double
        o.M((long)42);   // long
        o.M((double)42); // double
    }
}
```

**Expected Output**

```
long
double
```

**Why this output occurs**

- The explicit cast selects the intended overload, removing the ambiguity.

---

#### Example 3: `OverloadResolutionPriority` (C# 13 / .NET 9)

```csharp
using System;
using System.Runtime.CompilerServices;

public static class PriorityExample
{
    [OverloadResolutionPriority(3)]
    public static void Method(double d = 2) => Console.WriteLine("Method A");

    [OverloadResolutionPriority(2)]
    public static void Method(int i = 1) => Console.WriteLine("Method B");

    [OverloadResolutionPriority(1)]
    public static void Method(char c = ' ') => Console.WriteLine("Method C");
}

class Program
{
    static void Main()
    {
        PriorityExample.Method();   // Method A (highest priority)
    }
}
```

**Expected Output**

```
Method A
```

**Why this output occurs**

- Without priorities, this call would be ambiguous. The `OverloadResolutionPriority` attribute guides the compiler to select the overload with the highest priority.

---

### Real-World Cases

- **Library evolution** — Adding a `params` overload alongside an array overload can create ambiguity for callers.
- **Optional parameters** — Replacing distinct overloads with optional parameters can silently change which overload is selected, causing bugs.
- **C# 14 span changes** — New span overloads can bind in scenarios where non-span overloads previously bound, requiring explicit casts for `Expression` trees.

---

### References

- Expressions — C# language specification (overload resolution) - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/language-specification/expressions#1264-overload-resolution
- The attribute OverloadResolutionPriority in .NET 9 and C# 13 — NDepend Blog - https://blog.ndepend.com/the-attribute-overloadresolutionpriority/
- Breaking change: C# 14 overload resolution with span parameters — .NET - https://learn.microsoft.com/en-us/dotnet/core/compatibility/core-libraries/10.0/csharp-overload-resolution
- Question about method overloads with optional parameters ambiguity in C#/.NET — GitHub Discussion - https://github.com/dotnet/runtime/discussions/116056

---

## 5. Overloading Best Practices

### Definitions

**Core Definition**
Overloading best practices are the design guidelines that ensure overloaded methods are **intuitive, unambiguous, and maintainable** across API evolution.

**Technical Definition**
Best practices for overloading derive from Microsoft’s Method Usage Guidelines and the .NET Framework Design Guidelines. Key recommendations include: **use method overloading to provide different methods that do semantically the same thing**; **prefer overloading over default arguments** because default arguments do not version well and are not CLS-compliant; **use a consistent ordering and naming pattern for parameters** across overloads; **provide a complete set of overloads** so callers always have an exact match and avoid ambiguity; and **use shorter overloads with fewer parameters** to create an optimal path that does not rely on optional parameters.

**Beginner-Friendly Explanation**
Best practices are the habits that keep overloads **predictable**. Only overload methods that **do the same kind of thing**. Give overloads a **consistent parameter order**. Provide **all the common combinations** so callers rarely hit ambiguity. And be careful with **optional parameters** — they can silently change which overload is chosen.

---

### Purposes

- To keep overloaded APIs semantically coherent.
- To avoid ambiguity and surprising resolution outcomes.
- To ensure consistent behaviour across overloads.
- To support clean API versioning.
- To align with CLS compliance and cross-language interoperability.

---

### Guidelines and Structure

#### General Guidelines

```
// Use overloading for semantically identical operations
public void Draw(Circle circle) { ... }
public void Draw(Square square) { ... }

// Prefer overloads over default arguments
public void Log(string message) { ... }
public void Log(string message, LogLevel level) { ... }

// Use consistent parameter order and naming
public void Configure(string name) { ... }
public void Configure(string name, int timeout) { ... }

// Provide a complete set of overloads to avoid ambiguity
public void Execute() { ... }
public void Execute(string arg) { ... }
public void Execute(string arg, int count) { ... }

// Use shorter overloads with fewer parameters
public void Process() { Process(default); }
public void Process(int value) { ... }
```

**Component breakdown**

- **Semantic coherence** — Overloads should represent the **same conceptual operation** on different inputs.
- **Overloads over defaults** — Default arguments do not version well; distinct overloads provide clearer contracts.
- **Consistent ordering** — Shared parameters should appear in the same order across overloads.
- **Complete overload sets** — Providing all common combinations reduces ambiguity.
- **Shorter overloads** — Provide parameterless or minimal overloads that delegate to the full implementation.

#### Rules and Constraints

- **Do** use overloading for semantically identical operations.
- **Do** prefer explicit overloads over optional parameters for public APIs.
- **Do** keep parameter order consistent across overloads.
- **Do** provide a complete set of overloads covering common call patterns.
- **Do not** overload methods that do **different things**.
- **Do not** rely solely on optional parameters to distinguish overloads; this can cause ambiguity.
- **Do not** use overloads that differ only by return type (invalid).

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Complete Overload Set with Consistent Ordering

```csharp
using System;

class FileSaver
{
    public void Save(string path) => Save(path, overwrite: false);
    public void Save(string path, bool overwrite) => Save(path, overwrite, encoding: "UTF-8");
    public void Save(string path, bool overwrite, string encoding)
        => Console.WriteLine($"Saving {path} (overwrite={overwrite}, encoding={encoding})");
}

class Program
{
    static void Main()
    {
        var saver = new FileSaver();
        saver.Save("file.txt");                      // overwrite=False, encoding=UTF-8
        saver.Save("file.txt", true);                // overwrite=True, encoding=UTF-8
    }
}
```

**Expected Output**

```
Saving file.txt (overwrite=False, encoding=UTF-8)
Saving file.txt (overwrite=True, encoding=UTF-8)
```

**Why this output occurs**

- Each overload delegates to the next more detailed overload, providing a complete and consistent set.

---

#### Example 2: Avoiding Optional-Parameter Ambiguity

```csharp
using System;

class Logger
{
    // Preferred: explicit overloads with clear semantics
    public void Log(string message) => Log(message, LogLevel.Info);
    public void Log(string message, LogLevel level)
        => Console.WriteLine($"[{level}] {message}");
}

enum LogLevel { Info, Warning, Error }

class Program
{
    static void Main()
    {
        var logger = new Logger();
        logger.Log("Started");                 // [Info] Started
        logger.Log("Failed", LogLevel.Error);  // [Error] Failed
    }
}
```

**Expected Output**

```
[Info] Started
[Error] Failed
```

**Why this output occurs**

- Explicit overloads avoid the ambiguity that optional parameters could introduce and provide a clear default path.

---

### Real-World Cases

- **.NET Framework** — `String.IndexOf` provides a complete set of overloads for different search scenarios.
- **Logging libraries** — `Log(string)` and `Log(string, LogLevel)` provide a simple default and a detailed path.
- **UI frameworks** — `Draw(Shape)` overloads for different shape types.

---

### References

- Method Usage Guidelines — Microsoft Learn (legacy) - https://learn.microsoft.com/en-us/previous-versions/dotnet/netframework-1.1/xxfyae0c(v=vs.71)
- Overview of methods — C# — Microsoft Learn - https://learn.microsoft.com/en-us/dotnet/csharp/methods
- Basic concepts — C# language specification (signatures) - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/language-specification/basic-concepts#76-signatures-and-overloading
- The attribute OverloadResolutionPriority in .NET 9 and C# 13 — NDepend Blog - https://blog.ndepend.com/the-attribute-overloadresolutionpriority/

---

## Deprecated, Unsafe, Implementation-Dependent, and Version-Specific Notes

| Feature | Status / Note |
|---------|---------------|
| `OverloadResolutionPriorityAttribute` | Introduced in **C# 13 / .NET 9**; library authors can set overload priority to resolve ambiguity. |
| `params` with collection types | Introduced in **C# 13**; earlier versions require arrays. |
| C# 14 span overload resolution | **Behavioral change** in .NET 10; span-based overloads bind in more scenarios. |
| `object` vs. `dynamic` overloads | **Not distinguishable** — treated as identical signatures. |
| `in` / `out` / `ref`-only overloads | **Not allowed** in the same type; must also differ in parameter types. |
| Optional-parameter overloading | **Risky**; can lead to ambiguity or silent behaviour changes. |
| Default arguments in public APIs | **Not CLS-compliant**; prefer explicit overloads. |

---

## References (Consolidated)

- Basic concepts — C# language specification (signatures and overloading) - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/language-specification/basic-concepts#76-signatures-and-overloading
- Expressions — C# language specification (overload resolution) - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/language-specification/expressions#1264-overload-resolution
- Method Usage Guidelines — Microsoft Learn (legacy) - https://learn.microsoft.com/en-us/previous-versions/dotnet/netframework-1.1/xxfyae0c(v=vs.71)
- Overview of methods — C# — Microsoft Learn - https://learn.microsoft.com/en-us/dotnet/csharp/methods
- Overloaded Method Resolution — Microsoft Learn (legacy) - https://learn.microsoft.com/en-us/dotnet/visual-basic/reference/language-specification/overload-resolution
- Breaking change: C# 14 overload resolution with span parameters — .NET - https://learn.microsoft.com/en-us/dotnet/core/compatibility/core-libraries/10.0/csharp-overload-resolution
- The attribute OverloadResolutionPriority in .NET 9 and C# 13 — NDepend Blog - https://blog.ndepend.com/the-attribute-overloadresolutionpriority/
- Question about method overloads with optional parameters ambiguity in C#/.NET — GitHub Discussion - https://github.com/dotnet/runtime/discussions/116056
- Function overloading based on different return types — Microsoft Learn Q&A - https://learn.microsoft.com/en-us/answers/questions/1251249/function-overloading-based-on-different-return-typ

---