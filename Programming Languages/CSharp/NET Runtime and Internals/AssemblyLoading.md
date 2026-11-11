# C# Assembly Loading — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
An **assembly** is the fundamental unit of deployment, versioning, and security in .NET. It is a self-describing package (typically a `.dll` or `.exe`) containing **IL code, metadata, and resources**. **Assembly loading** is the process by which the CLR locates, resolves, and loads an assembly into an application domain (or the current process) so its types can be used. **Assembly resolution/binding** is the mechanism that determines *which* assembly to load when a type references another assembly, taking into account **version**, **culture**, **public key token**, and probing paths.

**Technical Definition**
In .NET, assemblies are the building blocks of applications. They form the fundamental unit of deployment, version control, reuse, activation scoping, and security permissions. An assembly contains a **manifest** (metadata describing the assembly's identity, version, culture, and references), **types** (classes, structs, interfaces, enums), **IL code**, and **resources**. Assemblies can be **private** (deployed with the application) or **shared** (installed in the Global Assembly Cache in .NET Framework). **Assembly loading** is performed by the CLR's **Assembly Loader**, which resolves assemblies by probing specific locations: the application base directory, the global assembly cache (in .NET Framework), the `AppDomain` base, and any configured probing paths. In **.NET Core / .NET 5+**, the **`AssemblyLoadContext`** (ALC) replaces AppDomains for assembly loading isolation. **Strong names** (in .NET Framework) provide unique identity and tamper-proofing. **Global Assembly Cache (GAC)** is a machine-wide repository for shared assemblies (removed in .NET Core / .NET 5+). **Versioning** is expressed as `major.minor.build.revision`.

**Beginner-Friendly Explanation**
An assembly is a **package of compiled code**—a `.dll` or `.exe` that contains IL, metadata, and resources. When your application needs a type, the CLR must **find and load** the assembly that contains it. That process is **assembly loading** (finding and loading the file) and **binding/resolution** (deciding which version to load when multiple versions exist). In .NET Framework, shared assemblies could be installed in the **GAC** (a machine-wide store), and **strong names** guaranteed uniqueness and integrity. In modern .NET (Core/5+), the GAC is gone; assemblies are resolved from the application's directory and NuGet packages, and **`AssemblyLoadContext`** provides isolation for advanced scenarios like plugins.

---

### Key Characteristics

- **Self-describing** — Assemblies contain a manifest with metadata about themselves and their references.
- **Unit of deployment** — Assemblies are the smallest deployable unit in .NET.
- **Unit of versioning** — Assemblies have a four-part version (`major.minor.build.revision`).
- **Unit of security** — Strong names (in .NET Framework) provide identity and integrity.
- **Side-by-side execution** — Multiple versions of an assembly can coexist (GAC in .NET Framework; NuGet + ALCs in .NET Core+).
- **Assembly Loader** — The CLR component that resolves and loads assemblies.
- **`AssemblyLoadContext`** — The .NET Core+ mechanism for assembly loading isolation.
- **Strong names** — (Legacy) unique identity with public key + hash.
- **GAC** — (Legacy) machine-wide assembly store, removed in .NET Core+.

---

### Prerequisites

- Understanding of the **CLR** and managed execution.
- Familiarity with **namespaces**, **types**, and **references**.
- Knowledge of **NuGet** and project references.
- Awareness of **versioning** and **binary compatibility**.
- Basic understanding of **AppDomains** (legacy) and **`AssemblyLoadContext`** (modern).

---

### Related Programming Areas

- **Deployment** — Assemblies are the deployment unit.
- **Versioning** — Strong names and assembly versions.
- **Plugin systems** — `AssemblyLoadContext` for isolation.
- **Dependency resolution** — NuGet, `deps.json`, `AssemblyLoadContext`.
- **Security** — Strong names (legacy), code signing, transparency.

---

### Core Concepts / Features

1. Assemblies
2. Assembly Loading
3. Assembly Resolution
4. Assembly Binding
5. Assembly Versioning
6. Strong Names
7. Global Assembly Cache (GAC)
8. Assembly Best Practices

---

## 1. Assemblies

### Definitions

**Core Definition**
An assembly is the **self-describing unit of deployment, versioning, and security** in .NET, containing IL code, metadata, and resources.

**Technical Definition**
An assembly is a collection of types and resources that are built to work together and form a logical unit of functionality. It is the fundamental unit of deployment, version control, reuse, activation scoping, and security permissions. An assembly contains:
- **Assembly manifest** — Identity (name, version, culture, public key), references to other assemblies, list of files, and security permissions.
- **Type metadata** — Descriptions of types and members.
- **IL code** — Compiled method bodies.
- **Resources** — Embedded files (images, strings, etc.).

Assemblies can be **private** (deployed with the application) or **shared** (installed in the GAC in .NET Framework).

**Beginner-Friendly Explanation**
An assembly is a **package of compiled code**—the `.dll` or `.exe` that contains your IL, metadata, and resources.

---

### Purposes

- To provide a **unit of deployment**.
- To enable **versioning** and **side-by-side execution**.
- To support **security** (strong names, signing).
- To organize code into **reusable libraries**.
- To enable **reflection** and **metadata inspection**.

---

### Syntax Rules and Structure (Assembly Contents)

```
Assembly (DLL/EXE)
├── Assembly Manifest (identity, version, references)
├── Type Metadata (types, members)
├── IL Code (method bodies)
└── Resources (embedded files)
```

**Component breakdown**

- **Manifest** — Identity and references.
- **Metadata** — Types and members.
- **IL** — Compiled code.
- **Resources** — Embedded data.

#### Syntax Rules

- Every type belongs to exactly one assembly.
- The assembly manifest is generated by the compiler.
- References to other assemblies are recorded in the manifest.

#### Constraints and Limitations

- Assemblies are immutable once built.
- Version conflicts require binding redirects (in .NET Framework) or ALCs (in .NET Core+).

---

### Annotated Code Examples and Expected Outputs

#### Example: Inspecting Assembly Metadata

```csharp
using System;
using System.Reflection;

class Program
{
    static void Main()
    {
        Assembly asm = Assembly.GetExecutingAssembly();
        Console.WriteLine($"Name: {asm.GetName().Name}");
        Console.WriteLine($"Version: {asm.GetName().Version}");
        Console.WriteLine($"Location: {asm.Location}");
    }
}
```

**Expected Output (example)**
```
Name: MyApp
Version: 1.0.0.0
Location: C:\path\to\MyApp.dll
```

**Why this output occurs**
`Assembly.GetExecutingAssembly()` returns the current assembly; `GetName()` exposes identity and version.

---

### Real-World Cases

- **Libraries** — Class libraries (`.dll`) are assemblies.
- **Applications** — Executables (`.exe`) are assemblies.
- **NuGet packages** — Packages contain one or more assemblies.

---

### References

- Assemblies in .NET — Microsoft Learn — https://learn.microsoft.com/en-us/dotnet/standard/assembly/

---

## 2. Assembly Loading

### Definitions

**Core Definition**
Assembly loading is the process by which the CLR **locates and loads an assembly** into memory so its types can be used.

**Technical Definition**
Assembly loading is performed by the **Assembly Loader** in the CLR. When a type reference is resolved, the loader looks for the assembly in the following order (simplified):
1. The **application base directory** (where the app's `.dll`/`.exe` resides).
2. The **Global Assembly Cache (GAC)** (in .NET Framework).
3. The **`AppDomain` base** (legacy).
4. Configured **probing paths** and **code bases**.
In **.NET Core / .NET 5+**, the loader uses `deps.json`, NuGet package cache, and `AssemblyLoadContext` for resolution.

**Beginner-Friendly Explanation**
Assembly loading is **"find the DLL and load it."** The CLR looks in the app directory, the GAC (legacy), and configured paths.

---

### Purposes

- To **locate and load** the assemblies your code needs.
- To support **side-by-side execution**.
- To enable **plugin architectures** (via `AssemblyLoadContext`).

---

### Syntax Rules and Structure (Loading Flow)

```
Type reference → Assembly Loader → locate assembly → verify identity → load into memory → JIT compile
```

**Component breakdown**

- **Type reference** — A type used by code.
- **Loader** — Resolves the assembly.
- **Verification** — Checks identity/version.
- **Load** — Loads into memory.

#### Syntax Rules

- Assemblies are loaded **on demand**.
- Loaded assemblies are cached for the process lifetime (unless ALCs are used).
- `Assembly.Load` and `AssemblyLoadContext.LoadFromAssemblyPath` can load explicitly.

#### Constraints and Limitations

- Assembly loading can fail (missing, version mismatch, load errors).
- `.NET Core+` has no GAC; resolution relies on NuGet and app directory.

---

### Annotated Code Examples and Expected Outputs

#### Example: Explicit Assembly Loading

```csharp
using System;
using System.Reflection;

class Program
{
    static void Main()
    {
        Assembly asm = Assembly.LoadFrom("MyLibrary.dll");
        Console.WriteLine($"Loaded: {asm.FullName}");

        Type? type = asm.GetType("MyLibrary.MyClass");
        Console.WriteLine($"Type: {type?.Name}");
    }
}
```

**Expected Output (example)**
```
Loaded: MyLibrary, Version=1.0.0.0, Culture=neutral, PublicKeyToken=null
Type: MyClass
```

**Why this output occurs**
`Assembly.LoadFrom` loads the specified assembly; `GetType` retrieves a type from it.

---

### Real-World Cases

- **Plugins** — Loading plugin assemblies at runtime.
- **Serialization** — Loading assemblies for types dynamically.
- **Dependency injection** — Loading services from assemblies.

---

### References

- Assembly loading — .NET — Microsoft Learn — https://learn.microsoft.com/en-us/dotnet/standard/assembly/loading

---

## 3. Assembly Resolution

### Definitions

**Core Definition**
Assembly resolution is the process of **identifying which assembly (version, culture, public key token) satisfies a reference** and where to find it.

**Technical Definition**
When a type references another assembly, the CLR resolves it by:
1. Reading the reference from the manifest (name, version, culture, public key token).
2. Searching the **application base directory**.
3. Searching the **GAC** (in .NET Framework).
4. Searching **probing paths**.
5. Applying **binding redirects** (in .NET Framework) to resolve version conflicts.
6. In .NET Core+, using **`deps.json`** and **`AssemblyLoadContext`** to find the assembly.

**Beginner-Friendly Explanation**
Assembly resolution is **"figure out which DLL version to use"** when multiple versions are available.

---

### Purposes

- To resolve **version conflicts**.
- To ensure the **correct assembly** is loaded.
- To support **side-by-side execution**.
- To enable **binding redirects** (legacy).

---

### Syntax Rules and Structure (Resolution Steps)

```
1. Manifest reference (name, version, culture, public key token)
2. Application base directory
3. GAC (legacy)
4. Probing paths
5. Binding redirects (legacy)
6. deps.json / ALC (modern)
```

**Component breakdown**

- **Manifest reference** — The identity of the referenced assembly.
- **Base directory** — Where the app resides.
- **GAC** — Machine-wide store (legacy).
- **Probing paths** — Configured directories.

#### Syntax Rules

- Resolution is **automatic** for normal references.
- Binding redirects (in `app.config`) resolve version conflicts (legacy).
- `AssemblyLoadContext` (modern) supports custom resolution.

#### Constraints and Limitations

- Resolution failures cause `FileNotFoundException` or `FileLoadException`.
- Version conflicts require explicit handling.

---

### Annotated Code Examples and Expected Outputs

#### Example: Handling Resolution Errors

```csharp
using System;
using System.Reflection;

class Program
{
    static void Main()
    {
        try
        {
            Assembly asm = Assembly.Load("NonExistentAssembly");
        }
        catch (FileNotFoundException ex)
        {
            Console.WriteLine($"Not found: {ex.FileName}");
        }
    }
}
```

**Expected Output**
```
Not found: NonExistentAssembly
```

**Why this output occurs**
The loader cannot resolve the requested assembly and throws `FileNotFoundException`.

---

### Real-World Cases

- **Version conflicts** — Resolving multiple versions of a library.
- **Plugins** — Custom resolution with `AssemblyLoadContext`.
- **Diagnostics** — Inspecting resolution failures.

---

### References

- Assembly resolution — Microsoft Learn — https://learn.microsoft.com/en-us/dotnet/standard/assembly/

---

## 4. Assembly Binding

### Definitions

**Core Definition**
Assembly binding is the process of **matching a reference to an assembly with an actual assembly** at load time, applying rules like version matching and binding redirects.

**Technical Definition**
Binding is the part of resolution that determines which version of an assembly to use. In .NET Framework, binding uses a strict version policy by default (exact version match) unless a **binding redirect** overrides it. In .NET Core+, binding is more flexible: `deps.json` and ALCs determine which assembly is loaded. **Late binding** uses reflection (`Assembly.Load`, `Type.GetType`, `Activator.CreateInstance`) to load assemblies and create instances at runtime.

**Beginner-Friendly Explanation**
Assembly binding is **"match the reference to the actual DLL."** It decides which version to load and how to handle conflicts.

---

### Purposes

- To **match references to actual assemblies**.
- To resolve **version conflicts**.
- To support **late binding** (reflection).

---

### Syntax Rules and Structure

```
// Early binding (compile-time reference)
var type = typeof(MyClass);

// Late binding (runtime reflection)
Assembly asm = Assembly.Load("MyLibrary");
Type? t = asm.GetType("MyLibrary.MyClass");
object? instance = Activator.CreateInstance(t!);
```

**Component breakdown**

- **Early binding** — Compile-time references.
- **Late binding** — Reflection-based loading.
- **Binding redirects** — (Legacy) version overrides.

#### Syntax Rules

- Early binding is resolved at compile time and JIT.
- Late binding is resolved at runtime via reflection.
- Binding redirects (in `app.config`) apply in .NET Framework.

#### Constraints and Limitations

- Late binding is slower and less type-safe.
- Binding redirects are not used in .NET Core+.

---

### Annotated Code Examples and Expected Outputs

#### Example: Late Binding with Reflection

```csharp
using System;
using System.Reflection;

class Program
{
    static void Main()
    {
        Type? t = Type.GetType("System.String");
        Console.WriteLine(t?.FullName);   // System.String

        object? obj = Activator.CreateInstance(t!, new[] { 'x', 3 });
        Console.WriteLine(obj);   // xxx
    }
}
```

**Expected Output**
```
System.String
xxx
```

**Why this output occurs**
Late binding resolves the type at runtime and creates an instance via `Activator.CreateInstance`.

---

### Real-World Cases

- **Plugin systems** — Loading plugin types at runtime.
- **Serialization** — Creating types from type names.
- **Scripting** — Executing dynamically loaded code.

---

### References

- Assembly binding — Microsoft Learn — https://learn.microsoft.com/en-us/dotnet/standard/assembly/

---

## 5. Assembly Versioning

### Definitions

**Core Definition**
Assembly versioning is the process of assigning a **four-part version number** (`major.minor.build.revision`) to an assembly and using it to control binding and side-by-side execution.

**Technical Definition**
Assemblies have a version number in their manifest. The version is used during binding to decide which assembly to load. In .NET Framework, the default is **strict binding** (exact version match), which can be overridden with binding redirects. In .NET Core+, the default is **roll-forward** (use the latest compatible version). Versioning supports **side-by-side execution** (multiple versions coexist). Strong-named assemblies (legacy) have unique identity via public key token + version.

**Beginner-Friendly Explanation**
Assembly versioning is **"which version of this DLL am I using?"** It controls binding and allows multiple versions to coexist.

---

### Purposes

- To **control which version** of an assembly is loaded.
- To support **side-by-side execution**.
- To enable **roll-forward** (use a newer compatible version).

---

### Syntax Rules and Structure

```xml
<PropertyGroup>
  <AssemblyVersion>1.0.0.0</AssemblyVersion>
  <FileVersion>1.0.0.0</FileVersion>
  <Version>1.0.0</Version>
</PropertyGroup>
```

**Component breakdown**

- `AssemblyVersion` — The binding version.
- `FileVersion` — The file's version (informational).
- `Version` — NuGet package version.

#### Syntax Rules

- Version is `major.minor.build.revision`.
- `.NET Framework`: strict binding by default.
- `.NET Core+`: roll-forward by default.

#### Constraints and Limitations

- Changing `AssemblyVersion` can break binary compatibility.
- `FileVersion` does not affect binding.

---

### Annotated Code Examples and Expected Outputs

#### Example: Setting Assembly Version

```xml
<Project Sdk="Microsoft.NET.Sdk">
  <PropertyGroup>
    <TargetFramework>net9.0</TargetFramework>
    <AssemblyVersion>1.2.3.4</AssemblyVersion>
  </PropertyGroup>
</Project>
```

**Expected Output (conceptual)**
```
// Assembly version 1.2.3.4 is used for binding.
```

**Why this configuration matters**
`AssemblyVersion` controls which version is loaded and referenced.

---

### Real-World Cases

- **Libraries** — Semantic versioning.
- **Plugins** — Version compatibility checks.

---

### References

- Assembly versioning — Microsoft Learn — https://learn.microsoft.com/en-us/dotnet/standard/assembly/versioning

---

## 6. Strong Names

### Definitions

**Core Definition**
A strong name is a **cryptographically unique identity** for an assembly (in .NET Framework), consisting of a name, version, culture, and public key token, plus a digital signature.

**Technical Definition**
Strong names provide:
- **Unique identity** — Name + version + culture + public key token.
- **Integrity** — Digital signature over the assembly contents.
- **Versioning** — Strong-named assemblies have unique identity per version.
Strong names use public-key cryptography: the assembly is signed with a private key, and the public key token is embedded in the assembly's identity. **Strong names are not a security feature**—they ensure uniqueness and integrity, not trust. In .NET Core / .NET 5+, strong names are supported for compatibility, but the GAC is gone and strong names are less central.

**Beginner-Friendly Explanation**
Strong names are **unique identities for assemblies** (in .NET Framework). They ensure no two assemblies collide and that the assembly has not been tampered with.

---

### Purposes

- To provide **unique identity**.
- To ensure **integrity** (digital signature).
- To support **side-by-side execution**.
- To enable **GAC installation** (legacy).

---

### Syntax Rules and Structure

```
Assembly identity = Name + Version + Culture + PublicKeyToken
```

**Component breakdown**

- **Name** — Assembly name.
- **Version** — Four-part version.
- **Culture** — Neutral or specific.
- **PublicKeyToken** — Hash of the public key.

#### Syntax Rules

- Strong names are applied by signing the assembly with a key pair (`.snk` file).
- The public key token is embedded in the identity.
- Strong-named assemblies cannot reference non-strong-named assemblies (in .NET Framework).

#### Constraints and Limitations

- Not a security feature (no trust).
- Deprecated in .NET Core+ (no GAC, no binding redirects).

---

### Annotated Code Examples and Expected Outputs

#### Example: Inspecting Strong Name

```csharp
using System;
using System.Reflection;

class Program
{
    static void Main()
    {
        Assembly asm = typeof(string).Assembly;
        byte[]? token = asm.GetName().GetPublicKeyToken();
        Console.WriteLine(token != null && token.Length > 0 ? "Strong-named" : "Not strong-named");
    }
}
```

**Expected Output**
```
Strong-named
```

**Why this output occurs**
`System.String` is in a strong-named assembly (`System.Private.CoreLib`).

---

### Real-World Cases

- **Framework assemblies** — Strong-named for unique identity.
- **GAC installation** — Requires strong names (legacy).

---

### References

- Strong-named assemblies — Microsoft Learn — https://learn.microsoft.com/en-us/dotnet/standard/assembly/strong-named

---

## 7. Global Assembly Cache (GAC)

### Definitions

**Core Definition**
The Global Assembly Cache (GAC) is a **machine-wide repository** for shared .NET Framework assemblies, allowing multiple versions to coexist and be shared across applications.

**Technical Definition**
The GAC is a special folder (`%windir%\Microsoft.NET\assembly` in .NET Framework 4+) that stores strong-named assemblies shared across applications. Assemblies in the GAC must be strong-named and installed with `gacutil` or an installer. **The GAC was removed in .NET Core / .NET 5+**; modern .NET resolves assemblies from the application directory, NuGet packages, and `AssemblyLoadContext`. The removal of the GAC simplifies deployment but requires apps to bundle their dependencies.

**Beginner-Friendly Explanation**
The GAC is a **machine-wide DLL store** (in .NET Framework). It no longer exists in modern .NET.

---

### Purposes

- To **share assemblies** across applications (legacy).
- To allow **side-by-side versions** (legacy).
- To provide **machine-wide updates** (legacy).

---

### Syntax Rules and Structure

```
gacutil /i MyAssembly.dll   # install (legacy)
gacutil /u MyAssembly       # uninstall (legacy)
```

**Component breakdown**

- **GAC** — Machine-wide store.
- **`gacutil`** — Installation tool.
- **Strong name required** — GAC only accepts strong-named assemblies.

#### Syntax Rules

- GAC is available only in .NET Framework.
- Assemblies must be strong-named.
- Removed in .NET Core / .NET 5+.

#### Constraints and Limitations

- Not available in modern .NET.
- Requires strong names.

---

### Annotated Code Examples and Expected Outputs

#### Example: Checking GAC Presence (Legacy)

```csharp
// .NET Framework only
// The GAC is not available in .NET Core / .NET 5+.
```

**Expected Output (conceptual)**
```
// GAC is not available in .NET Core+; assemblies are resolved from the app directory or NuGet.
```

**Why this matters**
Modern .NET deployments bundle dependencies with the app rather than installing them in a global store.

---

### Real-World Cases

- **.NET Framework applications** — Shared assemblies in the GAC.
- **.NET Core+ applications** — Dependencies bundled via NuGet.

---

### References

- Global Assembly Cache — Microsoft Learn (archive) — https://learn.microsoft.com/en-us/dotnet/framework/app-domains/gac

---

## 8. Assembly Best Practices

### Definitions

**Core Definition**
Assembly best practices are the guidelines for designing, versioning, deploying, and loading assemblies correctly and safely.

**Technical Definition**
Best practices include: **use semantic versioning** (`major.minor.patch` for NuGet packages; `AssemblyVersion` for binding), **avoid breaking changes** in minor versions, **use NuGet for dependency management** in .NET Core+, **prefer `AssemblyLoadContext` for plugins** (rather than `Assembly.LoadFrom` in .NET Core+), **avoid loading assemblies from untrusted paths**, **use strong names only for unique identity** (not security), **keep assembly references minimal**, **test against multiple assembly versions**, and **use `deps.json` and runtime config for deployment**.

**Beginner-Friendly Explanation**
The best practices are: **version carefully**, **use NuGet**, **use `AssemblyLoadContext` for plugins**, and **don't load from untrusted paths**.

---

### Guidelines and Structure

#### General Guidelines

```xml
<!-- 1. Use semantic versioning -->
<Version>1.2.3</Version>
<AssemblyVersion>1.0.0.0</AssemblyVersion>

<!-- 2. Use NuGet for dependencies -->
<PackageReference Include="Newtonsoft.Json" Version="13.0.3" />
```

```csharp
// 3. Use AssemblyLoadContext for plugins
var alc = new AssemblyLoadContext("Plugin", isCollectible: true);
Assembly plugin = alc.LoadFromAssemblyPath("Plugin.dll");

// 4. Avoid loading from untrusted paths
// Validate paths and signatures.

// 5. Keep references minimal
// Avoid unnecessary dependencies.
```

**Component breakdown**

- **Semantic versioning** — Predictable version numbers.
- **NuGet** — Dependency management.
- **`AssemblyLoadContext`** — Plugin isolation.
- **Path validation** — Security.
- **Minimal references** — Reduce coupling.

#### Rules and Constraints

- **Do** use semantic versioning.
- **Do** use NuGet for dependencies.
- **Do** use `AssemblyLoadContext` for plugins.
- **Do** validate assembly paths for security.
- **Do not** load assemblies from untrusted locations.
- **Do not** rely on the GAC in .NET Core+.
- **Do not** assume strong names provide security.

---

### Annotated Code Examples and Expected Outputs

#### Example: Loading a Plugin with `AssemblyLoadContext`

```csharp
using System;
using System.Reflection;
using System.Runtime.Loader;

class PluginLoadContext : AssemblyLoadContext
{
    public PluginLoadContext() : base("Plugin", isCollectible: true) { }

    protected override Assembly? Load(AssemblyName assemblyName) => null;   // default resolution
}

class Program
{
    static void Main()
    {
        var alc = new PluginLoadContext();
        Assembly plugin = alc.LoadFromAssemblyPath("Plugin.dll");
        Console.WriteLine($"Loaded plugin: {plugin.GetName().Name}");
    }
}
```

**Expected Output (example)**
```
Loaded plugin: Plugin
```

**Why this output occurs**
`AssemblyLoadContext` provides isolated loading for plugins; `LoadFromAssemblyPath` loads the assembly.

---

### Real-World Cases

- **Plugin systems** — `AssemblyLoadContext` for isolation and unloadability.
- **Dependency management** — NuGet for versioning.
- **Security** — Validate assembly paths.

---

### References

- Assemblies in .NET — Microsoft Learn — https://learn.microsoft.com/en-us/dotnet/standard/assembly/
- Assembly loading — Microsoft Learn — https://learn.microsoft.com/en-us/dotnet/standard/assembly/loading
- Assembly versioning — Microsoft Learn — https://learn.microsoft.com/en-us/dotnet/standard/assembly/versioning

---

## Deprecated, Unsafe, Implementation-Dependent, and Version-Specific Notes

| Feature | Status / Note |
|---|---|
| AppDomains | **Legacy** (.NET Framework only); replaced by `AssemblyLoadContext` in .NET Core+. |
| GAC | **Removed** in .NET Core / .NET 5+. |
| Strong names | **Supported** in .NET Core+ but not central; no GAC. |
| Binding redirects (`app.config`) | **Legacy** (.NET Framework only); not used in .NET Core+. |
| `Assembly.LoadFrom` | **Discouraged** in .NET Core+; use `AssemblyLoadContext`. |
| `AssemblyLoadContext` | **.NET Core 3.0+**; recommended for plugins. |
| `.deps.json` | Used by .NET Core+ for dependency resolution. |

---

## References (Consolidated)

- Assemblies in .NET — Microsoft Learn — https://learn.microsoft.com/en-us/dotnet/standard/assembly/
- Assembly loading — .NET — Microsoft Learn — https://learn.microsoft.com/en-us/dotnet/standard/assembly/loading
- Assembly versioning — Microsoft Learn — https://learn.microsoft.com/en-us/dotnet/standard/assembly/versioning
- Strong-named assemblies — Microsoft Learn — https://learn.microsoft.com/en-us/dotnet/standard/assembly/strong-named
- Global Assembly Cache — Microsoft Learn (archive) — https://learn.microsoft.com/en-us/dotnet/framework/app-domains/gac
- AssemblyLoadContext — .NET API — https://learn.microsoft.com/en-us/dotnet/api/system.runtime.loader.assemblyloadcontext

---