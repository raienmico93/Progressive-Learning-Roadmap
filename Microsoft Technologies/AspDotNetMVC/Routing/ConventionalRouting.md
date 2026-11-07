# Conventional Routing in ASP.NET Core MVC: A Comprehensive Cheat Sheet

> **Version Scope & Compatibility Note**
> This cheat sheet focuses on **ASP.NET Core MVC** (3.0 and later, including .NET 6/7/8/10) and the **endpoint routing** system that became the default in ASP.NET Core 3.0. Conventional routing remains the recommended approach for **controllers and views** (UI‑centric applications). REST APIs should generally prefer **attribute routing** . Legacy routing (`System.Web.Routing`, pre‑3.0) is mentioned only where its behaviour differs significantly. `MapDefaultControllerRoute()` is a convenience wrapper for the default conventional route and is covered under **MapControllerRoute()**.

---

## Topic Overview

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Conventional routing is a routing strategy in ASP.NET Core MVC where URL patterns are defined **centrally** (typically in `Program.cs`) using **route templates** that establish a convention: the first segment maps to a controller name, the second to an action name, and subsequent segments to optional parameters. |
| **Technical Definition** | Conventional routing uses `MapControllerRoute` (or `MapAreaControllerRoute`) to register a route template (`{controller=Home}/{action=Index}/{id?}`) with the endpoint routing middleware. At request time, the routing middleware tokenises the URL path, extracts route values for `controller`, `action`, and optional parameters, applies constraints, and selects the endpoint whose controller/action combination exists in the application. Unlike attribute routing, conventional routing **does not place route metadata on controllers or actions**; the mapping is externalised into a central configuration. |
| **Beginner‑Friendly Explanation** | Conventional routing is like a **universal address format** for your website. Instead of writing a separate address rule for every page, you define one rule: “The first part of the URL is the **controller** (the department), the second part is the **action** (the task), and the third part is an optional **ID**.” So `/Products/Details/5` means: go to the **Products** controller, run the **Details** action, and pass **5** as the ID. This one rule covers hundreds of pages, as long as you follow the convention. |

### Key Characteristics

- **Centralised configuration** – Routes are defined in `Program.cs` (or `Startup.Configure`), not scattered across controllers.
- **Convention over configuration** – The URL structure follows a predictable pattern: `/{controller}/{action}/{id?}`.
- **Default values** – `controller` defaults to `Home`, `action` defaults to `Index`, so `/` works without specifying them .
- **Optional parameters** – `id` is optional (`{id?}`), so `/Products/Details` and `/Products/Details/5` can both match the same action .
- **Order‑sensitive registration (historical)** – In endpoint routing (3.0+), `MapControllerRoute` calls are automatically assigned an order value based on the order they are invoked; earlier routes have higher priority for matching . However, endpoint routing does not guarantee execution order of extensibility; precedence is computed from template specificity .
- **Order‑dependent for dedicated routes** – Dedicated conventional routes (e.g., `blog/{*article}`) must be registered **before** the default route to take precedence .
- **Works with controllers and views** – It is the idiomatic choice for MVC applications with Razor views .

### Prerequisites

- **.NET SDK** (6.0, 7.0, 8.0, or 10.0 recommended; examples use .NET 8/10 minimal hosting).
- **ASP.NET Core MVC** services registered (`builder.Services.AddControllersWithViews()`).
- Basic understanding of **controllers**, **actions**, and **HTTP verbs**.
- Familiarity with the **middleware pipeline** (`UseRouting`, `UseEndpoints` – though minimal hosting wraps these automatically) .

### Related Programming Areas

- **Attribute Routing** – The alternative routing strategy, preferred for REST APIs.
- **URL Generation** – Conventional route names are used by `IUrlHelper`, Tag Helpers, and `LinkGenerator`.
- **Areas** – Conventional routing is used to map area‑based controllers (`MapAreaControllerRoute`).
- **Model Binding** – Route values extracted by conventional routing are bound to action parameters.
- **Razor Pages** – Razor Pages use a page‑centric routing convention, not controller/action conventional routing.

### Core Concepts / Features

1. **Conventional Routing** – The overarching strategy and its configuration.
2. **`MapControllerRoute()`** – The primary API for registering conventional routes.
3. **Route Templates** – The syntax used to describe URL patterns.
4. **Default Route** – The built‑in `{controller=Home}/{action=Index}/{id?}` template.
5. **Route Parameters** – Named placeholders that capture URL values.
6. **Conventional Routing Best Practices** – Guidelines for maintainable and predictable route design.

Each concept is elaborated with the uniform structure: Definitions, Purposes, Syntax Rules & Structure, Annotated Code Examples with Expected Outputs, Real‑World Cases, and References.

---

## 1. Conventional Routing

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Conventional routing is a URL‑mapping strategy where a single, centrally defined route template governs how URLs are matched to controller actions based on naming conventions. |
| **Technical Definition** | In ASP.NET Core endpoint routing, conventional routing is implemented by registering one or more `RouteEndpoint` instances via `MapControllerRoute`. The route template defines **route parameters** for `controller`, `action`, and optional data. During matching, the routing middleware tokenises the URL, populates `RouteData.Values`, and validates that a controller with the extracted name and an action with the extracted name exist. The selected endpoint is then executed. |
| **Beginner‑Friendly Explanation** | Conventional routing is the “**one rule to route them all**” approach. You set up one pattern like `{controller}/{action}/{id?}` and then create controllers and actions that follow that pattern. When a user visits `/Blog/Article/42`, the application automatically knows to look for `BlogController`, run the `Article` method, and pass `42` as the `id`. You don’t have to write a new routing rule for every single page. |

### Purposes (each begins with “To”)

- **To** provide a single, predictable URL structure across an entire MVC application, reducing configuration overhead.
- **To** enable developers to add new controllers and actions without writing new routing rules, as long as they follow the convention.
- **To** support default values (`Home`, `Index`) so that the application root (`/`) works without explicit configuration.
- **To** allow optional parameters (`{id?}`) so that the same action can handle both list and detail views.
- **To** facilitate URL generation through named routes, ensuring that links remain consistent even if the route template changes.

### Syntax Rules and Structure

#### General Syntax (Endpoint Routing, .NET 6+)

```csharp
app.MapControllerRoute(
    name: "default",
    pattern: "{controller=Home}/{action=Index}/{id?}");
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `app` | The `WebApplication` instance (implements `IEndpointRouteBuilder`). |
| `MapControllerRoute` | Extension method that registers a conventional route for MVC controllers. |
| `name` | A unique name for the route, used for URL generation. Must be unique application‑wide . |
| `pattern` | The route template string. |
| `{controller=Home}` | Route parameter `controller` with default value `"Home"`. |
| `{action=Index}` | Route parameter `action` with default value `"Index"`. |
| `{id?}` | Optional route parameter `id` (the `?` suffix makes it optional). |

#### Alternative Convenience Syntax

```csharp
app.MapDefaultControllerRoute();
```

This is functionally equivalent to calling `MapControllerRoute` with `name: "default"` and `pattern: "{controller=Home}/{action=Index}/{id?}"` .

#### Syntax Rules

- Conventional routes are typically registered after `UseRouting()` (or automatically in minimal hosting) and before `app.Run()`.
- Multiple conventional routes can be registered by calling `MapControllerRoute` multiple times .
- Route names must be **unique**; duplicate names cause an exception at startup .
- The order of `MapControllerRoute` calls **does** influence matching priority: earlier routes receive a lower order value (higher priority) .
- Conventional routing is **not** used with `[ApiController]` or `[Route]` attributes on controllers; mixing requires explicit configuration.

#### Constraints and Limitations

- **Order sensitivity** – Dedicated routes (e.g., `blog/{*article}`) must be placed **before** the default route, otherwise the default route may swallow the request .
- **Ambiguity** – If two conventional routes can match the same URL with equal precedence, an `AmbiguousMatchException` is thrown .
- **No fine‑grained control** – Conventional routing applies the same template to all controllers; it cannot easily make a parameter required for one action and optional for another. Attribute routing is preferred when per‑action control is needed .
- **REST APIs** – Microsoft explicitly recommends **attribute routing** for REST APIs; conventional routing is intended for controllers and views .

### Multiple Annotated Complete Step‑by‑Step Code Examples

#### Example 1: Minimal MVC Application with the Default Conventional Route

**Setup:** Create an ASP.NET Core MVC project (`dotnet new mvc`). The generated `Program.cs` contains the conventional route.

```csharp
// Program.cs (ASP.NET Core 8/10 minimal hosting)
var builder = WebApplication.CreateBuilder(args);
builder.Services.AddControllersWithViews(); // Register MVC services

var app = builder.Build();

app.UseRouting();          // Enable routing middleware
app.UseAuthorization();    // Authorization (optional but typical)

app.MapControllerRoute(    // Register the default conventional route
    name: "default",
    pattern: "{controller=Home}/{action=Index}/{id?}");

app.Run();
```

**Step‑by‑step explanation:**

1. `AddControllersWithViews()` registers the services required for controllers and Razor views.
2. `UseRouting()` adds the routing middleware to the pipeline. It parses the incoming URL and selects an endpoint.
3. `MapControllerRoute` registers a conventional route with the name `"default"` and the template `{controller=Home}/{action=Index}/{id?}`.
4. The template establishes the convention: first segment = controller, second = action, third (optional) = `id`.
5. When a request arrives, the routing middleware tokenises the path and looks for a matching controller/action combination.

**Expected Output:**

| Request URL | Controller | Action | Route Values | Response |
|-------------|------------|--------|--------------|----------|
| `/` | `HomeController` | `Index` | `controller = "Home"`, `action = "Index"`, `id = null` | Home page |
| `/Products` | `ProductsController` | `Index` | `controller = "Products"`, `action = "Index"`, `id = null` | Product list |
| `/Products/Details/5` | `ProductsController` | `Details` | `controller = "Products"`, `action = "Details"`, `id = "5"` | Product 5 detail |

**Why this output?** The default values (`Home`, `Index`) fill in missing segments, and the optional marker (`?`) allows `id` to be omitted. The routing middleware validates that `HomeController.Index`, `ProductsController.Index`, and `ProductsController.Details` exist .

---

#### Example 2: Adding a Dedicated Conventional Route Before the Default Route

```csharp
// Program.cs
app.MapControllerRoute(
    name: "blog",
    pattern: "blog/{*article}",
    defaults: new { controller = "Blog", action = "Article" });

app.MapControllerRoute(
    name: "default",
    pattern: "{controller=Home}/{action=Index}/{id?}");
```

**Step‑by‑step explanation:**

1. The `blog` route is a **dedicated conventional route**. Its template `blog/{*article}` does not contain `{controller}` or `{action}`; instead, it provides fixed defaults: `controller = "Blog"`, `action = "Article"`.
2. Because the `blog` route is registered **first**, it receives a lower order value and higher matching priority .
3. The `{*article}` is a **catch‑all parameter** that captures the remainder of the URL (including slashes) into the `article` route value.
4. The `default` route is registered second and handles all other URLs.

**Expected Output:**

| Request URL | Route Matched | Controller/Action | Route Values |
|-------------|---------------|-------------------|--------------|
| `/blog/my-first-post` | `blog` | `BlogController.Article` | `article = "my-first-post"` |
| `/blog` | `blog` | `BlogController.Article` | `article = null` (catch‑all can match empty) |
| `/Products/Details/5` | `default` | `ProductsController.Details` | `controller = "Products"`, `action = "Details"`, `id = "5"` |

**Why this output?** The `blog` route is more specific for URLs starting with `blog` and is registered first, so it wins over the default route. The catch‑all parameter `{*article}` captures the slug, enabling SEO‑friendly blog URLs .

---

#### Example 3: Multiple Controllers and Actions Under the Default Route

```csharp
// Controllers/HomeController.cs
public class HomeController : Controller
{
    public IActionResult Index() => View();
    public IActionResult About() => View();
}

// Controllers/ProductsController.cs
public class ProductsController : Controller
{
    public IActionResult Index() => View();
    public IActionResult Details(int id) => View(id);
    public IActionResult Edit(int id) => View(id);
}

// Program.cs
app.MapControllerRoute(
    name: "default",
    pattern: "{controller=Home}/{action=Index}/{id?}");
```

**Step‑by‑step explanation:**

- The single default route maps **all** controllers and actions that follow the naming convention.
- `HomeController.Index` is the default for `/`.
- `HomeController.About` is reachable at `/Home/About`.
- `ProductsController.Details` is reachable at `/Products/Details/{id}`.
- `ProductsController.Edit` is reachable at `/Products/Edit/{id}`.

**Expected Output:**

| URL | Action Executed |
|-----|-----------------|
| `/` | `HomeController.Index` |
| `/Home/About` | `HomeController.About` |
| `/Products` | `ProductsController.Index` |
| `/Products/Details/7` | `ProductsController.Details(7)` |
| `/Products/Edit/7` | `ProductsController.Edit(7)` |

**Why this output?** Conventional routing tokenises each URL and checks whether the extracted controller name and action name correspond to existing types and methods. The optional `id` is bound to the action parameter if present .

### Real‑World Cases

- **E‑commerce storefront:** A conventional route `{controller=Home}/{action=Index}/{id?}` handles `/Products`, `/Products/Details/42`, `/Cart`, `/Checkout`, etc., without any attribute decoration.
- **Content‑managed blog:** A dedicated route `blog/{*slug}` placed before the default route allows SEO‑friendly URLs like `/blog/aspnet-core-routing`, while the default route continues to serve admin pages.
- **Internal line‑of‑business app:** CRUD screens follow the pattern `/Customers/Edit/5`, `/Orders/Details/100`, etc., keeping URLs predictable for users and testers.

### References

- Routing to controller actions in ASP.NET Core – https://learn.microsoft.com/en-us/aspnet/core/mvc/controllers/routing 
- Conventional routing – https://learn.microsoft.com/en-us/aspnet/core/mvc/controllers/routing#conventional-routing 

---

## 2. `MapControllerRoute()`

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | `MapControllerRoute()` is the extension method used to register a **conventional route** for MVC controllers, specifying a route name, a route template pattern, and optional defaults and constraints. |
| **Technical Definition** | `MapControllerRoute` is an extension method on `IEndpointRouteBuilder` (`ControllerEndpointRouteBuilderExtensions.MapControllerRoute`). It creates a `RouteEndpoint` whose `RequestDelegate` is the MVC controller dispatcher. The method accepts `name` (route name), `pattern` (template string), `defaults` (object), and `constraints` (object). It adds the endpoint to the application’s `EndpointDataSource` and assigns an order value based on invocation order . |
| **Beginner‑Friendly Explanation** | `MapControllerRoute()` is the **command** you use to tell ASP.NET Core: “Here is the rule for translating URLs into controller actions.” You give the rule a name (so you can refer to it later when generating links) and a pattern (the template). Think of it as **programming the receptionist**: “When a URL comes in, split it into controller, action, and ID according to this pattern.” |

### Purposes (each begins with “To”)

- **To** register a single conventional route that maps many controller/action combinations without individual attributes.
- **To** provide a **name** for the route so that URL generation (`IUrlHelper.RouteUrl`) can reference it reliably.
- **To** specify **default values** for controller and action, ensuring that the application root works.
- **To** apply **constraints** (inline or via the `constraints` parameter) to route parameters.
- **To** control the **order** of route matching by the sequence of `MapControllerRoute` calls.

### Syntax Rules and Structure

#### General Syntax

```csharp
app.MapControllerRoute(
    name: string,
    pattern: string,
    defaults: object? = null,
    constraints: object? = null);
```

**Component breakdown:**

| Component | Type | Required | Description |
|-----------|------|----------|-------------|
| `name` | `string` | Yes | Unique name for the route (used in URL generation). |
| `pattern` | `string` | Yes | Route template string (e.g., `"{controller=Home}/{action=Index}/{id?}"`). |
| `defaults` | `object?` | No | An object whose properties provide default route values (e.g., `new { controller = "Blog", action = "Article" }`). |
| `constraints` | `object?` | No | An object whose properties specify constraints (e.g., `new { id = @"\d+" }`). Inline constraints in the pattern are preferred. |

#### Convenience Variant

```csharp
app.MapDefaultControllerRoute();
```

Equivalent to `MapControllerRoute("default", "{controller=Home}/{action=Index}/{id?}")` .

#### Syntax Rules

- `name` must be **unique** across the application; duplicate names throw `InvalidOperationException` at startup .
- `pattern` must be a valid route template; it can include literal segments, parameters, defaults, optional markers, and constraints.
- `defaults` and `constraints` are typically anonymous objects; property names must match route parameter names.
- When multiple routes are registered, the **first registered** route has the **highest matching priority** (lowest order value) .
- `MapControllerRoute` maps **both** conventionally routed and attribute‑routed controllers (the latter via `MapControllers`, but conventional routes still apply to controllers without `[Route]` attributes) .

#### Constraints and Limitations

- **Order matters** – A generic default route registered before a dedicated route can shadow the dedicated route. Always place more specific routes first .
- **Cannot override attribute routing** – If a controller or action has `[Route]`, it is **attribute‑routed** and is **not** matched by conventional routes for that action .
- **Limited per‑action control** – `MapControllerRoute` applies the same pattern to all matching controllers; it cannot make `id` required for one action and optional for another. Use attribute routing or multiple dedicated routes for such cases.
- **Deprecated in favour of attribute routing for APIs** – Microsoft recommends attribute routing for REST APIs .

### Multiple Annotated Complete Step‑by‑Step Code Examples

#### Example 1: Basic `MapControllerRoute` with Defaults and Constraints

```csharp
// Program.cs
app.MapControllerRoute(
    name: "default",
    pattern: "{controller=Home}/{action=Index}/{id?}",
    defaults: new { controller = "Home", action = "Index" },
    constraints: new { id = @"\d+" }); // id must be digits
```

**Step‑by‑step explanation:**

1. `name: "default"` names the route for URL generation.
2. `pattern` defines the URL structure.
3. `defaults` redundantly specifies `Home` and `Index` (the pattern already includes them, but this demonstrates the parameter).
4. `constraints` applies a regex constraint to `id`, requiring one or more digits.
5. The route matches `/Products/Details/5` but **not** `/Products/Details/abc` (constraint fails).

**Expected Output:**

| URL | Match? | Reason |
|-----|--------|--------|
| `/` | Yes | Defaults fill in `Home`/`Index`. |
| `/Products` | Yes | `Products` controller, default `Index`. |
| `/Products/Details/5` | Yes | `id = "5"` satisfies `\d+`. |
| `/Products/Details/abc` | **No** | `abc` fails the `\d+` constraint. |

**Why this output?** The regex constraint `\d+` ensures that only numeric IDs reach the action. Non‑numeric segments cause the route to be rejected, potentially resulting in a 404 .

---

#### Example 2: Dedicated Route with `defaults` and Catch‑All

```csharp
app.MapControllerRoute(
    name: "blog",
    pattern: "blog/{*article}",
    defaults: new { controller = "Blog", action = "Article" });

app.MapControllerRoute(
    name: "default",
    pattern: "{controller=Home}/{action=Index}/{id?}");
```

**Step‑by‑step explanation:**

1. The `blog` route uses `defaults` to hard‑code `controller = "Blog"` and `action = "Article"`.
2. `{*article}` is a catch‑all parameter that captures everything after `blog/`.
3. The `blog` route is registered **before** the default route, giving it higher priority for URLs starting with `blog`.
4. The default route handles all other URLs.

**Expected Output:**

| URL | Matched Route | Action | Route Values |
|-----|---------------|--------|--------------|
| `/blog/intro-to-routing` | `blog` | `BlogController.Article` | `article = "intro-to-routing"` |
| `/blog/2025/03/post` | `blog` | `BlogController.Article` | `article = "2025/03/post"` |
| `/Home/About` | `default` | `HomeController.About` | `controller = "Home"`, `action = "About"` |

**Why this output?** The `blog` route’s fixed defaults mean it **always** maps to `BlogController.Article`, regardless of the URL’s first segment (as long as it starts with `blog`). The catch‑all parameter captures the slug, including slashes .

---

#### Example 3: `MapDefaultControllerRoute()` as a Shortcut

```csharp
// These two lines are functionally equivalent:
app.MapDefaultControllerRoute();

// Equivalent to:
app.MapControllerRoute(
    name: "default",
    pattern: "{controller=Home}/{action=Index}/{id?}");
```

**Step‑by‑step explanation:**

- `MapDefaultControllerRoute()` is a convenience method that registers the standard default route without requiring explicit `name` and `pattern` arguments .
- It is useful for typical MVC applications that only need the default conventional route.
- For applications requiring multiple routes or custom defaults, use `MapControllerRoute` explicitly.

**Expected Output:**

| URL | Action |
|-----|--------|
| `/` | `HomeController.Index` |
| `/Products/Details/3` | `ProductsController.Details(3)` |

**Why this output?** The shortcut produces the same route endpoint as the explicit call, so the matching behaviour is identical.

### Real‑World Cases

- **Standard MVC application:** `app.MapDefaultControllerRoute()` covers `/Home/Index`, `/Account/Login`, `/Products/List`, etc., with zero additional routing code.
- **Blog with SEO slugs:** `MapControllerRoute("blog", "blog/{*slug}", defaults: new { controller = "Blog", action = "Post" })` placed before the default route.
- **Multi‑tenant admin panel:** `MapControllerRoute("admin", "admin/{controller=Dashboard}/{action=Index}/{id?}")` provides a separate URL prefix for admin controllers.

### References

- `MapControllerRoute` method – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.builder.controllerendpointroutebuilderextensions.mapcontrollerroute 
- `MapDefaultControllerRoute` – https://learn.microsoft.com/en-us/aspnet/core/mvc/controllers/routing#set-up-conventional-route 

---

## 3. Route Templates

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | A route template is a string pattern that describes the structure of URLs an endpoint can handle, using literal text and parameter placeholders. |
| **Technical Definition** | A route template is parsed into a `RoutePattern` object. It consists of **segments** separated by `/`. Each segment may be a **literal** (exact match), a **parameter** (`{name}`), an **optional parameter** (`{name?}`), a **parameter with default** (`{name=value}`), a **parameter with constraint** (`{name:constraint}`), or a **catch‑all parameter** (`{*name}`). The template drives both URL matching and URL generation. |
| **Beginner‑Friendly Explanation** | A route template is the **blueprint** for your URLs. It tells the router which parts of the URL are fixed text and which parts are changeable “blanks” to fill in. For example, `products/{category}/{id?}` means: “The URL must start with `products`, then have a category (any word), and may optionally end with an ID.” |

### Purposes (each begins with “To”)

- **To** describe URL patterns in a human‑readable, declarative syntax.
- **To** extract variable data from the URL into named route values.
- **To** support optional segments and default values so that one template can match multiple URL shapes.
- **To** apply constraints that validate route parameter values before the action executes.
- **To** enable reverse URL generation by providing the pattern from which links are constructed.

### Syntax Rules and Structure

#### General Syntax

```
segment1/segment2/{parameter1}/{parameter2?}/{parameter3=default}/{parameter4:constraint}/{*catchAll}
```

#### Component Breakdown

| Component | Syntax | Example | Meaning |
|-----------|--------|---------|---------|
| **Literal segment** | Plain text | `products` | Must match exactly (case‑insensitive). |
| **Parameter segment** | `{name}` | `{id}` | Captures the segment value into route value `name`. |
| **Optional parameter** | `{name?}` | `{id?}` | The segment may be omitted; if omitted, no route value is produced . |
| **Default value** | `{name=value}` | `{action=Index}` | If the segment is absent, the default is used; always produces a route value . |
| **Inline constraint** | `{name:constraint}` | `{id:int}` | The captured value must satisfy the constraint. |
| **Catch‑all parameter** | `{*name}` | `{*path}` | Captures the remainder of the URL, including slashes. |
| **Complex segment** | `{name=value:constraint}` | `{id=1:int}` | Combines default and constraint. |

#### Syntax Rules

- Parameters must be enclosed in `{}` and use valid C# identifier names.
- A template **cannot** contain two consecutive parameters without a literal delimiter (e.g., `{a}{b}` is invalid).
- Optional (`?`) and default (`=`) markers must come **after** the parameter name.
- Constraints (`:`) can be chained; the parameter name comes first.
- Catch‑all parameters (`{*name}`) must be the **last** segment of the template.
- **`[controller]`** and **`[action]`** tokens are **not** used in conventional routing; they are attribute‑routing tokens.

#### Constraints and Limitations

- **Empty segments** – An optional parameter does **not** match an empty segment; `/products/` does not match `products/{id?}`.
- **Default values** – A parameter with a default always produces a route value, even if the segment is absent .
- **Catch‑all greediness** – Catch‑all parameters are the least specific and can make a route “greedy,” swallowing URLs intended for other routes .
- **Regex complexity** – Inline regex constraints can be vulnerable to ReDoS if applied to untrusted input; use simple patterns or custom constraints with timeouts.

### Multiple Annotated Complete Step‑by‑Step Code Examples

#### Example 1: Basic Template with Required and Optional Parameters

```csharp
app.MapControllerRoute(
    name: "shop",
    pattern: "shop/{category}/{id?}");
```

**Step‑by‑step explanation:**

- `shop` is a literal segment.
- `{category}` is a required parameter.
- `{id?}` is an optional parameter.

**Expected Output:**

| URL | Route Values |
|-----|--------------|
| `/shop/electronics` | `category = "electronics"`, `id = null` |
| `/shop/electronics/42` | `category = "electronics"`, `id = "42"` |
| `/shop` | **No match** (required `category` missing) |

**Why this output?** The required parameter `{category}` must be present. The optional `{id?}` is omitted in the first URL, so `id` remains `null` and is not added to route values .

---

#### Example 2: Default Value and Constraint

```csharp
app.MapControllerRoute(
    name: "archive",
    pattern: "archive/{year:int}/{month:int=1}/{day?}");
```

**Step‑by‑step explanation:**

- `archive` is literal.
- `{year:int}` is required and must be an integer.
- `{month:int=1}` is optional with a default value of `1`; it must be an integer.
- `{day?}` is optional; no constraint.

**Expected Output:**

| URL | Route Values |
|-----|--------------|
| `/archive/2025` | `year = "2025"`, `month = "1"`, `day = null` |
| `/archive/2025/6` | `year = "2025"`, `month = "6"`, `day = null` |
| `/archive/2025/6/15` | `year = "2025"`, `month = "6"`, `day = "15"` |
| `/archive/abc` | **No match** (year fails `int`) |

**Why this output?** The `month` parameter has a default (`1`), so it always produces a route value even when absent. The `year` constraint rejects non‑integer values .

---

#### Example 3: Catch‑All Parameter

```csharp
app.MapControllerRoute(
    name: "files",
    pattern: "files/{*path}");
```

**Step‑by‑step explanation:**

- `files` is literal.
- `{*path}` is a catch‑all parameter that captures everything after `files/`, including slashes.

**Expected Output:**

| URL | `path` Value |
|-----|--------------|
| `/files/docs/report.pdf` | `docs/report.pdf` |
| `/files/images/logo.png` | `images/logo.png` |
| `/files` | `null` (catch‑all can match empty) |

**Why this output?** Catch‑all parameters consume the remaining path segments, making them ideal for file‑serving or SPA fallback routes. They are the least specific template element .

### Real‑World Cases

- **E‑commerce category browsing:** `shop/{category}/{id?}` handles both category listing and product detail.
- **Date‑based archives:** `archive/{year:int}/{month:int=1}/{day?}` supports yearly, monthly, and daily views.
- **File/CDN proxy:** `files/{*path}` serves arbitrary nested files.

### References

- Route template reference – https://learn.microsoft.com/en-us/aspnet/core/fundamentals/routing#route-template-reference 
- Route parameters – https://learn.microsoft.com/en-us/aspnet/core/fundamentals/routing#route-parameters 

---

## 4. Default Route

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | The default route is the conventional route template `{controller=Home}/{action=Index}/{id?}`, which is automatically used by the ASP.NET Core MVC template and is registered with the name `"default"`. |
| **Technical Definition** | The default route is a `RouteEndpoint` created by `MapControllerRoute` (or `MapDefaultControllerRoute`) with the pattern `{controller=Home}/{action=Index}/{id?}`. It provides default values for `controller` and `action`, and marks `id` as optional. It is the only route many MVC applications need . |
| **Beginner‑Friendly Explanation** | The default route is the **“home page rule.”** It says: “If the user doesn’t specify a controller, use **Home**; if they don’t specify an action, use **Index**; and if they include an ID, pass it along.” So `/` shows the home page, `/Products` shows the product list, and `/Products/Details/5` shows product 5. |

### Purposes (each begins with “To”)

- **To** provide a ready‑made routing convention that works for most CRUD‑style MVC applications without additional configuration.
- **To** ensure that the application root (`/`) resolves to a meaningful page (`HomeController.Index`) .
- **To** allow controllers to be accessed without explicitly naming the `Index` action (e.g., `/Products` instead of `/Products/Index`) .
- **To** support optional `id` parameters for detail/edit/delete actions.
- **To** serve as a fallback route when no more specific route matches.

### Syntax Rules and Structure

#### General Syntax

```csharp
app.MapControllerRoute(
    name: "default",
    pattern: "{controller=Home}/{action=Index}/{id?}");
```

Or the shortcut:

```csharp
app.MapDefaultControllerRoute();
```

**Component breakdown:**

| Component | Value | Effect |
|-----------|-------|--------|
| `{controller=Home}` | Default controller = `Home` | `/` and `/Products` use `Home` when controller omitted. |
| `{action=Index}` | Default action = `Index` | `/Products` runs `Index` when action omitted. |
| `{id?}` | Optional `id` | `/Products/Details` and `/Products/Details/5` both match. |

#### Syntax Rules

- The default route should typically be the **last** conventional route registered, unless more specific routes have been added before it .
- The name `"default"` is conventional but not mandatory; any unique name can be used.
- Default values are **always** applied when the corresponding segment is absent; they are not overridden by empty segments .

#### Constraints and Limitations

- **Overly broad** – The default route matches almost any three‑segment URL, which can lead to unexpected matches if not carefully ordered with dedicated routes.
- **`id` is optional for all actions** – In the default route, `id` is optional for every action. If an action requires `id`, it must validate it or use attribute routing to enforce the requirement .
- **Not suitable for APIs** – REST APIs should use attribute routing for precise HTTP verb and resource mapping .

### Multiple Annotated Complete Step‑by‑Step Code Examples

#### Example 1: The Default Route in Action

```csharp
app.MapDefaultControllerRoute();

// HomeController.cs
public class HomeController : Controller
{
    public IActionResult Index() => Content("Home Index");
    public IActionResult About() => Content("About Us");
}

// ProductsController.cs
public class ProductsController : Controller
{
    public IActionResult Index() => Content("Product List");
    public IActionResult Details(int id) => Content($"Product {id}");
}
```

**Step‑by‑step explanation:**

1. `MapDefaultControllerRoute()` registers the default pattern.
2. `HomeController.Index` matches `/`, `/Home`, and `/Home/Index`.
3. `HomeController.About` matches `/Home/About`.
4. `ProductsController.Index` matches `/Products` and `/Products/Index`.
5. `ProductsController.Details` matches `/Products/Details` (with `id = 0` by default binding) and `/Products/Details/5` (with `id = 5`).

**Expected Output:**

| URL | Response |
|-----|----------|
| `/` | `Home Index` |
| `/Home` | `Home Index` |
| `/Home/About` | `About Us` |
| `/Products` | `Product List` |
| `/Products/Details/5` | `Product 5` |
| `/Products/Details` | `Product 0` (model binding sets `id` to `0` when omitted)  |

**Why this output?** Default values fill in `Home` and `Index` when segments are missing. The optional `id` is omitted in `/Products/Details`, and model binding assigns `0` to the `int id` parameter .

---

#### Example 2: Default Route with a Dedicated Route for Specific URLs

```csharp
app.MapControllerRoute(
    name: "blog",
    pattern: "blog/{*article}",
    defaults: new { controller = "Blog", action = "Article" });

app.MapDefaultControllerRoute(); // equivalent to the default route
```

**Step‑by‑step explanation:**

1. The `blog` route handles URLs starting with `blog/`.
2. `MapDefaultControllerRoute()` handles all remaining URLs.
3. Because `blog` is registered first, it takes precedence for `blog/...` URLs .

**Expected Output:**

| URL | Matched Route | Action |
|-----|---------------|--------|
| `/blog/routing-basics` | `blog` | `BlogController.Article` |
| `/Products/Details/3` | `default` | `ProductsController.Details(3)` |

**Why this output?** The dedicated `blog` route is more specific for its URL prefix and is registered before the default route, so it wins for blog URLs .

### Real‑World Cases

- **Standard corporate website:** `/` → Home, `/About` → About, `/Services` → Services list, `/Services/Details/2` → Service detail.
- **Internal CRUD app:** `/Customers`, `/Customers/Edit/5`, `/Orders/Details/100` all handled by the default route.
- **Blog‑plus‑website:** Dedicated `blog` route for posts, default route for static pages and admin sections.

### References

- Set up conventional route – https://learn.microsoft.com/en-us/aspnet/core/mvc/controllers/routing#set-up-conventional-route 
- The default route – https://learn.microsoft.com/en-us/aspnet/core/mvc/controllers/routing#conventional-routing 

---

## 5. Route Parameters

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Route parameters are named placeholders in a route template that capture values from corresponding URL segments and make them available to the action method. |
| **Technical Definition** | A route parameter is defined by `{name}` in a template. During matching, the routing middleware splits the URL path into segments and assigns each segment to the parameter in the same position. The extracted values are stored in `RouteData.Values` (or `HttpContext.Request.RouteValues` in endpoint routing). Model binding then converts these string values to the action method’s parameter types. |
| **Beginner‑Friendly Explanation** | Route parameters are the **fill‑in‑the‑blanks** of a URL. In `products/{id}`, the `{id}` is a blank. If the URL is `/products/5`, the router fills the blank with `5` and hands it to your action as the `id` argument. If the URL is `/products/hello`, the blank is filled with `hello`. |

### Purposes (each begins with “To”)

- **To** pass variable data from the URL to controller actions without using query strings.
- **To** create hierarchical, RESTful‑style URLs (e.g., `/customers/5/orders/10`).
- **To** enable model binding to automatically populate action parameters from route values.
- **To** support optional and default values so that a single action can handle multiple URL shapes.
- **To** allow URL generation from route values, closing the loop between matching and link creation.

### Syntax Rules and Structure

#### General Syntax

```
{parameterName}
{parameterName?}          // optional
{parameterName=default}   // default value
{parameterName:constraint} // inline constraint
{*parameterName}          // catch-all
```

#### Component Breakdown

| Syntax | Meaning | Route Value Produced? |
|--------|---------|----------------------|
| `{id}` | Required parameter | Yes, when segment present. |
| `{id?}` | Optional parameter | Only if segment present; otherwise `null` . |
| `{id=5}` | Parameter with default | Always; default used if segment absent . |
| `{id:int}` | Parameter with constraint | Yes, if value satisfies constraint. |
| `{*path}` | Catch‑all parameter | Captures remainder of URL. |

#### Syntax Rules

- Parameter names are **case‑insensitive** during matching but are exposed with the casing used in the template.
- A parameter can have **one default** and **multiple constraints** (chained with `:`).
- Optional parameters must appear **after** required parameters in the same template segment.
- Parameters cannot span multiple segments unless they are catch‑all parameters (`{*name}`).
- **Empty segments** are not matched by optional parameters; `/products/` does not match `products/{id?}`.

#### Constraints and Limitations

- **Model binding failures** – If a route value cannot be converted to the action parameter type, model binding may fail (resulting in a 400 or a default value, depending on configuration). Inline constraints (e.g., `:int`) prevent the route from matching in the first place.
- **Sequential IDs** – Exposing sequential integer IDs in URLs can enable enumeration attacks; consider slugs or GUIDs for public‑facing resources.
- **Parameter name collisions** – Avoid using the same parameter name in overlapping templates if the routes can both match; this can lead to unpredictable binding.

### Multiple Annotated Complete Step‑by‑Step Code Examples

#### Example 1: Required and Optional Parameters

```csharp
app.MapControllerRoute(
    name: "products",
    pattern: "products/{category}/{id?}");

// ProductsController.cs
public IActionResult List(string category, int? id)
{
    return Content($"Category: {category}, ID: {id?.ToString() ?? "none"}");
}
```

**Step‑by‑step explanation:**

- `{category}` is required.
- `{id?}` is optional.
- The action signature uses `int?` for `id` to accommodate the optional value.

**Expected Output:**

| URL | Response |
|-----|----------|
| `/products/electronics` | `Category: electronics, ID: none` |
| `/products/electronics/42` | `Category: electronics, ID: 42` |
| `/products` | **404** (missing required `category`) |

**Why this output?** The required parameter `category` must be present for the route to match. The optional `id` is `null` when absent, which is reflected in the response .

---

#### Example 2: Default Value Parameter

```csharp
app.MapControllerRoute(
    name: "blog",
    pattern: "blog/{year:int}/{month:int=1}");
```

**Step‑by‑step explanation:**

- `{year:int}` is required and must be an integer.
- `{month:int=1}` is optional with a default value of `1`; it must be an integer.

**Expected Output:**

| URL | Route Values |
|-----|--------------|
| `/blog/2025` | `year = "2025"`, `month = "1"` |
| `/blog/2025/6` | `year = "2025"`, `month = "6"` |
| `/blog/abc` | **No match** (year constraint fails) |

**Why this output?** The `month` parameter has a default (`1`), so it always produces a route value even when absent .

---

#### Example 3: Route Parameters Bound to Action Arguments

```csharp
app.MapControllerRoute(
    name: "orders",
    pattern: "orders/{customerId:int}/orders/{orderId:int}");

// OrdersController.cs
public IActionResult Details(int customerId, int orderId)
{
    return Content($"Customer {customerId}, Order {orderId}");
}
```

**Step‑by‑step explanation:**

- The template has two `int` parameters: `customerId` and `orderId`.
- The action method has parameters with the **same names**.
- Model binding automatically maps route values to action parameters by name.

**Expected Output:**

| URL | Response |
|-----|----------|
| `/orders/5/orders/10` | `Customer 5, Order 10` |
| `/orders/abc/orders/10` | **404** (`customerId` fails `int`) |

**Why this output?** Route parameter names must match action parameter names for automatic binding. Constraints ensure that only integer values reach the action .

### Real‑World Cases

- **RESTful resource hierarchy:** `/customers/{customerId}/orders/{orderId}` captures both identifiers for nested resources.
- **Blog archive:** `/archive/{year:int}/{month:int?}` supports optional month filtering.
- **E‑commerce filtering:** `/shop/{category}/{priceRange?}` captures category and optional price range.

### References

- Route parameters – https://learn.microsoft.com/en-us/aspnet/core/fundamentals/routing#route-parameters 
- Model binding – https://learn.microsoft.com/en-us/aspnet/core/mvc/models/model-binding 

---

## 6. Conventional Routing Best Practices

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Conventional routing best practices are guidelines for designing route configurations that are predictable, maintainable, performant, and secure, leveraging the strengths of conventional routing while mitigating its limitations. |
| **Technical Definition** | Best practices encompass route ordering (specific before generic), template design (avoiding overly greedy catch‑alls), constraint usage (early rejection of invalid values), naming conventions (unique, descriptive route names), and the decision boundary between conventional and attribute routing (controllers/views vs REST APIs). |
| **Beginner‑Friendly Explanation** | Best practices are the “**rules of thumb**” that keep your routing easy to understand and safe. For example: put special rules before general rules, use clear names, add constraints to stop bad URLs early, and don’t use conventional routing for APIs that need precise HTTP verb mapping. |

### Purposes (each begins with “To”)

- **To** ensure deterministic matching by placing more specific routes before generic ones .
- **To** avoid `AmbiguousMatchException` by designing templates with distinct precedence or by using constraints.
- **To** keep URLs readable and meaningful through basic, descriptive routing schemes .
- **To** improve performance by using constraints to reject invalid URLs early in the matching pipeline.
- **To** simplify maintenance by using the default route for most CRUD actions and dedicated routes only for special cases.
- **To** separate concerns by using conventional routing for UI controllers/views and attribute routing for REST APIs .

### Syntax Rules and Structure

Best practices are not a single syntax but a set of structural guidelines:

#### Guideline Checklist

| Guideline | Rationale | Example |
|-----------|-----------|---------|
| **Register dedicated routes before the default route** | The default route is broad and can swallow URLs intended for dedicated routes . | `blog/{*article}` before `{controller=Home}/{action=Index}/{id?}` |
| **Use catch‑all parameters sparingly** | Catch‑alls are the least specific and can make routes “greedy” . | Use only for file serving or SPA fallback. |
| **Apply constraints to route parameters** | Reject invalid values early; avoid model binding errors and unnecessary action execution . | `{id:int}`, `{slug:regex(^[a-z0-9-]+$)}` |
| **Give every route a unique name** | Names are required for reliable URL generation . | `name: "blog"`, `name: "default"` |
| **Prefer `MapDefaultControllerRoute()` for simple apps** | Reduces boilerplate and leverages the built‑in convention . | One line covers most CRUD URLs. |
| **Use attribute routing for REST APIs** | Conventional routing cannot express HTTP verb semantics or per‑action templates precisely . | `[HttpGet("api/products/{id:int}")]` |
| **Avoid overly complex templates** | Complex segments are harder to read, debug, and generate links for. | Prefer separate routes or custom constraints. |
| **Validate business rules in the action** | Constraints only handle route matching, not business validation. | Check that `id > 0` inside the action. |

#### Constraints and Limitations of Best Practices

- **Order is not always guaranteed** – Endpoint routing computes precedence from template specificity, but `MapControllerRoute` order still assigns order values that influence matching . Do not rely solely on precedence when dedicated routes overlap with the default route.
- **Over‑constraining** – Adding too many constraints can make routes brittle and hard to extend. Use constraints only where they add clear value.
- **Mixing conventional and attribute routing** – Controllers with `[Route]` attributes are **attribute‑routed** and are not matched by conventional routes for those actions . Be intentional about which strategy each controller uses.

### Multiple Annotated Complete Step‑by‑Step Code Examples

#### Example 1: Correct Ordering – Dedicated Route Before Default

```csharp
// ✅ CORRECT: Specific route first
app.MapControllerRoute(
    name: "blog",
    pattern: "blog/{*article}",
    defaults: new { controller = "Blog", action = "Article" });

app.MapControllerRoute(
    name: "default",
    pattern: "{controller=Home}/{action=Index}/{id?}");

// ❌ INCORRECT: Default route first would swallow blog URLs
// app.MapControllerRoute("default", "{controller=Home}/{action=Index}/{id?}");
// app.MapControllerRoute("blog", "blog/{*article}", defaults: new { ... });
```

**Step‑by‑step explanation:**

1. The `blog` route is registered first, receiving a lower order value and higher matching priority.
2. The default route is registered second.
3. A request to `/blog/routing` matches the `blog` route because it is evaluated first.
4. If the default route were registered first, `/blog/routing` could be interpreted as `controller = "blog"`, `action = "routing"`, which would fail if no such controller/action exists, and the blog route would never be reached .

**Expected Output:**

| URL | Matched Route |
|-----|---------------|
| `/blog/routing` | `blog` (→ `BlogController.Article`) |
| `/Products/Details/5` | `default` (→ `ProductsController.Details`) |

**Why this output?** Ordering ensures that the more specific `blog` route is considered before the generic default route.

---

#### Example 2: Using Constraints to Prevent Invalid Matches

```csharp
app.MapControllerRoute(
    name: "product",
    pattern: "products/{id:int}",
    defaults: new { controller = "Products", action = "Details" });

app.MapControllerRoute(
    name: "default",
    pattern: "{controller=Home}/{action=Index}/{id?}");
```

**Step‑by‑step explanation:**

- The `product` route uses `{id:int}` to match only numeric IDs.
- A request to `/products/5` matches the `product` route and invokes `ProductsController.Details(5)`.
- A request to `/products/abc` does **not** match the `product` route (constraint fails) and falls through to the default route, where it may match a different controller/action or return 404.
- This prevents invalid IDs from reaching the `Details` action and avoids potential model binding errors .

**Expected Output:**

| URL | Matched Route | Action |
|-----|---------------|--------|
| `/products/5` | `product` | `ProductsController.Details(5)` |
| `/products/abc` | `default` (if `ProductsController` has an `Abc` action) or 404 | Varies |

**Why this output?** The `int` constraint makes the `product` route more specific and ensures that only numeric IDs are routed to `Details`. Non‑numeric segments are handled by other routes or result in a 404.

---

#### Example 3: Balancing Conventional and Attribute Routing

```csharp
// Program.cs – Conventional routes for UI controllers
app.MapControllerRoute(
    name: "default",
    pattern: "{controller=Home}/{action=Index}/{id?}");

// Attribute-routed API controller (not affected by conventional route)
[ApiController]
[Route("api/[controller]")]
public class ApiProductsController : ControllerBase
{
    [HttpGet("{id:int}")]
    public IActionResult Get(int id) => Ok(id);
}
```

**Step‑by‑step explanation:**

1. The conventional default route handles UI controllers (`HomeController`, `ProductsController`, etc.) that do **not** have `[Route]` attributes.
2. `ApiProductsController` has `[Route("api/[controller]")]`, making it **attribute‑routed**. It is **not** matched by the conventional route .
3. This separation follows Microsoft’s guidance: conventional routing for controllers/views, attribute routing for REST APIs .

**Expected Output:**

| URL | Matched By | Action |
|-----|------------|--------|
| `/Products/Details/5` | Conventional `default` route | `ProductsController.Details(5)` |
| `/api/ApiProducts/5` | Attribute route | `ApiProductsController.Get(5)` |

**Why this output?** Attribute routing takes precedence for controllers decorated with `[Route]`; conventional routes only apply to controllers without route attributes.

### Real‑World Cases

- **Large MVC application with a blog:** Dedicated `blog/{*slug}` route before the default route, with constraints on the default route’s `id` (`{id:int?}`) to prevent non‑numeric IDs from reaching CRUD actions.
- **Admin area:** `MapAreaControllerRoute` for admin controllers, registered before the default route, ensuring `/Admin/...` URLs are handled separately .
- **API + MVC hybrid:** Conventional routing for Razor view controllers, `MapControllers()` for attribute‑routed API controllers, keeping both worlds cleanly separated.

### References

- Routing best practices – https://learn.microsoft.com/en-us/aspnet/core/fundamentals/routing#routing-best-practices
- Conventional routing order – https://learn.microsoft.com/en-us/aspnet/core/mvc/controllers/routing#conventional-routing 
- Attribute routing for REST APIs – https://learn.microsoft.com/en-us/aspnet/core/mvc/controllers/routing#attribute-routing-for-rest-apis 

---

## Summary Table: Conventional Routing at a Glance

| Concept | Primary Purpose | Key Syntax | When to Use |
|---------|----------------|------------|-------------|
| **Conventional Routing** | Centralised URL‑to‑controller mapping | `MapControllerRoute`, `MapDefaultControllerRoute` | MVC apps with controllers and views |
| **`MapControllerRoute()`** | Register a named conventional route | `app.MapControllerRoute(name, pattern, defaults?, constraints?)` | When you need multiple routes or custom defaults |
| **Route Templates** | Describe URL patterns | `{controller=Home}/{action=Index}/{id?}`, `blog/{*slug}` | Defining URL structure |
| **Default Route** | Ready‑made MVC routing convention | `{controller=Home}/{action=Index}/{id?}` or `MapDefaultControllerRoute()` | Most CRUD‑style MVC applications |
| **Route Parameters** | Capture URL values | `{id}`, `{id?}`, `{id=5}`, `{id:int}` | Passing data from URL to actions |
| **Best Practices** | Maintainability, performance, security | Specific before generic; constraints; separate APIs | Production‑ready route design |

---

## References (Full List)

- Routing to controller actions in ASP.NET Core – https://learn.microsoft.com/en-us/aspnet/core/mvc/controllers/routing 
- Conventional routing – https://learn.microsoft.com/en-us/aspnet/core/mvc/controllers/routing#conventional-routing 
- Set up conventional route – https://learn.microsoft.com/en-us/aspnet/core/mvc/controllers/routing#set-up-conventional-route 
- `MapControllerRoute` API – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.builder.controllerendpointroutebuilderextensions.mapcontrollerroute 
- `MapDefaultControllerRoute` API – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.builder.controllerendpointroutebuilderextensions.mapdefaultcontrollerroute
- Route template reference – https://learn.microsoft.com/en-us/aspnet/core/fundamentals/routing#route-template-reference 
- Route parameters – https://learn.microsoft.com/en-us/aspnet/core/fundamentals/routing#route-parameters 
- Routing best practices – https://learn.microsoft.com/en-us/aspnet/core/fundamentals/routing#routing-best-practices
- Areas in ASP.NET Core – https://learn.microsoft.com/en-us/aspnet/core/mvc/controllers/areas 
- Model binding – https://learn.microsoft.com/en-us/aspnet/core/mvc/models/model-binding 
- GitHub: AspNetCore.Docs – routing.md – https://github.com/dotnet/AspNetCore.Docs/blob/main/aspnetcore/mvc/controllers/routing.md 