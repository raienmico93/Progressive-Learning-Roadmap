# Route Debugging in ASP.NET Core MVC: A Comprehensive Cheat Sheet

> **Version Scope & Compatibility Note**
> This cheat sheet focuses on **ASP.NET Core MVC** (3.0 and later, including .NET 6/7/8/10/11) and the **endpoint routing** system that became the default in ASP.NET Core 3.0. Route debugging techniques differ between **endpoint routing** (modern) and legacy `IRouter`-based routing (pre-3.0); this guide concentrates on the former. **RouteDebugger middleware** (`Edi.RouteDebugger`) and **RoutesList** are community tools compatible with .NET Core 3.1 through .NET 9.0. **EndpointDataSource** and **LinkGenerator** diagnostics are first-party, framework-provided mechanisms.

---

## Topic Overview

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Route debugging is the systematic process of inspecting, diagnosing, and resolving problems related to how an ASP.NET Core application matches incoming HTTP requests to endpoints and generates URLs from route definitions. |
| **Technical Definition** | Route debugging encompasses **logging diagnostics** (`Microsoft.AspNetCore.Routing` category), **route table introspection** (via `EndpointDataSource`), **middleware-based route inspection** (RouteDebugger), **URL generation troubleshooting** (`LinkGenerator` trace logs), **ambiguous match detection** (ASP0022 diagnostic), and **performance analysis** of route matching (DFA node analysis). It leverages the framework’s built-in diagnostic instrumentation (`DiagnosticListener`, `ILogger`) and extensibility points (`EndpointDataSource`, `IEnumerable<EndpointDataSource>`) to expose the routing system’s internal state. |
| **Beginner-Friendly Explanation** | Route debugging is **figuring out why a URL isn’t going where you expect**. Did you visit `/products/5` but get a 404? Did a link generate the wrong path? Route debugging tools show you the **list of all routes** your app knows about, **which route matched** (or didn’t match) a request, and **why**—so you can fix typos, ordering issues, or constraint mistakes. |

### Key Characteristics

- **Multi-layered diagnostics** – Combines **logging** (log levels), **introspection** (route table inspection), **middleware tools** (RouteDebugger), and **compiler diagnostics** (ASP0022).
- **Development-focused** – Most techniques are intended for the **Development environment**; exposing route tables in production is a security risk.
- **Endpoint-centric** – Modern diagnostics operate on `Endpoint` objects, `RoutePattern`, and `EndpointMetadata` rather than legacy `RouteCollection` structures.
- **Extensible** – The `EndpointDataSource` abstraction allows custom diagnostic endpoints to be registered.
- **Performance-aware** – Debugging includes identifying **large route tables** and **badly-behaving custom middleware** that degrade routing performance.

### Prerequisites

- **.NET SDK** (6.0+ recommended; examples use .NET 8/10 minimal hosting).
- **ASP.NET Core MVC** services registered (`AddControllersWithViews()` or `AddControllers()`).
- **`Microsoft.Extensions.Logging`** configured (default in ASP.NET Core templates).
- For community tools: NuGet package installation access (`Edi.RouteDebugger` or `RoutesList`).

### Related Programming Areas

- **Logging** – `Microsoft.AspNetCore.Routing` and `Microsoft.AspNetCore.Mvc` log categories.
- **Endpoint Routing** – `EndpointDataSource`, `RouteEndpoint`, `RoutePattern`.
- **URL Generation** – `LinkGenerator` trace logs for link-generation failures.
- **Diagnostic Listeners** – `DiagnosticListener` events for endpoint matching.
- **Middleware Pipeline** – Ordering and custom middleware diagnostics.
- **Route Constraints** – Constraint failures are a common cause of mismatches.

### Core Concepts / Features

1. **Route Debugging** – The overarching diagnostic process and its mechanisms.
2. **Route List** – Inspecting all registered endpoints via `EndpointDataSource` or tools like RoutesList / RouteDebugger.
3. **Route Matching** – Understanding how the router selects (or fails to select) an endpoint, using logs, diagnostics, and compiler warnings.
4. **Route Debugging Best Practices** – Guidelines for safe, effective, and performant debugging.

Each concept is elaborated using the uniform structure: Definitions, Purposes, Syntax Rules & Structure, Annotated Code Examples with Expected Outputs, Real-World Cases, and References.

---

## 1. Route Debugging (General)

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Route debugging is the practice of using diagnostic tools, logs, and introspection APIs to understand why a request did or did not match an endpoint, or why a generated URL is incorrect. |
| **Technical Definition** | ASP.NET Core routing emits diagnostic information through multiple channels: `ILogger` (`Microsoft.AspNetCore.Routing` category), `DiagnosticListener` (`Microsoft.AspNetCore.Routing.EndpointMatched` events), and extensibility points (`EndpointDataSource` for route table inspection). Debugging typically involves setting log levels to `Debug` or `Trace`, injecting `EndpointDataSource` to enumerate routes, and using middleware such as `Edi.RouteDebugger` to expose current-request route information via headers and a dedicated endpoint. Compiler diagnostic **ASP0022** detects ambiguous route conflicts at build time. |
| **Beginner-Friendly Explanation** | Route debugging is **being a detective for URLs**. When a URL behaves unexpectedly, you use special tools to see: “What routes does my app actually have?” “Which route (if any) matched this request?” “Why did it fail?” The tools range from **logging** (reading diagnostic messages) to **route listers** (printing every route) to **middleware debuggers** (showing the matched route in the response). |

### Purposes (each begins with “To”)

- **To** identify why a request returns 404 or is routed to the wrong endpoint.
- **To** verify that all expected routes are registered and that no unintended routes exist.
- **To** diagnose URL generation failures (e.g., `CreatedAtRoute` returning `null`).
- **To** detect ambiguous route conflicts before they cause runtime errors.
- **To** analyse routing performance issues such as large route tables or middleware delays.
- **To** provide a learning aid for understanding how routing works in a specific application.

### Syntax Rules and Structure

#### General Diagnostic Mechanisms

| Mechanism | How It Works | Scope |
|-----------|--------------|-------|
| **Logging** | Set `Logging:LogLevel:Microsoft` to `Debug` (or `Microsoft.AspNetCore.Routing` to `Trace`). | Framework-wide diagnostic output. |
| **`EndpointDataSource`** | Inject `IEnumerable<EndpointDataSource>` and enumerate `Endpoints`. | Route table introspection. |
| **RouteDebugger Middleware** | `app.UseRouteDebugger()` adds `x-aspnet-route` header and a `/route-debugger` endpoint. | Current route + full route list. |
| **RoutesList** | NuGet tool for visualising and analysing all routes with filtering and custom output formats. | Route table inspection and documentation. |
| **`LinkGenerator` Trace Logs** | Set `Microsoft.AspNetCore.Routing` to `Trace` to see link-generation details. | URL generation debugging. |
| **ASP0022 Compiler Diagnostic** | Emitted when a route conflict is detected at build time. | Ambiguous route detection. |

#### Syntax Rules

- **Log levels** – `Debug` for general routing diagnostics; `Trace` for detailed `LinkGenerator` processing.
- **Development-only** – RouteDebugger and custom route-list endpoints should be enabled **only in Development** to avoid information disclosure.
- **Middleware order** – RouteDebugger must be placed **before** `UseDeveloperExceptionPage()` if both are used.
- **`EndpointDataSource`** – Inject `IEnumerable<EndpointDataSource>` anywhere DI is available; it provides all registered endpoints.

#### Constraints and Limitations

- **Security risk** – Exposing route tables or RouteDebugger in production reveals internal endpoint names, patterns, and middleware structure.
- **Performance overhead** – Debug-level logging and route enumeration can be expensive; use only during development or targeted troubleshooting.
- **Endpoint routing only** – `EndpointDataSource` and RouteDebugger (modern versions) target endpoint routing (ASP.NET Core 3.0+); legacy `IRouter` apps require different approaches.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Enabling Debug Logging for Routing

```json
// appsettings.Development.json
{
  "Logging": {
    "LogLevel": {
      "Default": "Information",
      "Microsoft": "Debug",
      "Microsoft.Hosting.Lifetime": "Information"
    }
  }
}
```

**Step-by-step explanation:**

1. The `"Microsoft": "Debug"` setting enables debug-level logging for all Microsoft namespaces, including routing.
2. For more targeted logging, replace `"Microsoft"` with `"Microsoft.AspNetCore.Routing"` to isolate routing diagnostics.
3. Run the application and observe the **Debug** output window or console for messages about route matching, constraint evaluation, and endpoint selection.

**Expected Output:** Debug-level messages such as `Request matched endpoint '...'`, `Endpoint '...' did not match`, and constraint evaluation details appear in the log.

**Why this output?** The routing middleware uses `ILogger` to emit diagnostic messages; setting the log level to `Debug` surfaces these messages.

---

#### Example 2: Injecting `EndpointDataSource` to List All Routes

```csharp
// Minimal API: /debug/routes endpoint
app.MapGet("/debug/routes", (IEnumerable<EndpointDataSource> endpointSources) =>
{
    var sb = new StringBuilder();
    var endpoints = endpointSources.SelectMany(es => es.Endpoints);
    foreach (var endpoint in endpoints)
    {
        if (endpoint is RouteEndpoint routeEndpoint)
        {
            sb.AppendLine($"Pattern: {routeEndpoint.RoutePattern.RawText}");
            var httpMethods = endpoint.Metadata
                .OfType<HttpMethodMetadata>()
                .FirstOrDefault()?.HttpMethods;
            sb.AppendLine($"Methods: {string.Join(", ", httpMethods ?? new[] { "ANY" })}");
            var routeName = endpoint.Metadata
                .OfType<RouteNameMetadata>()
                .FirstOrDefault()?.RouteName;
            sb.AppendLine($"Name: {routeName ?? "(unnamed)"}");
            sb.AppendLine(new string('-', 40));
        }
    }
    return sb.ToString();
});
```

**Step-by-step explanation:**

1. `IEnumerable<EndpointDataSource>` is injected via DI; it aggregates all registered endpoint sources.
2. `SelectMany(es => es.Endpoints)` flattens all endpoints.
3. `RoutePattern.RawText` provides the original template string.
4. `HttpMethodMetadata` reveals the HTTP methods the endpoint responds to.
5. `RouteNameMetadata` provides the route name (if named).
6. The result is returned as plain text.

**Expected Output:** A list of all registered routes with their patterns, HTTP methods, and names, e.g.:

```
Pattern: api/Products/{id:int}
Methods: GET
Name: GetProduct
----------------------------------------
Pattern: /
Methods: GET
Name: (unnamed)
----------------------------------------
```

**Why this output?** `EndpointDataSource` is the framework’s route table abstraction; enumerating it exposes every registered endpoint.

---

#### Example 3: Using `Edi.RouteDebugger` Middleware

```csharp
// Install: dotnet add package Edi.RouteDebugger

var app = builder.Build();

if (app.Environment.IsDevelopment())
{
    // Must be BEFORE UseDeveloperExceptionPage()
    app.UseRouteDebugger(); // Default path: /route-debugger
    app.UseDeveloperExceptionPage();
}

app.UseRouting();
app.MapControllers();
app.Run();
```

**Step-by-step explanation:**

1. `Edi.RouteDebugger` is a NuGet package (`dotnet add package Edi.RouteDebugger`).
2. `UseRouteDebugger()` adds middleware that:
   - Adds an `x-aspnet-route` response header containing the **current matched route** information.
   - Exposes a `/route-debugger` endpoint listing **all routes** in the application.
3. The middleware is placed **before** `UseDeveloperExceptionPage()` to ensure the exception page still works.

**Expected Output:**

| Action | Observation |
|--------|-------------|
| Visit any page and inspect response headers | `x-aspnet-route` header contains route information. |
| Navigate to `/route-debugger` | A page (or JSON) listing all registered routes. |

**Why this output?** The middleware intercepts requests, captures route data, and exposes it for debugging.

### Real-World Cases

- **404 on a new API endpoint:** Enable debug logging and inspect `EndpointDataSource` to confirm the route is registered and the template is correct.
- **Wrong controller action invoked:** Use RouteDebugger’s `x-aspnet-route` header to see which endpoint was actually selected.
- **URL generation returns `null`:** Set `Microsoft.AspNetCore.Routing` to `Trace` to see `LinkGenerator`’s decision-making process.

### References

- Routing in ASP.NET Core: Debug diagnostics – https://learn.microsoft.com/en-us/aspnet/core/fundamentals/routing#debug-diagnostics
- `EndpointDataSource` API – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.routing.endpointdatasource
- ASP.NET Core RouteDebugger Middleware – https://github.com/ediwang/edi.routedebugger
- RoutesList NuGet Package – https://www.nuget.org/packages/RoutesList/
- Microsoft Q&A: Routing in ASP.NET Core (EndpointDataSource example) – https://learn.microsoft.com/en-us/answers/questions/1923251/routing-in-asp-net-core
- Stack Overflow: How to get a list of all routes in ASP.NET Core? – https://stackoverflow.com/questions/28435734/how-to-get-a-list-of-all-routes-in-asp-net-core

---

## 2. Route List

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | A route list is a comprehensive enumeration of all endpoints registered in an ASP.NET Core application, including their URL patterns, HTTP methods, route names, and associated metadata. |
| **Technical Definition** | The route list is represented by the collection of `Endpoint` objects aggregated from all `EndpointDataSource` instances registered in DI. In endpoint routing, the route table is built at startup from multiple sources: controller action descriptors (attribute routing and conventional routing), Razor Page conventions, minimal API mappings, and SignalR/gRPC endpoints. The list can be obtained by injecting `IEnumerable<EndpointDataSource>` and projecting each `RouteEndpoint`’s `RoutePattern`, `HttpMethodMetadata`, `RouteNameMetadata`, and `ControllerActionDescriptor`. |
| **Beginner-Friendly Explanation** | A route list is **an inventory of every URL your app understands**. Just like a restaurant menu lists every dish, the route list shows every endpoint—what URL pattern it responds to, what HTTP methods it accepts, and what controller/action handles it. If a URL isn’t on the list, your app doesn’t know it. |

### Purposes (each begins with “To”)

- **To** verify that all expected routes are registered and that no unintended routes exist.
- **To** identify duplicate or conflicting route patterns.
- **To** document the application’s URL surface for API consumers or internal teams.
- **To** debug 404 errors by confirming whether a route exists for the requested URL.
- **To** audit route names, HTTP methods, and controller/action mappings.
- **To** analyse route table size and complexity for performance tuning.

### Syntax Rules and Structure

#### General Syntax (`EndpointDataSource` Projection)

```csharp
var endpoints = endpointSources
    .SelectMany(es => es.Endpoints)
    .OfType<RouteEndpoint>();

foreach (var endpoint in endpoints)
{
    // Access:
    // endpoint.RoutePattern.RawText
    // endpoint.Metadata.OfType<HttpMethodMetadata>()
    // endpoint.Metadata.OfType<RouteNameMetadata>()
    // endpoint.Metadata.OfType<ControllerActionDescriptor>()
}
```

#### Component Breakdown

| Component | Description |
|-----------|-------------|
| `EndpointDataSource` | Aggregates endpoint sources; injected as `IEnumerable<EndpointDataSource>`. |
| `RouteEndpoint` | An endpoint with a `RoutePattern`; filter from general `Endpoint` objects. |
| `RoutePattern.RawText` | The original template string (e.g., `"api/[controller]/{id:int}"`). |
| `HttpMethodMetadata` | Contains accepted HTTP methods (`GET`, `POST`, etc.). |
| `RouteNameMetadata` | Contains the route name (if named). |
| `ControllerActionDescriptor` | Contains controller and action names (for MVC endpoints). |

#### Syntax Rules

- **Multiple data sources** – Always inject `IEnumerable<EndpointDataSource>` (not a single instance) to capture all sources.
- **Filter to `RouteEndpoint`** – Some endpoints may not have a route pattern; filtering ensures safe access to `RoutePattern`.
- **Development-only** – Exposing the route list via an endpoint should be restricted to Development to avoid information disclosure.
- **Community tools** – RoutesList and RouteDebugger provide pre-built route listing with filtering and formatting options.

#### Constraints and Limitations

- **Razor Pages partial metadata** – Razor Pages endpoints may not expose HTTP methods through `HttpMethodMetadata` in the same way as controller endpoints.
- **Performance** – Enumerating a very large route table can be slow; consider pagination or filtering.
- **Security** – Route lists reveal internal endpoint names and patterns; never expose in production without authentication and authorization.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Controller-Based Route Listing Endpoint

```csharp
[Route("/-/debug")]
public class DebugController : Controller
{
    private readonly IEnumerable<EndpointDataSource> _endpointSources;

    public DebugController(IEnumerable<EndpointDataSource> endpointSources)
    {
        _endpointSources = endpointSources;
    }

    [HttpGet("routes")]
    public IActionResult ListAllRoutes()
    {
        var endpoints = _endpointSources
            .SelectMany(es => es.Endpoints)
            .OfType<RouteEndpoint>();

        var output = endpoints.Select(e =>
        {
            var controller = e.Metadata
                .OfType<ControllerActionDescriptor>()
                .FirstOrDefault();
            var httpMethods = e.Metadata
                .OfType<HttpMethodMetadata>()
                .FirstOrDefault()?.HttpMethods;

            return new
            {
                Method = httpMethods != null ? string.Join(", ", httpMethods) : "ANY",
                Route = $"/{e.RoutePattern.RawText.TrimStart('/')}",
                Action = controller != null
                    ? $"{controller.ControllerName}.{controller.ActionName}"
                    : null,
                ControllerMethod = controller != null
                    ? $"{controller.ControllerTypeInfo.FullName}:{controller.MethodInfo.Name}"
                    : null
            };
        });

        return Json(output);
    }
}
```

**Step-by-step explanation:**

1. `IEnumerable<EndpointDataSource>` is injected into the controller.
2. Endpoints are flattened and filtered to `RouteEndpoint`.
3. For each endpoint, `ControllerActionDescriptor` provides controller/action names.
4. `HttpMethodMetadata` provides accepted HTTP methods.
5. The result is returned as JSON.

**Expected Output** (abbreviated):

```json
[
  {
    "method": "GET",
    "route": "/-/debug/routes",
    "action": "Debug.ListAllRoutes",
    "controllerMethod": "MyApp.Controllers.DebugController:ListAllRoutes"
  },
  {
    "method": "GET",
    "route": "/api/Products/{id:int}",
    "action": "Products.GetById",
    "controllerMethod": "MyApp.Controllers.ProductsController:GetById"
  }
]
```

**Why this output?** The projection exposes the route pattern, HTTP methods, and controller/action mapping for every MVC endpoint.

---

#### Example 2: Minimal API Route Listing Endpoint

```csharp
app.MapGet("/debug/routes", (IEnumerable<EndpointDataSource> endpointSources) =>
{
    var endpoints = endpointSources
        .SelectMany(es => es.Endpoints)
        .OfType<RouteEndpoint>()
        .Select(e => new
        {
            Pattern = e.RoutePattern.RawText,
            Methods = e.Metadata
                .OfType<HttpMethodMetadata>()
                .FirstOrDefault()?.HttpMethods ?? new[] { "ANY" },
            Name = e.Metadata
                .OfType<RouteNameMetadata>()
                .FirstOrDefault()?.RouteName
        });

    return Results.Json(endpoints);
});
```

**Step-by-step explanation:**

1. The endpoint is mapped at `/debug/routes`.
2. `EndpointDataSource` is injected as a minimal API parameter.
3. The projection includes pattern, methods, and route name.

**Expected Output:** JSON array of route descriptors.

**Why this output?** Minimal APIs support the same `EndpointDataSource` introspection as controllers.

---

#### Example 3: Using RoutesList NuGet Tool

```bash
# Install the tool
dotnet add package RoutesList
```

```csharp
// In Program.cs (Development only)
if (app.Environment.IsDevelopment())
{
    app.UseRoutesList(); // Exposes a route list UI/endpoint
}
```

**Step-by-step explanation:**

1. RoutesList is a NuGet package compatible with .NET Core 3.1–.NET 9.0.
2. It provides **comprehensive inspection** of HTTP methods, URIs, controllers, actions, middleware, and routing configurations.
3. Output formats and filtering are customizable.

**Expected Output:** A browsable route list (HTML or JSON, depending on configuration) showing all application routes.

**Why this output?** RoutesList provides a pre-built, user-friendly route-listing interface.

### Real-World Cases

- **API documentation:** Generating a route list to document available endpoints for front-end teams.
- **Pre-deployment audit:** Verifying that no debug or test routes are accidentally registered in production.
- **404 investigation:** Confirming that a route with the expected pattern is (or is not) present in the route table.

### References

- `EndpointDataSource` API – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.routing.endpointdatasource
- Microsoft Q&A: Routing in ASP.NET Core (route table dumper) – https://learn.microsoft.com/en-us/answers/questions/1923251/routing-in-asp-net-core
- Stack Overflow: How to get a list of all routes in ASP.NET Core? – https://stackoverflow.com/questions/28435734/how-to-get-a-list-of-all-routes-in-asp-net-core
- RoutesList NuGet Package – https://www.nuget.org/packages/RoutesList/
- ASP.NET Core RouteDebugger Middleware – https://github.com/ediwang/edi.routedebugger

---

## 3. Route Matching

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Route matching is the algorithm that compares an incoming request’s URL path against registered route templates and selects the best-matching endpoint, or determines that no endpoint matches. |
| **Technical Definition** | Endpoint routing matching proceeds in **phases**: (1) candidate collection from the `EndpointDataSource`, (2) **template matching** (tokenising the URL against each `RoutePattern`), (3) **constraint evaluation** (`IRouteConstraint.Match`), (4) **MatcherPolicy** filtering (HTTP method, host, etc.), and (5) **endpoint selection** by precedence (`EndpointSelector`). If multiple endpoints share the same precedence and both match, an **ambiguous match** occurs (detected at build time by **ASP0022** or at runtime by `AmbiguousMatchException`). Diagnostic logging (`Microsoft.AspNetCore.Routing`) and `DiagnosticListener` events expose each phase. |
| **Beginner-Friendly Explanation** | Route matching is **how the framework decides which code runs for a URL**. When you visit `/products/5`, the router holds that URL against every route template. If a template fits, it checks constraints (Is `5` an integer?). If multiple templates fit, it picks the most specific one. If two fit equally well, it’s confused (ambiguous match). Route debugging helps you see each step of this process. |

### Purposes (each begins with “To”)

- **To** understand why a specific URL matched one endpoint instead of another.
- **To** diagnose 404 errors caused by missing routes or failed constraints.
- **To** identify ambiguous matches where two routes compete for the same URL.
- **To** detect route conflicts at **build time** via ASP0022.
- **To** analyse matching performance issues (large route tables, DFA node explosion).
- **To** verify that HTTP method, host, and other matcher policies are applied correctly.

### Syntax Rules and Structure

#### Diagnostic Channels for Route Matching

| Channel | What It Reveals | How to Enable |
|---------|-----------------|---------------|
| **Debug logging** | Endpoint selected, constraint results, candidate elimination. | `Logging:LogLevel:Microsoft` = `Debug` |
| **Trace logging** (`Microsoft.AspNetCore.Routing`) | Detailed `LinkGenerator` and matching traces. | Set category to `Trace` |
| **`DiagnosticListener`** | `Microsoft.AspNetCore.Routing.EndpointMatched` event. | Subscribe via `DiagnosticListener.AllListeners` |
| **ASP0022** | Route conflict detected at build time. | Enabled by default in .NET SDK analyzers |
| **`HttpContext.GetEndpoint()`** | The endpoint selected for the current request. | Access in middleware or filters |

#### Syntax Rules

- **Log level granularity** – Use `Debug` for general matching, `Trace` for `LinkGenerator` and low-level matching details.
- **ASP0022** – Treats ambiguous route matches as build-time warnings/errors; resolve by changing the route pattern, HTTP method, or constraints.
- **`GetEndpoint()`** – Returns `null` if no endpoint matched; useful for custom middleware diagnostics.
- **Performance** – Large route tables cause **DFA node explosion** and slow startup; apply constraints to reduce ambiguity and move parameters to later segments.

#### Constraints and Limitations

- **Logging volume** – `Trace` logging is extremely verbose; enable only for short diagnostic sessions.
- **ASP0022 scope** – Detects **statically determinable** conflicts; some runtime-dependent conflicts may still occur.
- **Ambiguous matches at runtime** – If ASP0022 does not catch a conflict (e.g., dynamic route registration), `AmbiguousMatchException` is thrown at request time.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Inspecting the Matched Endpoint in Middleware

```csharp
app.Use(async (context, next) =>
{
    var endpoint = context.GetEndpoint();
    if (endpoint != null)
    {
        var routePattern = (endpoint as RouteEndpoint)?.RoutePattern.RawText;
        var displayName = endpoint.DisplayName;
        Console.WriteLine($"Matched endpoint: {displayName}");
        Console.WriteLine($"Route pattern: {routePattern}");
    }
    else
    {
        Console.WriteLine("No endpoint matched.");
    }
    await next();
});
```

**Step-by-step explanation:**

1. `context.GetEndpoint()` returns the endpoint selected by the routing middleware.
2. If an endpoint is matched, `RouteEndpoint.RoutePattern.RawText` provides the template.
3. `endpoint.DisplayName` provides a human-readable name.
4. If no endpoint matched, the middleware logs that fact.

**Expected Output:**

```
Matched endpoint: MyApp.Controllers.ProductsController.GetById (MyApp)
Route pattern: api/Products/{id:int}
```

**Why this output?** `GetEndpoint()` exposes the routing middleware’s selection result for the current request.

---

#### Example 2: Subscribing to `DiagnosticListener` Events

```csharp
var listener = new DiagnosticListener("Microsoft.AspNetCore.Routing");
listener.Subscribe(new Observer());

class Observer : IObserver<KeyValuePair<string, object>>
{
    public void OnNext(KeyValuePair<string, object> pair)
    {
        if (pair.Key == "Microsoft.AspNetCore.Routing.EndpointMatched")
        {
            var endpoint = pair.Value as Endpoint;
            Console.WriteLine($"Endpoint matched: {endpoint?.DisplayName}");
        }
    }
    // OnCompleted / OnError omitted for brevity
}
```

**Step-by-step explanation:**

1. A `DiagnosticListener` is created for the routing source.
2. The observer listens for the `EndpointMatched` event.
3. When the event fires, the endpoint is logged.

**Expected Output:** `Endpoint matched: MyApp.Controllers.ProductsController.GetById (MyApp)`

**Why this output?** The routing middleware emits diagnostic events that can be observed without changing application code.

---

#### Example 3: Detecting Route Conflicts with ASP0022

```csharp
// ASP0022 will flag this conflict at build time:
[HttpGet("products/{id:int}")]
public IActionResult GetById(int id) { ... }

[HttpGet("products/{slug}")]
public IActionResult GetBySlug(string slug) { ... }
```

**Step-by-step explanation:**

1. Both routes can match `/products/5` (slug = `"5"`).
2. ASP0022 detects the potential conflict and emits a diagnostic.
3. The fix is to differentiate the routes (e.g., add a literal segment or use different HTTP methods).

**Expected Output:** Build warning/error ASP0022: “Route conflict detected between route handlers”.

**Why this output?** ASP0022 statically analyses route templates for overlaps that could cause ambiguous matches.

### Real-World Cases

- **404 on `/api/products/5`:** Debug logs reveal that the `int` constraint failed (e.g., the segment was `"five"`), so the route did not match.
- **Wrong action invoked:** `GetEndpoint()` middleware shows that `/products/list` matched `GetById(id = "list")` instead of `List()`, indicating a precedence issue.
- **Startup performance degradation:** DFA node analysis reveals thousands of `DfaNode` instances; applying constraints and simplifying templates reduces the route table complexity.

### References

- Routing in ASP.NET Core: Debug diagnostics – https://learn.microsoft.com/en-us/aspnet/core/fundamentals/routing#debug-diagnostics
- Routing in ASP.NET Core: Troubleshoot URL generation – https://learn.microsoft.com/en-us/aspnet/core/fundamentals/routing#troubleshoot-url-generation
- ASP0022: Route conflict detected between route handlers – https://github.com/dotnet/aspnetcore/blob/main/src/Http/Routing/src/Diagnostics/RouteConflictDiagnostic.cs
- `HttpContext.GetEndpoint()` – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.http.endpointhttpcontextextensions.getendpoint
- `DiagnosticListener` in ASP.NET Core – https://learn.microsoft.com/en-us/dotnet/api/system.diagnostics.diagnosticlistener

---

## 4. Route Debugging Best Practices

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Route debugging best practices are guidelines for using diagnostic tools safely, efficiently, and effectively, while avoiding security risks, performance degradation, and misleading conclusions. |
| **Technical Definition** | Best practices encompass environment isolation (Development-only exposure), log-level management, selective use of route‑listing endpoints, middleware ordering for debug tools, resolving ASP0022 conflicts at build time, performance-aware route table analysis, and combining multiple diagnostic channels (logging + introspection + middleware) rather than relying on a single tool. |
| **Beginner-Friendly Explanation** | Best practices are the “rules of thumb” that keep debugging safe and productive. For example: **never** expose route lists in production, **always** start with debug logging before adding custom tools, and **fix route conflicts** as soon as the compiler warns you. |

### Purposes (each begins with “To”)

- **To** prevent sensitive routing information from being exposed in production.
- **To** avoid performance degradation caused by verbose logging or expensive route enumeration.
- **To** resolve route conflicts early via ASP0022 rather than discovering them at runtime.
- **To** use the **least invasive** diagnostic tool first (logging) before adding middleware or custom endpoints.
- **To** combine diagnostics for a complete picture (matching + generation + performance).
- **To** ensure debug tools do not interfere with exception handling or other middleware.

### Guideline Checklist

| Guideline | Rationale | Example |
|-----------|-----------|---------|
| **Enable debug tools only in Development** | Route lists and RouteDebugger reveal internal structure. | `if (app.Environment.IsDevelopment()) { app.UseRouteDebugger(); }` |
| **Start with debug logging** | No code changes required; may immediately reveal the issue. | Set `Microsoft.AspNetCore.Routing` to `Debug`. |
| **Use `Trace` for URL generation only** | `Trace` is extremely verbose; restrict to `LinkGenerator` troubleshooting. | Set only `Microsoft.AspNetCore.Routing` to `Trace`. |
| **Place RouteDebugger before `UseDeveloperExceptionPage`** | Otherwise the exception page will not function. | `app.UseRouteDebugger(); app.UseDeveloperExceptionPage();` |
| **Fix ASP0022 conflicts at build time** | Runtime ambiguous matches cause 500 errors; build-time detection is safer. | Change route pattern or constraints. |
| **Secure custom route-list endpoints** | Even in staging, route lists should require authentication. | Apply `[Authorize]` or restrict by IP. |
| **Apply constraints to reduce route table size** | Large route tables cause DFA node explosion and slow startup. | `{id:int}` on parameterised segments. |
| **Use `EndpointDataSource` instead of reflection** | Reflection-based route access is fragile and version-dependent. | Inject `IEnumerable<EndpointDataSource>`. |
| **Remove debug tools before deployment** | Debug endpoints increase attack surface. | Use conditional compilation or environment checks. |

#### Constraints and Limitations

- **ASP0022 is not exhaustive** – Dynamic route registration may bypass static analysis; runtime `AmbiguousMatchException` can still occur.
- **RouteDebugger is community-maintained** – Evaluate version compatibility and security before use.
- **Logging configuration should be in configuration files, not code** – ASP.NET Core guidance recommends specifying log levels in `appsettings.json` rather than code.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Development-Only Route Debugging Setup

```csharp
var app = builder.Build();

if (app.Environment.IsDevelopment())
{
    // ✅ RouteDebugger before exception page
    app.UseRouteDebugger();

    // ✅ Route list endpoint, development only
    app.MapGet("/debug/routes", (IEnumerable<EndpointDataSource> sources) =>
    {
        var routes = sources
            .SelectMany(s => s.Endpoints)
            .OfType<RouteEndpoint>()
            .Select(e => new
            {
                Pattern = e.RoutePattern.RawText,
                Methods = e.Metadata
                    .OfType<HttpMethodMetadata>()
                    .FirstOrDefault()?.HttpMethods
            });
        return Results.Json(routes);
    });

    app.UseDeveloperExceptionPage();
}

app.UseRouting();
app.MapControllers();
app.Run();
```

**Step-by-step explanation:**

1. All debug tools are wrapped in `IsDevelopment()`.
2. RouteDebugger is placed **before** `UseDeveloperExceptionPage()` to preserve exception handling.
3. The custom route list endpoint is also Development-only, avoiding production information disclosure.
4. In production, none of these tools are registered.

**Expected Output:** Debug tools are available in Development; production deployment exposes no route-debugging endpoints.

**Why this output?** Environment isolation prevents accidental exposure of sensitive routing information.

---

#### Example 2: Combining Logging and `EndpointDataSource` for 404 Diagnosis

```csharp
// 1. Enable debug logging in appsettings.Development.json
// { "Logging": { "LogLevel": { "Microsoft.AspNetCore.Routing": "Debug" } } }

// 2. Add a diagnostic endpoint
app.MapGet("/debug/why-404", (string url, IEnumerable<EndpointDataSource> sources) =>
{
    var matching = sources
        .SelectMany(s => s.Endpoints)
        .OfType<RouteEndpoint>()
        .Where(e => e.RoutePattern.RawText != null)
        .Select(e => new
        {
            Pattern = e.RoutePattern.RawText,
            Score = e.RoutePattern.InboundPrecedence // Precedence indicator
        })
        .OrderByDescending(e => e.Score);

    return Results.Json(new
    {
        RequestedUrl = url,
        AllRoutes = matching,
        Hint = "Check debug logs for constraint failures and candidate elimination."
    });
});
```

**Step-by-step explanation:**

1. Debug logging reveals **which phase** failed (template, constraint, policy).
2. The diagnostic endpoint lists all routes with their **inbound precedence**, helping identify which routes are most specific.
3. The combination provides a complete picture: logs for the matching process, route list for context.

**Expected Output:** A JSON object containing the requested URL, all routes ordered by precedence, and a hint to check logs.

**Why this output?** Logging and introspection are complementary; together they answer both “what happened?” and “what routes exist?”.

---

#### Example 3: Resolving an ASP0022 Conflict

```csharp
// ❌ Conflicting routes (ASP0022):
// [HttpGet("products/{id:int}")]
// [HttpGet("products/{slug}")]

// ✅ Resolved: Differentiate by literal segment
[HttpGet("products/by-id/{id:int}")]
public IActionResult GetById(int id) { ... }

[HttpGet("products/by-slug/{slug}")]
public IActionResult GetBySlug(string slug) { ... }

// ✅ Alternative: Differentiate by HTTP method
[HttpGet("products/{slug}")]
public IActionResult GetBySlug(string slug) { ... }

[HttpHead("products/{slug}")]
public IActionResult HeadBySlug(string slug) { ... }
```

**Step-by-step explanation:**

1. The original routes overlap for numeric segments.
2. ASP0022 flags the conflict at build time.
3. Resolution options: add literal segments (`by-id`, `by-slug`) or use different HTTP methods.

**Expected Output:** ASP0022 warning disappears; routes are unambiguous.

**Why this output?** ASP0022 enforces deterministic route matching by detecting statically identifiable conflicts.

### Real-World Cases

- **Production security audit:** Ensuring `UseRouteDebugger` and custom `/debug/routes` endpoints are **absent** from production deployments.
- **CI/CD pipeline:** Treating ASP0022 as an **error** in Release builds to prevent ambiguous routes from reaching production.
- **Performance tuning:** Using `EndpointDataSource` to count routes and identify opportunities for constraints or template simplification.

### References

- Routing in ASP.NET Core: Routing best practices – https://learn.microsoft.com/en-us/aspnet/core/fundamentals/routing#routing-best-practices
- Routing in ASP.NET Core: Debug diagnostics – https://learn.microsoft.com/en-us/aspnet/core/fundamentals/routing#debug-diagnostics
- ASP0022: Route conflict detected – https://github.com/dotnet/aspnetcore/blob/main/src/Http/Routing/src/Diagnostics/RouteConflictDiagnostic.cs
- ASP.NET Core RouteDebugger Middleware (usage and security note) – https://github.com/ediwang/edi.routedebugger
- Logging in .NET: Configuration – https://learn.microsoft.com/en-us/dotnet/core/extensions/logging

---

## Summary Table: Route Debugging at a Glance

| Concept | Primary Purpose | Key Mechanism | When to Use |
|---------|----------------|---------------|-------------|
| **Route Debugging** | Diagnose routing issues | Logging, `EndpointDataSource`, RouteDebugger, ASP0022 | Any unexpected routing behaviour |
| **Route List** | Enumerate all registered endpoints | `IEnumerable<EndpointDataSource>`, RoutesList, RouteDebugger | 404 investigation, audit, documentation |
| **Route Matching** | Understand endpoint selection | Debug/Trace logs, `GetEndpoint()`, `DiagnosticListener`, ASP0022 | Wrong endpoint, ambiguous match, constraint failure |
| **Best Practices** | Safe and effective debugging | Environment isolation, log-level management, build-time conflict resolution | All debugging activities |

---

## References (Full List)

- Routing in ASP.NET Core – https://learn.microsoft.com/en-us/aspnet/core/fundamentals/routing
- Routing in ASP.NET Core: Debug diagnostics – https://learn.microsoft.com/en-us/aspnet/core/fundamentals/routing#debug-diagnostics
- Routing in ASP.NET Core: Routing best practices – https://learn.microsoft.com/en-us/aspnet/core/fundamentals/routing#routing-best-practices
- Routing in ASP.NET Core: Troubleshoot URL generation – https://learn.microsoft.com/en-us/aspnet/core/fundamentals/routing#troubleshoot-url-generation
- `EndpointDataSource` API – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.routing.endpointdatasource
- `HttpContext.GetEndpoint()` – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.http.endpointhttpcontextextensions.getendpoint
- ASP.NET Core RouteDebugger Middleware – https://github.com/ediwang/edi.routedebugger
- RoutesList NuGet Package – https://www.nuget.org/packages/RoutesList/
- Microsoft Q&A: Routing in ASP.NET Core (route table dumper example) – https://learn.microsoft.com/en-us/answers/questions/1923251/routing-in-asp-net-core
- Stack Overflow: How to get a list of all routes in ASP.NET Core? – https://stackoverflow.com/questions/28435734/how-to-get-a-list-of-all-routes-in-asp-net-core
- ASP0022: Route conflict detected between route handlers – https://github.com/dotnet/aspnetcore/blob/main/src/Http/Routing/src/Diagnostics/RouteConflictDiagnostic.cs
- Logging in .NET: Configuration – https://learn.microsoft.com/en-us/dotnet/core/extensions/logging
- `DiagnosticListener` API – https://learn.microsoft.com/en-us/dotnet/api/system.diagnostics.diagnosticlistener