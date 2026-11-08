# View Data in ASP.NET Core MVC: A Comprehensive Cheat Sheet

> **Version Scope & Compatibility Note**
> This cheat sheet targets **ASP.NET Core MVC** (3.0 and later, including .NET 6/7/8/9/10/11). `ViewData` and `ViewBag` are **weakly typed, runtime-resolved** mechanisms for passing small amounts of data from a controller to a view within a **single request**. `TempData` is a **short-lived, cross-request** mechanism primarily used for redirect scenarios (e.g., flash messages). All three are supported but are **not the recommended primary data-passing strategy**; strongly typed **ViewModels** should be preferred wherever possible. `TempData` defaults to a **cookie-based provider** since ASP.NET Core 2.0; a session-based provider is opt-in via `AddSessionStateTempDataProvider()`.

---

## Topic Overview

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | View data mechanisms (`ViewData`, `ViewBag`, `TempData`) are lightweight, framework-provided containers for passing **small, transient amounts of data** between a controller and a view, or across a redirect, without relying on a strongly typed model. |
| **Technical Definition** | `ViewData` is a `ViewDataDictionary` (a string-keyed dictionary of `object` values) exposed on `Controller` and `RazorPage`. `ViewBag` is a `DynamicViewData` wrapper around the **same underlying `ViewData` collection**, exposing dynamic properties (`@ViewBag.Key`). Both are **per-request** and become inaccessible after a redirect. `TempData` is an `ITempDataDictionary` backed by a **cookie-based provider** (default) or a session-state provider, designed to survive **exactly one subsequent request** (e.g., across a redirect) and is then automatically cleared unless explicitly preserved with `Peek()` or `Keep()`. |
| **Beginner-Friendly Explanation** | `ViewData` and `ViewBag` are like **sticky notes** attached to a single request: the controller writes a note, and the view reads it. The note is discarded as soon as the request finishes (including after a redirect). `TempData` is like a **message that survives one redirect**: it’s useful for showing “Record saved successfully” after a POST-Redirect-GET, but it’s not meant for carrying application data. |

### Key Characteristics

- **Weakly typed** – `ViewData` requires casting for non-string types; `ViewBag` is `dynamic` with no compile-time checking.
- **Runtime-resolved** – Both are resolved at runtime; typos and type mismatches cause **runtime errors**, not compile-time errors.
- **Per-request (ViewData/ViewBag)** – Values are lost on redirect; they are scoped to the current request only.
- **Cross-request (TempData)** – Survives exactly one redirect by default; cleared automatically after being read.
- **Cookie-based by default (TempData)** – Encrypted, Base64Url-encoded, and chunked; total cookie size limit is **under 4096 bytes**.
- **Peek/Keep (TempData)** – `Peek()` reads without deleting; `Keep()` marks for retention across the next request.
- **Same underlying collection (ViewData/ViewBag)** – `ViewData["X"]` and `ViewBag.X` refer to the same value; mixing is possible but discouraged for readability.

### Prerequisites

- **.NET SDK** (6.0+ recommended; examples use .NET 8/10).
- **ASP.NET Core MVC** project with controllers and Razor views.
- Basic understanding of **controllers**, **views**, **Razor syntax**, and **redirects** (PRG pattern).
- Familiarity with **strongly typed ViewModels** (the recommended alternative).

### Related Programming Areas

- **ViewModels** – The strongly typed, recommended alternative for passing data to views.
- **TempData Providers** – Cookie-based (default) or session-state-based storage.
- **Session State** – A longer-lived, server-side (or distributed) store for per-user data.
- **PRG Pattern** – Post/Redirect/Get, where `TempData` is commonly used for flash messages.
- **Filters** – `ViewData` can be populated in action filters and consumed in views or layouts.

### Core Concepts / Features

1. **`ViewData`** – The dictionary-based, weakly typed container for per-request data.
2. **`ViewBag`** – The `dynamic` wrapper around `ViewData`.
3. **`TempData`** – The cross-request container for redirect scenarios, with `Peek`/`Keep`.
4. **View Data Best Practices** – Guidelines for choosing the right mechanism and avoiding common pitfalls.

---

## 1. `ViewData`

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | `ViewData` is a **string-keyed dictionary** of `object` values that passes data from a controller to a view within a single request. |
| **Technical Definition** | `ViewData` is an instance of `ViewDataDictionary`, exposed as a property on `Controller` and `RazorPage`. It derives from `ViewDataDictionary` and supports dictionary operations (`ContainsKey`, `Add`, `Remove`, `Clear`). Keys are strings (whitespace allowed), and values are `object`; non-string values **must be cast** in the view. It is populated in the controller and consumed in the view or layout, and is **discarded** at the end of the request. |
| **Beginner-Friendly Explanation** | `ViewData` is a **box with labeled compartments**. You put something in a compartment (`ViewData["Title"] = "Home"`), and the view takes it out (`@ViewData["Title"]`). Because the box holds `object`, you may need to tell the view what type to expect (cast it) when it’s not a string. The box is thrown away after the request. |

### Purposes (each begins with “To”)

- **To** pass **small amounts of data** from a controller to a view without a strongly typed model.
- **To** set simple values (e.g., page title, a single message) consumed by a layout or partial.
- **To** provide a dictionary-based interface (`ContainsKey`, `Add`, `Remove`) for conditional or dynamic data.
- **To** populate data from action filters that views can consume.

### Syntax Rules and Structure

#### General Syntax (Controller)

```csharp
ViewData["Title"] = "Home Page";
ViewData["Message"] = "Welcome!";
ViewData["ProductCount"] = 42;          // int stored as object
```

#### General Syntax (View)

```cshtml
@ViewData["Title"]
@ViewData["Message"]
@((int)ViewData["ProductCount"]!)       // cast required for non-string
```

#### Component Breakdown

| Component | Description |
|-----------|-------------|
| `ViewData["key"]` | Indexer access (string key). |
| Value type | `object`; casting required for non-string types. |
| Lifetime | Current request only; lost on redirect. |
| Availability | Controller, view, layout, partials. |

#### Syntax Rules

- **Keys are strings** and may contain whitespace.
- **Non-string values must be cast** in the view.
- **Null checks are advisable** – missing keys return `null`.
- **`ViewData` is not available in `PageModel`** (Razor Pages) by default; use `ViewData` on the page or `[ViewData]` properties.

#### Constraints and Limitations

- **No compile-time checking** – typos in keys or incorrect casts cause **runtime errors**.
- **Per-request only** – values are lost on redirect; use `TempData` for cross-request scenarios.
- **Not for complex data** – use a strongly typed ViewModel for anything beyond simple values.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Passing a Page Title via `ViewData`

```csharp
// Controller
public IActionResult Index()
{
    ViewData["Title"] = "Home Page";
    ViewData["WelcomeMessage"] = "Hello, ASP.NET Core!";
    return View();
}
```

```cshtml
@* View *@
<h1>@ViewData["Title"]</h1>
<p>@ViewData["WelcomeMessage"]</p>
```

**Step-by-step explanation:**

1. The controller stores `Title` and `WelcomeMessage` in `ViewData`.
2. The view retrieves them by key.
3. Both values are strings, so no casting is required.

**Expected Output:** `<h1>Home Page</h1><p>Hello, ASP.NET Core!</p>`

**Why this output?** `ViewData` keys map directly to the stored string values; the view outputs them.

---

#### Example 2: Storing and Casting a Non-String Value

```csharp
ViewData["ProductCount"] = 42;   // int boxed as object
```

```cshtml
@if (ViewData["ProductCount"] is int count)
{
    <p>Products: @count</p>
}
```

**Expected Output:** `<p>Products: 42</p>` (only when the value is an `int`).

**Why this output?** Pattern matching (`is int count`) safely unboxes the value without an explicit cast exception.

---

#### Example 3: Using `ViewData` in a Layout

```csharp
ViewData["Title"] = "Privacy Policy";
```

```cshtml
@* _Layout.cshtml *@
<title>@ViewData["Title"] - MyApp</title>
```

**Expected Output:** `<title>Privacy Policy - MyApp</title>`

**Why this output?** The layout reads the same `ViewData` collection populated by the action.

### Real-World Cases

- **Page titles** – set `ViewData["Title"]` in each action; the layout renders `<title>`.
- **Flash-style messages without redirects** – `ViewData["StatusMessage"]` for a success banner on the same request.
- **Filter-populated data** – an action filter sets `ViewData["Breadcrumbs"]` consumed by the layout.

### References

- Views in ASP.NET Core MVC: Weakly typed data (ViewData, ViewBag) – https://learn.microsoft.com/zh-tw/aspnet/core/mvc/views/overview?view=aspnetcore-11.0#VD_VB
- ViewData and ViewBag are equally valid for small amounts of data – https://learn.microsoft.com/zh-tw/aspnet/core/mvc/views/overview?view=aspnetcore-11.0#VD_VB

---

## 2. `ViewBag`

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | `ViewBag` is a **`dynamic` wrapper** around `ViewData` that exposes its values as **dynamic properties** (`@ViewBag.Key`) instead of dictionary keys. |
| **Technical Definition** | `ViewBag` is an instance of `DynamicViewData` (internal) that derives from `DynamicObject` and wraps the **same underlying `ViewDataDictionary`**. Property access (`ViewBag.Title`) maps to dictionary access (`ViewData["Title"]`). Because it is `dynamic`, member access is resolved at **runtime**, with no compile-time checking. `ViewBag` is **not available by default in Razor Pages `PageModel` classes**. |
| **Beginner-Friendly Explanation** | `ViewBag` is the **dot-notation version** of `ViewData`. Instead of `ViewData["Title"]`, you write `ViewBag.Title`. It’s shorter and cleaner, but because it’s `dynamic`, the compiler can’t catch typos—they only blow up at runtime. It’s the same data as `ViewData`, just accessed differently. |

### Purposes (each begins with “To”)

- **To** provide a **more readable, dot-notation syntax** for accessing `ViewData` values.
- **To** avoid explicit casting when storing and reading simple values (the dynamic type handles it).
- **To** simplify null checking with the null-conditional operator (`@ViewBag.Person?.Name`).
- **To** serve as a convenient, low-ceremony mechanism for small, ad-hoc data passing.

### Syntax Rules and Structure

#### General Syntax (Controller)

```csharp
ViewBag.Title = "Home Page";
ViewBag.ProductCount = 42;
ViewBag.Person = new { Name = "Alice", Age = 30 };
```

#### General Syntax (View)

```cshtml
@ViewBag.Title
@ViewBag.ProductCount
@ViewBag.Person?.Name
```

#### Component Breakdown

| Component | Description |
|-----------|-------------|
| `ViewBag.Property` | Dynamic property access (no cast required). |
| Underlying store | Same `ViewDataDictionary` as `ViewData`. |
| Lifetime | Current request only. |
| Null handling | `@ViewBag.Person?.Name` avoids `RuntimeBinderException`. |

#### Syntax Rules

- **Property names are case-insensitive** in `DynamicViewData` (via dictionary lookup).
- **No cast required** for reading, but the runtime type must match usage.
- **`ViewBag` and `ViewData` can be mixed** – both reference the same collection; however, consistency improves readability.
- **`ViewBag` is not available in `PageModel`** (Razor Pages) by default.

#### Constraints and Limitations

- **No compile-time checking** – typos and missing properties cause `RuntimeBinderException` at runtime.
- **No IntelliSense** for dynamic properties.
- **Per-request only** – lost on redirect.
- **Not for complex data** – use a ViewModel.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Basic `ViewBag` Usage

```csharp
ViewBag.Title = "Home Page";
ViewBag.WelcomeMessage = "Welcome to ASP.NET Core!";
```

```cshtml
<h1>@ViewBag.Title</h1>
<p>@ViewBag.WelcomeMessage</p>
```

**Expected Output:** `<h1>Home Page</h1><p>Welcome to ASP.NET Core!</p>`

**Why this output?** Dynamic property access resolves to the stored values without casting.

---

#### Example 2: Null-Safe Access with `?.`

```csharp
ViewBag.Person = null;   // or not set at all
```

```cshtml
<p>Name: @ViewBag.Person?.Name</p>
```

**Expected Output:** `<p>Name: </p>` (no exception; null-conditional short-circuits).

**Why this output?** `?.` prevents a `RuntimeBinderException` when `Person` is null.

---

#### Example 3: Mixing `ViewBag` and `ViewData`

```csharp
ViewData["Title"] = "Home";
ViewBag.Message = "Hello";
```

```cshtml
<h1>@ViewBag.Title</h1>   @* Reads the same value as ViewData["Title"] *@
<p>@ViewData["Message"]</p> @* Reads the same value as ViewBag.Message *@
```

**Expected Output:** `<h1>Home</h1><p>Hello</p>`

**Why this output?** Both mechanisms reference the same underlying `ViewDataDictionary`.

### Real-World Cases

- **Quick page metadata** – `ViewBag.Title` for the layout.
- **Small flags or counts** – `ViewBag.ShowBanner = true`.
- **Legacy codebases** – `ViewBag` is common in older ASP.NET Core MVC code.

### References

- ViewBag derives from `DynamicViewData` – https://learn.microsoft.com/zh-tw/aspnet/core/mvc/views/overview?view=aspnetcore-11.0#VD_VB
- ViewBag provides dynamic access to ViewData – https://learn.microsoft.com/zh-tw/aspnet/core/mvc/views/overview?view=aspnetcore-11.0#VD_VB

---

## 3. `TempData`

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | `TempData` is a **short-lived, cross-request container** that stores data until it is read (or explicitly retained), surviving exactly one subsequent request—typically a redirect. |
| **Technical Definition** | `TempData` is an `ITempDataDictionary` exposed on `Controller` and `RazorPage`. It is backed by a **cookie-based provider by default** (`CookieTempDataProvider`): data is encrypted with `IDataProtector`, Base64Url-encoded, and chunked, with a **total cookie size under 4096 bytes**. A **session-state provider** can be enabled via `AddSessionStateTempDataProvider()`. Values are **automatically removed** after being read in a subsequent request, unless `Peek()` is used (reads without deleting) or `Keep()` is called (retains across the next request). |
| **Beginner-Friendly Explanation** | `TempData` is a **message that survives one redirect**. When you save a record and redirect to a list page, you can put “Record saved!” in `TempData`, and the list page shows it once. After that, it’s gone—unless you `Peek` or `Keep` it. It’s perfect for flash messages, not for application data. |

### Purposes (each begins with “To”)

- **To** pass **flash messages** (success, error, status) across a redirect (PRG pattern).
- **To** survive **exactly one subsequent request**, then clear automatically.
- **To** support **Peek** (read without consuming) and **Keep** (retain for another request) when multiple reads are needed.
- **To** provide a **cookie-based default** that works in web farms without additional configuration (beyond Data Protection).
- **To** offer a **session-based provider** for larger payloads when session state is already in use.

### Syntax Rules and Structure

#### General Syntax (Controller)

```csharp
TempData["Message"] = "Record saved successfully!";
```

#### General Syntax (View)

```cshtml
@if (TempData["Message"] is string message)
{
    <div class="alert alert-success">@message</div>
}
```

#### Peek and Keep

```cshtml
@* Peek: read without deleting at the end of the request *@
<p>@TempData.Peek("Message")</p>

@* Keep: retain the value for the next request *@
@{
    TempData.Keep("Message");
}
```

#### Component Breakdown

| Member | Behaviour |
|--------|-----------|
| `TempData["key"]` | Reads and **removes** the value at the end of the request. |
| `TempData.Peek("key")` | Reads **without** removing; survives to the next request. |
| `TempData.Keep("key")` | Marks the key for **retention** across the next request. |
| `TempData.Keep()` | Retains **all** keys. |

#### Syntax Rules

- **Default provider is cookie-based** – encrypted, Base64Url-encoded, chunked, <4096 bytes total.
- **Session-based provider** – enable via `AddSessionStateTempDataProvider()` and `AddSession()` + `UseSession()`.
- **Peek vs Keep** – `Peek` is a read operation; `Keep` is a retention instruction (often called after reading).
- **Values must be serializable** – the cookie provider serializes to a cookie; the session provider serializes to session storage.

#### Constraints and Limitations

- **Cookie size limit** – keep payloads **small** (well under 4096 bytes after encryption/chunking); the cookie provider is not suitable for large objects.
- **Single redirect by default** – data is cleared after one subsequent request unless `Peek`/`Keep` is used.
- **Not for sensitive long-term data** – use session or a database for anything beyond transient flash messages.
- **Session provider requires session infrastructure** – `AddSession`, `UseSession`, and (in web farms) session affinity or a distributed cache.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Flash Message Across a Redirect (PRG Pattern)

```csharp
[HttpPost]
public async Task<IActionResult> Create(ProductDto dto)
{
    await _service.CreateAsync(dto);
    TempData["Message"] = "Product created successfully.";
    return RedirectToAction(nameof(Index));
}
```

```cshtml
@* Index view *@
@if (TempData["Message"] is string message)
{
    <div class="alert alert-success">@message</div>
}
```

**Expected Output:** After the redirect, the Index page displays “Product created successfully.” On the next request (e.g., refresh), the message is gone.

**Why this output?** `TempData` survives exactly one subsequent request; reading it in the Index view consumes it.

---

#### Example 2: `Peek` – Display Without Consuming

```cshtml
@if (TempData.Peek("Message") is string message)
{
    <div class="alert alert-info">@message</div>
}
```

**Expected Output:** The message is displayed, and it **remains** in `TempData` for the next request.

**Why this output?** `Peek` reads without marking the key for deletion at the end of the request.

---

#### Example 3: `Keep` – Retain for the Next Request

```cshtml
@if (TempData["Message"] is string message)
{
    <div class="alert alert-warning">@message</div>
}
@{
    TempData.Keep("Message");   // Retain for the next request
}
```

**Expected Output:** The message is displayed and **retained** for one more request.

**Why this output?** `Keep` marks the key for retention; without it, reading would schedule deletion.

---

#### Example 4: Enabling the Session-Based TempData Provider

```csharp
var builder = WebApplication.CreateBuilder(args);

builder.Services.AddControllersWithViews()
    .AddSessionStateTempDataProvider();   // Session-based TempData

builder.Services.AddSession();

var app = builder.Build();

app.UseSession();   // Required for session state
app.MapDefaultControllerRoute();
```

**Expected Output:** `TempData` is stored in session state instead of cookies, removing the cookie size limit (but requiring session infrastructure).

**Why this output?** The session provider serializes `TempData` into session storage; `AddSession` and `UseSession` are required.

### Real-World Cases

- **Success/error banners** – “Record saved,” “Invalid input,” “Item deleted” across a redirect.
- **Multi-step wizards** – carry a status message between steps.
- **One-time notifications** – display a message exactly once after an action.

### References

- Session and state management in ASP.NET Core: TempData – https://learn.microsoft.com/zh-tw/aspnet/core/fundamentals/app-state?view=aspnetcore-1.1#2
- TempData `Peek` and `Keep` – https://learn.microsoft.com/zh-tw/aspnet/core/fundamentals/app-state?view=aspnetcore-1.1#2
- Configuring the TempData provider – https://learn.microsoft.com/zh-tw/aspnet/core/fundamentals/app-state?view=aspnetcore-1.1#2
- Cookie-based TempData provider (default) – https://learn.microsoft.com/zh-tw/aspnet/core/fundamentals/app-state?view=aspnetcore-1.1#2

---

## 4. View Data Best Practices

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | View data best practices are guidelines for choosing between `ViewData`, `ViewBag`, and `TempData`, and for using them in ways that are **type-safe, maintainable, and appropriate** to their intended scope. |
| **Technical Definition** | Best practices include: preferring **strongly typed ViewModels** over weakly typed mechanisms, using `ViewData`/`ViewBag` only for **small, incidental data** (page titles, simple flags), using `TempData` **exclusively for cross-request flash messages**, keeping `TempData` payloads **small** (cookie limit), avoiding `TempData` for sensitive or large data, using `Peek`/`Keep` intentionally, and **never relying on runtime resolution** for critical data paths. |
| **Beginner-Friendly Explanation** | Best practices are the **rules of thumb** that keep view data clean and safe. The biggest rule: **use a ViewModel** whenever the data matters. Use `ViewData`/`ViewBag` only for tiny things like page titles, and use `TempData` only for “success!” messages after a redirect. Don’t put your product catalogue in `ViewBag`—you’ll regret it. |

### Purposes (each begins with “To”)

- **To** prefer **strongly typed ViewModels** for all non-trivial data, gaining compile-time safety and IntelliSense.
- **To** restrict `ViewData`/`ViewBag` to **small, incidental values** (titles, flags, simple messages) within a single request.
- **To** restrict `TempData` to **flash messages** and other short-lived, cross-request notifications.
- **To** keep `TempData` payloads **small** to stay within the cookie size limit.
- **To** use `Peek`/`Keep` deliberately rather than accidentally consuming or retaining data.
- **To** avoid mixing `ViewData` and `ViewBag` inconsistently, which hurts readability.

### Guideline Checklist

| Guideline | Rationale | Example |
|-----------|-----------|---------|
| **Prefer ViewModels for all non-trivial data** | Compile-time safety, IntelliSense, refactoring support. | `@model ProductDetailsViewModel` instead of `ViewBag.Product` |
| **Use `ViewData`/`ViewBag` only for small, incidental data** | They are weakly typed and runtime-resolved. | `ViewData["Title"]` for the layout `<title>`. |
| **Pick one (`ViewData` or `ViewBag`) and stay consistent** | Mixing hurts readability. | Use `ViewData["Title"]` everywhere, not `ViewBag.Title` in some views and `ViewData["Title"]` in others. |
| **Use `TempData` only for cross-request flash messages** | Designed for one redirect, not application data. | `TempData["Message"]` after PRG. |
| **Keep `TempData` payloads small** | Cookie-based default is <4096 bytes total. | Store short strings, not DTOs or lists. |
| **Never use `TempData` for sensitive or large data** | Cookie is client-side (encrypted but still on the wire); size limit. | Use session or a database for sensitive/large data. |
| **Use `Peek` to read without consuming** | Preserves the value for the next request. | `TempData.Peek("Message")` for a multi-display scenario. |
| **Use `Keep` to retain after reading** | Marks the key for retention. | `TempData.Keep("Message")` after reading. |
| **Check for null before using `ViewData`/`TempData`** | Missing keys return `null`. | `if (TempData["Message"] is string msg) { ... }` |
| **Avoid `ViewBag` in Razor Pages `PageModel`** | Not available by default. | Use `ViewData` or `[ViewData]` properties. |

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Recommended – ViewModel for Real Data, ViewData for Title

```csharp
// Controller
public IActionResult Details(int id)
{
    var product = _service.GetById(id);
    if (product == null) return NotFound();

    ViewData["Title"] = $"Details - {product.Name}";   // Small, incidental
    return View(new ProductDetailsViewModel { ... });   // Strongly typed
}
```

```cshtml
@model ProductDetailsViewModel
@{
    ViewData["Title"] = ViewData["Title"] ?? "Details";
}
<h1>@Model.Name</h1>
<p>@Model.FormattedPrice</p>
```

**Expected Output:** The view renders the strongly typed product details, with the page title supplied via `ViewData`.

**Why this output?** The ViewModel carries the real data; `ViewData` handles the incidental title.

---

#### Example 2: Recommended – `TempData` for Flash Message Only

```csharp
[HttpPost]
public async Task<IActionResult> Delete(int id)
{
    await _service.DeleteAsync(id);
    TempData["Message"] = "Product deleted.";
    return RedirectToAction(nameof(Index));
}
```

```cshtml
@if (TempData["Message"] is string msg)
{
    <div class="alert alert-success">@msg</div>
}
```

**Expected Output:** A one-time success banner after the redirect.

**Why this output?** `TempData` is used for its intended purpose—a short-lived flash message across a redirect.

---

#### Example 3: Discouraged – ViewBag for Structured Data

```csharp
// ❌ Discouraged: ViewBag for structured data
ViewBag.Products = _service.GetAll();
```

```cshtml
@* No compile-time checking; runtime failure if Products is missing *@
@foreach (var p in ViewBag.Products) { <p>@p.Name</p> }
```

**Expected Output:** Works if the data is present, but **fails at runtime** with a `RuntimeBinderException` if `Products` is missing or not enumerable.

**Why this output?** `ViewBag` is dynamic; there is no compile-time safety. Use `@model IEnumerable<Product>` instead.

---

#### Example 4: `Peek` for Multi-Display Flash Message

```cshtml
@if (TempData.Peek("Message") is string msg)
{
    <div class="alert alert-info">@msg</div>
}
```

**Expected Output:** The message is displayed and **remains** available for the next request.

**Why this output?** `Peek` reads without consuming; the value survives to the next request.

### Real-World Cases

- **E-commerce** – ViewModel for product details; `TempData` for “Added to cart” after redirect; `ViewData["Title"]` for the page title.
- **Admin panels** – ViewModel for forms; `TempData` for “User deleted”; `ViewData` for breadcrumb labels.
- **Blog** – ViewModel for posts; `TempData` for “Comment submitted” after redirect.

### References

- Both ViewData and ViewBag are equally valid for small amounts of data – https://learn.microsoft.com/zh-tw/aspnet/core/mvc/views/overview?view=aspnetcore-11.0#VD_VB
- Weakly typed data (ViewData, ViewBag) are dynamically resolved at runtime – https://learn.microsoft.com/zh-tw/aspnet/core/mvc/views/overview?view=aspnetcore-11.0#VD_VB
- Prefer ViewModel over ViewData/ViewBag (SSW Rule) – https://www.ssw.com.au/rules/do-you-use-viewmodel-instead-of-viewdata
- TempData `Peek` and `Keep` – https://learn.microsoft.com/zh-tw/aspnet/core/fundamentals/app-state?view=aspnetcore-1.1#2
- Cookie-based TempData size limit – https://learn.microsoft.com/zh-tw/aspnet/core/fundamentals/app-state?view=aspnetcore-1.1#2

---

## Summary Table: View Data at a Glance

| Mechanism | Type | Lifetime | Primary Use | Key Risk / Limitation |
|-----------|------|----------|-------------|----------------------|
| **`ViewData`** | `ViewDataDictionary` (string → object) | Current request | Small per-request data (titles, flags) | Casting required; runtime errors |
| **`ViewBag`** | `dynamic` wrapper over `ViewData` | Current request | Dot-notation access to per-request data | No compile-time checking; runtime errors |
| **`TempData`** | `ITempDataDictionary` (cookie or session) | One subsequent request (unless `Peek`/`Keep`) | Flash messages across redirects | Cookie size limit (<4096 bytes); not for large/sensitive data |
| **Best Practices** | – | – | Prefer ViewModels; restrict ViewData/ViewBag to small incidental data; TempData to flash messages | – |

---

## References (Full List)

- Views in ASP.NET Core MVC: Weakly typed data (ViewData, ViewBag) – https://learn.microsoft.com/zh-tw/aspnet/core/mvc/views/overview?view=aspnetcore-11.0#VD_VB
- ViewData and ViewBag are equally valid for small amounts of data – https://learn.microsoft.com/zh-tw/aspnet/core/mvc/views/overview?view=aspnetcore-11.0#VD_VB
- ViewBag derives from `DynamicViewData` – https://learn.microsoft.com/zh-tw/aspnet/core/mvc/views/overview?view=aspnetcore-11.0#VD_VB
- Session and state management in ASP.NET Core: TempData – https://learn.microsoft.com/zh-tw/aspnet/core/fundamentals/app-state?view=aspnetcore-1.1#2
- TempData `Peek` and `Keep` – https://learn.microsoft.com/zh-tw/aspnet/core/fundamentals/app-state?view=aspnetcore-1.1#2
- Configuring the TempData provider – https://learn.microsoft.com/zh-tw/aspnet/core/fundamentals/app-state?view=aspnetcore-1.1#2
- Cookie-based TempData provider (default) – https://learn.microsoft.com/zh-tw/aspnet/core/fundamentals/app-state?view=aspnetcore-1.1#2
- Prefer ViewModel over ViewData/ViewBag (SSW Rule) – https://www.ssw.com.au/rules/do-you-use-viewmodel-instead-of-viewdata
- `Controller` class (ViewData, ViewBag, TempData properties) – https://github.com/dotnet/aspnetcore/blob/main/src/Mvc/Mvc.ViewFeatures/src/Controller.cs#L91
- ViewData/ViewBag are per-request; TempData survives redirects (Microsoft Q&A) – https://learn.microsoft.com/en-us/answers/questions/1923251/routing-in-asp-net-core