# C# Other Operators — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
“Other operators” in C# is the catch-all category for the language’s specialised operators that do not fit neatly into arithmetic, comparison, logical, bitwise, assignment, or member-access groups. This family includes the **ternary conditional operator** (`?:`), the **null-coalescing operators** (`??` and `??=`), the **range** (`..`) and **index-from-end** (`^`) operators, the **lambda operator** (`=>`), the **discard** (`_`), the **`with` expression** for records, and the **`switch` expression**. Together with **operator overloading**, these operators give C# its expressive, modern, and functional flavour.

**Technical Definition**
The C# language specification classifies `?:` as the **conditional operator** and `??` / `??=` as the **null-coalescing operators**; both are distinct from the ternary pattern-matching forms. The `..` and `^` operators are **member access** operators that work with `System.Range` and `System.Index`. The `=>` token is the **lambda operator** and the separator in **expression-bodied members**. The underscore `_` is a **discard**—a placeholder for an intentionally unused value—and also a **discard pattern** in pattern matching. The `with` expression performs **non-destructive mutation** on records (and, since C# 10, structs and anonymous types). The `switch` expression (C# 8.0+) provides **switch-like semantics in an expression context**, evaluating the first matching pattern. **Operator overloading** allows user-defined types to redefine the behaviour of predefined operators via `public static` `operator` methods.

**Beginner-Friendly Explanation**
These are the **“extra tools”** that make C# pleasant to write. The ternary `?:` is a one-line if/else. The `??` and `??=` operators are “use this if that is missing” shortcuts. The `..` and `^` operators let you say “the last three items” or “everything except the first two.” The `=>` arrow is the “goes to” symbol of lambdas. The underscore `_` says “I do not care about this value.” The `with` expression creates a modified copy of a record without changing the original. The `switch` expression returns a value based on which pattern matches. And operator overloading lets your own types behave like numbers or strings. These operators are the difference between verbose, old-style C# and concise, modern C#.

---

### Key Characteristics

- **Expression-oriented** — Most of these operators produce a value, enabling them to be used inside larger expressions.
- **Null-aware** — `??` and `??=` are the primary tools for null-safe fallbacks.
- **Pattern-powered** — The `switch` expression and discard pattern rely on C#’s rich pattern-matching vocabulary.
- **Record-centric** — The `with` expression is the idiomatic way to perform non-destructive mutation on records.
- **Optional overloading** — Operator overloading lets custom types participate in the language’s operator syntax.
- **Version-sensitive** — Several of these operators were introduced in specific C# versions (`??=` in C# 8.0, `switch` expressions in C# 8.0, `with` for structs in C# 10, etc.).

---

### Prerequisites

- Understanding of C# expressions, statements, and operator precedence.
- Familiarity with nullable types and null-handling.
- Knowledge of records and pattern matching (for `with` and `switch` expressions).
- Awareness of delegates and lambdas (for `=>`).
- Basic understanding of operator overloading and user-defined types.

---

### Related Programming Areas

- **Functional programming** — `switch` expressions, lambdas, and `with` expressions bring functional idioms to C#.
- **Null safety** — `??` and `??=` are central to nullable reference types.
- **Data modelling** — Records and `with` expressions support immutable, value-based models.
- **API design** — Operator overloading makes domain types feel like primitives.
- **Collection processing** — Ranges and indices simplify slicing arrays and strings.

---

### Core Concepts / Features

1. Ternary Conditional Operator (`?:`)
2. Null-Coalescing Operator (`??`)
3. Null-Coalescing Assignment (`??=`)
4. Range Operator (`..`)
5. Index-from-End Operator (`^`)
6. Lambda Operator (`=>`)
7. Discard (`_`)
8. `with` Expression
9. `switch` Expression
10. Operator Overloading
11. Operator Best Practices

Each concept follows a uniform structure: definitions, purposes, syntax rules and structure, annotated examples with expected outputs, real-world cases, and references.

---

## 1. Ternary Conditional Operator (`?:`)

### Definitions

**Core Definition**
The conditional operator (`?:`), also known as the **ternary conditional operator**, evaluates a Boolean expression and returns one of two results depending on whether the Boolean expression is `true` or `false`.

**Technical Definition**
The conditional operator is **right-associative** and is an expression, meaning it produces a value. Its syntax is `condition ? consequent : alternative`. The `condition` must evaluate to `bool`. If `condition` is `true`, `consequent` is evaluated and its result becomes the result; otherwise, `alternative` is evaluated. Only one of `consequent` or `alternative` is evaluated. Conditional expressions are **target-typed**: if a target type is known, both branches must be implicitly convertible to it; otherwise, the types of the two branches must be the same or one must be implicitly convertible to the other. The `?:` operator **cannot be overloaded**.

**Beginner-Friendly Explanation**
The ternary is a **one-line if/else that returns a value**. `condition ? valueIfTrue : valueIfFalse` reads as “if the condition is true, use this value; otherwise, use that value.” It is handy for simple choices like `string status = score >= 60 ? "Pass" : "Fail";`.

---

### Purposes

- To choose between two values based on a condition in a single expression.
- To simplify short if/else assignments.
- To enable conditional values inside larger expressions.
- To support target-typed conditional expressions with `int?`, `IEnumerable<T>`, etc.
- To chain conditions in a readable, right-associative manner.

---

### Syntax Rules and Structure

```
condition   ?   consequent   :   alternative
```

**Component breakdown**

- `condition` — A `bool` expression.
- `?` — Introduces the consequent.
- `consequent` — The value returned when `condition` is `true`.
- `:` — Separates the two branches.
- `alternative` — The value returned when `condition` is `false`.

#### Syntax Rules

- `condition` must be a `bool` expression.
- Only one branch is evaluated.
- The operator is right-associative: `a ? b : c ? d : e` is `a ? b : (c ? d : e)`.
- Target-typed conditional expressions require both branches to be implicitly convertible to the target type.

#### Constraints and Limitations

- Cannot be overloaded.
- Both branches must be compatible; otherwise, a cast may be required.
- Not suitable for complex multi-branch logic; prefer `switch` expressions.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Basic Ternary

```csharp
using System;

class Program
{
    static string GetWeatherDisplay(double tempInCelsius) =>
        tempInCelsius < 20.0 ? "Cold." : "Perfect!";

    static void Main()
    {
        Console.WriteLine(GetWeatherDisplay(15));  // Cold.
        Console.WriteLine(GetWeatherDisplay(27));  // Perfect!
    }
}
```

**Expected Output**

```
Cold.
Perfect!
```

**Why this output occurs**

- When the temperature is below 20, the `consequent` (`"Cold."`) is returned; otherwise, the `alternative` (`"Perfect!"`) is returned.

---

#### Example 2: Target-Typed Conditional Expression

```csharp
using System;
using System.Collections.Generic;

class Program
{
    static void Main()
    {
        var rand = new Random();
        var condition = rand.NextDouble() > 0.5;

        int? x = condition ? 12 : null;   // both branches convertible to int?
        IEnumerable<int> xs = x is null
            ? new List<int>() { 0, 1 }
            : new int[] { 2, 3 };

        Console.WriteLine(x is null ? "x is null" : $"x = {x}");
    }
}
```

**Expected Output (varies)**

```
x = 12   (or "x is null", depending on the random condition)
```

**Why this output occurs**

- The target type `int?` allows `12` and `null`; the target type `IEnumerable<int>` allows both `List<int>` and `int[]`.

---

### Real-World Cases

- **Display formatting** — `quantity > 0 ? $"{quantity} items" : "Out of stock"`.
- **API responses** — `user is null ? NotFound() : Ok(user)`.
- **Configuration defaults** — `useCache ? cache : noCache`.

---

### References

- ?: operator — the ternary conditional operator — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/conditional-operator

---

## 2. Null-Coalescing Operator (`??`)

### Definitions

**Core Definition**
The null-coalescing operator (`??`) returns the value of its left-hand operand if it is **not null**; otherwise, it evaluates the right-hand operand and returns its result.

**Technical Definition**
`a ?? b` evaluates `a`. If `a` is non-null, the result is `a` and `b` is **not evaluated**. If `a` is `null`, `b` is evaluated and its result becomes the result. The operator is **right-associative**: `a ?? b ?? c` is `a ?? (b ?? c)`. It works with reference types and nullable value types; the left-hand operand cannot be a non-nullable value type. It can be used with unconstrained type parameters.

**Beginner-Friendly Explanation**
`??` means **“use this value, or if it is missing, use this other value.”** It is the cleanest way to provide a fallback for something that might be `null`.

---

### Purposes

- To provide a default value when an expression is `null`.
- To simplify null-checking code.
- To chain fallbacks safely (`a ?? b ?? c`).
- To support nullable value types and reference types.
- To make null-handling intent explicit.

---

### Syntax Rules and Structure

```
leftOperand   ??   rightOperand
```

**Component breakdown**

- `leftOperand` — A nullable value type or reference type expression.
- `??` — The null-coalescing operator.
- `rightOperand` — The fallback expression, evaluated only if the left is `null`.

#### Syntax Rules

- The left operand must be a nullable type or reference type.
- The right operand is evaluated only when the left is `null`.
- The operator is right-associative.
- The result type is determined by the operands.

#### Constraints and Limitations

- Cannot be used with non-nullable value types on the left.
- Does not perform a runtime type check; it only checks for `null`.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Basic Fallback

```csharp
using System;

class Program
{
    static void Main()
    {
        string? name = null;
        string displayName = name ?? "Guest";
        Console.WriteLine(displayName);   // Guest

        int? score = 42;
        int finalScore = score ?? 0;
        Console.WriteLine(finalScore);    // 42
    }
}
```

**Expected Output**

```
Guest
42
```

**Why this output occurs**

- `name` is `null`, so the fallback `"Guest"` is used.
- `score` has a value, so `42` is returned.

---

#### Example 2: Chaining `??`

```csharp
using System;

class Program
{
    static void Main()
    {
        string? a = null;
        string? b = null;
        string c = "default";

        Console.WriteLine(a ?? b ?? c);   // default
    }
}
```

**Expected Output**

```
default
```

**Why this output occurs**

- The chain is evaluated as `a ?? (b ?? c)`: `a` is null, so `b ?? c` is evaluated; `b` is null, so `c` is used.

---

### Real-World Cases

- **Configuration** — `timeout = config.Timeout ?? DefaultTimeout`.
- **Database values** — `displayName = row.MiddleName ?? row.FirstName`.
- **Caching** — `result = cache[key] ?? Compute(key)`.

---

### References

- ?? and ??= operators — null-coalescing operators — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/null-coalescing-operator

---

## 3. Null-Coalescing Assignment (`??=`)

### Definitions

**Core Definition**
The null-coalescing assignment operator (`??=`) assigns the value of its right-hand operand to its left-hand operand **only if the left-hand operand is `null`**.

**Technical Definition**
`a ??= b` assigns `b` to `a` if `a` is `null`; otherwise, `a` is unchanged and `b` is not evaluated. The left-hand operand must be a variable, a property, or an indexer element. The operator is right-associative. Introduced in **C# 8.0**.

**Beginner-Friendly Explanation**
`??=` means **“if this is empty, fill it with that; otherwise, leave it alone.”** It is perfect for lazy initialisation.

---

### Purposes

- To lazily initialise variables only when they are `null`.
- To avoid redundant assignments.
- To simplify null-check-and-assign patterns.
- To support caching and memoisation.
- To make null-handling code concise.

---

### Syntax Rules and Structure

```
leftHandSide   ??=   rightHandSide
```

**Component breakdown**

- `leftHandSide` — A variable, property, or indexer.
- `??=` — The null-coalescing assignment operator.
- `rightHandSide` — The value to assign if the left is `null`.

#### Syntax Rules

- The left-hand operand must be assignable.
- The right-hand operand is evaluated only when the left is `null`.
- The operator is right-associative.
- Requires C# 8.0 or later.

#### Constraints and Limitations

- Does not work with read-only variables.
- The left-hand operand cannot be a non-nullable value type.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Lazy Initialization

```csharp
using System;
using System.Collections.Generic;

class Program
{
    static void Main()
    {
        List<int>? numbers = null;
        numbers ??= new List<int> { 1, 2, 3 };
        Console.WriteLine(numbers.Count);   // 3
    }
}
```

**Expected Output**

```
3
```

**Why this output occurs**

- `numbers` is `null`, so `??=` assigns a new list.

---

#### Example 2: No Assignment When Non-Null

```csharp
using System;

class Program
{
    static void Main()
    {
        string? name = "Alice";
        name ??= "Guest";
        Console.WriteLine(name);   // Alice
    }
}
```

**Expected Output**

```
Alice
```

**Why this output occurs**

- `name` is not `null`, so the right-hand side is not evaluated.

---

### Real-World Cases

- **Caching** — `cache ??= new Dictionary<string, object>()`.
- **Configuration** — `settings ??= LoadDefaultSettings()`.
- **Lazy loading** — Deferring object creation until first access.

---

### References

- ?? and ??= operators — null-coalescing operators — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/null-coalescing-operator
- Null-coalescing assignment — C# 8.0 new features - https://learn.microsoft.com/en-us/dotnet/csharp/whats-new/csharp-8#null-coalescing-assignment

---

## 4. Range Operator (`..`)

### Definitions

**Core Definition**
The range operator (`..`) specifies a **sub-range** of a sequence. The start is inclusive, and the end is **exclusive**.

**Technical Definition**
The `..` operator creates a `System.Range` value. `start..end` represents the range from index `start` (inclusive) to index `end` (exclusive). Either operand can be omitted (`..end`, `start..`, `..`). Ranges are used with indexers and `Slice` methods on arrays, strings, and spans. The operator relies on `System.Range` and `System.Index`.

**Beginner-Friendly Explanation**
`..` is the **“slice” operator**. `array[1..4]` means “elements from index 1 up to, but not including, index 4.”

---

### Purposes

- To extract a sub-range of a sequence concisely.
- To replace verbose `Substring` or `Array.Copy` calls.
- To support slicing of arrays, strings, and spans.
- To work with `Index` and `Range` types in APIs.

---

### Syntax Rules and Structure

```
start   ..   end
```

**Component breakdown**

- `start` — Optional start index (inclusive).
- `..` — The range operator.
- `end` — Optional end index (exclusive).

#### Syntax Rules

- The start is inclusive; the end is exclusive.
- `[0..^0]` represents the entire sequence.
- Ranges can be open-ended: `[2..]`, `[..5]`, `[..]`.
- The operands can be `int` or `Index` (using `^`).

#### Constraints and Limitations

- The sequence type must support range indexing or `Slice`.
- Out-of-range values throw `ArgumentOutOfRangeException`.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Slicing an Array

```csharp
using System;

class Program
{
    static void Main()
    {
        string[] words = { "first", "second", "third", "fourth", "fifth" };
        var slice = words[1..4];
        Console.WriteLine(string.Join(", ", slice)); // second, third, fourth
    }
}
```

**Expected Output**

```
second, third, fourth
```

**Why this output occurs**

- `[1..4]` includes indices 1, 2, and 3; index 4 is excluded.

---

#### Example 2: Open-Ended Ranges

```csharp
using System;

class Program
{
    static void Main()
    {
        string text = "Hello, World!";
        Console.WriteLine(text[..5]);    // Hello
        Console.WriteLine(text[7..]);    // World!
        Console.WriteLine(text[..]);      // Hello, World!
    }
}
```

**Expected Output**

```
Hello
World!
Hello, World!
```

**Why this output occurs**

- `[..5]` takes from the start to index 5 (exclusive); `[7..]` takes from index 7 to the end; `[..]` takes everything.

---

### Real-World Cases

- **Pagination** — `items[skip..(skip + take)]`.
- **Text processing** — Removing prefixes or suffixes.
- **Binary parsing** — Slicing byte spans.

---

### References

- Explore ranges of data using indices and ranges — C# tutorial - https://learn.microsoft.com/en-us/dotnet/csharp/tutorials/ranges-indexes
- Range operator `..` — Member access operators - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/member-access-operators#range-operator-

---

## 5. Index-from-End Operator (`^`)

### Definitions

**Core Definition**
The index-from-end operator (`^`) indicates that an index is relative to the **end** of a sequence. `^0` is the length of the sequence; `^1` is the last element.

**Technical Definition**
`^n` creates a `System.Index` value that refers to the element `n` positions from the end. For a sequence of length `L`, `^n` is equivalent to `L - n`. `^0` equals `L` (one past the last element). The operator can be used with arrays, strings, and any type that supports `Index` indexing.

**Beginner-Friendly Explanation**
`^` lets you count **from the end**. `array[^1]` is the last element, `array[^2]` is the second-to-last, and so on.

---

### Purposes

- To access elements from the end of a sequence without computing `Length - n`.
- To simplify “last element” and “last N elements” code.
- To combine with ranges (`^` inside `..`).
- To improve readability of end-relative indexing.

---

### Syntax Rules and Structure

```
^   n
```

**Component breakdown**

- `^` — The index-from-end operator.
- `n` — The offset from the end.

#### Syntax Rules

- `^0` is the length of the sequence (one past the last element).
- `^1` is the last element.
- Valid indices: `^1` through `^Length`.
- `^0` is valid only in range contexts (e.g., `[0..^0]`).

#### Constraints and Limitations

- Using `^0` as an element index throws `IndexOutOfRangeException`.
- Requires C# 8.0 or later.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Accessing the Last Element

```csharp
using System;

class Program
{
    static void Main()
    {
        string[] words = { "first", "second", "third" };
        Console.WriteLine(words[^1]);   // third
        Console.WriteLine(words[^2]);   // second
    }
}
```

**Expected Output**

```
third
second
```

**Why this output occurs**

- `^1` refers to the last element, `^2` to the second-to-last.

---

#### Example 2: Combining `^` with Ranges

```csharp
using System;

class Program
{
    static void Main()
    {
        string text = "Hello, World!";
        Console.WriteLine(text[^6..]);   // World!
        Console.WriteLine(text[..^7]);   // Hello,
    }
}
```

**Expected Output**

```
World!
Hello,
```

**Why this output occurs**

- `[^6..]` starts six characters from the end; `[..^7]` ends seven characters before the end.

---

### Real-World Cases

- **Parsing** — Extracting file extensions or trailing tokens.
- **UI** — Getting the most recent entries from a list.
- **Text processing** — Removing a trailing suffix.

---

### References

- Index-from-end operator `^` — Member access operators - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/member-access-operators#index-from-end-operator-
- Explore ranges of data using indices and ranges — C# tutorial - https://learn.microsoft.com/en-us/dotnet/csharp/tutorials/ranges-indexes

---

## 6. Lambda Operator (`=>`)

### Definitions

**Core Definition**
The lambda operator (`=>`) separates the parameter list of a lambda expression from its body. The same token also separates a member declaration from its **expression body** in expression-bodied members.

**Technical Definition**
A **lambda expression** is an anonymous function. Its general form is `(input-parameters) => expression` or `(input-parameters) => { statements }`. The left side specifies parameters; the right side specifies the body. Lambdas can be converted to delegate types (`Func<>`, `Action<>`) or expression trees. The `=>` token is also used in **expression-bodied members** (methods, properties, constructors, finalizers, and accessors) to provide a concise implementation.

**Beginner-Friendly Explanation**
`=>` is pronounced **“goes to.”** In a lambda, `x => x * 2` means “take `x` and go to `x * 2`.” In an expression-bodied method, `public int Double(int x) => x * 2;` means “this method is defined as `x * 2`.”

---

### Purposes

- To create inline anonymous functions for delegates and LINQ.
- To write concise expression-bodied members.
- To pass behaviour as data to higher-order methods.
- To support functional programming patterns in C#.
- To reduce boilerplate for simple methods and properties.

---

### Syntax Rules and Structure

```
// Expression lambda
(parameters)   =>   expression

// Statement lambda
(parameters)   =>   { statements }

// Expression-bodied member
member   =>   expression
```

**Component breakdown**

- `parameters` — The lambda’s input parameters (parentheses optional for a single parameter).
- `=>` — The lambda operator.
- `expression` or `{ statements }` — The lambda body.
- `member` — A method, property, constructor, finalizer, or accessor declaration.

#### Syntax Rules

- A single parameter may omit parentheses: `x => x * 2`.
- Zero parameters use empty parentheses: `() => Console.WriteLine()`.
- The body can be an expression or a block.
- Expression-bodied members can be used for methods, constructors, finalizers, properties, and accessors.
- `=>` is right-associative.

#### Constraints and Limitations

- Lambdas require a target delegate type (or expression tree type).
- Expression-bodied members must be a single expression (or a statement expression for `void` members).
- Finalizers and constructors with expression bodies must use a statement expression.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Lambda in LINQ

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
            Console.WriteLine(n);   // 2, 4
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

#### Example 2: Expression-Bodied Members

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
        Console.WriteLine(c.Area);   // 12.566370614359172
        Console.WriteLine(c);        // Circle(r=2)
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
- **Expression-bodied properties** — `public string FullName => $"{First} {Last}";`.
- **Functional composition** — `Func<int, int> square = x => x * x;`.

---

### References

- Lambda expression (`=>`) operator — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/lambda-operator
- Lambda expressions — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/lambda-expressions

---

## 7. Discard (`_`)

### Definitions

**Core Definition**
The underscore (`_`) serves as a **discard**—a placeholder for a variable that is intentionally unused. It communicates to the compiler and readers that a value is not needed.

**Technical Definition**
A discard is a write-only placeholder. It can be used in **out arguments**, **lambda parameters**, **deconstruction**, and as a **discard pattern** in pattern matching. The compiler treats `_` as a discard only when no valid identifier named `_` is in scope. A discard cannot be read; attempting to read one produces compiler error CS0301. In pattern matching, the discard pattern `_` matches **any** expression, including `null`, without capturing it.

**Beginner-Friendly Explanation**
The underscore says **“I do not care about this value.”** It is useful when a method returns something you do not need, or when you are deconstructing a tuple and only want some of the values. In a `switch` expression, `_` is the catch-all “anything else” arm.

---

### Purposes

- To ignore values you do not need (out parameters, deconstruction).
- To signal intent that a value is intentionally unused.
- To satisfy exhaustiveness in `switch` expressions.
- To avoid compiler warnings about unused variables.
- To match any expression in pattern matching.

---

### Syntax Rules and Structure

```
_   // standalone discard
_   // discard pattern
```

**Component breakdown**

- `_` — The discard token or discard pattern.

#### Syntax Rules

- In `out` arguments: `M(out _)`.
- In lambda parameters: `(_, _) => ...`.
- In deconstruction: `(int _, var answer) = M()`.
- In pattern matching: `_` matches anything.
- `_` cannot be read or assigned.

#### Constraints and Limitations

- If an identifier named `_` is in scope, `_` is treated as a regular variable, not a discard.
- A discard cannot be used as a value.
- Discards are write-only; reading produces CS0301.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Discard in `out` and Deconstruction

```csharp
using System;

class Program
{
    static bool TryParse(string s, out int value) { value = 42; return true; }

    static (int, int) GetPair() => (1, 2);

    static void Main()
    {
        if (TryParse("42", out _))
            Console.WriteLine("Parsed successfully"); // Parsed successfully

        (_, int second) = GetPair();
        Console.WriteLine(second); // 2
    }
}
```

**Expected Output**

```
Parsed successfully
2
```

**Why this output occurs**

- `out _` discards the parsed value; `(_, int second)` discards the first tuple element.

---

#### Example 2: Discard Pattern in `switch`

```csharp
using System;

class Program
{
    static string Describe(int value) => value switch
    {
        1 => "One",
        2 => "Two",
        _ => "Anything else"
    };

    static void Main()
    {
        Console.WriteLine(Describe(3)); // Anything else
    }
}
```

**Expected Output**

```
Anything else
```

**Why this output occurs**

- `_` is the discard pattern, matching any value not handled by earlier arms.

---

### Real-World Cases

- **Parsing** — `int.TryParse(input, out _)` when only success matters.
- **Event handlers** — `(_, _) => { ... }` when parameters are unused.
- **Switch expressions** — `_` as the exhaustive fallback arm.
- **Deconstruction** — Ignoring tuple elements you do not need.

---

### References

- Discard `_` — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/tokens/discard
- Discards and the discard pattern — C# - https://learn.microsoft.com/en-us/dotnet/csharp/fundamentals/functional/discards

---

## 8. `with` Expression

### Definitions

**Core Definition**
A `with` expression creates a **copy** of its operand with the specified properties and fields modified. It is the mechanism for **non-destructive mutation**.

**Technical Definition**
A `with` expression produces a new object by copying the original and then setting the properties or fields listed in the `with` initializer. For `record class` types, the compiler synthesises a clone method and copy constructor; the `with` expression calls the clone method and then sets the modified properties. The result is a **shallow copy**: reference-type properties are copied by reference, so both the original and the copy refer to the same instances. Since **C# 10**, the left-hand operand of a `with` expression can also be a structure type, tuple, or anonymous type.

**Beginner-Friendly Explanation**
`with` lets you say **“give me a copy of this record, but change this one thing.”** The original is untouched; you get a new record with the modification. It is the safe, immutable way to update data.

---

### Purposes

- To perform non-destructive mutation on records.
- To create modified copies without changing the original.
- To support immutable data modelling.
- To simplify “update one property” scenarios.
- To work with structs and anonymous types (C# 10+).

---

### Syntax Rules and Structure

```
operand   with   {   Property1   =   value1,   Property2   =   value2   }
```

**Component breakdown**

- `operand` — A record, struct, tuple, or anonymous type instance.
- `with` — The non-destructive mutation keyword.
- `{ ... }` — The initializer specifying properties to change.

#### Syntax Rules

- The left-hand operand must be a record, struct, tuple, or anonymous type.
- The specified properties must have `init` or `set` accessors.
- The result is a shallow copy.
- For `record struct`, the compiler does not synthesise a copy constructor by default.

#### Constraints and Limitations

- Computed properties are not recalculated unless they are evaluated on access.
- Reference-type properties are shared between the original and the copy.
- Requires C# 9.0 for records; C# 10 for structs and anonymous types.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Modifying a Record

```csharp
using System;

public record Person(string FirstName, string LastName);

class Program
{
    static void Main()
    {
        var p1 = new Person("Alice", "Smith");
        var p2 = p1 with { LastName = "Jones" };

        Console.WriteLine(p1);   // Person { FirstName = Alice, LastName = Smith }
        Console.WriteLine(p2);   // Person { FirstName = Alice, LastName = Jones }
    }
}
```

**Expected Output**

```
Person { FirstName = Alice, LastName = Smith }
Person { FirstName = Alice, LastName = Jones }
```

**Why this output occurs**

- `with` creates a copy of `p1` and changes `LastName`; `p1` is unchanged.

---

#### Example 2: `with` on a Struct (C# 10)

```csharp
using System;

public struct Point
{
    public int X { get; init; }
    public int Y { get; init; }
}

class Program
{
    static void Main()
    {
        var p1 = new Point { X = 1, Y = 2 };
        var p2 = p1 with { Y = 10 };

        Console.WriteLine($"p1: ({p1.X}, {p1.Y})"); // p1: (1, 2)
        Console.WriteLine($"p2: ({p2.X}, {p2.Y})"); // p2: (1, 10)
    }
}
```

**Expected Output**

```
p1: (1, 2)
p2: (1, 10)
```

**Why this output occurs**

- Since C# 10, `with` works on structs; the original `p1` is unchanged.

---

### Real-World Cases

- **State management** — Updating a record in a Redux-style store.
- **DTOs** — Creating modified copies for API responses.
- **Configuration** — Deriving new settings from a base configuration.

---

### References

- `with` expression — Create a modified copy — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/with-expression
- Records — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/record

---

## 9. `switch` Expression

### Definitions

**Core Definition**
A `switch` expression (C# 8.0+) evaluates a **single expression** from a list of candidate expressions based on a **pattern match** with an input expression.

**Technical Definition**
The `switch` expression has the form `input switch { pattern1 => expression1, pattern2 => expression2, ... }`. Each arm consists of a pattern, an optional `when` case guard, the `=>` token, and an expression. The result is the value of the **first matching arm**. The arms are evaluated in text order. The compiler generates an error if a lower arm cannot be reached. If no arm matches, a `SwitchExpressionException` (or `InvalidOperationException` on .NET Framework) is thrown. A discard pattern `_` is commonly used as the final arm to make the expression exhaustive.

**Beginner-Friendly Explanation**
The `switch` expression is a **cleaner, value-returning switch**. Instead of writing `case` and `break`, you write `pattern => result`. It is like a series of `if` statements that produces a single value.

---

### Purposes

- To select a value based on pattern matching in an expression context.
- To replace verbose `switch` statements that only assign a result.
- To support constant, relational, type, property, and discard patterns.
- To enable exhaustive, compiler-checked branching.
- To make branching logic more readable and concise.

---

### Syntax Rules and Structure

```
input   switch
{
    pattern1   =>   expression1,
    pattern2   when   condition   =>   expression2,
    _   =>   defaultExpression
}
```

**Component breakdown**

- `input` — The expression to match.
- `switch` — The keyword.
- `pattern` — A pattern (constant, type, property, relational, discard, etc.).
- `when` — Optional case guard.
- `=>` — Separates the pattern from the result expression.
- `expression` — The value returned when the pattern matches.

#### Syntax Rules

- Arms are separated by commas.
- Arms are evaluated in text order; the first match wins.
- The compiler checks for unreachable arms.
- A discard pattern `_` makes the expression exhaustive.
- Case guards (`when`) add additional conditions.

#### Constraints and Limitations

- If no arm matches, a runtime exception is thrown.
- The compiler may not always prove exhaustiveness; a discard arm is recommended.
- Not all patterns are supported in all C# versions.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Basic `switch` Expression

```csharp
using System;

enum Direction { Up, Down, Left, Right }

class Program
{
    static string ToCardinal(Direction direction) => direction switch
    {
        Direction.Up => "North",
        Direction.Down => "South",
        Direction.Left => "West",
        Direction.Right => "East",
        _ => "Unknown"
    };

    static void Main()
    {
        Console.WriteLine(ToCardinal(Direction.Left)); // West
    }
}
```

**Expected Output**

```
West
```

**Why this output occurs**

- The pattern `Direction.Left` matches, returning `"West"`.

---

#### Example 2: Patterns with Case Guards

```csharp
using System;

class Program
{
    static string Describe(int value) => value switch
    {
        < 0 => "Negative",
        0 => "Zero",
        > 0 and < 10 => "Small positive",
        _ => "Large positive"
    };

    static void Main()
    {
        Console.WriteLine(Describe(-5));  // Negative
        Console.WriteLine(Describe(5));   // Small positive
        Console.WriteLine(Describe(100)); // Large positive
    }
}
```

**Expected Output**

```
Negative
Small positive
Large positive
```

**Why this output occurs**

- Relational patterns and the `and` pattern match the appropriate ranges.

---

### Real-World Cases

- **Mapping enums to display names** — `status switch { Status.Active => "Active", ... }`.
- **Type-based dispatch** — `shape switch { Circle c => ..., Square s => ... }`.
- **Range classification** — `score switch { >= 90 => "A", >= 80 => "B", ... }`.

---

### References

- `switch` expression — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/switch-expression
- Pattern matching overview — C# - https://learn.microsoft.com/en-us/dotnet/csharp/fundamentals/functional/pattern-matching

---

## 10. Operator Overloading

### Definitions

**Core Definition**
Operator overloading allows user-defined types to redefine the behaviour of predefined C# operators, so that instances of those types can be used with familiar operator syntax.

**Technical Definition**
C# allows user-defined types to overload operators by defining **`public static` member functions** using the `operator` keyword. The method must be named `operator X`, where `X` is the operator symbol. At least one parameter must be of the containing type. Overloadable unary operators include `+`, `-`, `!`, `~`, `++`, `--`, `true`, and `false`; overloadable binary operators include `+`, `-`, `*`, `/`, `%`, `&`, `|`, `^`, `<<`, `>>`, `>>>`, `==`, `!=`, `>`, `<`, `>=`, and `<=`. The `&&` and `||` operators cannot be overloaded directly, but can be enabled through `&`, `|`, `true`, and `false` overloads. The assignment operator `=` and the conditional operator `?:` cannot be overloaded.

**Beginner-Friendly Explanation**
Operator overloading lets your own types **behave like built-in types**. If you create a `Money` type, you can define `+` so that `money1 + money2` adds the amounts. It makes your types feel natural to use.

---

### Purposes

- To make domain types behave like primitive types.
- To provide intuitive syntax for mathematical or logical types.
- To enable custom types to participate in expressions.
- To support symmetric operations (e.g., `==` and `!=`).
- To improve readability of code that uses custom types.

---

### Syntax Rules and Structure

```
public   static   ReturnType   operator   op(Parameters)
{
    // implementation
}
```

**Component breakdown**

- `public static` — Operator methods must be public and static.
- `ReturnType` — The result type.
- `operator` — The keyword.
- `op` — The operator symbol (e.g., `+`, `==`).
- `Parameters` — At least one parameter must be of the containing type.

#### Syntax Rules

- Operator methods must be `public static`.
- At least one parameter must be of the containing type.
- Comparison operators must be overloaded in pairs (`==`/`!=`, `<`/`>`, `<=`/`>=`).
- When overloading `==`, also override `Equals` and `GetHashCode`.
- The `&&` and `||` operators are enabled indirectly.

#### Constraints and Limitations

- Cannot overload `=`, `?:`, `&&`, `||`, `[]`, `()`, `.`, `->`, `new`, `typeof`, `as`, `is`, `sizeof`, `nameof`.
- Cannot change operator precedence or associativity.
- Should not throw exceptions from operator implementations.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Overloading `+` for a `Complex` Type

```csharp
using System;

public readonly struct Complex
{
    public double Real { get; }
    public double Imaginary { get; }

    public Complex(double real, double imaginary)
    {
        Real = real;
        Imaginary = imaginary;
    }

    public static Complex operator +(Complex a, Complex b) =>
        new Complex(a.Real + b.Real, a.Imaginary + b.Imaginary);

    public override string ToString() => $"{Real} + {Imaginary}i";
}

class Program
{
    static void Main()
    {
        var a = new Complex(1, 2);
        var b = new Complex(3, 4);
        Console.WriteLine(a + b);   // 4 + 6i
    }
}
```

**Expected Output**

```
4 + 6i
```

**Why this output occurs**

- The overloaded `+` operator adds the real and imaginary parts.

---

#### Example 2: Overloading `==` and `!=`

```csharp
using System;

public readonly struct Point
{
    public int X { get; }
    public int Y { get; }

    public Point(int x, int y) => (X, Y) = (x, y);

    public static bool operator ==(Point a, Point b) => a.X == b.X && a.Y == b.Y;
    public static bool operator !=(Point a, Point b) => !(a == b);

    public override bool Equals(object? obj) => obj is Point p && this == p;
    public override int GetHashCode() => HashCode.Combine(X, Y);
}

class Program
{
    static void Main()
    {
        var p1 = new Point(1, 2);
        var p2 = new Point(1, 2);
        Console.WriteLine(p1 == p2);   // True
    }
}
```

**Expected Output**

```
True
```

**Why this output occurs**

- The overloaded `==` compares `X` and `Y`; `!=` is defined in terms of `==`; `Equals` and `GetHashCode` are overridden for consistency.

---

### Real-World Cases

- **Mathematical types** — `Complex`, `Vector`, `Matrix`, `Money`.
- **Date/time types** — `DateTime` overloads `+` and `-` for `TimeSpan`.
- **Domain primitives** — `Temperature`, `Distance`.

---

### References

- Operator overloading — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/operator-overloading
- Overloadable operators — C# programming guide - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/operator-overloading#overloadable-operators
- Operator Overloads — Framework Design Guidelines - https://learn.microsoft.com/en-us/dotnet/standard/design-guidelines/operator-overloads

---

## 11. Operator Best Practices

### Definitions

**Core Definition**
Operator best practices are the design guidelines that ensure overloaded operators and specialised operator usage remain intuitive, symmetric, and safe.

**Technical Definition**
Microsoft’s Framework Design Guidelines recommend **avoiding operator overloads except in types that should feel like primitive types**, overloading operators **symmetrically** (e.g., `==` with `!=`, `<` with `>`), providing **friendly-named methods** alongside operators, and **not throwing exceptions** from operator implementations. For `??`, `??=`, and `?:`, best practices include using them for null-fallback and simple branching rather than complex logic. For `switch` expressions, a discard arm should usually be included for exhaustiveness.

**Beginner-Friendly Explanation**
Best practices are the habits that keep operator code from becoming confusing: only overload operators on types that really feel like numbers or strings, always overload related operators together, and do not hide complex logic behind a symbol.

---

### Purposes

- To keep operator overloads intuitive and predictable.
- To ensure symmetry and consistency across related operators.
- To prevent exceptions from being thrown from operators.
- To use null-handling operators for clear, simple fallbacks.
- To make `switch` expressions exhaustive and readable.

---

### Guidelines and Structure

#### General Guidelines

```
// Only overload operators on primitive-like types
public readonly struct Money
{
    public decimal Amount { get; }
    public static Money operator +(Money a, Money b) => new(a.Amount + b.Amount);
}

// Overload comparison operators in pairs
public static bool operator ==(Point a, Point b) => ...;
public static bool operator !=(Point a, Point b) => ...;

// Provide friendly-named methods alongside operators
public static Money Add(Money a, Money b) => a + b;

// Use ?? and ??= for null fallbacks
var name = input ?? "Guest";
cache ??= new Dictionary<string, object>();

// Use switch expressions with a discard arm for exhaustiveness
var result = value switch { 1 => "One", _ => "Other" };
```

**Component breakdown**

- **Primitive-like types** — Overload operators only when the type represents a value.
- **Symmetry** — Overload related operators together.
- **Friendly names** — Provide methods like `Add`, `Equals`, `CompareTo`.
- **Null operators** — Use `??` and `??=` for fallbacks.
- **Exhaustive switch** — Include a discard arm `_`.

#### Rules and Constraints

- Do not overload operators on types that do not represent values.
- Do not throw exceptions from operator implementations.
- Do not provide implicit conversions that are lossy.
- Do not use `??` for control flow; use `if`/`switch` for complex logic.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Symmetric Operator Overloading

```csharp
using System;

public readonly struct Temperature
{
    public double Celsius { get; }

    public Temperature(double celsius) => Celsius = celsius;

    public static Temperature operator +(Temperature a, double delta) =>
        new(a.Celsius + delta);

    public static Temperature operator +(double delta, Temperature a) =>
        new(a.Celsius + delta);

    public static bool operator ==(Temperature a, Temperature b) =>
        a.Celsius == b.Celsius;

    public static bool operator !=(Temperature a, Temperature b) => !(a == b);

    public override bool Equals(object? obj) => obj is Temperature t && this == t;
    public override int GetHashCode() => Celsius.GetHashCode();
}
```

**Expected Output (conceptual)**

```
// Symmetric operators, consistent equality, no exceptions
```

**Why this is good practice**

- The `+` operator is defined in both orders for symmetry.
- `==` and `!=` are overloaded together.
- `Equals` and `GetHashCode` are overridden for consistency.

---

#### Example 2: Null-Safe Fallback with `??=`

```csharp
using System;
using System.Collections.Generic;

class Program
{
    static void Main()
    {
        List<string>? names = null;
        names ??= new List<string>();
        names.Add("Alice");
        Console.WriteLine(names.Count); // 1
    }
}
```

**Expected Output**

```
1
```

**Why this is good practice**

- `??=` initialises the list only when it is `null`, avoiding a `NullReferenceException`.

---

### Real-World Cases

- **Mathematical libraries** — Overloading `+`, `-`, `*`, `/` for vectors and matrices.
- **Financial systems** — `Money` with `+`, `-`, `==`, `!=`.
- **Configuration** — Using `??` and `??=` for defaults.
- **Domain logic** — Using `switch` expressions with discard arms.

---

### References

- Operator Overloads — Framework Design Guidelines - https://learn.microsoft.com/en-us/dotnet/standard/design-guidelines/operator-overloads
- Operator overloading — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/operator-overloading
- ?: operator — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/conditional-operator
- ?? and ??= operators — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/null-coalescing-operator
- switch expression — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/switch-expression

---

## Deprecated, Unsafe, Implementation-Dependent, and Version-Specific Notes

| Feature | Status / Note |
|---------|---------------|
| `??=` | Introduced in **C# 8.0**; version-specific. |
| `switch` expression | Introduced in **C# 8.0**; version-specific. |
| `with` on structs | Introduced in **C# 10**; earlier versions require records. |
| Range (`..`) and index-from-end (`^`) | Introduced in **C# 8.0**; require `System.Range` and `System.Index`. |
| Discard `_` | Introduced in **C# 7.0**; expanded in later versions. |
| Operator overloading `>>>` | Overloadable in **C# 11** with `op_UnsignedRightShift`. |
| User-defined compound assignment | **C# 14** allows explicit overloads; earlier versions use implicit overloads. |
| `&&` and `||` overloading | **Cannot be overloaded directly**; enabled via `&`, `|`, `true`, `false`. |
| `?:` overloading | **Cannot be overloaded**. |
| `with` shallow copy | **Language-defined**; reference-type properties are shared. |

---

## References (Consolidated)

- ?: operator — the ternary conditional operator — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/conditional-operator
- ?? and ??= operators — null-coalescing operators — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/null-coalescing-operator
- Explore ranges of data using indices and ranges — C# tutorial - https://learn.microsoft.com/en-us/dotnet/csharp/tutorials/ranges-indexes
- Range operator `..` — Member access operators - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/member-access-operators#range-operator-
- Index-from-end operator `^` — Member access operators - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/member-access-operators#index-from-end-operator-
- Lambda expression (`=>`) operator — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/lambda-operator
- Lambda expressions — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/lambda-expressions
- Discard `_` — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/tokens/discard
- Discards and the discard pattern — C# - https://learn.microsoft.com/en-us/dotnet/csharp/fundamentals/functional/discards
- `with` expression — Create a modified copy — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/with-expression
- Records — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/record
- `switch` expression — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/switch-expression
- Pattern matching overview — C# - https://learn.microsoft.com/en-us/dotnet/csharp/fundamentals/functional/pattern-matching
- Operator overloading — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/operator-overloading
- Overloadable operators — C# programming guide - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/operator-overloading#overloadable-operators
- Operator Overloads — Framework Design Guidelines - https://learn.microsoft.com/en-us/dotnet/standard/design-guidelines/operator-overloads
- Expressions — C# language specification (operator overloading) - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/language-specification/expressions

---