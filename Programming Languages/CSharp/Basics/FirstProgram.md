# First C# Program

Writing your first C# program introduces the fundamental building blocks of the language: program structure, entry points, namespaces, using directives, statements, expressions, and console input/output. This guide walks through each concept, from the simplest possible program to a complete annotated example, and explains how C# code is compiled and executed.

---

## 1. Program Structure

A C# program is built from several core building blocks:

> "You build C# programs from these core building blocks: namespaces organize your types, types (classes, structs, interfaces, enums, and delegates) define behavior and data, and statements and expressions perform work at run time."

### General Structure

```
┌─────────────────────────────────────┐
│  using directives                   │
├─────────────────────────────────────┤
│  namespace declaration              │
├─────────────────────────────────────┤
│  types (classes, structs, etc.)     │
├─────────────────────────────────────┤
│  entry point (top-level or Main)    │
├─────────────────────────────────────┤
│  statements and expressions         │
└─────────────────────────────────────┘
```

### File-Based vs. Project-Based Apps

C# supports two structural styles:

| Style | Description | When to Use |
|---|---|---|
| **File-based app** | A single `.cs` file with no project file | Small utilities, prototypes, experiments |
| **Project-based app** | Uses a `.csproj` file; can span multiple files | Multi-file programs, fine-grained build configuration |

> "A file-based app runs from a single `.cs` file with no project file. A project-based app uses a `.csproj` file and can span multiple source files."

Starting with **C# 14 and .NET 10**, file-based apps let you run a program contained in a single `.cs` file without a project file:

```csharp
#!/usr/bin/env dotnet
Console.WriteLine("Hello, World!");
```

> "File-based apps support all C# syntax... Use file-based apps for small command-line utilities, prototypes, and experiments."

---

## 2. Top-Level Statements

**Top-level statements** (introduced in C# 9) allow you to write executable code directly at the top of a file, without wrapping it in a class and `Main` method.

> "By using top-level statements, you can write executable code directly at the root of a file."

### Simplest C# Program

```csharp
Console.WriteLine("Hello, World!");
```

This single line is a **complete C# program**.

### Comparison: Traditional vs. Top-Level

**Traditional (explicit Main method):**

```csharp
using System;

namespace Application
{
    class Program
    {
        static void Main(string[] args)
        {
            Console.WriteLine("Hello World!");
        }
    }
}
```

> "Those 11 lines contain only one line of executable code."

**Top-level statements:**

```csharp
Console.WriteLine("Hello, World!");
```

> "You can simplify that program with the new top-level statements feature. That enables you to remove all but two of the lines in this program."

### Rules for Top-Level Statements

| Rule | Description |
|---|---|
| **Only one file** | Top-level statements can only be used in one source file per application |
| **Entry point is the first line** | The program's entry point is the first line of program text in that file |
| **Implicitly in global namespace** | Top-level statements are implicitly in the global namespace |
| **Using directives come first** | `using` directives must come before any other code |
| **Execution order** | Top-level statements execute in the order they appear in the file |

### What the Compiler Generates

When you use top-level statements, the compiler generates a hidden `Program` class with a synthesized `Main` method:

> "The C# templates for .NET 6 use top-level statements."

> "The compiler generates a method to serve as the program entry point for a project with top-level statements."

The generated `Main` signature depends on what your code contains:

| Top-Level Code Contains | Implicit Main Signature |
|---|---|
| `await` and `return` | `static async Task<int> Main(string[] args)` |
| `await` only | `static async Task Main(string[] args)` |
| `return` only | `static int Main(string[] args)` |
| Neither | `static void Main(string[] args)` |

> "The signature of the method depends on whether the top-level statements contain the await keyword or the return statement."

### Accessing Command-Line Arguments

Top-level statements provide an implicit `args` variable:

```csharp
// args is available implicitly
Console.WriteLine($"Argument count: {args.Length}");

foreach (var arg in args)
{
    Console.WriteLine(arg);
}
```

> "Top-level programs implicitly have an args variable containing the command-line arguments."

### Transitioning to Full Programs

> "Top-level statements do enable quick experimentation and beginner tutorials. They also provide a smooth path from experimentation to full programs."

As your program grows, you can refactor top-level statements into methods, classes, and other reusable components.

---

## 3. The `Main` Method

The **`Main` method** is the explicit entry point for a C# application. When your program starts, the runtime calls `Main` before any other code runs.

> "When you start a C# application, the runtime calls the Main method. The Main method is the entry point of a C# application. A C# program can have only one entry point."

### Declaration Rules

> "You declare Main with these rules: You must declare Main inside a class or struct. The enclosing class can be static. Main must be static. Main can have any access modifier. Main can return void, int, Task, or Task<int>. If and only if Main returns a Task or Task<int>, the declaration of Main can include the async modifier. You can declare the Main method with or without a string[] parameter that contains command-line arguments."

### Valid `Main` Signatures

```csharp
static void Main() { }
static int Main() { }
static void Main(string[] args) { }
static int Main(string[] args) { }
static async Task Main() { }
static async Task<int> Main() { }
static async Task Main(string[] args) { }
static async Task<int> Main(string[] args) { }
```

> "The preceding examples don't specify an access modifier, so they're implicitly private by default."

### Return Values

| Return Type | Meaning |
|---|---|
| `void` | No exit code returned |
| `int` | Exit code (0 = success, non-zero = failure) |
| `Task` | Async method with no exit code |
| `Task<int>` | Async method with exit code |

### Command-Line Arguments

> "Unlike C and C++, the name of the program isn't treated as the first command-line argument in the args array."

```csharp
class TestClass
{
    static void Main(string[] args)
    {
        Console.WriteLine($"Arguments: {args.Length}");
    }
}
```

### Multiple `Main` Methods

> "If you have more than one class with a Main method, you must use the StartupObject compiler option when you compile your program to specify which Main method serves as the entry point."

### `Main` vs. Top-Level Statements

> "If you have an existing application that uses an explicit Main method, there's no need to convert it. Both styles compile to equivalent code."

| Aspect | Top-Level Statements | `Main` Method |
|---|---|---|
| **Boilerplate** | Minimal | More ceremony required |
| **Best for** | New apps, small utilities, learning | Existing codebases, explicit structure |
| **Class required** | No (compiler generates one) | Yes |
| **Namespace** | Implicitly global | Explicit or implicit |
| **C# version** | C# 9+ | All versions |

---

## 4. Namespace Declarations

A **namespace** is an organized structure that groups related types together and prevents naming conflicts.

> "A namespace groups related types together and prevents name conflicts."

### Why Namespaces Exist

> "Namespaces in C# work similarly to packages in Java or modules in Python."

Every .NET type belongs to a namespace. For example:
- `Console` and `Math` belong to the `System` namespace (fully qualified: `System.Console`, `System.Math`)
- `List<T>` and `Dictionary<TKey, TValue>` belong to `System.Collections.Generic`

### Declaring a Namespace

**Traditional (block-scoped):**

```csharp
namespace MyApplication
{
    class Program
    {
        static void Main() { }
    }
}
```

**File-scoped (C# 10+):**

```csharp
namespace MyApplication;

class Program
{
    static void Main() { }
}
```

> "Beginning with C# 10, you can declare a namespace for all types defined in that file using a file-scoped namespace declaration. A file-scoped namespace declaration must precede all other members in the file, and it applies to the entire file."

### Nested Namespaces

```csharp
namespace Company.Project.Module
{
    class MyClass { }
}
```

### The Global Namespace

Types not declared in any namespace belong to the **global namespace**:

```csharp
class GlobalClass { }  // lives in the global namespace
```

> "The global namespace is the 'root' namespace."

---

## 5. `using` Directives

A **`using` directive** imports a namespace, allowing you to use its types by their simple names without the full namespace path.

> "A using directive lets your program consume those types by their simple names. You don't need to type the full namespace path every time you use one."

### Basic Usage

```csharp
using System;

class Program
{
    static void Main()
    {
        Console.WriteLine("Hello");  // System.Console is now just Console
    }
}
```

Without the `using` directive, you would need:

```csharp
System.Console.WriteLine("Hello");
```

### Types of `using` Directives

| Type | Syntax | Purpose |
|---|---|---|
| **Namespace import** | `using System;` | Import all types from a namespace |
| **Alias** | `using MyList = System.Collections.Generic.List<int>;` | Create an alias for a type |
| **Static import** | `using static System.Math;` | Import static members directly |
| **Global using** | `global using System;` | Apply to all files in the project |

### Placement Rules

> "For the single file containing top-level statements, using directives must come first in that file."

```csharp
using System;
using System.Collections.Generic;

// Then top-level statements
Console.WriteLine("Hello");
```

### Implicit `using` Directives (.NET 6+)

> "The .NET 6 SDK also adds a set of implicit global using directives for projects that use the following SDKs: Microsoft.NET.Sdk, Microsoft.NET.Sdk.Web, Microsoft.NET.Sdk.Worker. These implicit global using directives include the most common namespaces for the project type."

With **Implicit Usings** enabled in the `.csproj`, common namespaces like `System`, `System.Collections.Generic`, `System.Linq`, and `System.Threading.Tasks` are automatically imported.

---

## 6. Statements and Expressions

### Expressions

An **expression** is a combination of values, variables, operators, and method calls that evaluates to a single value.

> "An expression is a combination of values, variables, operators, and method calls that evaluates to a single value. Expressions produce a result, and can be used where a value is expected."

**Examples:**

```csharp
42                          // literal
x                           // variable
x + y                       // arithmetic
Math.Sqrt(16)               // method call
"Hello" + " World"          // string concatenation
```

### Statements

A **statement** is an instruction that performs an action. Every statement ends with a semicolon.

> "In C#, source code consists of a series of statements. A statement is an instruction to the compiler. For example, in C#, every statement ends with a semicolon."

**Examples:**

```csharp
int x = 5;                          // declaration statement
Console.WriteLine("Hello");         // expression statement
if (x > 0) { /* ... */ }            // control flow statement
return 0;                           // return statement
```

### Statements vs. Expressions

| Aspect | Expression | Statement |
|---|---|---|
| **Produces** | A value | An action |
| **Ends with** | No semicolon (when used as an expression) | Semicolon |
| **Example** | `x + y` | `int result = x + y;` |
| **Can be used inside** | Other expressions | Not inside expressions |

### Blocks

A **block** is a sequence of statements enclosed in `{ }`:

```csharp
{
    int x = 5;
    Console.WriteLine(x);
}
```

---

## 7. Compilation and Execution

C# uses a **two-stage compilation model**.

### Stage 1: C# → Intermediate Language (IL)

The C# compiler reads your source code and produces **Microsoft Intermediate Language (MSIL)** — also called **IL** or **CIL**.

> "The compiler reads your source code and produces MSIL."

> "The MSIL code is actually stored in an .exe file, but this file does not contain executable code. It contains the information needed by the JIT to execute the code when you run it."

### Stage 2: IL → Native Code (JIT)

When the program runs, the **CLR** (Common Language Runtime) uses the **Just-In-Time (JIT) compiler** to translate IL into native machine code.

> "When you run the program, the .NET Just In Time (JIT) compiler reads your MSIL code and produces an executable application in memory."

### The Full Pipeline

```
C# Source Code (.cs)
        │
        ▼
   C# Compiler (Roslyn)
        │
        ▼
Intermediate Language (IL) + Metadata
   stored in .exe or .dll
        │
        ▼
   CLR loads assembly
        │
        ▼
   JIT Compiler
        │
        ▼
   Native Machine Code
        │
        ▼
   Execution on CPU
```

### Build and Run Commands

> "In most C# programs, you use the dotnet build command to compile source files into a binary package. Then you can use the dotnet run command to run the program."

```bash
# Compile
dotnet build

# Run (compiles first if needed)
dotnet run

# Run a file-based app (C# 14+)
dotnet run hello-world.cs
```

> "You can simplify this because dotnet run compiles the program first if necessary, then runs it."

---

## 8. Console Input and Output

### Output: `Console.Write` and `Console.WriteLine`

| Method | Behavior |
|---|---|
| `Console.Write()` | Prints without a newline |
| `Console.WriteLine()` | Prints and moves to the next line |

> "The Console.WriteLine method prints to the console and moves the cursor to the next line."

**Examples:**

```csharp
Console.Write("Enter your name: ");     // no newline
Console.WriteLine("Hello, World!");     // with newline
Console.WriteLine($"Sum: {5 + 3}");     // string interpolation
```

### Input: `Console.ReadLine`

> "Console.ReadLine reads input from the standard input stream."

```csharp
Console.Write("Enter your name: ");
string? name = Console.ReadLine();      // returns string? (nullable)
Console.WriteLine($"Hello, {name}!");
```

> "It's best to use nullable types (with the ? symbol) for the input strings, since System.Console.ReadLine returns a nullable reference type."

### Input: `Console.Read` and `Console.ReadKey`

| Method | Returns | Description |
|---|---|---|
| `Console.Read()` | `int` | Reads a single character; returns -1 at end |
| `Console.ReadKey()` | `ConsoleKeyInfo` | Reads a key press without echoing |
| `Console.ReadLine()` | `string?` | Reads an entire line |

### Converting Input to Other Types

`Console.ReadLine()` always returns a `string`. Use `TryParse` for safe conversion:

```csharp
Console.Write("Enter a number: ");
string? input = Console.ReadLine();

if (double.TryParse(input, out double number))
{
    Console.WriteLine($"You entered: {number}");
}
else
{
    Console.WriteLine("Invalid input.");
}
```

> "Learn how to handle console input and output in C# by using Console.Write and Console.WriteLine for printing, capturing user input with Console.ReadLine, and converting string input to other data types safely with TryParse."

### Complete I/O Example

```csharp
using System;

class Program
{
    static void Main()
    {
        Console.Write("Enter your name: ");
        string? name = Console.ReadLine();

        Console.Write("Enter your age: ");
        string? ageInput = Console.ReadLine();

        if (int.TryParse(ageInput, out int age))
        {
            Console.WriteLine($"Hello, {name}! You are {age} years old.");
        }
        else
        {
            Console.WriteLine("Invalid age.");
        }
    }
}
```

---

## 9. Complete Annotated Example

### Modern C# (Top-Level Statements, .NET 6+)

```csharp
// ─── using directives ────────────────────────────────────
using System;                           // import System namespace

// ─── Top-level statements ────────────────────────────────
Console.WriteLine("Hello, World!");     // console output

Console.Write("Enter your name: ");     // prompt without newline
string? name = Console.ReadLine();      // read user input

Console.WriteLine($"Hello, {name}!");   // string interpolation
```

### Traditional C# (Explicit Main Method)

```csharp
// ─── using directive ─────────────────────────────────────
using System;

// ─── Namespace declaration ───────────────────────────────
namespace MyFirstApp
{
    // ─── Class declaration ───────────────────────────────
    class Program
    {
        // ─── Main method — entry point ───────────────────
        static void Main(string[] args)
        {
            // ─── Statements and expressions ──────────────
            Console.WriteLine("Hello, World!");

            Console.Write("Enter your name: ");
            string? name = Console.ReadLine();

            Console.WriteLine($"Hello, {name}!");
        }
    }
}
```

### Build and Run

```bash
# Create a new console project
dotnet new console -n MyFirstApp
cd MyFirstApp

# Run
dotnet run

# Output:
# Hello, World!
# Enter your name: Alice
# Hello, Alice!
```

---

## 10. Summary Table

| Concept | Definition | Example |
|---|---|---|
| **Program structure** | Namespaces, types, statements, expressions | `using`, `namespace`, `class`, `Main` |
| **Top-level statements** | Executable code at file root without `Main` | `Console.WriteLine("Hi");` |
| **`Main` method** | Explicit entry point | `static void Main(string[] args)` |
| **Namespace** | Named scope for types | `namespace MyApp;` |
| **`using` directive** | Imports a namespace | `using System;` |
| **Expression** | Produces a value | `x + y`, `Math.Sqrt(16)` |
| **Statement** | Performs an action; ends with `;` | `int x = 5;` |
| **Compilation** | C# → IL → (JIT) → native code | `dotnet build` |
| **Output** | `Console.Write`, `Console.WriteLine` | `Console.WriteLine("Hi");` |
| **Input** | `Console.ReadLine`, `Console.ReadKey` | `string? s = Console.ReadLine();` |

---

## Key Takeaways

1. **C# programs** are built from namespaces, types, statements, and expressions.
2. **Top-level statements** (C# 9+) eliminate the need for a class and `Main` method — you can write executable code directly at the top of a file.
3. **The `Main` method** is the explicit entry point; it must be `static` and inside a class or struct.
4. **Only one entry point** is allowed per program — either top-level statements or a single `Main` method.
5. **Namespaces** organize types and prevent naming conflicts; `using` directives import them for simpler access.
6. **Expressions** produce values; **statements** perform actions and end with semicolons.
7. **C# compiles to IL** (Intermediate Language), which is then JIT-compiled to native code at runtime by the CLR.
8. **`Console.WriteLine`** prints with a newline; **`Console.Write`** prints without one.
9. **`Console.ReadLine`** returns a nullable string; use `TryParse` for safe conversion to other types.
10. **Build and run** with `dotnet build` and `dotnet run`.

---

Would you like me to continue with the next topic — **C# Data Types**, **C# Variables and Constants**, or **C# Operators**?