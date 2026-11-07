# View Models in ASP.NET Core MVC: A Comprehensive Cheat Sheet

> **Version Scope & Compatibility Note**
> This cheat sheet targets **ASP.NET Core MVC** (3.0 and later, including .NET 6/7/8/9/10/11). ViewModels are plain CLR classes (POCOs) with **no framework-imposed base class or attributes**; their use is a **design pattern**, not a framework requirement. Data annotations (`[Required]`, `[StringLength]`, `[Range]`, etc.) on ViewModel properties integrate with **model validation** and `ModelState`. ViewModels are often used with **AutoMapper** or manual projection; both approaches are covered. The term "ViewModel" is distinct from **WPF/MVVM ViewModels** and **ASP.NET Core MVC's `ViewModel` in Razor Pages** (which is the `PageModel`); this guide focuses on MVC ViewModels.

---

## Topic Overview

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | A ViewModel is a **class shaped specifically for a view**, containing exactly the data that view needs—nothing more, nothing less—often combining data from multiple domain models and including validation rules tailored to the UI. |
| **Technical Definition** | A ViewModel is a POCO (Plain Old CLR Object) that acts as the **data contract between a controller action and a Razor view**. It is declared as the view's `@model` type and passed via `View(viewModel)`. ViewModels decouple the UI from domain entities, prevent **over-posting** (mass assignment) attacks, and enable view-specific concerns (validation, display formatting, aggregation of multiple sources). They reside in a `ViewModels` folder (or a dedicated project in layered architectures) and are typically projected from domain models via manual mapping or AutoMapper. |
| **Beginner-Friendly Explanation** | A ViewModel is a **custom-made data package for a view**. If a page needs a product's name, its category's name, and a list of reviews—but those come from three different database tables—you create a `ProductDetailsViewModel` that contains exactly those three things. The view gets exactly what it needs and nothing extra. This keeps the page simple, prevents exposing sensitive database fields, and makes the code easier to understand. |

### Key Characteristics

- **View-specific shape** – Contains only the properties the view actually uses.
- **No behaviour by default** – Typically data-only; computed properties are allowed but business logic belongs in services.
- **Validation-friendly** – Data annotations apply to the ViewModel (not the domain model), keeping UI validation separate from domain rules.
- **Over-posting protection** – Binds only the properties exposed on the ViewModel, ignoring extraneous form fields.
- **Aggregation** – Combines data from multiple domain models or services into a single object.
- **Decoupled from the database** – Changes to the domain model don't necessarily ripple to the UI.
- **Testable** – Plain classes are easy to instantiate and assert against in unit tests.

### Prerequisites

- **.NET SDK** (6.0+ recommended; examples use .NET 8/10).
- **ASP.NET Core MVC** project with controllers and Razor views.
- Basic understanding of **models**, **controllers**, **views**, and **data annotations**.
- Familiarity with **mapping** (manual or AutoMapper) is helpful.

### Related Programming Areas

- **Models** – Domain entities that ViewModels are projected from.
- **Controllers** – Actions create ViewModels and pass them to views.
- **Views** – Declare `@model` as the ViewModel type.
- **Model Validation** – Data annotations on ViewModels feed `ModelState`.
- **DTOs** – Similar concept for API boundaries; ViewModels are their UI counterparts.
- **AutoMapper** – Library for mapping between domain models and ViewModels.
- **Razor Pages** – Uses `PageModel` classes as a similar concept (page-centric).

### Core Concepts / Features

1. **View Models (General)** – The concept, role, and benefits.
2. **View Model Creation** – How to design and define ViewModel classes.
3. **View Model Usage** – Passing ViewModels to views, binding forms, and mapping data.
4. **View Model Best Practices** – Guidelines for maintainable, secure, and clean ViewModels.

---

## 1. View Models (General)

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | A ViewModel is a **class designed for a specific view**, containing the exact data the view needs in the shape it needs it. |
| **Technical Definition** | A ViewModel is a POCO used as the **`@model` type of a Razor view** and passed from a controller via `View(viewModel)`. It can aggregate data from multiple sources, expose computed properties, and carry **view-specific validation attributes**. ViewModels are conceptually similar to **Data Transfer Objects (DTOs)** but scoped to the presentation layer. They prevent **over-posting** because the model binder only populates properties on the ViewModel, and they decouple the view from domain entities. |
| **Beginner-Friendly Explanation** | A ViewModel is the **perfectly sized box for a view**. Instead of handing the view a giant domain object with lots of unnecessary or sensitive fields, you hand it a small, purpose-built object containing exactly what the view needs. This keeps views simple, prevents security issues, and makes the code easier to change. |

### Purposes (each begins with “To”)

- **To** shape data specifically for a view, including aggregated data from multiple domain models or services.
- **To** prevent **over-posting** (mass assignment) attacks by exposing only the properties a form should accept.
- **To** decouple the UI from domain entities, allowing both to evolve independently.
- **To** apply **view-specific validation** rules that differ from domain rules.
- **To** provide **computed or display-formatted** properties (e.g., `FullName`, `FormattedPrice`) directly to the view.
- **To** improve **testability** by keeping views "dumb" and controllers thin, with plain classes that are easy to instantiate and assert against.

### Syntax Rules and Structure

#### General Syntax

```csharp
public class ProductDetailsViewModel
{
    public int Id { get; set; }
    public string Name { get; set; } = string.Empty;
    public decimal Price { get; set; }
    public string CategoryName { get; set; } = string.Empty;
    public IEnumerable<ReviewDto> Reviews { get; set; } = Array.Empty<ReviewDto>();
    public string FormattedPrice => Price.ToString("C");
}
```

#### Component Breakdown

| Component | Description |
|-----------|-------------|
| Properties | Only the data the view needs. |
| Validation attributes | `[Required]`, `[StringLength]`, `[Range]` (view-specific rules). |
| Display attributes | `[Display(Name = "...")]` for labels. |
| Computed properties | Display-only properties (e.g., `FormattedPrice`, `FullName`). |
| No behaviour | Business logic belongs in services, not the ViewModel. |

#### Syntax Rules

- **ViewModel classes are POCOs** – no base class is required.
- **View-specific** – the shape reflects the view, not the database.
- **Placed in `ViewModels/`** by convention (or a separate project in layered architectures).
- **Validated by MVC's model validation** via data annotations on the properties.
- **`@model` in the view** must match the ViewModel type passed by the controller.

#### Constraints and Limitations

- **Not a domain model** – avoid mixing EF Core or business logic into ViewModels.
- **One ViewModel per view is common** – reuse across views is possible but can blur intent.
- **Manual mapping overhead** – projecting from domain models adds code (mitigated by AutoMapper or manual projection).

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Minimal ViewModel

```csharp
// ViewModels/ProductDetailsViewModel.cs
public class ProductDetailsViewModel
{
    public int Id { get; set; }
    public string Name { get; set; } = string.Empty;
    public decimal Price { get; set; }
    public string CategoryName { get; set; } = string.Empty;
}
```

**Step-by-step explanation:**

1. The ViewModel has four properties—exactly what the detail view needs.
2. It is a plain class; the framework treats it as data only.

**Expected Output:** The controller can pass an instance to the view; the view renders each property.

**Why this output?** ViewModels are simple containers; their purpose is to carry the right data to the view.

---

#### Example 2: ViewModel with Validation and Display Attributes

```csharp
// ViewModels/CreateProductViewModel.cs
using System.ComponentModel.DataAnnotations;

public class CreateProductViewModel
{
    [Required(ErrorMessage = "Name is required.")]
    [StringLength(100)]
    [Display(Name = "Product Name")]
    public string Name { get; set; } = string.Empty;

    [Range(0.01, 10000, ErrorMessage = "Price must be between 0.01 and 10,000.")]
    [Display(Name = "Price")]
    public decimal Price { get; set; }

    [StringLength(500)]
    [Display(Name = "Description")]
    public string? Description { get; set; }
}
```

**Expected Output:** Form labels use the `Display` names; validation messages appear on invalid input.

**Why this output?** Data annotations on the ViewModel drive both UI generation (labels) and server-side validation.

---

#### Example 3: ViewModel Aggregating Multiple Sources

```csharp
// ViewModels/DashboardViewModel.cs
public class DashboardViewModel
{
    public IEnumerable<OrderSummary> RecentOrders { get; set; } = Array.Empty<OrderSummary>();
    public int ActiveCustomers { get; set; }
    public decimal TotalRevenue { get; set; }
    public string FormattedRevenue => TotalRevenue.ToString("C");
}
```

**Expected Output:** The dashboard view displays aggregated metrics from multiple sources in a single view model.

**Why this output?** ViewModels can combine data that doesn't naturally live on a single entity, simplifying the view.

### Real-World Cases

- **E-commerce product page** – `ProductDetailsViewModel` combines product data, category name, and reviews.
- **Checkout flow** – `CheckoutViewModel` combines cart items, shipping address, and payment method.
- **Admin dashboard** – `DashboardViewModel` aggregates metrics from multiple services.
- **Registration form** – `RegisterViewModel` contains only email, password, and confirmation—not the full `User` entity.

### References

- ASP.NET Core MVC project structure (ViewModels) – https://github.com/OtakuDevs/Modular-ASPNetCore-MVC-Template
- ASP.NET Core Best Practices and Coding Standards – https://github.com/aios-labs/projectrules/blob/417776589dd52a6bd366bde067097938784f12c2/data/rules/manual/asp-net.md
- DTOs vs ViewModels (Microsoft Q&A) – https://learn.microsoft.com/en-us/answers/questions/1406728/dto-vs-viewmodel-in-asp-net-core-mvc

---

## 2. View Model Creation

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | ViewModel creation is the process of designing and defining ViewModel classes that match the shape of specific views, including properties, validation rules, and display metadata. |
| **Technical Definition** | Creating a ViewModel involves: (1) identifying the data the view needs, (2) defining a POCO class with those properties, (3) applying validation and display attributes, (4) placing it in a `ViewModels/` folder, and (5) mapping from domain models in the controller (manually or via AutoMapper). The ViewModel should be **input-focused** (forms) or **display-focused** (details), sometimes both. |
| **Beginner-Friendly Explanation** | ViewModel creation is **designing the perfect data box for a page**. You look at the page, list everything it needs to display or collect, then create a class with exactly those properties. For forms, you add validation rules (like "Name is required"). For display, you add formatted properties (like `FormattedPrice`). |

### Purposes (each begins with “To”)

- **To** define a **precise data contract** for a view.
- **To** apply **view-specific validation** (e.g., `[Required]` on a form field).
- **To** provide **display metadata** (`[Display]`, `[DataType]`) for labels and formatting.
- **To** expose **computed properties** (e.g., `FullName`, `FormattedDate`) for display.
- **To** separate **input models** (for forms) from **display models** (for read-only pages).
- **To** support **aggregation** of data from multiple domain models.

### Syntax Rules and Structure

#### General Syntax (Input ViewModel)

```csharp
public class CreateProductViewModel
{
    [Required]
    [StringLength(100)]
    [Display(Name = "Product Name")]
    public string Name { get; set; } = string.Empty;

    [Range(0.01, 10000)]
    public decimal Price { get; set; }

    [StringLength(500)]
    public string? Description { get; set; }
}
```

#### General Syntax (Display ViewModel)

```csharp
public class ProductDetailsViewModel
{
    public int Id { get; set; }
    public string Name { get; set; } = string.Empty;
    public decimal Price { get; set; }
    public string CategoryName { get; set; } = string.Empty;
    public string FormattedPrice => Price.ToString("C");
}
```

#### Component Breakdown

| Type | Purpose | Attributes Used |
|------|---------|-----------------|
| **Input ViewModel** | Bind form submissions | `[Required]`, `[StringLength]`, `[Range]`, `[EmailAddress]`, `[Compare]` |
| **Display ViewModel** | Render read-only data | `[Display]`, `[DataType]`, `[DisplayFormat]` |
| **Combined ViewModel** | Both input and display | Both sets |

#### Syntax Rules

- **Naming convention:** `<Feature><Action>ViewModel` (e.g., `CreateProductViewModel`, `ProductDetailsViewModel`).
- **Place in `ViewModels/`** folder (or `ViewModels/<Feature>/` for large apps).
- **Input ViewModels:** All properties needed by the form; use nullable types for optional fields.
- **Display ViewModels:** Only the properties the view renders; use computed properties for formatted values.
- **Do not include domain logic** – keep the class data-only.

#### Constraints and Limitations

- **Overly broad ViewModels** – combining unrelated concerns into one class reduces clarity; prefer separate input/display models.
- **Duplication** – ViewModels may duplicate properties from domain models; this is acceptable to protect the UI contract.
- **Nullable reference types** – for optional form fields, use `string?` to reflect absence.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Creating an Input ViewModel

```csharp
// ViewModels/CreateProductViewModel.cs
using System.ComponentModel.DataAnnotations;

public class CreateProductViewModel
{
    [Required(ErrorMessage = "Name is required.")]
    [StringLength(100, ErrorMessage = "Name must be 100 characters or fewer.")]
    [Display(Name = "Product Name")]
    public string Name { get; set; } = string.Empty;

    [Range(0.01, 10000, ErrorMessage = "Price must be between 0.01 and 10,000.")]
    [Display(Name = "Price (USD)")]
    public decimal Price { get; set; }

    [StringLength(500)]
    [Display(Name = "Description")]
    public string? Description { get; set; }
}
```

**Step-by-step explanation:**

1. `[Required]`, `[StringLength]`, and `[Range]` enforce validation.
2. `[Display]` provides form labels.
3. `string?` marks `Description` as optional.

**Expected Output:** The Create form renders with proper labels and validation messages.

**Why this output?** Data annotations drive both UI generation (Tag Helpers) and server-side validation.

---

#### Example 2: Creating a Display ViewModel

```csharp
// ViewModels/ProductDetailsViewModel.cs
public class ProductDetailsViewModel
{
    public int Id { get; set; }
    public string Name { get; set; } = string.Empty;
    public decimal Price { get; set; }
    public string CategoryName { get; set; } = string.Empty;
    public IEnumerable<ReviewDto> Reviews { get; set; } = Array.Empty<ReviewDto>();

    public string FormattedPrice => Price.ToString("C");
    public int ReviewCount => Reviews.Count();
}
```

**Step-by-step explanation:**

1. The ViewModel includes only read-only display properties.
2. Computed properties (`FormattedPrice`, `ReviewCount`) provide display formatting.

**Expected Output:** The Details view renders product details, formatted price, and review count.

**Why this output?** Display ViewModels shape data exactly as the view needs it, including computed values.

---

#### Example 3: Creating a Combined ViewModel

```csharp
// ViewModels/EditProductViewModel.cs
public class EditProductViewModel
{
    public int Id { get; set; }

    [Required]
    [StringLength(100)]
    public string Name { get; set; } = string.Empty;

    [Range(0.01, 10000)]
    public decimal Price { get; set; }

    public IEnumerable<SelectListItem> Categories { get; set; } = Array.Empty<SelectListItem>();
    public int SelectedCategoryId { get; set; }
}
```

**Expected Output:** The Edit form renders with pre-populated values and a category dropdown.

**Why this output?** The ViewModel carries both input fields and display-supporting data (categories).

### Real-World Cases

- **Registration forms** – `RegisterViewModel` with email, password, confirm password.
- **Product detail pages** – `ProductDetailsViewModel` aggregating product, category, and reviews.
- **Checkout flows** – `CheckoutViewModel` combining cart, address, and payment.
- **Admin edit forms** – `EditUserViewModel` with role dropdown and selected values.

### References

- ASP.NET Core MVC project structure (ViewModels) – https://github.com/OtakuDevs/Modular-ASPNETCore-MVC-Template
- Data annotations for validation – https://learn.microsoft.com/en-us/aspnet/core/mvc/models/validation

---

## 3. View Model Usage

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | ViewModel usage is the process of **populating ViewModels in controllers**, **passing them to views**, and **binding form submissions back** to ViewModels. |
| **Technical Definition** | In a controller action, a ViewModel is populated from domain models or services (via manual mapping or AutoMapper) and passed to `View(viewModel)`. The view declares `@model TViewModel`. For form submission, the ViewModel is bound by MVC's model binder (`[FromForm]` inference or explicit), validated via data annotations, and used to update the domain. Redirects (Post/Redirect/Get) often follow successful submissions. |
| **Beginner-Friendly Explanation** | ViewModel usage is **putting data into the ViewModel, sending it to the view, and reading it back from forms**. In a GET action, you fill the ViewModel from a database query and pass it to `View()`. In a POST action, the framework fills the ViewModel from the form fields, validates it, and you use it to update the data—then redirect. |

### Purposes (each begins with “To”)

- **To** pass view-specific data from controllers to views.
- **To** bind form submissions safely (only the ViewModel's properties are populated).
- **To** validate user input with view-specific rules.
- **To** populate dropdowns, checkboxes, and other UI controls with pre-selected values.
- **To** prevent **over-posting** attacks by binding only the ViewModel properties.
- **To** implement the **Post/Redirect/Get (PRG)** pattern reliably.

### Syntax Rules and Structure

#### General Syntax (GET Action)

```csharp
public async Task<IActionResult> Details(int id)
{
    var product = await _service.GetByIdAsync(id);
    if (product == null) return NotFound();

    var viewModel = new ProductDetailsViewModel
    {
        Id = product.Id,
        Name = product.Name,
        Price = product.Price,
        CategoryName = product.Category.Name
    };
    return View(viewModel);
}
```

#### General Syntax (POST Action)

```csharp
[HttpPost]
[ValidateAntiForgeryToken]
public async Task<IActionResult> Create(CreateProductViewModel model)
{
    if (!ModelState.IsValid) return View(model);

    var product = new Product { Name = model.Name, Price = model.Price };
    await _service.CreateAsync(product);

    return RedirectToAction(nameof(Index));
}
```

#### Component Breakdown

| Action Type | Flow |
|-------------|------|
| **GET (display)** | Fetch domain → map to ViewModel → `View(viewModel)`. |
| **GET (form)** | Populate supporting data (e.g., dropdowns) → `View(viewModel)`. |
| **POST** | Bind form → validate → map to domain → update → redirect. |
| **Form re-render** | If invalid, `return View(model)` to show errors with user input. |

#### Syntax Rules

- **View's `@model`** must match the type passed to `View(...)`.
- **Validation** – check `ModelState.IsValid` before mapping to domain.
- **PRG pattern** – redirect after successful POST to prevent duplicate submissions.
- **Antiforgery** – use `[ValidateAntiForgeryToken]` on POST actions and `<form asp-anti-forgery="true">` (default) in views.
- **Manual mapping** – explicit `new ProductDetailsViewModel { ... }`; **AutoMapper** uses `_mapper.Map<TViewModel>(domain)`.

#### Constraints and Limitations

- **Model binding errors** – invalid input keeps the user on the form; re-render with errors.
- **Navigation properties** – never include EF Core navigation properties directly in ViewModels (use projections).
- **Over-posting** – only bind properties present on the ViewModel; extra form fields are ignored.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: GET Action with Manual Mapping

```csharp
public async Task<IActionResult> Details(int id)
{
    var product = await _productService.GetByIdAsync(id);
    if (product == null) return NotFound();

    var viewModel = new ProductDetailsViewModel
    {
        Id = product.Id,
        Name = product.Name,
        Price = product.Price,
        CategoryName = product.Category.Name
    };
    return View(viewModel);
}
```

**Step-by-step explanation:**

1. Fetch the domain entity from the service.
2. Map to the ViewModel with only needed properties.
3. Pass the ViewModel to `View()`.

**Expected Output:** The Details view renders with the product data.

**Why this output?** The controller shapes the data for the view; the view renders it without knowing about the domain.

---

#### Example 2: POST Action with Validation and PRG

```csharp
[HttpPost]
[ValidateAntiForgeryToken]
public async Task<IActionResult> Create(CreateProductViewModel model)
{
    if (!ModelState.IsValid) return View(model);

    var product = new Product { Name = model.Name, Price = model.Price };
    await _productService.CreateAsync(product);

    return RedirectToAction(nameof(Index));
}
```

**Step-by-step explanation:**

1. The framework binds form fields to `CreateProductViewModel`.
2. `ModelState.IsValid` checks data annotations.
3. If invalid, re-render the form with errors.
4. If valid, map to the domain entity and save.
5. Redirect to `Index` (PRG pattern).

**Expected Output:**

| Scenario | Result |
|----------|--------|
| Invalid input | Form re-rendered with validation messages |
| Valid input | Redirect to Index (302) |

**Why this output?** The ViewModel isolates form binding; validation is checked before touching the domain.

---

#### Example 3: Using AutoMapper

```csharp
// Program.cs
builder.Services.AddAutoMapper(typeof(Program));

// Profiles/ProductProfile.cs
public class ProductProfile : Profile
{
    public ProductProfile()
    {
        CreateMap<Product, ProductDetailsViewModel>()
            .ForMember(dest => dest.CategoryName, opt => opt.MapFrom(src => src.Category.Name));
    }
}

// Controller action
public async Task<IActionResult> Details(int id)
{
    var product = await _productService.GetByIdAsync(id);
    if (product == null) return NotFound();
    var viewModel = _mapper.Map<ProductDetailsViewModel>(product);
    return View(viewModel);
}
```

**Expected Output:** The ViewModel is populated via the mapping profile.

**Why this output?** AutoMapper centralises and simplifies projection; the mapping profile documents the shape transformation.

---

#### Example 4: Form with Tag Helpers and Validation

```cshtml
@model CreateProductViewModel

<form asp-action="Create" method="post">
    <div class="mb-3">
        <label asp-for="Name" class="form-label"></label>
        <input asp-for="Name" class="form-control" />
        <span asp-validation-for="Name" class="text-danger"></span>
    </div>
    <div class="mb-3">
        <label asp-for="Price" class="form-label"></label>
        <input asp-for="Price" class="form-control" />
        <span asp-validation-for="Price" class="text-danger"></span>
    </div>
    <button type="submit" class="btn btn-primary">Create</button>
</form>
```

**Expected Output:** The form renders labels and inputs from the ViewModel; validation messages appear for invalid submissions.

**Why this output?** Tag Helpers (`asp-for`, `asp-validation-for`) read data annotations from the ViewModel to generate markup and validation hooks.

### Real-World Cases

- **CRUD forms** – Create/Edit actions with validation.
- **Detail pages** – Projections from joined data.
- **Dashboards** – Aggregated metrics mapped to a single ViewModel.
- **Wizard flows** – Multi-step forms using chained ViewModels.

### References

- Model binding in ASP.NET Core – https://learn.microsoft.com/en-us/aspnet/core/mvc/models/model-binding
- Model validation in ASP.NET Core – https://learn.microsoft.com/en-us/aspnet/core/mvc/models/validation
- ASP.NET Core Best Practices and Coding Standards – https://github.com/aios-labs/projectrules/blob/417776589dd52a6bd366bde067097938784f12c2/data/rules/manual/asp-net.md

---

## 4. View Model Best Practices

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | ViewModel best practices are guidelines for designing, naming, and using ViewModels that are **purpose-built, secure, testable, and maintainable**. |
| **Technical Definition** | Best practices include: creating **view-specific ViewModels** (input vs. display), applying validation and display attributes, keeping ViewModels **data-only**, using them to prevent **over-posting**, mapping from domain models in the controller, using `ModelState.IsValid` before touching the domain, separating input and display models, and avoiding navigation properties / database entities in ViewModels. |
| **Beginner-Friendly Explanation** | Best practices are the **rules of thumb** that keep ViewModels clean and safe. For example: make one ViewModel per view, put validation rules on the ViewModel (not the domain), don't mix business logic into ViewModels, and always check `ModelState` before saving. |

### Purposes (each begins with “To”)

- **To** keep ViewModels **view-specific** and focused, avoiding bloated classes.
- **To** prevent **over-posting** by limiting exposed properties.
- **To** enforce **view-specific validation** with data annotations.
- **To** decouple the UI from the domain model, allowing both to evolve independently.
- **To** maintain **testability** by keeping ViewModels as plain data objects.
- **To** keep **mapping** explicit and centralised (manual or AutoMapper).

### Guideline Checklist

| Guideline | Rationale | Example |
|-----------|-----------|---------|
| **One ViewModel per view** | Purpose-built shapes improve clarity. | `ProductDetailsViewModel`, `CreateProductViewModel` |
| **Separate input and display models** | Forms and read-only pages need different shapes. | `CreateProductViewModel` vs. `ProductDetailsViewModel` |
| **Put validation on the ViewModel** | UI rules differ from domain rules. | `[Required]` on `CreateProductViewModel.Name` |
| **Keep ViewModels data-only** | Business logic belongs in services. | No methods beyond computed properties |
| **Never expose navigation properties** | Prevents EF Core lazy-loading issues and leaks. | Use `CategoryName` (string) instead of `Category` entity |
| **Check `ModelState.IsValid` before touching the domain** | Ensures validation runs first. | `if (!ModelState.IsValid) return View(model);` |
| **Map explicitly in the controller** | Centralised, testable mapping. | Manual projection or AutoMapper profile |
| **Use `[Display]` and `[DataType]`** | Generates UI labels and formats. | `[Display(Name = "Product Name")]` |
| **Prefer nullable types for optional fields** | Reflects absence accurately. | `string? Description` |
| **Keep names consistent** | Improves discoverability. | `<Feature><Action>ViewModel` |

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Separate Input and Display ViewModels

```csharp
// ViewModels/Products/CreateProductViewModel.cs
public class CreateProductViewModel
{
    [Required, StringLength(100)]
    public string Name { get; set; } = string.Empty;

    [Range(0.01, 10000)]
    public decimal Price { get; set; }
}

// ViewModels/Products/ProductDetailsViewModel.cs
public class ProductDetailsViewModel
{
    public int Id { get; set; }
    public string Name { get; set; } = string.Empty;
    public decimal Price { get; set; }
    public string CategoryName { get; set; } = string.Empty;
    public string FormattedPrice => Price.ToString("C");
}
```

**Expected Output:** The Create form uses validation from `CreateProductViewModel`; the Details view uses `ProductDetailsViewModel` for display.

**Why this output?** Separation allows each view to have exactly the data and rules it needs without compromise.

---

#### Example 2: Avoiding Navigation Properties in ViewModels

```csharp
// ❌ BAD: Exposes EF Core entity with navigation property
public class ProductDetailsViewModel
{
    public Product Product { get; set; } = default!;   // Includes Category, Reviews, etc.
}

// ✅ GOOD: Exposes only flattened data
public class ProductDetailsViewModel
{
    public int Id { get; set; }
    public string Name { get; set; } = string.Empty;
    public decimal Price { get; set; }
    public string CategoryName { get; set; } = string.Empty;
}
```

**Expected Output:** The good version renders exactly what the view needs without lazy-loading or exposing internals.

**Why this output?** Flattening prevents navigation property issues and keeps the UI contract minimal.

---

#### Example 3: Manual Mapping in the Controller

```csharp
public async Task<IActionResult> Details(int id)
{
    var product = await _productService.GetByIdAsync(id);
    if (product == null) return NotFound();

    return View(new ProductDetailsViewModel
    {
        Id = product.Id,
        Name = product.Name,
        Price = product.Price,
        CategoryName = product.Category?.Name ?? "(uncategorized)"
    });
}
```

**Expected Output:** The view receives a fully populated ViewModel.

**Why this output?** Explicit mapping in the controller keeps the transformation visible and testable.

### Real-World Cases

- **E-commerce** – `ProductDetailsViewModel`, `CreateProductViewModel`, `CheckoutViewModel` kept separate.
- **Admin panels** – `EditUserViewModel` with role lists and selected values.
- **SaaS onboarding** – Multi-step `SignupViewModel` sequences with validated input.
- **CMS** – `EditArticleViewModel` combining content, tags, and category.

### References

- ASP.NET Core MVC project structure (ViewModels) – https://github.com/OtakuDevs/Modular-ASPNETCore-MVC-Template
- ASP.NET Core Best Practices and Coding Standards – https://github.com/aios-labs/projectrules/blob/417776589dd52a6bd366bde067097938784f12c2/data/rules/manual/asp-net.md
- Data annotations for validation – https://learn.microsoft.com/en-us/aspnet/core/mvc/models/validation
- DTOs vs ViewModels (Microsoft Q&A) – https://learn.microsoft.com/en-us/answers/questions/1406728/dto-vs-viewmodel-in-asp-net-core-mvc

---

## Summary Table: View Models at a Glance

| Concept | Primary Purpose | Key Takeaway | When to Use |
|---------|----------------|--------------|-------------|
| **View Models** | Shape data for a specific view | Purpose-built POCO, view-specific data, no domain logic | Every non-trivial view |
| **View Model Creation** | Design ViewModel classes | Separate input vs. display; apply validation/display attributes | When designing views and forms |
| **View Model Usage** | Populate and bind ViewModels | Map in controller, validate `ModelState`, PRG pattern | Every action returning a view |
| **Best Practices** | Maintainability, security | One ViewModel per view, flatten data, check `ModelState` before domain | All production applications |

---

## References (Full List)

- ASP.NET Core MVC project structure (ViewModels) – https://github.com/OtakuDevs/Modular-ASPNetCore-MVC-Template
- ASP.NET Core Best Practices and Coding Standards – https://github.com/aios-labs/projectrules/blob/417776589dd52a6bd366bde067097938784f12c2/data/rules/manual/asp-net.md
- Model binding in ASP.NET Core – https://learn.microsoft.com/en-us/aspnet/core/mvc/models/model-binding
- Model validation in ASP.NET Core – https://learn.microsoft.com/en-us/aspnet/core/mvc/models/validation
- Data annotations for validation – https://learn.microsoft.com/en-us/aspnet/core/mvc/models/validation
- DTOs vs ViewModels (Microsoft Q&A) – https://learn.microsoft.com/en-us/answers/questions/1406728/dto-vs-viewmodel-in-asp-net-core-mvc
- Views in ASP.NET Core MVC – https://learn.microsoft.com/en-us/aspnet/core/mvc/views/overview
- Controllers in ASP.NET Core MVC – https://learn.microsoft.com/en-us/aspnet/core/mvc/controllers/actions
- Tag Helpers in ASP.NET Core – https://learn.microsoft.com/en-us/aspnet/core/mvc/views/tag-helpers/intro
- Preventing over-posting in ASP.NET Core – https://learn.microsoft.com/en-us/aspnet/core/data/ef-mvc/crud#prevent-over-posting
- AutoMapper for .NET – https://docs.automapper.org/en/stable/