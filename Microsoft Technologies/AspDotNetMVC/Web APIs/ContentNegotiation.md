# ASP.NET Core Content Negotiation: A Comprehensive Cheat Sheet

> **Scope & Version Note:** This cheat sheet targets **ASP.NET Core** (.NET 6–10) and its **content negotiation** pipeline. Content negotiation is the HTTP mechanism by which clients and servers agree on the **media type** (JSON, XML, etc.) used in request and response bodies. ASP.NET Core implements it through **input formatters** (request body → .NET objects) and **output formatters** (.NET objects → response body). JSON is the default via `System.Text.Json`; XML requires explicit registration. **Minimal APIs** support content negotiation through `Results` and typed responses, though the formatter pipeline differs from MVC. Each requested topic is treated independently—no sections are merged.

---

## Topic Overview

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Content negotiation is the process by which a client and server agree on the **format** (media type) of the data exchanged in an HTTP request or response. |
| **Technical Definition** | Content negotiation in ASP.NET Core is driven by HTTP headers: the **`Accept`** header on requests (client tells the server which response media types it can handle) and the **`Content-Type`** header on both requests and responses (describing the actual media type of the body). ASP.NET Core uses **input formatters** (`IInputFormatter`) to deserialize request bodies and **output formatters** (`IOutputFormatter`) to serialize response bodies. The `ObjectResult` execution selects an output formatter based on the `Accept` header and the registered formatters. |
| **Beginner-Friendly Explanation** | Content negotiation is the **"what format do you want?"** conversation between client and server. A browser might say, "I prefer JSON, but XML is okay" (via the `Accept` header). The server picks the best format it can produce, serializes the response in that format, and labels the response with the chosen `Content-Type` (`application/json`). The client then knows how to parse the body. If the client can't accept any format the server can produce, the server returns `406 Not Acceptable`. |

### Key Characteristics

- **Header-driven** – The `Accept` and `Content-Type` headers are the primary signals.
- **Formatter-based** – ASP.NET Core uses input and output formatters to serialize/deserialize bodies.
- **JSON by default** – `System.Text.Json` is registered as the default output formatter in `AddControllers()`.
- **Extensible** – Custom formatters can be registered for CSV, MessagePack, Protobuf, or proprietary formats.
- **Negotiation-aware status codes** – `406 Not Acceptable` when no output formatter matches; `415 Unsupported Media Type` when no input formatter matches.
- **Attributes for control** – `[Produces]`, `[Consumes]`, and `[ApiController]` influence formatter selection and API documentation.

### Prerequisites

- .NET SDK (6.0+ recommended), ASP.NET Core Web API project.
- Working knowledge of **HTTP headers**, **REST**, and **API controllers**.
- Familiarity with **serialization** concepts (JSON, XML).
- Basic understanding of **dependency injection** and **MVC options configuration**.

### Related Programming Areas

- **Web API fundamentals** – The broader HTTP API architecture.
- **API controllers** – The classes that return `ObjectResult` bodies.
- **RESTful design** – Media types are part of the REST uniform interface.
- **OpenAPI** – Documents the media types an API supports.
- **Serialization** – `System.Text.Json`, `XmlSerializer`, and custom formatters.

### Core Concepts / Features

1. **Content negotiation** – The overall mechanism, formatters, and negotiation pipeline.
2. **`Accept` header** – The client's preferred response media types.
3. **`Content-Type` header** – The media type of the request or response body.
4. **JSON** – The default media type and `System.Text.Json` configuration.
5. **XML** – Optional XML support via `XmlSerializer` or `DataContractSerializer`.
6. **Content negotiation best practices** – Respecting `Accept`, `406`/`415` responses, and formatter configuration.

---

## 1. Content Negotiation (The Mechanism)

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Content negotiation is the HTTP mechanism that lets clients and servers agree on the media type of a request or response body. |
| **Technical Definition** | On the **request side**, the `Content-Type` header tells the server the format of the request body; the input formatter pipeline selects a matching `IInputFormatter`. On the **response side**, the `Accept` header tells the server which formats the client can consume; the output formatter pipeline selects a matching `IOutputFormatter`. If no output formatter matches, the framework returns `406 Not Acceptable` (when `ReturnHttpNotAcceptable = true`); if no input formatter matches, `415 Unsupported Media Type` is returned. `System.Text.Json` is the default output and input formatter for JSON. |
| **Beginner-Friendly Explanation** | Content negotiation is the **"translation service"** between the client's requested format and the server's available formats. The client says "I can read JSON and XML." The server checks what it can produce (JSON by default; XML if configured) and picks the best match. The response is labeled with the chosen format so the client knows how to read it. If nothing matches, the server politely declines with `406 Not Acceptable`. |

### Purposes

- **To** allow clients to request responses in a **preferred format** (JSON, XML, etc.) without changing the URL.
- **To** support **multiple client types** (JavaScript SPAs prefer JSON; legacy systems may require XML).
- **To** standardize **serialization and deserialization** through a pluggable formatter pipeline.
- **To** return **semantically correct status codes** (`406`, `415`) when formats cannot be negotiated.
- **To** enable **extensibility** through custom formatters (CSV, MessagePack, Protobuf).

### Syntax Rules and Structure

#### The Negotiation Pipeline

```text
Request:
  Content-Type: application/json
  Accept: application/xml, application/json;q=0.9
       │
       ▼
Input formatter pipeline (for request body)
       │
       ▼
Action executes → returns ObjectResult
       │
       ▼
Output formatter pipeline (for response body)
       │
       ▼
Response:
  Content-Type: application/xml
  Body: <Product>...</Product>
```

**Component breakdown:**

| Stage | Responsibility |
|-------|----------------|
| **Request `Content-Type`** | Determines which input formatter deserializes the body. |
| **Request `Accept`** | Determines which output formatter serializes the response. |
| **Action** | Executes and returns an `ObjectResult` (or `ActionResult<T>`). |
| **Output formatter selection** | Picks the best formatter based on `Accept` and registered formatters. |

#### General Syntax (Default Configuration — JSON Only)

```csharp
var builder = WebApplication.CreateBuilder(args);

builder.Services.AddControllers();   // Registers System.Text.Json formatters

var app = builder.Build();
app.MapControllers();
app.Run();
```

#### General Syntax (JSON + XML)

```csharp
builder.Services.AddControllers()
    .AddXmlSerializerFormatters();   // Adds XML output/input formatters
```

#### General Syntax (Enforcing `406 Not Acceptable`)

```csharp
builder.Services.Configure<ApiBehaviorOptions>(options =>
{
    options.ReturnHttpNotAcceptable = true;   // Default is false (falls back to JSON)
});
```

**Syntax rules:**

- **`AddControllers()` registers JSON formatters by default** — no additional configuration is needed for JSON-only APIs.
- **XML requires explicit registration** — `AddXmlSerializerFormatters()` (or `AddXmlDataContractSerializerFormatters()`).
- **`ReturnHttpNotAcceptable = true`** makes the API return `406` when the `Accept` header cannot be satisfied; otherwise, it falls back to the first registered formatter (usually JSON).
- **Input formatters are chosen by `Content-Type`** — a request with `Content-Type: application/xml` will use the XML input formatter if registered; otherwise, `415 Unsupported Media Type` is returned.

**Constraints and limitations:**

- **Fallback to JSON** — by default, if no formatter matches `Accept`, the first output formatter (JSON) is used instead of `406`. Set `ReturnHttpNotAcceptable = true` to enforce strict negotiation.
- **XML serialization differs** — `XmlSerializer` requires public parameterless constructors and writable properties; `System.Text.Json` supports records and init-only properties.
- **Minimal APIs** — content negotiation is less centralized; `Results.Json()` and `Results.Text()` often specify the format explicitly.

### Annotated Complete Step-by-Step Code Example

```csharp
// Program.cs
var builder = WebApplication.CreateBuilder(args);

builder.Services.AddControllers()
    .AddXmlSerializerFormatters();   // Enable JSON + XML

builder.Services.Configure<ApiBehaviorOptions>(options =>
{
    options.ReturnHttpNotAcceptable = true;   // Strict negotiation
});

var app = builder.Build();
app.MapControllers();
app.Run();

// ProductsController.cs
[ApiController]
[Route("api/[controller]")]
public class ProductsController : ControllerBase
{
    private static readonly Product[] _products =
    {
        new(1, "Laptop", 999.99m),
        new(2, "Mouse", 25.00m)
    };

    [HttpGet]
    public ActionResult<IEnumerable<Product>> GetAll() => Ok(_products);
}

public record Product(int Id, string Name, decimal Price);
```

**Step-by-step explanation:**

1. `AddControllers()` registers the default JSON formatters.
2. `AddXmlSerializerFormatters()` adds XML output and input formatters.
3. `ReturnHttpNotAcceptable = true` makes the API return `406` when the `Accept` header cannot be satisfied.
4. The `GetAll` action returns an `ObjectResult` wrapping the products.
5. The output formatter pipeline selects the formatter based on `Accept`.

**Expected Output:**

| Request `Accept` | Response `Content-Type` | Body |
|------------------|-------------------------|------|
| `application/json` | `application/json` | `[{"id":1,"name":"Laptop","price":999.99}, ...]` |
| `application/xml` | `application/xml` | `<ArrayOfProduct><Product>...</Product></ArrayOfProduct>` |
| `text/csv` | (none — `406 Not Acceptable`) | Empty |

**Why this output?** The output formatter pipeline matches `Accept` against registered formatters; when no formatter matches and `ReturnHttpNotAcceptable = true`, `406` is returned.

### Real-World Cases

- **Public APIs:** JSON for SPAs; XML for legacy enterprise integrations.
- **Content delivery:** Media types for images, PDFs, or custom formats.
- **Mobile backends:** JSON is universal; XML rarely used but supported for specific clients.

### References

- Format response data in ASP.NET Core Web API – https://learn.microsoft.com/en-us/aspnet/core/web-api/advanced/formatting
- Content negotiation (MDN) – https://developer.mozilla.org/en-US/docs/Web/HTTP/Content_negotiation

---

## 2. `Accept` Header

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | The `Accept` request header tells the server which media types the client is willing to receive in the response. |
| **Technical Definition** | `Accept` is a comma-separated list of media ranges, each optionally followed by **quality values** (`q`). Example: `Accept: application/json, application/xml;q=0.9, text/plain;q=0.5`. Quality values range from `0` to `1` and indicate preference. The server's output formatter selection algorithm respects `q` values and picks the highest-quality acceptable media type. Wildcards (`*/*`, `application/*`) indicate broad acceptance. |
| **Beginner-Friendly Explanation** | The `Accept` header is the client's **"I can read these formats"** list. It says "I prefer JSON, but I'll take XML if that's all you have." The server reads this list, checks what it can produce, and picks the best match. If the client says `*/*`, it means "anything you have is fine"—the server then uses its default (usually JSON). |

### Purposes

- **To** let clients **declare their preferred response formats** without changing the URL.
- **To** support **content negotiation** by guiding output formatter selection.
- **To** express **preference weights** via quality values (`q`).
- **To** enable **wildcard acceptance** (`*/*`) for clients that can handle anything.
- **To** allow the server to return **`406 Not Acceptable`** when the client's formats cannot be satisfied.

### Syntax Rules and Structure

#### General Syntax

```http
Accept: <media-type>[;q=<quality>][, <media-type>[;q=<quality>]]...
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `media-type` | A media type (e.g., `application/json`, `application/xml`, `text/*`). |
| `q` | Quality value from `0` to `1` (default `1`). Higher values indicate stronger preference. |
| `*/*` | Wildcard — accepts any media type. |
| `application/*` | Accepts any `application` media type. |

#### Examples

```http
Accept: application/json
Accept: application/json, application/xml;q=0.9
Accept: application/xml;q=0.8, application/json;q=0.5
Accept: */*
Accept: text/html, application/xhtml+xml, application/xml;q=0.9, */*;q=0.8
```

**Syntax rules:**

- **Multiple media types** are separated by commas.
- **Quality values** (`q=0.9`) are optional; default is `1.0`.
- **Higher `q` values indicate stronger preference** — the server should prefer `application/json;q=1.0` over `application/xml;q=0.9`.
- **Wildcards** (`*/*`, `application/*`) broaden acceptance.
- **The `Accept` header is optional** — if absent, the server may return any format (usually its default).

**Constraints and limitations:**

- **Not all servers respect `q` values strictly** — ASP.NET Core's formatter selection considers them, but fallback behaviour (default JSON) can override preference when `ReturnHttpNotAcceptable = false`.
- **`Accept` applies only to responses** — request body format is controlled by `Content-Type`.
- **Some clients send overly broad or inconsistent `Accept` headers** — servers should handle missing, malformed, or wildcard-only headers gracefully.

### Annotated Complete Step-by-Step Code Example

```csharp
// Client requests XML with higher preference than JSON
// Accept: application/xml;q=1.0, application/json;q=0.5

// Controller — same action, different formatters selected
[ApiController]
[Route("api/[controller]")]
public class ProductsController : ControllerBase
{
    [HttpGet]
    public ActionResult<IEnumerable<Product>> GetAll() => Ok(_products);
}
```

**Expected Output:** With `Accept: application/xml;q=1.0, application/json;q=0.5`, the response is XML (`Content-Type: application/xml`) because XML has a higher `q` value.

**Why this output?** The output formatter pipeline evaluates the `Accept` header's quality values and selects the highest-preference formatter it can produce.

### Real-World Cases

- **SPAs:** `Accept: application/json` — straightforward.
- **Legacy enterprise clients:** `Accept: application/xml` — requires XML formatters.
- **Browsers:** Complex `Accept` headers with HTML, XHTML, XML, and wildcards — APIs should still return JSON unless HTML is configured.

### References

- `Accept` header (MDN) – https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Accept
- Quality values (`q`) – https://developer.mozilla.org/en-US/docs/Glossary/Quality_values

---

## 3. `Content-Type` Header

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | The `Content-Type` header indicates the media type of the body in an HTTP request or response. |
| **Technical Definition** | `Content-Type` has the form `<media-type>[; charset=<encoding>][; boundary=<boundary>]`. On **requests**, it tells the server how to deserialize the body (input formatter selection). On **responses**, it tells the client how to parse the body (output formatter selection). Common values: `application/json`, `application/xml`, `text/plain`, `multipart/form-data`, `application/x-www-form-urlencoded`. |
| **Beginner-Friendly Explanation** | `Content-Type` is the **"label on the package"** that says what's inside. On a request, it says "the body is JSON"—so the server knows to use the JSON parser. On a response, it says "the body is JSON"—so the client knows to use the JSON parser. Without this label, neither side would know how to read the body. |

### Purposes

- **To** **label the format** of the request or response body.
- **To** drive **input formatter selection** on the server (e.g., `application/json` → JSON input formatter).
- **To** inform the **client how to parse** the response body.
- **To** support **content negotiation** on the response side (the server sets `Content-Type` to the chosen format).
- **To** enable **file uploads** (`multipart/form-data`) and form posts (`application/x-www-form-urlencoded`).

### Syntax Rules and Structure

#### General Syntax

```http
Content-Type: <media-type>[; charset=<encoding>]
```

#### Examples

```http
Content-Type: application/json
Content-Type: application/json; charset=utf-8
Content-Type: application/xml
Content-Type: text/plain; charset=utf-8
Content-Type: multipart/form-data; boundary=----WebKitFormBoundary
Content-Type: application/x-www-form-urlencoded
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `media-type` | The format of the body (`application/json`, `application/xml`, etc.). |
| `charset` | Character encoding (default `utf-8` for JSON). |
| `boundary` | Used with `multipart/form-data` to separate parts. |

**Syntax rules:**

- **`Content-Type` is required for requests with a body** — otherwise, the server may reject the request or guess.
- **`Content-Type` is set automatically by the server on responses** — based on the selected output formatter.
- **`charset` is optional** — `application/json` defaults to UTF-8 per RFC 8259.
- **The input formatter is selected by `Content-Type`** — a request with `application/xml` requires an XML input formatter.

**Constraints and limitations:**

- **`415 Unsupported Media Type`** is returned when the request `Content-Type` does not match any registered input formatter.
- **`Content-Type` on GET/DELETE requests with no body** is typically absent and ignored.
- **`charset` mismatches** can cause parsing errors; `System.Text.Json` expects UTF-8 by default.

### Annotated Complete Step-by-Step Code Example

```csharp
[ApiController]
[Route("api/[controller]")]
public class ProductsController : ControllerBase
{
    [HttpPost]
    [Consumes("application/json")]   // Only accept JSON requests
    public ActionResult<Product> Create([FromBody] Product product)
    {
        return CreatedAtAction(nameof(GetById),
            new { id = product.Id }, product);
    }
}
```

**Expected Output:**

| Request `Content-Type` | Result |
|------------------------|--------|
| `application/json` | `201 Created` — body deserialized by JSON input formatter |
| `application/xml` | `415 Unsupported Media Type` — no XML input formatter registered (or `[Consumes]` blocks it) |

**Why this output?** The `[Consumes]` attribute restricts the endpoint to JSON requests; the input formatter pipeline uses the `Content-Type` header to select the formatter.

### Real-World Cases

- **JSON APIs:** `Content-Type: application/json` on POST/PUT/PATCH requests.
- **File uploads:** `Content-Type: multipart/form-data` with `IFormFile` binding.
- **Form posts:** `application/x-www-form-urlencoded` for HTML forms.
- **XML integrations:** `Content-Type: application/xml` for SOAP-like or legacy clients.

### References

- `Content-Type` header (MDN) – https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Content-Type
- Media types (IANA) – https://www.iana.org/assignments/media-types/media-types.xhtml

---

## 4. JSON (The Default Media Type)

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | JSON (JavaScript Object Notation) is a lightweight, text-based data interchange format that is the default media type for ASP.NET Core Web APIs. |
| **Technical Definition** | ASP.NET Core uses **`System.Text.Json`** (since .NET Core 3.0) as the default JSON serializer. `AddControllers()` registers `SystemTextJsonInputFormatter` and `SystemTextJsonOutputFormatter`. Configuration is done via `JsonOptions` (`JsonSerializerOptions`): property naming policy, reference handling, converters, `DefaultIgnoreCondition`, `NumberHandling`, etc. The legacy `Newtonsoft.Json` can be added via the `Microsoft.AspNetCore.Mvc.NewtonsoftJson` package for backward compatibility with older behaviours. |
| **Beginner-Friendly Explanation** | JSON is the **default language of Web APIs**. It looks like JavaScript objects—`{"id":1,"name":"Laptop"}`—and is easy for browsers, mobile apps, and servers to read. ASP.NET Core uses `System.Text.Json` to convert C# objects to JSON and back. You can customize how it behaves (property names, null handling, cycles) through `JsonOptions`. |

### Purposes

- **To** provide a **lightweight, human-readable** format for API payloads.
- **To** serve as the **default response and request format** for ASP.NET Core APIs.
- **To** support **modern clients** (JavaScript, mobile apps) natively.
- **To** enable **customization** of serialization behaviour (naming, null handling, converters).
- **To** integrate with **OpenAPI** for schema documentation.

### Syntax Rules and Structure

#### General Syntax (Default JSON Configuration)

```csharp
builder.Services.AddControllers();   // JSON via System.Text.Json by default
```

#### General Syntax (Custom JSON Options)

```csharp
builder.Services.AddControllers()
    .AddJsonOptions(options =>
    {
        options.JsonSerializerOptions.PropertyNamingPolicy = JsonNamingPolicy.CamelCase;
        options.JsonSerializerOptions.DefaultIgnoreCondition = JsonIgnoreCondition.WhenWritingNull;
        options.JsonSerializerOptions.ReferenceHandler = ReferenceHandler.IgnoreCycles;
        options.JsonSerializerOptions.Converters.Add(new JsonStringEnumConverter());
    });
```

**Component breakdown:**

| Option | Effect |
|--------|--------|
| `PropertyNamingPolicy` | Controls property name casing (`camelCase` by default). |
| `DefaultIgnoreCondition` | Omits `null` properties when set to `WhenWritingNull`. |
| `ReferenceHandler` | Handles object cycles (`IgnoreCycles` prevents serialization loops). |
| `Converters` | Adds custom converters (e.g., string enums). |

#### General Syntax (Per-Action JSON Format)

```csharp
[HttpGet]
public IActionResult GetAll() =>
    new JsonResult(_products, new JsonSerializerOptions
    {
        PropertyNamingPolicy = JsonNamingPolicy.SnakeCaseLower
    });
```

**Syntax rules:**

- **JSON is the default** — no extra configuration is required.
- **`System.Text.Json` is case-insensitive on input** by default (via `PropertyNameCaseInsensitive`).
- **`JsonResult` allows per-action custom options** when needed.
- **Records and init-only properties are supported** by `System.Text.Json` (unlike `XmlSerializer`).

**Constraints and limitations:**

- **`System.Text.Json` is stricter than `Newtonsoft.Json`** — some legacy behaviours (e.g., `PreserveReferencesHandling`, custom converters) require `Newtonsoft.Json`.
- **`Newtonsoft.Json` is not registered by default** — add `AddNewtonsoftJson()` if you need it.
- **Cycles cause exceptions** unless `ReferenceHandler` is configured.

### Annotated Complete Step-by-Step Code Example

```csharp
public record Product(int Id, string Name, decimal Price, Category? Category);

public record Category(int Id, string Name);

builder.Services.AddControllers()
    .AddJsonOptions(options =>
    {
        options.JsonSerializerOptions.PropertyNamingPolicy = JsonNamingPolicy.CamelCase;
        options.JsonSerializerOptions.DefaultIgnoreCondition = JsonIgnoreCondition.WhenWritingNull;
        options.JsonSerializerOptions.Converters.Add(new JsonStringEnumConverter());
    });

[ApiController]
[Route("api/[controller]")]
public class ProductsController : ControllerBase
{
    [HttpGet]
    public ActionResult<IEnumerable<Product>> GetAll() => Ok(new[]
    {
        new Product(1, "Laptop", 999.99m, new Category(1, "Electronics")),
        new Product(2, "Mouse", 25.00m, null)
    });
}
```

**Expected Output:**

```json
[
  { "id": 1, "name": "Laptop", "price": 999.99, "category": { "id": 1, "name": "Electronics" } },
  { "id": 2, "name": "Mouse", "price": 25.00 }
]
```

**Why this output?** `PropertyNamingPolicy = CamelCase` converts `Id` to `id`; `DefaultIgnoreCondition = WhenWritingNull` omits the `Category` property for the mouse (which is `null`).

### Real-World Cases

- **SPA backends:** JSON is the native format for JavaScript.
- **Mobile apps:** JSON is the standard for iOS/Android networking.
- **Microservices:** JSON over HTTP is the default inter-service contract.
- **Public APIs:** JSON responses with camelCase and null omission are the industry norm.

### References

- JSON serialization in .NET – https://learn.microsoft.com/en-us/dotnet/standard/serialization/system-text-json/overview
- Format response data in ASP.NET Core Web API (JSON) – https://learn.microsoft.com/en-us/aspnet/core/web-api/advanced/formatting#system-text-json-formatters

---

## 5. XML (The Optional Media Type)

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | XML (eXtensible Markup Language) is a text-based markup format for structured data, supported in ASP.NET Core Web APIs when explicitly enabled. |
| **Technical Definition** | ASP.NET Core supports XML through **`XmlSerializerInputFormatter`/`XmlSerializerOutputFormatter`** (via `AddXmlSerializerFormatters()`) or **`XmlDataContractSerializerInputFormatter`/`XmlDataContractSerializerOutputFormatter`** (via `AddXmlDataContractSerializerFormatters()`). XML serialization requires public parameterless constructors and public read/write properties for `XmlSerializer`; `DataContractSerializer` uses `[DataContract]`/`[DataMember]` attributes. XML is **not** registered by default—it must be added explicitly. |
| **Beginner-Friendly Explanation** | XML is the **older, more verbose cousin of JSON**—it uses tags like `<Product><Name>Laptop</Name></Product>` instead of `{"Name":"Laptop"}`. ASP.NET Core does **not** support XML out of the box; you must add the XML formatters explicitly. Once added, clients can request XML with `Accept: application/xml` and send XML with `Content-Type: application/xml`. |

### Purposes

- **To** support **legacy clients** and integrations that require XML.
- **To** enable **content negotiation** for clients that prefer XML.
- **To** provide **schema-validated** payloads (XSD) when needed.
- **To** interoperate with **SOAP-like** or enterprise systems.
- **To** support **XSLT transformation** workflows.

### Syntax Rules and Structure

#### General Syntax (Register XML Formatters)

```csharp
builder.Services.AddControllers()
    .AddXmlSerializerFormatters();   // Or .AddXmlDataContractSerializerFormatters()
```

#### General Syntax (XML-Serializable Model)

```csharp
[XmlRoot("Product")]
public class Product
{
    [XmlElement("Id")]
    public int Id { get; set; }

    [XmlElement("Name")]
    public string Name { get; set; } = "";

    [XmlElement("Price")]
    public decimal Price { get; set; }

    public Product() { }   // Required by XmlSerializer
}
```

#### General Syntax (Request with XML)

```http
POST /api/products HTTP/1.1
Content-Type: application/xml
Accept: application/xml

<Product><Name>Laptop</Name><Price>999.99</Price></Product>
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `AddXmlSerializerFormatters()` | Registers `XmlSerializer`-based input/output formatters. |
| `[XmlRoot]`, `[XmlElement]` | Control the XML element names. |
| Parameterless constructor | Required by `XmlSerializer`. |
| `Content-Type: application/xml` | Selects the XML input formatter. |

**Syntax rules:**

- **XML is opt-in** — without `AddXmlSerializerFormatters()` or `AddXmlDataContractSerializerFormatters()`, XML requests receive `415 Unsupported Media Type` and XML `Accept` headers are ignored (JSON is returned).
- **`XmlSerializer` requires a parameterless constructor** and public settable properties.
- **Records may not serialize correctly with `XmlSerializer`** due to lack of parameterless constructors; use classes or `DataContractSerializer`.
- **`[Produces("application/xml")]`** can restrict an action to XML output.

**Constraints and limitations:**

- **`XmlSerializer` does not support some modern C# features** (records with positional parameters, init-only properties) without workarounds.
- **XML payloads are larger and slower to parse** than JSON.
- **`DataContractSerializer` requires `[DataContract]`/`[DataMember]` attributes** for explicit control.
- **`System.Text.Json` cannot serialize to XML** — the XML formatters are separate.

### Annotated Complete Step-by-Step Code Example

```csharp
builder.Services.AddControllers()
    .AddXmlSerializerFormatters();

[ApiController]
[Route("api/[controller]")]
public class ProductsController : ControllerBase
{
    [HttpGet]
    public ActionResult<IEnumerable<Product>> GetAll() => Ok(new[]
    {
        new Product { Id = 1, Name = "Laptop", Price = 999.99m }
    });
}

public class Product
{
    public int Id { get; set; }
    public string Name { get; set; } = "";
    public decimal Price { get; set; }
}
```

**Expected Output (for `Accept: application/xml`):**

```xml
<ArrayOfProduct xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:xsd="http://www.w3.org/2001/XMLSchema">
  <Product>
    <Id>1</Id>
    <Name>Laptop</Name>
    <Price>999.99</Price>
  </Product>
</ArrayOfProduct>
```

**Why this output?** The XML output formatter serializes the list using `XmlSerializer`, wrapping it in an `ArrayOfProduct` root and each item in a `Product` element.

### Real-World Cases

- **Legacy enterprise systems:** SOAP-adjacent integrations and XML-based EDI.
- **Financial services:** Some regulatory or partner APIs still require XML.
- **Content management:** XML-based content syndication or XSLT workflows.

### References

- Format response data in ASP.NET Core Web API (XML formatters) – https://learn.microsoft.com/en-us/aspnet/core/web-api/advanced/formatting#xml-formatters
- `XmlSerializer` in .NET – https://learn.microsoft.com/en-us/dotnet/standard/serialization/introducing-xml-serialization
- `DataContractSerializer` – https://learn.microsoft.com/en-us/dotnet/standard/serialization/data-contracts

---

## 6. Content Negotiation Best Practices

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Content negotiation best practices are guidelines for configuring and consuming the formatter pipeline so that clients and servers agree on formats reliably, securely, and maintainably. |
| **Technical Definition** | Best practices span **enforcing strict negotiation** (`ReturnHttpNotAcceptable = true`), **restricting formats per endpoint** (`[Produces]`, `[Consumes]`), **configuring JSON options** (camelCase, null handling, cycles), **avoiding `Newtonsoft.Json` unless necessary**, **returning correct status codes** (`406`, `415`), **documenting supported media types** in OpenAPI, and **handling XML safely** (avoiding XXE, limiting payload size). |
| **Beginner-Friendly Explanation** | Best practices are the **"do's and don'ts"** for content negotiation—they help you avoid accidentally returning the wrong format, allowing dangerous XML payloads, or confusing clients with inconsistent media types. |

### Purposes

- **To** ensure clients receive the **format they asked for**—or a clear `406 Not Acceptable` when impossible.
- **To** **restrict endpoints** to the formats they genuinely support.
- **To** **document supported media types** so clients know what to send and expect.
- **To** **secure XML parsing** against XXE and other XML-specific attacks.
- **To** **standardize JSON output** (camelCase, null omission) across the API.
- **To** **avoid unnecessary dependencies** (e.g., `Newtonsoft.Json` when `System.Text.Json` suffices).

### Guidelines Checklist

| Area | Do | Don't |
|------|----|-------|
| **Strict negotiation** | Set `ReturnHttpNotAcceptable = true` in production APIs. | Silently fall back to JSON when the client asked for XML—it confuses clients. |
| **Endpoint restrictions** | Use `[Produces("application/json")]` and `[Consumes("application/json")]` on JSON-only endpoints. | Allow XML input on endpoints that cannot safely handle it. |
| **JSON configuration** | Use camelCase, `WhenWritingNull`, and `IgnoreCycles` globally. | Leave `System.Text.Json` defaults if the API contract requires different naming. |
| **JSON vs. Newtonsoft** | Prefer `System.Text.Json`; add `Newtonsoft.Json` only for legacy behaviours. | Add `Newtonsoft.Json` "just in case" without a concrete need. |
| **XML** | Register XML only when needed; use `DataContractSerializer` for fine control; protect against XXE. | Enable XML with default settings without considering payload size and security. |
| **Status codes** | Return `406` for unsatisfiable `Accept`; `415` for unsupported `Content-Type`. | Return `200` with an empty body when negotiation fails. |
| **OpenAPI** | Document the supported `produces` and `consumes` media types per endpoint. | Leave media types undocumented—clients have to guess. |
| **Custom formatters** | Register custom formatters (CSV, MessagePack) explicitly and test them thoroughly. | Allow arbitrary formatters to process untrusted input without size limits. |
| **Request size** | Configure `RequestSizeLimit` and formatter-specific limits. | Accept unbounded XML/JSON payloads. |
| **Testing** | Test each supported media type and the `406`/`415` paths. | Test only the JSON happy path. |

### Annotated Code Example (Best-Practice Content Negotiation Setup)

```csharp
var builder = WebApplication.CreateBuilder(args);

builder.Services.AddControllers(options =>
{
    // Optional: restrict default output to JSON
    options.Filters.Add(new ProducesAttribute("application/json"));
})
.AddJsonOptions(options =>
{
    options.JsonSerializerOptions.PropertyNamingPolicy = JsonNamingPolicy.CamelCase;
    options.JsonSerializerOptions.DefaultIgnoreCondition = JsonIgnoreCondition.WhenWritingNull;
    options.JsonSerializerOptions.ReferenceHandler = ReferenceHandler.IgnoreCycles;
})
.AddXmlSerializerFormatters();   // Only if XML is genuinely needed

builder.Services.Configure<ApiBehaviorOptions>(options =>
{
    options.ReturnHttpNotAcceptable = true;   // 406 when Accept cannot be satisfied
});

builder.Services.Configure<MvcOptions>(options =>
{
    // Limit XML input size (defence in depth)
    var xml = options.InputFormatters
        .OfType<XmlSerializerInputFormatter>()
        .FirstOrDefault();
    if (xml is not null)
    {
        xml.SupportedEncodings.Clear();
        xml.SupportedEncodings.Add(System.Text.Encoding.UTF8);
    }
});

var app = builder.Build();
app.MapControllers();
app.Run();
```

**Why this follows best practices:**

- **`ReturnHttpNotAcceptable = true`** enforces strict negotiation—clients get `406` rather than an unexpected JSON response.
- **JSON options** are standardized (camelCase, null omission, cycle handling) across the API.
- **XML is opt-in** — only registered when genuinely needed.
- **XML input is restricted to UTF-8** — reduces encoding-based attack surface.
- **A global `ProducesAttribute`** sets the default media type, making OpenAPI documentation accurate.
- **Custom formatter limits** prevent oversized payloads from consuming resources.

### Real-World Cases

- **Public JSON APIs:** JSON-only with strict `406`/`415` handling and documented media types.
- **APIs with legacy XML clients:** XML formatters enabled on specific endpoints via `[Produces]`/`[Consumes]`.
- **High-security APIs:** XML input restricted, payload sizes limited, and XML external entities (XXE) disabled by default (which `XmlSerializer` already does—`DtdProcessing.Prohibit`).
- **Microservices:** JSON-only, with `System.Text.Json` options standardized via a shared configuration package.

### References

- Format response data in ASP.NET Core Web API – https://learn.microsoft.com/en-us/aspnet/core/web-api/advanced/formatting
- Content negotiation (MDN) – https://developer.mozilla.org/en-US/docs/Web/HTTP/Content_negotiation
- `ApiBehaviorOptions.ReturnHttpNotAcceptable` – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.apibehavioroptions.returnhttpnotacceptable
- `ProducesAttribute` – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.producesattribute
- `ConsumesAttribute` – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.consumesattribute
- XXE prevention in .NET – https://learn.microsoft.com/en-us/dotnet/standard/data/xml/xml-security-considerations

---

## Summary Table: Content Negotiation at a Glance

| Concept | Primary Role | Key Syntax / Tool | When to Use |
|---------|--------------|-------------------|-------------|
| **Content negotiation** | Agree on body media type | Input/output formatters, `Accept`, `Content-Type` | Every API response/request with a body |
| **`Accept` header** | Client's preferred response formats | `Accept: application/json, application/xml;q=0.9` | Every request that reads a response body |
| **`Content-Type` header** | Media type of the body | `Content-Type: application/json` | Requests with a body; server sets it on responses |
| **JSON** | Default, lightweight format | `System.Text.Json`, `AddJsonOptions` | Default for all APIs |
| **XML** | Optional legacy format | `AddXmlSerializerFormatters()`, `[XmlRoot]` | Legacy clients and integrations |
| **Best practices** | Strict, documented, secure negotiation | `ReturnHttpNotAcceptable`, `[Produces]`, `[Consumes]`, JSON options | Every production API |

---

## References (Full List)

- Format response data in ASP.NET Core Web API – https://learn.microsoft.com/en-us/aspnet/core/web-api/advanced/formatting
- Content negotiation (MDN) – https://developer.mozilla.org/en-US/docs/Web/HTTP/Content_negotiation
- `Accept` header (MDN) – https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Accept
- `Content-Type` header (MDN) – https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Content-Type
- Quality values (`q`) – https://developer.mozilla.org/en-US/docs/Glossary/Quality_values
- Media types (IANA) – https://www.iana.org/assignments/media-types/media-types.xhtml
- `System.Text.Json` overview – https://learn.microsoft.com/en-us/dotnet/standard/serialization/system-text-json/overview
- JSON serialization in ASP.NET Core – https://learn.microsoft.com/en-us/aspnet/core/web-api/advanced/formatting#system-text-json-formatters
- XML formatters in ASP.NET Core – https://learn.microsoft.com/en-us/aspnet/core/web-api/advanced/formatting#xml-formatters
- `XmlSerializer` in .NET – https://learn.microsoft.com/en-us/dotnet/standard/serialization/introducing-xml-serialization
- `DataContractSerializer` – https://learn.microsoft.com/en-us/dotnet/standard/serialization/data-contracts
- `ApiBehaviorOptions.ReturnHttpNotAcceptable` – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.apibehavioroptions.returnhttpnotacceptable
- `ProducesAttribute` – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.producesattribute
- `ConsumesAttribute` – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.consumesattribute
- XML security considerations (XXE) – https://learn.microsoft.com/en-us/dotnet/standard/data/xml/xml-security-considerations