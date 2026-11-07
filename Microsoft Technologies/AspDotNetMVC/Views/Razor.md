# Razor Syntax in ASP.NET Core MVC: A Comprehensive Cheat Sheet

> **Version Scope & Compatibility Note**
> This cheat sheet targets **ASP.NET Core MVC** (3.0 and later, including .NET 6/7/8/9/10/11). Razor syntax has been largely stable across these versions. The `@helper` directive is **legacy and not supported** in ASP.NET Core; it was removed when ASP.NET Core was introduced. The modern replacement is `@functions` (or local methods), and for Blazor components, `@code` is preferred over `@functions`. The `@model`, `@inject`, `@section`, and control-structure directives are stable. `@using` serves dual purposes: importing namespaces in a view and creating disposable HTML helper scopes. Files containing Razor typically have a `.cshtml` extension.

---

## Topic Overview

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Razor is a **markup syntax for embedding .NET-based C# code into webpages**, combining HTML markup with C# expressions and directives to produce dynamic HTML output. |
| **Technical Definition** | Razor is a templating engine that parses `.cshtml` files, recognizes transitions from HTML to C# via the `@` symbol, evaluates C# expressions, executes control-flow blocks, and renders the resulting HTML. It supports **implicit expressions** (`@Model.Name`), **explicit expressions** (`@(expression)`), **code blocks** (`@{ ... }`), **control structures** (`@if`, `@foreach`, etc.), and **directives** (`@model`, `@inject`, `@section`, `@functions`, etc.). The generated class derives from `RazorPage<TModel>` (for MVC views) or `RazorPage<dynamic>` when no `@model` is specified. |
| **Beginner-Friendly Explanation** | Razor is **HTML with C# superpowers**. You write normal HTML, and whenever you need dynamic content—like a product name or a list of items—you use `@` to switch into C#. Razor takes your mixed HTML and C# and turns it into a complete HTML page that the browser can display. It's designed to be clean and readable, so you don't have to write messy string concatenations to build HTML. |

### Key Characteristics

- **`@` transition character** – Signals the switch from HTML to C#.
- **Implicit and explicit expressions** – `@Model.Name` (implicit) vs. `@(DateTime.Now - TimeSpan.FromDays(7))` (explicit).
- **Code blocks** – `@{ ... }` for multi-statement C# logic.
- **Control structures** – `@if`, `@else`, `@foreach`, `@for`, `@while`, `@switch` for conditional and iterative rendering.
- **Directives** – `@model`, `@using`, `@inject`, `@functions`, `@section` for configuration and structure.
- **Automatic HTML encoding** – Razor encodes C# output by default, mitigating XSS.
- **`@@` escape** – Renders a literal `@` in the output.

### Prerequisites

- **.NET SDK** (6.0+ recommended; examples use .NET 8/10).
- **ASP.NET Core MVC** project with Razor views (`.cshtml`).
- Basic knowledge of **HTML** and **C#**.
- Familiarity with **controllers** and **models** (helpful but not required for syntax).

### Related Programming Areas

- **Views and View Discovery** – Razor files are the views rendered by MVC.
- **Tag Helpers** – Server-side attributes that complement Razor syntax.
- **Layouts and Sections** – `@section` works with layouts for content injection.
- **Dependency Injection** – `@inject` brings services into views.
- **Blazor Components** – Razor syntax is also used in `.razor` files, with `@code` replacing `@functions`.

### Core Concepts / Features

1. **Razor (General)** – The templating engine and its role.
2. **`@`** – The transition symbol.
3. **`@{ }`** – Code blocks.
4. **`@()`** – Explicit expressions.
5. **`@if` / `@else`** – Conditional rendering.
6. **`@foreach` / `@for` / `@while`** – Looping.
7. **`@switch`** – Switch-based rendering.
8. **`@using`** – Namespace imports and disposable scopes.
9. **`@model`** – Strongly typed model declaration.
10. **`@inject`** – Service injection.
11. **`@functions`** – Adding C# members to the generated class.
12. **`@section`** – Named content sections for layouts.
13. **`@helper` (Legacy)** – Removed directive and its replacement.
14. **Razor Best Practices** – Guidelines for clean, safe, and maintainable Razor code.

---

## 1. Razor (General)

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Razor is a markup syntax for embedding .NET-based code into webpages, consisting of Razor markup, C#, and HTML. |
| **Technical Definition** | Razor is a view engine that parses `.cshtml` files, generates a C# class deriving from `RazorPage<TModel>`, and executes it to produce HTML. It supports implicit expressions, explicit expressions, code blocks, control structures, and directives. The default language is HTML; `@` transitions to C#. |
| **Beginner-Friendly Explanation** | Razor is **the language of `.cshtml` files**. It lets you write HTML and sprinkle in C# where needed. The `@` symbol is the magic switch—anything after `@` is C# code that gets evaluated and inserted into the HTML. |

### Purposes (each begins with “To”)

- **To** embed server-side C# logic directly into HTML markup.
- **To** produce dynamic web pages that render data from models.
- **To** keep view code readable by minimizing string concatenation and code-behind files.
- **To** automatically HTML-encode output, reducing XSS risks.
- **To** support reusable UI through layouts, partials, and view components.

### Syntax Rules and Structure

#### General Syntax

```cshtml
@model MyApp.Models.Product
<h1>@Model.Name</h1>
@if (Model.Stock > 0) { <p>In stock</p> }
```

#### Syntax Rules

- **`@` transitions to C#**; `@@` escapes to a literal `@`.
- **HTML is rendered unchanged** unless Razor transitions are encountered.
- **Implicit expressions** (`@Model.Name`) must not contain spaces (except after `await`).
- **Explicit expressions** (`@( ... )`) allow spaces and complex expressions.
- **Generic method calls** must be wrapped in explicit expressions or code blocks.

#### Constraints and Limitations

- **No `@helper`** in ASP.NET Core; use `@functions` or local methods.
- **Email addresses** containing `@` are not treated as transitions.
- **`@` in attributes** works normally; Razor is context-aware.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Basic Razor View

```cshtml
@model MyApp.Models.Product

<h1>@Model.Name</h1>
<p>Price: @Model.Price.ToString("C")</p>
```

**Step-by-step explanation:**

1. `@model` declares the model type.
2. `@Model.Name` outputs the product name (HTML-encoded).
3. `@Model.Price.ToString("C")` formats the price as currency.

**Expected Output:** An HTML page with the product name and formatted price.

**Why this output?** Razor evaluates the C# expressions and inserts their encoded values into the HTML.

---

#### Example 2: Escaping `@`

```cshtml
<p>Follow us @@contoso</p>
```

**Expected Output:** `<p>Follow us @contoso</p>`

**Why this output?** `@@` escapes to a single `@` in the rendered HTML.

### Real-World Cases

- **Product detail pages** – displaying model data.
- **Blog posts** – rendering titles, content, and metadata.
- **Dashboards** – showing computed values.

### References

- Razor syntax reference for ASP.NET Core – https://learn.microsoft.com/en-us/aspnet/core/mvc/views/razor

---

## 2. `@` (Transition Symbol)

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | `@` is the character that signals a transition from HTML markup to C# code in Razor. |
| **Technical Definition** | When Razor encounters `@`, it evaluates the following C# expression or directive. If followed by a reserved keyword, it becomes a Razor directive; otherwise, it is an implicit expression. `@@` escapes to a literal `@`. |
| **Beginner-Friendly Explanation** | `@` is the **“switch to C#” button**. Write `@Model.Name` to output a value, or `@if (...) { ... }` to run conditional logic. |

### Purposes (each begins with “To”)

- **To** transition from HTML to C# within a view.
- **To** output the value of a C# expression directly into the HTML.
- **To** introduce Razor directives (`@model`, `@inject`, etc.) and control structures.
- **To** escape a literal `@` using `@@` when needed.

### Syntax Rules and Structure

#### General Syntax

```cshtml
@Model.Property
@DateTime.Now
@@literalAt
```

#### Syntax Rules

- **Implicit expressions** – `@` followed by a C# expression (no spaces except after `await`).
- **Explicit expressions** – `@( ... )` for complex or spaced expressions.
- **Directives** – `@model`, `@using`, `@inject`, etc.
- **Escape** – `@@` renders a single `@`.

#### Constraints and Limitations

- **Generic method calls** cannot be used directly after `@`; wrap them in `@( )` or a code block.
- **Email addresses** are not transitions because Razor contextually understands attribute values.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Implicit Expression

```cshtml
<p>@DateTime.Now</p>
```

**Expected Output:** `<p>3/15/2025 10:30:00 AM</p>` (current date/time).

**Why this output?** `@DateTime.Now` is an implicit expression evaluated by Razor.

---

#### Example 2: Escaping `@`

```cshtml
<p>Email: support@@contoso.com</p>
```

**Expected Output:** `<p>Email: support@contoso.com</p>`

**Why this output?** `@@` becomes a literal `@`.

### Real-World Cases

- **Dynamic timestamps** – `@DateTime.Now`.
- **User greetings** – `@User.Identity.Name`.
- **Literal `@` in content** – social handles or email addresses.

### References

- Razor syntax: `@` transitions – https://learn.microsoft.com/en-us/aspnet/core/mvc/views/razor#razor-syntax

---

## 3. `@{ }` (Code Block)

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | `@{ }` is a **Razor code block** that executes one or more C# statements without directly outputting their result. |
| **Technical Definition** | A code block starts with `@{` and ends with `}`. Inside, you can declare variables, run loops, compute values, and include markup via `@:` or `<text>`. Variables declared in a code block are in scope for the rest of the view. |
| **Beginner-Friendly Explanation** | `@{ }` is a **container for C# code**. Use it when you need multiple lines of C#—like declaring variables or preparing data—before displaying anything. |

### Purposes (each begins with “To”)

- **To** execute multiple C# statements within a view.
- **To** declare variables and compute values used later in the view.
- **To** prepare data for display without emitting output at the block’s location.
- **To** control the flow of Razor rendering with complex logic.

### Syntax Rules and Structure

#### General Syntax

```cshtml
@{
    var message = "Hello";
    var count = 3;
}
<p>@message (@count)</p>
```

#### Syntax Rules

- **Code blocks do not output their contents** unless you use `@:` or `<text>` to mix in markup.
- **Variables declared** are available in subsequent Razor expressions.
- **C# statements** inside must end with semicolons where required.

#### Constraints and Limitations

- **Markup inside a code block** requires `@:` (single line) or `<text>...</text>` (multi-line).
- **Overuse** makes views hard to read; prefer `@functions` for reusable logic.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Variable Declaration

```cshtml
@{
    var greeting = "Welcome";
    var year = DateTime.Now.Year;
}
<h1>@greeting to @year</h1>
```

**Expected Output:** `<h1>Welcome to 2025</h1>`

**Why this output?** Variables are declared and then used in implicit expressions.

---

#### Example 2: Markup Inside Code Block

```cshtml
@{
    var items = new[] { "A", "B" };
    foreach (var item in items)
    {
        <p>@item</p>
    }
}
```

**Expected Output:** `<p>A</p><p>B</p>`

**Why this output?** Markup inside a code block is rendered; Razor switches back to HTML when it encounters tags.

### Real-World Cases

- **Preparing view data** – computing totals, formatting strings.
- **Conditional variable assignment** – setting CSS classes based on model state.
- **Iterating with inline markup** – generating repeated HTML fragments.

### References

- Razor code blocks – https://learn.microsoft.com/en-us/aspnet/core/mvc/views/razor#razor-code-blocks

---

## 4. `@()` (Explicit Expression)

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | `@( ... )` is an **explicit Razor expression** that evaluates the C# expression inside the parentheses and renders the result. |
| **Technical Definition** | Explicit expressions are used when an implicit expression is ambiguous—for example, when the expression contains spaces, operators, or generic method calls. The parentheses define the exact extent of the expression. |
| **Beginner-Friendly Explanation** | `@()` is the **“do the math” wrapper**. Use it when you need to evaluate a complex expression, like `@(price * quantity)` or `@(DateTime.Now - TimeSpan.FromDays(7))`. |

### Purposes (each begins with “To”)

- **To** evaluate C# expressions that contain spaces, operators, or method calls.
- **To** disambiguate between Razor markup and C# code.
- **To** wrap generic method calls that would otherwise be parsed as HTML tags.
- **To** perform inline calculations and render the result.

### Syntax Rules and Structure

#### General Syntax

```cshtml
<p>@(DateTime.Now - TimeSpan.FromDays(7))</p>
<p>@(price * quantity)</p>
<p>@(GenericMethod<int>())</p>
```

#### Syntax Rules

- **Parentheses are required**; everything inside is evaluated as a single C# expression.
- **Result is HTML-encoded** by default.
- **Use for** arithmetic, method calls, ternaries, and generic calls.

#### Constraints and Limitations

- **Not needed for simple property access** (`@Model.Name` works).
- **Nested parentheses** are fine as long as they are balanced.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Date Arithmetic

```cshtml
<p>Last week: @(DateTime.Now - TimeSpan.FromDays(7))</p>
```

**Expected Output:** `<p>Last week: 3/8/2025 10:30:00 AM</p>`

**Why this output?** The expression inside `@()` is evaluated as a single C# expression.

---

#### Example 2: Generic Method Call

```cshtml
<p>@(GenericMethod<int>())</p>
```

**Expected Output:** The result of the method call.

**Why this output?** Wrapping in `@()` prevents Razor from interpreting `<int>` as an HTML tag.

### Real-World Cases

- **Price calculations** – `@(price * 1.1m)`.
- **Date math** – `@(DateTime.Now.AddDays(30))`.
- **Conditional expressions** – `@(isActive ? "Active" : "Inactive")`.

### References

- Explicit Razor expressions – https://learn.microsoft.com/en-us/aspnet/core/mvc/views/razor#explicit-razor-expressions

---

## 5. `@if` / `@else`

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | `@if` and `@else` are **conditional Razor statements** that render markup only when the specified condition is true. |
| **Technical Definition** | `@if (condition) { ... }` evaluates the C# condition; if true, the enclosed markup and code execute. `else if` and `else` are used without the `@` symbol because they are part of the same statement. |
| **Beginner-Friendly Explanation** | `@if` is **“show this only when…”**. Use it to display different content based on a condition—like showing “In Stock” only when `Model.Stock > 0`. |

### Purposes (each begins with “To”)

- **To** conditionally render HTML markup based on C# logic.
- **To** show or hide content depending on model values.
- **To** implement multi-branch logic with `else if` and `else`.
- **To** avoid rendering unnecessary HTML when conditions are not met.

### Syntax Rules and Structure

#### General Syntax

```cshtml
@if (condition)
{
    <p>True branch</p>
}
else if (otherCondition)
{
    <p>Else-if branch</p>
}
else
{
    <p>Else branch</p>
}
```

#### Syntax Rules

- **`@` is required only on the initial `if`**; `else if` and `else` do not need it.
- **Conditions are C# expressions** (booleans).
- **Markup inside braces** is rendered conditionally.

#### Constraints and Limitations

- **Braces are required** even for single statements (Razor enforces this).
- **Complex logic** should be extracted to `@functions` or the model to keep views readable.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Simple `if`

```cshtml
@if (Model.Stock > 0)
{
    <p>In stock</p>
}
```

**Expected Output:** `<p>In stock</p>` only if `Stock` is greater than 0.

**Why this output?** The condition is evaluated; the markup renders only when true.

---

#### Example 2: `if` / `else if` / `else`

```cshtml
@if (value % 2 == 0)
{
    <p>Even</p>
}
else if (value >= 1337)
{
    <p>Large</p>
}
else
{
    <p>Odd and small</p>
}
```

**Expected Output:** The branch matching the condition renders.

**Why this output?** Razor evaluates conditions in order and renders the first matching branch.

### Real-World Cases

- **Stock status** – “In Stock” / “Out of Stock”.
- **User roles** – show admin controls only for admins.
- **Validation messages** – display errors conditionally.

### References

- Conditionals (`@if`, `else`, `else if`) – https://learn.microsoft.com/en-us/aspnet/core/mvc/views/razor#conditionals-if-else-if-else-and-switch

---

## 6. `@foreach` / `@for` / `@while`

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | These are **looping Razor statements** that repeatedly render markup for each item in a collection or for a specified number of iterations. |
| **Technical Definition** | `@foreach (var item in collection) { ... }` iterates over `IEnumerable`. `@for (init; condition; iterator) { ... }` uses a standard C# `for` loop. `@while (condition) { ... }` repeats while the condition is true. `@do while` is also supported. |
| **Beginner-Friendly Explanation** | These loops are **“repeat this for each item”**. Use `@foreach` to list products, `@for` to repeat a block a specific number of times, or `@while` when you have a custom condition. |

### Purposes (each begins with “To”)

- **To** render a list of items from a collection.
- **To** repeat markup a specific number of times.
- **To** generate table rows, list items, or grid cells dynamically.
- **To** iterate with custom conditions using `while` or `do while`.

### Syntax Rules and Structure

#### General Syntax

```cshtml
@foreach (var item in Model.Items)
{
    <p>@item.Name</p>
}

@for (var i = 0; i < Model.Count; i++)
{
    <p>@Model[i]</p>
}

@while (condition)
{
    <p>...</p>
}
```

#### Syntax Rules

- **`@` is required on the loop keyword**.
- **Loop body can contain markup and C#**.
- **Variables declared in the loop** are scoped to the loop.

#### Constraints and Limitations

- **Avoid complex logic inside loops**; prepare data beforehand.
- **`@foreach` over `IAsyncEnumerable`** is not supported directly; use `await foreach` in C# 8+ with `.cshtml` runtime compilation.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: `@foreach` List

```cshtml
@foreach (var product in Model.Products)
{
    <p>@product.Name - @product.Price.ToString("C")</p>
}
```

**Expected Output:** One `<p>` per product.

**Why this output?** The loop renders the markup for each item.

---

#### Example 2: `@for` Loop

```cshtml
@for (var i = 0; i < 3; i++)
{
    <p>Item @i</p>
}
```

**Expected Output:** `<p>Item 0</p><p>Item 1</p><p>Item 2</p>`

**Why this output?** The loop runs three times, rendering the markup each iteration.

### Real-World Cases

- **Product listings** – `@foreach` over a collection.
- **Star ratings** – `@for` to render a fixed number of stars.
- **Paginated content** – `@while` for custom pagination logic.

### References

- Looping (`@for`, `@foreach`, `@while`) – https://learn.microsoft.com/en-us/aspnet/core/mvc/views/razor#looping-for-foreach-while-and-do-while

---

## 7. `@switch`

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | `@switch` is a **Razor control structure** that selects a markup block to render based on the value of an expression. |
| **Technical Definition** | `@switch (expression) { case value: ... break; default: ... }` evaluates the expression and executes the matching `case`. Razor renders the markup inside the matching case. |
| **Beginner-Friendly Explanation** | `@switch` is **“choose one of many options”**. Use it when you have several discrete cases—like rendering different icons based on a status value. |

### Purposes (each begins with “To”)

- **To** select one of multiple markup branches based on a value.
- **To** avoid long `if / else if` chains for discrete cases.
- **To** render different UI components for different states.

### Syntax Rules and Structure

#### General Syntax

```cshtml
@switch (Model.Status)
{
    case "Active":
        <p>Active</p>
        break;
    case "Inactive":
        <p>Inactive</p>
        break;
    default:
        <p>Unknown</p>
        break;
}
```

#### Syntax Rules

- **`@` is required on `switch`**.
- **`break` is required** for each case (C# rule).
- **Markup is rendered** for the matched case.

#### Constraints and Limitations

- **Only primitive or string-based switches** are typical; complex pattern matching may require `if` statements.
- **Razor markup inside cases** works normally.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Status Switch

```cshtml
@switch (Model.Status)
{
    case "Active":
        <span class="badge badge-success">Active</span>
        break;
    case "Inactive":
        <span class="badge badge-secondary">Inactive</span>
        break;
    default:
        <span class="badge badge-warning">Unknown</span>
        break;
}
```

**Expected Output:** The badge matching the status.

**Why this output?** Razor evaluates the expression and renders the matching case’s markup.

### Real-World Cases

- **Order status** – different badges or colors.
- **User roles** – different UI elements per role.
- **Payment states** – success, pending, failed.

### References

- `@switch` – https://learn.microsoft.com/en-us/aspnet/core/mvc/views/razor#conditionals-if-else-if-else-and-switch

---

## 8. `@using`

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | `@using` serves two purposes: it **imports a C# namespace** into the view, or it **creates a disposable HTML helper scope** (e.g., `@using (Html.BeginForm()) { ... }`). |
| **Technical Definition** | As a directive, `@using System.IO` adds a using directive to the generated class. As a statement, `@using (Html.BeginForm()) { ... }` wraps content in a disposable object, ensuring proper disposal after rendering. |
| **Beginner-Friendly Explanation** | `@using` is **“bring in a namespace”** or **“wrap this in a helper”**. Use `@using MyApp.Models` at the top of a view, or `@using (Html.BeginForm()) { ... }` to generate a form with proper disposal. |

### Purposes (each begins with “To”)

- **To** import namespaces so types can be used without full qualification.
- **To** create disposable HTML helper scopes (e.g., forms).
- **To** reduce repetitive namespace declarations across multiple views (via `_ViewImports.cshtml`).

### Syntax Rules and Structure

#### General Syntax (Directive)

```cshtml
@using System.IO
@{
    var dir = Directory.GetCurrentDirectory();
}
```

#### General Syntax (Statement)

```cshtml
@using (Html.BeginForm())
{
    <input type="text" name="q" />
}
```

#### Syntax Rules

- **Directive form** has no parentheses and no block.
- **Statement form** requires parentheses and a block.
- **`_ViewImports.cshtml`** applies `@using` directives globally.

#### Constraints and Limitations

- **`@using` directives must appear before markup** (or in `_ViewImports`).
- **`Html.BeginForm`** is a legacy HTML Helper; Tag Helpers (`<form asp-action="...">`) are preferred in modern ASP.NET Core.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Importing a Namespace

```cshtml
@using System.IO
@{
    var path = Path.Combine("a", "b");
}
<p>@path</p>
```

**Expected Output:** The combined path.

**Why this output?** The `@using` directive makes `Path` available.

---

#### Example 2: Disposable Scope

```cshtml
@using (Html.BeginForm("Search", "Products", FormMethod.Get))
{
    <input type="text" name="q" />
    <button type="submit">Search</button>
}
```

**Expected Output:** A `<form>` element wrapping the input and button.

**Why this output?** The using statement ensures the form is properly disposed.

### Real-World Cases

- **Namespace imports** – `@using MyApp.ViewModels`.
- **Form generation** – legacy `Html.BeginForm` scopes.
- **Custom disposable helpers** – wrapping sections of markup.

### References

- `@using` directive – https://learn.microsoft.com/en-us/aspnet/core/mvc/views/razor#using
- Compound `@using` – https://learn.microsoft.com/en-us/aspnet/core/mvc/views/razor#compound-using

---

## 9. `@model`

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | `@model` declares the **strongly typed model** passed to a view, enabling compile-time checking and IntelliSense. |
| **Technical Definition** | `@model TypeName` specifies the generic type parameter for the generated `RazorPage<TModel>` class. The `Model` property in the view is then typed accordingly. If omitted, `Model` is `dynamic`. |
| **Beginner-Friendly Explanation** | `@model` tells Razor **what type of data the view expects**. If you write `@model Product`, then `@Model.Name` works and you get IntelliSense. It makes views safer and easier to write. |

### Purposes (each begins with “To”)

- **To** declare the model type for compile-time checking.
- **To** enable IntelliSense for model properties in the view.
- **To** avoid runtime errors caused by typos in property names.
- **To** make views self-documenting about their expected data.

### Syntax Rules and Structure

#### General Syntax

```cshtml
@model MyApp.Models.Product
<h1>@Model.Name</h1>
```

#### Syntax Rules

- **`@model` must appear at the top** of the view (before markup).
- **Only one `@model` per view**.
- **Generic models** are supported: `@model List<Product>`.

#### Constraints and Limitations

- **Without `@model`** , `Model` is `dynamic` (no compile-time checking).
- **In `_ViewImports.cshtml`** , you can add `@using` but not `@model` (it's view-specific).

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Strongly Typed Model

```cshtml
@model MyApp.Models.Product
<h1>@Model.Name</h1>
<p>@Model.Price.ToString("C")</p>
```

**Expected Output:** Product name and formatted price.

**Why this output?** `@Model` is typed as `Product`; property access is checked at compile time.

---

#### Example 2: Collection Model

```cshtml
@model IEnumerable<MyApp.Models.Product>
@foreach (var p in Model)
{
    <p>@p.Name</p>
}
```

**Expected Output:** A list of product names.

**Why this output?** `Model` is `IEnumerable<Product>`; `foreach` iterates safely.

### Real-World Cases

- **Detail pages** – `@model Product`.
- **List pages** – `@model IEnumerable<Product>`.
- **Form pages** – `@model CreateProductViewModel`.

### References

- `@model` directive – https://learn.microsoft.com/en-us/aspnet/core/mvc/views/razor#model

---

## 10. `@inject`

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | `@inject` **injects a service** from the dependency injection container into a view. |
| **Technical Definition** | `@inject ServiceType PropertyName` adds a property to the generated view class, populated by the service provider. The service must be registered in `Program.cs`. |
| **Beginner-Friendly Explanation** | `@inject` lets you **use a service inside a view** without going through the controller. For example, `@inject IConfiguration Config` makes `Config` available in the view. |

### Purposes (each begins with “To”)

- **To** access a registered service directly from a view.
- **To** avoid passing services through the controller when only the view needs them.
- **To** use configuration, localization, or utility services in views.

### Syntax Rules and Structure

#### General Syntax

```cshtml
@inject IConfiguration Config
<p>@Config["MySetting"]</p>
```

#### Syntax Rules

- **Service must be registered** in DI; otherwise, runtime error.
- **Property name** can be any valid C# identifier.
- **`@inject` can appear multiple times**.

#### Constraints and Limitations

- **Overuse makes views depend on services**, reducing testability; prefer passing data via the model.
- **Not a replacement for ViewModels** for data.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Injecting Configuration

```cshtml
@inject IConfiguration Config
<p>App name: @Config["AppName"]</p>
```

**Expected Output:** The configured app name.

**Why this output?** `Config` is injected and queried.

---

#### Example 2: Injecting a Localizer

```cshtml
@inject IViewLocalizer Localizer
<h1>@Localizer["Title"]</h1>
```

**Expected Output:** The localized title string.

**Why this output?** The localizer service provides translated strings.

### Real-World Cases

- **Configuration values** – feature flags, app names.
- **Localization** – `IViewLocalizer`.
- **Utility services** – URL builders, HTML sanitizers.

### References

- `@inject` directive – https://learn.microsoft.com/en-us/aspnet/core/mvc/views/razor#inject
- Dependency injection into views – https://learn.microsoft.com/en-us/aspnet/core/mvc/views/dependency-injection

---

## 11. `@functions`

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | `@functions` adds **C# members (fields, properties, methods)** to the generated view class. |
| **Technical Definition** | `@functions { ... }` allows you to define methods and properties that can be called from markup. These methods can also contain markup, serving as templating methods. |
| **Beginner-Friendly Explanation** | `@functions` lets you **write helper methods inside a view**. If you need a reusable snippet of markup or logic, define it in `@functions` and call it from the view body. |

### Purposes (each begins with “To”)

- **To** add reusable helper methods to a view.
- **To** encapsulate small pieces of rendering logic.
- **To** avoid repeating markup patterns.
- **To** replace the legacy `@helper` directive (not supported in ASP.NET Core).

### Syntax Rules and Structure

#### General Syntax

```cshtml
@functions {
    public string GetHello() => "Hello";
}
<p>@GetHello()</p>
```

#### Syntax Rules

- **Can contain fields, properties, and methods**.
- **Methods can include markup** (templating methods).
- **Multiple `@functions` blocks are allowed**.

#### Constraints and Limitations

- **Not for heavy logic** – move complex code to services.
- **In Blazor components** , `@code` is preferred over `@functions`.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Simple Helper Method

```cshtml
@functions {
    public string FormatPrice(decimal price) => price.ToString("C");
}
<p>@FormatPrice(19.99m)</p>
```

**Expected Output:** `<p>$19.99</p>`

**Why this output?** The method formats the price and is called from markup.

---

#### Example 2: Templating Method with Markup

```cshtml
@functions {
    private void RenderName(string name)
    {
        <p>Name: <strong>@name</strong></p>
    }
}
@{ RenderName("Alice"); }
```

**Expected Output:** `<p>Name: <strong>Alice</strong></p>`

**Why this output?** The method includes markup, which is rendered when called.

### Real-World Cases

- **Formatting helpers** – currency, dates, status labels.
- **Reusable markup snippets** – rendering a user badge.
- **Replacing `@helper`** – migrating legacy code.

### References

- `@functions` directive – https://learn.microsoft.com/en-us/aspnet/core/mvc/views/razor#functions
- Templating methods – https://learn.microsoft.com/en-us/aspnet/core/mvc/views/razor#functions

---

## 12. `@section`

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | `@section` defines a **named content section** in a view that can be rendered in a specific location of a layout. |
| **Technical Definition** | `@section Name { ... }` defines a section. The layout renders it with `@await RenderSectionAsync("Name", required: false)`. Sections are used with layouts to inject scripts, styles, or other content into specific parts of the HTML page. |
| **Beginner-Friendly Explanation** | `@section` lets you **fill in a placeholder in your layout**. For example, define `@section Scripts { ... }` in a view, and the layout renders those scripts at the bottom of the page. |

### Purposes (each begins with “To”)

- **To** inject page-specific content (scripts, styles) into a shared layout.
- **To** keep layout markup clean by separating page-specific sections.
- **To** control where content is rendered in the final HTML.

### Syntax Rules and Structure

#### General Syntax (View)

```cshtml
@section Scripts {
    <script src="~/js/product.js"></script>
}
```

#### General Syntax (Layout)

```cshtml
@await RenderSectionAsync("Scripts", required: false)
```

#### Syntax Rules

- **Section names are case-sensitive**.
- **`required: false`** makes the section optional.
- **Sections are defined only in views**, not in layouts.

#### Constraints and Limitations

- **A section can be defined only once per view**.
- **Layout must call `RenderSection`** for the section to appear.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Scripts Section

```cshtml
@* View *@
@section Scripts {
    <script>console.log('page loaded');</script>
}
```

```cshtml
@* Layout *@
@await RenderSectionAsync("Scripts", required: false)
```

**Expected Output:** The script tag appears at the location of `RenderSectionAsync`.

**Why this output?** The view defines the section; the layout renders it.

### Real-World Cases

- **Page-specific JavaScript** – `@section Scripts`.
- **Custom CSS** – `@section Styles`.
- **Meta tags** – `@section Meta`.

### References

- `@section` directive – https://learn.microsoft.com/en-us/aspnet/core/mvc/views/razor#section
- Layout in ASP.NET Core – https://learn.microsoft.com/en-us/aspnet/core/mvc/views/layout

---

## 13. `@helper` (Legacy)

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | `@helper` was a legacy Razor directive for defining reusable markup helpers. **It is not supported in ASP.NET Core.** |
| **Technical Definition** | The `@helper` directive allowed defining methods that returned markup inline. In ASP.NET Core, this directive was removed; the recommended replacement is `@functions` with templating methods, or local methods, or Tag Helpers/View Components. |
| **Beginner-Friendly Explanation** | `@helper` was the **old way** to make reusable snippets. In modern ASP.NET Core, it doesn't work. Use `@functions` instead. |

### Purposes (each begins with “To”)

- **To** explain the legacy directive and why it is not used.
- **To** guide migration from `@helper` to `@functions`.
- **To** highlight the modern alternatives for reusable UI.

### Syntax Rules and Structure (Legacy – Not Supported)

```cshtml
@* ❌ Does NOT work in ASP.NET Core *@
@helper FormatPrice(decimal price) {
    <span>@price.ToString("C")</span>
}
```

#### Modern Replacement

```cshtml
@functions {
    private IHtmlContent FormatPrice(decimal price)
    {
        return new HtmlString($"<span>{price.ToString("C")}</span>");
    }
}
```

#### Syntax Rules

- **`@helper` is not available** in ASP.NET Core.
- **Use `@functions`** for helper methods.
- **Use View Components** for more complex reusable UI.

#### Constraints and Limitations

- **Legacy code must be migrated**; upgrade tools convert `@helper` to local methods.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Migration from `@helper` to `@functions`

```cshtml
@* Legacy (ASP.NET MVC 5) *@
@helper Bold(string text) { <strong>@text</strong> }

@* ASP.NET Core equivalent *@
@functions {
    private IHtmlContent Bold(string text) => new HtmlString($"<strong>{text}</strong>");
}
```

**Expected Output:** The bold markup is rendered when the function is called.

**Why this output?** `@functions` provides the same capability without the removed directive.

### Real-World Cases

- **Migrating legacy projects** – replacing `@helper` with `@functions`.
- **Modern alternatives** – Tag Helpers and View Components for UI reuse.

### References

- `@helper` not supported in ASP.NET Core – https://stackoverflow.com/questions/44493370/why-is-helper-not-supported-in-asp-net-core
- Upgrade Assistant conversion of `@helper` – https://www.codemag.com/article/1901071/Bring-Your-.NET-Apps-Forward-with-the-.NET-Upgrade-Assistant

---

## 14. Razor Best Practices

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Razor best practices are guidelines for writing views that are **readable, safe, maintainable, and testable**, keeping them focused on presentation. |
| **Technical Definition** | Best practices include: using strongly typed models (`@model`), avoiding complex logic in views, relying on Razor's automatic HTML encoding, using Tag Helpers over HTML Helpers, leveraging `_ViewImports.cshtml` for shared directives, keeping sections minimal, and choosing `@functions` or services for reusable logic instead of legacy `@helper`. |
| **Beginner-Friendly Explanation** | Best practices are the **rules of thumb** for clean Razor. For example: use `@model` for strong typing, don't put business logic in views, let Razor encode output to prevent XSS, and use `@functions` instead of the old `@helper`. |

### Purposes (each begins with “To”)

- **To** keep views **presentation-focused**, delegating logic to controllers/services.
- **To** ensure **security** by relying on Razor's automatic HTML encoding and avoiding `@Html.Raw` with untrusted input.
- **To** improve **maintainability** through consistent use of `@model`, `_ViewImports`, and `@functions`.
- **To** promote **reuse** with `@functions`, partials, and view components.
- **To** avoid **legacy syntax** like `@helper` that is not supported.

### Guideline Checklist

| Guideline | Rationale | Example |
|-----------|-----------|---------|
| **Use `@model` for strong typing** | Compile-time checking and IntelliSense. | `@model Product` instead of `ViewBag` |
| **Avoid business logic in views** | Views should display; logic belongs in services. | Compute totals in the controller. |
| **Rely on automatic HTML encoding** | Prevents XSS. | `@Model.Name` (encoded) instead of `@Html.Raw`. |
| **Use `_ViewImports.cshtml`** | Consistent directives and Tag Helpers. | `@addTagHelper *, Microsoft.AspNetCore.Mvc.TagHelpers` |
| **Use `@functions` instead of `@helper`** | `@helper` is not supported. | Define methods in `@functions`. |
| **Use Tag Helpers over HTML Helpers** | Cleaner, HTML-friendly syntax. | `<a asp-action="Index">` instead of `@Html.ActionLink`. |
| **Keep `@section` content minimal** | Sections are for page-specific assets. | `@section Scripts { ... }` |
| **Avoid deep nesting of control structures** | Readability. | Extract complex logic to `@functions` or services. |

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Strongly Typed, Encoded View

```cshtml
@model Product
<h1>@Model.Name</h1>
<p>@Model.Description</p>
```

**Expected Output:** Encoded product name and description.

**Why this output?** `@model` provides type safety; Razor encodes output, preventing XSS.

---

#### Example 2: Using `@functions` for Reuse

```cshtml
@functions {
    private IHtmlContent Badge(string status) =>
        new HtmlString($"<span class='badge'>{status}</span>");
}
<p>@Badge(Model.Status)</p>
```

**Expected Output:** A styled badge.

**Why this output?** The helper method encapsulates the badge markup.

### Real-World Cases

- **E-commerce** – strongly typed product views.
- **Admin dashboards** – view components for reusable widgets.
- **Multi-tenant apps** – ViewModels shape tenant-specific data.

### References

- Razor syntax reference – https://learn.microsoft.com/en-us/aspnet/core/mvc/views/razor
- Preventing XSS in ASP.NET Core – https://learn.microsoft.com/en-us/aspnet/core/security/cross-site-scripting

---

## Summary Table: Razor Syntax at a Glance

| Feature | Purpose | Key Syntax | Notes |
|---------|---------|------------|-------|
| **Razor** | Embed C# in HTML | `@` transitions | Default language is HTML |
| **`@`** | Switch to C# | `@Model.Name`, `@@` escape | Implicit/explicit expressions |
| **`@{ }`** | Code block | `@{ var x = 1; }` | No output unless markup added |
| **`@()`** | Explicit expression | `@(a + b)` | For complex/spaced expressions |
| **`@if` / `@else`** | Conditional rendering | `@if (cond) { ... }` | `else` needs no `@` |
| **`@foreach` / `@for` / `@while`** | Looping | `@foreach (var x in list) { ... }` | Templated loops |
| **`@switch`** | Multi-branch selection | `@switch (x) { case ... }` | Requires `break` |
| **`@using`** | Namespace import / scope | `@using System.IO` / `@using (Html.BeginForm())` | Dual purpose |
| **`@model`** | Strongly typed model | `@model Product` | Compile-time checking |
| **`@inject`** | Service injection | `@inject IConfig Config` | Requires DI registration |
| **`@functions`** | C# members in view | `@functions { ... }` | Replaces `@helper` |
| **`@section`** | Named layout section | `@section Scripts { ... }` | Rendered by layout |
| **`@helper`** | Legacy (not supported) | `@helper ...` | Use `@functions` instead |
| **Best Practices** | Clean, safe views | `@model`, encode, avoid logic | See checklist |

---

## References (Full List)

- Razor syntax reference for ASP.NET Core – https://learn.microsoft.com/en-us/aspnet/core/mvc/views/razor
- ASP.NET Core 的 Razor 語法參考 (Chinese) – https://learn.microsoft.com/zh-tw/aspnet/core/mvc/views/razor
- Razor syntax reference (GitHub) – https://github.com/dotnet/AspNetCore.Docs/blob/main/aspnetcore/mvc/views/razor.md
- `@helper` not supported (Code Magazine) – https://www.codemag.com/article/1901071/Bring-Your-.NET-Apps-Forward-with-the-.NET-Upgrade-Assistant
- `@helper` migration (Stack Overflow) – https://stackoverflow.com/questions/44493370/why-is-helper-not-supported-in-asp-net-core
- Dependency injection into views – https://learn.microsoft.com/en-us/aspnet/core/mvc/views/dependency-injection
- Layout in ASP.NET Core – https://learn.microsoft.com/en-us/aspnet/core/mvc/views/layout
- Preventing XSS in ASP.NET Core – https://learn.microsoft.com/en-us/aspnet/core/security/cross-site-scripting
- Blazor `@code` vs `@functions` – https://learn.microsoft.com/en-us/aspnet/core/blazor/components/