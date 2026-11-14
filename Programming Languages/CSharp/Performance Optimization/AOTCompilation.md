# AOT Compilation in C# — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
Ahead-of-Time (AOT) compilation is the process of translating C# code into native machine code before the application runs, rather than compiling it dynamically at runtime via the JIT (Just-In-Time) compiler.

**Technical Definition**
In the .NET ecosystem, AOT compilation exists on a spectrum. **ReadyToRun (R2R)** is a partial AOT form: assemblies contain both IL and precompiled native code, reducing JIT work at startup while preserving full runtime compatibility and falling back to JIT for methods that cannot be precompiled. **Native AOT** is a full AOT publishing model: the application, its dependencies, and the runtime components are compiled into a self-contained, RID-specific native executable with no JIT and aggressive trimming (dead-code elimination). Native AOT front-loads compilation to publish time, eliminating runtime JIT overhead but imposing strict constraints on dynamic code generation, unrestricted reflection, and runtime assembly loading. The toolchain involves an ILLink trimmer followed by the ILC (IL Compiler), which translates the trimmed IL into platform-native code.

**Beginner-Friendly Explanation**
AOT compilation is "translating your C# code into machine code before you ship it, instead of translating it while it runs." Normally, .NET compiles your code on the fly (JIT) as the program executes. With AOT, that translation happens during the build/publish step. Native AOT goes all the way: you get a single native executable that starts instantly and does not need the .NET runtime installed on the target machine. ReadyToRun is a lighter version: it precompiles some code to speed up startup but still includes the runtime and can fall back to JIT.

---

### Key Characteristics

- **Spectrum of AOT** — ReadyToRun (partial, compatibility-focused) → Native AOT (full, constraint-heavy).
- **Startup improvement** — Both R2R and Native AOT reduce JIT work at startup; Native AOT eliminates JIT entirely.
- **Trimming dependency** — Native AOT requires aggressive trimming; R2R does not.
- **RID-specific publishing** — Native AOT outputs are pinned to a Runtime Identifier (e.g., `win-x64`, `linux-arm64`); R2R is also RID-specific but retains IL.
- **Dynamic-code restrictions** — Native AOT cannot support `Reflection.Emit`, `Assembly.LoadFrom`, or unrestricted runtime type discovery; R2R supports all dynamic features.
- **Self-contained deployment** — Native AOT produces a single native executable with no separate .NET runtime requirement; R2R still requires the runtime (though it can be self-contained).
- **Diagnostics trade-off** — Native AOT has no managed debugger and no interpreter; R2R retains full debugging and diagnostics.

---

### Prerequisites

- Working knowledge of C#, .NET project structure, and `dotnet publish`.
- Familiarity with the .NET runtime: JIT, IL, and garbage collection.
- Understanding of reflection, serialization, and dependency injection patterns.
- Awareness of trimming concepts and the `PublishTrimmed` property.
- A target RID (Runtime Identifier) for Native AOT publishing.
- The .NET 8 SDK or later (Native AOT support for ASP.NET Core Minimal APIs, gRPC, and Worker Services).

---

### Related Programming Areas

- **Trimming** — Native AOT relies on the trimmer to remove unused code.
- **Source generators** — Replace runtime reflection with build-time code generation, enabling AOT compatibility.
- **Reflection and dynamic code** — The primary incompatibility surface for Native AOT.
- **Deployment and containers** — Native AOT reduces image size and startup time.
- **Performance optimization** — Startup latency and memory footprint reduction.
- **Serialization** — System.Text.Json source generation is required for Native AOT.

---

### Core Concepts / Features

1. Native AOT
2. ReadyToRun
3. AOT Benefits
4. AOT Limitations
5. AOT Best Practices

Each concept follows the same uniform structure: definitions, purposes (each beginning with “To”), syntax rules and structure, annotated code examples with expected outputs, real-world cases, and references.

---

## 1. Native AOT

### Definitions

**Core Definition**
Native AOT is a .NET publishing model that ahead-of-time compiles the entire application — including the runtime — into a self-contained, platform-specific native executable with no JIT compiler.

**Technical Definition**
Native AOT (`PublishAot=true`) invokes a two-stage toolchain: ILLink trims unreferenced code, then the ILC (IL Compiler) translates the remaining IL into native machine code for a specific RID. The output is a single native executable that contains the application code and the necessary runtime components. There is no JIT at runtime, no IL shipping, and no interpreter. Native AOT requires the application to be **trimmable** (all code paths statically analyzable) and **AOT-compatible** (no unrestricted reflection, `Reflection.Emit`, `Assembly.LoadFrom`, or runtime generic instantiation that cannot be resolved at publish time). ASP.NET Core support in .NET 8 is limited to Minimal APIs, gRPC, and Worker Services; MVC, Blazor Server, SignalR, and SPA templates are not supported. Native AOT apps demonstrate lower app size, memory usage, and startup time compared to JIT and trimmed-runtime apps.

**Beginner-Friendly Explanation**
Native AOT is "compile everything into a native `.exe` (or Linux/macOS binary) and ship that." The target machine does not need .NET installed. The app starts immediately because there is no JIT warm-up. But your code must be "AOT-friendly": no fancy runtime reflection, no dynamic code generation, and libraries must support trimming. If you use ASP.NET Core, you are limited to Minimal APIs.

---

### Purposes

- To achieve the fastest possible startup time by eliminating runtime JIT.
- To minimize deployment size with a single native executable and no runtime dependency.
- To reduce memory footprint for high-density containerized deployments.
- To deploy to environments where JIT is not permitted or unavailable.
- To simplify distribution by avoiding runtime provisioning.
- To improve cold-start performance for serverless and short-lived processes.

---

### Syntax Rules and Structure

#### General Syntax

```xml
<PropertyGroup>
  <TargetFramework>net8.0</TargetFramework>
  <PublishAot>true</PublishAot>
  <StripSymbols>true</StripSymbols>
</PropertyGroup>
```

```bash
dotnet publish -c Release -r linux-x64
```

**Component breakdown**

- `PublishAot` — Enables Native AOT compilation during `dotnet publish`.
- `StripSymbols` — Removes debug symbols from the native binary (optional, reduces size).
- `-r linux-x64` — Specifies the target Runtime Identifier; Native AOT is RID-specific.
- The output is a native executable (e.g., `MyApp` on Linux, `MyApp.exe` on Windows).

#### Syntax Rules

- Native AOT is enabled only at publish time; local `dotnet run` still uses JIT.
- The project must target a specific RID; cross-platform binaries are not produced.
- All dependencies must be trim-compatible and AOT-compatible; warnings must be resolved.
- JSON serialization requires source generation (`JsonSerializerContext`); reflection-based serialization is not supported.
- ASP.NET Core apps must use `WebApplication.CreateSlimBuilder()` and Minimal APIs.

#### Constraints and Limitations

- **No dynamic code generation** — `Reflection.Emit`, `DynamicMethod`, `AssemblyBuilder`, and `ILGenerator` are not supported.
- **Restricted reflection** — `Assembly.LoadFrom`, runtime type discovery, and unrestricted reflection are unsupported or unreliable.
- **No managed debugger** — Debugging Native AOT apps requires native debugging tools; the managed debugger is unavailable.
- **No C++/CLI** — Mixed-mode assemblies are not compatible.
- **Limited ASP.NET Core surface** — MVC, Blazor Server, SignalR, and SPA templates are not supported in .NET 8.
- **RID-specific publishing** — You must publish separately for each target platform.
- **Trimming warnings** — IL2xxx and IL3xxx warnings indicate code that may fail at runtime; they must be resolved.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Native AOT Minimal API

```csharp
using System.Text.Json.Serialization;

var builder = WebApplication.CreateSlimBuilder(args);

builder.Services.ConfigureHttpJsonOptions(options =>
{
    options.SerializerOptions.TypeInfoResolverChain.Insert(0, AppJsonSerializerContext.Default);
});

var app = builder.Build();

app.MapGet("/", () => "Hello, Native AOT!");

app.Run();

[JsonSerializable(typeof(string))]
internal partial class AppJsonSerializerContext : JsonSerializerContext { }
```

**Expected Output (publish)**

```
$ dotnet publish -c Release -r linux-x64
...
$ ./bin/Release/net8.0/linux-x64/publish/MyApp
```

**Why this output occurs**

- `CreateSlimBuilder` minimizes default features to reduce size.
- `AppJsonSerializerContext` provides source-generated JSON metadata, required for Native AOT.
- The published output is a single native executable.

---

#### Example 2: Resolving an AOT Warning with Source Generation

```csharp
// ❌ REFLECTION-BASED (AOT warning IL2026)
var json = JsonSerializer.Serialize(myObject);

// ✅ SOURCE-GENERATED (AOT-compatible)
[JsonSerializable(typeof(MyObject))]
internal partial class MyJsonContext : JsonSerializerContext { }

var json = JsonSerializer.Serialize(myObject, MyJsonContext.Default.MyObject);
```

**Expected Output (conceptual)**

```
Reflection-based serialization: IL2026 warning; may fail at runtime under Native AOT.
Source-generated serialization: no warning; works correctly in the published AOT binary.
```

**Why this output occurs**

- Native AOT cannot generate serialization code at runtime via reflection.
- Source generators produce the serialization metadata at build time, making it visible to the AOT compiler.

---

### Real-World Cases

- **Serverless functions** — Native AOT eliminates cold-start JIT overhead.
- **CLI tools** — Single native binary with instant startup and no runtime dependency.
- **Containerized microservices** — Smaller images, faster startup, higher deployment density.
- **Restricted environments** — Edge devices and locked-down systems where JIT is not permitted.

---

### References

- ASP.NET Core support for Native AOT — Microsoft Learn - https://learn.microsoft.com/en-us/aspnet/core/fundamentals/native-aot
- Native AOT deployment overview — Microsoft Learn - https://learn.microsoft.com/en-us/dotnet/core/deploying/native-aot/
- What Is .NET Native AOT? — Zenodo - https://zenodo.org/records/21614491
- Native AOT limitations — Microsoft Learn - https://learn.microsoft.com/en-us/dotnet/core/deploying/native-aot/#limitations-of-native-aot-deployment

---

## 2. ReadyToRun

### Definitions

**Core Definition**
ReadyToRun (R2R) is a form of ahead-of-time compilation that precompiles IL assemblies into native code while retaining the original IL, reducing JIT work at startup without eliminating the runtime JIT.

**Technical Definition**
R2R (`PublishReadyToRun=true`) produces assemblies that contain both IL and precompiled native code. At runtime, the JIT uses the precompiled code for methods that were successfully compiled ahead of time and falls back to JIT for methods that cannot be precompiled (e.g., generic types over value types in other assemblies, interop, or version-specific code). R2R does not require trimming and preserves full dynamic-code compatibility, including reflection, `Reflection.Emit`, and runtime assembly loading. The primary trade-off is **binary size**: R2R assemblies grow to 2–3× their IL-only size, which can negatively impact disk load performance and working set, though the reduced JIT work often improves startup. R2R is RID-specific (e.g., `win-x64`, `linux-x64`) and benefits applications with large code bases more than small ones, since the .NET runtime libraries are already R2R-compiled.

**Beginner-Friendly Explanation**
ReadyToRun is "precompiling some of your code to speed up startup, but keeping the runtime around." Your app still ships IL, and the JIT is still there. But many methods are already compiled to native code, so the first few seconds are faster. Unlike Native AOT, R2R does not restrict reflection or dynamic code, so it is a low-risk first step. The downside is that the output is larger (roughly 2–3×).

---

### Purposes

- To improve startup time and first-use latency without Native AOT's constraints.
- To reduce JIT compilation work during application startup.
- To serve as a low-risk, gradual path toward AOT-style startup improvement.
- To retain full compatibility with reflection, dynamic code, and all .NET libraries.
- To improve cold-start performance for applications with large code bases.

---

### Syntax Rules and Structure

#### General Syntax

```xml
<PropertyGroup>
  <PublishReadyToRun>true</PublishReadyToRun>
</PropertyGroup>
```

```bash
dotnet publish -c Release -r win-x64 -p:PublishReadyToRun=true
```

**Component breakdown**

- `PublishReadyToRun` — Enables R2R compilation during publish.
- `-r win-x64` — Specifies the RID; R2R is RID-specific.
- The output contains assemblies with both IL and native code.
- `PublishReadyToRunExclude` — Excludes specific assemblies from R2R compilation.

#### Syntax Rules

- R2R is enabled at publish time only.
- The target RID must be specified.
- C++/CLI binaries are not eligible for R2R compilation.
- Tiered compilation eventually replaces R2R methods with JIT-optimized versions for hot paths.
- R2R does not require trimming and does not impose AOT constraints.

#### Constraints and Limitations

- Binary size grows to 2–3× IL-only size.
- Not all methods can be precompiled; JIT fallback is expected for generics, interop, and version-specific code.
- Precompiled code is less optimized than JIT-generated code; tiered compilation mitigates this over time.
- R2R alone does not eliminate the runtime dependency or the JIT.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Enabling R2R

```xml
<PropertyGroup>
  <PublishReadyToRun>true</PublishReadyToRun>
</PropertyGroup>
```

```bash
dotnet publish -c Release -r win-x64
```

**Expected Output (conceptual)**

```
The published application includes R2R-compiled assemblies.
Startup time is improved compared to IL-only publishing.
Binary size is roughly 2–3× larger.
```

**Why this output occurs**

- R2R precompiles eligible methods to native code, reducing JIT work at startup.
- IL is retained for methods that cannot be precompiled.

---

#### Example 2: Excluding an Assembly

```xml
<ItemGroup>
  <PublishReadyToRunExclude Include="Contoso.Example.dll" />
</ItemGroup>
```

**Expected Output (conceptual)**

```
Contoso.Example.dll is not R2R-compiled; it remains IL-only.
Other assemblies are R2R-compiled.
```

**Why this output occurs**

- `PublishReadyToRunExclude` removes the specified assembly from R2R processing.

---

### Real-World Cases

- **ASP.NET Core applications** — Reduce first-request latency without AOT constraints.
- **Desktop applications** — Faster startup with full compatibility for UI frameworks.
- **Large code bases** — Significant startup improvement because many methods are precompiled.
- **Migration stepping stone** — Evaluate startup gains before committing to Native AOT.

---

### References

- ReadyToRun deployment overview — Microsoft Learn - https://learn.microsoft.com/en-us/dotnet/core/deploying/ready-to-run
- ReadyToRun compilation — Microsoft Learn - https://learn.microsoft.com/en-us/dotnet/core/deploying/ready-to-run
- What Is .NET Native AOT? (R2R comparison) — Zenodo - https://zenodo.org/records/21614491

---

## 3. AOT Benefits

### Definitions

**Core Definition**
AOT benefits are the performance, deployment, and operational advantages gained by compiling code ahead of time rather than relying solely on runtime JIT compilation.

**Technical Definition**
AOT compilation — particularly Native AOT — shifts compilation work from runtime to publish time. This produces measurable improvements in **startup latency** (no JIT warm-up), **memory footprint** (no JIT compiler in memory, smaller runtime), and **deployment size** (single native executable, no separate runtime). For containerized and serverless workloads, these benefits compound: faster scaling, higher instance density, and reduced cold-start penalties. ReadyToRun offers a subset of these benefits (startup improvement) while retaining full runtime compatibility.

**Beginner-Friendly Explanation**
AOT benefits are "what you gain by compiling early." Your app starts faster, uses less memory, and ships as a smaller package. In the cloud, this means you can fit more instances on the same server and scale up faster.

---

### Purposes

- To reduce startup latency for short-lived and serverless processes.
- To lower memory consumption and increase deployment density.
- To produce a single self-contained executable without runtime dependencies.
- To enable deployment to environments where JIT is prohibited.
- To improve container image size and pull/deploy time.
- To simplify distribution and installation.

---

### Syntax Rules and Structure

#### Benefit Comparison

| Dimension | JIT (Baseline) | ReadyToRun | Native AOT |
|---|---|---|---|
| **Startup speed** | Slowest (JIT warm-up) | Fast | Fastest |
| **App size** | Smallest (IL only) | Larger (2–3× IL) | Smallest (native, trimmed) |
| **Memory footprint** | Higher (JIT + IL) | Similar to JIT | Lower (no JIT) |
| **Runtime dependency** | Required | Required | Not required |
| **Dynamic code support** | Full | Full | None |
| **Best for** | General development | Startup improvement with compatibility | Aggressive startup, size, and restricted environments |

*Sources: Zenodo Native AOT comparison; Microsoft Learn ASP.NET Core Native AOT benefits* 

#### Syntax Rules

- Native AOT benefits are realized only with `dotnet publish` and `PublishAot=true`.
- R2R benefits are realized with `PublishReadyToRun=true`.
- Startup improvements are most visible in short-lived processes (CLI tools, serverless functions, container cold starts).

#### Constraints and Limitations

- Native AOT benefits come at the cost of dynamic-code restrictions.
- R2R size increases may offset startup gains on some storage/network configurations.
- Benefits are workload-dependent; CPU-bound long-running processes may see less improvement.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Startup Time Comparison (Conceptual)

```
JIT:          Startup ~800 ms (JIT warm-up)
ReadyToRun:   Startup ~300 ms (partial precompilation)
Native AOT:   Startup ~50 ms  (no JIT)
```

**Why this output occurs**

- JIT must compile methods on first use.
- R2R precompiles many methods, reducing JIT work.
- Native AOT has no JIT at all.

---

### Real-World Cases

- **AWS Lambda / Azure Functions** — Native AOT dramatically reduces cold-start latency.
- **Kubernetes microservices** — Smaller images and faster pod startup improve scaling responsiveness.
- **CLI tools** — Instant startup and no runtime prerequisite.

---

### References

- ASP.NET Core support for Native AOT (benefits section) — Microsoft Learn - https://learn.microsoft.com/en-us/aspnet/core/fundamentals/native-aot
- What Is .NET Native AOT? (benefits) — Zenodo - https://zenodo.org/records/21614491
- ReadyToRun deployment overview — Microsoft Learn - https://learn.microsoft.com/en-us/dotnet/core/deploying/ready-to-run

---

## 4. AOT Limitations

### Definitions

**Core Definition**
AOT limitations are the features, patterns, and libraries that are incompatible with or restricted by ahead-of-time compilation, particularly Native AOT.

**Technical Definition**
Native AOT requires that all code reachable at runtime be statically determinable at publish time. This makes **dynamic code generation** (`Reflection.Emit`, `DynamicMethod`), **runtime assembly loading** (`Assembly.LoadFrom`), and **unrestricted reflection** (walking types without static annotations) fundamentally incompatible. Libraries that rely on runtime scanning, dynamic proxies, or reflection-based serialization require updates or replacement with source-generated alternatives. Additionally, Native AOT has **no managed debugger** and **no interpreter**, limiting diagnostics. ASP.NET Core feature support is constrained: Minimal APIs, gRPC, and Worker Services are supported; MVC, Blazor Server, SignalR, and SPA templates are not.

**Beginner-Friendly Explanation**
AOT limitations are "the things you cannot do when you compile everything ahead of time." The compiler must know exactly which code will run. If your code says "load this assembly by name at runtime" or "create a new type on the fly," the AOT compiler cannot see it and cannot include it. Similarly, if a library uses reflection to discover types, those types might be trimmed away. Debugging is also harder because the managed debugger is not available.

---

### Purposes

- To understand which features require redesign for AOT compatibility.
- To evaluate whether a library or framework supports Native AOT.
- To plan migrations from JIT/reflection-heavy code to source-generated alternatives.
- To set realistic expectations for diagnostics and debugging.
- To choose between R2R and Native AOT based on feature requirements.

---

### Syntax Rules and Structure

#### Incompatibility Categories

| Category | Examples | Mitigation |
|---|---|---|
| **Dynamic code generation** | `Reflection.Emit`, `DynamicMethod`, `AssemblyBuilder` | Remove or replace with source generators |
| **Runtime assembly loading** | `Assembly.LoadFrom`, `Assembly.LoadFile` | Use static references |
| **Unrestricted reflection** | Runtime type discovery, `Type.GetType(string)` without annotations | Use `[DynamicallyAccessedMembers]` or source generators |
| **Reflection-based serialization** | `JsonSerializer.Serialize(object)` without source context | Use `JsonSerializerContext` source generation |
| **Built-in COM** | COM interop marshalling | Not supported; requires redesign |
| **Managed debugging** | Visual Studio managed debugger | Use native debugging tools |
| **ASP.NET Core features** | MVC, Blazor Server, SignalR, SPA | Use Minimal APIs, gRPC, or Worker Services |

*Sources: Microsoft Learn ASP.NET Core Native AOT compatibility; Zenodo Native AOT limitations; MSBuild AOT annotations guide* 

#### Syntax Rules

- Resolve all IL2xxx (trimming) and IL3xxx (AOT) warnings before publishing.
- Use `[RequiresUnreferencedCode]` and `[RequiresDynamicCode]` to mark incompatible methods and propagate the requirement to callers.
- Prefer `[DynamicallyAccessedMembers]` to express intent rather than suppressing warnings.
- Do not use `#pragma warning disable` for IL warnings; resolve the underlying incompatibility.

#### Constraints and Limitations

- Native AOT cannot support arbitrary runtime code generation; the compiler must see all code at publish time.
- Libraries without AOT support may fail or behave incorrectly.
- Debugging is limited to native tools; no managed stack traces.
- ASP.NET Core feature set is significantly reduced.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Reflection.Emit Incompatibility

```csharp
// ❌ NOT SUPPORTED under Native AOT
var method = new DynamicMethod("MyMethod", typeof(void), Type.EmptyTypes);
var il = method.GetILGenerator();
il.Emit(OpCodes.Ret);
```

**Expected Output (conceptual)**

```
IL3050: Using member 'System.Reflection.Emit.DynamicMethod' which has 'RequiresDynamicCodeAttribute'
may break functionality when AOT compiling.
```

**Why this output occurs**

- `DynamicMethod` generates IL at runtime, which Native AOT cannot support because it compiles ahead of time.

---

#### Example 2: Trim Warning for String-Based Property Access

```csharp
// ❌ Trimmer cannot see the reference
var property = typeof(MyClass).GetProperty("MyProperty");

// ✅ Use a lambda overload to give the compiler a direct reference
builder.Property(x => x.MyProperty);
```

**Expected Output (conceptual)**

```
IL2075: 'this' argument does not satisfy 'DynamicallyAccessedMembersAttribute' in call to 'System.Type.GetProperty'.
```

**Why this output occurs**

- The trimmer cannot determine that `MyProperty` is used because it is referenced only by string.
- The lambda overload creates a direct, visible reference.

---

### Real-World Cases

- **ASP.NET Core MVC applications** — Cannot use Native AOT; must migrate to Minimal APIs or stay on JIT/R2R.
- **Libraries using Newtonsoft.Json** — Reflection-based serialization is incompatible; migrate to System.Text.Json with source generation.
- **Dependency injection containers** — Runtime type scanning may be incompatible; use explicit registration.

---

### References

- ASP.NET Core support for Native AOT (compatibility table) — Microsoft Learn - https://learn.microsoft.com/en-us/aspnet/core/fundamentals/native-aot
- Native AOT limitations — Microsoft Learn - https://learn.microsoft.com/en-us/dotnet/core/deploying/native-aot/#limitations-of-native-aot-deployment
- What Is .NET Native AOT? (limitations) — Zenodo - https://zenodo.org/records/21614491
- Managing Trimming and Native AOT Annotations — dotnet/msbuild - https://github.com/dotnet/msbuild/blob/main/documentation/aot/managing-trimming-and-aot.md

---

## 5. AOT Best Practices

### Definitions

**Core Definition**
AOT best practices are empirically grounded guidelines for designing, configuring, and publishing applications that are compatible with ahead-of-time compilation and produce optimal results.

**Technical Definition**
Best practices span **early planning** (decide AOT compatibility before choosing libraries), **source generation adoption** (JSON serialization, regex, logging), **trimming-aware design** (avoid string-based reflection, use `[DynamicallyAccessedMembers]`), **warning resolution** (treat IL2xxx/IL3xxx as errors, propagate `[RequiresUnreferencedCode]`/`[RequiresDynamicCode]`), **framework selection** (use Minimal APIs, gRPC, or Worker Services for ASP.NET Core; avoid MVC), **publish configuration** (`PublishAot`, `StripSymbols`, RID-specific), and **diagnostics planning** (native debugging tools instead of managed debugger).

**Beginner-Friendly Explanation**
AOT best practices are "how to make your app work well with AOT." They include: choose libraries that support AOT, use source generators instead of reflection, fix all warnings, and test the published AOT binary thoroughly. If you follow these, the AOT output will behave like the JIT version — just faster and smaller.

---

### Purposes

- To ensure the AOT-published application behaves correctly.
- To avoid runtime failures caused by trimmed or unsupported code.
- To maximize the performance and size benefits of Native AOT.
- To choose libraries and frameworks that are AOT-compatible.
- To integrate AOT into the CI/CD pipeline with confidence.
- To minimize the effort of migrating from JIT to AOT.

---

### Guidelines and Structure

#### Decide Early, Choose Compatible Libraries

```
✅ Before adding a dependency, check if it supports trimming and Native AOT.
✅ Prefer libraries that use source generators (System.Text.Json, Microsoft.Extensions.Logging).
❌ Avoid libraries that rely on Reflection.Emit or runtime assembly scanning.
```

- AOT constraints affect library choice; retrofitting is expensive.

#### Adopt Source Generators

```csharp
[JsonSerializable(typeof(MyDto))]
internal partial class MyJsonContext : JsonSerializerContext { }

var json = JsonSerializer.Serialize(dto, MyJsonContext.Default.MyDto);
```

- Replace reflection-based serialization with `JsonSerializerContext`.
- Use source-generated regex (`[GeneratedRegex]`) and logging (`[LoggerMessage]`).

#### Resolve Warnings, Do Not Suppress

```
✅ Treat IL2xxx and IL3xxx warnings as errors.
✅ Use [DynamicallyAccessedMembers] to express intent.
✅ Use [RequiresUnreferencedCode] / [RequiresDynamicCode] on incompatible methods.
❌ Do not use #pragma warning disable for IL warnings.
❌ Suppress only verified false positives with [UnconditionalSuppressMessage].
```

- Warnings indicate code that may fail at runtime; resolution is mandatory.

#### Publish and Test the AOT Binary

```bash
dotnet publish -c Release -r linux-x64 -p:PublishAot=true
# Run the published binary and execute the full test suite
```

- AOT publishing can surface issues not present in JIT builds.
- Test the native binary, not just the JIT build.

#### Configure the Toolchain

```xml
<PropertyGroup>
  <PublishAot>true</PublishAot>
  <StripSymbols>true</StripSymbols>
  <InvariantGlobalization>true</InvariantGlobalization>
</PropertyGroup>
```

- `StripSymbols` reduces binary size.
- `InvariantGlobalization` removes ICU dependency and reduces size when culture-specific behavior is not needed.

#### Use CreateSlimBuilder for ASP.NET Core

```csharp
var builder = WebApplication.CreateSlimBuilder(args);
```

- `CreateSlimBuilder` minimizes default features, reducing deployed size.
- Use Minimal APIs, not MVC controllers.

#### Plan for Diagnostics

- Native AOT has no managed debugger; use `dotnet-dump`, `dotnet-trace` (limited), and native debugging tools.
- Logging and metrics remain available.

#### Prefer ReadyToRun as a First Step

- If full Native AOT is too restrictive, R2R provides startup improvement with full compatibility.
- Use R2R to evaluate startup gains before committing to Native AOT.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: AOT-Compatible JSON Serialization

```csharp
using System.Text.Json.Serialization;

public record Product(int Id, string Name, decimal Price);

[JsonSerializable(typeof(Product))]
[JsonSerializable(typeof(List<Product>))]
internal partial class AppJsonContext : JsonSerializerContext { }

// Usage
var json = JsonSerializer.Serialize(product, AppJsonContext.Default.Product);
var deserialized = JsonSerializer.Deserialize(json, AppJsonContext.Default.Product);
```

**Expected Output (conceptual)**

```
No IL2026/IL3050 warnings.
Serialization works correctly in the Native AOT binary.
```

**Why this output occurs**

- The source generator produces serialization metadata at build time.
- The AOT compiler sees the generated code and includes it.

---

#### Example 2: Resolving a Trim Warning with Annotations

```csharp
// ❌ Warning IL2075
var prop = type.GetProperty("Name");

// ✅ Use DynamicallyAccessedMembers
void AccessProperty([DynamicallyAccessedMembers(DynamicallyAccessedMemberTypes.PublicProperties)] Type type)
{
    var prop = type.GetProperty("Name");
}
```

**Expected Output (conceptual)**

```
The annotation tells the trimmer to preserve public properties of the annotated type.
The warning is resolved.
```

**Why this output occurs**

- `[DynamicallyAccessedMembers]` expresses which members must be preserved, allowing the trimmer to keep them.

---

### Real-World Cases

- **Serverless ASP.NET Core Minimal APIs** — Native AOT with source-generated JSON, `CreateSlimBuilder`, and resolved warnings.
- **CLI tools** — Native AOT with `StripSymbols` and `InvariantGlobalization` for a small, fast binary.
- **Library authors** — Annotate public APIs with `[RequiresUnreferencedCode]` and `[RequiresDynamicCode]` to signal AOT compatibility to consumers.

---

### References

- Managing Trimming and Native AOT Annotations — dotnet/msbuild - https://github.com/dotnet/msbuild/blob/main/documentation/aot/managing-trimming-and-aot.md
- ASP.NET Core support for Native AOT — Microsoft Learn - https://learn.microsoft.com/en-us/aspnet/core/fundamentals/native-aot
- What Is .NET Native AOT? (best practices) — Zenodo - https://zenodo.org/records/21614491
- Native AOT deployment overview — Microsoft Learn - https://learn.microsoft.com/en-us/dotnet/core/deploying/native-aot/

---

## Summary Table: Choosing Between ReadyToRun and Native AOT

| Requirement | ReadyToRun (R2R) | Native AOT |
|---|---|---|
| **Startup improvement** | ✅ Good | ✅ Best |
| **Binary size** | ❌ 2–3× IL | ✅ Smallest |
| **Runtime dependency** | Required | Not required |
| **Reflection / dynamic code** | ✅ Fully supported | ❌ Restricted |
| **Managed debugging** | ✅ Supported | ❌ Not supported |
| **ASP.NET Core MVC** | ✅ Supported | ❌ Not supported |
| **ASP.NET Core Minimal APIs** | ✅ Supported | ✅ Supported |
| **Deployment complexity** | Low | Medium (RID-specific, trimming) |
| **Best for** | First step, broad compatibility | Aggressive startup/size, restricted environments |

---

## Deprecated, Unsafe, Implementation-Dependent, and Version-Specific Notes

| Feature | Status / Note |
|---|---|
| **`Reflection.Emit` / `DynamicMethod`** | Not supported under Native AOT; IL3050 warning |
| **`Assembly.LoadFrom` / `Assembly.LoadFile`** | Not supported under Native AOT; IL2026/IL3050 |
| **Reflection-based `JsonSerializer.Serialize`** | Incompatible; use `JsonSerializerContext` source generation |
| **ASP.NET Core MVC** | Not supported under Native AOT in .NET 8 |
| **Blazor Server / SignalR / SPA** | Not supported under Native AOT |
| **Built-in COM marshalling** | Not supported under Native AOT |
| **Managed debugger** | Unavailable for Native AOT; use native tools |
| **ReadyToRun binary size** | 2–3× IL-only size; may offset startup gains on some storage |
| **`#pragma warning disable` for IL warnings** | Anti-pattern; resolve the underlying incompatibility |
| **Native AOT Android support** | Experimental (XA1040 warning); not for production use |

---

## References (Consolidated)

- ASP.NET Core support for Native AOT — Microsoft Learn - https://learn.microsoft.com/en-us/aspnet/core/fundamentals/native-aot
- Native AOT deployment overview — Microsoft Learn - https://learn.microsoft.com/en-us/dotnet/core/deploying/native-aot/
- Native AOT limitations — Microsoft Learn - https://learn.microsoft.com/en-us/dotnet/core/deploying/native-aot/#limitations-of-native-aot-deployment
- ReadyToRun deployment overview — Microsoft Learn - https://learn.microsoft.com/en-us/dotnet/core/deploying/ready-to-run
- ReadyToRun compilation — Microsoft Learn - https://learn.microsoft.com/en-us/dotnet/core/deploying/ready-to-run
- What Is .NET Native AOT? — How It Differs from JIT and Trimming — Zenodo - https://zenodo.org/records/21614491
- Managing Trimming and Native AOT Annotations — dotnet/msbuild - https://github.com/dotnet/msbuild/blob/main/documentation/aot/managing-trimming-and-aot.md
- Native AOT compatibility (ZA17xx) — dotnet/roslyn-analyzers - https://github.com/dotnet/roslyn-analyzers
- Native AOT and Trimming best practices — Microsoft Learn - https://learn.microsoft.com/en-us/dotnet/core/deploying/trimming/trim-self-contained