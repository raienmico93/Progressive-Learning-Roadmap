# CMake — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
CMake is a cross-platform, open-source meta-build system (also called a build-system generator) that manages the build process of software projects through a family of configuration files, primarily `CMakeLists.txt`, and generates native build files for Make, Ninja, Visual Studio, Xcode, and other toolsets.

**Technical Definition**
CMake uses its own domain-specific language to describe a software project as a set of high-level logical targets — executables, libraries, and custom targets. Each target has properties (source files, include directories, compile definitions, compile features, link dependencies) that encapsulate usage requirements. Since version 3.x, the "modern CMake" paradigm mandates target-centric configuration via `target_*` commands, replacing global directory-level settings. Dependencies between binary targets are expressed using `target_link_libraries()`, and external packages are discovered via `find_package()` or the `FetchContent` module. CMake evaluates project configuration through a two-step process: a **configure** step (where CMake reads `CMakeLists.txt` files, performs feature detection, and generates native build files) and a **build** step (where the native tool, e.g., Ninja, executes the generated build rules).

**Beginner-Friendly Explanation**
Imagine you are managing a large construction project with hundreds of workers (source files) who need to be organised into teams (libraries) and assigned tasks in the correct order (dependencies). Writing individual instructions for each worker by hand would be impossible. CMake is like the chief architect: you describe the overall plan once in a `CMakeLists.txt` file ("these are my source files, these are my libraries, this depends on that"), and CMake generates detailed blueprints (native build files) for each platform. You write the project description once, and it works on Windows, Linux, and macOS.

### Key Characteristics

- **Meta-build system** — generates native build files for other tools (Make, Ninja, Visual Studio, Xcode) rather than executing the build itself.
- **Target-centric** — modern CMake configures everything on targets using `target_*` commands, not on global directories or variables.
- **Cross-platform** — abstracts compiler, platform, and IDE differences behind a single project description.
- **Transitive usage requirements** — `PUBLIC`, `PRIVATE`, and `INTERFACE` keywords control how dependencies propagate to consumers.
- **Generator expressions** — conditional, configuration-specific logic embedded directly in target properties using `$<...>` syntax.
- **Presets** — `CMakePresets.json` and `CMakeUserPresets.json` provide a standard way to share configure, build, and test settings.
- **Extensible** — supports custom commands, code generation, and integration with package managers (vcpkg, Conan).

### Prerequisites

- Basic familiarity with compiling C++ code from the command line (`g++`/`clang++`/`cl`).
- Understanding of translation units, object files, static and shared libraries.
- Awareness of header files and include paths.
- Willingness to learn the CMake language (a small, purpose-built DSL).

### Related Programming Areas

- **Compiler toolchains** — GCC, Clang, MSVC, and their flags.
- **Package management** — Conan, vcpkg, and CMake's `FetchContent`.
- **IDE and tooling integration** — compile commands databases, Visual Studio, Xcode, CLion, VS Code.
- **Continuous integration (CI)** — automated builds, tests, and packaging.
- **Cross-compilation** — toolchain files for embedded and mobile targets.

### Core Concepts / Features

1. CMake (general)
2. `CMakeLists.txt`
3. CMake commands
4. Targets
5. Libraries
6. Executables
7. Dependencies
8. `find_package`
9. `target_link_libraries`
10. `target_include_directories`
11. `target_compile_options`
12. `target_compile_features`
13. Generator expressions
14. CMake presets
15. CMake best practices

---

## 1. CMake

### Definitions

**Core Definition**
CMake is a cross-platform build-system generator that reads project configuration files and produces native build files for a chosen toolchain and backend.

**Technical Definition**
CMake is implemented in C++ and distributed as a command-line tool (`cmake`) and a C++ library. It operates in two phases: **configure** (parsing `CMakeLists.txt`, evaluating generator expressions, detecting the compiler and platform, and writing a `CMakeCache.txt` and generated build files) and **generate** (producing the native build files). The generated build system is then executed by the native tool (e.g., Ninja or Make). CMake also provides a scripting mode (`cmake -P`), a command-line tool mode (`cmake -E`), and a testing driver (CTest).

**Beginner-Friendly Explanation**
CMake is the tool you run to set up your project's build. You tell it where your source code is and what kind of build files you want (e.g., Ninja or Visual Studio), and it produces everything needed to compile and link your program.

### Purposes

- To provide a single, portable project description across platforms and IDEs.
- To generate native build files for many different backends.
- To manage complex dependency graphs and third-party libraries.
- To integrate with IDEs, tooling, and package managers.
- To support modern C++ features and target-based configuration.

### Syntax Rules and Structure

**General Syntax**
```bash
cmake -S <source-dir> -B <build-dir> [options]
cmake --build <build-dir> [--config <config>]
```

**Component Breakdown**

| Command / Option | Description |
|------------------|-------------|
| `-S <dir>` | Source directory containing `CMakeLists.txt`. |
| `-B <dir>` | Build directory for generated files. |
| `-G <generator>` | Select the native build tool (e.g., Ninja, Visual Studio 17 2022). |
| `-D<VAR>=<value>` | Define a cache variable. |
| `--build` | Invoke the native build tool. |
| `--config` | Select the build configuration (Debug, Release, etc.). |

**Syntax Rules**
- The configure step must succeed before the build step can be executed.
- The build directory should be separate from the source directory (out-of-source build) to avoid polluting the source tree.
- CMake caches configuration results in `CMakeCache.txt`; delete the build directory to start fresh.

**Constraints and Limitations**
- CMake is not a general-purpose programming language; complex logic can become awkward.
- Configuration can be slow for very large projects.
- Legacy CMake code (pre-3.0) often uses anti-patterns; "modern CMake" requires discipline.

### Annotated Code Example

```bash
# Configure with Ninja backend
cmake -S . -B build -G Ninja -DCMAKE_BUILD_TYPE=Release

# Build the project
cmake --build build
```

**Expected Outcome**  
CMake configures the project in the `build/` directory, generating Ninja files. The build step compiles and links the project using Ninja.

**Why this works**  
`-S` specifies the source directory; `-B` specifies the build directory; `-G Ninja` selects the Ninja generator; `--build` invokes Ninja.

### Real-World Cases

- **Open-source C++ libraries** — the vast majority ship with CMake support.
- **Cross-platform applications** — one `CMakeLists.txt` generates Visual Studio, Xcode, and Ninja files.
- **Large organisations** — CMake is the de facto standard for C++ build management.

### References

- CMake Documentation — https://cmake.org/documentation/
- CMake Tutorial — https://cmake.org/cmake/help/latest/guide/tutorial/

---

## 2. `CMakeLists.txt`

### Definitions

**Core Definition**
`CMakeLists.txt` is the primary configuration file read by CMake, containing the project description written in the CMake language.

**Technical Definition**
A `CMakeLists.txt` file is a sequence of CMake commands that declare the project, define targets, set target properties, and specify dependencies. Every directory that contains a `CMakeLists.txt` is a **CMake directory**, and CMake processes them recursively (controlled by `add_subdirectory()`). The file may use variables, functions, macros, conditional logic, and generator expressions.

**Beginner-Friendly Explanation**
`CMakeLists.txt` is the recipe that tells CMake how to build your project. It lists your source files, defines your libraries and executables, and specifies where to find dependencies.

### Purposes

- To declare the project name, version, and languages.
- To define targets (libraries and executables).
- To specify source files, include directories, and compile options.
- To declare dependencies on other targets and external packages.
- To configure platform-specific and configuration-specific settings.

### Syntax Rules and Structure

**General Syntax**
```cmake
cmake_minimum_required(VERSION 3.25)
project(MyProject VERSION 1.0.0 LANGUAGES CXX)

add_library(mylib src/lib.cpp)
target_include_directories(mylib PUBLIC include)
target_compile_features(mylib PUBLIC cxx_std_20)

add_executable(myapp src/main.cpp)
target_link_libraries(myapp PRIVATE mylib)
```

**Component Breakdown**

| Command | Description |
|---------|-------------|
| `cmake_minimum_required` | Sets the minimum CMake version. |
| `project` | Declares the project name, version, and languages. |
| `add_library` | Defines a library target. |
| `add_executable` | Defines an executable target. |
| `target_*` | Configures properties on a specific target. |

**Syntax Rules**
- `cmake_minimum_required()` should be the first command (or immediately after comments).
- `project()` should be called early in the top-level file.
- Commands are case-insensitive but conventionally lowercase.
- Use 2-space indentation for readability.

**Constraints and Limitations**
- A `CMakeLists.txt` file must exist in the top-level source directory.
- Each subdirectory processed with `add_subdirectory()` must also contain a `CMakeLists.txt`.
- CMake language is not Turing-complete; complex logic should be avoided.

### Annotated Code Examples

**Example 1: Minimal `CMakeLists.txt`**

```cmake
cmake_minimum_required(VERSION 3.25)
project(hello LANGUAGES CXX)

add_executable(hello src/main.cpp)
```

**Expected Outcome**  
CMake generates build files that compile `src/main.cpp` into the `hello` executable.

**Why this works**  
`add_executable` declares the target; CMake generates the appropriate compile and link rules.

**Example 2: Library and executable with modern CMake**

```cmake
cmake_minimum_required(VERSION 3.25)
project(myapp VERSION 0.1.0 LANGUAGES CXX)

add_library(mylib src/api.cpp src/util.cpp)
target_include_directories(mylib PUBLIC
    $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
    $<INSTALL_INTERFACE:include>
    PRIVATE src
)
target_compile_features(mylib PUBLIC cxx_std_20)
target_compile_options(mylib PRIVATE
    $<$<CXX_COMPILER_ID:MSVC>:/W4 /WX /permissive->
    $<$<NOT:$<CXX_COMPILER_ID:MSVC>>:-Wall -Wextra -Wpedantic -Werror>
)

add_executable(myapp src/main.cpp)
target_link_libraries(myapp PRIVATE mylib)
```

**Expected Outcome**  
`mylib` is built as a static library; `myapp` links against it. Compiler warnings and C++20 features are configured per-target.

**Why this works**  
All configuration is attached to targets via `target_*` commands; generator expressions handle compiler-specific flags and build/install interface differences.

### Real-World Cases

- **Every CMake-based project** — the `CMakeLists.txt` is the single source of truth for the build.
- **Multi-directory projects** — `add_subdirectory()` composes a hierarchy of `CMakeLists.txt` files.
- **Library and application monorepos** — a top-level file declares common settings and includes subdirectories.

### References

- CMake Language Reference — https://cmake.org/cmake/help/latest/manual/cmake-language.7.html
- `CMakeLists.txt` (cmake-buildsystem manual) — https://cmake.org/cmake/help/latest/manual/cmake-buildsystem.7.html

---

## 3. CMake Commands

### Definitions

**Core Definition**
CMake commands are the built-in functions and control structures that make up the CMake language, used to define projects, targets, properties, and dependencies.

**Technical Definition**
CMake commands fall into categories: **project commands** (`project`, `cmake_minimum_required`), **target commands** (`add_library`, `add_executable`, `target_*`), **dependency commands** (`find_package`, `FetchContent`), **directory commands** (`add_subdirectory`, `include_directories`), **property commands** (`set_target_properties`, `get_target_property`), **control flow** (`if`, `foreach`, `while`, `function`, `macro`), and **utility commands** (`message`, `install`, `configure_file`). Commands are case-insensitive but conventionally lowercase.

**Beginner-Friendly Explanation**
CMake commands are the "verbs" of the `CMakeLists.txt` language. They tell CMake what to do: create a library, add source files, find a dependency, set a compiler flag, or print a message.

### Purposes

- To declare and configure project targets.
- To manage dependencies and third-party packages.
- To control the build process (conditionals, loops, functions).
- To generate files and install artifacts.
- To communicate information to the user.

### Syntax Rules and Structure

**General Syntax**
```cmake
command_name(arg1 arg2 ...)
```

**Component Breakdown**

| Category | Example Commands |
|----------|------------------|
| Project | `cmake_minimum_required`, `project` |
| Targets | `add_library`, `add_executable`, `add_custom_target` |
| Target properties | `target_link_libraries`, `target_include_directories`, `target_compile_options`, `target_compile_features`, `target_sources` |
| Dependencies | `find_package`, `FetchContent_Declare`, `FetchContent_MakeAvailable` |
| Control flow | `if`, `elseif`, `else`, `endif`, `foreach`, `while`, `function`, `macro` |
| Output | `message`, `install`, `configure_file` |

**Syntax Rules**
- Command names are case-insensitive.
- Arguments are separated by whitespace; arguments containing spaces must be quoted.
- Variable references use `${VAR}`; generator expressions use `$<...>`.
- Commands can be nested (e.g., `$<TARGET_PROPERTY:tgt,INCLUDE_DIRECTORIES>`).

**Constraints and Limitations**
- CMake commands are not a general-purpose programming language; avoid overly complex logic.
- Some commands are deprecated (e.g., `link_libraries`, `include_directories` without `target_`); prefer modern alternatives.

### Annotated Code Example

```cmake
cmake_minimum_required(VERSION 3.25)
project(demo LANGUAGES CXX)

# Control flow
if(CMAKE_CXX_COMPILER_ID STREQUAL "GNU")
    add_compile_options(-Wall -Wextra)
endif()

# Target commands
add_library(core src/core.cpp)
target_include_directories(core PUBLIC include)

add_executable(app src/main.cpp)
target_link_libraries(app PRIVATE core)

# Utility command
message(STATUS "Building ${PROJECT_NAME} with ${CMAKE_CXX_COMPILER_ID}")
```

**Expected Outcome**  
CMake configures the project, adds warnings for GCC, builds the `core` library and `app` executable, and prints a status message.

**Why this works**  
`if()` conditionally adds options; `add_library` and `add_executable` create targets; `target_*` commands configure them; `message` outputs information.

### Real-World Cases

- **All CMake projects** — every `CMakeLists.txt` is composed of these commands.
- **Conditional compilation** — platform- or compiler-specific settings via `if()`.
- **Dependency management** — `find_package` and `FetchContent` for external libraries.

### References

- CMake Commands Reference — https://cmake.org/cmake/help/latest/manual/cmake-commands.7.html

---

## 4. Targets

### Definitions

**Core Definition**
A target in CMake is a high-level logical unit representing an executable, a library, or a custom command, which encapsulates build rules, properties, and dependencies.

**Technical Definition**
Targets are the fundamental organisational unit of a CMake buildsystem. A target has a globally unique name, a type (executable, static library, shared library, object library, interface library, or custom target), and a set of properties (source files, include directories, compile definitions, compile options, compile features, link dependencies). Dependencies between binary targets are expressed with `target_link_libraries()`. Usage requirements (include directories, compile definitions, compile features) propagate transitively through `PUBLIC` and `INTERFACE` properties.

**Beginner-Friendly Explanation**
A target is like a "team" in your project: each team (library or executable) has members (source files), tools (include directories, compiler flags), and dependencies on other teams. You configure everything on the team, not on the whole organisation.

### Purposes

- To organise the build into logical units (libraries and executables).
- To encapsulate usage requirements and propagate them transitively.
- To provide a single, consistent interface for configuring build properties.
- To enable modular, composable project structures.

### Syntax Rules and Structure

**General Syntax**
```cmake
add_library(<name> [type] <sources>...)
add_executable(<name> <sources>...)
```

**Component Breakdown**

| Target Type | Command | Description |
|-------------|---------|-------------|
| Executable | `add_executable` | Produces an executable binary. |
| Static library | `add_library(<name> STATIC ...)` | Archives object files into a `.a`/`.lib`. |
| Shared library | `add_library(<name> SHARED ...)` | Produces a `.so`/`.dll`/`.dylib`. |
| Object library | `add_library(<name> OBJECT ...)` | Compiles sources without archiving. |
| Interface library | `add_library(<name> INTERFACE)` | Header-only or usage-requirement-only target. |

**Syntax Rules**
- Target names must be globally unique and may contain letters, digits, `_`, `.`, `+`, and `-`.
- By convention, imported targets use `::` as a namespace separator (e.g., `fmt::fmt`).
- Target properties are configured with `target_*` commands, not global variables.

**Constraints and Limitations**
- Target names cannot be empty.
- Non-`GLOBAL` imported targets are visible only in the directory that defines them and its subdirectories.
- `add_library` defines a `STATIC` library by default if no type is specified.

### Annotated Code Examples

**Example 1: Executable target**

```cmake
add_executable(myapp src/main.cpp src/util.cpp)
```

**Expected Outcome**  
CMake compiles the two source files and links them into the `myapp` executable.

**Why this works**  
`add_executable` declares the target; CMake generates compile and link rules.

**Example 2: Library and executable targets**

```cmake
add_library(mylib src/lib.cpp)
target_include_directories(mylib PUBLIC include)

add_executable(myapp src/main.cpp)
target_link_libraries(myapp PRIVATE mylib)
```

**Expected Outcome**  
`mylib` is built as a static library; `myapp` links against it and inherits its public include directories.

**Why this works**  
The library target encapsulates its usage requirements (`PUBLIC` include directories); the executable target depends on it.

### Real-World Cases

- **Library ecosystems** — each library is a target with its own include directories and dependencies.
- **Application bundles** — executables link against multiple library targets.
- **Testing** — test executables are targets that link against the library being tested.

### References

- `cmake-buildsystem(7)` — https://cmake.org/cmake/help/latest/manual/cmake-buildsystem.7.html

---

## 5. Libraries

### Definitions

**Core Definition**
A library target in CMake is a target that produces a reusable collection of compiled code (static, shared, object, or interface) that other targets can link against.

**Technical Definition**
CMake supports four main library types: `STATIC` (archive of object files), `SHARED` (dynamically loadable library), `MODULE` (plugin not linked at build time), and `OBJECT` (object files not archived). `INTERFACE` libraries have no build step and are used purely to carry usage requirements (e.g., header-only libraries). The `add_library()` command defines a library target; its type may be specified explicitly or inferred from the `BUILD_SHARED_LIBS` variable (if set to `ON`, `SHARED` is the default; otherwise `STATIC`).

**Beginner-Friendly Explanation**
A library is a reusable piece of code that other parts of your project (or other projects) can use. A static library is like a toolbox that gets copied into each program that uses it; a shared library is like a toolbox that is loaded at runtime and can be shared between programs.

### Purposes

- To encapsulate reusable code into a single unit.
- To control how code is linked (static vs. shared).
- To propagate usage requirements (include directories, compile features) to consumers.
- To support header-only libraries via `INTERFACE` targets.

### Syntax Rules and Structure

**General Syntax**
```cmake
add_library(<name> [STATIC | SHARED | MODULE | OBJECT | INTERFACE] <sources>...)
```

**Component Breakdown**

| Type | Description | File Extension (typical) |
|------|-------------|--------------------------|
| `STATIC` | Archived object files. | `.a` (Unix), `.lib` (Windows) |
| `SHARED` | Dynamically linked library. | `.so` (Linux), `.dylib` (macOS), `.dll` (Windows) |
| `MODULE` | Plugin, not linked at build time. | `.so`/`.dylib`/`.dll` |
| `OBJECT` | Object files compiled but not archived. | `.o`/`.obj` |
| `INTERFACE` | No build step; carries usage requirements. | — |

**Syntax Rules**
- If no type is given, `STATIC` is used unless `BUILD_SHARED_LIBS` is `ON`.
- `SHARED` libraries on Windows require exported symbols (e.g., `__declspec(dllexport)` or `WINDOWS_EXPORT_ALL_SYMBOLS`).
- `INTERFACE` libraries must not have sources; they are configured entirely with `target_*` commands.

**Constraints and Limitations**
- On platforms that do not support shared libraries, `add_library(SHARED)` fails.
- Object libraries cannot be linked directly; use `$<TARGET_OBJECTS:objlib>` or link via `target_link_libraries`.

### Annotated Code Examples

**Example 1: Static library**

```cmake
add_library(mylib STATIC src/lib.cpp)
target_include_directories(mylib PUBLIC include)
```

**Expected Outcome**  
`mylib` is built as a static library; its public include directory is propagated to consumers.

**Why this works**  
`STATIC` explicitly requests a static library; `PUBLIC` include directories are usage requirements.

**Example 2: Header-only `INTERFACE` library**

```cmake
add_library(mylib INTERFACE)
target_include_directories(mylib INTERFACE include)
target_compile_features(mylib INTERFACE cxx_std_20)
```

**Expected Outcome**  
No build step is performed; consumers of `mylib` inherit the include directory and C++20 requirement.

**Why this works**  
`INTERFACE` libraries carry usage requirements without producing a binary.

### Real-World Cases

- **Static libraries** — bundled into executables for self-contained deployment.
- **Shared libraries** — used by multiple executables, updated independently.
- **Header-only libraries** — `INTERFACE` targets for header-only utilities.
- **Plugin systems** — `MODULE` libraries loaded dynamically.

### References

- `add_library` — https://cmake.org/cmake/help/latest/command/add_library.html

---

## 6. Executables

### Definitions

**Core Definition**
An executable target in CMake is a target that produces a runnable binary program from a set of source files, linked against any required libraries.

**Technical Definition**
The `add_executable()` command defines an executable target. CMake generates build rules to compile the listed source files into object files and link them into an executable, using the platform's appropriate prefix, suffix, and extension. Executables can be linked against library targets via `target_link_libraries()`, and may use generator expressions in their source lists.

**Beginner-Friendly Explanation**
An executable is the final program that a user runs. You list its source files and the libraries it needs, and CMake produces the binary.

### Purposes

- To produce runnable programs from source code.
- To link application code against libraries.
- To create test executables and tools.
- To support platform-specific executable variants (`WIN32`, `MACOSX_BUNDLE`).

### Syntax Rules and Structure

**General Syntax**
```cmake
add_executable(<name> [WIN32] [MACOSX_BUNDLE] <sources>...)
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `<name>` | Target name (must be unique). |
| `WIN32` | Marks the executable as a Windows GUI application (no console). |
| `MACOSX_BUNDLE` | Produces a macOS application bundle. |
| `<sources>` | Source files to compile and link. |

**Syntax Rules**
- At least one source file is typically required (unless using `$<TARGET_OBJECTS:...>`).
- `WIN32` is ignored on non-Windows platforms.
- Executables can depend on object libraries via `$<TARGET_OBJECTS:objlib>`.

**Constraints and Limitations**
- Executable targets cannot be linked as libraries.
- The `main` function (or platform equivalent) must be provided by one of the sources.

### Annotated Code Examples

**Example 1: Basic executable**

```cmake
add_executable(myapp src/main.cpp)
```

**Expected Outcome**  
CMake compiles `src/main.cpp` and links it into the `myapp` executable.

**Why this works**  
`add_executable` declares the target and its sources.

**Example 2: Executable with libraries**

```cmake
add_executable(myapp src/main.cpp)
target_link_libraries(myapp PRIVATE mylib fmt::fmt)
```

**Expected Outcome**  
`myapp` is linked against `mylib` and the `fmt` library.

**Why this works**  
`target_link_libraries` attaches the dependencies to the executable target.

### Real-World Cases

- **Application binaries** — the final deliverable of most projects.
- **Test runners** — executables built for unit and integration tests.
- **Command-line tools** — utilities and scripts compiled as native binaries.

### References

- `add_executable` — https://cmake.org/cmake/help/latest/command/add_executable.html

---

## 7. Dependencies

### Definitions

**Core Definition**
Dependencies in CMake express the relationships between targets and external packages, determining build order and propagation of usage requirements.

**Technical Definition**
Dependencies are declared in two main ways: **target-to-target** dependencies via `target_link_libraries()`, and **external package** dependencies via `find_package()` or `FetchContent`. Target dependencies create a directed acyclic graph; CMake uses this graph to determine the correct build order and to propagate usage requirements (`PUBLIC`, `PRIVATE`, `INTERFACE`). External package dependencies load imported targets whose properties describe their usage requirements.

**Beginner-Friendly Explanation**
Dependencies are the "depends on" relationships in your project. If your app uses a library, you declare that dependency, and CMake makes sure the library is built first and that the app gets all the compiler flags and include paths it needs.

### Purposes

- To ensure correct build order.
- To propagate usage requirements transitively.
- To integrate third-party libraries.
- To model the structure of the project graph.

### Syntax Rules and Structure

**General Syntax**
```cmake
target_link_libraries(<target> <PRIVATE|PUBLIC|INTERFACE> <dep>...)
find_package(<PackageName> [REQUIRED] [COMPONENTS ...])
```

**Component Breakdown**

| Keyword | Meaning |
|---------|---------|
| `PRIVATE` | Dependency used only by the implementation; not propagated. |
| `PUBLIC` | Dependency used by implementation and header; propagated. |
| `INTERFACE` | Dependency used only by headers; propagated, not linked by the target itself. |

**Syntax Rules**
- Usage requirements are propagated by reading the `INTERFACE_*` properties of dependencies and appending them to the consumer's properties.
- A dependency should be `PRIVATE` if it is used only in implementation files, `PUBLIC` if used in headers, and `INTERFACE` if used only in headers.
- `find_package` creates imported targets (e.g., `fmt::fmt`) that can be linked with `target_link_libraries`.

**Constraints and Limitations**
- Dependency graphs must be acyclic.
- Incorrect visibility can lead to missing include directories or link errors in consumers.

### Annotated Code Examples

**Example 1: Target dependency with visibility**

```cmake
add_library(archive src/archive.cpp)
target_include_directories(archive PUBLIC include)

add_library(serialization src/serialization.cpp)
target_include_directories(serialization PRIVATE src)

add_library(archiveExtras src/extras.cpp)
target_link_libraries(archiveExtras
    PUBLIC archive
    PRIVATE serialization
)

add_executable(consumer src/consumer.cpp)
target_link_libraries(consumer archiveExtras)
```

**Expected Outcome**  
`consumer` inherits the public include directories of `archive` (through `archiveExtras`), but not those of `serialization`.

**Why this works**  
`PUBLIC` dependencies propagate their usage requirements; `PRIVATE` dependencies do not.

**Example 2: External package dependency**

```cmake
find_package(fmt 10 REQUIRED)
target_link_libraries(myapp PRIVATE fmt::fmt)
```

**Expected Outcome**  
CMake finds the `fmt` package and links `myapp` against it, inheriting its usage requirements.

**Why this works**  
`find_package` loads the package's imported target, which carries its include directories and compile features.

### Real-World Cases

- **Library composition** — libraries depend on other libraries with appropriate visibility.
- **Application assembly** — executables link against internal and external libraries.
- **Plugin architectures** — modules depend on interface libraries.

### References

- `cmake-buildsystem(7)` — https://cmake.org/cmake/help/latest/manual/cmake-buildsystem.7.html
- `target_link_libraries` — https://cmake.org/cmake/help/latest/command/target_link_libraries.html

---

## 8. `find_package`

### Definitions

**Core Definition**
`find_package` is a CMake command that locates an external package, loads its package-specific details, and creates imported targets for use in the project.

**Technical Definition**
`find_package(<PackageName> [<version>] [REQUIRED] [COMPONENTS <components>...])` searches for a package in two modes: **Module mode** (looks for a `Find<PackageName>.cmake` file in `CMAKE_MODULE_PATH` or CMake's built-in modules) and **Config mode** (looks for a `<PackageName>Config.cmake` or `<lowercase-package-name>-config.cmake` file installed by the package itself). The result is a set of imported targets (e.g., `fmt::fmt`) and/or variables (`<PackageName>_FOUND`, `<PackageName>_INCLUDE_DIRS`, `<PackageName>_LIBRARIES`).

**Beginner-Friendly Explanation**
`find_package` is how you tell CMake "go find this library on the system and set it up so I can use it." It is the standard way to integrate third-party dependencies.

### Purposes

- To locate and configure external packages.
- To create imported targets with usage requirements.
- To support both system-installed and package-manager-provided dependencies.
- To enable optional and component-based dependencies.

### Syntax Rules and Structure

**General Syntax**
```cmake
find_package(<PackageName> [<version>] [REQUIRED] [COMPONENTS <components>...])
```

**Component Breakdown**

| Argument | Description |
|----------|-------------|
| `<PackageName>` | The name of the package to find (mandatory). |
| `<version>` | Minimum required version. |
| `REQUIRED` | Fail configuration if the package is not found. |
| `COMPONENTS` | Specific components of the package to find. |

**Syntax Rules**
- The basic signature is recommended for general use; the full signature offers more control.
- `find_package` searches in implementation-defined locations; `CMAKE_PREFIX_PATH` can be used to add search prefixes.
- Imported targets use the `Namespace::Target` convention.

**Constraints and Limitations**
- Config mode requires the package to provide a config file; Module mode requires a `Find` module.
- Not all packages provide imported targets; some only set variables.

### Annotated Code Example

```cmake
find_package(fmt 10 REQUIRED)
find_package(Threads REQUIRED)

add_executable(myapp src/main.cpp)
target_link_libraries(myapp PRIVATE fmt::fmt Threads::Threads)
```

**Expected Outcome**  
CMake finds the `fmt` and `Threads` packages and links `myapp` against their imported targets.

**Why this works**  
`find_package` loads the package configuration; `target_link_libraries` uses the imported targets.

### Real-World Cases

- **Third-party libraries** — Boost, fmt, spdlog, Qt, and many others provide CMake config files.
- **System libraries** — `Threads`, `OpenGL`, `ZLIB` via built-in Find modules.
- **Package managers** — Conan and vcpkg generate config files compatible with `find_package`.

### References

- `find_package` — https://cmake.org/cmake/help/latest/command/find_package.html
- Using Dependencies Guide — https://cmake.org/cmake/help/latest/guide/using-dependencies/index.html

---

## 9. `target_link_libraries`

### Definitions

**Core Definition**
`target_link_libraries` is a CMake command that specifies libraries or targets to link against a given target, and controls the propagation of their usage requirements via visibility keywords.

**Technical Definition**
`target_link_libraries(<target> <PRIVATE|PUBLIC|INTERFACE> <item>...)` populates the target's `LINK_LIBRARIES` and `INTERFACE_LINK_LIBRARIES` properties. Items may be target names (linking to other CMake targets, with transitive usage requirements), full paths to library files, or plain library names. The visibility keyword determines whether the dependency is used only for linking the target (`PRIVATE`), also propagated to consumers (`PUBLIC`), or only propagated to consumers (`INTERFACE`).

**Beginner-Friendly Explanation**
`target_link_libraries` tells CMake "this target uses that library." The `PRIVATE`/`PUBLIC`/`INTERFACE` keywords control whether the dependency is just for this target, shared with things that use this target, or only for consumers.

### Purposes

- To link executables and libraries against dependencies.
- To propagate usage requirements transitively.
- To distinguish implementation-only dependencies from public API dependencies.
- To integrate both internal targets and external imported targets.

### Syntax Rules and Structure

**General Syntax**
```cmake
target_link_libraries(<target> <PRIVATE|PUBLIC|INTERFACE> <item>...)
```

**Component Breakdown**

| Keyword | Link the target? | Propagate to consumers? |
|---------|------------------|--------------------------|
| `PRIVATE` | Yes | No |
| `PUBLIC` | Yes | Yes |
| `INTERFACE` | No | Yes |

**Syntax Rules**
- At least one visibility keyword must be specified (with the modern signature).
- Multiple keywords may be used in a single call:
  ```cmake
  target_link_libraries(mylib PUBLIC fmt::fmt PRIVATE spdlog::spdlog)
  ```
- Plain library names (e.g., `pthread`) are passed to the linker; target names (e.g., `fmt::fmt`) carry transitive usage requirements.

**Constraints and Limitations**
- Using `target_link_libraries` without a visibility keyword (legacy signature) is deprecated in modern CMake.
- Library order can matter for some linkers; CMake generally handles this, but generator expressions may be needed in complex cases.

### Annotated Code Examples

**Example 1: Public and private dependencies**

```cmake
add_library(mylib src/lib.cpp)
target_link_libraries(mylib
    PUBLIC fmt::fmt        # consumers also need fmt (used in headers)
    PRIVATE spdlog::spdlog  # implementation only
)
```

**Expected Outcome**  
Consumers of `mylib` automatically link against `fmt::fmt`; `spdlog::spdlog` is used only when building `mylib`.

**Why this works**  
`PUBLIC` propagates the dependency's usage requirements to consumers; `PRIVATE` does not.

**Example 2: Interface library dependency**

```cmake
add_library(myheaderonly INTERFACE)
target_link_libraries(myheaderonly INTERFACE fmt::fmt)
```

**Expected Outcome**  
Consumers of `myheaderonly` link against `fmt::fmt`, even though `myheaderonly` itself has no build step.

**Why this works**  
`INTERFACE` dependencies are propagated to consumers without being linked by the target itself.

### Real-World Cases

- **Library APIs** — public dependencies that appear in headers must be `PUBLIC`.
- **Implementation details** — logging and internal utilities should be `PRIVATE`.
- **Header-only libraries** — `INTERFACE` dependencies propagate requirements without building.

### References

- `target_link_libraries` — https://cmake.org/cmake/help/latest/command/target_link_libraries.html
- `cmake-buildsystem(7)` (visibility) — https://cmake.org/cmake/help/latest/manual/cmake-buildsystem.7.html

---

## 10. `target_include_directories`

### Definitions

**Core Definition**
`target_include_directories` is a CMake command that adds include directories to a target, controlling whether those directories are used for the target itself, propagated to consumers, or both.

**Technical Definition**
`target_include_directories(<target> <PRIVATE|PUBLIC|INTERFACE> <dir>...)` populates the target's `INCLUDE_DIRECTORIES` and `INTERFACE_INCLUDE_DIRECTORIES` properties. The command replaces the legacy directory-level `include_directories()` command in modern CMake. Generator expressions such as `$<BUILD_INTERFACE:...>` and `$<INSTALL_INTERFACE:...>` are commonly used to specify different include paths for build-tree and install-tree usage.

**Beginner-Friendly Explanation**
This command tells CMake where to find header files for a target. `PUBLIC` means "my consumers need this include path too"; `PRIVATE` means "only I need it."

### Purposes

- To specify include directories for compiling a target.
- To propagate include directories to consumers via usage requirements.
- To distinguish between build-tree and install-tree include paths.
- To replace legacy `include_directories()` with target-scoped configuration.

### Syntax Rules and Structure

**General Syntax**
```cmake
target_include_directories(<target> <PRIVATE|PUBLIC|INTERFACE> <dir>...)
```

**Component Breakdown**

| Keyword | Use for target? | Propagate to consumers? |
|---------|-----------------|--------------------------|
| `PRIVATE` | Yes | No |
| `PUBLIC` | Yes | Yes |
| `INTERFACE` | No | Yes |

**Syntax Rules**
- The `SYSTEM` keyword marks the directories as system include directories (suppressing warnings from headers in those directories).
- `BEFORE` and `AFTER` control the order of the directories.
- Generator expressions are supported in directory arguments.

**Constraints and Limitations**
- Relative paths are interpreted relative to the current source directory.
- Using `include_directories()` (directory-level) is deprecated in modern CMake.

### Annotated Code Examples

**Example 1: Public include directory**

```cmake
add_library(mylib src/lib.cpp)
target_include_directories(mylib PUBLIC include)
```

**Expected Outcome**  
`mylib` is compiled with `include/` on its include path; consumers of `mylib` also get `include/` on their include path.

**Why this works**  
`PUBLIC` include directories are both used by the target and propagated.

**Example 2: Build and install interface**

```cmake
add_library(mylib src/lib.cpp)
target_include_directories(mylib PUBLIC
    $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
    $<INSTALL_INTERFACE:include>
)
```

**Expected Outcome**  
During the build, the source-tree include directory is used; after installation, the installed include directory is used.

**Why this works**  
Generator expressions select the appropriate path based on whether the target is being used from the build tree or the install tree.

### Real-World Cases

- **Library APIs** — public headers that consumers must include.
- **Private implementation headers** — `PRIVATE` include directories.
- **Installable packages** — `BUILD_INTERFACE`/`INSTALL_INTERFACE` for relocatable packages.

### References

- `target_include_directories` — https://cmake.org/cmake/help/latest/command/target_include_directories.html

---

## 11. `target_compile_options`

### Definitions

**Core Definition**
`target_compile_options` is a CMake command that adds compiler flags or options to a target, controlling whether those options apply only to the target or are also propagated to consumers.

**Technical Definition**
`target_compile_options(<target> <PRIVATE|PUBLIC|INTERFACE> <option>...)` populates the target's `COMPILE_OPTIONS` and `INTERFACE_COMPILE_OPTIONS` properties. The command is typically used with generator expressions to apply compiler-specific flags (e.g., `-Wall` for GCC/Clang, `/W4` for MSVC). It replaces the legacy `add_compile_options()` and `CMAKE_CXX_FLAGS` approaches.

**Beginner-Friendly Explanation**
This command sets compiler flags for a target, like enabling warnings or optimisations. `PRIVATE` means the flags apply only to this target; `PUBLIC` means they also apply to things that use this target.

### Purposes

- To apply compiler flags to a specific target.
- To propagate necessary compiler options to consumers.
- To handle compiler-specific flags via generator expressions.
- To replace global flag variables with target-scoped configuration.

### Syntax Rules and Structure

**General Syntax**
```cmake
target_compile_options(<target> <PRIVATE|PUBLIC|INTERFACE> <option>...)
```

**Component Breakdown**

| Keyword | Apply to target? | Propagate to consumers? |
|---------|------------------|--------------------------|
| `PRIVATE` | Yes | No |
| `PUBLIC` | Yes | Yes |
| `INTERFACE` | No | Yes |

**Syntax Rules**
- Options are passed directly to the compiler; CMake does not validate them.
- Generator expressions are commonly used to select flags by compiler or platform.
- `target_compile_options` should be preferred over `CMAKE_CXX_FLAGS` for target-specific flags.

**Constraints and Limitations**
- The command does not affect linking; use `target_link_options` for linker flags.
- Some options may not be supported by all compilers; generator expressions are needed for portability.

### Annotated Code Examples

**Example 1: Compiler-specific warnings**

```cmake
add_library(mylib src/lib.cpp)
target_compile_options(mylib PRIVATE
    $<$<CXX_COMPILER_ID:MSVC>:/W4 /WX /permissive->
    $<$<NOT:$<CXX_COMPILER_ID:MSVC>>:-Wall -Wextra -Wpedantic -Werror>
)
```

**Expected Outcome**  
MSVC uses `/W4 /WX /permissive-`; GCC and Clang use `-Wall -Wextra -Wpedantic -Werror`.

**Why this works**  
Generator expressions evaluate the compiler ID and select the appropriate flags.

**Example 2: Public compile option**

```cmake
add_library(mylib src/lib.cpp)
target_compile_options(mylib PUBLIC -fno-exceptions)
```

**Expected Outcome**  
`mylib` and all its consumers are compiled with `-fno-exceptions`.

**Why this works**  
`PUBLIC` options are applied to the target and propagated to consumers.

### Real-World Cases

- **Warning flags** — enabling strict warnings per-target.
- **Sanitisers** — applying `-fsanitize=address` to specific targets.
- **Platform-specific flags** — using generator expressions for Windows/Linux/macOS differences.

### References

- `target_compile_options` — https://cmake.org/cmake/help/latest/command/target_compile_options.html

---

## 12. `target_compile_features`

### Definitions

**Core Definition**
`target_compile_features` is a CMake command that specifies the C++ language features or standard level required to compile a target, ensuring that consumers also receive a sufficient language standard.

**Technical Definition**
`target_compile_features(<target> <PRIVATE|PUBLIC|INTERFACE> <feature>...)` populates the target's `COMPILE_FEATURES` and `INTERFACE_COMPILE_FEATURES` properties. Features are named requirements such as `cxx_std_20` (which requests at least C++20) or specific features like `cxx_constexpr`. CMake maps these requirements to the appropriate compiler flags (e.g., `-std=c++20`) and, critically, also raises the language standard of consumers if their own requirements are lower.

**Beginner-Friendly Explanation**
This command tells CMake "this target needs C++20" (or some other language feature). CMake automatically adds the right compiler flag, and any target that uses this target also gets at least C++20.

### Purposes

- To declare the C++ standard or specific language features required by a target.
- To automatically propagate language requirements to consumers.
- To replace `CMAKE_CXX_STANDARD` with target-scoped configuration.
- To ensure consistent language settings across a project.

### Syntax Rules and Structure

**General Syntax**
```cmake
target_compile_features(<target> <PRIVATE|PUBLIC|INTERFACE> <feature>...)
```

**Component Breakdown**

| Keyword | Use for target? | Propagate to consumers? |
|---------|-----------------|--------------------------|
| `PRIVATE` | Yes | No |
| `PUBLIC` | Yes | Yes |
| `INTERFACE` | No | Yes |

**Syntax Rules**
- The special feature `cxx_std_20` (and similar) requests at least that standard.
- Specific features (e.g., `cxx_constexpr`, `cxx_lambdas`) can be required individually.
- CMake selects the highest standard needed across all targets in a link chain.

**Constraints and Limitations**
- Not all compilers support all features; CMake will error if the requirement cannot be satisfied.
- `cxx_std_XX` features do not disable compiler extensions; use `CXX_EXTENSIONS` property to control that.

### Annotated Code Examples

**Example 1: C++20 requirement**

```cmake
add_library(mylib src/lib.cpp)
target_compile_features(mylib PUBLIC cxx_std_20)
```

**Expected Outcome**  
`mylib` is compiled with at least C++20; consumers also require at least C++20.

**Why this works**  
`PUBLIC` compile features propagate the language requirement to consumers.

**Example 2: Specific feature requirement**

```cmake
add_library(mylib src/lib.cpp)
target_compile_features(mylib PRIVATE cxx_constexpr)
```

**Expected Outcome**  
`mylib` requires `constexpr` support; consumers are unaffected.

**Why this works**  
`PRIVATE` features apply only to the target.

### Real-World Cases

- **Modern C++ libraries** — requiring C++17 or C++20 for consumers.
- **Header-only libraries** — `INTERFACE` compile features propagate language requirements.
- **Cross-platform projects** — CMake handles the compiler-specific `-std=` flag mapping.

### References

- `target_compile_features` — https://cmake.org/cmake/help/latest/command/target_compile_features.html
- `cmake-compile-features(7)` — https://cmake.org/cmake/help/latest/manual/cmake-compile-features.7.html

---

## 13. Generator Expressions

### Definitions

**Core Definition**
Generator expressions are CMake's mechanism for embedding configuration-specific, target-specific, or platform-specific logic directly into target properties, evaluated during build system generation.

**Technical Definition**
Generator expressions have the form `$<...>` and are evaluated by CMake during the generation phase, after all `CMakeLists.txt` files have been processed. They allow conditional linking, conditional compile definitions, conditional include directories, and more. The conditions may be based on build configuration (`Debug`/`Release`), target properties, platform information, or compiler ID. Generator expressions can be nested. They are permitted in the values of many target properties, including `LINK_LIBRARIES`, `INCLUDE_DIRECTORIES`, `COMPILE_DEFINITIONS`, `COMPILE_OPTIONS`, and `COMPILE_FEATURES`.

**Beginner-Friendly Explanation**
Generator expressions are like "if-then-else" statements embedded inside your CMake commands. They let you say "if the compiler is MSVC, use these flags; otherwise use those flags" or "use this include path during the build, but that include path after installation."

### Purposes

- To apply configuration-specific settings (Debug vs. Release).
- To handle compiler-specific and platform-specific differences.
- To specify different paths for build-tree and install-tree usage.
- To enable conditional linking and compilation.
- To avoid deeply nested `if()` statements in `CMakeLists.txt`.

### Syntax Rules and Structure

**General Syntax**
```cmake
$<condition:value>
$<IF:condition,true_value,false_value>
```

**Component Breakdown**

| Expression | Description |
|------------|-------------|
| `$<CXX_COMPILER_ID>` | Expands to the compiler ID (e.g., `GNU`, `Clang`, `MSVC`). |
| `$<CONFIG>` | Expands to the build configuration (e.g., `Debug`, `Release`). |
| `$<BUILD_INTERFACE:...>` | Content used only when building the target. |
| `$<INSTALL_INTERFACE:...>` | Content used only when the target is installed. |
| `$<TARGET_PROPERTY:tgt,prop>` | Value of a target property. |
| `$<TARGET_OBJECTS:objlib>` | Object files from an object library. |

**Syntax Rules**
- Expressions are evaluated during generation, not configuration.
- They can be nested, as in `$<$<CXX_COMPILER_ID:MSVC>:/W4>`.
- Expressions containing spaces or semicolons must be quoted.
- Not all properties support generator expressions; check the documentation for each property.

**Constraints and Limitations**
- Debugging generator expressions can be difficult; use `$<TARGET_GENEX_EVAL:tgt,...>` or `file(GENERATE)` for inspection.
- Overuse can make `CMakeLists.txt` hard to read.

### Annotated Code Examples

**Example 1: Compiler-specific compile options**

```cmake
add_library(mylib src/lib.cpp)
target_compile_options(mylib PRIVATE
    $<$<CXX_COMPILER_ID:MSVC>:/W4 /WX>
    $<$<NOT:$<CXX_COMPILER_ID:MSVC>>:-Wall -Wextra -Werror>
)
```

**Expected Outcome**  
MSVC receives `/W4 /WX`; other compilers receive `-Wall -Wextra -Werror`.

**Why this works**  
`$<CXX_COMPILER_ID:...>` evaluates to true or false based on the compiler; the boolean expression selects the flags.

**Example 2: Build vs. install interface**

```cmake
target_include_directories(mylib PUBLIC
    $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
    $<INSTALL_INTERFACE:include>
)
```

**Expected Outcome**  
During the build, the source-tree include path is used; after installation, the installed include path is used.

**Why this works**  
`BUILD_INTERFACE` and `INSTALL_INTERFACE` select content based on the usage context.

### Real-World Cases

- **Cross-platform libraries** — compiler-specific flags and platform-specific paths.
- **Multi-configuration generators** — Debug/Release-specific settings.
- **Installable packages** — correct include paths for build-tree and install-tree consumers.

### References

- `cmake-generator-expressions(7)` — https://cmake.org/cmake/help/latest/manual/cmake-generator-expressions.7.html

---

## 14. CMake Presets

### Definitions

**Core Definition**
CMake presets are a standardised mechanism, introduced in CMake 3.19, for specifying and sharing common configure, build, and test options via JSON files.

**Technical Definition**
CMake supports two preset files: `CMakePresets.json` (project-wide settings, intended for version control) and `CMakeUserPresets.json` (user-specific local settings, not committed). Both files share the same JSON schema and can define `configurePresets`, `buildPresets`, `testPresets`, and `packagePresets`. Presets can inherit from each other (`inherits`), be hidden (`hidden: true`), and define cache variables, environment variables, generators, binary directories, and more. Presets are versioned; the `version` field is required.

**Beginner-Friendly Explanation**
Presets are a way to save and share common build configurations. Instead of typing long `cmake` command lines, you define a preset once (e.g., "debug build with Ninja") and then run `cmake --preset debug`. It is like a saved configuration profile for your project.

### Purposes

- To standardise and share build configurations across a team.
- To simplify CI pipelines with named presets.
- To reduce command-line complexity.
- To separate project-wide settings from local developer preferences.

### Syntax Rules and Structure

**General Syntax** (`CMakePresets.json`)
```json
{
  "version": 6,
  "configurePresets": [
    {
      "name": "base",
      "hidden": true,
      "binaryDir": "${sourceDir}/build/${presetName}",
      "generator": "Ninja"
    },
    {
      "name": "debug",
      "inherits": "base",
      "cacheVariables": { "CMAKE_BUILD_TYPE": "Debug" }
    }
  ],
  "buildPresets": [
    { "name": "debug", "configurePreset": "debug" }
  ]
}
```

**Component Breakdown**

| Section | Description |
|---------|-------------|
| `version` | Required preset schema version (e.g., 6). |
| `configurePresets` | Configure-time settings (generator, binaryDir, cacheVariables). |
| `buildPresets` | Build-time settings (configuration, targets). |
| `testPresets` | Test-time settings (configuration, output options). |
| `inherits` | Inheritance from another preset. |
| `hidden` | If `true`, the preset is not directly usable but can be inherited. |

**Syntax Rules**
- `version` is mandatory; the schema evolves across CMake versions.
- `CMakePresets.json` should be committed to version control; `CMakeUserPresets.json` should be gitignored.
- Presets can use macros like `${sourceDir}` and `${presetName}`.
- The `$schema` field (preset version 8+) provides editor validation and autocompletion.

**Constraints and Limitations**
- Requires CMake 3.19 or later.
- Preset schema versions are not backward-compatible across all CMake versions.
- User presets cannot override project presets with the same name in conflicting ways.

### Annotated Code Example

```json
{
  "version": 6,
  "configurePresets": [
    {
      "name": "base",
      "hidden": true,
      "binaryDir": "${sourceDir}/build/${presetName}",
      "generator": "Ninja",
      "cacheVariables": {
        "CMAKE_EXPORT_COMPILE_COMMANDS": "ON"
      }
    },
    {
      "name": "debug",
      "inherits": "base",
      "cacheVariables": { "CMAKE_BUILD_TYPE": "Debug" }
    },
    {
      "name": "release",
      "inherits": "base",
      "cacheVariables": { "CMAKE_BUILD_TYPE": "Release" }
    }
  ],
  "buildPresets": [
    { "name": "debug", "configurePreset": "debug" },
    { "name": "release", "configurePreset": "release" }
  ]
}
```

**Expected Outcome**  
Running `cmake --preset debug` configures a Debug build with Ninja in `build/debug`; `cmake --build --preset debug` builds it.

**Why this works**  
`debug` inherits from `base`, adding the Debug build type; the build preset references the configure preset.

### Real-World Cases

- **CI pipelines** — named presets for different build configurations.
- **Team onboarding** — consistent build setup across developers.
- **Multi-platform projects** — presets for Windows, Linux, and macOS.

### References

- `cmake-presets(7)` — https://cmake.org/cmake/help/latest/manual/cmake-presets.7.html
- CMake Presets (blog) — https://cmake.org/cmake/help/latest/manual/cmake-presets.7.html

---

## 15. CMake Best Practices

### Definitions

**Core Definition**
CMake best practices are guidelines for writing modern, maintainable, and correct CMake code that leverages the target-centric model and avoids legacy anti-patterns.

**Technical Definition**
Modern CMake best practices centre on **target-centric configuration**: using `target_*` commands instead of global directory-level commands, specifying visibility (`PRIVATE`, `PUBLIC`, `INTERFACE`) correctly, using generator expressions for conditional settings, preferring `target_compile_features` over `CMAKE_CXX_STANDARD`, and using presets for configuration sharing. The C++ Core Guidelines and the CMake community codify these practices, which improve correctness, modularity, and maintainability.

**Beginner-Friendly Explanation**
Best practices are good habits: configure things on targets (not globally), use `PRIVATE`/`PUBLIC` correctly, prefer `target_*` commands over old-style global variables, and use presets to share build settings.

### Key Guidelines

| Guideline | Summary |
|-----------|---------|
| **Target-centric** | Use `target_*` commands, not `include_directories()` or `link_libraries()`. |
| **Visibility** | `PRIVATE` for implementation details, `PUBLIC` for API dependencies, `INTERFACE` for header-only requirements. |
| **`target_compile_features`** | Use it instead of `set(CMAKE_CXX_STANDARD ...)`. |
| **Generator expressions** | Use `$<...>` for compiler-, platform-, and configuration-specific settings. |
| **Presets** | Share configurations via `CMakePresets.json`. |
| **Avoid globbing** | List sources explicitly; globbing (`file(GLOB ...)`) is discouraged. |
| **Out-of-source builds** | Keep build directories separate from source. |
| **Minimum version** | Set a recent `cmake_minimum_required` and adopt modern practices. |

### Annotated Code Example

```cmake
cmake_minimum_required(VERSION 3.25)
project(myapp VERSION 0.1.0 LANGUAGES CXX)

# Target-centric library configuration
add_library(mylib src/api.cpp src/util.cpp)
target_include_directories(mylib PUBLIC
    $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
    $<INSTALL_INTERFACE:include>
    PRIVATE src
)
target_compile_features(mylib PUBLIC cxx_std_20)
target_compile_options(mylib PRIVATE
    $<$<CXX_COMPILER_ID:MSVC>:/W4 /WX /permissive->
    $<$<NOT:$<CXX_COMPILER_ID:MSVC>>:-Wall -Wextra -Wpedantic -Werror>
)

# Target-centric executable configuration
add_executable(myapp src/main.cpp)
target_link_libraries(myapp PRIVATE mylib)
```

**Expected Outcome**  
A modern, target-centric `CMakeLists.txt` that builds a library and executable with correct visibility, compile features, and compiler-specific warnings.

**Why this works**  
All configuration is attached to targets via `target_*` commands; generator expressions handle compiler-specific flags; visibility keywords control propagation.

### Real-World Cases

- **Open-source libraries** — modern CMake interfaces are expected by consumers.
- **Large codebases** — target-centric configuration scales better than global settings.
- **CI and onboarding** — presets and modern practices simplify setup.

### References

- CMake - Quick Reference (modern CMake) — https://github.com/claude-dev-suite/claude-dev-suite/blob/main/skills/build-tools/cmake/SKILL.md
- Modern CMake best practices (Kitware) — https://www.kitware.eu
- C++ Core Guidelines — https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines

---

## References

- CMake Documentation — https://cmake.org/documentation/
- `cmake-buildsystem(7)` — https://cmake.org/cmake/help/latest/manual/cmake-buildsystem.7.html
- `cmake-generator-expressions(7)` — https://cmake.org/cmake/help/latest/manual/cmake-generator-expressions.7.html
- `cmake-presets(7)` — https://cmake.org/cmake/help/latest/manual/cmake-presets.7.html
- `find_package` — https://cmake.org/cmake/help/latest/command/find_package.html
- `target_link_libraries` — https://cmake.org/cmake/help/latest/command/target_link_libraries.html
- `add_library` — https://cmake.org/cmake/help/latest/command/add_library.html
- `add_executable` — https://cmake.org/cmake/help/latest/command/add_executable.html
- `target_include_directories` — https://cmake.org/cmake/help/latest/command/target_include_directories.html
- `target_compile_options` — https://cmake.org/cmake/help/latest/command/target_compile_options.html
- `target_compile_features` — https://cmake.org/cmake/help/latest/command/target_compile_features.html
- CMake Commands Reference — https://cmake.org/cmake/help/latest/manual/cmake-commands.7.html
- CMake - Quick Reference (modern CMake) — https://github.com/claude-dev-suite/claude-dev-suite/blob/main/skills/build-tools/cmake/SKILL.md
- Modern CMake best practices (Kitware) — https://www.kitware.eu
- C++ Core Guidelines — https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines

---

This cheat sheet provides a comprehensive, structured reference for CMake, covering every requested component individually with definitions, purposes, syntax rules, annotated code examples, real-world cases, and authoritative references.