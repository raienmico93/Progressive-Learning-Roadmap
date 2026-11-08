# ASP.NET Core Response Compression: A Comprehensive Cheat Sheet

> **Scope & Version Note:** This cheat sheet targets **ASP.NET Core** (.NET 6–10). Response compression middleware reduces payload sizes by compressing HTTP responses using **Brotli** or **Gzip** before sending them to the client. Brotli has been the default provider since ASP.NET Core 3.0, with automatic fallback to Gzip when the client does not advertise Brotli support. A significant **breaking change arrives in .NET 11**: the middleware will append `Vary: Accept-Encoding` to **every** response it sees, not only compressed ones, to ensure correct cache behaviour. Each requested topic is treated independently—no sections are merged.

---

## Topic Overview

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Response compression is the technique of compressing HTTP response bodies before transmission to reduce bandwidth usage and improve client-perceived latency. |
| **Technical Definition** | The ASP.NET Core **Response Compression Middleware** (`ResponseCompressionMiddleware`) intercepts responses, checks the client's `Accept-Encoding` header, selects the best available compression provider (Brotli `br` → Gzip → identity), compresses the body, and sets the `Content-Encoding` header. Configuration is via `AddResponseCompression(options => ...)` and the middleware is activated with `UseResponseCompression()`. |
| **Beginner-Friendly Explanation** | Response compression is the **"shrink the package before shipping"** technique. When your API returns a large JSON payload, the middleware compresses it (like zipping a file) before sending it over the network. The browser unzips it automatically. This makes responses smaller and faster to download—especially important for mobile users and bandwidth-constrained clients. |

### Key Characteristics

- **Brotli by default** – ASP.NET Core prefers Brotli (`br`) when the client supports it, falling back to Gzip.
- **Automatic content negotiation** – The middleware reads `Accept-Encoding` and selects the best supported algorithm.
- **Configurable providers** – Brotli, Gzip, and custom `ICompressionProvider` implementations.
- **MIME-type gating** – Only configured MIME types are compressed; `text/*` wildcards are **not** supported.
- **HTTPS-disabled by default** – `EnableForHttps = false` to prevent BREACH/CRIME attacks; must be explicitly enabled if needed.
- **Vary header** – Emitted for correct cache behaviour (unconditionally from .NET 11).

### Prerequisites

- .NET SDK (6.0+ recommended), ASP.NET Core project.
- `Microsoft.AspNetCore.ResponseCompression` (included in the shared framework).
- Working knowledge of **middleware** and **dependency injection**.
- Familiarity with **HTTP headers** (`Accept-Encoding`, `Content-Encoding`, `Vary`).

### Related Programming Areas

- **Performance** – Compression is a core latency and bandwidth optimisation.
- **Middleware pipeline** – `UseResponseCompression()` must be placed correctly.
- **Caching** – Compressed responses interact with caching via `Vary: Accept-Encoding`.
- **Security** – HTTPS compression carries BREACH/CRIME risk.
- **MIME types** – Only compressible types should be compressed.

### Core Concepts / Features

1. **Response compression** – The overall mechanism, middleware, and pipeline placement.
2. **Gzip** – The widely supported, fast compression format.
3. **Brotli** – The modern, higher-ratio format (default in ASP.NET Core).
4. **Response compression best practices** – MIME types, compression levels, HTTPS, and security.

---

## 1. Response Compression (The Middleware)

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Response compression middleware is the ASP.NET Core component that compresses HTTP response bodies before they are sent to the client, based on the client's `Accept-Encoding` header. |
| **Technical Definition** | `ResponseCompressionMiddleware` (registered via `AddResponseCompression()`, activated via `UseResponseCompression()`) inspects the response `Content-Type` against configured `MimeTypes`, checks the request `Accept-Encoding` header, selects a registered `ICompressionProvider` (Brotli or Gzip), wraps the response body stream in a compression stream, and sets `Content-Encoding`. It also appends `Vary: Accept-Encoding` so caches can store separate variants. |
| **Beginner-Friendly Explanation** | The middleware is the **"packaging station"** on the outbound conveyor belt. Every response passes through it. If the client says "I can accept Brotli or Gzip" and the response is a compressible type like JSON, the middleware compresses it and labels it with `Content-Encoding: br` or `gzip`. The client unzips it on arrival. If the client doesn't support compression or the response is already compressed (like a PNG), it passes through untouched. |

### Purposes

- **To** **reduce response payload size** — often by 60–80% for JSON and text.
- **To** **improve client-perceived latency** — smaller payloads download faster, especially on slow connections.
- **To** **reduce bandwidth costs** — less data transferred from server to client.
- **To** **offload work from the network** — compression trades CPU for bandwidth, a favorable trade in most web scenarios.

### Syntax Rules and Structure

#### General Syntax (Service Registration)

```csharp
builder.Services.AddResponseCompression(options =>
{
    options.EnableForHttps = true;                                    // Security trade-off
    options.Providers.Add<BrotliCompressionProvider>();               // Preferred
    options.Providers.Add<GzipCompressionProvider>();                 // Fallback
    options.MimeTypes = ResponseCompressionDefaults.MimeTypes.Concat(
        new[] { "image/svg+xml" });                                   // Add custom types
});
```

#### General Syntax (Middleware Activation)

```csharp
var app = builder.Build();
app.UseResponseCompression();   // Must be before any middleware that generates compressible responses
app.UseStaticFiles();
app.UseRouting();
app.MapControllers();
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `AddResponseCompression` | Registers the middleware and options. |
| `EnableForHttps` | Whether to compress HTTPS responses (default `false`). |
| `Providers` | The ordered list of compression providers (Brotli first). |
| `MimeTypes` | The content types eligible for compression. |
| `UseResponseCompression()` | Activates the middleware in the pipeline. |

**Syntax rules:**

- **`UseResponseCompression()` must be called early** —before static files, routing, and endpoints, so it can compress the responses they generate.
- **Provider order matters** —the first provider that the client accepts is used; Brotli is preferred, so add it first.
- **MIME types are explicit** —only listed types are compressed; `text/*` wildcards are **not** supported.

**Constraints and limitations:**

- **HTTPS compression is disabled by default** —enabling it exposes the app to BREACH/CRIME attacks unless antiforgery tokens are used.
- **Already-compressed assets** (PNG, JPEG, WOFF2) should **not** be compressed —the overhead outweighs the negligible size reduction.
- **Small responses** (< 150–1000 bytes) may become **larger** after compression due to overhead.

### Annotated Complete Step-by-Step Code Example

```csharp
var builder = WebApplication.CreateBuilder(args);

builder.Services.AddResponseCompression(options =>
{
    options.EnableForHttps = false;                          // Security-safe default
    options.Providers.Add<BrotliCompressionProvider>();      // Preferred
    options.Providers.Add<GzipCompressionProvider>();        // Fallback
    // Default MIME types include text/plain, text/css,
    // application/javascript, application/json, application/xml
});

builder.Services.Configure<BrotliCompressionProviderOptions>(options =>
{
    options.Level = CompressionLevel.Fastest;                // Lower CPU, faster compression
});

builder.Services.Configure<GzipCompressionProviderOptions>(options =>
{
    options.Level = CompressionLevel.Fastest;
});

var app = builder.Build();

app.UseResponseCompression();   // Before static files and endpoints
app.UseStaticFiles();
app.UseRouting();
app.MapControllers();
app.Run();
```

**Step-by-step explanation:**

1. `AddResponseCompression` registers the middleware with Brotli (preferred) and Gzip (fallback) providers.
2. `EnableForHttps = false` keeps the security-safe default —compression over HTTPS is opt-in.
3. `CompressionLevel.Fastest` is configured for both providers, favoring CPU efficiency over maximum compression ratio.
4. `UseResponseCompression()` is placed before static files and routing so both static and dynamic responses can be compressed.

**Expected Output:** JSON API responses and text-based static files are compressed with Brotli (or Gzip if Brotli is unsupported). The response includes `Content-Encoding: br` (or `gzip`) and `Vary: Accept-Encoding`.

**Why this output?** The middleware selects the best provider based on `Accept-Encoding` and compresses eligible MIME types.

### Real-World Cases

- **JSON-heavy APIs:** Compressing JSON responses reduces payload size by 60–80%, directly improving mobile app performance.
- **SPA hosting:** Compressing JavaScript, CSS, and HTML reduces initial load time.
- **Public APIs with global users:** Bandwidth savings are especially significant for users on slow connections.

### References

- Response compression in ASP.NET Core – https://learn.microsoft.com/en-us/aspnet/core/performance/response-compression
- `UseResponseCompression` placement – https://learn.microsoft.com/th-th/aspnet/core/performance/response-compression?view=aspnetcore-9.0#2

---

## 2. Gzip (The Universal Fallback)

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Gzip is a widely supported lossless compression format based on the DEFLATE algorithm, used for compressing HTTP responses and supported by virtually every browser and HTTP client. |
| **Technical Definition** | `GzipCompressionProvider` (configured via `GzipCompressionProviderOptions`) implements Gzip compression using `System.IO.Compression`. It uses `CompressionLevel.Fastest` by default, which prioritizes speed over compression ratio. The client advertises `Accept-Encoding: gzip` to request it. |
| **Beginner-Friendly Explanation** | Gzip is the **"universal zipper"** —it has been supported by every browser and server for decades. It is fast to compress, so it is a good choice when CPU is limited or when responses are generated dynamically. Brotli often produces smaller files, but Gzip is the safe fallback that always works. |

### Purposes

- **To** provide **universal compatibility** —every browser, proxy, and HTTP client supports Gzip.
- **To** offer **fast compression** —significantly faster than Brotli at higher compression levels.
- **To** serve as the **automatic fallback** when Brotli is not supported.
- **To** reduce bandwidth for **dynamically generated content** where compression speed matters more than the last few percentage points of ratio.

### Syntax Rules and Structure

```csharp
options.Providers.Add<GzipCompressionProvider>();

builder.Services.Configure<GzipCompressionProviderOptions>(options =>
{
    options.Level = CompressionLevel.Fastest;   // Default; use Optimal for smaller output
});
```

**Syntax rules:**

- **Add the Gzip provider** after Brotli (so Brotli is preferred when supported).
- **Configure `CompressionLevel`** —`Fastest` (default) minimizes CPU; `Optimal` produces smaller output at higher CPU cost.
- **Gzip is the default fallback** —if no providers are explicitly added, Gzip is used by default alongside Brotli.

**Constraints and limitations:**

- **Larger output than Brotli** —typically 15–25% larger for text content, though still a massive reduction from uncompressed.
- **Slower than Brotli at decompression?** —No: Gzip decompression is generally comparable to Brotli. The trade-off is on the compression side.

### Annotated Code Example

```csharp
builder.Services.AddResponseCompression(options =>
{
    options.Providers.Add<BrotliCompressionProvider>();
    options.Providers.Add<GzipCompressionProvider>();
});

builder.Services.Configure<GzipCompressionProviderOptions>(options =>
{
    options.Level = CompressionLevel.Optimal;   // Smaller output for static assets
});
```

**Expected Output:** Clients that support only Gzip receive `Content-Encoding: gzip` responses.

**Why this output?** The Gzip provider handles clients that do not advertise Brotli support.

### Real-World Cases

- **Legacy clients:** Older browsers and HTTP clients that do not support Brotli.
- **CPU-constrained servers:** When dynamic compression speed matters more than ratio.
- **Fallback for all APIs:** Gzip ensures compression always works, even if Brotli is unavailable.

### References

- Gzip compression provider – https://learn.microsoft.com/de-at/aspnet/core/performance/response-compression?view=aspnetcore-8.0#3
- Compression levels – https://learn.microsoft.com/th-th/aspnet/core/performance/response-compression?view=aspnetcore-7.0#2

---

## 3. Brotli (The Modern Default)

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Brotli is a modern lossless compression algorithm (RFC 7932) optimized for web content, providing better compression ratios than Gzip while maintaining fast decompression. |
| **Technical Definition** | `BrotliCompressionProvider` implements Brotli compression using `System.IO.Compression`. It is the **default provider** in ASP.NET Core when no providers are explicitly added. It uses `CompressionLevel.Fastest` by default, which is **less efficient** than `Optimal` but faster. The client advertises `Accept-Encoding: br` to request it. |
| **Beginner-Friendly Explanation** | Brotli is the **"modern, tighter zipper"**. It produces smaller files than Gzip—often 15–25% smaller—while decompressing just as fast (or faster). The catch is that **compressing** with Brotli is slower than Gzip, so it is best for static content or when compression time is not critical. It is the default in ASP.NET Core because most traffic benefits from the smaller payload. |

### Purposes

- **To** achieve **smaller payloads** than Gzip —especially for text, JSON, HTML, and JavaScript.
- **To** serve as the **default provider** in ASP.NET Core —preferred whenever the client supports it.
- **To** improve **cache efficiency** —smaller responses fit more easily in caches.
- **To** provide **fast decompression** —Brotli decompression is at least as fast as Gzip.

### Syntax Rules and Structure

```csharp
options.Providers.Add<BrotliCompressionProvider>();   // Add first — preferred

builder.Services.Configure<BrotliCompressionProviderOptions>(options =>
{
    options.Level = CompressionLevel.Fastest;   // Default; use Optimal for smaller output
});
```

**Syntax rules:**

- **Add Brotli first** in the providers list —the middleware selects the first provider the client accepts.
- **`CompressionLevel.Fastest` is the default** —it produces less compression than `Optimal` but compresses faster.
- **Use `CompressionLevel.Optimal`** for static assets where maximum compression is worth the CPU cost.

**Constraints and limitations:**

- **Compression is slower than Gzip** —on large files, Brotli can be orders of magnitude slower to compress (e.g., 1.8s vs 0.05s on a 1.6 MB JS file).
- **Best for static/cacheable content** —dynamic responses may favor Gzip's speed.
- **Browser support is universal in modern browsers** —but very old clients may not advertise Brotli, in which case Gzip is used.

### Annotated Code Example

```csharp
builder.Services.AddResponseCompression(options =>
{
    options.Providers.Add<BrotliCompressionProvider>();
    options.Providers.Add<GzipCompressionProvider>();
});

builder.Services.Configure<BrotliCompressionProviderOptions>(options =>
{
    options.Level = CompressionLevel.Fastest;   // Balance CPU vs ratio
});
```

**Expected Output:** Clients that advertise `Accept-Encoding: br` receive Brotli-compressed responses (`Content-Encoding: br`); clients that only support Gzip receive Gzip.

**Why this output?** Brotli is listed first, so the middleware selects it when the client supports it.

### Real-World Cases

- **Static asset compression:** Brotli's higher ratio is ideal for JavaScript, CSS, and HTML bundles.
- **JSON API responses:** Smaller payloads improve mobile performance.
- **CDN integration:** Modern CDNs support Brotli, so origin responses benefit from the higher ratio.

### References

- Brotli compression provider – https://learn.microsoft.com/de-at/aspnet/core/performance/response-compression?view=aspnetcore-8.0#3
- Brotli vs Gzip comparison – https://www.ioriver.io/gzip-vs-brotli-compression

---

## 4. Response Compression Best Practices

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Response compression best practices are guidelines for configuring compression to maximize performance benefits while avoiding security vulnerabilities, CPU waste, and cache poisoning. |
| **Technical Definition** | Best practices span **MIME-type selection** (only compress compressible types), **compression level tuning** (`Fastest` vs `Optimal`), **HTTPS security** (`EnableForHttps` and BREACH/CRIME mitigation), **middleware ordering** (before static files and endpoints), **cache correctness** (`Vary: Accept-Encoding`), and **small-response avoidance** (do not compress payloads under ~150–1000 bytes). |
| **Beginner-Friendly Explanation** | Best practices are the **"do's and don'ts"** for compression—they help you avoid compressing files that are already compressed, wasting CPU on tiny responses, exposing HTTPS traffic to BREACH attacks, and breaking caches by forgetting the `Vary` header. |

### Purposes

- **To** maximize the **performance benefit** of compression while minimizing CPU cost.
- **To** avoid **security vulnerabilities** from HTTPS compression.
- **To** prevent **cache poisoning** through correct `Vary` headers.
- **To** avoid **compressing the wrong content** —already-compressed assets and tiny files.
- **To** ensure **correct middleware placement** so compression applies to all eligible responses.

### Guidelines Checklist

| Area | Do | Don't |
|------|----|-------|
| **MIME types** | Compress **text-based types**: JSON, XML, HTML, CSS, JS, SVG. | Compress **already-compressed** types: PNG, JPEG, WOFF2, ZIP, MP4. |
| **Small responses** | Skip compression for payloads **under ~150–1000 bytes**. | Compress tiny responses—overhead may make them **larger**. |
| **HTTPS** | Keep `EnableForHttps = false` unless you have **antiforgery tokens** and understand BREACH/CRIME risk. | Enable HTTPS compression blindly—it exposes the app to BREACH/CRIME attacks. |
| **Middleware order** | Call `UseResponseCompression()` **before** static files, routing, and endpoints. | Place it after middleware that generates compressible responses—compression won't apply. |
| **Compression level** | Use `CompressionLevel.Fastest` for dynamic content; `Optimal` for static assets. | Use `Optimal` everywhere—it can be **orders of magnitude slower**. |
| **Vary header** | Rely on the middleware's `Vary: Accept-Encoding` (unconditional from .NET 11). | Strip `Vary` unless you have a specific, documented reason. |
| **MIME wildcards** | Add explicit types via `MimeTypes`. | Expect `text/*` wildcards to work—they are **not supported**. |
| **Proxy compatibility** | Ensure Nginx **does not strip** `Accept-Encoding` (a known issue). | Assume compression works behind a proxy without verifying. |

### Annotated Code Example (Best-Practice Configuration)

```csharp
builder.Services.AddResponseCompression(options =>
{
    options.EnableForHttps = false;                          // Security-safe
    options.Providers.Add<BrotliCompressionProvider>();      // Preferred
    options.Providers.Add<GzipCompressionProvider>();        // Fallback
    options.MimeTypes = ResponseCompressionDefaults.MimeTypes.Concat(
        new[] { "image/svg+xml", "application/graphql+json" }); // Add custom types
});

builder.Services.Configure<BrotliCompressionProviderOptions>(options =>
{
    options.Level = CompressionLevel.Fastest;   // CPU-efficient
});

builder.Services.Configure<GzipCompressionProviderOptions>(options =>
{
    options.Level = CompressionLevel.Fastest;
});

var app = builder.Build();

app.UseResponseCompression();   // Early in the pipeline
app.UseStaticFiles();
app.UseRouting();
app.UseAuthentication();
app.UseAuthorization();
app.MapControllers();
```

**Why this follows best practices:**

- **HTTPS compression is disabled** —the secure default is preserved.
- **Brotli is first** —preferred when supported; Gzip is the fallback.
- **MIME types are explicit** —custom types are added; no wildcard assumptions.
- **`Fastest` compression level** —avoids the severe CPU cost of `Optimal` for dynamic responses.
- **Middleware is early** —before static files and endpoints.

### Real-World Cases

- **Public JSON APIs:** Brotli-first with Gzip fallback, `Fastest` level, HTTPS compression disabled.
- **SPA hosting:** `Optimal` Brotli for static assets, `Fastest` for API responses.
- **High-traffic services:** Monitor CPU impact; if compression CPU becomes a bottleneck, lower the compression level or offload to a reverse proxy.

### References

- When to use response compression middleware – https://learn.microsoft.com/th-th/aspnet/core/performance/response-compression?view=aspnetcore-7.0#2
- HTTPS compression security (BREACH/CRIME) – https://learn.microsoft.com/it-it/aspnet/core/performance/response-compression?view=aspnetcore-9.0#2
- `Vary: Accept-Encoding` unconditional emission – https://github.com/dotnet/aspnetcore.docs/blob/47d6f5c46f7a4c552fbf8ca9077439b95150c719/aspnetcore/breaking-changes/11/response-compression-always-vary.md
- MIME types and wildcard limitation – https://learn.microsoft.com/fr-fr/aspnet/core/performance/response-compression?view=aspnetcore-9.0#2

---

## Summary Table: Response Compression at a Glance

| Concept | Primary Role | Key API / Configuration | When to Use |
|---------|--------------|------------------------|-------------|
| **Response compression** | Reduce payload size | `AddResponseCompression()`, `UseResponseCompression()` | Every API and web app serving compressible content |
| **Gzip** | Universal fallback, fast compression | `GzipCompressionProvider`, `CompressionLevel.Fastest` | When Brotli is unsupported or CPU is constrained |
| **Brotli** | Modern default, higher ratio | `BrotliCompressionProvider`, `CompressionLevel.Optimal` | Most traffic; especially static assets |
| **Best practices** | Security, CPU efficiency, cache correctness | `EnableForHttps = false`, MIME filtering, early middleware | Every production deployment |

---

## References (Full List)

- Response compression in ASP.NET Core – https://learn.microsoft.com/en-us/aspnet/core/performance/response-compression
- Response compression (middleware, MIME types, providers) – https://learn.microsoft.com/th-th/aspnet/core/performance/response-compression?view=aspnetcore-9.0#2
- Compression levels – https://learn.microsoft.com/de-at/aspnet/core/performance/response-compression?view=aspnetcore-8.0#3
- `Vary: Accept-Encoding` unconditional emission (.NET 11 breaking change) – https://github.com/dotnet/aspnetcore.docs/blob/47d6f5c46f7a4c552fbf8ca9077439b95150c719/aspnetcore/breaking-changes/11/response-compression-always-vary.md
- HTTPS compression security (BREACH/CRIME) – https://learn.microsoft.com/it-it/aspnet/core/performance/response-compression?view=aspnetcore-9.0#2
- MIME types and wildcard limitation – https://learn.microsoft.com/fr-fr/aspnet/core/performance/response-compression?view=aspnetcore-9.0#2
- Gzip vs Brotli performance comparison – https://www.ioriver.io/gzip-vs-brotli-compression
- Brotli compression (MDN) – https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Content-Encoding
- Enable GZIP/Brotli in ASP.NET Core 8 (ASP.NET Zero support) – https://support.aspnetzero.com/QA/Questions/12431/Enable-GZIP--Brotli-Compression-in-Backend-ASPNET-Core-8
- ASP.NET Core response compression overview (Tencent Cloud) – https://cloud.tencent.com.cn/developer/article/2487064