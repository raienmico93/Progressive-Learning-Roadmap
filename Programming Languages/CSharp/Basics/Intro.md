# 1. Introduction to C#

C# (pronounced "C Sharp") is Microsoft's flagship programming language for the .NET platform. It is a modern, object-oriented, multi-paradigm language that combines the power of C and C++ with the productivity of higher-level languages. Understanding what C# is, where it came from, and how it fits into the .NET ecosystem provides the essential foundation for learning the language.

---

## 1.1 C# Definition and Purpose

### 1.1.1 Simple, Modern, General-Purpose, Object-Oriented Language

C# is intended to be a **simple, modern, general-purpose, object-oriented programming language**. The language was designed with the following goals:

| Design Goal | Description |
|---|---|
| **Simplicity** | Easy to learn, clean syntax |
| **Modernity** | Incorporates contemporary language features |
| **General-purpose** | Applicable to a wide range of domains |
| **Object-oriented** | Based on object-oriented principles |
| **Type safety** | Strong type checking, array bounds checking |
| **Automatic garbage collection** | Memory management handled by the runtime |
| **Distributed environments** | Suitable for building distributed software components |
| **Portability** | Source code and programmer portability, especially for C/C++ programmers |
| **Internationalization** | Support for international applications |

> "C# is intended to be a simple, modern, general-purpose, object-oriented programming language. The language, and implementations thereof, should provide support for software engineering principles such as strong type checking, array bounds checking, detection of attempts to use uninitialized variables, and automatic garbage collection."

### 1.1.2 Cross-Platform General-Purpose Language

C# is a **cross-platform general-purpose language** that makes developers productive while writing highly performant code. It is the **most popular language for the .NET platform**, a free, cross-platform, open-source development environment.

C# programs can run on many different devices, from **Internet of Things (IoT) devices to the cloud** and everywhere in between. You can write apps for **phone, desktop, laptop computers, and servers**.

### 1.1.3 Multi-Paradigm Language

Based on object-oriented principles, C# incorporates **many features from other paradigms, not least functional programming**. Low-level features support high-efficiency scenarios without writing unsafe code.

### 1.1.4 C Family of Languages

C# is in the **C family of languages**. Its syntax is familiar if you have used **C, C++, JavaScript, TypeScript, or Java**.

| Feature | C# Behavior |
|---|---|
| **Statement termination** | Semicolons (`;`) define the end of statements |
| **Identifiers** | Case-sensitive |
| **Braces** | `{` and `}` for blocks |
| **Control statements** | `if`, `else`, `switch` |
| **Loops** | `for`, `while`, `foreach` |

---

## 1.2 History and Evolution of C#

### 1.2.1 Origins

C# was developed within **Microsoft** by **Anders Hejlsberg**, **Scott Wiltamuth**, and **Peter Golde**. The first widely distributed implementation was released by Microsoft in **July 2000**, as part of its **.NET Framework** initiative.

> "The principal inventors of this language were Anders Hejlsberg, Scott Wiltamuth, and Peter Golde. The first widely distributed implementation of C# was released by Microsoft in July 2000, as part of its .NET Framework initiative."

### 1.2.2 Standardization

**Ecma Technical Committee 39 (TC39)** — later renamed to **TC49** — **Task Group 2 (TG2)** was formed in **September 2000** to produce a standard for C#. Another task group, **TG3**, was formed to produce a standard for a library and execution environment called **Common Language Infrastructure (CLI)**.

Ecma International ratified the first edition of the C# specification as **ECMA-334** in **December 2001**. The ECMA-334 standard has since reached its **seventh edition, published in December 2023**.

| Standard | Edition | Year |
|---|---|---|
| **ECMA-334** | 1st Edition | December 2001 |
| **ECMA-334** | 2nd Edition | December 2002 |
| **ECMA-334** | 7th Edition | December 2023 |
| **ISO/IEC 20619** | — | 2023 |

### 1.2.3 Version Evolution

C# has evolved rapidly, with new versions released alongside .NET platform updates.

| C# Version | Release | Key Features |
|---|---|---|
| **C# 1.0** | January 2002 | Initial release; classes, structs, interfaces, delegates, events |
| **C# 2.0** | November 2005 | Generics, nullable types, iterators, partial classes |
| **C# 3.0** | November 2007 | LINQ, lambda expressions, extension methods, anonymous types, `var` |
| **C# 4.0** | April 2010 | Dynamic binding, named/optional arguments, covariance/contravariance |
| **C# 5.0** | August 2012 | `async`/`await`, caller info attributes |
| **C# 6.0** | July 2015 | Expression-bodied members, string interpolation, null-conditional operators |
| **C# 7.0** | March 2017 | Tuples, pattern matching, `out` variables, local functions |
| **C# 8.0** | September 2019 | Nullable reference types, async streams, ranges, switch expressions |
| **C# 9.0** | November 2020 | Records, init-only setters, top-level statements, pattern matching enhancements |
| **C# 10** | November 2021 | Global usings, file-scoped namespaces, record structs |
| **C# 11** | November 2022 | Raw string literals, required members, generic math |
| **C# 12** | November 2023 | Primary constructors, collection expressions, alias any type |
| **C# 13** | November 2024 | `params` collections, new `lock` semantics, `\e` escape sequence |
| **C# 14** | November 2025 | Latest stable version |

> "As of November 2025, the most recent stable version of the language is C# 14."

---

## 1.3 Relationship Among C#, .NET, CLR, .NET SDK, and .NET Runtime

Understanding the relationship between these components is essential.

### 1.3.1 C# and .NET

C# is a **language** that runs on the **.NET platform**. The .NET platform provides the runtime, libraries, and tools that C# programs depend on.

> "The C# language is the most popular language for the .NET platform, a free, cross-platform, open source development environment."

### 1.3.2 .NET Platform Components

.NET is made up of **the runtime and the SDK**. The **runtime runs .NET apps**, and the **SDK is used to create apps**.

| Component | Purpose |
|---|---|
| **.NET Runtime** | Runs .NET applications |
| **.NET SDK** | Builds and develops .NET applications |
| **CLR** | The virtual machine that executes managed code |

### 1.3.3 Common Language Runtime (CLR)

The **CLR** is the **managed execution environment** for .NET applications. It:

- Manages memory, thread execution, code execution, code safety verification, and compilation
- Provides **automatic garbage collection**
- Enforces **type safety** through the Common Type System (CTS)
- Compiles **Intermediate Language (IL)** into machine code via **Just-In-Time (JIT) compilation**

> "The CLR is the managed execution environment that can host applications written in many different programming languages."

### 1.3.4 .NET SDK

The **.NET SDK** contains everything required to **build and develop** .NET programs. When you install an IDE like Visual Studio, it includes the .NET SDK (which includes the .NET Runtime as well).

The SDK includes:
- The **C# compiler** (Roslyn)
- The **.NET CLI** (`dotnet`)
- **Templates** for new projects
- **All three runtimes**: .NET Runtime, .NET Desktop Runtime, ASP.NET Core Runtime

### 1.3.5 .NET Runtime

The **.NET Runtime** is what **executes** .NET applications. Without the .NET Runtime, you cannot run any .NET programs.

> "The runtime runs .NET apps, and the SDK is used to create apps."

### 1.3.6 Relationship Diagram

```
┌─────────────────────────────────────────────────────────────┐
│                        .NET SDK                              │
│  ┌───────────────────────────────────────────────────────┐   │
│  │  C# Compiler (Roslyn)                                  │   │
│  │  .NET CLI (dotnet)                                     │   │
│  │  Templates                                             │   │
│  └───────────────────────────────────────────────────────┘   │
│  ┌───────────────────────────────────────────────────────┐   │
│  │              .NET Runtime (CLR + BCL)                  │   │
│  │  ┌─────────────────────────────────────────────────┐   │   │
│  │  │  Common Language Runtime (CLR)                  │   │   │
│  │  │  - JIT Compiler                                 │   │   │
│  │  │  - Garbage Collector                            │   │   │
│  │  │  - Type Safety                                  │   │   │
│  │  └─────────────────────────────────────────────────┘   │   │
│  │  ┌─────────────────────────────────────────────────┐   │   │
│  │  │  Base Class Library (BCL)                       │   │   │
│  │  └─────────────────────────────────────────────────┘   │   │
│  └───────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────┘
```

### 1.3.7 Summary Table

| Component | Role | Relationship |
|---|---|---|
| **C#** | Programming language | Runs on .NET |
| **.NET** | Development platform | Provides runtime and SDK |
| **CLR** | Execution engine | Part of .NET Runtime |
| **.NET SDK** | Build/development tools | Includes runtime and compiler |
| **.NET Runtime** | Application execution | Contains CLR and BCL |

---

## 1.4 Managed Versus Unmanaged Execution

### 1.4.1 Managed Code

**Managed code** is code whose **execution is managed by a runtime** — specifically, the **Common Language Runtime (CLR)**.

The CLR is in charge of:
- Taking the managed code
- **Compiling it into machine code**
- **Executing it**
- Providing **automatic memory management**
- Enforcing **security boundaries**
- Ensuring **type safety**

> "Managed code is just that: code whose execution is managed by a runtime. In this case, the runtime in question is called the Common Language Runtime or CLR."

**Managed code is written in high-level languages** that run on .NET:
- C#
- Visual Basic
- F#
- Others

### 1.4.2 Unmanaged Code

**Unmanaged code** is code that runs **directly by the operating system**, without the CLR managing execution. Examples include:

- C/C++ programs compiled to native binaries
- COM components
- Native Win32 applications
- Visual Basic 6 programs

> "In the unmanaged world, the programmer is in charge of pretty much everything. The actual program is, essentially, a binary that the operating system (OS) loads into memory and starts. Everything else, from memory management to security considerations are a burden of the programmer."

### 1.4.3 Comparison

| Aspect | Managed Code | Unmanaged Code |
|---|---|---|
| **Runtime** | CLR manages execution | OS executes directly |
| **Memory** | Automatic garbage collection | Manual (malloc/free, new/delete) |
| **Type safety** | Enforced by CLR | Programmer's responsibility |
| **Security** | CLR enforces boundaries | OS-level only |
| **Portability** | IL is platform-independent | Platform-specific binaries |
| **Languages** | C#, VB.NET, F# | C, C++, COM |

### 1.4.4 Interoperability

The CLR allows **passing boundaries between managed and unmanaged code** — a process called **interoperability** (or **interop** for short).

C# can use **unmanaged constructs such as pointers** directly in code by utilizing what is known as **unsafe context**, which designates a piece of code for which execution isn't managed by the CLR.

> "The CLR allows passing the boundaries between managed and unmanaged world, and there's a lot of code that does that, even in the .NET class libraries."

---

## 1.5 Compiled and Intermediate-Language Execution Model

### 1.5.1 The Two-Stage Compilation Process

C# uses a **two-stage compilation model**:

```
Stage 1: C# Source Code → C# Compiler (Roslyn) → Intermediate Language (IL)
Stage 2: IL → CLR JIT Compiler → Native Machine Code → Execution
```

> "C# code is not compiled directly into machine language. The C# compiler converts C# source code into Microsoft intermediate language (MSIL) files, called assemblies."

### 1.5.2 Stage 1: Compilation to Intermediate Language

The C# compiler (**Roslyn**) converts C# source code into **Common Intermediate Language (CIL)** — also called **MSIL** (Microsoft Intermediate Language) or simply **IL**.

The result is an **assembly** — a `.exe` or `.dll` file containing:
- **IL code** — the platform-independent instructions
- **Metadata** — type information, method signatures, references
- **Manifest** — assembly identity and dependencies

> "The C# compiler converts C# source code into Microsoft intermediate language (MSIL) files, called assemblies."

**Key point:** IL is **independent from any specific language** that runs on .NET. All .NET languages — C#, VB.NET, F# — compile to the same IL.

### 1.5.3 Stage 2: Just-In-Time (JIT) Compilation

When the program runs, the **CLR** compiles the IL into **native machine code** on demand — a process called **Just-In-Time (JIT) compilation**.

> "The CLR compiles the MSIL program into machine code when required; a process called Just-In-Time (JIT) compilation. This machine code is then executed directly."

**How JIT works:**
1. The CLR loads the assembly
2. When a method is first called, the JIT compiler translates its IL to native code
3. The native code is cached for subsequent calls
4. Execution proceeds at native speed

### 1.5.4 The Full Execution Pipeline

```
┌─────────────────────────────────────────────────────────────────────┐
│  1. C# SOURCE CODE                                                  │
│     (Program.cs)                                                    │
└─────────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────────┐
│  2. C# COMPILER (Roslyn)                                            │
│     - Parses C# source                                              │
│     - Performs semantic analysis                                    │
│     - Emits IL and metadata                                         │
└─────────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────────┐
│  3. ASSEMBLY (.dll / .exe)                                          │
│     - Common Intermediate Language (CIL)                            │
│     - Metadata                                                      │
│     - Manifest                                                      │
└─────────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────────┐
│  4. CLR (Common Language Runtime)                                   │
│     - Loads assembly                                                │
│     - Verifies type safety                                          │
│     - Manages memory                                                │
└─────────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────────┐
│  5. JIT COMPILER                                                    │
│     - Converts IL to native machine code                            │
│     - Optimizes for current CPU                                     │
│     - Caches native code                                            │
└─────────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────────┐
│  6. NATIVE MACHINE CODE                                             │
│     - Executed directly by the CPU                                  │
└─────────────────────────────────────────────────────────────────────┘
```

### 1.5.5 Advantages of the IL Model

| Advantage | Explanation |
|---|---|
| **Platform independence** | IL is hardware-neutral; JIT adapts to the target CPU |
| **Language interoperability** | All .NET languages compile to the same IL |
| **Security** | CLR verifies IL before execution |
| **Optimization** | JIT can optimize for the specific runtime environment |
| **Portability** | Same assembly runs on different architectures |

> "By generating intermediate, hardware-neutral code that is not converted into machine code until the last moment, safety, security, and portability are all increased."

### 1.5.6 Native Image Generation (NGen)

In some cases, it is desirable to **permanently convert a C# assembly into machine language**. This can be done with the **Native Image Generator (ngen.exe)** tool that ships with Visual Studio. This pre-compiles IL to native code at install time, reducing JIT overhead at startup.

---

## 1.6 C# Language Specifications and Version Evolution

### 1.6.1 The C# Language Specification

The **C# language specification** is the **definitive source** for the C# language. It describes:

- The **form** of programs written in C#
- The **interpretation** of those programs

The specification is produced by the **ECMA C# standard committee (TC49-TG2)**.

### 1.6.2 ECMA and ISO Standards

C# is standardized by **Ecma International** and **ISO/IEC**.

| Standard | Edition | Year | Description |
|---|---|---|---|
| **ECMA-334** | 1st | December 2001 | First edition |
| **ECMA-334** | 2nd | December 2002 | — |
| **ECMA-334** | 5th | December 2017 | — |
| **ECMA-334** | 6th | — | C# 6.0 |
| **ECMA-334** | 7th | December 2023 | Latest edition |
| **ISO/IEC 20619** | — | 2023 | Incorporates ECMA-334 by reference |

> "Ecma International ratified the first edition of the C# specification as ECMA-334 in December 2001, and the ECMA-334 standard has since reached its seventh edition, published in December 2023."

### 1.6.3 Specification vs. Implementation

The ECMA C# standard committee is currently **finalizing version 8 of the standard**. The draft published is a **feature-complete early draft of C# 13** based on the committee's initial work.

> "The committee is currently finalizing version 8 of the standard. The draft published here is a feature complete early draft of C# 13 based on the committee's initial work."

**Important:** The **standard specification** may lag behind the **Microsoft implementation**. Microsoft's C# compiler (Roslyn) often implements features before they are standardized.

### 1.6.4 Minimum SDK Versions

Each C# version requires a minimum SDK version to support all language features:

| C# Version | Minimum SDK / Visual Studio |
|---|---|
| **C# 11** | Visual Studio 2022 v17.4, or .NET 7 SDK |
| **C# 10** | Visual Studio 2022, or .NET 6 SDK |
| **C# 9.0** | Visual Studio 2019 v16.8, or .NET 5 SDK |
| **C# 8.0** | Visual Studio 2019 v16.3, or .NET Core 3.0 SDK |
| **C# 7.3** | Visual Studio 2017 v15.7 |
| **C# 7.0** | Visual Studio 2017 |
| **C# 6** | Visual Studio 2015 |
| **C# 5** | Visual Studio 2012 or .NET Framework 4.5 |
| **C# 4** | Visual Studio 2010 or .NET Framework 4.0 |
| **C# 3** | Visual Studio 2008 or .NET Framework 3.5 |
| **C# 2** | Visual Studio 2005 or .NET Framework 2.0 |
| **C# 1.0/1.2** | Visual Studio .NET 2002 |

> "The following table lists the minimum versions of the SDK with the C# compiler that supports the corresponding language version."

---

## 1.7 C# Application Domains and Common Application Types

### 1.7.1 Application Domains (AppDomain)

An **application domain** is the **runtime unit of isolation** in which a .NET program runs. It acts as a **container and boundary** for the types defined in the application and the class libraries it uses.

| Aspect | Description |
|---|---|
| **Purpose** | Isolation of security, versioning, reliability, and unloading |
| **Types** | Types loaded into one domain are distinct from the same type in another domain |
| **Instances** | Objects are not directly shared between domains |
| **Relationship to processes** | A single process can host multiple application domains |
| **Relationship to threads** | Threads can cross application domain boundaries |

> "An application domain forms an isolation boundary for security, versioning, reliability, and unloading of managed code."

**Note:** In modern .NET (Core and .NET 5+), `AppDomain` is largely replaced by **AssemblyLoadContext** for assembly isolation, though the concept remains relevant for understanding .NET architecture.

### 1.7.2 Common C# Application Types

C# can be used to build a wide variety of applications:

#### Console Applications

Command-line programs that interact via text input/output.

```csharp
Console.WriteLine("Hello, World!");
```

#### Web Applications

Websites, REST APIs, and microservices built with **ASP.NET Core**.

| Framework | Purpose |
|---|---|
| **ASP.NET Core MVC** | Full-featured web applications |
| **ASP.NET Core Web API** | RESTful APIs |
| **Blazor** | Interactive web UIs with C# |
| **SignalR** | Real-time web functionality |

#### Desktop Applications

Rich client applications for Windows, macOS, and Linux.

| Framework | Description |
|---|---|
| **Windows Forms** | Traditional Windows desktop apps |
| **WPF** | Modern Windows desktop apps with XAML |
| **MAUI** | Cross-platform desktop and mobile apps |
| **Avalonia** | Cross-platform desktop UI framework |

#### Mobile Applications

Cross-platform mobile apps using **.NET MAUI** (successor to Xamarin).

#### Cloud and Microservices

Cloud-native applications, serverless functions, and microservices.

| Platform | Description |
|---|---|
| **Azure Functions** | Serverless compute |
| **Azure App Service** | Web app hosting |
| **Docker/Kubernetes** | Containerized microservices |
| **gRPC** | High-performance RPC |

#### Games

Game development with **Unity**, one of the most popular game engines in the world.

#### IoT and Embedded Systems

Applications for **Internet of Things (IoT)** devices and embedded systems.

#### AI and Machine Learning

Machine learning applications using **ML.NET** and integration with **ONNX** models.

### 1.7.3 Application Types Summary

| Application Type | Key Frameworks | Typical Use |
|---|---|---|
| **Console** | .NET CLI | Utilities, scripts, learning |
| **Web** | ASP.NET Core | Websites, APIs, microservices |
| **Desktop** | WPF, WinForms, MAUI | Windows/macOS/Linux apps |
| **Mobile** | .NET MAUI | iOS, Android, Windows apps |
| **Cloud** | Azure Functions, ASP.NET Core | Serverless, containers |
| **Games** | Unity | 2D/3D games |
| **IoT** | .NET IoT | Raspberry Pi, sensors |
| **AI/ML** | ML.NET, ONNX | Prediction, classification |

---

## Summary Table

| Topic | Key Points |
|---|---|
| **C# Definition** | Simple, modern, general-purpose, object-oriented language |
| **Platform** | Runs on .NET; cross-platform |
| **Inventor** | Anders Hejlsberg, Scott Wiltamuth, Peter Golde |
| **First Release** | July 2000, .NET Framework |
| **Standard** | ECMA-334, ISO/IEC 20619 |
| **Latest Version** | C# 14 (November 2025) |
| **CLR** | Common Language Runtime — managed execution engine |
| **.NET SDK** | Build tools + runtime + compiler |
| **.NET Runtime** | Executes .NET applications |
| **Managed Code** | Execution managed by CLR |
| **Unmanaged Code** | Executed directly by OS |
| **IL** | Intermediate Language — platform-independent |
| **JIT** | Just-In-Time compilation to native code |
| **AppDomain** | Runtime isolation unit |
| **Application Types** | Console, Web, Desktop, Mobile, Cloud, Games, IoT, AI |

---

## Key Takeaways

1. **C#** is a **simple, modern, general-purpose, object-oriented** language designed for the .NET platform.
2. It was created by **Anders Hejlsberg** and team, first released in **July 2000**, and standardized as **ECMA-334** in December 2001.
3. **C# runs on .NET**, which consists of the **.NET Runtime** (for execution) and the **.NET SDK** (for development).
4. The **CLR** is the managed execution environment — it handles **memory management, type safety, security, and JIT compilation**.
5. **Managed code** runs under CLR control; **unmanaged code** runs directly by the OS.
6. C# uses a **two-stage compilation model**: C# → **IL** → (JIT) → **native machine code**.
7. **IL** is platform-independent and language-independent — all .NET languages compile to the same IL.
8. **JIT compilation** converts IL to native code on demand at runtime.
9. C# is standardized by **Ecma International** (ECMA-334) and **ISO/IEC** (20619).
10. C# is used for **console apps, web apps, desktop apps, mobile apps, cloud services, games, IoT, and AI/ML**.

---

Would you like me to continue with the next topic — **C# Setup and Environment**, **C# Basic Syntax**, or **C# Data Types**? I can format the next section in the same style.