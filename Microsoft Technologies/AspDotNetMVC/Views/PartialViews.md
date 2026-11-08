# Partial Views in ASP.NET Core MVC: A Comprehensive Cheat Sheet

> **Version Scope & Compatibility Note**
> This cheat sheet targets **ASP.NET Core MVC** (3.0 and later, including .NET 6/7/8/9/10/11). Partial views are shared with **Razor Pages**; the primary difference is the folder (`Pages/` vs. `Views/`), which is noted where relevant. The **Partial Tag Helper** (`<partial>`) requires ASP.NET Core 2.1 or later and is the **recommended** approach. The asynchronous HTML Helpers (`Html.PartialAsync`, `Html.RenderPartialAsync`) are supported but considered legacy compared to the Tag Helper. The **synchronous** HTML Helpers (`Html.Partial`, `Html.RenderPartial`) are **not recommended** because they can cause deadlocks and are targeted for removal in a future release. Partial views **do not run `_ViewStart.cshtml`** and therefore do not apply a layout by default.

---

## Topic Overview

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | A partial view is a **Razor markup file** (`.cshtml`) **without an `@page` directive** that renders HTML output **inside another markup file's rendered output**, enabling reusable UI fragments. |
| **Technical Definition** | A partial view is a `.cshtml` file maintained within the `Views` folder (MVC) or `Pages` folder (Razor Pages) that is **not a full view or page**: it has no `@page` directive, does not run `_ViewStart.cshtml`, and is rendered via the **Partial Tag Helper** (`<partial name="_Name" />`), **`Html.PartialAsync`**, or **`Html.RenderPartialAsync`**. Controllers can also return a `PartialViewResult` to render a partial as the response. Partial views receive a model and `ViewData` from the calling context; `ViewData` changes inside a partial are **not persisted** to the parent. |
| **Beginner-Friendly Explanation** | A partial view is a **reusable piece of HTML**—like a product card, a comment, or a navigation menu—that you can drop into multiple pages. Instead of copy-pasting the same markup everywhere, you write it once in a `.cshtml` file and render it wherever you need it. The file name usually starts with `_` (e.g., `_ProductCard.cshtml`) to signal that it's a fragment, not a full page. |

### Key Characteristics

- **No `@page` directive** – Distinguishes partials from Razor Pages.
- **No layout by default** – Partials do not execute `_ViewStart.cshtml`, so no `_Layout.cshtml` is applied unless explicitly set.
- **Underscore naming convention** – Optional but recommended (`_PartialName.cshtml`) to visually distinguish partials from full views.
- **Model and ViewData inheritance** – Partials receive a copy of the parent’s `ViewData` and can receive a model explicitly.
- **Reusable and chainable** – Partials can call other partials (no circular references).
- **Rendering options** – Tag Helper (recommended, async), `Html.PartialAsync`, `Html.RenderPartialAsync` (streaming), or synchronous variants (not recommended).
- **View components for logic** – When rendering requires code execution or complex logic, use a **view component** instead of a partial view.

### Prerequisites

- **.NET SDK** (6.0+ recommended; examples use .NET 8/10).
- **ASP.NET Core MVC** project with Razor views (`.cshtml`).
- Basic understanding of **Razor syntax**, **models**, and **views**.
- Tag Helpers enabled (via `_ViewImports.cshtml`) for the `<partial>` syntax.

### Related Programming Areas

- **Views** – Partials are rendered inside full views.
- **Layouts** – Layout elements belong in `_Layout.cshtml`; partials are for content fragments.
- **View Components** – The alternative when logic is required.
- **Tag Helpers** – The `<partial>` Tag Helper is the recommended rendering mechanism.
- **Model Binding** – Partials bind to a model passed by the parent.
- **ViewData** – Partials inherit a copy of `ViewData` from the parent.

### Core Concepts / Features

1. **Partial Views (General)** – The concept, role, and when to use them.
2. **`_PartialView.cshtml`** – Naming, file organisation, and declaration.
3. **`@Html.Partial()`** – Synchronous HTML Helper (legacy, not recommended).
4. **`@Html.PartialAsync()`** – Asynchronous HTML Helper (returns `IHtmlContent`).
5. **`@Html.RenderPartial()`** – Synchronous streaming HTML Helper (legacy, not recommended).
6. **`@Html.RenderPartialAsync()`** – Asynchronous streaming HTML Helper.
7. **Partial View Best Practices** – Guidelines for clean, performant, and safe partial views.

---

## 1. Partial Views (General)

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | A partial view is a reusable Razor fragment that renders a portion of HTML within a parent view or page. |
| **Technical Definition** | Partial views are `.cshtml` files **without an `@page` directive** that are referenced and rendered from within other markup files. In MVC, a controller action can return a `PartialViewResult` to render a partial as the entire response. Partial views are discovered by name through a search process (caller's folder → `Shared` folders → area folders). They are intended for **reusable content**, not for common layout elements or logic-heavy rendering. |
| **Beginner-Friendly Explanation** | A partial view is a **building block** for pages. Imagine you have a product card that appears on the homepage, the search results, and the category page. Instead of writing the card's HTML three times, you create a partial view `_ProductCard.cshtml` and render it in all three places. Change it once, and every page updates. |

### Purposes (each begins with “To”)

- **To** break large, complex markup files into smaller, manageable components.
- **To** reduce duplication of common markup across multiple views or pages.
- **To** enable reuse of UI fragments (cards, lists, forms, navigation) across different contexts.
- **To** keep the parent view focused on overall page structure, with details delegated to partials.
- **To** support component-based UI development in Razor (complemented by view components for logic-heavy scenarios).

### Syntax Rules and Structure

#### General Syntax (Declaring a Partial View)

```cshtml
@* Views/Shared/_ProductCard.cshtml *@
@model MyApp.ViewModels.ProductSummaryViewModel

<div class="card">
    <h3>@Model.Name</h3>
    <p>@Model.Price.ToString("C")</p>
</div>
```

#### Component Breakdown

| Component | Description |
|-----------|-------------|
| File extension | `.cshtml` (Razor markup). |
| `@page` directive | **Must be absent** for a partial view. |
| `@model` | Optional but recommended for strong typing. |
| File name | Conventionally starts with `_` (e.g., `_ProductCard.cshtml`). |
| Location | `Views/` (MVC) or `Pages/` (Razor Pages), typically under `Shared/` for cross-controller reuse. |

#### Syntax Rules

- **No `@page` directive** – part of what makes it a partial, not a page.
- **No `_ViewStart.cshtml` execution** – partials do not inherit the parent’s layout automatically.
- **Partial names can include or omit the `.cshtml` extension**; when the extension is present, the partial must be in the same folder as the calling file.
- **Search order for partial discovery** (by name without extension):
  - Currently executing page’s folder
  - Directory graph above the page’s folder
  - `/Shared`
  - `/Pages/Shared`
  - `/Views/Shared`
  - Area-specific folders (`/Areas/<Area>/Views/<Controller>` and `/Areas/<Area>/Views/Shared`).

#### Constraints and Limitations

- **Do not use partials for common layout elements** – use `_Layout.cshtml` instead.
- **Do not use partials for complex rendering logic** – use a **view component**.
- **Razor sections inside a partial are invisible to the parent** – sections defined in a partial are scoped to that partial only.
- **ViewData changes in a partial are not persisted** to the parent.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Basic Partial View and Its Use in a Parent View

```cshtml
@* Views/Shared/_ProductCard.cshtml *@
@model MyApp.ViewModels.ProductSummaryViewModel

<div class="card">
    <h3>@Model.Name</h3>
    <p>@Model.Price.ToString("C")</p>
</div>
```

```cshtml
@* Views/Products/Index.cshtml *@
@model IEnumerable<ProductSummaryViewModel>

<h2>Products</h2>
<div class="row">
    @foreach (var product in Model)
    {
        <partial name="_ProductCard" model="product" />
    }
</div>
```

**Step-by-step explanation:**

1. `_ProductCard.cshtml` declares `@model ProductSummaryViewModel`.
2. The parent view iterates the product list and renders the partial for each item.
3. The `<partial>` Tag Helper passes `product` as the model to the partial.

**Expected Output:** A grid of product cards, each rendered by the shared partial.

**Why this output?** The partial is discovered by name (searching the caller’s folder and `Shared`), and the model is passed explicitly.

---

#### Example 2: Controller Returning a PartialViewResult

```csharp
public IActionResult GetProductCard(int id)
{
    var product = _service.GetProductSummary(id);
    if (product == null) return NotFound();
    return PartialView("_ProductCard", product);
}
```

**Expected Output:** The response contains only the HTML fragment rendered by `_ProductCard.cshtml` (no layout, no full page).

**Why this output?** `PartialView` returns a `PartialViewResult`, which renders the partial without a layout; this is useful for AJAX responses.

---

#### Example 3: Chaining Partials (Partial Calling Another Partial)

```cshtml
@* Views/Shared/_ProductList.cshtml *@
@model IEnumerable<ProductSummaryViewModel>

<div class="product-list">
    @foreach (var product in Model)
    {
        <partial name="_ProductCard" model="product" />
    }
</div>
```

**Expected Output:** The list partial renders each product using the card partial.

**Why this output?** Partials can call other partials (as long as no circular reference exists).

### Real-World Cases

- **E-commerce** – `_ProductCard.cshtml`, `_PriceTag.cshtml`, `_RatingStars.cshtml`.
- **Blog** – `_Comment.cshtml`, `_AuthorBadge.cshtml`, `_PostMeta.cshtml`.
- **Admin panels** – `_StatCard.cshtml`, `_SidebarMenu.cshtml`, `_DataTable.cshtml`.
- **Multi-tenant apps** – `_TenantHeader.cshtml` reused across tenant-specific views.

### References

- Partial views in ASP.NET Core – https://github.com/dotnet/aspnetcore.docs/blob/47d6f5c46f7a4c552fbf8ca9077439b95150c719/aspnetcore/mvc/views/partial.md
- When to use partial views – https://github.com/dotnet/aspnetcore.docs/blob/47d6f5c46f7a4c552fbf8ca9077439b95150c719/aspnetcore/mvc/views/partial.md

---

## 2. `_PartialView.cshtml` (Naming and Declaration)

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | `_PartialView.cshtml` is the **conventional naming pattern** for partial view files, where the leading underscore signals that the file is a fragment, not a full view or page. |
| **Technical Definition** | Partial view file names **often begin with an underscore (`_`)**. This naming convention is **not required** by the framework but is strongly recommended to visually differentiate partials from full views and pages. The file resides in `Views/` (MVC) or `Pages/` (Razor Pages) and must **not contain an `@page` directive**. When referenced without a file extension, the partial is discovered via the search order (caller’s folder → `Shared` → area folders). |
| **Beginner-Friendly Explanation** | The underscore at the start of `_PartialView.cshtml` is like a **sticky note** that says “this is a reusable piece, not a whole page.” It’s not enforced by the framework, but it helps you and your team immediately recognise partial files when browsing the project. |

### Purposes (each begins with “To”)

- **To** visually distinguish partial views from full views and pages.
- **To** follow a widely adopted convention that improves project navigability.
- **To** signal to other developers that the file is intended for reuse and does not render a full page.
- **To** avoid confusion with Razor Pages (which use `@page` and do not typically start with `_`).

### Syntax Rules and Structure

#### General Syntax

```
Views/
├── Shared/
│   ├── _Layout.cshtml
│   ├── _ProductCard.cshtml      ← partial view
│   └── _Footer.cshtml           ← partial view
├── Products/
│   ├── Index.cshtml             ← full view
│   └── _ProductRow.cshtml       ← partial view
```

#### Component Breakdown

| Aspect | Convention |
|--------|------------|
| Prefix | `_` (underscore) |
| Extension | `.cshtml` |
| Location | `Views/` (MVC) or `Pages/` (Razor Pages), often under `Shared/` for cross-controller reuse |
| `@page` directive | **Absent** |
| `@model` | Optional but recommended |

#### Syntax Rules

- **The underscore is a convention, not a requirement**; partials without underscores work but are less discoverable.
- **When referencing with an extension** (e.g., `_PartialName.cshtml`), the partial must be in the same folder as the calling file.
- **When referencing without an extension**, the search order applies.

#### Constraints and Limitations

- **Do not use partials for layout elements** – use `_Layout.cshtml`.
- **Do not use partials for logic-heavy rendering** – use view components.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Creating and Referencing a `_`-Prefixed Partial

```cshtml
@* Views/Shared/_ProductCard.cshtml *@
@model ProductSummaryViewModel

<div class="card">
    <h3>@Model.Name</h3>
</div>
```

```cshtml
@* Views/Home/Index.cshtml *@
<partial name="_ProductCard" model="Model.FeaturedProduct" />
```

**Expected Output:** The product card is rendered inside the home page.

**Why this output?** The underscore prefix is a convention; the framework discovers the partial by name and renders it.

---

#### Example 2: Referencing with an Explicit Path

```cshtml
<partial name="~/Views/Shared/_ProductCard.cshtml" model="Model.Product" />
```

**Expected Output:** The partial is rendered from the explicit path, bypassing the name-based search.

**Why this output?** When the extension is present (and the path starts with `~/` or `/`), the path is treated as an app-root-relative reference.

---

#### Example 3: Partial Without Underscore (Works, But Not Recommended)

```cshtml
@* Views/Shared/ProductCard.cshtml (no underscore) *@
@model ProductSummaryViewModel
<div class="card">@Model.Name</div>
```

```cshtml
<partial name="ProductCard" model="Model.Product" />
```

**Expected Output:** The partial is rendered successfully.

**Why this output?** The underscore is not required; the partial is discovered by name. However, the lack of an underscore makes it visually indistinguishable from a full view.

### Real-World Cases

- **`_ProductCard.cshtml`** – product summary fragment.
- **`_Comment.cshtml`** – individual comment rendering.
- **`_Sidebar.cshtml`** – sidebar content.
- **`_ValidationScriptsPartial.cshtml`** – validation scripts for forms.

### References

- Declaring partial views (underscore convention) – https://github.com/dotnet/aspnetcore.docs/blob/47d6f5c46f7a4c552fbf8ca9077439b95150c719/aspnetcore/mvc/views/partial.md
- Partial view discovery – https://github.com/dotnet/aspnetcore.docs/blob/47d6f5c46f7a4c552fbf8ca9077439b95150c719/aspnetcore/mvc/views/partial.md

---

## 3. `@Html.Partial()`

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | `@Html.Partial()` is a **synchronous HTML Helper** that renders a partial view and returns an `IHtmlContent` object. |
| **Technical Definition** | `Html.Partial(string partialViewName)` (and overloads with model and `ViewData`) renders the partial **synchronously** and returns an `IHtmlContent` that can be output with `@`. It is the synchronous counterpart of `Html.PartialAsync`. **Microsoft does not recommend** the synchronous helpers because they can cause deadlocks in certain scenarios, and they are targeted for removal in a future release. |
| **Beginner-Friendly Explanation** | `@Html.Partial()` is the **old way** to render a partial synchronously. It gives you back the partial's HTML, which you can output with `@`. Modern ASP.NET Core recommends the `<partial>` Tag Helper or `Html.PartialAsync` instead, because the synchronous version can cause problems under load. |

### Purposes (each begins with “To”)

- **To** render a partial view synchronously and return its HTML content.
- **To** support legacy codebases that use the synchronous helper.
- **To** provide a simple inline syntax (`@Html.Partial("_Name")`) for rendering partials.

### Syntax Rules and Structure

#### General Syntax

```cshtml
@Html.Partial("_PartialName")
@Html.Partial("_PartialName", model)
@Html.Partial("_PartialName", model, new ViewDataDictionary(ViewData))
```

#### Component Breakdown

| Parameter | Description |
|-----------|-------------|
| `partialViewName` | Name of the partial (with or without `.cshtml`; path optional). |
| `model` | Optional model passed to the partial. |
| `viewData` | Optional `ViewDataDictionary` for the partial. |

#### Syntax Rules

- **Returns `IHtmlContent`** – output with `@Html.Partial(...)`.
- **Synchronous** – blocks the thread until rendering completes.
- **Calls the partial synchronously** – does not support `await` inside the partial.

#### Constraints and Limitations

- **Not recommended by Microsoft** – can cause deadlocks.
- **Visual Studio analyzer warning** – “Use of IHtmlHelper.Partial may result in application deadlocks”.
- **Synchronous partials with async operations** – should use `Html.PartialAsync` instead.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Basic Synchronous Partial

```cshtml
@Html.Partial("_ProductCard", Model.FeaturedProduct)
```

**Expected Output:** The partial's HTML is rendered at that location.

**Why this output?** `Html.Partial` renders the partial synchronously and returns `IHtmlContent`.

---

#### Example 2: Passing `ViewData`

```cshtml
@Html.Partial("_ProductCard", Model.Product, new ViewDataDictionary(ViewData) { ["ShowPrice"] = true })
```

**Expected Output:** The partial is rendered with the supplied model and `ViewData`.

**Why this output?** The third parameter provides a custom `ViewDataDictionary` for the partial.

### Real-World Cases

- **Legacy ASP.NET Core 1.x projects** – still using synchronous helpers.
- **Simple, synchronous partials** – when no async operations are involved and the codebase has not yet migrated.

### References

- `Html.Partial` (synchronous, not recommended) – https://github.com/dotnet/aspnetcore.docs/blob/47d6f5c46f7a4c552fbf8ca9077439b95150c719/aspnetcore/mvc/views/partial.md
- `IHtmlHelper.Partial` API – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.rendering.htmlhelperpartialextensions.partial

---

## 4. `@Html.PartialAsync()`

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | `@Html.PartialAsync()` is an **asynchronous HTML Helper** that renders a partial view asynchronously and returns a `Task<IHtmlContent>`. |
| **Technical Definition** | `Html.PartialAsync(string partialViewName, object model)` renders the partial **asynchronously** and returns a `Task<IHtmlContent>`. It is the **recommended asynchronous HTML Helper** for partials and is awaited with `@await`. It is a legacy alternative to the **Partial Tag Helper**, which is the modern recommended approach. |
| **Beginner-Friendly Explanation** | `@Html.PartialAsync()` is the **async version** of `Html.Partial`. You use it with `@await` to render a partial without blocking the thread. It's better than the synchronous version, but the `<partial>` Tag Helper is even more modern and readable. |

### Purposes (each begins with “To”)

- **To** render a partial view **asynchronously**, avoiding blocking the request thread.
- **To** support partials that contain asynchronous operations.
- **To** provide a safer alternative to the synchronous `Html.Partial`.
- **To** return `IHtmlContent` that can be composed with other content.

### Syntax Rules and Structure

#### General Syntax

```cshtml
@await Html.PartialAsync("_PartialName")
@await Html.PartialAsync("_PartialName", model)
@await Html.PartialAsync("_PartialName", model, new ViewDataDictionary(ViewData))
```

#### Component Breakdown

| Parameter | Description |
|-----------|-------------|
| `partialViewName` | Name or path of the partial. |
| `model` | Optional model. |
| `viewData` | Optional `ViewDataDictionary`. |

#### Syntax Rules

- **Must be awaited** with `@await`.
- **Returns `Task<IHtmlContent>`** – awaiting yields the rendered HTML.
- **Async partials** – if the partial itself uses `@await`, this is the correct helper.

#### Constraints and Limitations

- **Legacy alternative** – the `<partial>` Tag Helper is preferred for new code.
- **Not for streaming** – use `RenderPartialAsync` for streaming scenarios.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Basic Asynchronous Partial

```cshtml
@await Html.PartialAsync("_ProductCard", Model.FeaturedProduct)
```

**Expected Output:** The partial's HTML is rendered asynchronously.

**Why this output?** `PartialAsync` returns a task that resolves to the rendered content; `await` unwraps it.

---

#### Example 2: Partial with Custom `ViewData`

```cshtml
@await Html.PartialAsync("_ProductCard", Model.Product, new ViewDataDictionary(ViewData) { ["ShowPrice"] = true })
```

**Expected Output:** The partial is rendered with the specified model and `ViewData`.

**Why this output?** The `ViewDataDictionary` is passed explicitly to the partial.

### Real-World Cases

- **Async data rendering** – partials that call async services.
- **Migrating from synchronous helpers** – replacing `@Html.Partial` with `@await Html.PartialAsync`.

### References

- Asynchronous HTML Helper for partials – https://github.com/dotnet/aspnetcore.docs/blob/47d6f5c46f7a4c552fbf8ca9077439b95150c719/aspnetcore/mvc/views/partial.md
- `IHtmlHelper.PartialAsync` API – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.rendering.htmlhelperpartialextensions.partialasync

---

## 5. `@Html.RenderPartial()`

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | `@Html.RenderPartial()` is a **synchronous streaming HTML Helper** that renders a partial view directly to the response stream, returning no value. |
| **Technical Definition** | `Html.RenderPartial(string partialViewName)` renders the partial **synchronously** and **writes the output directly to the response stream**. Because it returns `void`, it must be called inside a Razor code block (`@{ Html.RenderPartial(...); }`). It may be slightly faster than `Html.Partial` in some scenarios because it streams the output rather than buffering it into an `IHtmlContent`. However, like `Html.Partial`, it is **synchronous and not recommended** due to deadlock risks. |
| **Beginner-Friendly Explanation** | `@Html.RenderPartial()` is the **old streaming version** of partial rendering. It writes HTML straight to the page as it renders, rather than returning a string. It can be slightly faster, but it's synchronous and has the same deadlock risks as `Html.Partial`. Modern code should use the `<partial>` Tag Helper or `RenderPartialAsync`. |

### Purposes (each begins with “To”)

- **To** stream the partial’s HTML directly to the response, potentially improving performance.
- **To** avoid buffering the partial’s output in memory.
- **To** support legacy codebases that use the synchronous streaming helper.

### Syntax Rules and Structure

#### General Syntax

```cshtml
@{
    Html.RenderPartial("_PartialName");
    Html.RenderPartial("_PartialName", model);
}
```

#### Component Breakdown

| Parameter | Description |
|-----------|-------------|
| `partialViewName` | Name or path of the partial. |
| `model` | Optional model. |

#### Syntax Rules

- **Returns `void`** – must be called inside a code block, not with `@` directly.
- **Streams output** – the HTML is written as it is rendered.
- **Synchronous** – blocks the thread.

#### Constraints and Limitations

- **Not recommended** – same deadlock risks as `Html.Partial`.
- **Requires a code block** – less readable than expression-based helpers.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Basic Streaming Partial

```cshtml
@{
    Html.RenderPartial("_ProductCard", Model.FeaturedProduct);
}
```

**Expected Output:** The partial's HTML is written directly to the response at that location.

**Why this output?** `RenderPartial` streams the output; no value is returned.

---

#### Example 2: Inside a Loop

```cshtml
@foreach (var product in Model.Products)
{
    Html.RenderPartial("_ProductCard", product);
}
```

**Expected Output:** Each product card is streamed sequentially.

**Why this output?** The helper writes each partial's HTML directly to the response.

### Real-World Cases

- **Performance-sensitive legacy pages** – where streaming was historically preferred.
- **Legacy ASP.NET Core 1.x codebases** – before async helpers became standard.

### References

- Synchronous streaming helper (not recommended) – https://github.com/dotnet/aspnetcore.docs/blob/47d6f5c46f7a4c552fbf8ca9077439b95150c719/aspnetcore/mvc/views/partial.md
- `Html.RenderPartial` API – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.rendering.htmlhelperpartialextensions.renderpartial

---

## 6. `@Html.RenderPartialAsync()`

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | `@Html.RenderPartialAsync()` is an **asynchronous streaming HTML Helper** that renders a partial view asynchronously and writes the output directly to the response stream. |
| **Technical Definition** | `Html.RenderPartialAsync(string partialViewName)` renders the partial **asynchronously** and **streams the rendered output directly to the response**. Because it does not return an `IHtmlContent`, it must be called inside a Razor code block with `await` (`@{ await Html.RenderPartialAsync(...); }`). It provides **better performance in some scenarios** because it streams rather than buffers. |
| **Beginner-Friendly Explanation** | `@Html.RenderPartialAsync()` is the **async streaming version** of partial rendering. It writes the partial's HTML directly to the page as it renders, without returning a string. It's faster in some cases than `PartialAsync` because it doesn't buffer the whole partial in memory. |

### Purposes (each begins with “To”)

- **To** render a partial view asynchronously while **streaming** the output to the response.
- **To** improve performance in scenarios where buffering the entire partial is unnecessary.
- **To** support partials that contain asynchronous operations without blocking.
- **To** provide a safer async alternative to `Html.RenderPartial`.

### Syntax Rules and Structure

#### General Syntax

```cshtml
@{
    await Html.RenderPartialAsync("_PartialName");
    await Html.RenderPartialAsync("_PartialName", model);
}
```

#### Component Breakdown

| Parameter | Description |
|-----------|-------------|
| `partialViewName` | Name or path of the partial. |
| `model` | Optional model. |

#### Syntax Rules

- **Returns `Task`** – must be awaited inside a code block.
- **Streams output** – the HTML is written incrementally.
- **Asynchronous** – does not block the thread.

#### Constraints and Limitations

- **Must be called inside a code block** – cannot be used with `@` directly.
- **Legacy alternative** – the `<partial>` Tag Helper is preferred for new code.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Basic Asynchronous Streaming Partial

```cshtml
@{
    await Html.RenderPartialAsync("_ProductCard", Model.FeaturedProduct);
}
```

**Expected Output:** The partial's HTML is streamed asynchronously to the response.

**Why this output?** `RenderPartialAsync` renders the partial asynchronously and writes the output directly.

---

#### Example 2: Inside a Loop with Async

```cshtml
@foreach (var product in Model.Products)
{
    await Html.RenderPartialAsync("_ProductCard", product);
}
```

**Expected Output:** Each product card is rendered asynchronously and streamed sequentially.

**Why this output?** The helper awaits each partial render and writes the output.

### Real-World Cases

- **Large partials** – streaming avoids buffering the entire fragment.
- **Async partials with I/O** – partials that call async services.
- **Performance-sensitive pages** – where streaming provides measurable benefits.

### References

- Asynchronous streaming helper – https://github.com/dotnet/aspnetcore.docs/blob/47d6f5c46f7a4c552fbf8ca9077439b95150c719/aspnetcore/mvc/views/partial.md
- `Html.RenderPartialAsync` API – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.rendering.htmlhelperpartialextensions.renderpartialasync

---

## 7. Partial View Best Practices

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Partial view best practices are guidelines for creating, referencing, and rendering partial views that are **reusable, performant, maintainable, and free of common pitfalls** like deadlocks and layout misuse. |
| **Technical Definition** | Best practices include: using the **Partial Tag Helper** (`<partial>`) over synchronous HTML Helpers, choosing **`PartialAsync` or `RenderPartialAsync`** when using HTML Helpers, avoiding partials for layout elements or logic-heavy rendering (use layouts or view components), following the **underscore naming convention**, passing models explicitly, being aware that `ViewData` changes do not persist, and understanding the **discovery search order**. |
| **Beginner-Friendly Explanation** | Best practices are the **rules of thumb** for partial views. For example: use the modern `<partial>` tag instead of the old `@Html.Partial`, never put layout stuff (header/footer) in a partial—use `_Layout.cshtml`—and if you need real logic (database calls, calculations), use a view component instead of a partial. |

### Purposes (each begins with “To”)

- **To** avoid **deadlocks** caused by synchronous partial rendering.
- **To** use the **most modern, readable syntax** (the `<partial>` Tag Helper).
- **To** keep partials **focused on reusable markup**, not layout or complex logic.
- **To** ensure **correct model and `ViewData` handling** (pass models explicitly; remember `ViewData` is a copy).
- **To** optimise performance by choosing streaming or buffered rendering based on the scenario.
- **To** follow conventions that improve **discoverability and maintainability** (underscore prefix, `Shared` folder for cross-cutting partials).

### Guideline Checklist

| Guideline | Rationale | Example |
|-----------|-----------|---------|
| **Use the `<partial>` Tag Helper** | Modern, async by default, readable HTML-like syntax. | `<partial name="_ProductCard" model="product" />` |
| **Use `Html.PartialAsync` or `Html.RenderPartialAsync` if using HTML Helpers** | Avoids deadlocks. | `@await Html.PartialAsync("_Name", model)` |
| **Never use `Html.Partial` or `Html.RenderPartial`** | Synchronous; deadlock risk; targeted for removal. | Replace with async or Tag Helper. |
| **Do not use partials for layout elements** | Layout belongs in `_Layout.cshtml`. | Header, footer, nav → layout. |
| **Do not use partials for logic-heavy rendering** | Use **view components** instead. | Database lookups, complex calculations → view component. |
| **Use the underscore prefix** | Convention improves discoverability. | `_ProductCard.cshtml` |
| **Pass models explicitly** | Avoids hidden dependencies; improves clarity. | `<partial name="_Card" model="item" />` |
| **Remember `ViewData` changes are not persisted** | Partials receive a copy. | Do not rely on mutations in partials. |
| **Place shared partials in `Shared/`** | Cross-controller reuse. | `Views/Shared/_Layout.cshtml`, `_ProductCard.cshtml` |
| **Be aware of the discovery order** | Understand which file wins when duplicates exist. | Caller’s folder → `Shared` → area folders. |
| **Avoid circular partial references** | Circular calls cause infinite recursion. | Partial A must not call Partial B if B calls A. |
| **Consider streaming for large partials** | `RenderPartialAsync` avoids buffering. | Large lists, heavy fragments. |

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Recommended `<partial>` Tag Helper Usage

```cshtml
@* Parent view *@
<div class="row">
    @foreach (var product in Model.Products)
    {
        <partial name="_ProductCard" model="product" />
    }
</div>
```

**Expected Output:** Each product card is rendered asynchronously by the Tag Helper.

**Why this output?** The `<partial>` Tag Helper renders asynchronously and passes the model explicitly.

---

#### Example 2: Using `PartialAsync` When HTML Helpers Are Required

```cshtml
@await Html.PartialAsync("_ProductCard", product)
```

**Expected Output:** The partial is rendered asynchronously and the HTML is output.

**Why this output?** `PartialAsync` is the recommended HTML Helper when the Tag Helper cannot be used.

---

#### Example 3: Avoiding Layout Elements in Partials

```cshtml
@* ❌ BAD: Partial containing layout elements *@
@* _Header.cshtml *@
<header><h1>My App</h1></header>

@* ✅ GOOD: Layout element in _Layout.cshtml *@
@* Views/Shared/_Layout.cshtml *@
<header><h1>My App</h1></header>
@RenderBody()
```

**Expected Output:** The header appears once in the layout, not duplicated across partials.

**Why this output?** Layout elements belong in `_Layout.cshtml`, not in partials.

---

#### Example 4: Using a View Component for Logic

```csharp
// ✅ GOOD: View component for logic-heavy rendering
public class CartSummaryViewComponent : ViewComponent
{
    public async Task<IViewComponentResult> InvokeAsync()
    {
        var count = await _cartService.GetItemCountAsync();
        return View(count);
    }
}
```

```cshtml
@await Component.InvokeAsync("CartSummary")
```

**Expected Output:** The cart summary is rendered by the view component, which can execute logic.

**Why this output?** View components are designed for logic-heavy rendering; partials are for markup reuse.

### Real-World Cases

- **E-commerce** – `<partial name="_ProductCard" model="product" />` in listings; view component for cart totals.
- **Blog** – `<partial name="_Comment" model="comment" />` for each comment; layout for header/footer.
- **Admin dashboard** – `<partial name="_StatCard" model="stat" />` for metrics; view component for real-time data.
- **Multi-tenant apps** – shared partials in `Shared/`; tenant-specific partials in area folders.

### References

- Partial views best practices (recommendations) – https://github.com/dotnet/aspnetcore.docs/blob/47d6f5c46f7a4c552fbf8ca9077439b95150c719/aspnetcore/mvc/views/partial.md
- Partial Tag Helper (recommended approach) – https://learn.microsoft.com/es-es/aspnet/core/mvc/views/tag-helpers/built-in/partial-tag-helper?view=aspnetcore-9.0
- Avoid synchronous helpers – https://github.com/dotnet/aspnetcore.docs/blob/47d6f5c46f7a4c552fbf8ca9077439b95150c719/aspnetcore/mvc/views/partial.md

---

## Summary Table: Partial Views at a Glance

| Concept | Primary Purpose | Key Takeaway | When to Use |
|---------|----------------|--------------|-------------|
| **Partial Views** | Reusable HTML fragments | No `@page`, no layout, no logic | Breaking up large markup; reducing duplication |
| **`_PartialView.cshtml`** | Naming convention | Underscore prefix signals a fragment | All partials |
| **`@Html.Partial()`** | Synchronous rendering (legacy) | Returns `IHtmlContent`; **not recommended** | Legacy code only |
| **`@Html.PartialAsync()`** | Asynchronous rendering | Returns `Task<IHtmlContent>`; use with `@await` | When using HTML Helpers |
| **`@Html.RenderPartial()`** | Synchronous streaming (legacy) | Writes directly; **not recommended** | Legacy code only |
| **`@Html.RenderPartialAsync()`** | Asynchronous streaming | Streams output; better performance in some cases | Large partials; async scenarios |
| **Best Practices** | Clean, safe, performant partials | `<partial>` Tag Helper, avoid layout/logic, underscore prefix | All production code |

---

## References (Full List)

- Partial views in ASP.NET Core (official docs, GitHub) – https://github.com/dotnet/aspnetcore.docs/blob/47d6f5c46f7a4c552fbf8ca9077439b95150c719/aspnetcore/mvc/views/partial.md
- Partial Tag Helper in ASP.NET Core (official docs) – https://learn.microsoft.com/es-es/aspnet/core/mvc/views/tag-helpers/built-in/partial-tag-helper?view=aspnetcore-9.0
- Asynchronous HTML Helper for partials – https://github.com/dotnet/aspnetcore.docs/blob/47d6f5c46f7a4c552fbf8ca9077439b95150c719/aspnetcore/mvc/views/partial.md
- Synchronous HTML Helpers not recommended – https://github.com/dotnet/aspnetcore.docs/blob/47d6f5c46f7a4c552fbf8ca9077439b95150c719/aspnetcore/mvc/views/partial.md
- Partial view discovery – https://github.com/dotnet/aspnetcore.docs/blob/47d6f5c46f7a4c552fbf8ca9077439b95150c719/aspnetcore/mvc/views/partial.md
- `IHtmlHelper.Partial` API – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.rendering.htmlhelperpartialextensions.partial
- `IHtmlHelper.PartialAsync` API – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.rendering.htmlhelperpartialextensions.partialasync
- `IHtmlHelper.RenderPartial` API – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.rendering.htmlhelperpartialextensions.renderpartial
- `IHtmlHelper.RenderPartialAsync` API – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.rendering.htmlhelperpartialextensions.renderpartialasync
- View components in ASP.NET Core – https://learn.microsoft.com/en-us/aspnet/core/mvc/views/view-components
- Layout in ASP.NET Core – https://learn.microsoft.com/en-us/aspnet/core/mvc/views/layout