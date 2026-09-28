# Python Standard Library: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Core Definition

The Python Standard Library is a vast collection of modules and packages included with every Python installation, providing implementations of common programming tasks such as mathematics, file handling, data serialization, system interaction, and more. These modules are written in Python or C and are available without any additional installation.

### Technical Definition

The Python Standard Library is distributed with the CPython reference implementation and documented in the official Python documentation. It includes over 200 modules covering areas such as text processing, data structures, numeric and mathematical operations, file and directory access, persistence, compression, cryptography, operating system services, concurrency, networking, internet data handling, structured markup processing, and development tools. Many modules are implemented in C for performance, while others are pure Python.

### Beginner-Friendly Explanation

The Python Standard Library is a big box of tools that comes free with Python. Instead of writing everything from scratch, you can use these pre-built modules to do things like work with dates, read files, do math, generate random numbers, handle JSON, parse command-line arguments, and much more. They are always available, so you can just `import` them and start using them.

### Key Characteristics

- **Batteries Included**: Python ships with a comprehensive standard library, reducing the need for third-party packages.
- **Portable**: Most modules work across Windows, macOS, and Linux.
- **Documented**: Every module is thoroughly documented in the official Python documentation.
- **Gradually enhanced**: New modules and features are added in each Python release (e.g., `zoneinfo` in 3.9, `tomllib` in 3.11).
- **Optional modules**: Some modules (e.g., `sqlite3`) are optional and may depend on system libraries.

### Prerequisites

- Python 3.x installed (some modules require Python 3.9+, 3.10+, or 3.11+).
- Basic knowledge of Python syntax and data types.
- A text editor or IDE.

### Related Programming Areas

- **Data science**: `math`, `statistics`, `random`, `decimal`.
- **Web development**: `json`, `csv`, `sqlite3`, `datetime`.
- **System administration**: `os`, `sys`, `subprocess`, `shutil`.
- **Functional programming**: `itertools`, `functools`, `collections`.
- **Configuration management**: `tomllib`, `zoneinfo`.

### Core Concepts / Features

The following sections cover each major module group using a uniform structure.

---

## 1. Data & Math

### 1.1 `math`

#### Definitions

**Core Definition**: The `math` module provides access to mathematical functions defined by the C standard.

**Technical Definition**: `math` provides functions for number-theoretic and representation tasks (`ceil`, `floor`, `factorial`, `comb`, `copysign`, `fabs`), power and logarithmic functions (`exp`, `log`, `pow`, `sqrt`), trigonometric functions (`sin`, `cos`, `tan`), angular conversion (`degrees`, `radians`), and constants (`pi`, `e`, `tau`, `inf`, `nan`). These functions cannot be used with complex numbers; the `cmath` module provides complex-number versions.

**Beginner-Friendly Explanation**: The `math` module gives you mathematical functions like square root, sine, cosine, and factorial. Use `math.sqrt(16)` to get `4.0`, or `math.pi` for the value of π.

#### Purposes

- To perform common mathematical calculations.
- To access mathematical constants.
- To round numbers up or down (`ceil`, `floor`).
- To compute factorials, combinations, and permutations.
- To perform trigonometric and logarithmic operations.

#### Syntax Rules and Structure

```python
import math

math.ceil(x)              # Smallest integer >= x
math.floor(x)             # Largest integer <= x
math.factorial(x)         # x! as integer
math.comb(n, k)           # Binomial coefficient (Python 3.8+)
math.sqrt(x)              # Square root
math.pow(x, y)            # x ** y
math.log(x[, base])       # Natural or base log
math.sin(x)               # Sine (radians)
math.cos(x)               # Cosine (radians)
math.pi                   # 3.141592653589793
math.e                    # 2.718281828459045
math.inf                  # Positive infinity
math.nan                  # Not a Number
```

#### Code Examples and Expected Outputs

```python
import math

# step1: Basic operations
print(math.sqrt(16))          # 4.0
print(math.ceil(4.1))         # 5
print(math.floor(4.9))        # 4
print(math.factorial(5))      # 120
print(math.comb(5, 2))        # 10

# step2: Constants
print(math.pi)                # 3.141592653589793
print(math.e)                 # 2.718281828459045

# step3: Trigonometry
print(round(math.sin(math.pi / 2), 4))  # 1.0
print(round(math.cos(0), 4))            # 1.0
```

**Expected Output**:
```
4.0
5
4
120
10
3.141592653589793
2.718281828459045
1.0
1.0
```

**Why**: `math` provides direct C-standard mathematical functions; all return floats except where noted.

#### Real-World Cases

- **Scientific computing**: Trigonometric and logarithmic calculations.
- **Game development**: Physics calculations.
- **Data analysis**: Statistical computations.

#### References

- math — Mathematical functions - https://docs.python.org/3/library/math.html

---

### 1.2 `statistics`

#### Definitions

**Core Definition**: The `statistics` module provides functions for calculating mathematical statistics of numeric data.

**Technical Definition**: `statistics` offers functions for averages and measures of central location (`mean`, `fmean`, `geometric_mean`, `harmonic_mean`, `median`, `mode`), measures of spread (`pstdev`, `pvariance`, `stdev`, `variance`), and statistics for relations between two inputs (`covariance`, `correlation`). It supports `int`, `float`, `Decimal`, and `Fraction`.

**Beginner-Friendly Explanation**: The `statistics` module helps you calculate averages, medians, modes, and standard deviations. Use `statistics.mean([1, 2, 3])` to get `2`.

#### Purposes

- To compute measures of central tendency (mean, median, mode).
- To compute measures of spread (standard deviation, variance).
- To compute relationships between datasets (covariance, correlation).
- To estimate probability density distributions.

#### Syntax Rules and Structure

```python
import statistics

statistics.mean(data)               # Arithmetic mean
statistics.median(data)             # Median
statistics.mode(data)               # Most common value
statistics.stdev(data)              # Sample standard deviation
statistics.pstdev(data)             # Population standard deviation
statistics.variance(data)           # Sample variance
statistics.pvariance(data)          # Population variance
statistics.correlation(x, y)        # Pearson correlation
statistics.covariance(x, y)         # Covariance
statistics.quantiles(data, n=4)     # Divide into intervals
```

#### Code Examples and Expected Outputs

```python
import statistics

data = [2, 4, 4, 4, 5, 5, 7, 9]

# step1: Central tendency
print(statistics.mean(data))        # 5.0
print(statistics.median(data))      # 4.5
print(statistics.mode(data))        # 4

# step2: Spread
print(round(statistics.stdev(data), 4))   # 2.1381
print(round(statistics.pstdev(data), 4))  # 2.0

# step3: Quantiles
print(statistics.quantiles(data, n=4))    # [3.5, 4.5, 6.0]
```

**Expected Output**:
```
5.0
4.5
4
2.1381
2.0
[3.5, 4.5, 6.0]
```

**Why**: `mean` computes the arithmetic average; `median` finds the middle value; `mode` finds the most frequent value; `stdev` and `pstdev` compute sample and population standard deviations.

#### Real-World Cases

- **Data analysis**: Summarizing datasets.
- **Quality control**: Measuring variance in manufacturing.
- **Finance**: Computing average returns and volatility.

#### References

- statistics — Mathematical statistics functions - https://docs.python.org/3/library/statistics.html

---

### 1.3 `random`

#### Definitions

**Core Definition**: The `random` module implements pseudo-random number generators for various distributions.

**Technical Definition**: `random` provides functions for generating random integers (`randint`, `randrange`), random floats (`random`, `uniform`), random selections from sequences (`choice`, `choices`, `sample`), random permutations (`shuffle`), and random values from various distributions (normal, lognormal, gamma, beta, exponential). The module uses the Mersenne Twister as its core generator.

**Beginner-Friendly Explanation**: The `random` module lets you generate random numbers, pick random items from a list, and shuffle sequences. Use `random.randint(1, 10)` to get a random integer between 1 and 10.

#### Purposes

- To generate random numbers for simulations and games.
- To select random items from sequences.
- To shuffle sequences in place.
- To generate random samples without replacement.
- To generate random values from statistical distributions.

#### Syntax Rules and Structure

```python
import random

random.random()                     # Float in [0.0, 1.0)
random.randint(a, b)                # Integer in [a, b]
random.randrange(start, stop[, step])
random.uniform(a, b)                # Float in [a, b]
random.choice(seq)                  # Random element
random.choices(seq, k=n)            # n random elements (with replacement)
random.sample(seq, k=n)             # n unique elements
random.shuffle(seq)                 # Shuffle in place
random.seed(a)                      # Seed the generator
```

#### Code Examples and Expected Outputs

```python
import random

# step1: Seed for reproducibility
random.seed(42)

# step2: Random integers
print(random.randint(1, 10))        # 2
print(random.randrange(0, 100, 5))  # 25

# step3: Random selection
fruits = ["apple", "banana", "cherry"]
print(random.choice(fruits))        # cherry

# step4: Sampling
print(random.sample(fruits, 2))     # ['apple', 'cherry']

# step5: Shuffle
random.shuffle(fruits)
print(fruits)                       # ['banana', 'cherry', 'apple']
```

**Expected Output**:
```
2
25
cherry
['apple', 'cherry']
['banana', 'cherry', 'apple']
```

**Why**: `random.seed()` ensures reproducibility; `randint` and `randrange` generate integers; `choice` and `sample` select from sequences; `shuffle` randomizes order.

#### Real-World Cases

- **Game development**: Random enemy placement, dice rolls.
- **Simulations**: Monte Carlo methods.
- **Testing**: Generating random test data.
- **Security**: NOT suitable for cryptographic purposes (use `secrets` instead).

#### References

- random — Generate pseudo-random numbers - https://docs.python.org/3/library/random.html

---

### 1.4 `decimal`

#### Definitions

**Core Definition**: The `decimal` module provides support for fast correctly rounded decimal floating-point arithmetic.

**Technical Definition**: `decimal` implements the General Decimal Arithmetic Specification, offering exact decimal representation and controlled rounding. It is preferred in accounting applications with strict equality invariants because numbers like `1.1` and `2.2` do not have exact binary representations. The module provides a `Decimal` type with user-alterable precision (default 28 places) and full control over rounding and signal handling.

**Beginner-Friendly Explanation**: The `decimal` module avoids floating-point rounding errors, which is crucial for financial calculations. `Decimal("0.1") + Decimal("0.2")` gives exactly `0.3`, unlike `float` arithmetic.

#### Purposes

- To perform exact decimal arithmetic for financial calculations.
- To avoid binary floating-point representation errors.
- To control rounding behaviour precisely.
- To maintain significant digits (e.g., `1.30 + 1.20 = 2.50`).
- To handle monetary values with strict equality invariants.

#### Syntax Rules and Structure

```python
from decimal import Decimal, getcontext

Decimal("0.1")                      # Create from string
Decimal(0.1)                        # From float (lossy)
getcontext().prec = 10              # Set precision
getcontext().rounding = "ROUND_HALF_UP"

Decimal("0.1") + Decimal("0.2")     # Exact arithmetic
```

#### Code Examples and Expected Outputs

```python
from decimal import Decimal, getcontext

# step1: Floating-point problem
print(0.1 + 0.2)                    # 0.30000000000000004

# step2: Decimal solution
print(Decimal("0.1") + Decimal("0.2"))  # 0.3

# step3: Significant digits
print(Decimal("1.30") + Decimal("1.20"))  # 2.50

# step4: Precision control
getcontext().prec = 6
print(Decimal(1) / Decimal(7))      # 0.142857
```

**Expected Output**:
```
0.30000000000000004
0.3
2.50
0.142857
```

**Why**: `Decimal` uses exact decimal representation; `getcontext().prec` controls the number of significant digits.

#### Real-World Cases

- **Financial applications**: Currency calculations.
- **Accounting**: Ensuring exact equality.
- **Tax calculations**: Avoiding rounding errors.
- **Scientific computing**: Precise decimal arithmetic.

#### References

- decimal — Decimal fixed-point and floating-point arithmetic - https://docs.python.org/3/library/decimal.html

---

## 2. Date & Time

### 2.1 `datetime`

#### Definitions

**Core Definition**: The `datetime` module supplies classes for manipulating dates and times.

**Technical Definition**: `datetime` provides `date`, `time`, `datetime`, `timedelta`, and `tzinfo` classes for date/time arithmetic and formatting. Objects may be "aware" (with timezone information) or "naive" (without). The module focuses on efficient attribute extraction for output formatting and manipulation.

**Beginner-Friendly Explanation**: The `datetime` module lets you work with dates and times, calculate differences, and format dates. Use `datetime.now()` for the current date and time.

#### Purposes

- To represent dates and times.
- To perform date/time arithmetic (adding days, subtracting times).
- To format dates and times as strings.
- To parse date/time strings.
- To work with time zones.

#### Syntax Rules and Structure

```python
from datetime import datetime, date, time, timedelta

datetime.now()                               # Current date and time
datetime(2026, 9, 28, 14, 30)                # Specific datetime
date.today()                                 # Current date
timedelta(days=7)                            # Duration
dt.strftime("%Y-%m-%d")                      # Format as string
datetime.strptime("2026-09-28", "%Y-%m-%d")  # Parse string
```

#### Code Examples and Expected Outputs

```python
from datetime import datetime, timedelta

# step1: Current time
now = datetime(2026, 9, 28, 14, 30, 45)
print(now)                          # 2026-09-28 14:30:45

# step2: Formatting
print(now.strftime("%Y-%m-%d %H:%M:%S"))  # 2026-09-28 14:30:45

# step3: Arithmetic
future = now + timedelta(days=7)
print(future.date())                # 2026-10-05

# step4: Parsing
parsed = datetime.strptime("2026-09-28", "%Y-%m-%d")
print(parsed)                       # 2026-09-28 00:00:00
```

**Expected Output**:
```
2026-09-28 14:30:45
2026-09-28 14:30:45
2026-10-05
2026-09-28 00:00:00
```

**Why**: `strftime` formats dates as strings; `strptime` parses strings into datetime objects; `timedelta` enables date arithmetic.

#### Real-World Cases

- **Logging**: Timestamping events.
- **Scheduling**: Calculating future dates.
- **Data processing**: Parsing timestamps from files.

#### References

- datetime — Basic date and time types - https://docs.python.org/3/library/datetime.html

---

## 3. File System & Formats

### 3.1 `os`

#### Definitions

**Core Definition**: The `os` module provides a portable way of using operating system-dependent functionality.

**Technical Definition**: `os` provides functions for file and directory operations (`listdir`, `mkdir`, `remove`, `rename`), process management (`system`, `getpid`), environment variables (`environ`), and path manipulation (via `os.path`). All functions accepting path names accept both bytes and string objects.

**Beginner-Friendly Explanation**: The `os` module lets you interact with the operating system: list files, create directories, get environment variables, and run system commands.

#### Purposes

- To interact with the file system (list, create, remove files/directories).
- To access environment variables.
- To get and change the current working directory.
- To execute system commands.
- To work with process IDs.

#### Syntax Rules and Structure

```python
import os

os.getcwd()                         # Current working directory
os.listdir(path)                    # List directory contents
os.mkdir(path)                      # Create directory
os.makedirs(path, exist_ok=True)    # Create nested directories
os.remove(path)                     # Remove file
os.rename(src, dst)                 # Rename file
os.environ["KEY"]                   # Environment variable
os.path.join(a, b)                  # Join paths
os.path.exists(path)                # Check existence
```

#### Code Examples and Expected Outputs

```python
import os

# step1: Current directory
print(os.getcwd())                  # /home/user

# step2: List directory
print(os.listdir(".")[:3])          # ['.bashrc', '.profile', 'Documents']

# step3: Path operations
path = os.path.join("/home", "user", "file.txt")
print(path)                         # /home/user/file.txt

# step4: Check existence
print(os.path.exists("/tmp"))       # True
```

**Expected Output**:
```
/home/user
['.bashrc', '.profile', 'Documents']
/home/user/file.txt
True
```

**Why**: `os` provides portable interfaces for OS-dependent operations; `os.path` handles path manipulation.

#### Real-World Cases

- **File management**: Automating file operations.
- **System administration**: Scripting system tasks.
- **Configuration**: Reading environment variables.

#### References

- os — Miscellaneous operating system interfaces - https://docs.python.org/3/library/os.html

---

### 3.2 `sys`

#### Definitions

**Core Definition**: The `sys` module provides access to system-specific parameters and functions that interact closely with the Python interpreter.

**Technical Definition**: `sys` exposes variables such as `argv` (command-line arguments), `path` (module search path), `stdin`/`stdout`/`stderr` (standard streams), and functions such as `exit()` (exit the interpreter), `getrecursionlimit()`, and `setrecursionlimit()`. It is always available.

**Beginner-Friendly Explanation**: The `sys` module lets you access command-line arguments (`sys.argv`), exit the program (`sys.exit()`), and interact with the Python runtime environment.

#### Purposes

- To access command-line arguments.
- To exit the program with a status code.
- To modify the module search path.
- To access standard input/output/error streams.
- To get Python version information.

#### Syntax Rules and Structure

```python
import sys

sys.argv                            # Command-line arguments
sys.exit([code])                    # Exit the interpreter
sys.path                            # Module search path
sys.version                         # Python version string
sys.platform                        # Platform identifier
sys.stdin / sys.stdout / sys.stderr # Standard streams
```

#### Code Examples and Expected Outputs

```python
import sys

# step1: Command-line arguments
print(sys.argv)                     # ['script.py']

# step2: Python version
print(sys.version[:6])              # 3.11.0

# step3: Platform
print(sys.platform)                 # linux

# step4: Exit
# sys.exit(0)                       # Exits with code 0
```

**Expected Output**:
```
['script.py']
3.11.0
linux
```

**Why**: `sys.argv` contains command-line arguments; `sys.version` and `sys.platform` provide runtime information.

#### Real-World Cases

- **CLI tools**: Parsing command-line arguments.
- **Scripting**: Exiting with error codes.
- **Debugging**: Inspecting the runtime environment.

#### References

- sys — System-specific parameters and functions - https://docs.python.org/3/library/sys.html

---

### 3.3 `pathlib`

#### Definitions

**Core Definition**: The `pathlib` module offers classes representing filesystem paths with semantics appropriate for different operating systems.

**Technical Definition**: `pathlib` provides `Path` (concrete path for the current OS) and `PurePath` (purely computational path) classes. Paths support `/` operator for joining, methods for file operations (`read_text`, `write_text`, `exists`, `is_dir`), and globbing (`glob`, `rglob`). Pure paths provide path manipulation without I/O.

**Beginner-Friendly Explanation**: `pathlib` lets you work with file paths as objects instead of strings. Use `Path("/home/user") / "file.txt"` to join paths, and `.read_text()` to read a file.

#### Purposes

- To represent file system paths as objects.
- To join paths in a cross-platform way.
- To read and write files.
- To check file existence and properties.
- To glob (search) for files matching patterns.

#### Syntax Rules and Structure

```python
from pathlib import Path

p = Path("/home/user")              # Create path
p / "file.txt"                      # Join paths
p.exists()                          # Check existence
p.is_dir()                          # Check if directory
p.read_text()                       # Read file content
p.write_text("content")             # Write file
p.glob("*.py")                      # Glob pattern
p.rglob("**/*.py")                  # Recursive glob
```

#### Code Examples and Expected Outputs

```python
from pathlib import Path

# step1: Path construction
p = Path("/home/user")
file = p / "document.txt"
print(file)                         # /home/user/document.txt

# step2: Path properties
print(p.name)                       # user
print(p.parent)                     # /home
print(p.suffix)                     # (empty for directories)

# step3: File operations
print(Path("/tmp").exists())        # True
print(Path("/tmp").is_dir())        # True
```

**Expected Output**:
```
/home/user/document.txt
user
/home

True
True
```

**Why**: `pathlib` provides an object-oriented interface for path manipulation; the `/` operator joins paths; properties like `.name` and `.parent` extract path components.

#### Real-World Cases

- **File management**: Cross-platform file operations.
- **Data processing**: Reading/writing files.
- **Configuration**: Locating config files.

#### References

- pathlib — Object-oriented filesystem paths - https://docs.python.org/3/library/pathlib.html

---

### 3.4 `json`

#### Definitions

**Core Definition**: The `json` module provides JSON encoder and decoder functionality for serializing and deserializing Python objects.

**Technical Definition**: `json` implements the JavaScript Object Notation (JSON) format as specified by RFC 7159 and ECMA-404. It provides `dump`/`dumps` for encoding Python objects to JSON strings, and `load`/`loads` for decoding JSON strings to Python objects. JSON is a lightweight data interchange format that maps cleanly to Python's `dict`, `list`, `str`, `int`, `float`, `bool`, and `None`.

**Beginner-Friendly Explanation**: The `json` module lets you convert Python objects to JSON strings and back. Use `json.dumps({"name": "Alice"})` to get `'{"name": "Alice"}'`.

#### Purposes

- To serialize Python objects to JSON strings.
- To deserialize JSON strings to Python objects.
- To read and write JSON files.
- To exchange data with web APIs.
- To pretty-print JSON for readability.

#### Syntax Rules and Structure

```python
import json

json.dumps(obj)                     # Python object to JSON string
json.dump(obj, fp)                  # Write to file
json.loads(s)                       # JSON string to Python object
json.load(fp)                       # Read from file
json.dumps(obj, indent=4)           # Pretty print
json.dumps(obj, sort_keys=True)     # Sort keys
```

#### Code Examples and Expected Outputs

```python
import json

# step1: Serialize
data = {"name": "Alice", "age": 30, "hobbies": ["reading", "coding"]}
json_str = json.dumps(data, indent=2)
print(json_str)
# {
#   "name": "Alice",
#   "age": 30,
#   "hobbies": [
#     "reading",
#     "coding"
#   ]
# }

# step2: Deserialize
parsed = json.loads(json_str)
print(parsed["name"])              # Alice
print(parsed["hobbies"][0])        # reading
```

**Expected Output**:
```
{
  "name": "Alice",
  "age": 30,
  "hobbies": [
    "reading",
    "coding"
  ]
}
Alice
reading
```

**Why**: `json.dumps` converts Python objects to JSON strings; `json.loads` parses JSON strings back to Python objects.

#### Real-World Cases

- **Web APIs**: Sending/receiving JSON data.
- **Configuration files**: Storing settings.
- **Data pipelines**: Exchanging data between systems.

#### References

- json — JSON encoder and decoder - https://docs.python.org/3/library/json.html

---

### 3.5 `csv`

#### Definitions

**Core Definition**: The `csv` module implements classes to read and write tabular data in CSV format.

**Technical Definition**: `csv` provides `reader` and `writer` objects for reading and writing sequences, and `DictReader`/`DictWriter` for dictionary-based access. The module supports dialect and formatting parameters to handle different CSV variants (delimiters, quoting, line terminators).

**Beginner-Friendly Explanation**: The `csv` module reads and writes CSV files, which are common for spreadsheets and databases. Use `csv.reader(f)` to read rows as lists.

#### Purposes

- To read CSV files into lists or dictionaries.
- To write data to CSV files from lists or dictionaries.
- To handle different CSV dialects (Excel, Unix).
- To process tabular data from spreadsheets.

#### Syntax Rules and Structure

```python
import csv

csv.reader(csvfile)                 # Read rows as lists
csv.writer(csvfile)                 # Write rows from lists
csv.DictReader(csvfile)             # Read rows as dicts
csv.DictWriter(csvfile, fieldnames) # Write rows from dicts
```

#### Code Examples and Expected Outputs

```python
import csv
from io import StringIO

# step1: Reading CSV
data = "name,age\nAlice,30\nBob,25\n"
reader = csv.reader(StringIO(data))
for row in reader:
    print(row)
# ['name', 'age']
# ['Alice', '30']
# ['Bob', '25']

# step2: Writing CSV
output = StringIO()
writer = csv.writer(output)
writer.writerow(["name", "age"])
writer.writerow(["Alice", 30])
print(output.getvalue())
# name,age\r\nAlice,30\r\n
```

**Expected Output**:
```
['name', 'age']
['Alice', '30']
['Bob', '25']
name,age
Alice,30
```

**Why**: `csv.reader` parses CSV rows into lists; `csv.writer` formats lists into CSV rows.

#### Real-World Cases

- **Data import/export**: Moving data between systems.
- **Reporting**: Generating CSV reports.
- **Data analysis**: Loading spreadsheets into Python.

#### References

- csv — CSV File Reading and Writing - https://docs.python.org/3/library/csv.html

---

## 4. Text Processing

### 4.1 `re`

#### Definitions

**Core Definition**: The `re` module provides regular expression matching operations similar to those found in Perl.

**Technical Definition**: `re` supports Unicode and 8-bit strings, with patterns compiled to bytecode for efficient execution. It provides functions for searching (`search`, `match`, `fullmatch`), finding all matches (`findall`, `finditer`), substitution (`sub`, `subn`), and splitting (`split`). Patterns use a rich syntax of metacharacters, character classes, quantifiers, groups, and lookarounds.

**Beginner-Friendly Explanation**: The `re` module lets you search for patterns in text. Use `re.search(r"\d+", text)` to find the first number in a string.

#### Purposes

- To search for patterns in text.
- To extract substrings matching patterns.
- To replace text patterns with new text.
- To validate input formats (email, phone, URLs).
- To tokenize text.

#### Syntax Rules and Structure

```python
import re

re.search(pattern, string)          # First match anywhere
re.match(pattern, string)           # Match at start
re.fullmatch(pattern, string)       # Entire string must match
re.findall(pattern, string)         # All matches as list
re.finditer(pattern, string)        # Iterator of match objects
re.sub(pattern, repl, string)       # Replace matches
re.compile(pattern, flags=0)        # Compile pattern
```

#### Code Examples and Expected Outputs

```python
import re

text = "The price is $49.99 and $10.00"

# step1: Search
match = re.search(r'\$\d+\.\d{2}', text)
print(match.group())                # $49.99

# step2: Find all
print(re.findall(r'\$\d+\.\d{2}', text))  # ['$49.99', '$10.00']

# step3: Substitution
print(re.sub(r'\$\d+\.\d{2}', 'PRICE', text))
# The price is PRICE and PRICE
```

**Expected Output**:
```
$49.99
['$49.99', '$10.00']
The price is PRICE and PRICE
```

**Why**: `re.search` finds the first match; `re.findall` returns all matches; `re.sub` replaces matches.

#### Real-World Cases

- **Log parsing**: Extracting timestamps, IPs.
- **Data validation**: Email, phone, URL formats.
- **Web scraping**: Extracting data from HTML.

#### References

- re — Regular expression operations - https://docs.python.org/3/library/re.html

---

## 5. System & Concurrency

### 5.1 `logging`

#### Definitions

**Core Definition**: The `logging` module defines functions and classes which implement a flexible event logging system for applications and libraries.

**Technical Definition**: `logging` provides a hierarchical logger system with five standard levels (DEBUG, INFO, WARNING, ERROR, CRITICAL), handlers for directing output to different destinations (streams, files, network), formatters for output layout, and filters for message filtering. The module is thread-safe and integrates with third-party modules.

**Beginner-Friendly Explanation**: The `logging` module lets you record messages about your program's execution. Use `logging.info("message")` to log an informational message.

#### Purposes

- To record diagnostic information about program execution.
- To debug applications without using `print()`.
- To log to files, streams, or remote servers.
- To control logging levels dynamically.
- To integrate logging from third-party libraries.

#### Syntax Rules and Structure

```python
import logging

logging.basicConfig(level=logging.INFO)
logging.debug("Debug message")
logging.info("Info message")
logging.warning("Warning message")
logging.error("Error message")
logging.critical("Critical message")
```

#### Code Examples and Expected Outputs

```python
import logging

# step1: Configure logging
logging.basicConfig(
    level=logging.DEBUG,
    format="%(asctime)s - %(levelname)s - %(message)s"
)

# step2: Log messages
logging.debug("This is a debug message")
logging.info("This is an info message")
logging.warning("This is a warning")
logging.error("This is an error")
```

**Expected Output**:
```
2026-09-28 14:30:45,123 - DEBUG - This is a debug message
2026-09-28 14:30:45,124 - INFO - This is an info message
2026-09-28 14:30:45,125 - WARNING - This is a warning
2026-09-28 14:30:45,126 - ERROR - This is an error
```

**Why**: `basicConfig` sets up logging; each log call outputs a formatted message based on the level.

#### Real-World Cases

- **Application debugging**: Tracking program flow.
- **Production monitoring**: Logging errors and warnings.
- **Audit trails**: Recording user actions.

#### References

- logging — Logging facility for Python - https://docs.python.org/3/library/logging.html

---

### 5.2 `subprocess`

#### Definitions

**Core Definition**: The `subprocess` module allows you to spawn new processes, connect to their input/output/error pipes, and obtain their return codes.

**Technical Definition**: `subprocess` replaces older modules like `os.system` and `os.spawn*`. The recommended approach is `subprocess.run()` for most use cases; for more advanced scenarios, the `Popen` class provides fine-grained control over process execution.

**Beginner-Friendly Explanation**: The `subprocess` module lets you run external commands from Python. Use `subprocess.run(["ls", "-la"])` to run the `ls` command.

#### Purposes

- To run shell commands from Python.
- To call external tools (git, ffmpeg, docker).
- To capture command output.
- To automate system administration tasks.
- To manage subprocess lifecycles.

#### Syntax Rules and Structure

```python
import subprocess

subprocess.run(["ls", "-la"])                  # Run command
subprocess.run(["ls"], capture_output=True)    # Capture output
subprocess.run(["echo", "hello"], text=True)   # Text mode
subprocess.run(["cmd"], check=True)            # Raise on error
```

#### Code Examples and Expected Outputs

```python
import subprocess

# step1: Run command and capture output
result = subprocess.run(
    ["echo", "Hello, World!"],
    capture_output=True,
    text=True
)
print(result.stdout)                # Hello, World!
print(result.returncode)            # 0

# step2: Run with check
try:
    subprocess.run(["ls", "/nonexistent"], check=True)
except subprocess.CalledProcessError as e:
    print(f"Command failed with {e.returncode}")
```

**Expected Output**:
```
Hello, World!
0
Command failed with 2
```

**Why**: `subprocess.run` executes a command and returns a `CompletedProcess` object; `capture_output=True` captures stdout/stderr; `check=True` raises an exception on non-zero exit.

#### Real-World Cases

- **DevOps**: Running deployment scripts.
- **Data processing**: Calling external tools.
- **Automation**: Batch file processing.

#### References

- subprocess — Subprocess management - https://docs.python.org/3/library/subprocess.html

---

### 5.3 `argparse`

#### Definitions

**Core Definition**: The `argparse` module makes it easy to write user-friendly command-line interfaces.

**Technical Definition**: `argparse` provides a `ArgumentParser` class that defines what arguments a program requires and automatically parses them from `sys.argv`. It supports positional arguments, optional arguments (flags), subcommands, type conversion, default values, and automatic help/usage message generation.

**Beginner-Friendly Explanation**: The `argparse` module lets you create command-line interfaces with options and arguments. Use `parser.add_argument("--verbose", action="store_true")` to add a flag.

#### Purposes

- To parse command-line arguments.
- To generate help and usage messages.
- To validate argument types.
- To support subcommands (e.g., `git add`, `git commit`).
- To provide default values for optional arguments.

#### Syntax Rules and Structure

```python
import argparse

parser = argparse.ArgumentParser(description="My program")
parser.add_argument("filename", help="Input file")
parser.add_argument("-v", "--verbose", action="store_true")
parser.add_argument("-c", "--count", type=int, default=1)
args = parser.parse_args()
```

#### Code Examples and Expected Outputs

```python
import argparse

# step1: Create parser
parser = argparse.ArgumentParser(
    prog="greet",
    description="Greet someone"
)

# step2: Add arguments
parser.add_argument("name", help="Name to greet")
parser.add_argument("-c", "--count", type=int, default=1,
                    help="Number of greetings")

# step3: Parse (simulate command line)
args = parser.parse_args(["Alice", "--count", "3"])
print(f"Hello {args.name}!" * args.count)
# Hello Alice!
# Hello Alice!
# Hello Alice!
```

**Expected Output**:
```
Hello Alice!
Hello Alice!
Hello Alice!
```

**Why**: `add_argument` defines positional and optional arguments; `parse_args` parses `sys.argv` (or a provided list) and returns a namespace.

#### Real-World Cases

- **CLI tools**: Building user-friendly command-line utilities.
- **Scripts**: Adding options to automation scripts.
- **Data pipelines**: Configuring processing parameters.

#### References

- argparse — Parser for command-line options, arguments and subcommands - https://docs.python.org/3/library/argparse.html

---

## 6. Databases

### 6.1 `sqlite3`

#### Definitions

**Core Definition**: The `sqlite3` module provides an SQL interface compliant with the DB-API 2.0 specification for SQLite databases.

**Technical Definition**: SQLite is a C library that provides a lightweight disk-based database that doesn't require a separate server process. The `sqlite3` module implements PEP 249 (DB-API 2.0) and provides `Connection` and `Cursor` objects for executing SQL statements and fetching results.

**Beginner-Friendly Explanation**: The `sqlite3` module lets you work with SQLite databases in Python. Use `sqlite3.connect("mydb.db")` to open a database and `cursor.execute("SELECT * FROM table")` to run queries.

#### Purposes

- To create and query SQLite databases.
- To store application data persistently.
- To prototype applications before migrating to larger databases.
- To perform SQL operations without a database server.
- To embed a database in applications.

#### Syntax Rules and Structure

```python
import sqlite3

con = sqlite3.connect("database.db")    # Connect/create database
cur = con.cursor()                      # Create cursor
cur.execute("CREATE TABLE ...")         # Execute SQL
cur.execute("INSERT INTO ...", data)    # Parameterized query
cur.executemany("INSERT ...", rows)     # Multiple rows
cur.fetchall()                          # Fetch all results
con.commit()                            # Commit transaction
con.close()                             # Close connection
```

#### Code Examples and Expected Outputs

```python
import sqlite3

# step1: Create in-memory database
con = sqlite3.connect(":memory:")
cur = con.cursor()

# step2: Create table
cur.execute("CREATE TABLE movies(title, year, score)")

# step3: Insert data
cur.execute("INSERT INTO movies VALUES (?, ?, ?)",
            ("Blade Runner", 1982, 8.1))
cur.execute("INSERT INTO movies VALUES (?, ?, ?)",
            ("Alien", 1979, 8.5))

# step4: Query
cur.execute("SELECT * FROM movies WHERE score > ?", (8.0,))
for row in cur.fetchall():
    print(row)
# ('Blade Runner', 1982, 8.1)
# ('Alien', 1979, 8.5)

con.close()
```

**Expected Output**:
```
('Blade Runner', 1982, 8.1)
('Alien', 1979, 8.5)
```

**Why**: `sqlite3.connect` creates a connection; `cursor.execute` runs SQL; parameterized queries use `?` placeholders to prevent SQL injection.

#### Real-World Cases

- **Small applications**: Local data storage.
- **Prototyping**: Developing before migrating to PostgreSQL.
- **Data analysis**: Querying datasets with SQL.
- **Embedded systems**: Lightweight database.

#### References

- sqlite3 — DB-API 2.0 interface for SQLite databases - https://docs.python.org/3/library/sqlite3.html

---

## 7. Functional Utilities

### 7.1 `itertools`

#### Definitions

**Core Definition**: The `itertools` module implements a number of iterator building blocks inspired by constructs from APL, Haskell, and SML.

**Technical Definition**: `itertools` provides fast, memory-efficient tools for creating iterators. Functions are grouped into infinite iterators (`count`, `cycle`, `repeat`), terminating iterators (`accumulate`, `chain`, `compress`, `dropwhile`, `filterfalse`, `groupby`, `islice`, `pairwise`, `starmap`, `takewhile`, `tee`, `zip_longest`), and combinatoric iterators (`product`, `permutations`, `combinations`, `combinations_with_replacement`).

**Beginner-Friendly Explanation**: The `itertools` module gives you tools for working with iterators. Use `itertools.count(10)` to create an infinite counter starting at 10, or `itertools.chain([1,2], [3,4])` to combine iterables.

#### Purposes

- To create infinite iterators (`count`, `cycle`, `repeat`).
- To combine iterables (`chain`, `zip_longest`).
- To filter iterables (`dropwhile`, `filterfalse`, `takewhile`).
- To group data (`groupby`).
- To generate combinations and permutations.

#### Syntax Rules and Structure

```python
import itertools

itertools.count(start, step)        # Infinite counter
itertools.cycle(iterable)           # Infinite cycle
itertools.repeat(obj, times)        # Repeat object
itertools.chain(*iterables)         # Combine iterables
itertools.islice(iterable, stop)    # Slice an iterator
itertools.groupby(iterable, key)    # Group by key
itertools.product(*iterables)       # Cartesian product
itertools.permutations(iterable, r) # Permutations
itertools.combinations(iterable, r) # Combinations
```

#### Code Examples and Expected Outputs

```python
import itertools

# step1: Chain
print(list(itertools.chain([1, 2], [3, 4])))  # [1, 2, 3, 4]

# step2: Combinations
print(list(itertools.combinations([1, 2, 3], 2)))
# [(1, 2), (1, 3), (2, 3)]

# step3: Product
print(list(itertools.product([1, 2], ["a", "b"])))
# [(1, 'a'), (1, 'b'), (2, 'a'), (2, 'b')]

# step4: Groupby
data = [("a", 1), ("a", 2), ("b", 3)]
for key, group in itertools.groupby(data, key=lambda x: x[0]):
    print(key, list(group))
# a [('a', 1), ('a', 2)]
# b [('b', 3)]
```

**Expected Output**:
```
[1, 2, 3, 4]
[(1, 2), (1, 3), (2, 3)]
[(1, 'a'), (1, 'b'), (2, 'a'), (2, 'b')]
a [('a', 1), ('a', 2)]
b [('b', 3)]
```

**Why**: `itertools` provides composable iterator building blocks; functions return iterators, not lists.

#### Real-World Cases

- **Data processing**: Combining and transforming data streams.
- **Combinatorial problems**: Generating all possibilities.
- **Memory-efficient processing**: Lazy evaluation of large datasets.

#### References

- itertools — Functions creating iterators for efficient looping - https://docs.python.org/3/library/itertools.html

---

### 7.2 `functools`

#### Definitions

**Core Definition**: The `functools` module is for higher-order functions: functions that act on or return other functions.

**Technical Definition**: `functools` provides tools for working with callable objects, including caching (`cache`, `lru_cache`), partial application (`partial`), method transformation (`cached_property`), comparison helpers (`total_ordering`), and function metadata preservation (`wraps`).

**Beginner-Friendly Explanation**: The `functools` module gives you tools for working with functions. Use `@functools.cache` to cache function results, or `functools.partial(f, 1)` to create a function with some arguments pre-filled.

#### Purposes

- To cache function results for performance (`@cache`, `@lru_cache`).
- To create partial functions (`partial`).
- To preserve function metadata when wrapping (`wraps`).
- To create cached properties (`cached_property`).
- To reduce boilerplate for comparison methods (`total_ordering`).

#### Syntax Rules and Structure

```python
import functools

@functools.cache                    # Unbounded cache
@functools.lru_cache(maxsize=128)   # LRU cache
functools.partial(func, *args)      # Partial function
@functools.cached_property          # Cached property
@functools.total_ordering           # Generate comparison methods
```

#### Code Examples and Expected Outputs

```python
import functools

# step1: Cache
@functools.cache
def fibonacci(n):
    return n if n < 2 else fibonacci(n - 1) + fibonacci(n - 2)

print(fibonacci(10))                # 55

# step2: Partial
def power(base, exponent):
    return base ** exponent

square = functools.partial(power, exponent=2)
print(square(5))                    # 25
```

**Expected Output**:
```
55
25
```

**Why**: `@cache` stores results of previous calls; `partial` pre-fills arguments to create a new function.

#### Real-World Cases

- **Performance optimization**: Caching expensive computations.
- **API simplification**: Creating specialized functions from general ones.
- **Metaprogramming**: Wrapping functions while preserving metadata.

#### References

- functools — Higher-order functions and operations on callable objects - https://docs.python.org/3/library/functools.html

---

### 7.3 `collections`

#### Definitions

**Core Definition**: The `collections` module implements specialized container datatypes providing alternatives to Python's general purpose built-in containers.

**Technical Definition**: `collections` provides `namedtuple` (tuple subclass with named fields), `deque` (double-ended queue), `ChainMap` (single view of multiple mappings), `Counter` (dict subclass for counting), `OrderedDict` (dict subclass remembering order), `defaultdict` (dict subclass with factory function), and `UserDict`/`UserList`/`UserString` (wrappers for easier subclassing).

**Beginner-Friendly Explanation**: The `collections` module gives you specialized containers. Use `Counter` to count items, `deque` for fast appends/pops on both ends, and `defaultdict` for dictionaries with default values.

#### Purposes

- To count hashable objects (`Counter`).
- To create tuples with named fields (`namedtuple`).
- To implement efficient queues and stacks (`deque`).
- To provide default values for missing keys (`defaultdict`).
- To remember insertion order (`OrderedDict`).

#### Syntax Rules and Structure

```python
from collections import Counter, deque, defaultdict, namedtuple, OrderedDict

Counter(iterable)                   # Count elements
deque(iterable)                     # Double-ended queue
defaultdict(factory)                # Dict with default values
namedtuple("Name", ["field1", "field2"])  # Named tuple
OrderedDict()                       # Ordered dictionary
```

#### Code Examples and Expected Outputs

```python
from collections import Counter, deque, defaultdict, namedtuple

# step1: Counter
c = Counter("abracadabra")
print(c.most_common(2))             # [('a', 5), ('b', 2)]

# step2: deque
d = deque([1, 2, 3])
d.appendleft(0)
d.append(4)
print(list(d))                      # [0, 1, 2, 3, 4]

# step3: defaultdict
dd = defaultdict(int)
dd["a"] += 1
print(dd["a"])                      # 1

# step4: namedtuple
Point = namedtuple("Point", ["x", "y"])
p = Point(3, 4)
print(p.x, p.y)                     # 3 4
```

**Expected Output**:
```
[('a', 5), ('b', 2)]
[0, 1, 2, 3, 4]
1
3 4
```

**Why**: `Counter` counts elements; `deque` supports O(1) appends/pops on both ends; `defaultdict` provides default values; `namedtuple` creates tuple subclasses with named fields.

#### Real-World Cases

- **Data analysis**: Counting occurrences.
- **Algorithms**: Implementing queues and stacks.
- **Configuration**: Grouping settings.

#### References

- collections — Container datatypes - https://docs.python.org/3/library/collections.html

---

## 8. Structural Utilities

### 8.1 `shutil`

#### Definitions

**Core Definition**: The `shutil` module offers a number of high-level operations on files and collections of files.

**Technical Definition**: `shutil` provides functions for copying (`copy`, `copy2`, `copytree`), moving (`move`), removing (`rmtree`), and archiving (`make_archive`, `unpack_archive`) files and directories. It also provides disk usage information (`disk_usage`).

**Beginner-Friendly Explanation**: The `shutil` module lets you copy, move, and delete files and directories. Use `shutil.copy("source.txt", "dest.txt")` to copy a file.

#### Purposes

- To copy files and directories.
- To move files and directories.
- To remove directory trees.
- To create and extract archives.
- To check disk usage.

#### Syntax Rules and Structure

```python
import shutil

shutil.copy(src, dst)               # Copy file
shutil.copy2(src, dst)              # Copy with metadata
shutil.copytree(src, dst)           # Copy directory tree
shutil.move(src, dst)               # Move file/directory
shutil.rmtree(path)                 # Remove directory tree
shutil.disk_usage(path)             # Disk usage
shutil.make_archive(base, format, root_dir)  # Create archive
```

#### Code Examples and Expected Outputs

```python
import shutil
import os

# step1: Copy file
shutil.copy("source.txt", "destination.txt")
print(os.path.exists("destination.txt"))  # True

# step2: Disk usage
usage = shutil.disk_usage("/")
print(f"Total: {usage.total // (2**30)} GB")
# Total: 500 GB

# step3: Remove directory tree
# shutil.rmtree("/tmp/mydir")       # Removes directory and contents
```

**Expected Output**:
```
True
Total: 500 GB
```

**Why**: `shutil` provides high-level file operations; `disk_usage` returns a named tuple with total, used, and free space.

#### Real-World Cases

- **Backup scripts**: Copying files and directories.
- **Deployment**: Moving application files.
- **Cleanup**: Removing temporary directories.

#### References

- shutil — High-level file operations - https://docs.python.org/3/library/shutil.html

---

### 8.2 `copy`

#### Definitions

**Core Definition**: The `copy` module provides generic shallow and deep copy operations.

**Technical Definition**: `copy` provides `copy()` for shallow copies (new compound object with references to original items) and `deepcopy()` for deep copies (new compound object with recursive copies of items). The module avoids recursive loops using a memo dictionary and allows classes to customize copying via `__copy__` and `__deepcopy__`.

**Beginner-Friendly Explanation**: The `copy` module lets you duplicate objects. A shallow copy creates a new object but shares references to nested objects; a deep copy creates completely independent copies.

#### Purposes

- To create shallow copies of objects.
- To create deep copies of objects.
- To avoid unintended sharing of mutable objects.
- To implement custom copy behaviour in classes.

#### Syntax Rules and Structure

```python
import copy

copy.copy(obj)                      # Shallow copy
copy.deepcopy(obj)                  # Deep copy
```

#### Code Examples and Expected Outputs

```python
import copy

# step1: Shallow copy
original = [[1, 2], [3, 4]]
shallow = copy.copy(original)
shallow[0][0] = 99
print(original)                     # [[99, 2], [3, 4]]
print(shallow)                      # [[99, 2], [3, 4]]

# step2: Deep copy
deep = copy.deepcopy(original)
deep[0][0] = 100
print(original)                     # [[99, 2], [3, 4]]
print(deep)                         # [[100, 2], [3, 4]]
```

**Expected Output**:
```
[[99, 2], [3, 4]]
[[99, 2], [3, 4]]
[[99, 2], [3, 4]]
[[100, 2], [3, 4]]
```

**Why**: Shallow copy shares nested objects; deep copy recursively copies all nested objects.

#### Real-World Cases

- **Data processing**: Avoiding unintended mutations.
- **Configuration**: Creating independent copies of settings.
- **Game development**: Cloning game objects.

#### References

- copy — Shallow and deep copy operations - https://docs.python.org/3/library/copy.html

---

## 9. Modern Additions

### 9.1 `zoneinfo`

#### Definitions

**Core Definition**: The `zoneinfo` module provides a concrete time zone implementation to support the IANA time zone database.

**Technical Definition**: `zoneinfo` was added in Python 3.9 via PEP 615. It provides `ZoneInfo`, a concrete implementation of the `datetime.tzinfo` abstract base class, using the system's time zone data if available, falling back to the `tzdata` PyPI package. It handles daylight saving time transitions and ambiguous times via the `fold` attribute.

**Beginner-Friendly Explanation**: The `zoneinfo` module lets you work with real time zones. Use `ZoneInfo("America/New_York")` to get the time zone object for New York.

#### Purposes

- To attach IANA time zones to datetime objects.
- To handle daylight saving time transitions correctly.
- To convert between time zones.
- To work with international date/time data.

#### Syntax Rules and Structure

```python
from zoneinfo import ZoneInfo
from datetime import datetime

tz = ZoneInfo("America/New_York")
dt = datetime(2026, 9, 28, 14, 30, tzinfo=tz)
dt.astimezone(ZoneInfo("Europe/London"))
```

#### Code Examples and Expected Outputs

```python
from zoneinfo import ZoneInfo
from datetime import datetime

# step1: Create aware datetime
dt = datetime(2026, 9, 28, 14, 30, tzinfo=ZoneInfo("America/New_York"))
print(dt)                           # 2026-09-28 14:30:00-04:00
print(dt.tzname())                  # EDT

# step2: Convert to another time zone
london = dt.astimezone(ZoneInfo("Europe/London"))
print(london)                       # 2026-09-28 19:30:00+01:00
```

**Expected Output**:
```
2026-09-28 14:30:00-04:00
EDT
2026-09-28 19:30:00+01:00
```

**Why**: `ZoneInfo` provides accurate time zone data; `astimezone` converts between time zones.

#### Real-World Cases

- **International applications**: Scheduling across time zones.
- **Log analysis**: Converting timestamps to local time.
- **Calendar systems**: Handling DST transitions.

#### References

- zoneinfo — IANA time zone support - https://docs.python.org/3/library/zoneinfo.html

---

### 9.2 `tomllib`

#### Definitions

**Core Definition**: The `tomllib` module provides an interface for parsing TOML files.

**Technical Definition**: `tomllib` was added in Python 3.11. It parses TOML (Tom's Obvious Minimal Language) documents into Python dictionaries. The module supports all TOML data types, including tables, arrays, and nested structures. It does not support writing TOML; the `tomli-w` and `tomlkit` packages provide write capability.

**Beginner-Friendly Explanation**: The `tomllib` module lets you read TOML configuration files. Use `tomllib.load(f)` to parse a TOML file into a Python dictionary.

#### Purposes

- To parse TOML configuration files.
- To read `pyproject.toml` files.
- To load TOML data into Python dictionaries.
- To validate TOML syntax.

#### Syntax Rules and Structure

```python
import tomllib

with open("pyproject.toml", "rb") as f:
    data = tomllib.load(f)

data = tomllib.loads(toml_string)
```

#### Code Examples and Expected Outputs

```python
import tomllib

# step1: Parse TOML string
toml_str = """
[project]
name = "my-project"
version = "1.0.0"
dependencies = ["requests", "click"]
"""

data = tomllib.loads(toml_str)
print(data["project"]["name"])      # my-project
print(data["project"]["dependencies"])  # ['requests', 'click']

# step2: Parse TOML file
# with open("pyproject.toml", "rb") as f:
#     config = tomllib.load(f)
```

**Expected Output**:
```
my-project
['requests', 'click']
```

**Why**: `tomllib.loads` parses TOML strings; `tomllib.load` parses binary file objects; both return dictionaries.

#### Real-World Cases

- **Python packaging**: Reading `pyproject.toml`.
- **Configuration management**: Loading application settings.
- **Build tools**: Parsing build configuration.

#### References

- tomllib — Parse TOML files - https://docs.python.org/3/library/tomllib.html

---

## References

- Python Standard Library — Official Documentation - https://docs.python.org/3/library/
- math — Mathematical functions - https://docs.python.org/3/library/math.html
- statistics — Mathematical statistics functions - https://docs.python.org/3/library/statistics.html
- random — Generate pseudo-random numbers - https://docs.python.org/3/library/random.html
- decimal — Decimal fixed-point and floating-point arithmetic - https://docs.python.org/3/library/decimal.html
- datetime — Basic date and time types - https://docs.python.org/3/library/datetime.html
- os — Miscellaneous operating system interfaces - https://docs.python.org/3/library/os.html
- sys — System-specific parameters and functions - https://docs.python.org/3/library/sys.html
- pathlib — Object-oriented filesystem paths - https://docs.python.org/3/library/pathlib.html
- json — JSON encoder and decoder - https://docs.python.org/3/library/json.html
- csv — CSV File Reading and Writing - https://docs.python.org/3/library/csv.html
- re — Regular expression operations - https://docs.python.org/3/library/re.html
- logging — Logging facility for Python - https://docs.python.org/3/library/logging.html
- subprocess — Subprocess management - https://docs.python.org/3/library/subprocess.html
- argparse — Parser for command-line options, arguments and subcommands - https://docs.python.org/3/library/argparse.html
- sqlite3 — DB-API 2.0 interface for SQLite databases - https://docs.python.org/3/library/sqlite3.html
- itertools — Functions creating iterators for efficient looping - https://docs.python.org/3/library/itertools.html
- functools — Higher-order functions and operations on callable objects - https://docs.python.org/3/library/functools.html
- collections — Container datatypes - https://docs.python.org/3/library/collections.html
- shutil — High-level file operations - https://docs.python.org/3/library/shutil.html
- copy — Shallow and deep copy operations - https://docs.python.org/3/library/copy.html
- zoneinfo — IANA time zone support - https://docs.python.org/3/library/zoneinfo.html
- tomllib — Parse TOML files - https://docs.python.org/3/library/tomllib.html
- PEP 615 – Support for the IANA Time Zone Database - https://peps.python.org/pep-0615/
- PEP 249 – Database API Specification 2.0 - https://peps.python.org/pep-0249/
- PEP 324 – subprocess - New process module - https://peps.python.org/pep-0324/
- PEP 305 – CSV File API - https://peps.python.org/pep-0305/