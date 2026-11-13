# Python YAML: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**

YAML (YAML Ain’t Markup Language) is a human-friendly, cross-language, Unicode-based data serialization language designed around the common native data types of dynamic programming languages. Python’s de facto YAML library is **PyYAML** (import name `yaml`), which provides parsing (loading) and emission (dumping) of YAML documents.

**Technical Definition**

YAML is a data serialization language specified by the YAML 1.2.2 specification (revision 1.2.2, 2021). It represents data using three fundamental node types—scalars, sequences (lists), and mappings (dictionaries)—with support for anchors, aliases, tags, and multi-document streams. PyYAML is a full-featured YAML framework for Python that implements a loader/dumper architecture. Its central security distinction is between the **safe API** (`safe_load` / `safe_dump`), which constructs only standard Python types (scalars, lists, dicts), and the **full API** (`load` / `dump`), which can construct arbitrary Python objects and therefore carries significant security risk.

**Beginner-Friendly Explanation**

YAML is a way to write data in a format that is easy for humans to read and write—similar to JSON but cleaner, with indentation instead of braces and brackets. Python’s PyYAML package is your tool for reading YAML files into Python objects and writing Python objects back out as YAML. The most important thing to remember is to use `yaml.safe_load()` instead of `yaml.load()` when you are reading YAML from an untrusted source, because the unsafe loader can execute arbitrary code.

### Key Characteristics

- **Human-readable**: YAML uses indentation-based structure that is easier to scan than JSON’s punctuation.
- **Safe vs. full API**: `safe_load` / `safe_dump` restrict construction to standard Python types; `load` / `dump` can construct arbitrary objects and are dangerous with untrusted input.
- **Multi-document support**: A single YAML stream can contain multiple documents separated by `---`; PyYAML provides `safe_load_all` / `safe_dump_all`.
- **Custom tags**: PyYAML allows extending `SafeLoader` with explicitly registered constructors for custom tags.
- **LibYAML acceleration**: `CSafeLoader` / `CSafeDumper` provide faster parsing and emission when the C library is available.
- **YAML 1.1 support**: PyYAML implements YAML 1.1, not YAML 1.2; notable differences include boolean interpretation (e.g., `yes`/`no` are booleans in YAML 1.1).

### Prerequisites

- Basic Python syntax: dictionaries, lists, functions, and file I/O.
- Familiarity with the `with` statement and exception handling.
- Understanding of the difference between trusted and untrusted data sources.
- Awareness of YAML’s indentation-based syntax.

### Related Programming Areas

- **Configuration management**: YAML is dominant in Kubernetes manifests, CI/CD pipelines (GitHub Actions, GitLab CI), and application configs.
- **Data serialization alternatives**: JSON, TOML, and XML serve overlapping roles.
- **Infrastructure as Code**: Ansible, Docker Compose, and Helm use YAML heavily.
- **Security**: Unsafe YAML loading is a recurring source of remote code execution vulnerabilities.

---

## Core Concepts / Features

---

### 1. YAML (YAML Ain’t Markup Language)

#### Definitions

**Core Definition**

YAML is a data serialization language that uses indentation, colons, and dashes to represent structured data in a form that is readable by both humans and machines.

**Technical Definition**

YAML is specified by the YAML 1.2.2 specification (October 2021), which defines the language as a strict superset of JSON. The specification describes three node types: scalars (strings, numbers, booleans, null), sequences (ordered lists), and mappings (key-value pairs). YAML supports advanced features including anchors and aliases (for referencing repeated nodes), tags (for typing), multi-document streams (separated by `---`), and block vs. flow styles. PyYAML implements YAML 1.1, which predates the JSON-compatibility goal of YAML 1.2 and has different boolean and type-resolution rules.

**Beginner-Friendly Explanation**

YAML is like a cleaner, indentation-based version of JSON. Instead of curly braces and square brackets, you use indentation and dashes. It is extremely popular for configuration files because it is easy to read and edit by hand. Python’s PyYAML library reads YAML and turns it into dictionaries and lists, and writes dictionaries and lists back out as YAML.

#### Purposes

- To provide a human-readable format for configuration files and data exchange.
- To serve as a portable, language-independent serialization format.
- To support complex data structures with anchors, aliases, and tags.
- To enable multi-document streams for batch processing.
- To act as the configuration language for major infrastructure tools (Kubernetes, Ansible, GitHub Actions).

#### Syntax Rules and Structure

**Complete General Syntax**

```yaml
# A YAML document
name: Alice
age: 30
skills:
  - Python
  - SQL
address:
  city: New York
  zip: "10001"
active: true
metadata: null
```

**Component Breakdown**

| YAML Construct | Description | Python Equivalent |
|----------------|-------------|-------------------|
| Mapping | `key: value` | `dict` |
| Sequence | `- item` | `list` |
| Scalar | `value` | `str`, `int`, `float`, `bool`, `None` |
| Document separator | `---` | (stream boundary) |
| Anchor | `&name` | (reference definition) |
| Alias | `*name` | (reference use) |
| Tag | `!!type value` | (type hint) |

**Syntax Rules**

- Indentation (spaces, not tabs) defines structure.
- Mappings use `key: value` with a space after the colon.
- Sequences use `- ` (dash followed by a space).
- Strings may be unquoted, single-quoted, or double-quoted.
- `true`/`false` and `null`/`~` represent booleans and null.
- Multiple documents in one stream are separated by `---`.
- Comments begin with `#`.

**Constraints and Limitations**

- YAML 1.1 (implemented by PyYAML) interprets `yes`, `no`, `on`, `off` as booleans, which differs from YAML 1.2 and JSON.
- Indentation must be consistent; tabs are not allowed for indentation.
- YAML is sensitive to whitespace, which can cause subtle bugs.
- The full YAML specification is complex; security features like tags can be abused.

#### Annotated Code Examples

**Example 1: A Simple YAML Document**

```yaml
# config.yaml
database:
  host: localhost
  port: 5432
  credentials:
    username: admin
    password: secret
features:
  - logging
  - caching
debug: false
```

**Expected Output (parsed by `safe_load`):**

```
{'database': {'host': 'localhost', 'port': 5432, 'credentials': {'username': 'admin', 'password': 'secret'}}, 'features': ['logging', 'caching'], 'debug': False}
```

**Step-by-Step Breakdown:**

1. `database` is a mapping containing `host`, `port`, and nested `credentials`.
2. `features` is a sequence of strings.
3. `debug` is a boolean scalar.

**Why the Output is Produced:**

PyYAML maps YAML mappings to Python `dict`, sequences to `list`, and scalars to `str`, `int`, `bool`, etc., following YAML 1.1 resolution rules.

#### Real-World Cases

- **Kubernetes manifests**: Pod, Service, and Deployment definitions are YAML.
- **CI/CD pipelines**: GitHub Actions workflows (`.github/workflows/*.yml`) and GitLab CI (`.gitlab-ci.yml`) are YAML.
- **Application configuration**: Many Python frameworks (Django, Flask extensions) support YAML configs.
- **Ansible playbooks**: Automation scripts are written in YAML.

---

### 2. PyYAML

#### Definitions

**Core Definition**

PyYAML is the standard Python package for parsing YAML into Python objects and emitting Python objects as YAML.

**Technical Definition**

PyYAML is a full-featured YAML framework for Python, distributed on PyPI as `PyYAML` and imported as `yaml`. It implements a loader/dumper architecture with multiple loader classes: `SafeLoader` (restricted to standard Python types), `FullLoader` (full YAML but historically vulnerable), `UnsafeLoader` (arbitrary object construction), and `BaseLoader` (all scalars as strings). Dumper classes mirror these. LibYAML-backed `CSafeLoader` and `CSafeDumper` provide faster C-based parsing and emission when available. Since PyYAML 5.1, `yaml.load()` without an explicit `Loader` argument emits a warning and, since PyYAML 6.0, the `Loader` argument is mandatory.

**Beginner-Friendly Explanation**

PyYAML is the Python library that lets you work with YAML. You install it with `pip install PyYAML` and then `import yaml` in your code. It gives you simple functions like `yaml.safe_load()` for reading YAML and `yaml.safe_dump()` for writing it. Behind the scenes, it has different “loaders” that control how much power the YAML parser has—the safe loader is the one you should almost always use.

#### Purposes

- To parse YAML documents into Python objects and serialize Python objects to YAML.
- To provide safe and unsafe loading modes with explicit opt-in for dangerous features.
- To support custom tags and types through loader subclassing.
- To enable multi-document streams with `safe_load_all` and `safe_dump_all`.
- To leverage LibYAML for high-performance parsing when available.

#### Syntax Rules and Structure

**Complete General Syntax**

```
pip install PyYAML
import yaml

data = yaml.safe_load(stream)
text = yaml.safe_dump(data, sort_keys=False)
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `safe_load(stream)` | Parses YAML into standard Python types only. |
| `safe_load_all(stream)` | Parses a multi-document stream into a generator of objects. |
| `safe_dump(data)` | Emits standard Python objects as YAML. |
| `safe_dump_all(documents)` | Emits multiple documents as a YAML stream. |
| `SafeLoader` | Loader class restricted to standard types. |
| `CSafeLoader` | LibYAML-backed safe loader (faster). |

**Syntax Rules**

- Install with `pip install PyYAML`.
- `safe_load` accepts a string, bytes, or file object.
- `safe_dump` accepts standard Python types (`dict`, `list`, `str`, `int`, `float`, `bool`, `None`).
- For custom tags, subclass `SafeLoader` and register constructors explicitly.
- Use `sort_keys=False` in `safe_dump` to preserve insertion order (PyYAML sorts keys by default).

**Constraints and Limitations**

- PyYAML implements YAML 1.1, not YAML 1.2; type resolution differs from JSON-superset YAML 1.2.
- `yaml.load()` without a safe loader is unsafe and should never be used on untrusted data.
- `FullLoader` has a history of security vulnerabilities despite its name.
- PyYAML does not support all YAML 1.2 features (e.g., JSON-style booleans in all contexts).

#### Annotated Code Examples

**Example 1: Basic Safe Loading and Dumping**

```python
import yaml

yaml_text = """
debug: true
port: 8080
hosts:
  - api.internal
  - worker.internal
"""

data = yaml.safe_load(yaml_text)
print(data)

output = yaml.safe_dump(data, sort_keys=False)
print(output)
```

**Expected Output:**

```
{'debug': True, 'port': 8080, 'hosts': ['api.internal', 'worker.internal']}
debug: true
port: 8080
hosts:
- api.internal
- worker.internal
```

**Step-by-Step Breakdown:**

1. `yaml.safe_load()` parses the YAML string into a Python dictionary.
2. `yaml.safe_dump()` serializes the dictionary back to YAML.
3. `sort_keys=False` preserves the original key order.

**Why the Output is Produced:**

`safe_load` maps YAML constructs to standard Python types; `safe_dump` performs the reverse mapping, emitting block-style YAML by default.

#### Real-World Cases

- **Configuration loading**: Reading `config.yaml` into application settings.
- **Kubernetes tooling**: Parsing and generating manifests programmatically.
- **CI/CD automation**: Modifying workflow files with Python scripts.

---

### 3. `yaml.safe_load()`

#### Definitions

**Core Definition**

`yaml.safe_load()` parses a YAML document into Python objects using only standard YAML tags and standard Python types, without the ability to construct arbitrary Python objects.

**Technical Definition**

`yaml.safe_load(stream)` is a convenience function that parses the given YAML stream using `SafeLoader`. It returns the first document in the stream as a Python object (typically a `dict`, `list`, or scalar). `SafeLoader` recognizes only the standard YAML tags (`!!str`, `!!int`, `!!float`, `!!bool`, `!!null`, `!!seq`, `!!map`, `!!timestamp`, `!!binary`, etc.) and does not resolve Python-specific tags such as `!!python/object`. This restriction is the primary defense against YAML-based remote code execution. For multi-document streams, `safe_load_all()` should be used instead. An empty document returns `None`; a scalar-only document returns that scalar’s Python value.

**Beginner-Friendly Explanation**

`yaml.safe_load()` is the safe way to read YAML in Python. It takes a YAML string or file and gives you back normal Python data—dictionaries, lists, strings, numbers, booleans, and `None`. It deliberately refuses to create custom Python objects, which means it cannot be tricked into running malicious code. If you are reading a YAML file that you did not write yourself, this is the function you should use.

#### Purposes

- To safely parse YAML from untrusted or semi-trusted sources.
- To convert YAML configuration files into Python dictionaries and lists.
- To prevent YAML-based remote code execution and object injection.
- To serve as the default loader for ordinary application configuration.
- To support multi-document streams via `safe_load_all()`.

#### Syntax Rules and Structure

**Complete General Syntax**

```
yaml.safe_load(stream)
yaml.safe_load_all(stream)
```

**Component Breakdown**

| Parameter | Description |
|-----------|-------------|
| `stream` | A string, bytes, or file-like object containing YAML. |
| Return value | The first document’s Python representation (or a generator for `_all`). |

**Syntax Rules**

- The input must be a valid YAML document (or stream for `_all`).
- `safe_load` returns a single Python object; `safe_load_all` returns a generator.
- Standard YAML tags are resolved to standard Python types.
- Python-specific tags (`!!python/...`) raise `ConstructorError`.
- Empty input returns `None`.

**Constraints and Limitations**

- `safe_load` cannot construct custom Python classes or objects.
- It does not support `!!python/` tags; attempting to use them raises an error.
- It is slightly slower than the unsafe loader for certain documents because it performs more checks.
- It does not validate against a schema; type and structure validation must be done after loading.

#### Annotated Code Examples

**Example 1: Safe Loading of Configuration**

```python
import yaml

config_yaml = """
server:
  host: 0.0.0.0
  port: 8000
  debug: false
workers: 4
"""

config = yaml.safe_load(config_yaml)
print(config["server"]["host"])
print(config["workers"])
```

**Expected Output:**

```
0.0.0.0
4
```

**Step-by-Step Breakdown:**

1. `yaml.safe_load()` parses the YAML into a dictionary.
2. Nested dictionaries and scalars are accessed normally.

**Why the Output is Produced:**

`SafeLoader` maps YAML mappings to `dict` and scalars to standard Python types, making the result immediately usable.

---

**Example 2: Rejection of Unsafe Tags**

```python
import yaml

malicious_yaml = "value: !!python/object/apply:os.system ['echo hacked']"

try:
    yaml.safe_load(malicious_yaml)
except yaml.constructor.ConstructorError as e:
    print("Blocked unsafe tag:", e)
```

**Expected Output:**

```
Blocked unsafe tag: could not determine a constructor for the tag 'tag:yaml.org,2002:python/object/apply:os.system'
```

**Step-by-Step Breakdown:**

1. The YAML contains a `!!python/object/apply` tag.
2. `safe_load` refuses to resolve it and raises `ConstructorError`.
3. The malicious code is never executed.

**Why the Output is Produced:**

`safe_load` uses `SafeLoader`, which only recognizes standard YAML tags and explicitly rejects Python-specific construction tags.

#### Real-World Cases

- **Application configuration**: Loading `.yaml` settings files without risk.
- **CI/CD inputs**: Parsing user-supplied YAML in pipelines safely.
- **Kubernetes manifest processing**: Reading manifests for validation or transformation.
- **Data ingestion**: Parsing YAML payloads from APIs or uploads.

---

### 4. `yaml.safe_dump()`

#### Definitions

**Core Definition**

`yaml.safe_dump()` serializes standard Python objects into a YAML representation without emitting Python-specific tags.

**Technical Definition**

`yaml.safe_dump(data, stream=None, **kwargs)` serializes `data` using `SafeDumper` and returns a YAML string (or writes to `stream` if provided). `SafeDumper` can represent `dict`, `list`, `tuple`, `str`, `int`, `float`, `bool`, `None`, `bytes` (as `!!binary`), `datetime.date`, and `datetime.datetime`. It does not emit Python-specific tags such as `!!python/object`, ensuring that the output is portable and safe to load back with `safe_load`. Common keyword arguments include `sort_keys` (default `True`), `default_flow_style` (default `False` in modern PyYAML), `indent`, and `allow_unicode`.

**Beginner-Friendly Explanation**

`yaml.safe_dump()` is the safe way to write YAML. You give it a Python dictionary or list, and it gives you back YAML text. It only knows how to write standard Python types, so it cannot accidentally emit dangerous tags. If you are generating YAML for other tools to read, this is the function to use.

#### Purposes

- To serialize Python data to YAML safely and portably.
- To ensure the output can be safely reloaded with `safe_load`.
- To produce human-readable YAML with controlled formatting.
- To support multi-document output via `safe_dump_all()`.
- To avoid emitting Python-specific tags that other languages cannot parse.

#### Syntax Rules and Structure

**Complete General Syntax**

```
yaml.safe_dump(data, stream=None, *, sort_keys=True, default_flow_style=False, indent=2, allow_unicode=True, **kw)
yaml.safe_dump_all(documents, stream=None, **kw)
```

**Component Breakdown**

| Parameter | Description |
|-----------|-------------|
| `data` | The Python object to serialize. |
| `stream` | Optional file-like object to write to. |
| `sort_keys` | Whether to sort dictionary keys (default `True`). |
| `default_flow_style` | Whether to use flow style by default (default `False`). |
| `indent` | Indentation width (default 2). |
| `allow_unicode` | Whether to emit non-ASCII characters directly. |

**Syntax Rules**

- `safe_dump` accepts standard Python types only.
- `sort_keys=False` preserves insertion order.
- `default_flow_style=False` produces block-style YAML (readable).
- `safe_dump_all` accepts an iterable of documents.
- Output is a `str` unless a `stream` is provided.

**Constraints and Limitations**

- Custom Python objects cannot be serialized without a custom `SafeDumper` representer.
- `sort_keys=True` may reorder keys, which can matter for readability or comparison.
- The default output style may vary between PyYAML versions; explicitly setting `default_flow_style` is recommended for reproducibility.
- `safe_dump` does not validate that the object is free of cycles; circular references raise `RepresenterError`.

#### Annotated Code Examples

**Example 1: Safe Dumping with Controlled Formatting**

```python
import yaml

payload = {
    "service": "billing",
    "replicas": 3,
    "features": ["invoices", "refunds"],
}

text = yaml.safe_dump(payload, sort_keys=False, indent=2)
print(text)
```

**Expected Output:**

```
service: billing
replicas: 3
features:
- invoices
- refunds
```

**Step-by-Step Breakdown:**

1. `sort_keys=False` preserves the insertion order.
2. `indent=2` sets the indentation width.
3. The output is block-style YAML.

**Why the Output is Produced:**

`SafeDumper` represents the dictionary as a YAML mapping and the list as a YAML sequence, using standard tags only.

---

**Example 2: Round-Trip Guarantee**

```python
import yaml

data = {"a": [1, 2], "b": {"c": True}}
dumped = yaml.safe_dump(data)
loaded = yaml.safe_load(dumped)
print(loaded == data)
```

**Expected Output:**

```
True
```

**Step-by-Step Breakdown:**

1. `safe_dump` serializes the dictionary to YAML.
2. `safe_load` parses it back.
3. The round-trip equality holds for standard types.

**Why the Output is Produced:**

`safe_dump` and `safe_load` form a complementary pair: the fundamental contract is `yaml.safe_load(yaml.safe_dump(obj)) == obj` for supported types.

#### Real-World Cases

- **Configuration generation**: Writing default or updated config files.
- **Kubernetes manifest generation**: Emitting YAML for `kubectl apply`.
- **Data export**: Converting Python structures to YAML for downstream tools.
- **Test fixtures**: Creating YAML fixtures that are safe to reload.

---

### 5. YAML Best Practices

#### Definitions

**Core Definition**

YAML best practices in Python are guidelines for using PyYAML securely, reliably, and maintainably, with safety as the paramount concern.

**Technical Definition**

Best practices center on the safe/unsafe API distinction and version hygiene. Key rules include: (1) always use `yaml.safe_load()` (or `safe_load_all()`) for any YAML you did not author yourself; (2) never use `yaml.load()` without a safe loader, and never use `FullLoader` or `UnsafeLoader` on untrusted input; (3) run PyYAML **6.0.2 or later**, which carries all loader security fixes and makes the `Loader` argument mandatory for `yaml.load()`; (4) use `CSafeLoader` / `CSafeDumper` for performance when LibYAML is available; (5) subclass `SafeLoader` and register minimal, explicit constructors only when custom tags are truly needed; (6) prefer `safe_dump(..., sort_keys=False)` for readable, order-preserving output; (7) be aware of YAML 1.1 boolean quirks (`yes`/`no`/`on`/`off`) and quote strings that could be misinterpreted.

**Beginner-Friendly Explanation**

Best practices are the safety rules of YAML. The single most important one is: **never use `yaml.load()` on data you don’t trust—use `yaml.safe_load()` instead.** If you need custom types, extend `SafeLoader` carefully rather than switching to the unsafe loader. Keep PyYAML up to date, and use `safe_dump` with `sort_keys=False` to produce clean, readable YAML.

#### Purposes

- To prevent remote code execution and object injection via YAML.
- To ensure YAML output is portable and safe to reload.
- To avoid subtle bugs caused by YAML 1.1 type resolution.
- To maintain performance without sacrificing safety.
- To handle custom types without exposing the full unsafe API.

#### Syntax Rules and Structure

**Complete General Syntax**

```python
import yaml

# Safe defaults.
data = yaml.safe_load(untrusted_yaml)
output = yaml.safe_dump(data, sort_keys=False)

# Custom tags with SafeLoader.
class MyLoader(yaml.SafeLoader):
    pass

MyLoader.add_constructor("!mytag", my_constructor)
data = yaml.load(source, Loader=MyLoader)

# Performance with LibYAML.
try:
    from yaml import CSafeLoader as SafeLoader, CSafeDumper as SafeDumper
except ImportError:
    from yaml import SafeLoader, SafeDumper
```

**Component Breakdown**

| Practice | Description |
|----------|-------------|
| `safe_load` / `safe_dump` | Default choice for ordinary and untrusted YAML. |
| `sort_keys=False` | Preserves key order in output. |
| `CSafeLoader` / `CSafeDumper` | Faster LibYAML-backed safe implementations. |
| `SafeLoader` subclass | Extend with explicit constructors for custom tags. |
| PyYAML ≥ 6.0.2 | Includes all loader security fixes. |

**Syntax Rules**

- Always prefer `safe_load` / `safe_dump` unless you have a specific, justified reason to use the full API.
- Never pass `Loader=yaml.Loader` or `Loader=yaml.UnsafeLoader` on untrusted data.
- If using `yaml.load()` with a custom loader, the loader must inherit from `SafeLoader`.
- Quote strings that resemble YAML 1.1 booleans (`yes`, `no`, `on`, `off`, `true`, `false`) if you intend them as strings.
- Use `sort_keys=False` when key order matters for readability or downstream tools.

**Constraints and Limitations**

- `safe_load` cannot construct custom objects; if you need them, you must extend `SafeLoader`.
- `FullLoader` is not a safe alternative to `SafeLoader`; it has a history of CVEs.
- `CSafeLoader` requires LibYAML to be installed; the fallback pattern handles its absence.
- YAML 1.1 boolean interpretation cannot be changed without custom resolvers.

#### Annotated Code Examples

**Example 1: Safe Loading with LibYAML Acceleration**

```python
import yaml

try:
    from yaml import CSafeLoader as SafeLoader
except ImportError:
    from yaml import SafeLoader

with open("config.yaml", "r", encoding="utf-8") as f:
    config = yaml.load(f, Loader=SafeLoader)

print(config)
```

**Expected Output:**

```
{'debug': True, 'port': 8080}
```

**Step-by-Step Breakdown:**

1. The code tries to import `CSafeLoader` for speed.
2. If LibYAML is unavailable, it falls back to the pure-Python `SafeLoader`.
3. `yaml.load` is called with the explicitly safe loader.

**Why the Output is Produced:**

Both `CSafeLoader` and `SafeLoader` restrict construction to standard Python types, preventing code execution while enabling optional performance acceleration.

---

**Example 2: Custom Tag with SafeLoader**

```python
import os
import yaml

def env_constructor(loader, node):
    return os.environ[loader.construct_scalar(node)]

class EnvLoader(yaml.SafeLoader):
    pass

EnvLoader.add_constructor("!env", env_constructor)

config = yaml.load("token: !env API_TOKEN\n", Loader=EnvLoader)
print(config["token"])
```

**Expected Output:**

```
(Value of the API_TOKEN environment variable)
```

**Step-by-Step Breakdown:**

1. `EnvLoader` inherits from `SafeLoader`.
2. A constructor for `!env` is registered explicitly.
3. The loader resolves only the standard tags plus the explicitly registered `!env` tag.

**Why the Output is Produced:**

Extending `SafeLoader` with explicit constructors provides custom functionality without exposing the arbitrary-object construction capabilities of the unsafe loaders.

#### Real-World Cases

- **Web applications**: Parsing YAML configuration from environment or uploads with `safe_load`.
- **CI/CD pipelines**: Handling user-supplied YAML workflow definitions without risk of code execution.
- **Kubernetes operators**: Reading manifests safely and generating updated YAML with `safe_dump`.
- **Infrastructure automation**: Extending `SafeLoader` for domain-specific tags (e.g., secret references) without using the unsafe API.

---

## References

- PyYAML Documentation – Official documentation for the PyYAML package - https://pyyaml.org/wiki/PyYAMLDocumentation
- PyYAML Python Package Guide – Safe and full API usage, installation, and examples - https://github.com/andrewyng/context-hub/blob/67dcbeb2eb42c808549f08397920ad58be7c2206/content/pyyaml/docs/package/python/DOC.md
- PyYAML Security Guide (2026) – CVEs, unsafe loading, and safe usage rules - https://safeguard.sh/resources/blog/pyyaml-security-guide
- YAML 1.2.2 Specification – Official YAML specification (October 2021) - https://yaml.org/spec/1.2.2/
- YAML Resources – Official YAML website and specification hub - https://yaml.com/
- PyYAML on PyPI – Package distribution and version history - https://pypi.org/project/PyYAML/
- CVE-2017-18342 – PyYAML `yaml.load()` arbitrary code execution - https://nvd.nist.gov/vuln/detail/CVE-2017-18342
- CVE-2020-14343 – PyYAML FullLoader incomplete fix - https://nvd.nist.gov/vuln/detail/CVE-2020-14343
- OWASP – Deserialization of Untrusted Data - https://owasp.org/www-community/vulnerabilities/Deserialization_of_untrusted_data