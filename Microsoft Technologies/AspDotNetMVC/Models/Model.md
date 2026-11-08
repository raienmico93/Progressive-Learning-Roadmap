# ASP.NET Core MVC Model Fundamentals: A Comprehensive Cheat Sheet

> **Scope & Version Note:** This cheat sheet targets **ASP.NET Core MVC** (controllers + Razor views) and applies equally to **Razor Pages**, which share the same model infrastructure. Examples use .NET 6–9 with minimal hosting (`Program.cs`). Model binding and validation behaviour is largely stable across these versions; deprecated or version-specific features are flagged. The **`Microsoft.Extensions.Validation`** package mentioned in some search results is **not supported for MVC or Razor Pages** — MVC continues to use `System.ComponentModel.DataAnnotations` and `ModelState`.

---

## Topic Overview

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | In MVC, a **model** is the component that represents the state of the application and encapsulates its business logic and operations. |
| **Technical Definition** | A model is a C# class (typically a **POCO** — Plain Old CLR Object) that defines the data shape, validation rules, and sometimes behaviour for a domain concept. In ASP.NET Core MVC, models participate in **model binding** (converting HTTP request data into .NET objects), **validation** (enforcing rules via data annotations or `IValidatableObject`), and **rendering** (strongly typed views consume models via `@model`). Models are distinct from **ViewModels** (shaped for a specific view) and **DTOs** (shaped for data transfer). |
| **Beginner‑Friendly Explanation** | A model is the **blueprint for your data**. If you are building a movie app, the `Movie` model says: “Every movie has an `Id`, a `Title`, a `ReleaseDate`, a `Genre`, and a `Price`.” It also says: “The title must be between 3 and 60 characters, and the price must be between 1 and 100.” The model is the single place where you define what the data looks like and what rules it must follow. |

### Key Characteristics

- **POCO‑based** – Models have no dependency on Entity Framework Core or any other framework; they are plain C# classes.
- **Declarative validation** – Rules are expressed with **data annotation attributes** (`[Required]`, `[StringLength]`, `[Range]`, `[RegularExpression]`, `[DataType]`) rather than imperative code.
- **Binding‑aware** – Properties are populated from form fields, route data, query strings, and request bodies by the **model binder**.
- **State‑carrying** – Models hold the data that flows between the controller and the view; `ModelState` tracks validation errors.
- **Separation of concerns** – The model **does not depend** on the view or controller, enabling independent compilation and testing.

### Prerequisites

- **.NET SDK** (6.0, 7.0, 8.0, or 9.0 recommended).
- **ASP.NET Core MVC** project (`dotnet new mvc`).
- **`System.ComponentModel.DataAnnotations`** namespace (included in the shared framework).
- Basic knowledge of **C# classes**, **properties**, and **attributes**.
- Familiarity with **HTTP verbs** and **form submissions** (helpful but not required).

### Related Programming Areas

- **Model binding** – The mechanism that populates model properties from HTTP requests.
- **Model validation** – The mechanism that enforces data annotation rules and populates `ModelState`.
- **Entity Framework Core** – Models often serve as EF Core entities, but POCO models have **no EF Core dependency**.
- **ViewModels and DTOs** – Shaped subsets of domain models used for specific views or API payloads.
- **Tag Helpers and HTML Helpers** – Generate form inputs and validation messages based on model metadata.

### Core Concepts / Features

1. **Models** – The role and responsibilities of models in MVC.
2. **Model Creation** – How to define POCO classes and integrate them with EF Core.
3. **Model Properties** – Data types, nullable reference types, and property conventions.
4. **Model Validation** – Data annotations, `ModelState`, and custom validation.
5. **Model Best Practices** – ViewModels/DTOs, over‑posting prevention, and maintainability.

Each concept is elaborated using a **uniform structure**: Definitions, Purposes, Syntax Rules & Structure, Annotated Code Examples, Real‑World Cases, and References.

---

## 1. Models (The Role and Responsibilities)

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | A model represents the **state of the application** and any **business logic or operations** it should perform. |
| **Technical Definition** | In ASP.NET Core MVC, the model is the **M** in MVC. It is a class (or set of classes) that defines the data shape, validation rules, and optionally domain behaviour. The model is **independent of the view and controller** — it does not reference them. Strongly typed views consume models via the `@model` directive, and controllers create/populate model instances to pass to views. |
| **Beginner‑Friendly Explanation** | The model is the **“data + rules” part of your app**. It knows what a `Movie` is, what fields it has, and what makes a `Movie` valid. It does **not** know how to display itself (that is the view’s job) or how to handle a button click (that is the controller’s job). |

### Purposes

- **To** define the **shape of application data** in a single, reusable place.
- **To** encapsulate **business rules and validation logic** that apply everywhere the data is used.
- **To** enable **strongly typed views** (`@model Movie`) so that views are checked at compile time.
- **To** provide a **binding target** for incoming HTTP request data.
- **To** keep the model **independent** of presentation and request‑handling concerns, improving testability and maintainability.

### Syntax Rules and Structure

#### General Syntax (A Simple Model)

```csharp
public class Movie
{
    public int Id { get; set; }
    public string? Title { get; set; }
    public DateTime ReleaseDate { get; set; }
    public string? Genre { get; set; }
    public decimal Price { get; set; }
}
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `public class Movie` | A plain C# class (POCO). |
| `Id` | Convention‑based primary key for EF Core. |
| `string?` | Nullable reference type — the property may be `null`. |
| `DateTime`, `decimal` | Value types — inherently required (no `[Required]` needed). |

#### Syntax Rules

- Models should be **POCO classes** with public properties.
- Property names become the **binding keys** (e.g., a form field named `Title` binds to `Movie.Title`).
- Models **do not** inherit from a base MVC class.
- Models can live in any folder (conventionally `Models/`).

#### Constraints and Limitations

- **Models are not ViewModels** – A domain model may expose properties that should not be bound from client input (see Best Practices).
- **No presentation logic** – Models should not contain HTML‑generating code.
- **Nullable reference types** – In nullable‑enabled projects, reference‑type properties should be marked `?` if they can be `null`; otherwise, the compiler and MVC may treat them as required.

### Annotated Complete Step‑by‑Step Code Examples

#### Example 1: A POCO Model Used in a Strongly Typed View

```csharp
// Models/Movie.cs
namespace MvcMovie.Models;

public class Movie
{
    public int Id { get; set; }
    public string? Title { get; set; }
    public DateTime ReleaseDate { get; set; }
    public string? Genre { get; set; }
    public decimal Price { get; set; }
}
```

```cshtml
@* Views/Movies/Details.cshtml *@
@model MvcMovie.Models.Movie

<h2>@Model.Title</h2>
<p>Release Date: @Model.ReleaseDate.ToShortDateString()</p>
<p>Price: @Model.Price.ToString("C")</p>
```

**Step‑by‑step explanation:**

1. The `Movie` class defines five properties.
2. The view uses `@model MvcMovie.Models.Movie` to declare that it is strongly typed to `Movie`.
3. The view accesses `Model.Title`, `Model.ReleaseDate`, and `Model.Price` directly.

**Expected Output (for a movie titled “Inception”, released 2010‑07‑16, priced at 9.99):**

```html
<h2>Inception</h2>
<p>Release Date: 7/16/2010</p>
<p>Price: $9.99</p>
```

**Why this output?** The controller passes a populated `Movie` instance to the view. The view’s `@model` directive enables compile‑time checking and direct property access.

### Real‑World Cases

- **E‑commerce:** A `Product` model represents the data and rules for products.
- **Content management:** An `Article` model holds the title, body, author, and publication date.
- **SaaS applications:** A `Subscription` model defines plan data and validation (e.g., renewal date must be in the future).

### References

- Overview of ASP.NET Core MVC (Model responsibilities) – https://learn.microsoft.com/en-us/aspnet/core/mvc/overview
- Part 4: Add a model to an ASP.NET Core MVC app – https://learn.microsoft.com/en-us/aspnet/core/tutorials/first-mvc-app/adding-model

---

## 2. Model Creation

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Model creation is the process of defining the C# classes that represent the application’s data and integrating them into the MVC application. |
| **Technical Definition** | Model creation involves writing **POCO classes** (Plain Old CLR Objects) with public properties, optionally decorating them with **data annotation attributes**, and registering them with **Entity Framework Core** (if persistence is required). The classes are typically placed in a `Models` folder. Scaffolding tools can generate CRUD controllers and views from a model. |
| **Beginner‑Friendly Explanation** | Model creation is **writing the blueprint classes** for your data. You create a `Movie` class with properties like `Title` and `Price`, add rules like “Title must be at least 3 characters,” and (if using a database) tell Entity Framework Core that this class maps to a `Movies` table. |

### Purposes

- **To** define a **single source of truth** for data shape and validation rules.
- **To** enable **EF Core migrations** and database schema generation from the model.
- **To** provide **binding targets** for form submissions and query parameters.
- **To** support **scaffolding** of CRUD controllers and views.
- **To** keep the domain model **framework‑agnostic** (POCO) so it can be reused across layers.

### Syntax Rules and Structure

#### General Syntax (POCO Model)

```csharp
using System.ComponentModel.DataAnnotations;

namespace MyApp.Models;

public class Movie
{
    public int Id { get; set; }

    [Required]
    [StringLength(60, MinimumLength = 3)]
    public string? Title { get; set; }

    [DataType(DataType.Date)]
    public DateTime ReleaseDate { get; set; }

    [Range(1, 100)]
    [DataType(DataType.Currency)]
    public decimal Price { get; set; }
}
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `namespace MyApp.Models` | Models typically live in a `Models` namespace. |
| `public int Id` | Primary key by EF Core convention. |
| `[Required]` | Data annotation — the property must have a value. |
| `[StringLength(60, MinimumLength = 3)]` | Maximum 60, minimum 3 characters. |
| `[DataType(DataType.Date)]` | Formatting hint for views (not validation). |
| `decimal Price` | Value type — inherently required. |

#### Syntax Rules (Creation Workflow)

1. **Create the class** in the `Models` folder.
2. **Add properties** with appropriate types.
3. **Apply data annotations** for validation and formatting.
4. **(Optional) Register with EF Core** via a `DbContext`:
   ```csharp
   public class MvcMovieContext : DbContext
   {
       public MvcMovieContext(DbContextOptions<MvcMovieContext> options)
           : base(options) { }
       public DbSet<Movie> Movies { get; set; }
   }
   ```
5. **Register the context** in `Program.cs`:
   ```csharp
   builder.Services.AddDbContext<MvcMovieContext>(options =>
       options.UseSqlServer(builder.Configuration.GetConnectionString("MvcMovieContext")));
   ```

#### Constraints and Limitations

- **POCO classes have no EF Core dependency** — they only define data properties.
- **Value types are inherently required** — `decimal`, `int`, `DateTime`, etc., do not need `[Required]`.
- **Nullable reference types** — In nullable‑enabled projects, `string?` allows `null`; `string` (non‑nullable) is treated as required by MVC validation.

### Annotated Complete Step‑by‑Step Code Examples

#### Example 1: Creating a POCO Model and a DbContext

```csharp
// Models/Movie.cs
using System.ComponentModel.DataAnnotations;

namespace MvcMovie.Models;

public class Movie
{
    public int Id { get; set; }

    [StringLength(60, MinimumLength = 3)]
    [Required]
    public string? Title { get; set; }

    [Display(Name = "Release Date")]
    [DataType(DataType.Date)]
    public DateTime ReleaseDate { get; set; }

    [Range(1, 100)]
    [DataType(DataType.Currency)]
    [Column(TypeName = "decimal(18, 2)")]
    public decimal Price { get; set; }

    [RegularExpression(@"^[A-Z]+[a-zA-Z\s]*$")]
    [Required]
    [StringLength(30)]
    public string? Genre { get; set; }
}
```

```csharp
// Data/MvcMovieContext.cs
using Microsoft.EntityFrameworkCore;
using MvcMovie.Models;

namespace MvcMovie.Data;

public class MvcMovieContext : DbContext
{
    public MvcMovieContext(DbContextOptions<MvcMovieContext> options)
        : base(options) { }

    public DbSet<Movie> Movies { get; set; }
}
```

**Step‑by‑step explanation:**

1. The `Movie` class is a **POCO** with data annotations for validation and formatting.
2. `[Column(TypeName = "decimal(18, 2)")]` ensures EF Core maps `Price` to the correct SQL Server type.
3. The `DbContext` exposes a `DbSet<Movie>` for querying and saving movies.
4. Registering the context in `Program.cs` (not shown) completes the integration.

**Expected Output (conceptual):** EF Core generates a `Movies` table with columns `Id`, `Title`, `ReleaseDate`, `Price`, and `Genre`, respecting the annotations.

**Why this output?** The model defines the schema through its properties and annotations; EF Core translates this into a database table.

### Real‑World Cases

- **Blog engine:** A `Post` model with `Title`, `Body`, `AuthorId`, and `PublishedAt`.
- **Inventory system:** A `Product` model with `SKU`, `Name`, `Price`, and `StockLevel`.
- **SaaS billing:** A `Subscription` model with `PlanId`, `RenewalDate`, and `Status`.

### References

- Part 4: Add a model to an ASP.NET Core MVC app – https://learn.microsoft.com/en-us/aspnet/core/tutorials/first-mvc-app/adding-model
- Entity Framework Core – https://learn.microsoft.com/en-us/ef/core/

---

## 3. Model Properties

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Model properties are the public fields on a model class that define the data the model carries. |
| **Technical Definition** | Model properties are C# properties (`get; set;`) that participate in **model binding** (their names are the binding keys), **validation** (data annotations are applied to them), and **rendering** (views access them via `@Model.PropertyName`). Property types determine how the model binder converts request data (simple types use `TypeConverter`/`TryParse`; complex types are bound recursively). |
| **Beginner‑Friendly Explanation** | Model properties are the **individual pieces of data** in your model. In a `Movie` model, `Title`, `ReleaseDate`, and `Price` are properties. Each property has a **type** (text, number, date) and can have **rules** (must be required, must be within a range). |

### Purposes

- **To** define the **data members** of a model with appropriate types.
- **To** provide **binding targets** whose names match form fields, route parameters, or query strings.
- **To** carry **validation metadata** through data annotations.
- **To** enable **strongly typed access** from views (`@Model.Price`).
- **To** express **nullability** explicitly using nullable reference types.

### Syntax Rules and Structure

#### General Syntax

```csharp
[Annotations]
public <Type> <PropertyName> { get; set; }
```

**Component breakdown:**

| Part | Description |
|------|-------------|
| `[Annotations]` | Zero or more data annotation attributes. |
| `<Type>` | Value type (`int`, `decimal`, `DateTime`) or reference type (`string?`, `string`). |
| `<PropertyName>` | Becomes the binding key and the view accessor. |

#### Common Data Types and Their Binding Behaviour

| Type | Binding Source | Notes |
|------|----------------|-------|
| `int`, `decimal`, `DateTime` | Form, route, query string | Simple types; converted via `TypeConverter` or `TryParse`. |
| `string` / `string?` | Form, route, query string | `string?` allows `null`; `string` is treated as required in nullable‑enabled projects. |
| `bool` | Form (checkbox) | Checkbox helpers emit a hidden fallback so `false` is posted. |
| `IFormFile` | Form (file upload) | Bound only when the target implements `IFormFile` or `IEnumerable<IFormFile>`. |
| Complex types | JSON body (`[FromBody]`) or nested form fields | Bound recursively. |

#### Syntax Rules

- Property **names are case‑insensitive** during binding.
- **`[BindProperty]`** (Razor Pages) or action parameters (MVC) control which properties are bound. By default, properties are **not bound on GET requests**; use `[BindProperty(SupportsGet = true)]` explicitly if needed.
- **`[FromQuery]`, `[FromRoute]`, `[FromForm]`, `[FromBody]`, `[FromHeader]`** can override the default binding source.

#### Constraints and Limitations

- **Over‑posting risk:** Binding directly to domain models can allow clients to set properties they should not (e.g., `IsAdmin`). Use ViewModels/DTOs or `[Bind]` to restrict binding.
- **Empty strings vs. null:** The model binder may convert empty strings to `null` for nullable types; validate accordingly.
- **Collection binding:** Complex collection types require indexed field names (e.g., `Items[0].Name`).

### Annotated Complete Step‑by‑Step Code Examples

#### Example 1: Value Types vs. Reference Types in Binding

```csharp
public class Product
{
    public int Id { get; set; }                 // Value type — required by default
    public string? Name { get; set; }           // Nullable reference — optional
    public string Description { get; set; }     // Non-nullable reference — required in nullable-enabled projects
    public decimal Price { get; set; }          // Value type — required
}
```

**Expected Binding Behaviour:**

| Form Field | `Name` | `Description` | `Price` |
|------------|--------|---------------|---------|
| `Name` = "Laptop" | Bound | — | — |
| `Description` absent | — | `null` | — |
| `Price` = "abc" | — | — | Binding fails; `ModelState` error |

**Why this output?** `string?` permits `null`; non‑nullable `string` is treated as required by MVC validation in nullable‑enabled projects. `decimal` binding fails for non‑numeric input, adding a `ModelState` error.

---

#### Example 2: Binding Source Override with `[FromQuery]`

```csharp
public class SearchModel
{
    [FromQuery(Name = "q")]
    public string? Query { get; set; }

    [FromQuery(Name = "page")]
    public int Page { get; set; } = 1;
}
```

**Controller action:**

```csharp
public IActionResult Search([FromQuery] SearchModel model)
{
    // ...
}
```

**Expected Output:** A request to `/search?q=laptop&page=2` binds `Query = "laptop"` and `Page = 2`.

**Why this output?** The `[FromQuery]` attribute forces binding from query‑string parameters, and the `Name` property maps `q` to `Query`.

### Real‑World Cases

- **Search filters:** A `SearchModel` with `[FromQuery]` properties for keyword, category, and page number.
- **File uploads:** A model property of type `IFormFile` for uploading images.
- **API payloads:** A DTO with `[FromBody]` for JSON deserialization.

### References

- Model binding in ASP.NET Core – https://learn.microsoft.com/en-us/aspnet/core/mvc/models/model-binding
- Model properties and data annotations – https://learn.microsoft.com/en-us/aspnet/core/mvc/models/validation

---

## 4. Model Validation

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Model validation is the process of checking that model data satisfies the rules defined by data annotations and custom validation logic. |
| **Technical Definition** | ASP.NET Core MVC performs validation during **model binding**. Validation attributes (`[Required]`, `[StringLength]`, `[Range]`, `[RegularExpression]`, etc.) are evaluated, and errors are recorded in **`ModelState`**. Controllers (and Razor Pages) inspect `ModelState.IsValid` to decide whether to proceed with business logic or re‑render the view with error messages. Validation can be extended with `IValidatableObject` and custom `ValidationAttribute` subclasses. |
| **Beginner‑Friendly Explanation** | Model validation is the **“quality check”** for your data. Before your app saves a movie, it checks: “Is the title filled in? Is it between 3 and 60 characters? Is the price between 1 and 100?” If any rule fails, the app shows an error message and does not save the data. |

### Purposes

- **To** enforce **data integrity** before data reaches the database.
- **To** provide **automatic, declarative rules** that are applied consistently across the application.
- **To** populate **`ModelState`** so that views can display field‑level error messages.
- **To** enable **client‑side validation** (via `jquery.validate` unobtrusive) in addition to server‑side validation.
- **To** support **custom validation logic** for complex rules (e.g., “end date must be after start date”).

### Syntax Rules and Structure

#### General Syntax (Data Annotations)

```csharp
[Required(ErrorMessage = "Title is required.")]
[StringLength(60, MinimumLength = 3, ErrorMessage = "Title must be 3–60 characters.")]
public string? Title { get; set; }

[Range(1, 100, ErrorMessage = "Price must be between 1 and 100.")]
public decimal Price { get; set; }
```

**Component breakdown:**

| Attribute | Purpose |
|-----------|---------|
| `[Required]` | The property must have a non‑null, non‑empty value. |
| `[StringLength(max, MinimumLength = min)]` | Restricts string length. |
| `[Range(min, max)]` | Restricts numeric range. |
| `[RegularExpression(pattern)]` | Restricts to a regex pattern. |
| `[DataType(DataType.X)]` | Formatting hint (not validation). |

#### General Syntax (Controller Validation Check)

```csharp
[HttpPost]
public IActionResult Create(Movie movie)
{
    if (!ModelState.IsValid)
    {
        return View(movie);   // Re‑render form with errors
    }

    _context.Movies.Add(movie);
    _context.SaveChanges();
    return RedirectToAction(nameof(Index));
}
```

**Component breakdown:**

| Part | Description |
|------|-------------|
| `if (!ModelState.IsValid)` | Checks whether any validation rule failed. |
| `return View(movie)` | Re‑renders the view with the posted data and error messages. |
| `RedirectToAction` | PRG (Post‑Redirect‑Get) pattern after successful save. |

#### Syntax Rules

- Validation runs **automatically** during model binding; you do not call a validation method manually.
- **Value types** (`int`, `decimal`, `DateTime`) are inherently required and do not need `[Required]`.
- `ModelState.IsValid` is `true` only when **all** properties pass validation.
- For **Web API controllers** decorated with `[ApiController]`, automatic 400 responses are returned when `ModelState` is invalid, so an explicit check is not required.

#### Constraints and Limitations

- **`[Required]` on non‑nullable value types** is redundant but harmless.
- **White‑space strings** satisfy `[Required]` unless you add `[RegularExpression]` or a custom rule.
- **Client‑side validation** requires jQuery validation scripts and `_ValidationScriptsPartial`.
- **Custom `IValidatableObject`** validation runs **after** attribute‑based validation.

### Annotated Complete Step‑by‑Step Code Examples

#### Example 1: Attribute‑Based Validation with Error Messages

```csharp
public class Movie
{
    public int Id { get; set; }

    [Required(ErrorMessage = "Title is required.")]
    [StringLength(60, MinimumLength = 3, ErrorMessage = "Title must be 3–60 characters.")]
    public string? Title { get; set; }

    [Range(1, 100, ErrorMessage = "Price must be between 1 and 100.")]
    [DataType(DataType.Currency)]
    public decimal Price { get; set; }

    [RegularExpression(@"^[A-Z]+[a-zA-Z\s]*$", ErrorMessage = "Genre must start with an uppercase letter.")]
    [StringLength(30)]
    public string? Genre { get; set; }
}
```

**Controller:**

```csharp
[HttpPost]
public IActionResult Create(Movie movie)
{
    if (!ModelState.IsValid)
    {
        return View(movie);
    }
    _context.Movies.Add(movie);
    _context.SaveChanges();
    return RedirectToAction(nameof(Index));
}
```

**Expected Output (when invalid data is posted):**

```html
<span class="text-danger field-validation-error" data-valmsg-for="Title">Title must be 3–60 characters.</span>
<span class="text-danger field-validation-error" data-valmsg-for="Price">Price must be between 1 and 100.</span>
```

**Why this output?** The model binder applies each attribute. Failed rules add errors to `ModelState` with the specified `ErrorMessage`. The view’s `ValidationMessageFor` helpers display them.

---

#### Example 2: `IValidatableObject` for Cross‑Property Validation

```csharp
public class Event : IValidatableObject
{
    public DateTime StartDate { get; set; }
    public DateTime EndDate { get; set; }

    public IEnumerable<ValidationResult> Validate(ValidationContext validationContext)
    {
        if (EndDate <= StartDate)
        {
            yield return new ValidationResult(
                "End date must be after start date.",
                new[] { nameof(EndDate) });
        }
    }
}
```

**Expected Output:** If `EndDate` is before `StartDate`, a model‑level (or property‑level) error is added to `ModelState`.

**Why this output?** `IValidatableObject.Validate` runs after attribute validation and can compare multiple properties, enabling rules that no single attribute can express.

### Real‑World Cases

- **Registration forms:** `[Required]`, `[EmailAddress]`, `[StringLength]` on username and password fields.
- **Booking systems:** `IValidatableObject` ensures check‑out date is after check‑in date.
- **Product pricing:** `[Range]` prevents negative or excessively high prices.

### References

- Model validation in ASP.NET Core MVC and Razor Pages – https://learn.microsoft.com/en-us/aspnet/core/mvc/models/validation
- Part 9: Add validation to an ASP.NET Core MVC app – https://learn.microsoft.com/en-us/aspnet/core/tutorials/first-mvc-app/validation
- `IValidatableObject` – https://learn.microsoft.com/en-us/dotnet/api/system.componentmodel.dataannotations.ivalidatableobject

---

## 5. Model Best Practices

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Model best practices are guidelines for designing models that are secure, maintainable, testable, and aligned with the separation of concerns. |
| **Technical Definition** | Best practices encompass using **ViewModels/DTOs** instead of exposing domain entities directly to views or APIs, preventing **over‑posting** with `[Bind]` or dedicated input models, keeping models **POCO and framework‑agnostic**, leveraging **data annotations for DRY validation** across MVC and EF Core, and **validating `ModelState`** before executing business logic. |
| **Beginner‑Friendly Explanation** | Best practices are the **“rules of thumb”** that keep your models clean and safe. For example: don’t let users set an `IsAdmin` flag by posting a form; create a separate `RegisterViewModel` with only the fields the user should provide. |

### Purposes

- **To** prevent **over‑posting attacks** by restricting which properties can be bound.
- **To** keep **domain models decoupled** from view‑specific concerns.
- **To** enable **DRY validation** — rules are defined once on the model and enforced by both MVC and EF Core.
- **To** improve **testability** by keeping models as plain, injectable classes.
- **To** optimise **payload size** and **security** by exposing only the required data in APIs.

### Syntax Rules and Structure (Guideline Checklist)

| Area | Do | Don’t |
|------|----|-------|
| **Binding targets** | Use a **ViewModel** or **DTO** for `[BindProperty]` / action parameters. | Bind directly to domain entities with sensitive properties. |
| **Over‑posting** | Use `[Bind("Prop1,Prop2")]` or a dedicated input model. | Expose `IsAdmin`, `Role`, or `RowVersion` to client binding. |
| **Validation location** | Declare rules with data annotations on the model. | Duplicate validation logic in controllers and views. |
| **`ModelState`** | Always check `ModelState.IsValid` before saving. | Proceed with invalid data. |
| **POCO purity** | Keep models free of EF Core or MVC dependencies. | Add `DbContext` references or HTML‑generating methods to models. |
| **API DTOs** | Use DTOs to shape API responses and hide internal fields. | Serialise domain entities directly in APIs. |
| **Nullable types** | Use `string?` for optional strings in nullable‑enabled projects. | Mark required strings as nullable or omit `[Required]` incorrectly. |
| **Custom validation** | Use `IValidatableObject` for cross‑property rules. | Write complex regexes for rules that need multiple fields. |

### Constraints and Limitations

- **ViewModels are not always necessary** for simple CRUD tutorials, but become essential as applications grow.
- **`[Bind]`** restricts binding but does **not** replace validation; always check `ModelState`.
- **DTO mapping** introduces a small overhead (manual mapping or AutoMapper) but pays off in security and maintainability.
- **Data annotations are not a substitute for database constraints**; use both where appropriate.

### Annotated Code Example (Demonstrating Best Practices)

```csharp
// Domain entity (not bound directly)
public class Customer
{
    public int Id { get; set; }
    public string Name { get; set; }
    public bool IsAdmin { get; set; }       // Sensitive — should not be bound
    public DateTime CreatedAt { get; set; }
}

// Input ViewModel / DTO (bound from the form)
public class CustomerInputModel
{
    [Required, StringLength(50)]
    public string Name { get; set; }

    [Required, EmailAddress]
    public string Email { get; set; }
}

// Controller
[HttpPost]
public IActionResult Create(CustomerInputModel input)
{
    if (!ModelState.IsValid)
        return View(input);

    var customer = new Customer
    {
        Name = input.Name,
        IsAdmin = false,                    // Set server‑side, never bound
        CreatedAt = DateTime.UtcNow
    };
    _context.Customers.Add(customer);
    _context.SaveChanges();
    return RedirectToAction(nameof(Index));
}
```

**Why this follows best practices:**

- **`CustomerInputModel`** exposes only the fields the user should provide, preventing over‑posting of `IsAdmin`.
- **Validation** is declared once on the input model and enforced by `ModelState`.
- **The domain entity** remains free of binding concerns and sensitive properties.
- **`ModelState.IsValid`** is checked before any business logic runs.

### Real‑World Cases

- **Public APIs:** DTOs prevent leaking internal fields (e.g., `PasswordHash`) and reduce payload size.
- **Admin vs. public forms:** Different ViewModels for admin and public user creation avoid accidental privilege escalation.
- **Multi‑step wizards:** Each step uses a focused ViewModel, simplifying validation and state management.

### References

- Tutorial: Create a web API with ASP.NET Core (DTOs) – https://learn.microsoft.com/en-us/aspnet/core/tutorials/first-web-api
- Over‑posting and `[Bind]` – https://learn.microsoft.com/en-us/aspnet/core/mvc/models/model-binding
- Model validation in ASP.NET Core MVC – https://learn.microsoft.com/en-us/aspnet/core/mvc/models/validation

---

## Summary Table: Core Concepts at a Glance

| Concept | Primary Role | Key Syntax / Tool | When to Use |
|---------|--------------|-------------------|-------------|
| **Models** | Represent state + business logic | POCO class, `@model` directive | Every MVC/Razor Pages app |
| **Model Creation** | Define data shape and EF Core mapping | POCO + data annotations + `DbContext` | When adding a new entity |
| **Model Properties** | Define data members and binding targets | `{ get; set; }`, nullable reference types | Every model |
| **Model Validation** | Enforce data rules | `[Required]`, `[StringLength]`, `[Range]`, `ModelState.IsValid` | Every form submission |
| **Model Best Practices** | Security, maintainability, DRY | ViewModels/DTOs, `[Bind]`, `ModelState` checks | Production applications |

---

## References (Full List)

- Overview of ASP.NET Core MVC – https://learn.microsoft.com/en-us/aspnet/core/mvc/overview
- Part 4: Add a model to an ASP.NET Core MVC app – https://learn.microsoft.com/en-us/aspnet/core/tutorials/first-mvc-app/adding-model
- Model binding in ASP.NET Core – https://learn.microsoft.com/en-us/aspnet/core/mvc/models/model-binding
- Model validation in ASP.NET Core MVC and Razor Pages – https://learn.microsoft.com/en-us/aspnet/core/mvc/models/validation
- Part 9: Add validation to an ASP.NET Core MVC app – https://learn.microsoft.com/en-us/aspnet/core/tutorials/first-mvc-app/validation
- `IValidatableObject` – https://learn.microsoft.com/en-us/dotnet/api/system.componentmodel.dataannotations.ivalidatableobject
- Entity Framework Core – https://learn.microsoft.com/en-us/ef/core/
- Tutorial: Create a web API with ASP.NET Core (DTOs) – https://learn.microsoft.com/en-us/aspnet/core/tutorials/first-web-api