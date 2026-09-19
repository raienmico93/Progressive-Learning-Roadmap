# Python Modules and the Import System: A Comprehensive Programming Cheat Sheet

## Topic Overview

### Definitions

**Core Definition**
A module is a file containing Python definitions and statements that can be imported and reused by other Python programs; the import system is the mechanism by which Python locates, loads, and binds modules into a namespace.

**Technical Definition**
A module is an object of type `module` that serves as a logical unit of Python code organisation. The import system is a fully exposed, hookable machinery that implements the two phases of PEP 302: (1) finding and loading a module via meta path finders and path entry finders, and (2) binding the resulting module object into the importing namespace. The system maintains caches (`sys.modules`, `sys.path_importer_cache`) and supports programmatic invocation through `importlib.import_module()` and the built-in `__import__()`. Since Python 3.3, there is no implicit import machinery — the full system is exposed through `sys.meta_path`.

**Beginner-Friendly Explanation**
A module is just a Python file (ending in `.py`) that you can reuse in other programs. Instead of copying and pasting the same functions into every script, you write them once in a module and `import` that module wherever you need it. The import system is Python's way of finding that file, running its code (once), and making its contents available to your program. It's like a library system: you request a book (module) by name, and the librarian (import system) finds it, checks if it's already on your desk (cached), and brings it to you.

### Key Characteristics

| Characteristic | Description |
|---|---|
| **File-Based** | Any `.py` file is a module; packages are directories with `__init__.py` |
| **Executed Once** | A module's top-level code runs only on first import |
| **Cached** | Imported modules are stored in `sys.modules` for fast subsequent access |
| **Namespace Isolation** | Each module has its own global namespace |
| **Searchable via `sys.path`** | Python searches a configurable list of directories for modules |
| **Programmatically Importable** | `importlib` enables dynamic, runtime imports |
| **Customisable** | Meta path finders and module-level `__getattr__` allow advanced control |

### Prerequisites

- **Basic Python syntax**: variables, functions, classes, and statements
- **Understanding of files and directories**: how to create and run `.py` files
- **Familiarity with the Python interpreter**: running scripts or using the REPL
- **Basic concepts of namespaces**: understanding global and local scope

### Related Programming Areas

- **Packages**: Directories containing multiple modules, with `__init__.py` defining the package interface
- **Virtual Environments**: Isolated Python environments with their own `site-packages`
- **Dependency Management**: `pip`, `venv`, and `pyproject.toml` for installing and distributing modules
- **Plugin Architectures**: Dynamic importing for extensible applications
- **Namespace Packages**: PEP 420 packages without `__init__.py`
- **Import Hooks**: Custom finders and loaders for specialised import behaviour

### Core Concepts / Features

1. **Module Creation** (Writing reusable Python files)
2. **Importing Modules** (`import`, `from ... import`, aliases)
3. **The Execution Block** (`if __name__ == "__main__":`)
4. **How Python Finds Modules** (`sys.path`, `sys.modules`, search order)
5. **Dynamic Importing** (`importlib` module)
6. **Module-Level `__getattr__` and `__dir__`** (PEP 562)


## Core Concept 1: Module Creation

### Definitions

**Core Definition**
Module creation is the process of writing a Python file (`.py`) that defines functions, classes, variables, and statements intended for reuse by other programs.

**Technical Definition**
A module is a file containing Python definitions and statements. The file name is the module name with the suffix `.py` appended. Within a module, the module's name (as a string) is available as the value of the global variable `__name__`. Modules can define functions, classes, and variables, and can also include runnable code. When a module is imported, its top-level code executes once, and its namespace is populated with the defined names.

**Beginner-Friendly Explanation**
Creating a module is as simple as writing a Python file. If you have functions you want to reuse, put them in a `.py` file (e.g., `my_helpers.py`), and then other Python files can import and use them. It's like saving a recipe in a cookbook so you don't have to remember it every time.

### Purposes

- **To organise code** into logical, reusable units
- **To avoid code duplication** by writing functions and classes once
- **To improve maintainability** by separating concerns across files
- **To enable collaboration** by letting different developers work on different modules
- **To create libraries and frameworks** that others can install and use

### Syntax Rules and Structure

**Complete General Syntax**

```
# my_module.py

"""Module docstring."""

# Module-level variables
CONSTANT = 42

# Function definitions
def my_function(arg):
    return arg * 2

# Class definitions
class MyClass:
    def __init__(self):
        self.value = 10

# Runnable code (executes on import)
print("Module loaded")
```

**Breakdown:**
- The file must have a `.py` extension.
- Module-level code (outside functions/classes) executes on first import.
- `__name__` is set to the module name during import.
- Docstrings document the module's purpose.

**Syntax Rules**

| Rule | Description |
|---|---|
| `.py` extension | Module files must end with `.py` |
| Valid identifier name | File name becomes the module name |
| Top-level code | Executes once on import |
| `__name__` available | Set to the module name |
| Docstring optional | Recommended for documentation |

**Constraints and Limitations**

- **Module names must be valid identifiers**: Hyphens, spaces, and special characters are not allowed.
- **Top-level code runs on import**: Side effects should be avoided or guarded with `if __name__ == "__main__"`.
- **No circular imports**: Modules that import each other can cause `ImportError`.

### Multiple Annotated Complete Code Examples and Their Expected Outputs

**Example 1: Creating and Using a Simple Module**

```python
# File: math_helpers.py
"""A simple module with math helper functions."""

def add(a, b):
    """Return the sum of a and b."""
    return a + b

def multiply(a, b):
    """Return the product of a and b."""
    return a * b

PI = 3.14159
```

```python
# File: main.py
import math_helpers

result1 = math_helpers.add(3, 5)
result2 = math_helpers.multiply(4, 7)
print(f"Add: {result1}")
print(f"Multiply: {result2}")
print(f"PI: {math_helpers.PI}")
```

**Expected Output:**
```
Add: 8
Multiply: 28
PI: 3.14159
```

**Breakdown:** `math_helpers.py` defines functions and a variable. `main.py` imports the module and accesses its contents using dot notation.

**Example 2: Module with Runnable Code**

```python
# File: greet.py
"""A module that greets on import."""

def greet(name):
    return f"Hello, {name}!"

print("Greet module loaded")  # This runs on import
```

```python
# File: app.py
import greet

print(greet.greet("Alice"))
```

**Expected Output:**
```
Greet module loaded
Hello, Alice!
```

**Breakdown:** The `print` statement in `greet.py` runs when the module is imported. The function is then called from `app.py`.

**Example 3: Module with `__name__` Check**

```python
# File: calculator.py
"""A calculator module with a self-test."""

def add(a, b):
    return a + b

if __name__ == "__main__":
    # This only runs when the module is executed directly
    print("Running self-test...")
    assert add(2, 3) == 5
    print("Self-test passed!")
```

```bash
# Running directly
python calculator.py
```

**Expected Output:**
```
Running self-test...
Self-test passed!
```

```python
# Importing in another script
import calculator
# No output from the self-test
```

**Breakdown:** The `if __name__ == "__main__":` guard prevents the self-test from running on import. When executed directly, `__name__` is `"__main__"`, so the test runs.

### Real-World Cases with Explanation

**Case 1: Utility Modules**

```python
# File: string_utils.py
def slugify(text):
    """Convert text to a URL-friendly slug."""
    return text.lower().replace(" ", "-")

def truncate(text, length):
    """Truncate text to a maximum length."""
    return text[:length] + "..." if len(text) > length else text
```

**Why it matters:** Utility modules group related helper functions, making them reusable across projects.

**Case 2: Configuration Modules**

```python
# File: config.py
DATABASE_URL = "postgresql://localhost/mydb"
DEBUG = True
SECRET_KEY = "change-me"
```

**Why it matters:** Configuration modules centralise settings, making them easy to change without editing application logic.


## Core Concept 2: Importing Modules

### Definitions

**Core Definition**
Importing is the process of making a module's definitions and statements available in another module's namespace using the `import` statement.

**Technical Definition**
The `import` statement combines two operations: it searches for the named module, then it binds the results of that search to a name in the local scope. The search operation is defined as a call to the `__import__()` function. The return value of `__import__()` is used to perform the name binding operation. A direct call to `__import__()` performs only the module search and, if found, the module creation operation — only the `import` statement performs the name binding. Python supports several import forms: `import module`, `from module import name`, `import module as alias`, and `from module import name as alias`. The `as` clause was introduced in Python 2.0 via PEP 221.

**Beginner-Friendly Explanation**
Importing is how you bring code from one file into another. You can import the whole module (`import math`), import specific things from it (`from math import sqrt`), or give it a shorter name (`import math as m`). Once imported, you use the module's contents with dot notation or directly, depending on the import style.

### Purposes

- **To reuse code** across multiple files without duplication
- **To access standard library and third-party modules**
- **To organise code** into logical units that can be imported selectively
- **To avoid naming conflicts** with aliases
- **To control exactly which names** are brought into the current namespace

### Syntax Rules and Structure

**Complete General Syntax**

```
# Import the whole module
import module_name

# Import with an alias
import module_name as alias

# Import specific names from a module
from module_name import name1, name2

# Import specific names with aliases
from module_name import name1 as alias1, name2 as alias2

# Import all public names (discouraged)
from module_name import *
```

**Breakdown:**
- `import module`: Binds the module object to the name `module`.
- `import module as alias`: Binds the module object to `alias`.
- `from module import name`: Binds the name directly in the current namespace.
- `from module import *`: Imports all names listed in `__all__` (or all names not starting with `_`).

**Import Rules**

| Form | What's Bound | Access Style |
|---|---|---|
| `import math` | `math` (module object) | `math.sqrt(4)` |
| `import math as m` | `m` (module object) | `m.sqrt(4)` |
| `from math import sqrt` | `sqrt` (function) | `sqrt(4)` |
| `from math import sqrt as s` | `s` (function) | `s(4)` |
| `from math import *` | All public names | `sqrt(4)` |

**Constraints and Limitations**

- **`from module import *` is discouraged**: It pollutes the namespace and makes it unclear where names come from.
- **Aliases must be valid identifiers**: Cannot use keywords or invalid names.
- **Circular imports**: Modules that import each other can cause `ImportError` or partially initialised modules.
- **Name collisions**: Importing the same name from different modules overwrites earlier imports.

### Multiple Annotated Complete Code Examples and Their Expected Outputs

**Example 1: Basic Import Forms**

```python
# Step 1: Import the whole module
import math
print(math.sqrt(16))

# Step 2: Import with alias
import math as m
print(m.pi)

# Step 3: Import specific names
from math import sqrt, pi
print(sqrt(25))
print(pi)

# Step 4: Import with alias for a specific name
from math import sqrt as square_root
print(square_root(36))
```

**Expected Output:**
```
4.0
3.141592653589793
5.0
3.141592653589793
6.0
```

**Breakdown:** Each import form binds different names. `import math` requires `math.` prefix; `from math import sqrt` allows direct use.

**Example 2: Importing from a Custom Module**

```python
# File: shapes.py
def area_circle(radius):
    return 3.14159 * radius ** 2

def area_rectangle(width, height):
    return width * height
```

```python
# File: app.py
# Import the whole module
import shapes
print(shapes.area_circle(5))

# Import specific functions
from shapes import area_rectangle
print(area_rectangle(4, 6))

# Import with alias
from shapes import area_circle as circle_area
print(circle_area(3))
```

**Expected Output:**
```
78.53975
24
28.27431
```

**Breakdown:** The custom module `shapes.py` is imported using three different styles, each producing the same underlying functionality.

**Example 3: `__all__` and `from module import *`**

```python
# File: my_module.py
__all__ = ["public_func", "PublicClass"]

def public_func():
    return "public"

def _private_func():
    return "private"

class PublicClass:
    pass

class _PrivateClass:
    pass
```

```python
# File: app.py
from my_module import *

print(public_func())      # Works
print(PublicClass)        # Works
# print(_private_func())  # NameError — not exported
# print(_PrivateClass)    # NameError — not exported
```

**Expected Output:**
```
public
<class 'my_module.PublicClass'>
```

**Breakdown:** `__all__` explicitly defines which names are exported by `from module import *`. Private names (starting with `_`) are excluded.

### Real-World Cases with Explanation

**Case 1: Standard Library Imports**

```python
import os
import sys
from datetime import datetime, timedelta
from collections import defaultdict, Counter
```

**Why it matters:** Standard library modules are imported using the same syntax; understanding import styles helps navigate Python's extensive standard library.

**Case 2: Third-Party Libraries**

```python
import numpy as np
import pandas as pd
from sklearn.model_selection import train_test_split
```

**Why it matters:** Aliases are commonly used for long library names (e.g., `numpy` → `np`), reducing verbosity while maintaining clarity.


## Core Concept 3: The Execution Block (`if __name__ == "__main__":`)

### Definitions

**Core Definition**
The `if __name__ == "__main__":` idiom is a conditional block that executes only when a Python file is run directly as a script, not when it is imported as a module.

**Technical Definition**
When Python runs a script, it sets the special `__name__` variable to `"__main__"` in the top-level code environment. When a module is imported, `__name__` is set to the module's name. The idiom `if __name__ == "__main__":` checks this value, allowing a file to serve as both a reusable module and a standalone program. Code nested under the idiom runs when the file is executed as a script, but Python skips it on import.

**Beginner-Friendly Explanation**
This trick lets a Python file have two jobs: it can be used as a library (imported by other files) and as a program (run directly). The code inside the `if` block only runs when you run the file yourself, not when another file imports it. It's like a book that can be read cover-to-cover or used as a reference — the "read the whole story" part only happens when you read it as a book.

### Purposes

- **To make a file both a module and a script**
- **To provide a command-line entry point** for a module
- **To include self-tests** that run only when executed directly
- **To prevent side effects** from running on import
- **To clarify the intended usage** of a file

### Syntax Rules and Structure

**Complete General Syntax**

```python
# Module code (functions, classes, variables)
def my_function():
    ...

if __name__ == "__main__":
    # Code that runs only when executed directly
    main()
```

**Breakdown:**
- `__name__`: Special variable set by Python.
- `"__main__"`: The value when the file is the entry point.
- The `if` block contains code that should not run on import.

**Execution Rules**

| Scenario | `__name__` Value | Block Runs? |
|---|---|---|
| `python script.py` | `"__main__"` | Yes |
| `import script` | `"script"` | No |
| `python -m module` | `"__main__"` | Yes |

**Constraints and Limitations**

- **Not required**: Simple scripts that are never imported don't need it.
- **Common but not mandatory**: Many production scripts use it; some small scripts skip it.
- **Naming convention**: The guarded function is often called `main()` by convention.

### Multiple Annotated Complete Code Examples and Their Expected Outputs

**Example 1: Basic Execution Block**

```python
# File: echo.py
def echo(text, repetitions=3):
    """Imitate a real-world echo."""
    echoes = [text[-i:].lower() for i in range(repetitions, 0, -1)]
    return "\n".join(echoes + ["."])

if __name__ == "__main__":
    text = input("Yell something at a mountain: ")
    print(echo(text))
```

```bash
$ python echo.py
Yell something at a mountain: HELLOOOO
echoooooooo
ooo
oo
.
```

**Expected Output:**
```
echoooooooo
ooo
oo
.
```

**Breakdown:** The `input()` and `print()` calls run only when `echo.py` is executed directly. When imported, only the `echo()` function is available.

**Example 2: Module with Self-Test**

```python
# File: math_utils.py
def add(a, b):
    return a + b

def subtract(a, b):
    return a - b

if __name__ == "__main__":
    # Self-test
    assert add(2, 3) == 5, "add() failed"
    assert subtract(5, 3) == 2, "subtract() failed"
    print("All tests passed!")
```

```bash
$ python math_utils.py
All tests passed!
```

```python
# Importing in another file — no test output
import math_utils
result = math_utils.add(10, 20)
print(result)  # 30
```

**Expected Output:**
```
30
```

**Breakdown:** The self-test runs only when `math_utils.py` is executed directly. When imported, the test is skipped, and only the functions are used.

**Example 3: Multiple Entry Points**

```python
# File: cli.py
import sys

def process_file(filename):
    with open(filename) as f:
        return f.read()

def main():
    if len(sys.argv) < 2:
        print("Usage: python cli.py <filename>")
        return
    content = process_file(sys.argv[1])
    print(f"File has {len(content)} characters")

if __name__ == "__main__":
    main()
```

**Expected Output:**
```
$ python cli.py data.txt
File has 1234 characters
```

**Breakdown:** `main()` encapsulates the CLI logic and is called only when the file is executed as a script. The `process_file()` function is reusable when imported.

### Real-World Cases with Explanation

**Case 1: Library with CLI**

```python
# library.py — reusable functions + CLI
def transform(data):
    return data.upper()

if __name__ == "__main__":
    import sys
    print(transform(sys.stdin.read()))
```

**Why it matters:** The same file can be used as a library (`import library`) and as a command-line tool (`python library.py`).

**Case 2: Test Runner**

```python
# module.py
def calculate(x):
    return x * 2

if __name__ == "__main__":
    import unittest
    class TestCalculate(unittest.TestCase):
        def test_calculate(self):
            self.assertEqual(calculate(5), 10)
    unittest.main()
```

**Why it matters:** Embedding tests in the execution block keeps tests close to the code while preventing them from running on import.


## Core Concept 4: How Python Finds Modules

### Definitions

**Core Definition**
Python finds modules by searching a configurable list of locations, caching results, and consulting meta path finders — the process is governed by `sys.path`, `sys.modules`, and `sys.meta_path`.

**Technical Definition**
When a module is first imported, Python searches for it through a well-defined sequence: (1) it checks `sys.modules`, the cache of previously imported modules; (2) it queries `sys.meta_path` finders in order; (3) path-based finders search `sys.path` entries — directories and zip files — left to right. The search order determines which module is found when multiple candidates exist. `sys.path` is initialised from the script directory, `PYTHONPATH`, and installation-dependent defaults. If the named module cannot be found, `ModuleNotFoundError` is raised.

**Beginner-Friendly Explanation**
When you type `import my_module`, Python looks for it in several places, in order: first, it checks if it's already been imported (cached). If not, it looks in a list of directories stored in `sys.path` — starting with the directory of the script you're running, then environment variables, then standard library locations. If it finds the file, it loads it. If not, you get a `ModuleNotFoundError`. Understanding this order helps you debug import errors.

### Purposes

- **To locate modules** in a predictable, configurable manner
- **To cache imported modules** for fast subsequent access
- **To allow customisation** of module search paths
- **To support debugging** of `ModuleNotFoundError`
- **To enable virtual environments** with isolated module sets

### Syntax Rules and Structure

**Search Order**

```
1. sys.modules (cache)           # Fastest — already imported
2. sys.meta_path finders         # Built-in and custom finders
3. sys.path entries              # Directories and zip files
   a. Script directory            # Directory of the running script
   b. PYTHONPATH                  # Environment variable
   c. Standard library            # Python installation
   d. site-packages               # Third-party packages
```

**Key Structures**

| Structure | Description |
|---|---|
| `sys.modules` | Dict mapping module names to module objects |
| `sys.path` | List of directories Python searches for modules |
| `sys.meta_path` | List of finder objects queried in order |
| `sys.path_importer_cache` | Cache of path finders for directories |

**Constraints and Limitations**

- **Script directory first**: The script's directory is prepended to `sys.path`, which can shadow standard library modules.
- **`sys.path` is mutable**: You can add directories at runtime, but this is fragile.
- **Module name collisions**: A local file named `math.py` shadows the standard library `math` module.

### Multiple Annotated Complete Code Examples and Their Expected Outputs

**Example 1: Inspecting `sys.path`**

```python
import sys

# Step 1: Print the module search path
for i, path in enumerate(sys.path):
    print(f"{i}: {path}")

# Step 2: Print the first few entries (script directory, PYTHONPATH, etc.)
print(f"\nFirst entry (script dir): {sys.path[0]}")
```

**Expected Output:**
```
0: /home/user/projects/myapp
1: /home/user/.local/lib/python3.12/site-packages
2: /usr/lib/python3.12
...
First entry (script dir): /home/user/projects/myapp
```

**Breakdown:** `sys.path[0]` is the directory of the script being run. Subsequent entries include user site-packages, standard library, and system paths.

**Example 2: Checking `sys.modules` Cache**

```python
import sys

# Step 1: Before importing, check if 'math' is cached
print(f"'math' in sys.modules: {'math' in sys.modules}")

# Step 2: Import math
import math

# Step 3: After importing, check again
print(f"'math' in sys.modules: {'math' in sys.modules}")
print(f"Module object: {sys.modules['math']}")
```

**Expected Output:**
```
'math' in sys.modules: False
'math' in sys.modules: True
Module object: <module 'math' (built-in)>
```

**Breakdown:** `sys.modules` caches imported modules. Before importing, `math` is absent; after, it's present.

**Example 3: Debugging `ModuleNotFoundError`**

```python
import sys

# Step 1: Attempt to import a missing module
try:
    import my_custom_module
except ModuleNotFoundError as e:
    print(f"Error: {e}")
    print(f"Search path: {sys.path}")
    print(f"Module in cache: {'my_custom_module' in sys.modules}")
```

**Expected Output:**
```
Error: No module named 'my_custom_module'
Search path: ['/home/user/project', '/usr/lib/python3.12', ...]
Module in cache: False
```

**Breakdown:** The error message and `sys.path` reveal where Python looked. Adding the module's directory to `sys.path` or placing the file in the script directory resolves the issue.

### Real-World Cases with Explanation

**Case 1: Virtual Environments**

```python
# In a virtual environment, sys.path points to the venv's site-packages
import sys
print(sys.path)
# ['', '/home/user/venv/lib/python3.12/site-packages', ...]
```

**Why it matters:** Virtual environments isolate module installations, ensuring project-specific dependencies don't conflict.

**Case 2: Adding Custom Paths**

```python
import sys
sys.path.append('/path/to/my/modules')
import my_module  # Now findable
```

**Why it matters:** Appending to `sys.path` allows importing modules from non-standard locations, useful for plugins or shared code.


## Core Concept 5: Dynamic Importing (`importlib`)

### Definitions

**Core Definition**
Dynamic importing is the programmatic importation of modules at runtime using the `importlib` module, enabling imports based on conditions or user input.

**Technical Definition**
The `importlib` package provides the implementation of the `import` statement in pure Python and exposes the components of the import system for importing modules programmatically. `importlib.import_module(name, package=None)` imports a module by its string name, returning the module object. Other functions include `importlib.reload()` for re-executing a module's code and `importlib.util.find_spec()` for locating a module's specification. Dynamic importing is essential for plugin architectures, lazy loading, and configuration-driven module selection.

**Beginner-Friendly Explanation**
Normally, you write `import my_module` with the module name fixed in your code. But sometimes you don't know which module to import until the program is running — for example, a plugin system where users choose modules at runtime. `importlib` lets you import modules by name as strings, enabling flexible, dynamic behavior.

### Purposes

- **To import modules based on runtime conditions** (user input, configuration)
- **To implement plugin architectures** where modules are discovered and loaded dynamically
- **To reload modules** during development without restarting the interpreter
- **To inspect module specifications** for introspection and debugging
- **To build custom importers** for specialised module sources

### Syntax Rules and Structure

**Complete General Syntax**

```
import importlib

# Import a module by name
module = importlib.import_module("module_name")

# Import a submodule within a package
submodule = importlib.import_module(".submodule", package="mypackage")

# Reload a previously imported module
importlib.reload(module)

# Find a module's specification
spec = importlib.util.find_spec("module_name")
```

**Breakdown:**
- `import_module(name, package)`: Imports and returns the named module.
- `reload(module)`: Re-executes the module's code and updates its namespace.
- `find_spec(name)`: Returns a `ModuleSpec` object or `None` if not found.

**Dynamic Import Rules**

| Function | Purpose | Returns |
|---|---|---|
| `import_module(name)` | Import by string name | Module object |
| `reload(module)` | Re-execute module code | Module object |
| `find_spec(name)` | Locate module | `ModuleSpec` or `None` |
| `invalidate_caches()` | Clear import caches | `None` |

**Constraints and Limitations**

- **`reload()` does not update existing references**: Objects created from the old module persist.
- **Circular imports with `import_module()`**: Same risks as static imports.
- **Security**: Dynamic imports based on untrusted input can execute arbitrary code.

### Multiple Annotated Complete Code Examples and Their Expected Outputs

**Example 1: Basic Dynamic Import**

```python
import importlib

# Step 1: Import a module by string name
math_module = importlib.import_module("math")

# Step 2: Use the module
result = math_module.sqrt(25)
print(f"Square root: {result}")

# Step 3: Import based on user input
module_name = input("Enter module name: ")
module = importlib.import_module(module_name)
print(f"Loaded: {module.__name__}")
```

**Expected Output:**
```
Square root: 5.0
Enter module name: json
Loaded: json
```

**Breakdown:** `import_module()` imports modules by string name, enabling dynamic selection.

**Example 2: Plugin Architecture**

```python
import importlib

# Step 1: Define available plugins
available_plugins = ["plugin_a", "plugin_b", "plugin_c"]

# Step 2: Load a plugin by name
def load_plugin(name):
    try:
        module = importlib.import_module(f"plugins.{name}")
        if hasattr(module, "run"):
            return module
        else:
            print(f"Plugin {name} has no run() function")
    except ModuleNotFoundError:
        print(f"Plugin {name} not found")
    return None

# Step 3: Use a plugin
plugin = load_plugin("plugin_a")
if plugin:
    plugin.run()
```

**Expected Output:**
```
Plugin plugin_a loaded
Running plugin A
```

**Breakdown:** The plugin loader dynamically imports modules from a `plugins` package, checking for a `run()` function before use.

**Example 3: Reloading a Module**

```python
import importlib
import my_module

# Step 1: Use the module
print(my_module.value)

# Step 2: Modify the source file externally...
# (e.g., change value from 10 to 20)

# Step 3: Reload the module
importlib.reload(my_module)
print(my_module.value)
```

**Expected Output:**
```
10
20
```

**Breakdown:** `reload()` re-executes the module's code, picking up changes made to the source file.

### Real-World Cases with Explanation

**Case 1: Configuration-Driven Imports**

```python
import importlib
config = {"parser": "json"}
parser_module = importlib.import_module(config["parser"])
```

**Why it matters:** Configuration files can specify which modules to load, enabling flexible application behaviour.

**Case 2: Development Reloading**

```python
import importlib
import my_module

# During development, reload after changes
importlib.reload(my_module)
```

**Why it matters:** `reload()` speeds up development by applying code changes without restarting the interpreter.


## Core Concept 6: Module-Level `__getattr__` and `__dir__`

### Definitions

**Core Definition**
Module-level `__getattr__` and `__dir__` are special functions defined in a module that customise attribute access and directory listing, enabling deprecation warnings, lazy loading, and dynamic attributes.

**Technical Definition**
PEP 562 allows modules to define `__getattr__` and `__dir__` functions for basic customization of module attribute access. The `__getattr__` function at the module level accepts one argument — the name of an attribute — and returns the computed value or raises `AttributeError`. It is called when normal attribute lookup fails. The `__dir__` function accepts no arguments and returns a list of strings representing the module's attributes, allowing dynamic attributes to appear in `dir()`. These functions were introduced in Python 3.7.

**Beginner-Friendly Explanation**
Normally, when you access `module.something`, Python looks for `something` in the module's namespace. With `__getattr__`, you can intercept that lookup and provide a value dynamically — for example, showing a deprecation warning when someone accesses an old function name. `__dir__` lets you control what `dir(module)` shows, so deprecated or lazy-loaded attributes appear in listings.

### Purposes

- **To provide deprecation warnings** for renamed or removed attributes
- **To enable lazy loading** of submodules or expensive resources
- **To dynamically generate attributes** based on external data
- **To customise `dir()` output** for introspection tools
- **To implement module-level proxies** or facades

### Syntax Rules and Structure

**Complete General Syntax**

```python
# Module-level __getattr__
def __getattr__(name):
    # Compute and return the attribute value
    # or raise AttributeError
    ...

# Module-level __dir__
def __dir__():
    # Return a list of strings
    return [...]
```

**Breakdown:**
- `__getattr__(name)`: Called when normal lookup fails; must return a value or raise `AttributeError`.
- `__dir__()`: Called by `dir()`; returns a list of attribute names.

**Behavior Rules**

| Rule | Description |
|---|---|
| Called only on failure | `__getattr__` is invoked after normal lookup fails |
| Must raise `AttributeError` | If the attribute doesn't exist, raise to signal failure |
| `__dir__` controls `dir()` | Returns the list shown by `dir(module)` |
| Module globals bypass | Looking up a name as a module global bypasses `__getattr__` (performance) |

**Constraints and Limitations**

- **`__getattr__` is not called for module globals**: Only for failed attribute lookups.
- **`__dir__` must return strings**: Elements must be valid attribute names.
- **Python 3.7+ only**: Not available in earlier versions (backport available).

### Multiple Annotated Complete Code Examples and Their Expected Outputs

**Example 1: Deprecation Warning via `__getattr__`**

```python
# File: lib.py
from warnings import warn

deprecated_names = ["old_function"]

def _deprecated_old_function(arg, other):
    return arg + other

def __getattr__(name):
    if name in deprecated_names:
        warn(f"{name} is deprecated", DeprecationWarning)
        return globals()[f"_deprecated_{name}"]
    raise AttributeError(f"module {__name__!r} has no attribute {name!r}")

def new_function(x):
    return x * 2
```

```python
# File: main.py
from lib import old_function  # Emits DeprecationWarning
result = old_function(3, 4)
print(result)
```

**Expected Output:**
```
7
DeprecationWarning: old_function is deprecated
```

**Breakdown:** Accessing `old_function` triggers `__getattr__`, which emits a warning and returns the deprecated implementation.

**Example 2: Lazy Submodule Loading**

```python
# File: mypackage/__init__.py
import importlib

__all__ = ["submodule"]

def __getattr__(name):
    if name in __all__:
        return importlib.import_module("." + name, __name__)
    raise AttributeError(f"module {__name__!r} has no attribute {name!r}")
```

```python
# File: main.py
import mypackage

# Submodule is loaded lazily on first access
mypackage.submodule  # Prints "Submodule loaded"
```

**Expected Output:**
```
Submodule loaded
```

**Breakdown:** The submodule is imported only when accessed, saving memory and startup time.

**Example 3: Custom `__dir__`**

```python
# File: lib.py
deprecated_names = ["old_function"]
__all__ = ["new_function_one", "new_function_two"]

def new_function_one(arg):
    return arg

def new_function_two(arg):
    return arg

def __dir__():
    return sorted(__all__ + deprecated_names)
```

```python
# File: main.py
import lib
print(dir(lib))
```

**Expected Output:**
```
['new_function_one', 'new_function_two', 'old_function']
```

**Breakdown:** `__dir__` includes deprecated names in the directory listing, making them discoverable while still warning on access.

### Real-World Cases with Explanation

**Case 1: Deprecating Module Attributes**

```python
# lib.py
def __getattr__(name):
    if name == "old_config":
        warn("old_config is deprecated; use config", DeprecationWarning)
        return config
    raise AttributeError(f"module {__name__!r} has no attribute {name!r}")
```

**Why it matters:** Libraries can deprecate attributes gracefully, warning users without breaking existing code immediately.

**Case 2: Lazy Loading Expensive Resources**

```python
# lib.py
def __getattr__(name):
    if name == "large_dataset":
        global large_dataset
        large_dataset = load_large_dataset()  # Expensive
        return large_dataset
    raise AttributeError(...)
```

**Why it matters:** Lazy loading defers expensive operations until actually needed, improving startup time.


## References

- Python Software Foundation. *5. The import system — Python Language Reference*. https://docs.python.org/3/reference/import.html
- Python Software Foundation. *6. Modules — Python Tutorial*. https://docs.python.org/3/tutorial/modules.html
- Python Software Foundation. *PEP 562 – Module `__getattr__` and `__dir__`*. https://peps.python.org/pep-0562/
- Python Software Foundation. *PEP 221 – Import As*. https://peps.python.org/pep-0221/
- Python Software Foundation. *PEP 302 – New Import Hooks*. https://peps.python.org/pep-0302/
- Python Software Foundation. *PEP 420 – Implicit Namespace Packages*. https://peps.python.org/pep-0420/
- Python Software Foundation. *importlib — The implementation of import*. https://docs.python.org/3/library/importlib.html
- Python Software Foundation. *sys — System-specific parameters and functions*. https://docs.python.org/3/library/sys.html
- Real Python. *Python import: Advanced Techniques and Tips*. https://realpython.com/python-import/
- Real Python. *importlib | Python Standard Library*. https://realpython.com/ref/stdlib/importlib/
- Real Python. *What Does if `__name__` == `"__main__"` Do in Python?*. https://realpython.com/if-name-main-python/
- Python Mailing List. *Tutor FAQ: What is if `__name__` == `"__main__"` for?*. https://mail.python.org/pipermail/tutor/2006-May/046497.html
- Python Mailing List. *Re: import aliases?*. https://mail.python.org/pipermail/tutor/2005-April/038333.html
- Python Software Foundation. *bpo-32225: Implementation of PEP 562*. https://mail.python.org/archives/list/python-checkins@python.org/thread/GRSP77INSOBTY2VPUOK7AHRYZAIL4OCB/
- Python Software Foundation. *Python 3.7 Release Notes — PEP 562*. https://docs.python.org/3/whatsnew/3.7.html