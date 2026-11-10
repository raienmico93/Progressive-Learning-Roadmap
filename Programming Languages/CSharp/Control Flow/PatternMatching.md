# C# Pattern Matching — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
Pattern matching is a C# language feature that lets you test an expression against a **pattern**—a declarative description of the data's type, shape, properties, or value—and, when the match succeeds, **extract information** from that expression. It replaces long sequences of `is` checks, casts, and `if`/`else` chains with concise, readable, and compiler-checked code. C# provides a rich vocabulary of patterns and two primary constructs for using them: the **`is` expression** (for testing and capturing a single value) and the **`switch` expression** (for evaluating a value from multiple candidate patterns).

**Technical Definition**
C# pattern matching is exposed through the `is` expression, the `switch` statement, and the `switch` expression. The language supports multiple pattern kinds: **declaration patterns** (check runtime type and assign to a variable), **type patterns** (check runtime type), **constant patterns** (compare against a compile-time constant), **relational patterns** (compare using `<`, `>`, `<=`, `>=`), **logical patterns** (combine patterns with `not`, `and`, `or`), **property patterns** (test properties/fields against nested patterns), **positional patterns** (deconstruct and test components), **var patterns** (match anything and assign), **discard patterns** (`_` matches anything), and **list patterns** (C# 11, match a sequence's shape). Logical, property, positional, and list patterns are **recursive**—they can contain nested patterns. A declaration pattern with type `T` matches when the expression is non-null and the runtime type has an identity conversion to `T`, derives from `T`, implements interface `T`, or another implicit reference conversion exists. Pattern matching can dramatically reduce boilerplate: a single `switch` expression can replace dozens of lines of `if`/`else if` with casts and temporary variables.

**Beginner-Friendly Explanation**
Pattern matching is a **declarative way to ask “what does this data look like?”** and **“what can I learn from it?”** Instead of writing `if (obj is Circle) { var c = (Circle)obj; ... }`, you write `if (obj is Circle c) { ... }` and immediately use `c`. In a `switch` expression, you can describe multiple shapes—`Circle c`, `Square s`, `null`, `> 100`—and C# selects the first match and gives you the extracted values. It makes complex branching logic read like a set of rules rather than a pile of imperative checks.

---

### Key Characteristics

- **Two primary constructs** — The `is` expression (single test + capture) and the `switch` expression (multi-way, value-producing dispatch).
- **Recursive patterns** — Logical, property, positional, and list patterns can contain nested patterns, enabling deep structural matching.
- **Compile-time safety** — The compiler checks pattern compatibility, subsumption (unreachable arms), and definite assignment of captured variables.
- **Exhaustiveness checking** — `switch` expressions warn about unhandled values and throw at runtime if no arm matches; a discard arm (`_`) makes the expression exhaustive.
- **Evolution across C# versions** — C# 7 introduced type, constant, var, and discard patterns; C# 8 added switch expressions, property, positional, and tuple patterns; C# 9 added relational and logical (`and`, `or`, `not`) patterns; C# 11 added list patterns.
- **Null handling** — Patterns like `{ }` (empty property pattern) and `not null` provide safe null checks, and `is` patterns work even when the type overloads `==`.

---

### Prerequisites

- Understanding of C# types (classes, structs, records, interfaces, enums).
- Familiarity with `is` and `as` operators and casts.
- Knowledge of `switch` statements and expressions.
- Awareness of deconstruction and `Deconstruct` methods (for positional patterns).
- Basic understanding of collections and indexing (for list patterns).

---

### Related Programming Areas

- **Control flow** — Pattern matching replaces many `if`/`else` and `switch` statement scenarios with expression-oriented dispatch.
- **Type-driven algorithms** — Visitor patterns, shape hierarchies, and message handling become cleaner with type and property patterns.
- **Data validation** — Property and relational patterns express complex validation rules declaratively.
- **Functional programming** — `switch` expressions bring expression-oriented style to C#.
- **API design** — Records and positional patterns work together to provide concise, readable domain models.

---

### Core Concepts / Features

1. Pattern Matching (Overarching Concept)
2. `is` Pattern
3. Type Patterns
4. Declaration Patterns
5. Constant Patterns
6. Relational Patterns
7. Logical Patterns
8. Property Patterns
9. Positional Patterns
10. List Patterns (C# 11)
11. `switch` Expressions
12. Pattern Matching Best Practices

Each concept follows a uniform structure: definitions, purposes, syntax rules and structure, annotated code examples with expected outputs, real-world cases, and references.

---

## 1. Pattern Matching (Overarching Concept)

### Definitions

**Core Definition**
Pattern matching is the language feature that tests an expression against a pattern and, on success, can extract values from the expression.

**Technical Definition**
C# uses patterns in `is` expressions, `switch` statements, and `switch` expressions. A pattern is a condition to test an input expression against; the pattern can check the value's **type**, **constant value**, **shape** (properties, positional elements), or **relationship** to other values. When a pattern matches, the code can use information learned by the test—such as a more specific type or a value extracted from an object. Patterns are combined with the `not`, `and`, and `or` logical keywords. The `switch` expression evaluates a single expression from a list of candidate expressions based on the first matching pattern, producing a value rather than executing statements.

**Beginner-Friendly Explanation**
Pattern matching lets you **describe the data you expect** rather than writing step-by-step checks. You say “if this is a `Circle` with a radius greater than 10, give me the circle,” and C# handles the type check, the cast, and the property access in one expression. It is like a smart `if` that also unpacks what it finds.

---

### Purposes

- To replace verbose `is`/cast sequences with concise, safe expressions.
- To enable declarative, data-driven branching logic.
- To support exhaustive dispatch in `switch` expressions.
- To make type hierarchies and record structures easy to inspect.
- To improve readability and reduce boilerplate in validation and classification code.

---

### Syntax Rules and Structure

```
// is expression with pattern
if (expression is pattern)
{
    // pattern variables are in scope here
}

// switch expression
var result = expression switch
{
    pattern1 => value1,
    pattern2 => value2,
    _ => defaultValue
};
```

**Component breakdown**

- `expression` — The input value to test.
- `pattern` — A type, constant, relational, logical, property, positional, or list pattern.
- `=>` — Separates a pattern from the result expression in a switch expression.
- `_` — The discard pattern, matching anything.

#### Syntax Rules

- Patterns can be used in `is` expressions, `case` labels, and `switch` expression arms.
- Pattern variables are in scope only where the pattern matches.
- The compiler checks that patterns are compatible with the input type.
- Switch expression arms are evaluated in order; the first match wins.
- A discard arm or `default` ensures exhaustiveness.

#### Constraints and Limitations

- Constant patterns must be compile-time constants.
- Property patterns require the object reference to be non-null.
- Deeply nested patterns can reduce readability.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: From Imperative to Pattern Matching

```csharp
using System;

class Program
{
    static string Describe(object? obj)
    {
        // Imperative style (before pattern matching)
        // if (obj is int) { int i = (int)obj; return $"int: {i}"; }

        // Pattern matching style
        if (obj is int i)
            return $"int: {i}";
        if (obj is string s)
            return $"string: {s}";
        if (obj is null)
            return "null";
        return "unknown";
    }

    static void Main()
    {
        Console.WriteLine(Describe(42));       // int: 42
        Console.WriteLine(Describe("hello"));  // string: hello
        Console.WriteLine(Describe(null));     // null
    }
}
```

**Expected Output**

```
int: 42
string: hello
null
```

**Why this output occurs**

- The `is` declaration patterns test the runtime type and capture the value in a strongly typed variable, eliminating the separate cast.

---

### Real-World Cases

- **Shape hierarchies** — Dispatching on `Circle`, `Square`, `Triangle` in graphics code.
- **Message handling** — Routing different message types in a messaging system.
- **Validation** — Checking whether a value is null, a specific type, or within a range.
- **Parsing** — Classifying tokens or JSON elements by their runtime type.

---

### References

- Patterns — Pattern matching using the `is` and `switch` expressions — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/patterns
- Pattern matching overview — C# - https://learn.microsoft.com/en-us/dotnet/csharp/fundamentals/functional/pattern-matching
- C# Pattern Matching Explained (2026) — NDepend Blog - https://blog.ndepend.com/c-pattern-matching-explained/

---

## 2. `is` Pattern

### Definitions

**Core Definition**
The `is` pattern tests an expression against a pattern and returns `true` if it matches. When the pattern is a **declaration pattern**, it also assigns the converted value to a new variable.

**Technical Definition**
The `is` expression with a pattern (`expression is pattern`) evaluates to `bool`. If the pattern is a declaration pattern (`T variable`), the variable is definitely assigned only when the pattern matches, and its scope is limited to the enclosing block where the match is true. The `is` pattern does **not** consider user-defined conversions or implicit span conversions; it tests identity, reference, boxing, unboxing, and nullable conversions. It works even when the type overloads `==`, making it safer for null checks than `==` in some cases.

**Beginner-Friendly Explanation**
The `is` pattern is a **smart type check**. Instead of `if (obj is Circle)` followed by `var c = (Circle)obj;`, you write `if (obj is Circle c)` and use `c` immediately. It combines the test and the capture in one step.

---

### Purposes

- To safely test an object's runtime type before using it.
- To capture the converted value in a strongly typed variable.
- To avoid `InvalidCastException` from unsafe casts.
- To perform null checks that are unaffected by operator overloading.
- To introduce temporary variables in complex Boolean expressions.

---

### Syntax Rules and Structure

```
expression   is   pattern
```

**Component breakdown**

- `expression` — The value to test.
- `is` — The pattern-matching operator.
- `pattern` — A type, constant, relational, logical, property, positional, or list pattern.

#### Syntax Rules

- Returns `true` when the pattern matches; otherwise `false`.
- Declaration patterns (`T variable`) assign the variable only on a successful match.
- Pattern variables are in scope only where the pattern is true.
- Works with `null` checks (`is null`, `is not null`).

#### Constraints and Limitations

- Does not consider user-defined conversions.
- The captured variable cannot be used outside the matching branch.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Declaration Pattern with `is`

```csharp
using System;

class Program
{
    static void Main()
    {
        object greeting = "Hello, World!";

        if (greeting is string message)
        {
            Console.WriteLine(message.ToLower());   // hello, world!
        }
    }
}
```

**Expected Output**

```
hello, world!
```

**Why this output occurs**

- The `is string message` pattern tests the runtime type and captures the string in `message`. The variable is usable inside the `if` block.

---

#### Example 2: Null Check with `is`

```csharp
using System;

class Program
{
    static void Main()
    {
        string? message = null;

        if (message is null)
            Console.WriteLine("No message");   // this branch

        if (message is not null)
            Console.WriteLine(message);
    }
}
```

**Expected Output**

```
No message
```

**Why this output occurs**

- `is null` and `is not null` use constant and logical patterns to test for null safely.

---

### Real-World Cases

- **Event handlers** — Checking whether `sender` is a specific control type.
- **Plugin systems** — Testing whether an object implements a required interface.
- **Visitor patterns** — Dispatching based on runtime type with captured variables.
- **Input validation** — Extracting typed values from `object` parameters.

---

### References

- The `is` operator — Type-testing operators and cast expressions — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/type-testing-and-cast#the-is-operator
- Pattern matching overview — `is` expression — C# - https://learn.microsoft.com/en-us/dotnet/csharp/fundamentals/functional/pattern-matching#is-expression

---

## 3. Type Patterns

### Definitions

**Core Definition**
A type pattern tests whether an expression's runtime type is compatible with a specified type.

**Technical Definition**
A type pattern (`T`) matches an expression when the expression result is non-null and the runtime type has an identity conversion to `T`, derives from `T`, implements interface `T`, or another implicit reference conversion exists. A **declaration pattern** (`T variable`) extends the type pattern by also declaring and assigning a variable when the match succeeds. The type pattern without a variable is useful when only the type check is needed, not the captured value.

**Beginner-Friendly Explanation**
A type pattern asks **“is this a `Circle` (or something derived from it)?”** If you also want to use the object afterward, use a declaration pattern (`Circle c`) to capture it.

---

### Purposes

- To test runtime type compatibility without a cast.
- To branch on the concrete type of an object.
- To combine type checks with variable capture (via declaration patterns).
- To replace `obj.GetType() == typeof(T)` with a more flexible `is` check that respects inheritance.

---

### Syntax Rules and Structure

```
TypeName
TypeName   variableName
```

**Component breakdown**

- `TypeName` — The type to test against.
- `variableName` — Optional variable to capture the converted value (declaration pattern).

#### Syntax Rules

- Matches when the runtime type is compatible with `T`.
- The variable is definitely assigned only when the pattern matches.
- Does not consider user-defined conversions.

#### Constraints and Limitations

- For value types, `is` checks the exact boxed type or nullable `HasValue`.
- The captured variable is scoped to the matching branch.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Type Test Without Capture

```csharp
using System;

class Program
{
    static void Main()
    {
        object value = 42;
        Console.WriteLine(value is int);     // True
        Console.WriteLine(value is string);  // False
    }
}
```

**Expected Output**

```
True
False
```

**Why this output occurs**

- The boxed `int` is compatible with `int`, but not with `string`.

---

#### Example 2: Type Pattern in a Switch Expression

```csharp
using System;

class Shape { }
class Circle : Shape { public double Radius { get; set; } }
class Square : Shape { public double Side { get; set; } }

class Program
{
    static string Describe(Shape shape) => shape switch
    {
        Circle c => $"Circle (r={c.Radius})",
        Square s => $"Square (s={s.Side})",
        _ => "Unknown shape"
    };

    static void Main()
    {
        Console.WriteLine(Describe(new Circle { Radius = 5 }));  // Circle (r=5)
        Console.WriteLine(Describe(new Square { Side = 3 }));   // Square (s=3)
    }
}
```

**Expected Output**

```
Circle (r=5)
Square (s=3)
```

**Why this output occurs**

- The type patterns `Circle c` and `Square s` match the runtime types and capture the values.

---

### Real-World Cases

- **Shape hierarchies** — Dispatching on concrete shape types.
- **Message handling** — Routing different message subclasses.
- **Plugin discovery** — Checking for specific interfaces.

---

### References

- Type patterns — Patterns — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/patterns#type-patterns
- Declaration and type patterns — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/patterns#declaration-and-type-patterns

---

## 4. Declaration Patterns

### Definitions

**Core Definition**
A declaration pattern tests an expression's runtime type and, on success, **declares and assigns a new variable** to the converted value.

**Technical Definition**
A declaration pattern has the form `T variable`. It matches an expression when the result is non-null and the runtime type is compatible with `T`. On a match, the variable is assigned the converted value and is definitely assigned only within the scope where the pattern is true. The variable's scope is limited to the enclosing block or switch arm. Declaration patterns are the mechanism that makes `if (obj is Circle c)` work.

**Beginner-Friendly Explanation**
A declaration pattern is a **type check plus capture**. It says: “If this is a `Circle`, give me a variable `c` so I can use it.”

---

### Purposes

- To eliminate the separate cast after a type check.
- To introduce strongly typed variables in conditional logic.
- To simplify complex Boolean expressions with temporary captures.
- To enable safe downcasting without `as` and null checks.

---

### Syntax Rules and Structure

```
TypeName   variableName
```

**Component breakdown**

- `TypeName` — The type to test against.
- `variableName` — The captured variable.

#### Syntax Rules

- Matches when the runtime type is compatible with `T` and non-null.
- The variable is definitely assigned in the true branch.
- The variable is not in scope outside the matching branch.

#### Constraints and Limitations

- The variable cannot be reassigned in a way that changes its pattern relationship.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Capturing a Value

```csharp
using System;

class Program
{
    static void Main()
    {
        object value = "Hello";

        if (value is string s)
        {
            Console.WriteLine(s.Length);   // 5
        }
    }
}
```

**Expected Output**

```
5
```

**Why this output occurs**

- `s` is captured as a `string` and used inside the `if` block.

---

#### Example 2: Declaration Pattern in a Switch

```csharp
using System;

class Program
{
    static string Describe(object obj) => obj switch
    {
        int i => $"int: {i}",
        string s => $"string: {s}",
        _ => "other"
    };

    static void Main()
    {
        Console.WriteLine(Describe(42));       // int: 42
        Console.WriteLine(Describe("hi"));     // string: hi
    }
}
```

**Expected Output**

```
int: 42
string: hi
```

**Why this output occurs**

- Each declaration pattern captures the value in a typed variable.

---

### Real-World Cases

- **Visitor patterns** — Capturing the concrete type for dispatch.
- **Validation** — Extracting and checking typed values.
- **Serialization** — Handling different data types.

---

### References

- Declaration patterns — Patterns — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/patterns#declaration-and-type-patterns

---

## 5. Constant Patterns

### Definitions

**Core Definition**
A constant pattern tests whether an expression result equals a specified **compile-time constant**.

**Technical Definition**
A constant pattern compares the input expression to a compile-time constant using the `==` semantics defined for the type. Supported constants include integer and floating-point literals, `char` and `string` literals, `bool` values, enum values, `const` fields, and `null`. The constant must be compatible with the input type. Constant patterns are the simplest form of pattern matching and are used in `is` expressions (`x is 42`) and `switch` arms (`42 => "answer"`).

**Beginner-Friendly Explanation**
A constant pattern is a **direct comparison** with a fixed value. `case 42:` is a constant pattern. `if (x is null)` uses a constant pattern for `null`.

---

### Purposes

- To match specific literal values.
- To match enum members.
- To match `null`.
- To match `const` values.
- To provide simple, readable case labels.

---

### Syntax Rules and Structure

```
constantExpression
```

**Component breakdown**

- `constantExpression` — A compile-time constant (literal, enum member, `const`, `null`).

#### Syntax Rules

- The constant must be compile-time evaluable.
- Must be compatible with the input type.
- `null` is permitted for reference and nullable types.

#### Constraints and Limitations

- Cannot use runtime values.
- Duplicate constants in a switch cause compile errors.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Constant Patterns for Enums

```csharp
using System;

enum Status { Active, Inactive, Pending }

class Program
{
    static string Describe(Status s) => s switch
    {
        Status.Active => "Active",
        Status.Inactive => "Inactive",
        Status.Pending => "Pending",
        _ => "Unknown"
    };

    static void Main()
    {
        Console.WriteLine(Describe(Status.Active));   // Active
    }
}
```

**Expected Output**

```
Active
```

**Why this output occurs**

- `Status.Active` matches the constant pattern.

---

#### Example 2: Constant Pattern for `null`

```csharp
using System;

class Program
{
    static void Main()
    {
        string? message = null;
        if (message is null)
            Console.WriteLine("No message");   // this branch
    }
}
```

**Expected Output**

```
No message
```

**Why this output occurs**

- `is null` uses the constant pattern for `null`.

---

### Real-World Cases

- **Enum dispatch** — Matching enum members.
- **Command strings** — Matching specific commands.
- **Null checks** — `case null:` in switch statements.

---

### References

- Constant patterns — Patterns — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/patterns#constant-pattern
- C#7: Null, Constant, Type, Discard and Var Patterns — NDepend Blog - https://blog.ndepend.com/c-pattern-matching-explained/

---

## 6. Relational Patterns

### Definitions

**Core Definition**
A relational pattern compares an expression result to a constant using `<`, `>`, `<=`, or `>=`.

**Technical Definition**
A relational pattern uses a relational operator (`<`, `>`, `<=`, `>=`) followed by a constant expression of an integer, floating-point, `char`, or enum type. It matches when the input value satisfies the comparison. Relational patterns **do not** work with types that lack a defined ordering, such as `string`. Multiple relational patterns can be combined with `and` and `or` to express ranges.

**Beginner-Friendly Explanation**
A relational pattern is a **range check**. `> 0` matches any positive number; `>= 0 and < 10` matches a specific range.

---

### Purposes

- To match values against numeric or ordered ranges.
- To classify measurements (e.g., temperatures, scores).
- To replace `if` chains with declarative range matching.
- To combine with logical patterns for complex conditions.

---

### Syntax Rules and Structure

```
<   constant
>   constant
<=  constant
>=  constant
```

**Component breakdown**

- Operator — `<`, `>`, `<=`, or `>=`.
- `constant` — A constant expression of an ordered type.

#### Syntax Rules

- The constant must be an integer, floating-point, `char`, or enum.
- Can be combined with `and` for ranges.
- `null` does not match a relational pattern.

#### Constraints and Limitations

- Cannot be used with `string` (no defined ordering).
- The constant must be compile-time.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Classifying a Measurement

```csharp
using System;

class Program
{
    static string Classify(double measurement) => measurement switch
    {
        < -4.0 => "Too low",
        > 10.0 => "Too high",
        double.NaN => "Unknown",
        _ => "Acceptable"
    };

    static void Main()
    {
        Console.WriteLine(Classify(13));   // Too high
        Console.WriteLine(Classify(-10));  // Too low
    }
}
```

**Expected Output**

```
Too high
Too low
```

**Why this output occurs**

- `13` is greater than `10.0`; `-10` is less than `-4.0`.

---

#### Example 2: Range with `and`

```csharp
using System;

class Program
{
    static string Describe(int value) => value switch
    {
        >= 0 and < 10 => "single digit",
        >= 10 and < 100 => "double digit",
        _ => "large"
    };

    static void Main()
    {
        Console.WriteLine(Describe(5));   // single digit
        Console.WriteLine(Describe(50));  // double digit
    }
}
```

**Expected Output**

```
single digit
double digit
```

**Why this output occurs**

- The relational patterns combined with `and` express ranges.

---

### Real-World Cases

- **Grading** — `>= 90`, `>= 80`.
- **Temperature classification** — `< 0`, `>= 30`.
- **Age ranges** — `>= 18 and < 65`.

---

### References

- Relational patterns — Patterns — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/patterns#relational-patterns
- C# 9: Combinator, Parenthesized and Relational Patterns — NDepend Blog - https://blog.ndepend.com/c-pattern-matching-explained/

---

## 7. Logical Patterns

### Definitions

**Core Definition**
Logical patterns combine other patterns using the `not`, `and`, and `or` keywords.

**Technical Definition**
`not pattern` matches when the negated pattern does **not** match. `pattern1 and pattern2` matches when both patterns match. `pattern1 or pattern2` matches when either pattern matches. `not` binds tighter than `and`, which binds tighter than `or`; parentheses can override this precedence. Logical patterns are **recursive**—they can contain any patterns, including other logical patterns.

**Beginner-Friendly Explanation**
Logical patterns let you **combine tests**. `> 0 and < 10` means “greater than zero AND less than ten.” `not null` means “not null.” `1 or 2 or 3` means “one of these.”

---

### Purposes

- To express ranges (`>= 0 and < 10`).
- To express alternatives (`1 or 2 or 3`).
- To negate patterns (`not null`).
- To build complex conditions declaratively.
- To avoid verbose `when` guards.

---

### Syntax Rules and Structure

```
not   pattern
pattern1   and   pattern2
pattern1   or    pattern2
```

**Component breakdown**

- `not` — Negation.
- `and` — Conjunction.
- `or` — Disjunction.

#### Syntax Rules

- `not` binds tightest, then `and`, then `or`.
- Parentheses can clarify complex combinations.
- Can combine any patterns.

#### Constraints and Limitations

- Overly complex logical patterns reduce readability.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Range and Alternative Patterns

```csharp
using System;

class Program
{
    static string GetCalendarSeason(int month) => month switch
    {
        >= 3 and < 6 => "spring",
        >= 6 and < 9 => "summer",
        >= 9 and < 12 => "autumn",
        12 or (>= 1 and < 3) => "winter",
        _ => "Invalid month"
    };

    static void Main()
    {
        Console.WriteLine(GetCalendarSeason(4));   // spring
        Console.WriteLine(GetCalendarSeason(12));  // winter
    }
}
```

**Expected Output**

```
spring
winter
```

**Why this output occurs**

- `4` matches `>= 3 and < 6`; `12` matches the `or` pattern `12 or (>= 1 and < 3)`.

---

#### Example 2: Negation with `not`

```csharp
using System;

class Program
{
    static void Main()
    {
        string? message = null;
        if (message is not null)
            Console.WriteLine(message);
        else
            Console.WriteLine("No message");   // this branch
    }
}
```

**Expected Output**

```
No message
```

**Why this output occurs**

- `is not null` negates the constant pattern for `null`.

---

### Real-World Cases

- **Season classification** — Month ranges.
- **Validation** — `not null and not empty`.
- **Categorisation** — `1 or 2 or 3`.

---

### References

- Logical patterns — Patterns — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/patterns#logical-patterns
- C# 9: Combinator Patterns — NDepend Blog - https://blog.ndepend.com/c-pattern-matching-explained/

---

## 8. Property Patterns

### Definitions

**Core Definition**
A property pattern tests an expression's **properties or fields** against nested patterns.

**Technical Definition**
A property pattern matches when the expression result is non-null and every nested pattern matches the corresponding property or field. The empty property pattern `{ }` matches anything non-null, serving as a null check. Property patterns can include a type check and variable capture, and they are **recursive**—they can contain any pattern, including other property patterns. The `var` pattern can be used to capture property values.

**Beginner-Friendly Explanation**
A property pattern lets you **look inside an object** and check its properties. `{ Radius: > 10 }` matches an object whose `Radius` is greater than `10`. `{ }` matches anything that is not null.

---

### Purposes

- To match objects based on their property values.
- To combine type checks with property checks.
- To deconstruct complex objects declaratively.
- To avoid nested `if` statements.
- To perform null checks safely.

---

### Syntax Rules and Structure

```
{   Property1:   pattern1,   Property2:   pattern2,   ...   }
```

**Component breakdown**

- `{ }` — The property pattern delimiters.
- `Property: pattern` — A property or field and its nested pattern.

#### Syntax Rules

- The expression must be non-null for properties to be accessed.
- Properties/fields must be accessible.
- Nested patterns can be any pattern.
- The empty pattern `{ }` matches any non-null value.

#### Constraints and Limitations

- Property patterns do not match `null`.
- Computed properties are evaluated.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Matching by Property Values

```csharp
using System;

class Program
{
    static bool IsConferenceDay(DateTime date) =>
        date is { Year: 2020, Month: 5, Day: 19 or 20 or 21 };

    static void Main()
    {
        Console.WriteLine(IsConferenceDay(new DateTime(2020, 5, 20)));   // True
        Console.WriteLine(IsConferenceDay(new DateTime(2021, 5, 20)));   // False
    }
}
```

**Expected Output**

```
True
False
```

**Why this output occurs**

- The property pattern checks `Year`, `Month`, and `Day`.

---

#### Example 2: Property Pattern with `var` Capture

```csharp
using System;

class Rectangle
{
    public double Width { get; set; }
    public double Height { get; set; }
}

class Program
{
    static string Describe(Rectangle r) => r switch
    {
        { Width: var w, Height: var h } when w == h => "Square",
        { Width: > 100 } => "Wide rectangle",
        { } => "Rectangle",
        null => "Null"
    };

    static void Main()
    {
        Console.WriteLine(Describe(new Rectangle { Width = 5, Height = 5 }));   // Square
        Console.WriteLine(Describe(new Rectangle { Width = 150, Height = 20 })); // Wide rectangle
        Console.WriteLine(Describe(null));                                       // Null
    }
}
```

**Expected Output**

```
Square
Wide rectangle
Null
```

**Why this output occurs**

- The property pattern captures `Width` and `Height`, checks equality, and matches wide rectangles; `{ }` matches non-null; `null` is handled explicitly.

---

### Real-World Cases

- **Validation** — Checking object properties against rules.
- **Domain classification** — Matching records by field values.
- **UI logic** — Matching view-model states.

---

### References

- Property patterns — Patterns — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/patterns#property-pattern
- C# 8: Property Patterns — NDepend Blog - https://blog.ndepend.com/c-pattern-matching-explained/

---

## 9. Positional Patterns

### Definitions

**Core Definition**
A positional pattern **deconstructs** an expression result and tests the resulting values against nested patterns.

**Technical Definition**
A positional pattern uses the deconstruction syntax `(pattern1, pattern2, ...)`. It works with types that support deconstruction: tuples, records with positional parameters, and types with a `Deconstruct` method. Each nested pattern is matched against the corresponding deconstructed value. Positional patterns are **recursive** and can include a type check and variable capture. The names of tuple elements and `Deconstruct` parameters can be used in the pattern.

**Beginner-Friendly Explanation**
A positional pattern **unpacks** a tuple or record and checks each part. `(0, 0)` matches a point at the origin; `(x, y)` captures both coordinates.

---

### Purposes

- To match tuples and positional records.
- To deconstruct values and test components.
- To capture multiple extracted values.
- To combine with property and relational patterns.

---

### Syntax Rules and Structure

```
(   pattern1,   pattern2,   ...   )
```

**Component breakdown**

- `(` and `)` — The positional pattern delimiters.
- `pattern1, pattern2` — Nested patterns for each position.

#### Syntax Rules

- The type must support deconstruction (tuple, record, or `Deconstruct` method).
- The number of patterns must match the deconstruction arity.
- Nested patterns can be any pattern.

#### Constraints and Limitations

- Requires the type to be deconstructable.
- Arity must match.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Matching a Point Record

```csharp
using System;

public record Point(int X, int Y);

class Program
{
    static string Describe(Point p) => p switch
    {
        (0, 0) => "Origin",
        (0, _) => "On Y-axis",
        (_, 0) => "On X-axis",
        _ => "Somewhere else"
    };

    static void Main()
    {
        Console.WriteLine(Describe(new Point(0, 0)));   // Origin
        Console.WriteLine(Describe(new Point(0, 5)));   // On Y-axis
        Console.WriteLine(Describe(new Point(3, 4)));   // Somewhere else
    }
}
```

**Expected Output**

```
Origin
On Y-axis
Somewhere else
```

**Why this output occurs**

- The positional pattern deconstructs the record and matches the values.

---

#### Example 2: Positional Pattern with Property Pattern

```csharp
using System;

public record WeightedPoint(int X, int Y)
{
    public double Weight { get; set; }
}

class Program
{
    static bool IsInDomain(WeightedPoint point) =>
        point is (>= 0, >= 0) { Weight: >= 0.0 };

    static void Main()
    {
        Console.WriteLine(IsInDomain(new WeightedPoint(1, 2) { Weight = 3.0 }));   // True
        Console.WriteLine(IsInDomain(new WeightedPoint(-1, 2) { Weight = 3.0 }));  // False
    }
}
```

**Expected Output**

```
True
False
```

**Why this output occurs**

- The positional pattern checks `X` and `Y`, and the nested property pattern checks `Weight`.

---

### Real-World Cases

- **Geometry** — Matching coordinates.
- **Parsing** — Matching tuple results.
- **Domain records** — Matching records by positional values.

---

### References

- Positional patterns — Patterns — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/patterns#positional-pattern
- C# 8: Positional and Tuple Patterns — NDepend Blog - https://blog.ndepend.com/c-pattern-matching-explained/

---

## 10. List Patterns (C# 11)

### Definitions

**Core Definition**
A list pattern (introduced in **C# 11**) matches a **sequence** of elements against a combination of an element-count requirement and nested patterns at ordered positions.

**Technical Definition**
A list pattern describes a sequence's shape by combining an element-count requirement with nested patterns at ordered positions. It tests only the requirements written in the pattern. A list pattern works when the input's compile-time type provides a `Length` or `Count` property and **indexed element access** (`input[index]`). Arrays, `List<T>`, strings, and spans are common supported types. The **slice pattern** `..` matches zero or more elements between specific positions. List patterns use length/count and indexing rather than enumeration.

**Beginner-Friendly Explanation**
A list pattern matches the **shape of a sequence**. `[1, 2, 3]` matches exactly three elements with those values. `[first, .., last]` captures the first and last elements and allows anything in between.

---

### Purposes

- To match sequences by exact length and element values.
- To capture elements at fixed positions.
- To allow an unmatched middle section via the slice pattern `..`.
- To validate headers, rows, and structured data.
- To replace verbose length and index checks.

---

### Syntax Rules and Structure

```
[   pattern1,   pattern2,   ..   ,   patternN   ]
```

**Component breakdown**

- `[` and `]` — The list pattern delimiters.
- `pattern1, pattern2` — Element patterns at specific positions.
- `..` — Slice pattern (matches zero or more elements).
- `patternN` — A pattern at the end.

#### Syntax Rules

- The input type must have `Length` or `Count` and indexed access.
- The empty pattern `[]` matches an empty sequence.
- The slice pattern `..` can appear between element patterns.
- Nested patterns can be any pattern.

#### Constraints and Limitations

- The compile-time type must support `Length`/`Count` and indexing.
- `IEnumerable<T>` alone is not sufficient.
- Requires **C# 11** or later.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Matching a Header Row

```csharp
using System;

class Program
{
    static bool IsHeader(string[] columns) => columns is ["Name", "Score"];

    static void Main()
    {
        Console.WriteLine(IsHeader(["Name", "Score"]));   // True
        Console.WriteLine(IsHeader(["Name", "Age"]));     // False
    }
}
```

**Expected Output**

```
True
False
```

**Why this output occurs**

- The list pattern `["Name", "Score"]` requires exactly two elements with those values.

---

#### Example 2: Capturing First and Last

```csharp
using System;

class Program
{
    static string Describe(int[] numbers) => numbers switch
    {
        [] => "Empty",
        [var first] => $"Single: {first}",
        [var first, .., var last] => $"First: {first}, Last: {last}",
        _ => "Multiple"
    };

    static void Main()
    {
        Console.WriteLine(Describe([1, 2, 3]));   // First: 1, Last: 3
        Console.WriteLine(Describe([42]));        // Single: 42
        Console.WriteLine(Describe([]));          // Empty
    }
}
```

**Expected Output**

```
First: 1, Last: 3
Single: 42
Empty
```

**Why this output occurs**

- The slice pattern `..` allows matching the first and last elements while ignoring the middle.

---

### Real-World Cases

- **CSV header validation** — Matching expected column names.
- **Protocol frames** — Matching start and end markers.
- **Command-line parsing** — Matching argument shapes.

---

### References

- List and slice patterns — C# - https://learn.microsoft.com/en-us/dotnet/csharp/fundamentals/patterns/list-patterns
- List patterns — Patterns — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/patterns#list-patterns
- C# 11 list patterns — Red Hat Developer - https://developers.redhat.com/articles/2023/01/02/c-11-pattern-matching-and-static-abstract-interfaces

---

## 11. `switch` Expressions

### Definitions

**Core Definition**
A `switch` expression (C# 8.0+) evaluates a **single expression** from a list of candidate expressions based on a pattern match with an input expression. It **produces a value**.

**Technical Definition**
The `switch` expression has the form `input switch { pattern1 => expression1, pattern2 => expression2, ... }`. Each arm contains a pattern, an optional `when` case guard, the `=>` token, and an expression. The value of the switch expression is the value of the **first arm whose pattern matches** and whose guard, if present, evaluates to `true`. Arms are evaluated in text order. If no arm matches, a `SwitchExpressionException` (or `InvalidOperationException` on .NET Framework) is thrown. A discard pattern (`_`) is commonly used as the final arm to make the expression **exhaustive**. The compiler warns about unhandled values.

**Beginner-Friendly Explanation**
The `switch` expression is a **compact, value-returning switch**. Instead of writing `case` and `break`, you write `pattern => result`. It is perfect for assignments, return statements, and expression-bodied members. It is like a series of `if` statements that produces a single value.

---

### Purposes

- To select a value based on pattern matching in an expression context.
- To replace verbose `switch` statements that only assign or return a result.
- To support all pattern kinds (constant, relational, type, property, positional, list).
- To encourage exhaustive handling.
- To make branching logic more concise and readable.

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
- `pattern` — Any pattern.
- `when` — Optional case guard.
- `=>` — Separates the pattern from the result.
- `expression` — The value returned on match.

#### Syntax Rules

- Arms are comma-separated.
- First match wins.
- The compiler warns about unhandled values.
- A discard arm (`_`) makes the expression exhaustive.
- Requires **C# 8.0** or later.

#### Constraints and Limitations

- If no arm matches, a runtime exception is thrown.
- Not all patterns are supported in all C# versions.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Basic Switch Expression

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
        Console.WriteLine(ToCardinal(Direction.Left));   // West
    }
}
```

**Expected Output**

```
West
```

**Why this output occurs**

- `Direction.Left` matches, returning `"West"`.

---

#### Example 2: Relational and Logical Patterns in a Switch Expression

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
        Console.WriteLine(Describe(-5));    // Negative
        Console.WriteLine(Describe(5));     // Small positive
        Console.WriteLine(Describe(100));   // Large positive
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

- Relational and logical patterns match the appropriate ranges.

---

### Real-World Cases

- **Mapping enums to display names** — `status switch { Status.Active => "Active", ... }`.
- **Type-based dispatch** — `shape switch { Circle c => ..., Square s => ... }`.
- **Range classification** — `score switch { >= 90 => "A", >= 80 => "B", ... }`.

---

### References

- switch expression — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/switch-expression
- Switch expressions — C# 8.0 new features - https://learn.microsoft.com/en-us/dotnet/csharp/whats-new/csharp-8#switch-expressions
- Pattern matching overview — switch expression — C# - https://learn.microsoft.com/en-us/dotnet/csharp/fundamentals/functional/pattern-matching#switch-expression

---

## 12. Pattern Matching Best Practices

### Definitions

**Core Definition**
Pattern matching best practices are the guidelines that ensure pattern-based code is readable, exhaustive, maintainable, and performant.

**Technical Definition**
Best practices for pattern matching derive from Microsoft's C# documentation, the .NET coding conventions, and community standards. Key recommendations include: **prefer switch expressions when each branch produces a value** and switch statements when branches perform actions; **use the most specific patterns first** because arms are evaluated in order; **always include a discard arm (`_`) or `default`** for exhaustiveness; **use property patterns for null checks** (`{ }` is a non-null check); **prefer declaration patterns over `is` followed by a cast**; **use relational and logical patterns to express ranges concisely**; **avoid deeply nested patterns** that reduce readability; and **use `when` guards sparingly** for conditions that cannot be expressed as patterns alone.

**Beginner-Friendly Explanation**
Best practices are the habits that keep pattern matching code clean: use switch expressions for values and switch statements for actions, put the most specific patterns first, always have a fallback, and use patterns instead of long `if` chains.

---

### Purposes

- To choose the right construct (`is` vs. `switch` expression vs. `switch` statement).
- To ensure all possible inputs are handled.
- To make pattern matching readable and maintainable.
- To avoid subtle ordering bugs.
- To align with .NET conventions and tooling (e.g., IDE0078).

---

### Guidelines and Structure

#### General Guidelines

```
// Prefer switch expressions for value-producing branches
static string Describe(int x) => x switch
{
    < 0 => "Negative",
    0 => "Zero",
    _ => "Positive"
};

// Use switch statements for actions
switch (command)
{
    case "run": Run(); break;
    case "save": Save(); break;
    default: ShowHelp(); break;
}

// Always include a fallback
_ => throw new ArgumentOutOfRangeException()

// Order specific patterns first
case Circle c when c.Radius > 10: ...
case Circle c: ...

// Use property patterns for null checks
if (obj is { }) { ... }   // non-null check

// Prefer declaration patterns over is + cast
if (obj is string s) { ... }   // not: if (obj is string) { var s = (string)obj; }
```

**Component breakdown**

- **Switch expression for values** — Concise, returns a result.
- **Switch statement for actions** — Executes multiple statements.
- **Fallback arm** — Ensures exhaustiveness.
- **Specific-first ordering** — Prevents subsumption.
- **Property patterns for null** — `{ }` is a non-null check.
- **Declaration patterns** — Eliminate separate casts.

#### Rules and Constraints

- **Do** use switch expressions when each branch produces a value.
- **Do** include a `default` or discard arm.
- **Do** order cases from most specific to least specific.
- **Do** use `when` guards for additional conditions.
- **Do** use pattern matching instead of `is`/cast.
- **Do not** use pattern matching for complex logic with many unrelated conditions; consider polymorphism or strategy patterns.
- **Do not** rely on fall-through; use `goto case` only for deliberate sharing.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Choosing Expression vs. Statement

```csharp
using System;

class Program
{
    // Switch expression: produces a value
    static string GetGrade(int score) => score switch
    {
        >= 90 => "A",
        >= 80 => "B",
        >= 70 => "C",
        _ => "F"
    };

    // Switch statement: performs actions
    static void HandleCommand(string cmd)
    {
        switch (cmd)
        {
            case "run":
                Console.WriteLine("Running");
                break;
            case "quit":
                Console.WriteLine("Quitting");
                break;
            default:
                Console.WriteLine("Unknown");
                break;
        }
    }

    static void Main()
    {
        Console.WriteLine(GetGrade(85));   // B
        HandleCommand("run");              // Running
    }
}
```

**Expected Output**

```
B
Running
```

**Why this output occurs**

- The switch expression returns a value; the switch statement executes actions.

---

#### Example 2: Exhaustive Switch Expression

```csharp
using System;

enum State { Idle, Running, Paused, Stopped }

class Program
{
    static string Describe(State state) => state switch
    {
        State.Idle => "Waiting",
        State.Running => "Working",
        State.Paused => "On hold",
        State.Stopped => "Finished",
        _ => throw new ArgumentOutOfRangeException(nameof(state))
    };

    static void Main()
    {
        Console.WriteLine(Describe(State.Running));   // Working
    }
}
```

**Expected Output**

```
Working
```

**Why this output occurs**

- The discard arm throws for unexpected states, making the expression exhaustive and defensive.

---

### Real-World Cases

- **API design** — Using switch expressions for mapping enums to results.
- **State machines** — Switch statements for state transitions.
- **Validation** — Exhaustive pattern matching with defensive fallbacks.

---

### References

- Patterns — Pattern matching using the `is` and `switch` expressions — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/patterns
- Pattern matching overview — C# - https://learn.microsoft.com/en-us/dotnet/csharp/fundamentals/functional/pattern-matching
- IDE0078: Use pattern matching — .NET - https://learn.microsoft.com/en-us/dotnet/fundamentals/code-analysis/style-rules/ide0078
- C# Pattern Matching Explained (2026) — NDepend Blog - https://blog.ndepend.com/c-pattern-matching-explained/

---

## Deprecated, Unsafe, Implementation-Dependent, and Version-Specific Notes

| Feature | Status / Note |
|---------|---------------|
| Type, constant, var, discard patterns | Introduced in **C# 7.0**. |
| Switch expressions | Introduced in **C# 8.0**. |
| Property, positional, tuple patterns | Introduced in **C# 8.0**. |
| Relational and logical patterns (`not`, `and`, `or`) | Introduced in **C# 9.0**. |
| List patterns | Introduced in **C# 11**. |
| `when` guards | Available in switch statements (C# 7.0+) and switch expressions (C# 8.0+); **not allowed in `if` statements**. |
| `var` pattern | Matches even `null`; **not recommended** for skipping null checks. |
| Property patterns and null | Property patterns **do not match `null`**; `{ }` is a non-null check. |
| Exhaustiveness in switch expressions | **Compiler warns** about unhandled values; runtime throws if no arm matches. |

---

## References (Consolidated)

- Patterns — Pattern matching using the `is` and `switch` expressions — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/patterns
- Pattern matching overview — C# - https://learn.microsoft.com/en-us/dotnet/csharp/fundamentals/functional/pattern-matching
- The `is` operator — Type-testing operators and cast expressions — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/type-testing-and-cast#the-is-operator
- switch expression — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/switch-expression
- List and slice patterns — C# - https://learn.microsoft.com/en-us/dotnet/csharp/fundamentals/patterns/list-patterns
- List patterns — Patterns — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/patterns#list-patterns
- IDE0078: Use pattern matching — .NET - https://learn.microsoft.com/en-us/dotnet/fundamentals/code-analysis/style-rules/ide0078
- C# Pattern Matching Explained (2026) — NDepend Blog - https://blog.ndepend.com/c-pattern-matching-explained/
- C# 11 list patterns — Red Hat Developer - https://developers.redhat.com/articles/2023/01/02/c-11-pattern-matching-and-static-abstract-interfaces
- List and Span Pattern Matching — JetBrains Blog - https://blog.jetbrains.com/dotnet/2023/03/14/list-and-span-pattern-matching-using-csharp-11-in-rider-and-resharper/

---