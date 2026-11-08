# ASP.NET Core API Versioning: A Comprehensive Cheat Sheet

> **Scope & Version Note:** This cheat sheet targets **ASP.NET Core** (.NET 6–10) using the **`Asp.Versioning.Mvc`** package (the modern successor to the deprecated `Microsoft.AspNetCore.Mvc.Versioning` package). API versioning allows you to evolve your API without breaking existing clients by maintaining multiple versions simultaneously. The library supports **URI path**, **header**, **query string**, and **media type** versioning strategies, with seamless integration into MVC controllers and Minimal APIs. **`Asp.Versioning` v10** is the first release to officially support .NET 10 and the new built-in OpenAPI library. Each requested topic is treated independently—no sections are merged.

---

## Topic Overview

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | API versioning is the practice of managing changes to an API over time by exposing multiple versions simultaneously, allowing clients to migrate at their own pace. |
| **Technical Definition** | API versioning in ASP.NET Core is implemented via the `Asp.Versioning` library, which adds version metadata to controllers/actions (`[ApiVersion]`, `[MapToApiVersion]`) and version readers (`QueryStringApiVersionReader`, `HeaderApiVersionReader`, `UrlSegmentApiVersionReader`). The version is used for **API selection**, not routing—the defined API version, even for an action, never directly influences routing. Requests are matched to the appropriate controller/action based on the version declared by the client (or the default version if none is specified). |
| **Beginner‑Friendly Explanation** | API versioning is the **"parallel roads"** approach to evolving your API. Instead of forcing all clients onto a single, ever‑changing road, you build new roads (v2, v3) while keeping the old ones open. Clients can keep using v1 until they are ready to move to v2. The `Asp.Versioning` library makes this easy: you decorate controllers with `[ApiVersion("1.0")]` or `[ApiVersion("2.0")]`, and the library routes each request to the right version based on the URL, a header, or a query string. |

### Key Characteristics

- **Multiple simultaneous versions** – Old and new versions coexist; clients migrate on their own schedule.
- **Breaking‑change isolation** – Breaking changes (removing fields, changing data types) go into a new version; non‑breaking additions stay in the current version.
- **Client‑controlled version selection** – The client chooses the version via URL, header, query string, or media type.
- **Default version support** – Requests without an explicit version can be routed to a configurable default version.
- **Deprecation signalling** – Versions can be marked `Deprecated = true`, and response headers (`api-supported-versions`, `api-deprecated-versions`) inform clients.
- **OpenAPI integration** – The library generates separate OpenAPI documents per version.

### Prerequisites

- .NET SDK (6.0+ recommended), ASP.NET Core MVC or Minimal API project.
- `Asp.Versioning.Mvc` NuGet package (and `Asp.Versioning.Mvc.ApiExplorer` for OpenAPI integration).
- Working knowledge of **controllers**, **routing**, and **middleware**.
- Basic understanding of **REST principles** and **HTTP headers**.

### Related Programming Areas

- **Web API fundamentals** – API controllers, routing, and HTTP semantics.
- **RESTful design** – Backward compatibility and versioning as a REST maturity concern.
- **OpenAPI** – Version‑aware API documentation.
- **Deprecation and sunset** – Communicating version lifecycle to clients.
- **Minimal APIs** – Versioning endpoints via `NewVersionedApi()` and `MapToApiVersion()`.

### Core Concepts / Features

1. **API versioning** – The overall concept, library setup, and configuration.
2. **URI versioning** – Version in the URL path (`/api/v1/products`).
3. **Header versioning** – Version in a custom HTTP header (`X-API-Version: 2.0`).
4. **Query versioning** – Version in a query parameter (`?api-version=2.0`).
5. **API versioning best practices** – Strategy selection, deprecation, and lifecycle management.

---

## 1. API Versioning (Setup and Configuration)

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | API versioning is the mechanism that allows an API to expose multiple versions concurrently, each with its own contract and behaviour. |
| **Technical Definition** | The `Asp.Versioning` library registers versioning services via `AddApiVersioning()`, configures a default version, enables version reporting, and selects an `IApiVersionReader` (or combination of readers). Controllers/actions are annotated with `[ApiVersion]`, and `[MapToApiVersion]` selects specific actions within a controller. The library integrates with MVC via `AddMvc()` and with OpenAPI via `AddApiExplorer()`. |
| **Beginner‑Friendly Explanation** | API versioning setup is **telling ASP.NET Core how to handle versions**. You install the `Asp.Versioning` package, configure a default version and how clients will send their version (URL, header, or query string), and then decorate your controllers with `[ApiVersion]`. After that, the library automatically routes requests to the right version. |

### Purposes

- **To** enable **backward compatibility** while evolving the API.
- **To** isolate **breaking changes** into new versions, protecting existing clients.
- **To** support **multiple client generations** simultaneously.
- **To** provide **clear deprecation signals** through response headers.
- **To** generate **version‑aware OpenAPI documentation**.

### Syntax Rules and Structure

#### Installation

```bash
dotnet add package Asp.Versioning.Mvc
dotnet add package Asp.Versioning.Mvc.ApiExplorer   # For OpenAPI integration
```

#### General Syntax (Service Configuration)

```csharp
var builder = WebApplication.CreateBuilder(args);

builder.Services.AddControllers();
builder.Services.AddApiVersioning(options =>
{
    options.DefaultApiVersion = new ApiVersion(1, 0);
    options.AssumeDefaultVersionWhenUnspecified = true;
    options.ReportApiVersions = true;
    options.ApiVersionReader = new QueryStringApiVersionReader("api-version");
})
.AddMvc()
.AddApiExplorer(options =>
{
    options.GroupNameFormat = "'v'VVV";
    options.SubstituteApiVersionInUrl = true;
});
```

**Component breakdown:**

| Option | Purpose |
|--------|---------|
| `DefaultApiVersion` | The version used when the client does not specify one. |
| `AssumeDefaultVersionWhenUnspecified` | Enables default‑version fallback for unversioned requests. |
| `ReportApiVersions` | Adds `api-supported-versions` and `api-deprecated-versions` response headers. |
| `ApiVersionReader` | Determines how the version is read from the request. |
| `GroupNameFormat` | Format for OpenAPI document group names (`'v'VVV` → `v1`, `v2`). |

#### Syntax Rules

- **`AddApiVersioning()` must be called before `AddMvc()`** to register versioning services.
- **`ReportApiVersions = true`** is strongly recommended so clients can discover supported and deprecated versions.
- **`AssumeDefaultVersionWhenUnspecified = true`** allows requests without a version to hit the default version—useful for backward compatibility.
- **The version reader can be a single reader or a combination** (`ApiVersionReader.Combine(...)`).

**Constraints and limitations:**

- **Versioning affects API selection, not routing** – the version does not change the URL structure unless URI versioning is used.
- **Combining readers** requires clients to use one of the configured methods; requests with no version fall back to the default if configured.
- **The deprecated `Microsoft.AspNetCore.Mvc.Versioning` package** should not be used for new projects—use `Asp.Versioning.*`.

### Annotated Complete Step‑by‑Step Code Example

```csharp
// Program.cs
var builder = WebApplication.CreateBuilder(args);

builder.Services.AddControllers();
builder.Services.AddApiVersioning(options =>
{
    options.DefaultApiVersion = new ApiVersion(1, 0);
    options.AssumeDefaultVersionWhenUnspecified = true;
    options.ReportApiVersions = true;
    options.ApiVersionReader = new UrlSegmentApiVersionReader();   // URI versioning
})
.AddMvc()
.AddApiExplorer(options =>
{
    options.GroupNameFormat = "'v'VVV";
    options.SubstituteApiVersionInUrl = true;
});

var app = builder.Build();
app.MapControllers();
app.Run();
```

**Step‑by‑step explanation:**

1. `AddApiVersioning()` registers the versioning services with a default version of `1.0`.
2. `AssumeDefaultVersionWhenUnspecified = true` lets unversioned requests hit v1.
3. `ReportApiVersions = true` adds version headers to responses.
4. `UrlSegmentApiVersionReader` tells the library to read the version from the URL path.
5. `AddApiExplorer()` configures OpenAPI grouping (`v1`, `v2`).

**Expected Output:** The API is now version‑aware. Requests without a version are treated as v1. Responses include `api-supported-versions: 1.0` headers.

**Why this output?** The configuration sets the default version and enables version reporting; the reader determines how versions are extracted from requests.

### Real‑World Cases

- **Public APIs:** Version from day one; use URI versioning for discoverability.
- **Internal microservices:** Header versioning for cleaner URLs.
- **APIs with breaking changes:** Introduce v2 while v1 remains available.

### References

- Asp.Versioning library (GitHub) – https://github.com/dotnet/aspnet-api-versioning
- ASP.NET Core API versioning (OneUptime guide) – https://raw.githubusercontent.com/OneUptime/blog/refs/heads/master/posts/2025-12-23-aspnet-core-api-versioning/README.md

---

## 2. URI Versioning

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | URI versioning places the API version in the URL path (e.g., `/api/v1/products`), making the version explicit and discoverable. |
| **Technical Definition** | URI versioning uses `UrlSegmentApiVersionReader` (or the `{version:apiVersion}` route constraint) to extract the version from the URL segment. Controllers/actions declare the version via `[ApiVersion("1.0")]` and the route template includes `{version:apiVersion}` (e.g., `[Route("api/v{version:apiVersion}/[controller]")]`). |
| **Beginner‑Friendly Explanation** | URI versioning is the **"version in the URL"** approach. You can see the version right in the address: `/api/v1/products` is version 1, `/api/v2/products` is version 2. It is the most common and most discoverable strategy—perfect for public APIs where clarity matters. |

### Purposes

- **To** make the API version **immediately visible** in the URL.
- **To** enable **easy testing and debugging**—the version is obvious in logs and browser address bars.
- **To** support **caching and CDN routing** based on URL paths.
- **To** provide the most **discoverable** versioning strategy for public APIs.
- **To** simplify **client migration**—clients simply change the URL.

### Syntax Rules and Structure

#### General Syntax (Controller with Versioned Route)

```csharp
[ApiController]
[Route("api/v{version:apiVersion}/[controller]")]
[ApiVersion("1.0")]
public class ProductsController : ControllerBase
{
    [HttpGet]
    public IActionResult GetAll() => Ok(/* v1 data */);
}
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `api/v{version:apiVersion}/[controller]` | Route template with version segment. |
| `{version:apiVersion}` | Route parameter constrained to a valid API version. |
| `[ApiVersion("1.0")]` | Declares the controller's version. |

#### General Syntax (Multiple Versions in One Controller)

```csharp
[ApiController]
[Route("api/v{version:apiVersion}/products")]
[ApiVersion("1.0")]
[ApiVersion("2.0")]
public class ProductsController : ControllerBase
{
    [HttpGet]
    [MapToApiVersion("1.0")]
    public IActionResult GetAllV1() => Ok(/* v1 */);

    [HttpGet]
    [MapToApiVersion("2.0")]
    public IActionResult GetAllV2() => Ok(/* v2 */);

    [HttpGet("{id}")]   // Works for both versions
    public IActionResult GetById(int id) => Ok(/* shared */);
}
```

#### General Syntax (Separate Controllers per Version)

```csharp
[ApiController]
[Route("api/v{version:apiVersion}/[controller]")]
[ApiVersion("1.0")]
public class ProductsV1Controller : ControllerBase { ... }

[ApiController]
[Route("api/v{version:apiVersion}/[controller]")]
[ApiVersion("2.0")]
public class ProductsV2Controller : ControllerBase { ... }
```

**Syntax rules:**

- **`{version:apiVersion}`** is the route constraint that validates and extracts the version.
- **`[MapToApiVersion]`** selects which actions belong to which version when multiple versions share a controller.
- **Shared actions** (without `[MapToApiVersion]`) are available in all versions declared on the controller.
- **Separate controllers** per version are clearer for large breaking changes; shared controllers work for small differences.

**Constraints and limitations:**

- **URLs change between versions** – clients must update their base URLs when migrating.
- **Version interleaving** (multiple versions in one controller) should be used sparingly to prevent developer confusion.
- **Route order** – ensure the versioned route does not conflict with unversioned routes.

### Annotated Complete Step‑by‑Step Code Example

```csharp
[ApiController]
[Route("api/v{version:apiVersion}/products")]
[ApiVersion("1.0")]
[ApiVersion("2.0")]
public class ProductsController : ControllerBase
{
    [HttpGet]
    [MapToApiVersion("1.0")]
    public IActionResult GetAllV1() =>
        Ok(new[] { new { Id = 1, Name = "Laptop" } });

    [HttpGet]
    [MapToApiVersion("2.0")]
    public IActionResult GetAllV2() =>
        Ok(new[] { new { Id = 1, Name = "Laptop", Category = "Electronics" } });

    [HttpGet("{id}")]
    public IActionResult GetById(int id) =>
        Ok(new { Id = id, Name = $"Product {id}" });
}
```

**Step‑by‑step explanation:**

1. The route template includes `{version:apiVersion}` so the version is part of the URL.
2. `[ApiVersion("1.0")]` and `[ApiVersion("2.0")]` declare that this controller serves both versions.
3. `GetAllV1` is mapped to v1 via `[MapToApiVersion("1.0")]`.
4. `GetAllV2` is mapped to v2 via `[MapToApiVersion("2.0")]`.
5. `GetById` has no version mapping, so it is available in both versions.

**Expected Output:**

| Request | Response |
|---------|----------|
| `GET /api/v1/products` | v1 data (Id, Name only) |
| `GET /api/v2/products` | v2 data (Id, Name, Category) |
| `GET /api/v1/products/42` | Shared GetById response |
| `GET /api/v2/products/42` | Shared GetById response |

**Why this output?** URI versioning extracts the version from the URL; `MapToApiVersion` selects the appropriate action; shared actions are available across declared versions.

### Real‑World Cases

- **Public REST APIs:** `/api/v1/users`, `/api/v2/users` (e.g., GitHub, Stripe).
- **E‑commerce APIs:** Versioning product and order endpoints as the schema evolves.
- **Microservices:** URI versioning for discoverable inter‑service contracts.

### References

- URL Path Versioning (dotnet.github.io) – https://dotnet.github.io/aspnet-api-versioning/aspnet-core/how-to/version-by-url-segment.html
- ASP.NET Core microservice versioning (Microsoft Learn) – https://learn.microsoft.com/ko-kr/dotnet/architecture/microservices/multi-container-microservice-net-applications/data-driven-crud-microservice

---

## 3. Header Versioning

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Header versioning places the API version in a custom HTTP request header (e.g., `X-API-Version: 2.0`), keeping URLs clean and unchanged across versions. |
| **Technical Definition** | Header versioning uses `HeaderApiVersionReader("X-API-Version")` to read the version from a request header. Controllers/actions declare versions via `[ApiVersion]`, and the library matches the header value to the appropriate version. The URL remains identical across versions (e.g., `/api/products` works for both v1 and v2). |
| **Beginner‑Friendly Explanation** | Header versioning is the **"version in the envelope"** approach. The URL stays the same (`/api/products`), but the client sends a header (`X-API-Version: 2.0`) to say which version it wants. This keeps URLs clean and stable—ideal for internal APIs where clients are controlled and can easily set headers. |

### Purposes

- **To** keep **URLs clean and stable** across versions.
- **To** support **internal APIs** where clients are controlled and can set headers.
- **To** avoid **URL churn** when versions change.
- **To** separate **resource identity** (URL) from **representation version** (header).
- **To** support **content negotiation** alongside versioning.

### Syntax Rules and Structure

#### General Syntax (Configuration)

```csharp
builder.Services.AddApiVersioning(options =>
{
    options.ApiVersionReader = new HeaderApiVersionReader("X-API-Version");
});
```

#### General Syntax (Controller)

```csharp
[ApiController]
[Route("api/[controller]")]
[ApiVersion("1.0")]
public class ProductsController : ControllerBase
{
    [HttpGet]
    public IActionResult GetAll() => Ok(/* v1 data */);
}
```

#### General Syntax (Request)

```http
GET /api/products HTTP/1.1
Host: localhost
X-API-Version: 1.0
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `HeaderApiVersionReader("X-API-Version")` | Reads the version from the specified header. |
| `X-API-Version` | The custom header name (configurable). |
| `[ApiVersion("1.0")]` | Declares the controller's version. |

**Syntax rules:**

- **The header name is configurable** – `X-API-Version`, `x-ms-version`, or any custom header.
- **`AssumeDefaultVersionWhenUnspecified = true`** allows requests without the header to hit the default version.
- **Multiple headers can be supported** by combining readers (`ApiVersionReader.Combine`).

**Constraints and limitations:**

- **Not explicit in URLs** – debugging and caching are less straightforward than URI versioning.
- **Clients must remember to send the header** – missing headers fall back to the default version (or fail if no default is configured).
- **API gateways and proxies** must forward the header correctly.

### Annotated Complete Step‑by‑Step Code Example

```csharp
builder.Services.AddApiVersioning(options =>
{
    options.DefaultApiVersion = new ApiVersion(1, 0);
    options.AssumeDefaultVersionWhenUnspecified = true;
    options.ReportApiVersions = true;
    options.ApiVersionReader = new HeaderApiVersionReader("x-ms-version");
});
```

```csharp
[ApiController]
[Route("api/[controller]")]
[ApiVersion("1.0")]
public class HelloWorldController : ControllerBase
{
    [HttpGet]
    public string Get() => "Hello world from v1!";
}

[ApiController]
[Route("api/[controller]")]
[ApiVersion("2.0")]
public class HelloWorldV2Controller : ControllerBase
{
    [HttpGet]
    public string Get() => "Hello world from v2!";
}
```

**Expected Output:**

| Request | Header | Response |
|---------|--------|----------|
| `GET /api/helloworld` | `x-ms-version: 1.0` | "Hello world from v1!" |
| `GET /api/helloworld` | `x-ms-version: 2.0` | "Hello world from v2!" |
| `GET /api/helloworld` | (no header) | "Hello world from v1!" (default) |

**Why this output?** The `HeaderApiVersionReader` extracts the version from the `x-ms-version` header; requests without the header fall back to the default version (v1).

### Real‑World Cases

- **Internal microservices:** Clean URLs, header‑based versioning between services.
- **APIs with API gateways:** Gateways can inject or forward version headers.
- **Enterprise APIs:** Versioning without changing resource URLs.

### References

- Header Versioning (dotnet.github.io) – https://dotnet.github.io/aspnet-api-versioning/aspnet-core/how-to/version-by-header.html
- Versioning strategies comparison – https://github.com/codewithmukesh/dotnet-claude-kit/blob/main/skills/api-versioning/SKILL.md

---

## 4. Query Versioning

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Query string versioning places the API version in a query parameter (e.g., `?api-version=2.0`), keeping URLs stable while making the version easy to add or change. |
| **Technical Definition** | Query string versioning uses `QueryStringApiVersionReader("api-version")` to read the version from a query parameter. Controllers/actions declare versions via `[ApiVersion]`. Requests without the parameter fall back to the default version if `AssumeDefaultVersionWhenUnspecified = true`. |
| **Beginner‑Friendly Explanation** | Query versioning is the **"version as a parameter"** approach. The URL stays the same (`/api/products`), and the client appends `?api-version=2.0` to choose a version. It is easy to test (just add the parameter in the browser) and does not require special headers—but it can clutter URLs and is less explicit than URI versioning. |

### Purposes

- **To** keep **URLs stable** while allowing version selection via a query parameter.
- **To** make versioning **easy to test**—just add `?api-version=1.0` to the URL.
- **To** support **simple client implementations** that may struggle with custom headers.
- **To** provide an **alternative** when URI changes are undesirable and headers are impractical.

### Syntax Rules and Structure

#### General Syntax (Configuration)

```csharp
builder.Services.AddApiVersioning(options =>
{
    options.DefaultApiVersion = new ApiVersion(1, 0);
    options.AssumeDefaultVersionWhenUnspecified = true;
    options.ApiVersionReader = new QueryStringApiVersionReader("api-version");
});
```

#### General Syntax (Controller)

```csharp
[ApiController]
[Route("api/[controller]")]
[ApiVersion("1.0")]
public class HelloWorldController : ControllerBase
{
    [HttpGet]
    public string Get() => "Hello world!";
}

[ApiController]
[Route("api/[controller]")]
[ApiVersion("2.0")]
public class HelloWorldV2Controller : ControllerBase
{
    [HttpGet]
    public string Get() => "Hello world!";
}
```

#### General Syntax (Request)

```http
GET /api/helloworld?api-version=1.0
GET /api/helloworld?api-version=2.0
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `QueryStringApiVersionReader("api-version")` | Reads the version from the specified query parameter. |
| `api-version` | The query parameter name (configurable). |

**Syntax rules:**

- **The query parameter name is configurable** – `api-version`, `version`, or any custom name.
- **Requests without the parameter** fall back to the default version if `AssumeDefaultVersionWhenUnspecified = true`.
- **The parameter must be on every request** for explicit version selection.

**Constraints and limitations:**

- **Query parameters are visible in logs and browser history** – less "clean" than header versioning.
- **Caching** – query strings are part of the cache key, which can cause cache fragmentation if clients use different version parameters.
- **Clients must remember to include the parameter** – missing it falls back to the default.

### Annotated Complete Step‑by‑Step Code Example

```csharp
builder.Services.AddApiVersioning(options =>
{
    options.DefaultApiVersion = new ApiVersion(1, 0);
    options.AssumeDefaultVersionWhenUnspecified = true;
    options.ReportApiVersions = true;
    options.ApiVersionReader = new QueryStringApiVersionReader("api-version");
});
```

```csharp
[ApiController]
[Route("api/[controller]")]
[ApiVersion("1.0")]
public class HelloWorldController : ControllerBase
{
    [HttpGet]
    public string Get() => "Hello world from v1!";
}

[ApiController]
[Route("api/[controller]")]
[ApiVersion("2.0")]
public class HelloWorldV2Controller : ControllerBase
{
    [HttpGet]
    public string Get() => "Hello world from v2!";
}
```

**Expected Output:**

| Request | Response |
|---------|----------|
| `GET /api/helloworld?api-version=1.0` | "Hello world from v1!" |
| `GET /api/helloworld?api-version=2.0` | "Hello world from v2!" |
| `GET /api/helloworld` | "Hello world from v1!" (default) |

**Why this output?** The `QueryStringApiVersionReader` extracts the version from the `api-version` parameter; requests without it fall back to the default version.

### Real‑World Cases

- **Simple APIs:** Query versioning is easy for clients to adopt.
- **Prototyping:** Quick versioning without URL changes.
- **Public APIs with API keys:** Combine query versioning with other query parameters.

### References

- Query String Versioning (dotnet.github.io) – https://dotnet.github.io/aspnet-api-versioning/aspnet-core/how-to/version-by-query-string.html

---

## 5. API Versioning Best Practices

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | API versioning best practices are guidelines for choosing a versioning strategy, managing version lifecycles, and communicating changes to clients in a way that minimizes disruption and maximizes clarity. |
| **Technical Definition** | Best practices span **strategy selection** (URI for public APIs, header for internal services), **backward‑compatible changes** (additive fields do not require a version bump), **deprecation signalling** (`Deprecated = true`, response headers, sunset policies), **version lifecycle management** (support at least two versions, announce deprecation early), **OpenAPI integration** (separate documents per version), and **semantic versioning discipline** (major = breaking, minor = backward‑compatible). |
| **Beginner‑Friendly Explanation** | Best practices are the **"do's and don'ts"** for API versioning—they help you avoid unnecessary version churn, support clients through transitions, and keep your API maintainable over time. |

### Purposes

- **To** choose the **right versioning strategy** for the API's audience and usage pattern.
- **To** avoid **unnecessary version bumps** for backward‑compatible changes.
- **To** give clients **clear deprecation signals** and enough time to migrate.
- **To** maintain **manageable version lifecycles** (support current + previous, retire old versions).
- **To** keep **OpenAPI documentation** aligned with each version.

### Guidelines Checklist

| Area | Do | Don't |
|------|----|-------|
| **Strategy selection** | Use **URI versioning** for public APIs (most discoverable, cache‑friendly). | Mix strategies inconsistently across the API. |
| **Strategy selection** | Use **header versioning** for internal APIs (clean URLs). | Use header versioning for public APIs where debugging clarity matters. |
| **Breaking changes** | Version only for **breaking changes** (removing fields, changing types, altering behaviour). | Bump the version for **additive** changes (new optional fields). |
| **Semantic versioning** | Major = breaking, minor = backward‑compatible. | Use arbitrary version numbers (v1, v2, v3 without meaning). |
| **Default version** | Set `AssumeDefaultVersionWhenUnspecified = true` for backward compatibility. | Require every client to specify a version from day one without a default. |
| **Version reporting** | Enable `ReportApiVersions = true` so clients see `api-supported-versions` and `api-deprecated-versions`. | Hide the supported versions from clients. |
| **Deprecation** | Mark versions `[ApiVersion("1.0", Deprecated = true)]`; announce deprecation early. | Remove a version without warning. |
| **Lifecycle** | Support at least **two versions** (current + previous); monitor version usage before retiring. | Retire a version while clients still depend on it. |
| **Sunset policy** | Publish sunset dates and a migration guide. | Leave clients guessing when a version will disappear. |
| **OpenAPI** | Generate **separate documents** per version (`GroupNameFormat = "'v'VVV"`). | Mix all versions into a single OpenAPI document. |
| **Version interleaving** | Use `[MapToApiVersion]` for small differences within one controller. | Interleave many versions in one controller—it becomes confusing. |
| **Testing** | Test each version independently and test version negotiation errors. | Test only the default version. |

### Annotated Code Example (Best‑Practice Versioning Setup)

```csharp
// Program.cs
var builder = WebApplication.CreateBuilder(args);

builder.Services.AddControllers();
builder.Services.AddApiVersioning(options =>
{
    options.DefaultApiVersion = new ApiVersion(1, 0);
    options.AssumeDefaultVersionWhenUnspecified = true;
    options.ReportApiVersions = true;                     // api-supported-versions header
    options.ApiVersionReader = ApiVersionReader.Combine(
        new UrlSegmentApiVersionReader(),                  // Primary: URI versioning
        new HeaderApiVersionReader("X-API-Version"));      // Fallback: header versioning
})
.AddMvc()
.AddApiExplorer(options =>
{
    options.GroupNameFormat = "'v'VVV";                    // v1, v2, v3
    options.SubstituteApiVersionInUrl = true;              // OpenAPI per version
});

var app = builder.Build();
app.MapControllers();
app.Run();
```

```csharp
// ProductsController.cs
[ApiController]
[Route("api/v{version:apiVersion}/products")]
[ApiVersion("1.0", Deprecated = true)]    // v1 deprecated
[ApiVersion("2.0")]                       // v2 current
public class ProductsController : ControllerBase
{
    [HttpGet]
    [MapToApiVersion("1.0")]
    public IActionResult GetAllV1() => Ok(/* v1 data */);

    [HttpGet]
    [MapToApiVersion("2.0")]
    public IActionResult GetAllV2() => Ok(/* v2 data */);
}
```

**Why this follows best practices:**

- **URI versioning is the primary strategy** (discoverable, cache‑friendly) with **header versioning as a fallback** for flexibility.
- **`ReportApiVersions = true`** adds `api-supported-versions` and `api-deprecated-versions` response headers.
- **v1 is marked `Deprecated = true`**, signalling clients to migrate.
- **OpenAPI grouping** generates separate documents per version.
- **`MapToApiVersion`** cleanly separates actions within the controller.
- **`AssumeDefaultVersionWhenUnspecified = true`** preserves backward compatibility for unversioned clients.

### Real‑World Cases

- **Public APIs:** URI versioning from day one; deprecation headers; support current + previous.
- **Internal microservices:** Header versioning; monitor usage before retiring versions.
- **APIs with OpenAPI:** Per‑version documents (`/openapi/v1.json`, `/openapi/v2.json`).

### References

- API versioning best practices (SSW Rules) – https://www.ssw.com.au/rules/do-you-use-api-versioning-to-avoid-breaking-changes/
- Combining API versioning with OpenAPI in .NET 10 – https://devblogs.microsoft.com/dotnet/api-versioning-in-dotnet-10-applications/
- Semantic versioning discipline for APIs – https://abp.io/community/articles/best-practices-for-designing-backwardcompatible-rest-apis-in-a-microservice-solution-for-net-developers

---

## Summary Table: API Versioning Strategies at a Glance

| Strategy | Example | Pros | Cons | Best For |
|----------|---------|------|------|----------|
| **URI Versioning** | `/api/v1/products` | Most discoverable, cache‑friendly, easy to test | URLs change between versions | Public APIs, new APIs from day one |
| **Header Versioning** | `X-API-Version: 2.0` | Clean URLs, explicit control, no URL churn | Less discoverable, harder to test in browser | Internal APIs, microservices |
| **Query Versioning** | `/api/products?api-version=2.0` | Easy to test, no custom headers | Clutters URLs, cache fragmentation | Simple APIs, prototyping |
| **Media Type Versioning** | `Accept: application/json; v=2.0` | REST‑pure, content negotiation | Complex, requires custom formatters | Standards‑heavy APIs (e.g., GitHub) |

---

## References (Full List)

- Asp.Versioning library (GitHub) – https://github.com/dotnet/aspnet-api-versioning
- URL Path Versioning (dotnet.github.io) – https://dotnet.github.io/aspnet-api-versioning/aspnet-core/how-to/version-by-url-segment.html
- Header Versioning (dotnet.github.io) – https://dotnet.github.io/aspnet-api-versioning/aspnet-core/how-to/version-by-header.html
- Query String Versioning (dotnet.github.io) – https://dotnet.github.io/aspnet-api-versioning/aspnet-core/how-to/version-by-query-string.html
- ASP.NET Core API versioning (OneUptime guide) – https://raw.githubusercontent.com/OneUptime/blog/refs/heads/master/posts/2025-12-23-aspnet-core-api-versioning/README.md
- Combining API versioning with OpenAPI in .NET 10 – https://devblogs.microsoft.com/dotnet/api-versioning-in-dotnet-10-applications/
- API versioning best practices (SSW Rules) – https://www.ssw.com.au/rules/do-you-use-api-versioning-to-avoid-breaking-changes/
- Versioning strategies comparison (codewithmukesh) – https://github.com/codewithmukesh/dotnet-claude-kit/blob/main/skills/api-versioning/SKILL.md
- Semantic versioning discipline for APIs – https://abp.io/community/articles/best-practices-for-designing-backwardcompatible-rest-apis-in-a-microservice-solution-for-net-developers
- ASP.NET Core microservice versioning (Microsoft Learn) – https://learn.microsoft.com/ko-kr/dotnet/architecture/microservices/multi-container-microservice-net-applications/data-driven-crud-microservice
- Deprecation and version reporting (Granit FX) – https://granit-fx.dev
- Asp.Versioning migration guide (deprecated Microsoft packages) – https://dotnet.github.io/aspnet-api-versioning/aspnet-core/migration.html