# Data Serialization: A Comprehensive Programming Cheat Sheet

## Topic Overview

### Definitions

**Core Definition**
Data serialization is the process of converting in-memory data structures and objects into a format that can be stored on disk or transmitted over a network, with the ability to reconstruct the original data later (deserialization).

**Technical Definition**
Serialization transforms Python objects into a linear byte or text representation that preserves their structure and values. Python's standard library provides multiple serialization modules, each with different trade-offs between human readability, cross-language interoperability, type support, and security. The `json` module handles JSON text format; `csv` manages tabular data; `pickle` performs Python-specific binary serialization; and configuration parsers (`configparser`, `tomllib`, PyYAML) handle structured config files.

**Beginner-Friendly Explanation**
Serialization is like packing your data into a suitcase so you can store it or send it somewhere. Later, you unpack the suitcase to get your data back. Different "suitcases" (formats) have different properties: some are readable by humans (JSON, YAML), some are compact and fast (pickle), and some only work within Python (pickle).

### Key Characteristics

| Format | Human-Readable | Cross-Language | Type Support | Security Risk |
|---|---|---|---|---|
| JSON | Yes | Yes | Primitives only | Low (safe by default) |
| CSV | Yes | Yes | Tabular only | Low |
| Pickle | No (binary) | No (Python-only) | Arbitrary Python objects | **High** (code execution) |
| YAML | Yes | Yes | Rich types | **High** with unsafe loader |
| TOML | Yes | Yes | Rich types | Low |
| INI | Yes | Yes | String values | Low |

### Prerequisites

- Basic Python syntax and data structures (dicts, lists, classes)
- File I/O basics (`open()`, `with` statement)
- Understanding of strings vs. bytes
- Basic exception handling

### Related Programming Areas

- **File I/O**: Serialization writes to and reads from files
- **Network Communication**: APIs often exchange JSON
- **Configuration Management**: TOML, YAML, INI for app settings
- **Data Persistence**: Saving and loading program state
- **Security**: Deserialization vulnerabilities and safe alternatives

### Core Concepts / Features

1. **JSON Serialization** (`json.dumps`, `json.loads`, non-serializable objects)
2. **CSV Parsing and Serialization** (`csv.reader`, `csv.DictWriter`, escaping)
3. **Pickle Serialization** (Python-specific object persistence)
4. **Serialization Security** (arbitrary code execution hazards)
5. **Custom Serialization** (encoder/decoder hooks)
6. **INI, TOML, and YAML Configuration Parsing**

---

## Core Concept 1: JSON Serialization

### Definitions

**Core Definition**
JSON (JavaScript Object Notation) is a lightweight, text-based data interchange format that is human-readable and language-independent.

**Technical Definition**
The `json` module serializes Python primitives (dicts, lists, strings, numbers, booleans, `None`) to JSON strings via `json.dumps()`, and deserializes JSON strings to Python objects via `json.loads()`. For objects not natively serializable, a custom `default` function or `JSONEncoder` subclass can be provided.

**Beginner-Friendly Explanation**
JSON is a simple text format that looks like Python dictionaries and lists. It's the standard way to send data between programs, especially over the web. Python's `json` module converts your data to JSON text and back.

### Purposes

- **To exchange data between programs** written in different languages
- **To store structured data** in a human-readable text file
- **To communicate with web APIs** that expect or return JSON
- **To persist simple Python data structures** without security risks

### Syntax Rules and Structure

```python
import json

# Serialize to string
json_string = json.dumps(obj, indent=2, default=custom_encoder)

# Deserialize from string
python_obj = json.loads(json_string, object_hook=custom_decoder)

# Serialize to file
with open("data.json", "w") as f:
    json.dump(obj, f, indent=2)

# Deserialize from file
with open("data.json", "r") as f:
    obj = json.load(f)
```

**Syntax Rules**

| Parameter | Purpose |
|---|---|
| `indent` | Pretty-print with indentation  |
| `default` | Function for non-serializable objects  |
| `object_hook` | Custom decoder for dicts  |
| `sort_keys` | Sort dictionary keys in output |

### Multiple Annotated Complete Code Examples and Expected Outputs

**Example 1: Basic JSON Serialization**

```python
import json

data = {"name": "Alice", "age": 30, "scores": [85, 92, 78]}

# Serialize to JSON string
json_str = json.dumps(data, indent=2)
print(json_str)
```

**Expected Output:**
```
{
  "name": "Alice",
  "age": 30,
  "scores": [
    85,
    92,
    78
  ]
}
```

**Breakdown:** `json.dumps` converts the dictionary to a formatted JSON string. `indent=2` makes it human-readable .

**Example 2: Handling Non-Serializable Objects**

```python
import json

class Point:
    def __init__(self, x, y):
        self.x = x
        self.y = y

def encode_point(obj):
    if isinstance(obj, Point):
        return {"__type__": "Point", "x": obj.x, "y": obj.y}
    raise TypeError(f"Object of type {type(obj)} is not JSON serializable")

p = Point(3, 4)
json_str = json.dumps(p, default=encode_point)
print(json_str)
```

**Expected Output:**
```
{"__type__": "Point", "x": 3, "y": 4}
```

**Breakdown:** The `default` function is called for objects that aren't natively serializable. It returns a serializable representation .

**Example 3: Custom Decoder with `object_hook`**

```python
import json

def decode_point(dct):
    if dct.get("__type__") == "Point":
        return Point(dct["x"], dct["y"])
    return dct

json_str = '{"__type__": "Point", "x": 3, "y": 4}'
p = json.loads(json_str, object_hook=decode_point)
print(f"x={p.x}, y={p.y}")
```

**Expected Output:**
```
x=3, y=4
```

**Breakdown:** `object_hook` is called for every JSON object (dict) and can reconstruct custom types .

### Real-World Cases with Explanation

**Case 1: API Communication**

```python
import requests
import json

response = requests.get("https://api.example.com/users")
users = response.json()  # Uses json.loads internally

for user in users:
    print(user["name"])
```

**Why it matters:** JSON is the lingua franca of web APIs. Python's `json` module makes it trivial to send and receive structured data.

---

## Core Concept 2: CSV Parsing and Serialization

### Definitions

**Core Definition**
CSV (Comma-Separated Values) is a tabular text format where fields are separated by a delimiter (typically a comma) and records by newlines.

**Technical Definition**
The `csv` module provides `reader` and `writer` objects for reading and writing CSV files, plus `DictReader` and `DictWriter` for dictionary-based access. Dialects control formatting parameters like delimiter, quote character, and escaping rules.

**Beginner-Friendly Explanation**
CSV is like a simple spreadsheet saved as text. Each line is a row, and commas separate the columns. Python's `csv` module handles the tricky parts—like fields that contain commas or quotes.

### Purposes

- **To exchange tabular data** with spreadsheets and databases
- **To import/export large datasets** in a simple format
- **To process log files** and structured text data
- **To generate reports** readable by Excel and other tools

### Syntax Rules and Structure

```python
import csv

# Reading
with open("data.csv", "r") as f:
    reader = csv.reader(f)
    for row in reader:
        print(row)

# Writing with DictWriter
with open("output.csv", "w", newline="") as f:
    writer = csv.DictWriter(f, fieldnames=["name", "age"])
    writer.writeheader()
    writer.writerow({"name": "Alice", "age": 30})
```

**Syntax Rules**

| Parameter | Purpose |
|---|---|
| `delimiter` | Field separator (default `,`)  |
| `quotechar` | Character for quoting fields (default `"`) |
| `escapechar` | Character for escaping special chars  |
| `doublequote` | Double quotes inside quoted fields  |
| `newline=""` | Required for correct line endings on Windows |

### Multiple Annotated Complete Code Examples and Expected Outputs

**Example 1: Reading CSV**

```python
import csv

with open("data.csv", "r") as f:
    reader = csv.reader(f)
    for row in reader:
        print(row)
```

**Expected Output (example):**
```
['Name', 'Age', 'City']
['Alice', '30', 'NYC']
['Bob', '25', 'LA']
```

**Breakdown:** `csv.reader` yields each row as a list of strings.

**Example 2: Writing with DictWriter**

```python
import csv

with open("output.csv", "w", newline="") as f:
    writer = csv.DictWriter(f, fieldnames=["name", "score"])
    writer.writeheader()
    writer.writerow({"name": "Alice", "score": 95})
    writer.writerow({"name": "Bob", "score": 87})
```

**Expected Output (file content):**
```
name,score
Alice,95
Bob,87
```

**Breakdown:** `DictWriter` maps dictionaries to CSV rows using `fieldnames` .

**Example 3: Handling Special Characters**

```python
import csv

data = [["Name", "Description"], ["Item1", "Contains, comma"]]

with open("escaped.csv", "w", newline="") as f:
    writer = csv.writer(f)
    writer.writerows(data)
```

**Expected Output (file content):**
```
Name,Description
Item1,"Contains, comma"
```

**Breakdown:** The comma inside the field is preserved by quoting the entire field .

### Real-World Cases with Explanation

**Case 1: Data Import from Spreadsheets**

```python
import csv

def load_users(path):
    users = []
    with open(path, "r") as f:
        reader = csv.DictReader(f)
        for row in reader:
            users.append(row)
    return users
```

**Why it matters:** CSV is the most common export format from Excel and Google Sheets. `DictReader` provides column-name access.

---

## Core Concept 3: Pickle Serialization

### Definitions

**Core Definition**
Pickle is Python's built-in binary serialization format for converting arbitrary Python objects into byte streams and back.

**Technical Definition**
The `pickle` module uses protocols (versions 0-5) to serialize objects. It supports most Python types, including custom class instances, by recording their `__dict__` or using `__reduce__()` methods. Functions and classes are pickled by reference (module and name), not by value .

**Beginner-Friendly Explanation**
Pickle is Python's way of saving objects to disk. Unlike JSON, it can save almost anything—including custom objects. But the file is binary (not human-readable) and can only be loaded by Python.

### Purposes

- **To persist complex Python objects** (custom classes, functions, lambdas)
- **To cache computed results** for later reuse
- **To send Python objects** between processes
- **To save machine learning models** (though security concerns exist)

### Syntax Rules and Structure

```python
import pickle

# Serialize to bytes
data = pickle.dumps(obj)

# Deserialize from bytes
obj = pickle.loads(data)

# Serialize to file
with open("data.pkl", "wb") as f:
    pickle.dump(obj, f)

# Deserialize from file
with open("data.pkl", "rb") as f:
    obj = pickle.load(f)
```

**Syntax Rules**

| Rule | Description |
|---|---|
| Binary mode required | Files must be opened with `"wb"` or `"rb"` |
| Protocol version | Higher protocols are more efficient but less compatible  |
| Functions by reference | Only top-level functions/classes can be pickled  |
| Custom classes | `__dict__` is pickled; `__getstate__`/`__setstate__` can customize |

### Multiple Annotated Complete Code Examples and Expected Outputs

**Example 1: Basic Pickle**

```python
import pickle

data = {"name": "Alice", "scores": [85, 92, 78]}

with open("data.pkl", "wb") as f:
    pickle.dump(data, f)

with open("data.pkl", "rb") as f:
    loaded = pickle.load(f)

print(loaded)
```

**Expected Output:**
```
{'name': 'Alice', 'scores': [85, 92, 78]}
```

**Breakdown:** Pickle serializes the dictionary to a binary file and reconstructs it exactly.

**Example 2: Custom Class**

```python
import pickle

class Point:
    def __init__(self, x, y):
        self.x = x
        self.y = y

p = Point(3, 4)

with open("point.pkl", "wb") as f:
    pickle.dump(p, f)

with open("point.pkl", "rb") as f:
    loaded = pickle.load(f)

print(f"x={loaded.x}, y={loaded.y}")
```

**Expected Output:**
```
x=3, y=4
```

**Breakdown:** Pickle records the class reference and instance state, reconstructing the object .

### Real-World Cases with Explanation

**Case 1: Caching Expensive Computations**

```python
import pickle
import os

def get_data():
    if os.path.exists("cache.pkl"):
        with open("cache.pkl", "rb") as f:
            return pickle.load(f)
    data = expensive_computation()
    with open("cache.pkl", "wb") as f:
        pickle.dump(data, f)
    return data
```

**Why it matters:** Pickle is convenient for caching, but only for trusted data.

---

## Core Concept 4: Serialization Security Considerations

### Definitions

**Core Definition**
Serialization security concerns the risks of deserializing data from untrusted sources, particularly the danger of arbitrary code execution.

**Technical Definition**
Python's `pickle` module can execute arbitrary code during deserialization through `__reduce__`, `__setstate__`, or other special methods. PyYAML's `yaml.load()` (without `SafeLoader`) similarly supports tags like `!!python/object/apply` that instantiate arbitrary objects. Both are documented as inherently unsafe for untrusted input.

**Beginner-Friendly Explanation**
Some serialization formats are like letters: opening them can trigger hidden actions. If you unpickle a malicious file, it could run commands on your computer. The rule is simple: never unpickle or unsafe-load YAML from sources you don't fully trust.

### Purposes (of Understanding Security)

- **To avoid remote code execution vulnerabilities**
- **To choose safe formats** for untrusted data
- **To implement safe deserialization** when a format is unavoidable
- **To audit code** for insecure deserialization patterns

### Syntax Rules and Structure

**Unsafe vs. Safe:**

```python
# UNSAFE: Never do this with untrusted data
import pickle
data = pickle.loads(untrusted_bytes)  # Can execute code

import yaml
data = yaml.load(untrusted_string, Loader=yaml.Loader)  # Can execute code

# SAFE alternatives
import json
data = json.loads(untrusted_string)  # Data only, no code execution

import yaml
data = yaml.safe_load(untrusted_string)  # Restricted to basic types
```

**Syntax Rules**

| Rule | Description |
|---|---|
| Pickle is never safe for untrusted input | No configuration makes it safe  |
| `yaml.safe_load()` is required | The default `load()` requires explicit Loader in modern PyYAML  |
| JSON is safe | No code execution mechanism  |

### Multiple Annotated Complete Code Examples and Expected Outputs

**Example 1: The Danger of Pickle**

```python
import pickle
import os

# A malicious pickle payload (conceptual)
class Exploit:
    def __reduce__(self):
        return (os.system, ("echo 'Code executed!'",))

malicious = pickle.dumps(Exploit())
# When loaded, this would execute the command
```

**Breakdown:** Pickle's `__reduce__` can specify a callable and arguments. Loading this payload executes the command .

**Example 2: Safe YAML Loading**

```python
import yaml

# UNSAFE: can execute code
# yaml.load(untrusted, Loader=yaml.Loader)

# SAFE: only basic types
data = yaml.safe_load("""
name: Alice
age: 30
""")
print(data)
```

**Expected Output:**
```
{'name': 'Alice', 'age': 30}
```

**Breakdown:** `safe_load` only constructs basic Python types (dicts, lists, strings, numbers) .

### Real-World Cases with Explanation

**Case 1: Machine Learning Model Loading**

```python
# UNSAFE: torch.load() uses pickle by default
# model = torch.load("untrusted_model.pt")

# SAFE: use safetensors format
# from safetensors.torch import load_file
# weights = load_file("model.safetensors")
```

**Why it matters:** PyTorch model files are pickled and can execute code. The `safetensors` format avoids pickle entirely .

**Case 2: Configuration Files from Users**

```python
import yaml

# If users can upload config files:
def load_config(content):
    return yaml.safe_load(content)  # Always safe
```

**Why it matters:** Any config file from an untrusted source must be parsed with a safe loader .

---

## Core Concept 5: Custom Serialization

### Definitions

**Core Definition**
Custom serialization allows developers to define how specific objects are encoded and decoded when standard serialization doesn't support them.

**Technical Definition**
For JSON, custom encoders subclass `json.JSONEncoder` and override `default()`, or provide a `default` function. Custom decoders use `object_hook` to reconstruct objects from dictionaries. For pickle, `__getstate__`/`__setstate__` or `__reduce__` customize object serialization.

**Beginner-Friendly Explanation**
If your object doesn't fit the standard format, you can teach the serializer how to handle it. You write a small function that says "when you see this type, convert it this way."

### Purposes

- **To serialize custom classes** not supported by default
- **To control the serialized representation** of objects
- **To reconstruct objects** during deserialization
- **To handle special cases** like dates, sets, or complex types

### Syntax Rules and Structure

**JSON Custom Encoder:**

```python
import json

class CustomEncoder(json.JSONEncoder):
    def default(self, obj):
        if isinstance(obj, MyClass):
            return {"__type__": "MyClass", "value": obj.value}
        return super().default(obj)

json.dumps(obj, cls=CustomEncoder)
```

**JSON Custom Decoder:**

```python
def custom_decoder(dct):
    if dct.get("__type__") == "MyClass":
        return MyClass(dct["value"])
    return dct

json.loads(json_str, object_hook=custom_decoder)
```

### Multiple Annotated Complete Code Examples and Expected Outputs

**Example 1: Custom JSON Encoder**

```python
import json
from datetime import datetime

class DateTimeEncoder(json.JSONEncoder):
    def default(self, obj):
        if isinstance(obj, datetime):
            return obj.isoformat()
        return super().default(obj)

data = {"timestamp": datetime(2026, 9, 25, 14, 30)}
json_str = json.dumps(data, cls=DateTimeEncoder)
print(json_str)
```

**Expected Output:**
```
{"timestamp": "2026-09-25T14:30:00"}
```

**Breakdown:** The custom encoder converts `datetime` objects to ISO format strings.

**Example 2: Pickle Custom State**

```python
import pickle

class Point:
    def __init__(self, x, y):
        self.x = x
        self.y = y

    def __getstate__(self):
        return {"x": self.x, "y": self.y}

    def __setstate__(self, state):
        self.x = state["x"]
        self.y = state["y"]

p = Point(3, 4)
with open("point.pkl", "wb") as f:
    pickle.dump(p, f)
```

**Breakdown:** `__getstate__` and `__setstate__` control what data is pickled and how it's restored.

### Real-World Cases with Explanation

**Case 1: API Response Serialization**

```python
import json
from decimal import Decimal

class DecimalEncoder(json.JSONEncoder):
    def default(self, obj):
        if isinstance(obj, Decimal):
            return str(obj)
        return super().default(obj)

# Serialize API response with Decimal values
json.dumps({"price": Decimal("19.99")}, cls=DecimalEncoder)
```

**Why it matters:** JSON doesn't support `Decimal` natively, so custom encoding is needed for financial data.

---

## Core Concept 6: INI, TOML, and YAML Configuration Parsing

### Definitions

**Core Definition**
Configuration file parsers read structured settings from text files, providing a standardized way to configure applications.

**Technical Definition**
- `configparser`: Parses INI files (sections with key-value pairs) 
- `tomllib`: Parses TOML files (Python 3.11+) 
- `PyYAML`: Parses YAML files (third-party) 

### Purposes

- **To separate configuration from code**
- **To allow users to customize application behavior**
- **To manage environment-specific settings** (development, production)
- **To provide a readable format** for non-programmers

### Syntax Rules and Structure

**INI (configparser):**

```python
import configparser

config = configparser.ConfigParser()
config.read("settings.ini")
value = config["section"]["key"]
```

**TOML (tomllib):**

```python
import tomllib

with open("config.toml", "rb") as f:
    config = tomllib.load(f)
```

**YAML (PyYAML):**

```python
import yaml

with open("config.yaml", "r") as f:
    config = yaml.safe_load(f)
```

### Multiple Annotated Complete Code Examples and Expected Outputs

**Example 1: INI Configuration**

```python
import configparser

config = configparser.ConfigParser()
config.read_string("""
[database]
host = localhost
port = 5432
""")

print(config["database"]["host"])
print(config["database"]["port"])
```

**Expected Output:**
```
localhost
5432
```

**Breakdown:** `configparser` reads INI files with sections and key-value pairs .

**Example 2: TOML Configuration**

```python
import tomllib

toml_str = """
[database]
host = "localhost"
port = 5432
"""

config = tomllib.loads(toml_str)
print(config["database"]["host"])
```

**Expected Output:**
```
localhost
```

**Breakdown:** `tomllib` parses TOML into nested dictionaries .

**Example 3: YAML Configuration**

```python
import yaml

yaml_str = """
database:
  host: localhost
  port: 5432
"""

config = yaml.safe_load(yaml_str)
print(config["database"]["host"])
```

**Expected Output:**
```
localhost
```

**Breakdown:** `yaml.safe_load` parses YAML into Python dictionaries .

### Real-World Cases with Explanation

**Case 1: Application Settings**

```python
import tomllib
from pathlib import Path

def load_settings():
    config_path = Path.home() / ".config" / "myapp" / "settings.toml"
    if config_path.exists():
        with open(config_path, "rb") as f:
            return tomllib.load(f)
    return {"debug": False}
```

**Why it matters:** TOML is the modern choice for Python project configuration (used in `pyproject.toml`).

---

## References

- Real Python. *json | Python Standard Library*. https://realpython.com/ref/stdlib/json/ 
- Python Software Foundation. *csv — CSV 文件读写*. https://docs.python.org/zh-cn/3.14/library/csv.html 
- Python Software Foundation. *pickle — Python object serialization*. https://docs.python.org/3/library/pickle.html 
- Python Software Foundation. *pickle — Python object serialization (3.10)*. https://docs.python.org/3.10/library/pickle.html 
- Safeguard. *YAML Code Security: Safe Parsing and Deserialization*. https://safeguard.sh/resources/blog/yaml-code 
- Python Software Foundation. *configparser — Configuration file parser*. https://docs.python.org/3.10/library/configparser.html 
- Python Software Foundation. *tomllib — Parse TOML files*. https://docs.python.org/3.11/library/tomllib.html 
- TechGig. *How to Work with YAML in Python – A Simple Guide for Beginners*. https://content.techgig.com/upskilling-at-techgig/https/yourwebsite-com/python-yaml-guide/amp_articleshow/125928467.cms 
- Python Software Foundation. *json — JSON encoder and decoder*. https://docs.python.org/fr/dev/library/json.html 
- Python Software Foundation. *csv — CSV File Reading and Writing*. https://docs.python.org/3/library/csv.html 
- Python Software Foundation. *pickle — Python 对象序列化*. https://docs.python.org/zh-cn/3.14/library/pickle.html 
- Open Standards. *ISO/IEC JTC1 SC22 WG23 N1306 Python Vulnerabilities*. http://www9.open-std.org/JTC1/SC22/WG23/docs/ISO-IECJTC1-SC22-WG23_N1306-tr24772-4-python-vulnerabilities-before-mtg-20230705.pdf 
- Safeguard. *Insecure Deserialization Prevention in Python: yaml.safe_load vs Pickle*. https://safeguard.sh/resources/blog/insecure-deserialization-prevention-in-python-yamlsafeload-vs-pickle 
- Python Software Foundation. *configparser — 구성 파일 구문 분석기*. https://docs.python.org/ko/3.16/library/configparser.html 
- Real Python. *tomllib | Python Standard Library*. https://realpython.com/ref/stdlib/tomllib/ 
- freeCodeCamp. *How to Work with YAML in Python – A Guide with Examples*. https://www.freecodecamp.org/news/how-to-work-with-yaml-in-python-a-guide-with-examples/ 
- Stack Overflow. *Revision cdd07468-a203-401b-b4b5-61d4f2943464*. https://stackoverflow.com/revisions/cdd07468-a203-401b-b4b5-61d4f2943464/view-source 
- Python Software Foundation. *csv — CSV ファイルの読み書き*. https://docs.python.org/ja/3.16/library/csv.html 
- Python Software Foundation. *pickle — Python オブジェクトの直列化*. https://docs.python.org/ja/3/library/pickle.html 
- VulDB. *CVE-2026-65179 in NeMo*. https://vuldb.com/cve/CVE-2026-65179 