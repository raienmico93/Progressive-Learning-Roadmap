# View Compilation in ASP.NET Core MVC: A Comprehensive Cheat Sheet

> **Version Scope & Compatibility Note**
> This cheat sheet targets **ASP.NET Core MVC** (3.0 and later, including .NET 6/7/8/9/10/11). Since ASP.NET Core 2.1, Razor files (`.cshtml`) are compiled at **both build time and publish time by default** using the **Razor SDK**. **Runtime compilation** is an **opt-in** feature that complements build-time compilation, allowing views to be edited while the app runs. Critically, **runtime compilation was obsoleted starting in .NET 10**; for development, Microsoft now recommends **.NET Hot Reload** instead. Precompiled views are the production standard: they yield smaller published bundles, faster startup, and eliminate file-watcher overhead on every request.

---

## Topic Overview

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | View compilation is the process of translating Razor view files (`.cshtml`) into executable C# classes that the application can render at runtime. |
| **Technical Definition** | The **Razor SDK** compiles `.cshtml` files during `dotnet build` and `dotnet publish`, generating a `Views.dll` (or embedding compiled views into the application assembly). The compiled views derive from `RazorPage<TModel>` and are executed by the MVC view engine. **Runtime compilation** (`Microsoft.AspNetCore.Mvc.Razor.RuntimeCompilation`) is an optional package that recompiles views when their source files change on disk during development, complementing the default build-time compilation. **Precompiled views** are the build-time output deployed to production—no `.cshtml` source files are shipped, and no file watchers run. |
| **Beginner-Friendly Explanation** | Views are `.cshtml` files containing HTML and C#. Before the browser can see them, they must be turned into code the server can run—this is **view compilation**. By default, this happens when you **build or publish** your app, producing a compiled `Views.dll`. **Runtime compilation** is an extra feature that lets you edit a view while the app is running and see the change on refresh, but it’s only for development. In production, **precompiled views** are faster and more secure because the raw `.cshtml` files aren’t even deployed. |

### Key Characteristics

- **Build-time compilation is the default** – The Razor SDK compiles `.cshtml` files during build and publish; no extra package is required.
- **Publish-time compilation is automatic** – The published output contains a compiled `Views.dll` (or embedded views) and no `.cshtml` files by default.
- **Runtime compilation is opt-in** – Enabled by installing `Microsoft.AspNetCore.Mvc.Razor.RuntimeCompilation` and calling `AddRazorRuntimeCompilation()`.
- **Runtime compilation is obsolete in .NET 10** – Use **.NET Hot Reload** for development instead.
- **Precompiled views improve performance** – Smaller published bundle, faster startup, and no disk checks per request.
- **Razor Class Libraries (RCLs)** – Views in RCLs can be compiled into the library and consumed by host apps; runtime compilation for RCLs requires additional file-provider configuration.
- **`MvcRazorCompileOnPublish` is legacy** – Setting this property disables the Razor SDK’s default compilation behaviour; do not use it in modern projects.

### Prerequisites

- **.NET SDK** (6.0+ recommended; examples use .NET 8/10).
- **ASP.NET Core MVC** project with Razor views (`.cshtml`).
- Basic understanding of **Razor syntax**, **views**, and the **build/publish pipeline**.

### Related Programming Areas

- **Razor SDK** – The MSBuild integration that compiles `.cshtml` files.
- **Razor Runtime Compilation** – The opt-in package for editing views at runtime.
- **.NET Hot Reload** – The recommended development-time mechanism for applying code and view changes without restarting.
- **Razor Class Libraries** – Packaged views compiled into a reusable library.
- **Publish Pipeline** – `dotnet publish` produces the production bundle with precompiled views.

### Core Concepts / Features

1. **View Compilation (General)** – The compilation pipeline and its stages.
2. **Runtime Compilation** – Enabling and configuring runtime compilation (with its obsolescence context).
3. **Precompiled Views** – The production standard and its benefits.
4. **View Compilation Best Practices** – Guidelines for development and production.

---

## 1. View Compilation (General)

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | View compilation is the transformation of Razor `.cshtml` files into executable C# classes during build, publish, or (optionally) runtime. |
| **Technical Definition** | The **Razor SDK** (included with `Microsoft.NET.Sdk.Web`) invokes the Razor compiler (`rzc`) as part of `dotnet build` and `dotnet publish`. Each `.cshtml` file is compiled into a class deriving from `RazorPage<TModel>` (or `RazorPage<dynamic>` when no `@model` is present). The compiled views are packaged into a `Views.dll` assembly (by default) or embedded into the main application assembly. At runtime, the MVC view engine resolves views from the compiled assembly, not from `.cshtml` source files. **Runtime compilation** adds an `IViewCompilerProvider` that watches `.cshtml` files on disk and recompiles them on change, allowing live edits during development. |
| **Beginner-Friendly Explanation** | Your `.cshtml` files are like recipes written in a mix of HTML and C#. Before the server can cook (render) them, they need to be translated into something the kitchen (the CLR) can execute. **Build-time compilation** does this translation when you build or publish the app. **Runtime compilation** is like having a chef who can rewrite the recipe on the fly while cooking—useful in development, but not something you’d want in a busy restaurant (production). |

### Purposes (each begins with “To”)

- **To** translate Razor markup into executable C# classes that the MVC view engine can render.
- **To** catch syntax and type errors in views at **build time** rather than at request time.
- **To** produce a deployable artifact (`Views.dll` or embedded views) that contains no editable `.cshtml` source files in production.
- **To** enable faster startup and smaller published bundles through precompilation.
- **To** support development-time iteration (via runtime compilation or Hot Reload) without full rebuilds.

### Syntax Rules and Structure

#### Compilation Pipeline (Default)

```
dotnet build    → Razor SDK compiles .cshtml → Views.dll (or embedded)
dotnet publish  → Razor SDK compiles .cshtml → Views.dll (or embedded)
                  → published output contains NO .cshtml files by default
runtime         → view engine loads compiled views from assembly
```

#### Enabling Runtime Compilation (Opt-In)

```bash
dotnet add package Microsoft.AspNetCore.Mvc.Razor.RuntimeCompilation
```

```csharp
// Program.cs (all environments)
builder.Services.AddControllersWithViews()
    .AddRazorRuntimeCompilation();
```

Or conditionally for Development only:

```csharp
var mvcBuilder = builder.Services.AddControllersWithViews();
if (builder.Environment.IsDevelopment())
{
    mvcBuilder.AddRazorRuntimeCompilation();
}
```

#### Component Breakdown

| Stage | Mechanism | Output |
|-------|-----------|--------|
| **Build** | Razor SDK (`rzc`) | Compiled views in assembly |
| **Publish** | Razor SDK | `Views.dll` (or embedded); no `.cshtml` |
| **Runtime (optional)** | `RuntimeCompilation` package | Recompiles on file change |

#### Syntax Rules

- **Build-time compilation is automatic** – no package or property required in modern SDK-style projects.
- **Runtime compilation requires the NuGet package** and a call to `AddRazorRuntimeCompilation()`.
- **Conditional enabling** is recommended to keep file watchers out of production.
- **Do not set `MvcRazorCompileOnPublish`** – it disables the Razor SDK’s default behaviour.

#### Constraints and Limitations

- **Runtime compilation is obsolete in .NET 10** – `AddRazorRuntimeCompilation()` is marked obsolete; use Hot Reload for development.
- **Runtime compilation disables Hot Reload** – they are mutually exclusive.
- **Runtime compilation does not support global/implicit using directives** or Blazor Razor components.
- **Runtime compilation is not recommended for production** – file watchers and per-request disk checks add overhead.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Default Build-Time Compilation (No Configuration)

```bash
dotnet new mvc -o MvcApp
cd MvcApp
dotnet build
```

**Step-by-step explanation:**

1. The project template uses `Microsoft.NET.Sdk.Web`, which includes the Razor SDK.
2. `dotnet build` compiles all `.cshtml` files into a compiled views assembly.
3. No additional packages or properties are required.

**Expected Output:** The build succeeds; the output directory contains a compiled views assembly (e.g., `MvcApp.Views.dll` or embedded views).

**Why this output?** The Razor SDK compiles views at build time by default in ASP.NET Core 2.1+.

---

#### Example 2: Conditionally Enabling Runtime Compilation (Development Only)

```bash
dotnet add package Microsoft.AspNetCore.Mvc.Razor.RuntimeCompilation
```

```csharp
var builder = WebApplication.CreateBuilder(args);
var mvcBuilder = builder.Services.AddControllersWithViews();

if (builder.Environment.IsDevelopment())
{
    mvcBuilder.AddRazorRuntimeCompilation();
}

var app = builder.Build();
app.UseRouting();
app.MapDefaultControllerRoute();
app.Run();
```

**Step-by-step explanation:**

1. The `RuntimeCompilation` package is installed.
2. `AddRazorRuntimeCompilation()` is called only when the environment is `Development`.
3. In production, the published output uses precompiled views and does **not** enable file watchers.

**Expected Output:** In Development, editing a `.cshtml` file and refreshing the browser shows the change without rebuilding. In Production, views are precompiled and immutable.

**Why this output?** Runtime compilation complements build-time compilation only when explicitly enabled in Development; production uses the compiled views.

---

#### Example 3: Verifying Precompiled Output (No `.cshtml` in Publish)

```bash
dotnet publish -c Release -o ./publish
ls ./publish/*.cshtml   # Should return no results
ls ./publish/*.Views.dll
```

**Expected Output:** No `.cshtml` files exist in the publish folder; a compiled `Views.dll` (or embedded views) is present.

**Why this output?** Publish-time compilation excludes `.cshtml` source files by default; only the compiled views are deployed.

### Real-World Cases

- **Production deployment** – build-time/precompiled views ensure fast startup and no file-watcher overhead.
- **Development iteration** – Hot Reload (preferred) or runtime compilation (legacy, obsolete in .NET 10) allows view edits without restarting.
- **Razor Class Libraries** – views compiled into the library are distributed as a `.Views.dll` for consumers.

### References

- Razor file compilation in ASP.NET Core (official docs) – https://learn.microsoft.com/en-us/aspnet/core/mvc/views/view-compilation?view=aspnetcore-10.0
- Razor file compilation (GitHub source) – https://github.com/dotnet/aspnetcore.docs/blob/47d6f5c46f7a4c552fbf8ca9077439b95150c719/aspnetcore/mvc/views/view-compilation.md
- View precompilation (GitHub) – https://github.com/dotnet/AspNetCore.Docs/blob/main/aspnetcore/mvc/views/view-compilation.md

---

## 2. Runtime Compilation

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Runtime compilation is an **opt-in feature** that recompiles Razor views when their `.cshtml` source files change on disk, allowing live edits during development without rebuilding the application. |
| **Technical Definition** | The `Microsoft.AspNetCore.Mvc.Razor.RuntimeCompilation` package registers `RazorRuntimeCompilationMvcBuilderExtensions.AddRazorRuntimeCompilation()`, which adds an `IViewCompilerProvider` backed by `RoslynCompilationService` and configures a `PhysicalFileProvider` that watches `.cshtml` files. When a view is requested, the runtime compiler checks whether the file has changed; if so, it recompiles the view and caches the new assembly. This **complements** build-time compilation—the app still has compiled views, but edits to `.cshtml` files override them at runtime. **Obsolete starting in .NET 10**; use **.NET Hot Reload** instead. |
| **Beginner-Friendly Explanation** | Runtime compilation lets you **edit a view and refresh the browser** without stopping and rebuilding the app. It’s convenient in development. But it’s slower, disables Hot Reload, and is **not recommended for production**. Starting with .NET 10, Microsoft has marked it **obsolete** and wants you to use **Hot Reload** instead. |

### Purposes (each begins with “To”)

- **To** allow developers to edit `.cshtml` files while the app is running and see changes on refresh.
- **To** complement build-time compilation during development without full rebuilds.
- **To** provide a fallback for legacy development workflows before Hot Reload became the standard.
- **To** enable runtime compilation for Razor Class Library views via additional file providers (advanced scenario).

### Syntax Rules and Structure

#### General Syntax (Enable for All Environments)

```bash
dotnet add package Microsoft.AspNetCore.Mvc.Razor.RuntimeCompilation
```

```csharp
builder.Services.AddControllersWithViews()
    .AddRazorRuntimeCompilation();
```

#### General Syntax (Conditional – Development Only)

```csharp
var mvcBuilder = builder.Services.AddControllersWithViews();
if (builder.Environment.IsDevelopment())
{
    mvcBuilder.AddRazorRuntimeCompilation();
}
```

#### Enabling via `launchSettings.json` (No Code Changes)

```json
{
  "profiles": {
    "MyApp": {
      "environmentVariables": {
        "ASPNETCORE_ENVIRONMENT": "Development",
        "ASPNETCORE_HOSTINGSTARTUPASSEMBLIES": "Microsoft.AspNetCore.Mvc.Razor.RuntimeCompilation"
      }
    }
  }
}
```

#### Component Breakdown

| Setup Method | Scope | When to Use |
|--------------|-------|-------------|
| `AddRazorRuntimeCompilation()` (unconditional) | All environments | Rare; not recommended for production. |
| Conditional in `Program.cs` | Development only | Recommended if runtime compilation is used. |
| `launchSettings.json` + HostingStartup | Per launch profile | No code changes; Development only. |

#### Syntax Rules

- **Package required** – `Microsoft.AspNetCore.Mvc.Razor.RuntimeCompilation`.
- **`AddRazorRuntimeCompilation()` is obsolete in .NET 10** – will produce a warning/error.
- **Use `MvcRazorRuntimeCompilationOptions.FileProviders`** to include RCL views.
- **Do not combine with Hot Reload** – runtime compilation disables Hot Reload.

#### Constraints and Limitations

- **Obsolete in .NET 10** – `AddRazorRuntimeCompilation()` is marked obsolete; use Hot Reload.
- **Disables Hot Reload** – mutually exclusive.
- **Not for production** – file watchers and disk checks add overhead.
- **Does not support global/implicit using directives** – compilation fails for views relying on them.
- **Not supported for Blazor Razor components**.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Enabling Runtime Compilation for Development Only

```csharp
var builder = WebApplication.CreateBuilder(args);
var mvcBuilder = builder.Services.AddControllersWithViews();

if (builder.Environment.IsDevelopment())
{
    mvcBuilder.AddRazorRuntimeCompilation();
}

var app = builder.Build();
app.UseRouting();
app.MapDefaultControllerRoute();
app.Run();
```

**Step-by-step explanation:**

1. The package is installed.
2. `AddRazorRuntimeCompilation()` is called only when `IsDevelopment()` is `true`.
3. In Development, editing a `.cshtml` file triggers recompilation on the next request.
4. In Production, the published output uses compiled views; no file watchers are enabled.

**Expected Output:** Development edits appear on refresh; production uses precompiled views.

**Why this output?** Conditional enabling ensures production keeps the default build-time compilation behaviour.

---

#### Example 2: Enabling via `launchSettings.json` (No Code Changes)

```json
{
  "profiles": {
    "MyApp": {
      "commandName": "Project",
      "environmentVariables": {
        "ASPNETCORE_ENVIRONMENT": "Development",
        "ASPNETCORE_HOSTINGSTARTUPASSEMBLIES": "Microsoft.AspNetCore.Mvc.Razor.RuntimeCompilation"
      }
    }
  }
}
```

**Expected Output:** Runtime compilation is enabled only for the `MyApp` launch profile; `Program.cs` remains unchanged.

**Why this output?** ASP.NET Core searches for a `HostingStartup` attribute in the specified assembly and executes its startup code to enable runtime compilation.

---

#### Example 3: Enabling Runtime Compilation for a Razor Class Library

```csharp
services.AddControllersWithViews()
    .AddRazorRuntimeCompilation(options =>
    {
        var libraryPath = Path.GetFullPath(
            Path.Combine(HostEnvironment.ContentRootPath, "..", "MyClassLib"));
        options.FileProviders.Add(new PhysicalFileProvider(libraryPath));
    });
```

**Expected Output:** Runtime compilation watches `.cshtml` files in the `MyClassLib` RCL.

**Why this output?** The `PhysicalFileProvider` makes the RCL’s files visible to the runtime compiler.

### Real-World Cases

- **Legacy development workflows** – teams that have not yet adopted Hot Reload.
- **Razor Class Library development** – editing shared views in an RCL and seeing changes immediately.
- **Migration awareness** – projects targeting .NET 10 must migrate away from `AddRazorRuntimeCompilation()`.

### References

- Enable runtime compilation for all environments – https://learn.microsoft.com/en-us/aspnet/core/mvc/views/view-compilation?view=aspnetcore-10.0#enable-runtime-compilation-for-all-environments
- Enable runtime compilation conditionally – https://learn.microsoft.com/en-us/aspnet/core/mvc/views/view-compilation?view=aspnetcore-10.0#enable-runtime-compilation-conditionally
- Runtime compilation via HostingStartup – https://learn.microsoft.com/en-us/aspnet/core/mvc/views/view-compilation?view=aspnetcore-10.0#enable-runtime-compilation-in-the-development-environment-for-specific-launch-profiles
- Enable runtime compilation for a Razor class library – https://learn.microsoft.com/en-us/aspnet/core/mvc/views/view-compilation?view=aspnetcore-10.0#enable-runtime-compilation-for-a-razor-class-library
- Razor runtime compilation is obsolete (.NET 10 breaking change) – https://learn.microsoft.com/en-us/dotnet/core/compatibility/aspnet-core/10/razor-runtime-compilation-obsolete

---

## 3. Precompiled Views

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Precompiled views are Razor `.cshtml` files that have been compiled into an assembly during **build or publish**, so the deployed application contains no editable `.cshtml` source files and renders views directly from compiled code. |
| **Technical Definition** | With precompiled views, the Razor SDK produces a `Views.dll` assembly (or embeds compiled views into the main assembly). At publish time, `.cshtml` files are **excluded** from the output by default. At runtime, the view engine resolves views from the compiled assembly via `CompiledViewProvider`. Because no `.cshtml` files exist in the deployed bundle, there are **no file watchers, no per-request disk checks, and no runtime compilation**. This results in a **smaller published bundle**, **faster startup**, and **deterministic behaviour**. |
| **Beginner-Friendly Explanation** | Precompiled views are **views that have already been translated into code before deployment**. It’s like baking a cake before the party instead of mixing ingredients on demand. The published app doesn’t ship the recipe (`.cshtml` files)—only the finished cake (compiled assembly). This makes the app start faster, use less disk space, and behave consistently in production. |

### Purposes (each begins with “To”)

- **To** produce a **smaller published bundle** by excluding `.cshtml` source files.
- **To** achieve **faster application startup** by eliminating runtime compilation and file-watching overhead.
- **To** ensure **deterministic rendering** – views cannot be edited in production, so behaviour is fixed at deployment.
- **To** improve **security** by not deploying editable source files that could be tampered with.
- **To** catch view compilation errors at **build/publish time** rather than at request time.

### Syntax Rules and Structure

#### Default Behaviour (No Configuration)

```
dotnet publish → precompiled views (Views.dll or embedded) → no .cshtml in output
```

#### Component Breakdown

| Aspect | Precompiled Views | Runtime Compilation |
|--------|-------------------|---------------------|
| Output | `Views.dll` (or embedded) | `.cshtml` files + runtime compiler |
| Startup speed | Faster | Slower |
| Bundle size | Smaller | Larger (includes `.cshtml`) |
| Editable in production | No | Yes (if enabled) |
| File watchers | None | Yes |
| Recommended for production | **Yes** | **No** |

#### Syntax Rules

- **Precompilation is the default** – no action required in modern SDK-style projects.
- **Do not set `MvcRazorCompileOnPublish`** – it disables the Razor SDK’s default precompilation.
- **`RazorCompileOnBuild` and `RazorCompileOnPublish`** MSBuild properties exist for fine-grained control, but defaults are recommended.
- **RCL views** are compiled into the RCL’s assembly and consumed by the host app.

#### Constraints and Limitations

- **Cannot edit views after publishing** – the `.cshtml` files are absent from the bundle.
- **Build-time errors surface during build/publish** – this is desirable but requires fixing compilation issues before deployment.
- **Some legacy properties** (`MvcRazorCompileOnPublish`) interfere with the Razor SDK; avoid them.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Default Precompilation with `dotnet publish`

```bash
dotnet publish -c Release -o ./publish
find ./publish -name "*.cshtml"   # Should return nothing
find ./publish -name "*.Views.dll" # Should find the compiled views
```

**Expected Output:** No `.cshtml` files; a compiled `Views.dll` (or embedded views) is present.

**Why this output?** The Razor SDK precompiles views at publish time and excludes `.cshtml` files by default.

---

#### Example 2: Inspecting the Published Bundle

```bash
ls -la ./publish
# Typical output includes:
# MyApp.dll
# MyApp.Views.dll    ← compiled views
# appsettings.json
# wwwroot/
# (no Views/ folder with .cshtml files)
```

**Expected Output:** The published bundle contains compiled assemblies and static assets, but no `Views/` folder with editable `.cshtml` files.

**Why this output?** Precompiled views are packaged into `Views.dll`; source `.cshtml` files are not deployed.

---

#### Example 3: Avoiding Legacy Properties That Disable Precompilation

```xml
<!-- ❌ Do NOT set this in modern projects -->
<PropertyGroup>
  <MvcRazorCompileOnPublish>true</MvcRazorCompileOnPublish>
</PropertyGroup>
```

**Expected Output:** Setting `MvcRazorCompileOnPublish` disables the Razor SDK’s default precompilation behaviour, potentially leaving `.cshtml` files in the output or causing unexpected behaviour.

**Why this output?** The Razor SDK is effective only when no precompilation-specific properties are set; legacy properties override the default pipeline.

### Real-World Cases

- **Production deployments** – precompiled views are the standard for performance and security.
- **Docker containers** – smaller images without `.cshtml` files.
- **CI/CD pipelines** – build-time view compilation catches errors before deployment.
- **Razor Class Libraries** – views compiled into the library assembly for distribution.

### References

- Precompilation considerations – https://github.com/dotnet/AspNetCore.Docs/blob/f3c89a1e54e05764d1d854a3b5c1038c28d8f0a1/aspnetcore/mvc/views/view-compilation.md#1
- Deploy precompiled files – https://github.com/dotnet/AspNetCore.Docs/blob/f3c89a1e54e05764d1d854a3b5c1038c28d8f0a1/aspnetcore/mvc/views/view-compilation.md#1
- Razor SDK and `MvcRazorCompileOnPublish` – https://github.com/dotnet/AspNetCore.Docs/blob/f3c89a1e54e05764d1d854a3b5c1038c28d8f0a1/aspnetcore/mvc/views/view-compilation.md#1

---

## 4. View Compilation Best Practices

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | View compilation best practices are guidelines for choosing and configuring the right compilation mode (build-time, runtime, or precompiled) for each environment, ensuring performance, security, and developer productivity. |
| **Technical Definition** | Best practices include: relying on the **Razor SDK’s default build-time compilation** for production; using **.NET Hot Reload** for development iteration (replacing runtime compilation, which is obsolete in .NET 10); **conditionally enabling** runtime compilation only if a legacy workflow requires it; **never deploying `.cshtml` files** to production (precompile at publish); avoiding legacy MSBuild properties (`MvcRazorCompileOnPublish`) that interfere with the Razor SDK; and configuring **RCL file providers** only when runtime compilation of library views is explicitly needed. |
| **Beginner-Friendly Explanation** | Best practices are the **rules of thumb** for view compilation. The most important: **precompile for production** (smaller, faster, safer) and **use Hot Reload for development** (instant feedback without the baggage of runtime compilation). If you must use runtime compilation, enable it **only in Development**. |

### Purposes (each begins with “To”)

- **To** maximise **production performance** through precompiled views (smaller bundle, faster startup).
- **To** ensure **security** by not deploying editable `.cshtml` source files.
- **To** provide a **fast, reliable development experience** via Hot Reload, which is the modern replacement for runtime compilation.
- **To** avoid **obsolete APIs** (`AddRazorRuntimeCompilation`) in projects targeting .NET 10+.
- **To** prevent **configuration mistakes** (e.g., setting `MvcRazorCompileOnPublish`) that disable the Razor SDK’s defaults.
- **To** keep **Razor Class Library** views compilation and distribution consistent.

### Guideline Checklist

| Guideline | Rationale | Example |
|-----------|-----------|---------|
| **Use default build-time compilation for production** | Faster startup, smaller bundle, no file watchers. | No action required; `dotnet publish` precompiles by default. |
| **Use .NET Hot Reload for development** | Modern, supported replacement for runtime compilation. | `dotnet watch` or IDE Hot Reload. |
| **Do not use `AddRazorRuntimeCompilation()` in .NET 10+** | Obsolete; produces warnings/errors. | Migrate to Hot Reload. |
| **If runtime compilation is required, enable it conditionally** | Keeps production precompiled and watcher-free. | `if (env.IsDevelopment()) mvcBuilder.AddRazorRuntimeCompilation();` |
| **Do not set `MvcRazorCompileOnPublish`** | Disables Razor SDK defaults. | Remove the property from `.csproj`. |
| **Precompile RCL views into the library** | Avoids runtime compilation dependencies. | Use `<Project Sdk="Microsoft.NET.Sdk.Razor">`. |
| **Verify published output contains no `.cshtml`** | Confirms precompilation is working. | `find ./publish -name "*.cshtml"` → empty. |
| **Keep views free of runtime-only features** | Runtime compilation lacks global/implicit usings. | Use explicit `@using` directives. |

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Production Configuration (Default Precompilation)

```xml
<!-- MyApp.csproj – default SDK-style project -->
<Project Sdk="Microsoft.NET.Sdk.Web">
  <PropertyGroup>
    <TargetFramework>net8.0</TargetFramework>
  </PropertyGroup>
</Project>
```

```bash
dotnet publish -c Release
```

**Step-by-step explanation:**

1. No runtime compilation package is referenced.
2. The Razor SDK compiles `.cshtml` files at build and publish.
3. The published output contains compiled views and no `.cshtml` files.

**Expected Output:** Fast startup, small bundle, no file watchers.

**Why this output?** Default build-time/publish-time compilation is enabled by the Razor SDK.

---

#### Example 2: Development with Hot Reload (Recommended Over Runtime Compilation)

```bash
dotnet watch
```

**Expected Output:** The app starts with Hot Reload; editing a `.cshtml` file applies the change without restarting.

**Why this output?** Hot Reload supports Razor view edits during development and is the recommended approach.

---

#### Example 3: Conditional Runtime Compilation for Legacy Workflows

```csharp
var mvcBuilder = builder.Services.AddControllersWithViews();
if (builder.Environment.IsDevelopment())
{
    mvcBuilder.AddRazorRuntimeCompilation();   // Only if legacy workflow requires it
}
```

**Expected Output:** Runtime compilation is active only in Development; production uses precompiled views.

**Why this output?** Conditional enabling keeps production free of file watchers while supporting legacy development needs.

---

#### Example 4: Verifying Precompilation in CI

```bash
dotnet publish -c Release -o ./out
if find ./out -name "*.cshtml" | grep -q .; then
  echo "ERROR: .cshtml files found in publish output"
  exit 1
fi
echo "Precompilation verified"
```

**Expected Output:** The script succeeds only if no `.cshtml` files are present in the publish output.

**Why this output?** Precompiled views exclude `.cshtml` files by default; the check guards against misconfiguration.

### Real-World Cases

- **CI/CD pipelines** – verify that no `.cshtml` files are published; fail the build if they are.
- **Docker images** – smaller images without editable view sources.
- **Legacy migration** – replacing `AddRazorRuntimeCompilation()` with Hot Reload when upgrading to .NET 10+.
- **Razor Class Libraries** – precompiling views into the library for distribution.

### References

- Razor file compilation in ASP.NET Core – https://learn.microsoft.com/en-us/aspnet/core/mvc/views/view-compilation?view=aspnetcore-10.0
- Razor runtime compilation is obsolete – https://learn.microsoft.com/en-us/dotnet/core/compatibility/aspnet-core/10/razor-runtime-compilation-obsolete
- View precompilation (GitHub) – https://github.com/dotnet/AspNetCore.Docs/blob/main/aspnetcore/mvc/views/view-compilation.md
- Precompilation considerations – https://github.com/dotnet/AspNetCore.Docs/blob/f3c89a1e54e05764d1d854a3b5c1038c28d8f0a1/aspnetcore/mvc/views/view-compilation.md#1

---

## Summary Table: View Compilation at a Glance

| Concept | Primary Purpose | Key Takeaway | When to Use |
|---------|----------------|--------------|-------------|
| **View Compilation** | Translate `.cshtml` to executable C# | Build-time compilation is the default; runtime compilation is opt-in. | Every ASP.NET Core MVC app |
| **Runtime Compilation** | Live-edit views during development | **Obsolete in .NET 10**; use Hot Reload instead. | Legacy development only; conditional in Development |
| **Precompiled Views** | Production-ready compiled views | Smaller bundle, faster startup, no `.cshtml` in output. | **Always in production** |
| **Best Practices** | Correct mode for each environment | Precompile for production; Hot Reload for development; avoid obsolete APIs. | All projects |

---

## References (Full List)

- Razor file compilation in ASP.NET Core (official docs, .NET 10) – https://learn.microsoft.com/en-us/aspnet/core/mvc/views/view-compilation?view=aspnetcore-10.0
- Razor file compilation in ASP.NET Core (GitHub source) – https://github.com/dotnet/aspnetcore.docs/blob/47d6f5c46f7a4c552fbf8ca9077439b95150c719/aspnetcore/mvc/views/view-compilation.md
- View precompilation (GitHub) – https://github.com/dotnet/AspNetCore.Docs/blob/main/aspnetcore/mvc/views/view-compilation.md
- Precompilation considerations – https://github.com/dotnet/AspNetCore.Docs/blob/f3c89a1e54e05764d1d854a3b5c1038c28d8f0a1/aspnetcore/mvc/views/view-compilation.md#1
- Razor runtime compilation is obsolete (.NET 10 breaking change) – https://learn.microsoft.com/en-us/dotnet/core/compatibility/aspnet-core/10/razor-runtime-compilation-obsolete
- Breaking changes in ASP.NET Core 10 (Razor runtime compilation obsolete) – https://github.com/dotnet/aspnetcore.docs/blob/main/aspnetcore/breaking-changes/10/razor-runtime-compilation-obsolete.md
- Enable runtime compilation for a Razor class library – https://learn.microsoft.com/en-us/aspnet/core/mvc/views/view-compilation?view=aspnetcore-10.0#enable-runtime-compilation-for-a-razor-class-library
- Enable runtime compilation conditionally – https://learn.microsoft.com/en-us/aspnet/core/mvc/views/view-compilation?view=aspnetcore-10.0#enable-runtime-compilation-conditionally
- Enable runtime compilation via HostingStartup – https://learn.microsoft.com/en-us/aspnet/core/mvc/views/view-compilation?view=aspnetcore-10.0#enable-runtime-compilation-in-the-development-environment-for-specific-launch-profiles
- Migrate apps to ASP.NET Core in Optimizely CMS (precompiled views distribution) – https://docs.developers.optimizely.com