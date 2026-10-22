# Interfaces in TypeScript — Comprehensive Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition (Academic).** A TypeScript **interface** is a purely **type-level construct** that names an object's shape by declaring the members—properties, methods, index signatures, call signatures, and construct signatures—that values of that type must provide. Interfaces have **no runtime representation**; they exist only to enable structural type checking, contract documentation, and tooling intelligence. They may **extend** one or more other interfaces, composing their contracts into a new named type, and classes may **implement** them to assert conformance.

**Technical Definition.** An interface declaration `interface Name { ... }` introduces a named object type into the **type space**. It may contain property signatures (required, optional, or `readonly`), method signatures, call signatures, construct signatures, and index signatures. Interfaces are **open**: multiple declarations with the same name in the same scope **merge** into a single type. They support **generic type parameters**, **heritage clauses** (`extends`), and **declaration merging**. A class's `implements` clause is a **compile-time assertion** that the class's public instance type satisfies the interface; it establishes no runtime relationship and installs no inherited implementation. All interface constructs are **erased** during compilation and emit no JavaScript.

**Beginner-Friendly Explanation.** An interface is **a contract that describes what an object must look like**. It lists the properties (data) and methods (actions) an object must have, without providing any actual values or code. When a class says `implements SomeInterface`, it is **promising** to provide everything that interface requires. TypeScript checks that promise at compile time. If the class is missing a required member or has the wrong type, you get an error before the code ever runs. Interfaces disappear completely when TypeScript becomes JavaScript—they are a development-time safety net, not a runtime enforcement mechanism.

---

### Key Characteristics

| Characteristic | Description |
|---|---|
| **Pure type construct** | Interfaces exist only at compile time and are entirely erased in emitted JavaScript. |
| **Structural contract** | An interface describes a shape; any value with that shape is compatible, regardless of nominal identity. |
| **Open and mergeable** | Interfaces with the same name in the same scope automatically merge (declaration merging). |
| **Extensible** | Interfaces may extend one or more other interfaces, composing their contracts. |
| **Optional and readonly members** | Interfaces support `?` for optional members and `readonly` for immutable members. |
| **Multiple signature kinds** | Interfaces can declare property signatures, method signatures, call signatures, construct signatures, and index signatures. |
| **Generic** | Interfaces may be parameterized with type parameters for reusable contracts. |
| **No implementation inheritance** | `extends` on interfaces composes contracts; it does not inherit method bodies. |

---

### Prerequisites

- **TypeScript basics**: type annotations, type aliases, and structural typing.
- **JavaScript objects**: object literals, properties, and functions.
- **Object-oriented concepts**: contracts, polymorphism, and the distinction between “is-a” (`extends`) and “can-do” (`implements`).
- **A TypeScript environment** (`tsc` or an editor with TypeScript support) to observe compiler behavior.

---

### Related Programming Areas

- **Structural typing** and **duck typing** (TypeScript’s compatibility model).
- **Class inheritance** (`extends`) and **abstract classes** — contrast with interface contracts.
- **Declaration merging** for augmenting third-party library types.
- **Dependency inversion** and **interface segregation** (SOLID principles).
- **Utility types** such as `Partial<T>`, `Readonly<T>`, and `Record<K, V>`.

---

## Core Concepts / Features

---

## 1. Interfaces

### Definitions

**Core Definition.** An **interface** is a named type declaration that specifies the **shape** — properties, methods, and signatures — that a value must provide. It contains no implementation and exists solely for compile-time checking and documentation.

**Technical Definition.** An interface declaration `interface Name { ... }` introduces a type into the type space. It may include property signatures (`prop: Type`), optional properties (`prop?: Type`), `readonly` properties (`readonly prop: Type`), method signatures (`method(params): ReturnType`), call signatures (`(args): ReturnType`), construct signatures (`new (args): Type`), and index signatures (`[key: string]: Type`). Interfaces are **open**: multiple declarations with the same name in the same scope merge. They can extend other interfaces via `extends`. They are **erased** during compilation and have no runtime counterpart.

**Beginner-Friendly Explanation.** An interface is **a checklist for objects**. It says “if you want to be a `User`, you must have a `name` that is a string and an `email` that is a string.” Any object that has those things is a `User` as far as TypeScript is concerned—even if it was never explicitly declared to be one. Interfaces are only for the type checker; they vanish when the code runs.

### Purposes

- **To name and document object shapes** so the same contract can be reused across a codebase.
- **To define contracts for classes** that must be verified at compile time.
- **To enable structural polymorphism** where any conforming value is accepted, regardless of its origin.
- **To support declaration merging** for augmenting library types and environment declarations.
- **To serve as the preferred tool for public object-type APIs** because interfaces produce clearer compiler errors and support open extension.

### Syntax Rules and Structure

**General syntax (interface declaration):**

```
interface InterfaceName {
  propertyName: Type;
  optionalProperty?: Type;
  readonly immutableProperty: Type;
  methodName(params: Type): ReturnType;
  [index: string]: Type;
}
```

**Component breakdown:**

- `interface` — declaration keyword.
- `InterfaceName` — identifier (PascalCase convention).
- `propertyName: Type` — required property and its type.
- `optionalProperty?: Type` — optional property; its type is `Type | undefined`.
- `readonly immutableProperty: Type` — property that cannot be reassigned after creation.
- `methodName(params): ReturnType` — method signature.
- `[index: string]: Type` — index signature for dynamic keys.

**Syntax rules.**

- Interfaces may extend one or more interfaces: `interface Derived extends Base1, Base2 { ... }`.
- Interfaces may be generic: `interface Box<T> { value: T; }`.
- Interfaces are **open**: redeclaring the same interface name in the same scope merges the declarations.
- Interfaces may describe function types via call signatures: `interface Fn { (x: number): string; }`.
- Interfaces may describe constructor types via construct signatures: `interface Ctor { new (): object; }`.

**Constraints and limitations.**

- Interfaces **cannot** represent unions, intersections, primitives, tuples, or mapped/conditional types directly; use `type` aliases for those.
- Interfaces are erased and cannot be used with `instanceof`.
- Declaration merging can be surprising; conflicting property types across merged declarations produce errors.
- Interfaces do not exist at runtime; runtime validation requires user-defined type guards.

### Annotated Complete Step by Step Code Examples

#### Example 1: Basic interface describing an object shape

```typescript
// example_interface_basic.ts
// Step 1: Declare an interface.
interface User {
  name: string;
  age: number;
  email?: string;         // Optional
  readonly id: string;    // Immutable after creation
}

// Step 2: Create a conforming object.
const user: User = {
  name: "Ada",
  age: 36,
  id: "u1",
};

// Step 3: Access properties.
console.log(user.name);  // Expected: Ada
console.log(user.age);   // Expected: 36
console.log(user.email); // Expected: undefined

// user.id = "u2"; // Error: Cannot assign to 'id' because it is a read-only property.
```

**Setup.** Compile with `npx tsc example_interface_basic.ts --strict`.

**Expected Output:**
```
Ada
36
undefined
```

**Why this output occurs.** The object literal is checked against `User`: `name` and `age` are required and present; `email` is optional and omitted; `id` is readonly and initialized once. The compiler enforces these constraints; at runtime, the interface is erased and only the object properties remain.

---

#### Example 2: Interface with method and index signatures

```typescript
// example_interface_methods.ts
interface StringMap {
  [key: string]: string | undefined;
}

interface Greeter {
  greet(name: string): string;
  defaultGreeting: string;
}

const map: StringMap = { hello: "world", foo: "bar" };
console.log(map["hello"]); // Expected: world

const greeter: Greeter = {
  defaultGreeting: "Hello",
  greet(name) {
    return `${this.defaultGreeting}, ${name}`;
  },
};

console.log(greeter.greet("Ada")); // Expected: Hello, Ada
```

**Expected Output:**
```
world
Hello, Ada
```

**Why this output occurs.** `StringMap` uses an index signature to allow arbitrary string keys with string-or-undefined values. `Greeter` requires a `greet` method and a `defaultGreeting` property. The object literal satisfies both. At runtime, the methods execute normally; the interface annotations are erased.

---

### Real-World Cases

- **API response contracts**: `interface ApiResponse<T> { data: T; status: number; error?: string; }`.
- **Configuration shapes**: `interface AppConfig { port: number; host: string; }`.
- **React props**: `interface ButtonProps { label: string; onClick: () => void; }`.
- **Library type augmentation**: merging a `Window` interface to add custom global properties.

---

## 2. `interface` Keyword

### Definitions

**Core Definition.** The **`interface` keyword** is the syntactic token that introduces an interface declaration. It signals that the following identifier names a **type** describing an object’s shape, not a runtime value.

**Technical Definition.** In TypeScript’s grammar, an interface declaration is `interface BindingIdentifier TypeParameters? InterfaceExtendsClause? ObjectType`. The `interface` token is a **type-space declaration keyword**; it does not create a value binding. The name introduced is available only in type positions (annotations, generic arguments, `extends`/`implements` clauses).

**Beginner-Friendly Explanation.** `interface` is **the word you type to start an interface**. Everything after it—the name, the body, the `extends` clause—describes the contract. It is the TypeScript equivalent of saying “I am defining a shape now.”

### Purposes

- **To introduce a named interface type** into the current scope.
- **To distinguish interface declarations from type aliases and class declarations**.
- **To enable declaration merging** when the same interface name is declared multiple times.

### Syntax Rules and Structure

**General syntax:**

```
interface InterfaceName {
  // members
}
```

**Component breakdown:**

- `interface` — declaration keyword.
- `InterfaceName` — binding identifier (PascalCase).
- `{ ... }` — body containing members.

**Syntax rules.**

- The interface name must be a valid identifier and must not be a predefined type name.
- Interfaces are **hoisted** within their scope; they can be referenced before their declaration.
- The same interface name may be declared multiple times; the declarations merge.
- The `interface` keyword cannot be used to declare a runtime value.

**Constraints and limitations.**

- Interfaces cannot declare **private** or **protected** members (those are class-only).
- An interface name cannot be redefined as a different kind of type in the same scope without merging or conflict.
- Interfaces are erased; the keyword produces no JavaScript.

### Annotated Complete Step by Step Code Examples

#### Example 1: Basic `interface` keyword usage

```typescript
// example_interface_keyword.ts
// Step 1: Use the interface keyword.
interface Point {
  x: number;
  y: number;
}

// Step 2: Use the interface as a type.
const p: Point = { x: 1, y: 2 };
console.log(p.x, p.y); // Expected: 1 2
```

**Expected Output:**
```
1 2
```

**Why this output occurs.** The `interface` keyword introduces the `Point` type. The object literal is checked against it. At runtime, the interface is erased and only the object remains.

---

#### Example 2: Declaration merging with the `interface` keyword

```typescript
// example_interface_merge.ts
interface Box {
  width: number;
}

// Second declaration merges with the first.
interface Box {
  height: number;
}

const box: Box = { width: 10, height: 20 };
console.log(box.width + box.height); // Expected: 30
```

**Expected Output:**
```
30
```

**Why this output occurs.** The two `interface Box` declarations merge into a single type with both `width` and `height`. Declaration merging is a feature unique to interfaces (and namespaces) and is not available for type aliases.

---

### Real-World Cases

- **Augmenting global types**: declaring `interface Window { myGlobal: string }` to extend the DOM.
- **Library authoring**: multiple interface declarations that merge across files.
- **Domain modeling**: naming object shapes for reuse across modules.

---

## 3. Interface Declaration

### Definitions

**Core Definition.** An **interface declaration** is the complete syntactic statement that defines an interface, including its name, optional type parameters, optional `extends` clause, and object-type body.

**Technical Definition.** An interface declaration introduces a **named type** into the containing declaration space. The declaration may be **generic** (`interface Box<T> { ... }`), may **extend** zero or more base interfaces or class types, and may contain any combination of property, method, call, construct, and index signatures. The declared type is **structural**: any value with the required members is compatible.

**Beginner-Friendly Explanation.** An interface declaration is **the full statement that defines an interface**. It includes the name, any generic parameters, any parent interfaces, and the body listing the required members.

### Purposes

- **To define a reusable named contract** for object shapes.
- **To compose existing contracts** via `extends`.
- **To parameterize contracts** with generic type parameters.
- **To document the required shape** of function parameters, return values, and class instances.

### Syntax Rules and Structure

**General syntax (full form):**

```
interface InterfaceName<T> extends Base1, Base2 {
  // members
}
```

**Component breakdown:**

- `interface` — keyword.
- `InterfaceName` — identifier.
- `<T>` — optional type parameters.
- `extends Base1, Base2` — optional heritage clause.
- `{ ... }` — object-type body.

**Syntax rules.**

- The heritage clause may list multiple base interfaces or class types.
- The interface’s instance type must be assignable to each base type.
- Inherited properties with the same name must be **identical** in type.
- If two bases have the same property with **different** types, the derived interface must **redeclare** the property with a type that is a subtype of both.
- An interface may not directly or indirectly extend itself.

**Constraints and limitations.**

- An interface cannot declare a property with the same name as an inherited **private** or **protected** property from a class base.
- Inherited members with the same name must be identical unless redeclared.
- All properties must satisfy any index signatures declared in the interface.

### Annotated Complete Step by Step Code Examples

#### Example 1: Generic interface declaration

```typescript
// example_interface_generic.ts
// Step 1: Declare a generic interface.
interface Box<T> {
  value: T;
}

// Step 2: Use with different type arguments.
const stringBox: Box<string> = { value: "hello" };
const numberBox: Box<number> = { value: 42 };

console.log(stringBox.value.toUpperCase()); // Expected: HELLO
console.log(numberBox.value.toFixed(2));    // Expected: 42.00
```

**Expected Output:**
```
HELLO
42.00
```

**Why this output occurs.** `Box<T>` is parameterized by `T`, which is substituted with `string` and `number` respectively. The compiler checks each object against the instantiated interface.

---

#### Example 2: Interface declaration with heritage

```typescript
// example_interface_heritage.ts
interface Shape {
  color: string;
}

interface Square extends Shape {
  sideLength: number;
}

const square: Square = {
  color: "red",
  sideLength: 10,
};

console.log(square.color);      // Expected: red
console.log(square.sideLength); // Expected: 10
```

**Expected Output:**
```
red
10
```

**Why this output occurs.** `Square` extends `Shape`, inheriting `color`. The object literal must satisfy both `color` and `sideLength`. The compiler enforces the combined contract.

---

### Real-World Cases

- **Generic API wrappers**: `interface PaginatedResponse<T> { items: T[]; total: number; }`.
- **Layered domain models**: `interface Employee extends Person { salary: number; }`.
- **Framework contracts**: interfaces describing component props, service shapes, and configuration objects.

---

## 4. Interface Properties

### Definitions

**Core Definition.** **Interface properties** are the named data members declared in an interface body. Each property declares a **name** and a **type**, and may be modified with `?` (optional) or `readonly` (immutable).

**Technical Definition.** A property signature has the form `propertyName: Type` or `propertyName?: Type` or `readonly propertyName: Type`. The property’s type may be any valid TypeScript type expression. All properties declared without `?` are **required**; the implementing object must provide them. A property’s type participates in structural compatibility checks.

**Beginner-Friendly Explanation.** Interface properties are **the pieces of data an object must have**. `name: string` means every object conforming to the interface must have a `name` that is a string.

### Purposes

- **To specify the data shape** that conforming objects must provide.
- **To document property types** for editor autocomplete and compile-time checking.
- **To combine required, optional, and readonly properties** in a single contract.
- **To model both simple and complex data** through property type annotations.

### Syntax Rules and Structure

**General syntax:**

```
interface InterfaceName {
  requiredProperty: Type;
  optionalProperty?: Type;
  readonly immutableProperty: Type;
}
```

**Component breakdown:**

- `requiredProperty: Type` — required property.
- `optionalProperty?: Type` — optional property.
- `readonly immutableProperty: Type` — immutable property.

**Syntax rules.**

- Properties may be separated by semicolons, commas, or newlines.
- Property names may be identifiers, string literals, or numeric literals.
- Computed property names are supported in type positions via `[Symbol.iterator]` etc.
- Properties may have function types: `onClick: (event: Event) => void`.

**Constraints and limitations.**

- Required properties must be present on conforming objects.
- Optional properties may be omitted; their type includes `undefined` when accessed.
- `readonly` prevents reassignment after object creation but is **shallow** for nested objects.
- Properties cannot be `private` or `protected` in interfaces.

### Annotated Complete Step by Step Code Examples

#### Example 1: Required and optional properties

```typescript
// example_interface_props.ts
interface User {
  name: string;
  age?: number;
}

const user1: User = { name: "Ada" };
const user2: User = { name: "Grace", age: 85 };

console.log(user1.age); // Expected: undefined
console.log(user2.age); // Expected: 85
```

**Expected Output:**
```
undefined
85
```

**Why this output occurs.** `name` is required and present in both objects. `age` is optional; `user1` omits it, so reading `user1.age` yields `undefined`. `user2` provides it, so reading it yields `85`.

---

#### Example 2: Properties with function types

```typescript
// example_interface_prop_function.ts
interface ButtonProps {
  label: string;
  onClick: (event: MouseEvent) => void;
}

const button: ButtonProps = {
  label: "Submit",
  onClick: (event) => console.log("Clicked", event.type),
};

console.log(button.label); // Expected: Submit
```

**Expected Output:**
```
Submit
```

**Why this output occurs.** `onClick` is a property whose type is a function. The object provides a compatible function. The compiler checks the function’s parameter and return types.

---

### Real-World Cases

- **React props**: required and optional properties for component configuration.
- **Configuration objects**: `interface Config { host: string; port?: number; }`.
- **API models**: properties representing JSON fields with optional or nullable types.

---

## 5. Optional Properties

### Definitions

**Core Definition.** An **optional property** is an interface property marked with `?` after its name, indicating that conforming objects **may omit** the property. Its type is effectively `Type | undefined` when accessed.

**Technical Definition.** An optional property signature `propertyName?: Type` tells the compiler that the property is not required. Objects may omit it entirely; when present, its value must be of the declared type. Reading an optional property yields `Type | undefined` (and under `exactOptionalPropertyTypes`, the distinction between missing and `undefined` is preserved).

**Beginner-Friendly Explanation.** An optional property is **a piece of data that an object may or may not have**. `email?: string` means an object can have an `email` string, or it can leave it out entirely.

### Purposes

- **To model properties that are not always present** in conforming objects.
- **To support “option bag” patterns** where only some configuration fields are supplied.
- **To reduce the burden on object creators** by not requiring every field.
- **To represent partial or conditional data** in API and configuration models.

### Syntax Rules and Structure

**General syntax:**

```
propertyName?: Type;
```

**Component breakdown:**

- `propertyName` — identifier or string literal.
- `?` — optionality marker.
- `: Type` — the property’s type when present.

**Syntax rules.**

- The `?` immediately follows the property name.
- Optional properties may be combined with `readonly`: `readonly prop?: Type`.
- Reading an optional property yields `Type | undefined`.
- Optional properties do not need to be provided by object literals or implementing classes.

**Constraints and limitations.**

- Optional properties are **not the same as `undefined`** in strict mode; `exactOptionalPropertyTypes` distinguishes missing from explicitly `undefined`.
- Optional properties can hide errors if consumers forget to check for `undefined`.
- Optional properties do not enforce that **at least one** of a group is present; for that, use unions or discriminated unions.

### Annotated Complete Step by Step Code Examples

#### Example 1: Optional property in an option bag

```typescript
// example_optional_prop.ts
interface PaintOptions {
  shape: string;
  xPos?: number;
  yPos?: number;
}

const opts1: PaintOptions = { shape: "circle" };
const opts2: PaintOptions = { shape: "square", xPos: 10 };

console.log(opts1.xPos); // Expected: undefined
console.log(opts2.xPos); // Expected: 10
```

**Expected Output:**
```
undefined
10
```

**Why this output occurs.** `xPos` and `yPos` are optional. `opts1` omits them, so `opts1.xPos` is `undefined`. `opts2` provides `xPos`, so it is `10`. The compiler permits both objects.

---

#### Example 2: Optional readonly property

```typescript
// example_optional_readonly.ts
interface Config {
  readonly apiKey?: string;
}

const config: Config = {};
console.log(config.apiKey); // Expected: undefined

// config.apiKey = "secret"; // Error: Cannot assign to 'apiKey' because it is a read-only property.
```

**Expected Output:**
```
undefined
```

**Why this output occurs.** `apiKey` is both optional and readonly. It may be omitted, but if present, it cannot be reassigned after object creation.

---

### Real-World Cases

- **React props**: optional callbacks, styles, and configuration.
- **API request parameters**: optional query parameters and headers.
- **Configuration objects**: optional overrides with sensible defaults.

---

## 6. Readonly Properties

### Definitions

**Core Definition.** A **readonly property** is an interface property marked with `readonly`, indicating that the property **cannot be reassigned** after the object is created. Reading is permitted; writing is rejected at compile time.

**Technical Definition.** A `readonly` property signature `readonly prop: Type` prevents assignment to `prop` on any value typed with the interface. The constraint is **compile-time only** and is **shallow**: it prevents reassignment of the property binding, not mutation of the referenced object. `readonly` can be combined with `?` for optional readonly properties.

**Beginner-Friendly Explanation.** A readonly property is **data that can be set once but not changed afterward**. `readonly id: string` means the object must provide an `id`, but once the object exists, you cannot assign a new value to it.

### Purposes

- **To enforce immutability of identity** (IDs, creation timestamps).
- **To document intent** that a property should not change after creation.
- **To prevent accidental reassignment** of configuration or reference values.
- **To support immutable data modeling** in functional and reactive architectures.

### Syntax Rules and Structure

**General syntax:**

```
readonly propertyName: Type;
```

**Component breakdown:**

- `readonly` — modifier.
- `propertyName` — identifier.
- `: Type` — property type.

**Syntax rules.**

- `readonly` may be combined with `?`: `readonly prop?: Type`.
- `readonly` applies to the property binding, not the value’s internal mutability.
- Arrays and objects typed as `readonly` can still be mutated internally unless their types are also read-only (`readonly T[]`, `ReadonlyArray<T>`).

**Constraints and limitations.**

- `readonly` is erased at runtime; it does not enforce immutability in JavaScript.
- `readonly` is shallow; `readonly arr: number[]` still allows `arr.push(1)`.
- `readonly` cannot be applied to methods.

### Annotated Complete Step by Step Code Examples

#### Example 1: Readonly property prevents reassignment

```typescript
// example_readonly_prop.ts
interface Point {
  readonly x: number;
  readonly y: number;
}

const p: Point = { x: 1, y: 2 };
console.log(p.x); // Expected: 1

// p.x = 10; // Error: Cannot assign to 'x' because it is a read-only property.
```

**Expected Output:**
```
1
```

**Why this output occurs.** `x` and `y` are readonly. The object literal initializes them; subsequent assignment is rejected at compile time.

---

#### Example 2: Readonly array property (shallow limitation)

```typescript
// example_readonly_shallow.ts
interface Container {
  readonly items: string[];
}

const c: Container = { items: [] };
c.items.push("a"); // Allowed: mutating the array, not reassigning the property.
console.log(c.items); // Expected: [ 'a' ]

// c.items = []; // Error: Cannot assign to 'items' because it is a read-only property.
```

**Expected Output:**
```
[ 'a' ]
```

**Why this output occurs.** `readonly` prevents reassignment of `items`, but the array itself remains mutable. For true read-only arrays, the type should be `readonly string[]` or `ReadonlyArray<string>`.

---

### Real-World Cases

- **Entity IDs**: `readonly id: string` assigned once.
- **Configuration values**: `readonly apiUrl: string`.
- **Immutable value objects**: `readonly x`, `readonly y` in `Point`.

---

## 7. Index Signatures

### Definitions

**Core Definition.** An **index signature** is a type-level declaration of the form `[key: KeyType]: ValueType` that permits an object type to be indexed with keys of `KeyType`, returning values of `ValueType`. It enables static typing of objects whose property names are not known at compile time.

**Technical Definition.** Index signatures may appear in interfaces and type aliases. The key type may be `string`, `number`, or `symbol` (and, since TypeScript 4.4, template literal types such as `` `data-${string}` ``). When both string and number index signatures are present, the number-indexed value type must be assignable to the string-indexed value type. Explicitly declared properties must be assignable to the index signature’s value type. Indexed accesses return `ValueType`; under `noUncheckedIndexedAccess`, they return `ValueType | undefined`.

**Beginner-Friendly Explanation.** An index signature is **a way to tell TypeScript “this object is a dictionary.”** You say: “Any string key is allowed, and every value is a `string`.” Then TypeScript lets you write `obj["anyKey"]` and knows the result is a `string`. It is useful when the keys are dynamic but the values follow a consistent type.

### Purposes

- **To type dynamically keyed objects** without falling back to `any`.
- **To enable safe bracket-notation access** on objects whose keys are not known in advance.
- **To model dictionaries, caches, and configuration maps** with variable keys.
- **To combine known and unknown properties** in a single type.

### Syntax Rules and Structure

**General syntax:**

```
interface TypeName {
  [key: KeyType]: ValueType;
}
```

**Component breakdown:**

- `[key` — opens the index signature.
- `: KeyType` — the permitted key type.
- `]: ValueType` — the value type returned for any key.
- `;` — terminates the declaration.

**Syntax rules.**

- The key type must be `string`, `number`, `symbol`, or a template literal type.
- Only **one** index signature per key type is allowed.
- Explicit properties must be assignable to `ValueType`.
- `readonly` may be applied to index signatures.

**Constraints and limitations.**

- Index signatures assign the **same value type** to every key; per-key types require mapped types or `Record` with literal unions.
- Index signatures **do not require any specific keys**; an empty object satisfies them.
- Index signatures **do not catch typos**; `obj.nmae` compiles if `"nmae"` is a valid string key.
- Under `noUncheckedIndexedAccess`, indexed access returns `ValueType | undefined`.

### Annotated Complete Step by Step Code Examples

#### Example 1: String index signature

```typescript
// example_index_string.ts
interface ConfigSettings {
  [settingName: string]: string;
}

const appConfig: ConfigSettings = {
  theme: "dark",
  language: "en-US",
};

console.log(appConfig.theme); // Expected: dark

// appConfig.maxRetries = 5; // Error: number not assignable to string.
```

**Expected Output:**
```
dark
```

**Why this output occurs.** The index signature permits any string key with a string value. The object literal satisfies the constraint. Assigning a number to a string-indexed property is rejected.

---

#### Example 2: Number index signature

```typescript
// example_index_number.ts
interface ErrorMessages {
  [statusCode: number]: string;
}

const errors: ErrorMessages = {
  404: "Not Found",
  500: "Internal Server Error",
};

console.log(errors[404]); // Expected: Not Found
```

**Expected Output:**
```
Not Found
```

**Why this output occurs.** The number index signature allows numeric keys. TypeScript tracks them as numeric indices. The value type is `string`, so `errors[404]` is a `string`.

---

#### Example 3: Template literal index signature

```typescript
// example_index_template.ts
interface DataAttributes {
  [key: `data-${string}`]: string;
}

const element: DataAttributes = {
  "data-id": "123",
  "data-testid": "submit",
};

console.log(element["data-id"]); // Expected: 123
```

**Expected Output:**
```
123
```

**Why this output occurs.** The template literal type constrains keys to the `data-` prefix. Keys outside the pattern are rejected.

---

### Real-World Cases

- **Configuration maps**: `interface EnvConfig { [key: string]: string | undefined }`.
- **API response caches**: `interface ResponseCache { [endpoint: string]: unknown }`.
- **CSS custom properties**: `interface ThemeVars { [key: \`--${string}\`]: string }`.

---

## 8. Method Signatures

### Definitions

**Core Definition.** A **method signature** is an interface member that declares a method’s name, parameter types, and return type, specifying that conforming objects must provide a callable member with that signature.

**Technical Definition.** A method signature has the form `methodName(params: Type): ReturnType`. It may be optional (`methodName?(): void`), generic (`method<T>(arg: T): T`), or overloaded (multiple signatures followed by a compatible implementation). TypeScript provides two syntactic forms: **method shorthand** (`greet(name: string): string`) and **property function type** (`greet: (name: string) => string`). The method shorthand form is bivariant in parameter checking under default settings; the property form is contravariant under `strictFunctionTypes`.

**Beginner-Friendly Explanation.** A method signature is **a required action that an object must be able to perform**. `greet(name: string): string` means the object must have a `greet` method that takes a string and returns a string.

### Purposes

- **To require specific behavior** from conforming objects.
- **To document method parameter and return types** for tooling and safety.
- **To support optional methods** when behavior is conditionally required.
- **To enable polymorphic calls** through interface-typed references.

### Syntax Rules and Structure

**General syntax (method shorthand):**

```
methodName(params: Type): ReturnType;
```

**General syntax (property function type):**

```
methodName: (params: Type) => ReturnType;
```

**Component breakdown:**

- `methodName` — the method name.
- `(params: Type)` — parameter list.
- `: ReturnType` — return type.

**Syntax rules.**

- Methods may be optional: `methodName?(): void`.
- Methods may be generic: `map<T>(fn: (x: T) => T): T`.
- Methods may be overloaded with multiple signatures.
- The method shorthand form is generally preferred for interfaces unless `strictFunctionTypes` requires property-style variance.

**Constraints and limitations.**

- Method parameters are **bivariant** by default under the method shorthand form, which is unsound but matches JavaScript patterns.
- The property function type form is **contravariant** under `strictFunctionTypes` and is safer.
- Method signatures cannot declare `private` or `protected` visibility.

### Annotated Complete Step by Step Code Examples

#### Example 1: Method shorthand

```typescript
// example_method_shorthand.ts
interface Greeter {
  greet(name: string): string;
}

const greeter: Greeter = {
  greet(name) {
    return `Hello, ${name}`;
  },
};

console.log(greeter.greet("Ada")); // Expected: Hello, Ada
```

**Expected Output:**
```
Hello, Ada
```

**Why this output occurs.** The `Greeter` interface requires a `greet` method taking a string and returning a string. The object literal provides a compatible method.

---

#### Example 2: Property function type

```typescript
// example_method_property.ts
interface Validator {
  validate: (value: string) => boolean;
}

const validator: Validator = {
  validate: (value) => value.length > 0,
};

console.log(validator.validate("hello")); // Expected: true
```

**Expected Output:**
```
true
```

**Why this output occurs.** The property function type form declares `validate` as a property whose value is a function. The object provides a compatible function. This form is contravariant under `strictFunctionTypes`.

---

### Real-World Cases

- **Service interfaces**: `interface ILogger { log(message: string): void; }`.
- **Validation contracts**: `interface Validator { validate(value: unknown): boolean; }`.
- **Event handlers**: `interface Handler { handle(event: Event): void; }`.

---

## 9. Call Signatures

### Definitions

**Core Definition.** A **call signature** is an interface member that declares the interface itself to be **callable as a function**, specifying the parameter types and return type of the call. It has the form `(params: Type): ReturnType` inside the interface body.

**Technical Definition.** A call signature makes the interface type a **function type with additional properties**. It is declared without a member name. An interface may have **multiple call signatures**, which act as overloads. The call signature syntax uses `:` between the parameter list and return type, unlike function type expressions which use `=>`.

**Beginner-Friendly Explanation.** A call signature is **a way to say “objects of this type can be called like functions.”** For example, an interface with a call signature describes something that is both a function and has extra properties.

### Purposes

- **To describe callable objects** that also have properties.
- **To type functions with attached metadata or methods**.
- **To support multiple call signatures** (overloads) in a single type.
- **To model JavaScript APIs** where functions have properties.

### Syntax Rules and Structure

**General syntax:**

```
interface Callable {
  (param: Type): ReturnType;
  property: Type;
}
```

**Component breakdown:**

- `(param: Type): ReturnType` — the call signature.
- `property: Type` — additional properties on the callable object.

**Syntax rules.**

- The call signature has no member name.
- Multiple call signatures may be declared for overloads.
- Call signatures can be combined with property, method, and index signatures.
- Call signatures are not implemented by classes; they describe existing callable values.

**Constraints and limitations.**

- A type with only a single call signature is often better expressed as a **function type** (`(param: Type) => ReturnType`); ESLint’s `prefer-function-type` rule recommends this.
- Call signatures are erased and have no runtime representation.
- Classes cannot implement call signatures.

### Annotated Complete Step by Step Code Examples

#### Example 1: Callable with properties

```typescript
// example_call_signature.ts
interface DescribableFunction {
  description: string;
  (someArg: number): boolean;
}

function doSomething(fn: DescribableFunction) {
  console.log(`${fn.description} returned ${fn(6)}`);
}

const fn: DescribableFunction = Object.assign(
  (n: number) => n > 5,
  { description: "greaterThanFive" }
);

doSomething(fn); // Expected: greaterThanFive returned true
```

**Expected Output:**
```
greaterThanFive returned true
```

**Why this output occurs.** The interface declares both a call signature and a `description` property. The function is created with `Object.assign`, combining a callable with a property. The interface type checks both aspects.

---

#### Example 2: Multiple call signatures (overloads)

```typescript
// example_call_signature_overload.ts
interface StringOrNumberParser {
  (input: string): number;
  (input: number): string;
}

const parse: StringOrNumberParser = (input: string | number): any => {
  return typeof input === "string" ? Number(input) : String(input);
};

console.log(parse("42")); // Expected: 42
console.log(parse(42));   // Expected: "42"
```

**Expected Output:**
```
42
"42"
```

**Why this output occurs.** The interface declares two call signatures, acting as overloads. The implementation handles both cases. The compiler checks calls against the overloads.

---

### Real-World Cases

- **Function objects with metadata**: libraries that attach properties to callable functions.
- **Overloaded APIs**: `interface Parser { (input: string): number; (input: number): string; }`.
- **Hybrid types**: jQuery-style APIs that are both callable and have methods.

---

## 10. Construct Signatures

### Definitions

**Core Definition.** A **construct signature** is an interface member that declares the interface to be **constructable with `new`**, specifying the constructor’s parameter types and the type of the instance produced. It has the form `new (params: Type): InstanceType`.

**Technical Definition.** A construct signature adds a `new`-able signature to an interface. It is used to describe **constructor functions** and classes as values. Construct signatures can be combined with call signatures to model functions that can be called both with and without `new`. They are **not implementable by classes**; they describe existing constructor values.

**Beginner-Friendly Explanation.** A construct signature is **a way to say “this type is a class or constructor function.”** It describes what you get when you use `new` on the value.

### Purposes

- **To type constructor functions** and class values passed as arguments.
- **To describe factory functions** that return instances.
- **To combine call and construct behavior** in hybrid types.
- **To enable dependency injection** with constructor types.

### Syntax Rules and Structure

**General syntax:**

```
interface Constructor {
  new (param: Type): InstanceType;
}
```

**Component breakdown:**

- `new` — construct signature keyword.
- `(param: Type)` — constructor parameters.
- `: InstanceType` — the instance type produced by `new`.

**Syntax rules.**

- Construct signatures may be combined with call signatures: `interface CallOrConstruct { new (s: string): Date; (n?: number): number; }`.
- Multiple construct signatures may be declared for overloads.
- Construct signatures are not implemented by classes; use them to type class values.

**Constraints and limitations.**

- Classes cannot `implement` a construct signature; they satisfy it by being constructor values.
- Abstract classes have abstract construct signatures and cannot be used with `new` directly.
- Construct signatures are erased at runtime.

### Annotated Complete Step by Step Code Examples

#### Example 1: Construct signature for a class value

```typescript
// example_construct_signature.ts
interface PointConstructor {
  new (x: number, y: number): { x: number; y: number };
}

class Point {
  constructor(public x: number, public y: number) {}
}

function createPoint(Ctor: PointConstructor, x: number, y: number) {
  return new Ctor(x, y);
}

const p = createPoint(Point, 1, 2);
console.log(p.x, p.y); // Expected: 1 2
```

**Expected Output:**
```
1 2
```

**Why this output occurs.** The `PointConstructor` interface has a construct signature. The `Point` class value satisfies it because `new Point(x, y)` produces the required instance shape. The factory function accepts any conforming constructor.

---

#### Example 2: Combined call and construct signatures

```typescript
// example_construct_call.ts
interface CallOrConstruct {
  new (s: string): Date;
  (n?: number): number;
}

function fn(input?: string | number): Date | number {
  if (typeof input === "string") return new Date(input);
  return input ?? 0;
}

const hybrid: CallOrConstruct = fn as CallOrConstruct;

const date = new hybrid("2025-01-01");
const num = hybrid(42);

console.log(date instanceof Date); // Expected: true
console.log(num);                  // Expected: 42
```

**Expected Output:**
```
true
42
```

**Why this output occurs.** The interface combines a construct signature and a call signature. The function is asserted to the hybrid type and used both with `new` and as a normal call. The compiler checks both usage forms.

---

### Real-World Cases

- **Factory functions**: `interface UserFactory { new (name: string): User; }`.
- **Dependency injection**: constructor types for service instantiation.
- **Hybrid libraries**: functions that can be called with or without `new`.

---

## 11. Interface Inheritance

### Definitions

**Core Definition.** **Interface inheritance** is the mechanism by which an interface acquires the members of one or more base interfaces via the `extends` keyword, composing their contracts into a new named interface.

**Technical Definition.** An interface may extend zero or more base interfaces (or class types). The derived interface inherits all members of its bases and may add new members or redeclare inherited ones. Inherited properties with the same name must be **identical** in type unless the derived interface redeclares them with a subtype-compatible type. An interface may not extend itself directly or indirectly. Multiple inheritance is supported for interfaces.

**Beginner-Friendly Explanation.** Interface inheritance is **one interface building on another**. `interface Employee extends Person` means “an `Employee` is also a `Person` and must satisfy both contracts.”

### Purposes

- **To reuse contracts** without duplicating member declarations.
- **To compose complex contracts** from smaller, focused ones.
- **To model “is-a” relationships** between object shapes.
- **To enable multiple inheritance of contracts** (unlike classes).

### Syntax Rules and Structure

**General syntax:**

```
interface Derived extends Base1, Base2 {
  // additional members
}
```

**Component breakdown:**

- `extends` — heritage keyword.
- `Base1, Base2` — comma-separated base interfaces.
- `{ ... }` — additional members.

**Syntax rules.**

- An interface may extend multiple interfaces.
- Inherited members with identical names and types are merged; duplicates are allowed.
- Inherited members with the same name but different types must be redeclared with a type that is a subtype of both.
- An interface cannot extend itself directly or indirectly.
- All required properties from all bases must be satisfied by conforming objects.

**Constraints and limitations.**

- Conflicting property types without redeclaration cause compile errors.
- An interface cannot extend a class with private or protected members unless the interface redeclares those members with compatible accessibility (generally not possible for private).
- Base types must be class or interface types.

### Annotated Complete Step by Step Code Examples

#### Example 1: Single inheritance

```typescript
// example_interface_single_inherit.ts
interface Shape {
  color: string;
}

interface Square extends Shape {
  sideLength: number;
}

const square: Square = { color: "red", sideLength: 10 };
console.log(square.color);      // Expected: red
console.log(square.sideLength); // Expected: 10
```

**Expected Output:**
```
red
10
```

**Why this output occurs.** `Square` extends `Shape`, inheriting `color`. The object literal must satisfy both `color` and `sideLength`.

---

#### Example 2: Conflicting inherited properties require redeclaration

```typescript
// example_interface_conflict.ts
interface Mover {
  getStatus(): { speed: number };
}

interface Shaker {
  getStatus(): { frequency: number };
}

// Must redeclare getStatus with a type that is a subtype of both.
interface MoverShaker extends Mover, Shaker {
  getStatus(): { speed: number; frequency: number };
}

const ms: MoverShaker = {
  getStatus: () => ({ speed: 10, frequency: 5 }),
};

console.log(ms.getStatus().speed);     // Expected: 10
console.log(ms.getStatus().frequency); // Expected: 5
```

**Expected Output:**
```
10
5
```

**Why this output occurs.** `Mover` and `Shaker` both declare `getStatus` with different return types. `MoverShaker` redeclares it with a return type that is a subtype of both, resolving the conflict. Without redeclaration, the compiler reports an error.

---

### Real-World Cases

- **Entity hierarchies**: `interface Employee extends Person { salary: number; }`.
- **Layered API models**: `interface DetailedUser extends User { lastLogin: Date; }`.
- **Composed contracts**: combining `Serializable` and `Timestamped` into `PersistedEntity`.

---

## 12. Extending Interfaces

### Definitions

**Core Definition.** **Extending an interface** is the act of using the `extends` keyword to create a new interface that inherits the members of one or more base interfaces, optionally adding or redeclaring members.

**Technical Definition.** The `extends` clause on an interface establishes a **contract composition** relationship. The derived interface’s instance type is a subtype of each base interface’s type. Members are inherited unless hidden by declarations in the derived interface. The compiler verifies that the derived interface is assignable to each base.

**Beginner-Friendly Explanation.** Extending an interface means **creating a new interface that includes everything from the old one, plus more**. It is a way to build richer contracts from simpler ones.

### Purposes

- **To compose contracts** from smaller, reusable pieces.
- **To add members** to an existing contract without modifying it.
- **To model specialization** in object shapes.
- **To promote contract reuse** across a codebase.

### Syntax Rules and Structure

**General syntax:**

```
interface Derived extends Base {
  // new members
}
```

**Component breakdown:**

- `extends Base` — heritage clause.
- `{ ... }` — new members.

**Syntax rules.**

- The base may be an interface or a class type.
- The derived interface inherits all non-hidden members.
- New members may be added.
- Inherited members may be redeclared with compatible types.
- The derived interface must be assignable to the base.

**Constraints and limitations.**

- A class with `private` or `protected` members cannot be fully extended by an interface.
- Redeclared members must be subtype-compatible.
- Extending a base with an index signature constrains the derived interface’s explicit properties.

### Annotated Complete Step by Step Code Examples

#### Example 1: Extending an interface with new members

```typescript
// example_extend_interface.ts
interface Person {
  name: string;
  age: number;
}

interface Employee extends Person {
  salary: number;
}

const emp: Employee = {
  name: "Ada",
  age: 36,
  salary: 100000,
};

console.log(emp.salary); // Expected: 100000
```

**Expected Output:**
```
100000
```

**Why this output occurs.** `Employee` extends `Person`, inheriting `name` and `age` and adding `salary`. The object literal satisfies all three properties.

---

#### Example 2: Extending a class type

```typescript
// example_extend_class.ts
class Control {
  state: string = "idle";
}

interface SelectableControl extends Control {
  select(): void;
}

const button: SelectableControl = {
  state: "ready",
  select() {
    console.log("selected");
  },
};

console.log(button.state); // Expected: ready
```

**Expected Output:**
```
ready
```

**Why this output occurs.** `SelectableControl` extends the `Control` class type, inheriting `state`. The object provides both `state` and `select`. Interfaces can extend class types that have only public members.

---

### Real-World Cases

- **Domain specialization**: `interface Admin extends User { permissions: string[]; }`.
- **Framework contracts**: extending base component props with additional fields.
- **API versioning**: `interface UserV2 extends UserV1 { newField: string; }`.

---

## 13. Multiple Inheritance (Multiple `extends`)

### Definitions

**Core Definition.** **Multiple inheritance** for interfaces is the capability of an interface to **extend two or more base interfaces simultaneously**, committing to satisfy the combined contract of all bases. TypeScript supports multiple interface inheritance, unlike multiple class inheritance.

**Technical Definition.** An interface’s `extends` clause may list multiple base types: `interface C extends A, B { ... }`. The derived interface must be assignable to each base. If bases declare the same member with **identical** types, the member is inherited once. If they declare the same member with **different** types, the derived interface must **redeclare** the member with a type that is a subtype of both.

**Beginner-Friendly Explanation.** Multiple inheritance means **one interface can build on several other interfaces at once**. `interface Manager extends Person, Employee` means “a `Manager` must satisfy both `Person` and `Employee`.”

### Purposes

- **To compose complex contracts** from multiple focused bases.
- **To avoid bloated single interfaces** by combining smaller ones.
- **To model objects that participate in multiple conceptual categories**.
- **To enable flexible contract reuse** across unrelated base types.

### Syntax Rules and Structure

**General syntax:**

```
interface Derived extends Base1, Base2, Base3 {
  // members
}
```

**Component breakdown:**

- `extends` — heritage keyword.
- `Base1, Base2, Base3` — comma-separated base types.
- `{ ... }` — additional members.

**Syntax rules.**

- Any number of base types may be listed.
- All required members of all bases must be satisfied.
- Identical inherited members are merged.
- Conflicting members must be redeclared with a subtype-compatible type.
- The derived interface must be assignable to each base.

**Constraints and limitations.**

- Conflicting members without redeclaration cause compile errors.
- The redeclared type must be a subtype of every conflicting base member type.
- Interfaces can extend multiple class types only if those classes have compatible visibility (no conflicting private/protected members).

### Annotated Complete Step by Step Code Examples

#### Example 1: Multiple interface extension

```typescript
// example_multiple_extends.ts
interface Person {
  name: string;
  age: number;
}

interface Employee {
  salary: number;
}

interface Manager extends Person, Employee {
  reports: string[];
}

const manager: Manager = {
  name: "Ada",
  age: 36,
  salary: 120000,
  reports: ["Grace", "Alan"],
};

console.log(manager.name);           // Expected: Ada
console.log(manager.reports.length); // Expected: 2
```

**Expected Output:**
```
Ada
2
```

**Why this output occurs.** `Manager` extends both `Person` and `Employee`, inheriting `name`, `age`, and `salary`, and adding `reports`. The object satisfies all four properties.

---

#### Example 2: Resolving conflicting members

```typescript
// example_multiple_conflict.ts
interface A {
  value: string;
}

interface B {
  value: string | number;
}

// A’s value type is a subtype of B’s, so no redeclaration is needed.
interface C extends A, B {
  extra: boolean;
}

const c: C = { value: "hello", extra: true };
console.log(c.value); // Expected: hello
```

**Expected Output:**
```
hello
```

**Why this output occurs.** `A.value` is `string`, which is assignable to `B.value` (`string | number`). The inherited members are compatible, so `C` inherits `value` without redeclaration. If the types were incompatible, `C` would need to redeclare `value` with a subtype of both.

---

### Real-World Cases

- **Cross-cutting domain models**: `interface Admin extends User, Auditable, Timestamped { ... }`.
- **UI component contracts**: combining `Clickable`, `Focusable`, and `Disableable`.
- **API composition**: merging `Paginated`, `Filterable`, and `Sortable` into query interfaces.

---

## 14. Interface Best Practices

### Definitions

**Core Definition.** **Interface best practices** are guidelines for designing and using interfaces in ways that maximize **clarity, type safety, extensibility, and maintainability**, while avoiding pitfalls such as bloated contracts, overuse of index signatures, and confusion between interfaces and type aliases.

**Technical Definition.** Best practices encompass **preferring interfaces for public object contracts** (they produce clearer errors, support declaration merging, and are recommended by the TypeScript Handbook), **preferring `type` for unions, intersections, and mapped/conditional types**, **using composition over deep inheritance**, **keeping interfaces focused and small** (Interface Segregation Principle), **avoiding empty interfaces**, **not prefixing interfaces with `I`** (Google Style Guide and TypeScript Handbook recommend against it), **using `readonly` for immutable properties**, **preferring function types over single-call-signature interfaces** (ESLint `prefer-function-type`), and **avoiding index signatures when keys are known**.

**Beginner-Friendly Explanation.** Best practices are **the habits experienced TypeScript developers use** to keep interfaces clean and useful. The biggest rules are: use `interface` for object shapes and `type` for unions and advanced types; keep interfaces small and focused; do not prefix interface names with `I`; prefer `readonly` for things that should not change; and do not use an index signature when you know the property names.

### Purposes

- **To choose the right tool** between `interface` and `type`.
- **To keep contracts focused** through interface segregation.
- **To improve error messages and tooling** by using interfaces for public object shapes.
- **To avoid anti-patterns** such as empty interfaces and I-prefixed names.
- **To maximize type safety** with `readonly`, precise function types, and narrow index signatures.

### Syntax Rules and Structure

Best practices are conventions that interact with interface syntax. The core rules are:

1. **Prefer `interface` for public object contracts.** Interfaces produce clearer compiler errors, support declaration merging, and are recommended by the TypeScript Handbook. Use `type` for unions, intersections, primitives, tuples, and mapped/conditional types.
2. **Do not prefix interfaces with `I`.** `interface User` is preferred over `interface IUser`. Both the TypeScript Handbook and Google Style Guide recommend against the `I` prefix.
3. **Keep interfaces small and focused.** Apply the Interface Segregation Principle: many small interfaces are better than one large one.
4. **Use composition over deep inheritance.** Prefer extending multiple focused interfaces or using intersection types over building deep `extends` chains.
5. **Avoid empty interfaces.** An empty interface (`interface Foo {}`) is equivalent to `{}` and can often be replaced by `object` or a more precise type.
6. **Prefer function types over single-call-signature interfaces.** ESLint’s `prefer-function-type` rule recommends `(x: string) => number` over `interface Fn { (x: string): number }` for single-signature callables.
7. **Use `readonly` for immutable properties.** Mark properties that should not be reassigned as `readonly`.
8. **Avoid index signatures when keys are known.** Use interfaces or `Record` with literal unions for known key sets.
9. **Use optional properties (`?`) for genuinely optional data.** Do not make properties required if they are not always present.

**Component breakdown (interface vs. type decision tree):**

```
Is the type a pure object shape used as a public contract?
  ├── Yes → Prefer interface.
  │        ├── Does it need to be extended or merged? → interface.
  │        └── Is it a single call signature? → Consider a function type.
  └── No → Is it a union, intersection, primitive, tuple, or mapped/conditional type?
           ├── Yes → Use type.
           └── No → Re-evaluate; interface is usually the better choice for objects.
```

**Constraints and limitations.**

- Interfaces cannot express unions, intersections, or mapped types directly.
- Declaration merging can be surprising; use `type` when merging is undesirable.
- `readonly` is shallow; it does not deep-freeze objects.
- Optional properties shift `undefined` handling to consumers.

### Annotated Complete Step by Step Code Examples

#### Example 1: Interface vs. type — choosing the right tool

```typescript
// example_best_interface_vs_type.ts
// ✅ Interface for an object contract.
interface User {
  id: string;
  name: string;
}

// ✅ Type for a union.
type UserRole = "admin" | "editor" | "viewer";

// ✅ Type for an intersection.
type AdminUser = User & { role: "admin" };

// ✅ Type for a mapped type.
type PartialUser = Partial<User>;

const user: User = { id: "u1", name: "Ada" };
const role: UserRole = "admin";

console.log(user.name); // Expected: Ada
console.log(role);      // Expected: admin
```

**Expected Output:**
```
Ada
admin
```

**Why this output occurs.** The object contract uses an interface; the union, intersection, and mapped types use `type`. This follows the TypeScript Handbook’s recommendation: “Use `interface` until you need to use `type`.”

---

#### Example 2: Avoiding the `I` prefix and empty interfaces

```typescript
// example_best_naming.ts
// ❌ Avoid: I-prefixed interface.
// interface IUser { name: string; }

// ✅ Correct: no prefix.
interface User {
  name: string;
}

// ❌ Avoid: empty interface.
// interface Empty {}

// ✅ Correct: use a precise type or `object` when appropriate.
type AnyObject = Record<string, unknown>;

const user: User = { name: "Ada" };
console.log(user.name); // Expected: Ada
```

**Expected Output:**
```
Ada
```

**Why this output occurs.** The interface is named `User`, not `IUser`, following the Google Style Guide and TypeScript Handbook. The empty interface is replaced with a precise type. These conventions improve readability and align with community standards.

---

#### Example 3: Interface segregation — small focused interfaces

```typescript
// example_best_segregation.ts
// ❌ Bloated interface.
// interface Worker {
//   work(): void;
//   eat(): void;
//   sleep(): void;
// }

// ✅ Segregated interfaces.
interface Workable {
  work(): void;
}

interface Eatable {
  eat(): void;
}

class Robot implements Workable {
  work() { console.log("Robot working"); }
}

class Human implements Workable, Eatable {
  work() { console.log("Human working"); }
  eat() { console.log("Human eating"); }
}

new Robot().work(); // Expected: Robot working
new Human().eat();  // Expected: Human eating
```

**Expected Output:**
```
Robot working
Human eating
```

**Why this output occurs.** The segregated interfaces allow `Robot` to implement only `Workable` without being forced to implement unrelated methods. `Human` implements both focused interfaces. This follows the Interface Segregation Principle.

---

### Real-World Cases

- **Public library APIs**: interfaces provide stable, mergeable contracts with clear error messages.
- **SOLID design**: interface segregation keeps contracts focused and implementation-agnostic.
- **Dependency injection**: services depend on narrow interfaces rather than broad concrete classes.
- **Testing**: mocks implement only the small interfaces they need, reducing test setup.

---

## References

- TypeScript Handbook — Object Types (Interfaces, Property Modifiers, Index Signatures, Call/Construct Signatures) - https://www.typescriptlang.org/docs/handbook/2/objects.html
- TypeScript Handbook — Interfaces (Legacy) - https://www.typescriptlang.org/docs/handbook/interfaces.html
- TypeScript Handbook — Type Compatibility (Structural Typing) - https://www.typescriptlang.org/docs/handbook/type-compatibility.html
- TypeScript Handbook — Everyday Types (Interfaces vs. Type Aliases) - https://www.typescriptlang.org/docs/handbook/2/everyday-types.html
- TypeScript Language Specification — Interfaces - https://chromium.googlesource.com/external/github.com/kythe/kythe/+/7c2c91248ad9bc8d22dfaa00764c6381af512113/third_party/typescript/doc/spec.md
- TypeScript Handbook — More on Functions (Call Signatures, Construct Signatures) - https://www.typescriptlang.org/docs/handbook/2/functions.html
- Microsoft Learn — Extend an Interface in TypeScript - https://learn.microsoft.com/en-us/training/modules/typescript-implement-interfaces/4-extend-interface
- Google TypeScript Style Guide — Naming and Interfaces - https://google.github.io/styleguide/tsguide.html
- Contentful — TypeScript Interfaces: Best Practices - https://www.contentful.com/blog/typescript-interfaces/
- Convex — TypeScript Interfaces: Declaration, Optional, Readonly, Index Signatures - https://www.convex.dev/typescript/core-concepts/object-oriented-programming/typescript-interface
- Educative — A Simple Guide to TypeScript Interfaces - https://www.educative.io/blog/typescript-interfaces
- TypeScript ESLint — `prefer-function-type` Rule - https://typescript-eslint.io/rules/prefer-function-type/
- TypeScript ESLint — `consistent-indexed-object-style` Rule - https://typescript-eslint.io/rules/consistent-indexed-object-style/
- Microsoft/TypeScript Issue #7633 — Index Signatures and `implements` Clauses - https://github.com/microsoft/TypeScript/issues/7633
- TSConfig Reference — `noUncheckedIndexedAccess` - https://www.typescriptlang.org/tsconfig/noUncheckedIndexedAccess.html
- TypeScript 4.4 Release Notes — Template Literal Types in Index Signatures - https://www.typescriptlang.org/docs/handbook/release-notes/typescript-4-4.html