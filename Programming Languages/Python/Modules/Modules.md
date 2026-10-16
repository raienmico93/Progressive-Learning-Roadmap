# Python Modules: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**

A Python module is a file containing Python definitions and statements. Modules allow you to organize related code into separate files, making programs easier to maintain, reuse, and share.

**Technical Definition**

In Python, a module is a file with a `.py` extension that contains Python code. When a module is imported, Python executes its top-level statements and creates a module object whose namespace is populated with the module's definitions. Each module has its own private namespace, which serves as the global namespace for all functions defined within it. The module's name is available as the value of the global variable `__name__`. Python's import system searches for modules in a configurable search path (`sys.path`), caches imported modules in `sys.modules`, and provides mechanisms for reloading modules via `importlib.reload()`.

**Beginner-Friendly Explanation**

A Python module is simply a `.py` file containing Python code that you can reuse in other programs. Think of it like a toolbox: instead of writing the same functions over and over, you put them in a file (a module) and import that file whenever you need those functions. For example, Python comes with a `math` module that contains mathematical functions like `sqrt()` and `pi`—you just write `import math` and you have access to all of them.

### Key Characteristics

- **File-Based**: A module is a `.py` file whose name becomes the module name.
- **Namespace Isolation**: Each module has its own private namespace, preventing name clashes.
- **Reusability**: Definitions in a module can be imported and used across multiple programs.
- **Executable Statements**: Modules can contain both definitions and executable statements; top-level code runs on first import.
- **Caching**: Imported modules are cached in `sys.modules`; subsequent imports return the cached module.
- **Search Path**: Python searches for modules in directories listed in `sys.path`.
- **Main Guard**: The `if __name__ == "__main__":` idiom allows a file to be both importable and runnable.

### Prerequisites

- Basic understanding of Python syntax and variables.
- Familiarity with functions and the `def` keyword.
- Knowledge of namespaces and scope (LEGB rule).
- Awareness of the file system and directory structure.

### Related Programming Areas

- **Packages**: Directories containing multiple modules with an `__init__.py` file.
- **Standard Library**: Python's built-in collection of modules (e.g., `os`, `sys`, `math`, `json`).
- **Third-Party Libraries**: External modules installed via pip (e.g., `numpy`, `requests`).
- **Virtual Environments**: Isolated Python environments for managing dependencies.
- **Import System**: The machinery that handles module search, loading, and caching.

---

## Core Concepts / Features

---

### 1. Modules and Module Creation

#### Definitions

**Core Definition**

A module is a file containing Python definitions and statements. Module creation is the process of writing a `.py` file that can be imported into other Python programs.

**Technical Definition**

A module is created by writing a Python source file with a `.py` extension. The file name (without the `.py` suffix) becomes the module name. When the module is imported, Python compiles the source code into bytecode, executes the module's top-level statements in a new namespace, and creates a module object. The module object's `__dict__` contains all names defined at the module level. Modules can define functions, classes, variables, and executable statements.

**Beginner-Friendly Explanation**

Creating a module is as simple as writing a Python file and saving it with a `.py` extension. The name of the file becomes the name of the module. For example, if you create a file called `mymath.py`, you can import it with `import mymath`.

#### Purposes

- To organize related code into reusable, self-contained files.
- To avoid code duplication by sharing functions and classes across programs.
- To break large programs into smaller, manageable pieces.
- To create libraries that others can import and use.

#### Syntax Rules and Structure

**Complete General Syntax (Module File)**

```py
"""Module docstring: describe the module's purpose."""

# Module-level variables
VERSION = "1.0.0"

# Module-level functions
def function_name(parameters):
    """Function docstring."""
    return value

# Module-level classes
class ClassName:
    """Class docstring."""
    pass

# Executable statements (run on first import)
print("Module initialized")
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| Module docstring | A string literal at the top describing the module. |
| Variables | Module-level constants and data. |
| Functions | Reusable function definitions. |
| Classes | Class definitions. |
| Executable statements | Code that runs when the module is imported. |

**Syntax Rules**

- The file name must be a valid Python identifier followed by `.py`.
- Module names should be short, lowercase, and use underscores if needed (PEP 8).
- A module can contain any valid Python code.
- Top-level statements are executed only on the first import.

**Constraints and Limitations**

- Module names cannot conflict with Python keywords.
- Module names should not shadow standard library modules.
- Modules cannot be empty files with no purpose; they should contain meaningful code.

#### Annotated Code Examples

**Example 1: Creating and Importing a Module**
`mymath.py`
```python
"""A simple math module."""

PI = 3.14159

def add(a, b):
    """Return the sum of a and b."""
    return a + b

def multiply(a, b):
    """Return the product of a and b."""
    return a * b

if __name__ == "__main__":
    print("Running as script")
    print(add(2, 3))
```
\
`main.py`
```python
import mymath

print(mymath.PI)
print(mymath.add(5, 3))
print(mymath.multiply(4, 6))
```

**Expected Output:**

```
3.14159
8
24
```

**Step-by-Step Breakdown:**

1. `mymath.py` defines `PI`, `add()`, and `multiply()`.
2. `import mymath` loads the module and executes its top-level code.
3. `mymath.PI` accesses the module's variable.
4. `mymath.add(5, 3)` calls the module's function.

**Why the Output is Produced:**

When `mymath` is imported, Python executes `mymath.py`, creating a module object with `PI`, `add`, and `multiply` in its namespace. The `if __name__ == "__main__"` block is skipped because the module is imported, not run as a script.

#### Real-World Cases

- **Utility Modules**: Reusable functions for string manipulation, file I/O, or math.
- **Configuration Modules**: Constants and settings shared across a project.
- **Domain Modules**: Classes and functions for a specific domain (e.g., `customers.py`, `orders.py`).

---

### 2. Module Import

#### Definitions

**Core Definition**

Module import is the process of making a module's definitions available in another module or script. Python provides several import statement forms.

**Technical Definition**

The `import` statement invokes Python's import machinery, which searches for the named module in `sys.path`, loads and executes it (if not already cached in `sys.modules`), and binds the resulting module object to a name in the current namespace. The import system supports various forms: `import module`, `from module import name`, `import module as alias`, and `from module import *`. Each form has different effects on the importing namespace.

**Beginner-Friendly Explanation**

Importing a module is like opening a toolbox and taking out the tools you need. You can bring in the whole toolbox (`import module`), bring in specific tools (`from module import tool`), or give the toolbox a nickname (`import module as nickname`).

#### Purposes

- To access functions, classes, and variables defined in other modules.
- To reuse code without copying it.
- To organize code into logical units.
- To leverage Python's standard library and third-party packages.

#### Syntax Rules and Structure

**Complete General Syntax**

```py
# Import the entire module
import module_name

# Import specific names from a module
from module_name import name1, name2

# Import with an alias
import module_name as alias

# Import all public names (discouraged)
from module_name import *

# Import specific names with aliases
from module_name import name1 as alias1, name2 as alias2
```

**Component Breakdown**

| Form | Description |
|------|-------------|
| `import module` | Binds the module object to `module` in the current namespace. |
| `from module import name` | Binds `name` directly in the current namespace. |
| `import module as alias` | Binds the module to `alias`. |
| `from module import *` | Imports all public names (those not starting with `_`). |
| `from module import name as alias` | Binds `name` to `alias`. |

**Syntax Rules**

- Imports should be at the top of the file, after module docstring and before other code.
- Imports should be grouped: standard library, third-party, local application (PEP 8).
- Each import should be on its own line, except `from` imports.
- Wildcard imports (`from module import *`) should be avoided in production code.
- Absolute imports are preferred over relative imports.

**Constraints and Limitations**

- `from module import *` pollutes the namespace and makes code unclear.
- Circular imports can cause `ImportError`.
- Import statements execute the module's top-level code, which may have side effects.

#### Annotated Code Examples

**Example 1: Import Forms**

```python
# Create a module file: greetings.py
"""A module with greeting functions."""

def hello(name):
    return f"Hello, {name}!"

def goodbye(name):
    return f"Goodbye, {name}!"

GREETING = "Welcome!"
```

```python
# File: main.py
# Import the entire module
import greetings
print(greetings.hello("Alice"))

# Import specific names
from greetings import goodbye, GREETING
print(goodbye("Bob"))
print(GREETING)

# Import with alias
import greetings as g
print(g.hello("Charlie"))
```

**Expected Output:**

```
Hello, Alice!
Goodbye, Bob!
Welcome!
Hello, Charlie!
```

**Step-by-Step Breakdown:**

1. `import greetings` binds `greetings` to the module object.
2. `from greetings import goodbye, GREETING` binds `goodbye` and `GREETING` directly.
3. `import greetings as g` binds the module to `g`.

**Why the Output is Produced:**

Each import form makes the module's definitions available in a different way, depending on how you want to reference them.

---

**Example 2: Wildcard Import Pitfall**

```python
# File: math_utils.py
def add(a, b):
    return a + b

def subtract(a, b):
    return a - b

def _internal_helper():
    return "hidden"

__all__ = ["add", "subtract"]  # Controls what * imports
```

```python
# File: wildcard_demo.py
from math_utils import *
print(add(3, 5))
print(subtract(10, 4))
# print(_internal_helper())  # NameError: not imported
```

**Expected Output:**

```
8
6
```

**Step-by-Step Breakdown:**

1. `__all__` defines which names are imported by `*`.
2. `_internal_helper` is not imported because it starts with `_` and is not in `__all__`.

**Why the Output is Produced:**

Wildcard imports only bring in public names (those listed in `__all__` or not starting with `_`).

#### Real-World Cases

- **Standard Library**: `import os`, `import sys`, `from datetime import datetime`.
- **Third-Party**: `import numpy as np`, `from flask import Flask`.
- **Local Modules**: `from . import utils`, `from mypackage import mymodule`.

---

### 3. Module Search Path, `sys.path`, and `PYTHONPATH`

#### Definitions

**Core Definition**

The module search path is the list of directories Python searches when looking for a module. It is accessible at runtime as `sys.path`.

**Technical Definition**

When Python starts, it initializes a module search path accessible via `sys.path`. The first entry is the directory containing the input script (or the current directory for interactive sessions). The `PYTHONPATH` environment variable can add directories to this search path. Next, the standard library directories and site-packages directories are added. The search path can be modified at runtime by manipulating `sys.path`.

**Beginner-Friendly Explanation**

When you import a module, Python doesn't know where to find it. It looks in a list of directories called the "search path." You can think of it like a list of places to look for a book. `sys.path` is that list, and you can add your own directories to it.

#### Purposes

- To control where Python looks for modules.
- To add custom directories containing your modules.
- To understand why a module cannot be found.
- To organize project structures and module locations.

#### Syntax Rules and Structure

```py
import sys
print(sys.path)

# Add a directory to the search path
sys.path.append("/path/to/my/modules")

# Insert at the beginning (higher priority)
sys.path.insert(0, "/path/to/my/modules")
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `sys.path` | List of directories Python searches for modules. |
| `PYTHONPATH` | Environment variable that adds directories to `sys.path`. |
| `sys.path.append()` | Adds a directory to the end of the search path. |
| `sys.path.insert(0, ...)` | Adds a directory to the beginning (highest priority). |

**Syntax Rules**

- `sys.path` is a list of strings.
- The first entry is the script's directory or the current directory.
- `PYTHONPATH` entries are added after the script directory but before standard library paths.
- Modifying `sys.path` affects the current session only.

**Constraints and Limitations**

- Modifying `sys.path` is session-specific and not persistent.
- `PYTHONPATH` affects all Python versions and environments, which can cause conflicts.
- The search path order matters: earlier entries take precedence.

#### Annotated Code Examples

**Example 1: Inspecting `sys.path`**

```python
import sys
print("First entry:", sys.path[0])
print("Number of entries:", len(sys.path))
print("\nAll entries:")
for i, path in enumerate(sys.path):
    print(f"  {i}: {path}")
```

**Expected Output (example):**

```
First entry: /current/script/directory
Number of entries: 10
All entries:
  0: /current/script/directory
  1: /usr/lib/python3.11
  2: /usr/lib/python3.11/lib-dynload
  3: /usr/local/lib/python3.11/site-packages
  ...
```

**Step-by-Step Breakdown:**

1. `sys.path[0]` is the directory of the script.
2. Subsequent entries are standard library and site-packages directories.

**Why the Output is Produced:**

Python initializes `sys.path` at startup based on the script location, environment variables, and installation directories.

---

**Example 2: Adding a Custom Module Directory**

```python
import sys
import os

# Add a custom directory to the search path
custom_dir = os.path.expanduser("~/my_modules")
if custom_dir not in sys.path:
    sys.path.append(custom_dir)

# Now modules in ~/my_modules can be imported
import my_custom_module
```

**Step-by-Step Breakdown:**

1. `os.path.expanduser("~/my_modules")` resolves the path to the user's home directory.
2. `sys.path.append(custom_dir)` adds the directory to the search path.
3. `import my_custom_module` now searches the custom directory.

**Why the Output is Produced:**

By appending the custom directory to `sys.path`, Python will search there when importing modules.

#### Real-World Cases

- **Development**: Adding project directories to `sys.path` during development.
- **Testing**: Adding test directories for unit testing.
- **Plugins**: Loading plugins from user-specified directories.

---

### 4. Module Caching and `sys.modules`

#### Definitions

**Core Definition**

Module caching is Python's mechanism for storing imported modules in a dictionary called `sys.modules`. Once a module is imported, it is cached and subsequent imports return the same module object.

**Technical Definition**

`sys.modules` is a dictionary that maps module names to module objects. When a module is imported, Python first checks `sys.modules` to see if the module has already been loaded. If it has, Python returns the cached module object. If not, Python searches for the module, loads it, and stores it in `sys.modules`. This caching ensures that a module's code is executed only once, even if it is imported multiple times.

**Beginner-Friendly Explanation**

When you import a module, Python remembers it. If you import the same module again, Python doesn't reload it—it just gives you the same module it loaded before. This is called caching.

#### Purposes

- To avoid re-executing a module's code multiple times.
- To ensure consistency: all imports of the same module share the same module object.
- To improve performance by avoiding redundant file I/O and compilation.

#### Syntax Rules and Structure

```
import sys

# Check if a module is loaded
if "module_name" in sys.modules:
    print("Module is loaded")

# Access the cached module
module = sys.modules["module_name"]

# Remove a module from the cache (not recommended)
del sys.modules["module_name"]
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `sys.modules` | Dictionary mapping module names to module objects. |
| `"module_name" in sys.modules` | Checks if a module is cached. |
| `del sys.modules["module_name"]` | Removes a module from the cache. |

**Syntax Rules**

- `sys.modules` is populated at interpreter startup with built-in modules.
- Importing a module automatically adds it to `sys.modules`.
- Removing a module from `sys.modules` does not unload it; it just removes the cache entry.

**Constraints and Limitations**

- Modifying `sys.modules` directly is generally discouraged.
- Removing a module from the cache does not unload it; existing references remain valid.
- Reloading modules with `importlib.reload()` is preferred over cache manipulation.

#### Annotated Code Examples

**Example 1: Checking `sys.modules`**

```python
import sys

# Import a module
import math

# Check if it's in sys.modules
print("math" in sys.modules)  # True
print("nonexistent" in sys.modules)  # False

# Access the cached module
cached_math = sys.modules["math"]
print(cached_math.sqrt(16))  # 4.0

# Importing again returns the same object
import math as math2
print(math is math2)  # True
```

**Expected Output:**

```
True
False
4.0
True
```

**Step-by-Step Breakdown:**

1. `import math` loads `math` and stores it in `sys.modules`.
2. `"math" in sys.modules` returns `True`.
3. `sys.modules["math"]` retrieves the cached module.
4. Re-importing `math` returns the same object.

**Why the Output is Produced:**

Python caches imported modules in `sys.modules`. Subsequent imports return the cached module rather than reloading it.

#### Real-World Cases

- **Debugging**: Checking `sys.modules` to see what modules are loaded.
- **Testing**: Manipulating `sys.modules` to mock modules.
- **Performance**: Understanding why a module's code runs only once.

---

### 5. Module Reloading and `importlib.reload()`

#### Definitions

**Core Definition**

Module reloading is the process of re-executing a module's code to pick up changes made to the source file. Python provides `importlib.reload()` for this purpose.

**Technical Definition**

`importlib.reload(module)` reloads a previously imported module. The argument must be a module object. The module's code is recompiled and re-executed, and the module's namespace is updated to point to new or changed objects. However, existing references to old objects (e.g., names imported with `from module import name`) are not updated, which is a common source of confusion.

**Beginner-Friendly Explanation**

If you change a module's code while your program is running, the changes won't take effect until you reload the module. `importlib.reload()` does this for you.

#### Purposes

- To pick up changes to a module without restarting the interpreter.
- To support interactive development and debugging.
- To test modified module code.

#### Syntax Rules and Structure

```
import importlib

# Reload a module
importlib.reload(module_name)
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `importlib.reload(module)` | Reloads the given module object. |
| `module` | A module object that has been previously imported. |

**Syntax Rules**

- The argument must be a module object (not a string).
- The module must have been successfully imported before.
- The return value is the module object (which may be different if re-importing causes a different object to be placed in `sys.modules`).

**Constraints and Limitations**

- Existing references to old objects are not updated.
- Reloading can be unpredictable with circular imports.
- Not all modules can be reloaded (e.g., extension modules).
- The `__name__` guard (`if __name__ == "__main__"`) may not behave as expected.

#### Annotated Code Examples

**Example 1: Reloading a Module**

```python
# File: mymodule.py
VERSION = 1.0

def greet():
    return "Hello from version 1"
```

```python
# File: main.py
import mymodule
import importlib
import time

print(mymodule.greet())

# Simulate editing the module (in practice, you'd edit the file)
# After editing mymodule.py to change VERSION and greet, reload:
importlib.reload(mymodule)
print(mymodule.greet())
```

**Expected Output:**

```
Hello from version 1
Hello from version 2
```

**Step-by-Step Breakdown:**

1. `import mymodule` loads the module.
2. `importlib.reload(mymodule)` re-executes `mymodule.py`, picking up changes.
3. The new `greet()` function is called.

**Why the Output is Produced:**

Reloading re-executes the module's code, updating the module's namespace with new definitions.

#### Real-World Cases

- **Interactive Development**: Reloading modules in Jupyter notebooks or REPL sessions.
- **Plugin Systems**: Reloading plugins after they are updated.
- **Testing**: Testing changes to a module without restarting the test runner.

---

### 6. Module Attributes: `__name__`, `__file__`, `__doc__`

#### Definitions

**Core Definition**

Module attributes are special variables that provide metadata about a module. Key attributes include `__name__` (the module's name), `__file__` (the file path), and `__doc__` (the documentation string).

**Technical Definition**

Every module object has predefined attributes that provide information about the module. `__name__` is the module's name as a string; when the module is run as a script, `__name__` is set to `"__main__"`. `__file__` is the pathname of the file from which the module was loaded (may be missing for some modules). `__doc__` is the module's documentation string, or `None` if not provided.

**Beginner-Friendly Explanation**

Modules have built-in information about themselves. `__name__` tells you the module's name, `__file__` tells you where it is on disk, and `__doc__` gives you its description.

#### Purposes

- To identify the module's name at runtime.
- To locate the module's source file.
- To access the module's documentation.
- To implement the `if __name__ == "__main__"` idiom.

#### Syntax Rules and Structure

```
# Access module attributes
print(__name__)
print(__file__)
print(__doc__)
```

**Component Breakdown**

| Attribute | Description |
|-----------|-------------|
| `__name__` | The module's name. `"__main__"` when run as a script. |
| `__file__` | The pathname of the module file. May be missing for built-in modules. |
| `__doc__` | The module's docstring, or `None`. |

**Syntax Rules**

- `__name__` is always defined.
- `__file__` may be missing for built-in modules.
- `__doc__` is `None` if no docstring is provided.

**Constraints and Limitations**

- `__file__` may be a relative path or an absolute path depending on how the module was loaded.
- `__file__` is not available in interactive sessions.
- `__doc__` is only set if the first statement in the module is a string literal.

#### Annotated Code Examples

**Example 1: Module Attributes**

```python
# File: mymodule.py
"""This is my module's docstring."""

print("Module name:", __name__)
print("Module file:", __file__)
print("Module doc:", __doc__)
```

```python
# File: main.py
import mymodule
```

**Expected Output:**

```
Module name: mymodule
Module file: /path/to/mymodule.py
Module doc: This is my module's docstring.
```

**Step-by-Step Breakdown:**

1. `__name__` is `"mymodule"` when imported.
2. `__file__` is the path to the module file.
3. `__doc__` is the module's docstring.

**Why the Output is Produced:**

Python sets these attributes automatically when the module is loaded.

#### Real-World Cases

- **Logging**: Using `__name__` to configure loggers.
- **Resource Loading**: Using `__file__` to find files relative to the module.
- **Documentation**: Accessing `__doc__` for help systems.

---

### 7. `if __name__ == "__main__"`

#### Definitions

**Core Definition**

The `if __name__ == "__main__":` idiom is a conditional block that runs code only when the file is executed as a script, not when it is imported as a module.

**Technical Definition**

When Python runs a script, it sets the script's `__name__` to `"__main__"`. When the script is imported as a module, `__name__` is set to the module's name. The `if __name__ == "__main__":` idiom checks this condition and executes the guarded code only when the file is run directly.

**Beginner-Friendly Explanation**

This idiom lets a file serve two purposes: it can be imported as a module, or it can be run as a standalone script. The code inside the `if` block runs only when you run the file directly.

#### Purposes

- To allow a file to be both importable and runnable.
- To prevent test code or demo code from running on import.
- To define a clear entry point for a script.

#### Syntax Rules and Structure

```
if __name__ == "__main__":
    # Code that runs only when the file is executed as a script
    main()
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `__name__` | Built-in variable containing the module's name. |
| `"__main__"` | The value of `__name__` when the file is run directly. |
| `main()` | A function containing the script's main logic. |

**Syntax Rules**

- The idiom should be placed at the bottom of the file.
- The main logic should be in a `main()` function for clarity.
- Code inside the `if` block runs only when `__name__ == "__main__"`.

**Constraints and Limitations**

- The idiom is not a strict requirement; it is a best practice.
- In interactive sessions, `__name__` is `"__main__"`.

#### Annotated Code Examples

**Example 1: Script and Module Dual Use**

```python
# File: calculator.py
"""A simple calculator module."""

def add(a, b):
    return a + b

def subtract(a, b):
    return a - b

def main():
    print("Calculator Demo")
    print(add(5, 3))
    print(subtract(10, 4))

if __name__ == "__main__":
    main()
```

```python
# File: use_calculator.py
import calculator

print(calculator.add(100, 200))
```

**Expected Output:**

```
# Running calculator.py directly:
Calculator Demo
8
6

# Running use_calculator.py:
300
```

**Step-by-Step Breakdown:**

1. When `calculator.py` is run directly, `__name__` is `"__main__"`, so `main()` runs.
2. When `calculator.py` is imported, `__name__` is `"calculator"`, so `main()` does not run.

**Why the Output is Produced:**

The `if` condition is `True` only when the file is executed directly.

#### Real-World Cases

- **CLI Tools**: Scripts that can be run from the command line or imported.
- **Test Files**: Test code that runs only when the file is executed.
- **Library Modules**: Modules that include demo code in the main guard.

---

### 8. Module Best Practices

#### Definitions

**Core Definition**

Module best practices are guidelines for creating well-organized, maintainable, and reusable Python modules.

**Technical Definition**

Module best practices include following PEP 8 naming conventions (short, lowercase names), grouping imports (standard library, third-party, local), avoiding wildcard imports, using `__all__` to define public APIs, writing module docstrings, and using the `if __name__ == "__main__"` guard. These practices improve readability, maintainability, and interoperability.

**Beginner-Friendly Explanation**

Best practices are good habits that make your modules clean and easy to use. They include naming your modules with lowercase letters, putting imports at the top, documenting what the module does, and using the main guard.

#### Purposes

- To write readable and maintainable modules.
- To avoid common pitfalls like namespace pollution and circular imports.
- To create reusable, well-documented code.
- To align with Python community conventions.

#### Syntax Rules and Structure

**General Guidelines**

```python
"""Module docstring."""

# Standard library imports
import os
import sys

# Third-party imports
import requests

# Local imports
from . import utils

__all__ = ["public_function", "PublicClass"]

def public_function():
    pass

class PublicClass:
    pass

if __name__ == "__main__":
    main()
```

**Best Practices Table**

| Practice | Description |
|----------|-------------|
| Use lowercase module names | Short, lowercase names with underscores if needed. |
| Group imports | Standard library, third-party, local (separated by blank lines). |
| Avoid wildcard imports | Use explicit imports instead of `from module import *`. |
| Define `__all__` | Explicitly declare the module's public API. |
| Write module docstrings | Describe the module's purpose at the top of the file. |
| Use the main guard | `if __name__ == "__main__":` for script entry points. |
| Keep modules focused | Each module should have a single, clear purpose. |
| Avoid circular imports | Structure modules to avoid mutual dependencies. |
| Use absolute imports | Prefer absolute imports over relative imports. |

**Constraints and Limitations**

- Best practices are guidelines, not absolute rules.
- Context matters; some projects may have specific conventions.
- Over-documentation can be as bad as under-documentation.

#### Annotated Code Examples

**Example 1: Well-Structured Module**

```python
"""A module for string utilities."""

import re
from typing import List

__all__ = ["clean_text", "extract_emails"]

def clean_text(text: str) -> str:
    """Remove extra whitespace and convert to lowercase."""
    return re.sub(r"\s+", " ", text).strip().lower()

def extract_emails(text: str) -> List[str]:
    """Extract all email addresses from text."""
    pattern = r"[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}"
    return re.findall(pattern, text)

if __name__ == "__main__":
    sample = "  Hello   World!  "
    print(clean_text(sample))
```

**Expected Output:**

```
hello world!
```

**Step-by-Step Breakdown:**

1. Module docstring describes the module.
2. `__all__` defines the public API.
3. Functions have type annotations and docstrings.
4. The main guard runs demo code only when executed directly.

**Why the Output is Produced:**

The module is well-documented, has a clear public API, and can be used both as a library and as a script.

#### Real-World Cases

- **Open-Source Libraries**: Well-structured modules are easier for others to use.
- **Team Projects**: Consistent module structure improves collaboration.
- **Long-Term Maintenance**: Good practices make code easier to modify and debug.

---

## References

- Python Official Documentation – Modules (Tutorial) - https://docs.python.org/3/tutorial/modules.html
- Python Official Documentation – The import system (Language Reference) - https://docs.python.org/3/reference/import.html
- Python Official Documentation – The initialization of the `sys.path` module search path - https://docs.python.org/3/library/sys_path_init.html
- Python Official Documentation – `importlib` — The implementation of import - https://docs.python.org/3/library/importlib.html
- Python Official Documentation – `sys.modules` - https://docs.python.org/3/library/sys.html#sys.modules
- Python Official Documentation – Module Objects (Data Model) - https://docs.python.org/3/reference/datamodel.html#module-objects
- PEP 8 – Style Guide for Python Code - https://peps.python.org/pep-0008/
- PEP 257 – Docstring Conventions - https://peps.python.org/pep-0257/
- PEP 302 – New Import Hooks - https://peps.python.org/pep-0302/
- PEP 328 – Imports: Multi-Line and Absolute/Relative - https://peps.python.org/pep-0328/
- Real Python – Python Modules and Packages – An Introduction - https://realpython.com/python-modules-packages/
- Real Python – What Does `if __name__ == "__main__"` Do in Python? - https://realpython.com/if-name-main-python/
- Real Python – The Python import System – Real Python - https://realpython.com/python-import/
- Real Python – Wildcard Import | Python Glossary - https://realpython.com/ref/glossary/wildcard-import/
- Python Mailing List – `sys.modules` is not a cache - https://mail.python.org/archives/list/python-list@python.org/message/KAWTRWV7FW3G3CQXSIZN6CFAOJZWN6WZ/
- ORNL Python Style Guide – Imports and Naming - https://ornl-training.github.io/python-style/instructor/style-guide.html
- CMU – Python Modules - https://www.cs.cmu.edu/~15110-s25/notes/python-modules.html