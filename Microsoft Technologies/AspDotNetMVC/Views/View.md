# View Fundamentals in ASP.NET Core MVC: A Comprehensive Cheat Sheet

> **Version Scope & Compatibility Note**
> This cheat sheet targets **ASP.NET Core MVC** (3.0 and later, including .NET 6/7/8/9/10/11). Razor views (`.cshtml`) and the **Razor view engine** have been stable across versions. Since .NET 6, Razor views are **compiled at build time** by default (via the Razor SDK); **runtime compilation** is opt-in through the `Microsoft.AspNetCore.Mvc.Razor.RuntimeCompilation` package. The view discovery mechanism (`RazorViewEngine`, `IViewLocationExpander`) and the default search paths (`/Views/{Controller}/{View}.cshtml`, `/Views/Shared/{View}.cshtml`) are consistent across modern ASP.NET Core. **Razor Pages** use a different discovery model (page paths under `Pages/`), which is not the focus here; MVC views are covered.

---

## Topic Overview

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Views are **Razor template files** (`.cshtml`) that combine HTML markup with server-side C# code to render the user interface of an MVC application. |
| **Technical Definition** | A view is a `.cshtml` file processed by the **Razor view engine**. The engine parses Razor syntax (`@` expressions, code blocks, directives like `@model`, `@using`, `@inject`) and compiles the file into a class deriving from `RazorPage<T>` (or `RazorPage` for non-generic views). Views are discovered via **`IViewLocationExpander`** implementations configured on `RazorViewEngineOptions.ViewLocationFormats` (and area variants). Views receive a strongly typed model via `@model`, may reference layouts (`_Layout.cshtml`), partial views, view components, and Tag Helpers, and are rendered by `ViewResult` returned from controller actions. |
| **Beginner-Friendly Explanation** | Views are the **HTML templates** of your application—what the user actually sees in the browser. A view is a `.cshtml` file that mixes normal HTML with C# code (Razor syntax), so you can display data from your controller. Controllers hand a model to a view, and the view turns it into the final HTML page. Views live in the `Views` folder, organized by controller name so the framework can find them automatically. |

### Key Characteristics

- **Razor syntax** – Combines HTML and C# using `@` expressions, code blocks, and directives.
- **Strongly typed models** – `@model T` provides compile-time checking and IntelliSense.
- **Layout support** – `_Layout.cshtml` wraps views with shared header/footer via `_ViewStart.cshtml` and `@RenderBody()`.
- **Partial views and view components** – Reusable UI fragments and logic-bearing components.
- **Tag Helpers** – Server-side attributes in markup (`asp-action`, `asp-controller`, `asp-for`) for cleaner HTML.
- **Shared imports** – `_ViewImports.cshtml` provides shared `@using` and `@addTagHelper` directives.
- **Convention-based discovery** – The view engine searches predefined paths (`/Views/{Controller}/{View}.cshtml`, `/Views/Shared/{View}.cshtml`).
- **Compiled at build time** – Razor views are compiled by default in modern .NET, with runtime compilation opt-in.
- **Extensible discovery** – `IViewLocationExpander` enables custom conventions (e.g., feature folders).
- **Testable** – Views should be "dumb" (display-only); logic belongs in controllers/services.

### Prerequisites

- **.NET SDK** (6.0+ recommended; examples use .NET 8/10).
- **ASP.NET Core MVC** services registered (`AddControllersWithViews()`).
- Basic understanding of **controllers**, **models**, **Razor syntax**, and **HTML**.
- Familiarity with **C# expressions** and **data annotations** (helpful but not required).

### Related Programming Areas

- **Controllers** – Pass models to views via `View(model)`.
- **ViewModels** – Shape data for specific views.
- **Layouts** – Shared page structure (`_Layout.cshtml`).
- **Partial Views** – Reusable fragments (`_Partial.cshtml`).
- **View Components** – Reusable UI with logic (`ViewComponent`).
- **Tag Helpers** – Server-side helpers for HTML generation.
- **Razor Pages** – An alternative page-centric model using Razor.
- **Razor Class Libraries** – Package reusable views for distribution.

### Core Concepts / Features

1. **Views (General)** – The role of views in MVC and how they work.
2. **View Discovery** – How the framework locates a view for an action.
3. **View Locations** – The default search paths and how to customize them.
4. **View Naming** – Conventions for naming views and referencing them explicitly.
5. **View Best Practices** – Guidelines for maintainable, secure, and performant views.

---

## 1. Views (General)

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | A view is a **Razor template** (`.cshtml`) that renders HTML to the browser, receiving data from the controller as a model. |
| **Technical Definition** | In MVC, a view is a `.cshtml` file processed by the Razor view engine. The engine compiles it into a class deriving from `RazorPage<TModel>` (for `@model T` views) or `RazorPage<dynamic>` (for views without `@model`). Directives (`@model`, `@using`, `@inject`, `@addTagHelper`), code blocks (`@{ ... }`), expressions (`@Model.Name`), and markup are combined. The framework renders the view via `ViewResult`, which locates the view using `IViewEngine`, executes it with the model, and writes the resulting HTML to `HttpResponse.Body`. |
| **Beginner-Friendly Explanation** | A view is the **page the user sees**. It's an HTML file with some C# mixed in (called Razor), so you can do things like `@Model.ProductName` to display data. Controllers prepare the data and hand it to a view; the view turns it into HTML for the browser. Views live in the `Views` folder, organized by controller name so the framework knows where to find them. |

### Purposes (each begins with “To”)

- **To** separate the presentation layer (HTML) from business logic, following the **separation of concerns** principle.
- **To** render data from the controller (models or ViewModels) as user-facing HTML.
- **To** reuse shared page structure (headers, footers, navigation) through **layouts**.
- **To** encapsulate reusable UI fragments in **partial views** and **view components**.
- **To** leverage **Tag Helpers** for cleaner, more maintainable markup than raw HTML helpers.
- **To** benefit from **compile-time checking** through strongly typed `@model` declarations.

### Syntax Rules and Structure

#### General Syntax (Basic View)

```cshtml
@model MyApp.Models.Product

<h1>@Model.Name</h1>
<p>Price: @Model.Price.ToString("C")</p>
```

#### Component Breakdown

| Component | Description |
|-----------|-------------|
| `@model` | Declares the strongly typed model. |
| `@{ ... }` | Code block for statements. |
| `@expression` | Outputs the value of a C# expression (HTML-encoded). |
| `@Html.Raw(...)` | Outputs unencoded HTML (use with care). |
| `@using`, `@addTagHelper`, `@inject` | Directives for imports and DI. |
| `<tag asp-...>` | Tag Helpers that run server-side. |

#### Syntax Rules

- **View files use `.cshtml` extension** and live in `Views/`.
- **`@model` must be the first Razor directive** (before any markup).
- **`@` outputs HTML-encoded values** by default; use `@Html.Raw` for trusted unencoded content.
- **Code blocks must be balanced** with `@if`, `@foreach`, `@for`, etc.
- **`_ViewImports.cshtml`** provides shared directives (see Section 4).

#### Constraints and Limitations

- **No business logic in views** – views should only display data; delegate logic to controllers/services.
- **`ViewBag`/`ViewData` are dynamic** – prefer `@model` for compile-time checking.
- **`@Html.Raw` requires trusted content** – unencoded user input is an XSS risk.
- **Views are compiled** – errors in Razor appear at build time (unless runtime compilation is enabled).

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Basic Strongly Typed View

```cshtml
@* Views/Products/Details.cshtml *@
@model MyApp.Models.Product

@{
    ViewData["Title"] = "Product Details";
}

<h1>@Model.Name</h1>
<p>Price: @Model.Price.ToString("C")</p>
<p>In stock: @(Model.Stock > 0 ? "Yes" : "No")</p>
```

**Step-by-step explanation:**

1. `@model MyApp.Models.Product` declares the model type.
2. `@{ ViewData["Title"] = ... }` sets the page title (used in `_Layout`).
3. `@Model.Name` and `@Model.Price` output encoded values.
4. The ternary expression inside `@(...)` computes the stock status.

**Expected Output:** An HTML page with the product name, price formatted as currency, and stock status.

**Why this output?** The controller passes a `Product` model to the view; Razor expressions render its properties.

---

#### Example 2: View with Loop and Conditional

```cshtml
@* Views/Products/Index.cshtml *@
@model IEnumerable<MyApp.Models.Product>

<h2>Products</h2>
@if (!Model.Any())
{
    <p>No products available.</p>
}
else
{
    <ul>
        @foreach (var product in Model)
        {
            <li>@product.Name - @product.Price.ToString("C")</li>
        }
    </ul>
}
```

**Expected Output:** A `<ul>` list of products, or a "No products available." message.

**Why this output?** Razor code blocks (`@if`, `@foreach`) execute C# control flow and emit HTML conditionally.

---

#### Example 3: View with Tag Helpers

```cshtml
@* Views/Products/Details.cshtml *@
@model MyApp.Models.Product

<a asp-controller="Products" asp-action="Edit" asp-route-id="@Model.Id">Edit</a>
<a asp-controller="Products" asp-action="Index">Back to List</a>
```

**Expected Output:** HTML anchors with URLs generated from route values: `<a href="/Products/Edit/5">Edit</a>` and `<a href="/Products">Back to List</a>`.

**Why this output?** The Anchor Tag Helper uses routing and `LinkGenerator` to produce URLs from controller/action/route values.

### Real-World Cases

- **E-commerce** – product listing, detail, checkout pages.
- **Admin dashboards** – data tables, forms, charts.
- **Public sites** – home page, about, privacy policy.

### References

- Views in ASP.NET Core MVC – https://learn.microsoft.com/en-us/aspnet/core/mvc/views/overview
- Razor syntax reference – https://learn.microsoft.com/en-us/aspnet/core/mvc/views/razor

---

## 2. View Discovery

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | View discovery is the process by which the **Razor view engine** locates the correct `.cshtml` file for a controller action based on naming conventions and configured search paths. |
| **Technical Definition** | When an action returns `ViewResult`, the framework asks each registered `IViewEngine` (typically `RazorViewEngine`) to find the view. `RazorViewEngine` iterates over `ViewLocationFormats` (and `AreaViewLocationFormats`) — each a format string with placeholders `{0}` (view name), `{1}` (controller name), and `{2}` (area name) — and checks the file system (via `IViewLocationExpander`-produced candidates) for a matching `.cshtml` file. The first match wins. If no view is found, `ViewResult` throws an `InvalidOperationException` listing the searched locations. |
| **Beginner-Friendly Explanation** | View discovery is **how the framework finds the right HTML file**. When an action says `return View();`, the framework looks for `Views/{Controller}/{Action}.cshtml`, then `Views/Shared/{Action}.cshtml`. If it finds one, it renders it. If not, it throws an error. You can customize these search paths if you want a different folder structure (like feature folders). |

### Purposes (each begins with “To”)

- **To** automatically locate the correct view for a controller action without explicit configuration.
- **To** support fallback search paths (`Views/Shared`) for views used by multiple controllers.
- **To** enable **customization** via `IViewLocationExpander` for alternative folder structures (e.g., feature folders).
- **To** provide **clear error messages** when a view cannot be found, listing the searched locations.
- **To** support **area-specific** views through `AreaViewLocationFormats`.
- **To** cache discovered view locations for performance.

### Syntax Rules and Structure

#### Default Search Order (MVC Views, No Area)

```
1. /Views/{Controller}/{View}.cshtml
2. /Views/Shared/{View}.cshtml
```

#### Default Search Order (MVC Views in an Area)

```
1. /Areas/{Area}/Views/{Controller}/{View}.cshtml
2. /Areas/{Area}/Views/Shared/{View}.cshtml
```

#### Component Breakdown

| Placeholder | Replaced with |
|-------------|---------------|
| `{0}` | View name (action name by default). |
| `{1}` | Controller name (without `Controller` suffix). |
| `{2}` | Area name (if applicable). |

#### Syntax Rules

- **First match wins** — the framework stops at the first file that exists.
- **Case sensitivity** depends on the file system (case-sensitive on Linux); use consistent casing.
- **Extensions** – `.cshtml` and `.razor` are searched; the default is `.cshtml`.
- **View location formats** can be extended or replaced via `RazorViewEngineOptions`.

#### Constraints and Limitations

- **Discovery is file-system–based** — views must exist on disk (or in a Razor Class Library).
- **Custom expanders run on every discovery** — keep them lightweight.
- **Razor Pages differ** — page discovery uses `PageViewLocationFormats`, not the MVC paths.
- **No view found** → `InvalidOperationException` with a detailed list of searched paths.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Default Discovery via `return View()`

```csharp
// Controllers/ProductsController.cs
public IActionResult Index()
{
    var products = _service.GetAll();
    return View(products);   // Discovery: /Views/Products/Index.cshtml, then /Views/Shared/Index.cshtml
}
```

**Step-by-step explanation:**

1. The action returns `View(products)` with no explicit name.
2. The view name defaults to the action name (`Index`).
3. The engine searches `/Views/Products/Index.cshtml` first, then `/Views/Shared/Index.cshtml`.

**Expected Output:** If `/Views/Products/Index.cshtml` exists, it is rendered with `products`; otherwise, the shared view is used.

**Why this output?** The view engine uses the default formats and picks the first existing file.

---

#### Example 2: Custom View Location Expander (Feature Folders)

```csharp
// Infrastructure/FeatureViewLocationExpander.cs
public class FeatureViewLocationExpander : IViewLocationExpander
{
    public void PopulateValues(ViewLocationExpanderContext context)
    {
        // No additional values needed; the controller name is used for the folder
    }

    public IEnumerable<string> ExpandViewLocations(
        ViewLocationExpanderContext context,
        IEnumerable<string> viewLocations)
    {
        // Feature folders: /Features/{Controller}/{View}.cshtml
        yield return "/Features/{1}/{0}.cshtml";
        yield return "/Features/Shared/{0}.cshtml";

        // Preserve default locations as fallback
        foreach (var location in viewLocations) yield return location;
    }
}

// Program.cs
builder.Services.Configure<RazorViewEngineOptions>(options =>
{
    options.ViewLocationExpanders.Add(new FeatureViewLocationExpander());
});
```

**Step-by-step explanation:**

1. `FeatureViewLocationExpander` implements `IViewLocationExpander`.
2. `ExpandViewLocations` yields feature folder paths before the defaults.
3. Registration adds the expander to `RazorViewEngineOptions.ViewLocationExpanders`.

**Expected Output:** Views are searched in `/Features/{Controller}/{View}.cshtml` first, then defaults.

**Why this output?** The view engine consults expanders when building the candidate list; yielding paths early makes them higher priority.

---

#### Example 3: Missing View Produces Detailed Error

```csharp
public IActionResult Missing()
{
    return View();   // No /Views/Home/Missing.cshtml and no shared view
}
```

**Expected Output:** `InvalidOperationException` with a message like:

```
The view 'Missing' was not found. The following locations were searched:
/Views/Home/Missing.cshtml
/Views/Shared/Missing.cshtml
```

**Why this output?** The engine lists all searched locations when no view matches, aiding debugging.

### Real-World Cases

- **Default MVC apps** – views in `Views/{Controller}/`.
- **Shared layouts and errors** – views in `Views/Shared/`.
- **Feature folders** – custom expanders for `Features/{Feature}/`.
- **Razor Class Libraries** – views packaged into a library and discovered via expanders.

### References

- Views in ASP.NET Core MVC – https://learn.microsoft.com/en-us/aspnet/core/mvc/views/overview
- Custom view location with `IViewLocationExpander` – https://learn.microsoft.com/en-us/aspnet/core/mvc/views/overview#custom-view-location-expander
- Razor view compilation – https://learn.microsoft.com/en-us/aspnet/core/mvc/views/view-compilation

---

## 3. View Locations

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | View locations are the **file system paths** (and the format strings that describe them) where the view engine searches for `.cshtml` files. |
| **Technical Definition** | `RazorViewEngineOptions` exposes four primary collections of format strings: `ViewLocationFormats` (MVC views), `AreaViewLocationFormats` (MVC views in areas), `PageViewLocationFormats` (Razor Pages), and `AreaPageViewLocationFormats` (Razor Pages in areas). Each format string contains placeholders `{0}` (view name), `{1}` (controller name), `{2}` (area name). The `RazorViewEngine` iterates these formats for every `FindView` call, augmented by any registered `IViewLocationExpander`s. |
| **Beginner-Friendly Explanation** | View locations are **where the framework looks for view files**. By default, it looks in `Views/{Controller}/` and `Views/Shared/`. If you want a different structure—like feature folders where everything for a feature is in one folder—you can add or change these search paths. |

### Purposes (each begins with “To”)

- **To** define where the view engine searches for views by default (`Views/{Controller}/` and `Views/Shared/`).
- **To** support **area-specific** views via `AreaViewLocationFormats`.
- **To** enable **custom folder structures** (feature folders, Razor Class Libraries) via expanders.
- **To** control **search priority** by ordering the format strings.
- **To** support **multiple view engines** (though Razor is the default) via `IViewEngine` registrations.

### Syntax Rules and Structure

#### Default Formats (from `RazorViewEngineOptions`)

```
ViewLocationFormats:
  /Views/{1}/{0}.cshtml
  /Views/Shared/{0}.cshtml

AreaViewLocationFormats:
  /Areas/{2}/Views/{1}/{0}.cshtml
  /Areas/{2}/Views/Shared/{0}.cshtml
```

#### Custom Formats (Feature Folders)

```csharp
builder.Services.Configure<RazorViewEngineOptions>(options =>
{
    options.ViewLocationFormats.Clear();
    options.ViewLocationFormats.Add("/Features/{1}/{0}.cshtml");
    options.ViewLocationFormats.Add("/Features/Shared/{0}.cshtml");
});
```

#### Component Breakdown

| Format | Meaning |
|--------|---------|
| `/Views/{1}/{0}.cshtml` | `/Views/{Controller}/{View}.cshtml` |
| `/Views/Shared/{0}.cshtml` | `/Views/Shared/{View}.cshtml` |
| `/Areas/{2}/Views/{1}/{0}.cshtml` | `/Areas/{Area}/Views/{Controller}/{View}.cshtml` |
| `/Areas/{2}/Views/Shared/{0}.cshtml` | `/Areas/{Area}/Views/Shared/{View}.cshtml` |

#### Syntax Rules

- **Order matters** – earlier formats are searched first.
- **`{0}`, `{1}`, `{2}`** are placeholders; format strings may contain any leading path.
- **Clear before adding** if you want to replace the defaults entirely.
- **Expanders augment** the format list at runtime; they do not modify the configured formats.

#### Constraints and Limitations

- **Mixing default and custom formats** – recommended to keep defaults as fallbacks.
- **File system case sensitivity** – Linux is case-sensitive; use consistent casing.
- **Razor Class Libraries** – views are embedded; expanders may need to point to their assembly-namespaced paths.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Default View Locations

```
ProjectName/
├── Views/
│   ├── Home/
│   │   └── Index.cshtml              ← /Views/Home/Index.cshtml
│   ├── Shared/
│   │   └── _Layout.cshtml            ← /Views/Shared/_Layout.cshtml
│   └── Products/
│       └── Details.cshtml            ← /Views/Products/Details.cshtml
```

**Expected Output:** `ProductsController.Details()` renders `Views/Products/Details.cshtml`; `HomeController.Index()` renders `Views/Home/Index.cshtml`.

**Why this output?** The default formats match the conventional folder structure.

---

#### Example 2: Replacing Defaults with Feature Folders

```csharp
builder.Services.Configure<RazorViewEngineOptions>(options =>
{
    options.ViewLocationFormats.Clear();
    options.ViewLocationFormats.Add("/Features/{1}/{0}.cshtml");
    options.ViewLocationFormats.Add("/Features/Shared/{0}.cshtml");
});
```

```
ProjectName/
└── Features/
    ├── Products/
    │   ├── Index.cshtml              ← /Features/Products/Index.cshtml
    │   └── Details.cshtml
    └── Shared/
        └── _Layout.cshtml            ← /Features/Shared/_Layout.cshtml
```

**Expected Output:** Views are discovered under `/Features/{Controller}/` and `/Features/Shared/`.

**Why this output?** Clearing and replacing `ViewLocationFormats` changes the search paths; the engine now searches feature folders.

---

#### Example 3: Custom Area Folders

```csharp
builder.Services.Configure<RazorViewEngineOptions>(options =>
{
    options.AreaViewLocationFormats.Clear();
    options.AreaViewLocationFormats.Add("/Modules/{2}/Views/{1}/{0}.cshtml");
    options.AreaViewLocationFormats.Add("/Modules/{2}/Views/Shared/{0}.cshtml");
});
```

```
ProjectName/
└── Modules/
    └── Admin/
        └── Views/
            └── Dashboard/
                └── Index.cshtml      ← /Modules/Admin/Views/Dashboard/Index.cshtml
```

**Expected Output:** Area `Admin` views are resolved under `/Modules/Admin/Views/...`.

**Why this output?** `AreaViewLocationFormats` controls area view discovery; custom formats remap `Areas/` to `Modules/`.

### Real-World Cases

- **Traditional MVC** – default `Views/` structure.
- **Feature folders** – `/Features/{Controller}/` for higher cohesion.
- **Modular apps** – `/Modules/{Area}/Views/` for area-specific paths.
- **Razor Class Libraries** – custom expanders pointing to embedded paths.

### References

- `RazorViewEngineOptions` API – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.razor.razorviewengineoptions
- Custom view location with `IViewLocationExpander` – https://learn.microsoft.com/en-us/aspnet/core/mvc/views/overview#custom-view-location-expander

---

## 4. View Naming

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | View naming is the convention of naming view files after their actions (and controller folders) so that discovery works automatically, and the practice of referencing views explicitly when needed. |
| **Technical Definition** | By convention, a view is named after the action that returns it (`Index.cshtml` for `Index()`, `Details.cshtml` for `Details()`), and stored in `Views/{ControllerName}/`. The `View()` helper overloads allow explicit naming (`View("Name")`, `View("Name", model)`), and paths can be absolute (`View("~/Views/Shared/Custom.cshtml")`) or relative to the current controller's folder. Partial views conventionally start with an underscore (`_ProductRow.cshtml`). Layout files conventionally start with an underscore (`_Layout.cshtml`, `_ViewStart.cshtml`, `_ViewImports.cshtml`). |
| **Beginner-Friendly Explanation** | View naming is **following the naming rules** so the framework can find your files. If your action is `Details()`, name the view `Details.cshtml` and put it in `Views/{Controller}/`. For partials, start with `_` (`_Partial.cshtml`) so they don't accidentally become full-page views. If you want a different name, you can say `View("MyCustomName")`. |

### Purposes (each begins with “To”)

- **To** enable **automatic discovery** by naming views after actions and placing them in controller-specific folders.
- **To** clearly identify **partial views** and **layouts** with an underscore prefix (`_Partial.cshtml`, `_Layout.cshtml`).
- **To** allow **explicit naming** when a view is reused across actions or shared (`View("SharedView")`).
- **To** support **absolute paths** for edge cases (`View("~/Views/Shared/Custom.cshtml")`).
- **To** maintain **consistency** across the codebase so developers can find views predictably.

### Syntax Rules and Structure

#### Naming Conventions

| Type | Convention | Example |
|------|------------|---------|
| Full view | PascalCase, matches action name | `Index.cshtml`, `Details.cshtml` |
| Partial view | Underscore prefix | `_ProductRow.cshtml` |
| Layout | Underscore prefix | `_Layout.cshtml` |
| View start | Fixed name | `_ViewStart.cshtml` |
| View imports | Fixed name | `_ViewImports.cshtml` |
| Error view | `Error.cshtml` | `Views/Shared/Error.cshtml` |

#### Explicit View References

```csharp
return View();                                  // Convention: ActionName.cshtml
return View(model);                             // Same, with model
return View("Details");                         // Explicit name
return View("Details", model);
return View("~/Views/Shared/Custom.cshtml");    // Absolute path
return View("/Views/Shared/Custom.cshtml");     // Absolute path (leading slash)
```

#### Component Breakdown

| Overload | Meaning |
|----------|---------|
| `View()` | View name = action name, in `Views/{Controller}/`. |
| `View(model)` | Same, with a model. |
| `View("Name")` | Explicit name; relative to `Views/{Controller}/` first, then `Views/Shared/`. |
| `View("Name", model)` | Explicit name with model. |
| `View("~/Views/...")` | Absolute path from the web root (`~/`). |
| `View("/Views/...")` | Absolute path (leading `/` is treated as web root). |

#### Syntax Rules

- **View name defaults to the action name** when no argument is given.
- **Explicit names are searched** using the same view location formats.
- **Absolute paths bypass the search formats** and load the file directly.
- **Extensions** may be included (`View("Details.cshtml")`); `.cshtml` is assumed otherwise.
- **Underscore prefix** for partials and layouts is a **convention**, not a requirement, but very strongly recommended.

#### Constraints and Limitations

- **Typos in view names** fail at runtime (if runtime compilation) or at build time (if precompiled with a known missing view — though missing views are runtime errors).
- **Case sensitivity** – use consistent casing; Linux is case-sensitive.
- **Absolute paths are brittle** – moving files breaks them; prefer relative names where possible.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Convention-Based Naming

```csharp
// Controllers/ProductsController.cs
public IActionResult Details(int id)
{
    var product = _service.GetById(id);
    if (product == null) return NotFound();
    return View(product);   // Renders /Views/Products/Details.cshtml
}
```

**Expected Output:** `GET /Products/Details/5` renders `Views/Products/Details.cshtml` with the product.

**Why this output?** The view name is derived from the action (`Details`), and the folder from the controller (`Products`).

---

#### Example 2: Explicit View Name for Reuse

```csharp
public IActionResult List()
{
    var products = _service.GetAll();
    return View("Index", products);   // Reuses Index.cshtml
}
```

**Expected Output:** `GET /Products/List` renders `Views/Products/Index.cshtml` with the products.

**Why this output?** The explicit view name overrides the convention; the same view serves multiple actions.

---

#### Example 3: Absolute Path for Shared View

```csharp
public IActionResult Confirmation()
{
    return View("~/Views/Shared/Confirmation.cshtml");
}
```

**Expected Output:** Renders `/Views/Shared/Confirmation.cshtml` regardless of the controller.

**Why this output?** The `~/` prefix marks the path as absolute, bypassing the search formats.

### Real-World Cases

- **Convention-based views** – `Index`, `Details`, `Create`, `Edit`, `Delete` views per controller.
- **Partial views** – `_ProductCard.cshtml`, `_ValidationScriptsPartial.cshtml`.
- **Layouts** – `_Layout.cshtml`, `_AdminLayout.cshtml`.
- **Shared views** – `_LoginPartial.cshtml`, `Error.cshtml`.

### References

- Views in ASP.NET Core MVC – https://learn.microsoft.com/en-us/aspnet/core/mvc/views/overview
- Partial views in ASP.NET Core – https://learn.microsoft.com/en-us/aspnet/core/mvc/views/partial
- Layout in ASP.NET Core – https://learn.microsoft.com/en-us/aspnet/core/mvc/views/layout

---

## 5. View Best Practices

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | View best practices are guidelines for writing views that are **readable, maintainable, secure, performant, and testable**, keeping them focused on presentation while delegating logic to controllers/services. |
| **Technical Definition** | Best practices include: using strongly typed views (`@model`), avoiding `ViewBag`/`ViewData` for structured data, leveraging layouts and partials for reuse, using Tag Helpers over HTML Helpers, importing shared directives via `_ViewImports.cshtml`, avoiding business logic in views, encoding user input (default in Razor), using `asp-append-version` for cache busting, preferring view components over HTML helpers for logic-bearing UI, and considering feature folders for large apps. |
| **Beginner-Friendly Explanation** | View best practices are the **rules of thumb** for writing clean, safe views. For example: use `@model` instead of `ViewBag` (so you get compile-time checking), don't put business logic in views (do it in controllers/services), use partials for reusable pieces, and always let Razor HTML-encode user input to prevent XSS. |

### Purposes (each begins with “To”)

- **To** keep views **presentation-only**, delegating business logic to controllers and services.
- **To** use **strongly typed models** for compile-time checking and IntelliSense.
- **To** promote **reuse** through layouts, partials, and view components.
- **To** prevent **XSS** by relying on Razor's automatic HTML encoding and avoiding `@Html.Raw` with untrusted input.
- **To** keep views **readable** by using Tag Helpers over verbose HTML Helpers.
- **To** improve **performance** with cache busting (`asp-append-version`) and server-side rendering optimizations.
- **To** support **testability** by keeping logic out of views.

### Guideline Checklist

| Guideline | Rationale | Example |
|-----------|-----------|---------|
| **Use `@model` (strongly typed views)** | Compile-time checking, IntelliSense. | `@model Product` instead of `ViewBag.Product` |
| **Avoid business logic in views** | Views should display; logic belongs in services. | Compute in the controller; pass a ready-to-display model. |
| **Use layouts for shared structure** | Avoid duplicating headers/footers. | `_Layout.cshtml` with `@RenderBody()` |
| **Use partials for reusable fragments** | DRY principle; easier maintenance. | `_ProductRow.cshtml` |
| **Use view components for logic-bearing UI** | Encapsulates logic and markup. | `@await Component.InvokeAsync("CartSummary")` |
| **Use Tag Helpers over HTML Helpers** | Cleaner, HTML-friendly syntax. | `<a asp-action="Index">` instead of `@Html.ActionLink(...)` |
| **Import shared directives in `_ViewImports.cshtml`** | Avoid repetition; consistent Tag Helpers. | `@addTagHelper *, Microsoft.AspNetCore.Mvc.TagHelpers` |
| **Avoid `@Html.Raw` with user input** | XSS risk. | Use `@Model.UserContent` (encoded) or sanitize first. |
| **Use `asp-append-version` for assets** | Cache busting after file changes. | `<link href="~/css/site.css" asp-append-version="true" />` |
| **Keep view names consistent** | Predictable discovery. | `Index.cshtml`, `Details.cshtml`, `_Partial.cshtml` |
| **Prefer feature folders for large apps** | Higher cohesion, less navigation friction. | `/Features/Products/Views/...` |

#### Constraints and Limitations

- **`ViewBag`/`ViewData` are dynamic** — no compile-time checking; prefer `@model`.
- **`@Html.Raw` is dangerous** — only use with trusted, sanitized content.
- **Runtime compilation overhead** — if enabled, views compile at first request; consider precompilation for production.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Strongly Typed View with Tag Helpers

```cshtml
@* Views/Products/Details.cshtml *@
@model ProductDetailsViewModel

<h1>@Model.Name</h1>
<p>@Model.Description</p>
<p>Price: @Model.Price.ToString("C")</p>

<a asp-controller="Products" asp-action="Edit" asp-route-id="@Model.Id" class="btn btn-primary">Edit</a>
<a asp-controller="Products" asp-action="Index" class="btn btn-secondary">Back</a>
```

**Step-by-step explanation:**

1. `@model ProductDetailsViewModel` provides compile-time safety.
2. Razor expressions (`@Model.Name`) are HTML-encoded by default.
3. `asp-controller`, `asp-action`, and `asp-route-id` Tag Helpers generate URLs and class attributes.

**Expected Output:** A product detail page with encoded user data and correctly generated links.

**Why this output?** Strong typing enables compile-time checking; Tag Helpers ensure URLs remain valid across route changes.

---

#### Example 2: Reusable Partial for Product Rows

```cshtml
@* Views/Products/_ProductRow.cshtml *@
@model Product

<tr>
    <td>@Model.Name</td>
    <td>@Model.Price.ToString("C")</td>
    <td>@(Model.Stock > 0 ? "In Stock" : "Out of Stock")</td>
</tr>
```

```cshtml
@* Views/Products/Index.cshtml *@
@model IEnumerable<Product>

<table>
    <thead>
        <tr><th>Name</th><th>Price</th><th>Status</th></tr>
    </thead>
    <tbody>
        @foreach (var product in Model)
        {
            <partial name="_ProductRow" model="product" />
        }
    </tbody>
</table>
```

**Expected Output:** A product table where each row is rendered by the partial view.

**Why this output?** The `<partial>` Tag Helper renders the partial with the specified model, promoting DRY and maintainability.

---

#### Example 3: Avoiding `@Html.Raw` with Untrusted Input

```cshtml
@* ❌ Bad: XSS risk *@
<div>@Html.Raw(Model.UserBio)</div>

@* ✅ Good: Razor encodes the value by default *@
<div>@Model.UserBio</div>

@* ✅ Acceptable: Only for trusted, sanitized HTML *@
<div>@Html.Raw(Model.TrustedHtmlFromSanitizer)</div>
```

**Expected Output:** The good version HTML-encodes the value, preventing script injection; the bad version would render untrusted markup.

**Why this output?** Razor's default encoding mitigates XSS; `Html.Raw` bypasses encoding and must only be used with sanitized content.

### Real-World Cases

- **E-commerce** – strongly typed product views with Tag Helpers for links and forms.
- **Admin dashboards** – view components for reusable widgets (cart summary, notifications).
- **Marketing sites** – layouts and partials for consistent branding.
- **Multi-tenant apps** – ViewModels shape tenant-specific data for the views.

### References

- Views in ASP.NET Core MVC – https://learn.microsoft.com/en-us/aspnet/core/mvc/views/overview
- Layout in ASP.NET Core – https://learn.microsoft.com/en-us/aspnet/core/mvc/views/layout
- Partial views in ASP.NET Core – https://learn.microsoft.com/en-us/aspnet/core/mvc/views/partial
- View components in ASP.NET Core – https://learn.microsoft.com/en-us/aspnet/core/mvc/views/view-components
- Tag Helpers in ASP.NET Core – https://learn.microsoft.com/en-us/aspnet/core/mvc/views/tag-helpers/intro
- Razor syntax reference – https://learn.microsoft.com/en-us/aspnet/core/mvc/views/razor
- Prevent Cross-Site Scripting (XSS) in ASP.NET Core – https://learn.microsoft.com/en-us/aspnet/core/security/cross-site-scripting

---

## Summary Table: View Fundamentals at a Glance

| Concept | Primary Purpose | Key Takeaway | When to Use |
|---------|----------------|--------------|-------------|
| **Views** | Render HTML from models | Razor templates with `@model`, `@` expressions, Tag Helpers | Every user-facing page |
| **View Discovery** | Find the right `.cshtml` file | `/Views/{Controller}/{Action}.cshtml` → `/Views/Shared/` | Every `ViewResult` execution |
| **View Locations** | Define search paths | `ViewLocationFormats`, `AreaViewLocationFormats`; expanders for custom layouts | Any custom folder structure |
| **View Naming** | Convention for auto-discovery | `Index.cshtml`, `Details.cshtml`, `_Partial.cshtml`; explicit names via `View("...")` | Every view file |
| **View Best Practices** | Maintainability, security, reuse | `@model`, Tag Helpers, partials, layouts, avoid `Html.Raw` with user input | All production views |

---

## References (Full List)

- Views in ASP.NET Core MVC – https://learn.microsoft.com/en-us/aspnet/core/mvc/views/overview
- Razor syntax reference – https://learn.microsoft.com/en-us/aspnet/core/mvc/views/razor
- Layout in ASP.NET Core – https://learn.microsoft.com/en-us/aspnet/core/mvc/views/layout
- Partial views in ASP.NET Core – https://learn.microsoft.com/en-us/aspnet/core/mvc/views/partial
- View components in ASP.NET Core – https://learn.microsoft.com/en-us/aspnet/core/mvc/views/view-components
- Tag Helpers in ASP.NET Core – https://learn.microsoft.com/en-us/aspnet/core/mvc/views/tag-helpers/intro
- Custom view location with `IViewLocationExpander` – https://learn.microsoft.com/en-us/aspnet/core/mvc/views/overview#custom-view-location-expander
- Razor view compilation – https://learn.microsoft.com/en-us/aspnet/core/mvc/views/view-compilation
- `RazorViewEngineOptions` API – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.razor.razorviewengineoptions
- `IViewLocationExpander` API – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.razor.iviewlocationexpander
- Preventing XSS in ASP.NET Core – https://learn.microsoft.com/en-us/aspnet/core/security/cross-site-scripting
- Razor Class Libraries – https://learn.microsoft.com/en-us/aspnet/core/razor-pages/ui-class