# ASP.NET Core Razor Pages: A Comprehensive Cheat Sheet

> **Scope & Version Note:** This cheat sheet focuses on **ASP.NET Core Razor Pages** (the page‑focused UI framework built on ASP.NET Core MVC). Examples target .NET 6–9 with minimal hosting (`Program.cs`). Razor Pages was introduced in ASP.NET Core 2.0; earlier ASP.NET MVC 5 did not have an equivalent feature. Where behaviour differs across versions, it is noted. Razor Pages is **not deprecated** — it is a supported, first‑class UI option alongside MVC and Blazor.

---

## Topic Overview

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Razor Pages is a page‑based programming model for building server‑rendered web UI in ASP.NET Core, where each page is a self‑contained unit consisting of a `.cshtml` view and an optional `.cshtml.cs` page model. |
| **Technical Definition** | Razor Pages is built on ASP.NET Core MVC. The `@page` directive turns a Razor view (`.cshtml`) into an MVC action that handles requests **directly**, without going through a controller. Pages are discovered from the `Pages` folder by default; the page’s file‑system path determines its URL. Handler methods on the `PageModel` class (`OnGet`, `OnPost`, `OnPost[Handler]Async`, etc.) process requests. Routing, model binding, validation, Tag Helpers, and HTML Helpers all function within the page model, providing the same MVC primitives with a page‑centric workflow. |
| **Beginner‑Friendly Explanation** | Think of Razor Pages as **self‑contained web pages**. Each page has two parts: an HTML template (`.cshtml`) and a small C# “code‑behind” file (`.cshtml.cs`) that controls what happens when the page loads or when a form is submitted. Unlike MVC, there is no separate controller — the page itself handles the request. This makes page‑focused scenarios like forms, dashboards, and content pages simpler and more productive. |

### Key Characteristics

- **Page‑centric** – Each page owns its logic, markup, and handlers. No shared controller is required.
- **`@page` directive** – Makes a `.cshtml` file an MVC action that handles requests directly.
- **Convention‑based routing** – URLs are derived from the file‑system path under the `Pages` folder: `/Pages/Index.cshtml` → `/` or `/Index`; `/Pages/Store/Contact.cshtml` → `/Store/Contact`.
- **PageModel separation** – Logic lives in a `<PageName>Model` class (`.cshtml.cs`), enabling dependency injection and unit testing.
- **Handler methods** – `OnGet`, `OnPost`, and named handlers (`OnPostDeleteAsync`, etc.) process HTTP verbs.
- **Antiforgery by default** – Razor Pages are protected by antiforgery validation.
- **Full Razor view‑engine support** – Layouts, partials, Tag Helpers, and HTML Helpers all work.

### Prerequisites

- **.NET SDK** (6.0, 7.0, 8.0, or 9.0 recommended).
- **ASP.NET Core** workload (`Microsoft.AspNetCore.App`).
- Basic **C#** and **HTML** knowledge.
- Familiarity with **Razor syntax** (`@` expressions, directives) and **HTTP verbs** (GET, POST).
- **Entity Framework Core** (for the database‑backed examples; optional for understanding routing and handlers).

### Related Programming Areas

- **ASP.NET Core MVC** – Razor Pages shares routing, model binding, validation, filters, and Tag Helpers with MVC.
- **Razor view engine** – The `.cshtml` syntax and Tag Helpers are identical to MVC views.
- **Tag Helpers & HTML Helpers** – Form generation, validation, and link generation work the same way in Razor Pages.
- **Endpoint routing** – Razor Pages uses the same endpoint routing infrastructure as MVC and minimal APIs.
- **Antiforgery / CSRF protection** – Razor Pages are protected by default.

### Core Concepts / Features

1. **Razor Pages** – The page‑based programming model and its file structure.
2. **Page Models** – The `PageModel` class (`<PageName>Model`) and its role.
3. **Handlers** – `OnGet`, `OnPost`, named handlers, and async conventions.
4. **Routing** – File‑based routing, `@page` templates, and route conventions.
5. **Razor Pages Best Practices** – Security, performance, maintainability, and testing guidelines.

Each concept is elaborated using a **uniform structure**: Definitions, Purposes, Syntax Rules & Structure, Annotated Code Examples, Real‑World Cases, and References.

---

## 1. Razor Pages (The Programming Model)

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Razor Pages is a page‑focused UI framework in ASP.NET Core where each page is a self‑contained unit of markup and logic. |
| **Technical Definition** | A Razor Page is a `.cshtml` file with an `@page` directive. The directive makes the file an MVC action that handles requests directly, bypassing controllers. The page optionally has a `.cshtml.cs` code‑behind file containing a `PageModel` class. Pages are discovered from the `Pages` folder; the file‑system path becomes the route template. `AddRazorPages()` registers services, and `MapRazorPages()` adds endpoints. |
| **Beginner‑Friendly Explanation** | A Razor Page is a **single web page** with its own HTML and C# logic. If you need a contact form, you create `Contact.cshtml` (the form) and `Contact.cshtml.cs` (what happens when the form is submitted). There is no separate controller — the page handles everything itself. |

### Purposes

- **To** simplify page‑focused web development by eliminating the controller‑view indirection.
- **To** keep markup and logic close together while still allowing separation through the `PageModel`.
- **To** provide a productive alternative to MVC for form‑heavy, content‑driven, and CRUD‑style applications.
- **To** leverage the full power of ASP.NET Core (model binding, validation, Tag Helpers, filters, dependency injection) in a page‑centric workflow.
- **To** enable unit testing of page logic through the `PageModel` class.

### Syntax Rules and Structure

#### General Syntax (Project Setup in `Program.cs`)

```csharp
var builder = WebApplication.CreateBuilder(args);
builder.Services.AddRazorPages();          // Register Razor Pages services

var app = builder.Build();

app.UseRouting();
app.MapRazorPages();                        // Add Razor Pages endpoints
app.Run();
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `AddRazorPages()` | Registers services required by Razor Pages (model binding, validation, antiforgery, etc.). |
| `MapRazorPages()` | Adds endpoints for all discovered Razor Pages. |
| `UseRouting()` | Enables endpoint routing (usually already added by `WebApplication`). |

#### General Syntax (A Basic Razor Page)

```cshtml
@page
@model IndexModel

<h1>Hello, Razor Pages!</h1>
<p>Server time: @DateTime.Now</p>
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `@page` | **Required first directive** – makes the file an MVC action that handles requests directly. |
| `@model IndexModel` | Associates the page with its `PageModel` class. |
| HTML + Razor | Markup and C# expressions (`@DateTime.Now`). |

#### General Syntax (Page + PageModel Pair)

```cshtml
@page
@model IndexModel
<p>@Model.Message</p>
```

```csharp
// Index.cshtml.cs
public class IndexModel : PageModel
{
    public string Message { get; private set; } = "PageModel in C#";
    public void OnGet()
    {
        Message += $" Server time is {DateTime.Now}";
    }
}
```

**Component breakdown:**

| File | Role |
|------|------|
| `Index.cshtml` | Markup; `@model` points to `IndexModel`. |
| `Index.cshtml.cs` | `PageModel` class; properties are accessible from the view; `OnGet` initialises state. |

#### Syntax Rules

- `@page` **must be the first Razor directive** on the page.
- The `PageModel` class is conventionally named `<PageName>Model` and placed in the same namespace as the page.
- The code‑behind file is named `<PageName>.cshtml.cs` by convention.
- The runtime looks for pages in the `Pages` folder by default; `Index` is the default page.

#### Constraints and Limitations

- Razor Pages are **not a replacement for all MVC scenarios**; large API‑centric applications may still benefit from controllers.
- `@page` cannot be omitted; without it, the file is treated as a regular Razor view, not a page.
- The `Pages` folder location can be customised, but most applications use the default.

### Annotated Complete Step‑by‑Step Code Examples

#### Example 1: Minimal Razor Page with Code‑Behind

**Setup:** Create a project (`dotnet new webapp`). Replace `Pages/Index.cshtml` and `Pages/Index.cshtml.cs`:

```cshtml
@* Pages/Index.cshtml *@
@page
@model IndexModel

<h1>Welcome</h1>
<p>@Model.Greeting</p>
```

```csharp
// Pages/Index.cshtml.cs
using Microsoft.AspNetCore.Mvc.RazorPages;

public class IndexModel : PageModel
{
    public string Greeting { get; private set; } = "";

    public void OnGet()
    {
        Greeting = $"Hello! The time is {DateTime.Now:HH:mm}.";
    }
}
```

**Step‑by‑step explanation:**

1. `@page` makes the file a Razor Page.
2. `@model IndexModel` links the view to the page model.
3. `OnGet` runs when the page is requested via GET; it sets `Greeting`.
4. The view renders `@Model.Greeting`.

**Expected Output (at 14:30):**

```html
<h1>Welcome</h1>
<p>Hello! The time is 14:30.</p>
```

**Why this output?** The `OnGet` handler runs before the page renders and populates the `Greeting` property, which the view then displays.

---

#### Example 2: Razor Page with a Database‑Backed List

```cshtml
@* Pages/Customers/Index.cshtml *@
@page
@model Customers.IndexModel
@addTagHelper *, Microsoft.AspNetCore.Mvc.TagHelpers

<h1>Customers</h1>
<ul>
    @foreach (var customer in Model.Customers)
    {
        <li>@customer.Name</li>
    }
</ul>
```

```csharp
// Pages/Customers/Index.cshtml.cs
using Microsoft.AspNetCore.Mvc.RazorPages;
using Microsoft.EntityFrameworkCore;

namespace MyApp.Pages.Customers
{
    public class IndexModel : PageModel
    {
        private readonly CustomerDbContext _context;
        public IndexModel(CustomerDbContext context) => _context = context;

        public IList<Customer> Customers { get; private set; } = new List<Customer>();

        public async Task OnGetAsync()
        {
            Customers = await _context.Customers.ToListAsync();
        }
    }
}
```

**Step‑by‑step explanation:**

1. The page model injects `CustomerDbContext` via constructor injection.
2. `OnGetAsync` is the asynchronous GET handler; it loads customers from the database.
3. The view iterates `Model.Customers` and renders each name.

**Expected Output:**

```html
<h1>Customers</h1>
<ul>
    <li>Alice</li>
    <li>Bob</li>
</ul>
```

**Why this output?** `OnGetAsync` retrieves the customer list before rendering; the view loops over the populated `Customers` property.

### Real‑World Cases

- **Contact forms and surveys:** A page model handles validation and submission without a controller.
- **Product catalogues:** `/Pages/Products/Index.cshtml` lists products; `/Pages/Products/Details.cshtml` shows one product via a route parameter.
- **Admin dashboards:** Page‑focused CRUD screens benefit from the co‑location of logic and markup.

### References

- Razor Pages architecture and concepts – https://learn.microsoft.com/en-us/aspnet/core/razor-pages/
- Introduction to Razor Pages – https://learn.microsoft.com/en-us/aspnet/core/razor-pages/?view=aspnetcore-9.0
- Choose an ASP.NET Core UI – https://learn.microsoft.com/en-us/aspnet/core/tutorials/choose-web-ui

---

## 2. Page Models

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | A PageModel is a C# class that contains the logic, data, and handlers for a Razor Page. |
| **Technical Definition** | The `PageModel` class (in `Microsoft.AspNetCore.Mvc.RazorPages`) is the code‑behind for a Razor Page. It exposes properties to the view, defines handler methods (`OnGet`, `OnPost`, etc.), and supports constructor dependency injection. By convention, the class is named `<PageName>Model` and resides in the same namespace as the page. |
| **Beginner‑Friendly Explanation** | The PageModel is the **“brains” of the page**. It holds the data the page needs and decides what happens when the page loads or when a form is submitted. It is separate from the HTML so the logic can be tested without rendering the page. |

### Purposes

- **To** separate page logic from presentation markup.
- **To** manage page dependencies through dependency injection.
- **To** enable unit testing of page behaviour without a browser or HTTP pipeline.
- **To** expose data to the view through public properties.
- **To** define handler methods for different HTTP verbs and named operations.

### Syntax Rules and Structure

#### General Syntax

```csharp
public class <PageName>Model : PageModel
{
    // Dependencies via constructor injection
    public <PageName>Model(<Dependency> dependency) { ... }

    // Properties exposed to the view
    public <Type> <PropertyName> { get; set; }

    // Handler methods
    public void OnGet() { ... }
    public IActionResult OnPost() { ... }
}
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `: PageModel` | Inherits from the base `PageModel` class. |
| Constructor | Receives services via dependency injection. |
| Public properties | Available in the view as `@Model.PropertyName`. |
| `OnGet` / `OnPost` | Handler methods (see Section 3). |

#### Syntax Rules

- The class **must inherit from `PageModel`**.
- The class name conventionally ends with `Model` and matches the page file name.
- Properties must be **public** to be accessible from the view.
- The code‑behind file is named `<PageName>.cshtml.cs`.

#### Constraints and Limitations

- The PageModel is **instantiated per request** — do not store per‑user state in instance fields across requests.
- Properties are **not automatically bound** from the request unless marked with `[BindProperty]` (see Section 3).
- **Over‑posting risk:** `[BindProperty]` should not be used on models containing properties that clients must not change.

### Annotated Code Examples

#### Example 1: PageModel with Injected Service and View Property

```csharp
// Pages/Products/Index.cshtml.cs
public class IndexModel : PageModel
{
    private readonly IProductService _service;

    public IndexModel(IProductService service) => _service = service;

    public IReadOnlyList<Product> Products { get; private set; } = Array.Empty<Product>();

    public async Task OnGetAsync()
    {
        Products = await _service.GetAllAsync();
    }
}
```

**Expected Output (in the view):**

```cshtml
@foreach (var p in Model.Products)
{
    <p>@p.Name</p>
}
```

**Why this output?** The `PageModel` injects `IProductService`, retrieves products in `OnGetAsync`, and exposes them through the `Products` property for the view to render.

---

#### Example 2: PageModel with `[BindProperty]` for Form Input

```csharp
public class CreateModel : PageModel
{
    private readonly CustomerDbContext _context;
    public CreateModel(CustomerDbContext context) => _context = context;

    [BindProperty]
    public Customer Customer { get; set; }

    public IActionResult OnGet() => Page();

    public async Task<IActionResult> OnPostAsync()
    {
        if (!ModelState.IsValid) return Page();

        _context.Customers.Add(Customer);
        await _context.SaveChangesAsync();
        return RedirectToPage("./Index");
    }
}
```

**Expected Output:** When the form is posted with valid data, the customer is saved and the user is redirected to the Index page. If validation fails, the page is re‑rendered with error messages.

**Why this output?** `[BindProperty]` enables model binding from the POST body to the `Customer` property. `ModelState.IsValid` checks data annotations. `RedirectToPage` performs a PRG (Post‑Redirect‑Get) redirect.

### Real‑World Cases

- **CRUD pages:** Page models handle create, read, update, and delete operations with injected `DbContext`.
- **Search pages:** A `[BindProperty(SupportsGet = true)]` property captures the search term from the query string.
- **Unit‑tested page logic:** Because the page model is a plain class, its handlers can be tested with mocked dependencies.

### References

- PageModel Class – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.razorpages.pagemodel
- Razor Pages architecture and concepts – https://learn.microsoft.com/en-us/aspnet/core/razor-pages/

---

## 3. Handlers

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Handlers are methods on the PageModel that process incoming HTTP requests for a Razor Page. |
| **Technical Definition** | Handler methods follow the naming convention `On<HTTP Verb>` or `On<HTTP Verb><HandlerName>` (optionally with an `Async` suffix). The runtime selects the handler based on the HTTP method and, for named handlers, the `handler` query‑string or form value (e.g., `asp-page-handler="delete"` → `OnPostDeleteAsync`). Common handlers are `OnGet` (initialise state) and `OnPost` (process form submissions). |
| **Beginner‑Friendly Explanation** | Handlers are **what the page does when someone interacts with it**. When you open a page, `OnGet` runs. When you submit a form, `OnPost` runs. If a page has multiple buttons (e.g., “Save” and “Delete”), each button can trigger a **named handler** like `OnPostSave` or `OnPostDelete`. |

### Purposes

- **To** process HTTP GET requests and initialise page state (`OnGet`).
- **To** handle form submissions and other POST requests (`OnPost`).
- **To** support multiple operations on a single page through named handlers (`OnPostDeleteAsync`, `OnPostSaveAsync`, etc.).
- **To** return appropriate action results (`Page()`, `RedirectToPage()`, `NotFound()`, etc.).
- **To** enable asynchronous processing of I/O‑bound operations (`OnGetAsync`, `OnPostAsync`).

### Syntax Rules and Structure

#### General Syntax

```
On<Verb>()                    // synchronous, no handler name
On<Verb>Async()               // asynchronous
On<Verb><HandlerName>()       // named synchronous handler
On<Verb><HandlerName>Async()  // named asynchronous handler
```

**Component breakdown:**

| Part | Meaning |
|------|---------|
| `On` | Required prefix. |
| `<Verb>` | HTTP verb: `Get`, `Post`, `Put`, `Delete`, `Patch`, etc. |
| `<HandlerName>` | Optional name matching `asp-page-handler` or the `handler` query string. |
| `Async` | Optional suffix indicating an asynchronous method (convention). |

#### Syntax Rules

- The method must be **public** (or `public async Task` for async handlers).
- Handler names are **case‑insensitive** in matching but conventionally Pascal‑cased.
- The `Async` suffix is **optional** but strongly recommended for asynchronous methods.
- A handler returning `void` or `Task` renders the page; returning `IActionResult` allows redirection, status codes, etc..
- `HEAD` requests fall back to the `OnGet` handler.

#### Constraints and Limitations

- **Named handlers** require the `handler` value to be supplied (e.g., via `asp-page-handler`). Without it, only the unnamed `OnPost` is considered.
- **GET binding is opt‑in:** By default, `[BindProperty]` does **not** bind on GET requests. Use `[BindProperty(SupportsGet = true)]` explicitly, and validate user input carefully for security reasons.
- **Over‑posting:** Avoid `[BindProperty]` on models with sensitive properties that clients should not modify.

### Annotated Complete Step‑by‑Step Code Examples

#### Example 1: Basic `OnGet` and `OnPost`

```csharp
public class ContactModel : PageModel
{
    [BindProperty]
    public string Name { get; set; }

    public string Message { get; private set; } = "";

    public void OnGet()
    {
        Message = "Please enter your name.";
    }

    public IActionResult OnPost()
    {
        if (string.IsNullOrWhiteSpace(Name))
        {
            ModelState.AddModelError(nameof(Name), "Name is required.");
            return Page();
        }
        return RedirectToPage("./ThankYou");
    }
}
```

**Step‑by‑step explanation:**

1. `OnGet` runs on GET and sets an introductory message.
2. The form posts back; `OnPost` runs.
3. If `Name` is empty, a validation error is added and the page is re‑rendered.
4. If valid, the user is redirected to a thank‑you page.

**Expected Output:** On GET, the page shows the message. On POST with an empty name, the page re‑renders with a validation error. On POST with a valid name, the browser is redirected.

**Why this output?** Handler methods are selected by HTTP verb. `ModelState` carries validation errors back to the view.

---

#### Example 2: Named Handlers with Multiple Buttons

```cshtml
<form method="post">
    <input asp-for="Name" />
    <button type="submit" asp-page-handler="Save">Save</button>
    <button type="submit" asp-page-handler="Delete">Delete</button>
</form>
```

```csharp
public IActionResult OnPostSave()
{
    // Save logic
    return RedirectToPage("./Index");
}

public async Task<IActionResult> OnPostDeleteAsync(int id)
{
    // Delete logic
    await _service.DeleteAsync(id);
    return RedirectToPage("./Index");
}
```

**Step‑by‑step explanation:**

1. The “Save” button submits with `handler=Save`; `OnPostSave` is invoked.
2. The “Delete” button submits with `handler=Delete`; `OnPostDeleteAsync` is invoked.
3. Both handlers return a redirect after completing their operation.

**Expected Output:** Clicking “Save” runs the save handler; clicking “Delete” runs the delete handler. Both redirect to the Index page.

**Why this output?** The `asp-page-handler` attribute populates the `handler` form value. The runtime matches it to the corresponding named handler using the `OnPost[Handler]Async` convention.

---

#### Example 3: Asynchronous Handler with Database Access

```csharp
public async Task<IActionResult> OnPostAsync()
{
    if (!ModelState.IsValid) return Page();

    _context.Customers.Add(Customer);
    await _context.SaveChangesAsync();
    return RedirectToPage("./Index");
}
```

**Expected Output:** Valid submissions are saved asynchronously and the user is redirected. Invalid submissions re‑render the page with errors.

**Why this output?** `OnPostAsync` performs async I/O without blocking the request thread. The `Async` suffix follows convention and signals asynchronous behaviour.

### Real‑World Cases

- **Multi‑action pages:** A single admin page with “Save”, “Publish”, and “Delete” buttons uses three named handlers.
- **Search and filter pages:** `OnGetAsync` reads query‑string parameters via `[BindProperty(SupportsGet = true)]` and loads filtered data.
- **File uploads:** `OnPostAsync` processes `IFormFile` from the posted form.

### References

- Handler methods in Razor Pages – https://learn.microsoft.com/en-us/aspnet/core/razor-pages/#handler-methods
- PageModel.RedirectToPage – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.razorpages.pagemodel.redirecttopage
- [BindProperty] attribute – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.bindpropertyattribute

---

## 4. Routing

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Razor Pages routing maps incoming URL paths to page files based on their location in the `Pages` folder. |
| **Technical Definition** | By default, the route template for a Razor Page is its file‑system path relative to the `Pages` folder, without the `.cshtml` extension. `Index.cshtml` maps to `/` (and `/Index`); `Store/Contact.cshtml` maps to `/Store/Contact`. The `@page` directive can include a route template (e.g., `@page "{id:int}"`) to add parameters or override the default path. Route conventions can be configured globally via `RazorPagesOptions.Conventions`. |
| **Beginner‑Friendly Explanation** | Routing for Razor Pages is **automatic**. If you create a file at `Pages/Products/Details.cshtml`, the URL `/Products/Details` will show that page. You can customise the URL by adding a route template after `@page`, such as `@page "{id:int}"` to accept `/Products/Details/5`. |

### Purposes

- **To** automatically map URLs to page files without manual route registration.
- **To** allow custom route templates for parameters, optional segments, and constraints.
- **To** support global route conventions (prefixes, custom routes) across all pages or folders.
- **To** enable URL generation (`RedirectToPage`, `asp-page`) that stays consistent with routing.
- **To** integrate with endpoint routing and middleware (authorization, CORS) through endpoint metadata.

### Syntax Rules and Structure

#### General Syntax (Default File‑Based Routing)

```
/Pages/Index.cshtml          →  /  or  /Index
/Pages/Contact.cshtml        →  /Contact
/Pages/Store/Contact.cshtml  →  /Store/Contact
/Pages/Store/Index.cshtml    →  /Store  or  /Store/Index
```

**Component breakdown:**

| File path | Matching URL |
|-----------|--------------|
| `Pages/Index.cshtml` | `/` or `/Index` |
| `Pages/Contact.cshtml` | `/Contact` |
| `Pages/Store/Contact.cshtml` | `/Store/Contact` |
| `Pages/Store/Index.cshtml` | `/Store` or `/Store/Index` |

*Source: official documentation table.*

#### General Syntax (Custom Route Template)

```cshtml
@page "{id:int}"
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `@page` | Required directive. |
| `"{id:int}"` | Route template with an `int` constraint; the page accepts URLs like `/Edit/5`. |

#### General Syntax (Global Route Convention)

```csharp
builder.Services.AddRazorPages(options =>
{
    options.Conventions.AddPageRoute("/Contact", "TheContactPage/{text?}");
});
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `AddRazorPages(options => ...)` | Configures `RazorPagesOptions`. |
| `Conventions.AddPageRoute` | Adds an additional route template for a specific page. |
| `"/Contact"` | The page path. |
| `"TheContactPage/{text?}"` | The custom route template. |

#### Syntax Rules

- `@page` must be the first Razor directive.
- Route templates are placed **after** `@page` in double quotes.
- Route constraints (`:int`, `:guid`, etc.) are supported and behave as in MVC routing.
- To make a route parameter optional, append `?` (e.g., `{id:int?}`).
- **Route order:** Routes are processed by `Order`; if `Order` is not set, the most specific route matches first. Avoid relying on `Order` in production if possible.

#### Constraints and Limitations

- **Reserved words** cannot be used as route segments or parameter names.
- The default `Pages` root can be changed (`WithRazorPagesAtContentRoot`, `WithRazorPagesRoot`), but this is rarely necessary.
- **Ambiguous routes** can occur if multiple pages share the same effective route; conventions can resolve this.
- Route templates in `@page` **override** the default file‑based route for that page.

### Annotated Complete Step‑by‑Step Code Examples

#### Example 1: Default Routing from File Structure

**Files:**

```
Pages/Index.cshtml
Pages/About.cshtml
Pages/Products/Index.cshtml
Pages/Products/Details.cshtml
```

**Expected Output (URL mappings):**

| URL | Page |
|-----|------|
| `/` or `/Index` | `Pages/Index.cshtml` |
| `/About` | `Pages/About.cshtml` |
| `/Products` or `/Products/Index` | `Pages/Products/Index.cshtml` |
| `/Products/Details` | `Pages/Products/Details.cshtml` |

**Why this output?** The routing system derives the route from the file‑system path relative to `Pages`, without the extension. `Index` is the default page for its folder.

---

#### Example 2: Custom Route with Constraint

```cshtml
@page "{id:int}"
@model EditModel
```

**Expected Output:**

| URL | Result |
|-----|--------|
| `/Edit/5` | `id = 5` is bound; page renders. |
| `/Edit/abc` | **404** (the `int` constraint fails). |
| `/Edit` | **404** (parameter is required). |

**Why this output?** The `@page "{id:int}"` template replaces the default route and applies the `int` constraint. Non‑integer or missing values do not match.

---

#### Example 3: Global Route Convention

```csharp
builder.Services.AddRazorPages(options =>
{
    options.Conventions.AddPageRoute("/Contact", "support/contact-us");
});
```

**Expected Output:** The `Pages/Contact.cshtml` page is accessible at both `/Contact` (default) and `/support/contact-us` (custom route).

**Why this output?** `AddPageRoute` adds an **additional** route template for the page, preserving the default route. This is useful for SEO‑friendly aliases.

### Real‑World Cases

- **E‑commerce product pages:** `@page "{id:int}"` on `Products/Details.cshtml` yields `/Products/Details/42`.
- **SEO‑friendly URLs:** A route convention maps `/About` to `/about-us`.
- **Localised sites:** Route conventions add a `{culture}` prefix to all pages.

### References

- Razor Pages route and app conventions – https://learn.microsoft.com/en-us/aspnet/core/razor-pages/razor-pages-conventions
- Routing in ASP.NET Core – https://learn.microsoft.com/en-us/aspnet/core/fundamentals/routing
- Razor Pages architecture and concepts (URL matching table) – https://learn.microsoft.com/en-us/aspnet/core/razor-pages/#razor-pages

---

## 5. Razor Pages Best Practices

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Best practices are guidelines that help developers build Razor Pages applications that are secure, performant, testable, and maintainable. |
| **Technical Definition** | These practices span security (antiforgery, over‑posting prevention), performance (async handlers, response compression), architecture (thin page models, separation of concerns), and testing (unit‑testing page models with mocked dependencies). |
| **Beginner‑Friendly Explanation** | Best practices are the **“do’s and don’ts”** that experienced developers follow to avoid common bugs, security holes, and messy code when building Razor Pages apps. |

### Purposes

- **To** protect against CSRF and over‑posting vulnerabilities.
- **To** keep page models thin and focused on presentation logic.
- **To** ensure handlers are asynchronous for I/O‑bound work.
- **To** enable unit testing through dependency injection and separation of concerns.
- **To** maintain consistent routing and URL generation.
- **To** optimise response performance through middleware ordering and compression.

### Syntax Rules and Structure (Guideline Checklist)

| Area | Do | Don’t |
|------|----|-------|
| **Antiforgery** | Rely on Razor Pages’ default antiforgery protection. | Disable antiforgery without a compelling reason. |
| **Over‑posting** | Use view models or DTOs for `[BindProperty]`; avoid binding domain entities with sensitive fields. | Use `[BindProperty]` on entities that contain `IsAdmin`, `Role`, or other protected properties. |
| **GET binding** | Use `[BindProperty(SupportsGet = true)]` sparingly and validate input. | Blindly bind GET query strings to sensitive properties. |
| **Async handlers** | Use `OnGetAsync` / `OnPostAsync` for database and network calls. | Use synchronous handlers for I/O‑bound operations. |
| **Page model size** | Keep page models focused; move complex business logic to services. | Put hundreds of lines of business logic in a page model. |
| **Testing** | Unit‑test page models by instantiating them with mocked dependencies. | Test only through integration tests when unit tests are possible. |
| **Routing** | Use relative page names (`./Index`) for redirects within a folder. | Hard‑code URLs that break when routes change. |
| **Middleware order** | Place `UseStaticFiles` before `UseRouting`; add `UseResponseCompression` after routing. | Misorder middleware so static files are not compressed or routed correctly. |
| **`Order` in routes** | Rely on route specificity; avoid explicit `Order` unless necessary. | Use `Order` as a substitute for clear route design. |

### Constraints and Limitations

- **Antiforgery** protects POST requests by default but requires the token to be present; Tag Helpers and `Html.AntiForgeryToken()` handle this automatically.
- **`[BindProperty]`** is **not** a substitute for validation; always check `ModelState.IsValid`.
- **Unit testing** page models requires mocking `HttpContext` and other abstractions when handlers access request data.
- **Response compression** must be placed correctly in the middleware pipeline to affect Razor Pages responses.

### Annotated Code Example (Demonstrating Multiple Best Practices)

```csharp
// Pages/Customers/Create.cshtml.cs
public class CreateModel : PageModel
{
    private readonly ICustomerService _service;

    public CreateModel(ICustomerService service) => _service = service;

    [BindProperty]
    public CustomerInputModel Input { get; set; }   // DTO prevents over-posting

    public IActionResult OnGet() => Page();

    public async Task<IActionResult> OnPostAsync()
    {
        if (!ModelState.IsValid) return Page();      // Validate before processing

        await _service.CreateAsync(Input);           // Async I/O
        return RedirectToPage("./Index");           // Relative redirect
    }
}

public class CustomerInputModel
{
    [Required, StringLength(50)]
    public string Name { get; set; }
}
```

```cshtml
@* Pages/Customers/Create.cshtml *@
@page
@model CreateModel
<form method="post">
    <input asp-for="Input.Name" />
    <span asp-validation-for="Input.Name"></span>
    <button type="submit">Create</button>
</form>
```

**Why this follows best practices:**

- `CustomerInputModel` is a DTO, preventing over‑posting of sensitive properties.
- `OnPostAsync` is asynchronous for I/O‑bound database work.
- `ModelState.IsValid` gates processing.
- `RedirectToPage("./Index")` uses a relative page name, robust to folder renames.
- Antiforgery token is added automatically by the Form Tag Helper.

### Real‑World Cases

- **Enterprise CRUD apps:** DTOs and service layers keep page models thin and testable.
- **High‑traffic pages:** Async handlers and response compression improve throughput.
- **Security‑sensitive forms:** Default antiforgery and explicit validation prevent CSRF and injection attacks.

### References

- Razor Pages architecture and concepts (best practices) – https://learn.microsoft.com/en-us/aspnet/core/razor-pages/
- Razor Pages route and app conventions – https://learn.microsoft.com/en-us/aspnet/core/razor-pages/razor-pages-conventions
- ASP.NET Core middleware (order and security) – https://learn.microsoft.com/en-us/aspnet/core/fundamentals/middleware
- Prevent Cross-Site Request Forgery (XSRF/CSRF) attacks – https://learn.microsoft.com/en-us/aspnet/core/security/anti-request-forgery

---

## Summary Table: Core Concepts at a Glance

| Concept | Primary Role | Key Syntax | When to Use |
|---------|--------------|------------|-------------|
| **Razor Pages** | Page‑focused UI model | `@page`, `AddRazorPages()`, `MapRazorPages()` | Form‑heavy, content‑driven, CRUD apps |
| **Page Models** | Logic and data for a page | `class IndexModel : PageModel` | Every Razor Page with behaviour |
| **Handlers** | Process HTTP requests | `OnGet`, `OnPost`, `OnPostDeleteAsync` | Initialising state, processing forms, named actions |
| **Routing** | Map URLs to pages | File‑based by default; `@page "{id:int}"`; conventions | Custom URLs, parameters, SEO aliases |
| **Best Practices** | Security, performance, maintainability | Antiforgery, DTOs, async handlers, thin models | Every production Razor Pages app |

---

## References (Full List)

- Razor Pages architecture and concepts in ASP.NET Core – https://learn.microsoft.com/en-us/aspnet/core/razor-pages/
- Introduction to Razor Pages in ASP.NET Core – https://learn.microsoft.com/en-us/aspnet/core/razor-pages/?view=aspnetcore-9.0
- Razor Pages route and app conventions in ASP.NET Core – https://learn.microsoft.com/en-us/aspnet/core/razor-pages/razor-pages-conventions
- PageModel Class – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.razorpages.pagemodel
- [BindProperty] Attribute – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.bindpropertyattribute
- PageModel.RedirectToPage Method – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.razorpages.pagemodel.redirecttopage
- Routing in ASP.NET Core – https://learn.microsoft.com/en-us/aspnet/core/fundamentals/routing
- ASP.NET Core Middleware – https://learn.microsoft.com/en-us/aspnet/core/fundamentals/middleware
- Prevent Cross-Site Request Forgery (XSRF/CSRF) attacks – https://learn.microsoft.com/en-us/aspnet/core/security/anti-request-forgery
- Choose an ASP.NET Core UI – https://learn.microsoft.com/en-us/aspnet/core/tutorials/choose-web-ui