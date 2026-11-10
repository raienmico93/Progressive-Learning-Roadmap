# C# Comparison Operators — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
Comparison operators in C# are symbols and expressions that test the relationship between values—whether they are equal, unequal, greater than, less than, or type-compatible. They form the decision-making backbone of every C# program, powering conditions, loops, sorting, validation, and type dispatch. The family includes equality operators (`==`, `!=`), relational operators (`<`, `>`, `<=`, `>=`), type-testing operators (`is`, `as`), and metadata expressions (`typeof`, `nameof`, `sizeof`).

**Technical Definition**
C# classifies `==`, `!=`, `<`, `>`, `<=`, `>=`, `is`, and `as` as **relational and type-testing operators**. The operators `==`, `!=`, `<`, `>`, `<=`, and `>=` are **comparison operators**; all return `bool` for the predefined types. For value types, equality means **content equality**; for reference types, equality means **reference identity** (both variables refer to the same storage), unless the type overloads the equality operators. Relational operators (`<`, `>`, `<=`, `>=`) are defined for all integral and floating-point numeric types, `char` (comparing character codes), and enum types (comparing underlying integral values). The `is` operator performs a runtime type check (and pattern match), the `as` operator performs a safe reference conversion that returns `null` on failure, `typeof` obtains a `System.Type` instance from a compile-time type name, `nameof` produces the compile-time string name of a symbol, and `sizeof` returns the byte size of an unmanaged type.

**Beginner-Friendly Explanation**
Comparison operators are the **questions your program asks**: “Are these equal?”, “Is this bigger?”, “Is this object a `Mammal`?”, “What is the name of this variable?” The equality operators `==` and `!=` ask whether two things match. The relational operators `<`, `>`, `<=`, `>=` ask about order. The type-testing operators `is` and `as` ask about an object’s runtime type. `typeof`, `nameof`, and `sizeof` are compile-time helpers that reveal information about types and names without running code. Understanding the difference between value equality and reference equality—and knowing when to use `StringComparison`—is essential to writing correct C#.

---

### Key Characteristics

- **Two equality philosophies** — Value types compare contents; reference types compare identity by default.
- **User overloading** — Types can overload `==` and `!=`, changing their meaning; `is` is not affected by overloading.
- **NaN anomaly** — For `==`, `<`, `>`, `<=`, and `>=`, any comparison involving `double.NaN` or `float.NaN` returns `false`.
- **String equality is ordinal by default** — `==` on strings performs a case-sensitive, ordinal (binary) comparison.
- **`is` is runtime, `typeof` is compile-time** — `is` checks an object’s runtime type; `typeof` gets a `Type` from a type name written in source code.
- **`as` never throws** — It returns `null` when the conversion is impossible.
- **`nameof` is free** — Evaluated at compile time; no runtime cost.
- **`sizeof` is context-dependent** — Safe code allows built-in unmanaged types; unsafe code allows more.

---

### Prerequisites

- Understanding of C#’s built-in types (`int`, `double`, `string`, `object`, etc.).
- Familiarity with value types vs. reference types and inheritance.
- Knowledge of interfaces, classes, and structs.
- Awareness of operator overloading and `Equals`/`GetHashCode`.
- Basic understanding of `unsafe` code for `sizeof` on arbitrary types.

---

### Related Programming Areas

- **Control flow** — Conditions (`if`, `while`, `switch`) rely on comparison operators.
- **Sorting and ordering** — Relational operators and `IComparable<T>` drive sorting.
- **Type dispatch** — `is` and `as` enable pattern matching and safe downcasting.
- **Reflection and diagnostics** — `typeof` and `nameof` feed reflection, logging, and argument validation.
- **Interoperability and memory** — `sizeof` supports low-level memory calculations and P/Invoke.

---

### Core Concepts / Features

1. Equality Operators (`==`, `!=`)
2. Relational Operators (`<`, `>`, `<=`, `>=`)
3. `is` Operator (Type Testing and Pattern Matching)
4. `as` Operator (Safe Reference Conversion)
5. `typeof` Operator (Compile-Time Type Retrieval)
6. `nameof` Expression (Compile-Time Name Retrieval)
7. `sizeof` Operator (Compile-Time Size Retrieval)
8. Comparison Best Practices

Each concept follows a uniform structure: definitions, purposes, syntax rules and structure, annotated examples with expected outputs, real-world cases, and references.

---

## 1. Equality Operators (`==`, `!=`)

### Definitions

**Core Definition**
The equality operator (`==`) returns `true` if its operands are equal; the inequality operator (`!=`) returns `true` if they are not equal. Equality means **content equality** for value types and **reference identity** for reference types (unless overloaded).

**Technical Definition**
For the built-in value types, operands are equal if their **values** are equal. For reference types, operands are equal if both variables refer to the **same storage location**—that is, the same object instance. User-defined types can overload `==` and `!=` to provide value-based equality; record types receive value equality automatically. For `string`, the `==` operator is defined to compare **content** using a case-sensitive, ordinal (binary) comparison. For floating-point types, if either operand is `NaN`, `==` returns `false` and `!=` returns `true`. Enum operands are equal if their underlying integral values are equal.

**Beginner-Friendly Explanation**
`==` asks “Are these the same?” and `!=` asks “Are these different?” For numbers and structs, it compares the actual data. For classes, the default `==` asks “Are these the exact same object?”—not “Do they look similar?” Strings are special: `==` compares their text, not their identity. And `NaN` is equal to nothing, not even itself.

---

### Purposes

- To test whether two values or references are equal.
- To branch program logic based on equality.
- To validate inputs against expected values.
- To compare strings for content equality.
- To support custom equality semantics through operator overloading.

---

### Syntax Rules and Structure

```
leftOperand   ==   rightOperand   ;
leftOperand   !=   rightOperand   ;
```

**Component breakdown**

- `leftOperand` — The first value or reference.
- `==` / `!=` — The equality or inequality operator.
- `rightOperand` — The second value or reference.

#### Syntax Rules

- Both operands must be comparable; for user-defined types, an applicable `operator ==` or `operator !=` must exist.
- The result is always `bool`.
- For reference types, the default is reference identity.
- For `string`, `==` and `!=` compare content.
- For `double`/`float`, `NaN != NaN` is `true`.

#### Constraints and Limitations

- Do not use `==` to compare floating-point values for exact equality when precision errors are possible.
- Overloading `==` requires overloading `!=` and overriding `Equals` and `GetHashCode` for consistency.
- `==` on `object`-typed references uses reference identity unless the runtime type overloads it.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Value Equality vs. Reference Equality

```csharp
using System;

class Person
{
    public string Name;
    public Person(string name) => Name = name;
}

class Program
{
    static void Main()
    {
        // Value types: content equality
        int a = 1 + 2 + 3;
        int b = 6;
        Console.WriteLine(a == b);          // True

        // Reference types: identity by default
        var p1 = new Person("Alice");
        var p2 = new Person("Alice");
        Console.WriteLine(p1 == p2);         // False (different objects)

        var p3 = p1;
        Console.WriteLine(p1 == p3);         // True (same object)
    }
}
```

**Expected Output**

```
True
False
True
```

**Why this output occurs**

- `int` compares contents, so `6 == 6`.
- `Person` uses the default reference equality inherited from `object`, so two distinct instances are unequal.
- `p3 = p1` copies the reference, so both point to the same object.

---

#### Example 2: String Equality and `NaN`

```csharp
using System;

class Program
{
    static void Main()
    {
        string s1 = "Hello";
        string s2 = "Hel" + "lo";          // compile-time concatenation
        Console.WriteLine(s1 == s2);        // True (content equality)

        double nan = double.NaN;
        Console.WriteLine(nan == nan);      // False
        Console.WriteLine(nan != nan);      // True
    }
}
```

**Expected Output**

```
True
False
True
```

**Why this output occurs**

- `string` `==` compares content, and both strings contain `"Hello"`.
- `NaN` is not equal to itself by IEEE 754 rules, so `nan == nan` is `false` and `nan != nan` is `true`.

---

### Real-World Cases

- **Authentication** — Comparing hashed passwords or tokens (using constant-time comparison for security).
- **Validation** — Checking user input against expected values.
- **State machines** — Testing whether the current state equals a target state.
- **Caching** — Comparing keys in dictionaries.

---

### References

- Equality operators — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/equality-operators
- How to compare strings in C# - https://learn.microsoft.com/en-us/dotnet/csharp/how-to/compare-strings
- Best practices for comparing strings in .NET - https://learn.microsoft.com/en-us/dotnet/standard/base-types/best-practices-strings
- Object.Equals Method — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.object.equals

---

## 2. Relational Operators (`<`, `>`, `<=`, `>=`)

### Definitions

**Core Definition**
Relational operators compare the order of two operands: `<` (less than), `>` (greater than), `<=` (less than or equal), and `>=` (greater than or equal). They return `bool`.

**Technical Definition**
All integral and floating-point numeric types support `<`, `>`, `<=`, and `>=`. The `char` type supports these operators by comparing the numeric character codes. Enum types support them by comparing the underlying integral values of operands of the same enum type. For `double` and `float`, if any operand is `NaN`, the result is `false` for all four relational operators, and also for `==`. The C# language specification defines the predefined integer, floating-point, decimal, and enum relational operators, and all return `bool`.

**Beginner-Friendly Explanation**
These operators ask about **order**: “Is this smaller?”, “Is that bigger?”, “Is this at most that?” They work on numbers, characters (by their codes), and enums (by their numeric values). Watch out for `NaN`: it is not smaller than, greater than, or equal to anything—even itself.

---

### Purposes

- To compare numeric magnitudes.
- To implement sorting and ordering logic.
- To enforce ranges and bounds (e.g., `age >= 18`).
- To drive loops with conditions (`i < length`).
- To compare `char` values by their character codes.
- To compare enum values by their underlying integers.

---

### Syntax Rules and Structure

```
leftOperand   <    rightOperand   ;
leftOperand   >    rightOperand   ;
leftOperand   <=   rightOperand   ;
leftOperand   >=   rightOperand   ;
```

**Component breakdown**

- `leftOperand` — The first comparable value.
- `<`, `>`, `<=`, `>=` — The relational operators.
- `rightOperand` — The second comparable value.

#### Syntax Rules

- Operands must be numeric, `char`, or the same enum type.
- The result is `bool`.
- Relational operators are not defined for `bool` (other than `==`/`!=`).
- `NaN` makes all relational comparisons return `false`.

#### Constraints and Limitations

- Relational operators cannot be used with `string` directly; use `string.Compare` or `StringComparer`.
- For custom types, implement `IComparable<T>` and consider overloading `<` and `>` for consistency.
- Floating-point comparisons can be imprecise; use tolerances when appropriate.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Numeric and Character Comparisons

```csharp
using System;

class Program
{
    static void Main()
    {
        Console.WriteLine(7.0 < 5.1);      // False
        Console.WriteLine(5.1 < 5.1);      // False
        Console.WriteLine(0.0 < 5.1);      // True

        char a = 'a';
        char A = 'A';
        Console.WriteLine(a < A);          // False ('a' = 97, 'A' = 65)
        Console.WriteLine(A < a);          // True
    }
}
```

**Expected Output**

```
False
False
True
False
True
```

**Why this output occurs**

- Numeric comparisons follow mathematical order.
- `char` comparisons use the character’s numeric code (`'A'` is `65`, `'a'` is `97`).

---

#### Example 2: Enum and `NaN` Comparisons

```csharp
using System;

enum Priority { Low, Medium, High }

class Program
{
    static void Main()
    {
        Console.WriteLine(Priority.Low < Priority.High);   // True

        double nan = double.NaN;
        Console.WriteLine(nan < 5.0);       // False
        Console.WriteLine(nan > 5.0);       // False
        Console.WriteLine(nan <= 5.0);      // False
        Console.WriteLine(nan >= 5.0);      // False
    }
}
```

**Expected Output**

```
True
False
False
False
False
```

**Why this output occurs**

- Enums compare by their underlying integer values (`Low = 0`, `High = 2`).
- `NaN` is unordered with respect to every number, so all relational comparisons return `false`.

---

### Real-World Cases

- **Pagination** — `page < totalPages`.
- **Grading** — `score >= 90`.
- **Game physics** — `velocity > 0`.
- **Time comparisons** — `DateTime` comparisons using `<` and `>`.

---

### References

- Comparison operators — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/comparison-operators
- Relational and type-testing operators — C# language specification - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/language-specification/expressions#1212-relational-and-type-testing-operators

---

## 3. `is` Operator (Type Testing and Pattern Matching)

### Definitions

**Core Definition**
The `is` operator checks whether an expression’s runtime type is compatible with a given type, returning `bool`. It also supports pattern matching, allowing the tested value to be captured in a new variable.

**Technical Definition**
The `is` operator returns `true` when the expression result is non-null and any of the following conditions hold: the runtime type has an identity conversion to `T`; the runtime type derives from, implements, or has an implicit reference conversion to `T`; or a boxing, unboxing, or nullable value-type conversion exists. The `is` operator **does not** consider user-defined conversions or implicit span conversions. The `is` pattern form (`expr is T variable`) tests and assigns in one step; the variable is definitely assigned only when the pattern succeeds. The `is` operator can also match constant patterns, property patterns, relational patterns, and list patterns.

**Beginner-Friendly Explanation**
`is` asks: “Is this object compatible with this type?” `obj is Mammal` returns `true` if `obj` is a `Mammal` or something derived from it. The pattern form `if (obj is Mammal m)` not only checks but also gives you a typed variable `m` to use inside the `if`. `is` never throws and never converts—it only tests.

---

### Purposes

- To safely test an object’s runtime type before casting.
- To avoid `InvalidCastException` by checking compatibility first.
- To combine type testing and variable assignment with the `is` pattern.
- To implement type-based branching and pattern matching.
- To test against constants, properties, and list shapes.

---

### Syntax Rules and Structure

```
expression   is   T
expression   is   T   variable
```

**Component breakdown**

- `expression` — The value to test.
- `is` — The type-testing/pattern-matching operator.
- `T` — The type or pattern.
- `variable` — Optional pattern variable assigned when the test succeeds.

#### Syntax Rules

- Returns `true` when the runtime type is compatible with `T`.
- Returns `false` for `null`.
- The pattern variable is in scope only where the pattern is true.
- `is` ignores user-defined conversion operators.

#### Constraints and Limitations

- Does not perform conversions; only tests.
- For value types, `is` checks the exact type (boxing) or nullable `HasValue`.
- Pattern variables cannot be used outside the successful branch.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Basic Type Test and Pattern

```csharp
using System;

class Animal { public string Name { get; set; } }
class Mammal : Animal { }

class Program
{
    static void Main()
    {
        Animal a = new Mammal { Name = "Otter" };

        Console.WriteLine(a is Animal);      // True
        Console.WriteLine(a is Mammal);      // True
        Console.WriteLine(a is string);      // False

        if (a is Mammal m)
        {
            Console.WriteLine(m.Name);       // Otter
        }
    }
}
```

**Expected Output**

```
True
True
False
Otter
```

**Why this output occurs**

- A `Mammal` is an `Animal` (inheritance) and a `Mammal` (identity).
- It is not a `string`.
- The `is Mammal m` pattern assigns `m` when the test succeeds.

---

#### Example 2: `is` with Nullable and Constants

```csharp
using System;

class Program
{
    static void Main()
    {
        int? maybe = 7;
        Console.WriteLine(maybe is int);     // True (HasValue is true)

        object value = null;
        Console.WriteLine(value is null);    // True (null pattern)

        object number = 42;
        Console.WriteLine(number is int n && n > 10); // True
    }
}
```

**Expected Output**

```
True
True
True
```

**Why this output occurs**

- A nullable with `HasValue == true` matches its underlying type.
- The `is null` pattern matches `null`.
- The pattern combines type testing with a relational condition.

---

### Real-World Cases

- **Event handlers** — Checking whether `sender` is a specific control type.
- **Plugin systems** — Verifying that an object implements a required interface.
- **Visitor patterns** — Dispatching behaviour based on runtime type with captured variables.
- **Input validation** — Extracting typed values from `object` parameters.

---

### References

- Type-testing operators and cast expressions — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/type-testing-and-cast
- The `is` operator — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/type-testing-and-cast#the-is-operator
- Pattern matching — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/patterns

---

## 4. `as` Operator (Safe Reference Conversion)

### Definitions

**Core Definition**
The `as` operator attempts to convert an expression to a reference type or nullable value type, returning `null` instead of throwing if the conversion fails.

**Technical Definition**
The `as` operator performs a safe conversion that considers only reference, nullable, boxing, and unboxing conversions. It **does not** consider user-defined conversions. In the form `E as T`, `E` is evaluated only once, and the result is equivalent to `E is T ? (T)E : (T)null`. If the conversion is not possible, the result is `null`; `as` never throws an exception. The target type `T` must be a reference type, a type parameter known to be a reference type, or a nullable value type.

**Beginner-Friendly Explanation**
`as` is the **safe cast**. Instead of crashing when a conversion fails, it quietly returns `null`. You then check for `null` before using the result. It is useful when you are not certain what type an object is.

---

### Purposes

- To safely attempt a reference-type conversion without exceptions.
- To simplify code that would otherwise use `is` followed by a cast.
- To obtain a nullable result when a conversion might fail.
- To work with base-typed references that may or may not be a derived type.
- To convert to interfaces safely in plugin-style scenarios.

---

### Syntax Rules and Structure

```
expression   as   T
```

**Component breakdown**

- `expression` — The value to convert.
- `as` — The safe conversion operator.
- `T` — A reference type or nullable value type.

#### Syntax Rules

- Returns `null` if the conversion fails.
- Considers reference, nullable, boxing, and unboxing conversions only.
- Cannot be used for user-defined conversions or numeric conversions.
- `T` must be a reference type or nullable value type.

#### Constraints and Limitations

- Cannot be used with non-nullable value types (use `is` pattern instead).
- The result must be checked for `null` before use.
- Does not consider user-defined conversion operators.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Safe Interface Conversion

```csharp
using System;
using System.Collections.Generic;

class Program
{
    static void Main()
    {
        IEnumerable<int> numbers = new List<int> { 10, 20, 30 };
        IList<int>? list = numbers as IList<int>;

        if (list != null)
        {
            Console.WriteLine(list[0] + list[list.Count - 1]); // 40
        }
    }
}
```

**Expected Output**

```
40
```

**Why this output occurs**

- `List<int>` implements `IList<int>`, so the `as` conversion succeeds.
- The result is checked for `null` before use.

---

#### Example 2: Failed Conversion Returns Null

```csharp
using System;

class Animal { }
class Mammal : Animal { }
class Reptile : Animal { }

class Program
{
    static void Main()
    {
        object reptile = new Reptile();
        Mammal? mammal = reptile as Mammal;
        Console.WriteLine(mammal is null ? "Safe null" : "Unexpected");
    }
}
```

**Expected Output**

```
Safe null
```

**Why this output occurs**

- A `Reptile` cannot be converted to `Mammal`, so `as` returns `null` without throwing.

---

### Real-World Cases

- **Event handlers** — Safely casting `sender` to a specific control type.
- **Plugin discovery** — Checking whether an object implements a plugin interface.
- **Deserialization** — Attempting to cast deserialized objects to expected types.
- **Dependency injection** — Resolving optional interface implementations.

---

### References

- The `as` operator — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/type-testing-and-cast#the-as-operator
- Type-testing operators and cast expressions — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/type-testing-and-cast

---

## 5. `typeof` Operator (Compile-Time Type Retrieval)

### Definitions

**Core Definition**
The `typeof` operator obtains the `System.Type` instance for a type specified at compile time.

**Technical Definition**
The `typeof(T)` operator returns the `System.Type` object for the type or type parameter `T`. The argument must be the **name of a type or a type parameter**, not an expression. `typeof` can be used with constructed generic types (`typeof(List<int>)`) and unbound generic types (`typeof(Dictionary<,>)`). It cannot be used with `dynamic` or nullable reference type annotations; use the underlying type instead. To obtain the runtime type of an expression, use `Object.GetType()`.

**Beginner-Friendly Explanation**
`typeof(SomeType)` gives you a **description of the type itself**. It is a compile-time way to get type information for reflection, comparison, or API calls. It is different from `GetType()`, which asks an actual object what its runtime type is.

---

### Purposes

- To obtain a `System.Type` instance for reflection.
- To pass type information to APIs that accept `Type` parameters.
- To compare compile-time types (`GetType() == typeof(T)`).
- To create instances via reflection (`Activator.CreateInstance`).
- To inspect metadata such as methods and attributes.

---

### Syntax Rules and Structure

```
typeof(T)
```

**Component breakdown**

- `typeof` — The operator.
- `T` — The type name or type parameter.

#### Syntax Rules

- The argument must be a type name, not an expression.
- Cannot be used with `dynamic` or nullable reference types.
- Unbound generic types use commas for type parameter count (`Dictionary<,>`).

#### Constraints and Limitations

- Does not accept variables or runtime expressions.
- Returns a `System.Type`, not an instance of the type.
- For runtime type discovery, use `GetType()`.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Basic `typeof` Usage

```csharp
using System;
using System.Collections.Generic;

class Program
{
    static void Main()
    {
        Console.WriteLine(typeof(int));              // System.Int32
        Console.WriteLine(typeof(List<string>));     // System.Collections.Generic.List`1[System.String]
        Console.WriteLine(typeof(Dictionary<,>));    // System.Collections.Generic.Dictionary`2[TKey,TValue]
    }
}
```

**Expected Output**

```
System.Int32
System.Collections.Generic.List`1[System.String]
System.Collections.Generic.Dictionary`2[TKey,TValue]
```

**Why this output occurs**

- `typeof` returns the `Type` object for each type, including constructed and unbound generic types.

---

#### Example 2: `typeof` vs. `GetType`

```csharp
using System;

class Animal { }
class Giraffe : Animal { }

class Program
{
    static void Main()
    {
        object b = new Giraffe();
        Console.WriteLine(b is Animal);                   // True
        Console.WriteLine(b.GetType() == typeof(Animal)); // False (runtime type is Giraffe)
        Console.WriteLine(b.GetType() == typeof(Giraffe)); // True
    }
}
```

**Expected Output**

```
True
False
True
```

**Why this output occurs**

- `is` checks compatibility; `GetType()` returns the exact runtime type.
- `typeof(Animal)` is the compile-time type, which does not match the runtime `Giraffe`.

---

### Real-World Cases

- **Dependency injection** — Registering services by `Type`.
- **Serialization** — Inspecting type metadata for JSON/XML serialization.
- **Dynamic instantiation** — `Activator.CreateInstance(typeof(T))`.
- **Attribute reflection** — Retrieving attributes applied to a type.

---

### References

- The `typeof` operator — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/type-testing-and-cast#the-typeof-operator
- Type-testing operators and cast expressions — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/type-testing-and-cast

---

## 6. `nameof` Expression (Compile-Time Name Retrieval)

### Definitions

**Core Definition**
The `nameof` expression produces the **string name** of a variable, type, or member, evaluated entirely at compile time.

**Technical Definition**
A `nameof` expression returns the simple (unqualified) name of its operand as a string constant. It is evaluated at compile time and has **no runtime effect**. The operand can be a type, namespace, variable, member, or method parameter. When the operand is a type or namespace, the result is the unqualified name. `nameof` is commonly used in argument validation (`ArgumentNullException(nameof(value))`), logging, and diagnostic messages.

**Beginner-Friendly Explanation**
`nameof(someVariable)` returns the **text `"someVariable"`** as a string, and the compiler does the work—no runtime lookup. It is a safe, refactoring-friendly alternative to hard-coded strings.

---

### Purposes

- To obtain the name of a symbol as a string at compile time.
- To make argument-checking code refactoring-safe.
- To avoid magic strings in logging and exceptions.
- To reference member names in attributes and diagnostics.
- To improve maintainability when identifiers change.

---

### Syntax Rules and Structure

```
nameof(expression)
```

**Component breakdown**

- `nameof` — The compile-time name expression.
- `expression` — A type, namespace, variable, member, or parameter.

#### Syntax Rules

- Evaluated at compile time; no runtime cost.
- Returns the unqualified name.
- For generic types, returns the simple name without type arguments.
- Works with verbatim identifiers (`@class` returns `"class"`).

#### Constraints and Limitations

- Cannot produce a fully qualified name directly.
- The operand must be a symbol that can be named; arbitrary expressions may not be valid.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Basic `nameof` Usage

```csharp
using System;
using System.Collections.Generic;

class Program
{
    static void Main()
    {
        Console.WriteLine(nameof(System.Collections.Generic)); // Generic
        Console.WriteLine(nameof(List<int>));                 // List
        Console.WriteLine(nameof(List<int>.Count));           // Count
        Console.WriteLine(nameof(List<int>.Add));             // Add

        List<int> numbers = new List<int> { 1, 2, 3 };
        Console.WriteLine(nameof(numbers));                   // numbers
        Console.WriteLine(nameof(numbers.Count));             // Count
    }
}
```

**Expected Output**

```
Generic
List
Count
Add
numbers
Count
```

**Why this output occurs**

- `nameof` returns the simple name of each symbol.
- Generic type arguments are stripped.

---

#### Example 2: Argument Validation

```csharp
using System;

class Program
{
    static void SetName(string value)
    {
        if (value is null)
            throw new ArgumentNullException(nameof(value));
    }

    static void Main()
    {
        try { SetName(null); }
        catch (ArgumentNullException ex)
        {
            Console.WriteLine(ex.ParamName); // value
        }
    }
}
```

**Expected Output**

```
value
```

**Why this output occurs**

- `nameof(value)` produces `"value"` at compile time, which is used as the parameter name.

---

### Real-World Cases

- **Argument exceptions** — `throw new ArgumentNullException(nameof(param));`
- **Logging** — `logger.Log($"{nameof(Customer)} created");`
- **INotifyPropertyChanged** — `OnPropertyChanged(nameof(FirstName));`
- **Attribute arguments** — Passing member names to attributes.

---

### References

- `nameof` expression — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/nameof

---

## 7. `sizeof` Operator (Compile-Time Size Retrieval)

### Definitions

**Core Definition**
The `sizeof` operator returns the number of **bytes** occupied by a variable of a given type.

**Technical Definition**
In **safe code**, the argument to `sizeof` must be the name of a built-in unmanaged type whose size is not platform-dependent, or an enum type. For these types, `sizeof` is evaluated at compile time to a constant. In **unsafe code**, `sizeof` can be applied to any non-void type, including constructed types. The size of a reference or pointer type is the size of the reference/pointer itself, not the object it refers to. The size of an enum is the size of its underlying integral type. A memory-safety preview in C# 15 allows `sizeof` on any unmanaged type outside an unsafe context.

**Beginner-Friendly Explanation**
`sizeof(int)` tells you that an `int` takes **4 bytes** of memory. In safe code, you can use it on built-in types like `int`, `double`, and `bool`. For custom structs or unmanaged types, you may need `unsafe` code—unless you are using a newer C# version that relaxes this rule.

---

### Purposes

- To determine the memory footprint of a type.
- To perform low-level memory calculations and buffer sizing.
- To support P/Invoke and interop scenarios.
- To validate serialization and binary layout assumptions.

---

### Syntax Rules and Structure

```
sizeof(T)
```

**Component breakdown**

- `sizeof` — The operator.
- `T` — A type name.

#### Syntax Rules

- Safe code: built-in unmanaged types and enums only.
- Unsafe code: any non-void type.
- Evaluated at compile time for built-in types.
- `sizeof` returns an `int`.

#### Constraints and Limitations

- Not all types can be used in safe code.
- The size of `string` or a class reference is the reference size, not the object size.
- Platform-dependent for pointers and references.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Built-in Type Sizes

```csharp
using System;

class Program
{
    static void Main()
    {
        Console.WriteLine(sizeof(int));      // 4
        Console.WriteLine(sizeof(double));   // 8
        Console.WriteLine(sizeof(decimal));  // 16
        Console.WriteLine(sizeof(bool));     // 1
        Console.WriteLine(sizeof(char));     // 2
    }
}
```

**Expected Output**

```
4
8
16
1
2
```

**Why this output occurs**

- `sizeof` returns the compile-time byte size of each built-in unmanaged type.

---

#### Example 2: Enum Size

```csharp
using System;

enum Status : byte { Active, Inactive }

class Program
{
    static void Main()
    {
        Console.WriteLine(sizeof(Status));   // 1
    }
}
```

**Expected Output**

```
1
```

**Why this output occurs**

- The enum’s underlying type is `byte`, so its size is 1 byte.

---

### Real-World Cases

- **Interop** — Sizing structs for native APIs.
- **Serialization** — Computing buffer sizes for binary formats.
- **Low-level programming** — Managing unmanaged memory.
- **Performance analysis** — Understanding memory layout.

---

### References

- `sizeof` operator — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/sizeof

---

## 8. Comparison Best Practices

### Definitions

**Core Definition**
Comparison best practices are the guidelines that ensure comparisons are correct, safe, performant, and predictable, especially for strings, floating-point numbers, and custom types.

**Technical Definition**
Best practices encompass using explicit `StringComparison` values, avoiding culture-sensitive defaults for culture-agnostic data, understanding reference vs. value equality, implementing `Equals` and `GetHashCode` consistently with `==` overloading, using tolerances for floating-point comparisons, and preferring `is` patterns over `as`-plus-null checks for type testing. Microsoft’s guidance emphasises `StringComparison.Ordinal` or `OrdinalIgnoreCase` as the safe default for culture-agnostic string matching, and `CurrentCulture` for user-facing linguistic comparisons.

**Beginner-Friendly Explanation**
Best practices are the habits that keep comparison code from producing surprising results: be explicit about string comparison rules, do not compare floating-point numbers for exact equality, use `is` patterns instead of dangerous casts, and keep `==`, `Equals`, and `GetHashCode` consistent.

---

### Purposes

- To avoid culture-related bugs in string comparisons.
- To prevent precision errors in floating-point comparisons.
- To ensure custom types behave correctly in collections and dictionaries.
- To write clear, intention-revealing comparison code.
- To choose the right operator for type testing and conversion.

---

### Guidelines and Structure

#### General Guidelines

```
// Use explicit StringComparison for string comparisons
if (text.Equals("value", StringComparison.Ordinal)) { ... }

// Use Ordinal or OrdinalIgnoreCase for identifiers, paths, and keys
if (path.StartsWith("C:\\", StringComparison.OrdinalIgnoreCase)) { ... }

// Use CurrentCulture for user-facing linguistic comparisons
if (displayName.Equals(other, StringComparison.CurrentCulture)) { ... }

// Avoid exact equality for floating-point numbers
if (Math.Abs(a - b) < 1e-9) { ... }   // tolerance-based

// Prefer is patterns over as + null checks
if (obj is Mammal m) { ... }

// Keep ==, Equals, and GetHashCode consistent
public override bool Equals(object? obj) => obj is MyType t && Value == t.Value;
public override int GetHashCode() => Value.GetHashCode();
```

**Component breakdown**

- **Explicit `StringComparison`** — Avoids hidden culture-sensitive defaults.
- **`Ordinal` / `OrdinalIgnoreCase`** — Safe default for culture-agnostic data.
- **`CurrentCulture`** — For user-visible linguistic comparisons.
- **Floating-point tolerance** — Avoids precision pitfalls.
- **`is` pattern** — Combines testing and assignment safely.
- **Equality consistency** — Required for hash-based collections.

#### Rules and Constraints

- **Do not** use `StringComparison.InvariantCulture` for culture-agnostic data; use `Ordinal` instead.
- **Do not** use `Compare` or `CompareTo` return value `0` to test equality; use `Equals`.
- **Do not** overload `==` without overriding `Equals` and `GetHashCode`.
- **Do not** compare `double` or `float` for exact equality when precision errors are possible.
- **Do not** use `as` when a `null` result is a valid outcome; prefer `is` pattern.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Explicit String Comparison

```csharp
using System;

class Program
{
    static void Main()
    {
        string root = @"C:\users";
        string root2 = @"C:\Users";

        Console.WriteLine(root == root2);                               // False (ordinal, case-sensitive)
        Console.WriteLine(root.Equals(root2, StringComparison.OrdinalIgnoreCase)); // True
    }
}
```

**Expected Output**

```
False
True
```

**Why this output occurs**

- `==` uses case-sensitive ordinal comparison by default.
- `OrdinalIgnoreCase` explicitly ignores case without culture rules.

---

#### Example 2: Floating-Point Tolerance

```csharp
using System;

class Program
{
    static void Main()
    {
        double a = 0.1 + 0.2;
        double b = 0.3;

        Console.WriteLine(a == b);                       // False (precision)
        Console.WriteLine(Math.Abs(a - b) < 1e-9);       // True (tolerance)
    }
}
```

**Expected Output**

```
False
True
```

**Why this output occurs**

- Binary floating-point cannot represent `0.1 + 0.2` exactly as `0.3`.
- A tolerance-based comparison handles the tiny difference.

---

### Real-World Cases

- **Security** — Using `Ordinal` for token and identifier comparisons to avoid culture tricks.
- **User interfaces** — Using `CurrentCulture` for sorting and display.
- **Financial systems** — Avoiding exact floating-point equality; using `decimal` for money.
- **Collection design** — Implementing `Equals` and `GetHashCode` consistently for custom keys.

---

### References

- Best practices for comparing strings in .NET - https://learn.microsoft.com/en-us/dotnet/standard/base-types/best-practices-strings
- How to compare strings in C# - https://learn.microsoft.com/en-us/dotnet/csharp/how-to/compare-strings
- Guidelines for implementing Equals and the equality operator (==) - https://learn.microsoft.com/en-us/dotnet/standard/design-guidelines/equality
- Object.Equals Method — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.object.equals
- Type-testing operators and cast expressions — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/type-testing-and-cast

---

## Deprecated, Unsafe, Implementation-Dependent, and Version-Specific Notes

| Feature | Status / Note |
|---------|---------------|
| `nameof` | Introduced in **C# 6.0**; compile-time only. |
| `is` pattern with variable designation | Introduced in **C# 7.0**; earlier `is` returned only `bool`. |
| `sizeof` on arbitrary unmanaged types in safe code | **C# 15 memory-safety preview**; otherwise requires `unsafe` context. |
| `typeof` with nullable reference types | **Not allowed**; use the underlying type (`string` instead of `string?`). |
| `==` on `double.NaN` / `float.NaN` | Always `false` (IEEE 754); **language-defined**, not implementation-dependent. |
| String `==` default | **Ordinal, case-sensitive**; not culture-sensitive. |
| Overloading `==` without `Equals`/`GetHashCode` | **Breaks hash-based collections**; violates guidelines. |
| `as` with non-nullable value types | **Compile-time error**; use `is` pattern instead. |

---

## References (Consolidated)

- Equality operators — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/equality-operators
- Comparison operators — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/comparison-operators
- Relational and type-testing operators — C# language specification - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/language-specification/expressions#1212-relational-and-type-testing-operators
- Type-testing operators and cast expressions — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/type-testing-and-cast
- The `is` operator — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/type-testing-and-cast#the-is-operator
- The `as` operator — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/type-testing-and-cast#the-as-operator
- The `typeof` operator — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/type-testing-and-cast#the-typeof-operator
- `nameof` expression — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/nameof
- `sizeof` operator — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/sizeof
- How to compare strings in C# - https://learn.microsoft.com/en-us/dotnet/csharp/how-to/compare-strings
- Best practices for comparing strings in .NET - https://learn.microsoft.com/en-us/dotnet/standard/base-types/best-practices-strings
- Guidelines for implementing Equals and the equality operator (==) - https://learn.microsoft.com/en-us/dotnet/standard/design-guidelines/equality
- Object.Equals Method — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.object.equals
- Pattern matching — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/patterns

---