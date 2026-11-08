# ASP.NET MVC Routing Fundamentals: A Comprehensive Cheat Sheet

> **Scope & Version Note:** This cheat sheet focuses on **ASP.NET Core MVC** (the modern, cross‑platform successor to ASP.NET MVC 5). Where a feature differs significantly in legacy ASP.NET MVC 5 (e.g., `System.Web.Routing`), it is explicitly marked. ASP.NET Core 3.0+ uses **endpoint routing** by default; the legacy routing logic can still be enabled via `MvcOptions.EnableEndpointRouting = false` for backward compatibility.

---

## Topic Overview

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Routing is the mechanism that maps incoming HTTP requests to executable endpoints (controller actions, Razor Pages, minimal‑API delegates, etc.) based on URL patterns. |
| **Technical Definition** | In ASP.NET Core, routing is a middleware (`UseRouting`) that tokenises the request’s URL path against a set of registered **route templates**, extracts **route values**, applies **route constraints** and **matcher policies**, and finally selects the highest‑priority endpoint via `EndpointSelector`. The selected endpoint is then executed by `UseEndpoints`. |
| **Beginner‑Friendly Explanation** | Think of routing as a **receptionist** for your web application. When a request arrives with a URL like `/Products/Details/5`, the receptionist looks at a list of rules (route templates) and decides which piece of code (controller action) should handle that request. The URL `/Products/Details/5` tells the receptionist: “Go to the Products controller, run the Details action, and pass 5 as the id.” |

### Key Characteristics

- **Convention‑based or attribute‑based** – Routes can be defined centrally in `Program.cs` (conventional) or decoratively on controllers/actions (`[Route]`, `[HttpGet]`).
- **Endpoint‑centric** – In ASP.NET Core 3.0+, every route maps to an **Endpoint** (a delegate + metadata) rather than to a controller/action pair directly.
- **Template‑driven** – URLs are described with **route templates** that contain literal segments, parameter placeholders, optional markers, defaults, and constraints.
- **Constraint‑aware** – Route parameters can be validated with built‑in or custom constraints (`int`, `guid`, `regex`, `minlength`, etc.).
- **Precedence‑based** – The routing system computes a **route precedence** score so that more specific templates win over generic ones without manual ordering.
- **Extensible** – Custom constraints (`IRouteConstraint`), custom route attributes (`IRouteTemplateProvider`), and matcher policies can be plugged in.

### Prerequisites

- **.NET SDK** (6.0, 7.0, 8.0, or later recommended; examples use .NET 8 minimal hosting).
- **ASP.NET Core MVC** workload (`Microsoft.AspNetCore.Mvc`).
- Basic knowledge of **C#**, **HTTP verbs** (GET, POST, PUT, DELETE), and **controllers**.
- Familiarity with **middleware pipeline** concepts (optional but helpful).

### Related Programming Areas

- **Model‑View‑Controller (MVC)** – Routing is the entry point that selects the controller and action.
- **RESTful API design** – Attribute routing is the preferred style for APIs.
- **URL generation** – `LinkGenerator`, `IUrlHelper`, and Tag Helpers rely on routing metadata to produce links.
- **Authorization & CORS** – These middleware components run **after** routing and inspect the selected endpoint’s metadata.
- **Razor Pages** – Uses the same routing infrastructure with page‑centric conventions.

### Core Concepts / Features

1. **Routing** – The overall framework that matches requests to endpoints.
2. **Route Matching** – The multi‑phase algorithm that narrows down candidate endpoints.
3. **Route Templates** – The syntax used to describe URL patterns.
4. **Route Parameters** – Named placeholders inside templates that capture URL segments.
5. **Route Constraints** – Validation rules applied to route parameters.
6. **Routing Best Practices** – Guidelines for maintainable, performant, and secure route design.

Each concept is elaborated below using a **uniform structure**: Definitions, Purposes, Syntax Rules & Structure, Annotated Code Examples, Real‑World Cases, and References.

---

## 1. Routing

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Routing is the process of taking an incoming HTTP request and deciding which handler (endpoint) should process it. |
| **Technical Definition** | In ASP.NET Core, routing is implemented by the `RoutingMiddleware` (`UseRouting`) and the `EndpointMiddleware` (`UseEndpoints`). The routing middleware parses the request path, matches it against the **EndpointDataSource**, applies constraints and matcher policies, and stores the selected `Endpoint` in `HttpContext`. The endpoint middleware then invokes the endpoint’s `RequestDelegate`. |
| **Beginner‑Friendly Explanation** | Routing is the **map** that tells your application: “When a user visits this URL, run that code.” Without routing, every URL would need to point to a physical file. Routing lets you create clean, descriptive URLs like `/products/list` instead of `/productlist.aspx`. |

### Purposes (each begins with “To”)

- **To** decouple URLs from physical file names, enabling clean, SEO‑friendly, and user‑readable URLs.
- **To** centralise the mapping between HTTP requests and application logic (controllers, Razor Pages, minimal APIs).
- **To** extract variable data (route values) from the URL without relying on query strings.
- **To** enable URL generation (reverse routing) so that links can be produced consistently across the application.
- **To** provide a single interception point for cross‑cutting concerns (authorization, CORS, logging) via endpoint metadata.

### Syntax Rules and Structure

#### General Syntax (Conventional Routing in `Program.cs`)

```csharp
app.MapControllerRoute(
    name: "default",
    pattern: "{controller=Home}/{action=Index}/{id?}"
);
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `MapControllerRoute` | Extension method that registers a conventional route for MVC controllers. |
| `name` | A unique name for the route (used for URL generation). |
| `pattern` | The route template string. |
| `{controller=Home}` | Parameter `controller` with a default value `Home`. |
| `{action=Index}` | Parameter `action` with a default value `Index`. |
| `{id?}` | Optional parameter `id` (the `?` suffix makes it optional). |

#### General Syntax (Attribute Routing)

```csharp
[Route("api/[controller]")]
[ApiController]
public class ProductsController : ControllerBase
{
    [HttpGet("{id:int}")]
    public IActionResult Get(int id) { ... }
}
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `[Route("api/[controller]")]` | Controller‑level route template; `[controller]` is a token replaced by the controller name (without the “Controller” suffix). |
| `[ApiController]` | Enables API‑specific behaviours (automatic 400 responses, binding source inference). |
| `[HttpGet("{id:int}")]` | Action‑level route; combines with the controller template → `api/Products/{id:int}`. |

#### Syntax Rules

- Route templates are **case‑insensitive**.
- Literal segments are matched exactly; parameter segments capture values.
- A template **must not** start with `/` or `~/` when used in attribute routing (ASP.NET Core normalises them).
- Conventional routes are typically registered in `Program.cs`; attribute routes are discovered from controllers at startup.
- The order of conventional route registration **does** matter for legacy routing, but **endpoint routing** processes all endpoints simultaneously and relies on **precedence** rather than registration order.

#### Constraints and Limitations

- **Endpoint routing** (ASP.NET Core 3.0+) does **not** guarantee ordering based on registration; use `Order` only when necessary.
- **Legacy routing** (`EnableEndpointRouting = false`) preserves the older `System.Web.Routing` behaviour but is considered legacy and should be avoided for new development.
- Route templates cannot contain **consecutive parameter segments** without a literal delimiter (e.g., `{a}{b}` is invalid; use `{a}-{b}` or complex segments).
- **Catch‑all parameters** (`{*path}`) must be the last segment of a template.

### Multiple Annotated Complete Step‑by‑Step Code Examples

#### Example 1: Conventional Routing with Defaults

**Setup:** Create an ASP.NET Core MVC project (`dotnet new mvc`). Replace `Program.cs` with the following:

`Program.cs`
```csharp
var builder = WebApplication.CreateBuilder(args);
builder.Services.AddControllersWithViews(); // Register MVC services

var app = builder.Build();

app.UseRouting(); // 1. Enable routing middleware

// 2. Define a conventional route
app.MapControllerRoute(
    name: "default",
    pattern: "{controller=Home}/{action=Index}/{id?}"
);

app.UseEndpoints(endpoints => { }); // Not strictly needed in .NET 6+ minimal hosting

app.Run();
```

**Step‑by‑step explanation:**

1. `AddControllersWithViews()` registers the MVC services required for controllers and views.
2. `UseRouting()` adds the routing middleware to the pipeline. It parses the incoming URL and selects an endpoint.
3. `MapControllerRoute` registers a conventional route template. The template `{controller=Home}/{action=Index}/{id?}` means:
   - If no controller is specified, use `Home`.
   - If no action is specified, use `Index`.
   - `id` is optional.
4. When a request arrives, the routing middleware tries to match the URL against this template.

**Expected Output:**

| Request URL | Matched Controller | Matched Action | Route Values |
|-------------|-------------------|----------------|--------------|
| `/` | `HomeController` | `Index` | `{}` |
| `/Products` | `ProductsController` | `Index` | `{controller = "Products"}` |
| `/Products/Details/5` | `ProductsController` | `Details` | `{controller = "Products", action = "Details", id = "5"}` |

**Why this output?** The template provides default values for `controller` and `action`, and the `id` parameter is optional (`?`). The routing system fills in missing segments from defaults.

---

#### Example 2: Attribute Routing on a Controller

```csharp
// Controllers/ProductsController.cs
using Microsoft.AspNetCore.Mvc;

[Route("api/[controller]")]          // → api/Products
[ApiController]
public class ProductsController : ControllerBase
{
    [HttpGet]                        // GET api/Products
    public IActionResult GetAll() => Ok(new[] { "Product1", "Product2" });

    [HttpGet("{id:int}")]            // GET api/Products/5
    public IActionResult GetById(int id) => Ok($"Product {id}");

    [HttpPost]                       // POST api/Products
    public IActionResult Create([FromBody] string product) => Created("", product);
}
```

**Step‑by‑step explanation:**

1. The `[Route("api/[controller]")]` attribute sets the base template for all actions in the controller. The `[controller]` token is replaced with `Products` (the controller name without the `Controller` suffix).
2. `[HttpGet]` on `GetAll` creates a route for `GET api/Products`.
3. `[HttpGet("{id:int}")]` on `GetById` creates a route `GET api/Products/{id}` and applies the `int` constraint to `id`.
4. `[HttpPost]` on `Create` creates a route for `POST api/Products`.

**Expected Output:**

| HTTP Method | URL | Action Invoked |
|-------------|-----|----------------|
| GET | `/api/Products` | `GetAll` |
| GET | `/api/Products/7` | `GetById(7)` |
| GET | `/api/Products/abc` | **404** (constraint fails) |
| POST | `/api/Products` | `Create` |

**Why this output?** Attribute routing combines controller‑level and action‑level templates. The `int` constraint rejects non‑integer values, so `/api/Products/abc` does not match any endpoint.

---

#### Example 3: Minimal API Routing (Endpoint Routing Without Controllers)

```csharp
// Program.cs
var builder = WebApplication.CreateBuilder(args);
var app = builder.Build();

app.MapGet("/", () => "Hello World!");                         // GET /
app.MapGet("/products/{id:int}", (int id) => $"Product {id}"); // GET /products/5
app.MapPost("/products", (Product p) => Results.Created($"/products/{p.Id}", p));

app.Run();

record Product(int Id, string Name);
```

**Step‑by‑step explanation:**

1. `MapGet("/", ...)` registers an endpoint that responds to `GET /`.
2. `MapGet("/products/{id:int}", ...)` registers an endpoint with a route parameter `id` constrained to integers.
3. `MapPost("/products", ...)` registers an endpoint for `POST /products`, binding the JSON body to a `Product` record.

**Expected Output:**

| Request | Response |
|---------|----------|
| `GET /` | `Hello World!` |
| `GET /products/3` | `Product 3` |
| `GET /products/abc` | 404 (constraint fails) |
| `POST /products` (with JSON body) | 201 Created |

**Why this output?** Minimal APIs use the same routing infrastructure as controllers. The `int` constraint on `{id:int}` ensures that only integer IDs match.

### Real‑World Cases

- **E‑commerce product catalogue:** Clean URLs like `/products/electronics/123` map to a `ProductsController.Details(category, id)` action. Routing extracts `category` and `id` as route values.
- **REST API versioning:** Attribute routes such as `[Route("api/v{version:apiVersion}/products")]` allow multiple API versions to coexist.
- **CMS with dynamic pages:** A catch‑all route `{*path}` can serve content from a database when no physical file exists.

### References

- Routing in ASP.NET Core – https://learn.microsoft.com/en-us/aspnet/core/fundamentals/routing
- Routing to controller actions in ASP.NET Core – https://learn.microsoft.com/en-us/aspnet/core/mvc/controllers/routing
- ASP.NET Core Middleware – https://learn.microsoft.com/en-us/aspnet/core/fundamentals/middleware

---

## 2. Route Matching

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Route matching is the algorithm that compares an incoming request’s URL path against all registered route templates and selects the best‑matching endpoint. |
| **Technical Definition** | In ASP.NET Core endpoint routing, matching proceeds in **phases**: (1) collect all endpoints whose templates match the URL, (2) remove matches that fail **route constraints**, (3) remove matches that fail **MatcherPolicy** instances, and (4) use `EndpointSelector` to choose the highest‑precedence match. If two matches share the same precedence, an `AmbiguousMatchException` is thrown. |
| **Beginner‑Friendly Explanation** | Imagine you have a set of stencils (route templates). When a URL arrives, the router holds each stencil against the URL. If the URL “fits” the stencil’s shape, it’s a candidate. Then the router applies extra filters (constraints) to eliminate candidates that don’t meet the rules. Finally, it picks the stencil that fits most precisely. |

### Purposes

- **To** deterministically select the correct controller action or endpoint for every request.
- **To** support multiple URL patterns that may overlap (e.g., `/Products/List` and `/Products/{id}`) by choosing the most specific one.
- **To** reject URLs that do not conform to expected data types (via constraints) before the action executes.
- **To** provide a predictable and testable mapping between URLs and application logic.

### Syntax Rules and Structure

Route matching is not configured with a single syntax; it is the **result** of the route templates and constraints you define. However, the following elements influence matching:

#### General Syntax (Conceptual)

```
For each incoming request:
    candidates = all endpoints whose template matches the URL path
    candidates = candidates where all route constraints return true
    candidates = candidates where all MatcherPolicies return true
    if candidates.Count == 0 → 404
    if candidates.Count == 1 → select that endpoint
    else → select endpoint with highest route precedence
           (if tie → AmbiguousMatchException)
```

**Component breakdown:**

| Component | Role in Matching |
|-----------|------------------|
| **Route template** | Defines the shape of the URL (literal segments, parameter segments, optional segments, catch‑alls). |
| **Route constraints** | Boolean predicates that validate route parameter values. |
| **MatcherPolicy** | Extensible policies (e.g., HTTP method matching, host matching) that can filter endpoints. |
| **Route precedence** | A computed score reflecting how specific a template is. |

#### Syntax Rules

- Matching is **case‑insensitive**.
- **Literal segments** take priority over **parameter segments**.
- A parameter with a **constraint** is more specific than one without.
- **More segments** → more specific.
- **Catch‑all parameters** (`{*path}`) are the least specific.
- `Order` can be used to override precedence, but should be used sparingly.

#### Constraints and Limitations

- **Ambiguous matches** throw an exception at runtime if two endpoints have the same precedence and both match.
- The matching algorithm does **not** guarantee a processing order of endpoints; all candidates are processed simultaneously.
- In **legacy routing** (pre‑ASP.NET Core 3.0), matching was order‑dependent; this behaviour is not replicated in endpoint routing.

### Multiple Annotated Complete Step‑by‑Step Code Examples

#### Example 1: Precedence – Literal vs Parameter

`Program.cs`
```csharp
var builder = WebApplication.CreateBuilder(args);
builder.Services.AddControllersWithViews();
var app = builder.Build();

app.UseRouting();

app.MapControllerRoute(
    name: "default",
    pattern: "{controller=Home}/{action=Index}/{id?}");

app.Run();
```

`Controllers/ProductsController.cs`
```csharp
public class ProductsController : Controller
{
    public IActionResult List() => Content("Product List");
    public IActionResult Details(int id) => Content($"Product {id}");
}
```

**Step‑by‑step explanation:**

- The conventional route `{controller}/{action}/{id?}` can match both `/Products/List` and `/Products/Details/5`.
- For the URL `/Products/List`, two endpoints are candidates:
  - `ProductsController.List` (template effectively `/Products/List` – literal `List`).
  - `ProductsController.Details` with `id = "List"` (template `/Products/{action}/{id?}` – parameter `action`).
- The routing system computes precedence: a literal segment (`List`) is more specific than a parameter segment (`{action}`).
- Therefore, `/Products/List` selects the `List` action.

**Expected Output:**

| URL | Action Selected | Response |
|-----|----------------|----------|
| `/Products/List` | `List` | `Product List` |
| `/Products/Details/5` | `Details(5)` | `Product 5` |

**Why this output?** The literal segment `List` outranks the parameter segment `{action}` in precedence, so the `List` action wins.

---

#### Example 2: Ambiguous Match and How to Resolve It

```csharp
// Two attribute routes with the same precedence
[Route("api/[controller]")]
[ApiController]
public class ValuesController : ControllerBase
{
    [HttpGet("all")]               // → api/Values/all
    public IActionResult GetAll() => Ok("All values");

    [HttpGet("{name}")]            // → api/Values/{name}
    public IActionResult GetByName(string name) => Ok($"Value: {name}");
}
```

**Step‑by‑step explanation:**

- `GET /api/Values/all` could match both:
  - `GetAll` (literal `all`).
  - `GetByName` with `name = "all"`.
- Because `all` is literal, it has **higher precedence** than `{name}`. The match is **not ambiguous** – `GetAll` wins.
- However, if you had two parameter routes with identical constraints and precedence (e.g., `{id:int}` and `{id:guid}` on the same template), an `AmbiguousMatchException` would be thrown at runtime for a URL that satisfies both.

**Resolution:** Use `Order` to disambiguate when precedence alone cannot:

```csharp
[HttpGet("{id:int}", Order = 1)]
public IActionResult GetById(int id) { ... }

[HttpGet("{slug}", Order = 2)]
public IActionResult GetBySlug(string slug) { ... }
```

**Expected Output:**

| URL | Result |
|-----|--------|
| `/api/Values/all` | `All values` (literal wins) |
| `/api/Values/42` | `GetById(42)` (constraint matches) |
| `/api/Values/hello` | `GetBySlug("hello")` |

**Why this output?** The `int` constraint on the first route rejects `"hello"`, so only the slug route remains for that URL. The `Order` property ensures a deterministic tie‑break when needed.

### Real‑World Cases

- **Multi‑tenant applications:** Host‑based matching policies select different endpoints based on the subdomain.
- **API versioning:** Routes like `/api/v1/products` and `/api/v2/products` are distinguished by literal segments, which have high precedence.
- **SPA fallback:** A catch‑all route (`{*path}`) with low precedence serves the SPA shell for client‑side routes, while API routes with higher precedence handle server calls.

### References

- Routing in ASP.NET Core – URL matching phases – https://learn.microsoft.com/en-us/aspnet/core/fundamentals/routing#url-matching
- Route template precedence – https://learn.microsoft.com/en-us/aspnet/core/fundamentals/routing#route-template-precedence-and-endpoint-selection-order

---

## 3. Route Templates

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | A route template is a string pattern that describes the structure of URLs that an endpoint can handle. |
| **Technical Definition** | A route template consists of **literal segments**, **parameter segments** (enclosed in `{}`), **optional markers** (`?`), **default values** (`=`), **constraints** (`:`), and **catch‑all parameters** (`*`). It is parsed into a `RoutePattern` object used by the matcher and link generator. |
| **Beginner‑Friendly Explanation** | A route template is like a **fill‑in‑the‑blank sentence**. For example, `products/{category}/{id?}` means: “The URL starts with `products`, then has a category (any word), and optionally an ID.” The router uses this sentence to understand which part of the URL is which. |

### Purposes

- **To** describe URL patterns in a human‑readable, declarative way.
- **To** capture variable data from the URL as named route values.
- **To** enforce a consistent URL structure across an application.
- **To** enable reverse URL generation (given route values, produce a URL).
- **To** support optional segments and default values so that a single template can handle multiple URL shapes.

### Syntax Rules and Structure

#### General Syntax

```
[literal]/[literal]/.../{parameter[=default][:constraint]}[?]...
```

#### Component Breakdown

| Component | Syntax | Example | Meaning |
|-----------|--------|---------|---------|
| **Literal segment** | Plain text | `products` | Must match exactly. |
| **Parameter segment** | `{name}` | `{id}` | Captures the segment value into route value `id`. |
| **Optional parameter** | `{name?}` | `{id?}` | The segment may be omitted. |
| **Default value** | `{name=value}` | `{action=Index}` | If the segment is missing, use `value`. |
| **Inline constraint** | `{name:constraint}` | `{id:int}` | The captured value must satisfy the constraint. |
| **Catch‑all parameter** | `{*name}` | `{*path}` | Captures the remainder of the URL (including slashes). |
| **Complex segment** | `{name=value:constraint}` | `{id=1:int}` | Combines default and constraint. |

#### Syntax Rules

- Parameter names must be valid C# identifiers.
- A template **cannot** contain two consecutive parameter segments without a delimiter (e.g., `{a}{b}` is invalid).
- The `?` optional marker must come **after** the parameter name and any default/constraint.
- Catch‑all parameters must be the **last** segment.
- In attribute routing, the leading `/` is optional; in conventional routing, it is not used.
- **`[controller]`** and **`[action]`** tokens are replaced with the controller and action names respectively (attribute routing only).

#### Constraints and Limitations

- **Empty segments** are not matched by optional parameters; `{id?}` matches `/products` but not `/products/`.
- **Default values** are used only when the parameter is absent from the URL; they do not override a present but empty value.
- **Catch‑all parameters** can match an empty string but may cause performance issues if overused.
- **Complex segments** should be used with caution; they can make URLs hard to read and debug.

### Multiple Annotated Complete Step‑by‑Step Code Examples

#### Example 1: Optional Parameter and Default Value

```csharp
// Program.cs
app.MapControllerRoute(
    name: "blog",
    pattern: "blog/{category=all}/{page?}");
```

**Step‑by‑step explanation:**

- `blog` is a literal segment.
- `{category=all}` means: if the category segment is missing, use `"all"`.
- `{page?}` means the page segment is optional.

**Expected Output:**

| URL | Route Values |
|-----|--------------|
| `/blog` | `category = "all"`, `page = null` |
| `/blog/tech` | `category = "tech"`, `page = null` |
| `/blog/tech/2` | `category = "tech"`, `page = "2"` |

**Why this output?** The default value fills in `category` when it is absent, and the optional marker allows `page` to be omitted.

---

#### Example 2: Inline Constraint in a Template

```csharp
[HttpGet("products/{id:int}")]
public IActionResult GetProduct(int id) { ... }
```

**Step‑by‑step explanation:**

- The template is `products/{id:int}`.
- `products` is literal.
- `{id:int}` captures the next segment and applies the `int` constraint.

**Expected Output:**

| URL | Match? | Reason |
|-----|--------|--------|
| `/products/42` | Yes | `42` is an integer. |
| `/products/abc` | No | `abc` is not an integer. |

**Why this output?** The `int` constraint returns `true` only for integer values, so non‑integer segments are rejected before the action is selected.

---

#### Example 3: Catch‑All Parameter

```csharp
[Route("files/{*path}")]
public IActionResult GetFile(string path) { ... }
```

**Step‑by‑step explanation:**

- `{*path}` is a catch‑all parameter. It captures everything after `/files/`, including slashes.
- The captured value is URL‑decoded and assigned to the `path` parameter.

**Expected Output:**

| URL | `path` Value |
|-----|--------------|
| `/files/docs/report.pdf` | `docs/report.pdf` |
| `/files/images/logo.png` | `images/logo.png` |

**Why this output?** Catch‑all parameters consume the remaining path segments, making them ideal for file‑serving or SPA fallback routes.

### Real‑World Cases

- **Blog engine:** `blog/{category=all}/{page?}` provides a default “all categories” view and optional pagination.
- **File server:** `files/{*path}` serves static or dynamic files from a virtual directory structure.
- **Localised URLs:** `{culture=en-US}/{controller=Home}/{action=Index}` allows language‑specific routes.

### References

- Route templates – https://learn.microsoft.com/en-us/aspnet/core/fundamentals/routing#route-templates
- Route template reference – https://learn.microsoft.com/en-us/aspnet/core/mvc/controllers/routing#route-template-reference

---

## 4. Route Parameters

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Route parameters are named placeholders in a route template that capture values from the URL and make them available to the action method. |
| **Technical Definition** | A route parameter is defined by `{name}` inside a template. During matching, the segment corresponding to the placeholder is extracted and stored in `RouteData.Values` (or `HttpContext.Request.RouteValues` in endpoint routing). The value is then bound to action parameters by the model binder. |
| **Beginner‑Friendly Explanation** | Route parameters are the **blanks** in a fill‑in‑the‑blank sentence. In `products/{id}`, the `{id}` is a blank. If the URL is `/products/5`, the router fills the blank with `5` and hands it to your action method as the `id` argument. |

### Purposes

- **To** pass variable data from the URL to the server without using query strings.
- **To** create descriptive, hierarchical URLs that reflect the application’s data model.
- **To** enable model binding to automatically populate action method parameters.
- **To** support link generation by providing the values needed to construct a URL.

### Syntax Rules and Structure

#### General Syntax

```
{parameterName}
{parameterName?}          // optional
{parameterName=default}   // default value
{parameterName:constraint} // inline constraint
```

#### Component Breakdown

| Component | Description |
|-----------|-------------|
| `parameterName` | A valid C# identifier; used as the key in route values. |
| `?` | Marks the parameter as optional. |
| `=default` | Provides a default value if the segment is missing. |
| `:constraint` | Applies a constraint to validate the captured value. |

#### Syntax Rules

- Parameter names are **case‑insensitive** during matching but are exposed with the casing used in the template.
- A parameter can have at most **one** default value and **multiple** constraints (chained with `:`).
- Optional parameters must appear **after** required parameters in the same segment.
- Parameters cannot span multiple segments unless they are catch‑all parameters (`{*name}`).

#### Constraints and Limitations

- **Empty values** are not captured by optional parameters; `/products/` does not match `products/{id?}`.
- Parameter values are **URL‑decoded** before being placed in route values.
- Very long parameter values can hit URL length limits imposed by servers/proxies.
- Parameters with the same name in different segments are not allowed in a single template.

### Multiple Annotated Complete Step‑by‑Step Code Examples

#### Example 1: Basic Route Parameter Binding

```csharp
[Route("products")]
public class ProductsController : Controller
{
    [HttpGet("{id}")]
    public IActionResult Details(int id)
    {
        return Content($"Product ID: {id}");
    }
}
```

**Step‑by‑step explanation:**

- The action route is `products/{id}`.
- A request to `/products/10` captures `id = "10"`.
- Model binding converts the string `"10"` to the integer `10` for the action parameter.

**Expected Output:**

| URL | Response |
|-----|----------|
| `/products/10` | `Product ID: 10` |
| `/products/abc` | **404** (model binding fails; no matching `int` endpoint) |

**Why this output?** The implicit `int` binding attempts to convert the route value. If conversion fails, the request does not match the action (in attribute routing, the constraint is not automatically added; however, ASP.NET Core may return a 400 if the action is reached and binding fails. To guarantee rejection, add `:int` to the template).

---

#### Example 2: Multiple Parameters and Optional Values

```csharp
[HttpGet("orders/{year:int}/{month:int}/{day:int?}")]
public IActionResult GetOrders(int year, int month, int? day)
{
    return Content($"Year: {year}, Month: {month}, Day: {day?.ToString() ?? "N/A"}");
}
```

**Step‑by‑step explanation:**

- The template has three parameters: `year` (required, `int`), `month` (required, `int`), `day` (optional, `int`).
- The action signature uses `int?` for `day` to accommodate the optional value.

**Expected Output:**

| URL | Response |
|-----|----------|
| `/orders/2025/3/15` | `Year: 2025, Month: 3, Day: 15` |
| `/orders/2025/3` | `Year: 2025, Month: 3, Day: N/A` |
| `/orders/2025/abc` | **404** (constraint fails) |

**Why this output?** The `int` constraints on `year` and `month` reject non‑numeric values. The optional `day` parameter is `null` when absent, which is reflected in the response.

---

#### Example 3: Route Parameter with Default Value

```csharp
app.MapControllerRoute(
    name: "default",
    pattern: "{controller=Home}/{action=Index}/{id?}");
```

**Step‑by‑step explanation:**

- `{controller=Home}` and `{action=Index}` have default values.
- `{id?}` is optional.
- A request to `/` populates `controller = "Home"`, `action = "Index"`, and `id = null`.

**Expected Output:**

| URL | Route Values |
|-----|--------------|
| `/` | `controller = "Home"`, `action = "Index"`, `id = null` |
| `/Products` | `controller = "Products"`, `action = "Index"`, `id = null` |
| `/Products/List/5` | `controller = "Products"`, `action = "List"`, `id = "5"` |

**Why this output?** Defaults fill in missing segments, and the optional marker allows `id` to be omitted entirely.

### Real‑World Cases

- **RESTful resource identifiers:** `/api/customers/{customerId}/orders/{orderId}` captures hierarchical resource identifiers.
- **Date‑based archives:** `/blog/archive/{year:int}/{month:int?}` supports optional month filtering.
- **E‑commerce categories:** `/shop/{category}/{subcategory?}` handles both broad and narrow category browsing.

### References

- Route parameters – https://learn.microsoft.com/en-us/aspnet/core/fundamentals/routing#route-parameters
- Model binding in ASP.NET Core – https://learn.microsoft.com/en-us/aspnet/core/mvc/models/model-binding

---

## 5. Route Constraints

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | A route constraint is a rule that a route parameter’s value must satisfy for the route to match. |
| **Technical Definition** | A route constraint implements `IRouteConstraint` (or is one of the built‑in constraints). It is applied inline (`{id:int}`) or via a `RouteValueDictionary`. During matching, the constraint’s `Match` method returns `true` or `false`; if `false`, the candidate endpoint is discarded. |
| **Beginner‑Friendly Explanation** | A route constraint is like a **security guard** at the door of a route. It checks the value coming from the URL and says “You’re allowed in” only if the value meets the rule (e.g., must be a number, must match a pattern). If the value fails the check, the route is not used. |

### Purposes

- **To** prevent invalid data from reaching action methods (e.g., non‑numeric IDs).
- **To** distinguish between overlapping route templates by data type or format.
- **To** improve performance by rejecting non‑matching URLs early in the matching pipeline.
- **To** enforce business rules at the routing layer (e.g., IDs must be positive, slugs must match a regex).
- **To** enable cleaner action method signatures by guaranteeing that certain parameters are of the correct type.

### Syntax Rules and Structure

#### General Syntax (Inline)

```
{parameter:constraintName}
{parameter:constraintName(argument)}
{parameter:constraint1:constraint2}   // multiple constraints
```

#### General Syntax (Fluent / RouteValueDictionary)

```csharp
routes.MapRoute(
    name: "default",
    template: "{controller=Home}/{action=Index}/{id?}",
    defaults: new { controller = "Home", action = "Index" },
    constraints: new { id = new IntRouteConstraint() }
);
```

#### Built‑in Constraints (Selected)

| Constraint | Example | Matches | Notes |
|------------|---------|---------|-------|
| `int` | `{id:int}` | `123`, `-45` | 32‑bit integer. |
| `bool` | `{active:bool}` | `true`, `FALSE` | Case‑insensitive. |
| `datetime` | `{date:datetime}` | `2025-03-15` | Invariant culture; use with caution. |
| `decimal` | `{price:decimal}` | `19.99` | |
| `double` | `{value:double}` | `3.14` | |
| `float` | `{value:float}` | `3.14` | |
| `guid` | `{id:guid}` | `xxxxxxxx-xxxx-...` | |
| `long` | `{id:long}` | `1234567890123` | |
| `minlength(n)` | `{name:minlength(4)}` | `John` | Minimum string length. |
| `maxlength(n)` | `{name:maxlength(10)}` | `Alice` | Maximum string length. |
| `length(n)` | `{code:length(5)}` | `ABCDE` | Exact string length. |
| `min(n)` | `{age:min(18)}` | `21` | Minimum numeric value. |
| `max(n)` | `{age:max(65)}` | `60` | Maximum numeric value. |
| `range(min,max)` | `{age:range(18,65)}` | `30` | Numeric range. |
| `alpha` | `{name:alpha}` | `John` | Alphabetic characters only. |
| `regex(expression)` | `{ssn:regex(^\\d{{3}}-\\d{{2}}-\\d{{4}}$)}` | `123-45-6789` | Regular expression. |
| `required` | `{name:required}` | Any non‑null value | Ensures the parameter is present. |

#### Syntax Rules

- Constraints are **case‑insensitive** (`int` and `Int` are equivalent).
- Multiple constraints can be chained: `{id:int:min(1)}`.
- Constraint arguments are passed in parentheses: `{name:minlength(3)}`.
- Constraints can be applied to **optional** parameters; if the parameter is absent, the constraint is not evaluated.
- Custom constraints must be registered in `RouteOptions.ConstraintMap` before use.

#### Constraints and Limitations

- **Regex constraints** can be vulnerable to **ReDoS** (Regular Expression Denial of Service) if used with untrusted input; always specify a timeout or use simple patterns.
- The `datetime` constraint uses the **invariant culture**, which may not match user expectations; consider using a custom constraint for locale‑specific dates.
- Constraints are **not** a substitute for **validation**; they only affect route matching. Action methods should still validate inputs.
- **Custom constraints** are rarely needed; built‑in constraints cover most scenarios.

### Multiple Annotated Complete Step‑by‑Step Code Examples

#### Example 1: Built‑in `int` and `min` Constraints

```csharp
[HttpGet("products/{id:int:min(1)}")]
public IActionResult GetProduct(int id)
{
    return Content($"Product {id}");
}
```

**Step‑by‑step explanation:**

- `{id:int:min(1)}` applies two constraints:
  1. `int` – the value must be a 32‑bit integer.
  2. `min(1)` – the integer must be ≥ 1.
- Only positive integers match.

**Expected Output:**

| URL | Match? | Reason |
|-----|--------|--------|
| `/products/5` | Yes | Integer ≥ 1. |
| `/products/0` | No | `min(1)` fails. |
| `/products/-3` | No | `min(1)` fails. |
| `/products/abc` | No | `int` fails. |

**Why this output?** The constraints are evaluated in order; if any fails, the route is rejected. This prevents invalid IDs from reaching the action.

---

#### Example 2: Regex Constraint for a Slug

```csharp
[HttpGet("blog/{slug:regex(^[a-z0-9-]+$)}")]
public IActionResult GetPost(string slug)
{
    return Content($"Post: {slug}");
}
```

**Step‑by‑step explanation:**

- `regex(^[a-z0-9-]+$)` allows only lowercase letters, digits, and hyphens.
- The `^` and `$` anchors ensure the entire segment matches.

**Expected Output:**

| URL | Match? |
|-----|--------|
| `/blog/my-first-post` | Yes |
| `/blog/My-Post` | No (uppercase `M`) |
| `/blog/post_1` | No (underscore not allowed) |

**Why this output?** The regex constraint enforces the slug format. Uppercase letters and underscores cause the match to fail.

---

#### Example 3: Custom `IRouteConstraint`

```csharp
// Custom constraint: only even integers are allowed.
public class EvenIntConstraint : IRouteConstraint
{
    public bool Match(HttpContext? httpContext, IRouter? route, string routeKey,
                      RouteValueDictionary values, RouteDirection routeDirection)
    {
        if (!values.TryGetValue(routeKey, out var value)) return false;
        return int.TryParse(value?.ToString(), out var number) && number % 2 == 0;
    }
}

// Registration (Program.cs)
builder.Services.Configure<RouteOptions>(options =>
{
    options.ConstraintMap.Add("evenint", typeof(EvenIntConstraint));
});

// Usage
[HttpGet("orders/{id:evenint}")]
public IActionResult GetOrder(int id) => Content($"Order {id}");
```

**Step‑by‑step explanation:**

1. `EvenIntConstraint` implements `IRouteConstraint`. Its `Match` method parses the route value and returns `true` only for even integers.
2. The constraint is registered in `RouteOptions.ConstraintMap` with the key `"evenint"`.
3. The route template uses `{id:evenint}` to apply the custom constraint.

**Expected Output:**

| URL | Match? |
|-----|--------|
| `/orders/4` | Yes |
| `/orders/3` | No (odd) |
| `/orders/abc` | No (not an integer) |

**Why this output?** The custom constraint’s logic is executed during matching. Only even integers satisfy it.

### Real‑World Cases

- **E‑commerce product IDs:** `{id:int:min(1)}` ensures only positive integer IDs are routed to product‑detail actions.
- **SEO‑friendly slugs:** `{slug:regex(^[a-z0-9-]+$)}` guarantees that blog post URLs contain only valid characters.
- **Date‑based APIs:** `{date:datetime}` restricts the `date` parameter to a parseable date, preventing 500 errors from invalid date strings.
- **Security‑sensitive routes:** `{token:guid}` ensures that only GUID‑format tokens reach authentication endpoints.

### References

- Route constraints – https://learn.microsoft.com/en-us/aspnet/core/fundamentals/routing#route-constraints
- IRouteConstraint interface – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.routing.irouteconstraint

---

## 6. Routing Best Practices

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Routing best practices are guidelines and conventions that help developers design route structures that are maintainable, performant, secure, and user‑friendly. |
| **Technical Definition** | Best practices encompass template design (precedence, constraints, optionality), route organisation (conventional vs attribute), URL generation, performance considerations (route table size, constraint complexity), and security (input validation, avoiding information leakage). |
| **Beginner‑Friendly Explanation** | Best practices are the “rules of thumb” that experienced developers follow to avoid common pitfalls. For example, using clear URLs, adding constraints to prevent errors, and keeping routes simple so the application remains easy to understand and maintain. |

### Purposes

- **To** keep the URL space predictable and easy to document.
- **To** avoid ambiguous or unreachable routes.
- **To** improve routing performance by reducing the number of candidate endpoints and the complexity of constraints.
- **To** enhance security by validating route values early.
- **To** facilitate URL generation and client‑side link consistency.
- **To** ensure maintainability as the application grows.

### Syntax Rules and Structure

Best practices do not have a single syntax, but the following **structural guidelines** are widely recommended:

#### General Guidelines (Checklist)

- **Prefer attribute routing for APIs** – It co‑locates the route with the action and makes the API surface explicit.
- **Use conventional routing for MVC views** – It reduces repetitive attributes for similar page structures.
- **Keep templates simple** – Avoid overly complex segments; use separate routes or a custom constraint if needed.
- **Apply constraints** – Use `:int`, `:guid`, `:regex`, etc., to reject invalid values early.
- **Avoid catch‑all routes unless necessary** – They have the lowest precedence and can swallow other routes if misordered.
- **Do not rely on route registration order in endpoint routing** – Precedence handles specificity; use `Order` only for genuine ambiguities.
- **Name your routes** – Named routes make URL generation more robust and refactoring‑friendly.
- **Use lowercase, hyphenated URLs** – Improves readability and SEO; enforce with a slug constraint.
- **Do not expose internal IDs as sequential integers in public URLs** – Consider slugs or GUIDs to prevent enumeration attacks.
- **Validate route values in the action** – Constraints are a first line of defence; always validate business rules.

#### Constraints and Limitations

- **Excessive use of `Order`** can mask design problems and make routing harder to reason about.
- **Regex constraints** should be anchored (`^...$`) and tested for performance.
- **Catch‑all routes** can hide 404s and should be used deliberately (e.g., SPA fallback).
- **Custom constraints** add startup cost and should be reserved for cases where built‑in constraints are insufficient.

### Multiple Annotated Complete Step‑by‑Step Code Examples

#### Example 1: API‑Focused Attribute Routing (Recommended for REST APIs)

```csharp
[Route("api/[controller]")]
[ApiController]
public class CustomersController : ControllerBase
{
    [HttpGet]                          // GET api/Customers
    public IActionResult GetAll() => Ok();

    [HttpGet("{id:guid}")]             // GET api/Customers/{guid}
    public IActionResult GetById(Guid id) => Ok(id);

    [HttpPost]                         // POST api/Customers
    public IActionResult Create([FromBody] CustomerDto dto) => Created("", dto);

    [HttpPut("{id:guid}")]             // PUT api/Customers/{guid}
    public IActionResult Update(Guid id, [FromBody] CustomerDto dto) => NoContent();

    [HttpDelete("{id:guid}")]          // DELETE api/Customers/{guid}
    public IActionResult Delete(Guid id) => NoContent();
}
```

**Step‑by‑step explanation:**

- The controller‑level route `api/[controller]` sets a consistent base path.
- Each action uses an HTTP verb attribute that also serves as a route template.
- The `{id:guid}` constraint ensures that only GUID‑format IDs reach the `GetById`, `Update`, and `Delete` actions.
- This style is **self‑documenting**: the API surface is visible directly from the controller code.

**Expected Output:**

| Method | URL | Action |
|--------|-----|--------|
| GET | `/api/Customers` | `GetAll` |
| GET | `/api/Customers/3fa85f64-5717-4562-b3fc-2c963f66afa6` | `GetById` |
| POST | `/api/Customers` | `Create` |
| PUT | `/api/Customers/3fa85f64-5717-4562-b3fc-2c963f66afa6` | `Update` |
| DELETE | `/api/Customers/3fa85f64-5717-4562-b3fc-2c963f66afa6` | `Delete` |

**Why this output?** Attribute routing makes the relationship between HTTP methods, URLs, and actions explicit, which is the recommended approach for REST APIs.

---

#### Example 2: Conventional Routing for MVC Views (Recommended for Page‑Based Apps)

```csharp
// Program.cs
app.MapControllerRoute(
    name: "default",
    pattern: "{controller=Home}/{action=Index}/{id?}");

app.MapControllerRoute(
    name: "blog",
    pattern: "blog/{action=Index}/{id?}",
    defaults: new { controller = "Blog" });
```

**Step‑by‑step explanation:**

- The first route is the conventional MVC default.
- The second route maps `blog/...` URLs to the `BlogController` without requiring the controller name in the URL.
- This reduces repetitive attributes on blog‑related actions.

**Expected Output:**

| URL | Controller | Action |
|-----|------------|--------|
| `/` | `Home` | `Index` |
| `/About` | `Home` | `About` (if action exists) |
| `/blog` | `Blog` | `Index` |
| `/blog/post/5` | `Blog` | `Post` (assuming action) |

**Why this output?** Conventional routes provide a fallback hierarchy that keeps view‑based URLs clean and consistent.

---

#### Example 3: Avoiding Ambiguity with Constraints and Precedence

```csharp
// Two routes that could overlap without constraints
app.MapControllerRoute(
    name: "product-by-id",
    pattern: "products/{id:int}",
    defaults: new { controller = "Products", action = "Details" });

app.MapControllerRoute(
    name: "product-by-slug",
    pattern: "products/{slug}",
    defaults: new { controller = "Products", action = "BySlug" });
```

**Step‑by‑step explanation:**

- `/products/5` matches the first route because `5` satisfies `int`. It **also** matches the second route (slug = `"5"`), but the `int` constraint makes the first route **more specific**.
- `/products/hello` does not satisfy `int`, so only the second route matches.
- Without the `int` constraint, both routes would match `/products/5` with the same precedence, causing an ambiguous match.

**Expected Output:**

| URL | Action | Reason |
|-----|--------|--------|
| `/products/5` | `Details(5)` | `int` constraint gives higher precedence. |
| `/products/hello` | `BySlug("hello")` | `int` constraint fails; slug route remains. |

**Why this output?** Constraints act as discriminators that allow overlapping templates to coexist without ambiguity. This is a core best practice for designing flexible yet deterministic routes.

### Real‑World Cases

- **Large REST API:** Attribute routing with verb‑specific templates and constraints keeps hundreds of endpoints organised and discoverable.
- **Multi‑language site:** Conventional route `{culture=en-US}/{controller=Home}/{action=Index}` provides a consistent structure for localised pages.
- **Legacy migration:** When migrating from ASP.NET MVC 5, keep conventional routes for existing views and introduce attribute routes for new APIs gradually.

### References

- Routing best practices – https://learn.microsoft.com/en-us/aspnet/core/fundamentals/routing#routing-best-practices
- ASP.NET Core web API best practices – https://learn.microsoft.com/en-us/aspnet/core/web-api/
- Route template precedence and endpoint selection order – https://learn.microsoft.com/en-us/aspnet/core/fundamentals/routing#route-template-precedence-and-endpoint-selection-order

---

## Summary Table: Core Concepts at a Glance

| Concept | Primary Purpose | Key Syntax | When to Use |
|---------|----------------|------------|-------------|
| **Routing** | Match requests to endpoints | `app.UseRouting()`, `MapControllerRoute`, `[Route]` | Every ASP.NET Core app |
| **Route Matching** | Select the best endpoint | Implicit; driven by templates, constraints, and precedence | Understanding why a route was chosen |
| **Route Templates** | Describe URL patterns | `{controller}/{action}/{id?}`, `products/{id:int}` | Defining URL structure |
| **Route Parameters** | Capture URL values | `{name}`, `{name?}`, `{name=default}` | Passing data from URL to actions |
| **Route Constraints** | Validate route values | `{id:int}`, `{slug:regex(...)}`, `{id:evenint}` | Rejecting invalid URLs early |
| **Routing Best Practices** | Maintainability & performance | Attribute routing for APIs, conventional for views, constraints, precedence awareness | Designing production‑ready routes |

---

## References (Full List)

- Routing in ASP.NET Core – https://learn.microsoft.com/en-us/aspnet/core/fundamentals/routing
- Routing to controller actions in ASP.NET Core – https://learn.microsoft.com/en-us/aspnet/core/mvc/controllers/routing
- ASP.NET Core Middleware – https://learn.microsoft.com/en-us/aspnet/core/fundamentals/middleware
- Route constraints – https://learn.microsoft.com/en-us/aspnet/core/fundamentals/routing#route-constraints
- IRouteConstraint Interface – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.routing.irouteconstraint
- Model Binding in ASP.NET Core – https://learn.microsoft.com/en-us/aspnet/core/mvc/models/model-binding
- ASP.NET Core Web API – https://learn.microsoft.com/en-us/aspnet/core/web-api/
- MvcOptions.EnableEndpointRouting Property – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.mvcoptions.enableendpointrouting
- ASP.NET Core Route Template Reference (GitHub source) – https://github.com/dotnet/aspnetcore/blob/main/src/Http/Routing/src/Template/RoutePrecedence.cs