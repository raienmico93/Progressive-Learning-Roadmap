# C# Method Fundamentals — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
A method in C# is a named block of executable code that performs a specific task. Methods are the fundamental unit of behaviour in C# programs: every executed instruction runs inside some method, and methods are how you organise, reuse, and abstract logic. A method can accept inputs (parameters), return a result (or `void`), and be invoked (called) from other code. C# offers a rich family of method forms—static, instance, virtual, abstract, sealed, extension, local, and expression-bodied—each serving a distinct architectural purpose.

**Technical Definition**
In C#, a method is a code block that contains a series of statements. A program causes the statements to be executed by **calling** the method and specifying any required **method arguments**. In C#, every executed instruction is performed in the context of a method. Methods are declared in a `class`, `record`, or `struct` by specifying an optional access level (default `private`), optional modifiers such as `abstract` or `sealed`, the return value (or `void` if none), the method name, and any method parameters enclosed in parentheses and separated by commas. A method’s **signature** consists of its name, type parameter count, and parameter list (including `ref` and `out` modifiers); the return type is **not** part of the signature for overload resolution but **is** part of the signature when determining delegate compatibility. Methods are either **instance** methods (invoked on an object instance and operating on that instance’s data) or **static** methods (invoked through the type name and not operating on instance data). **Overloaded** methods share a name but differ in parameter lists; **overridden** methods replace a base class’s `virtual` or `abstract` method and must have the same signature. A `sealed` method overrides an inherited virtual method with the same signature and must also be marked `override`; the `sealed` modifier prevents further overriding in derived classes. An `abstract` method is implicitly virtual, has no body (only a semicolon), and must be implemented by non-abstract derived classes; abstract methods are permitted only in abstract classes and interfaces. **Extension methods** are static methods invoked using instance-method syntax; they extend existing types without modifying them. **Local functions** are methods nested inside another member and callable only from their containing member. **Expression-bodied methods** use the `=>` token to provide a concise single-expression implementation.

**Beginner-Friendly Explanation**
A method is a **named recipe** in your program. You give it a name, tell it what ingredients it needs (parameters), and describe the steps (the body). When you want to make that recipe, you **call** the method and give it the ingredients (arguments). Some recipes produce a dish (return a value); others just do something and produce nothing (`void`). Methods are how you avoid repeating yourself: write the recipe once, call it as many times as you need. C# offers different **kinds** of methods: **static** methods belong to the type itself (like a utility function), **instance** methods belong to an object (like a `Car` starting its engine), **virtual** methods can be customised by subclasses, **abstract** methods must be customised by subclasses, and **sealed** methods lock down a customisation so no further subclass can change it. **Extension methods** let you add new tricks to existing types without changing them, **local functions** are little helper methods that live inside a bigger method, and **expression-bodied methods** are ultra-short methods written as a single expression.

---

### Key Characteristics

- **Every instruction lives in a method** — C# programs are composed entirely of methods.
- **Signature vs. return type** — Return type is excluded from the signature for overloading but included for delegate compatibility.
- **Instance vs. static** — Instance methods operate on object state; static methods operate only on their parameters and type-level state.
- **Overloading vs. overriding** — Overloading creates multiple methods with the same name but different parameter lists; overriding replaces a base class’s virtual/abstract method with the same signature.
- **`virtual` → `override` → `sealed override`** — A virtual method can be overridden; an overridden method can be sealed to prevent further overriding.
- **`abstract` requires implementation** — Abstract methods have no body and force derived classes to implement them.
- **Extension methods** — Static methods that appear as instance methods, extending types without modification.
- **Local functions** — Nested methods callable only from their containing member.
- **Expression-bodied methods** — Concise `=>` syntax for single-expression implementations.

---

### Prerequisites

- Basic understanding of C# program structure (`class`, `Main`, namespaces).
- Familiarity with variables, data types, and expressions.
- Knowledge of access modifiers (`public`, `private`, `protected`, `internal`).
- Awareness of inheritance and interfaces.
- Understanding of value types vs. reference types and `null`.

---

### Related Programming Areas

- **Object-oriented programming** — Methods are the behaviour of objects; virtual/abstract/sealed methods power polymorphism.
- **Functional programming** — Local functions, expression-bodied methods, and lambdas bring functional style to C#.
- **API design** — Method signatures, overloading, and extension methods shape public APIs.
- **Performance** — Static methods can be faster to invoke; local functions avoid delegate allocations.
- **Code organisation** — Methods are the primary unit of abstraction and reuse.

---

### Core Concepts / Features

1. Methods (Overarching Concept)
2. Method Declaration
3. Method Signature
4. Method Parameters
5. Method Arguments
6. Return Types
7. `void` Return
8. Method Body
9. Method Invocation
10. Method Overloading
11. Method Overriding
12. Static Methods
13. Instance Methods
14. Abstract Methods
15. Virtual Methods
16. Sealed Methods
17. Extension Methods
18. Local Functions
19. Expression-Bodied Methods
20. Method Best Practices

Each concept follows a uniform structure: definitions, purposes, syntax rules and structure, annotated code examples with expected outputs, real-world cases, and references.

---

## 1. Methods (Overarching Concept)

### Definitions

**Core Definition**
A method is a named, reusable block of code that performs a task, optionally accepts inputs, and optionally returns a result.

**Technical Definition**
A method is a code block that contains a series of statements. A program causes the statements to be executed by calling the method and specifying any required method arguments. In C#, every executed instruction is performed in the context of a method. Methods are declared in a `class`, `record`, or `struct` by specifying an optional access level, optional modifiers (`abstract`, `sealed`, `static`, `virtual`, etc.), the return type (or `void`), the method name, and any parameters.

**Beginner-Friendly Explanation**
A method is a **named action**. You write it once, and you can use it as many times as you like by calling its name.

---

### Purposes

- To encapsulate logic into reusable units.
- To organise code into readable, maintainable blocks.
- To abstract away implementation details behind a clear name.
- To enable code reuse across multiple call sites.
- To model the behaviour of objects and types.

---

### Syntax Rules and Structure

```
[accessModifier] [modifiers] returnType MethodName([parameters])
{
    // method body
}
```

**Component breakdown**

- `accessModifier` — `public`, `private`, `protected`, `internal`, etc. Default is `private`.
- `modifiers` — `static`, `virtual`, `abstract`, `sealed`, `override`, `async`, `extern`, etc.
- `returnType` — The type of the returned value, or `void`.
- `MethodName` — PascalCase by convention.
- `parameters` — Comma-separated parameter list in parentheses.

#### Syntax Rules

- Every method must have a name and a return type (or `void`).
- Methods must be declared inside a `class`, `record`, or `struct`.
- A method body is enclosed in `{ }` unless it is abstract or extern.

#### Constraints and Limitations

- Method names must follow C# identifier rules.
- A method cannot be declared outside a type.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: A Simple Method

```csharp
using System;

class Program
{
    static void SayHello()
    {
        Console.WriteLine("Hello!");
    }

    static void Main()
    {
        SayHello();   // Hello!
    }
}
```

**Expected Output**

```
Hello!
```

**Why this output occurs**

- `SayHello` is a method that prints a message. `Main` calls it.

---

### Real-World Cases

- **Utility methods** — `Math.Sqrt`, `Console.WriteLine`.
- **Domain behaviour** — `Order.CalculateTotal()`.
- **Event handlers** — `Button_Click`.

---

### References

- Methods in C# — Microsoft Learn - https://learn.microsoft.com/en-us/dotnet/csharp/methods
- Methods — C# language specification - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/language-specification/classes#156-methods

---

## 2. Method Declaration

### Definitions

**Core Definition**
Method declaration is the syntax that introduces a new method, specifying its access, modifiers, return type, name, parameters, and body.

**Technical Definition**
Methods are declared in a `class`, `record`, or `struct` by specifying an optional access level (default `private`), optional modifiers such as `abstract` or `sealed`, the return value (or `void`), the method name, and any method parameters enclosed in parentheses and separated by commas.

**Beginner-Friendly Explanation**
Declaration is **writing the recipe**. You give the method a name, list its ingredients, and describe the steps.

---

### Purposes

- To introduce a new method into a type.
- To specify the method’s accessibility and behaviour modifiers.
- To define the method’s parameters and return type.
- To provide the method’s implementation (body).

---

### Syntax Rules and Structure

```
[accessModifier] [modifiers] returnType MethodName([parameters])
{
    // body
}
```

**Component breakdown**

- `accessModifier` — Optional; default `private`.
- `modifiers` — Optional; `static`, `virtual`, `abstract`, `sealed`, `override`, `async`, etc.
- `returnType` — Required; `void` or a type.
- `MethodName` — Required; PascalCase.
- `parameters` — Optional; comma-separated.

#### Syntax Rules

- A method declaration must specify a return type.
- The body is `{ }` for concrete methods, `;` for abstract/extern methods.
- Modifiers like `abstract` and `sealed` have specific compatibility rules.

#### Constraints and Limitations

- Cannot combine `abstract` with `static`, `virtual`, `sealed`, or `extern`.
- `sealed` requires `override`.

---

### Annotated Code Examples and Expected Outputs

#### Example: Declaring Methods with Different Modifiers

```csharp
using System;

abstract class Motorcycle
{
    public void StartEngine() { Console.WriteLine("Engine started"); }
    protected void AddGas(int gallons) { Console.WriteLine($"Added {gallons} gallons"); }
    public virtual int Drive(int miles, int speed) { return miles; }
    public abstract double GetTopSpeed();
}

class SportBike : Motorcycle
{
    public override double GetTopSpeed() => 180.0;
}

class Program
{
    static void Main()
    {
        var bike = new SportBike();
        bike.StartEngine();          // Engine started
        Console.WriteLine(bike.GetTopSpeed()); // 180
    }
}
```

**Expected Output**

```
Engine started
180
```

**Why this output occurs**

- `StartEngine` is a public instance method; `GetTopSpeed` is abstract and overridden.

---

### Real-World Cases

- **Class design** — Declaring public APIs, protected extension points, private helpers.
- **Interface implementation** — Declaring methods that satisfy interface contracts.

---

### References

- Methods in C# — Method signatures — Microsoft Learn - https://learn.microsoft.com/en-us/dotnet/csharp/methods#method-signatures
- Method declarations — C# language specification - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/language-specification/classes#1561-general

---

## 3. Method Signature

### Definitions

**Core Definition**
A method signature uniquely identifies a method for overloading and override resolution. It consists of the method name, type parameter count, and parameter list (including `ref` and `out` modifiers).

**Technical Definition**
The method signature consists of the method name, the number of type parameters, and the number and types of its parameters (including `ref` and `out` modifiers). The **return type is not part of the signature** for the purposes of method overloading, but it **is** part of the signature when determining compatibility between a delegate and the method it points to. `ref`, `out`, and `ref readonly` modifiers are part of the signature; `params` and `scoped` modifiers are not.

**Beginner-Friendly Explanation**
The signature is the **method’s identity card** for overloading. It says: “I am `Drive(int, int)`.” Two methods with the same name but different signatures are different methods.

---

### Purposes

- To distinguish overloaded methods.
- To match overrides with their base methods.
- To determine delegate compatibility.
- To enable precise method resolution by the compiler.

---

### Syntax Rules and Structure (What Counts)

| Component | Part of signature? |
|---|---|
| Method name | ✔️ Yes |
| Type parameter count | ✔️ Yes |
| Parameter types | ✔️ Yes |
| `ref` / `out` modifiers | ✔️ Yes |
| `params` modifier | ❌ No |
| Return type | ❌ No (for overloading) |
| `scoped` modifier | ❌ No |

**Component breakdown**

- **Name** — The method’s identifier.
- **Type parameters** — Generic arity.
- **Parameters** — Number and types, including by-reference modifiers.

#### Syntax Rules

- Overloaded methods must have different signatures.
- Overridden methods must have the same signature.
- Return type alone cannot distinguish overloads.

#### Constraints and Limitations

- Cannot overload solely by return type.
- `ref` and `out` are part of the signature.

---

### Annotated Code Examples and Expected Outputs

#### Example: Overloading by Signature

```csharp
using System;

class Calculator
{
    public int Add(int a, int b) => a + b;
    public double Add(double a, double b) => a + b;   // different signature
}

class Program
{
    static void Main()
    {
        var calc = new Calculator();
        Console.WriteLine(calc.Add(1, 2));       // 3
        Console.WriteLine(calc.Add(1.5, 2.5));   // 4
    }
}
```

**Expected Output**

```
3
4
```

**Why this output occurs**

- The parameter types differ, so the signatures differ, enabling overloading.

---

### Real-World Cases

- **API design** — Providing multiple overloads for convenience.
- **Override correctness** — Ensuring derived methods match base signatures.

---

### References

- Method signatures — Methods in C# — Microsoft Learn - https://learn.microsoft.com/en-us/dotnet/csharp/methods#method-signatures
- Signatures and overloading — C# language specification - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/language-specification/basic-concepts#78-signatures-and-overloading

---

## 4. Method Parameters

### Definitions

**Core Definition**
Method parameters are the **named inputs** declared in a method’s parameter list. They define what information the method needs to do its job.

**Technical Definition**
A method’s parameter list declares the names and types of the values the method requires. Parameters are enclosed in parentheses and separated by commas; empty parentheses indicate no parameters. C# supports **value parameters** (default), **reference parameters** (`ref`), **output parameters** (`out`), **input reference parameters** (`in`), and **parameter arrays** (`params`). Optional parameters can specify default values. `ref`, `out`, and `ref readonly` modifiers are part of the method signature; `params` and `scoped` are not.

**Beginner-Friendly Explanation**
Parameters are the **ingredients listed in a recipe**. They tell the method what it needs to work. The caller supplies the actual ingredients (arguments) when calling the method.

---

### Purposes

- To define the inputs a method requires.
- To support value, reference, output, and variable-length inputs.
- To enable optional arguments with default values.
- To allow methods to return multiple values via `out` parameters.
- To enable efficient large-struct passing with `in`.

---

### Syntax Rules and Structure (Parameter Kinds)

| Kind | Syntax | Purpose |
|---|---|---|
| **Value** | `int x` | Pass by value (default). |
| **Reference** | `ref int x` | Pass by reference; must be initialized before call. |
| **Output** | `out int x` | Pass by reference; must be assigned inside method. |
| **Input reference** | `in int x` | Pass by reference; read-only inside method. |
| **Parameter array** | `params int[] xs` | Variable number of arguments. |
| **Optional** | `int x = 0` | Default value if argument omitted. |

**Component breakdown**

- `ref` — Caller and method share the same storage.
- `out` — Method must assign before returning.
- `in` — Read-only reference for performance.
- `params` — Must be the last parameter.

#### Syntax Rules

- `ref` and `out` are part of the signature; `params` is not.
- `params` must be the last parameter.
- Optional parameters must come after required parameters.
- `ref`/`out` parameters cannot have default values.

#### Constraints and Limitations

- `out` parameters must be definitely assigned by the method.
- `ref` arguments must be definitely assigned before the call.
- `in` parameters cannot be modified (unless the compiler can prove no modification).

---

### Annotated Code Examples and Expected Outputs

#### Example: `out` Parameter for Multiple Returns

```csharp
using System;

class Program
{
    static bool TryDivide(int dividend, int divisor, out int quotient)
    {
        if (divisor == 0)
        {
            quotient = 0;
            return false;
        }
        quotient = dividend / divisor;
        return true;
    }

    static void Main()
    {
        if (TryDivide(10, 2, out int result))
            Console.WriteLine(result);   // 5
    }
}
```

**Expected Output**

```
5
```

**Why this output occurs**

- `out int quotient` allows the method to return both a `bool` and the quotient.

---

### Real-World Cases

- **Parsing** — `int.TryParse(input, out int value)`.
- **Math** — `Math.DivRem` with `out` remainder.
- **Performance** — Passing large structs with `in`.

---

### References

- Method parameters — C# language specification - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/language-specification/classes#1562-parameters
- Parameter modifiers — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/method-parameters

---

## 5. Method Arguments

### Definitions

**Core Definition**
Method arguments are the **actual values** supplied by the caller when invoking a method. They are matched to the method’s parameters.

**Technical Definition**
When a caller invokes a method, it provides concrete values, called **arguments**, for each parameter. The arguments must be compatible with the parameter types, but the argument name, if one is used in the calling code, doesn’t have to be the same as the parameter name defined in the method. Arguments can be passed **positionally** or by **named argument** (`paramName: value`). The `ref`, `out`, and `in` modifiers must be repeated at the call site.

**Beginner-Friendly Explanation**
Arguments are the **actual ingredients you hand to the recipe**. If the recipe asks for “2 eggs” (parameter), you give it “2” (argument).

---

### Purposes

- To supply values to a method’s parameters.
- To support positional and named argument styles.
- To enable optional arguments to be skipped by name.
- To pass variables by reference using `ref`, `out`, or `in`.

---

### Syntax Rules and Structure

```
MethodName(argument1, argument2, ...)
MethodName(parameterName: argument, ...)
```

**Component breakdown**

- **Positional arguments** — Matched to parameters in order.
- **Named arguments** — Matched by parameter name.
- **`ref` / `out` / `in`** — Must match the parameter modifier.

#### Syntax Rules

- Arguments must be implicitly convertible to the parameter type.
- Named arguments can appear in any order after positional arguments.
- `ref`/`out` arguments must be variables (not constants).

#### Constraints and Limitations

- Cannot use named arguments before positional arguments in C# 7.2+? (Actually, named arguments can be mixed with positional in C# 7.2+ if positional are in correct positions.)
- `out` arguments need not be initialized before the call.

---

### Annotated Code Examples and Expected Outputs

#### Example: Named and Optional Arguments

```csharp
using System;

class Program
{
    static void PrintInfo(string name, int age = 0, string city = "Unknown")
    {
        Console.WriteLine($"{name}, {age}, {city}");
    }

    static void Main()
    {
        PrintInfo("Alice", 30, "London");   // Alice, 30, London
        PrintInfo("Bob", city: "Paris");    // Bob, 0, Paris
    }
}
```

**Expected Output**

```
Alice, 30, London
Bob, 0, Paris
```

**Why this output occurs**

- Named argument `city:` skips `age`, using its default value.

---

### Real-World Cases

- **Readable call sites** — `SendEmail(to: "a@b.com", subject: "Hi")`.
- **Optional configuration** — Skipping optional parameters by name.

---

### References

- Method invocation — Methods in C# — Microsoft Learn - https://learn.microsoft.com/en-us/dotnet/csharp/methods#method-invocation
- Arguments — C# language specification - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/language-specification/expressions#128102-method-invocations

---

## 6. Return Types

### Definitions

**Core Definition**
A method’s return type specifies the type of value the method produces and returns to its caller.

**Technical Definition**
The return type is the type of the value the method returns, or `void` if the method returns no value. A method with a non-void return type must return a value of that type (or a type implicitly convertible to it) on every path that completes normally. The return type is **not** part of the method signature for overload resolution but **is** part of the signature for delegate compatibility.

**Beginner-Friendly Explanation**
The return type is **what the method gives back**. A method that calculates a sum returns an `int`; a method that prints a message returns nothing (`void`).

---

### Purposes

- To specify the kind of result a method produces.
- To enable method chaining and expression composition.
- To support `void` for side-effecting methods.
- To enable generic return types for flexibility.

---

### Syntax Rules and Structure

```
returnType MethodName(...) { ... }
```

**Component breakdown**

- `returnType` — Any type, including `void`, `int`, `string`, `T`, tuples, etc.
- `return` — The keyword that returns a value.

#### Syntax Rules

- Non-void methods must return a value on all code paths.
- The returned value must be implicitly convertible to the return type.
- `void` methods cannot return a value.

#### Constraints and Limitations

- Return type alone cannot distinguish overloads.
- Async methods return `Task` or `Task<T>`.

---

### Annotated Code Examples and Expected Outputs

#### Example: Multiple Return Paths

```csharp
using System;

class Program
{
    static int Absolute(int value)
    {
        if (value >= 0)
            return value;
        return -value;
    }

    static void Main()
    {
        Console.WriteLine(Absolute(-5));   // 5
    }
}
```

**Expected Output**

```
5
```

**Why this output occurs**

- Both return paths return an `int`, satisfying the return type.

---

### Real-World Cases

- **Calculations** — Returning computed results.
- **Factories** — Returning new objects.
- **LINQ** — Returning sequences (`IEnumerable<T>`).

---

### References

- Return types — Methods in C# — Microsoft Learn - https://learn.microsoft.com/en-us/dotnet/csharp/methods#method-signatures
- Return statement — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/statements/jump-statements#the-return-statement

---

## 7. `void` Return

### Definitions

**Core Definition**
`void` is the return type used for methods that perform an action but do not return a value.

**Technical Definition**
A method declared with `void` as its return type does not return a value. It can use the `return;` statement (without an expression) to exit early, but it cannot return a value. `void` is not a type in the ordinary sense; it indicates the absence of a return value.

**Beginner-Friendly Explanation**
`void` means **“this method does something but gives nothing back.”** It is used for actions like printing, saving, or modifying state.

---

### Purposes

- To declare methods that perform side effects only.
- To signal that no result is expected.
- To support event handlers and action methods.
- To allow early exit with `return;`.

---

### Syntax Rules and Structure

```
void MethodName(...) { ... return; }
```

**Component breakdown**

- `void` — The return type.
- `return;` — Optional early exit.

#### Syntax Rules

- `void` methods cannot return a value.
- `return;` exits the method immediately.
- `void` cannot be used as a type for variables, parameters, or fields.

#### Constraints and Limitations

- Cannot assign the result of a `void` method to a variable.
- `void` is not a first-class type.

---

### Annotated Code Examples and Expected Outputs

#### Example: `void` Method with Early Return

```csharp
using System;

class Program
{
    static void PrintPositive(int value)
    {
        if (value <= 0)
            return;
        Console.WriteLine(value);
    }

    static void Main()
    {
        PrintPositive(5);    // 5
        PrintPositive(-1);   // (no output)
    }
}
```

**Expected Output**

```
5
```

**Why this output occurs**

- `return;` exits early for non-positive values; `void` means no value is returned.

---

### Real-World Cases

- **Logging** — `Logger.Log(message)`.
- **UI actions** — `button.Click += (s, e) => { ... };`.
- **State mutation** — `list.Add(item)`.

---

### References

- void — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/void
- Return statement — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/statements/jump-statements#the-return-statement

---

## 8. Method Body

### Definitions

**Core Definition**
The method body is the block of statements enclosed in braces `{ }` that defines what the method does.

**Technical Definition**
The method body contains the executable statements of the method. For abstract and extern methods, the body is a semicolon (`;`); for expression-bodied methods, the body is an expression following `=>`. The body is executed when the method is invoked.

**Beginner-Friendly Explanation**
The method body is the **step-by-step instructions** of the recipe. It is what actually happens when the method runs.

---

### Purposes

- To contain the executable logic of the method.
- To declare local variables and control flow.
- To return a result (if non-void).
- To perform side effects.

---

### Syntax Rules and Structure

```
returnType MethodName(...)
{
    // statements
}
```

**Component breakdown**

- `{ }` — The body block.
- Statements — Variable declarations, control flow, expressions, return.

#### Syntax Rules

- Concrete methods must have a body unless marked `abstract` or `extern`.
- The body can contain any valid statements.
- All paths in a non-void method must return a value or throw.

#### Constraints and Limitations

- Abstract methods have no body (`;`).
- Expression-bodied methods use `=>` instead of `{ }`.

---

### Annotated Code Examples and Expected Outputs

#### Example: Body with Local Variables and Loop

```csharp
using System;

class Program
{
    static int Sum(int[] numbers)
    {
        int total = 0;
        foreach (int n in numbers)
            total += n;
        return total;
    }

    static void Main()
    {
        Console.WriteLine(Sum(new[] { 1, 2, 3 }));   // 6
    }
}
```

**Expected Output**

```
6
```

**Why this output occurs**

- The body declares `total`, loops over the array, and returns the sum.

---

### Real-World Cases

- **All concrete methods** — The body implements the behaviour.

---

### References

- Method body — C# language specification - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/language-specification/classes#1564-method-body

---

## 9. Method Invocation

### Definitions

**Core Definition**
Method invocation (calling a method) is the act of executing a method by using its name and supplying arguments.

**Technical Definition**
A method invocation consists of an instance or type name (for instance or static methods, respectively), a period, the method name, and parentheses containing the arguments. The arguments must be compatible with the parameters. Invoking a method causes control to transfer to the method body; when the method returns, control returns to the caller.

**Beginner-Friendly Explanation**
Invoking a method is **using the recipe**. You say its name and give it the ingredients, and it runs.

---

### Purposes

- To execute a method’s logic.
- To pass data into a method.
- To obtain a method’s return value.
- To trigger side effects.

---

### Syntax Rules and Structure

```
TypeName.MethodName(args)      // static
instance.MethodName(args)      // instance
```

**Component breakdown**

- `TypeName` / `instance` — The target.
- `.` — Member access.
- `MethodName` — The method.
- `(args)` — The arguments.

#### Syntax Rules

- Static methods are invoked via the type name.
- Instance methods are invoked via an object reference.
- Arguments must match parameters.

#### Constraints and Limitations

- Calling an instance method on `null` throws `NullReferenceException`.
- Calling a static method via an instance is a compile error.

---

### Annotated Code Examples and Expected Outputs

#### Example: Static and Instance Invocation

```csharp
using System;

class Counter
{
    public int Count;
    public void Increment() => Count++;
    public static Counter Create() => new Counter();
}

class Program
{
    static void Main()
    {
        Counter c = Counter.Create();   // static
        c.Increment();                   // instance
        Console.WriteLine(c.Count);      // 1
    }
}
```

**Expected Output**

```
1
```

**Why this output occurs**

- `Create` is static and invoked via the type; `Increment` is instance and invoked via `c`.

---

### Real-World Cases

- **All method usage** — Every method call is an invocation.

---

### References

- Method invocation — Methods in C# — Microsoft Learn - https://learn.microsoft.com/en-us/dotnet/csharp/methods#method-invocation

---

## 10. Method Overloading

### Definitions

**Core Definition**
Method overloading is the practice of declaring multiple methods with the **same name** but **different signatures** (different parameter types, counts, or modifiers) within the same type.

**Technical Definition**
Overloaded methods share a name but are distinguished by their signatures—the number and types of their parameters (including `ref`/`out` modifiers). The return type is **not** part of the signature for overloading. When a method is invoked, the compiler selects the best overload based on the arguments provided.

**Beginner-Friendly Explanation**
Overloading lets you use the **same name for similar actions** that take different inputs. `Print(int)` and `Print(string)` are two versions of the same idea.

---

### Purposes

- To provide convenient APIs with the same conceptual operation.
- To support different parameter types or counts.
- To improve readability by avoiding arbitrary name variations.
- To enable default-like behaviour through overloads.

---

### Syntax Rules and Structure

```
void Print(int value) { ... }
void Print(string value) { ... }
void Print(int a, int b) { ... }
```

**Component breakdown**

- Same method name.
- Different parameter lists (types, counts, `ref`/`out`).

#### Syntax Rules

- Overloads must differ in signature.
- Return type alone cannot distinguish overloads.
- `ref` vs. `out` vs. value are different signatures.

#### Constraints and Limitations

- Ambiguous calls cause compile errors.
- Overloading across inheritance can hide base methods if not careful.

---

### Annotated Code Examples and Expected Outputs

#### Example: Overloaded `Drive` Methods

```csharp
using System;

class Motorcycle
{
    public virtual int Drive(int miles, int speed) => miles;
    public virtual int Drive(TimeSpan time, int speed) => (int)(time.TotalHours * speed);
}

class Program
{
    static void Main()
    {
        var m = new Motorcycle();
        Console.WriteLine(m.Drive(10, 50));                       // 10
        Console.WriteLine(m.Drive(TimeSpan.FromHours(2), 50));    // 100
    }
}
```

**Expected Output**

```
10
100
```

**Why this output occurs**

- The two `Drive` methods have different parameter types, so they are overloads.

---

### Real-World Cases

- **`Console.WriteLine`** — Many overloads for different types.
- **`Math.Max`** — Overloads for `int`, `double`, etc.

---

### References

- Overloading — Methods in C# — Microsoft Learn - https://learn.microsoft.com/en-us/dotnet/csharp/methods#method-signatures
- Overload resolution — C# language specification - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/language-specification/expressions#1263-overload-resolution

---

## 11. Method Overriding

### Definitions

**Core Definition**
Method overriding is the act of providing a **new implementation** for a `virtual` or `abstract` method inherited from a base class, using the `override` keyword and the **same signature**.

**Technical Definition**
A derived class can override an inherited `virtual` or `abstract` method by declaring a method with the `override` modifier and the same signature. The overridden method is called polymorphically: the runtime type of the object determines which implementation executes. The method signature must be the same as the overridden method.

**Beginner-Friendly Explanation**
Overriding is **customising an inherited behaviour**. If the base class says “a shape can be drawn,” a derived `Circle` can override `Draw` to draw a circle specifically.

---

### Purposes

- To provide type-specific behaviour for a base-class contract.
- To enable polymorphism.
- To customise inherited functionality.
- To implement abstract methods.

---

### Syntax Rules and Structure

```
override returnType MethodName(parameters) { ... }
```

**Component breakdown**

- `override` — Required modifier.
- Same signature as the base method.

#### Syntax Rules

- The base method must be `virtual` or `abstract`.
- The signature must match exactly.
- `sealed override` prevents further overriding.

#### Constraints and Limitations

- Cannot override a non-virtual method.
- Cannot change the signature.

---

### Annotated Code Examples and Expected Outputs

#### Example: Overriding a Virtual Method

```csharp
using System;

class Shape
{
    public virtual void Draw() => Console.WriteLine("Drawing a shape");
}

class Circle : Shape
{
    public override void Draw() => Console.WriteLine("Drawing a circle");
}

class Program
{
    static void Main()
    {
        Shape s = new Circle();
        s.Draw();   // Drawing a circle
    }
}
```

**Expected Output**

```
Drawing a circle
```

**Why this output occurs**

- `Circle` overrides `Draw`; the runtime type determines which implementation runs.

---

### Real-World Cases

- **UI frameworks** — Overriding `OnPaint` in custom controls.
- **Domain models** — Overriding `CalculatePrice` in derived product types.

---

### References

- Overriding — Methods in C# — Microsoft Learn - https://learn.microsoft.com/en-us/dotnet/csharp/methods#inherited-and-overridden-methods
- Override modifier — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/override

---

## 12. Static Methods

### Definitions

**Core Definition**
A static method belongs to the **type itself**, not to any instance, and is invoked through the type name.

**Technical Definition**
When a method declaration includes the `static` modifier, the method is a static method. A static method does not operate on a specific instance and cannot reference `this`; doing so is a compile-time error. Static methods are invoked by referencing the type name, not an object instance.

**Beginner-Friendly Explanation**
A static method is a **utility function** attached to a class. You do not need an object to use it—just the class name. `Math.Sqrt` is static; you do not create a `Math` object to take a square root.

---

### Purposes

- To provide utility functions that do not depend on object state.
- To implement factory methods.
- To group related operations under a type name.
- To avoid unnecessary object allocation.
- To support extension methods (which are static).

---

### Syntax Rules and Structure

```
static returnType MethodName(parameters) { ... }
```

**Component breakdown**

- `static` — The modifier.
- No access to instance members.

#### Syntax Rules

- Cannot use `this`.
- Can access static members of the type.
- Invoked via `TypeName.MethodName()`.

#### Constraints and Limitations

- Cannot be virtual, abstract, or override.
- Cannot access instance fields or methods directly.

---

### Annotated Code Examples and Expected Outputs

#### Example: Static Utility Method

```csharp
using System;

class MathHelper
{
    public static int Square(int x) => x * x;
}

class Program
{
    static void Main()
    {
        Console.WriteLine(MathHelper.Square(5));   // 25
    }
}
```

**Expected Output**

```
25
```

**Why this output occurs**

- `Square` is static and invoked via the type name.

---

### Real-World Cases

- **Math utilities** — `Math.Abs`, `Math.Max`.
- **Factories** — `DateTime.Parse`, `Guid.NewGuid`.
- **Extension methods** — Static methods with `this` on the first parameter.

---

### References

- Static methods — C# language specification - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/language-specification/classes#1563-static-and-instance-methods
- Static classes and static members — C# programming guide - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/static-classes-and-static-class-members

---

## 13. Instance Methods

### Definitions

**Core Definition**
An instance method operates on a specific **object instance** and can access that instance’s fields and properties.

**Technical Definition**
A method without the `static` modifier is an instance method. It operates on a particular instance and can reference `this` to access the instance’s data. Instance methods are invoked through an object reference.

**Beginner-Friendly Explanation**
An instance method is a **behaviour of an object**. A `Car` object has a `StartEngine` method that operates on that specific car.

---

### Purposes

- To model object behaviour.
- To access and modify instance state.
- To implement polymorphic behaviour via virtual/override.
- To encapsulate object-specific logic.

---

### Syntax Rules and Structure

```
returnType MethodName(parameters) { ... }
```

**Component breakdown**

- No `static` modifier.
- Access to `this` and instance members.

#### Syntax Rules

- Must be invoked on an instance.
- Can use `this`.
- Can be virtual, abstract, override, sealed.

#### Constraints and Limitations

- Cannot be invoked without an instance (except via `null` causing `NullReferenceException`).

---

### Annotated Code Examples and Expected Outputs

#### Example: Instance Method Modifying State

```csharp
using System;

class Counter
{
    private int _count;
    public void Increment() => _count++;
    public int Count => _count;
}

class Program
{
    static void Main()
    {
        var c = new Counter();
        c.Increment();
        Console.WriteLine(c.Count);   // 1
    }
}
```

**Expected Output**

```
1
```

**Why this output occurs**

- `Increment` is an instance method that modifies `_count` on the specific object.

---

### Real-World Cases

- **Domain objects** — `Order.AddItem`, `Account.Deposit`.
- **UI controls** — `Button.PerformClick`.

---

### References

- Instance methods — C# language specification - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/language-specification/classes#1563-static-and-instance-methods

---

## 14. Abstract Methods

### Definitions

**Core Definition**
An abstract method is a method declared without a body that **must be implemented by non-abstract derived classes**.

**Technical Definition**
When an instance method declaration includes the `abstract` modifier, the method is an abstract method. Although abstract methods are implicitly virtual, they cannot include the `virtual` modifier. An abstract method declaration introduces a new virtual method but provides no implementation; non-abstract derived classes must override the method to provide an implementation. The body of an abstract method is just a semicolon. Abstract methods are permitted only in abstract classes and interfaces.

**Beginner-Friendly Explanation**
An abstract method is a **contract with no implementation**. It says: “Every concrete subclass must provide this method.” It is like a job description that every hire must fulfil.

---

### Purposes

- To define a contract that derived classes must implement.
- To force customisation in subclasses.
- To support polymorphism with guaranteed implementation.
- To model abstract concepts with required behaviour.

---

### Syntax Rules and Structure

```
abstract returnType MethodName(parameters);
```

**Component breakdown**

- `abstract` — The modifier.
- `;` — No body.

#### Syntax Rules

- Only allowed in abstract classes and interfaces.
- Derived non-abstract classes must override.
- Cannot be `static`, `virtual`, `sealed`, or `extern`.

#### Constraints and Limitations

- Cannot have a body.
- Cannot be instantiated directly (abstract class).
- Cannot call `base.Method()` if it is abstract.

---

### Annotated Code Examples and Expected Outputs

#### Example: Abstract Method in a Shape Hierarchy

```csharp
using System;

abstract class Shape
{
    public abstract double Area();
}

class Circle : Shape
{
    public double Radius { get; set; }
    public override double Area() => Math.PI * Radius * Radius;
}

class Program
{
    static void Main()
    {
        Shape s = new Circle { Radius = 2 };
        Console.WriteLine(s.Area().ToString("F2"));   // 12.57
    }
}
```

**Expected Output**

```
12.57
```

**Why this output occurs**

- `Shape` declares `abstract Area()`; `Circle` provides the implementation.

---

### Real-World Cases

- **Framework design** — Abstract base classes with required overrides.
- **Plugin systems** — Abstract contracts for plugins.

---

### References

- Abstract methods — C# language specification - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/language-specification/classes#1567-abstract-methods
- Abstract modifier — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/abstract

---

## 15. Virtual Methods

### Definitions

**Core Definition**
A virtual method is a method that **can be overridden** by derived classes to provide a different implementation.

**Technical Definition**
When an instance method declaration includes the `virtual` modifier, the method is a virtual method. A virtual method provides an implementation that derived classes can override using the `override` modifier. Virtual methods enable polymorphism: the runtime type of the object determines which implementation is invoked.

**Beginner-Friendly Explanation**
A virtual method is a **default behaviour that can be customised**. The base class provides a reasonable implementation, but subclasses can replace it.

---

### Purposes

- To provide a default implementation that derived classes can override.
- To enable polymorphism.
- To allow customisation without forcing it (unlike abstract).
- To support extensible base classes.

---

### Syntax Rules and Structure

```
virtual returnType MethodName(parameters) { ... }
```

**Component breakdown**

- `virtual` — The modifier.
- A body is required.

#### Syntax Rules

- Can be overridden with `override`.
- Can be sealed in a derived class with `sealed override`.
- Cannot be `static`, `abstract`, or `sealed`.

#### Constraints and Limitations

- Only instance methods can be virtual.
- Virtual calls are slightly slower due to virtual dispatch (usually negligible).

---

### Annotated Code Examples and Expected Outputs

#### Example: Virtual Method with Default Behaviour

```csharp
using System;

class Animal
{
    public virtual string Speak() => "...";
}

class Dog : Animal
{
    public override string Speak() => "Woof!";
}

class Program
{
    static void Main()
    {
        Animal a = new Dog();
        Console.WriteLine(a.Speak());   // Woof!
    }
}
```

**Expected Output**

```
Woof!
```

**Why this output occurs**

- `Dog` overrides the virtual `Speak`; the runtime type determines the implementation.

---

### Real-World Cases

- **Framework base classes** — `Control.OnPaint` is virtual.
- **Domain models** — `Product.CalculateDiscount` is virtual.

---

### References

- Virtual methods — C# language specification - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/language-specification/classes#1564-virtual-methods
- Virtual modifier — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/virtual

---

## 16. Sealed Methods

### Definitions

**Core Definition**
A sealed method is an overridden method that **cannot be overridden further** by derived classes.

**Technical Definition**
When an instance method declaration includes the `sealed` modifier, the method is a sealed method. A sealed method overrides an inherited virtual method with the same signature. A sealed method must also be marked with the `override` modifier. The `sealed` modifier prevents a derived class from further overriding the method.

**Beginner-Friendly Explanation**
A sealed method is a **locked customisation**. A subclass has overridden a base method but wants to make sure no further subclass changes it.

---

### Purposes

- To prevent further overriding of a method.
- To lock down critical behaviour.
- To protect invariants in a class hierarchy.

---

### Syntax Rules and Structure

```
sealed override returnType MethodName(parameters) { ... }
```

**Component breakdown**

- `sealed` — Required modifier.
- `override` — Also required.

#### Syntax Rules

- Must be combined with `override`.
- The base method must be virtual or abstract.
- The signature must match.

#### Constraints and Limitations

- Cannot be used on a method that is not an override.
- Cannot be combined with `virtual` or `abstract`.

---

### Annotated Code Examples and Expected Outputs

#### Example: Sealed Override

```csharp
using System;

class A
{
    public virtual void F() => Console.WriteLine("A.F");
}

class B : A
{
    public sealed override void F() => Console.WriteLine("B.F");
}

class C : B
{
    // public override void F() => ...;  // Compile error: sealed
}

class Program
{
    static void Main()
    {
        A a = new B();
        a.F();   // B.F
    }
}
```

**Expected Output**

```
B.F
```

**Why this output occurs**

- `B` seals its override of `F`, preventing `C` from overriding it.

---

### Real-World Cases

- **Framework design** — Sealing critical methods in base classes.
- **Security** — Preventing derived classes from bypassing validation.

---

### References

- Sealed methods — C# language specification - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/language-specification/classes#1566-sealed-methods
- Sealed modifier — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/sealed

---

## 17. Extension Methods

### Definitions

**Core Definition**
An extension method is a **static method** that can be invoked using **instance-method syntax** on a type it extends, without modifying the original type.

**Technical Definition**
Extension methods are static methods where the first parameter is modified with the `this` keyword and specifies the type being extended. When the containing static class is in scope, the method can be called using instance-method syntax on a receiver of the extended type. Instance methods defined on the type itself take precedence over extension methods.

**Beginner-Friendly Explanation**
Extension methods let you **add new tricks to existing types** without changing them. For example, you can add a `WordCount()` method to `string` without modifying `System.String`.

---

### Purposes

- To extend existing types with new methods without inheritance or modification.
- To add functionality to sealed or third-party types.
- To provide fluent APIs and LINQ-style helpers.
- To keep utility methods discoverable via IntelliSense.

---

### Syntax Rules and Structure

```
public static class Extensions
{
    public static ReturnType MethodName(this ExtendedType value, parameters)
    {
        ...
    }
}
```

**Component breakdown**

- `static class` — Must be a non-nested static class.
- `this` — Modifies the first parameter.
- `ExtendedType` — The type being extended.

#### Syntax Rules

- Must be declared in a static class.
- The first parameter must have `this`.
- Cannot override instance methods (instance methods win).
- `ref`/`out`/`in` cannot be used on the `this` parameter (except `this ref` in C# 7.2+ for `ref` extension methods).

#### Constraints and Limitations

- Cannot access private members of the extended type.
- Lower priority than instance methods.
- Not discoverable if the static class is not in scope.

---

### Annotated Code Examples and Expected Outputs

#### Example: `WordCount` Extension for `string`

```csharp
using System;

public static class StringExtensions
{
    public static int WordCount(this string str) =>
        str.Split(new[] { ' ' }, StringSplitOptions.RemoveEmptyEntries).Length;
}

class Program
{
    static void Main()
    {
        string text = "Hello world from C#";
        Console.WriteLine(text.WordCount());   // 4
    }
}
```

**Expected Output**

```
4
```

**Why this output occurs**

- `WordCount` is an extension method invoked as if it were an instance method on `string`.

---

### Real-World Cases

- **LINQ** — `Where`, `Select`, `OrderBy` are extension methods on `IEnumerable<T>`.
- **Utility libraries** — String, collection, and validation helpers.
- **Fluent APIs** — Method chaining via extension methods.

---

### References

- Extension methods — C# programming guide - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/extension-methods
- Extension methods — C# language specification - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/language-specification/classes#1569-extension-methods

---

## 18. Local Functions

### Definitions

**Core Definition**
A local function is a **method nested inside another member** (method, constructor, property accessor, etc.) that can be called only from its containing member.

**Technical Definition**
Local functions are methods of a type that are nested in another member. They can be called only from their containing member. Local functions can be declared in and called from methods, constructors, property accessors, event accessors, anonymous methods, lambda expressions, finalizers, and other local functions. Local functions cannot be declared inside an expression-bodied member. Local functions can be `async`, `unsafe`, `static`, or `extern`; a `static` local function cannot capture enclosing local variables or instance state.

**Beginner-Friendly Explanation**
A local function is a **helper method that lives inside another method**. It keeps the helper close to where it is used and prevents other code from calling it.

---

### Purposes

- To keep helper logic close to its usage.
- To avoid polluting the class with private methods.
- To enable recursion without a separate class member.
- To support iterator and async helpers.
- To improve readability by localising implementation details.

---

### Syntax Rules and Structure

```
returnType ContainerMethod()
{
    returnType LocalFunction(parameters) { ... }
    return LocalFunction();
}
```

**Component breakdown**

- Declared inside another member.
- Callable only from the containing member.
- Can capture enclosing locals (unless `static`).

#### Syntax Rules

- Cannot have access modifiers.
- Can be `async`, `static`, `unsafe`, `extern`.
- Cannot be declared inside an expression-bodied member.

#### Constraints and Limitations

- Not callable from outside the containing member.
- `static` local functions cannot capture enclosing state.

---

### Annotated Code Examples and Expected Outputs

#### Example: Local Function Helper

```csharp
using System;

class Program
{
    static int Factorial(int n)
    {
        return Compute(n);

        int Compute(int x) => x <= 1 ? 1 : x * Compute(x - 1);
    }

    static void Main()
    {
        Console.WriteLine(Factorial(5));   // 120
    }
}
```

**Expected Output**

```
120
```

**Why this output occurs**

- `Compute` is a local function inside `Factorial`, capturing the recursion logic locally.

---

### Real-World Cases

- **Validation helpers** — Local functions for argument checks.
- **Recursive algorithms** — Local recursive functions.
- **Iterator helpers** — Local functions returning `IEnumerable<T>`.

---

### References

- Local functions — C# programming guide - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/local-functions
- Local functions — C# language specification - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/language-specification/statements#1354-local-functions

---

## 19. Expression-Bodied Methods

### Definitions

**Core Definition**
An expression-bodied method is a method whose body is a **single expression** following the `=>` token, instead of a brace-enclosed block.

**Technical Definition**
Expression-bodied members use the `=>` token to provide a concise implementation for methods, properties, constructors, finalizers, and accessors. For methods, the syntax is `returnType MethodName(parameters) => expression;`. The expression’s value is returned (for non-void methods) or evaluated (for void methods). Expression-bodied methods are syntactic sugar for a single-statement body.

**Beginner-Friendly Explanation**
An expression-bodied method is a **one-line method**. Instead of `{ return x * 2; }`, you write `=> x * 2`.

---

### Purposes

- To write concise methods for simple operations.
- To improve readability for trivial implementations.
- To reduce boilerplate.
- To express computation as an expression.

---

### Syntax Rules and Structure

```
returnType MethodName(parameters) => expression;
```

**Component breakdown**

- `=>` — The expression-body operator.
- `expression` — The single expression.

#### Syntax Rules

- The expression must be implicitly convertible to the return type.
- For `void` methods, the expression must be a statement expression.
- Can be used for methods, constructors, finalizers, properties, and accessors.

#### Constraints and Limitations

- Only one expression; cannot contain multiple statements.
- Not suitable for complex logic.

---

### Annotated Code Examples and Expected Outputs

#### Example: Expression-Bodied Method

```csharp
using System;

class Program
{
    static int Square(int x) => x * x;

    static void Main()
    {
        Console.WriteLine(Square(5));   // 25
    }
}
```

**Expected Output**

```
25
```

**Why this output occurs**

- `Square` returns the expression `x * x` directly.

---

### Real-World Cases

- **Mathematical helpers** — `Area => Math.PI * r * r`.
- **Simple getters** — `FullName => $"{First} {Last}"`.
- **ToString overrides** — `public override string ToString() => ...;`.

---

### References

- Expression-bodied members — C# programming guide - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/statements-expressions-operators/expression-bodied-members
- Expression bodies — C# language specification - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/language-specification/classes#1564-method-body

---

## 20. Method Best Practices

### Definitions

**Core Definition**
Method best practices are the guidelines that ensure methods are **well-named, focused, testable, and maintainable**.

**Technical Definition**
Best practices for methods derive from Microsoft’s C# coding conventions and .NET Framework Design Guidelines. Key recommendations include: **use PascalCase for method names**; **use verbs or verb phrases** for method names; **keep methods small and focused** on a single responsibility; **limit the number of parameters** (ideally ≤ 4, using objects for more); **avoid excessive overloading**; **prefer returning values over `out` parameters**; **use `static` only when the method depends solely on its parameters**; **favour expression-bodied members for simple methods**; **use local functions for helpers**; and **document public methods with XML comments**.

**Beginner-Friendly Explanation**
Best practices are the habits that keep methods clean: give them clear verb-based names, keep them short, avoid too many parameters, and do not make them do too many things.

---

### Purposes

- To improve readability and discoverability.
- To keep methods testable and maintainable.
- To avoid parameter bloat.
- To choose the right method kind (static, instance, extension, local).
- To align with .NET conventions and tooling.

---

### Guidelines and Structure

#### General Guidelines

```
// Use PascalCase and verb phrases
public void CalculateTotal() { ... }

// Keep methods small and focused
// Limit parameters (≤ 4)
// Prefer return values over out parameters
// Use static only for parameter-only logic
// Use expression-bodied members for simple logic
// Use local functions for helpers
// Document public APIs
```

**Component breakdown**

- **PascalCase** — Method names use PascalCase.
- **Verb phrases** — Methods are actions.
- **Small and focused** — Single responsibility.
- **Limited parameters** — Avoid long parameter lists.
- **Static when appropriate** — Only when the method depends solely on its parameters.

#### Rules and Constraints

- **Do** use PascalCase for method names.
- **Do** use verbs or verb phrases.
- **Do** keep methods focused on one task.
- **Do** prefer returning values over `out` parameters where practical.
- **Do not** use excessive overloading that confuses callers.
- **Do not** make methods longer than necessary.

---

### Annotated Code Examples and Expected Outputs

#### Example: Well-Designed Method

```csharp
using System;

class Order
{
    public decimal CalculateTotal(decimal subtotal, decimal taxRate) =>
        subtotal * (1 + taxRate);

    public void PrintTotal(decimal total) =>
        Console.WriteLine($"Total: {total:C}");
}

class Program
{
    static void Main()
    {
        var order = new Order();
        decimal total = order.CalculateTotal(100m, 0.2m);
        order.PrintTotal(total);   // Total: $120.00
    }
}
```

**Expected Output**

```
Total: $120.00
```

**Why this output occurs**

- `CalculateTotal` is focused, uses a verb phrase, and returns a value; `PrintTotal` is a void method with a single expression.

---

### Real-World Cases

- **API design** — Public methods follow naming and parameter guidelines.
- **Code reviews** — Method size and naming are common review criteria.

---

### References

- C# identifier naming rules and conventions — Microsoft Learn - https://learn.microsoft.com/en-us/dotnet/csharp/fundamentals/coding-style/identifier-names
- Names of type members — Framework Design Guidelines - https://learn.microsoft.com/en-us/dotnet/standard/design-guidelines/names-of-type-members
- C# coding conventions — Microsoft Learn - https://learn.microsoft.com/en-us/dotnet/csharp/fundamentals/coding-style/coding-conventions

---

## Deprecated, Unsafe, Implementation-Dependent, and Version-Specific Notes

| Feature | Status / Note |
|---------|---------------|
| Local functions | Introduced in **C# 7.0**; static local functions in **C# 8.0**. |
| Expression-bodied methods | Introduced in **C# 6.0** for methods; expanded to constructors, finalizers, accessors. |
| Extension methods | Introduced in **C# 3.0**; `this ref` extension methods in **C# 7.2**. |
| `ref readonly` parameters | C# 12.0. |
| `scoped` modifier | C# 11.0. |
| `abstract` / `virtual` / `sealed` methods | Core since C# 1.0. |
| Static methods vs. instance performance | **Implementation-dependent**; static methods can be slightly faster to invoke. |
| Return type in signature | **Not** part of signature for overloading; **is** part for delegate compatibility. |

---

## References (Consolidated)

- Methods in C# — Microsoft Learn - https://learn.microsoft.com/en-us/dotnet/csharp/methods
- Methods — C# language specification - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/language-specification/classes#156-methods
- Method signatures — Methods in C# — Microsoft Learn - https://learn.microsoft.com/en-us/dotnet/csharp/methods#method-signatures
- Method invocation — Methods in C# — Microsoft Learn - https://learn.microsoft.com/en-us/dotnet/csharp/methods#method-invocation
- Overloading — Methods in C# — Microsoft Learn - https://learn.microsoft.com/en-us/dotnet/csharp/methods#method-signatures
- Overriding — Methods in C# — Microsoft Learn - https://learn.microsoft.com/en-us/dotnet/csharp/methods#inherited-and-overridden-methods
- Static methods — C# language specification - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/language-specification/classes#1563-static-and-instance-methods
- Virtual methods — C# language specification - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/language-specification/classes#1564-virtual-methods
- Sealed methods — C# language specification - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/language-specification/classes#1566-sealed-methods
- Abstract methods — C# language specification - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/language-specification/classes#1567-abstract-methods
- Extension methods — C# programming guide - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/extension-methods
- Local functions — C# programming guide - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/local-functions
- Expression-bodied members — C# programming guide - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/statements-expressions-operators/expression-bodied-members
- Parameters — C# language specification - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/language-specification/classes#1562-parameters
- Return statement — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/statements/jump-statements#the-return-statement
- C# identifier naming rules and conventions — Microsoft Learn - https://learn.microsoft.com/en-us/dotnet/csharp/fundamentals/coding-style/identifier-names
- Names of type members — Framework Design Guidelines - https://learn.microsoft.com/en-us/dotnet/standard/design-guidelines/names-of-type-members
- C# coding conventions — Microsoft Learn - https://learn.microsoft.com/en-us/dotnet/csharp/fundamentals/coding-style/coding-conventions

---