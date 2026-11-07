# Route Naming in ASP.NET Core MVC: A Comprehensive Cheat Sheet

> **Version Scope & Compatibility Note**
> This cheat sheet focuses on **ASP.NET Core MVC** (3.0 and later, including .NET 6/7/8/10/11) and the **endpoint routing** system that became the default in ASP.NET Core 3.0. Route naming behaves consistently across **conventional routing** and **attribute routing**. Named routes are primarily used for **URL generation**; they have no impact on URL matching or request handling. Razor Pages uses a slightly different naming model (page names rather than route names), which is noted where relevant.

---

## Topic Overview

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Route naming is the practice of assigning a **unique, logical name** to a route so that URLs can be generated reliably without hard‑coding path strings. |
| **Technical Definition** | A route name is represented by `IEndpointNameMetadata` (or `EndpointNameMetadata`) attached to an endpoint. In conventional routing, the name is supplied as the `name` argument to `MapControllerRoute`; in attribute routing, it is supplied via the `Name` property of `[Route]`, `[HttpGet]`, and other verb attributes. The routing system resolves route names during **link generation** through `LinkGenerator` (using address schemes such as `GetPathByName`) or `IUrlHelper.RouteUrl`. Route names must be **globally unique** across the application; duplicates cause a startup exception. Route names are **case‑sensitive** and do not affect request matching. |
| **Beginner‑Friendly Explanation** | A route name is a **nickname** for a URL pattern. Instead of writing `"/products/5"` everywhere in your views, you give the product‑detail route a name like `"GetProduct"`. When you need a link, you say: “Give me the URL for `GetProduct` with `id = 5`.” If the URL pattern later changes from `/products/{id}` to `/catalog/{id}`, you update the route once, and every generated link updates automatically. |

### Key Characteristics

- **Logical identifier** – Route names are human‑readable labels, not URL fragments.
- **Generation‑only** – Route names do **not** participate in URL matching or request handling; they are used exclusively for link generation.
- **Globally unique** – Every route name must be unique across the entire application; duplicate names throw an exception at startup.
- **Case‑sensitive** – `"GetProduct"` and `"getproduct"` are different route names.
- **Stable reference** – Named routes decouple link generation from template changes, improving maintainability.
- **Works with conventional and attribute routing** – Both strategies support named routes.
- **Reserved names** – Keywords such as `action`, `area`, `controller`, `handler`, and `page` cannot be used as route names.
- **OpenAPI integration** – Route names are used as the **operation ID** when OpenAPI support is enabled.

### Prerequisites

- **.NET SDK** (6.0+ recommended; examples use .NET 8/10 minimal hosting).
- **ASP.NET Core MVC** services registered (`AddControllersWithViews()` or `AddControllers()`).
- Routing middleware configured (`UseRouting()` and `MapControllers()` / `MapControllerRoute`).
- Basic understanding of **route templates**, **route parameters**, and **URL generation**.

### Related Programming Areas

- **Route Templates** – Named routes are attached to templates; the template defines the URL shape.
- **Attribute Routing** – `Name` property on `[Route]` and HTTP verb attributes.
- **Conventional Routing** – `name` argument to `MapControllerRoute`.
- **URL Generation** – `LinkGenerator`, `IUrlHelper.RouteUrl`, Tag Helpers, and action results (`CreatedAtRoute`).
- **OpenAPI / Swagger** – Route names become operation IDs.
- **Razor Pages** – Uses page names rather than route names; the two concepts are related but distinct.

### Core Concepts / Features

1. **Route Naming (General)** – The concept, purpose, and rules of route names.
2. **Named Routes** – How to define named routes in conventional and attribute routing.
3. **Route Generation** – How named routes are used to generate URLs via `IUrlHelper`, `LinkGenerator`, and action results.
4. **Route Naming Best Practices** – Guidelines for choosing and maintaining route names.

Each concept is elaborated using the uniform structure: Definitions, Purposes, Syntax Rules & Structure, Annotated Code Examples with Expected Outputs, Real‑World Cases, and References.

---

## 1. Route Naming (General)

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Route naming is the assignment of a unique, logical name to a route endpoint so that the endpoint can be unambiguously referenced during URL generation. |
| **Technical Definition** | A route name is stored as `EndpointNameMetadata` on the endpoint. The routing system resolves names through the **endpoint‑name address scheme**: when `LinkGenerator.GetPathByName` (or `IUrlHelper.RouteUrl`) is called with a name, the scheme performs a dictionary lookup across all registered endpoints and selects the endpoint whose `EndpointNameMetadata` matches. If multiple endpoints share the same name, a startup exception is thrown. The name does not affect matching because matching operates on the route template, not the name. |
| **Beginner‑Friendly Explanation** | Route naming is like giving a **name to a recipe**. Instead of telling someone “go to the third shelf, second drawer, behind the flour” every time you want the recipe, you say: “Get the chocolate cake recipe.” In routing, instead of hard‑coding `"/products/5"`, you say: “Give me the URL for the `GetProduct` route with `id = 5`.” If the URL structure changes, the recipe name stays the same. |

### Purposes (each begins with “To”)

- **To** decouple URL generation from URL structure, so that template changes do not break links throughout the application.
- **To** provide a stable, human‑readable reference for endpoints in code, views, and API responses.
- **To** eliminate hard‑coded path strings, reducing the risk of broken links and typos.
- **To** support reliable `Location` headers in REST APIs (e.g., `CreatedAtRoute`).
- **To** integrate with OpenAPI/Swagger, where route names become operation IDs.
- **To** simplify link generation when route ordering or multiple candidate routes make value‑based generation complex.

### Syntax Rules and Structure

Route naming syntax differs between conventional and attribute routing:

| Strategy | Where the Name Is Defined | Syntax |
|----------|---------------------------|--------|
| **Conventional routing** | `Program.cs` | `app.MapControllerRoute(name: "routeName", pattern: "...")` |
| **Attribute routing** | Controller or action attribute | `[HttpGet("template", Name = "routeName")]` |

#### Component Breakdown (Conventional Routing)

| Component | Description |
|-----------|-------------|
| `name` | A unique string identifier for the route. |
| `pattern` | The route template (unchanged by naming). |

#### Component Breakdown (Attribute Routing)

| Component | Description |
|-----------|-------------|
| `Name` | The `Name` property on `[Route]`, `[HttpGet]`, `[HttpPost]`, etc. |
| `template` | The route template (unchanged by naming). |

#### Syntax Rules

- **Global uniqueness** – Route names must be unique across the entire application. Duplicate names throw an `InvalidOperationException` at startup.
- **Case sensitivity** – Route names are **case‑sensitive**; `"GetProduct"` and `"getproduct"` are distinct.
- **Reserved names** – The keywords `action`, `area`, `controller`, `handler`, and `page` are reserved and cannot be used as route names or parameter names.
- **Token replacement in attribute route names** – Route names defined on attribute routes can use token replacement. For example, `[Route("[controller]/[action]", Name = "[controller]_[action]")]` generates a unique name for each action.
- **No effect on matching** – Route names do not influence which endpoint matches a request; they are used only during URL generation.

#### Constraints and Limitations

- **Names are not templates** – A route name is an opaque identifier; it cannot contain route parameters or patterns.
- **No hierarchy** – Route names are flat strings; they do not imply parent‑child relationships.
- **Razor Pages difference** – Razor Pages does not provide the same simple route‑name mechanism; page names serve a similar purpose but are derived from the file path.

### Multiple Annotated Complete Step‑by‑Step Code Examples

#### Example 1: Naming a Conventional Route

```csharp
// Program.cs
var builder = WebApplication.CreateBuilder(args);
builder.Services.AddControllersWithViews();
var app = builder.Build();

app.UseRouting();

app.MapControllerRoute(
    name: "blog",
    pattern: "blog/{*article}",
    defaults: new { controller = "Blog", action = "Article" });

app.MapControllerRoute(
    name: "default",
    pattern: "{controller=Home}/{action=Index}/{id?}");

app.Run();
```

**Step‑by‑step explanation:**

1. `name: "blog"` assigns the name `"blog"` to the first route.
2. `name: "default"` assigns the name `"default"` to the second route.
3. Both names are unique and can be used later for URL generation.
4. The names do not affect how requests are matched; they are only used when generating links.

**Expected Output:** The application starts without errors. No observable output change from naming; the benefit appears when links are generated (see Section 3).

**Why this output?** Naming is a metadata operation; it does not alter runtime matching behavior. The names become available for `IUrlHelper.RouteUrl` and `LinkGenerator`.

---

#### Example 2: Naming an Attribute Route

```csharp
[Route("api/[controller]")]
[ApiController]
public class ProductsController : ControllerBase
{
    [HttpGet("{id:int}", Name = "GetProduct")]
    public IActionResult GetById(int id) => Ok(id);
}
```

**Step‑by‑step explanation:**

1. `[HttpGet("{id:int}", Name = "GetProduct")]` assigns the name `"GetProduct"` to the endpoint.
2. The name is used during URL generation, not during matching.
3. If another route also uses `"GetProduct"`, a startup exception occurs.

**Expected Output:** The endpoint matches `GET /api/Products/5` as usual. The name `"GetProduct"` is available for link generation.

**Why this output?** The `Name` property on the attribute attaches `EndpointNameMetadata` to the endpoint. Matching is unaffected; generation can now target this endpoint by name.

---

#### Example 3: Duplicate Route Name Causes Startup Error

```csharp
app.MapControllerRoute(name: "dup", pattern: "a/{id}");
app.MapControllerRoute(name: "dup", pattern: "b/{id}");
```

**Expected Output:** An `InvalidOperationException` is thrown at startup, indicating that multiple endpoints share the same name.

**Why this output?** The endpoint‑name address scheme requires unique names; duplicates make it impossible to resolve a name unambiguously.

### Real‑World Cases

- **E‑commerce API:** Naming the product‑detail route `"GetProduct"` and using `CreatedAtRoute("GetProduct", new { id })` in the POST response.
- **Blog:** Naming the article route `"BlogArticle"` and using `Url.RouteUrl("BlogArticle", new { slug })` in views.
- **Multi‑version API:** Using token replacement in attribute route names (`Name = "[controller]_[action]_v{version}"`) to generate unique names per version.

### References

- Routing in ASP.NET Core: Route names – https://learn.microsoft.com/en-us/aspnet/core/fundamentals/routing#route-names
- Conventional route names – https://learn.microsoft.com/en-us/aspnet/core/mvc/controllers/routing#conventional-route-names
- `EndpointNameMetadata` API – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.routing.endpointnamemetadata

---

## 2. Named Routes

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | A named route is a route endpoint that has been assigned a unique logical name, enabling it to be referenced by that name during URL generation. |
| **Technical Definition** | In conventional routing, a named route is created by passing a `name` argument to `MapControllerRoute`. In attribute routing, it is created by setting the `Name` property on a route attribute. The name is stored as `EndpointNameMetadata`. During link generation, the routing system resolves the name to the corresponding endpoint and evaluates its template with the supplied route values. |
| **Beginner‑Friendly Explanation** | A named route is a route with a **nickname**. You define it once (in `Program.cs` or on a controller action), and then you can say: “Give me the URL for the `Checkout` route” instead of writing the path manually. This makes links resilient to URL changes. |

### Purposes (each begins with “To”)

- **To** provide a stable reference for URL generation that survives template changes.
- **To** avoid hard‑coding paths in views, controllers, and API responses.
- **To** support `CreatedAtRoute` and `RouteUrl` calls in REST APIs and MVC views.
- **To** enable reliable link generation when route ordering or multiple candidate routes make value‑based generation complex.
- **To** integrate with OpenAPI, where route names become operation IDs.

### Syntax Rules and Structure

#### General Syntax (Conventional Routing)

```csharp
app.MapControllerRoute(
    name: "routeName",
    pattern: "template",
    defaults: new { controller = "...", action = "..." });
```

#### General Syntax (Attribute Routing)

```csharp
[HttpGet("template", Name = "routeName")]
[Route("template", Name = "routeName")]
```

#### Component Breakdown

| Component | Description |
|-----------|-------------|
| `routeName` | A unique string identifier (case‑sensitive). |
| `template` | The route template; unchanged by naming. |
| `Name` (attribute) | Property on `[Route]` and HTTP verb attributes. |

#### Syntax Rules

- **Uniqueness** – Names must be globally unique; duplicates throw at startup.
- **Case sensitivity** – Names are case‑sensitive.
- **Reserved names** – `action`, `area`, `controller`, `handler`, `page` cannot be used.
- **Token replacement** – Attribute route names can use `[controller]` and `[action]` tokens to generate unique names automatically.
- **Named routes in Razor Pages** – Razor Pages does not have the same route‑name mechanism; page names are used instead. A common workaround is to define route names as constants in a static class.

#### Constraints and Limitations

- **No matching impact** – Names do not affect request matching.
- **Razor Pages limitation** – The simple route‑name concept does not apply directly to Razor Pages.
- **Not a substitute for template design** – Names do not fix poorly designed templates; they only provide a reference for generation.

### Multiple Annotated Complete Step‑by‑Step Code Examples

#### Example 1: Named Conventional Route Used for Generation

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

1. The `"blog"` route is named and has a catch‑all parameter.
2. During generation, `Url.RouteUrl("blog", new { article = "my-post" })` produces `/blog/my-post`.

**Expected Output:**

| Generation Call | Result |
|-----------------|--------|
| `Url.RouteUrl("blog", new { article = "my-post" })` | `/blog/my-post` |
| `Url.RouteUrl("default", new { controller = "Products", action = "Details", id = 5 })` | `/Products/Details/5` |

**Why this output?** The name resolves to the endpoint, and the template is evaluated with the supplied route values.

---

#### Example 2: Named Attribute Route

```csharp
[HttpGet("{id:int}", Name = "GetProduct")]
public IActionResult GetById(int id) => Ok(id);
```

**Step‑by‑step explanation:**

1. The action is named `"GetProduct"`.
2. `CreatedAtRoute("GetProduct", new { id = 42 }, product)` returns `201 Created` with `Location: /api/Products/42`.

**Expected Output:**

| API Call | Response |
|----------|----------|
| `POST /api/Products` (creates product 42) | `201 Created`, `Location: /api/Products/42` |

**Why this output?** `CreatedAtRoute` resolves the name to the endpoint and generates the URL using the supplied route values.

---

#### Example 3: Token Replacement in Attribute Route Names

```csharp
[Route("[controller]/[action]", Name = "[controller]_[action]")]
public class BooksController : Controller
{
    public IActionResult List() => Content("List");
    public IActionResult Details(int id) => Content($"Details {id}");
}
```

**Step‑by‑step explanation:**

1. The route name template `"[controller]_[action]"` is replaced with `Books_List` and `Books_Details`.
2. Each action gets a unique route name automatically.

**Expected Output:**

| Action | Generated Route Name |
|--------|----------------------|
| `List` | `"Books_List"` |
| `Details` | `"Books_Details"` |

**Why this output?** Token replacement is applied as the final step of building attribute routes, including route names.

### Real‑World Cases

- **REST API creation:** Using `CreatedAtRoute("GetProduct", new { id })` in a `POST` action to return the canonical URL of the created resource.
- **MVC views:** Using `Url.RouteUrl("BlogArticle", new { slug })` to generate links to blog posts.
- **Multi‑version APIs:** Using token replacement in route names to generate unique names per controller and action.

### References

- Conventional route names – https://learn.microsoft.com/en-us/aspnet/core/mvc/controllers/routing#conventional-route-names
- Attribute route names – https://learn.microsoft.com/en-us/aspnet/core/mvc/controllers/routing#attribute-routing-for-rest-apis
- Token replacement in attribute routes – https://learn.microsoft.com/en-us/aspnet/core/mvc/controllers/routing#token-replacement-in-attribute-routing

---

## 3. Route Generation

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Route generation (URL generation) is the process of constructing a URL from a route name and a set of route values. |
| **Technical Definition** | URL generation begins with a call to `LinkGenerator.GetPathByName` (or `IUrlHelper.RouteUrl`). The **endpoint‑name address scheme** resolves the name to a candidate endpoint. Each candidate’s `RoutePattern` is evaluated with the supplied route values; the first pattern that can be successfully expanded produces the URL. `LinkGenerator` is a singleton service available through DI and can be used outside the context of a running request. `IUrlHelper` (and Tag Helpers, HTML Helpers, and action results) use `LinkGenerator` internally. |
| **Beginner‑Friendly Explanation** | Route generation is **building a URL on demand**. You say: “I need a link to the `GetProduct` route with `id = 5`,” and the framework produces `/api/Products/5`. You don’t need to know the exact path structure; the named route handles it. |

### Purposes (each begins with “To”)

- **To** generate URLs without hard‑coding paths, making links resilient to template changes.
- **To** produce correct `Location` headers in REST APIs (`CreatedAtRoute`).
- **To** generate links in views and Razor Pages using Tag Helpers (`asp-route`) or `Url.RouteUrl`.
- **To** support link generation outside the request context (e.g., in background services) via `LinkGenerator`.
- **To** ensure that generated URLs satisfy route constraints and parameter requirements.

### Syntax Rules and Structure

#### General Syntax (IUrlHelper)

```csharp
Url.RouteUrl("routeName", new { param1 = value1, param2 = value2 })
```

#### General Syntax (LinkGenerator)

```csharp
linkGenerator.GetPathByName("routeName", new { param1 = value1 })
linkGenerator.GetPathByRouteName("routeName", values) // alternate overload
```

#### General Syntax (Action Result)

```csharp
return CreatedAtRoute("routeName", new { id = 42 }, value);
```

#### Component Breakdown

| Component | Description |
|-----------|-------------|
| `routeName` | The unique name of the target route. |
| `routeValues` | An object containing parameter values to expand the template. |
| `LinkGenerator` | Singleton service injected via DI; supports `GetPath*` (path only) and `GetUri*` (absolute URI). |
| `IUrlHelper` | Available in controllers and views (`Url` property); delegates to `LinkGenerator`. |

#### Syntax Rules

- **Ambient values** – `IUrlHelper` uses the current request’s route values as ambient values unless overridden. `LinkGenerator` can operate outside a request context.
- **Explicit values** – Values passed to the generation call override ambient values.
- **Route value invalidation** – The routing system decides which ambient values to reuse and which to discard; this is a common source of `null` results.
- **GetPath vs. GetUri** – `GetPathByName` returns a path (e.g., `/api/Products/5`); `GetUriByName` returns an absolute URI (including scheme and host).
- **First successful match wins** – URL generation does not check for ambiguity; the first endpoint whose pattern can be expanded produces the result.

#### Constraints and Limitations

- **`null` results** – Generation returns `null` if no endpoint with the given name can expand the template with the supplied values. Common causes include missing required route values or route value invalidation.
- **Host header security** – `GetUri*` methods use the incoming request’s host header; ensure the host header is validated in production to prevent host‑header injection.
- **Razor Pages differences** – `IUrlHelper.Page` overrides the current handler route value with `null` unless explicitly overridden; `LinkGenerator.GetPathByPage` duplicates this behaviour for compatibility.

### Multiple Annotated Complete Step‑by‑Step Code Examples

#### Example 1: `IUrlHelper.RouteUrl` in a Controller

```csharp
public class HomeController : Controller
{
    public IActionResult Index()
    {
        var url = Url.RouteUrl("GetProduct", new { id = 5 });
        return Content($"URL: {url}");
    }
}
```

**Step‑by‑step explanation:**

1. `Url.RouteUrl("GetProduct", new { id = 5 })` resolves the name `"GetProduct"`.
2. The routing system finds the endpoint with that name and evaluates its template with `id = 5`.
3. The generated URL is returned as a string.

**Expected Output:** `URL: /api/Products/5` (assuming the route template is `api/Products/{id:int}`).

**Why this output?** `IUrlHelper.RouteUrl` delegates to `LinkGenerator`, which resolves the name and expands the template with the supplied values.

---

#### Example 2: `LinkGenerator.GetPathByName` in a Minimal API or Service

```csharp
app.MapGet("/products/{id:int}", (int id) => $"Product {id}")
   .WithName("GetProduct");

app.MapGet("/", (LinkGenerator linker) =>
{
    var url = linker.GetPathByName("GetProduct", new { id = 5 });
    return $"Link: {url}";
});
```

**Step‑by‑step explanation:**

1. The product endpoint is named `"GetProduct"` using `WithName`.
2. `LinkGenerator` is injected into the root endpoint.
3. `GetPathByName("GetProduct", new { id = 5 })` produces the path.

**Expected Output:** `Link: /products/5`

**Why this output?** `LinkGenerator` resolves the endpoint name and expands the template. It works even outside the context of a specific request.

---

#### Example 3: `CreatedAtRoute` in a REST API

```csharp
[HttpPost]
public IActionResult Create([FromBody] ProductDto dto)
{
    var id = 42;
    return CreatedAtRoute("GetProduct", new { id }, dto);
}
```

**Step‑by‑step explanation:**

1. `CreatedAtRoute` resolves the name `"GetProduct"`.
2. The route values `{ id = 42 }` are used to expand the template.
3. A `201 Created` response is returned with a `Location` header.

**Expected Output:** `201 Created`, `Location: /api/Products/42`.

**Why this output?** `CreatedAtRoute` uses the named route to produce the canonical URL of the newly created resource.

### Real‑World Cases

- **REST API creation:** `CreatedAtRoute` returns the `Location` header for a new resource.
- **MVC views:** `Url.RouteUrl("BlogArticle", new { slug })` generates blog post links.
- **Background services:** `LinkGenerator` generates links in emails or reports without an active HTTP request.
- **OpenAPI:** Route names become operation IDs, and generated URLs appear in API documentation.

### References

- URL generation in ASP.NET Core – https://learn.microsoft.com/en-us/aspnet/core/fundamentals/routing#url-generation
- `LinkGenerator` API – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.routing.linkgenerator
- `IUrlHelper.RouteUrl` – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.iurlhelper.routeurl
- `CreatedAtRoute` – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.controllerbase.createdatroute
- URL generation troubleshooting – https://learn.microsoft.com/en-us/aspnet/core/fundamentals/routing#troubleshoot-url-generation

---

## 4. Route Naming Best Practices

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Route naming best practices are guidelines for choosing names that are unique, stable, descriptive, and maintainable across the application’s lifetime. |
| **Technical Definition** | Best practices encompass naming conventions (PascalCase, descriptive), uniqueness management (especially in versioned APIs), token replacement for attribute routes, constant‑based names for Razor Pages, and the separation of route names from template strings. |
| **Beginner‑Friendly Explanation** | Best practices are the “rules of thumb” for naming routes. For example: give routes clear, descriptive names (like `"GetProduct"` instead of `"Route1"`), keep names unique, and don’t change them casually because other code depends on them. |

### Purposes (each begins with “To”)

- **To** prevent startup exceptions caused by duplicate route names.
- **To** make link generation code self‑documenting and easy to understand.
- **To** ensure that route names remain stable even as templates change.
- **To** simplify maintenance in large applications and versioned APIs.
- **To** avoid reserved keywords that cause runtime errors.

### Guideline Checklist

| Guideline | Rationale | Example |
|-----------|-----------|---------|
| **Use descriptive names** | Names should indicate the endpoint’s purpose. | `"GetProduct"`, `"ListOrders"` |
| **Ensure global uniqueness** | Duplicates throw at startup. | Prefix with controller/version if needed. |
| **Use PascalCase or snake_case consistently** | Consistency improves readability. | `"GetProduct"` or `"get_product"` |
| **Avoid reserved keywords** | `action`, `area`, `controller`, `handler`, `page` cause errors. | Use `"ProductAction"` instead. |
| **Prefer token replacement in attribute routes** | Automatically generates unique names. | `Name = "[controller]_[action]"` |
| **Keep names stable** | Changing a name breaks all code that references it. | Treat names as public API. |
| **Use constants for Razor Pages** | Razor Pages lacks built‑in route names. | Static class with `const string` properties. |
| **Document route names** | Names are part of the application’s contract. | Maintain a route‑name reference. |

#### Constraints and Limitations

- **Razor Pages** – The simple route‑name mechanism does not apply directly; use page names or constant classes.
- **Versioned APIs** – Route names must be unique per version; token replacement or version prefixes help.
- **OpenAPI operation IDs** – Route names become operation IDs; avoid spaces and special characters.

### Multiple Annotated Complete Step‑by‑Step Code Examples

#### Example 1: Descriptive and Unique Names in a REST API

```csharp
[Route("api/v1/[controller]")]
[ApiController]
public class ProductsController : ControllerBase
{
    [HttpGet("{id:int}", Name = "GetProduct_v1")]
    public IActionResult GetById(int id) => Ok(id);

    [HttpGet(Name = "ListProducts_v1")]
    public IActionResult GetAll() => Ok();
}
```

**Step‑by‑step explanation:**

1. Names are descriptive (`"GetProduct_v1"`, `"ListProducts_v1"`).
2. The `_v1` suffix ensures uniqueness if a future `v2` controller uses the same base names.
3. `CreatedAtRoute("GetProduct_v1", ...)` will generate the correct v1 URL.

**Expected Output:** Unique route names are registered; no startup conflict occurs.

**Why this output?** Version‑specific suffixes prevent duplicate names across API versions.

---

#### Example 2: Token Replacement for Unique Attribute Route Names

```csharp
[Route("[controller]/[action]", Name = "[controller]_[action]")]
public class BooksController : Controller
{
    public IActionResult List() => Content("List");
    public IActionResult Details(int id) => Content($"Details {id}");
}
```

**Step‑by‑step explanation:**

1. The name template `"[controller]_[action]"` produces `"Books_List"` and `"Books_Details"`.
2. Each action receives a unique name without manual effort.

**Expected Output:**

| Action | Route Name |
|--------|------------|
| `List` | `"Books_List"` |
| `Details` | `"Books_Details"` |

**Why this output?** Token replacement is applied as the final step of building attribute routes, including names.

---

#### Example 3: Constants for Razor Pages Route Names

```csharp
public static class RouteNames
{
    public const string ProductDetails = "ProductDetails";
    public const string Checkout = "Checkout";
}

// Usage in a Razor Page link
<a asp-route="@RouteNames.ProductDetails" asp-route-id="5">View Product</a>
```

**Step‑by‑step explanation:**

1. Razor Pages lacks built‑in route names, so a static class holds constants.
2. Pages reference the constants instead of hard‑coded strings.
3. Changing a route name requires updating only the constant.

**Expected Output:** Links are generated using the constant values, remaining consistent across the application.

**Why this output?** This pattern provides a centralised, maintainable naming scheme for Razor Pages.

### Real‑World Cases

- **Versioned APIs:** Suffixing route names with the version (`"GetProduct_v1"`) avoids conflicts when multiple API versions coexist.
- **Large MVC applications:** Using token replacement in attribute route names reduces manual naming overhead.
- **Razor Pages:** Constant classes provide a workaround for the lack of native route names.

### References

- Routing best practices – https://learn.microsoft.com/en-us/aspnet/core/fundamentals/routing#routing-best-practices
- Conventional route names – https://learn.microsoft.com/en-us/aspnet/core/mvc/controllers/routing#conventional-route-names
- Reserved routing names – https://learn.microsoft.com/en-us/aspnet/core/fundamentals/routing#reserved-routing-names
- Token replacement in attribute routing – https://learn.microsoft.com/en-us/aspnet/core/mvc/controllers/routing#token-replacement-in-attribute-routing

---

## Summary Table: Route Naming at a Glance

| Concept | Primary Purpose | Key Syntax | When to Use |
|---------|----------------|------------|-------------|
| **Route Naming** | Unique logical identifier for an endpoint | `name:` (conventional), `Name =` (attribute) | Whenever links to the endpoint must be generated |
| **Named Routes** | Routes with a stable reference | `MapControllerRoute(name: "GetProduct", ...)`, `[HttpGet(..., Name = "GetProduct")]` | REST APIs, MVC views, any link generation |
| **Route Generation** | Build URLs from names | `Url.RouteUrl("GetProduct", new { id = 5 })`, `LinkGenerator.GetPathByName("GetProduct", new { id = 5 })` | `CreatedAtRoute`, view links, background services |
| **Best Practices** | Maintainability and correctness | Descriptive names, global uniqueness, token replacement, constants for Razor Pages | All production applications |

---

## References (Full List)

- Routing in ASP.NET Core – https://learn.microsoft.com/en-us/aspnet/core/fundamentals/routing
- Route names – https://learn.microsoft.com/en-us/aspnet/core/fundamentals/routing#route-names
- URL generation – https://learn.microsoft.com/en-us/aspnet/core/fundamentals/routing#url-generation
- Troubleshoot URL generation – https://learn.microsoft.com/en-us/aspnet/core/fundamentals/routing#troubleshoot-url-generation
- Conventional route names – https://learn.microsoft.com/en-us/aspnet/core/mvc/controllers/routing#conventional-route-names
- Attribute routing for REST APIs – https://learn.microsoft.com/en-us/aspnet/core/mvc/controllers/routing#attribute-routing-for-rest-apis
- Token replacement in attribute routing – https://learn.microsoft.com/en-us/aspnet/core/mvc/controllers/routing#token-replacement-in-attribute-routing
- Reserved routing names – https://learn.microsoft.com/en-us/aspnet/core/fundamentals/routing#reserved-routing-names
- `LinkGenerator` API – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.routing.linkgenerator
- `IUrlHelper.RouteUrl` – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.iurlhelper.routeurl
- `CreatedAtRoute` – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.controllerbase.createdatroute
- `EndpointNameMetadata` – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.routing.endpointnamemetadata
- Routing best practices – https://learn.microsoft.com/en-us/aspnet/core/fundamentals/routing#routing-best-practices
- Razor Pages URL generation – https://learn.microsoft.com/en-us/aspnet/core/razor-pages/#url-generation-for-pages