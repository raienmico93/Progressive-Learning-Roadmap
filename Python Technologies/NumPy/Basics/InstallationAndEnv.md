# Installation and Environment

Setting up NumPy correctly is the first step toward efficient numerical computing in Python. This guide covers installation methods, importing conventions, version checking, interactive environments, documentation lookup, and IDE integration.

---

## 1. Installing NumPy

The recommended installation method depends on your workflow. The two primary package managers are **pip** and **conda**.

> "The two main tools that install Python packages are `pip` and `conda`."

### 1.1 Installation via `pip`

**pip** is the standard package manager for Python. It installs packages from the **Python Package Index (PyPI)**.

```bash
pip install numpy
```

> "The most common method involves using the pip package manager. `pip install numpy`."

**Installing in a virtual environment (recommended):**

> "Tip: Use a virtual environment for better management of Python dependencies."

```bash
# Create a virtual environment
python -m venv my-env

# Activate it
source my-env/bin/activate    # macOS/Linux
my-env\Scripts\activate       # Windows

# Install NumPy
pip install numpy
```

**Using `uv` (modern alternative):**

> "uv: A modern Python package manager designed for speed and simplicity."

```bash
uv pip install numpy
```

**Upgrading an existing installation:**

```bash
pip install --upgrade numpy
```

**Installing a specific version:**

```bash
pip install numpy==1.26.4
```

### 1.2 Installation via Conda

**Conda** is a multi-language package manager that installs from its own channels (**defaults** or **conda-forge**). It is preferred in scientific computing because it handles complex dependencies and non-Python libraries.

> "Conda: If you use conda, you can install NumPy from the default channel or conda-forge."

```bash
# Create and activate an environment
conda create -n my-env
conda activate my-env

# Install NumPy
conda install numpy
```

**Installing from conda-forge:**

```bash
conda install -c conda-forge numpy
```

**For scientific computing environments:**

> "For scientific computing environments, consider installing via Anaconda: `conda install numpy`."

### 1.3 Pip vs. Conda — Key Differences

| Aspect | pip | conda |
|---|---|---|
| **Package source** | PyPI (Python Package Index) | Conda channels (defaults, conda-forge) |
| **Languages** | Python only | Python, R, C/C++, Fortran |
| **Environment management** | Requires separate tools (venv, virtualenv) | Built-in |
| **Binary dependencies** | Wheels include binaries | Manages non-Python libraries |
| **Speed** | Fast for pure-Python packages | Slower but more thorough |

> "conda is multi-language and can install Python, while pip is installed into a particular Python on your system and installs other packages only for that same Python installation."

> "conda is an integrated solution for managing packages, dependencies, and environments, while with pip you may need another tool to handle environments or complex dependencies."

### 1.4 System Package Managers (Not Recommended)

> "Not recommended for most users, but available for convenience."

| System | Command |
|---|---|
| macOS (Homebrew) | `brew install numpy` |
| Linux (APT) | `sudo apt install python3-numpy` |
| Windows (Chocolatey) | `choco install numpy` |

**Why avoid:** System package managers often lag behind PyPI/conda versions and may conflict with virtual environments.

### 1.5 Installing from Source

> "For advanced users and developers who want to customize or debug NumPy."

Building from source requires a C compiler, Fortran compiler, and BLAS/LAPACK libraries. This is rarely necessary for typical use.

---

## 2. Importing NumPy

### 2.1 The Standard Import Convention

The universal convention is to import NumPy with the alias **`np`**:

```python
import numpy as np
```

> "This widespread convention allows access to NumPy features with a short, recognizable prefix `np`. I recommend you stick to it!"

### 2.2 Why `np`?

| Reason | Explanation |
|---|---|
| **Brevity** | `np.array()` is shorter than `numpy.array()` |
| **Convention** | Nearly all NumPy code, tutorials, and documentation uses `np` |
| **Readability** | Experienced Python developers instantly recognize `np` |
| **Community** | SciPy, Pandas, and other libraries assume this alias |

### 2.3 Importing Specific Functions

```python
# Import everything (generally discouraged)
from numpy import *

# Import specific functions
from numpy import array, zeros, linspace

# Import submodules
from numpy import linalg as LA
from numpy import random as rng
```

**Best practice:** Prefer `import numpy as np` over `from numpy import *` to avoid namespace pollution.

### 2.4 Verifying the Import

```python
import numpy as np

print(np.__version__)
# 2.2.0
```

---

## 3. Checking the Installed Version

### 3.1 Using `np.__version__`

```python
import numpy as np
print(np.__version__)
```

> "Import numpy and print its `__version__` attribute."

**Example output:**

```
2.2.0
```

### 3.2 From the Command Line

```bash
python -c "import numpy; print(numpy.__version__)"
```

> "python -c 'import numpy; numpy.__version__'"

### 3.3 Using `pip show`

```bash
pip show numpy
```

Output:

```
Name: numpy
Version: 2.2.0
Summary: Fundamental package for array computing in Python
...
```

### 3.4 Using `conda list`

```bash
conda list numpy
```

Output:

```
# packages in environment at /opt/anaconda3:
#
# Name                    Version                   Build  Channel
numpy                     2.2.0           py312h...  conda-forge
```

### 3.5 Why Version Matters

| Concern | Reason |
|---|---|
| **API compatibility** | NumPy 2.0 introduced breaking changes |
| **Feature availability** | New functions are added over time |
| **Bug fixes** | Older versions may have known bugs |
| **Dependency requirements** | Pandas, SciPy, etc. require specific NumPy versions |

---

## 4. Interactive Environments

Interactive environments are essential for exploratory data analysis, debugging, and learning.

### 4.1 Jupyter Notebook

**Jupyter Notebook** is a web-based interactive computing environment where you can combine code, output, visualizations, and narrative text.

**Starting Jupyter Notebook:**

```bash
jupyter notebook
```

> "If you have the Jupyter package installed, you can run the command 'jupyter notebook' or if you installed Anaconda, run its 'Jupyter Notebook'."

**Basic workflow:**

1. Open a notebook in the browser
2. Import NumPy in a cell
3. Run cells with **Shift+Enter**

```python
import numpy as np

# Create an array
arr = np.array([1, 2, 3, 4, 5])
print(arr)
# [1 2 3 4 5]
```

**Why Jupyter for NumPy:**

| Feature | Benefit |
|---|---|
| **Cell-based execution** | Run code in small chunks |
| **Inline output** | See results immediately |
| **Visualization** | Plot arrays with Matplotlib inline |
| **Narrative** | Mix code with Markdown explanations |
| **Persistence** | Save notebooks as `.ipynb` files |

**Setting the kernel:**

> "Open a notebook in the browser, select the environment's Python kernel if prompted, and run the cells from top to bottom."

```bash
# Register a virtual environment as a Jupyter kernel
pip install ipykernel
python -m ipykernel install --user --name=my-env
```

### 4.2 JupyterLab

**JupyterLab** is the next-generation interface for Jupyter — a full IDE-like environment with tabs, file browser, terminal, and multiple notebooks.

**Starting JupyterLab:**

```bash
jupyter lab
```

> "JupyterLab ... it is designed to be a full-featured development environment for data science."

**Differences from Notebook:**

| Aspect | Jupyter Notebook | JupyterLab |
|---|---|---|
| **Interface** | Single notebook per tab | Tabs, split panes |
| **File browser** | Limited | Full-featured |
| **Terminal** | Not built-in | Built-in |
| **Extensions** | Limited | Rich extension system |
| **Status** | Legacy | Current |

**Both use the same `.ipynb` format** and the same kernels.

### 4.3 Python REPL

The **Python REPL** (Read-Eval-Print Loop) is the interactive Python shell.

**Starting the REPL:**

```bash
python
```

Or with IPython (recommended for NumPy):

```bash
ipython
```

> "IPython: A REPL, much like the command window in MATLAB. Lets you write Python on the fly, debug and check the performance of your code, and much more!"

**Basic REPL usage:**

```python
>>> import numpy as np
>>> arr = np.array([1, 2, 3])
>>> arr
array([1, 2, 3])
>>> arr.sum()
6
```

**IPython advantages:**

| Feature | Benefit |
|---|---|
| **Tab completion** | `np.<TAB>` lists all functions |
| **Introspection** | `np.cos?` shows docstring |
| **Magic commands** | `%timeit`, `%run`, `%paste` |
| **Syntax highlighting** | Colored output |
| **History** | Persistent command history |

### 4.4 Comparing Interactive Environments

| Feature | Python REPL | IPython | Jupyter Notebook | JupyterLab |
|---|---|---|---|---|
| **Setup** | Built-in | `pip install ipython` | `pip install notebook` | `pip install jupyterlab` |
| **Tab completion** | Limited | Yes | Yes | Yes |
| **Inline plots** | No | No (terminal) | Yes | Yes |
| **Markdown** | No | No | Yes | Yes |
| **File browser** | No | No | Limited | Yes |
| **Best for** | Quick tests | Advanced REPL | Analysis, sharing | Full IDE experience |

### 4.5 VS Code with Jupyter

> "In VS Code, create a new file `numpy.ipynb`, and open it to run Python interactively."

VS Code supports Jupyter notebooks natively:

1. Install the **Jupyter** extension
2. Create a `.ipynb` file
3. Select a kernel (Python environment with NumPy)
4. Run cells directly in VS Code

---

## 5. Documentation Lookup

NumPy has extensive built-in documentation accessible from any Python environment.

### 5.1 Using `help()`

> "Use the built-in `help` function to view a function's docstring."

```python
import numpy as np

help(np.sort)
```

Output:

```
Help on function sort in module numpy:

sort(a, axis=-1, kind=None, order=None)
    Return a sorted copy of an array.
    ...
```

### 5.2 Using `np.info()`

> "For some objects, `np.info(obj)` may provide additional help. This is particularly true if you see the line 'Help on ufunc object:' at the top of the help() page. Ufuncs are implemented in C, not Python, for speed."

```python
np.info(np.sin)
```

Output:

```
sin(x, /, out=None, *, where=True, casting='same_kind', order='K', dtype=None, subok=True[, signature])

    Trigonometric sine, element-wise.
    ...
```

**When to use `np.info()` over `help()`:** For **ufuncs** (universal functions like `np.sin`, `np.cos`, `np.exp`), which are implemented in C and may not display fully with Python's `help()`.

### 5.3 Searching with `np.lookfor()`

> "To search for documents containing a keyword, do: `np.lookfor('keyword')`."

```python
np.lookfor('cosine')
```

Output:

```
Search results for 'cosine'
---------------------------
numpy.cos
    Cosine element-wise.
numpy.arccos
    Inverse cosine element-wise.
numpy.cosh
    Hyperbolic cosine, element-wise.
...
```

### 5.4 Browsing General Documentation

> "General-purpose documents like a glossary and help on the basic concepts of numpy are available under the `doc` sub-module."

```python
from numpy import doc
help(doc)
```

### 5.5 IPython Introspection

In IPython or Jupyter:

```python
np.cos?      # show docstring
np.cos??     # show source code
np.*cos*?    # search for functions matching pattern
```

> "To view the docstring for a function, use `np.cos?<ENTER>` (to view the docstring) and `np.cos??<ENTER>` (to view the source code)."

### 5.6 Online Documentation

| Resource | URL |
|---|---|
| **NumPy homepage** | [https://numpy.org](https://numpy.org) |
| **NumPy reference guide** | [https://numpy.org/doc/stable/reference/](https://numpy.org/doc/stable/reference/) |
| **NumPy user guide** | [https://numpy.org/doc/stable/user/](https://numpy.org/doc/stable/user/) |
| **NumPy for MATLAB users** | [https://numpy.org/doc/stable/user/numpy-for-matlab-users.html](https://numpy.org/doc/stable/user/numpy-for-matlab-users.html) |
| **SciPy documentation** | [http://docs.scipy.org/doc/](http://docs.scipy.org/doc/) |

### 5.7 Version-Specific Documentation

```python
np.__version__  # check your version
```

Use the version-specific docs at `https://numpy.org/doc/<version>/` for accuracy.

---

## 6. Working with NumPy in IDEs

### 6.1 PyCharm

**Setup:**

1. **Open project settings:** `File → Settings → Project → Python Interpreter` (Windows/Linux) or `PyCharm → Settings → Project → Python Interpreter` (macOS)
2. **Select interpreter:** Choose the environment where NumPy is installed
3. **Verify installation:** Check that NumPy appears in the package list

> "In PyCharm: Go to File > Settings > Project > Python Interpreter and check if NumPy is listed."

**Common issue with Anaconda:**

> "There are fairly common issues when using PyCharm together with Anaconda, please see the PyCharm support."

**PyCharm + Jupyter:**

PyCharm Professional supports Jupyter notebooks. Select the notebook kernel matching your NumPy environment.

### 6.2 VS Code

**Setup:**

1. **Install the Python extension** from the VS Code marketplace
2. **Open the Command Palette** (`Ctrl+Shift+P` on Windows/Linux, `Cmd+Shift+P` on macOS)
3. **Select interpreter:** Type `Python: Select Interpreter` and choose the environment with NumPy

> "Press Ctrl+Shift+P (Win/Linux) or Cmd+Shift+P (macOS), type 'Python..."

**Common issue with Anaconda:**

> "A commonly reported issue is related to the environment activation within VSCode. Please see the VSCode support for information on how to correctly set up VSCode with virtual environments or conda."

**VS Code + Jupyter:**

1. Install the **Jupyter** extension
2. Create a `.ipynb` file
3. Select the kernel with NumPy installed
4. Run cells directly in the editor

> "In VS Code, create a new file `numpy.ipynb`, and open it to run Python interactively."

### 6.3 Common IDE Issues — `ModuleNotFoundError: No module named 'numpy'`

This error almost always means the IDE is using a **different Python interpreter** than the one where NumPy is installed.

> "A quick command-line test is useful because it removes the IDE from the equation: if that works in the terminal, the package is fine and the IDE configuration is the part that needs correction."

**Diagnostic steps:**

```bash
# 1. Check which Python is used in the terminal
which python        # macOS/Linux
where python        # Windows

# 2. Verify NumPy is available in that Python
python -c "import numpy; print(numpy.__version__)"

# 3. In the IDE, check the selected interpreter
import sys
print(sys.executable)
```

**Fix:** In the IDE, select the **same interpreter** that works in the terminal.

> "Check the selected interpreter in VS Code, PyCharm, or Jupyter and make sure it matches the environment where NumPy was installed."

### 6.4 IDE-Specific Tips

| IDE | Feature | Benefit |
|---|---|---|
| **PyCharm** | Scientific mode | Integrated plots, arrays viewer |
| **PyCharm** | Jupyter support | Run notebooks in IDE |
| **VS Code** | Python Interactive | Run code in cells |
| **VS Code** | Jupyter extension | Full notebook support |
| **VS Code** | Pylance | Type hints for NumPy |
| **Spyder** | Variable explorer | Inspect arrays visually |
| **Spyder** | IPython console | Built-in REPL |

### 6.5 Spyder — Specialized Scientific IDE

**Spyder** (Scientific Python Development Environment) is designed specifically for data science:

- **Variable explorer** — view NumPy arrays in a table
- **IPython console** — integrated REPL
- **Plots pane** — inline visualizations
- **Debugger** — step through array operations

Included with **Anaconda**.

---

## Summary Table

| Topic | Key Points |
|---|---|
| **pip install** | `pip install numpy` — standard package manager |
| **conda install** | `conda install numpy` — preferred for scientific computing |
| **Virtual environments** | Always use `venv` or `conda` environments |
| **Import convention** | `import numpy as np` |
| **Version check** | `np.__version__` or `pip show numpy` |
| **Jupyter Notebook** | Cell-based interactive computing |
| **JupyterLab** | IDE-like notebook environment |
| **Python REPL** | `python` or `ipython` |
| **Help** | `help(np.sort)` |
| **Info** | `np.info(np.sin)` — for ufuncs |
| **Search** | `np.lookfor('keyword')` |
| **IPython introspection** | `np.cos?`, `np.cos??` |
| **PyCharm** | Select interpreter in Project Settings |
| **VS Code** | Select interpreter via Command Palette |
| **Common issue** | IDE using wrong interpreter → `ModuleNotFoundError` |

---

## Key Takeaways

1. **Install NumPy** with `pip install numpy` for general use, or `conda install numpy` for scientific computing.
2. **Always use a virtual environment** — it isolates dependencies and prevents conflicts.
3. **Import with `import numpy as np`** — this is the universal convention.
4. **Check the version** with `np.__version__` to ensure compatibility.
5. **Jupyter Notebook** and **JupyterLab** are ideal for exploratory data analysis with NumPy.
6. **IPython** enhances the REPL with tab completion, introspection, and magic commands.
7. **Documentation is built in** — use `help()`, `np.info()`, and `np.lookfor()`.
8. **`np.info()` is essential for ufuncs** (e.g., `np.sin`, `np.cos`) which are implemented in C.
9. **IDE configuration matters** — always select the interpreter where NumPy is installed.
10. **`ModuleNotFoundError`** almost always means the IDE is using the wrong interpreter — check `sys.executable`.
11. **VS Code and PyCharm** both support Jupyter notebooks natively for interactive NumPy work.
12. **Spyder** offers a specialized scientific IDE with a variable explorer for NumPy arrays.

---

Would you like me to continue with the next topic — **NumPy Arrays**, **NumPy Array Creation**, or **NumPy Indexing and Slicing**? I can format the next section in the same style.