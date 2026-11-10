# C# Conditional Statements — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
Conditional statements are the decision-making constructs of C#. They let a program choose **which path of execution** to take based on whether one or more Boolean expressions evaluate to `true` or `false`. C# provides two main categories: **selection statements** (`if`, `else if`, `else`, and `switch`) and **conditional expressions** (the ternary `?:` operator and `switch` expressions). Together, they form the backbone of all branching logic in C# programs—from simple validation checks to complex pattern-based dispatch.

**Technical Definition**
C# classifies conditional statements as **selection statements** and **conditional expressions**. The `if` statement executes a statement only if a provided Boolean expression evaluates to `true`; the `if-else` form selects one of two code paths based on the Boolean expression. The `switch` statement selects a statement list to execute based on a **pattern match** with an expression. Beginning with **C# 7.0**, `switch` statements support pattern matching (type patterns, constant patterns, `when` guards). Beginning with **C# 8.0**, the `switch` expression evaluates a single expression from a list of candidate expressions based on a pattern match, and the ternary conditional operator (`?:`) evaluates a Boolean expression and returns one of two results. `if` statements can be nested, and `else if` chains are evaluated in order with the **first matching condition winning**. Conditional expressions are **target-typed**, meaning both branches must be implicitly convertible to the target type when one is known.

**Beginner-Friendly Explanation**
Conditional statements are how your program **makes decisions**. Imagine you are a security guard at a building. An `if` statement is a single rule: “If the person has a badge, let them in.” An `if-else` adds a fallback: “If they have a badge, let them in; otherwise, turn them away.” An `else if` chain is a list of rules checked in order: “If they are an employee, let them in; else if they are a visitor with an appointment, let them in; else if they are a delivery person, send them to the loading dock; otherwise, deny entry.” The ternary operator is a quick, one-line rule: “Are you an adult? If yes, ‘allowed’; if no, ‘denied’.” The `switch` statement is a more organised way to handle many specific cases, and the `switch` expression is a modern, compact version that **returns a value**. Pattern matching adds the ability to check **what type** something is or **what shape** it has, and the `when` clause adds extra conditions to a pattern.

---

### Key Characteristics

- **Two main families** — Statement-based (`if`, `else if`, `else`, `switch`) and expression-based (`?:`, `switch` expression).
- **First-match-wins** — In `else if` chains and `switch` statements/expressions, the first matching condition or pattern is selected, and all later conditions are skipped.
- **Pattern matching** — Since C# 7.0, `switch` and `is` expressions can test types, constants, properties, relational ranges, and combinations of patterns.
- **`when` guards** — Additional Boolean conditions can refine a pattern match in `switch` statements and expressions.
- **Expression-oriented** — The ternary operator and `switch` expressions produce values, enabling them to be used inside larger expressions.
- **Target-typed** — Conditional expressions can infer their type from the context, allowing branches of different but compatible types.
- **Exhaustiveness** — `switch` expressions encourage exhaustive handling (often with a discard arm `_`), while `switch` statements allow a `default` case.

---

### Prerequisites

- Understanding of Boolean expressions and comparison operators (`==`, `!=`, `<`, `>`, `&&`, `||`).
- Familiarity with C#’s basic types (`int`, `string`, `bool`, enums).
- Knowledge of variables, assignment, and scope.
- Awareness of classes, structs, and interfaces (for type patterns).
- Basic understanding of expressions vs. statements.

---

### Related Programming Areas

- **Control flow** — Conditional statements are the foundation of branching and looping.
- **Pattern matching** — Type patterns, property patterns, relational patterns, and combinators extend conditional logic.
- **Functional programming** — `switch` expressions and ternary operators bring expression-oriented style to C#.
- **API design** — Enums and pattern matching make APIs self-documenting and safe.
- **Performance** — Choosing between `if`/`else` chains and `switch` can affect readability and, in some cases, execution speed.

---

### Core Concepts / Features

1. `if` Statement
2. `else if` and `else`
3. Nested Conditionals
4. Ternary Operator (`?:`)
5. `switch` Statement
6. `switch` Expression (C# 8)
7. Pattern Matching
8. `when` Clause
9. Conditional Best Practices

Each concept follows a uniform structure: definitions, purposes, syntax rules and structure, annotated examples with expected outputs, real-world cases, and references.

---

## 1. `if` Statement

### Definitions

**Core Definition**
The `if` statement executes a block of code **only when** a specified Boolean expression evaluates to `true`.

**Technical Definition**
An `if` statement executes a statement only if a provided Boolean expression evaluates to `true`. The `if` statement can appear in two forms: **with an `else` part** (selecting one of two statements based on the Boolean expression) and **without an `else` part** (executing its body only when the condition is true). The condition must be a `bool` expression; C# does not implicitly convert integers or other types to `bool`. The body can be a single statement or a block enclosed in curly braces `{}`.

**Beginner-Friendly Explanation**
The `if` statement is a **single rule**: “If this condition is true, do this.” If the condition is false, the code inside the `if` block is simply skipped, and the program continues after it.

---

### Purposes

- To execute code conditionally based on a Boolean expression.
- To validate inputs before proceeding with an operation.
- To guard against invalid states (e.g., null checks, range checks).
- To implement simple branching logic.
- To combine with `else` and `else if` for multi-path decisions.

---

### Syntax Rules and Structure

```
if (condition)
{
    // statements executed when condition is true
}
```

**Component breakdown**

- `if` — The keyword introducing the condition.
- `condition` — A `bool` expression.
- `{ ... }` — The body; executed only when `condition` is `true`.

#### Syntax Rules

- The condition must evaluate to `bool`.
- The body may be a single statement or a block.
- Curly braces are optional for a single statement but **highly recommended**.
- The `if` statement can be nested.

#### Constraints and Limitations

- The condition cannot be an integer or other non-`bool` type; there is no implicit conversion to `bool`.
- Omitting braces for multi-statement bodies leads to logic errors.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Basic `if` Without `else`

```csharp
using System;

class Program
{
    static void DisplayMeasurement(double value)
    {
        if (value < 0 || value > 100)
        {
            Console.Write("Warning: not acceptable value! ");
        }
        Console.WriteLine($"The measurement value is {value}");
    }

    static void Main()
    {
        DisplayMeasurement(45);   // The measurement value is 45
        DisplayMeasurement(-3);   // Warning: not acceptable value! The measurement value is -3
    }
}
```

**Expected Output**

```
The measurement value is 45
Warning: not acceptable value! The measurement value is -3
```

**Why this output occurs**

- For `45`, the condition `value < 0 || value > 100` is `false`, so the warning is skipped.
- For `-3`, the condition is `true`, so the warning is printed before the measurement.

---

#### Example 2: `if` with a Compound Condition

```csharp
using System;

class Program
{
    static void Main()
    {
        int stock = 15;
        bool isAvailable = stock > 0 && stock < 100;

        if (isAvailable)
        {
            Console.WriteLine("Item is available");   // this branch
        }
    }
}
```

**Expected Output**

```
Item is available
```

**Why this output occurs**

- `stock > 0` is `true` and `stock < 100` is `true`, so `isAvailable` is `true`.

---

### Real-World Cases

- **Input validation** — Checking that a value is within an acceptable range before processing.
- **Null guards** — `if (customer != null)` before accessing members.
- **Feature toggles** — `if (featureEnabled)` before executing optional logic.
- **Authentication** — `if (isAuthenticated)` before granting access.

---

### References

- if and switch statements — select a code path to execute — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/statements/selection-statements
- Control code flow with if, else if, and else — Microsoft Learn training - https://learn.microsoft.com/en-us/training/modules/csharp-if-elseif-else/3-control-flow-with-if-else
- if statement — C# language specification - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/language-specification/statements#1385-the-if-statement

---

## 2. `else if` and `else`

### Definitions

**Core Definition**
`else` provides an alternative code path when the `if` condition is `false`. `else if` chains additional conditions, each checked in order, with the **first matching condition winning**.

**Technical Definition**
The `else` part of an `if` statement executes when the `if` condition evaluates to `false`. An `else if` clause adds another condition to check when the preceding conditions are `false`. The chain is evaluated **in order from top to bottom**; the first condition that evaluates to `true` executes its block, and all subsequent conditions are skipped. If no condition matches, the optional final `else` block executes. The `else` clause must come last if present. Ordering conditions from most specific to least specific is recommended to avoid unintended skipping.

**Beginner-Friendly Explanation**
`else` is the **“otherwise”** path. `else if` lets you say “if not this, then maybe that.” C# checks each condition in order and runs **only the first one that is true**. Everything after that is skipped, even if it would also be true.

---

### Purposes

- To handle multiple mutually exclusive outcomes.
- To provide a fallback when no condition is met.
- To replace deeply nested `if` statements with a flat, readable chain.
- To implement classification logic (e.g., grading, categorisation).
- To ensure that exactly one branch executes in a chain.

---

### Syntax Rules and Structure

```
if (condition1)
{
    // ...
}
else if (condition2)
{
    // ...
}
else if (condition3)
{
    // ...
}
else
{
    // fallback
}
```

**Component breakdown**

- `condition1`, `condition2`, … — `bool` expressions checked in order.
- `else if` — Adds an alternative condition.
- `else` — Optional final fallback; must come last.

#### Syntax Rules

- Conditions are evaluated top to bottom; the first `true` wins.
- The `else` clause is optional.
- The `else` clause must be the last in the chain.
- Each block may be a single statement or a brace-enclosed block.

#### Constraints and Limitations

- Overlapping conditions can lead to unexpected results if not ordered carefully.
- Deeply long `else if` chains can be harder to read than a `switch`.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Grading with `else if`

```csharp
using System;

class Program
{
    static string GetGrade(int score)
    {
        if (score >= 90)
            return "A";
        else if (score >= 80)
            return "B";
        else if (score >= 70)
            return "C";
        else
            return "F";
    }

    static void Main()
    {
        Console.WriteLine(GetGrade(85));   // B
        Console.WriteLine(GetGrade(95));   // A
        Console.WriteLine(GetGrade(65));   // F
    }
}
```

**Expected Output**

```
B
A
F
```

**Why this output occurs**

- `85` is not `>= 90`, but it is `>= 80`, so `"B"` is returned. The later conditions are skipped.
- `95` matches the first condition (`>= 90`).
- `65` matches none, so the `else` fallback (`"F"`) runs.

---

#### Example 2: First Match Wins

```csharp
using System;

class Program
{
    static void Main()
    {
        int total = 12;

        if (total >= 16)
            Console.WriteLine("You win a new car!");
        else if (total >= 10)
            Console.WriteLine("You win a new laptop!");   // this branch
        else if (total == 7)
            Console.WriteLine("You win a trip for two!");
        else
            Console.WriteLine("You win a kitten!");
    }
}
```

**Expected Output**

```
You win a new laptop!
```

**Why this output occurs**

- `total` is `12`. The first condition (`>= 16`) is false. The second condition (`>= 10`) is true, so its block executes. The later `else if` (`== 7`) and `else` are **not evaluated**, even though `total` is not `7`.

---

### Real-World Cases

- **Pricing tiers** — Different discounts based on purchase amount.
- **Status codes** — Mapping HTTP status codes to categories.
- **User roles** — Admin, editor, viewer, guest.
- **Tax brackets** — Progressive tax calculation.

---

### References

- Control code flow with if, else if, and else — Microsoft Learn training - https://learn.microsoft.com/en-us/training/modules/csharp-if-elseif-else/3-control-flow-with-if-else
- if and switch statements — select a code path to execute — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/statements/selection-statements
- if statement — C# language specification - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/language-specification/statements#1385-the-if-statement

---

## 3. Nested Conditionals

### Definitions

**Core Definition**
A nested conditional is an `if` statement (or `if-else` chain) placed **inside the code block** of another `if` statement. Nesting is used when one decision **depends on** another decision being true.

**Technical Definition**
C# allows `if` statements to be nested arbitrarily. The inner `if` is evaluated only when the outer `if` condition is `true` and its block is entered. Nested `if` statements can be combined with `else` clauses, and the `else` is associated with the **nearest preceding unmatched `if`** (the “dangling else” rule). Deeply nested conditionals reduce readability and are often refactored into guard clauses, early returns, or separate methods.

**Beginner-Friendly Explanation**
Nesting is **a box inside a box**. The inner `if` only runs if the outer `if` is true. It is useful when you need to check one thing first, and only then check another thing. But too many nested boxes make code hard to follow—it is often better to check the most important condition first and return early.

---

### Purposes

- To model decisions that depend on a prior condition.
- To handle multi-level validation (e.g., check user is logged in, then check role).
- To implement hierarchical business rules.
- To keep related conditions grouped logically.
- To avoid evaluating expensive inner conditions when the outer condition already fails.

---

### Syntax Rules and Structure

```
if (outerCondition)
{
    // outer body
    if (innerCondition)
    {
        // inner body
    }
    else
    {
        // inner else
    }
}
else
{
    // outer else
}
```

**Component breakdown**

- `outerCondition` — The first condition to check.
- `innerCondition` — Checked only if `outerCondition` is `true`.
- `else` — Binds to the **nearest unmatched `if`**.

#### Syntax Rules

- Nesting can be arbitrarily deep.
- The inner `if` runs only when the outer `if` block is entered.
- `else` binds to the closest preceding `if` that does not already have an `else`.
- Curly braces clarify which `if` an `else` belongs to.

#### Constraints and Limitations

- Deep nesting harms readability (often called “arrow code”).
- The dangling-else ambiguity can cause bugs if braces are omitted.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Dependent Decisions

```csharp
using System;

class Program
{
    static void Main()
    {
        bool hasTicket = true;
        int age = 16;

        if (hasTicket)
        {
            if (age >= 18)
            {
                Console.WriteLine("Entry granted.");
            }
            else
            {
                Console.WriteLine("Minors must be accompanied by an adult."); // this branch
            }
        }
        else
        {
            Console.WriteLine("You need a ticket.");
        }
    }
}
```

**Expected Output**

```
Minors must be accompanied by an adult.
```

**Why this output occurs**

- `hasTicket` is `true`, so the outer block is entered.
- Inside, `age >= 18` is `false` (`16`), so the inner `else` runs.
- The outer `else` (`"You need a ticket."`) is skipped because the outer condition was true.

---

#### Example 2: Nested `if` with Early Exit (Refactored)

```csharp
using System;

class Program
{
    static void ProcessOrder(bool isLoggedIn, bool hasPermission, bool isStockAvailable)
    {
        if (!isLoggedIn)
        {
            Console.WriteLine("Please log in.");
            return;
        }

        if (!hasPermission)
        {
            Console.WriteLine("You do not have permission.");
            return;
        }

        if (!isStockAvailable)
        {
            Console.WriteLine("Item is out of stock.");
            return;
        }

        Console.WriteLine("Order processed successfully.");
    }

    static void Main()
    {
        ProcessOrder(true, true, false);   // Item is out of stock.
    }
}
```

**Expected Output**

```
Item is out of stock.
```

**Why this output occurs**

- The guard clauses check each condition in sequence and exit early when a check fails. This flattens the nesting and makes the code easier to follow.

---

### Real-World Cases

- **Access control** — Check authentication, then authorisation, then resource availability.
- **Form validation** — Check required fields, then format, then business rules.
- **Game logic** — Check game state, then player state, then move validity.

---

### References

- Control code flow with if, else if, and else — Microsoft Learn training (nesting) - https://learn.microsoft.com/en-us/training/modules/csharp-if-elseif-else/3-control-flow-with-if-else
- if statement — C# language specification (nested if) - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/language-specification/statements#1385-the-if-statement

---

## 4. Ternary Operator (`?:`)

### Definitions

**Core Definition**
The ternary conditional operator (`?:`) evaluates a Boolean expression and returns **one of two values** depending on whether the expression is `true` or `false`. It is an **expression**, not a statement, so it produces a value.

**Technical Definition**
The conditional operator `?:`, also known as the ternary conditional operator, evaluates a Boolean expression and returns the result of one of two expressions. Its syntax is `condition ? consequent : alternative`. The `condition` must evaluate to `true` or `false`. If `condition` is `true`, `consequent` is evaluated and its result becomes the result; if `false`, `alternative` is evaluated and its result becomes the result. **Only one of the two branches is evaluated.** Conditional expressions are **target-typed**: if a target type is known, the types of `consequent` and `alternative` must be implicitly convertible to the target type. The operator is **right-associative**: `a ? b : c ? d : e` is evaluated as `a ? b : (c ? d : e)`.

**Beginner-Friendly Explanation**
The ternary operator is a **one-line if/else that returns a value**. It reads: “Is this condition true? If yes, use this value; if no, use that value.” It is perfect for short, simple choices like `string status = score >= 60 ? "Pass" : "Fail";`.

---

### Purposes

- To choose between two values based on a condition in a single expression.
- To simplify short `if`/`else` assignments.
- To enable conditional values inside larger expressions.
- To support target-typed expressions with `int?`, `IEnumerable<T>`, and other types.
- To make simple branching more concise.

---

### Syntax Rules and Structure

```
condition   ?   consequent   :   alternative
```

**Component breakdown**

- `condition` — A `bool` expression.
- `?` — Introduces the consequent.
- `consequent` — The value returned when `condition` is `true`.
- `:` — Separates the branches.
- `alternative` — The value returned when `condition` is `false`.

#### Syntax Rules

- `condition` must be a `bool` expression.
- Only one branch is evaluated.
- The operator is right-associative.
- Target-typed conditional expressions require both branches to be implicitly convertible to the target type.
- The conditional operator cannot be overloaded.

#### Constraints and Limitations

- Both branches must be compatible; otherwise, a cast may be required.
- Nested ternary operators are **hard to read** and should be avoided.
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
        Console.WriteLine(GetWeatherDisplay(15));   // Cold.
        Console.WriteLine(GetWeatherDisplay(27));   // Perfect!
    }
}
```

**Expected Output**

```
Cold.
Perfect!
```

**Why this output occurs**

- When the temperature is below `20`, the `consequent` (`"Cold."`) is returned; otherwise, the `alternative` (`"Perfect!"`) is returned.

---

#### Example 2: Target-Typed Ternary

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

- The target type `int?` allows both `12` and `null`.
- The target type `IEnumerable<int>` allows both `List<int>` and `int[]`.

---

### Real-World Cases

- **Display formatting** — `quantity > 0 ? $"{quantity} items" : "Out of stock"`.
- **API responses** — `user is null ? NotFound() : Ok(user)`.
- **Configuration defaults** — `useCache ? cache : noCache`.

---

### References

- ?: operator — the ternary conditional operator — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/conditional-operator
- Conditional operator — C# language specification - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/language-specification/expressions#1215-conditional-operator

---

## 5. `switch` Statement

### Definitions

**Core Definition**
The `switch` statement selects **one statement list** to execute from many possible sections based on a **pattern match** with an expression. It is the traditional multi-way branching construct in C#.

**Technical Definition**
The `switch` statement selects a statement list to execute based on a pattern match with an expression. The governing type of a `switch` statement can be `string`, a nullable value type, an integral type, an enum type, or any type that supports pattern matching. Each `case` label specifies a pattern; if the pattern matches the selector expression, control transfers to that section. The `default` label specifies the section to execute if no other `case` matches. **C# enforces a “no fall-through” rule**: each switch section must end with a `break`, `goto case`, `goto default`, `return`, `throw`, or a construct that makes the end point unreachable. This prevents accidental fall-through from C/C++.

**Beginner-Friendly Explanation**
The `switch` statement is an **organised multi-way decision**. You give it a value, and it checks each `case` to see if it matches. When it finds a match, it runs that section and stops (thanks to `break`). If nothing matches, the `default` section runs. It is cleaner than a long `else if` chain when you are comparing **one value** against **many specific options**.

---

### Purposes

- To select one of many code paths based on a single value or pattern.
- To replace long `else if` chains with a more readable structure.
- To support pattern matching with types, constants, and `when` guards (C# 7.0+).
- To provide a clear `default` fallback.
- To enable compiler optimisations such as jump tables for integral and string values.

---

### Syntax Rules and Structure

```
switch (expression)
{
    case pattern1:
        // statements
        break;
    case pattern2 when condition:
        // statements
        break;
    default:
        // statements
        break;
}
```

**Component breakdown**

- `expression` — The value to match against.
- `case pattern` — A pattern (constant, type, property, relational, etc.).
- `when condition` — Optional case guard.
- `default` — Optional fallback section.
- `break` — Required to exit the switch section.

#### Syntax Rules

- Each section must end with `break`, `goto case`, `goto default`, `return`, `throw`, or an unreachable end point.
- The `default` label is optional but recommended.
- Case labels can include patterns (C# 7.0+).
- `when` clauses add additional conditions (C# 7.0+).
- String matching is case-sensitive and ordinal by default.

#### Constraints and Limitations

- Fall-through between sections is **not allowed** (unlike C/C++).
- The switch expression must be a value, not a complex expression in every C# version.
- Deeply complex patterns can reduce readability.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Basic `switch` with `default`

```csharp
using System;

class Program
{
    static void Main()
    {
        int priority = 2;

        switch (priority)
        {
            case 1:
                Console.WriteLine("High priority");
                break;
            case 2:
                Console.WriteLine("Medium priority");   // this branch
                break;
            case 3:
                Console.WriteLine("Low priority");
                break;
            default:
                Console.WriteLine("Unknown priority");
                break;
        }
    }
}
```

**Expected Output**

```
Medium priority
```

**Why this output occurs**

- `priority` is `2`, which matches `case 2`, so that section executes and `break` exits the switch.

---

#### Example 2: `switch` with Type Patterns and `when`

```csharp
using System;

class Shape { }
class Circle : Shape { public double Radius { get; set; } }
class Square : Shape { public double Side { get; set; } }

class Program
{
    static string Describe(Shape shape)
    {
        switch (shape)
        {
            case Circle c when c.Radius > 10:
                return "Large circle";
            case Circle c:
                return "Circle";
            case Square s when s.Side > 10:
                return "Large square";
            case Square s:
                return "Square";
            case null:
                return "Nothing";
            default:
                return "Unknown shape";
        }
    }

    static void Main()
    {
        Console.WriteLine(Describe(new Circle { Radius = 5 }));   // Circle
        Console.WriteLine(Describe(new Circle { Radius = 20 }));  // Large circle
    }
}
```

**Expected Output**

```
Circle
Large circle
```

**Why this output occurs**

- The first `Circle` has `Radius = 5`, so the `when` guard (`> 10`) fails, and the next `case Circle c` matches.
- The second `Circle` has `Radius = 20`, so the guarded case matches first.

---

### Real-World Cases

- **Command parsing** — Switching on a command string.
- **State machines** — Handling different states represented by enums.
- **Type dispatch** — Handling different subclasses of a base type.
- **Configuration** — Applying different settings based on a mode enum.

---

### References

- if and switch statements — select a code path to execute — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/statements/selection-statements
- switch statement — C# language specification - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/language-specification/statements#1388-the-switch-statement
- Exercise — Implement a switch statement — Microsoft Learn training - https://learn.microsoft.com/en-us/training/modules/csharp-evaluate-conditions/6-exercise-implement-switch-statement

---

## 6. `switch` Expression (C# 8)

### Definitions

**Core Definition**
A `switch` expression (introduced in **C# 8.0**) evaluates a **single expression** from a list of candidate expressions based on a pattern match. Unlike the `switch` statement, it **produces a value** and is used in expression contexts.

**Technical Definition**
Beginning with C# 8.0, the `switch` expression evaluates a single expression from a list of candidate expressions based on a pattern match with an input expression. Its syntax is `input switch { pattern1 => expression1, pattern2 => expression2, ... }`. Each arm consists of a **pattern**, an optional **`when` case guard**, the `=>` token, and an **expression**. The value of the switch expression is the value of the expression of the **first arm whose pattern matches** and whose guard, if present, evaluates to `true`. The arms are evaluated **in text order**. If no arm matches, a `SwitchExpressionException` (or `InvalidOperationException` on .NET Framework) is thrown at runtime. A discard pattern (`_`) is commonly used as the final arm to make the expression **exhaustive**.

**Beginner-Friendly Explanation**
The `switch` expression is a **compact, value-returning switch**. Instead of writing `case` and `break`, you write `pattern => result`. It is like a series of `if` statements that produces a single value. It fits naturally into assignments, return statements, and expression-bodied members.

---

### Purposes

- To select a value based on pattern matching in an expression context.
- To replace verbose `switch` statements that only assign or return a result.
- To support constant, relational, type, property, and discard patterns.
- To encourage exhaustive handling with compiler warnings for unhandled cases.
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
- `pattern` — A pattern (constant, relational, type, property, discard).
- `when` — Optional case guard.
- `=>` — Separates the pattern from the result expression.
- `expression` — The value returned when the pattern matches.

#### Syntax Rules

- Arms are separated by commas.
- Arms are evaluated in text order; the **first match wins**.
- The compiler warns about unhandled values (unlike the statement form, which allows a `default`).
- A discard arm (`_`) should be included to make the expression exhaustive.
- Case guards (`when`) add additional Boolean conditions.

#### Constraints and Limitations

- If no arm matches, a runtime exception is thrown.
- The compiler may not always prove exhaustiveness; a discard arm is recommended.
- Requires **C# 8.0** or later.

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
        Console.WriteLine(ToCardinal(Direction.Left));   // West
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

#### Example 2: Relational and `when` Patterns

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

- Relational patterns (`< 0`, `> 0 and < 10`) match the appropriate ranges.
- The discard pattern (`_`) handles all remaining values.

---

### Real-World Cases

- **Mapping enums to display names** — `status switch { Status.Active => "Active", ... }`.
- **Type-based dispatch** — `shape switch { Circle c => ..., Square s => ... }`.
- **Range classification** — `score switch { >= 90 => "A", >= 80 => "B", ... }`.
- **Expression-bodied members** — `public string Description => state switch { ... };`.

---

### References

- switch expression — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/switch-expression
- Pattern matching overview — C# - https://learn.microsoft.com/en-us/dotnet/csharp/fundamentals/patterns/pattern-matching
- Switch expressions — C# 8.0 new features - https://learn.microsoft.com/en-us/dotnet/csharp/whats-new/csharp-8#switch-expressions

---

## 7. Pattern Matching

### Definitions

**Core Definition**
Pattern matching is a C# feature that tests an expression against a **pattern**—a condition that checks the value’s type, shape, properties, or relationship to constants—and, when it matches, can **extract** information from the value.

**Technical Definition**
C# uses patterns in `is` expressions, `switch` statements, and `switch` expressions. A pattern is a condition to test an input expression against. The pattern tests whether the value has a particular **type**, **equals** a particular value, or has a particular **shape** (properties, positional elements, etc.). When a pattern matches, the code can use information learned by the test, such as a more specific type or a value extracted from an object. C# supports **constant patterns** (literals, `null`, enum values), **type patterns** (with optional variable designation), **property patterns**, **positional patterns**, **relational patterns** (`<`, `>`, `<=`, `>=`), **logical patterns** (`and`, `or`, `not`), **tuple patterns**, **list patterns** (C# 11), and the **discard pattern** (`_`). Patterns are often clearer than a sequence of comparison statements because each branch describes the data it handles.

**Beginner-Friendly Explanation**
Pattern matching lets you ask **“what does this data look like?”** and **“what can I learn from it?”** Instead of checking `if (obj is Circle)` and then casting, you can write `if (obj is Circle c)` and immediately use `c`. In a `switch`, you can match on types, property values, ranges, and combinations of these, all in a readable way.

---

### Purposes

- To test the runtime type of an object and safely extract a typed reference.
- To match against constants, ranges, and property values.
- To deconstruct tuples and positional records.
- To express complex conditions declaratively.
- To enable exhaustive dispatch in `switch` expressions.
- To replace verbose `is`/cast sequences with concise patterns.

---

### Syntax Rules and Structure (Pattern Types)

| Pattern | Example | Purpose |
|---|---|---|
| **Constant** | `42`, `"hello"`, `null`, `MyEnum.Value` | Matches a specific constant value. |
| **Type** | `Circle c` | Matches a type and optionally captures the value. |
| **Property** | `{ Radius: > 10 }` | Matches an object whose property satisfies a pattern. |
| **Positional** | `(x, y)` | Matches a tuple or deconstructable object. |
| **Relational** | `> 0`, `<= 100` | Matches a value against a relational bound. |
| **Logical** | `> 0 and < 10`, `not null` | Combines patterns with `and`, `or`, `not`. |
| **Discard** | `_` | Matches anything, including `null`. |
| **List** | `[1, 2, ..]` | Matches a sequence shape (C# 11). |

**Component breakdown**

- **Constant pattern** — Tests equality with a compile-time constant.
- **Type pattern** — Tests runtime type compatibility and optionally declares a variable.
- **Property pattern** — Tests whether a property or field matches a nested pattern.
- **Relational pattern** — Tests `<`, `>`, `<=`, or `>=` against a constant.
- **Logical pattern** — Combines patterns with `and`, `or`, `not`.
- **Discard pattern** — `_` matches anything.

#### Syntax Rules

- Patterns can be used in `is` expressions, `case` labels, and `switch` expression arms.
- Pattern variables are in scope only where the pattern matches.
- The `when` clause adds an additional Boolean condition to a pattern.
- Patterns are evaluated in order; the first match wins in `switch`.

#### Constraints and Limitations

- Constant patterns must be compile-time constants.
- Property patterns require the object reference to be non-null to access properties.
- Deeply nested patterns can be hard to read.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: `is` Pattern with Type Capture

```csharp
using System;

class Animal { }
class Mammal : Animal { public string Name { get; set; } }

class Program
{
    static void Main()
    {
        Animal a = new Mammal { Name = "Otter" };

        if (a is Mammal m)
        {
            Console.WriteLine(m.Name);   // Otter
        }
    }
}
```

**Expected Output**

```
Otter
```

**Why this output occurs**

- The `is Mammal m` pattern tests the runtime type and captures the value in `m`.

---

#### Example 2: Property and Relational Patterns

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
        { Width: 0, Height: 0 } => "Point",
        { Width: var w, Height: var h } when w == h => "Square",
        { Width: > 100 } => "Wide rectangle",
        _ => "Rectangle"
    };

    static void Main()
    {
        Console.WriteLine(Describe(new Rectangle { Width = 0, Height = 0 }));   // Point
        Console.WriteLine(Describe(new Rectangle { Width = 5, Height = 5 }));   // Square
        Console.WriteLine(Describe(new Rectangle { Width = 150, Height = 20 })); // Wide rectangle
    }
}
```

**Expected Output**

```
Point
Square
Wide rectangle
```

**Why this output occurs**

- The property pattern `{ Width: 0, Height: 0 }` matches the point.
- The `when` guard checks whether `Width == Height` for the square.
- The relational property pattern `{ Width: > 100 }` matches the wide rectangle.

---

### Real-World Cases

- **Visitor pattern** — Type-based dispatch with extracted data.
- **Serialization** — Handling different JSON token types.
- **Domain modelling** — Classifying shapes, states, or commands.
- **API input validation** — Extracting and validating typed values.

---

### References

- Pattern matching overview — C# - https://learn.microsoft.com/en-us/dotnet/csharp/fundamentals/patterns/pattern-matching
- Patterns — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/patterns
- Pattern matching — C# language specification - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/language-specification/patterns

---

## 8. `when` Clause

### Definitions

**Core Definition**
The `when` clause adds an **additional Boolean condition** to a `case` label or `switch` expression arm. The pattern must match **and** the `when` condition must be `true` for that branch to be selected.

**Technical Definition**
A `when` clause (also called a **case guard**) can be added to any pattern case in a `switch` statement or to any arm in a `switch` expression. The case guard expression must evaluate to `bool`. The branch is selected only if the pattern matches **and** the guard evaluates to `true`. If the guard is `false`, evaluation continues to the next arm. `when` clauses are commonly used to refine type patterns (e.g., `case Circle c when c.Radius > 10:`) or to add conditions that cannot be expressed as patterns alone.

**Beginner-Friendly Explanation**
The `when` clause is an **extra condition attached to a pattern**. It says: “Match this pattern, but only if this additional condition is also true.” It is useful when the pattern alone is not specific enough.

---

### Purposes

- To refine a pattern match with additional Boolean conditions.
- To order arms precisely by combining type/property patterns with guards.
- To avoid overly complex patterns.
- To express conditions that depend on computed values or method calls.
- To handle edge cases in `switch` expressions.

---

### Syntax Rules and Structure

```
case pattern when condition:
// or
pattern when condition => expression
```

**Component breakdown**

- `pattern` — The pattern to match.
- `when` — The keyword introducing the guard.
- `condition` — A `bool` expression; the branch is selected only if this is `true`.

#### Syntax Rules

- The `when` clause can be used in `switch` statements (C# 7.0+) and `switch` expressions (C# 8.0+).
- The guard expression must evaluate to `bool`.
- If the pattern matches but the guard is `false`, the next arm is evaluated.
- Guards can reference variables captured by the pattern.

#### Constraints and Limitations

- `when` cannot be used in `if` statements.
- Guards add runtime evaluation cost.
- Overly complex guards can reduce readability.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: `when` in a `switch` Statement

```csharp
using System;

class Program
{
    static void Main()
    {
        object value = "Hello, World!";

        switch (value)
        {
            case string s when s.StartsWith("Hello"):
                Console.WriteLine($"Greeting: {s}");
                break;
            case string s:
                Console.WriteLine($"String: {s}");
                break;
            default:
                Console.WriteLine("Other");
                break;
        }
    }
}
```

**Expected Output**

```
Greeting: Hello, World!
```

**Why this output occurs**

- The `string s` pattern matches, and the `when` guard `s.StartsWith("Hello")` is `true`, so the first case is selected.

---

#### Example 2: `when` in a `switch` Expression

```csharp
using System;

class Program
{
    static string Categorize(int temperature) => temperature switch
    {
        < 0 => "Freezing",
        >= 0 and < 15 when IsWindy() => "Cold and windy",
        >= 0 and < 15 => "Cold",
        >= 15 and < 25 => "Mild",
        _ => "Hot"
    };

    static bool IsWindy() => true;

    static void Main()
    {
        Console.WriteLine(Categorize(10));   // Cold and windy
        Console.WriteLine(Categorize(20));   // Mild
    }
}
```

**Expected Output**

```
Cold and windy
Mild
```

**Why this output occurs**

- `10` matches the pattern `>= 0 and < 15`, and `IsWindy()` is `true`, so the guarded arm is selected.
- `20` matches `>= 15 and < 25`.

---

### Real-World Cases

- **Type dispatch with conditions** — `case Circle c when c.Radius > 0:`.
- **String processing** — Matching strings with specific prefixes or lengths.
- **Business rules** — Applying rules only when additional conditions hold.
- **State machines** — Guarding transitions based on context.

---

### References

- Pattern matching in the switch statement — `when` clause - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/patterns#case-guards
- switch expression — C# reference (case guards) - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/switch-expression
- Pattern matching overview — C# (when guards) - https://learn.microsoft.com/en-us/dotnet/csharp/fundamentals/patterns/pattern-matching

---

## 9. Conditional Best Practices

### Definitions

**Core Definition**
Conditional best practices are the guidelines that ensure conditional code is readable, maintainable, correct, and performant. They cover formatting, ordering, choosing the right construct, and avoiding common pitfalls.

**Technical Definition**
Best practices for conditional statements derive from Microsoft’s C# coding conventions, .NET Framework Design Guidelines, and widely accepted community standards. Key recommendations include: **always use curly braces** for `if`, `else`, and `switch` sections; **order conditions from most specific to least specific** in `else if` chains; **avoid nested ternary operators**; **prefer `switch` for multi-way constant comparisons**; **use pattern matching** for type and property checks; **keep conditions simple** by extracting complex expressions into named Boolean variables or methods; and **avoid assignments inside conditions**. The choice between `if`/`else` and `switch` often depends on readability and the nature of the comparison, not raw performance, as modern C# compilers optimise both.

**Beginner-Friendly Explanation**
Best practices are the habits that keep conditional code from becoming messy: use braces even for one-liners, put the most important check first, do not nest ternaries, and use `switch` when you are comparing one value against many options. Keep conditions short and give them meaningful names.

---

### Purposes

- To improve readability and maintainability.
- To prevent logic errors caused by missing braces or misordered conditions.
- To choose the most appropriate conditional construct for the task.
- To make complex conditions easier to understand.
- To avoid performance pitfalls and compiler warnings.

---

### Guidelines and Structure

#### General Guidelines

```
// Always use curly braces
if (condition)
{
    DoSomething();
}

// Order conditions from most specific to least specific
if (score >= 90) { ... }
else if (score >= 80) { ... }
else if (score >= 70) { ... }
else { ... }

// Avoid nested ternaries
// Bad: var x = a ? b : c ? d : e;
// Good: use if/else or a switch expression

// Prefer switch for multi-way constant comparisons
switch (status)
{
    case Status.Active: ...
    case Status.Inactive: ...
    default: ...
}

// Use pattern matching for type checks
if (obj is Circle c) { ... }

// Extract complex conditions into named variables or methods
bool isValid = age >= 18 && hasLicense && !isSuspended;
if (isValid) { ... }
```

**Component breakdown**

- **Curly braces** — Always use them, even for single statements, to prevent misleading indentation bugs.
- **Specific-to-general ordering** — Ensures the intended branch is selected.
- **No nested ternaries** — They are hard to read; use `if`/`else` or `switch` expressions.
- **`switch` for multi-way** — Cleaner than long `else if` chains for a single value.
- **Pattern matching** — More concise and safer than `is` followed by a cast.
- **Named conditions** — Extract complex Boolean expressions into well-named variables or methods.

#### Rules and Constraints

- **Do** use curly braces for all `if`, `else`, and `switch` sections.
- **Do** order `else if` conditions from most specific to least specific.
- **Do** use `switch` when comparing one value against multiple constants.
- **Do** use `switch` expressions when the result is a value.
- **Do** use `when` guards to refine patterns.
- **Do not** use nested ternary operators.
- **Do not** evaluate Boolean conditions against `true` or `false` (e.g., `if (isValid == true)`).
- **Do not** assign inside conditions (e.g., `if ((x = GetValue()) > 0)`).
- **Do not** use compound conditions when a named variable would be clearer.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Braces and Specific-to-General Ordering

```csharp
using System;

class Program
{
    static void Main()
    {
        int score = 85;

        if (score >= 90)
        {
            Console.WriteLine("A");
        }
        else if (score >= 80)
        {
            Console.WriteLine("B");   // this branch
        }
        else if (score >= 70)
        {
            Console.WriteLine("C");
        }
        else
        {
            Console.WriteLine("F");
        }
    }
}
```

**Expected Output**

```
B
```

**Why this output occurs**

- The conditions are ordered from highest to lowest. `85` matches `>= 80`, and the later conditions are skipped.

---

#### Example 2: Extracting a Complex Condition

```csharp
using System;

class Program
{
    static bool CanPlaceOrder(bool isLoggedIn, bool hasPermission, bool isStockAvailable)
    {
        bool isAuthorized = isLoggedIn && hasPermission;
        bool canProceed = isAuthorized && isStockAvailable;

        if (canProceed)
        {
            Console.WriteLine("Order placed");
            return true;
        }

        Console.WriteLine("Order cannot be placed");
        return false;
    }

    static void Main()
    {
        CanPlaceOrder(true, true, false);   // Order cannot be placed
    }
}
```

**Expected Output**

```
Order cannot be placed
```

**Why this output occurs**

- The complex condition is split into named Boolean variables, making it easier to read and debug.

---

### Real-World Cases

- **Code reviews** — Curly braces and condition ordering are common review criteria.
- **Large codebases** — Named conditions and `switch` expressions improve maintainability.
- **Library APIs** — Pattern matching and `switch` expressions make dispatch logic clear.

---

### References

- C# coding conventions — Microsoft Learn - https://learn.microsoft.com/en-us/dotnet/csharp/fundamentals/coding-style/coding-conventions
- if and switch statements — select a code path to execute — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/statements/selection-statements
- C# if-else statement: Curly braces or not? An in-depth analysis — Microsoft Learn - https://learn.microsoft.com/en-us/archive/blogs/csharpfaq/c-if-else-statement-curly-braces-or-not-an-in-depth-analysis
- Ternary operators should not be nested — SonarSource rule - https://rules.sonarsource.com/csharp/RSPEC-3358
- Which is Faster: if-else or switch-case? — Microsoft Q&A - https://learn.microsoft.com/en-us/answers/questions/2094353/which-is-faster-if-else-or-switch-case

---

## Deprecated, Unsafe, Implementation-Dependent, and Version-Specific Notes

| Feature | Status / Note |
|---------|---------------|
| `switch` expression | Introduced in **C# 8.0**; version-specific. |
| Pattern matching in `switch` | Introduced in **C# 7.0**; expanded in C# 8, 9, 10, and 11. |
| `when` clause | Introduced in **C# 7.0** for `switch` statements; C# 8.0 for `switch` expressions. |
| Property patterns | Introduced in **C# 8.0**. |
| Relational and logical patterns | Introduced in **C# 9.0**. |
| List patterns | Introduced in **C# 11**. |
| Nested ternary operators | **Discouraged** by style guidelines; hard to read. |
| Omitting curly braces | **Strongly discouraged**; can cause misleading indentation bugs. |
| `switch` vs. `if`/`else` performance | **Implementation-dependent**; modern compilers optimise both. In general, no significant difference for typical cases. |

---

## References (Consolidated)

- if and switch statements — select a code path to execute — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/statements/selection-statements
- Control code flow with if, else if, and else — Microsoft Learn training - https://learn.microsoft.com/en-us/training/modules/csharp-if-elseif-else/3-control-flow-with-if-else
- ?: operator — the ternary conditional operator — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/conditional-operator
- switch expression — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/switch-expression
- Pattern matching overview — C# - https://learn.microsoft.com/en-us/dotnet/csharp/fundamentals/patterns/pattern-matching
- Patterns — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/patterns
- switch statement — C# language specification - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/language-specification/statements#1388-the-switch-statement
- if statement — C# language specification - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/language-specification/statements#1385-the-if-statement
- Conditional operator — C# language specification - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/language-specification/expressions#1215-conditional-operator
- C# coding conventions — Microsoft Learn - https://learn.microsoft.com/en-us/dotnet/csharp/fundamentals/coding-style/coding-conventions
- C# if-else statement: Curly braces or not? An in-depth analysis — Microsoft Learn - https://learn.microsoft.com/en-us/archive/blogs/csharpfaq/c-if-else-statement-curly-braces-or-not-an-in-depth-analysis
- Ternary operators should not be nested — SonarSource rule - https://rules.sonarsource.com/csharp/RSPEC-3358
- Which is Faster: if-else or switch-case? — Microsoft Q&A - https://learn.microsoft.com/en-us/answers/questions/2094353/which-is-faster-if-else-or-switch-case
- C# Pattern Matching Explained (2026) — NDepend Blog - https://blog.ndepend.com/c-pattern-matching-explained/

---