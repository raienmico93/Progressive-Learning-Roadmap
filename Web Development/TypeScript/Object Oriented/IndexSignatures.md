# Index Signatures in Classes — Comprehensive Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition (Academic).** An **index signature** is a TypeScript type-level construct that declares a class or interface to be **indexable by a family of keys** of a specified type, mapping each such key to a value of a specified type. In a class, an index signature declares that instances of the class may be accessed with **arbitrary keys** of a given type, beyond any explicitly declared members, and that all such accesses yield values of a declared type. Index signatures are the primary mechanism for typing **dynamically keyed objects** while retaining static type safety.

**Technical Definition.** An index signature has the form `[key: KeyType]: ValueType`, where `KeyType` is `string`, `number`, or `symbol` (or, since TypeScript 4.4, a template literal type or a union of these). When declared on a class, it becomes part of the class's **instance type**, allowing bracket-notation access with keys of `KeyType`. The compiler enforces that **explicitly declared properties are assignable to the index signature's value type**, and that index-signature accesses return `ValueType` (without automatically adding `undefined` unless `noUncheckedIndexedAccess` is enabled). A class that `implements` an interface with an index signature **must redeclare the index signature in its own body**; the `implements` clause alone does not install the signature on the class instance type. Index signatures are **erased** during compilation and have no runtime representation.

**Beginner-Friendly Explanation.** An index signature is **a rule that says “this object can be indexed with any key of a certain kind, and every such key gives you a value of a certain type.”** Think of a dictionary: you do not know every word in advance, but you know that looking up any word gives you a definition (a string). An index signature on a class says: “You can use bracket notation with any string key on instances of this class, and you will get back a value of this type.” It lets TypeScript check dynamic property access that would otherwise be `any` and lose all safety.

---

### Key Characteristics

| Characteristic | Description |
|---|---|
| **Dynamic key support** | Allows arbitrary keys of type `string`, `number`, or `symbol` (or template-literal patterns). |
| **Value-type guarantee** | Every indexed access returns the declared `ValueType`, providing static type safety for dynamic lookups. |
| **Explicit-property compatibility** | Explicitly declared properties must be assignable to the index signature's value type. |
| **Class redeclaration requirement** | A class implementing an interface with an index signature must **redeclare** the signature in its own body; `implements` alone is insufficient. |
| **`readonly` support** | Index signatures may be `readonly`, preventing assignment through indexed access. |
| **No distinct types per key** | An index signature assigns the **same** value type to all keys; per-key types require mapped types or `Record` with literal unions. |
| **Erased at runtime** | Index signatures exist only at compile time; they emit no JavaScript. |

---

### Prerequisites

- **TypeScript basics**: interfaces, type aliases, and type annotations.
- **Classes**: fields, methods, constructors, and `implements`.
- **Bracket notation** and the distinction between dot notation (`obj.prop`) and indexed access (`obj["prop"]`).
- **Utility types**: `Record<K, V>` as an alternative to index signatures.
- **A TypeScript environment** (`tsc` or an editor with TypeScript support) to observe compiler behavior.

---

### Related Programming Areas

- **Dictionary and map typing** in TypeScript.
- **Dynamic configuration objects** and API response shapes.
- **Template literal types** (TypeScript 4.4+) for constrained key patterns.
- **Mapped types** and `Record<K, V>` as more precise alternatives.
- **`Map` and `Set`** for truly dynamic runtime key-value storage.

---

### Core Concepts / Features

The following sections cover each requested concept. Each follows the uniform structure.

---

## 1. Index Signatures

### Definitions

**Core Definition.** An **index signature** is a type-level declaration of the form `[key: KeyType]: ValueType` that permits an object type to be indexed with keys of `KeyType`, returning values of `ValueType`. It enables static typing of objects whose property names are not known at compile time.

**Technical Definition.** Index signatures may appear in interfaces, type aliases, and class bodies. The key type may be `string`, `number`, or `symbol` (and, since TypeScript 4.4, template literal types such as `` `data-${string}` ``). When both a string and a number index signature are present, the number-indexed value type must be **assignable to** the string-indexed value type, reflecting JavaScript's coercion of numeric keys to strings. Explicitly declared properties must be assignable to the index signature's value type; otherwise, the compiler reports an error. Indexed accesses return `ValueType`; under `--noUncheckedIndexedAccess`, they return `ValueType | undefined`, acknowledging that the key may not exist.

**Beginner-Friendly Explanation.** An index signature is **a way to tell TypeScript “this object is a dictionary.”** You say: “Any string key is allowed, and every value is a `string`.” Then TypeScript lets you write `obj["anyKey"]` and knows the result is a `string`. It is useful when the keys are dynamic but the values follow a consistent type.

### Purposes

- **To type dynamically keyed objects** without falling back to `any`.
- **To enable safe bracket-notation access** on objects whose keys are not known in advance.
- **To model dictionaries, caches, configuration maps, and API responses** with variable keys.
- **To combine known and unknown properties** in a single type, where known properties have specific types and additional properties follow the index signature.
- **To support template-literal-constrained keys** such as `data-*` attributes or event-handler names.

### Syntax Rules and Structure

**General syntax (interface or type alias):**

```ts
interface TypeName {
  [key: KeyType]: ValueType;
}
```

**Component breakdown:**

- `[key` — opens the index signature; `key` is an arbitrary parameter name.
- `: KeyType` — the permitted key type (`string`, `number`, `symbol`, or a template literal / union thereof).
- `]: ValueType` — the value type returned for any key of `KeyType`.
- `;` — terminates the declaration.

**Combined with explicit properties:**

```ts
interface UserProfile {
  id: string;
  email: string;
  [key: string]: string;
}
```

- `id` and `email` are explicit required properties.
- The index signature allows additional string keys, all with `string` values.
- Explicit property types must be assignable to `ValueType`.

**Number index signatures:**

```ts
interface StatusCodes {
  [code: number]: string;
}
```

- Numeric keys are coerced to strings at runtime, but TypeScript tracks them as `number` keys.

**Readonly index signatures:**

```ts
interface ReadonlyConfig {
  readonly [key: string]: string;
}
```

- `readonly` prevents assignment through indexed access.

**Template literal index signatures (TypeScript 4.4+):**

```ts
interface DataAttributes {
  [key: `data-${string}`]: string;
}
```

- Only keys matching the template literal pattern are permitted.

**Syntax rules.**

- The key type must be `string`, `number`, `symbol`, or a template literal type / union thereof.
- Only **one** index signature per key type is allowed in a type.
- When both string and number signatures exist, the number value type must be a subtype of the string value type.
- Explicit properties must be assignable to the index signature's value type.
- `readonly` may be applied to index signatures.
- Index signatures in a class must be declared in the class body; an `implements` clause does not install them on the instance type.

**Constraints and limitations.**

- Index signatures assign **the same value type to every key**; they cannot give different keys different types (use mapped types or `Record` with literal unions instead).
- Index signatures **do not require any specific keys**; an empty object satisfies an index signature.
- Index signatures **do not catch typos**; `obj.nmae` compiles if `"nmae"` is a valid string key, because the index signature allows any string.
- Under `noUncheckedIndexedAccess`, indexed access returns `ValueType | undefined`, but by default the compiler **pretends the key exists** even when it does not.
- Index signatures are **erased** and do not exist at runtime.

### Annotated Complete Step by Step Code Examples

#### Example 1: Basic string index signature

```typescript
// example_index_basic.ts
// Step 1: Declare an interface with a string index signature.
interface ConfigSettings {
  [settingName: string]: string;
}

// Step 2: Create an object with arbitrary string keys.
const appConfig: ConfigSettings = {
  theme: "dark",
  language: "en-US",
  timezone: "UTC",
};

// Step 3: Add new properties dynamically.
appConfig.newSetting = "value";

// Step 4: Access properties; TypeScript knows the value type.
const theme: string = appConfig.theme;
console.log(theme); // Expected: dark

// appConfig.maxRetries = 5;
// Error: Type 'number' is not assignable to type 'string'.
```

**Setup.** Compile with `npx tsc example_index_basic.ts --strict`.

**Expected Output:**
```
dark
```

**Why this output occurs.** The index signature `[settingName: string]: string` permits any string key and requires every value to be a `string`. The object literal is checked against this rule; `theme` is a valid string key with a string value. `appConfig.maxRetries = 5` is rejected because `5` is not a `string`. At runtime, the index signature is erased and the object behaves normally.

---

#### Example 2: Number index signature

```typescript
// example_index_number.ts
// Step 1: Declare a number-indexed interface.
interface ErrorMessages {
  [statusCode: number]: string;
}

// Step 2: Create an object with numeric keys.
const errors: ErrorMessages = {
  404: "Not Found",
  500: "Internal Server Error",
};

// Step 3: Access by numeric key.
console.log(errors[404]); // Expected: Not Found
```

**Expected Output:**
```
Not Found
```

**Why this output occurs.** The number index signature allows numeric keys. Although JavaScript coerces `404` to the string `"404"` at runtime, TypeScript tracks it as a numeric index. The value type is `string`, so `errors[404]` is a `string`.

---

#### Example 3: Combining known properties with an index signature

```typescript
// example_index_combined.ts
// Step 1: Declare an interface with known and dynamic properties.
interface UserProfile {
  id: string;
  email: string;
  [key: string]: string;
}

// Step 2: Create a conforming object.
const user: UserProfile = {
  id: "user-123",
  email: "user@example.com",
  department: "Engineering",
  location: "New York",
};

// Step 3: Access both known and dynamic properties.
console.log(user.email);       // Expected: user@example.com
console.log(user.department);  // Expected: Engineering
```

**Expected Output:**
```
user@example.com
Engineering
```

**Why this output occurs.** `id` and `email` are explicit required properties, and both are `string`, so they are assignable to the index signature's `string` value type. Additional keys like `department` and `location` are permitted by the index signature. This pattern lets you type objects with a fixed core and flexible extensions.

---

#### Example 4: Readonly index signature

```typescript
// example_index_readonly.ts
interface ReadonlyConfig {
  readonly [key: string]: string;
}

const config: ReadonlyConfig = {
  apiUrl: "https://api.example.com",
};

console.log(config.apiUrl); // Expected: https://api.example.com

// config.apiUrl = "https://other.example.com";
// Error: Index signature in type 'ReadonlyConfig' only permits reading.
```

**Expected Output:**
```
https://api.example.com
```

**Why this output occurs.** The `readonly` modifier on the index signature prevents assignment through indexed access. Reading is permitted; writing is rejected at compile time.

---

#### Example 5: Template literal index signature (TypeScript 4.4+)

```typescript
// example_index_template.ts
interface DataAttributes {
  [key: `data-${string}`]: string;
}

const element: DataAttributes = {
  "data-id": "123",
  "data-testid": "submit-button",
};

console.log(element["data-id"]); // Expected: 123

// const invalid: DataAttributes = { id: "123" };
// Error: Property 'id' is not assignable to the template literal index signature.
```

**Expected Output:**
```
123
```

**Why this output occurs.** The template literal type `` `data-${string}` `` constrains keys to those starting with `data-`. The key `"id"` does not match the pattern and is rejected. This pattern is useful for HTML data attributes, CSS custom properties, and event-handler names.

---

### Real-World Cases

- **Configuration maps**: `interface EnvConfig { [key: string]: string | undefined }` for environment variables.
- **API response caches**: `interface ResponseCache { [endpoint: string]: { data: unknown; timestamp: number } }`.
- **Feature flags**: `interface FeatureFlags { [flagName: string]: boolean }`.
- **CSS custom properties**: `interface ThemeVars { [key: \`--${string}\`]: string | number }`.
- **HTTP status code maps**: `interface StatusMessages { [code: number]: string }`.

---

## 2. Class Index Signatures

### Definitions

**Core Definition.** A **class index signature** is an index signature declared **directly in a class body**, which becomes part of the class's **instance type** and allows instances of the class to be indexed with keys of the declared key type, returning values of the declared value type. It is the mechanism by which a class can support **dynamic property access** beyond its explicitly declared members.

**Technical Definition.** A class index signature is declared as `[key: KeyType]: ValueType` inside the class body. Unlike an interface index signature, a class index signature affects the **instance type** produced by the class and must satisfy the constraint that **all explicitly declared instance properties and methods are assignable to `ValueType`**. A class that `implements` an interface with an index signature **must redeclare the index signature in its own body**; the `implements` clause performs only a conformance check and does **not** add the signature to the class instance type. Class index signatures may be `readonly`, but `readonly` index signatures **cannot be assigned to in the constructor** the way readonly properties can; they must be initialized through other means such as `Object.assign()` or bracket notation before the `readonly` constraint is considered active.

**Beginner-Friendly Explanation.** A class index signature is **an index signature written inside a class**. It says: “Instances of this class can be indexed with arbitrary keys of this type, and every such access returns this value type.” You need it when your class supports dynamic property access—for example, a collection class that stores items by arbitrary string names. An interface with an index signature is **not enough**; the class must repeat the signature in its own body for TypeScript to allow bracket-notation access on instances.

### Purposes

- **To enable bracket-notation access on class instances** with dynamic keys.
- **To satisfy an interface that requires an index signature** when a class implements it.
- **To model collection-like classes** that store values by dynamic names.
- **To type classes that are augmented at runtime** with additional properties.
- **To enforce a consistent value type** for all dynamically added properties on instances.

### Syntax Rules and Structure

**General syntax:**

```ts
class ClassName {
  [key: KeyType]: ValueType;
  // explicit members...
}
```

**Component breakdown:**

- `[key` — opens the index signature.
- `: KeyType` — the key type (`string`, `number`, `symbol`, or template literal).
- `]: ValueType` — the value type for all indexed accesses.
- Explicit members must be assignable to `ValueType`.

**Implements clause interaction:**

```ts
interface IProduct {
  [key: string]: number | boolean;
}

class Product implements IProduct {
  [key: string]: number | boolean; // Must redeclare.
  b: number = 0;
  c: boolean = false;
}
```

- The interface declares the index signature.
- The class **must redeclare** the index signature in its body.
- Explicit fields `b` and `c` are assignable to `number | boolean`.

**Syntax rules.**

- The index signature must be declared in the class body to affect the instance type.
- All explicit instance properties and methods must be assignable to `ValueType`.
- Only one index signature per key type is allowed.
- `readonly` may be applied to the class index signature.
- The index signature may coexist with static members; static members are **not** constrained by the instance index signature.

**Constraints and limitations.**

- **Redeclaration is required when implementing an interface** with an index signature; `implements` alone does not install it.
- Explicit members with incompatible types cause a compile-time error; the index signature's value type must accommodate every explicit member.
- **Readonly class index signatures cannot be assigned to in the constructor** in the way readonly properties can; this is a documented limitation.
- Class index signatures assign the same value type to every key; per-key types require a mapped type or a different design.
- Static properties are not affected by instance index signatures.

### Annotated Complete Step by Step Code Examples

#### Example 1: Class implementing an interface with an index signature

```typescript
// example_class_index_implements.ts
// Step 1: Declare an interface with an index signature.
interface IProduct {
  [key: string]: number | boolean;
}

// Step 2: The class MUST redeclare the index signature.
class Product implements IProduct {
  [key: string]: number | boolean;
  b: number = 0;
  c: boolean = false;
}

// Step 3: Create an instance and use indexed access.
const product = new Product();
product.d = 42;          // Dynamic property added.
console.log(product.b);  // Expected: 0
console.log(product.d);  // Expected: 42

// product.e = "string";
// Error: Type 'string' is not assignable to type 'number | boolean'.
```

**Setup.** Compile with `npx tsc example_class_index_implements.ts --strict`.

**Expected Output:**
```
0
42
```

**Why this output occurs.** The interface declares the index signature, but the class **redeclares** it in its body. Without the redeclaration, TypeScript would report that the class does not implement the interface and that `product.d` has no index signature. With the redeclaration, `product.d = 42` is permitted because `42` is a `number`, which is assignable to `number | boolean`.

---

#### Example 2: Class with a number index signature

```typescript
// example_class_index_number.ts
// Step 1: Declare an interface with a number index signature.
interface Units {
  [key: number]: object;
}

// Step 2: Class redeclares the number index signature.
class Unit implements Units {
  [key: number]: object;

  constructor() {
    for (let x = 1; x <= 3; x++) {
      this[x] = { id: x };
    }
  }
}

// Step 3: Use the instance.
const unit = new Unit();
console.log(unit[1]); // Expected: { id: 1 }
console.log(unit[3]); // Expected: { id: 3 }
```

**Expected Output:**
```
{ id: 1 }
{ id: 3 }
```

**Why this output occurs.** The class redeclares the number index signature from `Units`. The constructor uses bracket notation with numeric keys to assign `object` values. Without the redeclaration, both the `implements` check and the `this[x]` assignment would fail.

---

#### Example 3: Class index signature with explicit members

```typescript
// example_class_index_members.ts
class Settings {
  [key: string]: string | number;

  version: string = "1.0.0";
  timeout: number = 5000;
}

const settings = new Settings();
settings.custom = "value";
settings.retries = 3;

console.log(settings.version); // Expected: 1.0.0
console.log(settings.retries); // Expected: 3
```

**Expected Output:**
```
1.0.0
3
```

**Why this output occurs.** The index signature's value type is `string | number`, which accommodates both `version: string` and `timeout: number`. Additional properties `custom` (string) and `retries` (number) are also permitted. If an explicit member had a type outside `string | number`—for example, `boolean`—the class would fail to compile.

---

#### Example 4: Readonly class index signature limitation

```typescript
// example_class_index_readonly.ts
class Environment {
  readonly [key: string]: string;

  // The following constructor assignment is REJECTED:
  // constructor() {
  //   this["host"] = "localhost";
  // }
  // Error: Index signature in type 'Environment' only permits reading.
}

// Workaround: assign before the readonly constraint takes effect.
const env = new Environment();
(env as Record<string, string>)["host"] = "localhost";
console.log((env as any)["host"]); // Expected: localhost
```

**Expected Output:**
```
localhost
```

**Why this output occurs.** Unlike readonly **properties**, which may be assigned in the constructor, a readonly **index signature** disallows all indexed assignments, including within the constructor. The workaround uses a type assertion to bypass the readonly check, but this sacrifices compile-time safety. For readonly dynamic maps, prefer `Readonly<Record<K, V>>` with `Object.freeze()`.

---

### Real-World Cases

- **ORM model classes** that support dynamic column access by name.
- **Collection classes** that store items by arbitrary string identifiers.
- **Configuration classes** augmented at runtime with plugin-specific settings.
- **Legacy class hierarchies** being incrementally typed with index signatures to allow dynamic properties.

---

## 3. Index Signature Best Practices

### Definitions

**Core Definition.** **Index signature best practices** are guidelines for using index signatures in TypeScript—both in interfaces and in classes—in ways that maximize **type safety, clarity, and tooling effectiveness**, while avoiding the loss of type information and autocomplete that occurs when index signatures are used where more precise types would suffice.

**Technical Definition.** Best practices center on the principle articulated in *Effective TypeScript*: **“If you know the property names, do not use an index signature.”** Index signatures are imprecise by design—they allow any key of the declared type (including typos), require no specific keys, cannot assign distinct types to different keys, and defeat autocomplete. Better alternatives include **interfaces** for known properties, **`Record<K, V>`** for unions of known keys, **optional properties** for partial known sets, **union types** for precise combinations, and **`Map`** for truly dynamic runtime keys. When index signatures are appropriate—allowing additional properties or constraining key patterns—they should be **as narrow as possible**, use **specific value types instead of `any`**, and enable **`noUncheckedIndexedAccess`** to acknowledge that indexed lookups may return `undefined`.

**Beginner-Friendly Explanation.** Best practices are **the habits experienced TypeScript developers use** to keep index signatures from destroying type safety. The biggest rule is: **if you know the property names, do not use an index signature**. Use an interface or a `Record` with specific keys instead. Index signatures let **any** key through, including typos, so they should only be used when the keys are genuinely dynamic. When you do use them, use narrow value types (not `any`) and turn on `noUncheckedIndexedAccess` so TypeScript reminds you that a key might not exist.

### Purposes

- **To preserve type safety** by avoiding `any` and overly broad index signatures.
- **To retain autocomplete and typo detection** by preferring precise types when keys are known.
- **To handle truly dynamic keys safely** with `Map` or narrow index signatures.
- **To enforce value consistency** through specific value types rather than `any`.
- **To make indexed access honest** by enabling `noUncheckedIndexedAccess` and handling `undefined`.

### Syntax Rules and Structure

Best practices are conventions that interact with index signature syntax. The core rules are:

1. **Do not use an index signature when keys are known.** Use an interface, `Record<K, V>` with literal unions, optional properties, or a union type instead.
2. **Avoid `[key: string]: any`.** It disables all type safety and autocomplete. Use the narrowest value type that fits.
3. **Prefer `Record<K, V>` for constrained key sets.** `Record<"GET" | "POST", Handler>` enforces required keys and catches typos.
4. **Use `Map` for truly dynamic keys.** `Map<K, V>` returns `V | undefined` from `get()`, models missing keys honestly, and avoids prototype pollution.
5. **Enable `noUncheckedIndexedAccess`.** Indexed access then returns `ValueType | undefined`, forcing you to handle missing keys.
6. **Narrow index signatures with template literal types.** Use `` `data-${string}` `` or `` `on${Capitalize<string>}` `` to constrain permitted key patterns.
7. **Combine known properties with index signatures carefully.** Explicit property types must be assignable to the index signature's value type; use a union value type to accommodate mixed explicit properties.
8. **Redeclare index signatures in classes.** When a class implements an interface with an index signature, repeat the signature in the class body.
9. **Prefer `readonly` index signatures for immutable maps.** Use `readonly [key: string]: T` or `Readonly<Record<string, T>>` and pair with `Object.freeze()` for runtime immutability.

**Component breakdown (index signature decision tree):**

```
Do you know the property names at compile time?
  ├── Yes → Do NOT use an index signature.
  │        ├── Fixed known set → Use an interface.
  │        ├── Union of known keys → Use Record<K, V>.
  │        └── Partial known set → Use optional properties or a union type.
  └── No → Are the keys truly dynamic at runtime?
           ├── Yes → Use Map<K, V> (safer) or a narrow index signature.
           └── No → Re-examine; you may know more than you think.
```

**Constraints and limitations.**

- Index signatures cannot provide per-key types; mapped types or `Record` with literal unions are required for that.
- `noUncheckedIndexedAccess` may be noisy in codebases that assume keys always exist; enable it deliberately.
- Template literal index signatures require TypeScript 4.4+.
- Readonly class index signatures cannot be assigned in the constructor; prefer `Readonly<Record<K, V>>` with runtime freezing.

### Annotated Complete Step by Step Code Examples

#### Example 1: Replacing an index signature with a precise interface

```typescript
// example_best_interface.ts
// ❌ Imprecise: index signature allows typos and wrong types.
// type Rocket = { [property: string]: string };
// const rocket: Rocket = { name: "Falcon 9", thrust: "7,607 kN" };
// rocket.Name; // Typo compiles!

// ✅ Precise: interface with known properties.
interface Rocket {
  name: string;
  variant: string;
  thrust_kN: number;
}

const falcon: Rocket = {
  name: "Falcon 9",
  variant: "Block 5",
  thrust_kN: 7607,
};

console.log(falcon.name); // Expected: Falcon 9

// falcon.Name; // Error: Property 'Name' does not exist.
// falcon.thrust_kN = "7607"; // Error: string not assignable to number.
```

**Expected Output:**
```
Falcon 9
```

**Why this output occurs.** The interface gives each property its own type and catches typos and type mismatches. The index signature version would have allowed `rocket.Name` and would have required `thrust` to be a `string`, even though it is conceptually a number. This follows the rule: **if you know the keys, do not use an index signature**.

---

#### Example 2: Using `Record` instead of an index signature

```typescript
// example_best_record.ts
// ❌ Index signature: no required keys, no typo protection.
// type HttpHandlers = { [method: string]: (req: Request) => Response };

// ✅ Record with literal union: enforces all keys and catches typos.
type HttpMethod = "GET" | "POST" | "PUT" | "DELETE";
type HttpHandlers = Record<HttpMethod, (req: Request) => Response>;

const handlers: HttpHandlers = {
  GET: (req) => new Response("ok"),
  POST: (req) => new Response("created"),
  PUT: (req) => new Response("updated"),
  DELETE: (req) => new Response("deleted"),
};

console.log(typeof handlers.GET); // Expected: function
```

**Expected Output:**
```
function
```

**Why this output occurs.** `Record<HttpMethod, ...>` requires **all four** methods and rejects any key outside the union. An index signature would have allowed any string key and required none. `Record` provides stronger type safety and better autocomplete.

---

#### Example 3: Narrow index signature with template literals

```typescript
// example_best_template.ts
// ❌ Too broad: allows any string key.
// interface Props { [key: string]: string }

// ✅ Narrow: only data-* keys are allowed.
interface DataProps {
  [key: `data-${string}`]: string;
}

const props: DataProps = {
  "data-testid": "submit",
  "data-value": "42",
};

console.log(props["data-testid"]); // Expected: submit

// const bad: DataProps = { theme: "dark" };
// Error: 'theme' does not match `data-${string}`.
```

**Expected Output:**
```
submit
```

**Why this output occurs.** The template literal index signature constrains keys to the `data-` prefix, catching keys that do not belong. This narrows the index signature to a meaningful pattern instead of accepting any string.

---

#### Example 4: `Map` for truly dynamic keys

```typescript
// example_best_map.ts
// ✅ Map models missing keys honestly.
const scores = new Map<string, number>();
scores.set("alice", 30);

const aliceScore = scores.get("alice");
console.log(aliceScore); // Expected: 30

const bobScore = scores.get("bob");
console.log(bobScore);   // Expected: undefined

// With an index signature, TypeScript would pretend bobScore is number.
```

**Expected Output:**
```
30
undefined
```

**Why this output occurs.** `Map.get()` returns `T | undefined`, correctly reflecting that the key may be missing. An index signature would return `number` for any key, even when the key does not exist—an unsafe lie. For truly dynamic runtime keys, `Map` is safer and avoids prototype pollution.

---

#### Example 5: Enabling `noUncheckedIndexedAccess`

```typescript
// example_best_unchecked.ts
interface Config {
  [key: string]: string;
}

const config: Config = { host: "localhost" };

// With noUncheckedIndexedAccess enabled:
const host: string | undefined = config.host;
console.log(host); // Expected: localhost

const missing: string | undefined = config.missing;
console.log(missing); // Expected: undefined

if (missing !== undefined) {
  console.log(missing.toUpperCase()); // Safe narrowing.
}
```

**Expected Output:**
```
localhost
undefined
```

**Why this output occurs.** Under `noUncheckedIndexedAccess`, every indexed access returns `ValueType | undefined`, forcing the developer to check for absence before using the value. Without this flag, `config.missing` would be typed as `string`, and calling `.toUpperCase()` would crash at runtime.

---

### Real-World Cases

- **API response typing**: prefer interfaces for known fields and `Record` for constrained maps; avoid index signatures when the schema is known.
- **Configuration management**: use `Record<"host" | "port" | "timeout", string | number>` instead of `[key: string]: any`.
- **CSV parsing**: use `Map<string, string>` for dynamic columns, then validate into a typed interface.
- **CSS-in-JS**: use template literal index signatures (`` `--${string}` ``) for custom properties.
- **Feature flags**: use `Record<string, boolean>` (a `Record` alias for a string index signature) when keys are genuinely dynamic, or an interface when the flags are known.

---

## References

- TypeScript Handbook — Index Signatures (Object Types) - https://www.typescriptlang.org/docs/handbook/2/objects.html#index-signatures
- TypeScript Handbook — Indexable Types (Legacy Interfaces) - https://www.typescriptlang.org/docs/handbook/interfaces.html#indexable-types
- TypeScript 4.4 Release Notes — Template Literal Types in Index Signatures - https://www.typescriptlang.org/docs/handbook/release-notes/typescript-4-4.html#template-literal-types
- TSConfig Reference — `noUncheckedIndexedAccess` - https://www.typescriptlang.org/tsconfig/noUncheckedIndexedAccess.html
- TSConfig Reference — `noPropertyAccessFromIndexSignature` - https://www.typescriptlang.org/tsconfig/noPropertyAccessFromIndexSignature.html
- Microsoft/TypeScript Issue #7633 — Index Signatures and `implements` Clauses - https://github.com/microsoft/TypeScript/issues/7633
- Microsoft/TypeScript Issue #6781 — Readonly Index Signatures and Constructors - https://github.com/microsoft/TypeScript/issues/6781
- Microsoft/TypeScript Issue #17845 — Readonly Indexer Assignment in Constructor - https://github.com/microsoft/TypeScript/issues/17845
- Stack Overflow — TypeScript Index Signatures Class Implementation - https://stackoverflow.com/questions/53847195/typescript-index-signatures-class-implementation
- Stack Overflow — TypeScript Index Signature Confusion (Class vs Interface) - https://stackoverflow.com/questions/78557467
- Convex — TypeScript Index Signatures Guide (String, Number, Symbol, Template Literals) - https://www.convex.dev/typescript/advanced/type-operators-manipulation/typescript-index-signature
- OneUptime Blog — How to Implement Index Signatures in TypeScript (Best Practices, Record Comparison) - https://github.com/OneUptime/blog/blob/master/posts/2026-01-30-typescript-index-signatures/README.md
- Effective TypeScript — Prefer More Precise Alternatives to Index Signatures - https://github.com/NeverSight/skills_feed/blob/main/data/skills-md/marius-townhouse/effective-typescript-skills/index-signature-alternatives/SKILL.md
- TypeScript ESLint — `consistent-indexed-object-style` (Index Signature vs `Record`) - https://typescript-eslint.io/rules/consistent-indexed-object-style/