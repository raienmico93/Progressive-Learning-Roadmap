# 2. Development Environment

A well-configured development environment is the foundation of productive C# development. This guide covers installation of the .NET SDK and Runtime, the major IDEs and editors, the .NET CLI, project file structure, and NuGet package management.

---

## 2.1 .NET SDK Installation

The **.NET SDK** (Software Development Kit) is the essential toolkit for building .NET applications. It includes the compiler, CLI tools, templates, and the runtime.

> "The .NET SDK allows you to develop apps with .NET. If you install the .NET SDK, you don't need to install the corresponding runtime."

### Windows Installation

**Option 1 — WinGet (command line):**

```powershell
winget install Microsoft.DotNet.SDK.10
```

> "Run the winget install command, and specify the name of the SDK package: `winget install Microsoft.DotNet.SDK.10`"

**Option 2 — Windows Installer:**

> "Install .NET with a Windows Installer package, which is an executable that you run. This method can install the SDK and Runtime. Installs are performed system-wide."

**Option 3 — Visual Studio:**

> "If you're developing .NET apps with Visual Studio, Visual Studio manages its own copy of .NET. This method installs the SDK, Runtime, and Visual Studio templates."

### Ubuntu / Debian Installation

```bash
sudo apt-get update && \
sudo apt-get install -y dotnet-sdk-10.0
```

> "To install the .NET SDK, run the following commands: `sudo apt-get update && \ sudo apt-get install -y dotnet-sdk-10.0`"

### macOS Installation

The recommended method is via the official installer from the [.NET downloads page](https://dotnet.microsoft.com/download) or via Homebrew:

```bash
brew install --cask dotnet-sdk
```

### Verifying the Installation

> "Use the `dotnet --list-sdks` and `dotnet --list-runtimes` commands to see which versions are installed."

```bash
dotnet --list-sdks
dotnet --list-runtimes
dotnet --info
```

---

## 2.2 .NET Runtime Installation

The **.NET Runtime** is what executes .NET applications. If you only need to **run** .NET apps — not build them — you can install just the runtime.

> "The runtime runs .NET apps, and the SDK is used to create apps."

### Types of Runtimes

| Runtime | Purpose |
|---|---|
| **.NET Runtime** | Runs console and basic .NET apps |
| **.NET Desktop Runtime** | Includes .NET Runtime + Windows desktop libraries (WPF, WinForms) |
| **ASP.NET Core Runtime** | Runs web-based apps |

> "ASP.NET Core Runtime runs web-based apps, and .NET Desktop Runtime runs desktop apps such as Windows Presentation Foundation (WPF) or Windows Forms apps."

### Installation (Ubuntu)

```bash
sudo apt-get update && \
sudo apt-get install -y aspnetcore-runtime-10.0
```

> "The following commands install the ASP.NET Core Runtime, which is the most compatible runtime for .NET."

For .NET Runtime without ASP.NET Core support:

```bash
sudo apt-get install -y dotnet-runtime-10.0
```

> "As an alternative to the ASP.NET Core Runtime, you can install the .NET Runtime, which doesn't include ASP.NET Core support: replace `aspnetcore-runtime-10.0` with `dotnet-runtime-10.0`."

---

## 2.3 Visual Studio

**Visual Studio** is Microsoft's full-featured IDE and the most powerful C# development environment on Windows.

### Installation

1. Download **Visual Studio Community** (free) from [visualstudio.microsoft.com](https://visualstudio.microsoft.com/)
2. Run the installer and select the **.NET desktop development** workload
3. Click **Install** and wait for completion

> "In the Visual Studio Installer, select the .NET desktop development workload."

### Creating a Project

> "Open Visual Studio and select **Create a new project**. In the **Create a new project** window, select **C#** from the language dropdown list. Select **Windows** from the platform list, and select **Console** from the project type list."

### Key Features

| Feature | Description |
|---|---|
| **IntelliSense** | Advanced code completion |
| **Integrated debugger** | Breakpoints, watches, call stack |
| **Solution Explorer** | Project and file navigation |
| **NuGet Package Manager** | GUI package management |
| **Test Explorer** | Run and debug unit tests |
| **Git integration** | Built-in source control |
| **Live Share** | Real-time collaboration |

---

## 2.4 Visual Studio Code

**Visual Studio Code** is a lightweight, cross-platform code editor that supports C# through extensions.

> "Visual Studio Code is one of the most popular code editors among C# developers. Visual Studio Code is quick and easy to install, and supports numerous extensions to enhance developer productivity."

### Installation

1. Download VS Code from [code.visualstudio.com](https://code.visualstudio.com/)
2. Install the **C# Dev Kit** extension

> "You can add .NET support by installing the C# Dev Kit extension."

> "In the Extensions view, search for 'C# Dev Kit'."

### Creating a Project

> "Initialize a C# project: Open a terminal/command prompt and navigate to the folder in which you'd like to create the app."

```bash
dotnet new console -n MyApp
code MyApp
```

### Key Extensions

| Extension | Purpose |
|---|---|
| **C# Dev Kit** | IntelliSense, debugging, project management |
| **C#** | Base language support (installed with Dev Kit) |
| **.NET Install Tool** | Manages .NET SDK installations |

---

## 2.5 JetBrains Rider

**JetBrains Rider** is a cross-platform .NET IDE known for its powerful code analysis and refactoring tools.

### Key Features

> "Rider 2025.3 brings full compatibility with .NET 10 and comprehensive support for C# 14, including extension members, extension operators, and user-defined compound assignment operators."

| Feature | Description |
|---|---|
| **Code analysis** | Advanced inspections and quick-fixes |
| **Refactoring** | Safe rename, extract, inline, and more |
| **Debugger** | Integrated .NET debugger |
| **Unit testing** | Built-in test runner |
| **Decompiler** | Built-in .NET decompiler |
| **Unity support** | Game development with Unity |
| **Cross-platform** | Windows, macOS, Linux |

> "Rider 2025.3 delivers a smoother and faster startup experience. Several internal initialization phases have been optimized, reducing total solution load time."

### Installation

Download from [jetbrains.com/rider](https://www.jetbrains.com/rider/). Free for students and open-source projects.

---

## 2.6 Command-Line Interface

The **command-line interface** (terminal, PowerShell, or Command Prompt) is essential for working with the .NET CLI, running builds, and managing projects.

### Terminal Options

| Platform | Terminal |
|---|---|
| **Windows** | PowerShell, Command Prompt, Windows Terminal |
| **macOS** | Terminal, iTerm2 |
| **Linux** | Bash, Zsh, Fish |

### Benefits of CLI

- **Automation** — scriptable for CI/CD
- **Speed** — faster than GUI for many tasks
- **Cross-platform** — same commands everywhere
- **Remote work** — works over SSH

---

## 2.7 .NET CLI

The **.NET CLI** is a cross-platform toolchain for creating, building, running, testing, and publishing .NET applications.

> "The .NET Command Line Interface (CLI) is a cross-platform tool that allows developers to create, build, test, run, and publish .NET applications directly from the command line."

### Command Structure

> "All .NET CLI commands follow this pattern: `dotnet [command] [arguments] [options]`"

### 2.7.1 `dotnet new`

Creates a new project, configuration file, or solution.

```bash
dotnet new console -n MyApp
dotnet new webapi -n MyApi
dotnet new classlib -n MyLibrary
dotnet new sln -n MySolution
```

> "Initialize a valid project for either a class library or a console application using C# as a language."

**Common templates:**

| Template | Description |
|---|---|
| `console` | Console application |
| `classlib` | Class library |
| `web` | ASP.NET Core empty web application |
| `webapi` | ASP.NET Core Web API |
| `mvc` | ASP.NET Core MVC application |
| `worker` | Background service |
| `xunit` | xUnit test project |
| `sln` | Solution file |

> "To see all available templates: `dotnet new list`"

### 2.7.2 `dotnet build`

Compiles the project and all dependencies.

```bash
dotnet build
dotnet build -c Release
```

> "Build your code! This command will produce an intermediate Language (IL) binary for your project. By default, the build command will output the built assemblies and executables to the `bin` directory."

### 2.7.3 `dotnet run`

Compiles and runs the application.

```bash
dotnet run
dotnet run --project MyApp.csproj
```

> "The run command will compile and run your application. You can think of this as a Ctrl+F5 analogue, just without Visual Studio."

### 2.7.4 `dotnet test`

Runs the test suite.

```bash
dotnet test
dotnet test --filter "Category=Unit"
```

> "This command allows you to run a suite of tests using a runner that you can specify in the project file. Currently supported are xUnit and NUnit test runners."

### 2.7.5 `dotnet publish`

Publishes the application for deployment.

```bash
dotnet publish -c Release
dotnet publish -c Release -r win-x64 --self-contained
```

> "Publish your application for running on the targeted machine."

### 2.7.6 Other Useful Commands

| Command | Purpose |
|---|---|
| `dotnet restore` | Restore NuGet dependencies |
| `dotnet clean` | Clean build outputs |
| `dotnet pack` | Create a NuGet package |
| `dotnet add package` | Add a NuGet package |
| `dotnet add reference` | Add a project reference |
| `dotnet --info` | Show SDK and runtime information |
| `dotnet --list-sdks` | List installed SDKs |
| `dotnet --list-runtimes` | List installed runtimes |

> "`dotnet --info` … will output SDK version, Runtime version, OS information, RID (Runtime Identifier), and installation path."

---

## 2.8 Project Files

### 2.8.1 `.csproj` — C# Project File

The **`.csproj`** file is an XML document that tells MSBuild how to build the project.

> "The `.csproj` file tells MSBuild how to build the project — which packages it depends on, what version of .NET Core to target, and so forth."

**Minimal SDK-style `.csproj`:**

```xml
<Project Sdk="Microsoft.NET.Sdk">

  <PropertyGroup>
    <OutputType>Exe</OutputType>
    <TargetFramework>net10.0</TargetFramework>
    <ImplicitUsings>enable</ImplicitUsings>
    <Nullable>enable</Nullable>
  </PropertyGroup>

  <ItemGroup>
    <PackageReference Include="Newtonsoft.Json" Version="13.0.3" />
  </ItemGroup>

</Project>
```

**Key elements:**

| Element | Purpose |
|---|---|
| `<TargetFramework>` | The .NET version to target (e.g., `net10.0`) |
| `<OutputType>` | `Exe` for applications, `Library` for libraries |
| `<PackageReference>` | NuGet package dependency |
| `<ProjectReference>` | Reference to another project |
| `<ImplicitUsings>` | Enable implicit global using directives |
| `<Nullable>` | Enable nullable reference types |

> "The `<PackageReference>` element in the middle of the `.csproj` file indicates a NuGet package on which the project depends."

**Key improvement in SDK-style projects:**

> "Source files no longer need to be listed explicitly. Instead, the .NET SDK will automatically compile any `.cs` files next to the project file or in any directories under the `.csproj`'s directory."

### 2.8.2 Solution Files (`.sln`)

A **solution** groups multiple projects together.

> "A project contains all the files that are compiled into an executable, library, or website. A solution is simply a container for one or more related projects."

**Creating a solution:**

```bash
dotnet new sln -n MySolution
dotnet sln add MyApp/MyApp.csproj
dotnet sln add MyLibrary/MyLibrary.csproj
```

**Solution file structure:**

```
MySolution/
├── MySolution.sln
├── MyApp/
│   ├── MyApp.csproj
│   └── Program.cs
├── MyLibrary/
│   ├── MyLibrary.csproj
│   └── Class1.cs
└── MyApp.Tests/
    ├── MyApp.Tests.csproj
    └── UnitTest1.cs
```

### 2.8.3 Modern `.slnx` Format

> "The `.slnx` format is a modern, simplified XML-based solution format that replaces the traditional `.sln` format."

---

## 2.9 NuGet Package Management

**NuGet** is the package manager for .NET. It allows developers to share and consume reusable code packages.

> "NuGet is the package manager for .NET. The NuGet client tools provide the ability to produce and consume packages. The NuGet Gallery is the central package repository used by all package authors and consumers."

### Installation Methods

> "You can install a NuGet package in a Visual Studio project by using the NuGet Package Manager, the Package Manager Console, or the .NET command-line interface (CLI)."

#### Method 1 — .NET CLI (Recommended)

```bash
dotnet add package Newtonsoft.Json
dotnet add package Newtonsoft.Json --version 13.0.3
```

#### Method 2 — Visual Studio NuGet Package Manager

> "Select **Project** > **Manage NuGet Packages**. On the NuGet Package Manager page, next to **Package source**, select **nuget.org**."

#### Method 3 — Package Manager Console

```powershell
Install-Package Newtonsoft.Json
```

#### Method 4 — NuGet CLI (`nuget.exe`)

> "You can use the `nuget.exe` command-line interface (CLI) to manage NuGet packages in Visual Studio projects and solutions."

### Common NuGet Commands

| Command | Purpose |
|---|---|
| `dotnet add package <name>` | Add a package |
| `dotnet remove package <name>` | Remove a package |
| `dotnet list package` | List installed packages |
| `dotnet restore` | Restore all dependencies |
| `dotnet nuget list source` | List package sources |

### Central Package Management

> "To get started with central package management, you can create a `Directory.Packages.props` file at the root of your solution and set the MSBuild property `ManagePackageVersionsCentrally` to true."

**`Directory.Packages.props`:**

```xml
<Project>
  <PropertyGroup>
    <ManagePackageVersionsCentrally>true</ManagePackageVersionsCentrally>
  </PropertyGroup>
  <ItemGroup>
    <PackageVersion Include="Newtonsoft.Json" Version="13.0.3" />
    <PackageVersion Include="xunit" Version="2.9.2" />
  </ItemGroup>
</Project>
```

**`.csproj` with central management:**

```xml
<ItemGroup>
  <PackageReference Include="Newtonsoft.Json" />  <!-- version resolved centrally -->
</ItemGroup>
```

> "No version numbers in individual `.csproj` files."

### Popular NuGet Packages

| Package | Purpose |
|---|---|
| **Newtonsoft.Json** | JSON serialization |
| **Serilog** | Structured logging |
| **AutoMapper** | Object-to-object mapping |
| **FluentValidation** | Fluent validation rules |
| **xUnit / NUnit** | Unit testing |
| **Moq** | Mocking framework |
| **Dapper** | Micro ORM |
| **Entity Framework Core** | Full ORM |
| **MediatR** | Mediator pattern |
| **Polly** | Resilience and transient-fault handling |

---

## Summary Table

| Topic | Key Points |
|---|---|
| **.NET SDK** | Install via `winget`, `apt`, or Visual Studio; includes runtime |
| **.NET Runtime** | Install if you only need to run apps; three types: .NET, Desktop, ASP.NET Core |
| **Visual Studio** | Full IDE; select .NET desktop development workload |
| **VS Code** | Lightweight; install C# Dev Kit extension |
| **Rider** | Cross-platform JetBrains IDE; strong code analysis |
| **CLI** | Terminal-based workflow; essential for automation |
| **`dotnet new`** | Create projects from templates |
| **`dotnet build`** | Compile the project |
| **`dotnet run`** | Build and run the project |
| **`dotnet test`** | Run unit tests |
| **`dotnet publish`** | Publish for deployment |
| **`.csproj`** | XML project file; defines target framework, packages, references |
| **`.sln`** | Solution file grouping multiple projects |
| **NuGet** | Package manager; use `dotnet add package` or VS Package Manager |

---

## Key Takeaways

1. **The .NET SDK includes the runtime** — install it for development; install the runtime alone for running apps.
2. **Visual Studio** is the most powerful C# IDE on Windows; **VS Code** with C# Dev Kit is lightweight and cross-platform; **Rider** is a powerful cross-platform JetBrains IDE.
3. **The .NET CLI** is the cross-platform toolchain: `dotnet new`, `dotnet build`, `dotnet run`, `dotnet test`, `dotnet publish`.
4. **`.csproj`** is the MSBuild project file — it defines the target framework, package references, and project settings.
5. **`.sln`** files group related projects into a solution.
6. **NuGet** is the package manager for .NET — use `dotnet add package` or the Visual Studio GUI.
7. **Central Package Management** via `Directory.Packages.props` keeps versions consistent across projects.
8. **The CLI is the foundation** — IDEs invoke the CLI under the hood, making it essential for automation and CI/CD.

---

Would you like me to continue with the next topic — **C# Basic Syntax**, **C# Data Types**, or **C# Variables and Constants**?