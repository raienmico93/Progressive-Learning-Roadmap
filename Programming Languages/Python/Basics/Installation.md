# Python Installation and Environment

This section explains how to install Python, configure your system, run Python code, and choose an appropriate development environment.

---

## Installing Python

Python can be installed in several ways depending on your operating system.

### Windows

Recommended options:

1. **Official installer** from [python.org](https://www.python.org/downloads/)
   - Download the latest Python 3.x installer.
   - During installation, check **“Add python.exe to PATH”**.
   - Choose **Customize installation** if you need to select optional features.

2. **Microsoft Store**
   - Simple installation and automatic updates.
   - Good for beginners, but sometimes less flexible for advanced configuration.

3. **Package managers**
   - `winget install Python.Python.3.12`
   - `choco install python`
   - `scoop install python`

### macOS

Recommended options:

1. **Official installer** from python.org
2. **Homebrew**
   ```bash
   brew install python
   ```
3. **Xcode Command Line Tools**
   - Provides a system Python 3, but avoid using it for project dependencies.

### Linux

Python 3 is usually preinstalled. Install or update it with your package manager:

```bash
# Debian/Ubuntu
sudo apt update
sudo apt install python3 python3-pip python3-venv

# Fedora
sudo dnf install python3 python3-pip

# Arch
sudo pacman -S python python-pip
```

> Avoid modifying the system Python used by your operating system. Use virtual environments or version managers for project work.

### Verify Installation

```bash
python --version
# or
python3 --version
```

Example output:

```text
Python 3.12.3
```

---

## Python Interpreter

The **Python interpreter** is the program that executes Python code. In the most common implementation, **CPython**, the process is roughly:

1. Read Python source code.
2. Compile it to bytecode.
3. Execute the bytecode on the Python Virtual Machine.

The interpreter can be started in several ways:

```bash
python
python3
py
```

It can also run files, modules, and one-liners.

---

## Python Launcher

On Windows, the **Python Launcher** is a tool called `py`. It helps select between multiple installed Python versions.

Examples:

```bash
py --version
py --list
py -3.12 --version
py -3.11 script.py
py -3 -m venv .venv
```

The launcher can also read a shebang line such as:

```python
#! python3
```

and choose the appropriate interpreter.

---

## `python` / `python3`

The command name depends on your platform and installation:

| Platform | Common command | Notes |
|---|---|---|
| Windows | `python` or `py` | `py` is often safer when multiple versions are installed. |
| macOS/Linux | `python3` | `python` may not exist or may refer to Python 2 on older systems. |

Examples:

```bash
python3 --version
python3 script.py
python3 -m venv .venv
```

On Windows:

```bash
python --version
py -3.12 script.py
```

---

## Python Version Management

Different projects may require different Python versions. Version managers help install and switch between them.

Popular tools:

- **pyenv** – macOS/Linux; `pyenv-win` for Windows
- **conda** – data science and scientific computing
- **asdf** – multi-language version manager
- **uv** – fast Python package and project manager
- **Windows py launcher** – simple switching via `py -3.x`

Example with `pyenv`:

```bash
pyenv install 3.12.3
pyenv install 3.11.9
pyenv global 3.12.3
pyenv local 3.11.9
```

### Virtual Environments

Virtual environments isolate project dependencies.

Create one:

```bash
python3 -m venv .venv
```

Activate it:

```bash
# macOS/Linux
source .venv/bin/activate

# Windows PowerShell
.venv\Scripts\Activate.ps1

# Windows CMD
.venv\Scripts\activate
```

Deactivate:

```bash
deactivate
```

Common tools: `venv`, `virtualenv`, `conda`, `pipenv`, `poetry`, `uv`.

---

## Environment Variables

Environment variables control how Python and your shell find and run programs.

Important Python-related variables:

| Variable | Purpose |
|---|---|
| `PATH` | Directories searched for executables such as `python`. |
| `PYTHONPATH` | Extra directories added to Python’s module search path. |
| `PYTHONHOME` | Alternative Python installation location. Rarely used. |
| `PYTHONSTARTUP` | Script executed when the interactive interpreter starts. |
| `PYTHONIOENCODING` | Encoding for stdin/stdout/stderr. |
| `PYTHONDONTWRITEBYTECODE` | Prevents `.pyc` files from being written. |
| `PYTHONUNBUFFERED` | Forces stdout/stderr to be unbuffered. |
| `VIRTUAL_ENV` | Set automatically when a virtual environment is active. |

Example:

```bash
export PYTHONPATH="/home/user/my_modules:$PYTHONPATH"
```

> Prefer installing packages into a virtual environment instead of relying heavily on `PYTHONPATH`.

---

## PATH Configuration

`PATH` is a list of directories your shell searches when you type a command.

If `python` or `python3` is not recognized, the Python installation directory is probably not in `PATH`.

### Windows

During installation, check **Add python.exe to PATH**.

Manual PATH entries usually include:

```text
C:\Users\<You>\AppData\Local\Programs\Python\Python312\
C:\Users\<You>\AppData\Local\Programs\Python\Python312\Scripts\
```

Check with:

```cmd
where python
echo %PATH%
```

### macOS/Linux

Add Python or version-manager paths to your shell configuration, such as `~/.zshrc` or `~/.bashrc`:

```bash
export PATH="$HOME/.local/bin:$PATH"
```

For Homebrew:

```bash
export PATH="/opt/homebrew/bin:$PATH"
```

Check with:

```bash
which python3
echo $PATH
```

---

## Interactive Interpreter

The interactive interpreter lets you type Python code and see results immediately.

Start it:

```bash
python3
```

You will see:

```text
Python 3.12.3 (main, ...)
Type "help", "copyright", "credits" or "license" for more information.
>>>
```

Example:

```python
>>> 2 + 2
4
>>> name = "Python"
>>> print(f"Hello, {name}")
Hello, Python
```

Exit with:

```python
exit()
```

or `quit()`, or press `Ctrl+D` on macOS/Linux, `Ctrl+Z` then Enter on Windows.

---

## Python REPL

REPL stands for **Read-Eval-Print Loop**. It is the interactive mode described above.

The basic Python REPL is useful for:

- Quick calculations
- Testing small code snippets
- Exploring modules and objects
- Debugging

Enhanced REPLs:

- **IPython**
- **bpython**
- **ptpython**

Example:

```bash
pip install ipython
ipython
```

---

## Integrated Development Environments

IDEs provide editing, running, debugging, and project management tools.

| IDE | Description | Best for |
|---|---|---|
| **VS Code** | Lightweight editor with Python, Pylance, Jupyter, and debugger extensions. | General development, data science, web. |
| **PyCharm** | Full-featured Python IDE. Community and Professional editions. | Large projects, professional development. |
| **IDLE** | Simple editor and REPL bundled with Python. | Beginners and quick experiments. |

Other options: Spyder, Thonny, Sublime Text, Vim/Neovim, Emacs.

### VS Code Setup

1. Install VS Code.
2. Install the **Python** extension.
3. Optionally install **Pylance** and **Jupyter**.
4. Select your interpreter:
   - `Ctrl+Shift+P` → **Python: Select Interpreter**
5. Run a file with the Run button or:
   ```bash
   python3 script.py
   ```

### PyCharm Setup

1. Install PyCharm.
2. Create a new project.
3. Configure a virtual environment.
4. Run and debug directly from the IDE.

### IDLE

IDLE is included with the standard Python installation.

Start it from the Start menu, Applications menu, or:

```bash
idle
```

It provides a shell window and an editor window.

---

## Jupyter Environments

Jupyter is widely used for data analysis, scientific computing, and machine learning.

Install:

```bash
pip install notebook jupyterlab
```

Start:

```bash
jupyter notebook
# or
jupyter lab
```

Features:

- Cell-based execution
- Rich output: charts, images, tables, HTML
- Kernel-based execution
- Supports Python, R, Julia, and more

Other Jupyter environments:

- Google Colab
- VS Code notebooks
- Kaggle Notebooks

---

## Running `.py` Files

To run a Python file:

```bash
python3 script.py
```

On Windows:

```bash
python script.py
py script.py
```

Example `script.py`:

```python
print("Running a Python file")
```

Run it:

```bash
python3 script.py
```

### Shebang on macOS/Linux

Add this to the first line:

```python
#!/usr/bin/env python3
```

Then make it executable:

```bash
chmod +x script.py
./script.py
```

### Run a One-Liner

```bash
python3 -c "print('Hello from -c')"
```

### Run a File and Enter Interactive Mode

```bash
python3 -i script.py
```

---

## Running Modules

The `-m` flag runs a module as a script.

Examples:

```bash
python3 -m http.server
python3 -m venv .venv
python3 -m pip install requests
python3 -m json.tool data.json
```

A module can detect whether it is run directly:

```python
def main():
    print("Running as a script")

if __name__ == "__main__":
    main()
```

When imported, `__name__` is the module name. When run directly, it is `"__main__"`.

---

## Executing Code Interactively

You can execute Python code interactively in several ways:

1. **Basic REPL**
   ```bash
   python3
   ```

2. **IPython**
   ```bash
   ipython
   ```

3. **Jupyter Notebook/Lab**
   ```bash
   jupyter lab
   ```

4. **IDE Python Console**
   - VS Code: Python Interactive Window
   - PyCharm: Python Console

5. **Run then stay interactive**
   ```bash
   python3 -i script.py
   ```

6. **Dynamic execution**
   ```python
   exec("x = 10")
   print(x)
   ```

Use `exec()` and `eval()` carefully, especially with untrusted input.

---

## Quick Start Checklist

1. Install Python 3.x.
2. Verify:
   ```bash
   python3 --version
   ```
3. Install `pip` if needed:
   ```bash
   python3 -m ensurepip --upgrade
   ```
4. Create a project folder.
5. Create and activate a virtual environment:
   ```bash
   python3 -m venv .venv
   source .venv/bin/activate   # macOS/Linux
   ```
6. Install packages:
   ```bash
   python3 -m pip install requests
   ```
7. Run code:
   ```bash
   python3 script.py
   ```

A properly configured Python environment makes development smoother, more reproducible, and easier to debug across projects.