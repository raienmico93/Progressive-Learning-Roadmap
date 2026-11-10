# C# Switch Statements — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
A switch statement in C# is a selection statement that chooses **one execution path** from many possible paths based on a **pattern match** with a governing expression. It is the language’s primary multi-way branching construct, cleaner and more structured than long chains of `if`/`else if` when comparing a single value against multiple options. C# provides two forms: the traditional **switch statement** (which executes statements) and the modern **switch expression** (C# 8.0+, which produces a value).

**Technical Definition**
The `switch` statement selects a *statement list* to execute based on a pattern match with a *selector expression*. The governing type of a `switch` statement can be `string`, a nullable value type, an integral type, an enum type, or any type that supports pattern matching. Each `case` label specifies a pattern; if the pattern matches the selector expression, control transfers to that section. The `default` label specifies the section to execute if no other case matches. C# enforces a strict **“no fall-through” rule**: each switch section must end with a construct that makes its endpoint unreachable—typically `break`, `goto case`, `goto default`, `return`, or `throw`. Since **C# 7.0**, `switch` statements support **pattern matching** (type patterns, constant patterns, `when` case guards). Since **C# 8.0**, the `switch` expression evaluates a single expression from a list of candidate expressions based on pattern matching, using the syntax `input switch { pattern => expression, ... }`. The `switch` expression is **target-typed**, **first-match-wins**, and produces a value, making it suitable for expression contexts such as assignments and expression-bodied members.

**Beginner-Friendly Explanation**
A switch statement is an **organised multi-way decision**. Imagine you are a receptionist with a list of instructions for different visitor types. You look at the visitor’s badge (the governing expression), check it against your list of cases, and follow the instructions for the first matching case. If nothing matches, you follow the `default` instructions. The traditional switch statement is like a list of “if this, then do that” rules that run code. The switch expression is a newer, more compact version that **returns a value**—like a vending machine: you put in a code (the input), and it gives you back the corresponding item (the result). Pattern matching adds the ability to check **what type** something is, **what properties it has**, or **what range** a value falls into, making switch statements far more powerful than simple constant comparisons.

---

### Key Characteristics

- **Two forms** — The `switch` statement (executes statements) and the `switch` expression (C# 8.0+, produces a value).
- **No fall-through** — C# forbids implicit fall-through between sections; each section must end with `break`, `goto case`, `goto default`, `return`, or `throw`.
- **Pattern-powered** — Since C# 7.0, `case` labels can use type patterns, constant patterns, relational patterns, logical patterns, property patterns, positional patterns, and (in C# 11) list patterns.
- **`when` guards** — Any pattern can be refined with a `when` Boolean condition.
- **`goto case` / `goto default`** — Controlled, explicit transfer between sections (the only legal way to “fall through”).
- **Exhaustiveness** — `switch` expressions encourage exhaustive handling with a discard arm (`_`); `switch` statements use `default`.
- **String matching is ordinal and case-sensitive** — `switch` on strings matches exactly, like the `==` operator.

---

### Prerequisites

- Understanding of Boolean expressions and comparison operators.
- Familiarity with C# types (`int`, `string`, `bool`, enums, classes, structs).
- Knowledge of variables, scope, and statements.
- Awareness of pattern matching (type checks, property access, deconstruction).
- Basic understanding of expressions vs. statements.

---

### Related Programming Areas

- **Pattern matching** — Switch statements and expressions are the primary consumers of C#’s pattern vocabulary.
- **Control flow** — Switch is a core selection statement alongside `if`/`else`.
- **Functional programming** — Switch expressions bring expression-oriented dispatch to C#.
- **State machines** — Enums + switch are a natural fit for state transitions.
- **Type dispatch** — Switch on runtime types replaces complex `is`/cast chains.

---

### Core Concepts / Features

1. `switch` Statement
2. `case` Labels
3. `break` Statement
4. `default` Label
5. `goto case`
6. `goto default`
7. Switch Expression (C# 8)
8. Pattern Matching in Switch
9. Type Patterns
10. Constant Patterns
11. Relational Patterns
12. Logical Patterns
13. Property Patterns
14. Positional Patterns
15. List Patterns (C# 11)
16. Switch Best Practices

Each concept follows a uniform structure: definitions, purposes, syntax rules and structure, annotated code examples with expected outputs, real-world cases, and references.

---

## 1. `switch` Statement

### Definitions

**Core Definition**
The `switch` statement is a selection statement that executes one statement list from many based on a pattern match with a governing expression.

**Technical Definition**
The `switch` statement selects a statement list to execute based on a pattern match with the selector expression. The governing type can be `string`, a nullable value type, an integral type, an enum type, or any type supporting pattern matching. Each `case` label specifies a pattern; the first matching section executes. The `default` label executes if no case matches. The **no fall-through rule** requires each section to end with `break`, `goto case`, `goto default`, `return`, `throw`, or an unreachable endpoint. String matching is case-sensitive and ordinal.

**Beginner-Friendly Explanation**
The switch statement is a **structured decision-maker**. You give it a value, and it checks each `case` in order. When it finds a match, it runs that section and stops (thanks to `break`). If nothing matches, the `default` section runs.

---

### Purposes

- To select one of many code paths based on a single value or pattern.
- To replace long `else if` chains with a clearer structure.
- To support pattern matching with types, constants, properties, and guards.
- To provide a clear `default` fallback.
- To enable compiler optimisations (jump tables for integral and string values).

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

- `expression` — The governing expression (selector).
- `case pattern` — A pattern label.
- `when condition` — Optional case guard (C# 7.0+).
- `default` — Optional fallback label.
- `break` — Exits the switch section.

#### Syntax Rules

- Each section must end with `break`, `goto case`, `goto default`, `return`, `throw`, or an unreachable endpoint.
- The `default` label is optional but recommended.
- Case labels can include patterns (C# 7.0+).
- String matching is case-sensitive and ordinal.
- `null` is permitted as a case label for `string` and nullable value types.

#### Constraints and Limitations

- Implicit fall-through between sections is **not allowed**.
- The governing expression must be a supported type.

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

- `priority` is `2`, matching `case 2`, so that section executes and `break` exits the switch.

---

#### Example 2: String Switch

```csharp
using System;

class Program
{
    static void DoCommand(string command)
    {
        switch (command.ToLower())
        {
            case "run":
                Console.WriteLine("Running");
                break;
            case "save":
                Console.WriteLine("Saving");
                break;
            case "quit":
                Console.WriteLine("Quitting");
                break;
            default:
                Console.WriteLine($"Invalid command: {command}");
                break;
        }
    }

    static void Main()
    {
        DoCommand("SAVE");   // Saving
    }
}
```

**Expected Output**

```
Saving
```

**Why this output occurs**

- `command.ToLower()` produces `"save"`, matching `case "save"`. String matching is ordinal and case-sensitive after the conversion.

---

### Real-World Cases

- **Command parsing** — Dispatching on command strings.
- **State machines** — Handling enum states.
- **Menu systems** — Selecting actions by number.
- **Type dispatch** — Handling different subclasses of a base type.

---

### References

- if and switch statements — select a code path to execute — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/statements/selection-statements
- switch statement — C# language specification - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/language-specification/statements#1388-the-switch-statement

---

## 2. `case` Labels

### Definitions

**Core Definition**
A `case` label associates a **pattern** with a switch section. When the governing expression matches the pattern, the section executes.

**Technical Definition**
A `case` label consists of the `case` keyword followed by a pattern. Since C# 7.0, the pattern can be a constant pattern, type pattern, or other supported pattern. Each `case` label must be unique and must not be subsumed by an earlier label. A `case` label may include a `when` case guard to add an additional Boolean condition.

**Beginner-Friendly Explanation**
A `case` label is a **specific test** the switch performs. It says: “If the input matches this pattern, run this section.” You can have many `case` labels, each checking for a different value, type, or shape.

---

### Purposes

- To specify the conditions under which a switch section executes.
- To support constant, type, relational, logical, property, positional, and list patterns.
- To refine matches with `when` guards.
- To provide readable, declarative branching.

---

### Syntax Rules and Structure

```
case   pattern   [when   condition]   :
```

**Component breakdown**

- `case` — The keyword.
- `pattern` — The pattern to match (constant, type, relational, etc.).
- `when condition` — Optional case guard.
- `:` — Terminates the label.

#### Syntax Rules

- Patterns must be compatible with the governing type.
- Labels are evaluated in order; the first match wins.
- A `case` label cannot be subsumed by an earlier label (compile-time error).
- `null` is permitted as a constant pattern for `string` and nullable types.

#### Constraints and Limitations

- Constant patterns must be compile-time constants.
- Duplicate or subsumed labels cause compile errors.

---

### Annotated Code Examples and Expected Outputs

#### Example: Case Labels with Constants

```csharp
using System;

class Program
{
    static void Main()
    {
        int code = 404;

        switch (code)
        {
            case 200:
                Console.WriteLine("OK");
                break;
            case 404:
                Console.WriteLine("Not Found");   // this branch
                break;
            case 500:
                Console.WriteLine("Server Error");
                break;
        }
    }
}
```

**Expected Output**

```
Not Found
```

**Why this output occurs**

- `code` is `404`, matching `case 404`.

---

### Real-World Cases

- **HTTP status handling** — `case 200`, `case 404`, `case 500`.
- **Enum dispatch** — `case Status.Active`, `case Status.Inactive`.

---

### References

- switch statement — case labels - C# language specification - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/language-specification/statements#1388-the-switch-statement
- Patterns — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/patterns

---

## 3. `break` Statement

### Definitions

**Core Definition**
The `break` statement exits the nearest enclosing `switch` or loop statement immediately, transferring control to the statement following the switch or loop.

**Technical Definition**
In a `switch` statement, `break` terminates the switch section and transfers control to the end of the switch statement. C# requires that switch sections not “fall through” to the next section; `break` is the most common way to satisfy this requirement. Other valid endpoints include `goto case`, `goto default`, `return`, `throw`, `while (true)` (infinite loop), and other constructs that make the endpoint unreachable.

**Beginner-Friendly Explanation**
`break` means **“stop here and leave the switch.”** Without it (or another exit), the compiler will complain, because C# refuses to let one case accidentally run into the next.

---

### Purposes

- To exit a switch section cleanly.
- To prevent fall-through.
- To transfer control to the statement after the switch.
- To terminate loops as well as switches.

---

### Syntax Rules and Structure

```
break;
```

**Component breakdown**

- `break` — The keyword.
- `;` — Statement terminator.

#### Syntax Rules

- `break` can appear only inside a `switch` or loop.
- It exits the innermost enclosing `switch` or loop.
- It makes the endpoint of the switch section unreachable.

#### Constraints and Limitations

- Cannot be used outside a switch or loop.
- In nested loops/switches, `break` exits only the innermost one.

---

### Annotated Code Examples and Expected Outputs

#### Example: `break` Exits the Switch

```csharp
using System;

class Program
{
    static void Main()
    {
        int x = 1;

        switch (x)
        {
            case 1:
                Console.WriteLine("One");
                break;
            case 2:
                Console.WriteLine("Two");
                break;
        }

        Console.WriteLine("After switch");   // this line
    }
}
```

**Expected Output**

```
One
After switch
```

**Why this output occurs**

- `break` exits the switch, and execution continues with the statement after it.

---

### Real-World Cases

- **All switch sections** — `break` is the default endpoint.
- **Loop termination** — `break` inside loops.

---

### References

- The break statement — C# language specification - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/language-specification/statements#1389-the-break-statement
- Statements — C# reference (break) - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/statements/jump-statements

---

## 4. `default` Label

### Definitions

**Core Definition**
The `default` label specifies the switch section to execute when no `case` pattern matches the governing expression.

**Technical Definition**
The `default` label is optional. When present, it must appear at most once. It is reachable when none of the `case` labels match the selector expression. The `default` section executes if the selector value does not match any pattern, or if no reachable `case` label matches. If no `default` label is present and no `case` matches, control falls through to the end of the switch statement.

**Beginner-Friendly Explanation**
`default` is the **“catch-all”** section. It runs when none of the specific cases match. It is like the `else` in an `if`/`else` chain.

---

### Purposes

- To provide a fallback when no case matches.
- To handle unexpected or unhandled values.
- To make switch statements total (covering all possible inputs).
- To support defensive programming.

---

### Syntax Rules and Structure

```
default:
    // statements
    break;
```

**Component breakdown**

- `default` — The keyword.
- `:` — Terminates the label.

#### Syntax Rules

- At most one `default` label per switch.
- Can appear anywhere in the switch block (commonly last).
- Executes when no `case` matches.
- The `default` section must also end with a valid endpoint (typically `break`).

#### Constraints and Limitations

- Optional; if omitted and no case matches, control exits the switch.
- Should not be used to hide unexpected values; consider throwing an exception instead.

---

### Annotated Code Examples and Expected Outputs

#### Example: `default` Fallback

```csharp
using System;

class Program
{
    static void Main()
    {
        int status = 99;

        switch (status)
        {
            case 0:
                Console.WriteLine("Inactive");
                break;
            case 1:
                Console.WriteLine("Active");
                break;
            default:
                Console.WriteLine("Unknown status");   // this branch
                break;
        }
    }
}
```

**Expected Output**

```
Unknown status
```

**Why this output occurs**

- `status` is `99`, which matches neither `0` nor `1`, so the `default` section runs.

---

### Real-World Cases

- **Enum handling** — Handling unexpected enum values.
- **Protocol parsing** — Handling unknown message types.
- **Input validation** — Rejecting invalid inputs.

---

### References

- switch statement — default label - C# language specification - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/language-specification/statements#1388-the-switch-statement
- if and switch statements — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/statements/selection-statements

---

## 5. `goto case`

### Definitions

**Core Definition**
The `goto case` statement transfers control to a specified `case` label within the **same** switch statement.

**Technical Definition**
The target of a `goto case` statement is the statement list in the immediately enclosing `switch` statement that contains a `case` label with a **constant pattern** of the given constant value and **no guard**. If the `goto case` is not enclosed by a switch, if the nearest enclosing switch does not contain such a case, or if the constant expression is not implicitly convertible to the governing type, a compile-time error occurs. `goto case` is the only legal way to explicitly fall through to another section.

**Beginner-Friendly Explanation**
`goto case` means **“jump to that other case in this switch.”** It is the explicit way to say “do the same thing as that case.” C# allows this because it is deliberate—unlike accidental fall-through, which it forbids.

---

### Purposes

- To explicitly share code between cases (deliberate fall-through).
- To route multiple inputs to the same section.
- To implement state-machine transitions within a switch.
- To avoid duplicating code across similar cases.

---

### Syntax Rules and Structure

```
goto case   constantExpression;
```

**Component breakdown**

- `goto case` — The keyword pair.
- `constantExpression` — The constant value of the target case label.
- `;` — Statement terminator.

#### Syntax Rules

- The target case must have a **constant pattern** and **no guard**.
- The constant must be implicitly convertible to the governing type.
- Must be inside a switch statement.
- The target case must exist in the same switch.

#### Constraints and Limitations

- Cannot target a case with a non-constant pattern (e.g., type pattern).
- Cannot target a case with a `when` guard.
- Cannot jump into a nested scope.

---

### Annotated Code Examples and Expected Outputs

#### Example: Sharing Code with `goto case`

```csharp
using System;

class Program
{
    static void Main()
    {
        int i = 0;

        switch (i)
        {
            case 0:
                Console.WriteLine("Zero");
                goto case 1;          // explicit fall-through
            case 1:
                Console.WriteLine("One");
                break;
        }
    }
}
```

**Expected Output**

```
Zero
One
```

**Why this output occurs**

- `case 0` matches, prints `"Zero"`, then `goto case 1` transfers control to `case 1`, which prints `"One"`.

---

### Real-World Cases

- **Categorisation** — Grouping multiple inputs that share behaviour.
- **State machines** — Transitioning between states within one switch.
- **Protocol handlers** — Routing related message types to the same logic.

---

### References

- goto case — C# language specification - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/language-specification/statements#13105-the-goto-statement
- Statements — C# reference (goto) - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/statements/jump-statements

---

## 6. `goto default`

### Definitions

**Core Definition**
The `goto default` statement transfers control to the `default` label of the **immediately enclosing** switch statement.

**Technical Definition**
The target of a `goto default` statement is the statement list in the immediately enclosing `switch` statement that contains the `default` label. If the `goto default` is not enclosed by a switch, or if the nearest enclosing switch does not contain a `default` label, a compile-time error occurs. Like `goto case`, it is an explicit, deliberate transfer.

**Beginner-Friendly Explanation**
`goto default` means **“jump to the default case.”** It is useful when a case wants to defer to the fallback logic.

---

### Purposes

- To route a case to the default handling logic.
- To share the default behaviour among specific cases.
- To implement fallback chains explicitly.
- To avoid duplicating default code.

---

### Syntax Rules and Structure

```
goto default;
```

**Component breakdown**

- `goto default` — The keyword pair.
- `;` — Statement terminator.

#### Syntax Rules

- Must be inside a switch statement.
- The enclosing switch must have a `default` label.
- No constant expression is needed.

#### Constraints and Limitations

- Cannot be used if the enclosing switch lacks a `default` label.
- Cannot jump into a nested scope.

---

### Annotated Code Examples and Expected Outputs

#### Example: Deferring to `default`

```csharp
using System;

class Program
{
    static void Main()
    {
        string command = "help";

        switch (command)
        {
            case "run":
                Console.WriteLine("Running");
                break;
            case "help":
                Console.WriteLine("Showing help");
                goto default;          // then also show default
            default:
                Console.WriteLine("Unknown command");
                break;
        }
    }
}
```

**Expected Output**

```
Showing help
Unknown command
```

**Why this output occurs**

- `"help"` matches `case "help"`, prints `"Showing help"`, then `goto default` transfers to the `default` section, which prints `"Unknown command"`.

---

### Real-World Cases

- **Fallback logic** — When a specific case should also apply the default handling.
- **Logging** — Logging a specific message and then the default diagnostic.
- **State machines** — Deferring to a default transition.

---

### References

- goto default — C# language specification - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/language-specification/statements#13105-the-goto-statement
- Statements — C# reference (goto) - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/statements/jump-statements

---

## 7. Switch Expression (C# 8)

### Definitions

**Core Definition**
A switch expression evaluates a **single expression** from a list of candidate expressions based on a pattern match with an input expression. It **produces a value** and is used in expression contexts.

**Technical Definition**
Beginning with **C# 8.0**, the `switch` expression evaluates a single expression from a list of candidate expressions based on a pattern match with an input expression. Its syntax is `input switch { pattern1 => expression1, pattern2 => expression2, ... }`. Each arm contains a *pattern*, an optional *case guard* (`when`), the `=>` token, and an *expression*. The value of the switch expression is the value of the **first arm whose pattern matches** and whose guard evaluates to `true`. Arms are evaluated in text order. If no arm matches, a `SwitchExpressionException` (or `InvalidOperationException` on .NET Framework) is thrown. A discard pattern (`_`) is commonly used as the final arm to make the expression exhaustive.

**Beginner-Friendly Explanation**
The switch expression is a **compact, value-returning switch**. Instead of writing `case` and `break`, you write `pattern => result`. It fits naturally into assignments, return statements, and expression-bodied members. It is like a series of `if` statements that produces a single value.

---

### Purposes

- To select a value based on pattern matching in an expression context.
- To replace verbose `switch` statements that only assign or return a result.
- To support constant, relational, type, property, and discard patterns.
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
- `pattern` — A pattern.
- `when` — Optional case guard.
- `=>` — Separates pattern from result.
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

#### Example 2: Relational Patterns

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

- Relational patterns match the appropriate ranges.

---

### Real-World Cases

- **Mapping enums to display names** — `status switch { Status.Active => "Active", ... }`.
- **Type-based dispatch** — `shape switch { Circle c => ..., Square s => ... }`.
- **Range classification** — `score switch { >= 90 => "A", >= 80 => "B", ... }`.

---

### References

- switch expression — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/switch-expression
- Pattern matching overview — C# - https://learn.microsoft.com/en-us/dotnet/csharp/fundamentals/patterns/pattern-matching
- Switch expressions — C# 8.0 new features - https://learn.microsoft.com/en-us/dotnet/csharp/whats-new/csharp-8#switch-expressions

---

## 8. Pattern Matching in Switch

### Definitions

**Core Definition**
Pattern matching is a C# feature that tests an expression against a **pattern**—a condition that checks the value’s type, shape, properties, or relationship to constants—and, when it matches, can extract information.

**Technical Definition**
C# uses patterns in `is` expressions, `switch` statements, and `switch` expressions. A pattern tests whether a value has a particular type, equals a particular value, or has a particular shape. When a pattern matches, the code can use information learned by the test. C# supports **constant patterns**, **type patterns** (declaration patterns), **property patterns**, **positional patterns**, **relational patterns**, **logical patterns** (`and`, `or`, `not`), **list patterns** (C# 11), **var patterns**, and the **discard pattern** (`_`). Logical, property, positional, and list patterns are **recursive**—they can contain nested patterns.

**Beginner-Friendly Explanation**
Pattern matching lets you ask **“what does this data look like?”** and **“what can I learn from it?”** In a switch, you can match on types, property values, ranges, and combinations, all in a readable way.

---

### Purposes

- To test the runtime type of an object and safely extract a typed reference.
- To match against constants, ranges, and property values.
- To deconstruct tuples and positional records.
- To express complex conditions declaratively.
- To enable exhaustive dispatch in switch expressions.

---

### Syntax Rules and Structure (Pattern Types)

| Pattern | Example | Purpose |
|---|---|---|
| **Constant** | `42`, `"hello"`, `null` | Matches a specific constant value. |
| **Type** | `Circle c` | Matches a type and optionally captures. |
| **Property** | `{ Radius: > 10 }` | Matches an object whose property satisfies a pattern. |
| **Positional** | `(x, y)` | Matches a tuple or deconstructable object. |
| **Relational** | `> 0`, `<= 100` | Matches a value against a relational bound. |
| **Logical** | `> 0 and < 10`, `not null` | Combines patterns with `and`, `or`, `not`. |
| **Discard** | `_` | Matches anything. |
| **List** | `[1, 2, ..]` | Matches a sequence shape (C# 11). |

**Component breakdown**

- **Constant pattern** — Equality with a compile-time constant.
- **Type pattern** — Runtime type compatibility, optional variable capture.
- **Property pattern** — Tests properties or fields against nested patterns.
- **Relational pattern** — Tests `<`, `>`, `<=`, `>=` against a constant.
- **Logical pattern** — Combines patterns with `and`, `or`, `not`.
- **Discard pattern** — `_` matches anything.

#### Syntax Rules

- Patterns can be used in `is` expressions, `case` labels, and switch expression arms.
- Pattern variables are in scope only where the pattern matches.
- The `when` clause adds an additional Boolean condition.
- Patterns are evaluated in order; the first match wins.

#### Constraints and Limitations

- Constant patterns must be compile-time constants.
- Property patterns require the object to be non-null to access properties.
- Deeply nested patterns can be hard to read.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Type Pattern with Capture

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

- The `is Mammal m` pattern tests the runtime type and captures the value.

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

- Property patterns and guards match the shapes.

---

### Real-World Cases

- **Visitor pattern** — Type-based dispatch with extracted data.
- **Serialization** — Handling different JSON token types.
- **Domain modelling** — Classifying shapes, states, or commands.

---

### References

- Pattern matching overview — C# - https://learn.microsoft.com/en-us/dotnet/csharp/fundamentals/patterns/pattern-matching
- Patterns — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/patterns

---

## 9. Type Patterns

### Definitions

**Core Definition**
A type pattern tests whether an expression’s runtime type is compatible with a specified type and, optionally, captures the converted value in a new variable (a **declaration pattern**).

**Technical Definition**
A declaration pattern with type `T` matches an expression when the expression result is non-null and the runtime type has an identity conversion to `T`, derives from `T`, implements interface `T`, or another implicit reference conversion exists. The pattern optionally declares a variable that is assigned the converted value.

**Beginner-Friendly Explanation**
A type pattern asks: **“Is this object a `Circle` (or something derived from it)?”** If yes, you can capture it in a variable and use it immediately.

---

### Purposes

- To test runtime type compatibility.
- To safely downcast without a separate cast.
- To dispatch based on the concrete type of an object.
- To combine type checks with variable capture.

---

### Syntax Rules and Structure

```
TypeName   variableName
```

**Component Breakdown**

- `TypeName` — The type to test against.
- `variableName` — Optional variable to capture the value.

#### Syntax Rules

- The pattern matches when the runtime type is compatible with `T`.
- The variable is definitely assigned only when the pattern matches.
- Variable scope is limited to the matching branch.

#### Constraints and Limitations

- Does not consider user-defined conversions.
- The variable cannot be used outside the matching branch.

---

### Annotated Code Examples and Expected Outputs

#### Example: Type Pattern in Switch

```csharp
using System;

class Shape { }
class Circle : Shape { public double Radius { get; set; } }
class Square : Shape { public double Side { get; set; } }

class Program
{
    static string Describe(Shape shape) => shape switch
    {
        Circle c => $"Circle with radius {c.Radius}",
        Square s => $"Square with side {s.Side}",
        _ => "Unknown shape"
    };

    static void Main()
    {
        Console.WriteLine(Describe(new Circle { Radius = 5 }));   // Circle with radius 5
    }
}
```

**Expected Output**

```
Circle with radius 5
```

**Why this output occurs**

- The `Circle c` pattern matches and captures the `Circle` instance.

---

### Real-World Cases

- **Shape hierarchies** — Dispatching on concrete shape types.
- **Message handling** — Handling different message subclasses.
- **Plugin systems** — Checking for specific plugin interfaces.

---

### References

- Type patterns — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/patterns#type-patterns
- Declaration patterns — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/patterns#declaration-and-type-patterns

---

## 10. Constant Patterns

### Definitions

**Core Definition**
A constant pattern tests whether an expression result equals a specified **compile-time constant**.

**Technical Definition**
The constant pattern is an alternative syntax for `==` when the right operand is a constant. It tests whether an expression result equals a specified constant. In a constant pattern, you can use any constant expression: literals, `null`, enum values, `const` variables, and `nameof` expressions.

**Beginner-Friendly Explanation**
A constant pattern is a **direct comparison** with a fixed value. `case 42:` is a constant pattern.

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

- Must be a compile-time constant.
- Compatible with the governing type.
- `null` is permitted for reference and nullable types.

#### Constraints and Limitations

- Cannot use runtime expressions.
- Duplicate constants cause compile errors.

---

### Annotated Code Examples and Expected Outputs

#### Example: Constant Patterns for Enum Values

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

### Real-World Cases

- **Enum dispatch** — Matching enum members.
- **Command strings** — Matching specific commands.
- **Null checks** — `case null:`.

---

### References

- Constant pattern — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/patterns#constant-pattern

---

## 11. Relational Patterns

### Definitions

**Core Definition**
A relational pattern compares an expression result with a constant using `<`, `>`, `<=`, or `>=`.

**Technical Definition**
In a relational pattern, use any of the relational operators `<`, `>`, `<=`, or `>=`. The right-hand part must be a constant expression of an integer, floating-point, `char`, or enum type. Relational patterns do **not** work with types that lack a defined ordering, such as `string`. To check a range, combine relational patterns with `and`.

**Beginner-Friendly Explanation**
A relational pattern is a **range check**. `> 0` matches any positive number; `< 0 and > -10` matches a specific range.

---

### Purposes

- To match values against ranges.
- To classify numeric measurements.
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

#### Example: Classifying a Measurement

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
    }
}
```

**Expected Output**

```
Too high
```

**Why this output occurs**

- `13` is greater than `10.0`, matching `> 10.0`.

---

### Real-World Cases

- **Grading** — `>= 90`, `>= 80`.
- **Temperature classification** — `< 0`, `>= 30`.
- **Age ranges** — `>= 18 and < 65`.

---

### References

- Relational patterns — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/patterns#relational-patterns

---

## 12. Logical Patterns

### Definitions

**Core Definition**
Logical patterns combine other patterns using the `not`, `and`, and `or` keywords.

**Technical Definition**
Use `not`, `and`, and `or` to create logical patterns. **Negation** (`not pattern`) matches when the negated pattern does **not** match. **Conjunction** (`and pattern`) matches when both patterns match. **Disjunction** (`or pattern`) matches when either pattern matches. `not` binds tightest, then `and`, then `or`. Parentheses can override precedence.

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
pattern   and   pattern
pattern   or    pattern
```

**Component breakdown**

- `not` — Negation.
- `and` — Conjunction.
- `or` — Disjunction.

#### Syntax Rules

- `not` binds tightest, then `and`, then `or`.
- Use parentheses to clarify complex combinations.
- Can combine any patterns.

#### Constraints and Limitations

- Overly complex logical patterns reduce readability.
- Parentheses are often needed to get the intended binding.

---

### Annotated Code Examples and Expected Outputs

#### Example: Range and Alternative Patterns

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

### Real-World Cases

- **Season classification** — Month ranges.
- **Validation** — `not null and not empty`.
- **Categorisation** — `1 or 2 or 3`.

---

### References

- Logical patterns — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/patterns#logical-patterns

---

## 13. Property Patterns

### Definitions

**Core Definition**
A property pattern tests an expression’s **properties or fields** against nested patterns.

**Technical Definition**
A property pattern matches an expression when the expression result is non-null and every nested pattern matches the corresponding property or field. You can add a runtime type check and variable declaration. The empty property pattern `{ }` matches everything non-null. Property patterns are **recursive**—they can contain any nested pattern.

**Beginner-Friendly Explanation**
A property pattern lets you **look inside an object** and check its properties. `{ Radius: > 10 }` matches an object whose `Radius` is greater than `10`.

---

### Purposes

- To match objects based on their property values.
- To combine type checks with property checks.
- To deconstruct complex objects declaratively.
- To avoid nested `if` statements.

---

### Syntax Rules and Structure

```
{   Property1:   pattern1,   Property2:   pattern2,   ...   }
```

**Component breakdown**

- `{ }` — The property pattern.
- `Property: pattern` — A property/field and its nested pattern.

#### Syntax Rules

- The expression must be non-null for properties to be accessed.
- Properties/fields must be accessible.
- Nested patterns can be any pattern.
- The empty pattern `{ }` matches non-null.

#### Constraints and Limitations

- The object must be non-null; property patterns do not match `null`.
- Computed properties are evaluated.

---

### Annotated Code Examples and Expected Outputs

#### Example: Matching Conference Days

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

### Real-World Cases

- **Validation** — Checking object properties against rules.
- **Domain classification** — Matching records by field values.
- **UI logic** — Matching view-model states.

---

### References

- Property patterns — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/patterns#property-pattern

---

## 14. Positional Patterns

### Definitions

**Core Definition**
A positional pattern **deconstructs** an expression result and tests the resulting values against nested patterns.

**Technical Definition**
A positional pattern uses the deconstruction syntax `(pattern1, pattern2, ...)` to match a value that supports deconstruction (tuples, records with positional parameters, types with a `Deconstruct` method). Each nested pattern is matched against the corresponding deconstructed value.

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

#### Example: Matching a Point Record

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
    }
}
```

**Expected Output**

```
Origin
On Y-axis
```

**Why this output occurs**

- The positional pattern deconstructs the record and matches the values.

---

### Real-World Cases

- **Geometry** — Matching coordinates.
- **Parsing** — Matching tuple results.
- **Domain records** — Matching records by positional values.

---

### References

- Positional patterns — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/patterns#positional-pattern

---

## 15. List Patterns (C# 11)

### Definitions

**Core Definition**
A list pattern (introduced in **C# 11**) matches a **sequence** of elements against a combination of an element-count requirement and nested patterns at ordered positions.

**Technical Definition**
A list pattern describes a shape by combining an element-count requirement with nested patterns at ordered positions. It tests only the requirements written in the pattern. A list pattern works when the input’s compile-time type provides a `Length` or `Count` property and **indexed element access** (`input[index]`). Arrays, `List<T>`, strings, and spans are common supported types. The slice pattern `..` allows matching an arbitrary number of elements between specific positions. List patterns use length/count and indexing rather than enumeration.

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

#### Example: Matching a Header Row

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

#### Example: Capturing First and Last

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
- List patterns — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/patterns#list-patterns

---

## 16. Switch Best Practices

### Definitions

**Core Definition**
Switch best practices are the guidelines that ensure switch statements and expressions are readable, maintainable, correct, and performant.

**Technical Definition**
Best practices for switch derive from Microsoft’s C# coding conventions, the .NET Framework Design Guidelines, and the C# language reference. Key recommendations include: **prefer switch expressions when producing a value** and switch statements when performing actions; **use the most specific patterns first** because arms are evaluated in order; **always include a discard arm (`_`) or `default`** for exhaustiveness; **use `when` guards to refine patterns**; **prefer pattern matching over `is`/cast chains**; **use enums instead of magic strings/numbers** where possible; and **order `if`/`else if` chains by frequency** when inputs are skewed.

**Beginner-Friendly Explanation**
Best practices are the habits that keep switch code clean and safe: use switch expressions for values and switch statements for actions, put the most specific cases first, always have a fallback, and use patterns instead of long `if` chains.

---

### Purposes

- To choose the right switch form (statement vs. expression).
- To ensure all possible inputs are handled.
- To make pattern matching readable.
- To avoid subtle ordering bugs.
- To align with .NET conventions and tooling (e.g., IDE0066).

---

### Guidelines and Structure

#### General Guidelines

```
// Prefer switch expressions when producing a value
static string Describe(int x) => x switch
{
    < 0 => "Negative",
    0 => "Zero",
    _ => "Positive"
};

// Use switch statements for actions
switch (command)
{
    case "run":
        Run();
        break;
    case "save":
        Save();
        break;
    default:
        ShowHelp();
        break;
}

// Always include a fallback
_ => throw new ArgumentOutOfRangeException()

// Order specific patterns first
case Circle c when c.Radius > 10: ...
case Circle c: ...

// Use enums instead of magic values
case Status.Active: ...

// Use IDE0066 to prefer switch expressions where appropriate
```

**Component breakdown**

- **Switch expression for values** — Concise, returns a result.
- **Switch statement for actions** — Executes multiple statements.
- **Fallback arm** — Ensures exhaustiveness.
- **Specific-first ordering** — Prevents subsumption.
- **Enums** — Type safety and readability.

#### Rules and Constraints

- **Do** prefer switch expressions when each branch produces a value.
- **Do** include a `default` or discard arm.
- **Do** order cases from most specific to least specific.
- **Do** use `when` guards for additional conditions.
- **Do** use pattern matching instead of `is`/cast.
- **Do not** use switch for complex logic with many unrelated conditions; consider polymorphism or strategy patterns.
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

- if and switch statements — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/statements/selection-statements
- switch expression — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/switch-expression
- IDE0066: Use switch expression - .NET - https://learn.microsoft.com/en-us/dotnet/fundamentals/code-analysis/style-rules/ide0066
- Pattern matching overview — C# - https://learn.microsoft.com/en-us/dotnet/csharp/fundamentals/patterns/pattern-matching

---

## Deprecated, Unsafe, Implementation-Dependent, and Version-Specific Notes

| Feature | Status / Note |
|---------|---------------|
| `switch` expression | Introduced in **C# 8.0**; version-specific. |
| Pattern matching in `switch` | Introduced in **C# 7.0**; expanded in later versions. |
| Type patterns | C# 7.0. |
| Property patterns | C# 8.0. |
| Relational and logical patterns | C# 9.0. |
| List patterns | C# 11. |
| `goto case` / `goto default` | Available since C# 1.0; explicit only. |
| Fall-through | **Forbidden** in C# (unlike C/C++); use `goto case` for deliberate sharing. |
| `switch` vs. `if`/`else` performance | **Implementation-dependent**; modern compilers optimise both. Skewed `if`/`else` can outperform switch for frequent cases. |

---

## References (Consolidated)

- if and switch statements — select a code path to execute — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/statements/selection-statements
- switch statement — C# language specification - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/language-specification/statements#1388-the-switch-statement
- switch expression — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/switch-expression
- Pattern matching overview — C# - https://learn.microsoft.com/en-us/dotnet/csharp/fundamentals/patterns/pattern-matching
- Patterns — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/patterns
- List and slice patterns — C# - https://learn.microsoft.com/en-us/dotnet/csharp/fundamentals/patterns/list-patterns
- goto statement — C# language specification - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/language-specification/statements#13105-the-goto-statement
- IDE0066: Use switch expression - .NET - https://learn.microsoft.com/en-us/dotnet/fundamentals/code-analysis/style-rules/ide0066
- Switch expressions — C# 8.0 new features - https://learn.microsoft.com/en-us/dotnet/csharp/whats-new/csharp-8#switch-expressions

---