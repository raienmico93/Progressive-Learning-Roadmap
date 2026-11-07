# Action Results in ASP.NET Core MVC: A Comprehensive Cheat Sheet

> **Version Scope & Compatibility Note**
> This cheat sheet targets **ASP.NET Core MVC** (3.0 and later, including .NET 6/7/8/9/10/11). It focuses on the **helper methods** exposed by `Controller` and `ControllerBase` — `View()`, `PartialView()`, `Json()`, `Redirect()`, `RedirectToAction()`, `RedirectToRoute()`, `File()`, `Content()`, `StatusCode()`, `NotFound()`, `BadRequest()`, `Ok()`, `Created()`, and `NoContent()` — that **construct** the corresponding action result objects. The underlying result types (`ViewResult`, `JsonResult`, `OkObjectResult`, etc.) are covered only in passing; the emphasis here is on how the helper methods are used and what they produce. `Controller` adds view helpers (`View`, `PartialView`) while `ControllerBase` provides the HTTP-focused helpers (`Ok`, `NotFound`, `Created`, etc.). All helpers are stable across modern ASP.NET Core versions. `[ApiController]` adds automatic behaviours (e.g., 400 responses on invalid `ModelState`) that complement these helpers.

---

## Topic Overview

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Action results are the **objects returned by action methods** that describe how the framework should construct the HTTP response (view, JSON, redirect, file, status code, etc.). Action **helpers** are convenience methods on `Controller`/`ControllerBase` that create these result objects. |
| **Technical Definition** | An action result is an instance of `IActionResult` (or `ActionResult<T>`). Helper methods such as `View()`, `Ok()`, `NotFound()`, and `CreatedAtAction()` return concrete implementations (`ViewResult`, `OkResult`, `NotFoundResult`, `CreatedAtActionResult`, etc.) that override `ExecuteResultAsync(ActionContext)` to write the HTTP response. The result-execution pipeline invokes `ExecuteResultAsync`; this is **separated from the action body**, enabling unit tests to inspect the result object without executing it. |
| **Beginner-Friendly Explanation** | Action results are the **“what to send back”** objects. When your action finishes, it doesn't write to the HTTP response directly—it returns a result object like `View()`, `Json(data)`, or `NotFound()`. The framework then executes that result to produce the actual HTTP response (HTML page, JSON body, 404 status, etc.). The helper methods like `Ok()` and `NotFound()` are shortcuts for constructing those result objects. |

### Key Characteristics

- **Declarative and testable** – Actions return result objects; tests can assert on the type and properties without executing HTTP responses.
- **Helper-based** – `Controller`/`ControllerBase` expose helpers (`View()`, `Ok()`, etc.) that construct results conveniently.
- **Typed alternatives** – `ActionResult<T>` combines typed returns with status-code results for APIs.
- **Framework-executed** – Results are executed **after** the action returns, by `IActionResultExecutor` implementations.
- **Composable** – Different helpers can be returned from the same action based on runtime conditions.
- **Semantic** – Each helper maps to a clear HTTP status code and body shape (200, 201, 204, 302, 400, 404).
- **Extensible** – Custom `IActionResult` implementations can be created; helpers are only the built-in set.

### Prerequisites

- **.NET SDK** (6.0+ recommended; examples use .NET 8/10).
- **ASP.NET Core MVC** services registered (`AddControllersWithViews()` or `AddControllers()`).
- Routing configured (`MapControllers()` or `MapControllerRoute()`).
- Basic understanding of **controllers**, **action methods**, and **HTTP status codes**.

### Related Programming Areas

- **Action Methods** – The methods that return these results.
- **Routing and URL Generation** – `RedirectToAction` and `CreatedAtAction` use URL generation.
- **Razor View Engine** – `View()` and `PartialView()` depend on view discovery.
- **JSON Serialization** – `Json()` uses the registered JSON serializer.
- **File I/O** – `File()` streams content to the client.
- **Filters** – Result filters run after the action and before/after result execution.

### Core Concepts / Features

1. **Action Results (General)** – The mechanism and the helper/result distinction.
2. **`View()`** – Render a Razor view.
3. **`PartialView()`** – Render a Razor partial (no layout).
4. **`Json()`** – Serialize data to JSON.
5. **`Redirect()`** – Redirect to a raw URL.
6. **`RedirectToAction()`** – Redirect to an action.
7. **`RedirectToRoute()`** – Redirect to a named route.
8. **`File()`** – Return file content.
9. **`Content()`** – Return plain text/HTML.
10. **`StatusCode()`** – Return a custom status code.
11. **`NotFound()`** – Return 404.
12. **`BadRequest()`** – Return 400.
13. **`Ok()`** – Return 200.
14. **`Created()`** – Return 201 with `Location`.
15. **`NoContent()`** – Return 204.
16. **Action Result Best Practices** – Guidelines for clean, correct result usage.

Each concept is elaborated with Definitions, Purposes, Syntax Rules & Structure, Annotated Code Examples with Expected Outputs, Real-World Cases, and References.

---

## 1. Action Results (General)

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | An action result is the **value returned by an action method** that instructs the framework how to produce the HTTP response. |
| **Technical Definition** | Action results implement `IActionResult` (or are wrapped in `ActionResult<T>`). The framework's result-execution pipeline (invoked after the action returns) calls `ExecuteResultAsync(ActionContext)` on the returned result, which writes headers, status, and body to `HttpResponse`. Helpers on `Controller`/`ControllerBase` are factory methods for these result objects. The returned object is **not** the HTTP response—it **describes** how to build it. |
| **Beginner-Friendly Explanation** | An action result is the **instruction sheet** for building the HTTP response. Instead of writing directly to the response, your action says: “Return a view,” “Return JSON,” “Return 404.” The framework reads the instruction and produces the response. Helpers like `View()`, `Json()`, and `NotFound()` make it easy to write these instructions. |

### Purposes (each begins with “To”)

- **To** separate **what** to return from **how** to write it to the HTTP response.
- **To** allow actions to return different result types based on runtime conditions (e.g., 200 or 404).
- **To** enable unit testing of actions by inspecting the returned result object without executing HTTP.
- **To** provide semantically clear helper methods (`Ok()`, `NotFound()`, `Created()`) that map to standard HTTP status codes.
- **To** encapsulate complex response construction (view rendering, JSON serialization, file streaming) in framework components.

### Syntax Rules and Structure

#### General Syntax (with helpers)

```csharp
public IActionResult Action()
{
    if (/* condition */) return NotFound();
    return Ok(data);           // or View(model), Json(data), Redirect(...)
}
```

#### General Syntax (typed for APIs)

```csharp
public ActionResult<ProductDto> GetById(int id)
{
    var product = _service.GetById(id);
    if (product == null) return NotFound();
    return product;            // Wrapped as OkObjectResult
}
```

#### Component Breakdown

| Component | Description |
|-----------|-------------|
| Return type | `IActionResult` (multiple outcomes) or `ActionResult<T>` (typed API). |
| Helper call | `View()`, `Ok()`, `NotFound()`, `CreatedAtAction()`, etc. |
| Concrete result | `ViewResult`, `OkObjectResult`, `NotFoundResult`, etc. |
| Execution | Framework invokes `ExecuteResultAsync` after the action returns. |

#### Syntax Rules

- **Do not return `null`** — the framework cannot execute a null result.
- **Use helpers, not `new` constructors**, for readability and correctness.
- **Choose the helper matching the semantic outcome** (200 → `Ok`, 201 → `Created`, 204 → `NoContent`, 404 → `NotFound`, 400 → `BadRequest`).
- **`ActionResult<T>` implicitly converts** both `T` and `IActionResult`.

#### Constraints and Limitations

- **Helpers are per-controller** – `View()` exists on `Controller`, not `ControllerBase`. `Ok()`, `NotFound()`, etc., exist on both.
- **One result per action execution** – you cannot “return two results”; use conditional logic to choose.
- **Async-safe** – helpers are synchronous constructors; execution is asynchronous inside the framework.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Multi-Outcome Action

```csharp
public IActionResult Details(int id)
{
    var product = _service.GetById(id);
    if (product == null) return NotFound();     // NotFoundResult (404)
    return View(product);                        // ViewResult (200 HTML)
}
```

**Step-by-step explanation:**

1. The action returns `IActionResult`, so multiple result types are allowed.
2. If `product` is `null`, `NotFound()` returns a `NotFoundResult` → 404.
3. Otherwise, `View(product)` returns a `ViewResult` → HTML page.

**Expected Output:**

| Scenario | Result |
|----------|--------|
| Product exists | 200 OK with HTML |
| Product missing | 404 Not Found |

**Why this output?** The framework executes whichever result object the action returns; helpers produce the correct concrete result type.

---

#### Example 2: Typed API Action with `ActionResult<T>`

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
        return product;   // Implicitly wrapped as 200 OK with JSON
    }
}
```

**Expected Output:**

| Scenario | Result |
|----------|--------|
| Product exists | 200 OK with JSON body |
| Product missing | 404 Not Found |

**Why this output?** `ActionResult<T>` implicitly converts `T` to `OkObjectResult` and accepts `IActionResult` returns like `NotFoundResult`.

### Real-World Cases

- **MVC CRUD controllers** – returning `View`, `NotFound`, `RedirectToAction`.
- **REST APIs** – returning `Ok`, `Created`, `NotFound`, `NoContent`, `BadRequest`.
- **Mixed endpoints** – API controllers returning JSON, view controllers returning HTML.

### References

- Controller action return types in ASP.NET Core web API – https://learn.microsoft.com/en-us/aspnet/core/web-api/action-return-types
- Handle requests with controllers in ASP.NET Core MVC – https://learn.microsoft.com/en-us/aspnet/core/mvc/controllers/actions

---

## 2. `View()`

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | `View()` returns a `ViewResult` that renders a **Razor view** (`.cshtml`) and writes the resulting HTML to the response. |
| **Technical Definition** | `Microsoft.AspNetCore.Mvc.Controller.View(...)` has several overloads: `View()`, `View(object model)`, `View(string viewName)`, and `View(string viewName, object model)`. The framework resolves the view via `IViewEngine` using search paths (`/Views/{Controller}/{View}.cshtml`, `/Views/Shared/{View}.cshtml`), executes it with the supplied model, and writes the HTML to the response with status 200. |
| **Beginner-Friendly Explanation** | `View()` means: “Send an **HTML page** back to the browser.” You give it a model (data for the page), and the framework finds the matching `.cshtml` file, renders it with your data, and sends the resulting HTML. |

### Purposes (each begins with “To”)

- **To** render an HTML page from a Razor view.
- **To** pass a strongly typed model to the view for display.
- **To** allow explicit view names to reuse views across actions (e.g., `View("Details", model)` from a `List` action).
- **To** integrate with layouts, partials, and Tag Helpers in the Razor view engine.

### Syntax Rules and Structure

#### General Syntax

```csharp
return View();                            // Convention: Views/{Controller}/ActionName.cshtml
return View(model);                       // With model
return View("ViewName");                  // Explicit view name
return View("ViewName", model);           // Explicit view name + model
```

#### Component Breakdown

| Parameter | Description |
|-----------|-------------|
| `viewName` (optional) | Name of the view to render; defaults to the action name. |
| `model` (optional) | Strongly typed model passed to the view (`@model T`). |

#### Syntax Rules

- **View name defaults to the action name**.
- **Search order:** `/Views/{Controller}/{View}.cshtml` → `/Views/Shared/{View}.cshtml`.
- **Model type should match `@model`** in the view; mismatch throws at runtime.
- **Available only on `Controller`** (`ControllerBase` lacks view support).

#### Constraints and Limitations

- **Not for APIs** – use `Ok()` / `ActionResult<T>` for JSON.
- **View name typos fail at runtime**, not compile time (use constants or Razor Pages conventions where possible).
- **Model binding mismatch** – passing a different model type than the view declares causes a runtime error.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Convention-Based View

```csharp
public IActionResult Index()
{
    var products = _service.GetAll();
    return View(products);
}
```

**Step-by-step explanation:**

1. `View(products)` looks for `Views/{Controller}/Index.cshtml`.
2. The model is passed to the view via `@model IEnumerable<Product>`.
3. The view engine renders HTML and writes it to the response.

**Expected Output:** 200 OK with HTML listing products.

**Why this output?** The view is discovered by convention (action name) and executed with the supplied model.

---

#### Example 2: Explicit View Name

```csharp
public IActionResult List()
{
    var products = _service.GetAll();
    return View("Index", products); // Renders Index.cshtml from List action
}
```

**Expected Output:** 200 OK with HTML using `Index.cshtml` and the product list.

**Why this output?** The explicit view name overrides the convention; the same view serves multiple actions.

---

#### Example 3: Form Re-Render After Validation Failure

```csharp
[HttpPost]
[ValidateAntiForgeryToken]
public IActionResult Create(ProductFormModel model)
{
    if (!ModelState.IsValid) return View(model);
    _service.Create(model);
    return RedirectToAction(nameof(Index));
}
```

**Expected Output:**

| Scenario | Result |
|----------|--------|
| Valid form | 302 redirect to `Index` |
| Invalid form | 200 OK re-rendering the Create view with errors |

**Why this output?** `View(model)` re-renders the form with validation messages, preserving user input.

### Real-World Cases

- **CRUD pages** – list, detail, create, edit views.
- **Form re-display** – re-render with validation errors after POST.
- **Shared views** – reusing views across controllers via explicit names.

### References

- Views in ASP.NET Core MVC – https://learn.microsoft.com/en-us/aspnet/core/mvc/views/overview
- `ViewResult` API – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.viewresult

---

## 3. `PartialView()`

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | `PartialView()` returns a `PartialViewResult` that renders a **partial view** (Razor fragment **without a layout**) and writes it to the response. |
| **Technical Definition** | `Microsoft.AspNetCore.Mvc.Controller.PartialView(...)` has overloads mirroring `View(...)`: no args, model, explicit name, and name + model. The partial is resolved through the view engine **without applying `_ViewStart.cshtml`**, so no layout is rendered. Partial views are commonly used for AJAX responses that inject HTML fragments into an existing page. |
| **Beginner-Friendly Explanation** | `PartialView()` means: “Send a **fragment of HTML**, not a whole page.” It renders only the partial's markup, skipping headers, footers, and layout. Useful when you want to load a section of a page dynamically via AJAX. |

### Purposes (each begins with “To”)

- **To** return an **HTML fragment** for dynamic injection into an existing page.
- **To** avoid re-rendering the full layout when only part of the page changes.
- **To** support AJAX-based pagination, search results, or modal content.
- **To** promote reusability by returning reusable markup that multiple pages share.

### Syntax Rules and Structure

#### General Syntax

```csharp
return PartialView();                    // Convention: _ActionName.cshtml (or ViewName)
return PartialView(model);
return PartialView("_PartialName");
return PartialView("_PartialName", model);
```

#### Component Breakdown

| Parameter | Description |
|-----------|-------------|
| `viewName` | Partial view name, often prefixed with `_` by convention. |
| `model` | Model passed to the partial (`@model T`). |

#### Syntax Rules

- **No layout** is applied (`_ViewStart.cshtml` is skipped).
- **Partial views typically begin with `_`** (convention, not requirement).
- **Model type should match** the partial’s `@model`.

#### Constraints and Limitations

- **Not a full page** – returning a partial directly to a browser yields a bare fragment.
- **XSS concerns** – HTML fragments returned via AJAX must be sanitised by the client when inserted into the DOM.
- **CSP considerations** – inline scripts in partials may need nonces in restrictive CSP configurations.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: AJAX Product Rows

```csharp
[HttpGet("rows")]
public IActionResult Rows(int page = 1)
{
    var products = _service.GetPage(page, 20);
    return PartialView("_ProductRows", products);
}
```

```cshtml
@* Views/Products/_ProductRows.cshtml *@
@model IEnumerable<Product>
@foreach (var p in Model)
{
    <tr>
        <td>@p.Name</td>
        <td>@p.Price.ToString("C")</td>
    </tr>
}
```

**Step-by-step explanation:**

1. `PartialView("_ProductRows", products)` renders only the `<tr>` rows.
2. The client inserts the returned HTML into an existing table body.

**Expected Output:** HTML `<tr>` fragments only (no `<html>`, `<head>`, or layout).

**Why this output?** `PartialViewResult` bypasses `_ViewStart.cshtml`, so only the fragment is rendered.

---

#### Example 2: Modal Dialog Content

```csharp
public IActionResult ConfirmDelete(int id)
{
    var product = _service.GetById(id);
    if (product == null) return NotFound();
    return PartialView("_ConfirmDelete", product);
}
```

**Expected Output:** HTML fragment of the confirm-delete dialog.

**Why this output?** The client fetches the partial and displays it inside a modal; no full page is needed.

### Real-World Cases

- **Pagination via AJAX** – return only the row fragments.
- **Inline search results** – return matching items for a live-search container.
- **Modal dialogs** – confirm-delete, edit dialogs loaded on demand.
- **Widgets** – sidebar widgets rendered independently of the main page.

### References

- Partial views in ASP.NET Core – https://learn.microsoft.com/en-us/aspnet/core/mvc/views/partial
- `PartialViewResult` API – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.partialviewresult

---

## 4. `Json()`

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | `Json()` returns a `JsonResult` that **serializes a value to JSON** and writes it to the response with `Content-Type: application/json`. |
| **Technical Definition** | `Microsoft.AspNetCore.Mvc.Controller.Json(...)` has overloads accepting a value, optional `JsonSerializerOptions`, and optional status code. The `JsonResult` uses the registered `IJsonHelper` (backed by `System.Text.Json` in modern ASP.NET Core) to serialize. `JsonResult` bypasses content negotiation and always returns JSON, unlike POCO returns from API controllers that respect `Accept` headers. |
| **Beginner-Friendly Explanation** | `Json()` means: “Send **structured data** back as JSON.” Great for AJAX requests or API endpoints. You pass an object (or anonymous type) and the framework serializes it to JSON. |

### Purposes (each begins with “To”)

- **To** return structured data to AJAX or API clients in JSON format.
- **To** allow MVC controllers (which don't use `[ApiController]`) to return JSON without switching to `ControllerBase`.
- **To** customize serialization (options, status code, content type) for specific responses.
- **To** bypass content negotiation when the endpoint must always return JSON.

### Syntax Rules and Structure

#### General Syntax

```csharp
return Json(data);                                               // 200, application/json
return Json(data, new JsonSerializerOptions { ... });            // Custom options
return new JsonResult(data) { StatusCode = 400 };                // Custom status
```

#### Component Breakdown

| Parameter | Description |
|-----------|-------------|
| `data` | Any serializable object (POCO, anonymous type, collection). |
| `options` (optional) | `JsonSerializerOptions` controlling naming, converters, etc. |
| `StatusCode` | Optional status override (via `JsonResult`). |

#### Syntax Rules

- **Response is JSON** – no content negotiation; appropriate for AJAX-only endpoints.
- **Serialization uses System.Text.Json** (unless configured otherwise).
- **Prefer POCO returns in `[ApiController]`** for content negotiation and OpenAPI schemas.

#### Constraints and Limitations

- **Not for content negotiation** – if the client wants XML, `JsonResult` won't comply.
- **Large payloads** – streaming via `IAsyncEnumerable<T>` or `FileStreamResult` may be preferable.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Anonymous Object Response

```csharp
public IActionResult Stats()
{
    return Json(new { users = 42, active = 30 });
}
```

**Expected Output:** `200 OK`, `Content-Type: application/json`, body `{ "users": 42, "active": 30 }`.

**Why this output?** `JsonResult` serializes the anonymous object and writes it with JSON content type.

---

#### Example 2: JSON with Custom Status

```csharp
public IActionResult Error()
{
    return new JsonResult(new { error = "Invalid input" }) { StatusCode = 400 };
}
```

**Expected Output:** `400 Bad Request`, JSON body `{ "error": "Invalid input" }`.

**Why this output?** `JsonResult.StatusCode` overrides the default 200.

---

#### Example 3: Autocomplete Suggestions

```csharp
public IActionResult Suggest(string q)
{
    var suggestions = _service.Suggest(q).Take(10);
    return Json(suggestions.Select(s => new { id = s.Id, text = s.Name }));
}
```

**Expected Output:** `200 OK` with JSON array of suggestions.

**Why this output?** `JsonResult` serializes the projected list for the client-side autocomplete widget.

### Real-World Cases

- **AJAX stats endpoints** – simple JSON without switching to API controllers.
- **Autocomplete** – return suggestion lists.
- **Bootstrap data for SPAs** – return initial state as JSON.

### References

- `JsonResult` API – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.jsonresult
- Format response data in ASP.NET Core web API – https://learn.microsoft.com/en-us/aspnet/core/web-api/advanced/formatting

---

## 5. `Redirect()`

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | `Redirect()` returns a `RedirectResult` that issues an HTTP redirect (302 by default) to a **supplied URL string**. |
| **Technical Definition** | `Microsoft.AspNetCore.Mvc.Controller.Redirect(string url)` constructs a `RedirectResult` with `Url = url`, `Permanent = false` (302). `RedirectPermanent(string url)` uses `Permanent = true` (301). `LocalRedirect(string url)` (added later) validates that the URL is local, preventing open redirects. |
| **Beginner-Friendly Explanation** | `Redirect()` means: “Send the browser **to a different URL**.” Use it for **external** URLs or full paths you supply directly. For internal app routes, prefer `RedirectToAction`/`RedirectToRoute` so URLs stay in sync with route templates. |

### Purposes (each begins with “To”)

- **To** redirect the client to a **specific URL** (external or internal) that you supply directly.
- **To** perform permanent (301) or temporary (302) redirects.
- **To** support open-redirect-safe patterns via `LocalRedirect` when the URL may come from user input.
- **To** redirect after operations where a **named route** is not available but a URL is known.

### Syntax Rules and Structure

#### General Syntax

```csharp
return Redirect("https://example.com");                    // 302 Found
return RedirectPermanent("https://example.com");           // 301 Moved Permanently
return LocalRedirect("/local/path");                       // 302 with local validation
return new RedirectResult("https://example.com", permanent: false);
```

#### Component Breakdown

| Helper | Status | Notes |
|--------|--------|-------|
| `Redirect(url)` | 302 | Temporary. |
| `RedirectPermanent(url)` | 301 | Permanent; SEO-friendly. |
| `LocalRedirect(url)` | 302 | Throws if URL is not local — open-redirect safe. |

#### Syntax Rules

- **Always prefer `LocalRedirect`** when the URL may come from user input.
- **`Redirect` can be abused for open redirects** if passed untrusted URLs.
- **Use `RedirectToAction`/`RedirectToRoute` for internal routes** – better than hard-coded URLs.

#### Constraints and Limitations

- **Hard-coded URLs break when route templates change.**
- **301 redirects are cached aggressively by browsers** – change with care.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: External Redirect

```csharp
public IActionResult Docs()
{
    return Redirect("https://learn.microsoft.com/aspnet/core");
}
```

**Expected Output:** `302 Found`, `Location: https://learn.microsoft.com/aspnet/core`.

**Why this output?** `RedirectResult` sets the status and `Location` header; the client follows the redirect.

---

#### Example 2: Safe Local Redirect

```csharp
public IActionResult GoTo(string returnUrl)
{
    if (Url.IsLocalUrl(returnUrl)) return LocalRedirect(returnUrl);
    return RedirectToAction(nameof(Index));
}
```

**Expected Output:** Local URLs redirect; external URLs fall back to `Index`.

**Why this output?** `LocalRedirect` throws on non-local URLs; the guard uses `Url.IsLocalUrl` to avoid exceptions.

---

#### Example 3: Permanent Redirect for SEO

```csharp
public IActionResult OldPage() => RedirectPermanent("/new-page");
```

**Expected Output:** `301 Moved Permanently`, `Location: /new-page`.

**Why this output?** Permanent redirects inform search engines that the page moved.

### Real-World Cases

- **External SSO/OAuth** – redirect to provider URLs.
- **Legacy URL migration** – permanent redirects to new URLs.
- **Post-login redirect** – redirect to `returnUrl` with validation.

### References

- `RedirectResult` API – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.redirectresult
- Prevent open redirects – https://learn.microsoft.com/en-us/aspnet/core/security/preventing-open-redirects

---

## 6. `RedirectToAction()`

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | `RedirectToAction()` returns a `RedirectToActionResult` that redirects to a **named action** by generating the URL from controller/action names and route values. |
| **Technical Definition** | `Microsoft.AspNetCore.Mvc.Controller.RedirectToAction(...)` has overloads accepting action name, controller name (optional), and route values. The framework uses `LinkGenerator` (via `IUrlHelper.Action`) to compute the URL, sets `Location` header, and returns 302 (or 301 with `RedirectToActionPermanent`). This keeps redirects resilient to route template changes. |
| **Beginner-Friendly Explanation** | `RedirectToAction()` means: “Send the browser to **another action** in this app, identified by controller and action names.” You don't supply a URL—you supply names, and the framework builds the URL for you. This is safer than hard-coded URLs when routes change. |

### Purposes (each begins with “To”)

- **To** redirect to a known action by **controller/action names**, avoiding hard-coded URLs.
- **To** implement the **Post/Redirect/Get (PRG)** pattern after form submissions.
- **To** redirect after successful operations (create/edit/delete) to a list or detail page.
- **To** preserve URL generation rules (routes, areas) automatically.
- **To** support `RedirectToActionPermanent` for SEO-friendly redirects.

### Syntax Rules and Structure

#### General Syntax

```csharp
return RedirectToAction("Index");                                // Same controller
return RedirectToAction("Details", new { id = 5 });              // Same controller + values
return RedirectToAction("Index", "Products");                    // Different controller
return RedirectToAction("Details", "Products", new { id = 5 });  // Controller + values
return RedirectToActionPermanent("Index");                       // 301 permanent
```

#### Component Breakdown

| Parameter | Description |
|-----------|-------------|
| `actionName` | Target action method name. |
| `controllerName` (optional) | Target controller; defaults to current. |
| `routeValues` (optional) | Anonymous object or `RouteValueDictionary` for template expansion. |

#### Syntax Rules

- **Action must exist**; otherwise, URL generation may fail.
- **Prefer `nameof(ActionName)`** for type safety and refactoring support.
- **Ambient values** (like `area`) may influence the generated URL; specify `area` explicitly if you need to change it.

#### Constraints and Limitations

- **Does not work across areas** without explicitly specifying `area` route value.
- **URL generation failures** may result in empty `Location` — guard with tests or fall back to a different result.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: PRG After Create

```csharp
[HttpPost]
[ValidateAntiForgeryToken]
public IActionResult Create(ProductFormModel model)
{
    if (!ModelState.IsValid) return View(model);
    var product = _service.Create(model);
    return RedirectToAction(nameof(Details), new { id = product.Id });
}
```

**Expected Output:** `302 Found`, `Location: /Products/Details/42`.

**Why this output?** `RedirectToAction` generates the URL from the action name and route values; the browser then issues a GET.

---

#### Example 2: Redirect Across Controllers

```csharp
public IActionResult Checkout()
{
    if (!_cart.HasItems()) return RedirectToAction("Index", "Home");
    return View();
}
```

**Expected Output:** `302 Found`, `Location: /Home/Index`.

**Why this output?** The `controllerName` argument overrides the current controller.

---

#### Example 3: Permanent Redirect

```csharp
public IActionResult OldDashboard() =>
    RedirectToActionPermanent("Index", "Dashboard");
```

**Expected Output:** `301 Moved Permanently`, `Location: /Dashboard`.

**Why this output?** `RedirectToActionPermanent` signals a permanent move to search engines.

### Real-World Cases

- **PRG pattern** – post-create redirects.
- **Authentication flows** – redirect to login after failed actions.
- **Deprecated actions** – permanent redirects to replacements.

### References

- `RedirectToActionResult` API – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.redirecttoactionresult
- URL generation in ASP.NET Core – https://learn.microsoft.com/en-us/aspnet/core/fundamentals/routing#url-generation

---

## 7. `RedirectToRoute()`

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | `RedirectToRoute()` returns a `RedirectToRouteResult` that redirects to a **named route**, generating the URL from the route’s template and supplied route values. |
| **Technical Definition** | `Microsoft.AspNetCore.Mvc.Controller.RedirectToRoute(...)` overloads accept a route name plus optional route values, or just route values for the default route. `RedirectToRouteResult` uses `LinkGenerator.GetPathByRouteName` to compute the URL. `RedirectToRoutePermanent` performs 301 redirects. |
| **Beginner-Friendly Explanation** | `RedirectToRoute()` means: “Redirect to a **named route**.” Use it when your route has a **name** (like `"blog"` or `"default"`) and you want to redirect to it using route values—useful when the route doesn't map cleanly to a controller/action pair (e.g., catch-all routes). |

### Purposes (each begins with “To”)

- **To** redirect to a **named route** whose URL is generated from the route template.
- **To** handle routes whose structure doesn't map to controller/action pairs (e.g., `{*slug}` catch-alls).
- **To** decouple redirects from controller/action names by using the stable route name.
- **To** support `RedirectToRoutePermanent` for SEO-friendly permanent redirects.

### Syntax Rules and Structure

#### General Syntax

```csharp
return RedirectToRoute("routeName");
return RedirectToRoute("routeName", new { id = 5 });
return RedirectToRoute(new { controller = "Home", action = "Index" });  // value-based
return RedirectToRoutePermanent("routeName", new { slug = "x" });
```

#### Component Breakdown

| Parameter | Description |
|-----------|-------------|
| `routeName` | The route name assigned in `MapControllerRoute` or via `[Route(Name = ...)]`. |
| `routeValues` | Values used to expand the route template. |

#### Syntax Rules

- **Route name must exist** – unknown names throw `InvalidOperationException` or produce `null` URLs.
- **Prefer named routes** over `RedirectToAction` when the route structure doesn't match controller/action.

#### Constraints and Limitations

- **No compile-time checking** – route names are strings; typos cause runtime errors. Use constants to mitigate.
- **Naming collisions** — named routes must be unique app-wide.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Redirect to Named Blog Route

```csharp
// Program.cs
app.MapControllerRoute(
    name: "blog",
    pattern: "blog/{*slug}",
    defaults: new { controller = "Blog", action = "Article" });

// Action
public IActionResult GoToPost(string slug) => RedirectToRoute("blog", new { slug });
```

**Expected Output:** `302 Found`, `Location: /blog/my-first-post`.

**Why this output?** The named route's template is expanded with `slug` to produce the URL.

---

#### Example 2: Permanent Redirect to Route

```csharp
public IActionResult OldBlog(string slug) =>
    RedirectToRoutePermanent("blog", new { slug });
```

**Expected Output:** `301 Moved Permanently` with `Location` generated from the route.

**Why this output?** `RedirectToRoutePermanent` signals a permanent move; the URL is computed from the named route.

### Real-World Cases

- **Catch-all blog routes** – redirect to named `"blog"` route.
- **Area-based routes** – redirect to named routes in specific areas.
- **Custom URL structures** – routes that don't map cleanly to controller/action.

### References

- `RedirectToRouteResult` API – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.redirecttorouteresult
- URL generation by route name – https://learn.microsoft.com/en-us/aspnet/core/fundamentals/routing#generate-urls-by-route-name

---

## 8. `File()`

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | `File()` returns a `FileResult` (concrete subclass such as `FileContentResult`, `FileStreamResult`, `PhysicalFileResult`, or `VirtualFileResult`) that streams content to the client with the specified content type. |
| **Technical Definition** | `Microsoft.AspNetCore.Mvc.Controller.File(...)` has overloads accepting `byte[]`, `Stream`, absolute file path (as `string`), or virtual path (via `PhysicalFile`/`VirtualFile`), plus a `contentType` and optional `fileDownloadName` (sets `Content-Disposition`). Range processing (via `enableRangeProcessing`) supports resumable downloads and media streaming. |
| **Beginner-Friendly Explanation** | `File()` means: “Send a **file** to the client.” The file could be an image (rendered inline), a PDF (downloaded), a CSV (exported), or anything else. You pass the content and the MIME type, and the framework streams it to the browser. |

### Purposes (each begins with “To”)

- **To** return a file (image, PDF, CSV, ZIP, etc.) to the client.
- **To** control the `Content-Type` and whether the browser displays or downloads it (via `Content-Disposition`).
- **To** stream large files efficiently (avoid loading fully into memory via `FileStreamResult` or `PhysicalFileResult`).
- **To** support HTTP range requests for resumable downloads and media playback.

### Syntax Rules and Structure

#### General Syntax

```csharp
return File(bytes, "application/pdf");                          // FileContentResult
return File(bytes, "application/pdf", "report.pdf");            // Downloadable
return File(stream, "application/octet-stream", "data.bin");    // FileStreamResult
return PhysicalFile(@"C:\file.pdf", "application/pdf");         // PhysicalFileResult
return File("~/files/report.pdf", "application/pdf");           // VirtualFileResult
```

#### Component Breakdown

| Parameter | Description |
|-----------|-------------|
| content | `byte[]`, `Stream`, or `string` path. |
| `contentType` | MIME type (e.g., `"application/pdf"`, `"text/csv"`). |
| `fileDownloadName` (optional) | Sets `Content-Disposition` attachment filename. |
| `enableRangeProcessing` (optional) | Supports HTTP 206 partial content. |

#### Syntax Rules

- **Without `fileDownloadName`** – browser typically displays inline (for renderable types).
- **With `fileDownloadName`** – browser prompts download with the specified filename.
- **Prefer stream/path overloads for large files** – avoids loading into memory.

#### Constraints and Limitations

- **Security** – never pass user-supplied file paths to `PhysicalFile`; sanitize with `Path.GetFullPath` and directory checks.
- **Range processing** – must be explicitly enabled; otherwise, clients cannot resume.
- **Content-Type spoofing** – mismatched content type may be rejected by browsers (e.g., `application/octet-stream` for scripts).

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Inline Image

```csharp
public IActionResult Logo()
{
    var bytes = System.IO.File.ReadAllBytes("wwwroot/images/logo.png");
    return File(bytes, "image/png");
}
```

**Expected Output:** `200 OK`, `Content-Type: image/png`, body = PNG bytes; browser displays inline.

**Why this output?** No `fileDownloadName` is supplied, so the browser renders the image.

---

#### Example 2: Downloadable CSV

```csharp
public IActionResult ExportCsv()
{
    var stream = _csvService.GenerateReport();
    return File(stream, "text/csv", "report.csv");
}
```

**Expected Output:** `200 OK`, `Content-Disposition: attachment; filename=report.csv`.

**Why this output?** `fileDownloadName` triggers attachment semantics; the browser saves the file.

---

#### Example 3: Physical File with Range Support

```csharp
public IActionResult Video()
{
    var path = Path.Combine(env.WebRootPath, "media", "intro.mp4");
    return PhysicalFile(path, "video/mp4", enableRangeProcessing: true);
}
```

**Expected Output:** Video streams with HTTP range support (206 partial content for seeks).

**Why this output?** `enableRangeProcessing: true` allows clients to request partial byte ranges for seeking and resuming.

### Real-World Cases

- **Image/asset serving** – logos, avatars, thumbnails.
- **Document exports** – PDF invoices, CSV reports.
- **Media streaming** – video/audio with range support.
- **File downloads** – downloads for user-generated content.

### References

- `FileResult` API – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.fileresult
- File responses in ASP.NET Core – https://learn.microsoft.com/en-us/aspnet/core/mvc/controllers/actions

---

## 9. `Content()`

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | `Content()` returns a `ContentResult` that writes a **plain string** to the response body with a specified content type. |
| **Technical Definition** | `Microsoft.AspNetCore.Mvc.Controller.Content(string content, string contentType)` (and overload with encoding) constructs a `ContentResult` with `Content`, `ContentType` (default `"text/plain; charset=utf-8"`), and optional `StatusCode`. The string is written directly to the response body without view rendering or serialization. |
| **Beginner-Friendly Explanation** | `Content()` means: “Send back a **plain string**.” Great for simple text responses, small HTML fragments, or CSV lines. It skips views and JSON—just text. |

### Purposes (each begins with “To”)

- **To** return a plain text or HTML fragment response directly.
- **To** specify a custom content type (`text/html`, `text/plain`, `text/csv`).
- **To** avoid the overhead of view rendering for trivial string responses.
- **To** support lightweight endpoints like health checks or simple pings.

### Syntax Rules and Structure

#### General Syntax

```csharp
return Content("Hello, world!");                              // text/plain, 200
return Content("<b>Bold</b>", "text/html");                   // text/html, 200
return new ContentResult { Content = "Oops", StatusCode = 400 };
```

#### Component Breakdown

| Parameter | Description |
|-----------|-------------|
| `content` | String content. |
| `contentType` | MIME type (default `text/plain; charset=utf-8`). |
| `statusCode` (via `ContentResult`) | HTTP status override. |

#### Syntax Rules

- **Directly writes** the string; no escaping/serialization is applied.
- **Use `Content` for small responses** – for large content, use `File`.

#### Constraints and Limitations

- **No encoding validation** – caller must escape correctly when `text/html`.
- **No content negotiation** – always returns the supplied content type.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Simple Text

```csharp
public IActionResult Ping() => Content("pong");
```

**Expected Output:** `200 OK`, `Content-Type: text/plain; charset=utf-8`, body `pong`.

**Why this output?** `ContentResult` uses the default text content type.

---

#### Example 2: HTML Fragment

```csharp
public IActionResult Badge() =>
    Content("<span class='badge'>New</span>", "text/html");
```

**Expected Output:** `200 OK`, `Content-Type: text/html`, body `<span class='badge'>New</span>`.

**Why this output?** The custom content type tells the browser to interpret the content as HTML.

---

#### Example 3: Custom Status

```csharp
public IActionResult Teapot() =>
    new ContentResult { Content = "I'm a teapot", ContentType = "text/plain", StatusCode = 418 };
```

**Expected Output:** `418 I'm a teapot` with text body.

**Why this output?** The `ContentResult` explicitly sets a non-standard status code.

### Real-World Cases

- **Health checks** – return `"ok"` in plain text.
- **AJAX HTML fragments** – return simple HTML snippets.
- **Text exports** – return simple line-delimited text.

### References

- `ContentResult` API – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.contentresult

---

## 10. `StatusCode()`

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | `StatusCode()` returns a `StatusCodeResult` that writes a **custom HTTP status code** with no body (or an `ObjectResult` with body when data is supplied). |
| **Technical Definition** | `Microsoft.AspNetCore.Mvc.Controller.StatusCode(int statusCode)` returns a `StatusCodeResult` with the given status. The overload `StatusCode(int statusCode, object value)` returns an `ObjectResult` containing the value. `StatusCodeResult` sets `HttpContext.Response.StatusCode`; `ObjectResult` additionally serializes the value using the configured formatter. |
| **Beginner-Friendly Explanation** | `StatusCode()` means: “Send back **this specific HTTP status code**.” Use it for custom codes (418, 429, 503) or when the status itself is the whole message. With a value argument, it also sends a body. |

### Purposes (each begins with “To”)

- **To** return a custom or non-standard HTTP status code.
- **To** signal conditions not covered by specific helper methods (e.g., 429 Too Many Requests, 503 Service Unavailable).
- **To** attach a body to a custom status via the `value` overload.
- **To** support API clients that key off status codes only.

### Syntax Rules and Structure

#### General Syntax

```csharp
return StatusCode(418);                            // StatusCodeResult, empty body
return StatusCode(429, new { retryAfter = 30 });   // ObjectResult with body
```

#### Component Breakdown

| Parameter | Description |
|-----------|-------------|
| `statusCode` | Integer HTTP status code. |
| `value` (optional) | Body object, serialized using the configured formatter. |

#### Syntax Rules

- **Use specific helpers where possible** (`NotFound`, `BadRequest`, `Ok`) for clarity.
- **`StatusCode(status, value)`** produces `ObjectResult` — the body is serialized by content negotiation.

#### Constraints and Limitations

- **No standard error shape** when used without a value; clients may not know how to interpret the response.
- **Range/validation** – clients may reject unusual codes or headers.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Custom Status Without Body

```csharp
public IActionResult Teapot() => StatusCode(418);
```

**Expected Output:** `418 I'm a teapot` with empty body.

**Why this output?** `StatusCodeResult` writes only the status code.

---

#### Example 2: Rate-Limited Response with Body

```csharp
public IActionResult RateLimited() =>
    StatusCode(429, new { retryAfter = 30, reason = "Too many requests" });
```

**Expected Output:** `429 Too Many Requests` with JSON body containing retry information.

**Why this output?** The `ObjectResult` serializes the anonymous type and writes it with 429 status.

### Real-World Cases

- **Rate limiting** – return 429.
- **Service unavailable** – return 503 with maintenance info.
- **Custom codes** – 418, 451, etc., for special scenarios.

### References

- `StatusCodeResult` API – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.statuscoderesult

---

## 11. `NotFound()`

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | `NotFound()` returns a `NotFoundResult` that responds with **HTTP 404 Not Found** (empty body) or `NotFoundObjectResult` (with a body) if a value is supplied. |
| **Technical Definition** | `Microsoft.AspNetCore.Mvc.ControllerBase.NotFound()` returns `NotFoundResult`, setting `Response.StatusCode = 404`. The overload `NotFound(object value)` returns `NotFoundObjectResult`, serializing the value with content negotiation (typically JSON in APIs). |
| **Beginner-Friendly Explanation** | `NotFound()` means: “**This thing doesn't exist**.” Use it when a requested resource can't be found—missing product, deleted user, unknown ID. This produces a 404, telling the client the resource is gone. |

### Purposes (each begins with “To”)

- **To** signal that the requested resource does not exist (404).
- **To** follow REST semantics for missing resources.
- **To** optionally return diagnostic details via `NotFound(object)` (e.g., `{ "error": "Product 42 not found" }`).
- **To** enable clean fallback patterns in actions (`if (x == null) return NotFound();`).

### Syntax Rules and Structure

#### General Syntax

```csharp
return NotFound();                                     // 404, empty body
return NotFound(new { error = "Not found" });          // 404 with body
```

#### Component Breakdown

| Parameter | Description |
|-----------|-------------|
| `value` (optional) | Body object for diagnostic details. |

#### Syntax Rules

- **Prefer `NotFound()` in APIs** – return 404 for missing resources.
- **`NotFound(object)`** – sends a body with content negotiation (JSON).

#### Constraints and Limitations

- **Don't hide real errors as 404** – 500 or specific 4xx codes may be more appropriate.
- **Don't use for authorization failures** – use 401/403 (`Unauthorized`, `Forbid`).

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

**Expected Output:** `404 Not Found` for missing IDs; 200 OK with JSON when found.

**Why this output?** `NotFound()` maps to the standard 404 status for missing resources.

---

#### Example 2: Diagnostic 404

```csharp
public IActionResult Details(int id)
{
    var product = _service.GetById(id);
    if (product == null) return NotFound(new { error = $"Product {id} not found" });
    return View(product);
}
```

**Expected Output:** `404 Not Found` with JSON body when missing.

**Why this output?** `NotFound(object)` serializes the value using content negotiation.

### Real-World Cases

- **REST lookups** – missing resources.
- **MVC pages** – friendly 404 pages (typically rendered by status code middleware).
- **Conditional APIs** – 404 for unknown IDs.

### References

- `NotFoundResult` API – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.notfoundresult

---

## 12. `BadRequest()`

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | `BadRequest()` returns a `BadRequestResult` (400 empty body) or `BadRequestObjectResult` (400 with body) indicating **client input is invalid**. |
| **Technical Definition** | `Microsoft.AspNetCore.Mvc.ControllerBase.BadRequest()` returns `BadRequestResult` (400). `BadRequest(object)` returns `BadRequestObjectResult`, serializing the value. `BadRequest(ModelState)` returns a `BadRequestObjectResult` with validation details; `ValidationProblem(ModelState)` produces standard Problem Details (RFC 7807) as `application/problem+json`. |
| **Beginner-Friendly Explanation** | `BadRequest()` means: “**The client sent bad data**.” Use it when the request is malformed or invalid—missing required fields, wrong types, broken JSON. Returns 400 to signal client error. |

### Purposes (each begins with “To”)

- **To** signal that the client’s input is invalid (400).
- **To** return validation errors via `BadRequest(ModelState)` or `ValidationProblem(ModelState)`.
- **To** produce standard Problem Details responses for API clients.
- **To** follow HTTP semantics for client errors.

### Syntax Rules and Structure

#### General Syntax

```csharp
return BadRequest();                                        // 400 empty
return BadRequest("Invalid payload");                       // 400 string body
return BadRequest(ModelState);                              // 400 with validation details
return ValidationProblem(ModelState);                       // 400 application/problem+json
```

#### Component Breakdown

| Helper | Body | When to Use |
|--------|------|-------------|
| `BadRequest()` | None | Minimal error signal. |
| `BadRequest(object)` | Custom | Custom error payload. |
| `ValidationProblem(ModelState)` | Problem Details | Standard validation response. |

#### Syntax Rules

- **Prefer `ValidationProblem`** for consistent RFC 7807 responses.
- **`[ApiController]` auto-returns 400** for invalid `ModelState` unless you customise.

#### Constraints and Limitations

- **Not for auth** – use `Unauthorized` (401) or `Forbid` (403).
- **Not for server errors** – use `Problem` (500) or exception middleware.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Validation Error

```csharp
[HttpPost]
public ActionResult<ProductDto> Create(CreateProductDto dto)
{
    if (!ModelState.IsValid) return ValidationProblem(ModelState);
    var created = _service.Create(dto);
    return CreatedAtAction(nameof(GetById), new { id = created.Id }, created);
}
```

**Expected Output:** `400 Bad Request` with `application/problem+json` field errors.

**Why this output?** `ValidationProblem` produces the standard Problem Details shape.

---

#### Example 2: Custom Bad Request

```csharp
public IActionResult Parse(string data)
{
    if (string.IsNullOrWhiteSpace(data))
        return BadRequest(new { error = "data is required" });
    return Ok(data);
}
```

**Expected Output:** `400 Bad Request` with JSON body containing the error.

**Why this output?** `BadRequest(object)` serializes the value; clients get a structured error.

### Real-World Cases

- **API validation** – consistent error responses.
- **Malformed JSON** – `[ApiController]` auto-400s.
- **Query string issues** – bad types or missing fields.

### References

- `BadRequestResult` API – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.badrequestresult
- Handle errors in ASP.NET Core web APIs – https://learn.microsoft.com/en-us/aspnet/core/web-api/handle-errors

---

## 13. `Ok()`

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | `Ok()` returns `OkResult` (200 **without body**) or `OkObjectResult` (200 **with body**) indicating success. |
| **Technical Definition** | `Microsoft.AspNetCore.Mvc.ControllerBase.Ok()` → `OkResult`. `Ok(object value)` → `OkObjectResult`, serializing `value` using content negotiation (JSON in APIs). In MVC view controllers, `Ok()` is uncommon; use `View()` for pages and `Ok()` for lightweight acknowledgements. |
| **Beginner-Friendly Explanation** | `Ok()` means: “**Success**.” Without arguments, it returns 200 with no body (useful for status-only responses). With an object, it returns 200 with that object serialized (JSON in APIs). |

### Purposes (each begins with “To”)

- **To** signal success (200).
- **To** return a response body (via `Ok(object)`).
- **To** provide a bodyless 200 for command or status-only responses.
- **To** support typed returns in APIs (`return product;` implicitly wraps as `OkObjectResult`).

### Syntax Rules and Structure

#### General Syntax

```csharp
return Ok();                       // 200 empty
return Ok(data);                   // 200 with serialized body
return Ok(new { status = "ok" });  // 200 anonymous
```

#### Component Breakdown

| Parameter | Description |
|-----------|-------------|
| `value` (optional) | Any serializable object. |

#### Syntax Rules

- **`Ok(object)`** — body serialized via content negotiation.
- **`Ok()`** — no body; use for status-only confirmations.

#### Constraints and Limitations

- **No body for `Ok()`** – if the client expects data, use `Ok(data)` or return POCO.
- **Don't confuse `Ok` with `Created`** — use 201 for resource creation.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Data Response

```csharp
public ActionResult<IEnumerable<ProductDto>> GetAll()
{
    return Ok(_service.GetAll());
}
```

**Expected Output:** `200 OK` with JSON array.

**Why this output?** `Ok(data)` serializes the collection as JSON with 200 status.

---

#### Example 2: Bodyless Success

```csharp
public IActionResult Health() => Ok();
```

**Expected Output:** `200 OK` with empty body.

**Why this output?** `Ok()` produces `OkResult` — status only.

---

#### Example 3: Implicit OK from Typed Return

```csharp
public ActionResult<ProductDto> GetById(int id)
{
    var product = _service.GetById(id);
    if (product == null) return NotFound();
    return product;   // Implicitly wrapped in OkObjectResult
}
```

**Expected Output:** `200 OK` with JSON when found; 404 otherwise.

**Why this output?** `ActionResult<T>` implicitly converts `T` to `OkObjectResult`.

### Real-World Cases

- **GET endpoints** – return data with 200.
- **Health checks** – return 200 with no body.
- **Command acknowledgements** – signal success without payload.

### References

- `OkResult` API – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.okresult
- `OkObjectResult` API – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.okobjectresult

---

## 14. `Created()`

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | `Created()` returns a `CreatedResult` (or `CreatedAtActionResult`/`CreatedAtRouteResult` via `CreatedAtAction`/`CreatedAtRoute`) that responds with **HTTP 201 Created**, sets the **`Location` header**, and includes the resource body. |
| **Technical Definition** | `Microsoft.AspNetCore.Mvc.ControllerBase.Created(string uri, object value)` → `CreatedResult` (explicit URI). `CreatedAtAction(string actionName, object routeValues, object value)` → `CreatedAtActionResult` (URI from action). `CreatedAtRoute(string routeName, object routeValues, object value)` → `CreatedAtRouteResult` (URI from named route). All produce 201 and set `Location` to the canonical URI of the newly created resource. |
| **Beginner-Friendly Explanation** | `Created()` means: “**I made a new thing**.” After a successful POST that creates a resource, return `Created(...)` with the URL where the resource can be found. The client learns the resource's URL from the `Location` header and the created data from the body. |

### Purposes (each begins with “To”)

- **To** signal successful resource creation (201) per REST semantics.
- **To** provide a `Location` header pointing to the canonical URL of the new resource.
- **To** return the created resource representation in the body.
- **To** leverage URL generation via `CreatedAtAction`/`CreatedAtRoute` so the location stays valid as routes change.

### Syntax Rules and Structure

#### General Syntax

```csharp
return Created("/api/products/42", dto);                     // Explicit URI
return CreatedAtAction(nameof(GetById), new { id = 42 }, dto); // URI from action
return CreatedAtRoute("GetProduct", new { id = 42 }, dto);    // URI from named route
```

#### Component Breakdown

| Helper | Location source |
|--------|-----------------|
| `Created(uri, value)` | Explicit string URI. |
| `CreatedAtAction(action, values, value)` | URL generated via action name. |
| `CreatedAtRoute(routeName, values, value)` | URL generated via named route. |

#### Syntax Rules

- **Always set `Location`** — the HTTP 201 contract requires it.
- **Prefer `CreatedAtAction`/`CreatedAtRoute`** over hard-coded URIs.

#### Constraints and Limitations

- **Hard-coded URIs break** when routes change.
- **Location header must be valid** — clients may follow it.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: `CreatedAtAction`

```csharp
[HttpPost]
public ActionResult<ProductDto> Create(CreateProductDto dto)
{
    var created = _service.Create(dto);
    return CreatedAtAction(nameof(GetById), new { id = created.Id }, created);
}
```

**Expected Output:** `201 Created`, `Location: /api/Products/42`, body = created product JSON.

**Why this output?** `CreatedAtAction` generates the `Location` from the named action and route values.

---

#### Example 2: `CreatedAtRoute`

```csharp
[HttpPost]
public IActionResult Create(CreateProductDto dto)
{
    var created = _service.Create(dto);
    return CreatedAtRoute("GetProduct", new { id = created.Id }, created);
}
```

**Expected Output:** `201 Created` with `Location` from the named route.

**Why this output?** Named route guarantees the URL even if the controller/action names change.

---

#### Example 3: Explicit `Created`

```csharp
[HttpPost]
public IActionResult Create(CreateProductDto dto)
{
    var created = _service.Create(dto);
    return Created($"/api/products/{created.Id}", created);
}
```

**Expected Output:** `201 Created`, `Location: /api/products/42`.

**Why this output?** Explicit URI is used as the `Location` header; less resilient to route changes but still valid.

### Real-World Cases

- **REST resource creation** – POST endpoints.
- **File uploads** – return the URL of the uploaded resource.
- **Order creation** – return the order URL.

### References

- `CreatedResult` API – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.createdresult
- `CreatedAtAction` API – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.controllerbase.createdataction
- `CreatedAtRoute` API – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.controllerbase.createdatroute

---

## 15. `NoContent()`

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | `NoContent()` returns a `NoContentResult` that responds with **HTTP 204 No Content** — success with **no body**. |
| **Technical Definition** | `Microsoft.AspNetCore.Mvc.ControllerBase.NoContent()` returns `NoContentResult`, which sets `Response.StatusCode = 204`. Per HTTP semantics, 204 responses carry no body; clients interpret 204 as “success, nothing else to say.” |
| **Beginner-Friendly Explanation** | `NoContent()` means: “**Done — nothing to return**.” Use it after successful updates or deletes where there’s no resource body to send back. The client knows it succeeded from the 204 status. |

### Purposes (each begins with “To”)

- **To** signal success (204) after a mutation with no payload.
- **To** follow REST semantics for updates/deletes.
- **To** save bandwidth by omitting a body when nothing is needed.
- **To** differentiate from `Ok()` (which allows a body).

### Syntax Rules and Structure

#### General Syntax

```csharp
return NoContent();                        // 204, no body
```

#### Constraints and Limitations

- **No body allowed** — 204 is defined without a body; don't try to attach one.
- **Use `Created` (201)** when you create a resource; 204 is for updates/deletes and other no-body successes.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Update Success

```csharp
[HttpPut("{id:int}")]
public IActionResult Update(int id, ProductDto dto)
{
    if (!_service.Exists(id)) return NotFound();
    _service.Update(id, dto);
    return NoContent();
}
```

**Expected Output:** `204 No Content` on success; 404 if missing.

**Why this output?** 204 signals “update succeeded” without a body.

---

#### Example 2: Delete Success

```csharp
[HttpDelete("{id:int}")]
public IActionResult Delete(int id)
{
    if (!_service.Exists(id)) return NotFound();
    _service.Delete(id);
    return NoContent();
}
```

**Expected Output:** `204 No Content` on success.

**Why this output?** Standard REST pattern for successful deletion.

### Real-World Cases

- **PUT/DELETE endpoints** — return 204 on success.
- **Bulk operations** — signal success without payload.
- **CQRS commands** — commands often return 204.

### References

- `NoContentResult` API – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.nocontentresult

---

## 16. Action Result Best Practices

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Action result best practices are guidelines for choosing and using action results that are **semantically correct, testable, and maintainable**. |
| **Technical Definition** | Best practices include: returning the result that matches the HTTP semantic outcome, preferring strongly typed APIs (`ActionResult<T>`), using named routes/actions for URL generation (`CreatedAtAction`, `RedirectToAction`), avoiding `Redirect` with untrusted URLs (open redirect risk), not returning `null`, using `NoContent`/`Created` appropriately, and choosing helpers that are testable. |
| **Beginner-Friendly Explanation** | Best practices are the **rules of thumb** for action results. For example: return `NotFound` for missing resources, `Created` for new ones, `NoContent` after successful updates/deletes, and never return `null`. Use `CreatedAtAction` instead of hard-coded URLs, and prefer `ActionResult<T>` in APIs for type safety. |

### Purposes (each begins with “To”)

- **To** produce semantically correct HTTP responses (200, 201, 204, 400, 404).
- **To** keep results testable by returning inspectable objects.
- **To** avoid security issues like open redirects and information leakage.
- **To** provide accurate OpenAPI documentation through typed returns.
- **To** keep URL generation robust via named routes/actions.

### Guideline Checklist

| Guideline | Rationale | Example |
|-----------|-----------|---------|
| **Match the helper to the semantic outcome** | HTTP status codes carry meaning. | 201 → `CreatedAtAction`; 204 → `NoContent`; 404 → `NotFound`. |
| **Use `ActionResult<T>` in APIs** | Type-safety plus multiple outcomes. | `public ActionResult<ProductDto> Get(int id)` |
| **Never return `null`** | The framework can't execute `null`. | Always return a concrete `IActionResult`. |
| **Prefer `CreatedAtAction`/`CreatedAtRoute` over `Created(uri)`** | URL generation stays correct if routes change. | `CreatedAtAction(nameof(GetById), ...)` |
| **Prefer `RedirectToAction`/`RedirectToRoute` over `Redirect(url)`** | Avoid hard-coded URLs. | `RedirectToAction(nameof(Index))` |
| **Use `LocalRedirect` for user-supplied URLs** | Prevents open redirects. | `if (Url.IsLocalUrl(returnUrl)) return LocalRedirect(returnUrl);` |
| **Use `ValidationProblem` for validation errors** | Standard Problem Details (RFC 7807). | `if (!ModelState.IsValid) return ValidationProblem(ModelState);` |
| **Use `NoContent` for updates/deletes** | Matches REST semantics. | `_service.Delete(id); return NoContent();` |
| **Avoid `StatusCode(200)`** | Use `Ok()` for readability. | `return Ok(data);` instead of `StatusCode(200, data);` |
| **Don't return `Ok()` for created resources** | Use 201 with Location. | `CreatedAtAction(...)` instead of `Ok()`. |
| **Stream large files** | Avoid loading into memory. | `File(stream, "application/pdf", "file.pdf")` |
| **Sanitise user-supplied paths** | Prevent directory traversal. | Check `Path.GetFullPath` is within allowed root. |

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Well-Designed Controller

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
        if (product == null) return NotFound();
        return product;
    }

    [HttpPost]
    public ActionResult<ProductDto> Create(CreateProductDto dto)
    {
        if (!ModelState.IsValid) return ValidationProblem(ModelState);
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

| Endpoint | Result |
|----------|--------|
| GET found | 200 JSON |
| GET missing | 404 |
| POST valid | 201 with Location |
| POST invalid | 400 Problem Details |
| PUT success | 204 |
| DELETE success | 204 |

**Why this output?** Each action returns the helper that matches its semantic outcome; the API is testable and consistent.

---

#### Example 2: Safe Redirect After Login

```csharp
[HttpPost]
public IActionResult Login(LoginDto dto, string? returnUrl)
{
    if (!_auth.SignIn(dto)) return Unauthorized();
    if (returnUrl != null && Url.IsLocalUrl(returnUrl))
        return LocalRedirect(returnUrl);
    return RedirectToAction("Index", "Home");
}
```

**Expected Output:**

| Scenario | Result |
|----------|--------|
| Valid credentials, local returnUrl | 302 to `returnUrl` |
| Valid credentials, external returnUrl | 302 to Home |
| Invalid credentials | 401 Unauthorized |

**Why this output?** `LocalRedirect` prevents open redirect attacks; `Url.IsLocalUrl` guards user input.

---

#### Example 3: File Download with Sanitised Path

```csharp
public IActionResult Download(string fileName)
{
    var safeName = Path.GetFileName(fileName);
    var fullPath = Path.GetFullPath(Path.Combine("uploads", safeName));
    if (!fullPath.StartsWith(Path.GetFullPath("uploads")))
        return BadRequest("Invalid file path");
    if (!System.IO.File.Exists(fullPath)) return NotFound();
    return PhysicalFile(fullPath, "application/octet-stream", safeName);
}
```

**Expected Output:**

| Scenario | Result |
|----------|--------|
| Valid file | 200 with file content |
| Path traversal attempt | 400 Bad Request |
| Missing file | 404 Not Found |

**Why this output?** Sanitising the filename prevents directory traversal; missing files yield `NotFound`.

### Real-World Cases

- **Public REST APIs** — semantically correct status codes ease client implementation.
- **MVC web apps** — PRG pattern with `RedirectToAction`.
- **File endpoints** — safe path handling with `PhysicalFile`/`File`.

### References

- Controller action return types in ASP.NET Core web API – https://learn.microsoft.com/en-us/aspnet/core/web-api/action-return-types
- Handle errors in ASP.NET Core web APIs – https://learn.microsoft.com/en-us/aspnet/core/web-api/handle-errors
- Prevent open redirects – https://learn.microsoft.com/en-us/aspnet/core/security/preventing-open-redirects
- URL generation in ASP.NET Core – https://learn.microsoft.com/en-us/aspnet/core/fundamentals/routing#url-generation
- File uploads and downloads in ASP.NET Core – https://learn.microsoft.com/en-us/aspnet/core/mvc/models/file-uploads

---

## Summary Table: Action Results at a Glance

| Helper | Concrete Result | Status | Body | Typical Use |
|--------|-----------------|--------|------|-------------|
| **`View()`** | `ViewResult` | 200 | HTML page | MVC pages |
| **`PartialView()`** | `PartialViewResult` | 200 | HTML fragment | AJAX / partials |
| **`Json()`** | `JsonResult` | 200 (or set) | JSON | MVC controllers returning JSON |
| **`Redirect()`** | `RedirectResult` | 301/302 | None | External URLs |
| **`RedirectToAction()`** | `RedirectToActionResult` | 301/302 | None | Redirect to action |
| **`RedirectToRoute()`** | `RedirectToRouteResult` | 301/302 | None | Redirect to named route |
| **`File()`** | `File*Result` | 200 | File | Downloads / streams |
| **`Content()`** | `ContentResult` | 200 (or set) | Text/HTML | Plain-text responses |
| **`StatusCode()`** | `StatusCodeResult` / `ObjectResult` | Custom | None / body | Custom codes |
| **`NotFound()`** | `NotFoundResult` / `NotFoundObjectResult` | 404 | None / body | Missing resources |
| **`BadRequest()`** | `BadRequestResult` / `BadRequestObjectResult` | 400 | None / body | Invalid input |
| **`Ok()`** | `OkResult` / `OkObjectResult` | 200 | None / body | Success |
| **`Created()`** | `CreatedResult` / `CreatedAtActionResult` / `CreatedAtRouteResult` | 201 | Created body | Resource creation |
| **`NoContent()`** | `NoContentResult` | 204 | None | Update/delete success |

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
- File uploads and downloads in ASP.NET Core – https://learn.microsoft.com/en-us/aspnet/core/mvc/models/file-uploads
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
- `OkObjectResult` API – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.okobjectresult
- `CreatedResult` API – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.createdresult
- `CreatedAtAction` API – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.controllerbase.createdataction
- `CreatedAtRoute` API – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.controllerbase.createdatroute
- `NoContentResult` API – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.nocontentresult