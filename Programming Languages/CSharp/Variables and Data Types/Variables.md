# C# Variables — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
A variable in C# is a named storage location whose value can change during program execution. Every variable has a **type** that determines what values it can hold and what operations can be performed on it. C# is a type-safe language: the compiler guarantees that values stored in a variable always match its declared type.

**Technical Definition**
The C# Language Specification defines a variable as a storage location. C# classifies variables into eight categories: static variables, instance variables, array elements, value parameters, input parameters, reference parameters, output parameters, and local variables. Each variable belongs to exactly one category, and that category determines its lifetime, scope, initial assignment state, and storage location.

**Beginner-Friendly Explanation**
Think of a variable as a labeled box. The label is the variable name, the shape of the box is the type (a box for whole numbers cannot hold text), and the contents are the value. You can put something in the box (assignment), look at what is inside (reading), and replace the contents with something else of the same shape (reassignment). C# checks at compile time that you never try to put a triangle into a square box.

---

### Key Characteristics

- **Strongly typed** — Every variable has a compile-time type; implicit conversions are limited and explicit conversions may require casts.
- **Definite assignment** — A local variable must be definitely assigned before its value can be read; the compiler performs static flow analysis to prove this.
- **Categories determine behaviour** — The eight variable categories differ in default values, lifetime, and whether they are initially assigned.
- **Storage location is an implementation detail** — The specification does not mandate stack vs. heap; it mandates lifetime and assignment guarantees. Local variables and parameters typically use the stack; objects referenced by variables live on the managed heap.
- **Scope and declaration space** — Names are introduced into declaration spaces; nested spaces may hide outer names, and it is an error for a nested space to redeclare a name from an enclosing space.
- **Modifiers change mutability and binding** — `const`, `readonly`, `static`, and `dynamic` alter when a value is fixed, where it lives, or when type checking occurs.

---

### Prerequisites

To get the most from this cheat sheet, you should be familiar with:

- Basic C# program structure (`class`, `Main`, namespaces).
- Primitive types (`int`, `double`, `bool`, `char`, `string`) and reference vs. value types.
- Basic control flow (`if`, `for`, `while`) because scope and definite assignment depend on control-flow paths.
- The concept of a compiler and runtime errors.

---

### Related Programming Areas

- **Type systems and type safety** — C# variables are the entry point to understanding static typing, implicit typing, and dynamic binding.
- **Memory management** — Scope and lifetime connect to stack allocation, heap allocation, and garbage collection.
- **Object-oriented programming** — Fields (instance and static) are variables associated with objects or types.
- **Concurrency** — `static` fields are shared state; `readonly` and `const` interact with thread-safety and immutability.
- **Interoperability** — `dynamic` is heavily used in COM interop, IronPython, and other dynamic-language scenarios.

---

### Core Concepts / Features

1. Variables (overarching concept and categories)
2. Variable declaration
3. Variable initialization
4. Variable assignment
5. `var` keyword
6. Implicit typing
7. `const`
8. `readonly`
9. `static`
10. `dynamic`
11. Variable scope
12. Variable lifetime
13. Variable naming
14. Variable best practices

Each concept below follows the same uniform structure: definitions, purposes, syntax rules and structure, annotated examples with expected outputs, real-world cases, and references.

---

## 1. Variables (Overarching Concept and Categories)

### Definitions

**Core Definition**
A variable is a named storage location with a type. The type determines the set of values the variable can hold and the operations that can be applied.

**Technical Definition**
The specification states that variables represent storage locations, and C# defines eight categories: static variables, instance variables, array elements, value parameters, input parameters, reference parameters, output parameters, and local variables. Each category has distinct rules for initial assignment, lifetime, and scope.

**Beginner-Friendly Explanation**
“Variable” is the general word for any labeled storage slot in your program. Some slots belong to the program as a whole (static), some belong to individual objects (instance), some exist only while a method runs (local), and some are placeholders for values passed into methods (parameters).

---

### Purposes

- To store data that can be read and modified during execution.
- To give meaningful names to values so code is readable and maintainable.
- To enforce type safety by restricting what can be stored in each location.
- To model state (fields) and intermediate computations (locals).
- To pass data into and out of methods through parameters.

---

### Syntax Rules and Structure

#### General Syntax (Variable Categories)

```
// Static variable (field with static modifier)
static   type   identifier   ;

// Instance variable (field without static)
type   identifier   ;

// Array element
arrayExpression [ index ]

// Value parameter
type   parameterName

// Reference parameter
ref   type   parameterName

// Output parameter
out   type   parameterName

// Input (readonly reference) parameter
in   type   parameterName

// Local variable
type   identifier   [ = initializer ]   ;
```

**Component breakdown**

- `static` — Modifier indicating the field belongs to the type, not an instance.
- `type` — The compile-time type of the variable; determines valid values and operations.
- `identifier` — The variable name; must follow identifier rules.
- `[ index ]` — Array access expression; array elements are variables in their own right.
- `ref` / `out` / `in` — Parameter modifiers that change the direction of data flow and definite-assignment rules.
- `= initializer` — Optional expression that supplies an initial value.

#### Syntax Rules

- A variable must be **definitely assigned** before its value is read.
- Static and instance fields are initially assigned to their default values; locals are initially unassigned.
- The eight categories are mutually exclusive: a variable belongs to exactly one category.
- Array elements are initially assigned (to default values) when the array is created.
- `out` parameters must be assigned by the called method before it returns.
- `in` parameters are passed by reference but cannot be modified inside the method (unless the compiler can prove no modification).

#### Constraints and Limitations

- You cannot declare a variable with the same name in the same declaration space as an enclosing declaration space.
- Local variables have no default value; reading an unassigned local is a compile-time error (CS0165).
- Static variables are shared across all instances; modifying them from multiple threads requires synchronization.
- Instance variables exist only after an object is constructed.
- The specification does not guarantee stack vs. heap placement for locals; this is an implementation detail.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: The Eight Categories in One Program

```csharp
using System;

class A
{
    public static int x;          // static variable
    int y;                        // instance variable

    void F(int[] v, int a, ref int b, out int c, in int d)
    {
        int i = 1;                // local variable
        c = a + b++ + d;          // out parameter must be assigned
        Console.WriteLine($"x={x}, y={y}, v[0]={v[0]}, a={a}, b={b}, c={c}, d={d}, i={i}");
    }

    static void Main()
    {
        var obj = new A();
        int[] arr = { 10 };
        int val = 5;
        int reference = 7;
        int output;
        obj.F(arr, val, ref reference, out output, 3);
    }
}
```

**Line-by-line comments**

- `public static int x;` — Static field: one storage location for the entire type `A`; default value `0`.
- `int y;` — Instance field: one copy per `A` object; default value `0`.
- `int[] v` — Value parameter holding a reference to an array; `v[0]` is an array element variable.
- `int a` — Value parameter: a copy of the caller’s value.
- `ref int b` — Reference parameter: aliases the caller’s variable; `b++` modifies the caller’s `reference`.
- `out int c` — Output parameter: must be assigned before the method returns.
- `in int d` — Input reference parameter: passed by reference for efficiency but treated as read-only.
- `int i = 1;` — Local variable: scope is the method body; must be initialized before reading.

**Expected Output**

```
x=0, y=0, v[0]=10, a=5, b=8, c=15, d=3, i=1
```

**Why this output occurs**

- `x` and `y` have default values `0` because they are fields.
- `v[0]` is `10` from the array initializer.
- `a` is `5` (value parameter copy).
- `b` becomes `8` because `b++` uses the original `7` and then increments the caller’s variable to `8`.
- `c` is computed as `a + b++ + d` = `5 + 7 + 3` = `15` (the `b++` expression yields the original `7`).
- `d` is `3`.
- `i` is `1`.

---

#### Example 2: Definite Assignment for Locals vs. Fields

```csharp
using System;

class Program
{
    static int staticField;          // initially assigned: default 0
    int instanceField;               // initially assigned: default 0

    static void Main()
    {
        Console.WriteLine(staticField);       // OK
        var p = new Program();
        Console.WriteLine(p.instanceField);   // OK

        int local;
        // Console.WriteLine(local);         // Compile-time error CS0165
        local = 42;
        Console.WriteLine(local);             // OK

        string s;
        if (DateTime.Now.Year > 2000)
            s = "modern";
        // Console.WriteLine(s);              // Error: not definitely assigned
        else
            s = "legacy";
        Console.WriteLine(s);                 // OK: assigned on both paths
    }
}
```

**Expected Output (assuming current year > 2000)**

```
0
0
42
modern
```

**Why this output occurs**

- Static and instance fields are initially assigned to their default values, so they can be read without explicit initialization.
- The local `local` is initially unassigned; the compiler rejects reading it before the assignment `local = 42`.
- The local `s` is assigned on both branches of the `if`/`else`, so the compiler proves it is definitely assigned at the `Console.WriteLine(s)` point.

---

### Real-World Cases

- **Game development** — Player health is an instance field (per player object); the maximum number of players might be a `static` field; the player’s score during a turn is a local variable.
- **Enterprise applications** — Configuration values are often `static readonly` fields; per-request data is stored in local variables; database connection strings may be `const` when they are compile-time literals.
- **Library design** — Public API fields are generally avoided; properties wrap instance variables, while `const` and `static readonly` expose constants safely.

---

### References

- Variables — C# Language Specification - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/language-specification/variables
- Declaration statements — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/statements/declarations
- Basic concepts — C# Language Specification - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/language-specification/basic-concepts

---

## 2. Variable Declaration

### Definitions

**Core Definition**
Variable declaration is the act of introducing a new variable into a declaration space by specifying its type and name.

**Technical Definition**
A declaration statement declares a new local variable, local constant, or local reference variable. The declaration specifies the variable’s type and identifier, and may optionally include an initializer. For fields, the declaration appears in a class or struct body and may include modifiers such as `static`, `readonly`, or `const`.

**Beginner-Friendly Explanation**
Declaration is “creating the labeled box.” You tell the compiler, “I want a box named `age` that can hold whole numbers.” Declaration does not necessarily put anything in the box; that is initialization or assignment.

---

### Purposes

- To reserve a named storage location of a specific type.
- To make the variable visible within its scope and declaration space.
- To enable the compiler to enforce type safety for all subsequent uses.
- To introduce fields that become part of an object’s state or the type’s state.
- To distinguish variables from constants and reference variables through the declaration form.

---

### Syntax Rules and Structure

#### General Syntax

```
// Local variable declaration
type   identifier   [, identifier2 ...]   [ = initializer ]   ;

// Local constant declaration
const   type   identifier   =   constantExpression   ;

// Field declaration
[modifiers]   type   identifier   [ = initializer ]   [, identifier2 [ = initializer2 ] ...]   ;

// Implicitly typed local variable declaration
var   identifier   =   expression   ;

// Ref local declaration
ref   type   identifier   =   ref   variable   ;
```

**Component breakdown**

- `type` — The compile-time type; required for explicitly typed declarations.
- `identifier` — The variable name; must follow C# identifier rules.
- `= initializer` — Optional for locals and fields; mandatory for `const`.
- `modifiers` — Field modifiers: `public`, `private`, `static`, `readonly`, `const`, etc.
- `var` — Replaces the explicit type when the compiler can infer it from the initializer.
- `ref` — Declares a ref local that aliases another variable.

#### Syntax Rules

- A single declaration statement can declare multiple variables of the same type: `int a, b, c;`.
- Local constants must be initialized in the declaration.
- Fields are implicitly initialized to default values if no initializer is supplied.
- The declared type must be accessible at the point of declaration.
- Identifier names must not collide with names in enclosing declaration spaces.

#### Constraints and Limitations

- You cannot declare a local variable without a name.
- `var` cannot be used for fields (except in specific preview features); it is limited to locals.
- A declaration cannot introduce the same name twice in the same declaration space.
- `const` declarations are restricted to compile-time constant types and values.
- Ref locals require the initializer to be a `ref` expression and the lifetimes must be compatible.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Multiple Declarations and Explicit Initialization

```csharp
using System;

class Program
{
    static void Main()
    {
        string greeting;                  // declaration only
        int a, b, c;                     // multiple variables, same type
        List<double> xs;                 // declaration of a generic list

        greeting = "Hello";              // assignment later
        a = 3; b = 2; c = a + b;        // separate assignment
        xs = new List<double>();         // assignment later

        Console.WriteLine($"{greeting}, a={a}, b={b}, c={c}, xs.Count={xs.Count}");
    }
}
```

**Expected Output**

```
Hello, a=3, b=2, c=5, xs.Count=0
```

**Why this output occurs**

- Declarations without initializers create variables in an unassigned state (for locals) or default state (for fields).
- The later assignments supply values before the variables are read.
- `xs.Count` is `0` because the list is empty.

---

#### Example 2: Declaration with Initializers and Constants

```csharp
using System;

class Program
{
    const string Greeting = "Hello";          // local constant, must initialize
    const double MinLimit = -10.0, MaxLimit = -MinLimit;  // multiple constants

    static void Main()
    {
        string message = "World";
        int x = 3, y = 2, z = x + y;
        List<double> values = new();

        Console.WriteLine($"{Greeting}, {message}! z={z}, count={values.Count}");
        Console.WriteLine($"MinLimit={MinLimit}, MaxLimit={MaxLimit}");
    }
}
```

**Expected Output**

```
Hello, World! z=5, count=0
MinLimit=-10, MaxLimit=10
```

**Why this output occurs**

- Local constants are initialized in the declaration and cannot be changed.
- `z` is computed from the initializers of `x` and `y`.
- `values` is initialized with a new empty list.
- `MaxLimit` uses the compile-time constant expression `-MinLimit`.

---

### Real-World Cases

- **Configuration classes** — Declaring `static readonly` fields for application-wide settings.
- **Data models** — Declaring instance fields for entity properties, often with initializers for default values.
- **Algorithms** — Declaring local variables for intermediate results, often without initializers until the correct value is computed.

---

### References

- Declaration statements — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/statements/declarations
- Variables — C# Language Specification - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/language-specification/variables
- Fields — C# Language Specification - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/language-specification/classes

---

## 3. Variable Initialization

### Definitions

**Core Definition**
Initialization is the process of giving a variable its first value.

**Technical Definition**
Initialization occurs when a variable is declared with an initializer (`type id = expr;`), when a field receives its default value automatically, or when an object is created and its fields are set to default values before the constructor runs. For locals, initialization is not automatic; a local with an initializer is considered definitely assigned only after the initializer executes.

**Beginner-Friendly Explanation**
Initialization is “putting something into the box for the first time.” For fields, the runtime puts a default value in the box automatically. For local variables, you must put something in yourself before you can take anything out.

---

### Purposes

- To ensure a variable has a well-defined value before it is read.
- To avoid compile-time errors related to unassigned locals.
- To set meaningful starting values for fields and objects.
- To leverage compile-time constant evaluation for `const` and `static readonly` fields.
- To support definite-assignment analysis by establishing assignment on all control-flow paths.

---

### Syntax Rules and Structure

#### General Syntax

```
// Local initialization
type   identifier   =   expression   ;

// Field initialization
[modifiers]   type   identifier   =   expression   ;

// Constant initialization (mandatory)
const   type   identifier   =   constantExpression   ;

// Array initialization
type[]   identifier   =   new   type[]   {   element1, element2, ...   }   ;

// Object initializer (related concept)
var   obj   =   new   Type   {   Property1 = value1, Property2 = value2   }   ;
```

**Component breakdown**

- `= expression` — The initializer; its type must be implicitly convertible to the variable’s type.
- `constantExpression` — An expression fully evaluable at compile time; required for `const`.
- `new type[] { ... }` — Array creation expression that initializes elements.

#### Syntax Rules

- Fields without explicit initializers receive default values: `0` for numeric types, `false` for `bool`, `null` for reference types, `'\0'` for `char`.
- Locals without initializers are initially unassigned; reading them before assignment is a compile-time error.
- `const` fields and locals must be initialized in the declaration.
- `readonly` fields may be initialized in the declaration or in a constructor.
- Static fields are initialized before the static constructor runs or before the first access to the type.

#### Constraints and Limitations

- A local variable’s initializer cannot refer to the variable itself (`var v = v++;` is an error).
- `const` initializers must be compile-time constant expressions; `new` is not permitted for non-string reference types in `const`.
- `var` declarations require an initializer from which the type can be inferred.
- Array initializers cannot be used directly with `var` without an explicit array creation expression in certain forms.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Default Values of Fields vs. Unassigned Locals

```csharp
using System;

class Program
{
    static int staticInt;            // default 0
    static bool staticBool;          // default false
    static string staticString;      // default null
    static char staticChar;          // default '\0'

    static void Main()
    {
        Console.WriteLine($"staticInt={staticInt}");
        Console.WriteLine($"staticBool={staticBool}");
        Console.WriteLine($"staticString is null: {staticString == null}");
        Console.WriteLine($"staticChar code={(int)staticChar}");

        int localInt;
        // Console.WriteLine(localInt); // CS0165: use of unassigned local
    }
}
```

**Expected Output**

```
staticInt=0
staticBool=False
staticString is null: True
staticChar code=0
```

**Why this output occurs**

- Static fields are automatically initialized to their default values before first use.
- `staticString` is `null`, so the equality check returns `True`.
- `staticChar` defaults to `'\0'`, whose numeric code is `0`.
- The local `localInt` remains unassigned and cannot be read.

---

#### Example 2: Initialization and Definite Assignment with Branches

```csharp
using System;

class Program
{
    static void Main()
    {
        int result;
        bool condition = DateTime.Now.Hour < 12;

        if (condition)
        {
            result = 1;
        }
        else
        {
            result = 2;
        }

        Console.WriteLine(result); // definitely assigned on both paths
    }
}
```

**Expected Output**

```
1
```

(assuming the program runs before noon; otherwise `2`)

**Why this output occurs**

- The compiler analyzes both branches and proves that `result` is assigned on every path that reaches the `Console.WriteLine`.
- Without the `else` assignment, the compiler would report CS0165.

---

### Real-World Cases

- **Configuration objects** — Initializing `static readonly` fields from environment variables or configuration files in a static constructor.
- **Data transfer objects** — Initializing collection properties to empty collections to avoid null-reference exceptions.
- **Numerical algorithms** — Initializing accumulators to zero and iterators to starting values.

---

### References

- Declaration statements — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/statements/declarations
- Variables — C# Language Specification - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/language-specification/variables
- Definite assignment — C# Language Specification - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/language-specification/variables#94-definite-assignment

---

## 4. Variable Assignment

### Definitions

**Core Definition**
Assignment is the operation of storing a new value into an existing variable.

**Technical Definition**
Assignment uses the assignment operator `=` to replace the current value of a variable with the value of an expression. The expression’s type must be implicitly convertible to the variable’s type. Compound assignments (`+=`, `-=`, etc.) combine a binary operation with assignment. The `++` and `--` operators also modify a variable’s value.

**Beginner-Friendly Explanation**
Assignment is “changing what is inside the box.” After a box has been created and given a name, you can replace its contents with a new value of the same shape.

---

### Purposes

- To update the state of an object or the progress of a computation.
- To supply values to variables after declaration.
- To satisfy definite-assignment requirements before a variable is read.
- To implement counters, accumulators, and state machines.
- To pass data back to the caller through `ref` and `out` parameters.

---

### Syntax Rules and Structure

#### General Syntax

```
// Simple assignment
variable   =   expression   ;

// Compound assignment
variable   +=   expression   ;   // variable = variable + expression
variable   -=   expression   ;   // variable = variable - expression
variable   *=   expression   ;   // variable = variable * expression
variable   /=   expression   ;   // variable = variable / expression
variable   %=   expression   ;   // variable = variable % expression
variable   &=   expression   ;   // logical/bitwise AND
variable   |=   expression   ;   // logical/bitwise OR
variable   ^=   expression   ;   // logical/bitwise XOR
variable   <<=  expression   ;   // left shift
variable   >>=  expression   ;   // right shift

// Increment/decrement
variable++   ;   // post-increment
++variable   ;   // pre-increment
variable--   ;   // post-decrement
--variable   ;   // pre-decrement
```

**Component breakdown**

- `variable` — An assignable expression: a local, field, array element, or parameter (except `in` parameters).
- `expression` — Any expression whose type is implicitly convertible to the variable’s type.
- Compound assignment operators — Perform the binary operation and then assign the result back to the variable.

#### Syntax Rules

- The left operand of an assignment must be a variable (or a property/indexer with a setter).
- The right operand must be implicitly convertible to the type of the left operand.
- Compound assignments are not allowed on `readonly` fields outside the constructor, nor on `const`.
- `++` and `--` are not allowed on `const` or `readonly` fields outside constructors.
- Assignment to `out` parameters is required before the method returns.

#### Constraints and Limitations

- You cannot assign to a `readonly` field after the constructor completes.
- You cannot assign to a `const` after declaration.
- You cannot assign to an `in` parameter inside the method (unless the compiler can prove no mutation in specific contexts).
- Assignment to a local variable does not automatically make it “definitely assigned” if the assignment occurs on only some control-flow paths.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Simple and Compound Assignment

```csharp
using System;

class Program
{
    static void Main()
    {
        int x = 10;
        x = 20;                     // simple assignment
        Console.WriteLine(x);       // 20

        x += 5;                    // compound: x = x + 5
        Console.WriteLine(x);      // 25

        x *= 2;                    // compound: x = x * 2
        Console.WriteLine(x);      // 50

        x++;                       // increment
        Console.WriteLine(x);      // 51

        --x;                       // pre-decrement
        Console.WriteLine(x);      // 50
    }
}
```

**Expected Output**

```
20
25
50
51
50
```

**Why this output occurs**

- Each assignment replaces the previous value.
- Compound assignments combine an arithmetic operation with assignment.
- `x++` returns the old value but increments the variable; because it is used as a statement here, only the side effect matters.

---

#### Example 2: Assignment and Definite Assignment with Loops

```csharp
using System;

class Program
{
    static void Main()
    {
        int sum;
        sum = 0;                    // assignment satisfies definite assignment

        for (int i = 1; i <= 5; i++)
        {
            sum += i;               // compound assignment in a loop
        }

        Console.WriteLine($"sum={sum}");
    }
}
```

**Expected Output**

```
sum=15
```

**Why this output occurs**

- `sum` is assigned `0` before the loop, so it is definitely assigned.
- Each iteration adds `i` to `sum`.
- The final value is `1 + 2 + 3 + 4 + 5 = 15`.

---

### Real-World Cases

- **Financial systems** — Accumulating transaction totals using compound assignment.
- **Game loops** — Updating player position, health, and score each frame.
- **Data processing** — Incrementing counters and aggregating values in loops.

---

### References

- Assignment operators — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/assignment-operator
- Variables — C# Language Specification - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/language-specification/variables
- Statements — C# Language Specification - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/language-specification/statements

---

## 5. `var` Keyword

### Definitions

**Core Definition**
The `var` keyword instructs the compiler to infer the type of a local variable from its initialization expression.

**Technical Definition**
`var` is an implicitly typed local variable declaration. The compiler determines the compile-time type from the initializer, and the variable is strongly typed exactly as if the type had been written explicitly. `var` is not a type itself; it is a syntactic convenience limited to local variables (and, in newer versions, some other contexts).

**Beginner-Friendly Explanation**
`var` means “you figure out the type, compiler.” You still get a strongly typed box, but you do not have to write the type name if it is obvious from the right-hand side.

---

### Purposes

- To reduce verbosity when the type is obvious from the initializer.
- To avoid repeating long generic type names.
- To make code easier to read when the right-hand side clearly communicates the type.
- To support anonymous types, which have no nameable type in source code.
- To keep code resilient to certain refactorings where the exact type name changes but the initializer remains valid.

---

### Syntax Rules and Structure

#### General Syntax

```
var   identifier   =   expression   ;
```

**Component breakdown**

- `var` — Contextual keyword indicating implicit typing.
- `identifier` — The variable name.
- `= expression` — Mandatory initializer; its compile-time type becomes the variable’s type.

#### Syntax Rules

- `var` can only be used for local variables (and `using` statements, `foreach` iteration variables, and some pattern contexts).
- The initializer is mandatory; `var x;` is an error.
- The initializer cannot be `null` because `null` has no type.
- The initializer cannot be a lambda expression or method group without a target type.
- The initializer cannot refer to the variable itself.
- `var` declarations are strongly typed; the inferred type is fixed at compile time.

#### Constraints and Limitations

- `var` cannot be used for fields, properties, or method return types (in standard C#).
- `var` cannot be used when the initializer is an array initializer without an explicit array creation expression: `var x = {1, 2};` is invalid.
- `var` with `null` is invalid because no type can be inferred.
- `var` with an anonymous function that has no natural type is invalid unless a target type is supplied.
- `var` is not the same as `dynamic`; the type is resolved at compile time, not runtime.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Basic Type Inference

```csharp
using System;

class Program
{
    static void Main()
    {
        var greeting = "Hello";                       // inferred as string
        var number = 32;                              // inferred as int
        var xs = new List<double>();                  // inferred as List<double>

        Console.WriteLine(greeting.GetType());        // System.String
        Console.WriteLine(number.GetType());          // System.Int32
        Console.WriteLine(xs.GetType());              // System.Collections.Generic.List`1[System.Double]
    }
}
```

**Expected Output**

```
System.String
System.Int32
System.Collections.Generic.List`1[System.Double]
```

**Why this output occurs**

- The compiler infers `string`, `int`, and `List<double>` from the initializers.
- `GetType()` returns the runtime type, which matches the inferred compile-time type.

---

#### Example 2: `var` with Complex Generic Types

```csharp
using System;
using System.Collections.Generic;

class Program
{
    static void Main()
    {
        var orders = new Dictionary<int, List<string>>();
        orders[1] = new List<string> { "Apple", "Banana" };

        var firstOrder = orders[1];
        var count = firstOrder.Count;

        Console.WriteLine($"count={count}");
        Console.WriteLine($"firstOrder type: {firstOrder.GetType()}");
    }
}
```

**Expected Output**

```
count=2
firstOrder type: System.Collections.Generic.List`1[System.String]
```

**Why this output occurs**

- `orders` is inferred as `Dictionary<int, List<string>>`.
- `firstOrder` is inferred as `List<string>`.
- `count` is inferred as `int`.
- The output reflects the actual runtime types.

---

### Real-World Cases

- **LINQ queries** — `var` is commonly used for query results whose exact generic type is complex and not useful to write explicitly.
- **Anonymous types** — `var` is required to hold the result of an anonymous type projection.
- **Factory patterns** — When a factory method returns a concrete type that may change, `var` reduces the impact on local code.

---

### References

- Implicitly typed local variables — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/statements/declarations#implicitly-typed-local-variables
- Declarations — C# Language Specification - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/language-specification/statements
- var (C# reference) — historical documentation - https://github.com/dotnet/docs/blob/main/docs/csharp/language-reference/keywords/var.md

---

## 6. Implicit Typing

### Definitions

**Core Definition**
Implicit typing is the compiler’s ability to determine a variable’s type from context, most commonly from an initialization expression, without an explicit type declaration.

**Technical Definition**
Implicitly typed local variables are declared with `var`. The compiler performs type inference using the initialization expression and assigns a compile-time type. The variable is strongly typed; implicit typing does not mean dynamic typing. Implicit typing is also used in other contexts such as lambda expressions, object initializers, and collection expressions.

**Beginner-Friendly Explanation**
Implicit typing means “the compiler guesses the type from what you put in the box.” It is still a strongly typed box; the compiler just saves you from writing the type name when it is obvious.

---

### Purposes

- To reduce repetitive type annotations in code.
- To make code more readable when the initializer clearly conveys the type.
- To enable the use of anonymous types.
- To simplify the use of complex generic types.
- To keep code focused on intent rather than type plumbing.

---

### Syntax Rules and Structure

#### General Syntax

```
var   identifier   =   expression   ;
```

**Component breakdown**

- `var` — The contextual keyword that triggers implicit typing.
- `identifier` — The variable name.
- `expression` — The initializer whose type is inferred.

#### Syntax Rules

- The initializer is mandatory and must have a determinable type.
- The inferred type is fixed at compile time; subsequent assignments must be implicitly convertible to that type.
- Implicit typing does not apply to fields, properties, or method return types in standard C#.
- `var` cannot be used with `null`, method groups, or anonymous functions lacking a natural type.
- The inferred type must be accessible at the declaration site.

#### Constraints and Limitations

- Implicit typing is limited to local variables and a few other contexts; it is not a replacement for explicit typing in public APIs.
- Overuse of `var` can reduce readability when the initializer’s type is not obvious.
- `var` cannot be used for `const` declarations (constants require explicit types).
- `var` is not `dynamic`; implicit typing is compile-time, while `dynamic` is runtime-bound.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Readability and Strong Typing

```csharp
using System;

class Program
{
    static void Main()
    {
        var count = 10;                 // int
        var price = 19.99m;             // decimal
        var name = "Widget";           // string
        var isAvailable = true;         // bool

        // count = "not a number";      // Compile error: cannot convert string to int

        Console.WriteLine($"{name} costs {price}, available: {isAvailable}, count: {count}");
    }
}
```

**Expected Output**

```
Widget costs 19.99, available: True, count: 10
```

**Why this output occurs**

- The compiler infers `int`, `decimal`, `string`, and `bool` from the literals.
- The commented assignment would fail because the inferred type of `count` is `int`, not `string`.

---

#### Example 2: Implicit Typing in LINQ

```csharp
using System;
using System.Linq;

class Program
{
    static void Main()
    {
        int[] numbers = { 1, 2, 3, 4, 5 };
        var evens = numbers.Where(n => n % 2 == 0);
        var squares = evens.Select(n => n * n);

        foreach (var n in squares)
        {
            Console.WriteLine(n);
        }
    }
}
```

**Expected Output**

```
4
16
```

**Why this output occurs**

- `evens` is inferred as `IEnumerable<int>` (or a related sequence type).
- `squares` is also inferred as `IEnumerable<int>`.
- The query filters even numbers (`2, 4`) and squares them (`4, 16`).

---

### Real-World Cases

- **LINQ and functional pipelines** — Implicit typing keeps query code readable when intermediate types are complex.
- **Anonymous types** — `var` is required to hold the result of anonymous object projections.
- **Dependency injection and factories** — When the concrete type is an implementation detail, `var` avoids coupling local code to a specific type name.

---

### References

- Implicitly typed local variables — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/statements/declarations#implicitly-typed-local-variables
- Declarations — C# Language Specification - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/language-specification/statements
- var keyword — historical documentation - https://github.com/dotnet/docs/blob/main/docs/csharp/language-reference/keywords/var.md

---

## 7. `const`

### Definitions

**Core Definition**
The `const` keyword declares a constant field or local constant whose value is fixed at compile time and cannot be modified.

**Technical Definition**
A `const` declaration introduces a symbolic name for a compile-time constant. The value is substituted directly into consuming code at compile time. `const` fields are implicitly static, and the modifier `static` cannot be combined with `const`. Permitted types include numeric types, `bool`, `char`, `string`, enum types, and reference types where the value is `null` (or `string` literals).

**Beginner-Friendly Explanation**
`const` creates a box whose contents are carved in stone at the moment the program is compiled. You cannot change the contents later, and the compiler literally copies the value into every place that uses the constant.

---

### Purposes

- To give a meaningful name to a value that never changes.
- To allow compile-time evaluation and substitution of constants.
- To enable constants to be used in contexts requiring compile-time values, such as `case` labels, attributes, and enum declarations.
- To improve performance by avoiding runtime field access.
- To enforce immutability by contract.

---

### Syntax Rules and Structure

#### General Syntax

```
// Constant field
[accessModifiers]   const   type   identifier   =   constantExpression   ;

// Local constant
const   type   identifier   =   constantExpression   ;

// Multiple constants in one declaration
const   type   identifier1   =   expr1   ,   identifier2   =   expr2   ;
```

**Component breakdown**

- `const` — Modifier indicating a compile-time constant.
- `type` — Must be one of the permitted constant types.
- `identifier` — The constant name.
- `= constantExpression` — An expression fully evaluable at compile time.

#### Syntax Rules

- `const` must be initialized in the declaration.
- `const` members are implicitly static; you cannot explicitly write `static const`.
- The initializer must be a constant expression, not a runtime expression.
- Constants can depend on other constants as long as there is no circular dependency.
- `const` can be used for local constants and field constants.
- `const` fields can be `public`, `private`, `protected`, `internal`, etc.

#### Constraints and Limitations

- `const` cannot be used with `static` explicitly.
- `const` cannot be used with `readonly` (they are mutually exclusive modifiers).
- The value must be computable at compile time; `new` is not permitted for non-string reference types.
- `const` values are substituted at compile time, so changing a `const` in a library requires recompiling all consuming code to observe the change.
- `const` cannot be used for types outside the permitted set (e.g., `DateTime`, arbitrary classes).

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Constant Fields and Local Constants

```csharp
using System;

class Program
{
    const int X = 5;
    public const double GravitationalConstant = 6.673e-11;
    private const string ProductName = "Visual C#";

    static void Main()
    {
        const string Language = "C#";
        const string Platform = ".NET";
        const string FullProductName = $"{Platform} - Language: {Language}";

        Console.WriteLine($"X={X}");
        Console.WriteLine($"GravitationalConstant={GravitationalConstant}");
        Console.WriteLine($"ProductName={ProductName}");
        Console.WriteLine($"FullProductName={FullProductName}");
    }
}
```

**Expected Output**

```
X=5
GravitationalConstant=6.673E-11
ProductName=Visual C#
FullProductName=.NET - Language: C#
```

**Why this output occurs**

- All constants are compile-time values substituted into the `Console.WriteLine` calls.
- Interpolated strings can be constant if all expressions are constant strings.

---

#### Example 2: Constants in Compile-Time Contexts

```csharp
using System;

class Program
{
    const int MaxRetries = 3;

    static void Main()
    {
        int attempt = 1;

        switch (attempt)
        {
            case MaxRetries:
                Console.WriteLine("Max retries reached");
                break;
            default:
                Console.WriteLine($"Attempt {attempt} of {MaxRetries}");
                break;
        }
    }
}
```

**Expected Output**

```
Attempt 1 of 3
```

**Why this output occurs**

- `MaxRetries` is a compile-time constant, so it can be used as a `case` label.
- The `default` branch executes because `attempt` is `1`, not `3`.

---

### Real-World Cases

- **Mathematical and scientific constants** — Pi approximations, physical constants, conversion factors.
- **Protocol and format constants** — Maximum buffer sizes, header sizes, magic numbers.
- **Business rules** — Maximum allowed items, default timeouts, fixed error codes.

---

### References

- const keyword — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/const
- Classes — C# Language Specification (constant declarations) - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/language-specification/classes
- readonly keyword — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/readonly

---

## 8. `readonly`

### Definitions

**Core Definition**
The `readonly` keyword indicates that a field can only be assigned during its declaration or in a constructor of the same class.

**Technical Definition**
`readonly` is a modifier used in five contexts: field declarations, `readonly struct` declarations, `readonly` instance members, `ref readonly` returns, and `ref readonly` parameters. For fields, assignment is permitted only in the declaration or in a constructor of the same class. Unlike `const`, the value may be computed at runtime, and different instances may have different values.

**Beginner-Friendly Explanation**
`readonly` means “you can write on this label only once, either when you create it or while the object is being built.” After the constructor finishes, the value is locked in. Unlike `const`, the value does not have to be known at compile time.

---

### Purposes

- To create immutable fields whose values are fixed after construction.
- To allow runtime-computed constants that cannot be `const`.
- To prevent accidental modification of critical fields.
- To support immutable value types and `readonly struct`.
- To improve thread-safety by guaranteeing that a field does not change after construction.

---

### Syntax Rules and Structure

#### General Syntax

```
// Readonly field
[accessModifiers]   readonly   type   identifier   [ = initializer ]   ;

// Readonly struct
readonly   struct   StructName   { ... }

// Readonly instance member
readonly   returnType   MethodName()   { ... }

// ref readonly return
ref readonly   type   MethodName()   { ... }

// ref readonly parameter
void   Method(ref readonly   type   parameterName)   { ... }
```

**Component breakdown**

- `readonly` — Modifier restricting assignment after construction.
- `type` — The field’s type; may be a value type or reference type.
- `= initializer` — Optional initializer; may also be assigned in a constructor.
- `readonly struct` — Declares a struct whose instance fields are all readonly.
- `ref readonly` — Returns or accepts a reference that cannot be written through.

#### Syntax Rules

- A `readonly` field can be assigned multiple times within the declaration and constructor, but not after the constructor completes.
- For reference types, `readonly` prevents reassigning the reference but does not make the referenced object immutable.
- For value types, `readonly` makes the field’s value immutable.
- `readonly` cannot be combined with `const` or `static` in the same field declaration (though `static readonly` is valid).
- `readonly` fields may be `static` or instance.

#### Constraints and Limitations

- A `readonly` field cannot be assigned in ordinary methods after construction.
- A `readonly` reference-type field does not prevent mutation of the object’s data.
- A `readonly` field of a mutable reference type can be a security concern if the reference is exposed publicly.
- `readonly` is not a substitute for `const` when compile-time evaluation is required.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Readonly Field Initialized in Constructor

```csharp
using System;

class Configuration
{
    public readonly string Environment;
    public readonly int MaxConnections;

    public Configuration(string environment, int maxConnections)
    {
        Environment = environment;
        MaxConnections = maxConnections;
    }

    public void TryChange()
    {
        // Environment = "Production"; // Compile error: readonly field
    }
}

class Program
{
    static void Main()
    {
        var config = new Configuration("Development", 10);
        Console.WriteLine($"{config.Environment}, max={config.MaxConnections}");
    }
}
```

**Expected Output**

```
Development, max=10
```

**Why this output occurs**

- The constructor assigns the `readonly` fields.
- After construction, the fields cannot be reassigned, as shown by the commented-out line.

---

#### Example 2: `static readonly` vs. `const`

```csharp
using System;

class MathConstants
{
    public const double PiConst = 3.14159;                       // compile-time
    public static readonly double PiReadonly = Math.PI;          // runtime

    static void Main()
    {
        Console.WriteLine($"PiConst={PiConst}");
        Console.WriteLine($"PiReadonly={PiReadonly}");
    }
}
```

**Expected Output**

```
PiConst=3.14159
PiReadonly=3.141592653589793
```

**Why this output occurs**

- `PiConst` is a compile-time constant substituted directly.
- `PiReadonly` is computed at runtime from `Math.PI` and stored in a static readonly field.

---

### Real-World Cases

- **Immutable configuration objects** — Connection strings, API endpoints, and feature flags set once at startup.
- **Dependency injection** — `readonly` fields for injected services, preventing accidental replacement.
- **Value objects** — `readonly struct` for types like `DateTime`, `Guid`, and custom immutable value objects.

---

### References

- readonly keyword — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/readonly
- Classes — C# Language Specification (fields and readonly) - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/language-specification/classes
- Structure types — readonly struct - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/struct#readonly-struct

---

## 9. `static`

### Definitions

**Core Definition**
The `static` modifier declares a member that belongs to the type itself rather than to a specific instance.

**Technical Definition**
A static field identifies exactly one storage location for the type, shared across all instances. Static members are accessed through the type name, not through an object reference. A static class cannot be instantiated and can contain only static members. Static constructors run at some point between program start and the first access to the type.

**Beginner-Friendly Explanation**
`static` means “there is one shared box for the whole class, not one box per object.” Every object of the class sees the same contents. You use the class name to get to the box, not an object name.

---

### Purposes

- To share data across all instances of a type.
- To create utility methods and constants that do not depend on object state.
- To implement counters and caches at the type level.
- To provide a single storage location for type-wide configuration.
- To support static classes for grouping related functionality.

---

### Syntax Rules and Structure

#### General Syntax

```
// Static field
[accessModifiers]   static   type   identifier   [ = initializer ]   ;

// Static method
[accessModifiers]   static   returnType   MethodName(parameters)   { ... }

// Static class
static   class   ClassName   { ... }

// Static constructor
static   ClassName()   { ... }

// Static local function
static   returnType   LocalFunction(parameters)   { ... }
```

**Component breakdown**

- `static` — Modifier indicating type-level membership.
- `type` — The field’s type.
- `identifier` — The static member name.
- `static class` — A class that cannot be instantiated and contains only static members.
- `static constructor` — A constructor that runs once, automatically, to initialize static members.

#### Syntax Rules

- Static fields are initialized to default values before the static constructor runs.
- Static members are accessed via `TypeName.MemberName`, not `instance.MemberName`.
- A static class cannot have instance members.
- A static constructor cannot have access modifiers or parameters.
- Static local functions cannot capture local variables or instance state.

#### Constraints and Limitations

- `static` cannot be used with indexers or finalizers.
- A static class cannot be instantiated or inherited from.
- Static fields are shared state; concurrent access requires synchronization.
- `static` cannot be combined with `const` (const is implicitly static).
- `static readonly` is valid and commonly used for runtime-initialized constants.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Shared Static Counter

```csharp
using System;

class Employee
{
    public static int EmployeeCount;    // shared across all instances
    public string Name;

    public Employee(string name)
    {
        Name = name;
        EmployeeCount++;                // increments the shared field
    }
}

class Program
{
    static void Main()
    {
        var e1 = new Employee("Alice");
        var e2 = new Employee("Bob");

        Console.WriteLine(Employee.EmployeeCount);   // 2
        Console.WriteLine(e1.Name);                 // Alice
        Console.WriteLine(e2.Name);                 // Bob
    }
}
```

**Expected Output**

```
2
Alice
Bob
```

**Why this output occurs**

- `EmployeeCount` is static and shared by all `Employee` instances.
- Each constructor increments the same storage location.
- `Name` is an instance field, so each object has its own copy.

---

#### Example 2: Static Class and Static Constructor

```csharp
using System;

static class AppSettings
{
    public static readonly string AppName;
    public static readonly int MaxUsers;

    static AppSettings()
    {
        AppName = "MyApplication";
        MaxUsers = 100;
        Console.WriteLine("Static constructor ran");
    }

    public static void PrintInfo()
    {
        Console.WriteLine($"{AppName}, max users={MaxUsers}");
    }
}

class Program
{
    static void Main()
    {
        AppSettings.PrintInfo();
        AppSettings.PrintInfo();
    }
}
```

**Expected Output**

```
Static constructor ran
MyApplication, max users=100
MyApplication, max users=100
```

**Why this output occurs**

- The static constructor runs exactly once, before the first access to `AppSettings`.
- Subsequent calls to `PrintInfo` reuse the initialized static fields.

---

### Real-World Cases

- **Utility classes** — `Math`, `Console`, `File` are static classes grouping related methods.
- **Counters and metrics** — Static counters for instances created, requests processed, or errors logged.
- **Configuration** — `static readonly` fields for application-wide settings.
- **Caches** — Static dictionaries for memoization or shared lookup tables.

---

### References

- static modifier — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/static
- Classes — C# Language Specification (static fields) - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/language-specification/classes
- Static classes and static class members - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/static-classes-and-static-class-members

---

## 10. `dynamic`

### Definitions

**Core Definition**
The `dynamic` type enables operations to bypass compile-time type checking and be resolved at runtime.

**Technical Definition**
`dynamic` is a static type in the C# type system, but an object of type `dynamic` bypasses static type checking. Operations on `dynamic` expressions are bound at runtime by the Dynamic Language Runtime (DLR). `dynamic` is similar to `object`, but operations on `object` are bound at compile time. You cannot use `new dynamic()`, and you cannot derive from `dynamic`.

**Beginner-Friendly Explanation**
`dynamic` means “I will not tell the compiler what type this is; figure it out while the program is running.” It is flexible but shifts type errors from compile time to runtime.

---

### Purposes

- To interoperate with dynamic languages and COM APIs.
- To simplify code that would otherwise require extensive reflection.
- To defer method resolution and member access until runtime.
- To handle scenarios where the type is not known at compile time.
- To reduce boilerplate in certain metaprogramming and scripting scenarios.

---

### Syntax Rules and Structure

#### General Syntax

```
dynamic   identifier   =   expression   ;

// As a return type
dynamic   MethodName()   { ... }

// As a parameter type
void   MethodName(dynamic   parameter)   { ... }

// As a field or property type
dynamic   FieldName;
```

**Component breakdown**

- `dynamic` — The type keyword enabling runtime binding.
- `identifier` — The variable name.
- `expression` — Any expression; the result is bound at runtime.

#### Syntax Rules

- `dynamic` can be used as a local variable type, field type, parameter type, return type, and generic type argument.
- Operations on `dynamic` are resolved at runtime, not compile time.
- `dynamic` is a static type, but it bypasses static checking for operations.
- You cannot use `new dynamic()`.
- You cannot derive a type from `dynamic`.

#### Constraints and Limitations

- Runtime binding can fail with `RuntimeBinderException` if the member does not exist.
- Performance overhead due to runtime binding and caching.
- Loss of compile-time safety: errors that would be caught by the compiler become runtime errors.
- Limited IntelliSense support in IDEs.
- `dynamic` is not allowed in certain contexts, such as `const` declarations.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Dynamic Method Invocation

```csharp
using System;

class Program
{
    static void Main()
    {
        dynamic d = "Hello";
        Console.WriteLine(d.Length);          // runtime binding to string.Length

        d = 42;
        Console.WriteLine(d + 1);             // runtime binding to int addition

        d = new int[] { 1, 2, 3 };
        Console.WriteLine(d[1]);              // runtime array indexing
    }
}
```

**Expected Output**

```
5
43
2
```

**Why this output occurs**

- The runtime binder resolves `Length` on a string, `+` on an integer, and indexing on an array.
- The same `dynamic` variable holds different types at different times.

---

#### Example 2: Dynamic vs. Static Typing

```csharp
using System;

class Program
{
    static void Main()
    {
        object o = "Hello";
        // Console.WriteLine(o.Length);       // Compile error: object has no Length

        dynamic d = "Hello";
        Console.WriteLine(d.Length);          // OK: runtime resolution

        d = 10;
        try
        {
            Console.WriteLine(d.Length);      // RuntimeBinderException
        }
        catch (Microsoft.CSharp.RuntimeBinder.RuntimeBinderException ex)
        {
            Console.WriteLine("Runtime binding failed: " + ex.Message);
        }
    }
}
```

**Expected Output**

```
5
Runtime binding failed: 'int' does not contain a definition for 'Length'
```

**Why this output occurs**

- `object` requires a cast to `string` before accessing `Length`.
- `dynamic` resolves `Length` at runtime; when `d` holds an `int`, the member does not exist and a `RuntimeBinderException` is thrown.

---

### Real-World Cases

- **COM interop** — Simplifying calls to Office and other COM APIs.
- **Dynamic languages** — Interoperating with IronPython, IronRuby, and JavaScript via Jint.
- **Reflection-heavy code** — Reducing boilerplate when invoking members by name.
- **Rapid prototyping** — When the exact type is intentionally fluid during development.

---

### References

- dynamic type — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/built-in-types
- Using type dynamic — C# Programming Guide - https://github.com/dotnet/docs/blob/main/docs/csharp/programming-guide/types/using-type-dynamic.md
- dynamic (C# reference) — historical documentation - https://github.com/dotnet/docs/blob/main/docs/csharp/language-reference/keywords/dynamic.md

---

## 11. Variable Scope

### Definitions

**Core Definition**
Scope is the region of program text in which a variable’s name can be used to refer to that variable.

**Technical Definition**
Every method declaration, property declaration, accessor, indexer, operator, instance constructor, anonymous function, and local function creates a local variable declaration space. Names are introduced into this space through parameters and local declarations. The scope of a local variable declared in a local-variable-declaration is the block in which the declaration occurs. It is an error to refer to a local variable before its declaration, even though the variable is in scope throughout the block.

**Beginner-Friendly Explanation**
Scope is “where the name of the box is visible.” A local variable is visible from the point of declaration to the end of the enclosing block. You cannot use the name before its declaration, even though the box technically exists from the start of the block.

---

### Purposes

- To control where a variable can be accessed, preventing accidental misuse.
- To limit the lifetime of variables to the region where they are needed.
- To enable name reuse in non-overlapping scopes.
- To support structured programming and information hiding.
- To help the compiler enforce definite assignment and detect errors.

---

### Syntax Rules and Structure

#### General Syntax

Scope is determined by the placement of declarations within blocks and declaration spaces:

```
{
    // outer block
    int x = 1;          // x is in scope from here to the end of this block

    {
        // nested block
        int y = 2;      // y is in scope only within this nested block
        Console.WriteLine(x + y);   // both x and y accessible
    }

    // y is no longer in scope
    Console.WriteLine(x);           // x still accessible
}
```

**Component breakdown**

- Block `{ ... }` — Creates a nested local variable declaration space.
- Declaration order — A local variable is in scope throughout the block but cannot be referred to before its declaration.
- Nested spaces — Inner scopes can see outer variables unless hidden by a same-named local.

#### Syntax Rules

- The scope of a local variable declared in a block is the entire block.
- It is an error to refer to a local variable before its declaration, even within the same block.
- A nested declaration space cannot declare a variable with the same name as a variable in an enclosing declaration space.
- Parameters are in scope throughout the method body.
- Variables declared in a `for` initializer are in scope for the entire `for` statement.
- Variables declared in a `switch` section may have surprising scope; direct declarations in a `switch` section belong to the enclosing `switch` block.

#### Constraints and Limitations

- You cannot access a local variable outside its scope.
- You cannot declare two locals with the same name in the same declaration space.
- Hiding an outer variable with a nested local is allowed only if the names differ according to the rules (actually, same-name hiding is an error in C# for locals).
- Loop variables have per-iteration scope, but the exact behaviour depends on the loop construct.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Block Scope

```csharp
using System;

class Program
{
    static void Main()
    {
        int outer = 10;
        {
            int inner = 20;
            Console.WriteLine(outer + inner);   // 30
        }
        // Console.WriteLine(inner);            // Compile error: inner not in scope
        Console.WriteLine(outer);                // 10
    }
}
```

**Expected Output**

```
30
10
```

**Why this output occurs**

- `inner` is declared in a nested block and is only visible within that block.
- `outer` is visible in both the outer and nested blocks.

---

#### Example 2: Declaration Order and Definite Assignment

```csharp
using System;

class Program
{
    static void Main()
    {
        // Console.WriteLine(x);    // Error: x is in scope but not definitely assigned
        int x = 5;
        Console.WriteLine(x);       // OK
    }
}
```

**Expected Output**

```
5
```

**Why this output occurs**

- `x` is in scope throughout the block, but the compiler rejects any use before the declaration.
- After the declaration and initialization, `x` is definitely assigned and readable.

---

### Real-World Cases

- **Method-local variables** — Keeping temporary data from leaking into other methods.
- **Loop bodies** — Ensuring loop counters and accumulators are scoped to the loop.
- **Conditional blocks** — Restricting variables to the branch where they are meaningful.
- **Lambda expressions** — Capturing variables from an enclosing scope, which affects lifetime.

---

### References

- Basic concepts — C# Language Specification (declaration spaces and scopes) - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/language-specification/basic-concepts
- Variables — C# Language Specification - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/language-specification/variables
- Local variables — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/statements/declarations

---

## 12. Variable Lifetime

### Definitions

**Core Definition**
Lifetime is the period during program execution during which a variable’s storage is guaranteed to be allocated.

**Technical Definition**
The lifetime of a local variable is the portion of program execution during which its storage is guaranteed to be allocated. It begins when control enters the scope to which the variable is associated and ends at least when execution of that scope ends. Captured variables in anonymous functions may have extended lifetimes, keeping the variable alive as long as the delegate or expression tree is reachable. The actual lifetime may be shorter than the scope if the compiler can prove the variable is no longer used.

**Beginner-Friendly Explanation**
Lifetime is “how long the box exists.” A local variable exists while its method or block is running. A field exists as long as its object exists. A static field exists for the entire life of the application. Once a local method finishes, its local boxes are gone (unless they are captured by a lambda).

---

### Purposes

- To determine how long storage is reserved for a variable.
- To understand when captured variables remain alive.
- To reason about memory usage and garbage collection.
- To avoid dangling references and unintended retention.
- To design efficient methods that do not hold onto data longer than necessary.

---

### Syntax Rules and Structure

Lifetime is determined by the variable category and the constructs that capture variables:

```
// Local variable lifetime: method/block scope
void M()
{
    int local = 1;      // lifetime: until M returns (or block exits)
}

// Captured variable lifetime: extended to delegate lifetime
Func<int> CreateCounter()
{
    int count = 0;      // captured variable
    return () => ++count;  // count remains alive as long as the delegate
}

// Static variable lifetime: application domain
static int staticCounter;   // alive until the application domain unloads

// Instance variable lifetime: object lifetime
class C
{
    int instanceField;      // alive as long as the C object is reachable
}
```

**Component breakdown**

- Local variable — Lifetime tied to the method or block.
- Captured variable — Lifetime extended to the delegate or expression tree that captures it.
- Static variable — Lifetime tied to the application domain.
- Instance variable — Lifetime tied to the object instance.

#### Syntax Rules

- A local variable’s lifetime begins when control enters its scope and ends at least when the scope ends.
- If a local is captured by an anonymous function, its lifetime is extended until the delegate and all objects referencing the captured variable are eligible for garbage collection.
- Each iteration of a `foreach` statement creates a fresh iteration variable with its own lifetime.
- The actual lifetime may be shorter than the scope; the compiler may optimize storage.

#### Constraints and Limitations

- You cannot explicitly control the lifetime of a local variable; it is determined by scope and capture.
- Extending lifetime through capture can cause memory retention if the delegate is kept alive longer than intended.
- The specification does not guarantee stack vs. heap; lifetime guarantees are semantic, not physical.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Local Variable Lifetime

```csharp
using System;

class Program
{
    static void CreateAndPrint()
    {
        int local = 42;
        Console.WriteLine(local);
    }   // local's lifetime ends here

    static void Main()
    {
        CreateAndPrint();
        CreateAndPrint();
    }
}
```

**Expected Output**

```
42
42
```

**Why this output occurs**

- Each call to `CreateAndPrint` creates a new `local` with its own lifetime.
- When the method returns, the local is no longer alive.

---

#### Example 2: Captured Variable Lifetime

```csharp
using System;

class Program
{
    static Func<int> CreateCounter()
    {
        int count = 0;
        return () => ++count;    // count is captured and lives with the delegate
    }

    static void Main()
    {
        var counter = CreateCounter();
        Console.WriteLine(counter());   // 1
        Console.WriteLine(counter());   // 2
    }
}
```

**Expected Output**

```
1
2
```

**Why this output occurs**

- `count` is captured by the lambda; its lifetime extends as long as the `counter` delegate is reachable.
- Each call increments the same captured variable.

---

### Real-World Cases

- **Event handlers and callbacks** — Captured variables remain alive as long as the event subscription exists.
- **Caching and memoization** — Static fields and long-lived objects extend variable lifetime intentionally.
- **Resource management** — Understanding when objects become eligible for garbage collection.
- **Iterators and async methods** — Compiler-generated state machines extend the lifetime of locals across `yield` and `await` boundaries.

---

### References

- Variables — C# Language Specification (lifetime) - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/language-specification/variables
- Basic concepts — C# Language Specification - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/language-specification/basic-concepts
- Captured variables — C# Language Specification - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/language-specification/variables

---

## 13. Variable Naming

### Definitions

**Core Definition**
Variable naming is the set of rules and conventions used to choose identifiers for variables.

**Technical Definition**
An identifier is the name assigned to a type, member, variable, or namespace. Valid identifiers must start with a letter or underscore, can contain Unicode letter characters, decimal digits, connecting characters, combining characters, and formatting characters. Identifiers that match keywords require the `@` prefix. Naming conventions are not enforced by the compiler but are widely followed for consistency.

**Beginner-Friendly Explanation**
Naming is “choosing the label for the box.” There are hard rules the compiler enforces (start with a letter or underscore, no spaces), and soft conventions that make code readable (use meaningful names, follow casing conventions).

---

### Purposes

- To make code readable and self-documenting.
- To follow community and team conventions for consistency.
- To avoid compiler errors from invalid identifiers.
- To communicate the variable’s role and type through its name.
- To support tooling such as IntelliSense and static analysis.

---

### Syntax Rules and Structure

#### General Syntax

```
identifier   =   letterOrUnderscore   {   letterOrDigitOrUnderscore   }
```

**Component breakdown**

- First character — Must be a letter (Unicode `Lu`, `Ll`, `Lt`, `Lm`, `Nl`) or underscore (`_`).
- Subsequent characters — May include letters, decimal digits, connecting characters, combining characters, and formatting characters.
- `@` prefix — Allows keywords to be used as identifiers (e.g., `@if`).

#### Naming Conventions

- **PascalCase** — Type names, method names, namespaces, and public members.
- **camelCase** — Local variables, method parameters, and private/internal non-constant fields.
- **_camelCase** — Private and internal non-constant instance fields start with an underscore.
- **PascalCase for constants** — Constant names, both fields and locals, use PascalCase.
- **Meaningful names** — Prefer descriptive names over abbreviations; avoid single-letter names except for simple loop counters.

#### Constraints and Limitations

- Identifiers cannot contain spaces or most special characters.
- Identifiers cannot start with a digit.
- Two consecutive underscores are reserved for compiler-generated identifiers.
- Naming conventions are not enforced by the compiler; teams may adopt different conventions.
- Unicode categories outside the permitted set are replaced with `_` in some tooling contexts.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Valid and Invalid Identifiers

```csharp
using System;

class Program
{
    static void Main()
    {
        int age = 30;                 // valid: starts with letter
        int _count = 5;               // valid: starts with underscore
        int @class = 10;              // valid: verbatim identifier for keyword
        // int 2fast = 1;             // invalid: starts with digit
        // int my-var = 2;            // invalid: hyphen not allowed

        Console.WriteLine($"{age}, {_count}, {@class}");
    }
}
```

**Expected Output**

```
30, 5, 10
```

**Why this output occurs**

- `age`, `_count`, and `@class` satisfy identifier rules.
- The commented declarations violate the rules and would cause compiler errors.

---

#### Example 2: Naming Conventions in Practice

```csharp
using System;

class Product
{
    private int _stockCount;          // _camelCase for private field
    public string Name { get; set; }  // PascalCase for public property

    public void UpdateStock(int newCount)   // camelCase for parameter
    {
        _stockCount = newCount;
    }

    public void PrintStock()
    {
        Console.WriteLine(_stockCount);
    }
}

class Program
{
    static void Main()
    {
        var product = new Product { Name = "Widget" };
        product.UpdateStock(15);
        product.PrintStock();
    }
}
```

**Expected Output**

```
15
```

**Why this output occurs**

- The naming conventions are followed: `_stockCount` for a private field, `Name` for a public property, `newCount` for a parameter, and `product` for a local variable.
- The output reflects the assigned stock count.

---

### Real-World Cases

- **Team codebases** — Consistent naming conventions reduce cognitive load and merge conflicts.
- **Public APIs** — PascalCase for public members aligns with .NET Framework Design Guidelines.
- **Code reviews** — Naming conventions are a common review criterion for maintainability.

---

### References

- Identifier names — rules and conventions - https://learn.microsoft.com/en-us/dotnet/csharp/fundamentals/coding-style/identifier-names
- C# Coding Conventions - https://learn.microsoft.com/en-us/dotnet/csharp/fundamentals/coding-style/coding-conventions
- C# Language Specification — Identifiers - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/language-specification/lexical-structure#643-identifiers

---

## 14. Variable Best Practices

### Definitions

**Core Definition**
Variable best practices are the recommended guidelines for declaring, initializing, naming, and using variables to produce clear, safe, and maintainable code.

**Technical Definition**
Best practices encompass language-level rules (definite assignment, scope, lifetime) and community conventions (naming, `var` usage, immutability, avoiding shared mutable state). They are not enforced by the compiler but are supported by analyzers, style guidelines, and the .NET ecosystem’s coding standards.

**Beginner-Friendly Explanation**
Best practices are “the habits that make your code easy to read, hard to break, and easy to change.” They include things like giving variables meaningful names, declaring them close to where they are used, preferring immutability, and avoiding global mutable state.

---

### Purposes

- To improve code readability and maintainability.
- To reduce bugs caused by uninitialized variables or shared mutable state.
- To make intent clear through naming and placement.
- To align with team and ecosystem conventions.
- To leverage compiler analysis and tooling effectively.

---

### Guidelines and Structure

#### General Guidelines

```
// Prefer meaningful names
int customerCount;          // good
int n;                     // poor (unless a loop counter)

// Declare close to first use
int result = Compute();
Console.WriteLine(result);

// Prefer initialization at declaration when possible
int total = 0;

// Use var when the type is obvious
var names = new List<string>();

// Prefer readonly for fields that should not change
private readonly ILogger _logger;

// Avoid shared mutable static state
// Prefer const/static readonly for constants
```

**Component breakdown**

- **Meaningful names** — Descriptive nouns or noun phrases; avoid abbreviations.
- **Close declaration** — Declare variables as close as possible to their first use.
- **Initialization** — Initialize locals at declaration whenever practical.
- **`var` usage** — Use `var` when the type is clear from the right-hand side; avoid when it obscures intent.
- **Immutability** — Prefer `readonly` and `const` for values that should not change.
- **Minimize scope** — Keep variables in the smallest scope necessary.
- **Avoid shared mutable static state** — Use dependency injection or instance state instead.

#### Rules and Constraints

- Always initialize locals before reading them (compiler-enforced).
- Prefer `readonly` fields over mutable fields for injected dependencies.
- Avoid `public` fields; use properties instead.
- Avoid `dynamic` unless runtime binding is genuinely required.
- Do not use `var` for `const` (constants require explicit types).
- Follow team naming conventions consistently.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Good vs. Poor Variable Usage

```csharp
using System;
using System.Collections.Generic;

class OrderProcessor
{
    private readonly ILogger _logger;          // readonly injected dependency
    private readonly List<Order> _orders = new();  // readonly collection reference

    public OrderProcessor(ILogger logger)
    {
        _logger = logger;
    }

    public void Process(Order order)
    {
        var validationResult = Validate(order);   // var when type is clear
        if (!validationResult.IsValid)
        {
            _logger.Log(validationResult.Message);
            return;
        }

        _orders.Add(order);
        _logger.Log($"Processed order {order.Id}");
    }

    private ValidationResult Validate(Order order)
    {
        // ...
        return new ValidationResult(true, "");
    }
}
```

**Expected Output (conceptual)**

```
Processed order 123
```

**Why this is good practice**

- `_logger` and `_orders` are `readonly`, preventing accidental replacement.
- `var` is used when the type is obvious from `Validate(order)`.
- The variable `validationResult` is declared close to its use.
- The method avoids shared mutable static state.

---

#### Example 2: Avoiding Common Pitfalls

```csharp
using System;

class Program
{
    static void Main()
    {
        // Pitfall: uninitialized local
        int x;
        // Console.WriteLine(x);    // CS0165

        // Good: initialize at declaration
        int y = 0;
        Console.WriteLine(y);

        // Pitfall: overusing var when type is unclear
        // var data = GetData();    // What type is data?
        // Good: explicit type when inference is not obvious
        // List<string> data = GetData();

        // Pitfall: mutable static state
        // Good: const or static readonly for constants
        const int MaxRetries = 3;
        Console.WriteLine(MaxRetries);
    }
}
```

**Expected Output**

```
0
3
```

**Why this is good practice**

- The uninitialized local is avoided by initializing `y`.
- `var` is used judiciously; explicit types are preferred when inference would be unclear.
- Constants are used instead of mutable static fields for fixed values.

---

### Real-World Cases

- **Enterprise codebases** — Consistent best practices reduce onboarding time and defect rates.
- **Open-source libraries** — Public APIs follow naming and immutability guidelines for predictability.
- **Code reviews and analyzers** — Tools like Roslyn analyzers enforce many best practices automatically.

---

### References

- C# identifier naming rules and conventions - https://learn.microsoft.com/en-us/dotnet/csharp/fundamentals/coding-style/identifier-names
- C# Coding Conventions - https://learn.microsoft.com/en-us/dotnet/csharp/fundamentals/coding-style/coding-conventions
- .NET Runtime coding style - https://github.com/dotnet/runtime/blob/main/docs/coding-guidelines/coding-style.md
- Framework Design Guidelines — Names of type members - https://learn.microsoft.com/en-us/dotnet/standard/design-guidelines/names-of-type-members

---

## Deprecated, Unsafe, Implementation-Dependent, and Version-Specific Notes

| Feature | Status / Note |
|---------|---------------|
| `dynamic` | Not deprecated, but runtime binding introduces performance overhead and runtime errors; use only when necessary. |
| `scoped` ref locals | Version-specific (C# 11+); `scoped` limits the lifetime of `ref` variables. |
| `readonly struct` | Introduced in C# 7.2; version-specific. |
| `static local functions` | Introduced in C# 8.0; version-specific. |
| `in` parameters | Introduced in C# 7.2; version-specific. |
| Stack vs. heap placement | Implementation-dependent; the specification guarantees lifetime, not physical storage. |
| `const` substitution | Changing a `const` in a library requires recompiling consuming code to observe the change. |
| Public readonly mutable reference fields | Potential security vulnerability (CA2104). |
| Shared static mutable fields | Not thread-safe without synchronization; shared state is a common source of bugs. |

---

## References (Consolidated)

- Variables — C# Language Specification - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/language-specification/variables
- Declaration statements — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/statements/declarations
- Basic concepts — C# Language Specification - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/language-specification/basic-concepts
- const keyword — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/const
- readonly keyword — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/readonly
- static modifier — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/static
- dynamic type — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/built-in-types
- Using type dynamic — C# Programming Guide - https://github.com/dotnet/docs/blob/main/docs/csharp/programming-guide/types/using-type-dynamic.md
- Identifier names — rules and conventions - https://learn.microsoft.com/en-us/dotnet/csharp/fundamentals/coding-style/identifier-names
- C# Coding Conventions - https://learn.microsoft.com/en-us/dotnet/csharp/fundamentals/coding-style/coding-conventions
- .NET Runtime coding style - https://github.com/dotnet/runtime/blob/main/docs/coding-guidelines/coding-style.md
- Framework Design Guidelines — Names of type members - https://learn.microsoft.com/en-us/dotnet/standard/design-guidelines/names-of-type-members
- Structure types — readonly struct - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/struct#readonly-struct
- Classes — C# Language Specification (fields, constants, static fields) - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/language-specification/classes
- Definite assignment — C# Language Specification - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/language-specification/variables#94-definite-assignment
- Implicitly typed local variables — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/statements/declarations#implicitly-typed-local-variables
- var (C# reference) — historical documentation - https://github.com/dotnet/docs/blob/main/docs/csharp/language-reference/keywords/var.md
- readonly (C# reference) — historical documentation - https://github.com/dotnet/docs/blob/main/docs/csharp/language-reference/keywords/readonly.md
- static (C# reference) — historical documentation - https://github.com/dotnet/docs/blob/main/docs/csharp/language-reference/keywords/static.md
- dynamic (C# reference) — historical documentation - https://github.com/dotnet/docs/blob/main/docs/csharp/language-reference/keywords/dynamic.md

---