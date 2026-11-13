# Python JSON: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**

JSON (JavaScript Object Notation) is a lightweight, text-based data interchange format standardized by RFC 7159 and ECMA-404. Python’s built-in `json` module provides the tools for converting Python objects to JSON text (serialization) and JSON text to Python objects (deserialization).

**Technical Definition**

JSON is a language-independent data format built on two structural constructs—objects (unordered collections of key–value pairs) and arrays (ordered sequences)—and four primitive types: strings, numbers, booleans, and `null`. The `json` module exposes an API familiar to users of `marshal` and `pickle`, implementing a full encoder–decoder pipeline through four core functions: `dumps()`, `dump()`, `loads()`, and `load()`. The module maps JSON objects to Python `dict`, arrays to `list`, strings to `str`, numbers to `int` or `float`, `true`/`false` to `True`/`False`, and `null` to `None`. Serialization can be customized through the `default` callable, the `cls` subclass of `JSONEncoder`, or the parameters `indent`, `sort_keys`, `ensure_ascii`, and `separators`; deserialization can be customized through `object_hook`, `object_pairs_hook`, `parse_float`, and `parse_int`.

**Beginner-Friendly Explanation**

JSON is a way to write data as text so that different programs—whether written in Python, JavaScript, or any other language—can read and understand it. Think of it as a universal recipe card: the ingredients (data) and measurements (structure) are written in a format everyone can follow. Python’s `json` module is your kitchen assistant: it takes Python data (like dictionaries and lists) and turns it into a JSON recipe card, or reads a JSON recipe card and turns it back into Python data.

### Key Characteristics

- **Text-based and human-readable**: JSON is plain text, making it easy to inspect and debug.
- **Language-agnostic**: JSON is supported by virtually every modern programming language.
- **Strict syntax**: JSON requires double quotes for strings, disallows trailing commas, and does not support comments.
- **Four core functions**: `dumps()`, `dump()`, `loads()`, and `load()` cover string-based and file-based operations.
- **Customizable encoding and decoding**: Encoders and decoders can be extended for non-standard types such as `datetime`, `Decimal`, and custom classes.
- **Security-aware**: The `json` module does not execute code during deserialization, unlike `pickle`, but malicious input can still cause resource exhaustion.

### Prerequisites

- Basic Python syntax: variables, dictionaries, lists, and functions.
- Familiarity with file I/O and the `with` statement for file-based operations.
- Understanding of exceptions and `try`/`except` for error handling.
- Awareness of the distinction between strings (`str`) and bytes (`bytes`).

### Related Programming Areas

- **Web APIs**: JSON is the dominant format for REST and GraphQL responses.
- **Configuration management**: JSON files store application settings.
- **Data pipelines**: JSON Lines (`.jsonl`) is used for streaming and logging.
- **NoSQL databases**: Document databases such as MongoDB use JSON-like documents.
- **Serialization alternatives**: YAML, XML, MessagePack, and `pickle` serve overlapping roles.

---

## Core Concepts / Features

---

### 1. JSON (JavaScript Object Notation)

#### Definitions

**Core Definition**

JSON is a text-based, language-independent data interchange format that represents structured data as objects (key–value mappings) and arrays (ordered sequences), with support for strings, numbers, booleans, and `null`.

**Technical Definition**

JSON is specified by RFC 7159 (which obsoletes RFC 4627) and ECMA-404. A JSON text may contain a single value, which can be an object, array, string, number, boolean, or `null`. Objects are enclosed in curly braces and contain string keys mapped to JSON values; arrays are enclosed in square brackets and contain comma-separated JSON values. Strings must be enclosed in double quotes and may contain Unicode characters; numbers follow a subset of JavaScript’s numeric syntax and do not distinguish between integers and floating-point values. The format does not permit trailing commas, comments, or single-quoted strings.

**Beginner-Friendly Explanation**

JSON looks a lot like Python dictionaries and lists, but with a few strict rules. Keys and strings must use double quotes. You cannot add comments. You cannot leave a comma after the last item. Once you learn these rules, JSON becomes an easy way to write data that any program can read.

#### Purposes

- To exchange structured data between programs written in different languages.
- To store configuration and settings in a human-readable text format.
- To transmit API request and response payloads over HTTP.
- To persist data in a format that is both machine-parsable and human-inspectable.
- To serve as a common ground between Python’s native data types and other languages’ data models.

#### Syntax Rules and Structure

**Complete General Syntax**

```
{
    "key1": "value1",
    "key2": 123,
    "key3": true,
    "key4": null,
    "key5": [1, 2, 3],
    "key6": {"nested": "object"}
}
```

**Component Breakdown**

| JSON Construct | Description | Python Equivalent |
|----------------|-------------|-------------------|
| Object | `{ "key": value }` | `dict` |
| Array | `[ value1, value2 ]` | `list` |
| String | `"text"` (double quotes) | `str` |
| Number | `42`, `3.14` | `int`, `float` |
| Boolean | `true`, `false` | `True`, `False` |
| Null | `null` | `None` |

**Syntax Rules**

- Strings must be enclosed in double quotes; single quotes are invalid.
- Objects consist of comma-separated key–value pairs; keys must be strings.
- Arrays consist of comma-separated values.
- Trailing commas are not allowed.
- Comments are not supported.
- The top-level value may be any JSON value (since RFC 7159); earlier RFC 4627 required an object or array.
- Numbers may be integers or floating-point but do not distinguish between the two.

**Constraints and Limitations**

- JSON cannot represent Python-specific types such as `datetime`, `Decimal`, `set`, or custom objects without custom encoding.
- JSON keys must be strings; Python allows non-string dictionary keys, which must be converted or skipped.
- `NaN`, `Infinity`, and `-Infinity` are not part of the JSON standard, though Python’s `json` module allows them by default (`allow_nan=True`).
- JSON does not support circular references; encoding them raises `ValueError`.

#### Annotated Code Examples

**Example 1: A Simple JSON Document**

```python
import json

json_text = '''
{
    "name": "Alice",
    "age": 30,
    "is_active": true,
    "skills": ["Python", "SQL"],
    "address": null
}
'''

data = json.loads(json_text)
print(data)
```

**Expected Output:**

```
{'name': 'Alice', 'age': 30, 'is_active': True, 'skills': ['Python', 'SQL'], 'address': None}
```

**Step-by-Step Breakdown:**

1. The triple-quoted string contains a valid JSON object.
2. `json.loads()` parses the JSON text into a Python dictionary.
3. Boolean `true` becomes `True`, and `null` becomes `None`.

**Why the Output is Produced:**

`json.loads()` performs the standard JSON-to-Python type mapping, converting JSON constructs into their Python equivalents.

#### Real-World Cases

- **REST API payloads**: Request and response bodies are almost universally JSON.
- **Configuration files**: `package.json`, `.eslintrc.json`, and many tool configs use JSON.
- **Data export**: Analytics platforms export data as JSON for downstream processing.
- **Inter-service communication**: Microservices exchange JSON messages via message queues.

---

### 2. `json` Module

#### Definitions

**Core Definition**

The `json` module is Python’s standard-library implementation of JSON encoding and decoding, providing four primary functions—`dumps()`, `dump()`, `loads()`, and `load()`—along with `JSONEncoder`, `JSONDecoder`, and `JSONDecodeError`.

**Technical Definition**

The `json` module exposes an API familiar to users of `marshal` and `pickle`, implementing JSON serialization and deserialization with support for custom encoders and decoders. The module defines `JSONEncoder` (with a `default()` method for unsupported types), `JSONDecoder` (with `object_hook`, `object_pairs_hook`, `parse_float`, and `parse_int` customization), and `JSONDecodeError` for parse failures. Module-level convenience functions `dumps()`, `dump()`, `loads()`, and `load()` instantiate the encoder or decoder with default or supplied parameters. The module also provides `json.tool` for command-line validation and pretty-printing.

**Beginner-Friendly Explanation**

The `json` module is Python’s built-in JSON toolkit. It has two pairs of functions: `dumps()`/`dump()` turn Python data into JSON, and `loads()`/`load()` turn JSON back into Python data. The functions ending in “s” work with strings; the ones without “s” work with files. The module also lets you customize how special types like dates are handled.

#### Purposes

- To provide a standard, well-tested implementation of JSON encoding and decoding.
- To convert between Python objects and JSON text for storage and transmission.
- To offer file-based convenience functions (`dump()` and `load()`) for reading and writing JSON files.
- To support customization through encoder and decoder subclasses and hook functions.
- To raise clear exceptions (`JSONDecodeError`) for invalid JSON input.

#### Syntax Rules and Structure

**Complete General Syntax**

```
import json

json.dumps(obj, *, skipkeys=False, ensure_ascii=True, check_circular=True,
           allow_nan=True, cls=None, indent=None, separators=None,
           default=None, sort_keys=False, **kw)

json.dump(obj, fp, *, ...)

json.loads(s, *, cls=None, object_hook=None, parse_float=None,
           parse_int=None, parse_constant=None, object_pairs_hook=None, **kw)

json.load(fp, *, ...)
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `dumps()` | Serializes a Python object to a JSON string. |
| `dump()` | Serializes a Python object to a file-like object. |
| `loads()` | Deserializes a JSON string to a Python object. |
| `load()` | Deserializes JSON from a file-like object to a Python object. |
| `JSONEncoder` | Encoder class; subclass and override `default()` for custom types. |
| `JSONDecoder` | Decoder class; subclass for custom decoding logic. |
| `JSONDecodeError` | Exception raised when JSON parsing fails. |

**Syntax Rules**

- The module must be imported before use: `import json`.
- `dumps()` returns a string; `dump()` writes to a file object.
- `loads()` accepts a string, bytes, or bytearray; `load()` reads from a file object.
- All keyword arguments are keyword-only (since Python 3.6).
- Custom encoders are specified via the `cls` parameter; custom decoders via `cls` or hook parameters.

**Constraints and Limitations**

- The `json` module cannot serialize arbitrary Python objects without customization.
- `dump()` and `load()` do not close the file object; use the `with` statement.
- The module does not support comments or trailing commas, unlike Python literals.
- Performance may be insufficient for very large datasets; faster third-party libraries like `orjson` exist.

#### Annotated Code Examples

**Example 1: Importing and Using the Module**

```python
import json

data = {"name": "Alice", "age": 30}
json_string = json.dumps(data)
print(json_string)
print(type(json_string))
```

**Expected Output:**

```
{"name": "Alice", "age": 30}
<class 'str'>
```

**Step-by-Step Breakdown:**

1. `import json` makes the module available.
2. `json.dumps(data)` serializes the dictionary.
3. The result is a JSON-formatted string.

**Why the Output is Produced:**

The `dumps()` function converts Python dictionaries to JSON objects, producing a string by default.

#### Real-World Cases

- **Web frameworks**: Flask and Django use `json` to serialize API responses.
- **Data science**: Jupyter notebooks store cell outputs and metadata as JSON.
- **DevOps**: Infrastructure-as-code tools emit JSON plans and reports.
- **Testing**: Test fixtures and expected outputs are often stored as JSON.

---

### 3. `json.loads()` — Deserialize a JSON String

#### Definitions

**Core Definition**

`json.loads()` (load from string) parses a JSON-formatted string and returns the corresponding Python object.

**Technical Definition**

`json.loads(s, *, cls=None, object_hook=None, parse_float=None, parse_int=None, parse_constant=None, object_pairs_hook=None, **kw)` deserializes the JSON document contained in the string (or bytes/bytearray) `s`. It uses the standard JSON-to-Python type mapping: objects become `dict`, arrays become `list`, strings become `str`, numbers become `int` or `float`, `true`/`false` become `True`/`False`, and `null` becomes `None`. Customization is available through `object_hook` (called with every decoded `dict`), `object_pairs_hook` (called with a list of key–value pairs, taking precedence over `object_hook`), `parse_float` and `parse_int` (called with the string representation of each number), and `parse_constant` (called for `-Infinity`, `Infinity`, and `NaN`). If the input is not valid JSON, `JSONDecodeError` is raised.

**Beginner-Friendly Explanation**

`json.loads()` takes a piece of JSON text—like the kind you get from an API response or a configuration file—and turns it into Python data. You give it a string, and it gives you back dictionaries, lists, and basic values. If the string is not valid JSON, Python raises an error, so it’s good practice to wrap the call in a `try`/`except` block.

#### Purposes

- To parse JSON strings received from APIs, network messages, or user input.
- To convert JSON text stored in databases or logs into Python objects.
- To support custom decoding of JSON objects into domain-specific Python types.
- To handle numeric types with custom parsers (e.g., `Decimal` for precise decimals).
- To provide clear error reporting when JSON is malformed.

#### Syntax Rules and Structure

**Complete General Syntax**

```
json.loads(s, *, cls=None, object_hook=None, parse_float=None,
           parse_int=None, parse_constant=None, object_pairs_hook=None, **kw)
```

**Component Breakdown**

| Parameter | Description |
|-----------|-------------|
| `s` | The JSON string (or bytes/bytearray) to parse. |
| `cls` | Custom `JSONDecoder` subclass. |
| `object_hook` | Callable invoked with every decoded `dict`. |
| `object_pairs_hook` | Callable invoked with a list of key–value pairs; takes precedence. |
| `parse_float` | Callable invoked with the string form of each float. |
| `parse_int` | Callable invoked with the string form of each int. |
| `parse_constant` | Callable invoked for `-Infinity`, `Infinity`, and `NaN`. |

**Syntax Rules**

- The input must be a `str`, `bytes`, or `bytearray`.
- Invalid JSON raises `JSONDecodeError`.
- `object_hook` and `object_pairs_hook` are mutually exclusive in practice; the latter takes precedence.
- `parse_float` and `parse_int` allow custom numeric types.
- The top-level value may be any JSON value (object, array, string, number, boolean, `null`).

**Constraints and Limitations**

- `loads()` does not validate against a schema; it only checks JSON syntax.
- Very deeply nested JSON can cause `RecursionError`.
- Malicious JSON with extremely large arrays or deeply nested structures can consume significant CPU and memory; limit input size when parsing untrusted data.
- JSON numbers are decoded as `int` or `float` by default; precision-sensitive data requires `parse_float=Decimal`.

#### Annotated Code Examples

**Example 1: Parsing a Simple JSON String**

```python
import json

json_string = '{"name": "Alice", "age": 30, "skills": ["Python", "SQL"]}'
data = json.loads(json_string)

print(data["name"])
print(data["skills"][0])
print(type(data))
```

**Expected Output:**

```
Alice
Python
<class 'dict'>
```

**Step-by-Step Breakdown:**

1. The JSON string contains a JSON object.
2. `json.loads()` converts it to a Python dictionary.
3. Keys and values are accessed as usual.

**Why the Output is Produced:**

`loads()` maps the JSON object to a `dict`, making key access natural.

---

**Example 2: Handling Parsing Errors**

```python
import json

bad_json = "{'name': 'Alice'}"  # Single quotes are invalid JSON

try:
    data = json.loads(bad_json)
except json.JSONDecodeError as e:
    print(f"Invalid JSON: {e}")
```

**Expected Output:**

```
Invalid JSON: Expecting property name enclosed in double quotes: line 1 column 2 (char 1)
```

**Step-by-Step Breakdown:**

1. The string uses single quotes, which are not valid JSON.
2. `json.loads()` raises `JSONDecodeError`.
3. The exception is caught and a descriptive message is printed.

**Why the Output is Produced:**

JSON requires double-quoted keys and strings; the decoder reports the exact location of the syntax error.

---

**Example 3: Custom Decoding with `object_hook`**

```python
import json

def as_complex(dct):
    if "__complex__" in dct:
        return complex(dct["real"], dct["imag"])
    return dct

json_string = '{"__complex__": true, "real": 1, "imag": 2}'
result = json.loads(json_string, object_hook=as_complex)
print(result)
```

**Expected Output:**

```
(1+2j)
```

**Step-by-Step Breakdown:**

1. The JSON object contains a special marker `__complex__`.
2. `object_hook` is called with the decoded dictionary.
3. The hook recognizes the marker and returns a `complex` object instead of a plain `dict`.

**Why the Output is Produced:**

`object_hook` allows post-processing of every decoded dictionary, enabling reconstruction of domain objects.

#### Real-World Cases

- **API client libraries**: Parsing JSON responses into Python dictionaries for further processing.
- **Log analysis**: Extracting structured fields from JSON log lines.
- **Configuration loading**: Reading JSON configuration files into application settings.
- **Data validation**: Parsing JSON and then validating the resulting structure against a schema.

---

### 4. `json.dumps()` — Serialize to a JSON String

#### Definitions

**Core Definition**

`json.dumps()` (dump to string) serializes a Python object into a JSON-formatted string.

**Technical Definition**

`json.dumps(obj, *, skipkeys=False, ensure_ascii=True, check_circular=True, allow_nan=True, cls=None, indent=None, separators=None, default=None, sort_keys=False, **kw)` converts the Python object `obj` to JSON text. The standard Python-to-JSON type mapping is used: `dict` becomes an object, `list` and `tuple` become arrays, `str` becomes a string, `int` and `float` become numbers, `True`/`False` become `true`/`false`, and `None` becomes `null`. The `default` callable is invoked for objects that are not natively serializable; it must return a serializable object or raise `TypeError`. `indent` controls pretty-printing, `sort_keys` orders dictionary keys, `ensure_ascii` controls non-ASCII escaping, and `separators` controls item and key separators. `skipkeys`, `check_circular`, and `allow_nan` govern edge-case behavior.

**Beginner-Friendly Explanation**

`json.dumps()` takes Python data—dictionaries, lists, strings, numbers—and turns it into a JSON string. You can make the output pretty by setting `indent`, sort the keys with `sort_keys`, and handle non-ASCII characters with `ensure_ascii`. If you have a custom object that JSON doesn’t understand, you can provide a `default` function to tell Python how to convert it.

#### Purposes

- To convert Python data structures into JSON text for transmission or storage.
- To produce compact or pretty-printed JSON depending on the use case.
- To control key ordering and character encoding for interoperability.
- To support custom serialization of non-standard types via the `default` parameter or `JSONEncoder` subclass.
- To handle edge cases such as non-string keys and circular references.

#### Syntax Rules and Structure

**Complete General Syntax**

```
json.dumps(obj, *, skipkeys=False, ensure_ascii=True, check_circular=True,
           allow_nan=True, cls=None, indent=None, separators=None,
           default=None, sort_keys=False, **kw)
```

**Component Breakdown**

| Parameter | Description |
|-----------|-------------|
| `obj` | The Python object to serialize. |
| `skipkeys` | If `True`, skip dictionary keys that are not basic types instead of raising `TypeError`. |
| `ensure_ascii` | If `True` (default), escape non-ASCII characters; if `False`, emit them as-is. |
| `check_circular` | If `True` (default), detect circular references and raise `ValueError`. |
| `allow_nan` | If `True` (default), serialize `NaN`, `Infinity`, and `-Infinity`; if `False`, raise `ValueError`. |
| `cls` | Custom `JSONEncoder` subclass. |
| `indent` | If a non-negative integer, pretty-print with that indentation level. |
| `separators` | A `(item_separator, key_separator)` tuple; defaults to `(', ', ': ')` without `indent`. |
| `default` | A callable for objects not otherwise serializable. |
| `sort_keys` | If `True`, sort dictionary keys in the output. |

**Syntax Rules**

- `dumps()` returns a `str`.
- `indent=None` produces compact output; `indent=0` inserts newlines but no spaces.
- `ensure_ascii=False` is useful for non-ASCII text but requires the consumer to handle Unicode.
- `default` must return a serializable object or raise `TypeError`.
- `sort_keys=True` is useful for deterministic output and testing.

**Constraints and Limitations**

- `dumps()` cannot serialize arbitrary Python objects without `default` or a custom encoder.
- Circular references raise `ValueError` unless `check_circular=False` (which can cause infinite recursion).
- The `separators` tuple must contain two strings.
- `ensure_ascii=True` can inflate output size for non-ASCII text.

#### Annotated Code Examples

**Example 1: Basic Serialization**

```python
import json

data = {"name": "Alice", "age": 30, "skills": ["Python", "SQL"]}
json_string = json.dumps(data)
print(json_string)
```

**Expected Output:**

```
{"name": "Alice", "age": 30, "skills": ["Python", "SQL"]}
```

**Step-by-Step Breakdown:**

1. The dictionary is passed to `json.dumps()`.
2. Keys and string values are enclosed in double quotes.
3. The result is a compact JSON string.

**Why the Output is Produced:**

`dumps()` applies the standard Python-to-JSON type mapping and default separators.

---

**Example 2: Pretty-Printing with `indent` and `sort_keys`**

```python
import json

data = {"c": 0, "b": 0, "a": 0}
pretty = json.dumps(data, indent=4, sort_keys=True)
print(pretty)
```

**Expected Output:**

```
{
    "a": 0,
    "b": 0,
    "c": 0
}
```

**Step-by-Step Breakdown:**

1. `indent=4` adds four-space indentation and newlines.
2. `sort_keys=True` orders the keys alphabetically.

**Why the Output is Produced:**

`indent` enables pretty-printing, and `sort_keys` ensures deterministic key ordering.

---

**Example 3: Custom Serialization with `default`**

```python
import json
from datetime import datetime

def datetime_serializer(obj):
    if isinstance(obj, datetime):
        return obj.isoformat()
    raise TypeError(f"Type {type(obj)} not serializable")

event = {"title": "Conference", "start_time": datetime(2025, 10, 15, 9, 30)}
json_string = json.dumps(event, default=datetime_serializer)
print(json_string)
```

**Expected Output:**

```
{"title": "Conference", "start_time": "2025-10-15T09:30:00"}
```

**Step-by-Step Breakdown:**

1. `datetime` is not natively serializable.
2. The `default` callable converts it to an ISO 8601 string.
3. The resulting JSON contains the string representation.

**Why the Output is Produced:**

The `default` function is invoked for unsupported types and must return a serializable value.

#### Real-World Cases

- **API request construction**: Building JSON payloads for HTTP POST/PUT requests.
- **Configuration export**: Writing application settings to JSON files.
- **Testing**: Generating JSON fixtures and expected outputs.
- **Logging**: Emitting structured log entries as JSON strings.

---

### 5. `json.load()` — Deserialize from a File

#### Definitions

**Core Definition**

`json.load()` reads JSON data from a file-like object and returns the corresponding Python object.

**Technical Definition**

`json.load(fp, *, cls=None, object_hook=None, parse_float=None, parse_int=None, parse_constant=None, object_pairs_hook=None, **kw)` deserializes JSON from the file-like object `fp`. It reads the entire file content and delegates to `loads()` with the same customization parameters. The file object must support `.read()` and return `str` (for text files) or `bytes` (for binary files). `load()` does not close the file; the caller is responsible for managing the file’s lifetime, typically via the `with` statement.

**Beginner-Friendly Explanation**

`json.load()` is like `json.loads()`, but instead of taking a string, it takes an open file. You point it at a JSON file, and it reads the contents and gives you back Python data. It’s the standard way to read configuration files and data files stored on disk.

#### Purposes

- To read JSON data directly from files without manually reading the file content first.
- To load configuration files, data exports, and cached JSON documents.
- To support the same customization options as `loads()` for file-based input.
- To integrate cleanly with the `with` statement for resource management.
- To simplify code that would otherwise call `file.read()` followed by `json.loads()`.

#### Syntax Rules and Structure

**Complete General Syntax**

```
json.load(fp, *, cls=None, object_hook=None, parse_float=None,
          parse_int=None, parse_constant=None, object_pairs_hook=None, **kw)
```

**Component Breakdown**

| Parameter | Description |
|-----------|-------------|
| `fp` | A file-like object with a `.read()` method. |
| `cls`, `object_hook`, `parse_float`, `parse_int`, `parse_constant`, `object_pairs_hook` | Same as for `loads()`. |

**Syntax Rules**

- The file object must be opened in text mode (`'r'`) or binary mode (`'rb'`); both are supported.
- `load()` reads the entire file; for very large files, streaming approaches (e.g., JSON Lines) are preferable.
- The file is not closed by `load()`; use `with open(...) as f:`.
- Invalid JSON in the file raises `JSONDecodeError`.

**Constraints and Limitations**

- `load()` loads the entire file into memory; it is unsuitable for files larger than available RAM.
- File encoding must be handled when opening the file; use `encoding="utf-8"` for text mode.
- The file pointer is advanced to the end after reading.
- `load()` does not validate the JSON against any schema.

#### Annotated Code Examples

**Example 1: Reading a JSON File**

```python
import json

with open("config.json", "r", encoding="utf-8") as f:
    config = json.load(f)

print(config["database"]["host"])
```

**Expected Output:**

```
localhost
```

**Step-by-Step Breakdown:**

1. The file is opened with UTF-8 encoding.
2. `json.load(f)` reads the file and parses the JSON.
3. The resulting dictionary is accessed by key.

**Why the Output is Produced:**

`load()` reads the file content and delegates parsing to the same machinery as `loads()`.

#### Real-World Cases

- **Configuration loading**: Reading `config.json` at application startup.
- **Data import**: Loading JSON data exports into pandas DataFrames or Python objects.
- **Caching**: Reading cached API responses stored as JSON files.
- **Test fixtures**: Loading JSON fixtures for unit tests.

---

### 6. `json.dump()` — Serialize to a File

#### Definitions

**Core Definition**

`json.dump()` serializes a Python object and writes it as JSON to a file-like object.

**Technical Definition**

`json.dump(obj, fp, *, skipkeys=False, ensure_ascii=True, check_circular=True, allow_nan=True, cls=None, indent=None, separators=None, default=None, sort_keys=False, **kw)` writes the JSON representation of `obj` to the file-like object `fp`. The serialization parameters are identical to those of `dumps()`. The file object must support `.write()` and accept `str` (text mode) or `bytes` (binary mode). `dump()` does not close the file; the caller manages the file’s lifetime, typically with the `with` statement.

**Beginner-Friendly Explanation**

`json.dump()` is the file-writing counterpart of `json.dumps()`. Instead of giving you a string, it writes the JSON directly to a file you provide. It’s the easiest way to save Python data as a JSON file.

#### Purposes

- To persist Python data structures as JSON files for later use.
- To write configuration files, data exports, and cached documents.
- To produce human-readable JSON files with `indent` and `sort_keys`.
- To support custom serialization when writing files.
- To avoid the intermediate step of creating a string and then writing it.

#### Syntax Rules and Structure

**Complete General Syntax**

```
json.dump(obj, fp, *, skipkeys=False, ensure_ascii=True, check_circular=True,
          allow_nan=True, cls=None, indent=None, separators=None,
          default=None, sort_keys=False, **kw)
```

**Component Breakdown**

| Parameter | Description |
|-----------|-------------|
| `obj` | The Python object to serialize. |
| `fp` | A file-like object with a `.write()` method. |
| All other parameters | Same as for `dumps()`. |

**Syntax Rules**

- The file should be opened in text mode (`'w'`) with an explicit encoding, or in binary mode (`'wb'`).
- `dump()` writes the JSON representation to the file and returns `None`.
- The file is not closed by `dump()`.
- `indent` and `sort_keys` are commonly used for configuration files.

**Constraints and Limitations**

- `dump()` serializes the entire object in memory before writing; it is not a streaming serializer.
- Large objects may consume significant memory.
- The file is truncated if opened with `'w'`; use `'a'` with caution (appending JSON documents is rarely valid).
- The output is not guaranteed to be deterministic unless `sort_keys=True`.

#### Annotated Code Examples

**Example 1: Writing a JSON File**

```python
import json

data = {"name": "Alice", "age": 30, "skills": ["Python", "SQL"]}

with open("output.json", "w", encoding="utf-8") as f:
    json.dump(data, f, indent=2, sort_keys=True)
```

**Expected Output:**

```
(A file output.json is created with pretty-printed, sorted JSON.)
```

**Step-by-Step Breakdown:**

1. The file is opened for writing with UTF-8 encoding.
2. `json.dump()` writes the JSON representation to the file.
3. `indent=2` and `sort_keys=True` produce readable, deterministic output.

**Why the Output is Produced:**

`dump()` serializes the object and writes it to the file object, using the same encoder as `dumps()`.

#### Real-World Cases

- **Saving application state**: Persisting user preferences and session data.
- **Data export**: Writing processed results to JSON files for downstream systems.
- **Configuration management**: Generating default configuration files.
- **Test data generation**: Creating JSON fixtures programmatically.

---

### 7. JSON Serialization

#### Definitions

**Core Definition**

JSON serialization is the process of converting Python objects into JSON text, either as a string (`dumps()`) or directly to a file (`dump()`).

**Technical Definition**

Serialization in the `json` module traverses the Python object graph and emits JSON according to the standard type mapping. Natively serializable types include `dict`, `list`, `tuple`, `str`, `int`, `float`, `True`, `False`, and `None`. Non-serializable types are handled by the `default` callable (which must return a serializable object) or by subclassing `JSONEncoder` and overriding its `default()` method. The encoder performs circular-reference detection, allows or disallows non-finite floats, and supports options for pretty-printing, key sorting, and ASCII escaping.

**Beginner-Friendly Explanation**

Serialization is the “encoding” step: turning Python data into JSON text. Python knows how to convert dictionaries, lists, strings, numbers, and booleans automatically. For anything else—like a date or a custom object—you need to teach Python how to convert it, either by providing a `default` function or by creating a custom encoder.

#### Purposes

- To convert Python data into a portable text format.
- To prepare data for storage or transmission over a network.
- To support custom type conversion for domain-specific objects.
- To control output formatting for human readability or compactness.
- To detect and handle circular references and non-finite numbers.

#### Syntax Rules and Structure

**Complete General Syntax**

```
# Simple serialization.
json.dumps(obj)
json.dump(obj, fp)

# Custom serialization with default.
json.dumps(obj, default=my_converter)

# Custom serialization with encoder subclass.
class MyEncoder(json.JSONEncoder):
    def default(self, obj):
        ...
json.dumps(obj, cls=MyEncoder)
```

**Component Breakdown**

| Mechanism | Description |
|-----------|-------------|
| `default` callable | Invoked for unsupported types; must return a serializable object. |
| `JSONEncoder` subclass | Override `default()` for reusable custom serialization. |
| `skipkeys` | Skip non-string dictionary keys instead of raising `TypeError`. |
| `check_circular` | Detect circular references (default `True`). |
| `allow_nan` | Allow `NaN`, `Infinity`, `-Infinity` (default `True`). |

**Syntax Rules**

- Natively serializable types are converted automatically.
- `default` is called only for types not otherwise serializable.
- `default` must raise `TypeError` if it cannot handle the object; otherwise, invalid output may be produced.
- Subclassing `JSONEncoder` is preferable when the same custom logic is used repeatedly.
- `sort_keys=True` ensures deterministic output for dictionaries.

**Constraints and Limitations**

- `default` cannot serialize types that the encoder considers supported, even if a subclass would handle them differently.
- Circular references raise `ValueError` unless explicitly disabled.
- Non-finite floats are not standard JSON but are allowed by Python’s default settings.

#### Annotated Code Examples

**Example 1: Serializing with a `default` Function**

```python
import json
from datetime import datetime

def serialize_datetime(obj):
    if isinstance(obj, datetime):
        return obj.isoformat()
    raise TypeError(f"Cannot serialize {type(obj)}")

data = {"event": "launch", "timestamp": datetime(2025, 1, 1, 12, 0)}
print(json.dumps(data, default=serialize_datetime))
```

**Expected Output:**

```
{"event": "launch", "timestamp": "2025-01-01T12:00:00"}
```

**Step-by-Step Breakdown:**

1. `datetime` is not natively serializable.
2. `default` converts it to an ISO 8601 string.
3. The resulting JSON contains the string.

**Why the Output is Produced:**

The `default` function acts as a fallback for unsupported types.

---

**Example 2: Subclassing `JSONEncoder`**

```python
import json

class ComplexEncoder(json.JSONEncoder):
    def default(self, obj):
        if isinstance(obj, complex):
            return [obj.real, obj.imag]
        return super().default(obj)

print(json.dumps(2 + 1j, cls=ComplexEncoder))
```

**Expected Output:**

```
[2.0, 1.0]
```

**Step-by-Step Breakdown:**

1. `ComplexEncoder` overrides `default()`.
2. Complex numbers are converted to a list of real and imaginary parts.
3. The base class’s `default()` is called for other unsupported types.

**Why the Output is Produced:**

Subclassing `JSONEncoder` provides a reusable serialization strategy.

#### Real-World Cases

- **Datetime handling**: Converting `datetime` objects to ISO strings for APIs.
- **Decimal precision**: Serializing `Decimal` as strings to preserve precision.
- **Custom domain objects**: Converting user-defined classes to dictionaries.
- **UUID and enum serialization**: Converting `UUID` and `Enum` members to strings.

---

### 8. JSON Deserialization

#### Definitions

**Core Definition**

JSON deserialization is the process of converting JSON text into Python objects, either from a string (`loads()`) or from a file (`load()`).

**Technical Definition**

Deserialization in the `json` module parses JSON text and produces Python objects according to the standard mapping. Customization is available through `object_hook` (called with every decoded `dict`), `object_pairs_hook` (called with a list of pairs, taking precedence), `parse_float` and `parse_int` (called with the string representation of numbers), and `parse_constant` (called for `NaN`, `Infinity`, `-Infinity`). Invalid JSON raises `JSONDecodeError`, which provides positional information about the failure.

**Beginner-Friendly Explanation**

Deserialization is the “decoding” step: turning JSON text back into Python data. Python gives you dictionaries and lists automatically. If you want JSON objects to become custom Python objects—like a `datetime` from a string—you can provide a hook function that processes each object as it is decoded.

#### Purposes

- To convert JSON text into usable Python data structures.
- To reconstruct domain objects from JSON representations.
- To parse numeric values with custom types for precision or performance.
- To handle JSON from APIs, files, and message queues.
- To provide clear error reporting for malformed JSON.

#### Syntax Rules and Structure

**Complete General Syntax**

```
json.loads(s, object_hook=my_hook)
json.loads(s, object_pairs_hook=my_pairs_hook)
json.loads(s, parse_float=decimal.Decimal)
json.load(fp, object_hook=my_hook)
```

**Component Breakdown**

| Hook | Description |
|------|-------------|
| `object_hook` | Called with every decoded `dict`; return value replaces the dict. |
| `object_pairs_hook` | Called with a list of `(key, value)` pairs; takes precedence over `object_hook`. |
| `parse_float` | Called with the string of every float; default is `float`. |
| `parse_int` | Called with the string of every int; default is `int`. |
| `parse_constant` | Called for `NaN`, `Infinity`, `-Infinity`. |

**Syntax Rules**

- `object_hook` is ideal for recognizing marker keys and returning domain objects.
- `object_pairs_hook` preserves key order and duplicate keys.
- `parse_float=decimal.Decimal` is useful for financial data.
- Hooks should return the appropriate Python object.

**Constraints and Limitations**

- `object_hook` is called bottom-up: inner objects are processed before outer ones.
- `object_pairs_hook` receives the raw list of pairs and must handle duplicates explicitly.
- `parse_float` and `parse_int` receive strings, not numbers.
- Custom hooks can introduce security risks if they evaluate untrusted input.

#### Annotated Code Examples

**Example 1: Custom Decoding with `object_hook`**

```python
import json

def datetime_hook(dct):
    for key, value in dct.items():
        if key.endswith("_time"):
            try:
                dct[key] = datetime.fromisoformat(value)
            except (ValueError, TypeError):
                pass
    return dct

json_string = '{"start_time": "2025-10-15T09:30:00", "title": "Conference"}'
data = json.loads(json_string, object_hook=datetime_hook)
print(type(data["start_time"]))
```

**Expected Output:**

```
<class 'datetime.datetime'>
```

**Step-by-Step Breakdown:**

1. The hook inspects each dictionary for keys ending in `_time`.
2. Matching string values are converted to `datetime` objects.
3. The modified dictionary is returned.

**Why the Output is Produced:**

`object_hook` transforms dictionaries during decoding, enabling reconstruction of typed objects.

---

**Example 2: Precise Decimal Parsing**

```python
import json
import decimal

json_string = '{"price": 19.99}'
data = json.loads(json_string, parse_float=decimal.Decimal)
print(data["price"])
print(type(data["price"]))
```

**Expected Output:**

```
19.99
<class 'decimal.Decimal'>
```

**Step-by-Step Breakdown:**

1. `parse_float=decimal.Decimal` is specified.
2. The float value `19.99` is decoded as a `Decimal` object instead of a binary float.

**Why the Output is Produced:**

`parse_float` allows precise decimal representation for financial or scientific data.

#### Real-World Cases

- **API response parsing**: Converting JSON payloads into application-specific objects.
- **Financial systems**: Using `Decimal` for monetary values.
- **Date and time handling**: Converting ISO strings to `datetime` objects via hooks.
- **Configuration validation**: Decoding JSON and validating the resulting structure.

---

### 9. Custom JSON Encoders

#### Definitions

**Core Definition**

A custom JSON encoder is a user-defined mechanism—either a `default` callable or a subclass of `JSONEncoder`—that teaches the `json` module how to serialize types it does not natively support.

**Technical Definition**

Custom encoding can be implemented at three levels: (1) a `default` callable passed to `dumps()` or `dump()`, which is invoked for unsupported objects and must return a serializable object; (2) a subclass of `json.JSONEncoder` that overrides `default()`, allowing reusable encoding logic across multiple call sites; and (3) in future Python versions (per PEP 837), a `__json__()` protocol and a `copyreg.json()` registry for library-level and application-level extensibility. The `default` callable is the simplest and most common approach.

**Beginner-Friendly Explanation**

A custom encoder is a translator you write for types Python doesn’t automatically know how to convert to JSON. For example, if you have a `datetime`, you can tell Python: “When you see a datetime, convert it to an ISO string.” You can do this with a one-off function for a single call, or by creating a reusable encoder class if you need the same translation in many places.

#### Purposes

- To serialize non-standard Python types such as `datetime`, `Decimal`, `UUID`, `set`, and custom classes.
- To control the JSON representation of domain objects.
- To reuse serialization logic across multiple calls via a custom encoder class.
- To ensure consistent output formatting for special types.
- To integrate with libraries and frameworks that accept custom encoders.

#### Syntax Rules and Structure

**Complete General Syntax**

```
# One-off default callable.
json.dumps(obj, default=my_converter)

# Reusable encoder subclass.
class MyEncoder(json.JSONEncoder):
    def default(self, obj):
        if isinstance(obj, MyType):
            return ...
        return super().default(obj)

json.dumps(obj, cls=MyEncoder)
```

**Component Breakdown**

| Approach | Description |
|----------|-------------|
| `default` callable | Simplest; invoked for unsupported types; must return serializable object. |
| `JSONEncoder` subclass | Reusable; override `default()`; call `super().default(obj)` for unhandled types. |
| `__json__` protocol (future) | Library-level opt-in via PEP 837. |

**Syntax Rules**

- `default` is called only for objects not natively serializable.
- It must return a serializable object or raise `TypeError`.
- Subclassing `JSONEncoder` is preferable when the logic is reused.
- The `cls` parameter specifies the encoder class.

**Constraints and Limitations**

- `default` cannot override handling for natively supported types.
- The `default` method in a subclass is not called for types the base encoder considers supported.
- PEP 837 is a draft and not yet part of the standard library in current Python versions.

#### Annotated Code Examples

**Example 1: `default` for `Decimal`**

```python
import json
import decimal

def decimal_default(obj):
    if isinstance(obj, decimal.Decimal):
        return str(obj)
    raise TypeError(f"Cannot serialize {type(obj)}")

data = {"price": decimal.Decimal("19.99")}
print(json.dumps(data, default=decimal_default))
```

**Expected Output:**

```
{"price": "19.99"}
```

**Step-by-Step Breakdown:**

1. `Decimal` is not natively serializable.
2. `default` converts it to a string.
3. The JSON output contains the string value.

**Why the Output is Produced:**

The `default` function provides a fallback conversion for unsupported types.

---

**Example 2: `JSONEncoder` Subclass for `datetime` and `UUID`**

```python
import json
from datetime import datetime
from uuid import UUID

class ExtendedEncoder(json.JSONEncoder):
    def default(self, obj):
        if isinstance(obj, datetime):
            return obj.isoformat()
        if isinstance(obj, UUID):
            return str(obj)
        return super().default(obj)

data = {
    "created": datetime(2025, 1, 1, 12, 0),
    "id": UUID("12345678-1234-5678-1234-567812345678")
}
print(json.dumps(data, cls=ExtendedEncoder))
```

**Expected Output:**

```
{"created": "2025-01-01T12:00:00", "id": "12345678-1234-5678-1234-567812345678"}
```

**Step-by-Step Breakdown:**

1. `ExtendedEncoder` handles both `datetime` and `UUID`.
2. Unhandled types are passed to the base class’s `default()`, which raises `TypeError`.

**Why the Output is Produced:**

The subclass provides reusable serialization logic for multiple custom types.

#### Real-World Cases

- **Web APIs**: Serializing `datetime` and `UUID` fields in responses.
- **Data pipelines**: Converting `Decimal` and `set` objects during export.
- **Logging**: Converting custom log-record objects to JSON.
- **Scientific computing**: Serializing NumPy scalars and arrays (via custom encoders).

---

### 10. Custom JSON Decoders

#### Definitions

**Core Definition**

A custom JSON decoder is a user-defined mechanism—either an `object_hook`, `object_pairs_hook`, `parse_float`/`parse_int`, or a `JSONDecoder` subclass—that controls how JSON text is converted into Python objects.

**Technical Definition**

Custom decoding can be implemented through several hooks: `object_hook` is called with every decoded `dict` and can return a replacement object; `object_pairs_hook` is called with a list of key–value pairs and takes precedence over `object_hook`; `parse_float` and `parse_int` are called with the string representation of each numeric value; and `parse_constant` is called for non-finite constants. For more complex scenarios, subclassing `JSONDecoder` and overriding `__init__` or `decode` is possible. These mechanisms enable reconstruction of domain objects, precise numeric parsing, and order-preserving or duplicate-aware decoding.

**Beginner-Friendly Explanation**

A custom decoder is the reverse of a custom encoder: it tells Python how to turn JSON data back into the objects you expect. For example, if your JSON contains a field like `"__datetime__": "2025-01-01T12:00:00"`, you can write a hook that recognizes the marker and converts the string to a real `datetime` object. Hooks are the easiest way to customize decoding; subclassing `JSONDecoder` is available for more advanced needs.

#### Purposes

- To reconstruct domain-specific Python objects from JSON representations.
- To parse numeric values with `Decimal` or other custom types.
- To preserve key order or handle duplicate keys during decoding.
- To centralize decoding logic for reuse across multiple calls.
- To validate and transform data as it is decoded.

#### Syntax Rules and Structure

**Complete General Syntax**

```
# object_hook
json.loads(s, object_hook=my_hook)

# object_pairs_hook
json.loads(s, object_pairs_hook=my_pairs_hook)

# parse_float / parse_int
json.loads(s, parse_float=decimal.Decimal, parse_int=int)

# JSONDecoder subclass
class MyDecoder(json.JSONDecoder):
    def __init__(self, **kwargs):
        super().__init__(object_hook=my_hook, **kwargs)
```

**Component Breakdown**

| Hook | Description |
|------|-------------|
| `object_hook` | Called with every decoded `dict`; return value replaces the dict. |
| `object_pairs_hook` | Called with a list of `(key, value)` pairs; takes precedence. |
| `parse_float` | Called with the string of every float. |
| `parse_int` | Called with the string of every int. |
| `parse_constant` | Called for `NaN`, `Infinity`, `-Infinity`. |

**Syntax Rules**

- `object_hook` is ideal for marker-key-based reconstruction.
- `object_pairs_hook` preserves key order and exposes duplicates.
- `parse_float=decimal.Decimal` is common for financial data.
- Hooks are called bottom-up: inner objects are processed before outer ones.

**Constraints and Limitations**

- `object_hook` cannot prevent the creation of the intermediate `dict`; it can only replace it.
- `object_pairs_hook` receives raw pairs and must handle duplicate keys explicitly.
- Custom decoders can introduce security risks if they evaluate untrusted input.
- Subclassing `JSONDecoder` is rarely necessary; hooks cover most use cases.

#### Annotated Code Examples

**Example 1: `object_hook` for Marker-Based Reconstruction**

```python
import json
from datetime import datetime

def decode_datetime(dct):
    if "__datetime__" in dct:
        return datetime.fromisoformat(dct["__datetime__"])
    return dct

json_string = '{"__datetime__": "2025-01-01T12:00:00"}'
result = json.loads(json_string, object_hook=decode_datetime)
print(result)
print(type(result))
```

**Expected Output:**

```
2025-01-01 12:00:00
<class 'datetime.datetime'>
```

**Step-by-Step Breakdown:**

1. The JSON object contains a `__datetime__` marker.
2. `object_hook` recognizes the marker and returns a `datetime` object.
3. The result replaces the intermediate dictionary.

**Why the Output is Produced:**

`object_hook` allows transformation of decoded dictionaries into domain objects.

---

**Example 2: `object_pairs_hook` for Order-Preserving Decoding**

```python
import json
from collections import OrderedDict

def ordered_hook(pairs):
    return OrderedDict(pairs)

json_string = '{"b": 2, "a": 1, "b": 3}'
result = json.loads(json_string, object_pairs_hook=ordered_hook)
print(result)
```

**Expected Output:**

```
OrderedDict([('b', 2), ('a', 1), ('b', 3)])
```

**Step-by-Step Breakdown:**

1. `object_pairs_hook` receives the raw list of pairs.
2. Duplicate keys are preserved.
3. The result is an `OrderedDict`.

**Why the Output is Produced:**

`object_pairs_hook` provides access to the raw pair list, enabling order preservation and duplicate detection.

#### Real-World Cases

- **Typed API responses**: Converting JSON payloads into dataclass or Pydantic models.
- **Financial data**: Parsing JSON numbers as `Decimal` for precision.
- **Legacy systems**: Handling JSON with duplicate keys that must be preserved.
- **Configuration management**: Decoding JSON settings into application-specific configuration objects.

---

### 11. JSON Best Practices

#### Definitions

**Core Definition**

JSON best practices are guidelines for using the `json` module safely, efficiently, and portably, covering serialization, deserialization, security, performance, and interoperability.

**Technical Definition**

Best practices for Python JSON include: (1) always use `json` (never `pickle`) for data from untrusted sources, because `pickle` can execute arbitrary code during deserialization; (2) limit the size of JSON input to prevent denial-of-service via deeply nested or extremely large documents; (3) specify `encoding="utf-8"` when opening JSON files, as recommended by RFC 7159; (4) use `ensure_ascii=False` when non-ASCII text is expected, and ensure the consumer handles Unicode; (5) prefer `sort_keys=True` for deterministic output in testing and configuration; (6) use `indent` for human-readable files and `separators` for compact output; (7) handle `JSONDecodeError` explicitly when parsing external data; (8) use `parse_float=decimal.Decimal` for precision-sensitive numeric data; (9) use custom encoders and decoders for domain types rather than relying on ad-hoc conversions; (10) for large datasets, stream with JSON Lines or use faster libraries such as `orjson` or `msgspec`.

**Beginner-Friendly Explanation**

Best practices are the “good habits” of working with JSON. The most important one is safety: never use `pickle` on untrusted data, and be careful with huge JSON files because they can exhaust memory. When writing files, always specify UTF-8 encoding. When reading, catch errors so your program doesn’t crash. And when you have special data types like dates or decimals, write a custom encoder or decoder instead of converting them by hand every time.

#### Purposes

- To prevent security vulnerabilities such as code execution and denial-of-service.
- To ensure correct handling of text encoding across platforms.
- To produce deterministic, human-readable output for configuration and testing.
- To maintain precision for numeric data.
- To handle large datasets without exhausting memory.
- To write maintainable, reusable JSON-handling code.

#### Syntax Rules and Structure

**Complete General Syntax**

```
# Secure file reading with explicit encoding.
with open("data.json", "r", encoding="utf-8") as f:
    try:
        data = json.load(f)
    except json.JSONDecodeError as e:
        ...

# Deterministic output.
json.dump(data, f, indent=2, sort_keys=True, ensure_ascii=False)

# Precision-sensitive parsing.
json.loads(s, parse_float=decimal.Decimal)

# Large file streaming (JSON Lines).
with open("data.jsonl", "r", encoding="utf-8") as f:
    for line in f:
        record = json.loads(line)
```

**Component Breakdown**

| Practice | Description |
|----------|-------------|
| Use `json` over `pickle` | `pickle` can execute arbitrary code; `json` is safer. |
| Limit input size | Prevent resource exhaustion from malicious JSON. |
| Explicit UTF-8 encoding | RFC 7159 recommends UTF-8 for best interoperability. |
| `ensure_ascii=False` | Emit non-ASCII characters as-is when appropriate. |
| `sort_keys=True` | Deterministic output for testing and configuration. |
| `indent` / `separators` | Balance readability and compactness. |
| Handle `JSONDecodeError` | Graceful recovery from malformed input. |
| `parse_float=Decimal` | Preserve numeric precision. |
| Custom encoders/decoders | Reusable handling of domain types. |
| Stream large files | Avoid loading entire datasets into memory. |

**Syntax Rules**

- Always specify `encoding="utf-8"` for text-mode file operations.
- Use `try`/`except json.JSONDecodeError` when parsing external data.
- Prefer `sort_keys=True` when output will be compared or version-controlled.
- Use `ensure_ascii=False` only when the consumer can handle Unicode.
- For numeric precision, use `parse_float=decimal.Decimal` and a custom encoder for `Decimal`.
- For large files, use JSON Lines (`.jsonl`) and iterate line by line.

**Constraints and Limitations**

- `ensure_ascii=False` may produce output that is not ASCII-safe; ensure downstream systems support Unicode.
- `sort_keys=True` only sorts dictionary keys, not list elements.
- `parse_float=Decimal` affects only floats; integers remain `int` unless `parse_int` is also customized.
- JSON Lines is not a single JSON document and cannot be parsed with `json.load()`.
- Third-party libraries like `orjson` and `msgspec` offer speed improvements but have different APIs and limitations.

#### Annotated Code Examples

**Example 1: Secure and Deterministic File Writing**

```python
import json

data = {"z": 1, "a": 2, "name": "Ünïcode"}

with open("output.json", "w", encoding="utf-8") as f:
    json.dump(data, f, indent=2, sort_keys=True, ensure_ascii=False)
```

**Expected Output:**

```
(A UTF-8 file with sorted keys and raw Unicode characters.)
```

**Step-by-Step Breakdown:**

1. The file is opened with explicit UTF-8 encoding.
2. `sort_keys=True` ensures deterministic key order.
3. `ensure_ascii=False` writes `Ünïcode` as-is.
4. `indent=2` produces readable output.

**Why the Output is Produced:**

This combination balances readability, determinism, and correct Unicode handling.

---

**Example 2: Safe Parsing with Error Handling**

```python
import json

untrusted = '{"valid": true, "data": [1, 2, 3]}'

try:
    data = json.loads(untrusted)
except json.JSONDecodeError as e:
    print(f"Invalid JSON: {e}")
else:
    print("Parsed successfully:", data)
```

**Expected Output:**

```
Parsed successfully: {'valid': True, 'data': [1, 2, 3]}
```

**Step-by-Step Breakdown:**

1. `json.loads()` parses the string.
2. If parsing fails, `JSONDecodeError` is caught.
3. Otherwise, the parsed data is used.

**Why the Output is Produced:**

Explicit error handling prevents crashes when input is malformed.

---

**Example 3: Streaming JSON Lines**

```python
import json

with open("events.jsonl", "r", encoding="utf-8") as f:
    for line in f:
        event = json.loads(line)
        print(event["type"])
```

**Expected Output:**

```
(Each event type printed line by line.)
```

**Step-by-Step Breakdown:**

1. The file is opened and iterated line by line.
2. Each line is independently parsed as JSON.
3. Memory usage remains constant regardless of file size.

**Why the Output is Produced:**

JSON Lines allows streaming deserialization, avoiding the need to load the entire file into memory.

#### Real-World Cases

- **Web services**: Parsing untrusted API payloads with size limits and error handling.
- **Data engineering**: Streaming multi-gigabyte JSONL logs without memory exhaustion.
- **Financial applications**: Using `Decimal` for monetary values to avoid floating-point errors.
- **Configuration management**: Writing deterministic, sorted JSON configs for version control.
- **Cross-platform tools**: Using UTF-8 encoding to ensure JSON files work identically on Windows, Linux, and macOS.

---

## References

- Python Documentation – `json` — JSON encoder and decoder (latest) - https://docs.python.org/3/library/json.html
- Python Documentation – `json` — JSON encoder and decoder (3.14, Chinese) - https://docs.python.org/zh-cn/3.14/library/json.html
- Real Python – Working With JSON Data in Python - https://realpython.com/python-json/
- Python JSON: Parse, Read, Write, and Convert JSON Data - https://docs.kanaries.net/topics/Python/python-json
- Python处理JSON数据的最佳实践：从基础到进阶的实用指南 - https://cloud.tencent.cn/developer/article/2557985
- PEP 837 – Extensible JSON serialization - https://peps.python.org/pep-0837/
- RFC 7159 – The JavaScript Object Notation (JSON) Data Interchange Format - https://datatracker.ietf.org/doc/html/rfc7159.html
- ECMA-404 – The JSON Data Interchange Syntax - https://www.ecma-international.org/publications-and-standards/standards/ecma-404/
- json — load | Precaution (SecureSauce) - https://docs.securesauce.dev
- Working with JSON (wolfcodes_docs) - https://github.com/wolf-math/wolfcodes_docs/blob/main/python/guides/standard_library/json.md
- JSON Lines – Streaming large files best practices - https://github.com/tteon/seocho