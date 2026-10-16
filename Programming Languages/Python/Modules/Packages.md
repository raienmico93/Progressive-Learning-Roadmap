# Python Packages: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**

A Python package is a way of organizing related modules into a directory hierarchy. It is a directory that contains one or more module files (`.py` files) and a special `__init__.py` file that tells Python to treat the directory as a package.

**Technical Definition**

In Python, a package is a namespace that contains other packages and modules. A regular package is a directory that contains an `__init__.py` file (which can be empty or contain initialization code) and can contain subpackages (subdirectories with their own `__init__.py`) and modules (`.py` files). Packages allow for a hierarchical structuring of the module namespace using dotted module names (e.g., `package.subpackage.module`). Since Python 3.3 (PEP 420), namespace packages allow a package to be split across multiple directories on disk without requiring an `__init__.py` file.

**Beginner-Friendly Explanation**

A Python package is like a folder that contains a collection of related Python files (modules). For example, you could have a package called `shapes` that contains files for `circle.py`, `square.py`, and `triangle.py`. The special `__init__.py` file tells Python, "This folder is a package." Packages help you organize your code into logical groups, just like you organize files on your computer into folders.

### Key Characteristics

- **Directory-Based**: A package is a directory containing modules and subpackages.
- **`__init__.py`**: Regular packages require an `__init__.py` file to be recognized as packages.
- **Hierarchical**: Packages can contain subpackages, allowing for nested organization.
- **Dotted Names**: Modules in packages are accessed using dotted notation (e.g., `package.module`).
- **Namespace**: Each package has its own namespace, preventing name collisions.
- **Importable**: Packages can be imported just like modules.
- **Namespace Packages**: Since Python 3.3, packages can be split across multiple directories without `__init__.py`.

### Prerequisites

- Basic understanding of Python modules and the `import` statement.
- Familiarity with file system structure and directories.
- Knowledge of Python namespaces and scope.
- Awareness of `sys.path` and module search path.

### Related Programming Areas

- **Module System**: Packages are built on top of Python's module system.
- **Distribution**: Packages are the unit of distribution on PyPI (Python Package Index).
- **Virtual Environments**: Isolated environments for managing package dependencies.
- **Setuptools/pip**: Tools for building, installing, and distributing packages.
- **Import System**: The machinery that handles package imports.

---

## Core Concepts / Features

---

### 1. Packages and Package Creation

#### Definitions

**Core Definition**

A package is a directory containing Python modules and a special `__init__.py` file. Package creation is the process of organizing related modules into such a directory structure.

**Technical Definition**

A regular package is created by making a directory with a valid Python identifier name and placing an `__init__.py` file inside it. The directory may also contain module files (`.py`) and subpackage directories. When the package is imported, Python executes the `__init__.py` file, which can be used to initialize the package, set up package-level variables, or import submodules. The package's `__path__` attribute is a list of directories that Python searches for submodules.

**Beginner-Friendly Explanation**

Creating a package is as simple as creating a folder and putting an `__init__.py` file inside it. You can then add Python files (modules) to the folder. For example, create a folder called `mypackage` with an empty `__init__.py` file and a file called `mymodule.py`. Now you can `import mypackage.mymodule`.

#### Purposes

- To organize related modules into logical groups.
- To prevent naming conflicts by providing a namespace hierarchy.
- To make code reusable and distributable.
- To structure large projects into manageable components.

#### Syntax Rules and Structure

**Complete General Syntax (Package Structure)**

```
my_package/
├── __init__.py
├── module1.py
├── module2.py
└── subpackage/
    ├── __init__.py
    └── submodule.py
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `my_package/` | The package directory. |
| `__init__.py` | Makes the directory a package. Executed on import. |
| `module1.py` | A module within the package. |
| `subpackage/` | A nested subpackage. |
| `subpackage/__init__.py` | Makes the subdirectory a subpackage. |

**Syntax Rules**

- The package directory name must be a valid Python identifier.
- The `__init__.py` file can be empty or contain initialization code.
- Subpackages are created by placing subdirectories with their own `__init__.py` inside the package.
- Package names should be short, lowercase, and use underscores if needed (PEP 8).

**Constraints and Limitations**

- Without `__init__.py`, Python 3.3+ may treat the directory as a namespace package.
- Circular imports between packages can cause `ImportError`.
- Package names should not conflict with standard library modules.

#### Annotated Code Examples

**Example 1: Creating a Simple Package**

```
# Directory structure:
# mypackage/
# ├── __init__.py
# ├── math_utils.py
# └── string_utils.py
```

```python
# File: mypackage/__init__.py
"""A package for utility functions."""
print("Initializing mypackage")
```

```python
# File: mypackage/math_utils.py
def add(a, b):
    return a + b

def subtract(a, b):
    return a - b
```

```python
# File: mypackage/string_utils.py
def reverse(s):
    return s[::-1]

def capitalize(s):
    return s.capitalize()
```

```python
# File: main.py
import mypackage.math_utils
import mypackage.string_utils

print(mypackage.math_utils.add(5, 3))
print(mypackage.string_utils.reverse("hello"))
```

**Expected Output:**

```
Initializing mypackage
8
olleh
```

**Step-by-Step Breakdown:**

1. `mypackage/__init__.py` is executed when the package is imported.
2. `import mypackage.math_utils` loads the `math_utils` module.
3. `mypackage.math_utils.add(5, 3)` calls the `add` function.

**Why the Output is Produced:**

When `mypackage` is imported, Python executes `__init__.py`, printing the initialization message. The `math_utils` and `string_utils` modules are then accessible through the package namespace.

#### Real-World Cases

- **Web Frameworks**: Flask, Django organize code into packages.
- **Data Science**: NumPy, pandas are organized into packages.
- **Standard Library**: `os`, `sys`, `collections` are packages.

---

### 2. `__init__.py`

#### Definitions

**Core Definition**

`__init__.py` is a special file that makes a directory a Python package. It is executed when the package is imported and can contain initialization code.

**Technical Definition**

The `__init__.py` file is required for a directory to be treated as a regular package (before Python 3.3). When a package is imported, Python executes the `__init__.py` file in the package's namespace. This file can be empty, or it can contain code to initialize the package, define package-level variables, or import submodules. The `__all__` variable can be defined in `__init__.py` to control what is imported by `from package import *`.

**Beginner-Friendly Explanation**

`__init__.py` is like a welcome sign for a package. It tells Python, "This folder is a package." You can leave it empty, or you can put code in it that runs when someone imports your package.

#### Purposes

- To mark a directory as a Python package.
- To execute initialization code when the package is imported.
- To define package-level variables and constants.
- To control what is exported by the package using `__all__`.
- To simplify imports by importing submodules into the package namespace.

#### Syntax Rules and Structure

```
# Empty __init__.py (minimal package)
# (file is empty)

# __init__.py with initialization
"""Package docstring."""
print("Package initialized")
VERSION = "1.0.0"

# __init__.py with __all__
__all__ = ["module1", "module2"]

# __init__.py importing submodules
from . import module1
from . import module2
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| Docstring | Optional module docstring. |
| Initialization code | Code executed on import. |
| `VERSION` | Package-level variable. |
| `__all__` | List of names exported by `from package import *`. |
| `from . import module` | Imports submodules into the package namespace. |

**Syntax Rules**

- The file must be named exactly `__init__.py`.
- It can be empty.
- It is executed only once, on first import.
- Relative imports within `__init__.py` are allowed.

**Constraints and Limitations**

- Adding heavy initialization code can slow down imports.
- Importing submodules in `__init__.py` can cause circular imports.
- `__all__` only affects `from package import *`, not other imports.

#### Annotated Code Examples

**Example 1: `__init__.py` with Initialization and `__all__`**

```
# Directory structure:
# mypackage/
# ├── __init__.py
# ├── module1.py
# └── module2.py
```

```python
# File: mypackage/__init__.py
"""My package for demonstration."""
print("Initializing mypackage")
VERSION = "1.0.0"
__all__ = ["module1", "module2"]

from . import module1
from . import module2
```

```python
# File: mypackage/module1.py
def func1():
    return "Function 1"
```

```python
# File: mypackage/module2.py
def func2():
    return "Function 2"
```

```python
# File: main.py
import mypackage

print(mypackage.VERSION)
print(mypackage.module1.func1())
print(mypackage.module2.func2())
```

**Expected Output:**

```
Initializing mypackage
1.0.0
Function 1
Function 2
```

**Step-by-Step Breakdown:**

1. `import mypackage` executes `__init__.py`.
2. `VERSION` is defined at package level.
3. `from . import module1` and `from . import module2` make the submodules accessible.
4. `mypackage.module1.func1()` calls the function.

**Why the Output is Produced:**

The `__init__.py` file initializes the package, defines a package-level variable, and imports the submodules so they are accessible as attributes of the package.

#### Real-World Cases

- **NumPy**: `numpy/__init__.py` imports many submodules and exposes them.
- **Flask**: `flask/__init__.py` defines the `Flask` class and other core objects.
- **Django**: Each app has an `__init__.py` file.

---

### 3. Subpackages

#### Definitions

**Core Definition**

A subpackage is a package nested inside another package. It is a subdirectory of a package that contains its own `__init__.py` file and modules.

**Technical Definition**

Subpackages allow for deeper hierarchical organization of code. A subpackage is created by placing a subdirectory with an `__init__.py` file inside a package directory. Subpackages can themselves contain further subpackages, creating an arbitrarily deep tree. Accessing modules in subpackages requires dotted notation (e.g., `package.subpackage.module`).

**Beginner-Friendly Explanation**

A subpackage is like a folder inside a folder. For example, you could have a `shapes` package with a `polygons` subpackage that contains modules for `triangle.py`, `square.py`, and `pentagon.py`.

#### Purposes

- To organize related modules into finer-grained groups.
- To create deeper namespace hierarchies.
- To avoid name clashes within a package.
- To structure large projects into logical layers.

#### Syntax Rules and Structure

```
parent_package/
├── __init__.py
├── module1.py
└── child_package/
    ├── __init__.py
    ├── module2.py
    └── grandchild_package/
        ├── __init__.py
        └── module3.py
```

**Syntax Rules**

- Each subpackage directory must contain an `__init__.py` file.
- Subpackages are accessed using dotted notation.
- Subpackages can be imported independently.

**Constraints and Limitations**

- Deep nesting can make imports verbose.
- Circular imports between subpackages can be problematic.

#### Annotated Code Examples

**Example 1: Subpackage Structure**

Directory structure:
```
shapes/
  ├── __init__.py
  ├── circle.py
  └── polygons/
      ├── __init__.py
      ├── triangle.py
      └── square.py
```

```python
# File: shapes/__init__.py
"""Shapes package."""
```

```python
# File: shapes/circle.py
import math

def area(radius):
    return math.pi * radius ** 2
```

```python
# File: shapes/polygons/__init__.py
"""Polygons subpackage."""
```

```python
# File: shapes/polygons/triangle.py
def area(base, height):
    return 0.5 * base * height
```

```python
# File: main.py
import shapes.circle
import shapes.polygons.triangle

print(shapes.circle.area(5))
print(shapes.polygons.triangle.area(4, 6))
```

**Expected Output:**

```
78.53981633974483
12.0
```

**Step-by-Step Breakdown:**

1. `shapes` is the parent package.
2. `shapes.circle` is a module in the parent package.
3. `shapes.polygons` is a subpackage.
4. `shapes.polygons.triangle` is a module in the subpackage.

**Why the Output is Produced:**

Subpackages allow deeper organization. The `circle` module is directly in `shapes`, while `triangle` is in the `polygons` subpackage.

#### Real-World Cases

- **Django**: Each app is a subpackage of the project.
- **NumPy**: `numpy.linalg`, `numpy.random` are subpackages.
- **SciPy**: Organized into many subpackages like `scipy.optimize`, `scipy.stats`.

---

### 4. Package Imports: Relative and Absolute

#### Definitions

**Core Definition**

Package imports are the mechanisms for bringing modules and packages into the current namespace. **Absolute imports** specify the full path from the project root; **relative imports** specify the path relative to the current module's package using leading dots.

**Technical Definition**

PEP 328 introduced a clear distinction between absolute and relative imports. Absolute imports use the full path starting from `sys.path` (e.g., `import package.module`). Relative imports use leading dots to indicate the current package (`.`) or parent package (`..`). Relative imports can only be used within a package and must use the `from` form. `import module` is always absolute.

**Beginner-Friendly Explanation**

Absolute imports use the full "address" of a module (like a complete street address). Relative imports use a shortcut that says "the module next door" or "the module one floor up." Relative imports are shorter but only work inside a package.

#### Purposes

- To access modules and packages in a structured way.
- To avoid ambiguity between modules with the same name.
- To make code more portable (absolute imports) or more concise (relative imports).

#### Syntax Rules and Structure

```
# Absolute imports
import package.module
from package.subpackage import module
from package.module import function

# Relative imports (only within packages)
from . import module          # Same package
from .module import function  # Module in same package
from .. import module         # Parent package
from ..subpackage import module  # Sibling subpackage
```

**Component Breakdown**

| Form | Description |
|------|-------------|
| `import package.module` | Absolute import of a module. |
| `from . import module` | Relative import from the same package. |
| `from .. import module` | Relative import from the parent package. |
| `from .module import func` | Import a specific name from a relative module. |

**Syntax Rules**

- Absolute imports are the default and work everywhere.
- Relative imports must use the `from` form.
- Leading dots indicate the level: `.` = current package, `..` = parent, `...` = grandparent.
- Relative imports can only be used inside packages (not in top-level scripts).

**Constraints and Limitations**

- Relative imports do not work in scripts run directly.
- Python 2 required `from __future__ import absolute_import` for absolute imports by default.
- Deep relative imports (`....module`) can become hard to read.

#### Annotated Code Examples

**Example 1: Relative vs Absolute Imports**

```
# Directory structure:
# mypackage/
# ├── __init__.py
# ├── module_a.py
# └── subpackage/
#     ├── __init__.py
#     └── module_b.py
```

```python
# File: mypackage/module_a.py
def hello():
    return "Hello from module_a"
```

```python
# File: mypackage/subpackage/module_b.py
# Relative import (same package)
from . import module_c  # would need module_c.py
# Absolute import
from mypackage import module_a

def greet():
    return module_a.hello()
```

```python
# File: main.py
from mypackage.subpackage import module_b

print(module_b.greet())
```

**Expected Output:**

```
Hello from module_a
```

**Step-by-Step Breakdown:**

1. `module_b.py` uses an absolute import `from mypackage import module_a`.
2. `module_b.greet()` calls `module_a.hello()`.

**Why the Output is Produced:**

The absolute import resolves `mypackage.module_a` from the project root, making `module_a` accessible.

#### Real-World Cases

- **Absolute Imports**: Most projects use absolute imports for clarity.
- **Relative Imports**: Some projects use relative imports for internal package imports to avoid repeating the package name.
- **PEP 8**: Recommends absolute imports as they are more readable and less error-prone.

---

### 5. Import System and `importlib`

#### Definitions

**Core Definition**

The import system is the machinery that finds, loads, and initializes modules and packages. `importlib` is a standard library package that provides the implementation of the import system and allows programmatic importing.

**Technical Definition**

Python's import system is responsible for locating modules, loading their code, and executing it. `importlib` provides the implementation of the `import` statement in pure Python, exposes the components of the import system for custom importers, and includes modules like `importlib.metadata` and `importlib.resources`. The `importlib.import_module()` function imports a module given its name as a string, supporting both absolute and relative imports.

**Beginner-Friendly Explanation**

`importlib` is a toolbox that lets you import modules dynamically at runtime. Instead of writing `import math` at the top of your file, you can write `importlib.import_module("math")` inside a function and import it only when needed.

#### Purposes

- To import modules programmatically based on runtime conditions.
- To create custom importers and loaders.
- To access package metadata and resources.
- To reload modules (`importlib.reload()`).

#### Syntax Rules and Structure

```
import importlib

# Import a module by name
module = importlib.import_module("module_name")

# Import a relative module
module = importlib.import_module(".module", package="mypackage")

# Reload a module
importlib.reload(module)

# Access metadata
from importlib.metadata import version
print(version("package_name"))
```

**Component Breakdown**

| Function | Description |
|----------|-------------|
| `importlib.import_module(name)` | Imports a module by name. |
| `importlib.reload(module)` | Reloads a previously imported module. |
| `importlib.metadata` | Provides access to package metadata. |
| `importlib.resources` | Provides access to package data files. |

**Syntax Rules**

- `import_module()` accepts absolute or relative names.
- For relative imports, the `package` argument specifies the package context.
- `reload()` requires the module object, not the module name.

**Constraints and Limitations**

- `import_module()` is slower than direct `import` statements.
- `reload()` does not update existing references to old objects.

#### Annotated Code Examples

**Example 1: Dynamic Import with `importlib`**

```python
import importlib

# List of modules to import
module_names = ["math", "os", "sys"]

for name in module_names:
    module = importlib.import_module(name)
    print(f"Imported {name}: {module.__name__}")

# Use the imported module
math_module = importlib.import_module("math")
print(f"sqrt(16) = {math_module.sqrt(16)}")
```

**Expected Output:**

```
Imported math: math
Imported os: os
Imported sys: sys
sqrt(16) = 4.0
```

**Step-by-Step Breakdown:**

1. `importlib.import_module("math")` imports the `math` module.
2. The module object is stored in a variable.
3. Functions from the module can be called.

**Why the Output is Produced:**

`importlib.import_module()` dynamically imports modules by name, allowing flexible runtime imports.

#### Real-World Cases

- **Plugin Systems**: Loading plugins dynamically based on configuration.
- **Testing**: Importing test modules programmatically.
- **CLI Tools**: Dispatching commands to modules based on user input.

---

### 6. Namespace Packages

#### Definitions

**Core Definition**

A namespace package is a package that can be split across multiple directories on disk, without requiring an `__init__.py` file. It allows multiple distributions to contribute subpackages to a shared namespace.

**Technical Definition**

PEP 420 introduced implicit namespace packages in Python 3.3. Unlike regular packages, namespace packages do not require an `__init__.py` file. The import system computes the package's `__path__` by combining portions found across multiple directories in `sys.path`. This allows a single logical package to be spread across multiple separate distributions, which is useful for plugin ecosystems and large projects.

**Beginner-Friendly Explanation**

A namespace package is like a shared folder that can be spread across multiple locations. Instead of one folder containing everything, different people can contribute their own folders to the same package name. For example, `company.plugins` could have plugins from different teams, each in their own directory.

#### Purposes

- To split a single package across multiple distributions.
- To allow multiple teams or projects to contribute to a shared namespace.
- To support plugin architectures where plugins are installed as separate packages.
- To avoid `__init__.py` conflicts in large ecosystems.

#### Syntax Rules and Structure

```
# Directory structure for namespace package:
# No __init__.py in the namespace package directory
project1/
└── company/
    └── plugins/
        └── plugin_a.py

project2/
└── company/
    └── plugins/
        └── plugin_b.py

# Both directories are on sys.path.
# `import company.plugins.plugin_a` and `import company.plugins.plugin_b` work.
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| No `__init__.py` | Namespace packages omit `__init__.py`. |
| `__path__` | Automatically computed by the import system. |
| Portions | Directories that contribute to the namespace package. |

**Syntax Rules**

- Namespace packages do not have `__init__.py`.
- They can span multiple directories on `sys.path`.
- The import system merges the portions into a single namespace.
- Regular packages take precedence over namespace packages.

**Constraints and Limitations**

- Namespace packages cannot have package-level code (no `__init__.py`).
- Mixing regular and namespace packages can be confusing.
- Some tools may not fully support namespace packages.

#### Annotated Code Examples

**Example 1: Creating a Namespace Package**

```
# Directory structure:
# project1/company/plugins/plugin_a.py
# project2/company/plugins/plugin_b.py
# (no __init__.py in any directory)
```

```python
# File: project1/company/plugins/plugin_a.py
def hello_a():
    return "Hello from plugin A"
```

```python
# File: project2/company/plugins/plugin_b.py
def hello_b():
    return "Hello from plugin B"
```

```python
# File: main.py
import sys
sys.path.extend(["project1", "project2"])

import company.plugins.plugin_a
import company.plugins.plugin_b

print(company.plugins.plugin_a.hello_a())
print(company.plugins.plugin_b.hello_b())
```

**Expected Output:**

```
Hello from plugin A
Hello from plugin B
```

**Step-by-Step Breakdown:**

1. Both `project1` and `project2` are added to `sys.path`.
2. `company.plugins` is a namespace package spanning both directories.
3. `plugin_a` is found in `project1`; `plugin_b` is found in `project2`.

**Why the Output is Produced:**

The namespace package merges the portions from both directories, allowing modules from different locations to be imported under the same package name.

#### Real-World Cases

- **Plugin Ecosystems**: Multiple packages contribute to a shared namespace.
- **Enterprise Projects**: Different teams contribute to a common namespace.
- **Large Libraries**: Splitting a large library across multiple distributions.

---

### 7. Package Structure and Best Practices

#### Definitions

**Core Definition**

Package structure refers to the organization of files and directories within a Python project. Best practices are guidelines for creating maintainable, distributable packages.

**Technical Definition**

Package structure best practices encompass the layout of source code, tests, configuration files, and documentation. The `src/` layout places importable packages inside a `src/` directory, which prevents accidental imports from the project root and forces proper installation for testing. PEP 8 provides naming conventions for packages (short, lowercase, underscores if needed). The `__all__` variable controls wildcard imports. Proper structure includes a `pyproject.toml` for build configuration, a `README`, a `LICENSE`, and a `tests/` directory.

**Beginner-Friendly Explanation**

A good package structure is like a well-organized filing cabinet. Your code goes in one folder (`src/`), your tests in another (`tests/`), and your documentation and configuration files in the top-level directory. This makes your project easy to understand and share.

#### Purposes

- To make projects easy to navigate and understand.
- To prevent import errors and circular dependencies.
- To facilitate packaging and distribution.
- To support testing and development workflows.

#### Syntax Rules and Structure

**Recommended Project Structure (src layout)**

```
my_project/
├── LICENSE
├── README.md
├── pyproject.toml
├── src/
│   └── my_package/
│       ├── __init__.py
│       ├── module1.py
│       └── subpackage/
│           ├── __init__.py
│           └── module2.py
└── tests/
    ├── __init__.py
    └── test_module1.py
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `src/` | Contains the importable package(s). |
| `my_package/` | The package directory (matches project name). |
| `__init__.py` | Marks the directory as a package. |
| `pyproject.toml` | Build configuration file. |
| `tests/` | Test files. |
| `README.md` | Project documentation. |
| `LICENSE` | License file. |

**Best Practices Table**

| Practice | Description |
|----------|-------------|
| Use `src/` layout | Prevents accidental imports from the project root. |
| Match package name to project name | Simplifies configuration and usage. |
| Include `__init__.py` | Even if empty, it marks the directory as a package. |
| Define `__all__` | Control what `from package import *` exports. |
| Keep `__init__.py` minimal | Avoid heavy initialization logic. |
| Use absolute imports | More readable and less error-prone. |
| Group tests separately | Keep test code out of the package. |
| Include `pyproject.toml` | Modern build configuration. |
| Document with README | Explain the package's purpose and usage. |
| Use virtual environments | Isolate dependencies. |

**Constraints and Limitations**

- `src/` layout requires installing the package for testing.
- Overly deep nesting can make imports verbose.
- Namespace packages do not use `__init__.py`.

#### Annotated Code Examples

**Example 1: Well-Structured Package**

```
# Project structure:
# myproject/
# ├── pyproject.toml
# ├── README.md
# ├── src/
# │   └── myproject/
# │       ├── __init__.py
# │       ├── math_utils.py
# │       └── string_utils.py
# └── tests/
#     └── test_math_utils.py
```

```python
# File: src/myproject/__init__.py
"""MyProject: A sample package."""
__version__ = "0.1.0"
__all__ = ["math_utils", "string_utils"]
```

```python
# File: src/myproject/math_utils.py
def add(a, b):
    return a + b
```

```python
# File: src/myproject/string_utils.py
def reverse(s):
    return s[::-1]
```

```python
# File: main.py
from myproject import math_utils, string_utils

print(math_utils.add(3, 5))
print(string_utils.reverse("hello"))
```

**Expected Output:**

```
8
olleh
```

**Step-by-Step Breakdown:**

1. The `src/` layout places the package inside `src/`.
2. `__init__.py` defines the package version and `__all__`.
3. Modules are imported using absolute imports.

**Why the Output is Produced:**

The well-structured package makes imports clean and predictable.

#### Real-World Cases

- **Open-Source Projects**: Follow the src layout for distribution on PyPI.
- **Team Projects**: Consistent structure improves collaboration.
- **Libraries**: Well-structured packages are easier for users to adopt.

---

## References

- Python Official Documentation – Packages (Tutorial) - https://docs.python.org/3/tutorial/modules.html#packages
- Python Official Documentation – The import system (Language Reference) - https://docs.python.org/3/reference/import.html
- Python Official Documentation – `importlib` — The implementation of import - https://docs.python.org/3/library/importlib.html
- PEP 328 – Imports: Multi-Line and Absolute/Relative - https://peps.python.org/pep-0328/
- PEP 420 – Implicit Namespace Packages - https://peps.python.org/pep-0420/
- PEP 8 – Style Guide for Python Code - https://peps.python.org/pep-0008/
- Python Packaging User Guide – Packaging Python Projects - https://packaging.python.org/en/latest/tutorials/packaging-projects/
- Python Packaging User Guide – src layout vs flat layout - https://packaging.python.org/en/latest/discussions/src-layout-vs-flat-layout/
- Real Python – Python Modules and Packages – An Introduction - https://realpython.com/python-modules-packages/
- Real Python – Python Packages: A Comprehensive Guide - https://realpython.com/python-packages/
- Real Python – Python Namespace Packages - https://realpython.com/python-namespace-package/
- Real Python – `importlib` — Python Standard Library - https://realpython.com/python-importlib/
- Real Python – Relative Imports in Python - https://realpython.com/absolute-vs-relative-python-imports/
- PyPI – etude-structure-package - https://pypi.org/project/etude-structure-package/
- GeeksforGeeks – Python Packages - https://www.geeksforgeeks.org/python-packages/
- Microsoft Learn – Python Packages - https://learn.microsoft.com/en-us/training/modules/python-packages/