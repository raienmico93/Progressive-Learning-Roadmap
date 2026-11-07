# Strongly Typed Views in ASP.NET Core MVC: A Comprehensive Cheat Sheet

> **Version Scope & Compatibility Note**
> This cheat sheet targets **ASP.NET Core MVC** (3.0 and later, including .NET 6/7/8/9/10/11). The `@model` directive and strongly typed view features have been stable across ASP.NET Core versions. `RazorPage<TModel>` is the generated base class for strongly typed views; views without `@model` derive from `RazorPage<dynamic>` (dynamic `Model`), which the framework treats as weakly typed. **Nullable reference types** (C# 8+) and `@model T?` are supported in modern ASP.NET Core. Razor views are **compiled at build time** by default (via the Razor SDK); runtime compilation is opt-in. Tag Helpers (`asp-for`, `asp-validation-for`, `asp-items`) and HTML Helpers (`Html.TextBoxFor`, `Html.DisplayFor`) both work with strongly typed views; Tag Helpers are recommended for new code.

---

## Topic Overview

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | A strongly typed view is a Razor view that declares the **exact CLR type** of its model using the `@model` directive, enabling compile-time checking, IntelliSense, and type-safe access to model properties. |
| **Technical Definition** | A strongly typed view is a `.cshtml` file whose first Razor directive is `@model T` (where `T` is a fully qualified or imported CLR type). The Razor compiler generates a class deriving from `RazorPage<TModel>` (rather than the weakly typed `RazorPage<dynamic>`), so the `Model` property is statically typed as `T`. This enables **compile-time validation** of property access, **IntelliSense** in editors, and **Tag Helper** binding (`asp-for` uses the expression's type metadata). The model is passed from the controller via `View(viewModel)`, where the runtime type must be assignable to `T`. |
| **Beginner-Friendly Explanation** | A strongly typed view is a view that **declares upfront what type of data it expects**. Instead of `ViewBag.Product` (which could be anything or nothing), you write `@model Product` at the top of the view. Now, when you write `@Model.Name`, the compiler knows `Model` is a `Product` and can check that `Name` actually exists. This catches typos and mistakes before the page ever runs, and gives you autocomplete in your editor. |

### Key Characteristics

- **`@model` directive** – Declares the model type as the first Razor directive in the view.
- **Compile-time safety** – Typos in property names fail at build time (or first render with runtime compilation).
- **IntelliSense** – Editors understand the model type and offer autocomplete for properties.
- **Tag Helper integration** – `asp-for="Name"` binds to `Model.Name` with type-aware metadata.
- **Refactor-safe** – Renaming a model property surfaces all affected views at build time.
- **Foundation for forms** – Strongly typed models enable `asp-for`, `asp-validation-for`, and model-bound POST actions.
- **Documentation by declaration** – The `@model` line tells readers exactly what data the view expects.

### Prerequisites

- **.NET SDK** (6.0+ recommended; examples use .NET 8/10).
- **ASP.NET Core MVC** project with Razor views (`.cshtml`).
- Familiarity with **controllers**, **models**, **ViewModels**, and basic **Razor syntax**.
- Understanding of **namespaces** and **`@using`** directives (for referencing model types).

### Related Programming Areas

- **Razor Syntax** – `@model`, `@using`, `@inject`, `@` expressions.
- **ViewModels** – The typical `@model` type for strongly typed views.
- **Tag Helpers** – `asp-for`, `asp-validation-for`, `asp-items` rely on typed model expressions.
- **HTML Helpers** – `Html.TextBoxFor`, `Html.DisplayFor` accept typed lambda expressions.
- **Model Validation** – Data annotations on the model drive validation in forms.
- **Partial Views** – Partials also declare `@model` for strong typing.

### Core Concepts / Features

1. **Strongly Typed Views (General)** – The concept, benefits, and role in MVC.
2. **`@model`** – The directive that declares the model type.
3. **Model Properties** – Type-safe access to properties in Razor markup.
4. **Strongly Typed View Best Practices** – Guidelines for clean, safe, and maintainable strongly typed views.

---

## 1. Strongly Typed Views (General)

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | A strongly typed view is a Razor view that **declares the CLR type of its model** via `@model T`, giving the framework and the compiler full knowledge of the data the view expects. |
| **Technical Definition** | In ASP.NET Core MVC, a view is compiled into a class. When `@model T` is present, the generated class derives from `RazorPage<T>` (typically `RazorPage<TModel>`), and the `Model` property is statically typed as `T`. Without `@model`, the view derives from `RazorPage<dynamic>`, and `Model` is `dynamic`—no compile-time checking, no IntelliSense, and **runtime errors** for typos. Strongly typed views are the foundation for **Tag Helpers** that require an expression tree (`asp-for`, `asp-validation-for`), and they integrate with `ModelState` validation on POST. |
| **Beginner-Friendly Explanation** | A strongly typed view is **a view that knows what it's displaying**. You tell it at the top: "I'm showing a `Product`," and then everything about the `Product`—its properties, their types, their validation rules—becomes available inside the view. This is much safer than the old `ViewBag` approach, where any typo becomes a runtime error. Strong typing catches mistakes early and makes the code easier to write and change. |

### Purposes (each begins with “To”)

- **To** provide **compile-time checking** of model property access, catching typos early.
- **To** enable **IntelliSense and autocompletion** for model properties in editors.
- **To** integrate seamlessly with **Tag Helpers** (`asp-for`, `asp-validation-for`, `asp-items`) that require typed expressions.
- **To** support **model validation** through data annotations on the declared type.
- **To** make views **self-documenting** by declaring their data contract.
- **To** enable **safe refactoring**: renaming a property surfaces all affected views.
- **To** reduce runtime errors caused by `ViewBag`/`ViewData` inconsistencies.

### Syntax Rules and Structure

#### General Syntax

```cshtml
@model MyApp.ViewModels.ProductDetailsViewModel

<h1>@Model.Name</h1>
<p>@Model.Price.ToString("C")</p>
```

#### Component Breakdown

| Component | Description |
|-----------|-------------|
| `@model` | Razor directive declaring the model type. Must appear at the top of the view. |
| `MyApp.ViewModels.ProductDetailsViewModel` | Fully qualified type name (or short name with `@using`). |
| `Model` | The strongly typed instance of the declared type. |
| `@Model.Name` | Expression accessing a property (encoded by default). |

#### Syntax Rules

- **`@model` must be the first Razor directive** (before any markup or other directives like `@using`).
- **Only one `@model` per view**.
- **Use fully qualified names** or add `@using` in the view or `_ViewImports.cshtml`.
- **`Model`** is the property name; do not shadow it with local variables.
- **Nullable reference types** – `@model T?` is supported; `Model` may be `null` if the controller passes `null`.

#### Constraints and Limitations

- **Missing `@model`** – the view is dynamic; typos become runtime errors (or `RuntimeBinderException`).
- **Type mismatch** – the controller's `View(viewModel)` argument must be assignable to the `@model` type; otherwise, a runtime `InvalidOperationException` is thrown.
- **No logic in views** – strongly typed views display data; business logic belongs in services.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Basic Strongly Typed View

```cshtml
@* Views/Products/Details.cshtml *@
@model MyApp.ViewModels.ProductDetailsViewModel

@{
    ViewData["Title"] = "Product Details";
}

<h1>@Model.Name</h1>
<p>Category: @Model.CategoryName</p>
<p>Price: @Model.Price.ToString("C")</p>
```

```csharp
// Controller action
public async Task<IActionResult> Details(int id)
{
    var product = await _productService.GetByIdAsync(id);
    if (product == null) return NotFound();

    var viewModel = new ProductDetailsViewModel
    {
        Id = product.Id,
        Name = product.Name,
        Price = product.Price,
        CategoryName = product.Category?.Name ?? "(uncategorized)"
    };
    return View(viewModel);   // Type must match @model
}
```

**Step-by-step explanation:**

1. `@model MyApp.ViewModels.ProductDetailsViewModel` declares the type.
2. The controller constructs a `ProductDetailsViewModel` and passes it via `View(viewModel)`.
3. Razor's `Model` property is typed as `ProductDetailsViewModel`.
4. `@Model.Name`, `@Model.CategoryName`, and `@Model.Price` are compile-checked expressions.

**Expected Output:** An HTML page with the product details; typos in property names would fail at build time.

**Why this output?** The strongly typed model gives the compiler full knowledge of the view's data.

---

#### Example 2: Strongly Typed Collection View

```cshtml
@* Views/Products/Index.cshtml *@
@model IEnumerable<MyApp.ViewModels.ProductSummaryViewModel>

<h2>Products</h2>
<table class="table">
    <thead>
        <tr><th>Name</th><th>Price</th></tr>
    </thead>
    <tbody>
        @foreach (var product in Model)
        {
            <tr>
                <td>@product.Name</td>
                <td>@product.Price.ToString("C")</td>
            </tr>
        }
    </tbody>
</table>
```

```csharp
public async Task<IActionResult> Index()
{
    var products = await _productService.GetAllAsync();
    var viewModels = products.Select(p => new ProductSummaryViewModel
    {
        Id = p.Id,
        Name = p.Name,
        Price = p.Price
    });
    return View(viewModels);
}
```

**Expected Output:** An HTML table of products.

**Why this output?** `@model IEnumerable<ProductSummaryViewModel>` makes `Model` a strongly typed collection; `@product.Name` is checked against the element type.

---

#### Example 3: Strongly Typed View with Tag Helpers

```cshtml
@model MyApp.ViewModels.CreateProductViewModel

<form asp-action="Create" method="post">
    <div class="mb-3">
        <label asp-for="Name" class="form-label"></label>
        <input asp-for="Name" class="form-control" />
        <span asp-validation-for="Name" class="text-danger"></span>
    </div>
    <div class="mb-3">
        <label asp-for="Price" class="form-label"></label>
        <input asp-for="Price" class="form-control" />
        <span asp-validation-for="Price" class="text-danger"></span>
    </div>
    <button type="submit" class="btn btn-primary">Create</button>
</form>
```

**Expected Output:** A form with labels and inputs bound to the model; validation messages appear for invalid submissions.

**Why this output?** Tag Helpers use the typed expression (`asp-for="Name"`) to read model metadata (labels, validation attributes) and generate markup.

### Real-World Cases

- **E-commerce** – strongly typed product list, detail, and checkout views.
- **Admin panels** – strongly typed edit forms with validation.
- **Blog engines** – strongly typed post lists and detail views.
- **APIs with MVC fallback** – even JSON endpoints can benefit from strongly typed views when HTML rendering is needed.

### References

- Views in ASP.NET Core MVC – https://learn.microsoft.com/en-us/aspnet/core/mvc/views/overview
- Strongly typed data (ViewData and ViewBag limitations) – https://learn.microsoft.com/en-us/aspnet/core/mvc/views/overview#strongly-typed-data
- Razor syntax reference – https://learn.microsoft.com/en-us/aspnet/core/mvc/views/razor

---

## 2. `@model`

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | `@model` is the Razor directive that declares the **CLR type** of the view's model, enabling compile-time checking, IntelliSense, and Tag Helper type inference. |
| **Technical Definition** | `@model T` causes the generated view class to derive from `RazorPage<TModel>` (typically named `RazorPage<T>` in the ASP.NET Core runtime), and the `Model` property becomes strongly typed as `T`. Without `@model`, the generated class derives from `RazorPage<dynamic>`, making `Model` a `dynamic` reference with **runtime-bound** member access. The directive must be the **first Razor directive** in the file (before any markup or other directives). |
| **Beginner-Friendly Explanation** | `@model` is the **"this view shows a ___"** declaration. Write `@model Product` at the top of the file, and suddenly `Model.Name`, `Model.Price`, and all the other `Product` properties work with full compile-time checking. It's a one-line declaration that unlocks type safety for the entire view. |

### Purposes (each begins with “To”)

- **To** declare the CLR type of the view's model.
- **To** enable **compile-time checking** of property access.
- **To** enable **IntelliSense** for model members in editors.
- **To** provide the type metadata needed by **Tag Helpers** (`asp-for`, `asp-validation-for`).
- **To** serve as documentation of the view's expected data contract.
- **To** support nullable reference types (`@model T?`) in C# 8+.

### Syntax Rules and Structure

#### General Syntax

```cshtml
@model MyApp.ViewModels.ProductDetailsViewModel
```

Or, using a short name via `@using`:

```cshtml
@using MyApp.ViewModels
@model ProductDetailsViewModel
```

Or via `_ViewImports.cshtml` (applies to all views in the folder):

```cshtml
@* Views/_ViewImports.cshtml *@
@using MyApp.ViewModels
```

```cshtml
@* Views/Products/Details.cshtml *@
@model ProductDetailsViewModel
```

#### Component Breakdown

| Component | Description |
|-----------|-------------|
| `@model` | Razor directive. |
| `TypeName` | Fully qualified or imported type. |
| `T?` | Nullable annotation (C# 8+). |
| `Model` | The strongly typed instance created by the framework. |

#### Syntax Rules

- **Must be the first Razor directive**; otherwise, a compile error may occur.
- **Only one `@model` per file**.
- **Directives order** – `@model` must precede `@using`, `@inject`, and markup.
- **`@model` supports generic types** – `@model List<Product>`, `@model IEnumerable<Order>`.
- **Nullable annotations** – `@model Product?` allows `Model` to be null.

#### Constraints and Limitations

- **Type mismatch causes runtime error** – if the controller passes a different type, an `InvalidOperationException` is thrown when rendering.
- **`@model` is per-view** – cannot be set globally in `_ViewImports.cshtml` (only `@using` and `@addTagHelper`).
- **Views with no `@model` are dynamic** – `Model` is `dynamic`; typos become runtime errors.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Simple `@model`

```cshtml
@model MyApp.ViewModels.ProductDetailsViewModel

<h1>@Model.Name</h1>
```

**Step-by-step explanation:**

1. `@model` declares the type.
2. `Model.Name` is compiled against that type.

**Expected Output:** The product name is rendered; typos in `Name` would fail to compile.

**Why this output?** The generated class has `Model` typed as `ProductDetailsViewModel`.

---

#### Example 2: `@model` with Generic Collection

```cshtml
@model IEnumerable<MyApp.ViewModels.ProductSummaryViewModel>

<ul>
@foreach (var product in Model)
{
    <li>@product.Name - @product.Price.ToString("C")</li>
}
</ul>
```

**Expected Output:** A `<ul>` list of products.

**Why this output?** `@model IEnumerable<ProductSummaryViewModel>` types `Model` as an enumerable of the summary type.

---

#### Example 3: Nullable `@model`

```cshtml
@model MyApp.ViewModels.ProductDetailsViewModel?

@if (Model == null)
{
    <p>No product found.</p>
}
else
{
    <h1>@Model.Name</h1>
}
```

**Expected Output:** A friendly "No product found" message if the model is null.

**Why this output?** `@model T?` allows the view to handle missing data safely.

### Real-World Cases

- **Detail views** – `@model ProductDetailsViewModel`.
- **List views** – `@model IEnumerable<ProductSummaryViewModel>`.
- **Edit forms** – `@model EditProductViewModel` with validation attributes.
- **Confirmation pages** – `@model ConfirmationViewModel?` handling optional data.

### References

- `@model` directive – https://learn.microsoft.com/en-us/aspnet/core/mvc/views/razor#model
- Razor syntax reference – https://learn.microsoft.com/en-us/aspnet/core/mvc/views/razor
- Strongly typed data – https://learn.microsoft.com/en-us/aspnet/core/mvc/views/overview#strongly-typed-data

---

## 3. Model Properties

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Model properties are the **members of the strongly typed model** that a view accesses via `@Model.PropertyName` in Razor expressions, with full compile-time checking and IntelliSense. |
| **Technical Definition** | In a strongly typed view, `Model` is of the declared type `T`. Property access uses standard C# member lookup rules; the compiler validates that each referenced property exists on `T` (or its base types/interfaces). Razor outputs the value using the type's `ToString()` (HTML-encoded by default). For collections, iteration is type-safe (`@foreach (var x in Model)` infers `x`'s type). Computed and display-formatted properties are supported (e.g., `FormattedPrice`), but business logic should remain in services. |
| **Beginner-Friendly Explanation** | Model properties are the **fields you display in the view**. Because the model is strongly typed, `@Model.Name` is checked by the compiler—if `Name` doesn't exist, you get an error before the page runs. You can also iterate collections (`@foreach (var item in Model)`), access nested properties (`@Model.Category.Name`), and use computed properties like `@Model.FormattedPrice`. |

### Purposes (each begins with “To”)

- **To** display model data in HTML with **type-safe access**.
- **To** iterate collections safely (`@foreach` over typed enumerables).
- **To** access nested properties with **compile-time checking**.
- **To** leverage **computed properties** for display-only formatting.
- **To** enable **Tag Helper** binding (`asp-for="Model.PropertyName"`) with type metadata.
- **To** support **data annotations** on the model (for display labels and validation).

### Syntax Rules and Structure

#### General Syntax

```cshtml
@Model.Property
@Model.Nested.Property
@(Model.Price * 1.1m)     @* Explicit expression for arithmetic *@
@foreach (var item in Model.Items) { <p>@item.Name</p> }
```

#### Component Breakdown

| Syntax | Behaviour |
|--------|-----------|
| `@Model.Property` | Implicit expression; HTML-encoded output. |
| `@Model.Nested.Property` | Access nested members; safe (nulls throw). |
| `@(expression)` | Explicit expression for calculations, ternaries, method calls. |
| `@foreach (var x in Model)` | Type inference for iteration. |
| `@Model.FormattedPrice` | Computed property. |

#### Syntax Rules

- **Implicit expressions cannot contain spaces** (except after `await`); use `@( ... )` for complex expressions.
- **Nested property access** requires non-null intermediate references; use `@(Model.Category?.Name ?? "none")` for nullable chains.
- **Collections** iterate with `@foreach`, `@for`, or LINQ-derived `Model.Where(...)`.
- **Computed properties** must return display-ready values.

#### Constraints and Limitations

- **`@Model.Property` is HTML-encoded** by default; use `@Html.Raw(...)` only for trusted content.
- **Null reference exceptions** – accessing a property on a null nested object throws; use the null-conditional operator (`?.`).
- **Performance** – LINQ operations in views should be minimal; project data in the controller.
- **No business logic** – keep computed properties simple (formatting only); complex calculations belong in services.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Basic Property Access

```cshtml
@model MyApp.ViewModels.ProductDetailsViewModel

<h1>@Model.Name</h1>
<p>@Model.Price.ToString("C")</p>
<p>@Model.CategoryName</p>
```

**Expected Output:** Product name, formatted price, and category name.

**Why this output?** Each property is compiled against `ProductDetailsViewModel`; typos fail at build time.

---

#### Example 2: Nested Property Access with Null Safety

```cshtml
@model MyApp.ViewModels.OrderDetailsViewModel

<h1>Order #@Model.OrderId</h1>
<p>Customer: @(Model.Customer?.FullName ?? "(unknown)")</p>
<p>Shipping: @(Model.ShippingAddress?.City ?? "(not set)")</p>
```

**Expected Output:** Order ID with customer and city, or fallbacks if the customer/address is missing.

**Why this output?** Null-conditional operators prevent `NullReferenceException` when nested data is absent.

---

#### Example 3: Iterating a Typed Collection

```cshtml
@model MyApp.ViewModels.OrderDetailsViewModel

<ul>
@foreach (var item in Model.LineItems)
{
    <li>@item.ProductName x @item.Quantity — @item.LineTotal.ToString("C")</li>
}
</ul>
```

**Expected Output:** A bulleted list of order line items.

**Why this output?** `Model.LineItems` is typed (e.g., `IEnumerable<LineItemViewModel>`); `item.ProductName` is compile-checked.

---

#### Example 4: Computed Property for Display

```cshtml
@model MyApp.ViewModels.ProductDetailsViewModel

<p>Price: @Model.FormattedPrice</p>
@if (Model.InStock)
{
    <span class="badge badge-success">In Stock</span>
}
else
{
    <span class="badge badge-secondary">Out of Stock</span>
}
```

```csharp
public class ProductDetailsViewModel
{
    public string Name { get; set; } = string.Empty;
    public decimal Price { get; set; }
    public int Stock { get; set; }

    public string FormattedPrice => Price.ToString("C");
    public bool InStock => Stock > 0;
}
```

**Expected Output:** Formatted price and a stock badge based on the computed property.

**Why this output?** Computed properties encapsulate display logic in the ViewModel, keeping the view clean.

### Real-World Cases

- **Product listings** – iterate `IEnumerable<ProductSummaryViewModel>`.
- **Order details** – nested access to customer and line items.
- **Profile pages** – display computed values like `FullName` and `MemberSince`.
- **Dashboards** – iterate metric collections.

### References

- Razor syntax reference (implicit and explicit expressions) – https://learn.microsoft.com/en-us/aspnet/core/mvc/views/razor
- Strongly typed data – https://learn.microsoft.com/en-us/aspnet/core/mvc/views/overview#strongly-typed-data

---

## 4. Strongly Typed View Best Practices

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Strongly typed view best practices are guidelines for **declaring, structuring, and using strongly typed models** in Razor views to maximise safety, readability, and maintainability. |
| **Technical Definition** | Best practices include: always declaring `@model` on views that access model data, using **ViewModels** rather than domain entities, importing namespaces via `_ViewImports.cshtml`, using nullable annotations for optional models, leveraging Tag Helpers with typed expressions, avoiding `dynamic`/`ViewBag` for structured data, using computed properties for display formatting, handling null safety with `?.` operators, and keeping views presentation-only (no business logic). |
| **Beginner-Friendly Explanation** | Best practices are the **rules of thumb** that keep strongly typed views clean and safe. For example: always add `@model`, use ViewModels instead of domain entities, use nullable annotations when data might be missing, and use `?.` when accessing nested data. These habits catch bugs early and make views easier to understand. |

### Purposes (each begins with “To”)

- **To** ensure **compile-time safety** for every view that accesses model data.
- **To** keep views **decoupled from domain entities** by using ViewModels.
- **To** reduce boilerplate by importing namespaces through `_ViewImports.cshtml`.
- **To** handle **optional data** safely via nullable annotations and null-conditional access.
- **To** leverage **Tag Helpers** (which require typed expressions) for cleaner markup.
- **To** keep views **presentation-only** by moving logic to controllers/services.
- **To** enable **safe refactoring** when model properties change.

### Guideline Checklist

| Guideline | Rationale | Example |
|-----------|-----------|---------|
| **Always declare `@model`** | Enables compile-time checking and IntelliSense. | `@model ProductDetailsViewModel` |
| **Use ViewModels, not domain entities** | Prevents exposing internals and over-posting. | `@model ProductDetailsViewModel`, not `@model Product` |
| **Import namespaces via `_ViewImports.cshtml`** | Reduces repetition across views. | `@using MyApp.ViewModels` in `_ViewImports.cshtml` |
| **Use nullable `@model T?` when data may be null** | Avoids null reference errors. | `@model ProductDetailsViewModel?` |
| **Use `?.` for nested access** | Prevents `NullReferenceException`. | `@(Model.Category?.Name ?? "—")` |
| **Use Tag Helpers over HTML Helpers** | Cleaner markup, type-aware. | `<input asp-for="Name" />` instead of `@Html.TextBoxFor(m => m.Name)` |
| **Avoid `ViewBag`/`ViewData` for structured data** | No compile-time checking. | Use strongly typed ViewModel properties. |
| **Keep computed properties display-only** | Business logic belongs in services. | `FormattedPrice`, not `CalculateDiscount()` |
| **Avoid LINQ with side effects in views** | Views should be pure rendering. | Project in the controller. |
| **Validate `Model` before use in nullable views** | Explicit null checks. | `@if (Model == null) { ... }` |

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Nullable Model with Null-Conditional Access

```cshtml
@model MyApp.ViewModels.ProductDetailsViewModel?

@if (Model == null)
{
    <p class="text-muted">Product not found.</p>
}
else
{
    <h1>@Model.Name</h1>
    <p>@Model.FormattedPrice</p>
    <p>Category: @(Model.CategoryName ?? "Uncategorized")</p>
}
```

**Expected Output:** A friendly message if the model is null; product details otherwise.

**Why this output?** Nullable `@model` and null checks prevent `NullReferenceException`.

---

#### Example 2: Using `_ViewImports.cshtml` to Avoid Repetition

```cshtml
@* Views/_ViewImports.cshtml *@
@using MyApp.ViewModels
@addTagHelper *, Microsoft.AspNetCore.Mvc.TagHelpers
```

```cshtml
@* Views/Products/Details.cshtml *@
@model ProductDetailsViewModel   @* No need to fully qualify *@

<h1>@Model.Name</h1>
```

**Expected Output:** The view compiles with the imported type without a fully qualified name.

**Why this output?** `_ViewImports.cshtml` applies `@using` directives to all views in the folder and its subfolders.

---

#### Example 3: Refactor-Safe Renaming

```csharp
// Before
public class ProductDetailsViewModel
{
    public string Title { get; set; } = string.Empty;
}

// Rename to Name
public class ProductDetailsViewModel
{
    public string Name { get; set; } = string.Empty;
}
```

```cshtml
@* ❌ Old view uses Title — this fails to compile *@
<h1>@Model.Title</h1>

@* ✅ Updated view uses Name *@
<h1>@Model.Name</h1>
```

**Expected Output:** Compilation errors highlight views still using the old property name.

**Why this output?** Strong typing surfaces all affected views during refactoring, preventing silent breakage.

---

#### Example 4: Avoiding `ViewBag` for Structured Data

```cshtml
@* ❌ Weakly typed (runtime errors possible) *@
<h1>@ViewBag.Product.Name</h1>

@* ✅ Strongly typed *@
@model ProductDetailsViewModel
<h1>@Model.Name</h1>
```

**Expected Output:** The strongly typed version is compile-checked; the `ViewBag` version fails at runtime if `Product` is missing.

**Why this output?** `ViewBag` is dynamic; typos become runtime exceptions. Strongly typed models are checked at build time.

### Real-World Cases

- **E-commerce** – strongly typed product, cart, and checkout views with nullable handling for missing data.
- **Admin panels** – strongly typed edit forms with validation and safe null handling.
- **Multi-tenant SaaS** – strongly typed ViewModels with tenant-specific properties; nullability signals optional data.
- **CMS** – strongly typed article views with nested category and tag collections.

### References

- Views in ASP.NET Core MVC – https://learn.microsoft.com/en-us/aspnet/core/mvc/views/overview
- Strongly typed data – https://learn.microsoft.com/en-us/aspnet/core/mvc/views/overview#strongly-typed-data
- Razor syntax reference – https://learn.microsoft.com/en-us/aspnet/core/mvc/views/razor
- Importing shared directives (`_ViewImports.cshtml`) – https://learn.microsoft.com/en-us/aspnet/core/mvc/views/layout#importing-shared-directives
- Tag Helpers in ASP.NET Core – https://learn.microsoft.com/en-us/aspnet/core/mvc/views/tag-helpers/intro
- Model validation in ASP.NET Core – https://learn.microsoft.com/en-us/aspnet/core/mvc/models/validation
- ViewModels in ASP.NET Core MVC (companion cheat sheet) – *(this conversation series)*

---

## Summary Table: Strongly Typed Views at a Glance

| Concept | Primary Purpose | Key Takeaway | When to Use |
|---------|----------------|--------------|-------------|
| **Strongly Typed Views** | Declare the view's data type | Compile-time safety, IntelliSense, Tag Helper integration | Every view that accesses model data |
| **`@model`** | Declare the model type | First Razor directive; fully qualified or imported | Every strongly typed view |
| **Model Properties** | Access data safely | Compile-checked access; null-conditional for nested data | Every data access in a view |
| **Best Practices** | Maintainability and safety | Use ViewModels, nullable annotations, `_ViewImports`, avoid `ViewBag` | All production views |

---

## References (Full List)

- Views in ASP.NET Core MVC – https://learn.microsoft.com/en-us/aspnet/core/mvc/views/overview
- Strongly typed data – https://learn.microsoft.com/en-us/aspnet/core/mvc/views/overview#strongly-typed-data
- Razor syntax reference – https://learn.microsoft.com/en-us/aspnet/core/mvc/views/razor
- `@model` directive – https://learn.microsoft.com/en-us/aspnet/core/mvc/views/razor#model
- Importing shared directives (`_ViewImports.cshtml`) – https://learn.microsoft.com/en-us/aspnet/core/mvc/views/layout#importing-shared-directives
- Tag Helpers in ASP.NET Core – https://learn.microsoft.com/en-us/aspnet/core/mvc/views/tag-helpers/intro
- Model validation in ASP.NET Core – https://learn.microsoft.com/en-us/aspnet/core/mvc/models/validation
- `RazorPage<TModel>` API – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.razor.razorpage-1
- Razor view compilation – https://learn.microsoft.com/en-us/aspnet/core/mvc/views/view-compilation
- Partial views in ASP.NET Core (strongly typed partials) – https://learn.microsoft.com/en-us/aspnet/core/mvc/views/partial