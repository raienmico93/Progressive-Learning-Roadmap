# Getters and Setters in TypeScript — Comprehensive Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition (Academic).** **Getters and setters** are **accessor properties** — special methods declared with the `get` and `set` keywords — that intercept property reads and writes on an object, allowing computed values, validation, side-effect-free transformation, and controlled access to internal state while preserving **property-style syntax** at the call site (`obj.prop` rather than `obj.getProp()`).

**Technical Definition.** In TypeScript and ECMAScript, an accessor property is defined by a **getter function** (invoked on property read) and/or a **setter function** (invoked on property write). TypeScript adds **type annotations** to accessors and applies **special inference rules**: if a getter exists without a setter, the property is inferred as `readonly`; if a setter parameter lacks an explicit type, it is inferred from the getter’s return type; and since TypeScript 4.3, getters and setters may have **different types** (the getter’s type must be assignable to the setter’s parameter type) to model APIs that accept broad input but guarantee narrow output. Accessors are installed on the prototype, not on individual instances, and are **erased** as TypeScript constructs (though the underlying JavaScript accessors remain at runtime).

**Beginner-Friendly Explanation.** Getters and setters are **special methods that look like ordinary properties**. A getter runs when you **read** a property (`user.fullName`); a setter runs when you **assign** to it (`user.fullName = "Ada Lovelace"`). They let a class **control** what happens during reads and writes — for example, validating input, computing a value on the fly, or keeping internal data private while exposing a clean public interface. To the outside world, it looks like a normal property; behind the scenes, your code runs.

---

### Key Characteristics

| Characteristic | Description |
|---|---|
| **Property-style syntax** | Accessed and assigned like fields (`obj.prop`), not called like methods (`obj.getProp()`). |
| **Get-only means `readonly`** | A property with a getter but no setter is automatically inferred as `readonly` by TypeScript. |
| **Setter parameter inferred** | If the setter parameter type is omitted, it is inferred from the getter’s return type. |
| **Separate read/write types** | Since TypeScript 4.3, getters and setters may have different types, with the getter type assignable to the setter parameter type. |
| **Setters have no return type** | A `set` accessor cannot have a return type annotation; it must have **exactly one** parameter. |
| **Prototype installation** | Accessors are defined on the class prototype and shared across instances. |
| **Erased TS annotations** | Type annotations on accessors are compile-time only; the runtime accessor functions remain. |

---

### Prerequisites

- **JavaScript classes**: `class`, fields, methods, and instantiation.
- **TypeScript classes**: fields, constructors, methods, and access modifiers.
- **Encapsulation concepts**: private state, controlled mutation, and validation.
- **A TypeScript environment** (`tsc` or an editor with TypeScript support) to observe compiler behavior.

---

### Related Programming Areas

- **Encapsulation and information hiding** in object-oriented design.
- **Computed properties** and derived state.
- **Validation and normalization** at assignment boundaries.
- **Vue 3 Composition API** and **MobX** — reactive programming models built on getters/setters.
- **ECMAScript auto-accessors** (`accessor` keyword, TypeScript 4.9+) — a shorthand for field-backed getter/setter pairs.

---

### Core Concepts / Features

The following sections cover each requested concept. Each follows the uniform structure.

---

## 1. Getters

### Definitions

**Core Definition.** A **getter** is an accessor method declared with the `get` keyword that is **invoked automatically when a property is read**. It allows a class to compute or control the value returned for a property access without exposing the underlying storage.

**Technical Definition.** A getter is declared as `get name(): ReturnType { ... }`. It takes **no parameters** and must return a value. TypeScript checks that the returned value is assignable to the declared return type. When a getter exists without a corresponding setter, TypeScript treats the property as `readonly` at compile time. Getters are installed on the class prototype and are inherited by subclasses.

**Beginner-Friendly Explanation.** A getter is **a function that runs when you read a property**. You write `get fullName()` and return a computed value; callers just write `user.fullName` as if it were a normal field. It is useful for computed values (like full names or totals) and for exposing private data safely.

### Purposes

- **To compute derived values on demand** without storing redundant state.
- **To expose private state safely** through a controlled read-only interface.
- **To enforce immutability** by providing a getter without a setter (inferred `readonly`).
- **To abstract internal representation** so the public read shape can differ from internal storage.

### Syntax Rules and Structure

**General syntax:**

```
get propertyName(): ReturnType {
  return expression;
}
```

**Component breakdown:**

- `get` — keyword marking the accessor as a getter.
- `propertyName` — the property name.
- `(): ReturnType` — getters take **no parameters** and declare a return type.
- `{ return expression; }` — the body must return a value assignable to `ReturnType`.

**Syntax rules.**

- Getters must have **zero parameters**.
- Getters must **return a value**.
- A getter without a setter makes the property `readonly`.
- Getters may be `public`, `protected`, or `private`.
- Accessors cannot have the same name as a data field in the same class.

**Constraints and limitations.**

- Getters should be **cheap and side-effect-free**; expensive computation or observable side effects are discouraged.
- Getters should **avoid throwing exceptions**.
- Getters are invoked implicitly; debugging can be harder because the access looks like a field read.

### Annotated Complete Step by Step Code Examples

#### Example 1: Basic getter exposing private state

```typescript
// example_getter_basic.ts
class Student {
  private _firstName: string = "Monica";

  // Getter — accessed like a property.
  get firstName(): string {
    return this._firstName;
  }
}

const student = new Student();
console.log(student.firstName); // Expected: Monica

// student._firstName; // Error: Property '_firstName' is private.
```

**Setup.** Compile with `npx tsc example_getter_basic.ts --strict`.

**Expected Output:**
```
Monica
```

**Why this output occurs.** `firstName` is a getter that returns the private `_firstName`. External code reads `student.firstName` as if it were a field; the getter runs and returns `"Monica"`. Direct access to `_firstName` is rejected because it is private.

---

#### Example 2: Computed getter

```typescript
// example_getter_computed.ts
class Rectangle {
  constructor(private width: number, private height: number) {}

  get area(): number {
    return this.width * this.height;
  }

  get perimeter(): number {
    return 2 * (this.width + this.height);
  }
}

const rect = new Rectangle(4, 5);
console.log(rect.area);      // Expected: 20
console.log(rect.perimeter); // Expected: 18
```

**Expected Output:**
```
20
18
```

**Why this output occurs.** `area` and `perimeter` are computed on each read from the private `width` and `height`. No stored `area` or `perimeter` fields exist; the getters derive values on demand.

---

#### Example 3: Getter-only property is inferred as `readonly`

```typescript
// example_getter_readonly.ts
class User {
  protected _age!: number;

  get age(): number {
    return this._age;
  }
}

const user = new User();
// user.age = 20; // Error: Cannot assign to 'age' because it is a read-only property.
```

**Expected Output:**
```
(no output; compile-time error if uncommented)
```

**Why this output occurs.** Because `age` has a getter but **no setter**, TypeScript automatically infers the property as `readonly`. Assignment is rejected at compile time.

---

### Real-World Cases

- **Computed domain properties**: `get fullName()` combining first and last names.
- **Derived totals**: `get total()` in shopping cart or invoice classes.
- **Safe exposure of private data**: `get balance()` without allowing direct mutation.
- **Lazy initialization**: getters that compute and cache a value on first access.

---

## 2. Setters

### Definitions

**Core Definition.** A **setter** is an accessor method declared with the `set` keyword that is **invoked automatically when a property is assigned**. It allows a class to validate, transform, or reject incoming values before storing them.

**Technical Definition.** A setter is declared as `set name(value: Type) { ... }`. It must have **exactly one parameter** and **cannot have a return type annotation**. TypeScript checks the parameter type against assignments. If the parameter type is omitted, it is inferred from the corresponding getter’s return type. Since TypeScript 4.3, the setter parameter may accept a **broader** type than the getter returns, provided the getter type is assignable to the setter parameter type.

**Beginner-Friendly Explanation.** A setter is **a function that runs when you assign to a property**. You write `set age(value: number)` and put validation or transformation logic inside. Callers just write `user.age = 25` as if it were a normal field. Setters let you enforce rules — like “age must be positive” — without callers needing to call a special method.

### Purposes

- **To validate incoming values** before they are stored.
- **To normalize or transform input** (e.g., trimming strings, converting types).
- **To enforce invariants** and reject invalid assignments with errors or fallbacks.
- **To keep internal state consistent** when properties are assigned.

### Syntax Rules and Structure

**General syntax:**

```
set propertyName(value: Type) {
  // validation and assignment
}
```

**Component breakdown:**

- `set` — keyword marking the accessor as a setter.
- `propertyName` — the property name (must match the getter’s name if both exist).
- `(value: Type)` — **exactly one** parameter; no return type allowed.
- `{ ... }` — the body; must not return a value.

**Syntax rules.**

- Setters must have **exactly one parameter**.
- Setters **cannot have a return type annotation**.
- The parameter type may be omitted; it is inferred from the getter’s return type.
- Setters may be `public`, `protected`, or `private`.
- A setter without a getter is legal but unusual; reading the property yields `undefined` and is generally a design mistake.

**Constraints and limitations.**

- Setters cannot return a value; attempting to annotate a return type is a compile-time error.
- Setter logic should be **predictable and fast**; heavy computation or asynchronous operations should use explicit methods instead.
- A setter-only property is counterintuitive and should be replaced by a method if write-only behavior is genuinely needed.

### Annotated Complete Step by Step Code Examples

#### Example 1: Basic setter with validation

```typescript
// example_setter_basic.ts
class Student {
  private _firstName: string = "Monica";

  get firstName(): string {
    return this._firstName;
  }

  set firstName(newName: string) {
    if (newName.length === 0) {
      throw new Error("Name cannot be empty");
    }
    this._firstName = newName;
  }
}

const student = new Student();
console.log(student.firstName); // Expected: Monica

student.firstName = "Sarah";
console.log(student.firstName); // Expected: Sarah

// student.firstName = ""; // Throws: Name cannot be empty
```

**Expected Output:**
```
Monica
Sarah
```

**Why this output occurs.** The setter validates that `newName` is non-empty before assigning it to `_firstName`. The getter reads the stored value. Assignment uses property syntax, but the setter’s logic runs automatically.

---

#### Example 2: Setter without explicit parameter type (inferred from getter)

```typescript
// example_setter_inference.ts
class User {
  private _age!: number;

  get age(): number {
    return this._age;
  }

  // Parameter type inferred as number from getter return type.
  set age(value) {
    if (value < 0) throw new Error("Age cannot be negative");
    this._age = value;
  }
}

const user = new User();
user.age = 30;
console.log(user.age); // Expected: 30
```

**Expected Output:**
```
30
```

**Why this output occurs.** The setter parameter `value` has no explicit annotation, but TypeScript infers it as `number` from the getter’s return type. Assigning `30` satisfies the inferred type.

---

#### Example 3: Setter-only property (design smell)

```typescript
// example_setter_only.ts
class Book {
  private _title: string = "untitled";

  // Setter without getter — reading yields undefined.
  set title(value: string) {
    this._title = value;
  }
}

const book = new Book();
book.title = "The Hobbit";
console.log(book.title); // Expected: undefined (no getter)
```

**Expected Output:**
```
undefined
```

**Why this output occurs.** A setter without a getter means the property can be written but not meaningfully read; reading it returns `undefined` because no getter exists. This is generally a design mistake; if write-only behavior is intended, an explicit method such as `setTitle()` is clearer.

---

### Real-World Cases

- **Input validation**: rejecting negative quantities, empty names, or invalid emails.
- **Normalization**: trimming whitespace or lowercasing email addresses.
- **Reactive state**: Vue 3 and MobX use setters to trigger reactivity.
- **Configuration guards**: preventing invalid port numbers or timeouts.

---

## 3. `get` Keyword

### Definitions

**Core Definition.** The **`get` keyword** is the syntactic marker that declares a **getter accessor** on a class (or object literal). It creates a property whose reads are intercepted by a function.

**Technical Definition.** In a class body, `get` precedes a method declaration with no parameters: `get name(): Type { ... }`. The TypeScript compiler recognizes this as an accessor declaration and applies **accessor-specific inference rules**: a getter-only property is `readonly`; a setter’s parameter type is inferred from the getter’s return type; and the getter’s return type participates in separate read/write typing when a setter with a broader parameter type exists. At runtime, `get` is standard ECMAScript syntax and defines an accessor on the prototype.

**Beginner-Friendly Explanation.** `get` is **the word you type to create a getter**. It says “when someone reads this property, run this function instead of looking up a stored value.”

### Purposes

- **To declare a getter accessor** with a clear, recognizable syntax.
- **To trigger TypeScript’s accessor inference rules**, including automatic `readonly` for getter-only properties.
- **To enable computed properties** that are read like fields.
- **To integrate with decorators and metadata** (accessors can be decorated).

### Syntax Rules and Structure

**General syntax:**

```
get propertyName(): ReturnType {
  return value;
}
```

**Component breakdown:**

- `get` — accessor keyword.
- `propertyName` — property name.
- `(): ReturnType` — no parameters; return type annotation optional but recommended.
- `{ return value; }` — body must return a value.

**Syntax rules.**

- `get` must be followed by a property name and a parameterless method signature.
- The return type may be explicit or inferred from the body.
- A getter-only property is inferred as `readonly`.
- A getter may coexist with a setter of the same name; they form an accessor pair.

**Constraints and limitations.**

- Getters cannot have parameters.
- Getters should not have observable side effects or throw exceptions.
- Getter return types should be stable; returning different types on different reads is confusing and unsafe.

### Annotated Complete Step by Step Code Examples

#### Example 1: `get` with explicit return type

```typescript
// example_get_keyword.ts
class Circle {
  constructor(private radius: number) {}

  get area(): number {
    return Math.PI * this.radius ** 2;
  }
}

const c = new Circle(2);
console.log(c.area.toFixed(2)); // Expected: 12.57
```

**Expected Output:**
```
12.57
```

**Why this output occurs.** `get area(): number` declares a getter with an explicit return type. TypeScript verifies that the returned expression is a `number`. The property is read like a field.

---

#### Example 2: Getter and setter with separate read/write types (TypeScript 4.3+)

```typescript
// example_get_separate_types.ts
class Thing {
  #size = 0;

  get size(): number {
    return this.#size;
  }

  set size(value: string | number | boolean) {
    const num = Number(value);
    if (!Number.isFinite(num)) {
      this.#size = 0;
      return;
    }
    this.#size = num;
  }
}

const thing = new Thing();
thing.size = "42";      // Accepted: setter allows string.
console.log(thing.size); // Expected: 42 (getter returns number)
```

**Expected Output:**
```
42
```

**Why this output occurs.** The setter accepts `string | number | boolean`, converts the value to a number, and stores it. The getter always returns a `number`. TypeScript 4.3 permits this because the getter’s type (`number`) is assignable to the setter’s parameter type (`string | number | boolean`).

---

### Real-World Cases

- **Computed read-only properties**: `get isValid()` on form models.
- **Separate read/write types**: APIs that accept strings but expose numbers (as in the `Thing.size` example).
- **Lazy getters**: getters that initialize expensive resources on first read.

---

## 4. `set` Keyword

### Definitions

**Core Definition.** The **`set` keyword** is the syntactic marker that declares a **setter accessor** on a class (or object literal). It creates a property whose writes are intercepted by a function.

**Technical Definition.** In a class body, `set` precedes a method declaration with **exactly one parameter**: `set name(value: Type) { ... }`. The setter cannot have a return type annotation. TypeScript applies **accessor inference**: if the parameter type is omitted, it is inferred from the getter’s return type; if both accessors exist, their types must be related (getter type assignable to setter parameter type). At runtime, `set` defines a standard ECMAScript setter on the prototype.

**Beginner-Friendly Explanation.** `set` is **the word you type to create a setter**. It says “when someone assigns to this property, run this function instead of storing the value directly.”

### Purposes

- **To declare a setter accessor** with a clear, recognizable syntax.
- **To trigger TypeScript’s accessor inference rules**, including parameter type inference from the getter.
- **To enable separate write types** (broader input types than the getter returns).
- **To centralize validation and normalization logic** at the assignment boundary.

### Syntax Rules and Structure

**General syntax:**

```
set propertyName(value: Type) {
  // body
}
```

**Component breakdown:**

- `set` — accessor keyword.
- `propertyName` — property name.
- `(value: Type)` — exactly one parameter; type annotation optional.
- `{ ... }` — body must not return a value.

**Syntax rules.**

- Setters must have **exactly one** parameter.
- Setters **cannot have a return type annotation**.
- If the parameter type is omitted, it is inferred from the getter’s return type.
- Since TypeScript 4.3, the setter parameter may be **broader** than the getter return type.
- A setter may exist without a getter, but this is discouraged.

**Constraints and limitations.**

- Setter bodies must be fast and predictable; avoid heavy computation or side effects.
- A setter-only property is usually a design mistake; prefer a method for write-only operations.
- Inheriting a setter without the getter can make the getter `undefined`; keep accessor pairs together.

### Annotated Complete Step by Step Code Examples

#### Example 1: `set` with parameter type inference

```typescript
// example_set_keyword.ts
class User {
  private _username = "";

  get username(): string {
    return this._username;
  }

  // Parameter type inferred as string from the getter.
  set username(value) {
    this._username = value.trim();
  }
}

const user = new User();
user.username = "  Ada  ";
console.log(user.username); // Expected: Ada
```

**Expected Output:**
```
Ada
```

**Why this output occurs.** The setter parameter type is inferred as `string` from the getter’s return type. The setter trims the input before storing it. The getter returns the normalized value.

---

#### Example 2: Setter with explicit broader type (separate write type)

```typescript
// example_set_broader.ts
class Config {
  #port = 3000;

  get port(): number {
    return this.#port;
  }

  set port(value: string | number) {
    const num = Number(value);
    if (!Number.isInteger(num) || num < 0 || num > 65535) {
      throw new Error("Invalid port");
    }
    this.#port = num;
  }
}

const config = new Config();
config.port = "8080";  // String accepted.
console.log(config.port); // Expected: 8080 (number)
```

**Expected Output:**
```
8080
```

**Why this output occurs.** The setter accepts `string | number`, converts to a number, and validates the range. The getter returns a `number`. This models an API that is **permissive on write** and **precise on read**, enabled by TypeScript 4.3’s separate write types.

---

### Real-World Cases

- **Configuration validation**: accepting strings from environment variables but storing numbers.
- **Form models**: trimming and validating input on assignment.
- **Reactive frameworks**: Vue 3’s `ref` and MobX’s `observable` use setters to trigger updates.

---

## 5. Getter and Setter Best Practices

### Definitions

**Core Definition.** **Getter and setter best practices** are guidelines for using accessors in ways that preserve **clarity, predictability, performance, and API stability**, while avoiding common pitfalls such as side-effectful getters, setter-only properties, and accessor pairs that are scattered or type-incompatible.

**Technical Definition.** Best practices encompass **grouping accessor pairs** for the same property (ESLint `grouped-accessor-pairs`), **keeping getters cheap and side-effect-free**, **avoiding exceptions in getters**, **preferring explicit methods for complex or write-only operations**, **using separate read/write types deliberately** (TypeScript 4.3+), and **using `readonly` or public fields when no accessor logic is needed**. The `accessor` keyword (TypeScript 4.9+) offers a concise alternative for field-backed accessor pairs.

**Beginner-Friendly Explanation.** Best practices are **the habits experienced TypeScript developers use** to keep getters and setters clean. The big rules are: do not make getters expensive or surprising; do not create a setter without a getter; keep the getter and setter for the same property next to each other; and if you do not need custom logic, just use a plain public field instead of writing a do-nothing accessor pair.

### Purposes

- **To keep accessors predictable** so callers can treat them like ordinary properties.
- **To avoid performance and debugging surprises** from expensive or side-effectful getters.
- **To prevent API design mistakes** such as setter-only properties or scattered accessor pairs.
- **To use accessors only when they add value**, avoiding pointless boilerplate.

### Syntax Rules and Structure

Best practices are conventions that interact with accessor syntax. The core rules are:

1. **Group accessor pairs.** Define the getter and setter for the same property together in the class body.
2. **Keep getters cheap and pure.** Getters should not perform expensive operations, cause observable side effects, or throw exceptions.
3. **Avoid setter-only properties.** A setter without a getter makes reads return `undefined` and is usually a mistake.
4. **Prefer methods for complex operations.** If an operation is expensive or has side effects, use an explicit method rather than a getter or setter.
5. **Use public fields when no logic is needed.** A field-backed get/set pair with no extra logic is rarely useful; expose a public field instead.
6. **Use separate read/write types deliberately.** When modeling APIs that convert input, use TypeScript 4.3’s separate accessor types.
7. **Consider the `accessor` keyword** for simple field-backed accessors (TypeScript 4.9+).

**Component breakdown (accessor decision tree):**

```
Does the property need custom logic on read or write?
  ├── No → Use a public field (or readonly field).
  └── Yes → Is the logic expensive or side-effectful?
            ├── Yes → Use explicit methods (getX / setX).
            └── No → Use get/set accessors.
                     ├── Group the pair together.
                     └── Keep getters pure and cheap.
```

**Constraints and limitations.**

- Grouping accessor pairs is a style convention; it does not affect runtime behavior.
- The `accessor` keyword is TypeScript 4.9+ and may not be supported in all build pipelines.
- Separate read/write types can confuse readers if overused; document the intent.

### Annotated Complete Step by Step Code Examples

#### Example 1: Grouped accessor pair

```typescript
// example_best_grouped.ts
class User {
  private _fullName = "";

  // ✅ Getter and setter grouped together.
  get fullName(): string {
    return this._fullName;
  }

  set fullName(name: string) {
    this._fullName = name.trim();
  }
}

const user = new User();
user.fullName = "  Ada Lovelace  ";
console.log(user.fullName); // Expected: Ada Lovelace
```

**Expected Output:**
```
Ada Lovelace
```

**Why this output occurs.** The getter and setter are adjacent, making the accessor pair easy to find and reason about. The setter trims the input; the getter returns the normalized value. This follows the grouped-accessor-pairs best practice.

---

#### Example 2: Preferring an explicit method for side-effectful operations

```typescript
// example_best_method.ts
class Stream {
  private pointer = 0;
  private data = ["a", "b", "c"];

  // ❌ Avoid: getter with an observable side effect.
  // get next(): string {
  //   return this.data[this.pointer++]; // Mutates state on read.
  // }

  // ✅ Prefer: explicit method communicates the side effect.
  readNext(): string {
    return this.data[this.pointer++];
  }
}

const s = new Stream();
console.log(s.readNext()); // Expected: a
console.log(s.readNext()); // Expected: b
```

**Expected Output:**
```
a
b
```

**Why this output occurs.** Reading a property should not secretly advance a stream pointer. The explicit `readNext()` method signals that a state change occurs, avoiding the surprising behavior that a side-effectful getter would create.

---

#### Example 3: Using the `accessor` keyword (TypeScript 4.9+)

```typescript
// example_best_auto_accessor.ts
class Person {
  // Auto-accessor: creates a private backing field and public get/set.
  accessor name: string = "Anonymous";
}

const person = new Person();
console.log(person.name); // Expected: Anonymous
person.name = "Ada";
console.log(person.name); // Expected: Ada
```

**Expected Output:**
```
Anonymous
Ada
```

**Why this output occurs.** The `accessor` keyword automatically creates a field-backed getter/setter pair, equivalent to a private field with trivial accessors. It reduces boilerplate when no custom logic is needed.

---

### Real-World Cases

- **Form models**: grouped `get`/`set` pairs with validation on assignment.
- **Reactive state**: `accessor`-based fields in frameworks that observe property changes.
- **Domain entities**: pure getters for computed values; explicit methods for state-changing operations.
- **API design**: separate read/write types for permissive input and precise output.

---

## References

- TypeScript Handbook — Classes: Getters / Setters - https://www.typescriptlang.org/docs/handbook/2/classes.html#getters--setters
- TypeScript 4.3 Release Notes — Separate Write Types on Properties - https://www.typescriptlang.org/docs/handbook/release-notes/typescript-4-3.html#separate-write-types-on-properties
- MDN Web Docs — `get` (Accessor) - https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/get
- MDN Web Docs — `set` (Accessor) - https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/set
- TypeScript 4.9 Release Notes — Auto-Accessors in Classes - https://devblogs.microsoft.com/typescript/announcing-typescript-4-9/#auto-accessors-in-classes
- ESLint — `grouped-accessor-pairs` - https://eslint.org/docs/latest/rules/grouped-accessor-pairs
- API Extractor — `ae-missing-getter` (Getter/Setter Design Guidelines) - https://api-extractor.com/pages/messages/ae-missing-getter/
- TypeScript Book — Limit Usage of Property Setters - https://github.com/sorenlouv/typescript-book/blob/master/docs/tips/propertySetters.md
- Refine — TypeScript Classes: Getter and Setter (Inference Rules and Pitfalls) - https://raw.githubusercontent.com/refinedev/refine/724f795367cd8c5bd4fdb7c77e9fed45bbda000d/documentation/blog/2025-01-13-ts-classes.md