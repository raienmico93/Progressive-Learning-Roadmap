# Python Execution Model

Understanding how Python runs code helps you debug performance issues, reason about imports, and understand why certain behaviors occur. This section walks through the full path from source code to execution.

---

## Overview

When you run a Python program, it goes through several stages:

```text
Source code (.py)
      │
      ▼
   Parsing
      │
      ▼
Abstract Syntax Tree (AST)
      │
      ▼
   Compilation
      │
      ▼
Bytecode (.pyc)
      │
      ▼
Python Virtual Machine (PVM)
      │
      ▼
   Execution
```

Python is often described as **interpreted**, but it is more accurate to say it is **compiled to bytecode, then interpreted by a virtual machine**.

---

## Source Code

**Source code** is the human-readable Python text stored in `.py` files.

```python
# hello.py
def greet(name):
    return f"Hello, {name}!"

print(greet("World"))
```

Source files are typically encoded in **UTF-8** by default (PEP 3120). You can declare a different encoding with a coding comment on the first or second line:

```python
# -*- coding: latin-1 -*-
```

Source code can also come from:

- The interactive REPL (typed input)
- `-c` command-line strings
- `exec()` and `eval()`
- Compiled code objects
- Jupyter cells

---

## Parsing

**Parsing** is the process of analyzing source code and turning it into a structured representation.

Steps:

1. **Tokenization (lexing)** — the source text is split into tokens: names, numbers, strings, operators, keywords, indentation markers.
2. **Parsing** — tokens are arranged into a syntax tree according to Python's grammar (defined in `Grammar/python.gram` for CPython 3.9+).
3. **Validation** — syntax errors are raised here.

Example tokens for `x = 1 + 2`:

```text
NAME    'x'
OP      '='
NUMBER  '1'
OP      '+'
NUMBER  '2'
NEWLINE ''
```

If parsing fails, you get a `SyntaxError`:

```python
>>> x = 
  File "<stdin>", line 1
    x =
       ^
SyntaxError: invalid syntax
```

The `ast` module exposes parsing:

```python
import ast

tree = ast.parse("x = 1 + 2")
print(ast.dump(tree, indent=2))
```

Output (simplified):

```text
Module(
  body=[
    Assign(
      targets=[Name(id='x', ctx=Store())],
      value=BinOp(
        left=Constant(value=1),
        op=Add(),
        right=Constant(value=2)))])
```

---

## Abstract Syntax Tree

The **Abstract Syntax Tree (AST)** is a tree representation of the program's structure. Each node represents a syntactic construct: expressions, statements, functions, classes, and so on.

Common AST node types:

| Node | Represents |
|---|---|
| `Module` | Top-level container |
| `Assign` | Assignment statement |
| `BinOp` | Binary operation (`+`, `-`, etc.) |
| `Call` | Function call |
| `FunctionDef` | Function definition |
| `ClassDef` | Class definition |
| `If`, `For`, `While` | Control flow |
| `Name` | Variable reference |
| `Constant` | Literal value |

Uses of the AST:

- **Static analysis** (linters like flake8, ruff)
- **Code transformation** (macros, decorators via AST)
- **Type checking** (mypy, pyright)
- **Code generation** and metaprogramming
- **Security tooling** (bandit)

Walk an AST:

```python
import ast

source = """
def add(a, b):
    return a + b
"""

tree = ast.parse(source)

for node in ast.walk(tree):
    print(type(node).__name__)
```

Modify and recompile:

```python
import ast

class Rewrite(ast.NodeTransformer):
    def visit_Constant(self, node):
        if isinstance(node.value, int):
            return ast.Constant(value=node.value + 1)
        return node

tree = ast.parse("x = 1 + 2")
tree = Rewrite().visit(tree)
ast.fix_missing_locations(tree)

code = compile(tree, "<ast>", "exec")
exec(code)
print(x)   # 4
```

> **Note:** Python 3.8+ uses `Constant` for literals; older versions used `Num`, `Str`, and similar nodes.

---

## Bytecode

After parsing and AST construction, CPython **compiles** the AST into **bytecode** — a compact, low-level set of instructions for the Python Virtual Machine.

You can inspect bytecode with the `dis` module:

```python
import dis

def add(a, b):
    return a + b

dis.dis(add)
```

Output:

```text
  2           0 RESUME                   0
              2 LOAD_FAST                0 (a)
              4 LOAD_FAST                1 (b)
              6 BINARY_OP                0 (+)
             10 RETURN_VALUE
```

Bytecode instructions include:

| Opcode | Purpose |
|---|---|
| `LOAD_FAST` | Push a local variable |
| `LOAD_CONST` | Push a constant |
| `LOAD_GLOBAL` | Push a global |
| `STORE_FAST` | Pop into a local variable |
| `BINARY_OP` | Arithmetic/logic |
| `CALL` | Call a callable |
| `RETURN_VALUE` | Return from function |
| `POP_TOP` | Discard top of stack |
| `JUMP_FORWARD` | Unconditional jump |
| `POP_JUMP_IF_FALSE` | Conditional jump |
| `COMPARE_OP` | Comparison |

Bytecode is **stack-based**: instructions push and pop values on an evaluation stack.

Compile source to a code object:

```python
code = compile("x = 1 + 2", "<string>", "exec")
print(code.co_code)          # raw bytes
print(code.co_consts)        # constants
print(code.co_names)         # names referenced
```

> **Important:** Bytecode is an implementation detail. It changes between Python versions and even minor releases. Do not depend on specific opcodes.

---

## Python Virtual Machine

The **Python Virtual Machine (PVM)** is the runtime engine that executes bytecode. In CPython, it is implemented in C as the "ceval" loop (`Python/ceval.c` in older versions, `Python/bytecodes.c` in 3.11+).

The PVM:

1. Reads bytecode instructions one by one.
2. Maintains a call stack and an evaluation stack.
3. Manages frames, locals, globals, and built-ins.
4. Handles exceptions and control flow.

A **frame** represents an active function call:

```python
import sys

def f():
    frame = sys._getframe()
    print(frame.f_code.co_name)   # f

f()
```

Each frame contains:

- The code object being executed
- Local variables
- The evaluation stack
- A reference to the previous frame
- The current instruction pointer

Inspect the call stack:

```python
import inspect

def a():
    b()

def b():
    for frame_info in inspect.stack():
        print(frame_info.function)

a()
```

Output:

```text
b
a
<module>
```

---

## Interpreter Execution

Putting it together, when you run `python script.py`:

1. The interpreter **initializes** (sets up built-ins, sys.path, etc.).
2. The source is **read and parsed** into an AST.
3. The AST is **compiled** into a code object containing bytecode.
4. A **module object** is created and added to `sys.modules`.
5. The PVM **executes** the module's bytecode in the `__main__` frame.
6. Names created at top level become attributes of the module.

For the main script, `__name__` is set to `"__main__"`. For imported modules, `__name__` is the module name.

```python
# main.py
print(__name__)       # __main__
```

```python
# other.py
import main           # prints "main"
```

---

## `.pyc` Files

When a module is imported, CPython **caches** the compiled bytecode in a `.pyc` file. This avoids re-parsing and re-compiling the next time the module is imported.

The `.pyc` file contains:

- A **magic number** (Python version identifier)
- A **bit field** with flags
- A **timestamp** or **hash** of the source
- The **marshalled code object**

You can compile manually:

```bash
python -m py_compile script.py
python -m compileall mypackage/
```

Inspect a `.pyc`:

```python
import marshal, importlib.util

with open("__pycache__/script.cpython-312.pyc", "rb") as f:
    f.read(16)  # skip header
    code = marshal.load(f)

print(code.co_names)
```

### When `.pyc` Files Are Invalid

A `.pyc` is recompiled if:

- The source file is newer.
- The magic number differs (different Python version).
- The hash/timestamp differs.

### Disabling `.pyc` Generation

```bash
python -B script.py
```

Or set:

```bash
export PYTHONDONTWRITEBYTECODE=1
```

---

## `__pycache__`

`__pycache__` is the directory where CPython stores `.pyc` files. It is created next to the source file.

Example layout:

```text
myproject/
├── main.py
├── utils.py
└── __pycache__/
    ├── main.cpython-312.pyc
    └── utils.cpython-312.pyc
```

The filename encodes the Python version:

```text
utils.cpython-312.pyc
      │       │
      │       └── Python 3.12
      └── Implementation (cpython, pypy, etc.)
```

Notes:

- `__pycache__` is safe to delete; it will be regenerated.
- It should be ignored in version control:
  ```gitignore
  __pycache__/
  *.py[cod]
  ```
- It is **not** used to run scripts directly (`python script.py` compiles in memory and writes `.pyc` only if imported).

---

## Import Mechanisms

Imports are one of the most complex parts of Python's runtime. The import system is defined in **PEP 302** (and later PEPs) and implemented in `importlib`.

### Import Steps

When you write `import mymodule`:

1. **Check `sys.modules`** — if already imported, return the cached module.
2. **Find the module** using **finders** and **path hooks**.
3. **Load the module** using a **loader**.
4. **Execute the module's code** in a fresh namespace.
5. **Cache** the module in `sys.modules`.
6. **Bind** the name in the importing namespace.

### Key Components

| Component | Role |
|---|---|
| `sys.modules` | Cache of imported modules |
| `sys.meta_path` | List of meta path finders |
| `sys.path` | List of directories to search |
| `importlib` | Public API for importing |
| `Finder` | Locates a module |
| `Loader` | Loads and executes a module |
| `ModuleSpec` | Metadata about a module |

### Example: Inspect Import

```python
import sys, importlib.util

spec = importlib.util.find_spec("json")
print(spec.origin)   # /usr/lib/python3.12/json/__init__.py
print(spec.loader)   # <_frozen_importlib_external.SourceFileLoader ...>
print("json" in sys.modules)   # False before import

import json
print("json" in sys.modules)   # True
```

### Import Variants

```python
import math                      # bind module name
import math as m                 # bind alias
from math import sqrt            # bind a name
from math import sqrt as s       # bind an alias
from math import *               # wildcard (avoid)
from . import sibling            # relative import
from ..pkg import mod            # parent-relative
```

### Import Hooks

You can customize importing:

```python
import sys

class MyFinder:
    def find_spec(self, name, path, target=None):
        if name == "magic":
            import importlib.util
            return importlib.util.spec_from_loader(name, MyLoader())
        return None

class MyLoader:
    def create_module(self, spec):
        return None
    def exec_module(self, module):
        module.value = 42

sys.meta_path.append(MyFinder())

import magic
print(magic.value)   # 42
```

### Import Caching and Reloading

Modules are imported only once. To reload:

```python
import importlib
importlib.reload(mymodule)
```

### Common Pitfalls

- **Circular imports** — module A imports B, B imports A.
- **Shadowing** — a local file `random.py` shadows the standard library.
- **Side effects at import time** — heavy work, network calls, or prints in module scope.
- **Mutable module-level state** — shared across importers.

### `__name__ == "__main__"`

This guard prevents code from running when the module is imported:

```python
def main():
    ...

if __name__ == "__main__":
    main()
```

---

## Runtime Behavior

### Name Resolution: LEGB

Python resolves names using the **LEGB** rule:

| Scope | Meaning |
|---|---|
| **L**ocal | Inside the current function |
| **E**nclosing | Outer function (closures) |
| **G**lobal | Module-level |
| **B**uilt-in | `builtins` module |

Example:

```python
x = "global"

def outer():
    x = "enclosing"
    def inner():
        x = "local"
        print(x)     # local
    inner()
    print(x)         # enclosing

outer()
print(x)             # global
```

Use `globals()` and `locals()` to inspect:

```python
def f():
    a = 1
    print(locals())   # {'a': 1}

f()
print(globals().keys())
```

`nonlocal` and `global` alter binding:

```python
def counter():
    count = 0
    def inc():
        nonlocal count
        count += 1
        return count
    return inc
```

### Object Model at Runtime

- Everything is an object, including functions, classes, and modules.
- Types are instances of `type`.
- Functions are first-class objects.
- Attributes live in `__dict__`.

```python
def f(): pass
print(type(f))         # <class 'function'>
print(f.__dict__)      # {}
print(type(type))      # <class 'type'>
```

### Garbage Collection

CPython uses:

- **Reference counting** — objects are freed when refcount reaches 0.
- **Cycle collector** — detects and cleans reference cycles.

```python
import sys, gc

a = []
print(sys.getrefcount(a))   # typically 2+ (variable + getrefcount arg)
gc.collect()
```

### Exceptions and Tracebacks

Exceptions unwind the call stack. Each frame contributes a traceback entry.

```python
import traceback

def a(): b()
def b(): c()
def c(): raise ValueError("boom")

try:
    a()
except ValueError:
    traceback.print_exc()
```

### Threads and the GIL

CPython has a **Global Interpreter Lock (GIL)** that allows only one thread to execute Python bytecode at a time.

- CPU-bound work does not scale with threads.
- I/O-bound work benefits from threads.
- Use `multiprocessing` or native extensions for CPU parallelism.

Python 3.13 introduced an **experimental free-threaded build** (no GIL) as an opt-in.

### `sys` Internals

Useful runtime attributes:

```python
import sys
print(sys.version)
print(sys.executable)
print(sys.path)
print(sys.modules)
print(sys.getrecursionlimit())
print(sys.flags)
```

### Runtime Introspection

```python
import inspect

print(inspect.getsource(inspect.getsource))
print(inspect.signature(print))
print(inspect.isfunction(print))
```

### `eval`, `exec`, and `compile`

```python
exec("x = 10")
print(x)                     # 10

print(eval("2 + 3"))         # 5

code = compile("a + b", "<expr>", "eval")
print(eval(code, {"a": 1, "b": 2}))   # 3
```

> Avoid `exec`/`eval` on untrusted input.

---

## Summary

| Stage | Description |
|---|---|
| Source code | Human-readable `.py` files (UTF-8 by default) |
| Parsing | Tokenization and grammar check; produces AST |
| AST | Tree representation of the program |
| Bytecode | Compact instructions for the PVM |
| PVM | Executes bytecode in frames |
| Interpreter execution | Runs the module under `__main__` |
| `.pyc` files | Cached compiled bytecode |
| `__pycache__` | Directory holding `.pyc` files |
| Import mechanisms | `sys.modules`, finders, loaders, `importlib` |
| Runtime behavior | LEGB, GC, GIL, introspection |

### Key Takeaways

- Python compiles source to bytecode, then interprets it on the PVM.
- The AST is a powerful hook for analysis and transformation.
- `.pyc` and `__pycache__` speed up repeated imports.
- Imports are cached in `sys.modules`; reload explicitly when needed.
- Name lookup follows LEGB; scoping is lexical.
- Runtime introspection via `dis`, `ast`, `inspect`, `sys`, and `importlib` is a core Python strength.

A solid mental model of the execution pipeline makes you a more effective Python developer — whether you are optimizing code, debugging imports, or building tools that analyze and transform Python itself.