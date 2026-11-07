# Route Parameters in ASP.NET Core MVC: A Comprehensive Cheat Sheet

> **Version Scope & Compatibility Note**
> This cheat sheet focuses on **ASP.NET Core MVC** (3.0 and later, including .NET 6/7/8/10) and the **endpoint routing** system that became the default in ASP.NET Core 3.0. Route parameters behave consistently across **conventional routing** and **attribute routing**; examples use both strategies where appropriate to illustrate the underlying mechanics. Legacy routing (`System.Web.Routing`, pre‑3.0) is referenced only where its behaviour differs meaningfully. Parameter transformers (e.g., `slugify`) and catch‑all parameters (`{*path}`) are covered as advanced forms of route parameters.

---

## Topic Overview

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Route parameters are named placeholders in a URL template that capture variable segments of an incoming request path and make those values available to the application’s endpoint (controller action, Razor Page, or minimal‑API handler). |
| **Technical Definition** | In ASP.NET Core routing, a route parameter is defined by enclosing a parameter name in braces (`{name}`) within a **route template**. During matching, the routing middleware tokenises the URL path into segments and assigns each segment to the parameter occupying the same position in the template. The captured values are stored in `RouteData.Values` (or `HttpContext.Request.RouteValues` in endpoint routing) and subsequently bound to action‑method parameters by the **model binder**. Route parameters support **optional markers** (`?`), **default values** (`=`), **inline constraints** (`:`), and **catch‑all semantics** (`*`). |
| **Beginner‑Friendly Explanation** | Route parameters are the **fill‑in‑the‑blanks** of a URL. In `products/{id}`, the `{id}` is a blank. If a user visits `/products/5`, the blank is filled with `5` and handed to your code as the `id` argument. If the URL is `/products/hello`, the blank is filled with `hello`. Route parameters let you create flexible, readable URLs without hard‑coding every possible path. |

### Key Characteristics

- **Named placeholders** – Every route parameter has a name that is used for binding to action parameters and for URL generation.
- **Positional capture** – Parameters capture values from the URL path in the order they appear, unless they are **complex segments** with literal delimiters.
- **Optionality** – Appending `?` to a parameter name makes it optional; appending `=value` supplies a default value.
- **Constraint‑aware** – Inline constraints (`{id:int}`, `{slug:regex(...)}`) validate captured values before the endpoint is selected.
- **Transformer‑aware** – Parameters can specify **transformers** (e.g., `{article:slugify}`) that modify values during link generation and matching.
- **Catch‑all capability** – Prefixing a parameter with `*` or `**` captures the remainder of the URL, including slashes.
- **Case‑insensitive matching** – Parameter names and literal segments are matched case‑insensitively; captured values retain their original casing.

### Prerequisites

- **.NET SDK** (6.0, 7.0, 8.0, or 10.0 recommended; examples use .NET 8/10 minimal hosting).
- **ASP.NET Core MVC** services registered (`builder.Services.AddControllersWithViews()` or `AddControllers()`).
- Routing middleware configured (`app.UseRouting()` and `app.MapControllers()` / `MapControllerRoute`).
- Basic familiarity with **route templates**, **controllers**, and **action methods**.
- Understanding of **HTTP verbs** and **model binding** is helpful but not required.

### Related Programming Areas

- **Route Constraints** – Validate parameter values during matching.
- **Model Binding** – Converts captured route values into action‑method parameter types.
- **URL Generation** – Uses route parameters and their values to construct links.
- **Attribute Routing** – Frequently uses route parameters in action‑level templates.
- **Conventional Routing** – Uses route parameters in centrally defined templates (e.g., `{controller}/{action}/{id?}`).
- **API Versioning** – Route parameters often carry version identifiers (`v{version}`).
- **SEO‑Friendly URLs** – `{slug}` parameters and the `slugify` transformer produce readable, keyword‑rich URLs.

### Core Concepts / Features

1. **Route Parameters** – The overarching mechanism for capturing URL segments.
2. **`{id}`** – A common parameter name for numeric or GUID identifiers.
3. **`{slug}`** – A parameter name for human‑readable, URL‑friendly strings.
4. **Optional Parameters** – Parameters marked with `?` that may be omitted from the URL.
5. **Default Values** – Parameters marked with `=value` that supply a fallback when the segment is absent.
6. **Route Parameter Best Practices** – Guidelines for designing robust, maintainable, and secure parameterised URLs.

Each concept is elaborated using the uniform structure: Definitions, Purposes, Syntax Rules & Structure, Annotated Code Examples with Expected Outputs, Real‑World Cases, and References.

---

## 1. Route Parameters (General)

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | A route parameter is a named variable embedded in a route template that captures a corresponding segment of the request URL. |
| **Technical Definition** | Route parameters are declared inside curly braces (`{name}`) in a route template. The routing system parses the template into a `RoutePattern` containing `RoutePatternParameterPart` objects. During matching, the URL path is split into segments; each segment is assigned to the parameter at the same position. Captured values are stored as strings in `RouteData.Values` and are later converted by model binding. Parameters may carry metadata: optionality (`?`), default values (`=`), inline constraints (`:`), and transformers (`:transformerName`). |
| **Beginner‑Friendly Explanation** | Route parameters are the **changeable parts** of a URL pattern. Think of a form letter: “Dear `{name}`, your order `{orderId}` is ready.” The words in braces are replaced with actual values. In routing, `/orders/{orderId}` matches `/orders/100` and hands `100` to your code as `orderId`. |

### Purposes (each begins with “To”)

- **To** capture variable data from the URL path and make it available to the endpoint without relying on query strings.
- **To** create hierarchical, resource‑oriented URLs that reflect the application’s data model (e.g., `/customers/5/orders/10`).
- **To** enable model binding to automatically populate action‑method parameters from URL segments.
- **To** support optional and default values so that a single template can handle multiple URL shapes.
- **To** allow inline constraints and transformers that validate and normalise parameter values during matching and link generation.
- **To** facilitate URL generation by providing the values needed to construct links from route templates.

### Syntax Rules and Structure

#### General Syntax

```
{parameterName}
{parameterName?}          // optional
{parameterName=default}   // default value
{parameterName:constraint} // inline constraint
{parameterName:transformer} // inline transformer
{*parameterName}          // catch-all
{**parameterName}         // catch-all (preserves slashes in generated URLs)
```

#### Component Breakdown

| Component | Syntax | Example | Meaning |
|-----------|--------|---------|---------|
| **Required parameter** | `{name}` | `{id}` | The segment must be present for the route to match. |
| **Optional parameter** | `{name?}` | `{id?}` | The segment may be omitted; if omitted, no route value is produced. |
| **Default value** | `{name=value}` | `{action=Index}` | If the segment is absent, `value` is used; a route value is **always** produced. |
| **Inline constraint** | `{name:constraint}` | `{id:int}` | The captured value must satisfy the constraint. |
| **Inline transformer** | `{name:transformer}` | `{article:slugify}` | The value is transformed during link generation and matching. |
| **Catch‑all** | `{*name}` or `{**name}` | `{*path}`, `{**slug}` | Captures the remainder of the URL, including slashes. |
| **Complex segment** | `{name=value:constraint}` | `{id=1:int}` | Combines default and constraint in one segment. |

#### Syntax Rules

- Parameter names must be valid C# identifiers and are **case‑insensitive** during matching.
- A template **cannot** contain two consecutive parameters without a literal delimiter (e.g., `{a}{b}` is invalid).
- Optional (`?`) and default (`=`) markers must come **after** the parameter name; constraints (`:`) and transformers follow the name and precede `?` or `=`.
- Catch‑all parameters must be the **last** segment of the template.
- Route parameters with a **default value** always produce a route value when the route matches; **optional parameters** produce a value only when the URL supplies one.

#### Constraints and Limitations

- **Empty segments** – Optional parameters do not match empty segments; `/products/` does not match `products/{id?}`.
- **Default values always produce a value** – A parameter with a default is **not** the same as an optional parameter; the default is used when the segment is absent, but a route value is still produced.
- **Catch‑all greediness** – Catch‑all parameters are the least specific template element and can consume URLs intended for other routes.
- **Regex constraints and ReDoS** – Complex regex constraints applied to untrusted input can cause denial‑of‑service; use anchored patterns and timeouts.
- **Parameter transformers are not constraints** – Transformers modify values (e.g., slugify) but do not validate them; use constraints for validation.

### Multiple Annotated Complete Step‑by‑Step Code Examples

#### Example 1: Route Parameters in Conventional Routing

```csharp
// Program.cs
var builder = WebApplication.CreateBuilder(args);
builder.Services.AddControllersWithViews();
var app = builder.Build();

app.UseRouting();

app.MapControllerRoute(
    name: "default",
    pattern: "{controller=Home}/{action=Index}/{id?}");

app.Run();

// Controllers/ProductsController.cs
public class ProductsController : Controller
{
    public IActionResult Details(int id) => Content($"Product ID: {id}");
}
```

**Step‑by‑step explanation:**

1. `MapControllerRoute` registers the template `{controller=Home}/{action=Index}/{id?}`.
2. `{controller}`, `{action}`, and `{id?}` are route parameters.
3. A request to `/Products/Details/5` captures `controller = "Products"`, `action = "Details"`, and `id = "5"`.
4. Model binding converts the string `"5"` to the integer `5` for the `Details` action.

**Expected Output:**

| URL | Action Invoked | Response |
|-----|----------------|----------|
| `/Products/Details/5` | `Details(5)` | `Product ID: 5` |
| `/Products/Details` | `Details(0)` (default int) | `Product ID: 0` |

**Why this output?** The `id` parameter is optional (`?`), so `/Products/Details` matches with `id` absent. Model binding assigns the default value `0` to the `int` parameter when no route value is present.

---

#### Example 2: Route Parameters in Attribute Routing

```csharp
[Route("api/[controller]")]
[ApiController]
public class BooksController : ControllerBase
{
    [HttpGet("{id:int}")]
    public IActionResult GetById(int id) => Ok($"Book {id}");
}
```

**Step‑by‑step explanation:**

1. `[Route("api/[controller]")]` sets the controller prefix to `api/Books`.
2. `[HttpGet("{id:int}")]` appends `{id:int}` to the prefix → `api/Books/{id:int}`.
3. The `int` constraint ensures only integer segments reach the action.

**Expected Output:**

| URL | Match? | Response |
|-----|--------|----------|
| `/api/Books/42` | Yes | `Book 42` |
| `/api/Books/abc` | No (constraint fails) | 404 |

**Why this output?** The `int` constraint rejects non‑integer values before the action is selected, so `abc` never reaches `GetById`.

---

#### Example 3: Catch‑All Route Parameter

```csharp
app.MapGet("/files/{*path}", (string path) => $"File path: {path}");
```

**Step‑by‑step explanation:**

1. `{*path}` is a catch‑all parameter.
2. It captures everything after `/files/`, including slashes.
3. The captured value is assigned to the `path` parameter.

**Expected Output:**

| URL | `path` Value |
|-----|--------------|
| `/files/docs/report.pdf` | `docs/report.pdf` |
| `/files/images/logo.png` | `images/logo.png` |

**Why this output?** Catch‑all parameters bind to the rest of the URI, making them suitable for file paths and SPA fallback routes.

### Real‑World Cases

- **E‑commerce product detail:** `/products/{id}` captures the product identifier.
- **Blog post URLs:** `/blog/{slug}` captures a human‑readable slug.
- **File serving:** `/files/{*path}` serves arbitrary nested files.
- **Hierarchical APIs:** `/customers/{customerId}/orders/{orderId}` models nested resources.

### References

- Route parameters – https://learn.microsoft.com/en-us/aspnet/core/fundamentals/routing#route-parameters
- Route template reference – https://learn.microsoft.com/en-us/aspnet/core/fundamentals/routing#route-template-reference
- Catch‑all parameters – https://learn.microsoft.com/en-us/aspnet/core/fundamentals/routing#catch-all-parameters

---

## 2. `{id}` Parameter

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | `{id}` is a conventional route parameter name used to capture an identifier—typically a numeric, GUID, or string value—that uniquely identifies a resource. |
| **Technical Definition** | `{id}` is not a special keyword in routing; it is simply a parameter named `id`. However, by convention, `id` is widely used to represent the primary key of a resource. It can carry inline constraints (`{id:int}`, `{id:guid}`) and is bound to action parameters named `id` by the model binder. In attribute routing, `{id}` is frequently combined with HTTP verb attributes to form RESTful endpoint templates. |
| **Beginner‑Friendly Explanation** | `{id}` is the **standard blank for an identifier**. When you see `/products/5`, the `5` is the ID of the product. Using `{id}` in your route template tells the framework: “The last part of this URL is the identifier of the thing the user wants.” |

### Purposes (each begins with “To”)

- **To** capture the unique identifier of a resource from the URL path.
- **To** enable RESTful endpoint design where the resource identifier is part of the URL (`/api/products/{id}`).
- **To** allow model binding to convert the captured string to the action parameter’s type (int, Guid, string).
- **To** support inline constraints that reject malformed identifiers before the action executes.
- **To** provide a consistent naming convention that makes route templates self‑documenting.

### Syntax Rules and Structure

#### General Syntax

```
{id}
{id:int}
{id:guid}
{id?}
{id=1}
{id:int:min(1)}
```

#### Component Breakdown

| Syntax | Meaning | Typical Use |
|--------|---------|-------------|
| `{id}` | Unconstrained identifier | String IDs, slugs, or when validation occurs in the action. |
| `{id:int}` | 32‑bit integer identifier | Database primary keys. |
| `{id:guid}` | GUID identifier | Distributed systems, non‑sequential IDs. |
| `{id?}` | Optional identifier | Detail views that can also show a list when `id` is absent. |
| `{id=1}` | Default identifier | Fallback when the segment is missing. |
| `{id:int:min(1)}` | Integer ≥ 1 | Ensures positive IDs. |

#### Syntax Rules

- `id` is **not reserved**; you can use other names (e.g., `{productId}`), but `id` is the conventional choice.
- In attribute routing, `{id}` is often placed in the action template, e.g., `[HttpGet("{id:int}")]`.
- In conventional routing, `{id?}` is commonly the third segment of the default route.
- The `int` constraint on `{id:int}` ensures that only integer values match; non‑integer segments cause a 404 (or fall through to another route).

#### Constraints and Limitations

- **Sequential integer IDs** can enable enumeration attacks; consider GUIDs or slugs for public‑facing resources.
- **Overly permissive `{id}`** without constraints may allow unexpected values to reach the action, requiring additional validation.
- **`{id:int}` rejects negative numbers only if combined with `min(0)` or `min(1)`**; the `int` constraint alone accepts negative integers.

### Multiple Annotated Complete Step‑by‑Step Code Examples

#### Example 1: Integer ID with Constraint

```csharp
[Route("api/[controller]")]
[ApiController]
public class ProductsController : ControllerBase
{
    [HttpGet("{id:int}")]
    public IActionResult GetProduct(int id) => Ok($"Product {id}");
}
```

**Step‑by‑step explanation:**

1. The action template `{id:int}` is appended to `api/Products`.
2. Only integer segments match the route.
3. Model binding converts the string segment to `int`.

**Expected Output:**

| URL | Match? | Response |
|-----|--------|----------|
| `/api/Products/10` | Yes | `Product 10` |
| `/api/Products/abc` | No | 404 |

**Why this output?** The `int` constraint validates the segment before the action is selected. Non‑integer values are rejected early, avoiding model‑binding errors in the action.

---

#### Example 2: GUID ID

```csharp
[HttpGet("{id:guid}")]
public IActionResult GetOrder(Guid id) => Ok($"Order {id}");
```

**Step‑by‑step explanation:**

1. `{id:guid}` requires the segment to be a valid GUID.
2. The action parameter is of type `Guid`.

**Expected Output:**

| URL | Match? |
|-----|--------|
| `/api/Orders/3fa85f64-5717-4562-b3fc-2c963f66afa6` | Yes |
| `/api/Orders/123` | No (not a GUID) |

**Why this output?** The `guid` constraint validates the format, ensuring only GUID‑format identifiers reach the action. This is common in distributed systems where sequential integers are undesirable.

---

#### Example 3: String ID Without Constraint

```csharp
[HttpGet("{id}")]
public IActionResult GetByCode(string id) => Ok($"Code: {id}");
```

**Step‑by‑step explanation:**

1. No constraint is applied to `{id}`.
2. Any non‑empty segment matches.
3. The action receives the raw string value.

**Expected Output:**

| URL | Response |
|-----|----------|
| `/api/Products/ABC-123` | `Code: ABC-123` |
| `/api/Products/hello` | `Code: hello` |

**Why this output?** Without a constraint, the parameter accepts any segment. This is useful when the identifier format is not strictly numeric or GUID‑based, but validation must then occur inside the action.

### Real‑World Cases

- **Database‑backed CRUD APIs:** `GET /api/products/42` retrieves product 42.
- **User profiles:** `GET /api/users/7` fetches user 7.
- **Order tracking:** `GET /api/orders/{id:guid}` retrieves an order by its GUID.
- **Hybrid ID systems:** `GET /api/invoices/{id}` where `id` is an alphanumeric invoice number.

### References

- Route constraints – https://learn.microsoft.com/en-us/aspnet/core/fundamentals/routing#route-constraints
- `int` constraint – https://learn.microsoft.com/en-us/aspnet/core/fundamentals/routing#int
- `guid` constraint – https://learn.microsoft.com/en-us/aspnet/core/fundamentals/routing#guid

---

## 3. `{slug}` Parameter

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | `{slug}` is a route parameter name conventionally used to capture a **human‑readable, URL‑friendly string** that identifies a resource, often derived from its title or name. |
| **Technical Definition** | `{slug}` is an ordinary route parameter; its name signals intent. Slugs are typically lowercase, hyphen‑separated strings (e.g., `my-first-post`). ASP.NET Core provides a **parameter transformer** called `slugify` (`{article:slugify}`) that can normalise values during link generation and matching. Slugs are often combined with `regex` constraints (`{slug:regex(^[a-z0-9-]+$)}`) to enforce URL‑safe characters. |
| **Beginner‑Friendly Explanation** | A **slug** is the readable part of a URL that describes the content. For example, in `/blog/my-first-post`, `my-first-post` is the slug. Using `{slug}` in your route tells the framework: “This part of the URL is a friendly name, not a number.” Slugs make URLs easier to read and better for search engines. |

### Purposes (each begins with “To”)

- **To** create SEO‑friendly, readable URLs that describe the resource instead of exposing raw identifiers.
- **To** capture a human‑readable string from the URL and bind it to an action parameter.
- **To** support the `slugify` transformer, which normalises values (e.g., converts spaces to hyphens and lowercases text) during link generation.
- **To** enforce URL‑safe character sets through regex constraints (`^[a-z0-9-]+$`).
- **To** provide a stable, non‑sequential identifier that mitigates enumeration attacks.

### Syntax Rules and Structure

#### General Syntax

```
{slug}
{slug:regex(^[a-z0-9-]+$)}
{article:slugify}
{slug?}
```

#### Component Breakdown

| Syntax | Meaning | Effect |
|--------|---------|--------|
| `{slug}` | Unconstrained slug | Accepts any segment; validation occurs in the action. |
| `{slug:regex(...)}` | Regex‑constrained slug | Enforces allowed characters (e.g., lowercase letters, digits, hyphens). |
| `{article:slugify}` | Slugify transformer | Normalises the value during link generation and matching. |
| `{slug?}` | Optional slug | Slug may be omitted. |

#### Syntax Rules

- **Slugify transformer** – `{article:slugify}` transforms a value by lowercasing, removing diacritics, and replacing spaces with hyphens. It is a **transformer**, not a constraint; it does not reject values.
- **Regex constraint** – `{slug:regex(^[a-z0-9-]+$)}` ensures only lowercase letters, digits, and hyphens match. The anchors (`^` and `$`) are essential to match the entire segment.
- **Catch‑all slug** – `blog/{*slug}` or `blog/{**slug}` captures multi‑segment slugs (e.g., `2025/03/my-post`).

#### Constraints and Limitations

- **Regex performance** – Complex slug regexes applied to untrusted input can be vulnerable to ReDoS; use simple, anchored patterns and timeouts.
- **Slug uniqueness** – Routing does not enforce slug uniqueness; that is an application‑level concern.
- **Slug stability** – If slugs are derived from titles, changing the title can break existing URLs; consider storing a stable slug or redirecting old slugs.

### Multiple Annotated Complete Step‑by‑Step Code Examples

#### Example 1: Slug with Regex Constraint

```csharp
[HttpGet("blog/{slug:regex(^[a-z0-9-]+$)}")]
public IActionResult GetPost(string slug) => Ok($"Post: {slug}");
```

**Step‑by‑step explanation:**

1. `{slug:regex(^[a-z0-9-]+$)}` applies a regex constraint.
2. Only lowercase letters, digits, and hyphens match.
3. The action receives the slug as a string.

**Expected Output:**

| URL | Match? | Response |
|-----|--------|----------|
| `/blog/my-first-post` | Yes | `Post: my-first-post` |
| `/blog/My-Post` | No (uppercase `M`) | 404 |
| `/blog/post_1` | No (underscore) | 404 |

**Why this output?** The regex enforces a URL‑safe slug format, rejecting uppercase letters and underscores before the action is invoked.

---

#### Example 2: Slugify Transformer

```csharp
[HttpGet("articles/{title:slugify}")]
public IActionResult GetArticle(string title) => Ok($"Article: {title}");
```

**Step‑by‑step explanation:**

1. `{title:slugify}` applies the `slugify` transformer.
2. During matching, values are normalised (lowercased, spaces → hyphens).
3. The action receives the transformed value.

**Expected Output:**

| URL | Response |
|-----|----------|
| `/articles/My First Article` | `Article: my-first-article` (transformed) |
| `/articles/hello-world` | `Article: hello-world` |

**Why this output?** The `slugify` transformer normalises the captured value, ensuring consistent slug formatting even if the raw URL contains spaces or uppercase letters.

---

#### Example 3: Catch‑All Slug for Hierarchical Paths

```csharp
[HttpGet("docs/{**slug}")]
public IActionResult GetDoc(string slug) => Ok($"Doc: {slug}");
```

**Step‑by‑step explanation:**

1. `{**slug}` is a catch‑all parameter that preserves slashes during URL generation.
2. It captures multi‑segment paths after `docs/`.

**Expected Output:**

| URL | `slug` Value |
|-----|--------------|
| `/docs/getting-started/installation` | `getting-started/installation` |
| `/docs/api/routing` | `api/routing` |

**Why this output?** The double‑asterisk catch‑all (`{**slug}`) captures the full remaining path and preserves slashes, making it suitable for documentation or wiki‑style URLs.

### Real‑World Cases

- **Blog engines:** `/blog/{slug}` produces URLs like `/blog/aspnet-core-routing`.
- **E‑commerce categories:** `/category/{slug}` yields `/category/electronics`.
- **Documentation sites:** `/docs/{**slug}` serves nested documentation pages.
- **SEO landing pages:** `/landing/{slug}` for marketing pages with keyword‑rich URLs.

### References

- Parameter transformers – https://learn.microsoft.com/en-us/aspnet/core/fundamentals/routing#parameter-transformers
- `slugify` transformer – https://learn.microsoft.com/en-us/aspnet/core/fundamentals/routing#slugify
- Regex constraints – https://learn.microsoft.com/en-us/aspnet/core/fundamentals/routing#regex

---

## 4. Optional Parameters

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | An optional route parameter is a parameter marked with a question mark (`?`) that may be omitted from the URL without causing the route to fail matching. |
| **Technical Definition** | Appending `?` to a parameter name (`{id?}`) signals that the parameter is optional. During matching, if the corresponding URL segment is absent, the parameter produces **no route value** (unlike a default value, which always produces a value). Optional parameters must appear **after all required parameters and literals** in the template. In URL generation, if a value for an optional parameter is not supplied, the segment is omitted from the generated URL. |
| **Beginner‑Friendly Explanation** | An optional parameter is a **“nice to have”** part of the URL. In `products/{id?}`, the `id` can be left out. So `/products` and `/products/5` both work. If the `id` is missing, your code receives no value for it (or `null` for reference types). |

### Purposes (each begins with “To”)

- **To** allow a single route template to match multiple URL shapes (with and without a particular segment).
- **To** support list‑and‑detail views from the same action (e.g., `Details` without `id` shows a list).
- **To** provide flexibility in URL design without creating separate routes for every combination of segments.
- **To** enable incremental adoption of parameters: clients can start supplying the optional value without requiring a new route.
- **To** distinguish between “value not supplied” (optional) and “use a fallback” (default value).

### Syntax Rules and Structure

#### General Syntax

```
{parameter?}
{parameter?}/{other?}
{required}/{optional?}
```

#### Component Breakdown

| Syntax | Meaning | Route Value Produced? |
|--------|---------|----------------------|
| `{id?}` | Optional parameter | Only if the segment is present; otherwise `null` (or no value). |
| `{page?}` | Optional page number | Same behaviour. |
| `{id?}/{page?}` | Multiple optional parameters | Must come after all required parameters. |

#### Syntax Rules

- Optional parameters must appear **after all required parameters and literals** in the template.
- An optional parameter **does not produce a route value** when the segment is absent; the action parameter receives `null` (for nullable/reference types) or the type’s default value.
- In URL generation, if no value is supplied for an optional parameter, the segment is omitted from the generated URL.
- **Optional parameters cannot be followed by required parameters** in a way that would create ambiguity; the template must be structured so that omission is unambiguous.

#### Constraints and Limitations

- **Empty segments** – An optional parameter does not match an empty segment; `/products/` does not match `products/{id?}`.
- **Model binding defaults** – If the action parameter is non‑nullable (e.g., `int id`), omitting the optional route value causes model binding to assign the type’s default (`0`), which may be indistinguishable from a legitimate `0` value.
- **Order sensitivity in URL generation** – If any parameter to the right of a missing optional parameter has a value, URL generation may fail.

### Multiple Annotated Complete Step‑by‑Step Code Examples

#### Example 1: Optional Page Number

```csharp
[HttpGet("articles/{category}/{page:int?}")]
public IActionResult List(string category, int? page) =>
    Ok($"Category: {category}, Page: {page?.ToString() ?? "1"}");
```

**Step‑by‑step explanation:**

1. `{category}` is required.
2. `{page:int?}` is optional and constrained to integers.
3. The action uses `int?` to distinguish “no page” from “page 0”.

**Expected Output:**

| URL | Response |
|-----|----------|
| `/articles/tech` | `Category: tech, Page: 1` (page is `null`, displayed as `1`) |
| `/articles/tech/2` | `Category: tech, Page: 2` |

**Why this output?** The optional `page` parameter is `null` when absent; the action substitutes a display default. If `page` were declared as `int`, model binding would assign `0`, which might be confused with a real page number.

---

#### Example 2: Optional Filter Segment

```csharp
[HttpGet("products/{category}/{filter?}")]
public IActionResult Browse(string category, string? filter) =>
    Ok($"Category: {category}, Filter: {filter ?? "none"}");
```

**Step‑by‑step explanation:**

1. `{filter?}` is optional and unconstrained.
2. When absent, `filter` is `null`.

**Expected Output:**

| URL | Response |
|-----|----------|
| `/products/electronics` | `Category: electronics, Filter: none` |
| `/products/electronics/sale` | `Category: electronics, Filter: sale` |

**Why this output?** The optional `filter` parameter allows the same action to serve both unfiltered and filtered browsing without a separate route.

---

#### Example 3: Optional Parameters Must Come After Required

```csharp
// ✅ Valid: optional parameters at the end
[HttpGet("reports/{year:int}/{month:int?}/{day:int?}")]
public IActionResult Report(int year, int? month, int? day) =>
    Ok($"Year: {year}, Month: {month?.ToString() ?? "all"}, Day: {day?.ToString() ?? "all"}");

// ❌ Invalid: optional parameter before a required parameter
// [HttpGet("reports/{month:int?}/{year:int}")]
```

**Step‑by‑step explanation:**

1. Required `year` comes first.
2. Optional `month` and `day` follow.
3. This ordering ensures that omitting the optional segments does not create ambiguity.

**Expected Output:**

| URL | Response |
|-----|----------|
| `/reports/2025` | `Year: 2025, Month: all, Day: all` |
| `/reports/2025/6` | `Year: 2025, Month: 6, Day: all` |
| `/reports/2025/6/15` | `Year: 2025, Month: 6, Day: 15` |

**Why this output?** Optional parameters must trail required ones; placing an optional parameter before a required one would make the template ambiguous or invalid.

### Real‑World Cases

- **Product listing with optional filters:** `/products/{category}/{filter?}`.
- **Archive browsing:** `/archive/{year:int}/{month:int?}/{day:int?}`.
- **Search pagination:** `/search/{query}/{page:int?}`.
- **User profiles with optional tab:** `/users/{id}/{tab?}`.

### References

- Optional parameters – https://learn.microsoft.com/en-us/aspnet/core/fundamentals/routing#optional-parameters
- Optional route parameter order – https://learn.microsoft.com/en-us/aspnet/core/fundamentals/routing#optional-route-parameter-order

---

## 5. Default Values

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | A default route parameter value is a fallback value specified after an equals sign (`{parameter=value}`) that is used when the corresponding URL segment is absent. |
| **Technical Definition** | A parameter with a default value (`{action=Index}`) **always produces a route value** when the route matches. If the URL supplies a value for the segment, that value is used; if the segment is absent, the default is used. This is the key distinction from optional parameters, which produce **no route value** when absent. Default values can be combined with constraints (`{id=1:int}`) and appear in both conventional and attribute routing. |
| **Beginner‑Friendly Explanation** | A default value is a **backup value**. In `{action=Index}`, if the user doesn’t specify an action in the URL, the application automatically uses `Index`. So `/` works the same as `/Home/Index`. The difference from an optional parameter is that a default **always gives you a value**, even if the user didn’t provide one. |

### Purposes (each begins with “To”)

- **To** provide a fallback value when a URL segment is omitted, ensuring that the action always receives a meaningful value.
- **To** enable short, friendly URLs (e.g., `/` → `Home/Index`) without requiring all segments to be present.
- **To** simplify URL generation by supplying values for segments that clients commonly omit.
- **To** distinguish between “value not supplied” (optional) and “use this specific fallback” (default value).
- **To** support templates where a parameter should always have a value, even if the client does not provide one.

### Syntax Rules and Structure

#### General Syntax

```
{parameter=value}
{parameter=value:constraint}
{controller=Home}/{action=Index}/{id?}
```

#### Component Breakdown

| Syntax | Meaning | Route Value Produced? |
|--------|---------|----------------------|
| `{action=Index}` | Default value `Index` | Always; `Index` if segment absent. |
| `{id=1:int}` | Default `1` with `int` constraint | Always; `1` if segment absent, and must be an integer if present. |
| `{page=1}` | Default page number `1` | Always; `1` if segment absent. |

#### Syntax Rules

- A default value is specified after the parameter name using `=`.
- Default values are used **only when the segment is absent**; an empty segment does not trigger the default.
- Default values can be combined with constraints: `{id=1:int}`.
- In conventional routing, `{controller=Home}/{action=Index}/{id?}` combines defaults (`Home`, `Index`) with an optional parameter (`id`).
- A parameter with a default value **always produces a route value**, even when the URL does not supply one.

#### Constraints and Limitations

- **Defaults are not validation** – A default value does not validate user input; combine with constraints or action‑level validation.
- **Default values affect URL generation** – When generating URLs, if no value is supplied, the default is used, which may produce URLs that differ from the current request.
- **Ambiguity with multiple defaults** – Multiple parameters with defaults can make route matching less predictable; keep defaults to a minimum.

### Multiple Annotated Complete Step‑by‑Step Code Examples

#### Example 1: Default Action and Controller

```csharp
app.MapControllerRoute(
    name: "default",
    pattern: "{controller=Home}/{action=Index}/{id?}");
```

**Step‑by‑step explanation:**

1. `{controller=Home}` defaults to `HomeController`.
2. `{action=Index}` defaults to the `Index` action.
3. `{id?}` is optional.

**Expected Output:**

| URL | Controller | Action |
|-----|------------|--------|
| `/` | `Home` | `Index` |
| `/Products` | `Products` | `Index` (default action) |
| `/Products/Details/5` | `Products` | `Details` (explicit action) |

**Why this output?** The defaults fill in `Home` and `Index` when those segments are missing, enabling the application root to work without an explicit `/Home/Index` URL.

---

#### Example 2: Default Page Number

```csharp
[HttpGet("articles/{category}/{page:int=1}")]
public IActionResult List(string category, int page) =>
    Ok($"Category: {category}, Page: {page}");
```

**Step‑by‑step explanation:**

1. `{page:int=1}` has a default value of `1` and an `int` constraint.
2. If `page` is absent, `1` is used.
3. If present, the value must be an integer.

**Expected Output:**

| URL | Response |
|-----|----------|
| `/articles/tech` | `Category: tech, Page: 1` |
| `/articles/tech/2` | `Category: tech, Page: 2` |

**Why this output?** The default value ensures `page` always has a value, even when the segment is omitted. This differs from an optional parameter, which would produce `null` (or `0` for `int`) instead of a meaningful default.

---

#### Example 3: Default with Constraint

```csharp
[HttpGet("orders/{status=all}/{page:int=1}")]
public IActionResult Orders(string status, int page) =>
    Ok($"Status: {status}, Page: {page}");
```

**Step‑by‑step explanation:**

1. `{status=all}` defaults to `"all"`.
2. `{page:int=1}` defaults to `1` and must be an integer.
3. Both parameters always produce values.

**Expected Output:**

| URL | Response |
|-----|----------|
| `/orders` | `Status: all, Page: 1` |
| `/orders/shipped` | `Status: shipped, Page: 1` |
| `/orders/shipped/3` | `Status: shipped, Page: 3` |

**Why this output?** Default values provide sensible fallbacks, allowing the same action to handle unfiltered and filtered views without requiring separate routes.

### Real‑World Cases

- **Default controller/action:** `{controller=Home}/{action=Index}` makes the site root work.
- **Default page number:** `{page:int=1}` simplifies pagination URLs.
- **Default sort order:** `{sort=date}` provides a fallback sorting criterion.
- **Default language:** `{culture=en}` supplies a default culture when none is specified.

### References

- Default values – https://learn.microsoft.com/en-us/aspnet/core/fundamentals/routing#default-values
- Inline default values – https://learn.microsoft.com/en-us/aspnet/core/fundamentals/routing#inline-default-values

---

## 6. Route Parameter Best Practices

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Route parameter best practices are guidelines for designing parameterised URLs that are predictable, maintainable, secure, and performant. |
| **Technical Definition** | Best practices encompass parameter naming, constraint usage, optionality design, default‑value placement, slug vs. ID selection, catch‑all avoidance, and the separation of routing concerns from validation. They aim to minimise ambiguity, prevent routing conflicts, and ensure that URL generation remains reliable. |
| **Beginner‑Friendly Explanation** | Best practices are the **rules of thumb** that keep your URLs clean and your routing predictable. For example: use constraints to reject bad IDs, prefer slugs for public‑facing content, keep optional parameters at the end, and don’t use catch‑all routes unless you really need them. |

### Purposes (each begins with “To”)

- **To** ensure deterministic route matching by using constraints and correct parameter ordering.
- **To** improve security by rejecting malformed identifiers before they reach application code.
- **To** enhance SEO and user experience through readable slugs and clean URL structures.
- **To** maintain reliable URL generation by choosing parameter names and templates consistently.
- **To** avoid ambiguity and routing conflicts that cause `AmbiguousMatchException` or unreachable endpoints.
- **To** separate routing concerns (matching) from validation concerns (business rules).

### Syntax Rules and Structure (Guideline Checklist)

| Guideline | Rationale | Example |
|-----------|-----------|---------|
| **Constrain identifiers** | Reject invalid values early; avoid model‑binding errors. | `{id:int}`, `{id:guid}` |
| **Prefer slugs for public content** | Readable URLs improve SEO and user trust. | `/blog/{slug}` |
| **Place optional parameters last** | Required parameters must precede optional ones. | `{year}/{month?}/{day?}` |
| **Use default values sparingly** | Defaults always produce a value and can mask missing input. | `{page:int=1}` |
| **Avoid catch‑all unless necessary** | Catch‑alls are greedy and can swallow other routes. | Use only for file serving or SPA fallback. |
| **Use regex constraints with anchors** | Anchored patterns match the entire segment and reduce false positives. | `{slug:regex(^[a-z0-9-]+$)}` |
| **Pass a timeout to regex constraints** | Prevents ReDoS attacks on untrusted input. | `regex(..., TimeSpan.FromSeconds(1))` |
| **Distinguish optional from default** | Optional produces no value; default always produces a value. | Use `?` for “may be absent,” `=` for “fallback.” |
| **Validate business rules in the action** | Constraints only handle matching, not business validation. | Check `id > 0` inside the action. |
| **Keep parameter names consistent** | Consistent names improve readability and URL generation. | Use `id`, `slug`, `page` consistently. |

#### Constraints and Limitations

- **Over‑constraining** – Too many constraints can make routes brittle and hard to extend.
- **Order overuse** – Rely on template precedence (literals > constrained parameters > unconstrained parameters) before using `Order`.
- **Catch‑all greediness** – Catch‑all parameters can match unintended URLs; place them after more specific routes.
- **Regex complexity** – Complex regex constraints can degrade performance and introduce ReDoS vulnerabilities; keep patterns simple and anchored.

### Multiple Annotated Complete Step‑by‑Step Code Examples

#### Example 1: Constrain IDs and Prefer Slugs for Public Content

```csharp
[Route("api/[controller]")]
[ApiController]
public class ArticlesController : ControllerBase
{
    // Internal ID-based access
    [HttpGet("{id:int}")]
    public IActionResult GetById(int id) => Ok($"Article {id}");

    // Public slug-based access
    [HttpGet("by-slug/{slug:regex(^[a-z0-9-]+$)}")]
    public IActionResult GetBySlug(string slug) => Ok($"Article: {slug}");
}
```

**Step‑by‑step explanation:**

1. `{id:int}` ensures only integer IDs reach `GetById`.
2. `{slug:regex(...)}` ensures only URL‑safe slugs reach `GetBySlug`.
3. The two templates are distinct and constrained, preventing ambiguity.

**Expected Output:**

| URL | Action |
|-----|--------|
| `/api/Articles/42` | `GetById(42)` |
| `/api/Articles/by-slug/my-article` | `GetBySlug("my-article")` |
| `/api/Articles/by-slug/My-Article` | 404 (uppercase not allowed) |

**Why this output?** Constraints act as discriminators, allowing ID‑based and slug‑based access to coexist without ambiguity. The slug regex enforces URL‑safe formatting.

---

#### Example 2: Correct Ordering of Optional and Default Parameters

```csharp
[HttpGet("reports/{year:int}/{month:int?}/{day:int?}")]
public IActionResult Report(int year, int? month, int? day) =>
    Ok($"Year: {year}, Month: {month?.ToString() ?? "all"}, Day: {day?.ToString() ?? "all"}");
```

**Step‑by‑step explanation:**

1. Required `year` comes first.
2. Optional `month` and `day` follow.
3. This ordering respects the rule that optional parameters must come after required ones.

**Expected Output:**

| URL | Response |
|-----|----------|
| `/reports/2025` | `Year: 2025, Month: all, Day: all` |
| `/reports/2025/6` | `Year: 2025, Month: 6, Day: all` |
| `/reports/2025/6/15` | `Year: 2025, Month: 6, Day: 15` |

**Why this output?** Proper ordering ensures that omission of optional segments is unambiguous and does not break matching.

---

#### Example 3: Avoiding Catch‑All Greediness

```csharp
// ✅ Specific routes before catch-all
app.MapControllerRoute(
    name: "blog",
    pattern: "blog/{year:int}/{month:int}",
    defaults: new { controller = "Blog", action = "Archive" });

app.MapControllerRoute(
    name: "blog-catchall",
    pattern: "blog/{*slug}",
    defaults: new { controller = "Blog", action = "Post" });
```

**Step‑by‑step explanation:**

1. The specific `blog/{year:int}/{month:int}` route is registered first.
2. The catch‑all `blog/{*slug}` route is registered second.
3. Requests matching the specific pattern are handled by `Archive`; all other blog URLs fall through to the catch‑all `Post` action.

**Expected Output:**

| URL | Action |
|-----|--------|
| `/blog/2025/06` | `Archive` (specific route) |
| `/blog/my-post` | `Post` (catch‑all route) |

**Why this output?** Registering specific routes before catch‑all routes prevents the catch‑all from swallowing URLs intended for more specific handlers.

### Real‑World Cases

- **E‑commerce API:** Constrain `{id:int}` for database‑backed product lookups; use `{slug}` for SEO‑friendly public product pages.
- **Documentation site:** Specific routes for versioned API docs (`/docs/v1/{page}`) before a catch‑all (`/docs/{**slug}`) for general documentation.
- **Multi‑language site:** Required `{culture}` parameter first, followed by optional page parameters, ensuring correct ordering.

### References

- Routing best practices – https://learn.microsoft.com/en-us/aspnet/core/fundamentals/routing#routing-best-practices
- Route constraint reference – https://learn.microsoft.com/en-us/aspnet/core/fundamentals/routing#route-constraint-reference
- Optional route parameter order – https://learn.microsoft.com/en-us/aspnet/core/fundamentals/routing#optional-route-parameter-order
- Catch‑all parameters – https://learn.microsoft.com/en-us/aspnet/core/fundamentals/routing#catch-all-parameters

---

## Summary Table: Route Parameters at a Glance

| Concept | Primary Purpose | Key Syntax | When to Use |
|---------|----------------|------------|-------------|
| **Route Parameters** | Capture URL segments | `{name}`, `{*path}` | Passing data from URL to actions |
| **`{id}`** | Capture resource identifiers | `{id:int}`, `{id:guid}` | Database keys, RESTful resource IDs |
| **`{slug}`** | Capture human‑readable strings | `{slug:regex(...)}`, `{article:slugify}` | SEO‑friendly blog posts, category pages |
| **Optional Parameters** | Allow segments to be omitted | `{param?}` | List/detail views, optional filters |
| **Default Values** | Provide fallback values | `{param=value}` | Default pages, default sorting |
| **Best Practices** | Maintainability & security | Constraints, ordering, slug preference | Production‑ready route design |

---

## References (Full List)

- Routing in ASP.NET Core – https://learn.microsoft.com/en-us/aspnet/core/fundamentals/routing
- Route parameters – https://learn.microsoft.com/en-us/aspnet/core/fundamentals/routing#route-parameters
- Route template reference – https://learn.microsoft.com/en-us/aspnet/core/fundamentals/routing#route-template-reference
- Catch‑all parameters – https://learn.microsoft.com/en-us/aspnet/core/fundamentals/routing#catch-all-parameters
- Optional parameters – https://learn.microsoft.com/en-us/aspnet/core/fundamentals/routing#optional-parameters
- Default values – https://learn.microsoft.com/en-us/aspnet/core/fundamentals/routing#default-values
- Optional route parameter order – https://learn.microsoft.com/en-us/aspnet/core/fundamentals/routing#optional-route-parameter-order
- Parameter transformers – https://learn.microsoft.com/en-us/aspnet/core/fundamentals/routing#parameter-transformers
- `slugify` transformer – https://learn.microsoft.com/en-us/aspnet/core/fundamentals/routing#slugify
- Route constraints – https://learn.microsoft.com/en-us/aspnet/core/fundamentals/routing#route-constraints
- Route constraint reference – https://learn.microsoft.com/en-us/aspnet/core/fundamentals/routing#route-constraint-reference
- Routing best practices – https://learn.microsoft.com/en-us/aspnet/core/fundamentals/routing#routing-best-practices
- Minimal APIs quick reference: Route parameters – https://learn.microsoft.com/en-us/aspnet/core/fundamentals/minimal-apis
- Raw GitHub: Routing in ASP.NET Core (route template reference) – https://raw.githubusercontent.com/dotnet/AspNetCore.Docs/448b927ad97cbef4bfaeb3eace46b67791e25f9d/aspnetcore/fundamentals/routing.md