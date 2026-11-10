# C# Parameter Passing — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
Parameter passing is the mechanism by which a caller supplies values (arguments) to a method's parameters, and how those values are made available inside the method body. C# offers a rich set of parameter-passing styles—value, reference, output, input-reference, and parameter arrays—each with distinct semantics for how data flows between caller and callee. Choosing the right style is fundamental to writing correct, efficient, and expressive APIs.

**Technical Definition**
In C#, a method declaration creates a separate declaration space for parameters and type parameters. A method invocation creates a copy, specific to that invocation, of the parameters and local variables of the method; the argument list of the invocation assigns values or variable references to the newly created parameters. C# distinguishes several kinds of parameters: **value parameters** (declared with no modifiers), **input parameters** (`in`), **output parameters** (`out`), **reference parameters** (`ref`), **reference readonly parameters** (`ref readonly`), and **parameter collections** (`params`). A parameter declared with one of the `in`, `ref`, or `out` modifiers is a **by-reference parameter**. The `in`, `out`, `ref`, and `ref readonly` modifiers are part of a method's signature, but the `params` and `scoped` modifiers are not. Parameters with `ref`, `out`, or `this` modifiers cannot have a default argument.

**Beginner-Friendly Explanation**
Parameter passing is **how you hand ingredients to a recipe**. Sometimes you hand over a copy (value parameter): the cook can chop the copy however they like, but your original ingredient is untouched. Sometimes you hand over the actual ingredient with permission to modify it (`ref`): the cook can change it, and you will see the change. Sometimes you hand over an empty container and ask the cook to fill it (`out`): the cook must put something in it before returning. Sometimes you hand over a large ingredient and say “please do not change it, but you can look at it” (`in`): this avoids the cost of copying a large item. And sometimes you hand over a shopping list of many items (`params`): the method can accept any number of arguments. Named arguments and optional parameters add convenience: you can label your ingredients or skip the optional ones.

---

### Key Characteristics

- **Six parameter kinds** — Value (default), `ref`, `out`, `in`, `ref readonly`, and `params`.
- **Signature rules** — `ref`, `out`, `in`, and `ref readonly` are part of the method signature; `params` and `scoped` are not.
- **Definite assignment** — `out` parameters must be assigned by the method before it returns; `ref` arguments must be definitely assigned before the call.
- **No default for by-reference** — `ref`, `out`, and `this` parameters cannot have default values.
- **`params` flexibility** — The `params` parameter must be last; only one `params` keyword is permitted per method. Since C# 13, `params` supports collection types beyond arrays.
- **Optional and named** — Optional parameters must appear after required parameters; named arguments can be combined with positional arguments if positional arguments remain in their correct positions.

---

### Prerequisites

- Understanding of C# methods and method declarations.
- Familiarity with value types vs. reference types.
- Knowledge of definite assignment rules.
- Awareness of method overloading and signature rules.
- Basic understanding of collections and arrays (for `params`).

---

### Related Programming Areas

- **API design** — Parameter modifiers shape public contracts and performance characteristics.
- **Performance engineering** — `in` and `ref readonly` avoid copying large structs; `ref` enables in-place mutation.
- **Interoperability** — `out` parameters are common in `TryParse`-style APIs and COM interop.
- **Functional programming** — `in` parameters support immutable, efficient data passing.
- **Convenience APIs** — Optional and named arguments improve call-site readability.

---

### Core Concepts / Features

1. Value Parameters
2. Reference Parameters (`ref`)
3. Output Parameters (`out`)
4. Input Reference Parameters (`in`)
5. `ref readonly` Parameters
6. Parameter Arrays (`params`)
7. Optional Parameters
8. Named Arguments
9. Default Values
10. Parameter Best Practices

Each concept follows a uniform structure: definitions, purposes, syntax rules and structure, annotated code examples with expected outputs, real-world cases, and references.

---

## 1. Value Parameters

### Definitions

**Core Definition**
A value parameter is the default parameter kind. The method receives a **copy** of the argument's value, and modifications to the parameter do not affect the caller's variable.

**Technical Definition**
A parameter declared with no modifiers is a value parameter. A value parameter is a local variable that gets its initial value from the corresponding argument supplied in the method invocation. The corresponding argument shall be an expression that is implicitly convertible to the parameter type. A method is permitted to assign new values to a value parameter; such assignments only affect the local storage location represented by the value parameter—they have no effect on the actual argument given in the method invocation.

**Beginner-Friendly Explanation**
A value parameter is a **photocopy**. The method gets its own copy of the value. If it writes on the copy, your original is untouched.

---

### Purposes

- To pass data into a method safely without risk of side effects.
- To allow the method to treat the parameter as a local variable.
- To support immutable, predictable APIs.
- To avoid unintended mutation of caller state.

---

### Syntax Rules and Structure

```
returnType MethodName(type parameterName)
{
    // parameterName is a local copy
}
```

**Component breakdown**

- `type` — The parameter type.
- `parameterName` — The local name inside the method.
- No modifier — This is the default.

#### Syntax Rules

- The argument must be implicitly convertible to the parameter type.
- The method can reassign the parameter without affecting the caller.
- For reference types, the **reference** is copied, not the object; the method can mutate the object but cannot reassign the caller's reference.

#### Constraints and Limitations

- Large structs are copied, which can be expensive.
- Mutations to the object (for reference types) are visible to the caller.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Value Parameter with Reassignment

```csharp
using System;

class Program
{
    static void Modify(int x)
    {
        x = 100;          // only the local copy changes
        Console.WriteLine(x); // 100
    }

    static void Main()
    {
        int value = 5;
        Modify(value);
        Console.WriteLine(value); // 5 (unchanged)
    }
}
```

**Expected Output**

```
100
5
```

**Why this output occurs**

- `x` is a copy of `value`; assigning `100` to `x` does not change `value`.

---

#### Example 2: Reference Type Value Parameter

```csharp
using System;

class Box { public int Value; }

class Program
{
    static void Mutate(Box b)
    {
        b.Value = 99;       // mutates the shared object
        b = new Box();      // reassigns the local reference only
        b.Value = 1;
    }

    static void Main()
    {
        var box = new Box { Value = 10 };
        Mutate(box);
        Console.WriteLine(box.Value); // 99
    }
}
```

**Expected Output**

```
99
```

**Why this output occurs**

- The reference is copied, so `b.Value = 99` mutates the shared object. Reassigning `b` does not affect the caller's `box`.

---

### Real-World Cases

- **Mathematical functions** — `Math.Abs(int value)` does not need to modify the argument.
- **Immutable APIs** — Methods that compute new values without mutating inputs.
- **Default choice** — Most parameters in well-designed APIs are value parameters.

---

### References

- Value parameters — C# language specification - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/language-specification/classes#15622-value-parameters
- Method parameters — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/method-parameters

---

## 2. Reference Parameters (`ref`)

### Definitions

**Core Definition**
A `ref` parameter passes a **reference** to the caller's variable. The method can read and write the same storage location as the caller.

**Technical Definition**
A parameter declared with the `ref` modifier is a reference parameter. The corresponding argument must be a variable with the same type as the parameter, and it must be definitely assigned before the call. Inside the method, the parameter acts as an alias for the caller's variable; assignments to the parameter are visible to the caller. The `ref` modifier is part of the method signature.

**Beginner-Friendly Explanation**
A `ref` parameter is a **shared notebook**. You and the method write in the same notebook. Whatever the method writes, you see.

---

### Purposes

- To allow a method to modify the caller's variable.
- To avoid copying large structs while allowing mutation.
- To implement swap methods and in-place algorithms.
- To return multiple values via mutation.

---

### Syntax Rules and Structure

```
returnType MethodName(ref type parameterName)
{
    // parameterName aliases the caller's variable
}

// Call site
MethodName(ref variable);
```

**Component breakdown**

- `ref` — The modifier (required at declaration and call site).
- `type` — The parameter type.
- `parameterName` — The alias name.

#### Syntax Rules

- The argument must be a variable (not a constant or expression).
- The argument must be definitely assigned before the call.
- The `ref` modifier must appear at both the declaration and the call site.
- `ref` parameters cannot have default values.

#### Constraints and Limitations

- Cannot pass properties or indexers without a backing field (in older C#; `ref` returns and `ref` locals extend this).
- Cannot be used with literals or computed expressions.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Swapping Two Variables

```csharp
using System;

class Program
{
    static void Swap(ref int a, ref int b)
    {
        int temp = a;
        a = b;
        b = temp;
    }

    static void Main()
    {
        int x = 1, y = 2;
        Swap(ref x, ref y);
        Console.WriteLine($"{x}, {y}"); // 2, 1
    }
}
```

**Expected Output**

```
2, 1
```

**Why this output occurs**

- `a` and `b` alias `x` and `y`; swapping them swaps the caller's variables.

---

#### Example 2: Large Struct Mutation

```csharp
using System;

struct BigStruct { public int A, B, C, D, E, F, G, H; }

class Program
{
    static void Reset(ref BigStruct s)
    {
        s = default;   // zeroes the caller's struct in place
    }

    static void Main()
    {
        var big = new BigStruct { A = 1, B = 2 };
        Reset(ref big);
        Console.WriteLine(big.A); // 0
    }
}
```

**Expected Output**

```
0
```

**Why this output occurs**

- `ref` avoids copying the large struct and allows the method to reset it in place.

---

### Real-World Cases

- **Swap utilities** — `Interlocked.Exchange` patterns.
- **In-place algorithms** — Sorting, shuffling.
- **Interop** — Passing buffers by reference to native code.

---

### References

- Reference parameters — C# language specification - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/language-specification/classes#156233-reference-parameters
- ref keyword — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/ref

---

## 3. Output Parameters (`out`)

### Definitions

**Core Definition**
An `out` parameter is a reference parameter that the method **must assign** before returning. The caller does not need to initialize the variable before the call.

**Technical Definition**
A parameter declared with the `out` modifier is an output parameter. The corresponding argument must be a variable, but it need not be definitely assigned before the call. The method must definitely assign the `out` parameter before it returns. Inside the method, the parameter is treated as unassigned until the method assigns it. `out` parameters are commonly used to return multiple values from a method.

**Beginner-Friendly Explanation**
An `out` parameter is an **empty container** you hand to the method. The method promises to put something inside before it returns. You can declare the variable right in the call.

---

### Purposes

- To return multiple values from a method.
- To implement `TryParse`-style APIs (success flag + result).
- To avoid exceptions for expected failure cases.
- To support APIs that compute results without requiring pre-initialization.

---

### Syntax Rules and Structure

```
returnType MethodName(out type parameterName)
{
    parameterName = value; // must assign before return
}

// Call site (variable can be declared inline)
MethodName(out type variable);
```

**Component breakdown**

- `out` — The modifier (required at declaration and call site).
- `type` — The parameter type.
- `parameterName` — The output variable.

#### Syntax Rules

- The argument must be a variable.
- The argument need not be initialized before the call.
- The method must assign the parameter on all paths before returning.
- The `out` modifier must appear at both the declaration and the call site.
- `out` parameters cannot have default values.

#### Constraints and Limitations

- Cannot be used with properties that lack a setter.
- The variable is considered unassigned inside the method until explicitly assigned.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: `TryParse` Pattern

```csharp
using System;

class Program
{
    static void Main()
    {
        string input = "123";
        if (int.TryParse(input, out int result))
        {
            Console.WriteLine(result); // 123
        }
    }
}
```

**Expected Output**

```
123
```

**Why this output occurs**

- `out int result` declares the variable inline; `TryParse` assigns it only on success.

---

#### Example 2: Returning Multiple Values

```csharp
using System;

class Program
{
    static void Divide(int dividend, int divisor, out int quotient, out int remainder)
    {
        quotient = dividend / divisor;
        remainder = dividend % divisor;
    }

    static void Main()
    {
        Divide(10, 3, out int q, out int r);
        Console.WriteLine($"{q}, {r}"); // 3, 1
    }
}
```

**Expected Output**

```
3, 1
```

**Why this output occurs**

- Both `out` parameters are assigned by the method and returned to the caller.

---

### Real-World Cases

- **Parsing** — `int.TryParse`, `DateTime.TryParse`.
- **Math** — `Math.DivRem`.
- **Interop** — COM methods with output parameters.

---

### References

- Output parameters — C# language specification - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/language-specification/classes#156234-output-parameters
- out keyword — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/out-parameter-modifier
- Method parameters and modifiers — C# reference (out) - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/method-parameters

---

## 4. Input Reference Parameters (`in`)

### Definitions

**Core Definition**
An `in` parameter passes a **read-only reference** to the caller's variable, avoiding the copy cost of value parameters while preventing modification.

**Technical Definition**
A parameter declared with the `in` modifier is an input reference parameter. It is a by-reference parameter that cannot be written to inside the method. The `in` modifier is part of the method signature. The `in` modifier is required at the declaration but **optional at the call site**; the compiler can create a temporary variable when necessary (e.g., for literals or property values) and pass a read-only reference to it. `in` parameters are primarily a performance optimization for passing large structs without copying.

**Beginner-Friendly Explanation**
An `in` parameter is a **glass display case**. The method can look at the value and read it, but it cannot change it. It is useful for large items that would be expensive to copy.

---

### Purposes

- To pass large structs efficiently without copying.
- To guarantee that the method does not modify the argument.
- To avoid defensive copies in performance-sensitive code.
- To express clear read-only intent in APIs.

---

### Syntax Rules and Structure

```
returnType MethodName(in type parameterName)
{
    // read-only access to parameterName
}

// Call site (modifier optional)
MethodName(in variable);
MethodName(variable);   // also valid
```

**Component breakdown**

- `in` — The modifier (required at declaration, optional at call site).
- `type` — The parameter type.
- `parameterName` — The read-only reference.

#### Syntax Rules

- The method cannot assign to an `in` parameter (compile error CS8331).
- The call site may omit the `in` modifier; the compiler may create a temporary if the argument is not a variable.
- `in` is part of the method signature.
- `in` parameters cannot have default values.

#### Constraints and Limitations

- Primarily beneficial for large structs; for small types, the overhead may outweigh the benefit.
- The compiler may create defensive copies if the argument is not a variable, potentially negating the performance benefit.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Read-Only Large Struct

```csharp
using System;

struct LargeStruct { public int A, B, C, D, E, F, G, H; }

class Program
{
    static void Process(in LargeStruct data)
    {
        // data.A = 99; // Compile error: cannot assign to 'in' parameter
        Console.WriteLine(data.A); // 42
    }

    static void Main()
    {
        var large = new LargeStruct { A = 42 };
        Process(in large);
    }
}
```

**Expected Output**

```
42
```

**Why this output occurs**

- `in` allows reading `data.A` without copying the struct; assignment is forbidden.

---

#### Example 2: Call Site Without `in`

```csharp
using System;

struct Point { public int X, Y; }

class Program
{
    static void Show(in Point p) => Console.WriteLine($"{p.X},{p.Y}");

    static void Main()
    {
        var point = new Point { X = 1, Y = 2 };
        Show(point);      // in modifier omitted
    }
}
```

**Expected Output**

```
1,2
```

**Why this output occurs**

- The `in` modifier is optional at the call site.

---

### Real-World Cases

- **Math libraries** — Passing large vectors or matrices without copying.
- **Performance-critical APIs** — Avoiding heap allocations for structs.
- **Read-only contracts** — Expressing that a method will not mutate its input.

---

### References

- Input parameters — C# language specification - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/language-specification/classes#156232-input-parameters
- in parameter modifier — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/in-parameter-modifier
- Method parameters and modifiers — C# reference (in) - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/method-parameters

---

## 5. `ref readonly` Parameters

### Definitions

**Core Definition**
A `ref readonly` parameter passes a **read-only reference** that the caller must pass explicitly with `ref` or `in`. It is a stricter alternative to `in` for APIs that require the argument to be a variable.

**Technical Definition**
The `ref readonly` modifier indicates that a parameter is a reference parameter that is also `readonly`. The `ref readonly` modifier is required at the method declaration; at the call site, either `in` or `ref` can be used. Unlike `in`, `ref readonly` **cannot** be used with expressions that are not variables; the argument must be a variable. This makes it suitable when the API requires a genuine reference and the compiler should not create a temporary. When an API requires that the argument be passed by reference, choose `ref readonly` instead of `in`.

**Beginner-Friendly Explanation**
`ref readonly` is like `in`, but stricter: you must hand over a real variable, not a temporary value. It guarantees the method will not modify it, and it avoids the ambiguity of temporary copies.

---

### Purposes

- To pass a read-only reference when the argument must be a variable.
- To avoid defensive copies that `in` may introduce for non-variable expressions.
- To express a strong read-only contract for reference-required APIs.
- To optimize large struct passing with explicit caller intent.

---

### Syntax Rules and Structure

```
returnType MethodName(ref readonly type parameterName)
{
    // read-only access
}

// Call site
MethodName(in variable);
MethodName(ref variable);
```

**Component breakdown**

- `ref readonly` — The modifier (required at declaration).
- Call site — `in` or `ref` required.

#### Syntax Rules

- The argument must be a variable.
- The method cannot assign to the parameter.
- `ref readonly` is part of the signature.
- Cannot have a default value.

#### Constraints and Limitations

- More restrictive than `in`; cannot accept literals or computed expressions.
- Requires C# 12.0 or later.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: `ref readonly` with Variable Only

```csharp
using System;

struct Big { public int X; }

class Program
{
    static void Read(ref readonly Big b) => Console.WriteLine(b.X);

    static void Main()
    {
        var big = new Big { X = 7 };
        Read(in big);    // valid
        Read(ref big);   // valid
        // Read(new Big()); // compile error: argument must be a variable
    }
}
```

**Expected Output**

```
7
7
```

**Why this output occurs**

- `ref readonly` requires a variable argument; `in` or `ref` must be used at the call site.

---

### Real-World Cases

- **High-performance libraries** — When the API must guarantee a true reference.
- **Interop** — Passing structs by reference without allowing mutation.

---

### References

- ref readonly — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/ref#ref-readonly-parameters
- Method parameters and modifiers — C# reference (ref readonly) - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/method-parameters

---

## 6. Parameter Arrays (`params`)

### Definitions

**Core Definition**
A `params` parameter allows a method to accept a **variable number of arguments** of a specified type. The arguments are collected into an array (or, since C# 13, other collection types).

**Technical Definition**
The `params` keyword lets you specify a method parameter that takes an argument where the number of arguments is variable. No additional parameters are permitted after the `params` keyword in a method declaration, and only one `params` keyword is permitted in a method declaration. The caller can pass a comma-separated list of arguments of the specified type, an array of the specified type, or no arguments at all. Since C# 13, `params` is no longer limited to array types; it can be used with any recognized collection type, including `Span<T>`, `ReadOnlySpan<T>`, and types implementing `IEnumerable<T>` with an `Add` method.

**Beginner-Friendly Explanation**
`params` lets a method accept **any number of arguments** of the same type. It is like a shopping list: you can add as many items as you want, and the method receives them all as a collection.

---

### Purposes

- To provide convenient APIs for variable-length argument lists.
- To simplify calls to methods like `Console.WriteLine` and `string.Format`.
- To support logging, formatting, and aggregation APIs.
- To avoid manual array creation at call sites.

---

### Syntax Rules and Structure

```
returnType MethodName(params type[] parameterName)
{
    // parameterName is an array
}

// Call sites
MethodName(arg1, arg2, arg3);
MethodName(new type[] { arg1, arg2 });
MethodName();   // empty array
```

**Component breakdown**

- `params` — The modifier.
- `type[]` — The element type (or collection type in C# 13+).
- `parameterName` — The collected parameter.

#### Syntax Rules

- `params` must be the **last** parameter.
- Only one `params` parameter per method.
- The caller can omit arguments; the parameter becomes an empty collection.
- Since C# 13, `params` can use `Span<T>`, `ReadOnlySpan<T>`, and other collection types.

#### Constraints and Limitations

- `params` cannot be combined with `ref`, `out`, or `in`.
- `params` is not part of the method signature.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Basic `params` Array

```csharp
using System;

class Program
{
    static void PrintAll(params int[] numbers)
    {
        foreach (int n in numbers)
            Console.Write(n + " ");
        Console.WriteLine();
    }

    static void Main()
    {
        PrintAll(1, 2, 3);          // 1 2 3
        PrintAll(new[] { 4, 5 });   // 4 5
        PrintAll();                  // (empty line)
    }
}
```

**Expected Output**

```
1 2 3 
4 5 
```

**Why this output occurs**

- The arguments are collected into an `int[]`; an empty call produces an empty array.

---

#### Example 2: `params` with `object`

```csharp
using System;

class Program
{
    static void PrintObjects(params object[] items)
    {
        foreach (var item in items)
            Console.Write($"{item} ");
        Console.WriteLine();
    }

    static void Main()
    {
        PrintObjects(1, "hello", 3.14); // 1 hello 3.14
    }
}
```

**Expected Output**

```
1 hello 3.14 
```

**Why this output occurs**

- `params object[]` accepts heterogeneous arguments.

---

### Real-World Cases

- **Logging** — `Log(string format, params object[] args)`.
- **String formatting** — `string.Format`, `Console.WriteLine`.
- **Mathematical aggregation** — `Sum(params int[] values)`.

---

### References

- params keyword — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/params
- Parameter arrays — C# language specification - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/language-specification/classes#15624-parameter-arrays
- params collection types — C# 13 new features - https://learn.microsoft.com/en-us/dotnet/csharp/whats-new/csharp-13#params-collections

---

## 7. Optional Parameters

### Definitions

**Core Definition**
An optional parameter is a parameter that the caller may omit, in which case a **default value** is used.

**Technical Definition**
Optional parameters are defined at the end of the parameter list, after any required parameters. Each optional parameter specifies a default value as part of its declaration. If the caller does not supply an argument for an optional parameter, the default value is used. If a method is called using named arguments or a combination of positional and named arguments, the caller can omit any arguments that follow the last positional argument.

**Beginner-Friendly Explanation**
An optional parameter is a **parameter with a fallback**. If you do not provide a value, the method uses its default.

---

### Purposes

- To simplify common calls by providing sensible defaults.
- To add new parameters to existing APIs without breaking callers.
- To support configuration-style APIs.
- To reduce overload proliferation.

---

### Syntax Rules and Structure

```
returnType MethodName(type required, type optional = defaultValue)
{
    ...
}
```

**Component breakdown**

- `optional` — The parameter name.
- `= defaultValue` — The default value.

#### Syntax Rules

- Optional parameters must come after required parameters.
- Default values must be compile-time constants (or `default` for value types).
- `ref`, `out`, and `this` parameters cannot have default values.

#### Constraints and Limitations

- Changing a default value in a library requires recompiling callers to observe the change.
- Optional parameters affect overload resolution.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Optional Parameter with Default

```csharp
using System;

class Program
{
    static void Greet(string name, string greeting = "Hello")
    {
        Console.WriteLine($"{greeting}, {name}!");
    }

    static void Main()
    {
        Greet("Alice");               // Hello, Alice!
        Greet("Bob", "Hi");           // Hi, Bob!
    }
}
```

**Expected Output**

```
Hello, Alice!
Hi, Bob!
```

**Why this output occurs**

- When `greeting` is omitted, the default `"Hello"` is used.

---

### Real-World Cases

- **Configuration methods** — `Configure(bool enableLogging = true)`.
- **Formatting** — `ToString(string format = "G")`.

---

### References

- Named and optional arguments — C# programming guide - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/named-and-optional-arguments
- Optional parameters — C# language specification - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/language-specification/classes#15621-general

---

## 8. Named Arguments

### Definitions

**Core Definition**
A named argument specifies the argument value by **parameter name** rather than by position, allowing arguments to be supplied in any order.

**Technical Definition**
Named arguments enable you to specify an argument for a parameter by matching the argument with its name rather than with its position in the parameter list. Named arguments can be supplied in any order. When named arguments are mixed with positional arguments, the positional arguments must appear in their correct positions; out-of-order named arguments followed by positional arguments cause a compile error.

**Beginner-Friendly Explanation**
Named arguments let you **label your ingredients**. Instead of remembering the order, you say `name: "Alice"` and the compiler matches it to the right parameter.

---

### Purposes

- To improve call-site readability when parameter names are more meaningful than positions.
- To skip optional parameters without supplying all preceding defaults.
- To reduce errors when many parameters have the same type.
- To make APIs self-documenting.

---

### Syntax Rules and Structure

```
MethodName(parameterName: value, ...)
```

**Component breakdown**

- `parameterName` — The name of the parameter.
- `:` — Separator.
- `value` — The argument value.

#### Syntax Rules

- Named arguments can appear in any order.
- Positional arguments must remain in their correct positions.
- Named arguments cannot precede positional arguments in a way that leaves positional arguments out of order.

#### Constraints and Limitations

- Names must match the declared parameter names.
- Overload resolution considers named arguments.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Named Arguments in Any Order

```csharp
using System;

class Program
{
    static void PrintOrder(string product, int count, decimal price)
    {
        Console.WriteLine($"{product}: {count} @ {price}");
    }

    static void Main()
    {
        PrintOrder(price: 9.99m, product: "Widget", count: 3);
    }
}
```

**Expected Output**

```
Widget: 3 @ 9.99
```

**Why this output occurs**

- The named arguments are matched to parameters regardless of order.

---

#### Example 2: Mixing Positional and Named

```csharp
using System;

class Program
{
    static void Main()
    {
        PrintOrder("Widget", count: 3, price: 9.99m);
    }

    static void PrintOrder(string product, int count, decimal price)
    {
        Console.WriteLine($"{product}: {count} @ {price}");
    }
}
```

**Expected Output**

```
Widget: 3 @ 9.99
```

**Why this output occurs**

- The positional argument `"Widget"` is in the correct first position; the remaining arguments are named.

---

### Real-World Cases

- **COM interop** — Calling Office APIs with many optional parameters.
- **Configuration APIs** — `Configure(options: new Options(), validate: true)`.

---

### References

- Named and optional arguments — C# programming guide - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/named-and-optional-arguments
- Named arguments — C# language specification - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/language-specification/expressions#128102-method-invocations

---

## 9. Default Values

### Definitions

**Core Definition**
A default value is the value assigned to a parameter when the caller omits the corresponding argument.

**Technical Definition**
Default values are specified in the parameter declaration using `= defaultValue`. The value must be a compile-time constant (or `default` for value types). Default values are embedded in the caller's compiled code; changing a default value in a library requires recompiling consuming assemblies to observe the change. Parameters with `ref`, `out`, or `this` modifiers cannot have default values.

**Beginner-Friendly Explanation**
A default value is the **fallback ingredient** used when you do not supply one.

---

### Purposes

- To provide sensible defaults for optional parameters.
- To reduce the number of overloads.
- To support versioning of APIs (with caution).

---

### Syntax Rules and Structure

```
type parameterName = constantExpression
```

**Component breakdown**

- `parameterName` — The parameter.
- `= constantExpression` — The default value.

#### Syntax Rules

- Must be a compile-time constant (or `default`).
- Cannot be used with `ref`, `out`, `in`, or `this`.
- Optional parameters must come last.

#### Constraints and Limitations

- Changing a default value breaks binary compatibility; callers must recompile.
- Default values are not part of the method signature for overloading.

---

### Annotated Code Examples and Expected Outputs

#### Example: Default Value in Action

```csharp
using System;

class Program
{
    static void Log(string message, bool timestamp = false)
    {
        Console.WriteLine(timestamp ? $"{DateTime.Now}: {message}" : message);
    }

    static void Main()
    {
        Log("Hello");              // Hello
        Log("Hello", true);        // <timestamp>: Hello
    }
}
```

**Expected Output**

```
Hello
<timestamp>: Hello
```

**Why this output occurs**

- When `timestamp` is omitted, `false` is used.

---

### Real-World Cases

- **Logging** — `Log(string message, LogLevel level = LogLevel.Info)`.
- **Formatting** — `Format(string format = "G")`.

---

### References

- Named and optional arguments — C# programming guide (default values) - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/named-and-optional-arguments
- Optional parameters — C# language specification - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/language-specification/classes#15621-general

---

## 10. Parameter Best Practices

### Definitions

**Core Definition**
Parameter best practices are the guidelines for choosing the right parameter modifier, keeping signatures clean, and ensuring APIs are safe, efficient, and readable.

**Technical Definition**
Best practices for parameter passing derive from Microsoft's C# coding conventions, the .NET Framework Design Guidelines, and the C# language reference. Key recommendations include: **prefer value parameters by default**; **use `ref` only when mutation is required**; **use `out` for multiple return values**; **use `in` or `ref readonly` for large read-only structs**; **limit the number of parameters** (ideally ≤ 4, using parameter objects for more); **place optional parameters last**; **use named arguments for readability at call sites**; and **avoid `ref`/`out` in public APIs unless the semantics are clear**.

**Beginner-Friendly Explanation**
Best practices are the habits that keep parameter code clean: use the simplest modifier that works, do not use `ref`/`out` unless you really need to, keep parameter lists short, and label your arguments when it helps.

---

### Purposes

- To improve API readability and safety.
- To avoid unnecessary performance overhead.
- To reduce parameter-related bugs.
- To align with .NET conventions.

---

### Guidelines and Structure

#### General Guidelines

```
// Prefer value parameters by default
void Process(int value) { ... }

// Use ref only when mutation is required
void Swap(ref int a, ref int b) { ... }

// Use out for multiple return values
bool TryParse(string s, out int result) { ... }

// Use in for large read-only structs
void Render(in Matrix4x4 matrix) { ... }

// Limit parameters (≤ 4)
void Configure(Options options) { ... }

// Place optional parameters last
void Log(string message, bool timestamp = false) { ... }

// Use named arguments for clarity
SendEmail(to: "a@b.com", subject: "Hi");
```

**Component breakdown**

- **Value by default** — Safest and most predictable.
- **`ref` only for mutation** — Avoid surprising side effects.
- **`out` for multiple returns** — `TryParse` pattern.
- **`in` for large structs** — Avoid copying.
- **Parameter objects** — Group related parameters.

#### Rules and Constraints

- **Do** keep parameter lists short (≤ 4).
- **Do** use `in` or `ref readonly` for large read-only structs.
- **Do** use named arguments for readability.
- **Do not** use `ref`/`out` when a return value suffices.
- **Do not** use excessive optional parameters; consider overloads or options objects.
- **Do not** mix `ref` and `out` in confusing ways.

---

### Annotated Code Examples and Expected Outputs

#### Example: Well-Designed Parameter List

```csharp
using System;

class Program
{
    static void SendEmail(string to, string subject, string body, bool isHtml = false)
    {
        Console.WriteLine($"To: {to}, Subject: {subject}, HTML: {isHtml}");
    }

    static void Main()
    {
        SendEmail(to: "a@b.com", subject: "Hello", body: "Hi there");
    }
}
```

**Expected Output**

```
To: a@b.com, Subject: Hello, HTML: False
```

**Why this output occurs**

- Named arguments clarify the call; the optional `isHtml` uses its default.

---

### Real-World Cases

- **Public API design** — Parameter lists are designed for clarity and versioning.
- **Performance tuning** — Choosing `in` vs. value for structs.
- **Code reviews** — Parameter count and modifier choices are reviewed.

---

### References

- C# identifier naming rules and conventions — Microsoft Learn - https://learn.microsoft.com/en-us/dotnet/csharp/fundamentals/coding-style/identifier-names
- Names of type members — Framework Design Guidelines - https://learn.microsoft.com/en-us/dotnet/standard/design-guidelines/names-of-type-members
- C# coding conventions — Microsoft Learn - https://learn.microsoft.com/en-us/dotnet/csharp/fundamentals/coding-style/coding-conventions
- Method parameters and modifiers — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/method-parameters

---

## Deprecated, Unsafe, Implementation-Dependent, and Version-Specific Notes

| Feature | Status / Note |
|---------|---------------|
| `ref readonly` parameters | Introduced in **C# 12.0**. |
| `params` with collection types | Introduced in **C# 13**; earlier versions require arrays. |
| `in` parameters | Introduced in **C# 7.2**. |
| `scoped` modifier | Introduced in **C# 11.0**. |
| Default values in library APIs | Changing a default value requires **recompiling callers**. |
| `ref`/`out` in public APIs | **Use with caution**; can complicate overload resolution. |
| `out` variable declaration inline | Introduced in **C# 7.0**. |
| `params` + `ref`/`out` | **Not allowed**. |

---

## References (Consolidated)

- Classes — C# language specification (parameters) - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/language-specification/classes#1562-parameters
- Value parameters — C# language specification - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/language-specification/classes#15622-value-parameters
- Reference parameters — C# language specification - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/language-specification/classes#156233-reference-parameters
- Output parameters — C# language specification - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/language-specification/classes#156234-output-parameters
- Input parameters — C# language specification - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/language-specification/classes#156232-input-parameters
- Parameter arrays — C# language specification - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/language-specification/classes#15624-parameter-arrays
- Method parameters and modifiers — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/method-parameters
- ref keyword — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/ref
- out parameter modifier — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/out-parameter-modifier
- in parameter modifier — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/in-parameter-modifier
- params keyword — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/params
- Named and optional arguments — C# programming guide - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/named-and-optional-arguments
- C# 13 params collections — https://learn.microsoft.com/en-us/dotnet/csharp/whats-new/csharp-13#params-collections
- C# coding conventions — Microsoft Learn - https://learn.microsoft.com/en-us/dotnet/csharp/fundamentals/coding-style/coding-conventions
- Names of type members — Framework Design Guidelines - https://learn.microsoft.com/en-us/dotnet/standard/design-guidelines/names-of-type-members

---