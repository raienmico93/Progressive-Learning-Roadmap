# C# Razor Pages — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
**Razor Pages** is a page-focused programming model in ASP.NET Core that makes building server-rendered web UI **easier and more productive than MVC** for page-based scenarios. Each page is a pair of files: a `.cshtml` file containing the Razor markup and a `.cshtml.cs` file (the **PageModel**) containing the page's handlers and data. Razor Pages is built on top of ASP.NET Core's routing and Razor view engine, but removes the controller/view separation in favour of a per-page model.

**Technical Definition**
Razor Pages is a feature of ASP.NET Core that enables page-based coding scenarios. Each Razor Page is a `.cshtml` file with an associated `PageModel` class (conventionally named `{PageName}Model` and stored in a `.cshtml.cs` file). The `PageModel` class is typically derived from `Microsoft.AspNetCore.Mvc.RazorPages.PageModel`. Page handlers are public methods named `OnGet`, `OnPost`, `OnGetAsync`, `OnPostAsync`, or `On{Handler}{Verb}` (e.g., `OnPostDelete`). Routing maps the page's file path relative to the `Pages` folder to its URL (e.g., `/Pages/Products/Index.cshtml` → `/Products/Index`). Razor Pages supports **model binding**, **validation**, **filters**, **tag helpers**, and **layouts**, and integrates fully with the rest of ASP.NET Core (DI, configuration, authentication, authorization).

**Beginner-Friendly Explanation**
Razor Pages is **“one page, one file pair, one job.”** In MVC, you have a controller that handles many actions, each rendering a view. In Razor Pages, each page (e.g., `Products/Index`) owns its own `.cshtml` markup and `.cshtml.cs` logic. The `PageModel` class holds the page's data and handlers—`OnGet` for loading the page, `OnPost` for handling form submission. This makes small pages **self-contained, easier to find, and easier to test** than hunting through controllers and view folders.

---

### Key Characteristics

- **Page-focused** — Each URL corresponds to a page with its own markup and logic.
- **Two-file pair** — `.cshtml` (markup) + `.cshtml.cs` (PageModel).
- **Handlers** — `OnGet`, `OnPost`, and named handlers (`OnPostDelete`, etc.).
- **File-based routing** — URL derived from the file path under `Pages/`.
- **Model binding and validation** — `[BindProperty]`, `ModelState`, DataAnnotations.
- **Tag helpers** — `<form asp-page>`, `<a asp-page>`, `<input asp-for>`.
- **Filters** — Page filters, model state filters, and result filters.
- **Layouts and partials** — Shared UI via `_Layout.cshtml` and partial pages.
- **Integration** — Full access to DI, configuration, auth, and middleware.
- **Unit-testable** — PageModel is a plain class; handlers can be tested without HTTP.

---

### Prerequisites

- Basic understanding of **ASP.NET Core fundamentals** (`Program.cs`, DI, middleware).
- Familiarity with **Razor syntax** (`@model`, `@foreach`, tag helpers).
- Knowledge of **HTTP** (GET, POST, status codes).
- Awareness of **DataAnnotations** for validation.
- Understanding of **MVC basics** (helpful for comparison).

---

### Related Programming Areas

- **ASP.NET Core MVC** — Razor Pages is built on the same infrastructure but with a page-centric model.
- **Razor view engine** — Shared markup syntax.
- **Tag helpers** — `<form>`, `<a>`, `<input>`, `<validation-summary>`.
- **Minimal APIs** — Alternative for HTTP APIs with even less ceremony.
- **Blazor** — Component-based UI alternative for interactive scenarios.

---

### Core Concepts / Features

1. Razor Pages (Overarching Concept)
2. Page Models (`PageModel` class)
3. Handlers (`OnGet`, `OnPost`, named handlers)
4. Routing (file-based, custom routes)
5. Razor Pages Best Practices

---

## 1. Razor Pages (Overarching Concept)

### Definitions

**Core Definition**
Razor Pages is a page-centric programming model where each web page is a self-contained unit with its own markup and logic.

**Technical Definition**
A Razor Page is a `.cshtml` file with a corresponding `PageModel` class. The page is mapped to a URL based on its file path under the `Pages` folder. The `PageModel` contains handler methods (`OnGet`, `OnPost`, etc.) that are invoked by the framework based on the HTTP verb and optional handler name. Pages support model binding, validation, layouts, partials, filters, and all other ASP.NET Core features.

**Beginner-Friendly Explanation**
Razor Pages is **“a page that handles itself.”** Instead of a controller choosing a view, the page has both the markup and the code behind it.

---

### Purposes

- To **simplify page-based web apps** by removing controller/view separation.
- To make pages **self-contained and discoverable** (one folder per page).
- To reduce **ceremony** for typical CRUD and form scenarios.
- To improve **testability** of page logic via `PageModel`.
- To support **server-rendered HTML** with tag helpers and validation.

---

### Syntax Rules and Structure

```
Pages/
├── _Layout.cshtml
├── _ViewImports.cshtml
├── _ViewStart.cshtml
├── Index.cshtml
├── Index.cshtml.cs
├── Products/
│   ├── Index.cshtml
│   ├── Index.cshtml.cs
│   ├── Details.cshtml
│   ├── Details.cshtml.cs
│   ├── Create.cshtml
│   └── Create.cshtml.cs
```

```csharp
// Program.cs
builder.Services.AddRazorPages();
...
app.MapRazorPages();
```

**Component breakdown**

- `Pages/` — Root folder for Razor Pages.
- `.cshtml` — Razor markup.
- `.cshtml.cs` — `PageModel` class.
- `AddRazorPages()` — Registers services.
- `MapRazorPages()` — Adds endpoints.

#### Syntax Rules

- Razor Pages live under the `Pages` folder by default.
- The URL is the file path relative to `Pages/`, without the `.cshtml` extension.
- Each page may have a `.cshtml.cs` PageModel; pages without one are still valid.
- `_ViewImports.cshtml` adds shared `@using` and tag helpers.
- `_ViewStart.cshtml` sets the default layout.

#### Constraints and Limitations

- Razor Pages is optimised for page-based UI; complex API surfaces may prefer controllers or Minimal APIs.
- File-based routing is opinionated; custom routes require `@page "{...}"`.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Minimal Razor Page

```cshtml
@* Pages/Index.cshtml *@
@page
@model IndexModel
<h1>@Model.Message</h1>
```

```csharp
// Pages/Index.cshtml.cs
using Microsoft.AspNetCore.Mvc.RazorPages;

public class IndexModel : PageModel
{
    public string Message { get; private set; } = "";
    public void OnGet() => Message = "Hello from Razor Pages!";
}
```

**Expected Output (rendered HTML)**
```html
<h1>Hello from Razor Pages!</h1>
```

**Why this output occurs**
`OnGet` runs when the page is requested; it sets `Message`, which is rendered in the markup.

---

#### Example 2: Page with a Form

```cshtml
@page
@model ContactModel
<form method="post">
    <input asp-for="Name" />
    <button type="submit">Send</button>
</form>
@if (Model.Submitted) { <p>Thanks, @Model.Name!</p> }
```

```csharp
public class ContactModel : PageModel
{
    [BindProperty] public string? Name { get; set; }
    public bool Submitted { get; private set; }

    public void OnGet() { }
    public void OnPost() => Submitted = true;
}
```

**Expected Output (after POST)**
```html
<form method="post">
    <input name="Name" value="Alice" />
    <button type="submit">Send</button>
</form>
<p>Thanks, Alice!</p>
```

**Why this output occurs**
`[BindProperty]` binds form fields; `OnPost` sets `Submitted`, and the markup conditionally renders the message.

---

### Real-World Cases

- **Line-of-business forms** — CRUD pages for entities.
- **Account pages** — Login, register, profile.
- **Admin portals** — List/detail pages with search and paging.
- **Marketing pages** — Server-rendered content with dynamic sections.

---

### References

- Introduction to Razor Pages in ASP.NET Core — Microsoft Learn — https://learn.microsoft.com/en-us/aspnet/core/razor-pages/
- Razor Pages — GitHub — https://github.com

---

## 2. Page Models

### Definitions

**Core Definition**
A `PageModel` is a class that encapsulates the data, handlers, and logic for a Razor Page.

**Technical Definition**
The `PageModel` class (conventionally in a `.cshtml.cs` file with the same name as the page) derives from `Microsoft.AspNetCore.Mvc.RazorPages.PageModel`. It exposes:
- **Properties** bound to the view (`@model`).
- **`[BindProperty]`** properties for input binding.
- **Handler methods** (`OnGet`, `OnPost`, etc.).
- **`Page()`** to render the page.
- **`RedirectToPage()`**, **`NotFound()`**, etc., for result control.
- **`ModelState`** for validation.

**Beginner-Friendly Explanation**
The `PageModel` is the **code-behind** for the page. It holds the data the page displays and the handlers that respond to requests.

---

### Purposes

- To **separate markup from logic** while keeping them together per page.
- To **hold page-specific data** (view model properties).
- To **handle requests** (GET, POST) via handler methods.
- To **validate input** with `ModelState`.
- To be **unit-testable** independently of HTTP.

---

### Syntax Rules and Structure

```csharp
public class CreateModel : PageModel
{
    private readonly IProductService _service;
    public CreateModel(IProductService service) => _service = service;

    [BindProperty] public CreateProductDto Dto { get; set; } = new();

    public void OnGet() { }

    public async Task<IActionResult> OnPostAsync()
    {
        if (!ModelState.IsValid) return Page();
        await _service.CreateAsync(Dto);
        return RedirectToPage("Index");
    }
}
```

**Component breakdown**

- `: PageModel` — Base class.
- Constructor injection — Services from DI.
- `[BindProperty]` — Binds input to the property.
- `OnGet` / `OnPostAsync` — Handlers.
- `Page()` / `RedirectToPage()` — Result helpers.

#### Syntax Rules

- The `PageModel` class should be in the same folder as the `.cshtml` file.
- Constructor injection is supported; the model is activated per request.
- `[BindProperty]` binds on POST by default; use `SupportsGet = true` to bind on GET.
- `[BindProperties]` at the class level binds all public properties.

#### Constraints and Limitations

- PageModel is transient per request; do not store cross-request state.
- `[BindProperty]` is for form data; JSON APIs should use `[FromBody]`.

---

### Annotated Code Examples and Expected Outputs

#### Example: PageModel with Validation

```csharp
public class CreateModel : PageModel
{
    [BindProperty] public CreateProductDto Dto { get; set; } = new();
    public string? Message { get; private set; }

    public void OnGet() { }

    public IActionResult OnPost()
    {
        if (!ModelState.IsValid) return Page();
        Message = $"Created {Dto.Name}";
        return Page();
    }
}

public class CreateProductDto
{
    [Required, StringLength(100)] public string Name { get; set; } = "";
    [Range(0.01, 1_000_000)] public decimal Price { get; set; }
}
```

**Expected Output**

- Invalid POST → page re-renders with validation messages.
- Valid POST → page renders `Created <name>`.

**Why this output occurs**
`ModelState.IsValid` reflects DataAnnotations; `Page()` renders the current page.

---

### Real-World Cases

- **Form pages** — Create/Edit with validation.
- **Detail pages** — Load entity by ID in `OnGet`.
- **Confirmation pages** — POST then show a summary.

---

### References

- Razor Pages page model — Microsoft Learn — https://learn.microsoft.com/en-us/aspnet/core/razor-pages/
- PageModel class — .NET API — https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.razorpages.pagemodel

---

## 3. Handlers

### Definitions

**Core Definition**
Handlers are **public methods on a PageModel** invoked by the Razor Pages framework based on the HTTP verb and an optional handler name.

**Technical Definition**
Handlers follow the naming convention `On{Verb}` (e.g., `OnGet`, `OnPost`) or `On{Verb}{HandlerName}` (e.g., `OnPostDelete`, `OnPostSaveDraft`). The verb can be `Get`, `Post`, `Put`, `Delete`, or `Patch` (custom verbs require filters). Handlers can be synchronous (return `void` or `IActionResult`) or asynchronous (return `Task` or `Task<IActionResult>`). Multiple handlers can exist per verb, distinguished by name via the `?handler=` query string or an `<input type="hidden" name="handler" value="...">` field.

**Beginner-Friendly Explanation**
Handlers are the **“what to do when” methods**. `OnGet` handles page loads; `OnPost` handles form submissions; `OnPostDelete` handles a delete button.

---

### Purposes

- To **respond to specific HTTP verbs**.
- To **support multiple actions per page** (save, delete, publish).
- To **keep page logic local** to the page.
- To enable **async I/O** via `OnGetAsync`/`OnPostAsync`.

---

### Syntax Rules and Structure

```csharp
public void OnGet() { }
public Task OnGetAsync() => Task.CompletedTask;
public IActionResult OnPost() => Page();
public async Task<IActionResult> OnPostAsync() { ... }
public IActionResult OnPostDelete(int id) { ... }
```

**Component breakdown**

- `On{Verb}` — Handler for the verb.
- `On{Verb}{Name}` — Named handler.
- `Async` suffix — Async handler.
- Return types — `void`, `IActionResult`, `Task`, `Task<IActionResult>`.

#### Syntax Rules

- Handlers must be public and non-static.
- Handlers cannot have `[NonAction]`; all public methods are handlers.
- Named handlers are invoked via `?handler=Name` (GET) or a form field named `handler` (POST).
- `OnGet` is optional; without it, the page renders with default data.

#### Constraints and Limitations

- Only one handler per verb/name combination.
- Custom HTTP verbs require custom filters or middleware.
- Handlers must not throw unhandled exceptions; return error results instead.

---

### Annotated Code Examples and Expected Outputs

#### Example: Multiple Named Handlers

```cshtml
@page
@model OrderModel
<form method="post">
    <input type="hidden" name="id" value="42" />
    <button type="submit" asp-page-handler="Save">Save</button>
    <button type="submit" asp-page-handler="Delete">Delete</button>
</form>
```

```csharp
public class OrderModel : PageModel
{
    public string? Message { get; private set; }

    public void OnGet() { }
    public void OnPostSave(int id) => Message = $"Saved order {id}";
    public void OnPostDelete(int id) => Message = $"Deleted order {id}";
}
```

**Expected Output**

- Click “Save” → page shows `Saved order 42`.
- Click “Delete” → page shows `Deleted order 42`.

**Why this output occurs**
`asp-page-handler` sets the `handler` form field; the framework dispatches to `OnPostSave` or `OnPostDelete`.

---

### Real-World Cases

- **Multi-action forms** — Save draft, publish, delete.
- **AJAX handlers** — Named handlers returning JSON.
- **Confirmation flows** — `OnPostConfirm`, `OnPostCancel`.

---

### References

- Handler methods in Razor Pages — Microsoft Learn — https://learn.microsoft.com/en-us/aspnet/core/razor-pages/#handler-methods
- Razor Pages handler methods — Microsoft Learn — https://learn.microsoft.com/en-us/aspnet/core/razor-pages/

---

## 4. Routing

### Definitions

**Core Definition**
Routing in Razor Pages maps URLs to page files based on their **location under the `Pages` folder**, with optional custom route templates.

**Technical Definition**
By default, a Razor Page's URL is its file path relative to `Pages/`, without the `.cshtml` extension (e.g., `Pages/Products/Index.cshtml` → `/Products/Index`). A page can define a custom route with `@page "{template}"` (e.g., `@page "{id:int}"` for `/Products/Details/42`). Multiple routes can be declared with multiple `@page` directives. Route constraints are supported. `MapRazorPages()` registers the endpoints in `Program.cs`.

**Beginner-Friendly Explanation**
Routing is **where the page lives in the URL**. The file `Pages/Products/Details.cshtml` becomes `/Products/Details`; add `@page "{id:int}"` to accept an ID in the URL.

---

### Purposes

- To **map URLs to pages** without controller configuration.
- To support **RESTful URLs** via custom routes.
- To **validate segments** with constraints.
- To **generate URLs** with `asp-page` and `RedirectToPage`.

---

### Syntax Rules and Structure

```cshtml
@page
@page "{id:int}"
@page "{id:int}/{slug?}"
```

```csharp
app.MapRazorPages();
```

**Component breakdown**

- `@page` — Makes a `.cshtml` file a Razor Page and optionally defines a route.
- `{id:int}` — Route parameter with constraint.
- `{slug?}` — Optional route parameter.

#### Syntax Rules

- The default route is the file path relative to `Pages/`.
- `@page` is required at the top of every Razor Page.
- `@page "{template}"` overrides the default route.
- Route parameters bind to PageModel properties or handler parameters.

#### Constraints and Limitations

- Custom routes must not conflict with other pages or endpoints.
- Route parameters are optional only if marked `?` or given defaults.

---

### Annotated Code Examples and Expected Outputs

#### Example: Custom Route with ID

```cshtml
@* Pages/Products/Details.cshtml *@
@page "{id:int}"
@model DetailsModel
<h1>Product @Model.Id</h1>
```

```csharp
public class DetailsModel : PageModel
{
    public int Id { get; private set; }
    public void OnGet(int id) => Id = id;
}
```

**Expected Output**

- `GET /Products/Details/42` → `<h1>Product 42</h1>`
- `GET /Products/Details/abc` → `404 Not Found` (fails `int` constraint)

**Why this output occurs**
The `@page "{id:int}"` template binds `id` from the URL; the constraint rejects non-numeric values.

---

### Real-World Cases

- **Blog slugs** — `@page "{slug}"`.
- **Localized routes** — `@page "{culture}/{id:int}"`.
- **Nested resources** — `@page "/customers/{customerId:int}/orders/{orderId:int}"`.

---

### References

- Razor Pages route and app conventions — Microsoft Learn — https://learn.microsoft.com/en-us/aspnet/core/razor-pages/razor-pages-conventions
- Razor Pages routing — Microsoft Learn — https://learn.microsoft.com/en-us/aspnet/core/razor-pages/

---

## 5. Razor Pages Best Practices

### Definitions

**Core Definition**
Razor Pages best practices are the guidelines that keep Razor Pages applications **secure, maintainable, testable, and performant**.

**Technical Definition**
Best practices include: **keep PageModels thin** (delegate to services), **use `[BindProperty]` and DTOs** rather than binding entities directly, **validate with DataAnnotations and `ModelState`**, **use the PRG (Post/Redirect/Get) pattern** to avoid duplicate submissions, **enable anti-forgery tokens** on POST handlers (they are enabled by default in Razor Pages), **prefer async handlers** for I/O, **use `asp-page` and `asp-page-handler` tag helpers** for URL generation, **organise pages by feature**, **use layouts and partials** for shared UI, and **test handlers directly** without HTTP.

**Beginner-Friendly Explanation**
The best practices are: **keep pages small**, **don't bind entities directly**, **validate inputs**, **redirect after POST**, **use anti-forgery tokens**, **use async for I/O**, and **use tag helpers**.

---

### Guidelines and Structure

#### General Guidelines

```csharp
// 1. Thin PageModel
public class IndexModel : PageModel
{
    private readonly IProductService _svc;
    public IndexModel(IProductService svc) => _svc = svc;

    public IReadOnlyList<ProductDto> Products { get; private set; } = Array.Empty<ProductDto>();

    public async Task OnGetAsync() => Products = await _svc.GetAllAsync();
}

// 2. DTO, not entity
[BindProperty] public CreateProductDto Dto { get; set; } = new();

// 3. Validation
public IActionResult OnPost()
{
    if (!ModelState.IsValid) return Page();
    ...
}

// 4. PRG pattern
public IActionResult OnPost()
{
    if (!ModelState.IsValid) return Page();
    _svc.Create(Dto);
    return RedirectToPage("Index");
}

// 5. Async handlers for I/O
public async Task OnGetAsync() => Products = await _svc.GetAllAsync();

// 6. Tag helpers
<form method="post" asp-page-handler="Save">...</form>
<a asp-page="/Products/Details" asp-route-id="@p.Id">@p.Name</a>
```

**Component breakdown**

- **Thin PageModel** — Orchestrate, don't implement business logic.
- **DTOs** — Prevent over-posting.
- **Validation** — DataAnnotations + `ModelState`.
- **PRG** — Redirect after successful POST.
- **Async** — Scalability.
- **Tag helpers** — Type-safe URLs and form generation.

#### Rules and Constraints

- **Do** keep PageModels small and focused.
- **Do** use DTOs for input binding.
- **Do** validate with `ModelState.IsValid`.
- **Do** use PRG for form submissions.
- **Do** use async handlers for I/O.
- **Do** use tag helpers for URL and form generation.
- **Do not** bind entities directly to `[BindProperty]`.
- **Do not** put business logic in the PageModel.
- **Do not** disable anti-forgery tokens without a strong reason.

---

### Annotated Code Examples and Expected Outputs

#### Example: Well-Structured Razor Page

```cshtml
@* Pages/Products/Create.cshtml *@
@page
@model CreateModel
<form method="post">
    <input asp-for="Dto.Name" />
    <span asp-validation-for="Dto.Name"></span>
    <input asp-for="Dto.Price" />
    <span asp-validation-for="Dto.Price"></span>
    <button type="submit">Create</button>
</form>
```

```csharp
public class CreateModel : PageModel
{
    private readonly IProductService _svc;
    public CreateModel(IProductService svc) => _svc = svc;

    [BindProperty] public CreateProductDto Dto { get; set; } = new();

    public void OnGet() { }

    public async Task<IActionResult> OnPostAsync()
    {
        if (!ModelState.IsValid) return Page();
        await _svc.CreateAsync(Dto);
        return RedirectToPage("Index");
    }
}
```

**Expected Output**

- Invalid POST → re-renders form with validation messages.
- Valid POST → `302 Found` redirect to `Index`.

**Why this is well-structured**
It uses a DTO, validation, async I/O, PRG, and tag helpers—all Razor Pages best practices.

---

### Real-World Cases

- **CRUD pages** — Create/Edit/Delete with DTOs and validation.
- **Account pages** — Login/register with anti-forgery.
- **Admin dashboards** — List/detail with async data loading.

---

### References

- Introduction to Razor Pages in ASP.NET Core — Microsoft Learn — https://learn.microsoft.com/en-us/aspnet/core/razor-pages/
- Razor Pages route and app conventions — Microsoft Learn — https://learn.microsoft.com/en-us/aspnet/core/razor-pages/razor-pages-conventions
- Model validation in ASP.NET Core — Microsoft Learn — https://learn.microsoft.com/en-us/aspnet/core/mvc/models/validation

---

## Deprecated, Unsafe, Implementation-Dependent, and Version-Specific Notes

| Feature | Status / Note |
|---|---|
| `[BindProperty]` | Binds on POST by default; use `SupportsGet = true` for GET binding. |
| Anti-forgery tokens | **Enabled by default** in Razor Pages POST handlers. |
| `Page()` vs. `RedirectToPage()` | Use `RedirectToPage` after successful POST (PRG). |
| `[BindProperties]` | Class-level binding of all public properties; use with caution (over-posting). |
| `@page` directive | **Required** at the top of every Razor Page. |
| Custom HTTP verbs | Require custom filters or middleware. |
| `IAsyncActionFilter` on pages | Supported via page filters. |

---

## References (Consolidated)

- Introduction to Razor Pages in ASP.NET Core — Microsoft Learn — https://learn.microsoft.com/en-us/aspnet/core/razor-pages/
- Razor Pages route and app conventions — Microsoft Learn — https://learn.microsoft.com/en-us/aspnet/core/razor-pages/razor-pages-conventions
- PageModel class — .NET API — https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.razorpages.pagemodel
- Razor Pages handler methods — Microsoft Learn — https://learn.microsoft.com/en-us/aspnet/core/razor-pages/
- Model validation in ASP.NET Core — Microsoft Learn — https://learn.microsoft.com/en-us/aspnet/core/mvc/models/validation
- Razor Pages — GitHub — https://github.com

---