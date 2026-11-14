# Python Polymorphism: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**

Polymorphism is a fundamental principle of object-oriented programming that allows a single interface—a method name, function, or operator—to represent different behaviors depending on the type of object it operates on. It shifts the focus from what an object **is** to what an object **can do** .

**Technical Definition**

In Python, polymorphism is the property by which objects of different classes can be used interchangeably through a shared interface. Python achieves polymorphism primarily through **duck typing**—a structural type system where an object’s compatibility is determined by the methods and attributes it exposes at runtime, not by its declared type or inheritance hierarchy . Polymorphism is also realized through **inheritance and method overriding** (where a subclass replaces a method from its superclass) and through **operator overloading** (where special methods define how built-in operators behave for user-defined types) . The result is code that is flexible, extensible, and decoupled from specific implementations.

**Beginner-Friendly Explanation**

Polymorphism is the “many forms” principle. It means you can write code that says “make this object speak” without caring whether the object is a dog, a cat, or a parrot. As long as the object knows how to speak, your code works. In Python, this is often described with the saying: “If it walks like a duck and quacks like a duck, then it must be a duck.” You do not need to prove an object is a duck—you just call its `quack()` method, and if it works, it is a duck for your purposes .

### Key Characteristics

- **Interface over type**: Polymorphism focuses on what an object can do, not what class it belongs to .
- **Runtime resolution**: Python resolves method calls at runtime, enabling flexible, behavior-based dispatch .
- **Multiple mechanisms**: Polymorphism is achieved through duck typing, inheritance-based overriding, method overloading simulations, and operator overloading .
- **Liskov Substitution Principle**: Objects of a superclass should be replaceable with objects of subclasses without breaking the program—a formal guarantee that polymorphism depends on .
- **Extensibility without modification**: New classes can participate in existing polymorphic code without changing that code, as long as they implement the expected interface.

### Prerequisites

- Basic Python syntax: classes, objects, methods, and inheritance.
- Understanding of the `self` parameter and attribute lookup.
- Familiarity with special (dunder) methods.
- Awareness of Python’s dynamic typing model.

### Related Programming Areas

- **Object-oriented design**: Polymorphism is one of the four pillars of OOP (alongside encapsulation, inheritance, and abstraction).
- **Duck typing and protocols**: Structural subtyping via `typing.Protocol` (PEP 544).
- **Operator overloading**: Customizing built-in operators for user-defined types.
- **Functional programming**: Generic functions and `functools.singledispatch`.
- **Design patterns**: Strategy, template method, and factory patterns rely on polymorphic behavior.

---

## Core Concepts / Features

---

### 1. Polymorphism

#### Definitions

**Core Definition**

Polymorphism is the ability of a single interface to represent different underlying types, allowing objects of different classes to be used interchangeably through a shared set of behaviors.

**Technical Definition**

Polymorphism is a type system property where a value can belong to more than one type, and operations on that value can behave differently depending on its actual type. In Python’s dynamic type system, polymorphism is primarily **structural**: an object is compatible with an expected type if it provides the required methods and attributes, regardless of its class hierarchy. This is formalized by the Liskov Substitution Principle (LSP): if `S` is a subtype of `T`, then objects of type `T` may be replaced with objects of type `S` without altering the correctness of the program. Python’s polymorphism is realized through duck typing, inheritance-based overriding, and operator overloading .

**Beginner-Friendly Explanation**

Polymorphism means “one name, many behaviors.” If you have a function called `area()`, it can work for a rectangle, a circle, or a triangle—as long as each object knows how to calculate its own area. You write the function once, and it works with any shape that supports the `area()` method. This makes your code flexible and reusable .

#### Purposes

- To write generic, reusable code that operates on abstractions rather than concrete types.
- To allow new classes to integrate with existing code without modification.
- To decouple function implementations from the specific types they operate on.
- To enable flexible and extensible designs through shared interfaces.
- To support the Liskov Substitution Principle, ensuring subclasses can safely replace their base classes.

#### Syntax Rules and Structure

**Complete General Syntax**

Polymorphism is not a single syntax but a design pattern realized through various mechanisms:

```python
# Duck typing: any object with the required method works.
def process(obj):
    return obj.do_something()

# Inheritance and overriding: subclasses replace base methods.
class Base:
    def do_something(self):
        return "base"

class Derived(Base):
    def do_something(self):
        return "derived"

# Operator overloading: special methods define operator behavior.
class Vector:
    def __init__(self, x, y):
        self.x, self.y = x, y
    def __add__(self, other):
        return Vector(self.x + other.x, self.y + other.y)
```

**Component Breakdown**

| Mechanism | Description |
|-----------|-------------|
| Duck typing | Objects are interchangeable if they provide the required methods/attributes. |
| Method overriding | Subclasses replace inherited methods with their own implementations. |
| Operator overloading | Special methods (`__add__`, `__eq__`, etc.) define operator behavior. |
| Method overloading (simulated) | Using defaults, `*args`, or `singledispatch` to handle varied inputs. |

**Syntax Rules**

- Polymorphism relies on a **shared interface**: the set of method names and signatures that calling code expects.
- In Python, the interface is implicit (duck typing) unless formalized with ABCs or Protocols.
- Method resolution occurs at runtime based on the object’s type.
- Polymorphic code should depend only on the interface, not on concrete types.

**Constraints and Limitations**

- Duck typing shifts error detection to runtime; missing methods cause `AttributeError`.
- Polymorphism without a formal contract can lead to subtle bugs when objects accidentally satisfy the interface incorrectly.
- Overusing `isinstance()` checks defeats the purpose of polymorphism.
- The Liskov Substitution Principle can be violated if a subclass narrows a contract or raises unexpected exceptions.

#### Annotated Code Examples

**Example 1: Polymorphism Through Duck Typing**

```python
class Duck:
    def swim(self):
        print("The duck is swimming.")

class Albatross:
    def swim(self):
        print("The albatross is swimming.")

# No common base class required.
birds = [Duck(), Albatross()]

for bird in birds:
    bird.swim()
```

**Expected Output:**

```
The duck is swimming.
The albatross is swimming.
```

**Step-by-Step Breakdown:**

1. `Duck` and `Albatross` do not share a base class.
2. Both define a `swim()` method.
3. The loop calls `bird.swim()` on each object, and Python resolves the method at runtime.

**Why the Output is Produced:**

Python does not check the type of `bird`; it simply calls `swim()`. As long as the method exists, the call succeeds. This is duck typing in action .

---

**Example 2: Polymorphism Through Inheritance and Overriding**

```python
class Animal:
    def speak(self):
        return "Animal speaks"

class Dog(Animal):
    def speak(self):
        return "Woof, Woof"

class Cat(Animal):
    def speak(self):
        return "Meow, Meow"

def make_animal_speak(animal):
    print(animal.speak())

make_animal_speak(Dog())
make_animal_speak(Cat())
```

**Expected Output:**

```
Woof, Woof
Meow, Meow
```

**Step-by-Step Breakdown:**

1. `Dog` and `Cat` inherit from `Animal`.
2. Both override `speak()`.
3. `make_animal_speak()` accepts any `Animal` and calls `speak()`.
4. The actual method that runs depends on the object’s type.

**Why the Output is Produced:**

The function is written against the `Animal` interface, but the overridden methods in the subclasses provide specialized behavior .

#### Real-World Cases

- **File-like objects**: Functions that accept any object with `read()` or `write()` work with files, `StringIO`, sockets, and compressed streams.
- **Iteration**: Any object implementing `__iter__()` can be used in a `for` loop.
- **Serialization**: Libraries that call `__getstate__()` or `to_dict()` work with any object that provides them.
- **Web frameworks**: Plugins and middleware are interchangeable if they implement the expected hooks.

---

### 2. Method Overriding

#### Definitions

**Core Definition**

Method overriding is the mechanism by which a subclass provides a new implementation of a method that is already defined in its superclass, allowing the subclass to specialize or replace inherited behavior.

**Technical Definition**

When a subclass defines a method with the same name as a method in its superclass, the subclass’s method **overrides** the superclass’s method for instances of the subclass. The superclass method remains accessible via `super()`. Overriding is the inheritance-based mechanism for polymorphism: calling code that depends on the superclass interface automatically invokes the subclass’s specialized implementation when given a subclass instance. The override can fully replace the parent’s behavior or extend it by calling `super().method()` .

**Beginner-Friendly Explanation**

Method overriding is like a child inheriting a recipe from a parent but changing one step. The parent’s recipe (the base method) still exists, but when you ask the child to cook (call the method on a child object), the child uses its modified step. You can even have the child call the parent’s version for the parts that do not need changing.

#### Purposes

- To specialize inherited behavior for a subclass.
- To provide polymorphic dispatch through a shared base-class interface.
- To extend rather than replace a parent method by calling `super()`.
- To implement the template method pattern, where a base class defines the algorithm skeleton and subclasses fill in specific steps.
- To support the Liskov Substitution Principle by keeping the interface consistent while changing behavior.

#### Syntax Rules and Structure

**Complete General Syntax**

```python
class Base:
    def method(self, args):
        # base implementation

class Derived(Base):
    def method(self, args):
        # specialized implementation
        # optionally: super().method(args)
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| Same method name | The subclass method must have the same name as the base method. |
| Compatible signature | The override should accept the same parameters (or be more permissive). |
| `super()` | Optional call to the base implementation for extension. |

**Syntax Rules**

- The overriding method must have the same name as the overridden method.
- The signature should be compatible; violating this can break polymorphic dispatch.
- `super().method()` calls the next implementation in the MRO.
- The base method remains accessible on the base class itself.

**Constraints and Limitations**

- Narrowing the contract (e.g., accepting fewer arguments) violates the Liskov Substitution Principle.
- Overriding without calling `super()` can break base-class invariants.
- Deeply nested overrides can make behavior hard to trace.

#### Annotated Code Examples

**Example 1: Full Replacement**

```python
class Logger:
    def log(self, message):
        print(f"[LOG] {message}")

class ErrorLogger(Logger):
    def log(self, message):
        print(f"[ERROR] {message}")

logger = ErrorLogger()
logger.log("disk full")
```

**Expected Output:**

```
[ERROR] disk full
```

**Step-by-Step Breakdown:**

1. `Logger` defines `log()`.
2. `ErrorLogger` overrides `log()` with a different prefix.
3. Calling `log()` on an `ErrorLogger` instance uses the overridden method.

**Why the Output is Produced:**

The subclass’s method shadows the superclass’s method for instances of the subclass.

---

**Example 2: Extension with `super()`**

```python
class Logger:
    def log(self, message):
        print(f"[LOG] {message}")

class TimestampLogger(Logger):
    def log(self, message):
        from datetime import datetime
        super().log(f"{datetime.now().isoformat()} - {message}")

logger = TimestampLogger()
logger.log("server started")
```

**Expected Output:**

```
[LOG] 2026-11-14T12:00:00.000000 - server started
```

**Step-by-Step Breakdown:**

1. `TimestampLogger` overrides `log()`.
2. It calls `super().log()` to reuse the parent’s formatting.
3. The timestamp is added before delegating to the parent.

**Why the Output is Produced:**

`super()` delegates to the next method in the MRO, preserving the parent’s behavior while adding new functionality .

#### Real-World Cases

- **Template method pattern**: Base classes define the algorithm; subclasses override specific steps.
- **Framework hooks**: Django views override `get()` or `post()`; pytest plugins override hooks.
- **Exception hierarchies**: Subclasses override `__str__()` to provide custom error messages.
- **Custom containers**: Subclasses override `__getitem__()` or `__setitem__()` to provide specialized storage.

---

### 3. Method Overloading

#### Definitions

**Core Definition**

Method overloading is the ability to define multiple methods with the same name but different parameter lists, allowing a single method name to handle different numbers or types of arguments. **Python does not support compile-time method overloading by signature**; redefining a method simply replaces the previous definition .

**Technical Definition**

In statically typed languages like Java and C++, the compiler selects the appropriate overloaded method based on argument types and count. Python, being dynamically typed, binds method names to single objects at runtime; a second definition of the same name overwrites the first. To achieve overloading-like flexibility, Python programmers use **default arguments**, **variable-length arguments (`*args`/`**kwargs`)** , **runtime type checking**, and **`functools.singledispatch`** for type-based dispatch . The `@typing.overload` decorator provides **static type-checker hints** but does not perform runtime dispatch .

**Beginner-Friendly Explanation**

In Java, you can write two `add` methods: one that adds two integers and one that adds three. Python does not do that. If you write two methods with the same name, the second one wins and the first disappears. Instead, Python gives you tools to write a single method that can handle different inputs: you can give parameters default values, accept any number of arguments with `*args`, or use a special decorator (`singledispatch`) that picks the right version based on the argument’s type .

#### Purposes

- To handle different numbers of arguments with a single method using defaults or `*args`.
- To handle different argument types with `functools.singledispatch`.
- To provide clear, type-hinted overload signatures for static type checkers via `@typing.overload`.
- To emulate the flexibility of overloaded APIs without sacrificing Python’s dynamic nature.
- To avoid cluttering the namespace with multiple nearly identical method names.

#### Syntax Rules and Structure

**Complete General Syntax**

```python
from functools import singledispatch

# Approach 1: Default and variable-length arguments.
def greet(name, greeting="Hello", *args, **kwargs):
    ...

# Approach 2: singledispatch for type-based dispatch.
@singledispatch
def process(arg):
    ...

@process.register(int)
def _(arg):
    ...

# Approach 3: typing.overload for static type hints (no runtime effect).
from typing import overload

@overload
def parse(value: int) -> str: ...
@overload
def parse(value: str) -> int: ...
def parse(value):
    ...
```

**Component Breakdown**

| Approach | Mechanism | Runtime Effect |
|----------|-----------|----------------|
| Defaults / `*args` / `**kwargs` | Single function handles varied arity | Yes |
| `isinstance()` checks | Manual type dispatch inside function | Yes |
| `functools.singledispatch` | Type-based dispatch on first argument | Yes |
| `@typing.overload` | Static type-checker stubs | No (type hints only) |

**Syntax Rules**

- Redefining a method with the same name replaces the previous definition.
- `singledispatch` dispatches on the type of the **first** argument.
- `singledispatchmethod` applies the same pattern to methods.
- `@overload` stubs must be followed by a single runtime implementation.
- `NotImplemented` (not `NotImplementedError`) should be returned when an operation is not supported in operator special methods.

**Constraints and Limitations**

- Python has no compile-time overload resolution.
- `singledispatch` supports only single-argument type dispatch; for multiple arguments, use the third-party `multipledispatch` library.
- `@overload` does not affect runtime behavior; it only informs type checkers.
- Overusing `isinstance()` checks can lead to brittle, non-polymorphic code.

#### Annotated Code Examples

**Example 1: Default and Variable-Length Arguments**

```python
def make_pizza(size, *toppings, **details):
    print(f"Making a {size} pizza with {', '.join(toppings)}")
    for key, value in details.items():
        print(f"  {key}: {value}")

make_pizza("large", "pepperoni", "mushrooms", extra_cheese=True)
```

**Expected Output:**

```
Making a large pizza with pepperoni, mushrooms
  extra_cheese: True
```

**Step-by-Step Breakdown:**

1. `size` is a required positional argument.
2. `*toppings` collects extra positional arguments into a tuple.
3. `**details` collects extra keyword arguments into a dictionary.

**Why the Output is Produced:**

A single function definition handles varying numbers of positional and keyword arguments, providing overload-like flexibility .

---

**Example 2: `functools.singledispatch`**

```python
from functools import singledispatch

@singledispatch
def describe(value):
    return f"Object: {value}"

@describe.register(int)
def _(value):
    return f"Integer: {value}"

@describe.register(str)
def _(value):
    return f"String: {value}"

print(describe(42))
print(describe("hello"))
print(describe([1, 2]))
```

**Expected Output:**

```
Integer: 42
String: hello
Object: [1, 2]
```

**Step-by-Step Breakdown:**

1. `@singledispatch` creates a generic function with a default implementation.
2. `@describe.register(int)` and `@describe.register(str)` add type-specific implementations.
3. At call time, Python dispatches based on the type of the first argument .

**Why the Output is Produced:**

`singledispatch` selects the most specific registered implementation for the argument’s type, falling back to the default for unregistered types .

#### Real-World Cases

- **Serialization libraries**: `json` encoders often use `singledispatch`-style dispatch for different types.
- **Generic functions**: `pprint`, `len`, and `copy` are built-in generic functions.
- **API clients**: Handling different response types with a single `parse()` entry point.
- **Data validation**: Type-specific validators registered via `singledispatch`.

---

### 4. Duck Typing

#### Definitions

**Core Definition**

Duck typing is a programming style in which an object’s suitability for a purpose is determined by whether it has the required methods and attributes, rather than by its declared type or inheritance hierarchy.

**Technical Definition**

Duck typing is a structural type system where type compatibility is checked at **call time** by attempting the operation. An object is considered compatible with a given type if it provides all the methods and attributes that the type requires. Python’s dynamic nature makes duck typing its default polymorphism mechanism: calling code does not check types; it simply invokes methods, and if the object supports them, the call succeeds. This is captured by the saying: “If it walks like a duck and quacks like a duck, then it must be a duck.” Since Python 3.8, `typing.Protocol` allows static type checkers to verify the same structural compatibility, a concept PEP 544 calls “static duck typing” .

**Beginner-Friendly Explanation**

Duck typing means you do not ask an object, “What class are you?” Instead, you ask, “Can you quack?” If the object has a `quack()` method, you treat it as a duck—even if it is a robot or a person. This makes Python code very flexible: any object that provides the right methods can be used, without needing to inherit from a specific class .

#### Purposes

- To write flexible, decoupled code that does not depend on concrete types.
- To allow objects from unrelated class hierarchies to be used interchangeably.
- To reduce the need for inheritance and complex type hierarchies.
- To enable rapid prototyping and extension with minimal boilerplate.
- To support generic programming through behavior-based interfaces.

#### Syntax Rules and Structure

**Complete General Syntax**

```python
def process(obj):
    return obj.required_method()  # no type check; duck typing in action
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| Method call | The only requirement is that the object has the method. |
| No `isinstance()` check | Type checks are omitted; behavior is checked at runtime. |
| Shared interface | The set of methods the calling code expects. |

**Syntax Rules**

- Calling code invokes methods without checking the object’s type.
- If the method is missing, `AttributeError` is raised at runtime.
- Formal contracts can be expressed with `typing.Protocol` (PEP 544) for static checking.
- ABCs (`abc.ABC`) can complement duck typing when implicit contracts are too loose .

**Constraints and Limitations**

- Errors surface at runtime, potentially in production.
- Without a formal protocol, it can be unclear what interface an object must provide.
- Static type checkers cannot verify duck-typed code without `Protocol` annotations.
- Duck typing can obscure the expected interface, making code harder to understand for newcomers.

#### Annotated Code Examples

**Example 1: Duck Typing with Unrelated Classes**

```python
class Dog:
    def speak(self):
        return "Woof!"

class Cat:
    def speak(self):
        return "Meow!"

class Robot:
    def speak(self):
        return "Beep boop"

def make_sound(entity):
    print(entity.speak())

for entity in [Dog(), Cat(), Robot()]:
    make_sound(entity)
```

**Expected Output:**

```
Woof!
Meow!
Beep boop
```

**Step-by-Step Breakdown:**

1. `Dog`, `Cat`, and `Robot` do not share a base class.
2. All three define `speak()`.
3. `make_sound()` calls `speak()` on any object it receives.

**Why the Output is Produced:**

Python does not care about the object’s class; it only checks that `speak()` exists at the moment of the call .

---

**Example 2: Static Duck Typing with `Protocol`**

```python
from typing import Protocol

class Speaker(Protocol):
    def speak(self) -> str: ...

def make_sound(entity: Speaker) -> None:
    print(entity.speak())

make_sound(Dog())
```

**Expected Output:**

```
Woof!
```

**Step-by-Step Breakdown:**

1. `Speaker` is a `Protocol` defining the required `speak()` method.
2. `make_sound()` accepts any object structurally compatible with `Speaker`.
3. Static type checkers verify compatibility without requiring inheritance .

**Why the Output is Produced:**

`Protocol` provides a formal, checkable version of duck typing without runtime overhead.

#### Real-World Cases

- **File-like objects**: Any object with `read()` can be passed to functions expecting a file .
- **Iteration**: Any object with `__iter__()` works in a `for` loop.
- **Testing**: Mocks and fakes work without inheriting from production classes.
- **Plugin systems**: Plugins are validated by the methods they expose, not their base classes.

---

### 5. Operator Overloading

#### Definitions

**Core Definition**

Operator overloading is the practice of defining special methods (dunder methods) in a class so that built-in operators such as `+`, `-`, `==`, and `<` behave in a custom way for instances of that class.

**Technical Definition**

Python allows user-defined classes to overload operators by implementing special methods named with double underscores. For example, `x + y` invokes `x.__add__(y)`; `x == y` invokes `x.__eq__(y)`; `x < y` invokes `x.__lt__(y)` . Rich comparison operators (`<`, `<=`, `>`, `>=`, `==`, `!=`) are defined by `__lt__`, `__le__`, `__gt__`, `__ge__`, `__eq__`, and `__ne__` . When a binary operation is not supported by the left operand, Python tries the right operand’s **reflected** method (e.g., `__radd__`) before raising `TypeError`. Special methods should return `NotImplemented` (not raise `NotImplementedError`) when they cannot handle the other operand, allowing the interpreter to try the other operand’s method .

**Beginner-Friendly Explanation**

Operator overloading lets your objects use familiar symbols like `+` and `==`. If you have a `Vector` class, you can define what `v1 + v2` means by writing an `__add__` method. Then you can write math-like code with your objects instead of calling methods like `v1.add(v2)`. Python handles the translation from the symbol `+` to your `__add__` method automatically .

#### Purposes

- To make user-defined types behave like built-in types with natural syntax.
- To support mathematical and logical operations on custom objects.
- To enable custom objects to be used with Python’s built-in functions and protocols.
- To provide readable, expressive code for domain-specific operations.
- To integrate user-defined types with Python’s rich comparison machinery.

#### Syntax Rules and Structure

**Complete General Syntax**

```python
class MyClass:
    def __add__(self, other):
        if not isinstance(other, MyClass):
            return NotImplemented
        return MyClass(...)

    def __eq__(self, other):
        if not isinstance(other, MyClass):
            return NotImplemented
        return ...
```

**Component Breakdown**

| Operator | Special Method | Reflected Method |
|----------|----------------|------------------|
| `+` | `__add__` | `__radd__` |
| `-` | `__sub__` | `__rsub__` |
| `*` | `__mul__` | `__rmul__` |
| `/` | `__truediv__` | `__rtruediv__` |
| `==` | `__eq__` | (swapped automatically) |
| `<` | `__lt__` | `__gt__` |
| `>` | `__gt__` | `__lt__` |
| `<=` | `__le__` | `__ge__` |
| `>=` | `__ge__` | `__le__` |

**Syntax Rules**

- Special methods are invoked automatically by the interpreter when the corresponding operator is used.
- Return `NotImplemented` (the special value) when the operation is not supported for the other operand.
- Do not raise `NotImplementedError` from operator special methods.
- Reflected methods are tried when the left operand returns `NotImplemented`.
- Rich comparison methods should return a boolean or `NotImplemented`.

**Constraints and Limitations**

- Overloading operators can make code harder to understand if the semantics are surprising.
- `NotImplemented` and `NotImplementedError` are different; using the wrong one breaks the protocol.
- Not all operators can be overloaded (e.g., `and`, `or`, `not` cannot be overloaded directly).
- Overloading must respect the Liskov Substitution Principle when subclassing built-in types.

#### Annotated Code Examples

**Example 1: Overloading `__add__` and `__eq__`**

```python
class Vector:
    def __init__(self, x, y):
        self.x, self.y = x, y

    def __add__(self, other):
        if not isinstance(other, Vector):
            return NotImplemented
        return Vector(self.x + other.x, self.y + other.y)

    def __eq__(self, other):
        if not isinstance(other, Vector):
            return NotImplemented
        return self.x == other.x and self.y == other.y

    def __repr__(self):
        return f"Vector({self.x}, {self.y})"

v1 = Vector(1, 2)
v2 = Vector(3, 4)
print(v1 + v2)
print(v1 == Vector(1, 2))
```

**Expected Output:**

```
Vector(4, 6)
True
```

**Step-by-Step Breakdown:**

1. `v1 + v2` invokes `Vector.__add__(v1, v2)`.
2. The method checks the type of `other` and returns a new `Vector`.
3. `v1 == Vector(1, 2)` invokes `__eq__`, which compares components.
4. `NotImplemented` is returned for unsupported types, allowing Python to try the other operand’s method.

**Why the Output is Produced:**

Python translates the operators `+` and `==` into calls to the corresponding special methods, which define the custom behavior .

---

**Example 2: `NotImplemented` for Unsupported Types**

```python
class Money:
    def __init__(self, amount):
        self.amount = amount

    def __add__(self, other):
        if not isinstance(other, Money):
            return NotImplemented
        return Money(self.amount + other.amount)

    def __repr__(self):
        return f"Money({self.amount})"

print(Money(10) + Money(5))
try:
    print(Money(10) + 5)
except TypeError as e:
    print("TypeError:", e)
```

**Expected Output:**

```
Money(15)
TypeError: unsupported operand type(s) for +: 'Money' and 'int'
```

**Step-by-Step Breakdown:**

1. `Money(10) + Money(5)` works because `__add__` handles `Money` operands.
2. `Money(10) + 5` returns `NotImplemented`, so Python tries `int.__radd__`, which also fails, and raises `TypeError`.

**Why the Output is Produced:**

Returning `NotImplemented` signals that the operation is unsupported, allowing the interpreter to try the other operand’s reflected method before raising `TypeError` .

#### Real-World Cases

- **Numerical types**: `decimal.Decimal`, `fractions.Fraction`, and NumPy arrays overload arithmetic operators.
- **Path objects**: `pathlib.Path` overloads `/` for path joining.
- **Datetime objects**: `datetime` and `timedelta` overload subtraction and comparison.
- **Domain models**: Money, vectors, matrices, and complex numbers.

---

### 6. Magic Methods (Dunder Methods)

#### Definitions

**Core Definition**

Magic methods, also called dunder methods (double underscore methods), are special methods with names surrounded by double underscores (e.g., `__init__`, `__str__`, `__add__`) that Python calls automatically to implement built-in operations on user-defined objects.

**Technical Definition**

Magic methods form the bridge between user-defined classes and Python’s built-in protocols. They are not called directly by the programmer; instead, the interpreter invokes them in response to specific syntax or built-in functions. Examples include `__init__` (object construction), `__str__` and `__repr__` (string representation), `__len__` (length), `__getitem__` (subscript access), `__iter__` (iteration), `__contains__` (`in` operator), and the rich comparison methods `__eq__`, `__lt__`, etc. . Magic methods are the foundation of polymorphism with built-in types: they allow user-defined objects to participate in Python’s protocols without inheriting from built-in classes.

**Beginner-Friendly Explanation**

Magic methods are the secret handshakes that let your objects work with Python’s built-in features. When you write `print(my_object)`, Python calls `my_object.__str__()`. When you write `len(my_object)`, it calls `__len__()`. When you use `my_object + other`, it calls `__add__()`. You do not call these methods yourself—Python calls them for you. By defining them, you make your objects behave like built-in types .

#### Purposes

- To integrate user-defined objects with Python’s built-in syntax and protocols.
- To provide string representations (`__str__`, `__repr__`).
- To enable length, iteration, and membership testing (`__len__`, `__iter__`, `__contains__`).
- To support indexing and slicing (`__getitem__`, `__setitem__`).
- To define comparison, arithmetic, and other operator behavior.
- To make custom objects interchangeable with built-in types in polymorphic code.

#### Syntax Rules and Structure

**Complete General Syntax**

```python
class MyClass:
    def __init__(self, value):
        self.value = value

    def __str__(self):
        return f"MyClass({self.value})"

    def __repr__(self):
        return f"MyClass(value={self.value!r})"

    def __len__(self):
        return len(self.value)

    def __getitem__(self, index):
        return self.value[index]

    def __iter__(self):
        return iter(self.value)
```

**Component Breakdown**

| Magic Method | Triggered By | Purpose |
|--------------|--------------|---------|
| `__init__` | `MyClass()` | Initialization |
| `__str__` | `str(obj)`, `print(obj)` | Human-readable string |
| `__repr__` | `repr(obj)`, containers | Developer-friendly string |
| `__len__` | `len(obj)` | Length |
| `__getitem__` | `obj[key]` | Subscript access |
| `__setitem__` | `obj[key] = value` | Subscript assignment |
| `__contains__` | `item in obj` | Membership test |
| `__iter__` | `for x in obj` | Iteration |
| `__eq__` | `obj == other` | Equality |
| `__lt__` | `obj < other` | Less than |
| `__add__` | `obj + other` | Addition |

**Syntax Rules**

- Magic methods are invoked implicitly by the interpreter.
- They should return `NotImplemented` when an operation is not supported (for binary special methods).
- `__repr__` should ideally return a string that could recreate the object.
- `__str__` falls back to `__repr__` if not defined.
- Defining `__eq__` without `__hash__` sets `__hash__` to `None` (unhashable).

**Constraints and Limitations**

- Do not invent new dunder names; they are reserved for Python’s use.
- Overusing magic methods can make code harder to understand.
- Inconsistent `__eq__` and `__hash__` implementations break dictionary and set behavior.
- Magic methods add overhead; avoid them in performance-critical inner loops when simple methods suffice.

#### Annotated Code Examples

**Example 1: `__str__`, `__repr__`, and `__len__`**

```python
class Playlist:
    def __init__(self, songs):
        self.songs = songs

    def __str__(self):
        return f"Playlist with {len(self.songs)} songs"

    def __repr__(self):
        return f"Playlist({self.songs!r})"

    def __len__(self):
        return len(self.songs)

p = Playlist(["Song A", "Song B"])
print(str(p))
print(repr(p))
print(len(p))
```

**Expected Output:**

```
Playlist with 2 songs
Playlist(['Song A', 'Song B'])
2
```

**Step-by-Step Breakdown:**

1. `print(p)` calls `__str__`.
2. `repr(p)` calls `__repr__`.
3. `len(p)` calls `__len__`.

**Why the Output is Produced:**

Python translates built-in operations into calls to the corresponding magic methods .

#### Real-World Cases

- **Custom containers**: `__getitem__`, `__len__`, and `__iter__` make objects behave like lists or dictionaries.
- **ORM models**: `__repr__` and `__eq__` provide useful debugging and comparison behavior.
- **Numerical types**: `__add__`, `__mul__`, and comparison methods enable arithmetic.
- **Context managers**: `__enter__` and `__exit__` enable the `with` statement.

---

### 7. Polymorphism Best Practices

#### Definitions

**Core Definition**

Polymorphism best practices are guidelines for designing and using polymorphic code that is correct, maintainable, and aligned with Python’s dynamic nature and the Liskov Substitution Principle.

**Technical Definition**

Best practices for polymorphism include: programming to an interface, not an implementation; preferring duck typing and Protocols over explicit `isinstance()` checks; honoring the Liskov Substitution Principle by ensuring subclasses are substitutable for their base classes; using `super()` consistently in overridden methods; returning `NotImplemented` from operator special methods when an operation is unsupported; and using `@singledispatch` for clean type-based dispatch instead of long `isinstance()` chains. These practices are informed by SOLID principles, particularly the Liskov Substitution Principle (LSP) and the Dependency Inversion Principle .

**Beginner-Friendly Explanation**

Best practices are the “good habits” of polymorphism. The biggest one is: **code to the interface, not the implementation**. Do not check types with `isinstance()` everywhere; just call the method you need. If a subclass inherits from a base class, it should behave like the base class—no surprising exceptions or narrowed contracts. When overriding a method, call `super()` if you want to keep the parent’s behavior. And when overloading operators, return `NotImplemented` if you do not know how to handle the other operand .

#### Purposes

- To write polymorphic code that is flexible, extensible, and easy to maintain.
- To avoid brittle type-checking code that violates the open/closed principle.
- To ensure subclasses can safely replace their base classes (LSP).
- To make operator overloading and special methods predictable and correct.
- To choose the right polymorphism mechanism for each situation.

#### Syntax Rules and Structure

**Complete General Syntax**

```python
# Prefer interfaces (Protocols) over concrete type checks.
from typing import Protocol

class Drawable(Protocol):
    def draw(self) -> None: ...

def render(item: Drawable) -> None:
    item.draw()

# Use super() in overrides.
class Base:
    def method(self):
        ...

class Derived(Base):
    def method(self):
        super().method()
        ...

# Return NotImplemented for unsupported operator operands.
class MyClass:
    def __add__(self, other):
        if not isinstance(other, MyClass):
            return NotImplemented
        ...
```

**Component Breakdown**

| Practice | Recommendation |
|----------|----------------|
| Interface-first design | Use Protocols or implicit duck-typed interfaces. |
| Avoid `isinstance()` | Prefer polymorphic dispatch over type checks. |
| Liskov Substitution | Subclasses must honor the base class contract. |
| `super()` in overrides | Call the parent implementation when extending. |
| `NotImplemented` | Return it from operator methods for unsupported types. |
| `singledispatch` | Use it for type-based dispatch instead of `isinstance` chains. |

**Syntax Rules**

- Define interfaces with `typing.Protocol` for static checking.
- Use duck typing for runtime flexibility.
- Call `super()` in overridden methods when extending behavior.
- Return `NotImplemented` (not raise) from binary special methods.
- Prefer `functools.singledispatch` over manual type dispatch.

**Constraints and Limitations**

- Protocols require `@runtime_checkable` for `isinstance()` checks, which has performance costs.
- LSP violations can be subtle; thorough testing is essential.
- Duck typing’s flexibility trades compile-time safety for runtime flexibility.
- Not all codebases benefit from formal Protocols; simple duck typing may suffice.

#### Annotated Code Examples

**Example 1: Programming to an Interface**

```python
from typing import Protocol

class Notifier(Protocol):
    def send(self, message: str) -> None: ...

class EmailNotifier:
    def send(self, message: str) -> None:
        print(f"Email: {message}")

class SMSNotifier:
    def send(self, message: str) -> None:
        print(f"SMS: {message}")

def alert(notifier: Notifier, message: str) -> None:
    notifier.send(message)

alert(EmailNotifier(), "Server down")
alert(SMSNotifier(), "Server down")
```

**Expected Output:**

```
Email: Server down
SMS: Server down
```

**Step-by-Step Breakdown:**

1. `Notifier` defines the `send()` interface as a Protocol.
2. `EmailNotifier` and `SMSNotifier` both implement `send()`.
3. `alert()` depends only on the interface, not on concrete classes.

**Why the Output is Produced:**

Polymorphic dispatch through a shared interface allows different notifiers to be used interchangeably without changing `alert()` .

#### Real-World Cases

- **Plugin architectures**: Plugins implement a Protocol or ABC; the core system calls their methods without type checks.
- **Dependency injection**: Services depend on interfaces, not concrete implementations.
- **Testing**: Fakes and mocks implement the same interface as production classes.
- **Framework extension**: Django, pytest, and other frameworks define hooks that subclasses override.

---

## References

- Python Documentation – `functools.singledispatch` (PEP 443) - https://peps.python.org/pep-0443/
- Python Documentation – Rich Comparisons (PEP 207) - https://peps.python.org/pep-0207/
- Python Documentation – Data Model (Special Methods) - https://docs.python.org/3/reference/datamodel.html
- Python Documentation – `typing.Protocol` (PEP 544) - https://peps.python.org/pep-0544/
- Python Documentation – Abstract Base Classes (`abc`) - https://docs.python.org/3/library/abc.html
- Real Python – Polymorphism (Glossary) - https://realpython.com/ref/glossary/polymorphism/
- Real Python – Duck Typing in Python: Writing Flexible and Decoupled Code - https://realpython.com/duck-typing-python/
- Real Python – Duck Typing (Glossary) - https://realpython.com/ref/glossary/duck-typing/
- Real Python – Subtyping and Liskov Substitution Principle - https://realpython.com/ref/glossary/subtyping/
- Guvi – What Is Method Overloading in Python? - https://www.guvi.in/blog/what-is-method-overloading-in-python/
- upGrad – Types of Polymorphism in Python - https://www.upgrad.com/blog/types-of-polymorphism-in-python/
- upGrad – Method Overriding in Python - https://www.upgrad.com/tutorials/software-engineering/python-tutorial/method-overriding-in-python/
- Williams College – Special Methods and Operator Overloading (CS134 Lecture 18) - https://williams-cs.github.io/cs134-s20-www/shikha-lectures/134-Lecture18.pdf
- Imperial College London – Magic/Dunder Methods (COMP70050) - https://intro2ml.pages.doc.ic.ac.uk/autumn2021/modules/lab-java/dunder
- DeepSource – PTC-W0054: Special method should return `NotImplemented` - https://app.deepsource.com
- CodeSignal – Applying SOLID Principles in Python (Liskov Substitution) - https://codesignal.com