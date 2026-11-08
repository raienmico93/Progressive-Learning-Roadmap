# ASP.NET Core MVC View Models: A Comprehensive Cheat Sheet

> **Scope & Version Note:** This cheat sheet targets **ASP.NET Core MVC** and **Razor Pages**. Examples use .NET 6–9 with minimal hosting. The ViewModel pattern is framework-agnostic; the implementation details (Tag Helpers, `[BindProperty]`, `ModelState`) are specific to ASP.NET Core MVC/Razor Pages. **AutoMapper licensing:** Starting with AutoMapper 15.0, a commercial license is required for use; this is flagged explicitly in the AutoMapper section. Alternatives such as **Mapperly** (a source generator) are noted for teams that cannot adopt the commercial license.

---

## Topic Overview

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | A ViewModel is a class shaped specifically for the needs of a single view (or a small group of related views), containing only the data and validation rules that the view requires. |
| **Technical Definition** | A ViewModel is a POCO (Plain Old CLR Object) that serves as the **strongly typed model** for a Razor view. In ASP.NET Core MVC, the controller creates and populates ViewModel instances from domain entities (or other sources) and passes them to the view via `return View(viewModel)`. The view declares its type with `@model MyViewModel`. ViewModels are distinct from **domain models** (business entities) and **DTOs** (data-transfer objects for APIs), although the terms are sometimes used interchangeably in practice. The official guidance recommends ViewModels with **little or no behaviour** (methods) and **no dependencies on ASP.NET Core types** such as `Session`, `ViewBag`, or `TempData`. |
| **Beginner-Friendly Explanation** | A ViewModel is a **custom-made container for a page’s data**. Instead of passing a complex database entity directly to a view—exposing fields the page doesn’t need and creating security risks—you create a simple class that holds exactly what the page should display and accept. For example, a product page might need the product name, price, and a dropdown list of categories. The ViewModel contains those three things and nothing else. |

### Key Characteristics

- **View-specific** – Shaped for one view’s requirements; may combine data from multiple entities.
- **POCO / behaviour-light** – Recommended to have few or no methods, and no dependency on ASP.NET Core infrastructure.
- **Strongly typed** – Enables compile-time checking in views, unlike `ViewData`/`ViewBag`.
- **Security boundary** – Exposing only safe properties prevents **over-posting** (mass assignment) attacks.
- **Validation carrier** – Data annotation attributes applied to ViewModel properties drive both client-side and server-side validation for the view.

### Prerequisites

- .NET SDK (6.0+ recommended), ASP.NET Core MVC or Razor Pages project.
- Basic understanding of **controllers**, **Razor views**, and **model binding**.
- Familiarity with **data annotations** (for validation) and **Entity Framework Core** (if mapping from database entities).

### Related Programming Areas

- **Model binding** – ViewModels are the primary binding targets for form submissions.
- **Validation** – Validation attributes on ViewModels populate `ModelState`.
- **Tag Helpers / HTML Helpers** – `asp-for`, `asp-validation-for`, `EditorFor`, etc., read ViewModel metadata.
- **DTOs** – ViewModels for APIs are often called DTOs; the same shaping principles apply.
- **AutoMapper / Mapperly** – Tools that automate the mapping between domain entities and ViewModels.

### Core Concepts / Features

1. **View models** – The concept and role of ViewModels.
2. **View model creation** – Designing and structuring ViewModel classes.
3. **View model mapping** – Converting between domain entities and ViewModels (manual and automated).
4. **AutoMapper** – The popular mapping library, its setup, usage, licensing, and alternatives.
5. **View model best practices** – Security, maintainability, and performance guidelines.

Each concept is elaborated below using a uniform structure: **Definitions**, **Purposes**, **Syntax Rules & Structure**, **Annotated Code Examples**, **Real-World Cases**, and **References**.

---

## 1. View Models (The Concept and Role)

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | A ViewModel is a class designed to carry exactly the data a view needs, in the shape the view needs it. |
| **Technical Definition** | In ASP.NET Core MVC, a ViewModel is a POCO used as the `@model` type of a Razor view. It is populated by the controller (or Razor Page handler) and passed to the view. The official MVC overview states: “Strongly typed views typically use ViewModel types designed to contain the data to display on that view. The controller creates and populates ViewModel instances from the model”. ViewModels are recommended to be **POCOs with little or no behaviour**, stored in a `ViewModels` folder or similar. |
| **Beginner-Friendly Explanation** | A ViewModel is the **“shopping list” for a page**. The controller goes to the database, picks up the ingredients (data), and arranges them on a tray (the ViewModel) in exactly the way the page needs. The page then displays the tray. The page never sees the messy kitchen (database schema, entity navigation properties, internal flags). |

### Purposes

- **To** provide a **strongly typed** model for views, enabling compile-time checking and IntelliSense.
- **To** shape data for the UI, exposing only the fields the view needs and excluding sensitive or irrelevant properties.
- **To** prevent **over-posting** (mass assignment) attacks by not exposing domain entities directly to form binding.
- **To** combine data from multiple entities into a single object when a view needs information from several sources.
- **To** carry view-specific validation rules (e.g., “confirm password”) that do not belong on the domain entity.
- **To** keep views free of business logic and database access, honouring the separation of concerns.

### Syntax Rules and Structure

#### General Syntax (ViewModel Class)

```csharp
public class ProductViewModel
{
    public int Id { get; set; }
    public string Name { get; set; }
    public decimal Price { get; set; }
    public string CategoryName { get; set; }
}
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `ProductViewModel` | A POCO class, typically in a `ViewModels` folder. |
| Public properties | Only the data the view needs; validation attributes are applied here. |

#### General Syntax (Controller Passing a ViewModel)

```csharp
public IActionResult Details(int id)
{
    var product = _context.Products
        .Where(p => p.Id == id)
        .Select(p => new ProductViewModel
        {
            Id = p.Id,
            Name = p.Name,
            Price = p.Price,
            CategoryName = p.Category.Name
        })
        .FirstOrDefault();

    if (product == null) return NotFound();
    return View(product);
}
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `Select(...)` | Projects directly into the ViewModel (efficient database query). |
| `return View(product)` | Passes the ViewModel to the strongly typed view. |

#### General Syntax (View Declaration)

```cshtml
@model ProductViewModel

<h2>@Model.Name</h2>
<p>Price: @Model.Price.ToString("C")</p>
<p>Category: @Model.CategoryName</p>
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `@model ProductViewModel` | Declares the view’s model type. |
| `Model.Property` | Strongly typed access; compile-time checked. |

#### Syntax Rules

- ViewModels are **plain classes**; no base class or interface is required.
- The `@model` directive (lowercase `m`) declares the type; `@Model` (uppercase `M`) accesses the instance.
- ViewModels should **not** reference `ViewData`, `ViewBag`, `Session`, or `HttpContext`.
- Validation attributes (`[Required]`, `[StringLength]`, etc.) are applied to ViewModel properties, not to the domain entity, for view-specific rules.

#### Constraints and Limitations

- **ViewModels are not domain models** – they should not contain business logic or database access code.
- **ViewModels are per-view** – a ViewModel designed for one view may not fit another; avoid trying to make a single ViewModel serve many views.
- **Not every view requires a ViewModel** – simple read-only views that display a single entity faithfully can sometimes use the entity directly, though security and coupling concerns usually favour a ViewModel.

### Annotated Complete Step-by-Step Code Examples

#### Example 1: A Read-Only ViewModel

```csharp
// ViewModels/ProductDetailsViewModel.cs
public class ProductDetailsViewModel
{
    public int Id { get; set; }
    public string Name { get; set; } = null!;
    public decimal Price { get; set; }
    public string CategoryName { get; set; } = null!;
}

// Controller
public async Task<IActionResult> Details(int id)
{
    var vm = await _context.Products
        .Where(p => p.Id == id)
        .Select(p => new ProductDetailsViewModel
        {
            Id = p.Id,
            Name = p.Name,
            Price = p.Price,
            CategoryName = p.Category.Name
        })
        .FirstOrDefaultAsync();

    if (vm == null) return NotFound();
    return View(vm);
}
```

```cshtml
@model ProductDetailsViewModel
<h1>@Model.Name</h2>
<p>Price: @Model.Price.ToString("C")</p>
<p>Category: @Model.CategoryName</p>
```

**Step-by-step explanation:**

1. The ViewModel exposes only the fields the details page needs.
2. The controller projects directly into the ViewModel using LINQ `Select`, avoiding loading unnecessary entity data.
3. The view uses `@model` to declare the type and accesses properties strongly typed.

**Expected Output:** A page displaying the product name, formatted price, and category name.

**Why this output?** The ViewModel shapes the data for the view, and the `Select` projection ensures only the required columns are queried.

---

#### Example 2: A Form (Input) ViewModel

```csharp
public class ProductCreateViewModel
{
    [Required, StringLength(100)]
    public string Name { get; set; } = null!;

    [Range(1, 10000)]
    public decimal Price { get; set; }

    [Required]
    public int CategoryId { get; set; }

    public IEnumerable<SelectListItem> Categories { get; set; } = Enumerable.Empty<SelectListItem>();
}
```

```csharp
[HttpPost]
public async Task<IActionResult> Create(ProductCreateViewModel input)
{
    if (!ModelState.IsValid)
    {
        input.Categories = await GetCategoriesAsync();
        return View(input);
    }

    var product = new Product { Name = input.Name, Price = input.Price, CategoryId = input.CategoryId };
    _context.Products.Add(product);
    await _context.SaveChangesAsync();
    return RedirectToAction(nameof(Index));
}
```

**Expected Output:** A form with validation attributes; invalid submissions re-display the form with errors and the category dropdown repopulated.

**Why this output?** The ViewModel carries both the form fields and the data needed to render the form (dropdown items). Validation attributes on the ViewModel drive `ModelState`.

### Real-World Cases

- **Product catalogue pages:** A `ProductListViewModel` containing a page of products, total count, and pagination metadata.
- **Checkout flows:** A `CheckoutViewModel` combining cart items, shipping address, and payment method options.
- **Admin dashboards:** A `DashboardViewModel` aggregating counts, recent activity, and chart data from multiple services.

### References

- ASP.NET Core MVC overview (Model responsibilities, ViewModel types) – https://learn.microsoft.com/en-us/aspnet/core/mvc/overview
- Views in ASP.NET Core MVC (recommended POCO ViewModels) – https://learn.microsoft.com/en-us/aspnet/core/mvc/views/overview
- Do you use ViewModel instead of ViewData? (SSW Rules) – https://www.ssw.com.au/rules/do-you-use-view-models-instead-of-viewdata

---

## 2. View Model Creation

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | ViewModel creation is the process of designing and defining the classes that will carry data to and from views, choosing the right properties, validation rules, and organisational structure. |
| **Technical Definition** | Creating a ViewModel involves defining a POCO class with public properties that map to the view’s data requirements. The class typically resides in a `ViewModels` folder (or `Models` folder for smaller projects) and may include data annotation attributes for validation and display metadata. For form scenarios, the ViewModel includes properties for user input and may include `SelectListItem` collections for dropdowns. Official guidance recommends **POCOs with little or no behaviour**. |
| **Beginner-Friendly Explanation** | Creating a ViewModel means **writing a new class** that fits a page like a glove. You list every piece of data the page needs to show or collect, give each piece a sensible type, and add validation rules where necessary. You keep it simple—just properties, no complex methods. |

### Purposes

- **To** define a clear contract between the controller and the view.
- **To** include exactly the properties the view needs—no more, no less.
- **To** attach validation rules that are specific to the view’s input (e.g., “confirm password”).
- **To** include supporting data for the view (e.g., dropdown lists, lookup values).
- **To** keep domain entities untouched by view-specific concerns.

### Syntax Rules and Structure

#### General Syntax (Basic Creation)

```csharp
public class <ViewName>ViewModel
{
    // Properties the view needs
    public <Type> <PropertyName> { get; set; }

    // Optional validation attributes
    [Required]
    public <Type> <PropertyName> { get; set; }
}
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| Class name | Conventionally ends with `ViewModel` or `VM`; matches the view or feature. |
| Properties | Only the data required by the view. |
| Validation attributes | Applied directly to properties. |

#### General Syntax (Form ViewModel with Lookup Data)

```csharp
public class <Action>ViewModel
{
    [Required]
    public <InputType> <InputProperty> { get; set; }

    public IEnumerable<SelectListItem> <LookupProperty> { get; set; }
        = Enumerable.Empty<SelectListItem>();
}
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| Input properties | Bound from the form. |
| `SelectListItem` collection | Populated by the controller for dropdowns. |

#### Syntax Rules

- **Name by view/action** – `EditViewModel`, `CreateViewModel`, `DetailsViewModel`, or feature-based names.
- **Public properties with `get`/`set`** – required for model binding and view access.
- **Validation attributes** – placed on ViewModel properties, not domain entities, for view-specific rules.
- **No behaviour** – methods are discouraged; keep the class as a data carrier.
- **No ASP.NET Core dependencies** – do not reference `HttpContext`, `Session`, `ViewBag`, etc..

#### Constraints and Limitations

- **Avoid ViewModel explosion** – not every view needs a unique ViewModel; reuse when the shape is identical, but do not force a single ViewModel to serve unrelated views.
- **Do not put mapping logic in the constructor** – mapping is a separate concern; constructor mapping becomes unwieldy as complexity grows.
- **Nullable reference types** – use `string?` for optional strings; non-nullable `string` is treated as required in nullable-enabled projects.

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Creating a Form ViewModel with Validation

```csharp
public class RegisterViewModel
{
    [Required, StringLength(50, MinimumLength = 3)]
    [Display(Name = "Username")]
    public string Username { get; set; } = null!;

    [Required, EmailAddress]
    public string Email { get; set; } = null!;

    [Required, MinLength(8), DataType(DataType.Password)]
    public string Password { get; set; } = null!;

    [DataType(DataType.Password)]
    [Compare(nameof(Password), ErrorMessage = "Passwords do not match.")]
    public string ConfirmPassword { get; set; } = null!;
}
```

**Step-by-step explanation:**

1. The class defines exactly the four fields the registration form needs.
2. Validation attributes enforce rules at the ViewModel level.
3. `[Compare]` handles password confirmation—a view-specific rule that does not belong on the domain `User` entity.

**Expected Output:** A registration form with client-side and server-side validation for each field.

**Why this output?** The ViewModel carries both the data shape and the validation rules; Tag Helpers read these attributes to generate validation attributes and error messages.

---

#### Example 2: ViewModel with Lookup Data for a Dropdown

```csharp
public class ProductEditViewModel
{
    public int Id { get; set; }

    [Required, StringLength(100)]
    public string Name { get; set; } = null!;

    [Range(1, 10000)]
    public decimal Price { get; set; }

    [Required, Display(Name = "Category")]
    public int CategoryId { get; set; }

    public IEnumerable<SelectListItem> Categories { get; set; }
        = Enumerable.Empty<SelectListItem>();
}
```

```csharp
public async Task<IActionResult> Edit(int id)
{
    var product = await _context.Products.FindAsync(id);
    if (product == null) return NotFound();

    var vm = new ProductEditViewModel
    {
        Id = product.Id,
        Name = product.Name,
        Price = product.Price,
        CategoryId = product.CategoryId,
        Categories = await GetCategorySelectListAsync()
    };
    return View(vm);
}
```

**Expected Output:** An edit form with a populated category dropdown.

**Why this output?** The ViewModel includes both the editable fields and the `SelectListItem` collection needed to render the dropdown.

### Real-World Cases

- **Multi-step wizards:** Each step has a focused ViewModel (e.g., `ShippingViewModel`, `PaymentViewModel`).
- **Search pages:** A `SearchViewModel` containing search criteria, results, and pagination info.
- **Dashboard pages:** A `DashboardViewModel` aggregating data from multiple services into one shape.

### References

- Views in ASP.NET Core MVC (POCO ViewModels) – https://learn.microsoft.com/en-us/aspnet/core/mvc/views/overview
- ASP.NET Core MVC overview (ViewModel types) – https://learn.microsoft.com/en-us/aspnet/core/mvc/overview
- ViewModel best practices (POCO, no behaviour) – https://learn.microsoft.com/en-us/aspnet/core/mvc/views/overview

---

## 3. View Model Mapping

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | ViewModel mapping is the process of converting data from domain entities (or other sources) into ViewModel instances, and (for input ViewModels) converting bound ViewModel data back into domain entities. |
| **Technical Definition** | Mapping involves reading properties from a source object (e.g., an EF Core entity) and assigning them to corresponding properties on a ViewModel (and vice versa). Mapping can be performed **manually** (explicit property assignments), **via LINQ projections** (`Select` into the ViewModel), or **via mapping libraries** (AutoMapper, Mapperly). Manual mapping is often the most transparent and performant approach; libraries reduce boilerplate but introduce configuration and, in AutoMapper’s case, licensing considerations. |
| **Beginner-Friendly Explanation** | Mapping is **translating** between the database’s version of the data (the entity) and the page’s version (the ViewModel). For example, the entity might have `FirstName` and `LastName`, but the ViewModel has `FullName`. Mapping is the code that builds `FullName` from the two entity fields. |

### Purposes

- **To** transform domain entities into the shape the view expects.
- **To** combine data from multiple entities into a single ViewModel.
- **To** exclude sensitive or unnecessary properties from what the view receives.
- **To** convert bound input ViewModel data back into domain entities for persistence (for form submissions).
- **To** keep mapping logic isolated from controllers and views.

### Syntax Rules and Structure

#### General Syntax (Manual Mapping – Read)

```csharp
var vm = new ProductViewModel
{
    Id = product.Id,
    Name = product.Name,
    Price = product.Price,
    CategoryName = product.Category.Name
};
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| Object initializer | Explicitly assigns each property. |
| `product.Category.Name` | Navigates relationships to gather data. |

#### General Syntax (Manual Mapping – Write)

```csharp
var product = new Product
{
    Name = input.Name,
    Price = input.Price,
    CategoryId = input.CategoryId
};
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `input` | The bound input ViewModel. |
| `product` | The domain entity to persist. |

#### General Syntax (LINQ Projection)

```csharp
var vm = await _context.Products
    .Select(p => new ProductViewModel
    {
        Id = p.Id,
        Name = p.Name,
        Price = p.Price,
        CategoryName = p.Category.Name
    })
    .ToListAsync();
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `.Select(...)` | Projects directly into the ViewModel within the query. |
| Database efficiency | Only the required columns are queried. |

#### Syntax Rules

- **Manual mapping** is explicit and easy to debug; it is the recommended baseline for simple cases.
- **LINQ projection** is the most efficient read mapping because it translates to SQL and fetches only needed columns.
- **Library mapping** (AutoMapper/Mapperly) reduces boilerplate but requires configuration and, for AutoMapper, a license.
- **Mapping logic should not live in the ViewModel constructor**; use a separate mapper or the controller.

#### Constraints and Limitations

- **AutoMapper performance** – reflection-based mapping is generally slower than manual mapping; benchmarks show AutoMapper around 40% slower than manual mapping for simple cases.
- **AutoMapper licensing** – AutoMapper 15.0+ requires a commercial license for use.
- **LINQ projection limitations** – `ProjectTo` with AutoMapper supports only what the underlying LINQ provider allows; value resolvers and converters that require DI are not supported in `ProjectTo`.

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Manual Read Mapping

```csharp
public IActionResult Index()
{
    var products = _context.Products
        .Include(p => p.Category)
        .ToList();

    var vm = products.Select(p => new ProductListViewModel
    {
        Id = p.Id,
        Name = p.Name,
        Price = p.Price,
        CategoryName = p.Category.Name
    }).ToList();

    return View(vm);
}
```

**Step-by-step explanation:**

1. Entities are loaded with their categories included.
2. Each entity is manually mapped to a `ProductListViewModel`.
3. The list of ViewModels is passed to the view.

**Expected Output:** A product list page displaying name, price, and category for each product.

**Why this output?** Manual mapping explicitly shapes the data and avoids exposing navigation properties or internal fields.

---

#### Example 2: LINQ Projection (Most Efficient Read Mapping)

```csharp
public async Task<IActionResult> Index()
{
    var vm = await _context.Products
        .Select(p => new ProductListViewModel
        {
            Id = p.Id,
            Name = p.Name,
            Price = p.Price,
            CategoryName = p.Category.Name
        })
        .ToListAsync();

    return View(vm);
}
```

**Expected Output:** The same product list, but with a single efficient SQL query that selects only the required columns.

**Why this output?** The `Select` projection is translated into SQL, fetching only `Id`, `Name`, `Price`, and the category name—no unnecessary columns or entity tracking.

---

#### Example 3: Manual Write Mapping (Form Submission)

```csharp
[HttpPost]
public async Task<IActionResult> Create(ProductCreateViewModel input)
{
    if (!ModelState.IsValid)
    {
        input.Categories = await GetCategoriesAsync();
        return View(input);
    }

    var product = new Product
    {
        Name = input.Name,
        Price = input.Price,
        CategoryId = input.CategoryId
    };

    _context.Products.Add(product);
    await _context.SaveChangesAsync();
    return RedirectToAction(nameof(Index));
}
```

**Expected Output:** A new product is created from the bound ViewModel data.

**Why this output?** The input ViewModel is mapped to the domain entity, ensuring only the intended properties are set—preventing over-posting.

### Real-World Cases

- **Admin CRUD:** Manual or LINQ-projected mapping between entities and list/edit ViewModels.
- **APIs:** DTO mapping (often with AutoMapper or Mapperly) between entities and response models.
- **Complex dashboards:** Combining multiple entity types into a single ViewModel through manual mapping or a dedicated mapper service.

### References

- Views in ASP.NET Core MVC (strongly typed models) – https://learn.microsoft.com/en-us/aspnet/core/mvc/views/overview
- ASP.NET Core MVC overview (ViewModel creation by controllers) – https://learn.microsoft.com/en-us/aspnet/core/mvc/overview
- Manual mapping vs. library mapping performance (benchmark data) – https://www.nuget.org/packages/EggMapper

---

## 4. AutoMapper

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | AutoMapper is a .NET library that automates object-to-object mapping by convention, reducing the boilerplate code required to copy properties between types. |
| **Technical Definition** | AutoMapper uses **profiles** to define mappings between source and destination types (`CreateMap<Source, Destination>()`). At startup, the mappings are registered via `services.AddAutoMapper(...)`, and an `IMapper` instance is injected where needed. `IMapper.Map<TDestination>(source)` performs the mapping; `IMapper.ProjectTo<TDestination>(queryable)` performs LINQ projection. **Starting with version 15.0, AutoMapper requires a commercial license** for use; a license key must be configured. |
| **Beginner-Friendly Explanation** | AutoMapper is a **robot that copies data from one object to another**. Instead of writing `vm.Name = entity.Name; vm.Price = entity.Price;` for every property, you tell AutoMapper “map a `Product` to a `ProductViewModel`,” and it figures out the matching properties automatically. However, the robot now requires a paid license. |

### Purposes

- **To** eliminate repetitive property-by-property mapping code.
- **To** centralise mapping configuration in profiles for maintainability.
- **To** support convention-based mapping (matching property names and types).
- **To** provide LINQ projection support (`ProjectTo`) for efficient database queries.
- **To** reduce the risk of forgetting to map a new property (when validation is enabled).

### Syntax Rules and Structure

#### General Syntax (Installation and Registration)

```csharp
// Program.cs
builder.Services.AddAutoMapper(cfg =>
{
    cfg.LicenseKey = "<license key>";
}, typeof(Program));
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `AddAutoMapper` | Registers AutoMapper services. |
| `cfg.LicenseKey` | **Required from v15.0 onward**; without it, AutoMapper will not function. |
| `typeof(Program)` | Marker type for the assembly containing profiles. |

#### General Syntax (Profile Definition)

```csharp
public class ProductProfile : Profile
{
    public ProductProfile()
    {
        CreateMap<Product, ProductViewModel>();
        CreateMap<ProductCreateViewModel, Product>();
    }
}
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `Profile` | Base class for mapping configurations. |
| `CreateMap<Source, Destination>()` | Defines a mapping. |

#### General Syntax (Usage in a Controller)

```csharp
public class ProductsController : Controller
{
    private readonly IMapper _mapper;

    public ProductsController(IMapper mapper) => _mapper = mapper;

    public IActionResult Details(int id)
    {
        var product = _context.Products.Find(id);
        var vm = _mapper.Map<ProductViewModel>(product);
        return View(vm);
    }
}
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `IMapper` | Injected mapper instance. |
| `_mapper.Map<TDestination>(source)` | Performs the mapping. |

#### Syntax Rules

- **Profiles** group related mappings; register the assembly containing profiles.
- **`Map`** works in-memory; **`ProjectTo`** works with `IQueryable` for database projection.
- **`AssertConfigurationIsValid()`** should be called during development to catch unmapped properties.
- **License key** must be configured from v15.0 onward.

#### Constraints and Limitations

- **Commercial license required** – AutoMapper is dual-licensed; commercial use requires a paid license from v15.0 onward.
- **Reflection-based performance** – AutoMapper is slower than manual mapping and source generators.
- **`ProjectTo` limitations** – does not support value resolvers/converters that require dependency injection.
- **Configuration errors** – unmapped properties may silently go unset unless validation is enabled.

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Basic AutoMapper Setup and Usage

```csharp
// Profile
public class ProductProfile : Profile
{
    public ProductProfile()
    {
        CreateMap<Product, ProductViewModel>();
        CreateMap<ProductCreateViewModel, Product>();
    }
}
```
```csharp
// Program.cs
builder.Services.AddAutoMapper(cfg =>
{
    cfg.LicenseKey = builder.Configuration["AutoMapper:LicenseKey"];
}, typeof(Program));

// Controller
public class ProductsController : Controller
{
    private readonly IMapper _mapper;
    public ProductsController(IMapper mapper) => _mapper = mapper;

    public async Task<IActionResult> Details(int id)
    {
        var product = await _context.Products.FindAsync(id);
        if (product == null) return NotFound();
        var vm = _mapper.Map<ProductViewModel>(product);
        return View(vm);
    }

    [HttpPost]
    public async Task<IActionResult> Create(ProductCreateViewModel input)
    {
        if (!ModelState.IsValid) return View(input);
        var product = _mapper.Map<Product>(input);
        _context.Products.Add(product);
        await _context.SaveChangesAsync();
        return RedirectToAction(nameof(Index));
    }
}
```

**Step-by-step explanation:**

1. The profile defines two mappings: entity → ViewModel and input ViewModel → entity.
2. AutoMapper is registered with a license key.
3. The controller injects `IMapper` and uses `Map<T>()` for both directions.

**Expected Output:** Product details display correctly, and new products are created from form input.

**Why this output?** AutoMapper convention matches properties by name and type; `Name`, `Price`, and `CategoryId` map automatically.

---

#### Example 2: LINQ Projection with `ProjectTo`

```csharp
public async Task<IActionResult> Index()
{
    var vm = await _context.Products
        .ProjectTo<ProductListViewModel>(_mapper.ConfigurationProvider)
        .ToListAsync();

    return View(vm);
}
```

**Expected Output:** A product list fetched with an efficient SQL query that selects only the columns required by the ViewModel.

**Why this output?** `ProjectTo` translates the mapping into a LINQ `Select`, allowing the database to do the projection.

### Real-World Cases

- **Large CRUD applications:** AutoMapper reduces boilerplate across many entity/ViewModel pairs.
- **API layers:** DTO mapping for request/response models.
- **Teams with existing AutoMapper investments:** Reuse profiles across MVC and API layers.

### References

- AutoMapper documentation (Getting Started) – https://docs.automapper.io/
- AutoMapper Dependency Injection – https://docs.automapper.io/en/stable/Dependency-injection.html
- AutoMapper 15.0 Upgrade Guide (Licensing) – https://docs.automapper.io/en/stable/15.0-Upgrade-Guide.html
- AutoMapper licensing (GitHub) – https://github.com/LuckyPennySoftware/AutoMapper/blob/main/LICENSE.md
- AutoMapper now requires a license (official announcement) – https://www.jimmybogard.com/automapper-and-mediatr-going-commercial/
- Mapperly (source generator alternative) – https://mapperly.riok.app/
- Migrating from AutoMapper to Mapperly (ABP.IO) – https://abp.io/docs/10.4/release-info/migration-guides/automapper-to-mapperly

---

## 5. View Model Best Practices

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | ViewModel best practices are guidelines for designing, using, and maintaining ViewModels that are secure, maintainable, and performant. |
| **Technical Definition** | Best practices encompass keeping ViewModels as behaviour-light POCOs, avoiding ASP.NET Core dependencies, preventing over-posting, applying validation on ViewModels, choosing the right mapping strategy (manual vs. library), and understanding licensing and performance implications of mapping tools. |
| **Beginner-Friendly Explanation** | Best practices are the **“do’s and don’ts”** for ViewModels—they help you avoid security holes, messy code, and performance problems. For example: don’t bind directly to database entities, and don’t put database-access code inside a ViewModel. |

### Purposes

- **To** prevent over-posting and mass-assignment security vulnerabilities.
- **To** keep ViewModels simple, testable, and free of infrastructure dependencies.
- **To** ensure validation rules live on the ViewModel, not scattered across controllers or domain entities.
- **To** choose the most appropriate mapping strategy for the project’s scale and licensing constraints.
- **To** maintain a clear separation between domain logic and view presentation.

### Guidelines Checklist

| Area | Do | Don’t |
|------|----|-------|
| **POCO discipline** | Keep ViewModels as simple property bags with little or no behaviour. | Add business logic, database access, or service calls to ViewModels. |
| **ASP.NET Core isolation** | Keep ViewModels free of `HttpContext`, `Session`, `ViewBag`, `TempData`. | Couple ViewModels to the web framework. |
| **Over-posting prevention** | Expose only the properties the user should edit; use ViewModels instead of entities for form binding. | Bind directly to entities containing `IsAdmin`, `Role`, or audit fields. |
| **Validation location** | Apply validation attributes to ViewModel properties; rely on `ModelState.IsValid`. | Duplicate validation in controllers or views. |
| **Mapping strategy** | Use manual mapping or LINQ projection for simple cases; consider Mapperly for performance-sensitive projects. | Assume AutoMapper is free for commercial use—verify licensing. |
| **Mapping location** | Keep mapping out of ViewModel constructors; use a mapper service or controller. | Hide complex mapping inside constructors. |
| **ViewModel granularity** | Create focused ViewModels per view/feature; reuse only when shapes are truly identical. | Force one ViewModel to serve many unrelated views. |
| **Read-only views** | Still consider a ViewModel to avoid exposing internal fields, even for display-only pages. | Pass entities directly to views without evaluating security and coupling implications. |

### Annotated Code Example (Best-Practice ViewModel Setup)

```csharp
// Focused input ViewModel with validation
public class ProductCreateViewModel
{
    [Required, StringLength(100)]
    public string Name { get; set; } = null!;

    [Range(1, 10000)]
    public decimal Price { get; set; }

    [Required]
    public int CategoryId { get; set; }

    public IEnumerable<SelectListItem> Categories { get; set; }
        = Enumerable.Empty<SelectListItem>();
}

// Controller: mapping and validation
[HttpPost]
public async Task<IActionResult> Create(ProductCreateViewModel input)
{
    if (!ModelState.IsValid)
    {
        input.Categories = await GetCategoriesAsync();
        return View(input);
    }

    var product = new Product
    {
        Name = input.Name,
        Price = input.Price,
        CategoryId = input.CategoryId
    };

    _context.Products.Add(product);
    await _context.SaveChangesAsync();
    return RedirectToAction(nameof(Index));
}
```

**Why this follows best practices:**

- **Focused ViewModel** exposes only editable fields and lookup data.
- **Validation attributes** live on the ViewModel.
- **Manual mapping** to the entity prevents over-posting (no `Id`, `IsAdmin`, or audit fields exposed).
- **`ModelState.IsValid`** is checked before persistence.
- **No ASP.NET Core dependencies** in the ViewModel.

### Real-World Cases

- **Public-facing forms:** ViewModels prevent privilege escalation and data leakage.
- **High-performance APIs:** Manual mapping or Mapperly avoids reflection overhead.
- **Teams with AutoMapper:** License keys are managed centrally; `AssertConfigurationIsValid` runs in CI.

### References

- Views in ASP.NET Core MVC (POCO ViewModels) – https://learn.microsoft.com/en-us/aspnet/core/mvc/views/overview
- ASP.NET Core MVC overview (ViewModel types) – https://learn.microsoft.com/en-us/aspnet/core/mvc/overview
- Do you use ViewModel instead of ViewData? (SSW Rules) – https://www.ssw.com.au/rules/do-you-use-view-models-instead-of-viewdata
- Over-posting prevention with ViewModels – https://learn.microsoft.com/en-us/aspnet/core/data/ef-mvc/crud#overposting
- AutoMapper licensing – https://docs.automapper.io/en/stable/15.0-Upgrade-Guide.html

---

## Summary Table: Core Concepts at a Glance

| Concept | Primary Role | Key Tool / Syntax | When to Use |
|---------|--------------|-------------------|-------------|
| **View models** | Shape data for a specific view | POCO class, `@model` directive | Every view that needs shaped or secure data |
| **View model creation** | Define the class and its properties | Public properties + validation attributes | When designing a new page or form |
| **View model mapping** | Convert entities ↔ ViewModels | Manual mapping, LINQ `Select`, or a mapper library | Before passing data to a view, or after binding input |
| **AutoMapper** | Automated object-to-object mapping | `CreateMap`, `IMapper.Map`, `ProjectTo` | Projects willing to pay for a commercial license |
| **Best practices** | Security, maintainability, performance | POCO discipline, over-posting prevention, validation on ViewModels | Every production ViewModel |

---

## References (Full List)

- ASP.NET Core MVC overview (Model responsibilities, ViewModel types) – https://learn.microsoft.com/en-us/aspnet/core/mvc/overview
- Views in ASP.NET Core MVC (POCO ViewModels, strongly typed views) – https://learn.microsoft.com/en-us/aspnet/core/mvc/views/overview
- Do you use ViewModel instead of ViewData? (SSW Rules) – https://www.ssw.com.au/rules/do-you-use-view-models-instead-of-viewdata
- Over-posting prevention with ViewModels – https://learn.microsoft.com/en-us/aspnet/core/data/ef-mvc/crud#overposting
- AutoMapper documentation – https://docs.automapper.io/
- AutoMapper Dependency Injection – https://docs.automapper.io/en/stable/Dependency-injection.html
- AutoMapper 15.0 Upgrade Guide (Licensing) – https://docs.automapper.io/en/stable/15.0-Upgrade-Guide.html
- AutoMapper licensing (GitHub) – https://github.com/LuckyPennySoftware/AutoMapper/blob/main/LICENSE.md
- AutoMapper now requires a license (announcement) – https://www.jimmybogard.com/automapper-and-mediatr-going-commercial/
- Mapperly (source generator alternative) – https://mapperly.riok.app/
- Migrating from AutoMapper to Mapperly (ABP.IO) – https://abp.io/docs/10.4/release-info/migration-guides/automapper-to-mapperly
- Manual mapping vs. AutoMapper performance benchmark – https://www.nuget.org/packages/EggMapper