# C# ASP.NET Core MVC — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
**MVC (Model-View-Controller)** is an architectural pattern that separates an application into three responsibilities: **Models** (data and business rules), **Views** (presentation), and **Controllers** (request handling and coordination). ASP.NET Core MVC is the framework implementation of this pattern for building web applications and APIs on .NET. It provides routing, model binding, validation, filters, Razor views, and a testable, convention-driven programming model.

**Technical Definition**
ASP.NET Core MVC is a framework built on ASP.NET Core's middleware pipeline. It implements the Model-View-Controller design pattern, separating concerns into Models, Views, and Controllers. **Controllers** are classes (conventionally ending in `Controller`) that contain **actions**—public methods that handle incoming requests and return `IActionResult` or `Task<IActionResult>`. **Routing** maps request URLs to controller actions via conventional or attribute routes. **Model binding** converts request data (form, route, query, body) into action parameters. **Model validation** uses DataAnnotations and `ModelState` to validate bound models. **Filters** provide cross-cutting concerns (authorization, logging, error handling, caching) at well-defined stages of the request pipeline. **Views** are Razor templates (`.cshtml`) that render HTML from models.

**Beginner-Friendly Explanation**
MVC is a **way to organize a web app so each part has one job**. The **Model** is the data and rules; the **View** is what the user sees; the **Controller** is the middleman that takes a request, asks the model for data, and hands it to the view. ASP.NET Core MVC gives you all the plumbing—routing, form handling, validation, and rendering—so you can focus on those three pieces. It is the older, more structured sibling of Minimal APIs: more ceremony, but more built-in structure for larger applications.

---

### Key Characteristics

- **Separation of concerns** — Models, Views, and Controllers have distinct responsibilities.
- **Convention over configuration** — Naming and folder conventions drive discovery and routing.
- **Razor views** — `.cshtml` templates combining HTML and C#.
- **Action-based routing** — URLs map to controller actions via conventional or attribute routes.
- **Model binding and validation** — Request data is bound to parameters and validated automatically.
- **Filters** — Cross-cutting concerns at well-defined pipeline stages.
- **Testability** — Controllers are plain classes; actions can be unit tested without a web server.
- **Razor Pages and Minimal APIs as alternatives** — MVC is one of several ASP.NET Core programming models.

---

### Prerequisites

- Basic understanding of **ASP.NET Core fundamentals** (`Program.cs`, DI, configuration, middleware).
- Familiarity with **C# classes, interfaces, and async/await**.
- Knowledge of **HTTP** (verbs, status codes, headers, forms).
- Awareness of **Razor syntax** (`@model`, `@foreach`, `@Html`).
- Understanding of **DataAnnotations** for validation.

---

### Related Programming Areas

- **ASP.NET Core routing** — Conventional and attribute routing.
- **Dependency injection** — Controllers receive services via constructor injection.
- **Filters and middleware** — Cross-cutting concerns at different levels.
- **Razor Pages** — A page-based alternative to MVC for server-rendered apps.
- **Minimal APIs** — A lower-ceremony alternative for HTTP APIs.

---

### Core Concepts / Features

1. MVC Pattern
2. Controllers
3. Actions
4. Views
5. Models
6. Routing
7. Filters
8. Model Binding
9. Model Validation
10. MVC Best Practices

---

## 1. MVC Pattern

### Definitions

**Core Definition**
MVC is a software architectural pattern that separates data, presentation, and request handling into three interacting components.

**Technical Definition**
In ASP.NET Core MVC:
- **Model** — The domain data, business rules, and validation logic. May include entities, view models, and DTOs.
- **View** — Razor templates that render HTML from a model.
- **Controller** — Classes that handle HTTP requests, orchestrate work, and select views or return results.

The pattern promotes separation of concerns, testability, and parallel development.

**Beginner-Friendly Explanation**
MVC is **three jobs for three parts**: Model holds the data, View shows it, Controller decides what to show.

---

### Purposes

- To **separate concerns** and reduce coupling.
- To make controllers **unit-testable**.
- To support **parallel development** (UI, logic, data).
- To provide a **structured foundation** for medium-to-large apps.

---

### Syntax Rules and Structure

```
Request → Routing → Controller Action → Model (data) → View (HTML) → Response
```

**Component breakdown**

- **Routing** — Selects the controller and action.
- **Controller** — Executes logic and populates the model.
- **Model** — Carries data to the view.
- **View** — Renders HTML.

#### Syntax Rules

- Controllers live in `Controllers/` by convention.
- Views live in `Views/{Controller}/{Action}.cshtml` or `Views/Shared/`.
- Models are typically classes in `Models/` or `ViewModels/`.

#### Constraints and Limitations

- MVC is heavier than Minimal APIs; overkill for tiny services.
- Poorly structured MVC code can become "fat controllers."

---

### Annotated Code Examples and Expected Outputs

#### Example: MVC Flow

```csharp
// Model
public record Product(int Id, string Name, decimal Price);

// Controller
public class ProductsController : Controller
{
    public IActionResult Index()
    {
        var products = new[] { new Product(1, "Widget", 9.99m) };
        return View(products);
    }
}
```

```cshtml
@* View: Views/Products/Index.cshtml *@
@model IEnumerable<Product>
<h1>Products</h1>
<ul>
@foreach (var p in Model)
{
    <li>@p.Name — @p.Price.ToString("C")</li>
</ul>
```

**Expected Output (rendered HTML)**
```html
<h1>Products</h1>
<ul>
    <li>Widget — $9.99</li>
</ul>
```

**Why this output occurs**
The controller selects the `Index` view, passes the product collection as the model, and Razor renders each product.

---

### Real-World Cases

- **Line-of-business apps** — Server-rendered dashboards and forms.
- **E-commerce storefronts** — Product listing and detail pages.
- **Admin portals** — CRUD screens with validation.

---

### References

- ASP.NET Core MVC overview — Microsoft Learn — https://learn.microsoft.com/en-us/aspnet/core/mvc/overview

---

## 2. Controllers

### Definitions

**Core Definition**
A controller is a class that handles incoming HTTP requests, executes application logic, and returns a response (typically a view or a result).

**Technical Definition**
Controllers derive from `Controller` (MVC with views) or `ControllerBase` (APIs without views). They are discovered by convention (class name ending in `Controller` in the `Controllers` folder), activated per request, and receive dependencies via constructor injection. Controllers are **not** static and are **not** kept alive between requests.

**Beginner-Friendly Explanation**
A controller is the **traffic cop** of MVC. It receives the request, asks the model for data, and picks the view to render.

---

### Purposes

- To **handle HTTP requests** and route them to actions.
- To **orchestrate** services and business logic.
- To **select views** or return `IActionResult`.
- To be **unit-testable** without a web server.

---

### Syntax Rules and Structure

```csharp
public class ProductsController : Controller
{
    private readonly IProductService _service;
    public ProductsController(IProductService service) => _service = service;
}
```

**Component breakdown**

- `: Controller` — Base class with view support and helpers (`View`, `RedirectToAction`, `NotFound`).
- Constructor injection — Dependencies provided by DI.
- Convention — Class name ends in `Controller`.

#### Syntax Rules

- Controllers must be public and non-abstract.
- Constructors should be free of logic beyond null checks.
- Actions are public methods; other public methods must be marked `[NonAction]`.

#### Constraints and Limitations

- Controllers are created per request; do not store request-scoped state in fields.
- Avoid "fat controllers"—delegate to services.

---

### Annotated Code Examples and Expected Outputs

#### Example: Controller with DI and Actions

```csharp
public class ProductsController : Controller
{
    private readonly IProductService _service;
    public ProductsController(IProductService service) => _service = service;

    public async Task<IActionResult> Index()
    {
        var products = await _service.GetAllAsync();
        return View(products);
    }

    public async Task<IActionResult> Details(int id)
    {
        var product = await _service.GetAsync(id);
        if (product is null) return NotFound();
        return View(product);
    }
}
```

**Expected Output**

- `GET /Products/Index` → `200 OK` with rendered view.
- `GET /Products/Details/1` → `200 OK` if found; `404 Not Found` otherwise.

**Why this output occurs**
`View` returns a `ViewResult`; `NotFound` returns a `NotFoundResult` with status 404.

---

### Real-World Cases

- **CRUD controllers** — `Create`, `Read`, `Update`, `Delete` actions.
- **Account controllers** — Login, logout, register.
- **Dashboard controllers** — Aggregated views.

---

### References

- Controllers — Microsoft Learn — https://learn.microsoft.com/en-us/aspnet/core/mvc/controllers/actions
- Handle requests with controllers in ASP.NET Core MVC — Microsoft Learn — https://learn.microsoft.com/en-us/aspnet/core/mvc/controllers/actions

---

## 3. Actions

### Definitions

**Core Definition**
An action is a **public method on a controller** that handles a request and returns a result.

**Technical Definition**
Actions are discovered by convention: any public, non-static, non-abstract method on a controller is an action, unless decorated with `[NonAction]`. Actions can be synchronous (`IActionResult`) or asynchronous (`Task<IActionResult>`). They are selected by routing and can be restricted to HTTP verbs with `[HttpGet]`, `[HttpPost]`, etc. Action results (`IActionResult`) include `ViewResult`, `JsonResult`, `RedirectResult`, `StatusCodeResult`, and `FileResult`.

**Beginner-Friendly Explanation**
An action is a **method that answers a specific URL**. `Index` answers `/Products/Index`; `Details` answers `/Products/Details/{id}`.

---

### Purposes

- To **handle a specific request** (URL + verb).
- To **return a result** (view, JSON, redirect, status).
- To be **restricted by HTTP verb** and route.
- To be **unit-testable**.

---

### Syntax Rules and Structure

```csharp
[HttpGet]
public IActionResult Details(int id) { ... }

[HttpPost]
[ValidateAntiForgeryToken]
public async Task<IActionResult> Create(CreateDto dto) { ... }
```

**Component breakdown**

- `[HttpGet]`/`[HttpPost]` — Verb restriction.
- `IActionResult`/`Task<IActionResult>` — Return type.
- `[ValidateAntiForgeryToken]` — CSRF protection for POST.

#### Syntax Rules

- Actions must be public and non-static.
- Async actions should return `Task<IActionResult>`.
- Use `[ActionName]` to override the action name.
- Mark helper methods `[NonAction]`.

#### Constraints and Limitations

- Action names must be unique within a controller unless overloaded by verb or route.
- Actions cannot have `ref`/`out` parameters.

---

### Annotated Code Examples and Expected Outputs

#### Example: Verb-Restricted Actions

```csharp
public class ProductsController : Controller
{
    [HttpGet]
    public IActionResult Create() => View();

    [HttpPost]
    [ValidateAntiForgeryToken]
    public IActionResult Create(CreateProductDto dto)
    {
        if (!ModelState.IsValid) return View(dto);
        return RedirectToAction(nameof(Index));
    }
}
```

**Expected Output**

- `GET /Products/Create` → form view.
- `POST /Products/Create` with valid data → `302 Found` redirect to `Index`.
- `POST /Products/Create` with invalid data → `200 OK` re-rendering the form with errors.

**Why this output occurs**
The GET action renders the form; the POST action validates and redirects (PRG pattern) or re-renders on failure.

---

### Real-World Cases

- **Form submission** — GET renders, POST processes.
- **AJAX endpoints** — JSON actions.
- **File downloads** — `FileResult`.

---

### References

- Handle requests with controllers in ASP.NET Core MVC — Microsoft Learn — https://learn.microsoft.com/en-us/aspnet/core/mvc/controllers/actions

---

## 4. Views

### Definitions

**Core Definition**
A view is a **Razor template** (`.cshtml`) that renders HTML from a model.

**Technical Definition**
Views are stored in `Views/{ControllerName}/{ActionName}.cshtml` or `Views/Shared/`. They use Razor syntax, starting with `@model` to declare the model type. Views can use **layouts** (`_Layout.cshtml`), **partial views** (`_PartialName.cshtml`), **tag helpers**, and **HTML helpers**. The view engine searches well-known locations and uses `_ViewStart.cshtml` to set defaults.

**Beginner-Friendly Explanation**
A view is the **HTML page** that the user sees. It mixes HTML with C# using Razor syntax.

---

### Purposes

- To **render HTML** from a model.
- To **reuse layout and partials** across pages.
- To **support tag helpers** and validation messages.
- To **separate presentation** from controller logic.

---

### Syntax Rules and Structure

```cshtml
@model Product
@{
    ViewData["Title"] = "Details";
}
<h1>@Model.Name</h1>
<p>@Model.Price.ToString("C")</p>
```

**Component breakdown**

- `@model` — Declares the model type.
- `@{ ... }` — Code block.
- `@expression` — Outputs the expression.
- `ViewData`/`ViewBag` — Pass ad-hoc data.

#### Syntax Rules

- Views require `_ViewImports.cshtml` for common `@using` and tag helpers.
- Layouts are applied via `_ViewStart.cshtml` or `Layout = "_Layout"`.
- Partial views are rendered with `<partial name="_Partial" model="..." />`.

#### Constraints and Limitations

- Avoid business logic in views.
- Strongly typed views (`@model T`) are preferred over `ViewBag`.

---

### Annotated Code Examples and Expected Outputs

#### Example: Strongly Typed View with Layout

```cshtml
@* Views/Shared/_Layout.cshtml *@
<!DOCTYPE html>
<html>
<head><title>@ViewData["Title"]</title></head>
<body>@RenderBody()</body>
</html>
```

```cshtml
@* Views/Products/Details.cshtml *@
@model Product
@{
    ViewData["Title"] = "Product Details";
}
<h1>@Model.Name</h1>
<p>@Model.Price.ToString("C")</p>
```

**Expected Output (rendered)**
```html
<!DOCTYPE html>
<html>
<head><title>Product Details</title></head>
<body>
    <h1>Widget</h1>
    <p>$9.99</p>
</body>
</html>
```

**Why this output occurs**
The view sets `ViewData["Title"]`, the layout renders it, and `RenderBody()` injects the view content.

---

### Real-World Cases

- **Server-rendered pages** — Product pages, dashboards.
- **Email templates** — Razor views rendered to strings.
- **Partial views** — Reusable UI fragments.

---

### References

- Views in ASP.NET Core MVC — Microsoft Learn — https://learn.microsoft.com/en-us/aspnet/core/mvc/views/overview

---

## 5. Models

### Definitions

**Core Definition**
A model is a class that represents the data and rules used by the application.

**Technical Definition**
In ASP.NET Core MVC, "model" encompasses **domain models** (entities), **view models** (shaped data for a view), and **input models** (DTOs for actions). Models carry data between controllers and views, and often carry validation attributes. Best practice is to use **view models** or **DTOs** rather than exposing domain entities directly.

**Beginner-Friendly Explanation**
A model is the **data** your app works with. It can be a database entity, a form submission, or a shaped object for a view.

---

### Purposes

- To **carry data** between controllers and views.
- To **encapsulate business rules** and validation.
- To **shape data** for specific views or APIs.
- To **decouple** the domain from presentation.

---

### Syntax Rules and Structure

```csharp
public class ProductDetailsViewModel
{
    public int Id { get; set; }
    public string Name { get; set; } = string.Empty;
    public decimal Price { get; set; }
}
```

**Component breakdown**

- Properties — Data carried to the view.
- Validation attributes — Optional DataAnnotations.

#### Syntax Rules

- View models are typically POCOs.
- Use `required` or `= string.Empty` to satisfy nullable analysis.
- Do not expose entities directly if they contain sensitive fields.

#### Constraints and Limitations

- Entities may have navigation properties that cause serialization cycles.
- Over-posting vulnerabilities occur when binding entities directly.

---

### Annotated Code Examples and Expected Outputs

#### Example: View Model for a Details Page

```csharp
public record ProductDetailsViewModel(int Id, string Name, decimal Price);

public IActionResult Details(int id)
{
    var p = _service.Get(id);
    if (p is null) return NotFound();
    var vm = new ProductDetailsViewModel(p.Id, p.Name, p.Price);
    return View(vm);
}
```

**Expected Output (conceptual)**
```
// View receives a ProductDetailsViewModel with Id, Name, Price.
```

**Why this view model is used**
It exposes only the fields needed by the view, reducing coupling and over-posting risk.

---

### Real-World Cases

- **Form models** — `CreateProductDto`, `LoginViewModel`.
- **View models** — `ProductDetailsViewModel`, `DashboardViewModel`.
- **Domain entities** — `Product`, `Order` (used in services, not directly in views).

---

### References

- Model validation in ASP.NET Core MVC — Microsoft Learn — https://learn.microsoft.com/en-us/aspnet/core/mvc/models/validation

---

## 6. Routing

### Definitions

**Core Definition**
Routing is the mechanism that maps incoming URLs to controller actions.

**Technical Definition**
ASP.NET Core MVC supports **conventional routing** (`{controller=Home}/{action=Index}/{id?}`) and **attribute routing** (`[Route]`, `[HttpGet]`, `[HttpPost]`). Conventional routes are defined with `MapControllerRoute` in `Program.cs`; attribute routes are applied directly to controllers and actions. Attribute routing takes precedence over conventional routing. Route constraints (`{id:int}`) validate segments.

**Beginner-Friendly Explanation**
Routing is the **map from URLs to code**. `/Products/Details/5` goes to `ProductsController.Details(5)`.

---

### Purposes

- To **map URLs** to controller actions.
- To **support RESTful URLs** via attribute routing.
- To **validate segments** with constraints.
- To **generate URLs** with tag helpers and `Url.Action`.

---

### Syntax Rules and Structure

```csharp
// Conventional routing
app.MapControllerRoute(
    name: "default",
    pattern: "{controller=Home}/{action=Index}/{id?}");

// Attribute routing
[Route("api/[controller]")]
public class ProductsController : Controller
{
    [HttpGet("{id:int}")]
    public IActionResult Get(int id) => Ok(...);
}
```

**Component breakdown**

- `MapControllerRoute` — Conventional routing entry point.
- `[Route]` — Attribute routing.
- `{id:int}` — Route constraint.
- `[controller]`, `[action]` — Token replacement.

#### Syntax Rules

- Attribute routes override conventional routes for the decorated action.
- Route constraints return `404` on mismatch.
- Route names enable link generation.

#### Constraints and Limitations

- Conflicting routes cause ambiguous match exceptions.
- Conventional routing requires the controller/action names in the URL.

---

### Annotated Code Examples and Expected Outputs

#### Example: Conventional and Attribute Routing Together

```csharp
app.MapControllerRoute(
    name: "default",
    pattern: "{controller=Home}/{action=Index}/{id?}");

[Route("api/products")]
public class ProductsApiController : Controller
{
    [HttpGet("{id:int}")]
    public IActionResult Get(int id) => Ok(new { Id = id });
}
```

**Expected Output**

- `GET /Home/Index` → `HomeController.Index` (conventional).
- `GET /api/products/42` → `ProductsApiController.Get(42)` (attribute).

**Why this output occurs**
Attribute routes take precedence for decorated actions; conventional routing handles the rest.

---

### Real-World Cases

- **SEO-friendly URLs** — `[Route("blog/{slug}")]`.
- **API versioning** — `[Route("api/v{version:apiVersion}/products")]`.
- **Localized routes** — Attribute routes per culture.

---

### References

- Routing to controller actions in ASP.NET Core — Microsoft Learn — https://learn.microsoft.com/en-us/aspnet/core/mvc/controllers/routing

---

## 7. Filters

### Definitions

**Core Definition**
Filters are components that run **before or after** specific stages of the MVC request pipeline, providing cross-cutting concerns.

**Technical Definition**
ASP.NET Core MVC defines five filter types, executed in this order:
1. **Authorization filters** — Run first; short-circuit unauthorized requests.
2. **Resource filters** — Run after authorization and before model binding; useful for caching.
3. **Action filters** — Run before and after the action method.
4. **Exception filters** — Apply global error-handling policies.
5. **Result filters** — Run before and after result execution.

Filters can be applied globally, at the controller level, or at the action level. They can be implemented as attributes (`IActionFilter`, `IAsyncActionFilter`) or as types registered in DI.

**Beginner-Friendly Explanation**
Filters are **checkpoints** in the request pipeline. Logging, authorization, and caching are implemented as filters that run at the right moment.

---

### Purposes

- To **centralize cross-cutting concerns** (logging, auth, caching).
- To **avoid repetitive code** in every action.
- To **control execution order** with `IOrderedFilter`.
- To **short-circuit** requests early when needed.

---

### Syntax Rules and Structure

```csharp
public class LogActionFilter : IActionFilter
{
    public void OnActionExecuting(ActionExecutingContext context) { }
    public void OnActionExecuted(ActionExecutedContext context) { }
}

// Apply
[ServiceFilter(typeof(LogActionFilter))]
public IActionResult Index() => View();
```

**Component breakdown**

- `IActionFilter` — Synchronous action filter.
- `IAsyncActionFilter` — Asynchronous action filter.
- `[ServiceFilter]` — Resolves filter from DI.
- `[TypeFilter]` — Instantiates filter with DI arguments.

#### Syntax Rules

- Filters execute in the documented order.
- Global filters are registered in `AddControllers(options => options.Filters.Add(...))`.
- Filters can be attributes or service-registered types.

#### Constraints and Limitations

- Filters cannot access the response body after it has started.
- Authorization filters should not be used for business authorization; use policies.

---

### Annotated Code Examples and Expected Outputs

#### Example: Action Filter Logging

```csharp
public class TimingFilter : IAsyncActionFilter
{
    private readonly ILogger<TimingFilter> _logger;
    public TimingFilter(ILogger<TimingFilter> logger) => _logger = logger;

    public async Task OnActionExecutionAsync(ActionExecutingContext ctx, ActionExecutionDelegate next)
    {
        var sw = Stopwatch.StartNew();
        var result = await next();
        _logger.LogInformation("{Action} took {Ms}ms", ctx.ActionDescriptor.DisplayName, sw.ElapsedMilliseconds);
    }
}

// Register globally
builder.Services.AddControllers(o => o.Filters.Add<TimingFilter>());
```

**Expected Output (log)**
```
info: TimingFilter[0]
      ProductsController.Index (Products) took 12ms
```

**Why this output occurs**
The filter wraps the action execution and logs the elapsed time.

---

### Real-World Cases

- **Authorization** — `[Authorize]`.
- **Caching** — `[ResponseCache]`.
- **Exception handling** — `[ExceptionFilter]`.
- **Auditing** — Custom action filters.

---

### References

- Filters in ASP.NET Core — Microsoft Learn — https://learn.microsoft.com/en-us/aspnet/core/mvc/controllers/filters

---

## 8. Model Binding

### Definitions

**Core Definition**
Model binding converts incoming request data (form, route, query, headers, body) into action parameters and model properties.

**Technical Definition**
Model binding runs before the action executes. It uses **value providers** ordered by precedence: **form**, **route**, **query**, **header**, and **body** (for `[FromBody]`). By default, complex types bind from the body (JSON) in API controllers and from form/route/query in MVC controllers. Binding sources can be forced with `[FromQuery]`, `[FromRoute]`, `[FromForm]`, `[FromBody]`, `[FromHeader]`, and `[FromServices]`. Custom types can implement `IModelBinder` or use `TryParse`/`BindAsync` (Minimal APIs).

**Beginner-Friendly Explanation**
Model binding is **automatic form reading**. You declare `CreateProductDto dto` as a parameter, and MVC fills it from the form, route, or body.

---

### Purposes

- To **eliminate manual request parsing**.
- To **convert strings to strongly typed values**.
- To **support form, route, query, and JSON binding**.
- To **integrate with validation**.

---

### Syntax Rules and Structure

```csharp
public IActionResult Search([FromQuery] string? q, [FromRoute] int page) { ... }

[HttpPost]
public IActionResult Create([FromForm] CreateProductDto dto) { ... }
```

**Component breakdown**

- `[FromQuery]` — Bind from query string.
- `[FromRoute]` — Bind from route values.
- `[FromForm]` — Bind from form data.
- `[FromBody]` — Bind from JSON body.
- `[FromHeader]` — Bind from headers.
- `[FromServices]` — Bind from DI.

#### Syntax Rules

- Complex types bind from body in API controllers by default.
- Simple types bind from route/query by default.
- Binding failures add errors to `ModelState`.

#### Constraints and Limitations

- Over-posting occurs when entities are bound directly.
- Large bodies should be streamed or limited.

---

### Annotated Code Examples and Expected Outputs

#### Example: Mixed Binding Sources

```csharp
[HttpPost("products/{category}")]
public IActionResult Create(
    [FromRoute] string category,
    [FromQuery] bool featured,
    [FromForm] CreateProductDto dto)
{
    return Ok(new { category, featured, dto.Name });
}
```

**Expected Output (conceptual)**
```
POST /products/books?featured=true
Form: Name=Widget&Price=9.99
→ { "category": "books", "featured": true, "name": "Widget" }
```

**Why this output occurs**
Each parameter is bound from its declared source.

---

### Real-World Cases

- **Forms** — `[FromForm]`.
- **APIs** — `[FromBody]`.
- **Filters** — `[FromQuery]`.
- **Multi-tenant** — `[FromRoute]` for tenant ID.

---

### References

- Model binding in ASP.NET Core — Microsoft Learn — https://learn.microsoft.com/en-us/aspnet/core/mvc/models/model-binding

---

## 9. Model Validation

### Definitions

**Core Definition**
Model validation checks that bound data satisfies the rules declared by validation attributes or custom validators.

**Technical Definition**
ASP.NET Core MVC uses **DataAnnotations** (`[Required]`, `[StringLength]`, `[Range]`, `[RegularExpression]`, `[EmailAddress]`, `[Compare]`) and **`IValidatableObject`** for validation. Validation runs during model binding and populates `ModelState`. Actions check `ModelState.IsValid`; in `[ApiController]` controllers, invalid models automatically return `400 Bad Request` with a `ProblemDetails` payload. Client-side validation is supported via jQuery validation unobtrusive attributes in Razor views.

**Beginner-Friendly Explanation**
Model validation is **automatic rule checking**. You annotate properties with rules, and MVC tells you whether the submitted data is valid.

---

### Purposes

- To **enforce input rules** declaratively.
- To **populate `ModelState`** with errors.
- To **re-render forms** with error messages.
- To **return standardized `400` responses** for APIs.

---

### Syntax Rules and Structure

```csharp
public class CreateProductDto
{
    [Required]
    [StringLength(100, MinimumLength = 2)]
    public string Name { get; set; } = string.Empty;

    [Range(0.01, 1_000_000)]
    public decimal Price { get; set; }
}
```

**Component breakdown**

- `[Required]` — Non-null/non-empty.
- `[StringLength]` — Length bounds.
- `[Range]` — Numeric range.
- `ModelState.IsValid` — Validation result.

#### Syntax Rules

- Validation attributes are checked after binding.
- `ModelState` contains errors keyed by property name.
- `[ApiController]` auto-returns `400` on invalid models.
- Custom validation implements `IValidatableObject` or `ValidationAttribute`.

#### Constraints and Limitations

- DataAnnotations are attribute-based; complex rules may require `IValidatableObject`.
- Client-side validation requires jQuery validation scripts.

---

### Annotated Code Examples and Expected Outputs

#### Example: Validation in a Controller

```csharp
[HttpPost]
public IActionResult Create(CreateProductDto dto)
{
    if (!ModelState.IsValid) return View(dto);
    _service.Create(dto);
    return RedirectToAction(nameof(Index));
}
```

**Expected Output**

- Valid data → `302 Found` redirect.
- Invalid data → `200 OK` with the form and validation messages.

**Why this output occurs**
`ModelState.IsValid` reflects the DataAnnotations validation results.

---

### Real-World Cases

- **Registration forms** — Email, password rules.
- **Checkout** — Required address fields.
- **APIs** — Automatic `400` responses with problem details.

---

### References

- Model validation in ASP.NET Core MVC — Microsoft Learn — https://learn.microsoft.com/en-us/aspnet/core/mvc/models/validation

---

## 10. MVC Best Practices

### Definitions

**Core Definition**
MVC best practices are the guidelines that keep MVC applications **maintainable, secure, testable, and performant**.

**Technical Definition**
Best practices include: **keep controllers thin** (delegate to services), **use view models/DTOs** instead of entities, **validate all input** with DataAnnotations or custom validators, **use the PRG (Post/Redirect/Get) pattern** for form submissions, **enable anti-forgery tokens** on state-changing requests, **prefer async actions** for I/O, **use attribute routing for APIs** and conventional routing for pages, **apply filters for cross-cutting concerns**, **avoid business logic in views**, and **use `IActionResult` return types** with meaningful status codes.

**Beginner-Friendly Explanation**
The best practices are: **keep controllers small**, **don't bind entities directly**, **validate inputs**, **redirect after POST**, **use anti-forgery tokens**, and **use async for I/O**.

---

### Guidelines and Structure

#### General Guidelines

```csharp
// 1. Thin controller
public async Task<IActionResult> Index()
{
    var vm = await _service.GetDashboardAsync();
    return View(vm);
}

// 2. View model, not entity
public IActionResult Details(int id) =>
    _service.GetDetails(id) is { } vm ? View(vm) : NotFound();

// 3. PRG pattern
[HttpPost]
[ValidateAntiForgeryToken]
public IActionResult Create(CreateDto dto)
{
    if (!ModelState.IsValid) return View(dto);
    _service.Create(dto);
    return RedirectToAction(nameof(Index));
}

// 4. Async actions for I/O
public async Task<IActionResult> Index() => View(await _service.GetAllAsync());

// 5. Attribute routing for APIs
[Route("api/products")]
public class ProductsApiController : ControllerBase { ... }
```

**Component breakdown**

- **Thin controllers** — Delegate to services.
- **View models** — Prevent over-posting and coupling.
- **PRG** — Avoid duplicate submissions.
- **Anti-forgery** — CSRF protection.
- **Async** — Scalability.

#### Rules and Constraints

- **Do** keep controllers focused on orchestration.
- **Do** use view models for views and DTOs for APIs.
- **Do** validate with DataAnnotations and `ModelState`.
- **Do** use `[ValidateAntiForgeryToken]` on POST actions.
- **Do** use async actions for I/O.
- **Do not** expose entities directly to views or APIs.
- **Do not** put business logic in views.

---

### Annotated Code Examples and Expected Outputs

#### Example: Well-Structured MVC Action

```csharp
[HttpPost]
[ValidateAntiForgeryToken]
public async Task<IActionResult> Create(CreateProductDto dto)
{
    if (!ModelState.IsValid) return View(dto);

    var created = await _service.CreateAsync(dto);
    TempData["Message"] = $"Created {created.Name}.";
    return RedirectToAction(nameof(Index));
}
```

**Expected Output**

- Valid: `302 Found` to `Index` with a `TempData` message.
- Invalid: `200 OK` with the form and validation errors.

**Why this is well-structured**
It validates input, uses async, protects against CSRF, and follows PRG.

---

### Real-World Cases

- **Enterprise forms** — PRG + anti-forgery + validation.
- **APIs** — DTOs + attribute routing + problem details.
- **Dashboards** — Thin controllers + view models.

---

### References

- ASP.NET Core MVC overview — Microsoft Learn — https://learn.microsoft.com/en-us/aspnet/core/mvc/overview
- Model validation in ASP.NET Core MVC — Microsoft Learn — https://learn.microsoft.com/en-us/aspnet/core/mvc/models/validation
- Filters in ASP.NET Core — Microsoft Learn — https://learn.microsoft.com/en-us/aspnet/core/mvc/controllers/filters

---

## Deprecated, Unsafe, Implementation-Dependent, and Version-Specific Notes

| Feature | Status / Note |
|---|---|
| `Startup.cs` | **Legacy**; replaced by top-level `Program.cs` in .NET 6+. |
| `IHostingEnvironment` | **Obsolete**; use `IWebHostEnvironment`. |
| `[ApiController]` | Recommended for API controllers; adds automatic `400` on invalid models and binding source inference. |
| `ViewBag` / `ViewData` | **Weakly typed**; prefer strongly typed view models. |
| Binding entities directly | **Over-posting risk**; use DTOs or `[Bind]`. |
| Synchronous actions for I/O | **Scalability risk**; prefer async. |
| `Html.BeginForm` | **Legacy** helper; prefer `<form asp-action="...">` tag helper. |
| `JsonResult` | Stable; prefer `TypedResults`/`Results` in Minimal APIs. |

---

## References (Consolidated)

- ASP.NET Core MVC overview — Microsoft Learn — https://learn.microsoft.com/en-us/aspnet/core/mvc/overview
- Handle requests with controllers in ASP.NET Core MVC — Microsoft Learn — https://learn.microsoft.com/en-us/aspnet/core/mvc/controllers/actions
- Views in ASP.NET Core MVC — Microsoft Learn — https://learn.microsoft.com/en-us/aspnet/core/mvc/views/overview
- Routing to controller actions in ASP.NET Core — Microsoft Learn — https://learn.microsoft.com/en-us/aspnet/core/mvc/controllers/routing
- Filters in ASP.NET Core — Microsoft Learn — https://learn.microsoft.com/en-us/aspnet/core/mvc/controllers/filters
- Model binding in ASP.NET Core — Microsoft Learn — https://learn.microsoft.com/en-us/aspnet/core/mvc/models/model-binding
- Model validation in ASP.NET Core MVC — Microsoft Learn — https://learn.microsoft.com/en-us/aspnet/core/mvc/models/validation

---