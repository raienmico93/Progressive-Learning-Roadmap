# Action Methods in ASP.NET Core MVC: A Comprehensive Cheat Sheet

> **Version Scope & Compatibility Note**
> This cheat sheet targets **ASP.NET Core MVC** (3.0 and later, including .NET 6/7/8/9/10/11). Action return types are stable across versions. `ActionResult<T>` was introduced in ASP.NET Core 2.1 and is recommended for web APIs. The distinction between `IActionResult` and `ActionResult<T>` matters most for **API controllers** (`ControllerBase`); for **MVC view controllers** (`Controller`), `IActionResult` remains the standard. All concrete result types (`ViewResult`, `JsonResult`, etc.) inherit from `ActionResult` and implement `IActionResult`. Helper methods on `Controller`/`ControllerBase` (e.g., `View()`, `Ok()`, `NotFound()`) construct these results conveniently.

---

## Topic Overview

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Action methods are **public methods on a controller** that handle HTTP requests, execute application logic, and return a response in the form of an **action result** (view, JSON, status code, redirect, file, etc.). |
| **Technical Definition** | An action method is a public, non-static method on a controller class. It is exposed as an HTTP endpoint via routing and typically returns an `IActionResult`, an `ActionResult<T>`, or a specific POCO type. The framework's result-execution pipeline inspects the returned object: if it implements `IActionResult`, its `ExecuteResultAsync` method runs; if it is a POCO, it is serialized to JSON in API controllers or wrapped in a `ViewResult` in MVC controllers. |
| **Beginner-Friendly Explanation** | Action methods are the **specific tasks** a controller can perform. If `ProductsController` is the manager of products, then `Index()`, `Details(id)`, `Create(dto)`, and `Delete(id)` are its actions. Each action handles one URL/HTTP-method combination and decides what to send back—an HTML page, JSON data, a redirect, or a status code like "404 Not Found." |

### Key Characteristics

- **Public and discoverable** – Only `public`, non-static methods become actions.
- **Model binding–driven** – Parameters are populated automatically from route values, query strings, form data, and JSON bodies.
- **Return-type flexibility** – `IActionResult` (multiple outcomes), `ActionResult<T>` (typed multi-outcome APIs), or a concrete type (single outcome).
- **Async support** – `Task<IActionResult>` and `Task<ActionResult<T>>` are fully supported.
- **Result execution separated** – Action methods **return** results; the framework **executes** them (rendering views, serializing JSON, etc.).
- **Testable** – Actions can be unit-tested by inspecting the returned result object.

### Prerequisites

- **.NET SDK** (6.0+ recommended; examples use .NET 8/10).
- **ASP.NET Core MVC** services registered (`AddControllers()` or `AddControllersWithViews()`).
- Routing configured (`MapControllers()` or `MapControllerRoute()`).
- Basic understanding of **controllers**, **routing**, and **model binding**.

### Related Programming Areas

- **Controllers** – Action methods live on controllers.
- **Model Binding** – Populates action parameters.
- **Filters** – Run before/after actions.
- **Razor Views** – Rendered by `ViewResult`.
- **JSON Serialization** – Used by `JsonResult` and POCO return types.
- **HTTP Status Codes** – Encoded by `StatusCodeResult` and friends.

### Core Concepts / Features

1. **Action Methods (General)** – What they are and how they work.
2. **`IActionResult`** – The common interface for all action results.
3. **`ActionResult<T>`** – Typed multi-outcome result for APIs.
4. **`ViewResult`** – Renders a Razor view.
5. **`PartialViewResult`** – Renders a partial view.
6. **`JsonResult`** – Returns JSON.
7. **`RedirectResult`** – Redirects to a URL.
8. **`RedirectToActionResult`** – Redirects to an action.
9. **`RedirectToRouteResult`** – Redirects to a named route.
10. **`FileResult`** – Returns file content (abstract; concrete variants: `FileContentResult`, `FileStreamResult`, `PhysicalFileResult`, `VirtualFileResult`).
11. **`ContentResult`** – Returns plain text.
12. **`StatusCodeResult`** – Returns a raw HTTP status code.
13. **`NotFoundResult`** – Returns 404.
14. **`BadRequestResult`** – Returns 400.
15. **`OkResult`** – Returns 200 with no body.
16. **`CreatedResult`** – Returns 201 with a body and `Location` header.
17. **`NoContentResult`** – Returns 204.
18. **Action Method Best Practices** – Guidelines for clean, testable actions.

---

## 1. Action Methods (General)

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | An action method is a public method on a controller that responds to an HTTP request and returns a result that becomes the HTTP response. |
| **Technical Definition** | Action methods are discovered by the MVC application model from public, non-static methods on controllers (excluding methods decorated with `[NonAction]`). They participate in the action invocation pipeline: authorization filters → resource filters → model binding → action filters → **action execution** → action filters → result filters → result execution. The return value is processed by `IActionResultExecutor` (for `IActionResult`) or by the output formatter (for POCOs). |
| **Beginner-Friendly Explanation** | Action methods are the **tasks** your controller performs. When a user hits a URL, one action method runs. It receives input (from the URL or body), calls the right business logic, and returns a response—an HTML page, JSON, a redirect, or a status code. |

### Purposes (each begins with “To”)

- **To** handle a specific HTTP request (URL + HTTP method) and execute the corresponding application logic.
- **To** receive input via model binding and validate it (often with filters or `ModelState`).
- **To** delegate business logic to services and translate the outcome into an HTTP response.
- **To** return an appropriate result type that matches the semantic outcome (view, JSON, redirect, status code).
- **To** remain small and focused, keeping the controller thin and testable.

### Syntax Rules and Structure

#### General Syntax

```csharp
public [async Task<]IActionResult|ActionResult<T>|T[>] ActionName([parameters])
{
    // ...
    return /* result */;
}
```

#### Component Breakdown

| Component | Description |
|-----------|-------------|
| `public` | Required for the method to be an action. |
| Return type | `IActionResult`, `ActionResult<T>`, or a concrete type. |
| Method name | Used by routing to match the `action` route value. |
| Parameters | Populated by model binding. |
| Return statement | Produces the result executed by the framework. |

#### Syntax Rules

- **Public only** – Private/protected methods are not actions; use `[NonAction]` to exclude a public method explicitly.
- **No `ref`/`out` parameters** – Model binding cannot populate them.
- **Overloads not permitted** – Two actions with the same name but different parameters cause an `AmbiguousMatchException`; use different names or explicit routing.
- **Async** – Use `async Task<IActionResult>` (or `Task<ActionResult<T>>`) for I/O-bound work.

#### Constraints and Limitations

- **Returning `void`** – Not recommended; the client receives an empty 200 with no meaningful content.
- **Throwing exceptions** – Prefer returning appropriate results or letting exception middleware handle them.
- **God actions** – Overly long actions indicate that logic should move to services.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: MVC Action Returning a View

```csharp
public IActionResult Index()
{
    var products = _productService.GetAll();
    return View(products); // ViewResult
}
```

**Step-by-step explanation:**

1. `Index` is public and parameterless.
2. It fetches data from the service.
3. It returns `View(products)`, a `ViewResult` that renders `Views/{Controller}/Index.cshtml`.

**Expected Output:** HTML page listing products.

**Why this output?** The `ViewResult` invokes the Razor view engine to render the matched view with the supplied model.

---

#### Example 2: API Action Returning JSON via POCO

```csharp
[HttpGet("{id:int}")]
public ProductDto GetById(int id) => _service.GetById(id);
```

**Step-by-step explanation:**

1. The method returns `ProductDto` (a POCO).
2. In an `[ApiController]`, the framework serializes the POCO to JSON with status 200.

**Expected Output:** `200 OK` with JSON body `{ "id": 5, "name": "...", ... }`.

**Why this output?** API controllers use output formatters (System.Text.Json by default) for POCO returns.

### Real-World Cases

- **CRUD actions:** `Index`, `Details`, `Create`, `Edit`, `Delete`.
- **Search:** `Search(string q, int page)`.
- **Health/diagnostics:** `Ping()` returning `Ok()`.

### References

- Handle requests with controllers in ASP.NET Core MVC – https://learn.microsoft.com/en-us/aspnet/core/mvc/controllers/actions

---

## 2. `IActionResult`

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | `IActionResult` is the **interface implemented by all action results**; returning it allows an action to produce different response types based on runtime conditions. |
| **Technical Definition** | `Microsoft.AspNetCore.Mvc.IActionResult` defines a single method, `Task ExecuteResultAsync(ActionContext context)`. Every concrete result (e.g., `ViewResult`, `JsonResult`, `NotFoundResult`) implements it. The framework invokes `ExecuteResultAsync` on the returned instance, which writes the HTTP response. |
| **Beginner-Friendly Explanation** | `IActionResult` is a **wrapper that can hold any kind of response**. When an action might return a view *or* a 404 *or* a redirect, `IActionResult` is the flexible return type that allows any of them. |

### Purposes (each begins with “To”)

- **To** allow an action to return different result types depending on runtime conditions (success, not found, redirect).
- **To** provide a uniform abstraction for all response-producing classes in MVC.
- **To** enable unit testing by inspecting the concrete result type (e.g., `Assert.IsType<NotFoundResult>(result)`).
- **To** support both MVC views and API responses in the same controller.

### Syntax Rules and Structure

#### General Syntax

```csharp
public IActionResult ActionName()
{
    if (/* not found */) return NotFound();      // NotFoundResult
    if (/* invalid */) return BadRequest();      // BadRequestResult
    return View();                               // ViewResult
}
```

#### Component Breakdown

| Component | Description |
|-----------|-------------|
| Return type `IActionResult` | Interface implemented by all result types. |
| Helper methods | `View()`, `Ok()`, `NotFound()`, `BadRequest()`, `RedirectToAction()`, etc. |

#### Syntax Rules

- **Use when multiple result types are possible**.
- **Not suitable for API controllers requiring typed responses** – prefer `ActionResult<T>`.

#### Constraints and Limitations

- **No compile-time type information** – for APIs, `ActionResult<T>` provides stronger typing.
- **Returning `null`** – not a valid result; will cause a runtime error.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Multi-Outcome Action

```csharp
public IActionResult Details(int id)
{
    var product = _service.GetById(id);
    if (product == null) return NotFound();
    return View(product);
}
```

**Expected Output:**

| Scenario | Result |
|----------|--------|
| Product exists | `ViewResult` (HTML page) |
| Product missing | `NotFoundResult` (404) |

**Why this output?** `IActionResult` accommodates both concrete types, selected by runtime conditions.

### Real-World Cases

- **MVC CRUD controllers** – `Details`, `Edit` actions return views or 404s.
- **Mixed controllers** – endpoints that return views for browsers and JSON for API clients.

### References

- `IActionResult` interface – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.iactionresult

---

## 3. `ActionResult<T>`

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | `ActionResult<T>` is a **generic wrapper** that allows an action to return either a concrete value of type `T` (auto-serialized to JSON) or an `IActionResult` (e.g., a status code result). |
| **Technical Definition** | `Microsoft.AspNetCore.Mvc.ActionResult<T>` has two implicit conversions: from `T` (wrap the value) and from `IActionResult` (wrap the result). When executed, the framework checks whether the wrapped value is an `IActionResult`; if so, it executes it; otherwise, the framework serializes `T` via output formatters (typically JSON). Recommended for web API actions per Microsoft guidance. |
| **Beginner-Friendly Explanation** | `ActionResult<T>` combines the flexibility of `IActionResult` with the type safety of a specific type. You can return a `ProductDto` directly (auto-JSON), or return `NotFound()` when the product is missing. It’s the best of both worlds for APIs. |

### Purposes (each begins with “To”)

- **To** provide **type safety** for API actions while still allowing multiple status codes.
- **To** simplify unit testing by allowing inspection of either the value or the result type.
- **To** integrate with OpenAPI/Swagger for accurate API documentation.
- **To** avoid the verbosity of wrapping every successful return in `Ok(...)`.

### Syntax Rules and Structure

#### General Syntax

```csharp
public ActionResult<ProductDto> GetById(int id)
{
    var product = _service.GetById(id);
    if (product == null) return NotFound();  // Implicit wrap of IActionResult
    return product;                           // Implicit wrap of T
}
```

#### Component Breakdown

| Return statement | Behaviour |
|------------------|-----------|
| `return product;` | Wraps `T`, serialized with 200 OK. |
| `return NotFound();` | Wraps `IActionResult` and executes it (404). |

#### Syntax Rules

- **Use in web API controllers** for typed responses.
- **No need to explicitly wrap success returns** in `Ok(...)`, though `Ok(product)` still works.
- **Documented by OpenAPI** – Swashbuckle reads the generic type for schema generation.

#### Constraints and Limitations

- **Not for MVC view controllers** – for view actions, use `IActionResult`.
- **Slightly less discoverable for beginners** – implicit conversions can confuse debugging.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Typed API Action with Two Outcomes

```csharp
[ApiController]
[Route("api/[controller]")]
public class ProductsController : ControllerBase
{
    [HttpGet("{id:int}")]
    public ActionResult<ProductDto> GetById(int id)
    {
        var product = _service.GetById(id);
        if (product == null) return NotFound();
        return product;
    }
}
```

**Expected Output:**

| Scenario | Response |
|----------|----------|
| Product 5 exists | 200 OK with JSON body |
| Product 5 missing | 404 Not Found |

**Why this output?** The implicit conversion wraps `product` as a successful body; `NotFound()` wraps the 404 result.

---

#### Example 2: Mixed with Error Details

```csharp
[HttpPost]
public ActionResult<ProductDto> Create(CreateProductDto dto)
{
    if (!ModelState.IsValid) return ValidationProblem(ModelState);
    var product = _service.Create(dto);
    return CreatedAtAction(nameof(GetById), new { id = product.Id }, product);
}
```

**Expected Output:** `201 Created` with the product body and a `Location` header, or `400` on validation failure.

**Why this output?** `ActionResult<T>` accepts any `IActionResult`, including `CreatedAtActionResult` and `ValidationProblemDetails`.

### Real-World Cases

- **REST CRUD endpoints** – `GET`, `POST`, `PUT`, `DELETE` actions with well-documented type contracts.
- **OpenAPI-driven APIs** – Swagger generation relies on the generic type for response schemas.

### References

- Controller action return types in ASP.NET Core web API – https://learn.microsoft.com/en-us/aspnet/core/web-api/action-return-types#actionresultt-type

---

## 4. `ViewResult`

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | `ViewResult` renders a **Razor view** (`.cshtml`) and writes the resulting HTML to the response. |
| **Technical Definition** | `Microsoft.AspNetCore.Mvc.ViewResult` selects a view (by name or convention), optionally passes a `ViewDataDictionary`, and executes the view via the `IViewEngine`. Constructed by `Controller.View(...)`. Execution involves finding the view (by search paths), running its Razor code, and writing HTML. |
| **Beginner-Friendly Explanation** | `ViewResult` is the **HTML page result**. When an action returns `View(model)`, the framework looks for the matching `.cshtml` file, renders it with the model, and sends the resulting HTML to the browser. |

### Purposes (each begins with “To”)

- **To** render an HTML page to the client using a Razor view.
- **To** pass a strongly typed model to the view for display.
- **To** allow the developer to override the view name (e.g., `View("Details")` from a `Details` action).
- **To** integrate with layouts, partials, and view components.

### Syntax Rules and Structure

#### General Syntax

```csharp
return View();                // Convention: Views/{Controller}/{Action}.cshtml
return View(model);           // Pass model
return View("ViewName");      // Explicit view name
return View("ViewName", model);
```

#### Component Breakdown

| Component | Description |
|-----------|-------------|
| `View()` | Renders the view matching the action name. |
| `View(model)` | Same, with a model. |
| `View("Name", model)` | Explicit view name (usually relative to the controller folder). |

#### Syntax Rules

- **View name defaults to the action name**.
- **View search order:** `/Views/{Controller}/{Action}.cshtml` → `/Views/Shared/{Action}.cshtml`.
- **Model must match `@model`** in the view; mismatches throw at runtime.

#### Constraints and Limitations

- **Not for APIs** – use `ControllerBase` + `ActionResult<T>` instead.
- **Model-binding mismatches** – Passing a model of a different type than the view declares causes errors.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Standard View

```csharp
public IActionResult Index()
{
    var products = _service.GetAll();
    return View(products);
}
```

**Expected Output:** Renders `Views/Products/Index.cshtml` with `@model IEnumerable<Product>`.

**Why this output?** The `ViewResult` resolves the view by convention (action name), passes the model, and renders.

---

#### Example 2: Explicit View Name

```csharp
public IActionResult Details(int id)
{
    var product = _service.GetById(id);
    if (product == null) return NotFound();
    return View("ProductDetails", product);
}
```

**Expected Output:** Renders `Views/Products/ProductDetails.cshtml` with the product.

**Why this output?** The explicit view name overrides the default `Details.cshtml` selection.

### Real-World Cases

- **CRUD pages** – list, detail, and form pages.
- **Reusable pages across actions** – `View("List", data)` from multiple actions.

### References

- Views in ASP.NET Core MVC – https://learn.microsoft.com/en-us/aspnet/core/mvc/views/overview
- `ViewResult` API – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.viewresult

---

## 5. `PartialViewResult`

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | `PartialViewResult` renders a **partial view**—a reusable Razor fragment **without a layout**—and writes the HTML to the response. |
| **Technical Definition** | `Microsoft.AspNetCore.Mvc.PartialViewResult` resolves a partial view (by convention), executes the Razor view engine **without applying `_ViewStart.cshtml`** (no layout), and writes the resulting HTML. Constructed by `Controller.PartialView(...)`. Often used for AJAX responses that inject HTML fragments into existing pages. |
| **Beginner-Friendly Explanation** | `PartialViewResult` is a **HTML fragment result**. Unlike a full page (with header, footer, layout), a partial view is just the piece of HTML you want to insert into an existing page—useful for dynamic sections loaded via AJAX. |

### Purposes (each begins with “To”)

- **To** return a **reusable HTML fragment** that can be embedded in multiple pages or fetched via AJAX.
- **To** avoid rendering the full layout when only a portion of the page is needed.
- **To** support incremental updates in client-side frameworks (jQuery, HTMX, etc.).
- **To** reduce page weight by returning only the relevant HTML.

### Syntax Rules and Structure

#### General Syntax

```csharp
return PartialView();                 // Convention: Views/{Controller}/_Name.cshtml or Shared/
return PartialView(model);
return PartialView("_PartialName");
return PartialView("_PartialName", model);
```

#### Syntax Rules

- **Partial views typically start with `_`** (convention, not requirement).
- **No layout applied** – `_ViewStart.cshtml` is skipped.
- **Model binding works the same** – `@model` in the partial should match the passed model.

#### Constraints and Limitations

- **Not for full pages** – use `ViewResult` for full pages with layouts.
- **CSP/security** – returning HTML snippets via AJAX requires careful XSS handling.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: AJAX Partial

```csharp
public IActionResult _ProductRow(Product p) => PartialView(p);
```

```cshtml
@* Views/Products/_ProductRow.cshtml *@
@model Product
<tr>
    <td>@Model.Name</td>
    <td>@Model.Price.ToString("C")</td>
</tr>
```

**Expected Output:** Returns an HTML `<tr>` fragment for the product.

**Why this output?** `PartialView` renders only the fragment (no layout), suitable for insertion into a table.

### Real-World Cases

- **AJAX pagination** – return partial rows for the next page.
- **Search results** – return HTML snippets rendered into a container.
- **Modal dialog content** – load only the dialog body HTML.

### References

- Partial views in ASP.NET Core – https://learn.microsoft.com/en-us/aspnet/core/mvc/views/partial
- `PartialViewResult` API – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.partialviewresult

---

## 6. `JsonResult`

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | `JsonResult` **serializes a value to JSON** and writes it to the response with a `Content-Type` of `application/json`. |
| **Technical Definition** | `Microsoft.AspNetCore.Mvc.JsonResult` holds a `Value`, `ContentType` (default `"application/json"`), and `StatusCode` (default 200). During execution, it uses the registered `IJsonHelper` (backed by System.Text.Json in modern ASP.NET Core) to serialize the value. Constructed by `Controller.Json(...)`. |
| **Beginner-Friendly Explanation** | `JsonResult` is the **JSON response result**. Return it when you want to send structured data (objects, arrays) to a client—often an AJAX request or API client—without rendering an HTML view. |

### Purposes (each begins with “To”)

- **To** return structured data to AJAX or API clients in JSON format.
- **To** customize the HTTP status code (e.g., 400) or content type.
- **To** enable MVC controllers (not just API controllers) to return JSON without switching to `ControllerBase`.
- **To** control serialization for edge cases (e.g., camelCase options via `JsonSerializerOptions`).

### Syntax Rules and Structure

#### General Syntax

```csharp
return Json(data);                                         // 200 OK, application/json
return Json(data, new JsonSerializerOptions { ... });      // Custom options
return new JsonResult(data) { StatusCode = 400 };          // Custom status
```

#### Component Breakdown

| Component | Description |
|-----------|-------------|
| `data` | Any object (POCO, anonymous type, collection). |
| `JsonSerializerOptions` | Controls naming policy, converters, etc. |
| `StatusCode` | Optional HTTP status code override. |

#### Syntax Rules

- **Response type is JSON, not a view** – intended for data, not HTML.
- **Async-safe** – returns `JsonResult`, which the framework executes.

#### Constraints and Limitations

- **In API controllers**, prefer returning POCOs or `ActionResult<T>` for automatic formatting and OpenAPI schema generation.
- **`JsonResult` bypasses content negotiation** – it always returns JSON, unlike POCO returns that respect `Accept` headers.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Simple JSON Response

```csharp
public IActionResult GetCount()
{
    return Json(new { count = _service.Count() });
}
```

**Expected Output:** `200 OK`, `Content-Type: application/json`, body `{ "count": 42 }`.

**Why this output?** `JsonResult` serializes the anonymous object and sets the JSON content type.

---

#### Example 2: JSON with Custom Status

```csharp
public IActionResult Error()
{
    return new JsonResult(new { error = "Invalid input" }) { StatusCode = 400 };
}
```

**Expected Output:** `400 Bad Request`, JSON body `{ "error": "Invalid input" }`.

**Why this output?** The `StatusCode` property overrides the default 200.

### Real-World Cases

- **AJAX endpoints** – return JSON for client-side rendering.
- **Autocomplete** – return JSON for suggestions.
- **Health checks** – return `{ "status": "ok" }`.

### References

- `JsonResult` API – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.jsonresult
- Format response data in ASP.NET Core web API – https://learn.microsoft.com/en-us/aspnet/core/web-api/advanced/formatting

---

## 7. `RedirectResult`

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | `RedirectResult` issues an HTTP redirect to a **URL** (typically 302 Found, configurable to 301, 307, or 308). |
| **Technical Definition** | `Microsoft.AspNetCore.Mvc.RedirectResult` sets the `Location` header to the supplied `Url` and the response status to the given `Permanent` flag (302 default or 301 permanent). Constructed by `Controller.Redirect(...)`. The client reissues the request to the new location. |
| **Beginner-Friendly Explanation** | `RedirectResult` tells the **browser** to go to a different URL. Use it when you want to send the user to an external site or a full URL that isn’t easily generated by route values. |

### Purposes (each begins with “To”)

- **To** redirect the client to a **specific URL** (external or internal).
- **To** perform permanent (301) or temporary (302) redirects as appropriate.
- **To** redirect after form submissions (Post/Redirect/Get pattern) when the target is a URL rather than an action.

### Syntax Rules and Structure

#### General Syntax

```csharp
return Redirect("https://example.com");                    // 302 Found
return RedirectPermanent("https://example.com");           // 301 Moved Permanently
return new RedirectResult("https://example.com", permanent: false);
```

#### Component Breakdown

| Helper | Status | When to Use |
|--------|--------|-------------|
| `Redirect(url)` | 302 | Temporary redirect. |
| `RedirectPermanent(url)` | 301 | Permanent redirect (SEO-friendly). |
| `LocalRedirect(url)` | 302 | Enforces local-only URL (safety). |

#### Constraints and Limitations

- **Prefer `RedirectToAction`/`RedirectToRoute`** – they generate URLs from routes and survive URL changes.
- **Open redirect risk** – never redirect to user-supplied URLs without validation; use `LocalRedirect`.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: External Redirect

```csharp
public IActionResult Docs()
{
    return Redirect("https://learn.microsoft.com/aspnet/core");
}
```

**Expected Output:** `302 Found` with `Location: https://learn.microsoft.com/aspnet/core`.

**Why this output?** The client is instructed to re-issue the request to the external URL.

---

#### Example 2: Local Redirect (Safe)

```csharp
public IActionResult GoTo(string returnUrl)
{
    if (Url.IsLocalUrl(returnUrl))
        return LocalRedirect(returnUrl);
    return RedirectToAction("Index", "Home");
}
```

**Expected Output:** Local URLs are allowed; external URLs fall back to home.

**Why this output?** `LocalRedirect` throws if the URL is not local, preventing open redirects.

### Real-World Cases

- **OAuth callbacks** – redirect to provider URLs.
- **Legacy URL migration** – permanent redirects to new URLs.
- **Post-login** – redirect to an external `returnUrl` (with validation).

### References

- `RedirectResult` API – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.redirectresult
- Prevent open redirects in ASP.NET Core – https://learn.microsoft.com/en-us/aspnet/core/security/preventing-open-redirects

---

## 8. `RedirectToActionResult`

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | `RedirectToActionResult` issues a redirect to a **controller action** by generating the URL from route values. |
| **Technical Definition** | `Microsoft.AspNetCore.Mvc.RedirectToActionResult` uses `IUrlHelper.Action(...)` (via `LinkGenerator`) to compute the target URL from `ActionName`, `ControllerName`, and route values, then sets the `Location` header and 302 (or 301) status. Constructed by `Controller.RedirectToAction(...)`. |
| **Beginner-Friendly Explanation** | `RedirectToActionResult` tells the browser to go to **another action in your app**, identified by controller and action names—not a raw URL. This makes redirects robust if URL templates change. |

### Purposes (each begins with “To”)

- **To** redirect to a known action in the app **by name**, avoiding hard-coded URLs.
- **To** preserve URL generation rules (route templates, areas) as the URL is computed.
- **To** implement the Post/Redirect/Get pattern after form submissions.
- **To** redirect after successful operations (create, edit, delete) to a list or detail page.

### Syntax Rules and Structure

#### General Syntax

```csharp
return RedirectToAction("Index");                                // Same controller
return RedirectToAction("Details", new { id = 5 });              // Same controller, with route values
return RedirectToAction("Index", "Products");                    // Different controller
return RedirectToAction("Details", "Products", new { id = 5 });  // Different controller + route values
```

#### Component Breakdown

| Parameter | Description |
|-----------|-------------|
| `actionName` | Target action method name. |
| `controllerName` | (Optional) target controller; defaults to current. |
| `routeValues` | Route values for URL generation. |

#### Constraints and Limitations

- **Action must exist** – if URL generation fails, the `Location` may be empty; guard with tests.
- **Ambient values** – current request’s route values (like `area`) can influence generated URLs.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: PRG After Create

```csharp
[HttpPost]
public IActionResult Create(ProductDto dto)
{
    if (!ModelState.IsValid) return View(dto);
    var created = _service.Create(dto);
    return RedirectToAction(nameof(Details), new { id = created.Id });
}
```

**Expected Output:** `302 Found` with `Location: /Products/Details/42`.

**Why this output?** `RedirectToAction` generates the target URL using route values and the named action.

### Real-World Cases

- **PRG pattern** – POST → redirect → GET to prevent duplicate form submissions.
- **Login redirect** – after sign-in, redirect to the intended action.

### References

- `RedirectToActionResult` API – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.redirecttoactionresult
- URL generation in ASP.NET Core – https://learn.microsoft.com/en-us/aspnet/core/fundamentals/routing#url-generation

---

## 9. `RedirectToRouteResult`

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | `RedirectToRouteResult` issues a redirect to a **named route**, generating the URL using the route’s template and supplied route values. |
| **Technical Definition** | `Microsoft.AspNetCore.Mvc.RedirectToRouteResult` uses `LinkGenerator.GetPathByRouteName` to compute the target URL from a route name and route values, then sets the `Location` header and 302 (or 301) status. Constructed by `Controller.RedirectToRoute(...)`. |
| **Beginner-Friendly Explanation** | `RedirectToRouteResult` is like `RedirectToAction` but keyed by **route name** instead of controller/action. Use it when your route is defined explicitly and named, ensuring redirects track route template changes. |

### Purposes (each begins with “To”)

- **To** redirect to a **named route** (e.g., `"blog"`, `"default"`) whose URL is generated from the route template.
- **To** target URLs whose structure doesn’t map cleanly to controller/action pairs (e.g., catch-all routes).
- **To** decouple redirects from controller/action names, using the stable route name as the reference.

### Syntax Rules and Structure

#### General Syntax

```csharp
return RedirectToRoute("routeName");
return RedirectToRoute("routeName", new { id = 5 });
return RedirectToRoute(new { controller = "Home", action = "Index" });  // value-based
```

#### Component Breakdown

| Parameter | Description |
|-----------|-------------|
| `routeName` | The name assigned to a route (`MapControllerRoute` `name` or `[Route(Name=...)]`). |
| `routeValues` | Values used to expand the route template. |

#### Constraints and Limitations

- **Route name must exist** – unknown names cause `InvalidOperationException` or `null` URLs.
- **Harder to debug** – route names are strings; typos are not caught by the compiler (consider constants).

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Redirect to a Named Route

```csharp
// Program.cs
app.MapControllerRoute(name: "blog", pattern: "blog/{*slug}",
    defaults: new { controller = "Blog", action = "Article" });

// Action
public IActionResult GoToPost(string slug) => RedirectToRoute("blog", new { slug });
```

**Expected Output:** `302 Found` with `Location: /blog/my-first-post`.

**Why this output?** The named route's template is expanded with `slug` to produce the URL.

### Real-World Cases

- **Redirect to blog posts** – route name `"blog"` handles all post URLs.
- **Multi-tenant redirects** – route name captures the tenant slug in the URL.

### References

- `RedirectToRouteResult` API – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.redirecttorouteresult
- URL generation by route name – https://learn.microsoft.com/en-us/aspnet/core/fundamentals/routing#generate-urls-by-route-name

---

## 10. `FileResult`

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | `FileResult` is the **abstract base class** for file responses, with four concrete subclasses: `FileContentResult`, `FileStreamResult`, `PhysicalFileResult`, and `VirtualFileResult`. |
| **Technical Definition** | `Microsoft.AspNetCore.Mvc.FileResult` (abstract) is implemented by: `FileContentResult` (byte array), `FileStreamResult` (stream), `PhysicalFileResult` (absolute file path), `VirtualFileResult` (path under `wwwroot`). Each sets `Content-Type`, `FileDownloadName` (for `Content-Disposition: attachment`), and optionally supports range requests. Constructed by `Controller.File(...)` overloads. |
| **Beginner-Friendly Explanation** | `FileResult` returns **a file to the client**—images, PDFs, CSVs, or any downloadable content. You can hand over bytes, a stream, or a path, and the framework streams it with the right content type and optionally as an attachment download. |

### Purposes (each begins with “To”)

- **To** send file content (PDF, image, CSV, ZIP) to the client.
- **To** control `Content-Type` and `Content-Disposition` (inline vs. attachment).
- **To** stream large files efficiently (via `FileStreamResult`) without loading them fully into memory.
- **To** support range requests for media playback and resumable downloads.

### Syntax Rules and Structure

#### General Syntax (via helper)

```csharp
return File(bytes, "application/pdf");                                  // FileContentResult
return File(bytes, "application/pdf", "report.pdf");                    // Download name
return File(stream, "application/octet-stream", "data.bin");            // FileStreamResult
return PhysicalFile(@"C:\path\file.pdf", "application/pdf");            // PhysicalFileResult
return File("~/files/report.pdf", "application/pdf");                   // VirtualFileResult
```

#### Component Breakdown

| Parameter | Description |
|-----------|-------------|
| Content | `byte[]`, `Stream`, or `string` path. |
| `contentType` | MIME type (e.g., `"application/pdf"`). |
| `fileDownloadName` | (Optional) `Content-Disposition` attachment filename. |
| `enableRangeProcessing` | (Optional) enables HTTP range requests. |

#### Constraints and Limitations

- **Large files** – prefer `FileStreamResult` (or `PhysicalFile`) over `byte[]` to avoid memory pressure.
- **Security** – sanitise file paths; never allow user-controlled paths directly (`Path.GetFullPath` + directory check).

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Returning an Image Inline

```csharp
public IActionResult Logo()
{
    var bytes = System.IO.File.ReadAllBytes("wwwroot/images/logo.png");
    return File(bytes, "image/png"); // Inline render
}
```

**Expected Output:** `200 OK`, `Content-Type: image/png`, body = PNG bytes.

**Why this output?** With no `fileDownloadName`, the browser renders the image inline.

---

#### Example 2: Downloading a CSV

```csharp
public IActionResult DownloadCsv()
{
    var stream = _csvService.GenerateCsv();
    return File(stream, "text/csv", "orders.csv");
}
```

**Expected Output:** `200 OK`, `Content-Disposition: attachment; filename=orders.csv`.

**Why this output?** The `fileDownloadName` triggers an attachment download.

### Real-World Cases

- **Invoice downloads** – PDF generation and return.
- **CSV exports** – bulk data download for admins.
- **Image serving** – thumbnail or avatar responses.

### References

- `FileResult` API – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.fileresult
- File responses in ASP.NET Core – https://learn.microsoft.com/en-us/aspnet/core/mvc/controllers/actions

---

## 11. `ContentResult`

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | `ContentResult` writes a **plain string** to the response body with a specified content type and status code. |
| **Technical Definition** | `Microsoft.AspNetCore.Mvc.ContentResult` holds `Content`, `ContentType` (default `"text/plain; charset=utf-8"`), and `StatusCode` (default 200). During execution it writes `Content` directly to the response body. Constructed by `Controller.Content(...)`. |
| **Beginner-Friendly Explanation** | `ContentResult` is the **plain text result**. Use it when you want to return a simple string (HTML fragment, plain text, CSV line) without a view or JSON serialization. |

### Purposes (each begins with “To”)

- **To** return plain text, HTML fragments, or small data strings directly.
- **To** specify a custom content type (`text/html`, `text/plain`, `text/csv`).
- **To** set a non-standard HTTP status code alongside content.
- **To** avoid the overhead of view rendering for trivial strings.

### Syntax Rules and Structure

#### General Syntax

```csharp
return Content("Hello, world!");                             // text/plain, 200
return Content("<b>Hello</b>", "text/html");                 // text/html, 200
return new ContentResult { Content = "Oops", StatusCode = 400 };
```

#### Component Breakdown

| Parameter | Description |
|-----------|-------------|
| `content` | String content. |
| `contentType` | MIME type. |
| `statusCode` | HTTP status code. |

#### Constraints and Limitations

- **No encoding validation** – caller must ensure proper escaping when `text/html` is used.
- **Not for large data** – use `FileResult` for large content.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Simple Text

```csharp
public IActionResult Ping() => Content("pong");
```

**Expected Output:** `200 OK`, body `pong`, `Content-Type: text/plain; charset=utf-8`.

**Why this output?** The default content type is `text/plain`.

---

#### Example 2: HTML Fragment with Custom Status

```csharp
public IActionResult Badge() =>
    new ContentResult { Content = "<span class='badge'>New</span>",
                        ContentType = "text/html",
                        StatusCode = 200 };
```

**Expected Output:** HTML `<span>` fragment rendered by the client.

**Why this output?** The custom `ContentType` instructs the client to treat the content as HTML.

### Real-World Cases

- **Health checks** – `Content("ok")`.
- **Dynamic HTML fragments** – for AJAX responses without full views.
- **CSV lines** – `Content(csv, "text/csv")`.

### References

- `ContentResult` API – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.contentresult

---

## 12. `StatusCodeResult`

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | `StatusCodeResult` sets the HTTP response **status code** without a body. |
| **Technical Definition** | `Microsoft.AspNetCore.Mvc.StatusCodeResult` sets `HttpContext.Response.StatusCode` to the supplied value and returns. Constructed by `Controller.StatusCode(...)` (e.g., `StatusCode(418)`) or via `IActionResult` helpers. Specific subclasses like `NotFoundResult`, `BadRequestResult`, `OkResult`, `NoContentResult`, and `CreatedResult` provide fixed status codes. |
| **Beginner-Friendly Explanation** | `StatusCodeResult` is the **“just send a status code”** result. Use it when the HTTP status code itself is the meaningful response (e.g., 418 I’m a teapot, 403 Forbidden, 429 Too Many Requests) and no body is required. |

### Purposes (each begins with “To”)

- **To** return a specific HTTP status code without additional content.
- **To** signal success or failure using standard HTTP semantics.
- **To** integrate with API clients that inspect status codes only.

### Syntax Rules and Structure

#### General Syntax

```csharp
return StatusCode(418);                 // 418 I'm a teapot
return StatusCode(429);                 // Too Many Requests
```

#### Component Breakdown

| Parameter | Description |
|-----------|-------------|
| `statusCode` | Any integer status code. |

#### Constraints and Limitations

- **No body** – if the client expects a JSON error, use `ObjectResult` or `BadRequest(object)` instead.
- **Use specific subclasses where possible** – `NotFound()` is clearer than `StatusCode(404)`.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Custom Status

```csharp
public IActionResult Teapot() => StatusCode(418);
```

**Expected Output:** `418 I'm a teapot` with empty body.

**Why this output?** `StatusCodeResult` sets the response code directly.

### Real-World Cases

- **Rate limiting** – return 429.
- **Maintenance mode** – return 503.
- **Easter eggs** – return 418.

### References

- `StatusCodeResult` API – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.statuscoderesult

---

## 13. `NotFoundResult`

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | `NotFoundResult` returns **HTTP 404 Not Found** with no body. |
| **Technical Definition** | `Microsoft.AspNetCore.Mvc.NotFoundResult` sets `Response.StatusCode = 404`. Constructed by `ControllerBase.NotFound()`. For APIs, `NotFound(object)` returns a 404 with an optional body via `NotFoundObjectResult`. |
| **Beginner-Friendly Explanation** | `NotFoundResult` is the **“resource missing”** result. When a product with the requested ID doesn’t exist, return `NotFound()` so the client gets a 404. |

### Purposes (each begins with “To”)

- **To** signal that the requested resource does not exist.
- **To** follow REST semantics for missing resources.
- **To** provide a clear, standard status code for clients and search engines.

### Syntax Rules and Structure

```csharp
return NotFound();                       // 404 empty body
return NotFound(new { msg = "..." });    // 404 with body (NotFoundObjectResult)
```

#### Constraints and Limitations

- **Not for validation failures** – use `BadRequest` (400) or `ValidationProblem`.
- **Not for authorization failures** – use `Forbid` (403) or `Challenge` (401).

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Missing Resource

```csharp
public ActionResult<ProductDto> GetById(int id)
{
    var product = _service.GetById(id);
    if (product == null) return NotFound();
    return product;
}
```

**Expected Output:** `404 Not Found` when the product is missing.

**Why this output?** The action explicitly signals absence using the standard 404.

### Real-World Cases

- **API detail lookups** – item not found.
- **MVC pages** – friendly 404 page (often via `UseStatusCodePages`).

### References

- `NotFoundResult` API – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.notfoundresult

---

## 14. `BadRequestResult`

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | `BadRequestResult` returns **HTTP 400 Bad Request** with no body. |
| **Technical Definition** | `Microsoft.AspNetCore.Mvc.BadRequestResult` sets `Response.StatusCode = 400`. Constructed by `ControllerBase.BadRequest()`. `BadRequest(object)` returns `BadRequestObjectResult` with a body—often used with `ModelState` for validation errors. With `[ApiController]`, invalid `ModelState` automatically produces a `ValidationProblemDetails` (400). |
| **Beginner-Friendly Explanation** | `BadRequestResult` is the **“client sent invalid data”** result. Return it (or `BadRequest(modelState)`) when the request cannot be processed because of input issues. |

### Purposes (each begins with “To”)

- **To** signal that the client’s input is invalid (400).
- **To** return `ModelState` errors (via `BadRequest(ModelState)` or `ValidationProblem`).
- **To** follow HTTP semantics for client errors.

### Syntax Rules and Structure

```csharp
return BadRequest();                        // 400 no body
return BadRequest("Invalid payload");       // 400 string body
return BadRequest(ModelState);              // 400 with validation details
return ValidationProblem(ModelState);       // 400 application/problem+json
```

#### Constraints and Limitations

- **Not for authentication/authorization** – use `Unauthorized`/`Forbid`.
- **Not for server errors** – use `Problem` (500) or exception middleware.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Validation Failure

```csharp
[HttpPost]
public ActionResult<ProductDto> Create(ProductDto dto)
{
    if (!ModelState.IsValid) return ValidationProblem(ModelState);
    return CreatedAtAction(nameof(GetById), new { id = _service.Create(dto).Id }, dto);
}
```

**Expected Output:** `400 Bad Request` with `application/problem+json` details on invalid input.

**Why this output?** `ValidationProblem` produces standard Problem Details (RFC 7807) responses with field errors.

### Real-World Cases

- **Form validation** – return errors to API clients.
- **Malformed JSON** – automatic 400 with `[ApiController]`.

### References

- `BadRequestResult` API – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.badrequestresult
- Handle errors in ASP.NET Core web APIs – https://learn.microsoft.com/en-us/aspnet/core/web-api/handle-errors

---

## 15. `OkResult`

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | `OkResult` returns **HTTP 200 OK** with **no body**. |
| **Technical Definition** | `Microsoft.AspNetCore.Mvc.OkResult` sets `Response.StatusCode = 200`. Constructed by `ControllerBase.Ok()` (no args). `Ok(object)` returns `OkObjectResult` with a serialized body. |
| **Beginner-Friendly Explanation** | `OkResult` is the **“success, no payload”** result. Use `Ok()` for actions that succeeded but don’t need to return data (e.g., a HEAD check or a health endpoint). |

### Purposes

- **To** signal success (200) without returning a body.
- **To** support HEAD or health-check endpoints.
- **To** respond to commands whose outcome is confirmed by status alone.

### Syntax Rules and Structure

```csharp
return Ok();                     // 200 no body
return Ok(data);                 // 200 with serialized body (OkObjectResult)
```

#### Constraints and Limitations

- **Use `NoContent` (204)** for status-only responses after mutations (POST/PUT/DELETE) where you don’t have a body.

### Multiple Annotated Complete Step-by-Step Code Examples

```csharp
public IActionResult Health() => Ok();
```

**Expected Output:** `200 OK`, empty body.

**Why this output?** `Ok()` with no arguments produces `OkResult`, which writes only a status.

### Real-World Cases

- **Health endpoints** – `GET /health`.
- **Command acknowledgements** – signal success with no payload.

### References

- `OkResult` API – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.okresult

---

## 16. `CreatedResult`

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | `CreatedResult` returns **HTTP 201 Created**, setting the `Location` header to the URL of the new resource and including a response body. |
| **Technical Definition** | `Microsoft.AspNetCore.Mvc.CreatedResult` sets `StatusCode = 201`, `Location` to the supplied URI string, and serializes the body. Constructed by `ControllerBase.Created(uri, value)`. For route-based creation, `CreatedAtAction`/`CreatedAtRoute` generate the `Location` via URL generation. |
| **Beginner-Friendly Explanation** | `CreatedResult` is the **“I made a new thing”** result. After a POST creates a resource, return `Created(...)` with the URL of the new item and its data. Clients can then GET that URL to retrieve it. |

### Purposes

- **To** signal successful creation (201) per REST semantics.
- **To** provide a `Location` header pointing to the new resource.
- **To** return the created resource representation in the body.

### Syntax Rules and Structure

```csharp
return Created($"/api/products/{id}", dto);                 // Explicit URL
return CreatedAtAction(nameof(GetById), new { id }, dto);   // URL from action
return CreatedAtRoute("GetProduct", new { id }, dto);       // URL from named route
```

#### Constraints and Limitations

- **Always include Location** – the HTTP 201 contract requires it.
- **Prefer `CreatedAtAction`/`CreatedAtRoute`** – they adapt to route template changes.

### Multiple Annotated Complete Step-by-Step Code Examples

```csharp
[HttpPost]
public ActionResult<ProductDto> Create(ProductDto dto)
{
    var created = _service.Create(dto);
    return CreatedAtAction(nameof(GetById), new { id = created.Id }, created);
}
```

**Expected Output:** `201 Created`, `Location: /api/Products/42`, body = created product JSON.

**Why this output?** `CreatedAtAction` generates the `Location` from the named action and route values.

### Real-World Cases

- **REST resource creation** – POST endpoints.
- **File uploads** – return the new file’s URL.

### References

- `CreatedResult` API – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.createdresult
- `CreatedAtAction` API – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.controllerbase.createdataction

---

## 17. `NoContentResult`

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | `NoContentResult` returns **HTTP 204 No Content** with no body. |
| **Technical Definition** | `Microsoft.AspNetCore.Mvc.NoContentResult` sets `Response.StatusCode = 204`. Constructed by `ControllerBase.NoContent()`. Semantically means “success, but there is nothing to return.” |
| **Beginner-Friendly Explanation** | `NoContentResult` is the **“done, nothing to say”** result. After successfully updating or deleting a resource, return `NoContent()` to signal success without a body. |

### Purposes

- **To** signal success (204) after a mutation with no payload.
- **To** follow REST semantics for updates/deletes.
- **To** save bandwidth by omitting a body when unnecessary.

### Syntax Rules and Structure

```csharp
return NoContent();                       // 204
```

#### Constraints and Limitations

- **Don’t confuse with `Ok()`** – 200 OK allows a body; 204 signals none by contract.
- **Not for creation** – use `Created` (201) so the client knows the resource URL.

### Multiple Annotated Complete Step-by-Step Code Examples

```csharp
[HttpPut("{id:int}")]
public IActionResult Update(int id, ProductDto dto)
{
    var exists = _service.Exists(id);
    if (!exists) return NotFound();
    _service.Update(id, dto);
    return NoContent();
}
```

**Expected Output:** `204 No Content` on success; `404 Not Found` if the resource doesn’t exist.

**Why this output?** The action differentiates between “updated successfully” and “nothing to update.”

### Real-World Cases

- **PUT/DELETE endpoints** – standard 204 responses.
- **Bulk operations** – signal success without body.

### References

- `NoContentResult` API – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.nocontentresult

---

## 18. Action Method Best Practices

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Action method best practices are guidelines for writing actions that are **small, predictable, testable, and semantically correct** in their use of HTTP. |
| **Technical Definition** | Best practices include: returning the correct result type for the scenario, using `IActionResult` only when multiple outcomes are possible, using `ActionResult<T>` for APIs, following HTTP semantics (200/201/204/400/404), avoiding business logic in actions, using model validation via filters or `[ApiController]`, and keeping action signatures consistent for testability. |
| **Beginner-Friendly Explanation** | Best practices are the **rules of thumb** that keep actions clean. For example: return `NotFound()` when something is missing, `CreatedAtAction` when creating, `NoContent` after deletes, and keep actions short by moving logic to services. |

### Purposes

- **To** produce semantically correct HTTP responses (status codes matching the outcome).
- **To** keep actions small, focused, and easy to unit test.
- **To** provide clear API contracts through typed return types.
- **To** avoid duplicating validation and error handling by using filters and `[ApiController]`.
- **To** make actions predictable across the codebase.

### Guideline Checklist

| Guideline | Rationale | Example |
|-----------|-----------|---------|
| **Return the semantically correct result** | HTTP status codes communicate intent. | `CreatedAtAction` for create; `NoContent` for update/delete; `NotFound` for missing. |
| **Use `ActionResult<T>` for typed APIs** | Type-safety + OpenAPI schemas. | `public ActionResult<ProductDto> Get(int id)` |
| **Use `IActionResult` for MVC view actions** | Allows multiple result types (view, 404). | `public IActionResult Details(int id)` |
| **Don’t return `null`** | The framework cannot execute `null`; runtime error. | Always return a concrete `IActionResult`. |
| **Keep actions thin** | Business logic belongs in services. | `_service.Create(dto); return CreatedAtAction(...)` |
| **Let `[ApiController]` handle validation** | Automatic 400 with Problem Details. | Don’t write `if (!ModelState.IsValid)` manually in APIs. |
| **Use `ValidationProblem` for validation errors** | Standard Problem Details shape. | `if (!ModelState.IsValid) return ValidationProblem(ModelState);` |
| **Avoid `try/catch` in actions** | Exception handling is middleware/filters’ concern. | Use `UseExceptionHandler` or exception filters. |
| **Use async actions for I/O** | Improves scalability. | `public async Task<IActionResult> Index()` |
| **Be careful with open redirects** | Validate `returnUrl` before redirecting. | Use `Url.IsLocalUrl(returnUrl)` or `LocalRedirect`. |

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Complete REST Endpoint Following Best Practices

```csharp
[ApiController]
[Route("api/[controller]")]
public class ProductsController : ControllerBase
{
    private readonly IProductService _service;
    public ProductsController(IProductService service) => _service = service;

    [HttpGet("{id:int}")]
    public ActionResult<ProductDto> GetById(int id)
    {
        var product = _service.GetById(id);
        return product is null ? NotFound() : product;
    }

    [HttpPost]
    public ActionResult<ProductDto> Create(ProductDto dto)
    {
        var created = _service.Create(dto);
        return CreatedAtAction(nameof(GetById), new { id = created.Id }, created);
    }

    [HttpPut("{id:int}")]
    public IActionResult Update(int id, ProductDto dto)
    {
        if (!_service.Exists(id)) return NotFound();
        _service.Update(id, dto);
        return NoContent();
    }

    [HttpDelete("{id:int}")]
    public IActionResult Delete(int id)
    {
        if (!_service.Exists(id)) return NotFound();
        _service.Delete(id);
        return NoContent();
    }
}
```

**Expected Output:**

| Verb | URL | Response |
|------|-----|----------|
| GET | `/api/Products/5` | 200 JSON or 404 |
| POST | `/api/Products` | 201 with Location |
| PUT | `/api/Products/5` | 204 or 404 |
| DELETE | `/api/Products/5` | 204 or 404 |

**Why this output?** Each action uses the semantically correct result and remains focused on HTTP concerns.

### Real-World Cases

- **Public REST API** – consistent status codes (200/201/204/400/404) simplify client integration.
- **MVC web app** – `IActionResult` actions that return views or 404s.
- **Hybrid API + MVC** – separate controllers for views (`Controller`) and data (`ControllerBase` + `ActionResult<T>`).

### References

- Controller action return types in ASP.NET Core web API – https://learn.microsoft.com/en-us/aspnet/core/web-api/action-return-types
- Handle errors in ASP.NET Core web APIs – https://learn.microsoft.com/en-us/aspnet/core/web-api/handle-errors
- URL generation & `CreatedAtAction` – https://learn.microsoft.com/en-us/aspnet/core/fundamentals/routing#url-generation
- Prevent open redirects – https://learn.microsoft.com/en-us/aspnet/core/security/preventing-open-redirects

---

## Summary Table: Action Return Types at a Glance

| Type | HTTP Status | Body | Typical Use |
|------|-------------|------|-------------|
| **`IActionResult`** | Variable | Variable | MVC actions with multiple outcomes |
| **`ActionResult<T>`** | Variable | Value or none | Typed API actions with multiple outcomes |
| **`ViewResult`** | 200 | HTML page | MVC view pages |
| **`PartialViewResult`** | 200 | HTML fragment | AJAX / partial rendering |
| **`JsonResult`** | 200 (or set) | JSON | MVC controllers returning JSON |
| **`RedirectResult`** | 301/302 | None | External URL redirects |
| **`RedirectToActionResult`** | 301/302 | None | Redirect to an action |
| **`RedirectToRouteResult`** | 301/302 | None | Redirect to a named route |
| **`FileResult`** | 200 | File | Downloads and streams |
| **`ContentResult`** | 200 (or set) | Text | Plain text / HTML fragments |
| **`StatusCodeResult`** | Any | None | Custom status codes |
| **`NotFoundResult`** | 404 | None | Missing resources |
| **`BadRequestResult`** | 400 | None (or `ValidationProblem`) | Invalid input |
| **`OkResult`** | 200 | None | Success, no body |
| **`CreatedResult`** | 201 | Created value | Resource creation with Location |
| **`NoContentResult`** | 204 | None | Success, no body after mutation |

---

## References (Full List)

- Handle requests with controllers in ASP.NET Core MVC – https://learn.microsoft.com/en-us/aspnet/core/mvc/controllers/actions
- Controller action return types in ASP.NET Core web API – https://learn.microsoft.com/en-us/aspnet/core/web-api/action-return-types
- Format response data in ASP.NET Core web API – https://learn.microsoft.com/en-us/aspnet/core/web-api/advanced/formatting
- Handle errors in ASP.NET Core web APIs – https://learn.microsoft.com/en-us/aspnet/core/web-api/handle-errors
- Views in ASP.NET Core MVC – https://learn.microsoft.com/en-us/aspnet/core/mvc/views/overview
- Partial views in ASP.NET Core – https://learn.microsoft.com/en-us/aspnet/core/mvc/views/partial
- URL generation in ASP.NET Core – https://learn.microsoft.com/en-us/aspnet/core/fundamentals/routing#url-generation
- Prevent open redirects in ASP.NET Core – https://learn.microsoft.com/en-us/aspnet/core/security/preventing-open-redirects
- `IActionResult` API – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.iactionresult
- `ActionResult<T>` API – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.actionresult-1
- `ViewResult` API – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.viewresult
- `PartialViewResult` API – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.partialviewresult
- `JsonResult` API – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.jsonresult
- `RedirectResult` API – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.redirectresult
- `RedirectToActionResult` API – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.redirecttoactionresult
- `RedirectToRouteResult` API – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.redirecttorouteresult
- `FileResult` API – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.fileresult
- `ContentResult` API – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.contentresult
- `StatusCodeResult` API – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.statuscoderesult
- `NotFoundResult` API – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.notfoundresult
- `BadRequestResult` API – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.badrequestresult
- `OkResult` API – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.okresult
- `CreatedResult` API – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.createdresult
- `NoContentResult` API – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.nocontentresult