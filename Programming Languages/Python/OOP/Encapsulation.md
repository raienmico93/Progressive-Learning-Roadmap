# Python Encapsulation: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**

**Encapsulation** is the object-oriented principle that an object should hide its internal implementation details and expose a clean, controlled public interface. In Python, encapsulation is achieved through naming conventions (leading underscores) and managed attributes (properties), rather than through strict access modifiers like `private` or `protected` found in other languages.

**Technical Definition**

Python does not enforce strict privacy through access modifiers. Instead, it relies on a system of **naming conventions** and **name mangling** to signal intent. Names with no leading underscore are **public**; names with a single leading underscore (`_name`) are **non-public** by convention and signal internal implementation details; names with two leading underscores (and at most one trailing underscore, `__name`) undergo **name mangling**, where the interpreter textually rewrites the identifier to `_ClassName__name` to avoid accidental clashes in subclasses. The `@property` decorator provides a mechanism to expose attribute-like access while still running validation, computation, or other logic in **getter**, **setter**, and **deleter** methods. Encapsulation protects object invariants, enables safe evolution of implementation, and defines the public API of a class.

**Beginner-Friendly Explanation**

Encapsulation is about drawing a clear boundary around an object: “here is what you can safely use, and here is what is internal to me.” Python does not lock the door—it just puts a sign on it. A single underscore (`_name`) is a polite “internal, please don’t touch” sign. A double underscore (`__name`) is a stronger sign that also makes the name harder to accidentally collide with in subclasses. When you want to control exactly how an attribute is read or written—for example, to validate a value—you use a **property**, which looks like a normal attribute from the outside but runs your code behind the scenes.

### Key Characteristics

- **Convention over enforcement**: Python signals intent through underscores rather than compiling access restrictions.
- **Name mangling**: Double-leading underscores trigger automatic renaming to `_ClassName__name`, reducing accidental subclass collisions.
- **Properties as managed attributes**: `@property` exposes methods as attributes, enabling validation, computation, and read-only access without changing the public API.
- **Public API stability**: Encapsulation allows internal implementation to change without breaking external code.
- **Invariant protection**: Setters can reject invalid values, keeping objects in a valid state.
- **Module-level encapsulation**: The same underscore conventions apply to modules, with `__all__` controlling wildcard imports.

### Prerequisites

- Basic Python syntax: classes, objects, attributes, and methods.
- Understanding of the `self` parameter and instance state.
- Familiarity with decorators (for `@property`).
- Awareness of inheritance and subclassing (for name mangling).

### Related Programming Areas

- **Properties and descriptors**: The `property` class and the descriptor protocol underpin managed attributes.
- **Inheritance**: Name mangling is specifically designed to avoid attribute clashes in subclasses.
- **Data classes**: `@dataclass` generates `__init__` and related methods; properties can complement them.
- **API design**: Encapsulation defines what is public, private, and stable.
- **Validation frameworks**: Pydantic and similar libraries enforce schema validation, but properties offer a lightweight built-in alternative.

---

## Core Concepts / Features

---

### 1. Encapsulation

#### Definitions

**Core Definition**

Encapsulation is the practice of bundling data and the methods that operate on that data within a single unit (a class) while restricting direct external access to the internal state.

**Technical Definition**

Encapsulation in Python is achieved through **conventions and protocols**, not access modifiers. The language defines three categories of names: **public** (no leading underscore), **non-public** (single leading underscore, `_name`), and **class-private** (double leading underscore, `__name`, subject to name mangling). Properties (`@property`) allow a class to present attribute-like access while executing getter, setter, or deleter logic. The `__slots__` attribute can further restrict which attributes an instance may have, providing both encapsulation and memory optimization. Encapsulation is not about security; it is about **interface design** and **maintainability**.

**Beginner-Friendly Explanation**

Encapsulation is like a TV remote: you press “volume up” without needing to know how the electronics inside work. The remote’s buttons are the **public interface**; the circuit board is the **internal implementation**. In Python, you use underscores to label the circuit board as “internal” and properties to build the buttons that safely control it.

#### Purposes

- To protect an object’s internal state from unintended external modification.
- To define a clear, stable public API that can remain unchanged even as internals evolve.
- To enforce invariants by validating values before they are stored.
- To reduce coupling between a class and the code that uses it.
- To signal intent to other developers about which names are safe to use.

#### Syntax Rules and Structure

**Complete General Syntax**

```python
class MyClass:
    def __init__(self):
        self.public = "anyone can use this"
        self._internal = "please treat as internal"
        self.__private = "name-mangled to avoid subclass clashes"

    @property
    def managed(self):
        return self._internal
```

**Component Breakdown**

| Name Form | Convention | Behavior |
|-----------|------------|----------|
| `name` | Public | No special treatment; part of the API. |
| `_name` | Non-public | Signals internal use; still accessible but discouraged. |
| `__name` | Class-private | Name-mangled to `_ClassName__name`. |
| `__name__` | Dunder | Reserved for Python’s special methods and attributes. |

**Syntax Rules**

- A single leading underscore marks a name as non-public by convention.
- Two leading underscores (and at most one trailing underscore) trigger name mangling.
- Names with double leading and trailing underscores are reserved for Python’s use.
- Properties are defined with `@property` and optional `.setter` and `.deleter` decorators.

**Constraints and Limitations**

- Python does not enforce privacy; any “private” attribute can still be accessed with effort.
- Name mangling is textual and class-specific; it does not create true security.
- Overusing double underscores harms readability and testability; the Google Python Style Guide recommends preferring a single underscore.

#### Annotated Code Examples

**Example 1: Public, Non-Public, and Mangled Names**

```python
class BankAccount:
    def __init__(self, balance):
        self.balance = balance          # public
        self._history = []              # non-public by convention
        self.__pin = "1234"             # name-mangled

    def deposit(self, amount):
        self.balance += amount
        self._history.append(amount)

acct = BankAccount(100)
acct.deposit(50)
print(acct.balance)        # 150
print(acct._history)       # [50]
print(acct._BankAccount__pin)  # 1234 (mangled access)
```

**Expected Output:**

```
150
[50]
1234
```

**Step-by-Step Breakdown:**

1. `balance` is public and part of the API.
2. `_history` is internal by convention.
3. `__pin` is name-mangled to `_BankAccount__pin`.
4. Accessing the mangled name reveals the value, demonstrating that mangling is not encryption.

**Why the Output is Produced:**

Name mangling rewrites `__pin` to `_BankAccount__pin`, but the value remains accessible if the mangled name is known.

#### Real-World Cases

- **Library APIs**: Public functions and classes are documented; underscore-prefixed helpers are internal.
- **Framework base classes**: Internal attributes use single underscores to signal “do not override.”
- **Security-sensitive fields**: Double underscores reduce accidental exposure in subclasses, though they are not a security mechanism.

---

### 2. Public Attributes

#### Definitions

**Core Definition**

A **public attribute** is an attribute with no leading underscore. It is part of the class’s public API and is intended to be safely read and modified by external code.

**Technical Definition**

Public attributes are ordinary instance or class attributes whose names do not begin with an underscore. They are documented as part of the class’s interface and are considered stable. Python does not restrict their access. The Google Python Style Guide recommends exposing simple public attributes directly rather than creating accessor methods for them, and using properties only when additional logic is needed.

**Beginner-Friendly Explanation**

A public attribute is like the front door of a house—everyone is welcome to use it. It is the official way to interact with the object.

#### Purposes

- To expose data that is safe for external code to read or modify.
- To form the stable, documented interface of a class.
- To keep simple attributes simple, avoiding unnecessary boilerplate.
- To allow direct access when no validation or computation is required.
- To serve as the contract that other code can rely on.

#### Syntax Rules and Structure

**Complete General Syntax**

```python
class Config:
    def __init__(self, timeout):
        self.timeout = timeout  # public attribute
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `self.timeout` | A public instance attribute. |
| `Config.timeout` | If defined in the class body, a public class attribute. |

**Syntax Rules**

- Public attributes have no leading underscore.
- They can be read and written directly: `obj.timeout = 30`.
- They are documented as part of the API.

**Constraints and Limitations**

- Public mutable attributes can be changed to invalid values unless validation is added via a property.
- Changing a public attribute’s semantics later is a breaking change.

#### Annotated Code Examples

**Example 1: Public Configuration Attribute**

```python
class Config:
    def __init__(self, timeout):
        self.timeout = timeout

config = Config(30)
config.timeout = 60
print(config.timeout)
```

**Expected Output:**

```
60
```

**Step-by-Step Breakdown:**

1. `timeout` is public.
2. External code reads and writes it directly.

**Why the Output is Produced:**

Public attributes are part of the object’s API and are freely accessible.

#### Real-World Cases

- **Configuration objects**: Public attributes for settings that are safe to change.
- **DTOs (Data Transfer Objects)**: Simple containers with public fields.
- **Named tuples and dataclasses**: Fields are public by design.

---

### 3. Protected Attributes and `_single_underscore`

#### Definitions

**Core Definition**

A **protected** or **non-public** attribute is an attribute whose name begins with a single underscore (`_name`). It signals that the attribute is an internal implementation detail and should not be relied upon by external code.

**Technical Definition**

A single leading underscore is a **convention**, not an enforcement mechanism. It tells other developers: “You can access this, but you probably shouldn’t; it is not part of the stable API and may change without notice.” The Python documentation describes such names as “non-public” and notes that they are still accessible. This convention applies to attributes, methods, and module-level names.

**Beginner-Friendly Explanation**

A single underscore is a polite “internal, please don’t touch” sign. Python won’t stop you from opening the door, but the sign tells you that the room behind it is for the class’s own use.

#### Purposes

- To mark internal implementation details without hiding them completely.
- To signal that a name is not part of the public API.
- To allow subclasses to access internal helpers when necessary.
- To avoid name mangling when subclass cooperation is needed.
- To keep the public API small and focused.

#### Syntax Rules and Structure

**Complete General Syntax**

```python
class MyClass:
    def __init__(self):
        self._internal = "non-public"

    def _helper(self):
        """Internal helper method."""
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `_attribute` | Non-public instance or class attribute. |
| `_method()` | Non-public method. |
| `_module_name` | Non-public module-level name. |

**Syntax Rules**

- A single leading underscore is used.
- The name remains fully accessible from outside the class.
- Subclasses can access and override non-public names.
- Linters and IDEs often warn when non-public names are accessed externally.

**Constraints and Limitations**

- The convention is not enforced; external code can ignore it.
- Overuse of non-public attributes can make testing and debugging harder.
- Non-public names are not imported by `from module import *` unless listed in `__all__`.

#### Annotated Code Examples

**Example 1: Non-Public Helper**

```python
class Temperature:
    def __init__(self, celsius):
        self._celsius = celsius  # non-public

    def to_fahrenheit(self):
        return self._celsius * 9 / 5 + 32

t = Temperature(20)
print(t.to_fahrenheit())
print(t._celsius)  # accessible, but discouraged
```

**Expected Output:**

```
68.0
20
```

**Step-by-Step Breakdown:**

1. `_celsius` is marked non-public by convention.
2. `to_fahrenheit` is the public API.
3. External access to `_celsius` works but violates the convention.

**Why the Output is Produced:**

The underscore does not prevent access; it only signals intent.

#### Real-World Cases

- **Library internals**: Helper functions and attributes prefixed with `_`.
- **Template methods**: Non-public hooks that subclasses may override.
- **Caching**: Internal caches stored in `_cache`.

---

### 4. Private Attributes, `__double_underscore`, and Name Mangling

#### Definitions

**Core Definition**

A **class-private** attribute is an attribute whose name begins with two underscores (and at most one trailing underscore). Python **mangles** the name to `_ClassName__name`, reducing accidental access and subclass collisions.

**Technical Definition**

When an identifier textually occurring in a class definition begins with two or more underscores and does not end in two or more underscores, it is **textually replaced** with `_ClassName__name`, where `ClassName` is the current class name with leading underscores stripped. This transformation occurs at compile time and is independent of syntactic context. Name mangling is designed to let subclasses define attributes with the same name without clobbering the base class’s attribute. It is **not** a security mechanism; the mangled name is predictable and accessible.

**Beginner-Friendly Explanation**

Double underscores are a stronger “keep out” sign: Python automatically renames the attribute to include the class name, so if a subclass accidentally uses the same name, it gets a different mangled name and does not overwrite the parent’s attribute. It is like putting your name on your lunch in the office fridge—someone else with the same lunch box won’t take yours by mistake.

#### Purposes

- To avoid accidental attribute name collisions between a base class and its subclasses.
- To make attributes harder (but not impossible) to access from outside the class.
- To support framework design where base-class internals should not be overridden accidentally.
- To signal that an attribute is strictly internal to the defining class.
- To provide a stronger convention than a single underscore when subclass clashes are a concern.

#### Syntax Rules and Structure

**Complete General Syntax**

```python
class MyClass:
    def __init__(self):
        self.__secret = "mangled"

# Actual attribute name is _MyClass__secret
```

**Component Breakdown**

| Original Name | Mangled Name |
|---------------|--------------|
| `__secret` in class `MyClass` | `_MyClass__secret` |
| `__method` in class `MyClass` | `_MyClass__method` |

**Syntax Rules**

- Two leading underscores and at most one trailing underscore trigger mangling.
- The class name has leading underscores stripped and a single underscore prepended.
- Mangling applies to identifiers textually inside the class definition.
- Names ending with two or more underscores (dunders) are not mangled.

**Constraints and Limitations**

- Mangling is not encryption; the mangled name is predictable.
- Subclasses cannot easily access the base class’s private attributes without knowing the mangled name.
- Overuse harms readability and testability; the Google Python Style Guide discourages it.

#### Annotated Code Examples

**Example 1: Name Mangling in Subclasses**

```python
class Mapping:
    def __init__(self):
        self.__update("initial")  # mangled to _Mapping__update

    def __update(self, value):
        print(f"Mapping update: {value}")

class MappingSubclass(Mapping):
    def __init__(self):
        super().__init__()
        self.__update("subclass")  # mangled to _MappingSubclass__update

    def __update(self, value):
        print(f"Subclass update: {value}")

obj = MappingSubclass()
```

**Expected Output:**

```
Mapping update: initial
Subclass update: subclass
```

**Step-by-Step Breakdown:**

1. In `Mapping`, `__update` becomes `_Mapping__update`.
2. In `MappingSubclass`, `__update` becomes `_MappingSubclass__update`.
3. The two methods do not collide, so the base class’s internal call uses its own method, and the subclass’s call uses its own.

**Why the Output is Produced:**

Name mangling ensures that a subclass’s `__update` does not accidentally override the base class’s `__update`.

#### Real-World Cases

- **Framework base classes**: Preventing subclasses from accidentally overriding internal hooks.
- **Attribute collision avoidance**: Ensuring base-class state remains intact in deep hierarchies.
- **Legacy code**: Understanding mangled names when debugging or introspecting objects.

---

### 5. Properties: `@property`, Getters, Setters, and Deleters

#### Definitions

**Core Definition**

A **property** is a managed attribute that exposes method-based logic through attribute-like access. The `@property` decorator defines a **getter**; the `@name.setter` decorator defines a **setter**; the `@name.deleter` decorator defines a **deleter**.

**Technical Definition**

The `property` class implements the descriptor protocol, allowing a method to be accessed like an attribute. The `@property` decorator turns a method into a getter. `@name.setter` registers a setter that runs on assignment, and `@name.deleter` registers a deleter that runs on `del`. Properties enable validation, computed attributes, lazy evaluation, and read-only attributes without changing the public interface. They are the primary tool for encapsulation in Python.

**Beginner-Friendly Explanation**

A property is a “smart attribute.” It looks like a normal attribute from the outside (`obj.value`), but behind the scenes it runs a method. You use a property when you want to **check** a value before storing it, **compute** a value on the fly, or make an attribute **read-only**.

#### Purposes

- To validate data before assigning it to an attribute.
- To compute attribute values dynamically from other state.
- To create read-only attributes by omitting a setter.
- To add side effects when an attribute is accessed or modified.
- To change the internal implementation of an attribute without breaking the public API.

#### Syntax Rules and Structure

**Complete General Syntax**

```python
class MyClass:
    @property
    def name(self):
        return self._name

    @name.setter
    def name(self, value):
        self._name = value

    @name.deleter
    def name(self):
        del self._name
```

**Component Breakdown**

| Decorator | Purpose |
|-----------|---------|
| `@property` | Defines the getter. |
| `@name.setter` | Defines the setter (runs on assignment). |
| `@name.deleter` | Defines the deleter (runs on `del`). |

**Syntax Rules**

- The getter is defined first with `@property`.
- The setter uses `@<property_name>.setter`.
- The deleter uses `@<property_name>.deleter`.
- The underlying storage is typically a non-public attribute (e.g., `_name`).
- Omitting the setter makes the property read-only.

**Constraints and Limitations**

- Properties add overhead compared with direct attribute access; avoid them in performance-critical inner loops if direct access is sufficient.
- Inheritance with properties can be non-obvious if the property itself is not overridden.
- A property cannot be used to store arbitrary data without an underlying attribute.

#### Annotated Code Examples

**Example 1: Getter, Setter, and Deleter**

```python
class Temperature:
    def __init__(self, celsius):
        self._celsius = celsius

    @property
    def celsius(self):
        """Get the temperature in Celsius."""
        return self._celsius

    @celsius.setter
    def celsius(self, value):
        if value < -273.15:
            raise ValueError("Temperature below absolute zero!")
        self._celsius = value

    @celsius.deleter
    def celsius(self):
        print("Deleting temperature.")
        del self._celsius

t = Temperature(25)
print(t.celsius)
t.celsius = 30
print(t.celsius)
del t.celsius
```

**Expected Output:**

```
25
30
Deleting temperature.
```

**Step-by-Step Breakdown:**

1. `t.celsius` calls the getter and returns `_celsius`.
2. `t.celsius = 30` calls the setter, which validates and stores the value.
3. `del t.celsius` calls the deleter, which prints and deletes the attribute.

**Why the Output is Produced:**

Properties intercept attribute access, assignment, and deletion, routing them to the corresponding methods.

#### Real-World Cases

- **Validation**: Ensuring an age is non-negative or an email contains `@`.
- **Computed values**: `area` on a rectangle derived from `width` and `height`.
- **Read-only fields**: `id` or `created_at` that should not be changed.
- **Lazy loading**: Computing an expensive value only on first access.

---

### 6. Read-Only Properties

#### Definitions

**Core Definition**

A **read-only property** is a property with a getter but **no setter**. Attempting to assign to it raises `AttributeError`.

**Technical Definition**

A property object created with only `@property` and no `@name.setter` has no setter function. When the interpreter attempts to set the attribute, the property’s `__set__` method raises `AttributeError` because `fset` is `None`. Read-only properties are the idiomatic way to expose computed or immutable attributes without allowing external modification.

**Beginner-Friendly Explanation**

A read-only property is a window, not a door: you can look through it, but you cannot climb in. You can read the value, but trying to change it raises an error.

#### Purposes

- To expose computed values that should not be assigned directly.
- To protect attributes that are derived from other state.
- To provide immutable fields on otherwise mutable objects.
- To enforce the invariant that certain values are controlled by the class.
- To signal that an attribute is informational rather than configurable.

#### Syntax Rules and Structure

**Complete General Syntax**

```python
class MyClass:
    @property
    def read_only(self):
        return self._computed_value
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `@property` | Defines the getter; no setter is registered. |
| `self._computed_value` | The underlying data or computation. |

**Syntax Rules**

- Define the getter with `@property` and omit the setter entirely.
- Assignment attempts raise `AttributeError`.
- The property can still be deleted if a deleter is defined, but typically it is not.

**Constraints and Limitations**

- Read-only properties can still be overridden in subclasses unless the property itself is overridden.
- They do not make the underlying storage immutable; internal methods can still mutate it.

#### Annotated Code Examples

**Example 1: Computed Read-Only Property**

```python
class Rectangle:
    def __init__(self, width, height):
        self.width = width
        self.height = height

    @property
    def area(self):
        return self.width * self.height

r = Rectangle(3, 4)
print(r.area)
try:
    r.area = 20
except AttributeError as e:
    print("Cannot set area:", e)
```

**Expected Output:**

```
12
Cannot set area: can't set attribute
```

**Step-by-Step Breakdown:**

1. `area` is a computed property with only a getter.
2. Reading `r.area` computes `3 * 4 = 12`.
3. Assigning `r.area = 20` raises `AttributeError`.

**Why the Output is Produced:**

The property has no setter, so the interpreter refuses assignment.

#### Real-World Cases

- **Derived metrics**: `area`, `perimeter`, `full_name` computed from stored fields.
- **Immutable identifiers**: `id` or `uuid` that should not change.
- **Status flags**: `is_active` derived from other state.

---

### 7. Computed Properties

#### Definitions

**Core Definition**

A **computed property** calculates its value dynamically from other attributes each time it is accessed, rather than storing a separate value.

**Technical Definition**

A computed property is a property whose getter derives its return value from existing state. It is typically read-only because assigning to it would conflict with the derived nature of the value. Computed properties avoid data duplication and ensure consistency: the computed value always reflects the current state of the underlying attributes.

**Beginner-Friendly Explanation**

A computed property is like a calculator built into the object: you ask for `area`, and it multiplies `width` by `height` on the spot. You never store `area` separately, so it can never become out of sync.

#### Purposes

- To derive values from other attributes without storing redundant data.
- To keep related attributes consistent automatically.
- To provide convenient, readable access to derived quantities.
- To avoid the overhead and risk of maintaining duplicate state.
- To support read-only views of changing underlying data.

#### Syntax Rules and Structure

**Complete General Syntax**

```python
class MyClass:
    @property
    def computed(self):
        return self._base * 2
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `@property` | Defines the computed getter. |
| Return expression | Calculates the value from current state. |

**Syntax Rules**

- The getter performs the computation each time it is called.
- No setter is typically defined.
- The computation should be pure (no side effects) and reasonably fast.

**Constraints and Limitations**

- Computed properties are recalculated on every access; expensive computations should be cached or converted to methods.
- If the underlying state changes, the computed value changes automatically.
- Computed properties cannot be assigned to unless a setter is explicitly defined.

#### Annotated Code Examples

**Example 1: Computed `area` and `diameter`**

```python
import math

class Circle:
    def __init__(self, radius):
        self.radius = radius

    @property
    def area(self):
        return math.pi * self.radius ** 2

    @property
    def diameter(self):
        return self.radius * 2

c = Circle(5)
print(c.area)
print(c.diameter)
c.radius = 10
print(c.area)
```

**Expected Output:**

```
78.53981633974483
10
314.1592653589793
```

**Step-by-Step Breakdown:**

1. `area` and `diameter` are computed from `radius`.
2. Changing `radius` automatically changes the computed values.
3. No redundant storage is needed.

**Why the Output is Produced:**

The getters compute fresh values each time, reflecting the current `radius`.

#### Real-World Cases

- **Geometric shapes**: `area`, `perimeter`, `volume`.
- **Financial models**: `total` computed from line items.
- **User profiles**: `full_name` computed from `first_name` and `last_name`.
- **Health metrics**: `bmi` computed from weight and height.

---

### 8. Encapsulation Best Practices

#### Definitions

**Core Definition**

Encapsulation best practices are guidelines for using underscores and properties effectively to create clear, maintainable, and safe class interfaces without over-engineering.

**Technical Definition**

Best practices include: using a **single leading underscore** for internal attributes and methods; using **double underscores sparingly** and only to avoid subclass collisions; preferring **properties** over explicit getter/setter methods when validation or computation is needed; keeping simple public attributes public; using `__slots__` when attribute restriction or memory optimization is required; and documenting the public API. The Google Python Style Guide recommends exposing simple public attributes directly and using properties for managed access. Overusing double underscores is discouraged because it harms readability and testability.

**Beginner-Friendly Explanation**

Best practices are the “good habits” of encapsulation. Use a single underscore for internal things. Use double underscores only when you are building a class that will be subclassed and you want to avoid name clashes. Use properties when you need to check or compute values; otherwise, keep attributes public and simple. Don’t hide everything just for the sake of hiding it.

#### Purposes

- To write classes that are easy to understand, test, and maintain.
- To avoid the pitfalls of over-encapsulation and unnecessary complexity.
- To protect invariants without sacrificing usability.
- To keep the public API minimal, stable, and well-documented.
- To use the right tool (underscore, property, or `__slots__`) for each situation.

#### Syntax Rules and Structure

**Complete General Syntax**

```python
class WellDesigned:
    __slots__ = ("_value",)

    def __init__(self, value):
        self._value = value

    @property
    def value(self):
        return self._value

    @value.setter
    def value(self, new_value):
        if new_value < 0:
            raise ValueError("Must be non-negative")
        self._value = new_value
```

**Component Breakdown**

| Practice | Recommendation |
|----------|----------------|
| Internal attributes | Use `_name`. |
| Subclass collision avoidance | Use `__name` sparingly. |
| Validation or computation | Use `@property`. |
| Simple data | Keep public; avoid unnecessary accessors. |
| Attribute restriction | Use `__slots__` when needed. |
| Module internals | Use `_name` and `__all__`. |

**Syntax Rules**

- Prefer `_name` for internal use.
- Use `__name` only when subclass collisions are a real risk.
- Define properties only when logic is required.
- Document public attributes and methods.
- Use `__slots__` to prevent dynamic attribute creation and save memory.

**Constraints and Limitations**

- Best practices are guidelines; context matters.
- Overusing properties for trivial attributes adds unnecessary overhead.
- `__slots__` complicates inheritance and dynamic attribute use.

#### Annotated Code Examples

**Example 1: A Balanced Encapsulation Design**

```python
class Product:
    __slots__ = ("name", "_price")

    def __init__(self, name, price):
        self.name = name          # public: safe to read/write
        self._price = price       # non-public storage

    @property
    def price(self):
        return self._price

    @price.setter
    def price(self, value):
        if value < 0:
            raise ValueError("Price cannot be negative")
        self._price = value
```

**Expected Output:**

```
(No output; class definition.)
```

**Step-by-Step Breakdown:**

1. `name` is public because it is simple and safe.
2. `_price` is non-public storage.
3. `price` is a property that validates before storing.
4. `__slots__` restricts attributes to `name` and `_price`.

**Why the Output is Produced:**

This design exposes a clean public API (`name` and `price`) while protecting the price invariant and preventing arbitrary attribute creation.

#### Real-World Cases

- **Library design**: Public API documented; internals prefixed with `_`.
- **Data validation**: Properties enforce domain rules.
- **Memory-sensitive applications**: `__slots__` reduces per-instance overhead.
- **Framework base classes**: Double underscores prevent accidental overrides.

---

## References

- Python Documentation – Private Variables (Name Mangling) - https://docs.python.org/3/tutorial/classes.html#private-variables
- Python Documentation – `property()` Built-in Function - https://docs.python.org/3/library/functions.html#property
- Python Documentation – Reserved Classes of Identifiers (Lexical Analysis) - https://docs.python.org/3/reference/lexical_analysis.html#reserved-classes-of-identifiers
- Real Python – Single and Double Underscores in Python Names - https://realpython.com/python-double-underscore/
- Real Python – Python's `property()`: Add Managed Attributes to Your Classes - https://realpython.com/python-property/
- Real Python – `@property` in Python: Getters, Setters, and Computed Attributes Done Right - https://prod.codegym.cc/groups/posts/python-property-getters-setters-computed-attributes
- Real Python – Exercise: Compute Attributes on the Fly With Properties - https://realpython.com/python-property/
- Google Python Style Guide – Properties - https://google.github.io/styleguide/pyguide.html#properties
- wolfcodes.dev – Encapsulation (OOP Guide) - https://wolfcodes.dev
- Stack Overflow – Private Name Mangling Rules - https://stackoverflow.com/revisions/3740a6b3-2406-490f-aa61-6b9af6e831ef/view-source
- Python-for-Security-Professionals – Property Decorators - https://github.com/armourinfosec/Python-for-Security-Professionals